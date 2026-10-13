# `C_driver` — the chase AI-driver record (~0x29C bytes)

**Created** 2026-07-11 13:22 +0200 · **Modified** 2026-10-11 21:37 +0200

One `C_driver` is the per-car AI that physically **steers / throttles / brakes** a chase car along a
cached path of nav-graph nodes. It is not police-specific: police pursuits, AI combat chases and
scripted mission chases all drive their cars through this object.

| | |
|---|---|
| PS2 class | `C_driver` (`8C_driver`) |
| PC prefix | `driver_` |
| Size | **~0x29C** |
| Instance location | allocated per chase via `operator_new(0x29C)` + `driver_ctor` in `pursuit_event_dispatch_pursuit_vehicle` / `pursuit_event_dispatch_escape_vehicle`, stored at the pursuit **S_ELEMENT `record+0x04`** (`structs/pursuit-event-manager.md`) |
| Driven vehicle | `this+0x10` |
| Method family | 6 core + 17 driving-AI helpers, all separate PC functions, all CONFIRMED PS2 twins (§4) |

Reconstructed from `Game.exe` disassembly and annotated-decomp static analysis, body-diffed
against the non-stripped PS2 ELF.

**Confidence tags:** **[H]** body-read + writer + reader; **[M]** body-read, semantics partly
inferred; **[U]** not observed. PC names track the PS2 method names (drop `C_`, lowercase
`driver_` prefix); the word "frame" is avoided in names (it denotes the scene2.bin / I3D_frame
concept).

## 1. Field layout (~0x29C bytes)

Offsets relative to the `C_driver` `this`. `car` = the driven `C_car` at `this+0x10`; its
`C_Vehicle` sub-object is `car+0x70` (`structs/vehicle.md`). `node` = a path-node element of the `this+0x164` vector (stride 0x70, §3).

```c
typedef struct C_driver {                 // ~0x29C bytes; pursuit S_ELEMENT record+0x04
    /* +0x04 */ vec_node* motion_begin;         // [H] std::vector<node> (stride 0x70) swept motion samples
    /* +0x08 */ vec_node* motion_end;           // [H]   built by driver_predict_collision; ctor zeroes, dtor frees
    /* +0x0C */ vec_node* motion_cap;           // [H]
    /* +0x10 */ C_car*    car;                  // [H] the driven car (driver_set_car); C_Vehicle at +0x70
    /* +0x14 */ float     coef_040;             // [M] 0.4f ctor; accel/steer gain (with +0xec)
    /* +0x18 */ float     coef_070;             // [M] 0.7f ctor; accel/steer gain (with +0xec)
    /* +0x1C */ float     box_corner_a[3];      // [H] +0x1C..+0x24 collision half-box corner A
    /* +0x28 */ float     box_corner_b[3];      // [H] +0x28..+0x30 collision half-box corner B
    /* +0x34 */ float     collision_radius;     // [H] min(vehicle size)*0.7 (driver_set_car)
    /* +0x38 */ float     brake_onset_time;     // [H] time-to-corner (driver_compute_brake_onset / driver_compute_corner_speed_limit)
    /* +0x3C */ float     brake_onset_dist;     // [H] distance-to-corner
    /* +0x40 */ float     collision_hit[3];     // [M] +0x40..+0x48 predicted collision point (actor_is_possible_coll_all out)
    /* +0x4C */ float     avoid_dir[3];         // [M] +0x4C..+0x54 obstacle-avoidance direction
    /* +0x58 */ float     obstacle_seg[3];      // [M] +0x58..+0x60 obstacle segment endpoint
    /* +0x64 */ uint32_t  path_node_count;      // [H] (end-begin)/0x70, recomputed each tick
    /* +0x68 */ uint32_t  next_node_cursor;     // [H] cursor+1 (wraps); NODE INDEX (not an I3D_frame)
    /* +0x6C */ float     target_pos[3];        // [H] +0x6C..+0x74 current target-node world position
    /* +0x78 */ uint32_t  committed_cursor;     // [M] cursor snapshot after path recompile/erase
    /* +0x7C */ float     scale_a;              // [H] 1.0f steer-blend scale
    /* +0x80 */ float     scale_b;              // [H] 1.0f steer-blend scale (reverse)
    /* +0x84 */ float     speed_factor;         // [H] THROTTLE SCALE = g_police_speed_factor (reset=1.0); boost hook
    /* +0x88 */ float     dir_to_target[3];     // [H] +0x88..+0x90 direction target - car pos
    /* +0xA0 */ float     car_basis_x[3];       // [M] +0xA0..+0xA8 cached car right/basis vector
    /* +0xAC */ float     car_forward_v[3];     // [M] +0xAC..+0xB4 cached car forward vector
    /* +0xB8 */ float     car_up_v[3];          // [M] +0xB8..+0xC0 cached car up vector
    /* +0xC4 */ float     car_pos[3];           // [H] +0xC4..+0xCC cached car world position
    /* +0xD0 */ float     steer_lookat[3];      // [M] +0xD0..+0xD8 steering look-at vector (driver_turn)
    /* +0xDC */ float     brake_out;            // [H] BRAKE PEDAL [0,1] (driver_compute_brake_pedal); → vehicle_set_brake + brake light
    /* +0xE0 */ float     avoid_point[3];       // [M] +0xE0..+0xE8 predicted avoidance point (driver_reduce_speed_in_corner)
    /* +0xEC */ float     accel_limit;          // [M] derived accel/curvature limit (driver_fill_start_values)
    /* +0xF0 */ float     brake_gain;           // [M] +0x18 * accel_limit
    /* +0xF4 */ float     accel_gain;           // [M] +0x14 * accel_limit
    /* +0xF8 */ float     brake_divisor;        // [M] car speed-accel; divides brake_out
    /* +0xFC */ float     accel_cap;            // [M] GetForwardForceMax-derived
    /* +0x100 */ float    side_cap;             // [M] GetSideForceMax-derived
    /* +0x104 */ float    throttle_scale;       // [H] 0..1; gas pedal = speed_factor(+0x84) * this
    /* +0x108 */ uint32_t lookahead_node;       // [H] look-ahead node index (driver_last_route_in_brake_path)
    /* +0x110 */ float    prev_dir[3];          // [M] +0x110..+0x11C previous-frame steer dir (driver_predictive_sphere_lane_cross reads)
    /* +0x124 */ float    seed_pos[3];          // [H] +0x124..+0x12C seed target (driver_reset: 1st node)
    /* +0x130 */ int16_t  blink_l_table[..];    // [M] +0x130..+0x148 left/brake indicator random timing table (driver_do_lights)
    /* +0x14C */ int16_t  blink_r_table[..];    // [M] +0x14C..+0x15C engine-sound timing table (driver_do_sounds)
    /* +0x164 */ node*    path_begin;           // [H] std::vector<node> chase path (stride 0x70, §3)
    /* +0x168 */ node*    path_end;             // [H]
    /* +0x16C */ node*    path_cap;             // [H]
    /* +0x170 */ uint32_t path_cursor;          // [H] current path-node index
    /* +0x174 */ uint32_t prev_node;            // [H] cursor-1 (wraps)
    /* +0x178 */ uint8_t  near_end_flag;        // [M] within 2 nodes of path end
    /* +0x17C */ float    lookahead_dist;       // [H] 17.0f ctor; = node_width*jitter; → vehicle_set_speed_limit in tail
    /* +0x19C */ uint32_t flags;                // [H] bit0 reverse-req, bit2(0x4) overshoot, bit3(0x8) stop,
                                                //     bit4(0x10) path-recompiled, 0x100/0x200 lane side, sign=escape
    /* +0x1A0 */ void*    cur_obstacle;         // [H] current predicted obstacle entity
    /* +0x1B0 */ void*    owned_buf;            // [H] +0x1B0..+0x1B8 owned buffer (dtor frees)
    /* +0x1C0 */ float    target_speed;         // [H] cornering speed cap (driver_compute_corner_speed_limit / driver_reduce_speed_in_corner)
    /* +0x1C8 */ float    speed_jitter;         // [M] 1.0f ctor; randomized 0.65..1.0 per period
    /* +0x1CC */ float    jitter_period;        // [M] 5000.0f ctor
    /* +0x1D4 */ float    node_reach_dist;      // [M] 30.0f default; randomized 20..40 per node (driver_advance_checkpoint)
    /* +0x1D8 */ uint8_t  mode_flags;           // [M] &1 / &2 driving sub-mode bits
    /* +0x1DC */ uint32_t state_flags;          // [H] 0x2, 0x1000, 0x10000000 (path-recompiled) masks
    /* +0x1E0 */ float    steer_out;            // [H] steering output; → vehicle_set_steer
    /* +0x1E4 */ uint32_t signal_flags;         // [H] 0x100 horn, 0x1000000 L-indicator armed, 0x2000000 sound armed, &3 lane
    /* +0x1FC */ C_actor* target_actor;         // [H] pursued actor (set at dispatch)
    /* +0x200 */ C_Vehicle* target_vehicle;     // [H] target's vehicle (set at dispatch)
    /* +0x204 */ void*    excluded_obstacle;    // [M] obstacle to ignore (self/target)
    /* +0x208 */ uint32_t drive_mode;           // [H] 0 free chase, 1 follow actor, 2 follow vehicle (1 at dispatch)
    /* +0x20C */ uint32_t stuck_timer;          // [H] += dt; = 0 on a collision-grid hit
    /* +0x210 */ float    brake_lookahead_dist; // [M] ramped look-ahead braking DISTANCE, not speed (driver_ramp_brake_distance)
    /* +0x214 */ uint8_t  interp_enable;        // [M] 1 ctor; enables node-interp path smoothing
    /* +0x21C */ uint32_t run_timer;            // [H] += dt each tick (gates gas: run_timer<1 => gas 0)
    /* +0x224 */ int32_t  committed_node;       // [H] -1 sentinel (ctor + driver_set_car)
} C_driver;                                     // ~0x29C bytes
```

## 2. Field evidence (key fields)

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x10 | car (C_car*) | H | `driver_set_car` | ALL | driven car; base of every `car+off`. `driver_tick` passes it straight to `car_get_ai_probe_point` and adds `0x70` before every `vehicle_*` call (`vehicle_set_gear`, `vehicle_set_handbrake`) |
| +0x64 | path_node_count | H | `driver_tick` | all helpers | (end-begin)/0x70 |
| +0x68 | next_node_cursor | H | `driver_fill_start_values` (=cursor+1) | `driver_optimize_next_checkpoints` | **int cursor, not an I3D_frame** |
| +0x6C..+0x74 | target_pos | H | fill/advance/path_correction | steering/braking helpers | current target-node pos |
| +0x84 | speed_factor | H | dispatch (`= g_police_speed_factor`); `driver_reset`(1.0) | `driver_tick` tail (gas = +0x84*+0x104) | throttle scale; boost hook |
| +0xDC | brake_out | H | `driver_compute_brake_pedal`; `driver_tick`(0) | `driver_tick` tail → `vehicle_set_brake`; `driver_do_lights` (brake light) | **brake pedal [0,1]**, not a throttle limit |
| +0x104 | throttle_scale | H | `driver_tick`(1.0); helpers zero when blocked | `driver_tick` tail (gas = +0x84*this) | 0..1 |
| +0x108 | lookahead_node | H | `driver_last_route_in_brake_path` | `driver_compute_corner_speed_limit`, `driver_fill_start_values` | - |
| +0x164/68/6C | path-node vector | H | `pursuit_event_prepare_path`/`_recompile_path`; `driver_tick` | all helpers | stride 0x70 (§3) |
| +0x170/+0x174 | path_cursor / prev_node | H | `driver_advance_checkpoint`, `driver_fill_start_values`, dispatch(0) | all helpers | - |
| +0x17C | lookahead_dist | H | `driver_ctor`(17.0), `driver_fill_start_values` | `driver_tick` tail → `vehicle_set_speed_limit` | - |
| +0x19C | flags | H | many helpers | many helpers | bit2=overshoot, bit4=recompiled, bit0=reverse-req |
| +0x1C0 | target_speed | H | `driver_compute_corner_speed_limit`, `driver_reduce_speed_in_corner` | `driver_compute_brake_pedal` | cornering speed cap |
| +0x1E0 | steer_out | H | `driver_turn` | `driver_tick` tail → `vehicle_set_steer` | steering angle |
| +0x208 | drive_mode | H | dispatch(1) | tick main switch (0/1/2) | 0 free / 1 actor / 2 vehicle |
| +0x21C | run_timer | H | `driver_tick` (+=dt) | `driver_tick` tail (run_timer<1 => gas 0) | - |
| +0x224 | committed_node | H | `driver_ctor`(-1), `driver_set_car`(-1) | `driver_tick`, `driver_advance_checkpoint` | -1 sentinel |

## 3. Path-node sub-struct (stride 0x70; `S_car_node`)

**Racing-side producer.** `opponent_fill_path_node` builds one of these from scratch, so its writes
are an unambiguous field list: `+0x00` position (vec3) · `+0x30` distance from a reference point ·
`+0x44` byte = 0 · `+0x45` byte = 1 · `+0x46` byte = 1 · `+0x48` float = the node radius (the
racing AI passes **1000.0**) · `+0x4c` float = 0.5 · `+0x54` byte = 1. The whole 0x70 is zeroed
first. `C_racing`'s overtaking planner and `opponent_create_driver_path` are the two producers on
the racing side; see `structs/racing.md` §9e.

| Off | Field | Conf | Notes |
|---|---|---|---|
| +0x00.. | sub-point vec3 slots | H | candidate world positions; active one chosen by entry byte +0x44 |
| +0x30 + i*4 | per-sub-point segment lengths | M | float array; summed for look-ahead (`driver_last_route_in_brake_path`) |
| +0x44 | entry sub-point index | H | byte; selects the node's entry vec3. The steering helpers also read the NEXT node's byte (`i*0x70 + 0xb4` = `node[i+1]+0x44`) to pick the exit direction |
| +0x48 | node width / segment length | H | float; `lookahead_dist = width * speed_jitter` |
| +0x4C | corner speed gate | H | `driver_advance_checkpoint` / `driver_compute_brake_pedal` compare speed vs this |
| +0x50 | turn-radius cap | H | `driver_compute_radius`: `clamp_len*tan(angle/2)`; `0x497423f0`=straight |
| +0x58/+0x5C | nav marker ptrs | M | char* one-way/lane markers, tested for `&0x300` lane side |
| +0x60 | nav node/edge index | H | ushort; high bit 0x8000 = edge |
| +0x61 | node flag byte | M | `&0x80` gates a path-advance branch |
| +0x68 | traffic-light phase pointer | H | `int *`, §3a |

### 3a. `driver_path_node_at` and node `+0x68`

**`driver_path_node_at` (0x004f57e0)** is the accessor for the vector above:
`return *(this + 0x04) + i * 0x70`, invoked on the vector object at `C_driver+0x160`. It lives in
the `C_sledovacka` translation unit as an out-of-line inline member: 2 callsites, BOTH
`lea ecx,[eax+0x160]`, 0 datarefs, and the index passed is `+0x170`, the path cursor. Its
erase/insert siblings in the same TU are shared across modules and keep generic names
(`std_vector70_erase_range`, `std_vector70_insert_n`); each was measured separately.

**Path node `+0x68` points at a traffic-light phase.** Three consumers read it the same way:

```
driver_tick                : (node->+0x68 != NULL) && ((&g_driver_semaphore_states)[**(int**)(node+0x68) * 4] & 4) == 0
driver_advance_checkpoint  : the same expression, verbatim
pursuit_event_tick         : (node->+0x68 == NULL) || ((&g_pursuit_event_semaphore_states)[**(int**)(node+0x68) * 4] & 4) != 0
```

The table addresses hold **byte-identical** 10-dword data — `[1,1,1,1,1,4,4,4,4,2]` — i.e. one
file-local `static const` array emitted once per translation unit. Four copies exist:
`g_driver_semaphore_states` (the `C_driver` file), `g_pursuit_event_semaphore_states` (the
`C_sledovacka` file), `g_railway_semaphore_states` and `g_game_semaphore_states`. The index is the
`int` that `+0x68` points at, a **traffic-light phase 0..9**, and the entry's bits are the three bulbs
`game_tick_traffic_signals` drives: bit `1` red, bit `2` orange, bit `4` green (`structs/game.md` §4d).
Phases 0..4 are red, 5..8 green and 9 orange.

**The two writers** are the path builders `traffic_car_activate_elements` and
`pursuit_event_recompile_path`. Both point `+0x68` at one of three phase variables:

| target | chosen when |
|---|---|
| `C_game+0x19c` | the node's X offset to its neighbours is larger than its Z offset |
| `C_game+0x1a0` | otherwise, the Z offset being at least as large |
| a bridge actor's `act_state` (`+0x138`) | the next node is a bridge slot of `g_traffic_bridge_slots_begin` ¹ |

¹ The `C_traffic_car::S_BRIDGE` vector, `{actor *, ushort node}`; the match is on the next node's
nav index.

The two `C_game` phases are one city-wide light cycle seen from the two crossing axes: `game_init`
seeds them 0 and 5 and `game_tick` advances both together, so one axis is green while the other is
red. A bridge's `act_state` is 1 while it cycles (red) and 7 at rest (green), `structs/bridge.md`.

**Only bit `4` (green) is ever tested here:**

* **not green** → `driver_tick` forces `+0x1c0` (the cornering speed cap) to **0** once the node is
  inside the braking distance, and `driver_advance_checkpoint` zeroes the throttle `+0x104`: the car
  stops at the node.
* **`pursuit_event_tick`'s stuck-driver timer** (latching `driver+0x298` after 2 s with no predicted
  obstacle) runs on to **15 s** when an obstacle is predicted and the node has no light or a green
  one. At a red or orange light the timer is held, so a car waiting at the light is never reported
  blocked.

The symbols `g_driver_semaphore_states` / `g_pursuit_event_semaphore_states` are the **array bases** (element [0]). Every reference
is `test byte [reg*4 + <symbol>], 4` (`f6 04 85 2c c2 63 00 04`), so the index scales from the
symbol itself; the dword before each table (`0x0063c228` = `0x00594700`, `0x0063cd30` =
`0x0060b1a0`) is unrelated data with no reference.

## 4. `C_driver` method family — PS2 twins

`this` = a `C_driver`. Every twin is **CONFIRMED** by PS2↔PC body-diff (shared constants /
field-role map / call structure). `driver_tick` calls the §4b helpers in PS2 `Tick` order.

### 4a. Core lifecycle / tick

| VA | PC name | PS2 twin | Status | Role |
|---|---|---|---|---|
| 0x00482c00 | `driver_ctor` | `__ct__8C_driverFv` | CONFIRMED | ctor: zero lists + float defaults |
| 0x00482d50 | `driver_dtor` | `__dt__8C_driverFv` | CONFIRMED | dtor: free +0x1b0/+0x164/+0x04 |
| 0x00482de0 | `driver_compute_radius` | `ComputeRadius__8C_driverFii` | CONFIRMED | per-node turn cap `node+0x50` |
| 0x00482ff0 | `driver_reset` | `Reset__8C_driverFv` | CONFIRMED | per-run reset + rand jitter |
| 0x004831c0 | `driver_set_car` | `SetCar__8C_driverFP5C_car` | CONFIRMED | bind car + collision box + SetGear(1) |
| 0x00483360 | `driver_tick` | `Tick__8C_driverFUi` | CONFIRMED | monolith: calls §4b helpers, applies controls |

### 4b. Driving-AI helpers (called by `driver_tick`; separate functions, not inlined)

| VA | PC name | PS2 twin | Status | Role (body-verified) |
|---|---|---|---|---|
| 0x00485ec0 | `driver_fill_start_values` | `FillStartValues__8C_driverFib` | CONFIRMED | Per-frame setup: cursors, speed-jitter, look-ahead, ComputeRadius, car matrix, target+dir ¹ |
| 0x004861a0 | `driver_optimize_next_checkpoints` | `OptimizeNextCheckpoints__8C_driverFv` | CONFIRMED | Set flag `0x4` when `dir·next<0`; AngleTo>0.7 & dist²<900 corner-cut |
| 0x004863f0 | `driver_advance_checkpoint` | `NevimJakToNazvat__8C_driverFv` | CONFIRMED | Advance cursor past reached/blocked node; random reach dist (rand%0x28+0x14)/100, 25.0 clamp |
| 0x00486760 | `driver_compute_brake_onset` | `TakNaCemToUbrzdim__8C_driverFv` | CONFIRMED | `sinf(angle)·g·10` → arrival time/dist +0x38/+0x3c; `0x4e6e6b28` straight-sentinel |
| 0x00486830 | `driver_last_route_in_brake_path_tol` | `LastRouteInBrakePathPlusToleranciaLaskaASvoboda__8C_driverFf` | CONFIRMED | Accumulate node distances until > arrival_dist + tolerance → +0x1bc/+0x204 |
| 0x004869e0 | `driver_ramp_brake_distance` | `MinSOld__8C_driverFf` | CONFIRMED | Ramp look-ahead dist toward arrival_dist (accel 5/s, decel 15/s) |
| 0x00486a60 | `driver_last_route_in_brake_path` | `LastRouteInBrakePath__8C_driverFv` | CONFIRMED | Walk nodes accumulating segment lengths until > ramp → look-ahead node +0x108; near-end flag |
| 0x00486b80 | `driver_compute_corner_speed_limit` | `MinV__8C_driverFv` | CONFIRMED | Min over look-ahead of `sqrt(force·turn_cap/g)` → target speed +0x1c0; 1000.0 init, 5.0 floor |
| 0x00486d70 | `driver_path_correction` | `PathCorrection__8C_driverFv` | CONFIRMED | Nudge target back onto path / toward seed (mode-scaled 3/6, ·11/22) |
| 0x004873d0 | `driver_lane_change_and_park` | `PrejezdZPruhuDoPruhuAParkovani__8C_driverFv` | CONFIRMED | Lane-change/park avoidance (node-index change + edge `0x8000` + `actor_is_possible_coll_all`); can force reverse |
| 0x00487a70 | `driver_predictive_sphere_lane_cross` | `PredvidaciKoulePripPrejezduMeziPruhy__8C_driverFv` | CONFIRMED | Project car onto prev segment (guard flag `&2`), write lateral avoidance target to car+0x20c0 |
| 0x00487f40 | `driver_predict_collision` | `PredvidaciKolize__8C_driverFf` | CONFIRMED | Swept-path build + time-to-collision quadratic vs moving obstacle (largest helper; float arg) |
| 0x00488e20 | `driver_reduce_speed_in_corner` | `SnizRychlostVzatacce__8C_driverFv` | CONFIRMED | Curve-radius stop check: `sqrt(force·dist·0.85/g)` vs speed → target speed +0x1c0; returns "must brake" |
| 0x004891e0 | `driver_compute_brake_pedal` | `UtekloPraseZase__8C_driverFv` | CONFIRMED | Overspeed → brake pedal +0xDC (stopping-dist formula, clamp to +0xec/+0xf4, ÷+0xf8) |
| 0x004893e0 | `driver_turn` | `Zatoc__8C_driverFv` | CONFIRMED | Steering angle +0x1e0 (Cross look-at, clamp car max-steer); calls SetClutch/SetHandbrake/SetGear |
| 0x00489950 | `driver_do_lights` | `DoLights__8C_driverFUif` | CONFIRMED | Brake/indicator/hazard light bits on car+0x20c, driven by brake arg + blink table |
| 0x00489c00 | `driver_do_sounds` | `DoSounds__8C_driverFUi` | CONFIRMED | Engine/siren sound-timing → car+0x51c from the +0x14c timing table |

¹ Shared constants rand%100/100·0.35+0.65, 0.9 scaling, 1000.0 branch.

### 4c. The `S_car_node` container helpers (not `C_driver` methods)

`C_driver` is non-polymorphic on this build, so it contributes no vtable. Its code sits
immediately below the CRT static-init pair at `0x0048a0a0`, followed by the MSVC container
helpers for the path-node vector at `driver+0x164`/`+0x168`/`+0x16c`, stride **0x70**
(`S_car_node`):

| PC VA | PC name | role |
|---|---|---|
| `0x00489cc0` | `std_vector70_size` | `(end - begin) / 0x70` |
| `0x00489cf0` | `std_vector70_insert_n` | `insert(pos, n, val)` |
| `0x0048a010` | `std_vector70_uninit_copy` | `uninitialized_copy` over a range |
| `0x0048a050` | `std_vector70_uninit_fill_n` | `uninitialized_fill_n(dest, count, val)` |
| `0x0048a080` | `std_vector70_construct` | one guarded element copy |
| `0x004f5800` | `std_vector70_insert_one` | `insert(pos, 1, val)` — **off-cluster**, 420 KB away |

⚠️ **None of them is a `C_driver` method**, decided by measurement rather than by the
neighbourhood: the three loop helpers take raw pointers and index no struct field at all, and
`std_vector70_insert_one` indexes `+0x04`/`+0x08`/`+0x0c`, i.e. `this` **is** the vector. Their
entire callsite census is two functions — `driver_tick` and `std_vector70_insert_one` — so the
family is shared template code with exactly one instantiation in this binary. Each was censused
separately. `std_vector70_insert_one` was surfaced only by that census: compiler-generated members
live beside the container helpers, not in the owning class's cluster.

### 4d. Address-region neighbours that are not `C_driver` methods

- `door_set_open_angle` — `C_door::SetOpenAngle` (`structs/door.md`).
- `i3d_copy_bound_volume` (`0x00482ba0`) — copies a 10-dword bound volume of an `I3D_frame`; a
  shared helper in the `C_door` TU (`structs/door.md` §6).

## 5. PC vs PS2

| item | PC | PS2 |
|---|---|---|
| `C_driver` size | ~0x29C | 0x320 |
| driven vehicle | `this+0x10` | `this+0x12c` |
| path-node record (`S_car_node`) stride | 0x70 | 0x90 |
| nav node/edge index in the node | `+0x60` | `+0x84` |
| light bits written by `DoLights` | car+0x20c | car+0x5f8 |
| sound timing written by `DoSounds` | car+0x51c | car+0x229c |

- `opponent_fill_path_node`'s twin is `FillObjzNode__10C_opponentFR8S_vectorfR8S_vectorP10S_car_node`,
  which also supplies the `S_car_node` type name.
- `std_vector70_insert_n` corresponds to `vector<S_car_node>::insert`.
- `vehicle_set_brake` (`0x00517600`) is `C_Vehicle::SetBrake`, which fixes `driver+0xDC` as the
  brake pedal.
- The 10-entry table of §3a is the PS2 `semaphore_states` (40 bytes, indexed from 0 by
  `IsSemaPoint` and `TickSemaphores__6C_gameFv`).
- The phase pointer is PS2 node `+0x80`; the PS2 builders `RecompilePath__12C_sledovackaFi`,
  `ActivateElems__13C_traffic_carFbi` and `AI__13C_traffic_carFUi` point it at `C_game+0x1e4` /
  `+0x1e8` (PC `+0x19c` / `+0x1a0`) by the same axis test, or at `+0x140` of a bridge list entry.

## 6. Cross-references

- `structs/vehicle.md` — the driven car's `C_Vehicle` (`this+0x10`, then `+0x70`) + gas/brake/steer setters.
- `structs/pursuit-event-manager.md` — the S_ELEMENT record that owns this driver at `+0x04`.
- `police-system/artifacts/boosting.md` — `speed_factor` / model-level boost.
