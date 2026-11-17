# `C_Vehicle` — the vehicle physics / drivetrain object

**Created** 2026-07-11 13:22 +0200 · **Modified** 2026-11-16 21:19 +0100

The physics and drivetrain half of a car: gears and the automatic gearbox, the pedal ramps, steering,
handbrake, ignition, fuel, size and pivot geometry, wheels, the body-collision tree, deformation and
damage, doors, windows, lamps and the whole vehicle sound bank. A `C_car` holds it at `C_car + 0x70`;
the chase AI (`structs/driver.md`) and the player drive a car entirely through its setters.

| | |
|---|---|
| PC prefix | `vehicle_` |
| PC kind (`+0x10`) | `-`: not an entity; its owner `C_car` is kind 4 (`structs/car.md`) |
| PC vtable | `g_vehicle_vt` @ `0x0063cf7c`, 1 slot (`vehicle_open_sound`) |
| PC base class | `-` |
| PC instance size | `0x1f48`: the last member is the vector object at `+0x1f38`, and `car_update` uses `car + 0x1fb8` (= `+0x1f48`) as a `C_car` field |
| PC constructor | `vehicle_ctor` @ `0x005096d0` |
| PC translation unit | `[0x00509400, 0x0053c280)`: four source files guard to guard (§8), 206 starts, all named. The out-of-line inline members at `0x00425440`..`0x00425490`, `0x004f5a30`..`0x004f5ac0` and `0x005c8780`..`0x005c89e0` are emitted outside it |
| PS2 class | `C_Vehicle` (tag `9C_Vehicle`), 153 methods: 134 with a PC function, 9 inlined into one, 10 with none (§12) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Offsets are relative to the `C_Vehicle`, which is `C_car + 0x70`. `vehicle_set_engine_on` is reached
as `lea ecx,[reg + 0x70]` at 11 of its 28 callsites, and the receiver of `vehicle_set_seat_mass` is
`<C_car> + 0x70` at all 10. Code outside the class reaches the same fields with the `C_car` spelling,
`N + 0x70`: `car + 0x20c` is `+0x19c`, `car + 0x688` is `+0x618`, `car + 0x5b8` is `+0x548`. The
`C_car`-relative readouts are in `structs/car.md` §4a.

The typedef holds every offset the class's own functions access, except the two bytes
`+0x50e`..`+0x50f`, which nothing in the image accesses, and the padding after byte fields. A
field with no reader carries a mechanical name (`unread_`, `unused_`) or an [M] tag, with the
scope of the search in its comment.

```c
typedef struct C_Vehicle {                         // 0x1f48 bytes; PC offsets
    /*  +0x000 */ void       *vtable;                     // [H] g_vehicle_vt: one slot, vehicle_open_sound
    /*  +0x004 */ uint32_t    rest_ms;                    // [H] ms the root has stayed within 0.15 m of rest_anchor
    /*  +0x008 */ float       rest_anchor[3];             // [H] the root position the rest timer measures from
    /*  +0x014 */ uint16_t    burst_tyre_quiet_ms;        // [H] ms since a burst tyre last rolled on the ground
    /*  +0x018 */ float       strip_scroll_rate;          // [H] the strip UV scroll rate (CARINIT)
    /*  +0x01c */ float       strip_phase_left;           // [H] the left strips' scroll phase, -1..1
    /*  +0x020 */ float       strip_phase_right;          // [H] the right strips' scroll phase, -1..1
    /*  +0x024 */ float       idle_throttle_floor;        // [H] the lowest throttle the motor force uses
    /*  +0x028 */ uint8_t     contact_latch;              // [H] a contact with collision flag 0x10000 this step
    /*  +0x02c */ float       bumps_enabled;              // [H] non-zero runs the per-wheel bump generator
    /*  +0x030 */ float       steer_to_point[3];          // [H] the point the chase and traffic AI steer toward
    /*  +0x03c */ uint8_t     steer_to_point_on;          // [H] 1 = vehicle_move steers toward steer_to_point
    /*  +0x040 */ float       grip_bonus;                 // [H] added to both grips of every wheel on the ground
    /*  +0x044 */ float       drive_force_scale;          // [H] scales the clutch capacity and both motor forces
    /*  +0x048 */ uint8_t     body_hit_last_step;         // [H] the previous step's body-collision result
    /*  +0x04c */ float       front_axle_mid[3];          // [H] midpoint of the first two wheels' rest positions
    /*  +0x058 */ float       rear_axle_mid[3];           // [H] midpoint of the last two wheels' rest positions
    /*  +0x064 */ uint16_t    substep_count;              // [H] the frame's sub-step count
    /*  +0x066 */ uint16_t    substep_index;              // [H] the current sub-step
    /*  +0x068 */ S_matrix    step_start_matrix;          // [H] world_matrix at the start of the frame or sub-step
    /*  +0x0a8 */ S_matrix    step_start_matrix_inv;      // [H] world_matrix_inv at the start of the step
    /*  +0x0e8 */ float       step_start_pos[3];          // [H] the root position at the start of the frame
    /*  +0x0f4 */ float       step_start_forward[3];      // [H] the forward axis at the start of the frame
    /*  +0x100 */ float       step_start_up[3];           // [H] the up axis at the start of the frame
    /*  +0x10c */ float       step_start_right[3];        // [H] the right axis at the start of the frame
    /*  +0x118 */ uint8_t     dyn_colls_off;              // [H] 1 = the dynamic-collision records are erased
    /*  +0x119 */ uint8_t     has_roof;                   // [H] the CARINIT roof flag; ctor 1
    /*  +0x11a */ uint8_t     has_strips;                 // [H] 1 when either strip vector is non-empty
    /*  +0x11c */ uint32_t    wheel_coll_masks[2];        // [H] the mask arguments of the wheel line test; ctor 0xf02, 0
    /*  +0x124 */ uint32_t    body_coll_masks[2];         // [H] the mask arguments of the body tests; ctor 2, 0
    /*  +0x12c */ I3D_frame  *root_parent_frame;          // [H] the root frame's parent; every lamp is linked to it
    /*  +0x130 */ float       particle_velocity_scale;    // [H] scales the emitted particles' velocity; ctor 1.0
    /*  +0x134 */ uint32_t    sector_refresh_ms;          // [H] ms timer of the 300 ms particle_velocity_scale refresh
    /*  +0x138 */ int32_t     gear_drive_state;           // [H] 0 / 2 = forward, 1 = reverse
    /*  +0x13c */ uint8_t     handbrake_locks_wheels;     // [H] 1 = a handbrake wheel locks
    /*  +0x140 */ int32_t     lock_count;                 // [H] vehicle_lock +1 / -1
    /*  +0x144 */ float       lamp_damage_divisor;        // [H] the lamp hit is scaled by 1 / this
    /*  +0x148 */ float       lamp_damage_threshold;      // [H] lamp hits below it are ignored
    /*  +0x14c */ float       lamp_health_init;           // [H] every lamp record's +0x18 is reset to it
    /*  +0x150 */ float       lamp_break_health;          // [H] a lamp whose health falls below it breaks
    /*  +0x154 */ float       lamp_damage_resist;         // [H] lamp hit reduction, percent
    /*  +0x158 */ std_vector  strips_left;                // [H] {alloc +0x158, begin +0x15c, end +0x160, cap +0x164}; 0xc elements
    /*  +0x168 */ std_vector  strips_right;               // [H] {alloc +0x168, begin +0x16c, end +0x170, cap +0x174}; 0xc elements
    /*  +0x178 */ uint32_t    net_damage_dirty;           // [H] network damage-dirty bits 0x1..0x8
    /*  +0x17c */ std_vector  lights;                     // [H] {alloc +0x17c, begin +0x180, end +0x184, cap +0x188}; 0x20 records (§7)
    /*  +0x18c */ std_vector  special_projectors;         // [H] {alloc +0x18c, begin +0x190, end +0x194, cap +0x198}; 0x1c records (§7)
    /*  +0x19c */ uint32_t    light_state;                // [H] one bit per lamp or mode (§2a)
    /*  +0x1a0 */ uint32_t    light_state_prev;           // [H] light_state of the previous frame
    /*  +0x1a4 */ float       blink_lit_s;                // [H] lit half of the indicator blink; ctor 0.4
    /*  +0x1a8 */ float       blink_dark_s;               // [H] dark half; ctor 0.3
    /*  +0x1ac */ float       blink_timer_s;              // [H] frame time since the last blink toggle
    /*  +0x1b0 */ float       auto_indicator_steer_min;   // [H] |steering| above it lights the automatic indicator; ctor 0.2
    /*  +0x1b4 */ float       light_bar_period_s;         // [H] light-bar phase length; ctor 0.3
    /*  +0x1b8 */ float       light_bar_timer_s;          // [H] frame time since the last light-bar toggle
    /*  +0x1bc */ uint8_t     own_body_friction;          // [H] 1 = the body uses own_friction / own_impulse_factor
    /*  +0x1bd */ uint8_t     own_bump_ranges;            // [H] 1 = the wheels' own bump ranges replace the surface's
    /*  +0x1c0 */ uint32_t    unread_1c0;                 // [U] 0 every frame by vehicle_update_visuals, saved; no reader (.text scan)
    /*  +0x1c4 */ float       side_force_sum;             // [H] per-step sum of the wheels' side force
    /*  +0x1c8 */ float       forward_force_sum;          // [H] per-step sum of the wheels' forward force
    /*  +0x1cc */ float       forward_force_cap;          // [H] sum of every wheel's forward-force cap +0x118
    /*  +0x1d0 */ uint32_t    unread_1d0;                 // [U] 0 by vehicle_game_init, saved; no reader (class census)
    /*  +0x1d4 */ uint8_t     handbrake_sound_latch;      // [H] the handbrake sound played once
    /*  +0x1d8 */ float       size_world[3];              // [H] size_model times the root frame's scale
    /*  +0x1e4 */ float       size_model[3];              // [H] bound-box max - min of the root frame; the quantiser's range
    /*  +0x1f0 */ float       pivot_world[3];             // [H] pivot_model times the root frame's scale
    /*  +0x1fc */ float       pivot_model[3];             // [H] minus the bound-box minimum; the quantiser's origin
    /*  +0x208 */ float       skid_spin_speed_min;        // [H] slip speed a spinning wheel must exceed
    /*  +0x20c */ float       skid_spin_speed_range;      // [H] the divisor of that excess
    /*  +0x210 */ float       skid_volume_rate;           // [H] rise rate of skid_volume per second
    /*  +0x214 */ float       skid_volume;                // [H] the skid handle's volume
    /*  +0x218 */ float       steer_linearity;            // [H] steering linearity [0,1]; ctor 0.5
    /*  +0x21c */ float       clutch_linearity;           // [H] clutch linearity; ctor 0.5
    /*  +0x220 */ uint32_t    unread_220;                 // [U] save pair only; no reader (class census)
    /*  +0x224 */ uint32_t    master_flags;               // [H] bit map in §2b
    /*  +0x228 */ uint32_t    asleep_ms;                  // [H] ms counter of the sleep test
    /*  +0x22c */ float       engine_damage_floor;        // [H] the motor factor at an empty engine pool
    /*  +0x230 */ float       motor_force_factor;         // [H] floor + (1 - floor) * pool / max
    /*  +0x234 */ float       engine_damage;              // [H] engine pool, current (car.md §4b channel 1)
    /*  +0x238 */ float       engine_damage_max;          // [H] engine pool, maximum
    /*  +0x23c */ float       engine_damage_resist;       // [H] hit reduction, percent
    /*  +0x240 */ float       engine_damage_divisor;      // [H] the hit is scaled by max / this
    /*  +0x244 */ float       engine_damage_threshold;    // [H] hits below it are ignored
    /*  +0x248 */ float       smoke_damage_light;         // [H] engine_damage below it makes the light smoke
    /*  +0x24c */ float       smoke_damage_heavy;         // [H] engine_damage below it makes the heavy smoke
    /*  +0x250 */ float       smoke_speed_max;            // [H] the motor smoke stops above this speed
    /*  +0x254 */ int32_t     smoke_particle_light;       // [H] particle id of the light smoke
    /*  +0x258 */ int32_t     smoke_particle_heavy;       // [H] particle id of the heavy smoke
    /*  +0x25c */ uint32_t    smoke_emit_ms;              // [H] ms since the last motor-smoke particle
    /*  +0x260 */ float       engine_point[3];            // [H] the SMOKE_MOTOR frame in BODY space
    /*  +0x26c */ float       gearbox_damage;             // [H] gearbox pool, current (car.md §4b channel 0)
    /*  +0x270 */ float       gearbox_damage_max;         // [H] gearbox pool, maximum
    /*  +0x274 */ float       gearbox_damage_resist;      // [H] hit reduction, percent
    /*  +0x278 */ float       gearbox_damage_divisor;     // [H] the hit is scaled by max / this
    /*  +0x27c */ float       gearbox_damage_threshold;   // [H] hits below it are ignored
    /*  +0x280 */ int32_t     gear_sound_variant;         // [H] which of the two gear sounds plays
    /*  +0x284 */ std_vector  deform_zones_pristine;      // [H] {alloc +0x284, begin +0x288, end +0x28c, cap +0x290}; 0x34 records
    /*  +0x294 */ std_vector  deform_zones;               // [H] {alloc +0x294, begin +0x298, end +0x29c, cap +0x2a0}; 0x34 records (§7)
    /*  +0x2a4 */ int32_t     traction_cap_gear;          // [H] from this gear up, the drive force is capped to the grip
    /*  +0x2a8 */ uint8_t     brake_grip_cap;             // [H] 1 = a wheel's brake force is capped at its grip
    /*  +0x2a9 */ uint8_t     yaw_stability_brake;        // [H] 1 = one wheel brakes against the yaw
    /*  +0x2ac */ S_dyn_coll_record dyn_coll_record;            // [H] the body's dynamic-collision record, mask 0x201 (§7)
    /*  +0x2f4 */ S_dyn_coll_record dyn_coll_record_2;          // [H] a second body record, mask 0x202, box floor lowered by 0.5 (§7)
    /*  +0x33c */ int32_t     skid_mark_pairs;            // [H] skid-mark model pairs per wheel; ctor 50
    /*  +0x340 */ uint8_t     has_driver;                 // [H] a driver sits in the car; = car+0x3b0
    /*  +0x344 */ float       unread_344[4];              // [U] CARINIT [0x184]..[0x187]; no reader (class census)
    /*  +0x354 */ float       head_pos[3];                // [H] the driver's head position
    /*  +0x360 */ uint8_t     in_camera_sector;           // [H] copy of C_actor +0x5f; 1 = the full visual pass
    /*  +0x364 */ void       *callbacks[14];              // [H] the owner callback bank +0x364..+0x398 (§10)
    /*  +0x39c */ float       root_pos[3];                // [H] the root frame's world position
    /*  +0x3a8 */ std_vector  axle_frames;                // [H] {alloc +0x3a8, begin +0x3ac, end +0x3b0, cap +0x3b4}; 0x18 records
    /*  +0x3b8 */ std_vector  axle_side_frames;           // [H] {alloc +0x3b8, begin +0x3bc, end +0x3c0, cap +0x3c4}; 0x18 records
    /*  +0x3c8 */ I3D_frame  *root_frame;                 // [H] the model's root frame
    /*  +0x3cc */ float       turn_radius;                // [M] the frame's turning radius; no reader (class census)
    /*  +0x3d0 */ float       turn_radius_min;            // [H] the smallest turning radius
    /*  +0x3d4 */ float       velocity_dir[3];            // [H] the velocity normalised
    /*  +0x3e0 */ float       centre_box_min[3];          // [H] the CENTRE frame's bounding box minimum, vehicle space
    /*  +0x3ec */ float       centre_box_max[3];          // [H] the CENTRE frame's bounding box maximum
    /*  +0x3f8 */ float       steering_wheel_axis[3];     // [H] the SWHEEL frame's normalised axis
    /*  +0x404 */ float       top_speed;                  // [H] top-gear speed at maximum revs (§6)
    /*  +0x408 */ float       own_friction;               // [H] the body's friction while own_body_friction is set
    /*  +0x40c */ float       own_impulse_factor;         // [H] the body's impulse factor while own_body_friction is set
    /*  +0x410 */ uint8_t     driven_wheel_down;          // [H] a driven wheel touched the ground this step
    /*  +0x414 */ float       centre_of_mass[3];          // [H] world centre of mass
    /*  +0x420 */ float       throttle;                   // [H] the engine throttle (§4)
    /*  +0x424 */ float       throttle_reverse;           // [H] the second throttle channel (§4)
    /*  +0x428 */ float       gear_speed_factor;          // [H] CARINIT GearSpeedFactor * 0.01
    /*  +0x42c */ float       handbrake_force;            // [H] handbrake output scale
    /*  +0x430 */ float       handbrake_ramp_max;         // [H] handbrake ramp denominator
    /*  +0x434 */ float       handbrake_ramp_accum;       // [H] handbrake ramp accumulator
    /*  +0x438 */ float       throttle_out_prev;          // [H] the previous throttle_out
    /*  +0x43c */ float       throttle_reverse_out_prev;  // [M] the previous throttle_reverse_out; no reader (.text scan)
    /*  +0x440 */ float       wheel_force_base;           // [H] CARINIT [0x175] * 1000
    /*  +0x444 */ float       pedal_ramp_max;             // [H] pedal ramp denominator (§4)
    /*  +0x448 */ float       pedal_ramp_accum;           // [H] pedal ramp accumulator
    /*  +0x44c */ float       downforce_factors[4];       // [H] two pairs from CARINIT [0x37]..[0x3a]
    /*  +0x45c */ float       downforce_point_front[3];   // [H] where the front downforce acts
    /*  +0x468 */ float       downforce_point_rear[3];    // [H] where the rear downforce acts
    /*  +0x474 */ float       centre_of_mass_model[3];    // [H] the centre of mass in model space
    /*  +0x480 */ float       centre_of_mass_base[3];     // [H] the CENTRE box min + size * com_box_fraction
    /*  +0x48c */ float       weight_force[3];            // [H] (0, -body_mass * weight_gravity, 0)
    /*  +0x498 */ float       engine_inertia;             // [H] CARINIT [0xc]
    /*  +0x49c */ float       angular_velocity[3];        // [H] world angular velocity
    /*  +0x4a8 */ float       driven_wheel_count;         // [H] the number of driven wheels, as a float
    /*  +0x4ac */ uint8_t     horn_on;                    // [H] the horn sounds
    /*  +0x4ad */ uint8_t     siren_on;                   // [H] the siren sounds
    /*  +0x4ae */ uint8_t     sounds_enabled;             // [H] the car's sounds may play; ctor 1
    /*  +0x4b0 */ float       handbrake_out;              // [H] (ramp_accum / ramp_max) * handbrake_force
    /*  +0x4b4 */ float       inertia_inv[3];             // [H] the effective inverse inertia, per axis
    /*  +0x4c0 */ float       inertia_inv_base[3];        // [H] 1 / CARINIT [0x174], [0x172], [0x173]
    /*  +0x4cc */ std_vector  exhausts;                   // [H] {alloc +0x4cc, begin +0x4d0, end +0x4d4, cap +0x4d8}; 0x18 records
    /*  +0x4dc */ int32_t     exhaust_particle;           // [H] particle id of the exhaust
    /*  +0x4e0 */ uint32_t    exhaust_emit_ms;            // [H] ms since the last exhaust particle
    /*  +0x4e4 */ float       exhaust_speed_max;          // [H] no exhaust above this speed
    /*  +0x4e8 */ float       exhaust_exit_speed_min;     // [H] the exhaust exit speed at idle
    /*  +0x4ec */ float       exhaust_exit_speed_max;     // [H] the exhaust exit speed at max revs
    /*  +0x4f0 */ float       torque_idle;                // [H] the torque curve at idle revs
    /*  +0x4f4 */ float       torque_peak;                // [H] the torque curve's flat top
    /*  +0x4f8 */ float       torque_redline;             // [H] the torque curve at the redline
    /*  +0x4fc */ float       torque_max_revs;            // [H] the torque curve at max revs
    /*  +0x500 */ float       torque_peak_from_revs;      // [H] where the flat top starts, revs per second
    /*  +0x504 */ float       torque_peak_to_revs;        // [H] where the flat top ends, revs per second
    /*  +0x508 */ float       torque_redline_revs;        // [H] CARINIT RedLineRPM / 60, as redline_revs
    /*  +0x50c */ uint8_t     camera_inside_request;      // [H] the camera's inside flag, not yet applied
    /*  +0x50d */ uint8_t     camera_inside;              // [H] selects the interior or exterior sound handle
    /*  +0x510 */ float       standstill_shift_timer;     // [H] time the automatic gearbox has waited at standstill
    /*  +0x514 */ float       fuel_rate;                  // [H] fuel used per second at full load
    /*  +0x518 */ float       weight_gravity;             // [H] CARINIT Cohesion with road: the g of the weight force
    /*  +0x51c */ float       engine_load;                // [H] 0..1 engine load: the engine banks' volume blend
    /*  +0x520 */ float       speed_limit;                // [H] the AI speed limiter
    /*  +0x524 */ uint8_t     steer_analog;               // [H] 1 = steering input is analog (§2c)
    /*  +0x525 */ uint8_t     accel_analog;               // [H] 1 = accelerator input is analog (§2c)
    /*  +0x526 */ uint8_t     brake_analog;               // [H] 1 = brake input is analog: no pedal ramp (§2c)
    /*  +0x527 */ uint8_t     clutch_analog;              // [H] 1 = clutch input is analog (§2c)
    /*  +0x528 */ std_vector  body_coll_heads;            // [H] {alloc +0x528, begin +0x52c, end +0x530, cap +0x534}; S_COLBODY_HEAD, 0x74 (§7)
    /*  +0x538 */ char       *root_name;                  // [H] a heap copy of the root frame's name
    /*  +0x53c */ int32_t     axle_count;                 // [H] wheel_count / 2
    /*  +0x540 */ int32_t     wheel_count;                // [H] length of the wheel array +0xcb4
    /*  +0x544 */ float       ramp_rate;                  // [H] per-call step of the pedal and handbrake ramps
    /*  +0x548 */ uint8_t     automatic_gearbox;          // [H] 1 = automatic, 0 = manual; = car+0x5b8
    /*  +0x54c */ float       upshift_revs[12];           // [H] per gear, index gear + 1
    /*  +0x57c */ float       downshift_revs[12];         // [H] per gear, index gear + 1
    /*  +0x5ac */ float       engine_power;               // [H] CARINIT Power * 1000 * gear_speed_factor
    /*  +0x5b0 */ uint32_t    unused_5b0;                 // [U] no access; no reader (class census)
    /*  +0x5b4 */ float       drag_factor_a;              // [H] CARINIT [0x35], 0.029..1.331
    /*  +0x5b8 */ float       drag_factor_b;              // [H] CARINIT [0x36], at least 0
    /*  +0x5bc */ float       throttle_out;               // [H] throttle after the drivetrain, from +0x420
    /*  +0x5c0 */ float       throttle_reverse_out;       // [H] from +0x424
    /*  +0x5c4 */ float       engine_revs;                // [H] engine speed in revolutions per second
    /*  +0x5c8 */ float       engine_revs_prev;           // [M] the previous engine_revs; no reader (.text scan)
    /*  +0x5cc */ uint8_t     driven_wheel_spun;          // [H] a driven wheel spun this frame
    /*  +0x5d0 */ float       road_load_torque;           // [H] the road-load torque of the driven wheels
    /*  +0x5d4 */ float       idle_revs_half;             // [M] idle_revs * 0.5; no reader (.text scan)
    /*  +0x5d8 */ int32_t     gear;                       // [H] the engaged gear: -1 reverse, 0 neutral, 1.. forward
    /*  +0x5dc */ int32_t     gear_selected;              // [H] the gear asked for
    /*  +0x5e0 */ float       redline_revs;               // [H] CARINIT RedLineRPM / 60
    /*  +0x5e4 */ int32_t     gear_count;                 // [H] CARINIT GEARS, clamped to 1..10
    /*  +0x5e8 */ float       gear_ratio[12];             // [H] index gear + 1: reverse, neutral (= 1st), 1st..10th
    /*  +0x618 */ float       speed;                      // [H] the scalar speed; = car+0x688
    /*  +0x61c */ float       speed_prev;                 // [H] the previous speed
    /*  +0x620 */ float       speed_along_wheels;         // [H] the speed along the wheels' summed direction
    /*  +0x624 */ float       speed_along_wheels_prev;    // [M] the previous speed_along_wheels; no reader (.text scan)
    /*  +0x628 */ float       drive_circumference;        // [H] 2 pi r of a driven wheel
    /*  +0x62c */ float       drive_radius;               // [H] the radius of the driven wheel that sets drive_circumference
    /*  +0x630 */ float       step_distance;              // [H] the distance travelled in the frame
    /*  +0x634 */ float       brake_force;                // [H] the brake output (the accelerator in automatic reverse, §4)
    /*  +0x638 */ float       engine_brake;               // [H] the engine braking at max revs
    /*  +0x63c */ float       launch_revs[3];             // [H] the launch rev thresholds, revs per second
    /*  +0x648 */ float       launch_target;              // [H] the launch clutch target, -1 = none
    /*  +0x64c */ float       clutch_capacity;            // [H] the clutch's torque capacity
    /*  +0x650 */ float       clutch_engage_rate;         // [H] 1 / CARINIT [0xf], per second
    /*  +0x654 */ float       clutch_disengage_rate;      // [H] 1 / CARINIT [0xe], per second
    /*  +0x658 */ float       clutch_engagement;          // [H] the clutch engagement, 0..1
    /*  +0x65c */ float       clutch_out;                 // [H] clutch output
    /*  +0x660 */ float       clutch_pedal_smoothed;      // [H] clutch_out after the smoothing
    /*  +0x664 */ uint32_t    gearbox_state;              // [H] the gearbox state machine (§2b); game_init 0x20
    /*  +0x668 */ float       idle_revs;                  // [H] CARINIT idling rpm / 60
    /*  +0x66c */ uint8_t     engine_dead;                // [H] latched when the engine pool runs out
    /*  +0x670 */ float       mass;                       // [H] the mass the impulse code divides by
    /*  +0x674 */ float       body_mass;                  // [H] CARINIT Weight
    /*  +0x678 */ float       com_box_fraction[3];        // [H] CARINIT [0x42]..[0x44], 0..1
    /*  +0x684 */ float       steer_lock[2];              // [H] lock at standstill / at top speed (§6)
    /*  +0x68c */ float       steer_lock_now;             // [H] the current lock (§6)
    /*  +0x690 */ float       steer_rate[4];              // [H] turn-in / return rate, standstill then top speed (§6)
    /*  +0x6a0 */ float       steer_target;               // [H] angle * +0x68c
    /*  +0x6a4 */ float       steer_angle;                // [H] moves toward +0x6a0 (§6)
    /*  +0x6a8 */ float       steer_split_max;            // [H] wheel split at full lock (§6)
    /*  +0x6b0 */ float       steer_split;                // [H] the current split (§6)
    /*  +0x6b4 */ uint8_t     engine_on_mirror;           // [H] follows +0xca8
    /*  +0x6b8 */ float       load_torque_min;            // [H] the torque where engine_load is 0
    /*  +0x6bc */ float       load_torque_max;            // [H] the torque where engine_load is 1
    /*  +0x6c0 */ float       load_rise_rate;             // [H] engine_load rise per second
    /*  +0x6c4 */ float       load_fall_rate;             // [H] engine_load fall per second
    /*  +0x6c8 */ I3D_sound  *int_snd_engine_start;       // [M] interior engine start handle; never opened (§8e)
    /*  +0x6cc */ I3D_sound  *int_snd_engine_stop;        // [M] interior engine stop handle; never opened (§8e)
    /*  +0x6d0 */ I3D_sound  *int_snd_start_fail;         // [M] interior failed start handle; never opened (§8e)
    /*  +0x6d4 */ I3D_sound  *int_snd_engine_load[5];     // [M] interior on-load engine bank handle; never opened (§8e)
    /*  +0x6e8 */ I3D_sound  *int_snd_engine_coast[5];    // [M] interior coast engine bank handle; never opened (§8e)
    /*  +0x6fc */ I3D_sound  *int_snd_engine_reverse[5];  // [M] interior reverse engine bank handle; never opened (§8e)
    /*  +0x710 */ I3D_sound  *int_snd_engine_idle;        // [M] interior idle handle; never opened (§8e)
    /*  +0x714 */ I3D_sound  *int_snd_environment;        // [M] interior environment handle; never opened (§8e)
    /*  +0x718 */ I3D_sound  *int_snd_horn;               // [M] interior horn handle; never opened (§8e)
    /*  +0x71c */ I3D_sound  *int_snd_siren;              // [M] interior siren handle; never opened (§8e)
    /*  +0x720 */ I3D_sound  *int_snd_handbrake;          // [M] interior handbrake handle; never opened (§8e)
    /*  +0x724 */ I3D_sound  *int_snd_gear[2];            // [M] interior gear-change handle; never opened (§8e)
    /*  +0x72c */ I3D_sound  *int_snd_skid;               // [M] interior skid handle; never opened (§8e)
    /*  +0x730 */ I3D_sound  *int_snd_brake_loop;         // [M] interior brake loop handle; never opened (§8e)
    /*  +0x734 */ I3D_sound  *int_snd_wheel_thump;        // [M] interior wheel thump handle; never opened (§8e)
    /*  +0x738 */ I3D_sound  *int_snd_tyre_burst;         // [M] interior tyre burst handle; never opened (§8e)
    /*  +0x73c */ I3D_sound  *int_snd_crash[4];           // [M] interior crash tier 0..3 handle; never opened (§8e)
    /*  +0x74c */ I3D_sound  *int_snd_crash_alt[4];       // [M] interior alternate crash tier 0..3 handle; never opened (§8e)
    /*  +0x75c */ I3D_sound  *int_snd_door[2];            // [M] interior door handle; never opened (§8e)
    /*  +0x764 */ I3D_sound  *int_snd_puncture;           // [M] interior puncture handle; never opened (§8e)
    /*  +0x768 */ S_engine_sound_bank int_bank_load;              // [H] the on-load engine bank (§7)
    /*  +0x810 */ S_engine_sound_bank int_bank_coast;             // [H] the coast engine bank (§7)
    /*  +0x8b8 */ S_engine_sound_bank_nomute int_bank_reverse;           // [H] the reverse engine bank (§7)
    /*  +0x958 */ float       int_idle_revs_end;          // [H] revs where the idle sound ends
    /*  +0x95c */ float       int_idle_revs_start;        // [H] revs where the idle sound starts
    /*  +0x960 */ float       int_idle_volume;            // [H] the idle sound's volume
    /*  +0x964 */ float       int_environment_pitch[2];   // [H] the environment sound's pitch range
    /*  +0x96c */ float       int_environment_speed_start; // [H] speed where the environment sound starts
    /*  +0x970 */ float       int_environment_speed_cap;  // [H] the environment sound's speed cap
    /*  +0x974 */ float       int_environment_speed_full; // [H] speed of the environment sound's full volume
    /*  +0x978 */ float       unread_978;                 // [U] CARINIT +0xc4c, at most 1.0; no reader (.text scan)
    /*  +0x97c */ float       int_puncture_pitch[2];      // [H] the puncture sound's pitch range
    /*  +0x984 */ float       int_puncture_speed_start;   // [H] speed where the puncture sound starts
    /*  +0x988 */ float       int_puncture_speed_cap;     // [H] the puncture sound's speed cap
    /*  +0x98c */ float       int_puncture_speed_full;    // [H] speed of the puncture sound's full volume
    /*  +0x990 */ float       unread_990;                 // [U] 1.0 by vehicle_apply_carinit_params; no reader (.text scan)
    /*  +0x994 */ float       int_crash_impulse[4];       // [H] impulse bounds of the crash tiers 0..3
    /*  +0x9a4 */ uint8_t     int_crash_mute_grounded[4]; // [H] per tier: no sound while a driven wheel is down
    /*  +0x9a8 */ I3D_sound  *ext_snd_engine_start;       // [H] exterior engine start handle (§8e)
    /*  +0x9ac */ I3D_sound  *ext_snd_engine_stop;        // [H] exterior engine stop handle (§8e)
    /*  +0x9b0 */ I3D_sound  *ext_snd_start_fail;         // [H] exterior failed start handle (§8e)
    /*  +0x9b4 */ I3D_sound  *ext_snd_engine_loop;        // [H] exterior single engine loop handle (§8e)
    /*  +0x9b8 */ I3D_sound  *ext_snd_engine_load[5];     // [H] exterior on-load engine bank handle (§8e)
    /*  +0x9cc */ I3D_sound  *ext_snd_engine_coast[5];    // [H] exterior coast engine bank handle (§8e)
    /*  +0x9e0 */ I3D_sound  *ext_snd_engine_reverse[5];  // [H] exterior reverse engine bank handle (§8e)
    /*  +0x9f4 */ I3D_sound  *ext_snd_engine_idle;        // [H] exterior idle handle (§8e)
    /*  +0x9f8 */ I3D_sound  *ext_snd_environment;        // [H] exterior environment handle (§8e)
    /*  +0x9fc */ I3D_sound  *ext_snd_horn;               // [H] exterior horn handle (§8e)
    /*  +0xa00 */ I3D_sound  *ext_snd_siren;              // [H] exterior siren handle (§8e)
    /*  +0xa04 */ I3D_sound  *ext_snd_handbrake;          // [H] exterior handbrake handle (§8e)
    /*  +0xa08 */ I3D_sound  *ext_snd_gear[2];            // [H] exterior gear-change handle (§8e)
    /*  +0xa10 */ I3D_sound  *ext_snd_skid;               // [H] exterior skid handle (§8e)
    /*  +0xa14 */ I3D_sound  *ext_snd_brake_loop;         // [M] exterior brake-loop handle; opened, never started (§8e)
    /*  +0xa18 */ I3D_sound  *ext_snd_wheel_thump;        // [H] exterior wheel thump handle (§8e)
    /*  +0xa1c */ I3D_sound  *ext_snd_tyre_burst;         // [H] exterior tyre burst handle (§8e)
    /*  +0xa20 */ I3D_sound  *ext_snd_crash[4];           // [H] exterior crash tier 0..3 handle (§8e)
    /*  +0xa30 */ I3D_sound  *ext_snd_crash_alt[4];       // [H] exterior alternate crash tier 0..3 handle (§8e)
    /*  +0xa40 */ I3D_sound  *ext_snd_door[2];            // [H] exterior door handle (§8e)
    /*  +0xa48 */ I3D_sound  *ext_snd_puncture;           // [H] exterior puncture handle (§8e)
    /*  +0xa4c */ float       ext_engine_loop_pitch[2];   // [H] the single engine loop's pitch at idle / max revs
    /*  +0xa54 */ float       ext_engine_loop_revs_full;  // [H] revs of the single loop's full volume
    /*  +0xa58 */ float       ext_engine_loop_volume;     // [H] the single engine loop's volume
    /*  +0xa5c */ S_engine_sound_bank ext_bank_load;              // [H] the on-load engine bank (§7)
    /*  +0xb04 */ S_engine_sound_bank ext_bank_coast;             // [H] the coast engine bank (§7)
    /*  +0xbac */ S_engine_sound_bank_nomute ext_bank_reverse;           // [H] the reverse engine bank (§7)
    /*  +0xc4c */ float       ext_idle_revs_end;          // [H] revs where the idle sound ends
    /*  +0xc50 */ float       ext_idle_revs_start;        // [H] revs where the idle sound starts
    /*  +0xc54 */ float       ext_idle_volume;            // [H] the idle sound's volume
    /*  +0xc58 */ float       ext_environment_pitch[2];   // [H] the environment sound's pitch range
    /*  +0xc60 */ float       ext_environment_speed_start; // [H] speed where the environment sound starts
    /*  +0xc64 */ float       ext_environment_speed_cap;  // [H] the environment sound's speed cap
    /*  +0xc68 */ float       ext_environment_speed_full; // [H] speed of the environment sound's full volume
    /*  +0xc6c */ float       unread_c6c;                 // [U] CARINIT +0x110c, at most 1.0; no reader (.text scan)
    /*  +0xc70 */ float       ext_puncture_pitch[2];      // [H] the puncture sound's pitch range
    /*  +0xc78 */ float       ext_puncture_speed_start;   // [H] speed where the puncture sound starts
    /*  +0xc7c */ float       ext_puncture_speed_cap;     // [H] the puncture sound's speed cap
    /*  +0xc80 */ float       ext_puncture_speed_full;    // [H] speed of the puncture sound's full volume
    /*  +0xc84 */ float       unread_c84;                 // [U] 1.0 by vehicle_apply_carinit_params; no reader (.text scan)
    /*  +0xc88 */ float       ext_crash_impulse[4];       // [H] impulse bounds of the crash tiers 0..3
    /*  +0xc98 */ uint8_t     ext_crash_mute_grounded[4]; // [H] per tier: no sound while a driven wheel is down
    /*  +0xc9c */ float       engine_sound_fade_s;        // [H] time since the engine sound started, capped at 0.3
    /*  +0xca0 */ float       physics_substep_s;          // [H] sub-step length; ctor 0.028 (§8d)
    /*  +0xca4 */ I3D_frame  *steering_wheel_frame;       // [H] the SWHEEL frame, 0 when absent
    /*  +0xca8 */ uint8_t     engine_on;                  // [H] the engine runs
    /*  +0xcac */ float       fuel;                       // [H] a start is refused at 0.02 or below
    /*  +0xcb0 */ void       *wheel_node_list;            // [H] head of the wheel-record linked list
    /*  +0xcb4 */ void      **wheels;                     // [H] wheel record pointers (§7)
    /*  +0xcb8 */ float       axis_forward[3];            // [H] world forward: row 2 of +0xd64
    /*  +0xcc4 */ float       axis_right[3];              // [H] world right: row 0 of +0xd64
    /*  +0xcd0 */ float       axis_up[3];                 // [H] world up: row 1 of +0xd64
    /*  +0xcdc */ uint8_t     wake_request;               // [H] set by every function that changes the motion or the mass
    /*  +0xce0 */ float       velocity_heading_angle;     // [H] angle between axis_forward and the velocity
    /*  +0xce4 */ uint32_t    feature_flags;              // [H] bit map in §2b; ctor 0xdfff
    /*  +0xce8 */ uint32_t    feature_flags_prev;         // [H] feature_flags of the previous frame
    /*  +0xcec */ uint32_t    build_flags;                // [H] bit 0x1 no tyre width or skid marks, 0x2 missing parts accepted
    /*  +0xcf0 */ C_car      *owner_car;                  // [H] the owning C_car; the ECX of every bank callback
    /*  +0xcf4 */ uint32_t    unread_cf4[10];             // [U] 0 by vehicle_game_init; no reader (.text scan)
    /*  +0xd1c */ std_vector  seats;                      // [H] {alloc +0xd1c, begin +0xd20, end +0xd24, cap +0xd28}; S_SEAT 0x24 (§7)
    /*  +0xd2c */ std_vector  door_windows;               // [H] {alloc +0xd2c, begin +0xd30, end +0xd34, cap +0xd38}; 0x58 records (§7)
    /*  +0xd3c */ std_vector  ballast;                    // [H] {alloc +0xd3c, begin +0xd40, end +0xd44, cap +0xd48}; S_BALLAST 0x1c (§7)
    /*  +0xd4c */ float       ext_force_sum[3];           // [H] the accumulated external force
    /*  +0xd58 */ float       ext_torque_sum[3];          // [H] the accumulated external torque
    /*  +0xd64 */ S_matrix    world_matrix;               // [H] the cached body matrix
    /*  +0xda4 */ S_matrix    world_matrix_inv;           // [H] its inverse
    /*  +0xde4 */ I3D_frame  *body_frame;                 // [H] the BODY frame
    /*  +0xde8 */ S_CARINIT   init;                       // [H] 0x1124 bytes: the per-car tuning block
    /* +0x1f0c */ float       velocity[3];                // [H] world velocity; = car+0x1f7c
    /* +0x1f18 */ float       max_revs;                   // [H] CARINIT maximum rpm / 60
    /* +0x1f1c */ float       fuel_capacity;              // [H] tank capacity
    /* +0x1f20 */ float       odometer;                   // [H] distance along wheel 0's rolling direction
    /* +0x1f24 */ uint8_t     engine_cut_request;         // [H] 1 = the engine is held off
    /* +0x1f25 */ uint8_t     engine_on_saved;            // [H] engine_on saved by the cut-out
    /* +0x1f26 */ uint8_t     special_dyn_colls;          // [H] 1 = the +0x1f38 records replace +0x2ac
    /* +0x1f28 */ std_vector  special_coll_boxes;         // [H] {alloc +0x1f28, begin +0x1f2c, end +0x1f30, cap +0x1f34}; S_vector pairs
    /* +0x1f38 */ std_vector  special_dyn_coll_records;   // [H] {alloc +0x1f38, begin +0x1f3c, end +0x1f40, cap +0x1f44}; 0x48 records
} C_Vehicle;
```

`std_vector` is the 16-byte MSVC vector object: an allocator byte at `+0x0`, then begin, end and
capacity pointers. `S_matrix` is a 4 x 4 float matrix (`0x40` bytes).

**Why these array sizes.**

| Field | Count | What each entry is |
|---|---|---|
| `rest_anchor`, `steer_to_point`, `front_axle_mid`, `rear_axle_mid`, `step_start_pos`, `step_start_forward`, `step_start_up`, `step_start_right`, `size_world`, `size_model`, `pivot_world`, `pivot_model`, `engine_point`, `head_pos`, `root_pos`, `velocity_dir`, `centre_box_min`, `centre_box_max`, `steering_wheel_axis`, `centre_of_mass`, `downforce_point_front`, `downforce_point_rear`, `centre_of_mass_model`, `centre_of_mass_base`, `weight_force`, `angular_velocity`, `inertia_inv`, `inertia_inv_base`, `com_box_fraction`, `axis_forward`, `axis_right`, `axis_up`, `ext_force_sum`, `ext_torque_sum`, `velocity` | 3 | x, y, z |
| `wheel_coll_masks`, `body_coll_masks` | 2 | the last two mask arguments of the collision test, in order |
| `callbacks` | 14 | `+0x364`..`+0x398`, one function pointer per slot: 13 slots with a setter and `+0x384`, which has none (§10) |
| `upshift_revs`, `downshift_revs`, `gear_ratio` | 12 | one per gear, indexed `gear + 1` for gears -1..10: reverse, neutral, then 1st..10th. `vehicle_apply_carinit_params` clamps the CARINIT `GEARS` count to 1..10 and fills entries 2..11 from the record; the neutral ratio is a copy of 1st. Each array ends where the next begins |
| `steer_lock` | 2 | the lock at standstill, then at top speed (§6) |
| `steer_rate` | 4 | turn-in at standstill, return at standstill, turn-in at top speed, return at top speed (§6) |
| `dyn_coll_record`, `dyn_coll_record_2` | 1 | one `0x48`-byte `S_dyn_coll_record` each (§7) |
| `unread_344` | 4 | CARINIT `[0x184]`..`[0x187]` in order |
| `downforce_factors` | 4 | two pairs from CARINIT `[0x37]`..`[0x3a]`: the front term, then the rear term |
| `launch_revs` | 3 | CARINIT `[0x10]`, `[0x11]`, `[0x12]`, rpm / 60 |
| `int_snd_engine_load`, `int_snd_engine_coast`, `int_snd_engine_reverse`, `ext_snd_engine_load`, `ext_snd_engine_coast`, `ext_snd_engine_reverse` | 5 | one handle per sound of the bank, the bank's five CARINIT names in order; entry `k` plays the bank record's entry `k` (§7) |
| `int_snd_gear`, `ext_snd_gear` | 2 | the two gear sounds; `gear_sound_variant` picks one |
| `int_snd_crash`, `int_snd_crash_alt`, `ext_snd_crash`, `ext_snd_crash_alt`, `int_crash_impulse`, `ext_crash_impulse`, `int_crash_mute_grounded`, `ext_crash_mute_grounded` | 4 | one per crash tier 0..3 (§8e) |
| `int_snd_door`, `ext_snd_door` | 2 | index 0 when `vehicle_play_door_sound`'s flag argument is set, 1 when it is clear |
| `int_environment_pitch`, `ext_environment_pitch`, `int_puncture_pitch`, `ext_puncture_pitch`, `ext_engine_loop_pitch` | 2 | the pitch minimum, then the maximum |
| `unread_cf4` | 10 | the ten dwords `vehicle_game_init` zeroes |

**The three world axes** are the rows of the cached body matrix `+0xd64`:
`vehicle_reset_from_frame` seeds forward `+0xcb8` = (0, 0, 1), right `+0xcc4` = (1, 0, 0) and up
`+0xcd0` = (0, 1, 0) and multiplies them by the body matrix; `vehicle_update_cached_transform` copies
row 0 into `+0xcc4`, row 1 into `+0xcd0` and row 2 into `+0xcb8`. Every consumer of the right axis
uses it for left and right: the seat swap of `entity_action_tick` negates it per seat side, and
`car_update` takes the sign of its slip angle from it. The wheel contact point is
`hub - radius * up`.

## 2. Field evidence

Writers and readers are the class's own functions (a census of `this`-relative accesses, plus a
displacement census of `vehicle_init`, which reaches `this` through a stack slot), and the external
accesses read in the bodies named; the external users not read are Open item 6.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x000` | `vtable` | H | `vehicle_ctor`, `vehicle_dtor` | the virtual call of `vehicle_open_sound` | the PS2 vtable also has one method, `OpenSnd` |
| `+0x004` | `rest_ms` | H | `vehicle_game_init`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` (0 on a contact latch) | `vehicle_tick_drivetrain` | past 4000 with `master_flags` `0x2`, bit `0x1` is cleared: the car sleeps |
| `+0x008` | `rest_anchor[3]` | H | `vehicle_game_init`, `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain` | reset to `+0x39c`, with `+0x04` = 0, once the root is 0.15 m or more away |
| `+0x014` | `burst_tyre_quiet_ms` | H | `vehicle_game_init` (30000), `vehicle_reset` (30000 on a repair), `vehicle_update_visuals` | `vehicle_update_visuals` | + frame ms, 0 while any wheel has bits `0x80000000` and `0x8` and neither `0x20` nor `0x40000000`, held at 1000; below 1000 `vehicle_update_flat_tyre_sound(1)`, else `(0)` |
| `+0x018` | `strip_scroll_rate` | H | `vehicle_apply_carinit_params` (CARINIT `+0x5f8`) | `vehicle_update_strips` | phase += ((revs - idle) / (max - idle)) * rate * dt |
| `+0x01c` | `strip_phase_left` | H | `vehicle_game_init` (0), `vehicle_update_strips` | `vehicle_update_strips` | stored into each strip frame's UV-shift object (frame `+0x1e0`, its `+0x5c`) when the frame is a visual tagged `OBJ_` |
| `+0x020` | `strip_phase_right` | H | `vehicle_game_init` (0), `vehicle_update_strips` | `vehicle_update_strips` | as `+0x01c` |
| `+0x024` | `idle_throttle_floor` | H | `vehicle_apply_carinit_params` (`f / (f + +0x4f0)`, `f = idle_revs / max_revs * +0x638`) | `vehicle_tick_drivetrain`, `vehicle_compute_motor_force` | an asleep car whose throttles are both at or below it only turns its revs |
| `+0x028` | `contact_latch` | H | `vehicle_game_init`, `vehicle_update_visuals`, `vehicle_wheel_contact_solve`, `vehicle_body_coll_resolve` | `vehicle_update_visuals` | 1 when a wheel's or the body's contact record has flag `0x10000`; `vehicle_update_visuals` copies it into bit `0x2000000` of `+0x224`, sets `master_flags` `0x1`, zeroes `+0x228` and `+0x04`, and clears it |
| `+0x02c` | `bumps_enabled` | H | `vehicle_game_init` (1.0) | `vehicle_update_visuals` | the only store in the image (a scan of the class and of every `car_`, `driver_`, `pursuit_event_`, `traffic_`, `player_`, `police_` function) |
| `+0x030` | `steer_to_point[3]` | H | `vehicle_game_init` (0), `driver_turn` (as car+0xa0) | `vehicle_move` | the driver's `+0xc4` point + 20 * its `+0xd0` direction; the angle to it / `+0x68c`, signed by the right axis and clamped to 1, goes through `vehicle_set_steer` |
| `+0x03c` | `steer_to_point_on` | H | `vehicle_game_init`, `driver_turn` (1), `driver_tick` (0) (both as car+0xac), `vehicle_update_visuals` (0 every frame) | `vehicle_move` | - |
| `+0x040` | `grip_bonus` | H | `vehicle_game_init` (0), `driver_tick` (0.3 in its state 2, as car+0xb0) | `vehicle_move` (2) | added to wheel `+0xa0` / `+0xa4` |
| `+0x044` | `drive_force_scale` | H | `vehicle_game_init` (1.0), `car_update` (as car+0xb4 = car+0x201c, every frame) | `vehicle_tick_drivetrain` (5), `vehicle_update_motor_revs`, `vehicle_move` | 1.0 on the flat, raised on a slope (the forward axis' y between 0.17 and 0.29) while car `+0x2020` is below 3.4, never below car `+0x2018` |
| `+0x048` | `body_hit_last_step` | H | `vehicle_game_init` (1), `vehicle_move` | `vehicle_move` | a hit in the previous step forces the next resolve |
| `+0x04c` | `front_axle_mid[3]` | H | `vehicle_game_init` | `vehicle_wheel_contact_solve` | times the body matrix: the front probe point. With `feature_flags` `0x800` clear and the car moving, one vertical line test per axle gives a ground plane; a wheel whose rest z (`+0x18`) is above `+0x488` uses the front plane |
| `+0x058` | `rear_axle_mid[3]` | H | `vehicle_game_init` | `vehicle_wheel_contact_solve` | the rear probe point; the other wheels use the rear plane |
| `+0x064` | `substep_count` | H | `vehicle_substep_move` | `vehicle_move` | the body collision runs only on the last sub-step unless `feature_flags` `0x400` |
| `+0x066` | `substep_index` | H | `vehicle_substep_move` (0), `vehicle_move` (+1) | `vehicle_move` | - |
| `+0x068` | `step_start_matrix` | H | `vehicle_substep_move`, `vehicle_move` (with `feature_flags` `0x400`) | `vehicle_body_coll_resolve` | the moving-box test of a head with its box byte set |
| `+0x0a8` | `step_start_matrix_inv` | H | `vehicle_substep_move`, `vehicle_move` (with `feature_flags` `0x400`) | `vehicle_move` | `vehicle_move` copies it into a local (`0x005320ef`), zeroes the translation row and turns a vector into body space with it (`S_vector::operator*`) |
| `+0x0e8` | `step_start_pos[3]` | H | `vehicle_substep_move` | `vehicle_move` | with the three axes below, the pose restored after a body hit |
| `+0x0f4` | `step_start_forward[3]` | H | `vehicle_substep_move` | `vehicle_move` | as `+0x0e8` |
| `+0x100` | `step_start_up[3]` | H | `vehicle_substep_move` | `vehicle_move` | as `+0x0e8` |
| `+0x10c` | `step_start_right[3]` | H | `vehicle_substep_move` | `vehicle_move` | as `+0x0e8` |
| `+0x118` | `dyn_colls_off` | H | `vehicle_game_init`, `vehicle_set_dynamic_colls_off` | `vehicle_save_state`, `vehicle_set_dynamic_colls_off`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals` | `vehicle_set_dynamic_colls_off` changes it only on a change and then inserts (0) or erases (1) `+0x2ac` or the `+0x1f38` records |
| `+0x119` | `has_roof` | H | `vehicle_ctor` (1), `vehicle_apply_carinit_params` (CARINIT byte `+0x5fc`) | `car_on_collision_apply_damage` (as car+0x189) | the hit-from-above occupant damage applies only to cars without it (`def-files-docs/vehicles-bin.md` §4c) |
| `+0x11a` | `has_strips` | H | `vehicle_game_init` | `vehicle_tick_drivetrain`, `vehicle_update_visuals`, `vehicle_move` | the drive force is shared by each driven wheel's grip `+0xa0` against its side's sum instead of being divided per wheel; with `+0x410` it lets the automatic shift run |
| `+0x11c` | `wheel_coll_masks[2]` | H | `vehicle_ctor` | `vehicle_wheel_contact_solve` (3 each) | the last two arguments of `collision_test_line_vertical_thunk` |
| `+0x124` | `body_coll_masks[2]` | H | `vehicle_ctor` | `vehicle_body_coll_resolve` (3 each) | the last two arguments of `collision_test_sphere_swept_thunk` and of the moving-box test |
| `+0x12c` | `root_parent_frame` | H | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init`, `vehicle_update_visuals` (every frame) | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_apply_light_state` | frame `+0x120`; every lamp and special-projector object is linked to it (vtable `+0x2c`) |
| `+0x130` | `particle_velocity_scale` | H | `vehicle_ctor`, `vehicle_game_init`, `vehicle_update_visuals` | `vehicle_emit_body_collision_particles`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_particles`, `vehicle_update_wheel_particles` | `vehicle_update_visuals` stores the scene's slot `+0x68` value at the centre of mass every 300 ms (`+0x134`) while `+0x360` is set |
| `+0x134` | `sector_refresh_ms` | H | `vehicle_ctor`, `vehicle_game_init` (0), `vehicle_update_visuals` | `vehicle_update_visuals` | - |
| `+0x138` | `gear_drive_state` | H | `vehicle_apply_carinit_params`, `vehicle_ctor` | `vehicle_move`, `vehicle_set_brake`, `vehicle_set_gear`, `vehicle_set_power`, `vehicle_tick_drivetrain`, `vehicle_update_motor_revs`, `vehicle_update_visuals` | `vehicle_set_gear` refuses a reverse gear while it is 1 |
| `+0x13c` | `handbrake_locks_wheels` | H | `vehicle_game_init`, `vehicle_load_state`, `pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_dispatch_escape_vehicle`, `pursuit_event_remove_vehicle`, `pursuit_event_save_game_load` (as car+0x1ac) | `vehicle_move`, `vehicle_save_state` | set: a handbrake wheel's force is minus its grip along its slip and it gets wheel bit `0x20`; clear: the handbrake adds `+0x4b0` along the rolling direction. Set on every car a chase record takes |
| `+0x140` | `lock_count` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_lock` | `vehicle_lock`, `vehicle_save_state`, `vehicle_tick_drivetrain` | bit `0x10000000` of `+0x224` follows it |
| `+0x144` | `lamp_damage_divisor` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1bf]` km/h to m/s, times `+0x670`) | `vehicle_deform` | `lamp+0x18 -= ((F - F * +0x154 / 100) / +0x144) * +0x14c` for every unbroken lamp within the impact radius whose zone is not type 5 |
| `+0x148` | `lamp_damage_threshold` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1c0]` km/h to m/s, times `+0x670`) | `vehicle_deform` | the lamp loss runs only above it or with argument bit `0x1`, never with argument bit `0x2` |
| `+0x14c` | `lamp_health_init` | H | `vehicle_apply_carinit_params` | `vehicle_deform`, `vehicle_init_damage` | read per lamp by `vehicle_init_damage`; `vehicle_deform` also scales the lamp loss by it (`+0x144`) |
| `+0x150` | `lamp_break_health` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1c2]`) | `vehicle_deform` (2) | sets lamp `+0x14` bit `0x1`, also when a type-5 zone passes its health to its lamps |
| `+0x154` | `lamp_damage_resist` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1c3]`) | `vehicle_deform` | `+0x144`..`+0x154` are the lamp damage block, in the shape and formula of the engine and gearbox pools; a type-5 zone takes the same words |
| `+0x158` | `strips_left` | H | `vehicle_ctor`, `vehicle_init_strip_and_smoke_emitters` (through `stl_insert_n_shift_stride_0xc`) | `vehicle_game_init` (count), `vehicle_apply_carinit_params`, `vehicle_update_strips` | one element per `STRIPL*`, `STRIPML*`, `STRIPFL*`, `STRIPBL*` frame: `+0x00` frame, `+0x04` rest length (scale z = span / it), `+0x08` ushort wheel index (axle * 2), `+0x0a` ushort kind (bit `0x1` processed; `0x2` / `0x4` one end at the frame's own position; else between wheel `i` and `i + 2`) |
| `+0x168` | `strips_right` | H | `vehicle_ctor`, `vehicle_init_strip_and_smoke_emitters` (through `stl_insert_n_shift_stride_0xc`) | `vehicle_game_init` (count), `vehicle_apply_carinit_params`, `vehicle_update_strips` | as `+0x158`, from `STRIPR*`, `STRIPMR*`, `STRIPFR*`, `STRIPBR*` (wheel index axle * 2 + 1); a wheel with a kind-`0x1` element gets wheel bit `0x200` |
| `+0x178` | `net_damage_dirty` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_deform` (`0x1`, `0x2`), `vehicle_apply_damage` (`0x4`, `0x8`), `vehicle_update_visuals` (clears `0x2`, `0x4`, `0x8`), `car_hit` (`0x2`, `0x4`, `0x8`), `car_on_collision_apply_damage` (clears `0x1`) (both as car+0x1e8) | `vehicle_apply_damage`, `vehicle_update_visuals`, `vehicle_save_state`, `car_on_collision_apply_damage`, `car_post_dirty_net_event_msgs` (as car+0x1e8) | `0x1` a zone mesh moved (in session state 7 `car_on_collision_apply_damage` calls `car_queue_collision_damage_record`), `0x2` zone or wheel damage applied, `0x4` engine pool drained, `0x8` gearbox pool drained; `0x2` / `0x4` / `0x8` post the messages `0x28` / `0x29` / `0x2a`; each drain runs only while its bit is clear |
| `+0x17c` | `lights` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_init_lights` | `vehicle_add_projector_lights_to_sectors`, `vehicle_apply_light_state`, `vehicle_deactivate`, `vehicle_deform`, `vehicle_dtor`, `vehicle_get_save_state_size`, `vehicle_init_damage`, `vehicle_init_lights`, `vehicle_load_state`, `vehicle_remove_projector_lights_from_sectors`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_save_state`, `vehicle_set_projector_size`, `vehicle_light_at` (the vector as this) | - |
| `+0x18c` | `special_projectors` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_init_lights` | `vehicle_add_projector_lights_to_sectors`, `vehicle_apply_light_state`, `vehicle_deactivate`, `vehicle_dtor`, `vehicle_init_damage`, `vehicle_remove_projector_lights_from_sectors`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_switch_special_projector_texture`, `vehicle_special_projector_at` (the vector as this) | - |
| `+0x19c` | `light_state` | H | `vehicle_apply_light_state`, `vehicle_game_done`, `vehicle_game_init`, `vehicle_load_state`, `vehicle_set_auto_indicators_from_steering`, `vehicle_set_emergency_lights`, `vehicle_set_headlights`, `vehicle_set_left_indicator`, `vehicle_set_light_script_override`, `vehicle_set_right_indicator`, `car_update`, `car_deactivate`, `car_unpack_light_state` (as car+0x20c) | `vehicle_apply_light_state`, `vehicle_get_light_script_override`, `vehicle_save_state`, `vehicle_set_auto_indicators_from_steering`, `vehicle_set_emergency_lights`, `vehicle_set_headlights`, `vehicle_set_left_indicator`, `vehicle_set_light_script_override`, `vehicle_set_right_indicator`, `vehicle_update_visuals`, `car_update`, `car_pack_light_state` (as car+0x20c) | - |
| `+0x1a0` | `light_state_prev` | H | `vehicle_apply_light_state`, `vehicle_game_init`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | `vehicle_apply_light_state` copies `+0x19c` into it on exit and tests its bit `0x80` |
| `+0x1a4` | `blink_lit_s` | H | `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | §5 |
| `+0x1a8` | `blink_dark_s` | H | `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | §5 |
| `+0x1ac` | `blink_timer_s` | H | `vehicle_apply_light_state`, `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | §5 |
| `+0x1b0` | `auto_indicator_steer_min` | H | `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | compared with `+0x6a4` while bit `0x8` of `+0x19c` is set |
| `+0x1b4` | `light_bar_period_s` | H | `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | §5 |
| `+0x1b8` | `light_bar_timer_s` | H | `vehicle_apply_light_state`, `vehicle_ctor`, `vehicle_load_state` | `vehicle_apply_light_state`, `vehicle_save_state` | §5 |
| `+0x1bc` | `own_body_friction` | H | `vehicle_ctor` (0), `vehicle_apply_carinit_params` (CARINIT byte `+0x5e4`) | `vehicle_body_coll_resolve` (2) | `+0x408` / `+0x40c` replace the struck material's `+0x18` / `+0x1c` |
| `+0x1bd` | `own_bump_ranges` | H | `vehicle_ctor` (0), `vehicle_apply_carinit_params` (CARINIT byte `+0x620`) | `vehicle_update_visuals` | wheel `+0x1a4`..`+0x1b4` instead of the surface material's `+0x04`..`+0x14` |
| `+0x1c0` | `unread_1c0` | U | `vehicle_game_init`, `vehicle_load_state`, `vehicle_update_visuals` | `vehicle_save_state` | every other `+0x1c0` load in the class is a wheel's bump height |
| `+0x1c4` | `side_force_sum` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_move` | `vehicle_move`, `vehicle_save_state`, `driver_fill_start_values` (as car+0x234) | `vehicle_move` zeroes it and adds `wheel+0xa4` per wheel |
| `+0x1c8` | `forward_force_sum` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_move` | `vehicle_move`, `vehicle_save_state`, `driver_fill_start_values` (as car+0x238) | `vehicle_move` adds `min(wheel+0xa0, wheel+0x118)` per wheel |
| `+0x1cc` | `forward_force_cap` | H | `vehicle_apply_carinit_params` | `car_ai_tick` (3), `car_is_possible_collision`, `driver_fill_start_values` (as car+0x23c) | `car_ai_tick` brakes by `min(+0x1c8, +0x1cc) * 0.7 / +0x1cc * 0.4`; `car_is_possible_collision` returns at once below 0.0001 |
| `+0x1d0` | `unread_1d0` | U | `vehicle_game_init`, `vehicle_load_state` | `vehicle_save_state` | Open item 3 |
| `+0x1d4` | `handbrake_sound_latch` | H | `vehicle_game_init`, `vehicle_play_handbrake_sound`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_tick_drivetrain` | `vehicle_play_handbrake_sound` | - |
| `+0x1d8` | `size_world[3]` | H | `vehicle_init` | `vehicle_get_size` | `vehicle_get_size(0)` |
| `+0x1e4` | `size_model[3]` | H | `vehicle_init` | `vehicle_decode_position`, `vehicle_get_size` | `vehicle_get_size(1)`; `vehicle_encode_position` / `vehicle_decode_position` |
| `+0x1f0` | `pivot_world[3]` | H | `vehicle_init` | `vehicle_get_pivot_pos` | `vehicle_get_pivot_pos(0)`; `vehicle_init` writes the root frame's scale times `+0x1fc` |
| `+0x1fc` | `pivot_model[3]` | H | `vehicle_init` | `vehicle_decode_position`, `vehicle_get_pivot_pos` | `vehicle_get_pivot_pos(1)`; decode = `(q / max) * +0x1e4 - +0x1fc` per axis = the bound-box minimum plus a fraction of the size |
| `+0x208` | `skid_spin_speed_min` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1ea]` km/h to m/s) | `vehicle_update_skid_sound` | a spinning driven wheel adds `(wheel surface speed - (speed + +0x208)) / +0x20c` to the skid level |
| `+0x20c` | `skid_spin_speed_range` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1eb]` km/h to m/s, at least `[0x1ea]`) | `vehicle_update_skid_sound` | as `+0x208` |
| `+0x210` | `skid_volume_rate` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1ec]`) | `vehicle_update_skid_sound` | - |
| `+0x214` | `skid_volume` | H | `vehicle_ctor`, `vehicle_reset`, `vehicle_game_init` (0), `vehicle_update_skid_sound` | `vehicle_update_skid_sound` | follows the skid level up at `+0x210` per second and drops to it at once; the skid handle plays from 0.01 |
| `+0x218` | `steer_linearity` | H | `vehicle_ctor`, `vehicle_load_state`, `vehicle_set_steering_linearity` | `vehicle_save_state`, `vehicle_set_steer` | - |
| `+0x21c` | `clutch_linearity` | H | `vehicle_ctor`, `vehicle_load_state` | `vehicle_save_state`, `vehicle_set_clutch` | - |
| `+0x220` | `unread_220` | U | `vehicle_load_state` | `vehicle_save_state` | Open item 3 |
| `+0x224` | `master_flags` | H | `vehicle_compute_motor_force`, `vehicle_game_init`, `vehicle_load_state`, `vehicle_lock`, `vehicle_move`, `vehicle_reset`, `vehicle_set_brake`, `vehicle_set_power`, `vehicle_settle`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | `vehicle_compute_motor_force`, `vehicle_is_unstable`, `vehicle_lock`, `vehicle_move`, `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_power`, `vehicle_settle`, `vehicle_tick_drivetrain`, `vehicle_update_visuals`, `vehicle_wheel_contact_solve`, `human_find_blocking_drawbridge` (as car+0x294) | - |
| `+0x228` | `asleep_ms` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` (0 on a contact latch) | `vehicle_tick_drivetrain`, `vehicle_save_state` | zeroed by `+0xcdc`; past 1000 with no driver it sets `master_flags` `0x2`; at the end of a tick with bit `0x2`, past 1000 (or `+0x04` past 4000) bit `0x1` is cleared |
| `+0x22c` | `engine_damage_floor` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x230` | `motor_force_factor` | H | `vehicle_apply_damage`, `vehicle_init_damage`, `vehicle_load_state`, `car_hit` (as car+0x2a0) | `vehicle_compute_motor_force`, `vehicle_save_state` | `vehicle_compute_motor_force` multiplies by it |
| `+0x234` | `engine_damage` | H | `vehicle_apply_damage`, `vehicle_init_damage`, `vehicle_load_state`, `car_hit` (as car+0x2a4) | `vehicle_apply_damage`, `vehicle_save_state`, `vehicle_set_engine_on`, `vehicle_update_motor_particles` | `vehicle_set_engine_on` refuses a start at 0 or below while `+0xce4` bit `0x4000` is set |
| `+0x238` | `engine_damage_max` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage`, `vehicle_init_damage` | - |
| `+0x23c` | `engine_damage_resist` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x240` | `engine_damage_divisor` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x244` | `engine_damage_threshold` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x248` | `smoke_damage_light` | H | `vehicle_apply_carinit_params` (CARINIT `[0x193]`) | `vehicle_update_motor_particles` | gated by `feature_flags` `0x200`; smoke while `engine_damage` < `+0x248` and `speed` < `+0x250`, emitted only while `+0x360` is set |
| `+0x24c` | `smoke_damage_heavy` | H | `vehicle_apply_carinit_params` (CARINIT `[0x194]`) | `vehicle_update_motor_particles` | switches to particle `+0x258` |
| `+0x250` | `smoke_speed_max` | H | `vehicle_apply_carinit_params` (CARINIT `[0x195]` km/h to m/s) | `vehicle_update_motor_particles` | - |
| `+0x254` | `smoke_particle_light` | H | `vehicle_apply_carinit_params` (CARINIT `[0x196]`) | `vehicle_update_motor_particles` | - |
| `+0x258` | `smoke_particle_heavy` | H | `vehicle_apply_carinit_params` (CARINIT `[0x197]`) | `vehicle_update_motor_particles` | - |
| `+0x25c` | `smoke_emit_ms` | H | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init` (0), `vehicle_update_motor_particles` | `vehicle_update_motor_particles` | sums frame ms up to the particle's period, then 0 |
| `+0x260` | `engine_point[3]` | H | `vehicle_init_strip_and_smoke_emitters` | `vehicle_update_motor_particles`, `vehicle_sound_slot_init` | zero when the frame is absent, which is then unlinked; the motor smoke and every sound opened with the slot's last flag set (the engine group) sit there |
| `+0x26c` | `gearbox_damage` | H | `vehicle_apply_damage`, `vehicle_init_damage`, `vehicle_load_state` | `vehicle_apply_damage`, `vehicle_save_state`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | under damage an empty pool refuses every gear change |
| `+0x270` | `gearbox_damage_max` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage`, `vehicle_init_damage` | - |
| `+0x274` | `gearbox_damage_resist` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x278` | `gearbox_damage_divisor` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x27c` | `gearbox_damage_threshold` | H | `vehicle_apply_carinit_params` | `vehicle_apply_damage` | - |
| `+0x280` | `gear_sound_variant` | H | `vehicle_ctor`, `vehicle_game_init` (0), `vehicle_play_gear_change_sound` | `vehicle_play_gear_change_sound` | `rand() % 100 > 79` picks 0 or 1 |
| `+0x284` | `deform_zones_pristine` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_prepare_deform_zone` | `vehicle_apply_carinit_params`, `vehicle_deform`, `vehicle_dtor`, `vehicle_get_save_state_size`, `vehicle_init_deform`, `vehicle_load_state`, `vehicle_prepare_deform_zone`, `vehicle_save_state` | the copy `vehicle_init_deform` restores the meshes from (§7) |
| `+0x294` | `deform_zones` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_cb_enum_deform_zones` (from `vehicle_init`'s frame walk) | `vehicle_animate_windows`, `vehicle_apply_carinit_params`, `vehicle_deform`, `vehicle_deform_glass`, `vehicle_dtor`, `vehicle_encode_part_position`, `vehicle_find_part_at_encoded_position`, `vehicle_find_part_by_name_tag`, `vehicle_get_part_name_tag`, `vehicle_get_save_state_size`, `vehicle_init_deform`, `vehicle_init_deform_glass_texture`, `vehicle_init_lights`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_is_window_open`, `vehicle_load_state`, `vehicle_prepare_deform_zone`, `vehicle_save_state`, `vehicle_update_visuals` | - |
| `+0x2a4` | `traction_cap_gear` | H | `vehicle_apply_carinit_params` (CARINIT int `+0x5f0`), `vehicle_load_state` | `vehicle_move`, `vehicle_save_state` | a driven wheel's drive force is capped to the grip its side force leaves (`sqrt((a² s² - a² f²) / s²)`); the shipped 20 is above every gear |
| `+0x2a8` | `brake_grip_cap` | H | `vehicle_apply_carinit_params` (CARINIT byte `+0x5f4`), `vehicle_load_state`, `vehicle_move` | `vehicle_move`, `vehicle_save_state` | also forced for the step while the speed is within 2.8 above `+0x520` (the brake then 1.0); the cap is the grip `+0xa0` minus 0.7 * the wheel's other force; restored at the end of the step |
| `+0x2a9` | `yaw_stability_brake` | H | `vehicle_apply_carinit_params` (CARINIT byte `+0x5f5`) | `vehicle_move` (`0x00530026`) | with the body-space yaw rate above 0 and the second-last wheel's slip `+0x108` above its limit `+0xbc` + 1 degree, wheel 1 gets the whole brake and loses its other force; at or below 0 the same for wheel 0 against the last wheel. Not in the save block |
| `+0x2ac` | `dyn_coll_record` | H | `vehicle_ctor` (zero), `vehicle_game_init` (fills it), `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals` (the centre `+0x2bc`) | `vehicle_ctor`, `vehicle_deactivate`, `vehicle_game_done`, `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_dynamic_colls_off`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals`, `vehicle_body_coll_resolve` (2) | passed by address to `collision_dyn_insert` / `_update`; `vehicle_update_visuals` moves its centre to `+0x39c` while `master_flags` `0x1` unless `+0x118`; `vehicle_body_coll_resolve` passes it to `collision_dyn_set_next_owner` around each sphere and box test, so the car's own record is skipped (§7) |
| `+0x2f4` | `dyn_coll_record_2` | H | `vehicle_ctor` (zero), `vehicle_game_init` (fills it), `vehicle_reset_from_frame`, `vehicle_reset` (centre), `vehicle_update_visuals` (centre `+0x304`, every frame) | `vehicle_deactivate`, `vehicle_game_done` (both erase it unconditionally), `vehicle_update_visuals` | inserted on every reset, whatever `+0x1f26` says |
| `+0x33c` | `skid_mark_pairs` | H | `vehicle_ctor` (50) | `vehicle_game_init` | `shlaif01.i3d` / `shlaif02.i3d` pairs created into each wheel's ring `+0x12c` |
| `+0x340` | `has_driver` | H | `vehicle_game_init`, `vehicle_load_state`, `car_set_has_driver`, `car_set_owner` (as car+0x3b0) | `vehicle_save_state`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | `car_set_owner` clears it when the driver seat is vacated; all 7 callers of `car_set_has_driver` pass 1 |
| `+0x344` | `unread_344[4]` | U | `vehicle_apply_carinit_params` | - | shipped `+0x607`..`+0x613`: 0, 3, -4, 2 (`def-files-docs/vehicles-bin.md` §4c); Open item 3 |
| `+0x354` | `head_pos[3]` | H | `vehicle_init`, `vehicle_init` | `vehicle_get_head_pos`, `vehicle_init`, `camera_tick`, `camera_begin_mode` (through `vehicle_get_head_pos`) | - |
| `+0x360` | `in_camera_sector` | H | `vehicle_ctor` (1), `vehicle_load_state`, `car_update` (as car+0x3d0, every frame) | `vehicle_update_visuals` (2), `vehicle_update_motor_particles`, `vehicle_save_state` | set: steering wheel, wheel and exhaust particles, axes, strips, wheel rotation, the `+0x130` refresh and the motor smoke; clear: `vehicle_update_axis` only when `+0xce4` differs from `+0xce8` (`structs/actor.md` §2) |
| `+0x364` | `callbacks[14]` | H | `vehicle_ctor`, `vehicle_set_callback_actor_filter`, `vehicle_set_callback_body_collision_filter`, `vehicle_set_callback_break_glass`, `vehicle_set_callback_collision`, `vehicle_set_callback_collision_info`, `vehicle_set_callback_collision_quiet`, `vehicle_set_callback_deform_part`, `vehicle_set_callback_prepare_dropout`, `vehicle_set_callback_prepare_wheel_dropout`, `vehicle_set_callback_repair`, `vehicle_set_callback_wake_up`, `vehicle_set_callback_was_collision`, `vehicle_set_callback_wheel_collision_filter`, the 13 setters of §3b | `vehicle_apply_external_force`, `vehicle_apply_external_impulse`, `vehicle_deform`, `vehicle_deform_glass`, `vehicle_do_collision`, `vehicle_init`, `vehicle_init_deform`, `vehicle_open_door`, `vehicle_set_angular_velocity`, `vehicle_set_emergency_lights`, `vehicle_set_headlights`, `vehicle_set_left_indicator`, `vehicle_set_light_script_override`, `vehicle_set_right_indicator`, `vehicle_set_velocity`, `vehicle_settle`, `vehicle_wheel_contact_solve` | 13 slots have a setter; `+0x384` has none (§10b) |
| `+0x39c` | `root_pos[3]` | H | `vehicle_init_strip_and_smoke_emitters` (its local position), `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init`, `vehicle_update_cached_transform` | `vehicle_set_special_dynamic_colls`, `vehicle_tick_drivetrain`, `vehicle_update_visuals`, `vehicle_save_state` | the centre of `dyn_coll_record` and of the `+0x1f38` records; the rest anchor `+0x08` |
| `+0x3a8` | `axle_frames` | H | `vehicle_init_axes` (through `stl_insert_n_stride_0x18`), `vehicle_dtor` (frees it) | `vehicle_update_axis`, `vehicle_dtor` | one record per `AXIS_<n>0` frame: `+0x00` the frame, placed and stretched (scale z = distance) between the two wheels of axle `+0x04`, offset `+0x08`..`+0x10` (the frame name * 0.01); a record with `+0x15` set also aims both wheels' steered frames |
| `+0x3b8` | `axle_side_frames` | H | `vehicle_init_axes`, `vehicle_dtor` (frees it) | `vehicle_init_axes`, `vehicle_update_axis`, `vehicle_dtor` | one record per `AXIS_<n>L` / `AXIS_<n>R` frame (x negated on R): stretched toward wheel `axle * 2` when the left byte `+0x14` is set, else `axle * 2 + 1`; `+0x15` = the only record of its side on that axle, which also aims that wheel's steered frame |
| `+0x3c8` | `root_frame` | H | `vehicle_ctor`, `vehicle_init` | `vehicle_dtor`, `vehicle_encode_part_position`, `vehicle_find_part_at_encoded_position`, `vehicle_game_init`, `vehicle_init`, `vehicle_init_axes`, `vehicle_init_lights`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_save_state`, `vehicle_sound_slot_init`, `vehicle_update_cached_transform`, `vehicle_update_visuals` | `vehicle_encode_part_position` walks a part's chain up to it |
| `+0x3cc` | `turn_radius` | M | `vehicle_ctor`, `vehicle_game_init` (0), `vehicle_apply_carinit_params` (1e8 when neither end wheel steers), `vehicle_load_state`, `vehicle_substep_move` | `vehicle_substep_move`, `vehicle_save_state` | the frame distance `+0x630` / the heading change, signed by the steering; at low speed with steering and a near-zero radius it takes `+0x3d0`. No reader outside the class: the car `+0x43c` hits are `C_human`'s `+0x43c` (Open item 3) |
| `+0x3d0` | `turn_radius_min` | H | `vehicle_apply_carinit_params`, `vehicle_load_state`, `vehicle_substep_move` | `vehicle_substep_move`, `vehicle_save_state` | 1.0, or with three wheels or more `wheelbase / tan(steer_lock[0])` (halved when both end wheels steer) * 1.1; `vehicle_substep_move` lowers it while no wheel has bits `0x90` |
| `+0x3d4` | `velocity_dir[3]` | H | `vehicle_reset`, `vehicle_game_init`, `vehicle_apply_external_impulse`, `vehicle_do_collision`, `vehicle_update_cached_transform`, `vehicle_load_state`, `vehicle_move` | `vehicle_move`, `vehicle_update_visuals` | at zero length one axis is set to +1 or -1; the drag acts along minus it, and the odometer `+0x1f20` uses it |
| `+0x3e0` | `centre_box_min[3]` | H | `vehicle_init`, `vehicle_game_init` (lowers `+0x3e4` by 0.5 and restores it) | `vehicle_game_init`, `vehicle_apply_carinit_params` | the box of both collision records; it gives `+0x45c`..`+0x470` and `+0x480`..`+0x488` |
| `+0x3ec` | `centre_box_max[3]` | H | `vehicle_init` | `vehicle_game_init`, `vehicle_apply_carinit_params` | as `+0x3e0` |
| `+0x3f8` | `steering_wheel_axis[3]` | H | `vehicle_init_strip_and_smoke_emitters` | `vehicle_update_visuals` | its rotation's third row; `steering_wheel_frame` turns about it |
| `+0x404` | `top_speed` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_update_steering_wheels` | - |
| `+0x408` | `own_friction` | H | `vehicle_apply_carinit_params` (CARINIT `[0x17a]`, at least 0) | `vehicle_body_coll_resolve` | `def-files-docs/vehicles-bin.md` §4c |
| `+0x40c` | `own_impulse_factor` | H | `vehicle_apply_carinit_params` (CARINIT `[0x17b]`, at least 1) | `vehicle_body_coll_resolve` | as `+0x408` |
| `+0x410` | `driven_wheel_down` | H | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init` (0), `vehicle_move` (0 per step, 1 per driven wheel down) | `vehicle_tick_drivetrain`, `vehicle_update_visuals`, `vehicle_move`, `vehicle_body_coll_resolve` | `vehicle_body_coll_resolve` passes it to `vehicle_play_crash_sound`: the tier mute bytes act only while it is set |
| `+0x414` | `centre_of_mass[3]` | H | `vehicle_game_init`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_update_cached_transform` | `vehicle_apply_external_force`, `vehicle_apply_external_impulse`, `vehicle_deform`, `vehicle_do_collision`, `vehicle_emit_body_collision_particles`, `vehicle_get_contact_impulse_terms`, `vehicle_move`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_particles`, `vehicle_update_visuals`, `vehicle_wheel_contact_solve` | - |
| `+0x420` | `throttle` | H | `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_brake`, `vehicle_set_engine_on`, `vehicle_set_power`, `vehicle_tick_drivetrain` | `vehicle_move`, `vehicle_tick_drivetrain` | zeroed by `vehicle_set_engine_on` when the engine stops |
| `+0x424` | `throttle_reverse` | H | `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_brake`, `vehicle_set_engine_on`, `vehicle_set_power`, `vehicle_tick_drivetrain` | `vehicle_move`, `vehicle_tick_drivetrain` | - |
| `+0x428` | `gear_speed_factor` | H | `vehicle_apply_carinit_params` (CARINIT `[2]`) | `vehicle_apply_carinit_params` | feeds `+0x5ac` |
| `+0x42c` | `handbrake_force` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_handbrake` | - |
| `+0x430` | `handbrake_ramp_max` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_handbrake` | - |
| `+0x434` | `handbrake_ramp_accum` | H | `vehicle_game_init`, `vehicle_reset`, `vehicle_set_handbrake` | `vehicle_set_handbrake` | - |
| `+0x438` | `throttle_out_prev` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain`, `vehicle_save_state` | below 0.3 it is one condition of the automatic downshift |
| `+0x43c` | `throttle_reverse_out_prev` | M | `vehicle_game_init`, `vehicle_load_state`, `vehicle_tick_drivetrain` | `vehicle_save_state` | saved and restored; no other reader in either spelling (`+0x43c`, car+0x4ac) |
| `+0x440` | `wheel_force_base` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params` | each wheel's `+0x118` is a percent of it |
| `+0x444` | `pedal_ramp_max` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_brake`, `vehicle_set_power` | - |
| `+0x448` | `pedal_ramp_accum` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_brake`, `vehicle_set_power`, `vehicle_substep_move` | `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_power` | - |
| `+0x44c` | `downforce_factors[4]` | H | `vehicle_apply_carinit_params` | `vehicle_substep_move` | a value in 0.029..1.331, then one at least 0, per pair; each product times v² * 0.65 is a downforce term passed to `vehicle_move` |
| `+0x45c` | `downforce_point_front[3]` | H | `vehicle_apply_carinit_params` (`+0x480`, `+0x484`, box max z `+0x3f4`) | `vehicle_move` | with `feature_flags` `0x8` each downforce term acts along minus up at one of the two points |
| `+0x468` | `downforce_point_rear[3]` | H | `vehicle_apply_carinit_params` (`+0x480`, `+0x484`, box min z `+0x3e8`) | `vehicle_move` | as `+0x45c` |
| `+0x474` | `centre_of_mass_model[3]` | H | `vehicle_apply_carinit_params` (= `+0x480`), `vehicle_set_seat_mass`, `vehicle_load_state` | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init`, `vehicle_update_cached_transform`, `vehicle_save_state` | the world matrix times it is `centre_of_mass` `+0x414`; `vehicle_set_seat_mass` moves it toward the seats' and ballast points' mass centre by `m / (m + +0x674)` |
| `+0x480` | `centre_of_mass_base[3]` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_seat_mass` | - |
| `+0x48c` | `weight_force[3]` | H | `vehicle_apply_carinit_params` | `vehicle_move` | added to the step's force sum before `velocity += dt / +0x670 * F`; also transformed in the below-3 m/s hold |
| `+0x498` | `engine_inertia` | H | `vehicle_apply_carinit_params` | `vehicle_update_motor_revs` | `revs += torque * dt / (+0x498 * 2 pi)` |
| `+0x49c` | `angular_velocity[3]` | H | `vehicle_apply_external_impulse`, `vehicle_do_collision`, `vehicle_game_init`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_set_angular_velocity`, `vehicle_settle`, `vehicle_deactivate`, `vehicle_reset_from_frame` | `vehicle_apply_external_impulse`, `vehicle_deform`, `vehicle_do_collision`, `vehicle_emit_body_collision_particles`, `vehicle_get_contact_impulse_terms`, `vehicle_move`, `vehicle_save_state`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_particles`, `vehicle_wheel_contact_solve` | `vehicle_deactivate` and `vehicle_reset_from_frame` scale it in place (`operator*=` on its address) |
| `+0x4a8` | `driven_wheel_count` | H | `vehicle_apply_carinit_params` | `vehicle_move` | in gear state 0 without strips the drive force is divided by `+0x4a8 / 2` |
| `+0x4ac` | `horn_on` | H | `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_horn`, `driver_do_sounds` (as car+0x51c) | `vehicle_update_visuals` | while `+0x4ae` is set, `vehicle_update_visuals` passes it to `vehicle_play_horn_sound` every frame (`0x0052d51d`) |
| `+0x4ad` | `siren_on` | H | `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `police_board_create_and_engage` (1), `police_action_tick`, `police_action_dispatch_officer_commands`, `police_action_disband` (0; all as car+0x51d) | `vehicle_get_siren`, `vehicle_update_visuals`, `car_update` (as car+0x51d) | while `+0x4ae` is set, `vehicle_update_visuals` passes it to `vehicle_play_siren_sound` every frame (`0x0052d52b`); while it is set `car_update` keeps the car's sounds on out to distance 110 instead of 60 |
| `+0x4ae` | `sounds_enabled` | H | `vehicle_ctor`, `vehicle_enable_sounds`, `vehicle_load_state`, `vehicle_open_door` | `vehicle_ctor`, `vehicle_enable_sounds`, `vehicle_open_door`, `vehicle_play_door_sound`, `vehicle_play_tyre_burst_sound`, `vehicle_save_state`, `vehicle_set_engine_on`, `vehicle_tick_drivetrain`, `vehicle_update_engine_sound`, `vehicle_update_motor_revs`, `vehicle_update_visuals`, `vehicle_wheel_contact_solve` | `vehicle_enable_sounds(0)` stops every sound |
| `+0x4b0` | `handbrake_out` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_handbrake` | `vehicle_move`, `vehicle_save_state`, `vehicle_tick_drivetrain` | applied to the handbrake wheels only (§6) |
| `+0x4b4` | `inertia_inv[3]` | H | `vehicle_apply_carinit_params` (= `+0x4c0`), `vehicle_set_seat_mass` (`1 / (1 / base + r^2 m)`), `vehicle_load_state` | `vehicle_apply_external_impulse`, `vehicle_do_collision`, `vehicle_get_contact_impulse_terms`, `vehicle_move`, `vehicle_save_state` | scales the inverse body matrix before a torque turns into angular velocity |
| `+0x4c0` | `inertia_inv_base[3]` | H | `vehicle_apply_carinit_params` | `vehicle_set_seat_mass` | - |
| `+0x4cc` | `exhausts` | H | `vehicle_init` (through `std_vector18_insert_at`), `vehicle_dtor` (frees it) | `vehicle_update_exhaust_particles`, `vehicle_dtor` | one record per `EXHAUST*` frame: a point `+0x00` and a direction `+0x0c`, both transformed |
| `+0x4dc` | `exhaust_particle` | H | `vehicle_apply_carinit_params` (CARINIT int `[0x180]`) | `vehicle_update_exhaust_particles` | emitted with the engine on, `feature_flags` `0x200` and `speed` below `+0x4e4` |
| `+0x4e0` | `exhaust_emit_ms` | H | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init` (0), `vehicle_update_exhaust_particles` | `vehicle_update_exhaust_particles` | - |
| `+0x4e4` | `exhaust_speed_max` | H | `vehicle_apply_carinit_params` (CARINIT `[0x181]` km/h to m/s) | `vehicle_update_exhaust_particles` | - |
| `+0x4e8` | `exhaust_exit_speed_min` | H | `vehicle_apply_carinit_params` (CARINIT `[0x182]`) | `vehicle_update_exhaust_particles` | exit speed `+0x4e8 + (+0x4ec - +0x4e8) * revs / max revs` |
| `+0x4ec` | `exhaust_exit_speed_max` | H | `vehicle_apply_carinit_params` (CARINIT `[0x183]`, at least `[0x182]`) | `vehicle_update_exhaust_particles` | as `+0x4e8` |
| `+0x4f0` | `torque_idle` | H | `vehicle_apply_carinit_params` (CARINIT `[7]`) | `vehicle_compute_moment`, `vehicle_apply_carinit_params` | the curve runs `+0x4f0` to `+0x4f4` from idle to `+0x500`, flat to `+0x504`, to `+0x4f8` at `+0x508`, to `+0x4fc` at max revs |
| `+0x4f4` | `torque_peak` | H | `vehicle_apply_carinit_params` (CARINIT `[6]`) | `vehicle_compute_moment`, `vehicle_apply_carinit_params` | also scales `+0x6bc` |
| `+0x4f8` | `torque_redline` | H | `vehicle_apply_carinit_params` (`+0x5ac / (redline rps * 2 pi)`) | `vehicle_compute_moment` | - |
| `+0x4fc` | `torque_max_revs` | H | `vehicle_apply_carinit_params` (CARINIT `[8]`) | `vehicle_compute_moment` | - |
| `+0x500` | `torque_peak_from_revs` | H | `vehicle_apply_carinit_params` (CARINIT `[9]` rpm / 60) | `vehicle_compute_moment`, `vehicle_tick_drivetrain` | `idle <= [9] <= [10] < redline`; with `+0x504` it also bounds the launch clutch |
| `+0x504` | `torque_peak_to_revs` | H | `vehicle_apply_carinit_params` (CARINIT `[10]` rpm / 60) | `vehicle_compute_moment`, `vehicle_tick_drivetrain` | as `+0x500` |
| `+0x508` | `torque_redline_revs` | H | `vehicle_apply_carinit_params` (CARINIT `[3]` rpm / 60) | `vehicle_compute_moment` | equals `redline_revs` `+0x5e0` |
| `+0x50c` | `camera_inside_request` | H | `vehicle_ctor` (0), `vehicle_load_state`, `human_intern_use_car`, `human_update`, `human_put_in_vehicle_at_seat`, `player_ai` (as car+0x57c) | `vehicle_update_engine_sound` (3), `vehicle_update_engine_pitch` (2), `vehicle_save_state` | each writer copies `C_game +0x6e`, the camera's `camera_inside` (`structs/camera.md` §2), when the player enters a car or presses the camera-change key; both sound functions copy it into `+0x50d` (§8e) |
| `+0x50d` | `camera_inside` | H | `vehicle_ctor`, `vehicle_load_state`, `vehicle_update_engine_pitch`, `vehicle_update_engine_sound` | `vehicle_ctor`, `vehicle_deform_glass`, `vehicle_play_crash_sound`, `vehicle_play_door_sound`, `vehicle_play_engine_start_fail_sound`, `vehicle_play_gear_change_sound`, `vehicle_play_handbrake_sound`, `vehicle_play_horn_sound`, `vehicle_play_siren_sound`, `vehicle_play_tyre_burst_sound`, `vehicle_play_wheel_sound`, `vehicle_save_state`, `vehicle_update_engine_pitch`, `vehicle_update_environment_sound`, `vehicle_update_flat_tyre_sound`, `vehicle_update_skid_sound` | §8e; it follows `+0x50c`: `vehicle_update_engine_sound` copies it on a completed start, `vehicle_update_engine_pitch` on a change, after silencing the old set |
| `+0x510` | `standstill_shift_timer` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_gear`, `vehicle_tick_drivetrain` | `vehicle_save_state`, `vehicle_tick_drivetrain` | accumulates frame time while the car stands at idle revs with a pedal above 0.3; past 0.002 it switches between first and reverse |
| `+0x514` | `fuel_rate` | H | `vehicle_apply_carinit_params` (`[0x47] / (100000 / (max rps * 0.4167 / top-gear ratio * +0x628)) * 1.7`) | `vehicle_tick_drivetrain` | `fuel -= (0.8 * abs(throttle) + 0.2 * revs / max revs) * +0x514 * dt`, the two throttles averaged in gear state 1 |
| `+0x518` | `weight_gravity` | H | `vehicle_apply_carinit_params` (CARINIT `[0x45]`) | `vehicle_tick_drivetrain` | with `+0x670` and the sine of the forward axis' slope, the force the next gear's torque must beat for an upshift |
| `+0x51c` | `engine_load` | H | `vehicle_reset`, `vehicle_game_init` (0), `vehicle_load_state`, `vehicle_compute_motor_force` | `vehicle_compute_motor_force`, `vehicle_update_engine_pitch` (6), `vehicle_save_state` | moves toward the torque's position between `+0x6b8` and `+0x6bc`, up at `+0x6c0` per second, down at `+0x6c4`; the on-load and reverse banks play at `p7 + (p6 - p7) * +0x51c`, the coast bank at `p7 + (p6 - p7) * (1 - +0x51c)` |
| `+0x520` | `speed_limit` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_set_speed_limit` | `vehicle_compute_motor_force`, `vehicle_move`, `vehicle_save_state` | - |
| `+0x524` | `steer_analog` | H | `vehicle_game_init`, `vehicle_load_state`, `player_ai_drive`, `driver_tick`, `traffic_car_activate_elements`, `car_net_snapshot_state` (as car+0x594) | `vehicle_save_state`, `vehicle_set_steer`, `vehicle_update_steering_wheels`, `car_update` (as car+0x594) | - |
| `+0x525` | `accel_analog` | H | `vehicle_game_init`, `vehicle_load_state`, `player_ai_drive`, `driver_tick`, `traffic_car_activate_elements`, `car_net_snapshot_state` (as car+0x595) | `vehicle_save_state` | no reader located |
| `+0x526` | `brake_analog` | H | `vehicle_game_init`, `vehicle_load_state`, `player_ai_drive`, `driver_tick`, `traffic_car_activate_elements`, `car_net_snapshot_state` (as car+0x596) | `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_power` | - |
| `+0x527` | `clutch_analog` | H | `vehicle_game_init`, `vehicle_load_state`, `player_ai_drive`, `driver_tick`, `traffic_car_activate_elements`, `car_net_snapshot_state` (as car+0x597) | `vehicle_save_state`, `vehicle_set_clutch`, `vehicle_tick_drivetrain` | - |
| `+0x528` | `body_coll_heads` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_cb_enum_body_colls` | `vehicle_dtor`, `vehicle_game_init`, `vehicle_init`, `vehicle_move` (by address) | - |
| `+0x538` | `root_name` | H | `vehicle_init` (frees the old one) | `vehicle_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_init_axes`, `vehicle_dtor` (frees it) | the prefix every frame-name test skips |
| `+0x53c` | `axle_count` | H | `vehicle_init` | `vehicle_init_strip_and_smoke_emitters`, `vehicle_init_axes`, `vehicle_apply_carinit_params` | bounds every axle loop (`def-files-docs/vehicles-bin.md` §4b) |
| `+0x540` | `wheel_count` | H | `vehicle_init` | `vehicle_apply_carinit_params`, `vehicle_deactivate`, `vehicle_deform`, `vehicle_game_done`, `vehicle_game_init`, `vehicle_get_save_state_size`, `vehicle_get_wheel_count`, `vehicle_init`, `vehicle_init_damage`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_save_state`, `vehicle_substep_move`, `vehicle_tick_drivetrain`, `vehicle_update_rotation_wheels`, `vehicle_update_skid_sound`, `vehicle_update_steering_wheels`, `vehicle_update_visuals`, `vehicle_update_wheel_particles`, `vehicle_wheel_contact_solve` | - |
| `+0x544` | `ramp_rate` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_substep_move` | `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_handbrake`, `vehicle_set_power` | - |
| `+0x548` | `automatic_gearbox` | H | `vehicle_game_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_set_automatic_gearbox`, `player_ai` (as car+0x5b8) | `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_power`, `vehicle_tick_drivetrain`, `vehicle_update_visuals`, `player_ai` (to the HUD speedometer, as car+0x5b8) | the automatic shift and the reverse pedal swap need it (§4) |
| `+0x54c` | `upshift_revs[12]` | H | `vehicle_apply_carinit_params` | `vehicle_tick_drivetrain` | the automatic gearbox shifts up above it |
| `+0x57c` | `downshift_revs[12]` | H | `vehicle_apply_carinit_params` | `vehicle_tick_drivetrain` | and down below it |
| `+0x5ac` | `engine_power` | H | `vehicle_apply_carinit_params` (CARINIT `[1]`) | `vehicle_apply_carinit_params` | feeds `+0x4f8` |
| `+0x5b0` | `unused_5b0` | U | - | - | no access in the class's functions (Open item 3) |
| `+0x5b4` | `drag_factor_a` | H | `vehicle_apply_carinit_params` | `vehicle_substep_move` | `+0x5b4 * +0x5b8 * v² * 0.65` is the drag term |
| `+0x5b8` | `drag_factor_b` | H | `vehicle_apply_carinit_params` | `vehicle_substep_move` | as `+0x5b4` |
| `+0x5bc` | `throttle_out` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_tick_drivetrain` | `vehicle_is_throttle_applied`, `vehicle_move`, `vehicle_save_state`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | - |
| `+0x5c0` | `throttle_reverse_out` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_tick_drivetrain` | `vehicle_is_throttle_applied`, `vehicle_move`, `vehicle_save_state`, `vehicle_tick_drivetrain` | - |
| `+0x5c4` | `engine_revs` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_update_motor_revs` | `vehicle_compute_motor_force`, `vehicle_get_engine_rpm`, `vehicle_move`, `vehicle_save_state`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_engine_pitch`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_revs`, `vehicle_update_rotation_wheels`, `vehicle_update_skid_sound`, `vehicle_update_strips`, `car_extended_update` | `vehicle_get_engine_rpm` returns it x 60 while the engine runs |
| `+0x5c8` | `engine_revs_prev` | M | `vehicle_reset`, `vehicle_game_init` (0), `vehicle_tick_drivetrain` (2) | - | no reader in either spelling (`+0x5c8`, car+0x638) |
| `+0x5cc` | `driven_wheel_spun` | H | `vehicle_move` (1), `vehicle_update_visuals` (0), `vehicle_load_state` | `vehicle_tick_drivetrain`, `vehicle_save_state` | set with wheel bit `0x80`; blocks the asleep revs-only path |
| `+0x5d0` | `road_load_torque` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_move` (0, then + grip * radius per driven wheel down) | `vehicle_update_motor_revs`, `vehicle_save_state` | over the gear ratio, the cap of the clutch torque |
| `+0x5d4` | `idle_revs_half` | M | `vehicle_apply_carinit_params` | - | no reader in either spelling; the car+0x644 loads outside the class read `C_human`'s `cur_health` |
| `+0x5d8` | `gear` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` | `vehicle_apply_light_state`, `vehicle_compute_motor_force`, `vehicle_move`, `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_gear`, `vehicle_set_power`, `vehicle_tick_drivetrain`, `vehicle_update_engine_pitch`, `vehicle_update_motor_revs`, `vehicle_update_rotation_wheels`, `vehicle_update_skid_sound`, `vehicle_update_visuals` | - |
| `+0x5dc` | `gear_selected` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_gear`, `vehicle_tick_drivetrain` | `vehicle_save_state`, `vehicle_set_gear`, `vehicle_update_motor_revs`, `vehicle_update_visuals` | - |
| `+0x5e0` | `redline_revs` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params` | - |
| `+0x5e4` | `gear_count` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_get_max_engine_torque_at_rpm`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | - |
| `+0x5e8` | `gear_ratio[12]` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_get_max_engine_torque_at_rpm` | - |
| `+0x618` | `speed` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_set_velocity`, `vehicle_settle`, `vehicle_update_cached_transform` | `vehicle_compute_motor_force`, `vehicle_get_speed`, `vehicle_move`, `vehicle_substep_move`, `vehicle_tick_drivetrain`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_particles`, `vehicle_update_motor_revs`, `vehicle_update_skid_sound`, `vehicle_update_steering_wheels`, `vehicle_update_visuals`, `vehicle_update_wheel_particles`, `vehicle_wheel_contact_solve` | - |
| `+0x61c` | `speed_prev` | H | `vehicle_game_init` (0), `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain` | `gearbox_state` bit `0x8` is set during the launch when the speed falls below it |
| `+0x620` | `speed_along_wheels` | H | `vehicle_game_init` (0), `vehicle_move` | `vehicle_tick_drivetrain` | speed * the absolute dot of the velocity direction with the wheel solver's summed vector, normalised, or the speed when that vector is short |
| `+0x624` | `speed_along_wheels_prev` | M | `vehicle_game_init` (0), `vehicle_tick_drivetrain` | - | no reader in either spelling (`+0x624`, car+0x694) |
| `+0x628` | `drive_circumference` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_init_sounds`, `vehicle_move`, `vehicle_tick_drivetrain`, `vehicle_update_motor_revs`, `vehicle_update_skid_sound`, `vehicle_update_visuals` | - |
| `+0x62c` | `drive_radius` | H | `vehicle_apply_carinit_params` (1e-4, then the radius) | `vehicle_update_rotation_wheels` | used for wheels with bit `0x80` |
| `+0x630` | `step_distance` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_substep_move` (0), `vehicle_move` (+ the step distance) | `vehicle_substep_move`, `vehicle_update_visuals`, `vehicle_save_state` | added to each wheel's `+0x1c8` and, along `+0x3d4`, to the odometer |
| `+0x634` | `brake_force` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_brake`, `vehicle_set_power`, `vehicle_tick_drivetrain` | `vehicle_apply_light_state`, `vehicle_move`, `vehicle_save_state`, `vehicle_substep_move`, `vehicle_tick_drivetrain`, `vehicle_update_visuals` | lights the brake lamps (§5) |
| `+0x638` | `engine_brake` | H | `vehicle_apply_carinit_params` (CARINIT `[0xb]`) | `vehicle_compute_moment`, `vehicle_apply_carinit_params` | scaled by revs / max revs; feeds `+0x24` and `+0x6b8` |
| `+0x63c` | `launch_revs[3]` | H | `vehicle_apply_carinit_params` (CARINIT `[0x10]`..`[0x12]` rpm / 60) | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` | `[0x10]`, `[0x11]` <= `[0x12]` <= redline |
| `+0x648` | `launch_target` | H | `vehicle_game_init` (-1.0), `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain` | - |
| `+0x64c` | `clutch_capacity` | H | `vehicle_apply_carinit_params` (CARINIT `[0xd]`, stored only when at least 0) | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` | capacity = gear ratio * `+0x658` * `+0x64c` * `+0x44` |
| `+0x650` | `clutch_engage_rate` | H | `vehicle_apply_carinit_params` | `vehicle_tick_drivetrain` | `+0x658` rises at it while `gearbox_state` bit `0x1` is set |
| `+0x654` | `clutch_disengage_rate` | H | `vehicle_apply_carinit_params` | `vehicle_tick_drivetrain` | `+0x658` falls at it while `gearbox_state` bit `0x2` is set, then the gear engages; `+0x660` follows `+0x65c` at it |
| `+0x658` | `clutch_engagement` | H | `vehicle_reset`, `vehicle_game_init` (1.0), `vehicle_load_state`, `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs`, `vehicle_move`, `vehicle_save_state` | - |
| `+0x65c` | `clutch_out` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_set_clutch` | `vehicle_save_state`, `vehicle_tick_drivetrain` | - |
| `+0x660` | `clutch_pedal_smoothed` | H | `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init` (1.0), `vehicle_load_state`, `vehicle_tick_drivetrain` | `vehicle_tick_drivetrain`, `vehicle_save_state` | follows `+0x65c` at `+0x654` per second, at once while `+0x527` is set |
| `+0x664` | `gearbox_state` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` | `vehicle_move`, `vehicle_reset`, `vehicle_save_state`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` | bit map in §2b |
| `+0x668` | `idle_revs` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_compute_moment`, `vehicle_tick_drivetrain`, `vehicle_update_engine_pitch`, `vehicle_update_strips` | - |
| `+0x66c` | `engine_dead` | H | `vehicle_apply_damage`, `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_engine_on`, `vehicle_tick_drivetrain`, `vehicle_update_engine_sound` | `vehicle_save_state`, `vehicle_tick_drivetrain` | cleared by `vehicle_set_engine_on` |
| `+0x670` | `mass` | H | `vehicle_apply_carinit_params` (= `+0x674`), `vehicle_load_state` | `vehicle_apply_carinit_params`, `vehicle_apply_external_impulse`, `vehicle_do_collision`, `vehicle_get_contact_impulse_terms`, `vehicle_move`, `vehicle_tick_drivetrain`, `vehicle_wheel_contact_solve`, `vehicle_body_coll_resolve`, `vehicle_save_state` | also scales the km/h damage thresholds; impulse / mass above 1.4 plays the wheel thump, above 5 (or a speed above 2) emits body particles |
| `+0x674` | `body_mass` | H | `vehicle_apply_carinit_params` (CARINIT `[0]`) | `vehicle_apply_carinit_params`, `vehicle_set_seat_mass` | `def-files-docs/vehicles-bin.md` §4a |
| `+0x678` | `com_box_fraction[3]` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params` | `[0x43]` is the Center mass row |
| `+0x684` | `steer_lock[2]` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_update_steering_wheels` | - |
| `+0x68c` | `steer_lock_now` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_update_steering_wheels` | `vehicle_move`, `vehicle_save_state`, `vehicle_set_steer`, `vehicle_update_visuals` | - |
| `+0x690` | `steer_rate[4]` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_update_steering_wheels` | - |
| `+0x6a0` | `steer_target` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_steer` | `vehicle_save_state`, `vehicle_update_steering_wheels` | - |
| `+0x6a4` | `steer_angle` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_update_steering_wheels` | `vehicle_apply_light_state`, `vehicle_save_state`, `vehicle_substep_move`, `vehicle_update_steering_wheels`, `vehicle_update_visuals` | - |
| `+0x6a8` | `steer_split_max` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_update_steering_wheels` | - |
| `+0x6b0` | `steer_split` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_update_steering_wheels` | `vehicle_save_state` | - |
| `+0x6b4` | `engine_on_mirror` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_set_engine_on`, `vehicle_update_engine_sound` | `vehicle_save_state`, `vehicle_tick_drivetrain`, `vehicle_update_engine_sound` | - |
| `+0x6b8` | `load_torque_min` | H | `vehicle_apply_carinit_params` (`[0x1ed] * +0x638 * -0.01`) | `vehicle_compute_motor_force` | - |
| `+0x6bc` | `load_torque_max` | H | `vehicle_apply_carinit_params` (`[0x1ee] * +0x4f4 * 0.01`) | `vehicle_compute_motor_force` | - |
| `+0x6c0` | `load_rise_rate` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1ef]`) | `vehicle_compute_motor_force` | - |
| `+0x6c4` | `load_fall_rate` | H | `vehicle_apply_carinit_params` (CARINIT `[0x1f0]`) | `vehicle_compute_motor_force` | - |
| `+0x6c8` | `int_snd_engine_start` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9a8` |
| `+0x6cc` | `int_snd_engine_stop` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9ac` |
| `+0x6d0` | `int_snd_start_fail` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9b0` |
| `+0x6d4` | `int_snd_engine_load[5]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9b8` |
| `+0x6e8` | `int_snd_engine_coast[5]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9cc` |
| `+0x6fc` | `int_snd_engine_reverse[5]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9e0` |
| `+0x710` | `int_snd_engine_idle` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9f4` |
| `+0x714` | `int_snd_environment` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9f8` |
| `+0x718` | `int_snd_horn` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0x9fc` |
| `+0x71c` | `int_snd_siren` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa00` |
| `+0x720` | `int_snd_handbrake` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa04` |
| `+0x724` | `int_snd_gear[2]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa08` |
| `+0x72c` | `int_snd_skid` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa10` |
| `+0x730` | `int_snd_brake_loop` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa14` |
| `+0x734` | `int_snd_wheel_thump` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa18` |
| `+0x738` | `int_snd_tyre_burst` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa1c` |
| `+0x73c` | `int_snd_crash[4]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa20` |
| `+0x74c` | `int_snd_crash_alt[4]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa30` |
| `+0x75c` | `int_snd_door[2]` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa40` |
| `+0x764` | `int_snd_puncture` | M | `vehicle_ctor` (0), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | never opened; the exterior twin is `+0xa48` |
| `+0x768` | `int_bank_load` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in a forward or neutral gear; never played: the interior handles are never opened |
| `+0x810` | `int_bank_coast` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in a forward or neutral gear; never played |
| `+0x8b8` | `int_bank_reverse` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in reverse gear; never played |
| `+0x958` | `int_idle_revs_end` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | the idle volume is scaled by `(end - revs) / (end - start)` |
| `+0x95c` | `int_idle_revs_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | - |
| `+0x960` | `int_idle_volume` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | - |
| `+0x964` | `int_environment_pitch[2]` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0x96c` | `int_environment_speed_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0x970` | `int_environment_speed_cap` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0x974` | `int_environment_speed_full` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0x978` | `unread_978` | U | `vehicle_apply_carinit_params` | - | clamped at `0x00514954`..`0x0051498d`; no other access in the image |
| `+0x97c` | `int_puncture_pitch[2]` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0x984` | `int_puncture_speed_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0x988` | `int_puncture_speed_cap` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0x98c` | `int_puncture_speed_full` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0x990` | `unread_990` | U | `vehicle_apply_carinit_params` | - | no other access in the image |
| `+0x994` | `int_crash_impulse[4]` | H | `vehicle_apply_carinit_params` (CARINIT `[0x314]`..`[0x317]` km/h to m/s, times `+0x670`) | `vehicle_play_crash_sound` | below the first bound nothing plays |
| `+0x9a4` | `int_crash_mute_grounded[4]` | H | `vehicle_apply_carinit_params` (CARINIT bytes `+0xc60`..`+0xc63`) | `vehicle_play_crash_sound` | the grounded argument is `+0x410` |
| `+0x9a8` | `ext_snd_engine_start` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xc64` |
| `+0x9ac` | `ext_snd_engine_stop` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xc74` |
| `+0x9b0` | `ext_snd_start_fail` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xc84` |
| `+0x9b4` | `ext_snd_engine_loop` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xc94`; it has no interior twin |
| `+0x9b8` | `ext_snd_engine_load[5]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xca4`..`+0xce4` |
| `+0x9cc` | `ext_snd_engine_coast[5]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xcf4`..`+0xd34` |
| `+0x9e0` | `ext_snd_engine_reverse[5]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xd44`..`+0xd84` |
| `+0x9f4` | `ext_snd_engine_idle` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xd94` |
| `+0x9f8` | `ext_snd_environment` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xda4`, only when `+0x628` is above 0.01 |
| `+0x9fc` | `ext_snd_horn` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xdb4` |
| `+0xa00` | `ext_snd_siren` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xdc4` with 4.0 / 110.0 / 0.45 |
| `+0xa04` | `ext_snd_handbrake` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xdd4` |
| `+0xa08` | `ext_snd_gear[2]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xde4`, `+0xdf4` |
| `+0xa10` | `ext_snd_skid` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xe04` |
| `+0xa14` | `ext_snd_brake_loop` | M | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds`, `vehicle_release_sounds` | opened from CARINIT `+0xe14`, never started; the role is the shipped file name of its slot (`brzdy_loop.wav`) |
| `+0xa18` | `ext_snd_wheel_thump` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xe24` |
| `+0xa1c` | `ext_snd_tyre_burst` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xe34` |
| `+0xa20` | `ext_snd_crash[4]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xe54`..`+0xe84` |
| `+0xa30` | `ext_snd_crash_alt[4]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xe94`..`+0xec4` |
| `+0xa40` | `ext_snd_door[2]` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from CARINIT `+0xed4`, `+0xee4`; index 0 = `vehicle_play_door_sound`'s flag argument set, 1 = clear |
| `+0xa48` | `ext_snd_puncture` | H | `vehicle_ctor` (0), `vehicle_init_sounds` (through `vehicle_sound_slot_init`), `vehicle_release_sounds` (0) | the sound functions of §8e, `vehicle_add_sounds_to_sectors`, `vehicle_remove_sounds_from_sectors`, `vehicle_stop_all_sounds` | opened from the hard-coded `PNEU_PUNC.WAV` (6.0 / 30.0) |
| `+0xa4c` | `ext_engine_loop_pitch[2]` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` (`0x005387c4`..`0x00538826`) | pitch `+0xa4c + (revs - idle) * (+0xa50 - +0xa4c) / (max - idle)` |
| `+0xa54` | `ext_engine_loop_revs_full` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | the volume is scaled by `(revs - idle) / (+0xa54 - idle)` below it |
| `+0xa58` | `ext_engine_loop_volume` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | - |
| `+0xa5c` | `ext_bank_load` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in a forward or neutral gear; its mute bytes silence it in neutral |
| `+0xb04` | `ext_bank_coast` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in a forward or neutral gear; its mute bytes silence it in neutral |
| `+0xbac` | `ext_bank_reverse` | H | `vehicle_apply_carinit_params` (only for a non-empty CARINIT sound name) | `vehicle_update_engine_pitch` | plays only in reverse gear |
| `+0xc4c` | `ext_idle_revs_end` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | the idle volume is scaled by `(end - revs) / (end - start)` |
| `+0xc50` | `ext_idle_revs_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | - |
| `+0xc54` | `ext_idle_volume` | H | `vehicle_apply_carinit_params` | `vehicle_update_engine_pitch` | - |
| `+0xc58` | `ext_environment_pitch[2]` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0xc60` | `ext_environment_speed_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0xc64` | `ext_environment_speed_cap` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0xc68` | `ext_environment_speed_full` | H | `vehicle_apply_carinit_params` | `vehicle_update_environment_sound` | - |
| `+0xc6c` | `unread_c6c` | U | `vehicle_apply_carinit_params` | - | clamped at `0x00514b9a`; no other access in the image |
| `+0xc70` | `ext_puncture_pitch[2]` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0xc78` | `ext_puncture_speed_start` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0xc7c` | `ext_puncture_speed_cap` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0xc80` | `ext_puncture_speed_full` | H | `vehicle_apply_carinit_params` | `vehicle_update_flat_tyre_sound` | - |
| `+0xc84` | `unread_c84` | U | `vehicle_apply_carinit_params` | - | no other access in the image |
| `+0xc88` | `ext_crash_impulse[4]` | H | `vehicle_apply_carinit_params` (CARINIT `[0x444]`..`[0x447]` km/h to m/s, times `+0x670`) | `vehicle_play_crash_sound` | below the first bound nothing plays |
| `+0xc98` | `ext_crash_mute_grounded[4]` | H | `vehicle_apply_carinit_params` (CARINIT bytes `+0x1120`..`+0x1123`, the record's last dword) | `vehicle_play_crash_sound` | the grounded argument is `+0x410` |
| `+0xc9c` | `engine_sound_fade_s` | H | `vehicle_game_init`, `vehicle_set_engine_on`, `vehicle_update_engine_pitch`, `vehicle_update_engine_sound` | `vehicle_update_engine_pitch` | the engine volume is scaled by `t / 0.3`; the forced stop sets 5000 |
| `+0xca0` | `physics_substep_s` | H | `vehicle_ctor`, `vehicle_load_state`, `vehicle_set_cut_time`, `car_update` (as car+0xd10: 0.028 or 0.042) | `vehicle_save_state`, `vehicle_substep_move` | - |
| `+0xca4` | `steering_wheel_frame` | H | `vehicle_init_strip_and_smoke_emitters` | `vehicle_update_visuals` | with `feature_flags` `0x2` turned about `+0x3f8` by `+0x6a4 / +0x68c * 1.57` |
| `+0xca8` | `engine_on` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_set_engine_on`, `vehicle_tick_drivetrain`, `vehicle_update_engine_sound`, `vehicle_update_visuals` | `vehicle_compute_motor_force`, `vehicle_get_engine_on`, `vehicle_get_engine_rpm`, `vehicle_move`, `vehicle_save_state`, `vehicle_set_brake`, `vehicle_set_engine_on`, `vehicle_set_power`, `vehicle_tick_drivetrain`, `vehicle_update_engine_pitch`, `vehicle_update_engine_sound`, `vehicle_update_exhaust_particles`, `vehicle_update_visuals` | - |
| `+0xcac` | `fuel` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_set_fuel`, `vehicle_tick_drivetrain` | `vehicle_save_state`, `vehicle_set_engine_on`, `vehicle_tick_drivetrain` | - |
| `+0xcb0` | `wheel_node_list` | H | `vehicle_ctor` (0), `vehicle_init` (the frame enumeration) | `vehicle_init`, `vehicle_reset`, `vehicle_game_done`, `vehicle_dtor` (frees it) | `vehicle_cb_enum_objects` links the `0x1d4`-byte wheel records through `+0x1d0`; `wheels` `+0xcb4` is built from it; `vehicle_game_done` resets each steered node's frame rotation |
| `+0xcb4` | `wheels` | H | `vehicle_ctor`, `vehicle_init` | `vehicle_apply_carinit_params`, `vehicle_deactivate`, `vehicle_deform`, `vehicle_dtor`, `vehicle_emit_wheel_skid_segment`, `vehicle_end_wheel_skid_segment`, `vehicle_game_done`, `vehicle_game_init`, `vehicle_get_wheel_at`, `vehicle_get_wheel_cam_pos`, `vehicle_init`, `vehicle_init_damage`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_play_tyre_burst_sound`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_save_state`, `vehicle_substep_move`, `vehicle_tick_drivetrain`, `vehicle_update_axis`, `vehicle_update_rotation_wheels`, `vehicle_update_skid_sound`, `vehicle_update_steering_wheels`, `vehicle_update_strips`, `vehicle_update_visuals`, `vehicle_update_wheel_particles`, `vehicle_wheel_contact_solve` | - |
| `+0xcb8` | `axis_forward[3]` | H | `vehicle_game_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_update_cached_transform` | `vehicle_apply_light_state`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_substep_move`, `vehicle_tick_drivetrain`, `vehicle_update_flat_tyre_sound`, `vehicle_update_rotation_wheels`, `vehicle_wheel_contact_solve` | seeded (0, 0, 1) |
| `+0xcc4` | `axis_right[3]` | H | `vehicle_game_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_update_cached_transform` | `vehicle_get_right_axis_ptr`, `vehicle_load_state`, `vehicle_move`, `vehicle_substep_move`, `vehicle_wheel_contact_solve`, `entity_action_tick`, `pursuit_event_tick` (through `vehicle_get_right_axis_ptr`); `car_update`, `pursuit_event_tick_driver` (as car+0xd34) | seeded (1, 0, 0) |
| `+0xcd0` | `axis_up[3]` | H | `vehicle_game_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_update_cached_transform` | `vehicle_get_up_axis`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset_from_frame`, `vehicle_substep_move`, `vehicle_wheel_contact_solve` | seeded (0, 1, 0) |
| `+0xcdc` | `wake_request` | H | `vehicle_reset_from_frame` (1), `vehicle_reset` (0, then 1), `vehicle_game_init` (0), `vehicle_apply_external_impulse`, `vehicle_apply_external_force`, `vehicle_do_collision`, `vehicle_set_seat_mass`, `vehicle_settle` (all 1), `vehicle_load_state`, `vehicle_update_visuals` (0 at its end) | `vehicle_tick_drivetrain`, `vehicle_move` (3), `vehicle_update_visuals` (2), `vehicle_save_state` | `vehicle_tick_drivetrain` sets `master_flags` `0x1` and zeroes `+0x228` and `+0x04`; `vehicle_move` skips the in-gear stop and the rest test and forces the body collision; `vehicle_update_visuals` does not return 2 while it is set |
| `+0xce0` | `velocity_heading_angle` | H | `vehicle_reset`, `vehicle_game_init` (0), `vehicle_load_state`, `vehicle_move` (the end of each step) | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs`, `vehicle_move`, `vehicle_save_state` | `S_vector::AngleTo` (the `ls3df.dll` import) of `+0xcb8` and `+0x1f0c`; above pi / 2 the car moves against its gear |
| `+0xce4` | `feature_flags` | H | `vehicle_ctor`, `vehicle_load_state`, `vehicle_settle`, `vehicle_update_visuals` | `vehicle_apply_light_state`, `vehicle_deform`, `vehicle_do_collision`, `vehicle_emit_body_collision_particles`, `vehicle_move`, `vehicle_play_gear_change_sound`, `vehicle_play_handbrake_sound`, `vehicle_save_state`, `vehicle_set_engine_on`, `vehicle_set_gear`, `vehicle_settle`, `vehicle_tick_drivetrain`, `vehicle_update_axis`, `vehicle_update_engine_pitch`, `vehicle_update_environment_sound`, `vehicle_update_exhaust_particles`, `vehicle_update_flat_tyre_sound`, `vehicle_update_motor_particles`, `vehicle_update_visuals`, `vehicle_update_wheel_particles`, `vehicle_wheel_contact_solve` | - |
| `+0xce8` | `feature_flags_prev` | H | `vehicle_ctor`, `vehicle_load_state`, `vehicle_update_visuals` (= `+0xce4`, after the light and axis updates) | `vehicle_apply_light_state`, `vehicle_save_state`, `vehicle_update_axis`, `vehicle_update_visuals` | the previous frame's `feature_flags`: `vehicle_update_visuals` calls `vehicle_update_axis` when the two words differ, a change of bit `0x1` shows or hides the axle frames, and bit `0x80` keeps the lamp pass running one frame after the lamps are switched off |
| `+0xcec` | `build_flags` | H | `vehicle_ctor` (0), `car_explosion` (3), `traffic_car_game_init` (1), `police_manager_apply_state` (3) (as car+0xd5c) | `vehicle_init` (4, bit `0x2`), `vehicle_game_init` (bit `0x1`), `vehicle_apply_carinit_params` (bit `0x2`) | bit `0x2` relaxes the seat and wheel requirements and accepts a car with no driven wheel; bit `0x1` skips the tyre-width parse and the skid-mark models (Open item 10) |
| `+0xcf0` | `owner_car` | H | `vehicle_ctor` (0), `car_ctor` (as car+0xd60) | every function that fires a bank slot (31 hits), `vehicle_game_init` | also the owner word of both collision records and of each `+0x1f38` record (§10) |
| `+0xcf4` | `unread_cf4[10]` | U | `vehicle_game_init` | - | no other access in the image |
| `+0xd1c` | `seats` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_init` | `vehicle_animate_doors`, `vehicle_animate_windows`, `vehicle_dtor`, `vehicle_game_init`, `vehicle_get_save_state_size`, `vehicle_get_seat_count`, `vehicle_init`, `vehicle_init_strip_and_smoke_emitters`, `vehicle_is_window_open`, `vehicle_load_state`, `vehicle_open_door`, `vehicle_open_window`, `vehicle_play_door_sound`, `vehicle_save_state`, `vehicle_set_door_open_speed`, `vehicle_set_max_door_angle`, `vehicle_set_seat_mass`, `vehicle_update_visuals` | - |
| `+0xd2c` | `door_windows` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_init` | `vehicle_dtor`, `vehicle_init` | - |
| `+0xd3c` | `ballast` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_init` | `vehicle_dtor`, `vehicle_init`, `vehicle_set_seat_mass` | - |
| `+0xd4c` | `ext_force_sum[3]` | H | `vehicle_apply_external_force` (adds the force unless its flag bit `0x1`), `vehicle_game_init`, `vehicle_update_visuals` (0 every frame), `vehicle_load_state` | `vehicle_move`, `vehicle_save_state` | added to the step's force sum |
| `+0xd58` | `ext_torque_sum[3]` | H | `vehicle_apply_external_force` (adds `(point - +0x414) x force` unless its flag bit `0x2`), `vehicle_game_init`, `vehicle_update_visuals` (0 every frame), `vehicle_load_state` | `vehicle_move`, `vehicle_save_state` | added to the step's torque sum |
| `+0xd64` | `world_matrix` | H | `vehicle_game_init`, `vehicle_reset_from_frame`, `vehicle_update_cached_transform`, `vehicle_reset` | `vehicle_game_init`, `vehicle_update_cached_transform`, `vehicle_apply_external_impulse`, `vehicle_apply_external_force`, `vehicle_substep_move`, `vehicle_move` | `vehicle_substep_move` and `vehicle_move` copy it into `+0x068`; `&+0xd64` is the transform pointer of both collision records |
| `+0xda4` | `world_matrix_inv` | H | `vehicle_init`, `vehicle_reset_from_frame`, `vehicle_reset`, `vehicle_game_init`, `vehicle_update_cached_transform` | `vehicle_game_init`, `vehicle_init`, `vehicle_deactivate`, `vehicle_apply_external_impulse`, `vehicle_apply_external_force`, `vehicle_do_collision`, `vehicle_get_contact_impulse_terms`, `vehicle_substep_move`, `vehicle_move` | `vehicle_substep_move` and `vehicle_move` copy it into `+0x0a8`; `vehicle_move` turns the angular velocity into body space with it; `&+0xda4` is the inverse pointer of both collision records |
| `+0xde4` | `body_frame` | H | `vehicle_init` (0, then the enumeration's result) | `vehicle_init` | `vehicle_init` returns 0 without it and enumerates `BALLAST*` under it |
| `+0xde8` | `init` | H | `vehicle_apply_carinit_params` | `vehicle_game_init`, `vehicle_get_carinit_params`, `vehicle_init_sounds`, `vehicle_apply_carinit_params`, `vehicle_deform_glass` | with its argument bit `0x1`, `vehicle_apply_carinit_params` replaces an invalid CARINIT value from the stored record; `vehicle_deform_glass` passes the glass-break name at `init + 0x994` (while `+0x50d` is set) or `+0xe44` to bank slot `+0x380` |
| `+0x1f0c` | `velocity[3]` | H | `vehicle_apply_external_impulse`, `vehicle_do_collision`, `vehicle_game_init`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_set_velocity`, `vehicle_settle`, `vehicle_deactivate`, `vehicle_reset_from_frame` | `vehicle_apply_external_impulse`, `vehicle_compute_motor_force`, `vehicle_deform`, `vehicle_do_collision`, `vehicle_emit_body_collision_particles`, `vehicle_game_init`, `vehicle_get_contact_impulse_terms`, `vehicle_get_velocity_ptr`, `vehicle_load_state`, `vehicle_move`, `vehicle_reset`, `vehicle_save_state`, `vehicle_set_velocity`, `vehicle_settle`, `vehicle_update_cached_transform`, `vehicle_update_environment_sound`, `vehicle_update_exhaust_particles`, `vehicle_update_flat_tyre_sound`, `vehicle_update_motor_particles`, `vehicle_update_visuals`, `vehicle_wheel_contact_solve` | - |
| `+0x1f18` | `max_revs` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_compute_moment`, `vehicle_compute_motor_force`, `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_update_engine_pitch`, `vehicle_update_exhaust_particles`, `vehicle_update_motor_revs`, `vehicle_update_strips` | - |
| `+0x1f1c` | `fuel_capacity` | H | `vehicle_apply_carinit_params` | `vehicle_apply_carinit_params`, `vehicle_set_fuel` | - |
| `+0x1f20` | `odometer` | H | `vehicle_game_init` (0), `vehicle_load_state`, `vehicle_update_visuals`, `car_game_init` (as car+0x1f90 = 251000 + rand() * 150) | `vehicle_update_visuals`, `vehicle_save_state`, `player_ai`, `player_ai_drive` (as car+0x1f90) | `+= (wheel 0 +0x58 . +0x3d4) * +0x630` every frame; the fourth argument of `hud_speedometer_set_values`, the HUD's five-digit strip |
| `+0x1f24` | `engine_cut_request` | H | `vehicle_reset`, `vehicle_game_init`, `vehicle_load_state`, `car_ai_tick` (as car+0x1f94, 2) | `vehicle_tick_drivetrain` (2), `vehicle_update_visuals`, `vehicle_save_state` | while set `vehicle_tick_drivetrain` saves `+0xca8` into `+0x1f25`, switches the engine off, zeroes both throttles and stops the motor sounds; `car_ai_tick` sets it in the off phase of the random cut-out of a car with `broken_motor` (car `+0x204d`; phases of 100..299 / 100..399 ms, brake 0.3, below 1 m/s the engine switched off) |
| `+0x1f25` | `engine_on_saved` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_tick_drivetrain` | `vehicle_update_visuals`, `vehicle_save_state` | `vehicle_update_visuals` restores `+0xca8` from it |
| `+0x1f26` | `special_dyn_colls` | H | `vehicle_game_init`, `vehicle_load_state`, `vehicle_reset`, `vehicle_set_special_dynamic_colls` | `vehicle_deactivate`, `vehicle_game_done`, `vehicle_get_special_dynamic_colls`, `vehicle_reset_from_frame`, `vehicle_save_state`, `vehicle_set_dynamic_colls_off`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals` | - |
| `+0x1f28` | `special_coll_boxes` | H | `vehicle_init` (`SPECCOLL*` frames), `vehicle_dtor` (frees it) | `vehicle_game_init`, `vehicle_dtor` | two `S_vector`s per box (minimum, maximum); `vehicle_game_init` builds one `0x48` record per pair into `+0x1f38` (mask `0x201`) |
| `+0x1f38` | `special_dyn_coll_records` | H | `vehicle_ctor`, `vehicle_dtor`, `vehicle_game_done`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals` | `vehicle_deactivate`, `vehicle_dtor`, `vehicle_game_done`, `vehicle_game_init`, `vehicle_reset_from_frame`, `vehicle_set_dynamic_colls_off`, `vehicle_set_special_dynamic_colls`, `vehicle_update_visuals` | each record's centre `+0x10` follows `+0x39c` (`vehicle_set_special_dynamic_colls`; `vehicle_update_visuals` while `+0x1f26` is set) |

### 2a. `light_state` (`+0x19c`) — the lamp bits

Every exterior lamp of a car is one bit of this word. `car_update` and `car_deactivate` reach it as
`car + 0x20c`.

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | left turn indicator | script case `0xc2` `carlight_indic_l`; the Mafia Script Data Reference gives *"left indicator"*; confirmed in game |
| `0x2` | right turn indicator | case `0xc3` (`script-mapping/artifacts/opcodes-verified.md`); the Mafia Script Data Reference gives *"right indicator"*; confirmed in game |
| `0x8` | indicators driven automatically from the steering angle | gates both side setters; `vehicle_apply_light_state` lights the side of the steering sign while `abs(+0x6a4)` exceeds `+0x1b0` |
| `0x10` / `0x20` | the two phases of the emergency light bar | `vehicle_apply_light_state` alternates them while `0x80` is set and clears both once it is off (§5) |
| `0x80` | emergency light bar | case `0xc1` `carlight_light` through `vehicle_set_emergency_lights`, and the police dispatch paths (§5). An in-game test of `carlight_light` showed no visible effect; only the fire engines and the police cars carry `LIGHTLI*` frames |
| `0x100` | brake lights | `vehicle_apply_light_state` sets it from `+0x634 > 0` while `0x800` is set (§5) |
| `0x800` | automatic brake lights | set with `0x8` and `0x8000` when a car is initialised or a driver takes it (§5); `driver_tick` sets it, `driver_reset` clears it |
| `0x1000` | reverse lights | `vehicle_apply_light_state` sets it from `+0x5d8 < 0` while `0x8000` is set |
| `0x8000` | automatic reverse lights | set with `0x800` (§5) |
| `0x10000` | main headlights | case `0xc0` `carlight_main`, confirmed in game; `car_update` drives it while `C_game + 0x2d44` (night mission) is set |
| `0x0ff00000` | script override sub-field | read and changed by `car_update`, cleared by `car_deactivate`, read / written by `vehicle_get_light_script_override` / `vehicle_set_light_script_override`. One bit per lamp (`0x100000` indicators, `0x200000` light bar, `0x800000` headlights, §5), set when its `carlight_*` command gets the argument 2 (force) |
| `0x10000000` | the indicator blink phase | toggled by `vehicle_apply_light_state` (§5) |
| `0x20000000` | the light-bar phase | the same (§5) |

`driver_do_lights`, the chase AI's light logic, sets `0x800` and `0x8` and flashes `0x10000` on a timed
pattern after `car_set_auto_headlights_suppressed(1)`.

### 2b. `master_flags` (`+0x224`), `feature_flags` (`+0xce4`) and `gearbox_state` (`+0x664`)

"Asleep" means `master_flags` bit `0x1` clear.

| Value | Meaning | Evidence |
|---|---|---|
| `master_flags` `0x1` | awake | set by `+0xcdc`, a contact latch, any motion or input; cleared at the end of a tick (`+0x228`, `vehicle_tick_drivetrain`) |
| `master_flags` `0x2` | at rest this tick | cleared at each tick start; set by `vehicle_move` (speed at most 0.01 and angular speed at most 0.015 with a driven wheel down and no wake request, or below 0.2 / 0.08 after a body hit) and by `vehicle_tick_drivetrain` after 1000 ms asleep without a driver |
| `master_flags` `0x4` | held by the gear | `vehicle_compute_motor_force`: gear in, speed below 0.5, engine off or clutch above 0.9; `vehicle_move` then stops the car while the summed contact normal is within 0.98 of up and the drive pushes back |
| `master_flags` `0x8` | no wheel on the ground: the vehicle is unstable | set by `vehicle_tick_drivetrain` when no wheel has bit `0x8`, cleared only when all have it and none has `0x10`; `vehicle_is_unstable` |
| `master_flags` `0x1000000` | the brake forced to 1.0 this tick | set by `vehicle_tick_drivetrain` when the car moves against its gear with the throttle open, or at idle revs below 0.7 m/s with the engine on; cleared after the same tick's shift block and by both pedal setters; the automatic shift waits while it is set |
| `master_flags` `0x2000000` | the copied contact latch: a contact with collision flag `0x10000` | `vehicle_update_visuals` copies `+0x28` into it; `vehicle_wheel_contact_solve` and `human_find_blocking_drawbridge` (as `car + 0x294`) read it |
| `master_flags` `0x10000000` | locked | `vehicle_lock` |
| `feature_flags` `0x1` | the axle frames of `+0x3a8` / `+0x3b8` are updated; a change against `+0xce8` shows or hides all of them | `vehicle_update_axis` |
| `feature_flags` `0x2` | the steering wheel turns | `vehicle_update_visuals` |
| `feature_flags` `0x4` | the wheels rotate | `vehicle_update_visuals` |
| `feature_flags` `0x8` | downforce acts at the two downforce points | `vehicle_move` |
| `feature_flags` `0x10` | body collision; set by `vehicle_update_visuals` when a wheel has come off | `vehicle_move`, `vehicle_update_visuals` |
| `feature_flags` `0x20` | clear: the exterior engine is the single loop `+0x9b4` and the handbrake, gear, environment, puncture and wheel-thump sounds are off; set: the three engine banks play | `vehicle_update_engine_pitch`, `vehicle_play_handbrake_sound`, `vehicle_play_gear_change_sound`, `vehicle_update_environment_sound`, `vehicle_update_flat_tyre_sound`, `vehicle_wheel_contact_solve` |
| `feature_flags` `0x40` | skid marks | `vehicle_update_visuals` |
| `feature_flags` `0x80` | the lamps are on: every lamp arm lights only while it is set, and the pass returns at once when neither `+0xce4` nor `+0xce8` has it | `vehicle_apply_light_state` |
| `feature_flags` `0x100` | the lamp light `+0x08` is placed and shown, else hidden | `vehicle_apply_light_state` |
| `feature_flags` `0x200` | all four particle emitters: motor smoke, wheel, exhaust and body particles | `vehicle_update_motor_particles`, `vehicle_update_wheel_particles`, `vehicle_update_exhaust_particles`, `vehicle_emit_body_collision_particles` |
| `feature_flags` `0x400` | the body collision runs on every sub-step, not only the last | `vehicle_move` |
| `feature_flags` `0x800` | every wheel runs its own ground line test instead of the two axle planes | `vehicle_wheel_contact_solve` |
| `feature_flags` `0x1000` | the lamp light `+0x0c` is placed and shown, else hidden | `vehicle_apply_light_state` |
| `feature_flags` `0x4000` | damage acts on the drivetrain: an empty gearbox pool refuses every gear change and stops the automatic shift, an empty engine pool refuses a start; it also gates the wheel damage of `vehicle_deform` and, in `vehicle_do_collision`, `vehicle_apply_damage` and the was-collision slot `+0x374` | `vehicle_set_gear`, `vehicle_tick_drivetrain`, `vehicle_set_engine_on`, `vehicle_deform`, `vehicle_do_collision` |
| `feature_flags` `0x8000` | impacts deform: `vehicle_do_collision` and a wheel reaching its bump stop call `vehicle_deform` | `vehicle_do_collision`, `vehicle_wheel_contact_solve` |
| `gearbox_state` `0x1` | the clutch engages after a shift (`+0x658` rises at `+0x650`) | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` |
| `gearbox_state` `0x2` | the clutch disengages for a shift; at 0 the gear becomes `+0x5dc` and the gear sound plays | `vehicle_tick_drivetrain`, `vehicle_update_motor_revs` |
| `gearbox_state` `0x4` | the launch phase | `vehicle_tick_drivetrain` |
| `gearbox_state` `0x8` | set during the launch when the speed falls below `+0x61c` (Open item 5) | `vehicle_tick_drivetrain` |
| `gearbox_state` `0x10` | revs at idle | `vehicle_tick_drivetrain` |
| `gearbox_state` `0x20` | the automatic clutch; `vehicle_game_init` sets it | `vehicle_game_init`, `vehicle_tick_drivetrain` |

`vehicle_ctor` seeds `feature_flags` with `0xdfff` (every bit of the low 16 but `0x2000`), and every
bit of that seed is in the table. `vehicle_settle` sets `master_flags` to 1 and `feature_flags` to
`0x801` for its loop and then restores `feature_flags`. The automatic shift runs only while no
`gearbox_state` bit other than `0x10` / `0x20` is set.

### 2c. The four input bytes `+0x524`..`+0x527`

One byte per driving axis. Set, the setter treats its input as an analog value (shaped by the
linearity coefficient, or applied without the pedal ramp); clear, as a digital full-scale value.

| Byte | Axis | Reader and effect |
|---|---|---|
| `+0x524` | steer | `vehicle_set_steer` (linearity `+0x218`), `vehicle_update_steering_wheels`; `car_update` runs force feedback only while it is set |
| `+0x525` | accelerate | no reader located |
| `+0x526` | brake | `vehicle_set_power` and `vehicle_set_brake` both skip the pedal ramp `+0x444` / `+0x448` while it is set |
| `+0x527` | clutch | `vehicle_set_clutch` (linearity `+0x21c`), `vehicle_tick_drivetrain` |

`player_ai_drive` writes all four every frame the player drives: 1 while that axis comes from its
secondary key binding, 0 once the primary binding takes over (`structs/player.md`). `driver_tick`
writes 0/0/0/0 or 1/1/1/0, `traffic_car_activate_elements` 1/1/1/1 and `car_net_snapshot_state` 0 or
0/1/1/1 (all as `car + 0x594`..`+0x597`); `vehicle_game_init` zeroes them and the save-state pair keeps them. The ramp gate for the accelerator
is the brake byte `+0x526`; `+0x525` is written and never read.

## 3. Method family

153 PS2 methods: 134 mapped to a PC function, 9 inlined into one (rows `inlined`), 10 with no PC
function (§12); the 8 PS2 free callbacks of the class map to §3d. 53 of the 195 `vehicle_` functions
have no PS2 twin.

### 3a. Construction & destruction

The unit initializers open each of the four source files (§8); the five file-scope vectors of `C_Vehicle.cpp` each have a quartet (§8a). The two `S_COLBODY_HEAD` destructors free one record of the body-collision tree (§7).

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00509400 | `crt_static_init_guard_00509400` | the translation unit | - | - | - | the `.CRT$XCU` initializer that opens `C_Vehicle.cpp` |
| 0x00509420 | `crt_atexit_register_00509420` | the translation unit | - | - | - | its `atexit` registrar |
| 0x00509430 | `thunk_stub_noop_cdecl_00509440` | the translation unit | - | - | - | a `.CRT$XCU` entry: a 5-byte `jmp` to the empty initializer below; no registry row |
| 0x00509440 | `stub_noop_cdecl_00509440` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x00509450 | `crt_static_init_00509450` | `g_vehicle_tyre_list_begin` | - | - | - | quartet entry, guard bit `0x01` |
| 0x00509460 | `vehicle_tyre_list_static_ctor` | `g_vehicle_tyre_list_begin` | - | - | - | constructs the tyre list |
| 0x00509480 | `crt_atexit_register_00509480` | `g_vehicle_tyre_list_begin` | - | - | - | registers its destructor |
| 0x00509490 | `vehicle_tyre_list_static_dtor` | `g_vehicle_tyre_list_begin` | - | - | - | frees the tyre list |
| 0x005094d0 | `crt_static_init_005094d0` | `g_vehicle_materials_begin` | - | - | - | quartet entry, guard bit `0x02` |
| 0x005094e0 | `vehicle_material_list_static_ctor` | `g_vehicle_materials_begin` | - | - | - | constructs the material list |
| 0x00509500 | `crt_atexit_register_00509500` | `g_vehicle_materials_begin` | - | - | - | registers its destructor |
| 0x00509510 | `vehicle_material_list_static_dtor` | `g_vehicle_materials_begin` | - | - | - | frees the material list |
| 0x00509550 | `crt_static_init_00509550` | `g_vehicle_particles_begin` | - | - | - | quartet entry, guard bit `0x04` |
| 0x00509560 | `vehicle_particle_list_static_ctor` | `g_vehicle_particles_begin` | - | - | - | constructs the particle list |
| 0x00509580 | `crt_atexit_register_00509580` | `g_vehicle_particles_begin` | - | - | - | registers its destructor |
| 0x00509590 | `vehicle_particle_list_static_dtor` | `g_vehicle_particles_begin` | - | - | - | frees the particle list |
| 0x005095d0 | `crt_static_init_005095d0` | `g_vehicle_projector_light_sectors_begin` | - | - | - | quartet entry, guard bit `0x08` |
| 0x005095e0 | `vehicle_projector_sector_list_static_ctor` | `g_vehicle_projector_light_sectors_begin` | - | - | - | constructs the projector-light sector list |
| 0x00509600 | `crt_atexit_register_00509600` | `g_vehicle_projector_light_sectors_begin` | - | - | - | registers its destructor |
| 0x00509610 | `vehicle_projector_sector_list_static_dtor` | `g_vehicle_projector_light_sectors_begin` | - | - | - | frees it |
| 0x00509650 | `crt_static_init_00509650` | `g_vehicle_sound_sectors_begin` | - | - | - | quartet entry, guard bit `0x10` |
| 0x00509660 | `vehicle_sound_sector_list_static_ctor` | `g_vehicle_sound_sectors_begin` | - | - | - | constructs the sound sector list |
| 0x00509680 | `crt_atexit_register_00509680` | `g_vehicle_sound_sectors_begin` | - | - | - | registers its destructor |
| 0x00509690 | `vehicle_sound_sector_list_static_dtor` | `g_vehicle_sound_sectors_begin` | - | - | - | frees it |
| 0x005096d0 | `vehicle_ctor` | instance | `__ct__9C_VehicleFv` | 0x0022d660 | CONFIRMED | stores `g_vehicle_vt`, zeroes the callback bank, seeds the lamp timers (0.4 / 0.3 / 0.2 / 0.3 s), `+0x130` = 1.0, the two linearities 0.5, `+0xca0` = 0.028 s and `+0xce4` / `+0xce8` = `0xdfff` |
| 0x0050db60 | `vehicle_dtor` | instance | `__dt__9C_VehicleFv` | 0x002337b0 | CONFIRMED | frees the vectors, the body-collision tree (through `vehicle_colbody_head_scalar_deleting_dtor`) and the projector records |
| - | - | instance | `CloseSpecialLightProjectors__9C_VehicleFv` | 0x002335a0 | inlined | inlined into `vehicle_dtor`: the same per-record release of the `+0x18c` projectors (§12) |
| - | - | instance | `CloseVehicle__9C_VehicleFv` | 0x00233a80 | inlined | inlined into `vehicle_dtor`, the one PC function that frees the head tree (§12) |
| 0x0051d330 | `vehicle_colbody_head_dtor` | an `S_COLBODY_HEAD` record | `ReleaseBodyColls__9C_VehicleFP14S_COLBODY_HEAD` | 0x00234110 | STRUCTURAL | frees one head's sphere vector and child-head vector, recursing into every child (§12) |
| 0x0051f7a0 | `vehicle_colbody_head_scalar_deleting_dtor` | an `S_COLBODY_HEAD` record | - | - | - | the head's scalar deleting destructor; `vehicle_dtor` reaches the tree through it |
| 0x0051fcd0 | `crt_static_init_guard_0051fcd0` | the translation unit | - | - | - | the initializer that opens `C_Vehicle_effects.cpp` |
| 0x0051fcf0 | `crt_atexit_register_0051fcf0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x0052c380 | `crt_static_init_guard_0052c380` | the translation unit | - | - | - | the initializer that opens `C_Vehicle_kernel.cpp` |
| 0x0052c3a0 | `crt_atexit_register_0052c3a0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x00537300 | `vehicle_body_coll_sphere_sort_list_static_dtor` | `g_vehicle_body_coll_sphere_sort_list` | - | - | - | `atexit` destructor of the function-local sort vector of `vehicle_body_coll_resolve` (§8d) |
| 0x00537320 | `vehicle_body_coll_head_sort_list_static_dtor` | `g_vehicle_body_coll_head_sort_list` | - | - | - | the same for the head sort vector (§8d) |
| 0x005373b0 | `crt_static_init_guard_005373b0` | the translation unit | - | - | - | the initializer that opens `C_Vehicle_sounds.cpp` |
| 0x005373d0 | `crt_atexit_register_005373d0` | the translation unit | - | - | - | its `atexit` registrar |

### 3b. Accessors

One load or store of one `this` field. The 13 callback setters are the bank of §10.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425440 | `vehicle_get_speed` | - | - | - | returns `+0x618`; 38 callsites: `pursuit_event_tick` 24, `entity_action_tick` 13, `program_process_command` 1 |
| 0x00425450 | `vehicle_get_velocity_ptr` | - | - | - | returns `&+0x1f0c`; `entity_action_tick` 3, `pursuit_event_tick` 2 |
| 0x00425490 | `vehicle_get_right_axis_ptr` | - | - | - | returns `&+0xcc4`; `entity_action_tick`, `pursuit_event_tick` (§1) |
| 0x004f5a30 | `vehicle_get_engine_on` | - | - | - | reads `+0xca8`; `pursuit_event_tick` |
| 0x004f5a40 | `vehicle_set_automatic_gearbox` | - | - | - | writes `+0x548`; `pursuit_event_tick` |
| 0x004f5a50 | `vehicle_set_horn` | - | - | - | writes `+0x4ac`; `pursuit_event_tick` 2 |
| 0x004f5a60 | `vehicle_get_siren` | - | - | - | reads `+0x4ad`; `pursuit_event_tick` |
| 0x004f5aa0 | `vehicle_get_up_axis` | - | - | - | returns `&+0xcd0`; `pursuit_event_tick` 2 |
| 0x004f5ab0 | `vehicle_get_wheel_count` | - | - | - | reads `+0x540`; `pursuit_event_tick` 2, `program_process_command` 3 |
| 0x00518670 | `vehicle_get_head_pos` | `GetHeadPos__9C_VehicleFv` | 0x00240900 | CONFIRMED | returns `&+0x354`; `camera_tick`, `camera_begin_mode` |
| 0x0051ddf0 | `vehicle_set_callback_collision_info` | `SetCallbackACI__9C_VehicleFPFPCvRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vectorPfRb_b` | 0x00247fd0 | CONFIRMED | bank slot `+0x36c` (§10) |
| 0x0051de00 | `vehicle_set_callback_collision` | `SetCallbackCC__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vectorRC8S_vectorf_v` | 0x00247fe0 | CONFIRMED | bank slot `+0x364` (§10) |
| 0x0051de10 | `vehicle_set_callback_collision_quiet` | `SetCallbackCC1__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vectorRC8S_vector_v` | 0x00247ff0 | CONFIRMED | bank slot `+0x368` (§10) |
| 0x0051de20 | `vehicle_set_callback_actor_filter` | `SetCallbackAF__9C_VehicleFPFPCvPCvRb_b` | 0x00248000 | CONFIRMED | bank slot `+0x370` (§10) |
| 0x0051de30 | `vehicle_set_callback_was_collision` | `SetCallbackWC__9C_VehicleFPFPCvRC8S_vectorRC8S_vectorRC8S_vectorfRC8S_vectorPCvPCvb_v` | 0x00248010 | CONFIRMED | bank slot `+0x374` (§10) |
| 0x0051de40 | `vehicle_set_callback_prepare_dropout` | `SetCallbackPD__9C_VehicleFPFPCviRC8S_vectorPC8S_vector_b` | 0x00248020 | CONFIRMED | bank slot `+0x378` (§10) |
| 0x0051de50 | `vehicle_set_callback_prepare_wheel_dropout` | `SetCallbackPDW__9C_VehicleFPFPCviRC8S_vectorPC8S_vector_b` | 0x00248030 | CONFIRMED | bank slot `+0x37c` (§10) |
| 0x0051de60 | `vehicle_set_callback_break_glass` | `SetCallbackBG__9C_VehicleFPFPCviRC8S_vectorPCc_v` | 0x00248040 | CONFIRMED | bank slot `+0x380` (§10); fired by `vehicle_deform_glass` |
| 0x0051de70 | `vehicle_set_callback_wake_up` | `SetCallbackWU__9C_VehicleFPFPCv_v` | 0x00248050 | CONFIRMED | bank slot `+0x388` (§10a); 3 callsites |
| 0x0051de80 | `vehicle_set_callback_body_collision_filter` | `SetCallbackCFB__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vector_PCv` | 0x00248060 | CONFIRMED | bank slot `+0x38c` (§10) |
| 0x0051de90 | `vehicle_set_callback_wheel_collision_filter` | `SetCallbackCFW__9C_VehicleFPFPCvPCvi_PCv` | 0x00248070 | CONFIRMED | bank slot `+0x390` (§10) |
| 0x0051dea0 | `vehicle_set_callback_deform_part` | `SetCallbackDVP__9C_VehicleFPFPCvP9I3D_frameUiRC8S_vectorRC8S_vectorb_v` | 0x00248080 | CONFIRMED | bank slot `+0x394` (§10) |
| 0x0051deb0 | `vehicle_set_callback_repair` | `SetCallbackVR__9C_VehicleFPFPCv_v` | 0x00248090 | CONFIRMED | bank slot `+0x398` (§10); fired by `vehicle_init_deform` |
| 0x005c8780 | `vehicle_get_special_dynamic_colls` | - | - | - | reads `+0x1f26` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425460 | `vehicle_get_seat_count` | - | - | - | `(end - begin) / 0x24` over the seat vector `+0xd1c`, 0 when empty; `entity_action_tick` 4, `program_process_command` 3 (§12) |
| 0x004f5a70 | `vehicle_is_throttle_applied` | - | - | - | `+0x634 > 0.9 && +0x5bc > 0.0`; `pursuit_event_tick` |
| 0x004f5ac0 | `vehicle_get_wheel_at` | - | - | - | indexes `+0xcb4`; `pursuit_event_tick` 2 |
| 0x00509bb0 | `vehicle_init` | `Init__9C_VehicleFP9I3D_frame` | 0x0022daf0 | CONFIRMED | builds the vehicle from the model: the `BODYCOL*`, `WINDOW*`, `BALLAST*` and `SEAT*` frames (§7, §9), the wheels and the size; calls the hook `+0x384` when it is set (§10b) |
| 0x0050b6f0 | `vehicle_init_strip_and_smoke_emitters` | - | - | - | the PS2 build holds this inside `Init` (§12) |
| 0x0050cc10 | `vehicle_init_axes` | `InitAxis__9C_VehicleFv` | 0x00231650 | CONFIRMED | - |
| 0x0050d0c0 | `vehicle_init_lights` | `InitLights__9C_VehicleFv` | 0x00232000 | CONFIRMED | types every lamp frame (§5); calls `vehicle_create_light_projector` 6 times |
| - | - | `InitSpecialLightProjectors__9C_VehicleFv` | 0x002331c0 | inlined | inlined into the tail of `vehicle_init_lights`: the same two frame enumerations (§12) |
| 0x0050e0c0 | `vehicle_create_light_projector` | `CreateLightProjector__9C_VehicleFP9I3D_framePP9I3D_light` | 0x002341a0 | STRUCTURAL | the PS2 body is an 8-byte stub (§12) |
| 0x0050e670 | `vehicle_create_special_light_projector` | `CreateSpecialLightProjector__9C_VehicleFP9I3D_framePP9I3D_light` | 0x002341b0 | CONFIRMED | - |
| 0x0050ee60 | `vehicle_deactivate` | `Deactivate__9C_VehicleFUi` | 0x00234860 | CONFIRMED | also holds the dynamic-collision erase of `SetDynamicCollsOn` (§12) |
| 0x0050f140 | `vehicle_reset_from_frame` | `Reset1__9C_VehicleFv` | 0x00234ca0 | CONFIRMED | seeds the basis (§1), rebuilds the wheels' world positions, fills the matrix cache `+0xd64` / `+0xda4` |
| 0x0050f7f0 | `vehicle_reset` | `Reset__9C_VehicleFfb` | 0x002351d0 | CONFIRMED | the respawn reset |
| 0x00510150 | `vehicle_game_init` | `GameInit__9C_VehicleFv` | 0x00235b70 | CONFIRMED | zeroes the drivetrain state, sets `+0x548` = 1 and `+0x664` = `0x20`, clears the `add_track` vectors of `+0x528` |
| 0x00511890 | `vehicle_game_done` | `GameDone__9C_VehicleFv` | 0x002383d0 | CONFIRMED | `car_game_done` calls it between `vehicle_enable_sounds(0)` and `actor_game_done` |
| 0x00511b50 | `vehicle_set_tyres` | `SetTyres__9C_VehicleFPCQ23std42vector<6S_TYRE,Q23std18allocator<6S_TYRE>>` | 0x00238820 | CONFIRMED | fills the tyre list (§8a) |
| 0x00511eb0 | `vehicle_close_tyres` | `CloseTyres__9C_VehicleFv` | 0x00238a10 | CONFIRMED | - |
| 0x00511f40 | `vehicle_set_materials` | `SetVehicleMaterials__9C_VehicleFPCQ23std60vector<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>>` | 0x00238aa0 | CONFIRMED | fills the material list (§8a) |
| 0x00511fb0 | `vehicle_close_materials` | `CloseVehicleMaterials__9C_VehicleFv` | 0x00238ba0 | CONFIRMED | - |
| 0x00511ff0 | `vehicle_set_particles` | `AddParticle__9C_VehicleFRC14S_VEH_PARTICLE` | 0x00239300 | STRUCTURAL | one function for `CloseParticles` + N x `AddParticle` (§12) |
| 0x005122b0 | `vehicle_close_particles` | `CloseParticles__9C_VehicleFv` | 0x00239680 | CONFIRMED | - |
| 0x00512340 | `vehicle_get_wheel_cam_pos` | `GetWheelCamPos__9C_VehicleFiRC8S_vector` | 0x00239930 | CONFIRMED | - |
| 0x005123d0 | `vehicle_apply_carinit_params` | `InitCar__9C_VehicleFP9S_CARINITUi` | 0x002399f0 | CONFIRMED | imports the CARINIT record: wheels, gear tables (§1), steering (§6), damage pools, revs |
| 0x00517370 | `vehicle_get_carinit_params` | `GetInit__9C_VehicleFP9S_CARINIT` | 0x0023f380 | CONFIRMED | copies `S_CARINIT` out of `+0xde8` |
| 0x005173a0 | `vehicle_set_gear` | `SetGear__9C_VehicleFi` | 0x0023f3c0 | CONFIRMED | `+0x5dc` in `-1..gear_count`; refused into reverse while `+0x138` = 1, with an empty gearbox pool under damage (`+0xce4` bit `0x4000`) and when the gear would over-rev; clears `+0x510` |
| 0x00517460 | `vehicle_set_power` | `SetPower__9C_VehicleFfb` | 0x0023f4e0 | CONFIRMED | the accelerator (§4) |
| 0x00517600 | `vehicle_set_brake` | `SetBrake__9C_VehicleFf` | 0x0023f650 | CONFIRMED | the brake pedal (§4) |
| 0x00517790 | `vehicle_set_clutch` | `SetClutch__9C_VehicleFf` | 0x0023f7b0 | CONFIRMED | `+0x65c`, shaped by `+0x21c` while `+0x527` is set |
| 0x00517800 | `vehicle_set_steer` | `SetSteer__9C_VehicleFf` | 0x0023f830 | CONFIRMED | `+0x6a0 = angle * +0x68c` |
| 0x005178a0 | `vehicle_set_steering_linearity` | `SetSteeringLinearity__9C_VehicleFf` | 0x0023f8e0 | CONFIRMED | `+0x218` |
| 0x005178e0 | `vehicle_set_engine_on` | `SetEngineOn__9C_VehicleFbb` | 0x0023f930 | CONFIRMED | ignition `+0xca8`; refused at fuel 0.02 or below and, under damage, with an empty engine pool; 28 callsites |
| 0x005179d0 | `vehicle_set_speed_limit` | `SetSpeedLimit__9C_VehicleFf` | 0x0023fac0 | CONFIRMED | `+0x520` |
| 0x00517a10 | `vehicle_set_fuel` | `SetFuel__9C_VehicleFf` | 0x0023fb00 | CONFIRMED | `+0xcac`, clamped against `+0x1f1c` |
| 0x00517a40 | `vehicle_set_handbrake` | `SetHandbrake__9C_VehicleFbb` | 0x0023fb30 | CONFIRMED | `+0x4b0` through the ramp (§4) |
| 0x00517ab0 | `vehicle_enable_sounds` | `EnableSounds__9C_VehicleFb` | 0x0023fb90 | CONFIRMED | `+0x4ae`; calls `vehicle_stop_all_sounds` when cleared |
| 0x00517ae0 | `vehicle_set_cut_time` | `SetCutTime__9C_VehicleFi` | 0x0023fbe0 | CONFIRMED | `+0xca0 = ms / 1000` for 5..50 ms, refused otherwise; `traffic_car_game_init` passes 42 |
| 0x00517b10 | `vehicle_apply_external_impulse` | `ExternalImpuls__9C_VehicleFRC8S_vectorRC8S_vectorUiRC8S_vector` | 0x0023fc30 | CONFIRMED | fires the wake-up slot `+0x388` |
| 0x00517f00 | `vehicle_apply_external_force` | `ExternalForce__9C_VehicleFRC8S_vectorRC8S_vectorUiRC8S_vector` | 0x0023fea0 | CONFIRMED | fires `+0x388` |
| 0x005180a0 | `vehicle_do_collision` | `DoCollision__9C_VehicleFPCvfRC8S_vectorRC8S_vectorRC8S_vectorf` | 0x0023ffa0 | CONFIRMED | - |
| 0x00518500 | `vehicle_get_contact_impulse_terms` | `GetRVelMassJ__9C_VehicleFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vector` | 0x002407f0 | CONFIRMED | centre of mass `+0x414`, angular velocity `+0x49c`, velocity `+0x1f0c` |
| 0x00518680 | `vehicle_get_max_engine_torque_at_rpm` | `_GetMaxMomentByRot__9C_VehicleFfPfib` | 0x00240910 | CONFIRMED | reads the gear table (§1) and the CARINIT rpm fields `+0xdf8` / `+0xdfc` |
| 0x00518730 | `vehicle_set_seat_mass` | `SetSeatMass__9C_VehicleFif` | 0x00240a30 | CONFIRMED | one seat's occupant mass, then the centre of mass (§9); 10 callsites |
| - | - | `ComputeMass__9C_VehicleFii` | 0x00240ad0 | inlined | spliced into `vehicle_set_seat_mass` (§9) |
| 0x00518a90 | `vehicle_encode_position` | `CodePos__9C_VehicleFiRC8S_vector` | 0x00240ed0 | CONFIRMED | quantises a position against `+0x1e4` (range) and `+0x1fc` (origin) |
| 0x00518b40 | `vehicle_decode_position` | `DecodePos__9C_VehicleFiUi` | 0x00241080 | CONFIRMED | the inverse |
| 0x00518c10 | `vehicle_get_part_name_tag` | `CodeIndexDan__9C_VehicleFi` | 0x002411e0 | CONFIRMED | over the deform-zone vector `+0x294` |
| 0x00518d10 | `vehicle_find_part_by_name_tag` | `DecodeIndexDan__9C_VehicleFUi` | 0x00241310 | CONFIRMED | the inverse |
| 0x00518d70 | `vehicle_encode_part_position` | - | - | - | quantises one body part's world position, walking its frame chain up to `+0x3c8` |
| 0x00518f70 | `vehicle_find_part_at_encoded_position` | - | - | - | the inverse: the nearest matching body part |
| 0x005195e0 | `vehicle_get_engine_rpm` | `GetRotations__9C_VehicleFv` | 0x002413a0 | CONFIRMED | `+0xca8 ? +0x5c4 * 60 : 0`; `player_ai` (the HUD tachometer) and `player_ai_drive` |
| 0x00519600 | `vehicle_is_unstable` | `IsVehicleUnstable__9C_VehicleFv` | 0x002413d0 | CONFIRMED | bit 3 of `+0x224` |
| 0x00519610 | `vehicle_get_size` | `GetVehicleSize__9C_VehicleFb` | 0x002413f0 | CONFIRMED | `&+0x1d8` or `&+0x1e4` |
| 0x00519630 | `vehicle_get_pivot_pos` | `GetVehiclePivotPos__9C_VehicleFb` | 0x00241410 | CONFIRMED | `&+0x1f0` or `&+0x1fc` |
| 0x00519650 | `vehicle_lock` | `LockVehicle__9C_VehicleFb` | 0x00241430 | CONFIRMED | `+0x140` +/-1 and bit `0x10000000` of `+0x224` |
| 0x00519690 | `vehicle_set_projector_size` | `SetProjectorSize__9C_VehicleFUiRC8S_vector` | 0x00241490 | CONFIRMED | walks the light vector `+0x17c` for six selector bits |
| 0x00519a70 | `vehicle_open_door` | `OpenDoor__9C_VehicleFifb` | 0x00241930 | CONFIRMED | sets a door's target angle (§7); fires `+0x388` |
| 0x00519b10 | `vehicle_set_max_door_angle` | `SetMaxDoorAngle__9C_VehicleFif` | 0x00241ad0 | CONFIRMED | door `+0x28` |
| 0x00519b30 | `vehicle_set_door_open_speed` | `SetSpeedDoorOpen__9C_VehicleFif` | 0x00241b00 | CONFIRMED | door `+0x2c` |
| 0x00519b60 | `vehicle_set_special_dynamic_colls` | `SetSpecialDynamicColls__9C_VehicleFb` | 0x00241b40 | CONFIRMED | `+0x1f26` |
| 0x00519d30 | `vehicle_set_dynamic_colls_off` | `SetDynamicCollsOn__9C_VehicleFbb` | 0x00241de0 | CONFIRMED | inserts or erases the dynamic-collision records (§12); 16 callsites |
| 0x00519e50 | `vehicle_update_cached_transform` | `UpdateImportantVariables__9C_VehicleFv` | 0x002420d0 | CONFIRMED | refreshes the matrix cache, then copies its rows into the basis (§1) |
| 0x0051a150 | `vehicle_clear_body_coll_add_track` | `ClearBodyCollAddTrack__9C_VehicleFP14S_COLBODY_HEAD` | 0x00242420 | CONFIRMED | `this` is a head vector: zeroes `add_track` recursively (§7) |
| 0x0051a250 | `vehicle_add_projector_lights_to_sectors` | `AddProjectorLightsToSectors__9C_VehicleFv` | 0x002424c0 | CONFIRMED | `I3D_sector` slot `0x50` per light |
| 0x0051a3b0 | `vehicle_remove_projector_lights_from_sectors` | `DeleteProjectorLightsFromSectors__9C_VehicleFv` | 0x002425f0 | CONFIRMED | slot `0x54` |
| 0x0051a520 | `vehicle_register_projector_light_sector` | `AddSectorForProjectorLights__9C_VehicleFP10I3D_sector` | 0x002426f0 | CONFIRMED | push-back-if-absent onto the sector list (§8a) |
| 0x0051a720 | `vehicle_register_sound_sector` | `AddSectorForSounds__9C_VehicleFP10I3D_sector` | 0x00242790 | CONFIRMED | the same over the sound sector list |
| 0x0051a920 | `vehicle_settle` | `CalmVehicle__9C_VehicleFUifi` | 0x00242830 | CONFIRMED | runs `vehicle_tick_drivetrain` with `master_flags` = 1 until bit `0x2` is set or the time is up (§2b), zeroes the velocity, the speed and the angular velocity `+0x49c`, then runs `vehicle_update_visuals`; fires `+0x388` twice |
| 0x0051aac0 | `vehicle_get_save_state_size` | `GetVehicleStateSize__9C_VehicleFv` | 0x00243540 | CONFIRMED | the vehicle savegame block size |
| - | - | `GetVehicleDamageDeformSize__9C_VehicleFv` | 0x00245290 | inlined | folded into `vehicle_get_save_state_size` (§12) |
| 0x0051ad90 | `vehicle_save_state` | `SaveVehicleState__9C_VehicleFPv` | 0x002435d0 | CONFIRMED | drivetrain, damage, lamp and wheel state |
| - | - | `SaveVehicleDamageDeform__9C_VehicleFPv` | 0x00245650 | inlined | folded into `vehicle_save_state` (§12) |
| 0x0051ba40 | `vehicle_load_state` | `LoadVehicleState__9C_VehicleFPv` | 0x00243db0 | CONFIRMED | the inverse |
| - | - | `LoadVehicleDamageDeform__9C_VehicleFPv` | 0x00245f70 | inlined | folded into `vehicle_load_state` (§12) |
| 0x0051d710 | `vehicle_prepare_body_colls` | `PrepareBodyColls__9C_VehicleFP14S_COLBODY_HEAD` | 0x002473e0 | CONFIRMED | `this` is a head: bakes its spheres into world space, recursively |
| 0x0051fd00 | `vehicle_emit_wheel_skid_segment` | `Slaif__9C_VehicleFi` | 0x0024a710 | CONFIRMED | opens a skid-ring slot on one wheel (§7) |
| 0x005214e0 | `vehicle_end_wheel_skid_segment` | `SlaifEnd__9C_VehicleFi` | 0x0024c000 | CONFIRMED | closes it and advances the wheel's cursor (§7) |
| 0x00521580 | `vehicle_prepare_deform_zone` | `PrepareDeformZone__9C_VehicleFv` | 0x0024c080 | CONFIRMED | clones each zone mesh into `+0x284` (§7) |
| 0x005219a0 | `vehicle_init_deform_glass_texture` | `InitDeformGlassTexture__9C_VehicleFPCcPCc` | 0x0024c3a0 | CONFIRMED | the cracked-glass textures on every type-3 zone |
| 0x00521a40 | `vehicle_init_deform` | `InitDeform__9C_VehicleFb` | 0x0024c4a0 | CONFIRMED | restores the pristine vertices (repair); fires `+0x398` |
| 0x00521d30 | `vehicle_init_damage` | `InitDamage__9C_VehicleFv` | 0x0024cbc0 | CONFIRMED | refills both pools, `+0x230` = 1.0, every wheel and lamp health |
| 0x00521e60 | `vehicle_deform` | `Deform__9C_VehicleFRC8S_vectorRC8S_vectorffUiPC8S_vector` | 0x0024d960 | CONFIRMED | the impact entry point (§8c) |
| 0x00522dd0 | `vehicle_switch_special_projector_texture` | `SwitchSpecialProjectorTexture__9C_VehicleFP9I3D_frameUs` | 0x0024f830 | CONFIRMED | - |
| 0x00522ec0 | `vehicle_deform_glass` | `DeformGlass__9C_VehicleFiRC8S_vector` | 0x0024f970 | CONFIRMED | fires `+0x380` |
| 0x00522f70 | `vehicle_apply_damage` | `DamageVehicle__9C_VehicleFf` | 0x0024fb50 | CONFIRMED | drains the two pools (§2) |
| 0x00523090 | `vehicle_update_axis` | `UpdateAxis__9C_VehicleFv` | 0x0024fcb0 | CONFIRMED | called twice per frame on both builds |
| 0x00523fa0 | `vehicle_animate_doors` | `DoDoors__9C_VehicleFf` | 0x00250680 | CONFIRMED | moves each door toward its target (§7) |
| 0x005242d0 | `vehicle_open_window` | `OpenWindow__9C_VehicleFib` | 0x00250a50 | CONFIRMED | - |
| 0x00524320 | `vehicle_animate_windows` | `DoWindows__9C_VehicleFf` | 0x00250ab0 | CONFIRMED | - |
| 0x005244d0 | `vehicle_is_window_open` | `IsWindowOpen__9C_VehicleFi` | 0x00250df0 | CONFIRMED | - |
| 0x00524520 | `vehicle_apply_light_state` | `DoLights__9C_VehicleFf` | 0x00250e90 | CONFIRMED | the per-frame lamp update (§5) |
| 0x00528b80 | `vehicle_update_strips` | `UpdateStrips__9C_VehicleFf` | 0x00256270 | CONFIRMED | - |
| 0x0052a1b0 | `vehicle_update_motor_particles` | `UpdateMotorParticles__9C_VehicleFv` | 0x00256c10 | CONFIRMED | - |
| 0x0052a670 | `vehicle_update_steering_wheels` | `UpdateSteeringWheels__9C_VehicleFf` | 0x002570a0 | CONFIRMED | the steering geometry (§6) |
| 0x0052aa30 | `vehicle_update_rotation_wheels` | `UpdateRotationWheels__9C_VehicleFf` | 0x00257770 | CONFIRMED | - |
| 0x0052af10 | `vehicle_update_wheel_particles` | `UpdateWheelParticles__9C_VehicleFv` | 0x00258120 | CONFIRMED | reads the material-id table (§8c) |
| 0x0052b8a0 | `vehicle_update_exhaust_particles` | `UpdateExhaustParticles__9C_VehicleFv` | 0x00258960 | CONFIRMED | - |
| 0x0052bf20 | `vehicle_emit_body_collision_particles` | `UpdateBodyParticles__9C_VehicleFv` | 0x00258fa0 | CONFIRMED | one impact particle per contact, called from `vehicle_body_coll_resolve` (§12) |
| 0x0052c3b0 | `vehicle_tick_drivetrain` | `Tick__9C_VehicleFUi` | 0x002594c0 | CONFIRMED | the physics half of the frame, incl. the automatic gearbox |
| 0x0052d480 | `vehicle_update_visuals` | `Update__9C_VehicleFUi` | 0x0025b4c0 | CONFIRMED | the visual and audio half (§8c) |
| 0x0052dfa0 | `vehicle_compute_motor_force` | `MotorForce__9C_VehicleFff` | 0x0025d530 | CONFIRMED | the drive force, scaled by `+0x230` |
| 0x0052e220 | `vehicle_compute_moment` | `ComputeMoment__9C_VehicleFff` | 0x0025d8c0 | CONFIRMED | - |
| 0x0052e3c0 | `vehicle_update_motor_revs` | `MotorRot__9C_VehicleFfff` | 0x0025da60 | CONFIRMED | the revs `+0x5c4`, clamped to `+0x1f18` |
| 0x0052e6d0 | `vehicle_substep_move` | `Engine__9C_VehicleFfff` | 0x0025dff0 | CONFIRMED | splits the frame into sub-steps of `+0xca0` (§8d) |
| 0x0052ea80 | `vehicle_move` | `Move__9C_VehicleFffffff` | 0x0025e390 | CONFIRMED | the integrator |
| - | - | `GetForwardForceMax__9C_VehicleFv` | 0x00242c30 | inlined | folded into the wheel loop of `vehicle_move` (`+0x1c8`, §12) |
| - | - | `GetSideForceMax__9C_VehicleFv` | 0x00242c80 | inlined | folded into the same loop (`+0x1c4`, §12) |
| 0x005334e0 | `vehicle_wheel_contact_solve` | `DoWheelsCollision__9C_VehicleFfRC8S_vectorb` | 0x00262850 | CONFIRMED | the wheel solver |
| 0x00535610 | `vehicle_body_coll_resolve` | `BodyCollision__9C_VehicleFP14S_COLBODY_HEADR8S_vectorR8S_vectorRfR8S_vectorR8S_vectorUi` | 0x00264de0 | CONFIRMED | the body-collision solver; `this` is the `C_Vehicle` (ECX is kept in `ebp`), the head vector is the first stack argument, and the recursion passes the child vector `head + 0x10` (§12) |
| 0x005373e0 | `vehicle_init_sounds` | `InitSounds__9C_VehicleFP9S_CARINIT` | 0x00267870 | CONFIRMED | opens the sound bank from the CARINIT names (§8e) |
| 0x00537880 | `vehicle_add_sounds_to_sectors` | `AddSoundsToSectors__9C_VehicleFv` | 0x00267dc0 | CONFIRMED | `I3D_sector` slot `0x58` per sound |
| 0x00537d50 | `vehicle_remove_sounds_from_sectors` | `DeleteSoundsFromSectors__9C_VehicleFv` | 0x00268720 | CONFIRMED | slot `0x5c` |
| 0x00538220 | `vehicle_update_engine_sound` | `EngineSnd__9C_VehicleFb` | 0x00269080 | CONFIRMED | - |
| 0x005385b0 | `vehicle_update_engine_pitch` | `EngineFreq__9C_VehicleFf` | 0x00269570 | CONFIRMED | fades the engine in over 0.3 s (`+0xc9c`) |
| 0x00539550 | `vehicle_update_environment_sound` | `EnvironmentalSnd__9C_VehicleFv` | 0x0026ac20 | CONFIRMED | - |
| 0x005398d0 | `vehicle_update_flat_tyre_sound` | `CrashTyreSnd__9C_VehicleFb` | 0x0026b080 | CONFIRMED | level from the speed along `+0xcb8` (§8e) |
| 0x00539c60 | `vehicle_update_skid_sound` | `SkidSnd__9C_VehicleFf` | 0x0026b4e0 | CONFIRMED | - |
| 0x00539f80 | `vehicle_play_gear_change_sound` | `GearSnd__9C_VehicleFv` | 0x0026b900 | CONFIRMED | - |
| 0x0053a080 | `vehicle_play_handbrake_sound` | `HandBrake__9C_VehicleFv` | 0x0026baa0 | CONFIRMED | one-shot latched by `+0x1d4` |
| 0x0053a120 | `vehicle_play_engine_start_fail_sound` | `EngineSndNFS__9C_VehicleFv` | 0x0026bba0 | CONFIRMED | - |
| 0x0053a1b0 | `vehicle_play_horn_sound` | `HornSnd__9C_VehicleFb` | 0x0026bca0 | CONFIRMED | - |
| 0x0053a280 | `vehicle_play_siren_sound` | `SirenSnd__9C_VehicleFb` | 0x0026bdf0 | CONFIRMED | - |
| 0x0053a350 | `vehicle_play_tyre_burst_sound` | `TyreBreakSnd__9C_VehicleFi` | 0x0026bf40 | CONFIRMED | - |
| 0x0053a410 | `vehicle_play_crash_sound` | `CrashSnd__9C_VehicleFfb` | 0x0026c040 | CONFIRMED | - |
| 0x0053b0c0 | `vehicle_play_wheel_sound` | `WheelSnd__9C_VehicleFf` | 0x0026d0e0 | CONFIRMED | - |
| 0x0053b120 | `vehicle_play_door_sound` | `DoorSnd__9C_VehicleFib` | 0x0026d260 | CONFIRMED | - |
| 0x0053b2c0 | `vehicle_stop_motor_sounds` | `MotorSoundOff__9C_VehicleFb` | 0x0026d4f0 | CONFIRMED | - |
| 0x0053b3f0 | `vehicle_stop_all_sounds` | `SoundOff__9C_VehicleFv` | 0x0026d720 | CONFIRMED | - |
| 0x0053b850 | `vehicle_open_sound` | `OpenSnd__9C_VehicleFP9I3D_soundPCcUi` | 0x0026de10 | CONFIRMED | vtable slot 0: forwards to `i3d_cache_sound_open` |
| 0x0053b880 | `vehicle_sound_slot_init` | `SoundInit__9C_VehicleFPcPP9I3D_soundUiffffbb` | 0x0026e2a0 | CONFIRMED | opens one sound slot; defaults 4.0 / 50.0 / 0.3 (§8e) |
| 0x0053bbf0 | `vehicle_release_sounds` | `ReleaseSounds__9C_VehicleFv` | 0x0026e800 | CONFIRMED | - |
| 0x005c8790 | `vehicle_set_velocity` | - | - | - | writes `+0x1f0c`; fires `+0x388` |
| 0x005c8840 | `vehicle_set_angular_velocity` | - | - | - | writes `+0x49c`; fires `+0x388` |
| 0x005c8880 | `vehicle_set_auto_indicators_from_steering` | - | - | - | bit `0x8` of `+0x19c`; the one light setter that does not notify (§5) |
| 0x005c88b0 | `vehicle_set_emergency_lights` | - | - | - | bit `0x80` + notify; `program_process_command` 3 (§5) |
| 0x005c88e0 | `vehicle_set_left_indicator` | - | - | - | bit `0x1`, only while bit `0x8` is clear; notifies only when it sets the bit |
| 0x005c8920 | `vehicle_set_right_indicator` | - | - | - | bit `0x2`, the same gate; notifies only when it sets the bit |
| 0x005c8960 | `vehicle_set_headlights` | - | - | - | bit `0x10000` + notify |
| 0x005c89a0 | `vehicle_set_light_script_override` | - | - | - | writes the `0x0ff00000` sub-field (§2a) |
| 0x005c89e0 | `vehicle_get_light_script_override` | - | - | - | reads it |

### 3d. Free helpers

No `C_Vehicle` receiver: the frame-enumeration callbacks the model walks call (free functions on both builds), and the container helpers of the class's vectors, whose `this` is the vector. The `std_vector*` helpers are shared templates that the class's methods call.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0051ccf0 | `vehicle_cb_enum_objects` | `CbEnumObj__FP9I3D_frameUi` | 0x00246cb0 | CONFIRMED | appends one node per frame to a linked list (`+0x1d0` next) |
| 0x0051cdc0 | `vehicle_cb_enum_first_object` | `CbEnumObj1__FP9I3D_frameUi` | 0x00246de0 | CONFIRMED | stores the first frame |
| 0x0051cdd0 | `vehicle_cb_enum_object_list` | `CbEnumObj6__FP9I3D_frameUi` | 0x00246df0 | CONFIRMED | pushes each frame onto a vector |
| 0x0051cf80 | `vehicle_cb_enum_body_colls` | `CbEnumBodyColl__FP9I3D_frameUi` | 0x00246f10 | CONFIRMED | builds the `+0x528` tree |
| 0x0051d280 | `vehicle_body_coll_attach_to_parent` | `IsParent__FP14S_COLBODY_HEADP14S_COLBODY_HEAD` | 0x00246e40 | CONFIRMED | - |
| 0x0051d3b0 | `vehicle_collect_coll_spheres` | `EnumColSphere__FP9S_COL_POM` | 0x002471b0 | CONFIRMED | - |
| 0x0051d440 | `vehicle_cb_enum_coll_sphere` | `CbEnumCollsActiveSphere__FP9I3D_frameUi` | 0x00247240 | CONFIRMED | - |
| 0x0051d9a0 | `vehicle_cb_enum_deform_zones` | `CbEnumDeform__FP9I3D_frameUi` | 0x00247670 | CONFIRMED | builds the deform-zone vector (§7) |
| 0x0051dec0 | `vehicle_colbody_head_vector_destroy` | - | - | - | container helper of the head vector |
| 0x0051df00 | `std_vector18_insert_at` | - | - | - | shared template |
| 0x0051e100 | `vehicle_window_list_insert_at` | - | - | - | container helper of the window vector `+0xd2c` |
| 0x0051e310 | `vehicle_window_list_size` | - | - | - | the same |
| 0x0051e340 | `vehicle_colbody_head_vector_insert_n` | - | - | - | container helper of the head vector |
| 0x0051e6d0 | `std_vector20_insert_n` | - | - | - | shared template |
| 0x0051e970 | `std_vector24_insert_n` | - | - | - | shared template; `vehicle_init` inserts the seats with it |
| 0x0051ec70 | `std_vector1c_insert_n` | - | - | - | shared template; `vehicle_init` inserts the ballast points with it |
| 0x0051ef90 | `vehicle_special_coll_list_insert_n` | - | - | - | container helper of `+0x1f38` |
| 0x0051f290 | `std_vector34_uninit_copy` | - | - | - | shared template |
| 0x0051f2d0 | `std_vector34_uninit_fill_n` | - | - | - | shared template |
| 0x0051f300 | `std_vector1c_copy_range` | - | - | - | shared template |
| 0x0051f330 | `vehicle_special_coll_list_copy_range` | - | - | - | container helper of `+0x1f38` |
| 0x0051f360 | `vehicle_colbody_head_copy` | - | - | - | copies one head |
| 0x0051f520 | `std_vector20_construct_copy` | - | - | - | shared template |
| 0x0051f540 | `vehicle_deform_zone_copy` | - | - | - | copies one deform zone |
| 0x0051f560 | `vehicle_colbody_sphere_vector_copy_assign` | - | - | - | container helper |
| 0x0051f7c0 | `vehicle_coll_sphere_list_copy_assign` | - | - | - | container helper |
| 0x0051fa20 | `vehicle_colbody_head_vector_copy_assign` | - | - | - | container helper |
| 0x0051fca0 | `vehicle_coll_sphere_list_copy_range` | - | - | - | container helper |
| 0x0052c350 | `vehicle_light_at` | - | - | - | indexes the light vector `+0x17c` (`this` = the vector); 301 callsites |
| 0x0052c360 | `vehicle_special_projector_at` | - | - | - | indexes `+0x18c` the same way; 51 callsites |
| 0x00537340 | `std_vector74_size` | - | - | - | shared template; the head vector's size |
| 0x00537370 | `std_vector8_erase_range` | - | - | - | shared template |
| 0x0053b840 | `vehicle_set_global_sound_volume` | - | - | - | writes `g_vehicle_sound_volume_scale` (§8e), its only writer |

## 4. Power and brake delivery

* **Manual gearbox (`+0x548` = 0).** `vehicle_set_power` drives the throttle `+0x420` and
  `vehicle_set_brake` the brake force `+0x634`, in every gear. A brake above 0 also cuts both throttle
  channels.
* **Automatic gearbox (`+0x548` != 0).** In a forward gear the two setters do the same; in reverse
  (`gear < 0`) they swap: the accelerator writes `+0x634` and the brake drives `+0x420`, so in reverse
  the brake pedal drives the engine and the accelerator brakes. Holding the brake above 0.3 at
  standstill and idle revs makes the automatic gearbox select reverse after `+0x510` passes 0.002,
  and the accelerator selects first the same way.
* **The ramp.** Unless `+0x526` is set, both writes of `+0x634` go through
  `(pedal_ramp_accum / pedal_ramp_max) * pedal`, the accumulator growing by `+0x544` per call. The
  brake lamps follow `+0x634` (§5). `driver_tick` calls `vehicle_set_power` and then
  `vehicle_set_brake`; in either gear state the two calls write different fields. A pedal of 0 means
  cut and coast.
* **The parking brake is `vehicle_set_handbrake`** (`+0x4b0`, its own ramp), applied only to the
  handbrake wheels (§6). The chase AI's `driver + 0xdc` is its brake pedal and feeds
  `vehicle_set_brake` and the brake light (`driver_do_lights`).
* **Boosting is a `C_car` model-level swap, not a `C_Vehicle` field**:
  `police-system/artifacts/boosting.md`.

## 5. Lamps: types, blink phase, light bar and the automatic bits

`vehicle_init_lights` gives every lamp frame a type from its name prefix, and
`vehicle_apply_light_state` switches on that type (`type - 1` indexes its jump table). Each arm
gates its lamp's two light objects (`+0x08` / `+0x0c`) by one script-override bit of the
`0x0ff00000` sub-field (§2a); the glow object `+0x04` follows the lamp bit, the broken bit and
`feature_flags` `0x80`:

| Type | Frame prefix | Lamp bit | Override bit |
|---|---|---|---|
| 1 / 2 | `LIGHTIN*` | `0x1` / `0x2`, in the blink phase | `0x100000` |
| 3 / 4 | `LIGHTBI*` | `0x1` / `0x2` in the blink phase while that indicator is on, else `0x100` | `0x400000` |
| 5 / 6 | `LIGHTLI*` | `0x10` / `0x20` | `0x200000` |
| 7 | `LIGHTRE*` | `0x10000` | `0x800000` |
| 8 | `LIGHTBR*` | `0x100` | `0x1000000` |
| 9 | `LIGHTBA*` | `0x1000` | `0x2000000` |

**The blink phase** is bit `0x10000000`: `vehicle_apply_light_state` adds the frame time to `+0x1ac`
and toggles the bit each time the sum passes `+0x1a8` (while it is clear, 0.3 s) or `+0x1a4` (while
it is set, 0.4 s). An indicator lamp is lit only while its side bit and the phase bit are both set,
so types 3 / 4 serve as the rear indicator and the brake lamp of their side at once.

**The light bar.** While bit `0x80` is set, `vehicle_apply_light_state` adds the frame time to
`+0x1b8`; each time it passes the period `+0x1b4` (0.3 s) the timer restarts and bit `0x20000000`
toggles, and that phase lights `0x20` or `0x10` and clears the other. When `0x80` is clear and the
previous word `+0x1a0` had it, both phase bits are cleared. `LIGHTLI*` frames ship only on `fire00` /
`fire10`, `polimFor00` / `polimFor10` and `polimTud00` / `polimTud10`. The bit is set by
`vehicle_set_emergency_lights` and, inline, by `police_manager_deploy_vehicle_from_pool`,
`police_manager_acquire_from_pool_by_name`,
`police_manager_spawn_patrol_car_reinforcements_at_radius` and
`police_manager_try_dispatch_pursuit_for_target`; `car_unpack_light_state` restores it.

**The automatic bits.** `0x8` (indicators from the steering), `0x800` (brake lights from
`+0x634 > 0`) and `0x8000` (reverse lights from `+0x5d8 < 0`) are set together by `car_game_init`,
`human_intern_use_car` and `human_put_in_vehicle_at_seat`. `driver_tick` sets `0x800` on its own,
and `driver_reset` clears `0x8` and `0x800` and sets `0x8000`.

**The notify hook.** Every light setter that changes a lamp calls the wake-up slot `+0x388` when it
is non-null; `vehicle_set_auto_indicators_from_steering` is the one that does not (§10a).

## 6. Steering geometry and the wheel role bits

`vehicle_apply_carinit_params` converts the seven CARINIT steering angles from degrees
(`def-files-docs/vehicles-bin.md` §4a), and `vehicle_update_steering_wheels` uses them every frame
with the speed factor `t = max(0, 1 - +0x618 / +0x404)`.

| Offset | Field | Source | Use |
|---|---|---|---|
| `+0x404` | top speed | `vehicle_apply_carinit_params`: the drive circumference `+0x628` over the top-gear ratio, times the maximum revs `+0x1f18` | the speed factor `t` |
| `+0x684` / `+0x688` | lock at standstill / at top speed | CARINIT `+0x0EC` / `+0x0F0` | `+0x68c = +0x688 + (+0x684 - +0x688) * t` |
| `+0x690` / `+0x698` | turn-in rate at standstill / at top speed, per second | CARINIT `+0x0F8` / `+0x100` | while the angle moves away from the centre |
| `+0x694` / `+0x69c` | return rate at standstill / at top speed, per second | CARINIT `+0x0FC` / `+0x104` | while it moves back toward the centre or changes sign |
| `+0x6a4` | current angle | `vehicle_update_steering_wheels` | moves toward `+0x6a0` by rate x frame time; set straight to it while `+0x524` is set |
| `+0x6a8` / `+0x6b0` | split at full lock / current split | CARINIT `+0x0F4`; `+0x6b0 = abs(+0x6a4) / +0x684 * +0x6a8` | added to one steered wheel's angle and subtracted from the other's |

**The wheel role bits.** Three bits of the wheel flag word `+0x120` come from the bytes of the CARINIT
axle record (`def-files-docs/vehicles-bin.md` §4b):

| Bit | Meaning | Reader |
|---|---|---|
| `0x1` | steered | `vehicle_update_steering_wheels`, `vehicle_reset`, `vehicle_update_visuals` |
| `0x2` | driven | `vehicle_apply_carinit_params`: only driven wheels set the drive circumference `+0x628 = 2 pi r` |
| `0x4` | handbrake | `vehicle_move` applies the handbrake output `+0x4b0` only to these wheels |

**The runtime bits** of the same word:

| Bit | Meaning | Reader |
|---|---|---|
| `0x8` | on the ground; cleared when the suspension load turns negative | `vehicle_tick_drivetrain`, `vehicle_move`, `vehicle_update_wheel_particles`, `vehicle_update_rotation_wheels` |
| `0x10` | skidding: slip above `+0x170`, a locked wheel above 1 m/s, or a spinning driven wheel | `vehicle_tick_drivetrain`, `vehicle_move` |
| `0x20` | locked: the brake or handbrake beyond its grip | `vehicle_move`, `vehicle_update_visuals` |
| `0x40` | on the bump stop: set once the hub reaches `+0x14`, until it drops below it | `vehicle_wheel_contact_solve` |
| `0x80` | spinning under drive; also sets `+0x5cc` | `vehicle_move`, `vehicle_update_rotation_wheels` |
| `0x100` | sliding: slip sine above 0.6 above 15 m/s | `vehicle_move` |
| `0x200` | strip-driven, from a kind-`0x1` strip element | `vehicle_apply_carinit_params`, `vehicle_move` |
| `0x10000` | the contact velocity `+0x4c` along the rolling direction `+0x58` is 0 or more; clear, the forces are negated | `vehicle_wheel_contact_solve` |
| `0x10000000` | damaged since the last `vehicle_update_visuals` pass | `vehicle_deform`, `vehicle_update_visuals` |
| `0x20000000` | the contact was kept by the standstill path; cleared at the end of every wheel's pass | `vehicle_wheel_contact_solve` |
| `0x80000000` | the burst tyre: the radius `+0x110` and the second grip set (wheel `+0xe4`, `+0xe8`, `+0x104`) replace the first | set by `car_hit`, `car_on_body_surface_contact`, `car_apply_net_event_msg`; read by `vehicle_move`, `vehicle_wheel_contact_solve`, `vehicle_update_visuals` |

`vehicle_move` clears `0x1b0` at each step.

**Bit `0x40000000` of the same word means the wheel has come off.** `car_prepare_wheel_dropout` sets
it on the live damage path, `car_track_spawn_dropout_part` on recorded playback and
`car_apply_net_event_msg` from the network; `vehicle_reset` and `vehicle_game_init` clear it; the
wheel solver, the integrator and the wheel emitters test it, and `pursuit_event_tick` ends a chase
record when any wheel of the target carries it. The spike strip does not write it.

## 7. The records the class owns

| Record | Size | Offset | Field | Evidence |
|---|---|---|---|---|
| wheel (pointed to by `+0xcb4`) | `0x1d4` | `+0x04` | `I3D_frame *` | `vehicle_reset_from_frame` |
| wheel | `0x1d4` | `+0x0c` | lateral offset of the hub: the line test starts at `+0x10` + (`+0x0c`, 0, 0) | `vehicle_get_wheel_cam_pos`, `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x14` | the hub's top stop, local y | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x1c` | world hub position | `vehicle_reset_from_frame` |
| wheel | `0x1d4` | `+0x88` | the spring rate | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x8c` | the spring's free length | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x90` | the load: `(+0x8c - +0x94) * +0x88 + rate * damping`; the wheel leaves the ground at a negative load | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x94` | the hub's distance below its top stop | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x98` / `+0x9c` | the damping while the spring compresses / extends | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0xa8` | the tyre id; negative: no contact is taken | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0xb4` | the struck material id (`g_vehicle_material_index_by_id`; material `+0x2d` = leaves skid marks); a change resets the bump distance `+0x1c8` and re-resolves `+0xb0` | `vehicle_wheel_contact_solve`, `vehicle_emit_wheel_skid_segment` |
| wheel | `0x1d4` | `+0x10c` | radius | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x110` | the burst radius, 0.8 * radius, used while bit `0x80000000` is set | `vehicle_wheel_contact_solve` |
| wheel | `0x1d4` | `+0x120` | flag word: the role bits, the runtime bits and `0x40000000` (§6) | §6 |
| wheel | `0x1d4` | `+0x124` | the contact record; its flag `0x10000` ends the skid mark | `vehicle_wheel_contact_solve`, `vehicle_emit_wheel_skid_segment` |
| wheel | `0x1d4` | `+0x12c` | skid-mark ring, `0x14`-byte slots: four objects `+0x00`..`+0x0c`, bytes `+0x10` active, `+0x11` started, `+0x12` the `+0x08` object shown | `vehicle_emit_wheel_skid_segment`, `vehicle_end_wheel_skid_segment` |
| wheel | `0x1d4` | `+0x13c`..`+0x168` | the open skid segment: two edge points, its direction and its last point | `vehicle_emit_wheel_skid_segment` |
| wheel | `0x1d4` | `+0x16c` | the ring cursor, wrapping at the ring length | `vehicle_end_wheel_skid_segment` |
| wheel | `0x1d4` | `+0x188` | the damage wobble, zeroed on repair | `vehicle_init_damage`, `vehicle_update_visuals` |
| wheel | `0x1d4` | `+0x18c` / `+0x190` | health and its repair value | `vehicle_deform`, `vehicle_init_damage` |
| wheel | `0x1d4` | `+0x194` / `+0x198` / `+0x19c` | resistance percent, divisor, threshold; zero health fires bank slot `+0x37c` | `vehicle_deform` |
| wheel | `0x1d4` | `+0x1c0` | the bump height, added to the radius | `vehicle_wheel_contact_solve` |
| `S_SEAT` (`+0xd1c`) | `0x24` | `+0x00` | position, one per `SEAT*` frame | `vehicle_init`, `vehicle_set_seat_mass` |
| `S_SEAT` | `0x24` | `+0x18` | occupant mass | `vehicle_set_seat_mass` |
| `S_SEAT` | `0x24` | `+0x20` | the seat's door / window record (`+0xd2c` entry) | `vehicle_init`, `vehicle_open_door` |
| door / window (`+0xd2c`) | `0x58` | `+0x00` | the door frame object | `vehicle_open_door` |
| door / window | `0x58` | `+0x04`..`+0x10` | the closed rotation, a quaternion | `vehicle_animate_doors` |
| door / window | `0x58` | `+0x14` | a float whose sign picks the hinge axis | `vehicle_animate_doors` |
| door / window | `0x58` | `+0x20` | current door angle | `vehicle_open_door` |
| door / window | `0x58` | `+0x24` | target door angle = fraction x `+0x28` | `vehicle_open_door` |
| door / window | `0x58` | `+0x28` | maximum door angle | `vehicle_set_max_door_angle` |
| door / window | `0x58` | `+0x2c` | opening speed | `vehicle_set_door_open_speed` |
| door / window | `0x58` | `+0x30` | flags: `0x2` the door is moving, `0x4` / `0x8` axis variants; the door sound fires on opening from below 0.0001 and on reaching the closed target | `vehicle_open_door`, `vehicle_animate_doors` |
| door / window | `0x58` | `+0x34` | the window frame | `vehicle_animate_windows` |
| door / window | `0x58` | `+0x38`..`+0x40` | the window's closed position | `vehicle_animate_windows` |
| door / window | `0x58` | `+0x44` / `+0x48` | its current x / y | `vehicle_animate_windows` |
| door / window | `0x58` | `+0x4c` | its lowest y | `vehicle_animate_windows` |
| door / window | `0x58` | `+0x50` | its speed | `vehicle_open_window`, `vehicle_animate_windows` |
| door / window | `0x58` | `+0x54` | ushort deform-zone index: a broken zone stops the window and makes `vehicle_is_window_open` true | `vehicle_animate_windows`, `vehicle_is_window_open` |
| door / window | `0x58` | `+0x56` / `+0x57` | moving byte, open byte (the frame is hidden once fully lowered) | `vehicle_open_window`, `vehicle_animate_windows`, `vehicle_is_window_open` |
| `S_BALLAST` (`+0xd3c`) | `0x1c` | `+0x00` | position, one per `BALLAST*` frame | `vehicle_init` |
| `S_BALLAST` | `0x1c` | `+0x18` | mass | `vehicle_set_seat_mass` |
| lamp (`+0x17c`) | `0x20` | `+0x04` | the glow object: follows the lamp bit, the broken bit and `feature_flags` `0x80` | `vehicle_apply_light_state` |
| lamp | `0x20` | `+0x08` / `+0x0c` | the two light objects (vtable `+0x0c` direction, `+0x24` on / off, `+0x2c` parent), handled only while the arm's override bit is set (§5) | `vehicle_apply_light_state`, `vehicle_add_projector_lights_to_sectors` |
| lamp | `0x20` | `+0x10` | lamp type (§5); 1 / 2 = projector | `vehicle_set_projector_size` |
| lamp | `0x20` | `+0x14` | flags: `0x1` broken (set by `vehicle_deform`, cleared on repair), `0x80000000` the lamp has a special projector, whose index is the ushort `+0x1c` | `vehicle_deform`, `vehicle_init_damage`, `vehicle_apply_light_state` |
| lamp | `0x20` | `+0x18` | health, reset to `+0x14c` | `vehicle_init_damage` |
| special projector (`+0x18c`) | `0x1c` | `+0x00` | `I3D_frame *` | `vehicle_switch_special_projector_texture` |
| special projector | `0x1c` | `+0x04` / `+0x08` | the light and the object, released by the destructor | `vehicle_dtor` |
| special projector | `0x1c` | `+0x0c` / `+0x10` / `+0x14` | three textures | `vehicle_dtor` |
| special projector | `0x1c` | `+0x18` | state: bits `0x1` / `0x2` select texture `+0x14` / `+0x10` (`vehicle_init_damage` sets 3); byte bit `0x80` = in use this frame | `vehicle_switch_special_projector_texture`, `vehicle_init_damage`, `vehicle_apply_light_state` |
| `S_COLBODY_HEAD` (`+0x528`) | `0x74` | `+0x00` | the sphere vector, `0x3c`-byte spheres, `add_track` at sphere `+0x1c` | `vehicle_clear_body_coll_add_track` |
| `S_COLBODY_HEAD` | `0x74` | `+0x10` | the child-head vector, the recursion's argument | `vehicle_clear_body_coll_add_track`, `vehicle_body_coll_resolve` |
| `S_COLBODY_HEAD` | `0x74` | `+0x24` | the centre, model space | `vehicle_body_coll_resolve` |
| `S_COLBODY_HEAD` | `0x74` | `+0x30` | the centre's sweep over the step | `vehicle_body_coll_resolve` |
| `S_COLBODY_HEAD` | `0x74` | `+0x3c` | `add_track`, three floats | `vehicle_clear_body_coll_add_track` |
| `S_COLBODY_HEAD` | `0x74` | `+0x48` | the centre, world space | `vehicle_body_coll_resolve` |
| `S_COLBODY_HEAD` | `0x74` | `+0x54` | the radius | `vehicle_body_coll_resolve` |
| `S_COLBODY_HEAD` | `0x74` | `+0x58` / `+0x5c` | a byte selecting the moving-box test, and the box | `vehicle_body_coll_resolve` |
| sphere (`S_COLBODY_HEAD +0x00`) | `0x3c` | `+0x04` / `+0x28` | the centre, model / world space | `vehicle_body_coll_resolve` |
| sphere | `0x3c` | `+0x10` | the sweep over the step | `vehicle_body_coll_resolve` |
| sphere | `0x3c` | `+0x34` | the radius | `vehicle_body_coll_resolve` |
| sphere | `0x3c` | `+0x38` | ms since its last impact particle | `vehicle_body_coll_resolve` |
| deform zone (`+0x294`) | `0x34` | `+0x00` / `+0x04` | the zone frame and its mesh | `vehicle_deform`, `vehicle_prepare_deform_zone` |
| deform zone | `0x34` | `+0x08` | the cracked-glass material of a type-3 zone | `vehicle_prepare_deform_zone`, `vehicle_deform_glass` |
| deform zone | `0x34` | `+0x0c` | type 1..9; `vehicle_prepare_deform_zone` moves the type-2 zones to the end | `vehicle_prepare_deform_zone`, `vehicle_deform` |
| deform zone | `0x34` | `+0x0e` | flags: `0x1` broken (skipped by the impact and window code; set by `vehicle_deform_glass` at zero health), `0x2` hit since the last `vehicle_update_visuals` pass (health drops only while it is clear); `vehicle_init_deform` clears the word | `vehicle_get_part_name_tag`, `vehicle_deform`, `vehicle_deform_glass`, `vehicle_init_deform` |
| deform zone | `0x34` | `+0x10` / `+0x14` | divisor and threshold (km/h times mass) | `vehicle_deform` |
| deform zone | `0x34` | `+0x18` | the health scale and the repair value of `+0x20` | `vehicle_deform`, `vehicle_init_deform` |
| deform zone | `0x34` | `+0x1c` | the glass crack threshold: below it the cracked material is swapped in | `vehicle_deform_glass` |
| deform zone | `0x34` | `+0x20` | health; zero drops the zone through bank slot `+0x378` and zeroes the lamps on its frame | `vehicle_deform` |
| deform zone | `0x34` | `+0x24` | resistance percent | `vehicle_deform` |
| deform zone | `0x34` | `+0x28`..`+0x30` | the per-axis limit of a vertex's displacement from the pristine copy | `vehicle_deform` |
| `S_dyn_coll_record` (`+0x2ac`, `+0x2f4`) | `0x48` | `+0x00` | type `0x40`, the transformed box | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x04` | the root frame | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x08` | the query-exclusion mask: `0x201` (`+0x2ac`, the `+0x1f38` records), `0x202` (`+0x2f4`) | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x0c` | the grid-bucket back-link, set by `collision_dyn_insert` | `vehicle_game_init` (0) |
| `S_dyn_coll_record` | `0x48` | `+0x10` | the centre: the root world position `+0x39c` | `vehicle_game_init`, `vehicle_reset`, `vehicle_reset_from_frame`, `vehicle_update_visuals` |
| `S_dyn_coll_record` | `0x48` | `+0x1c` | the bounding radius | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x24` / `+0x30` | the box: the `CENTRE` box +/- 0.1, its minimum y lowered by 0.5 in `+0x2f4` | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x3c` / `+0x40` | `&+0xd64` / `&+0xda4` | `vehicle_game_init` |
| `S_dyn_coll_record` | `0x48` | `+0x44` | the owner, `+0xcf0` | `vehicle_game_init` |
| `S_engine_sound_bank` (`+0x768`, `+0x810`, `+0xa5c`, `+0xb04`) | `0xa8` | `+0x00` / `+0x14` | `pitch_min[5]` / `pitch_max[5]`: the pitch at the lowest / highest audible revs | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |
| `S_engine_sound_bank` | `0xa8` | `+0x28` / `+0x3c` | `revs_min[5]` / `revs_max[5]`: the audible band, percent of idle..max revs | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |
| `S_engine_sound_bank` | `0xa8` | `+0x50` / `+0x64` | `revs_fade_in_end[5]` / `revs_fade_out_start[5]` | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |
| `S_engine_sound_bank` | `0xa8` | `+0x78` / `+0x8c` | `volume_load[5]` / `volume_coast[5]`: the `p6` / `p7` volumes, 0..1 | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |
| `S_engine_sound_bank` | `0xa8` | `+0xa0` | `uint8_t mute_in_neutral[5]`, then 3 bytes of padding | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |
| `S_engine_sound_bank_nomute` (`+0x8b8`, `+0xbac`) | `0xa0` | `+0x00`..`+0x8c` | the same eight arrays, no mute bytes | `vehicle_apply_carinit_params`, `vehicle_update_engine_pitch` |

`S_SEAT`, `S_BALLAST` and `S_COLBODY_HEAD` are the developers' record names (§12). The deform zones
are one per model frame tagged `[DEFORM BODY]` .. `[DEFORM ROOF]` (types 1..9, `structs/car.md`
§4c); `vehicle_cb_enum_deform_zones` builds them from `vehicle_init`'s frame walk and
`vehicle_prepare_deform_zone` clones each mesh into the pristine copy `+0x284`, from which
`vehicle_init_deform` restores them. The `add_track` vectors are written only as zero: none of the
functions that hold a head or a sphere pointer stores to them or passes their address on.

## 8. The translation units

The class occupies four contiguous PC units mapping 1:1 onto the four PS2 `C_Vehicle*.cpp` source
files; each opens at its CRT initializer and `atexit` pair. A "no PC twin" claim for a `C_Vehicle`
method has to say which of the four it searched.

| PC range | PS2 source file | PC starts | PC bytes | PS2 bytes | Ratio |
|---|---|---|---|---|---|
| `0x00509400`..`0x0051fcd0` | `C_Vehicle.cpp` | 136 | 92 368 | 118 960 | 0.78 |
| `0x0051fcd0`..`0x0052c380` | `C_Vehicle_effects.cpp` | 29 | 50 864 | 60 848 | 0.84 |
| `0x0052c380`..`0x005373b0` | `C_Vehicle_kernel.cpp` | 15 | 45 104 | 58 288 | 0.77 |
| `0x005373b0`..`0x0053c280` | `C_Vehicle_sounds.cpp` | 26 | 20 176 | 31 320 | 0.64 |

The 206 starts are all named. Three foreign functions sit inside, emitted into whichever object first
used them: `i3d_frame_get_local_direction_normalized` and `i3d_frame_get_parent_ptr` in the effects
unit, `string_ensure_buffer` at the tail of the sounds unit. The next unit, `C_wagon.cpp`
(`structs/wagon.md`), opens at `crt_static_init_guard_0053c280`; PC and PS2 link order agree across
`C_turnout.cpp`, the four `C_Vehicle*.cpp` and `C_wagon.cpp`.

### 8a. The five file-scope vectors

`C_Vehicle.cpp` opens with five MSVC static quartets (§3a), each guarded by one bit of
`g_vehicle_static_dtor_guard` (`0x00656318`).

| Guard bit | Begin pointer | What | Element |
|---|---|---|---|
| `0x01` | `g_vehicle_tyre_list_begin` | the tyre list | `S_TYRE`, stride `0x28` |
| `0x02` | `g_vehicle_materials_begin` | the material list | `S_VEH_MATERIAL`, stride `0x30` |
| `0x04` | `g_vehicle_particles_begin` | the particle list | `S_VEH_PARTICLE`, stride `0x1c` |
| `0x08` | `g_vehicle_projector_light_sectors_begin` | the sectors the projector lights are registered with | `I3D_sector *` |
| `0x10` | `g_vehicle_sound_sectors_begin` | the sectors the sounds are registered with | `I3D_sector *` |

The last two are each held by a registrar that push-backs uniquely
(`vehicle_register_projector_light_sector` / `vehicle_register_sound_sector`) and an add / remove pair
that walks the list. The two pairs use adjacent `I3D_sector` vtable slots, `0x50` / `0x54` for lights
and `0x58` / `0x5c` for sounds. Beside them sits `g_vehicle_material_index_by_id`, a 256-byte
material-id to list-index table at `0x00656208`; `DAT_00656204 + 4` would place it four bytes early,
because `0x00656204` is the particle vector's capacity pointer.

### 8b. `C_Vehicle.cpp`

Anchored on a 12-for-12 in-order block, PC `0x005173a0`..`0x00517ae0` against `SetGear`..`SetCutTime`.
It holds construction, the geometry and CARINIT import, the setter family, the save-state triple, the
body-collision tree builders, the callback bank (§10) and the container helper tail.

### 8c. `C_Vehicle_effects.cpp` — deform, damage, doors, windows, lamps

**One caller validates the whole unit.** `vehicle_update_visuals` calls every `Do*` / `Update*`
function here, and its PS2 twin calls the same set with the same multiplicities; `UpdateAxis` x2 and
`CrashTyreSnd` x2 match on both builds, and `UpdateSteeringWheels` is called once from both `Tick`
and `Update` on both builds.

| Stage | Function | What it does |
|---|---|---|
| build | `vehicle_prepare_deform_zone` | partitions the deform zones, clones each mesh and fills the pristine copy `+0x284` |
| texture | `vehicle_init_deform_glass_texture` | the cracked-glass texture pair on every type-3 (glass) zone |
| reset | `vehicle_init_deform` | restores the pristine vertices, the repair path; fires `+0x398` |
| reset | `vehicle_init_damage` | refills the damage pools, the lamps and the projectors |
| impact | `vehicle_deform` | the entry point; dispatches to the three below |
| impact | `vehicle_deform_glass` | breaks one zone's glass; fires `+0x380` |
| impact | `vehicle_apply_damage` | drains the two damage pools |
| impact | `vehicle_switch_special_projector_texture` | swaps a damaged projector's texture |

The seven per-frame emitters (`vehicle_update_strips`, `_motor_particles`, `_steering_wheels`,
`_rotation_wheels`, `_wheel_particles`, `_exhaust_particles`, `vehicle_emit_body_collision_particles`)
are a forced 7-for-7 block. Two of them read the material-id table; outside them it is read by
`vehicle_update_visuals`, `vehicle_body_coll_resolve`, `vehicle_emit_wheel_skid_segment`,
`vehicle_update_skid_sound` and `car_on_collision_apply_damage` (`def-files-docs/materialy.md` §1a).

### 8d. `C_Vehicle_kernel.cpp` — the motor model and the integrator

| PC | PS2 | Ratio | What it is |
|---|---|---|---|
| `vehicle_tick_drivetrain` | `Tick` | 0.53 | the physics half of the frame |
| `vehicle_update_visuals` | `Update` | 0.35 | the visual and audio half (§8c) |
| `vehicle_compute_motor_force` | `MotorForce` | 0.71 | drive force; calls `ComputeMoment` once |
| `vehicle_compute_moment` | `ComputeMoment` | 1.00 | the mid-gap anchor |
| `vehicle_update_motor_revs` | `MotorRot` | 0.55 | revs and gearbox; fires `GearSnd` once |
| `vehicle_substep_move` | `Engine` | 1.03 | subdivides the frame and calls `Move` per sub-step |
| `vehicle_move` | `Move` | 1.08 | the integrator; six float parameters |
| `vehicle_wheel_contact_solve` | `DoWheelsCollision` | 0.88 | the wheel solver |
| `vehicle_body_coll_resolve` | `BodyCollision` | 0.89 | the body-collision tree solver |

The two per-frame drivers are named for their callee sets, which are disjoint: drivetrain against
visuals.

**The sub-step.** `vehicle_substep_move` runs one `vehicle_move` when the frame time is at most
`+0xca0`, otherwise `floor(dt / +0xca0) + 1` equal sub-steps. `vehicle_ctor` seeds 0.028 s,
`vehicle_set_cut_time(ms)` accepts 5 to 50 ms, and `traffic_car_game_init` passes 42. The local chain
from the pedal setters to the velocity writes is in `basic/artifacts/vehicle-movement.md` §3.

**The two function-local static sort vectors** of `vehicle_body_coll_resolve`, built on first call:

| Object | Begin | Entries index | `atexit` destructor |
|---|---|---|---|
| `g_vehicle_body_coll_sphere_sort_list` `0x00656344` | `0x00656348` | the `0x3c`-stride spheres of a head | `vehicle_body_coll_sphere_sort_list_static_dtor` |
| `g_vehicle_body_coll_head_sort_list` `0x00656354` | `0x00656358` | the `0x74`-stride heads at `+0x528` | `vehicle_body_coll_head_sort_list_static_dtor` |

Both destructors are taken by address only (`atexit(&...)`), so an `E8` call census reports zero
callsites for them. Two more function-local statics: `g_vehicle_body_coll_contact_normal`
(`0x00656330`, one `S_vector`) receives the contact normal from the body-contact surface filter
(`+0x38c`, §10); `g_vehicle_body_coll_struck_actor` (`0x0065633c`) holds the struck actor, the contact
record's `+0x44` when its flag `0x40` is set, else 0. A zero skips the actor filter (`+0x370`) and the
collision-info query (`+0x36c`), and the value is passed to the post-collision notification
(`+0x374`).

### 8e. `C_Vehicle_sounds.cpp` — the sound bank

21 of the 22 PS2 methods have a PC function, plus two PC-only ones.

* **41 handles, named by CARINIT data.** `vehicle_init_sounds` opens 40 exterior handles through
  `vehicle_sound_slot_init`, each name taken from the `S_CARINIT` table at
  `S_CARINIT + 0xc64`..`+0xee4` (stride `0x10`), and the 41st from the hard-coded `PNEU_PUNC.WAV`
  (tyre puncture), the handle of `vehicle_update_flat_tyre_sound`. The table below gives each role's
  name slot and handles.
* **The interior band is never opened and never selected.** Every sound function picks the interior
  handle (`+0x6c8`..`+0x764`) or the exterior one (`+0x9a8`..`+0xa48`) by `+0x50d`, but both
  `vehicle_init_sounds` callsites (`0x005117a1`, `0x0051732f`) pass `this`, and the interior handles
  are stored only as 0 (`vehicle_ctor`, `vehicle_release_sounds`). `+0x50d` follows `+0x50c`, the
  camera's `camera_inside`, which only the camera records `P In`, `CP Sniper`, `C RailwayIn` and the
  truck cargo-bed mode set; no in-car view sets it (`structs/camera.md` §4a). In game, the one car
  with an interior view, the Black Dragon 4WD, shows it in the bonnet view's place of the
  camera-change cycle. The single loop `+0x9b4` has no interior twin.
* **`feature_flags` `0x20` selects the engine.** Clear: the exterior engine is the single loop
  `+0x9b4` (pitch `+0xa4c`..`+0xa50`, volume `+0xa58` below `+0xa54`) and the handbrake, gear,
  environment, puncture and wheel-thump sounds are off. Set: the three engine banks play (§7
  `S_engine_sound_bank`); the reverse bank only in reverse gear, the other two only in a forward or
  neutral gear, at the volume blend `engine_load` `+0x51c` sets.
* **The crash tiers.** `vehicle_play_crash_sound` compares the impulse with four bounds (`+0x994` /
  `+0xc88`; below the first nothing plays). Each tier plays its primary or its alternate at random,
  neither cuts in before the other has played 800 ms, and the tier's mute byte (`+0x9a4` / `+0xc98`)
  silences it while a driven wheel is on the ground (`+0x410`, passed by
  `vehicle_body_coll_resolve`).
* **A master volume scale.** `g_vehicle_sound_volume_scale` (`0x0064c7c0`) is passed to the sound
  object's volume slot `0x68` by every call in the unit; `vehicle_set_global_sound_volume` is its only
  writer, and `vehicle_update_engine_pitch` scales it by the engine fade `+0xc9c`.

| Role | CARINIT name | Exterior handle | Interior handle |
|---|---|---|---|
| engine start / stop | `+0xc64` / `+0xc74` | `+0x9a8` / `+0x9ac` | `+0x6c8` / `+0x6cc` |
| failed start | `+0xc84` | `+0x9b0` | `+0x6d0` |
| single engine loop | `+0xc94` | `+0x9b4` | - |
| on-load, coast, reverse banks, 5 each | `+0xca4`, `+0xcf4`, `+0xd44` | `+0x9b8`, `+0x9cc`, `+0x9e0` | `+0x6d4`, `+0x6e8`, `+0x6fc` |
| idle | `+0xd94` | `+0x9f4` | `+0x710` |
| environment, opened only when `+0x628` is above 0.01 | `+0xda4` | `+0x9f8` | `+0x714` |
| horn | `+0xdb4` | `+0x9fc` | `+0x718` |
| siren, 4.0 / 110.0 / 0.45 | `+0xdc4` | `+0xa00` | `+0x71c` |
| handbrake | `+0xdd4` | `+0xa04` | `+0x720` |
| gear change, two | `+0xde4`, `+0xdf4` | `+0xa08`, `+0xa0c` | `+0x724`, `+0x728` |
| skid | `+0xe04` | `+0xa10` | `+0x72c` |
| brake loop, opened and never started | `+0xe14` | `+0xa14` | `+0x730` |
| wheel thump | `+0xe24` | `+0xa18` | `+0x734` |
| tyre burst | `+0xe34` | `+0xa1c` | `+0x738` |
| crash tiers 0..3 | `+0xe54`..`+0xe84` | `+0xa20`..`+0xa2c` | `+0x73c`..`+0x748` |
| their alternates | `+0xe94`..`+0xec4` | `+0xa30`..`+0xa3c` | `+0x74c`..`+0x758` |
| door, flag argument set / clear | `+0xed4` / `+0xee4` | `+0xa40` / `+0xa44` | `+0x75c` / `+0x760` |
| puncture, 6.0 / 30.0 | `PNEU_PUNC.WAV` | `+0xa48` | `+0x764` |

Default 3D parameters are minimum distance 4.0, maximum 50.0, volume 0.3 and 0.2.

## 9. Seats, ballast and the centre of mass

`vehicle_init` builds two point-mass lists from the model: one `S_SEAT` per `SEAT*` frame into
`+0xd1c` (`std_vector24_insert_n`) and one `S_BALLAST` per `BALLAST*` frame into `+0xd3c`
(`std_vector1c_insert_n`), each carrying a position and a mass (§7). `vehicle_set_seat_mass` stores one
seat's occupant mass and then recomputes the centre of mass as the running weighted average over the
seats and then the ballast points, skipping pairs whose masses are both below 0.001. Its 10 callsites
are `car_set_owner` 2, `traffic_car_adopt_car_with_new_driver` 3, `traffic_car_activate_elements` 2,
`traffic_car_seat_pedestrian_as_driver`, `opponent_game_init` and `vehicle_game_init`; every receiver is
`<C_car> + 0x70`.

`vehicle_get_seat_count` returns the number of `S_SEAT` records; a seat's `+0x20` points at its door /
window record, so a car has one door record per seat.

## 10. The callback bank, `+0x364`..`+0x398`

`C_Vehicle` does not implement collision policy; it calls back into its owner. The owner installs
thirteen function pointers through thirteen one-line setters, for a `C_car` in `car_init`.

| Offset | Setter | Role | What `car_init` installs |
|---|---|---|---|
| `+0x364` | `vehicle_set_callback_collision` | full collision reaction | `car_react_to_and_report_collision` |
| `+0x368` | `vehicle_set_callback_collision_quiet` | consequence-free reaction | `car_react_to_collision` |
| `+0x36c` | `vehicle_set_callback_collision_info` | response-parameter query | `car_get_collision_info` |
| `+0x370` | `vehicle_set_callback_actor_filter` | struck-object relevance filter | `car_collision_filter_actor` |
| `+0x374` | `vehicle_set_callback_was_collision` | post-collision notification | `car_on_collision_apply_damage` |
| `+0x378` | `vehicle_set_callback_prepare_dropout` | body-panel drop-out | `car_prepare_body_part_dropout` |
| `+0x37c` | `vehicle_set_callback_prepare_wheel_dropout` | wheel drop-out | `car_prepare_wheel_dropout` |
| `+0x380` | `vehicle_set_callback_break_glass` | glass shatter | `car_break_glass` |
| `+0x384` | - | an `Init`-time hook nothing installs (§10b) | - |
| `+0x388` | `vehicle_set_callback_wake_up` | wake a sleeping car | NULL; `car_on_act_state_changed` installs `car_wake_up` later |
| `+0x38c` | `vehicle_set_callback_body_collision_filter` | body-contact surface filter | `car_on_body_surface_contact` |
| `+0x390` | `vehicle_set_callback_wheel_collision_filter` | wheel-contact surface filter | `car_collision_filter_wheel` |
| `+0x394` | `vehicle_set_callback_deform_part` | panel deformation | `car_add_deform_scratch` |
| `+0x398` | `vehicle_set_callback_repair` | repair / decal wipe | `car_clear_damage_decals` |

Each setter's body is a single `mov [this + N], arg`, and the thirteen N are consecutive, so the
install order carries no skew. Every bank call passes `+0xcf0`, the owning `C_car`, in ECX
(`mov ecx,[esi+0xcf0]; call eax`, `0x00519afe`).

### 10a. `+0x388` — the wake-up slot

`vehicle_set_callback_wake_up` has 3 callsites: `car_init` (NULL) and two in
`car_on_act_state_changed`, which installs `car_wake_up` on entering act state 1 and clears it on
state 0. Eleven class functions call the slot when it is non-null: the light setters
(`vehicle_set_headlights`, `_left_indicator`, `_right_indicator`, `_emergency_lights`,
`_light_script_override`), the motion setters `vehicle_set_velocity`, `vehicle_set_angular_velocity`,
`vehicle_apply_external_force`, `vehicle_apply_external_impulse` and `vehicle_settle`, and
`vehicle_open_door`; `car_unpack_light_state` does the same. They are consumers of the slot: changing
a lamp, a door or the motion of a car that is asleep (act state 1) wakes it, and on an awake car the
slot is NULL. `car_wake_up`'s whole body is `actor_set_act_state(0)`.

### 10b. `+0x384` — a hook nothing installs

`vehicle_ctor` zeroes it and `vehicle_init` calls it when it is non-null, once per element of a list
it is building. No setter exists and no instruction writes any other value, so in the shipped game the
call never happens.

## 11. `C_car` is a different class

Everything above describes `C_Vehicle` (mangled `9C_Vehicle`). The `C_car` entity that holds it
(mangled `5C_car`, entity kind 4) is a separate class: `C_car` → `car_*`, `C_Vehicle` → `vehicle_*`,
and a prefix is assigned only after the receiver has been measured at the callsites. The `C_car`
layout, the seat and owner API, the model-level (boost) switching and the `C_car` methods are in
`structs/car.md`.

## 12. PC vs PS2

All cross-build material for this class lives here; the twin columns of §3 and the tables of §8 are
the mapping itself.

**PS2 methods with no PC function.**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00231060 | `reserve__Q23gif29vector<Q29C_Vehicle6S_SEAT,8>Fi` | a helper of the PS2 container template; the PC vector code is the shared helpers of §3d |
| 0x002311d0 | `reserve__Q23gif32vector<Q29C_Vehicle9S_BALLAST,8>Fi` | the same |
| 0x00231330 | `__ct__Q29C_Vehicle6S_DOORFRCQ29C_Vehicle6S_DOOR` | the copy constructor of the PS2 door record; no PC function copies a door record on its own |
| 0x002313d0 | `reserve__Q23gif29vector<Q29C_Vehicle6S_DOOR,8>Fi` | a helper of the PS2 container template |
| 0x00238bb0 | `ClearParticleRef__9C_VehicleFi` | PS2-only: the particle-slot pool |
| 0x00238d60 | `LockParticle__9C_VehicleFii` | PS2-only: the particle-slot pool |
| 0x00239270 | `UnlockParticle__9C_VehicleFi` | PS2-only: the particle-slot pool |
| 0x002397f0 | `TickParticles__9C_VehicleFi` | PS2-only: the particle-slot pool |
| 0x00240370 | `DoCollisionStatic__9C_VehicleFRC8S_vectorRC8S_vectorRC8S_vector` | inlined into its two PC callers, `railway_collide` and `wagon_ai` |
| 0x00241a50 | `IsDoor__9C_VehicleFi` | no PC start located (Open item 1) |
| 0x00242cd0 | `DuplicateDamageAndDeform__9C_VehicleFP9C_Vehicle` | PS2-only: its one caller chain has no PC counterpart |
| 0x002480a0 | `SetCallbackLP__9C_VehicleFPFR14S_VEH_PARTICLEi_b` | PS2-only: the particle callbacks |
| 0x002480b0 | `SetCallbackCP__9C_VehicleFPFii_i` | PS2-only: the particle callbacks |
| 0x0024cd30 | `UpdateDeformPart__9C_VehicleFiPC8S_vectorb` | no PC start located (Open item 2) |

**Field pairs.** The two builds do not lay the class out as a uniform shift. Each pair below is
measured: the PS2 method reads its offset where its PC twin reads the PC one.

| PC | PS2 | What | Proof |
|---|---|---|---|
| `+0x130` | `+0x510` | particle velocity scale | `UpdateBodyParticles` |
| `+0x140` | `+0x51c` | lock count | `LockVehicle` |
| `+0x14c` | `+0x528` | lamp health default | `InitDamage` |
| `+0x180` | `+0x560` | lamp vector begin | `InitDamage` |
| `+0x190` | `+0x574` | special-projector vector begin | `InitDamage` |
| `+0x19c` | `+0x578` | light state | `DoLights`, same bit numbers |
| `+0x224` | `+0x5c8` | master flags | `IsVehicleUnstable`, bit 3 |
| `+0x230` | `+0x5d4` | motor force factor | `InitDamage` |
| `+0x234` / `+0x238` | `+0x604` / `+0x608` | engine pool | `InitDamage` |
| `+0x26c` / `+0x270` | `+0x5d8` / `+0x5dc` | gearbox pool | `InitDamage` |
| `+0x294`, stride `0x34` | `+0x620`, stride `0x3c` | deform zones (begin `+0x298` / `+0x624`) | `CodeIndexDan` |
| `+0x540` | `+0x768` | wheel count | `InitDamage` |
| `+0x5bc` | `+0x804` | throttle out | both builds' `Reset`, save / load pair and `Update` |
| `+0x618` | `+0x8f0` | speed | `UpdateSteeringWheels`, the steering speed factor |
| `+0x634` | `+0x88c` | brake force | `SetBrake` |
| `+0xcb4` | `+0xf0c` | wheel array | `InitDamage`, `SlaifEnd` |
| `+0xce4` | `+0xf14` | feature flags, gate bit `0x200` | `UpdateBodyParticles` |
| `+0xd20`, stride `0x24` | `+0xf50`, stride `0x30` | seat vector begin, mass at `+0x18` / `+0x20` | `SetSeatMass` |
| `+0xd40`, stride `0x1c` | `+0xf68`, stride `0x30` | ballast vector begin, mass at `+0x18` / `+0x20` | `SetSeatMass`, `ComputeMass` |
| `+0xde8` (4388 B) | `+0xf78` (4672 B) | `S_CARINIT` | `GetInit` |
| wheel `+0x16c`, slot `0x14` | wheel `+0x214`, slot `0x0c` | skid-ring cursor | `SlaifEnd` |

**Record names.** The PS2 symbols name the point-mass records `C_Vehicle::S_SEAT` and
`C_Vehicle::S_BALLAST` (`gif::vector` helpers, PS2-only table) and a third, `C_Vehicle::S_DOOR`, with
its own vector; the PC holds its door data in the `0x58`-byte records of `+0xd2c` that each seat
points at (§7). Which PC vector corresponds to `S_DOOR` is Open item 8.

**`DoLights` fixes the PC lamp bits.** The PS2 method at `0x00250e90` reads the flag word `+0x578`
with the PC's bit numbers:

| PS2 `DoLights` behaviour | What it fixes on the PC side |
|---|---|
| the bit `0x8` block: `fabs(steer) > threshold(+0x58c)` lights the side of the steering sign, else clears both | bit `0x8` is automatic indicators, and `+0x1b0` is their threshold |
| `steer < 0` sets bit `0x1`; `steer > 0` sets bit `0x2` | left = `0x1`, right = `0x2` |
| `0.0 < *(float *)(this + 0x88c)` sets bit `0x100` | brake lights follow `+0x634` |
| the bit `0x80` block: a timer `+0x594` against a period `+0x590`, `0x10` / `0x20` alternated, both cleared when the previous word `+0x57c` had `0x80` | bit `0x80` is the light bar and `+0x1a0` the previous word |
| `*(int *)(this + 0x81c) < 0` sets bit `0x1000` | reverse lights |

**No PS2 twin exists for the one-line light setters, and that is a measurement.** All 153 PS2
`C_Vehicle` methods were enumerated; the sub-100-byte tail is 35 methods (the 15 `SetCallbackXX`
registrars, `CreateLightProjector`, `CloseVehicleMaterials`, `GetInit`, `SetSteeringLinearity`,
`SetSpeedLimit`, `SetFuel`, `SetHandbrake`, `EnableSounds`, `SetCutTime`, `GetHeadPos`,
`GetRotations`, `IsVehicleUnstable`, `GetVehicleSize`, `GetVehiclePivotPos`, `LockVehicle`,
`SetMaxDoorAngle`, `SetSpeedDoorOpen`, `GetForwardForceMax`, `GetSideForceMax`, `OpenWindow`) and none
of them touches a light bit. GCC inlined that work into `DoLights` and `Process__9C_program`; MSVC
emitted the one-line members of §3.

**The callback bank as an identification lever.** PS2 `Init__5C_carFP9I3D_frame` calls
`SetCallbackXX__9C_Vehicle(this + 0x80, <method address>)` fifteen times: the thirteen of §10 and two
particle callbacks. The PC `car_init` calls the thirteen setters with a literal function pointer each;
because the setters write consecutive offsets, the two sequences align without a body diff, and slot 9
is NULL on both builds. The 9th PS2 install is `SetCallbackWU__9C_VehicleFPFPCv_v(this + 0x80, 0)`, and
PS2 `ChangeState__5C_carFv` installs `Wake_Up__5C_carFPCv` at the two points where
`car_on_act_state_changed` installs and clears `car_wake_up`. PS2 `C_Vehicle + 0x654`, between
`SetCallbackBG` (`+0x650`) and `SetCallbackWU` (`+0x658`), is the slot without a setter, the twin of
PC `+0x384`. `SetCallbackLP` (`+0x670`) and `SetCallbackCP` (`+0x674`) sit past `SetCallbackVR`;
`Init__5C_car` installs `Load_Particle__5C_car` and `Change_Particle__5C_car` there.

**The particle-slot pool is PS2-only.** `LockParticle`, `UnlockParticle` and `ClearParticleRef`
manage a pool of `0x50`-byte slots (`v_particle__9C_Vehicle`, capped by
`max_particle_emitors__9C_Vehicle`) through the `SetCallbackCP` callback, and `C_game::Tick` runs
`TickParticles` between `ClientTick__17C_ParticleManager` and the blood manager's `Tick`. The PC
emitters call no lock or unlock, and PC `game_tick` goes from `particle_manager_tick` straight to
`blood_manager_tick`.

**`GetForwardForceMax` and `GetSideForceMax` are folded into `vehicle_move`.** On the PS2 each sums a
per-wheel force limit (`wheel + 0x1b0` / `+0x1b4`) into a cache field (`+0x5a0` / `+0x59c`) on demand,
and `FillStartValues__8C_driver` calls both. The PC accumulates the two sums in `vehicle_move`'s wheel
loop into `+0x1c8` / `+0x1c4`, and `driver_fill_start_values` reads those fields at the mirrored
position. `FillStartValues__8C_driver` and `driver_fill_start_values` agree statement for statement:
both start the brake cap from a third value and clamp it first to 0.9 x the side sum, then to
0.9 x the forward sum. So `GetForwardForceMax` is `+0x1c8` and `GetSideForceMax` is `+0x1c4`;
`+0x1cc`, the cap `car_ai_tick` divides by, is the start value, which the PS2 computes inline as
`car + 0x624` x `car + 0x74c` and which is no getter.

**`vehicle_get_seat_count` has no PS2 symbol because GCC inlines it.** The script arms that call it on
the PC (`car_setdooropen`, `car_getseatcount`, `car_disable_uo`) compute
`(car + 0xfd4 - car + 0xfd0) / 0x30` inline on the PS2, the count of the `S_SEAT` vector.

**`DuplicateDamageAndDeform` is PS2-only.** Its one caller chain is
`C_traffic_car::SebraliMiKaaaruSvine` to `C_car::DuplicateDamageAndDeform`, which copies the dummy
cruiser's damage onto the car that replaces it. The PC twin of that caller,
`traffic_car_take_over_cruising_car`, goes from the release call straight to the random
`vehicle_set_fuel` at the mirrored position, with no copy called or inlined.

**`UpdateDeformPart`'s callers** are `C_car::Hit`, `LoadVehicleDamageDeform` and `Deform`. Their PC
twins (`car_hit`, `vehicle_load_state`, `vehicle_deform`) call `vehicle_deform_glass` directly on a
type-3 zone, and none of the five PC callers of `vehicle_deform_glass` tests zone type 8, so the PS2
pass that breaks every window whose parent is a broken type-8 zone has no PC counterpart. The
projector-texture switch it calls is reached only from `vehicle_deform` (three callsites); the callee
counts do not line up one for one, so a fold of the rest is not proven (Open item 2).

**`ReleaseBodyColls` became the head record's destructor.** The PS2 keeps a head's spheres and child
heads in linked lists (spheres through `+0x48`, the first child at `+0x64`, siblings at `+0x74`) and
frees them recursively in `ReleaseBodyColls`, which `CloseVehicle` calls. The PC keeps both as vectors
in the `0x74`-byte head, and `vehicle_colbody_head_dtor` frees them recursively; `vehicle_dtor`
reaches it through `vehicle_colbody_head_scalar_deleting_dtor`. The role, the recursion and the call
from the teardown agree, the container types differ, so the twin is STRUCTURAL. `CloseVehicle` is
called first by the PS2 destructor and by `C_car::CloseCar`; the PC has no `CloseCar`, and the method
is inlined into `vehicle_dtor`. So is `CloseSpecialLightProjectors`, which `CloseVehicle` calls: after
emptying the projector-sector list, `vehicle_dtor` walks the special projectors and, per record,
detaches and releases the light at `+0x04`, releases the textures at `+0x10` / `+0x14` / `+0x0c` and
detaches and releases the object at `+0x08` when it is set, the PS2 method's steps in the same order.
Its other PS2 caller, `C_car::SetOwner`, has no such walk in `car_set_owner`.

**`InitSpecialLightProjectors` is inlined into `vehicle_init_lights`.** Called by `InitLights`, it
enumerates the model's frames of type `0x10020` named `PROJECTOR` and then of type `0x10021` named
`LIGHT` through `CbEnumObj6`; the tail of `vehicle_init_lights` makes the same two enumerations
through `vehicle_cb_enum_object_list`, with the same type codes and names.

**The two kernel sort vectors are both `vector<S_SORT>`.** `BodyCollision` constructs both
function-local statics with the `S_SORT` allocator and registers the `vector<S_SORT>` destructor for
both. `vector<S_COLBODY_EMITOR>` is the element type of a member vector at PS2 `C_Vehicle + 0x744`, to
which `BodyCollision` appends one record per hard impact for the body particle emitter, the queue the
PC replaced with immediate emission from `vehicle_body_coll_resolve`.

**`ClearBodyCollAddTrack`'s PS2 body is an empty stub**; the PC body clears the `add_track` vectors
(§7).

**`DoCollisionStatic` is inlined on the PC at both callsites.** Its two PS2 callsites,
`Collide__9C_railwayFv` and `AI__7C_wagonFUi`, are both in the kind-4 arm and both invoke it on
`car + 0x80`. On the PC both twins, `railway_collide` and `wagon_ai`, call
`vehicle_get_contact_impulse_terms` once each at the mirrored position and no separate response
function. The PS2 inlined `GetRVelMassJ` into `DoCollisionStatic` (its callees are vector math only),
the PC inlined `DoCollisionStatic` into its callers. The symbol's other two PC callers,
`box_solve_impulse_vs_car` and `car_get_collision_info`, are the generic kind-dispatched query the
PS2 also has.

**Factoring differences, marked in the decomp at their sites.**

* `ComputeMass` is inlined into `vehicle_set_seat_mass`: `156 + 1016 = 1172` B against 864 B = 0.74.
* `SetDynamicCollsOn`'s erase is also open-coded in `vehicle_deactivate`, which is why the two
  builds' callee multisets there differ. `vehicle_set_dynamic_colls_off` folds the PS2 method's second
  bool into the `+0x1f26` read, and its parameter is the inverse of the PS2 `on`.
* `CloseParticles` + N x `AddParticle` are fused into `vehicle_set_particles`, which is why that twin
  is STRUCTURAL.
* The PS2 inlined `vehicle_init_strip_and_smoke_emitters` into `Init`:
  `6976 + 5408 = 12 384` against `Init`'s 13 560 = 0.91.
* `CreateLightProjector`'s PS2 symbol is an 8-byte stub against 1456 B of PC code, so its twin is
  STRUCTURAL.
* `UpdateStrips` is 2.30x on the PC, because MSVC inlined the
  `LockVertices` / `UnlockVertices` / `UpdateBBox` / `UpdateBoundVolume` quartet the PS2 calls out of
  line four times each.

**The damage / deform save triple: three 2 → 1 folds.**

| PC function | PS2 methods folded into it | Ratio |
|---|---|---|
| `vehicle_get_save_state_size` | `GetVehicleStateSize` + `GetVehicleDamageDeformSize` | 0.66 |
| `vehicle_save_state` | `SaveVehicleState` + `SaveVehicleDamageDeform` | 0.75 |
| `vehicle_load_state` | `LoadVehicleState` + `LoadVehicleDamageDeform` | 0.57 |

All three PC functions walk the pristine deform copy `+0x284` and run the mesh lock / unlock vertex
walk, and that vector exists only for the deform machinery. The ratios are coherent only under the
fold: 0.66 / 0.75 / 0.57 as pairs against 5.29 / 1.61 / 0.90 as singles.

**`BodyCollision`'s receiver** is `C_Vehicle` on both builds; the PC passes the head vector where
the PS2 passes a head pointer, because the PC keeps heads in vectors (see `ReleaseBodyColls`).
**The pristine geometry lives in different places:** PS2
`InitDeform` restores from a file-scope `deform_orig`, the PC from the member vector `+0x284`.

**Developers' names from the PS2 statics.** `s_material_conv_tbl__9C_Vehicle` and
`v_particle__9C_Vehicle` are the original names for `g_vehicle_material_index_by_id` and
`g_vehicle_particles_begin`; `S_hlp$`-style source tags supply the four `C_Vehicle*.cpp` file names of
§8.

**The container helpers are not comparable by count:** the kernel unit has 4 PC functions against 10
PS2 template symbols, and two of the four are shared across unrelated classes (`std_vector74_size`,
`std_vector8_erase_range`). The record layouts differ between builds even where the algorithm does
not: the seat record `0x24` against `0x30`, the ballast record `0x1c` against `0x30`, and the skid-ring
slot `0x14` against `0x0c`.

## 13. Related

- `structs/car.md` — the `C_car` entity that holds this object at `car + 0x70`.
- `structs/driver.md` — the `C_driver` chase AI that calls these setters.
- `structs/pursuit_event.md` — `pursuit_event`, whose translation unit hosts the
  `0x004f5a30`..`0x004f5ac0` accessors.
- `structs/wagon.md` — `C_wagon`, the next class in link order.
- `structs/player.md` — the player's drive inputs (§2c) and the gearbox key that writes `+0x548`.
- `basic/artifacts/vehicle-movement.md` — the per-frame integrator, the wheel solver and the keyframe
  playback.
- `basic/artifacts/hud_elements.md` §6b — the tachometer that consumes `vehicle_get_engine_rpm`.
- `script-mapping/artifacts/opcodes-verified.md` — the `carlight_*` script cases of §2a.
- `police-system/artifacts/boosting.md`, `def-files-docs/vehicles-bin.md` — the CARINIT model levels.
- `def-files-docs/materialy.md` §1a — the `S_VEH_MATERIAL` fields behind the material list of §8a.

## 14. Open items

1. **`IsDoor`** has no PC start located, the odd one out of an otherwise forced 4-for-5 door block; its
   PS2 callers are `C_grenade::LoadData`, `C_human::Do_ClimbInCarLR`, `C_player::GoToCar_Process` and
   `C_player::Do_AB_OwnerNULL`.
2. **`UpdateDeformPart`** has no PC start; its parent-to-window pass has no PC counterpart (§12), and
   whether the rest is inlined into `car_hit`, `vehicle_load_state` and `vehicle_deform` is not diffed.
3. **Fields resting on a census of the class's own functions.** `unread_1d0`, `unread_220`,
   `unread_344`, `unused_5b0` and `turn_radius` have no reader in the class; their hits in other
   functions were not read one by one.
4. **`accel_analog` (`+0x525`)** has writers and no reader located.
5. **Undecoded bits:** `gearbox_state` bit `0x8`, set during the launch when the speed falls below
   `+0x61c`, has no decoded reader. Bit `0x2000` of `feature_flags`, which the ctor leaves clear, is
   not traced.
6. **External users.** Code outside the class reaches these fields as `car + N + 0x70` and through a
   `C_Vehicle *`; §2 lists the external accesses read for this document. The rest, among them
   `driver_set_car` (size and pivot), `entity_action_tick` and `pursuit_event_tick` (speed, velocity,
   wheels), are not read.
7. **`traffic_car_adopt_car_with_new_driver`'s sub-step**: its argument to `vehicle_set_cut_time` is not
   rendered by the decompiler and is not read from the disassembly.
8. **`S_DOOR`.** The PS2 `C_Vehicle` has a third record vector of `S_DOOR`; whether the PC door /
   window vector `+0xd2c` is its counterpart is not diffed.
9. **The collision masks.** What the record masks `0x201` / `0x202` (§7 `S_dyn_coll_record`) and
   the test masks `0xf02` / 2 (`+0x11c`, `+0x124`) select is not in `structs/collision.md`.
10. **`build_flags` and `vehicle_deform`'s flags from outside.** What `car_explosion` (3),
    `traffic_car_game_init` (1) and `police_manager_apply_state` (3) mean by the bits they store in
    `+0xcec`, and the flag arguments (bits `0x1`, `0x2`) of `vehicle_deform`'s callers, are not read
    beyond the store.
