# `C_recorder` — the keyframe track recorder and `.tck` loader

**Created** 2026-08-04 18:07 +0200 · **Modified** 2026-09-27 15:52 +0200

| | |
|---|---|
| PC prefix | **`recorder_`** |
| Singleton | **`0x00656368`** (§1) |
| PC vtable | **`0x0063d0e8`, 53 slots** |
| PC translation unit | `[0x005407f0, 0x005469a0)` — every start named |
| Embedded sub-objects | the **name↔id table at `+0x80`** (own 6-slot serialisation vtable `0x0063d0d0`) and the **track↔object binding table at `+0x90`** — both byte-proven from the forwarder band |
| PS2 class | none — PC-only by measurement (§6) |

`C_recorder` owns every scene object's keyframe **track**: it creates them, names them,
keeps the object↔frame bindings needed to re-attach a track after a scene reload, and loads
`.tck` track files from disk. Its per-object client interface is `C_actor_track`
(`structs/actor_track.md`), which every `C_actor`-family class inherits.

The class name is the developers' own — it is string-proven (§2).

---

## 1. The singleton

**`0x00656368`**, proven two ways:

* `recorder_singleton_construct` is literally `mov ecx, 0x656368 ; jmp recorder_ctor`, and
  `recorder_singleton_destruct` is `mov ecx, 0x656368 ; jmp recorder_dtor`.
* A whole-`.text` disassembly scan finds **59 `mov ecx, 0x656368` sites** — a singleton loaded as an
  immediate gives the receiver *and* the full method census in one scan. The heaviest consumers are
  `engine_game_loop` (8), `program_process_command` (7), `game_done`, `mission_close` and
  `actor_dtor`; the rest are the `C_actor_track` slot implementations and their per-class
  overrides.

**The singleton has no symbol in the decomp.** An immediate loaded straight into ECX produces no
identifier in Ghidra's output: the callsite renders as `recorder_add_track(obj, 0x5c, 0, 0)` with
the receiver invisible, and `DAT_00656368` occurs **0** times in the annotated decomp. The address
is recorded here instead.

The object is polymorphic (`recorder_add_track` dispatches through `[this]`, slots 5 and 12),
so a derived recorder variant may exist (§7).

---

## 2. What names the class

`recorder_load_tck` emits all of these through `dbgPrintf`:

```
C_recorder:LoadTCK(%s)          C_recorder:tck opened...
C_recorder:version = 4          C_recorder:wrong version
C_recorder:LoadTCK() done ok    C_recorder:LoadTCK() done failed
```

Shipping `.tck` files include `Ddeath.tck`, `DturnL.tck`, `DturnR.tck`, `Nuda01.tck`,
`Nuda02.tck`, `turn1.tck`. The format is **version 4**; anything else is rejected.

---

## 3. Method coverage

Ordered by vtable slot; all 53 slots are placed. Slot indices are relative to the real vtable at
`0x0063d0e8`. A plain run-of-code-pointers scan of `.rdata` merges it with the adjacent six-slot
`0x0063d0d0` table (no separator between them), which reports every slot 6 too high.

| slot | PC name | Role |
|---:|---|---|
| 0 | `recorder_set_mode` | 0 stop / 1 and 2 the record-playback pair / 4 idle |
| 1 | `recorder_get_mode` | `+0xe4` |
| 2 | `recorder_sync_clock_from_system` | set the source clock from a parameter + prune finished tracks (§3c) |
| 3 | `recorder_get_source_clock_ms` | `+0x11c` |
| 4 | `recorder_get_playback_clock_ms` | `+0x120` — what every `*_track_apply_*` compares against |
| 5 | `recorder_new_track` | allocate a `0x5c`-byte track and name it by kind |
| 6 | `recorder_track_reset_fields` | re-init a track record |
| 7 | `recorder_attach_existing_track` | register a track the caller built |
| 8 | `recorder_add_track` | the target of every `*_track_create` override |
| 9 | `recorder_remove_track` | sole external caller `actor_dtor` |
| 10 | `recorder_release_all_key_streams` | free every track's key data |
| 11 | `recorder_snapshot_track_list` | copy both vectors into the file-scope snapshot |
| 12 | `recorder_has_track` | - |
| 13 | `recorder_set_track_active` | move a track between the active and idle vectors |
| 14 | `recorder_has_track_in_either_list` | - |
| 15 | `recorder_get_or_create_name_id` | builds the object on a miss |
| 16 | `recorder_load_anim_by_name` | `anims\%s` |
| 17 | `recorder_get_object_by_name` | - |
| 18 | `recorder_get_object_by_id` | **10 callsites, all `*_track_apply_*`** |
| 19, 35 | `recorder_get_name_for_id` | one function in two slots — ICF fold or alias |
| 20 | `recorder_load_tck` | the `.tck` loader, version 4 |
| 21 | `recorder_load_anim_for_object` | - |
| 22 | `recorder_rebuild_object_table` | - |
| 23 | `recorder_rewind_timeline_cursor` | `+0x70` |
| 24 | `recorder_track_rewind_cursors` | - |
| 25 | `recorder_load_replay_file` | **the `.rep` loader** |
| 26 | `recorder_save_replay_file` | **the `.rep` writer**, magic `0x1631` |
| 27 / 28 | `recorder_set_timeline_enabled` / `_get_` | `+0x129` |
| 29-33, 38 | `recorder_clear_binding_values` · `_add_binding_value` · `_load_bindings_file` · `_detach_object_binding` · `_attach_object_binding` · `_compact_bindings` | forwarders — `add ecx, 0x90` |
| 34 | `recorder_get_or_create_name_id_direct` | `lea esi,[ecx+0x80]` |
| 36 / 37 | `recorder_get_name_value` / `_set_name_value` | forwarders — `add ecx, 0x80` |
| 39 | `recorder_end_sequence_and_restore_scene` | restores the scene from the actor snapshot |
| 40-42 | `recorder_binding_block_size` / `_write` / `_read` | the `.rep` binding section |
| 43 / 44 | `recorder_bind_object_by_name` / `_unbind_` | - |
| 45 | `recorder_open_speech_sound_for_text_id` | bridge into `structs/speech.md` |
| 46 | `recorder_get_constant_5` | `return 5;` — dead; the meaning of 5 is undecoded |
| 47 | `recorder_set_overlay_texture` | the cutscene letterbox |
| 48 | `recorder_rebind_speaker_channel` | bridge into `structs/speech.md` |
| 49 | `recorder_tick_speech` | bridge into `structs/speech.md` |
| 50-52 | `recorder_open_synced_stream` / `_set_stream_time_base` / `_sync_stream_to_clock` | the streamed audio track |

Non-virtual members: `recorder_ctor`, `recorder_dtor`, the static-init/atexit wrappers
(`recorder_singleton_static_init`, `recorder_singleton_construct`,
`recorder_singleton_register_atexit`, `recorder_singleton_destruct`), `recorder_tick_tracks`,
`recorder_advance_clock`, `recorder_is_sequence_running`, `recorder_stop`,
`recorder_track_step_keys_to_clock`, `recorder_track_seek_to_clock`,
`recorder_track_stop_and_rewind`, `recorder_release_object_table`, the three
`recorder_replay_state_*` helpers and the two dead interpolators `recorder_interp_key_pos_and_pair`
/ `recorder_interp_key_pos`.

### 3a. The two embedded sub-objects

**`recorder+0x04` / `+0x14` — the idle and active client vectors.** Two `std::vector`s of owner
objects (each entry's `+0x08` is its track, `+0x10` its kind). `recorder_set_track_active` moves a
client between them: mode 1 takes it out of the idle vector at `+0x04` and appends it to the active
vector at `+0x14`, clearing the client's flag `0x10`; mode 0 does the reverse and sets the flag.
`recorder_has_track_in_either_list` scans both. `recorder_bindings_compact_and_rebind` is the only
function that reaches them by absolute address (§5); everywhere else they are `this`-relative.

**`recorder+0x80` — the name↔id table.** Rows are `{int32 id; char name[0x30]; int32
value}` = `0x38` bytes; ids start at `0x100`. Its own vtable `0x0063d0d0` is a **six-slot
serialisation interface**: `read_dta` / `read_stdio` / `copy_from_block` / `write_stdio` /
`copy_to_block` / `serialized_size`. Prefix `recorder_names_`.

**`recorder+0x90` — the track↔object binding table.** Rows are `0x58` bytes (§4c); a second
`0x28`-stride list at `+0x18` holds `{name, value}` pairs the script can set. Prefix
`recorder_bindings_`. `recorder_add_track_binding` is a member of this sub-object, not of
`C_recorder` — its name is right and its prefix is one level off.

### 3b. Owner proof for the four one-line accessors

Identical-COMDAT folding is active in this link, so a one-line accessor needs its own owner proof.
All four keep the `recorder_` prefix:

| accessor | body | what proves the owner |
|---|---|---|
| `recorder_get_mode` | `mov eax,[ecx+0xe4]` | **0 callsites, 1 dataref** = `C_recorder` vtable `0x0063d0e8` slot 1 |
| `recorder_get_source_clock_ms` | `mov eax,[ecx+0x11c]` | 0 callsites, 1 dataref = same vtable, slot 3 |
| `recorder_get_playback_clock_ms` | `mov eax,[ecx+0x120]` | 0 callsites, 1 dataref = same vtable, slot 4 |
| `recorder_names_get_count` | `mov eax,[ecx+4]` | 2 callsites, receiver `lea ecx,[ebx+0x80]` = the `+0x80` sub-object |

**A dataref count of exactly 1 is a single-owner proof.** The function appears in one class's
vtable and nowhere else, so nothing can have been folded into it — and it is the only proof
available for the first three, which nothing calls directly.

**Two of these bodies are byte-identical to accessors of other classes and are still not folds:**
`recorder_get_mode` has the same three bytes as `game_get_player_actor` (`0x00425530`), and
`recorder_names_get_count` the same as `std_vector_begin` (`0x005a17c0`). Two addresses, two owners
— surviving duplicates from the non-COMDAT population that could not fold.

**`recorder_names_get_count` really is a count, despite reading `[ecx+4]`** — the offset a
`std::vector` uses for its *begin* pointer. `recorder_rebuild_object_table` does `test eax,eax; jle`
and `cmp ebp,eax; jl`, using the result as a signed loop bound.

### 3c. `recorder_sync_clock_from_system` does not read the system clock

`recorder_sync_clock_from_system` (`0x00540fc0`, vtable `0x0063d0e8` slot 2) opens with
`folded_return_this()`, whose body is `mov eax, ecx; ret`. Its ECX at that callsite is
`mov ecx, [esp+0x14]`, and the frame arithmetic settles what that is:

```
00540fc0  sub esp, 8          ; locals
00540fc3  push ebp            ; -> retaddr at [esp+0xc], arg1 at [esp+0x10]
00540fc4  push esi            ; -> retaddr at [esp+0x10], arg1 at [esp+0x14]
00540fc5  mov esi, ecx        ; esi = this (C_recorder)
00540fc7  mov ecx, [esp+0x14] ; <- the first STACK ARGUMENT
00540fcc  call folded_return_this
00540fd7  mov [esi+0x11c], eax
```

So the clock value is an **incoming parameter Ghidra dropped from the signature**
(`void __fastcall f(int param_1)` renders one parameter where the frame carries two). Nothing
reads a system timer.

The sibling settles what the function is: `recorder_advance_clock` (`0x00543540`) does
`+0x11c += *delta`; this one does `+0x11c = param`. Both then run the identical remap of `+0x11c`
through the `+0x68` keyframe table into `+0x120`/`+0x124` and stop every track whose slot-3 test
fails at that time. The pair differs only by **absolute versus delta**, and
`recorder_get_source_clock_ms` returns exactly the `+0x11c` this writes. The function name does
not reflect this (§7).

### 3d. Binding-table teardown: `recorder_bindings_clear_all` tail-calls the arena release

| function | body | what it does |
|---|---|---|
| `recorder_bindings_clear_all` (`0x005457b0`) | `[0x005457b0, 0x00545838)`, no `ret` | walks the `0x58`-stride rows: a row with mode flag 0 has its key stream cleared, and a bound scene object is unlinked; then empties the row vector and ends in `jmp 0x00545840` |
| `recorder_bindings_release_key_arena` (`0x00545840`) | `[0x00545840, 0x005458a6)`, one `ret` | when the key arena (`table+0x00`) is set, counts the rows whose mode flag (`row+0x00`, §4c) is `1`; with none, frees the arena and clears the pointer |

Both take the table in ECX. The closing `jmp` is the release's only reference, so it has no
direct caller of its own.

---

## 4. The track object (`0x5c` bytes)

Allocated by `recorder_new_track`.

| Offset | Field | Evidence |
|---|---|---|
| `+0x00` | current key cursor | read by `actor_track_apply_pose_at_clock` as `*(this+8)[0]` |
| `+0x04` | next key cursor | the pair the clock is bracketed against |
| `+0x08` | last type-`&2` key | maintained by the playback stepper |
| `+0x10` | a fourth cursor | reset with the others by `recorder_track_rewind_cursors` |
| `+0x14` | **`lastKeyTime`** | the game's own name, from the `human_track_dump_keys` output; both `recorder_new_track` and `recorder_track_reset_fields` seed it with the `-666` sentinel (`0xfffffd66`) |
| `+0x18` | first key chunk | every stream walk starts here; `recorder_key_stream_clear` nulls it |
| `+0x20` | track name | `act_%d` / `hum_%d` / … ; copied into the binding record |
| `+0x44` | **key-size table**, 4 entries | `{8, size1, size2, size3}`; indexed by `key[1]` |
| `+0x54` | cleared on rewind | purpose untraced |
| `+0x58` | owner back-pointer | written by `recorder_add_track` |

### 4a. Track naming, and what it proves

`recorder_new_track` switches on the owner's **kind** and post-increments one counter per
kind:

| Kind | Prefix | Counter global |
|---|---|---|
| 1 (actor) | `act_%d` | `g_recorder_track_seq_actor` |
| 2 / 3 / 0x1b (player, human, entity) | `hum_%d` | `g_recorder_track_seq_human` |
| 4 (car) | `car_%d` | `g_recorder_track_seq_car` |
| 9 (box) | `box_%d` | `g_recorder_track_seq_box` |
| 0x11 (drop-out) | `drp_%d` | `g_recorder_track_seq_dropout` |
| anything else | `unk_%d` | `g_recorder_track_seq_unknown` |

These are exactly the documented `C_entity` type enum values, which independently confirms
that the value `recorder_add_track` forwards is the object's `+0x10` kind field. The counters
are **sequence numbers**, not populations — nothing decrements them.

### 4b. The key stream is a chain of chunks, with no separate allocator object

`recorder_key_stream_alloc` and `recorder_key_stream_new_chunk` are `__stdcall` FREE functions —
`ret 8` and `ret 4`, so the receiver is PUSHED, not in ECX — and the object they are handed is the
TRACK documented in §4. Byte-proven at the call site:

```
car_track_record_key:   mov  eax, [esi + 8]     ; esi = this (C_car); [this+8] = the track
                        push 2                  ; the key type
                        push eax                ; the track
                        call recorder_key_stream_alloc
```

All **eleven** `recorder_key_stream_alloc` call sites are the four `*_track_record_key`
overrides (`human_` x7, `car_` x2, `box_` x1, `dropout_` x1), and every one passes
`actor+0x08`. That is the same object `recorder_key_stream_clear` and
`recorder_key_stream_size` receive — their call sites load `mov ecx, [row + 0x54]`, the
track field of the binding record (§4c).

**Chunk layout** (0x1800 bytes from `operator_new`):

| Offset | Field | Evidence |
|---|---|---|
| `chunk+0x00` | next-chunk link, 0 on the last | `recorder_key_stream_new_chunk` zeroes it; the successor is written through the predecessor's pointer |
| `chunk+0x04` | soft limit = `chunk + 0xFF0` | `lea ecx,[eax + 0xff0]` ¹ |
| `chunk+0x08` | first record | the cursor is seeded here on both the first-chunk and the chain paths |

¹ So ~4 KB of the 6 KB allocation is usable and the ~2 KB tail is slack (§7).

**Why only two functions.** Allocation needs exactly bump + grow;
`recorder_key_stream_new_chunk` has two call sites and both are inside its parent. The rest
of the interface lives on the track: `recorder_key_stream_clear` (free),
`recorder_key_stream_size` / `actor_track_key_stream_size` (walk),
`actor_track_key_offset_of` (index) and vtable slot 10
`recorder_release_all_key_streams` (bulk free).

**The alloc advances past the record already under the cursor, not the one being asked for.** It
reads that record's own type out of `record+0x04`, looks the stride up in the track's key-size
table at `track + 0x44 + type*4`, and only then checks whether the NEW type fits. On overflow it
stamps **`0xEEEEEEEE`** at the old cursor — the continuation sentinel `recorder_key_stream_size`
and `actor_track_key_offset_of` walk, and the reason size-table entry 0 is the 8-byte chunk-link
record (`structs/actor_track.md` §4).

### 4c. The binding record (`0x58` bytes)

Written by `recorder_bindings_bind_track_by_name`, `recorder_bindings_attach_object` and
`recorder_bindings_detach_object`.

| Offset | Field | Evidence |
|---|---|---|
| `+0x00` | mode flag | written from the caller's argument; when it is 0 and the track changes, the upsert first discards the recorded keys through `recorder_key_stream_clear`; a row with flag 1 keeps the table's key arena alive (§3d) |
| `+0x04` | owner **kind** | `recorder_bindings_attach_object` adopts a row only when this equals the actor's `+0x10` |
| `+0x08` | `char[0x24]` frame/model name | seeded to the literal `"<NONE>"` (`s_none_006477a0`), overwritten from `*(*(object+0x68)+0x100)`; `_detach_object` restores `"<NONE>"` |
| `+0x2c` | `char[0x24]` the binding NAME | the lookup key; copied from `track+0x20`, the track name §4a builds |
| `+0x50` | the scene object | cleared by `_detach_object` |
| `+0x54` | the track | also written into `object+0x08`, which is what hooks the track onto the actor |

---

## 5. Globals

| VA | Name | Meaning |
|---|---|---|
| `0x00656368` | - (no symbol, §1) | the `C_recorder` singleton |
| `0x00656370`…`0x00656384` | `g_recorder_idle_clients_begin` / `_end`, `g_recorder_active_clients_begin` / `_end` | the singleton's own client vectors (§3a) at `+0x08/+0x0c` and `+0x18/+0x1c`, named because `recorder_bindings_compact_and_rebind` walks them by absolute address to re-bind every client whose track holds recorded keys |
| `0x00656488` | `g_recorder_playback_clock_ms` | the global playback clock every pose-apply compares key times against |
| `0x00656494` | `g_zero_vector_00656494` | a file-scope zero `S_vector`, zeroed by `recorder_static_zero_globals`; no reader and no registered destructor |
| `0x006564c4`…`0x006564d8` | `g_recorder_track_seq_*` | the six per-kind name counters (§4a) |

`g_recorder_playback_clock_ms` has no initial bytes in `Game.exe`. `.data` is `VA 0x00646000,
virtual size 0xb4e7c, raw size 0x10000`, so the section's file bytes stop at VA `0x00656000`;
`0x00656488` lies in the **BSS tail**, zero-initialised at load. File offset `0x256488` belongs to
the next section's raw data (`.rsrc`, raw `0x256000`), so bytes read there say nothing about this
global.

---

## 6. PC vs PS2

**`C_recorder` does not exist on PS2.** No symbol carries the class, and none of
`lastKeyTime`, `animflags`, `animid`, `.dump` or any `C_recorder:` string appears anywhere in
`SLUS_206.71`. `__vt__7C_actor` on PS2 holds only `C_actor`'s own 27 methods, with no client
interface before them.

The port kept the **data and the readers** and cut the **producer**: `.tck` files still ship,
and PS2 retains `LoadTCK__5C_dogFv`, `GetPlacementFromTCK__5C_car`,
`GetPlacementFromTCK1__5C_car`, `GetPlacementFromTCK__28C_animations_machine_de_luxe`,
`GetEndPointOfTCK__…` and `GetMixingPointOfTCK__…`.

⇒ Every function and global **of `C_recorder` itself** is PC-only by measurement, not by search
failure. The claim covers only those symbols, not the address range: the 16-channel speech bank
shares this TU's range and **does** exist on PS2, as free functions in `g_dabing.cpp`, three of
them byte-identical (`structs/speech.md` §1). Here the PC build has a subsystem the PS2 port
removed — the reverse of the usual direction, where the PS2 build adds subsystems.

---

## 7. Open items

| Item | state |
|---|---|
| the derived recorder variant | the singleton is polymorphic (slots 5 and 12 dispatched) but only one implementation has been located |
| `track+0x54` | cleared on rewind; no reader traced |
| the `.tck` on-disk layout | `recorder_load_tck` reads a version dword, a 4-byte field and two 12-byte blocks before the key data; the rest is unmapped |
| the chunk slack | why only `0xFF0` of the `0x1800`-byte chunk is used is unknown |
| `recorder_get_constant_5` | what the returned 5 means is undecoded |
| `recorder_sync_clock_from_system` | the name says "from system" but the body sets `+0x11c` from a parameter (§3c); the name is unchanged |