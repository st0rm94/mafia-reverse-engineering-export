# `C_airplane` — the scripted aeroplane actor

**Created** 2026-08-04 13:27 +0200 · **Modified** 2026-11-09 18:20 +0100

A `C_actor` subclass: the aeroplane the player shoots down from a moving car in
**MISE16-LETISTE** (the scene name is the Czech word for airport). It flies a route over the nav web, takes damage into two
engine health pools shown on a HUD gauge, carries six dynamic collision proxies and two engine
particle effects, and is fully data-driven from its scene definition record. The same class also
flies the planes of MISE10-LETISTE and the free-ride EXTREME scene (§5).

| | |
|---|---|
| PC prefix | `airplane_` |
| PC kind (`+0x10`) | `0x16`, stamped by `airplane_ctor` |
| PC vtable | `0x0063b7c0`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x3f0` (`operator_new(0x3f0)` in `mission_create_actor` case `0x16`) |
| PC constructor | `airplane_ctor` @ `0x0044ef50` |
| PC translation unit | `[0x0044ef20, 0x00453a20)` (§6) |
| PS2 class | `C_airplane` (tag `10C_airplane`), 32 methods: 30 CONFIRMED twins and 2 base stubs, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC. The band `+0x70..+0xb0`
holds the 17 tunables of the scene definition record (§5).

```c
typedef struct C_airplane {                 // 0x3f0 bytes; PC offsets
    /* +0x000 */ C_actor     base;              // [H] the C_actor band (structs/actor.md §1)
    /* +0x070 */ float       acceleration;      // [H] 5.0; speed gained per second
    /* +0x074 */ float       takeoff_speed;     // [H] 25.0; the takeoff run accelerates to it
    /* +0x078 */ float       initial_speed;     // [H] 0.0; copied into speed by GameInit
    /* +0x07c */ float       blur_swap_speed;   // [H] 5.0; above it the blades swap to the disc
    /* +0x080 */ float       route_speed;       // [H] 9.0; the route-follow cruise speed
    /* +0x084 */ float       bank_factor;       // [H] 1.0; the roll angle is the turn angle times this
    /* +0x088 */ float       pitch_min;         // [H] 0.8; engine-sound pitch at standstill
    /* +0x08c */ float       pitch_max;         // [H] 1.5; engine-sound pitch at takeoff speed
    /* +0x090 */ int32_t     sound_id[3];       // [H] 0x82/0x83/0x84; one per E_AIRPLANE_SOUND
    /* +0x09c */ float       initial_prop_speed;// [H] 0.01; copied into prop_speed by GameInit
    /* +0x0a0 */ float       prop_speed_max;    // [H] 0.1; propeller angular-speed cap
    /* +0x0a4 */ float       prop_speed_rate;   // [H] 0.015; its ramp per ms
    /* +0x0a8 */ float       engine_health_max; // [H] 400.0; each engine's starting health
    /* +0x0ac */ int32_t     damaged_effect_id; // [H] 9; particle below half health
    /* +0x0b0 */ int32_t     destroyed_effect_id;//[H] 6; particle at zero health
    /* +0x0b4 */ int32_t     flight_state;      // [H] 0 idle, 1 route-follow, 2 takeoff run (§4)
    /* +0x0b8 */ float       speed;             // [H] the current speed along the route
    /* +0x0bc */ float       segment_t;         // [H] 0..1 along the current route segment
    /* +0x0c0 */ S_quat      attitude_offset;   // [H] a constant -0.24 rad tilt; composed into rot_to outside route-follow
    /* +0x0d0 */ S_quat      rot_cur;           // [H] the orientation written to the frame
    /* +0x0e0 */ S_quat      rot_spawn;         // [H] the frame orientation at GameInit; saved; no other reader (.text scan)
    /* +0x0f0 */ S_quat      rot_from;          // [H] slerp start of the current segment
    /* +0x100 */ S_quat      rot_to;            // [H] slerp end of the current segment
    /* +0x110 */ C_web_node *node_target;       // [H] the node being flown to
    /* +0x114 */ C_web_node *node_prev;         // [H] the node the segment starts at
    /* +0x118 */ vector      prop_blades;       // [H] the vrt1..vrt3 frames; shown by +0x169
    /* +0x128 */ vector      prop_discs;        // [H] the on_vrt1..on_vrt3 frames; shown by +0x168
    /* +0x138 */ float       prop_angle;        // [H] the shared blade angle, wraps at 2*pi
    /* +0x13c */ float       prop_speed;        // [H] ramps toward prop_speed_max
    /* +0x140 */ S_vector    prop_axis;         // [H] the rotation axis passed to each frame
    /* +0x14c */ I3D_sound  *sound[3];          // [H] the emitters built from sound_id[]
    /* +0x158 */ int32_t     sound_prev;        // [H] the fading-out slot; -1 = none
    /* +0x15c */ int32_t     sound_cur;         // [H] the fading-in slot; -1 = none
    /* +0x160 */ float       vol_prev;          // [H] fades out over 6000 ms
    /* +0x164 */ float       vol_cur;           // [H] fades in over 6000 ms, clamped to 1.0
    /* +0x168 */ uint8_t     prop_disc_visible; // [H] visibility of prop_discs
    /* +0x169 */ uint8_t     prop_blades_visible;//[H] visibility of prop_blades
    /* +0x16a */ uint8_t     damage_hud_shown;  // [H] the HUD damage gauge is shown
    /* +0x16c */ tDynamicCollObject *coll[6];   // [H] 2 engine spheres, then 4 Dummy boxes
    /* +0x184 */ S_matrix    world_inv;         // [H] inverse of the frame's world matrix
    /* +0x1c4 */ int32_t     route_start_id;    // [H] start node id; 0x29a = none
    /* +0x1c8 */ float       engine_health[2];  // [H] per engine; 0 = engine destroyed
    /* +0x1d0 */ int32_t     engine_particle[2];// [H] per engine particle handle; -1 = none
    /* +0x1d8 */ int32_t     engine_effect[2];  // [H] per engine: 0 none, 1 damaged, 2 destroyed
    /* +0x1e0 */ I3D_frame  *dummy_frame[4];    // [H] the Dummy01..Dummy04 frames
    /* +0x1f0 */ S_matrix    dummy_matrix[4];   // [H] each Dummy frame's unscaled world matrix
    /* +0x2f0 */ S_matrix    dummy_matrix_inv[4];//[H] their inverses
} C_airplane;
```

**Why these arrays, and what each entry is.**

| Field | Count | Entry |
|---|---|---|
| `sound_id[3]`, `sound[3]` | 3 | one per value of the developers' `E_AIRPLANE_SOUND` (the PS2 signature of `PlaySound` takes it): **0** the engine running (played by GameInit and every route-follow tick), **1** the takeoff run (played when a run starts; its pitch follows the speed), **2** airborne (played once speed reaches `takeoff_speed`). `airplane_create_sounds` builds `sound[i]` from `sound_id[i]`, and `airplane_play_sound` refuses an index of 3 or more |
| `coll[6]` | 6 | `airplane_fly_along_route` hands `coll` to `collision_dyn_set_next_multiple_owners` with a count of 6, so the six proxies are one array. Entries 0 and 1 are kind `0x42` spheres of radius 0.9 at the first two positions `airplane_get_engines_pos` returns; entries 2–5 are kind `0x40` boxes, one per `DummyNN` frame found, else 0 |
| `dummy_frame[4]`, `dummy_matrix[4]`, `dummy_matrix_inv[4]` | 4 | `airplane_game_init` looks up `Dummy01`..`Dummy04` (format `Dummy0%i`, `i` = 1..4) under the plane's frame; a frame of type 6 becomes entry `i-1` of all three arrays and the box proxy `coll[i+1]`, whose `+0x3c` / `+0x40` point at its two matrices. The matrix is the frame's world matrix with each axis divided by the frame scale; `airplane_update` refreshes both every tick |
| `engine_health[2]`, `engine_particle[2]`, `engine_effect[2]` | 2 | the two engines. `airplane_hit` drains the nearer engine; `airplane_update_engine_particles` places engine `i`'s effect at the `i`-th local offset of `airplane_get_engine_local_offsets` |
| `prop_blades`, `prop_discs` | at most 3 each | `airplane_game_init` looks up `vrt1`..`vrt3` and `on_vrt1`..`on_vrt3` and pushes each frame it finds; exactly one of the two sets is visible at a time |

## 2. Field evidence

Measured by a census of every operand with each displacement in the `airplane_*` functions, then
read in the bodies (the same displacement on a frame or a save buffer is excluded).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `airplane_ctor` (through `actor_ctor`; kind `+0x10` = `0x16`) | `folded_get_dword_0x10` (the kind); the methods of §3 (the position `+0x24`, the frame `+0x68`) | the inherited band |
| +0x070 | acceleration | H | `airplane_ctor`, `airplane_load_data`, `airplane_duplicate_from` | `airplane_ai_tick`, `airplane_save_data` | speed += `dt × acceleration × 0.001` in both moving states (§4) |
| +0x074 | takeoff_speed | H | the band writers ¹ | `airplane_ai_tick`, `airplane_save_data` | the takeoff run's target, and the denominator of the sound-1 pitch |
| +0x078 | initial_speed | H | the band writers | `airplane_game_init`, `airplane_save_data` | copied into `speed` |
| +0x07c | blur_swap_speed | H | `airplane_ctor`, `airplane_load_data` | `airplane_spin_propellers`, `airplane_save_data` | once `speed` exceeds it while the blades are shown, the blades hide, the disc shows and the angle resets. Not copied by `airplane_duplicate_from` ² |
| +0x080 | route_speed | H | the band writers | `airplane_ai_tick`, `airplane_save_data` | the route-follow target (§4) |
| +0x084 | bank_factor | H | the band writers | `airplane_pick_next_web_node`, `airplane_save_data` | `S_quat::Make(local, frame+0x70, turn_angle * bank_factor)` (`fmul [esi+0x84]` in the disassembly): the angle between the previous and the next route segment, scaled into a roll about the frame's third rotation row. Shipped values 1.0, 0.3 and -1.0 (§5); the one reader outside the save (.text scan) |
| +0x088 | pitch_min | H | the band writers | `airplane_ai_tick`, `airplane_save_data` | sound-1 pitch = `pitch_min + (pitch_max - pitch_min) × speed / takeoff_speed` |
| +0x08c | pitch_max | H | the band writers | `airplane_ai_tick`, `airplane_save_data` | the same formula |
| +0x090 | sound_id | H | the band writers (`0x82`, `0x83`, `0x84`) | `airplane_create_sounds`, `airplane_save_data` | the 12-byte block is one `dtaWrite`; each id goes to `sound_table_construct_sound` |
| +0x09c | initial_prop_speed | H | the band writers | `airplane_game_init`, `airplane_save_data` | copied into `prop_speed` |
| +0x0a0 | prop_speed_max | H | the band writers | `airplane_spin_propellers`, `airplane_save_data` | the ramp target |
| +0x0a4 | prop_speed_rate | H | the band writers | `airplane_spin_propellers`, `airplane_save_data` | the ramp step per ms |
| +0x0a8 | engine_health_max | H | the band writers (400.0) | `airplane_game_init`, `airplane_hud_damage_toggle`, `airplane_update_engine_particles`, `airplane_save_data` | seeds both `engine_health` entries; the gauge denominator and the half-health threshold |
| +0x0ac | damaged_effect_id | H | the band writers (9) | `airplane_update_engine_particles`, `airplane_save_data` | created on an engine whose health is below half |
| +0x0b0 | destroyed_effect_id | H | the band writers (6) | `airplane_update_engine_particles`, `airplane_save_data` | created on an engine whose health is 0 |
| +0x0b4 | flight_state | H | `airplane_game_init` (= 0), `airplane_start_ground_run` (1 or 2), `airplane_begin_takeoff` (= 2), `airplane_ai_tick` (= 0 at the route end), `airplane_pick_next_web_node` (= 0 with no next node), `airplane_save_game_load` | `airplane_ai_tick`, `airplane_pick_next_web_node`, `airplane_update`, `airplane_begin_takeoff`, `airplane_save_game_save` | §4 |
| +0x0b8 | speed | H | `airplane_game_init`, `airplane_start_ground_run`, `airplane_ai_tick`, `airplane_save_game_load` | `airplane_fly_along_route` (velocity = heading × speed), `airplane_spin_propellers`, `airplane_tick_sounds` (the emitters' velocity), `airplane_ai_tick`, `airplane_save_game_save` | 0 at the route end |
| +0x0bc | segment_t | H | `airplane_ctor` (= 0), `airplane_pick_next_web_node` (= 0), `airplane_fly_along_route`, `airplane_save_game_load` | `airplane_ai_tick` (the slerp parameter; ≥ 1.0 ends the segment), `airplane_fly_along_route`, `airplane_save_game_save` | position = lerp(`node_prev`, `node_target`, `segment_t`) |
| +0x0c0 | attitude_offset | H | `airplane_game_init` (`S_quat::Make(local, axis, -0.24)` at `0x0044f8f0`, the normalised `frame+0x70` row as the axis), `airplane_pick_next_web_node` (in state 2, `S_quat::Make(this+0xc0, frame+0x50, -0.24)`, `push 0xbe75c28f`) | `airplane_pick_next_web_node` | a constant tilt of 0.24 rad (13.75 degrees); composed by hand into `rot_to` (a quaternion product, then `quat_multiply`) when the state is not 1; in state 1 `rot_to` stays the plain aimed orientation; no other reader (.text scan) |
| +0x0d0 | rot_cur | H | `airplane_game_init` (the frame orientation), `airplane_ai_tick` (`S_quat::Slerp(rot_from, rot_to, segment_t)`), `airplane_start_ground_run` (`S_quat::SetDir` toward the next node), `airplane_save_game_load` | `airplane_update` (into the frame's rotation `frame+0x9c`), `airplane_pick_next_web_node`, `airplane_save_game_save` | - |
| +0x0e0 | rot_spawn | H | `airplane_game_init`, `airplane_save_game_load` | `airplane_save_game_save` | the frame orientation captured at GameInit and round-tripped by the savegame pair; no other reader (.text scan) |
| +0x0f0 | rot_from | H | `airplane_game_init`, `airplane_pick_next_web_node` (= `rot_cur`), `airplane_start_ground_run` | `airplane_ai_tick` | - |
| +0x100 | rot_to | H | `airplane_game_init` (= `rot_from`), `airplane_pick_next_web_node` (the frame orientation aimed along the next segment) | `airplane_ai_tick` | - |
| +0x110 | node_target | H | `airplane_ctor` (= 0), `airplane_start_ground_run` (the start node), `airplane_pick_next_web_node`, `airplane_save_game_load` | `airplane_ai_tick` (arrival: squared distance < 0.5), `airplane_fly_along_route`, `airplane_pick_next_web_node`, `airplane_save_game_save` | a `C_web_node` record of the mission's nav web, stride `0x1e`; saved as an index |
| +0x114 | node_prev | H | `airplane_ctor` (= 0), `airplane_game_init` (= 0), `airplane_start_ground_run` (= 0), `airplane_pick_next_web_node` (= the old target), `airplane_save_game_load` | `airplane_ai_tick`, `airplane_fly_along_route`, `airplane_pick_next_web_node`, `airplane_save_game_save` | saved as an index |
| +0x118 | prop_blades | H | `airplane_ctor` (empty), `airplane_game_init` (`std_vector4_insert_n`) | `airplane_apply_propeller_visibility`, `airplane_spin_propellers`, `airplane_dtor` (frees it second) | allocator byte `+0x118`, begin / end / capacity `+0x11c` / `+0x120` / `+0x124` |
| +0x128 | prop_discs | H | `airplane_ctor` (empty), `airplane_game_init` | `airplane_apply_propeller_visibility`, `airplane_spin_propellers`, `airplane_dtor` (frees it first) | begin / end / capacity `+0x12c` / `+0x130` / `+0x134` |
| +0x138 | prop_angle | H | `airplane_ctor`, `airplane_game_init` (= 0), `airplane_spin_propellers`, `airplane_save_game_load` | `airplane_spin_propellers`, `airplane_save_game_save` | wraps at 2π |
| +0x13c | prop_speed | H | `airplane_game_init`, `airplane_spin_propellers`, `airplane_save_game_load` | `airplane_spin_propellers`, `airplane_save_game_save` | - |
| +0x140 | prop_axis | H | `airplane_game_init` (the axis of the last propeller frame found) | `airplane_spin_propellers` | - |
| +0x14c | sound | H | `airplane_ctor` (= 0), `airplane_create_sounds`, `airplane_destroy_sounds` | `airplane_play_sound`, `airplane_tick_sounds`, `airplane_ai_tick` (`sound[1]`, the pitch) | - |
| +0x158 | sound_prev | H | `airplane_ctor` (= -1), `airplane_create_sounds`, `airplane_play_sound`, `airplane_tick_sounds` (= -1 once faded) | `airplane_tick_sounds` | - |
| +0x15c | sound_cur | H | `airplane_ctor` (= -1), `airplane_create_sounds`, `airplane_play_sound` | `airplane_play_sound` (the change guard), `airplane_tick_sounds`, `airplane_save_game_save` | `airplane_save_game_load` replays it through `airplane_play_sound` |
| +0x160 | vol_prev | H | `airplane_ctor`, `airplane_create_sounds`, `airplane_play_sound` (= `vol_cur`), `airplane_tick_sounds` | `airplane_tick_sounds` | the fade is 6000 ms |
| +0x164 | vol_cur | H | `airplane_ctor`, `airplane_create_sounds`, `airplane_play_sound` (= 0), `airplane_tick_sounds` | `airplane_play_sound`, `airplane_tick_sounds` | clamped to 1.0 |
| +0x168 | prop_disc_visible | H | `airplane_ctor`, `airplane_game_init` (= 0), `airplane_spin_propellers`, `airplane_start_ground_run` (= 1), `airplane_save_game_load` | `airplane_apply_propeller_visibility`, `airplane_spin_propellers`, `airplane_save_game_save` | - |
| +0x169 | prop_blades_visible | H | `airplane_ctor`, `airplane_game_init` (= 1), `airplane_spin_propellers`, `airplane_start_ground_run` (= 0), `airplane_save_game_load` | `airplane_apply_propeller_visibility`, `airplane_spin_propellers`, `airplane_save_game_save` | exactly one of the two flags is set |
| +0x16a | damage_hud_shown | H | `airplane_ctor` (= 0), `airplane_hud_damage_toggle` | `airplane_ai_tick` (passes it back to `airplane_hud_damage_toggle` every tick) | - |
| +0x16c | coll | H | `airplane_game_init` (allocates and `collision_dyn_insert`s each), `airplane_game_done` and `airplane_on_act_state_changed` (`collision_dyn_erase`, freed, = 0) | `airplane_update` (`collision_dyn_update` on each), `airplane_fly_along_route` (`collision_dyn_set_next_multiple_owners(coll, 6)`) | each proxy's `+0x44` is the plane |
| +0x184 | world_inv | H | `airplane_game_init`, `airplane_update` (`S_matrix::Inverse` of the frame's world matrix) | `airplane_update` | - |
| +0x1c4 | route_start_id | H | `airplane_ctor` (= `0x29a`), `airplane_start_ground_run` | `airplane_ai_tick` | the start node is found with `web_path_find_node_id`; the route ends at the node whose id is `route_start_id + 1`. `0x29a` is the project-wide "none" value |
| +0x1c8 | engine_health | H | `airplane_ctor` and `airplane_game_init` (= `engine_health_max`), `airplane_hit` (drains, clamps at 0), `airplane_save_game_load` | `airplane_hit`, `airplane_is_destroyed` (both 0 = destroyed), `airplane_hud_damage_toggle` (fill = sum / 2 × max), `airplane_update_engine_particles`, `airplane_save_game_save` | - |
| +0x1d0 | engine_particle | H | `airplane_ctor` and `airplane_game_init` (= -1), `airplane_update_engine_particles` | `airplane_game_done`, `airplane_on_act_state_changed` (`particle_manager_destroy_particle`), `airplane_update_engine_particles` | - |
| +0x1d8 | engine_effect | H | `airplane_game_init` (= 0), `airplane_update_engine_particles` | `airplane_update_engine_particles` | the swap guard: an effect is replaced only when the value changes |
| +0x1e0 | dummy_frame | H | `airplane_game_init` | `airplane_update` | - |
| +0x1f0 | dummy_matrix | H | `airplane_game_init`, `airplane_update` | the box proxies (`coll[i]+0x3c`) | - |
| +0x2f0 | dummy_matrix_inv | H | `airplane_game_init`, `airplane_update` (`S_matrix::Inverse`) | the box proxies (`coll[i]+0x40`) | ends at `0x3f0`, the instance size |

¹ The band writers are `airplane_ctor` (the defaults), `airplane_load_data` and
`airplane_duplicate_from`; `airplane_save_data` writes the band back in the same order.
² `airplane_duplicate_from` copies 16 of the 17 band values; `blur_swap_speed` is the one it skips.

## 3. Method family

Coverage: all **32** PS2 `10C_airplane` methods are in the tables below — **30 CONFIRMED twins**
(one of them an ICF fold) and **2 base stubs**. The scalar-deleting destructor has no PS2 twin;
the CRT pair belongs to the translation unit; `airplane_get_engine_local_offsets` is PC-only.
Slot 13 is `airplane_ai_tick` and slot 14 `airplane_update`, the `C_actor`-family `AI` / `Update`
slots.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0044ef20 | `crt_static_init_guard_0044ef20` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x0044ef40 | `crt_atexit_register_0044ef40` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0044ef50 | `airplane_ctor` | instance | `__ct__10C_airplaneFv` | 0x001192c0 | CONFIRMED | `actor_ctor`, the two empty vectors, kind `0x16`, the 17 defaults; 368 bytes on both builds, every constant in the same order |
| 0x0044f0c0 | `airplane_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor, vtable slot 16 ¹ |
| 0x0044f0e0 | `airplane_dtor` | instance | `__dt__10C_airplaneFv` | 0x00119440 | CONFIRMED | frees `prop_discs`, then `prop_blades`, then `actor_dtor` |

¹ No standalone PS2 twin: GCC carries the free flag inside `__dt__10C_airplaneFv`.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0044f140 | `airplane_save_data` | `SaveData__10C_airplaneFP7C_chunk` | 0x00119510 | CONFIRMED | slot 19; a guarded `dtaWrite` chain over the tunable band |
| 0x0044f380 | `airplane_load_data` | `LoadData__10C_airplaneFP7C_chunk` | 0x001197c0 | CONFIRMED | slot 20; a version byte, then the 17 values in save order through `chunk_read` (§7) |
| 0x0044f600 | `airplane_duplicate_from` | `DuplicateFrom__10C_airplaneFP7C_actor` | 0x00119f30 | CONFIRMED | slot 21; copies 16 band values, the same 16 on both builds |
| 0x0044f6c0 | `airplane_game_init` | `GameInit__10C_airplaneFv` | 0x00119fe0 | CONFIRMED | slot 25; orientation, the six proxies, the propellers, the sounds; `airplane_play_sound(0)` |
| 0x004503c0 | `airplane_game_done` | `GameDone__10C_airplaneFv` | 0x0011ac30 | CONFIRMED | slot 26; `actor_game_done` first, then hides the gauge, erases and frees the six proxies, destroys the sounds and the two particles |
| 0x004504a0 | `airplane_pick_next_web_node` | `NextNode__10C_airplaneFv` | 0x0011ad50 | CONFIRMED | advances `node_prev` / `node_target` along the node's connection array and builds `rot_to` (§4) |
| 0x00450990 | `airplane_fly_along_route` | `AI_fly__10C_airplaneFUi` | 0x0011b4c0 | CONFIRMED | lerps the position between the two nodes by `segment_t`, sweeps a sphere test for obstacles |
| 0x00451540 | `airplane_spin_propellers` | `Vrtule__10C_airplaneFUi` | 0x0011c240 | CONFIRMED | ramps `prop_speed`, advances the angle with a 2π wrap, swaps blades for the disc (*vrtule* = propeller) |
| 0x004516e0 | `airplane_ai_tick` | `AI__10C_airplaneFUi` | 0x0011c470 | CONFIRMED | slot 13; the flight-state tick (§4) |
| 0x00451990 | `airplane_update` | `Update__10C_airplaneFUi` | 0x0011c8b0 | CONFIRMED | slot 14; writes `rot_cur` into the frame with the shared masks `&0xfffffedf\|0x40000000` and `&0xfffffecb\|0x40000008`, refreshes the matrices and the six proxies |
| 0x004520e0 | `airplane_start_ground_run` | `Start__10C_airplaneFUif` | 0x0011d1b0 | CONFIRMED | engine side of script `airplane_start`; `SetActState(0)`, `web_path_find_node_id`, the connection walk, statement-aligned (§4) |
| 0x004522c0 | `airplane_begin_takeoff` | `Start2__10C_airplaneFv` | 0x0011d3d0 | CONFIRMED | engine side of script `airplane_start2`; `if (state != 2) { state = 2; PlaySound(1); }` on both builds |
| 0x004522e0 | `airplane_apply_propeller_visibility` | `InitVrtule__10C_airplaneFv` | 0x0011d400 | CONFIRMED | applies the two visibility flags to the two frame vectors |
| 0x00452350 | `airplane_hit` | `Hit__10C_airplaneF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0011d4b0 | CONFIRMED | slot 31; picks the nearer engine by squared distance and drains its health |
| 0x004525d0 | `airplane_update_engine_particles` | `UpdateParticles__10C_airplaneFv` | 0x0011d820 | CONFIRMED | per engine: the destroyed effect at 0, the damaged effect below half, the `engine_effect` guard |
| 0x00452810 | `airplane_get_engines_pos` | `GetEnginesPos__10C_airplaneFR8S_vectorR8S_vectorR8S_vector` | 0x0011da90 | CONFIRMED | three world positions through the frame matrix and the triple ±1.0 normalisation cascade |
| 0x00453090 | `airplane_create_sounds` | `InitSounds__10C_airplaneFv` | 0x0011e060 | CONFIRMED | the same create → construct → mute → loop → attach → register loop over the three slots |
| 0x00453150 | `airplane_destroy_sounds` | `ReleaseSounds__10C_airplaneFv` | 0x0011e190 | CONFIRMED | unregisters and destroys the three emitters, statement for statement |
| 0x00453190 | `airplane_is_destroyed` | `IsDestroyed__10C_airplaneFv` | 0x0011e230 | CONFIRMED | engine side of script `airplane_isdestroyed`; 1 when both engines' health is 0 |
| 0x004531c0 | `airplane_save_game_get_size` | `SaveGameGetSize__10C_airplaneFv` | 0x0011e280 | CONFIRMED | slot 22; the base size plus `0x7c`, identical |
| 0x004531d0 | `airplane_save_game_save` | `SaveGameSave__10C_airplaneFPPv` | 0x0011e2a0 | CONFIRMED | slot 23; version byte 2; the nodes stored as indices |
| 0x00453300 | `airplane_save_game_load` | `SaveGameLoad__10C_airplaneFPPv` | 0x0011e3f0 | CONFIRMED | slot 24; the four-call tail `PlaySound`, `InitVrtule`, a vtable call, `UpdateParticles` matches exactly |
| 0x00453450 | `airplane_play_sound` | `PlaySound__10C_airplaneFQ210C_airplane16E_AIRPLANE_SOUND` | 0x0011e590 | CONFIRMED | starts a crossfade from `sound_cur` to the given slot (§1) |
| 0x004534c0 | `airplane_tick_sounds` | `TickSounds__10C_airplaneFUi` | 0x0011e630 | CONFIRMED | the 6000 ms crossfade and the emitter positions |
| 0x004537c0 | `airplane_on_act_state_changed` | `ChangeState__10C_airplaneFv` | 0x0011e8d0 | CONFIRMED | slot 27; when the act state `+0x1c` is 2, the `airplane_game_done` teardown without its base call |
| 0x004538b0 | `airplane_hud_damage_toggle` | `ShowDamageIndicator__10C_airplaneFb` | 0x0011e9f0 | CONFIRMED | engine side of script `airplaneshowdamage`; shows or hides the HUD gauge and sets its fill |
| 0x00453a10 | `airplane_get_material_id` | `GetMaterialID__10C_airplaneFP9I3D_frame` | 0x0011ea90 | CONFIRMED | slot 32; `return 0x13;` (material 19) on both builds |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__10C_airplaneFUi` | 0x0011c8a0 | base-stub | slot 15; the shared one-argument no-op (`c2 04 00`, in 30 vtables); the PS2 body is a bare `return` |
| 0x004ae890 | `actor_subclass_init_require_frame` | `Init__10C_airplaneFP9I3D_frame` | 0x001194e0 | CONFIRMED | slot 18; `if (!frame) return false; return C_actor::Init(frame);` on both builds — an ICF fold shared by 6 vtables, listed in each owner's doc |
| 0x00574a40 | `stub_noop_cdecl_00574a40` | `Death__10C_airplaneFv` | 0x0011ad40 | base-stub | slot 29, the `Death` slot (PS2 slot 18, between `GetActorSpeed` and `Collision`); the PS2 body is a bare `return` |

The other slots are `C_actor`'s: `actor_release` (17), `actor_get_actor_speed` (28),
`stub_retfalse_args24_00482690` (30, the inherited `Collision`), `actor_get_bound_points_2d` (34)
and the base stubs of `structs/actor.md` §3c.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00452000 | `quat_multiply` | - | - | - | an `S_quat` product emitted out of line here; 3 callsites: `airplane_pick_next_web_node`, `door_ai`, `door_save_game_load` |
| 0x00452e70 | `airplane_get_engine_local_offsets` | - | - | - | PC-only; three constant frame-local offsets built from `-1.0` / `2.9` / `4.0`, no `this` ¹ |

¹ The prologue is `sub esp,0x48`; every `lea ecx,[esp+N]` is a stack `S_vector` for
`vec3_set_xyz`, and ECX is written before it is ever read, so the function is not `__thiscall` —
the `mov ecx, esi` at its one callsite in `airplane_update_engine_particles` is the caller's own
state. `airplane_update_engine_particles` passes the offsets to `particle_manager_create_particle`
together with the frame, so they are frame-local (§7).

## 4. The flight states, the route and the orientation

`airplane_ai_tick` returns at once in state 0; otherwise it ends the segment when `segment_t`
reaches 1.0 or the plane is within a squared distance of 0.5 of `node_target`, calls
`airplane_pick_next_web_node`, flies the segment (`airplane_fly_along_route`), slerps the
orientation, and then runs the state's speed rule. Every tick it also spins the propellers, runs
the sound crossfade and re-applies the damage gauge.

| `flight_state` | Entered by | Per tick |
|---|---|---|
| 0 idle | `airplane_game_init`; the route end; `airplane_pick_next_web_node` when the node has no next connection (it also sets the act state and calls the frame's vtable `+0x24`) | nothing moves |
| 1 route-follow | `airplane_start_ground_run` with a start speed ≤ 0 (it plays sound 0) | while `node_target`'s id is `route_start_id + 1`, `speed = route_speed × d² / L²` (distance to that node over the segment length, both squared); otherwise speed ramps toward `route_speed`. Plays sound 0. When the newly picked node is the end node, state 0 and speed 0 |
| 2 takeoff run | `airplane_start_ground_run` with a start speed > 0 (it places the plane on the start node, aims it at the next one, shows the disc and plays sound 1); `airplane_begin_takeoff` | speed ramps toward `takeoff_speed`; sound 1's pitch follows the speed (§2); once `takeoff_speed` is reached, sound 2 every tick |

**The orientation** is a slerp per segment. `airplane_pick_next_web_node` stores the current
orientation into `rot_from`, temporarily aims the frame along the new segment and reads that
orientation back into `rot_to`, builds a roll about the frame's `+0x70` row from the turn angle × `bank_factor`, and outside
state 1 composes `attitude_offset`, a constant tilt of -0.24 rad, into `rot_to`. Each tick `airplane_ai_tick` sets
`rot_cur = Slerp(rot_from, rot_to, segment_t)` and `airplane_update` writes it into the frame.

## 5. Where instances come from

Only from scene placement. `mission_open_bin` walks each actor-definition record (`0xAE21`) of a
scene's `scene2.bin`: `0xAE23` names the frame, `0xAE22` gives the kind, and `mission_create_actor`
case `0x16` allocates `0x3f0` bytes and runs `airplane_ctor`; the actor's vtable `Init` (slot 18)
binds the frame, `actor_mark_scene_placed` and `mission_add_actor` register it, and `0xAE24` goes to
its `LoadData` slot, `airplane_load_data` (a version byte, then the 17 band values in save order).
`ACTOR_DUPLICATE` (`program_process_command` case `0xda`) could copy one, but in the extracted
script corpus only MISE07B uses that command, and not on an airplane. A census of all 79 shipped
`scene2.bin` files (13 631 definitions) finds four airplanes:

| Scene | Frame | +0x70 | +0x74 | +0x78 | +0x7c | +0x80 | +0x84 | +0x88 / +0x8c | Sound ids | +0x9c / +0xa0 / +0xa4 | +0xa8 | +0xac / +0xb0 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EXTREME | `Letadlo` | 5.0 | 25.0 | 13.89 | 5.0 | 9.0 | 1.0 | 0.8 / 1.5 | 0x82 0x83 0x84 | 0.01 / 0.1 / 0.015 | 400.0 | 9 / 6 |
| EXTREME | `UfoTalir` | 17.0 | 27.78 | 0.0 | 5.0 | 27.78 | 0.3 | 0.8 / 1.5 | 0x148 x3 | 0.01 / 0.1 / 0.015 | 400.0 | 9 / 6 |
| MISE10-LETISTE | `letadylko` | 10.0 | 33.33 | 25.0 | 5.0 | 20.0 | -1.0 | 1.0 / 10.0 | 0x84 x3 | 0.0 / 0.0 / 0.015 | 1.0 | 2 / 1 |
| MISE16-LETISTE | `xletadlo17413` | 2.0 | 22.22 | 0.0 | 4.17 | 9.72 | 1.0 | 0.8 / 1.8 | 0x82 0x83 0x84 | 0.01 / 0.1 / 0.015 | 250.0 | 9 / 6 |

The ctor's defaults equal `Letadlo`'s record except `+0x78` (ctor 0.0). Several speeds are round
km/h figures divided by 3.6 (22.22 = 80, 9.72 = 35, 27.78 = 100, 13.89 = 50, 4.17 = 15). MISE16's
scripts drive its plane with `airplane_start 0, 5, 0` (`airplane_start_ground_run`),
`airplane_start2` (`airplane_begin_takeoff`) and `airplaneshowdamage`
(`airplane_hud_damage_toggle`).

## 6. The translation unit

`[0x0044ef20, 0x00453a20)`: it opens with its CRT static-init pair
(`crt_static_init_guard_0044ef20` / `crt_atexit_register_0044ef40`) and holds 33 more function
starts — the 31 `airplane_*` functions, `quat_multiply` (§3d) and
`i3d_frame_build_scaled_rot_matrix` (`0x00453900`), a shared I3D helper with 35 callsites in 12
functions that belongs to `structs/i3d_frame.md`. The next unit's guard,
`crt_static_init_guard_00453a20`, closes it; that unit is `C_bottle`. The previous unit is
`C_actor_shot` (`structs/actor_shot.md` §6).

The region is forced 1:1 against the PS2 class: 31 `airplane_*` functions = 32 PS2 methods − 3
with no function of their own (`Init`, `NetDirect`, `Death`, all folded, §3c) + the scalar-deleting
destructor + `airplane_get_engine_local_offsets`.

## 7. PC vs PS2

**The class is a straight port.** The tunable band holds the same 17 values on both builds,
`SaveData` / `LoadData` walk it in the same order, and PS2 `DuplicateFrom` skips the `+0x7c`
counterpart too. The field rebase is **not a constant delta** (PS2 `+0x33c` ↔ PC `+0x70`, but PS2
`+0x3f0` ↔ PC `+0x1c4`) because the `C_actor` base is larger on the PS2 *and* the two vector
members have different sizes — map by rank, never by subtracting. PS2 `Start2` keeps the flight
state at `+0x320` (`800`) and PS2 `IsDestroyed` reads the engine health at `+0x37c` / `+0x380`.

**`airplane_load_data`'s size ratio is 0.33 and that is not a rank error.** GCC open-codes every
guarded `dtaRead`; MSVC factors them into `chunk_read(field, size)`. The field *sequence* still
matches one for one.

**The three PS2 methods with no PC function of their own** are all folded (§3c):

| PS2 method | Size | PC realisation |
|---|---|---|
| `Init__10C_airplaneFP9I3D_frame` | 44 | the bare forwarder `if (!frame) return false; return C_actor::Init(frame);`; slot 18 holds the shared `actor_subclass_init_require_frame`, statement-identical to it and present in 6 vtables |
| `NetDirect__10C_airplaneFUi` | 8 | slot 15 holds `stub_noop_args4_00461180`, whose whole body is `c2 04 00` (`ret 4` = exactly one 4-byte argument), in 30 vtables |
| `Death__10C_airplaneFv` | 8 | slot 29 holds `stub_noop_cdecl_00574a40`; the PS2 vtable puts `Death` between `GetActorSpeed` and `Collision`, which is PC slot 29 |

`GetMaterialID` survives as its own PC function only because it returns a distinct constant
(`0x13`) rather than nothing. `GetEnginesPos` has no PC counterpart at the one place PS2
`UpdateParticles` calls it: the PC passes the frame-local offsets of
`airplane_get_engine_local_offsets` with the frame instead (§3d).

**Corollary twins this class produced:**

| PC | PS2 | How |
|---|---|---|
| `airplane_hud_damage_toggle` | `ShowDamageIndicator__10C_airplaneFb` | the second statement of the GameDone pair on both builds |
| `collision_dyn_insert` | `DynInsert__11g_collisionFP18tDynamicCollObject` | ×3 ↔ ×3 in the GameInit pair |
| `particle_manager_destroy_particle` | `DestroyParticle__17C_ParticleManagerFi` ¹ | the particle teardown |
| `particle_manager_find_by_guid` | `FindGuid__17C_ParticleManagerFiRi` | the particle teardown |
| `chunk_read` | `Read__7C_chunkFPvi` ² | ×11 in `airplane_load_data` |
| `airplane_play_sound` | `PlaySound__10C_airplaneFQ210C_airplane16E_AIRPLANE_SOUND` | witnessed twice: the `Start2` pair and the `SaveGameLoad` tail |
| `airplane_apply_propeller_visibility` | `InitVrtule__10C_airplaneFv` | its own diff, then re-confirmed by the `SaveGameLoad` tail |
| `airplane_update_engine_particles` | `UpdateParticles__10C_airplaneFv` | the same tail |

¹ The record stride differs (PC 8 vs PS2 0x10), but that is record content: the key field `+0x04`
and the sorted-by-guid access are identical (`structs/recorder.md`).
² A bounds-checked `dtaRead` of one chunk field; a shared helper with 212 callsites across 20
functions (`mission_open_bin` ×46, `airplane_load_data` ×11, `human_load_data`). It leads
`C_bottle`'s unit (`structs/bottle.md` §8).

**Two cross-build constants worth reusing elsewhere.**

- **The `C_web_node` record size shows up in the savegame math.** `airplane_save_game_save` stores
  node pointers as `(node − web_base) / 0x1e` on the PC and `>> 5` (0x20) on the PS2 — an
  independent re-proof of the record-size difference documented in `structs/web_node.md`.
- **The 6000 ms sound crossfade is identical on both builds** — the PS2 divides the frame delta
  by `6000.0`, MSVC multiplies by `0.00016666666`. Same tuning, different codegen.

## 8. Related

- `structs/actor.md` — the base class; `structs/actor_track.md` — vtable slots 0–12.
- `structs/hud.md` — the `0x40000` damage-gauge bit; `basic/artifacts/hud_elements.md`.
- `structs/web_node.md` — the nav-web records the route walks.
- `structs/bottle.md` — the next class in the link order.

## 9. Open items

1. **In game:** which visible motion the roll (`bank_factor`) and the constant tilt (`attitude_offset`)
   produce has not been observed; the shipped `bank_factor` of -1.0 on MISE10's plane would mirror
   the roll.
