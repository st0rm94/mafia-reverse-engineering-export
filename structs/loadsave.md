# `G_LoadSaveClass` and the `g_loadsave` translation unit

**Created** 2026-08-23 16:22 +0200 · **Modified** 2026-11-17 10:40 +0100

The profile, savegame, replay, objective, Free Ride, settings and credits singleton, with the encrypted stream class and the car
permission record of the same unit. `G_CarDatabase`, the unit's second singleton, has its own document, `structs/car_database.md`, and the
`G_GarageClass` band of this object is typed in `structs/garage.md` §1.

| | |
|---|---|
| PC prefix | `loadsave_` for the singleton, `stream_` for `G_Stream`, `enabled_cars_` for `G_EnabledCars`, `credits_` for the two credits free functions |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `G_LoadSaveClass` `0x5d8`, the singleton at `0x00671fb0`; `G_Stream` `0x3c`; `G_EnabledCars` `0x40` |
| PC constructor | `loadsave_ctor`, `stream_ctor` |
| PC translation unit | `[0x00556ee0, 0x00561b40)`, guard to guard; this doc hosts the unit (§4) |
| PS2 class | `G_LoadSaveClass` (tag `15G_LoadSaveClass`), `G_Stream` (tag `8G_Stream`), `G_EnabledCars` (tag `13G_EnabledCars`) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `G_LoadSaveClass`

The singleton sits at `0x00671fb0`, so every field also has a constant address, and the settings fields are read under their global names.

```c
typedef struct G_LoadSaveClass {                  // 0x5d8 bytes, singleton at 0x00671fb0
    /* +0x000 */ uint8_t  profiles_alloc;         // [H] MSVC's empty allocator member of the profile vector
    /* +0x004 */ S_profile_head *profiles_begin;  // [H] the enumerated profiles, stride 0x54
    /* +0x008 */ S_profile_head *profiles_end;    // [H] count = (end - begin) / 0x54
    /* +0x00c */ S_profile_head *profiles_cap;    // [H] one past the allocation
    /* +0x010 */ uint8_t  slots_alloc;            // [H] the empty allocator of the slot vector
    /* +0x014 */ S_savegame_head *slots_begin;    // [H] the enumerated savegame slots, stride 0x18
    /* +0x018 */ S_savegame_head *slots_end;      // [H] count = (end - begin) / 0x18
    /* +0x01c */ S_savegame_head *slots_cap;      // [H] one past the allocation
    /* +0x020 */ S_profile_head active_profile;   // [H] the selected profile; streamed first in the profile file
    /* +0x074 */ S_savegame_head active_slot;     // [H] the selected savegame slot; opens the slot file
    /* +0x08c */ int32_t  replay_mode;            // [H] -2 record, 1 play, 0 off
    /* +0x090 */ uint8_t  unread_90;              // [U] set to 1 by loadsave_init and loadsave_replay_set_mode; never read
    /* +0x091 */ uint8_t  profile_load_failed;    // [H] set by loadsave_ctor and at the head of a load, cleared by a good load
    /* +0x094 */ uint32_t race_unlock_mask;       // [H] the cars unlocked for racing
    /* +0x098 */ uint32_t steal_mask;             // [H] the garage's steal permission mask; typed in structs/garage.md section 1
    /* +0x09c */ uint32_t freeride_cars;          // [H] the Free Ride enabled-car mask: bit n = car group 'A' + n
    /* +0x0a0 */ uint8_t  garage_band_b[0x12e];   // [H] the 25-slot garage table, its snapshot and the two request bytes; structs/garage.md section 1
    /* +0x1d0 */ uint32_t mission_name_id;        // [H] the textdb id stored in a new slot's header
    /* +0x1d4 */ uint8_t  suppress_next_save;     // [H] one-shot (section 10h)
    /* +0x1d8 */ uint8_t  objectives_alloc;       // [H] the empty allocator of the objectives vector
    /* +0x1dc */ uint32_t *objectives_begin;      // [H] dword string ids, stride 4
    /* +0x1e0 */ uint32_t *objectives_end;        // [H] count = (end - begin) >> 2
    /* +0x1e4 */ uint32_t *objectives_cap;        // [H] one past the allocation
    /* +0x1e8 */ uint8_t  garage_band_c[8];       // [H] last_entered_car and blocked_car; structs/garage.md section 1
    /* +0x1f0 */ uint32_t freeride_mode;          // [H] the Free Ride mode word; above 2 the script parameters apply (section 7)
    /* +0x1f4 */ S_key_binding bindings[60];      // [H] the key-binding table, 0x2d0 bytes, section 9
    /* +0x4c4 */ float    steering_linearity;     // [H] g_steering_linearity, the steering linearity slider
    /* +0x4c8 */ float    blood_level;            // [H] g_blood_level, the "Level of Blood" slider
    /* +0x4cc */ float    look_filter_scale;      // [H] the mouse-look filter's time scale; 0 on the PC (§2a)
    /* +0x4d0 */ float    look_filter_base;       // [H] the mouse-look filter's decay base; 0 on the PC (§2a)
    /* +0x4d4 */ uint32_t unread_4d4;             // [U] zeroed and streamed with the profile; never read
    /* +0x4d8 */ float    aiming_sensitivity_x;   // [H] g_aiming_sensitivity_x, slider
    /* +0x4dc */ float    aiming_sensitivity_y;   // [H] g_aiming_sensitivity_y, slider
    /* +0x4e0 */ float    max_aiming_speed_x;     // [H] g_max_aiming_speed_x, the "Max Aiming Speed X" slider
    /* +0x4e4 */ float    sounds_volume;          // [H] g_saved_sounds_volume
    /* +0x4e8 */ float    music_volume;           // [H] g_music_volume
    /* +0x4ec */ float    cars_volume;            // [H] g_cars_volume
    /* +0x4f0 */ float    speech_volume;          // [H] g_speech_volume
    /* +0x4f4 */ uint32_t camera_onfoot_mode;     // [H] g_camera_last_onfoot_mode; loadsave_apply_settings resets it to 1 when out of range
    /* +0x4f8 */ uint32_t camera_vehicle_mode;    // [H] g_camera_last_vehicle_mode; loadsave_apply_settings resets it to 7 when out of range
    /* +0x4fc */ float    gamma;                  // [H] g_gamma, slider
    /* +0x500 */ float    ff_global_volume;       // [H] g_force_feedback_global_volume, car force-feedback slider
    /* +0x504 */ float    ff_suspension;          // [H] g_force_feedback_suspension, car force-feedback slider
    /* +0x508 */ float    ff_collisions;          // [H] g_force_feedback_collisions, car force-feedback slider
    /* +0x50c */ float    ff_revs;                // [H] g_force_feedback_revs, car force-feedback slider
    /* +0x510 */ float    ff_tire_grip;           // [H] g_force_feedback_tire_grip, car force-feedback slider
    /* +0x514 */ float    ff_strength;            // [H] g_force_feedback_strength, the "Player Movement" force-feedback slider
    /* +0x518 */ uint32_t cursor_style;           // [H] g_cursor_style, the cursor style button
    /* +0x51c */ uint8_t  speed_unit_is_mph;      // [H] g_speed_unit_is_mph, the speedometer unit
    /* +0x51d */ uint8_t  side_roll_enabled;      // [H] g_side_roll_enabled, the Side Roll toggle
    /* +0x51e */ uint8_t  mouse_inverted;         // [H] g_mouse_control_inverted, mouse Y inversion
    /* +0x51f */ uint8_t  subtitles_enabled;      // [H] g_subtitles_enabled, the Subtitles toggle
    /* +0x520 */ uint8_t  net_settings[0x9c];     // [H] g_net_settings_saved, the saved multiplayer settings, structs/net_session.md section 1b
    /* +0x5bc */ uint32_t unread_5bc[6];          // [U] cleared by the constructor; never read
    /* +0x5d4 */ void    *load_progress_screen;   // [H] the screen shown while a savegame reopens its mission
} G_LoadSaveClass;
```

**The arrays.** `bindings` has 60 entries because `loadsave_get_key_binding_label_id` bounds its slot at `0x3c` and the two conflict walkers
cover `0..0x20` and `0x21..0x3b` (§9). `net_settings` is `0x9c` bytes because the stream reads and writes exactly that block and it mirrors
the session's settings record. `unread_5bc` has six dwords because the constructor's clear loop counts six; **it is touched only by that
clear.** No instruction references its absolute range `0x0067256c..0x00672583`; inside this unit the clear is the only access at
displacements `0x5bc..0x5d3`; and none of the 125 instructions that load the singleton's address is followed by an access in that range. It
is not part of the profile stream. The `garage_band_*` lines are byte ranges only here, because `structs/garage.md` §1 types those fields
with the same PC offsets.

**The profile file** streams `+0x20` (`0x54`), `+0x1f4` (`0x2d0`), `+0x4c4` (`0x5c`) and `+0x520` (`0x9c`) in that order:
`loadsave_profile_load` and `loadsave_profile_save` are exact inverses, which is how the block sizes are byte-proven.

### 1b. `S_profile_head` — the profile file header and one element of the profile vector

```c
typedef struct S_profile_head {                   // 0x54 bytes; PS2 type G_SaveGame_FileHead
    /* +0x00 */ uint32_t sigil;                   // [H] 'Prof' (0x50726f66), written on save, tested on load
    /* +0x04 */ uint32_t version;                 // [H] 1, written on save, tested on load
    /* +0x08 */ uint32_t number;                  // [H] the profile's number in savegame\mafia%03d.sav; a load keeps the number it had
    /* +0x0c */ uint32_t unread_0c;               // [U] loadsave_profile_save writes 0; never read
    /* +0x10 */ char     name[0x34];              // [H] the profile name, g_loadsave_active_profile_name for the active record
    /* +0x44 */ uint32_t freeride_mode;           // [H] the copy of G_LoadSaveClass+0x1f0 in the file
    /* +0x48 */ uint32_t freeride_cars;           // [H] the copy of G_LoadSaveClass+0x9c in the file
    /* +0x4c */ uint32_t unread_4c;               // [U] loadsave_profile_save writes 0; never read
    /* +0x50 */ uint32_t race_unlock_mask;        // [H] the copy of G_LoadSaveClass+0x94 in the file
} S_profile_head;
```

`name` is `0x34` bytes because it is the room between `+0x10` and the next located field; the name functions treat it as a NUL-terminated string.

### 1c. `S_savegame_head` — the slot file header and one element of the slot vector

```c
typedef struct S_savegame_head {                  // 0x18 bytes
    /* +0x00 */ uint32_t sigil;                   // [H] 0x47766153, written by loadsave_savegame_save
    /* +0x04 */ uint32_t version;                 // [H] 0x10, written on save, tested on load
    /* +0x08 */ uint32_t number;                  // [H] the slot number in savegame\mafia%03d.%03d
    /* +0x0c */ uint32_t unread_0c;               // [U] zeroed with the record and streamed; never read
    /* +0x10 */ uint32_t unread_10;               // [U] zeroed with the record and streamed; never read
    /* +0x14 */ uint32_t mission_name_id;         // [H] copied from G_LoadSaveClass+0x1d0 on save
} S_savegame_head;
```

### 1d. `S_key_binding` — one row of the table of §9

```c
typedef struct S_key_binding {                    // 0xc bytes
    /* +0x00 */ int32_t  mode;                    // [H] 0 whole axis, 1 positive half, 2 negative half
    /* +0x04 */ int16_t  device_kind;             // [H] 1 keyboard, 2 mouse, 3 joystick (section 9)
    /* +0x06 */ uint16_t code;                    // [H] a scancode, button or axis code by device_kind
    /* +0x08 */ int32_t  device_index;            // [H] the joystick; a stale one renders as "Not attached joy"
} S_key_binding;
```

### 1e. `G_Stream` — the encrypted file stream

```c
typedef struct G_Stream {                         // 0x3c bytes, on the caller's stack
    /* +0x00 */ uint8_t  encrypt;                 // [H] stream_set_params' second argument; gates the cipher (section 6)
    /* +0x01 */ uint8_t  unread_01;               // [U] stream_set_params' third argument, always 0; never read
    /* +0x04 */ uint32_t key1;                    // [H] seeded 0x23101976, section 6
    /* +0x08 */ uint32_t key2;                    // [H] seeded 0x10072002, section 6
    /* +0x0c */ uint32_t magic;                   // [H] stream_set_params' first argument, copied into the file header
    /* +0x10 */ uint32_t file_header[6];          // [H] the 0x18 bytes stream_open_write writes first: [0] = magic, [2] = 1
    /* +0x28 */ FILE    *file;                    // [H] crt_fopen's result; 0 = closed
    /* +0x2c */ uint8_t  unread_2c;               // [U] stream_ctor writes 0; never read
    /* +0x2d */ uint8_t  error_flag;              // [M] what stream_close returns; only stream_ctor writes it
    /* +0x30 */ uint32_t block_size;              // [H] the block stream_begin_block allocates
    /* +0x34 */ uint32_t unread_34;               // [U] stream_ctor writes 0; never read
    /* +0x38 */ void    *block;                   // [H] the block buffer of stream_begin_block and stream_read_block
} G_Stream;
```

`file_header` is six dwords because `stream_open_write` zeroes six and `crt_fwrite` writes `0x18` bytes.

### 1f. `G_EnabledCars` — the car permission bitmap

```c
typedef struct G_EnabledCars {                    // 0x40 bytes
    /* +0x00 */ uint32_t flat[4];                 // [H] used when enabled_cars_have_car's third argument is 0: index = the id's low 16 bits
    /* +0x10 */ uint32_t bank0[4];                // [H] third argument set, byte 2 of the id = 0
    /* +0x20 */ uint32_t bank1[4];                // [H] third argument set, byte 2 of the id = 1
    /* +0x30 */ uint32_t bank2[4];                // [H] third argument set, byte 2 of the id = 2
} G_EnabledCars;
```

Each member holds four dwords because the test indexes `(id >> 5) * 4` for an id's low bits. **The PC has no instance.** The record's pointer is the third
stack argument of `car_database_collect_cars_by_class_mask` and `car_database_collect_cars_by_manufacturer`, which test it for null before calling
`enabled_cars_have_car`; all four of their callers pass 0 (`structs/car_database.md`), so the test is never reached.

### 1g. `S_savegame_info` — the slot file's `0x20`-byte block after the header

```c
typedef struct S_savegame_info {                  // 0x20 bytes
    /* +0x00 */ uint32_t slot_number;             // [H] the active slot's number; the shot%03d.bmp key
    /* +0x04 */ uint32_t unread_04;               // [U] zeroed, never filled; no reader (class census)
    /* +0x08 */ uint8_t  save_second;             // [H] wSecond of GetLocalTime; no reader (class census)
    /* +0x09 */ uint8_t  save_minute;             // [H] wMinute
    /* +0x0a */ uint8_t  save_hour;               // [H] wHour
    /* +0x0b */ uint8_t  unread_0b;               // [U] the packed time dword's top byte, always 0; no reader (class census)
    /* +0x0c */ uint8_t  save_day;                // [H] wDay
    /* +0x0d */ uint8_t  save_month;              // [H] wMonth
    /* +0x0e */ uint16_t save_year;               // [H] wYear
    /* +0x10 */ uint32_t health_percent;          // [H] the player's health in percent
    /* +0x14 */ uint32_t play_time_ms;            // [H] g_hud_elapsed_ms
    /* +0x18 */ uint32_t unread_18;               // [U] zeroed, never filled; no reader (class census)
    /* +0x1c */ uint32_t steal_mask;              // [H] the copy of G_LoadSaveClass+0x98; restored on load
} S_savegame_info;
```

`loadsave_savegame_save` zeroes the eight dwords and fills them after the slot header: `+0x08` is
`(wHour << 16) | (wMinute << 8) | wSecond` and `+0x0c` is `(wYear << 16) | (wMonth << 8) | wDay`,
both from `GetLocalTime`. `loadsave_savegame_load_header` copies the block whole into the load-game
screen (`structs/game_menus.md` §1i); `loadsave_savegame_load` reads it in each of its three passes
and keeps only `steal_mask`, in the first.

### 1h. `S_savegame_scene` — the slot file's `0x108`-byte block after `S_savegame_info`

```c
typedef struct S_savegame_scene {                 // 0x108 bytes
    /* +0x000 */ char     mission_name[0x20];     // [H] the mission's name; NUL at +0x1f
    /* +0x020 */ uint32_t game_block_size;        // [H] game_save_game_get_size; the game block that follows
    /* +0x024 */ uint32_t unread_024;             // [U] zeroed, never filled; no reader
    /* +0x028 */ uint32_t packed_car_ids[25];     // [H] the garage slots' packed_car_id (§2h)
    /* +0x08c */ uint32_t previous_packed_ids[25]; // [H] the garage's previous_packed_ids
    /* +0x0f0 */ uint32_t ai_groups_size;         // [H] ai_groups_get_save_size; 0 = no block
    /* +0x0f4 */ uint32_t ai_follow_size;         // [H] ai_follow_manager_get_save_size; 0 = no block
    /* +0x0f8 */ uint32_t freeride_active;        // [H] g_freeride_active
    /* +0x0fc */ float    police_ratio;           // [H] g_freeride_stored_traffic_police_ratio
    /* +0x100 */ float    car_ratio;              // [H] g_freeride_stored_traffic_car_ratio
    /* +0x104 */ float    all_ratio;              // [H] g_freeride_stored_traffic_all_ratio
} S_savegame_scene;
```

`loadsave_savegame_save` zeroes the block, fills it and writes it after `S_savegame_info`, then the game
block, the `ai_groups` and `ai_follow_manager` blocks when their sizes are not 0, and one
`S_savegame_actor_head` with its block per saved actor. `loadsave_savegame_load` reads it in each of its
three passes. The two car-id arrays are 25 entries because the garage table has 25 slots
(`structs/garage.md` §1).

### 1i. `S_savegame_actor_head` — the record before each saved actor's block

```c
typedef struct S_savegame_actor_head {            // 0x8c bytes; PS2 type G_SaveGame_ActorHead
    /* +0x00 */ char     frame_name[0x40];        // [H] the actor's frame name; NUL at +0x3f
    /* +0x40 */ char     model_name[0x40];        // [H] a MODEL frame's model file name without its directory; else empty
    /* +0x80 */ uint32_t kind;                    // [H] the actor's kind (+0x10)
    /* +0x84 */ uint32_t block_size;              // [H] the size of the actor's block that follows
    /* +0x88 */ int32_t  garage_slot;             // [H] kind 4: its index in the 25-slot garage table; else -1
} S_savegame_actor_head;
```

`loadsave_save_actor_proc` fills it from the actor and its frame (`frame+0x100`, and `frame+0x154` when
`frame_type` is 9), writes it and then the block the actor's own save slot fills. `loadsave_savegame_save`
saves the kind-4 and kind-8 actors first, then the rest; `loadsave_savegame_load` reads records until the
stream ends.

## 2. Field evidence

Cells come from a census of `loadsave_*` and `garage_*` over the singleton's receiver and its absolute address, read against the bodies.

### 2a. `G_LoadSaveClass`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | profiles_alloc | H | `loadsave_ctor` | - | an empty class |
| +0x4 | profiles_begin | H | `loadsave_ctor`, `loadsave_dtor`, `loadsave_profile_enum_files` | `loadsave_dtor`, `loadsave_profile_count`, `loadsave_profile_create`, `loadsave_profile_delete`, `loadsave_profile_enum_files`, `loadsave_profile_enum_release`, `loadsave_profile_select` | - |
| +0x8 | profiles_end | H | `loadsave_ctor`, `loadsave_dtor`, `loadsave_profile_delete`, `loadsave_profile_enum_files`, `loadsave_profile_enum_release` | `loadsave_profile_count`, `loadsave_profile_create`, `loadsave_profile_delete`, `loadsave_profile_enum_files`, `loadsave_profile_enum_release`, `loadsave_profile_select` | - |
| +0xc | profiles_cap | H | `loadsave_ctor`, `loadsave_dtor`, `loadsave_profile_enum_files` | `loadsave_profile_enum_files` | - |
| +0x10 | slots_alloc | H | `loadsave_ctor` | - | an empty class |
| +0x14 | slots_begin | H | `loadsave_ctor`, `loadsave_dtor` | `loadsave_dtor`, `loadsave_savegame_create`, `loadsave_savegame_enum_files`, `loadsave_savegame_enum_release`, `loadsave_savegame_load_header`, `loadsave_savegame_select` | - |
| +0x18 | slots_end | H | `loadsave_ctor`, `loadsave_dtor`, `loadsave_savegame_enum_release` | `loadsave_savegame_enum_files`, `loadsave_savegame_enum_release`, `loadsave_savegame_load_header`, `loadsave_savegame_select` | - |
| +0x1c | slots_cap | H | `loadsave_ctor`, `loadsave_dtor` | - | - |
| +0x20 | active_profile | H | `loadsave_profile_load`, `loadsave_profile_save`, `loadsave_profile_select` | `loadsave_profile_load`, `loadsave_profile_save`, `loadsave_profile_select`, `loadsave_replay_load`, `loadsave_replay_save`, `loadsave_savegame_enum_files`, `loadsave_savegame_load`, `loadsave_savegame_load_header`, `loadsave_savegame_save`, `racing_load_champs_profile`, `racing_save_champs_profile` | §2b |
| +0x74 | active_slot | H | `loadsave_savegame_save`, `loadsave_savegame_select` | `loadsave_savegame_get_selected_number`, `loadsave_savegame_load`, `loadsave_savegame_save`, `loadsave_savegame_select` | §2c |
| +0x8c | replay_mode | H | `loadsave_ctor`, `loadsave_replay_set_mode`, `loadsave_replay_tick` | `loadsave_replay_tick` | - |
| +0x90 | unread_90 | U | `loadsave_init`, `loadsave_replay_set_mode` | - | both store 1; no reader in the class census or in a scan of `.text` for its absolute address (§10k) |
| +0x91 | profile_load_failed | H | `loadsave_ctor`, `loadsave_profile_load` | `loadsave_profile_load` | - |
| +0x94 | race_unlock_mask | H | `loadsave_init`, `loadsave_profile_load`, `loadsave_race_cars_disable`, `loadsave_race_cars_enable`, `loadsave_race_cars_unlock_from_car` | `loadsave_freeride_apply_script_params`, `loadsave_profile_save`, `loadsave_race_cars_disable`, `loadsave_race_cars_enable`, `loadsave_race_cars_get_enabled`, `loadsave_race_cars_unlock_from_car` | - |
| +0x98 | steal_mask | H | `garage_enable_steal_for_letter`, `loadsave_freeride_apply_script_params`, `loadsave_savegame_load`, `loadsave_savegame_new_game` | `garage_can_steal_car_model`, `loadsave_savegame_save` | `structs/garage.md` §2 |
| +0x9c | freeride_cars | H | `loadsave_all_default`, `loadsave_freeride_enable_car_group`, `loadsave_init`, `loadsave_profile_load` | `loadsave_freeride_get_enabled_cars`, `loadsave_profile_save` | §10e |
| +0xa0 | garage_band_b | H | `garage_game_init`, `garage_request_add_last_stolen`, `garage_request_show_results` | `garage_add_car_by_actor`, `garage_add_car_by_index`, `garage_clear`, `garage_remove_car_by_model`, `garage_run_car_select_menu`, `garage_show_results`, `loadsave_savegame_save` | `structs/garage.md` §2 |
| +0x1d0 | mission_name_id | H | `loadsave_savegame_from_script`, `loadsave_set_mission_name_id` | `loadsave_savegame_save` | - |
| +0x1d4 | suppress_next_save | H | `loadsave_profile_load`, `loadsave_savegame_from_script` | `loadsave_savegame_from_script` | §10h |
| +0x1d8 | objectives_alloc | H | `loadsave_ctor` | `loadsave_objectives_add` | an empty class |
| +0x1dc | objectives_begin | H | `loadsave_ctor`, `loadsave_dtor` | `loadsave_dtor`, `loadsave_objectives_add`, `loadsave_objectives_clear`, `loadsave_objectives_display`, `loadsave_objectives_remove` | - |
| +0x1e0 | objectives_end | H | `loadsave_ctor`, `loadsave_dtor`, `loadsave_objectives_clear`, `loadsave_objectives_remove` | `loadsave_objectives_add`, `loadsave_objectives_clear`, `loadsave_objectives_display`, `loadsave_objectives_remove` | - |
| +0x1e4 | objectives_cap | H | `loadsave_ctor`, `loadsave_dtor` | `loadsave_objectives_add` | - |
| +0x1e8 | garage_band_c | H | `garage_set_blocked_car`, `garage_set_last_entered_car`, `loadsave_init`, `loadsave_savegame_new_game` | `garage_run_car_select_menu` | `structs/garage.md` §2 |
| +0x1f0 | freeride_mode | H | `loadsave_all_default`, `loadsave_freeride_set_mode`, `loadsave_init`, `loadsave_profile_load` | `gui_free_ride_ctor`, `gui_free_ride_mode_ctor`, `gui_main_menu_ctor`, `loadsave_freeride_apply_script_params`, `loadsave_profile_save` | - |
| +0x1f4 | bindings | H | `loadsave_binding_clear_conflicts` (through `loadsave_binding_set`), `loadsave_binding_set`, `loadsave_binding_set_from_event`, `loadsave_ctor`, `loadsave_default_key_bindings`, `loadsave_profile_load` (by address, `stream_read_at`) | `input_binding_get_label`, `input_set_binding`, `input_update`, `loadsave_binding_clear_conflicts`, `loadsave_binding_find_conflict`, `loadsave_profile_save` (by address, `stream_write_at`) | §9; `loadsave_ctor` and `loadsave_default_key_bindings` zero the whole table; `input_set_binding`, `input_update` and `input_binding_get_label` read it through their own pointer |
| +0x4c4 | steering_linearity | H | `gui_options_audio_video_tick`, `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update`, `loadsave_ctor`, `loadsave_default_settings`, `loadsave_profile_load`, `loadsave_profile_save`, `player_update_controls` | - |
| +0x4c8 | blood_level | H | `gui_options_audio_video_tick` | `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `hud_push_blood_splat` | `hud_push_blood_splat` pushes a splat only at 0.9 or more |
| +0x4cc | look_filter_scale | H | `loadsave_ctor` | `camera_tick` | `camera_tick` walks the 30 records of `g_camera_frame_time_filter_ring` and weights each older sample by `pow(look_filter_base, look_filter_scale × accumulated frame time × 100)`, then scales the mean by `aiming_sensitivity_x` / `_y`; only the zeroing and the profile stream write it, so it is 0 on the PC |
| +0x4d0 | look_filter_base | H | `loadsave_ctor` | `camera_tick` | the base of that `pow` (`fld [0x00672480]` at `0x005d8cf9`); 0 on the PC |
| +0x4d4 | unread_4d4 | U | `loadsave_ctor` | - | zeroed and moved whole by the profile stream; no other access in `.text` |
| +0x4d8 | aiming_sensitivity_x | H | `gui_options_audio_video_tick`, `gui_options_controls_tick`, `loadsave_default_settings` | `camera_free_look`, `camera_free_look_base`, `camera_tick`, `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `player_ai_walk` | - |
| +0x4dc | aiming_sensitivity_y | H | `gui_options_audio_video_tick`, `gui_options_controls_tick`, `loadsave_default_settings` | `camera_free_look`, `camera_free_look_base`, `camera_tick`, `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `player_ai_walk` | - |
| +0x4e0 | max_aiming_speed_x | H | `gui_options_audio_video_tick`, `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `player_ai_walk` | - |
| +0x4e4 | sounds_volume | H | `gui_options_audio_video_tick`, `loadsave_default_settings` | `game_init`, `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `human_do_throw_occupant_from_car`, `human_game_init`, `human_new_scream`, `loadsave_apply_settings`, `mission_open`, `mission_tick`, `pedestrian_manager_play_pedestrian_scream`, `speech_channel_speak_line` | - |
| +0x4e8 | music_volume | H | `gui_options_audio_video_tick`, `loadsave_default_settings` | `game_media_apply_volumes`, `game_media_create`, `game_media_fade`, `game_media_fade_all`, `game_save_game_load`, `game_save_game_save`, `game_tick`, `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `program_process_command` | - |
| +0x4ec | cars_volume | H | `gui_options_audio_video_tick`, `loadsave_default_settings` | `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `loadsave_apply_settings` | - |
| +0x4f0 | speech_volume | H | `gui_options_audio_video_tick`, `loadsave_default_settings` | `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `human_do_throw_occupant_from_car`, `human_game_init`, `human_new_scream`, `pedestrian_manager_play_pedestrian_scream`, `speech_channel_speak_line` | - |
| +0x4f4 | camera_onfoot_mode | H | `camera_set_mode`, `loadsave_all_default`, `loadsave_apply_settings` | `camera_link_to_frame`, `camera_reset`, `camera_set_car`, `loadsave_apply_settings` | - |
| +0x4f8 | camera_vehicle_mode | H | `camera_set_mode`, `loadsave_all_default`, `loadsave_apply_settings` | `camera_set_car`, `camera_set_climbing_into_car`, `loadsave_apply_settings` | - |
| +0x4fc | gamma | H | `gui_options_audio_video_tick`, `loadsave_default_settings` | `gui_options_audio_video_ctor`, `gui_options_audio_video_tick`, `loadsave_apply_settings` | - |
| +0x500 | ff_global_volume | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `loadsave_apply_settings` | `loadsave_apply_settings` (bit `8`) passes it to `car_set_force_feedback_gains` (`structs/force_feedback.md` §8c) |
| +0x504 | ff_suspension | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `loadsave_apply_settings` | `loadsave_apply_settings` (bit `8`) passes it to `car_set_force_feedback_gains` (`structs/force_feedback.md` §8c) |
| +0x508 | ff_collisions | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `loadsave_apply_settings` | `loadsave_apply_settings` (bit `8`) passes it to `car_set_force_feedback_gains` (`structs/force_feedback.md` §8c) |
| +0x50c | ff_revs | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `loadsave_apply_settings` | `loadsave_apply_settings` (bit `8`) passes it to `car_set_force_feedback_gains` (`structs/force_feedback.md` §8c) |
| +0x510 | ff_tire_grip | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `loadsave_apply_settings` | `loadsave_apply_settings` (bit `8`) passes it to `car_set_force_feedback_gains` (`structs/force_feedback.md` §8c) |
| +0x514 | ff_strength | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `human_do_jump`, `human_hit`, `human_movement_intern`, `human_shooting` | scales every human effect |
| +0x518 | cursor_style | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `hud_render_all` | - |
| +0x51c | speed_unit_is_mph | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `hud_load_speedometer_odometer_texture` | - |
| +0x51d | side_roll_enabled | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `player_ai_walk` | - |
| +0x51e | mouse_inverted | H | `gui_options_controls_tick`, `loadsave_default_settings` | `camera_free_look`, `camera_free_look_base`, `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `player_ai_walk` | - |
| +0x51f | subtitles_enabled | H | `gui_options_controls_tick`, `loadsave_default_settings` | `gui_options_controls_on_exchange`, `gui_options_controls_tick`, `hud_render_all` | - |
| +0x520 | net_settings | H | `net_session_settings_save` | `loadsave_all_default`, `loadsave_ctor`, `loadsave_profile_load`, `net_session_settings_apply_saved` | `net_session_settings_apply_saved` hands it back to the session; zeroed by `loadsave_all_default` |
| +0x5bc | unread_5bc | U | `loadsave_ctor` | - | the constructor's clear is the only access (§1a) |
| +0x5d4 | load_progress_screen | H | `game_init`, `loadsave_ctor`, `loadsave_savegame_load`, `mission_open` | `game_init`, `gui_show_load_progress`, `load_progress_callback`, `mission_open`, `race_net_load_sync` | `loadsave_savegame_load` creates it (`operator_new(0x50)`, `gui_load_progress_ctor`) and destroys it after `game_init` |

### 2b. `S_profile_head`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | sigil | H | `loadsave_profile_save` | `loadsave_profile_load` | - |
| +0x4 | version | H | `loadsave_profile_save` | `loadsave_profile_load` | - |
| +0x8 | number | H | `loadsave_profile_enum_files`, `loadsave_profile_load`, `loadsave_profile_select` | `loadsave_profile_load`, `loadsave_profile_save`, `loadsave_replay_load`, `loadsave_replay_save`, `loadsave_savegame_load`, `loadsave_savegame_load_header`, `loadsave_savegame_save` | the file names |
| +0xc | unread_0c | U | `loadsave_profile_save` | - | `loadsave_profile_create` writes a zeroed head; the load reads back `+0x44`, `+0x48` and `+0x50` only |
| +0x10 | name | H | `loadsave_profile_create`, `loadsave_profile_select` | `loadsave_profile_create` | `net_session_settings_init_defaults` copies it |
| +0x44 | freeride_mode | H | `loadsave_profile_save` | `loadsave_profile_load` | - |
| +0x48 | freeride_cars | H | `loadsave_profile_save` | `loadsave_profile_load` | - |
| +0x4c | unread_4c | U | `loadsave_profile_save` | - | the same |
| +0x50 | race_unlock_mask | H | `loadsave_profile_save` | `loadsave_profile_load` | - |

### 2c. `S_savegame_head`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | sigil | H | `loadsave_savegame_save` | - | the stream's own magic is the other spelling, `0x53617647` |
| +0x4 | version | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `== 0x10` gates a load |
| +0x8 | number | H | `loadsave_savegame_create`, `loadsave_savegame_select` | `loadsave_savegame_get_selected_number`, `loadsave_savegame_load`, `loadsave_savegame_load_header`, `loadsave_savegame_save` | - |
| +0xc | unread_0c | U | `loadsave_savegame_create` | - | zero-filled with the record and streamed by `loadsave_savegame_save`; the load reads `+0x04` and `+0x08` only |
| +0x10 | unread_10 | U | `loadsave_savegame_create` | - | the same |
| +0x14 | mission_name_id | H | `loadsave_savegame_save` | - | no reader: the load-game screen (`gui_load_game_tick`) reads `S_savegame_info`, not this field |

### 2d. `S_key_binding`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | mode | H | `loadsave_binding_set`, `loadsave_binding_set_from_event`, `loadsave_default_key_bindings` | `input_binding_get_label`, `input_set_binding`, `input_update`, `loadsave_binding_clear_conflicts`, `loadsave_binding_find_conflict` | - |
| +0x4 | device_kind | H | `loadsave_binding_set`, `loadsave_binding_set_from_event`, `loadsave_default_key_bindings` | `input_binding_get_label`, `input_set_binding`, `input_update`, `loadsave_binding_clear_conflicts`, `loadsave_binding_find_conflict` | the two conflict walkers compare at `+0x06` and `+0x04` |
| +0x6 | code | H | `loadsave_binding_set`, `loadsave_binding_set_from_event`, `loadsave_default_key_bindings` | `input_binding_get_label`, `input_set_binding`, `input_update`, `loadsave_binding_clear_conflicts`, `loadsave_binding_find_conflict` | - |
| +0x8 | device_index | H | `loadsave_binding_set`, `loadsave_binding_set_from_event`, `loadsave_default_key_bindings` | `input_binding_get_label`, `input_set_binding`, `input_update`, `loadsave_binding_clear_conflicts`, `loadsave_binding_find_conflict` | - |

### 2e. `G_Stream`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | encrypt | H | `stream_set_params` | `stream_end_block`, `stream_read_at`, `stream_read_block`, `stream_write_at` | - |
| +0x1 | unread_01 | U | `stream_set_params` | - | all eight `stream_set_params` calls pass `(magic, 1, 0)` |
| +0x4 | key1 | H | `stream_ctor`, `stream_decrypt_dwords`, `stream_encrypt_dwords`, `stream_open_read`, `stream_open_write` | `stream_decrypt_dwords`, `stream_encrypt_dwords` | - |
| +0x8 | key2 | H | `stream_ctor`, `stream_decrypt_dwords`, `stream_encrypt_dwords`, `stream_open_read`, `stream_open_write` | `stream_decrypt_dwords`, `stream_encrypt_dwords` | - |
| +0xc | magic | H | `stream_set_params` | `stream_open_write` | - |
| +0x10 | file_header | H | `stream_open_write` | `stream_open_write` | `stream_open_write` zeroes and writes it |
| +0x28 | file | H | `stream_close`, `stream_ctor`, `stream_open_read`, `stream_open_write` | `stream_close`, `stream_end_block`, `stream_read_at`, `stream_read_block`, `stream_write_at` | - |
| +0x2c | unread_2c | U | `stream_ctor` | - | - |
| +0x2d | error_flag | M | `stream_ctor` | `stream_close` | no non-zero writer, so `stream_close` and its thunk return false (§10k) |
| +0x30 | block_size | H | `stream_begin_block`, `stream_ctor`, `stream_end_block` | `stream_end_block` | - |
| +0x34 | unread_34 | U | `stream_ctor` | - | - |
| +0x38 | block | H | `stream_begin_block`, `stream_ctor`, `stream_read_block` | `stream_end_block`, `stream_read_block` | - |

### 2f. `G_EnabledCars`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | flat | H | - | `enabled_cars_have_car` | no instance on the PC (§1f) |
| +0x10 | bank0 | H | - | `enabled_cars_have_car` | - |
| +0x20 | bank1 | H | - | `enabled_cars_have_car` | - |
| +0x30 | bank2 | H | - | `enabled_cars_have_car` | - |

### 2g. `S_savegame_info`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | slot_number | H | `loadsave_savegame_save` | `gui_load_game_tick` | the active slot's `number`; passed to `loadsave_get_screenshot_idx` for the `shot%03d.bmp` name |
| +0x4 | unread_04 | U | `loadsave_savegame_save` | - | zeroed, never filled; no reader (class census) |
| +0x8 | save_second | H | `loadsave_savegame_save` | - | `wSecond`; no reader (class census) |
| +0x9 | save_minute | H | `loadsave_savegame_save` | `gui_load_game_tick` | `wMinute`, in the format of textdb `0xaf` |
| +0xa | save_hour | H | `loadsave_savegame_save` | `gui_load_game_tick` | `wHour` |
| +0xb | unread_0b | U | `loadsave_savegame_save` | - | the packed time dword's top byte, always 0; no reader (class census) |
| +0xc | save_day | H | `loadsave_savegame_save` | `gui_load_game_tick` | `wDay` |
| +0xd | save_month | H | `loadsave_savegame_save` | `gui_load_game_tick` | `wMonth` |
| +0xe | save_year | H | `loadsave_savegame_save` | `gui_load_game_tick` | `wYear` |
| +0x10 | health_percent | H | `loadsave_savegame_save` | `gui_load_game_tick` | 100 without a player, at least 1 while alive |
| +0x14 | play_time_ms | H | `loadsave_savegame_save` | `gui_load_game_tick` | `g_hud_elapsed_ms`; the tick splits it into hours, minutes, seconds |
| +0x18 | unread_18 | U | `loadsave_savegame_save` | - | zeroed, never filled; no reader (class census) |
| +0x1c | steal_mask | H | `loadsave_savegame_save` | `loadsave_savegame_load` | the copy of `steal_mask` (`+0x98`); the load's first pass stores it back there |

### 2h. `S_savegame_scene`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | mission_name | H | `loadsave_savegame_save` | `loadsave_savegame_load` | the mission name at `C_mission+0x68`; the load compares it with the mission it opens |
| +0x20 | game_block_size | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `game_save_game_get_size`; the size of the block `stream_read_block` reads next |
| +0x24 | unread_024 | U | `loadsave_savegame_save` | - | zeroed with the block, never filled; no reader |
| +0x28 | packed_car_ids | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `G_LoadSaveClass+0xa0 + 8i`; the load stores each back with byte 2 cleared, clears the slot's second dword and clamps the id |
| +0x8c | previous_packed_ids | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `G_LoadSaveClass+0x168 + 4i`; the load stores each back with byte 2 cleared and clamps it |
| +0xf0 | ai_groups_size | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `ai_groups_get_save_size`; a block follows when it is not 0 |
| +0xf4 | ai_follow_size | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `ai_follow_manager_get_save_size`; a block follows when it is not 0 |
| +0xf8 | freeride_active | H | `loadsave_savegame_save` | `loadsave_savegame_load` | `g_freeride_active`; restored as a bool |
| +0xfc | police_ratio | H | `loadsave_savegame_save` | `loadsave_savegame_load` | restored into `g_freeride_stored_traffic_police_ratio` |
| +0x100 | car_ratio | H | `loadsave_savegame_save` | `loadsave_savegame_load` | restored into `g_freeride_stored_traffic_car_ratio` |
| +0x104 | all_ratio | H | `loadsave_savegame_save` | `loadsave_savegame_load` | restored into `g_freeride_stored_traffic_all_ratio` |

### 2i. `S_savegame_actor_head`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | frame_name | H | `loadsave_save_actor_proc` | `loadsave_savegame_load` | `frame+0x100`, cut to `0x3f` characters |
| +0x40 | model_name | H | `loadsave_save_actor_proc` | `loadsave_savegame_load` | `frame+0x154` after the last `\` or `/`, only when `frame_type` is 9 |
| +0x80 | kind | H | `loadsave_save_actor_proc` | `loadsave_savegame_load` | `actor+0x10` |
| +0x84 | block_size | H | `loadsave_save_actor_proc` | `loadsave_savegame_load` | the actor's save-size slot result; the size `stream_read_block` reads next |
| +0x88 | garage_slot | H | `loadsave_save_actor_proc` | `loadsave_savegame_load` | the index whose `G_LoadSaveClass+0xa4 + 8i` is the actor, below 25; else -1 |

## 3. Method family

This unit holds 176 function starts, all named, 32 of them PC-only. The table covers the singleton, `G_Stream`, `G_EnabledCars`, the unit's
CRT rows and the helpers the unit hosts; `G_CarDatabase` and `G_GarageClass` rows are in their own documents. Six functions that would read as
`C_game`'s wear the class prefix instead, all six body-diffed CONFIRMED (§10h): `loadsave_savegame_create`, `loadsave_savegame_load`,
`loadsave_save_actor_proc`, `loadsave_savegame_save`, `loadsave_savegame_from_script` and `loadsave_execute_credits`.

### 3a. Construction & destruction

#### `G_LoadSaveClass`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00556ee0 | `crt_static_init_guard_00556ee0` | - | - | - | - | The unit's opening guard |
| 0x00556f00 | `crt_atexit_register_00556f00` | - | - | - | - | The guard's `atexit` helper |
| 0x00556f10 | `loadsave_static_init` | `0x00671fb0` | - | - | - | First of the quartet |
| 0x00556f20 | `loadsave_static_ctor_call` | `0x00671fb0` | - | - | - | Second: calls `loadsave_ctor` |
| 0x00556f30 | `loadsave_static_atexit_register` | `0x00671fb0` | - | - | - | Third: registers the destructor |
| 0x00556f40 | `loadsave_static_dtor_call` | `0x00671fb0` | - | - | - | Fourth: calls `loadsave_dtor` |
| 0x005571d0 | `loadsave_ctor` | instance | `__ct__15G_LoadSaveClassFv` | 0x002b69d0 | CONFIRMED | Zeroes the vectors and blocks; sets `profile_load_failed` |
| 0x00560890 | `loadsave_dtor` | instance | `__dt__15G_LoadSaveClassFv` | 0x002bfc00 | CONFIRMED | Frees the three vectors |

#### `G_Stream`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0055d010 | `stream_ctor` | instance | `__ct__8G_StreamFv` | 0x002bb860 | CONFIRMED | Seeds the two keys, clears the rest |

### 3b. Accessors

#### `G_LoadSaveClass`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00557eb0 | `loadsave_savegame_get_selected_number` | `SaveGameGetSelectedNumber__15G_LoadSaveClassFv` | 0x002b79d0 | CONFIRMED | Returns the active save slot's number |
| 0x0055d6f0 | `loadsave_race_cars_get_enabled` | `RaceCarsGetEnabled__15G_LoadSaveClassFv` | 0x002bc420 | CONFIRMED | Returns the race-unlock mask |
| 0x0055ffb0 | `loadsave_freeride_get_enabled_cars` | `FreerideGetEnabledCars__15G_LoadSaveClassFv` | 0x002bc730 | CONFIRMED | `return +0x9c`, the Free Ride enabled-car mask |
| 0x005608e0 | `loadsave_freeride_set_mode` | `FreerideEnable__15G_LoadSaveClassFUi` | 0x002bd300 | CONFIRMED | Sets the Free Ride mode word |
| 0x005c8770 | `loadsave_set_mission_name_id` | - | - | - | Stores the textdb id a new slot's header takes |

### 3c. Behaviour

#### `G_LoadSaveClass`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00556f90 | `loadsave_init` | `Init__15G_LoadSaveClassFv` | 0x002b61d0 | CONFIRMED | Defaults, HUD, tables, profile selection menu |
| 0x005570c0 | `loadsave_close` | `Close__15G_LoadSaveClassFv` | 0x002b66c0 | CONFIRMED | Five statements including the `profileIgnoreSave` guard |
| 0x00557100 | `loadsave_profile_change` | `ProfileChange__15G_LoadSaveClassFv` | 0x002b6720 | CONFIRMED | Saves the profile, runs the profile-select menu, loads the choice |
| 0x00557270 | `loadsave_all_default` | `AllDefault__15G_LoadSaveClassFv` | 0x002b6ba0 | CONFIRMED | Resets every persisted setting to its factory value |
| 0x005572f0 | `loadsave_profile_enum_files` | `ProfileEnumFiles__15G_LoadSaveClassFv` | 0x002b6c80 | CONFIRMED | Enumerates `savegame\mafia%03d.sav` into the profile vector |
| 0x005575c0 | `loadsave_profile_enum_release` | `ProfileEnumRelease__15G_LoadSaveClassFv` | 0x002b6db0 | CONFIRMED | Empties the enumerated profile list |
| 0x005575f0 | `folded_return_this` | - | - | - | Realises `ProfileGetList`, a three-owner fold (§10i) |
| 0x00557600 | `loadsave_profile_select` | `ProfileSelect__15G_LoadSaveClassFUi` | 0x002b6df0 | CONFIRMED | Makes profile[idx] the active profile |
| 0x00557650 | `loadsave_profile_create` | `ProfileCreate__15G_LoadSaveClassFPUc` | 0x002b6f00 | CONFIRMED | Creates a new player profile file and selects it |
| 0x005577d0 | `loadsave_profile_delete` | `ProfileDelete__15G_LoadSaveClassFUi` | 0x002b7150 | CONFIRMED | Deletes a profile and every savegame slot belonging to it |
| 0x00557880 | `loadsave_profile_load` | `ProfileLoad__15G_LoadSaveClassFv` | 0x002b7300 | CONFIRMED | Reads the selected profile file into the live settings |
| 0x00557a20 | `loadsave_profile_save` | `ProfileSave__15G_LoadSaveClassFv` | 0x002b7490 | CONFIRMED | Writes the live settings back to the selected profile file |
| 0x00557b50 | `loadsave_savegame_enum_files` | `SaveGameEnumFiles__15G_LoadSaveClassFb` | 0x002b76c0 | CONFIRMED | Enumerates the save slots of the active profile |
| 0x00557e40 | `loadsave_savegame_enum_release` | `SaveGameEnumRelease__15G_LoadSaveClassFv` | 0x002b7960 | CONFIRMED | Empties the enumerated save-slot list |
| 0x00557e70 | `loadsave_savegame_select` | `SaveGameSelect__15G_LoadSaveClassFUi` | 0x002b7970 | CONFIRMED | Makes slot[idx] the active save slot |
| 0x00557ec0 | `loadsave_savegame_create` | `SaveGameCreate__15G_LoadSaveClassFUib` | 0x002b79e0 | CONFIRMED | Find-or-create the savegame slot-table entry for a slot id, then select it |
| 0x00558020 | `loadsave_savegame_new_game` | `SaveGameNewGame__15G_LoadSaveClassFv` | 0x002b7b80 | CONFIRMED | Clears the last car, the steal mask, the garage table and the Free Ride flag |
| 0x00558050 | `loadsave_savegame_load_header` | `SaveGameLoadHeader__15G_LoadSaveClassFUiP19G_SaveGame_FileHead` | 0x002b7bb0 | CONFIRMED | Reads one save slot's header without loading the game |
| 0x00558180 | `loadsave_savegame_load` | `SaveGameLoad__15G_LoadSaveClassFv` | 0x002b7c10 | CONFIRMED | Load a savegame slot |
| 0x00558db0 | `loadsave_save_actor_proc` | `_SaveProc__15G_LoadSaveClassFP8G_StreamP7C_actorP20G_SaveGame_ActorHeadUiRUi` | 0x002b8ad0 | CONFIRMED | Serialises one scene object's savegame record |
| 0x00558ed0 | `loadsave_savegame_save` | `SaveGameSave__15G_LoadSaveClassFb` | 0x002b8c40 | CONFIRMED | Write the main savegame file |
| 0x00559310 | `loadsave_get_screenshot_idx` | `GetScreenShotIdx__15G_LoadSaveClassFUi` | 0x002b91f0 | CONFIRMED | Maps a save slot to its screenshot index |
| 0x00559320 | `loadsave_savegame_from_script` | `SaveGameFromScript__15G_LoadSaveClassFUibb` | 0x002b9380 | CONFIRMED | The script-initiated save entry point |
| 0x00559390 | `loadsave_language_select` | `LanguageSelect__15G_LoadSaveClassFUi` | 0x002ba170 | CONFIRMED | Switches the active language |
| 0x005593f0 | `loadsave_binding_set` | - | - | - | Writes one entry of the input-binding table |
| 0x00559430 | `loadsave_binding_set_from_event` | - | - | - | Binds an entry from a raw device event |
| 0x00559490 | `loadsave_binding_find_conflict` | - | - | - | Finds another entry bound to the same input |
| 0x00559550 | `loadsave_binding_clear_conflicts` | - | - | - | Clears every entry that collides with a new binding |
| 0x00559630 | `loadsave_default_key_bindings` | - | - | - | Restores the factory keyboard and mouse bindings |
| 0x00559a90 | `loadsave_apply_key_bindings` | - | - | - | Pushes the binding table into the input driver |
| 0x00559ae0 | `loadsave_default_settings` | `DefaultSettings__15G_LoadSaveClassFUi` | 0x002be3d0 | CONFIRMED | Restores the factory audio and display settings |
| 0x0055d3c0 | `loadsave_replay_tick` | `ReplayTick__15G_LoadSaveClassFRi` | 0x002bc180 | CONFIRMED | Records or replays one frame of input |
| 0x0055d430 | `loadsave_replay_set_mode` | `ReplaySet__15G_LoadSaveClassFbb` | 0x002bc210 | CONFIRMED | Arms recording, playback or neither |
| 0x0055d490 | `loadsave_replay_load` | `ReplayLoad__15G_LoadSaveClassFUi` | 0x002bc250 | CONFIRMED | Loads a recorded replay from disk |
| 0x0055d560 | `loadsave_replay_save` | - | - | - | Writes a recorded replay to disk |
| 0x0055d640 | `loadsave_replay_read_next_frame` | `_replayGetNext__15G_LoadSaveClassFv` | 0x002bc340 | CONFIRMED | Reads the next recorded control word |
| 0x0055d680 | `loadsave_replay_write_next_frame` | `_replayPutNext__15G_LoadSaveClassFUi` | 0x002bc3a0 | CONFIRMED | Appends a control word to the replay buffer |
| 0x0055d6b0 | `loadsave_race_cars_enable` | `RaceCarsEnable__15G_LoadSaveClassFUi` | 0x002bc3f0 | CONFIRMED | Unlocks cars for racing |
| 0x0055d6d0 | `loadsave_race_cars_disable` | `RaceCarsDisable__15G_LoadSaveClassFUi` | 0x002bc400 | CONFIRMED | Locks cars out of racing |
| 0x0055d700 | `loadsave_race_cars_unlock_from_car` | `RaceCarsAddWin__15G_LoadSaveClassFUi` | 0x002bc430 | CONFIRMED | Unlocks whatever the given car unlocks |
| 0x0055d730 | `loadsave_objectives_clear` | `ObjectivesClear__15G_LoadSaveClassFv` | 0x002bc4a0 | CONFIRMED | Empties the current objective list |
| 0x0055d760 | `loadsave_objectives_add` | `ObjectivesAdd__15G_LoadSaveClassFUi` | 0x002bc4b0 | CONFIRMED | Appends one objective string id |
| 0x0055d940 | `loadsave_objectives_remove` | `ObjectivesRemove__15G_LoadSaveClassFUi` | 0x002bc550 | CONFIRMED | Removes one objective string id |
| 0x0055d990 | `loadsave_objectives_display` | `ObjectivesDisplay__15G_LoadSaveClassFv` | 0x002bc5f0 | CONFIRMED | Republishes the objective list to the on-screen mission help |
| 0x0055ff70 | `loadsave_freeride_enable_car_group` | `FreerideEnableCar__15G_LoadSaveClassFUc` | 0x002bc680 | CONFIRMED | ORs bit `letter - 'A'` into the Free Ride enabled-car mask |
| 0x0055ffc0 | `loadsave_apply_settings` | `Refresh__15G_LoadSaveClassFUi` | 0x002bc740 | CONFIRMED | Pushes the settings block into the engine, sound and force feedback |
| 0x00560120 | `loadsave_get_key_binding_label_id` | - | - | - | `g_key_binding_label_string_ids[slot]`, fallback 100 (§9a) |
| 0x00560690 | `loadsave_execute_credits` | `ExecuteCredits__15G_LoadSaveClassFv` | 0x002bceb0 | CONFIRMED | Run the end-credits scroll to completion |
| 0x005608f0 | `loadsave_freeride_store_params` | `FreerideStoreA__15G_LoadSaveClassFUiUifff` | 0x002bd310 | CONFIRMED | Parks the Free Ride world parameters until the scene is ready |
| 0x00560930 | `loadsave_freeride_store_state` | `FreerideStoreB__15G_LoadSaveClassFUi` | 0x002bd340 | CONFIRMED | Parks the Free Ride state word |
| 0x00560940 | `loadsave_freeride_get_state` | `FreerideGetState__15G_LoadSaveClassFv` | 0x002bd350 | CONFIRMED | Reads the parked Free Ride state word |
| 0x00560950 | `loadsave_freeride_apply_script_params` | `FreerideSetupFromScript__15G_LoadSaveClassFv` | 0x002bd360 | CONFIRMED | Publishes the parked Free Ride parameters (§7) |
| 0x005609a0 | `loadsave_freeride_setup_scene` | `FreerideSetupCar__15G_LoadSaveClassFv` | 0x002bd390 | STRUCTURAL | Builds the `Tommy` and `TomyhoAuto` frames and loads both models (§7) |
| 0x00560ec0 | `loadsave_freeride_setup_after` | `FreerideSetupAfter__15G_LoadSaveClassFv` | 0x002be1b0 | CONFIRMED | Seats the player in the car once the scene is live (§7) |
| 0x00560f20 | `loadsave_profile_count` | - | - | - | Number of enumerated profiles |

#### `G_Stream`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0055d040 | `thunk_stream_close` | - | - | - | Jump to `stream_close` |
| 0x0055d050 | `stream_set_params` | `SetParams__8G_StreamFUibb` | 0x002bb900 | CONFIRMED | Sets the file magic and the encryption flags before opening |
| 0x0055d070 | `stream_open_read` | `OpenRead__8G_StreamFPc` | 0x002bb920 | CONFIRMED | Opens a stream for reading and consumes its header |
| 0x0055d0c0 | `stream_open_write` | `OpenWrite__8G_StreamFPc` | 0x002bb9e0 | CONFIRMED | Opens a stream for writing and stamps its header |
| 0x0055d130 | `stream_close` | `Close__8G_StreamFv` | 0x002bbb20 | CONFIRMED | Closes the stream and reports whether anything went wrong |
| 0x0055d160 | `stream_begin_block` | `WantWrite__8G_StreamFUi` | 0x002bbba0 | CONFIRMED | Allocates a pending output block and hands the caller its buffer |
| 0x0055d190 | `stream_end_block` | `Write__8G_StreamFv` | 0x002bbc70 | CONFIRMED | Encrypts and flushes the pending output block |
| 0x0055d1d0 | `stream_read_block` | `Read__8G_StreamFUi` | 0x002bbcc0 | CONFIRMED | Reads and decrypts a block, returning a freshly allocated buffer |
| 0x0055d220 | `stream_encrypt_dwords` | `Crypt__8G_StreamFPvUi` | 0x002bbda0 | STRUCTURAL | Encrypts a dword-aligned buffer with the running-key cipher |
| 0x0055d270 | `stream_decrypt_dwords` | `Decrypt__8G_StreamFPvUi` | 0x002bbdb0 | STRUCTURAL | Decrypts a dword-aligned buffer |
| 0x0055d2b0 | `stream_write_at` | `WriteAdr__8G_StreamFPvUi` | 0x002bbdc0 | CONFIRMED | Writes a caller-owned buffer without modifying it |
| 0x0055d340 | `stream_read_at` | `ReadAdr__8G_StreamFPvUi` | 0x002bc040 | CONFIRMED | Reads into a caller-owned buffer |

#### `G_EnabledCars`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0055c270 | `enabled_cars_have_car` | `HaveCar__13G_EnabledCarsFUib` | 0x002ba1f0 | CONFIRMED | Tests a car's bit (§1f) |

### 3d. Free helpers

#### The unit

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00560140 | `credits_render_one` | `RenderOneCredit__FP15G_CreditsStructf` | 0x002bc880 | CONFIRMED | Draws one credit line |
| 0x005605b0 | `credits_parse` | `ParseCredits__Fv` | 0x002bcd70 | CONFIRMED | Parses the credits file |
| 0x00560f50 | `stl_insert_n_grow_stride_0xc` | - | - | - | Shared `vector::insert`, stride `0xc`; callers outside the singleton too |
| 0x00561480 | `stl_insert_n_stride_0x18` | - | - | - | Shared `vector::insert`, stride `0x18` |
| 0x00561780 | `stl_insert_n_shift_stride_0xc` | - | - | - | Shared `vector::insert` shift, stride `0xc` |
| 0x00561ab0 | `copy_range_stride_0x54` | - | - | - | Copies profile records; sole caller `loadsave_profile_enum_files` |
| 0x00561af0 | `fill_n_stride_0x54` | - | - | - | Fills profile records; sole caller `loadsave_profile_enum_files` |
| 0x00561b20 | `vec3_copy` | - | - | - | Copies one vector; shared by the callers outside this doc too |

## 4. What the translation unit is

One file holding **five classes** and two file-scope singletons. Its boundaries come from the
MSVC dynamic-initializer table at `0x0064615c`, read as an ordered list:

```
0x00646164: 0x00556ee0   crt_static_init_guard_00556ee0    the G_IndicatorsClass TU closes
0x00646168: 0x00556f10   loadsave_static_init              <- this TU opens
0x0064616c: 0x00556f50   car_database_static_init          the SECOND singleton
0x00646170: 0x00561b40   crt_static_init_guard_00561b40    this TU closes
0x00646174: 0x00563b00   crt_static_init_guard_00563b00    the autosalon TU closes
```

The two init groups resolve to ctor `loadsave_ctor` / dtor `loadsave_dtor` and ctor
`car_database_ctor` / dtor `car_database_close`. **Two adjacent groups at the head of a
block is the fingerprint of a TU with two file-scope objects** — that single observation
establishes the unit's identity without reading a body.

| class | PC prefix | starts | what it owns |
|---|---|---|---|
| `G_LoadSaveClass` | `loadsave_` | 57 | profiles, savegame slots, replay, objectives, freeride, settings, credits |
| `G_CarDatabase` | `car_database_` | 37 | the three `.def` tables and packed car ids — see `structs/car_database.md` |
| `G_Stream` | `stream_` | 13 | the encrypted savegame stream |
| `G_EnabledCars` | `enabled_cars_` | 1 | the four-bank car permission bitmap |
| `G_GarageClass` | `garage_` | 16 | the Free Ride garage |

The remaining starts are CRT scaffolding, five MSVC file-I/O routines, two STL helpers,
two credits free functions and `textdb_unload`.

> ⚠️ **`data_unload_def_tables` is NOT in this translation unit.** It is reached from
> `loadsave_close`'s call lockstep, but its VA `0x00592240` lies in `[0x00592050, 0x005922c0)`,
> PS2 `g_material.cpp`; see `structs/def_tables.md`. A caller tells you who calls a function,
> never where it lives.

⚠️ **`G_GarageClass` state lives inside the `G_LoadSaveClass` object on the PC.** Every
`garage_*` function here takes the same `this` as the `loadsave_*` functions (§1a).

## 5. The on-disk layout

Two file families, two four-character sigils, both written through `G_Stream`.

| path | sigil | version | content |
|---|---|---|---|
| `savegame\mafia%03d.sav` | `'Prof'` (`0x50726f66`) | 1 | one PLAYER PROFILE: name, settings, key bindings, unlock masks |
| `savegame\mafia%03d.%03d` | `'SavG'` (`0x53617647`) | 0x10 | one SAVE SLOT of that profile |
| `savegame\reply%03d.r%02d` | - | - | a race replay: setup blob + 0x4000 dwords of input |

`loadsave_profile_delete` removes the `.sav` **and all 600** slot files, which is what
fixes the per-profile slot cap at 600. Slot *names* come from the text database: slot *i*
uses string id **5000 + i**, and `loadsave_savegame_enum_files` skips any slot whose id
does not resolve — so the number of visible save slots is a localisation-table property,
not a code constant.

The replay path literal is `savegame\reply%03d.r%02d`. **"reply" is the developers' own
typo** and is present on both builds; it is not a transcription error here.

## 6. 🔑 The savegame cipher — a complete spec, and PC-only

`stream_ctor` seeds two accumulators with values that are **dates**:

```
key1 = 0x23101976     // 23.10.1976
key2 = 0x10072002     // 10.07.2002
```

Encryption walks the buffer one dword at a time (a trailing partial dword is left in
clear):

```
encrypt:  key2 += plaintext;  ciphertext = plaintext ^ key1;  key1 += key2;
decrypt:  plaintext = ciphertext ^ key1;  key2 += plaintext;  key1 += key2;
```

The two are exact inverses, which is how `stream_decrypt_dwords` is confirmed — against
its own sibling rather than against the PS2, because **the PS2 bodies of `Crypt` and
`Decrypt` are empty 8-byte stubs** (§10d). A `.sav` reader written from this section alone
will round-trip.

`stream_set_params(magic, encrypt, unread_01)` decides whether a given stream is encrypted at
all; `stream_write_at` copies to scratch before encrypting so a caller's buffer is never
mutated in place.

## 7. Free Ride

The mode word at `+0x1f0` drives everything. The script parks its world parameters in a
file-scope block first and they are published only once the scene is live:

```
loadsave_freeride_store_params(mode, flags, ratio_a, ratio_all, ratio_c)
        -> g_freeride_stored_mode / _flags / _traffic_car_ratio / _traffic_all_ratio /
           _traffic_ratio_c

loadsave_freeride_apply_script_params()          // only when mode > 2
        g_freeride_stored_traffic_car_ratio     -> g_traffic_car_elems_coef   (0x00656154)
        g_freeride_stored_traffic_all_ratio     -> g_traffic_ratio_all        (0x0064c058)
        g_freeride_stored_traffic_police_ratio  -> g_traffic_car_police_coef  (0x0065615c)
                                                   and g_traffic_ratio_police (0x0064c05c)
        this+0x98 = this+0x94                // steal mask := race-unlock mask
```

`g_traffic_car_elems_coef` is the traffic element-count coefficient, `g_traffic_car_police_coef`
weights the police spawn entries in `traffic_car_game_init`, and `g_traffic_ratio_police` has no
reader on the PC (§10j).

The last line is a real gameplay fact: **in Free Ride you may steal every car you have
unlocked for racing.**

Scene construction is `loadsave_freeride_setup_scene` (builds the `Tommy` and
`TomyhoAuto` frames and loads both models) followed, once the scene is live, by
`loadsave_freeride_setup_after`, which seats the player in the car.

## 8. Replay

A **0x4000-dword** ring of packed control words in `g_replay_frame_buffer`, with
`g_replay_frame_cursor` as the index and `g_replay_input_word` as the live word. Recording
and playback are the same function, `loadsave_replay_tick`, switched by the sign of the
mode field; the control word is masked to `0x7f`, so **at most seven control bits are
recorded**.

`loadsave_replay_load` also carries a racing setup blob, handed to
`racing_load_replay_setup` when `g_racing` is non-null — which is the only reason the
replay system touches the racing module at all. `loadsave_replay_save` is PC-only.

## 9. The key-binding table (PC-only)

60 entries × 0xC bytes at `+0x1f4`, in two bands: **0..0x20** and **0x21..0x3b**.
`loadsave_binding_find_conflict` and `loadsave_binding_clear_conflicts` both walk the band
an entry belongs to, so the split is real and not cosmetic.

The record is `S_key_binding` (§1d). It is read by
`input_set_binding` / `input_update` / `input_binding_get_label`, and the shipped label
strings decode it end to end — see `structs/input.md` §1a:

| `device_kind` | meaning |
|---|---|
| 1 | keyboard scancode |
| 2 | **mouse**, buttons and axes: `1`/`2`/`4` = the three buttons, `0x100`/`0x200`/`0x300` = axes X/Y/Z |
| 3 | **joystick**, `device_index` selecting it: `code < 0x100` an axis, `0x100`..`0x1ff` a button, `>= 0x200` a POV hat ¹ |

¹ Written only via `loadsave_binding_set_from_event`. A stale `device_index` renders as the
shipped string "Not attached joy".

The whole family is PC-only — the PS2 has no rebindable input — and the table is streamed
into the profile file as one 0x2d0 block.

### 9a. The action-label table `g_key_binding_label_string_ids` — 60 textdb string ids

A second 60-entry table, **60 dwords at `0x0064d668`**, gives each binding slot its
on-screen name. `loadsave_get_key_binding_label_id(slot)` (`0x00560120`) returns
`table[slot]` for `0 <= slot < 0x3c` and the fallback id **100** otherwise. Decoded against
`textdb.json` the entries are exactly the Controls-screen action names, and **the two bands of
§9 fall out of the data**:

| slot range | contents |
|---|---|
| 0..0x17 | on-foot actions, in PAIRS (each action has a primary and a secondary binding): 301 `Forwards`, 302 `Backwards`, 303 `Left`, 304 `Right`, 305 `Walk / Run`, 307 `Action Button`, 308 `Fire`, 309 `Crouch`, 311 `Jump`, 312 `Aiming`, 317 `Cycle Up` (x4) |
| 0x18..0x1f | the unpaired on-foot singles: 319 `Inventory`, 320 `Hide Weapon`, 321 `Drop Weapon`, 322 `Toggle Run / Walk`, 323 `Reload`, 324 `Sniper Mode`, 325 `Objectives`, 343 `City Map` |
| 0x20 | **0 — no label.** The last slot of band A, unused |
| 0x21..0x38 | in-car actions, again in pairs: 330 `Accelerate`, 331 `Brake / Reverse`, 332 `Steer Left`, 333 `Steer Right`, 334 `Handbrake`, 335 `Speed Limiter`, 337 `Change Gears Up`, 338 `Change Gears Down`, 339 `Horn`, 340 `Look Left`, 341 `Look Right`, 342 `Use Clutch` |
| 0x39..0x3b | the in-car singles: 336 `Manual/Auto Gearing`, 310 `Change camera`, 344 `Replace car (racing)` |

The split at slot `0x20`/`0x21` is the same one `loadsave_binding_find_conflict` and
`loadsave_binding_clear_conflicts` walk, so **band A is on-foot and band B is in-car** —
a key may be bound to one action on foot and a different one in a car without conflicting.

`loadsave_get_key_binding_label_id`'s two callers are GUI row constructors in the
`GM_Menu` framework cluster `[0x00566280, 0x0056adf0)`; each stores the slot index in
its own object and pushes the resolved label into element `0xc`. The function belongs to
`G_LoadSaveClass` — the class that owns the whole binding family — not to the framework
it is called from.

## 10. PC vs PS2

Everything cross-build lives here.

### 10a. The unit's identity

The PS2 file is **`g_loadsave.cpp`** (`__sinit_g_loadsave.cpp` at `0x004c8bc0`), emitted
immediately after `g_indicators.cpp`. With this TU, **PC and PS2 link order agree across
seven consecutive translation units**. PS2 code range: `0x002b61d0`–`0x002bfcd0`,
156 symbols against the PC's 176 starts.

`G_GarageClass` is the exception: on the PS2 it is a separate class at
`0x0048a5c0`–`0x0048efb0` in its own file, and the PC folded its state into the
`G_LoadSaveClass` object.

### 10b. The three locksteps that carried the identification

| PC function | PS2 method | what matched |
|---|---|---|
| `loadsave_close` | `Close__15G_LoadSaveClassFv` | five statements, same order, including the `profileIgnoreSave` guard. Named five functions and one global at once |
| `loadsave_all_default` | `AllDefault__15G_LoadSaveClassFv` | call for call, including the argument pair `RaceCarsDisable(-1)` then `RaceCarsEnable(1)` and the `SaveGameNewGame` tail call |
| `garage_game_init` / `garage_game_done` | `GameInit`/`GameDone__13G_GarageClassFv` | pinned by CALLER NAME — `game_init` and `game_done`, exactly as PS2 `C_game` calls them |

### 10c. Where symbol RANK disagrees with the call graph, the call graph wins

* `loadsave_default_key_bindings` (`0x00559630`) occupies `DefaultSettings`' rank, but its body
  is ~90 keyboard-scancode writes and it is PC-only. The real twin of `DefaultSettings` is
  `loadsave_default_settings` (`0x00559ae0`), two ranks later, sharing the `0.5f`/`1.0f`
  constants **and** the `AllDefault(0xffff)` callsite.
* Rank put `GarageShowResultsProc`/`CarManager` on `0x0055f510`/`0x0055fa90`. PS2 `CarManager`
  builds a `GM_GarageSelect` menu **and calls `DestroyCar`**: that is
  `garage_run_car_select_menu` (`0x0055f510`), which builds a menu and calls
  `garage_destroy_car` (`0x0055fa90`). `GarageShowResultsProc` is called *from* `GameDone` on
  both builds, which is `garage_show_results` (`0x0055fcb0`). Two independent edges, both
  agreeing.

### 10d. 🔴 Eight PS2 methods in this unit have no real body

Read as raw instruction words, each is 8 bytes. Seven are `jr ra; nop`, i.e. empty: `_Sesyp`,
`SetCanHaveBigWeapon` and `SetInCar` (`G_Inventory`), `Crypt` / `Decrypt` (`G_Stream`) and
`GenerateCars` / `ReleaseCars` (`G_GarageClass`). The eighth, `HaveSpaceFor`, is
`jr ra; li v0, 1`: it always answers yes, and the decompiler drops the delay-slot load and shows
`{ return; }`. On the PC the inventory and stream ones are real code: the port gutted the PC's
four-band inventory and its savegame encryption. `GenerateCars` / `ReleaseCars` are empty on both
builds: their script arms (`GARAGE_GENERATECARS` / `GARAGE_RELEASECARS`) call the shared empty
`stub_noop_cdecl_00574a40` on the PC.

⇒ **A PS2 method count is an upper bound whose entries can be hollow.** Five rows in this unit
are graded STRUCTURAL for exactly this reason (`HaveSpaceFor`, `SetCanHaveBigWeapon`, `_Sesyp`,
`Crypt`, `Decrypt`): no code diff is possible, so they rest on field, rank and arity instead.
`SetInCar` is CONFIRMED on its callers (`structs/inventory.md` §3).

### 10e. PS2 methods with no PC start

`RemoveInHand` is inlined into `human_notify_callback` as `inventory_remove_slot(0, 0)`
(`structs/inventory.md` §3).

**`AddLastStolen` and `ShowResults` ARE located.** Their PS2 bodies store 1 to `+0x7d` / `+0x7c`;
the PC twins `garage_request_add_last_stolen` / `garage_request_show_results` store 1 to `+0x1cd` /
`+0x1cc`. Both builds call them from the `GARAGE_ADDLASTSTOLEN` / `GARAGE_SHOW` arms, and both
`GameDone` bodies test the pair in one condition and open the results screen on the first byte
alone. `ShowResults` is a flag set, not a forwarder, which is why `garage_show_results` (484 B,
called from `GameDone`) is `GarageShowResultsProc`.

**Nine have no PC realisation**, measured at every PS2 caller:

| PS2 method | PS2 callers | at that position on the PC |
|---|---|---|
| `Vysum__13G_GarageClassFP5C_car` | InterCity code, `C_game::InvalidateActor`, the `C_car` destructor | `game_invalidate_actor` and `car_dtor` call nothing in this unit and touch no garage slot |
| `AutoVybuchlo__13G_GarageClassFP5C_car` | `C_car::Update` (twice) | `car_update` calls nothing in this unit and touches no garage slot |
| `CanStart__13G_GarageClassFPUc` | `C_player::AI_drive` (twice) | `player_ai_drive` starts the engine on any pedal above 0.25 with no car-mask test |
| `GetCarsByRaceMask__13G_CarDatabaseF…` | `C_racing::EndRace`, `G_GarageClass::DoFreerideMenu` | `racing_end_race` makes no car-database call; `DoFreerideMenu` is PS2-only (`structs/garage.md` §6) |
| `EnableCarsForFree__13G_CarDatabaseFUi` | `FreerideEnableCar` | `loadsave_freeride_enable_car_group` has no such call (below) |
| `GetWreckageByID__13G_CarDatabaseFi` | `C_racing::PreloadWrecks` | `racing_game_init` has no wreck preload (`structs/racing.md`) |
| `WriteAdrAtPos__8G_StreamFPvUiUi` | `SaveGameSave` (twice: the actor count and the header, written back) | `loadsave_savegame_save` closes the stream right after the actor walk, and `loadsave_savegame_load` reads `0x8c`-byte actor records until the stream ends |
| `ObjectivesComplete__15G_LoadSaveClassFUi` | the `C_program` arm of an objectives-complete command | the PC command table has only `MISSION_OBJECTIVES`, `MISSION_OBJECTIVESREMOVE` and `MISSION_OBJECTIVESCLEAR` |
| `ObjectivesGetList__15G_LoadSaveClassFv` | `GM_GameMenu::OnCreate` | `gui_game_menu_ctor` only builds its components; the PC shows objectives through `loadsave_objectives_display` |

**`FreerideEnableCar` IS located.** Its PC twin is **`loadsave_freeride_enable_car_group`** (VA
`0x0055ff70`), CONFIRMED: both bodies decode the letter `A`..`Z` / `a`..`z` to a bit index and OR
that bit into the mask the accessor below returns (PS2 `+0x2b8`, PC `+0x9c`). The PS2 body also
maps the digits `0`..`6` to bits 26..32, remaps `U` to `0`, and ORs
`G_CarDatabase::EnableCarsForFree` into `+0x2b4`; the PC has none of the three.

**`FreerideGetEnabledCars` IS located.** It is the 8-byte accessor `{ return *(this + 0x2b8); }`.
Its PC twin is **`loadsave_freeride_get_enabled_cars`** (VA `0x0055ffb0`,
`{ return *(this + 0x9c); }`), whose three call sites all load `ecx = 0x00671fb0` — this
singleton. Two one-line accessors match on shape alone, so the grade rests on the call POSITION,
which is decisive: in the Carcyclopedia's car-list builder both builds take this accessor's
result, OR in `0x2000000` and hand it to `GetCarsByCarcyclopediaMask` /
`car_database_collect_cars_by_carcyclopedia_mask` — the same edge on both builds. ⇒ `this+0x9c`
is the **Free-Ride enabled-car mask** (§1a). See `structs/carcyclopedia.md` §8e.

The whole `PSX*` family (memory-card handling, ~35 methods) is PS2-port-only.

### 10f. Divergent record strides

| record | PC | PS2 |
|---|---|---|
| car index (`carindex.def`) | **0xa4** | 0xa8 |
| carcyclopedia detail | **200 (0xc8)** | 204 (0xcc) |
| player index (`playerindex.def`) | 0x24 | 0x24 |
| profile header (`G_SaveGame_FileHead`) | 0x54 | 0x54 |

### 10g. Two PS2 symbols read against their bodies

* `GetWeaponInLeftHand__11G_Inventory` is `return gameItemTable;` on both builds: it returns the
  `predmety.def` table base without reading the inventory. It is still a `G_Inventory` method —
  both callers pass the human's inventory — so the PC name is `inventory_get_left_hand_item_def`.
* `RaceCarsAddWin__15G_LoadSaveClassFUi` unlocks whatever a car unlocks; nothing in the
  body mentions a win. PC name: `loadsave_race_cars_unlock_from_car`.

### 10h. The six class-prefix rows, body-diffed

Ordered by PC VA, PC name first.

| PC VA | PC name | PC size | PS2 method | PS2 size | grade | what the diff turned on |
|---|---|---|---|---|---|---|
| 0x00557ec0 | `loadsave_savegame_create` | 352 | `SaveGameCreate__15G_LoadSaveClassFUib` | 416 | **CONFIRMED** | the `id < 100000` split; the find-by-`entry+0x08` loop tail-calling SaveGameSelect(index); the else-arm that starts at id 1 and resets its cursor to 0xffffffff on every collision; `insert(.., 1, &zeroed)` then SaveGameSelect(count-1) |
| 0x00558180 | `loadsave_savegame_load` | 3120 | `SaveGameLoad__15G_LoadSaveClassFv` | 3672 | **CONFIRMED** | THREE OpenRead passes on both builds; `FreerideStoreB(0)`; SetParams(0x53617647,1,0); `C_game::Done` before, `C_game::Init` between; the inlined mission-name strcmp; `GM_LoadProgress` ↔ `gui_load_progress_ctor`; `Open(..,0,0xffffffff,1)` ↔ `mission_open(..,0,0xffffffff,1)`; SaveGameLoadGuid; ai_groups/ai_follow_manager Load; the tail sets the suppress flag and runs `GameAfterLoad` |
| 0x00558db0 | `loadsave_save_actor_proc` | 288 | `_SaveProc__15G_LoadSaveClassFP8G_StreamP7C_actorP20G_SaveGame_ActorHeadUiRUi` | 356 | **CONFIRMED** | statement for statement: strncpy 0x40 + NUL at [0x3f]; [0x84] = size; [0x80] = kind; the `== 9` sub-name branch with strrchr('\\') then strrchr('/') and +1; [0x88..0x8b] = -1; WriteAdr(0x8c); WantWrite; the virtual SaveGameSave slot; Write |
| 0x00558ed0 | `loadsave_savegame_save` | 1088 | `SaveGameSave__15G_LoadSaveClassFb` | 1456 | **CONFIRMED** | both 'SavG' spellings (0x53617647 and 0x47766153); the 100-default score with the `0 -> 1` fixup; the packed GetLocalTime words; ai_groups + ai_follow_manager GetSize then Save; the four freeride globals; the two-pass actor walk gated on kind 4/8 then the rest — which re-confirms `loadsave_save_actor_proc` |
| 0x00559320 | `loadsave_savegame_from_script` | 112 | `SaveGameFromScript__15G_LoadSaveClassFUibb` | 3056 | **CONFIRMED** ¹ | the one-shot suppress flag read-and-cleared in the else arm; the script id store; `SaveGameCreate(id - 5000)`; SaveGameSave; SaveGameEnumRelease; ProfileSave; and the tail `if (verbose) ConsoleAddTextByID(0x89b, 0xffffff)` — same id, same colour |
| 0x00560690 | `loadsave_execute_credits` | 512 | `ExecuteCredits__15G_LoadSaveClassFv` | 1104 | **CONFIRMED** ² | dtaOpen/dtaSeek(0,2)/`< 1` bail/LoadFont(1,0)/`size / 0x8c`/new/dtaSeek(0,0)/dtaRead/dtaClose/ParseCredits/the `+= dt * 0.03` scroll/the `array + count*0x8c - 0x88` terminator/delete/LoadFont(0,0) |

¹ **The 27× size ratio is not a mismatch** — every byte of the difference is the PS2 memory-card
user interface (the `GM_PSX_*` menu tree, `MC_GetMemoryCardsInfo`, `MC_SaveData`, `MC_Format`,
the format/overwrite question boxes), which the PC build has no counterpart for. Strip that and
the two functions are the same six calls in the same order; the shared literals (5000, 0x89b,
0xffffff) and the shared callee sequence carry the row.

² `loadsave_execute_credits` keeps its helpers on the `credits_` prefix: `credits_parse` and
`credits_render_one` twin `ParseCredits__Fv` and `RenderOneCredit__FP15G_CreditsStructf`, which
are FREE functions on the PS2, not `G_LoadSaveClass` members. Only `ExecuteCredits` is a method.

🔑 **The suppress flag is a cross-function proof.** PS2 `+0x2c0` / PC `+0x1d4` is SET at the
end of `loadsave_savegame_load` and READ-AND-CLEARED at the head of
`loadsave_savegame_from_script`, on both builds. One field, two functions, two twins — each
row corroborates the other.

### 10i. `ProfileGetList` has no PC symbol of its own — it is one owner of a three-way fold

`ProfileGetList__15G_LoadSaveClassFv` compiles to `return this`, because the profile vector
sits at **offset 0** of the singleton. On the PC that is the three-byte body
`mov eax,ecx; ret` at **`0x005575f0`**, named `folded_return_this` — and identical-COMDAT
folding put **three unrelated source functions on that one address**:

| owner | how its callsites reach it | callsites |
|---|---|---|
| **`G_LoadSaveClass::ProfileGetList`** — this class | `mov ecx, 0x671fb0`, then `[eax+4]` | 9 |
| the trivial `C_blood` constructor | `operator_new(0x24)`, `mov ecx,eax` — `structs/blood.md` | 1 |
| no class: a one-argument identity inline on a time value | `mov ecx,[esp+0x14]` in `recorder_set_clock` — its first stack argument | 1 |

Read the row as *"this class's `ProfileGetList` is realised by the code at `0x005575f0`"*, never
as *"`0x005575f0` is `ProfileGetList`"*, and expect no `PS2 twin:` stamp on the PC symbol.

Method: a branch-aware census of the receiver at each callsite. The body is *not* byte-unique —
`identity_thunk` (`0x0045aea0`) is an unfolded second copy — so uniqueness is not what proves
this; the three disagreeing receivers are.

⚠️ Consequence for §3: every PS2 method still has located PC code, but `ProfileGetList` is the
one row whose PC code is **not exclusively its own**. There is no dedicated function to find.

### 10j. The Free Ride traffic knobs

The four globals `loadsave_freeride_apply_script_params` publishes to (§7) are identified by their
PS2 names: `g_traffic_car_elems_coef` = `elems_coefff__13C_traffic_car`, `g_traffic_ratio_all` =
`ratio_all__19C_traffic_generator`, `g_traffic_car_police_coef` = `police_coefff__13C_traffic_car`
and `g_traffic_ratio_police` = `ratio_police__19C_traffic_generator`.

### 10k. Two fields the PS2 bodies explain

`ReplaySet__15G_LoadSaveClassFbb` stores 1 into its byte on every branch, as `loadsave_replay_set_mode` does with
`unread_90`, and `ReplayTick__15G_LoadSaveClassFRi` does not read it. `Close__8G_StreamFv` returns `+0x31 || +0x30`,
two error bytes; `stream_close` returns `error_flag` (`+0x2d`), which no PC code sets.

## 11. Related

`structs/car_database.md` · `structs/garage.md` · `structs/input.md` · `structs/inventory.md` · `structs/force_feedback.md` §8c · `structs/net_session.md` §1b · `structs/carcyclopedia.md` §8e ·
`structs/blood.md` §4a (the fold's second owner) · `structs/def_tables.md`.

## 12. Open items

| # | Item | State |
|---|---|---|
| 1 | In game: profile delete | delete a profile and confirm all its save slots disappear with it (§5) |
| 2 | In game: Free Ride stealing | start a Free Ride and check that every car unlocked for racing is stealable (§7) |
