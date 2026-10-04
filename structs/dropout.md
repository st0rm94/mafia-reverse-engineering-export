# `C_DropOut` — a part that has come off a car (kind `0x11`)

**Created** 2026-08-13 17:18 +0200 · **Modified** 2026-10-01 18:52 +0200

PC prefix **`dropout_`**. The translation unit follows `C_driver`'s.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x11`** |
| PC vtable | `.rdata 0x0063c320` (39 slots) |
| PC constructor | `dropout_ctor` @ `0x0048a530` |
| PC instance size | **`0x3b8`** — `operator_new(0x3b8)` at all three creation sites |
| PC translation unit | `[0x0048a530, 0x0048af00)` — 16 function starts, all named |
| Base class | **`C_PhThing`** (`structs/physical_object.md`), which derives from `C_actor` |
| PS2 class | `C_DropOut`, 12 methods, 12 mapped (§6) |

---

## 1. What it is

A wheel or a body panel that has detached from a car and is now a physics object in its own right.
`C_PhThing` does the tumbling; this class contributes four things — its own scene frame, the two
initialisers that copy the car's descriptor into the physics body, a "has it settled yet" gate on
every per-frame virtual, and a **cap of fifteen live parts**.

`car_track_spawn_dropout_part` replays a *"a part came off"* keyframe by spawning a kind-`0x11`
actor — wheel via `car+0xd24`, body panel via the `0x34`-stride table at `car+0x308`
(`structs/actor_track.md` §2c) — the same two part sources the `InitWheel` / `InitBox` pair below
initialises. The replay calls neither initialiser: the recorded track supplies the pose.
`recorder_new_track` formats this kind's track names **`drp_%d`**.

Three creators, and they are the whole feature:

| creator | when |
|---|---|
| `car_drop_out_part` | the live path — a car sheds a part during play |
| `car_track_spawn_dropout_part` | the replay path — a recorded keyframe re-creates it |
| `car_apply_net_event_msg` | the network path — a received event message (the case-`0x2d` arm) re-creates it ¹ |

¹ Payload byte `+8` selects the part: `1` = wheel (from `car+0xd24`, then
`thunk_physical_object_init_wheel`), `2`..`6` = body panel (looked up through
`vehicle_find_part_at_encoded_position` into the `car+0x308` table, then `dropout_init_box`). The
new part is handed to `game_add_temporary_actor`.

`structs/physical_object.md` §2 lists `dropout_ctor` as row 1 of `C_PhThing`'s closed five-subclass
set.

---

## 2. Field layout

Everything below `+0x3b0` belongs to `C_actor` / `C_actor_track` / `C_PhThing`.

| Offset | Size | Field | Evidence |
|---|---|---|---|
| `+0x04` | 4 | `C_actor_track` mode flags — bit `0x01` playback, `0x02` recording | `structs/actor_track.md` §3a; read by `dropout_game_done` |
| `+0x08` | 4 | `C_recorder` track pointer | written by `dropout_track_create` |
| `+0x10` | 4 | `kind` = `0x11` | written by the ctor |
| `+0x65` | 1 | `C_actor` save_excluded, raised at scene start: the savegame skips the part | `dropout_game_init` |
| `+0x68` | 4 | `I3D_frame*` — the part's own scene frame | created by `dropout_init` |
| `+0x3b0` | 4 | **body-panel part kind** 2..7; 0 for a wheel and for a replayed part | `dropout_init_box` copies `S_DROPOUT_BOX+0x14` (§2a); nothing reads it |
| `+0x3b4` | 1 | **settled flag** — 0 while the part is still moving | `dropout_game_init` clears it; every per-frame virtual gates on it |

`0x3b4 + 4 = 0x3b8`, exactly the `operator_new` size at all three creation sites — the layout and
the allocation confirm each other.

`+0x3b0` is written twice in the translation unit (the ctor's 0 and `dropout_init_box`'s copy) and
read nowhere; the creators carry the part kind separately, in the network message and in the
recorder key.

### 2a. The two spawn descriptors

Read from the stack stores of `car_prepare_wheel_dropout` and `car_prepare_body_part_dropout`, the
two builders that hand a descriptor to `car_drop_out_part`. The network arm of
`car_apply_net_event_msg` initialises from the same block, carried in the message from the sender.

| struct | offset | field | value |
|---|---|---|---|
| `S_DROPOUT_WHEEL` | `+0x00` | wheel radius | wheel record `+0x10c` |
| `S_DROPOUT_WHEEL` | `+0x04` | wheel parameter | wheel record `+0x138` |
| `S_DROPOUT_WHEEL` | `+0x08` | mass | 10.0 |
| `S_DROPOUT_WHEEL` | `+0x0c` | initial velocity (vec3) | the part's velocity + 0.1 × the impulse |
| `S_DROPOUT_WHEEL` | `+0x18` | friction | wheel record `+0xd4` |
| `S_DROPOUT_WHEEL` | `+0x1c` | damping | wheel record `+0xe0` |
| `S_DROPOUT_BOX` | `+0x00` | mass | by zone type: 10.0 bumper, 4.0 light, 2.0 plate, 2.0 mirror, 7.0 wing, 20.0 door |
| `S_DROPOUT_BOX` | `+0x04` | initial velocity (vec3) | the part's velocity + the impulse ÷ the mass |
| `S_DROPOUT_BOX` | `+0x10` | friction | 0.5 |
| `S_DROPOUT_BOX` | `+0x14` | part kind | 2 bumper, 3 light, 4 plate, 5 mirror, 6 wing, 7 door |

`S_DROPOUT_WHEEL` is `0x20` bytes with exactly the `S_THING_WHEEL` layout, and `S_DROPOUT_BOX` is
`0x18`: `S_THING_BOX` plus the part kind (`structs/box_new.md` §1a). The part kind is the
selector `car_drop_out_part` also receives, and the one `structs/car.md` §4b maps onto the
deform-zone types.

---

## 3. The 15-part arena

Four file-scope statics, all written by exactly three functions:

| global | what it is |
|---|---|
| `g_dropout_count` (`0x00655ae0`) | live instance count; the arena is allocated on its 0→1 edge and freed on its 1→0 edge |
| `g_dropout_arena` (`0x00655ad4`) | `C_DropOut*[15]`, `operator_new(0x3c)` |
| `g_dropout_arena_capacity` (`0x00655ad8`) | fixed at **15** by the ctor; never written again |
| `g_dropout_arena_used` (`0x00655adc`) | the array's live length |

`dropout_on_act_state_changed` is the only mutator, and the policy is one statement:

```
if (used >= capacity) { arena[0]->SetActState(2); shift the array down; used = capacity - 1; }
arena[used++] = this;
```

— i.e. **the sixteenth part to hit the ground kills the oldest one still lying there.** The PS2
build has a different policy for the same function (§6).

---

## 4. Method coverage — 16/16 PC starts

*(PC-VA ordered, PC name first.)*

| PC VA | PC name | slot | PS2 twin | status |
|---|---|---|---|---|
| 0x0048a530 | `dropout_ctor` | - | `__ct__9C_DropOutFv` | CONFIRMED |
| 0x0048a5d0 | `dropout_scalar_deleting_dtor` | 16 | `__dt__9C_DropOutFv` | STRUCTURAL ¹ |
| 0x0048a5f0 | `dropout_dtor` | - | `__dt__9C_DropOutFv` | CONFIRMED |
| 0x0048a630 | `dropout_init` | 18 | `Init__9C_DropOutFP9I3D_frame` | CONFIRMED |
| 0x0048a810 | `dropout_game_init` | 25 | `GameInit__9C_DropOutFv` | CONFIRMED |
| 0x0048a830 | `dropout_game_done` | 26 | `GameDone__9C_DropOutFv` | CONFIRMED — §5 |
| 0x0048a880 | `dropout_ai_tick` | 13 | `AI__9C_DropOutFUi` | CONFIRMED |
| 0x0048a8a0 | `dropout_update` | 14 | `Update__9C_DropOutFUi` | CONFIRMED |
| 0x0048a8c0 | `dropout_on_act_state_changed` | 27 | `ChangeState__9C_DropOutFv` | CONFIRMED — §3 |
| 0x0048aa30 | `thunk_physical_object_init_wheel` | - | `InitWheel__9C_DropOutFP15S_DROPOUT_WHEEL` | CONFIRMED — §4a |
| 0x0048aa40 | `dropout_init_box` | - | `InitBox__9C_DropOutFP13S_DROPOUT_BOX` | CONFIRMED |
| 0x0048aa60 | `dropout_on_collision` | 30 | `Collision__9C_DropOutF…` | CONFIRMED |
| 0x0048aaa0 | `dropout_hit` | 31 | `Hit__9C_DropOutF…` | CONFIRMED |
| 0x0048aaf0 | `track_apply_rigid_pose_at_clock` | *track 2* | - | STRUCTURAL — ICF fold, §4b |
| 0x0048ada0 | `dropout_track_create` | *track 9* | - | STRUCTURAL — PC-only subsystem |
| 0x0048adc0 | `dropout_track_record_key` | *track 0* | - | STRUCTURAL — PC-only subsystem |

¹ The delete arm of the fused PS2 destructor.

Slots 15/19/20/21/35 and the rest of the vtable hold the shared `C_actor`-family stubs; `C_DropOut`
overrides nothing there.

### 4a. `InitWheel` is a pure forwarder

`0x0048aa30` is a 5-byte `jmp` to `physical_object_init_wheel`, with **0 datarefs** — it is not
virtual; the wheel arms of `car_track_spawn_dropout_part` and `car_apply_net_event_msg` call it
directly. The derived class adds nothing to `InitWheel`, while `InitBox` adds exactly one field copy
(`+0x3b0`).

The PC name stays `thunk_<target>`: a thunk's name carries its target, which is how the thunk's VA
is recovered from the name.

### 4b. Track slot 2 is an ICF fold shared with `C_box` — it belongs to neither class

`track_apply_rigid_pose_at_clock` sits inside this translation unit and fills `C_DropOut`'s
`C_actor_track` slot 2, but a scan of `.rdata` for the address as a pointer finds two datarefs:
slot 2 of `0x0063c320` (this class) **and** slot 2 of `0x0063b980` (`C_box`, kind 9, ctor
`box_ctor`). The linker merged two rigid bodies' identical playback overrides onto one symbol.

This is the trap described in `structs/actor_track.md` §2b; `stub_zero_vec3_out` /
`stub_zero_quat_out` are the other instance. **A vtable slot tells you the role; only the dataref
count tells you the owner.**

The other two track overrides have **one dataref each**, so they *are* this class's own and carry
the `dropout_` prefix.

Read from the `C_box` side, the fold is sound: `box_track_record_key` and `dropout_track_record_key`
both allocate their keys through `recorder_key_stream_alloc` and write the same prefix — `[1]` = 1,
`[2..4]` the position, `[5..8]` the quaternion — and the shared playback body reads exactly the
time `[0]`, the position and the quaternion. Both classes replay correctly through the one symbol.

---

## 5. `dropout_game_done`

Slot 26 is `GameDone` across the whole `C_PhThing` family, and the function sits inside the
CRT-bracketed TU that opens with `dropout_ctor`. Its body:

* calls `physical_object_game_done` first;
* tests `+0x04 & 3` — the `C_actor_track` mode word (playback / recording);
* rewinds the key cursors;
* unbinds and releases the frame at `+0x68`.

The `C_recorder` interface it touches is the same one the genuine `C_entity` track overrides
implement; the function itself is a `C_DropOut` method.

---

## 6. PC vs PS2

* **Field bridge:** PC `+0x3b0` ↔ PS2 `+0x640` and PC `+0x3b4` ↔ PS2 `+0x644`, fixed by
  `dropout_game_init` and re-used by AI / Update / Collision / Hit / ChangeState; PC `+0x65` ↔ PS2
  `+0x69`. The class band starts where `C_PhThing` ends on each build.
* **The arena is PC-only in effect.** The PS2 allocates the same arena in its constructor and never
  puts anything in it. Its `ChangeState__9C_DropOutFv` handles the identical act-state-1 transition
  by calling `AddDeathHumanFade__6C_game` and `DynErase__11g_collision` — it fades the part out
  instead. The two builds implement *different* LOD policies for one source function, which is why
  that row's size ratio is 2.14.
* **Arena global names.** `g_dropout_count` and `g_dropout_arena` are named from the PS2 static
  class members `dropout_count__9C_DropOut` and `arena__9C_DropOut`, paired with them in the same
  two statements of a statement-level diff of both destructors. The other two globals have no PS2
  symbol because the PS2 never uses them.
* **`InitWheel`.** The PS2 symbol is 8 bytes, which on MIPS is either an empty body or a tail jump;
  it is the tail jump (Ghidra renders it as the base's body), so the PS2 derived class adds nothing
  to `InitWheel` either.
* **The whole `C_actor_track` / `C_recorder` layer is PC-only** (`structs/recorder.md`), which is
  why three of the sixteen starts have no PS2 counterpart, why the slot-2 fold and the vtable stubs
  have no PS2 symbol, and why `dropout_game_done` is 4 bytes *shorter* on PS2 rather than absent.
* **The PC adds an `else SetActState(2)` arm** to `Update`, `Collision` and `Hit` that the PS2
  versions do not have; conversely **PS2 wraps `AI`'s retire arm in a `frame->flags & 1` test** the
  PC omits.
* Size ratios, PC/PS2: ctor 1.60 · dtor 0.43 · Init 0.78 · GameInit 0.67 · GameDone 0.95 · AI 0.42
  · Update 0.80 · ChangeState 2.14 · InitBox 0.57 · Collision 1.45 · Hit 1.82. The small ones swing
  wide because they are 32–80 byte functions where one inlined call moves the ratio a long way.

---

## 7. Related

`structs/physical_object.md` (the base, §2 lists this class) · `structs/drop_in_weapon.md` (the
sibling TU) · `structs/actor_track.md` (§2a the override table, §2b the fold trap, §4 the key sizes)
· `structs/recorder.md` (the PC-only subsystem) · `structs/car.md` (where the parts come from) ·
`structs/entity.md` §1 (the kind enum).

---

## 8. Open items

| item | state |
|---|---|
| is the 15-part cap observable in play? | a fifteen-part pile-up should make the oldest wreckage vanish on the PC; not tested in game |
