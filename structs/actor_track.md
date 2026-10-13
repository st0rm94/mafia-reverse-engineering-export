# `C_actor_track` — the base class of `C_actor`, and the `C_recorder` client interface

**Created** 2026-08-04 18:07 +0200 · **Modified** 2026-10-12 21:02 +0200

PC prefix **`actor_track_`**.

`C_actor_track` is the 13-virtual base that every `C_actor`-family class inherits. Its
entire purpose is to give one scene object a **keyframe track** in the `C_recorder`
system (`structs/recorder.md`): record the object's pose over time, and play it back by
driving the object's `I3D_frame` directly.

> **The class name is a project-assigned name, not the developers'.** It is unrecoverable from
> either binary: the PC ships `/GR-` with zero RTTI, and the PS2 build does not have the class
> at all (§5). `C_recorder` *is* the developers' name — string-proven — but its client base is
> nameless in both binaries.

## 1. Why it is a base class and not part of `C_actor`

Read directly from the vtables in `Game.exe`'s `.rdata`:

* The **same 13 functions occupy slots 0–12 of 28–30 vtables** — every vtable in the
  `C_actor` family, i.e. exactly the set identified by the slot-16 scalar-deleting-destructor
  fingerprint.
* `C_actor`'s **own** virtuals begin at slot 13. **PC slot = PS2 vtable index + 13**,
  anchored by nine confirmed twins:

| PC slot | PC function | PS2 index | PS2 method |
|---|---|---|---|
| 17 | `actor_release` | 4 | `Release` |
| 18 | `actor_init` | 5 | `Init` |
| 22 | `actor_save_game_get_size` | 9 | `SaveGameGetSize` |
| 23 | `actor_save_game_save` | 10 | `SaveGameSave` |
| 24 | `actor_save_game_load` | 11 | `SaveGameLoad` |
| 25 | `actor_game_init` | 12 | `GameInit` |
| 26 | `actor_game_done` | 13 | `GameDone` |
| 28 | `actor_get_actor_speed` | 15 | `GetActorSpeed` |
| 34 | `actor_get_bound_points_2d` | 21 | `GetBoundPoints2D` |

* The interface is **slots 0–12**. Slots 13/14/15 in `C_actor`'s vtable are its *own* `AI` /
  `Update` / `NetDirect`, all folded by `/OPT:ICF` onto the single stub
  `stub_noop_args4_004776f0`.

**What is measured:** the 13 form one coherent block; they are emitted as **COMDATs in a
shared band** (`0x0044a650`–`0x0044b5c0`) rather than inside any one translation unit —
`C_actor_shot`'s two overrides sit in that band while its own TU is 0x4000 bytes away — and they
are a self-contained interface to one subsystem. Because the band is COMDAT-grouped, its emission order carries no information.

**Whether the base is a separate class is not decidable from the binaries.** No class outside the
`C_actor` family uses the interface. No stand-alone 13-slot vtable exists in `.rdata` — all 28
vtables holding the slot-0 default run 39 slots or more — and `actor_ctor` and `actor_dtor` each
store a single vptr, so a separate base could only have been declared `__declspec(novtable)`. The
PC ships no RTTI and the PS2 has neither the class nor the 13 methods (§5). A `novtable` base and
13 leading virtuals of `C_actor` produce the same bytes.

## 2. The vtable slots

| Slot | `C_actor` default | What it does |
|---|---|---|
| 0 | `stub_retminus1_args4_0044a780` | record a key now if one is due; **−1 = nothing recorded**. ICF-folded across 28 vtables |
| 1 | `actor_track_apply_key_texture_anim` | apply the key's texture-animation payload (`flags & 0x7ff` = id, bit `0x2000` = loop) |
| 2 | `actor_track_apply_pose_at_clock` | apply the interpolated pose at `g_recorder_playback_clock_ms`, using the cached cursor pair |
| 3 | `actor_track_apply_pose_at_time` | same at an explicit time, searching for the bracketing key; gated on `kind == 1` |
| 4 | `actor_track_get_pos_at_time` | read-only position sample → out-vec3 |
| 5 | `actor_track_get_rot_at_time` | read-only rotation sample → out-quat |
| 6 | `actor_track_set_recording` | bit `0x02` active / `0x08` requested |
| 7 | `actor_track_set_playback` | bit `0x01` active / `0x04` requested |
| 8 | `actor_track_apply_mode_changes` | promote `0x08→0x02` and `0x04→0x01` |
| 9 | `actor_track_create` | ask `C_recorder` for a track, declaring the per-key-type sizes |
| 10 | `actor_track_key_stream_size` | total bytes of the key stream |
| 11 | `actor_track_key_offset_of` | byte offset of a cursor inside another track's stream |
| 12 | a bare `ret` | debug dump; `C_human` overrides it with `human_track_dump_keys` |

### 2a. Known overrides

| Class | Slot | PC function |
|---|---|---|
| `C_actor_shot` | 1 | `actor_shot_track_apply_key_effects` |
| `C_actor_shot` | 4 | `stub_zero_vec3_out` — ICF fold (§2b) |
| `C_actor_shot` | 5 | `stub_zero_quat_out` — ICF fold (§2b) |
| `C_actor_shot` | 9 | `actor_shot_track_create` |
| `C_human` | 0 | `human_track_record_key` ¹ |
| `C_human` | 1 | `human_track_apply_key_anim` ¹ |
| `C_human` | 2 | `human_track_apply_pose_at_clock` ¹ |
| `C_human` | 3 | `human_track_apply_pose_at_time` ¹ |
| `C_human` | 4 | `human_track_get_pos_at_time` ¹ |
| `C_human` | 5 | `human_track_get_rot_at_time` ¹ |
| `C_human` | 9 | `human_track_create` ¹ |
| `C_human` | 12 | `human_track_dump_keys` ¹ |
| `C_car` | 0 | `car_track_record_key` ² |
| `C_car` | 1 | `car_track_spawn_dropout_part` ² |
| `C_car` | 2 | `car_track_apply_pose_at_clock` ² |
| `C_car` | 3 | `car_track_apply_pose_at_time` ² |
| `C_car` | 4 | `car_track_get_pos_at_time` ² |
| `C_car` | 9 | `car_track_create` ² |
| `C_box` | 0 | `box_track_record_key` — 45° basis rotation **or** 100 ms; returns the `-1` sentinel |
| `C_box` | 2 | `track_apply_rigid_pose_at_clock` — ICF fold (§2b) |
| `C_box` | 4 | `stub_zero_vec3_out` — ICF fold (§2b) |
| `C_box` | 5 | `stub_zero_quat_out` — ICF fold (§2b) |
| `C_box` | 9 | `box_track_create` — `recorder_add_track(0x28, 0x28, 0x30)` |
| `C_DropOut` | 0 | `dropout_track_record_key` — `{time, type 1, pos, quat}`; 200 ms suppression while act-state 1; returns the `-1` sentinel |
| `C_DropOut` | 2 | `track_apply_rigid_pose_at_clock` — ICF fold (§2b) |
| `C_DropOut` | 9 | `dropout_track_create` — `recorder_add_track(0x24, 0x24, 0x24)` |
| `C_airplane` | 13 | `airplane_ai_tick` — the class's own AI, not an override of this interface |
| `C_airplane` | 14 | `airplane_update` — same; slots 13–15 are `C_actor`'s own |

¹ **Declared by `C_human`, inherited by `C_entity` and `C_player`.** A scan of `.rdata` for each
of the eight addresses as a pointer returns the same three vtables every time: `0x0063b488`
`C_entity`, `0x0063c468` `C_human`, `0x0063c908` `C_player`. The hierarchy is `C_actor` ←
`C_human` ← {`C_entity`, `C_player`}; a function declared by `C_entity` could not appear in its
base `C_human`'s vtable. The TU placement agrees: all eight sit inside `C_human.cpp`, between the
CRT pair at `0x0048b5b0` and `human_ctor` at `0x0048f430`. A ninth member is not a vtable slot:
`human_track_queue_anim_event` (`0x00499380`, 0 `.rdata` datarefs) has exactly one caller,
`human_track_apply_pose_at_time`.

² See §2c.

### 2b. ICF folds — a slot gives the role, only the dataref count gives the owner

The rigid-body classes have no animation, so their track overrides reduce to the same pos+quat
code and the linker's identical-COMDAT folding merges them. Each of these sits inside one
class's TU and fills that class's slot, yet belongs to no single class:

| function | VA | slot | vtables holding it (scan of `.rdata` for the address) |
|---|---|---|---|
| `stub_zero_vec3_out` | `0x0045f0f0` | 4 | `0x0063b720` `C_actor_shot` · `0x0063b980` `C_box` |
| `stub_zero_quat_out` | `0x0045f110` | 5 | `0x0063b720` `C_actor_shot` · `0x0063b980` `C_box` |
| `track_apply_rigid_pose_at_clock` | `0x0048aaf0` | 2 | `0x0063c320` `C_DropOut` · `0x0063b980` `C_box` |
| `bottle_subclass_init_require_frame` | `0x0048af90` | 18 | `C_cartridge` · `C_grenade` · `C_smoke_trail` |

* `stub_zero_vec3_out` / `stub_zero_quat_out` are two classes' identical "no positional /
  rotational playback" overrides; they sit inside the `C_box` TU.
* `track_apply_rigid_pose_at_clock` sits inside the `C_DropOut` TU. Its two sibling overrides in
  the same TU have **one dataref each** and keep the `dropout_` prefix, so the census has to be
  run per function, not per translation unit.
* `bottle_subclass_init_require_frame` (slot 18, `Init`, outside this interface) sits inside the
  `C_grenade` TU and calls `bottle_init`; it is named for the family.

Expect a fold whenever two `C_PhThing`-family classes fill the same slot.

### 2c. Slot 21 = `DuplicateFrom`, and the `C_car` overrides

**Slot 21** is not part of the 0–12 interface; it is recorded here because this is where the
family slot map lives. `box_new_duplicate_from` pins it: a **96-byte** body copying six fields in
order after a base call (its PS2 twin: §5). Every class that does not override it holds a
folded no-op stub there: `C_bottle` → `thunk_stub_noop_args4_00461180` (`0x004773f0`, a 5-byte
JMP thunk; Ghidra names a thunk after its target), `C_box` → `stub_noop_args4_004776f0`.

**The six `C_car` overrides** (§2a) carry `car_track_`, not `vehicle_`: the receiver is
byte-proven `5C_car`. Every one reads the track from `this+0x08` and the I3D frame from
`this+0x68`, both `C_actor` **base** fields that `C_Vehicle` — which lives at `car+0x80` — does
not have. Vtable `0x0063bc08` also carries `car_ai_tick`, `car_update`, `car_game_init` and
`car_get_actor_speed` (their PS2 twins: §5), and a vtable belongs to exactly one class.

`car_track_spawn_dropout_part` (slot 1) replays an "a part came off" key by spawning a
**`C_DropOut`** actor (kind `0x11`) bound to the named car part. That kind is labelled data:
`recorder_new_track` formats track names with `case 0x11: "drp_%d"` beside
`act_`/`hum_`/`car_`/`box_`. The function's two branches — wheel via `car+0xd24`, body panel via
the `0x34`-stride table at `car+0x308` — match `C_DropOut`'s two initialisers (§5). See
`structs/car.md` and `structs/dropout.md`.

## 3. Field layout

| Offset | Field | Evidence |
|---|---|---|
| `+0x00` | vtable pointer | `actor_ctor` writes `&g_actor_vt` |
| `+0x04` | **mode/state flags** — see §3a | zeroed by `actor_ctor` |
| `+0x08` | `track*` — the `C_recorder` track | written by `recorder_add_track`; every slot reads it |
| `+0x10` | `kind` (the C_entity type enum) | forwarded to `recorder_new_track`, which switches on it |
| `+0x60` | the pending event record (`0x48` bytes) | written by `human_track_queue_anim_event`, encoded into the next key and cleared by `human_track_record_key` (`structs/actor.md` §1b) |
| `+0x68` | `I3D_frame*` — the bound scene frame | slots 1/2/3 write its local pos/quat |

`actor_ctor` calls no base constructor: the base has none to call — it is a pure interface —
and `actor_ctor` writes the derived vptr at `+0x00`.

### 3a. The `+0x04` flag word

| Bit | Meaning | Evidence |
|---|---|---|
| `0x01` | **playback active** | `human_update` runs `human_animation_evaluate_and_dispatch` **only while it is clear** — the bit stands the normal animation driver down so the track can own the pose. Tested together with `0x04` (mask `5`) by `actor_tick`, `car_update` (four sites), `car_on_act_state_changed`, `human_update` and `entity_suspend_ai_and_hide` |
| `0x02` | **recording active** | `human_track_queue_anim_event` is gated on it, and the record it produces has exactly one consumer: `human_track_record_key`. `dropout_on_act_state_changed` returns early while it is set, so a `C_DropOut` going inactive is not recycled mid-recording, and `dropout_game_done` tests it with `0x01` (mask `3`) |
| `0x04` | playback **requested** | promoted to `0x01` by `actor_track_apply_mode_changes` |
| `0x08` | recording **requested** | promoted to `0x02` by the same |
| `0x10` | **has a track** | set by `recorder_add_track`; read by `human_update` (skip the frame's own tick) and `game_init` (find the player actor that has no track yet) |

**Enabling is deferred, disabling is immediate.** `set(true)` raises only the *request* bit
and only when the mode is not already active; `set(false)` clears the active and request
bits together. So a mode change always lands at a controlled point.

**Where the meanings come from.** The three setters have **0 direct callsites and 35 vtable
datarefs each**, so no caller passes a literal and the meaning comes from the readers.
`0x02 = recording` rests on a closed writer→reader data chain plus the `C_DropOut` readers;
`0x01 = playback` on the suppression readers above, which always test it together with its own
*requested* bit `0x04` and never with `0x02` or `0x08`. The readers were found by disassembling
every function for bit tests on displacement `+0x04` and keeping those whose receiver is the actor.

## 4. The key record

Field names below are **the game's own**, taken from `human_track_dump_keys`'s format
strings (`lastKeyTime = %d`, `key %d: %d, animid = %d animflags = %d`).

| Offset | Field |
|---|---|
| `key[0]` | time (ms) |
| `key[1]` | **type** — indexes the track's size table at `track+0x44` |
| `key[2..4]` | position |
| `key[5..8]` | rotation quaternion |
| `key[9]` | `animid` (`& 0x7ff`) \| `animflags` (`& 0xf800`) |
| `key[0x0b..0x0d]` | an extra vec3, key types `0x0a`/`0x0b` only |

Records are **variable-size**: each class declares its own per-type sizes through its slot-9
override, and every walk takes the stride from `track+0x44 + key[1]*4`.

| Class | Key-record sizes |
|---|---|
| `C_actor` | `0x5c` |
| `C_actor_shot` | `0x10` |
| `C_human` (and `C_entity`, `C_player`) | `0x28`, `0x2c`, `0x38` |
| `C_car` | `0x2c`, `0x38`, `0x38` |
| `C_box` | `0x28`, `0x28`, `0x30` |
| `C_DropOut` | `0x24`, `0x24`, `0x24` — one shape for all three key types |

Keys are bump-allocated into chunks (`recorder_key_stream_alloc`) linked by an **`0xEEEEEEEE`**
sentinel; entry 0 of the size table is the 8-byte chunk-link record.

**The captured pose.** When `actor_track_apply_pose_at_time` plays back a key whose word
`key[0x16]` (the last dword of `C_actor`'s `0x5c` record) has bit `0x4000`, it fills
`g_actor_track_captured_pose` (`0x00656464`, 0x20 bytes): `valid` is set to 1 by a byte store,
`pos` takes the frame's world position and `rot0`..`rot3` its rotation quaternion. Nothing in the
image reads it back: every dword in every section whose value falls in
`0x00656464..0x00656483` was enumerated, and the only ones are these eight stores and four bytes
of the text `"freeride"` in `.data` (`0x006512a9`). The capture is write-only in this build; what
it was meant for is not recoverable from the binary.

## 5. PC vs PS2

**The entire subsystem is PC-only.** Not one of `lastKeyTime`, `animflags`, `animid`, `.dump`,
or any `C_recorder:` string exists in `SLUS_206.71`, and no PS2 symbol carries the class —
`C_actor_track` occurs 0 times in the PS2 symbol table and decomp (so there is no
`Create__13C_actor_track` or `RecordKey__13C_actor_track`). `__vt__7C_actor` on PS2 holds
**exactly** the 27 methods that map to PC slots 13–38 and nothing before them. So every function
in this file has **no PS2 twin, by measurement rather than by search failure** — the inverse of
the usual case, where the PS2 port *added* subsystems.

* **The PS2 vptr placement.** PS2 `__ct__7C_actorFv` stores its vtable at `this+0x78`: GCC 2.x
  places the vptr **after** the data for a class with no polymorphic base, and PS2 `C_actor`
  has none — `__dt__7C_actorFv` restores exactly one vptr and calls no base destructor.
* **Field layout:** the base fields this interface uses — `+0x04` mode flags and `+0x08` track —
  have no PS2 counterpart, and PS2's `C_actor` data starts at `+0x00` with its cached vectors.
  The full pair table is `structs/actor.md` §6.
* **`.tck` track files ship on both builds** (`Ddeath.tck`, `DturnL.tck`, `DturnR.tck`,
  `Nuda01.tck`, `Nuda02.tck`, `turn1.tck`) and the PS2 keeps consumers for them
  (`LoadTCK__5C_dogFv`, `GetPlacementFromTCK__5C_car`,
  `GetPlacementFromTCK__28C_animations_machine_de_luxe`) — the port kept the *data* and the
  readers, and cut the recorder that produced it.
* **`C_DropOut`'s two initialisers** on the PS2 are `InitWheel` and `InitBox` — the same
  two-branch split `car_track_spawn_dropout_part` makes (§2c).
* **Slot 21 = `DuplicateFrom`** (§2c) is pinned by the CONFIRMED pair `box_new_duplicate_from` ↔
  `DuplicateFrom__9C_box_newFP7C_actor`: **96 bytes on both builds**, copying the same six fields
  in the same order after a base call.
* **The `C_car` vtable `0x0063bc08`** (§2c) holds `car_ai_tick` ↔ `AI__5C_carFUi`, `car_update` ↔
  `Update__5C_carFUi`, `car_game_init` ↔ `GameInit__5C_carFv` and `car_get_actor_speed` ↔
  `GetActorSpeed__5C_carFR8S_vector`.

## 6. Related

`structs/recorder.md` (the manager) · `structs/actor.md` (the derived root) ·
`structs/actor_shot.md` · `structs/entity.md` · `structs/human.md` · `structs/car.md` ·
`structs/dropout.md` · `structs/box.md`.

## 7. Open items

Nothing is open.
