# `C_railway` — the ambient rail-traffic system (entity kind 8)

**Created** 2026-08-16 22:24 +0200 · **Modified** 2026-11-09 18:18 +0100

`C_railway` is the trams **and** the elevated train; the split against `C_wagon` is documented in `entity_kind.md` §3. The track
switches are a separate class, `C_turnout` (entity kind `0x18`, PC prefix `railway_switch_`, `structs/railway_switch.md`): its AI
calls `wagon_rebuild_all_forward_paths`, and the two shipped turnouts are in `MISE15-PRISTAV`, a scene with no `C_railway`, so they
route the mission wagons, not the trams. `railway_switch_` extends this class's prefix, so a prefix match on `railway_` also catches
the turnout's functions.

| | |
|---|---|
| PC prefix | `railway_`, `rail_seats_`, `rail_passenger_`, `rail_doors_`, `rail_glass_`, `rail_window_`, `rail_entries_` |
| PC kind (`+0x10`) | `8` |
| PC vtable | `g_railway_vt` @ `0x0063cbd0` |
| PC base class | `C_actor` |
| PC instance size | `0x36c` (entity factory `case 8`) |
| PC constructor | `railway_ctor` |
| PC translation unit | `[0x004d35f0, 0x004ea1a0)`, 127 function starts, all named |
| PS2 class | `C_railway` (tag `9C_railway`), 76 methods; `S_RailSeats` 11; `S_rail_human` 10; `rail_doors` 7; `rail_glass` 4; `rail_window` 3; `rail_entries` 2 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The record holds every offset the class's code accesses; `unused_114` is a band with no access at all.

The vectors are MSVC `std::vector`: an allocator byte, then begin, end and capacity. `+0x1a8` is the designer's trailing-vagon count, so `vagon_positions` and `vagon_frames` hold `vagon_count + 1` entries, the engine being index 0. `spring_phases`, `position_deltas`, `direction_deltas` and `interior_coll_lists` hold `vagon_count + 1` entries too; `vagon_directions` holds one per vagon.

`back_nodes` begins at the target node ahead of the front bogie and runs back along the route behind it; `forward_nodes` holds the route beyond the target. The back builder follows links whose flag byte has bit `0x80`, the forward builder links without it. When the front bogie passes the target, `railway_place_on_track` moves `forward_nodes_begin[0]` to the head of `back_nodes`; `railway_must_stop` tests `back_nodes_begin[0]` and then walks `forward_nodes`.

### 1a. `C_railway`

```c
typedef struct C_railway {            // 0x36c bytes; C_actor base, 0x70 bytes
    /* +0x70 */ S_vector direction; // [H] front bogie to the target node; the node-passed test (§2a)
    /* +0x7c */ uint8_t back_nodes_alloc; // [H] allocator byte of the back node vector
    /* +0x80 */ C_web_node ** back_nodes_begin; // [H] the target node ahead, then the route behind, begin
    /* +0x84 */ C_web_node ** back_nodes_end; // [H] the back route, end
    /* +0x88 */ C_web_node ** back_nodes_cap; // [H] the back route, end of storage
    /* +0x8c */ float back_path_radius; // [H] length the back path must cover, (vagon_count + 1) * vagon_length + 30
    /* +0x90 */ S_vector front_position; // [H] front bogie position
    /* +0x9c */ S_vector rear_position; // [H] rear bogie position
    /* +0xa8 */ uint8_t forward_nodes_alloc; // [H] allocator byte of the forward node vector
    /* +0xac */ C_web_node ** forward_nodes_begin; // [H] the route beyond the target node, begin
    /* +0xb0 */ C_web_node ** forward_nodes_end; // [H] the forward route, end
    /* +0xb4 */ C_web_node ** forward_nodes_cap; // [H] the forward route, end of storage
    /* +0xb8 */ float * spring_phases; // [H] per-carriage sway phase, vagon_count + 1 entries
    /* +0xbc */ S_vector * vagon_positions; // [H] one position per vagon, stride 0xc
    /* +0xc0 */ S_vector * vagon_directions; // [H] one direction per vagon
    /* +0xc4 */ I3D_frame ** vagon_frames; // [H] index 0 is the engine's own frame
    /* +0xc8 */ S_vector * position_deltas; // [H] each body's position change in the last place; no consumer (.text scan)
    /* +0xcc */ S_vector * direction_deltas; // [H] each body's direction change in the last place; no consumer (.text scan)
    /* +0xd0 */ float target_speed; // [H] seeded from speed_scale by railway_game_init, recomputed by railway_ai
    /* +0xd4 */ float speed; // [H] current speed
    /* +0xd8 */ uint32_t stop_timer_ms; // [H] seeded 10000; railway_ai adds the frame time and tests it against 2000
    /* +0xdc */ uint32_t run_state; // [H] written by railway_set_state, railway_ai and the ctor
    /* +0xe0 */ uint32_t held_state; // [H] the state saved while holding behind another vehicle
    /* +0xe4 */ uint32_t ai_flags; // [H] railway_ai tests bit 2; saved with the game
    /* +0xe8 */ void * vagon_collisions; // [H] per-vagon dynamic-collision records, stride 0x48
    /* +0xec */ uint8_t coll_records_alloc; // [H] allocator byte of the RAIL_COLL record vector
    /* +0xf0 */ tDynamicCollObject ** coll_records_begin; // [H] the RAIL_COLL collision records (0x48 bytes, type 0x40), begin
    /* +0xf4 */ tDynamicCollObject ** coll_records_end; // [H] one past the last record
    /* +0xf8 */ tDynamicCollObject ** coll_records_cap; // [H] end of the vector's storage
    /* +0xfc */ void * interior_coll_lists; // [H] per-carriage collision-list vectors, 16 bytes each, vagon_count + 1
    /* +0x100 */ uint8_t added_persons_alloc; // [H] allocator byte of the vector below
    /* +0x104 */ tDynamicCollObject ** added_persons_begin; // [H] boarding persons' collision records (0x48 bytes, type 0x42)
    /* +0x108 */ tDynamicCollObject ** added_persons_end; // [H] one past the last element
    /* +0x10c */ tDynamicCollObject ** added_persons_cap; // [H] end of the vector's storage
    /* +0x110 */ S_matrix * vagon_matrices; // [H] allocated by railway_game_init, read as matrices by railway_update
    /* +0x114 */ uint8_t unused_114[0x44]; // [U] no access; no load or store (.text scan)
    /* +0x158 */ uint32_t cycle_accum_ms; // [H] ctor 0; railway_ai accumulates
    /* +0x15c */ uint32_t cycle_period_ms; // [H] ctor 200
    /* +0x160 */ C_railway * vehicle_ahead; // [H] the rail vehicle ahead, found by railway_find_vehicle_ahead
    /* +0x164 */ I3D_frame * dummy_col_frame; // [H] the model's DUMMY_COL frame, resolved by railway_game_init; read by railway_update
    /* +0x168 */ S_matrix dummy_col_matrix; // [H] the DUMMY_COL frame's world matrix, scale divided out; the collision record reads it
    /* +0x1a8 */ uint32_t vagon_count; // [H] trailing vagons; designer value, ctor 1
    /* +0x1ac */ float bogie_spacing; // [H] designer value, ctor 8.0; railway_place_on_track offsets the bogies by (vagon_length - this) / 2
    /* +0x1b0 */ float vagon_length; // [H] designer value, ctor 17.0; the train is (vagon_count + 1) * this long
    /* +0x1b4 */ float speed_scale; // [H] designer speed scale, ctor 12.5; seeds target_speed
    /* +0x1b8 */ float acceleration; // [H] designer value, ctor 3.0; speed changes by dt * 0.001 * this
    /* +0x1bc */ float mass; // [H] designer value, ctor 10000.0; the collision impulse uses 1 / this
    /* +0x1c0 */ float unread_1c0; // [U] ctor 0.0001; no reader (.text scan)
    /* +0x1c4 */ float braking_distance; // [H] speed^2 / (2 * acceleration), each frame
    /* +0x1c8 */ float inverse_inertia; // [H] 1 / (avg_extent^2 * mass * 0.25); scales the contact levers
    /* +0x1cc */ S_vector contact_velocity; // [H] dir * speed at the last contact; saved with the game
    /* +0x1d8 */ uint8_t is_elevated; // [H] 1 when the model file is metro.i3d; selects the sound ids (§5b)
    /* +0x1d9 */ uint8_t collision_latch; // [H] set when a collider touches the tram; the traffic clamp reports one block and clears it
    /* +0x1dc */ uint32_t * semaphore_phase; // [H] the signal phase that governs the current node; railway_ai state 10 tests it
    /* +0x1e0 */ rail_entries entries; // [H] the source boarding points (§1h)
    /* +0x1e8 */ rail_doors doors; // [H] the two door banks (§1e)
    /* +0x234 */ rail_glass glass; // [H] the breakable windows (§1f)
    /* +0x248 */ S_RailSeats seats; // [H] the seat banks (§1b)
    /* +0x250 */ uint8_t passengers_alloc; // [H] allocator byte of the passenger vector
    /* +0x254 */ S_rail_human ** passengers_begin; // [H] the passenger list, begin
    /* +0x258 */ S_rail_human ** passengers_end; // [H] the passenger list, end
    /* +0x25c */ S_rail_human ** passengers_cap; // [H] end of the list's storage
    /* +0x260 */ uint8_t unread_vector_260_alloc; // [U] ctor; no reader (.text scan)
    /* +0x264 */ void * unread_vector_260_begin; // [U] ctor and dtor; the dtor's load only feeds the free (.text scan)
    /* +0x268 */ void * unread_vector_260_end; // [U] ctor and dtor; no reader (.text scan)
    /* +0x26c */ void * unread_vector_260_cap; // [U] ctor and dtor; no reader (.text scan)
    /* +0x270 */ S_rail_entry * inner_entries; // [H] published inner bank, stride 0x38
    /* +0x274 */ S_rail_entry * outer_entries; // [H] published outer bank, stride 0x38
    /* +0x278 */ uint8_t person_slots_alloc; // [H] allocator byte of the 0x24-stride vector
    /* +0x27c */ void * person_slots_begin; // [H] records of stride 0x24; +0x1c holds an S_rail_human *
    /* +0x280 */ void * person_slots_end; // [H] one past the last record
    /* +0x284 */ void * person_slots_cap; // [H] end of the vector's storage
    /* +0x288 */ uint32_t boarding_lock; // [H] counter: railway_lock raises, railway_unlock lowers
    /* +0x28c */ void * sound_running; // [H] sound object 1, the running loop (§5b)
    /* +0x290 */ void * sound_bell_or_door; // [H] sound object 2 (§5b)
    /* +0x294 */ void * sound_squeal; // [H] sound object 3, the curve squeal (§5b)
    /* +0x298 */ void * unread_298; // [U] only 0 stored, null-tested before release; no non-zero store (.text scan)
    /* +0x29c */ uint32_t sound_id_running; // [H] sounds.dat id of object 1
    /* +0x2a0 */ uint32_t sound_id_bell_or_door; // [H] sounds.dat id of object 2
    /* +0x2a4 */ uint32_t sound_id_squeal; // [H] sounds.dat id of object 3
    /* +0x2a8 */ uint32_t range_state; // [H] in-range state; 0 or 2 is eligible
    /* +0x2ac */ uint32_t material_id; // [H] collision material; written by railway_hit
    /* +0x2b0 */ S_rail_human driver; // [H] the driver, embedded (§1d)
    /* +0x340 */ float bound_mid_z; // [H] the bound box's local z mid-point; the seat-side test
    /* +0x344 */ float bound_volume[10]; // [H] copy of the frame's bound volume: box min, box max, sphere
} C_railway;
```

### 1b. `S_RailSeats`

```c
typedef struct S_RailSeats {          // 8 bytes, at C_railway + 0x248
    /* +0x00 */ uint32_t row_count;       // [H] rows of all carriages
    /* +0x04 */ S_rail_seat_row *rows;    // [H] the row array, stride 0x38
} S_RailSeats;
```

### 1c. `S_rail_seat_row`

```c
typedef struct S_rail_seat_row {      // 0x38 bytes; a row is a bench
    /* +0x00 */ S_vector start;           // [H] one end of the bench
    /* +0x0c */ S_vector end;             // [H] the other end; seats interpolate between the two
    /* +0x18 */ S_vector facing;          // [H] the facing vector
    /* +0x24 */ uint32_t base_seat;       // [H] the row's first seat index
    /* +0x28 */ uint32_t seat_count;      // [H] seats on this bench
    /* +0x2c */ uint32_t carriage;        // [H] the carriage index, 0 the engine
    /* +0x30 */ uint8_t  *occupancy;      // [H] one state per seat, §6a
    /* +0x34 */ void    **owners;         // [H] one owner per seat
} S_rail_seat_row;
```

### 1d. `S_rail_human`

```c
typedef struct S_rail_human {         // 0x90 bytes (operator_new in railway_game_init)
    /* +0x00 */ uint32_t state;           // [H] the state switch of railway_traffic_ai; 0x11 dead, 0x12 alighted, 0x15 / 0x16 panic
    /* +0x04 */ I3D_frame *frame;         // [H] the passenger's frame
    /* +0x08 */ C_animations_machine_de_luxe *anim; // [H] a heap machine (structs/anim_machine.md §4)
    /* +0x0c */ uint32_t stops_left;      // [H] stops before the passenger alights; railway_ai counts it down
    /* +0x10 */ uint32_t applied_anim;    // [H] the clip rail_passenger_set_anim last applied (§2d)
    /* +0x14 */ uint32_t requested_anim;  // [H] the clip rail_passenger_set_anim applies when it differs
    /* +0x18 */ uint32_t carriage;        // [H] drawn by railway_game_init with rand_range(vagon count + 1)
    /* +0x1c */ uint32_t seat;            // [H] the seat handed to rail_seats_empty_seat
    /* +0x20 */ uint8_t  anim_finished;   // [H] rail_passenger_set_anim clears it
    /* +0x24 */ S_vector walk_target;     // [H] the point railway_traffic_ai walks the passenger to
    /* +0x30 */ S_vector walk_target_dir; // [H] the direction at that point
    /* +0x3c */ uint32_t panic_elapsed_ms; // [H] ms into the current panic
    /* +0x40 */ uint32_t panic_duration_ms; // [H] how long the panic lasts, ms
    /* +0x44 */ uint32_t saved_state;     // [H] state saved while panicking
    /* +0x48 */ uint32_t saved_anim;      // [H] applied_anim saved while panicking
    /* +0x4c */ S_matrix inverse;         // [H] the inverse transform for the collision record
    /* +0x8c */ void    *collision;       // [H] the published 0x48-byte dynamic-collision record (type 0x40)
} S_rail_human;
```

### 1e. `rail_doors`

```c
typedef struct rail_doors {           // 0x4c bytes, at C_railway + 0x1e8
    /* +0x00 */ I3D_frame **leaves_a;     // [H] the first bank's leaf frames
    /* +0x04 */ I3D_frame **leaves_b;     // [H] the second bank's leaf frames
    /* +0x08 */ uint32_t state;           // [H] 0 closed, 1 open, 2 closing, 3 opening
    /* +0x0c */ uint32_t leaf_count_a;    // [H] entries of leaves_a
    /* +0x10 */ uint32_t leaf_count_b;    // [H] entries of leaves_b
    /* +0x14 */ S_vector axis;            // [H] the hinge axis; init (0, 1, 0)
    /* +0x20 */ S_quat   leaf_rot_a;      // [H] the first bank's current leaf rotation, Make(axis, -angle)
    /* +0x30 */ S_quat   leaf_rot_b;      // [H] the second bank's current leaf rotation, Make(axis, +angle)
    /* +0x40 */ float    angle;           // [H] the shared angle, 0 to the maximum
    /* +0x44 */ float    speed;           // [H] radians per second, init 1.0
    /* +0x48 */ float    open_angle;      // [H] init pi / 2; no reader (.text scan)
} rail_doors;
```

### 1f. `rail_glass`

```c
typedef struct rail_glass {           // 0x14 bytes, at C_railway + 0x234
    /* +0x00 */ uint32_t unread_00;       // [U] stored 0 by railway_enumerate_scene_frames; no reader (class census)
    /* +0x04 */ uint32_t window_count;    // [H] windows in the array
    /* +0x08 */ rail_window *windows;     // [H] the array, stride 0x14
    /* +0x0c */ void    *source_material; // [H] the first window's material
    /* +0x10 */ void    *cracked_material; // [H] the per-vehicle clone window hits swap in
} rail_glass;
```

### 1g. `rail_window`

```c
typedef struct rail_window {          // 0x14 bytes, seeded {1, 0, -1, 0, 0}
    /* +0x00 */ uint32_t hit_state;       // [H] 1 intact, 2 cracked; the next hit breaks it
    /* +0x04 */ I3D_frame *frame;         // [H] the window's frame; rail_window_hit reads its +0x1e0
    /* +0x08 */ uint32_t carriage;        // [H] the carriage index: seed -1, 0 the engine, i + 1 vagon i
    /* +0x0c */ void    *crack_material;  // [H] the clone rail_window_hit binds while cracked, released by rail_window_restore
    /* +0x10 */ void    *saved_material;  // [H] the original material kept for rail_window_restore
} rail_window;
```

### 1h. `rail_entries`

```c
typedef struct rail_entries {         // 8 bytes, at C_railway + 0x1e0
    /* +0x00 */ uint32_t count;           // [H] source entries
    /* +0x04 */ void    *entries;         // [H] source records, stride 0x18: start point, end point, flags
} rail_entries;
```

## 2. Field evidence

### 2a. `C_railway`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | direction | H | `railway_game_init`, `railway_place_on_track`, `railway_save_game_load` | `railway_place_on_track` (by address, `S_vector::AngleTo`) | the vector from the front bogie to the target node `back_nodes_begin[0]`; zeroed by `railway_game_init` and `railway_save_game_load`; `railway_place_on_track` seeds it when shorter than 1e-8 and rewrites it at each passed node. A node counts as passed when the angle to the fresh to-node vector is 90 degrees or more, or the node is closer than 0.1 (`ai_flags` bit 1; bit 2 a stop point, bit 4 a semaphore) |
| +0x7c | back_nodes_alloc | H | `railway_ctor` | `railway_build_back_node_vector`, `railway_dtor`, `railway_game_done`, `railway_game_init`, `railway_place_on_track`, `railway_save_game_load` | - |
| +0x80 | back_nodes_begin | H | `railway_ctor` | `railway_ai`, `railway_fill_track_curve`, `railway_find_back_pos`, `railway_find_vehicle_ahead`, `railway_game_done`, `railway_game_init`, `railway_get_safe_speed`, `railway_must_stop`, `railway_place_on_track`, `railway_save_game_load`, `railway_save_game_save`, `railway_update_curve_squeal_sound` | - |
| +0x84 | back_nodes_end | H | `railway_ctor` | `railway_build_back_node_vector`, `railway_find_back_pos`, `railway_find_vehicle_ahead`, `railway_game_done`, `railway_game_init`, `railway_must_stop`, `railway_place_on_track`, `railway_update_curve_squeal_sound` | - |
| +0x88 | back_nodes_cap | H | `railway_ctor` | - | - |
| +0x8c | back_path_radius | H | `railway_game_init` | `railway_build_back_node_vector`, `railway_place_on_track` | `(vagon_count + 1) * vagon_length + 30.0`; the back builder stops 3 nodes past it; `railway_place_on_track` trims the back path's tail while it is farther than this from the front bogie |
| +0x90 | front_position | H | `railway_game_init`, `railway_place_on_track`, `railway_save_game_load` | `railway_ai`, `railway_build_back_node_vector`, `railway_clamp_movement_against_traffic`, `railway_collide`, `railway_fill_track_curve`, `railway_get_safe_speed`, `railway_must_stop`, `railway_place_on_track` | `pos + dir * bogie_spacing / 2`, snapped to the nearest web node; advanced by `dir * speed * dt` each frame; `railway_ai` tests it against the camera frame (500 units), `railway_collide` sweeps from it, 0.7 up |
| +0x9c | rear_position | H | `railway_game_init`, `railway_save_game_load`, `railway_place_on_track` (by address, `railway_find_back_pos`) | `railway_game_init`, `railway_place_on_track` (by address, `railway_spring_process`) | `pos - dir * bogie_spacing / 2`; each frame `railway_find_back_pos` sets it `bogie_spacing` behind the front bogie and `railway_spring_process` settles the engine between the two |
| +0xa8 | forward_nodes_alloc | H | `railway_ctor` | `railway_build_forward_node_vector`, `railway_game_done`, `railway_game_init`, `railway_place_on_track`, `railway_save_game_load` | - |
| +0xac | forward_nodes_begin | H | `railway_ctor`, `railway_dtor` | `railway_build_forward_node_vector`, `railway_dtor`, `railway_fill_track_curve`, `railway_game_done`, `railway_game_init`, `railway_get_safe_speed`, `railway_must_stop`, `railway_place_on_track`, `railway_save_game_load`, `railway_update_curve_squeal_sound` | - |
| +0xb0 | forward_nodes_end | H | `railway_ctor`, `railway_dtor` | `railway_game_done`, `railway_game_init`, `railway_must_stop`, `railway_save_game_load`, `railway_update_curve_squeal_sound` | - |
| +0xb4 | forward_nodes_cap | H | `railway_ctor`, `railway_dtor` | - | - |
| +0xb8 | spring_phases | H | `railway_game_init` | `railway_game_done`, `railway_game_init`, `railway_spring_process` | `float[vagon_count + 1]`, seeded `rand * pi`; `railway_spring_process` advances each entry by `dt / (1000 / (speed / speed_scale * 12))`, wraps it at 2 pi and sways the body by `sin(phase + pi/2) * 0.03 * speed / speed_scale`; freed by `railway_game_done` |
| +0xbc | vagon_positions | H | `railway_game_init` | `railway_game_done`, `railway_game_init`, `railway_get_nearest_vagon`, `railway_is_possible_coll`, `railway_place_on_track`, `railway_update` | - |
| +0xc0 | vagon_directions | H | `railway_game_init` | `railway_game_done`, `railway_game_init`, `railway_is_possible_coll`, `railway_place_on_track`, `railway_update` | `S_vector[vagon_count]`, seeded with the engine direction; `railway_spring_process` fills entries through an out argument; `railway_update` hands entry i to vagon i's SetDir; `railway_is_possible_coll` sweeps spheres along it |
| +0xc4 | vagon_frames | H | `railway_game_done`, `railway_game_init` | `railway_ai`, `railway_enumerate_scene_frames`, `railway_export_entries`, `railway_game_done`, `railway_game_init`, `railway_gen_entries`, `railway_get_bound_points_2d`, `railway_get_outer_entry_at`, `railway_get_outer_entry_for_actor`, `railway_get_outer_entry_near`, `railway_get_vagon`, `railway_on_act_state_changed`, `railway_spring_process`, `railway_traffic_ai`, `railway_update` | - |
| +0xc8 | position_deltas | H | `railway_game_init` | `railway_ai`, `railway_game_done`, `railway_game_init`, `railway_place_on_track` | `S_vector[vagon_count + 1]`: `railway_place_on_track` stores each body's old position, runs `railway_spring_process`, then stores new minus old; `railway_ai` state 6 and `railway_game_init` zero it, `railway_game_done` frees it; no consumer (.text scan) |
| +0xcc | direction_deltas | H | `railway_game_init` | `railway_ai`, `railway_game_done`, `railway_game_init`, `railway_place_on_track` | the same for each body's direction; no consumer (.text scan) |
| +0xd0 | target_speed | H | `railway_ai`, `railway_game_init` | `railway_ai`, `railway_save_game_load`, `railway_save_game_save` | - |
| +0xd4 | speed | H | `railway_ai`, `railway_game_init` | `railway_ai`, `railway_clamp_movement_against_traffic`, `railway_collide`, `railway_fill_track_curve`, `railway_get_actor_speed`, `railway_hit`, `railway_is_in_stop`, `railway_is_possible_coll`, `railway_place_on_track`, `railway_save_game_load`, `railway_save_game_save`, `railway_spring_process`, `railway_update_curve_squeal_sound` | - |
| +0xd8 | stop_timer_ms | H | `railway_ai`, `railway_ctor`, `railway_game_init` | `railway_ai`, `railway_save_game_load`, `railway_save_game_save` | - |
| +0xdc | run_state | H | `railway_ai`, `railway_ctor`, `railway_set_state`, `railway_update` | `railway_ai`, `railway_hit`, `railway_hold_behind_vehicle_ahead`, `railway_is_in_station`, `railway_is_in_stop`, `railway_is_possible_coll`, `railway_save_game_load`, `railway_save_game_save`, `railway_update` | - |
| +0xe0 | held_state | H | `railway_hit`, `railway_hold_behind_vehicle_ahead` | `railway_ai`, `railway_update` | - |
| +0xe4 | ai_flags | H | `railway_ai`, `railway_place_on_track` | `railway_ai`, `railway_place_on_track`, `railway_save_game_load`, `railway_save_game_save` | - |
| +0xe8 | vagon_collisions | H | `railway_game_init` | `railway_collide`, `railway_game_done`, `railway_game_init`, `railway_hit`, `railway_on_act_state_changed`, `railway_on_collision`, `railway_update` | - |
| +0xec | coll_records_alloc | H | `railway_ctor` | `railway_enumerate_scene_frames`, `railway_game_done` | - |
| +0xf0 | coll_records_begin | H | `railway_ctor`, `railway_dtor` | `railway_dtor`, `railway_enumerate_scene_frames`, `railway_game_done` | `RAIL_COLL` dynamic-collision records, `0x48` bytes, type `0x40`, owner `this`, two heap matrices at `+0x3c` / `+0x40`: `railway_enumerate_scene_frames` builds and inserts them; `railway_game_done` frees each record's two matrices and the record |
| +0xf4 | coll_records_end | H | `railway_ctor`, `railway_dtor` | `railway_enumerate_scene_frames` | - |
| +0xf8 | coll_records_cap | H | `railway_ctor`, `railway_dtor` | - | - |
| +0xfc | interior_coll_lists | H | `railway_game_init` | `railway_enumerate_scene_frames`, `railway_game_done`, `railway_game_init`, `railway_traffic_ai` | `vagon_count + 1` collision-list vectors of 16 bytes (`crt_ehvec_ctor`, `gif_vector_ctor`); `railway_enumerate_scene_frames` adds every `RAIL_COLL` record to each list (`collision_dyn_take_out_to_list`); `railway_traffic_ai` (boarding states 5 and 6) and `railway_game_init` (the seated start passengers) drop a passenger onto the floor with `collision_test_line_in_vehicle` on the carriage's list; destroyed by `railway_game_done` (`crt_ehvec_dtor`) |
| +0x100 | added_persons_alloc | H | `railway_ctor` | `railway_add_person`, `railway_on_collision`, `railway_update` | - |
| +0x104 | added_persons_begin | H | `railway_ctor`, `railway_dtor` | `railway_dtor`, `railway_game_done`, `railway_hit`, `railway_on_collision`, `railway_traffic_ai`, `railway_update` | the element points at a `0x48`-byte dynamic-collision record: type `0x42`, flags `0x2002`, `+0x04` the boarding person's frame, `+0x44` owner `this`; `railway_add_person` builds and inserts it (`collision_dyn_insert`); `railway_traffic_ai` state 5 (boarded), `railway_on_collision`, `railway_hit`, `railway_update` (out of range) and `railway_game_done` erase it (`collision_dyn_erase`); `railway_update` refreshes each record's centre |
| +0x108 | added_persons_end | H | `railway_ctor`, `railway_dtor`, `railway_game_done`, `railway_hit`, `railway_traffic_ai` | `railway_add_person`, `railway_game_done`, `railway_hit`, `railway_traffic_ai`, `railway_update` | - |
| +0x10c | added_persons_cap | H | `railway_ctor`, `railway_dtor` | `railway_add_person` | - |
| +0x110 | vagon_matrices | H | `railway_game_init` | `railway_game_done`, `railway_game_init`, `railway_update` | - |
| +0x114 | unused_114 | U | - | - | `0x44` bytes; no load or store (.text scan); the four `railway_traffic_ai` hits at `+0x120` walk an `I3D_frame` parent chain |
| +0x158 | cycle_accum_ms | H | `railway_ai`, `railway_ctor`, `railway_dtor` | `railway_ai` | - |
| +0x15c | cycle_period_ms | H | `railway_ai`, `railway_ctor`, `railway_dtor` | `railway_ai` | - |
| +0x160 | vehicle_ahead | H | `railway_ctor`, `railway_find_vehicle_ahead`, `railway_game_done` | `railway_ai`, `railway_find_vehicle_ahead`, `railway_hold_behind_vehicle_ahead`, `railway_update` | - |
| +0x164 | dummy_col_frame | H | `railway_ctor`, `railway_game_init` | `railway_game_init`, `railway_update` | - |
| +0x168 | dummy_col_matrix | H | `railway_game_init`, `railway_update` | `railway_game_init`, `railway_update` | an `S_matrix` to `+0x1a7`: the `DUMMY_COL` frame's world matrix with rows 0, 1, 2 divided by the frame's scale; both writers copy it into `vagon_matrices[0]` and invert that; `railway_game_init` points the engine's collision record `+0x3c` at it, so the collision system reads it |
| +0x1a8 | vagon_count | H | `railway_ctor`, `railway_duplicate_from` | `railway_add_inner_entries`, `railway_add_outer_entries`, `railway_add_person`, `railway_ai`, `railway_enumerate_scene_frames`, `railway_export_entries`, `railway_game_done`, `railway_game_init`, `railway_gen_entries`, `railway_get_bound_points_2d`, `railway_get_nearest_vagon`, `railway_get_outer_entry_at`, `railway_get_outer_entry_for_actor`, `railway_get_outer_entry_near`, `railway_get_vagon`, `railway_hit`, `railway_is_possible_coll`, `railway_load_data`, `railway_on_act_state_changed`, `railway_on_collision`, `railway_place_on_track`, `railway_remove_inner_entries`, `railway_remove_outer_entries`, `railway_save_data`, `railway_update` | read by `railway_load_data` from the scene |
| +0x1ac | bogie_spacing | H | `railway_ctor` | `railway_game_init`, `railway_load_data`, `railway_place_on_track`, `railway_save_data`, `railway_save_game_load` | - |
| +0x1b0 | vagon_length | H | `railway_ctor` | `railway_game_init`, `railway_is_possible_coll`, `railway_load_data`, `railway_place_on_track`, `railway_save_data` | - |
| +0x1b4 | speed_scale | H | `railway_ctor`, `railway_duplicate_from` | `railway_ai`, `railway_game_init`, `railway_get_safe_speed`, `railway_load_data`, `railway_save_data`, `railway_save_game_load`, `railway_save_game_save`, `railway_spring_process` | - |
| +0x1b8 | acceleration | H | `railway_ctor`, `railway_load_data`, `railway_save_game_load` | `railway_ai`, `railway_place_on_track`, `railway_save_data`, `railway_save_game_save` | designer value (units/s^2), ctor 3.0, scene chunk; `railway_ai` changes the speed by `dt * 0.001 * acceleration`; `railway_place_on_track` derives `braking_distance` from it |
| +0x1bc | mass | H | `railway_ctor`, `railway_load_data` | `railway_collide`, `railway_game_init`, `railway_save_data` | designer value, ctor 10000.0; `railway_game_init` derives `inverse_inertia`; `railway_collide` uses `1 / mass` as the impulse's inverse mass and passes the mass to the collidee's hit callback (vtable `+0x78`) |
| +0x1c0 | unread_1c0 | U | `railway_ctor` | - | stored 0.0001; no reader (.text scan) |
| +0x1c4 | braking_distance | H | `railway_place_on_track`, `railway_save_game_load` | `railway_ai`, `railway_clamp_movement_against_traffic`, `railway_get_safe_speed`, `railway_place_on_track`, `railway_save_game_save` | `speed^2 / (2 * acceleration)` each frame; `railway_ai` state 0 brakes for a stop point inside it (state 9); `railway_get_safe_speed` looks `+ 4.0` ahead, `railway_clamp_movement_against_traffic` `+ 4.3` (at least 4.0); `railway_place_on_track` and `railway_save_game_load` build the forward path `+ 5.0` long; saved with the game |
| +0x1c8 | inverse_inertia | H | `railway_game_init` | `railway_collide` | `1 / (avg_extent^2 * mass * 0.25)`; `railway_collide` scales the contact lever cross products by it (car and box arms) |
| +0x1cc | contact_velocity | H | `railway_collide`, `railway_save_game_load` | `railway_collide`, `railway_save_game_save` | `dir * speed` stored before each impulse (the box arm stores the contact solver's output); the car-arm impulse reads it as the relative velocity; 12 bytes in the savegame |
| +0x1d8 | is_elevated | H | `railway_game_init` | `railway_ai`, `railway_clamp_movement_against_traffic`, `railway_game_done`, `railway_game_init`, `railway_gen_entries`, `railway_get_safe_speed`, `railway_is_in_stop`, `railway_is_possible_coll`, `railway_on_act_state_changed`, `railway_remove_person`, `railway_traffic_ai`, `railway_update` | - |
| +0x1d9 | collision_latch | H | `railway_clamp_movement_against_traffic`, `railway_collide`, `railway_ctor`, `railway_game_init`, `railway_on_collision` | `railway_clamp_movement_against_traffic` | set to 1 by `railway_on_collision` (a collider touched one of the tram's records) and by `railway_collide` (after the human, car and kind `0x13` arms); `railway_clamp_movement_against_traffic` clears it and returns blocked once, so `railway_ai` state 0 goes to state 7 (stop for traffic); `railway_game_init` clears it, the ctor copies `g_ambient_ai_suspended` (0) |
| +0x1dc | semaphore_phase | H | `railway_ctor`, `railway_is_semaphore_point`, `railway_save_game_load` | `railway_ai`, `railway_is_semaphore_point` | ctor 0; set to the mission object's `+0x19c` or `+0x1a0` (`g_act_mission +0x24`), by whether the heading runs more along x or z; `railway_is_semaphore_point` and `railway_ai` state 10 index `g_railway_semaphore_states` with the phase it points at and test bits 1 / 4 |
| +0x1e0 | entries | H | `railway_enumerate_scene_frames` (0, then by address, `rail_entries_add_entry`) | `railway_add_inner_entries`, `railway_add_outer_entries`, `railway_add_person`, `railway_enumerate_scene_frames`, `railway_export_entries`, `railway_game_done`, `railway_gen_entries`, `railway_get_inner_entry_for_actor`, `railway_get_inner_entry_near`, `railway_get_outer_entry_at`, `railway_get_outer_entry_for_actor`, `railway_get_outer_entry_near`, `railway_remove_inner_entries`, `railway_remove_outer_entries` | embedded |
| +0x1e8 | doors | H | `railway_ai` (by address, `rail_doors_tick`, `rail_doors_open`, `rail_doors_close`), `railway_enumerate_scene_frames` (0, then by address, `rail_doors_init`, `rail_doors_add_leaf_frame`), `railway_update` (by address, `rail_doors_place`) | `railway_ai`, `railway_enumerate_scene_frames`, `railway_game_done`, `railway_update` | embedded, reached by `lea` |
| +0x234 | glass | H | `railway_enumerate_scene_frames` (0, then by address, `rail_glass_add_window`), `railway_game_done` (by address, `rail_glass_game_done`), `railway_game_init` (by address, `rail_glass_game_init`) | `railway_enumerate_scene_frames`, `railway_game_done`, `railway_game_init`, `railway_hit`, `railway_on_collision` | embedded, reached by `lea` |
| +0x248 | seats | H | `railway_enumerate_scene_frames` (0, then by address, `rail_seats_add_row`), `railway_game_init` (by address, `rail_seats_init`) | `railway_add_person`, `railway_enumerate_scene_frames`, `railway_game_done`, `railway_game_init`, `railway_hit`, `railway_on_collision`, `railway_traffic_ai`, `railway_update` | embedded, reached by `lea` |
| +0x250 | passengers_alloc | H | `railway_ctor` | `railway_add_person`, `railway_game_done`, `railway_game_init`, `railway_hit`, `railway_update` | - |
| +0x254 | passengers_begin | H | `railway_ctor`, `railway_dtor` | `railway_ai`, `railway_any_passenger_wants_off`, `railway_anyone_waiting_to_board`, `railway_do_panic`, `railway_dtor`, `railway_game_done`, `railway_hit`, `railway_on_collision`, `railway_remove_person`, `railway_traffic_ai`, `railway_update` | - |
| +0x258 | passengers_end | H | `railway_ctor`, `railway_dtor`, `railway_on_collision`, `railway_remove_person` | `railway_add_person`, `railway_ai`, `railway_any_passenger_wants_off`, `railway_anyone_waiting_to_board`, `railway_do_panic`, `railway_game_init`, `railway_hit`, `railway_on_collision`, `railway_remove_person`, `railway_traffic_ai`, `railway_update` | - |
| +0x25c | passengers_cap | H | `railway_ctor`, `railway_dtor` | - | - |
| +0x260 | unread_vector_260_alloc | U | `railway_ctor` | - | no reader (.text scan) |
| +0x264 | unread_vector_260_begin | U | `railway_ctor`, `railway_dtor` | `railway_dtor` | the dtor's load only feeds the free; no other load (.text scan) |
| +0x268 | unread_vector_260_end | U | `railway_ctor`, `railway_dtor` | - | no reader (.text scan) |
| +0x26c | unread_vector_260_cap | U | `railway_ctor`, `railway_dtor` | - | no reader (.text scan) |
| +0x270 | inner_entries | H | `railway_gen_entries` | `railway_add_inner_entries`, `railway_destroy_entries`, `railway_gen_entries`, `railway_remove_inner_entries` | - |
| +0x274 | outer_entries | H | `railway_gen_entries` | `railway_add_outer_entries`, `railway_destroy_entries`, `railway_gen_entries`, `railway_remove_outer_entries` | - |
| +0x278 | person_slots_alloc | H | `railway_ctor` | `railway_game_done`, `railway_game_init` | - |
| +0x27c | person_slots_begin | H | `railway_ctor`, `railway_dtor` | `railway_dtor`, `railway_game_done`, `railway_game_init`, `railway_update` | - |
| +0x280 | person_slots_end | H | `railway_ctor`, `railway_dtor` | `railway_game_init`, `railway_update` | - |
| +0x284 | person_slots_cap | H | `railway_ctor`, `railway_dtor` | - | - |
| +0x288 | boarding_lock | H | `railway_game_init`, `railway_lock`, `railway_unlock`, `railway_update` | `railway_ai`, `railway_unlock` | - |
| +0x28c | sound_running | H | `railway_ctor` | `railway_ai`, `railway_game_done`, `railway_game_init`, `railway_on_act_state_changed`, `railway_set_state`, `railway_update` | - |
| +0x290 | sound_bell_or_door | H | `railway_ctor` | `railway_ai` | - |
| +0x294 | sound_squeal | H | `railway_ctor` | `railway_ai`, `railway_update`, `railway_update_curve_squeal_sound` | - |
| +0x298 | unread_298 | U | `railway_game_done`, `railway_game_init`, `railway_on_act_state_changed` | `railway_game_done`, `railway_on_act_state_changed`, `railway_update` | only 0 is stored; the loads are null tests before the Update and release calls the three sound objects also get; no non-zero store (.text scan) |
| +0x29c | sound_id_running | H | `railway_game_init` | - | - |
| +0x2a0 | sound_id_bell_or_door | H | `railway_game_init` | `railway_ai` | - |
| +0x2a4 | sound_id_squeal | H | `railway_game_init` | - | - |
| +0x2a8 | range_state | H | `railway_ai`, `railway_game_init` | `railway_ai`, `railway_any_passenger_wants_off`, `railway_anyone_waiting_to_board`, `railway_save_game_save`, `railway_update` | - |
| +0x2ac | material_id | H | `railway_hit` | `railway_get_material_id` | - |
| +0x2b0 | driver | H | `railway_game_init` | `railway_ctor`, `railway_do_panic`, `railway_game_done`, `railway_game_init`, `railway_hit` | - |
| +0x340 | bound_mid_z | H | `railway_game_init` | `railway_traffic_ai` | `min.z + (max.z - min.z) / 2` of the engine frame's bound box; `railway_traffic_ai` state 6 passes it to `rail_seats_find_free_seat`, which picks a seat on the passenger's side of that z |
| +0x344 | bound_volume | H | `railway_game_init` (a 10-dword copy of frame `+0xb0`) | `railway_traffic_ai` | box min `+0x344`, box max `+0x350`, then the sphere, to the instance end `+0x36b`; `railway_traffic_ai` state `0x11` clamps a dead passenger's x into `[+0x344, +0x350]` and z into `[+0x34c, +0x358]` so the body stays inside the carriage |

### 2b. `S_RailSeats`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | row_count | H | `rail_seats_add_row`, `rail_seats_duplicate_rows` | `rail_seats_add_row`, `rail_seats_count_free`, `rail_seats_destroy`, `rail_seats_duplicate_rows`, `rail_seats_empty_seat`, `rail_seats_find_free_seat`, `rail_seats_get_seat_dir`, `rail_seats_get_seat_pos`, `rail_seats_init`, `rail_seats_occupy_seat`, `rail_seats_reserve_seat` | - |
| +0x4 | rows | H | `rail_seats_add_row`, `rail_seats_duplicate_rows` | `rail_seats_add_row`, `rail_seats_count_free`, `rail_seats_destroy`, `rail_seats_duplicate_rows`, `rail_seats_empty_seat`, `rail_seats_find_free_seat`, `rail_seats_get_seat_dir`, `rail_seats_get_seat_pos`, `rail_seats_init`, `rail_seats_occupy_seat`, `rail_seats_reserve_seat` | - |

### 2c. `S_rail_seat_row`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | start | H | `rail_seats_add_row` | `rail_seats_get_seat_pos` | through the row pointer, which the census cannot follow |
| +0xc | end | H | `rail_seats_add_row` | `rail_seats_get_seat_pos` | - |
| +0x18 | facing | H | `rail_seats_add_row` | `rail_seats_get_seat_dir` | - |
| +0x24 | base_seat | H | `rail_seats_add_row`, `rail_seats_init` | `rail_seats_find_free_seat` | - |
| +0x28 | seat_count | H | `rail_seats_add_row` | `rail_seats_find_free_seat`, `rail_seats_count_free` | - |
| +0x2c | carriage | H | `rail_seats_init` | `rail_seats_find_free_seat`, `rail_seats_count_free` | - |
| +0x30 | occupancy | H | `rail_seats_init` | `rail_seats_empty_seat`, `rail_seats_reserve_seat`, `rail_seats_occupy_seat` | §6a |
| +0x34 | owners | H | `rail_seats_init` | `rail_seats_empty_seat`, `rail_seats_reserve_seat`, `rail_seats_occupy_seat` | - |

### 2d. `S_rail_human`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | state | H | `rail_passenger_do_panic`, `rail_passenger_kill` | `rail_passenger_do_panic`, `rail_passenger_kill`, `railway_any_passenger_wants_off`, `railway_traffic_ai` | - |
| +0x4 | frame | H | `railway_game_init` | `rail_passenger_create_coll`, `rail_passenger_is_woman`, `rail_passenger_move_to`, `rail_passenger_rotate_to`, `rail_passenger_set_anim`, `rail_passenger_update_coll` | - |
| +0x8 | anim | H | `rail_passenger_done`, `railway_game_init` | `rail_passenger_done`, `rail_passenger_set_anim` | - |
| +0xc | stops_left | H | `railway_game_init`, `railway_add_person`, `railway_ai` | `railway_ai`, `railway_any_passenger_wants_off`, `railway_traffic_ai` | `railway_game_init` stores `rand + 3`, `railway_add_person` `rand + 2`, the driver -1; `railway_ai` state 1 decrements it for every passenger at each arrival; `railway_any_passenger_wants_off` and `railway_traffic_ai` state 9 leave the seat when it is 0 |
| +0x10 | applied_anim | H | `rail_passenger_set_anim` | `rail_passenger_do_panic`, `rail_passenger_set_anim` | carries the boarding entry index (or -1) from `railway_add_person` until `railway_traffic_ai` state 0 passes it to `railway_get_outer_entry_at` and stores -1 |
| +0x14 | requested_anim | H | `rail_passenger_do_panic`, `rail_passenger_kill`, `rail_passenger_move_to`, `rail_passenger_rotate_to` | `rail_passenger_set_anim` | - |
| +0x18 | carriage | H | `railway_game_init`, `railway_add_person`, `railway_traffic_ai` (by address, `railway_get_outer_entry_near`) | `railway_do_panic` | - |
| +0x1c | seat | H | `railway_game_init`, `railway_traffic_ai` | `railway_hit`, `railway_update` | `railway_traffic_ai` state 6 |
| +0x20 | anim_finished | H | `rail_passenger_set_anim` | `railway_traffic_ai` | - |
| +0x24 | walk_target | H | `railway_traffic_ai` | `railway_traffic_ai` | the point the passenger walks to |
| +0x30 | walk_target_dir | H | `railway_traffic_ai` | `railway_traffic_ai` | the direction at that point |
| +0x3c | panic_elapsed_ms | H | `rail_passenger_ctor`, `rail_passenger_do_panic`, `railway_traffic_ai` | `railway_traffic_ai` | zeroed by `rail_passenger_do_panic`; `railway_traffic_ai` states `0x15` / `0x16` add the frame time and end the panic when it passes `panic_duration_ms` |
| +0x40 | panic_duration_ms | H | `rail_passenger_ctor`, `rail_passenger_do_panic` | `railway_traffic_ai` | the argument of `rail_passenger_do_panic`: `railway_do_panic` passes `rand_float * 10000` truncated; `railway_hit` calls `railway_do_panic` with 10000 and the hit carriage |
| +0x44 | saved_state | H | `rail_passenger_ctor`, `rail_passenger_do_panic` | `rail_passenger_kill`, `railway_traffic_ai` | - |
| +0x48 | saved_anim | H | `rail_passenger_do_panic` | `railway_traffic_ai` | state `0x16` restores it as `requested_anim` together with `saved_state` |
| +0x4c | inverse | H | `rail_passenger_create_coll`, `rail_passenger_update_coll` (by address, `S_matrix::Inverse`) | `rail_passenger_create_coll`, `rail_passenger_update_coll` | - |
| +0x8c | collision | H | `rail_passenger_create_coll`, `rail_passenger_ctor`, `rail_passenger_done` | `rail_passenger_create_coll`, `rail_passenger_done`, `rail_passenger_update_coll` | - |

### 2e. `rail_doors`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | leaves_a | H | `rail_doors_add_leaf_frame` | `rail_doors_add_leaf_frame`, `rail_doors_destroy`, `rail_doors_place` | - |
| +0x4 | leaves_b | H | `rail_doors_add_leaf_frame` | `rail_doors_add_leaf_frame`, `rail_doors_destroy`, `rail_doors_place` | - |
| +0x8 | state | H | `rail_doors_close`, `rail_doors_init`, `rail_doors_open`, `rail_doors_tick` | `rail_doors_close`, `rail_doors_open`, `rail_doors_tick` | - |
| +0xc | leaf_count_a | H | `rail_doors_add_leaf_frame`, `rail_doors_init` | `rail_doors_add_leaf_frame`, `rail_doors_place` | - |
| +0x10 | leaf_count_b | H | `rail_doors_add_leaf_frame`, `rail_doors_init` | `rail_doors_add_leaf_frame`, `rail_doors_place` | - |
| +0x14 | axis | H | `rail_doors_init` | `rail_doors_init`, `rail_doors_place` | - |
| +0x20 | leaf_rot_a | H | `rail_doors_place` (by address, `S_quat::Make`) | `rail_doors_place` | `Make(axis, -angle)` (`fchs`) on every call, then copied onto each leaf frame of the first bank |
| +0x30 | leaf_rot_b | H | `rail_doors_place` (by address, `S_quat::Make`) | `rail_doors_place` | `Make(axis, +angle)` for the second bank, so the banks swing in opposite directions |
| +0x40 | angle | H | `rail_doors_init`, `rail_doors_tick` | `rail_doors_place`, `rail_doors_tick` | - |
| +0x44 | speed | H | `rail_doors_init` | `rail_doors_tick` | - |
| +0x48 | open_angle | H | `rail_doors_init` | - | pi/2; no reader (.text scan); `rail_doors_tick` clamps against the literal `0x3fc90fdb` |

### 2f. `rail_glass`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | unread_00 | U | `railway_enumerate_scene_frames` | - | stored 0; no reader in the four `rail_glass_*` bodies nor in any formation of `C_railway +0x234`, each of which leads into a `rail_glass_*` method |
| +0x4 | window_count | H | `rail_glass_add_window`, `rail_glass_game_done` | `rail_glass_add_window`, `rail_glass_game_done`, `rail_glass_game_init`, `railway_hit` | - |
| +0x8 | windows | H | `rail_glass_add_window`, `rail_glass_game_done` | `rail_glass_add_window`, `rail_glass_game_done`, `rail_glass_game_init`, `railway_hit` | - |
| +0xc | source_material | H | `rail_glass_game_init` | `rail_glass_game_done`, `rail_glass_game_init` | - |
| +0x10 | cracked_material | H | `rail_glass_game_init` | `rail_glass_game_done`, `rail_glass_game_init`, `railway_hit` | - |

### 2g. `rail_window`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | hit_state | H | `rail_window_hit`, `rail_window_restore` | `rail_window_hit` | - |
| +0x4 | frame | H | `rail_glass_add_window` | `rail_window_hit`, `rail_window_restore` | - |
| +0x8 | carriage | H | `rail_glass_add_window`, `railway_enumerate_scene_frames` | `railway_hit`, `railway_on_collision` | seed -1; `railway_enumerate_scene_frames` passes 0 for the engine's windows and `i + 1` for vagon i's; the two readers match it against the hit carriage |
| +0xc | crack_material | H | `rail_window_hit`, `rail_window_restore` | `rail_window_hit`, `rail_window_restore` | `rail_window_hit` creates the clone, `rail_window_restore` releases it |
| +0x10 | saved_material | H | `rail_window_hit`, `rail_window_restore` | `rail_window_hit`, `rail_window_restore` | - |

### 2h. `rail_entries`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | count | H | `rail_entries_add_entry` | `rail_entries_add_entry` | - |
| +0x4 | entries | H | `rail_entries_add_entry` | `rail_entries_add_entry`, `rail_entries_destroy` | - |

## 3. Method family

**127 of 127 PC starts are named.** The per-class PS2 accounting is in §7. Role cells are taken from each function's docblock.

### 3a. Construction & destruction

#### `C_railway` and the file-scope vectors

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004d7300 | `crt_static_init_rail_clamp_candidates` | `g_rail_clamp_candidates_*` | - | - | - | calls the zero-init body and then the crt_atexit registrar |
| 0x004d7310 | `crt_zero_init_rail_clamp_candidates` | `g_rail_clamp_candidates_*` | - | - | - | clears the guard byte and the three vector dwords |
| 0x004d7330 | `crt_atexit_register_004d7330` | the file scope | - | - | - | calls `crt_atexit()` on `rail_clamp_candidates_static_dtor` (taken by address) |
| 0x004d7340 | `rail_clamp_candidates_static_dtor` | `g_rail_clamp_candidates_*` | - | - | - | with no guard bit, unlike the other three groups, frees `g_rail_clamp_candidates`: the list `railway_clamp_movement_against_traffic` reads; then clears its three vector dwords |
| 0x004d7360 | `crt_static_init_rail_schedule_log` | `g_rail_schedule_log_*` | - | - | - | calls the zero-init body and then the crt_atexit registrar |
| 0x004d7370 | `crt_zero_init_rail_schedule_log` | `g_rail_schedule_log_*` | - | - | - | clears the guard byte and the three vector dwords |
| 0x004d7390 | `crt_atexit_register_004d7390` | the file scope | - | - | - | calls `crt_atexit()` on `rail_schedule_log_static_dtor` (taken by address) |
| 0x004d73a0 | `rail_schedule_log_static_dtor` | `g_rail_schedule_log_*` | - | - | - | once only (mask 1 of `g_rail_static_teardown_flags`), frees `g_rail_schedule_log`: the departure log `railway_is_on_schedule` uses; then clears its three vector dwords |
| 0x004d73e0 | `crt_static_init_rail_semaphore_list` | `g_rail_semaphore_list_*` | - | - | - | calls the zero-init body and then the crt_atexit registrar |
| 0x004d73f0 | `crt_zero_init_rail_semaphore_list` | `g_rail_semaphore_list_*` | - | - | - | clears the guard byte and the three vector dwords |
| 0x004d7410 | `crt_atexit_register_004d7410` | the file scope | - | - | - | calls `crt_atexit()` on `rail_semaphore_list_static_dtor` (taken by address) |
| 0x004d7420 | `rail_semaphore_list_static_dtor` | `g_rail_semaphore_list_*` | - | - | - | once only (mask 2 of `g_rail_static_teardown_flags`), frees `g_rail_semaphore_list`: the list `railway_is_semaphore_point` searches; then clears its three vector dwords |
| 0x004d7460 | `crt_static_init_railway_list` | `g_railway_list_*` | - | - | - | calls the zero-init body and then the crt_atexit registrar |
| 0x004d7470 | `crt_zero_init_railway_list` | `g_railway_list_*` | - | - | - | clears the guard byte and the three vector dwords |
| 0x004d7490 | `crt_atexit_register_004d7490` | the file scope | - | - | - | calls `crt_atexit()` on `railway_list_static_dtor` (taken by address) |
| 0x004d74a0 | `railway_list_static_dtor` | `g_railway_list_*` | - | - | - | once only (mask 4 of `g_rail_static_teardown_flags`), frees `g_railway_list`: every live rail vehicle; then clears its three vector dwords |
| 0x004d74e0 | `railway_ctor` | instance | `__ct__9C_railwayFv` | 0x001e3c50 | CONFIRMED | chains `actor_ctor`, constructs six member vectors, constructs the embedded driver `S_rail_human` at `+0x2b0`, installs vtable `0x0063cbd0`, writes kind 8 and clears the shared cutscene-suspend flag |
| 0x004d7670 | `railway_scalar_deleting_dtor` | instance | - | - | - | calls `railway_dtor` then, when bit 0 of the flag is set, frees the object |
| 0x004d7690 | `railway_dtor` | instance | `__dt__9C_railwayFv` | 0x001e3ee0 | CONFIRMED | restores the vtable pointer, releases the dynamic-collision object when the frame type is 9, then frees the six file-scope-shaped member vectors in reverse constructor order and chains the base |

#### `S_rail_human`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004d4210 | `rail_passenger_ctor` | instance | `__ct__12S_rail_humanFv` | 0x001e0c30 | CONFIRMED | clears the panic timers at `+0x3c`/`+0x40`, sets the state at `+0x44` to 0x12 (inactive) and clears the collision pointer at `+0x8c` |

### 3b. Accessors

#### `C_railway`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e8bd0 | `railway_lock` | `Lock__9C_railwayFv` | 0x001f6620 | CONFIRMED | raises the boarding lock |
| 0x004e9cc0 | `railway_get_material_id` | `GetMaterialID__9C_railwayFP9I3D_frame` | 0x001f7bb0 | CONFIRMED | returns `material_id` |

### 3c. Behaviour

#### `C_railway`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__9C_railwayFUi` | 0x001e8810 | base-stub | Slot 15, `NetDirect` on the PS2; a shared stub serving many vtables |
| 0x004d5520 | `railway_traffic_ai` | `trafficAI__9C_railwayFUi` | 0x001e24b0 | CONFIRMED | the bulk of the vehicle's per-frame decision making — chooses the next node, honours the schedule and semaphores, opens and closes the doors at stops, and generates and retires passenger traffic |
| 0x004d7250 | `railway_any_passenger_wants_off` | `WantLeave__9C_railwayFv` | 0x001e3b30 | CONFIRMED | true when a seated passenger of the current stop wants off; excludes states `0x11` and `0x12` and a passenger with stops left |
| 0x004d72b0 | `railway_anyone_waiting_to_board` | `WantEnter__9C_railwayFv` | 0x001e3bd0 | CONFIRMED | gated on the vehicle's range state being 0 or 2; then scans the passenger list at `+0x254`/`+0x258` for a record whose state is below 6 |
| 0x004d77f0 | `railway_init` | `Init__9C_railwayFP9I3D_frame` | 0x001e4050 | CONFIRMED | refuses any frame whose type is not 9, then chains `actor_init` |
| 0x004d7820 | `railway_duplicate_from` | `DuplicateFrom__9C_railwayFP7C_actor` | 0x001e40a0 | CONFIRMED | chains the base copy, then copies exactly two fields: the vagon count at `+0x1a8` and the speed scale at `+0x1b4` |
| 0x004d7850 | `railway_save_data` | `SaveData__9C_railwayFP7C_chunk` | 0x001e40e0 | CONFIRMED | writes the vagon count and the per-line settings with `dtaWrite` |
| 0x004d7950 | `railway_load_data` | `LoadData__9C_railwayFP7C_chunk` | 0x001e4230 | CONFIRMED | reads the vagon count, models, schedule and per-line settings out of the chunk with `dtaSeek`/`dtaRead` |
| 0x004d7b60 | `railway_build_back_node_vector` | `CreateBackVector__9C_railwayFR8S_vector` | 0x001e4590 | CONFIRMED | walks the rail graph backwards along links whose flag byte has bit `0x80`, appending node indices to the back vector at `+0x80`/`+0x84`, capped at 100 nodes |
| 0x004d7ce0 | `railway_build_forward_node_vector` | `CreateForwardVector__9C_railwayFUif` | 0x001e4820 | CONFIRMED | walks the rail graph forward from the current node for the requested distance along links without bit `0x80`, appending each node index to the forward vector at `+0xac`/`+0xb0` |
| 0x004d8050 | `railway_is_semaphore_point` | `IsSemaPoint__9C_railwayFUi` | 0x001e4d30 | CONFIRMED | requires bit 8 of the node's flag byte, then searches the file-scope semaphore list for an `S_rail_semaphore` bound to that node and evaluates its state against the vehicle's heading |
| 0x004d8440 | `railway_must_stop` | `MustStop__9C_railwayFPUi` | 0x001e4f40 | CONFIRMED | checks the node ahead for stop points, semaphores and station markers and reports the reason code together with the distance at which the stop applies |
| 0x004d86d0 | `railway_find_back_pos` | `FindBackPos__9C_railwayFR8S_vectorfPi` | 0x001e52a0 | CONFIRMED | finds the track position a given distance behind a reference point |
| 0x004d8c60 | `railway_spring_process` | `SpringProcess__9C_railwayFUiiR8S_vectorR8S_vectorR8S_vectorR8S_vector` | 0x001e5a30 | CONFIRMED | settles one body segment on the rails with a damped spring |
| 0x004d9070 | `railway_ai` | `AI__9C_railwayFUi` | 0x001e5e40 | CONFIRMED | returns immediately while ambient AI is suspended; otherwise runs the traffic AI, ticks the doors, places and collides the body, computes the safe speed, plays the curve squeal, evaluates stop |
| 0x004da400 | `railway_update` | `Update__9C_railwayFUi` | 0x001e7c50 | CONFIRMED | the tram's per-frame update |
| 0x004db2c0 | `railway_game_init` | `GameInit__9C_railwayFv` | 0x001e8820 | CONFIRMED | chains `actor_game_init`, enumerates the frame tree, builds the vagon array, the seat banks, the door leaf banks and the glass set, constructs the three sound objects at `+0x28c`..`+0x294`, allocates |
| 0x004dded0 | `railway_game_done` | `GameDone__9C_railwayFv` | 0x001eb000 | CONFIRMED | chains `actor_game_done`, releases every vagon model into the mission cache, stores the actor sounds, removes the vehicle and vagons from the HUD radar, erases the dynamic-collision records, and |
| 0x004de460 | `railway_on_collision` | `Collision__9C_railwayFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001eb5e0 | CONFIRMED | resolves a collision with the vehicle |
| 0x004de7c0 | `railway_is_possible_coll` | `IsPossibleColl__9C_railwayFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x001eb970 | CONFIRMED | cheap pre-test for whether an actor could collide with this vehicle |
| 0x004def00 | `railway_fill_track_curve` | `FillCurve__9C_railwayFPQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>` | 0x001ec570 | CONFIRMED | samples the intended path and fills the caller's container with `gcol_CurveSphereStruct` entries — a position plus a time/radius term per sample |
| 0x004df6b0 | `railway_clamp_movement_against_traffic` | `PreduceMovement__9C_railwayFv` | 0x001eda50 | CONFIRMED | builds the swept-sphere curve for the intended motion (`railway_fill_track_curve`), tests the cars and actors around the vehicle against it with `railway_sweep_sphere_along_curve`, and returns the |
| 0x004e0790 | `railway_place_on_track` | `Place__9C_railwayFi` | 0x001ee780 | CONFIRMED | checks the semaphore state, rebuilds the forward and back node vectors, then for the engine and each vagon calls `railway_find_back_pos` and `railway_spring_process` to settle the body on the rails |
| 0x004e1070 | `railway_collide` | `Collide__9C_railwayFv` | 0x001ef190 | CONFIRMED | runs the vehicle's collision resolution for the current position, always immediately after `railway_place_on_track` |
| 0x004e1d90 | `railway_hit` | `Hit__9C_railwayF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001ef760 | CONFIRMED | breaks up to three windows, kills up to three passengers and panics the rest (§5) |
| 0x004e2710 | `railway_get_safe_speed` | `GetSafeSpeed__9C_railwayFRf` | 0x001f01b0 | CONFIRMED | looks ahead along the track for stops, semaphores, station limits and the vehicle ahead, and returns the permitted speed while reporting the limiting distance |
| 0x004e3b30 | `railway_add_person` | `AddPerson__9C_railwayFP9I3D_modelii` | 0x001f1790 | CONFIRMED | takes a pedestrian model, builds an `S_rail_human`, reserves a seat through the seat bank and links the record into the passenger list |
| 0x004e3f10 | `railway_remove_person` | `RemovePerson__9C_railwayFP12S_rail_humanUi` | 0x001f1ab0 | CONFIRMED | finds the passenger in the list at `+0x254`/`+0x258`, shifts the tail down, calls `rail_passenger_done` to release its animation machine and collision record, frees it and returns its slot to the |
| 0x004e3fb0 | `railway_get_nearest_vagon` | `GetNearestVagon__9C_railwayFP8S_vector` | 0x001f1d40 | CONFIRMED | compares the squared distance from the point to the engine against each vagon in the `+0xbc` array and returns the winner's index, 0 meaning the engine itself |
| 0x004e4050 | `railway_enumerate_scene_frames` | `Enumerate__9C_railwayFv` | 0x001f1f50 | CONFIRMED | runs the scene frame enumerator several times with different type filters: the train model's frames go to `railway_cb_enum_dummies` (filter 0x21), each wagon's frames (the loop over `railway+0xc4`, |
| 0x004e4c50 | `railway_cb_enum_dummies` | `cbEnumDummies__FP9I3D_frameUi` | 0x001f28e0 | CONFIRMED | reads the frame's name (frame+0xf0, an empty string when null) and tests it in one arm per railway frame tag: an exact compare with `RAIL_COLL`, then `[DEFORM GLASS]`, then prefix tests against |
| 0x004e57e0 | `railway_cb_enum_dummies_vagons` | `cbEnumDummiesVagons__FP9I3D_frameUi` | 0x001f2ed0 | CONFIRMED | compares the frame's name (frame+0xf0, an empty string when null) with the tag `s_frame_tag_deform_glass` ("[DEFORM GLASS]") and, on a match, appends a 0x10-byte `raildummy` record for that frame to |
| 0x004e5a20 | `railway_cb_enum_lights` | `cbEnumLights__FP9I3D_frameUi` | 0x001f2f70 | CONFIRMED | compares each frame's name against the lamp tags (`RAIL_LIGHT_ON` and its siblings) and switches the frame on or off according to the mission's day/night byte |
| 0x004e5f10 | `railway_cb_enum_destroy` | `cbEnumDestroy__FP9I3D_frameUi` | 0x001f3070 | CONFIRMED | calls the frame's virtual slot 0 (Release) and returns 0 so the enumeration continues |
| 0x004e5fa0 | `railway_destroy_entries` | `DestroyEntries__9C_railwayFv` | 0x001f31a0 | CONFIRMED | releases the arrays at `+0x270` and `+0x274` |
| 0x004e5fd0 | `railway_gen_entries` | `GenEntries__9C_railwayFv` | 0x001f3210 | CONFIRMED | allocates `(vagon count + 1) * entry count` zeroed 0x38-byte records into the inner bank at `+0x270` and the same again into the outer bank at `+0x274` |
| 0x004e6350 | `railway_add_outer_entries` | `AddOuterEntries__9C_railwayFv` | 0x001f35e0 | CONFIRMED | walks the `(vagons + 1) * entries` records of the outer bank at `+0x274` and adds each to the global entry registry |
| 0x004e63b0 | `railway_add_inner_entries` | `AddInnerEntries__9C_railwayFv` | 0x001f3660 | CONFIRMED | walks the `(vagons + 1) * entries` records of the inner bank at `+0x270` and adds each to the global entry registry |
| 0x004e6410 | `railway_remove_outer_entries` | `RemoveOuterEntries__9C_railwayFv` | 0x001f36e0 | CONFIRMED | walks the `(vagons + 1) * entries` records of the outer bank at `+0x274` and removes each from the global entry registry |
| 0x004e6470 | `railway_remove_inner_entries` | `RemoveInnerEntries__9C_railwayFv` | 0x001f3760 | CONFIRMED | walks the `(vagons + 1) * entries` records of the inner bank at `+0x270` and removes each from the global entry registry |
| 0x004e64d0 | `railway_get_outer_entry_for_actor` | `GetOuterEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi` | 0x001f37e0 | CONFIRMED | resolves the nearest street-side entry to the actor, refreshing each candidate vagon frame's world matrix, and reports position, direction and index |
| 0x004e69d0 | `railway_get_outer_entry_at` | `GetOuterEntry__9C_railwayFUiUiP8S_vectorP8S_vector` | 0x001f3d90 | CONFIRMED | reads one source entry from `+0x1e4`, stride `0x18` |
| 0x004e6d00 | `railway_get_inner_entry_for_actor` | `GetInnerEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi` | 0x001f41a0 | CONFIRMED | interior boarding entry for a specific actor |
| 0x004e6f80 | `railway_get_entry_for_actor` | `GetEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi` | 0x001f4470 | CONFIRMED | when the actor's kind is 2, 3 or 0x1b AND it is riding a vehicle (`actor+0x98`), routes to the interior entry; otherwise to the street-side entry |
| 0x004e6fc0 | `railway_get_outer_entry_near` | `GetOuterEntry__9C_railwayFRC8S_vectorP8S_vectorP8S_vectorPUi` | 0x001f44e0 | CONFIRMED | the nearest published street-side entry to a point |
| 0x004e7450 | `railway_get_inner_entry_near` | `GetInnerEntry__9C_railwayFRC8S_vectorP8S_vectorP8S_vectorPUi` | 0x001f4ab0 | CONFIRMED | the nearest published interior entry to a point |
| 0x004e7650 | `railway_export_entries` | `ExportEntries__9C_railwayFv` | 0x001f4d00 | CONFIRMED | recomputes every entry's world position and hands the set to the systems that let pedestrians walk to a door |
| 0x004e7fe0 | `railway_sweep_sphere_along_curve` | `PreduceMovementCar__9C_railwayFP5C_carbRQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>` | 0x001ed3b0 | CONFIRMED | walks the curve's 0x10-byte entries (position plus a time/radius term), solves the swept-sphere quadratic against each segment and keeps the smallest non-negative root |
| 0x004e8250 | `railway_is_in_station` | `isInStation__9C_railwayFv` | 0x001f5b60 | CONFIRMED | returns true for states 1 through 4 |
| 0x004e8270 | `railway_is_in_stop` | `isInStop__9C_railwayFv` | 0x001f5bb0 | CONFIRMED | > bool — |
| 0x004e82d0 | `railway_get_vagon` | `GetVagon__9C_railwayFUi` | 0x001f5c20 | CONFIRMED | index 0 returns the actor's own frame at `+0x68`; an index past the vagon count at `+0x1a8` returns null; otherwise indexes the vagon array at `+0xc4` with `[i-1]` |
| 0x004e8300 | `railway_save_game_get_size` | `SaveGameGetSize__9C_railwayFv` | 0x001f5d10 | CONFIRMED | returns the base actor size plus 0x50 |
| 0x004e8310 | `railway_save_game_save` | `SaveGameSave__9C_railwayFPPv` | 0x001f5d60 | CONFIRMED | chains `actor_save_game_save`, then writes 14 fields with `entity_mem_write` |
| 0x004e8440 | `railway_save_game_load` | `SaveGameLoad__9C_railwayFPPv` | 0x001f5f10 | CONFIRMED | chains `actor_save_game_load`, reads 14 fields with `entity_mem_read`, normalises and rebuilds the rotation, resolves the nearest web node, then rebuilds the node vectors and re- places the vehicle |
| 0x004e8be0 | `railway_unlock` | `Unlock__9C_railwayFv` | 0x001f6630 | CONFIRMED | lowers the boarding lock |
| 0x004e8c00 | `railway_is_on_schedule` | `IsOnSchedule__9C_railwayFP10C_web_node` | 0x001f6650 | CONFIRMED | searches the global schedule log for a {node, last-time} pair matching the node; when found, returns 0 unless more than 60000 ms have passed, in which case it restamps the entry and returns 1 |
| 0x004e8e50 | `railway_do_panic` | `DoPanic__9C_railwayFUiUi` | 0x001f6780 | CONFIRMED | walks the passenger list at `+0x254`/`+0x258`, skips the record embedded at `this+0x2b0` (the driver), and for each passenger whose carriage index `+0x18` matches the second argument calls `rail_passenger_do_panic` with the first, the panic duration |
| 0x004e8ee0 | `railway_set_state` | `SetRLWState__9C_railwayFUi` | 0x001f68f0 | CONFIRMED | when the new state is 0 and the model at `+0x28c` exists, switches it off; then writes the state to `+0xdc` |
| 0x004e8f10 | `railway_update_curve_squeal_sound` | `DoBrousicka__9C_railwayFv` | 0x001f6940 | CONFIRMED | requires at least two nodes in the back vector and one in the forward vector |
| 0x004e92b0 | `railway_on_act_state_changed` | `ChangeState__9C_railwayFv` | 0x001f6c80 | CONFIRMED | when the act state at `+0x1c` is 2: releases the three sound objects at `+0x28c`, `+0x290` and `+0x294`, then the object at `+0x298`, removes the vehicle and every vagon from the HUD radar, and |
| 0x004e93d0 | `railway_get_bound_points_2d` | `GetBoundPoints2D__9C_railwayFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x001f6e60 | CONFIRMED | refreshes each vagon frame's world matrix and hierarchical bound volume, then pushes the projected corner points into the caller's vector |
| 0x004e99e0 | `railway_find_vehicle_ahead` | - | - | - | walks the rail graph forward from this vehicle's current node, following the first link whose flag word lacks bit 0x80, and at each step scans the global railway list for a vehicle standing on that |
| 0x004e9b60 | `railway_hold_behind_vehicle_ahead` | - | - | - | saves the current state and forces state `0xc` when the vehicle ahead is within 40 units (§5) |
| 0x004e9bf0 | `railway_set_rain_dummy` | `SetRainDummy__9C_railwayFbP9I3D_frame` | 0x001f7230 | CONFIRMED | with the flag clear it detaches the shared dummy object `g_rail_rain_dummy` and removes it from the mission's list; with the flag set it queries the mission, copies an 8-dword bound volume into the |
| 0x004e9cd0 | `railway_get_actor_speed` | `GetActorSpeed__9C_railwayFR8S_vector` | 0x001f7bc0 | CONFIRMED | reads the vehicle frame at `+0x68`; when the frame's `+0xac` bit 0x10 is clear it normalises the quaternion and refreshes the rotation matrix first, then takes the frame's forward vector at |

#### `S_RailSeats`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004d35f0 | `rail_seats_find_free_seat` | `FindFreeSeat__11S_RailSeatsFUiP8S_vectorf` | 0x001dfb50 | CONFIRMED | walks the rows and the seats within each row starting from a shared rotating cursor so successive calls spread out, requires occupancy 0 and a matching carriage, and when a reference point is |
| 0x004d3740 | `rail_seats_count_free` | `NumFreeSeats__11S_RailSeatsFi` | 0x001dfd40 | CONFIRMED | walks every row and every seat and counts those whose occupancy is 0 and whose row carriage index matches, `-1` meaning any carriage |
| 0x004d3780 | `rail_seats_get_seat_pos` | `GetSeatPos__11S_RailSeatsFUi` | 0x001dfde0 | CONFIRMED | locates the row holding the seat index, then interpolates linearly between the row's start point at `+0x00` and its end point at `+0x0c` by the seat's offset within the row; zeroes the output when no |
| 0x004d39a0 | `rail_seats_get_seat_dir` | `GetSeatDir__11S_RailSeatsFUi` | 0x001e0030 | CONFIRMED | locates the row holding the seat index and returns the row's stored facing vector at `+0x18`; zeroes the output when no row matches |
| 0x004d3a10 | `rail_seats_empty_seat` | `EmptySeat__11S_RailSeatsFUiPv` | 0x001e00b0 | CONFIRMED | locates the row holding the seat index and, when the owner matches, clears both the occupancy and the owner entries |
| 0x004d3aa0 | `rail_seats_reserve_seat` | `ReserveSeat__11S_RailSeatsFUiPv` | 0x001e0160 | CONFIRMED | locates the row holding the seat index and, when its occupancy is 0, sets it to 1 and records the owner |
| 0x004d3b20 | `rail_seats_occupy_seat` | `FullSeat__11S_RailSeatsFUiPv` | 0x001e0220 | CONFIRMED | locates the row holding the seat index; occupancy 0 claims it outright for the owner and sets 2, occupancy 1 upgrades to 2 only when the owner matches, and occupancy 2 just reports whether the owner |
| 0x004d3c10 | `rail_seats_add_row` | `AddRow__11S_RailSeatsFP13rail_seat_row` | 0x001e0380 | CONFIRMED | grows the row array by one, copies the existing rows and stores the new 0x38-byte row |
| 0x004d3c90 | `rail_seats_duplicate_rows` | `Duplicate__11S_RailSeatsFUi` | 0x001e0460 | CONFIRMED | allocates `rows * n` records, copies the existing row block `n` times, frees the old array and updates the count |
| 0x004d3d50 | `rail_seats_init` | `Init__11S_RailSeatsFPv` | 0x001e0590 | CONFIRMED | reads the seat layout out of the vehicle description and builds one row per bench, each with its start and end point, its base seat index, its seat count, its carriage index (row index divided by the |
| 0x004d41c0 | `rail_seats_destroy` | `Destroy__11S_RailSeatsFv` | 0x001e0ba0 | CONFIRMED | for each row releases the occupancy array at `+0x30` and the owner array at `+0x34`, then frees the row array itself |

#### `S_rail_human`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004d4240 | `rail_passenger_rotate_to` | `RotateTo__12S_rail_humanFR8S_vectorUi` | 0x001e0c70 | CONFIRMED | turns the passenger towards a heading; stores `requested_anim` |
| 0x004d44b0 | `rail_passenger_move_to` | `MoveTo__12S_rail_humanFR8S_vectorUi` | 0x001e0e30 | CONFIRMED | walks the passenger towards a point; stores `requested_anim` |
| 0x004d50d0 | `rail_passenger_set_anim` | `SetAnim__12S_rail_humanFUi` | 0x001e1dc0 | CONFIRMED | when the requested clip at `+0x14` differs from the applied clip at `+0x10`, selects it from the passenger anim table and starts it; then ticks the machine, updates the frame and records the applied |
| 0x004d5160 | `rail_passenger_kill` | `Kill__12S_rail_humanFP9C_railway` | 0x001e1ee0 | CONFIRMED | unwinds a panic state back to the saved one, then by current state either releases a boarding lock or not, seeds a randomised death-animation id, sets state 0x11 and re-drives the animation |
| 0x004d5200 | `rail_passenger_create_coll` | `CreateColl__12S_rail_humanFP9C_railway` | 0x001e1fb0 | CONFIRMED | allocates a 0x48-byte record, sets its type to 0x40, seeds it from the passenger frame's bound volume and transform, links the owning vehicle and inserts it into the collision system |
| 0x004d5360 | `rail_passenger_update_coll` | `UpdateColl__12S_rail_humanFv` | 0x001e22a0 | CONFIRMED | when a collision record exists, refreshes the frame's world matrix, copies the 16-dword transform into `+0x4c`, inverts it and pushes the record to the collision system |
| 0x004d5430 | `rail_passenger_done` | `Done__12S_rail_humanFv` | 0x001e2390 | CONFIRMED | destroys the animation machine at `+0x8` and erases the dynamic-collision record at `+0x8c`, clearing both pointers |
| 0x004d5480 | `rail_passenger_do_panic` | `DoPanic__12S_rail_humanFUi` | 0x001e23e0 | CONFIRMED | states 6-8 and 10-15 are saved to `+0x44` and replaced with panic state 0x16; state 9 becomes 0x15 |
| 0x004d71e0 | `rail_passenger_is_woman` | `IsWoman__12S_rail_humanFv` | 0x001e3a80 | CONFIRMED | resolves the frame's `back1` child and reports whether the model name's first letter is `'B'` |

#### `rail_doors`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e7a50 | `rail_doors_open` | `Open__10rail_doorsFv` | 0x001f52c0 | CONFIRMED | sets the door state |
| 0x004e7a60 | `rail_doors_close` | `Close__10rail_doorsFv` | 0x001f52e0 | CONFIRMED | sets the door state |
| 0x004e7a70 | `rail_doors_place` | `Place__10rail_doorsFv` | 0x001f5300 | CONFIRMED | builds one quaternion per bank and stamps it onto every leaf frame (§6c) |
| 0x004e7b50 | `rail_doors_tick` | `Tick__10rail_doorsFi` | 0x001f5450 | CONFIRMED | state 2 (closing) decreases the angle by `dt * 0.001 * speed` and settles at 0 in state 0; state 3 (opening) increases it and settles at pi/2 in state 1 |
| 0x004e7bc0 | `rail_doors_add_leaf_frame` | `AddFrame__10rail_doorsFP9I3D_frameb` | 0x001f5510 | CONFIRMED | grows the selected bank's pointer array by one and stores the frame; the boolean picks bank 0 or bank 1 |
| 0x004e7c90 | `rail_doors_init` | `Init__10rail_doorsFv` | 0x001f5610 | CONFIRMED | clears both leaf banks and their counts, sets the current angle to 0, the opening speed at `+0x44` to 1.0 and the open-angle limit at `+0x48` to pi/2 |
| 0x004e7ce0 | `rail_doors_destroy` | `Destroy__10rail_doorsFv` | 0x001f5660 | CONFIRMED | releases the arrays at `+0x0` and `+0x4` |

#### `rail_glass`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e7d10 | `rail_glass_game_init` | `GameInit__10rail_glassFv` | 0x001f56b0 | CONFIRMED | takes the source material from the vehicle frame's `+0x1e0`, creates a runtime copy through `g_i3d_driver`, stores the pair at `+0xc`/`+0x10` and registers it with the engine |
| 0x004e7d90 | `rail_glass_game_done` | `GameDone__10rail_glassFv` | 0x001f5780 | CONFIRMED | calls `rail_window_restore` for each of the `+0x4` windows, frees the array at `+0x8`, clears the count, and releases the cloned material at `+0x10` |
| 0x004e7df0 | `rail_glass_add_window` | `AddWindow__10rail_glassFP11rail_window` | 0x001f57e0 | CONFIRMED | grows the 0x14-stride array by one, seeding the new record `{1, 0, -1, 0, 0}`, copies the existing records across and frees the old block |

#### `rail_window`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e7e90 | `rail_window_restore` | `Restore__11rail_windowFP12I3D_material` | 0x001f5960 | CONFIRMED | sets the record's state back to 1 and, when a cloned material exists, unbinds it, restores the original and releases the clone |
| 0x004e7f00 | `rail_window_hit` | `Hit__11rail_windowFRC8S_vectorP12I3D_materialP12I3D_material` | 0x001f5a00 | CONFIRMED | in state 1 it clones the window's material so the crack texture can be swapped in, advances to state 2 and re-binds the frame |

#### `rail_entries`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e5f20 | `rail_entries_add_entry` | `AddEntry__12rail_entriesFP10rail_entry` | 0x001f30a0 | CONFIRMED | grows the 0x18-stride array by one, copies the existing entries and stores the new record |
| 0x004e5f90 | `rail_entries_destroy` | `Destroy__12rail_entriesFv` | 0x001f3170 | CONFIRMED | releases the array at `+0x4` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004e9e90 | `vec_count_stride_0x24` | - | - | - | returns 0 when begin is null, else `(end - begin) / 0x24` |
| 0x004e9ec0 | `vec_insert_stride_0x10` | - | - | - | reallocates when the capacity at `+0xc` is exhausted, copies the existing records and appends the new one |
| 0x004ea0a0 | `vec_erase_tail_stride_0x24` | - | - | - | shifts 0x24-byte records down over the erased span and truncates the vector's end pointer at `+0x8` |
| 0x004ea0f0 | `stl_fill_n_stride_0x24` | - | - | - | copies 9 dwords from the template into each of `param_2` records |
| 0x004ea120 | `stl_fill_n_dwords` | - | - | - | std::fill_n<int*> for trivially-copyable |
| 0x004ea150 | `stl_copy_stride_0x24` | - | - | - | copies 9 dwords per record from `[param_1, param_2)` to `param_3` |
| 0x004ea180 | `stl_copy_construct_4dwords` | - | - | - | copies four dwords from the source to the destination, skipping a null destination |

**The frame-enumeration callbacks.** Four free functions are handed to the scene's frame
enumerator (vtable `+0x34`). `railway_enumerate_scene_frames` passes two, both with filter
`0x21`: `railway_cb_enum_dummies` sorts the train model's named frames (`RAIL_COLL`,
`[DEFORM GLASS]`, and the numbered `RAIL_ENTRY` / `RAIL_DOOR` / `RAIL_ROW` tags) into a
`raildummy` vector, and `railway_cb_enum_dummies_vagons` collects each vagon's `[DEFORM GLASS]`
frames (the loop over `+0xc4`). `railway_game_init` passes `railway_cb_enum_lights` (filter
`0xffff`) and `railway_game_done` passes `railway_cb_enum_destroy`. `railway_cb_enum_dummies`
(`0x004e4c50`, 16 name arms) is reached only through that address, so it has no direct caller.
Four of its `copy_range_stride_0x10` calls are dead code: each sits behind `mov ecx,edi; sub ecx,edi;
cmp ecx,1; jae`, which subtracts a register from itself, so the "shift the tail" branch of the
inlined `vector::insert(end(), 1, x)` never runs.

## 4. What the class is

A rail vehicle is an engine plus *n* trailing vagons, running a route of `C_web_node`s, carrying passengers who walk to seats, with doors that open at stops. §1a is the map from each concern to its fields.

## 5. Behaviour worth knowing

* **Departures are rate-limited to one per 60 000 ms per stop.** `railway_is_on_schedule`
  keeps a file-scope `{node, last-departure}` log shared by every vehicle.
* **A tram holds back when the vehicle ahead is within 40 units.**
  `railway_find_vehicle_ahead` walks the rail graph forward and records the next vehicle
  at `+0x160`; `railway_hold_behind_vehicle_ahead` saves the current state and forces
  state `0xc`.
* **The flange squeals on a bend.** `railway_update_curve_squeal_sound` turns on sound
  slot 3 when the track bends between ~10° and ~80° within 5 units of the next node, and
  scales it by speed.
* **It does not rain inside a tram.** `railway_set_rain_dummy` moves the shared
  `g_rail_rain_dummy` volume into and out of the scene's rain-exclusion list.
* **Shooting a tram breaks its windows and kills passengers.** `railway_hit` breaks up to
  three windows, kills up to three passengers and panics the rest.
* **Everything stops for a cutscene.** `railway_ai` returns immediately while
  `g_ambient_ai_suspended` is set.

### 5a. The globals

All four file-scope vectors have an MSVC static-init/atexit group in
`[0x004d7300, 0x004d74e0)`; three carry a bit in `g_rail_static_teardown_flags`.

| global | what it is |
|---|---|
| `g_railway_list_*` | every live rail vehicle |
| `g_rail_semaphore_list_*` | the rail signals |
| `g_rail_schedule_log_*` | `{node, last-departure-ms}` pairs |
| `g_rail_clamp_candidates_*` | never filled: the file holds 11 references to its four addresses — the zero-init, the atexit free and the one reader, `railway_clamp_movement_against_traffic` — so as shipped that loop never runs |
| `g_rail_rain_dummy` | the shared rain-exclusion volume |
| `g_rail_seat_search_cursor` | the rotating seat-search cursor |
| `g_rail_passenger_anims` | the passenger animation table, 12 bytes per state |

### 5b. The three sound slots

`railway_game_init` builds sound object *i* at `+0x28c + 4i` from the `sounds.dat` id at
`+0x29c + 4i`; the ids depend on the elevated-train flag `+0x1d8`:

| slot | object | tram | elevated train | played by |
|---|---|---|---|---|
| 1 | `+0x28c` | 158 `salina_jizda` (`tram1.wav`) | 157 `nadzemka_jizda` (`railway.wav`) | the running loop: `railway_ai`, `railway_update`, `railway_set_state` |
| 2 | `+0x290` | 160 `salina_cink` (`tram2.1.wav`, `tram2.2.wav`) | 291 `nadzemka_dvere (open)` (`t-door.wav`) | `railway_ai` rebuilds and starts it in its door and departure states |
| 3 | `+0x294` | 161 `salina_zatacka` (`tram3.1.wav` .. `tram3.3.wav`) | 161, the same | `railway_update_curve_squeal_sound` |

## 6. The helper types

### 6a. `S_RailSeats` — the seat banks (`rail_seats_`)

A bank is an array of **rows**, stride `0x38`. A row is a *bench*: it stores a start point
at `+0x00` and an end point at `+0x0c`, and seat positions are **linearly interpolated
between them** by the seat's offset within the row (`rail_seats_get_seat_pos`). The row
also carries a facing vector at `+0x18`, its base seat index at `+0x24`, its seat count at
`+0x28`, its carriage index at `+0x2c`, and two parallel arrays: occupancy at `+0x30` and owner at
`+0x34`.

Occupancy is a three-state value, which is what the three mutators are for:

| value | meaning | set by |
|---|---|---|
| 0 | free | `rail_seats_empty_seat` |
| 1 | reserved — a passenger is walking to it | `rail_seats_reserve_seat` |
| 2 | seated | `rail_seats_occupy_seat` |

🔑 `rail_seats_find_free_seat` scans rows and seats from a **shared rotating cursor**
(`g_rail_seat_search_cursor`), incremented on every successful pick. That is why boarding
passengers spread through a carriage instead of all choosing the same bench. It returns
`0xffff` when nothing is free.

`rail_seats_duplicate_rows` replicates the row block once per carriage (the engine plus each
vagon), so every carriage shares one layout, and `rail_seats_init` then stamps each copy's `+0x2c`
with its carriage index: the row index divided by the rows per carriage, 0 being the engine.
`rail_seats_find_free_seat` matches the index exactly; `rail_seats_count_free` takes `-1` for any
carriage.

### 6b. `S_rail_human` — a passenger (`rail_passenger_`)

State values of `+0x00` that matter: **`0x11`** dead, **`0x12`** inactive or alighted (both excluded by
`railway_any_passenger_wants_off`), **`0x15`** and **`0x16`** the two panic states; `rail_passenger_do_panic` saves the state and the
applied clip first, in `+0x44` and `+0x48`, zeroes the elapsed time `+0x3c` and stores the duration `+0x40`; `railway_traffic_ai` adds the frame time to `+0x3c` and ends the panic when it passes `+0x40`. A passenger alights at the stop where `stops_left` (`+0x0c`) reaches 0: `railway_ai` decrements it for every passenger at each arrival.

Passengers are solid — `rail_passenger_create_coll` publishes a `0x48`-byte
dynamic-collision record and `rail_passenger_update_coll` keeps it in step — so you can
bump into somebody standing in a tram.

`rail_passenger_is_woman` resolves the frame's **`back1`** child and reports whether the
model name's first letter is `'B'`. (The ambient pedestrian crowd reads a height scale off
the same child frame.)

### 6c. `rail_doors` — the doors (`rail_doors_`)

Two leaf banks (one per side) sharing a single angle. States: **0** closed, **1** open,
**2** closing, **3** opening. `rail_doors_tick` integrates the angle at
`dt * 0.001 * speed` and clamps at **π/2**; `rail_doors_place` builds one quaternion per
bank (`leaf_rot_a` from `-angle`, `leaf_rot_b` from `+angle`, so the two banks swing in opposite directions) and stamps it onto every leaf frame with the flag rewrite
`(flags & 0xfffffecb) | 0x40000008`. `open_angle` (`+0x48`, π/2) is stored by `rail_doors_init` and read by nothing; the tick clamps against the literal.

### 6d. `rail_glass` / `rail_window` — breakable windows

A window is `0x14` bytes seeded `{1, 0, -1, 0, 0}`. `rail_glass_game_init` clones the
source material per vehicle so one broken window does not crack every tram in the city.
**Two hits break a window**: `rail_window_hit` state 1 swaps in the cracked material,
state 2 calls the glass-breaking helper with the sound literal `lustr1.wav`. The window's `+0x0c` holds the clone bound while it is cracked, `+0x10` the original; `+0x08` is its carriage index, which `railway_hit` and `railway_on_collision` match against the hit carriage.

### 6e. `rail_entries` — the boarding points

Entries are `0x18`-byte records (a start point, an end point and flags). There are two
banks, **inner** (`+0x270`) and **outer** (`+0x274`), each holding
`(vagon count + 1) × entry count` published records — `railway_gen_entries` allocates both.

The public accessor is `railway_get_entry_for_actor`: an actor of kind 2, 3 or `0x1b`
that is **riding a vehicle** gets an interior entry, everyone else the street side. The
four `railway_add_*_entries` / `railway_remove_*_entries` siblings publish and retract the
banks as passengers board and alight.

## 7. PC vs PS2

**How the twins are established.** The PC callee sequence of `railway_ai` and the PS2 callee
sequence of `AI__9C_railwayFUi` run in step for **35 consecutive calls with zero
inversions**. That single measurement pinned `Place(i)`, `Collide`, `GetSafeSpeed`,
`DoBrousicka`, `MustStop`, `SetRLWState`, `PreduceMovement`, `ExportEntries`, the four
entry-bank siblings and both door verbs, and it positively proved five inlinings — a
statement-identical caller acting as a callee oracle at a scale no order alignment could
reach. `railway_place_on_track` then repeated it one level down: its
five-call tail is `FindBackPos, SpringProcess, FindBackPos, FindBackPos, SpringProcess`,
matching PS2 `Place(i)` exactly.

**Method accounting.** 76/76 `C_railway` PS2 methods accounted, plus `S_RailSeats` 11/11,
`S_rail_human` 10/10, `rail_doors` 7/7, `rail_glass` 3/4, `rail_window` 2/3, `rail_entries` 2/2.
The per-function PC↔PS2 twins are in the function registry, `Game-ghidra.exe.functions.md`.
What is *not* a simple 1:1 is listed here:

| PS2 method | status |
|---|---|
| `CheckDistance`, `GetCheckpoints`, `IsStopPoint`, `InRangeActivate`, `OutOfRangeDeactivate` | **inlined on PC** — proven positively, each is the only call missing at its mirrored position. Marked with `// ===== INLINED ON PC:` comments at those sites in the decomp |
| `PreduceMovementCar` | **partly inlined**: only its swept-sphere kernel stayed out of line, as `railway_sweep_sphere_along_curve`. One source function, two PC symbols ⇒ STRUCTURAL |
| `RestoreWindows` | **not located** (Open item 1) |
| `Place(int, C_web_node*)`, `GeneratePersonTrafic`, `FreePersonTrafic`, `ReGenerate` | **no PC counterpart**: on the PS2 they are reached only from `C_rail_generator` ¹ |
| `MemWrite`, `MemRead` | **present on PC** as the shared `entity_mem_write` / `entity_mem_read`, called ×14 each from the SaveGame pair on both builds |

¹ `C_rail_generator::AI` calls `ReGenerate`, which calls `Place(node)` and `GeneratePersonTrafic`;
`C_rail_generator::GetFreeRailHuman` calls `FreePersonTrafic`. None of the four is virtual, and
`C_rail_generator` (entity kind `0x26`) exists only on the PS2 (`structs/entity_kind.md` §4), so the
band has no caller a PC function could mirror.

**Field deltas measured** (do NOT extrapolate — this class is not a uniform shift):

| PC | PS2 | field |
|---|---|---|
| `+0x68` | `+0x70` | the actor's frame |
| `+0xc4` | `+0x148` | vagon array |
| `+0xd4` | `+0x340` | speed |
| `+0xdc` | `+0x310` | run state |
| `+0x1a8` | `+0x334` | vagon count |
| `+0x1b4` | `+0x34c` | speed scale |
| `+0x1d8` | `+0x368` | the stop-test flag |
| `+0x254`/`+0x258` | `+0x2ec`/`+0x2e8` | passenger list (PS2 stores a count, the PC an end pointer) |
| `+0x288` | `+0x31c` | boarding lock |
| `+0x28c` | `+0x16c` | first sound object |
| `+0x298` | `+0x178` | `unread_298`, the empty fourth object slot |
| `+0x2ac` | `+0x330` | material id |
| `+0x2b0` | `+0x210` | the embedded driver |

`unread_298`: `GameDone__9C_railwayFv` and `ChangeState__9C_railwayFv` release the PS2 `+0x178` with the same three vtable calls (`+0x40`, `+0x38`, `+0x08`) as the three sound objects before it and clear it, and no other PS2 `C_railway` body touches `+0x178`, so the slot is never filled on either build.

`S_RailSeats` row stride is **0x38 on PC against 0x50 on PS2**, with the occupancy/owner
arrays at `+0x30`/`+0x34` against `+0x3c`/`+0x40`.

**Static class members.** Two of this class's globals are PS2 static members rather than
instance fields, which is why one object serves every vehicle:
`rain_dummy__9C_railway` (→ `g_rail_rain_dummy`) and **`salina_v_kine__9C_railway`**
(→ `g_ambient_ai_suspended`). The latter is Brno slang for a tram at the cinema,
so the flag's origin is literally "stop the trams, a cutscene is playing".
The PC name is kept because its consumer census is much wider than rail.

**Toolchain traps in this TU.** PS2 `Hit`, `Init__11S_RailSeats` and `GameInit` are
dominated by VU intrinsics and `GetCheckpoints`, so a callee-multiset comparison cannot
work on them by construction; they are graded from vtable slot, signature and
the surviving distinctive callee instead. PS2 `strcpy`×6 in `GameInit` is MSVC `string_*`
on the PC.

### 7a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001e4cf0 | `IsStopPoint__9C_railwayFUi` | inlined: the only call missing at its mirrored position |
| 0x001e7690 | `OutOfRangeDeactivate__9C_railwayFv` | inlined: the only call missing at its mirrored position |
| 0x001e7aa0 | `InRangeActivate__9C_railwayFv` | inlined: the only call missing at its mirrored position |
| 0x001ee4c0 | `Place__9C_railwayFiP10C_web_node` | reached only from `C_rail_generator`, which the PC does not have |
| 0x001f1e40 | `CheckDistance__9C_railwayFR8S_vector` | inlined: the only call missing at its mirrored position |
| 0x001f5c70 | `MemWrite__9C_railwayFPPvPvUiUi` | the shared `entity_mem_write`, called 14 times from the save pair |
| 0x001f5cc0 | `MemRead__9C_railwayFPPvPvUiUi` | the shared `entity_mem_read`, called 14 times from the load pair |
| 0x001f7330 | `GeneratePersonTrafic__9C_railwayFi` | reached only from `C_rail_generator` |
| 0x001f7670 | `ReGenerate__9C_railwayFP10C_web_node` | reached only from `C_rail_generator` |
| 0x001f7820 | `FreePersonTrafic__9C_railwayFv` | reached only from `C_rail_generator` |
| 0x001f78f0 | `ChangeSceneInit__9C_railwayFv` | not located: the PC table ends at slot 38, this is the PS2's slot 27, one past the mirrored range (Open item 1) |
| 0x001f79f0 | `ChangeSceneDone__9C_railwayFv` | not located: slot 28, past the PC table's end (Open item 1) |
| 0x001f7b60 | `GetCheckpoints__9C_railwayFb` | inlined: the only call missing at its mirrored position |
| 0x001f58f0 | `RestoreWindows__10rail_glassFv` | not located (Open item 1) |
| 0x001f58c0 | `__ct__11rail_windowFv` | inlined into `rail_glass_add_window`, which writes the seed |

## 8. Related

`structs/entity_kind.md` §1 (kind 8) and §3 (`C_railway` vs `C_wagon`) · `structs/seagull_flock.md` (the unit immediately after this
one) · `structs/anim_machine.md` (`S_rail_human` is one of its six owners) · `structs/railway_switch.md` (the turnout).

## 9. Open items

1. **`ChangeSceneInit` and `ChangeSceneDone`** have no located PC slot. **`RestoreWindows`:** no distinct PC function. Its restore-all loop exists once on the PC, inside `rail_glass_game_done`, so a fold there is the likely explanation; not proven.
2. **In game:** does a tram wait 60 s at a stop, and does the squeal fire on the sharp turns? Shoot a tram window: two hits should break it, and a third shot should kill a passenger.
