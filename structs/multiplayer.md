# The PC multiplayer front end — `gui_mp_*`

**Created** 2026-08-24 11:05 +0200 · **Modified** 2026-11-16 10:50 +0100

Retail PC Mafia ships a complete DirectPlay front end: a mode menu, a session lobby with chat and an
eight-slot participant list, a connection-settings screen with TCP/IP and serial support, and a
driver-appearance picker. Nothing is stubbed; retail removes the door (§4). Every function here is
PC-only (§11).

| | |
|---|---|
| PC prefix | `gui_mp_`, `gui_race_champ_select_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_gui_menu_vt_mp_person_select` @ `0x0063d398`, `g_gui_menu_vt_mp_menu` @ `0x0063d3b4`, `g_gui_menu_vt_mp_lobby` @ `0x0063d3cc`, `g_gui_menu_vt_mp_settings` @ `0x0063d3e4`, `g_gui_menu_vt_race_champ_select` @ `0x0063d404`, `g_gui_menu_vt_mp_message_screen` @ `0x0063d41c`, 6 slots each |
| PC base class | `GM_Menu` (`structs/gui_screen.md`) |
| PC instance size | per screen, §1 |
| PC constructor | per screen, §3a |
| PC translation unit | `[0x00563b00, 0x00566250)` guard to guard; the driver-appearance picker lies in the unit of `structs/carcyclopedia.md` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Every screen inherits the `GM_Menu` layout, `0x48` bytes (`structs/gui_screen.md` §1). `gui_mp_menu_`, `gui_race_champ_select_` and `gui_mp_message_screen_` add no field: `operator_new(0x48)` at their callers. Each size below is read at the `operator_new` before the constructor.

### 1a. The driver-appearance picker — `gui_mp_person_select_`

```c
typedef struct S_gui_mp_person_select {                  // 0x68 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t selected_index;           // [H] the row in `tables\playerindex.def`; seeded from `*accepted_out`, 1 if that pointer is 0
    /* +0x4c */ I3D_frame * preview_a;             // [H] preview object A, I3D factory type 9, one facing; gets the model and `walk1.i3d`
    /* +0x50 */ I3D_frame * preview_b;             // [H] preview object B, the same model, the opposite facing
    /* +0x54 */ I3D_frame * camera;                // [H] the orbit camera, factory type 3, placed on a sphere of radius 8 by the tick
    /* +0x58 */ I3D_frame * fill_light;            // [H] the fill light, factory type 2, colour (1,1,1), parented to the scene root
    /* +0x5c */ float yaw_degrees;                 // [H] turntable yaw; the tick wraps it into [0, 360) and feeds `sin` / `cos`
    /* +0x60 */ float pitch_degrees;               // [H] camera pitch; the tick clamps it to 10..80
    /* +0x64 */ uint32_t * accepted_out;           // [H] where the accepted index goes: the lobby's `person_index`
} S_gui_mp_person_select;
```

The picker uses three base fields in its own way: `+0x04` holds the `missions\carcyclopedia\scene.i3d` scene object, created through the I3D driver factory (type `0xd`) and released by `gui_mp_person_select_on_close`; `+0x18` is the parent screen; the ctor clears bit `0x00010000` of the flag word `+0x40`.

### 1b. The lobby — `gui_mp_lobby_`

```c
typedef struct S_gui_mp_lobby {                  // 0x58 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t roster_refresh_ms;        // [H] countdown to the next participant-row refresh; 0 from the ctor, 1000 on each refresh
    /* +0x4c */ uint32_t garage_car;               // [H] the Garage button's car id: copied from `g_racing+0x64`, written by the garage chain through its out pointer
    /* +0x50 */ uint32_t person_index;             // [H] the Person button's row: copied from `g_net_session+0x228`, written by the picker through `accepted_out`
    /* +0x54 */ uint32_t added_ai_car;             // [H] the car id the Add AI Opponent chain returns, passed to `net_session_add_ai_racer`
} S_gui_mp_lobby;
```

### 1c. Multiplayer Settings — `gui_mp_settings_`

```c
typedef struct S_gui_mp_settings {                  // 0x168 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ S_gui_option_list providers;       // [H] combo `0xb`: the rows of `net_session_enumerate_providers`, value = type + 1000; a row sets `settings.provider_type`
    /* +0x58 */ S_gui_option_list provider_entries; // [H] combo `0xc`: what the INet driver lists for the chosen provider (its slots `+0x20`, `+0x24`), value = index + 1000
    /* +0x68 */ S_gui_option_list unread_vector_68; // [U] zeroed by the ctor, freed by the dtor; no reader (class census)
    /* +0x78 */ S_gui_option_list com_ports;       // [H] combo `0x32c`: `Com1`..`Com4`
    /* +0x88 */ S_gui_option_list baud_rates;      // [H] combo `0x32d`: 9600..115200; a row's value is the rate plus 1000
    /* +0x98 */ S_gui_option_list stop_bits;       // [H] combo `0x32e`: the stop-bit choices
    /* +0xa8 */ S_gui_option_list parities;        // [H] combo `0x32f`: `No parity`, `Odd`, `Even`, `Mark`
    /* +0xb8 */ S_gui_option_list flow_controls;   // [H] combo `0x330`: `No flow`, `RtSf flow`, `Dtr flow`, `RtSdTr flow`, `XonXoff flow`
    /* +0xc8 */ S_net_settings settings;           // [H] the screen's working copy of the `0x9c`-byte block (`structs/net_session.md` §1b)
    /* +0x164 */ uint8_t advanced_visible;          // [H] the advanced block is shown; 0 from the ctor
} S_gui_mp_settings;
```

**Eight option lists at stride `0x10`:** the ctor builds eight, one per combo the design can show, and seven are used. `unread_vector_68` follows `provider_entries` in the header order and has no combo: no `std_vector8_insert_at` call of the ctor targets it. `settings` is embedded whole because the screen edits a copy and applies it on OK.

### 1d. `S_gui_option_list` — one combo's rows

```c
typedef struct S_gui_option_list {                  // 0x10 bytes
    /* +0x00 */ uint8_t allocator;                 // [H] the vector's empty allocator byte; no reader
    /* +0x04 */ S_gui_option_row * begin;          // [H] the rows; `gui_combo_dropdown_screen_ctor` reads them at stride 8
    /* +0x08 */ S_gui_option_row * end;            // [H] one past the last row
    /* +0x0c */ S_gui_option_row * capacity_end;   // [H] one past the allocation
} S_gui_option_list;
```

A row is 8 bytes, `{label, value}`, stride proved by `end - begin >> 3` in `gui_combo_dropdown_screen_ctor`, which reads the label at the row's `+0x00`. The value is what the row stores into `settings` plus 1000: `gui_menu_set_component_text_from_option_list` shows the row whose value equals the setting plus 1000, so a baud row carries its rate plus 1000 (`0x2968` for 9600 .. `0x1c5e8` for 115200). `gui_mp_settings_ctor` fills seven lists through `std_vector8_insert_at`, ECX = `lea [this + N]`; `gui_widget_combo_range_ctor` builds a ninth list for its `'cmbs'` widget, whose rows carry a textdb id (`structs/gui_widget.md` §2b).

## 2. Field evidence

Cells come from a census of each screen's `this` accesses in `.text` plus a read of the functions named; a write through an out pointer is listed at the field it lands in.

### 2a. `S_gui_mp_person_select`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | selected_index | H | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | passed to `gui_mp_person_select_show_model` |
| +0x4c | preview_a | H | `gui_mp_person_select_ctor` | `gui_mp_person_select_ctor`, `gui_mp_person_select_show_model`, `gui_mp_person_select_tick` | - |
| +0x50 | preview_b | H | `gui_mp_person_select_ctor` | `gui_mp_person_select_ctor`, `gui_mp_person_select_show_model`, `gui_mp_person_select_tick` | - |
| +0x54 | camera | H | `gui_mp_person_select_ctor` | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | - |
| +0x58 | fill_light | H | `gui_mp_person_select_ctor` | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | - |
| +0x5c | yaw_degrees | H | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | `gui_mp_person_select_tick` | - |
| +0x60 | pitch_degrees | H | `gui_mp_person_select_ctor`, `gui_mp_person_select_tick` | `gui_mp_person_select_tick` | - |
| +0x64 | accepted_out | H | `gui_mp_person_select_ctor` | `gui_mp_person_select_tick` | - |

### 2b. `S_gui_mp_lobby`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | roster_refresh_ms | H | `gui_mp_lobby_ctor`, `gui_mp_lobby_tick` | `gui_mp_lobby_tick` | - |
| +0x4c | garage_car | H | `gui_mp_lobby_ctor`, `gui_mp_lobby_on_child_result`, `gui_mp_lobby_tick`, `gui_race_car_maker_on_close_child` | `gui_mp_lobby_on_child_result` | - |
| +0x50 | person_index | H | `gui_mp_lobby_ctor`, `gui_mp_lobby_tick`, `gui_mp_person_select_tick` | `gui_mp_lobby_on_child_result` | - |
| +0x54 | added_ai_car | H | `gui_mp_lobby_ctor`, `gui_race_car_maker_on_close_child` | `gui_mp_lobby_on_child_result` | - |

### 2c. `S_gui_mp_settings`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | providers | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0x58 | provider_entries | H | `gui_mp_settings_ctor`, `gui_mp_settings_on_exchange`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange` | - |
| +0x68 | unread_vector_68 | U | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | - | zeroed and freed; no reader (class census) |
| +0x78 | com_ports | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0x88 | baud_rates | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0x98 | stop_bits | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0xa8 | parities | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0xb8 | flow_controls | H | `gui_mp_settings_ctor`, `gui_mp_settings_dtor` | `gui_mp_settings_on_exchange`, `gui_mp_settings_write_widgets` | - |
| +0xc8 | settings | H | `gui_mp_settings_ctor`, `gui_mp_settings_on_change`, `gui_mp_settings_on_exchange`, `gui_mp_settings_read_widgets` | `gui_mp_settings_on_exchange`, `gui_mp_settings_resize_window`, `gui_mp_settings_tick`, `gui_mp_settings_write_widgets` | `+0xe8` = its `provider_type`, `+0xec` `provider_index`, `+0xf0` `ip_address`, `+0x110` `port`, `+0x130` `serial_port`, `+0x150`..`+0x160` the five serial selections |
| +0x164 | advanced_visible | H | `gui_mp_settings_ctor`, `gui_mp_settings_set_advanced_visible` | `gui_mp_settings_on_exchange`, `gui_mp_settings_resize_window`, `gui_mp_settings_set_advanced_visible`, `gui_mp_settings_tick` | - |

### 2d. `S_gui_option_list`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | allocator | H | `gui_mp_settings_ctor`, `gui_widget_combo_range_ctor` | - | stored from an argument's low byte; no reader (class census) |
| +0x4 | begin | H | `gui_mp_settings_ctor` (and by address, `std_vector8_insert_at`), `gui_mp_settings_dtor`, `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor`, `gui_widget_combo_range_dtor` | `gui_combo_dropdown_screen_ctor`, `gui_menu_set_component_text_from_option_list`, `gui_mp_settings_ctor` (by address, `std_vector8_insert_at`), `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor`, `gui_widget_combo_range_dtor` | - |
| +0x8 | end | H | `gui_mp_settings_ctor` (and by address, `std_vector8_insert_at`), `gui_mp_settings_dtor`, `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor`, `gui_widget_combo_range_dtor` | `gui_combo_dropdown_screen_ctor`, `gui_menu_set_component_text_from_option_list`, `gui_mp_settings_ctor` (by address, `std_vector8_insert_at`), `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor` | - |
| +0xc | capacity_end | H | `gui_mp_settings_ctor` (and by address, `std_vector8_insert_at`), `gui_mp_settings_dtor`, `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor`, `gui_widget_combo_range_dtor` | `gui_mp_settings_ctor` (by address, `std_vector8_insert_at`), `gui_mp_settings_on_exchange`, `gui_widget_combo_range_ctor` | - |

## 3. Method family

The 28 rows cover the six screens and the unit. All are PC-only except `gui_menu_scalar_deleting_dtor_common`, a framework row graded in `structs/gui_screen.md`. The drivers that open these screens, `net_multiplayer_run` and `race_menu_run`, are rows of `structs/race_session.md`; the provider helpers around `0x00597190` are rows of `structs/net_session.md`. Slot defaults are in §10.

### 3a. Construction & destruction

#### The unit

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00563b00 | `crt_static_init_guard_00563b00` | the translation unit | - | `.CRT$XCU` guard of this unit's static destructor |
| 0x00563b20 | `crt_atexit_register_00563b20` | the translation unit | - | registers the unit's static destructor |
| 0x00578790 | `gui_menu_scalar_deleting_dtor_common` | shared | - | slot 0 of five screens here: calls `gui_menu_dtor`, frees on request; the address lies in the unit of `structs/game_menus.md` |

#### `gui_mp_person_select_` — the driver-appearance picker

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00562fa0 | `gui_mp_person_select_ctor` | instance | - | design `0x2c`: loads the showroom scene, the two previews, camera and light, one row per `playerindex.def` record |

#### `gui_mp_menu_` — the Multiplayer menu

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00563b30 | `gui_mp_menu_ctor` | instance | - | design `0x28`: the mode items, enabled from the supported-mode mask; disables `0x6f` |

#### `gui_mp_lobby_` — the lobby

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00563ee0 | `gui_mp_lobby_ctor` | instance | - | design `0x2a`, gate `1 << mode`, shifted by 8 on a client (§7) |

#### `gui_mp_settings_` — Multiplayer Settings

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00564f20 | `gui_mp_settings_ctor` | instance | - | design `0x2b`: copies the settings, fills the provider and serial combos |
| 0x00565610 | `gui_mp_settings_scalar_deleting_dtor` | instance | - | slot 0 |
| 0x00565630 | `gui_mp_settings_dtor` | instance | - | frees the eight lists, then `gui_menu_dtor` |

#### `gui_race_champ_select_` — the championship picker

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00565f00 | `gui_race_champ_select_ctor` | instance | - | design `0x17`: clones the round template `0x1b8` per championship |

#### `gui_mp_message_screen_` — the dead-item screen

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00566190 | `gui_mp_message_screen_ctor` | instance | - | design `0x23`, opened only by the disabled item `0x6f` |

### 3b. Accessors

No accessor: every field is reached from a constructor, a tick or a child handler.

### 3c. Behaviour

#### `gui_mp_person_select_` — the driver-appearance picker

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x005634b0 | `gui_mp_person_select_on_close` | - | slot 2: releases the scene object, then `gui_menu_destroy` |
| 0x005634d0 | `gui_mp_person_select_tick` | - | slot 5: steps the row, turns and tilts the preview, writes the accepted index |
| 0x005639e0 | `gui_mp_person_select_show_model` | - | loads a row's model into both previews, binds `walk1.i3d`, moves the selection to row `index + 1000` |

#### `gui_mp_menu_` — the Multiplayer menu

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00563c20 | `gui_mp_menu_refresh_mode_items` | - | slot 3: recomputes the mode items from the mask; disables `0x6f` |
| 0x00563cc0 | `gui_mp_menu_tick` | - | slot 5: writes the session mode and returns its code, or opens a child (§5) |

#### `gui_mp_lobby_` — the lobby

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00564110 | `gui_mp_lobby_on_child_result` | - | slot 3: applies a closed child's result to the session and the widgets |
| 0x00564320 | `gui_mp_lobby_tick` | - | slot 5: refreshes the rows, sends chat, opens the pickers, starts |

#### `gui_mp_settings_` — Multiplayer Settings

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00565760 | `gui_mp_settings_on_exchange` | - | slot 1: rebuilds the provider combo for the chosen type, reseeds every widget |
| 0x00565a70 | `gui_mp_settings_read_widgets` | - | copies the three edit fields into `settings` |
| 0x00565ac0 | `gui_mp_settings_write_widgets` | - | shows `settings` in every widget |
| 0x00565bc0 | `gui_mp_settings_on_change` | - | slot 3: stores a combo's row into `settings` |
| 0x00565d00 | `gui_mp_settings_tick` | - | slot 5: the OK arm `0x26d` reads the widgets and calls `net_session_settings_apply` |
| 0x00565db0 | `gui_mp_settings_set_advanced_visible` | - | shows or hides the advanced widgets, stores the flag |
| 0x00565e90 | `gui_mp_settings_resize_window` | - | resizes the frame for the provider type and the flag |

#### `gui_race_champ_select_` — the championship picker

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x005660f0 | `gui_race_champ_select_tick` | - | slot 5 of two vtables: `0x1b8`..`0x1cb` to `racing_set_current_champ`, `0x1a7` EXIT |

### 3d. Free helpers

#### The unit

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00566200 | `vector_dispose_at_0x04` | - | shared STL helper: frees the vector at `+0x04`, zeroes the three pointers; 531 references, no dataref |
| 0x00566220 | `copy_range_stride_0x8` | - | shared STL helper: copies a range of 8-byte rows; 21 references |

## 4. What it is, and how to reach it

Retail removes the **door**, in the same belt-and-braces way it removed Racing
(`basic/artifacts/racing.md` §3):

* `tables/menu.def`'s main-menu design (`0x06`) has **no `meti` record `0xf1b9c`**, and
* `gui_main_menu_ctor` calls `gui_menu_set_component_enabled(0xf1b9c, 0)` and
  `gui_menu_delete_component(0xf1b9c)` at construction time, so the item is
  deleted at runtime if it ever reappears.

**`0xf1b9c` was the "Multiplayer" item**, on three independent witnesses:

| witness | what it says |
|---|---|
| `gui_main_menu_tick` (`0x00576a20`) | `case 0xf1b9c:` returns menu_choice **`0x14`**, and `engine_main` case `0x14` calls `net_multiplayer_run` |
| `g_menu_camera_position_table` (`0x0064daf0`) | pairs `0xf1b9c` with the `00menu` camera frame **`multiplayer_pos`** |
| `textdb` | **104 = "Multiplayer" / "Multijoueur" / "Mehrspieler" / "Multiplayer"**, unused, sitting between 103 "Load Game" and 105 "Tutorial" |

### 4a. The reachable route — through the Racing menu

`net_multiplayer_run` has a **second** caller: `race_menu_run` case `0x2e`, reached from the
Racing menu's own item **`0xf1ccd`**, which is labelled `textdb 413 = "Multiplayer"` and
**does ship in retail `menu.def`** (design `0x16`: *Racing: / Single Race / Championship /
Multiplayer / Exit*).

So in a build where the Racing menu is reachable — i.e. with the community racing mod,
which restores main-menu item `0xf1b9b` — the path is:

```
Main Menu → Racing → Multiplayer → Host SingleRace → the lobby → "Person:"
```

**The racing mod's own patch touches this id.** It redirects `gui_main_menu_ctor`'s two
`push 0xf1b9b` operands to `push 0xf1ccd` (`basic/artifacts/racing.md` §4) — the Racing menu's
Multiplayer item. The redirect is nevertheless safe, because
`gui_menu_set_component_enabled` and `gui_menu_delete_component` walk only the
CURRENT screen's child vector (`screen+0x20`/`+0x24`) and `gui_main_menu_ctor` is building the
main menu — so both redirected calls are no-ops and the Racing menu's Multiplayer entry is
left untouched.

### 4b. The runtime gate — DirectPlay

`net_multiplayer_run` calls `net_session_init_provider` first. That function creates the INet driver
through the `inet.dll` import `_CreateINetBase@4`; on failure it zeroes both the driver
pointer (`g_net_session+0x10c`) and the session state (`g_net_session+0x100`, the enum of
`structs/net_session.md` §1a) and stores
textdb **900** in `+0x1fc`. `net_multiplayer_run` then refuses with a message box —
textdb **907 "Network error!"** over **900 "Error in Direct Play initialization!"** — and
returns to the menu.

⇒ On a modern Windows install, where DirectPlay is an optional feature that is off by
default, the menu item is reachable and the mode is not. That is a *runtime* block, not a
data one.

## 5. The Multiplayer menu — design `0x28`, vtable `0x0063d3b4`

| Item id | textdb | label | what `gui_mp_menu_tick` does |
|---|---|---|---|
| `0x65` | 601 | **Host Deathmatch** | `session+0x38 = 1`, return `0x37` |
| `0x66` | 602 | **Host Team Deathmatch** | `session+0x38 = 2`, return `0x37` |
| `0x67` | 603 | Host SingleRace | `session+0x38 = 3`, return `0x37` |
| `0x68` | 604 | Host Championship | `session+0x38 = 4`, return `0x37` |
| `0x6e` | 605 | Join existing game | return `0x36` |
| `0x6f` | - | *(no record anywhere)* | opens `gui_mp_message_screen_ctor` (§9) |
| `0x78` | 606 | Settings | opens `gui_mp_settings_ctor` (§8) |
| `0x82` | 607 | Back to Main Menu | return `0x38` |

**`0x65` and `0x66` — the two DEATHMATCH modes — have menu logic but no game behind them,
and no `menu.def` record in retail.** Both `gui_mp_menu_ctor` and
`gui_mp_menu_refresh_mode_items` compute their enable state from bits 1..4 of the session's
supported-mode mask `session+0x4c`, and `gui_mp_menu_tick` writes the mode. The labels ship
in all four languages.

Past the menu, nothing plays a deathmatch. `net_host_session_start` names the session
`"Mafia Deathmatch"` or `"Mafia Team Deathmatch"` and creates it, building no `g_racing` (modes
3 and 4 build one), and the lobby opens. On the lobby's start code `0x3d`,
`net_multiplayer_run` launches a game only for mode 3 (`race_run`) or mode 4 (the championship
loop); any other mode falls through to the exit tail, which returns to the menu scene. Every
other reader of the mode tests for 3 or 4, gates the lobby design on it, or only copies it (the
roster message, the wait for a non-zero mode). So a deathmatch session can be hosted and its
lobby reached, but no game ever starts.

The `0x36` (Join) return is handled by `net_multiplayer_run`, which opens the "Select session:"
browser `gui_session_list_ctor` (`0x0057b150`, design `0x2f`, textdb 610; `structs/game_menus.md`).

The whole mask is forced to zero unless `net_session_selected_provider_is_valid` confirms the
configured connection provider still exists, so a machine with no usable provider gets every
host and join item greyed out.

## 6. The driver-appearance picker — design `0x2c`, vtable `0x0063d398`

This class lives inside the `GM_Carcyclopedia` translation unit; its fields are in §1a. It is
**not** a car viewer: it selects the **character model your driver wears**, out of
`tables\playerindex.def`.

### 6a. What is on screen

The ctor loads the showroom set `missions\carcyclopedia\scene.i3d`, then clones the
design's `seli` list-row template (id 1000, FourCC `'seli'`) once per `playerindex.def`
record, labelling each row with that record's model name, and grows the enclosing scroll
window (`swnd` id 4, 240×528) by the accumulated row height.

`gui_mp_person_select_show_model(this, index)` — `__thiscall`, `ret 4`, the second argument
invisible in the decomp — resolves the index through `car_database_get_player_model`, loads
the model into **both** preview objects, binds the shared `walk1.i3d` clip at weight 1.0
through frame vtable `+0x60`, and moves the list selection to row `index + 1000`.

### 6b. Why `walk1.i3d` is not an anomaly

`car_database_get_player_model` reads `G_CarDatabase+0x0c`, the buffer of
**`tables\playerindex.def`** — 2376 bytes / 0x24 = **66 records** of
`{char model[0x20]; u32 civil_mask}`, every one a `civilNN.i3d` **character** model
(`structs/car_database.md` §1). The previewed model is a person, so a human walk clip is
exactly right.

`car_database_get_player_model` is named for what it reads: a character model for a driver
(PS2 name and consumer in §11).

The chosen index lands in **`g_net_session+0x228`**, the session's driver-model field.

## 7. The lobby — design `0x2a`, vtable `0x0063d3cc`

This is where a session is configured and started.

**One design serves four layouts.** `gui_mp_lobby_ctor` builds design `0x2a` with the gate
`1 << session_mode`, shifted left by 8 when this machine is a CLIENT
(`g_net_session+0x200 == 0`), so the four `gate` records in the shipped file — `8`, `0x10`,
`0x800`, `0x1000` — select host/client × single-race/championship. A fifth gate `0x1010`
carries the row labels and `0xffff` the chat panel.

| Component | id | role |
|---|---|---|
| participant rows | `100`..`0x6b` | nine `netp` rows; `gui_mp_lobby_tick` fills eight from `g_net_session+0x268` (humans) then `+0x294` (AI slots) |
| chat log | `200`..`0x6b`+ | seven `stxt` lines, textdb 702 `<no messages>` when empty |
| chat input | `10` | an `edtl`; ENTER sends it and clears it |
| Circuit / Championship | `11000` (`0x2af8`) | textdb 660 / 665 |
| **Garage:** | `0x2af9` | textdb 661 — opens the car picker `gui_race_car_maker_ctor` (`structs/racing_menus.md`), result into `g_racing+0x64` |
| **Person:** | `0x2afa` | textdb 662 — opens §6, result into `g_net_session+0x228` |
| laps − / + | `0x2afb` / `0x2b05` | steps `g_racing+0x27c`, clamped 1..99; shown on `ntxt` `0x3f5` |
| Direction: | `0x2afc` | flips `g_racing+0x2b4`; textdb 688 "Normal" / 689 "Reverse" |
| circuit / championship picker | `0x2afd` | opens §9 |
| damage | `0x2afe` | `racing_set_damage_enabled`, `g_racing+0x2b7` |
| Cancel / Start / Race / Ready | `0x1e` / `0x14` / `0x28` | textdb 690 / 692 / 693 / 691 |
| Add AI Opponent | `0x32` | opens the car picker, then `net_session_add_ai_racer` |
| Generate All Opponents | `0x3c` | enabled while `g_racing+0x64 != -1` |

The Garage and Person buttons are greyed once the local player has readied up
(`g_net_session+0x252`).

## 8. Multiplayer Settings — design `0x2b`, vtable `0x0063d3e4`

The only class in these two units with its own scalar deleting destructor and its own destructor, because it owns eight option lists (§1c).

The combo contents are the module's own string literals at `0x0064d9a0..0x0064da68`:
`Com1`..`Com4`, `9600`/`14400`/`19200`/`38400`/`56000`/`57600`/`115200` (each row's value
the rate plus 1000, §1d), `No parity` / `Odd` / `Even` / `Mark`, and `No flow` / `RtSf flow` /
`Dtr flow` / `RtSdTr flow` / `XonXoff flow`.

`gui_mp_settings_tick`'s OK arm (`0x26d`) reads every widget back, applies the block to the
session through `net_session_settings_apply`, and calls the settings-committed hook.

## 9. The championship picker and the dead-item screen

`gui_race_champ_select_ctor` (design `0x17`, vtable `0x0063d404`) clones design `0x17`'s
`rari` round template `0x1b8` ("ROUND 1", textdb 440) once per entry of the `g_racing+0x1c`
championship vector; `gui_race_champ_select_tick` maps `0x1b8`..`0x1cb` to
`racing_set_current_champ` and `0x1a7` to EXIT.

`gui_mp_message_screen_ctor` (design `0x23`, vtable `0x0063d41c`) is the odd one out. It
adds no members and overrides nothing — vtable `0x0063d41c` is **byte-identical** to
`0x0063d404`, so it inherits the championship tick, which handles none of design `0x23`'s
component ids. Design `0x23` in the shipped file is the bonus-car award layout (textdb 780
"You have won a bonus car"), but the *working* bonus-car screen is a different class,
`gui_race_bonus_car_ctor` (vtable `0x0063d56c`, four parameters, called from `racing_end_race` case
`0x33`). Its only entry point is the Multiplayer menu's item `0x6f`, which has no record in
any design and is disabled unconditionally. As shipped, the screen closes on ESC and does
nothing else.

## 10. The `GM_Menu` framework

The slot map and the design table belong to `structs/gui_screen.md`. What the screens here rely on: slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, slot 1 forwards to the parent, slot 2 destroys, slot 3 hears a closed child, slot 4 pops the stack and slot 5 is each screen's tick; `screen+0x18` is the **parent**, since `gui_menu_close` assigns it to `g_gui_active_screen` and every constructor here is called as `ctor(g_gui_active_screen, …)`. Eight design ids are verified one-for-one against the shipped file: `0x06` main menu, `0x0d` carcyclopedia, `0x16` racing menu, `0x17` championship, `0x28` multiplayer menu, `0x2a` lobby, `0x2b` multiplayer settings, `0x2c` driver picker.

## 11. PC vs PS2

**Everything here is PC-only.** The PS2 build ships no networking: `Game.exe` imports
`_CreateINetBase@4` from `inet.dll`, and the PS2, having no DLLs, would have had to compile
that in — it did not (`structs/net_session.md`). The PS2's `GM_*` roster
(109 classes) contains **no** multiplayer menu, no lobby, no connection-settings screen and
no character picker; `GM_NetPlayer` exists as a class with a `G_Factory<>` but nothing
drives it.

The framework default `gui_menu_on_exchange_forward_to_parent` has a PS2 twin, recorded in `structs/gui_screen.md`: `OnExchange__7GM_MenuFv` is COMDAT-emitted into `g_m_carc.cpp` on both builds — 16
bytes each, the same forward through a member pointer, different member offset (PS2
`+0x28`, PC `+0x18`) and different slot index (PS2 5, PC 1), so the mapping is STRUCTURAL.

`car_database_get_player_model` ↔ `GetPlayerModel__13G_CarDatabaseFUi`. Its PS2 consumer is
`GameInit__10C_opponentFP17I3D_animation_setUi` — a race opponent building its **driver's**
character model from `+0x17c`, chosen by code. Only the PC lets a human choose it.

## 12. Related

`structs/gui_screen.md` (the `GM_Menu` base, the vtable map, the design table) · `structs/gui_widget.md` (the widgets) · `structs/carcyclopedia.md` (the unit that holds the picker) · `structs/car_database.md` (`playerindex.def`) · `structs/net_session.md` (the session and `S_net_settings`) · `structs/race_session.md` (the drivers) · `structs/racing_menus.md` (the car picker, the other screen unit) · `basic/artifacts/racing.md`.

## 13. Open items

| item | state |
|---|---|
| in game (modded install, DirectPlay enabled) | confirm that Racing → Multiplayer opens the mode menu, that Host SingleRace reaches the lobby, and that the **Person:** button lists 66 `civilNN.i3d` rows with a walking preview |
