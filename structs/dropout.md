# `C_DropOut` — a part that has come off a car (kind `0x11`)

**Created** 2026-08-13 17:18 +0200 · **Modified** 2026-11-02 17:43 +0100

A wheel or a body panel that has detached from a car and is now a physics object of its own.
`C_PhThing` does the tumbling; this class adds its own scene frame, the two initialisers that copy
the car's part descriptor into the physics body, a gate on every per-frame virtual, and a **cap of
fifteen live parts** (§5).

| | |
|---|---|
| PC prefix | `dropout_` |
| PC kind (`+0x10`) | `0x11` |
| PC vtable | `g_dropout_vt` @ `0x0063c320` (39 slots) |
| PC base class | `C_PhThing` (`structs/physical_object.md`), derived from `C_actor` |
| PC instance size | `0x3b8` (`operator_new(0x3b8)` at all three creation sites; the last field ends at `+0x3b5`, padded to `+0x3b8`) |
| PC constructor | `dropout_ctor` @ `0x0048a530` |
| PC translation unit | `[0x0048a500, 0x0048af00)`, guard to guard: 18 starts, all named |
| PS2 class | `C_DropOut` (tag `9C_DropOut`), 12 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x3b0` is the `C_PhThing` base (`structs/physical_object.md`
§1, `structs/actor.md` §1). The class uses six base fields: the `C_actor_track` mode flags `+0x04`
(bit `0x01` playback, `0x02` recording; `structs/actor_track.md`), read by `dropout_game_done` and
`dropout_on_act_state_changed`; the recorder track `+0x08`, read by `dropout_track_record_key`; the
kind `+0x10` (the ctor); the act state `+0x1c`; the savegame exclusion byte `+0x65`, which
`dropout_game_init` raises so the savegame skips the part; and the frame `+0x68`, which
`dropout_init` creates.

```c
typedef struct C_DropOut {                    // 0x3b8 bytes; C_PhThing base +0x00..+0x3b0
    /* +0x3b0 */ uint32_t part_kind;          // [M] body-panel kind 2..7; 0 for a wheel (§4a); no reader (.text scan)
    /* +0x3b4 */ uint8_t  retire_pending;     // [M] non-zero: every virtual switches the part off; no non-zero writer (.text scan)
} C_DropOut;
```

**`part_kind`** is the `S_DROPOUT_BOX +0x14` value `dropout_init_box` copies in (§4a); a wheel and
a replayed part keep the ctor's 0. Nothing reads it: the creators carry the part kind separately,
in the network message and in the recorder key.

**`retire_pending`** is named from its readers. `dropout_ai_tick` and `dropout_update` run the
`C_PhThing` body while it is 0 and otherwise call `actor_set_act_state`; `dropout_on_collision` and
`dropout_hit` do the same with act state 2. Its only writer stores 0 (Open item 2).

## 2. Field evidence

Measured by a census of every `this + N` access in the `dropout_*` functions, a `.text`
displacement census for `+0x3b0` and `+0x3b4` (every byte store at `+0x3b4` in the image), and body
reads.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x3b0 | part_kind | M | `dropout_ctor` (0), `dropout_init_box` | - | no load on this class (`.text` scan, Open item 2) |
| +0x3b4 | retire_pending | M | `dropout_game_init` (0) | `dropout_ai_tick`, `dropout_update`, `dropout_on_collision`, `dropout_hit` | named from its readers; no PC writer stores a non-zero value (`.text` scan, Open item 2) |

## 3. Method family

12 of 12 PS2 methods mapped (11 CONFIRMED, 1 STRUCTURAL); 5 PC-only starts: the unit's static-init
pair and the three `C_actor_track` overrides (the track layer is PC-only, `structs/recorder.md`).
Every other vtable slot holds an inherited `C_actor` / `C_PhThing` function.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0048a500 | `crt_static_init_guard_0048a500` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0048a520 | `crt_atexit_register_0048a520` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x0048a530 | `dropout_ctor` | instance | `__ct__9C_DropOutFv` | 0x00161940 | CONFIRMED | `physical_object_ctor`, vtable, kind `0x11`, `part_kind` 0; allocates the arena on the 0→1 edge of `g_dropout_count` (§5); three callsites (§4) |
| 0x0048a5d0 | `dropout_scalar_deleting_dtor` | instance | `__dt__9C_DropOutFv` | 0x001619b0 | STRUCTURAL | slot 16; the delete arm of the fused PS2 destructor |
| 0x0048a5f0 | `dropout_dtor` | instance | `__dt__9C_DropOutFv` | 0x001619b0 | CONFIRMED | frees the arena on the 1→0 edge of `g_dropout_count`, chains `physical_object_dtor` |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | - | - | - | `C_actor_track` slot 1, the key replay: an empty override (the part replays no key flags); a generic no-op shared by 21 vtables |
| 0x0048a630 | `dropout_init` | `Init__9C_DropOutFP9I3D_frame` | 0x00161a50 | CONFIRMED | slot 18; creates an `OBJ_` frame (`0x5f4a424f`), copies the source frame's pose and scale, links it into the mission scene, then `physical_object_init`; releases the frame and returns 0 when that fails |
| 0x0048a810 | `dropout_game_init` | `GameInit__9C_DropOutFv` | 0x00161cc0 | CONFIRMED | slot 25; `physical_object_game_init`, `+0x65` = 1, `retire_pending` = 0 |
| 0x0048a830 | `dropout_game_done` | `GameDone__9C_DropOutFv` | 0x00161cf0 | CONFIRMED | slot 26; `physical_object_game_done`; while `+0x04 & 3` (a track plays or records) rewinds the key cursors and detaches the recorder binding; then releases the frame and clears `+0x68` |
| 0x0048a880 | `dropout_ai_tick` | `AI__9C_DropOutFUi` | 0x00161d50 | CONFIRMED | slot 13; the `retire_pending` gate around the `C_PhThing` AI |
| 0x0048a8a0 | `dropout_update` | `Update__9C_DropOutFUi` | 0x00161da0 | CONFIRMED | slot 14; the same gate around the `C_PhThing` update |
| 0x0048a8c0 | `dropout_on_act_state_changed` | `ChangeState__9C_DropOutFv` | 0x00161dd0 | CONFIRMED | slot 27; the arena policy of §5 |
| 0x0048aa30 | `thunk_physical_object_init_wheel` | `InitWheel__9C_DropOutFP15S_DROPOUT_WHEEL` | 0x00161e80 | CONFIRMED | a 5-byte `jmp` to `physical_object_init_wheel`, 0 datarefs (non-virtual; the wheel arms of `car_track_spawn_dropout_part` and `car_apply_net_event_msg` call it directly) ¹ |
| 0x0048aa40 | `dropout_init_box` | `InitBox__9C_DropOutFP13S_DROPOUT_BOX` | 0x00161e90 | CONFIRMED | `physical_object_init_box`, then `part_kind` = descriptor `+0x14` |
| 0x0048aa60 | `dropout_on_collision` | `Collision__9C_DropOutFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00161ed0 | CONFIRMED | slot 30; the gate around `physical_object_on_collision`, else act state 2; returns 1 |
| 0x0048aaa0 | `dropout_hit` | `Hit__9C_DropOutF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00161f00 | CONFIRMED | slot 31; the gate around `physical_object_hit`, else act state 2; returns 1 |
| 0x0048aaf0 | `folded_track_apply_rigid_pose_at_clock` | - | - | - | `C_actor_track` slot 2, an ICF fold shared with `C_box` (§6b) |
| 0x0048ada0 | `dropout_track_create` | - | - | - | `C_actor_track` slot 9; `recorder_add_track(this, 0x24, 0x24, 0x24)` |
| 0x0048adc0 | `dropout_track_record_key` | - | - | - | `C_actor_track` slot 0; writes a `[time, 1, position, quaternion]` key; while the part is asleep (act state 1) it skips keys less than 200 ms apart and repeats the last recorded pose |

¹ The PC name stays `thunk_<target>`: a thunk's name carries its target, which is how the thunk's
VA is recovered from the name. The derived class adds nothing to `InitWheel`; `InitBox` adds the
one `part_kind` copy.

### 3d. Free helpers

The unit has no free helper.

## 4. The three creators

`car_track_spawn_dropout_part` replays a *"a part came off"* keyframe by spawning a kind-`0x11`
actor — a wheel from `car+0xd24`, a body panel from the `0x34`-stride table at `car+0x308`
(`structs/actor_track.md` §4c) — the same two part sources the `InitWheel` / `InitBox` pair
initialises. The replay calls neither initialiser: the recorded track supplies the pose.
`recorder_new_track` formats this kind's track names **`drp_%d`**.

| Creator | When |
|---|---|
| `car_drop_out_part` | the live path: a car sheds a part during play |
| `car_track_spawn_dropout_part` | the replay path: a recorded keyframe re-creates it |
| `car_apply_net_event_msg` | the network path: a received event message (the case-`0x2d` arm) re-creates it ¹ |

¹ Payload byte `+8` selects the part: `1` = wheel (from `car+0xd24`, then
`thunk_physical_object_init_wheel`), `2`..`6` = body panel (looked up through
`vehicle_find_part_at_encoded_position` into the `car+0x308` table, then `dropout_init_box`). The new
part is handed to `game_add_temporary_actor`.

`structs/physical_object.md` lists `dropout_ctor` as one of `C_PhThing`'s five subclasses.

### 4a. The two spawn descriptors

Read from the stack stores of `car_prepare_wheel_dropout` and `car_prepare_body_part_dropout`, the
two builders that hand a descriptor to `car_drop_out_part`. The network arm of
`car_apply_net_event_msg` initialises from the same block, carried in the message.

| Struct | Offset | Field | Value |
|---|---|---|---|
| `S_DROPOUT_WHEEL` | +0x00 | wheel radius | wheel record `+0x10c` |
| `S_DROPOUT_WHEEL` | +0x04 | wheel parameter | wheel record `+0x138` |
| `S_DROPOUT_WHEEL` | +0x08 | mass | 10.0 |
| `S_DROPOUT_WHEEL` | +0x0c | initial velocity (vec3) | the part's velocity + 0.1 × the impulse |
| `S_DROPOUT_WHEEL` | +0x18 | friction | wheel record `+0xd4` |
| `S_DROPOUT_WHEEL` | +0x1c | damping | wheel record `+0xe0` |
| `S_DROPOUT_BOX` | +0x00 | mass | by zone type: 10.0 bumper, 4.0 light, 2.0 plate, 2.0 mirror, 7.0 wing, 20.0 door |
| `S_DROPOUT_BOX` | +0x04 | initial velocity (vec3) | the part's velocity + the impulse ÷ the mass |
| `S_DROPOUT_BOX` | +0x10 | friction | 0.5 |
| `S_DROPOUT_BOX` | +0x14 | part kind | 2 bumper, 3 light, 4 plate, 5 mirror, 6 wing, 7 door |

`S_DROPOUT_WHEEL` is `0x20` bytes with exactly the `S_THING_WHEEL` layout, and `S_DROPOUT_BOX` is
`0x18`: `S_THING_BOX` plus the part kind (`structs/box_new.md` §4). The part kind is the selector
`car_drop_out_part` also receives, and the one `structs/car.md` §6b maps onto the deform-zone types.

## 5. The 15-part arena

Four file-scope statics:

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_dropout_count` | 0x00655ae0 | live instance count | `dropout_ctor`, `dropout_dtor` | `dropout_ctor`, `dropout_dtor` |
| `g_dropout_arena` | 0x00655ad4 | `C_DropOut *[15]`, `operator_new(0x3c)` on the count's 0→1 edge, freed on its 1→0 edge | `dropout_ctor`, `dropout_dtor` | `dropout_on_act_state_changed`, `dropout_dtor` |
| `g_dropout_arena_capacity` | 0x00655ad8 | 15, set by the ctor on the 0→1 edge | `dropout_ctor` | `dropout_ctor`, `dropout_on_act_state_changed` |
| `g_dropout_arena_used` | 0x00655adc | the array's live length | `dropout_ctor` (0), `dropout_on_act_state_changed` | `dropout_on_act_state_changed` |

`dropout_on_act_state_changed` runs `physical_object_on_act_state_changed` first and then applies
the policy:

* **act state 1** (asleep), and the part is not being recorded (`+0x04 & 2` clear):
  `if (used >= capacity) { arena[0]->SetActState(2); shift the array down; used = capacity - 1; }`
  then `arena[used++] = this` — **the sixteenth part to come to rest switches off the oldest one
  still lying there**;
* **act state 2**: removes the part from the arena, then `game_invalidate_actor` and
  `game_remove_temporary_actor`;
* **act state 0**: removes the part from the arena (a part that wakes up leaves the queue).

## 6. The track overrides

### 6a. `InitWheel` is a pure forwarder

`0x0048aa30` is a 5-byte `jmp` to `physical_object_init_wheel` with no `.rdata` dataref: it is not
virtual, and the two wheel arms of §4 call it directly.

### 6b. Track slot 2 is an ICF fold shared with `C_box`

`folded_track_apply_rigid_pose_at_clock` sits inside this unit and fills `C_DropOut`'s `C_actor_track`
slot 2, but a scan of `.rdata` for the address as a pointer finds two datarefs: slot 2 of
`0x0063c320` (this class) **and** slot 2 of `0x0063b980` (`C_box`, kind 9). The linker merged two
rigid bodies' identical playback overrides onto one symbol, so the function belongs to neither
class (`structs/actor_track.md` §4b; `stub_zero_vec3_out` / `stub_zero_quat_out` are another
instance). The other two track overrides have one dataref each and carry the `dropout_` prefix.

The fold is sound from the `C_box` side: `box_track_record_key` and `dropout_track_record_key` both
allocate their keys through `recorder_key_stream_alloc` and write the same prefix — `[1]` = 1,
`[2..4]` the position, `[5..8]` the quaternion — and the shared playback body reads exactly the time
`[0]`, the position and the quaternion.

## 7. PC vs PS2

* **Field bridge:** PC `+0x3b0` ↔ PS2 `+0x640` and PC `+0x3b4` ↔ PS2 `+0x644`, set by
  `dropout_game_init` and used by AI / Update / Collision / Hit / ChangeState; PC `+0x65` ↔ PS2
  `+0x69`. The class band starts where `C_PhThing` ends on each build. The PS2 `GameInit` stores 0
  at `+0x644` too, and `ChangeState__9C_DropOutFv` stores 1 there at act state 1, which is what
  supports the name `retire_pending`; the PC has no such store.
* **The arena is PC-only in effect.** The PS2 allocates the same arena in its constructor and never
  puts anything in it. Its `ChangeState__9C_DropOutFv` handles the act-state-1 transition by calling
  `AddDeathHumanFade__6C_game` and `DynErase__11g_collision`: it fades the part out instead. The two
  builds implement different policies for one source function, which is why that row's size ratio
  is 2.14.
* **Arena global names.** `g_dropout_count` and `g_dropout_arena` are named from the PS2 static
  class members `dropout_count__9C_DropOut` and `arena__9C_DropOut`, paired with them in the same two
  statements of a statement-level diff of both destructors. The other two globals have no PS2
  symbol because the PS2 never uses them.
* **`InitWheel`.** The PS2 symbol is 8 bytes, which on MIPS is either an empty body or a tail jump;
  it is the tail jump (Ghidra renders it as the base's body), so the PS2 derived class adds nothing
  to `InitWheel` either.
* **The whole `C_actor_track` / `C_recorder` layer is PC-only** (`structs/recorder.md`), which is
  why three of the PC starts have no PS2 counterpart, why the slot-2 fold has no PS2 symbol, and why
  `GameDone` is 4 bytes shorter on PS2 rather than absent.
* **The `retire_pending` else arms differ.** The PC adds an else arm (`actor_set_act_state`) to
  `Update`, `Collision` and `Hit` that the PS2 versions do not have; the PS2 `AI` wraps its retire
  arm (`SetActState(2)`) in a frame `+0xe0 & 1` test the PC omits.
* Size ratios, PC/PS2 (from the two symbol tables): ctor 1.60 · dtor 0.43 · Init 0.78 · GameInit
  0.67 · GameDone 0.95 · AI 0.42 · Update 0.80 · ChangeState 2.14 · InitBox 0.57 · Collision 1.45 ·
  Hit 1.82. The small ones swing wide because they are 32–80 byte functions where one inlined call
  moves the ratio a long way.

## 8. Related

`structs/physical_object.md` (the base) · `structs/drop_in_weapon.md` (the previous unit) ·
`structs/actor_track.md` (the override table, the fold trap, the key sizes) · `structs/recorder.md`
(the PC-only subsystem) · `structs/car.md` (where the parts come from) · `structs/entity_kind.md` §1
(the kind enum).

## 9. Open items

1. **Is the 15-part cap observable in play?** A pile-up of more than fifteen resting parts should
   make the oldest wreckage vanish on the PC; not tested in game.
2. **`part_kind` and `retire_pending` have no counterpart access.** Every `[reg + 0x3b4]` byte
   store in `.text` is `dropout_game_init`'s 0, so the four else arms of §3c are unreachable by that
   search; and no load of `+0x3b0` is on a `C_DropOut`: the other hits are `C_crash_obj`,
   `C_drop_in_weapon`, `C_box_new`, `C_box`, `C_vehicle` and `C_car`. Searched: every `.text` operand
   with both displacements, by whether its base register holds the function's receiver, and the
   functions testing kind `0x11` or calling a `dropout_` method. That search cannot see a block copy
   or a pointer used with no kind test; the save pair of `C_PhThing` touches neither field.
