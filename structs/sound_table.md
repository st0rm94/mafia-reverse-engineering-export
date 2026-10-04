# `C_sound_table` — the `sounds.dat` sound bank

**Created** 2026-09-04 08:51 +0200 · **Modified** 2026-09-30 23:08 +0200

PC prefix **`sound_table_`**. Hosted by the tail of the `tools.cpp` translation unit,
`[0x0061cba0, 0x0061d820)`.

The engine's **named-sound registry**. Every "play sound N" in the game goes through it:
a caller hands `sound_table_construct_sound` an empty `I3D_sound` and a numeric sound id,
and the table picks one of up to six alternative `.wav` files by a weighted random roll,
opens it through the sound cache, and configures the object's volume / distance / looping
from a stored playback block. **56 callsites image-wide.**

The whole bank is a shipped data file, `tables\sounds.dat`, read once at startup.

Singleton: **`g_sound_table` @ `0x006f9720`**. It has *no textual existence* in the decomp — every
access is `__thiscall`, so the receiver travels in ECX and a text search for the address returns
nothing. The address comes from the disassembly of `sound_table_static_ctor`
(`mov ecx, 0x006f9720; jmp sound_table_ctor`).

---

## 1. The class

```c
typedef struct C_sound_table {
/* +0x00 */ void            *vt;             // [H] g_sound_table_vt @ 0x0063eee0, 12 slots
/* +0x04 */ int              next_group_id;  // [H] post-incremented by sound_table_add_group
/* +0x08 */ int              next_sound_id;  // [H] post-incremented by sound_table_add_sound
/* +0x0c */ char             rec_alloc;      // [H] std::vector<S_sound_table> allocator byte
/* +0x10 */ S_sound_table   *rec_begin;      // [H]
/* +0x14 */ S_sound_table   *rec_end;        // [H]
/* +0x18 */ S_sound_table   *rec_cap;        // [H]
} C_sound_table;                             // 0x1c bytes
```

The group table is **not** a member: it is a file-scope
`std::vector<S_sound_group>` — `g_sound_group_alloc` / `_begin` / `_end` / `_cap` at
`0x006f973c`..`0x006f9748` — and the class only holds the id counter for it.

## 1a. `S_sound_table` — one sound definition, `0xd0` bytes

```c
typedef struct S_sound_table {
/* +0x00 */ char   *name;              // [H] display name; freed by DelSound/FreeTables
                                       //     NOT restored on load — see §4
/* +0x04 */ int     id;                // [H] the key every consumer looks up
/* +0x08 */ int     group_id;          // [H] matches S_sound_group.id
/* +0x0c */ char   *variant_file[6];   // [H] up to six alternative .wav names
/* +0x24 */ int     variant_thresh[6]; // [H] CUMULATIVE percentages, ASCENDING, last = 100
/* +0x3c */ byte    variant_count;     // [H] 0..6
/* +0x3d */ byte    param_count;       // [H] 0..4 playback blocks
/* +0x3e */ byte    playable;          // [H] 1 = usable, 0 = its file would not open
/* +0x3f */ byte    pad;               // [U]
/* +0x40 */ S_sound_params params[4];  // [H] stride 0x24; param_count of them are live
} S_sound_table;                       // 0x40 + 4*0x24 = 0xd0 exactly
```

```c
typedef struct S_sound_params {        // 0x24 bytes
/* +0x00 */ byte  mode;                // [H] 0/1/2 -> vtable slot 0x58 arg 3/1/2; 3 = error
/* +0x04 */ float a, b, c, d;          // [H] pushed to the object's vtable slot 0x5c
/* +0x14 */ float e, f;                // [H] pushed to slot 0x60
/* +0x1c */ float g;                   // [H] pushed to slot 0x68
/* +0x20 */ float h;                   // [H] pushed to slot 0x64
} S_sound_params;
```

**The `+0x3e` latch.** `sound_table_load_tables` sets it to 1 on every record it reads.
`sound_table_construct_sound` **clears it to 0** the first time the chosen variant's file fails to
open, and every later request for that id then returns 0 at the first gate.
`sound_table_clear_invisible_flag` re-arms every record at once.

In-game observation by the user:

> *"if sound is missing — it is not played, skipped. Nothing happens, no game crash or anything."*

That is exactly what the latch buys: one failed open per id, then silence, and no repeated I/O.

## 1b. The two random rolls

`sound_table_construct_sound` rolls **twice**, and they are independent:

1. **Which file** — `rand()` over 100, then walk `variant_thresh` upward and take the first
   entry that meets the roll. This only selects correctly because the array is ascending,
   which is what `sound_table_sort_variants_by_weight` guarantees.
2. **Which playback block** — a *uniform* roll over `param_count`.

### 1c. The bank is reachable from mission script

`playsoundex xx, yy` — dispatcher `case 0x16` of `program_process_command` — is a direct
script entry point into this class: `xx` is the script's `frm[]` index and the frame must be
an I3D type-4 (sound) frame; **`yy` is a sound id passed straight to
`sound_table_construct_sound`**, i.e. an index into this bank. On success the case enables the
frame (virtual `+0x6c`) and starts it (virtual `+0x88`).

The Mafia Script Data Reference documents `yy` only as *"set to 119 ???"* — 119 is not a magic
constant, it is one `sounds.dat` entry, and any other id in the bank works the same way. This is
also the cheapest way to hear a given entry in game: point a sound frame at it from a script.

---

## 2. PC coverage — 17 of 21

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061cba0 | `sound_table_ctor` | `__ct__13C_sound_tableFv` | 0x003b01e0 | CONFIRMED | install the vtable, zero both counters and the vector |
| 0x0061cbd0 | `sound_table_dtor` | `__dt__13C_sound_tableFv` | 0x003b0250 | CONFIRMED | re-install vtable, `FreeTables`, destroy the vector |
| 0x0061cc30 | `sound_table_free_tables` | `FreeTables__13C_sound_tableFv` | 0x003b02d0 | CONFIRMED | free all seven owned strings per record, then empty |
| 0x0061cd00 | `sound_table_add_group` | `AddGroup__13C_sound_tableFPCc` | 0x003b03b0 | CONFIRMED | slot 1 — append a named group, assign the next id |
| 0x0061cd50 | `sound_table_del_group` | `DelGroup__13C_sound_tableFi` | 0x003b0470 | CONFIRMED | slot 2 — release the group's sounds, then swap-and-pop |
| 0x0061ce50 | `sound_table_add_sound` | `AddSound__13C_sound_tableFPCcUi` | 0x003b05f0 | CONFIRMED | slot 3 — append a zeroed record with name, id, group |
| 0x0061cef0 | `sound_table_del_sound` | `DelSound__13C_sound_tableFUi` | 0x003b0830 | CONFIRMED | slot 4 — find by id, free its strings, shift down |
| 0x0061cfe0 | `sound_table_get_record_by_id` | `GetTableFromIDX__13C_sound_tableFUi` | 0x003b0920 | CONFIRMED | slot 7 — linear search on `+0x04`; 0 on a miss |
| 0x0061d040 | `sound_table_set_file_name` | `SetFileName__13C_sound_tableFPQ213C_sound_table13S_sound_tableiPCc` | 0x003b0980 | CONFIRMED | slot 5 — replace variant `i`'s filename |
| 0x0061d0a0 | `sound_table_sort_variants_by_weight` | `OptimalizeTable__13C_sound_tableFPQ213C_sound_table13S_sound_table` | 0x003b09f0 | CONFIRMED | slot 6 — selection-sort the thresholds ascending |
| 0x0061d160 | `sound_table_load_tables` | `LoadTables__13C_sound_tableFPCc` | 0x003b0aa0 | CONFIRMED | read `tables\sounds.dat`; sole caller `engine_init` |
| 0x0061d3b0 | `sound_table_save_tables` | `SaveTables__13C_sound_tableFPCc` | 0x003b0f90 | CONFIRMED | slot 0 — the exact mirror of the loader |
| 0x0061d5f0 | `sound_table_construct_sound` | `ConstructSound__13C_sound_tableFP9I3D_soundUi` | 0x003b1220 | CONFIRMED | the whole point of the class; 56 callsites |
| 0x0061d7d0 | `thunk_sound_table_construct_sound` | `TestConstructSound__13C_sound_tableFP9I3D_soundUi` | 0x003b2330 | CONFIRMED | slot 10 — folded to a jump on both builds |
| 0x0061d7e0 | `sound_table_clear_invisible_flag` | `ClearInvisibleFlag__13C_sound_tableFv` | 0x003b2340 | CONFIRMED | slot 11 — re-arm `+0x3e` on every record |
| 0x0061dbb0 | `sound_table_get_num_sounds` | `GetNumSounds__13C_sound_tableFv` | 0x003b4f10 | CONFIRMED | slot 8 — record count |
| 0x004253a0 | `folded_get_dword_0x10` | `GetSounds__13C_sound_tableFv` | 0x003b4f20 | CONFIRMED | slot 9 — `return this+0x10`; a 3-owner fold (§5) |
| - | - | `GameInit__13C_sound_tableFv` | 0x003b1950 | PS2-port-only CANDIDATE | §6 |
| - | - | `GameDone__13C_sound_tableFv` | 0x003b1a20 | PS2-port-only CANDIDATE | §6 |
| - | - | `PreloadSoundPermanent__13C_sound_tableFUi` | 0x003b1af0 | PS2-port-only CANDIDATE | §6 |
| - | - | `GetPreloadedSound__13C_sound_tableFUi` | 0x003b20c0 | PS2-port-only CANDIDATE | §6 |

Two container helpers belong to the class's record vector rather than to the class:
`sound_table_record_list_erase_range` (`0x0061db10`) and `sound_table_record_list_insert_n`
(`0x0061dbe0`). A census of their callsites finds one and two respectively, every one loading
`C_sound_table + 0x0c`, and no reference on the other three channels (`E9` jumps, data pointers,
vtables). Their neighbour `std_vector10_erase_range` (`0x0061db60`) has five callsites in five
unrelated modules, so it is a generic helper.

---

## 3. The vtable — 12 slots

`sound_table_ctor` installs **`g_sound_table_vt` @ `0x0063eee0`**. The vtable is what ties the
scattered members to the class.

| slot | PC | role |
|---|---|---|
| 0 | `sound_table_save_tables` | write the bank |
| 1 | `sound_table_add_group` | - |
| 2 | `sound_table_del_group` | - |
| 3 | `sound_table_add_sound` | - |
| 4 | `sound_table_del_sound` | - |
| 5 | `sound_table_set_file_name` | - |
| 6 | `sound_table_sort_variants_by_weight` | - |
| 7 | `sound_table_get_record_by_id` | - |
| 8 | `sound_table_get_num_sounds` | 0x430 bytes past the class's own band |
| 9 | `folded_get_dword_0x10` | a shared fold 1.9 MB away (§5) |
| 10 | `thunk_sound_table_construct_sound` | a 5-byte `E9` |
| 11 | `sound_table_clear_invisible_flag` | no textual caller at all |

`sound_table_load_tables` and `sound_table_construct_sound` are **not** virtual.

---

## 4. `tables\sounds.dat` — the file format, round-trip verified

Loaded once, from `engine_init`, immediately after `anim_index_init`:
`mov ecx, 0x006f9720 / push 0x6513ac / call sound_table_load_tables`, where `0x6513ac` is
the literal `"tables\sounds.dat"`.

```
u32   version              // must be 2 or the loader closes the file and returns
u32   next_group_id
u32   next_sound_id
u32   group_count
S_sound_group[group_count] // 0x24 each: char name[0x20]; u32 id
u32   record_count
record_count x {
    u32  name_len;  char name[name_len]      // NO terminator
    u32  id
    u32  group_id
    u8   variant_count
    u8   param_count
    variant_count x { u32 len; char file[len]; u32 threshold }
    byte params[param_count * 0x24]
}
```

**Verified against the shipped file** (in the repo at `game-sources/tables/sounds.dat`): a parser
derived from the code alone consumes **39 760 of 39 760 bytes exactly**, with `version = 2`,
26 groups, 311 records, `next_group_id = 28`, `next_sound_id = 345`.

The data corroborates two naming decisions independently of any body read:

* **every record's thresholds are ascending and the last is 100** — which is what
  `sound_table_sort_variants_by_weight` exists to guarantee and what the `rand()`-over-100
  walk consumes. Worked example, sound id 22 *"Body"* in group 5 *"Hit baseball bat"*:
  `b_body1..6.wav` at 16 / 32 / 48 / 64 / 80 / 100 — six evenly weighted alternates.
* **variant counts run 0..6 and param counts 0..2**, inside the 6 and 4 slots the record
  layout provides. The **38 records with zero variants** are surface materials (`Dlazba`,
  `Koleje (cely tramvajovy pas)`, `Mramor/kamen/dlazdice`, `Sklo`, …) and are exactly the
  ones the `variant_count != 0` gate rejects early.

The 26 groups: `Weapons`, `Steps`, `Reload`, `Grenade & Molotov`, `Hit baseball bat`,
`Hit bullet`, `Cartridge drop`, `Hit punch`, `Actors`, `Script`, `Action`, `Hit knife`,
`Cartridge drop shotgun`, and thirteen Czech `Fyzika - …` (physics-material) groups.

**Two fields are written and then ignored on load.** The loader reads each record's
`name_len` and **seeks past the name** — so `record+0x00` stays NULL at runtime — and it
reads the group count and **seeks past the whole group block**. Both are authored by the
tool that produced the file; only ids matter to the engine. So the human-readable names
above exist in the FILE and never in memory: anything that wants to show a sound's name has
to re-read `sounds.dat`.

The `mode` byte is 1 in 298 of the 299 shipped blocks and 0 once. Modes 2 and 3 never
occur, and mode 3 is the arm that stops the renderer and raises a message box — a
developer assert.

---

## 5. Slot 9 is a three-owner fold

Slot 9 is `folded_get_dword_0x10` (`0x004253a0`), whose body is `return this+0x10`: MSVC's
identical-COMDAT folding merged this class's slot-9 accessor onto the address that also serves
`C_actor::GetKind` (~317 calls) and `C_mission::GetScene` (29 calls).

**No callsite census can find this owner**: slot 9 is never called by name, so the function has zero
`E8` references from this class and exactly one data reference — the vtable entry at `0x0063ef04`.
The owner comes from the vtable-sequence comparison in §6. No `PS2 twin:` line is stamped on the
folded symbol (it has three owners); the mapping lives in the §2 table, in the fold's registry row
and in an inline comment in its body.

---

## 6. PC vs PS2

**The PS2 record is the PC record plus 0x18 bytes.** PS2 stride `0xe8`, PC stride `0xd0`.
The difference is an array of **six preloaded `I3D_sound *` at `record + 0x40`**, which
pushes the playback blocks from `+0x40` to `+0x58`. Both builds then hold four `0x24`-byte
blocks: `0x58 + 4*0x24 = 0xe8` and `0x40 + 4*0x24 = 0xd0`. Exact, both ways.

That array is the whole of the four unmatched methods:

| PS2 method | what it does | why the PC cannot have it |
|---|---|---|
| `PreloadSoundPermanent` | fills `record+0x40` with opened `I3D_sound` objects | the PC record has no such field |
| `GetPreloadedSound` | picks one of them with `rnd(variant_count)` and plays it without touching the cache | the PC record has no such field |
| `GameInit` / `GameDone` | release all six pointers of every record and null them | there is nothing to release |

Graded a **PS2-port-only CANDIDATE**, not a terminal tag: the structural no-room argument is strong
and the 32 MB motive fits — keeping a permanent decoded copy of a sound avoids a cache miss
mid-gameplay — but the caller set is not closed (§8).

**Other differences, all representational rather than behavioural:**

* The PS2 class stores `{count, base}` at `+0x0c`/`+0x10` where the PC stores
  `{begin, end, cap}` at `+0x10`/`+0x14`/`+0x18`, and its vtable pointer is at `+0x14`
  rather than `+0x00`. Field roles are otherwise identical, and `record+0x3e` is literally
  the same offset on both builds.
* The PS2 links Metrowerks' STL, so its `std::vector` grows by **doubling** while MSVC
  **adds**; that is the whole of the 0.62 size ratio on the insert helper.
* PS2 `ConstructSound` **inlines** `C_I3D_cache<I3D_sound>::Open` (its
  `rep_fname_localstatic3_Open__23C_I3D_cache<9I3D_sound>…` local statics are the tell)
  where the PC calls `i3d_cache_sound_open`. That single difference is the entire 480 B
  against 1840 B ratio of 0.26; the bodies are otherwise statement-identical. Marked
  inline at the callsite.
* Four PC sites inline what the PS2 calls out of line — the `vector_deleter` destructor and
  `clear()` for both element types, and the Metrowerks `compressed_pair_imp::first()`
  size accessor. All four are bannered in the decomp.
* `TestConstructSound` is folded to a jump into `ConstructSound` on **both** builds. Its
  PS2 symbol is 8 bytes, which is a MIPS tail jump, not an empty stub.

**Slot 9 (§5).** The PS2 `GetSounds__13C_sound_tableFv` is `return this+0x10`; comparing **both
builds' vtable sequences** slot by slot pairs it with PC slot 9, `folded_get_dword_0x10`.

---

## 7. Related

* `structs/soundArena.md` — the *other* sound object in this translation unit; it is the
  per-mission propagation ring and has nothing to do with this class.
* `structs/i3d_cache.md` — `g_i3d_sound_cache` @ `0x006f93f0`, which
  `sound_table_construct_sound` opens through.
* `basic/artifacts/dll.md` — `ls3df.dll`, where the `I3D_sound` vtable consumers live.

---

## 8. Open items

* **The four PS2-port-only candidates are not closed.** What would close it: census every PS2
  caller of `PreloadSoundPermanent` and check they are all in port-added code.
* **`sound_table_clear_invisible_flag` has no caller.** Its only reference in the whole
  image is the vtable slot. Either the game never re-arms a failed sound within a session,
  or it is reached through a slot-11 dispatch nothing static can see. Worth an in-game
  check across a mission reload.
* **`S_sound_params`' eight floats are undecoded.** Their consumers are the target
  `I3D_sound`'s vtable slots `0x5c` / `0x60` / `0x64` / `0x68`, which live in `ls3df.dll`
  and are therefore not in this binary at all (`basic/artifacts/dll.md`). The shipped values
  are suggestive — block 0 of *Explosion* is `10, 155, 0.4, 0.3, 0, 0, 1, 0`, and 155 next to
  10 reads like a max/min audible distance pair — but nothing here proves it.
