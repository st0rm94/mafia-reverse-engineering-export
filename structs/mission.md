# `C_mission` — the mission singleton

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-11-01 15:38 +0100

The object every `g_act_mission` dereference in the decomp lands on: one file-scope object of
`Main.cpp` at **`0x006f9440`** that owns the open scene, the `C_game`, the scene's actor and script
lists, the quality settings and the caches that carry actors across a mission change. Its own
translation unit `Mission.cpp` also hosts the scene-file reader, a stack-local frame-name map and
several other classes' code (§3d, §6).

| | |
|---|---|
| PC prefix | `mission_` (`mission_settings_` for the settings accessors) |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0xdc`: the last vector ends at `+0xdc`, and `g_i3d_driver` begins at `+0xe0` (`0x006f9520`) |
| PC constructor | `mission_ctor` @ `0x005fe5b0`, reached by `mission_static_ctor`'s tail jump |
| PC translation unit | `[0x005fe4f0, 0x006083c0)`, `Mission.cpp`: 96 starts, all named, between its opening guards `0x005fe4f0` / `0x005fe520` and its closing pair at `0x00608390`; the static quartet sits in `Main.cpp` and seven header inlines in the `C_program` unit's tail (§3b) |
| PS2 class | `C_mission` (tag `9C_mission`), 68 methods: 35 placed in §3, 33 PS2-only (§8) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_mission {                                // 0xdc bytes @ 0x006f9440
    /* +0x00 */ uint32_t scene_identity[2];               // [H] chunk 0xAFFF of the scene file
    /* +0x08 */ int32_t  next_actor_guid;                 // [H] the actor net-GUID counter
    /* +0x0c */ uint8_t  is_open;                         // [H] a mission is open
    /* +0x10 */ I3D_scene *scene;                         // [H] the LS3D scene
    /* +0x14 */ std::vector<S_anim_model> anim_models;    // [H] the animatable models (§1b)
    /* +0x24 */ C_game  *game;                            // [H] the C_game singleton
    /* +0x28 */ C_tshift *uv_scroll;                      // [H] the UV-scroll animator
    /* +0x2c */ C_web_path *web_path;                     // [H] the navigation graph
    /* +0x30 */ C_roads *roads;                           // [H] the road network
    /* +0x34 */ std::vector<C_actor *> actors;            // [H] every actor of the scene
    /* +0x44 */ std::vector<C_actor *> transient_actors;  // [H] the actors game_done destroys
    /* +0x54 */ std::vector<C_program *> global_programs; // [H] the global scripts
    /* +0x64 */ void    *camera_sector;                   // [H] the sector the camera is in
    /* +0x68 */ char    *mission_name;                    // [H] operator new[]-owned
    /* +0x6c */ C_particle_manager *particle_manager;     // [H] the particle manager
    /* +0x70 */ int32_t  next_mission_number;             // [H] the pending mission number
    /* +0x74 */ int32_t  mission_number;                  // [H] the current mission number
    /* +0x78 */ int32_t  frame_skip_count;                // [H] frames left without a present
    /* +0x7c */ uint8_t  restore_sound_volume;            // [H] restore the volume when it runs out
    /* +0x80 */ std::vector<S_t_transparent> fade_ins;    // [H] the pending fade-ins (§1c)
    /* +0x90 */ S_mission_settings settings;              // [H] the quality settings (§1a)
    /* +0xac */ std::vector<S_frame_actor_buffer> carried_frames;     // [H] §1d
    /* +0xbc */ std::vector<C_actor *> carried_active_actors;         // [H] parked live actors
    /* +0xcc */ std::vector<S_actor_sound_buffer> carried_actor_sounds; // [H] §1e
} C_mission;
```

Every `std::vector` is the MSVC `{alloc byte, begin, end, cap}` quadruple; `mission_ctor` writes the
allocator byte at the first offset and zeroes the three pointers, and `mission_dtor` frees them in
descending offset order.

**`scene_identity[2]` — why two.** `mission_open_bin` copies the 8-byte payload of the scene
file's chunk `0xAFFF` into `+0x00..+0x07` as one block, and the network code compares the two
dwords with a peer's pair (`race_net_load_sync`, `net_session_post_scene_done`,
`net_deactivate_actors_of_peer`, `net_session_handle_event_msg`, `net_session_remove_peer`); two
peers are in the same scene when both dwords match. The entries carry no separate meaning.

**`camera_sector`.** `game_tick` copies `I3D_scene+0x13c`, the scene's current sector, into
`+0x64` every frame; `pedestrian_tick` compares a frame's own sector (`I3D_frame+0x13c`) with it,
the "same sector as the camera" test of `structs/i3d_frame.md` §2, and `human_hit` compares it
with the sector it stored on its previous call.

### 1a. `S_mission_settings` — the quality settings, 0x1c bytes at `+0x90`

```c
typedef struct S_mission_settings {     // at C_mission+0x90
    /* +0x00 */ float details_scale;     // [H] clamped 0.4 .. 2.0
    /* +0x04 */ float draw_distance_scale; // [H] clamped 0.3 .. 1.0
    /* +0x08 */ float view_distance;     // [H] world units, default 250.0
    /* +0x0c */ int32_t effects_quality; // [H] 0..2, the weather detail level
    /* +0x10 */ int32_t sounds_quality;  // [H] 0..2
    /* +0x14 */ int32_t shadow_quality;  // [H] an index 0..5 into {0, 1, 3, 6, 9, 15}
    /* +0x18 */ int32_t particles_quality; // [H] 0..2, the hydrant detail level
} S_mission_settings;
```

The block has its own accessor family, and the family's receiver is byte-proven: an ECX-tracking
census of every callsite of the seven accessors finds **38 callsites, 0 unsafe**, and every
receiver is `C_mission + 0x90` (`mov ecx,[0x65115c]; add ecx,0x90`, or `lea ecx,[reg+0x90]` where
`reg` holds the mission pointer).

Two of the six quality values were decoded independently of the accessors and agree: `+0x9c` is
the **weather detail** level (it scales the weather particle count by `/0x14`, `>>3`, or unscaled)
and `+0xa8` the **hydrant detail** level (broken hydrants spray only at level 2 or above). Both are
confirmed in game: hydrants spray only at level 2, and at level 0 the weather shows a twentieth of
its particles. The block is mirrored into `g_ls3d_setup + 0x27 … +0x3b` on the way out to the
registry and restored from it on the way in (`structs/app_bootstrap.md`).

### 1b. `S_anim_model` — one animatable model, 8 bytes

```c
typedef struct S_anim_model {
    /* +0x00 */ I3D_model *model;        // [H] the registered model
    /* +0x04 */ uint8_t    per_game;     // [H] registered for the lifetime of the game
} S_anim_model;
```

### 1c. `S_t_transparent` — one pending fade-in, 8 bytes

```c
typedef struct S_t_transparent {
    /* +0x00 */ I3D_objects *object;     // [H] the object being faded in, referenced
    /* +0x04 */ int16_t      elapsed_ms; // [H] time since the fade started
    /* +0x06 */ int16_t      duration_ms; // [H] the fade's length, at least 82 ms
} S_t_transparent;
```

### 1d. `S_frame_actor_buffer` — one parked actor or model, 0x10 bytes

```c
typedef struct S_frame_actor_buffer {
    /* +0x00 */ C_actor   *actor;        // [H] the parked actor, null for a bare model
    /* +0x04 */ I3D_frame *frame;        // [H] the actor's frame, or the bare model
    /* +0x08 */ char      *name;         // [H] the model file's basename, heap copy
    /* +0x0c */ uint8_t    owns_frame;   // [H] release the frame with the actor
} S_frame_actor_buffer;
```

### 1e. `S_actor_sound_buffer` — one parked actor sound, 0xc bytes

```c
typedef struct S_actor_sound_buffer {
    /* +0x00 */ C_actor   *actor;        // [H] the owning actor
    /* +0x04 */ int32_t    sound_id;     // [H] the caller's key for the sound
    /* +0x08 */ I3D_sound *sound;        // [H] the parked sound, released on clear
} S_actor_sound_buffer;
```

## 2. Field evidence

Measured by a census of every `this+N` access in the `mission_*` functions, a census of every
`g_act_mission + N` access across the decomp (attributed by containing function), and body reads
of `mission_ctor`, `mission_dtor`, the carry-cache functions and the `+0x64` / `+0x78` users. The
heavily used pointer fields carry a count; the functions are listed by the census.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | scene_identity | H | `mission_ctor` (0), `mission_open_bin` (chunk `0xAFFF`) | `race_net_load_sync`, `net_session_post_scene_done`, `net_deactivate_actors_of_peer`, `net_session_handle_event_msg`, `net_session_remove_peer` | §6 |
| +0x08 | next_actor_guid | H | `mission_open` (0, before the teardown), `actor_init` (pre-increment) | `actor_init` | stamps `C_actor+0x18` (`structs/actor.md` §7) |
| +0x0c | is_open | H | `mission_ctor` (0), `mission_open` (1), `mission_close` (0) | `mission_close` | `mission_close` does nothing while it is clear |
| +0x10 | scene | H | `mission_ctor` (0), `mission_init`, `mission_uninit` | `mission_init`, `mission_uninit`, `mission_open`, `mission_open_bin`, `mission_close`, `mission_tick`, `mission_update_settings`, `mission_restore_settings`, `folded_get_dword_0x10` (29 callsites), and 165 other functions | `structs/i3d_frame.md` §1a |
| +0x14 | anim_models | H | `mission_ctor`, `mission_add_anim_model`, `mission_del_anim_model`, `mission_clear_game_anim_models`, `mission_close` | the same, `mission_tick` | §5a |
| +0x24 | game | H | `mission_init` (`game_ctor`), `mission_uninit` | `act_mission_get_game` (328 callsites), `mission_init`, `mission_uninit`, `mission_open`, `mission_close`, `mission_tick`, and 255 other functions | `structs/game.md` |
| +0x28 | uv_scroll | H | `mission_ctor` (0), `mission_init`, `mission_uninit` | `mission_get_uv_scroll`, `mission_init`, `mission_uninit`, `mission_close`, `mission_tick` | `structs/uv_scroll.md` |
| +0x2c | web_path | H | `mission_init`, `mission_uninit` | `mission_init`, `mission_uninit`, `mission_close`, and 77 other functions | `structs/web_path.md` |
| +0x30 | roads | H | `mission_init`, `mission_uninit` | `mission_init`, `mission_uninit`, `mission_close`, and 19 other functions | `structs/roads.md` |
| +0x34 | actors | H | `mission_ctor`, `mission_dtor`, `mission_add_actor`, `mission_unload_actor`, `mission_del_actor`, `mission_del_actors`, `mission_del_not_me_actors` | the same, `mission_is_persistent_actor`, `mission_find_actor_by_name`, `mission_get_actor_list`, and 13 other functions ¹ | dwords |
| +0x44 | transient_actors | H | `mission_ctor`, `mission_dtor`, `mission_add_actor`, `mission_unload_actor`, `mission_del_actor`, `mission_del_actors`, `mission_del_not_me_actors` | `mission_add_actor`, `mission_unload_actor`, `mission_del_actor`, `mission_del_not_me_actors`, `mission_is_persistent_actor` | §5a |
| +0x54 | global_programs | H | `mission_ctor`, `mission_dtor`, `mission_open_bin` (chunk `0xAE50`) | `mission_global_program_game_init`, `mission_global_program_game_done`, `mission_global_program_run`, `mission_close` | `C_program *` |
| +0x64 | camera_sector | H | `game_tick` | `human_hit`, `pedestrian_tick` | §1 |
| +0x68 | mission_name | H | `mission_ctor` (0), `mission_open`, `mission_uninit` | `mission_open`, `mission_uninit`, `check_seg_clusters_load`, `gui_load_progress_ctor`, `loadsave_savegame_load`, `loadsave_savegame_save`, `particle_manager_load_static_particles`, `random_morph_mission_start` | - |
| +0x6c | particle_manager | H | `mission_init`, `mission_uninit` | `mission_get_particle_manager`, `mission_init`, `mission_uninit`, `net_session_handle_event_msg` | not `C_game+0x6c` (`structs/particle.md` §4) |
| +0x70 | next_mission_number | H | `mission_set_next_mission_number`, `engine_main`, `engine_game_loop`, `game_savegame_load_before`, `loadsave_freeride_setup_after`, `race_run` | `mission_commit_next_mission_number`, `game_savegame_load_before`, `loadsave_freeride_setup_after`, `race_run` | §5d |
| +0x74 | mission_number | H | `mission_commit_next_mission_number` | `mission_get_mission_number`, `game_save_game_save` | §5d |
| +0x78 | frame_skip_count | H | §5d | `engine_game_loop`, `race_game_loop`, `race_game_loop_net`, `mission_get_frame_skip_count`, `mission_tick`, `human_do_change_weapon`, `human_weapon_toggle_held`, `race_winners_replay_begin`, `race_winners_replay_end`, `recorder_stop`, `recorder_end_sequence_and_restore_scene` | §5d |
| +0x7c | restore_sound_volume | H | `mission_open` (1), `mission_tick` (0) | `mission_tick` | §5d |
| +0x80 | fade_ins | H | `mission_ctor`, `mission_dtor`, `mission_init`, `mission_fade_in_object`, `mission_finish_object_fade_ins`, `mission_tick`, `mission_close` | the same | §5b |
| +0x90 | settings | H | `mission_ctor` (the defaults), the setters of §3c, `mission_open_bin` (`view_distance`) | §2a | §1a |
| +0xac | carried_frames | H | `mission_ctor`, `mission_dtor`, `mission_actor_cache_store`, `mission_model_cache_store`, `mission_actor_cache_take`, `mission_model_cache_take`, `mission_carry_caches_clear` | the same | §5c |
| +0xbc | carried_active_actors | H | `mission_ctor`, `mission_dtor`, `mission_active_actor_store`, `mission_active_actor_drop`, `mission_active_actor_take`, `mission_carry_caches_clear` | the same | §5c |
| +0xcc | carried_actor_sounds | H | `mission_ctor`, `mission_dtor`, `mission_actor_sound_cache_store`, `mission_actor_sound_cache_take`, `mission_carry_caches_clear` | the same | §5c |

¹ `event_msg_actor_guid_table_serialize`, `game_done`, `game_init`, `game_tick`,
`game_update_controls`, `human_track_apply_pose_at_time`, `net_session_build_actor_guid_table`,
`player_do_action_button_in_car`, `police_manager_apply_state`, `pumper_script_can_pump`,
`pursuit_event_init`, `railway_game_init`, `traffic_car_game_init`; `loadsave_savegame_load`
takes the vector's address.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | details_scale | H | `mission_ctor` (1.0), `mission_settings_set_details_scale` | `mission_update_settings`, `engine_main`, `gui_options_audio_video_ctor` | - |
| +0x04 | draw_distance_scale | H | `mission_ctor` (1.0), `mission_settings_set_draw_distance_scale` | `mission_update_settings`, `engine_main`, `gui_options_audio_video_ctor` | - |
| +0x08 | view_distance | H | `mission_open_bin` (250.0, then scene chunk `0x3011`) | `mission_update_settings`, `mission_restore_settings` | handed to the active camera with `0.05f`; no accessor |
| +0x0c | effects_quality | H | `mission_ctor` (2), `mission_settings_set_effects_quality` | 25 functions, among them `game_apply_weather_detail_level`, `game_tick`, `car_update`, `human_update` | weather detail |
| +0x10 | sounds_quality | H | `mission_ctor` (2), `mission_settings_set_sounds_quality` | 14 functions, among them `game_new_explosion`, `game_new_fire`, `grenade_ai_tick`, `door_set_state` | - |
| +0x14 | shadow_quality | H | `mission_ctor` (3), `mission_settings_set_shadow_quality` | `mission_settings_get_shadow_quality_idx` (called by `game_init`, `game_tick`, `engine_main` and the options screen) | - |
| +0x18 | particles_quality | H | `mission_ctor` (2), `mission_settings_set_particles_quality` | `folded_get_dword_0x18`, and 19 functions, among them `game_add_hydrant`, `game_new_small_fires`, `crash_obj_ai`, `human_hit` | hydrant detail |

The setters are called by `gui_options_audio_video_tick` and `engine_main`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | model | H | `mission_add_anim_model` | `mission_del_anim_model`, `mission_clear_game_anim_models`, `mission_tick`, `mission_close` | - |
| +0x04 | per_game | H | `mission_add_anim_model` | `mission_clear_game_anim_models` | §5a |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | object | H | `mission_fade_in_object` | `mission_tick`, `mission_finish_object_fade_ins`, `mission_close` | released when the fade ends |
| +0x04 | elapsed_ms | H | `mission_fade_in_object` (0), `mission_tick` | `mission_tick` | - |
| +0x06 | duration_ms | H | `mission_fade_in_object` | `mission_tick` | the setter's floor `0x52` rejects a duration under 82 ms |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | actor | H | `mission_actor_cache_store`, `mission_model_cache_store` (0) | `mission_actor_cache_take`, `mission_model_cache_take`, `mission_carry_caches_clear`, `mission_actor_cache_store` (duplicate test) | - |
| +0x04 | frame | H | `mission_actor_cache_store` (`C_actor+0x68`), `mission_model_cache_store` | `mission_model_cache_take` (returned), `mission_carry_caches_clear` (released) | - |
| +0x08 | name | H | `mission_actor_cache_store`, `mission_model_cache_store` | `mission_actor_cache_take`, `mission_model_cache_take` (`_stricmp` key) | split out of `I3D_frame+0x154` with `strrchr('\\')` |
| +0x0c | owns_frame | H | `mission_actor_cache_store` (its bool argument: 1 from `traffic_car_game_done`, 0 from `garage_game_done` and `police_manager_game_done`), `mission_model_cache_store` (0) | `mission_carry_caches_clear` | when set, clearing an actor record also releases `frame` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | actor | H | `mission_actor_sound_cache_store` | `mission_actor_sound_cache_take` | - |
| +0x04 | sound_id | H | `mission_actor_sound_cache_store` | `mission_actor_sound_cache_take` | the take matches actor and id |
| +0x08 | sound | H | `mission_actor_sound_cache_store` | `mission_actor_sound_cache_take`, `mission_carry_caches_clear` (released) | - |

### 2a. The settings readers

`gui_options_audio_video_ctor` / `_tick` read every quality value; `engine_main` reads and writes
the block around the registry mirror. The effects, sounds and particles levels are read directly
through `g_act_mission + 0x9c` / `+0xa0` / `+0xa8` by the effects code of `C_game`, `C_human`,
`C_car`, `C_dog`, the pedestrians and the railway; the shadow level only through
`mission_settings_get_shadow_quality_idx`.

## 3. Method family

Coverage: **35 of the 68** PS2 `C_mission` methods are mapped below (all CONFIRMED), plus the
static helper `cbSectorBitFill` of the nested `S_hlp`, three free functions of the PS2 band with
twins (§3d) and three inlined into `mission_open_bin`. The other **33** methods are PS2-only (§8):
26 belong to `InterCity_*` and the FQL / animation-path families of the PS2 port, three to the
garage, and `ClearFramesRefBuff` is a container difference.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005f7cd0 | `mission_static_init` | `g_mission` | - | - | - | `Main.cpp`'s `.CRT$XCU` entry |
| 0x005f7ce0 | `mission_static_ctor` | `g_mission` | - | - | - | `mov ecx, 0x006f9440; jmp mission_ctor` |
| 0x005f7cf0 | `mission_register_atexit_dtor` | `g_mission` | - | - | - | `atexit(&mission_static_dtor)` |
| 0x005f7d00 | `mission_static_dtor` | `g_mission` | - | - | - | `mov ecx, 0x006f9440; jmp mission_dtor` |
| 0x005fe4f0 | `crt_static_init_guard_005fe4f0` | the translation unit | - | - | - | the unit's first `.CRT$XCU` initializer |
| 0x005fe510 | `crt_atexit_register_005fe510` | the translation unit | - | - | - | its `atexit` partner |
| 0x005fe520 | `crt_static_init_guard_005fe520` | the translation unit | - | - | - | the second initializer |
| 0x005fe540 | `crt_atexit_register_005fe540` | the translation unit | - | - | - | its `atexit` partner |
| 0x005fe5b0 | `mission_ctor` | instance | `__ct__9C_missionFv` | 0x0037a520 | CONFIRMED | zeroes the leading fields, builds the eight vectors, writes the six settings defaults (§8a) |
| 0x005fe6b0 | `mission_dtor` | instance | `__dt__9C_missionFv` | 0x0037a750 | CONFIRMED | `mission_uninit`, then frees every vector in descending offset order |
| 0x00607610 | `frame_map_ctor` | the stack-local frame map of `mission_open_bin` | - | - | - | constructs the `std::map<C_str, I3D_frame*>` (§8e) |
| 0x00608390 | `crt_static_init_guard` | the translation unit | - | - | - | the unit's closing initializer: tests bit 0 of `g_crt_static_init_flags` |

### 3b. Accessors

The seven `0x005c86xx` accessors are header inlines out-lined into the `C_program` unit's tail.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425390 | `act_mission_get_game` | - | - | - | returns `+0x24`; 328 callsites (§8) |
| 0x004253a0 | `folded_get_dword_0x10` | - | - | - | returns `+0x10`, the `I3D_scene`; 29 of its callsites have the receiver `mov ecx,[0x65115c]`. One ICF-folded address that also serves `C_actor` and `C_sound_table` (`structs/actor.md` §3b) |
| 0x005a17d0 | `folded_get_dword_0x18` | - | - | - | returns `particles_quality` when the receiver is the settings block; ICF-folded with `C_actor`'s network GUID reader (`structs/net_event_msg.md` §6c) |
| 0x005c8640 | `mission_get_actor_list` | - | - | - | returns the address of `actors`; 3 callsites in `program_process_command` |
| 0x005c8650 | `mission_get_particle_manager` | - | - | - | returns `+0x6c`; 2 callsites in `program_process_command` |
| 0x005c8660 | `mission_set_next_mission_number` | - | - | - | writes `+0x70` |
| 0x005c8670 | `mission_get_mission_number` | - | - | - | reads `+0x74` |
| 0x005c8690 | `mission_set_frame_skip_count` | - | - | - | writes `+0x78` |
| 0x005c86a0 | `mission_get_frame_skip_count` | - | - | - | reads `+0x78` |
| 0x00606390 | `mission_get_uv_scroll` | - | - | - | returns `+0x28` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8680 | `mission_commit_next_mission_number` | - | - | - | `+0x70` → `+0x74`, at `change_mission` |
| 0x005fe7d0 | `mission_init` | `Init__9C_missionFP10I3D_driver` | 0x0037a900 | CONFIRMED | builds the engine sub-objects and the `C_game` |
| 0x005fe950 | `mission_uninit` | `UnInit__9C_missionFv` | 0x0037b310 | CONFIRMED | tears them down |
| 0x005feac0 | `mission_add_anim_model` | `AddAnimModel__9C_missionFP9I3D_modelb` | 0x0037bca0 | CONFIRMED | registers an animatable model (§5a) |
| 0x005fecb0 | `mission_del_anim_model` | `DelAnimModel__9C_missionFP9I3D_model` | 0x0037bd60 | CONFIRMED | drops one |
| 0x005fed10 | `mission_clear_game_anim_models` | `ClearGameAnimModels__9C_missionFv` | 0x0037bdc0 | CONFIRMED | purges the `per_game` ones |
| 0x005fed70 | `mission_create_actor` | `CreateActor__9C_missionFi` | 0x0037be70 | CONFIRMED | the kind → `operator_new` → ctor factory |
| 0x005ff400 | `mission_add_actor` | `AddActor__9C_missionFP7C_actorb` | 0x0037c300 | CONFIRMED | `(actor, transient)` (§5a) |
| 0x005ff5e0 | `mission_unload_actor` | `UnloadActor__9C_missionFP7C_actor` | 0x0037c420 | CONFIRMED | unregisters without destroying |
| 0x005ff690 | `mission_del_actor` | `DelActor__9C_missionFP7C_actor` | 0x0037c520 | CONFIRMED | unregisters and destroys |
| 0x005ff7a0 | `mission_del_actors` | `DelActors__9C_missionFv` | 0x0037c690 | CONFIRMED | destroys all |
| 0x005ff860 | `mission_del_not_me_actors` | `DelNotMeActors__9C_missionFv` | 0x0037c790 | CONFIRMED | destroys the transient set; its one caller is `game_done` |
| 0x005ff980 | `mission_is_persistent_actor` | - | - | - | survives `game_done`? PC-only; its one caller is `garage_game_done` |
| 0x005ff9f0 | `mission_find_actor_by_name` | `FindActorByName__9C_missionFPCc` | 0x0037c8f0 | CONFIRMED | walks `actors` backwards and compares each frame's name |
| 0x005ffa80 | `mission_global_program_game_init` | `GlobalProgramGameInit__9C_missionFv` | 0x0037c9b0 | CONFIRMED | `Init` every global script |
| 0x005ffab0 | `mission_global_program_game_done` | `GlobalProgramGameDone__9C_missionFv` | 0x0037ca20 | CONFIRMED | `Done` every global script |
| 0x005ffae0 | `mission_global_program_run` | `GlobalProgramRun__9C_missionFPc` | 0x0037ca90 | CONFIRMED | invokes a named subroutine |
| 0x005ffb40 | `mission_close` | `Close__9C_missionFv` | 0x0037cb40 | CONFIRMED | closes whatever is open; calls `mission_carry_caches_clear`. A banner marks the PS2-only `ClearFramesRefBuff` rank (§8e) |
| 0x005ffd40 | `mission_tick` | `Tick__9C_missionFi` | 0x0037cfe0 | CONFIRMED | one frame: the fade-ins, the anim models, the sound-restore latch |
| 0x005fff30 | `mission_open` | `Open__9C_missionFPCcbUib` | 0x0037d3d0 | CONFIRMED | opens a mission by name |
| 0x006005a0 | `mission_open_bin` | `OpenBin__9C_missionFPCcUi` | 0x0037e840 | CONFIRMED | the scene `.bin` reader, 16 KB (§6) |
| - | - | `ReadVector3__FR7C_chunk` | 0x0037e320 | inlined | in `mission_open_bin` at chunks `0x0020`, `0x0026`, `0x002c`, `0x002d` (§8d) |
| - | - | `ReadQuatOld__FR7C_chunk` | 0x0037e480 | inlined | in `mission_open_bin` at chunk `0x0022` |
| - | - | `ReadBBoxOld__FR7C_chunkR8I3D_bbox` | 0x0037e600 | inlined | in `mission_open_bin` at chunk `0x4050` |
| 0x00604ce0 | `mission_fade_in_object` | `AddTimeTransparentObject__9C_missionFP10I3D_objects` | 0x003862c0 | CONFIRMED | starts a timed fade-in (§5b) |
| 0x00604ef0 | `mission_finish_object_fade_ins` | `FinishTimeTransparentObjects__9C_missionFv` | 0x00386370 | CONFIRMED | forces them all to completion |
| 0x00604f70 | `mission_update_settings` | `UpdateSettings__9C_missionFv` | 0x00386410 | CONFIRMED | pushes the quality settings into the engine |
| 0x00605010 | `mission_restore_settings` | `RestoreSettings__9C_missionFv` | 0x00386520 | CONFIRMED | puts the engine back on unscaled settings; its one caller is `game_done` |
| 0x00605060 | `mission_settings_set_details_scale` | - | - | - | clamps to 0.4 .. 2.0 |
| 0x006050a0 | `mission_settings_set_draw_distance_scale` | - | - | - | clamps to 0.3 .. 1.0 |
| 0x006050e0 | `mission_settings_set_effects_quality` | - | - | - | 0 .. 2 |
| 0x00605110 | `mission_settings_set_sounds_quality` | - | - | - | 0 .. 2 |
| 0x00605140 | `mission_settings_set_shadow_quality` | - | - | - | stores the index of the value in `{0, 1, 3, 6, 9, 15}` |
| 0x006051a0 | `mission_settings_get_shadow_quality_idx` | - | - | - | reads it back |
| 0x006051f0 | `mission_settings_set_particles_quality` | - | - | - | 0 .. 2 |
| 0x00605220 | `mission_get_scene_bounds` | - | - | - | the world bounding box through `scene_accumulate_frame_bounds`; its one caller is `net_session_build_position_grid` |
| 0x00605340 | `mission_actor_cache_store` | `FrmActBuff_StoreActor__9C_missionFP7C_actorb` | 0x00386630 | CONFIRMED | parks an actor across a scene change (§5c) |
| 0x00605670 | `mission_model_cache_store` | `FrmActBuff_StoreModel__9C_missionFP9I3D_model` | 0x00386820 | CONFIRMED | parks a bare model |
| 0x006058e0 | `mission_actor_cache_take` | `FrmActBuff_GetActor__9C_missionFPci` | 0x00386920 | CONFIRMED | reclaims one by name and kind, rebirths it |
| 0x00605a30 | `mission_model_cache_take` | `FrmActBuff_GetModel__9C_missionFPc` | 0x00386ab0 | CONFIRMED | reclaims a bare model by name |
| 0x00605b30 | `mission_active_actor_store` | `FrmActBuff_StoreActiveActor__9C_missionFP7C_actor` | 0x00386bc0 | CONFIRMED | parks a live actor |
| 0x00605d40 | `mission_active_actor_drop` | `FrmActBuff_RemoveActiveActor__9C_missionFP7C_actor` | 0x00386cb0 | CONFIRMED | discards one without reviving it |
| 0x00605dd0 | `mission_active_actor_take` | `FrmActBuff_IsActiveActor__9C_missionFP7C_actor` | 0x00386d60 | CONFIRMED | revives and unparks; not a predicate on either build ² |
| 0x00605e90 | `mission_actor_sound_cache_store` | `FrmActBuff_StoreActorSound__9C_missionFP7C_actoriP9I3D_sound` | 0x00386e70 | CONFIRMED | parks an actor's sound |
| 0x00606100 | `mission_actor_sound_cache_take` | `FrmActBuff_GetActorSound__9C_missionFP7C_actori` | 0x00386f10 | CONFIRMED | reclaims it |
| 0x006061d0 | `mission_carry_caches_clear` | `FrmActBuff_ClearAll__9C_missionFv` | 0x00386fd0 | CONFIRMED | empties all three carry caches |

² Both bodies reset a car, call `Rebirth` and REMOVE the entry; the PC name states what the body
does.

### 3d. Free helpers

`Mission.cpp` is the host unit of these helpers (§6 holds the rest of the unit). The frame-map
family serves only `mission_open_bin`.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005fe550 | `frame_map_key_less` | - | - | - | the map's case-insensitive key comparator |
| 0x005ffeb0 | `scene_load_report_progress` | `ShowProgress__FP7C_chunk` | 0x0037d2d0 | CONFIRMED | posts the load progress from the archive cursor (`dtaSeek`), scaled by the load phase; called by `mission_open_bin` and `cache_base_block_open` |
| 0x00600540 | `scene_load_progress_callback` | `__SceneErrFnc__F21LS3D_CALLBACK_MESSAGEUiUiPv` | 0x0037d2a0 | CONFIRMED | the LS3D callback: forwards every message except `0x10` and `0x20` to `load_progress_callback` |
| 0x00600560 | `scene_fill_sector_bits` | `cbSectorBitFill__Q29C_mission21S_hlp$4890Mission_cppFP9I3D_frame` | 0x0037e190 | CONFIRMED | a file-static frame-enumeration callback; called by `mission_open` |
| 0x00604b20 | `scene_register_frame_by_name_cb` | - | - | - | the enumeration callback `mission_open_bin` passes by address: inserts each frame into the frame map |
| 0x00604c20 | `frame_map_destroy` | - | - | - | destroys the map |
| 0x00605250 | `scene_accumulate_frame_bounds` | - | - | - | grows a bounding box frame by frame; for `mission_get_scene_bounds` |
| 0x00605630 | `car_clear_all_seat_owners` | `ClearCarSeats__FP5C_car` | 0x003865a0 | CONFIRMED | called by `mission_actor_cache_store` and `mission_active_actor_store` (`structs/car.md`) |
| 0x006063a0 | `vec_dword_push_back_by_ref` | - | - | - | a generic dword `push_back`; 11 callsites, one in `mission_open_bin` |
| 0x00606550 | `vec_count_stride_0x10` | - | - | - | a generic element count, 42 callsites across the binary |
| 0x00606570 | `vec_count_stride_0xc` | - | - | - | the same for stride `0xc`, 31 callsites |
| 0x006065d0 | `vec_index_addr_stride_0x8` | - | - | - | a generic 8-byte `operator[]`, 377 callsites, 13 in `mission_open_bin` (§6b) |
| 0x006065e0 | `frame_map_at_or_insert` | - | - | - | `operator[]` |
| 0x00607500 | `folded_vector_pair_free_at_0x04_0x14_00607500` | - | - | - | frees two vectors; `game_dtor` and four unwind funclets (`structs/game.md`) |
| 0x00607530 | `frame_map_insert` | - | - | - | - |
| 0x00607640 | `frame_map_begin` | - | - | - | - |
| 0x00607650 | `frame_map_pair_construct` | - | - | - | - |
| 0x00607820 | `frame_map_insert_00607820` | - | - | - | the insert with rebalance |
| 0x00607a80 | `frame_map_erase_node` | - | - | - | - |
| 0x00607e70 | `frame_map_destroy_subtree` | - | - | - | - |
| 0x00607ec0 | `frame_map_init` | - | - | - | - |
| 0x00607f40 | `stl_fill_n_stride_0x1c` | - | - | - | a generic `uninitialized_fill_n`, 16 callsites across the binary |
| 0x00607f70 | `frame_map_insert_node` | - | - | - | - |
| 0x006081e0 | `frame_map_rotate_left` | - | - | - | - |
| 0x00608240 | `frame_map_rotate_right` | - | - | - | - |
| 0x006082a0 | `frame_map_alloc_node` | - | - | - | - |
| 0x006082c0 | `frame_map_iter_decrement` | - | - | - | - |
| 0x00608320 | `frame_map_iter_increment` | - | - | - | - |
| 0x006083b0 | `crt_register_atexit_stub` | - | - | - | the closing guard jumps to it |

## 4. The object, and the pointer that names it

| Symbol | Address | What it is |
|---|---|---|
| `g_mission` | `0x006f9440` | **the static object**, a `Main.cpp` file-scope variable |
| `g_act_mission` | `0x0065115c` | **a pointer to it**, statically initialised |

`.data:0x0065115c` holds the literal `0x006f9440`, byte-read from `Game.exe`. That is what ties the
initializer stub in `Main.cpp` to the 1500-odd `g_act_mission` dereferences elsewhere in the
decomp. `g_mission` has **zero textual existence** in the decomp: every access goes through the
pointer or through ECX, so a search for `0x006f9440` returns nothing, and its name can only ever be
a registry row, exactly as for `g_input`.

## 5. The lifecycle lists

### 5a. The scene's actors and models

`mission_add_anim_model` / `mission_del_anim_model` / `mission_clear_game_anim_models` own
`anim_models`; the `per_game` byte marks a model registered for the lifetime of the GAME, and
`mission_clear_game_anim_models` purges exactly those with a swap-with-last pass.

`mission_add_actor(actor, transient)` appends to `actors` always and to `transient_actors` when the
flag is set. `mission_del_not_me_actors`, whose sole caller is `game_done`, destroys everything in
`transient_actors`, so **`transient_actors` is the set that does NOT survive a scene change**, and
`mission_is_persistent_actor` is the predicate for the complement. Callers that pass 1:
`loadsave_savegame_load`, `racing_spawn_winners_scene`. Callers that pass 0:
`engine_new_mission` (the carried-over player and car) and `loadsave_freeride_setup_scene`.

### 5b. The pending fade-ins

`mission_fade_in_object(obj, duration_ms)` takes a reference, sets the object's alpha to **0** and
appends `{obj, elapsed = 0, duration}`. `mission_tick` ramps the alpha to `elapsed / duration`
each frame and, on reaching the duration, forces 1.0, releases and swap-pops.
`mission_finish_object_fade_ins` forces all of them at once.

**Its sole producer is the city streamer.** `cache_base_block_walk_block_objects` calls it with the
literal **700** for every frame of a newly activated block whose visual type is 1 and whose tag is
`'OBJ_'`, so this is the pop-in hider: streamed-in geometry fades up over 700 ms instead of
appearing at once.

### 5c. The three scene-change carry caches

Between `mission_close` and the next `mission_open`, actors and models that must survive are parked
here. `mission_carry_caches_clear` empties all three, and `mission_close` calls it.

| Vector | Record | Store | Take |
|---|---|---|---|
| `carried_frames` | `S_frame_actor_buffer` (§1d) | `mission_actor_cache_store` (callers `traffic_car_game_done`, `garage_game_done`, `police_manager_game_done`), `mission_model_cache_store` | `mission_actor_cache_take` (callers `traffic_car_game_init`, `garage_game_init`, `police_manager_apply_state`), `mission_model_cache_take` (callers `pedestrian_create_model`, `railway_game_init`) |
| `carried_active_actors` | `C_actor *` | `mission_active_actor_store` | `mission_active_actor_take`, `mission_active_actor_drop` |
| `carried_actor_sounds` | `S_actor_sound_buffer` (§1e) | `mission_actor_sound_cache_store` | `mission_actor_sound_cache_take` |

A `carried_frames` record holds EITHER an actor (`actor` set) or a bare model (`actor` null), which
is what the two `take` functions discriminate on. Storing an actor refuses a null frame, a frame
that is not a MODEL (type 9), the two carried new-mission actors and a duplicate; it deactivates a
car (and clears its seats) or a human, resets the placement and keys the record by the model's
basename.

### 5d. The mission number and the frame-skip countdown

`next_mission_number` and `mission_number` are the **pending** and **current** mission number,
and the pair is superglobal: `mission_set_next_mission_number` fills `+0x70`,
`mission_commit_next_mission_number` moves it into `+0x74` at `change_mission`, and
`mission_get_mission_number` reads `+0x74`. The savegame chunk hook `game_savegame_load_before`
reads and writes the same pair.

`frame_skip_count` is a **frame countdown**. While it is above 0, `engine_game_loop` (and
`race_game_loop` / `race_game_loop_net`) decrements it once per frame and skips that frame's
present call (`g_engine` slot `+0x3c`). Its writers:

| Writer | Value | How |
|---|---|---|
| `mission_open` | 0 | stores it directly |
| `game_init` | 10 | through `mission_set_frame_skip_count` |
| `recorder_stop` | 4 | only when it is lower |
| `recorder_end_sequence_and_restore_scene` | 2 | only when it is lower |
| `race_winners_replay_begin` | 1 | only when it is lower |
| `race_winners_replay_end` | 2 | only when it is lower |
| script command `recloadfull` | 1 | only when it is lower, through the accessor pair |
| script command `recunload` | 2 | only when it is lower, through the accessor pair |
| `engine_game_loop`, `race_game_loop`, `race_game_loop_net` | minus 1 per frame | while it is above 0 |
| `human_do_change_weapon`, `human_weapon_toggle_held` | minus 1 | while it is above 0 a weapon change consumes one count and skips its change animation |

`restore_sound_volume` is the sound-restore latch. `mission_open` sets the sound manager's volume
to 0 and sets the latch; `mission_tick` restores the saved volume and clears the latch on the first
frame the countdown is below 1. So a scene load presents no frame for ten frames, and the sound
returns when the countdown runs out.

## 6. The scene file and the rest of the unit

### 6a. The `scene2.bin` chunk vocabulary

`mission_open_bin` reads `missions\<mission>\scene2.bin` through a stack `C_chunk`
(`structs/chunk.md` §4). The *shipped* column counts occurrences in the 79 shipped files; an id the
reader handles that no shipped file contains shows 0. Frame types are named by the type-name
function of `LS3DF.dll` (`structs/i3d_frame.md` §5b).

**The root chunk `0x4c53`:**

| Id | Shipped | Payload | What `mission_open_bin` does |
|---|---:|---|---|
| `0x0001` | 79 | `u16` major, `u16` minor, a text banner | the file version; every shipped file is 5.10. Later arms gate on it: `0x40a0` needs a major above 2, `0xb110` a major of at least 5, and a light's flag word gains `0x40` when `major × 100 + minor` is below 510 |
| `0x3010` | 79 | float, radians | the active camera's vtable `+0x50`; 70° (1.2217) in 50 scenes |
| `0x3011` | 79 | float | the view distance: the active camera's vtable `+0x58(0.05, value)` and `settings.view_distance` (§1a); 300 in 38 scenes |
| `0x3012` | 0 | - | the active camera's vtable `+0x64(1)` |
| `0x3013` | 0 | dword | the active camera's vtable `+0x60` |
| `0x3200` | 79 | 3 floats, an RGB colour | read and discarded: the arm then stores `(0, 0, 0)` at `I3D_scene+0x220`; 7 scenes carry a non-black colour |
| `0x3211` | 79 | 2 floats | `I3D_scene` vtable `+0xb8(a, b)`; `(1, 1000)` in 49 scenes |
| `0x4000` | 79 | the frame block | one `0x4010` record per frame (below) |
| `0xAE02` | 3 | `0xAE03` records | UV-scroll channels: each `{u16 channel, float, float, u32 length, frame name}` looks the frame up and, when it is a VISUAL tagged `'LMAP'` or `'OBJ_'`, adds a channel through `uv_scroll_channel_init` and `uv_scroll_add_channel` (`structs/uv_scroll.md`) |
| `0xAE20` | 78 | `0xAE21` records | the object definitions (below) |
| `0xAE40` | 0 | - | `web_path_load_nodes`; the shipped graphs are in `check.bin` (`structs/web_path.md` §4) |
| `0xAE41` | 0 | - | `roads_load_data`; the shipped data is in `road.bin` (`structs/roads.md`) |
| `0xAE50` | 73 | `0xAE51` records | the global scripts: each builds a `C_program` (`program_ctor`, then `program_load_source_code` over the serialised form of `structs/chunk.md` §4c) and pushes it onto `global_programs` |
| `0xAEF0` | 4 | `(u32 id, u32 value)` pairs, ids 0..13 | weather parameters, `I3D_scene` vtable `+0xe8(id, value)`; only the four MISE09 scenes |
| `0xAEF1` | 4 | `u32` count, then `(u32 length, name)` entries | each DUMMY frame named is handed to `I3D_scene` vtable `+0xf4`; the shipped frames are named `sem_neprsi…` (roughly "no rain here") |
| `0xAFFF` | 79 | 8 bytes | copied into `scene_identity` (§1) |

**The frame record `0x4010`** (133 801 shipped):

| Id | Shipped | Payload | What `mission_open_bin` does |
|---|---:|---|---|
| `0x0010` | 133 801 | name | finds the frame by name; when it does not exist, creates it through the I3D driver as the type in `0x4011` (a VISUAL through the driver's `+0x48` with the FourCC of `0x4012`, any other type through `+0x4c`) and names it |
| `0x4011` | 97 730 | dword | the frame type |
| `0x4012` | 0 | FourCC | the class a VISUAL is created with |
| `0x0020` | 99 643 | vec3 | the local position |
| `0x0022` | 98 700 | 4 floats | the rotation quaternion |
| `0x002c` | 98 805 | vec3 | the position used instead of `0x0020` when the parent named in `0x4020` is not found |
| `0x002d` | 99 168 | vec3 | the scale, default `(1, 1, 1)` |
| `0x4020` | 93 193 | a `0x0010` name | the parent frame |
| `0x4033` | 713 | empty | marks the frame to be switched off (record flag `0x10000`; `structs/cache_base_block.md` §5a); ignored on a SOUND frame |
| `0x2012` | 72 585 | name | the model file of a MODEL frame, opened through `i3d_cache_model_open` with `.i3d` appended |
| `0x4040` | 9 095 | the light block | on a LIGHT frame (below) |
| `0x4050` | 2 378 | `0x18` bytes | on a DUMMY frame, stored by `i3d_dummy_set_bound_box`: the frame's bounding box, minimum then maximum corner (`structs/i3d_frame.md` §1c) |
| `0x4060` | 5 254 | the sound block | on a SOUND frame (below) |
| `0x4083` | 7 188 | `u32` n, n × 12-byte vertices, `u32` m, m × 6-byte triangles | on an OCCLUDER frame, passed to the frame's vtable `+0x50` |
| `0x4090` | 31 671 | FourCC | on a VISUAL whose tag differs, a visual of that class is created through the driver's `+0x48` and takes over the frame's name, children and place; the shipped values are `'LMAP'` 31 175, `'LENS'` 495 and `'MRPH'` 1 |
| `0x40a0` | 31 175 | lightmap data | on an `'LMAP'` visual, streamed through the frame's vtable `+0x94` with the file's dta handle |
| `0xB100` | 0 | - | skipped |
| `0xB110` | 495 | sub-chunks `0xB112..0xB117` | on a `'LENS'` visual, six values into its `+0x220`, `+0x21c`, `+0x228` (converted from a float), `+0x254`, `+0x258` and `+0x274` |
| `0xB151` | 493 | as `0x4083` | on an AREA frame, passed to the frame's vtable `+0x50` |
| `0xB400` | 0 | `(id, dword)` sub-chunks | on a SECTOR frame, each passed to the sector's vtable `+0x60(id, value)` |
| `0xB401` | 1 073 | sub-chunks `0x00..0x15` | on a SECTOR frame, each passed to the sector's vtable `+0x60(id, value)`; ids `0x14` and `0x15` carry a vec3, the others a dword |

**The light block `0x4040`** (on a LIGHT frame; the arm first detaches the light from every sector
it is in):

| Id | Shipped | Payload | Consumer |
|---|---:|---|---|
| `0x0026` | 9 095 | vec3 | the colour, the light's vtable `+0x58` |
| `0x4041` | 9 095 | dword | the light's vtable `+0x50` |
| `0x4042` | 9 095 | float | the light's vtable `+0x5c` |
| `0x4043` | 9 095 | 2 floats | the light's vtable `+0x68` |
| `0x4044` | 9 095 | 2 floats | a range pair, the light's vtable `+0x64`; a near value below 1e-8 becomes 0.1, and a far value below the near one becomes near + 0.1 |
| `0x4045` | 9 095 | dword | the flag word, the light's vtable `+0x60` |
| `0x4046` | 12 554 | name | a SECTOR the light is added to (sector vtable `+0x50`); deferred to the end of the file when the sector is not loaded yet |

**The sound block `0x4060`** (on a SOUND frame; the arm first switches the frame off and calls the
sound's vtable `+0x6c(0)`):

| Id | Shipped | Payload | Consumer |
|---|---:|---|---|
| `0x0010` | 5 254 | name | the sound file: taken from the sound cache when a cached name matches it, otherwise opened through the sound's vtable `+0x50` and cached |
| `0x4061` | 5 254 | dword | the sound's vtable `+0x58` |
| `0x4062` | 5 254 | dword | the sound's vtable `+0x68` |
| `0x4063` | 5 254 | dword | the sound's vtable `+0x64` |
| `0x4064` | 5 254 | 2 floats | the sound's vtable `+0x60(a, b)` |
| `0x4065` | 0 | vec3 | the sound's vtable `+0x5c(x, y, z, 1.0)` |
| `0x4066` | 3 416 | empty | the sound's vtable `+0x6c(1)` |
| `0x4067` | 5 206 | empty | skipped |
| `0x4068` | 5 254 | 4 dwords | the sound's vtable `+0x5c(a, b, c, d)`, the four-value form of `0x4065` |
| `0xB200` | 8 091 | name | a SECTOR the sound is added to (sector vtable `+0x58`); deferred like a light's |
| `0xB201` | 212 | name | an AREA the sound is linked to (area vtable `+0x54`); deferred likewise |
| `0xB800` | 5 254 | dword | the sound's vtable `+0x74` |

**The object definition `0xAE21`** (13 631 shipped, 78 scenes):

| Id | Shipped | Payload | What `mission_open_bin` does |
|---|---:|---|---|
| `0xAE23` | 13 631 | name | the frame the object binds to, looked up by name |
| `0xAE22` | 13 631 | dword | the entity kind: `mission_create_actor(kind)`, the actor's vtable `Init` with the frame, then `actor_mark_scene_placed` and `mission_add_actor` |
| `0xAE24` | 13 631 | bytes | the kind's own data, handed to the actor's `LoadData` slot |

### 6b. The deferred frame fix-ups

`mission_open_bin` drives **13 callsites** of `vec_index_addr_stride_0x8` (`0x006065d0`) against
two stack-local vectors of `{I3D_frame *, int}` deferred frame-fixup pairs. That address is a
generic `std::vector<8-byte>::operator[]` with 377 callsites image-wide; the script VM indexes
`{opcode, args}` through the same address. Whether one declared pair type or identical-COMDAT
folding of two types is behind the sharing cannot be decided from either binary
(`structs/program.md` §9).

### 6c. The rest of the unit

`Mission.cpp` also hosts other classes' code, documented with those classes: `game_ctor`,
`game_dtor` (`structs/game.md`), the `uv_scroll_*` (`structs/uv_scroll.md`), `roads_*`
(`structs/roads.md`), `chunk_*` (`structs/chunk.md`), `i3d_cache_items_*` (`structs/i3d_cache.md`)
and `string_*` (`structs/string.md`) helpers, and `i3d_dummy_set_bound_box`
(`structs/i3d_frame.md`). Full list: `Game-ghidra.exe.functions.md`, the `Mission.cpp` section and
its three siblings.

**The end of the unit.** After `0x00607f70` come five more `frame_map_*` primitives,
`string_ref_copy_construct` (`0x00608370`) and the closing CRT pair: `crt_static_init_guard`
(`0x00608390`), the unit's copy of the end-of-unit guard, and the `crt_register_atexit_stub` it
jumps to (`0x006083b0`). The MSVC initializer table lists `0x005fe4f0`, `0x005fe520` and
`0x00608390` in a row, the unit's opening and closing entries. The `C_web_node` / `C_web_path`
code (`structs/web_path.md`) follows from `0x006083c0` (`web_node_dtor`) and ends with its own
guard copy at `0x0060b740`.

## 7. The new-mission transition

A mission change requested by the script command `newmission` (`program_process_command` case
`0x33`) is not performed on the spot. The case fills a block of file-scope globals, raises
`g_reload_pending`, and the next iteration of `engine_game_loop` clears the flag and calls
**`engine_new_mission`** (`0x005f8700`, a `Main.cpp` free function).

### 7a. The request block

| PC address | PC name | What it holds |
|---|---|---|
| `0x006f9210` | `g_new_mission_rot_dir` (`S_vector`) | the arrival FORWARD vector |
| `0x006f921c` | `g_new_mission_rot_up` (`S_vector`) | the arrival UP vector |
| `0x006f9228` | `g_new_mission_pos` (`S_vector`) | the arrival position |
| `0x006f9234` | `g_new_mission_entry` | the ENTRY FRAME name in the new scene |
| `0x006f9254` | `g_reload_pending` | the one-shot request flag |
| `0x006f9258` | `g_new_mission_name` | the MISSION name |
| `0x006f9570` | `g_new_mission_actors` (`pointer[2]`) | the carried actors: the player and his car |
| `0x006f9578` | `g_new_mission_speed_kmh` | arrival speed, **km/h** |

The PC's `S_matrix::SetDir3(dir, up)` call splits the rotation block into its two vectors, and the
script arm's own loop bound `< 0x6f9578` bounds the 2-slot actor array (§8c). The speed's unit is
proven by `car_reset(value * 0.2777778, 1)`, i.e. 1/3.6.

### 7b. What `engine_new_mission` does

Two paths, chosen by whether `g_new_mission_entry` is empty:

* **empty** — a plain reload: `game_done`, `mission_close`, `mission_open(g_new_mission_name)`,
  `collision_load_level` on `missions\%s\tree.klz`, `game_init`.
* **set** — the same, but the player and (if he was driving) his car are **carried over**. Both
  are deactivated, the inventory block `C_human+0x480 .. +0x560` is saved to the stack and cleared,
  the old scene is torn down, every kind-2 actor in the NEW scene is deleted, the two carried
  actors are pushed back into the new `C_game`'s actor list at `+0x148`, the inventory is restored,
  and the entry frame is resolved so the actor can be placed and the car reset to the requested
  speed.

## 8. PC vs PS2

PS2 class `C_mission`; the static object is at `0x00509b10`, constructed by `__sinit_Main.cpp` in
the same lockstep slot as the PC's. `engine_new_mission` is the PS2's `InternMissionInit__Fv`
(`Game-ghidra.exe.functions.md`). `act_mission_get_game` has no PS2 method: the PS2 reads
`act_mission+0x20` inline wherever the PC calls the accessor, and the seven `0x005c86xx` and the
settings accessors are likewise PS2 inline code (the PS2 writes `C_mission+0xcbc..+0xcd4`
directly).

### 8a. What the ctor / dtor diff established

**Constructor.** Both zero the leading dword, construct the member vectors, and write the six
settings defaults. The PC writes `1.0f, 1.0f, 2, 2, 3, 2` at `+0x90/+0x94/+0x9c/+0xa0/+0xa4/+0xa8`;
the PS2 writes the same six values at `+0xcbc/+0xcc0/+0xcc8/+0xccc/+0xcd0/+0xcd4`. That reproduces
the `+0x9c ↔ +0xcc8` and `+0xa8 ↔ +0xcd4` pairs the weather / hydrant detail evidence produces from
a completely different argument, an independent confirmation of both.

**Destructor.** Both call an un-init helper first and then free every member vector in
**descending offset order**: PC `0xd0, 0xc0, 0xb0, 0x84, 0x58, 0x48, 0x38, …`; PS2 `0xd50, 0xd38,
0xd08, 0xcfc, 0xcf0, 0xce4, 0xcd8, 0xcb0, … 0x4c`.

### 8b. The offset delta is the PS2's embedded particle manager

The two field maps differ by a constant `0xc2c` in the tail, and that number has a cause: **PS2
`C_mission` embeds a `C_ParticleManager` at `+0x64`**, spanning to about `+0xcb0`
(`__ct__17C_ParticleManagerFv((int)param_1 + 100)` in the constructor,
`__dt__17C_ParticleManagerFv(iVar2 + 100, …)` in the destructor). The PC has no such member: its
particle manager is the pointer at `+0x6c` (`structs/particle.md`). Below `+0x64` the two maps
agree up to the PS2's `C_game` at `+0x20` (PC `+0x24`); above it they do not. Do **not** rebase a
PS2 `C_mission` offset onto the PC by a constant.

### 8c. The new-mission request block

The PS2 symbol table names every global of the request block (§7a), with sizes, and the PC layout
matches field for field:

| PC name | PS2 symbol |
|---|---|
| `g_new_mission_rot_dir` | `NewMission_ROT` |
| `g_new_mission_rot_up` | `NewMission_ROT` + `0x10` |
| `g_new_mission_pos` | `NewMission_POS` |
| `g_new_mission_entry` | `NewMission_ENTRY` |
| `g_reload_pending` | `ActiveNewMission` |
| `g_new_mission_name` | `NewMission` |
| `g_new_mission_actors` | `NewMissionActors` |
| `g_new_mission_speed_kmh` | `NewMissionSpeed` |

The PS2 gives `NewMission_ROT` a size of 32 and `NewMissionActors` a size of 8; the PC's
`SetDir3(dir, up)` call and the script arm's loop bound prove the same splits independently.

### 8d. Three read helpers inlined on the PC

`ReadVector3__FR7C_chunk`, `ReadQuatOld__FR7C_chunk` and `ReadBBoxOld__FR7C_chunkR8I3D_bbox` have
`OpenBin` as their only caller, and each reads one value and closes the chunk. At the same
frame-record chunk ids `mission_open_bin` does exactly that inline: `0x0020`, `0x0026`, `0x002c`
and `0x002d` read 12 bytes, `0x0022` reads 16 and `0x4050` reads `0x18`, each followed by
`chunk_leave`. The PS2's one further `ReadVector3` site reads sub-chunks `0x001d` / `0x001e` of
chunk `0xB100` on an `'EMTR'` visual; the PC skips `0xB100` and no shipped scene contains it
(§6a). The PS2 helpers exist for its own layout: its vectors are 16 bytes (`ReadVector3` widens 12
to 16), it keeps a quaternion's `w` last (`ReadQuatOld` turns the file's `(w, x, y, z)` into
`(x, y, z, w)`, while the PC copies the 16 bytes as read), and its bounding box holds two 16-byte
corners (`ReadBBoxOld` reads twice, the PC once).

### 8e. The frames-ref buffer is a CONTAINER difference

The PS2 keeps a `C_mission` member `vector<I3D_frame *>` and clears it with
`ClearFramesRefBuff__9C_missionFv`. The PC keeps **no such member**: `mission_open_bin` builds a
**stack-local `std::map<C_str, I3D_frame *>`**, indexes every scene frame by name into it while
reading the file, and destroys it before returning. That map is the entire `frame_map_*` family
(§3d), 18 functions, whose single-owner census is what earns them a family name rather than a
generic red-black-tree one (`Game-ghidra.exe.functions.md`, the frame-name-map section). A banner
marks the absent rank inside `mission_close`.

### 8f. The new-mission transition needs two PS2 functions the PC does not

At the rank between the teardown and the reopen (§7b), PS2 `InternMissionInit__Fv` calls
`InternMission_Vykur__Fv`; `C_game::Init` later calls `InternMission_Spolkni__Fv`. Both are
**PS2-port-only**, measured over a closed set:

* their private API `C_car::DamageDeformGetSize` / `DamageDeformSave` / `DamageDeformLoad` has
  **exactly three callsites in the whole PS2 image, and all three are this pair**;
* the PC has no `C_car`-level deform trio at all, and the complete caller set of the `C_Vehicle`
  ones is `car_save_game_get_size` / `_save` / `_load` plus `vehicle_settle`, none in the
  mission-transition path;
* `Vykur` calls `SchovejPero__7C_human` and `Spolkni` calls `VystrcPero__7C_human`, the
  PS2-port-only model teardown / rebuild pair.

**The cause is the model cache.** `Spolkni` asks the driver for a fresh `I3D_model`, evicts the
stale cache entry and re-opens by the saved name before restoring the deformation, because the PS2
flushes its model cache between city parts. The PC keeps the object alive and re-parents its
frame, so there is nothing to serialise. A banner marking the absent rank sits in
`engine_new_mission` in the decomp.

`InternMissionInit_InterCity__FR11G_Inventory` is PS2-only for the ordinary reason: one caller,
gated on the InterCity flag, every distinctive callee `InterCity_*`. Two PS2 **free** functions of
the band are PS2-only on the same argument: `ReleaseUniqueTextures__Fv` (its state is
`inter_city_texture`) and `DelPicaModelDel__Fv`. `MC_DeleteData` / `MC_LoadData` / `MC_Format`,
immediately before the constructor, are the memory-card layer.

### 8g. The 33 PS2 methods with no PC counterpart

`InterCity` appears 127 times on the PS2 and **0** times on the PC: the city-part streaming
subsystem is PS2-port-only.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00387240 | `ClearFramesRefBuff__9C_missionFv` | the PC keeps the frames as a stack-local map in `mission_open_bin` (§8e) |
| 0x00388aa0 | `InterCity_Prebuffering__9C_missionFPc` | the InterCity subsystem |
| 0x00389130 | `InterCity_Init__9C_missionFPc` | the InterCity subsystem |
| 0x00389dd0 | `InterCity_PicaDelActors__9C_missionFv` | the InterCity subsystem |
| 0x00389e50 | `InterCity_Done__9C_missionFv` | the InterCity subsystem |
| 0x0038a070 | `InterCity_GameInit__9C_missionFv` | the InterCity subsystem |
| 0x0038a420 | `InterCity_GameDone__9C_missionFv` | the InterCity subsystem |
| 0x0038a4c0 | `InterCity_GameDoneInGameDone__9C_missionFv` | the InterCity subsystem |
| 0x0038a8e0 | `InterCity_ActorsToMission__9C_missionFb` | the InterCity subsystem |
| 0x0038b5f0 | `InterCity_ActorsFromMission__9C_missionFv` | the InterCity subsystem |
| 0x0038b660 | `InterCity_ChangeMission__9C_missionFv` | the InterCity subsystem |
| 0x0038bd50 | `InterCity_SetMissionName__9C_missionFPc` | the InterCity subsystem |
| 0x0038bd60 | `InterCity_RepairDirName__9C_missionFPc` | the InterCity subsystem |
| 0x0038bde0 | `InterCity_AddGlobalActor__9C_missionFP7C_actor` | the InterCity subsystem |
| 0x0038be70 | `InterCity_DelGlobalActor__9C_missionFP7C_actor` | the InterCity subsystem |
| 0x0038bed0 | `IsInterCityGlobalActor__9C_missionFP7C_actor` | the InterCity subsystem; its PS2 call in `FrmActBuff_StoreActor` has no PC counterpart, and a banner marks the rank in `mission_actor_cache_store` |
| 0x0038bf20 | `LoadFQLPath__9C_missionFv` | the FQL path / index family: `mission_open` calls nothing at the rank where the PS2 `Open` calls `OpenFQLFile` |
| 0x0038bfc0 | `UnloadFQLPath__9C_missionFv` | the FQL family |
| 0x0038bff0 | `GetFQLPath__9C_missionFPc` | the FQL family |
| 0x0038c0c0 | `LoadAnimationPath__9C_missionFv` | the same family, animation half |
| 0x0038c160 | `UnloadAnimationPath__9C_missionFv` | the same family, animation half |
| 0x0038c190 | `GetAnimationPath__9C_missionFPc` | the same family, animation half |
| 0x0038c260 | `InterCity_SaveLoadInit__9C_missionFv` | the InterCity subsystem |
| 0x0038c2d0 | `InterCity_SaveLoadDone__9C_missionFv` | the InterCity subsystem |
| 0x0038c3b0 | `InterCity_RepairActorActState__9C_missionFv` | the InterCity subsystem |
| 0x0038c550 | `InterCity_AddTempAct__9C_missionFP7C_actor` | the InterCity subsystem |
| 0x0038c5e0 | `InterCity_DelTempAct__9C_missionFP7C_actor` | the InterCity subsystem |
| 0x0038c640 | `InterCity_TickTempAct__9C_missionFv` | the InterCity subsystem |
| 0x0038c890 | `Garage_AddCar__9C_missionFP5C_car` | the PC has a `garage_*` family in its own unit; not searched against it (Open item 1) |
| 0x0038c9b0 | `Garage_DelCar__9C_missionFP5C_car` | as `Garage_AddCar` |
| 0x0038cde0 | `Freeride_ChangeCar__9C_missionFP5C_carUi` | as `Garage_AddCar` |
| 0x0038d710 | `OpenFQLFile__9C_missionFPc` | the FQL family |
| 0x0038d790 | `CloseFQLFile__9C_missionFv` | the FQL family |

The grouping is an inventory by family; no single method of it was body-read to a verdict.

## 9. Related

- `structs/app_bootstrap.md` — `Main.cpp`, the static object's host, and the settings mirror.
- `structs/game.md` — the `C_game` at `+0x24`.
- `structs/i3d_frame.md` — the `I3D_scene` at `+0x10` and the frame types the scene file uses.
- `structs/actor.md` — the net-GUID stamp and `folded_get_dword_0x10`.
- `structs/i3d_cache.md` — the item-vector helpers `mission_close` and `mission_uninit` call.
- `structs/chunk.md` — the scene file's chunk reader.
- `structs/uv_scroll.md`, `structs/web_path.md`, `structs/roads.md`, `structs/particle.md` — the
  sub-objects.
- `structs/inventory.md` — the inventory block `engine_new_mission` carries across.

## 10. Open items

1. **`Garage_AddCar` / `Garage_DelCar` / `Freeride_ChangeCar` (§8g)** have not been searched
   against the PC `garage_*` family.
2. **In game: the fade-in.** Streamed-in city geometry should fade up over about 0.7 s rather than
   popping (`mission_fade_in_object`, §5b).
3. **In game: scene-placed cars.** A car placed by the scene `.bin` should survive a mission change
   while one spawned by script should not: the `C_actor+0x66` `scene_placed` flag (§5c and
   `structs/actor.md`).
