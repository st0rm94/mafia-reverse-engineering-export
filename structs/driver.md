# `C_driver` — the AI driver that steers a car along a node path

**Created** 2026-07-11 13:22 +0200 · **Modified** 2026-11-09 09:39 +0100

One `C_driver` is the per-car AI that physically **steers, throttles and brakes** a car along a
cached path of nav-graph nodes. It is not police-specific: police pursuits, escape and move-to
orders, ambient traffic and the racing opponents all drive their cars through this object. It is a
plain heap record (no vtable), allocated by its owners and stored at the pursuit S_ELEMENT
`record+0x04` (`structs/pursuit_event.md`), in the traffic crew records and in the racing
opponent.

| | |
|---|---|
| PC prefix | `driver_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x29c` (`operator_new(0x29c)` before every `driver_ctor` call) |
| PC constructor | `driver_ctor` @ `0x00482c00` |
| PC translation unit | `[0x00482bd0, 0x0048a0a0)`: the unit's CRT pair, the 23 `driver_*` methods and the five `std_vector70_*` helpers; the next unit's guard `crt_static_init_guard_0048a0a0` opens `C_drop_in_weapon` |
| PS2 class | `C_driver` (tag `8C_driver`), 23 methods, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

PC offsets. `car` is the driven `C_car` at `+0x10`; its `C_Vehicle` sub-object is `car+0x70`
(`structs/vehicle.md`). Every offset the class or its outside users access is in the typedef, and
so are the bands no code accesses (`unused_`); only the padding after byte and short fields is
left out.

```c
typedef struct C_driver {                    // 0x29c bytes; PC offsets
    /* +0x000 */ std_vector10 swept_path;      // [H] 16-byte samples of the predicted path (§1b)
    /* +0x010 */ C_car   *car;                 // [H] the driven car
    /* +0x014 */ float    min_brake_fraction;  // [H] 0.4: +0xf4 = this * brake_force_cap
    /* +0x018 */ float    decel_fraction;      // [H] 0.7: +0xf0 = this * brake_force_cap
    /* +0x01c */ float    front_sphere_centre[3]; // [H] car-space centre of the front probe sphere
    /* +0x028 */ float    rear_sphere_centre[3]; // [H] car-space centre of the rear sphere; no reader (class census)
    /* +0x034 */ float    collision_radius;    // [H] 0.7 * the smallest car half-extent
    /* +0x038 */ float    brake_onset_time;    // [H] time to the corner
    /* +0x03c */ float    brake_onset_dist;    // [H] distance to the corner
    /* +0x040 */ float    collision_hit[3];    // [H] predicted collision point (out of actor_is_possible_coll_all)
    /* +0x04c */ float    probe_extent[3];     // [H] the obstacle probe: its direction times its length
    /* +0x058 */ float    probe_origin[3];     // [H] the point the obstacle probe starts from
    /* +0x064 */ uint32_t path_node_count;     // [H] (end - begin) / 0x70, recomputed each tick
    /* +0x068 */ uint32_t next_node_index;     // [H] path_cursor + 1, wrapping to 0
    /* +0x06c */ float    target_pos[3];       // [H] the world position the car steers for
    /* +0x078 */ uint32_t detour_node_index;   // [H] the detour node driver_tick inserts after a front hit
    /* +0x07c */ float    power_coeff_in_sector;  // [H] racing power coefficient, used in the camera's sector
    /* +0x080 */ float    power_coeff_out_sector; // [H] the same, used out of the camera's sector
    /* +0x084 */ float    speed_factor;        // [H] throttle scale; gas = this * throttle
    /* +0x088 */ float    dir_to_target[3];    // [H] target_pos - car position
    /* +0x094 */ uint32_t unused_94[3];        // [U] no access; no reader (class census)
    /* +0x0a0 */ float    car_forward[3];      // [H] the car's forward axis (C_actor cached_forward)
    /* +0x0ac */ float    car_right[3];        // [H] the car's right axis (C_Vehicle axis_right)
    /* +0x0b8 */ float    car_up[3];           // [H] the car's up axis (C_Vehicle axis_up)
    /* +0x0c4 */ float    car_pos[3];          // [H] cached car world position
    /* +0x0d0 */ float    steer_lookat[3];     // [H] dir_to_target projected onto the car's plane
    /* +0x0dc */ float    brake_pedal;         // [H] [0, 1], fed to vehicle_set_brake
    /* +0x0e0 */ float    turn_circle_centre[3]; // [H] centre of the car's minimum turning circle
    /* +0x0ec */ float    brake_force_cap;     // [H] the brake-force cap, at least 1.0
    /* +0x0f0 */ float    decel_force;         // [H] decel_fraction * brake_force_cap
    /* +0x0f4 */ float    min_brake_force;     // [H] min_brake_fraction * brake_force_cap
    /* +0x0f8 */ float    forward_force_cap;   // [H] C_Vehicle forward_force_cap; divides the pedal force
    /* +0x0fc */ float    side_force_sum;      // [H] C_Vehicle side_force_sum, the wheels' side-force sum
    /* +0x100 */ float    forward_force_sum;   // [H] C_Vehicle forward_force_sum, the wheels' forward-force sum
    /* +0x104 */ float    throttle;            // [H] [0, 1]; 1.0 each tick, 0 when blocked
    /* +0x108 */ uint32_t lookahead_node;      // [H] the node index the braking look-ahead reaches
    /* +0x10c */ float    prev_probe_extent[3]; // [H] probe_extent at the end of the last prediction; no reader (class census)
    /* +0x118 */ float    prev_probe_origin[3]; // [H] probe_origin at the end of the last prediction; no reader (class census)
    /* +0x124 */ float    seed_pos[3];         // [H] the lane anchor the corrections steer back to (§2)
    /* +0x130 */ uint16_t flash_on_ms[6];      // [H] headlight flash pattern: on-time per flash
    /* +0x13c */ uint16_t flash_off_ms[6];     // [H] off-time per flash
    /* +0x148 */ uint16_t flash_count;         // [H] entries used, 2..6
    /* +0x14a */ uint16_t flash_pause_ms;      // [H] pause after the pattern, 1000..4999
    /* +0x14c */ uint16_t sound_on_ms[4];      // [H] car+0x51c pattern: on-time per pulse
    /* +0x154 */ uint16_t sound_off_ms[4];     // [H] off-time per pulse
    /* +0x15c */ uint16_t sound_count;         // [H] entries used, 2..4
    /* +0x15e */ uint16_t sound_pause_ms;      // [H] pause after the pattern, 1000..4999
    /* +0x160 */ std_vector70 path;            // [H] S_car_node records (§1a)
    /* +0x170 */ uint32_t path_cursor;         // [H] the current path node
    /* +0x174 */ uint32_t prev_node_index;     // [H] path_cursor - 1, wrapping
    /* +0x178 */ uint8_t  path_extend_request; // [H] traffic_car_ai extends the path to the next crossing
    /* +0x17c */ float    speed_limit;         // [H] the limit fed to vehicle_set_speed_limit (§2)
    /* +0x180 */ uint32_t unread_180;          // [U] the pursuit save stream; no reader (.text scan)
    /* +0x184 */ uint32_t exit_entry_index;    // [H] the entry chosen at the path's last crossing
    /* +0x188 */ uint16_t exit_approach;       // [H] the approach the path leaves its last crossing by
    /* +0x18a */ uint16_t cross_approach;      // [H] the approach of the crossing the car is on
    /* +0x18c */ uint16_t cross_node;          // [H] the road node of the crossing the car is on
    /* +0x190 */ float    exit_segment_xz[2];  // [H] (x, z) of the path's last segment
    /* +0x198 */ uint32_t unused_198;          // [U] no access; no reader (class census)
    /* +0x19c */ uint32_t flags;               // [H] bit map §2a; the sign bit = oncoming obstacle
    /* +0x1a0 */ C_actor *predicted_obstacle;  // [H] the obstacle the collision prediction found
    /* +0x1a4 */ C_actor *lane_obstacle;       // [H] the obstacle the lane-change probe found
    /* +0x1a8 */ uint32_t unused_1a8;          // [U] no access; no reader (class census)
    /* +0x1ac */ std_vector lock_crossings;    // [H] the crossings this car has locked, S_LOCKCROSS records (§2)
    /* +0x1bc */ uint32_t lock_range_end_node; // [H] upper node of the range whose crossings traffic_car_ai locks
    /* +0x1c0 */ float    target_speed;        // [H] the cornering speed cap
    /* +0x1c4 */ uint32_t effect_request_mask; // [H] 0 each tick; traffic_car_ai copies it to the car
    /* +0x1c8 */ float    speed_jitter;        // [H] 1.0 at ctor; 0.65..1.0 per jitter period
    /* +0x1cc */ float    jitter_period_ms;    // [H] 5000.0; 3000..10999 per traffic activation
    /* +0x1d0 */ float    jitter_elapsed_ms;   // [H] accumulates the frame delta in drive_mode 0
    /* +0x1d4 */ float    node_reach_dist;     // [H] 30.0 at ctor/reset; 20..40 per reached node
    /* +0x1d8 */ uint8_t  traffic_mode_flags;  // [H] bit map §2a: 0x1 skip the lane-change probe, 0x2 park
    /* +0x1dc */ uint32_t state_flags;         // [H] bit map §2a
    /* +0x1e0 */ float    steer;               // [H] steering output, fed to vehicle_set_steer
    /* +0x1e4 */ uint32_t signal_flags;        // [H] bit map §2a
    /* +0x1e8 */ uint32_t brake_light_ms;      // [H] time since the last brake; the light holds 500 ms
    /* +0x1ec */ uint32_t flash_elapsed_ms;    // [H] time inside the current flash entry
    /* +0x1f0 */ uint16_t flash_index;         // [H] the current flash entry
    /* +0x1f4 */ uint32_t sound_elapsed_ms;    // [H] time inside the current sound entry
    /* +0x1f8 */ uint16_t sound_index;         // [H] the current sound entry
    /* +0x1fc */ C_actor *target_actor;        // [H] the pursued actor; 1 in drive_mode 0
    /* +0x200 */ C_car   *target_actor_vehicle;// [H] the target's vehicle at dispatch; 1 in drive_mode 0
    /* +0x204 */ C_car   *target_vehicle_now;  // [H] the target's current vehicle, refreshed by the owner
    /* +0x208 */ uint32_t drive_mode;          // [H] 0 default and move-to, 1 pursue / escape, 2 racing
    /* +0x20c */ int32_t  line_clear_ms;       // [H] += dt; 0 when the line to the target is blocked
    /* +0x210 */ float    brake_lookahead_dist;// [H] the ramped braking look-ahead distance
    /* +0x214 */ uint8_t  path_correction_enabled; // [H] 1 at ctor; gates driver_path_correction
    /* +0x218 */ float    end_speed_cap;       // [H] the speed cap at the last node
    /* +0x21c */ int32_t  throttle_hold_ms;    // [H] += dt; the gas is 0 while it is below 1
    /* +0x220 */ int32_t  stuck_ms;            // [H] += dt while the car is stuck; 0 on every other tick
    /* +0x224 */ int32_t  committed_node;      // [H] -1 sentinel
    /* +0x228 */ uint8_t  unused_228[0x70];    // [U] no access; no reader (class census)
    /* +0x298 */ uint8_t  blocked_latch;       // [H] the owner's blocked-driver latch
} C_driver;
```

**Why the pattern arrays have six and four entries.** In drive_mode 0, while the `flags` sign bit
says an obstacle is coming toward the car (§2a), `driver_tick` arms the headlight pattern by
drawing `rand % 5 + 2` flashes, i.e. **2 to 6**,
each with a random on-time and off-time of 100..249 ms; the sound pattern draws `rand % 3 + 2`
pulses, **2 to 4**, of 100..399 ms each. The arrays are sized for the largest draw, and
`flash_count` / `sound_count` say how many entries are live. Each pattern is followed by a random
pause of 1000..4999 ms (`flash_pause_ms`, `sound_pause_ms`), after which `driver_do_lights` /
`driver_do_sounds` clear the armed bit and the next tick draws a new pattern.

### 1a. `S_car_node` — the path record (stride `0x70`)

```c
typedef struct S_car_node {                   // 0x70 bytes
    /* +0x00 */ float    entry_point[4][3];    // [H] candidate world positions; entry_index picks one
    /* +0x30 */ float    segment_len[4];       // [H] per entry: the length to the next node's same entry
    /* +0x40 */ uint8_t  entry_lane[4];        // [H] per entry: the road lane it uses
    /* +0x44 */ uint8_t  entry_index;          // [H] the active entry point
    /* +0x45 */ uint8_t  entry_count;          // [H] the number of entries: road lanes of type 2 or 3
    /* +0x46 */ uint8_t  driving_lane_count;   // [H] the number of type-2 (driving) lanes
    /* +0x48 */ float    speed_cap;            // [H] speed_limit = speed_cap * speed_jitter
    /* +0x4c */ float    reach_radius;         // [H] the node counts as reached below this distance
    /* +0x50 */ float    turn_radius_cap;      // [H] driver_compute_radius
    /* +0x54 */ uint8_t  claim_flag;           // [H] a copy of the road slot's claim_flag (structs/roads.md §1d)
    /* +0x58 */ uint8_t *straight_lane_counter; // [H] a road-slot lane's straight_reservations byte (structs/roads.md §1e)
    /* +0x5c */ uint8_t *turn_lane_counter;    // [H] a road-slot lane's turn_reservations byte (structs/roads.md §1e)
    /* +0x60 */ uint16_t nav_index;            // [H] the road graph's tagged id; bit 0x8000 = a road node
    /* +0x61 */ uint8_t  nav_flags;            // [H] high byte of nav_index; 0x80 = the edge bit
    /* +0x64 */ S_road_slot *road_slot;        // [H] the road slot the node lies on (structs/roads.md §1d)
    /* +0x68 */ int32_t *light_phase;          // [H] the traffic-light phase this node waits on (§4)
    /* +0x6c */ uint8_t  corners_done;         // [H] the node's corner entries are smoothed
} S_car_node;
```

**Why four entry points.** `entry_index` is a byte the producers set, and every consumer selects
the entry with `node + entry_index * 0xc`; the four `vec3` slots (`0x30` bytes) end exactly where
the per-entry lengths begin, and `+0x30 + i*4` holds one length per slot. A slot is one lateral
position across the road (the lane the car takes through that node): `entry_lane` names the road
lane of each slot, `entry_count` says how many are live, and the racing producer writes only slot
0.

### 1b. `std_vector10` — the swept-path samples

The vector at `+0x00` holds 16-byte samples: `driver_predict_collision` erases it, appends with
`std_vector10_insert_n` and walks it with a 16-byte stride (`(end - begin) >> 4`). Each sample is
`{float point[3]; float path_fraction}`. Sample 0 is `car_pos` with 0.0; each path node then adds
its entry point with its distance along the path divided by the look-ahead length; the last sample
is interpolated to exactly the look-ahead length, with 1.0. The reader walks the samples as segment
pairs and solves the time-to-collision quadratic against the obstacle's position (`C_actor +0x24`)
plus its velocity (car `+0x1f7c`, or its vtable slot `0x70`) times 6 times the fraction, with a
radius² of 3.24 (1.8 m).

## 2. Field evidence

Measured by a census of every `this+N` access in the 28 functions of the unit, plus body reads of
the external code that builds, drives and restores a driver: every `traffic_car_*` body,
`pursuit_event_tick_driver`, `pursuit_event_tick`, the three `pursuit_event_dispatch_*` orders, the
pursuit save pair, `racing_update_driver_power_coeff`, `opponent_create_driver` and
`driver_path_node_at`. The other functions that call `driver_*` methods were checked access by
access, with each `.text` hit's base register read; "no access" and "no reader" rest on that trace.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | swept_path | H | `driver_ctor`, `driver_dtor`, `driver_predict_collision` (`std_vector10_erase_range`, `std_vector10_insert_n`) | `driver_predict_collision` | §1b; the alloc byte, then begin `+0x04`, end `+0x08`, cap `+0x0c` |
| +0x010 | car | H | `driver_ctor` (0), `driver_set_car` | every `driver_*` method but the vector helpers | `driver_tick` passes `car+0x70` to every `vehicle_*` setter |
| +0x014 | min_brake_fraction | H | `driver_ctor` (0.4) | `driver_fill_start_values` | `+0xf4 = this * +0xec`; a pedal force below `+0xf4` is 0 |
| +0x018 | decel_fraction | H | `driver_ctor` (0.7) | `driver_fill_start_values` | `+0xf0 = this * +0xec` |
| +0x01c | front_sphere_centre | H | `driver_set_car` | `driver_tick` (`0x00483d74`, `0x00484f09`) | (half size x, half size y, size z - `collision_radius`) minus the pivot. In drive modes 1 and 2, after a front hit below 2.0 speed in a forward gear, `driver_tick` turns it into the world with the car frame's matrix (`S_vector::operator*`) and sweeps a `collision_radius` sphere from it along 2 x `+0xa0` and ±3 x `+0xac` |
| +0x028 | rear_sphere_centre | H | `driver_set_car` | - | z = `collision_radius` minus the pivot. No reader: the only driver-method access is the `lea` of `driver_set_car` (`0x004832d4`), and the `+0x28` hits of the functions that call driver methods are other records |
| +0x034 | collision_radius | H | `driver_set_car` | `driver_set_car`, `driver_tick` | 0.7 * min of the three half-extents |
| +0x038 | brake_onset_time | H | `driver_compute_brake_onset`, `driver_compute_corner_speed_limit` | `driver_tick` | - |
| +0x03c | brake_onset_dist | H | `driver_compute_brake_onset` | `driver_tick`, `driver_last_route_in_brake_path_tol`, `driver_ramp_brake_distance`, `driver_lane_change_and_park`, `driver_predictive_sphere_lane_cross`, `driver_reduce_speed_in_corner` | - |
| +0x040 | collision_hit | H | `driver_lane_change_and_park`, `driver_predict_collision` (out argument of `actor_is_possible_coll_all`) | `driver_predict_collision` | the distance from the probe origin to the hit decides the stop |
| +0x04c | probe_extent | H | `driver_lane_change_and_park`, `driver_predict_collision` | `driver_lane_change_and_park`, `driver_predict_collision` | the direction argument of `actor_is_possible_coll_all`: the car's velocity direction (car `+0x1f7c`) x (6 x speed + 0.1), or `dir_to_target` below 0.1 speed; in the lane change, the previous node's entry point minus the current one's, normalised, x (distance + 2) |
| +0x058 | probe_origin | H | `driver_lane_change_and_park`, `driver_predict_collision` | `driver_lane_change_and_park`, `driver_predict_collision` | `car_pos`, or the AI probe point when reversing or when the velocity points more than 1.57 rad off forward; in the lane change, `target_pos` or the current node's entry point |
| +0x064 | path_node_count | H | `driver_tick` | `driver_tick` and ten of the helpers | the tick returns 0 when it is 0 or not above `path_cursor` |
| +0x068 | next_node_index | H | `driver_fill_start_values` | `driver_optimize_next_checkpoints` | a node index, not a frame |
| +0x06c | target_pos | H | `driver_tick`, `driver_fill_start_values`, `driver_advance_checkpoint`, `driver_path_correction`, `driver_lane_change_and_park` | `driver_tick`, the steering and braking helpers | - |
| +0x078 | detour_node_index | H | `driver_tick` | `driver_tick` | in drive modes 1 and 2, after a front hit, `driver_tick` inserts a detour node here and sets `flags` `0x10` and `state_flags` `0x10000000`; once the cursor passes it, or after a static rear hit in reverse, the node is erased and the cursor returns to this index |
| +0x07c | power_coeff_in_sector | H | `driver_ctor`, `driver_reset` (1.0), `opponent_create_driver`, `racing_update_driver_power_coeff` | `driver_tick` | the racing power coefficient by difficulty (`racing_get_power_coeff_by_difficulty`), x0.7 when the racer leads by 100..150 gates and x0.6 beyond 150 (the opponent's `+0x48` is this driver); drive mode 2 copies it to car `climb_power_floor` (`+0x2018`) while `C_actor+0x5f`, the car in the camera's sector, is set |
| +0x080 | power_coeff_out_sector | H | `driver_ctor`, `driver_reset` (1.0), `opponent_create_driver`, `racing_update_driver_power_coeff` | `driver_tick` | the same, copied while `C_actor+0x5f` is clear |
| +0x084 | speed_factor | H | `driver_ctor`, `driver_reset` (1.0), `pursuit_event_dispatch_pursuit_vehicle` / `_escape_vehicle` (the caller's factor), `pursuit_event_dispatch_move_to_vehicle` (1.0) | `driver_tick` | gas = `+0x84 * +0x104` (`police-system/artifacts/boosting.md`) |
| +0x088 | dir_to_target | H | `driver_tick`, `driver_fill_start_values`, `driver_advance_checkpoint`, the steering and avoidance helpers | `driver_tick`, the braking and avoidance helpers | `driver_tick` zeroes it on a stop order (flag `0x8`) |
| +0x094 | unused_94 | U | - | - | no access in a driver method; the `+0x94`..`+0x9f` hits of the functions that call driver methods are save-buffer reads of `pursuit_event_save_game_load`, `target +0x98` in the dispatchers and fields of other classes |
| +0x0a0 | car_forward | H | `driver_fill_start_values` (car `+0x30`, the `C_actor` `cached_forward`) | `driver_tick`, `driver_optimize_next_checkpoints`, `driver_compute_brake_onset` | the brake onset takes its pitch (the angle to its horizontal projection, signed by y); a node is behind when `dir · this < 0`; the mode-2 sweep runs along 2 x this |
| +0x0ac | car_right | H | `driver_fill_start_values` (car `+0xd34`, the `C_Vehicle` `axis_right`) | `driver_tick`, `driver_reduce_speed_in_corner`, `driver_turn` | the half-width offsets (signed by car `+0x714`), the side of the turning circle and the sign of the steer |
| +0x0b8 | car_up | H | `driver_fill_start_values` (car `+0xd40`, the `C_Vehicle` `axis_up`) | `driver_tick`, `driver_turn` | the plane normal of the steering look-at |
| +0x0c4 | car_pos | H | `driver_fill_start_values` | `driver_tick` and seven helpers | the start of the horizontal line test of `+0x20c` |
| +0x0d0 | steer_lookat | H | `driver_tick`, `driver_turn` | `driver_turn` | its angle is the steer; in drive modes 1 and 2 `driver_turn` writes `car_pos` + this x 20 to car `+0xa0` (the `C_Vehicle` `steer_to_point`) and 1 to car `+0xac` |
| +0x0dc | brake_pedal | H | `driver_tick` (0 each tick, 0 below 0.3 speed), `driver_compute_brake_pedal`, `driver_turn` | `driver_tick` (`vehicle_set_brake`), `driver_do_lights` (argument) | - |
| +0x0e0 | turn_circle_centre | H | `driver_reduce_speed_in_corner` | `driver_reduce_speed_in_corner` | the car position ± `car_right` x r, r = mass x speed² / `+0xfc` (at least car `+0x440`), on the target's side; a target inside the circle caps `target_speed` at sqrt(0.85 x f x d / m) |
| +0x0ec | brake_force_cap | H | `driver_fill_start_values` | `driver_fill_start_values`, `driver_compute_brake_pedal` | starts as `+0xf8`, capped at 0.9 x `+0xfc` and 0.9 x `+0x100`, at least 1.0 |
| +0x0f0 | decel_force | H | `driver_fill_start_values` | `driver_compute_brake_onset`, `driver_compute_brake_pedal` | divided by `car+0x6e0` in the pedal formula |
| +0x0f4 | min_brake_force | H | `driver_fill_start_values` | `driver_compute_brake_pedal` | a pedal force below it is 0 |
| +0x0f8 | forward_force_cap | H | `driver_fill_start_values` (`car+0x23c`) | `driver_compute_brake_pedal` | the `C_Vehicle` `forward_force_cap` (`structs/vehicle.md` `+0x1cc`); `brake_pedal = clamp(f) / +0xf8` |
| +0x0fc | side_force_sum | H | `driver_fill_start_values` (`car+0x234`) | `driver_tick`, `driver_fill_start_values`, `driver_compute_corner_speed_limit`, `driver_reduce_speed_in_corner` | `structs/vehicle.md` (`+0x1c4`) |
| +0x100 | forward_force_sum | H | `driver_fill_start_values` (`car+0x238`) | `driver_fill_start_values` | `structs/vehicle.md` (`+0x1c8`) |
| +0x104 | throttle | H | `driver_tick` (1.0 each tick), `driver_advance_checkpoint`, `driver_lane_change_and_park`, `driver_predict_collision` (0) | `driver_tick` | - |
| +0x108 | lookahead_node | H | `driver_last_route_in_brake_path` | `driver_tick`, `driver_last_route_in_brake_path`, `driver_compute_corner_speed_limit` | - |
| +0x10c | prev_probe_extent | H | `driver_ctor` (`+0x10c`..`+0x114` = 0), `driver_predict_collision` (every call, `0x00488ddf`) | - | a copy of `probe_extent` at the end of each prediction. No reader: the `+0x110` / `+0x120` hits of the functions that call driver methods are I3D frame, model and pursuit-record fields |
| +0x118 | prev_probe_origin | H | `driver_predict_collision` (every call, `0x00488de5`) | - | a copy of `probe_origin`; no reader, as `+0x10c` |
| +0x124 | seed_pos | H | `driver_reset`, `driver_optimize_next_checkpoints`, `driver_advance_checkpoint`, `driver_lane_change_and_park`, `pursuit_event_save_game_load` | `driver_path_correction`, `driver_predictive_sphere_lane_cross` | the first node's entry point after `driver_reset`, then the lane anchor the corrections steer back to |
| +0x130 | flash_on_ms | H | `driver_tick`, `pursuit_event_save_game_load` | `driver_do_lights` | §1 |
| +0x13c | flash_off_ms | H | `driver_tick`, `pursuit_event_save_game_load` | `driver_do_lights` | §1 |
| +0x148 | flash_count | H | `driver_tick` | `driver_tick`, `driver_do_lights` | - |
| +0x14a | flash_pause_ms | H | `driver_tick` | `driver_do_lights` | - |
| +0x14c | sound_on_ms | H | `driver_tick`, `pursuit_event_save_game_load` | `driver_do_sounds` | §1 |
| +0x154 | sound_off_ms | H | `driver_tick`, `pursuit_event_save_game_load` | `driver_do_sounds` | §1 |
| +0x15c | sound_count | H | `driver_tick` | `driver_tick`, `driver_do_sounds` | - |
| +0x15e | sound_pause_ms | H | `driver_tick` | `driver_do_sounds` | - |
| +0x160 | path | H | `driver_ctor`, `driver_dtor`, `driver_tick`, `pursuit_event_dispatch_escape_vehicle`, `pursuit_event_dispatch_move_to_vehicle` (`std_vector70_*`) | every helper; `pursuit_event_tick` (`driver_path_node_at`) | §1a, §4; the alloc byte, then begin `+0x164`, end `+0x168`, cap `+0x16c` |
| +0x170 | path_cursor | H | `driver_tick`, `driver_advance_checkpoint`, the three dispatchers (0, then 1 once built), `pursuit_event_tick_driver` (sets the cursor and adds 1), `pursuit_event_save_game_load` | `driver_tick` and every helper | - |
| +0x174 | prev_node_index | H | `driver_fill_start_values`, `driver_advance_checkpoint`, `pursuit_event_save_game_load` | `driver_tick`, `driver_fill_start_values`, `driver_optimize_next_checkpoints`, `driver_advance_checkpoint`, `driver_last_route_in_brake_path_tol`, `driver_lane_change_and_park` | - |
| +0x178 | path_extend_request | H | `driver_advance_checkpoint`, `driver_last_route_in_brake_path` (1), `traffic_car_activate_elements`, `traffic_car_adopt_car_with_new_driver` (0) | `traffic_car_ai` | `traffic_car_ai` clears it, drops the passed nodes, extends the path to the next crossing (`roads_find_car_path_to_next_crossing`) and calls `traffic_car_prepare_next_direction` |
| +0x17c | speed_limit | H | `driver_ctor` (17.0), `driver_tick`, `driver_fill_start_values`, `pursuit_event_save_game_load` | `driver_tick` (`vehicle_set_speed_limit`) | 17.0 at ctor; the node's `speed_cap` * `speed_jitter`, or 1000.0 outside drive_mode 0 |
| +0x180 | unread_180 | U | `pursuit_event_save_game_load` | `pursuit_event_save_game_save` | only the pursuit save pair touches it (`.text` scan) |
| +0x184 | exit_entry_index | H | `traffic_car_activate_elements` (2), `traffic_car_prepare_next_direction` (2), `pursuit_event_save_game_load` | `traffic_car_ai`, `traffic_car_lock_crossing`, `pursuit_event_save_game_save` | the entry chosen at the path's last crossing, capped at the count of type-2 lanes - 1; `traffic_car_ai` makes it the `entry_index` of the extended path's nodes |
| +0x188 | exit_approach | H | `traffic_car_activate_elements`, `traffic_car_prepare_next_direction`, `pursuit_event_save_game_load` | `traffic_car_deactivate_element`, `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_lock_crossing`, `pursuit_event_save_game_save` | the approach (0..3) the path leaves its last crossing by; the car holds one count in that road approach's `+0x2c`, which `traffic_car_deactivate_element` releases; passed to `roads_find_car_path_to_next_crossing` |
| +0x18a | cross_approach | H | `traffic_car_activate_elements`, `traffic_car_ai` (a passed crossing, `flags` `0x10000`), `pursuit_event_save_game_load` | `traffic_car_deactivate_element`, `traffic_car_activate_elements`, `traffic_car_ai`, `pursuit_event_save_game_save` | the approach index of the crossing the car is on; the car holds one count in that approach's `+0x2c` |
| +0x18c | cross_node | H | `traffic_car_activate_elements`, `traffic_car_ai` (a passed crossing), `pursuit_event_save_game_load` | `traffic_car_deactivate_element`, `traffic_car_activate_elements`, `traffic_car_ai`, `pursuit_event_save_game_save` | the road node of that crossing (index into the `0xec`-stride table) |
| +0x190 | exit_segment_xz | H | `traffic_car_activate_elements`, `traffic_car_ai`, `pursuit_event_save_game_load` | `traffic_car_activate_elements`, `traffic_car_ai`, `pursuit_event_save_game_save` | the line `vec2_line_intersect` cuts the corner entries with |
| +0x198 | unused_198 | U | - | - | no access |
| +0x19c | flags | H | `driver_ctor`, `driver_reset` (0), `driver_tick` and five helpers, `pursuit_event_dispatch_move_to_vehicle` (`0x10`), `pursuit_event_save_game_load`, `traffic_car_activate_elements`, `traffic_car_prepare_next_direction`, `traffic_car_ai` | `driver_tick` and five helpers, `traffic_car_ai` | §2a |
| +0x1a0 | predicted_obstacle | H | `driver_ctor`, `driver_predict_collision`, `pursuit_event_drop_actor_references` (0) | `driver_tick`, `driver_lane_change_and_park`, `driver_predict_collision` | `racing_predict_car_collision` supplies it in drive_mode 2 |
| +0x1a4 | lane_obstacle | H | `driver_ctor`, `driver_lane_change_and_park` (out argument), `traffic_car_ai` (0) | `traffic_car_ai` | when two elements' `+0x1a4` point at each other's car, the one behind clears its own and sets `traffic_mode_flags` `0x1`; `pursuit_event_save_game_load` re-resolves the saved reference |
| +0x1a8 | unused_1a8 | U | - | - | no `C_driver` access: the `[eax+0x1a8]` of `driver_reset` is car `+0x218` through the `C_Vehicle` |
| +0x1ac | lock_crossings | H | `driver_ctor`, `driver_dtor`, `traffic_car_ai`, `traffic_car_lock_crossing` | `driver_dtor`, `traffic_car_clear_lane_blocking`, `traffic_car_lock_crossing` | `structs/traffic_car.md` §1e; the alloc byte, then `+0x1b0`, `+0x1b4`, `+0x1b8`; stride `0x22`; no access in this unit but the ctor and dtor |
| +0x1bc | lock_range_end_node | H | `driver_tick` (0 each tick), `driver_last_route_in_brake_path_tol`, `pursuit_event_save_game_load` | `driver_last_route_in_brake_path_tol`, `traffic_car_ai` | the upper node of the range whose crossings `traffic_car_ai` locks |
| +0x1c0 | target_speed | H | `driver_tick`, `driver_compute_corner_speed_limit`, `driver_lane_change_and_park`, `driver_predictive_sphere_lane_cross`, `driver_predict_collision`, `driver_reduce_speed_in_corner` | `driver_tick`, `driver_compute_corner_speed_limit`, `driver_lane_change_and_park`, `driver_predictive_sphere_lane_cross`, `driver_compute_brake_pedal` | `driver_tick` forces it to 0 at a red or orange light (§4) |
| +0x1c4 | effect_request_mask | H | `driver_ctor`, `driver_tick` (0), `driver_predict_collision` (`\|= 0x10`) | `driver_predict_collision`, `traffic_car_ai` | `traffic_car_ai` copies it to car `effect_request_mask` (`+0x2038`), with `0x10` added when the car is unstable or within 8 of the player; bit `0x10` = a predicted hit within sqrt 3 |
| +0x1c8 | speed_jitter | H | `driver_ctor` (1.0), `driver_fill_start_values` | `driver_fill_start_values` | `rand % 100 * 0.01 * 0.35 + 0.65` |
| +0x1cc | jitter_period_ms | H | `driver_ctor` (5000.0), `traffic_car_activate_elements` (`rand % 8000 + 3000` per activation) | `driver_fill_start_values` | - |
| +0x1d0 | jitter_elapsed_ms | H | `driver_ctor`, `driver_fill_start_values` | `driver_fill_start_values` | wraps to 0 at `+0x1cc` |
| +0x1d4 | node_reach_dist | H | `driver_ctor`, `driver_reset` (30.0), `driver_advance_checkpoint`, `traffic_car_ai` | `driver_lane_change_and_park` | `(rand % 0x28 + 0x14)` per reached node; the park manoeuvre of `traffic_car_ai` stores `(rand % 40 + 20) x dist x 0.01`, at least 25 |
| +0x1d8 | traffic_mode_flags | H | `driver_ctor`, `driver_reset` (0), `traffic_car_ai`, `traffic_car_seat_pedestrian_as_driver`, `pursuit_event_save_game_load` | `driver_tick`, `driver_lane_change_and_park`, `driver_predictive_sphere_lane_cross` | §2a |
| +0x1dc | state_flags | H | `driver_ctor`, `driver_reset`, `driver_tick`, `driver_lane_change_and_park`, `driver_predict_collision`, `traffic_car_seat_pedestrian_as_driver` (clears `0x2`), `pursuit_event_save_game_load` | `driver_tick`, `driver_optimize_next_checkpoints`, `driver_path_correction`, `driver_lane_change_and_park`, `driver_predict_collision`, `traffic_car_ai`, `pursuit_event_save_game_save` | §2a |
| +0x1e0 | steer | H | `driver_ctor`, `driver_tick`, `driver_turn`, `pursuit_event_save_game_load` | `driver_tick` (`vehicle_set_steer`), `driver_turn` | - |
| +0x1e4 | signal_flags | H | `driver_ctor`, `driver_reset`, `driver_tick`, `driver_advance_checkpoint`, `driver_lane_change_and_park`, `driver_predictive_sphere_lane_cross`, `driver_do_lights`, `driver_do_sounds` | the same functions | §2a |
| +0x1e8 | brake_light_ms | H | `driver_ctor`, `driver_do_lights` | `driver_do_lights` | - |
| +0x1ec | flash_elapsed_ms | H | `driver_ctor`, `driver_tick`, `driver_do_lights` | `driver_do_lights` | - |
| +0x1f0 | flash_index | H | `driver_ctor`, `driver_tick`, `driver_do_lights` | `driver_do_lights` | - |
| +0x1f4 | sound_elapsed_ms | H | `driver_tick`, `driver_do_sounds` | `driver_do_sounds` | - |
| +0x1f8 | sound_index | H | `driver_tick`, `driver_do_sounds` | `driver_do_sounds` | - |
| +0x1fc | target_actor | H | `driver_ctor` (0), `driver_tick` (1 in drive_mode 0), `pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_tick` (the record's target, or 0 with `+0x204` set), `pursuit_event_save_game_load` | `driver_predict_collision` | - |
| +0x200 | target_actor_vehicle | H | `driver_ctor` (0), `driver_tick` (1 in drive_mode 0), `pursuit_event_dispatch_pursuit_vehicle` (`target+0x98`), `pursuit_event_tick` (the record's `chase_object`, or 0 with `+0x204` set), `pursuit_event_save_game_load` | `driver_predict_collision` | - |
| +0x204 | target_vehicle_now | H | `driver_ctor` (0), `pursuit_event_tick` (S_ELEMENT `+0x08`, or 0) | `driver_predict_collision` | in drive_mode 2 a predicted collision with this car stops the driver (`state_flags \|= 0x1000`) |
| +0x208 | drive_mode | H | `driver_ctor` (0), `pursuit_event_dispatch_pursuit_vehicle` / `_escape_vehicle` (1), `pursuit_event_dispatch_move_to_vehicle` (0), `pursuit_event_save_game_load` | `driver_tick` (its main switch) and seven helpers | in mode 2 `driver_predict_collision` asks `racing_predict_car_collision` for the obstacle |
| +0x20c | line_clear_ms | H | `driver_ctor` (2000), `driver_tick` | `driver_tick` | `driver_tick` adds dt and zeroes it when `collision_test_line_horizontal_static` from `car_pos` along `dir_to_target` hits |
| +0x210 | brake_lookahead_dist | H | `driver_ctor`, `driver_reset`, `driver_tick`, `driver_ramp_brake_distance` | `driver_tick`, `driver_ramp_brake_distance`, `driver_last_route_in_brake_path`, `driver_compute_corner_speed_limit`, `driver_predict_collision` | ramps toward `brake_onset_dist` |
| +0x214 | path_correction_enabled | H | `driver_ctor` (1), `pursuit_event_tick_driver` (0, then 1 after `driver_tick`) | `driver_tick`, `driver_path_correction` | the pursuit suppresses the path correction for one tick when the target lies behind the car and more than 0.6 away |
| +0x218 | end_speed_cap | H | `driver_ctor`, `driver_reset` (0), `pursuit_event_dispatch_pursuit_vehicle` (1000.0), `_escape_vehicle`, `_move_to_vehicle`, `pursuit_event_save_game_load` | `driver_tick` | replaces `target_speed` when it is lower and the cursor is on the last node |
| +0x21c | throttle_hold_ms | H | `driver_ctor`, `driver_reset` (1), `driver_tick` (`+= dt`; in drive_mode 0 with an oncoming obstacle -3000 when the obstacle is kind 2, the player, else -1500) | `driver_tick` | the gas is 0 while it is below 1, so a driver facing an oncoming obstacle waits 3 s for the player and 1.5 s for anything else |
| +0x220 | stuck_ms | H | `driver_ctor`, `driver_reset` (0), `driver_tick` | `driver_tick`, `pursuit_event_tick` | `+= dt` only while reversing (`flags` `0x1` or `0x10`), on a detour node, or in mode 1 facing an oncoming obstacle; 0 on every other tick. The pursuit acts (element state 1) once it passes 20 s (10 s with no target vehicle) with a static front or rear hit and the car in the camera's sector |
| +0x224 | committed_node | H | `driver_ctor`, `driver_set_car` (-1) | `driver_tick`, `driver_advance_checkpoint` | - |
| +0x228 | unused_228 | U | - | - | no access; `0x70` bytes, one `S_car_node` stride |
| +0x298 | blocked_latch | H | `driver_ctor`, `driver_reset` (0), `pursuit_event_tick` (1 after 2 s with no predicted obstacle; 0) | `driver_compute_corner_speed_limit` | §4 |

### 2a. The flag words

| Field | Bit | Meaning | Evidence |
|---|---|---|---|
| `flags` (`+0x19c`) | `0x1` | reverse requested | `driver_tick` passes `(flags & 1) == 0` to `driver_fill_start_values` |
| `flags` | `0x2` | with `traffic_mode_flags` bit `0x2`, a 4.0 speed cap | `driver_tick` caps `target_speed` at 4.0 when both are set |
| `flags` | `0x4` | overshoot | `driver_optimize_next_checkpoints` sets it when the next node lies behind |
| `flags` | `0x8` | stop | `driver_tick` zeroes `dir_to_target` and `target_speed` |
| `flags` | `0x10` | the drive_mode switch is skipped | `pursuit_event_dispatch_move_to_vehicle` sets it, and `driver_tick` with a detour node (`+0x78`); `driver_tick` runs its drive_mode switch only while it is clear |
| `flags` | `0x100` / `0x200` | the turn at the crossing ahead, left / right | `traffic_car_ai` moves the pending bits `0x1000000` / `0x2000000` here (`>> 16`) when a crossing is passed; `driver_tick` copies them into `signal_flags` bits `0x1` / `0x2` when no indicator is on, the target is within 30.0 and the next node's `nav_flags` bit `0x80` is clear; with both clear `driver_tick` and `driver_advance_checkpoint` wait at a node while its `straight_lane_counter` byte is non-zero, with either set while its `turn_lane_counter` byte is |
| `flags` | `0x1000`, `0x40000000` | cleared with the sign bit | `driver_predict_collision` masks the word with `0x3fffefff` when the obstacle clears |
| `flags` | `0x10000` | a crossing was passed | `driver_advance_checkpoint` sets it when the node the cursor leaves is not an edge; `traffic_car_ai` acts on it (`cross_approach`, `cross_node`, the turn bits); `driver_tick` clears it |
| `flags` | `0x1000000` / `0x2000000` | the turn chosen at the path's last crossing, left / right | set by `traffic_car_activate_elements` and `traffic_car_prepare_next_direction`; `traffic_car_ai` moves them into `0x100` / `0x200` |
| `flags` | sign bit | oncoming obstacle | `driver_predict_collision` sets it when the predicted obstacle's forward axis points back along the probe; in drive_mode 0 it arms the flash and sound patterns and the throttle hold |
| `state_flags` (`+0x1dc`) | `0x1` | the lane change is blocked | `driver_lane_change_and_park` sets it; `traffic_car_ai` then steps `entry_index` back and clears the park bit |
| `state_flags` | `0x2` | stopped after the park manoeuvre; kept across ticks | `driver_tick` masks the word with `0x10000002` each tick; read by `traffic_car_ai`, cleared by `traffic_car_seat_pedestrian_as_driver` |
| `state_flags` | `0x1000` | stop for the target | `driver_predict_collision` in drive_mode 2 |
| `state_flags` | `0x10000000` | a detour node is in the path; kept across ticks | `driver_tick` sets it with the detour node (`+0x78`) |
| `traffic_mode_flags` (`+0x1d8`) | `0x1` | skip the lane-change probe | `traffic_car_ai` sets it to break a mutual block (`+0x1a4`) and clears it (`&= 2`) after every traffic tick |
| `traffic_mode_flags` | `0x2` | park manoeuvre | `traffic_car_ai` sets it with `entry_index + 1` onto a type-3 (parking) lane; the stop flag `0x8` then switches the engine off, the lane change uses the current node and sets `state_flags` `0x1` when blocked, and `driver_predictive_sphere_lane_cross` sets `flags` `0x8` once the car is in the lane |
| `signal_flags` (`+0x1e4`) | `0x1` / `0x2` | left / right indicator | `driver_do_lights` mirrors them to `car+0x20c` bits 1 / 2 |
| `signal_flags` | `0x100` | brake light on | `driver_do_lights` sets `car+0x20c` bit `0x100` while the pedal is down and 500 ms after |
| `signal_flags` | `0x1000000` | headlight pattern armed | `driver_do_lights` flashes `car+0x20c` bit `0x10000`, the main headlights (`structs/vehicle.md` §2a) |
| `signal_flags` | `0x2000000` | sound pattern armed | `driver_do_sounds` pulses the byte `car+0x51c` |

### 2b. `S_car_node` evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | entry_point | H | `opponent_fill_path_node` (slot 0) | `driver_reset`, the steering helpers | §1a |
| +0x30 | segment_len | H | `opponent_fill_path_node` (slot 0), `traffic_car_activate_elements`, `traffic_car_ai`, `driver_tick` | `driver_last_route_in_brake_path` | per entry, the length to the next node's same entry; summed for the look-ahead |
| +0x40 | entry_lane | H | `traffic_car_activate_elements`, `traffic_car_ai` | `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_prepare_next_direction` | per entry, the road lane (type 2 or 3) it uses; indexes the lane counters behind `+0x58` / `+0x5c` |
| +0x44 | entry_index | H | `opponent_fill_path_node` (0) | `driver_reset`, `driver_predict_collision`, the steering helpers | the steering also reads the next node's byte, `node[i+1]+0x44` |
| +0x45 | entry_count | H | `traffic_car_activate_elements`, `traffic_car_ai`, `opponent_fill_path_node` (1), `driver_tick` (1) | `traffic_car_activate_elements`, `traffic_car_ai` | the bound of the entry loops; `traffic_car_ai` parks only while `entry_index < this - 1` |
| +0x46 | driving_lane_count | H | `traffic_car_activate_elements`, `traffic_car_ai`, `opponent_fill_path_node` (1), `driver_tick` (1) | `traffic_car_activate_elements`, `traffic_car_prepare_next_direction` | the lane choice at the last crossing (`exit_entry_index`) |
| +0x48 | speed_cap | H | `traffic_car_activate_elements`, `traffic_car_ai` (road node `+0x10`, crossing `+0xc`), `opponent_fill_path_node` (1000.0), `driver_tick` (100.0, 10.0) | `driver_fill_start_values`, `driver_compute_corner_speed_limit` | `speed_limit = this * speed_jitter`; `driver_compute_corner_speed_limit` takes the minimum |
| +0x4c | reach_radius | H | `traffic_car_activate_elements`, `traffic_car_ai` (3.5, an edge 1.5), `opponent_fill_path_node` (0.5), `driver_tick` (1.0) | `driver_advance_checkpoint`, `driver_compute_brake_pedal`, `driver_compute_corner_speed_limit`, `driver_compute_radius` | the node counts as reached below it; `driver_compute_radius` clamps the segment lengths to it, and the pedal and corner formulas subtract it from the distance |
| +0x50 | turn_radius_cap | H | `driver_compute_radius` | the braking helpers | `clamp_len * tan(angle / 2)`; `0x497423f0` = straight |
| +0x54 | claim_flag | H | `traffic_car_activate_elements` (`byte [crossing + k * 0x34 + 0x28]`, slot `k`'s `claim_flag`, for the first node and for each node a crossing's `first_node[k]` names), `traffic_car_ai`, `pursuit_event_recompile_path` (`*(node + 0x64) + 0x0c`), `opponent_fill_path_node` (1), `driver_tick` (1) | `traffic_car_ai` | gates the crossing lock range (`structs/roads.md` §2c) |
| +0x58 | straight_lane_counter | H | `traffic_car_activate_elements`, `traffic_car_prepare_next_direction` | `driver_tick`, `driver_advance_checkpoint` | points at the `straight_reservations` byte of one lane of one of the crossing's road slots (`structs/roads.md` §1e): crossing `+0x31` + slot x `0x34` + lane x 8, the lanes starting at slot `+0x14`; the count `S_LOCKCROSS` `blocks_a` holds (`structs/traffic_car.md` §1e); with `flags` `0x300` clear the car waits at the node while it is non-zero |
| +0x5c | turn_lane_counter | H | `traffic_car_activate_elements`, `traffic_car_prepare_next_direction` | `driver_tick`, `driver_advance_checkpoint` | the same lane's `turn_reservations` byte, crossing `+0x30` + slot x `0x34` + lane x 8, held by `S_LOCKCROSS` `blocks_b`; the same wait with a `flags` `0x300` bit set |
| +0x60 | nav_index | H | `roads_find_car_path_from_node_to_crossing`, `roads_find_car_path_to_next_crossing`, `pursuit_event_dispatch_escape_vehicle`, `pursuit_event_dispatch_move_to_vehicle`, `pursuit_event_tick` (each the tagged id of a route step) | `driver_lane_change_and_park`, `pursuit_event_recompile_path` and `pursuit_event_prepare_path` (the id selects the node or the crossing record) | the road graph's tagged id (`structs/roads.md` §1a): bit `0x8000` set = a road node, clear = a crossing; "edge" in this doc means a road node |
| +0x61 | nav_flags | H | - | `driver_fill_start_values`, `driver_advance_checkpoint`, `driver_tick`, `traffic_car_activate_elements`, `traffic_car_ai` | bit `0x80` = the edge bit of `nav_index` |
| +0x64 | road_slot | H | `roads_find_car_path_from_node_to_crossing` (`crossing_table + crossing * 0xec + 0x1c + slot * 0x34`), `roads_find_car_path_to_next_crossing` (the same slot, also into the previous last node), `pursuit_event_prepare_path`, `pursuit_event_save_game_load` (re-link after a load) | `pursuit_event_recompile_path` (`+0x0c`, lane kinds at `+0x16 + 8k`, widths at `+0x18 + 8k`), `traffic_car_activate_elements` (kinds, widths), `traffic_car_ai` (the next lane's kind 3 = parking) | an `S_road_slot *`, the road the node lies on (`structs/roads.md` §1d); lane type 2 = driving lane, 3 = parking lane |
| +0x68 | light_phase | H | `traffic_car_activate_elements`, `pursuit_event_recompile_path` | `driver_tick`, `driver_advance_checkpoint`, `pursuit_event_tick` | §4 |
| +0x6c | corners_done | H | `traffic_car_activate_elements`, `traffic_car_ai` | `traffic_car_activate_elements`, `traffic_car_ai` | set for edge nodes and after the line-intersection smoothing (`vec2_line_intersect`); the smoothing skips a node with it set |

## 3. Method family

Coverage: all **23** PS2 `8C_driver` methods have a PC function and every twin is CONFIRMED by a
body diff (shared constants, the field-role map and the call structure); one PC method,
`driver_path_node_at`, has no PS2 symbol. `driver_tick` calls the helpers in the PS2 `Tick` order.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00482bd0 | `crt_static_init_guard_00482bd0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00482bf0 | `crt_atexit_register_00482bf0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00482c00 | `driver_ctor` | instance | `__ct__8C_driverFv` | 0x00157200 | CONFIRMED | empties the three vectors, seeds the float defaults (§2) |
| 0x00482d50 | `driver_dtor` | instance | `__dt__8C_driverFv` | 0x00157360 | CONFIRMED | frees the vectors at `+0x1ac`, `+0x160` and `+0x00` |

### 3b. Accessors

The class has no accessor of its own (`driver_path_node_at` computes an address, §3c).

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00482de0 | `driver_compute_radius` | `ComputeRadius__8C_driverFii` | 0x00157470 | CONFIRMED | per-node turn cap `node+0x50` |
| 0x00482ff0 | `driver_reset` | `Reset__8C_driverFv` | 0x00157790 | CONFIRMED | per-run reset: seed point, flags, the car's light bits, timers |
| 0x004831c0 | `driver_set_car` | `SetCar__8C_driverFP5C_car` | 0x001579a0 | CONFIRMED | binds the car, sizes the collision radius and the two sphere centres |
| 0x00483360 | `driver_tick` | `Tick__8C_driverFUi` | 0x00157ba0 | CONFIRMED | the per-frame driver: runs the helpers, then applies speed limit, gas, brake and steer and calls the light and sound helpers |
| 0x00485ec0 | `driver_fill_start_values` | `FillStartValues__8C_driverFib` | 0x0015b590 | CONFIRMED | per-frame setup: cursors, speed jitter, speed limit, `driver_compute_radius`, the force limits, the car basis and position |
| 0x004861a0 | `driver_optimize_next_checkpoints` | `OptimizeNextCheckpoints__8C_driverFv` | 0x0015b8d0 | CONFIRMED | sets flag `0x4` when `dir·next < 0`; cuts a corner when the angle is above 0.7 and the distance² below 900 |
| 0x004863f0 | `driver_advance_checkpoint` | `NevimJakToNazvat__8C_driverFv` | 0x0015bc30 | CONFIRMED | advances the cursor past a reached or blocked node; a new random reach distance (`rand % 0x28 + 0x14`) / 100, clamped at 25.0 |
| 0x00486760 | `driver_compute_brake_onset` | `TakNaCemToUbrzdim__8C_driverFv` | 0x0015c190 | CONFIRMED | `sinf(angle) * g * 10` → `+0x38` / `+0x3c`; `0x4e6e6b28` = no corner |
| 0x00486830 | `driver_last_route_in_brake_path_tol` | `LastRouteInBrakePathPlusToleranciaLaskaASvoboda__8C_driverFf` | 0x0015c290 | CONFIRMED | accumulates node distances until they pass the braking distance plus a tolerance → `+0x1bc` |
| 0x004869e0 | `driver_ramp_brake_distance` | `MinSOld__8C_driverFf` | 0x0015c4a0 | CONFIRMED | ramps `+0x210` toward the braking distance (5 per s up, 15 per s down) |
| 0x00486a60 | `driver_last_route_in_brake_path` | `LastRouteInBrakePath__8C_driverFv` | 0x0015c530 | CONFIRMED | walks the nodes until the ramped distance → `+0x108`, `+0x178` |
| 0x00486b80 | `driver_compute_corner_speed_limit` | `MinV__8C_driverFv` | 0x0015c6a0 | CONFIRMED | minimum over the look-ahead of `sqrt(force * turn_cap / g)` → `+0x1c0`; 1000.0 start, 5.0 floor |
| 0x00486d70 | `driver_path_correction` | `PathCorrection__8C_driverFv` | 0x0015ca20 | CONFIRMED | nudges the target back onto the path or toward `seed_pos` |
| 0x004873d0 | `driver_lane_change_and_park` | `PrejezdZPruhuDoPruhuAParkovani__8C_driverFv` | 0x0015d9b0 | CONFIRMED | lane change and parking avoidance (node index change, the edge bit, `actor_is_possible_coll_all`); can force reverse |
| 0x00487a70 | `driver_predictive_sphere_lane_cross` | `PredvidaciKoulePripPrejezduMeziPruhy__8C_driverFv` | 0x0015e1a0 | CONFIRMED | projects the car onto the previous segment (gated on `mode_flags & 2`) and writes a lateral avoidance target to `car+0x20c0` |
| 0x00487f40 | `driver_predict_collision` | `PredvidaciKolize__8C_driverFf` | 0x0015e760 | CONFIRMED | builds the swept path and solves the time-to-collision quadratic against a moving obstacle; the largest helper |
| 0x00488e20 | `driver_reduce_speed_in_corner` | `SnizRychlostVzatacce__8C_driverFv` | 0x00160220 | CONFIRMED | curve-radius stop check `sqrt(force * dist * 0.85 / g)` against the speed → `+0x1c0`; returns "must brake" |
| 0x004891e0 | `driver_compute_brake_pedal` | `UtekloPraseZase__8C_driverFv` | 0x00160650 | CONFIRMED | overspeed → `brake_pedal` (stopping-distance formula, clamped to `+0xf4`..`+0xec`, divided by `+0xf8`) |
| 0x004893e0 | `driver_turn` | `Zatoc__8C_driverFv` | 0x001608b0 | CONFIRMED | the steering angle `+0x1e0` (a cross-product look-at, clamped to the car's maximum steer); clutch, handbrake and gear |
| 0x00489950 | `driver_do_lights` | `DoLights__8C_driverFUif` | 0x00160e50 | CONFIRMED | brake light, indicators and the headlight flash pattern on `car+0x20c` (§2a) |
| 0x00489c00 | `driver_do_sounds` | `DoSounds__8C_driverFUi` | 0x00161170 | CONFIRMED | pulses `car+0x51c` from the sound pattern (§1) |
| 0x004f57e0 | `driver_path_node_at` | - | - | - | `return begin + i * 0x70` on the `+0x160` vector; 2 callsites in `pursuit_event_tick`, both `lea ecx,[eax+0x160]`, each with `path_cursor` as the index. It sits in the `pursuit_event` unit as an out-of-line inline member |

### 3d. Free helpers

The five container helpers of the path vector (`S_car_node`, stride `0x70`). Each takes raw
pointers or the vector itself as `this`, and indexes no `C_driver` field.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00489cc0 | `std_vector70_size` | - | - | - | `(end - begin) / 0x70`; 46 callsites: `driver_tick`, `pursuit_event_save_game_load`, `std_vector70_insert_one`, `traffic_car_activate_elements` (33), `traffic_car_ai` (10) |
| 0x00489cf0 | `std_vector70_insert_n` | - | - | - | `insert(pos, n, val)`; 23 callsites in `driver_tick`, the `pursuit_event_*` orders, `opponent_*` and `roads_*` |
| 0x0048a010 | `std_vector70_uninit_copy` | - | - | - | `uninitialized_copy` over a range; called by `driver_tick` and `std_vector70_insert_one` |
| 0x0048a050 | `std_vector70_uninit_fill_n` | - | - | - | `uninitialized_fill_n(dest, count, val)`; called by `driver_tick` and `std_vector70_insert_one` |
| 0x0048a080 | `std_vector70_construct` | - | - | - | one guarded element copy; called by `driver_tick` and `std_vector70_insert_one` |

## 4. The traffic-light phase of a path node

**Path node `+0x68` points at a traffic-light phase.** Three consumers read it the same way:

```
driver_tick                : (node->+0x68 != NULL) && ((&g_driver_semaphore_states)[**(int**)(node+0x68) * 4] & 4) == 0
driver_advance_checkpoint  : the same expression, verbatim
pursuit_event_tick         : (node->+0x68 == NULL) || ((&g_pursuit_event_semaphore_states)[**(int**)(node+0x68) * 4] & 4) != 0
```

The table addresses hold byte-identical 10-dword data, `[1,1,1,1,1,4,4,4,4,2]`: one file-local
`static const` array emitted once per translation unit. Four copies exist:
`g_driver_semaphore_states` (this unit), `g_pursuit_event_semaphore_states` (the `pursuit_event`
unit), `g_railway_semaphore_states` and `g_game_semaphore_states`. The index is the `int` that
`+0x68` points at, a **traffic-light phase 0..9**, and the entry's bits are the three bulbs
`game_tick_traffic_signals` drives: bit `1` red, bit `2` orange, bit `4` green
(`structs/game.md` §4d). Phases 0..4 are red, 5..8 green and 9 orange.

The symbols are the array bases (element [0]): every reference is `test byte [reg*4 + <symbol>], 4`
(`f6 04 85 2c c2 63 00 04`), so the index scales from the symbol itself; the dword before each
table (`0x0063c228` = `0x00594700`, `0x0063cd30` = `0x0060b1a0`) is unrelated data with no
reference.

**The two writers** are the path builders `traffic_car_activate_elements` and
`pursuit_event_recompile_path`. Both point `+0x68` at one of three phase variables:

| Target | Chosen when |
|---|---|
| `C_game+0x19c` | the node's X offset to its neighbours is larger than its Z offset |
| `C_game+0x1a0` | otherwise, the Z offset being at least as large |
| a bridge actor's `act_state` (`+0x138`) | the next node is a bridge slot of `g_traffic_bridge_slots_begin` ¹ |

¹ The `C_traffic_car::S_BRIDGE` vector, `{actor *, ushort node}`; the match is on the next node's
nav index.

The two `C_game` phases are one city-wide light cycle seen from the two crossing axes: `game_init`
seeds them 0 and 5 and `game_tick` advances both together, so one axis is green while the other is
red. A bridge's `act_state` is 1 while it cycles (red) and 7 at rest (green), `structs/bridge.md`.

**Only bit `4` (green) is tested:**

* **not green** → `driver_tick` forces `target_speed` to 0 once the node is inside the braking
  distance, and `driver_advance_checkpoint` zeroes `throttle`: the car stops at the node.
* **`pursuit_event_tick`'s blocked-driver timer** (latching `blocked_latch`, `+0x298`, after 2 s
  with no predicted obstacle) runs on to 15 s when an obstacle is predicted and the node has no
  light or a green one. At a red or orange light the timer is held, so a car waiting at the light
  is never reported blocked.

## 5. The path-vector container helpers

`C_driver` is non-polymorphic, so it contributes no vtable. Its code is followed by the MSVC
container helpers of the path vector (§3d). **None of them is a `C_driver` method**, decided by
measurement: the three loop helpers take raw pointers and index no struct field at all, and the
size and insert helpers take the vector itself as `this`. They are shared template code: `size`
and `insert_n` serve the traffic, racing, road-search and pursuit code as well (46 and 23
callsites), while the three loop helpers are reached only from `driver_tick` and
`std_vector70_insert_one`.

Two more members of the same instantiation live elsewhere: `std_vector70_insert_one`
(`0x004f5800`, `insert(pos, 1, val)`, 6 callsites in `pursuit_event_dispatch_move_to_vehicle` and
`pursuit_event_tick`) and `std_vector70_erase_range` (`0x004f5f10`), both in the `pursuit_event`
unit; compiler-generated members are emitted beside their first user, not in the owning class's
unit.

Two functions just below the unit belong to the `C_door` unit: `door_set_open_angle` and
`i3d_copy_bound_volume` (`structs/door.md` §3).

## 6. PC vs PS2

| Item | PC | PS2 |
|---|---|---|
| `C_driver` size | `0x29c` | `0x320` |
| driven car | `+0x10` | `+0x12c` |
| `S_car_node` stride | `0x70` | `0x90` |
| nav index in the node | `+0x60` | `+0x84` |
| light-phase pointer in the node | `+0x68` | `+0x80` |
| light bits written by `DoLights` | `car+0x20c` | `car+0x5f8` |
| sound byte written by `DoSounds` | `car+0x51c` | `car+0x229c` |

- `opponent_fill_path_node`'s twin is `FillObjzNode__10C_opponentFR8S_vectorfR8S_vectorP10S_car_node`,
  which also supplies the `S_car_node` type name.
- `std_vector70_insert_n` corresponds to `vector<S_car_node>::insert`.
- `vehicle_set_brake` (`0x00517600`) is `C_Vehicle::SetBrake`, which fixes `+0xdc` as the brake
  pedal.
- The 10-entry table of §4 is the PS2 `semaphore_states` (40 bytes, indexed from 0 by `IsSemaPoint`
  and `TickSemaphores__6C_gameFv`).
- The PS2 builders `RecompilePath__12C_sledovackaFi`, `ActivateElems__13C_traffic_carFbi` and
  `AI__13C_traffic_carFUi` point the node's phase pointer at `C_game+0x1e4` / `+0x1e8` (PC `+0x19c`
  / `+0x1a0`) by the same axis test, or at `+0x140` of a bridge list entry.
- `driver_path_node_at` has no PS2 symbol: the PS2 build keeps the accessor inline.
- `C_driver` is non-polymorphic on both builds: `__vt__8C_driver` does not occur in the PS2 decomp.

## 7. Related

- `structs/vehicle.md` — the driven car's `C_Vehicle` and its gas, brake and steer setters.
- `structs/car.md` — the light-state word `car+0x20c` and the headlight byte `car+0x20bc`.
- `structs/pursuit_event.md` — the S_ELEMENT record that owns a driver at `+0x04`.
- `structs/racing.md` — the racing producers of the path records.
- `structs/game.md` §4d — the traffic-light cycle.
- `police-system/artifacts/boosting.md` — `speed_factor` and the model-level boost.

## 8. Open items

None.
