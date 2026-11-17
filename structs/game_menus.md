# `game_menus` — the in-game and front-end SCREEN translation unit

**Created** 2026-08-25 17:18 +0200 · **Modified** 2026-11-16 10:49 +0100

The unit that follows `g_menu.cpp` holds the **screens** the GUI framework exists to run: the main menu, the in-game menu and its racing variant, options and controls, the three player-profile screens, load game, free ride, the three modal boxes, game over, and the loading-progress screen.

| | |
|---|---|
| PC prefix | `gui_options_audio_video_`, `gui_main_menu_exit_box_`, `gui_game_menu_exit_box_`, `gui_free_ride_location_`, `gui_options_controls_`, `gui_racing_game_menu_`, `gui_combo_dropdown_`, `gui_profile_select_`, `gui_profile_create_`, `gui_profile_delete_`, `gui_free_ride_mode_`, `gui_load_progress_`, `gui_notice_panel_`, `gui_question_box_`, `gui_key_conflict_`, `gui_session_list_`, `gui_message_box_`, `gui_key_capture_`, `gui_action_menu_`, `gui_main_menu_`, `gui_game_menu_`, `gui_inventory_`, `gui_load_game_`, `gui_free_ride_`, `gui_game_over_`, `gui_gun_shop_`, `gui_options_` |
| PC kind (`+0x10`) | - |
| PC vtable | 27 tables from `g_gui_menu_vt_combo_dropdown_screen` @ `0x0063ddfc` to `g_gui_menu_vt_free_ride_mode` @ `0x0063e0cc`, 6 slots each (§4b) |
| PC base class | `GM_Menu` (`structs/gui_screen.md`) |
| PC instance size | per screen, §1 |
| PC constructor | per screen, §3a, §4a |
| PC translation unit | `[0x00575c20, 0x0057d6e0)` guard to guard; `gui_garage_*` hosted in `structs/garage.md` |
| PS2 class | 23 `GM_*` classes (§13), 147 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Every screen inherits the `GM_Menu` layout, `0x48` bytes (`structs/gui_screen.md` §1). These twelve add no field, `operator_new(0x48)` at every caller: the notice panel, the question box, the message box, both exit boxes, the main menu, the game menu, Audio & Video, Options, the profile delete screen, game over, the racing game menu. The fifteen others follow, each size read at the `operator_new` before the constructor.

### 1a. the combo drop-down — `gui_combo_dropdown_`

```c
typedef struct S_gui_combo_dropdown {                  // 0x50 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t * out_picked_id;          // [H] where the picked row id goes; the ctor compares it with each row to find the current one
    /* +0x4c */ S_gui_option_list * options;       // [H] the rows to list, `structs/multiplayer.md` §1d; cleared when empty, which every tick path tests
} S_gui_combo_dropdown;
```

### 1b. the controls page — `gui_options_controls_`

```c
typedef struct S_gui_options_controls {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t device_page;              // [H] the shown input device: 0 keyboard, 1 mouse, other joystick; picks the design gate 1 / 2 / 4
} S_gui_options_controls;
```

### 1c. key capture — `gui_key_capture_`

```c
typedef struct S_gui_key_capture {                  // 0x50 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t binding_slot;             // [H] the index into `g_key_binding_table`
    /* +0x4c */ uint8_t axis_only;                 // [H] 1 on the two Aiming rows: the capture accepts analog axes only (§2c)
} S_gui_key_capture;
```

### 1d. the key conflict box — `gui_key_conflict_`

```c
typedef struct S_gui_key_conflict {                  // 0x5c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t binding_slot;             // [H] the index into `g_key_binding_table`
    /* +0x4c */ uint8_t axis_only;                 // [H] the capture screen's flag; stored, no reader (class census)
    /* +0x50 */ uint32_t captured_mode;            // [H] the binding's `mode`; the tick rewrites it for the paired axis codes
    /* +0x54 */ uint32_t captured_device_kind;     // [H] the binding's `device_kind`
    /* +0x58 */ uint32_t captured_device_index;    // [H] the binding's `device_index`
} S_gui_key_conflict;
```

### 1e. the inventory screen — `gui_inventory_`

```c
typedef struct S_gui_inventory {                  // 0x850 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ char row_labels[32][0x40];         // [H] row label strings; a permanent row of id `0x100 + i` uses entry i
    /* +0x848 */ G_Inventory * inventory;           // [H] the `G_Inventory` shown: `human+0x480`
    /* +0x84c */ void * removed_items;              // [H] the caller's stack vector that receives removed items; second argument of `inventory_remove_item_by_id` and third of `inventory_select_slot`
} S_gui_inventory;
```

**`row_labels` has 32 entries because the space between `+0x48` and `+0x848` holds 32 of `0x40` bytes;** the code uses entries 0 and 2..8 only (ids `0x100`, `0x102`..`0x108`), since an item row's text comes from `textdb_get_string_by_id`. The object's size is the bound, and no constant 32 appears. `+0x48` and `+0xc8` are the entries the ctor passes for Hands and the first weapon.

### 1f. the action menu — `gui_action_menu_`

```c
typedef struct S_gui_action_menu {                  // 0x106c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ char row_labels[64][0x40];         // [H] the `%d) %s` string per option row, numbered from 1
    /* +0x1048 */ G_Inventory * inventory;           // [H] the `G_Inventory`: `human+0x480`
    /* +0x104c */ void * npcs;                       // [H] the caller's NPC vector, walked for their inventories
    /* +0x1050 */ void * world_items;                // [H] the caller's world-item vector
    /* +0x1054 */ void * taken_world_items;          // [H] the caller's list of taken world items
    /* +0x1058 */ void * taken_npc_items;            // [H] the caller's list of items taken from an NPC
    /* +0x105c */ uint8_t options_allocator;         // [H] the options vector's empty allocator byte; no reader
    /* +0x1060 */ void ** options_begin;             // [H] the gathered options; a row indexes it by id minus `0x100`
    /* +0x1064 */ void ** options_end;               // [H] one past the last option; one option left makes the tick auto-take it
    /* +0x1068 */ void ** options_capacity_end;      // [H] one past the allocation
} S_gui_action_menu;
```

**`row_labels` has 64 entries because `+0x48..+0x1047` holds 64 of `0x40` bytes.** The ctor labels entry 0 before its option loop and entries 1..62 inside it: the loop ends at the end of the options or after entry 62 (`cmp ebx, 0x3e` at `0x0057938e`), so at most 63 option rows get a label, and the Exit row takes the next entry, 63 at most. The 64 entries are exactly enough; options past the 63rd get no row. `+0x105c..+0x1068` is one `std::vector` with its allocator byte, grown by `std_vector4_insert_n` through `lea edi, [esi + 0x105c]`.

### 1g. the profile select screen — `gui_profile_select_`

```c
typedef struct S_gui_profile_select {                  // 0x6c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t pending_action;           // [H] 0 none, 1 open the name box, 2 open the delete screen, 3 raise the exists box; the next tick acts on it
    /* +0x4c */ char new_profile_name[0x20];       // [H] seeded with textdb `0x84` "Default"; the name box edits it, `loadsave_profile_create` reads it
} S_gui_profile_select;
```

`new_profile_name` holds `0x20` bytes because the object ends at `0x6c`; the profile record's name at `+0x10` of `0x54` is copied from it by `loadsave_profile_create`.

### 1h. the name box — `gui_profile_create_`

```c
typedef struct S_gui_profile_create {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ char * name;                       // [H] the caller's `new_profile_name`
} S_gui_profile_create;
```

### 1i. load game — `gui_load_game_`

```c
typedef struct S_gui_load_game {                  // 0x28c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ char total_text[0x100];            // [H] "Total Saved Games: %d", component 10
    /* +0x148 */ char details_text[0x100];          // [H] textdb `0xaf`, health, save time and play time, component `0x3f2`
    /* +0x248 */ S_savegame_head head;              // [H] the slot's `0x18`-byte header, `structs/loadsave.md` §1
    /* +0x260 */ S_savegame_info info;              // [H] the slot's `0x20`-byte info block, `structs/loadsave.md` §1g
    /* +0x280 */ uint32_t preview_delay_frames;     // [H] 3 after each load; the tick waits this many frames on a newly focused row
    /* +0x284 */ uint32_t focused_row;              // [H] the row last previewed; -1 from the ctor
    /* +0x288 */ uint32_t save_count;               // [H] the file vector's count at construction; undoes the row reversal
} S_gui_load_game;
```

`total_text` and `details_text` are `0x100` bytes each, the space up to the next field. `head` and `info` are the two out-records of `loadsave_savegame_load_header`, `0x18` and `0x20` bytes. `gui_load_game_tick` passes `info.slot_number` to `loadsave_get_screenshot_idx` for the `shot%03d.bmp` name and formats `save_day`, `save_month`, `save_year`, `save_hour`, `save_minute`, `health_percent` and `play_time_ms` into textdb `0xaf` (divided by 1000 and 60 into hours, minutes, seconds); it reads no other byte of the block (§9).

### 1j. free ride — `gui_free_ride_`

```c
typedef struct S_gui_free_ride {                  // 0x50 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t mode;                     // [H] the free-ride mode, `0x1c`..`0x20`; seeded `0x1c`, the mode picker's result lands here
    /* +0x4c */ uint32_t chosen_car;               // [H] the packed car id; seeded `0x17`, written by the garage chain through its out pointer
} S_gui_free_ride;
```

### 1k. the district picker — `gui_free_ride_location_`

```c
typedef struct S_gui_free_ride_location {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t mode;                     // [H] the free-ride mode; above `0x1e` the ctor relabels the rows for the countryside
} S_gui_free_ride_location;
```

### 1l. the session browser — `gui_session_list_`

```c
typedef struct S_gui_session_list {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t refresh_ms;               // [H] countdown from the ctor's argument, 2000; at the end the tick closes with `0x41`
} S_gui_session_list;
```

### 1m. the gun shop — `gui_gun_shop_`

```c
typedef struct S_gui_gun_shop {                  // 0xc54 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ char row_labels[32][0x40];         // [H] row label strings; a carried slot of id `0x100 + i` uses entry i
    /* +0x848 */ G_Inventory * inventory;           // [H] the `G_Inventory`: the script's actor `+0x480`
    /* +0x84c */ uint32_t unused_84c;               // [U] no access; no reader (class census)
    /* +0x850 */ uint32_t catalogue[256];           // [H] the ids of `g_game_item_table` whose flags lack `0x800000`, ascending from 2
    /* +0xc50 */ uint32_t catalogue_count;          // [H] the entries filled
} S_gui_gun_shop;
```

**`row_labels`** has the same 32-entry shape as the inventory's; entries 0 and 2..7 are used. **`catalogue` holds 256 dwords because `+0x850..+0xc4f` is that space;** the ctor's loop is bounded by `g_num_game_items`, not by 256.

### 1n. loading progress — `gui_load_progress_`

```c
typedef struct S_gui_load_progress {                  // 0x50 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ float mission_open_share;          // [H] the bar's split, the ctor's argument: 0 or `0x3f333333` = 0.7 at its two callers
    /* +0x4c */ uint32_t unused_4c;                 // [U] no access; no reader (class census)
} S_gui_load_progress;
```

The callers allocate `0x50` bytes; no function touches `unused_4c`.

### 1o. the scenario picker — `gui_free_ride_mode_`

```c
typedef struct S_gui_free_ride_mode {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t focused_row;              // [H] the row whose preview is shown; seeded 401
} S_gui_free_ride_mode;
```

## 2. Field evidence

Cells come from a census of each screen's `this` accesses in `.text` plus a read of the functions named; a write through an out pointer is listed at the field it lands in.

### 2a. `S_gui_combo_dropdown`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | out_picked_id | H | `gui_combo_dropdown_screen_ctor` | `gui_combo_dropdown_screen_ctor`, `gui_combo_dropdown_tick` | - |
| +0x4c | options | H | `gui_combo_dropdown_screen_ctor` | `gui_combo_dropdown_screen_ctor`, `gui_combo_dropdown_tick` | - |

### 2b. `S_gui_options_controls`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | device_page | H | `gui_options_controls_ctor`, `gui_options_controls_tick` | `gui_options_controls_on_exchange`, `gui_options_controls_tick` | - |

### 2c. `S_gui_key_capture`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | binding_slot | H | `gui_key_capture_ctor` | `gui_key_capture_tick` | - |
| +0x4c | axis_only | H | `gui_key_capture_ctor` | `gui_key_capture_tick` | the ctor stores the `'vidc'` row's `record_color != 0`; the tick turns it into the source mask of `input_capture_binding` (a set bit skips a source: `0x1` mouse axes, `0x2` joystick axes, `0x4` mouse buttons, `0x8` joystick buttons, `0x10` POV hats, `0x20` keyboard): 0 gives `0x1` (0 for slots `0x12` / `0x13`), 1 gives `0x3d` (`0x3c` for those slots), so 1 leaves analog axes only. In the shipped design 10 exactly two of the 59 `'vidc'` records carry colour 1: ids 212 and 312, slots `0x12` / `0x13`, the Aiming pair |

### 2d. `S_gui_key_conflict`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | binding_slot | H | `gui_key_conflict_ctor` | `gui_key_conflict_tick` | - |
| +0x4c | axis_only | H | `gui_key_conflict_ctor` | - | the capture screen's `axis_only`; no reader (class census) |
| +0x50 | captured_mode | H | `gui_key_conflict_ctor`, `gui_key_conflict_tick` | `gui_key_conflict_tick`, `loadsave_binding_clear_conflicts` | - |
| +0x54 | captured_device_kind | H | `gui_key_conflict_ctor` | `gui_key_conflict_tick`, `loadsave_binding_clear_conflicts` | - |
| +0x58 | captured_device_index | H | `gui_key_conflict_ctor` | `gui_key_conflict_tick`, `loadsave_binding_clear_conflicts` | - |

### 2e. `S_gui_inventory`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | row_labels | H | `gui_inventory_format_slot_row` | `gui_menu_set_component_text_ptr` | - |
| +0x848 | inventory | H | `gui_inventory_ctor` | `gui_inventory_ctor`, `gui_inventory_on_close_child`, `gui_inventory_tick` | - |
| +0x84c | removed_items | H | `gui_inventory_ctor` | `gui_inventory_tick` | - |

### 2f. `S_gui_action_menu`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | row_labels | H | `gui_action_menu_ctor` | `gui_menu_set_component_text_ptr` | - |
| +0x1048 | inventory | H | `gui_action_menu_ctor` | `gui_action_menu_on_close_child`, `gui_action_menu_tick` | - |
| +0x104c | npcs | H | `gui_action_menu_ctor` | `gui_action_menu_ctor`, `gui_action_menu_pick_up` | - |
| +0x1050 | world_items | H | `gui_action_menu_ctor` | `gui_action_menu_ctor`, `gui_action_menu_pick_up` | - |
| +0x1054 | taken_world_items | H | `gui_action_menu_ctor` | `gui_action_menu_pick_up`, `gui_action_menu_tick` | - |
| +0x1058 | taken_npc_items | H | `gui_action_menu_ctor` | `gui_action_menu_pick_up` | - |
| +0x105c | options_allocator | H | `gui_action_menu_ctor` | - | stored from an argument's low byte, as every project vector's allocator byte; no reader |
| +0x1060 | options_begin | H | `gui_action_menu_ctor`, `gui_action_menu_dtor` | `gui_action_menu_ctor`, `gui_action_menu_dtor`, `gui_action_menu_on_close_child`, `gui_action_menu_pick_up`, `gui_action_menu_tick` | - |
| +0x1064 | options_end | H | `gui_action_menu_ctor`, `gui_action_menu_dtor` | `gui_action_menu_ctor`, `gui_action_menu_tick` | - |
| +0x1068 | options_capacity_end | H | `gui_action_menu_ctor` (by address, `std_vector4_insert_n`), `gui_action_menu_dtor` | `gui_action_menu_ctor` (by address, `std_vector4_insert_n`) | - |

### 2g. `S_gui_profile_select`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | pending_action | H | `gui_profile_select_ctor`, `gui_profile_select_on_close_child`, `gui_profile_select_tick` | `gui_profile_select_tick` | - |
| +0x4c | new_profile_name | H | `gui_profile_select_build_rows` | `gui_profile_select_on_close_child`, `gui_profile_create_ctor` | - |

### 2h. `S_gui_profile_create`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | name | H | `gui_profile_create_ctor` | `gui_profile_create_ctor`, `gui_profile_create_tick` | - |

### 2i. `S_gui_load_game`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | total_text | H | `gui_load_game_ctor` | `gui_menu_set_component_text_ptr` | - |
| +0x148 | details_text | H | `gui_load_game_tick` | `gui_menu_set_component_text_ptr` | - |
| +0x248 | head | H | `loadsave_savegame_load_header` | `gui_load_game_tick` | - |
| +0x260 | info | H | `loadsave_savegame_load_header` | `gui_load_game_tick` | `structs/loadsave.md` §2g |
| +0x280 | preview_delay_frames | H | `gui_load_game_ctor`, `gui_load_game_tick` | `gui_load_game_tick` | - |
| +0x284 | focused_row | H | `gui_load_game_ctor`, `gui_load_game_tick` | `gui_load_game_tick` | - |
| +0x288 | save_count | H | `gui_load_game_ctor` | `gui_load_game_tick` | - |

### 2j. `S_gui_free_ride`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | mode | H | `gui_free_ride_ctor`, `gui_free_ride_on_close_child` | `gui_free_ride_on_close_child`, `gui_free_ride_tick` | - |
| +0x4c | chosen_car | H | `gui_free_ride_ctor`, `gui_race_car_maker_on_close_child` | `gui_free_ride_on_close_child`, `gui_free_ride_tick` | - |

### 2k. `S_gui_free_ride_location`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | mode | H | `gui_free_ride_location_ctor` | `gui_free_ride_location_ctor`, `gui_free_ride_location_tick` | - |

### 2l. `S_gui_session_list`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | refresh_ms | H | `gui_session_list_ctor`, `gui_session_list_tick` | `gui_session_list_tick` | - |

### 2m. `S_gui_gun_shop`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | row_labels | H | `gui_gun_shop_format_slot_row` | `gui_menu_set_component_text_ptr` | - |
| +0x848 | inventory | H | `gui_gun_shop_ctor` | `gui_gun_shop_refresh_slots`, `gui_gun_shop_tick` | - |
| +0x84c | unused_84c | U | - | - | no access; no reader (class census) |
| +0x850 | catalogue | H | `gui_gun_shop_ctor` | `gui_gun_shop_ctor`, `gui_gun_shop_tick` | - |
| +0xc50 | catalogue_count | H | `gui_gun_shop_ctor` | `gui_gun_shop_ctor` | - |

### 2n. `S_gui_load_progress`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | mission_open_share | H | `gui_load_progress_ctor` | `gui_load_progress_set_progress_game_init`, `gui_load_progress_set_progress_mission_open` | - |
| +0x4c | unused_4c | U | - | - | no access; no reader (class census) |

### 2o. `S_gui_free_ride_mode`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | focused_row | H | `gui_free_ride_mode_ctor`, `gui_free_ride_mode_tick` | `gui_free_ride_mode_tick` | - |

## 3. Method family

47 of 147 PS2 methods of these classes have a row here, the other 100 are in §13e. The garage screens' rows are in `structs/garage.md`.

### 3a. Construction & destruction

#### The unit

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00575c20 | `crt_static_init_guard_00575c20` | the translation unit | - | - | - | `.CRT$XCU` guard of this unit's static destructor |
| 0x00575c40 | `crt_atexit_register_00575c40` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x00578790 | `gui_menu_scalar_deleting_dtor_common` | shared | - | - | - | slot 0 of every screen without a destructor of its own, in this unit and the two other screen units: calls `gui_menu_dtor`, frees on request |
| 0x005787b0 | `thunk_gui_menu_dtor` | shared | - | - | - | jumps to `gui_menu_dtor` |

#### `GM_ComboMenu` (`gui_combo_dropdown_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00575c50 | `gui_combo_dropdown_screen_ctor` | instance | `__ct__12GM_ComboMenuFP12GM_Component` | 0x002cc6c0 | STRUCTURAL | Design 0x13 |

#### the notice panel (`gui_notice_panel_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00575f50 | `gui_notice_panel_ctor` | instance | - | - | - | Design 0 |
| 0x005761d0 | `gui_notice_panel_ctor_from_strings` | instance | - | - | - | Design 0, pointer form |

#### `GM_QuestionBox` (`gui_question_box_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005764c0 | `gui_question_box_ctor` | instance | `__ct__14GM_QuestionBoxFUiUib` | 0x002d0c80 | CONFIRMED | Design 1 |

#### the message box (`gui_message_box_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576620 | `gui_message_box_ctor` | instance | - | - | - | design 2: one message and OK |

#### `GM_MainMenuExit` (`gui_main_menu_exit_box_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576810 | `gui_main_menu_exit_box_ctor` | instance | `OnCreate__15GM_MainMenuExitFv` | 0x002d41b0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063de5c`, sets the modal bit `0x80000`, builds design 0x36 (a frame plus items 138 "Yes" and 139 "No") and sets component `0x82` to textdb `0x77` "Quit Game?" |

#### `GM_MainMenu` (`gui_main_menu_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576920 | `gui_main_menu_ctor` | instance | `OnCreate__11GM_MainMenuFv` | 0x002cf340 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063de74`, sets the modal bit `0x80000`, and builds design 6 with gate 2 - the variant of the shipped `tables\menu.def` group that carries the extra "Free Ride Extreme" row (§5) |

#### `GM_GameMenuExit` (`gui_game_menu_exit_box_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576cc0 | `gui_game_menu_exit_box_ctor` | instance | `OnCreate__15GM_GameMenuExitFv` | 0x002d4090 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063de8c`, sets the modal bit `0x80000`, builds design 0x36 and sets component `0x82` to textdb `0x95` "Exit Game?" |

#### `GM_GameMenu` (`gui_game_menu_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576dd0 | `gui_game_menu_ctor` | instance | `OnCreate__11GM_GameMenuFv` | 0x002cfac0 | STRUCTURAL | chains `gui_menu_ctor`, installs vtable `0x0063dea4`, and builds design 7 |

#### Audio & Video (`gui_options_audio_video_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00576fe0 | `gui_options_audio_video_ctor` | instance | - | - | - | runs the `GM_Menu` base ctor, installs vtable `0x0063dec4`, builds shipped design 8, clears flag `0x10000`, then seeds every control from the display/audio settings block at `g_act_mission+0x90` (§7a) |

#### `GM_OptionsControls` (`gui_options_controls_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00577b10 | `gui_options_controls_ctor` | instance | `OnCreate__18GM_OptionsControlsFv` | 0x002d0320 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063def4`, builds design 10 with gate 1 (the keyboard page - the shipped group has 149 records across three device gates), zeroes the device selector at `+0x48` and clears flag bit `0x10000` |

#### `GM_Options` (`gui_options_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00577990 | `gui_options_ctor` | instance | `OnCreate__10GM_OptionsFv` | 0x002cff90 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063dedc`, builds design 9, CLEARS flag bit `0x10000` (the recursive parent-redraw bit) and deletes components 1000 and `0x3e9`, which the shipped group does not use on the PC |

#### key capture (`gui_key_capture_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005782e0 | `gui_key_capture_ctor` | instance | - | - | - | Design 0xb |

#### the key conflict box (`gui_key_conflict_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005786e0 | `gui_key_conflict_ctor` | instance | - | - | - | Design 0x35 |

#### the inventory screen (`gui_inventory_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00578a30 | `gui_inventory_ctor` | instance | - | - | - | Design 0xf, gate 1/2 = on foot / in car |

#### `GM_ItemPickUp` (`gui_action_menu_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00579020 | `gui_action_menu_ctor` | instance | `OnCreate__13GM_ItemPickUpFv` | 0x002d1c60 | CONFIRMED | - |
| 0x00579480 | `gui_action_menu_scalar_deleting_dtor` | instance | - | - | - | the MSVC scalar deleting destructor |
| 0x005794a0 | `gui_action_menu_dtor` | instance | `__dt__13GM_ItemPickUpFv` | 0x002d3b70 | CONFIRMED | frees the option vector's storage at `+0x1060`, zeroes the begin/end/cap triple `+0x1060`/`+0x1064`/`+0x1068`, then tail-calls `gui_menu_dtor` |

#### `GM_ProfileSelect` (`gui_profile_select_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00579900 | `gui_profile_select_ctor` | instance | `OnCreate__16GM_ProfileSelectFv` | 0x002df500 | STRUCTURAL | chains `gui_menu_ctor`, installs vtable `0x0063df6c`, builds design 4, sets the modal bit `0x80000`, points the menu camera at `0xf1b31`, delegates the row building to `gui_profile_select_build_rows`, and sets `pending_action` to 1 when no profile exists, else 0 (§11) |

#### `GM_ProfileCreate` (`gui_profile_create_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00579fb0 | `gui_profile_create_ctor` | instance | `OnCreate__16GM_ProfileCreateFv` | 0x002e16c0 | CONFIRMED | Design 3 |
| 0x0057a070 | `gui_profile_create_scalar_deleting_dtor` | instance | `__dt__16GM_ProfileCreateFv` | 0x002e28a0 | STRUCTURAL | calls `gui_profile_create_dtor` and then, when bit 0 of the MSVC delete flag is set, `operator_delete_wrapper` on the object |
| 0x0057a090 | `gui_profile_create_dtor` | instance | `__dt__16GM_ProfileCreateFv` | 0x002e28a0 | STRUCTURAL | re-installs the class vtable (the MSVC destructor prologue), calls `g_engine` vtable `+0x88` with argument 1 - restoring the normal input mode that the constructor switched to 2 - and chains `gui_menu_dtor` |

#### `GM_ProfileDelete` (`gui_profile_delete_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057a1f0 | `gui_profile_delete_ctor` | instance | `OnCreate__16GM_ProfileDeleteFv` | 0x002e21f0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063df9c`, builds design 5, clears flag bit `0x10000`, and clones one `'item'` row per profile from the same 0x54-byte list the select screen uses, ids 100, 101, ..., then grows and re-centres the frame (§11) |

#### `GM_LoadGame` (`gui_load_game_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057a4a0 | `gui_load_game_ctor` | instance | `OnCreate__11GM_LoadGameFv` | 0x002d1080 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063dfb4`, builds design 0xc, then enumerates the save files through `loadsave_savegame_enum_files()` - a vector of 0x18-byte records - formats textdb `0xac` "Total Saved Games: %d" into `total_text` for component 10 and stores the count in `save_count` (§9) |

#### the district picker (`gui_free_ride_location_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057aff0 | `gui_free_ride_location_ctor` | instance | - | - | - | Design 0x34 |

#### the scenario picker (`gui_free_ride_mode_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057d4b0 | `gui_free_ride_mode_ctor` | instance | - | - | - | runs the base ctor, installs vtable `0x0063e0cc`, builds shipped design 0x33, seeds the focused row to 401 at `+0x48` and the preview to `freeride0.bmp`, then gates the rows by `g_freeride_unlock_level` (§7f) |

#### `GM_FreeRide` (`gui_free_ride_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057aaf0 | `gui_free_ride_ctor` | instance | `OnCreate__11GM_FreeRideFv` | 0x002d0870 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063dfcc`, builds design 0xe and clears flag bit `0x10000` |

#### the session browser (`gui_session_list_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057b150 | `gui_session_list_ctor` | instance | - | - | - | Design 0x2f |

#### `GM_GunShop` (`gui_gun_shop_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057b3a0 | `gui_gun_shop_ctor` | instance | `OnCreate__10GM_GunShopFv` | 0x002d29e0 | CONFIRMED | 1 -> 1 |

#### `GM_GameFailed` (`gui_game_over_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057c8e0 | `gui_game_over_ctor` | instance | `OnCreate__13GM_GameFailedFv` | 0x002d1ab0 | CONFIRMED | - |

#### `GM_GameMenuRacing` (`gui_racing_game_menu_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057cab0 | `gui_racing_game_menu_ctor` | instance | `OnCreate__17GM_GameMenuRacingFv` | 0x002dee40 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063e08c` and builds design 0x2e - "Game menu:" with rows 1001 Exit Game, 1002 Options, 1003 Skip Race, 1004 Resume Game |

#### `GM_LoadProgress` (`gui_load_progress_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057cc90 | `gui_load_progress_ctor` | instance | `OnCreate__15GM_LoadProgressFv` | 0x002d2cc0 | CONFIRMED | - |

### 3b. Accessors

No accessor: every field is reached from a constructor, a tick or a child handler.

### 3c. Behaviour

#### The unit

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00574a40 | `stub_noop_cdecl_00574a40` | - | - | - | slot 1 of the audio-video vtable: does nothing, so the screen does not rebuild on exchange |
| 0x005771a0 | `thunk_gui_menu_on_close_child` | - | - | - | slot 3 of the audio-video, gun-shop and two garage vtables, and of the colour screen in `structs/racing_menus.md`: jumps to `gui_menu_on_close_child` |

#### `GM_ComboMenu` (`gui_combo_dropdown_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00575ea0 | `gui_combo_dropdown_tick` | `OnClick__12GM_ComboMenuFUi` | 0x002cce30 | CONFIRMED | slot 5: a picked row is written through `out_picked_id`; a click outside component 2 closes with -1 |

#### the notice panel (`gui_notice_panel_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056e3f0 | `gui_notice_panel_present_once` | - | - | - | drives one panel for exactly one frame; the address lies in `structs/gui_screen.md`'s unit |
| 0x00576450 | `gui_notice_panel_tick` | - | - | - | slot 5: any key or click closes |

#### `GM_QuestionBox` (`gui_question_box_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00576560 | `gui_question_box_tick` | `OnClick__14GM_QuestionBoxFUi` | 0x002d0ff0 | CONFIRMED | slot 5: Yes `0xb`, No or ESC `0xc` |

#### the message box (`gui_message_box_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005767a0 | `gui_message_box_tick` | - | - | - | slot 5: any click, ESC or ENTER closes |

#### `GM_MainMenuExit` (`gui_main_menu_exit_box_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00576890 | `gui_main_menu_exit_box_tick` | `OnClick__15GM_MainMenuExitFUi` | 0x002cf2f0 | CONFIRMED | slot 5: "Quit Game?", result `0x18` |

#### `GM_MainMenu` (`gui_main_menu_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00576a20 | `gui_main_menu_tick` | `OnClick__11GM_MainMenuFUi` | 0x002cf490 | CONFIRMED | slot 5: 12 arms over ids `0xf1b95`..`0xf1ba1` |

#### `GM_GameMenuExit` (`gui_game_menu_exit_box_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00576d40 | `gui_game_menu_exit_box_tick` | `OnClick__15GM_GameMenuExitFUi` | 0x002cfa40 | CONFIRMED | slot 5: "Exit Game?", result `0x21` |

#### `GM_GameMenu` (`gui_game_menu_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00576e30 | `gui_game_menu_on_exchange` | - | - | - | slot 1: rebuilds design 7 |
| 0x00576e60 | `gui_game_menu_tick` | `OnClick__11GM_GameMenuFUi` | 0x002cfbd0 | CONFIRMED | slot 5: ids `0x66`, `0x67`, `0x6c`, `0x6f` |

#### Audio & Video (`gui_options_audio_video_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005771b0 | `gui_options_audio_video_tick` | - | - | - | slot 5: applies whichever control moved |

#### `GM_OptionsControls` (`gui_options_controls_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00577b90 | `gui_options_controls_on_exchange` | `SetUpTexts__18GM_OptionsControlsFv` | 0x002d0480 | STRUCTURAL | slot 1: gate 1 / 2 / 4 by device page, re-seeds every value |
| 0x00577d20 | `gui_options_controls_close` | - | - | - | slot 4: applies the bindings, reloads the speedo texture |
| 0x00577d50 | `gui_options_controls_tick` | `OnClick__18GM_OptionsControlsFUi` | 0x002d0390 | STRUCTURAL | slot 5: every slider and toggle; `0x35` restores defaults |

#### `GM_Options` (`gui_options_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00577a20 | `gui_options_tick` | `OnClick__10GM_OptionsFUi` | 0x002d00b0 | CONFIRMED | slot 5: rows 100 Controls, 101 Audio & Video, 400 Back |

#### key capture (`gui_key_capture_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00578370 | `gui_key_capture_on_close_child` | - | - | - | slot 3: closes itself once the conflict child returns |
| 0x005783a0 | `gui_key_capture_tick` | - | - | - | slot 5: captures the event, commits or opens the conflict box |

#### the key conflict box (`gui_key_conflict_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005787c0 | `gui_key_conflict_tick` | - | - | - | slot 5: Yes commits the captured triple, No or ESC abandons |

#### the inventory screen (`gui_inventory_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00578d10 | `gui_inventory_on_close_child` | - | - | - | slot 3: takes an item a child handed back |
| 0x00578d90 | `gui_inventory_tick` | - | - | - | slot 5: select, drop or leave |

#### `GM_ItemPickUp` (`gui_action_menu_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005794d0 | `gui_action_menu_pick_up` | `PickUp__13GM_ItemPickUpFUib` | 0x002d26c0 | CONFIRMED | records a take, or strips the item from every NPC |
| 0x00579590 | `gui_action_menu_on_close_child` | - | - | - | slot 3: takes the item a child confirmed |
| 0x00579610 | `gui_action_menu_tick` | `OnKey__13GM_ItemPickUpFUcUi` | 0x002d2430 | CONFIRMED | slot 5: number keys 2..10 pick rows `0x100`..`0x108`; one option is auto-taken |

#### `GM_ProfileSelect` (`gui_profile_select_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005799d0 | `gui_profile_select_on_exchange` | - | - | - | slot 1: rebuilds design 4 |
| 0x00579a00 | `gui_profile_select_build_rows` | `OnCreate__16GM_ProfileSelectFv` | 0x002df500 | STRUCTURAL | one `item` row per profile, then the layout |
| 0x00579bf0 | `gui_profile_select_on_close_child` | `OnCloseChild__16GM_ProfileSelectFP7GM_MenuUiUi` | 0x002dfcb0 | STRUCTURAL | slot 3: creates the profile |
| 0x00579cc0 | `gui_profile_select_tick` | `OnClick__16GM_ProfileSelectFUi` | 0x002e14a0 | CONFIRMED | slot 5: the pending action, then the picked row |

#### `GM_ProfileCreate` (`gui_profile_create_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057a0f0 | `gui_profile_create_tick` | `OnClick__16GM_ProfileCreateFUi` | 0x002e1a40 | CONFIRMED | slot 5: OK or ENTER `0xd`, Cancel or ESC `0xe` |

#### `GM_ProfileDelete` (`gui_profile_delete_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057a370 | `gui_profile_delete_on_close_child` | `OnCloseChild__16GM_ProfileDeleteFP7GM_MenuUiUi` | 0x002e2680 | STRUCTURAL | slot 3: performs the delete |
| 0x0057a3c0 | `gui_profile_delete_tick` | `OnClick__16GM_ProfileDeleteFUi` | 0x002e2730 | CONFIRMED | slot 5: opens the confirm box per row |

#### `GM_LoadGame` (`gui_load_game_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057a670 | `gui_load_game_tick` | `OnClick__11GM_LoadGameFUi` | 0x002d19f0 | CONFIRMED | slot 5: scrolling, the thumbnail preview, the load |

#### the district picker (`gui_free_ride_location_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057b0b0 | `gui_free_ride_location_tick` | - | - | - | slot 5: ids 101..107 to `loadsave_freeride_store_state(id-100)` |

#### the scenario picker (`gui_free_ride_mode_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057d580 | `gui_free_ride_mode_tick` | - | - | - | slot 5: rows 401..405 to modes `0x1c`, `0x1e`, `0x20`, `0x1d`, `0x1f` |

#### `GM_FreeRide` (`gui_free_ride_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057ac40 | `gui_free_ride_on_close_child` | `OnCloseChild__11GM_FreeRideFP7GM_MenuUiUi` | 0x002d0a00 | CONFIRMED | slot 3: car name, mode label and `freerideN.bmp` |
| 0x0057ad80 | `gui_free_ride_tick` | `OnClick__11GM_FreeRideFUi` | 0x002d0aa0 | CONFIRMED | slot 5: ids 100, `0x65`, 200, `0xc9` |

#### the session browser (`gui_session_list_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057b2e0 | `gui_session_list_tick` | - | - | - | slot 5: pick, cancel, or time out and refresh |

#### `GM_GunShop` (`gui_gun_shop_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057b600 | `gui_gun_shop_refresh_slots` | `RefreshInventory__10GM_GunShopFv` | 0x002d2b70 | STRUCTURAL | re-renders the player's slots after every trade |
| 0x0057b700 | `gui_gun_shop_tick` | `OnClick__10GM_GunShopFUi` | 0x002d2b20 | CONFIRMED | slot 5: drop a slot, or take from the catalogue |

#### `GM_GameFailed` (`gui_game_over_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057ca40 | `gui_game_over_tick` | `OnClick__13GM_GameFailedFUi` | 0x002d1bb0 | CONFIRMED | slot 5: 1000 Retry `0x4fd`, 1002 Exit `0x4fb` |

#### `GM_GameMenuRacing` (`gui_racing_game_menu_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057cb40 | `gui_racing_game_menu_tick` | `OnClick__17GM_GameMenuRacingFUi` | 0x002ddc30 | CONFIRMED | slot 5: ids `0x3e9`..`0x3ec` |

#### `GM_LoadProgress` (`gui_load_progress_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057cf80 | `gui_load_progress_on_exchange` | - | - | - | slot 1: rebuilds, refreshes the roster |
| 0x0057d0c0 | `gui_load_progress_tick` | `OnTick__15GM_LoadProgressFUi` | 0x002d2ff0 | STRUCTURAL | slot 5: roster refresh, session-readiness codes `0x39`, `0x3a` |
| 0x0057d240 | `gui_load_progress_set_progress_game_init` | `SetProgressGameInit__15GM_LoadProgressFf` | 0x002d3030 | CONFIRMED | `base + (1 - base) * t` into component 300 |
| 0x0057d320 | `gui_load_progress_set_progress_mission_open` | `SetProgressMissionOpen__15GM_LoadProgressFf` | 0x002d3050 | CONFIRMED | `t * base` into component 300 |
| 0x0057d400 | `gui_load_progress_set_caption` | `OnCreate__15GM_LoadProgressFv` | 0x002d2cc0 | STRUCTURAL | component 400 |
| 0x0057d420 | `gui_load_progress_present` | `Present__15GM_LoadProgressFv` | 0x002d3070 | CONFIRMED | a hand-rolled frame, drawn while the loader blocks |

### 3d. Free helpers

#### the inventory screen (`gui_inventory_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00578940 | `gui_inventory_format_slot_row` | - | - | - | formats one permanent row's label |

#### `GM_GunShop` (`gui_gun_shop_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057b530 | `gui_gun_shop_format_slot_row` | - | - | - | formats one carried slot's label |

## 4. What it is, and the lever that opened it

**88 real function starts between the two guards' tails, all 88 named.** The raw start count is 98; ten of
those are decompiler artifacts rather than functions (§12).

**Every screen constructor in the GUI is identifiable without reading its body.** Each one pushes
exactly one literal into

```c
gui_menu_build_components_from_design(&g_gui_screen_design_table, <design_id>, <gate>);
```

and that literal is the **group index in the shipped `tables\menu.def`** — byte-verified, the
group walker compares each record's tag against `'stop'` (`cmp ecx, 0x73746f70`). Decoding the
file therefore hands you a screen's component ids, its widget classes, its geometry and — via
`textdb.json` — its on-screen labels, before a single instruction is read.

Decoding `menu.def` by walking those group records yields **72 groups**, ids `0x00`..`0x47` (the
last 16 empty), **809 records of `0x24` bytes**:
`{u32 id, FourCC tag, f32 x, y, w, h, u32 text_id, u32 flags, u32 colour}`.

**Seven non-empty shipped designs are never built.** The builder has 58 `E8` callers in the whole
binary, no `E9` referrer, and every one of the 58 pushes a literal design id; the design table's
only other reference is its loader. None of the 58 pushes `0x11` (*Your inventory is full*),
`0x12` (a *Pickup/Use* list), `0x15` (three unlabelled items), `0x24` (*You have won a bonus
car*), `0x29` (a *Multiplayer:* name field), `0x2d` (a name field, a combo box and two buttons)
or `0x31` (*Exit to Main Menu* / *Retry*: the game-over design `0x30` with the two rows in the
other order).

### 4a. The design map of this TU

Thirty-five of the 88 starts push a design literal. `gate` selects a variant group within the
same design id.

| PC constructor | VA | design | gate | shipped caption |
|---|---|---|---|---|
| `gui_combo_dropdown_screen_ctor` | `0x00575c50` | `0x13` | 0 | *(the combo drop-down list)* |
| `gui_notice_panel_ctor` | `0x00575f50` | `0` | 0 | *(two message rows, no buttons)* |
| `gui_notice_panel_ctor_from_strings` | `0x005761d0` | `0` | 0 | *(same, `const char *` form)* |
| `gui_question_box_ctor` | `0x005764c0` | `1` | 0 | Yes / No |
| `gui_message_box_ctor` | `0x00576620` | `2` | 0 | OK |
| `gui_main_menu_exit_box_ctor` | `0x00576810` | `0x36` | 0 | "Quit Game?" (textdb 119) |
| `gui_main_menu_ctor` | `0x00576920` | `6` | **2** | Main Menu: |
| `gui_game_menu_exit_box_ctor` | `0x00576cc0` | `0x36` | 0 | "Exit Game?" (textdb 149) |
| `gui_game_menu_ctor` | `0x00576dd0` | `7` | 0 | Game menu: |
| `gui_game_menu_on_exchange` | `0x00576e30` | `7` | 0 | *(rebuild)* |
| `gui_options_audio_video_ctor` | `0x00576fe0` | `8` | `g_net_session != 0` | Audio & Video |
| `gui_options_ctor` | `0x00577990` | `9` | 0 | Options: |
| `gui_options_controls_ctor` | `0x00577b10` | `10` | 1 | Player Controls: |
| `gui_options_controls_on_exchange` | `0x00577b90` | `10` | 1/2/4 | *(per input device)* |
| `gui_key_capture_ctor` | `0x005782e0` | `0xb` | 0 | Setup Controls: |
| `gui_key_conflict_ctor` | `0x005786e0` | `0x35` | 0 | *(the conflict caption: the control is taken)* |
| `gui_inventory_ctor` | `0x00578a30` | `0xf` | var | Inventory: |
| `gui_action_menu_ctor` | `0x00579020` | `0x10` | 0 | Pickup/Use |
| `gui_profile_select_ctor` | `0x00579900` | `4` | 0 | Player Profile: |
| `gui_profile_select_on_exchange` | `0x005799d0` | `4` | 0 | *(rebuild)* |
| `gui_profile_create_ctor` | `0x00579fb0` | `3` | 0 | Enter Profile Name: |
| `gui_profile_delete_ctor` | `0x0057a1f0` | `5` | 0 | Delete Profile: |
| `gui_load_game_ctor` | `0x0057a4a0` | `0xc` | 0 | Load Game: |
| `gui_free_ride_ctor` | `0x0057aaf0` | `0xe` | 0 | Free Ride: |
| `gui_free_ride_location_ctor` | `0x0057aff0` | `0x34` | 0 | Location: *(start district)* |
| `gui_session_list_ctor` | `0x0057b150` | `0x2f` | 0 | Select session: |
| `gui_gun_shop_ctor` | `0x0057b3a0` | `0x14` | 0 | Inventory (Drop) *(the weapon shop)* |
| `gui_garage_select_ctor` | `0x0057b930` | `0x25` | 0 | Garage |
| `gui_garage_full_ctor` | `0x0057bfa0` | `0x26` | 0 | Garage |
| `gui_garage_summary_ctor` | `0x0057c540` | `0x27` | var | Close |
| `gui_game_over_ctor` | `0x0057c8e0` | `0x30` | 0 | Game Over |
| `gui_racing_game_menu_ctor` | `0x0057cab0` | `0x2e` | 0 | Game menu: *(with Skip Race)* |
| `gui_load_progress_ctor` | `0x0057cc90` | `0x32` | 0 | Loading progress |
| `gui_load_progress_on_exchange` | `0x0057cf80` | `0x32` | 0 | *(rebuild)* |
| `gui_free_ride_mode_ctor` | `0x0057d4b0` | `0x33` | 0 | Location: *(free-ride scenario)* |

### 4b. Slot 5 is the TICK

The PC `GM_Menu` vtable has **one** input slot, and every override has the same shape:

```c
id = gui_menu_tick(...);           // the per-frame focus/mouse/key walk, RETURNS the picked id
if (id == 0 && (key & 0xff) == 1)  // ESC
switch (id) { ... }                // the dispatch
```

So a slot-5 function is named `_tick` after the **PC slot**. The cross-build fold counts are in §13b.

## 5. The main menu and the in-game menu

Both are plain item lists whose rows come straight from `menu.def`, and both delegate the
"really quit?" question to a tiny design-`0x36` box.

| function | VA | role |
|---|---|---|
| `gui_main_menu_ctor` | `0x00576920` | design 6 gate 2; camera target `0xf1b31`; deletes the two unshipped rows |
| `gui_main_menu_tick` | `0x00576a20` | 12 arms over ids `0xf1b95`..`0xf1ba1` |
| `gui_main_menu_exit_box_ctor` / `_tick` | `0x00576810` / `0x00576890` | "Quit Game?" → result `0x18` |
| `gui_game_menu_ctor` | `0x00576dd0` | design 7 |
| `gui_game_menu_on_exchange` | `0x00576e30` | slot 1 rebuild |
| `gui_game_menu_tick` | `0x00576e60` | ids `0x66`/`0x67`/`0x6c`/`0x6f` |
| `gui_game_menu_exit_box_ctor` / `_tick` | `0x00576cc0` / `0x00576d40` | "Exit Game?" → result `0x21` |

**The main-menu row ids are the cut-content evidence.** Shipped design 6 gate 2 lists
990101..990106 and 990109..990113. The constructor disables `0xf1b9a` (990106, *Free Ride
Extreme*) below a licence level and **deletes `0xf1b9b` and `0xf1b9c`** (990107, 990108) —
neither of which the retail `menu.def` contains. `0xf1b9c` is the Multiplayer entry
(`structs/multiplayer.md` §1); the tick still has live arms for both.

## 6. The three modal boxes

The PC splits its modal vocabulary three ways, one shipped design each. The discriminator is the
**component id set**, not the auto-fit layout pass — all three run that pass, so it separates
nothing.

| class | design | components | dismissal | PC functions |
|---|---|---|---|---|
| **notice panel** | `0` | `vers`, frame `1`, `ctxt` `0x82`, `ctxt` `0x83` — **no buttons** | any key or click | `gui_notice_panel_ctor` `0x00575f50`, `gui_notice_panel_ctor_from_strings` `0x005761d0`, `gui_notice_panel_tick` `0x00576450` |
| **question box** | `1` | frame `0x82`, message `0x83`, **item `0x8a` "Yes"**, **item `0x8b` "No"** | Yes → `0xb`, No/ESC → `0xc` | `gui_question_box_ctor` `0x005764c0`, `gui_question_box_tick` `0x00576560` |
| **message box** | `2` | frame `1`, message `0x82`, `butt` `0x83` "OK" | any click, ESC or ENTER | `gui_message_box_ctor` `0x00576620`, `gui_message_box_tick` `0x005767a0` |

The notice panel is used for plain game messages: script-dispatcher cases `0x100f`/`0x1010`,
and the inventory pair *"You do not have enough room."* / *"You must drop an item."*
`gui_notice_panel_present_once` (`0x0056e3f0`, in the `g_menu.cpp` TU) drives one for exactly
one frame — a splash, not an interactive modal. The notice panel is PC-only (§13c).

### 6a. The combo drop-down

`gui_combo_dropdown_screen_ctor` (`0x00575c50`, design `0x13`) is the pop-up list a combo-box
widget opens, and `gui_combo_dropdown_tick` (`0x00575ea0`) is its slot 5. Both guard on the
`options` field (§1a); the picked row id is written through `out_picked_id`.

**Component id 2 is the dismiss target.** Design `0x13` ships exactly three records — a `wind`
frame (1), a `rect` background (2) and an `orez` clipper (3) — and the tick hit-tests component
2 on every mouse button, closing with `-1` when the click landed outside it. That is the PC's
click-away dismissal (§13c).

## 7. Options, controls, items and pickers

| function | VA | role |
|---|---|---|
| `gui_options_ctor` / `_tick` | `0x00577990` / `0x00577a20` | design 9; rows 100 Controls, 101 Audio & Video, 400 Back |
| `gui_options_controls_ctor` | `0x00577b10` | design 10 gate 1 |
| `gui_options_controls_on_exchange` | `0x00577b90` | slot 1; gate 1/2/4 = keyboard / mouse / joystick, then re-seed every value |
| `gui_options_controls_close` | `0x00577d20` | slot 4; apply bindings + reload the speedo texture |
| `gui_options_controls_tick` | `0x00577d50` | every slider and toggle; `0x35` restores defaults |
| `gui_free_ride_ctor` | `0x0057aaf0` | design `0xe` |
| `gui_free_ride_on_close_child` | `0x0057ac40` | slot 3; car name, mode label and `freerideN.bmp` |
| `gui_free_ride_tick` | `0x0057ad80` | ids 100 / `0x65` / 200 / `0xc9` |

**Design 10 is the biggest group in the file — 149 records across three gates**, one per input
device. The controls page persists straight into the settings globals and calls
`loadsave_apply_settings` with a per-block dirty mask (8 = the joystick axis block, `0x10` =
force feedback). Toggle `0x4b` is the **mph/kph** switch, which is why closing the page reloads
the speedometer texture.

Free ride keeps the chosen car and the mode in its own fields (§1j); modes `0x1c`..`0x20` each
have their own preview bitmap and their own label (textdb `0x41a`..`0x41e`). Mode `0x1c` starts
immediately; the others open a location picker first (§7f), as observed in game.

### 7a. Audio & Video — design 8 *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_options_audio_video_ctor` | `0x00576fe0` | design 8; seeds every control from the settings block |
| `gui_options_audio_video_tick` | `0x005771b0` | slot 5; applies whichever control moved |

**This screen is what decodes the display settings block.** Every control writes a field of
`g_act_mission+0x90`, and joining the component ids to their shipped labels by matching Y
coordinate names the five setters:

| component | shipped id | label (textdb) | setter |
|---|---|---|---|
| `0x6e` | 110 | 231 *Level of Details* | `mission_settings_set_details_scale`, block `+0x00`, clamp [0.4, 2.0] |
| `0x6f` | 111 | 232 *Draw Distance* | `mission_settings_set_draw_distance_scale`, block `+0x04`, clamp [0.3, 1.0] |
| `0x70` | 112 | 238 *Level of Blood* | `g_blood_level` |
| `0x71` | 113 | 260 *In Game Effects:* | `mission_settings_set_effects_quality`, block `+0x0c`, 0..2 |
| `0x72` | 114 | 270 *In Game Sounds:* | `mission_settings_set_sounds_quality`, block `+0x10`, 0..2 |
| `0x73` | 115 | 280 *Shadows:* | `mission_settings_set_shadow_quality`, block `+0x14`, 0..5 → `{0,1,3,6,9,15}` |
| `0x74` | 116 | 290 *Particles:* | `mission_settings_set_particles_quality`, block `+0x18`, 0..2 |
| `0x50` | 80 | 239 *Gamma* | `g_gamma`, settings block `4` |
| `0x51`..`0x54` | 81..84 | 251..254 *Sounds / Music / Cars / Speech* | `loadsave_apply_settings` masks 1 / 2 / `0x20` / `0x40` |

Three details pin that join:

* **Gamma is component `0x50`**, not one of the `0x6e`/`0x6f` sliders — it writes a different
  global entirely.
* **Every global the screen writes has a consumer outside it.** `g_gamma` is settings block
  `+0x4fc` of `g_loadsave_singleton`, the input of the 256-entry gamma ramp that
  `loadsave_apply_settings` builds for mask 4 and hands to engine slot `+0x110`; `g_cars_volume`
  is `+0x4ec`, divided by the sounds volume and passed to `vehicle_set_global_sound_volume` for
  mask `0x21`; `g_speech_volume` scales `speech_channel_speak_line` and the scream volumes of
  `human_new_scream`, `human_game_init`, `human_do_throw_occupant_from_car` and
  `pedestrian_manager_play_pedestrian_scream`; `g_blood_level` gates `hud_push_blood_splat` at
  0.9; `g_freeride_unlock_level` is read by `gui_main_menu_ctor`, `gui_free_ride_ctor` and
  `gui_free_ride_mode_ctor`.
* **The Shadows tier has six labels**, textdb 281..286 *Very Low … Highest*, matching its
  six-entry map exactly.
* `mission_update_settings` (`0x00604f70`) corroborates the first two rows independently: it
  pushes block `+0x00` to the I3D driver as render parameter `0x14`, and multiplies block `+0x04`
  by the base view range at `+0x98`.

The presets `0x65`/`0x66`/`0x67` (Low/Medium/High) set all four quality tiers at once; `0x5a`
restores defaults through `loadsave_default_settings(8)`. The tick returns close code `0x22`
only when the screen has no parent, i.e. when it was opened from the main menu.

### 7b. Key binding — designs `0xb` and `0x35` *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_key_capture_ctor` | `0x005782e0` | design `0xb` "Setup Controls:" |
| `gui_key_capture_on_close_child` | `0x00578370` | slot 3; closes itself once the conflict child returns |
| `gui_key_capture_tick` | `0x005783a0` | slot 5; captures the event, commits or escalates |
| `gui_key_conflict_ctor` | `0x005786e0` | design `0x35`, the conflict caption; carries the captured triple |
| `gui_key_conflict_tick` | `0x005787c0` | slot 5; Yes commits, No/ESC abandons |

A committed binding is three dwords at **`g_key_binding_table + slot*0xc`**; the input side
reads it through `input_set_binding`, `input_update` and `input_binding_get_label`
(`structs/loadsave.md` §9). Both screens normalise the
paired axis codes the same way (5/7, `0x26`/`0x28` swap; `0x12`/`0x13` clear), which is what
pairs them. Both are PC-only (§13c).

### 7c. The inventory screen — design `0xf` *(PC-only as a screen)*

| function | VA | role |
|---|---|---|
| `gui_inventory_ctor` | `0x00578a30` | design `0xf`, gate 1/2 = on foot / in car |
| `gui_inventory_format_slot_row` | `0x00578940` | free function; one row's label |
| `gui_inventory_on_close_child` | `0x00578d10` | slot 3; takes an item a child handed back |
| `gui_inventory_tick` | `0x00578d90` | slot 5; select, drop, or leave |

Permanent rows: `0x100` Hands, `0x102`..`0x106` the five weapon slots, `0x107` Coat, `0x108` Car,
then one cloned `'seli'` row per carried item in the `0x200+` band. Drop is key `0xe`/`0x2d`; selecting a row calls
`inventory_select_slot(index, kind, removed_items)` with kind 0 Hands / 1 weapon / 2 Coat / 3 item /
4 Car. The console counterpart is not a screen (§13c).

### 7d. The action menu — design `0x10`

| function | VA | role |
|---|---|---|
| `gui_action_menu_ctor` | `0x00579020` | design `0x10`; gathers world items + NPC inventories |
| `gui_action_menu_scalar_deleting_dtor` | `0x00579480` | slot 0 — the only class-specific one in the TU |
| `gui_action_menu_dtor` | `0x005794a0` | frees the options vector |
| `gui_action_menu_pick_up` | `0x005794d0` | records a take, or strips the item from every NPC |
| `gui_action_menu_on_close_child` | `0x00579590` | slot 3; takes the item a child confirmed |
| `gui_action_menu_tick` | `0x00579610` | slot 5; number keys 2..10 → rows `0x100`..`0x108` |

This is the interact-button menu, and it is **not only a pickup menu** — it also carries
open-door, enter-car and the other context actions, which is why the prefix is
`gui_action_menu_` rather than the PS2 class's `GM_ItemPickUp`.
`basic/artifacts/player-interactions.md` §4 is the behavioural write-up.

**The tick auto-takes a lone option**, gated by an `& 2` flag on a mission field (§13a).

### 7e. The gun shop — design `0x14`

| function | VA | role |
|---|---|---|
| `gui_gun_shop_ctor` | `0x0057b3a0` | design `0x14`; builds the catalogue |
| `gui_gun_shop_format_slot_row` | `0x0057b530` | free function; one carried-slot label |
| `gui_gun_shop_refresh_slots` | `0x0057b600` | re-render the player's slots after every trade |
| `gui_gun_shop_tick` | `0x0057b700` | slot 5; drop a slot, or take from the catalogue |

**This is the weapon shop.** Two independent artifacts say so here, and the twin class name is a
third (§13a).

1. The shipped scripts call it: `gunshop_menu` appears twice in mission `MISE19-MESTO`, script
   `1ProdZ`, and once in mission `MISE20-MESTO`, script `1ProdZ` — `1ProdZ` the weapons dealer — each time directly after `dialog_end` with him and directly before
   `inventory_push` / `inventory_clear`.
2. The opcode: `GUNSHOP_MENU` is parser raw `0x060`, so dispatcher **case `0x61`**, whose body
   opens this design as a modal and then calls `human_change_weapon_model`. Documented in
   `script-mapping/artifacts/opcodes-verified.md`.

It is a weapon **PICKUP**, not a drop, despite the shipped window caption *"Inventory (Drop)"*
(textdb 1300): the right band lists every `g_game_item_table` entry whose flags lack `0x800000`
and inserts the pick, while the left band only frees capacity. Textdb 1310 labels the right band
*"Select (Pick Up)"*. There is no money or score — the main story has neither.

### 7f. The free-ride pickers — designs `0x33` and `0x34` *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_free_ride_mode_ctor` | `0x0057d4b0` | design `0x33`; the SCENARIO picker with previews |
| `gui_free_ride_mode_tick` | `0x0057d580` | slot 5; rows 401..405 → modes `0x1c`/`0x1e`/`0x20`/`0x1d`/`0x1f` |
| `gui_free_ride_location_ctor` | `0x0057aff0` | design `0x34`; the START-DISTRICT picker |
| `gui_free_ride_location_tick` | `0x0057b0b0` | slot 5; ids 101..107 → `loadsave_freeride_store_state(id-100)` |

Both ship the caption *"Location:"* (textdb 199), which is why they are easy to confuse. They do
different jobs and sit at different points in the flow:

`gui_free_ride_tick` row 100 opens **design `0x33`**, the five scenarios (textdb 1050..1054:
*City - Small*, *City - Daytime*, *City - Night*, *Countryside - Daytime*, *Countryside -
Night*), each with its own `freerideN.bmp` preview swapped on focus change. Picking one closes
with the free-ride mode. Then `gui_free_ride_tick`'s Start row (`0xc9`) starts immediately for
mode `0x1c`, and for anything else opens **design `0x34`**, the four districts (*Little Italy*,
*Central Island*, *Down Town*, *Hoboken*).

**Design `0x34`'s four rows serve both maps.** When the mode is `0x1f` or `0x20` — the two
COUNTRYSIDE scenarios — the constructor relabels rows 101..104 to textdb 4010..4013: *China Town
Exit*, *Dam*, *Lay-by*, *Motel*.

`g_freeride_unlock_level` gates how many scenarios design `0x33` enables: 0 leaves only *City - Small*, 1
adds the two City rows, 2 and above enables all five.

## 8. Game over and the racing in-game menu

| function | VA | role |
|---|---|---|
| `gui_game_over_ctor` | `0x0057c8e0` | design `0x30`; pushes the failure text into the `'list'` block, then re-lays-out |
| `gui_game_over_tick` | `0x0057ca40` | 1000 Retry → `0x4fd`, 1002 Exit → `0x4fb` |
| `gui_racing_game_menu_ctor` | `0x0057cab0` | design `0x2e`; relabels row `0x3e9` in a network session |
| `gui_racing_game_menu_tick` | `0x0057cb40` | ids `0x3e9`..`0x3ec` |

In **free ride** the game-over screen disables *Retry* and focuses *Exit to Main Menu* — there
is no mission to retry (§13a).

## 9. Load game and loading progress

| function | VA | role |
|---|---|---|
| `gui_load_game_ctor` | `0x0057a4a0` | design `0xc`; one `'seli'` row per save, newest first |
| `gui_load_game_tick` | `0x0057a670` | scrolling, the thumbnail preview, and the load |
| `gui_load_progress_ctor` | `0x0057cc90` | design `0x32`; `tables\load.def`; the MP roster |
| `gui_load_progress_on_exchange` | `0x0057cf80` | slot 1 rebuild + roster refresh |
| `gui_load_progress_tick` | `0x0057d0c0` | roster refresh + session-readiness codes `0x39`/`0x3a` |
| `gui_load_progress_set_progress_game_init` | `0x0057d240` | `base + (1 - base) * t` → component 300 |
| `gui_load_progress_set_progress_mission_open` | `0x0057d320` | `t * base` → component 300 |
| `gui_load_progress_set_caption` | `0x0057d400` | component 400 |
| `gui_load_progress_present` | `0x0057d420` | a hand-rolled frame, drawn while the loader blocks |

**The save list is reversed.** Rows are cloned walking the file vector backwards from `end`, so
row 100 is the newest save, and the click handler undoes it with `(count - id) + 99`. The
save-file record is **`0x18` bytes** (§1i).

**The scroll range is `'swnd'` + `0x28`.** `gui_load_game_ctor` computes it from the last row's
geometry and writes it directly (§13a).
`structs/gui_widget.md` §8f documents the scroll window this range feeds.

**The progress bar has two phases split at `+0x48`**: mission-open fills `[0, base]` and
game-init fills `[base, 1]`. In a network session each update is also broadcast as a **type
`0x36`** event message so the other players' bars advance; that is why those two functions touch
the message vtable `0x0063e0bc` — the vtable belongs to the message they construct, not to them.

**The save-slot widget's slot 8, `gui_widget_save_slot_set_text`, discards three lookups.** It
asks the text database for textdb `0xae`, `0xaf` and `0xb0` and drops all three results before
forwarding to its child at `+0x170`. The lookup is a pure binary search with no side effect;
`0xaf` is formatted by `gui_load_game_tick`, and `0xae` and `0xb0` are referenced nowhere else in
the binary.

**`tables\load.def`** is read into an 8000-byte buffer and walked as **`0x44`-byte** records:
the mission name at `+0x00`, the loading caption at `+0x20`, and an optional textdb id at
`+0x40`.

## 10. The multiplayer session browser

| function | VA | role |
|---|---|---|
| `gui_session_list_ctor` | `0x0057b150` | design `0x2f`; clones one `'seli'` row per session name |
| `gui_session_list_tick` | `0x0057b2e0` | pick / cancel / time out and refresh |

Design `0x2f` ships exactly **two** records — the frame (textdb 610 *"Select session:"*) and one
row template with id 1000 (textdb 611 *"<none found>"*). The constructor takes the caption's
textdb id, a `vector<char*>` of session names and a **refresh timeout in milliseconds**; both
callers in `net_multiplayer_run` pass **2000**. The tick counts that down and closes with `0x41`,
which is the case that re-opens the browser — a 2-second auto-refresh. The focused row is
remembered in `g_gui_session_list_focused_row_id` so the refresh does not lose the selection.

## 11. The three player-profile screens

| function | VA | role |
|---|---|---|
| `gui_profile_select_ctor` | `0x00579900` | design 4 |
| `gui_profile_select_on_exchange` | `0x005799d0` | slot 1 rebuild |
| `gui_profile_select_build_rows` | `0x00579a00` | one `'item'` row per profile + re-layout |
| `gui_profile_select_on_close_child` | `0x00579bf0` | slot 3; creates the profile |
| `gui_profile_select_tick` | `0x00579cc0` | pending action, then the picked row |
| `gui_profile_create_ctor` | `0x00579fb0` | design 3 |
| `gui_profile_create_scalar_deleting_dtor` | `0x0057a070` | slot 0 |
| `gui_profile_create_dtor` | `0x0057a090` | leaves text-input mode |
| `gui_profile_create_tick` | `0x0057a0f0` | OK/ENTER → `0xd`, Cancel/ESC → `0xe` |
| `gui_profile_delete_ctor` | `0x0057a1f0` | design 5 |
| `gui_profile_delete_on_close_child` | `0x0057a370` | slot 3; performs the delete |
| `gui_profile_delete_tick` | `0x0057a3c0` | opens the confirm box per row |

**The PC profile record is `0x54` bytes with the name at `+0x10`.** At most **12** profiles exist:
`gui_profile_select_build_rows` disables *Create New Profile* once the count exceeds 11, and
disables *Delete Profile* below two.

**`g_engine` vtable `+0x88` is the keyboard text-input mode selector.** The name-entry
constructor sets it to **2** and its destructor restores **1**; that 2 → 1 pairing is the only
thing in the binary that pins the field's meaning, and it is why this class is one of the few
screens that does not use the shared slot-0 destructor.

The select screen drives itself through `pending_action` (§1g). The constructor seeds it
to 1 when no profile exists, so a fresh install lands straight in the name box.

## 12. The ten addresses in this range that are not function starts

Ten defs inside the unit are Aggressive-Instruction-Finder artifacts, in two
distinct shapes. They are what separates the raw start count of 98 from the 88 real starts.

**Nine are EPILOGUE arms:** `0x00576612` · `0x00576ed5` · `0x00577dd2` · `0x00578e65` ·
`0x0057a866` · `0x0057a887` · `0x0057cbb3` · `0x0057d1e1` · `0x0057d206`. For each one, a byte
scan of `.text` finds no `E8 rel32` reaching it, a scan of the data sections finds it as no dword,
it is the target of an earlier `jcc` inside its host, and the eight bytes before it are a
`ret imm16` teardown (`c2 1c 00` — a `__thiscall` screen tick). Seven of the nine sit inside a
`*_tick`, whose bodies are one long switch full of early-outs.

**One is a MID-BODY SPLIT:** `0x0057bd70`, inside `gui_garage_select_tick`. Ghidra decompiles it
as a verbatim copy of its host's tail, reading `unaff_EBX` / `unaff_EDI` / `in_stack_*` — values
live on entry, which a real function start cannot have. The discriminator here is the **vtable
order in `.rdata`**: it appears in none of the eleven screen vtables of this TU while every one
of its neighbours does, and a screen-TU function in no vtable, with no `E8` caller and no
dataref, cannot be reached at all. The split sits 96 bytes into its host, a 608-byte function
(§13a).

## 13. PC vs PS2

The corresponding PS2 range is **`[0x002cf2f0, 0x002d41e0)`**, an 18-class TU:
`GM_MainMenu(Exit)`, `GM_GameMenu(Exit)`, `GM_Options`, `GM_OptionsControls`,
`GM_OptionsScreenSlide`, `GM_FreeRide`, `GM_QuestionBox`, `GM_LoadGame`, `GM_GameFailed`,
`GM_ItemPickUp`, `GM_GunShop`, `GM_LoadProgress`, `GM_PressStart`, `GM_GameMap`,
`GM_TutorialMessage`, `GM_HotRodMessage`. The PC TU additionally absorbs the three
`GM_Profile*` screens (a separate TU on the PS2, at `0x002df4c0`) and `GM_GameMenuRacing` (which
sits with the racing menus there), so the PC file is a wider cut of the same material.

### 13a. The strongest shared constants

These are what make the twins measurements rather than positional guesses.

| PC function | PS2 method | the shared fact |
|---|---|---|
| `gui_profile_delete_tick` | `OnClick__16GM_ProfileDeleteFUi` | both pass textdb **`0x82`** ("Delete") to the confirm box at the same site |
| `gui_profile_create_tick` | `OnClick__16GM_ProfileCreateFUi` | close arguments **1**/**0** and results **`0xd`**/**`0xe`** |
| `gui_profile_select_tick` | `OnClick__16GM_ProfileSelectFUi` | results **`0xf`**/**`0x10`**, same `100..199` row band |
| `gui_load_game_tick` | `OnClick__11GM_LoadGameFUi` | reversal `(count-1)-(id-100)`, close result **`0x13`**, textdb **`0xaf`** into component **`0x3f2`** |
| `gui_load_game_ctor` | `OnCreate__11GM_LoadGameFv` | textdb **`0xac`** into component **10**; the range write on component 4 |
| `gui_load_progress_ctor` | `OnCreate__15GM_LoadProgressFv` | 8000-byte buffer, **`0x44`** stride, caption `+0x20`, textdb id `+0x40`, constant **`0x2ce`** |
| `gui_load_progress_set_progress_*` | `SetProgressGameInit` / `SetProgressMissionOpen` | the two formulas verbatim |
| `gui_question_box_tick` | `OnClick__14GM_QuestionBoxFUi` | results **`0xb`**/**`0xc`** and the root-vs-child fork |
| `gui_free_ride_ctor` | `OnCreate__11GM_FreeRideFv` | car-name fallback textdb **`0x2bf`** under the same range test |
| `gui_main_menu_exit_box_tick` | `OnClick__15GM_MainMenuExitFUi` | result **`0x18`** |
| `gui_action_menu_pick_up` | `PickUp__13GM_ItemPickUpFv` | a statement-level diff of both bodies matches statement for statement |
| `gui_gun_shop_refresh_slots` | `RefreshInventory__10GM_GunShopFv` | STRUCTURAL: the PS2 body is a 16-byte stub, pinned by two identical call positions |

Further correspondences, keyed to the PC sections they concern:

* **Question box (§6).** `GM_QuestionBox` dispatches components `0x8a`/`0x8b` and returns
  `0xb`/`0xc`; those ids live in design **1**, so that class is the design-1 question box. Its
  constructor flag only picks the console layout `psx_questionbox`; the Yes/No pair is always
  there. The design-**0** notice panel has no PS2 counterpart (§13c).
* **Action menu (§7d).** The field map is 1:1 with `GM_ItemPickUp` at a constant `+0x101c` delta:
  PC `+0x1048`/`+0x104c`/`+0x1050`/`+0x1054`/`+0x1058`/`+0x1060` ↔ PS2
  `+0x2c`/`+0x30`/`+0x34`/`+0x38`/`+0x3c`/`+0x40`. One behaviour is RELOCATED, not folded: the PS2
  auto-takes a lone option inside `OnCreate`; the PC runs the same test — same `& 2` flag on the
  same mission field — in the tick. Both builds have both functions, so this is not an inlining.
* **Gun shop (§7e).** The twin class is `GM_GunShop`.
* **Game over (§8).** The free-ride tail — disable *Retry*, focus *Exit to Main Menu* — is
  identical on both builds.
* **Load game (§9).** `gui_load_game_ctor` writes the `'swnd'` + `0x28` scroll range directly at
  the place where `OnCreate__11GM_LoadGameFv` calls the widget's `SetRangeFloat`.
* **Garage select (§12).** A size ratio taken against the `GM_GarageSelect` methods uses the full
  608 bytes of `gui_garage_select_tick`, not the 96 bytes before the mid-body split.

### 13b. Folds — PC inlines what the PS2 calls

Each of these carries a `// PS2-INLINED HERE:` banner at the fold site in the annotated decomp.

| PC function | what it absorbs |
|---|---|
| `gui_options_controls_on_exchange` | `SetUpTexts__18GM_OptionsControlsFv` — a real method `OnCreate` CALLS; the PC inlines it **twice** (here and in the tick's `0x35` arm) |
| `gui_load_progress_set_progress_game_init` | `SetRawProgress__15GM_LoadProgressFf` |
| `gui_load_progress_set_progress_mission_open` | `SetRawProgress__15GM_LoadProgressFf` |
| `gui_profile_create_tick` | `OnKey__16GM_ProfileCreateFUcUi`, which on the PS2 calls `OnClick(0x66)` back through the vtable |
| `gui_question_box_ctor` | `OnCreate__14GM_QuestionBoxFv` |
| every `*_tick` | 1–5 of `OnClick` / `OnKey` / `OnTick` / `OnChange` / `OnMouse`: the PS2 base has these six input virtuals (with `Tick`) where the PC has slot 5. A tick's `PS2 twin` names the largest it absorbs, usually `OnClick`, and the registry row records the fold count; `gui_main_menu_tick` is 1 → 1, `gui_load_game_tick` 5 → 1 |

### 13c. PC-only, measured against the closed PS2 roster

* **Every `*_on_exchange`** (slot 1). No `GM_*` class in the PS2 TU overrides `OnExchange`; the
  console build cannot change language or resolution at runtime.
* **`gui_options_controls_close`** (slot 4) — the PS2 class has no `OnClose`/`MenuClose`
  override, and no mph/kph speedometer switch to commit.
* **`gui_message_box_ctor` / `_tick`** — the PS2 TU's only modal box is `GM_QuestionBox`.
* **The whole session browser** — the DirectPlay front end is PC-only.
* **The 15-row loading roster** — `GM_LoadProgress` has no network roster; its `OnTick` is an
  8-byte stub.
* **The five `freerideN.bmp` previews** and the free-ride location pickers.
* **The design-0 notice panel** (§6). At every call position of its constructors the PS2 builds
  nothing: its `C_entity::Process` arms for the two entity-script commands `0x100f` / `0x1010`
  only advance the script, its `GM_GunShop` is gutted (`OnKey` and `RefreshInventory` are
  stubs), and the remaining callers are the multiplayer front end.
* **`GM_GarageGarage` and `GM_GarageComplete`** — PS2-only: they are reached only through PS2
  script commands the PC lacks (`structs/garage.md` §6b).
* **The combo drop-down's click-away dismissal** (§6a) — `GM_ComboMenu` has no such arm; the
  console build has no mouse.
* **The key-capture and key-conflict screens** (§7b) — `GM_OptionsControls::OnKey` merely cycles
  a 0..3 control PRESET, and the closed `GM_*` roster has no capture or conflict class.
* **The inventory as a `GM_Menu` screen** (§7c) — the feature exists on the PS2, but that build
  draws the inventory in the HUD (`G_IndicatorsClass::_InventorySet`, `::InventoryScroll`,
  `::DrawInventory`, all taking a `G_Inventory *`). Measured against the closed roster plus a
  named alternative location.

### 13d. PS2-only, measured at the PS2 callers

Each PS2 method below is reached from one PS2 caller or one vtable slot, and at that position the
PC builds nothing or keeps the base `GM_Menu` method.

| PS2 method | its PS2 caller or slot | the PC at that position |
|---|---|---|
| every `GM_HotRodMessage` method | `GameDone__17G_IndicatorsClassFb`, when the HUD holds a hot-rod unlock that the PS2's rewritten `IntroSubtitlesAddSimple` recorded | `hud_game_done` ends after its six resets and has no such branch |
| every `GM_TutorialMessage` method | `TutorialAddText__17G_IndicatorsClassFUi` | the HUD method itself has no PC counterpart (`structs/hud.md`) |
| every `GM_PressStart` method | `Init__15G_LoadSaveClassFv`, before the front end opens | `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| every `GM_GameMap` method | `GameLoopSingle__Fv`, on the map key | `engine_game_loop` builds one screen, `gui_game_menu_ctor` (the ESC menu) |
| `GM_LoadGame::PreloadTextures`, and `IsTexturePreloaded`, which only it calls | `OnCreate__11GM_LoadGameFv` | `gui_load_game_ctor` loads no texture; `gui_load_game_tick` binds `shot%03d.bmp` by name |
| `GM_LoadGame::ReleasePreloadedTextures` | `OnClose__11GM_LoadGameFv` | the load-game vtable `0x0063dfb4` keeps the base `gui_menu_close` in slot 4 |
| `GM_LoadProgress::Destroy` | its own vtable slot; the body unloads the resources, then calls the base | the load-progress vtable `0x0063e0a4` keeps the base `gui_menu_destroy` in slot 2 |

### 13e. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002b8a70 | `__dt__15GM_LoadProgressFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002ccb70 | `__dt__12GM_ComboMenuFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002ccbd0 | `OnBeforeCreate__12GM_ComboMenuFv` | fused into `gui_combo_dropdown_ctor` |
| 0x002ccbe0 | `OnCreate__12GM_ComboMenuFv` | fused into `gui_combo_dropdown_ctor` |
| 0x002ccde0 | `OnClose__12GM_ComboMenuFv` | the PC vtable keeps `gui_menu_close` in slot 4 |
| 0x002cf450 | `OnChange__11GM_MainMenuFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_main_menu_tick` |
| 0x002cf730 | `OnCloseChild__11GM_MainMenuFP7GM_MenuUiUi` | the PC vtable keeps `gui_menu_on_close_child` in slot 3 |
| 0x002cf820 | `OnTick__11GM_MainMenuFUi` | folded into slot 5, `gui_main_menu_tick` |
| 0x002cf9f0 | `OnKey__11GM_MainMenuFUcUi` | folded into slot 5, `gui_main_menu_tick` |
| 0x002cfb70 | `OnKey__11GM_GameMenuFUcUi` | folded into slot 5, `gui_game_menu_tick` |
| 0x002cfd60 | `OnCloseChild__11GM_GameMenuFP7GM_MenuUiUi` | the PC vtable keeps `gui_menu_on_close_child` in slot 3 |
| 0x002cfea0 | `OnTick__11GM_GameMenuFUi` | folded into slot 5, `gui_game_menu_tick` |
| 0x002d0310 | `OnTick__10GM_OptionsFUi` | folded into slot 5, `gui_options_tick` |
| 0x002d03a0 | `OnKey__18GM_OptionsControlsFUcUi` | folded into slot 5, `gui_options_controls_tick` |
| 0x002d0660 | `OnCreate__21GM_OptionsScreenSlideFv` | no PC class: none of the 27 screen vtables of this unit has its method set, and design 8 Audio & Video takes the screen's place in the options list |
| 0x002d06a0 | `OnKey__21GM_OptionsScreenSlideFUcUi` | no PC class: none of the 27 screen vtables of this unit has its method set, and design 8 Audio & Video takes the screen's place in the options list |
| 0x002d0990 | `OnChange__11GM_FreeRideFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_free_ride_tick` |
| 0x002d0d10 | `OnCreate__14GM_QuestionBoxFv` | fused into `gui_question_box_ctor` |
| 0x002d0f20 | `OnTick__14GM_QuestionBoxFUi` | folded into slot 5, `gui_question_box_tick` |
| 0x002d0fa0 | `OnKey__14GM_QuestionBoxFUcUi` | folded into slot 5, `gui_question_box_tick` |
| 0x002d12a0 | `IsTexturePreloaded__11GM_LoadGameFPc` | only `PreloadTextures` calls it (§13d) |
| 0x002d1340 | `PreloadTextures__11GM_LoadGameFPQ23std70vector<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>>` | `gui_load_game_ctor` loads no texture; `gui_load_game_tick` binds `shot%03d.bmp` by name (§13d) |
| 0x002d14c0 | `ReleasePreloadedTextures__11GM_LoadGameFv` | the load-game vtable `0x0063dfb4` keeps `gui_menu_close` in slot 4 (§13d) |
| 0x002d1520 | `OnClose__11GM_LoadGameFv` | the PC vtable keeps `gui_menu_close` in slot 4 |
| 0x002d1560 | `OnTick__11GM_LoadGameFUi` | folded into slot 5, `gui_load_game_tick` |
| 0x002d1730 | `OnKey__11GM_LoadGameFUcUi` | folded into slot 5, `gui_load_game_tick` |
| 0x002d1800 | `OnMouse__11GM_LoadGameFiiiUiUi` | folded into slot 5, `gui_load_game_tick` |
| 0x002d1930 | `OnChange__11GM_LoadGameFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_load_game_tick` |
| 0x002d1960 | `OnCloseChild__11GM_LoadGameFP7GM_MenuUiUi` | the PC vtable keeps `gui_menu_on_close_child` in slot 3 |
| 0x002d25d0 | `OnClick__13GM_ItemPickUpFUi` | folded into slot 5, `gui_action_menu_tick` |
| 0x002d2b80 | `__ct__15GM_LoadProgressFUifPUc` | fused into `gui_load_progress_ctor` |
| 0x002d2c60 | `Destroy__15GM_LoadProgressFv` | the vtable keeps `gui_menu_destroy` in slot 2 (§13d) |
| 0x002d2c90 | `OnBeforeCreate__15GM_LoadProgressFv` | fused into `gui_load_progress_ctor` |
| 0x002d3000 | `OnKey__15GM_LoadProgressFUcUi` | folded into slot 5, `gui_load_progress_tick` |
| 0x002d3060 | `SetRawProgress__15GM_LoadProgressFf` | inlined into `gui_load_progress_set_progress_game_init` and `gui_load_progress_set_progress_mission_open` (§13b) |
| 0x002d3110 | `OnCreate__13GM_PressStartFv` | no PC class: `Init__15G_LoadSaveClassFv` builds it, and `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| 0x002d3170 | `OnTick__13GM_PressStartFUi` | no PC class: `Init__15G_LoadSaveClassFv` builds it, and `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| 0x002d3270 | `OnKey__13GM_PressStartFUcUi` | no PC class: `Init__15G_LoadSaveClassFv` builds it, and `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| 0x002d32d0 | `OnCreate__10GM_GameMapFv` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d3340 | `OnKey__10GM_GameMapFUcUi` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d3390 | `OnClick__10GM_GameMapFUi` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d33a0 | `OnTick__10GM_GameMapFUi` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d3670 | `OnCreate__18GM_TutorialMessageFv` | no PC class: `TutorialAddText__17G_IndicatorsClassFUi` builds it, and that HUD method has no PC counterpart (`structs/hud.md`) |
| 0x002d3800 | `OnKey__18GM_TutorialMessageFUcUi` | no PC class: `TutorialAddText__17G_IndicatorsClassFUi` builds it, and that HUD method has no PC counterpart (`structs/hud.md`) |
| 0x002d3840 | `OnCreate__16GM_HotRodMessageFv` | no PC class: `GameDone__17G_IndicatorsClassFb` builds it for a hot-rod unlock, and `hud_game_done` has no such branch |
| 0x002d3870 | `OnTick__16GM_HotRodMessageFUi` | no PC class: `GameDone__17G_IndicatorsClassFb` builds it for a hot-rod unlock, and `hud_game_done` has no such branch |
| 0x002d38e0 | `OnKey__16GM_HotRodMessageFUcUi` | no PC class: `GameDone__17G_IndicatorsClassFb` builds it for a hot-rod unlock, and `hud_game_done` has no such branch |
| 0x002d3920 | `__dt__16GM_HotRodMessageFv` | no PC class: `GameDone__17G_IndicatorsClassFb` builds it for a hot-rod unlock, and `hud_game_done` has no such branch |
| 0x002d3980 | `OnBeforeCreate__16GM_HotRodMessageFv` | no PC class: `GameDone__17G_IndicatorsClassFb` builds it for a hot-rod unlock, and `hud_game_done` has no such branch |
| 0x002d39a0 | `__dt__18GM_TutorialMessageFv` | no PC class: `TutorialAddText__17G_IndicatorsClassFUi` builds it, and that HUD method has no PC counterpart (`structs/hud.md`) |
| 0x002d3a00 | `OnBeforeCreate__18GM_TutorialMessageFv` | no PC class: `TutorialAddText__17G_IndicatorsClassFUi` builds it, and that HUD method has no PC counterpart (`structs/hud.md`) |
| 0x002d3a10 | `__dt__10GM_GameMapFv` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d3a70 | `OnBeforeCreate__10GM_GameMapFv` | no PC class: `GameLoopSingle__Fv` builds it on the map key, and `engine_game_loop` builds one screen, `gui_game_menu_ctor` |
| 0x002d3a80 | `__dt__13GM_PressStartFv` | no PC class: `Init__15G_LoadSaveClassFv` builds it, and `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| 0x002d3ae0 | `OnBeforeCreate__13GM_PressStartFv` | no PC class: `Init__15G_LoadSaveClassFv` builds it, and `loadsave_init` builds one screen, `gui_profile_select_ctor` |
| 0x002d3af0 | `__dt__10GM_GunShopFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3b50 | `OnBeforeCreate__10GM_GunShopFv` | fused into `gui_gun_shop_ctor` |
| 0x002d3b60 | `OnKey__10GM_GunShopFUcUi` | folded into slot 5, `gui_gun_shop_tick` |
| 0x002d3c00 | `OnBeforeCreate__13GM_ItemPickUpFv` | fused into `gui_action_menu_ctor` |
| 0x002d3c10 | `__dt__13GM_GameFailedFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3c70 | `OnBeforeCreate__13GM_GameFailedFv` | fused into `gui_game_over_ctor` |
| 0x002d3c80 | `OnKey__13GM_GameFailedFUcUi` | folded into slot 5, `gui_game_over_tick` |
| 0x002d3c90 | `__dt__11GM_LoadGameFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3d10 | `OnBeforeCreate__11GM_LoadGameFv` | fused into `gui_load_game_ctor` |
| 0x002d3d20 | `__dt__14GM_QuestionBoxFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3d80 | `OnBeforeCreate__14GM_QuestionBoxFv` | fused into `gui_question_box_ctor` |
| 0x002d3dd0 | `__dt__11GM_FreeRideFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3e30 | `OnBeforeCreate__11GM_FreeRideFv` | fused into `gui_free_ride_ctor` |
| 0x002d3e40 | `__dt__21GM_OptionsScreenSlideFv` | no PC class: none of the 27 screen vtables of this unit has its method set, and design 8 Audio & Video takes the screen's place in the options list |
| 0x002d3ea0 | `OnBeforeCreate__21GM_OptionsScreenSlideFv` | no PC class: none of the 27 screen vtables of this unit has its method set, and design 8 Audio & Video takes the screen's place in the options list |
| 0x002d3eb0 | `__dt__18GM_OptionsControlsFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3f10 | `OnBeforeCreate__18GM_OptionsControlsFv` | fused into `gui_options_controls_ctor` |
| 0x002d3f20 | `__dt__10GM_OptionsFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d3f80 | `OnBeforeCreate__10GM_OptionsFv` | fused into `gui_options_ctor` |
| 0x002d3f90 | `OnKey__10GM_OptionsFUcUi` | folded into slot 5, `gui_options_tick` |
| 0x002d3fa0 | `__dt__11GM_GameMenuFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d4000 | `OnBeforeCreate__11GM_GameMenuFv` | fused into `gui_game_menu_ctor` |
| 0x002d4020 | `__dt__15GM_GameMenuExitFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d4080 | `OnBeforeCreate__15GM_GameMenuExitFv` | fused into `gui_game_menu_exit_box_ctor` |
| 0x002d40d0 | `__dt__11GM_MainMenuFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d4130 | `OnBeforeCreate__11GM_MainMenuFv` | fused into `gui_main_menu_ctor` |
| 0x002d4140 | `__dt__15GM_MainMenuExitFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002d41a0 | `OnBeforeCreate__15GM_MainMenuExitFv` | fused into `gui_main_menu_exit_box_ctor` |
| 0x002ddd70 | `OnCloseChild__17GM_GameMenuRacingFP7GM_MenuUiUi` | the PC vtable keeps `gui_menu_on_close_child` in slot 3 |
| 0x002dde00 | `OnTick__17GM_GameMenuRacingFUi` | folded into slot 5, `gui_racing_game_menu_tick` |
| 0x002dde70 | `OnKey__17GM_GameMenuRacingFUcUi` | folded into slot 5, `gui_racing_game_menu_tick` |
| 0x002dedd0 | `__dt__17GM_GameMenuRacingFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002dee30 | `OnBeforeCreate__17GM_GameMenuRacingFv` | fused into `gui_racing_game_menu_ctor` |
| 0x002df4c0 | `OnKey__16GM_ProfileSelectFUcUi` | folded into slot 5, `gui_profile_select_tick` |
| 0x002e05d0 | `OnTick__16GM_ProfileSelectFUi` | folded into slot 5, `gui_profile_select_tick` |
| 0x002e1a30 | `OnClose__16GM_ProfileCreateFv` | the PC vtable keeps `gui_menu_close` in slot 4 |
| 0x002e1ca0 | `MartinovaKlavesa__16GM_ProfileCreateFUiUi` | moves the focus over an on-screen letter grid, nine letters per row from textdb `0x27e`; the PC types through the engine's text-input mode (§11) |
| 0x002e2150 | `OnKey__16GM_ProfileCreateFUcUi` | folded into slot 5, `gui_profile_create_tick` |
| 0x002e2190 | `OnTick__16GM_ProfileCreateFUi` | folded into slot 5, `gui_profile_create_tick` |
| 0x002e27d0 | `OnTick__16GM_ProfileDeleteFUi` | folded into slot 5, `gui_profile_delete_tick` |
| 0x002e2830 | `__dt__16GM_ProfileDeleteFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002e2890 | `OnBeforeCreate__16GM_ProfileDeleteFv` | fused into `gui_profile_delete_ctor` |
| 0x002e2900 | `OnBeforeCreate__16GM_ProfileCreateFv` | fused into `gui_profile_create_ctor` |
| 0x002e2910 | `__dt__16GM_ProfileSelectFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002e2970 | `OnBeforeCreate__16GM_ProfileSelectFv` | fused into `gui_profile_select_ctor` |

## 14. Related

`structs/gui_screen.md` (the `GM_Menu` base, its 6-slot vtable, the framework API) · `structs/gui_widget.md` (`GM_Component` and all 53 widget classes) · `structs/racing_menus.md`, `structs/multiplayer.md` (the two other screen units) · `structs/garage.md` (the three garage screens of this unit) · `structs/inventory.md` · `structs/loadsave.md` · `basic/artifacts/player-interactions.md`.

## 15. Open items

None.
