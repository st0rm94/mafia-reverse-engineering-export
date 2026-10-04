# `G_Camera` — the camera manager

**Created** 2026-09-01 22:14 +0200 · **Modified** 2026-09-30 11:44 +0200

The range `[0x005d4800, 0x005dc180]`. PC prefix **`camera_`**. PS2 class **`G_Camera`**, source
file `g_camera.cpp`.

**It is not a singleton.** `G_Camera` is the sub-object **embedded at `C_game+0x4c`**, reached
through the address accessor `game_get_camera` (`0x00425510`), which is why every method is
`__thiscall` on that address and why a search for a global camera address never finds it. The
receiver is byte-proven twice: the instruction loading ECX before the single `E8` call to
`camera_get_race_camera_at` is `lea ecx, [eax + 0x4c]`, and the police arrest cutscene calls
`game_get_camera()` immediately before `camera_lock_at_human_pair`.

**Coverage: 36 of 39 PS2 methods, every row body-diffed.** The three that are absent are
explained in §6a.

---

## 1. The translation unit

`.CRT$XCU` at `.data:0x00646224` lists `… 0x5d4720, 0x5d4750, 0x5d4780, 0x5d47a0,
0x5d47c0, 0x5d47e0, 0x5d4800, 0x5d4860, 0x5d4880, 0x5dc160, 0x5dc190, 0x5dc1d0 …`, which splits
the range into three emission blocks:

| block | span | what it is | defs |
|---|---|---|---|
| A | `[0x005d4800, 0x005d48a0)` | the last three static-init pairs of this TU — the `g_camera_dialog_actors` quartet plus two empty pairs | 8 |
| B | `[0x005d48a0, 0x005dc160)` | the `G_Camera` TU body | 38 |
| C | `[0x005dc160, …)` | the **next** TU's opening CRT pair (`crt_static_init_guard_005dc160`) | - |

**No phantom function starts.** All 46 non-CRT starts were tested for being a function's
early-exit block misread as a start (no `E8` callers, no data reference, branched into from
earlier code, no aligned padding, restore bytes instead of a prologue): all 46 are real, each
either 16-aligned with padding or carrying direct `E8` callers. The order-alignment against the
PS2 corroborates it independently (§4).

Of the 38 defs in block B, 36 are `G_Camera` methods, one is `camera_get_race_camera_at` (no PS2
counterpart, §6a), and one is `copy_range_stride_0x14` — a genuinely shared STL helper with **22
callers across at least six unrelated families**, which is why it keeps a mechanical name. Four of
those 22 callsites load `g_camera_dialog_actors_alloc`, which is the second proof of that vector's
base address.

---

## 2. Field layout

| Offset | Field | Evidence |
|---|---|---|
| `+0x00` | linked `I3D_frame` | `camera_link_to_frame` stores it and picks mode `0x11` |
| `+0x04` | the `I3D_camera` being driven | `camera_set_i3d_camera`; every pose write goes to `frame+0x80..0x88` |
| `+0x08` | the player `C_human` | `camera_set_player` |
| `+0x0c` | the bound `C_car` | written by `camera_set_car`, read by `camera_get_car` (`0x005c8ae0`); `camera_end_mode` re-shows the driver model through it ¹ |
| `+0x10` | **current mode** | the field every mode predicate tests; see §3 |
| `+0x14` | restore mode | seeded from mode-record `+0x04`; `camera_set_mode(-1)` reads it back |
| `+0x18` | mode flag word | copied from mode-record `+0x40` by `camera_begin_mode` |
| `+0x1c` | → the current `g_camera_mode_table` record's `param` block (`+0x08`) | `camera_begin_mode`, `camera_reset`; `camera_tick` reads it as `pfVar8[3]`, `[4]`, `[9]` |
| `+0x22` | bit 0 of the mode flag word | split out by `camera_begin_mode` |
| `+0x24` | camera distance (5.0f default) | `camera_reset` |
| `+0x28` | camera height (0.5f default) | `camera_reset` |
| `+0x2c` | look-around target angle, half-turns | `camera_look_around` |
| `+0x30` | look-around current angle | wrapped by ±2.0 in `camera_look_around` |
| `+0x58` | swing enabled | `camera_set_swing` |
| `+0x5c` | swing amount | `camera_set_swing`; read back by `camera_apply_desired_pos` |
| `+0x60` | swing reference `I3D_frame` | `camera_set_swing` |
| `+0x64` | **snap latch** — the next pose update CUTS instead of interpolating ² | set by `camera_set_car`, `camera_set_climbing_into_car`, `camera_tick`, `camera_snap_next_update`; cleared by `camera_pop_mode(1)` and at the end of the tick |
| `+0x6c` `+0x70` `+0x74` | **push/pop mode stack** — begin/end/cap, stride `0x10` | `camera_push_mode` / `camera_pop_mode` / `camera_game_done` |
| `+0x7c` `+0x80` `+0x84` | **race-camera vector** — begin/end/cap, stride 4 | `camera_add_race_camera` / `_remove_` / `_init_race_cameras` / `_get_race_camera_at` |
| `+0x88` | look-to target `I3D_frame` | `camera_set_look_to` |
| `+0x90` | sniper field of view | `camera_set_sniper_fov` |
| `+0x94` | free-look enabled | `camera_enable_free_look`; read by both `camera_apply_desired_pos*` |

¹ The `car_setspeed` script arm calls `game_get_camera`, reads `+0xc`, and re-binds the camera
when the value equals the car that was just reset.

² `camera_apply_desired_pos` skips its blend with `g_camera_prev_pos` / `g_camera_prev_dir`,
and `camera_tick` forces the per-mode lerp factor to `1.0`, zeroes `g_camera_stability_factor` and
the three in-car direction statics `g_camera_car_old_dir_y_mode_7_9` / `_0xa` / `_0xb` (§5), and drops the
velocity-lead term. The one-line setter `camera_snap_next_update` (`0x005ee210`) has a single call
at the end of `game_init`, so a fresh scene does not drift in from the previous mission's pose.

The mode-stack record is **`G_CameraPushPop`** and the dialogue-actor record is
**`G_CameraDialog`** — both type names come from the PS2 template arguments.

| `G_CameraPushPop` (0x10 B) | |
|---|---|
| `+0x00` | the saved mode |
| `+0x04` `+0x08` `+0x0c` | three saved `I3D_camera` parameters, read from `frame+0x140/+0x144/+0x148` and pushed back through frame vtable slots `0x50` and `0x58` |

| `G_CameraDialog` (0x14 B) | |
|---|---|
| `+0x00` | the `C_human *` speaker |
| `+0x10` | the side, 1 or 2, alternating as `camera_dialog_start` appends |

---

## 3. The mode map

Every mode value below is pinned by the function that sets or tests it, not inferred.

| Mode | Meaning | Pinned by |
|---|---|---|
| `1`..`3` | on foot | latched into `g_camera_last_onfoot_mode` by `camera_set_mode` |
| `5` | **sniper scope** | `camera_set_sniper_mode` pushes it; `camera_set_sniper_fov` only acts in it; `camera_end_mode` restores the HUD when leaving it |
| `6` | **death camera** | set by `human_death` and twice by `human_movement_intern`; `camera_tick` interpolates towards the victim's `neck` frame through `g_camera_death_interp` / `_pos` / `_dest_pos` |
| `7`..`0xf` | in a vehicle | latched into `g_camera_last_vehicle_mode`; `7` is also what `camera_set_climbing_into_car(true)` forces and what `camera_enable_free_look` restores from modes `0xc`..`0xf` |
| `0x11` | linked to a frame | set by `camera_link_to_frame`; suppresses the whole dialogue camera |
| `0x13` | **dialogue** | set by `camera_dialog_start`, tested by `camera_is_in_dialog` |
| `0x14` | race camera | set from `racing_set_race_camera_index` (§3b) |
| `0x15` | look-to | set by `camera_set_look_to` |
| `0x16` | **locked** | entered by `camera_lock_at` and `camera_lock_at_human_pair`, left by `camera_unlock` |
| `0x17` | **riding on a truck's cargo bed** | set by `human_intern_use_car_truck` when the human is the player, right after `camera_link_to_frame` binds the camera to the truck's frame; shares `camera_end_mode`'s teardown with `0x11` |
| `0x18` | **menu camera** — the owner drives the `I3D_camera` itself | set by `carcyclopedia_ctor` and `gui_menu_execute_modal`, each after `camera_push_mode`; `camera_tick` returns at once in this mode. It has no `g_camera_mode_table` record |

The table is complete: a byte scan of `.text` finds 17 calls and jumps to `camera_set_mode`, and
together with the direct stores to `+0x10` in `camera_reset`, `camera_set_car` and
`camera_set_climbing_into_car` they produce exactly the modes above.

**`g_camera_mode_table` (`0x00650450`) is the table behind all of this**: 17 records of `0x44`
bytes, scanned by `camera_begin_mode` up to `0x006508d4`. Layout, measured from the bytes:

```c
typedef struct {            /* S_camera_mode_record, 0x44 bytes */
/* +0x00 */ int   mode;
/* +0x04 */ int   restore_mode;
/* +0x08 */ float param[10];  // camera+0x1c points here
/* +0x30 */ char  name[16];   // the developers' label for the mode
/* +0x40 */ int   flags;      // bit 0 -> camera+0x22, bit 1 hides the player model,
                              // bit 2 suppresses the fade reset and the g_camera_look_from_eyes clear
} S_camera_mode_record;
```

The decomp types the `param` block as ten separate fields `param0`..`param9`, so the pointer that
`camera_begin_mode` and `camera_reset` store in `camera+0x1c` reads `&g_camera_mode_table[i].param0`.

### 3a. The 17 records

| # | mode | restore | `name` | flags |
|---|---|---|---|---|
| 0 | `0x01` | `0x00` | `P Behind` | `0x4` |
| 1 | `0x03` | `0x01` | `P In` | `0x5` |
| 2 | `0x06` | `0x00` | `P Behind` | `0x0` |
| 3 | `0x0b` | `0x0e` | `C Beh. Hellboy` | `0x0` |
| 4 | `0x07` | `0x08` | `C Beh. Hellboy` | `0x0` |
| 5 | `0x08` | `0x09` | `C Behind Emeth` | `0x0` |
| 6 | `0x09` | `0x0c` | `C Behind3` | `0x0` |
| 7 | `0x0c` | `0x0d` | `C Chase` | `0x2` |
| 8 | `0x0d` | `0x0e` | `C In` | `0x0` |
| 9 | `0x0e` | `0x07` | `C Over` | `0x0` |
| 10 | `0x11` | `0x00` | `C RailwayIn` | `0x1` |
| 11 | `0x17` | `0x00` | `P Behind` | `0x1` |
| 12 | `0x05` | `0x00` | `CP Sniper` | `0x1` |
| 13 | `0x15` | `0x00` | `C LookTo` | `0x8` |
| 14 | `0x16` | `0x00` | `C Lock` | `0x8` |
| 15 | `0x13` | `0x00` | `C Dialog` | `0x8` |
| 16 | `0x14` | `0x07` | `C Racing` | `0x0` |

The in-car records' `restore` values chain the camera-change key through the vehicle modes:
`0x07 -> 0x08 -> 0x09 -> 0x0c -> 0x0d -> 0x0e -> 0x07`.

**Dead code.** Modes `2`, `0xa` and `0x12` have arms in `camera_tick` (§5) but no record, and none
of the setters above ever produces them — the restore chain, the pushed modes and the two latches
only carry values that were set first — so those arms never run. Flag bit `0x8` (records 13..15)
has no reader: `camera_begin_mode` and `camera_tick` test bits `0x1`, `0x2` and `0x4` only, and a
scan of the whole of `.text` finds no `test` of bit `0x8` at `+0x18`.

### 3b. Race cameras in the retail game

The race-camera path runs in mission 6. Its `GameInitStart` script calls
`racing_mission6_init 5, 12, 0, 44`, and `race_mission6_init` constructs `g_racing`. The mission's
`scene2.bin` carries 26 `C_racecam` actors (kind `0x1c`). `camera_add_race_camera` has one caller,
`racecam_game_init`, which registers the actor's record only while `g_racing` is set, and
`camera_init_race_cameras` (called from `racing_game_init`) only pairs the already-registered
records with the circuit's gates.
`racing_tick` then steps the camera to the next racer with `racing_set_race_camera_index`
(mode `0x14`) whenever its dwell timer expires. `racecam_game_done` unregisters through
`camera_remove_race_camera` without testing `g_racing`, so in every other mission the removal runs
against an empty vector. **Confirmed in game:** after the race finishes, the
player watches the other racers from these trackside cameras.

---

## 4. PC ↔ PS2 method coverage — 36 of 39

Ordered by PC VA. 38 PC defs against 39 PS2 methods, forced 1:1 in every gap.

| PC VA | PC name | PS2 method | Status |
|---|---|---|---|
| 0x005d48a0 | `camera_ctor` | `__ct__8G_CameraFv` | CONFIRMED |
| 0x005d4910 | `camera_dtor` | `__dt__8G_CameraFv` | CONFIRMED |
| 0x005d4950 | `camera_reset` | `Reset__8G_CameraFv` | CONFIRMED |
| 0x005d4a30 | `camera_game_done` | `GameDone__8G_CameraFv` | CONFIRMED |
| 0x005d4a60 | `camera_set_i3d_camera` | `SetCamera__8G_CameraFP10I3D_camera` | CONFIRMED |
| 0x005d4a80 | `camera_set_player` | `SetPlayer__8G_CameraFP7C_human` | CONFIRMED |
| 0x005d4aa0 | `camera_set_car` | `SetCar__8G_CameraFP5C_car` | CONFIRMED |
| 0x005d4af0 | `camera_link_to_frame` | `Link__8G_CameraFP9I3D_frame` | CONFIRMED |
| 0x005d4b20 | `camera_set_swing` | `SetSwing__8G_CameraFbfP9I3D_frame` | CONFIRMED |
| 0x005d4c40 | `camera_look_around` | `LookAround__8G_CameraFi` | CONFIRMED |
| 0x005d4cf0 | `camera_is_look_from_eyes` | `LookFromEyes__8G_CameraFv` | CONFIRMED |
| 0x005d4d10 | `camera_set_mode` | `SetMode__8G_CameraFUib` | CONFIRMED |
| 0x005d4d70 | `camera_set_climbing_into_car` | `LezeDoToyoty__8G_CameraFb` | CONFIRMED |
| 0x005d4dd0 | `camera_tick` | `Tick__8G_CameraFUi` | CONFIRMED |
| - | - | `CalcUpDown__8G_CameraFRC8S_vector` | PS2-port-only candidate (§6a) |
| 0x005d9710 | `camera_apply_desired_pos_car` | `WantPosCar__8G_CameraFRC8S_vectorRC8S_vectorf` | CONFIRMED |
| 0x005d9b90 | `camera_apply_desired_pos` | `WantPos__8G_CameraFRC8S_vectorRC8S_vectorRC8S_vectorf` | CONFIRMED |
| 0x005da4b0 | `camera_begin_mode` | `Begin__8G_CameraFv` | CONFIRMED |
| 0x005daa60 | `camera_end_mode` | `End__8G_CameraFv` | CONFIRMED |
| 0x005daad0 | `camera_enable_free_look` | `EnableFreeLook__8G_CameraFbb` | CONFIRMED |
| 0x005dab30 | `camera_free_look` | `FreeLook__8G_CameraFff` | CONFIRMED |
| 0x005dae20 | `camera_free_look_base` | `FreeLookBase__8G_CameraFffR8S_vectorR8S_vectorR8S_vector` | CONFIRMED |
| 0x005db000 | `camera_lock_at` | `LockAt__8G_CameraFRC8S_vectorRC8S_vector` | CONFIRMED |
| 0x005db080 | `camera_lock_at_human_pair` | `CumNaNe__8G_CameraFP7C_humanP7C_human` | CONFIRMED |
| 0x005db5d0 | `camera_unlock` | `Unlock__8G_CameraFb` | CONFIRMED |
| - | - | `HardLockAt__8G_CameraFRC8S_vectorRC8S_vector` | PS2-only (§6a) |
| - | - | `HardUnlock__8G_CameraFv` | PS2-only (§6a) |
| 0x005db5f0 | `camera_push_mode` | `PushMode__8G_CameraFv` | CONFIRMED |
| 0x005db820 | `camera_pop_mode` | `PopMode__8G_CameraFb` | CONFIRMED |
| 0x005db8e0 | `camera_dialog_look` | `DialogLook__8G_CameraFP7C_humanP7C_human` | CONFIRMED |
| 0x005db970 | `camera_dialog_start` | `DialogStart__8G_CameraFPP7C_humanUi` | CONFIRMED |
| 0x005dbc50 | `camera_dialog_stop` | `DialogStop__8G_CameraFv` | CONFIRMED |
| 0x005dbc90 | `camera_is_in_dialog` | `IsInDialog__8G_CameraFv` | CONFIRMED |
| 0x005dbca0 | `camera_add_race_camera` | `AddRaceCamera__8G_CameraFP12G_RaceCamera` | CONFIRMED |
| 0x005dbe70 | `camera_remove_race_camera` | `RemoveRaceCamera__8G_CameraFP12G_RaceCamera` | CONFIRMED |
| 0x005dbed0 | `camera_init_race_cameras` | `InitRaceCameras__8G_CameraFv` | CONFIRMED |
| 0x005dc000 | `camera_set_look_to` | `SetLookTo__8G_CameraFRC8S_vectorP9I3D_frame` | CONFIRMED |
| 0x005dc070 | `camera_set_sniper_fov` | `SetSniperFov__8G_CameraFf` | CONFIRMED |
| 0x005dc0b0 | `camera_set_sniper_mode` | `SetSniperMode__8G_CameraFb` | CONFIRMED |
| 0x005dc0f0 | `camera_get_race_camera_at` | - | PC-only (§6a) |
| 0x005dc120 | `copy_range_stride_0x14` | - | shared STL helper, no owner |

### 4a. The static-init quartet (block A)

| PC VA | PC name |
|---|---|
| 0x005d4800 | `camera_dialog_actors_static_init` |
| 0x005d4810 | `camera_dialog_actors_static_ctor` |
| 0x005d4830 | `camera_dialog_actors_register_atexit_dtor` |
| 0x005d4840 | `camera_dialog_actors_static_dtor` |

Graded STRUCTURAL: they are compiler-emitted CRT scaffolding, and no PS2 counterpart *can* exist
because the PS2 emits one `__sinit_<file>.cpp` per source file rather than a per-object quartet.
The spelling is the house form the registry uses for the other static-init families.

`camera_dialog_actors_static_dtor` is handed to `atexit` **by address**, so its only reference is
an address push, not an `E8` call.

---

## 5. File-scope globals

Every one is pinned by a statement that matches on both builds.

| PC address | name | role |
|---|---|---|
| `0x00650438` | `g_camera_stability_factor` | settle weight, `+= dt*0.003` / `-= dt*0.01`, clamped to 1.0 |
| `0x00650448` | `g_camera_dialog_side` | ±1.0, which shoulder the dialogue camera sits over |
| `0x00650450` | `g_camera_mode_table` | the 17 × 0x44 mode descriptor table (§3) |
| `0x00672488` | `g_aiming_sensitivity_x` | Controls-screen slider, textdb 222 |
| `0x0067248c` | `g_aiming_sensitivity_y` | Controls-screen slider, textdb 223 |
| `0x006724ce` | `g_mouse_control_inverted` | Controls-screen toggle, textdb 257 / 255 / 256 |
| `0x006f8f38` | `g_camera_frame_time_filter_ring` | 30 records × 0xc, ends exactly at `0x006f90a0` |
| `0x006f9148` | `g_camera_frame_time_filter_cursor` | ring cursor, wraps at `0x1d` |
| `0x006f90a0` | `g_camera_death_interp` | death-camera interpolation parameter |
| `0x006f90a4` | `g_camera_death_dest_pos` | death-camera destination |
| `0x006f90b0` | `g_camera_death_pos` | death-camera current position |
| `0x006f90bc` | `g_camera_dialog_actors_alloc/_begin/_end/_cap` | the `G_CameraDialog` vector |
| `0x006f90cc` | `g_active_race_camera` | the active race camera |
| `0x006f90d0` | `g_camera_behind_pos` | third-person pose |
| `0x006f90dc` | `g_camera_eye_dir` | first-person view direction |
| `0x006f90e8` | `g_camera_eye_pos` | first-person eye position, `+1.65` standing height |
| `0x006f90fc` | `g_camera_prev_distance` | last frame's distance |
| `0x006f910c` | `g_camera_prev_dir` | last frame's direction |
| `0x006f9118` | `g_camera_prev_pos` | last frame's position |
| `0x006f9128` | `g_camera_crouch_eye_drop` | eye height is `1.65 - this` |
| `0x006f912c` | `g_camera_rot_in_act` | zeroed by `camera_begin_mode` case 0xd (head cam); no reader |
| `0x006f9124` | `g_camera_swing_phase` | advanced by `camera_tick` (`+= dt·0.3`, wraps at 2.0); `camera_apply_desired_pos` tilts the up vector by `sin(phase·π) × camera+0x5c` |
| `0x006f9130` | `g_camera_look_from_eyes` | the first-person request flag |
| `0x006f9134` | `g_camera_eye_blend` | weights behind-pose against eye-pose |
| `0x006f9138` / `0x006f913c` | `g_camera_dialog_look_from` / `_to` | the two speakers |
| `0x006f9140` / `0x006f9144` | `g_camera_free_look_yaw` / `_pitch` | free-look accumulators |
| `0x006f8f10` | `g_camera_car_sway_filtered` | `camera_tick` static `S_vector`, modes 0xc/0xd/0xe |
| `0x006f8f1c` | `g_camera_car_prev_velocity` | `camera_tick` static `S_vector`, a copy of `C_car+0x1f7c` |
| `0x006f8f28` | `g_camera_tick_static_dtor_guard` | bit 0 / bit 1 guard registering the atexit destructors of the two vectors above |
| `0x006f8f34` | `g_camera_car_old_dir_y_mode_7_9` | modes 7/8/9: look-direction Y, smoothed through `g_camera_stability_factor` |
| `0x006f8f30` | `g_camera_car_old_dir_y_mode_0xa` | mode 0xa: the same Y, copied unsmoothed; the arm never runs (§3a) |
| `0x006f8f2c` | `g_camera_car_old_dir_y_mode_0xb` | mode 0xb: smoothed like the mode 7/8/9 value |
| `0x006f9158` | `g_camera_dir_yaw` | mode 2: integrates the filtered mouse X × `g_aiming_sensitivity_x`, wrapped to [0, 2π); the arm never runs (§3a) |
| `0x006f915c` | `g_camera_dir_pitch` | mode 2: pitch as a fraction of π, clamped to 0.01..0.99; the arm never runs (§3a) |
| `0x006f9150` | `g_camera_sniper_sway_phase_pitch` | mode 5: `+= dt·2.5`, wrapped; its sine is added to the view's vertical component |
| `0x006f9154` | `g_camera_sniper_sway_phase_yaw` | mode 5: `+= dt·3.0`, wrapped; its sine offsets the eye horizontally |
| `0x006f914c` | `g_camera_orbit_yaw` | mode 0x12: `+= dt·0.0002`, wrapped; the camera circles the target 2 m up; the arm never runs (§3a) |
| `0x006724a4` / `0x006724a8` | `g_camera_last_onfoot_mode` / `_last_vehicle_mode` | the last on-foot / vehicle mode (§3) |

`DAT_005d96f0` / `DAT_005d9700` are **not globals**: they are code inside `.text`, the atexit
destructors of the two `camera_tick` static vectors, passed to `crt_atexit` by address under
`g_camera_tick_static_dtor_guard`.

---

## 6. PC vs PS2

### 6a. The three PS2 methods with no PC counterpart

Each is a measurement, not a gap.

**`CalcUpDown__8G_CameraFRC8S_vector` — PS2-port-only candidate.** Its *only* PS2 caller is
`AI_walk__8C_playerFUiP17G_ConsoleControls`. The PC twin of that caller, `player_ai_walk`
(`0x004c6ed0`), carries the **same `[0.1, 0.85]` clamp at the same point** — but fed from
`g_aiming_sensitivity_y × mouse_delta` instead of from the aim frame's geometry. The PS2 computes
an aim pitch from the crosshair frame because it is driving a gamepad; the PC reads the mouse. So
the computation does not exist on the PC and this is **not** an inlining. Graded candidate rather
than terminal, because its caller is not itself a PS2-only subsystem.

**`HardLockAt__8G_CameraFRC8S_vectorRC8S_vector` and `HardUnlock__8G_CameraFv` — PS2-only.** Not
just the two methods: **their consumer is absent too.** PS2 `Tick` opens a branch on `camera+0xa5`
that copies the two hard-lock vectors from `+0xb0` and `+0xc0`, and PS2 `SetMode` gates its entire
body on that same `+0xa5`. The PC's `camera_tick` contains no reference to the corresponding
`+0x95`, and `camera_set_mode` has no such gate. A feature whose flag, whose setters and whose
only reader are all absent was never in the PC source.

**`camera_get_race_camera_at` is the mirror case — a PC function with no PS2 method.** The PS2
gutted per-class networking (19 of its 21 `NetDirect` bodies are 8-byte stubs), and this
accessor's single consumer is `net_session_partition_peers_by_relevance`. Here the PS2 method
count is a **lower** bound.

### 6b. Behaviour differences

**The PC ships three more camera modes.** `camera_begin_mode` scans `g_camera_mode_table` to
`0x006508d4`, i.e. **17** records of `0x44`; the PS2's `Begin` loops `while (iVar6 < 0xe)`, i.e.
**14**. ✅ Confirmed in game by the user.

**Look-around is unrestricted on the PC.** PS2 `LookAround` opens with
`strcasecmp(<a mission name>, act_mission+0x5c)` and forces the selector to 0 on a match — a
per-mission kill switch. The PC has no such test. ✅ Confirmed in game by the user. The two builds
also disagree on units and on the case numbering: PC half-turns with a ±2.0 short-way wrap and
cases `{0, 1, 2, -1}`, PS2 radians with no wrap and cases `{0, 1, 2, 3}`, mapping PC `{0,1,2,-1}`
onto PS2 `{0,2,3,1}`.

**Mode defaults.** Where the PS2 hard-codes mode `1`, the PC restores
`g_camera_last_onfoot_mode` — in `camera_reset`, `camera_set_car` and `camera_link_to_frame`
alike.

**Free-look input.** PS2 `FreeLookBase` reads its two deltas from two globals; the PC reads them
from `input_get_action_value(0x13)` and `(0x12)` — action slots labelled textdb 312 "Aiming" —
and scales them by `g_aiming_sensitivity_x` / `_y`, with `g_mouse_control_inverted` flipping the
pitch. None of that exists on the PS2.

**The graphics interface.** PC `camera_tick` reads `g_engine`'s vtable slots `0xd0` and `0xd4` to
fill the frame-time ring; the PS2 reads `igraph`'s `0xf0` and `0xf4` at the same statements.

### 6c. Layout differences

**Field-offset deltas.** `+0x10` mode is common. Above `+0x58` the PS2 runs `0x18` higher
(`+0x58/+0x5c/+0x60/+0x64` ↔ `+0x70/+0x74/+0x78/+0x7c`), and above `+0x6c` it runs `0x14` higher
(`+0x6c..+0x74` ↔ `+0x80..+0x88`, `+0x7c..+0x84` ↔ `+0x8c..+0x94`, `+0x88` ↔ `+0x98`,
`+0x90` ↔ `+0xa0`, `+0x94` ↔ `+0xa4`). The delta is **not constant** — do not rebase one offset
from another.

**`+0x0c` is byte-proven across builds** by the `car_setspeed` script arm: the PC reads
`camera+0xc`, the PS2 reads `*(int *)(game + 0x7c)`, and the PS2 camera is `C_game+0x70`, so
`game+0x7c` IS `camera+0xc`.

**`G_CameraDialog`** is `0x14` bytes on the PC and `0x30` on the PS2.

**Globals.** `g_camera_swing_phase` is PS2 `naklon` (twin CONFIRMED); `g_camera_tick_static_dtor_guard` is a
packed flag word (bits 0 and 1) where the PS2 keeps separate `init$NNNN` statics. The `camera_tick`
statics are PS2 function-local statics of `Tick`, each matched by case label and statement:

| PC name | PS2 static | matched by |
|---|---|---|
| `g_camera_car_old_dir_y_mode_0xb` | `oldDirY$2455` | case `0xb`, smoothed with `faktorStability` |
| `g_camera_car_old_dir_y_mode_0xa` | `oldDirY$2443` | case `10`, plain copy |
| `g_camera_car_old_dir_y_mode_7_9` | `oldDirY$2417` | cases `7`/`8`/`9` |
| `g_camera_orbit_yaw` | `rot$2351` | mode `0x12`, `+= dt·0.0002`, 2π wrap |
| `g_camera_sniper_sway_phase_pitch` | `kl$2564` | mode 5, `+= dt·2.5` |
| `g_camera_sniper_sway_phase_yaw` | `kl2$2567` | mode 5, `+= dt·3.0` |
| `g_camera_dir_yaw` | `cameraDirRot$2605` | 2π wrap |
| `g_camera_dir_pitch` | `cameraDirUpDown$2608` | the same 0.01..0.99 clamp |

### 6d. `ParheliaSetFov` and `S_auto_hide_machine`

**`ParheliaSetFov__17G_IndicatorsClassFP10I3D_cameraf` — PS2-only.** It has three PS2 callsites,
and at every one the PC calls the `I3D_camera`'s own FOV vtable slot `0x50` directly with the same
argument: `CumNaNe` → `camera_lock_at_human_pair` (π/4 = `0x3f490fdb`), `SetSniperFov` →
`camera_set_sniper_fov`, and `GM_Carcyclopedia::OnCreate` → `carcyclopedia_ctor`
(π/3 = `0x3f860a92`). The PS2 wrapper adjusts the field of view and the aspect ratio for Matrox
Parhelia multi-head output (`structs/hud.md` §4b); the PC has no such branch at any caller.

**`Tick`'s `S_auto_hide_machine` — PS2-only.** PS2 `Tick` calls
`Tick__19S_auto_hide_machineFUi(*(act_mission+0x20) + 0x1578, 100000)`, a `C_game` member whose area
data `C_mission::OpenBin` loads from `scene2.bin` chunk `0xae52`. The PC `mission_open_bin` has no
case for chunk `0xae52` (its arms above `0xae50` are `0xaef0` and `0xaef1`), so the subsystem has no
data path on the PC, and `camera_tick` calls nothing in its place.

### 6e. What MSVC inlined that the PS2 calls out of line

Marked in the decomp with `PS2-INLINED HERE` / `PS2 CALLS A REAL FUNCTION HERE` banners.

| PC function | the real PS2 call it absorbed |
|---|---|
| `camera_ctor` | two `compressed_pair_imp<>` constructors **and** `reserve__std::__vector_imp<>(3)` + `reserve__std::__vector_pod<>(15)` — the PC emits no reserve at all and starts both vectors empty |
| `camera_dtor` | `__dt__std::__vector_pod<>` and `__dt__std::__vector_deleter<G_CameraPushPop>` |
| `camera_push_mode` | `insert__std::__vector_imp<G_CameraPushPop,…>` (1180 B) — the whole grow path, which is the 549 B vs 104 B difference |
| `camera_pop_mode` | `first__compressed_pair_imp<>` |
| `camera_dialog_start` | `reserve__gif::vector<G_CameraDialog,8>` (348 B) |
| `camera_add_race_camera` | `data__std::__vector_pod<>` **and** `insert__std::__vector_pod<>` — the 462 B vs 136 B difference |
| `camera_lock_at_human_pair`, `camera_set_sniper_fov` | `ParheliaSetFov__17G_IndicatorsClass` → the `I3D_camera` FOV slot |

`copy_range_stride_0x14`, `fill_n_stride_0x14`, `vec_count_stride_0x14`, `stl_copy_dwords`,
`stl_fill_n_dwords`, `stl_construct_dword` and `stub_noop_args8_00575c00` are **not** inlinings
and **not** camera members — they are the shared STL primitives MSVC still calls out to, with
callers spread across the image. Naming any of them `camera_*` would be a false ownership claim.

---

## 7. Related

- `structs/game.md` §2 — the `+0x4c` field and `game_get_camera`.
- `structs/racecam.md` — `C_racecam`, whose `+0x70` record is the `G_RaceCamera`.
- `structs/hud.md` — `g_hud_flags` and the `ParheliaSetFov` row.
- `structs/input.md` — the action slots `0x12`/`0x13` the free-look path reads.
- `basic/artifacts/racing.md` — the racing mode; mission 6 is the retail path into `C_racing` (§3b).
- `structs/carcyclopedia.md` — the screen that uses menu mode `0x18`.

---

## 8. Open items

* **In game:** the sniper scope is mode 5, and entering/leaving it flips two `g_hud_flags` bits
  (`0x20000` on entry, `0x20` on exit) — confirm the HUD really disappears under the scope and
  comes back.
* **In game:** `camera_set_swing` arms a tilt driven by `sin(g_camera_swing_phase · π)`. What
  triggers the swing, and on which vehicles, is undecoded.
