# `C_box` — the PC-only legacy rigid-body box

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-11-02 17:43 +0100

A `C_actor` subclass carrying **its own complete rigid-body solver**: force accumulation,
semi-implicit integration into a double-buffered state, an OBB-vs-OBB separating-axis test, a
30-slot contact-plane cache, sequential impulse resolution, write-back to the I3D frame,
plus savegame, recorder tracks and network replication. At ~36 KB across 55 function starts it
is the largest single-class translation unit in the actor family. It derives from `C_actor`
directly, not from `C_PhThing`: this class predates the physics base and duplicates its job
(§8c).

`C_box` is not dead code: it has a live vtable, a kind value the recorder formats, a definition
chunk id, and a matching network sender and receiver. **The class NAME is inferred; the PREFIX is
not.** `box_` rests on the developers' own string: `recorder_new_track` names this kind's recorder
tracks `box%d` (`s_box_d_0064c9e0`). `C_box` as the class name rests on the link-order argument
(§8b); if the name were falsified the prefix would still stand.

| | |
|---|---|
| PC prefix | `box_` |
| PC kind (`+0x10`) | `9`, stamped by `box_ctor` |
| PC vtable | `0x0063b980`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0xcec` (`operator_new(0xcec)` in `mission_create_actor` case `9`) |
| PC constructor | `box_ctor` @ `0x0045ae00` |
| PC translation unit | `[0x00456460, 0x0045f130)` (§7) |

The class does not exist in the PS2 build (§8a), so the card has no PS2 row, the method tables
have no PS2 columns, and every method is graded STRUCTURAL: a CONFIRMED tag would assert a body
agreement that cannot exist.

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC. Two functions are the
backbone: `box_init` seeds every tunable with a literal, and `box_accumulate_forces` consumes the
mass, the damping, the friction and the inertia in one expression.

```c
typedef struct C_box {                      // 0xcec bytes; PC offsets
    /* +0x000 */ C_actor     base;              // [H] the C_actor band (structs/actor.md §1)
    /* +0x070 */ float       half_extent[3];    // [H] x, y, z from the frame's bound box
    /* +0x07c */ int32_t     contact_result;    // [H] what the last contact pass found (§2c)
    /* +0x080 */ int32_t     state_read_idx;    // [H] index of the live state record
    /* +0x084 */ int32_t     state_write_idx;   // [H] index of the back state record
    /* +0x088 */ S_vector    corner_local[8];   // [H] the body's corners in local space
    /* +0x0e8 */ C_box      *resting_on[5];     // [H] boxes that came to rest on this one
    /* +0x0fc */ int32_t     resting_count;     // [H] entries of resting_on
    /* +0x100 */ uint8_t     unread_100;        // [U] box_game_init stores 1; no reader (.text scan)
    /* +0x104 */ int32_t     sleep_counter;     // [H] 5 on any external impulse
    /* +0x108 */ int32_t     probe_tick;        // [H] counts substeps; its parity picks the collision probe
    /* +0x10c */ int32_t     ground_contact_slot; // [H] contact slot of the ground under the box; -1 none
    /* +0x110 */ float       inv_mass;          // [H] every impulse denominator
    /* +0x114 */ float       mass;              // [H] volume × density (§4)
    /* +0x118 */ float       inv_inertia[3];    // [H] the inverse of inertia_diag, per axis
    /* +0x124 */ float       inertia_diag[3];   // [H] the solid's inertia, per axis (§4)
    /* +0x130 */ S_box_state state[3];          // [H] the body state; two records are indexed (§1a)
    /* +0x388 */ float       linear_damping;    // [H] 0.32 by box_init
    /* +0x38c */ float       angular_damping;   // [H] 0.5 by box_init
    /* +0x390 */ S_vector    force_accum;       // [H] the force of the pending impulses
    /* +0x39c */ S_vector    torque_accum;      // [H] r x F of the same impulses
    /* +0x3a8 */ uint8_t     friction_active;   // [H] the last step resolved impacts; gates the friction
    /* +0x3ac */ I3D_frame  *collided_human_frame; // [M] frame of the last human that bumped it; no reader (.text scan)
    /* +0x3b0 */ int32_t     step_counter;      // [H] bumped by box_integrate_state; every third clamps the basis
    /* +0x3b4 */ S_vector    contact_normal[30]; // [H] the contact ring (§5): plane normal, zero = free
    /* +0x51c */ float       contact_plane_d[30]; // [H] the plane constant of each slot
    /* +0x594 */ C_actor    *contact_actor[30]; // [H] the actor touched there; 0 = static
    /* +0x60c */ int32_t     contact_cursor;    // [H] ring head, modulo 30
    /* +0x610 */ int32_t     contact_count;     // [H] live ring entries, at most 30
    /* +0x614 */ S_vector    corner_world_prev[8]; // [H] corner_local under the pose before the step
    /* +0x674 */ S_vector    corner_world_next[8]; // [H] corner_local under the pose after the step
    /* +0x6d4 */ int32_t     impact_slot[50];   // [H] the contact slot of each queued impact
    /* +0x79c */ S_vector    impact_point[50];  // [H] the world point of each queued impact
    /* +0x9f4 */ int32_t     impact_count;      // [H] entries of impact_slot and impact_point
    /* +0x9f8 */ float       impact_fraction;   // [H] -1.0 seed; the earliest crossing as a fraction of the step
    /* +0x9fc */ S_vector    support_point[30]; // [H] points box_accumulate_forces spreads over
    /* +0xb64 */ int32_t     support_count;     // [H] entries of support_point
    /* +0xb68 */ uint8_t     awake;             // [H] 0 = asleep; set by every wake path
    /* +0xb69 */ uint8_t     just_woke;         // [H] read by box_track_record_key
    /* +0xb6c */ uint32_t    unused_b6c[18];    // [U] no access; no reader (.text scan)
    /* +0xbb4 */ S_matrix    dyn_world;         // [H] the dyn* frame's world matrix, scale divided out
    /* +0xbf4 */ S_matrix    dyn_world_inv;     // [H] the inverse of dyn_world
    /* +0xc34 */ tDynamicCollObject dyn_coll;   // [H] 0x12 dwords; the collision proxy
    /* +0xc7c */ I3D_frame  *dyn_coll_frame;    // [H] the dyn* frame it was built from
    /* +0xc80 */ float       density;           // [H] 50.0; mass = volume × density (§4)
    /* +0xc84 */ float       friction;          // [H] 0.4; the contact friction coefficient
    /* +0xc88 */ float       rolling_friction;  // [M] 2.5; read only for a cylinder, §2
    /* +0xc8c */ float       restitution;       // [H] 0.6; every solver uses -(e + 1)
    /* +0xc90 */ uint8_t     is_cylinder;       // [H] set = an upright cylinder (§4); 0 in play
    /* +0xc94 */ float       bound_radius;      // [H] |half_extent|
    /* +0xc98 */ uint32_t    unused_c98;        // [U] no access; no reader (.text scan)
    /* +0xc9c */ S_vector    net_pos;           // [H] the replication band (§6)
    /* +0xca8 */ S_vector    net_lin_vel;       // [H] linear velocity of the replication band
    /* +0xcb4 */ S_quat      net_rot;           // [H] rotation of the replication band
    /* +0xcc4 */ S_quat      unread_cc4;        // [U] box_game_init stores the frame's rotation; no reader (.text scan)
    /* +0xcd4 */ int32_t     ms_since_state_msg; // [H] adds the frame time; cleared by the sender
    /* +0xcd8 */ uint32_t    net_flags;         // [H] bit map in §2b
    /* +0xcdc */ int32_t     unread_cdc;        // [U] box_init stores 0; no reader (.text scan)
    /* +0xce0 */ uint32_t    unused_ce0[3];     // [U] no access; no reader (.text scan)
} C_box;
```

The typedef lists every band of four bytes or more; a shorter gap that no instruction touches is left out.

**Why these arrays, and what each entry is.**

| Field | Count | Entry |
|---|---|---|
| `half_extent[3]`, `inv_inertia[3]`, `inertia_diag[3]` | 3 | one per body axis x, y, z |
| `corner_local[8]`, `corner_world_prev[8]`, `corner_world_next[8]` | 8 | the eight corners of the box. `box_ai_tick` runs two identical loops over `corner_local`, transforming each corner with two different matrices into the two world arrays (stride `0xc`); the loop count is 8, or 2 for a cylinder (§4), and the copy length `count × 12`, so each array holds 8 vec3 — `+0x614 + 8 × 0xc = +0x674`, and `+0x674` ends at `+0x6d4`. `box_resolve_contacts` indexes `corner_world_next` with `ecx*4` (three floats per corner) |
| `resting_on[5]` | 5 | room for five pointers between `+0xe8` and `resting_count`; `box_ai_tick` appends to the box underneath while its count is under 5, and `box_wake_resting_bodies` walks and clears them |
| `state[3]` | 3 | the read and the write buffer of the integrator, selected by `state_read_idx` / `state_write_idx`, and a third record neither index reaches (§1a) |
| `contact_normal[30]`, `contact_plane_d[30]`, `contact_actor[30]` | 30 | one cached contact plane per slot of the ring (§5): `box_contact_slot_acquire` stores the normal, the plane constant and its fourth argument, the collision object's owner (`+0x44`), at the same index |
| `impact_slot[50]`, `impact_point[50]` | 50 | one queued impact per entry: both contact passes append while `impact_count` is under `0x32`, and the regions tile exactly, `+0x6d4 + 50 × 4 = +0x79c` and `+0x79c + 50 × 0xc = +0x9f4` |
| `support_point[30]` | 30 | the region between `+0x9fc` and `support_count` holds exactly 30 vec3; `box_accumulate_forces` loops over `support_count` of them, taking each point minus the centre of mass. `box_ai_tick` writes them from `impact_point` in the arm of `box_solve_impulse_vs_static` |

### 1a. `S_box_state` — the double-buffered state record

`box_ctor` constructs three of these at stride `0xc8`, and they fill `+0x130..+0x387` exactly.
`box_integrate_state` reads the one indexed by `state_read_idx` and writes the one indexed by
`state_write_idx`; `box_write_state_to_frame` publishes whichever it is handed. Both indices only
take 0 and 1, so the third record is reached by two instructions only: `box_game_init` writes the
identity to its `inv_inertia_world`, and `box_ai_tick` clamps its `basis` at `+0x2cc`.

```c
typedef struct S_box_state {                // 200 (0xc8) bytes; C_box::state element
    /* +0x00 */ S_vector pos;                   // [H] the centre of mass
    /* +0x0c */ float    basis[16];             // [H] the 4×4 orientation
    /* +0x4c */ S_vector lin_vel;               // [H] linear velocity
    /* +0x58 */ S_vector ang_momentum;          // [H] angular momentum
    /* +0x64 */ S_vector force;                 // [H] the accumulated force
    /* +0x70 */ S_vector torque;                // [H] the accumulated torque
    /* +0x7c */ S_matrix inv_inertia_world;     // [H] the inverse inertia in world space
    /* +0xbc */ S_vector ang_vel_world;         // [H] angular velocity, world
} S_box_state;
```

`basis[16]` is one 4×4 matrix (four rows of four floats).

## 2. Field evidence

Measured by a census of every operand with each displacement in the `box_*` functions, then read
in the bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `box_ctor` (through `actor_ctor`; kind `+0x10` = 9) | `folded_get_dword_0x10` (the kind); `box_init` (rejects a frame whose type is not 9); the methods of §3 (`+0x20`, `+0x24`, the frame `+0x68`) | `+0x20` (this machine is not authoritative) gates the replication mirror; `box_init` sets `may_broadcast` `+0x6d`, which `box_ai_tick` tests before a state message; `box_game_init` refreshes the position `+0x24` |
| +0x070 | half_extent | H | `box_recompute_dimensions_from_frame` | `box_accumulate_forces`, `box_ai_tick`, `box_collect_contacts`, `box_game_init`, `box_generate_contacts_vs_box`, `box_integrate_state`, `box_resolve_contacts`, `box_solve_impulse_vs_box`, `box_solve_impulse_vs_car` | §4 |
| +0x07c | contact_result | H | `box_collect_contacts`, `box_resolve_contacts` | `box_ai_tick`, `box_collect_contacts`, `box_resolve_contacts` | §2c |
| +0x080 | state_read_idx | H | `box_init` (= 0), `box_ai_tick`, `box_apply_net_event_msg` | `box_add_impulse_at_point`, `box_ai_tick`, `box_apply_impulse_pair_at_point`, `box_collect_contacts`, `box_game_init`, `box_get_contact_state_at_point`, `box_integrate_state`, `box_on_collision`, `box_solve_impulse_vs_box`, `box_track_record_key` | - |
| +0x084 | state_write_idx | H | `box_init` (= 1), `box_ai_tick` | `box_ai_tick`, `box_game_init`, `box_integrate_state` | - |
| +0x088 | corner_local | H | `box_init` (zeroed), `box_game_init` (from the frame's bound box) | `box_ai_tick`, `box_intersect_segment_side_faces` | eight corners, or two points on the axis for a cylinder (§4) |
| +0x0e8 | resting_on | H | `box_ai_tick` (on the box underneath) | `box_wake_resting_bodies` | - |
| +0x0fc | resting_count | H | `box_ai_tick`, `box_game_init` (= 0), `box_wake_resting_bodies` (= 0) | `box_ai_tick`, `box_wake_resting_bodies` | - |
| +0x100 | unread_100 | U | `box_game_init` (= 1) | - | no byte load at this offset in `.text`; dword loads there are other classes |
| +0x104 | sleep_counter | H | `box_add_impulse_at_point`, `box_apply_impulse_pair_at_point` (= 5), `box_ai_tick`, `box_game_init` | `box_ai_tick` | - |
| +0x108 | probe_tick | H | `box_init` (= 0), `box_ai_tick` (+1 per sweep, 0 when a solver drops a contact) | `box_ai_tick` | zero or odd runs `collision_test_sphere_slide`; zero or even runs the vertical ground line |
| +0x10c | ground_contact_slot | H | `box_init` (= -1), `box_ai_tick` | `box_accumulate_forces`, `box_ai_tick` | the slot from `box_contact_slot_acquire` for the ground line; -1 = none |
| +0x110 | inv_mass | H | `box_recompute_dimensions_from_frame` | `box_apply_impulse_pair_at_point`, `box_build_dyn_collision_from_frame`, `box_init`, `box_integrate_state`, `box_solve_impulse_vs_box`, `box_solve_impulse_vs_static`, `box_solve_impulse_vs_car` | - |
| +0x114 | mass | H | `box_recompute_dimensions_from_frame` | `box_accumulate_forces` (gravity = `g_box_gravity × mass`), `box_ai_tick`, `box_get_contact_state_at_point`, `box_on_collision`, `box_solve_impulse_vs_box`, `box_solve_impulse_vs_car` | §4 |
| +0x118 | inv_inertia | H | `box_recompute_dimensions_from_frame` | `box_integrate_state` | for the box, `3 / ((b² + c²) × m)` per axis |
| +0x124 | inertia_diag | H | `box_recompute_dimensions_from_frame` | `box_accumulate_forces` (the angular damping term) | §4 |
| +0x130 | state | H | `box_ctor` (constructs three), `box_game_init` (both indexed records from the frame), `box_integrate_state` (the write record), `box_apply_impulse_pair_at_point`, `box_accumulate_forces` | the three solvers, `box_integrate_state` (the read record), `box_write_state_to_frame`, `box_get_contact_state_at_point` | §1a |
| +0x388 | linear_damping | H | `box_init` (= 0.32, `0x3ea3d70a`) | `box_accumulate_forces` | the force `-linear_damping × mass × lin_vel` |
| +0x38c | angular_damping | H | `box_init` (= 0.5) | `box_accumulate_forces` | - |
| +0x390 | force_accum | H | `box_add_impulse_at_point` | `box_add_impulse_at_point`, `box_accumulate_forces` | the public "push this box" producer and its consumer |
| +0x39c | torque_accum | H | `box_add_impulse_at_point` | `box_add_impulse_at_point`, `box_accumulate_forces` | `r × F` |
| +0x3a8 | friction_active | H | `box_init`, `box_add_impulse_at_point`, `box_apply_impulse_pair_at_point` (= 0), `box_ai_tick` (0 each step, 1 when the step resolved impacts) | `box_accumulate_forces` | clear: no friction term this step |
| +0x3ac | collided_human_frame | M | `box_ctor` (= 0), `box_on_collision` (kinds 2, 3, `0x1b`: the actor's frame) | - | no load on a `C_box` (`.text` scan), so the name rests on its writer alone |
| +0x3b0 | step_counter | H | `box_game_init`, `box_integrate_state` | `box_integrate_state` | every third step clamps the basis |
| +0x3b4 | contact_normal | H | `box_contact_slot_acquire`, `box_contact_slot_release` (zeroes) | `box_contact_slot_acquire`, `box_collect_contacts`, `box_resolve_contacts`, `box_ai_tick` (and hands it to the solvers) | a slot is free when its normal is (0, 0, 0) |
| +0x51c | contact_plane_d | H | `box_contact_slot_acquire` | `box_contact_slot_acquire` (matched to 0.01), `box_ai_tick`, `box_collect_contacts`, `box_resolve_contacts` | a point's distance from the plane is `dot(normal, p) + d` |
| +0x594 | contact_actor | H | `box_contact_slot_acquire` | `box_ai_tick`, `box_collect_contacts` | `box_collect_contacts` runs `box_generate_contacts_vs_box` when the actor's kind is 9 or 4 |
| +0x60c | contact_cursor | H | `box_contact_slot_acquire`, `box_game_init` | `box_ai_tick`, `box_collect_contacts`, `box_contact_slot_acquire`, `box_contact_slot_release` | - |
| +0x610 | contact_count | H | `box_contact_slot_acquire` (capped at 30), `box_ai_tick`, `box_game_init` | `box_ai_tick`, `box_collect_contacts`, `box_contact_slot_acquire`, `box_contact_slot_release` | - |
| +0x614 | corner_world_prev | H | `box_ai_tick` (a completed step copies `corner_world_next` here) | `box_collect_contacts` | - |
| +0x674 | corner_world_next | H | `box_ai_tick` | `box_ai_tick`, `box_collect_contacts`, `box_resolve_contacts` | - |
| +0x6d4 | impact_slot | H | `box_collect_contacts`, `box_resolve_contacts` | `box_ai_tick` | - |
| +0x79c | impact_point | H | `box_collect_contacts`, `box_resolve_contacts` | `box_ai_tick` | - |
| +0x9f4 | impact_count | H | `box_init` (= 0), `box_collect_contacts`, `box_resolve_contacts` | `box_ai_tick`, `box_collect_contacts`, `box_resolve_contacts` | at most `0x32` |
| +0x9f8 | impact_fraction | H | `box_collect_contacts` (-1.0 seed, keeps the minimum of `a / (a - b)`) | `box_ai_tick` (shortens the step to it), `box_collect_contacts` | `a` and `b` are a corner's distances from the plane before and after the step |
| +0x9fc | support_point | H | `box_ai_tick` | `box_accumulate_forces` | - |
| +0xb64 | support_count | H | `box_ai_tick` | `box_accumulate_forces`, `box_ai_tick` | the force is divided by it, one share per point. `box_ai_tick` zeroes it before each queued impact, so it ends at 0 or 1 |
| +0xb68 | awake | H | `box_add_impulse_at_point`, `box_apply_impulse_pair_at_point`, `box_ai_tick`, `box_game_init`, `box_wake_resting_bodies` | `box_add_impulse_at_point`, `box_ai_tick`, `box_apply_impulse_pair_at_point`, `box_track_record_key`, `box_wake_resting_bodies` | - |
| +0xb69 | just_woke | H | `box_add_impulse_at_point`, `box_ai_tick`, `box_apply_impulse_pair_at_point`, `box_game_init`, `box_track_record_key`, `box_wake_resting_bodies` | `box_track_record_key` | - |
| +0xb6c | unused_b6c | U | - | - | no access on a `C_box` (`.text` scan) |
| +0xbb4 | dyn_world | H | `box_build_dyn_collision_from_frame`, `box_ai_tick`, `box_apply_net_event_msg`, `box_net_apply_remote_state` | `collision_line_in_dynamics` (through `dyn_coll +0x3c`) | - |
| +0xbf4 | dyn_world_inv | H | `box_build_dyn_collision_from_frame`, `box_ai_tick`, `box_apply_net_event_msg`, `box_net_apply_remote_state` | `collision_line_in_dynamics` (through `dyn_coll +0x40`) | - |
| +0xc34 | dyn_coll | H | `box_ctor` (zeroed), `box_build_dyn_collision_from_frame` | `box_ai_tick`, `box_game_init`, `box_game_done` (`collision_dyn_erase`) | - |
| +0xc7c | dyn_coll_frame | H | `box_build_dyn_collision_from_frame` | `box_ai_tick`, `box_apply_net_event_msg`, `box_net_apply_remote_state` | the frame whose name matches `s_dyn_0064b5e8` (`dyn`, a 3-character compare) |
| +0xc80 | density | H | `box_init` (= 50.0), `box_recompute_dimensions_from_frame` (1.0 when ≤ 0) | `box_recompute_dimensions_from_frame` | §4 |
| +0xc84 | friction | H | `box_init` (= 0.4), `box_recompute_dimensions_from_frame` (0.1 when ≤ 0) | `box_accumulate_forces` | the friction force is `gravity × mass × friction` |
| +0xc88 | rolling_friction | M | `box_init` (= 2.5), `box_recompute_dimensions_from_frame` (0.1 when ≤ 0) | `box_accumulate_forces` | read only in the `is_cylinder` arm, as `(0.01 / half_extent.x) × rolling_friction × gravity × mass`. Inferred from that form; a shipped record with `is_cylinder` set would promote it |
| +0xc8c | restitution | H | `box_init` (= 0.6), `box_recompute_dimensions_from_frame` (0.1 when ≤ 0) | `box_apply_impulse_pair_at_point`, `box_solve_impulse_vs_box`, `box_solve_impulse_vs_static`, `box_solve_impulse_vs_car` | - |
| +0xc90 | is_cylinder | H | `box_init` (= 0) | `box_accumulate_forces`, `box_ai_tick`, `box_intersect_segment_side_faces`, `box_collect_contacts`, `box_game_init`, `box_on_collision`, `box_recompute_dimensions_from_frame`, `box_resolve_contacts` | §4 |
| +0xc94 | bound_radius | H | `box_game_init` | `box_ai_tick` | - |
| +0xc98 | unused_c98 | U | - | - | no access on a `C_box` (`.text` scan) |
| +0xc9c | net_pos | H | `box_game_init` (seed), `box_write_state_to_frame` (mirror), `box_apply_net_event_msg`, `box_net_apply_remote_state` | `box_net_apply_remote_state` | §6 |
| +0xca8 | net_lin_vel | H | `box_game_init`, `box_apply_net_event_msg` | `box_net_apply_remote_state` | - |
| +0xcb4 | net_rot | H | `box_game_init`, `box_write_state_to_frame`, `box_apply_net_event_msg` | `box_net_apply_remote_state` | - |
| +0xcc4 | unread_cc4 | U | `box_game_init` (the frame's rotation) | - | no load on a `C_box` (`.text` scan) |
| +0xcd4 | ms_since_state_msg | H | `box_init` (= 0), `box_ai_tick` (adds the frame time), `box_post_state_net_event_msg` (= 0) | `box_ai_tick` | past the interval at session settings `+0x1c` a state message falls due |
| +0xcd8 | net_flags | H | `box_init`, `box_on_collision`, `box_hit`, `box_ai_tick`, `box_apply_net_event_msg`, `box_net_apply_remote_state` | the same functions | §2b |
| +0xcdc | unread_cdc | U | `box_init` (= 0) | - | of the 23 `disp32 0xcdc` operands in `.text` this is the only one on a `C_box`; the others are `this` of `C_vehicle` |
| +0xce0 | unused_ce0 | U | - | - | no access on a `C_box` (`.text` scan) |

### 2a. `S_box_state`

The record is reached through `this + 0x130 + index × 200`, so the displacement census cannot see
its accesses; the rows below come from the bodies named in them.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | `box_game_init` | the three solvers (every `r = point - com`) | - |
| +0x0c | basis | H | `box_game_init` (copied wholesale), `box_integrate_state`, `box_clamp_state_matrix` (clamped to ±1000) | `box_integrate_state`, `box_collect_contacts`, `box_resolve_contacts`, `box_write_state_to_frame` | - |
| +0x4c | lin_vel | H | `box_apply_impulse_pair_at_point` | `box_on_collision`, `box_ai_tick` | - |
| +0x58 | ang_momentum | H | `box_game_init`, `box_ai_tick` (= 0), `box_integrate_state`, `box_apply_impulse_pair_at_point`, the three solvers | `box_integrate_state`, the three solvers | `torque × dt` adds to it with no inertia term; times `inv_inertia_world` it gives `ang_vel_world` |
| +0x64 | force | H | `box_accumulate_forces` | `box_integrate_state` | adds `force × inv_mass × dt` to `lin_vel` |
| +0x70 | torque | H | `box_accumulate_forces` | `box_integrate_state` | adds `torque × dt` to `ang_momentum` |
| +0x7c | inv_inertia_world | H | `box_game_init` (identity, all three records), `box_integrate_state` | `box_integrate_state`, the three solvers, `box_apply_impulse_pair_at_point`, `box_get_contact_state_at_point` | built from `basis` and `inv_inertia` |
| +0xbc | ang_vel_world | H | `box_integrate_state`, `box_apply_impulse_pair_at_point`, the solvers (through `S_vector::operator*` after every impulse) | `box_ai_tick`, `box_integrate_state`, `box_accumulate_forces`, `box_get_contact_state_at_point` | - |

### 2b. `net_flags` (`+0xcd8`) — bit map

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | collided this frame — the state is dirty, broadcast it | set by `box_on_collision` |
| `0x2` | the act state arrived as 1 | set by `box_apply_net_event_msg` on type `0x13` when payload word 1 is 1 (§6); no instruction tests it |
| `0x8` | a remote state arrived | set by `box_apply_net_event_msg` on type `0x1b` |
| `0x10` | the remote body is asleep | set from the payload's sleep byte, cleared when awake |
| `0x20` | first state message pending | part of the `box_init` value below; `box_ai_tick` posts flag byte 0 and clears it |
| `0xfffffffe` | every bit but 0 | the value `box_init` ORs in at construction |

### 2c. `contact_result` (`+0x07c`) — values

| Value | Meaning | Evidence |
|---|---|---|
| 0 | a corner crossed a contact plane during the step | `box_collect_contacts`; `box_ai_tick` re-integrates up to `impact_fraction` and runs `box_resolve_contacts` |
| 1 | a corner touches a plane and moves into it; impacts are queued | both contact passes; `box_ai_tick` runs one solver per `impact_slot` entry |
| 2 | nothing to act on | the seed of both contact passes |
| 3 | a corner lay below the plane before and after the step | `box_collect_contacts`; `box_ai_tick` pushes the body out by `g_box_depenetration_offset` and repeats the step |

## 3. Method family

Coverage: the class has **no PS2 counterpart** (§8a). Every row is STRUCTURAL: a PC body read
plus a byte-proven anchor — a vtable slot role transferred across three sibling classes, a
field-offset and callsite census, or a proven internal call chain. Slots 0 and 9 implement the
`C_actor_track` recorder interface (`structs/actor_track.md`).

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00456460 | `crt_static_init_guard_00456460` | the translation unit | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00456480 | `crt_atexit_register_00456480` | the translation unit | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00456490 | `thunk_box_static_init_gravity` | `g_box_gravity` | - | the `.CRT$XCU` entry (`0x00646054`): a 5-byte `jmp` to the initializer below ¹ |
| 0x004564a0 | `box_static_init_gravity` | `g_box_gravity` | STRUCTURAL | seeds the gravity `(0, -9.81, 0)` |
| 0x00458c30 | `thunk_stub_noop_cdecl_00458c40` | the translation unit | - | the `.CRT$XCU` entry `0x00646058` ¹ |
| 0x00458c40 | `stub_noop_cdecl_00458c40` | the translation unit | - | an empty dynamic initializer of a file-scope object |
| 0x00458c50 | `thunk_stub_noop_cdecl_00458c60` | the translation unit | - | the `.CRT$XCU` entry `0x0064605c` ¹ |
| 0x00458c60 | `stub_noop_cdecl_00458c60` | the translation unit | - | an empty dynamic initializer of a file-scope object |
| 0x0045ae00 | `box_ctor` | instance | STRUCTURAL | `actor_ctor`, kind 9, vtable `0x0063b980` |
| 0x0045ae80 | `box_scalar_deleting_dtor` | instance | STRUCTURAL | vtable slot 16 |
| 0x0045b300 | `box_dtor` | instance | STRUCTURAL | restores the vtable, then `actor_dtor` |

¹ Ghidra names a thunk after its target, so the three thunks carry the ids `thunk_FUN_004564a0`,
`thunk_FUN_00458c40` and `thunk_FUN_00458c60`, not `FUN_` ids of their own.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x004564c0 | `box_wake_resting_bodies` | STRUCTURAL | wakes the stack resting on this box and clears `resting_count` |
| 0x00456530 | `box_add_impulse_at_point` | STRUCTURAL | the public "push this box" entry: accumulates force and torque, wakes the body |
| 0x00456630 | `box_get_contact_state_at_point` | STRUCTURAL | public query used by `car_get_collision_info`; indexes the class's own state record ¹ |
| 0x00456720 | `box_solve_impulse_vs_box` | STRUCTURAL | the two-body contact solver |
| 0x00456b00 | `box_solve_impulse_vs_car` | STRUCTURAL | the solver against a car: the car's terms come from `vehicle_get_contact_impulse_terms` on its `C_Vehicle` at `+0x70`; the kind-4 arm of `box_ai_tick` is its only caller |
| 0x00456e60 | `box_apply_impulse_pair_at_point` | STRUCTURAL | applies a resolved impulse to the state |
| 0x00456fb0 | `box_on_collision` | STRUCTURAL | slot 30, `Collision`; sets `net_flags` bit 0 |
| 0x00457350 | `box_hit` | STRUCTURAL | slot 31, `Hit` |
| 0x00457540 | `box_accumulate_forces` | STRUCTURAL | gravity, damping and friction into the accumulators |
| 0x004585b0 | `box_integrate_state` | STRUCTURAL | the integrator, read buffer → write buffer |
| 0x00458830 | `box_solve_impulse_vs_static` | STRUCTURAL | the one-body solver: a slot with no actor, an actor of another kind, or the ground slot |
| 0x00458a60 | `box_contact_slot_acquire` | STRUCTURAL | the contact cache, acquire (§5) |
| 0x00458bc0 | `box_contact_slot_release` | STRUCTURAL | the contact cache, release |
| 0x00458c70 | `box_intersect_segment_side_faces` | STRUCTURAL | clips a segment against the four faces around the box's Y axis and returns the point and the normal; 0 for a cylinder. Its only caller is `box_on_collision` |
| 0x00459820 | `box_generate_contacts_vs_box` | STRUCTURAL | contact generation against another box |
| 0x00459d30 | `box_collect_contacts` | STRUCTURAL | tests the corners against the ring: fills the resolve scratch list (§5) and the impact queue, returns `contact_result` |
| 0x0045a8c0 | `box_resolve_contacts` | STRUCTURAL | re-tests the listed contacts after the shortened step and rebuilds the impact queue; `box_ai_tick` applies the impulses |
| 0x0045aeb0 | `box_init` | STRUCTURAL | slot 18, `Init`; rejects a frame whose type is not 9; seeds every tunable |
| 0x0045afb0 | `box_recompute_dimensions_from_frame` | STRUCTURAL | clamps the tunables, derives the half-extents, the mass and the inertia (§4) |
| 0x0045b360 | `box_build_dyn_collision_from_frame` | STRUCTURAL | builds `dyn_coll` from the `dyn*` frame |
| 0x0045baf0 | `box_game_init` | STRUCTURAL | slot 25, `GameInit` |
| 0x0045c0d0 | `box_game_done` | STRUCTURAL | slot 26, `GameDone` |
| 0x0045c0f0 | `box_save_data` | STRUCTURAL | slot 19, `SaveData`; the definition record, chunk id `0x11` (§4) |
| 0x0045c150 | `box_load_data` | STRUCTURAL | slot 20, `LoadData` |
| 0x0045c240 | `box_ai_tick` | STRUCTURAL | slot 13; the physics tick — the unit's apex |
| 0x0045dcf0 | `box_write_state_to_frame` | STRUCTURAL | publishes a state record to the I3D frame |
| 0x0045ddf0 | `box_track_create` | STRUCTURAL | slot 9; `recorder_add_track(0x28, 0x28, 0x30)` |
| 0x0045de10 | `box_track_record_key` | STRUCTURAL | slot 0; records a key; -1 = nothing recorded |
| 0x0045e1d0 | `box_clamp_state_matrix` | STRUCTURAL | clamps a 4×4 state matrix to ±1000 |
| 0x0045e220 | `box_post_state_net_event_msg` | STRUCTURAL | the **sender**, message type `0x1b` (§6) |
| 0x0045e280 | `box_apply_net_event_msg` | STRUCTURAL | slot 38; the **receiver**, types `0x13` and `0x1b` (§6) |
| 0x0045e5a0 | `box_net_apply_remote_state` | STRUCTURAL | slot 15, `NetDirect`; dead reckoning |
| 0x0045f0f0 | `stub_zero_vec3_out` | - | slot 4 of two vtables (`0x0063b720` `C_actor_shot` and `0x0063b980` `C_box`): an ICF fold of two classes' identical overrides, owned by neither (`structs/actor_track.md` §4b) |
| 0x0045f110 | `stub_zero_quat_out` | - | slot 5 of the same two vtables, the same fold |
| 0x00461180 | `stub_noop_args4_00461180` | - | slot 1, the `C_actor_track` key replay: an empty override (the box replays no key flags); a generic no-op shared by 21 vtables |
| 0x004776f0 | `stub_noop_args4_004776f0` | - | slot 21, `DuplicateFrom`: an empty override, so a script duplicate copies no box state; shared by 87 vtables |
| 0x0048aaf0 | `folded_track_apply_rigid_pose_at_clock` | - | slot 2, the `C_actor_track` pose playback: an ICF fold shared with slot 2 of `C_DropOut` (`0x0063c320`), owned by neither (`structs/dropout.md` §6b, `structs/actor_track.md` §4b) |

¹ An external caller does not make it a free function: it indexes `this+0x130 +
this[0x80] × 200`, the class's own state block.

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00458000 | `vec3_mul_componentwise` | - | no receiver; its only caller is `box_accumulate_forces` |
| 0x00458030 | `matrix3_rotate_by_angular_velocity` | - | no receiver; its only caller is `box_integrate_state` |
| 0x004580a0 | `matrix4_from_transposed_basis` | - | no receiver; callers `box_collect_contacts`, `box_resolve_contacts` |
| 0x00458130 | `matrix3_orthonormalize` | - | no receiver; callers `box_integrate_state`, `box_game_init` |
| 0x004592f0 | `obb_vs_obb_separating_axis_test` | - | the 15-axis SAT; its only caller is `box_generate_contacts_vs_box` |
| 0x00459cf0 | `vec3_equals` | - | 4 callsites in 3 functions: `box_generate_contacts_vs_box` ×2, `physical_object_move_wheel`, `car_net_apply_remote_state` |
| 0x0045aea0 | `identity_thunk` | - | a shared identity thunk, 67 callsites in 14 functions |

## 4. Mass, inertia and the cylinder flag

`box_recompute_dimensions_from_frame` computes the mass and the inertia from the half-extents
`hx`, `hy`, `hz`:

| `is_cylinder` | Mass | Inertia |
|---|---|---|
| 0 | `8 · hx · hy · hz · density`, the box volume times the density | `m · (a² + b²) / 3` per axis, the solid box |
| set | `2π · r² · hy · density`, with `r` the mean of `hx` and `hz`, which replaces both | `m · (3r² + 4hy²) / 12` about the two cross axes |

Both branches multiply a volume by `density`, which is why that field is a density (default
50.0). The other readers follow the same split: `box_ai_tick` transforms two corner points
instead of eight (§1), and `box_intersect_segment_side_faces`, `box_accumulate_forces`,
`box_on_collision`, `box_collect_contacts`, `box_resolve_contacts` and `box_game_init` branch on
the flag.

**The flag is 0 in shipped play.** The five fields `+0xc80..+0xc90` form one `0x14`-byte parameter
block: `box_init` writes the defaults 50.0, 0.4, 2.5, 0.6 and 0,
`box_recompute_dimensions_from_frame` clamps the four floats, and `box_save_data` /
`box_load_data` copy the whole block in the definition chunk `0x11`. No other instruction in
`.text` writes `+0xc90`, so the cylinder branches run only from a definition record that carries a
non-zero byte.

## 5. The contact cache and the solver's scratch arrays

**The contact cache — 30 slots of contact planes.** `box_contact_slot_acquire` searches the ring
`(cursor + i) % 30` for a slot **whose normal is within 1e-4 (squared) of the given normal and whose
plane constant is within 0.01 of the given constant** and returns that slot; only if none matches
does it reuse an all-zero slot or append. A slot holds the plane and the actor and no impulse.
`box_contact_slot_release` zeroes a slot's normal by index.

**The scratch globals.** The unit owns 16 globals in one contiguous band that tiles with no gaps
(only two 4-byte alignment pads):

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_box_resolve_count` | `0x00655378` | the resolve-list count, capped at `0x28` (40) | `box_collect_contacts` | `box_collect_contacts`, `box_resolve_contacts` (the loop bound) |
| `g_box_resolve_slot_index` | `0x00655380` | `int[40]`: the contact-ring slot index of each queued contact | `box_collect_contacts` (by address) | `box_resolve_contacts` (by address) |
| `g_box_resolve_partner_index` | `0x00655420` | `int[40]`: the partner index of the same entry | `box_collect_contacts` (by address) | `box_resolve_contacts` (by address) |
| `g_box_depenetration_offset` | `0x006554c0` | vec3, `depth × contact_normal`, gated `-0.4 <= depth` | `box_collect_contacts` | `box_ai_tick` (subtracts it from the body position) |
| `g_box_contact_dyn_object` | `0x006554d0` | `ptr[30]`: the hit `tDynamicCollObject` (record `+0x1c`), 0 for a static hit | `box_ai_tick` (by address) | `box_ai_tick` (by address) |
| `g_box_contact_plane_d` | `0x00655548` | `float[30]`: the contact plane **constant `d`** (record `+0x0c`) | `box_ai_tick` (by address) | `box_ai_tick` (by address) |
| `g_box_contact_normal` | `0x006555c0` | interleaved vec3 × 29: the contact plane's **normal** (record `+0x00`) | `box_ai_tick` | `box_ai_tick` (by address) |
| `g_box_scratch_count` | `0x0065571c` | the shared index of the three contact arrays | `box_ai_tick` | `box_ai_tick` |
| `g_box_gravity` | `0x00655720` | vec3 `(0, -9.81, 0)` (`0xc11cf5c3`) | `box_static_init_gravity` | `box_accumulate_forces` (× mass), `box_ai_tick` (× 0.1), `box_net_apply_remote_state` |

**The contact arrays.** `box_ai_tick` calls `collision_test_sphere_slide` and copies each returned
contact record into the three parallel arrays while `g_box_scratch_count < 0x1d`, so at most **29**
entries are written. The normal array has room for exactly 29 vec3s before the count; the other
two have room for 30. Ghidra shows `plane_d` and `dyn_object` as the raw addresses `0x00655544` /
`0x006554cc` because they are indexed with the post-increment count, i.e. the base is element
`[-1]`. The consumer fixes the meaning: the read loop computes

```
n.x*p.x + n.y*p.y + n.z*p.z + plane_d[i] < 0.01
```

over the box's own centre — the plane equation `dot(n, p) + d`. The contact POINT exists at record
`+0x10`, and `box_ai_tick` never copies it. Each entry enters the ring with `bound_radius + plane_d`
as its constant. The records are produced by `collision_slide_sphere`, a
PC-only solver whose only consumer is this class.

## 6. Replication — types `0x1b` and `0x13`

**The replication band `+0xc9c..+0xcd8`** is written by `box_game_init` (the seed),
`box_write_state_to_frame` (the mirror, only when `+0x20` says this machine is authoritative) and
`box_apply_net_event_msg` (the remote update); `box_net_apply_remote_state` reads it for dead
reckoning.

| Type | Descriptor | Sender | Payload | `box_apply_net_event_msg` |
|---|---|---|---|---|
| `0x1b` | `g_event_msg_vt_physics_state` (`.rdata 0x0063ba90`) | `box_post_state_net_event_msg`, and `physical_object_post_state_net_event_msg` (`structs/physical_object.md` §6b) | actor-state base + 16-byte quaternion at `+0x1c` + three velocity floats quantised through `net_encode_fixed16` + flag byte at `+0x38` | fills the replication band, sets bit `0x8`, bit `0x10` from the sleep byte |
| `0x13` | `g_event_msg_vt_actor_act_state` (`.rdata 0x0063b6dc`) | `actor_set_act_state`, host role only | `{actor guid, act state}` | sets bit `0x2` when the act state is 1 |

Type `0x1b` is the shared **rigid-body state update**, not `C_box`'s own: `C_box` and `C_PhThing`
install the same descriptor, and `event_msg_physics_state_ctor` (`0x005a1860`) takes the type as a
parameter, so its serialize / deserialize pair is named for the payload
(`event_msg_physics_state_serialize` / `event_msg_physics_state_deserialize`). Type `0x13` is the
generic act-state change every actor posts; `bottle_apply_net_event_msg` handles it too. Full
write-up: `structs/net_event_msg.md`.

**The network pair verifies itself with no cross-build side.** Same shape as `structs/bottle.md`
§5a, and the same grading argument: `box_ai_tick` and `box_apply_net_event_msg` **both**
re-broadcast through `box_post_state_net_event_msg` under the byte-identical host-role gate
`g_net_session+0x100 == 7`. Two independent code paths agreeing on a non-obvious guard is usable
evidence when there is no PS2 body to diff.

## 7. The translation unit

`[0x00456460, 0x0045f130)`, **55 function starts** after the CRT pair: it opens with
`crt_static_init_guard_00456460` / `crt_atexit_register_00456480`, and its `.CRT$XCU` entries are
`0x00456460`, `0x00456490`, `0x00458c30`, `0x00458c50` (§3a). The next unit's guard,
`crt_static_init_guard_0045f130`, closes it; that unit is `C_box_new`. The previous unit is
`C_bottle` (`structs/bottle.md` §8).

Besides the `box_*` functions and the free helpers of §3d, the unit holds three I3D helpers that
belong to `structs/i3d_frame.md`: `i3d_frame_set_position` (`0x0045f000`, 42 callsites in 8
functions), `i3d_frame_set_orientation` (`0x0045f040`, 9 callsites in 5 functions) and
`i3d_frame_get_scale` (`0x0045f090`, 2 callers: `box_net_apply_remote_state` and
`program_process_command`). Graded by offset first and callsite census second, each member of a
family separately; adjacency proves nothing in either direction.

## 8. PC vs PS2

### 8a. The class is absent from the PS2 build, proven by the port's own file manifest

The PS2 ELF carries its source-file list in the clear: **173 `__sinit_<file>.cpp` symbols**. The
relevant run reads

```
... C_blood.cpp   C_bottle.cpp   C_box_new.cpp   C_bridge.cpp   C_car.cpp ...
```

with **no `C_box.cpp`**. The file was neither given another name nor inlined — it was never compiled into the
port. Three weaker checks agree: no `5C_box` mangled tag in the symbol table, no `__vt__5C_box`
among the vtable data symbols, and no PS2 constructor writes 9 to its kind field `+0x50`. The PC
has **more** code than the PS2 symbol table accounts for — the same direction as `C_bottle`'s
network layer.

### 8b. Why the name `C_box` is the inference it is

Three things point at it and nothing points away:

1. **Link order.** Aligning the vtables in `.rdata` with the PS2 source-file order places this
   vtable strictly between `C_bottle`'s `0x0063b870` and `C_box_new`'s `0x0063baa0`, and the PS2
   manifest shows the sort is case-insensitive by file name. `C_box.cpp` is the only plausible
   name that sorts between `C_bottle.cpp` and `C_box_new.cpp` (a prefix sorts before its
   extension).
2. **The successor is literally called `C_box_new`** — a name that presupposes a `C_box`.
3. **The developers' own word.** `recorder_new_track` labels this kind's tracks `box%d`.

### 8c. What the PS2 replaced it with

`C_box_new` (`structs/box_new.md`) does the same job in **12 functions instead of 55**, because
all the dynamics moved into the shared base `C_PhThing` (`structs/physical_object.md`). The
refactor is visible in the numbers: this class carries three bespoke impulse solvers, its own SAT
and its own contact cache; `C_box_new` carries a six-field descriptor and hands it to `InitBox` /
`InitWheel`.

## 9. Related

- `structs/box_new.md` — the class that replaced it, and the one the PS2 kept.
- `structs/physical_object.md` — `C_PhThing`, the physics base `C_box` does **not** use.
- `structs/actor.md` / `structs/actor_track.md` — the base class and the slot 0–12 recorder
  interface `box_track_create` / `box_track_record_key` implement.
- `structs/bottle.md` — the neighbouring class, and the template for the network-pair argument.
- `structs/net_event_msg.md` — the two message types.

## 10. Open items

1. **Is `C_box` instantiated by any mission?** It needs an in-game observation or a mission-file
   scan; the `dyn*` frame-name convention of `box_build_dyn_collision_from_frame` is what to search
   the level data for.
2. **What `unread_100`, `collided_human_frame`, `unused_b6c`, `unused_c98`, `unread_cc4`,
   `unread_cdc` and `unused_ce0` were for is not recoverable.** Searched: every `.text` operand with
   each displacement, by whether its base register holds the function's receiver, and the functions
   able to hold a `C_box`, meaning those testing kind 9 or calling a `box_` method. That search
   cannot see a pointer used with no kind test, or a kind switch the decomp does not render as a
   `case`.
