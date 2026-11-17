# `C_car` — the car entity

**Created** 2026-07-23 10:48 +0200 · **Modified** 2026-11-17 10:39 +0100

The car ENTITY: the seat / owner system, model-level (CARINIT) switching, collision, damage and
deform, save and load, the per-frame AI and update. Its physics and drivetrain half is the embedded
`C_Vehicle` (`structs/vehicle.md`) at **`car+0x70`**: two classes, two prefixes, `car_` for the
entity and `vehicle_` for the sub-object. A function in the `0x0046xxxx` cluster whose `this`
dereferences a `C_car` field (the seat vector `+0x210c`, the level vector `+0x20f4`) is a `C_car`
method whatever its other offsets look like. `C_car` is not part of the `C_actor` ← `C_human` line.

| | |
|---|---|
| PC prefix | `car_` (`car_extended_` and `car_database_` are other classes) |
| PC kind (`+0x10`) | 4 (`structs/entity_kind.md` §1) |
| PC vtable | `0x0063bc08` (41 slots) and the `C_Vehicle` vtable at `+0x70`, `0x0063bcac` |
| PC base class | `C_actor` (`structs/actor.md`), with `C_Vehicle` embedded at `+0x70` |
| PC instance size | `0x21f4` (`operator_new(0x21f4)` in `mission_create_actor` `case 4`) |
| PC constructor | `car_ctor` @ `0x00462af0` |
| PC translation unit | `[0x00462a90, 0x00474cc0)`, guard to guard (§5); the replication functions sit in `[0x0059ab90, 0x0059d9d0)` (§8) and six out-of-line inline members elsewhere |
| PS2 class | `C_car` (tag `5C_car`), 89 methods: 77 twins, 2 inlined into one, 10 with none (§9a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band, `+0x1fb8..+0x21f4`. `+0x00..+0x70` is the `C_actor` base
(`structs/actor.md` §1) and `+0x70..+0x1fb8` the embedded `C_Vehicle` (`structs/vehicle.md` §1,
`0x1f48` bytes); code outside those classes reaches their fields with the `C_car` spelling, which
§4 lists. The typedef holds every accessed offset of the band; the gaps no instruction touches are
under 4 bytes each and stay out.

```c
typedef struct C_car {                      // 0x21f4 bytes; PC offsets
    /* +0x0000 */ C_actor    actor;                  // [H] structs/actor.md
    /* +0x0070 */ C_Vehicle  vehicle;                // [H] structs/vehicle.md, 0x1f48 bytes
    /* +0x1fb8 */ float occupant_damage_sum;         // [H] occupant damage handed out this frame
    /* +0x1fbc */ uint8_t shadow_on;                 // [H] the cached state of car_set_shadow_enabled
    /* +0x1fc0 */ float ai_probe_local[2][3];        // [H] two model-space AI probe points, seeded by car_init
    /* +0x1fd8 */ float ai_probe_world[2][3];        // [H] the same two points in world space
    /* +0x1ff0 */ uint8_t last_hit_not_body;         // [H] the last shot hit an attached frame, not the body
    /* +0x1ff1 */ uint8_t sounds_enabled;            // [H] the car's sounds run; the scene-0x7b test
    /* +0x1ff2 */ uint8_t is_police_model;           // [H] the model name is POLI / POLCAD / POLIMFOR / POLIMTUD
    /* +0x1ff3 */ uint8_t unk_1ff3;                  // [U] stored 0 or restored; read by car_update, meaning not decoded
    /* +0x1ff4 */ float damage2_full_impulse;        // [H] impulse that empties pool 2 at 0 % resistance
    /* +0x1ff8 */ float damage2_min_impulse;         // [H] smallest impulse that touches pool 2
    /* +0x1ffc */ float damage2_max;                 // [H] maximum of damage pool 2, from the CARINIT block
    /* +0x2000 */ float damage2_resistance;          // [H] percentage resistance of pool 2
    /* +0x2004 */ float damage2;                     // [H] damage pool 2, current value
    /* +0x2008 */ uint8_t damage2_hit_latch;         // [H] pool 2 took a car_hit case-1 hit this frame
    /* +0x200c */ C_actor *last_struck_actor;        // [H] the actor this car last collided with
    /* +0x2010 */ uint32_t parking_start_ms;         // [H] the mission clock when the car was parked
    /* +0x2014 */ uint8_t tunnel_lights;             // [H] the car is in a tunnel sector; lamps forced on
    /* +0x2018 */ float climb_power_floor;           // [H] floor of climb_power_scale
    /* +0x201c */ float climb_power_scale;           // [H] power factor on a climb, copied into C_Vehicle+0x44
    /* +0x2020 */ float power_mass_ratio;            // [H] (+0x660 * +0x564) / (+0x6e0 * +0x69c)
    /* +0x2024 */ int32_t horn_yield_timer_ms;       // [H] ms since the horn or siren last sounded
    /* +0x2028 */ int32_t explosion_timer_ms;        // [H] time on the delayed-explosion fuse
    /* +0x202c */ int32_t explosion_delay_ms;        // [H] length of the delayed-explosion fuse
    /* +0x2030 */ uint8_t explosion_pending;         // [H] the delayed explosion is armed
    /* +0x2031 */ uint8_t show_damage;               // [H] the car-damage bar shows for this car
    /* +0x2032 */ uint8_t surface_push_applied;      // [H] the surface-0x2b push-back ran this frame
    /* +0x2033 */ uint8_t holds_mafia_car_slot;      // [H] the car holds a mafia pursuit-car slot
    /* +0x2034 */ uint8_t enter_protect;             // [H] the flag car_set_enter_protect stores
    /* +0x2035 */ uint8_t police_target_link;        // [M] set when the police link a pursuit target to this car
    /* +0x2036 */ uint8_t must_steal;                // [H] the lock must be forced before the car can be entered
    /* +0x2038 */ uint32_t effect_request_mask;      // [H] effect-request bits; the pursuit dispatchers set 0xC00
    /* +0x203c */ uint32_t damage_inhibit_mask;      // [H] damage-inhibit bits (script car_unbreakable)
    /* +0x2040 */ float physics_substep_cap_s;       // [H] upper bound on C_Vehicle physics_substep_s
    /* +0x2044 */ uint8_t is_police_car;             // [H] the car serves the police
    /* +0x2045 */ uint8_t is_gang_car;               // [H] a traffic car of a gang element
    /* +0x2046 */ uint8_t replaced_by_wreck;         // [H] exploded; its wreck took its place
    /* +0x2048 */ int32_t lock_timer_ms;             // [H] countdown that keeps the car locked
    /* +0x204c */ uint8_t lock_timer_locked;         // [H] the lock_timer_ms lock is applied
    /* +0x204d */ uint8_t broken_motor;              // [H] the engine cannot be started
    /* +0x2050 */ int32_t stall_phase_ms;            // [H] length of the current broken_motor stall phase
    /* +0x2054 */ int32_t stall_timer_ms;            // [H] time in the current stall phase
    /* +0x2058 */ uint8_t stall_engine_runs;         // [H] 1 = the engine runs in this phase, 0 = stalled
    /* +0x2059 */ uint8_t keep_alive_offscreen;      // [H] sounds and automatic lights run while not visible
    /* +0x205c */ int32_t model_variant;             // [H] the two digits before .i3d in the model name
    /* +0x2060 */ float prev_pos[3];                 // [H] the position at the end of the last frame
    /* +0x206c */ uint32_t path_ahead_state;         // [H] bit 0: the path-ahead cache is valid this frame
    /* +0x2070 */ uint8_t path_ahead_cached;         // [H] cached: no wheel on a drivable surface
    /* +0x2071 */ uint8_t driverless_roll;           // [H] the driver is dead; the car rolls on
    /* +0x2074 */ float driverless_steer;            // [H] the steer value of that roll
    /* +0x2078 */ float fuel_dummy_pos[3];           // [H] the FUEL dummy, model space; zero without one
    /* +0x2084 */ float tank_dummy_pos[3];           // [H] the TANK dummy, model space
    /* +0x2090 */ float tank_hit_radius;             // [H] |bbox extent| * 0.5 of the TANK frame
    /* +0x2094 */ int32_t tank_hit_count;            // [H] fuel-tank hits left
    /* +0x2098 */ int32_t tank_hit_max;              // [H] fuel-tank hit threshold
    /* +0x209c */ uint8_t player_is_driver;          // [H] the player sits in seat 0
    /* +0x20a0 */ float boarding_driver_pos[3];      // [H] where a traffic driver stands at the seat-0 door
    /* +0x20ac */ int32_t ms_since_collision;        // [H] ms since the last collision
    /* +0x20b0 */ uint8_t on_sidewalk;               // [H] a wheel is on the sidewalk while the police track the car
    /* +0x20b4 */ float net_velocity_scale;          // [H] the step-clamp ratio; scales a snapshot's velocities
    /* +0x20b8 */ int32_t condition_channel;         // [H] which meter car_get_condition_fraction reports
    /* +0x20bc */ uint8_t auto_headlights_suppressed; // [H] stops car_update's automatic headlights
    /* +0x20bd */ uint8_t is_traffic_car;            // [H] an ambient traffic car
    /* +0x20be */ uint8_t lane_cross_sphere_on;      // [H] the lane-cross sphere is set
    /* +0x20c0 */ float lane_cross_sphere_pos[3];    // [H] the chase driver's predicted lane-cross point
    /* +0x20cc */ char model_name[0x20];             // [H] the uppercased model name
    /* +0x20ec */ int32_t model_level;               // [H] the current CARINIT level
    /* +0x20f0 */ uint8_t levels_alloc;              // [H] allocator byte of the level vector
    /* +0x20f4 */ S_CARINIT *levels_begin;           // [H] the vehicles.bin levels, stride 0x113c
    /* +0x20f8 */ S_CARINIT *levels_end; // [H] one past the last level; (end - begin) / 0x113c is the level count
    /* +0x20fc */ S_CARINIT *levels_cap; // [H] end of the level storage; car_load_model_level_tables grows it when it equals the end
    /* +0x2100 */ uint32_t claim_mask;               // [H] per-seat claimed bits
    /* +0x2104 */ uint32_t enable_mask;              // [H] per-seat enabled bits
    /* +0x2108 */ uint8_t seats_alloc;               // [H] allocator byte of the seat vector
    /* +0x210c */ S_car_seat *seats_begin;           // [H] one record per door, stride 0xa0
    /* +0x2110 */ S_car_seat *seats_end; // [H] one past the last seat; (end - begin) / 0xa0 is the seat count
    /* +0x2114 */ S_car_seat *seats_cap; // [H] end of the seat storage; car_seat_vector_insert_n grows it through the header +0x2108
    /* +0x2118 */ I3D_frame *shadow_object;          // [H] the car's shadow object
    /* +0x211c */ int32_t brake_hold_count;          // [H] brake-hold reference count
    /* +0x2120 */ uint32_t playback_controls;        // [H] recorder key dword 9: revs, steer, gear
    /* +0x2124 */ uint32_t playback_wheel_contacts;  // [H] recorder key dword 10: one bit per wheel
    /* +0x2128 */ float playback_prev_pos[3];        // [H] the previous interpolated playback position
    /* +0x2134 */ uint8_t pending_dropouts_alloc;     // [H] allocator byte of the pending drop-out queue
    /* +0x2138 */ void *pending_dropouts_begin;       // [H] 8-byte part drop-outs pending for the recorder
    /* +0x213c */ void *pending_dropouts_end; // [H] one past the last pending drop-out; (end - begin) >> 3 is the count
    /* +0x2140 */ void *pending_dropouts_cap; // [H] end of the queue's storage; car_drop_out_part grows it through the header +0x2134
    /* +0x2144 */ uint8_t unread_2144;               // [U] copy of C_actor flag bit 1; no reader (.text scan)
    /* +0x2148 */ uint8_t horn_yield;                // [H] yield to a car sounding its horn behind
    /* +0x2149 */ uint8_t horn_yield_side;           // [H] the side of that swerve
    /* +0x214a */ uint8_t destroy_motor;             // [H] the engine cuts out repeatedly while driving
    /* +0x214b */ uint8_t destroy_motor_brake;       // [H] the full-brake latch of destroy_motor
    /* +0x214c */ uint8_t remove_at_game_done;       // [H] car_game_done removes the car
    /* +0x2150 */ int32_t ai_party_claim;            // [H] 0 free, 666 / 1332 held by an AI car party
    /* +0x2154 */ uint8_t front_hit_static;          // [H] front impulse with no actor struck, this tick
    /* +0x2155 */ uint8_t rear_hit_static;           // [H] rear impulse with no actor struck, this tick
    /* +0x2156 */ uint8_t front_hit;                 // [H] any front impulse, this tick
    /* +0x2157 */ uint8_t rear_hit;                  // [H] any rear impulse, this tick
    /* +0x2158 */ uint8_t reset_flag;                // [H] car_reset stores param == 0
    /* +0x2159 */ uint8_t in_water;                  // [H] deep-water contact; script car_inwater
    /* +0x215a */ uint8_t collisions_off;            // [H] script car_invisible
    /* +0x215b */ uint8_t struck_actor;              // [H] car_on_collision_with_actor sets it
    /* +0x215c */ uint32_t net_broadcast_timer;      // [H] time since the last state broadcast
    /* +0x2160 */ uint32_t unread_2160;              // [U] one store of 0; no reader (.text scan)
    /* +0x2164 */ float net_pos[3];                  // [H] dead-reckoning: position
    /* +0x2170 */ float net_velocity[3];             // [H] dead-reckoning: linear velocity
    /* +0x217c */ float net_quat[4];                 // [H] dead-reckoning: orientation
    /* +0x218c */ float net_correction[3];           // [H] dead-reckoning: correction accumulator
    /* +0x2198 */ uint32_t net_flags;                // [H] replication bits (§8)
    /* +0x219c */ float net_snap_threshold;          // [H] dead-reckoning: snap-back threshold
    /* +0x21a0 */ uint8_t unread_21a0;               // [U] one byte store of 0; no reader (.text scan)
    /* +0x21a1 */ uint8_t net_collision_hold_ms;     // [H] countdown that holds net_flags 0x80
    /* +0x21a4 */ uint32_t unread_21a4;              // [U] one store of 0; no reader (.text scan)
    /* +0x21a8 */ float net_previous_velocity[3];     // [H] dead-reckoning: net_velocity before the last message, then advanced with it
    /* +0x21b4 */ float net_velocity_change_rate[3];  // [H] dead-reckoning: per second, subtracted from net_velocity between messages
    /* +0x21c0 */ float net_received_velocity[3];    // [H] the velocity a message carried
    /* +0x21cc */ uint32_t net_msg_time;             // [H] the message timestamp
    /* +0x21d0 */ uint32_t net_window;               // [H] the extrapolation window
    /* +0x21d4 */ uint8_t damage_queue_alloc;        // [H] allocator byte of the collision-damage queue
    /* +0x21d8 */ void **damage_queue_begin;         // [H] heap {impulse, position, u32} records (§8)
    /* +0x21dc */ void **damage_queue_end; // [H] one past the last record pointer; (end - begin) >> 2 is the count
    /* +0x21e0 */ void **damage_queue_cap; // [H] end of the queue's storage; car_queue_collision_damage_record grows it when it equals the end
    /* +0x21e4 */ float net_correction_velocity[3];  // [H] dead-reckoning: correction velocity
    /* +0x21f0 */ uint8_t net_broadcast_counter;     // [H] raises net_flags 0x100 on one call in three
} C_car;
```

**Why two AI probe points.** `ai_probe_local` holds two model-space points that `car_init` seeds;
`car_get_ai_probe_point` (PS2 `GetAIPos(bool)`) takes a bool that selects one of the two, transforms
it by the body matrix into `ai_probe_world` and returns it. `car_get_navigation_corners` reads the
first point's height `+0x1fc4` as the AI ground height.

**The four hit bytes `+0x2154..+0x2157`.** `car_on_collision_apply_damage` sets them from the
direction of the impulse on this car (three `S_vector::AngleTo` tests in the disassembly). Only a
horizontal impulse counts (45° to 135° from world up). Against `axis_forward` (`car+0xd28`) an angle
above 2.355 rad sets `front_hit`, and `front_hit_static` when no actor was struck; below 0.78 rad it
sets `rear_hit`, and `rear_hit_static` when no actor was struck; a side hit sets none. The 6.0 and 10
constants of that body are debris counts. `car_ai_tick` clears all four every tick, so each lives for
one tick. `driver_tick` fixes which pair is the front: it tests `front_hit` / `front_hit_static` only
with a forward gear (below 2.0 speed) to start its obstacle manoeuvre, and `rear_hit` /
`rear_hit_static` only with `gear == -1`, to end a reverse or to drop the current path point.
`pursuit_event_tick` reads the two static bytes as one test, "hit something that is no actor", and
gives the chase car up on it after 20 s of chase; `racing_predict_car_collision` skips its path
prediction when `front_hit` is set and the last struck actor is a car. The save stream holds only
`front_hit_static` and `rear_hit_static`.

**The CARINIT block `+0x1ff4..+0x200b`.** Six dwords that move as one: `car_set_model_level` copies
them in from the level record `+0x1124`, preserving `damage2` (`+0x2004`) around the copy so a boost
swap keeps the damage; `car_load_model_level_tables` copies the car's six into each level record;
`car_save_game_save` and `car_save_game_load` stream them as one block (record `+0x63`). `car_init`
sets `damage2_full_impulse` 27.7, `damage2_min_impulse` 3.0, `damage2_max` 100.0,
`damage2_resistance` 10.0, `damage2` 100.0 and `damage2_hit_latch` 0. `car_unpack_save_record`
restores four of them from `vehicles.bin`: the two impulses from the file's km/h values (110.526 and
18.4211 in every shipped block, `def-files-docs/vehicles-bin.md` §4) times the mass term `car+0x6e0`
/ 3.6, `damage2_max` and `damage2_resistance` as stored; `car_pack_save_record` writes the inverse.
A hit of impulse `d` lowers `damage2` by `d × (1 − damage2_resistance / 100) / damage2_full_impulse ×
damage2_max`. `car_on_collision_apply_damage` touches the pool only above `damage2_min_impulse`, and
`damage2_hit_latch` lets `car_hit` case 1 take one hit per frame.

**`model_name` and the vectors.** `model_name` is a 32-byte buffer (`car_get_model_name_ptr` returns
its address) that `car_init` uppercases and matches against the four police models. The level
vector holds one `S_CARINIT` per `vehicles.bin` boost level (`car_get_model_level_count` =
`(end − begin) / 0x113c`); the seat vector one `0xa0`-byte record per door (§6d); the `pending_dropouts` vector the 8-byte part drop-outs pending for the recorder, which `car_drop_out_part` appends
through `g_car_current_dropout_record_queue` (`car_ai_tick` points it at `+0x2134`) and
`car_track_record_key` (§6a) removes, the last one per call; the damage queue the records of §8.

**The other array fields.** `prev_pos`, `fuel_dummy_pos`, `tank_dummy_pos`, `boarding_driver_pos`,
`lane_cross_sphere_pos`, `playback_prev_pos`, `net_pos`, `net_velocity`, `net_correction`,
`net_previous_velocity`, `net_received_velocity` and `net_correction_velocity` are one 3D vector each, x, y, z.
`net_quat[4]` is one quaternion, the body orientation. `net_velocity_change_rate[3]` has one entry per axis:
`car_apply_net_event_msg` copies `net_velocity` into `net_previous_velocity` and stores the message's velocity, then
sets each entry to `(net_previous_velocity - net_velocity)` divided by the seconds since the last message, times 0.12
when the message raises contact flag `0x20` on any entry of `car+0xd24` and times 0.8 otherwise (all three are 0 when
no time has passed). While the time since that message is under `net_window`, `car_net_apply_remote_state` subtracts
`dt * rate * 0.001` from `net_velocity` and from `net_previous_velocity`, which carries the measured change on.

## 2. Field evidence

Measured by a census of every `this + N` access in the `car_*` functions joined with a displacement
census of every offset of the band over the whole `.text`; `(address)` marks a function that takes
the field's address. The band is reached only by `car_*` code and the named external users.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0000 | actor | H | `car_ctor` (`actor_ctor`) | - | the base; `structs/actor.md` |
| +0x0070 | vehicle | H | `car_ctor` (the `C_Vehicle` ctor) | - | the embedded sub-object; `structs/vehicle.md` and §4a |
| +0x1fb8 | occupant_damage_sum | H | `car_update` | `car_extended_update`, `car_update` | zeroed each frame, then the sum of the pending occupant damage (seat `+0x9c`) `car_update` hands to the occupants (vtable `+0x7c`, hit type 5); `car_extended_update` passes it on, above 0.1, to every passenger in `+0x2248` (`structs/car_extended.md`) |
| +0x1fbc | shadow_on | H | `car_game_init`, `car_set_shadow_enabled` | `car_set_shadow_enabled` | - |
| +0x1fc0 | ai_probe_local | H | `car_init` | `car_get_ai_probe_point` (address), `car_get_navigation_corners`, `car_is_possible_collision` (address), `opponent_game_init`, `traffic_car_activate_elements` | §1 ¹ |
| +0x1fd8 | ai_probe_world | H | `car_init` | `car_get_ai_probe_point` (address) | written through its address by `car_get_ai_probe_point` |
| +0x1ff0 | last_hit_not_body | H | `car_game_init`, `car_hit` | `car_get_surface_material_id` | `car_hit` case 0: 1 when the shot resolves to an attached frame (an occupant, to which the hit is forwarded), 0 on the car's own model frame; `car_get_surface_material_id` returns -1 while it is 1 |
| +0x1ff1 | sounds_enabled | H | `car_enable_sounds_for_scene`, `car_explosion`, `car_game_init`, `car_on_act_state_changed`, `car_reset_seats_and_routes`, `car_reset` | `car_explosion` (address), `car_game_init` (address), `car_on_act_state_changed`, `car_update` | - |
| +0x1ff2 | is_police_model | H | `car_ctor`, `car_init` | `car_ctor` (address), `garage_game_done`, `police_manager_tick` | ² |
| +0x1ff3 | unk_1ff3 | U | `car_game_init`, `car_save_game_load` | `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save`, `car_update` | only 0 or the restored byte; while it is set `car_update` keeps `C_Vehicle+0xce4` bit `0x1000` off for a player-driven car; meaning not decoded |
| +0x1ff4 | damage2_full_impulse | H | `car_init`, `car_unpack_save_record` | `car_hit`, `car_load_model_level_tables` (address), `car_on_collision_apply_damage`, `car_pack_save_record`, `car_save_game_load` (address), `car_set_model_level` (address) | the divisor of every pool-2 subtraction; §1, "The CARINIT block" |
| +0x1ff8 | damage2_min_impulse | H | `car_init`, `car_unpack_save_record` | `car_on_collision_apply_damage`, `car_pack_save_record` | `damage2` is touched only when `mass × impulse / mass` exceeds it; §1 |
| +0x1ffc | damage2_max | H | `car_init`, `car_unpack_save_record` | `car_game_init`, `car_get_condition_fraction`, `car_hit`, `car_on_collision_apply_damage`, `car_pack_save_record`, `car_reset` | §4b |
| +0x2000 | damage2_resistance | H | `car_init`, `car_unpack_save_record` | `car_hit`, `car_on_collision_apply_damage`, `car_pack_save_record` | a hit `d` counts as `d − r × 0.01 × d`, the shape of the engine pool's `+0x2ac` and the gearbox pool's `+0x2e4` |
| +0x2004 | damage2 | H | `car_explosion`, `car_game_init`, `car_hit`, `car_init`, `car_on_collision_apply_damage`, `car_reset`, `car_set_model_level` | `car_get_condition_fraction`, `car_hit`, `car_on_collision_apply_damage`, `car_set_model_level` | ³ |
| +0x2008 | damage2_hit_latch | H | `car_hit`, `car_init`, `car_on_collision_apply_damage`, `car_update` | `car_hit` | set by `car_hit` cases 1 / 2 and `car_on_collision_apply_damage`; `car_hit` case 1 (`d = 4 × mass`) subtracts only while it is 0; `car_update` clears it every frame |
| +0x200c | last_struck_actor | H | `car_game_init`, `car_on_collision_apply_damage`, `car_reset_seats_and_routes`, `car_reset`, `pursuit_event_tick`, `racing_predict_car_collision` | `pursuit_event_tick`, `racing_predict_car_collision` | `car_on_collision_apply_damage` stores the struck actor when there is one; both readers clear it after reading |
| +0x2010 | parking_start_ms | H | `car_game_init`, `car_mark_parking_start_time` | `car_get_parked_duration`, `crt_old_sbh_decommit_pages` (address) | - |
| +0x2014 | tunnel_lights | H | `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_update` | `car_update` | on a change of sector (`car+0x19c`) `car_update` sets it on entering `sector tunelmnh` with the engine on, and clears it, `auto_headlights_suppressed` and by day lamp bits `0x0ff00000` on any other; while it is set and the car visible it forces `0x10000` and `0x0ff00000` |
| +0x2018 | climb_power_floor | H | `car_game_init`, `car_save_game_load`, `driver_tick` | `car_save_game_save`, `car_update`, `crt_old_sbh_alloc_block` (address) | 1.0 from `car_game_init` |
| +0x201c | climb_power_scale | H | `car_game_init`, `car_save_game_load`, `car_update` | `car_save_game_save`, `car_update` | 1.0, or on a climb (`car+0xd2c` > 0.17) below `power_mass_ratio` 3.4 `(3.4 / ratio − 1) × (min(y, 0.29) − 0.17) × 8.333 + 1`; at least `climb_power_floor`; copied into `car+0xb4` (`C_Vehicle+0x44`) |
| +0x2020 | power_mass_ratio | H | `car_game_init`, `car_set_model_level` | `car_update` | - |
| +0x2024 | horn_yield_timer_ms | H | `car_game_init`, `car_reset`, `car_save_game_load`, `car_update` | `car_save_game_save`, `car_update` | 0 while the horn or siren sounds (not on a traffic car); under 3000 ms (with no player driver, or above 18 speed) `car_update` sets `horn_yield` and `horn_yield_side` on every traffic car ahead within `min(speed / 28, 1) × 50` and 0.5 rad that is neither police nor `is_gang_car` |
| +0x2028 | explosion_timer_ms | H | `car_game_init`, `car_reset`, `car_save_game_load`, `car_update` | `car_extended_update`, `car_save_game_save`, `car_update` | `car_update` adds the step while `explosion_pending` (no cutscene, not dead) and past `explosion_delay_ms` calls `car_explosion(0)`; `car_extended_update` runs the same test first and throws out its truck passengers |
| +0x202c | explosion_delay_ms | H | `car_game_init`, `car_hit`, `car_save_game_load`, `car_update` | `car_extended_update`, `car_save_game_save`, `car_update` | `rand % 6000 + 10000` when the engine pool `+0x2a4` drops under 0.0001 (`car_update`), `rand % 100` after the last fuel-tank hit (`car_hit` case 0) |
| +0x2030 | explosion_pending | H | `car_explosion`, `car_game_init`, `car_hit`, `car_reset`, `car_save_game_load`, `car_update` | `car_extended_update`, `car_on_act_state_changed`, `car_save_game_save`, `car_update` | set with `explosion_delay_ms`, cleared by `car_explosion`; while it is set the act state does not change |
| +0x2031 | show_damage | H | `car_game_init`, `car_save_game_load`, `car_set_show_damage` | `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save`, `player_show_car_damage_bar` | ⁴ |
| +0x2032 | surface_push_applied | H | `car_game_init`, `car_on_body_surface_contact`, `car_update` | `car_game_init` (address), `car_on_body_surface_contact` | set once the push-back of a surface-`0x2b` contact is applied this frame; `car_update` clears it every frame |
| +0x2033 | holds_mafia_car_slot | H | `car_game_init`, `car_reset`, `car_save_game_load`, `traffic_car_spawn_mafia_pursuit_car` | `car_game_done`, `car_game_done` (address), `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save` | set by `traffic_car_spawn_mafia_pursuit_car`; `car_game_done` then calls `traffic_car_release_mafia_car_slot` |
| +0x2034 | enter_protect | H | `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load`, `car_set_enter_protect` | `car_save_game_save`, `car_set_owner`, `car_update` | - |
| +0x2035 | police_target_link | M | `car_game_init`, `car_save_game_load`, `police_manager_link_target_to_vehicle` | `car_ai_tick`, `car_game_init` (address), `car_on_collision_apply_damage`, `car_react_to_and_report_collision`, `car_save_game_load` (address), `car_save_game_save` | ⁵ |
| +0x2036 | must_steal | H | `car_game_init`, `car_save_game_load`, `car_set_must_steal`, `human_do_throw_occupant_from_car`, `human_movement`, `police_action_dispatch_officer_commands`, `police_action_tick`, `traffic_car_activate_elements` | `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save`, `player_do_action_button_on_foot` | ⁶ |
| +0x2038 | effect_request_mask | H | `car_game_init`, `car_save_game_load`, `car_set_effect_mask`, `pursuit_event_dispatch_escape_vehicle`, `pursuit_event_dispatch_move_to_vehicle`, `pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_remove_vehicle`, `traffic_car_ai` | `car_save_game_save`, `car_update` | - |
| +0x203c | damage_inhibit_mask | H | `car_game_init`, `car_save_game_load`, `car_set_damage_inhibit_mask`, `opponent_game_init`, `traffic_car_activate_elements` | `car_save_game_save`, `car_update` | - |
| +0x2040 | physics_substep_cap_s | H | `car_game_init`, `car_save_game_load`, `pursuit_event_remove_vehicle` | `car_save_game_save`, `car_update` | `car_update` caps `C_Vehicle+0xca0` `physics_substep_s` (car `+0xd10`: 0.0421, or 0.028 with a player driver) from above by it; 1000.0 from `car_game_init`, 1.0 from `pursuit_event_remove_vehicle`, both above every sub-step, so no known writer makes the cap bite |
| +0x2044 | is_police_car | H | `car_game_init`, `car_save_game_load`, `police_action_tick`, `police_manager_spawn_patrol_car_reinforcements_at_radius`, `police_manager_try_dispatch_pursuit_for_target`, `traffic_car_adopt_car_with_new_driver`, `traffic_car_game_init` | `car_save_game_save`, `car_update`, `player_do_action_button_on_foot`, `player_try_use_car`, `police_manager_npc_list_erase`, `police_manager_npc_list_tick` | ⁷ |
| +0x2045 | is_gang_car | H | `car_game_init`, `car_save_game_load`, `traffic_car_game_init` | `car_game_init` (address), `car_hit`, `car_save_game_load` (address), `car_save_game_save`, `car_update`, `player_do_action_button_on_foot` | `traffic_car_game_init` stores 1 for an element with `is_gang` (`structs/traffic_car.md`); `car_hit` case 0 then calls `traffic_car_on_collision_try_register_assault_on_gang_car` instead of `traffic_car_kill_occupant_on_hit`, `car_update` does not make it yield, and the on-foot hijack (`0x7d9`) refuses it |
| +0x2046 | replaced_by_wreck | H | `car_explosion`, `car_game_init`, `car_reset`, `car_save_game_load` | `car_explosion` (address), `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save`, `car_update` | `car_explosion` sets it on a non-traffic car; `car_update` then holds act state 2 every frame while the wreck, a new `C_car`, takes its place |
| +0x2048 | lock_timer_ms | H | `car_ai_tick`, `car_game_init`, `car_reset`, `car_save_game_load`, `human_intern_throw_self_from_car`, `human_use_actor` | `car_ai_tick`, `car_save_game_save` | while above 0 `car_ai_tick` lowers it by the step and locks the car once (`vehicle_lock(1)`); at 0 it unlocks it once |
| +0x204c | lock_timer_locked | H | `car_ai_tick`, `car_game_init`, `car_reset`, `car_save_game_load` | `car_ai_tick`, `car_save_game_save` | 1 while `lock_timer_ms` holds the lock |
| +0x204d | broken_motor | H | `car_game_init`, `car_reset`, `car_save_game_load`, `car_set_broken_motor` | `car_ai_tick`, `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save` | script `car_breakmotor` (case `0x95`): `car_ai_tick` runs a random stall cycle (`stall_phase_ms`, `stall_timer_ms`, `stall_engine_runs`), so the car coasts to a halt and its engine dies |
| +0x2050 | stall_phase_ms | H | `car_ai_tick`, `car_game_init`, `car_save_game_load` | `car_ai_tick`, `car_save_game_save` | `rand % 200 + 100`, or `rand % 300 + 100` when `stall_engine_runs` was set |
| +0x2054 | stall_timer_ms | H | `car_ai_tick`, `car_game_init`, `car_save_game_load` | `car_ai_tick`, `car_save_game_save` | += step; past `stall_phase_ms` it restarts, a new phase is drawn and `stall_engine_runs` toggles |
| +0x2058 | stall_engine_runs | H | `car_ai_tick`, `car_game_init`, `car_save_game_load` | `car_ai_tick`, `car_save_game_save` | while it is 0 `car_ai_tick` sets `C_Vehicle+0x1f24`, brakes 0.3 and stops the engine below 1.0 speed |
| +0x2059 | keep_alive_offscreen | H | `camera_begin_mode`, `camera_end_mode`, `car_game_init`, `car_reset` | `car_game_init` (address), `car_update` | while it is set `car_update` keeps the car's sounds and automatic lights going though its frame is not visible |
| +0x205c | model_variant | H | `car_init` | `garage_add_car_by_actor`, `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_spawn_mafia_pursuit_car`, `traffic_car_take_over_cruising_car` | `atoi` of the two digits that end the model file name before `.i3d` |
| +0x2060 | prev_pos | H | `car_game_init`, `car_update` | `car_game_init` (address), `car_recompute_velocity`, `car_update` (address) | seeded from the actor position; `car_update` stores the position at the end of each frame and `car_recompute_velocity` derives the velocity as `(pos − prev_pos) / step` |
| +0x206c | path_ahead_state | H | `car_ctor`, `car_game_init`, `car_is_point_in_path_ahead`, `car_save_game_load`, `car_update` | `car_is_point_in_path_ahead`, `car_save_game_save` | bit 0 = the cache is valid this frame; `car_update` clears the dword every frame |
| +0x2070 | path_ahead_cached | H | `car_is_point_in_path_ahead`, `car_save_game_load` | `car_is_point_in_path_ahead`, `car_save_game_save` | 1 = no wheel touches a drivable surface (flag 8, not detached, surface not 1 or 10); the test then refuses |
| +0x2071 | driverless_roll | H | `car_game_init`, `car_reset`, `car_save_game_load`, `car_set_owner`, `human_hit_in_car`, `traffic_car_kill_occupant_on_hit` | `car_ai_tick`, `car_game_init` (address), `car_is_usable`, `car_save_game_load` (address), `car_save_game_save`, `driver_do_lights`, `driver_do_sounds`, `human_do_throw_occupant_from_car`, `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_spawn_mafia_pursuit_car` | set when the driver is killed in the car; while it is set `car_ai_tick` steers by `driverless_steer` with power and brake 0; `car_set_owner` clears it when seat 0 is vacated |
| +0x2074 | driverless_steer | H | `car_game_init` | `car_ai_tick` | `(rand % 200 − 100) × 0.01`, −1.0 to 0.99 |
| +0x2078 | fuel_dummy_pos | H | `car_init` | `pumper_can_pump` (address), `pumper_start_refuel` (address) | §4a |
| +0x2084 | tank_dummy_pos | H | `car_init` | `car_hit` (address) | §4a |
| +0x2090 | tank_hit_radius | H | `car_init` | `car_hit`, `car_init` | §4a |
| +0x2094 | tank_hit_count | H | `car_game_init`, `car_hit`, `car_reset`, `car_save_game_load`, `car_set_tank_hit_count` | `car_hit`, `car_save_game_save`, `car_update` | §4a |
| +0x2098 | tank_hit_max | H | `car_ctor`, `car_save_game_load`, `car_set_tank_hit_count` | `car_game_init`, `car_reset`, `car_save_game_save`, `car_update` | §4a |
| +0x209c | player_is_driver | H | `car_game_init`, `car_refresh_player_is_driver_flag`, `car_set_owner` | `car_react_to_and_report_collision`, `car_set_owner`, `car_update`, `traffic_car_on_collision_try_register_assault_on_gang_car` | - |
| +0x20a0 | boarding_driver_pos | H | `car_game_init`, `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_deactivate_element` | `car_game_init` (address), `car_is_possible_collision` | while a traffic driver plays the get-in or get-out animation of seat 0, `traffic_car_ai` stores where he stands (`car_seat_get_anim_placement(0, ...)`); otherwise every writer stores (0, −900000, 0), none; `car_is_possible_collision` tests a radius-1.2 sphere there while y > −100000 |
| +0x20ac | ms_since_collision | H | `car_game_init`, `car_on_collision_apply_damage`, `car_update` | `car_update` | 1000 from `car_game_init`; `car_update` adds the step, `car_on_collision_apply_damage` restarts it; under 1000 (or with `horn_yield`) `car_update` sets `+0xd54` bit `0x400` |
| +0x20b0 | on_sidewalk | H | `car_ai_tick`, `car_game_init`, `car_save_game_load` | `car_save_game_save`, `police_manager_tick` | while `police_target_link` and `C_game+0x2da0` are set: 1 when any wheel contact has surface 3 (sidewalk), after `police_manager_register_sidewalk_offence_for_target` |
| +0x20b4 | net_velocity_scale | H | `car_ai_tick`, `car_game_init` | `car_post_reduced_state_net_event_msg`, `car_post_state_net_event_msg` | 1.0, or 83 / step when `car_ai_tick` clamps the step to 83 ms; §8 |
| +0x20b8 | condition_channel | H | `car_game_init`, `car_set_condition_channel`, `opponent_game_init`, `pursuit_event_tick` | `car_get_condition_fraction`, `car_set_condition_channel`, `pursuit_event_tick` | §4b |
| +0x20bc | auto_headlights_suppressed | H | `car_game_init`, `car_save_game_load`, `car_set_auto_headlights_suppressed`, `car_update`, `driver_do_lights` | `car_save_game_save`, `car_update` | ⁸ |
| +0x20bd | is_traffic_car | H | `car_ctor`, `car_save_game_load`, `car_set_is_traffic_car` | `ai_group_get_closest_usable_car`, `car_ai_tick`, `car_ctor` (address), `car_deactivate`, `car_explosion`, `car_game_init`, `car_hit`, `car_on_body_surface_contact`, `car_react_to_and_report_collision`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load` (address), `car_save_game_save`, `car_update`, `player_do_action_button_on_foot` | ⁹ |
| +0x20be | lane_cross_sphere_on | H | `car_ctor`, `car_game_init`, `driver_predictive_sphere_lane_cross`, `driver_reset` | `car_ctor` (address), `car_game_init` (address), `car_is_possible_collision` | gates the `lane_cross_sphere_pos` test of `car_is_possible_collision` |
| +0x20c0 | lane_cross_sphere_pos | H | `driver_predictive_sphere_lane_cross` | `car_is_possible_collision` | stored through `add edx, 0x20c0`; `car_is_possible_collision` tests a radius-0.9 sphere there |
| +0x20cc | model_name | H | - | `car_ctor` (address), `car_explosion` (address), `car_get_model_name_ptr` (address), `car_init` (address), `car_load_model_level_tables` (address), `car_set_shadow_enabled` (address), `garage_add_car_by_actor` (address), `human_use_actor` (address), `pedestrian_manager_player_in_taxi` (address), `player_try_use_car` (address) | every access takes its address; `car_get_model_name_ptr` returns it |
| +0x20ec | model_level | H | `car_game_init`, `car_set_model_level` | `car_get_model_level`, `car_save_game_save` | `police-system/artifacts/boosting.md` |
| +0x20f0 | levels_alloc | H | `car_ctor` | `car_load_model_level_tables` (address) | - |
| +0x20f4 | levels_begin | H | `car_ctor`, `car_dtor`, `car_load_model_level_tables` | `car_dtor`, `car_game_done`, `car_get_model_level_count`, `car_load_model_level_tables`, `car_set_model_level` | - |
| +0x20f8 | levels_end | H | `car_ctor`, `car_dtor`, `car_game_done`, `car_load_model_level_tables` | `car_game_done`, `car_get_model_level_count`, `car_load_model_level_tables`, `car_set_model_level` | - |
| +0x20fc | levels_cap | H | `car_ctor`, `car_dtor`, `car_load_model_level_tables` | `car_load_model_level_tables` | - |
| +0x2100 | claim_mask | H | `car_deactivate`, `car_game_done`, `car_game_init`, `car_on_act_state_changed`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load`, `car_set_enter_protect`, `car_update` | `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_save`, `car_set_enter_protect`, `car_update` | §6d |
| +0x2104 | enable_mask | H | `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_set_enter_protect1` | `car_reset`, `car_set_enter_protect1`, `car_set_enter_protect`, `car_set_owner`, `car_update` | §6d |
| +0x2108 | seats_alloc | H | `car_ctor` | `car_init` (address) | - |
| +0x210c | seats_begin | H | `car_ctor`, `car_dtor` | `car_deactivate`, `car_dtor`, `car_explosion`, `car_game_done`, `car_game_init`, `car_get_owner`, `car_get_seat_property`, `car_init`, `car_is_seat_left`, `car_on_act_state_changed`, `car_on_body_surface_contact`, `car_on_collision_apply_damage`, `car_refresh_player_is_driver_flag`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load`, `car_seat_get_anim_placement`, `car_seat_get_attach_positions`, `car_seat_index_of_usable`, `car_set_enter_protect`, `car_set_owner`, `car_update`, `human_can_enter_car`, `human_can_exit_car_free_space`, `human_climb_out_of_car`, `human_do_climb_in_car_lr`, `human_do_throw_occupant_from_car`, `human_eject_out_of_car`, `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update`, `opponent_game_init`, `traffic_car_adopt_car_with_new_driver`, `traffic_car_game_init` | §6d |
| +0x2110 | seats_end | H | `car_ctor`, `car_dtor` | `car_deactivate`, `car_explosion`, `car_game_done`, `car_game_init`, `car_get_owner`, `car_get_seat_property`, `car_init`, `car_is_seat_left`, `car_on_act_state_changed`, `car_refresh_player_is_driver_flag`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load`, `car_seat_get_attach_positions`, `car_seat_index_of_usable`, `car_set_enter_protect`, `car_set_owner`, `car_update` | - |
| +0x2114 | seats_cap | H | `car_ctor`, `car_dtor` | - | - |
| +0x2118 | shadow_object | H | `car_game_done`, `car_game_init`, `car_init` | `car_game_done`, `car_game_init`, `car_set_shadow_enabled`, `car_update`, `traffic_car_activate_elements` | - |
| +0x211c | brake_hold_count | H | `car_adjust_brake_hold_count`, `car_deactivate`, `car_game_init`, `car_save_game_load`, `human_death`, `opponent_do_recover`, `opponent_tick_recover`, `pumper_choose_next_action`, `pumper_save_game_load`, `pumper_start_refuel`, `racing_activate_racers`, `racing_tick` | `car_adjust_brake_hold_count`, `car_ai_tick`, `car_save_game_save`, `human_death`, `opponent_do_recover`, `opponent_tick_recover`, `pumper_choose_next_action`, `pumper_save_game_load`, `pumper_start_refuel` | - |
| +0x2120 | playback_controls | H | `car_ctor`, `car_game_init`, `car_recompute_velocity`, `car_track_apply_pose_at_clock` | `car_recompute_velocity` | key dword 9 of `car_track_record_key`: revs u16, steer byte, gear + 1 in the top byte; `car_recompute_velocity` reads two words: revs / 65535 × `max_revs`, steer byte / 127 × `car+0x6f4`, gear (§6a) |
| +0x2124 | playback_wheel_contacts | H | `car_ctor`, `car_game_init`, `car_track_apply_pose_at_clock` | `car_recompute_velocity` | key dword 10, one bit per wheel (at most 8); `car_recompute_velocity` sets or clears contact flags `0x1000010` from it |
| +0x2128 | playback_prev_pos | H | `car_track_apply_pose_at_clock` | `car_track_apply_pose_at_clock` | written only on the interpolating path, never reset elsewhere; §6a |
| +0x2134 | pending_dropouts_alloc | H | `car_ctor` | `car_ai_tick` (address) | `car_drop_out_part` reaches the vector through `g_car_current_dropout_record_queue` |
| +0x2138 | pending_dropouts_begin | H | `car_ctor`, `car_dtor` | `car_dtor`, `car_game_init`, `car_track_record_key` | - |
| +0x213c | pending_dropouts_end | H | `car_ctor`, `car_dtor`, `car_game_init`, `car_track_record_key` | `car_game_init`, `car_track_record_key` | - |
| +0x2140 | pending_dropouts_cap | H | `car_ctor`, `car_dtor` | - | - |
| +0x2144 | unread_2144 | U | `car_ai_tick`, `car_game_init` | - | a byte copy of `C_actor` flag bit 1 (`actor+0x04 >> 1 & 1`); no reader (.text scan) |
| +0x2148 | horn_yield | H | `car_game_init`, `car_update` | `car_ai_tick`, `car_update` | set on a traffic car ahead by another car's `car_update`, which clears it on itself each frame; `car_ai_tick` then brakes above 0.2 speed and steers aside by 0.2 |
| +0x2149 | horn_yield_side | H | `car_update` | `car_ai_tick` | 0 steers −0.2, else +0.2 |
| +0x214a | destroy_motor | H | `car_game_init`, `car_save_game_load`, `car_set_destroy_motor` | `car_ai_tick`, `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save` | script `car_setdestroymotor` (case `0x6a`); the effect is the brake latch `destroy_motor_brake`, the engine stays on |
| +0x214b | destroy_motor_brake | H | `car_ai_tick`, `car_game_init`, `car_reset`, `car_save_game_load` | `car_ai_tick`, `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save` | while `destroy_motor` is set it switches on above 4.0 speed and off below 1.0; while it is on the car brakes fully |
| +0x214c | remove_at_game_done | H | `car_ctor`, `car_explosion`, `car_save_game_load`, `car_set_remove_at_game_done`, `garage_game_done`, `police_manager_apply_state`, `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_spawn_mafia_pursuit_car`, `traffic_car_take_over_cruising_car` | `car_extended_update`, `car_game_done`, `car_save_game_save`, `game_done` | every access is a byte |
| +0x2150 | ai_party_claim | H | `ai_fight_manager_disband_followship`, `ai_fight_manager_new_car_party`, `ai_fight_manager_tick`, `car_ctor`, `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load` | `ai_fight_manager_disband_followship`, `ai_group_get_closest_usable_car`, `car_save_game_save` | 0 free, 666 held by a fight-manager car party, 1332 held and still blocked after the party disbands (the disband subtracts 666 once); `ai_group_get_closest_usable_car` offers only cars below 666 |
| +0x2154 | front_hit_static | H | `car_ai_tick`, `car_game_init`, `car_on_collision_apply_damage`, `car_save_game_load` | `car_game_init` (address), `car_save_game_load` (address), `car_save_game_save`, `driver_tick`, `pursuit_event_tick` | §1, "The four hit bytes" |
| +0x2155 | rear_hit_static | H | `car_ai_tick`, `car_game_init`, `car_on_collision_apply_damage`, `car_save_game_load` | `car_save_game_save`, `driver_tick`, `pursuit_event_tick` | §1 |
| +0x2156 | front_hit | H | `car_ai_tick`, `car_game_init`, `car_on_collision_apply_damage` | `driver_tick`, `racing_predict_car_collision` | §1 |
| +0x2157 | rear_hit | H | `car_ai_tick`, `car_game_init`, `car_on_collision_apply_damage` | `driver_tick` | §1 |
| +0x2158 | reset_flag | H | `car_explosion`, `car_game_init`, `car_reset_seats_and_routes`, `car_reset`, `car_save_game_load`, `car_update`, `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_spawn_mafia_pursuit_car`, `traffic_car_take_over_cruising_car` | `car_save_game_save`, `car_update` | - |
| +0x2159 | in_water | H | `car_game_init`, `car_on_body_surface_contact`, `car_reset` | `car_game_init` (address), `car_on_body_surface_contact`, `program_process_command` | ¹⁰ |
| +0x215a | collisions_off | H | `car_game_init`, `car_save_game_load`, `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `opponent_do_recover`, `opponent_tick_recover`, `program_process_command`, `pursuit_event_remove_vehicle`, `pursuit_event_tick` | `car_game_init` (address), `car_on_body_surface_contact`, `car_save_game_load` (address), `car_save_game_save` | ¹⁰ |
| +0x215b | struck_actor | H | `car_game_init`, `car_on_collision_with_actor`, `car_reset_seats_and_routes`, `car_reset`, `car_update` | `car_game_init` (address), `car_on_body_surface_contact` | - |
| +0x215c | net_broadcast_timer | H | `car_net_snapshot_state`, `car_update` | `car_update` | `car_update` adds `g_net_session+0x98` and, past the session setting `+0x14`, calls `car_broadcast_state_net_event_msg(0)` and resets it |
| +0x2160 | unread_2160 | U | `car_net_snapshot_state` | - | one store of 0; no reader (.text scan) |
| +0x2164 | net_pos | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` (address) | §8 |
| +0x2170 | net_velocity | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` (address), `car_update` | §8 |
| +0x217c | net_quat | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_apply_net_event_msg` (address), `car_net_apply_remote_state`, `car_net_snapshot_state` (address), `car_update` | §8 |
| +0x218c | net_correction | H | `car_apply_net_event_msg`, `car_net_snapshot_state` | `car_apply_net_event_msg` (address), `car_net_apply_remote_state`, `car_update` | §8 |
| +0x2198 | net_flags | H | `car_apply_net_event_msg`, `car_broadcast_state_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state`, `car_on_collision_with_actor`, `car_update` | `car_apply_net_event_msg`, `car_broadcast_state_net_event_msg`, `car_net_apply_remote_state`, `car_on_collision_with_actor`, `car_post_reduced_state_net_event_msg`, `car_post_state_net_event_msg`, `car_update` | the bits in §8 |
| +0x219c | net_snap_threshold | H | `car_net_snapshot_state` | `car_net_apply_remote_state`, `car_net_snapshot_state` | §8 |
| +0x21a0 | unread_21a0 | U | `car_net_snapshot_state` | - | one byte store of 0; no reader (.text scan) |
| +0x21a1 | net_collision_hold_ms | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_net_apply_remote_state` | §8 |
| +0x21a4 | unread_21a4 | U | `car_net_snapshot_state` | - | one store of 0; no reader (.text scan) |
| +0x21a8 | net_previous_velocity | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_apply_net_event_msg`, `car_net_apply_remote_state` | §8 |
| +0x21b4 | net_velocity_change_rate | H | `car_apply_net_event_msg`, `car_net_snapshot_state` | `car_apply_net_event_msg`, `car_net_apply_remote_state` | §8 |
| +0x21c0 | net_received_velocity | H | `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_net_apply_remote_state`, `car_net_snapshot_state` (address) | §8 |
| +0x21cc | net_msg_time | H | `car_apply_net_event_msg`, `car_net_snapshot_state` | `car_apply_net_event_msg`, `car_net_apply_remote_state` | §8 |
| +0x21d0 | net_window | H | `car_apply_net_event_msg` | `car_net_apply_remote_state` | §8 |
| +0x21d4 | damage_queue_alloc | H | `car_ctor` | `car_get_collision_damage_records` (address), `car_queue_collision_damage_record` (address) | §8 |
| +0x21d8 | damage_queue_begin | H | `car_ctor`, `car_dtor` | `car_apply_net_event_msg`, `car_clear_collision_damage_records`, `car_dtor`, `car_post_collision_damage_net_event_msg` | §8 |
| +0x21dc | damage_queue_end | H | `car_clear_collision_damage_records`, `car_ctor`, `car_dtor` | `car_apply_net_event_msg`, `car_clear_collision_damage_records`, `car_post_collision_damage_net_event_msg`, `car_queue_collision_damage_record` | - |
| +0x21e0 | damage_queue_cap | H | `car_ctor`, `car_dtor` | `car_queue_collision_damage_record` | - |
| +0x21e4 | net_correction_velocity | H | `car_apply_net_event_msg`, `car_net_apply_remote_state`, `car_net_snapshot_state` | `car_apply_net_event_msg` (address), `car_net_apply_remote_state` | §8 |
| +0x21f0 | net_broadcast_counter | H | `car_broadcast_state_net_event_msg`, `car_net_snapshot_state` | `car_broadcast_state_net_event_msg` | every access is a byte; §8 |

¹ `opponent_game_init` and `traffic_car_activate_elements` each read one component of the first
point.
² `car_ctor` clears it; `car_init` sets it when the uppercased model name matches one of `POLI`,
`POLCAD`, `POLIMFOR`, `POLIMTUD` (`basic/artifacts/vehicle-movement.md` §5.1). `police_manager_tick`
reads it, and `garage_game_done` never banks such a car
(`basic/artifacts/garage/storing-cars-in-the-garage.md`). A different field from `is_police_car`.
³ `car_hit` and `car_on_collision_apply_damage` subtract damage from it and clamp at 0;
`car_get_condition_fraction` channel 2 reads it against `damage2_max`. Nothing else reads it, and
nothing selects channel 2 (§4b).
⁴ Written only by `car_set_show_damage`, the script command `showcardamage` (dispatcher case
`0xd5`, six shipped calls); its one reader is `player_show_car_damage_bar`, so the car-damage bar
appears only for cars a script marked.
⁵ `car_on_collision_apply_damage` reports to `police_manager_register_collision_offence_for_target`
only while it is set.
⁶ Script `car_muststeal` (dispatcher case `0xc5`). Also written 1 by `traffic_car_activate_elements`
and by `police_action_tick` / `police_action_dispatch_officer_commands` when a stopped chase car's
crew is ordered out (event `0xf`); cleared by `human_movement` and
`human_do_throw_occupant_from_car`. `player_do_action_button_on_foot` reads it three times and takes
its ordinary enter-car path only while it is 0.
⁷ The task seed; read by the police-car sighting and assault path. `police_action_tick` clears it
just before handing the car to `traffic_car_adopt_car_with_new_driver`, so a released chase car stops
being a police car.
⁸ `car_update`'s automatic headlight block runs only while it is 0. Script command `carlight_main`
sets it when it forces the lights off, so the per-frame night-time re-enable cannot undo it.
⁹ Written only by `car_ctor`, `car_set_is_traffic_car` and `car_save_game_load`. `car_ai_tick` runs
its ambient-traffic block on it; `car_deactivate` clears a traffic car's script light overrides
(`0x0ff00000`) when `C_game+0x2d44` says day.
¹⁰ `in_water` is set by `car_on_body_surface_contact` on deep water and read by script command
`car_inwater` (dispatcher case `0xca`). `collisions_off` is written by script command `car_invisible`
(case `0x129`) alongside `vehicle_set_dynamic_colls_off` and `car_set_enter_protect`: the three writes
that together make a car non-solid, non-enterable and still visible.

## 3. Method family

77 of the 89 PS2 `C_car` methods are twinned, 2 are inlined into a PC function and 10 have none
(§9a). The six recorder slots (§6a), the force-feedback members and the replication functions (§8)
are PC-only.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00462a90 | `crt_static_init_guard_00462a90` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00462ab0 | `crt_atexit_register_00462ab0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x00462ad0 | `car_static_state_init` | `g_car_ff_prev_velocity` | - | - | - | The TU's CRT static initialiser, reached through the `crt_static_init_guard_00462a90` pair. Zeroes `g_car_ff_prev_velocity` (`0x00655ab0`), the previous car-space velocity `car_update`'s force-feedback impact block differences, which is why it must start at zero |
| 0x00462af0 | `car_ctor` | instance | `__ct__5C_carFv` | 0x00129cd0 | CONFIRMED | `actor_ctor` then the `C_Vehicle` ctor; installs BOTH MI vtables `0x0063bc08`/`0x0063bcac`, stamps kind `+0x10 = 4`, empties the four owned vectors, seeds `+0x2098` from `rand()` |
| 0x00462c20 | `car_scalar_deleting_dtor` | instance | - | - | - | Vtable slot 16 of `0x0063bc08`; an MSVC-only shape |
| 0x00462c40 | `car_dtor` | instance | `__dt__5C_carFv` | 0x00129e50 | CONFIRMED | Re-installs both MI vtables, `hud_radar_remove_car`, frees the four owned vectors, chains the `C_Vehicle` dtor and `actor_dtor` |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00464f40 | `car_set_has_driver` | `SetDriver__5C_carFb` | 0x0012d2b0 | CONFIRMED | One-line byte setter on `car+0x3b0`. The field bridge is proven from a CONFIRMED twin, not from position: `car_set_owner` writes `+0x209c = 0`, `+0x3b0 = 0` and `seat+0x40 = 0x7d3` when vacating; PS2 `SetOwner` writes `+0x2462 = 0`, `+0x2299 = 0` and `seat+0x74 = 0x7d3` at the same statement, and `+0x2299` is the byte PS2 `SetDriver` writes. All 7 callsites pass 1 |
| 0x00465a80 | `car_set_is_traffic_car` | `SetTrafficCar__5C_carFb` | 0x00130260 | CONFIRMED | One-line byte setter on `car+0x20bd` — the exact ambient-traffic gate `car_ai_tick` reads, and a field `car_save_game_save` round-trips. PS2 `+0x2460` |
| 0x00465ac0 | `car_get_model_level` | `GetActualLevel__5C_carFv` | 0x00130280 | CONFIRMED | Returns the current model level `car+0x20ec` (PS2 `car+0x242c`) |
| 0x0059d740 | `car_get_collision_damage_records` | - | - | - | One line, `return car + 0x21d4` |
| 0x005c8a10 | `car_set_must_steal` | - | - | - | script `car_muststeal` (case `0xc5`): `+0x2036` |
| 0x005c8a20 | `car_set_destroy_motor` | - | - | - | `+0x214a`: the engine cuts out repeatedly while driving |
| 0x005c8a30 | `car_set_broken_motor` | - | - | - | `+0x204d`: the engine cannot be started |
| 0x005c8a40 | `car_get_model_name_ptr` | - | - | - | returns the address of the model name `+0x20cc` |
| 0x005c8a50 | `car_set_show_damage` | - | - | - | script `showcardamage` (case `0xd5`): `+0x2031` |
| 0x005c8a90 | `car_set_remove_at_game_done` | - | - | - | `+0x214c`: `car_game_done` removes the car at the end of the game |
| 0x005c8aa0 | `car_set_auto_headlights_suppressed` | - | - | - | Writes `car+0x20bc`, the per-car flag that stops `car_update` from driving the headlight bit automatically at night. All 89 PS2 `C_car` methods were enumerated; none matches |
| 0x005c8ab0 | `car_set_effect_mask` | - | - | - | writes the effect-request mask `+0x2038` |
| 0x005c8ac0 | `car_set_damage_inhibit_mask` | - | - | - | writes the damage-inhibit mask `+0x203c` (script `car_unbreakable`) |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004254a0 | `car_get_fuel_tank_world_pos` | - | - | - | Transforms the `"TANK"` point at `+0x2084` by the body matrix `+0xdd4` (§1a) |
| 0x00462d30 | `car_init` | `Init__5C_carFP9I3D_frame` | 0x00129f40 | CONFIRMED | `actor_init`, the six tuning floats (27.7/3.0/10.0/100.0/100.0/0) in the same order on both builds, `vehicle_init`, then the THIRTEEN-entry `C_Vehicle` callback bank `+0x364..+0x398` in an order identical on both builds, the AI probe points `+0x1fc0`/`+0x1fcc`, and the `SM_`/`LG_` seat-dummy scan |
| 0x00463500 | `car_seat_get_anim_placement` | `GetPlacementFromTCK__5C_carFiiR8S_vectorR8S_vector` | 0x0012ac50 | CONFIRMED | Seat enter/exit placement from the `.tck` track. Byte-exact opening: seat index × `0xa0` (PS2 × `0xd0`), the `+0xc` present-flag early-out (PS2 `+0xc4`), then a 128-byte clip-name buffer. 304 vs 364 B |
| 0x00463630 | `car_seat_get_attach_positions` | `GetPlacementFromTCK1__5C_carFiR8S_vectorR8S_vectorb` | 0x0012adc0 | CONFIRMED | 5-arg `(this, seat, vec*, vec*, bool)`; guard seat-flag byte + `int == 0`; two matrix-transformed seat placements; returns 1/0. Seat array `+0x210c` stride `0xa0` (PS2 `+0x2444` stride `0xd0`) |
| 0x00463740 | `car_seat_index_of_usable` | `GetUSindex__5C_carFPCQ214C_using_object14S_using_object` | 0x0012af00 | CONFIRMED | Reverse lookup of a `C_using_object` to its seat index: backwards scan of `+0x210c` stride `0xa0` over the TWO usables each seat owns, `-1` sentinel. 96 vs 128 B. US = using-object |
| 0x004637a0 | `car_load_materials` | `LoadMaterials__5C_carFv` | 0x0012af80 | CONFIRMED | Gated on `g_num_surface_materials == 0`; builds the default `S_TSRFC` record (grip 0.35, scale 1.0, four 100.0 limits) and grows the raw-material vector (stride `0x30`). Rank forced after `car_seat_index_of_usable`. 656 vs 788 B (0.83) |
| 0x00463a30 | `car_load_particle_effects` | `LoadParticles__5C_carFv` | 0x0012b310 | STRUCTURAL | N→1: MSVC inlined `Load_Particle` into `LoadParticles`, so no single body agreement can be asserted. 336 + 2288 = 2624 against the PC's 2560 B is a ratio of 0.98, and the PC body carries the `s_Car_particle_0064b7cc` string the PS2 loop uses |
| - | - | `Load_Particle__5C_carFR14S_VEH_PARTICLEi` | 0x0012b460 | inlined | folded into `car_load_particle_effects`: `336 + 2288 = 2624` B against its 2560, and it carries the `s_Car_particle_0064b7cc` string the PS2 loop uses |
| 0x00464430 | `car_load_tyre_surfaces` | `LoadTyres__5C_carFv` | 0x0012c680 | CONFIRMED | STRING ANCHOR: `dtaOpen(tables\tyres.bin)` plus the same `version == 1` gate on both builds; per record it reads the raw floats, converts through `car_convert_tyre_surface` and appends with stride `0x5c`. 752 vs 824 B (0.91) |
| 0x00464720 | `car_convert_tyre_surface` | `ConvertTyres__5C_carFP7S_TSRFCP6S_SRFC` | 0x0012ca40 | CONFIRMED | 2-parameter signature exact. 18 source floats copied out, two scaled by 1000.0, two by 0.01745328 (deg→rad), and three max−min span fields precomputed at `+0x1c` / `+0x28` / `+0x48`, in the same order on both builds. 160 vs 140 B (1.14) |
| 0x004647c0 | `car_set_owner` | `SetOwner__5C_carFP7C_actorif` | 0x0012cad0 | CONFIRMED | Assign or vacate a seat occupant; seat array `+0x210c` stride `0xa0` |
| 0x00464c50 | `car_get_seat_property` | `GetSeatProperty__5C_carFiRbRbRbRb` | 0x0012d060 | CONFIRMED | 4-flag seat query; returns 1/0 |
| 0x00464ee0 | `car_get_owner` | `GetOwner__5C_carFi` | 0x0012d230 | CONFIRMED | The occupant of a seat |
| 0x00464f50 | `car_copy_carinit_from_actor` | `DuplicateFrom__5C_carFP7C_actor` | 0x0012d2d0 | CONFIRMED | `kind == 4` gate, actor `DuplicateFrom` (folded onto `stub_noop_args4_00461180` on the PC), `GetInit__9C_Vehicle`, `InitCar__9C_Vehicle`. The PS2 body's 4672-byte local is why the PC body opens with `__chkstk` |
| 0x00464fa0 | `car_save_data_chunk` | `SaveData__5C_carFP7C_chunk` | 0x0012d330 | CONFIRMED | `operator_new(0x111d)` (PS2 `0x124b`), `car_pack_save_record`, Compress, three guarded `dtaWrite` calls, free. Identical control flow |
| 0x00465090 | `car_load_data_chunk` | `LoadData__5C_carFP7C_chunk` | 0x0012d450 | CONFIRMED | Mirror of `SaveData`; the version dword must be 1; calls `car_unpack_save_record` at the exact statement PS2 `LoadData` calls `ReadSaveStruct` |
| 0x00465290 | `car_load_model_level_tables` | `LoadLevelTables__5C_carFv` | 0x0012d610 | CONFIRMED | Empties the level vector (`+0x20f8 = +0x20f4`), `dtaOpen`, uppercases each record name, matches it against the car model name at `+0x20cc` and appends `S_CARINIT` levels of stride `0x113c`. Ratio 0.10 — the PS2 inlined the field parser; the role is anchored on the level-vector pair `car_get_model_level_count` divides |
| 0x00465760 | `car_repair_position_on_ground` | `_RepairPosition__5C_carFb` | 0x0012ff50 | CONFIRMED | Downward collision probe at −200 then a −2000 retry, then cross-product re-orthonormalisation of the body axes `+0x30`/`+0x34`/`+0x38` with a 1e-08 tolerance. 800 vs 780 B (1.03) |
| 0x00465a90 | `car_get_model_level_count` | `GetLevelsCount__5C_carFv` | 0x00130270 | CONFIRMED | Returns the CARINIT model-level count `(+0x20f8 − +0x20f4)/0x113c` (PS2 caches it at `car+0x2434`). These are the `vehicles.bin` boosting levels |
| 0x00465ad0 | `car_set_model_level` | `SetActualLevel__5C_carFi` | 0x00130290 | CONFIRMED | Switches CARINIT model level: bound check, `InitCar` on the `C_Vehicle`, copy 6 fields, set `car+0x20ec`, recompute the ratio. Byte-structural match |
| 0x00465bd0 | `car_create_force_feedback_effects` | - | - | - | No PS2 rank between `SetActualLevel` and `GameInit`. Builds the FIVE shared car force-feedback effects (types 10/7 Condition, 0 Constant 220 ms, 2 Periodic ×2), handles in `g_ff_car_effect_*`. It creates no sound channel. See `structs/force_feedback.md` §8b |
| 0x00465d30 | `car_release_force_feedback_effects` | - | - | - | Releases all five car force-feedback effects, `0x0064b73c` included |
| 0x00465d90 | `car_game_init` | `GameInit__5C_carFv` | 0x001303d0 | CONFIRMED | Per-scene car init: `vehicle_enable_sounds(0)` → base-class actor `GameInit` → model level `-1` → `car_set_model_level(0)` → randomise |
| 0x00466680 | `car_game_done` | `GameDone__5C_carFv` | 0x00131370 | CONFIRMED | `EnableSounds(0)`, `vehicle_game_done`, `actor_game_done`, the seat `DelObject` loop, claim mask 0, `ShadowCarOn(0)`, `hud_radar_remove_car`. Forced 1:1 gap; 448 vs 568 B. This diff identified `vehicle_game_done` |
| 0x00466840 | `car_ai_tick` | `AI__5C_carFUi` | 0x001315b0 | CONFIRMED | The per-frame tick of every car: the traffic block gated on `+0x20bd` (brake ×0.7/×0.4, steer ±0.2 = `0xbe4ccccd`/`0x3e4ccccd`), the `rand()%200` / `%300+100` stall timer, then `vehicle_tick_drivetrain` unconditionally (`basic/artifacts/vehicle-movement.md` §3). Byte-exact |
| 0x00466b80 | `car_update` | `Update__5C_carFUi` | 0x00131a90 | CONFIRMED | The primary per-frame vtable tick: Reset-flag velocity guard, `0x53` dt-clamp, distance² LOD/cull flags `+0xd54`, inlined path-node management, tail `SetPower`/`SetBrake`/`IsCarUsable`. Head statement-for-statement against the PS2 |
| - | - | `UpdateShadow__5C_carFv` | 0x00133360 | inlined | inlined into `car_update`: its `car_set_shadow_enabled` call and the shadow placement (the `-1000.0` writes) sit there; the PS2 also calls it from `C_game::Tick` |
| 0x00468970 | `car_recompute_velocity` | - | - | - | Derives the velocity vec3 at `+0x1f7c` from the position delta ÷ dt; a `car_update` callee. Inlined into `Update` on the PS2 |
| 0x00468bc0 | `car_explosion` | `CarExplosion__5C_carFUi` | 0x00134270 | CONFIRMED | Turns the car's lights and indicators off by progressively clearing the light-flag word `car+0x20c` (= PS2 `car+0x5f8`) plus the `code*` callback `+0x3f8`; `car_update` calls it as `(0)` |
| 0x004699c0 | `car_deactivate` | `DeactivateCar__5C_carFUi` | 0x001354c0 | CONFIRMED | `0xf00fffff` mask clear |
| 0x00469ad0 | `car_reset` | `Reset__5C_carFfb` | 0x00135620 | CONFIRMED | Flag `+0x2158` from `param == 0`, Reset the `C_Vehicle`, clear route masks `+0x2100`/`+0x2104`, re-seat door states `0x7d3`/`0x7d4`/`0x7d9`, speed-gated (< 0.3), disable sounds. Statement-for-statement against the PS2 |
| 0x00469dd0 | `car_reset_seats_and_routes` | `ResetCar__5C_carFv` | 0x00135d80 | CONFIRMED | Byte-exact on every landmark: the PS2 writes `+0x2440 = 0xffffffff` and `+0x2470 = 0`, gates on speed `+0x8f0 < 0.3`, and re-stamps the seat door codes `0x7d9` / `0x7d3` / `0x7d4` in the same branch order with the index-1 special case at `+0x150`; the PC writes `+0x2104` / `+0x2034`, gates `+0x688 < 0.3`, same codes, special case at `+0xec`. The no-arg sibling of `car_reset`. 512 vs 1560 B — the PS2 inlined its vector maths |
| 0x00469fd0 | `car_is_point_in_path_ahead` | `ChcesMeSejmout__5C_carFRC8S_vectorf` | 0x00136400 | CONFIRMED | A run-over corridor test. The opening is byte-exact (bit 0 of `+0x206c` / PS2 `+0x23e4`, cached byte `+0x2070` / PS2 `+0x2465`, the wheel-contact scan over `+0xd24` / PS2 `+0xf8c` with flags `&8` and `&0x40000000` and surface `!= 1 && != 10`); the tail accepts the point only within ±4.0 laterally, inside `max(speed*1.8, range)`, a 1.6 rad cone and a `sin(theta)*d < 2.0` miss distance. It returns the car itself, so the caller gets a predicate and a handle at once. The PS2 name is slang and is deliberately not transliterated. 608 vs 912 B (0.67) |
| 0x0046a230 | `car_pack_light_state` | - | - | - | Folds `car+0x20c` bits `0x1` / `0x2` / `0x100` / `0x1000` / `0x10000` / `0x80` into a 6-bit byte. Bit map in `structs/vehicle.md` §2a; the exact inverse of `car_unpack_light_state`, which is what makes the mapping checkable |
| 0x0046a270 | `car_unpack_light_state` | - | - | - | The inverse, applied one bit at a time: each lamp is only touched when its override bit is clear (`0x8` guards the indicators, `0x800` the brake/reverse pair), and every applied change fires `car+0x3f8` = `C_Vehicle+0x388` — the evidence behind `structs/vehicle.md` §10a, that the slot is the WAKE-UP hook used here as a change notification rather than owned by the light field |
| 0x0046a3b0 | `car_set_shadow_receiver_frames` | `SetShadowsToMe__5C_carFb` | 0x00136790 | CONFIRMED | Frame enumeration by name match off the model at `+0x438` (PS2 `+0x710`), one vtable call with 0 per hit, temporary vector freed. 160 vs 196 B |
| 0x0046a450 | `car_set_transparency` | `SetTransparency__5C_carFf` | 0x00136860 | CONFIRMED | Enumerates the model frames, SKIPS glass panes (`+0x308`) and wheels whose contact carries flag `0x40` at `+0x123`, then filters on the frame magic `'OBJ_'` (`0x5f4a424f`) / `'SNGM'` (`0x4d474e53`) and calls the material vtable slot `+0x84` with the alpha. The same enumerate-then-per-frame idiom as `car_set_shadow_receiver_frames`. 352 vs 600 B (0.59) |
| 0x0046a5b0 | `car_get_surface_material_id` | `GetMaterialID__5C_carFP9I3D_frame` | 0x00136ac0 | CONFIRMED | BYTE-EXACT: the `-1` guard at `+0x1ff0`, the glass array `+0x308` stride `0x34` (type 3 → `0x18`, type 9 → `0x12`), the wheel array `+0xd24` → `0x13`, else `0x14`. True size 192 B (§3c) |
| 0x0046a670 | `car_hit` | `Hit__5C_carF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00136ba0 | CONFIRMED | The class's damage entry point. The 8-argument signature matches; the switch is on the `E_hit_type` parameter exactly as the PS2 symbol declares, and `actor_post_hit_event` is handed `param_2` plus `param_5..param_8` — the same forwarding the PS2 body performs. Damage arms walk `+0x308`, raise `+0x1e8 \| 2`, clamp negative part health and compute the impact point off `+0x484` with the body axes at `+0x510`. True size 4656 B against 5424 (0.86) |
| 0x0046b8a0 | `car_detect_wheel` | `DetectWheel__5C_carFiRC8S_vectorb` | 0x001380d0 | CONFIRMED | Wheel-contact distance classifier: mode 0 = 3D distance ref→contact, else matrix-transform 2D distance; identical 1e-08 / SQRT / 1.0 normalise; contact array `car+0xd24` = PS2 `+0xf8c` |
| 0x0046ba50 | `car_is_possible_collision` | `IsPossibleColl__5C_carFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x00138380 | CONFIRMED | The 11-parameter signature matches the PS2 symbol exactly once Ghidra's expansion of the two `S_vector&` arguments into six floats is accounted for. Sweeps the car along the velocity vec3 `+0x1f7c` that `car_recompute_velocity` writes, returning the earliest contact time and the struck actor; it returns at once while `+0x23c` (`C_Vehicle+0x1cc`, the summed forward-force cap, `structs/vehicle.md` §2) is below 0.0001. It also tests two optional spheres: `lane_cross_sphere_pos` (radius 0.9, while `lane_cross_sphere_on`) and `boarding_driver_pos` (radius 1.2, while its y is above −100000). Forced 2:2 with `car_on_collision_with_actor`. 1888 vs 3200 B (0.59) |
| 0x0046c1b0 | `car_on_collision_with_actor` | `Collision__5C_carFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00139000 | CONFIRMED | `+0x2198 \|= 1`, `actor_set_act_state(0)`, `+0x215b = 1`, then a switch on the STRIKER's kind `{2, 0x1b}`. True size 432 B vs 492 B (0.88) |
| 0x0046c360 | `car_get_collision_info` | `Actor_Col_Info__5C_carFPCvRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vectorPfRb` | 0x001391f0 | CONFIRMED | QUERIES the per-struck-kind collision-response parameters (contact radius 80.0, restitution −1.0/−1.2, normal, matrix); it does not apply the collision |
| 0x0046c6e0 | `car_react_to_and_report_collision` | `Call_Collision__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vectorf` | 0x00139480 | CONFIRMED | The FULL collision handler: the struck entity's `react_to_impact` vtable call **+** the `CoDelasTyCuraku` traffic-car assault **+** `CarCollision__C_shvestky` police-offence registration, impact-strength keyed. 6-arg |
| 0x0046c910 | `car_react_to_collision` | `Call_Collision1__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vector` | 0x00139790 | CONFIRMED | The consequence-free sibling: the same `react_to_impact` dispatch with a scaled and normalised direction, no offence and no assault. 5-arg |
| 0x0046cbd0 | `car_collision_filter_actor` | `Actor_Filter__5C_carFPCvPCvRb` | 0x00139a40 | CONFIRMED | BYTE-EXACT: the `&0x40` gate, kinds `{6,8,0x1e,0x16}` → 1, kind 4 plus flag bit `0x10000000`, kinds `{2,0x1b,0x13}` plus speed `+0x688 < 4.0` (PS2 `+0x8f0`). Registered by `car_init` in the `SetCallbackAF` slot, `C_Vehicle+0x370` |
| 0x0046cc40 | `car_on_collision_apply_damage` | `Was_Collision__5C_carFPCvRC8S_vectorRC8S_vectorRC8S_vectorfRC8S_vectorPCvPCvb` | 0x00139b20 | CONFIRMED | Body-diffed AND slot-proven (`SetCallbackWC`, `C_Vehicle+0x374`). Normalises the impact by mass `+0x6e0`, sets the four hit bytes `+0x2154..+0x2157` from the impulse direction (§1), walks the seat list injuring occupants through `car_get_owner`, and reports to `police_manager_register_collision_offence_for_target` when `+0x2035` is set. 9 arguments on both sides. 2272 vs 2920 B (0.78). Distinct from `car_react_to_and_report_collision`, which dispatches to the STRUCK entity |
| 0x0046d520 | `car_add_deform_scratch` | `Deform_Vehicle_Part__5C_carFPCvP9I3D_frameUiRC8S_vectorRC8S_vectorb` | 0x0013a690 | CONFIRMED | A one-line forward of all five arguments to `C_game::ScratchAdd`. `SetCallbackDVP` slot, `C_Vehicle+0x394`. 48 vs 44 B |
| 0x0046d550 | `car_clear_damage_decals` | `Vehicle_Repair__5C_carFPCv` | 0x0013a6c0 | CONFIRMED | `ScratchDeleteFromCar` plus `ClearFrameFromFireHole` on the root frame. `SetCallbackVR` slot, `C_Vehicle+0x398`. 48 vs 64 B |
| 0x0046d580 | `car_break_glass` | `Break_Glass__5C_carFPCviRC8S_vectorPCc` | 0x0013a700 | CONFIRMED | Glass array `+0x308` stride `0x34`, and the same NULL-defaults-the-sound branch on both builds (PC `s_lustr1_wav_0064b83c`). `SetCallbackBG` slot, `C_Vehicle+0x380`. 160 vs 128 B |
| 0x0046d620 | `car_wake_up` | `Wake_Up__5C_carFPCv` | 0x0013a780 | CONFIRMED | Forces the act state back to 0. `car_init` leaves the WU slot NULL on BOTH builds; `car_on_act_state_changed` installs THIS address exactly where PS2 `ChangeState` installs `Wake_Up` |
| 0x0046d630 | `car_on_body_surface_contact` | `Collision_Filter_Body__5C_carFPCvPCvRC8S_vectorRC8S_vector` | 0x0013a790 | CONFIRMED | Body-diffed AND slot-proven (`SetCallbackCFB`, `C_Vehicle+0x38c`). Above 0.1 speed a surface-`0x29` contact (a crash object, flags `&0xc0`) reflects the velocity `+0x1f7c`; surface `0x2b` pushes the car back once per frame (`surface_push_applied`); surface `0x28` sets `in_water` and, with the player driving, `C_game+0x2ad8` and `game_raise_water_fall_script_event`; lethal water `0x1f` sets `in_water` once, spawns particle `0x25`, plays sound `0xec` within 30 m and kills the occupants of a non-traffic car, none of it gated on the player. True size 2640 B against 2952 (0.89) |
| 0x0046e080 | `car_collision_filter_wheel` | `Collision_Filter_Wheel__5C_carFPCvPCvi` | 0x0013b320 | CONFIRMED | Hit-byte special case: `0x1f` (lethal water) → `contact+0xb4 = 0x1f`, return 0; `0x2b` (slippery) → return 0. Byte-exact |
| 0x0046e0c0 | `car_prepare_wheel_dropout` | `Prepare_DropOut_Wheel__5C_carFPCviRC8S_vectorPC8S_vector` | 0x0013b370 | CONFIRMED | BYTE-EXACT on both documented field bridges: wheel array `+0xd24` (PS2 `+0xf8c`), contact flag `+0x120 \|= 0x40000000` (PS2 `+0x1fc`), mass 10.0, impulse scale 0.1 (`0x3dcccccd`). `SetCallbackPDW` slot, `C_Vehicle+0x37c` |
| 0x0046e1b0 | `car_prepare_body_part_dropout` | `Prepare_DropOut__5C_carFPCviRC8S_vectorPC8S_vector` | 0x0013b490 | CONFIRMED | Body-part array `+0x308`, switch on the part type into a mass/kind pair from `{10.0/2, 2.0/5, 4.0/3, 7.0/6, 2.0/4, 20.0/7}`, then the shared spawner. `SetCallbackPD` slot, `C_Vehicle+0x378` |
| 0x0046e300 | `car_drop_out_part` | `Drop_Out__5C_carFP5C_carP9I3D_framePvii` | 0x0013bb40 | CONFIRMED | The shared back half of both drop-out paths and the **live** part-detach path: it allocates a `C_DropOut`, branches `if (selector == 1) InitWheel else InitBox`, and registers it as a temporary actor. Called from `car_prepare_wheel_dropout` and `car_prepare_body_part_dropout` at exactly the positions where the PS2 calls `Drop_Out` |
| 0x0046e660 | `car_pack_save_record` | `WriteSaveStruct__5C_carFP6STCarsP9S_CARINITb` | 0x0013bc40 | CONFIRMED | Called by `car_save_data_chunk` between the `0x111d` allocation and the compression, at PS2 `SaveData`'s exact `WriteSaveStruct` position. 4240 vs 3956 B (1.07) |
| 0x0046f6f0 | `car_unpack_save_record` | `ReadSaveStruct__5C_carFP6STCarsP9S_CARINITb` | 0x0013cbc0 | CONFIRMED | Called by `car_load_data_chunk` after the decompression and before `vehicle_apply_carinit_params`, at PS2 `LoadData`'s exact `ReadSaveStruct` position. 4192 vs 6024 B |
| 0x00470750 | `car_get_inverse_model_matrix` | `GetInvModelMatrix__5C_carFb` | 0x0013e350 | CONFIRMED | BYTE-EXACT: the `&0x20` dirty test before `UpdateWMatrixProc`, two 16-dword copies into `+0xdd4` and `+0xe14` (PS2 `+0x2e0` / `+0x320`), `S_matrix::Inverse`, return `+0xe14` |
| 0x004707c0 | `car_get_condition_fraction` | `ShowCarEnergy__5C_carFv` | 0x0013e420 | CONFIRMED | The death-processed latch `+0x5e` (PS2 `+0x62`), then a four-way switch on the channel field `+0x20b8` (PS2 `+0x2404`) over three damage pools and one deform zone (§1b; the build difference in arm 3 is in §6). It DISPLAYS nothing — it is the getter the HUD widget reads |
| 0x004709f0 | `car_get_ai_probe_point` | `GetAIPos__5C_carFb` | 0x0013e570 | CONFIRMED | BYTE-EXACT and self-validating: it transforms the two model-space probe points `car_init` seeds (`+0x1fc0` / `+0x1fcc`) into `+0x1fd8` / `+0x1fe4`; the PS2 quartet is `+0x22e0` / `+0x22f0` / `+0x2300` / `+0x2310`. 112 vs 112 B |
| 0x00470a60 | `car_set_enter_protect` | `SetEnterProtect__5C_carFb` | 0x0013e5e0 | CONFIRMED | BYTE-EXACT: seat array `+0x210c` stride `0xa0` (PS2 `+0x2444` stride `0xd0`), the claim/enable mask pair `+0x2100` / `+0x2104` (PS2 `+0x243c` / `+0x2440`), the seat list `+0xd90` stride `0x24` (PS2 `+0xfd0` stride `0x30`), the SAME door action codes `0x7d3` / `0x7d4` / `0x7d9` in the same branch order, and the same `index == 1` special case. Sibling of `car_set_enter_protect1` |
| 0x00470c50 | `car_set_enter_protect1` | `SetEnterProtect1__5C_carFib` | 0x0013e880 | CONFIRMED | Seat-claimed guard — one door at a time |
| 0x00470c90 | `car_get_wheel_coll` | `GetCarWheelColl__5C_carFPbi` | 0x0013e8c0 | CONFIRMED | Reads `(contact+0x120 >> 3) & 1` into an out bool array for `min(count, car+0x5b0)` contacts (array `car+0xd24` = PS2 `+0xf8c`, flag `+0x120` = PS2 `+0x1fc` bit 8) |
| 0x00470ce0 | `car_set_wheel_coll` | `SetCarWheelColl__5C_carFPbi` | 0x0013e950 | CONFIRMED | Sets or clears bit 8 of `contact+0x120` per an in bool array, the same loop as `GetCarWheelColl` |
| 0x00470d30 | `car_is_usable` | `IsCarUsable__5C_carFUi` | 0x0013ea10 | CONFIRMED | `(mask & 1)` → three `float > 0` tests (fuel `+0xd1c` included) + two byte flags + a `0x40000000` contact-array loop. Body-exact. Harvested from `Update`'s callee set |
| 0x00470ea0 | `car_get_navigation_corners` | `GetNavigationPoints__5C_carFfR8S_vectorR8S_vectorR8S_vectorR8S_vector` | 0x0013ec60 | CONFIRMED | The four footprint corners from the bbox fields `+0x254` / `+0x25c` / `+0x26c` / `+0x274` inflated by the margin argument, at the AI ground height `+0x1fc4`. 5-parameter signature exact. 192 vs 180 B |
| 0x00470f60 | `car_on_act_state_changed` | `ChangeState__5C_carFv` | 0x0013ed20 | CONFIRMED | Three-way dispatch on the act state `+0x1c` (PS2 `+0x5c`): state 2 teardown, state 1 installs `car_wake_up` in the WU callback slot, state 0 re-enables sounds and clears it. 304 vs 448 B |
| 0x00471090 | `car_set_shadow_enabled` | `ShadowCarOn__5C_carFb` | 0x0013eee0 | CONFIRMED | Idempotent through the cached flag `+0x1fbc`; `game_add` / `del_frame_for_shadow` around the shadow object at `+0x2118`. 176 vs 224 B |
| 0x00471140 | `car_get_actor_speed` | `GetActorSpeed__5C_carFR8S_vector` | 0x0013efc0 | CONFIRMED | Copies the car velocity vec3 `car+0x1f7c` — the field `car_recompute_velocity` writes — to the out parameter; the PS2 copies `car+0x460` |
| 0x00471160 | `car_enable_sounds_for_scene` | `SoundsChange__5C_carFi` | 0x0013efd0 | CONFIRMED | BYTE-EXACT including the magic scene id `0x7b` and the sounds-enabled byte `+0x1ff1` (PS2 `+0x2468`). 64 vs 80 B |
| 0x004711a0 | `car_mark_parking_start_time` | `SetTimeParking__5C_carFv` | 0x0013f020 | CONFIRMED | Copies the `C_game` mission clock into `car+0x2010` (PS2 `car+0x23a8` from `game+0xe54`) |
| 0x004711c0 | `car_get_parked_duration` | `GetTimeParking__5C_carFv` | 0x0013f040 | CONFIRMED | The exact inverse of the setter, reading the same two fields. 32 vs 24 B |
| 0x004711e0 | `car_prepare_showroom_model` | `PrepareCarToShow__5C_carFP9I3D_model` | 0x0013f060 | CONFIRMED | The menu / showroom model builder — it runs no simulation at all. Duplicates the model name buffer, enumerates frames twice (filters `0x10021` then `0x10001` against the `AXIS_` tag), rebuilds the frame vector, scales the bbox by 0.13, re-centres it and re-runs the model update vtable. Single `I3D_model*` parameter matches. 7072 vs 6180 B (1.14) |
| 0x00472d80 | `car_is_seat_left` | `IsSeatLeft__5C_carFi` | 0x001408c0 | CONFIRMED | BYTE-EXACT: the bounds test, then `0.0 < seat[i].f0` gives 0, else 1. 96 vs 124 B |
| 0x00472de0 | `car_set_roof_visible` | `OdjebajTuStrechu__5C_carFP9I3D_modelb` | 0x00140940 | CONFIRMED | BYTE-EXACT: the same frame enumeration, the same per-frame vtable call with `param == 0`, the same 1/0 return and the same vector teardown on both arms. Reached from `game_cheat_check` case `0xc` (JIZDA), confirmed in game |
| 0x00472ea0 | `car_refresh_player_is_driver_flag` | `RefreshActivePlayer__5C_carFv` | 0x00140a40 | CONFIRMED | BYTE-EXACT: the empty-seat guard, seat 0's occupant against the `C_game` active player (PC `game+0xe4`, PS2 `game+0x150`), the cached byte `+0x209c` (PS2 `+0x2462`) |
| 0x00472f00 | `car_set_force_feedback_gains` | - | - | - | Writes the five force-feedback gain weights `g_ff_car_gain_*` through `force_feedback_manager_set_gain`. It sets no volume. See `structs/force_feedback.md` §8b |
| 0x00472f40 | `car_save_game_get_size` | `SaveGameGetSize__5C_carFv` | 0x00140ae0 | CONFIRMED | `vehicle_get_save_state_size + 0x7b + actor_save_game_get_size` (the PS2 uses `0x80` for the constant term). This diff identified `vehicle_get_save_state_size` |
| 0x00472f60 | `car_save_game_save` | `SaveGameSave__5C_carFPPv` | 0x00140b20 | CONFIRMED | Record version 3; the base writer, then `vehicle_save_state`, then 43 field copies out of the `+0x1ff3..+0x215a` band and the six CARINIT dwords as one block (record `+0x63`); record size `0x7b`. 464 vs 540 B |
| 0x00473130 | `car_save_game_load` | `SaveGameLoad__5C_carFPPv` | 0x00140d40 | CONFIRMED | The exact inverse; it vacates every seat first, gates on `version == 3`, and re-drives `car_set_enter_protect` and `car_set_model_level` from the restored bytes rather than poking them. 592 vs 760 B |
| 0x00473380 | `car_track_record_key` | - | - | - | `C_actor_track` slot 0 — §4a |
| 0x00473590 | `car_track_spawn_dropout_part` | - | - | - | `C_actor_track` slot 1 — §4a, §4b |
| 0x00473750 | `car_track_apply_pose_at_clock` | - | - | - | `C_actor_track` slot 2 — §4a |
| 0x00473b20 | `car_track_create` | - | - | - | `C_actor_track` slot 9 — §4a |
| 0x00473b40 | `car_track_apply_pose_at_time` | - | - | - | `C_actor_track` slot 3 — §4a |
| 0x00473e40 | `car_track_get_pos_at_time` | - | - | - | `C_actor_track` slot 4 — §4a. True size 336 B (§3c) |
| 0x004f5ad0 | `car_set_condition_channel` | - | - | - | Out-of-line inline member: writes `+0x20b8` and returns the previous value (§1b) |
| 0x0059abc0 | `car_net_snapshot_state` | - | - | - | slot 39; Latches the transform as the dead-reckoning baseline |
| 0x0059ad60 | `car_net_apply_remote_state` | - | - | - | slot 15; `NetDirect(Ui)` — extrapolates a REMOTE car between state messages. 4928 B |
| 0x0059c0a0 | `car_broadcast_state_net_event_msg` | - | - | - | On the host, the full snapshot to every near peer (`g_net_session+0xb4`) and, on one call in three, the reduced one to every far peer (`+0xc4`); a client sends to the host only. `car_update` calls it when `net_flags` bit `0x1` is set (a collision on a car this peer owns; that send carries reason 3, which starts the receiver's `net_collision_hold_ms`) or when `net_broadcast_timer` passes the session's update interval |
| 0x0059c200 | `car_post_reduced_state_net_event_msg` | - | - | - | The cheap payload for far peers, types `0x21` / `0x23` (`structs/net_event_msg.md` §7) |
| 0x0059c3a0 | `car_post_state_net_event_msg` | - | - | - | Builds one type-`0x1e`/`0x1f` snapshot |
| 0x0059c690 | `car_apply_net_event_msg` | - | - | - | slot 38; Receive: applies types `0x29` / `0x2a` / `0x2b` and the glass arms ¹ |
| 0x0059d2f0 | `car_post_break_glass_net_event_msg` | - | - | - | Type `0x27`, from `car_break_glass` |
| 0x0059d3a0 | `car_post_tyre_burst_net_event_msg` | - | - | - | Type `0x2c`, from `car_hit`: the index of a wheel a shot burst |
| 0x0059d410 | `car_post_part_dropout_net_event_msg` | - | - | - | Type `0x2d`, from `car_drop_out_part` |
| 0x0059d4d0 | `car_post_dirty_net_event_msgs` | - | - | - | Flushes `car+0x1e8` dirty bits 2/4/8 |
| 0x0059d640 | `car_post_collision_damage_net_event_msg` | - | - | - | Type `0x2b`, then clears the queue |
| 0x0059d6d0 | `car_clear_collision_damage_records` | - | - | - | frees and empties the collision-damage queue `+0x21d4` (§8) |
| 0x0059d750 | `car_queue_collision_damage_record` | - | - | - | Pushes one record onto the `+0x21d4` queue |
| 0x0059d950 | `car_post_explosion_net_event_msg` | - | - | - | Type `0x24`, from `car_explosion` |
| 0x005c89f0 | `car_set_tank_hit_count` | - | - | - | script `settankhitcount` (case `0xbe`): writes BOTH `+0x2094` and `+0x2098` from its one argument (§1) |
| 0x005c8a60 | `car_adjust_brake_hold_count` | - | - | - | raises or lowers the brake-hold count `+0x211c` |
| 0x00605630 | `car_clear_all_seat_owners` | `ClearCarSeats__FP5C_car` | 0x003865a0 | CONFIRMED | evicts every occupant: the seat count from `+0xd90..+0xd94`, `car_set_owner(NULL)` per seat; a free function on the PS2 |

### 3d. Free helpers

No `C_car` receiver: the container helpers of the class's vectors, whose `this` is the vector, and
the shared container helpers of the unit that the class's methods call (§5b).

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004741d0 | `car_carinit_vector_count` | - | - | - | CARINIT level-vector helper, one owner, class-specific stride |
| 0x00474200 | `vec_count_stride_0x8` | - | - | - | a shared container helper of the unit (§5b) |
| 0x00474220 | `vec_count_stride_0x4` | - | - | - | a shared container helper of the unit (§5b) |
| 0x00474240 | `vec_count_stride_0x18` | - | - | - | a shared container helper of the unit (§5b) |
| 0x00474270 | `vector_erase_range_dwords` | - | - | - | a shared container helper of the unit (§5b) |
| 0x004742a0 | `car_material_vector_insert_n` | - | - | - | `std::vector<S_TSRFC>::insert`, stride `0x30`. Family-specific rather than generic, measured: 3 callsites, 2 in `car_load_materials` plus one `C_Vehicle` function — both owners in this family and sharing the record type. The type name comes from the PS2 signature `ConvertTyres__5C_carFP7S_TSRFCP6S_SRFC` |
| 0x00474590 | `car_seat_vector_insert_n` | - | - | - | `std::vector<seat>::insert`, stride `0xa0`. ONE callsite, `car_init`, receiver `lea ecx,[ebp+0x2108]` — the header of the seat vector whose records live at `car+0x210c` |
| 0x004748a0 | `car_tyre_surface_vector_insert_n` | - | - | - | `std::vector<S_SRFC>::insert`, stride `0x5c`. ONE callsite, `car_load_tyre_surfaces`; the stride is corroborated independently by `car_convert_tyre_surface` writing that record out to `+0x58` |
| 0x00474bd0 | `stl_uninit_copy_stride_0x1c` | - | - | - | a shared container helper of the unit (§5b) |
| 0x00474c10 | `stl_copy_back_stride_0x18` | - | - | - | a shared container helper of the unit (§5b) |
| 0x00474c40 | `car_carinit_vector_copy` | - | - | - | CARINIT level-vector helper, one owner, class-specific stride |
| 0x00474c80 | `car_carinit_vector_copy_back` | - | - | - | CARINIT level-vector helper, one owner, class-specific stride |

## 4. Fields reached through the `C_car` spelling

### 4a. `C_actor` and `C_Vehicle` fields

Code outside the two classes, the chase AI (`C_driver`, through `driver+0x10`) and the HUD producer
among it, reads these as `car + N`. Each is a field of the base or of the embedded `C_Vehicle`
(`car + N` = `C_Vehicle + (N − 0x70)`); the authority for its meaning is the base document.

| `C_car` offset | Base field | Field | Conf | Evidence |
|---|---|---|---|---|
| +0x10 | `C_actor+0x10` | entity kind, 4 | H | `structs/entity_kind.md` §1 |
| +0x30..+0x38 | `C_actor+0x30` | `cached_forward` (`structs/actor.md` §1) | H | `driver_fill_start_values` copies it into `car_forward` (`structs/driver.md` `+0xa0`): the brake-onset pitch, the next-node-behind test and the direction of the mode-2 sweep |
| +0x70 | `C_Vehicle` | the embedded sub-object | H | `lea ecx,[car+0x70]` at the `vehicle_set_engine_on` / `vehicle_get_engine_rpm` callsites |
| +0xa0..+0xa8 | `C_Vehicle+0x30` | the AI look-at target | H | `driver_turn` writes it, and the valid flag `+0xac` = 1 |
| +0x1ac | `C_Vehicle+0x13c` | the under-pursuit flag | H | `pursuit_event_dispatch_pursuit_vehicle` and `pursuit_event_dispatch_escape_vehicle` write 1, `pursuit_event_remove_vehicle` 0, `pursuit_event_save_game_load` restores it |
| +0x20c / +0x3f8 | `C_Vehicle+0x19c` / `+0x388` | `light_state` and the wake-up slot the light setters invoke | H | ¹ |
| +0x294 | `C_Vehicle+0x224` | `master_flags`; bit 25 (`0x2000000`) = the last physics step made a contact whose collision record carries flag `0x10000` | H | ² |
| +0x51c | `C_Vehicle+0x4ac` | `horn_on` | H | `driver_do_sounds` writes it |
| +0x51d | `C_Vehicle+0x4ad` | `siren_on` | H | set by `police_board_create_and_engage` and the `police_action_*` dispatch, cleared on every cancel and teardown |
| +0x594..+0x597 | `C_Vehicle+0x524..+0x527` | the four input bytes | H | written by `player_ai_drive`, `driver_tick`, `traffic_car_activate_elements` and `car_net_snapshot_state`; `car_update` plays force feedback only while `+0x594` is set (`structs/vehicle.md` §2c) |
| +0x5b8 | `C_Vehicle+0x548` | `automatic_gearbox` (1 = automatic) | H | `driver_tick`; the HUD producer passes it to `hud_speedometer_set_gear` (`basic/artifacts/hud_elements.md` §6b) |
| +0x634 | `C_Vehicle+0x5c4` | `engine_revs`, revolutions per second | H | written by the drivetrain and, on a remote car, by `car_recompute_velocity` from the received 16-bit value × `+0x1f88`; `vehicle_get_engine_rpm` returns it × 60 while the engine runs; `car_extended_update` builds its rev fraction from it |
| +0x648 | `C_Vehicle+0x5d8` | `gear`; −1 = reverse | H | `vehicle_set_gear` is called with `+0x648 ± 1` |
| +0x688 | `C_Vehicle+0x618` | `speed` | H | the police speed gate and most driver helpers; getter `vehicle_get_speed` (`0x00425440`), declared on the `C_Vehicle` (`lea ecx,[car+0x70]`) |
| +0x6d8 | `C_Vehicle+0x668` | `idle_revs` | H | `vehicle_apply_carinit_params` sets it from the CARINIT `idling rpm` ÷ 60 (`def-files-docs/vehicles-bin.md`) |
| +0x6e0 | `C_Vehicle+0x670` | a mass term | M | the denominator of `power_mass_ratio`, the impact normalisation of `car_on_collision_apply_damage`, the brake math of `driver_compute_brake_onset` / `driver_compute_brake_pedal` |
| +0x6fc | `C_Vehicle+0x68c` | `steer_lock_now` | H | the `driver_turn` clamp |
| +0xd18 | `C_Vehicle+0xca8` | `engine_on` | H | `vehicle_set_engine_on` writes it; `car_recompute_velocity` sets it on a remote car while the received revs are at least 1 per second. Read by `car_update` (four times, the steady-vibration force-feedback effect among them), `car_extended_update`, `player_ai_drive`, `player_do_action_button_in_car` and `traffic_car_take_over_cruising_car` |
| +0xd90 / +0xd94 | `C_Vehicle+0xd20` / `+0xd24` | the door / window record array, stride `0x24`, index-parallel to the seat vector | H | `vehicle_get_seat_count` (`0x00425460`) returns `(end − begin) / 0x24` (§6d) |
| +0x1f7c..+0x1f84 | `C_Vehicle+0x1f0c` | `velocity` | H | `car_recompute_velocity` writes it, `car_get_actor_speed` copies it out; the wheel-contact proximity helpers use it |
| +0x1f88 | `C_Vehicle+0x1f18` | `max_revs` | H | `vehicle_apply_carinit_params` sets it from the CARINIT `maximum rpm` ÷ 60; the HUD normalises RPM as `vehicle_get_engine_rpm / (car+0x1f88 * 60)` (`basic/artifacts/hud_elements.md` §6b) |
| +0x1f8c | `C_Vehicle+0x1f1c` | `fuel_capacity` | H | the denominator of the HUD fuel-needle ratio |
| +0x1f90 | `C_Vehicle+0x1f20` | the odometer reading | H | `car_game_init` stores `251000 + 150 × crt_rand()` and nothing else writes it; `player_ai` and `player_ai_drive` pass it to `hud_speedometer_set_values`, which shows the five low digits of ⌊value × 0.1⌋ (`basic/artifacts/hud_elements.md` §6b) |

¹ `structs/vehicle.md` §2a is the bit map and §10a the slot. `car_update` and `car_deactivate` reach
them through the `C_car` base while the script setters reach the same memory as `C_Vehicle`
(`add ecx, 0x70`). Bit `0x10000` is the **main headlights**: `driver_do_lights` sets
`auto_headlights_suppressed` and then **flashes** `0x10000` on a timed pattern from the duration
table at `driver+0x130`, the chase AI flashing its headlights. Script command `carlight_main`
(dispatcher case `0xc0`) drives the same bit, verified in game. Two gates are confirmed with it:
`0x800` gates `0x100` (brake) and `0x8` gates bits `0x1` / `0x2` (indicators).
² `vehicle_update_visuals` sets `0x2000001` when `C_Vehicle+0x28` is set and clears bit 25
otherwise; `+0x28` is raised by `vehicle_wheel_contact_solve` and `vehicle_body_coll_resolve` on such
a contact. Readers: `human_find_blocking_drawbridge` (`car+0x294 >> 0x19 & 1`, the drawbridge
exit-block) and two arms of `vehicle_wheel_contact_solve`. `0x10000` is bit 16 of a static collision
primitive's flag dword, loaded unchanged from `missions\<mission>\tree.klz` by `collision_load_level`.
A parse of all 81 shipped `tree.klz` files finds it only on box primitives (kind `0x83`): five per
leaf on the drawbridge leaves `LLsklap01` / `LLsklap02` and `sklapx01` / `sklapx02` in every city
scene (the east leaves are named `sklap01` / `sklap02` in three scenes), plus four single boxes in
three other scenes. So bit 25 marks a contact with a drawbridge leaf.

**The fuel tank.** `car_get_fuel_tank_world_pos` (`0x004254a0`) transforms `tank_dummy_pos` by the
body matrix at **`car+0xdd4`**; Ghidra renders the call as `S_vector::operator*((S_matrix *)(this +
0x2084))`, which mis-assigns the operands: the disassembly is `f(point = car+0x2084, out, matrix =
car+0xdd4)`. The receiver is the **C_car itself** (`mov ecx,eax` straight off `actor_get_vehicle`,
with the kind checked one instruction earlier). Consumers use the result as the AI's aim point when
`entity+0xf08 & 0x200000` and the target's kind is 4: **shooters aim at the fuel tank**.
`tank_hit_count` is decremented when something passes within `tank_hit_radius` of the tank point
(`if (0 < +0x2094 && 0.0 < +0x2090)`); below 1 it sets `explosion_pending` with a 0..99 ms
`explosion_delay_ms`. While `tank_hit_count` is under `tank_hit_max` and the fuel `car+0xd1c` is above
2.0, `car_update` drains the fuel by `(max − count) × step × 0.0005`: each lost hit leaks fuel.
`car_set_tank_hit_count` writes **both** `tank_hit_count` and `tank_hit_max` from its one argument,
so script command `settankhitcount` (dispatcher case `0xbe`) resets the running count as well as the
threshold: re-issuing it heals a tank that has been shot. `fuel_dummy_pos` and
`tank_dummy_pos` are filled at car init from the `"FUEL"` / `"TANK"` frames' `frame+0x80..+0x88`
through the model's find-child-by-name virtual (`+0x38`) and zeroed when the model has no such dummy;
`tank_hit_radius` is `ABS(bbox extent) * 0.5` of the `"TANK"` frame (`frame+0x140`, six dwords).

**The script-side level trio** is `car_getactlevel` / `car_getmaxlevels` / `car_setactlevel`
(dispatcher cases `0x164` / `0x163` / `0x165`). **Both GETTERS gate on `GetKind == 6` (`C_door`)
instead of 4 and are therefore dead**, with 0 shipped call sites each. `car_setactlevel` gates
correctly and is the script entry to the boosting swap. The Mafia Script Data Reference reads all
three as a FUEL gauge; there is no fuel model here.

### 4b. The condition channel and the four damage pools

`car_get_condition_fraction` returns a 0..1 fraction, and `condition_channel` (`+0x20b8`) selects
WHICH meter it reports. Ghidra renders its FPU returns as integer returns, so the arms are read from
the disassembly: a four-entry jump table at `0x00470868`. Arms 0..2 are **current ÷ maximum** of a
separate damage pool; arm 3 is one deform zone's integrity.

| Channel | Formula (PC offsets) | Notes |
|---|---|---|
| 0 | `+0x2dc / +0x2e0` (`C_Vehicle+0x26c / +0x270`) | **the gearbox pool.** `car_hit` subtracts from it and clamps at 0; `car_is_usable` requires it above 0. With damage enabled (`C_Vehicle+0xce4` bit `0x4000`) an empty pool makes `vehicle_set_gear` refuse every gear change and stops the automatic shift in `vehicle_tick_drivetrain`; `vehicle_update_visuals` tests the same gate |
| 1 | `+0x2a4 / +0x2a8` (`C_Vehicle+0x234 / +0x238`) | **the engine pool.** `car_hit` subtracts from it and then derives `+0x2a0 = +0x29c + (1 − +0x29c)·(pool/max)` (`C_Vehicle+0x230` from `+0x22c`), the factor `vehicle_compute_motor_force` multiplies the motor force by; `vehicle_apply_damage` drains it the same way and latches `C_Vehicle+0x66c` once it is empty (`structs/vehicle.md` §2); `car_is_usable` requires the pool above 0. **This is the channel the chase AI uses** |
| 2 | `damage2 / damage2_max` | `car_hit` and `car_on_collision_apply_damage` subtract from it (the collision path touches this pool only); the one pool `car_set_model_level` preserves across a boost swap. **No consumer:** a census of every instruction in `.text` with displacement `0x2004` finds only its updaters and this arm, and no code selects channel 2: the channel is 3 after `car_game_init`, and every override sets 1 |
| 3 | `zone+0x20 / zone+0x18` of the **last** `DEFORM BODY` zone (type 1) in the deform-zone table `+0x308` (§6c) | `1.0f` when the car has no body zone |
| default | `1.0f` | - |

Before any of that, the getter returns **0.0f** when `+0x5e` is set, the `C_actor` death-processed
latch (`actor_is_death_processed`, `structs/actor.md` §1).

**Reading and restoring the channel** is `car_set_condition_channel` (`0x004f5ad0`), an out-of-line
`C_car` inline member that writes `+0x20b8` and returns the previous value. All nine of its
callsites are a scoped override (`old = set(1); read(); set(old)`), and the pushed immediate is
literally `1` at every one, disassembled rather than inferred. The chase AI reads `== 0.0` as
*wrecked* and `< 0.15` as *give up*.

## 5. The translation unit

**`[0x00462a90, 0x00474cc0)`**, guard to guard: the CRT static-init / atexit pairs at
`0x00462a90` / `0x00462ab0` and `0x00474cc0` / `0x00474ce0`. **0 unnamed starts.**

### 5a. The accounting closes with no remainder

```
108 real starts (119 Ghidra starts - 11 decompiler artifact starts)
  =  77 twin-mapped bodies
  +   2 CRT static-init / atexit stubs
  +  12 PC-only members
  +   3 S_TSRFC / seat / S_SRFC vector helpers   (family-specific, census-graded)
  +   8 foreign or genuinely-generic helpers
  +   4 thunks
  +   2 named C_car members outside the twin set
```

### 5b. Not `C_car`: foreign or genuinely generic

`string_ctor_from_cstr` (`0x00464bd0`) and `string_wildcard_match` (`0x004656b0`) are a refcounted
string helper class linked into this cluster (`structs/string.md`); they shift the ranks an
order-alignment comparison computes. Also `collision_earliest_contact_between_spheres`
(`0x00470880`), `i3d_frame_match_deform_roof` (`0x00473f90`), and the shared STL helpers
`stl_uninit_copy_stride_0x1c` (27 callsites across ≥10 owners), `stl_copy_back_stride_0x18` (10
across 7), `vec_count_stride_0x4/0x8/0x18` and `vector_erase_range_dwords`, which §3d lists because
the class's methods call them.

The same five-function container-helper family **splits both ways** under a per-member receiver
census: the two string helpers stay mechanical, while `car_carinit_vector_count` / `_copy` /
`_copy_back` and the three `*_vector_insert_n` functions each have one owner and a class-specific
record stride. Census each member separately rather than grading the family as a whole.

### 5c. Eleven decompiler artifact starts sat inside this unit

They are Aggressive-Instruction-Finder splits, not functions: none is reached by any `E8 rel32`, none
appears as a dword in any data section, and each is the target of an earlier `jcc` inside its host.
Such a split truncates the real function's apparent size, and that size feeds every order-alignment
smell test:

| Real function | Apparent → true | Ratio vs PS2 |
|---|---|---|
| `car_get_surface_material_id` | 151 → **192 B** | 0.69 → **0.87** |
| `car_hit` | 488 → **4656 B** | 0.09 → **0.86** |
| `car_on_collision_with_actor` | 81 → **432 B** | 0.16 → **0.88** |
| `car_collision_filter_actor` | 66 → **112 B** | 0.30 → **0.51** |
| `car_on_body_surface_contact` | 2103 → **2640 B** | 0.71 → **0.89** |
| `car_collision_filter_wheel` | 38 → **64 B** | 0.53 → **0.89** |
| `car_track_get_pos_at_time` | 39 → **336 B** | - |

Three of the eleven are unanswerable by shape alone: their raw bytes decode as `pop edi ; mov eax,
0x18 ; pop esi ; ret 4` and the same with `0x12` and `0x13`, **three of the four return constants
of `car_get_surface_material_id` itself**. `car_hit` is why this is not cosmetic: at its apparent
488 B it looked unmappable against a 5424-byte PS2 method.

### 5d. The unit's globals

Every global the unit's functions reference is named: `0x0064b73c..0x0064b74c` are the five
force-feedback effect handles (`g_ff_car_effect_wheel_friction` / `_lateral_spring` / `_impact` /
`_speed_vibration` / `_steady_vibration`), `0x0064b750..0x0064b768` their gain weights and
change-detect caches (`g_ff_car_gain_*`, `g_ff_car_last_gain_*`), `0x00655ab0` is
`g_car_ff_prev_velocity` (zeroed by `car_static_state_init`: `car_update`'s force-feedback impact
block differences the previous car-space velocity, so it must start at zero), and the frame-name
suffix strings `".DWHL"` / `".DWHR"` are `s_dwhl_0064b85c` / `s_dwhr_0064b864`. The remaining data
references are string literals Ghidra labels itself.

## 6. The recorder track, the detachable parts and the seats

### 6a. The `car_track_*` family — `C_car`'s override of the `C_actor_track` interface

`C_car` overrides **six** slots of the 13-virtual `C_actor_track` base (`structs/actor_track.md`),
the PC-only `C_recorder` keyframe system.

| Slot | PC function | VA | What it does |
|---|---|---|---|
| 0 | `car_track_record_key` | 0x00473380 | Appends a pos / quat / wheel-contact keyframe, 50 ms-gated. Returns **-1** when nothing was recorded, the contract of the base slot-0 stub `stub_retminus1_args4_0044a780` |
| 1 | `car_track_spawn_dropout_part` | 0x00473590 | Replays a "part came off" key (§6b) |
| 2 | `car_track_apply_pose_at_clock` | 0x00473750 | Playback at the **global** `g_recorder_playback_clock_ms`; the only one that derives velocity, into `car+0x1f7c`, from the difference against `playback_prev_pos`; it also sets `playback_controls` / `playback_wheel_contacts` from key dwords 9 / 10 and the cached position `+0x24` / `+0x40c` |
| 3 | `car_track_apply_pose_at_time` | 0x00473b40 | Playback at an **explicit time**; walks the variable-stride key list (stride from `track+0x44 + tag*4`) |
| 4 | `car_track_get_pos_at_time` | 0x00473e40 | Read-only **position** query at an arbitrary time |
| 9 | `car_track_create` | 0x00473b20 | Declares this class's three key sizes: `0x2c`, `0x38`, `0x38` |

**The receiver is byte-proven `C_car`, not `C_Vehicle`:** every one of them reads the track from
`this+0x08` and the I3D frame from `this+0x68`, both `C_actor` **base** fields that `C_Vehicle` does
not have. Their vtable `0x0063bc08` also carries `AI__5C_carFUi`, `Update__5C_carFUi`,
`GameInit__5C_carFv` and `GetActorSpeed__5C_carFR8S_vector`.

### 6b. `C_DropOut` — the parts a car sheds

A **`C_DropOut`** is an actor of `0x3b8` bytes, kind `0x11`, vtable `0x0063c320`, constructor
`dropout_ctor` (`0x0048a530`). Two paths create one:

* the **live** damage path: `car_prepare_wheel_dropout` / `car_prepare_body_part_dropout` call
  `car_drop_out_part`, which allocates the actor, branches `if (selector == 1) InitWheel else
  InitBox` and registers it as a temporary actor;
* the **playback** path: `car_track_spawn_dropout_part` (slot 1), on a key of type 2, does the same
  for a recorded track: it binds the actor to the car part the key names, hides the part on the car
  body, registers it with `game_add_temporary_actor`, gives it its model from the key's name string
  and starts it.

The key's selector word `key+0x2c` splits into a low 3-bit part CLASS and a high index:

| Class | Resolved through | Which parts |
|---|---|---|
| 1 | the wheel list at `car+0xd24` | wheels |
| 2..7 | remapped to deform-zone types 2/5/7/4/6/8, then the `0x34`-stride deform-zone table at `car+0x308` (§6c) | bumper / light / licence plate / mirror / wing / door |

So a DropOut is either a **wheel** or a **body panel**, two initialisation paths and nothing else.
Which panels can come off depends on the model's zones: the shipped models tag bumpers, lights and
wings, and none tags a mirror, a licence plate or a door (§6c). The kind byte is labelled by the
game itself: `recorder_new_track` formats these tracks `drp_%d` for kind `0x11`.

**In game (user-observed, PC build):** a car can lose its **wheels, bumpers and lights**, the PC's
DropOut vocabulary.

**`C_DropOut`'s base is `C_PhThing`, not `C_actor` directly:** `dropout_ctor` chains to
`physical_object_ctor` (`0x004be910`), the engine's physics-body base. Both builds have exactly
**five** `C_PhThing` subclasses.

### 6c. The deform-zone table `car+0x308`

`car+0x308` / `car+0x30c` is `C_Vehicle+0x298` / `+0x29c`: a vector of **`0x34`-byte deform zones**,
one per breakable or deformable body part. It is built from the car's MODEL, not from `tables/`:
`vehicle_init` enumerates every mesh frame of the model (type mask `0x10001`, name `*`) through
`vehicle_cb_enum_deform_zones`, which reads the text between `[` and `]` in the frame's
user-parameter string and appends one zone per recognised tag:

| Tag | Zone type | Shipped models carrying it |
|---|---|---|
| `DEFORM BODY` | 1 | 437 |
| `DEFORM BUMPER` | 2 | 178 |
| `DEFORM GLASS` | 3 | 381 |
| `DEFORM MIRROR` | 4 | 0 |
| `DEFORM LIGHT` | 5 | 193 |
| `DEFORM WING` | 6 | 2 |
| `DEFORM SPZ` (licence plate) | 7 | 0 |
| `DEFORM DOOR` | 8 | 0 |
| `DEFORM ROOF` | 9 | 48 |

The counts come from the 446 shipped `.4ds` models that carry any `[DEFORM ...]` tag. Two frames
spell the tag `DEFORMGLASS` without the space and are not recognised, so they get no zone.

| Zone offset | Type | Meaning |
|---|---|---|
| `+0x00` | `I3D_frame *` | **the model frame this zone is**; Ghidra types the cast directly (`this = (I3D_frame *)*puVar11`) |
| `+0x0c` | `short` | zone type, 1..9 (table above); type 3 (glass) takes its own handling path |
| `+0x0e` | flag byte | bit `1` = detached (the gate `car_track_spawn_dropout_part` sets), bit `2` = damaged |
| `+0x18` | `float` | maximum integrity, the denominator of condition channel 3 (§4b) |
| `+0x20` | `float` | integrity / remaining health, clamped at 0 |

**Consumers**, receiver confirmed by vtable membership rather than proximity: `car_hit` and
`car_get_surface_material_id` are `C_car` vtable slots 31 and 32; `car_set_transparency` also walks
it; `car_prepare_body_part_dropout` switches on the zone type; `car_get_condition_fraction` reads the
last body zone; and `car_track_spawn_dropout_part` indexes it through
`vehicle_find_part_at_encoded_position`.

### 6d. The seats

The seat vector `+0x210c` holds one `0xa0`-byte record per door, index-parallel to the
`C_Vehicle` door / window array `car+0xd90` (`C_Vehicle+0xd20`, stride `0x24`, one record per door
with the door / window object at record `+0x20`; `structs/vehicle.md` §9). Record fields measured
from the seat functions: occupancy byte `+0x0c`, occupant `+0x10`, door action code `+0x40`
(`0x7d3` when `car_set_owner` vacates the seat), and two placement vec3s `+0x84` / `+0x90`
(`car_seat_get_attach_positions`). `car_init` builds one record per element and passes whether
element 0 has a door object to `human_get_anim_id_for_use_car`, which picks the enter-car animation
variant. Three consumers feed a seat index straight into `car_get_owner`:
`ai_fight_followship_count_free_seats`, `ai_follow_manager_get_car_seat_for_follower` and
`ai_follow_manager_group_fits_leader_vehicle`. `claim_mask` / `enable_mask` hold one bit per seat:
`car_set_enter_protect` and `car_set_enter_protect1` toggle the per-seat using-object nodes
(`using_object_add_object` / `_del_object`) and the masks, with the door action codes `0x7d3` /
`0x7d4` / `0x7d9` and an `index == 1` special case.

## 7. `C_car_ex` — the extended car, this class's one subclass

`C_car_ex` (PC prefix `car_extended_`, doc `structs/car_extended.md`) extends `C_car`. It is a plain
single-inheritance subclass, and the vtables say so precisely:

* its table `0x0063bd68` is **41 slots, the same width as `C_car`'s `0x0063bc08`**, and the slots
  are **aligned one-for-one with no shift**;
* **exactly 8 slots differ** (14, 16, 18, 19, 20, 21, 25, 26), which are exactly the 8 methods
  `C_car_ex` overrides on the PS2 side: `Update`, the scalar deleting destructor, `Init`, `SaveData`,
  `LoadData`, `DuplicateFrom`, `GameInit`, `GameDone`;
* it keeps `kind +0x10 = 4`, so every "is this a car" test in the engine still passes. The
  discriminator is the `C_actor`-level flavour field **`+0x14`**: `C_actor` = 1, `C_human` = 2,
  **`C_car_ex` = 3** (`structs/entity_kind.md` §3a). `human_game_done` gates on `kind == 4 && +0x14
  == 3`.

`sizeof(C_car_ex)` is `0x2254` against `C_car`'s `0x21f4`; the extra `0x60` is four vector triples
plus the `~DCS~` spot (`dcs_present` `+0x2214` and `taken_out_list` `+0x2230`, which
`human_intern_use_car_truck` tests). The entity factory builds it at `case 0x24`.

## 8. The replication band

`C_car`'s multiplayer code does **not** live in the `C_car` cluster. It is split across the
event-message translation unit `[0x005929c0, 0x00594740)` and the sender cluster at `[0x0059ab90,
0x0059d9d0)`; `car_apply_net_event_msg` sits 1.1 MB away from the rest of the class. The functions are
the `car_net_*`, `car_*_net_event_msg` and `*_collision_damage_records` rows of §3c and §3b. Ghidra
renders `car_apply_net_event_msg`'s switch selector as a float, so its case labels print as
`5.74532e-44` (= `0x29`) and friends; read the raw integers.

**The dead-reckoning band** is the `net_*` fields of §1, latched by slot 39 (`car_net_snapshot_state`)
and refreshed by slot 38 (`car_apply_net_event_msg`); `net_broadcast_counter` raises bit `0x100` of
`net_flags` on one call in three, and `car_update` broadcasts when `net_broadcast_timer` passes the
session's update interval.

**`net_flags` bits**, from every access in `.text`:

| Bit | Meaning | Set / read by |
|---|---|---|
| `0x1` | a collision on a car this peer owns | set by `car_on_collision_with_actor`; `car_update` then sends at once with reason 3 and clears it |
| `0x8` | a state message arrived | set by `car_apply_net_event_msg`; `car_net_apply_remote_state` then skips its correction-quaternion block and snaps the body back to `net_pos` beyond 5.0 (12.5 with `0x200`) |
| `0x40` | this remote car's collision is posted | set when `car_on_collision_with_actor` posts it; while `0xc0` is set no further collision event is posted |
| `0x80` | the collision hold | held by `net_collision_hold_ms` |
| `0x100` | feed the far peers | raised by `net_broadcast_counter` on one call in three |
| `0x200` | the last message was a reduced one | set by `car_apply_net_event_msg` |

**`net_collision_hold_ms` (`+0x21a1`)** is a byte countdown in ms. `car_apply_net_event_msg` starts it
at half the session's update interval (`+0x14 >> 1`) on a type-`0x1e` / `0x1f` message whose reason
byte is 3; `car_net_apply_remote_state` counts it down by the step, keeps `net_flags` `0x80` set while
it runs and clears `0x80` at 0. While `0xc0` is set, `car_on_collision_with_actor` posts no collision
event (`actor_post_collision_event`, type `0x1d`) for a car this peer does not own: a peer's collision
snapshot holds the local collision report for half an update interval.

**Slot 15 is `NetDirect(Ui)` family-wide:** `structs/bottle.md` §9a byte-proves the role on
`C_airplane`, whose slot 15 is a bare `ret 4`. It extrapolates a remotely-owned car; the velocity it
integrates arrives over the wire.

**Slot 39 exists only in the car family.** `C_PhThing`'s and `C_bottle`'s vtables end at slot 39 (the
next dword is float data), while `C_car`'s `0x0063bc08` and `C_car_ex`'s `0x0063bd64` carry one more.
`car_net_snapshot_state` has **0 textual callsites and 2 datarefs** (pure virtual dispatch), and
`car_game_init` calls slot `0x9c`.

**The collision-damage queue `+0x21d4`** holds heap **`{S_vector impulse, S_vector position,
u32}` records**: the impulse's direction and length give the deformation's direction and strength, the
position is the point the receiver deforms (`structs/net_event_msg.md` §1l). They are pushed by
`car_queue_collision_damage_record`, whose **sole caller is
`car_on_collision_apply_damage`**, drained once per frame into the type-`0x2b` message and then freed
by `car_clear_collision_damage_records`. That producer is invisible to every decomp text search; it
was found by a disassembly scan of the whole `.text` for the displacement `0x21d4` across 719 589
instructions.

**What a car snapshot carries.** `car_post_state_net_event_msg`'s payload maps one-for-one onto
`event_msg_car_state_serialize`: the body quaternion from `frame+0x9c`, a door / window bitmask built
from the part flags `0x10` / `0x20` over `car+0xd24`, the gear index `car+0x648`,
`car_pack_light_state`, the GUID `car+0x18`, and two velocity triples scaled by `net_velocity_scale`:
the velocity `car+0x1f7c` in both messages and, in the full snapshot, `angular_velocity` `car+0x50c`.
`net_velocity_scale` is 1.0, or 83 / step when `car_ai_tick` clamps the step to 83 ms, so the peers
receive real-time velocities.
The type is chosen bit-wise from `net_flags`: `0x1f` on the one call in three that also feeds the far
peers, `0x1e` otherwise. Full write-up of the message layer: `structs/net_event_msg.md`.

## 9. PC vs PS2

This is the one section carrying PS2 offsets and cross-build reasoning.

* **Embedded `C_Vehicle` base:** PC `car+0x70`, PS2 `car+0x80`. Field offsets differ throughout (the
  PS2 `C_car` is a larger struct), so the method table maps by ROLE, not raw offset.
* **Detachable parts: the PS2 sheds DOORS, the PC does not.** *(User in-game observation.)* On the
  **PC** a car can lose its wheels, bumpers and lights (§6b); on the **PS2** it can lose doors as
  well. The difference is MODEL DATA, not code: the PC handles a door zone (deform type 8, DropOut
  class 7) exactly like the other panels, both builds run the identical two-branch `if (selector ==
  1) InitWheel else InitBox` logic, and no shipped PC model carries a `[DEFORM DOOR]` tag (§6c). The
  PS2 side of that reading is Open item 1.
* **Condition channels (`ShowCarEnergy`).** The PS2 arms are: 0 `+0x684 / +0x688` (resistance
  `+0x68c`); 1 `+0x658 / +0x65c` (resistance `+0x660`; exhaustion latches `+0x22a8` and de-rates
  `+0x654 = +0x650 + (1 − +0x650)·(pool/max)`); 2 `(+0x238c <= 0) ? 0 : +0x239c / +0x2394`
  (resistance `+0x2398`, `+0x238c` an enabled gate); 3 the mean of `part+0x24 / part+0x1c` over the
  body-part array (`+0x6a0` count, `+0x6a4` base, stride `0x3c`) counting parts with `part+0x3a ==
  1`. Channel 1 carries the same de-rate on both builds (§4b). The PC arm 2 has no enabled gate, and
  the PC arm 3 reads one body zone where the PS2 averages every flagged part.
* **`C_DropOut` cross-build material:** `C_DropOut` exposes exactly two initialisers,
  `InitWheel__9C_DropOutFP15S_DROPOUT_WHEEL` and `InitBox__9C_DropOutFP13S_DROPOUT_BOX`, matching the
  PC's `selector == 1` wheel branch versus the body-panel branch; those two parameter types name PC
  structures the PC decomp cannot supply. `C_PhThing`'s method set (the shared base) is `MoveWheel`
  (6276 B), `FindActors`, `Init`, `GameInit`, `GameDone`, `AI`, `Update`, `ChangeState`;
  `__ct__9C_PhThingFv` is what identifies PC `physical_object_ctor`, and `__ct__9C_DropOutFv` is the
  CONFIRMED twin of `dropout_ctor`. PS2 `C_DropOut` is **0x650 bytes** against the PC's **0x3b8**.
* **The `C_recorder` track interface is PC-ONLY.** All six `car_track_*` overrides (§6a) have no PS2
  counterpart: no class symbol, and none of the subsystem's debug strings exist in SLUS_206.71. A
  measurement, not a failed search.
* **Model-level fields:** current level PC `car+0x20ec` = PS2 `car+0x242c`; the level table is PC
  `car+0x20f4` / `+0x20f8` (begin / end, stride `0x113c`) against a PS2 **cached count** at
  `car+0x2434` and a table pointer at `car+0x2438` (stride `0x1258`); the ratio inputs PC `+0x660` /
  `+0x564` / `+0x6e0` / `+0x69c` = PS2 `+0x8b4` / `+0x794` / `+0x940` / `+0x904`.
* **Seat arrays.** The record array is PC `car+0x210c` stride `0xa0`, PS2 `car+0x2444` stride `0xd0`.
  Seat guard fields: PC seat `+0xc` byte / `+0x10` int → PS2 seat `+0xc4` byte / `+0x30` int;
  placement sources PC seat `+0x84` / `+0x90` → PS2 seat `+0x10` / `+0x20`. The parallel door array
  is PC `car+0xd90` / `+0xd94` stride `0x24` → PS2 `car+0xfd0` / `+0xfd4` stride `0x30`;
  `ai_fight_followship_count_free_seats` body-diffs against that pair.
* **Cheap accessor offsets:** `SetDriver` / `GetDriver` byte PC `car+0x3b0` = PS2 `car+0x2299`;
  `SetTrafficCar` byte PC `car+0x20bd` = PS2 `car+0x2460`; `GetActorSpeed` vec3 PC `car+0x1f7c` =
  PS2 `car+0x460`; `SetTimeParking` writes PC `car+0x2010` = PS2 `car+0x23a8` (the PS2 from
  `*(act_mission+0x20)+0xe54`), and `GetTimeParking` returns the delta. Other own-band pairs from the
  twin bodies: `+0x1ff1` = PS2 `+0x2468`, `+0x2070` = PS2 `+0x2465`, `+0x206c` = PS2 `+0x23e4`,
  `+0x20b8` = PS2 `+0x2404`, `+0x209c` = PS2 `+0x2462`, `+0x2100` / `+0x2104` = PS2 `+0x243c` /
  `+0x2440`, the AI probe points `+0x1fc0` / `+0x1fcc` / `+0x1fd8` / `+0x1fe4` = PS2 `+0x22e0` /
  `+0x22f0` / `+0x2300` / `+0x2310`.
* **Drawbridge flag `+0x294`:** what CONFIRMED `human_find_blocking_drawbridge`. The PC renders the
  test as `(*(uint *)(car + 0x294) >> 0x19 & 1) != 0`, the PS2 as `(*(uint *)(car + 0x648) &
  0x2000000) != 0`: the same bit 25, differently decompiled. So PC `car+0x294` = PS2 `car+0x648`.
* **`C_car_ex` fields:** PC `car+0x2214` = PS2 `car+0x24b0`; PC `car+0x2230` = PS2 `car+0x24b4`
  (the `human_intern_use_car_truck` guard).
* **Readout equivalents (§4a):** light / control flags PC `car+0x20c` = PS2 `car+0x5f8`; horn PC
  `car+0x51c` = PS2 `car+0x229c`; speed PC `car+0x688` = PS2 `car+0x8f0`; mass term PC `car+0x6e0` =
  PS2 `car+0x940`; steering lock PC `car+0x6fc` = PS2 `car+0x95c`.
* **The method count is an UPPER BOUND.** There are 89 PS2 `C_car` methods; some are inlined on the
  PC, live in a second PC cluster, are virtual-only, or are PS2-port-only.

### 9a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0012abc0 | `CloseCar__5C_carFv` | No PC start between `car_init` and `car_seat_get_anim_placement` |
| 0x0012bde0 | `Change_Particle__5C_carFii` | No PC rank at all, and `car_init` never registers `SetCallbackCP` — the two agree |
| 0x0012d2c0 | `GetDriver__5C_carFv` | an 8-byte accessor of the byte `car_set_has_driver` writes; the PC has no function for it: the byte is `C_Vehicle+0x340`, read inline by `vehicle_tick_drivetrain` and `vehicle_update_visuals` |
| 0x001363a0 | `CalmVehicle__5C_carFUifi` | A three-line forwarder to `CalmVehicle__9C_Vehicle` |
| 0x00140ab0 | `DuplicateDamageAndDeform__5C_carFP5C_car` | A three-line forwarder to the `C_Vehicle` method |
| 0x00141040 | `DamageDeformGetSize__5C_carFv` | No PC rank in the tail; the three tail functions there are container helpers |
| 0x00141060 | `DamageDeformSave__5C_carFPPv` | no PC rank in the tail, as `DamageDeformGetSize` |
| 0x00141150 | `DamageDeformLoad__5C_carFPPv` | no PC rank in the tail, as `DamageDeformGetSize` |
| 0x001417e0 | `ChangeSceneInit__5C_carFv` | 12 B; with `ChangeSceneDone` the city-streaming pair the PC build does not have |
| 0x001417f0 | `ChangeSceneDone__5C_carFv` | 64 B; see `ChangeSceneInit` |

## 10. Related

- `structs/vehicle.md` — the `C_Vehicle` physics / drivetrain sub-object at `car+0x70`.
- `police-system/artifacts/boosting.md` — the CARINIT model-level swap (`car_get_model_level` /
  `car_set_model_level`) IS the boost mechanism; `def-files-docs/vehicles-bin.md` — the level records.
- `structs/driver.md` — the chase `C_driver` AI that reads the §4a readouts.
- `structs/actor.md`, `structs/entity_kind.md` — the base and the kind field.
- `structs/car_extended.md` — the one subclass (§7).
- `structs/actor_track.md` — the recorder interface (§6a); `structs/dropout.md` — `C_DropOut`.
- `structs/glass_shatter.md` — `car_break_glass` and the window list.
- `structs/net_event_msg.md` — the replication layer (§8).
- `structs/force_feedback.md` — the five car effects.
- `def-files-docs/materialy.md` — the surface material ids `car_get_surface_material_id` returns.

## 11. Open items

1. **PS2 door drop-outs (§9):** attributed to model data because no shipped PC model carries a
   `[DEFORM DOOR]` tag; the PS2 model archives were not examined.
