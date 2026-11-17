# `C_sound_table` — the `sounds.dat` sound bank

**Created** 2026-09-04 08:51 +0200 · **Modified** 2026-11-16 16:16 +0100

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

| | |
|---|---|
| PC prefix | `sound_table_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_sound_table_vt` @ `0x0063eee0` (12 slots) |
| PC base class | - |
| PC instance size | `0x1c` |
| PC constructor | `sound_table_ctor` @ `0x0061cba0` |
| PC translation unit | `[0x0061b050, 0x0061df00)`, the `tools.cpp` unit, guard to guard, 57 starts; this doc hosts it (§4). The class's own methods are `[0x0061cba0, 0x0061d820)` plus the helpers at the tail |
| PS2 class | `C_sound_table`, 21 methods, 17 mapped |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_sound_table {
    /* +0x00 */ void          *vt;            // [H] g_sound_table_vt
    /* +0x04 */ int            next_group_id; // [H] post-incremented by sound_table_add_group
    /* +0x08 */ int            next_sound_id; // [H] post-incremented by sound_table_add_sound
    /* +0x0c */ char           rec_alloc;     // [H] std::vector<S_sound_table> allocator byte
    /* +0x10 */ S_sound_table *rec_begin;     // [H] the records: begin
    /* +0x14 */ S_sound_table *rec_end;       // [H] the records: end
    /* +0x18 */ S_sound_table *rec_cap;       // [H] the records: capacity
} C_sound_table;                              // 0x1c bytes
```

The group table is **not** a member: it is a file-scope
`std::vector<S_sound_group>` (`g_sound_group_alloc` / `_begin` / `_end` / `_cap` at
`0x006f973c`..`0x006f9748`), and the class only holds the id counter for it.

### 1a. `S_sound_table` — one sound definition, `0xd0` bytes

```c
typedef struct S_sound_table {
    /* +0x00 */ char   *name;              // [H] display name; freed by DelSound / FreeTables; not restored on load (§5)
    /* +0x04 */ int     id;                // [H] the key every consumer looks up
    /* +0x08 */ int     group_id;          // [H] matches S_sound_group.id
    /* +0x0c */ char   *variant_file[6];   // [H] up to six alternative .wav names
    /* +0x24 */ int     variant_thresh[6]; // [H] cumulative percentages, ascending, last = 100
    /* +0x3c */ byte    variant_count;     // [H] live entries of the two arrays above, 0..6
    /* +0x3d */ byte    param_count;       // [H] live entries of params, 0..4
    /* +0x3e */ byte    playable;          // [H] 1 = usable, 0 = its file would not open
    /* +0x3f */ byte    pad_3f;            // [U] no access; alignment before params
    /* +0x40 */ S_sound_params params[4];  // [H] the playback blocks, stride 0x24
} S_sound_table;                           // 0x40 + 4*0x24 = 0xd0 exactly
```

**Why these counts.** `variant_file` and `variant_thresh` are parallel: entry `i` is one alternative `.wav` and the
cumulative percentage up to which the file roll picks it (§1d). `params` holds alternative playback settings for the same
sound, picked by a second, uniform roll. The shipped file uses at most six variants and at most two blocks (§5): six and
four are the room the record provides, the loader writes `variant_count` / `param_count` entries, and nothing in the class
states where the limits come from.

### 1b. `S_sound_params` — one playback block, `0x24` bytes

```c
typedef struct S_sound_params {        // the I3D_sound setters are named in §7
    /* +0x00 */ byte  mode;            // [H] 0/1/2 -> sound type 3/1/2 (vtable slot 0x58); 3 = error
    /* +0x04 */ float range[4];        // [H] the four arguments of the range setter, slot 0x5c
    /* +0x14 */ float cone[2];         // [H] inner and outer cone angle in radians, slot 0x60
    /* +0x1c */ float volume;          // [H] slot 0x68
    /* +0x20 */ float out_volume;      // [H] the volume outside the cone, slot 0x64
} S_sound_params;
```

`range` and `cone` have the entry counts of the two setters they feed (`range`: Open item 2):
block 0 of the shipped *Explosion* record is range `10, 155, 0.4, 0.3`, cone `0, 0`, volume 1, outside-cone volume 0.

**The shape of `range` in the shipped file** (all 299 blocks): `range[0] < range[1]` in 298 and equal once; `range[0]`
runs 0..150 (3, 5 and 4 the most common), `range[1]` 0..690 (20, 15, 25), `range[2]` 0..0.5 (0.4 in 169 blocks, 0.3 in
108), `range[3]` 0..1 (0.3 in 149, 0.4 in 127). `speech_channel_speak_line` passes the same setter two literal sets,
`(1.0, 80.0, 0.3, 0.25)` and `(2.5, 30.0, 0.35, 0.2)`. So the first two read as distances, the second the larger, and the
last two as fractions; what each one sets is inside `ls3df.dll` (Open item 2). Bytes `+0x01..+0x03` of every shipped
block are 0.

### 1c. `S_sound_group` — one group, `0x24` bytes

```c
typedef struct S_sound_group {
    /* +0x00 */ char name[0x20];       // [H] the group's display name, zero-padded in the file
    /* +0x20 */ int  id;               // [H] the key S_sound_table.group_id holds
} S_sound_group;
```

`name` is a constant-size text buffer of the file format (§5), so `0x20` is its capacity in characters, not an entry count.

### 1d. The two random rolls

`sound_table_construct_sound` rolls **twice**, and they are independent:

1. **Which file** — `rand()` over 100, then walk `variant_thresh` upward and take the first
   entry that meets the roll. This only selects correctly because the array is ascending,
   which is what `sound_table_sort_variants_by_weight` guarantees.
2. **Which playback block** — a *uniform* roll over `param_count`.

**The `+0x3e` latch.** `sound_table_load_tables` sets it to 1 on every record it reads.
`sound_table_construct_sound` **clears it to 0** the first time the chosen variant's file fails to
open, and every later request for that id then returns 0 at the first gate.
`sound_table_clear_invisible_flag` re-arms every record at once.

If sound is missing, it's skipped. That is exactly what the latch buys: one failed open per id, then silence, and no repeated I/O.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vt | H | `sound_table_ctor`, `sound_table_dtor` | `sound_table_del_group` | read as the receiver of the slot calls |
| +0x4 | next_group_id | H | `sound_table_add_group`, `sound_table_ctor`, `sound_table_load_tables` | `sound_table_add_group`, `sound_table_load_tables`, `sound_table_save_tables` | the file's value on load |
| +0x8 | next_sound_id | H | `sound_table_add_sound`, `sound_table_ctor`, `sound_table_load_tables` | `sound_table_add_sound`, `sound_table_load_tables`, `sound_table_save_tables` | the file's value on load |
| +0xc | rec_alloc | H | `sound_table_ctor` | `sound_table_add_sound`, `sound_table_load_tables` | the address is the receiver of the two record-vector helpers |
| +0x10 | rec_begin | H | `sound_table_ctor`, `sound_table_dtor`, `sound_table_record_list_insert_n` | `sound_table_clear_invisible_flag`, `sound_table_construct_sound`, `sound_table_del_group`, `sound_table_del_sound`, `sound_table_dtor`, `sound_table_free_tables`, `sound_table_get_num_sounds`, `sound_table_get_record_by_id`, `sound_table_load_tables`, `sound_table_save_tables` | - |
| +0x14 | rec_end | H | `sound_table_ctor`, `sound_table_del_sound`, `sound_table_dtor`, `sound_table_free_tables`, `sound_table_record_list_erase_range`, `sound_table_record_list_insert_n` | `sound_table_add_sound`, `sound_table_clear_invisible_flag`, `sound_table_construct_sound`, `sound_table_del_group`, `sound_table_del_sound`, `sound_table_free_tables`, `sound_table_get_num_sounds`, `sound_table_get_record_by_id`, `sound_table_load_tables`, `sound_table_save_tables` | - |
| +0x18 | rec_cap | H | `sound_table_ctor`, `sound_table_dtor`, `sound_table_record_list_insert_n` | `sound_table_record_list_insert_n` | - |

### 2a. `S_sound_table`

The cells of §2a to §2c come from the bodies and a displacement census over the class's functions: the records are walked with a moving pointer.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `sound_table_add_sound`, `sound_table_del_sound` | `sound_table_del_sound`, `sound_table_free_tables` | the loader seeks past the name (§5), so at run time it is non-NULL only for a record `sound_table_add_sound` made |
| +0x4 | id | H | `sound_table_add_sound`, `sound_table_load_tables` | `sound_table_construct_sound`, `sound_table_del_sound`, `sound_table_get_record_by_id`, `sound_table_save_tables` | - |
| +0x8 | group_id | H | `sound_table_add_sound`, `sound_table_load_tables` | `sound_table_del_group`, `sound_table_save_tables` | - |
| +0xc | variant_file | H | `sound_table_load_tables`, `sound_table_set_file_name`, `sound_table_sort_variants_by_weight` | `sound_table_construct_sound`, `sound_table_del_sound`, `sound_table_free_tables`, `sound_table_save_tables`, `sound_table_sort_variants_by_weight` | - |
| +0x24 | variant_thresh | H | `sound_table_load_tables`, `sound_table_sort_variants_by_weight` | `sound_table_construct_sound`, `sound_table_save_tables`, `sound_table_sort_variants_by_weight` | - |
| +0x3c | variant_count | H | `sound_table_add_sound`, `sound_table_load_tables` | `sound_table_construct_sound`, `sound_table_save_tables`, `sound_table_sort_variants_by_weight` | - |
| +0x3d | param_count | H | `sound_table_add_sound`, `sound_table_load_tables` | `sound_table_construct_sound`, `sound_table_save_tables` | - |
| +0x3e | playable | H | `sound_table_clear_invisible_flag`, `sound_table_construct_sound`, `sound_table_load_tables` | `sound_table_construct_sound` | §1d |
| +0x3f | pad_3f | U | - | - | no access; the record is zero-filled and copied whole, so the byte is the alignment gap before `params` |
| +0x40 | params | H | `sound_table_load_tables` | `sound_table_construct_sound`, `sound_table_save_tables` | §2b |

### 2b. `S_sound_params`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | mode | H | `sound_table_load_tables` | `sound_table_construct_sound` | §5 |
| +0x4 | range | H | `sound_table_load_tables` | `sound_table_construct_sound` | - |
| +0x14 | cone | H | `sound_table_load_tables` | `sound_table_construct_sound` | - |
| +0x1c | volume | H | `sound_table_load_tables` | `sound_table_construct_sound` | - |
| +0x20 | out_volume | H | `sound_table_load_tables` | `sound_table_construct_sound` | - |

### 2c. `S_sound_group`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `sound_table_add_group` | `sound_table_save_tables` | the loader seeks past the file's whole group block (§5) |
| +0x20 | id | H | `sound_table_add_group` | `sound_table_del_group`, `sound_table_save_tables` | - |

## 3. Method family

17 of 21 PS2 methods mapped; the other 4 are in §7. Two container helpers belong to the class's record vector: a census of
their callsites finds one and two respectively, every one loading `C_sound_table + 0x0c`, and no reference on the other
three channels (`E9` jumps, data pointers, vtables). This doc hosts the `tools.cpp` unit: its CRT pair and the static
quartets and table initializers no class of the unit claims sit here; every other class of the unit points at this section.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0061b050 | `crt_static_init_guard_0061b050` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0061b070 | `crt_atexit_register_0061b070` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x0061b080 | `thunk_dir8_unit_xz_006f96c0_static_init` | `0x006f96c0` | - | - | - | a jump to the next row |
| 0x0061b090 | `dir8_unit_xz_006f96c0_static_init` | `0x006f96c0` | - | - | - | fill the 8-direction unit vector table |
| 0x0061b190 | `thunk_dir8_square_xz_006f9590_static_init` | `0x006f9590` | - | - | - | a jump to the next row |
| 0x0061b1a0 | `dir8_square_xz_006f9590_static_init` | `0x006f9590` | - | - | - | fill the 8-direction square-ring table |
| 0x0061b2a0 | `thunk_dir16_square_xz_006f95f0_static_init` | `0x006f95f0` | - | - | - | a jump to the next row |
| 0x0061b2b0 | `dir16_square_xz_006f95f0_static_init` | `0x006f95f0` | - | - | - | fill the 16-direction square-ring table |
| 0x0061b4a0 | `sound_group_static_init` | `g_sound_group_alloc` | - | - | - | `.CRT$XCU` entry of the group vector |
| 0x0061b4b0 | `sound_group_static_ctor` | `g_sound_group_alloc` | - | - | - | construct the group vector |
| 0x0061b4d0 | `sound_group_register_atexit_dtor` | `g_sound_group_alloc` | - | - | - | registers the next row |
| 0x0061b4e0 | `sound_group_static_dtor` | `g_sound_group_alloc` | - | - | - | frees the group vector |
| 0x0061b500 | `sound_table_static_init` | `g_sound_table` | - | - | - | `.CRT$XCU` initializer of the singleton |
| 0x0061b510 | `sound_table_static_ctor` | `g_sound_table` | - | - | - | `mov ecx, 0x006f9720; jmp sound_table_ctor` |
| 0x0061b520 | `sound_table_register_atexit_dtor` | `g_sound_table` | - | - | - | registers the next row |
| 0x0061b530 | `sound_table_static_dtor` | `g_sound_table` | - | - | - | runs `sound_table_dtor` on the singleton |
| 0x0061cba0 | `sound_table_ctor` | instance | `__ct__13C_sound_tableFv` | 0x003b01e0 | CONFIRMED | install the vtable, zero both counters and the vector |
| 0x0061cbd0 | `sound_table_dtor` | instance | `__dt__13C_sound_tableFv` | 0x003b0250 | CONFIRMED | re-install the vtable, `FreeTables`, destroy the vector |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004253a0 | `folded_get_dword_0x10` | `GetSounds__13C_sound_tableFv` | 0x003b4f20 | CONFIRMED | slot 9: `return this+0x10`; a three-owner fold (§6a), also in `structs/actor.md` and `structs/mission.md` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061b540 | `compass_activate` | `Activate__9C_compassFP9I3D_frame` | 0x003ad490 | CONFIRMED | a method of the `C_compass` embedded in `C_game` (`structs/game.md`), hosted here: sets the rectangle from the engine's screen size; called by `game_init` and `game_done` |
| 0x0061cc30 | `sound_table_free_tables` | `FreeTables__13C_sound_tableFv` | 0x003b02d0 | CONFIRMED | free all seven owned strings per record, then empty |
| 0x0061cd00 | `sound_table_add_group` | `AddGroup__13C_sound_tableFPCc` | 0x003b03b0 | CONFIRMED | slot 1: append a named group, assign the next id |
| 0x0061cd50 | `sound_table_del_group` | `DelGroup__13C_sound_tableFi` | 0x003b0470 | CONFIRMED | slot 2: release the group's sounds, then swap-and-pop |
| 0x0061ce50 | `sound_table_add_sound` | `AddSound__13C_sound_tableFPCcUi` | 0x003b05f0 | CONFIRMED | slot 3: append a zeroed record with name, id, group |
| 0x0061cef0 | `sound_table_del_sound` | `DelSound__13C_sound_tableFUi` | 0x003b0830 | CONFIRMED | slot 4: find by id, free its strings, shift down |
| 0x0061cfe0 | `sound_table_get_record_by_id` | `GetTableFromIDX__13C_sound_tableFUi` | 0x003b0920 | CONFIRMED | slot 7: linear search on `+0x04`; 0 on a miss |
| 0x0061d040 | `sound_table_set_file_name` | `SetFileName__13C_sound_tableFPQ213C_sound_table13S_sound_tableiPCc` | 0x003b0980 | CONFIRMED | slot 5: replace variant `i`'s filename |
| 0x0061d0a0 | `sound_table_sort_variants_by_weight` | `OptimalizeTable__13C_sound_tableFPQ213C_sound_table13S_sound_table` | 0x003b09f0 | CONFIRMED | slot 6: selection-sort the thresholds ascending |
| 0x0061d160 | `sound_table_load_tables` | `LoadTables__13C_sound_tableFPCc` | 0x003b0aa0 | CONFIRMED | read `tables\sounds.dat`; sole caller `engine_init` |
| 0x0061d3b0 | `sound_table_save_tables` | `SaveTables__13C_sound_tableFPCc` | 0x003b0f90 | CONFIRMED | slot 0: the exact mirror of the loader |
| 0x0061d5f0 | `sound_table_construct_sound` | `ConstructSound__13C_sound_tableFP9I3D_soundUi` | 0x003b1220 | CONFIRMED | the whole point of the class; 56 callsites |
| 0x0061d7d0 | `thunk_sound_table_construct_sound` | `TestConstructSound__13C_sound_tableFP9I3D_soundUi` | 0x003b2330 | CONFIRMED | slot 10: folded to a jump on both builds |
| 0x0061d7e0 | `sound_table_clear_invisible_flag` | `ClearInvisibleFlag__13C_sound_tableFv` | 0x003b2340 | CONFIRMED | slot 11: re-arm `+0x3e` on every record |
| 0x0061db10 | `sound_table_record_list_erase_range` | - | - | - | erases a record range; one callsite, `sound_table_load_tables` |
| 0x0061dbb0 | `sound_table_get_num_sounds` | `GetNumSounds__13C_sound_tableFv` | 0x003b4f10 | CONFIRMED | slot 8: record count |
| 0x0061dbe0 | `sound_table_record_list_insert_n` | - | - | - | inserts records; two callsites |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061db60 | `std_vector10_erase_range` | - | - | - | five callsites in five unrelated modules: a generic helper |

## 4. The unit this doc hosts

`[0x0061b050, 0x0061df00)` is the `tools.cpp` unit, guard to guard. Its class runs are documented in their own docs, whose
cards point here: `fire_*` (`structs/fire.md`), `using_object_*` (`structs/using_object.md`), `screen_fade_tick`
(`structs/screen_fade.md`) and `sound_arena_*` (`structs/sound_arena.md`).

### 4a. The bank is reachable from mission script

`playsoundex xx, yy` — dispatcher `case 0x16` of `program_process_command` — is a direct
script entry point into this class: `xx` is the script's `frm[]` index and the frame must be
an I3D type-4 (sound) frame; **`yy` is a sound id passed straight to
`sound_table_construct_sound`**, i.e. an index into this bank. On success the case enables the
frame (virtual `+0x6c`) and starts it (virtual `+0x88`).

The Mafia Script Data Reference documents `yy` only as *"set to 119 ???"* — 119 is not a magic
constant, it is one `sounds.dat` entry, and any other id in the bank works the same way. This is
also the cheapest way to hear a given entry in game: point a sound frame at it from a script.

## 5. `tables\sounds.dat` — the file format, round-trip verified

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

## 6. The vtable — 12 slots

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
| 9 | `folded_get_dword_0x10` | a shared fold 1.9 MB away (§6a) |
| 10 | `thunk_sound_table_construct_sound` | a 5-byte `E9` |
| 11 | `sound_table_clear_invisible_flag` | no textual caller at all |

`sound_table_load_tables` and `sound_table_construct_sound` are **not** virtual.

### 6a. Slot 9 is a three-owner fold

Slot 9 is `folded_get_dword_0x10` (`0x004253a0`), whose body is `return this+0x10`: MSVC's
identical-COMDAT folding merged this class's slot-9 accessor onto the address that also serves
`C_actor::GetKind` (~317 calls) and `C_mission::GetScene` (29 calls).

**No callsite census can find this owner**: slot 9 is never called by name, so the function has zero
`E8` references from this class and exactly one data reference — the vtable entry at `0x0063ef04`.
The owner comes from the vtable-sequence comparison in §7. No `PS2 twin:` line is stamped on the
folded symbol (it has three owners); the mapping lives in the §3c table, in the fold's registry row
and in an inline comment in its body.

## 7. PC vs PS2

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

**PS2-port-only, measured over a closed caller set.** `PreloadSoundPermanent`'s only caller is the
free function `PreloadPicovinNaZasranyPSKram__Fv` (a vulgar developer name that singles out the
PS2), called once from `C_game::Init`; the PC `game_init` has no such preload. `GameInit` and
`GameDone` are called once each, from `C_game::Init` and `C_game::Done`. Where the PS2 callers of
`GetPreloadedSound` take a preloaded sound, the PC twins open the sound through
`sound_table_construct_sound` (`human_ai`, `game_tick_shoot`).

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

**Slot 9 (§6a).** The PS2 `GetSounds__13C_sound_tableFv` is `return this+0x10`; comparing **both
builds' vtable sequences** slot by slot pairs it with PC slot 9, `folded_get_dword_0x10`.

**The playback block's setters (§1b).** The PS2 `ConstructSound` calls the same `I3D_sound`
virtuals the PC calls by slot, and the PS2 symbol table names them: the mode goes to
`SetSoundType__9I3D_soundF13I3D_SOUNDTYPE`, the four range floats to `SetRange__9I3D_soundFffff`,
the two cone floats to `SetCone__9I3D_soundFff`, then `SetVolume__9I3D_soundFf` and
`SetOutVol__9I3D_soundFf`. Each PC slot is the PS2 slot minus `0x2c` (`0x84`/`0x88`/`0x8c`/
`0x94`/`0x90` against `0x58`/`0x5c`/`0x60`/`0x68`/`0x64`). The engine code linked into the PS2
build shows what the setters store: `SetCone` clamps its two angles and keeps the cosine of each
half-angle; `SetRange` keeps its four floats, whose constructor defaults are `1.0, 10.0, 1.0, 0.0`;
the default volume is 1.0 and the default outside-cone volume 0.

### 7a. PS2 methods without a PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x003b1950 | `GameInit__13C_sound_tableFv` | PS2-port-only: releases the preloaded array the PC record has no field for |
| 0x003b1a20 | `GameDone__13C_sound_tableFv` | PS2-port-only: the same |
| 0x003b1af0 | `PreloadSoundPermanent__13C_sound_tableFUi` | PS2-port-only: fills the preloaded array; only caller `PreloadPicovinNaZasranyPSKram__Fv` |
| 0x003b20c0 | `GetPreloadedSound__13C_sound_tableFUi` | PS2-port-only: the PC twins of its callers open the sound through `sound_table_construct_sound` |

## 8. Related

* `structs/sound_arena.md` — the *other* sound object in this translation unit; it is the
  per-mission propagation ring and has nothing to do with this class.
* `structs/i3d_cache.md` — `g_i3d_sound_cache` @ `0x006f93f0`, which
  `sound_table_construct_sound` opens through.
* `basic/artifacts/dll.md` — `ls3df.dll`, where the `I3D_sound` vtable consumers live.

## 9. Open items

* **`sound_table_clear_invisible_flag` has no caller.** Its only reference in the whole
  image is the vtable slot. Either the game never re-arms a failed sound within a session,
  or it is reached through a slot-11 dispatch nothing static can see. Worth an in-game
  check across a mission reload.
* **`S_sound_params.range`.** The shipped shape of the four floats is in §1b; what each one sets is inside `ls3df.dll`,
  outside this binary (§7 has what the PS2 setter keeps).
