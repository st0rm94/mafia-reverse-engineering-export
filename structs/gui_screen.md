# `GM_Menu` — the GUI screen framework

**Created** 2026-08-16 18:02 +0200 · **Modified** 2026-11-16 10:50 +0100

A screen (window) of the menu system. Screens form a **stack**: each one holds its parent at `+0x18`,
and the top of the stack is the global `g_gui_active_screen`. A screen owns a vector of `GM_Component`
**widgets** and dispatches draw, layout and input to them. `GM_Menu` is the base class of every screen
in the game; the widget classes in the same translation unit are documented in
`structs/gui_widget.md`.

| | |
|---|---|
| PC prefix | `gui_menu_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_gui_menu_vt_base` @ `0x0063d584`, 6 slots (§8) |
| PC base class | - |
| PC instance size | `0x48` (`gui_notice_panel_present_once` allocates a field-less screen with `operator_new(0x48)`; every subclass's first own field is `+0x48`) |
| PC constructor | `gui_menu_ctor` @ `0x0056ae50` |
| PC translation unit | `[0x0056adc0, 0x00575c20)` guard to guard (§7), plus `gui_menu_on_exchange_forward_to_parent` at `0x005634a0` and `gui_menu_scalar_deleting_dtor_common` at `0x00578790` |
| PS2 class | `GM_Menu` (tag `7GM_Menu`), 60 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct GM_Menu {                  // 0x48 bytes; base of every screen
    /* +0x00 */ void *vtable;             // [H] 6 slots (§8)
    /* +0x04 */ I3D_scene *own_scene;     // [H] the screen's own 3-D scene, 0 = none; inherited from the parent
    /* +0x08 */ float rect_x;             // [H] screen rectangle x, stored 0; no reader (class census)
    /* +0x0c */ float rect_y;             // [H] screen rectangle y, stored 0; no reader (class census)
    /* +0x10 */ float rect_w;             // [H] screen rectangle: the live width in pixels
    /* +0x14 */ float rect_h;             // [H] screen rectangle: the live height in pixels
    /* +0x18 */ GM_Menu *parent;          // [H] the screen below on the stack, 0 for a root
    /* +0x1c */ std_vector widgets;       // [H] GM_Component * elements; allocator byte, begin +0x20, end +0x24, cap +0x28
    /* +0x2c */ GM_Component *focused;    // [H] the focused widget, or 0
    /* +0x31 */ u8 pointer_steers;        // [H] the pointer, not the keyboard, picks the focused widget
    /* +0x32 */ u8 pointer_muted;         // [H] mouse input ignored until the left button is released
    /* +0x34 */ int mouse_x;              // [H] last accepted cursor x in pixels
    /* +0x38 */ int mouse_y;              // [H] last accepted cursor y in pixels
    /* +0x3c */ u32 prev_buttons;         // [H] the mouse button word of the previous tick
    /* +0x40 */ u32 flags;                // [H] screen flag word, seeded 0x10001 (§2a)
    /* +0x44 */ u32 id;                   // [H] the id handed back to the parent on close
} GM_Menu;
```

`widgets` is one `std::vector`, as every project vector: its first byte is the MSVC allocator object,
which `gui_menu_ctor` copies from a stack byte. `+0x30` and `+0x33` have no access in any screen
function and are alignment padding.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x00` | `vtable` | H | `gui_menu_ctor`, `gui_menu_dtor`, every screen constructor | `gui_menu_close`, `gui_menu_destroy`, `gui_notice_panel_tick` | every screen constructor installs its own table after `gui_menu_ctor` |
| `+0x04` | `own_scene` | H | `gui_menu_ctor` (0, then the parent's value), `gui_mp_person_select_ctor`, `gui_mp_person_select_on_close` | `gui_menu_execute_modal`, `gui_mp_person_select_ctor`, `gui_mp_person_select_on_close` | ¹ |
| `+0x08` | `rect_x` | H | `gui_menu_build_components_from_design` | - | stored as 0 beside `rect_w` / `rect_h`; no GUI receiver loads it, so no reader (class census) |
| `+0x0c` | `rect_y` | H | `gui_menu_build_components_from_design` | - | stored as 0 beside `rect_w` / `rect_h`; no GUI receiver loads it, so no reader (class census) |
| `+0x10` | `rect_w` | H | `gui_menu_build_components_from_design` | `gui_menu_execute_modal` | the engine's width; `own_scene`'s camera aspect is `rect_w / rect_h` |
| `+0x14` | `rect_h` | H | `gui_menu_build_components_from_design` | `gui_menu_execute_modal` | the engine's height |
| `+0x18` | `parent` | H | `gui_menu_ctor` | `gui_menu_close`, `gui_menu_draw`, `gui_menu_execute_modal`, `gui_menu_on_exchange_forward_to_parent`, `gui_notice_panel_tick`, `carcyclopedia_on_exchange` | every constructor is called as `ctor(g_gui_active_screen, …)` |
| `+0x1c` | `widgets` | H | `gui_menu_ctor`, `gui_menu_dtor`, `gui_menu_build_components_from_design`, `gui_menu_add_component_from_template`, `gui_menu_clone_component_under`, `gui_menu_delete_component`, `gui_menu_free_components`, `gui_menu_destroy` | every widget walker of §3c | the allocator byte `+0x1c` has one write in the framework (`mov byte ptr [esi + 0x1c], cl` in `gui_menu_ctor`) and no read |
| `+0x2c` | `focused` | H | `gui_menu_ctor`, `gui_menu_tick`, `gui_menu_update_hover_focus`, `gui_menu_activate_by_id`, `gui_menu_activate_by_id_or_prev`, `gui_menu_clear_focus`, `gui_menu_refocus_if_disabled`, `gui_menu_delete_component`, `gui_menu_build_components_from_design`, `gui_combo_dropdown_screen_ctor`, `gui_race_car_model_ctor` | those, `gui_menu_execute_modal`, `gui_free_ride_mode_tick`, `gui_garage_select_tick`, `gui_inventory_tick`, `gui_load_game_tick` | `gui_menu_tick` moves widget flag `0x10` with it; `gui_menu_build_components_from_design` seeds it with `gui_menu_find_next_selectable_in_order(1, 0)` |
| `+0x31` | `pointer_steers` | H | `gui_menu_ctor` (1), `gui_menu_tick`, `gui_menu_on_close_child` (0), `gui_menu_build_components_from_design` (0), `gui_load_game_tick`, `gui_session_list_ctor` | `gui_menu_tick` | ² |
| `+0x32` | `pointer_muted` | H | `gui_menu_ctor` (1), `gui_menu_on_close_child` (1), `gui_menu_tick` (0), `gui_inventory_ctor`, `gui_mp_settings_ctor`, `gui_mp_settings_tick` | `gui_menu_tick` | ² |
| `+0x34` | `mouse_x` | H | `gui_menu_tick`, `gui_menu_on_close_child`, `gui_menu_execute_modal` | `gui_menu_tick`, `gui_menu_draw`, `gui_menu_update_hover_focus`, `carcyclopedia_tick` | `gui_menu_draw` converts it with `fild` for `hud_draw_mouse_cursor`, which is what proves integer pixels |
| `+0x38` | `mouse_y` | H | `gui_menu_tick`, `gui_menu_on_close_child`, `gui_menu_execute_modal` | `gui_menu_tick`, `gui_menu_draw`, `gui_menu_update_hover_focus`, `carcyclopedia_tick` | as `mouse_x` |
| `+0x3c` | `prev_buttons` | H | `gui_menu_ctor` (0), `gui_menu_tick`, `gui_menu_build_components_from_design` (0) | `gui_menu_tick`, `gui_menu_draw` | `gui_menu_tick` stores its button argument on every path that reaches its tail; bit 0 = left button |
| `+0x40` | `flags` | H | `gui_menu_ctor`, 41 screen functions | `gui_menu_draw`, `gui_menu_execute_modal` | §2a |
| `+0x44` | `id` | H | `gui_menu_ctor` | `gui_menu_close`, `gui_notice_panel_tick`, 39 screen functions | the constructor's second parameter |

¹ `gui_mp_person_select_ctor` creates a driver object of type `0xd` and stores it here, and registers
`gui_menu_i3d_draw_callback` on it through the object's vtable `+0x50`. While the active screen has
one, `gui_menu_execute_modal` renders it (camera at `+0x17c`, the slots `+0x54` and `+0x104`) in place
of the mission scene. It is the same call shape the mission scene gets at `g_act_mission + 0x10`.

² `pointer_steers`: `gui_menu_tick` sets it when the Manhattan distance the cursor moved exceeds 10
pixels and on a left-button press, and clears it when a key moves the focus; while it is set the tick
hit-tests the widgets under the cursor. `pointer_muted`: `gui_menu_tick` skips the move test and the
pointer branch while it is set, and clears it on the first tick with the left button up. A screen is
therefore built with the pointer dormant, and a child's close (`gui_menu_on_close_child`) puts the
parent in the same state with the cursor resampled.

### 2a. `flags` (`+0x40`)

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | seeded; tested by no screen function and by none of the 47 widget Draw functions | `gui_menu_ctor` |
| `0x10` | a parent drawn under this screen keeps the focus highlight of its `'item'` / `'iten'` / `'rari'` rows | set with `0x8000` only by `gui_combo_dropdown_screen_ctor`; `gui_widget_item_draw`, `gui_widget_item_num_draw` and `gui_widget_race_round_draw` draw the focused row highlighted only while their mask carries it |
| `0x8000` | a parent drawn under this screen keeps the active colours of its `'wind'` / `'wint'` windows | set with `0x10` only by `gui_combo_dropdown_screen_ctor`; `gui_widget_window_draw` and `gui_widget_window_titled_draw` fill with `0xffffffff` while their mask carries it, `0xff7f7f7f` without |
| `0x10000` | draw the parent first | seeded by `gui_menu_ctor`, tested by `gui_menu_draw`; a full-screen constructor clears it |
| `0x40000` | run one extra tick of the screen at the head of the modal loop | `gui_menu_execute_modal` |
| `0x80000` | the screen steers the `00menu` camera from its focused widget's id | `gui_menu_execute_modal` (`g_menu_camera_item_id`) |
| `0x100000` | poll the game input while open; a non-zero action value enters the key path as scancode 1 | `gui_menu_execute_modal` (`input_update`, `input_get_action_values`) |
| `0x200000` | draw only while `racing_get_results_visibility` allows it | `gui_menu_draw` |
| `0x400000` | do not play the open sound when the modal loop loads it | `gui_menu_execute_modal` |
| `0x800000` | draw the screen alone, without rendering the mission scene | `gui_menu_execute_modal` |

**A screen's bits filter what its parent shows.** `gui_menu_draw(mask)` draws the parent first, with
`flags & mask`, when the screen has a parent and flag `0x10000`; then it hands every visible widget the
mask in slot 2. The top-level draw passes `0xffffffff` (as `gui_load_progress_present` does), so a
screen's own bits never reach its own widgets. That is why `0x10` and `0x8000` keep a menu lit under its
open drop-down, while under any other child its windows dim and its focus highlight goes out.

## 3. Method family

All 60 PS2 methods are placed: 31 on PC functions, 2 as `inlined` rows and 27 in the PS2-only table of
§11b. 29 of the 62 rows carry no `GM_Menu` method. The default of slot 1 lives in the Carcyclopedia
unit and the common deleting destructor in the next translation unit. `gui_race_root_ctor` and
`carcyclopedia_on_exchange` carry `GM_Menu` twin cells in the registry, but they build or override a
subclass and are listed in `structs/racing_menus.md` and `structs/carcyclopedia.md`.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0056adc0 | `crt_static_init_guard_0056adc0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer guard |
| 0x0056ade0 | `crt_atexit_register_0056ade0` | the translation unit | - | - | - | registers the guard's teardown |
| 0x0056adf0 | `thunk_gui_scale_static_init` | `g_gui_scale_x` / `_y` | - | - | - | `.CRT$XCU` entry of the next row |
| 0x0056ae00 | `gui_scale_static_init` | `g_gui_scale_x` / `_y` | - | - | STRUCTURAL | both = 1.0 |
| 0x0056ae20 | `thunk_gui_mouse_pos_static_init` | `g_gui_mouse_x` / `_y` | - | - | - | `.CRT$XCU` entry of the next row |
| 0x0056ae30 | `gui_mouse_pos_static_init` | `g_gui_mouse_x` / `_y` | - | - | STRUCTURAL | 400.0 / 300.0, the centre of the design space (§4) |
| 0x0056ae50 | `gui_menu_ctor` | instance | `__ct__7GM_MenuFv` | 0x002e4220 | CONFIRMED | `(parent, id)`: stores both, inherits `own_scene`, seeds the pointer state and flags `0x10001`, plays the open sound |
| 0x0056aef0 | `gui_menu_scalar_deleting_dtor` | instance | - | - | STRUCTURAL | slot 0 of the base table |
| 0x0056af10 | `gui_menu_dtor` | instance | `__dt__7GM_MenuFv` | 0x002e4260 | CONFIRMED | frees the widgets and the vector's storage |
| 0x00578790 | `gui_menu_scalar_deleting_dtor_common` | instance | - | - | STRUCTURAL | slot 0 of the screens that add no destructor work; calls `thunk_gui_menu_dtor` |
| 0x005787b0 | `thunk_gui_menu_dtor` | instance | - | - | - | jump to `gui_menu_dtor` |

### 3b. Accessors

No function of the class is a single load or store of one `this` field: every `gui_menu_get_*` and
`gui_menu_set_*` walks the widget vector to find its component by id, so they are 3c rows.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005634a0 | `gui_menu_on_exchange_forward_to_parent` | `OnExchange__7GM_MenuFv` | 0x002c1b60 | STRUCTURAL | slot 1 default: calls the parent's slot 1 |
| 0x0056c1d0 | `gui_menu_clone_component_under` | `PridejComponentPod__7GM_MenuFUiUiUiUiUiUi` | 0x002e60c0 | CONFIRMED | a new widget of a FourCC, copied from an existing one and placed below it |
| 0x0056c600 | `gui_menu_add_component_from_template` | `PridejComponent__7GM_MenuFfUiUiUiUiUiUi` | 0x002e5e20 | CONFIRMED | a new widget of a FourCC, from an explicit template |
| 0x0056cb80 | `gui_menu_layout_add_vertical_space` | - | - | - | grows one widget's height and moves up to three widgets down by a scaled delta; returns the advanced y |
| 0x0056cc20 | `gui_menu_delete_component` | `DeleteComponent__7GM_MenuFUi` | 0x002e4a90 | CONFIRMED | destroys one widget by id, clears `focused` when it was that one, compacts the vector |
| 0x0056ccc0 | `gui_menu_execute_modal` | `ExecuteMenu__7GM_MenuFP7GM_Menubbb` | 0x002e6350 | CONFIRMED | the modal loop: makes the screen active, ticks it and everything under it until the stack empties, tears the stack down |
| - | - | `TickCamera__7GM_MenuFUi` | 0x002e6e90 | inlined | the `00menu` camera walk of `gui_menu_execute_modal` |
| - | - | `SetCameraTargetID__7GM_MenuFUi` | 0x002e6920 | inlined | the camera-target lookup of `gui_menu_execute_modal` (`g_menu_camera_position_table`) |
| 0x0056e500 | `gui_menu_build_components_from_design` | `CreateCompsByDef__7GM_MenuFPc` | 0x002e4400 | STRUCTURAL | recomputes the scale globals and the screen rectangle, instantiates one `menu.def` design (§6), seeds `focused` |
| 0x0056e850 | `gui_menu_free_components` | - | - | - | destroys every widget and empties the vector, keeping its storage |
| 0x0056e8a0 | `gui_menu_destroy` | `Destroy__7GM_MenuFv` | 0x002e4a10 | CONFIRMED | slot 2: destroys every widget, then deletes the screen through slot 0 |
| 0x0056e8f0 | `gui_menu_on_close_child` | `OnCloseChild__7GM_MenuFP7GM_MenuUiUi` | 0x002c1b90 | STRUCTURAL | slot 3: mutes the pointer and resamples the cursor |
| 0x0056e930 | `gui_menu_close` | `MenuClose__7GM_MenuFUi` | 0x002e5210 | STRUCTURAL | slot 4 `(unread, result)`: pops the stack, calls the parent's slot 3 `(id, result)`, then its own slot 2 |
| 0x0056e960 | `gui_menu_draw` | `Draw__7GM_MenuFUi` | 0x002e4be0 | CONFIRMED | draws the parent first under flag `0x10000`, every visible widget, and the cursor when it is the active screen |
| 0x0056ea70 | `gui_menu_tick` | `Tick__7GM_MenuFiiiUiUiUiUi` | 0x002e4d50 | CONFIRMED | slot 5: blink phase, widget layout pass, pointer and key routing, focus move; returns the activated widget id |
| 0x0056eeb0 | `gui_menu_update_hover_focus` | - | - | - | focuses the last selectable widget under the stored cursor |
| 0x0056ef20 | `gui_menu_activate_by_id` | `ActivateByID__7GM_MenuFUi` | 0x002e59d0 | STRUCTURAL | focuses the widget with this id |
| 0x0056ef60 | `gui_menu_activate_by_id_or_prev` | `ActivateByIDOrPrev__7GM_MenuFUi` | 0x002e5b30 | STRUCTURAL | focuses this id or the nearest selectable below it |
| 0x0056efa0 | `gui_menu_clear_focus` | - | - | - | drops `focused` and its widget flag |
| 0x0056efc0 | `gui_menu_refocus_if_disabled` | - | - | - | moves `focused` on when its widget stopped being selectable |
| 0x0056eff0 | `gui_menu_set_component_text` | `SetText__7GM_MenuFUiPUc` | 0x002e56b0 | CONFIRMED | widget slot `+0x20` by id |
| 0x0056f030 | `gui_menu_set_component_string` | `SetString__7GM_MenuFUiPUc` | 0x002e56f0 | STRUCTURAL | widget slot `+0x24` by id |
| 0x0056f070 | `gui_menu_get_component_enabled` | `GetDisabled__7GM_MenuFUi` | 0x002e54d0 | CONFIRMED | widget flag bit 0 by id |
| 0x0056f0a0 | `gui_menu_set_component_enabled` | `SetDisabled__7GM_MenuFUib` | 0x002e5470 | CONFIRMED | sets or clears widget flag bit 0 on every widget with this id |
| 0x0056f100 | `gui_menu_set_component_visible` | `SetHidden__7GM_MenuFUib` | 0x002e5510 | CONFIRMED | clears or sets widget flag `0x100` on every widget with this id |
| 0x0056f160 | `gui_menu_get_component_text_ptr` | `GetText__7GM_MenuFUi` | 0x002e5730 | STRUCTURAL | widget `+0x30` by id |
| 0x0056f190 | `gui_menu_set_component_text_ptr` | `SetText__7GM_MenuFUiPUc` | 0x002e56b0 | STRUCTURAL | widget `+0x30` by id |
| 0x0056f1c0 | `gui_menu_get_component_float` | `GetFloat__7GM_MenuFUi` | 0x002e57b0 | STRUCTURAL | widget `+0x2c` by id |
| 0x0056f1f0 | `gui_menu_set_component_float` | `SetFloat__7GM_MenuFUif` | 0x002e5770 | STRUCTURAL | widget `+0x2c` by id |
| 0x0056f220 | `gui_menu_get_component_pointer` | - | - | STRUCTURAL | widget `+0x20` by id |
| 0x0056f250 | `gui_menu_set_component_pointer` | `SetPointer__7GM_MenuFUiPv` | 0x002e58f0 | STRUCTURAL | widget `+0x20` by id |
| 0x0056f280 | `gui_menu_get_component_dword` | `GetDword__7GM_MenuFUi` | 0x002e5830 | CONFIRMED | widget `+0x24` by id |
| 0x0056f2b0 | `gui_menu_set_component_dword` | `SetDword__7GM_MenuFUiUi` | 0x002e57f0 | CONFIRMED | widget `+0x24` by id |
| 0x0056f2e0 | `gui_menu_get_component_text_id` | `GetTextID__7GM_MenuFUi` | 0x002e5670 | CONFIRMED | widget `+0x0c` by id |
| 0x0056f310 | `gui_menu_set_component_text_id` | `SetTextID__7GM_MenuFUiUi` | 0x002e5630 | CONFIRMED | widget `+0x0c` by id, and the resolved string in `+0x30` |
| 0x0056f360 | `gui_menu_force_update_component` | `ForceUpdate__12GM_ComponentFv` | 0x002c4d10 | STRUCTURAL | widget slot `+0x0c` on every widget with this id |
| 0x0056f390 | `gui_menu_force_update_all` | `ForceUpdate__12GM_ComponentFv` | 0x002c4d10 | STRUCTURAL | widget slot `+0x0c` on every widget |
| 0x0056f3c0 | `gui_menu_tick_components` | - | - | STRUCTURAL | widget slot `+0x1c` `Tick(dt)` on every widget |
| 0x0056f3f0 | `gui_menu_set_component_text_from_option_list` | - | - | - | sets a widget's text to the option whose value matches |
| 0x0056f430 | `gui_menu_set_component_option_list` | `SetPointer__11GM_ComboBoxFPv` | 0x002cc290 | STRUCTURAL | writes a combo widget's option vector at `+0x38` |
| 0x0056f450 | `gui_menu_find_component_by_id` | `FindComponentByID__7GM_MenuFUi` | 0x002e5c00 | CONFIRMED | the widget with this id, or 0 |
| 0x0056f480 | `gui_menu_find_component_by_id_prev` | `FindComponentByIDPrev__7GM_MenuFUi` | 0x002e5c40 | CONFIRMED | the selectable widget before this id, two passes: bounded, then wrapping |
| 0x0056f510 | `gui_menu_find_component_by_id_next` | `FindComponentByIDNext__7GM_MenuFUi` | 0x002e5d30 | CONFIRMED | the selectable widget after this id, two passes: bounded, then wrapping |
| 0x0056f5a0 | `gui_menu_find_prev_selectable_in_order` | - | - | - | the previous selectable widget in vector order, wrapping |
| 0x0056f610 | `gui_menu_find_next_selectable_in_order` | - | - | - | the next selectable widget in vector order, wrapping |

### 3d. Free helpers

The unit's other helpers (`gui_draw_*`, `gui_format_ip_address`, the `stub_*` vtable fillers) are used
by the widget classes and belong to `structs/gui_widget.md`.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056af70 | `gui_screen_design_table_load` | - | - | - | loads `tables\menu.def` into the table in ECX (§6) |
| 0x0056cc90 | `gui_menu_i3d_draw_callback` | - | - | - | draw callback of a screen's `own_scene`: on pass 2, draws the active screen and `hud_render_all` |
| 0x0056e3f0 | `gui_notice_panel_present_once` | - | - | - | builds the notice panel of §9 and presents it for one frame |
| 0x00570400 | `gui_set_clip_rect` | `SetViewport__Fiiii` | 0x002e3710 | CONFIRMED | the clip rectangle `gui_menu_draw` opens and closes with |
| 0x00575c00 | `stub_noop_args8_00575c00` | - | - | - | the empty element destructor of `gui_menu_build_components_from_design`'s vector growth |
| 0x00575c10 | `stl_construct_dword` | - | - | - | the element copy of the same growth |

## 4. Globals

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_gui_active_screen` | 0x0067a4d8 | the top of the screen stack | `gui_menu_close`, `gui_menu_execute_modal`, `gui_notice_panel_present_once` and every caller that opens a screen | `gui_menu_draw`, `gui_menu_execute_modal`, `gui_menu_i3d_draw_callback`, every screen constructor call; 129 lines of the decomp |
| `g_gui_blink_phase_ms` | 0x0067a4e8 | blink phase in ms ¹ | `gui_menu_tick` | `gui_menu_tick`, `gui_menu_draw` |
| `g_gui_screen_design_table` | 0x00672630 | the parsed `tables\menu.def` (§6) | `gui_screen_design_table_load` | every screen constructor, which passes its address to `gui_menu_build_components_from_design` |
| `g_gui_scale_x` | 0x0067a4c0 | `screen_width / 800` | `gui_scale_static_init`, `gui_menu_build_components_from_design` | `gui_menu_build_components_from_design`, `gui_widget_ctor`, `gui_widget_frame_draw`, `gui_widget_save_slot_ctor`, `gui_widget_save_slot_copy_ctor`, `gui_widget_window_titled_draw` |
| `g_gui_scale_y` | 0x0067a4c4 | `screen_height / 600` | `gui_scale_static_init`, `gui_menu_build_components_from_design` | `gui_menu_build_components_from_design`, `gui_menu_layout_add_vertical_space`, `gui_garage_full_ctor`, `gui_garage_select_ctor`, `gui_widget_ctor`, `gui_widget_create_from_design` and four more widget functions |
| `g_gui_mouse_x` | 0x00672618 | menu cursor X in pixels, seeded 400.0 | `gui_mouse_pos_static_init`, `gui_menu_execute_modal` | `gui_menu_execute_modal` |
| `g_gui_mouse_y` | 0x0067261c | menu cursor Y in pixels, seeded 300.0 | `gui_mouse_pos_static_init`, `gui_menu_execute_modal` | `gui_menu_execute_modal` |
| `g_gui_menu_open_sound` | 0x0067a4e0 | `sounds\menuopen.wav` ² | `gui_menu_execute_modal` | `gui_menu_ctor`, `gui_menu_execute_modal` |
| `g_gui_menu_change_sound` | 0x0067a4dc | `sounds\menuchange.wav`, played on every focus move | `gui_menu_execute_modal` | `gui_menu_execute_modal`, `gui_menu_tick` |

¹ Accumulated by `gui_menu_tick`, wraps above `0x578` = 1400; above 700 `gui_menu_draw` clears bit
`0x4000` of the flag argument it passes to each widget's Draw: a 50 % duty cycle. Three Draw functions
test the bit: `gui_widget_edit_text_draw` and `gui_widget_edit_ip_draw` blink their caret, and
`gui_widget_resize_grip_draw` blinks its mark.

² `gui_menu_execute_modal` loads it on first use and plays it then unless the top screen carries flag
`0x400000`; `gui_menu_ctor` plays it every time a screen is built; the tail of the modal loop releases
it.

**The GUI is authored in an 800x600 design space.** `g_gui_scale_x` / `g_gui_scale_y` are recomputed
from the live resolution at the top of `gui_menu_build_components_from_design` (`width * 0.00125`,
`height * 0.0016666667`), and every `menu.def` coordinate passes through them. That is also why the
cursor is seeded to 400/300, the centre of that space.

## 5. What this subsystem borrows from `G_IndicatorsClass`

The GUI does not own a text engine. It calls **HUD methods** on the HUD singleton
(`mov ecx, 0x658330` at every site):

* `hud_text_draw_line` — 44 of its 70 call sites are in this `0x0057xxxx` region.
* `hud_text_draw_line_alt_font` and `hud_text_measure_fixed_pitch` — used **only** here,
  by the text-input widget `gui_widget_edit_ip_draw` (caret and space width), the draw of the
  `'edip'` class, which nothing ever builds (`structs/gui_widget.md` §6). See
  `structs/hud.md` §1 for the two-font-table layout those two read.
* `hud_draw_mouse_cursor` — the cursor quad, out of the `1int.tga` sheet at `hud+0x9080`.
* `hud_render_all` — called by `gui_menu_i3d_draw_callback` right after the GUI itself.

## 6. `tables\menu.def` — the UI shipped as data

`g_gui_screen_design_table` (`0x00672630`) is the parsed file: **72 designs**, ids `0x00..0x47` (the
last 16 empty), each a run of `0x24`-byte records
`{u32 id, FourCC tag, float x, y, w, h, u32 text_id, u32 flags, u32 colour}` terminated by a `stop`
record.

`gui_menu_build_components_from_design(this, table, design_id, gate)` walks `design_id`
`stop`-terminated groups and then instantiates that group — byte-verified, `cmp ecx, 0x73746f70` at
`0x0056e5c3` — so **the group index in the shipped file IS the literal each constructor pushes**.
Decoding the file therefore gives every screen its component ids, its geometry and, through
`textdb.json`, its labels. Inside a group a `base` record supplies an id base and a position offset, a
`gate` record switches the following widgets on or off against the `gate` argument, and a `vers` record
is skipped.

**The loader is `gui_screen_design_table_load` (`0x0056af70`).** It is
`dtaOpen("tables\menu.def", 0)` → `dtaRead(handle, table, 0x7e90)` → `dtaClose`, and on a missing
archive entry it stamps `'stop'` (`0x73746f70`) into the first record's tag so the whole table reads as
empty instead of as garbage. `0x7e90` = 32400 bytes is the capacity; the shipped file is 29124.

**Read the fr/cz textdb columns, not en.** Retail English overwrote several racing strings (ids 410-412)
with debug text; fr/cz still carry the real labels.

Construction is uniform across every screen in the game:

```
    gui_menu_ctor(parent, screen_id)              // base
    *this = &<the screen's own vtable>            // 6 slots
    gui_menu_build_components_from_design(g_gui_screen_design_table, <design_id>, gate)
    … seed the widgets …
    gui_menu_force_update_all()
```

## 7. The translation unit

`[0x0056adc0, 0x00575c20)`: 210 function starts, all named, from the unit's own CRT guard
`crt_static_init_guard_0056adc0` to the next unit's `crt_static_init_guard_00575c20`. The range holds
one `.CRT$XCU` group, the initializers `thunk_gui_scale_static_init` and
`thunk_gui_mouse_pos_static_init`. The widget half runs from about `0x0056f680` to the end of the range
(`structs/gui_widget.md`).

**Input map, from `gui_menu_tick`:** scancode `0xc8` up and `0xd0` down drive the ID-order walkers;
`0xcb` left and `0xcd` right, plus `0x0f` tab, drive the LIST-order walkers. Tab with bit 0 of the tick's first argument set
walks backwards. A focus change plays `g_gui_menu_change_sound`.

**`gui_menu_i3d_draw_callback` has zero call sites.** Its only reference in the whole decomp is
`&gui_menu_i3d_draw_callback`, handed to an I3D object's vtable `+0x50`: it is only ever taken by
address.

## 8. The 6-slot vtable

`gui_menu_close` is the proof. Its body sets `g_gui_active_screen = this->parent`, calls the **parent's**
vtable `+0x0c` with `(this->id, result)`, and then its **own** `+0x08`. That single function pins slots
2, 3 and 4 at once.

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 0 | 0x0056aef0 | `gui_menu_scalar_deleting_dtor` (`gui_menu_scalar_deleting_dtor_common` on the screens that add no destructor work) | `__dt__7GM_MenuFv` |
| 1 | 0x005634a0 | `gui_menu_on_exchange_forward_to_parent`: re-create in place on a language or resolution change; the default in **45** vtables | `OnExchange__7GM_MenuFv` |
| 2 | 0x0056e8a0 | `gui_menu_destroy`: free every widget, then delete self | `Destroy__7GM_MenuFv` |
| 3 | 0x0056e8f0 | `gui_menu_on_close_child`: `OnCloseChild(child_id, result)` | `OnCloseChild__7GM_MenuFP7GM_MenuUiUi` |
| 4 | 0x0056e930 | `gui_menu_close`: notify the parent, then destroy | `MenuClose__7GM_MenuFUi` |
| 5 | 0x0056ea70 | `gui_menu_tick`: per-frame update **and** input; returns the activated widget id | `Tick__7GM_MenuFiiiUiUiUiUi` |

**Slot 4 also carries `OnClose` work.** A class that needs teardown work overrides slot 4 and chains
`gui_menu_close` (`gui_race_car_model_close` is the worked example). A class that only needs widget
teardown overrides slot 2 instead, which is what the existing `*_on_close` names denote.

## 9. Two screens in the NEXT translation unit that `g_menu.cpp` constructs

That TU is documented in full in `structs/game_menus.md`. Both screens below are the first two functions
after the framework's range and are reached only from widgets or helpers inside the GUI TU.

| PC function | VA | screen | design | vtable | reached from |
|---|---|---|---|---|---|
| `gui_combo_dropdown_screen_ctor` | 0x00575c50 | the combo drop-down list | `0x13` | 0x0063ddfc | `gui_widget_combo_key`, `gui_widget_combo_mouse`, `gui_widget_combo_range_key`, `gui_widget_combo_range_mouse` |
| `gui_notice_panel_ctor` | 0x00575f50 | the two-message NOTICE PANEL (no buttons) | `0` | 0x0063de14 | `gui_notice_panel_present_once`, `entity_process_command`, `gui_gun_shop_tick`, `net_host_session_start`, `net_open_session_browser`, `net_multiplayer_run` |

**`gui_combo_dropdown_screen_ctor`** takes the field's screen rectangle, the option vector and the owning
combo's write-back slot, builds design `0x13`, and sets the flag bits `0x8010`.

**`gui_notice_panel_ctor`** takes **two textdb string ids**, pushes them into components `0x82` and `0x83`
of design `0`, then measures each message and — when a message is wider than its component — widens the
frame and both message rows by the overflow and re-centres the frame around it. That auto-fit pass is why
the function is 600+ bytes. Design `0` ships a `vers` marker, a frame and TWO `ctxt` rows and **no
buttons**. A `const char *` overload, `gui_notice_panel_ctor_from_strings` (`0x005761d0`), shares the
vtable and the size.

**`gui_notice_panel_present_once`** (`0x0056e3f0`, inside the GUI TU) is the caller the multiplayer front
end uses for a one-frame panel: it pushes `g_hud_flags` onto the save stack
(`g_hud_flags_save_stack_begin` / `_end`), forces the flags to 1, builds the panel, drives **one** frame
and pops. There is no input loop: it is a splash, not an interactive modal.

## 10. The widget fields the screen code reads

The full field map, the 10-slot vtable, the 53-class FourCC roster and the drawing primitives of
`GM_Component` are in `structs/gui_widget.md`. The screen code reads these fields directly.

### 10a. `GM_Component`, the screen's view

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | vtable — **10 slots** ¹ | `gui_widget.md` §5 |
| `+0x04` | the widget **id** — the `menu.def` record's id | every accessor matches on it |
| `+0x08` | **flags**: bit 0 = **selectable / enabled** · `0x10` = focused · `0x100` = hidden · `0x20` = wants a per-frame layout pass | all four focus walkers skip a widget with `(flags & 1) == 0`; `gui_menu_draw` tests `0x100`; `gui_menu_tick` tests `0x20` |
| `+0x0c` | textdb string id | `gui_menu_set_component_text_id` writes it and caches the resolved pointer in `+0x30` |
| `+0x14` / `+0x1c` | y / height in screen pixels | `gui_menu_layout_add_vertical_space` reads and adjusts them |
| `+0x20` | an opaque pointer slot | `gui_menu_get_component_pointer`, `gui_menu_set_component_pointer` |
| `+0x24` | an integer value (lap count, position, points) | `gui_menu_get_component_dword`, `gui_menu_set_component_dword` |
| `+0x2c` | a float value — **and the widget's boolean state** | the getter returns `*(float *)`; the racing screens write `1.0f` / `0.0f` here |
| `+0x30` | the resolved `const char *` the widget renders | written both by `gui_menu_set_component_text_ptr` and by `gui_menu_set_component_text_id` |

¹ `+0x08` Draw · `+0x0c` ForceUpdate · `+0x10` `IsOverActive(x, y)` · `+0x14` `Key(scancode, ch)` ·
`+0x18` mouse (5 args) · `+0x1c` `Tick(dt)`, broadcast by `gui_menu_tick_components` and overridden only
by `'swnd'` and `'lstc'` · `+0x20` SetText · `+0x24` SetString.

## 11. PC vs PS2

**The framework has PS2 twins throughout**, confirmed by body diffs (§3). The PS2 file name of the unit is
`g_menu.cpp`, the name the registry section uses; the PC binary carries no source-file string.
`gui_widget_create_from_design` = `G_Factory<12GM_Component>::Make`.
`gui_menu_execute_modal` folds `ExecuteMenu` + `TickCamera` + `SetCameraTargetID` into one function.

**PC-only is the pointer:** `gui_menu_update_hover_focus`, the `+0x31` / `+0x32` / `+0x34` / `+0x38` mouse
block, the LIST-order focus walkers behind tab / left / right, `hud_draw_mouse_cursor`, widget vtable slot
`+0x18`, and the two `.CRT$XCU` initialisers. The PS2 has no mouse; its equivalent in-game inventory UI
(`DrawInventory`, `InventoryScroll`, `_InventorySet`) is tagged **PS2-ONLY** on the HUD class in
`structs/hud.md` §3. Widget slot `+0x1c` `Tick(dt)` is the twin of the PS2 `GM_Component::Tick`; `'swnd'`
and `'lstc'`, its only overriders, are exactly the two classes with a PS2 `Tick`.

**Factoring differences worth knowing before diffing anything else here:**

* The PS2 dispatches every widget value through a `GM_Component` virtual (`SetDword` → vtable `+0x58`,
  `GetBool` → `+0x64`, …). The PC writes a **shared field** on the record instead (`+0x2c` where the PS2
  calls `SetBool`), which is why the PC accessors are 48-byte scans and the PS2 ones are forwarders.
* **`SetDisabled` and `SetHidden` are inverted against their own names on the PS2.** At every matched
  callsite both builds pass the identical argument, and PC flag bit 0 is proven "selectable" by the focus
  walkers, so the PC pair is named `gui_menu_set_component_enabled` and `gui_menu_set_component_visible`.
* The PS2 resolves a widget type through `G_GlobalFactory`; the PC uses a literal FourCC switch inside
  `gui_menu_add_component_from_template`.
* The PS2 `MenuClose` only FLAGS a screen and lets the next `Tick` tear it down; the PC destroys inline.
  The PS2 has a separate `OnClose` virtual; the PC folds it into slot 4 (§8). The PS2 table has 15
  virtual slots (`__vt__7GM_Menu` @ `0x004d9510`) against the PC's 6, so `OnCreate` through
  `MartinovaKlavesa` have nowhere to live on the PC: each screen's constructor and `Tick` do their work.
* The PS2 `OnCloseChild` base is an empty 8-byte stub; the PC base re-arms input steering.
* **Source files.** The PS2 link order puts `g_menu.cpp` after every `g_m_*.cpp`, which the PC order
  matches. The PS2 splits the PC range across FOUR files (`g_menu.cpp` + `g_m_comps.cpp` / `_comps2` /
  `_compsm`), each with its own `__sinit`; the PC range is one unit (§7).
* **Menu sounds.** The PS2 `LoadResources` creates `menuSoundOpen` and `menuSoundChange`, the same pair the
  PC keeps in `g_gui_menu_open_sound` and `g_gui_menu_change_sound`; `menuSoundClose`, played when a child
  screen closes, is declared and has no writer. The PS2 `ExecuteMenu` tests the same screen flag `0x400000`
  to choose between its two draw paths.

### 11a. The two next-TU screens (§9)

Both fold N PS2 functions into one PC function, because the PC has a shipped design table where the PS2
has code:

| PC function | PS2 functions | grade |
|---|---|---|
| `gui_combo_dropdown_screen_ctor` | `__ct__12GM_ComboMenuFP12GM_Component` + `OnCreate__12GM_ComboMenuFv` | STRUCTURAL ¹ |
| `gui_notice_panel_ctor` | - (PC-only, below) | - |

¹ One PC function against two PS2 functions, so no single-body agreement is assertable. The PS2
`GM_ComboMenu` constructor stores only the owner and pulls the option vector back through the owner's
`GetPointer` virtual (slot `0x6c`); its `OnCreate` clones one row per option with `PridejComponent` and
sets each row's text, and its `OnClick` writes the picked value back through the owner's `SetDword` virtual
(slot `0x58`) before closing. The PC passes the vector and the write-back slot in as arguments — the same
devirtualisation as the widget accessors. The fold site carries a `// PS2-INLINED HERE:` banner in the
annotated decomp.

**The design-0 notice panel is NOT `GM_QuestionBox`.** The auto-fit pass is shared boilerplate (three PC
classes run it), so the discriminator is the component id set:

| evidence | PS2 `GM_QuestionBox` | PC design **1** | PC design **0** |
|---|---|---|---|
| components | `0x82` `0x83` **`0x8a` `0x8b`** | `0x82` `0x83` **`0x8a` Yes `0x8b` No** | `0x82` `0x83`, **no buttons** |
| Yes result | close(1) → **`0xb`** | close(1) → **`0xb`** | - |
| No / ESC | close(0) → **`0xc`** | close(0) → **`0xc`** | - |
| root vs child | forks on `+0x20` | forks on `+0x18` | closes unconditionally |

`OnCreate__14GM_QuestionBoxFv` shifts components `0x8a` and `0x8b`, which design 0 does not ship, and
`GM_QuestionBox` takes a `bool` third constructor argument the design-0 ctor does not. The PC twin is
**`gui_question_box_ctor` / `gui_question_box_tick`** (`0x005764c0` / `0x00576560`, design 1) —
`structs/game_menus.md` §6. The `GM_QuestionBox` constructor flag only picks the console layout
`psx_questionbox`; the Yes/No pair is always there.

**The notice panel itself is PC-only.** At every call position of its constructors (§9) the PS2 builds
nothing: its `C_entity::Process` arms for the entity-script commands `0x100f` / `0x1010` only advance the
script, its `GM_GunShop` is gutted, and the other callers are the multiplayer front end
(`structs/game_menus.md` §13c).

### 11b. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002c1b50 | `OnBeforeExchange__7GM_MenuFv` | a virtual with no PC slot (§8) |
| 0x002c1b70 | `OnCreate__7GM_MenuFv` | a virtual with no PC slot: each screen's constructor does it |
| 0x002c1b80 | `OnCreateChild__7GM_MenuFP7GM_MenuUi` | a virtual with no PC slot |
| 0x002c1ba0 | `OnChange__7GM_MenuFP12GM_ComponentP12GM_Component` | a virtual with no PC slot |
| 0x002c1bb0 | `MartinovaKlavesa__7GM_MenuFUiUi` | a virtual with no PC slot |
| 0x002cce20 | `OnClick__7GM_MenuFUi` | a virtual with no PC slot: each screen's `Tick` tests the returned widget id |
| 0x002cced0 | `OnKey__7GM_MenuFUcUi` | a virtual with no PC slot: each screen's `Tick` tests the key |
| 0x002ccf00 | `OnMouse__7GM_MenuFiiiUiUi` | a virtual with no PC slot |
| 0x002ccf10 | `OnTick__7GM_MenuFUi` | a virtual with no PC slot: each screen's `Tick` |
| 0x002d3990 | `OnClose__7GM_MenuFv` | folded into slot 4 (§8) |
| 0x002e3840 | `LoadResources__7GM_MenuFv` | the sound half is open-coded in `gui_menu_execute_modal`; no PC function for the texture half located (Open item 2) |
| 0x002e3ac0 | `UnloadResources__7GM_MenuFv` | the sound half is open-coded in `gui_menu_execute_modal`; no PC function for the texture half located (Open item 2) |
| 0x002e4120 | `PreCache__7GM_MenuFv` | no PC function in the unit's closed list of starts |
| 0x002e42e0 | `Create__7GM_MenuFv` | open-coded in each screen's constructor (§6) |
| 0x002e5070 | `MenuOpen__7GM_MenuFP7GM_MenuUi` | open-coded: `g_gui_active_screen = ctor(g_gui_active_screen, …)` |
| 0x002e50b0 | `MenuOpenForMartin__7GM_MenuFP7GM_MenuUi` | no PC function in the unit's closed list of starts |
| 0x002e50f0 | `MenuExchange__7GM_MenuFv` | no single function: the slot 1 overrides free the widgets and rebuild (`carcyclopedia_on_exchange` is the worked example) |
| 0x002e5290 | `MenuCloseAll__7GM_MenuFUiUi` | no single function: the tail of `gui_menu_execute_modal` destroys the whole stack |
| 0x002e5330 | `UtilCenterH__7GM_MenuFUi` | no PC function in the unit's closed list of starts |
| 0x002e53d0 | `UtilCenterV__7GM_MenuFUi` | no PC function in the unit's closed list of starts |
| 0x002e5570 | `SetAlternateStyle__7GM_MenuFUib` | no PC function in the unit's closed list of starts |
| 0x002e55d0 | `SetAlternateColor__7GM_MenuFUib` | no PC function in the unit's closed list of starts |
| 0x002e5870 | `SetBool__7GM_MenuFUib` | the PC writes the shared field through `gui_menu_set_component_float` |
| 0x002e58b0 | `GetBool__7GM_MenuFUi` | the PC reads the shared field through `gui_menu_get_component_float` |
| 0x002e5930 | `GetRangeFloat__7GM_MenuFUiRfRf` | no PC function in the unit's closed list of starts |
| 0x002e5980 | `SetRangeDword__7GM_MenuFUiUiUi` | no PC function in the unit's closed list of starts |
| 0x002e5a60 | `ActivateByIDOrNext__7GM_MenuFUi` | open-coded at the same position: `gui_menu_build_components_from_design` stores `gui_menu_find_next_selectable_in_order(1, 0)` in `focused` |

## 12. Related

* `structs/gui_widget.md` — the widget class `GM_Component` and its subclasses.
* `structs/game_menus.md` — the next TU: the main, options, load and modal screens.
* `structs/racing_menus.md` — the 14 racing screens.
* `structs/multiplayer.md` — the multiplayer front end.
* `structs/carcyclopedia.md` — the Carcyclopedia screen.
* `structs/hud.md` — the HUD methods of §5.

## 13. Open items

1. **In game: one cursor per stack?** Opening a submenu over a menu should leave exactly one cursor drawn,
   on the topmost menu (`gui_menu_draw` draws it only when `g_gui_active_screen == this`).
2. **The menu textures of the PS2 `LoadResources`** (`textureMenuTop` and its four siblings): the PC
   counterpart of those five loads was not located.
