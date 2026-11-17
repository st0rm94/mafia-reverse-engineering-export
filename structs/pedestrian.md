# The ambient pedestrian crowd — `C_traffic_element`, `C_traffic_generator`, `C_don_ocas`

**Created** 2026-08-14 17:51 +0200 · **Modified** 2026-11-08 22:29 +0100

The pedestrians of the streets: light records in an arena (`C_traffic_element`), a scene-placed spawner that owns them (`C_traffic_generator`, entity kind `0x12`) and one proxy actor that stands in for the whole crowd towards the engine (`C_don_ocas`, entity kind `0x13`). `C_traffic_generator` generates **only pedestrians**; the ambient **car** traffic is `C_traffic_car` (entity kind `0xc`, `structs/traffic_car.md`). The class globals keep the `g_traffic_*` spelling (§6).

⚠️ `pedestrian_` is a proper prefix of **both** `pedestrian_crowd_` **and** `pedestrian_manager_`, so any name filter on `pedestrian_` picks up all three classes.

| | |
|---|---|
| PC prefix | `pedestrian_crowd_`, `pedestrian_manager_`, `pedestrian_` |
| PC kind (`+0x10`) | `0x12` generator, `0x13` crowd proxy; the element has none |
| PC vtable | `0x0063c778` generator, `0x0063c6d8` crowd proxy, 39 slots each; the element has none |
| PC base class | `C_actor` for the generator and the proxy; the element has none |
| PC instance size | element `0x200`; proxy `0x70`; generator `0xb8` (`operator_new(0xb8)` at the factory) |
| PC constructor | `pedestrian_ctor`, `pedestrian_manager_ctor`, `pedestrian_crowd_ctor` |
| PC translation unit | `[0x004b0d30, 0x004be810)`, 86 starts, all named |
| PS2 class | `C_traffic_element` (tag `17C_traffic_element`), 26 methods; `C_traffic_generator` (`19C_traffic_generator`), 49 methods; `C_don_ocas` (`10C_don_ocas`), 7 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `C_traffic_element`

```c
typedef struct C_traffic_element {    // 0x200 bytes, stride of the generator's arena; no vtable
    /* +0x00 */ uint8_t is_corpse; // [H] 1 once the death clip has ended; walks skip the record
    /* +0x04 */ void    *owner; // [H] the owning generator; NULL for a free person, which is destroyed, not returned
    /* +0x08 */ anim_machine *anim; // [H] the record's C_animations_machine_de_luxe, from g_pedestrian_anim_list
    /* +0x0c */ uint8_t active; // [H] walks gate on it
    /* +0x10 */ float   position[3]; // [H] world position, x y z
    /* +0x1c */ float   direction[3]; // [H] motion direction, x y z
    /* +0x28 */ float   velocity[3]; // [H] look-ahead velocity: the walk direction x 5 x scale while walking
    /* +0x34 */ float   anim_start_position[3]; // [H] the position when the clip began, x y z
    /* +0x40 */ I3D_model *model; // [H] the pedestrian's model
    /* +0x44 */ I3D_frame *dog_head; // [H] a replacement head model on the neck frame, or 0
    /* +0x48 */ float   cull_distance; // [H] 90.0 seeded, then the generator's +0x88
    /* +0x4c */ I3D_frame *umbrella; // [H] the umbrella frame; dropped through a public physics object
    /* +0x50 */ uint8_t in_camera_sector; // [H] the model is in the camera's sector
    /* +0x54 */ float   player_distance_sq; // [H] squared distance to the player
    /* +0x58 */ int32_t spawn_grace_ticks; // [H] while non-zero the far fade-out and the sector cull are skipped
    /* +0x5c */ int32_t out_of_sector_ms; // [H] ms out of the camera's sector
    /* +0x60 */ int32_t target_node; // [H] the web node walked to
    /* +0x64 */ float   target_pos[3]; // [H] the point walked to, x y z
    /* +0x70 */ float   node_radius; // [H] the radius of the node walked to
    /* +0x74 */ float   fall_speed; // [H] the fall speed; -1.0 when not falling
    /* +0x78 */ int32_t ground_miss_count; // [H] missed ground probes in a row; above 3 the fall starts
    /* +0x7c */ int32_t previous_node; // [H] the web node walked from; -1 before the first
    /* +0x80 */ float   path_points[4][3]; // [H] path points, walked from index path_state - 1 down to 0
    /* +0xb0 */ int32_t path_state; // [H] path state and node countdown; below 0 means no path
    /* +0xb4 */ uint8_t on_detour; // [H] the path is an obstacle detour
    /* +0xb8 */ I3D_frame *base_frame; // [H] the model's base sub-frame when it is a mesh or a morph visual
    /* +0xbc */ float   alpha; // [H] fade value handed to pedestrian_set_alpha, 1.0 from the activators
    /* +0xc0 */ float   alpha_rate; // [H] fade change per second
    /* +0xc4 */ tDynamicCollObject collision; // [H] its collision object, type 0x40, owner the crowd proxy
    /* +0x10c */ float   inverse_world[16]; // [H] the collision object's inverse world matrix, pointed at by +0x104
    /* +0x14c */ int32_t collision_insert_delay; // [H] a countdown of pedestrian_set_anim calls; at 1 the object enters the collision list
    /* +0x150 */ uint8_t in_collision_list; // [H] the collision object is inserted
    /* +0x154 */ float   scale; // [H] model scale: 1.0, then the back1 tag's three digits x 0.01
    /* +0x158 */ uint8_t tag_letter_0; // [H] back1 tag first letter minus 'A'; 1 selects the female animations
    /* +0x159 */ uint8_t tag_letter_1; // [H] back1 tag second letter minus 'A'; no reader (.text scan)
    /* +0x15a */ uint8_t category; // [H] back1 tag third letter minus 'A', decoded in §2a-i
    /* +0x15c */ float   health; // [H] -1.0 = dead
    /* +0x160 */ int32_t current_anim; // [H] the clip id now playing
    /* +0x164 */ int32_t target_anim; // [H] the clip id wanted next
    /* +0x168 */ int32_t following_anim; // [H] the clip after target_anim; the second of the mega-panic pair
    /* +0x16c */ uint8_t anim_one_shot; // [H] passed to anim_machine_set_anim; arms anim_end_ms
    /* +0x16d */ uint8_t following_anim_one_shot; // [H] the one-shot flag of following_anim
    /* +0x170 */ int32_t anim_end_ms; // [H] the clip's time minus 250 ms, counted down by the tick
    /* +0x174 */ float   anim_end_point[3]; // [H] the clip's track end point, turned into the world, x y z
    /* +0x180 */ int32_t ground_probe_ms; // [H] ms to the next ground probe
    /* +0x184 */ I3D_pose *neck_pose; // [H] the pose attached to the neck frame
    /* +0x188 */ I3D_frame *look_target; // [H] the frame the head turns to, the player's neck
    /* +0x18c */ uint8_t head_returning; // [H] the head is turning back to straight ahead
    /* +0x190 */ int32_t wait_ms; // [H] while positive the tick counts it down and returns early
    /* +0x194 */ uint8_t side_stepping; // [H] stepping across the segment around a blocking pedestrian
    /* +0x198 */ uint32_t unread_198; // [U] the savegame stream; no reader (.text scan)
    /* +0x19c */ uint8_t panicking; // [H] the pedestrian is fleeing
    /* +0x19d */ uint8_t standing; // [H] standing at a stand node until despawned
    /* +0x19e */ uint8_t panic_latch; // [H] panic happened once
    /* +0x1a0 */ int32_t panic_expiry_ms; // [H] rand + 20000 ms
    /* +0x1a4 */ uint8_t mega_panic_level; // [H] 1 to 3, picks the clip pair
    /* +0x1a5 */ uint8_t promoted; // [H] a real actor took this pedestrian's place
    /* +0x1a8 */ C_car   *assigned_car; // [H] the car it boards
    /* +0x1ac */ C_car   *hit_by_car; // [H] the car that struck it
    /* +0x1b0 */ uint8_t knocked_down; // [H] knocked down by a car; gets up with clip 0x152
    /* +0x1b4 */ int32_t walk_style; // [H] 0 to 4 selects the male walk clip
    /* +0x1b8 */ C_actor *witness_of; // [H] the actor it saw; drives the police report
    /* +0x1bc */ S_using_object usable; // [H] its registered usable object (structs/using_object.md §1a)
    /* +0x1f4 */ uint16_t web_link_mask; // [H] a web link is followed only when its mask ANDs with this
    /* +0x1f6 */ uint8_t voice; // [H] the row of the game voice table
    /* +0x1f8 */ int32_t greet_timer_ms; // [H] ms to the next greeting check
    /* +0x1fc */ uint8_t may_greet; // [H] the pedestrian may greet the player
    /* +0x1fd */ uint8_t recognised_player; // [H] the pedestrian recognised the chased player
    /* +0x1fe */ uint8_t hailing_taxi; // [H] waving at a taxi
    /* +0x1ff */ uint8_t on_sidewalk; // [H] the last ground probe hit the sidewalk
} C_traffic_element;
```

`position`, `direction`, `velocity`, `anim_start_position`, `target_pos` and `anim_end_point` are one vec3 each. `inverse_world` holds sixteen floats because it is a 4 x 4 matrix. `path_points` holds four points because a path is walked from point `path_state - 1` down to point 0 (`+0x80 + (path_state - 1) x 0xc`); `path_state` is never set above 3, so point 3 (`+0xa4`) is only written and saved.

**The web nodes.** `target_node` and `previous_node` index the node table at `*(C_mission +0x2c) + 4`, stride `0x1e`: position vec3 `+0x00`, flag bytes `+0x0c` / `+0x0d`, radius `+0x10` (u16 x 0.01), the link array `+0x18` of 8-byte `{u16 node, u16 mask}` entries and the link count `+0x1d`. A node with `+0x0c` bit `0x1` sets `web_link_mask` to 1; one with `+0x0d` bit `0x2` is a stand node (`standing`) and does not allow side-stepping.

### 1b. `C_traffic_generator`

Inherits `C_actor` (`structs/actor.md` §1) up to `+0x70`.

```c
typedef struct C_traffic_generator {  // own band; the C_actor base ends at +0x70
    /* +0x70 */ uint8_t is_global; // [H] designer, saved, 1 from the ctor; set: a candidate anywhere
    /* +0x74 */ float   area_radius; // [H] designer, saved, 100.0 from the ctor; the activation radius
    /* +0x78 */ uint8_t selected; // [H] the generator chosen this frame; only it spawns
    /* +0x7c */ C_traffic_element *arena; // [H] the pedestrian arena, stride 0x200
    /* +0x80 */ int32_t live_count; // [H] live pedestrians, compared with the budget in pedestrian_manager_ai
    /* +0x84 */ int32_t slot_count; // [H] arena slots, the sum of the per-name spawn counts
    /* +0x88 */ float   cull_distance; // [H] designer, 100.0; handed to each pedestrian's +0x48
    /* +0x8c */ float   spawn_distance_min; // [H] designer, 50.0; 5.0 replaces it on the initial fill
    /* +0x90 */ float   spawn_distance_max; // [H] designer, 90.0
    /* +0x94 */ float   unread_94; // [U] designer, 50.0; no reader (class census)
    /* +0x98 */ char   *model_names; // [H] the model-name array, 0x11 bytes per name
    /* +0x9c */ int32_t model_name_count; // [H] entries of the two arrays
    /* +0xa0 */ int32_t *model_spawn_counts; // [H] the per-name spawn counts
    /* +0xa4 */ int32_t spawn_accumulator_ms; // [H] compared against 0x70 in pedestrian_manager_ai
    /* +0xa8 */ int32_t population_budget; // [H] 10 seeded, scaled by g_traffic_ratio_all
    /* +0xac */ int32_t designer_budget; // [H] saved across the scene, restored by pedestrian_manager_game_done
    /* +0xb0 */ int32_t spawn_cursor; // [H] the round-robin cursor
    /* +0xb4 */ int32_t cop_slot_boundary; // [H] arena index of the last foot-cop slot allowed to spawn, -1 for none
} C_traffic_generator;
```

A savegame record is `0xbc` bytes per live pedestrian plus a 10-byte header (`pedestrian_manager_save_game_get_size`); §7 lays both out.

### 1c. `C_don_ocas`

No own field: `operator_new(0x70)` holds the `C_actor` base alone. `+0x65`, the only byte it writes (`pedestrian_crowd_game_init`), is a base field. One instance, at `C_game+0x2a1c`.

## 2. Field evidence

The cells come from a census of each class's functions, plus the external users that the bodies name. The savegame pair copies the fields of §7 in and out; the cells name it only where it is a field's only access or more than a copy. Accesses inside the class's functions that belong to other objects are listed in the `Owned by another object:` paragraphs after the §2a and §2b tables.

### 2a. `C_traffic_element`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | is_corpse | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (1) | `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_set_anim`, `pedestrian_set_mega_panic`, `pedestrian_tick`, every crowd walk | `pedestrian_tick` stores 1 when the death clip ends with health at or below 0, erasing the collision and removing the shadow; such a record only waits until the player is beyond `cull_distance` and is then deactivated |
| +0x4 | owner | H | `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_set_model` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_deactivate`, `pedestrian_destroy_model`, `pedestrian_spawn_gang_on_hit`, `pedestrian_tick` | `pedestrian_destroy_model` decides by it |
| +0x8 | anim | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_ctor`, `pedestrian_deactivate` | `pedestrian_activate`, `pedestrian_deactivate`, `pedestrian_drop_umbrella`, `pedestrian_end_taxi_wave`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_tick`, `pedestrian_wave_at_taxi` | - |
| +0xc | active | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_deactivate`, `pedestrian_set_model` | `pedestrian_tick` | - |
| +0x10 | position | H | `pedestrian_tick` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_crowd_collision`, `pedestrian_crowd_is_possible_coll`, `pedestrian_greet_player`, `pedestrian_hit`, `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_manager_railway_in_station`, `pedestrian_on_player_talk`, `pedestrian_set_anim`, `pedestrian_set_panic`, `pedestrian_spawn_gang_on_hit`, `pedestrian_tick`, `pedestrian_update_model` | - |
| +0x1c | direction | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_tick`, `pedestrian_wave_at_taxi` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_greet_player`, `pedestrian_hit`, `pedestrian_spawn_gang_on_hit`, `pedestrian_tick`, `pedestrian_update_model`, `pedestrian_wave_at_taxi` | - |
| +0x28 | velocity | H | `pedestrian_tick` (0 each tick; while walking, the walk direction x 5 x `scale`) | `pedestrian_manager_is_possible_coll_pedestrian` | the relative motion in its time-to-collision quadratic, radius 0.6 |
| +0x34 | anim_start_position | H | `pedestrian_set_anim` | `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_set_anim` copies `position` here |
| +0x40 | model | H | `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_destroy_model`, `pedestrian_set_model` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_create_model`, `pedestrian_deactivate`, `pedestrian_destroy_model`, `pedestrian_greet_player`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_set_model`, `pedestrian_spawn_gang_on_hit`, `pedestrian_tick`, `pedestrian_update_model`, `pedestrian_wave_at_taxi` | - |
| +0x44 | dog_head | H | `pedestrian_activate` (0, then for 1 in N civilians a model from `hlavapsa.i3d`), `pedestrian_activate_free` (0), `pedestrian_deactivate` (released, 0) | `pedestrian_activate`, `pedestrian_deactivate`, `pedestrian_tick`, `pedestrian_manager_railway_in_station` | the model is linked to the `neck` frame, the neck scaled to 0.01 and the model to 100; `pedestrian_tick` keeps the neck at 0.01 while it is set, and such a pedestrian never boards a train. The asset name means "dog's head" (`structs/dog.md`) |
| +0x48 | cull_distance | H | `pedestrian_manager_game_init`, `pedestrian_set_model` | `pedestrian_tick`, `pedestrian_wave_at_taxi` | - |
| +0x4c | umbrella | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_deactivate`, `pedestrian_drop_umbrella` | `pedestrian_activate`, `pedestrian_deactivate`, `pedestrian_drop_umbrella` | `pedestrian_drop_umbrella` |
| +0x50 | in_camera_sector | H | `pedestrian_tick` (`C_mission +0x64`, the camera's sector, equal to the model's sector `I3D_frame +0x13c`) | `pedestrian_tick`, `pedestrian_crowd_collision` | out of the sector with the player beyond 20 m, a collision deactivates the pedestrian instead of hitting it |
| +0x54 | player_distance_sq | H | `pedestrian_manager_tick_pedestrians` | `pedestrian_tick`, `pedestrian_wave_at_taxi` | - |
| +0x58 | spawn_grace_ticks | H | `pedestrian_activate`, `pedestrian_activate_free` (2), `pedestrian_tick` (-1 per tick while non-zero) | `pedestrian_tick` | while non-zero the far fade-out and the out-of-sector deactivation are skipped |
| +0x5c | out_of_sector_ms | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (+dt out of the sector, 0 in it) | `pedestrian_tick` | a free person (`owner` 0) past 10000 with no grace left is deactivated |
| +0x60 | target_node | H | `pedestrian_activate`, `pedestrian_activate_free` (`web_path_find_nearest_node`), `pedestrian_advance_to_next_node`, `pedestrian_set_panic`, `pedestrian_tick` (the swaps with `+0x7c`) | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_set_panic`, `pedestrian_tick` | an index into the web-node table (§1a) |
| +0x64 | target_pos | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node` (the node position plus a random lateral offset up to 0.9 x `+0x70`, or a path point), `pedestrian_set_panic`, `pedestrian_tick` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_set_panic`, `pedestrian_tick` (the steering and the reach test), `pedestrian_manager_try_send_pedestrian_to_parked_car` | - |
| +0x70 | node_radius | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_set_panic`, `pedestrian_tick` (the node's `+0x10` x 0.01 at every node change) | `pedestrian_advance_to_next_node` (the span of the lateral offset), `pedestrian_tick` (the reach distance off a path, the side-step bound) | - |
| +0x74 | fall_speed | H | `pedestrian_activate`, `pedestrian_activate_free` (-1.0), `pedestrian_tick` (0.1 after the fourth missed ground probe, 0.5 on surface 31, + dt x 0.001 x 8.9 while falling, -1.0 on landing) | `pedestrian_tick` | above 50 in water the pedestrian is deactivated |
| +0x78 | ground_miss_count | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (+1 per missed ground probe, 0 on a hit or a landing) | `pedestrian_tick` | above 3 the fall starts |
| +0x7c | previous_node | H | `pedestrian_activate`, `pedestrian_activate_free` (the nearest linked node), `pedestrian_advance_to_next_node`, `pedestrian_set_panic`, `pedestrian_tick` (the swaps with `+0x60`) | `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_set_panic`, `pedestrian_tick` | an int index, -1 before the first node |
| +0x80 | path_points | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle` (points 0, 1), `pedestrian_manager_add_free_person_from_car` (points 0, 1), `pedestrian_manager_release_pedestrians_from_car` (point 0 = point 2), `pedestrian_manager_try_send_pedestrian_to_parked_car` (points 0 and 3 by `car_seat_get_attach_positions`, points 1 and 2 the car corners) | `pedestrian_advance_to_next_node` (`+0x80 + (path_state - 1) x 0xc`), `pedestrian_avoid_obstacle`, `pedestrian_manager_release_pedestrians_from_car`, `pedestrian_tick` | four path points (§1a); point 3 is only written and saved |
| +0xb0 | path_state | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_tick` | `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_manager_railway_in_station`, `pedestrian_manager_release_pedestrians_from_car`, `pedestrian_tick` | - |
| +0xb4 | on_detour | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_try_send_pedestrian_to_parked_car`, `pedestrian_tick` (0), `pedestrian_avoid_obstacle` (1) | `pedestrian_advance_to_next_node`, `pedestrian_tick` | at the end of a detour path `pedestrian_advance_to_next_node` returns the pedestrian to node `+0x60` |
| +0xb8 | base_frame | H | `pedestrian_create_model`, `pedestrian_set_model` | `pedestrian_set_alpha` | kept only for a visual (type 1) of visual type `'SNGM'` or `'SMMR'`, which `LS3DF.dll`'s type table names `I3D_VISUAL_SINGLEMESH` and `I3D_VISUAL_SINGLEMORPH`; `pedestrian_set_alpha` calls its vtable `+0x84` |
| +0xbc | alpha | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_tick` | `pedestrian_tick`, `pedestrian_manager_generate_passenger_for_taxi` | - |
| +0xc0 | alpha_rate | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_tick` | `pedestrian_tick` | `pedestrian_tick`: `rate * dt * 0.001 + alpha` |
| +0xc4 | collision | H | `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_set_model`, `pedestrian_tick` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_deactivate`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_set_model`, `pedestrian_tick`, `pedestrian_update_model` | box (-0.5, 0, -0.5)..(0.5, 1.8, 0.5) at `+0xe8` / `+0xf4`; sphere centre `+0xd4` = the frame origin with Y + 0.9; radius `+0xe0` = `scale` x 1.15; `pedestrian_ctor` zeroes all 0x12 dwords |
| +0x10c | inverse_world | H | `pedestrian_create_model`, `pedestrian_set_model`, `pedestrian_update_model` | `pedestrian_create_model`, `pedestrian_set_model`, `pedestrian_tick`, `pedestrian_update_model` | sixteen because it is a 4 x 4 matrix; read by the type-`0x40` query arm |
| +0x14c | collision_insert_delay | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_set_anim`, `pedestrian_tick`, `pedestrian_update_model` | - |
| +0x150 | in_collision_list | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_deactivate`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_hit`, `pedestrian_tick`, `pedestrian_update_model` | `pedestrian_deactivate` clears it after `collision_dyn_erase` |
| +0x154 | scale | H | `pedestrian_create_model`, `pedestrian_set_model` | `pedestrian_create_model`, `pedestrian_hit`, `pedestrian_manager_try_send_pedestrian_to_parked_car`, `pedestrian_set_model`, `pedestrian_tick` | - |
| +0x158 | tag_letter_0 | H | `pedestrian_create_model`, `pedestrian_set_model` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_manager_railway_in_station`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_set_anim`, `pedestrian_spawn_gang_on_hit`, `pedestrian_wave_at_taxi` | `1` (`B`) in every `Czena*` model, `0` in every male one |
| +0x159 | tag_letter_1 | H | `pedestrian_create_model`, `pedestrian_set_model` | - | no reader: a `.text` displacement scan finds the three byte stores and two `C_Vehicle` dword accesses |
| +0x15a | category | H | `pedestrian_create_model`, `pedestrian_set_model`, `police_manager_free_temp_enemy` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_crowd_collision`, `pedestrian_hit`, `pedestrian_manager_ai`, `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_set_panic`, `pedestrian_tick` | `police_manager_free_temp_enemy` stores `1` |
| +0x15c | health | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit` | `pedestrian_crowd_collision`, `pedestrian_crowd_hit`, `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_set_mega_panic`, `pedestrian_tick` | - |
| +0x160 | current_anim | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_set_anim` | `pedestrian_crowd_is_possible_coll`, `pedestrian_hit`, `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_set_anim` compares it with `target_anim` |
| +0x164 | target_anim | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_set_mega_panic`, `pedestrian_tick`, `pedestrian_wave_at_taxi` | `pedestrian_hit`, `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_set_mega_panic` stores the first of its id pair |
| +0x168 | following_anim | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_set_mega_panic`, `pedestrian_tick` | `pedestrian_tick` | - |
| +0x16c | anim_one_shot | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_tick` | `pedestrian_set_anim`, `pedestrian_tick` | - |
| +0x16d | following_anim_one_shot | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_hit`, `pedestrian_set_mega_panic`, `pedestrian_tick` | `pedestrian_tick` | copied into `anim_one_shot` when `following_anim` becomes the target |
| +0x170 | anim_end_ms | H | `pedestrian_set_anim`, `pedestrian_tick` | `pedestrian_tick` | - |
| +0x174 | anim_end_point | H | `pedestrian_set_anim` | `pedestrian_crowd_is_possible_coll`, `pedestrian_set_anim` | - |
| +0x180 | ground_probe_ms | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (-dt; at 0, rand + 100 and a ground probe) | `pedestrian_tick` | - |
| +0x184 | neck_pose | H | `pedestrian_create_model`, `pedestrian_destroy_model`, `pedestrian_set_model` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_destroy_model`, `pedestrian_tick` | - |
| +0x188 | look_target | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_greet_player`, `pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn`, `pedestrian_tick` | `pedestrian_tick` | - |
| +0x18c | head_returning | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` | `pedestrian_tick` | 1 when `look_target` is more than 1.3 rad off, 0 once the head is back within 0.1 rad |
| +0x190 | wait_ms | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_tick` | `pedestrian_tick` | set to 500 + random, 1000, 3000 (the witness path) or 10000 |
| +0x194 | side_stepping | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` | `pedestrian_tick` | 1 when a pedestrian ahead blocks and the node has no `+0x0d` bit `0x2`, 0 beyond the node radius from the segment; meanwhile clip `0xb` moves the pedestrian across the segment |
| +0x198 | unread_198 | U | `pedestrian_manager_save_game_load` | `pedestrian_manager_save_game_save` | only the savegame pair touches it (`.text` scan, indexed accesses included) |
| +0x19c | panicking | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_set_panic`, `pedestrian_tick` | `pedestrian_manager_try_send_pedestrian_to_parked_car`, `pedestrian_tick` | - |
| +0x19d | standing | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (1 at a stand node, generator-owned civilians only) | `pedestrian_tick` | the pedestrian turns to its target, then idles 10 s at a time until it is despawned |
| +0x19e | panic_latch | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_set_panic`, `pedestrian_tick` | `pedestrian_advance_to_next_node`, `pedestrian_greet_player`, `pedestrian_manager_is_car_claimed_by_pedestrian`, `pedestrian_manager_railway_in_station`, `pedestrian_set_panic`, `pedestrian_tick` | - |
| +0x1a0 | panic_expiry_ms | H | `pedestrian_set_panic`, `pedestrian_tick` | `pedestrian_tick` | - |
| +0x1a4 | mega_panic_level | H | `pedestrian_set_mega_panic`, `pedestrian_set_panic` | `pedestrian_hit`, `pedestrian_tick` | - |
| +0x1a5 | promoted | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_spawn_gang_on_hit` | `pedestrian_manager_spawn_cop_from_pedestrian` | - |
| +0x1a8 | assigned_car | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `pedestrian_tick` | `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_manager_is_car_claimed_by_pedestrian`, `pedestrian_manager_release_pedestrians_from_car`, `pedestrian_tick` | - |
| +0x1ac | hit_by_car | H | `pedestrian_hit` (the attacking car, on a collision hit above 16 damage), `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_set_anim` (0 when the collision object re-enters), `pedestrian_tick` (0) | `pedestrian_tick` | within 2 m and below 10 speed it knocks the pedestrian over (clip `0x151`, a run-over for the police manager) |
| +0x1b0 | knocked_down | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (1 when knocked over; 0 once no car is within 4 m after `wait_ms`) | `pedestrian_crowd_is_possible_coll` (skipped while set), `pedestrian_tick` | - |
| +0x1b4 | walk_style | H | `pedestrian_activate`, `pedestrian_activate_free` | `pedestrian_set_anim` | `pedestrian_set_anim`: 0 hands behind, 1 stroll, 2 worker with a bag, 3 hand in pocket, 4 hurry |
| +0x1b8 | witness_of | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_advance_to_next_node` | `pedestrian_advance_to_next_node`, `pedestrian_avoid_obstacle`, `pedestrian_set_panic` | - |
| +0x1bc | usable | H | `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_set_model` | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_create_model`, `pedestrian_ctor`, `pedestrian_deactivate`, `pedestrian_hit`, `pedestrian_set_model` | `pedestrian_set_model` and `_create_model` fill it, `self` `+0x1f0` points back here; `pedestrian_deactivate` calls `using_object_del_object` on it |
| +0x1f4 | web_link_mask | H | `pedestrian_activate` (1), `pedestrian_activate_free` (3 for a free person), `pedestrian_advance_to_next_node` (1 at a node with `+0x0c` bit `0x1`), `pedestrian_manager_save_game_load` (1) | `pedestrian_activate` (the link filter), `pedestrian_activate_free`, `pedestrian_advance_to_next_node`, `web_path_find_nearest_node` (argument) | lies past `usable`, whose `S_using_object` is `0x38` bytes and ends at `+0x1f3` |
| +0x1f6 | voice | H | `pedestrian_activate`, `pedestrian_activate_free` (cops rand + 10, women rand + 29, men rand + 26) | `pedestrian_advance_to_next_node`, `pedestrian_greet_player`, `pedestrian_hit`, `pedestrian_on_player_talk`, `pedestrian_tick` | passed in ECX to `pedestrian_manager_play_pedestrian_scream` (disassembly) |
| +0x1f8 | greet_timer_ms | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_tick` (-dt; at 0, rand + 1500 and a 1-in-N `pedestrian_greet_player`) | `pedestrian_tick` | - |
| +0x1fc | may_greet | H | `pedestrian_activate`, `pedestrian_activate_free` (1 for civilians, 0 for cops), `pedestrian_hit`, `pedestrian_on_player_talk`, `pedestrian_tick` (0) | `pedestrian_greet_player` | `pedestrian_greet_player` requires it set |
| +0x1fd | recognised_player | H | `pedestrian_activate`, `pedestrian_activate_free` (0), `pedestrian_greet_player` (1) | `pedestrian_on_player_talk` (selects the line), `pedestrian_tick` | set when the player is a squad's target or officer, on foot, shares the model's parent frame and the police manager counts no officer on him; cleared only by an activation |
| +0x1fe | hailing_taxi | H | `pedestrian_activate`, `pedestrian_activate_free`, `pedestrian_end_taxi_wave` | `pedestrian_manager_end_all_taxi_waves`, `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_tick` | - |
| +0x1ff | on_sidewalk | H | `pedestrian_tick` (the ground probe: surface id 3, the sidewalk) | `pedestrian_manager_generate_passenger_for_taxi` | only a pedestrian on a sidewalk hails the taxi |

**Owned by another object:** `+0x6f4` -> `structs/human.md` (`spawn_record` of the human that
`pedestrian_wave_at_taxi` spawns).

#### 2a-i. `category`

| Value | Meaning | Evidence |
|---|---|---|
| `0` | civilian | `pedestrian_set_panic`, `pedestrian_activate` |
| `1` | patrol cop (`pol01`-`pol03`) | `pedestrian_manager_spawn_cop_from_pedestrian`; `police_manager_free_temp_enemy` stores it |
| `2` | some `civil*` models; no reader tests it | the tags of the shipped models |
| `3` | mission cop (`pol11`-`pol13`) | `pedestrian_manager_spawn_cop_from_pedestrian` |
| `4` | gang (`FREEgang01`, `FREEgang02`) | `pedestrian_hit`: a hit spawns the gang |

The three tag bytes are written only for a six-character tag, and `pedestrian_create_model` does not clear them first, so a model whose tag fails that test keeps what the record's memory held (`basic/artifacts/game-loop.md` §10).

### 2b. `C_traffic_generator`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | is_global | H | `pedestrian_manager_ctor` (1), `pedestrian_manager_duplicate_from`, `pedestrian_manager_load_data` | `pedestrian_manager_save_data`, `game_init`, `game_tick` | the generator choice of `game_init` / `game_tick` (disassembly `0x005dd879`, `0x005e142e`): set, the generator is a candidate anywhere and the nearest by camera distance wins; clear, it is a candidate only while the camera is inside `area_radius`, and then it wins |
| +0x74 | area_radius | H | `pedestrian_manager_ctor` (100.0), `pedestrian_manager_duplicate_from`, `pedestrian_manager_load_data` | `pedestrian_manager_save_data`, `game_init`, `game_tick` | the activation radius of a generator with `is_global` clear; no spawn function reads it |
| +0x78 | selected | H | `game_init`, `game_tick` (0 on every kind-`0x12` actor, then 1 on the chosen one after `actor_set_act_state(0)`), `pedestrian_manager_game_init` (0), `pedestrian_manager_save_game_load` | `pedestrian_manager_ai`, `pedestrian_manager_save_game_save` | `pedestrian_manager_ai` spawns only while it is set; with it clear and no live pedestrian it calls `actor_set_act_state(1)` |
| +0x7c | arena | H | `pedestrian_manager_ctor`, `pedestrian_manager_game_done`, `pedestrian_manager_game_init` | `pedestrian_manager_ai`, `pedestrian_manager_game_done`, `pedestrian_manager_game_init`, `pedestrian_manager_save_game_load`, `pedestrian_manager_save_game_save` | - |
| +0x80 | live_count | H | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_game_init`, `pedestrian_manager_save_game_load` | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_ai`, `pedestrian_manager_save_game_get_size`, `pedestrian_manager_save_game_save` | - |
| +0x84 | slot_count | H | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_game_done`, `pedestrian_manager_game_init` | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_ai`, `pedestrian_manager_game_done`, `pedestrian_manager_game_init`, `pedestrian_manager_save_game_load`, `pedestrian_manager_save_game_save` | - |
| +0x88 | cull_distance | H | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_ctor`, `pedestrian_manager_duplicate_from` | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | - |
| +0x8c | spawn_distance_min | H | `pedestrian_manager_ctor`, `pedestrian_manager_duplicate_from` | `pedestrian_manager_ai`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | `pedestrian_manager_ai` passes it to `pedestrian_activate`, which places the pedestrian at a random distance between it and `+0x90`; both are scaled at entry by a factor read from `C_game+0x2a0c` |
| +0x90 | spawn_distance_max | H | `pedestrian_manager_ctor`, `pedestrian_manager_duplicate_from` | `pedestrian_manager_ai`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | see `spawn_distance_min` |
| +0x94 | unread_94 | U | `pedestrian_manager_ctor` (50.0), `pedestrian_manager_duplicate_from`, `pedestrian_manager_load_data` | `pedestrian_manager_duplicate_from`, `pedestrian_manager_save_data` | a designer value; no other access (a trace over the kind-`0x12` functions and the `pedestrian_manager_` prefix) |
| +0x98 | model_names | H | `pedestrian_manager_ctor`, `pedestrian_manager_dtor`, `pedestrian_manager_duplicate_from`, `pedestrian_manager_load_data` | `pedestrian_manager_dtor`, `pedestrian_manager_duplicate_from`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | - |
| +0x9c | model_name_count | H | `pedestrian_manager_ctor`, `pedestrian_manager_duplicate_from` | `pedestrian_manager_dtor`, `pedestrian_manager_duplicate_from`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | - |
| +0xa0 | model_spawn_counts | H | `pedestrian_manager_ctor`, `pedestrian_manager_dtor`, `pedestrian_manager_duplicate_from`, `pedestrian_manager_load_data` | `pedestrian_manager_dtor`, `pedestrian_manager_duplicate_from`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | - |
| +0xa4 | spawn_accumulator_ms | H | `pedestrian_manager_ai`, `pedestrian_manager_save_game_load` | `pedestrian_manager_ai`, `pedestrian_manager_save_game_save` | - |
| +0xa8 | population_budget | H | `pedestrian_manager_ctor`, `pedestrian_manager_game_done`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data` | `pedestrian_manager_ai`, `pedestrian_manager_game_init`, `pedestrian_manager_load_data`, `pedestrian_manager_save_data` | - |
| +0xac | designer_budget | H | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_game_init` | `pedestrian_manager_add_free_person_from_car`, `pedestrian_manager_game_done` | - |
| +0xb0 | spawn_cursor | H | `pedestrian_manager_ai`, `pedestrian_manager_game_init` | `pedestrian_manager_ai` | - |
| +0xb4 | cop_slot_boundary | H | `pedestrian_manager_game_init` | `pedestrian_manager_ai` | `pedestrian_manager_game_init` allows round(cop slots x `g_traffic_ratio_police`); `pedestrian_manager_ai` restricts categories 1 and 3 to slots up to this |

**Owned by another object:** `+0x20cc` -> `structs/car.md` (`model_name` of the car
`pedestrian_manager_player_in_taxi` tests).

## 3. Method family

All 82 PS2 methods of the three classes are placed: in the tables below, as `inlined` or `base-stub` rows, or in §8a. A row with no twin is PC-only. A unit helper stays here, the unit's host doc.

### 3a. Construction & destruction

#### file-scope vectors

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004b0d30 | `crt_static_init_004b0d30` | `g_pedestrian_list` | - | - | - | static-init entry of the live-pedestrian vector |
| 0x004b0d40 | `pedestrian_list_static_ctor` | `g_pedestrian_list` | - | - | - | builds the empty vector |
| 0x004b0d60 | `crt_atexit_register_004b0d60` | `g_pedestrian_list` | - | - | - | registers the static destructor |
| 0x004b0d70 | `pedestrian_list_static_dtor` | `g_pedestrian_list` | - | - | - | frees the storage; guard bit 1 |
| 0x004b0db0 | `crt_static_init_004b0db0` | `g_pedestrian_anim_list` | - | - | - | static-init entry of the animation-machine pool |
| 0x004b0dc0 | `pedestrian_anim_list_static_ctor` | `g_pedestrian_anim_list` | - | - | - | builds the empty vector |
| 0x004b0de0 | `crt_atexit_register_004b0de0` | `g_pedestrian_anim_list` | - | - | - | registers the static destructor |
| 0x004b0df0 | `pedestrian_anim_list_static_dtor` | `g_pedestrian_anim_list` | - | - | - | frees the storage; guard bit 2 |

#### `C_don_ocas`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004b0e50 | `pedestrian_crowd_ctor` | instance | `__ct__10C_don_ocasFv` | 0x00191db0 | CONFIRMED | kind `0x13`, vtable `0x0063c6d8` |
| 0x004b0e70 | `pedestrian_crowd_scalar_deleting_dtor` | instance | - | - | - | slot 16 |
| 0x004b0e90 | `pedestrian_crowd_dtor` | instance | `__dt__10C_don_ocasFv` | 0x00191df0 | CONFIRMED | reinstalls the vtable and chains `actor_dtor` |

#### `C_traffic_generator`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004b1c20 | `pedestrian_manager_ctor` | instance | `__ct__19C_traffic_generatorFv` | 0x00192c50 | CONFIRMED | kind `0x12`, vtable `0x0063c778`, the designer defaults |
| 0x004b1c90 | `pedestrian_manager_scalar_deleting_dtor` | instance | - | - | - | slot 16 |
| 0x004b1cb0 | `pedestrian_manager_dtor` | instance | `__dt__19C_traffic_generatorFv` | 0x00192cd0 | CONFIRMED | frees the name tables |

#### `C_traffic_element`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004b5bb0 | `pedestrian_ctor` | instance | `__ct__17C_traffic_elementFv` | 0x00199420 | CONFIRMED | zeroes the collision object and the pointers; installs no vtable |

### 3b. Accessors

No class here has a one-load or one-store method.

### 3c. Behaviour

#### `C_don_ocas`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004b0ea0 | `pedestrian_crowd_game_init` | `GameInit__10C_don_ocasFv` | 0x00191e50 | CONFIRMED | the proxy's per-scene init |
| 0x004b0eb0 | `pedestrian_crowd_is_possible_coll` | `IsPossibleColl__10C_don_ocasFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x00191e90 | CONFIRMED | swept-sphere query against the whole crowd |
| 0x004b1260 | `pedestrian_crowd_collision` | `Collision__10C_don_ocasFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00192380 | CONFIRMED | resolves the struck pedestrian, forwards, registers the offences |
| 0x004b1500 | `pedestrian_crowd_hit` | `Hit__10C_don_ocasF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00192750 | CONFIRMED | resolves the struck pedestrian and forwards to `pedestrian_hit` |
| 0x004b19d0 | `pedestrian_crowd_get_bound_points_2d` | `GetBoundPoints2D__10C_don_ocasFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x00192b00 | CONFIRMED | a `0.7`-half-width diamond around one pedestrian, for navigation |

#### `C_traffic_generator`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0044cf90 | `stub_retfalse_args40_0044cf90` | `IsPossibleColl__19C_traffic_generatorFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x00194f10 | base-stub | slot 33, an 8-byte PS2 body |
| 0x00461180 | `stub_noop_args4_00461180` | `Update__19C_traffic_generatorFUi` | 0x00198360 | base-stub | slot 14, an 8-byte PS2 body |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__19C_traffic_generatorFUi` | 0x00198370 | base-stub | slot 15, an 8-byte PS2 body |
| 0x00482690 | `stub_retfalse_args24_00482690` | `Collision__19C_traffic_generatorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001952b0 | base-stub | slot 30, an 8-byte PS2 body |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__19C_traffic_generatorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001952c0 | base-stub | slot 31, an 8-byte PS2 body |
| 0x004b1d20 | `pedestrian_manager_duplicate_from` | `DuplicateFrom__19C_traffic_generatorFP7C_actor` | 0x00192dc0 | CONFIRMED | copies the model-name table |
| 0x004b1e80 | `pedestrian_manager_save_data` | `SaveData__19C_traffic_generatorFP7C_chunk` | 0x00192f50 | CONFIRMED | writes the designer fields |
| 0x004b2030 | `pedestrian_manager_load_data` | `LoadData__19C_traffic_generatorFP7C_chunk` | 0x00193180 | CONFIRMED | reads the designer fields |
| 0x004b2610 | `pedestrian_manager_save_game_get_size` | `SaveGameGetSize__19C_traffic_generatorFv` | 0x00193ce0 | CONFIRMED | `0xbc` bytes per live pedestrian plus the header |
| 0x004b2630 | `pedestrian_manager_save_game_save` | `SaveGameSave__19C_traffic_generatorFPPv` | 0x00193d30 | CONFIRMED | saves the live pedestrians |
| 0x004b2930 | `pedestrian_manager_save_game_load` | `SaveGameLoad__19C_traffic_generatorFPPv` | 0x001940e0 | CONFIRMED | restores the live pedestrians |
| 0x004b2d10 | `pedestrian_manager_game_init` | `GameInit__19C_traffic_generatorFv` | 0x001945e0 | CONFIRMED | sizes and allocates the arena |
| 0x004b3000 | `pedestrian_manager_game_done` | `GameDone__19C_traffic_generatorFv` | 0x00194a00 | CONFIRMED | frees the arena |
| 0x004b3080 | `pedestrian_manager_player_talked_to_pedestrian` | `PlayerPromluvilNaTehoChuja__19C_traffic_generatorFP9I3D_frame` | 0x00194c60 | CONFIRMED | resolves the addressed pedestrian and calls its reply |
| 0x004b30a0 | `pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn` | `PlayerHoMaVytaseneho__19C_traffic_generatorFv` | 0x00194c90 | CONFIRMED | pedestrians within 25 units look at the player |
| 0x004b3170 | `pedestrian_manager_set_visible` | `SetVisible__19C_traffic_generatorFb` | 0x00194e00 | CONFIRMED | compare-and-set on `g_traffic_all_visible`, then the models |
| 0x004b31d0 | `pedestrian_manager_is_possible_coll_pedestrian` | `IsPossibleCollElem__19C_traffic_generatorFP17C_traffic_element` | 0x00194f20 | CONFIRMED | pedestrian against pedestrian |
| 0x004b33f0 | `pedestrian_manager_add_pedestrian` | `AddElement__19C_traffic_generatorFP17C_traffic_element` | 0x001952d0 | CONFIRMED | appends to `g_pedestrian_list` |
| 0x004b3510 | `pedestrian_manager_del_pedestrian` | `DelElement__19C_traffic_generatorFP17C_traffic_element` | 0x001957d0 | CONFIRMED | removes from `g_pedestrian_list` |
| 0x004b35c0 | `pedestrian_manager_ai` | `AI__19C_traffic_generatorFUi` | 0x001958c0 | CONFIRMED | the `0x70` ms round-robin spawner |
| 0x004b3780 | `pedestrian_manager_tick_pedestrians` | `TickElements__19C_traffic_generatorFUi` | 0x00195c80 | CONFIRMED | the crowd's per-frame driver |
| 0x004b38a0 | `pedestrian_manager_is_car_claimed_by_pedestrian` | `IsCarHasOcas__19C_traffic_generatorFP5C_car` | 0x00195e40 | CONFIRMED | some live pedestrian has the car assigned |
| 0x004b38f0 | `pedestrian_manager_try_send_pedestrian_to_parked_car` | `TryGoToAssHole__19C_traffic_generatorFv` | 0x00195ee0 | CONFIRMED | fired every `0x84b` ms; sends a pedestrian to a parked car |
| 0x004b3f50 | `pedestrian_manager_player_in_taxi` | `PlayerInTaxi__19C_traffic_generatorFP5C_car` | 0x001967f0 | CONFIRMED | sets `g_traffic_player_in_taxi` by the car's model name (§5) |
| 0x004b3fe0 | `pedestrian_manager_generate_passenger_for_taxi` | `GeneratePassengerForTaxi__19C_traffic_generatorFv` | 0x001968a0 | CONFIRMED | builds a passenger at the taxi |
| 0x004b4500 | `pedestrian_manager_enable_taxi_driver` | `EnableTaxiDriver__19C_traffic_generatorFb` | 0x00197020 | CONFIRMED | writes `g_traffic_taxi_driver_enabled` |
| 0x004b4520 | `pedestrian_manager_end_all_taxi_waves` | `ZrusVsechnyMavajici__19C_traffic_generatorFv` | 0x00197060 | CONFIRMED | cancels every hail |
| 0x004b4560 | `pedestrian_manager_clear_pedestrians` | `ClearElements__19C_traffic_generatorFv` | 0x001970f0 | CONFIRMED | destroys the vector's elements |
| 0x004b45f0 | `pedestrian_manager_spawn_cop_from_pedestrian` | `FizlosNamakatDoAkce__19C_traffic_generatorFP7C_humanUiRQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>bfb` | 0x001972e0 | CONFIRMED | turns a foot-cop record into a real `C_entity` |
| 0x004b4c60 | `pedestrian_manager_explosion` | `Explosion__19C_traffic_generatorFR8S_vectorff` | 0x00197a50 | CONFIRMED | pedestrians inside the radius react |
| 0x004b4f90 | `pedestrian_manager_fire` | `Fire__19C_traffic_generatorFR8S_vectorff` | 0x00197ec0 | CONFIRMED | pedestrians inside the radius panic |
| 0x004b5040 | `pedestrian_manager_get_pedestrian_from_frame` | `GetElemFromFrame__19C_traffic_generatorFP9I3D_frame` | 0x00197ff0 | CONFIRMED | maps a frame back to its pedestrian |
| 0x004b5070 | `pedestrian_manager_deactivate_all_pedestrians` | `FreeDynCollElements__19C_traffic_generatorFv` | 0x00198070 | CONFIRMED | retires every record |
| 0x004b50b0 | `pedestrian_manager_get_nearest_pedestrian_dist_sq` | `GetNearElementDist2__19C_traffic_generatorFR8S_vector` | 0x00198100 | CONFIRMED | squared distance of the nearest pedestrian |
| 0x004b5130 | `pedestrian_manager_find_pedestrian_ahead` | `IsNearElementBefore__19C_traffic_generatorFP17C_traffic_element` | 0x001981f0 | CONFIRMED | a pedestrian in front of this one |
| 0x004b51e0 | `pedestrian_manager_new_panic_at_position` | `NewPanic__19C_traffic_generatorFR8S_vector` | 0x00198380 | CONFIRMED | stores `g_traffic_last_panic` and panics the crowd |
| 0x004b5240 | `pedestrian_manager_new_panic_at_frame` | `NewPanic__19C_traffic_generatorFP9I3D_frame` | 0x00198440 | CONFIRMED | the same from a frame |
| 0x004b5270 | `pedestrian_manager_railway_in_station` | `RailwayInStation__19C_traffic_generatorFP9C_railwayRQ23std42vector<6rentry,Q23std18allocator<6rentry>>` | 0x001984b0 | CONFIRMED | fills a station platform from `g_pedestrian_list`, 15-unit radius |
| 0x004b5710 | `pedestrian_manager_add_free_person` | `AddFreePerson__19C_traffic_generatorFP9I3D_modelP7C_actorb` | 0x00198a00 | CONFIRMED | `operator_new(0x200)`, a record outside the arena |
| 0x004b5790 | `pedestrian_manager_add_free_person_from_car` | `AddFreePersonFromCar__19C_traffic_generatorFP9I3D_modelP7C_actorR8S_vectorR8S_vector` | 0x00198a80 | CONFIRMED | the same for a car's occupant |
| 0x004b58c0 | `pedestrian_manager_play_pedestrian_scream` | `ElementNewScream__19C_traffic_generatorFiiR8S_vectorffff` | 0x00198b90 | CONFIRMED | gated by `g_pedestrian_scream_enabled` and the budget |
| 0x004b5ab0 | `pedestrian_manager_release_pedestrians_from_car` | `VyfunOdTeMasiny__19C_traffic_generatorFP5C_car` | 0x001992d0 | CONFIRMED | restores `saved_move_target` |
| 0x0053c400 | `folded_actor_init_bool` | `Init__19C_traffic_generatorFP9I3D_frame` | 0x00192da0 | CONFIRMED | slot 18; one 32-byte body shared with `C_door` and `C_wagon` (`structs/door.md` §3) |
| 0x005c8d30 | `pedestrian_manager_set_scream_enabled` | - | - | - | `npc_shutup`: stores `argument == 0` |
| - | - | `DetachMesh__19C_traffic_generatorFP9I3D_model` | 0x001984a0 | base-stub | an 8-byte PS2 body; no PC slot identified |

#### `C_traffic_element`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004b5c10 | `pedestrian_avoid_obstacle` | `ObejdiTuPicovinu__17C_traffic_elementFP7C_actorP9I3D_frame` | 0x00199510 | CONFIRMED | detours around an obstacle |
| 0x004b60b0 | `pedestrian_advance_to_next_node` | `SetNewNode__17C_traffic_elementFv` | 0x00199c00 | CONFIRMED | pops the next path node into `move_target` |
| 0x004b6840 | `pedestrian_wave_at_taxi` | `MavaNaTaxi__17C_traffic_elementFUi` | 0x0019a5a0 | CONFIRMED | hails a taxi; reads `system\taxi_pas.dat` |
| 0x004b74f0 | `pedestrian_end_taxi_wave` | `EndMavaniNaTaxi__17C_traffic_elementFv` | 0x0019b2f0 | CONFIRMED | ends the hail |
| 0x004b7520 | `pedestrian_tick` | `Tick__17C_traffic_elementFUi` | 0x0019b340 | CONFIRMED | the per-pedestrian tick, 14 KB |
| 0x004bacf0 | `pedestrian_update_model` | `UpdateModel__17C_traffic_elementFv` | 0x0019f390 | CONFIRMED | refreshes the head point and the inverse matrix |
| 0x004bae30 | `pedestrian_set_model` | `SetModel__17C_traffic_elementFP9I3D_model` | 0x0019f550 | CONFIRMED | attaches a built model, seeds 90.0 |
| 0x004bb240 | `pedestrian_create_model` | `CreateModel__17C_traffic_elementFPcP19C_traffic_generator` | 0x0019f990 | CONFIRMED | loads the named model, the neck pose and the `back1` tag |
| 0x004bb6a0 | `pedestrian_set_alpha` | `SetAlpha__17C_traffic_elementFf` | 0x001a02a0 | CONFIRMED | forwards the fade to `base_frame` |
| 0x004bb6c0 | `pedestrian_destroy_model` | `DestroyModel__17C_traffic_elementFv` | 0x001a02d0 | CONFIRMED | clears the owner cell and releases the pose |
| 0x004bb7a0 | `pedestrian_activate_free` | `ActivateFree__17C_traffic_elementFbb` | 0x001a0400 | CONFIRMED | starts a free person |
| 0x004bbff0 | `pedestrian_activate` | `Activate__17C_traffic_elementFbff` | 0x001a0cb0 | CONFIRMED | the generator's scheduled spawn |
| 0x004bcfc0 | `pedestrian_deactivate` | `Deactivate__17C_traffic_elementFv` | 0x001a2400 | CONFIRMED | retires one pedestrian |
| 0x004bd0d0 | `pedestrian_hit` | `Hit__17C_traffic_elementF10E_hit_typeRC8S_vectorUiP7C_actorf` | 0x001a25b0 | CONFIRMED | the damage entry point; a 4 -> 1 fold with `HitFall`, `HitGun`, `HitStroke` |
| - | - | `HitFall__17C_traffic_elementFRC8S_vectorUi` | 0x001a3040 | inlined | host `pedestrian_hit`: `collision_dyn_erase` and the 45 and 135 degree tests |
| - | - | `HitGun__17C_traffic_elementFRC8S_vectorUib` | 0x001a3340 | inlined | host `pedestrian_hit`: the 90 degree test, `human_get_anim_id_for_death`, the scream |
| - | - | `HitStroke__17C_traffic_elementFRC8S_vectorUi` | 0x001a35c0 | inlined | host `pedestrian_hit`: the `pedestrian_manager_new_panic_at_position` call |
| 0x004bdc50 | `pedestrian_set_anim` | `SetAnim__17C_traffic_elementFv` | 0x001a2dc0 | CONFIRMED | starts `target_anim` and arms the collision insert |
| 0x004bde50 | `pedestrian_set_panic` | `SetPanic__17C_traffic_elementFv` | 0x001a37a0 | CONFIRMED | panic with a flee direction |
| 0x004bdfb0 | `pedestrian_set_mega_panic` | `SetMegaPanic__17C_traffic_elementFi` | 0x001a3980 | CONFIRMED | stores the level and its clip pair; drops the umbrella |
| 0x004be050 | `pedestrian_on_player_talk` | `PlayerNaMneMluvi__17C_traffic_elementFv` | 0x001a3a60 | CONFIRMED | the reply to being addressed |
| 0x004be0d0 | `pedestrian_greet_player` | `Karle_hoj__17C_traffic_elementFv` | 0x001a3b20 | CONFIRMED | the recognition greeting |
| 0x004be250 | `pedestrian_drop_umbrella` | `DropUmbrella__17C_traffic_elementFv` | 0x001a3cf0 | CONFIRMED | releases the umbrella frame |
| 0x004be390 | `pedestrian_spawn_gang_on_hit` | `CreateMafianos__17C_traffic_elementF10E_hit_typeRC8S_vectorUiP7C_actorf` | 0x001a3e50 | CONFIRMED | a hit on a gang pedestrian brings real actors |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004b0e40 | `stub_noop_cdecl_004b0e40` | - | - | - | an empty body reached by a jump from the slot before it |
| 0x004b6070 | `vec2_length` | - | - | - | 2D vector length, shared math |
| 0x004b6090 | `vec2_dot` | - | - | - | 2D dot product, shared math |

The unit also holds the `C_chunk` tail: `chunk_vector_allocate` (vtable `0x0063c828` slot 0), `chunk_ctor`, `chunk_open_file` and `chunk_dtor`, because `pedestrian_wave_at_taxi` builds a stack `C_chunk` to read `system\taxi_pas.dat`, the taxi passenger's script template (`structs/chunk.md` §4c). `C_chunk::Close` itself, `chunk_close`, sits in the `C_recorder` unit.

## 4. Why there are three classes

**`C_traffic_element` is the ambient pedestrian.** It is *not* an entity: it has **no
vtable** (`pedestrian_ctor` installs none), **no `kind` byte**, and it never appears in
the global actor list. It is a plain **`0x200`-byte record** living in an arena the
generator allocates.

**`C_don_ocas` is the crowd's stand-in ACTOR, and there is exactly one instance.**
`game_init` does `operator_new(0x70)` → `pedestrian_crowd_ctor` and stores the result at
**`C_game + 0x2a1c`**. `pedestrian_set_model` then stamps that singleton as the owner of
*every* pedestrian's `I3D_model`:

```
operator_new(4); *cell = *(C_game + 0x2a1c); model->+0x08 = cell;
```

So every engine-level collision, hit or bounds query aimed at a pedestrian model is
dispatched to the one `C_don_ocas`, whose four real methods either sweep the whole list
(`pedestrian_crowd_is_possible_coll`) or call
`pedestrian_manager_get_pedestrian_from_frame` to find *which* pedestrian was involved and
forward to that record. `pedestrian_destroy_model` clears the cell again.

**Why the split exists:** a real `C_actor` is heavyweight — the cop entity this very TU
spawns is `operator_new(0x128c)`, ten times a pedestrian's `0x200` — and every actor is
ticked, saved and replicated. A crowd of dozens could not afford that, so the crowd is an
array of light records with **one** actor in front of it.
`pedestrian_manager_spawn_cop_from_pedestrian` is the seam: when an ambient pedestrian
must become a real participant, it deactivates the record and allocates a full
`C_entity`.

**`C_traffic_generator`** (entity kind `0x12`, vtable `0x0063c778`) is the scene-placed
spawner: it owns the arena, the designer model-name table, the population budget, the
save/load path and the taxi feature. Two of its methods look like car work and are not:
`pedestrian_manager_railway_in_station` iterates `g_pedestrian_list` to fill a station
platform, and the four taxi methods are pedestrians hailing and riding taxis.

### 4a. Every element is a humanoid — measured

* `pedestrian_create_model` attaches an `I3D_pose` to the frame **`neck`** and reads the
  user-property tag of the frame **`back1`**, three letters and three digits such as `ABA095`
  (§1a: the letters give `+0x158` / `+0x159` / `+0x15a`, the digits the scale `+0x154`) —
  **unconditionally, for every element**.
* Every element owns a `C_animations_machine_de_luxe`, taken from the parallel pool
  `g_pedestrian_anim_list`.
* The only creation paths are `pedestrian_activate` (from the generator's model-name
  table) and `pedestrian_activate_free` (from `pedestrian_manager_add_free_person`,
  `_add_free_person_from_car`, `_generate_passenger_for_taxi`).
* The behaviour set is human-only: `pedestrian_drop_umbrella`, `pedestrian_wave_at_taxi`,
  `pedestrian_greet_player`, `pedestrian_on_player_talk`,
  `pedestrian_spawn_gang_on_hit`.
* Ambient **cars** are a different class entirely — `C_traffic_car`, kind `0xc`, its own
  `traffic_car_*` family.

The one "special" element type is a **foot cop** (`+0x15a` ∈ {1, 3}) — still a
pedestrian, and the population `pedestrian_manager_spawn_cop_from_pedestrian` draws from.

## 5. The taxi model table

`pedestrian_manager_player_in_taxi` sets `g_traffic_player_in_taxi` when the player's
current car's **model name** (`car+0x20cc`) case-insensitively matches one of a
hard-coded table of model names. The flag's only reader is
`pedestrian_manager_tick_pedestrians`, `&&`-ed with `g_traffic_taxi_driver_enabled`, and
it is what allows a passenger to be generated.

The PC table is `.data` `0x0064c060..0x0064c0c0`, walked **downwards** from the last
entry; the PS2 table holds the same list in the same order plus two entries (§8d):

| # | PC (9 entries) | PS2 (11 entries) |
|---|---|---|
| 1 | `taxi` | `taxi` |
| 2 | `hotrodp4` | `hotrodp4` |
| 3 | `hotrodp5` | `hotrodp5` |
| 4 | `tbirdold` | `tbirdold` |
| 5 | `fordhot` | `fordhot` |
| 6 | `phantomtaxi` | `phantomtaxi` |
| 7 | `hotrodp2` | `hotrodp2` |
| 8 | `fordapicktaxi` | `fordapicktaxi` |
| 9 | `blackdragon` | `blackdragon` |
| 10 | - | **`hotrodp3`** |
| 11 | - | **`thunderbird`** |

⚠️ The list is **not** "the taxis": it is the set of models the passenger generator will
accept. Six of the nine are not taxis at all (§10).

## 6. File-scope data

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_traffic_ratio_all` | 0x0064c058 | population multiplier on the designer budget | - | `pedestrian_manager_game_init` |
| `g_traffic_ratio_police` | 0x0064c05c | multiplier on the foot-cop slots allowed to spawn (§1b `+0xb4`) | - | `pedestrian_manager_game_init` |
| `g_pedestrian_scream_enabled` | 0x00656084 | the crowd's shout permission, a static member of `C_traffic_generator` | `pedestrian_manager_set_scream_enabled` | `pedestrian_manager_play_pedestrian_scream` |
| `g_traffic_player_in_taxi` | 0x00656085 | the player's car counts for the passenger generator | `pedestrian_manager_player_in_taxi` | `pedestrian_manager_tick_pedestrians` |
| `g_traffic_taxi_driver_enabled` | 0x00656086 | the taxi-driver feature is on | `pedestrian_manager_enable_taxi_driver` | `pedestrian_manager_tick_pedestrians` |
| `g_traffic_taxi_timer_ms` | 0x00656088 | countdown to the next taxi passenger | `pedestrian_manager_game_init` (0), `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_wave_at_taxi` (20000), `pedestrian_manager_tick_pedestrians` (-dt) | `pedestrian_manager_tick_pedestrians` |
| `g_traffic_all_visible` | 0x0065608c | master crowd visibility | `pedestrian_manager_set_visible` | `pedestrian_manager_ai`, `pedestrian_manager_tick_pedestrians`, `pedestrian_manager_explosion`, `pedestrian_manager_fire` |
| `g_traffic_parked_car_scan_timer_ms` | 0x00656090 | countdown to the next parked-car scan; resets to `0x84b` ms | `pedestrian_manager_game_init`, `pedestrian_manager_tick_pedestrians` | `pedestrian_manager_tick_pedestrians` |
| `g_traffic_horn_timer_ms` | 0x00656094 | decremented at the head of the crowd tick | `pedestrian_manager_tick_pedestrians` | `pedestrian_manager_tick_pedestrians` |
| `g_traffic_scream_budget` | 0x00656098 | above 100 refuses a scream; each scream adds 500 | `pedestrian_manager_play_pedestrian_scream` | `pedestrian_manager_play_pedestrian_scream` |
| `g_traffic_last_panic` | 0x0065609c | the last panic's position, 12 bytes | `pedestrian_manager_new_panic_at_position` | `pedestrian_set_panic` |
| `g_traffic_first_frame` | 0x006560a8 | the extra gate inside the spawner | - | `pedestrian_manager_ai` |
| `g_pedestrian_anim_list` | 0x006560ac | pool of `C_animations_machine_de_luxe`, one per pedestrian | - | - |
| `g_traffic_static_dtor_guard` | 0x006560bc | bits 1 and 2 guard the two static destructors | `pedestrian_list_static_dtor`, `pedestrian_anim_list_static_dtor` | `pedestrian_list_static_dtor`, `pedestrian_anim_list_static_dtor` |
| `g_pedestrian_list` | 0x006560c0 | the `std::vector<C_traffic_element*>` of live pedestrians | `pedestrian_manager_add_pedestrian`, `pedestrian_manager_del_pedestrian` | the crowd's walks |

`npc_shutup` writes `g_pedestrian_scream_enabled` through `pedestrian_manager_set_scream_enabled` with **inverted** polarity (`flag = argument == 0`), byte-verified on both builds and agreeing with the Mafia Script Data Reference (*0 — the pedestrian will talk*). It is not a sound master switch.

## 7. The savegame record

`pedestrian_manager_save_game_save` writes, after the `C_actor` block of `actor_save_game_save`, a 10-byte header: a version byte 1 (`pedestrian_manager_save_game_load` refuses any other), the generator's `selected` byte, the record count (`live_count`, int32) and `spawn_accumulator_ms` (int32). Then one packed `0xbc`-byte record per live pedestrian, in arena order:

| Offset | Size | Field | Evidence |
|---|---|---|---|
| +0x00 | 2 | the arena index | the load restores the record into that slot |
| +0x02 | 1 | `is_corpse` | - |
| +0x03 | 12 | `position` | the load also places the model frame there |
| +0x0f | 12 | `direction` | the load also turns the model frame (vtable `+0x0c`) |
| +0x1b | 12 | `velocity` | - |
| +0x27 | 12 | `anim_start_position` | - |
| +0x33 | 1 | `in_camera_sector` | - |
| +0x34 | 4 | `player_distance_sq` | - |
| +0x38 | 4 | `spawn_grace_ticks` | - |
| +0x3c | 4 | `target_node` | - |
| +0x40 | 12 | `target_pos` | - |
| +0x4c | 4 | `node_radius` | - |
| +0x50 | 4 | `previous_node` | - |
| +0x54 | 48 | `path_points` | the four points in order |
| +0x84 | 4 | `path_state` | - |
| +0x88 | 4 | `alpha` | written, not restored: the load stores 1.0 |
| +0x8c | 4 | `alpha_rate` | - |
| +0x90 | 4 | `health` | - |
| +0x94 | 4 | `current_anim` | restored into `target_anim`; the load zeroes `current_anim`, so the clip starts again |
| +0x98 | 4 | `following_anim` | - |
| +0x9c | 1 | `anim_one_shot` | - |
| +0x9d | 1 | `following_anim_one_shot` | - |
| +0x9e | 4 | `anim_end_ms` | - |
| +0xa2 | 4 | `ground_probe_ms` | - |
| +0xa6 | 4 | `wait_ms` | - |
| +0xaa | 1 | `side_stepping` | - |
| +0xab | 4 | `unread_198` | - |
| +0xaf | 1 | `panicking` | - |
| +0xb0 | 1 | `standing` | - |
| +0xb1 | 1 | `panic_latch` | - |
| +0xb2 | 4 | `panic_expiry_ms` | - |
| +0xb6 | 1 | `mega_panic_level` | - |
| +0xb7 | 1 | `promoted` | - |
| +0xb8 | 4 | `assigned_car` | the car's `net_guid` (`C_actor +0x18`), or -1 for none; the load resolves it with `game_get_actor_from_guid` and keeps it only for kind 4 |

The fields are copied in the record's order, so a writer of this layout reproduces every byte `pedestrian_manager_save_game_save` emits. The load is a copy with five differences: before it, it sets `web_link_mask` to 1 and calls `pedestrian_activate_free(0, 0)` on the slot; it stores 1.0 in `alpha` and 0 in `current_anim` (the saved clip goes to `target_anim`); it resolves `assigned_car` by guid; and it ends with `pedestrian_set_alpha(1.0)`. `live_count` is reset to 0 and rebuilt by the activations.

## 8. PC vs PS2

Everything cross-build lives here.

| topic | PC | PS2 |
|---|---|---|
| `C_traffic_element` size | `0x200` | `0x250` |
| element field base | `+0x00` | `+0x180` (a 0x180-byte prefix the PC lacks) — ⚠️ the delta is **not** uniform: the path/AI band differs again (`+0xb0` ↔ `+0x1c4`, `+0x190` ↔ `+0x210`, `+0x1b8` ↔ `+0x238`). Do not rebase band-wise |
| `C_traffic_generator` fields | see §1b | a clean **+0x10** skew throughout, agreed by the ctor, `DuplicateFrom` and `SaveData` |
| savegame header | 10 bytes | `0x10` bytes (the per-record `0xbc` is shared) |
| `NPC_speed_ratio_player` | absent — no such global and no such write | `GameInit__10C_don_ocas` seeds it to `1.0f` and `AI__19C_traffic_generator` recomputes it every frame from the player's speed, clamped 1.0–2.5. A PS2-only crowd-speed adaptation |
| `SetVisible` | toggles the flag and the model visibility | additionally adds/removes each pedestrian's fake shadow in the same loop |
| `ChangeSceneInit` / `ChangeSceneDone` | - | present — the city-streaming pair the PC build does not have |
| `C_traffic_element` polymorphism | no vtable | `__vt__17C_traffic_element` occurs **0 times** — non-polymorphic on both builds, so the vtable link-order channel can never show it |
| mega-panic animation ids | `0x14a`/`0x14b`, `0x14d`/`0x14e`, `0x150`/`0x151` | `0x118`/`0x119`, `0x11b`/`0x11c`, `0x11e`/`0x11f` — rebased per build; the **pairing** is the build-invariant |
| CRT string routines | intrinsics (`rep movsb`) | newlib calls (`strcpy`, `strcat`, `strcasecmp`), visible in `GeneratePassengerForTaxi` — a size ratio there measures library style, not a different implementation |
| `InvalidateActor` | `game_invalidate_actor` sends kinds `0xc` and `0x12` to the exit of its kind switch; the kinds it does handle match the PS2 set | `C_game::InvalidateActor` calls `InvalidateActor__13C_traffic_car` for kind `0xc` and `InvalidateActor__19C_traffic_generator` for kind `0x12`; the latter walks the pedestrian array clearing element `+0x248` and `+0x208` when they reference the dying actor |
| `HitFall` / `HitGun` / `HitStroke` | inlined in `pedestrian_hit`: `HitFall`'s `collision_dyn_erase` and 45° / 135° tests (`0.785398` / `2.356194`), `HitStroke`'s `pedestrian_manager_new_panic_at_position` call, `HitGun`'s 90° test, `human_get_anim_id_for_death` call and scream | three `C_traffic_element` methods (760, 640 and 476 B) called by `Hit` |
| generator `+0x94` | never read | `+0xa4`, never read either |

### 8a. The PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00194bc0 | `ChangeSceneInit__19C_traffic_generatorFv` | the city-streaming pair the PC does not have |
| 0x00194c50 | `ChangeSceneDone__19C_traffic_generatorFv` | as `ChangeSceneInit` |
| 0x00197240 | `InvalidateActor__19C_traffic_generatorFP7C_actor` | `game_invalidate_actor` has no kind-`0x12` arm |
| 0x001994c0 | `__dt__17C_traffic_elementFv` | a 68-byte body with no PC start in the gap |

### 8b. Cross-build evidence that settled PC-side questions

| PC question | what the PS2 answered |
|---|---|
| what does the global vector at `0x006560c0` hold? | `AddElement__19C_traffic_generatorFP17C_traffic_element` — a `std::vector<C_traffic_element*>`. The container lever; the PC decomp can only show a stride-4 pointer vector |
| what is the parallel vector at `0x006560ac`? | `ClearElements` destroys its elements as `__dt__28C_animations_machine_de_luxeFv` |
| is `pedestrian_manager_explosion` really an explosion? | `Explosion__19C_traffic_generatorFR8S_vectorff` |
| which of the two `NewPanic` overloads is which? | the PS2 signatures (`R8S_vector` vs `P9I3D_frame`) split them; the PC pair is otherwise identical in shape |
| is the 14 KB function at `0x004b7520` the element tick? | `Tick__17C_traffic_elementFUi`, matched on exact callee counts (`SetAnim` 5↔5, `SetNewNode` 4↔4, `AngleTo2` 3↔3) |

### 8c. Independent witness for `C_game+0x2a1c`

`pumper_on_collision` and `pumper_hit` both forward the impact to `*(C_game+0x2a1c)`'s
vtable slots 30 and 31 — the gas-station attendant hands collisions and hits to the
**`C_don_ocas` proxy**, so the engine processes them the way it processes a pedestrian's.
The PS2 twins forward to `C_game+0xe00`.

⇒ **PC `C_game+0x2a1c` ↔ PS2 `C_game+0xe00`**, corroborated from outside this subsystem:
the offset is otherwise derived from `game_init`'s own `operator_new(0x70)` store — one
producer; this is a consumer, in a different translation unit, reached from a different
direction (`structs/pumper.md`).

⚠️ It also says something about the class: `C_pumper` is a `C_actor` subclass with its
own vtable and its own kind byte, and it STILL routes impacts through the crowd proxy.
So the proxy is not only the pedestrians' stand-in — it is the shared impact handler
for human-shaped actors that do not implement one (§10).

### 8d. The taxi model table on PS2

PS2 `car+0x240c` is the twin of the PC model-name field `car+0x20cc` (§5). The PS2
pointer table at `0x00495b30` holds the PC's nine entries in the same order plus
**`hotrodp3`** and **`thunderbird`**, and its loop also walks the table downwards from the
last entry. Identical models in identical order with two extra PS2 rows is as strong as a
shared-magic-constant match gets, and it is what grades the `PlayerInTaxi` twin
**CONFIRMED**.

🎮 On PS2 a `hotrodp3` or a `thunderbird` counts for the Free-Ride passenger generator and
on PC neither does (§10).

## 9. Related

`structs/traffic_car.md` (the ambient cars) · `structs/using_object.md` (`usable`) · `structs/anim_machine.md` (`anim`) · `structs/collision.md` (`tDynamicCollObject`) · `structs/police_manager.md` (`police_manager_free_temp_enemy`, `pedestrian_manager_spawn_cop_from_pedestrian`) · `structs/pumper.md` (the proxy as impact handler) · `structs/chunk.md` §4c.

## 10. Open items

1. **In game:** does `g_traffic_scream_budget` audibly cap crowd screams during a shoot-out? A grenade in a busy street should hit the `> 100` refusal.
2. Why the hot-rods and `blackdragon` are among the models the passenger generator accepts is not established (§5).
3. **In game:** does a `hotrodp3` or a `thunderbird` generate Free-Ride taxi passengers on PS2 and not on PC (§8d)? One in-game test settles it.
4. `C_pumper` routes impacts through `C_don_ocas` (§8c); whether any third class does the same is not established.
