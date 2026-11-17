# `C_actor_track` — the base class of `C_actor`, and the `C_recorder` client interface

**Created** 2026-08-04 18:07 +0200 · **Modified** 2026-10-29 22:29 +0100

`C_actor_track` is the 13-virtual base that every `C_actor`-family class inherits. Its
entire purpose is to give one scene object a **keyframe track** in the `C_recorder`
system (`structs/recorder.md`): record the object's pose over time, and play it back by
driving the object's `I3D_frame` directly.

> **The class name is a project-assigned name, not the developers'.** It is unrecoverable from
> either binary: the PC ships `/GR-` with zero RTTI, and the PS2 build does not have the class
> at all (§7). `C_recorder` *is* the developers' name — string-proven — but its client base is
> nameless in both binaries.

| | |
|---|---|
| PC prefix | `actor_track_` |
| PC kind (`+0x10`) | - (the field is the derived class's kind; the interface forwards it, §2) |
| PC vtable | - (no vtable of its own: slots 0–12 of every `C_actor`-family vtable, §4) |
| PC base class | - |
| PC instance size | - (data-free; it uses fields of the `C_actor` object, §1) |
| PC constructor | - (none: `actor_ctor` writes the derived vptr and calls no base constructor) |
| PC translation unit | `[0x00548830, 0x00548ac0)`, shared with the `recorder_key_stream_*` helpers, plus the COMDAT band `[0x0044a650, 0x0044b5c0)` inside `C_actor.cpp` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The interface has no data of its own. These are the fields of the `C_actor` object
(`structs/actor.md` §1) that its methods read and write.

```c
typedef struct C_actor_track {              // view of a C_actor; PC offsets
    /* +0x00 */ void      *vtable;              // [H] the derived class's vptr
    /* +0x04 */ uint32_t   mode_flags;          // [H] playback / recording state (§2a)
    /* +0x08 */ void      *track;               // [H] the C_recorder track
    /* +0x10 */ int32_t    kind;                // [H] the derived class's kind, forwarded to the recorder
    /* +0x60 */ void      *pending_event;       // [H] 0x48-byte event record queued for the next key
    /* +0x68 */ I3D_frame *frame;               // [H] the bound scene frame
} C_actor_track;
```

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `actor_ctor` (writes `&g_actor_vt`) and every subclass ctor | virtual dispatch | there is no base vptr: `actor_ctor` writes the derived one at `+0x00` |
| +0x04 | mode_flags | H | `actor_ctor` (=0), `actor_track_set_recording`, `actor_track_set_playback`, `actor_track_apply_mode_changes`, `recorder_add_track` (bit `0x10`) | `actor_tick`, `car_update`, `car_on_act_state_changed`, `human_update`, `entity_suspend_ai_and_hide`, `human_track_queue_anim_event`, `dropout_on_act_state_changed`, `dropout_game_done`, `game_init` | a dword; the bit map is §2a |
| +0x08 | track | H | `recorder_add_track` | every slot of §4 | the `C_recorder` track |
| +0x10 | kind | H | the derived class's ctor (`structs/actor.md` §2) | `recorder_new_track` (switches on it), `actor_track_apply_pose_at_time` (gated on `kind == 1`) | the `C_entity` type enum |
| +0x60 | pending_event | H | `human_track_queue_anim_event` | `human_track_record_key` | the `0x48`-byte record is encoded into the next key and its byte 0 cleared (`structs/actor.md` §2) |
| +0x68 | frame | H | `actor_init` (`structs/actor.md` §2) | slots 1, 2 and 3 | the playback slots write its local position and quaternion |

### 2a. The `+0x04` flag word

| Value | Meaning | Evidence |
|---|---|---|
| `0x01` | **playback active** | `human_update` runs `human_animation_evaluate_and_dispatch` **only while it is clear** — the bit stands the normal animation driver down so the track can own the pose. Tested together with `0x04` (mask `5`) by `actor_tick`, `car_update` (four sites), `car_on_act_state_changed`, `human_update` and `entity_suspend_ai_and_hide` |
| `0x02` | **recording active** | `human_track_queue_anim_event` is gated on it, and the record it produces has exactly one consumer: `human_track_record_key`. `dropout_on_act_state_changed` returns early while it is set, so a `C_DropOut` going inactive is not recycled mid-recording, and `dropout_game_done` tests it with `0x01` (mask `3`) |
| `0x04` | playback **requested** | promoted to `0x01` by `actor_track_apply_mode_changes` |
| `0x08` | recording **requested** | promoted to `0x02` by the same |
| `0x10` | **has a track** | set by `recorder_add_track`; read by `human_update` (skip the frame's own tick) and `game_init` (find the player actor that has no track yet) |

**Enabling is deferred, disabling is immediate.** `set(true)` raises only the *request* bit
and only when the mode is not active; `set(false)` clears the active and request bits together.
So a mode change always lands at a controlled point.

**Where the meanings come from.** The three setters have **0 direct callsites and 35 vtable
datarefs each**, so no caller passes a literal and the meaning comes from the readers.
`0x02 = recording` rests on a closed writer→reader data chain plus the `C_DropOut` readers;
`0x01 = playback` on the suppression readers above, which always test it together with its own
*requested* bit `0x04` and never with `0x02` or `0x08`. The readers were found by disassembling
every function for bit tests on displacement `+0x04` and keeping those whose receiver is the actor.

## 3. Method family

Coverage: the 13 interface slots are filled by the 11 `actor_track_` functions and two shared
defaults (slots 0 and 12). The class is PC-only (§7), so the tables carry no PS2 columns.

### 3a. Construction & destruction

The interface has no constructor or destructor. The rows below are the static-init code of the
translation unit it shares with the recorder's key-stream helpers.

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00548830 | `crt_static_init_guard_00548830` | the translation unit | - | the unit's `.CRT$XCU` dynamic initializer; tail-jumps to the registrar below |
| 0x00548850 | `crt_atexit_register_00548850` | the translation unit | - | registers the shared do-nothing unit destructor with `crt_atexit` |
| 0x00548870 | `crt_static_init_zero_vector_00656930` | `g_zero_vector_00656930` | - | zeroes a file-scope `{begin, end, capacity}` triple that nothing else in the image references |

### 3b. Accessors

None: every interface method does more than one field access.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x0044a650 | `actor_track_create` | - | slot 9: asks `C_recorder` for a track, declaring the per-key-type sizes |
| 0x0044a780 | `stub_retminus1_args4_0044a780` | - | slot 0 default: record a key now if one is due; **−1 = nothing recorded**. ICF-folded across 28 vtables |
| 0x0044a790 | `actor_track_apply_key_texture_anim` | - | slot 1: applies the key's texture-animation payload (`flags & 0x7ff` = id, bit `0x2000` = loop) |
| 0x0044a860 | `actor_track_apply_pose_at_clock` | - | slot 2: applies the interpolated pose at `g_recorder_playback_clock_ms`, using the cached cursor pair |
| 0x0044ad10 | `actor_track_apply_pose_at_time` | - | slot 3: the same at an explicit time, searching for the bracketing key; gated on `kind == 1` |
| 0x0044b250 | `actor_track_get_pos_at_time` | - | slot 4: read-only position sample → out-vec3 |
| 0x0044b470 | `actor_track_get_rot_at_time` | - | slot 5: read-only rotation sample → out-quat |
| 0x00548900 | `actor_track_key_offset_of` | - | slot 11: byte offset of a cursor inside another track's stream |
| 0x00548950 | `actor_track_key_stream_size` | - | slot 10: total bytes of the key stream |
| 0x00548a60 | `actor_track_set_recording` | - | slot 6: bit `0x02` active / `0x08` requested |
| 0x00548a80 | `actor_track_set_playback` | - | slot 7: bit `0x01` active / `0x04` requested |
| 0x00548aa0 | `actor_track_apply_mode_changes` | - | slot 8: promotes `0x08→0x02` and `0x04→0x01` |
| 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | - | slot 12 default: a bare `ret` (the debug dump); `C_human` overrides it with `human_track_dump_keys` |

### 3d. Free helpers

None: the helpers in the shared translation unit are the recorder's `recorder_key_stream_*`
functions (`structs/recorder.md`).

## 4. The vtable slots

| Slot | PC VA | PC name | Role |
|---|---|---|---|
| 0 | 0x0044a780 | `stub_retminus1_args4_0044a780` | record a key now if one is due; −1 = nothing recorded |
| 1 | 0x0044a790 | `actor_track_apply_key_texture_anim` | apply the key's texture-animation payload |
| 2 | 0x0044a860 | `actor_track_apply_pose_at_clock` | apply the interpolated pose at the playback clock |
| 3 | 0x0044ad10 | `actor_track_apply_pose_at_time` | the same at an explicit time |
| 4 | 0x0044b250 | `actor_track_get_pos_at_time` | read-only position sample |
| 5 | 0x0044b470 | `actor_track_get_rot_at_time` | read-only rotation sample |
| 6 | 0x00548a60 | `actor_track_set_recording` | recording request / stop |
| 7 | 0x00548a80 | `actor_track_set_playback` | playback request / stop |
| 8 | 0x00548aa0 | `actor_track_apply_mode_changes` | promote the request bits |
| 9 | 0x0044a650 | `actor_track_create` | create the recorder track |
| 10 | 0x00548950 | `actor_track_key_stream_size` | total bytes of the key stream |
| 11 | 0x00548900 | `actor_track_key_offset_of` | byte offset of a cursor in another track's stream |
| 12 | 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | debug dump; a bare `ret` by default |

### 4a. Known overrides

| Class | Slot | PC function |
|---|---|---|
| `C_actor_shot` | 1 | `actor_shot_track_apply_key_effects` |
| `C_actor_shot` | 4 | `stub_zero_vec3_out` — ICF fold (§4b) |
| `C_actor_shot` | 5 | `stub_zero_quat_out` — ICF fold (§4b) |
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
| `C_box` | 2 | `folded_track_apply_rigid_pose_at_clock` — ICF fold (§4b) |
| `C_box` | 4 | `stub_zero_vec3_out` — ICF fold (§4b) |
| `C_box` | 5 | `stub_zero_quat_out` — ICF fold (§4b) |
| `C_box` | 9 | `box_track_create` — `recorder_add_track(0x28, 0x28, 0x30)` |
| `C_DropOut` | 0 | `dropout_track_record_key` — `{time, type 1, pos, quat}`; 200 ms suppression while act-state 1; returns the `-1` sentinel |
| `C_DropOut` | 2 | `folded_track_apply_rigid_pose_at_clock` — ICF fold (§4b) |
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

² See §4c.

### 4b. ICF folds — a slot gives the role, only the dataref count gives the owner

The rigid-body classes have no animation, so their track overrides reduce to the same pos+quat
code and the linker's identical-COMDAT folding merges them. Each of these sits inside one
class's TU and fills that class's slot, yet belongs to no single class:

| Function | VA | Slot | Vtables holding it (scan of `.rdata` for the address) |
|---|---|---|---|
| `stub_zero_vec3_out` | `0x0045f0f0` | 4 | `0x0063b720` `C_actor_shot` · `0x0063b980` `C_box` |
| `stub_zero_quat_out` | `0x0045f110` | 5 | `0x0063b720` `C_actor_shot` · `0x0063b980` `C_box` |
| `folded_track_apply_rigid_pose_at_clock` | `0x0048aaf0` | 2 | `0x0063c320` `C_DropOut` · `0x0063b980` `C_box` |
| `bottle_subclass_init_require_frame` | `0x0048af90` | 18 | `C_cartridge` · `C_grenade` · `C_smoke_trail` |

* `stub_zero_vec3_out` / `stub_zero_quat_out` are two classes' identical "no positional /
  rotational playback" overrides; they sit inside the `C_box` TU.
* `folded_track_apply_rigid_pose_at_clock` sits inside the `C_DropOut` TU. Its two sibling overrides in
  the same TU have **one dataref each** and keep the `dropout_` prefix, so the census has to be
  run per function, not per translation unit.
* `bottle_subclass_init_require_frame` (slot 18, `Init`, outside this interface) sits inside the
  `C_grenade` TU and calls `bottle_init`; it is named for the family.

Expect a fold whenever two `C_PhThing`-family classes fill the same slot.

### 4c. Slot 21 = `DuplicateFrom`, and the `C_car` overrides

**Slot 21** is not part of the 0–12 interface; it is recorded here because this is where the
family slot map lives. `box_new_duplicate_from` pins it: a **96-byte** body copying six fields in
order after a base call (its PS2 twin: §7). Every class that does not override it holds a
folded no-op stub there: `C_actor` → `stub_noop_args4_00461180`, `C_bottle` →
`thunk_stub_noop_args4_00461180` (`0x004773f0`, a 5-byte JMP thunk; Ghidra names a thunk after
its target), `C_box` → `stub_noop_args4_004776f0`.

**The six `C_car` overrides** (§4a) carry `car_track_`, not `vehicle_`: the receiver is
byte-proven `5C_car`. Every one reads the track from `this+0x08` and the I3D frame from
`this+0x68`, both `C_actor` **base** fields that `C_Vehicle` — which lives at `car+0x80` — does
not have. Vtable `0x0063bc08` also carries `car_ai_tick`, `car_update`, `car_game_init` and
`car_get_actor_speed` (their PS2 twins: §7), and a vtable belongs to exactly one class.

`car_track_spawn_dropout_part` (slot 1) replays an "a part came off" key by spawning a
**`C_DropOut`** actor (kind `0x11`) bound to the named car part. That kind is labelled data:
`recorder_new_track` formats track names with `case 0x11: "drp_%d"` beside
`act_`/`hum_`/`car_`/`box_`. The function's two branches — wheel via `car+0xd24`, body panel via
the `0x34`-stride table at `car+0x308` — match `C_DropOut`'s two initialisers (§7). See
`structs/car.md` and `structs/dropout.md`.

## 5. Why it is a base class and not part of `C_actor`

Read directly from the vtables in `Game.exe`'s `.rdata`:

* The **same 13 functions occupy slots 0–12 of 28–30 vtables** — every vtable in the
  `C_actor` family, i.e. exactly the set identified by the slot-16 scalar-deleting-destructor
  fingerprint.
* `C_actor`'s **own** virtuals begin at slot 13 (`structs/actor.md` §4). Slots 13/14/15 in
  `C_actor`'s vtable are its own `AI` / `Update` / `NetDirect`, all folded by `/OPT:ICF` onto the
  single stub `stub_noop_args4_004776f0`.

**What is measured:** the 13 form one coherent block; they are emitted as **COMDATs in a
shared band** (`0x0044a650`–`0x0044b5c0`) and in the small unit at `0x00548830` rather than
inside any one class's translation unit — `C_actor_shot`'s two overrides sit in that band while
its own TU is 0x4000 bytes away — and they are a self-contained interface to one subsystem.
Because the band is COMDAT-grouped, its emission order carries no information.

**Whether the base is a separate class is not decidable from the binaries.** No class outside the
`C_actor` family uses the interface. No stand-alone 13-slot vtable exists in `.rdata` — all 28
vtables holding the slot-0 default run 39 slots or more — and `actor_ctor` and `actor_dtor` each
store a single vptr, so a separate base could only have been declared `__declspec(novtable)`. The
PC ships no RTTI and the PS2 has neither the class nor the 13 methods (§7). A `novtable` base and
13 leading virtuals of `C_actor` produce the same bytes.

## 6. The key record

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

## 7. PC vs PS2

**The entire subsystem is PC-only.** Not one of `lastKeyTime`, `animflags`, `animid`, `.dump`,
or any `C_recorder:` string exists in `SLUS_206.71`, and no PS2 symbol carries the class —
`C_actor_track` occurs 0 times in the PS2 symbol table and decomp (so there is no
`Create__13C_actor_track` or `RecordKey__13C_actor_track`). Every function in this file has
**no PS2 twin, by measurement rather than by search failure** — the inverse of the usual case,
where the PS2 port *added* subsystems.

* **The vtable offset.** PS2 `__vt__7C_actor` starts with `C_actor`'s own methods and holds no
  interface slots before them, so **PC slot = PS2 vtable index + 13**. The formula is anchored by
  nine CONFIRMED twins:

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

PS2 `__vt__7C_actor` holds 27 methods; indices 0–24 map to PC slots 13–37, and the last two
have no PC slot (`structs/actor.md` §8).

* **The PS2 vptr placement.** PS2 `__ct__7C_actorFv` stores its vtable at `this+0x78`: GCC 2.x
  places the vptr **after** the data for a class with no polymorphic base, and PS2 `C_actor`
  has none — `__dt__7C_actorFv` restores exactly one vptr and calls no base destructor.
* **Field layout:** the base fields this interface uses — `+0x04` mode flags and `+0x08` track —
  have no PS2 counterpart, and PS2's `C_actor` data starts at `+0x00` with its cached vectors.
  The full pair table is `structs/actor.md` §8.
* **`.tck` track files ship on both builds** (`Ddeath.tck`, `DturnL.tck`, `DturnR.tck`,
  `Nuda01.tck`, `Nuda02.tck`, `turn1.tck`) and the PS2 keeps consumers for them
  (`LoadTCK__5C_dogFv`, `GetPlacementFromTCK__5C_car`,
  `GetPlacementFromTCK__28C_animations_machine_de_luxe`) — the port kept the *data* and the
  readers, and cut the recorder that produced it.
* **`C_DropOut`'s two initialisers** on the PS2 are `InitWheel` and `InitBox` — the same
  two-branch split `car_track_spawn_dropout_part` makes (§4c).
* **Slot 21 = `DuplicateFrom`** (§4c) is pinned by the CONFIRMED pair `box_new_duplicate_from` ↔
  `DuplicateFrom__9C_box_newFP7C_actor`: **96 bytes on both builds**, copying the same six fields
  in the same order after a base call.
* **The `C_car` vtable `0x0063bc08`** (§4c) holds `car_ai_tick` ↔ `AI__5C_carFUi`, `car_update` ↔
  `Update__5C_carFUi`, `car_game_init` ↔ `GameInit__5C_carFv` and `car_get_actor_speed` ↔
  `GetActorSpeed__5C_carFR8S_vector`.

## 8. Related

`structs/recorder.md` (the manager) · `structs/actor.md` (the derived root) ·
`structs/actor_shot.md` · `structs/entity.md` · `structs/human.md` · `structs/car.md` ·
`structs/dropout.md` · `structs/box.md`.

## 9. Open items

Nothing is open.
