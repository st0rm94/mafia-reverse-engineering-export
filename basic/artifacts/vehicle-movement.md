# Vehicle Movement & Physics — Mafia: The City of Lost Heaven

**Created** 2026-06-07 11:06 +0200 · **Modified** 2026-10-12 20:58 +0200

How a car moves each frame in `Game.exe`. Anchors are VAs / function names / field offsets.

> Companion docs: `basic/artifacts/game-loop.md` (the per-frame loop the vehicle ticks hang off) and
> `police-system/artifacts/police-system.md` (how vehicle motion that strikes a cop or cop car
> becomes a police offence).

---

## 1. The vehicle object (multiple inheritance)

A vehicle is a C++ object built on an LS3DF `I3D_frame` scene node, **multiply inherited**. `car_ctor` (`0x00462af0`) installs **two** vtable pointers:

| obj offset | vtable VA | role |
|---|---|---|
| `+0x00` | `0x0063bc08` | primary game-class vtable (41 slots) |
| `+0x70` | `0x0063bcac` | the `C_Vehicle` second base of the `C_car` / `C_Vehicle` pair (1 slot → `0x0053b850` `vehicle_open_sound`) |

The subclass `C_car_ex` uses `car_extended_ctor` (`0x00474cf0`), installing the derived pair `0x0063bd68` / `0x0063bd64` and the flavour tag `+0x14 = 3`. Both primary tables are **41 slots, slot-for-slot aligned**, and **exactly 8** slots differ (14, 16, 18, 19, 20, 21, 25, 26) — the 8 methods `C_car_ex` overrides. Doc: `structs/car_extended.md`.

All the per-frame work is reached by **polymorphic vtable dispatch** — the vehicle tick methods have zero static callers, so a search of the decomp text for a caller dead-ends; the slot assignment comes from the vtable order in `.rdata`.

---

## 2. The per-frame virtual cluster (primary vtable `0x0063bc08`)

There is no single "Update" — the vehicle is ticked through a cluster of virtuals:

| slot | offset | function | role |
|---|---|---|---|
| #0  | `+0x00` | `car_track_record_key` | RECORD side of the transform record/playback subsystem (§4) |
| #2  | `+0x08` | `car_track_apply_pose_at_clock` | PLAYBACK: interpolate transform, derive velocity (§4) |
| #3  | `+0x0c` | `car_track_apply_pose_at_time` | playback over a streamed keyframe list (§4) |
| #4  | `+0x10` | `car_track_get_pos_at_time` | sample recorded motion vec3 at a time (§4) |
| #13 | `+0x34` | `car_ai_tick` | the per-frame tick of every car: the traffic brake/steer block (gated on `+0x20bd`, §5), stall timer, flasher and horn, then **`vehicle_tick_drivetrain`, unconditionally** (§3), then **the sidewalk-offence check** |
| #14 | `+0x38` | `car_update` | camera-distance LOD/cull flags `+0xd54`, path-node lifecycle `+0x210c`, `I3D_frame::UpdateWMatrixProc`, motion cues off `+0x1f7c` |
| #14 | `+0x38` (`C_car_ex` override) | `car_extended_update` | `C_car_ex::Update`; the burnt-corpse spawn is one of its five blocks |
| #15 | `+0x3c` | `car_net_apply_remote_state` | 🔴 **NOT the live integrator** — the dead-reckoning extrapolator for a remotely-owned car (§3); inherited unchanged by `C_car_ex` |

Slot 15 is the network-extrapolation slot family-wide; `C_airplane`'s slot 15 is a bare `ret 4` (`structs/bottle.md` §3a).

---

## 3. Driving physics — what is measured

### The local physics step

A locally simulated car moves through one chain, reached from slot 13:
`car_ai_tick` → `vehicle_tick_drivetrain` → `vehicle_substep_move` → `vehicle_move` (once per
sub-step) → `vehicle_wheel_contact_solve` (twice per `vehicle_move`). The offsets in this table
are relative to the `C_Vehicle` base, `car+0x70`.

| step | function | what it does |
|---|---|---|
| input | `vehicle_set_power`, `vehicle_set_brake`, `vehicle_set_handbrake`, `vehicle_set_steer`, `vehicle_set_clutch` | write the gas channels `+0x420` / `+0x424`, the net wheel force `+0x634`, the handbrake `+0x4b0`, the steering target `+0x6a0` and the clutch `+0x65c` (`structs/vehicle.md` §2, §4) |
| drivetrain | `vehicle_tick_drivetrain` | reads the clutch and gas channels, produces the throttle outputs `+0x5bc` / `+0x5c0`, runs the automatic gearbox (`vehicle_set_gear`), the engine revs (`vehicle_update_motor_revs`) and the motor force (`vehicle_compute_motor_force`, three calls, scaled by the engine-damage factor `+0x230`), then calls `vehicle_substep_move` |
| sub-steps | `vehicle_substep_move` | when the frame time exceeds `+0xca0` it runs `⌊dt / +0xca0⌋ + 1` equal sub-steps, otherwise one; every `vehicle_move` call receives three speed-squared terms `+0x618² · a · b · 0.65`, from the pairs `+0x5b4` / `+0x5b8`, `+0x44c` / `+0x450` and `+0x454` / `+0x458` |
| integrator | `vehicle_move` | the only writer in its translation unit of the linear velocity `+0x1f0c..+0x1f14`; also writes the angular velocity `+0x49c..+0x4a4`, the scalar speed `+0x618`, the wheel force sums `+0x1c4` / `+0x1c8` and the body basis `+0xcc4..+0xcd0`; reads the gas, handbrake and force channels; calls the wheel solver twice and `vehicle_body_coll_resolve` once |

`+0xca0` is the sub-step length in seconds: `vehicle_ctor` seeds 0.028, `vehicle_set_cut_time`
accepts 5 to 50 ms, and `traffic_car_game_init` and `traffic_car_adopt_car_with_new_driver` set
42 ms on traffic cars.

### `car_net_apply_remote_state` (`0x0059ad60`) — the REMOTE extrapolator
`__thiscall(C_car, float ms_delta)`, `C_car` vtable `0x0063bc08` slot 15.

This function does not move the car the local player drives. It advances a car whose simulation
belongs to another peer, between the state messages that peer sends — the `C_car` sibling of
`bottle_net_apply_remote_state` (2112 B) and `physical_object_net_apply_remote_state` (1696 B).
The velocity it integrates arrives over the wire: it is latched by `car_net_snapshot_state`
(slot 39) and refreshed by `car_apply_net_event_msg` (slot 38) from the type-0x1e/0x1f snapshot.

Each frame it:
- **Damps velocity:** linear velocity `+0x2170/+0x2174/+0x2178` and a second (angular) velocity `+0x21a8/+0x21ac/+0x21b0` are decayed by `dt × per-axis coefficient` (`+0x21b4/+0x21b8/+0x21bc`), windowed by `timeGetTime()` vs `+0x21cc/+0x21d0`.
- **Integrates world position:** `+0x24/+0x28/+0x2c += velocity·dt`, mirrored to `+0x40c/+0x410/+0x414`. (dt = `ms_delta * 0.001` seconds.)
- **Max-displacement snap-back clamp:** if the step exceeds `5.0` (or `12.5` when flag `+0x2198` bit `0x200` is set) it reverts to the previous position `+0x2164/68/6c` and zeroes the `+0x21c0/c4/c8` + `+0x21e4/e8/ec` accumulators — anti-tunneling / sanity guard.
- **Normalizes the orientation basis** (`+0xd28..`, `+0xd44..`).
- **Runs the wheel solver** `vehicle_wheel_contact_solve(...)` with dt in seconds.

### `vehicle_wheel_contact_solve` (`0x005334e0`) — the wheel/suspension solver
Called twice per `vehicle_move`, and once each by `car_recompute_velocity` and
`car_net_apply_remote_state`. Per wheel (the wheel-record fields are those of
`def-files-docs/vehicles-bin.md` §4b):

1. **Contact query.** A settled car (speed `+0x618` below 1.5626 and every wheel's two velocity
   vectors below their thresholds) reuses each wheel's last contact plane. Otherwise, unless
   `+0xce4` bit `0x800` is set, two vertical line tests from two points on the body give a front and
   a rear contact plane, and each wheel intersects its probe with the plane of its half
   (`wheel+0x18` against `+0x488`); a wheel without a usable plane runs its own
   `collision_test_line_vertical`. The probe runs from the hub along the down vector over the
   radius `+0x10c` (`+0x110` while bit 31 of the wheel flags is set), the travel `+0x8c` and
   `+0x1c0`.
2. **Surface.** On contact the wheel stores the contact point `+0x28`, the normal `+0x40` and the
   collision record `+0x124`, and sets flag bit `0x8`. On a new surface material (`+0xb4`, byte 3
   of the contact record) it looks up the tyre's surface entry (index `+0xb0`) in the tyre's surface
   list. A record with flag `0x10000` raises the contact latch `+0x28` (`structs/vehicle.md` §2a).
   Material 3 is the sidewalk the offence check in `car_ai_tick` looks for.
3. **Spring and damper.** With the extension `x = +0x14 − min(hub height, +0x14)` and
   `v = (+0x94 − x) / dt`, the suspension force is `+0x90 = (+0x8c − x) · +0x88`, plus `v · +0x9c`
   while the wheel extends (`v < 0`) or `v · +0x98` while it compresses and has not bottomed out.
   A negative force clears contact bit `0x8` (the wheel lifts off); `+0x94` keeps `x`.
4. **Bump stop.** At full compression, when the caller asks for it, the wheel latches bit `0x40`
   and the solver applies a rigid impulse along the contact normal from the inverse inertia
   (`+0xda4` scaled by `+0x4b4` / `+0x4b8` / `+0x4bc`) and the inverse mass `1 / +0x670`; the
   impulses of all bottomed wheels are averaged into the body. A bump stop can deform the body
   (`vehicle_deform`, `+0xce4` bit `0x8000`), damage it (`vehicle_apply_damage`, `+0xce4` bit
   `0x4000`) and play the wheel sound above 1.4.
5. **Wheel axes.** The lateral axis `+0x4c` and the longitudinal axis `+0x58` are rebuilt from the
   contact normal and the body basis, or the wheel frame's forward axis for a steered wheel
   (bit `0x1`), and turned by `+0x114` when `+0x188 > 0`.

Related wheel functions: `car_detect_wheel` (`0x0046b8a0`), `car_collision_filter_wheel`
(`0x0046e080`).

### Key physics state fields (on the vehicle entity)
| offset | meaning |
|---|---|
| `+0x24/+0x28/+0x2c` | vehicle **world position** (integrated; mirrored to `+0x40c/410/414`) |
| `+0x1f7c/+0x1f80/+0x1f84` | a velocity vector (also derived on the playback path, §4) |
| `+0x2164/68/6c` | previous-frame position (snap-back reference) |
| `+0x2170/74/78` | linear velocity (damped each frame) |
| `+0x21a8/ac/b0` | second velocity (angular) |
| `+0x21b4/b8/bc` | per-axis damping coefficients |
| `+0x21c0/c4/c8` | position accumulator |
| `+0x2198` | physics flags (bit `0x8` enable, bit `0x200` = larger snap-back radius) |
| `+0xd24` / count `+0x5b0` | **wheel ground-contact list** (stride 4); each entry `+0xb4` = surface-material id |

---

## 4. Transform record / playback (keyframe interpolation)

A separate subsystem (vtable slots #0/#2/#3/#4) records and replays transform **keyframes** — the `C_recorder` track system (`structs/recorder.md`), driven by the playback clock `g_recorder_playback_clock_ms` (`0x00656488`, field `+0x120` of the `C_recorder` singleton at `0x00656368`):
- **Record** (`car_track_record_key`): appends a `pos + quat + wheel-flags` keyframe to a buffer (`recorder_key_stream_alloc`), 50 ms-gated.
- **Playback** (`car_track_apply_pose_at_clock` / `car_track_apply_pose_at_time`): finds the two keyframes bracketing the playback clock, interpolates position + `S_quat::Slerp` orientation, **writes** the I3D_frame transform (`+0x80/84/88`, `+0x9c`) and the vehicle world pos (`+0x24/28/2c`), and **derives** velocity `+0x1f7c = (pos_now − pos_prev)/dt` — the *reverse* of integration.
- **Sample** (`car_track_get_pos_at_time`): read-only velocity/position query at an arbitrary time.

So for vehicles on this path the transform is *interpolated from recorded keyframes*, not integrated from forces. There is also a **published-transform block** at `0x00656464` (valid-flag, singleton `+0xfc`) + `0x00656468/6c/70` (pos) + `0x00656474..80` (quat), set by `actor_track_apply_pose_at_time` when the track flag `& 0x4000` is set — a shared interpolated transform other systems can read (camera / minimap).

---

## 5. `+0x20bd` — the ambient-traffic flag (NOT a police marker)

`car_set_is_traffic_car` (`0x00465a80`) is a one-line setter `*(vehicle+0x20bd) = byte`.

| value | written by |
|---|---|
| **1** | `traffic_car_game_init` (a traffic element promoted to a real simulated vehicle); `traffic_car_adopt_car_with_new_driver` |
| **0** | `traffic_car_deactivate_element`; `opponent_game_init` |

`car_ai_tick` is gated on it, and `car_save_game_save` / `car_save_game_load` round-trip it. It is set for ambient civilian traffic, so it is **not** a police marker. The police car-assault offence uses it only as a "the struck car is a live simulated vehicle" gate; the actual cop identity is decided by the `g_traffic_car_registry` scan.

## 5.1. `+0x1ff2` (u8) — IS-A-POLICE-CAR flag (the genuine police marker)

`vehicle+0x1ff2` **does** mark a police car. `car_ctor` zeroes it (default not police); `car_init` uppercases the model file name (`+0x20cc`) and prefix-matches it against the **4 police-car model strings** (null-terminated literals in `.data`):

| Symbol | VA | Content | Prefix len |
|---|---|---|---|
| `s_POLCAD` | `0x0064b79c` | `"POLCAD"` | 7 |
| `s_POLI` | `0x0064b794` | `"POLI"` | 5 |
| `s_POLIMFOR` | `0x0064b788` | `"POLIMFOR"` | 9 |
| `s_POLIMTUD` | `0x0064b77c` | `"POLIMTUD"` | 9 |

The prefix length counts the terminating NUL.

These are the police vehicle pool `poli00 / polCad00 / polimFor00 / polimTud00`. On a prefix match `+0x1ff2 = 1`.

Two consumers:
- **`police_manager_tick`** — if the suspect's linked vehicle has `+0x1ff2` set, a *pending FINE* is dropped (committed `+0x3c < 2`). **In-game (user-confirmed):** while the player drives a police car, fine-level offences are waived; handcuffs/gun escalations still apply.
- **The garage car scan** (`garage_game_done`, `garage_run_car_select_menu`) — collects kind-4 vehicles inside the garage box; a car is skipped when `+0x1ff2` is set, and separately when its frame name (`frame+0x100`) contains `s_nesahat_0064d8d4` (`0x0064d8d4`, the literal `"nesahat"`, Czech *"do not touch"*).

---

## 6. Where Game.exe ends and the engine begins

`LS3DF.dll` ("LS3D Framework", 49 exports) supplies only math primitives (`S_vector`/`S_matrix`/`S_quat`: AngleTo/Mult4X4/Normalize/Slerp/Inverse), the scene-graph node (`I3D_frame::UpdateWMatrixProc`, `I3D_visual`), and driver bootstrap. **All vehicle movement, integration, wheel/suspension solving, and collision logic is in `Game.exe`** — LS3DF only provides the matrix/quat math those functions call.

---

## 7. PC vs PS2

Slot 15's PS2 twin is `C_car::NetDirect(Ui)`, and slot 15 is `NetDirect` across the family, which
is what makes `C_airplane`'s bare `ret 4` in that slot a network stub (§2).

The local chain of §3 maps one to one: `car_ai_tick` is `C_car::AI`, `vehicle_tick_drivetrain`
`C_Vehicle::Tick`, `vehicle_substep_move` `C_Vehicle::Engine`, `vehicle_move` `C_Vehicle::Move` and
`vehicle_wheel_contact_solve` `C_Vehicle::DoWheelsCollision`. The PS2 `Engine` sub-steps by the
field `SetCutTime` writes (PS2 `+0xefc`, default 0.028 s, 42 ms on traffic cars), exactly as the
PC does with `+0xca0`.

---

## 8. Open items

Nothing is open.
