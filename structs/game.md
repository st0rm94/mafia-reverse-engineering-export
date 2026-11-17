# `C_game` — the game-world manager singleton

**Created** 2026-07-18 18:46 +0200 · **Modified** 2026-11-15 18:46 +0100

The world-manager singleton that owns sound, the breakage and particle effects, the actor lists (active /
inactive / temporary / invisible / dying / death queue), fake shadows, save/load, the scene lifecycle and
the per-frame tick of all of the above. The object is `*(int *)(g_act_mission + 0x24)`, reached in code
through `act_mission_get_game` or inline; any function taking that pointer as its `this` (ECX) is a
`C_game` method. It is heap-allocated by `mission_init` and owned by `C_mission`.

| | |
|---|---|
| PC prefix | `game_` |
| PC kind (`+0x10`) | - |
| PC vtable | - (`game_ctor` stores the allocator byte of `frame_placements` at `+0x00`, its only image-address constant is its exception handler, and every method is called directly) |
| PC base class | - |
| PC instance size | `0x3698` (`mission_init` calls `operator_new(0x3698)`) |
| PC constructor | `game_ctor` @ `0x00606750` |
| PC translation unit | `[0x005dc160, 0x005f79a0)` guard to guard (§7); the constructor and destructor sit in `Mission.cpp` (§5), the accessors at `0x00425510..0x00425580` and `0x005c8b20..0x005c8c70` in other units |
| PS2 class | `C_game` (tag `6C_game`), 177 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; the embedded classes are pointers to their own documents (§6). A
`std_vector` is 16 bytes: the MSVC allocator byte, then begin, end and cap. Its row in §2 covers all four.

```c
typedef struct C_game {                    // 0x3698 bytes; heap, owned by C_mission
    /* +0x00 */ std_vector frame_placements;         // [H] S_frame_placement elements (§1a)
    /* +0x10 */ std_vector broken_models;            // [H] I3D_frame * scene models broken into crash pieces
    /* +0x20 */ std_vector created_frames;           // [H] I3D_frame * script-created frames
    /* +0x30 */ std_vector box_saved_info;           // [H] S_box_placement_temp elements (§1b)
    /* +0x40 */ u8 running;                          // [H] set for the whole of a scene: the run guard
    /* +0x44 */ C_mission *mission;                  // [H] back-pointer to the owner; no reader (class census)
    /* +0x48 */ int mission_end_action;              // [H] 1, 2 or 3, from the game-over menu
    /* +0x4c */ G_Camera camera;                     // [H] embedded (§6)
    /* +0xe4 */ C_actor *player;                     // [H] the player's actor
    /* +0xe8 */ C_actor *camera_human;               // [H] the human the camera follows; no reader (class census)
    /* +0xec */ std_vector active_actors;            // [H] C_actor *: every actor in act state 0
    /* +0xfc */ std_vector inactive_actors;          // [H] C_actor *: every actor in act state 1
    /* +0x10c */ u8 act_state_dirty;                 // [H] the two lists need re-sorting
    /* +0x110 */ std_vector dying_humans;            // [H] C_human * running their death animation
    /* +0x120 */ std_vector temp_actors;             // [H] C_actor *: the master temporary-actor vector
    /* +0x130 */ std_vector temp_retire_queue;       // [H] C_actor *: temporaries awaiting removal
    /* +0x140 */ std_vector carried_actors;          // [H] C_actor *: the player and car of `change_mission`
    /* +0x150 */ std_vector lightness_records;       // [H] the ambient-lightness records
    /* +0x160 */ int lightness_cursor;               // [H] round-robin cursor over the records
    /* +0x164 */ u8 actor_lightness_on;              // [H] enables the lightness pass
    /* +0x168 */ std_vector light_cache;             // [H] I3D_light * of the scene
    /* +0x178 */ int light_cache_countdown;          // [H] re-test countdown, reloaded with `0x7e7`
    /* +0x17c */ std_vector invisible_actors;        // [H] S_invisible_actor elements (§1c)
    /* +0x18c */ int traffic_pick_timer;             // [H] traffic-generator pick timer, ms
    /* +0x190 */ int traffic_pick_period;            // [H] its period: 200 from the ctor, 6000 from `game_init`
    /* +0x194 */ int signal_elapsed_ms;              // [H] traffic-light clock
    /* +0x198 */ int signal_period_ms;               // [H] phase period: 200 from the ctor, 3000 from `game_init`
    /* +0x19c */ int signal_phase_a;                 // [H] phase of one crossing axis, 0..9
    /* +0x1a0 */ int signal_phase_b;                 // [H] phase of the other axis, 0..9
    /* +0x1a4 */ std_vector signals;                 // [H] S_semaphore elements (§1d)
    /* +0x1b4 */ std_vector road_blocks;             // [H] S_RoadBlock elements (§1e)
    /* +0x1c4 */ std_vector telephones;              // [H] S_Telephone elements (§1f)
    /* +0x1d4 */ std_vector shoots;                  // [H] S_shoot: the in-flight bullets (§1g)
    /* +0x1e4 */ std_vector signal_model_ids;        // [H] u16 ids of the `dopravni1B..8B.I3D` models
    /* +0x1f4 */ std_vector taxi_points;             // [H] I3D_frame * of the `taxi_point` scene objects
    /* +0x204 */ C_actor *taxi_passenger;            // [H] the current taxi-passenger candidate
    /* +0x208 */ C_Break_plain_glass glass;          // [H] embedded (§6)
    /* +0x240 */ C_Debris debris;                    // [H] embedded (§6)
    /* +0x26c */ C_compass compass;                  // [H] embedded (§6)
    /* +0x280 */ std_vector shadow_meshes;           // [H] S_shadow_temp source meshes
    /* +0x290 */ std_vector shadow_items_a;          // [H] S_shadow_item casters, `E_shadow_type` 0
    /* +0x2a0 */ std_vector shadow_items_b;          // [H] S_shadow_item casters, other types
    /* +0x2b0 */ int shadow_lod_timer;               // [H] fake-shadow LOD timer
    /* +0x2b4 */ int shadow_lod_period;              // [H] its period: 200 from the ctor, 823 from `game_init`
    /* +0x2b8 */ u8 shadow_lod_toggle;               // [H] which caster vector this pass walks
    /* +0x2bc */ C_rmorf rmorf;                      // [H] embedded (§6)
    /* +0x2c4 */ std_vector level_effects;           // [H] S_hydrant: the live level-effect registry (§1h)
    /* +0x2d4 */ std_vector paused_sounds;           // [H] S_paused_sound elements (§1w)
    /* +0x2e4 */ std_vector paused_media;            // [H] media ids that were playing
    /* +0x2f4 */ std_vector sounds;                  // [H] S_sound_play: the active 3-D sounds (§1i)
    /* +0x304 */ int sound_walk_timer;               // [H] sound-list walk timer
    /* +0x308 */ int sound_walk_period;              // [H] its period: 200 from the ctor, 813 from `game_init`
    /* +0x30c */ int sound_handle_next;              // [H] the next play handle
    /* +0x310 */ int sounds_fading;                  // [H] records still fading
    /* +0x314 */ std_vector fires;                   // [H] C_fire * registered by `game_new_fire`
    /* +0x324 */ C_smoke_trail *smoke_trails[32];    // [H] the smoke-trail actor ring
    /* +0x3a4 */ int smoke_trail_cursor;             // [H] its ring cursor
    /* +0x3a8 */ C_cartridge *cartridges_a[100];     // [H] casings, model `2nabojnice.i3d`
    /* +0x538 */ int cartridge_a_cursor;             // [H] its ring cursor
    /* +0x53c */ C_cartridge *cartridges_b[30];      // [H] casings, model `2nabojniceb.i3d`
    /* +0x5b4 */ int cartridge_b_cursor;             // [H] its ring cursor
    /* +0x5b8 */ I3D_frame *bullet_holes[100];       // [H] the bullet-hole frames, named `dUra_fiRe`
    /* +0x748 */ int bullet_hole_cursor;             // [H] its ring cursor
    /* +0x74c */ std_vector bullet_hole_materials;   // [H] I3D_material *: one per hole class
    /* +0x75c */ S_decal_projector projectors[100];  // [H] the decal projector pool (§1j)
    /* +0x29bc */ I3D_material *decal_materials[10]; // [H] the decal textures
    /* +0x29e4 */ int decal_cursor;                  // [H] the decal ring cursor
    /* +0x29e8 */ int decals_growing;                // [H] decals still growing in
    /* +0x29ec */ C_using_object using_object;       // [H] embedded (§6)
    /* +0x2a0c */ float player_speed;                // [H] the player's speed, units per second
    /* +0x2a10 */ S_vector player_prev_pos;          // [H] the player frame's position last tick
    /* +0x2a1c */ C_don_ocas *crowd_proxy;           // [H] the pedestrian crowd's proxy actor
    /* +0x2a20 */ I3D_sound *impact_sounds[10];      // [H] pool of positional impact sounds
    /* +0x2a48 */ S_shoot_light shoot_lights[10];    // [H] the shoot-light pool (§1k)
    /* +0x2ac0 */ S_flyby_sound flyby_sounds[3];     // [H] bullet fly-by sounds (§1l)
    /* +0x2ad8 */ u8 player_in_death_volume;         // [H] the player touched a death-collision volume
    /* +0x2ad9 */ u8 clear_differences_request;      // [H] one-shot: drop the difference data
    /* +0x2adc */ std_vector diff_added_frames;      // [H] I3D_frame *: the frames a difference file added
    /* +0x2aec */ std_vector diff_added_actors;      // [H] C_actor *: the actors a difference file added
    /* +0x2afc */ std_vector global_programs;        // [H] the global script programs
    /* +0x2b0c */ int frame_time_ms;                 // [H] the frame timestamp, ms: the global game clock
    /* +0x2b10 */ int tick_stamp;                    // [H] tick counter stored in the save
    /* +0x2b14 */ u32 frame_counter;                 // [H] frames since `game_init`
    /* +0x2b18 */ pursuit_event pursuit_events;       // [H] embedded (§6)
    /* +0x2b3c */ soundArena sound_arena;            // [H] embedded (§6)
    /* +0x2d44 */ u8 night_mission;                  // [H] 0 = day, 1 = night
    /* +0x2d48 */ S_script_event script_events[4];   // [H] timer timeout, fire, horn, water fall (§1m)
    /* +0x2d68 */ std_vector npc_kill_events;        // [H] S_npc_kill_event elements (§1n)
    /* +0x2d78 */ std_vector driver_men;             // [H] S_driver_man: the driver rigs (§1o)
    /* +0x2d88 */ float wheel_frames_car;            // [H] frame count of `AutoRidicVolant.I3D`, kept as a float
    /* +0x2d8c */ float wheel_frames_formula;        // [H] frame count of `FormuleVolant.I3D`, kept as a float
    /* +0x2d90 */ C_blood_manager blood;             // [H] embedded (§6)
    /* +0x2da0 */ police_manager police;               // [H] embedded (§6)
    /* +0x2e4c */ u8 stacks_clear_pending;           // [H] the next push of a script stack empties all three first
    /* +0x2e50 */ std_vector inventory_stack;        // [H] inventory snapshots
    /* +0x2e60 */ std_vector character_stack;        // [H] character stat snapshots
    /* +0x2e70 */ std_vector float_stack;            // [H] script floats
    /* +0x2e80 */ std_vector boxes;                  // [H] S_box_placement *: the placeable boxes (§1p)
    /* +0x2e90 */ std_vector box_dests;              // [H] S_box_dest *: the box destinations (§1q)
    /* +0x2ea0 */ S_using_object carried_box_node;   // [H] the carried box's destination record (`structs/using_object.md` §1a)
    /* +0x2ed8 */ u8 tearing_down;                   // [H] `game_done` is running
    /* +0x2edc */ int race_state;                    // [H] the mission-6 race state
    /* +0x2ee0 */ float *race_result_register;       // [H] the script float register the race publishes its result into
    /* +0x2ee4 */ std_vector death_queue;            // [H] C_actor *: the queued dead actors
    /* +0x2ef4 */ char cheat_buffer[0x20];           // [H] the typed cheat characters
    /* +0x2f14 */ u32 cheat_flags;                   // [H] gameplay flag word (§2z)
    /* +0x2f18 */ std_vector media;                  // [H] S_i_show: the media / in-game-FMV vector (§1r)
    /* +0x2f28 */ std_vector media_fades;            // [H] S_i_show_fade elements (§1x)
    /* +0x2f38 */ int media_id_next;                 // [H] the media id counter
    /* +0x2f40 */ std_vector talk_queue;             // [H] S_enemy_talk: the human-talk queue (§1s)
    /* +0x2f50 */ I3D_frame *car_gun_frame;          // [H] the gun frame linked under the player's car; nothing creates it
    /* +0x2f54 */ I3D_frame *unk_2f54[2];            // [U] zeroed by `game_init`, released by `game_done`; nothing creates them
    /* +0x2f5c */ I3D_frame *car_gun_muzzle_frames[2]; // [H] the two muzzle frames; no writer (.text scan)
    /* +0x2f64 */ I3D_frame *car_gun_eject_frames[2]; // [H] the two casing-eject frames; no writer (.text scan)
    /* +0x2f6c */ C_actor *car_gun_car;              // [H] the player's car the gun is on
    /* +0x2f70 */ C_actor *car_gun_target;           // [H] the car or human it aims at
    /* +0x2f74 */ int car_gun_fire_timer;            // [H] ms to the next double shot, reloads 330
    /* +0x2f78 */ int car_gun_scan_timer;            // [H] target-scan countdown: reloads 20 / 250
    /* +0x2f7c */ std_vector pending_actor_lookups;  // [H] S_find_actor elements (§1y)
    /* +0x2f8c */ std_vector script_particles;       // [H] script particle guids
    /* +0x2f9c */ u8 heavy_init;                     // [H] run the heavy half of `game_init`
    /* +0x2fa0 */ std_vector city_music_areas;       // [H] the city-music area frames
    /* +0x2fb0 */ int city_music_timer;              // [H] re-evaluation countdown, reloads 10000 ms
    /* +0x2fb4 */ int city_music_track;              // [H] the current track id
    /* +0x2fb8 */ int city_music_pending;            // [H] the pending track id
    /* +0x2fbc */ int city_music_channel;            // [H] the active channel handle, -1 = none
    /* +0x2fc0 */ u8 city_music_paused;              // [H] the paused gate
    /* +0x2fc4 */ std_vector film_music;             // [H] the film-sequence music handles
    /* +0x2fd4 */ int game_over_delay;               // [H] ms from the player's death to the game-over screen
    /* +0x2fd8 */ S_scratch scratches[100];          // [H] the car scratch decal pool (§1t)
    /* +0x3618 */ std_vector scream_areas;           // [H] S_scream_area elements (§1u)
    /* +0x3628 */ u32 scream_mask;                   // [H] OR of the camera's area masks, 1 when none hit
    /* +0x362c */ int scream_timer;                  // [H] scream-area pass timer, reloads `0x201` ms
    /* +0x3630 */ u8 weather_on;                     // [H] the weather enable flag
    /* +0x3634 */ int weather_particles;             // [H] the base particle count
    /* +0x3638 */ int rain_buffer_size;              // [H] the script's rain buffer size
    /* +0x363c */ u8 load_pending;                   // [H] the scene load latch of `game_init`
    /* +0x363d */ u8 done_intern_ran;                // [H] `game_done_intern` has run
    /* +0x363e */ u8 score_on;                       // [H] freeride score-display flag
    /* +0x3640 */ int score;                         // [H] freeride bonus score
    /* +0x3644 */ u8 camera_rot_repair;              // [H] repair the camera rotation
    /* +0x3648 */ S_quat camera_rot_saved;           // [H] the saved orientation
    /* +0x3658 */ I3D_model *sheet_model;            // [H] `9sheet.i3d`
    /* +0x365c */ I3D_model *unk_365c;               // [U] a model slot: only 0 is stored; hidden and released when set
    /* +0x3660 */ I3D_model *money_model;            // [H] `9money.i3d`
    /* +0x3664 */ u8 heal_cheat_ready;               // [H] one-shot gate of `KREFJEZIVOT`
    /* +0x3665 */ u8 script_mission_end;             // [H] the mission is ending under script control
    /* +0x3668 */ int car_sound_budget;              // [H] the nearest N cars get sound
    /* +0x366c */ int car_sound_timer;               // [H] the budget pass timer, reloads 200 / 800 ms
    /* +0x3670 */ u8 car_sound_pass;                 // [H] which of the two passes is next
    /* +0x3674 */ std_vector name_free_list;         // [H] recycled `0x20`-byte shadow names
    /* +0x3684 */ u8 item_blur_on;                   // [H] gates `game_item_blur_tick`
    /* +0x3688 */ std_vector item_blurs;             // [H] S_item_blur: motion-blur streak trails (§1v)
} C_game;
```

Gaps between the listed fields have no access by the census of §2 and are alignment padding.

**Why these array counts.**

| Field | Count | Each entry | Why that count |
|---|---|---|---|
| `smoke_trails` | 32 | a `C_smoke_trail` actor built by `game_init` | a ring: `game_new_smoke_trail` wraps its cursor at 32 |
| `cartridges_a` | 100 | a `C_cartridge` for the small casing | a ring: `game_new_cartridge` wraps its cursor at 100, and nothing allocates at runtime |
| `cartridges_b` | 30 | a `C_cartridge` for the big casing | the same ring code, wrapping at 30 |
| `bullet_holes` | 100 | an `OBJ_` driver frame named `dUra_fiRe` | a ring: `game_tick_shoot` wraps its cursor above 99, so the 101st hole reuses the oldest frame |
| `projectors` | 100 | `S_decal_projector` | a ring: `game_projector_advance_ring_cursor` wraps above 99 |
| `decal_materials` | 10 | a decal texture | `game_init` loads exactly ten named textures (§2) |
| `impact_sounds` | 10 | a driver sound object | `game_tick_shoot` scans the ten for one that is not playing; a burst beyond ten impacts is silent |
| `shoot_lights` | 10 | `S_shoot_light` | `game_add_shoot_light` and `game_tick` loop over exactly ten |
| `flyby_sounds` | 3 | `S_flyby_sound` | `game_new_shoot` and `game_tick` loop over exactly three; `game_new_shoot` takes the first idle one |
| `script_events` | 4 | `S_script_event` | one per script hook: timer timeout, player fire, player horn, player water fall |
| `cheat_buffer` | `0x20` | a typed character, complemented | 31 characters and a NUL; the longest code is shorter and `game_cheat_check` clears the buffer above 30 |
| `scratches` | 100 | `S_scratch` | `game_scratch_get_free_idx` reuses a nearby mark, then a free slot, then evicts the farthest of the 100 |
| `unk_2f54` | 2 | a released object pointer | `game_init` zeroes and `game_done` releases exactly two slots |
| `car_gun_muzzle_frames` | 2 | the frame a barrel fires from | `game_car_gun_tick` runs two passes, one per barrel |
| `car_gun_eject_frames` | 2 | the frame a casing leaves from | the same two passes |

### 1a. `S_frame_placement` — a frame's spawn placement, `0x34` bytes

`game_add_frame_placement` copies these out of the frame; `game_init` and `game_done` walk the vector.

```c
typedef struct S_frame_placement {                   // 0x34 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the key
    /* +0x04 */ S_vector pos;                  // [H] `frame+0x80`
    /* +0x10 */ S_quat rotation;               // [H] `frame+0x9c`
    /* +0x20 */ S_vector scale;                // [H] `frame+0x90`
    /* +0x2c */ I3D_frame *parent;             // [H] `frame+0x120`
    /* +0x30 */ u8 enabled;                    // [H] bit 0 of `frame+0xac`
} S_frame_placement;
```

### 1b. `S_box_placement_temp` — a box's saved start, `0x20` bytes

```c
typedef struct S_box_placement_temp {                // 0x20 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the key
    /* +0x04 */ I3D_frame *parent;             // [H] the frame the position is relative to
    /* +0x08 */ S_vector pos;                  // [H] the saved local position
    /* +0x14 */ S_vector dir;                  // [H] the saved direction, normalised
} S_box_placement_temp;
```

### 1c. `S_invisible_actor` — an actor toggled by visibility, `0x8` bytes

```c
typedef struct S_invisible_actor {                   // 0x8 bytes
    /* +0x00 */ C_actor *actor;                // [H] the actor; `game_tick` flips it between act states 0 and 1 once `stamp` is over 5 ticks old
    /* +0x04 */ u32 stamp;                     // [H] `tick_stamp` at the last toggle, 0 at first
} S_invisible_actor;
```

### 1d. `S_semaphore` — one traffic light, `0x14` bytes

```c
typedef struct S_semaphore {                         // 0x14 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the key: a `semafor.i3d` frame
    /* +0x04 */ I3D_frame *red;                // [H] child frame `RED`
    /* +0x08 */ I3D_frame *orange;             // [H] child frame `ORANGE`
    /* +0x0c */ I3D_frame *green;              // [H] child frame `GREEN`
    /* +0x10 */ int *phase;                    // [H] points at `signal_phase_a` or `_b`
} S_semaphore;
```

### 1e. `S_RoadBlock` — a roadblock node, `0x14` bytes

```c
typedef struct S_RoadBlock {                         // 0x14 bytes
    /* +0x00 */ u16 node;                      // [H] from `roads_get_near_node`
    /* +0x02 */ u8 engaged;                    // [H] a roadblock-eligible suspect is near: the block is manned and collides
    /* +0x04 */ std_vector actors;             // [H] C_actor * of the cars and officers spawned there
} S_RoadBlock;
```

### 1f. `S_Telephone` — a callable device, `0xc` bytes

```c
typedef struct S_Telephone {                         // 0xc bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the key; 0 for a device of the city cache
    /* +0x04 */ S_cache_model_inbb *cache_instance; // [H] the city-cache instance of a device with no frame
    /* +0x08 */ u8 type;                       // [H] 0 = wall alarm box, 1 = phone booth; no reader (class census)
} S_Telephone;
```

### 1g. `S_shoot` — one bullet in flight, `0x5c` bytes

`game_new_shoot` builds the record on its stack and `std_vector5c_insert_at` copies it in. The fields cover
every byte but the alignment gaps after `hit` and `material`.

```c
typedef struct S_shoot {                             // 0x5c bytes
    /* +0x00 */ S_vector pos;                  // [H] the current position
    /* +0x0c */ S_vector velocity;             // [H] advanced by `dt * 0.3`
    /* +0x18 */ float range;                   // [H] the remaining range
    /* +0x1c */ S_vector prev_pos;             // [H] the start of the tested segment
    /* +0x28 */ float damage;                  // [H] the hit's base damage; `game_tick_shoot` scales it down against `range`
    /* +0x2c */ C_actor *owner;                // [H] the shooter
    /* +0x30 */ I3D_frame *hit_frame;          // [H] the struck breakable frame, 0 = none
    /* +0x34 */ void *hit_face;                // [H] the struck face record
    /* +0x38 */ S_vector hit_pos;              // [H] the hit point
    /* +0x44 */ S_vector hit_normal;           // [H] the hit normal
    /* +0x50 */ u8 hit;                        // [H] a face was struck
    /* +0x54 */ int impact_sound;              // [H] the struck material's sound, -1 = none
    /* +0x58 */ u8 material;                   // [H] the struck material's index
} S_shoot;
```

### 1h. `S_hydrant` — one level effect, `0x18` bytes

```c
typedef struct S_hydrant {                           // 0x18 bytes
    /* +0x00 */ int particle_guid;             // [H] the effect's particle: `game_tick` tests it alive, `game_del_all_hydrants` destroys it
    /* +0x04 */ u8 covered;                    // [H] something is overhead: emission is stopped
    /* +0x08 */ S_vector pos;                  // [H] the effect's position, a hydrant's frame position `+0x40`; `game_tick` reads it
    /* +0x14 */ I3D_sound *sound;              // [H] the looping sound object, 0 = none; `game_del_all_hydrants` stops and deletes it
} S_hydrant;
```

### 1i. `S_sound_play` — one active 3-D sound, `0x18` bytes

```c
typedef struct S_sound_play {                        // 0x18 bytes
    /* +0x00 */ I3D_sound *sound;              // [H] the playing sound object; `game_tick` retires it once stopped
    /* +0x04 */ int handle;                    // [H] the play handle
    /* +0x08 */ float target_volume;           // [H] of a fade-in
    /* +0x0c */ float fade_rate;               // [H] signed, per ms; positive fades in, otherwise out and stops at 0
    /* +0x10 */ u8 fading;                     // [H] 1 = `game_stop_3d_sound` asked for a fade-out, counted in `sounds_fading`
    /* +0x14 */ int last_position;             // [H] shows when a sound has finished
} S_sound_play;
```

### 1j. `S_decal_projector` — one decal, `0x58` bytes

```c
typedef struct S_decal_projector {                   // 0x58 bytes
    /* +0x00 */ I3D_frame *projector;          // [H] the decal's projector frame; `game_clear_projectors_for_frame` hides it
    /* +0x04 */ tDynamicCollObject coll;       // [H] type `0x48`
    /* +0x4c */ u8 registered;                 // [H] `coll` is in the dynamic grid
    /* +0x4d */ u8 growing;                    // [H] 1 while `grow_scale` ramps; `game_tick` clears it at 1.0
    /* +0x50 */ float grow_scale;              // [H] the frame's scale factor: starts at 0.1, `game_tick` adds `dt * 0.0003` up to 1.0
    /* +0x54 */ I3D_frame *owner;              // [H] the frame the decal is bound to; `game_clear_projectors_for_frame` matches it
} S_decal_projector;
```

### 1k. `S_shoot_light` — one shoot light, `0xc` bytes

```c
typedef struct S_shoot_light {                       // 0xc bytes
    /* +0x00 */ I3D_light *light;              // [H] the pool's light; `game_add_shoot_light` moves it and switches it on
    /* +0x04 */ int lifetime_ms;               // [H] seeded -1 by `game_init`
    /* +0x08 */ u16 flags;                     // [H] bit 0 = flicker, then fade over the last 500 ms
} S_shoot_light;
```

### 1l. `S_flyby_sound` — one bullet fly-by sound, `0x8` bytes

```c
typedef struct S_flyby_sound {                       // 0x8 bytes
    /* +0x00 */ I3D_sound *sound;              // [H] sound id `0xbf`
    /* +0x04 */ int delay_ms;                  // [H] -1 = idle; the sound starts when it runs out
} S_flyby_sound;
```

### 1m. `S_script_event` — one script-event slot, `0x8` bytes

```c
typedef struct S_script_event {                      // 0x8 bytes
    /* +0x00 */ C_program *program;            // [H] the program the event is raised on, 0 = no hook
    /* +0x04 */ int event;                     // [H] the event id passed to `program_set_event`
} S_script_event;
```

### 1n. `S_npc_kill_event` — one registered kill event, `0x28` bytes

```c
typedef struct S_npc_kill_event {                    // 0x28 bytes
    /* +0x00 */ C_program *program;            // [H] the program the event is raised on
    /* +0x04 */ int event;                     // [H] the event id `game_do_npc_kill_event` raises
    /* +0x08 */ char name[0x20];               // [H] the model name it matches; `0x20` is the copy's capacity
} S_npc_kill_event;
```

### 1o. `S_driver_man` — one driver rig, `0x10` bytes

```c
typedef struct S_driver_man {                        // 0x10 bytes
    /* +0x00 */ I3D_model *model;              // [H] the key
    /* +0x04 */ I3D_animation_set *anim_set;   // [H] the wheel animation
    /* +0x08 */ C_car *car;                    // [H] the car whose steering sets the wheel animation's frame (§8a)
    /* +0x0c */ u8 formula;                    // [H] `FormuleVolant.I3D` rather than `AutoRidicVolant.I3D`
} S_driver_man;
```

### 1p. `S_box_placement` — one placeable box, `0x98` bytes

The fields cover every byte up to `+0x95`; `+0x95..+0x97` is the alignment gap at the record's end.

```c
typedef struct S_box_placement {                     // 0x98 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the key
    /* +0x04 */ S_using_object *use_node;      // [H] its pick-up record, registered while the box can be picked up
    /* +0x08 */ tDynamicCollObject coll;       // [H] type `0x40`
    /* +0x50 */ S_matrix inv_world;            // [H] the inverse of the frame's world matrix; no reader (class census)
    /* +0x90 */ int owner;                     // [H] `0x1d97c` = no owner yet
    /* +0x94 */ u8 usable;                     // [H] 1 = can be picked up; 0 while loaded on a car
} S_box_placement;
```

### 1q. `S_box_dest` — one box destination, `0x8` bytes

```c
typedef struct S_box_dest {                          // 0x8 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] the destination frame, the record's key
    /* +0x04 */ int owner;                     // [H] the owner id; `game_box_dest_count_owned` counts its matches
} S_box_dest;
```

### 1r. `S_i_show` — one media entry, `0x14` bytes

```c
typedef struct S_i_show {                            // 0x14 bytes
    /* +0x00 */ IShow *show;                   // [H] the media player object; `game_media_get` returns it
    /* +0x04 */ int id;                        // [H] the entry's id from `media_id_next`, the key of the `game_media_*` lookups
    /* +0x08 */ u32 flags;                     // [H] bit 0 = loop, bit 1 = left out of the save
    /* +0x0c */ char *name;                    // [H] a heap copy of the opened file name
    /* +0x10 */ float volume;                  // [H] the entry's volume; the player runs at `g_music_volume` times this
} S_i_show;
```

### 1s. `S_enemy_talk` — one queued speech, `0x8` bytes

```c
typedef struct S_enemy_talk {                        // 0x8 bytes
    /* +0x00 */ C_human *human;                // [H] the speaking human; `game_human_talk_stop_for_human` erases by it
    /* +0x04 */ int speech_id;                 // [H] the speech id; `game_human_talk_del` erases by it
} S_enemy_talk;
```

### 1t. `S_scratch` — one car scratch, `0x10` bytes

```c
typedef struct S_scratch {                           // 0x10 bytes
    /* +0x00 */ I3D_frame *scratch_frame;      // [H] the scratch's own mesh frame; `game_scratch_recompile` rebuilds it
    /* +0x04 */ C_car *car;                    // [H] 0 = free slot
    /* +0x08 */ I3D_frame *source_frame;       // [H] the frame the mesh is built from; with `car` the key of `game_scratch_recompile`
    /* +0x0c */ int age;                       // [H] reuse count: `game_scratch_get_free_idx` zeroes it on a new slot, adds 1 per reuse
} S_scratch;
```

### 1u. `S_scream_area` — one scream area, `0x8` bytes

```c
typedef struct S_scream_area {                       // 0x8 bytes
    /* +0x00 */ I3D_frame *frame;              // [H] named `enm_scrm_*`
    /* +0x04 */ u16 mask;                      // [H] the hexadecimal number in the frame name
} S_scream_area;
```

### 1v. `S_item_blur` — one motion-blur streak trail, `0x28` bytes

Nothing appends a record to `item_blurs`, so the fields the tick only reads have no writer (§2v).

```c
typedef struct S_item_blur {                         // 0x28 bytes
    /* +0x00 */ C_actor *actor;                // [H] the actor the trail follows; no writer
    /* +0x04 */ I3D_frame *frame;              // [H] the frame the streaks start from; no writer
    /* +0x08 */ I3D_frame **sub_items;         // [H] the streak quads; no writer
    /* +0x0c */ int sub_item_count;            // [H] entries of `sub_items`; `game_item_blur_del_item` destroys each; no writer
    /* +0x10 */ S_vector prev_pos;             // [H] the actor frame's camera-space position last tick
    /* +0x1c */ float speed;                   // [H] smoothed speed: (old + distance / dt) / 2
    /* +0x20 */ u8 shown;                      // [H] the streaks are drawn
    /* +0x24 */ float max_length;              // [H] the streak length cap; no writer
} S_item_blur;
```

### 1w. `S_paused_sound` — one paused sound, `0x14` bytes

The name is the project's. `game_pause_scene_sounds_recursive` builds one for every playing sound frame;
`game_play_all_sounds` puts each value back.

```c
typedef struct S_paused_sound {                      // 0x14 bytes
    /* +0x00 */ I3D_sound *sound;              // [H] the paused sound frame
    /* +0x04 */ int volume;                    // [H] its `volume` (`+0x18c`)
    /* +0x08 */ int unk_08;                    // [U] its `+0x188`; put back through the sound's vtable `+0x64`
    /* +0x0c */ u8 looped;                     // [H] its byte `+0x164`; put back through the vtable `+0x6c`
    /* +0x10 */ int play_position;             // [H] the result of its vtable `+0x80`; put back through `+0x70`
} S_paused_sound;
```

### 1x. `S_i_show_fade` — one running media fade, `0x10` bytes

```c
typedef struct S_i_show_fade {                       // 0x10 bytes
    /* +0x00 */ int media_id;                  // [H] the `S_i_show.id` it fades
    /* +0x04 */ float from_volume;             // [H] the start volume
    /* +0x08 */ float to_volume;               // [H] the target volume
    /* +0x0c */ s16 elapsed_ms;                // [H] advanced by the frame time
    /* +0x0e */ s16 duration_ms;               // [H] the fade's length
} S_i_show_fade;
```

### 1y. `S_find_actor` — one deferred actor lookup, `0xc` bytes

Nothing appends a record to `pending_actor_lookups` (§2y).

```c
typedef struct S_find_actor {                        // 0xc bytes
    /* +0x00 */ C_program *program;            // [H] the program the lookup runs on, skipped when 0; no writer
    /* +0x04 */ int actor_reg;                 // [M] the first argument of `program_find_actor_later`
    /* +0x08 */ u32 guid;                      // [M] its second argument
} S_find_actor;
```

## 2. Field evidence

The cells come from a register census over all of `.text`: a register holds `C_game *` when it is
ECX on entry to a `game_` method, or it loads `[g_act_mission + 0x24]`, or it receives `act_mission_get_game`;
tags follow moves, stack slots and jumps. A function that reaches the object through a path the census cannot
follow (an argument, a pointer kept in a member) appears only when a body read found it, and those cells were
checked for the field's displacement in the function's disassembly. The bound of the census is Open item 3.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame_placements` | H | `game_add_frame_placement`, `game_ctor`, `game_del_frame_placement`, `game_done`, `game_init` | `game_add_frame_placement`, `game_del_frame_placement`, `game_done`, `game_init`, `human_collision`, `railway_ai`, `railway_hit` | the spawn transform of each registered frame; records are keyed by frame |
| +0x10 | `broken_models` | H | `game_ctor`, `crash_obj_create_from_model` | `game_done`, `game_init` | `crash_obj_create_from_model` pushes each scene model it breaks and switches it off; `game_done` switches every one back on (frame vtable `+0x24` with 1); `game_init` empties it |
| +0x20 | `created_frames` | H | `game_add_created_frame`, `game_ctor`, `game_del_created_frame` | `game_add_created_frame`, `game_del_created_frame`, `game_done`, `game_init` | `game_done` releases each frame |
| +0x30 | `box_saved_info` | H | `game_box_del_info`, `game_box_restore_info`, `game_ctor` | `game_box_del_info`, `game_box_get_start_point`, `game_box_restore_info`, `game_box_save_info`, `game_done`, `game_init` | - |
| +0x40 | `running` | H | `game_ctor`, `game_done`, `game_init` | `game_apply_weather_detail_level`, `game_done`, `game_init`, `game_tick`, `game_update_controls`, `door_set_state`, `engine_game_loop` and 19 more | `game_done` returns unless it is set and clears it; `game_init` returns at once while it is set; `game_update_controls` gates on it |
| +0x44 | `mission` | H | `mission_init` | - | `mission_init` stores its own `this` right after `game_ctor`; no access from the class's own code (Open item 3) |
| +0x48 | `mission_end_action` | H | `game_init`, `game_set_mission_end_action`, `game_tick`, `police_manager_tick` | `engine_game_loop` | `game_tick` writes it when the game-over screen closes; `engine_game_loop` acts on it |
| +0x4c | `camera` | H | `game_ctor` | `game_done`, `game_get_camera`, `game_init`, `game_tick`, `car_explosion`, `engine_new_mission`, `human_animation_evaluate_and_dispatch` and 17 more | `game_get_camera` returns its address; 13 callsites |
| +0xe4 | `player` | H | `game_done`, `game_init`, `game_invalidate_actor`, `engine_new_mission`, `human_change_model`, `loadsave_savegame_load`, `racing_activate_racers` | `game_box_repose_dest_player`, `game_character_stack_pop`, `game_cheat_check`, `game_done`, `game_get_player_actor`, `game_init`, `game_invalidate_actor` and 79 more | `game_init` clears it, then takes the last kind-2 actor of the mission list whose `+0x04` track bit `0x10` is clear; `engine_new_mission`, `racing_activate_racers` and `loadsave_savegame_load` set it too; `game_done` clears it, and `game_invalidate_actor` when the dying actor is the player. Read through `game_get_player_actor` and inline, also as the `*(world+0xe4) == this` am-I-the-player gate of several `C_human` methods |
| +0xe8 | `camera_human` | H | `game_init`, `game_set_human`, `engine_new_mission`, `loadsave_savegame_load`, `racing_activate_racers` | - | `game_set_human` accepts kinds 2 and `0x1b` only and calls `camera_set_player`; `game_init` copies `+0xe4` here. No reader in the class or in the functions that call a `game_` method (Open item 3) |
| +0xec | `active_actors` | H | `game_ctor`, `game_init`, `garage_game_done` | `game_add_temporary_actor`, `game_change_act_state`, `game_cheat_check`, `game_clear_differences`, `game_done`, `game_entity_block`, `game_get_active_actor_list` and 20 more | scene-placed and runtime alike; §4b. Address accessor `game_get_active_actor_list` |
| +0xfc | `inactive_actors` | H | `game_change_act_state`, `game_ctor`, `game_init`, `game_remove_temporary_actor_intern`, `garage_game_done` | `game_add_temporary_actor`, `game_change_act_state`, `game_clear_differences`, `game_done`, `game_entity_block`, `game_init`, `game_invalidate_coll` and 10 more | §4b. It has no address accessor: every reader reads `+0x100` / `+0x104` inline |
| +0x10c | `act_state_dirty` | H | `game_change_act_state`, `game_init`, `actor_rebirth`, `actor_set_act_state` | `game_init`, `game_tick` | `actor_set_act_state` and `actor_rebirth` set it, `game_change_act_state` clears it |
| +0x110 | `dying_humans` | H | `game_ctor`, `game_dying_human_del`, `game_dying_human_tick`, `game_init` | `game_done`, `game_dying_human_add`, `game_dying_human_del`, `game_dying_human_tick`, `game_init`, `game_tick` | - |
| +0x120 | `temp_actors` | H | `game_clear_temporary_actors`, `game_ctor`, `game_remove_temporary_actor_now` | `game_add_temporary_actor`, `game_add_temporary_actor_intern`, `game_clear_differences`, `game_clear_temporary_actors`, `game_new_smoke_trail`, `game_remove_temporary_actor_intern`, `game_remove_temporary_actor_now` | §4b |
| +0x130 | `temp_retire_queue` | H | `game_ctor`, `game_init` | `game_done`, `game_init`, `game_remove_temporary_actor`, `game_remove_temporary_actor_intern`, `game_tick` | §4b: the deferred half of the removal |
| +0x140 | `carried_actors` | H | `game_ctor`, `game_done`, `engine_new_mission` | `game_done`, `game_init`, `loadsave_savegame_save` | `engine_new_mission` pushes them after `game_init` (from `g_new_mission_actors`), sets `owns_frame` (`+0x6c`) on the one that is not a car and clears the frame parent `+0x58` of both; `game_done` first drops a car parked in the garage, then calls `mission_active_actor_drop` and `mission_del_actor` on every entry; `loadsave_savegame_save` writes these actors into a mission save |
| +0x150 | `lightness_records` | H | `game_ctor`, `game_lightness_clear`, `game_lightness_del` | `game_lightness_add`, `game_lightness_clear`, `game_lightness_del`, `game_lightness_tick`, `game_tick` | `structs/lightness.md` §1 |
| +0x160 | `lightness_cursor` | H | `game_init`, `game_lightness_tick` | `game_lightness_tick` | `structs/lightness.md` §1 |
| +0x164 | `actor_lightness_on` | H | `game_init`, `game_set_actor_lightness_flag` | `game_tick` | ¹ |
| +0x168 | `light_cache` | H | `game_ctor` | `game_done`, `game_init`, `game_recompute_light_cache`, `game_tick`, `game_use_light_cache_system` | `structs/lightness.md` |
| +0x178 | `light_cache_countdown` | H | `game_init`, `game_recompute_light_cache`, `game_tick`, `game_use_light_cache_system` | `game_tick` | `structs/lightness.md` |
| +0x17c | `invisible_actors` | H | `game_ctor` | `game_add_invisible_actor`, `game_init`, `game_tick` | - |
| +0x18c | `traffic_pick_timer` | H | `game_ctor`, `game_init`, `game_tick` | `game_tick` | §8a |
| +0x190 | `traffic_pick_period` | H | `game_ctor`, `game_init` | `game_init`, `game_tick` | §8a |
| +0x194 | `signal_elapsed_ms` | H | `game_ctor`, `game_init`, `game_tick` | `game_tick` | §4d, `structs/driver.md` §4 |
| +0x198 | `signal_period_ms` | H | `game_ctor`, `game_init` | `game_tick` | §4d |
| +0x19c | `signal_phase_a` | H | `game_init`, `game_tick` | `game_tick` | `game_init` seeds it 0; `game_tick` advances it mod 10 |
| +0x1a0 | `signal_phase_b` | H | `game_init`, `game_tick` | `game_tick` | `game_init` seeds it 5; `game_tick` advances it mod 10 |
| +0x1a4 | `signals` | H | `game_ctor`, `game_del_traffic_signal`, `game_free_traffic_signals` | `game_del_traffic_signal`, `game_free_traffic_signals`, `game_load_traffic_signals`, `game_tick_traffic_signals` | built wholesale at scene load (§4d) |
| +0x1b4 | `road_blocks` | H | `game_ctor`, `game_done` | `game_done`, `game_load_road_blocks`, `actor_is_possible_coll_all`, `police_manager_spawn_roadblock_in_neighbor_cells`, `police_manager_tick` | built by `game_load_road_blocks` |
| +0x1c4 | `telephones` | H | `game_ctor`, `game_done`, `game_telephone_list_erase_by_key` | `game_done`, `game_get_telephone_list`, `game_load_telephones`, `game_telephone_list_contains_key`, `game_telephone_list_erase_by_key` | ² |
| +0x1d4 | `shoots` | H | `game_ctor`, `game_new_shoot`, `game_tick_shoot` | `game_done`, `game_invalidate_actor`, `game_new_shoot`, `game_tick_shoot` | `game_new_shoot` produces them through `std_vector5c_insert_at`, `game_tick_shoot` consumes them |
| +0x1e4 | `signal_model_ids` | H | `game_ctor`, `game_done` | `game_build_traffic_signal_model_idx`, `game_destroy_control_signal`, `game_done` | produced by `game_build_traffic_signal_model_idx`, consumed by `game_destroy_control_signal` |
| +0x1f4 | `taxi_points` | H | `game_ctor`, `game_done` | `game_done`, `game_load_taxi_points`, `pedestrian_manager_generate_passenger_for_taxi`, `pedestrian_wave_at_taxi` | built by `game_load_taxi_points` |
| +0x204 | `taxi_passenger` | H | `game_init`, `game_invalidate_actor`, `pedestrian_wave_at_taxi` | `game_invalidate_actor`, `game_is_valid_taxi_passenger` | - |
| +0x208 | `glass` | H | `game_ctor` | `game_done`, `game_init`, `game_tick`, `game_tick_shoot` | - |
| +0x240 | `debris` | H | `game_ctor` | `game_done`, `game_init`, `game_tick`, `car_explosion` | - |
| +0x26c | `compass` | H | `game_ctor` | `game_done`, `game_init` | - |
| +0x280 | `shadow_meshes` | H | `game_ctor`, `game_done_intern` | `game_add_fake_shadow_buffer`, `game_add_frame_for_shadow`, `game_bind_fake_shadow_buffer`, `game_done_intern`, `game_init`, `game_set_frame_for_shadow`, `game_tick` | `structs/fake_shadow.md` |
| +0x290 | `shadow_items_a` | H | `game_ctor` | `game_add_frame_for_shadow`, `game_done`, `game_init`, `game_set_frame_for_shadow`, `game_tick` | `structs/fake_shadow.md` |
| +0x2a0 | `shadow_items_b` | H | `game_ctor` | `game_add_frame_for_shadow`, `game_done`, `game_init`, `game_set_frame_for_shadow`, `game_tick` | `structs/fake_shadow.md` |
| +0x2b0 | `shadow_lod_timer` | H | `game_ctor`, `game_init`, `game_tick` | `game_init`, `game_tick` | §8a |
| +0x2b4 | `shadow_lod_period` | H | `game_ctor`, `game_init` | `game_init`, `game_tick` | §8a |
| +0x2b8 | `shadow_lod_toggle` | H | `game_init`, `game_tick` | `game_init`, `game_tick` | §8a |
| +0x2bc | `rmorf` | H | `game_ctor` | `game_done`, `game_init`, `game_tick` | ³ |
| +0x2c4 | `level_effects` | H | `game_ctor`, `game_del_all_hydrants` | `game_add_hydrant`, `game_del_all_hydrants`, `game_tick` | §4c |
| +0x2d4 | `paused_sounds` | H | `game_ctor`, `game_play_all_sounds` | `game_done`, `game_init`, `game_pause_all_sounds`, `game_play_all_sounds` | `game_pause_all_sounds` fills it, `game_play_all_sounds` replays it; §2w |
| +0x2e4 | `paused_media` | H | `game_ctor`, `game_play_all_sounds` | `game_done`, `game_init`, `game_pause_all_sounds`, `game_play_all_sounds` | the same pair |
| +0x2f4 | `sounds` | H | `game_clear_effects`, `game_ctor` | `game_clear_effects`, `game_play_3d_sound_by_name`, `game_stop_3d_sound`, `game_tick` | §4e |
| +0x304 | `sound_walk_timer` | H | `game_ctor`, `game_init`, `game_tick` | `game_tick` | §8a |
| +0x308 | `sound_walk_period` | H | `game_ctor`, `game_init` | `game_tick` | §8a |
| +0x30c | `sound_handle_next` | H | `game_init`, `game_play_3d_sound_by_name` | `game_play_3d_sound`, `game_play_3d_sound_by_name` | - |
| +0x310 | `sounds_fading` | H | `game_init`, `game_play_3d_sound_by_name`, `game_stop_3d_sound`, `game_tick` | `game_stop_3d_sound`, `game_tick` | while non-zero the walk runs every frame; `game_stop_3d_sound` increments it, `game_tick` decrements it |
| +0x314 | `fires` | H | `game_ctor`, `game_init`, `game_tick` | `game_done`, `game_init`, `game_invalidate_actor`, `game_new_fire`, `game_tick` | `game_tick` retires them, `game_done` clears them, `game_invalidate_actor` drops them |
| +0x324 | `smoke_trails` | H | `game_init` (by address: zeroes the 32 slots, then stores each `C_smoke_trail` it builds) | `game_done_intern`, `game_init`, `game_new_smoke_trail` | built by `game_init`, served by `game_new_smoke_trail` |
| +0x3a4 | `smoke_trail_cursor` | H | `game_init`, `game_new_smoke_trail` | `game_new_smoke_trail` | - |
| +0x3a8 | `cartridges_a` | H | `game_init` (by address: zeroes the 100 slots, then stores each `C_cartridge` it builds) | `game_clear_cartridges`, `game_done`, `game_done_intern`, `game_init`, `game_new_cartridge` | §4l |
| +0x538 | `cartridge_a_cursor` | H | `game_init`, `game_new_cartridge` | `game_new_cartridge` | §4l |
| +0x53c | `cartridges_b` | H | `game_init` (by address: zeroes the 30 slots, then stores each `C_cartridge` it builds) | `game_clear_cartridges`, `game_done`, `game_done_intern`, `game_init`, `game_new_cartridge` | §4l |
| +0x5b4 | `cartridge_b_cursor` | H | `game_init`, `game_new_cartridge` | `game_new_cartridge` | §4l |
| +0x5b8 | `bullet_holes` | H | `game_init` (by address: zeroes the 100 slots, then stores each `dUra_fiRe` frame it creates), `game_done_intern` (by address: 0 after releasing each frame) | `game_clear_frame_from_bullet_holes`, `game_done`, `game_done_intern`, `game_init`, `game_tick_shoot` | `game_init` creates them, `game_tick_shoot` places one per wall hit, `game_clear_frame_from_bullet_holes` detaches those on a frame |
| +0x748 | `bullet_hole_cursor` | H | `game_init`, `game_tick_shoot` | `game_tick_shoot` | - |
| +0x74c | `bullet_hole_materials` | H | `game_ctor`, `game_done_intern` | `game_done_intern`, `game_init`, `game_tick_shoot` | `game_tick_shoot` indexes it with the surface table's hole class (`g_surface_material_table + 0x3c`, stride `0x170`), falling back to 0 |
| +0x75c | `projectors` | H | `game_ctor` (by address: zeroes each record's `coll`), `game_init` (each record's `projector` and `registered`), `game_create_track_effect`, `game_create_track_effect_between`, `game_tick` (§2j) | `game_clear_projectors_for_frame`, `game_create_track_effect`, `game_create_track_effect_between`, `game_ctor`, `game_done`, `game_done_intern`, `game_tick` | `structs/blood.md` §7a |
| +0x29bc | `decal_materials` | H | `game_init` | `game_init`, `game_scratch_add` | loaded by name in `game_init`: `2blood.bmp`, `2flek.bmp`, `2dirbet.bmp`, `2dirdre.bmp`, `2dirple.bmp`, `2dirsklo.bmp`, `2bfootprint.bmp`, `2blood3.bmp`, `2dogfoot.bmp`, `skrob.tga`; `structs/blood.md` §7 |
| +0x29e4 | `decal_cursor` | H | `game_init`, `game_projector_advance_ring_cursor` | `game_create_track_effect`, `game_create_track_effect_between`, `game_done_intern`, `game_projector_advance_ring_cursor`, `game_tick` | wraps above 99; `game_init` resets it |
| +0x29e8 | `decals_growing` | H | `game_create_track_effect`, `game_create_track_effect_between`, `game_init`, `game_tick` | `game_create_track_effect`, `game_create_track_effect_between`, `game_tick` | gates the grow loop; `structs/blood.md` §7c |
| +0x29ec | `using_object` | H | `game_ctor` | `game_box_dest_set_ou`, `game_box_repose_dest_player`, `game_done`, `game_get_using_object`, `game_init`, `game_save_game_get_size`, `game_save_game_load` and 7 more | - |
| +0x2a0c | `player_speed` | H | `game_init`, `game_tick` | `game_tick`, `pedestrian_activate` | `game_tick`: the distance moved since `player_prev_pos` times `1000 / dt`; read by `pedestrian_activate` |
| +0x2a10 | `player_prev_pos` | H | `game_init`, `game_tick` | `game_init`, `game_tick` | falls back to the scene camera's in `game_init` when there is no player |
| +0x2a1c | `crowd_proxy` | H | `game_done`, `game_init` | `game_done`, `actor_is_possible_coll_all`, `pedestrian_create_model`, `pedestrian_set_model`, `pumper_hit`, `pumper_on_collision`, `railway_hit` and 1 more | built by `pedestrian_crowd_ctor` in `game_init`; `structs/pedestrian.md` §8c |
| +0x2a20 | `impact_sounds` | H | `game_init` | `game_done`, `game_done_intern`, `game_init`, `game_tick_shoot` | `game_init` creates them; `game_tick_shoot` starts the first that is not playing, with the shot's sound id |
| +0x2a48 | `shoot_lights` | H | `game_init` | `game_add_shoot_light`, `game_done`, `game_done_intern`, `game_init`, `game_tick` | §4a; `structs/lightness.md` §1b |
| +0x2ac0 | `flyby_sounds` | H | `game_init` | `game_add_shoot_light`, `game_done`, `game_done_intern`, `game_init`, `game_new_shoot`, `game_tick` | ⁴ |
| +0x2ad8 | `player_in_death_volume` | H | `game_init`, `car_on_body_surface_contact`, `human_movement_intern` | `game_tick`, `race_game_loop`, `race_game_loop_net` | material 40 (`def-files-docs/materialy.md`); `human_movement_intern` (on foot) and `car_on_body_surface_contact` (a car the player drives) set it, each also raising the water-fall script event; `game_init` clears it; `game_tick` then opens the game-over screen, and `race_game_loop` / `race_game_loop_net` read it |
| +0x2ad9 | `clear_differences_request` | H | `game_init`, `game_request_clear_differences`, `game_tick`, `race_winners_replay_end` | `game_tick` | `game_request_clear_differences` and `race_winners_replay_end` set it; sole reader `game_tick` |
| +0x2adc | `diff_added_frames` | H | `game_clear_differences`, `game_ctor`, `game_load_differences` | `game_clear_differences`, `game_init` | the frames `game_load_differences` built and linked into the scene; `game_clear_differences` deletes each frame's actor, frees its placement and takes the frame out of the scene (§4h) |
| +0x2aec | `diff_added_actors` | H | `game_ctor`, `game_dtor`, `game_load_differences` | `game_clear_differences`, `game_init`, `game_load_differences` | `game_clear_differences` deletes every actor of it (§4h) |
| +0x2afc | `global_programs` | H | `game_clear_differences`, `game_ctor`, `game_dtor` | `game_clear_differences`, `game_global_program_run`, `game_load_differences` | scanned by name in `game_global_program_run` |
| +0x2b0c | `frame_time_ms` | H | `game_init`, `game_save_game_load`, `game_tick` | `game_get_frame_time_ms`, `game_lightness_tick`, `game_save_game_save`, `game_tick`, `ai_action_new_back_look`, `ai_action_new_barricade_fight`, `ai_action_new_blocked_way` and 102 more | `game_get_frame_time_ms`, 148 callsites |
| +0x2b10 | `tick_stamp` | H | `game_init`, `game_save_game_load`, `game_tick` | `game_save_game_save`, `game_tick` | `game_tick` increments it and compares it with the `S_invisible_actor` stamps |
| +0x2b14 | `frame_counter` | H | `game_init`, `game_tick` | `game_get_frame_count`, `game_tick`, `police_manager_register_collision_offence_for_target`, `police_manager_tick` | `game_init` zeroes it; `game_tick` increments it and runs its staged start-up switch while it is below 7 |
| +0x2b18 | `pursuit_events` | H | `game_ctor` | `game_done`, `game_get_pursuit_event_manager`, `game_init`, `game_invalidate_actor`, `game_save_game_get_size`, `game_save_game_load`, `game_save_game_save` and 8 more | - |
| +0x2b3c | `sound_arena` | H | `game_init` (by address, `sound_arena_init`), `game_invalidate_actor` (by address, `sound_arena_invalidate_actor`), `game_new_shoot` (by address, `sound_arena_emit_sound`), `human_notify_callback` (by address, `sound_arena_emit_sound`), `race_winners_replay_end` (by address, `sound_arena_init`) | `game_get_sound_arena` (returns the address), `entity_ai` (by address, `sound_arena_update_entity_sound_contacts`), `game_invalidate_actor`, `game_new_shoot`, `human_notify_callback` | `structs/sound_arena.md` |
| +0x2d44 | `night_mission` | H | `game_init`, `game_set_night_mission` | `actor_shot_set_weapon`, `car_deactivate`, `car_game_init`, `car_reset`, `car_reset_seats_and_routes`, `car_update`, `hud_load_speedometer_odometer_texture` and 6 more | ⁵ |
| +0x2d48 | `script_events` | H | `game_init`, `game_invalidate_actor`, `game_set_player_fire_event`, `game_set_player_horn_event`, `game_set_player_water_fall_event`, `game_set_timer_timeout_event` | `game_invalidate_actor`, `game_raise_horn_script_event`, `game_raise_player_fire_script_event`, `game_raise_timer_timeout_script_event`, `game_raise_water_fall_script_event` | the four `game_raise_*_script_event` functions |
| +0x2d68 | `npc_kill_events` | H | `game_ctor` | `game_add_npc_kill_event`, `game_do_npc_kill_event`, `game_done`, `game_init` | matched by a killed entity's model name |
| +0x2d78 | `driver_men` | H | `game_ctor`, `game_del_driver_man` | `game_add_driver_man`, `game_del_driver_man`, `game_done`, `game_init`, `game_tick` | - |
| +0x2d88 | `wheel_frames_car` | H | `game_init` | `game_tick`, `human_movement` | `game_init` stores the frame count converted to a float; `human_movement` multiplies by it; §8a |
| +0x2d8c | `wheel_frames_formula` | H | `game_init` | `game_tick` | `game_init` stores the frame count converted to a float; §8a |
| +0x2d90 | `blood` | H | `game_ctor` | `game_done`, `game_init`, `game_tick` | - |
| +0x2da0 | `police` | H | `game_ctor` | `game_destroy_control_signal`, `game_done`, `game_get_police_manager`, `game_init`, `game_tick`, `actor_is_possible_coll_all`, `car_ai_tick` and 26 more | address accessor `game_get_police_manager` |
| +0x2e4c | `stacks_clear_pending` | H | `game_character_stack_push`, `game_float_stack_push`, `game_init`, `game_inventory_stack_push` | `game_character_stack_push`, `game_float_stack_push`, `game_inventory_stack_push` | §4f; `game_init` sets it; the first push of any of the three stacks after that clears it and empties all three; `game_save_game_load` refills the stacks and leaves the byte alone |
| +0x2e50 | `inventory_stack` | H | `game_ctor`, `game_inventory_stack_clear` | `game_inventory_stack_clear`, `game_inventory_stack_pop`, `game_inventory_stack_push`, `game_save_game_get_size`, `game_save_game_load`, `game_save_game_save` | §4f |
| +0x2e60 | `character_stack` | H | `game_character_stack_clear`, `game_ctor` | `game_character_stack_clear`, `game_character_stack_pop`, `game_character_stack_push`, `game_save_game_get_size`, `game_save_game_load`, `game_save_game_save` | §4f |
| +0x2e70 | `float_stack` | H | `game_ctor`, `game_float_stack_clear`, `game_float_stack_pop` | `game_float_stack_clear`, `game_float_stack_pop`, `game_float_stack_push`, `game_save_game_get_size`, `game_save_game_load`, `game_save_game_save` | §4f |
| +0x2e80 | `boxes` | H | `game_box_del`, `game_ctor` | `game_box_add`, `game_box_count_owned`, `game_box_count_owned_on_car`, `game_box_del`, `game_box_dest_set_ou`, `game_box_get_enemy_dest`, `game_box_move_to_dest` and 5 more | §4k |
| +0x2e90 | `box_dests` | H | `game_ctor` | `game_box_dest_add`, `game_box_dest_count_owned`, `game_box_dest_set_ou`, `game_box_get_enemy_dest`, `game_box_move_to_dest`, `game_box_place_at_next_dest`, `game_box_repose_dest_player` and 2 more | §4k |
| +0x2ea0 | `carried_box_node` | H | `game_box_dest_set_ou`, `game_box_repose_dest_player`, `game_ctor` | `game_box_dest_set_ou`, `game_box_repose_dest_player`, `game_ctor` | an `S_using_object` (`structs/using_object.md` §1a): `game_ctor` zeroes the members a new record gets; `game_box_dest_set_ou` and `game_box_repose_dest_player` fill it (the destination frame and its world position, `radius_sq` 1.0, `item_id` 1, `action_code` `0x7db`, `self`) and register it with `using_object_add_object` |
| +0x2ed8 | `tearing_down` | H | `game_done`, `game_init` | `human_game_done`, `police_manager_free_temp_enemy` | `game_done` sets it, `game_init` clears it; `human_game_done` and `police_manager_free_temp_enemy` test it |
| +0x2edc | `race_state` | H | `game_init`, `game_set_race_state`, `game_tick` | `game_get_race_state`, `game_tick` | - |
| +0x2ee0 | `race_result_register` | H | `game_set_race_result_register`, `game_tick` | `game_tick` | - |
| +0x2ee4 | `death_queue` | H | `game_ctor`, `game_dtor`, `game_process_death_actors` | `game_death_actor_add`, `game_done`, `game_init`, `game_process_death_actors`, `game_tick` | `game_process_death_actors` forces each death |
| +0x2ef4 | `cheat_buffer` | H | `game_init` | `game_cheat_check` | holds 31 characters and a NUL: `game_cheat_check` clears it above 30 |
| +0x2f14 | `cheat_flags` | H | `game_cheat_check`, `game_init` | `game_car_gun_tick`, `game_cheat_check`, `gui_action_menu_ctor`, `gui_action_menu_tick`, `human_ai`, `human_do_castrate`, `human_do_hard_head` and 5 more | §2z; `basic/artifacts/cheats.md` |
| +0x2f18 | `media` | H | `game_ctor`, `game_media_destroy`, `game_media_destroy_all` | `game_init`, `game_media_apply_volumes`, `game_media_create`, `game_media_destroy`, `game_media_destroy_all`, `game_media_fade_all`, `game_media_get` and 8 more | the `game_media_*` family |
| +0x2f28 | `media_fades` | H | `game_ctor`, `game_media_destroy_all` | `game_init`, `game_media_destroy_all`, `game_media_fade`, `game_save_game_get_size`, `game_save_game_load`, `game_save_game_save`, `game_tick` | `game_media_fade` fills it, `game_tick` advances it; §2x |
| +0x2f38 | `media_id_next` | H | `game_init`, `game_media_create`, `game_save_game_load` | `game_media_create`, `game_save_game_save` | - |
| +0x2f40 | `talk_queue` | H | `game_ctor`, `game_human_talk_del`, `game_human_talk_stop_for_human` | `game_human_talk_add`, `game_human_talk_del`, `game_human_talk_stop_for_human`, `game_init` | §4g |
| +0x2f50 | `car_gun_frame` | H | `game_done`, `game_init` | `game_car_gun_tick`, `game_done` | `game_init` stores 0 and `game_done` releases it; no code creates it, so the tick's body past its gate never has a frame to work on (§3c `game_car_gun_tick`) |
| +0x2f54 | `unk_2f54` | U | `game_done`, `game_init` | `game_done` | two slots: `game_init` stores 0, `game_done` releases each through its vtable slot 0 and stores 0; no code creates them and `game_car_gun_tick` does not touch them (Open item 5) |
| +0x2f5c | `car_gun_muzzle_frames` | H | - | `game_car_gun_tick` | read through the walking pointer of the `+0x2f64` row; a set slot gives the origin and the frame argument of `game_new_shoot`; no store in `.text` |
| +0x2f64 | `car_gun_eject_frames` | H | no store in `.text` (`game_car_gun_tick` takes the address only to read) | `game_car_gun_tick` (by address: a pointer walking down from `+0x2f64` reads the four slots `+0x2f5c..+0x2f68`) | a set slot is the frame reference of `game_new_cartridge` |
| +0x2f6c | `car_gun_car` | H | `game_car_gun_tick`, `game_init` | `game_car_gun_tick` | the player's car the frame is linked under, at the height `size.y - pivot.y` |
| +0x2f70 | `car_gun_target` | H | `game_car_gun_tick`, `game_init` | `game_car_gun_tick` | the nearest car or active human of the scan; a human is aimed at 1.3 above its position |
| +0x2f74 | `car_gun_fire_timer` | H | `game_car_gun_tick` | `game_car_gun_tick` | reloads 330 ms |
| +0x2f78 | `car_gun_scan_timer` | H | `game_car_gun_tick` | `game_car_gun_tick` | reloads 20 ms without a target, 250 ms with one |
| +0x2f7c | `pending_actor_lookups` | H | `game_ctor`, `game_flush_pending_actor_lookups` | `game_flush_pending_actor_lookups`, `game_init`, `game_tick` | drained by `game_flush_pending_actor_lookups`; no code appends to it: the end pointer `+0x2f84` is stored only by `game_ctor` and the flush (§2y) |
| +0x2f8c | `script_particles` | H | `game_ctor`, `game_particle_destroy_all` | `game_particle_add`, `game_particle_destroy_all` | §3 `game_particle_add` |
| +0x2f9c | `heavy_init` | H | `game_init`, `game_set_heavy_init_enabled` | `game_done`, `game_init`, `game_tick` | `game_init` sets it to 1, the script command `noanimpreload` clears it through `game_set_heavy_init_enabled(0)`. While set, `game_init` loads the 43 shadow meshes (also gated on `load_pending`), preloads the animation tracks and builds the model-backed pools, and the racing life cycle runs: `racing_game_init` in `game_init`, `racing_game_done` in `game_done`, the race check in `game_tick`. While clear, `game_init` restores the saved sound volume instead |
| +0x2fa0 | `city_music_areas` | H | `game_ctor` | `game_done`, `game_load_city_music_areas`, `game_tick` | §4h |
| +0x2fb0 | `city_music_timer` | H | `game_init`, `game_tick` | `game_tick` | §4h |
| +0x2fb4 | `city_music_track` | H | `game_city_music_off`, `game_init`, `game_tick` | `game_tick` | §4h |
| +0x2fb8 | `city_music_pending` | H | `game_city_music_off`, `game_init`, `game_save_game_load`, `game_tick` | `game_city_music_off`, `game_save_game_save`, `game_tick` | §4h |
| +0x2fbc | `city_music_channel` | H | `game_city_music_off`, `game_init`, `game_save_game_load`, `game_tick` | `game_city_music_off`, `game_save_game_save`, `game_tick` | §4h |
| +0x2fc0 | `city_music_paused` | H | `game_city_music_set_paused`, `game_init` | `game_tick` | §4h |
| +0x2fc4 | `film_music` | H | `game_clear_film_music`, `game_ctor`, `game_media_destroy_all` | `game_clear_film_music`, `game_get_film_music`, `game_init`, `game_media_destroy_all`, `game_set_film_music` | §4h |
| +0x2fd4 | `game_over_delay` | H | `game_init`, `game_tick` | `game_tick` | `game_init` sets 7000; once the player actor's death latch `+0x5e` is set, `game_tick` counts it down, starts a 1000 ms fade and opens `gui_game_over` |
| +0x2fd8 | `scratches` | H | `game_init` | `game_init`, `game_scratch_add`, `game_scratch_delete_from_car`, `game_scratch_get_free_idx` | §4i; `game_init` zeroes the 400 dwords |
| +0x3618 | `scream_areas` | H | `game_ctor` | `game_done`, `game_init`, `game_load_scream_areas`, `game_tick`, `game_tick_scream_areas` | - |
| +0x3628 | `scream_mask` | H | `game_init`, `game_tick_scream_areas` | `human_new_scream` | read by `human_new_scream` |
| +0x362c | `scream_timer` | H | `game_init`, `game_tick` | `game_tick` | `game_init` seeds it -1 |
| +0x3630 | `weather_on` | H | `game_apply_weather_detail_level`, `game_init` | `game_apply_weather_detail_level`, `game_done` | `game_init` reads it from the scene, `game_done` writes it back |
| +0x3634 | `weather_particles` | H | `game_apply_weather_detail_level`, `game_init` | `game_apply_weather_detail_level`, `game_done` | `game_init` reads it from the scene, `game_done` writes it back |
| +0x3638 | `rain_buffer_size` | H | `game_init`, `game_set_rain_buffer_size`, `game_tick` | `game_tick` | the script command `weather_preparebuffer`; the Mafia Script Data Reference gives 3900. `game_init` seeds -1; `game_tick` applies it and resets it |
| +0x363c | `load_pending` | H | `game_init`, `mission_open` | `game_init` | `mission_open` sets it, `game_init` clears it at its tail |
| +0x363d | `done_intern_ran` | H | `game_done_intern`, `game_init`, `mission_open` | `game_done_intern` | `mission_open` clears it |
| +0x363e | `score_on` | H | `game_freeride_score_set_on`, `game_init`, `game_save_game_load` | `game_freeride_score_set`, `game_save_game_load`, `game_save_game_save`, `car_explosion`, `human_hit`, `human_hit_in_car`, `pedestrian_hit` | §4j |
| +0x3640 | `score` | H | `game_freeride_score_set`, `game_init`, `game_save_game_load` | `game_freeride_score_get`, `game_freeride_score_set_on`, `game_save_game_load`, `game_save_game_save`, `car_explosion`, `human_hit`, `human_hit_in_car` and 2 more | §4j |
| +0x3644 | `camera_rot_repair` | H | `game_init`, `game_set_camera_rot_repair`, `game_tick` | `game_tick` | `game_set_camera_rot_repair` sets it with the saved quaternion |
| +0x3648 | `camera_rot_saved` | H | `game_set_camera_rot_repair` (the scene camera's rotation `+0x9c..+0xa8`) | `game_tick` (by address: the start-up step 6 copies it back into the scene camera while `camera_rot_repair` is set) | - |
| +0x3658 | `sheet_model` | H | `game_done_intern`, `game_init` | `game_done`, `game_done_intern`, `game_init` | `game_init` opens it; `game_done` hides it, `game_done_intern` releases it |
| +0x365c | `unk_365c` | U | `game_done_intern`, `game_init` | `game_done`, `game_done_intern` | `game_init` and `game_done_intern` store 0; `game_done` hides it and `game_done_intern` releases it when set; no non-zero writer (Open item 4) |
| +0x3660 | `money_model` | H | `game_done_intern`, `game_init` | `game_done`, `game_done_intern`, `game_init` | `game_init` opens it; `game_done` hides it, `game_done_intern` releases it |
| +0x3664 | `heal_cheat_ready` | H | `game_cheat_check`, `game_init` | `game_cheat_check` | `game_init` arms it; `game_cheat_check` tests and clears it |
| +0x3665 | `script_mission_end` | H | `game_init`, `game_set_script_mission_end` | `garage_game_done` | set by BOTH `endofmission` and `change_mission`; sole reader `garage_game_done` |
| +0x3668 | `car_sound_budget` | H | `game_init`, `game_tick` | `game_tick`, `game_tick_car_sound_manager` | `game_tick_car_sound_manager` |
| +0x366c | `car_sound_timer` | H | `game_tick` | `game_tick` | `game_tick` |
| +0x3670 | `car_sound_pass` | H | `game_tick` | `game_tick`, `game_tick_car_sound_manager` | `game_tick` |
| +0x3674 | `name_free_list` | H | `game_add_frame_for_shadow`, `game_ctor`, `game_done` | `game_add_frame_for_shadow`, `game_done`, `game_init` | `game_add_frame_for_shadow` interns and returns shadow names here |
| +0x3684 | `item_blur_on` | H | `game_init`, `game_item_blur_done` | `game_tick` | - |
| +0x3688 | `item_blurs` | H | `game_ctor`, `game_item_blur_del_item`, `game_item_blur_done` | `game_init`, `game_invalidate_actor`, `game_item_blur_del_item`, `game_item_blur_done`, `game_item_blur_tick` | - |

¹ The gate is `mov al,[ebp+0x164]; test al,al; je` around the `game_lightness_tick` call, followed
by the non-empty `+0x154 .. +0x158` test and `*(g_act_mission + 0x9c) >= 2` (a detail-quality setting).
Ghidra folds the displacement into a register, so the read is only visible in the disassembly. No
shipped mission script uses `actorlightnesson`, so the byte stays clear in the retail game. The
Mafia Script Data Reference describes the command as making *no* light affect the player; the
binary shows only the gate above.

² Type 0 = wall alarm box, type 1 = phone booth; the list is consumed by the cop "go to a phone and
call HQ" task. `game_get_telephone_list` (`0x00425570`) is the address accessor; its result is
handed to `vec_count_stride_0xc`, which reads `obj+4` / `obj+8` — that is what pins the header to
this pair.

⁵ `game_set_night_mission` is the engine side of the script command `game_nightmission nn`, which
the Mafia Script Data Reference describes as *"sets the mission to day or night; **affects car
lights** and shooting effects"*. `car_update`'s automatic-headlight block (`C_Vehicle+0x19c` bit
`0x10000`, `structs/vehicle.md` §2a) runs only while the flag is non-zero; `car_deactivate` clears a
car's script light-override bits only while it is ZERO. The shooting-effects half is
`actor_shot_set_weapon`, which adds 1 to a timed weapon-fire actor's shot effect id on a night
mission (`structs/actor_shot.md` §2).

³ `game_tick`'s `(*(code *)**(game + 0x2bc))(dt)` is a
virtual call to its slot 0, `random_morph_tick`: the code is `mov edx,[ebp+0x2bc]; lea
ecx,[ebp+0x2bc]; push dt; call [edx]`. `game_ctor` writes the vtable through ECX inside
`random_morph_ctor`, so a scan for stores at the displacement `+0x2bc` finds none.

⁴ `game_new_shoot` computes the point of the shot line nearest the scene camera; when that point lies more than 12
units along the shot and within 10 units of the camera, it starts the first idle `S_flyby_sound` there after a
delay from the shot's travel time. `game_tick` starts the sound when the delay runs out.

`game_stop_3d_sound` either stops a sound at once or sets `S_sound_play.fading`, stores `-rate` and increments
`+0x310`.

**`g_net_session` (`DAT_006efbac`) is a different object.** It is a separate top-level
session object (`structs/net_session.md`). The two are easy to conflate because **both have
a meaningful `+0x204`**: here it is the taxi-passenger candidate actor, there it is the
embedded local multiplayer player record.

### 2a. `S_frame_placement`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_add_frame_placement` | `game_del_frame_placement` | - |
| +0x04 | `pos` | H | `game_add_frame_placement` | `game_done`, `game_init` | - |
| +0x10 | `rotation` | H | `game_add_frame_placement` | `game_done`, `game_init` | the frame's quaternion is materialised first when it is stale |
| +0x20 | `scale` | H | `game_add_frame_placement` | `game_done`, `game_init` | - |
| +0x2c | `parent` | H | `game_add_frame_placement` | `game_done`, `game_init` | - |
| +0x30 | `enabled` | H | `game_add_frame_placement` | `game_done`, `game_init` | - |

### 2b. `S_box_placement_temp`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_box_save_info` | `game_box_del_info`, `game_box_get_start_point`, `game_box_restore_info` | - |
| +0x04 | `parent` | H | `game_box_save_info` | `game_box_get_start_point`, `game_box_restore_info` | - |
| +0x08 | `pos` | H | `game_box_save_info` | `game_box_get_start_point`, `game_box_restore_info` | - |
| +0x14 | `dir` | H | `game_box_save_info` | `game_box_restore_info` | - |

### 2c. `S_invisible_actor`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `actor` | H | `game_add_invisible_actor` | `game_tick` | - |
| +0x04 | `stamp` | H | `game_add_invisible_actor`, `game_tick` | `game_tick` | the toggle waits for more than 5 ticks |

### 2d. `S_semaphore`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_enum_traffic_signal_frames` | `game_del_traffic_signal` | - |
| +0x04 | `red` | H | `game_enum_traffic_signal_frames` | `game_free_traffic_signals`, `game_tick_traffic_signals` | - |
| +0x08 | `orange` | H | `game_enum_traffic_signal_frames` | `game_free_traffic_signals`, `game_tick_traffic_signals` | - |
| +0x0c | `green` | H | `game_enum_traffic_signal_frames` | `game_free_traffic_signals`, `game_tick_traffic_signals` | - |
| +0x10 | `phase` | H | `game_enum_traffic_signal_frames` | `game_tick_traffic_signals` | - |

### 2e. `S_RoadBlock`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `node` | H | `game_collect_road_block_node` | `police_manager_spawn_roadblock_in_neighbor_cells` | - |
| +0x02 | `engaged` | H | `game_collect_road_block_node`, `police_manager_tick` | `actor_is_possible_coll_all`, `police_manager_tick` | `game_collect_road_block_node` stores 0; `police_manager_tick` sets it when a roadblock-eligible suspect is within 150 units of the node and calls `police_manager_spawn_roadblock_in_neighbor_cells`, and clears it when none is within 200; `actor_is_possible_coll_all` collides only against engaged records, 40 units around the node |
| +0x04 | `actors` | H | `game_collect_road_block_node`, `police_manager_spawn_roadblock_in_neighbor_cells`, `police_manager_tick` | `police_manager_tick` | `police_manager_spawn_roadblock_in_neighbor_cells` appends each car and officer it deploys from the pools; on disengage `police_manager_tick` sends every kind-`0x1b` entry event `0x13`, hands it to `police_manager_npc_list_push` and empties the vector |

### 2f. `S_Telephone`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_collect_telephone`, `game_load_telephones` | `game_telephone_list_contains_key`, `game_telephone_list_erase_by_key`, `ai_action_new_police_call`, `entity_action_tick` | 0 for a device of the city cache: the two task readers then take the position from `cache_instance` |
| +0x04 | `cache_instance` | H | `game_collect_telephone`, `game_load_telephones` | `ai_action_new_police_call`, `entity_action_tick` | `game_collect_telephone` stores 0; the second half of `game_load_telephones` stores the alarm or phone-box instance of the city cache (`structs/cache_base_block.md`); read as `[+0x04] + 0x08`, the instance's `pos`, in `entity_action_tick` case `0x28` |
| +0x08 | `type` | H | `game_collect_telephone`, `game_load_telephones` | - | no reader in the 6 callsites of `game_telephone_at` or in `ai_action_new_police_call` |

### 2g. `S_shoot`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `pos` | H | `game_new_shoot`, `game_tick_shoot` | `game_tick_shoot` | - |
| +0x0c | `velocity` | H | `game_new_shoot` | `game_tick_shoot` | - |
| +0x18 | `range` | H | `game_new_shoot`, `game_tick_shoot` | `game_tick_shoot` | - |
| +0x1c | `prev_pos` | H | `game_new_shoot`, `game_tick_shoot` | `game_tick_shoot` | - |
| +0x28 | `damage` | H | `game_new_shoot` | `game_tick_shoot` | - |
| +0x2c | `owner` | H | `game_new_shoot` | `game_invalidate_actor`, `game_tick_shoot` | - |
| +0x30 | `hit_frame` | H | `game_new_shoot` | `game_tick_shoot` | set when the struck face's material is `0x18` (glass) or `0x29` and its flags have `0xc0`; `game_tick_shoot` breaks it with `glass_shatter_break_frame` (`0x18`) or `crash_obj_create_from_model` (`0x29`) |
| +0x34 | `hit_face` | H | `game_new_shoot` | `game_tick_shoot` | the face record: dword flags `0x20` broken, `0x20000` control sign; byte `+3` the material |
| +0x38 | `hit_pos` | H | `game_new_shoot` | `game_tick_shoot` | - |
| +0x44 | `hit_normal` | H | `game_new_shoot` | `game_tick_shoot` | - |
| +0x50 | `hit` | H | `game_new_shoot` | `game_tick_shoot` | 1 when a face was struck |
| +0x54 | `impact_sound` | H | `game_new_shoot` | `game_tick_shoot` | `g_surface_material_table + 0x24`, stride `0x170` |
| +0x58 | `material` | H | `game_new_shoot` | `game_tick_shoot` | - |

### 2h. `S_hydrant`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `particle_guid` | H | `game_add_hydrant` | `game_del_all_hydrants`, `game_tick` | - |
| +0x04 | `covered` | H | `game_add_hydrant`, `game_tick` | `game_tick` | - |
| +0x08 | `pos` | H | `game_add_hydrant` | `game_tick` | - |
| +0x14 | `sound` | H | `game_add_hydrant` | `game_del_all_hydrants`, `game_tick` | - |

### 2i. `S_sound_play`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `sound` | H | `game_play_3d_sound` | `game_clear_effects`, `game_stop_3d_sound`, `game_tick` | - |
| +0x04 | `handle` | H | `game_play_3d_sound` | `game_stop_3d_sound` | - |
| +0x08 | `target_volume` | H | `game_play_3d_sound_by_name` | `game_tick` | - |
| +0x0c | `fade_rate` | H | `game_play_3d_sound_by_name`, `game_stop_3d_sound` | `game_tick` | - |
| +0x10 | `fading` | H | `game_stop_3d_sound` | `game_tick` | - |
| +0x14 | `last_position` | H | `game_tick` | `game_tick` | - |

### 2j. `S_decal_projector`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `projector` | H | `game_init` | `game_clear_projectors_for_frame`, `game_create_track_effect`, `game_done_intern` | - |
| +0x04 | `coll` | H | `game_ctor` | `game_create_track_effect` | - |
| +0x4c | `registered` | H | `game_create_track_effect` | `game_create_track_effect` | - |
| +0x4d | `growing` | H | `game_create_track_effect`, `game_tick` | `game_tick` | - |
| +0x50 | `grow_scale` | H | `game_create_track_effect`, `game_tick` | `game_tick` | - |
| +0x54 | `owner` | H | `game_create_track_effect` | `game_clear_projectors_for_frame` | - |

### 2k. `S_shoot_light`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `light` | H | `game_init` | `game_add_shoot_light`, `game_tick` | - |
| +0x04 | `lifetime_ms` | H | `game_add_shoot_light`, `game_init`, `game_tick` | `game_add_shoot_light`, `game_tick` | - |
| +0x08 | `flags` | H | `game_add_shoot_light` | `game_tick` | - |

### 2l. `S_flyby_sound`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `sound` | H | `game_init` | `game_new_shoot`, `game_tick` | - |
| +0x04 | `delay_ms` | H | `game_init`, `game_new_shoot`, `game_tick` | `game_new_shoot`, `game_tick` | - |

### 2m. `S_script_event`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `program` | H | `game_set_player_fire_event`, `game_set_player_horn_event`, `game_set_player_water_fall_event`, `game_set_timer_timeout_event` | `game_raise_horn_script_event`, `game_raise_player_fire_script_event`, `game_raise_timer_timeout_script_event`, `game_raise_water_fall_script_event` | - |
| +0x04 | `event` | H | `game_set_player_fire_event`, `game_set_player_horn_event`, `game_set_player_water_fall_event`, `game_set_timer_timeout_event` | `game_raise_horn_script_event`, `game_raise_player_fire_script_event`, `game_raise_timer_timeout_script_event`, `game_raise_water_fall_script_event` | - |

### 2n. `S_npc_kill_event`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `program` | H | `game_add_npc_kill_event` | `game_do_npc_kill_event` | - |
| +0x04 | `event` | H | `game_add_npc_kill_event` | `game_do_npc_kill_event` | - |
| +0x08 | `name` | H | `game_add_npc_kill_event` | `game_do_npc_kill_event` | - |

### 2o. `S_driver_man`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `model` | H | `game_add_driver_man` | `game_del_driver_man`, `game_tick` | - |
| +0x04 | `anim_set` | H | `game_add_driver_man` | `game_del_driver_man`, `game_tick` | - |
| +0x08 | `car` | H | `game_add_driver_man` | `game_tick` | - |
| +0x0c | `formula` | H | `game_add_driver_man` | `game_tick` | - |

### 2p. `S_box_placement`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_box_add` | `game_box_add`, `game_box_del` | - |
| +0x04 | `use_node` | H | `game_box_add` | `game_box_del`, `game_box_recompile` | a `0x38`-byte `S_using_object` (`structs/using_object.md` §1a) allocated by `game_box_add`; `game_box_recompile` fills it (the frame and its world position, `radius_sq` 0.5, `item_id` 1, `action_code` `0x7da`, `self`) and registers it, or takes it out of the registry |
| +0x08 | `coll` | H | `game_box_add`, `game_box_recompile_for_car` | `game_box_del`, `game_box_recompile_for_car` | type `0x40`, the frame, flags `0x1000`, the world centre, radius = the largest scale times the bound radius, the six bound dwords, and pointers to the frame's world matrix and to `inv_world`; entered with `collision_dyn_insert`, removed with `collision_dyn_erase` |
| +0x50 | `inv_world` | H | `game_box_add`, `game_box_recompile_for_car` | - | no reader in the class: `coll` holds its address at record `+0x48` |
| +0x90 | `owner` | H | `game_box_add` | `game_box_count_owned`, `game_box_count_owned_on_car`, `game_box_recompile` | - |
| +0x94 | `usable` | H | `game_box_add`, `game_box_place_at_next_dest`, `game_box_recompile_for_car` | `game_box_recompile` | 1 from `game_box_add`; cleared while the box is loaded on a car (`game_box_recompile_for_car` with 0, `game_box_place_at_next_dest`) and set by `game_box_recompile_for_car` with 1 |

### 2q. `S_box_dest`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_box_dest_add` | `game_box_dest_set_ou`, `game_box_get_enemy_dest` | - |
| +0x04 | `owner` | H | `game_box_dest_add` | `game_box_dest_count_owned` | - |

### 2r. `S_i_show`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `show` | H | `game_media_create` | `game_media_destroy`, `game_media_get` | - |
| +0x04 | `id` | H | `game_media_create` | `game_media_destroy`, `game_media_get` | - |
| +0x08 | `flags` | H | `game_media_create`, `game_media_set_loop` | `game_save_game_get_size`, `game_save_game_save`, `game_tick` | `game_media_create` stores its creation argument; bit `0x2` = the entry is left out of the save |
| +0x0c | `name` | H | `game_media_open` | `game_media_destroy` | - |
| +0x10 | `volume` | H | `game_media_store_volume` | `game_media_apply_volumes` | - |

### 2s. `S_enemy_talk`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `human` | H | `game_human_talk_add` | `game_human_talk_stop_for_human` | - |
| +0x04 | `speech_id` | H | `game_human_talk_add` | `game_human_talk_del` | - |

### 2t. `S_scratch`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `scratch_frame` | H | `game_scratch_add` | `game_scratch_delete_from_car` | - |
| +0x04 | `car` | H | `game_scratch_add`, `game_scratch_delete_from_car` | `game_scratch_delete_from_car`, `game_scratch_get_free_idx`, `game_scratch_recompile` | - |
| +0x08 | `source_frame` | H | `game_scratch_add` | `game_scratch_recompile` | - |
| +0x0c | `age` | H | `game_scratch_add`, `game_scratch_get_free_idx` | `game_scratch_add`, `game_scratch_get_free_idx` | the name is the field's use as a tie-break: the least reused of three nearby marks is taken |

### 2u. `S_scream_area`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `frame` | H | `game_collect_scream_area` | `game_tick_scream_areas` | - |
| +0x04 | `mask` | H | `game_collect_scream_area` | `game_tick_scream_areas` | - |

### 2v. `S_item_blur`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `actor` | H | - | `game_invalidate_actor`, `game_item_blur_tick` | the tick gates on its `in_camera_sector` (`+0x5f`) and its frame (`+0x68`) |
| +0x04 | `frame` | H | - | `game_item_blur_tick` | - |
| +0x08 | `sub_items` | H | - | `game_item_blur_del_item`, `game_item_blur_tick` | `game_item_blur_del_item` frees the array |
| +0x0c | `sub_item_count` | H | - | `game_item_blur_del_item`, `game_item_blur_tick` | - |
| +0x10 | `prev_pos` | H | `game_item_blur_tick` | `game_item_blur_tick` | - |
| +0x1c | `speed` | H | `game_item_blur_tick` | `game_item_blur_tick` | the streaks show at 1.0 or more |
| +0x20 | `shown` | H | `game_item_blur_tick` | `game_item_blur_tick` | - |
| +0x24 | `max_length` | H | - | `game_item_blur_tick` | the streak length is the tick's speed times `speed` times 0.01, capped here |

No code appends a record: the vector's end `+0x3690` is stored only by `game_ctor`, `game_item_blur_del_item` and
`game_item_blur_done`. So the five fields with no writer are never filled (Open item 5).

### 2w. `S_paused_sound`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `sound` | H | `game_pause_scene_sounds_recursive` | `game_play_all_sounds` | a playing sound frame (visual type 4) |
| +0x04 | `volume` | H | `game_pause_scene_sounds_recursive` | `game_play_all_sounds` | the sound's `volume` (`structs/i3d_frame.md`); put back through its vtable `+0x68` |
| +0x08 | `unk_08` | U | `game_pause_scene_sounds_recursive` | `game_play_all_sounds` | the sound's `+0x188`; put back through its vtable `+0x64`; the meaning is inside `LS3DF.dll` (Open item 5) |
| +0x0c | `looped` | H | `game_pause_scene_sounds_recursive` | `game_play_all_sounds` | the sound's byte `+0x164`; put back through its vtable `+0x6c`, the slot `game_add_hydrant` calls with 1 on its looped sound |
| +0x10 | `play_position` | H | `game_pause_scene_sounds_recursive` | `game_play_all_sounds` | the result of the sound's vtable `+0x80`; put back through `+0x70` once the sound is switched on and playing |

### 2x. `S_i_show_fade`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `media_id` | H | `game_media_fade`, `game_save_game_load` | `game_media_fade`, `game_save_game_save`, `game_tick` | `game_media_fade` erases the id's older record, sets the volume to `from_volume` and appends `{id, from, to, 0, duration}` |
| +0x04 | `from_volume` | H | `game_media_fade`, `game_save_game_load` | `game_save_game_save`, `game_tick` | - |
| +0x08 | `to_volume` | H | `game_media_fade`, `game_save_game_load` | `game_save_game_save`, `game_tick` | when a fade ends with a target below 0.005 and falling, `game_tick` stops the media; if the id is `city_music_channel` it destroys the media and stores -1 there |
| +0x0c | `elapsed_ms` | H | `game_media_fade`, `game_save_game_load`, `game_tick` | `game_save_game_save`, `game_tick` | - |
| +0x0e | `duration_ms` | H | `game_media_fade`, `game_save_game_load` | `game_save_game_save`, `game_tick` | - |

### 2y. `S_find_actor`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `program` | H | - | `game_flush_pending_actor_lookups` | ECX of `program_find_actor_later`; a record with 0 is skipped |
| +0x04 | `actor_reg` | M | - | `game_flush_pending_actor_lookups` | the first argument of `program_find_actor_later`; named from the argument order, the callee's body was not read |
| +0x08 | `guid` | M | - | `game_flush_pending_actor_lookups` | its second argument |

No code appends a record (the `pending_actor_lookups` row of §2), so no field has a writer.

### 2z. `cheat_flags` (`+0x2f14`)

`game_init` clears the word. `game_cheat_check` is the only writer located, and it toggles only
three of the nine bits it holds.

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | the player and every human with `+0x5fc` == 2 take no hits; no writer located (Open item 2) | `human_hit`, `human_do_castrate`, `human_do_hard_head` |
| `0x2` | `gui_action_menu_ctor` / `_tick` do not auto-take a single option; no writer located (Open item 2) | `gui_action_menu_ctor`, `gui_action_menu_tick` |
| `0x4` | the S&W model 27 Magnum (id 7) takes its own branch when fired; no writer located (Open item 2) | `human_shooting`; `structs/grenade.md` |
| `0x8` | enables the car-mounted gun of `game_car_gun_tick`; no writer located (Open item 2) | `game_car_gun_tick` |
| `0x10` | `BOXER`: big hands | `game_cheat_check`, `player_set_hand_scale` |
| `0x20` | any car can be taken: `player_try_use_car` skips its refusal when `garage_can_steal_car_model` says no; no writer located (Open item 2) | `player_do_action_button_on_foot`, `player_try_use_car` |
| `0x40` | `police_manager_tick` skips its offence block while set; no writer located (Open item 2) | `police_manager_tick` |
| `0x80` | `PARNIK`: the steamboat figures | `game_cheat_check`, `bridge_set_steamboat_figures` |
| `0x100` | `MUNICAK`: infinite ammo | `game_cheat_check`, `human_ai` |

## 3. Method family

177 of 177 PS2 `C_game` methods are placed: on a PC function or as an `inlined` row directly after the
PC function that absorbed it, or in the PS2-only table of §8b. One row is not a `C_game` method and its Role
cell says whose it is: `game_telephone_at`. The destructors of the embedded sub-objects carry their own
classes' names (§5).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005dc160 | `crt_static_init_guard_005dc160` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer guard |
| 0x005dc180 | `crt_atexit_register_005dc180` | the translation unit | - | - | - | registers the guard's teardown |
| 0x005dc190 | `console_log_static_init` | `g_console_log` | - | - | - | the unit's first object, `.CRT$XCU` (§7) |
| 0x005dc1a0 | `console_log_static_ctor` | `g_console_log` | - | - | - | its constructor call |
| 0x005dc1b0 | `console_log_register_atexit_dtor` | `g_console_log` | - | - | - | its `atexit` registration |
| 0x005dc1c0 | `console_log_static_dtor` | `g_console_log` | - | - | - | its destructor call |
| 0x005dc1d0 | `console_overlay_static_init` | `g_console_overlay` | - | - | - | the unit's second object, `.CRT$XCU` (§7) |
| 0x005dc1e0 | `console_overlay_static_ctor` | `g_console_overlay` | - | - | - | its constructor call |
| 0x005dc1f0 | `console_overlay_register_atexit_dtor` | `g_console_overlay` | - | - | - | its `atexit` registration |
| 0x005dc200 | `console_overlay_static_dtor` | `g_console_overlay` | - | - | - | its destructor call |
| 0x00606750 | `game_ctor` | instance | - | - | - | The constructor (§5); the PS2 inlines it into the mission init. |
| 0x00606e20 | `game_dtor` | instance | - | - | - | The destructor (§5); the PS2 inlines it into the mission uninit. |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425510 | `game_get_camera` | - | - | - | `return this+0x4c`, the embedded `G_Camera` (§6); 13 callsites. |
| 0x00425520 | `game_get_active_actor_list` | - | - | - | `return this+0xec`, the active actor list (§4b); sole caller `entity_action_tick`. The inactive list has no accessor: every reader reads `+0x100` / `+0x104` inline. |
| 0x00425530 | `game_get_player_actor` | - | - | - | `return *(this+0xe4)`, the player's `C_actor`; 48 references. |
| 0x00425540 | `game_get_frame_time_ms` | - | - | - | `return *(this+0x2b0c)`, the frame timestamp in ms; 148 callsites. |
| 0x00425550 | `game_get_sound_arena` | - | - | - | `return this+0x2b3c`, the embedded `soundArena` (§6). |
| 0x00425560 | `game_get_police_manager` | - | - | - | `return this+0x2da0`, the embedded police manager (`structs/police_manager.md`). |
| 0x00425570 | `game_get_telephone_list` | - | - | - | `return this+0x1c4`, the callable-device list header (§2). |
| 0x00425580 | `game_freeride_score_get` | - | - | - | `return this+0x3640`. PC-only (an MSVC-emitted accessor) — the PS2 reads `+0x153c` inline. Consumed by the `FREERIDE_SCOREGET` / `SCOREADD` script commands. |
| 0x005c8b20 | `game_get_using_object` | - | - | - | `return this+0x29ec`, the embedded `C_using_object` (§6). |
| 0x005c8b30 | `game_request_clear_differences` | - | - | - | Sets the one-shot byte `+0x2ad9`; `game_tick` then drops the difference data (§4h). |
| 0x005c8b40 | `game_get_pursuit_event_manager` | - | - | - | `return this+0x2b18`, the embedded pursuit-event manager (`structs/pursuit_event.md`). |
| 0x005c8b50 | `game_get_frame_count` | - | - | - | Reads the frame counter `+0x2b14`. |
| 0x005c8b60 | `game_set_night_mission` | - | - | - | Writes the night-mission byte `+0x2d44` (script command `game_nightmission`). |
| 0x005c8bf0 | `game_set_mission_end_action` | - | - | - | Writes the mission-end action `+0x48` (1, 2 or 3). |
| 0x005c8c20 | `game_get_race_state` | - | - | - | Reads the mission-6 race state `+0x2edc`. |
| 0x005c8c30 | `game_set_race_result_register` | - | - | - | Writes `+0x2ee0`, the address of the script float register the race publishes its result into. |
| 0x005c8c40 | `game_set_actor_lightness_flag` | - | - | - | Writes the actor-lightness byte `+0x164` (script command `actorlightnesson`). |
| 0x005c8c50 | `game_set_heavy_init_enabled` | - | - | - | Writes the heavy-init byte `+0x2f9c`; its one caller, the script command `noanimpreload`, passes 0. |
| 0x005c8c60 | `game_set_rain_buffer_size` | - | - | - | Writes the rain buffer size `+0x3638` (script command `weather_preparebuffer`). |
| 0x005c8c70 | `game_set_script_mission_end` | - | - | - | Sets `+0x3665`: the mission is ending under script control. |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043ac50 | `game_telephone_at` | - | - | - | A method of the callable-device list, not of `C_game`: the address of entry `idx` (stride `0xc`) of the list `game_get_telephone_list` returns. |
| 0x005c8b70 | `game_set_timer_timeout_event` | - | - | - | Stores the timer-timeout script-event slot `+0x2d48` / `+0x2d4c`. |
| 0x005c8b90 | `game_set_player_fire_event` | - | - | - | Stores the player-fire script-event slot `+0x2d50` / `+0x2d54`. |
| 0x005c8bb0 | `game_set_player_horn_event` | - | - | - | Stores the player-horn script-event slot `+0x2d58` / `+0x2d5c`. |
| 0x005c8bd0 | `game_set_player_water_fall_event` | - | - | - | Stores the player-water-fall script-event slot `+0x2d60` / `+0x2d64`. |
| 0x005c8c00 | `game_set_race_state` | - | - | - | Writes the mission-6 race state `+0x2edc`: 2 for a non-zero argument, else 0. |
| 0x005dc210 | `game_set_human` | `SetHuman__6C_gameFP7C_actor` | 0x002ea010 | CONFIRMED | Accepts an actor of kind 2 or `0x1b`, stores it at `+0xe8` and hands it to the camera. |
| 0x005dc240 | `game_init` | `Init__6C_gameFbb` | 0x002ea0a0 | CONFIRMED | Per-scene STATE reset: freeride score, render flags, stop sounds, state vectors, `police_board_reset`, `+0x2fd4 = 7000`, the decal-texture loads. Called by `loadsave_savegame_load` after `mission_open`. |
| - | - | `PreloadAnims__6C_gameFv` | 0x00306510 | inlined | the heavy-init block's clip preload and base-track seed, statement for statement (`structs/anim_machine.md` §6) |
| 0x005df240 | `game_create_easter_egg_detectors` | - | - | - | Builds the two hidden easter-egg trigger volumes (`C_detector`) at hard-coded positions, each with its own script. |
| 0x005df610 | `game_enum_sectors_for_projector_lights` | `cbEnumSectors__Q26C_game18S_hlp$3427game_cppFP9I3D_frame` | 0x002ed870 | CONFIRMED | Scene-walk callback: registers every sector frame (type 5) for projector lighting. |
| 0x005df6b0 | `game_done` | `Done__6C_gameFv` | 0x002eda50 | CONFIRMED | Teardown behind the `+0x40` guard: stops sounds, tears down camera / police / garage / blood / particles, despawns pursuit assets, sniper off, clears the death queue and the temporary actors. Inlines most `Clear*` / `Del*` helpers. |
| 0x005e0780 | `game_done_intern` | `DoneIntern__6C_gameFv` | 0x002ef110 | CONFIRMED | The resource half of the teardown, once-gated on `+0x363d`; drains the cartridge pools. |
| 0x005e0ad0 | `game_change_act_state` | `ChangeActState__6C_gameFv` | 0x002ef680 | CONFIRMED | Clears the dirty flag `+0x10c`, then moves every actor whose act state left 0 from the active list `+0xec` to the inactive list `+0xfc`, and every actor whose state left 1 back, calling the actor virtual `+0x6c` on each move and dropping actors switched off (state 2) from both (§4b). Callers `game_init` ×2, `game_tick` ×2, `loadsave_savegame_load` |
| 0x005e0c20 | `game_tick` | `Tick__6C_gameFUi` | 0x002ef870 | CONFIRMED | The per-frame world tick: pursuit / zone / waypath / police ticks, plus seven helpers MSVC inlined here (§8a). |
| - | - | `ShootLightTick__6C_gameFUi` | 0x002ff640 | inlined | the shoot-light loop over `shoot_lights` |
| - | - | `TickEffects__6C_gameFUi` | 0x002f7860 | inlined | the per-frame walk over `sounds` |
| - | - | `TickCityMusic__6C_gameFUi` | 0x002f9430 | inlined | the `city_music_*` block |
| - | - | `SetActiveTrafficP__6C_gameFUi` | 0x002f23d0 | inlined | the traffic-generator pick; `game_init` runs a second copy |
| - | - | `TickDriverMan__6C_gameFv` | 0x002fbd50 | inlined | the steering-wheel loop over `driver_men` |
| - | - | `TickHydrant__6C_gameFUi` | 0x00303eb0 | inlined | the level-effect sweep over `level_effects` (§4c) |
| 0x005e27e0 | `game_car_gun_tick` | `DoubleKiss__6C_gameFUi` | 0x002f1350 | CONFIRMED | The tick of a car-mounted twin gun, gated on `cheat_flags & 0x8`, which no code sets. Past the gate, when the player actor (`+0xe4`) is a car (kind 4): links the frame `+0x2f50` under it, turns it toward the nearest car or active human (squared distance between 25 and 1600, dropped past 2500), and every 330 ms fires both barrels through `game_new_shoot`, ejects two casings through `game_new_cartridge` and pushes the car back along the frame. |
| 0x005e3220 | `game_add_temporary_actor` | `AddTemporaryActor__6C_gameFP7C_actor` | 0x002f1d20 | CONFIRMED | Register a `C_actor` as a scene-lifetime temporary: dedup the master vector, actor-virtual, branch on `act_state` (0/1) into the active list `+0xec` or the inactive list `+0xfc`, push into the master vector. |
| 0x005e3400 | `game_remove_temporary_actor` | `RemoveTemporaryActor__6C_gameFP7C_actor` | 0x002f1e30 | CONFIRMED | The DEFERRED retirement queue: return if it is in the queue, `SetActState(2)` (spent), push onto `+0x134`. That act-state 2 is what `grenade_on_collision` and `grenade_hit` test. |
| 0x005e35d0 | `game_add_temporary_actor_intern` | `InternTempActAdd__6C_gameFP7C_actor` | 0x002f1ed0 | CONFIRMED | The same without the register-virtual: dedup master, branch on act-state into `+0xec` / `+0xfc`, push master. |
| 0x005e37a0 | `game_remove_temporary_actor_intern` | `InternTempActDel__6C_gameFP7C_actor` | 0x002f1fd0 | CONFIRMED | The removal without the actor virtuals: returns if the actor is queued for retirement (`+0x134`), else swap-removes it from the master vector `+0x124` and the two act-state buckets `+0xec` / `+0xfc`. `police_manager_game_done` uses it for a police car carried into the next mission. |
| 0x005e3920 | `game_remove_temporary_actor_now` | `_RemoveTemporaryActor__6C_gameFP7C_actor` | 0x002f2100 | CONFIRMED | The IMMEDIATE half: find in the master vector, swap-remove, `game_invalidate_actor(actor, 0)`, two actor virtuals. |
| 0x005e39c0 | `game_clear_temporary_actors` | `ClearTemporaryActors__6C_gameFv` | 0x002f2280 | CONFIRMED | Empty the master temp-actor vector: loop, skip two sentinel globals, `game_invalidate_actor` plus two virtuals each, zero the count. |
| 0x005e3a50 | `game_update_controls` | `UpdateControls__6C_gameFv` | 0x002f2340 | CONFIRMED | Per-frame: if `+0x40` is set, walk the world actor list (`g_act_mission + 0x38`) and call `player_update_controls` on every kind-2 entry. |
| 0x005e3a90 | `game_cheat_check` | - | - | - | Adds one typed character to the cheat buffer `+0x2ef4` and fires the cheat whose code the buffer ends with (`basic/artifacts/cheats.md`). |
| 0x005e3dc0 | `game_set_traffic_visible` | `SetTrafficVisible__6C_gameFb` | 0x002f2660 | CONFIRMED | Master toggle for the ambient population: the pedestrian manager and the traffic-car registry. |
| 0x005e3df0 | `game_new_shoot` | `NewShoot__6C_gameFP7C_actorRC8S_vectorRC8S_vectorfiP9I3D_framei` | 0x002f26a0 | CONFIRMED | Fire a weapon shot: normalise the direction, a bullet fly-by sound at the point nearest the camera (`flyby_sounds`, 12.0 / 100.0 gates), cast the ray (single below count 2, else a `count`-pellet spread on an orthonormal basis with 0.05 jitter), per-hit material record (`g_surface_material_table + 0x24`, stride `0x170`), impact effect, muzzle particle, gunshot sound (r = 50), NPC look-at broadcast. It is also the producer of the `+0x1d8` in-flight record vector (§4a). |
| 0x005e47c0 | `game_tick_shoot` | `TickShoot__6C_gameFUi` | 0x002f34b0 | CONFIRMED | Advances every in-flight shot of `+0x1d8` and applies its damage (§4a). |
| 0x005e6760 | `game_new_explosion` | `NewExplosion__6C_gameFP7C_actorR8S_vectorffbbbi` | 0x002f5580 | CONFIRMED | Both builds sweep the actor lists TWICE, backwards, splitting on entity kinds `0xc` / `0xb` (hit virtual, hit type 2, a three-float radius block) versus a squared-distance test into a shared `_explosion` helper. Callers `crash_obj_ai` and `grenade_ai_tick`. |
| 0x005e72e0 | `game_new_smoke_trail` | `NewSmokeTrail__6C_gameFR8S_vectorff` | 0x002f6040 | CONFIRMED | Spawn a smoke-trail particle actor into the 32-slot ring at `+0x324` (cursor `+0x3a4`): dedup, random unit direction (`rand × 2 − 1` / `rand + 0.3`, `< 0.4 → +0.4`, normalise) scaled by a length lerped between the two float arguments, then `game_add_temporary_actor`. The ring's 32 `C_smoke_trail` actors are built by `game_init`; the only caller is `game_new_explosion`. |
| 0x005e7580 | `game_new_small_fires` | `NewSmallFires__6C_gameFR8S_vectori` | 0x002f6380 | CONFIRMED | Spawn N small fires at random offsets around a position (`rx² + ry² + 0.25`, `rand × 2` per fire). |
| 0x005e7800 | `game_new_fire` | `NewFire__6C_gameFR8S_vectorP9I3D_frameiffb` | 0x002f6660 | CONFIRMED | Spawn one `C_fire` (its `fire_ctor` / `fire_init` pair) and register it at `+0x318`; its three callers are in `structs/fire.md` §5. |
| 0x005e7a40 | `game_new_cartridge` | `NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` | 0x002f6860 | CONFIRMED | Eject a shell casing from one of the two pools (§4l). The PS2 twin is an 8-byte `{ return; }` stub, so no casing is ever ejected on that build. |
| 0x005e8010 | `game_new_grenade` | `NewGrenade__6C_gameFiR8S_vectorR8S_vectorbP9I3D_frame` | 0x002f6870 | CONFIRMED | Spawn a grenade/molotov projectile actor: type 0/1 → `2grenade.i3d`, 2/3 → `2mol.i3d`; allocate renderer frame type 9, model-cache lookup/evict, spawn a temporary actor. |
| 0x005e8600 | `game_play_3d_sound_by_name` | `Play3DSound__6C_gameFP9I3D_framePCc13I3D_SOUNDTYPERC8S_vectorfffbfb` | 0x002f7020 | CONFIRMED | Creates, positions and starts a 3-D sound from a file name and returns its handle. |
| 0x005e8790 | `game_stop_3d_sound` | `Stop3DSound__6C_gameFibf` | 0x002f7720 | CONFIRMED | Stop a 3D sound by handle: scan the `S_sound_play` vector at `+0x2f8`, match the id, stop and swap-remove. |
| 0x005e8840 | `game_play_3d_sound` | `Play3DSound__6C_gameFP9I3D_sound` | 0x002f77d0 | CONFIRMED | Start a positional sound: virtual play call, increment the handle counter `+0x30c`, return the pre-increment value as the handle, `push_back` an `S_sound_play` record. |
| 0x005e8a90 | `game_clear_effects` | `ClearEffects__6C_gameFv` | 0x002f7bd0 | CONFIRMED | Stops and destroys every live 3-D sound of `+0x2f8`. |
| 0x005e8b40 | `game_fake_shadow_compare_by_distance` | - | - | - | `qsort` comparator over the distance `+0x18` of two shadow items. |
| 0x005e8b70 | `game_add_fake_shadow_buffer` | - | - | - | Preloads one shadow source mesh into the `+0x284` table; the PS2 does this inline in `Init`. |
| 0x005e8da0 | `game_bind_fake_shadow_buffer` | - | - | - | Attaches a shadow item to its source mesh in the `+0x284` table. |
| 0x005e8e30 | `game_add_frame_for_shadow` | `AddFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_typeP9I3D_framePc` | 0x002f8ca0 | CONFIRMED | Add an `I3D_frame` as a shadow caster: bounds-check the `+0x284` slot table, select the shadow vector by `E_shadow_type`, dedup the frame, intern the name through the `+0x3678` free list. Called on the vehicle-exit path. |
| 0x005e91d0 | `game_set_frame_for_shadow` | `SetFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_type` | 0x002f8ec0 | STRUCTURAL | Repoints an existing shadow item at another buffer. |
| 0x005e9350 | `game_del_frame_for_shadow` | `DelFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type` | 0x002f8ed0 | STRUCTURAL | Remove a frame from `+0x290` (type 0) or `+0x2a0` (else), stride `0x1c`; the inverse of the adder. The PS2 twin is an 8-byte `return 0;` stub, so there is no body to diff. |
| 0x005e9540 | `game_enum_traffic_signal_frames` | `cbSemaphores__FP9I3D_frameUi` | 0x002f8ee0 | CONFIRMED | Scene-walk callback: collects each `semafor.i3d` frame with its three lights (§4d). |
| 0x005e9950 | `game_load_traffic_signals` | `LoadSemaphores__6C_gameFv` | 0x002f90c0 | CONFIRMED | Clear the `+0x1a8` signal vector and re-enumerate the scene into it through `game_enum_traffic_signal_frames` (record stride `0x14`). |
| 0x005e99b0 | `game_free_traffic_signals` | `FreeSemaphores__6C_gameFv` | 0x002f9120 | CONFIRMED | Release the three sub-light objects per signal (`elem+0x4/+0x8/+0xc`, vtable `+0x24`), then clear. |
| 0x005e9a60 | `game_tick_traffic_signals` | `TickSemaphores__6C_gameFv` | 0x002f9200 | CONFIRMED | Drive the three sub-lights of each `+0x1a8` record from the phase table `g_game_semaphore_states` (PS2 `semaphore_states`). |
| 0x005e9b20 | `game_load_city_music_areas` | `LoadCityMusicAreas__6C_gameFv` | 0x002f9330 | CONFIRMED | Rebuilds the city-music area list `+0x2fa4` from the scene. |
| 0x005e9b70 | `game_enum_city_music_areas` | `cbCityMusic__Q26C_game19S_hlp$12237game_cppFP9I3D_frameUi` | 0x002f9370 | CONFIRMED | Scene-walk callback: collects each `city_music_` area frame. |
| 0x005e9d70 | `game_city_music_off` | `CityMusicOff__6C_gameFv` | 0x002f9c90 | STRUCTURAL | Silence the city music: clear the current track id `+0x2fb4`, destroy the outgoing track's media `+0x2fbc` at once, make the incoming track `+0x2fb8` the active one and fade it, unlooped, from its current volume to 0 over 5000 ms; `+0x2fb8 = -1`. Graded on call position (§8f). |
| 0x005e9df0 | `game_city_music_set_paused` | `CityMusicPaused__6C_gameFb` | 0x002f9cf0 | CONFIRMED | Set the gate `+0x2fc0`; when set, call `game_city_music_off`. Byte-match with the PS2 (`+0x2fc0` ↔ `+0x13d0`). |
| 0x005e9e10 | `game_set_film_music` | `SetFilmMusic__6C_gameFi` | 0x002f9d20 | STRUCTURAL | Adds a music handle to the film-sequence list `+0x2fc8` (`setfilmmusic`). |
| 0x005e9fd0 | `game_get_film_music` | `GetFilmMusic__6C_gameFi` | 0x002f9d30 | STRUCTURAL | Reads one film-sequence music handle by index (`getfilmmusic`). |
| 0x005ea010 | `game_clear_film_music` | - | - | - | Destroys every film-sequence music handle and empties the list. |
| 0x005ea070 | `game_clear_frame_from_bullet_holes` | `ClearFrameFromFireHole__6C_gameFP9I3D_frame` | 0x002f9d40 | STRUCTURAL | Detaches every `bullet_holes` frame parented under a frame. |
| 0x005ea0c0 | `game_load_differences` | - | - | - | Loads a `.chg` scene-difference file (`loaddifferences`, §4h). |
| 0x005eb4e0 | `game_clear_differences` | `ClearDiffData__6C_gameFv` | 0x002f9d50 | CONFIRMED | Drops everything a difference file added (`cleardifferences`, §4h). |
| 0x005eb920 | `game_free_frame_placement_recursive` | `cbFreeFramePlacement__Q26C_game19S_hlp$13685game_cppFP9I3D_frameRQ23gif38vector<Q26C_game17S_frame_placement,8>` | 0x002fa2e0 | CONFIRMED | Scene-walk callback: drops a frame subtree from the placement registry `+0x00`. |
| 0x005eb9d0 | `game_global_program_run` | `GlobalProgramRun__6C_gameFPc` | 0x002fa520 | CONFIRMED | Runs a registered global script program by name (`+0x2b00`). |
| 0x005eba30 | `game_invalidate_coll` | `InvalidateColl__6C_gameFPUi` | 0x002fa5c0 | CONFIRMED | Invalidate collision for a set of ids; paired with `collision_dyn_erase` in the erase path. |
| 0x005ebac0 | `game_invalidate_actor` | `InvalidateActor__6C_gameFP7C_actorb` | 0x002fa670 | CONFIRMED | Walk the global actor table, dispatch per kind, clear every reference to a dying actor. |
| 0x005ebe90 | `game_invalidate_frame` | `InvalidateFrame__6C_gameFP9I3D_frame` | 0x002fac10 | CONFIRMED | Drops a frame and its subtree from the placement registry. |
| 0x005ebeb0 | `game_invalidate_frame_recursive` | `cbDelFrame__Q26C_game19S_hlp$14436game_cppFP9I3D_frameRQ23gif38vector<Q26C_game17S_frame_placement,8>` | 0x002fad20 | CONFIRMED | The recursive worker of `game_invalidate_frame`. |
| 0x005ebf80 | `game_add_frame_placement` | `AddFramePlacement__6C_gameFP9I3D_frame` | 0x002faf10 | CONFIRMED | Records a frame's spawn transform in the placement registry. |
| 0x005ec250 | `game_del_frame_placement` | `DelFramePlacement__6C_gameFP9I3D_frame` | 0x002fb2a0 | CONFIRMED | Drop a record from the frame-placement registry at `+0x04`. |
| 0x005ec2e0 | `game_raise_timer_timeout_script_event` | `Timer_TimeOut__6C_gameFv` | 0x002fb3b0 | CONFIRMED | Raise the registered program's event when a script timer times out (slot `+0x2d48` / `+0x2d4c`). |
| 0x005ec300 | `game_raise_player_fire_script_event` | `PFire_Fire__6C_gameFv` | 0x002fb3e0 | CONFIRMED | Player-fired-a-weapon hook: raise the event at `+0x2d54` on the program at `+0x2d50` through `program_set_event`. |
| 0x005ec320 | `game_raise_horn_script_event` | `PHorn_Horn__6C_gameFv` | 0x002fb410 | CONFIRMED | Player sounds the horn (slot `+0x2d58` / `+0x2d5c`). |
| 0x005ec340 | `game_raise_water_fall_script_event` | `WFall_Player__6C_gameFv` | 0x002fb440 | CONFIRMED | Player water-fall event (slot `+0x2d60` / `+0x2d64`); the exact in-game trigger is unverified. |
| 0x005ec360 | `game_add_npc_kill_event` | `AddNPCKillEvent__6C_gameFP9C_programiPc` | 0x002fb470 | CONFIRMED | Registers a script event for one NPC model name in `+0x2d6c`. |
| 0x005ec3d0 | `game_do_npc_kill_event` | `Do_NPCKillEvent__6C_gameFPc` | 0x002fb560 | CONFIRMED | The player killed an entity: take the killed entity's model-name basename, scan the `+0x2d6c` registry `{program +0x0, event +0x4, name +0x8}` and fire the matching script event. Sole caller `pedestrian_hit`, which is also the PS2 caller. |
| 0x005ec460 | `game_add_driver_man` | `AddDriverMan__6C_gameFP9I3D_modelP5C_carib` | 0x002fb620 | CONFIRMED | Attach a driver rig to a car and register it at `+0x2d7c`. Picks between two shipped assets: `AutoRidicVolant.I3D` (Czech for car-driver steering wheel) for an ordinary car and `FormuleVolant.I3D` for an open-wheel racer, selected by the trailing bool. |
| 0x005ec700 | `game_del_driver_man` | `DelDriverMan__6C_gameFP9I3D_model` | 0x002fbc60 | CONFIRMED | Remove a driver rig from `+0x2d7c`. |
| 0x005ec7e0 | `game_create_track_effect` | `CreateTrackEffect__6C_gameFR8S_vectorR8S_vectorfR8S_vectorfibfbbP9I3D_frame` | 0x002fbec0 | STRUCTURAL | Places a decal projector of the `+0x75c` pool at the ring cursor. |
| 0x005ecff0 | `game_create_track_effect_between` | `CreateTrackEffect__6C_gameFR8S_vectorR8S_vectorR8S_vectorR8S_vectorfibfbbP9I3D_frame` | 0x002fbed0 | STRUCTURAL | The two-point form of the decal placer. |
| 0x005ed920 | `game_create_grenade_track_effect` | `CreateTrackEffect_Grenade__6C_gameFR8S_vector` | 0x002fbee0 | CONFIRMED | Spawn a grenade-impact track/scorch effect at a position (constants −1.0 / 0.8 / 1.45 / 0.2, `pos.y + 1.0`, random). |
| 0x005ed9d0 | `game_projector_advance_ring_cursor` | - | - | - | Steps the decal ring cursor `+0x29e4`, wrapping past 99. |
| 0x005ed9f0 | `game_clear_projectors_for_frame` | `ClearProjectorsForFrame__6C_gameFP9I3D_frame` | 0x002fc0f0 | CONFIRMED | Hides every decal projector bound to a frame. |
| 0x005eda20 | `game_inventory_stack_clear` | `InventoryStackClear__6C_gameFv` | 0x002fc180 | CONFIRMED | Destroys and empties the inventory script stack (§4f). |
| 0x005eda90 | `game_inventory_stack_push` | `InventoryStackPush__6C_gameFP7C_human` | 0x002fc260 | CONFIRMED | Snapshots a human's inventory onto the inventory stack (§4f). |
| 0x005edd30 | `game_inventory_stack_pop` | `InventoryStackPop__6C_gameFP7C_human` | 0x002fc630 | CONFIRMED | Restores a human's inventory from the inventory stack (§4f). |
| 0x005edfa0 | `game_float_stack_clear` | `FloatStackClear__6C_gameFv` | 0x002fca00 | CONFIRMED | Empties the float script stack (§4f). |
| 0x005edfd0 | `game_float_stack_push` | `FloatStackPush__6C_gameFf` | 0x002fca10 | CONFIRMED | Pushes one float onto the float stack (§4f). |
| 0x005ee1b0 | `game_float_stack_pop` | `FloatStackPop__6C_gameFPf` | 0x002fca80 | CONFIRMED | Pops one float off the float stack (§4f). |
| 0x005ee220 | `game_character_stack_clear` | `CharacterStackClear__6C_gameFv` | 0x002fcab0 | CONFIRMED | Empties the character script stack (§4f). |
| 0x005ee250 | `game_character_stack_push` | `CharacterStackPush__6C_gameFP7C_human` | 0x002fcac0 | CONFIRMED | Snapshots 16 stat dwords of a human onto the character stack (§4f). |
| 0x005ee3d0 | `game_character_stack_pop` | `CharacterStackPop__6C_gameFP7C_human` | 0x002fcda0 | CONFIRMED | Restores those stats and re-drives death, health and the wingman indicator (§4f). |
| 0x005ee690 | `game_box_add` | `BoxPlacementAdd__6C_gameFP9I3D_framei` | 0x002fd040 | CONFIRMED | Register an `I3D_frame` as a placeable box in `+0x2e84` (marker `0x1d97c`); calls `game_add_temporary_actor_intern`. |
| 0x005eeb70 | `game_box_del` | `BoxPlacementDel__6C_gameFP9I3D_frame` | 0x002fd440 | CONFIRMED | Unregister a box: `collision_dyn_erase`, `using_object_del_object` on its `use_node`, free of both, swap-remove from `+0x2e84`. |
| 0x005eec50 | `game_box_recompile` | `BoxPlacementRecompile__6C_gameFv` | 0x002fd530 | CONFIRMED | Registers the `use_node` of every usable owned box (`action_code` `0x7da`) unless another usable box sits 0.2 to 1.0 above it within 0.4 squared horizontally; takes the record out of the registry otherwise. |
| 0x005eeeb0 | `game_box_dest_add` | `BoxPlacementDestAdd__6C_gameFP9I3D_framei` | 0x002fd880 | CONFIRMED | Push a `{dest_frame, owner}` pair into the dest vector `+0x2e94`. |
| 0x005ef080 | `game_box_dest_set_ou` | `BoxPlacementDestSetOU__6C_gameFbP9I3D_frame` | 0x002fd940 | CONFIRMED | Fills `carried_box_node` (`+0x2ea0`) for a destination frame (`action_code` `0x7db`) and registers it with `using_object_add_object`, plus `game_box_save_info`; returns the owner. |
| 0x005ef220 | `game_box_repose_dest_player` | `BoxPlacementReposeDestPlayer__6C_gameFv` | 0x002fdb00 | CONFIRMED | Refills `carried_box_node` from `player+0x6f8` / `+0x6fc` and registers it. |
| 0x005ef340 | `game_box_save_info` | `BoxPlacement_SaveInfo__6C_gameFP9I3D_frame` | 0x002fdc30 | CONFIRMED | Store a box frame's transform into the `+0x34` saved-info vector (stride `0x20`). |
| 0x005ef720 | `game_box_del_info` | `BoxPlacement_DelInfo__6C_gameFP9I3D_frame` | 0x002fe010 | CONFIRMED | Swap-remove a box's saved-info record from `+0x34`. |
| 0x005ef7a0 | `game_box_restore_info` | `BoxPlacement_RestoreInfo__6C_gameFP9I3D_frame` | 0x002fe0c0 | CONFIRMED | Re-apply a box's saved transform to its frame, then remove the record. |
| 0x005ef890 | `game_box_get_start_point` | `BoxPlacement_GetStartPoint__6C_gameFP9I3D_frameR8S_vector` | 0x002fe230 | CONFIRMED | Copy a box's saved start position out of `+0x34` into an out-vector, world-transformed; returns 1/0. |
| 0x005ef910 | `game_box_place_at_next_dest` | `BoxPlacementGetNextDest__6C_gameFP9I3D_frameP9I3D_frame` | 0x002fe310 | CONFIRMED | PLACE a box at its dest: copy the transform, reparent, remove the dest, `game_box_add(box, 0x1d97c)`, car branch. The PS2 verb `GetNextDest` does not describe the body. |
| 0x005efca0 | `game_box_get_enemy_dest` | `BoxPlacementGetEnemyDest__6C_gameFP7C_humani` | 0x002fe600 | CONFIRMED | Pick an NPC's box dest: the nearest owned box (squared distance plus a 10.0 height penalty) when not carrying (`human+0x6f8`), else the assigned dest. |
| 0x005efe20 | `game_box_count_owned` | `BoxPlacement_GetBox__6C_gameFi` | 0x002fe7c0 | CONFIRMED | COUNT of boxes in `+0x2e84` whose owner (`box+0x90`) equals the argument. The PS2 verb `GetBox` does not describe the body. |
| 0x005efe60 | `game_box_dest_count_owned` | `BoxPlacement_GetBoxDest__6C_gameFi` | 0x002fe810 | CONFIRMED | COUNT of dests in `+0x2e94` whose owner (`+0x04`) equals the argument. |
| 0x005efea0 | `game_box_move_to_dest` | `BoxPlacement_MoveBoxToDest__6C_gameFi` | 0x002fe860 | CONFIRMED | Find an owner's box and dest and run the move sequence (set-ou / del / recompile / reparent / repose). |
| 0x005eff70 | `game_box_recompile_for_car` | `BoxRecompileBoxForCar__6C_gameFP5C_carb` | 0x002fe9b0 | CONFIRMED | For every box under a car's frame (`car+0x68`): with the flag 0, clears `usable` and takes `coll` out of the grid; with 1, sets `usable`, rebuilds `coll` and `inv_world`, enters `coll` and ends in `game_box_recompile`. |
| 0x005f0170 | `game_box_count_owned_on_car` | `BoxPlacementPoPiciNaZkurvenemAute__6C_gameFP5C_cari` | 0x002fec50 | CONFIRMED | COUNT of an owner's boxes attached to a car: iterate `+0x2e84` by `owner+0x90`, walk each box's child list, count nodes whose object is the car. The PS2 name is developer slang. |
| 0x005f01e0 | `game_death_actor_add` | `DeathActorAdd__6C_gameFP7C_actor` | 0x002fedc0 | CONFIRMED | Dedup-enqueue an actor onto the death queue `+0x2ee8`. |
| 0x005f03a0 | `game_process_death_actors` | `DeathActorProcess__6C_gameFv` | 0x002fee50 | CONFIRMED | Per-frame: drain `+0x2ee8` / `+0x2eec` and force-kill each kind-2 / kind-`0x1b` entry through `human_force_death`. |
| 0x005f0410 | `game_get_actor_from_guid` | `GetActorFromGUID__6C_gameFUi` | 0x002feef0 | CONFIRMED | Finds a live actor by its savegame GUID (`actor+0x18`). |
| 0x005f0450 | `game_save_game_get_size` | `SaveGameGetSize__6C_gameFb` | 0x002fef80 | CONFIRMED | Serialized size of the save block: pursuit, the `0x43`-byte header, two stack tables, the inventory-size loop, using-object; the full flag adds the media block. |
| 0x005f05a0 | `game_save_game_save` | `SaveGameSave__6C_gameFPPvb` | 0x002ff020 | CONFIRMED | Serialize: header (counts, score `[0x3e]` / `[0x3f]` from `+0x363e` / `+0x3640`, timer), `cursor += 0x43`, pursuit save, sub-objects. |
| 0x005f0990 | `game_save_game_load` | `SaveGameLoad__6C_gameFPPv` | 0x002ff1f0 | CONFIRMED | Deserialize behind a tag-`0x0a` gate: header, score (through `game_freeride_score_set_on` / `_set`), timer, `cursor += 0x43`, pursuit load, stack clears, sub-objects. The score restore is byte-identical to the PS2. |
| 0x005f1090 | `game_savegame_load_before` | `SaveGameLoadBefore__6C_gameFPPv` | 0x002ff4c0 | CONFIRMED | Pre-load hook; reads the mission-number pair on `C_mission`. |
| 0x005f10d0 | `game_lightness_add` | `LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector` | 0x002ff510 | STRUCTURAL | Registers a frame for ambient-light sampling (`structs/lightness.md`). |
| 0x005f1370 | `game_lightness_del` | `LightnessDel__6C_gameFP9I3D_frame` | 0x002ff520 | CONFIRMED | Removes a frame's lightness record. |
| 0x005f1400 | `game_lightness_clear` | `LightnessClear__6C_gameFv` | 0x002ff5c0 | CONFIRMED | Deletes every lightness record. |
| 0x005f1460 | `game_lightness_tick` | `LightnessTick__6C_gameFUi` | 0x002ff620 | STRUCTURAL | Per-frame ambient lightness sampling and fade. |
| 0x005f1750 | `game_lightness_apply_recursive` | - | - | - | Applies a lightness value down a frame subtree. |
| 0x005f17c0 | `game_add_shoot_light` | `AddShootLight__6C_gameFR8S_vectorfiUs` | 0x002ff630 | STRUCTURAL | Light a slot of the 10-slot shoot-light pool at `+0x2a48` for a given lifetime (§4a). The PS2 twin is an 8-byte stub, so the PC body is the only one that exists. |
| 0x005f1900 | `game_use_light_cache_system` | `UseLightCacheSystem__6C_gameFv` | 0x002ff7d0 | CONFIRMED | Arms the `I3D_light` cache `+0x16c`. |
| 0x005f1950 | `game_light_cache_collect_recursive` | `cbLight__Q26C_game19S_hlp$18015game_cppFP9I3D_frameRQ23std50vector<P9I3D_light,Q23std22allocator<P9I3D_light>>` | 0x002ff8c0 | CONFIRMED | Scene-walk callback: collects the lights of a frame subtree into the cache. |
| 0x005f19c0 | `game_recompute_light_cache` | `RecomputeLightCache__6C_gameFv` | 0x002ffa50 | CONFIRMED | Forces one pass of the light-cache distance test (§8a). |
| - | - | `LightCacheTick__6C_gameFUi` | 0x002ffa90 | inlined | one of its two copies; the other is the `light_cache` block of `game_tick` (§8a) |
| 0x005f1b10 | `game_media_create` | `CreateIShow__6C_gameFUi` | 0x002ffc20 | CONFIRMED | Allocate a media entry: fresh id from the `+0x2f38` counter, engine create, push into `+0x2f1c`. |
| 0x005f1d90 | `game_media_open` | `OpenIShow__6C_gameFPCci` | 0x002ffd20 | CONFIRMED | Open/load an in-game FMV clip by id into `+0x2f1c`, with a `.ext` → `.mp3` audio fallback. |
| 0x005f1f40 | `game_media_destroy` | `DestroyIShow__6C_gameFi` | 0x002fff30 | CONFIRMED | Remove one entry by id: release, free the name, swap-remove. |
| 0x005f2030 | `game_media_destroy_all` | `DestroyIShowAll__6C_gameFv` | 0x00300060 | CONFIRMED | Release every entry and reset the vector. |
| 0x005f2140 | `game_media_get` | `GetIShow__6C_gameFi` | 0x00300120 | CONFIRMED | Find a media entry by id and return its show object. |
| 0x005f21a0 | `game_media_store_volume` | `IShowStoreVolume__6C_gameFif` | 0x00300180 | CONFIRMED | Store the per-entry volume (`element+0x10`) by id. |
| 0x005f2200 | `game_media_set_loop` | `IShowLoop__6C_gameFib` | 0x003001d0 | CONFIRMED | Set or clear the loop flag (`element+0x08` bit 0) by id. |
| 0x005f2280 | `game_media_apply_volumes` | `IShowSetVolume__6C_gameFv` | 0x003002a0 | CONFIRMED | Re-apply `g_music_volume` × per-entry volume to every entry. |
| 0x005f22e0 | `game_media_fade_all` | `IShowFadeDownAll__6C_gameFi` | 0x00300320 | CONFIRMED | Fade every entry through `game_media_fade`, or stop it when the volume is low. |
| 0x005f2390 | `game_media_fade` | `IShowFade__6C_gameFiffi` | 0x00300420 | CONFIRMED | Fade one entry by id, from one volume to another over a duration. |
| 0x005f2650 | `game_load_road_blocks` | `LoadRoadBlocks__6C_gameFv` | 0x00300550 | CONFIRMED | Rebuilds the roadblock list `+0x1b4` from the scene. |
| 0x005f26d0 | `game_collect_road_block_node` | `cbRoadBlock__Q26C_game19S_hlp$18574game_cppFP9I3D_frameUi` | 0x003005b0 | CONFIRMED | Scene-walk callback: one roadblock frame. |
| 0x005f2790 | `game_load_taxi_points` | `LoadTaxiPoints__6C_gameFv` | 0x00300680 | CONFIRMED | Clear and re-enumerate the scene for `taxi_point` objects into `+0x1f4`. The PC refactors the PS2's inline scan into the shared scene enumerator. |
| 0x005f27e0 | `game_collect_taxi_point` | `cbFindTaxiPoints__Q26C_game19S_hlp$18659game_cppFP9I3D_frameRQ23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>>` | 0x00300910 | STRUCTURAL | Scene-walk callback: one taxi point. |
| 0x005f2990 | `game_is_valid_taxi_passenger` | `IsValidTaxiPassenger__6C_gameFv` | 0x00300ad0 | CONFIRMED | The candidate at `+0x204` is valid iff its flag byte `+0x5e` is clear, its group `+0x98` is set, and the player's group (`+0xe4`) matches or is unset. |
| 0x005f29d0 | `game_del_traffic_signal` | `DelSemaphore__6C_gameFP9I3D_frame` | 0x00300b30 | CONFIRMED | Swap-remove a traffic-light record by frame key from `+0x1a8` (stride `0x14`). |
| 0x005f2a80 | `game_load_telephones` | `LoadTelephones__6C_gameFv` | 0x00300bf0 | CONFIRMED | Clear and re-enumerate the scene for `alarm01.i3d` / `PHbox01.i3d` callable devices into the `+0x1c4` list. This diff is what proved the singleton the PS2 calls `world` is `C_game`. |
| 0x005f2e30 | `game_collect_telephone` | `cbTelephones__Q26C_game19S_hlp$18990game_cppFP9I3D_frameUi` | 0x00300e40 | CONFIRMED | Scene-walk callback: one alarm box or phone box (§4d). |
| 0x005f30c0 | `game_telephone_list_erase_by_key` | `DelTelephones__6C_gameFP9I3D_frame` | 0x00300f60 | CONFIRMED | Swap-remove a callable device by frame key (stride `0xc`). |
| 0x005f3180 | `game_telephone_list_contains_key` | `IsTelephoneExist__6C_gameFP9I3D_frame` | 0x00301010 | CONFIRMED | Is a callable device registered for this frame? A backward scan of the list. |
| 0x005f31e0 | `game_human_talk_add` | `AddHumanTalk__6C_gameFP7C_humani` | 0x00301080 | CONFIRMED | Pushes `{human, speech id}` onto the talk queue (§4g). |
| 0x005f33b0 | `game_human_talk_del` | `DelHumanTalk__6C_gameFP7C_humani` | 0x003010c0 | CONFIRMED | Erases queue entries by speech id (§4g). |
| 0x005f3440 | `game_human_talk_stop_for_human` | `DeathHumanTalk__6C_gameFP7C_human` | 0x00301180 | CONFIRMED | Erases a human's queue entries and stops their speech (§4g). |
| 0x005f34e0 | `game_build_traffic_signal_model_idx` | `CreateSignalMarkIDX__6C_gameFv` | 0x00301250 | CONFIRMED | Scan the scene model table for the phase models `dopravni1B..8B.I3D` and build the u16 model-index list at `+0x1e8`. |
| 0x005f3860 | `game_destroy_control_signal` | `DestroyControlSignal__6C_gameFPUiR8S_vectorR8S_vectorR8S_vectorP7C_human` | 0x003015c0 | CONFIRMED | Knocks down the nearest breakable street sign within 10 units. |
| 0x005f3b50 | `game_entity_block` | `EntityBlock__6C_gameFbb` | 0x00301950 | CONFIRMED | Freezes or thaws every kind-`0x1b` entity of the two actor lists. |
| 0x005f3bf0 | `game_add_invisible_actor` | `AddInvisibleActor__6C_gameFP7C_actor` | 0x00301a60 | CONFIRMED | Appends `{actor, 0}` to the invisible-actor list `+0x180`. |
| 0x005f3dd0 | `game_flush_pending_actor_lookups` | `FindActorsVect__6C_gameFv` | 0x00301ae0 | CONFIRMED | Drain the deferred actor-lookup queue at `+0x2f80` (stride `0xc`) and clear it. It touches no audio. |
| 0x005f3e60 | `game_particle_add` | - | - | - | Pushes a particle guid onto the script particle registry `+0x2f90`. |
| 0x005f4010 | `game_particle_destroy_all` | - | - | - | Destroys every registered particle and empties the registry. |
| 0x005f4070 | `game_stop_all_sounds` | `StopAllSounds__6C_gameFv` | 0x00301b60 | CONFIRMED | Silences every actor through its sound virtual (§4e). |
| 0x005f40b0 | `game_pause_all_sounds` | `PauseAllSounds__6C_gameFv` | 0x00301c00 | CONFIRMED | Pause every sound, saving state into `+0x2d8` and the media-id sibling `+0x2e8`. The exact inverse of `game_play_all_sounds`, proven by the shared callee: `speech_pause_all_channels()` is `DabingPauseSounds(1,0)` here and `(0,0)` there, in the same statement position. |
| 0x005f41c0 | `game_pause_scene_sounds_recursive` | - | - | - | Scene-walk helper of `game_pause_all_sounds`: saves every playing sound frame of a subtree (§4e). |
| 0x005f4270 | `game_play_all_sounds` | `PlayAllSounds__6C_gameFb` | 0x003021c0 | CONFIRMED | Replays everything `game_pause_all_sounds` saved (§4e). |
| 0x005f4400 | `game_scratch_add` | `ScratchAdd__6C_gameFP5C_carP9I3D_frameUiRC8S_vectorRC8S_vector` | 0x00302500 | CONFIRMED | Stamp a car scratch/decal: `OBJ_`-frame and surface-type gate (excludes {2,3,4,5,7}), allocate a pool slot, orient (`SetDir3` or random spin), speed alpha (`× 1/55.0`, clamped 0.2 … 1.0), age alpha, then recompile. PS2 caller `Deform_Vehicle_Part`. |
| 0x005f4830 | `game_scratch_recompile` | `ScratchRecompile__6C_gameFP5C_carP9I3D_frame` | 0x00302ae0 | CONFIRMED | Rebuild the decal mesh for the pool record matching `{car, source_frame}`: save state, set flag, mesh-build virtual `(0x10, source)`, restore state. |
| 0x005f48a0 | `game_scratch_delete_from_car` | `ScratchDeleteFromCar__6C_gameFP5C_car` | 0x00302be0 | CONFIRMED | Remove every scratch belonging to a car: scan the 100-record pool for `car == param`, two frame virtuals (`+0x2c`(0,0) then `+0x24`(0)). |
| 0x005f48e0 | `game_scratch_get_free_idx` | `ScratchGetFreeIdx__6C_gameFP5C_carP9I3D_frameRC8S_vector` | 0x00302c90 | CONFIRMED | Allocate a pool slot in three passes: dedup (< 0.16, reuse the min-age of three or more), free slot (`car == 0`), LRU evict (farthest from the camera, 25.0 initial). Returns the index. |
| 0x005f4af0 | `game_add_created_frame` | `AddCreatedFrameToVector__6C_gameFP9I3D_frame` | 0x00302fe0 | CONFIRMED | Appends a script-created frame to the vector `+0x20`. |
| 0x005f4cc0 | `game_del_created_frame` | `DelCreatedFrameToVector__6C_gameFP9I3D_frame` | 0x00303080 | CONFIRMED | Removes a script-created frame from the vector `+0x20`. |
| 0x005f4d30 | `game_load_scream_areas` | `LoadScreamAreas__6C_gameFv` | 0x00303100 | CONFIRMED | Rebuilds the scream-area list `+0x361c` from the scene. |
| 0x005f4d90 | `game_collect_scream_area` | `cbScreamArea__Q26C_game19S_hlp$20556game_cppFP9I3D_frameUi` | 0x00303160 | CONFIRMED | Scene-walk callback: one `enm_scrm_` area (§4d). |
| 0x005f5010 | `game_tick_scream_areas` | `TickScreamAreas__6C_gameFv` | 0x00303290 | CONFIRMED | Walk the scream-area vector `+0x361c` (stride 8, `{frame, u16 mask}`) against the camera and OR the hit masks into `+0x3628` (1 when none hit). |
| 0x005f51c0 | `game_apply_weather_detail_level` | `OsefujPocasi__6C_gameFbUi` | 0x003034a0 | CONFIRMED | Scales the weather particle budget by the detail level (`+0x3630` / `+0x3634`). |
| 0x005f5310 | `game_dying_human_add` | `AddDeadHumanForTick__6C_gameFP7C_human` | 0x00303660 | CONFIRMED | Appends a dying human to the list `+0x114`. |
| 0x005f54d0 | `game_dying_human_del` | `DelDeadHumanForTick__6C_gameFP7C_human` | 0x003036f0 | CONFIRMED | Removes a human from the dying-human list. |
| 0x005f5540 | `game_dying_human_tick` | `TickDeadHumanForTick__6C_gameFUi` | 0x00303750 | CONFIRMED | Runs the death animation of every dying human. |
| 0x005f57e0 | `game_add_hydrant` | `AddHydrant__6C_gameFP9I3D_frame` | 0x00303a50 | CONFIRMED | Starts a hydrant water spray in the level-effect registry (§4c). |
| 0x005f5b40 | `game_del_all_hydrants` | `DelAllHydrants__6C_gameFv` | 0x00304210 | CONFIRMED | Destroys every record of the level-effect registry (§4c). |
| 0x005f5bf0 | `game_freeride_score_set_on` | `ScoreSetOn__6C_gameFb` | 0x00304330 | CONFIRMED | Freeride score-display toggle: `+0x363e = on`; refresh the HUD either way (the PS2 hides it with `BonusSetCount(0x7fffffff)`). |
| 0x005f5c30 | `game_freeride_score_set` | `ScoreSet__6C_gameFi` | 0x00304380 | CONFIRMED | `+0x3640 = value`; if `+0x363e` is set, refresh the HUD. |
| 0x005f5c60 | `game_set_camera_rot_repair` | `SetCameraRotRepair__6C_gameFv` | 0x003043c0 | CONFIRMED | Snapshots the scene camera's orientation into `+0x3644`. |
| 0x005f5cd0 | `game_clear_cartridges` | `ClearCartridges__6C_gameFv` | 0x00304450 | CONFIRMED | Silences every shell casing of both pools (§4l). |
| 0x005f5d60 | `game_tick_car_sound_manager` | `TickCarSoundManager__6C_gameFv` | 0x00304550 | CONFIRMED | Budget car sounds to the nearest N cars by camera distance (gate `+0x3668` ↔ PS2 `+0x1550`; car virtuals `+0x94` / `+0x51e`). No radio is involved. |
| 0x005f5f60 | `game_item_blur_done` | `ItemBlurDone__6C_gameFv` | 0x00304830 | CONFIRMED | Clear every blur item, reset `end = begin` and the enable byte `+0x3684`. The PC clears and RETAINS capacity; the PS2 frees it (the PC storage free is in `game_dtor`). |
| 0x005f5fe0 | `game_item_blur_tick` | `ItemBlurTick__6C_gameFUi` | 0x00304910 | CONFIRMED | Per-frame motion-blur streak-trail update: ms to seconds, camera frame and `S_matrix::Inverse`, iterate `+0x368c`, 900.0 (30 u²) gate, × 0.5 fade, virtual sub-item destroy. Called from `game_tick` behind `+0x3684`. |
| 0x005f6690 | `game_item_blur_del_item` | `ItemBlurDelItem__6C_gameFib` | 0x00305300 | CONFIRMED | Remove one blur item from `+0x368c` (stride `0x28`): destroy the sub-item list; when `compact` is set, swap-remove. |

### 3d. Free helpers

The `[0x005f6770, 0x005f79a0)` COMDAT tail (§8e) serves many unrelated owners; the rows are here
because the unit is this document's.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005df210 | `gui_show_load_progress` | `_show_progress___Ff` | 0x002ea060 | CONFIRMED | advances the load-progress screen; called through `game_init` |
| 0x005eb480 | `std_vector_deleter_ensure_alloc` | - | - | - | COMDAT container helper of `game_load_differences` |
| 0x005eb4a0 | `std_vector_deleter_assign_alloc` | - | - | - | COMDAT container helper of `game_load_differences` |
| 0x005ecfb0 | `vec3_dot_difference` | - | - | - | the decal placers' dot product of a vector difference |
| 0x005ed7e0 | `vec3_make_basis_from_direction` | - | - | - | the decal placers' orthonormal basis |
| 0x005f5d30 | `compare_car_snd_by_distance` | `compare_car_snd__FPCvPCv` | 0x00304510 | CONFIRMED | `qsort` comparator of `game_tick_car_sound_manager` |
| 0x005f6770 | `vec_clear_keep_capacity_005f6770` | - | - | - | COMDAT: clear, keeping the storage |
| 0x005f67a0 | `vec_count_stride_0x34` | - | - | - | COMDAT: `S_frame_placement` count |
| 0x005f67d0 | `vec_clear_keep_capacity_005f67d0` | - | - | - | COMDAT: clear, keeping the storage |
| 0x005f6800 | `vec_erase_range_stride2` | - | - | - | COMDAT: range erase, stride 2 |
| 0x005f6830 | `std_vector4_insert_at` | - | - | - | COMDAT: insert, stride 4; 55 callsites (§8e) |
| 0x005f69d0 | `vec_erase_range_stride_0xc` | - | - | - | COMDAT: range erase, stride `0xc` |
| 0x005f6a20 | `std_vector5c_insert_at` | - | - | - | COMDAT: insert, stride `0x5c`; the `S_shoot` producer |
| 0x005f6c40 | `vec_erase_at_stride0x18` | - | - | - | COMDAT: erase one, stride `0x18` |
| 0x005f6c90 | `vec_erase_range_stride_0x14` | - | - | - | COMDAT: range erase, stride `0x14` |
| 0x005f6ce0 | `std_vector8_insert_at_005f6ce0` | - | - | - | COMDAT: insert, stride 8 |
| 0x005f6e90 | `std_vector14_insert_n` | - | - | - | COMDAT: insert n, stride `0x14` |
| 0x005f71a0 | `std_vector28_insert_n` | - | - | - | COMDAT: insert n, stride `0x28` |
| 0x005f74a0 | `copy_range_stride_0x10` | - | - | - | COMDAT: range copy, stride `0x10` |
| 0x005f74f0 | `copy_range_stride_0x5c` | - | - | - | COMDAT: range copy, stride `0x5c` |
| 0x005f7530 | `copy_range_stride_0x18` | - | - | - | COMDAT: range copy, stride `0x18` |
| 0x005f7570 | `fill_n_stride_0x10` | - | - | - | COMDAT: fill n, stride `0x10` |
| 0x005f75b0 | `stl_copy_range_stride_0x34` | - | - | - | COMDAT: range copy, stride `0x34` |
| 0x005f75e0 | `stl_copy_range_stride_0x20` | - | - | - | COMDAT: range copy, stride `0x20` |
| 0x005f7610 | `stl_copy_range_stride_0x5c` | - | - | - | COMDAT: range copy, stride `0x5c` |
| 0x005f7640 | `stl_copy_range_stride_0x28` | - | - | - | COMDAT: range copy, stride `0x28` |
| 0x005f7670 | `stl_copy_range_stride_0x14` | - | - | - | COMDAT: range copy, stride `0x14` |
| 0x005f76a0 | `vec_copy_range_stride0x10` | - | - | - | COMDAT: range copy, stride `0x10` |
| 0x005f76e0 | `stl_copy_range_stride_0xc` | - | - | - | COMDAT: range copy, stride `0xc` |
| 0x005f7710 | `stl_copy_construct_stride_0x14` | - | - | - | COMDAT: copy-construct, stride `0x14` |
| 0x005f77b0 | `stl_copy_one_stride_0x5c` | - | - | - | COMDAT: copy one, stride `0x5c` |
| 0x005f77d0 | `stl_copy_one_stride8` | - | - | - | COMDAT: copy one, stride 8 |
| 0x005f77e0 | `std_vector4_copy_assign_at_0x04` | - | - | - | COMDAT: vector copy-assign, stride 4 |
| 0x005f7980 | `vec_owner_scalar_deleting_dtor_005f7980` | - | - | - | COMDAT: scalar deleting destructor of a vector owner |

## 4. Subsystems

### 4a. Shooting

`game_new_shoot` is the producer and `game_tick_shoot` (`TickShoot__6C_gameFUi`) is the
consumer. `game_tick_shoot` is 8036 bytes that touch no sound at all: it steps the
**`S_shoot`** record vector at `+0x1d8` (stride `0x5c`) — position `+0x00`, velocity `+0x0c`,
range `+0x18`, previous position `+0x1c`, damage `+0x28`, owner `+0x2c` — advancing each
record by `dt * 0.3`, ray-testing the travelled segment and applying damage.

`game_add_shoot_light(pos, range, lifetime_ms, flags)` claims the first of the ten slots at
`+0x2a48` whose lifetime has run out, sets the light's range to `(range * 0.5, range)`, moves it to
`pos`, links it into the scene, switches it on with an intensity of `rand × 0.4 + 0.8` and stores
the lifetime and the flags. The per-frame part is inlined into `game_tick` (§8a): each slot's
lifetime counts down, and at zero the light is unlinked and switched off; while it lasts, a slot
with flag bit 0 flickers (`rand × 0.4 + 0.8`) and fades linearly over its last 500 ms. A slot
without the flag keeps a steady light until it goes out.

| caller | range | lifetime | flags |
|---|---|---|---|
| `human_shooting` (muzzle flash) | 7.0 | 50 ms | 0 |
| `actor_shot_update` | 7.0 | 50 ms | 0 |
| `game_new_explosion` | 12.0 | 100 ms | 0 |
| `game_new_fire` | 10.0 | the fire's own lifetime | 1 |

The muzzle effect of `game_new_shoot` itself is a particle: `particle_manager_create_particle` with
the weapon's particle type at the muzzle frame. Shell casings come out of the two cartridge pools
(§4l).

### 4b. The active and inactive lists, and the temporary actors

Every actor of the scene is in exactly one of two lists, chosen by its act state (`actor+0x1c`,
set by the script command `act_setstate`: `active` = 0, `inactive` = 1, `off` = 2;
`structs/actor.md` §2b): the **active list** `+0xec` holds state 0 and the **inactive list**
`+0xfc` state 1. `game_init` fills the active list with the whole mission actor list;
`game_change_act_state` then moves an actor between the two whenever its state changes and drops
it from both when it is switched off. Whole-scene scans walk both lists, so an actor switched off
is invisible to them; the garage's end-of-mission car scan is one
(`basic/artifacts/garage/storing-cars-in-the-garage.md`). Ambient traffic cars are not actors of
either list until something takes one over (`traffic_car_take_over_cruising_car`).

Runtime actors use six functions over four containers: the two lists, the master vector
`+0x124`, and the deferred retirement queue `+0x134`.

| function | what it does |
|---|---|
| `game_add_temporary_actor` | the public add, with the register-virtual |
| `game_add_temporary_actor_intern` | the same without it |
| `game_remove_temporary_actor` | DEFERRED: set act-state 2 (spent) and queue onto `+0x134` |
| `game_remove_temporary_actor_intern` | swap-remove from the master vector and both buckets, unless queued for retirement; no virtuals |
| `game_remove_temporary_actor_now` | IMMEDIATE: swap-remove from the master vector and invalidate |
| `game_clear_temporary_actors` | empty the master vector, skipping two sentinel globals |

### 4c. The level-effect registry and the hydrants

`C_game + 0x2c8` is one stride-`0x18` vector of `{particle guid, covered flag, vec3 pos,
I3D_sound *}`. It has two producers and one destroyer:

* **level-placed effects** are read from `missions\%s\effects.bin` at scene load
  (`structs/particle.md` §6b);
* **`game_add_hydrant`** pushes a broken fire hydrant's water spray: particle type `0x57`,
  a driver sound object of type 4 carrying sound `0x147`, looped and attached to the frame.
  It returns immediately unless the detail level at `C_mission + 0xa8` is 2 or more and the
  frame is within 150 units of the camera. Its callers are `crash_obj_init` and
  `crash_obj_save_game_load` — hydrants are `C_crash_obj` instances;
* **`game_del_all_hydrants`** (callers `game_init` and `game_done`) destroys every record's
  particle, releases its sound object through the object's own virtual destructor and empties
  the vector.

`game_tick` walks the vector every frame and does three things per entry:

1. `particle_manager_is_particle_alive(guid)` — a dead effect is unlinked and swap-erased;
2. beyond **150 units** of the player (`22500.0` = 150²) the effect is destroyed outright;
3. a vertical ray is cast **upward**; while it hits anything whose owner is not a `C_car`,
   `particle_manager_set_particle_emitting(guid, 0)` stops emission, and it is switched back
   on with `1` when the sky is clear again.

In game: a street plume stops making new particles under a bridge or an awning, and a car
parked over it does not count. The particles still alive keep drawing.

### 4d. Traffic signals and callable devices

Six functions over `+0x1a8`: `game_load_traffic_signals`, `game_enum_traffic_signal_frames`
(its enumerator callback), `game_free_traffic_signals`, `game_tick_traffic_signals`,
`game_build_traffic_signal_model_idx` and `game_del_traffic_signal`. There is **no
single-record adder** (§8f): the list is built wholesale at scene load, exactly like the
roadblock and telephone lists, and `Del` is the only single-record mutator.

`game_enum_traffic_signal_frames` takes every `semafor.i3d` frame and looks up its child frames
`RED`, `ORANGE` and `GREEN` into the record's three sub-light slots. Each tick of the light clock
(every 3000 ms) `game_tick_traffic_signals` switches the three on or off from
`g_game_semaphore_states[phase]` = `[1,1,1,1,1,4,4,4,4,2]`: bit `1` red, bit `2` orange, bit `4`
green. A cycle is therefore 15 s red (phases 0..4), 12 s green (5..8) and 3 s orange (9). The two
phases start at 0 and 5 and advance together, so the two crossing axes show opposite colours. The
traffic AI reads the same phases to stop at a red light (`structs/driver.md` §4).

`game_collect_telephone` matches frame model basenames against exactly two literals —
`alarm01.i3d` → type 0 (the wall alarm box) and `PHbox01.i3d` → type 1 (the phone box) — so
the "telephone" list holds **two** device kinds.
The second half of `game_load_telephones` adds the alarm and phone-box instances of the city cache
(`structs/cache_base_block.md`) with `frame` 0 and `cache_instance` set; a reader takes such a
device's position from the instance.

`game_collect_scream_area` accepts only frames named `enm_scrm_*` and parses each area's u16
mask out of the hexadecimal number in parentheses inside the frame name.

### 4e. Sound

`game_stop_all_sounds` (`StopAllSounds`), `game_pause_all_sounds` (`PauseAllSounds`) and
`game_play_all_sounds` (`PlayAllSounds`) are the whole-scene family; `game_play_3d_sound` /
`game_stop_3d_sound` are the per-sound pair over `+0x2f8`; `game_flush_pending_actor_lookups`
drains an actor-lookup queue and has nothing to do with audio.
`game_pause_scene_sounds_recursive` exists only on the PC. The fades of the `+0x2f8` records
(§2) are advanced by the sound-list walk inlined into `game_tick` (§8a), which also retires every
sound that has stopped playing.

### 4f. The three script stacks are one API

`game_inventory_stack_push` / `_pop` / `_clear`, `game_character_stack_push` / `_pop` /
`_clear` and `game_float_stack_push` / `_pop` / `_clear` are nine functions over three
vectors that share the byte `stacks_clear_pending` (`+0x2e4c`): `game_init` sets it, and the first
push of any of them after that empties all three and clears it. The inventory stack snapshots
`C_human+0x480 .. +0x560` (the `G_Inventory`
sub-object) into a `0xe4`-byte heap copy; the character stack snapshots 16 dwords from
`C_human+0x640 .. +0x67c`, and its pop re-drives `human_force_death`,
`hud_set_player_health_value` and `hud_wingman_set_indicator_value_for_slot`.

### 4g. The human-talk queue

`game_human_talk_add(who, id)` pushes the pair onto `+0x2f44`. `game_human_talk_del(who, id)`
erases by the **speech id** and stops nothing. `game_human_talk_stop_for_human(who)` erases by
the **human** and calls `speech_channel_stop` on each entry it removes — that call is the
whole discriminator between the two erasers, and it is why the third is the one invoked from
`human_hit`, `human_movement`, `human_die_in_exploding_car`, `human_throw_from_car_truck`
and `entity_force_ai`.

### 4h. City music, media and film music

The city-music block is `+0x2fb0 .. +0x2fc0` with its area vector at `+0x2fa4`;
`game_city_music_set_paused` gates it and calls `game_city_music_off`, which destroys the outgoing
track at once and fades the incoming one to silence over 5000 ms. The script arm that starts an
in-game music clip calls `game_city_music_off` too.

The media / in-game-FMV player is eleven functions over `+0x2f1c` (§3), with per-entry volume
re-applied against `g_music_volume` by `game_media_apply_volumes`.

`game_load_differences` (the script command `loaddifferences`, dispatcher case `0x43`) loads a
`.chg` scene-difference file from `Mafia\diff` through a stack `C_chunk`, and
`game_clear_differences` (`cleardifferences`) drops everything it added; the other consumer of
the loader is `race_winners_replay_begin`, with `difrace.chg` and `difracech.chg`.
`game_set_film_music` / `game_get_film_music` are `setfilmmusic` (case `0x149`) and
`getfilmmusic` (case `0x14a`), and `game_clear_film_music` runs at the tail of the cutscene
teardown, case `0x2e`.

### 4i. The scratch decal pool

The inline array `scratches` at `+0x2fd8`, stride `0x10`. `game_scratch_add` stamps a
mark, `game_scratch_get_free_idx` allocates the slot in three passes, `game_scratch_recompile`
rebuilds the mesh and `game_scratch_delete_from_car` clears a car's marks (pool sizes: §8d).

### 4j. Freeride score

`+0x363e` is the display flag and `+0x3640` the value. `game_freeride_score_set_on` /
`game_freeride_score_set` write them and refresh the HUD; `game_freeride_score_get` is a
PC-only accessor consumed by the `FREERIDE_SCOREGET` / `SCOREADD` script commands. All three
are re-driven by `game_save_game_load`.

### 4k. Box placement

Sixteen functions over two vectors (`+0x2e84` boxes, `+0x2e94` destinations), one saved-info
vector (`+0x34`) and `carried_box_node` (`+0x2ea0`), the `S_using_object` of the carried box's
destination. Each box owns a second such record, `use_node`, registered while the box can be picked
up; the two differ by `action_code`: `0x7db` for the destination, `0x7da` for a box. The functions
are listed in §3, and the PC names state what the bodies do (§8f).

### 4l. The two shell-casing pools

Both are ring buffers of `C_cartridge *` built by `game_init`, drained by `game_done` and
`game_done_intern`, and served by `game_new_cartridge`; nothing allocates at runtime.
Pool A is 100 entries at `+0x3a8` with its cursor at `+0x538` (model `2nabojnice.i3d`); pool B
is 30 entries at `+0x53c` with its cursor at `+0x5b4` (model `2nabojniceb.i3d`).
`structs/cartridge.md` §4a.

### 4m. Script-written scalars

Nine scalar fields the script VM writes: `+0x48`, `+0x164`, `+0x2ad9`, `+0x2b14`,
`+0x2d48 .. +0x2d67`, `+0x2edc` / `+0x2ee0`, `+0x2f9c`, `+0x3638` and `+0x3665`. Their
meanings are in §2; the cross-build check is in §8f.

## 5. The constructor and destructor

`game_ctor` (`0x00606750`) and `game_dtor` (`0x00606e20`) are **not** in `g_game.cpp`. They
sit in the `Mission.cpp` translation unit (§8f), out of line, and `mission_init` calls
`game_ctor` straight after `operator_new(0x3698)`.

The sub-object construction order is `G_Camera`, the 100-slot decal pool at `+0x75c`,
`C_rmorf`, `C_sledovacka`, `C_blood_manager` at `+0x2d90`, `C_shvestky`; it settles the
identification (§8f). `game_dtor` unwinds it in reverse, ending at `camera_dtor`, and also
frees the ItemBlur storage that `game_item_blur_done` only empties.

The pursuit-event manager at `+0x2b18` is destroyed through
`folded_vector_pair_free_at_0x04_0x14`, a body shared with the force-feedback manager's
destructor (`structs/pursuit_event.md` §1). The two breakage sub-objects are destroyed
through `glass_shatter_dtor` (`+0x208`, its only receiver) and
`folded_vector_pair_free_at_0x04_0x14_00607500` (`+0x240`), a second copy of the two-vector
body whose other receiver is the `C_using_object` at `+0x29ec`, reached from two unwind
funclets. `game_done` calls their `GameDone`-style methods
(`lea ecx,[ebp+0x208]; call glass_shatter_done` and
`lea ecx,[ebp+0x240]; call debris_done`), which is what identifies them (§6).

## 6. Embedded sub-objects

Every one of these is EMBEDDED, not a pointer, and none has a global of its own — which is
why their translation units contain no data symbols. Each receiver was byte-proven by a
census: every lifecycle callsite is `lea ecx,[reg + <off>]` inside `game_ctor` / `game_init`
/ `game_tick` / `game_done`, and every external producer reaches it as `[obj + 0x24] + <off>`
through the singleton accessor.

| offset | type | evidence | doc |
|---|---|---|---|
| `+0x4c` | **`G_Camera`** | all **13** callsites of the address accessor `game_get_camera` (`0x00425510`) load its result into ECX and immediately call a `G_Camera` method; one resolves to `camera_set_car` = `SetCar__8G_CameraFP5C_car`. Method: locate every `E8` callsite in `.text` and decode *the next call at each* | `structs/camera.md` — 36 of 39 PS2 methods, all body-diffed |
| `+0x208` | **`C_Break_plain_glass`** | 3 lifecycle callsites `lea ecx,[reg + 0x208]`; 7 producers — `car_break_glass`, `crash_obj_ai`, `door_hit`, `human_ai`, `rail_window_hit`, `game_tick_shoot`, `game_new_explosion`. Its `GameDone`-style method is `glass_shatter_done` | `structs/glass_shatter.md` |
| `+0x240` | **`C_Debris`** | 3 lifecycle callsites `lea ecx,[reg + 0x240]`; 4 producers for pieces (`car_explosion`, `car_on_collision_apply_damage`, `crash_obj_ai`, `human_ai`) and 4 for flashes (`airplane_hit`, `car_on_collision_apply_damage` ×2, `crash_obj_ai`). Its `GameDone`-style method is `debris_done` | `structs/debris.md` |
| `+0x26c` | **`C_compass`** | `lea ecx,[esi+0x26c]` in `game_init` and `lea ecx,[ebp+0x26c]` in `game_done` — the only two callsites of `compass_activate` | - |
| `+0x2b18` | **`C_sledovacka`** | the address accessor `game_get_pursuit_event_manager`; `game_ctor` constructs it | `structs/pursuit_event.md` |
| `+0x2da0` | **`C_shvestky`** | the address accessor `game_get_police_manager`; `game_ctor` constructs it | `structs/police_manager.md` |
| `+0x2bc` | **`C_rmorf`** | `lea ecx,[esi+0x2bc]; call random_morph_ctor` in `game_ctor`; `game_init`, `game_tick` and `game_done` call its vtable slots 1, 0 and 2 with `lea ecx,[reg+0x2bc]`; `game_dtor` calls `random_morph_dtor` | `structs/random_morph.md` |
| `+0x29ec` | **`C_using_object`** | every safe callsite of all nine members loads `[reg+0x24] + 0x29ec` — 45 for `using_object_del_object`, 33 for `_add_object`, 24 and 16 for the invalid-item pair. PS2 `+0xde0` | `structs/using_object.md` |
| `+0x2b3c` | **`soundArena`** | address accessor `game_get_sound_arena` (`0x00425550`) | `structs/sound_arena.md` |
| `+0x2d90` | **`C_blood_manager`** | 12 of 12 callsites: `lea ecx,[this+0x2d90]` in `game_init` / `game_tick` / `game_done`, and `mov ecx,[display+0x24]; add ecx,0x2d90` from every `human_*` / `dog_*` / `pedestrian_hit` wound path; `game_ctor` / `game_dtor` construct and free it through the shared folds `folded_vector_ctor_at_0x04` / `folded_vector_free_at_0x04` | `structs/blood.md` |

Three more `C_game` features — per-object ambient shading, the shoot-light pool at `+0x2a48`
and the `I3D_light` cache at `+0x16c` — have their own document, `structs/lightness.md` (§8c).

## 7. The translation unit

`[0x005dc160, 0x005f79a0)`: from the unit's own CRT guard `crt_static_init_guard_005dc160` to the next
unit's `crt_static_init_guard_005f79a0`. Foreign symbols inside the range:

* the static-init quartets `console_log_*` and `console_overlay_*` (`0x005dc190..0x005dc200`; `structs/console.md`);
* `anim_track_load_by_name` (`0x005df650`; `structs/anim_machine.md`), called by the heavy-init block of `game_init`;
* `actor_apply_explosion_damage` (`0x005e7070`; `structs/actor.md`), the `_explosion` helper of `game_new_explosion`;
* `actor_apply_net_guid_table` (`0x005e9490`; `structs/actor.md`);
* `camera_snap_next_update` (`0x005ee210`; `structs/camera.md`).

The accessors outside the range are COMDAT copies emitted by the units that used them first.

## 8. PC vs PS2

### 8a. Inlined into `game_tick`

MSVC folded PS2 `C_game` methods into `game_tick`, so they have no separate PC function (the `inlined` rows of §3c).
Each site is bracketed with an `// ===== INLINED ... =====` banner in the decomp.

| PS2 method | where it went | proof |
|---|---|---|
| `ShootLightTick__6C_gameFUi` | the shoot-light loop over `+0x2a48` | byte-verified; the PS2 walks a pool of 2 records, the PC its 10 |
| `TickEffects__6C_gameFUi` | the per-frame `S_sound_play` walk over `+0x2f8` | statement for statement: the timer gate bypassed while fades are running, the is-playing test, the playback-position test, the three-virtual retire with swap-pop, both fade branches. The sound virtuals sit `0x2c` higher on the PS2 and the inherited frame virtuals `0x14` higher |
| `TickCityMusic__6C_gameFUi` | the `+0x2fb0` block | bannered |
| `TickShadows__6C_gameFv` | - | an empty PS2 stub; nothing to inline |
| `LightCacheTick__6C_gameFUi` | twice: the `light_cache` block of `game_tick`, and `game_recompute_light_cache`, which drops the countdown test | statement for statement against the 388-byte PS2 method, whose callers are `Tick` and `RecomputeLightCache` |
| `SetActiveTrafficP__6C_gameFUi` | the traffic-generator pick over `+0x18c` / `+0x190`; `game_init` runs a second copy with the period plus one, which forces a pick on the first frame | statement for statement: the kind-`0x12` scan, the in-range-first then nearest choice, `SetActState(0)` and the selected byte |
| `TickDriverMan__6C_gameFv` | the steering-wheel loop over `+0x2d7c`: frame = (current steer `car+0x714` / max steer `car+0x6fc` + 1) × 0.5 × the wheel animation's frame count `+0x2d88` / `+0x2d8c` | statement for statement, gated on the detail setting |

`game_tick` also folds in per-frame blocks that are its OWN logic rather than separate
methods: actor visibility (`+0x180`), the level-effect registry sweep (`+0x2c8`), the periodic scream-area pass
(`+0x361c`) and the fake-shadow LOD pass over `+0x290` / `+0x2a0`, which no PS2 method performs
(`structs/fake_shadow.md` §8).

### 8b. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002f8c90 | `TickShadows__6C_gameFUi` | an empty PS2 stub; nothing to inline |
| 0x00305fd0 | `AddInterCityArea__6C_gameFP8I3D_areaPc` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x00306360 | `ClearInterCityAreas__6C_gameFv` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x003063d0 | `TickInterCity__6C_gameFUiPc` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x002fed50 | `InterCityNajeb__6C_gameFv` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x002fece0 | `InterCityVyjeb__6C_gameFv` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x002f21c0 | `VyhjebejTemporaryho__6C_gameFP7C_actor` | PS2-port-only city streaming: `InterCity` appears 0 times in the PC decomp (§8b) |
| 0x00306cf0 | `AddActorForFQL__6C_gameFPCcPCci` | the PC creates a missing save-game actor at once instead of queueing it ¹ |
| 0x00306d70 | `CreateActorsFQL__6C_gameFv` | the PC creates a missing save-game actor at once instead of queueing it ¹ |
| 0x00305460 | `AddDeathHumanFade__6C_gameFP10I3D_objectP9I3D_frameb` | the PS2's alternative to the PC's part arena (`structs/dropout.md`) |
| 0x003057d0 | `ClearDeathHumanFade__6C_gameFv` | the PS2's alternative to the PC's part arena (`structs/dropout.md`) |
| 0x00305970 | `TickDeathHumanFade__6C_gameFUi` | the PS2's alternative to the PC's part arena (`structs/dropout.md`) |
| 0x00305bb0 | `AddFireFlame__6C_gameFRC8S_vectorRC8S_vectorP9I3D_frameP7C_human` | replaced: at this call position in `NewShoot` the PC calls `particle_manager_create_particle` with the weapon's muzzle particle |
| 0x00305f20 | `TickFireFlames__6C_gameFUi` | replaced: nothing on the PC keeps or ticks a flame ring |
| 0x002f7c90 | `AddFakeShadow__6C_gameFP9I3D_frameP7C_actor` | a second shadow system, for people, whose calls the PC twins of every caller drop (`structs/fake_shadow.md` §8) |
| 0x002f81c0 | `DelFakeShadow__6C_gameFP9I3D_frame` | a second shadow system, for people, whose calls the PC twins of every caller drop (`structs/fake_shadow.md` §8) |
| 0x002f8330 | `TickFakeShadows__6C_gameFUi` | a second shadow system, for people, whose calls the PC twins of every caller drop (`structs/fake_shadow.md` §8) |
| 0x00306a60 | `AddNewActorToActive__6C_gameFP7C_actor` | PS2-port-only: every PS2 caller is InterCity code, the PS2-only rail generator or the freeride setup tail (§8b) |
| 0x00306b60 | `DelNewActorFromAll__6C_gameFP7C_actor` | PS2-port-only: every PS2 caller is InterCity code, the PS2-only rail generator or the freeride setup tail (§8b) |
| 0x00306c60 | `IsActorInActList__6C_gameFP7C_actor` | PS2-port-only: every PS2 caller is InterCity code, the PS2-only rail generator or the freeride setup tail (§8b); it has no caller |
| 0x002fb350 | `IsFramePlacement__6C_gameFP9I3D_frame` | PS2-port-only: its one caller is the PS2-only `S_auto_hide_machine` (`structs/camera.md` §7d) |

¹ Each method has exactly one PS2 caller. `AddActorForFQL` is called by
`G_LoadSaveClass::SaveGameLoad` for a saved actor that is not in the scene, and queues it in the
`C_game::S_save_game_fql` vector, whose `insert` / `swap` / `clear` / constructor / destructor
template family has no other user. At that position the PC `loadsave_savegame_load` asks the
driver for a type-9 object and opens the model through the model cache immediately.
`CreateActorsFQL` is called by `Init` between `InterCity_PicaDelActors` and
`G_GarageClass::GameInit`; the PC `game_init` calls `garage_game_init` straight after
`racing_game_init`, with nothing between them.

`TickHydrant` (864 bytes) runs the same three steps per record as `game_tick`'s sweep: a dead
particle is erased, a distant record is destroyed, and an upward ray pauses or resumes emission.
The PS2 destroys beyond 50 units (`2500.0` = 50²), the PC beyond 150.

### 8c. Gutted on the PS2, shipped on the PC

Where the usual direction reverses, the PC body is the only one that exists and there is
nothing to diff against:

* `AddShootLight__6C_gameFR8S_vectorfiUs` and
  `NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` are 8-byte `return;` stubs, so no muzzle
  light is added and no casing is ever ejected on the PS2 — although it keeps both cartridge
  pools and `ClearCartridges__6C_gameFv`;
* `TickShadows`, `SetFrameForShadow` and `DelFrameForShadow` are `return` / `return 0` stubs;
* `LightnessAdd` and `LightnessTick` are stubbed there as well (`structs/lightness.md`), so that band is
  where the PS2 method count is a LOWER bound rather than an upper one; the `I3D_light` cache is live on
  both builds.

### 8d. Container and layout differences

* **`ItemBlurDone`:** the PC clears and RETAINS the vector's capacity; the PS2 frees it. The
  PC storage free lives in `game_dtor` instead.
* **Pool sizes:** cartridges 100 + 30 on PC against 10 + 5 on PS2; scratch decals 100 against
  20; the level-effect record is `0x18` bytes on PC and `0x20` on PS2.
* **Inlined `std::vector<T>::insert`:** five functions (`game_human_talk_add`,
  `game_add_invisible_actor`, `game_dying_human_add`, `game_add_created_frame`,
  `game_collect_scream_area`) inline an insert the PS2 calls out of line, and that alone is
  the entire 3–8× size difference against their PS2 twins — a fact about the toolchain, not
  about the functions. Three erasers (`game_human_talk_del`,
  `game_human_talk_stop_for_human`, `game_del_traffic_signal`) are bannered for the opposite
  reason: the PS2 calls a `compressed_pair_imp<>::first` allocator hook there that MSVC's
  `std::vector` does not have.
* **Folds the other way:** `game_use_light_cache_system` delegates a walk the PS2 does inline,
  and `game_pause_scene_sounds_recursive` exists only on the PC.
* **The box band:** from `CharacterStackPop` to `SaveGameLoadBefore` the PS2 emits 28 entries
  and the PC 25 functions, aligned one to one. The three PS2 entries with no PC function are
  `InterCityVyjeb` and `InterCityNajeb` (§8b) and `gif::vector<C_game::S_box_placement_temp,8>::reserve`:
  `game_box_save_info` grows its stride-`0x20` saved-info vector inline, with `operator_new` and
  the stride-`0x20` copy helpers.

### 8e. The COMDAT container tail

`[0x005f6770, 0x005f79a0)` holds 28 out-of-line copies of header inlines, emitted into this
object file because this is where they were used. A branch-aware receiver census over all 28
puts them on many unrelated owners — `std_vector4_insert_at` alone has **55** callsites across
about 30 — so none of them may carry a class-qualified name.

### 8f. Cross-build identification notes

* **Singleton and code span:** the PS2 singleton is `*(int *)(act_mission + 0x20)` (PC `+0x24`),
  and PS2 `C_game` code spans `0x002ea010`–`0x00306d70`.
* **Traffic signals (§4d):** there is no `AddSemaphore` on either build.
* **Broken models (§2):** PS2 `CreateCrashModel` pushes the broken model onto `C_game+0x10` and
  switches it off, exactly as `crash_obj_create_from_model` does on the PC.
* **City music (§4h):** `CityMusicOff` releases its two channel objects at once; the PC destroys
  the outgoing track and fades the incoming one. Both PC callers sit where the PS2 calls it — the
  music-clip script arm, statement for statement, and `CityMusicPaused` — so the twin is
  STRUCTURAL. The PS2 `Done` calls it as well; `game_done` has no call at that position.
* **Box placement (§4k):** three of the PS2 verbs (`GetBox`, `GetBoxDest`, `GetNextDest`)
  describe neither the return value nor the action of the twin bodies.
* **Script-written scalars (§4m):** each is body-diffed against the PS2 `C_program::Process` arm
  for the same command; the PS2 offsets are in §2.
* **Constructor and destructor (§5):** both builds place them in `Mission.cpp`. The PS2 has no
  `__ct__6C_gameFv`; it inlines the identical constructor into `Init__9C_missionFP10I3D_driver`
  and the identical destructor into `UnInit__9C_missionFv`, and its
  `std::__vector_deleter<C_game::S_*>` helpers are emitted in the same PS2 band. The PC
  sub-object construction order is the same sequence the PS2 emits inline, which settles the
  identification without a symbol to diff against.
* **The car gun (§2, `+0x2f50..+0x2f78`):** the PS2 block is `+0x1368..+0x1390`, one delta for all
  of it, and `DoubleKiss__6C_gameFUi` matches `game_car_gun_tick` statement for statement. Neither
  build creates the frame or stores the four barrel slots, and neither has a writer of gate bit
  `0x8`; the PS2 `Done` releases the same pair that `game_done` releases at `+0x2f54`.
* **Vectors with no producer (§2v, §2y):** the PS2 symbol table has `clear`, the deleter's
  constructor and destructor for `S_find_actor` and no `insert`, and it has `ItemBlurDone`,
  `ItemBlurTick` and `ItemBlurDelItem` and no adder, as on the PC. `actor_reg` and `guid` follow the
  argument order of `FindActorLater__9C_programFiUi`.

### 8g. PC and PS2 field offsets

The deltas are **not constant**: the packing differs, so pair per field and never extrapolate a delta across a
band. Each row was re-measured: the PS2 offset occurs in the body named in the last column, the twin of a PC
function that touches the PC field.

| PC offset | PC field | PS2 offset | Checked in |
|---|---|---|---|
| +0x0 | `frame_placements` | +0x00 / +0x04, 0x40-byte records, key at +0x30 | `AI__9C_railwayFUi` |
| +0x10 | `broken_models` | +0x10 / +0x14 | `Done__6C_gameFv` |
| +0x20 | `created_frames` | +0x18 / +0x1c / +0x20 | `AddCreatedFrameToVector__6C_gameFP9I3D_frame` |
| +0x44 | `mission` | +0x5c | `Init__9C_missionFP10I3D_driver` |
| +0x48 | `mission_end_action` | +0x60 | `Init__6C_gameFbb` |
| +0xe4 | `player` | +0x150 | `BoxPlacementReposeDestPlayer__6C_gameFv` |
| +0xec | `active_actors` | +0x158 | `AddTemporaryActor__6C_gameFP7C_actor` |
| +0xfc | `inactive_actors` | +0x164 | `AddTemporaryActor__6C_gameFP7C_actor` |
| +0x10c | `act_state_dirty` | +0x170 | `ChangeActState__6C_gameFv` |
| +0x124 | `temp_actors` | +0x180 / +0x184 / +0x188 | `AddTemporaryActor__6C_gameFP7C_actor` |
| +0x164 | `actor_lightness_on` | +0x1b4 | `Tick__6C_gameFUi` |
| +0x18c | `traffic_pick_timer` | +0x1d4 / +0x1d8 | `Init__6C_gameFbb` |
| +0x194 | `signal_elapsed_ms` | +0x1dc .. +0x1e8 | `Init__6C_gameFbb` |
| +0x1a8 | `signals` | +0x1f0 / +0x1f4 (PS2 count/begin) | `DelSemaphore__6C_gameFP9I3D_frame` |
| +0x1c4 | `telephones` | +0x208 / +0x20c (PS2 count/begin) | `LoadTelephones__6C_gameFv` |
| +0x1d8 | `shoots` | +0x210 / +0x214, stride 0x70 | `Done__6C_gameFv` |
| +0x1e8 | `signal_model_ids` | +0x21c (count +0x220) | `CreateSignalMarkIDX__6C_gameFv` |
| +0x1f4 | `taxi_points` | +0x228 | `LoadTaxiPoints__6C_gameFv` |
| +0x204 | `taxi_passenger` | +0x234 | `Init__6C_gameFbb` |
| +0x284 | `shadow_meshes` | +0x2c4 / +0x2c8 | `DoneIntern__6C_gameFv` |
| +0x304 | `sound_walk_timer` | +0x32c / +0x330 | `Init__6C_gameFbb` |
| +0x30c | `sound_handle_next` | +0x334 | `Init__6C_gameFbb` |
| +0x310 | `sounds_fading` | +0x338 | `Init__6C_gameFbb` |
| +0x3a8 | `cartridges_a` | +0x36c, **10** entries | `ClearCartridges__6C_gameFv` |
| +0x53c | `cartridges_b` | +0x398, **5** entries | `ClearCartridges__6C_gameFv` |
| +0x29ec | `using_object` | +0xde0 | `BoxPlacementDestSetOU__6C_gameFbP9I3D_frame` |
| +0x2b14 | `frame_counter` | +0xe5c | `Init__6C_gameFbb` |
| +0x2b3c | `sound_arena` | +0xe80 | `InvalidateActor__6C_gameFP7C_actorb` |
| +0x2d48 | `script_events` | +0x1194 .. +0x11b3 | `Init__6C_gameFbb` |
| +0x2d6c | `npc_kill_events` | +0x11b8 / +0x11bc (PS2 count/begin) | `Do_NPCKillEvent__6C_gameFPc` |
| +0x2d7c | `driver_men` | +0x11c4 / +0x11c8 | `DelDriverMan__6C_gameFP9I3D_model` |
| +0x2d88 | `wheel_frames_car` | +0x11cc / +0x11d0 | `Init__6C_gameFbb` |
| +0x2e84 | `boxes` | +0x12a0 / +0x12a4 | `BoxPlacementAdd__6C_gameFP9I3D_framei` |
| +0x2e94 | `box_dests` | +0x12ac / +0x12b0 | `BoxPlacementDestAdd__6C_gameFP9I3D_framei` |
| +0x2edc | `race_state` | +0x1304 / +0x1308 | `Init__6C_gameFbb` |
| +0x2ee8 | `death_queue` | +0x130c .. +0x1314 | `DeathActorAdd__6C_gameFP7C_actor` |
| +0x2f1c | `media` | +0x1340 / +0x1344 | `CreateIShow__6C_gameFUi` |
| +0x2f50 | `car_gun_frame` | +0x1368; the block to `+0x2f78` is `+0x1368..+0x1390` | `DoubleKiss__6C_gameFUi` |
| +0x2f9c | `heavy_init` | +0x13ac | `Done__6C_gameFv` |
| +0x2fb0 | `city_music_timer` | …/+0x13c4/+0x13c8/+0x13d0 — the PS2 uses TWO channels | `Init__6C_gameFbb` |
| +0x2fd8 | `scratches` | +0x13d8, **20** records | `Init__6C_gameFbb` |
| +0x3638 | `rain_buffer_size` | +0x1534 | `Init__6C_gameFbb` |
| +0x363e | `score_on` | +0x153a | `ScoreSet__6C_gameFi` |
| +0x3640 | `score` | +0x153c | `ScoreSet__6C_gameFi` |
| +0x3665 | `script_mission_end` | +0x154d | `Init__6C_gameFbb` |
| +0x3668 | `car_sound_budget` | +0x1550 | `Init__6C_gameFbb` |
| +0x3684 | `item_blur_on` | +0x1568 | `ItemBlurDone__6C_gameFv` |
| +0x368c | `item_blurs` | +0x156c / +0x1570 / +0x1574, stride 0x30 | `Init__6C_gameFbb` |

## 9. Related

* `structs/mission.md` — the owner, `C_mission`.
* `structs/camera.md`, `structs/glass_shatter.md`, `structs/debris.md`, `structs/random_morph.md`,
  `structs/using_object.md`, `structs/pursuit_event.md`, `structs/sound_arena.md`, `structs/blood.md`,
  `structs/police_manager.md` — the embedded sub-objects (§6).
* `structs/lightness.md`, `structs/fake_shadow.md` — two `C_game` features with their own documents.
* `structs/particle.md`, `structs/cartridge.md`, `structs/fire.md`, `structs/smoke_trail.md`,
  `structs/grenade.md` — the actors and effects it spawns.
* `basic/artifacts/cheats.md` — the cheat table behind `cheat_buffer` and `cheat_flags`.

## 10. Open items

1. **PS2 field pairs not re-measured.** These pairs of the §8g table's source could not be found in the PS2
   twin of any function that touches the PC field, so they are not listed there: `+0x290` / +0x2d0 / +0x2dc, `+0x2f8` / +0x320 / +0x324 / +0x328.
2. **`cheat_flags` bits `0x1`, `0x2`, `0x4`, `0x8`, `0x20`, `0x40`** have readers and no writer located in the
   decomp; `game_init` clears the word and `game_cheat_check` toggles only `0x10`, `0x80` and `0x100`.
3. **The census of §2 does not see a `C_game *` held in a member or passed as an argument.** Cells are a lower
   bound outside the class's own functions. `mission` (`+0x44`) has no access from the class's code, and
   `camera_human` (`+0xe8`) no reader there or in the functions that call a `game_` method; other bases at these
   displacements were not read.
4. **`unk_365c` (`+0x365c`)** sits between the `9sheet.i3d` and `9money.i3d` slots; only 0 is ever stored, while
   `game_done` hides it and `game_done_intern` releases it when set.
5. **Record bytes with no decoded meaning or no producer:** `S_paused_sound.unk_08` (the sound's `+0x188`; the
   meaning is inside `LS3DF.dll`); `unk_2f54` (two released pointers that no code creates); no code appends an
   `S_find_actor` or an `S_item_blur`, so their read-only fields have no writer; `S_find_actor.actor_reg` and
   `.guid` are named from the argument order of `program_find_actor_later`, whose body was not read.
6. **In game:** the water-fall script event's exact trigger is not verified.
