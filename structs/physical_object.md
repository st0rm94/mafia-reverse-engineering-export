# `C_PhThing` — the physics-body base ("**Ph**ysical **Thing**")

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-11-02 17:43 +0100

The shared rigid-body base behind everything in the game that falls, rolls, tumbles and collides
on its own: dropped car parts, crash debris, boxes, dropped weapons and scripted physics props. The
base is **abstract**: each of its five subclasses (§5) writes its own kind, and the script commands
`CREATE_PHYSICALOBJECT` / `DESTROY_PHYSICALOBJECT` (dispatcher cases `0xcc` / `0xcd`) build the
subclass `C_public_physics` (`structs/public_physics.md`). A body is simulated in one of two shapes,
a **wheel** (one contact point under a disc) or a **box** (fourteen probe points on a cuboid).

| | |
|---|---|
| PC prefix | `physical_object_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_physical_object_vt` @ `0x0063c840`, 39 slots |
| PC base class | `C_actor` (`structs/actor.md`); `physical_object_ctor` calls `actor_ctor` |
| PC instance size | `0x3b0` (`C_public_physics` adds no field and is allocated with `operator_new(0x3b0)`; the last base field ends at `0x3b0`) |
| PC constructor | `physical_object_ctor` @ `0x004be910` |
| PC translation unit | `[0x004be810, 0x004c6b50)`: the unit's CRT pair, the 35 `physical_object_*` functions and `i3d_frame_set_direction` (`structs/i3d_frame.md`); the next unit's guard `crt_static_init_guard_004c6b50` opens `C_player` |
| PS2 class | `C_PhThing` (tag `9C_PhThing`), 26 methods, all mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC. The base fields this class
leans on are `+0x1c` act state (0 awake, 1 asleep), `+0x20` simulated by a remote peer, `+0x24`
world position, `+0x30` forward axis (the world matrix's third row), `+0x68` the scene frame and
`+0x6d` `may_broadcast`, which `physical_object_net_snapshot_state` sets and
`physical_object_update` tests before a state message (`structs/actor.md`).

```c
typedef struct C_PhThing {                       // 0x3b0 bytes; extends C_actor
    /* +0x070 */ uint32_t body_type;             // [H] 1 = wheel, 2 = box, 0 = not configured
    /* +0x074 */ uint8_t  probe_all_points;      // [H] box: test every probe point this step
    /* +0x075 */ uint8_t  reset_rest_on_wake;    // [H] waking zeroes the two rest timers
    /* +0x076 */ uint8_t  full_inertia;          // [H] 1 = full world tensor +0x1a0, 0 = diagonal
    /* +0x077 */ uint8_t  box_configured;        // [H] physical_object_init_box has run
    /* +0x078 */ float    local_centre[3];       // [H] bounding-box centre in body space
    /* +0x084 */ float    world_centre[3];       // [H] centre of mass in world space
    /* +0x090 */ float    rest_test_pos[3];      // [H] position latched for the "has it moved?" test
    /* +0x09c */ uint32_t motion_age_ms;         // [H] ms since the last significant motion
    /* +0x0a0 */ float    axis_x[3];             // [H] world matrix row 0
    /* +0x0ac */ float    axis_y[3];             // [H] world matrix row 1 (row 2 is C_actor +0x30)
    /* +0x0b8 */ tDynamicCollObject coll;        // [H] 0x48 bytes, the body's collision proxy
    /* +0x100 */ float    ground_normal[3];      // [H] normal of the last ground contact, (0,1,0) idle
    /* +0x10c */ S_matrix world_matrix;          // [H] copy of the frame's world matrix
    /* +0x14c */ S_matrix world_matrix_inv;      // [H] its inverse
    /* +0x18c */ float    max_substep_s;         // [H] longest integration sub-step, 0.028 s
    /* +0x190 */ float    mass;                  // [H] the body's mass, descriptor +0x00
    /* +0x194 */ float    inv_inertia_diag[3];   // [H] body-space inverse inertia, diagonal form
    /* +0x1a0 */ S_matrix inv_inertia_world;     // [H] world-space inverse inertia tensor
    /* +0x1e0 */ float    linear_velocity[3];    // [H] the body's linear velocity
    /* +0x1ec */ float    angular_velocity[3];   // [H] the body's angular velocity
    /* +0x1f8 */ float    friction;              // [H] descriptor friction coefficient
    /* +0x1fc */ float    restitution_factor;    // [H] 1 + restitution: 1.05 wheel, 1.01 box
    /* +0x200 */ float    contact_skin;          // [H] contact distance: 0.03 wheel, 0.04 box
    /* +0x204 */ float    contact_stiffness;     // [H] spring constant of the contact
    /* +0x208 */ float    weight[3];             // [H] mass * g * (0, -1, 0)
    /* +0x214 */ float    gravity;               // [H] g = 10.0
    /* +0x218 */ float    box_mass_x4;           // [M] box: mass * 4.0; no reader but the save (.text scan)
    /* +0x21c */ float    wheel_width;           // [H] S_THING_WHEEL +0x04
    /* +0x220 */ float    wheel_radius;          // [H] S_THING_WHEEL +0x00
    /* +0x224 */ float    wheel_damping;         // [H] S_THING_WHEEL +0x1c
    /* +0x228 */ float    probe_point[14][3];    // [H] box: 8 corners + 6 face centres, body space
    /* +0x2d0 */ uint16_t probe_contact_mask;    // [H] bit i = probe point i touched this step
    /* +0x2d4 */ float    contact_force;         // [H] wheel: (skin - gap) * stiffness
    /* +0x2d8 */ float    contact_point[3];      // [H] wheel: world contact point
    /* +0x2e4 */ float    contact_gap;           // [H] distance of the contact above the ground
    /* +0x2e8 */ uint32_t unused_2e8;            // [U] no access; no reader (.text scan)
    /* +0x2ec */ float    rest_lifetime_s;       // [H] time spent nearly at rest
    /* +0x2f0 */ float    rest_lifetime_limit_s; // [H] 0.6 s by default
    /* +0x2f4 */ float    frame_clamp_ratio;     // [H] 83.0 / frame ms, 1.0 when idle
    /* +0x2f8 */ I3D_sound *impact_sound;        // [H] the impact sound object
    /* +0x2fc */ float    hit_pos[3];            // [H] position of the strongest hit this frame
    /* +0x308 */ int32_t  impact_sound_column;   // [H] -1 = silent
    /* +0x30c */ int32_t  collision_material;    // [H] material an actor collision reports, -1 = none
    /* +0x310 */ uint16_t probe_material[14];    // [H] surface material under each probe point
    /* +0x32c */ uint16_t hit_probe_index;       // [H] probe point of the strongest hit
    /* +0x330 */ float    hit_magnitude;         // [H] magnitude of the strongest hit
    /* +0x334 */ uint32_t ms_since_impact_sound; // [H] time since the impact sound last started
    /* +0x338 */ uint8_t  impact_sound_may_restart; // [H] a playing impact sound may be restarted
    /* +0x33c */ uint8_t  riders_alloc;          // [M] the riders vector's MSVC allocator byte; never read (.text scan)
    /* +0x340 */ std::vector<S_SURR_ACT> riders; // [H] {begin, end, cap}: actors standing on it
    /* +0x34c */ float    net_pos[3];            // [H] last received / latched world position
    /* +0x358 */ float    net_linear_velocity[3];  // [H] last received / latched linear velocity
    /* +0x364 */ float    net_latched_ground_normal[3]; // [H] ground_normal when it was last reported
    /* +0x370 */ float    net_correction_velocity[3]; // [H] derived on receive (§6a)
    /* +0x37c */ float    net_angular_velocity[3]; // [H] last received / latched angular velocity
    /* +0x388 */ float    net_rotation[4];       // [H] last received / latched quaternion
    /* +0x398 */ uint32_t ms_since_state_msg;    // [H] time since the last state message
    /* +0x39c */ uint32_t state_msg_holdoff_ms;  // [H] hold-off before the next urgent message
    /* +0x3a0 */ uint32_t net_flags;             // [H] §2a
    /* +0x3a4 */ uint32_t net_msg_time;          // [H] timeGetTime() at the last snapshot / message
    /* +0x3a8 */ uint32_t timer_elapsed_ms;      // [M] counts up to timer_limit_ms; no reader but that count (.text scan)
    /* +0x3ac */ uint32_t timer_limit_ms;        // [M] 3000 when armed, 200 from the constructor; the same
} C_PhThing;
```

**The two shapes.** `body_type` selects the solver: `physical_object_ai_tick` sub-steps the body in
slices of at most `max_substep_s` and calls `physical_object_move_wheel` for 1 and
`physical_object_move_box` for 2.

* **`probe_point[14]` — why fourteen.** `physical_object_init_box` reads the frame's bounding box
  (min / max corner) and writes 14 points: the **8 corners** (every min/max combination of x, y and
  z) and the **6 face centres** (one coordinate at its min or max, the other two at the mid-point).
  `physical_object_move_box` walks the 14 entries backwards from `+0x2d0` in steps of `0xc`,
  transforms each into world space and tests it against the ground plane `ground_normal`; a point
  below the plane sets its bit in `probe_contact_mask`, which is why the mask is a 16-bit word.
  While `probe_all_points` is clear only the points on the leading side of the velocity are tested;
  the flag toggles every step and is forced on when the speed falls below 1.0.
* **`probe_material[14]` — one per probe point.** The box solver stores the material byte of the
  surface under each touching point (`+0x310 + 2*i`); the wheel solver uses entry 0 for its single
  contact, and `physical_object_on_collision` writes `collision_material` into entry 0.
  `hit_probe_index` names the entry of the strongest hit, so `physical_object_update` reads the
  struck material as `probe_material[hit_probe_index]` (the impact-sound rule below).
* **The wheel contact.** `physical_object_move_wheel` casts a vertical line under the wheel
  (`collision_test_line_vertical_thunk`, which returns the surface normal into `ground_normal`),
  takes the gap between the rim and the ground as `contact_gap`, and treats the wheel as touching
  while `contact_gap < contact_skin`. The contact point `contact_point` sits half `wheel_width`
  along `axis_x` from the hub, pushed `contact_skin` along the normal, and the next step applies
  `contact_force = (contact_skin - contact_gap) * contact_stiffness` along `ground_normal`.
  `contact_stiffness` is `gravity * mass * 37.03704` for a wheel and `gravity * mass * 12.5` for a
  box; friction applies `-friction * mass * gravity`.
* **Inertia.** `physical_object_init_box` keeps the diagonal model (`full_inertia` = 0, one value in
  all three slots) when every side ratio of the box lies within 0.5..2, and otherwise writes three
  diagonal values and sets `full_inertia`, so the contact solvers use the world tensor
  `inv_inertia_world` that `physical_object_build_world_inertia_tensor` composes each step.
* **The impact sound.** When `hit_magnitude` exceeds `mass * 0.4` and the column is not −1,
  `physical_object_update` loads sound id `g_surface_material_table + 0x130 + (column + material *
  0x5c) * 4` — the struck surface's record holds 14 impact-sound ids at `+0x130` — places it at
  `hit_pos`, and restarts a playing sound only when `impact_sound_may_restart` is set and
  `ms_since_impact_sound` exceeds 150.

### 1a. `S_SURR_ACT` — the riding-actor record, 0x14 bytes

```c
typedef struct S_SURR_ACT {      // 0x14 bytes
    /* +0x00 */ float    pos[3];    // [H] where the actor was last seen standing
    /* +0x0c */ C_actor *actor;     // [H] the riding actor, the removal key
    /* +0x10 */ int32_t  ttl_ms;    // [H] counted down by the frame delta; < 0 = dropped
} S_SURR_ACT;
```

A body that has come to rest may not fall asleep while somebody stands on it, and an actor that
walks away must stop pinning it awake; the `riders` vector is the memory that makes both possible.
Four walks agree on the layout: `physical_object_update` (TTL tick and expiry),
`physical_object_update_rest_state` (count), `physical_object_remove_surrounding_actor`
(swap-with-last) and `physical_object_move_box` (append, `ttl_ms` = 1000).

## 2. Field evidence

Measured by a census of every `this+N` access in the `physical_object_*` functions, by body reads
of the members that reach a band through a pointer (the matrix copies, the probe walk, the riders
vector), and by a census of the five subclasses' accesses to the band.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x070 | body_type | H | `physical_object_ctor` (0), `physical_object_init_wheel` (1), `physical_object_init_box` (2), `physical_object_save_game_load` | `physical_object_game_init`, `physical_object_ai_tick`, `physical_object_save_game_save`, `physical_object_net_restart_local_sim` | selects `move_wheel` / `move_box` |
| +0x074 | probe_all_points | H | `physical_object_move_box` (toggled each call; 1 when the speed is below 1.0) | `physical_object_move_box` | §1 |
| +0x075 | reset_rest_on_wake | H | `physical_object_game_init`, `physical_object_on_act_state_changed` (1), `physical_object_update_rest_state` (0 while riders hold the body awake) | `physical_object_on_act_state_changed` | when set, waking zeroes `rest_lifetime_s` and `motion_age_ms` |
| +0x076 | full_inertia | H | `physical_object_ctor` (0), `physical_object_init_wheel` (0), `physical_object_init_box`, `physical_object_save_game_load` | `physical_object_game_init`, `physical_object_move_box`, `physical_object_on_collision`, `physical_object_get_contact_terms`, `physical_object_apply_impulse_at_point`, `physical_object_save_game_save`, `physical_object_net_restart_local_sim` | selects between the two contact solvers (§7c) |
| +0x077 | box_configured | H | `physical_object_ctor` (0), `physical_object_init_box` (1) | `physical_object_move_box`, `physical_object_integrate_transform` | - |
| +0x078 | local_centre | H | `physical_object_ctor` (0), `physical_object_init_box`, `physical_object_save_game_load` | `physical_object_game_init`, `physical_object_refresh_world_transform`, `physical_object_save_game_save` | transformed into `world_centre` on every refresh |
| +0x084 | world_centre | H | `physical_object_game_init`, `physical_object_refresh_world_transform`, `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | `physical_object_move_box`, `physical_object_integrate_transform`, `physical_object_accumulate_contact_impulse_diag`, `physical_object_accumulate_contact_impulse_matrix`, `physical_object_on_collision`, `physical_object_hit`, `physical_object_get_contact_terms`, `physical_object_apply_impulse_at_point` | the origin every contact arm is measured from |
| +0x090 | rest_test_pos | H | `physical_object_game_init`, `physical_object_update` | `physical_object_update` | - |
| +0x09c | motion_age_ms | H | `physical_object_game_init`, `physical_object_update_rest_state` (0; `0xfa1` while riders hold it), `physical_object_update`, `physical_object_on_act_state_changed`, `physical_object_on_collision`, `physical_object_hit`, `physical_object_apply_impulse_at_point`, `physical_object_net_restart_local_sim` | `physical_object_ai_tick`, `physical_object_update` | the body sleeps at 4000 |
| +0x0a0 | axis_x | H | `physical_object_init`, `physical_object_game_init`, `physical_object_refresh_world_transform`, `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_save_game_load` | `physical_object_move_wheel` (the axle), `physical_object_move_box`, `physical_object_hit` | - |
| +0x0ac | axis_y | H | the writers of `axis_x`, and `physical_object_integrate_transform` | `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_integrate_transform`, `physical_object_hit` | - |
| +0x0b8 | coll | H | `physical_object_ctor` (zeroed), `physical_object_game_init` (frame `+0xbc`, flags `+0xc0` = `0x800`, position), `physical_object_update`, `physical_object_save_game_load` (position) | `collision_dyn_set_next_owner` from `physical_object_game_done`, `physical_object_update`, `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | the `tDynamicCollObject` the collision world tests against |
| +0x100 | ground_normal | H | `physical_object_ctor` (0, 1, 0), `physical_object_move_wheel` (out-parameter of `collision_test_line_vertical_thunk`) | `physical_object_move_wheel`, `physical_object_move_box` | §1 |
| +0x10c | world_matrix | H | `physical_object_refresh_world_transform` (copied from the frame), `physical_object_game_init` | `physical_object_refresh_world_transform`, `physical_object_game_init`, `physical_object_move_wheel` | rows 0 / 1 / 2 feed `axis_x`, `axis_y` and `C_actor` `+0x30` |
| +0x14c | world_matrix_inv | H | `physical_object_refresh_world_transform`, `physical_object_game_init` | `physical_object_build_world_inertia_tensor` (from `physical_object_game_init`, `physical_object_move_box`, `physical_object_net_restart_local_sim`) | - |
| +0x18c | max_substep_s | H | `physical_object_ctor` (0.028), `physical_object_save_game_load` | `physical_object_ai_tick`, `physical_object_save_game_save` | - |
| +0x190 | mass | H | `physical_object_init_wheel`, `physical_object_init_box` (descriptor `+0x00`), `physical_object_save_game_load` | every solver and both initialisers; `physical_object_update` (`mass * 0.4` impact threshold) | - |
| +0x194 | inv_inertia_diag | H | `physical_object_init_wheel`, `physical_object_init_box`, `physical_object_save_game_load` | `physical_object_build_world_inertia_tensor`, `physical_object_move_wheel`, `physical_object_accumulate_contact_impulse_diag`, `physical_object_on_collision`, `physical_object_get_contact_terms`, `physical_object_apply_impulse_at_point`, `physical_object_save_game_save` | §1 |
| +0x1a0 | inv_inertia_world | H | `physical_object_build_world_inertia_tensor` (its receiver is this field: `lea ecx,[ebp+0x1a0]` at all three callsites) | `physical_object_get_contact_terms`, `physical_object_accumulate_contact_impulse_matrix` | read only while `full_inertia` is set |
| +0x1e0 | linear_velocity | H | `physical_object_ctor`, `physical_object_init_wheel`, `physical_object_init_box` (descriptor `+0x04`), `physical_object_update`, `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_on_collision`, `physical_object_apply_impulse_at_point`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | every solver, `physical_object_update_rest_state`, `physical_object_save_game_save`, `physical_object_post_state_net_event_msg`, `physical_object_net_snapshot_state` | below `1e-05` squared at init the body starts asleep |
| +0x1ec | angular_velocity | H | `physical_object_ctor`, `physical_object_game_init`, `physical_object_update`, `physical_object_move_wheel`, `physical_object_on_collision`, `physical_object_apply_impulse_at_point`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | every solver, `physical_object_save_game_save`, `physical_object_post_state_net_event_msg`, `physical_object_net_snapshot_state` | - |
| +0x1f8 | friction | H | `physical_object_init_wheel`, `physical_object_init_box` (descriptor `+0x10`), `physical_object_save_game_load` | `physical_object_move_wheel` (`-friction * mass * gravity`), both `physical_object_accumulate_contact_impulse_*` (tangential term), `physical_object_save_game_save` | `structs/box_new.md` §4 |
| +0x1fc | restitution_factor | H | `physical_object_init_wheel` (1.05), `physical_object_init_box` (1.01), `physical_object_save_game_load` | both `physical_object_accumulate_contact_impulse_*`, `physical_object_save_game_save` | - |
| +0x200 | contact_skin | H | `physical_object_init_wheel` (0.03), `physical_object_init_box` (0.04), `physical_object_save_game_load` | `physical_object_move_wheel` (the contact test and the contact-point offset), `physical_object_save_game_save` (twice: also in the slot the load restores into `contact_gap`) | §1 |
| +0x204 | contact_stiffness | H | `physical_object_init_wheel` (`gravity * mass * 37.03704`), `physical_object_init_box` (`gravity * mass * 12.5`), `physical_object_save_game_load` | `physical_object_move_wheel` (`contact_force`), `physical_object_save_game_save` | - |
| +0x208 | weight | H | `physical_object_init_wheel`, `physical_object_init_box`, `physical_object_save_game_load`, `box_new_game_init` (0, −1.0, 0 before the initialiser rewrites it) | `physical_object_move_wheel`, `physical_object_move_box` | `mass * gravity * (0, -1, 0)` |
| +0x214 | gravity | H | `physical_object_ctor` (10.0) | `physical_object_init_wheel`, `physical_object_init_box`, `physical_object_move_wheel`, both `physical_object_accumulate_contact_impulse_*`, `physical_object_save_game_load` | every reader multiplies it by the mass |
| +0x218 | box_mass_x4 | M | `physical_object_init_box` (`mass * 4.0`), `physical_object_save_game_load` | `physical_object_save_game_save` | no physics reader (`.text` scan, Open item 2) |
| +0x21c | wheel_width | H | `physical_object_ctor`, `physical_object_init_wheel`, `physical_object_save_game_load` | `physical_object_move_wheel` (half of it along `axis_x`, signed by the side of the contact), `physical_object_save_game_save` | - |
| +0x220 | wheel_radius | H | `physical_object_ctor`, `physical_object_init_wheel`, `physical_object_save_game_load` | `physical_object_move_wheel`, `physical_object_init_wheel`, `physical_object_save_game_save` | - |
| +0x224 | wheel_damping | H | `physical_object_init_wheel`, `physical_object_save_game_load` | `physical_object_move_wheel` (divided by the radius), `physical_object_save_game_save` | - |
| +0x228 | probe_point | H | `physical_object_init_box` | `physical_object_move_box` (the backwards walk from `+0x2d0`) | §1 |
| +0x2d0 | probe_contact_mask | H | `physical_object_move_box` | `physical_object_move_box` | §1 |
| +0x2d4 | contact_force | H | `physical_object_game_init`, `physical_object_move_wheel`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | `physical_object_move_wheel`, `physical_object_save_game_save` | - |
| +0x2d8 | contact_point | H | the writers of `contact_force` | `physical_object_move_wheel`, `physical_object_save_game_save` | - |
| +0x2e4 | contact_gap | H | `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_save_game_load` | `physical_object_move_wheel` | the save stores `contact_skin` in this field's slot |
| +0x2e8 | unused_2e8 | U | - | - | no access in this family; its 17 `.text` hits are `this` of `C_dog`, `C_game`, `C_vehicle` |
| +0x2ec | rest_lifetime_s | H | `physical_object_game_init`, `physical_object_update_rest_state`, `physical_object_update`, `physical_object_on_act_state_changed`, `physical_object_on_collision`, `physical_object_hit`, `physical_object_apply_impulse_at_point`, `physical_object_save_game_load`, `physical_object_net_restart_local_sim` | `physical_object_ai_tick`, `physical_object_update`, `physical_object_save_game_save` | - |
| +0x2f0 | rest_lifetime_limit_s | H | `physical_object_game_init` (0.6), `physical_object_net_restart_local_sim` | `physical_object_update_rest_state`, `physical_object_ai_tick`, `physical_object_update` | - |
| +0x2f4 | frame_clamp_ratio | H | `physical_object_game_init`, `physical_object_ai_tick` | `physical_object_post_state_net_event_msg` | - |
| +0x2f8 | impact_sound | H | `physical_object_game_init` (through `g_i3d_driver` slot `0x4c`) | `physical_object_game_init`, `physical_object_game_done`, `physical_object_update` (`IsPlaying` `+0x54`, `SetOn` `+0x24`, `sound_table_construct_sound`) | - |
| +0x2fc | hit_pos | H | both `physical_object_accumulate_contact_impulse_*`, `physical_object_on_collision` | `physical_object_update` | copied to the sound's position |
| +0x308 | impact_sound_column | H | `physical_object_ctor` (−1), `crash_obj_init` (the `I` tag), `box_new_load_data` | `physical_object_update`, `box_new_save_data` | §1 |
| +0x30c | collision_material | H | `physical_object_ctor` (−1), `box_new_load_data` | `physical_object_on_collision`, `box_new_save_data` | −1 = an actor collision records no hit |
| +0x310 | probe_material | H | `physical_object_move_box` (per probe), `physical_object_move_wheel` (entry 0), `physical_object_on_collision` (entry 0) | `physical_object_update` | §1 |
| +0x32c | hit_probe_index | H | `physical_object_game_init`, `physical_object_update` (0), both `physical_object_accumulate_contact_impulse_*`, `physical_object_on_collision` (0) | `physical_object_update` | - |
| +0x330 | hit_magnitude | H | `physical_object_game_init`, `physical_object_update` (0 each frame), both `physical_object_accumulate_contact_impulse_*`, `physical_object_on_collision` | `physical_object_update`, both `physical_object_accumulate_contact_impulse_*`, `physical_object_on_collision` | each writer keeps the larger value |
| +0x334 | ms_since_impact_sound | H | `physical_object_game_init`, `physical_object_update` | `physical_object_update` | - |
| +0x338 | impact_sound_may_restart | H | `physical_object_ctor` (= 1), `crash_obj_init` (the `T` tag), `box_new_load_data` | `physical_object_update`, `box_new_save_data` | §1 |
| +0x33c | riders_alloc | M | `physical_object_ctor` | - | an empty allocator, never read (`.text` scan); `physical_object_move_box` takes this address as the vector object for `std_vector20_insert` |
| +0x340 | riders | H | `physical_object_ctor`, `physical_object_dtor`, `physical_object_game_done`, `physical_object_update`, `physical_object_move_box`, `physical_object_remove_surrounding_actor` | `physical_object_update_rest_state`, `physical_object_update`, `physical_object_move_box`, `physical_object_remove_surrounding_actor` | §1a |
| +0x34c | net_pos | H | `physical_object_net_snapshot_state`, `physical_object_apply_net_event_msg` | `physical_object_apply_net_event_msg`, `physical_object_net_apply_remote_state` | the 5.0-unit snap-back target (§6a) |
| +0x358 | net_linear_velocity | H | `physical_object_init_wheel`, `physical_object_init_box` (remote body), `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state` | `physical_object_update`, `physical_object_net_restart_local_sim`, `physical_object_apply_net_event_msg`, `physical_object_net_apply_remote_state` | - |
| +0x364 | net_latched_ground_normal | H | `physical_object_update`, `physical_object_net_snapshot_state` (= 0), `physical_object_move_wheel`, `physical_object_move_box` (= `ground_normal`) | `physical_object_move_wheel`, `physical_object_move_box` | a change of `ground_normal` raises `net_flags` bit `0x1` (§2a) |
| +0x370 | net_correction_velocity | H | `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state`, `physical_object_net_apply_remote_state` | `physical_object_net_apply_remote_state` | §6a |
| +0x37c | net_angular_velocity | H | `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state` | `physical_object_update`, `physical_object_net_restart_local_sim`, `physical_object_net_apply_remote_state` | - |
| +0x388 | net_rotation | H | `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state`, `physical_object_net_apply_remote_state` | `physical_object_update` | - |
| +0x398 | ms_since_state_msg | H | `physical_object_update` (adds the frame time), `physical_object_post_state_net_event_msg` (0), `physical_object_net_snapshot_state` (0) | `physical_object_update` | a periodic message falls due once it exceeds the session update interval |
| +0x39c | state_msg_holdoff_ms | H | `physical_object_update` (counts down; half the session interval after an urgent send), `physical_object_net_snapshot_state` | `physical_object_update` | - |
| +0x3a0 | net_flags | H | `physical_object_update`, `physical_object_move_wheel`, `physical_object_move_box`, `physical_object_init_wheel`, `physical_object_init_box`, `physical_object_on_collision`, `physical_object_hit`, `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state`, `physical_object_net_apply_remote_state` | the same set | §2a |
| +0x3a4 | net_msg_time | H | `physical_object_apply_net_event_msg`, `physical_object_net_snapshot_state` | `physical_object_net_apply_remote_state` | - |
| +0x3a8 | timer_elapsed_ms | M | `physical_object_ctor` (0), `physical_object_ai_tick` (adds the frame time), `physical_object_on_act_state_changed`, `physical_object_on_collision`, `physical_object_hit`, `physical_object_net_restart_local_sim`, `physical_object_net_snapshot_state` | `physical_object_ai_tick` | only its own count reads it (`.text` scan, Open item 2) |
| +0x3ac | timer_limit_ms | M | `physical_object_ctor` (200), `physical_object_on_collision`, `physical_object_hit`, `physical_object_net_restart_local_sim` (3000), `physical_object_ai_tick` and `physical_object_on_act_state_changed` (0), `physical_object_net_snapshot_state` | `physical_object_ai_tick` | the same |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | `physical_object_move_box` (append) | `physical_object_update` | - |
| +0x0c | actor | H | `physical_object_move_box` | `physical_object_update`, `physical_object_remove_surrounding_actor` | the removal key |
| +0x10 | ttl_ms | H | `physical_object_move_box` (1000), `physical_object_update` (counts down) | `physical_object_update` | below 0 the entry is dropped |

### 2a. `net_flags` (`+0x3a0`) — bit map

`physical_object_net_snapshot_state` zeroes the whole word. The flag byte of a state message is the
payload byte `+0x38` of the type-`0x1b` message (§6b).

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | state changed, send now | set by `physical_object_move_box` / `_move_wheel` when `ground_normal` differs from `net_latched_ground_normal`, `physical_object_init_wheel`, `physical_object_on_collision` (local body), `physical_object_hit`; `physical_object_update` sends flag byte 3 once `state_msg_holdoff_ms` is 0, clears the bit and sets the hold-off to half the session update interval |
| `0x8` | rotation blend suppressed | set by `physical_object_apply_net_event_msg` on every message; read and cleared after one step by `physical_object_net_apply_remote_state` |
| `0x10` | the last message's flag byte was 0 | set and cleared by `physical_object_apply_net_event_msg`; `physical_object_net_apply_remote_state` then runs an extra rotation step |
| `0x20` | first state message pending | set by `physical_object_init_box`; `physical_object_update` sends flag byte 0, zeroes `net_latched_ground_normal` and `ms_since_state_msg`, and clears it |
| `0x40` | nothing new to report this frame | set by `physical_object_move_box` / `_move_wheel` when `ground_normal` is unchanged and by `physical_object_on_collision` after posting a remote body's collision event; cleared by `physical_object_update` at the end of every call and by `physical_object_apply_net_event_msg`; it limits `physical_object_on_collision` to one event per frame and makes the periodic message carry flag byte 3 instead of 0 |

## 3. Method family

Coverage: all **26** PS2 `C_PhThing` methods are mapped — 27 PC rows (`CollisionCount` maps onto two
PC functions, §7c), 25 CONFIRMED and 3 STRUCTURAL; 8 PC functions have no PS2 counterpart (the
scalar deleting destructor, the inertia-tensor helper, five network members, §6, and the CRT pair).
The vtable slots 19 / 20 / 21 (`SaveData`, `LoadData`, `DuplicateFrom`) are not overridden: each
holds a folded no-op stub of the `C_actor` interface, while `C_box_new` overrides all three
(`structs/box_new.md` §3).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004be810 | `crt_static_init_guard_004be810` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x004be830 | `crt_atexit_register_004be830` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x004be910 | `physical_object_ctor` | instance | `__ct__9C_PhThingFv` | 0x001a4450 | CONFIRMED | `actor_ctor`, zeroes `coll`, sets the vtable and the defaults of §1 (`gravity` 10.0, `max_substep_s` 0.028, `ground_normal` (0, 1, 0), both −1 ids) |
| 0x004bea00 | `physical_object_scalar_deleting_dtor` | instance | - | - | - | slot 16, the MSVC scalar deleting destructor |
| 0x004bea20 | `physical_object_dtor` | instance | `__dt__9C_PhThingFv` | 0x001a4610 | CONFIRMED | frees `riders`, chains the `C_actor` destructor |

### 3b. Accessors

The class has no accessor of its own: every member reads or writes more than one field.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004be840 | `physical_object_build_world_inertia_tensor` | - | - | - | builds `inv_inertia_world = R * diag * R_inv` from `world_matrix_inv` and `inv_inertia_diag`; its receiver is the tensor at `+0x1a0`, not the class base; three callsites, all in the class (§7c) |
| 0x004bea60 | `physical_object_init` | `Init__9C_PhThingFP9I3D_frame` | 0x001a46a0 | CONFIRMED | slot 18: binds the frame and copies its pose |
| 0x004bebd0 | `physical_object_game_init` | `GameInit__9C_PhThingFv` | 0x001a4840 | CONFIRMED | slot 25: registers `coll`, creates `impact_sound`, resets the rest and contact state |
| 0x004beff0 | `physical_object_update_rest_state` | `FindActors__9C_PhThingFv` | 0x001a4d70 | CONFIRMED | below a speed of 0.1 the body falls asleep unless `riders` is non-empty, which holds it awake with the timers near their limits |
| 0x004bf0c0 | `physical_object_game_done` | `GameDone__9C_PhThingFv` | 0x001a4e60 | CONFIRMED | slot 26: removes `coll`, releases `impact_sound`, empties `riders` |
| 0x004bf120 | `physical_object_ai_tick` | `AI__9C_PhThingFUi` | 0x001a4f00 | CONFIRMED | slot 13: sub-steps the body through the solver `body_type` selects; sleeps it after the rest lifetime or 4001 ms of age |
| 0x004bf260 | `physical_object_update` | `Update__9C_PhThingFUi` | 0x001a5100 | CONFIRMED | slot 14: the impact sound, the riders' TTLs, the rest test and the periodic state message |
| 0x004bfa80 | `physical_object_on_act_state_changed` | `ChangeState__9C_PhThingFv` | 0x001a58a0 | CONFIRMED | slot 27: on waking resets the rest timers (when `reset_rest_on_wake`); on sleeping in a session, broadcasts the state |
| 0x004bfae0 | `physical_object_move_wheel` | `MoveWheel__9C_PhThingFf` | 0x001a58d0 | CONFIRMED | one wheel step: vertical ground test, contact spring, friction, damping |
| 0x004c1dd0 | `physical_object_move_box` | `MoveBox__9C_PhThingFf` | 0x001a7160 | CONFIRMED | one box step: the 14-point probe against the ground plane, contact impulses, the riders' append |
| 0x004c2a40 | `physical_object_integrate_transform` | `MoveCount__9C_PhThingFfRC8S_vector` | 0x001a83f0 | CONFIRMED | advances position and orientation and renormalises the axes |
| 0x004c31a0 | `physical_object_accumulate_contact_impulse_diag` | `CollisionCount__9C_PhThingFR8S_vectorR8S_vectorR8S_vectorR8S_vectorfUsP7C_actor` | 0x001a8b50 | STRUCTURAL | the contact impulse with the diagonal inertia; records the strongest hit (§7c) |
| 0x004c3890 | `physical_object_accumulate_contact_impulse_matrix` | `CollisionCount__9C_PhThingFR8S_vectorR8S_vectorR8S_vectorR8S_vectorfUsP7C_actor` | 0x001a8b50 | STRUCTURAL | the same with `inv_inertia_world` (§7c) |
| 0x004c4000 | `physical_object_init_wheel` | `InitWheel__9C_PhThingFP13S_THING_WHEEL` | 0x001a9260 | CONFIRMED | configures the wheel shape from an `S_THING_WHEEL` |
| 0x004c41a0 | `physical_object_init_box` | `InitBox__9C_PhThingFP11S_THING_BOX` | 0x001a93b0 | CONFIRMED | configures the box shape from an `S_THING_BOX` and the frame's bounding box (§1) |
| 0x004c4880 | `physical_object_on_collision` | `Collision__9C_PhThingFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001a9a10 | CONFIRMED | slot 30: an actor bumped the body; applies the impulse, records the hit with `collision_material` |
| 0x004c4b10 | `physical_object_hit` | `Hit__9C_PhThingF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001a9c30 | CONFIRMED | slot 31: a shot or an explosion pushes the body |
| 0x004c5260 | `physical_object_get_contact_terms` | `GetRVelMassJ__9C_PhThingFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vector` | 0x001aa270 | CONFIRMED | relative velocity and effective mass at a contact point |
| 0x004c53f0 | `physical_object_apply_impulse_at_point` | `ExternalImpuls__9C_PhThingFRC8S_vectorRC8S_vector` | 0x001aa3a0 | CONFIRMED | adds an impulse at a world point and wakes the body |
| 0x004c5590 | `physical_object_remove_surrounding_actor` | `InvalidateActor__9C_PhThingFP7C_actor` | 0x001aa4c0 | CONFIRMED | swap-with-last removal from `riders` |
| 0x004c5640 | `physical_object_collision_filter` | `CollisionFilter__9C_PhThingFPUiR8S_vectorR8S_vectorRb` | 0x001aa5a0 | CONFIRMED | adjusts a collision hit before the solver uses it |
| 0x004c5670 | `physical_object_refresh_world_transform` | `UpdateImportantVariables__9C_PhThingFv` | 0x001aa660 | CONFIRMED | copies the frame matrix and its inverse, recomputes `world_centre` and the three axes |
| 0x004c5750 | `physical_object_save_game_get_size` | `SaveGameGetSize__9C_PhThingFv` | 0x001aa820 | CONFIRMED | slot 22 |
| 0x004c5760 | `physical_object_save_game_save` | `SaveGameSave__9C_PhThingFPPv` | 0x001aa840 | CONFIRMED | slot 23 |
| 0x004c5940 | `physical_object_save_game_load` | `SaveGameLoad__9C_PhThingFPPv` | 0x001aaa80 | CONFIRMED | slot 24 |
| 0x004c5d90 | `physical_object_broadcast_state_net_event_msg` | - | - | - | network layer (§6): builds the state payload with a flag byte |
| 0x004c5e40 | `physical_object_post_state_net_event_msg` | - | - | - | posts the type-`0x1b` message (§6b) |
| 0x004c5fd0 | `physical_object_net_restart_local_sim` | - | - | - | on gaining ownership, moves the snapshot velocities into the live ones |
| 0x004c6150 | `physical_object_apply_net_event_msg` | - | - | - | slot 38: unpacks a received state into the snapshot band, derives `net_correction_velocity` |
| 0x004c6380 | `physical_object_net_snapshot_state` | - | - | - | latches position, rotation and velocities with `timeGetTime` |
| 0x004c6480 | `physical_object_net_apply_remote_state` | `NetDirect__9C_PhThingFUi` | 0x001aaeb0 | STRUCTURAL | slot 15: extrapolates a remotely simulated body between messages (§7d) |

### 3d. Free helpers

The unit holds no free helper. Its last function, `i3d_frame_set_direction` (`0x004c6b20`), is an
`I3D_frame` method (`structs/i3d_frame.md`): `physical_object_move_wheel` calls it with
`this->frame`, and `camera_apply_desired_pos` with a frame of its own.

## 4. What "Ph" means

| Evidence | What it establishes |
|---|---|
| `CREATE_PHYSICALOBJECT` / `DESTROY_PHYSICALOBJECT`, script command names present in both binaries | the team's designer-facing word for these objects is PHYSICAL OBJECT, the adjective |
| the Mafia Script Data Reference glosses `destroy_physicalobject xx` as "removes physical object xx" | the same reading, from an independent source |
| the sibling class is spelled `C_public_physics` in full | where the team means the noun "physics" it writes it out, so the abbreviated `Ph` carries the adjective |
| `Thing` ≈ object; the source file is `C_PhThings.cpp` (plural) | "physical things", a family, which is what the five subclasses are |
| `fyz` / `Fyz` are absent from both binaries | not a Czech abbreviation |
| the only other `Ph*` symbol is `C_PData_Phase`, where `Ph` starts "Phase" | no competing expansion survives the method set |
| the method set: `MoveBox`, `MoveWheel`, `GetRVelMassJ`, `ExternalImpuls`, `CollisionFilter`, `InitBox`, `InitWheel` | rigid-body dynamics, the domain matches |

The PC prefix states what the class is rather than transliterating the tag.

The two dispatcher cases are byte-confirmed from the game's own keyword table
(`script-mapping/artifacts/opcode-table.md`) and written up in
`script-mapping/artifacts/opcodes-verified.md`. Case `0xcc` (`create_physicalobject ww, xx, yy, zz`
in the Mafia Script Data Reference) always constructs a `C_public_physics` (`operator_new(0x3b0)`);
none of the four arguments selects a class. It initialises the body through
`physical_object_init_box` from an `S_THING_BOX` of mass `max(zz, 1.0)`, initial velocity
`(0, -xx, 0)` and friction `0.45`, and when `yy` is non-zero applies a random impulse scaled by the
mass.

## 5. The five subclasses — a closed set

A census of the base constructor's callers gives a closed set: exactly five classes, each with a
named method set and a body-diffed constructor.

| PC ctor | PC vtable | Kind | Subclass | Evidence |
|---|---|---|---|---|
| `dropout_ctor` (0x0048a530) | 0x0063c320 | `0x11` | `C_DropOut` | `recorder_new_track` prefix `drp%d`; `structs/dropout.md` |
| `crash_obj_ctor` (0x004786c0) | 0x0063bf68 | `0x1a` | `C_crash_obj` | `structs/crash_obj.md` |
| `box_new_ctor` (0x0045f160) | 0x0063baa0 | `0x23` | `C_box_new` | `structs/box_new.md` |
| `public_physics_ctor` (0x004cde20) | 0x0063c9b8 | `0x20` | `C_public_physics` | `structs/public_physics.md`; the `create_physicalobject` class, which turns any scene frame into a physics prop ¹ |
| `drop_in_weapon_ctor` (0x0048a0d0) | 0x0063c280 | `0x25` | `C_drop_in_weapon` | `structs/drop_in_weapon.md` |

¹ `C_public_physics` was identified from the Mafia Script Data Reference, whose 30-object cap for
`create_physicalobject` is the literal `0x1e < size` gate in `public_physics_game_init`. Three of
its vtable slots are shared with `C_drop_in_weapon` by identical-code folding: `0x0063c9b8` slots
13 / 15 / 18 hold `thunk_physical_object_ai_tick`, `thunk_physical_object_net_apply_remote_state`
and `thunk_physical_object_init`, so `C_public_physics` adds nothing to `AI`, `NetDirect` or `Init`.

`C_box` (`structs/box.md`) does the same job without this base: a PC-only rigid-body class (kind 9)
derived straight from `C_actor`, with 55 functions of its own.

## 6. The network replication layer

Six functions, all inside this unit, all with the class base as receiver (byte-proven by an ECX
census of their callsites), none with a PS2 counterpart except the slot-15 member. They form one
loop:

```
physical_object_game_init ─┐
                           ├─► net_snapshot_state          latch pos/quat/velocities + timeGetTime
net_session_transfer_actor_ownership (ownership) ──┘

update / on_act_state_changed ─► broadcast_state_net_event_msg ─► post_state_net_event_msg
                                                                        │  type 0x1b
   slot 38  apply_net_event_msg  ◄─────────────────────────────────────┘
                                 unpacks into the snapshot band, derives the correction velocity
   slot 15  net_apply_remote_state   extrapolates between messages, 5.0-unit snap-back
   net_session_transfer_actor_ownership (gain) ─► net_restart_local_sim
                                 snapshot velocities → live velocities
```

### 6a. How the correction velocity is derived

`physical_object_apply_net_event_msg` computes

```
correction = (remote_pos + peer_latency_ms * remote_vel * 0.001 - local_pos)
             / (session_update_interval_ms * 0.001)
```

reading the latency from `net_session_get_peer_clock_ms` and the interval from
`net_session_game_settings_addr()+0x18`. `physical_object_net_apply_remote_state` then advances the
body on `correction + remote_vel` while the message is fresh and on `remote_vel` alone once it is
stale, snapping back to `net_pos` outright if the error ever exceeds **5.0** units.

### 6b. Message type `0x1b` is shared with `C_box`

`physical_object_post_state_net_event_msg` builds a 0x18-byte message with the descriptor
`g_event_msg_vt_physics_state` (`.rdata 0x0063ba90`) and **type `0x1b`**, the same descriptor and
type `box_post_state_net_event_msg` uses. `event_msg_physics_state_ctor` (`0x005a1860`) is that
message class's generic constructor and takes the type as a parameter, `(this, type, guid, flag)`.
The descriptor's slots 2 / 3 are `event_msg_physics_state_serialize` / `_deserialize`. Payload: the
actor-state base (`{vec3 → i16, 3× i16, u32 guid, u8}`) plus a 16-byte quaternion at `+0x1c`,
three quantised velocity words, and a flag byte at `+0x38` (§2a). Full write-up:
`structs/net_event_msg.md`.

## 7. PC vs PS2

### 7a. Field pairs proven by the body diffs

| PC | PS2 | Field |
|---|---|---|
| `+0x1c` | `+0x5c` | act state |
| `+0x20` | `+0x60` | remotely simulated |
| `+0x24`.. | `+0x00`.. | world position |
| `+0x30`.. | `+0x10`.. | forward axis |
| `+0x68` | `+0x70` | `I3D_frame *` |
| `+0x70` | `+0x590` | `body_type` |
| `+0x75` | `+0x614` | `reset_rest_on_wake` |
| `+0x77` | `+0x615` | `box_configured` |
| `+0x84`.. | `+0x90`.. | `world_centre` |
| `+0x90`.. | `+0xa0`.. | `rest_test_pos` |
| `+0x9c` | `+0x594` | `motion_age_ms` |
| `+0xa0`.. | `+0xb0`.. | world basis rows |
| `+0xb8` | `+0x530` | `tDynamicCollObject` |
| `+0x10c` | `+0xe0` | `world_matrix` |
| `+0x14c` | `+0x120` | `world_matrix_inv` |
| `+0x18c` | `+0x59c` | `max_substep_s` (0.028) |
| `+0x190` | `+0x5a0` | `mass` |
| `+0x194`.. | `+0x1e0`.. | `inv_inertia_diag` |
| `+0x1e0`.. | `+0x1f0`.. | `linear_velocity` |
| `+0x1ec`.. | `+0x200`.. | `angular_velocity` |
| `+0x1fc` | `+0x5a8` | `restitution_factor` |
| `+0x200` | `+0x5ac` | `contact_skin` |
| `+0x204` | `+0x5b0` | `contact_stiffness` |
| `+0x214` | `+0x5b4` | `gravity` (10.0) |
| `+0x218` | `+0x5b8` | `box_mass_x4`: `InitBox` stores `mass * 4.0`, and only the save reads it, as on the PC |
| `+0x21c` | `+0x5bc` | `wheel_width` |
| `+0x220` | `+0x5c0` | `wheel_radius` |
| `+0x2ec` | `+0x5d0` | `rest_lifetime_s` |
| `+0x2f0` | `+0x5d4` | `rest_lifetime_limit_s` |
| `+0x2f4` | `+0x5d8` | `frame_clamp_ratio` |
| `+0x2f8` | `+0x5dc` | `impact_sound` (PS2 vtable `+0x80` `IsPlaying`, `+0x38` `SetOn`) |
| `+0x308` / `+0x30c` | `+0x5e0` / `+0x5e4` | `impact_sound_column` / `collision_material` (both −1-initialised) |
| `+0x310` | `+0x5f4` | `probe_material` |
| `+0x32c` | `+0x610` | `hit_probe_index` |
| `+0x330` | `+0x5e8` | `hit_magnitude` |
| `+0x338` | `+0x612` | `impact_sound_may_restart` |
| `+0x340`.. | `+0x400`.. | the `S_SURR_ACT` vector |

The deltas are not constant (`+0x478`, `+0x410`, `+0x3a0`, `+0x2e4`, `+0xc0`, and several negative):
map fields by role, never by rebasing (`structs/actor.md`). `S_SURR_ACT` is **0x14 bytes on the PC
and 0x20 on the PS2**, with the actor pointer at `+0x0c` / `+0x10`; every walk encodes the stride
literally. The PS2 types it in the template argument `gif::vector<Q29C_PhThing10S_SURR_ACT,8>`,
whose `reserve` (`0x001a82d0`) is the only other `C_PhThing`-scoped PS2 symbol.

### 7b. A 2× tuning delta, seen three times

| What | PC | PS2 | Seen in |
|---|---|---|---|
| age cap before forced sleep | `0xfa1` (4001) | `0x7d1` (2001) | `physical_object_update_rest_state`, `physical_object_ai_tick` |
| `Update`'s own sleep gate | 4000 | 2000 | `physical_object_update` |

Three functions, one ratio: the PC allows twice the motion age before a body is put to sleep.

### 7c. `CollisionCount` is one PS2 method and two PC functions

`physical_object_accumulate_contact_impulse_diag` and `_matrix` have identical 7-parameter
signatures and are selected by `full_inertia` inside `physical_object_move_box`: the diag arm
multiplies by the scalar `+0x194`, the matrix arm goes through `S_vector::operator*(S_matrix *)` on
the `+0x1a0` tensor. The PS2's single `CollisionCount` compiles only the matrix sandwich
(`__amu__(+0x1a0)` → `_vmul(+0x1e0)` → `__amu__(+0x160)`).

| Lever | Points at |
|---|---|
| algorithm, the arm the PS2 compiles | `_matrix` |
| callsite count, 3 on the PS2 and 3 for `_diag` (2 in `move_wheel`, 1 in `move_box`) | `_diag` |
| size ratio, 0.99 and 1.06 | neither |

Both rows carry the same twin, graded STRUCTURAL (Open item 1). The PS2 also rebuilds the sandwich
from `+0x160` / `+0x1a0` / `+0x1e0` at every contact, where the PC precomposes the world tensor once
per step in `physical_object_build_world_inertia_tensor`, which is why that helper has no PS2
counterpart.

### 7d. `NetDirect` — an 8-byte PS2 stub against 1696 PC bytes

`.rdata 0x0063c840` slot 15 holds `physical_object_net_apply_remote_state` (1696 B);
`NetDirect__9C_PhThingFUi` is an empty 8-byte body. The mapping is a slot-role transfer: `C_bottle`'s
vtable `0x0063b870` slot 15 is `bottle_net_apply_remote_state` = `NetDirect__8C_bottleFUi`
(`structs/bottle.md` §9a), same slot, role and signature. An empty body can be matched as a symbol
and never as a body, hence STRUCTURAL. In the replication subsystem a PS2 method count is a lower
bound: the five PC-only members of §6 have no PS2 symbol at all.

### 7e. Three arms the PC body does not have

- `physical_object_collision_filter` — the PS2 also early-outs on `+0x617` and rejects kinds
  `0x13` / `0x1b` / `2` explicitly.
- `physical_object_remove_surrounding_actor` — the PS2 also clears a 14-slot pointer table at
  `+0x4f0`.
- `physical_object_refresh_world_transform` — the PS2 performs a third matrix copy and a zeroing
  tail.

None of these changes what the PC function is; they explain the size ratios. `physical_object_save_game_save`
is 480 B against the PS2's 564 B (ratio 0.85).

### 7f. The PS2 has no `C_box`

The PS2 source manifest has no `C_box.cpp`. Read together, `C_box` is the PC-only rigid body that
does this base's job without it, `C_box_new` is the same feature rebuilt on `C_PhThing`, and the
port shipped only the second.

### 7g. The timer has no PS2 side

`AI__9C_PhThingFUi` has no counterpart of the `timer_elapsed_ms` / `timer_limit_ms` block at the end
of `physical_object_ai_tick`.

## 8. Related

- `structs/actor.md` — the base class.
- `structs/box_new.md`, `structs/dropout.md`, `structs/crash_obj.md`, `structs/public_physics.md`,
  `structs/drop_in_weapon.md` — the five subclasses.
- `structs/box.md` — the PC-only class that does the same job without this base.
- `structs/bottle.md` §9a — the anchor of the `NetDirect` slot-15 role.
- `structs/net_event_msg.md` — the type-`0x1b` message.
- `structs/entity_kind.md` — the kind enum.
- `structs/i3d_frame.md` — `i3d_frame_set_direction`, the unit's foreign member.

## 9. Open items

1. **The `CollisionCount` twin.** Which of `_diag` / `_matrix` is the PS2 method's twin is
   undecided: the algorithm points at `_matrix`, the callsite count at `_diag`, the size ratio at
   neither (§7c).
2. **What the timer `+0x3a8` / `+0x3ac`, `box_mass_x4` `+0x218` and `unused_2e8` were for is not
   recoverable.** A collision, a hit and an ownership restart arm the 3000 ms timer, which
   `physical_object_ai_tick` counts up and clears at expiry; `physical_object_init_box` stores
   `mass * 4.0` and only the save pair touches it. Searched: every `.text` operand with each
   displacement, by whether its base register holds the function's receiver, and the functions
   testing a kind of the five subclasses or calling this family's methods. That search cannot see a
   pointer used with no kind test.
