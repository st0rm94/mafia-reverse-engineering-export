# `GM_Menu` — the GUI screen framework

**Created** 2026-08-16 18:02 +0200 · **Modified** 2026-09-27 17:33 +0200

The framework's own translation unit is `g_menu.cpp` `[0x0056adf0, 0x00575c20)` (file name
inferred, §10): 208 function starts, all named. This file documents the screen class and its API
(§7); the widget classes in the same TU are documented in `structs/gui_widget.md`.

**Prefix:** `gui_menu_` for the base class itself; `gui_<screen family>_<screen>_<method>`
for the screens built on it (`gui_mp_lobby_tick`, `gui_race_setup_ctor`,
`carcyclopedia_tick`).

**Related docs:** `structs/gui_widget.md` (the widget class), `structs/game_menus.md` (the next
TU: the main, options, load and modal screens), `structs/racing_menus.md` (the 14 racing
screens), `structs/multiplayer.md` (the multiplayer front end), `structs/carcyclopedia.md`.

---

## 1. What it is

A screen (window) in the menu system. Screens form a **stack**: each one holds its parent
at `+0x18`, and the top of the stack is the global `g_gui_active_screen` (`0x0067a4d8`,
215 references). A screen owns a vector of `GM_Component` **widgets** and dispatches draw,
layout and input to them.

The base class is **`GM_Menu`** (a real class on both builds, §8). The **mouse** is PC-only: the
cursor, the hit-testing and the pointer-vs-keyboard steering.

Construction is uniform across every screen in the game:

```
    gui_menu_ctor(parent, screen_id)              // base
    *this = &<the screen's own vtable>            // 6 slots
    gui_menu_build_components_from_design(g_gui_screen_design_table, <design_id>, gate)
    … seed the widgets …
    gui_menu_force_update_all()
```

That `<design_id>` literal is the join key into the shipped `tables\menu.def` (§6), which
is what makes every screen in the binary identifiable without reading its body.

## 2. Field map (measured)

### 2a. `GM_Menu`

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | vtable — 6 slots, see §3 | installed by every screen constructor after `gui_menu_ctor` |
| `+0x04` | inherited from the parent by the constructor | `gui_menu_ctor` copies `parent+0x04` |
| `+0x18` | **the PARENT screen**, 0 for a root | `gui_menu_close` assigns it to `g_gui_active_screen` and notifies it through slot 3; every constructor is called as `ctor(g_gui_active_screen, …)` |
| `+0x1c` | low byte of the parent pointer, kept as a flag | written alongside `+0x18` |
| `+0x20` / `+0x24` / `+0x28` | the **widget vector** — begin / end / cap, elements are POINTERS | every accessor walks `[+0x20]`…`[+0x24]` stepping 4; `gui_menu_dtor` frees all three |
| `+0x2c` | the **focused widget**, or 0 | the four focus walkers write it; `gui_menu_tick` moves flag `0x10` with it |
| `+0x31` | **"the pointer has moved"** flag | set when the Manhattan distance the mouse travelled this frame **exceeds 10 pixels**, and unconditionally on a click |
| `+0x32` | a gate that suppresses the move test | checked immediately before it; `gui_menu_on_close_child` re-arms it |
| `+0x34` / `+0x38` | **mouse x / y**, in integer pixels | written by `gui_menu_tick` from two of its arguments; read by the cursor draw with `fild` (integer→float), which is what proves they are pixels |
| `+0x3c` | click / consumed state | zeroed on a button-down; bit 0 tested before the widget hit-test |
| `+0x40` | flag word, seeded `0x10001` ¹ | `gui_menu_ctor`; screen constructors |
| `+0x44` | the **screen id** handed back to the parent on close | `gui_menu_ctor`'s second parameter; `(**(*this + 0x10))(this[0x11], n)` is "close me with result n" |

¹ Bit `0x200000` gates a `g_racing` results-visibility early-out; bit `0x10000` drives the
recursive parent redraw; screens clear `0x10000` and set `0x80000` in their constructors.

### 2b. `GM_Component` (the widget record)

The full field map, the 10-slot vtable, the 53-class FourCC roster and the drawing primitives are
in `structs/gui_widget.md`. The summary below is kept because the screen code reads these fields
directly.

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | vtable — **10 slots** ¹ | `gui_widget.md` §3 |
| `+0x04` | the widget **id** — the `menu.def` record's id | every accessor matches on it |
| `+0x08` | **flags**: bit 0 = **selectable/enabled** · `0x10` = focused · `0x100` = hidden · `0x20` = wants a per-frame layout pass | all four focus walkers skip a widget with `(flags & 1) == 0` |
| `+0x0c` | textdb string id | `gui_menu_set_component_text_id` writes it and caches the resolved pointer in `+0x30` |
| `+0x14` / `+0x1c` | y / height in screen pixels | the row-stacking helpers read and adjust them |
| `+0x20` | an opaque pointer slot | `gui_menu_get/set_component_pointer` |
| `+0x24` | an integer value (lap count, position, points) | `gui_menu_get/set_component_dword` |
| `+0x2c` | a float value — **and the widget's boolean state** | the getter returns `*(float *)`; the racing screens write `1.0f`/`0.0f` here |
| `+0x30` | the resolved `const char*` the widget renders | written both by `..._set_component_text_ptr` and by `..._set_component_text_id` |

¹ `+0x08` Draw · `+0x0c` ForceUpdate · `+0x10` `IsOverActive(x, y)` · `+0x14` `Key(scancode, ch)` ·
`+0x18` mouse (5 args, **PC-only**) · `+0x1c` `Tick(dt)`, broadcast by `gui_menu_tick_components`
and overridden only by `'swnd'` and `'lstc'` · `+0x20` SetText · `+0x24` SetString.

## 3. The 6-slot vtable — byte-proven

`gui_menu_close` is the proof. Its body sets `g_gui_active_screen = this->parent`, calls
the **parent's** vtable `+0x0c` with `(this->id, result)`, and then its **own** `+0x08`.
That single function pins slots 2, 3 and 4 at once.

| slot | offset | role | framework default |
|---|---|---|---|
| 0 | `+0x00` | scalar deleting destructor | `gui_menu_scalar_deleting_dtor_common` (`0x00578790`), or `gui_menu_scalar_deleting_dtor` on the base |
| 1 | `+0x04` | `OnExchange` — re-create in place (language / resolution change) | `gui_menu_on_exchange_forward_to_parent`, in **45** vtables |
| 2 | `+0x08` | `Destroy` — free every widget, then delete self | `gui_menu_destroy` |
| 3 | `+0x0c` | `OnCloseChild(child_id, result)` | `gui_menu_on_close_child` |
| 4 | `+0x10` | `MenuClose(result)` — notify the parent, then destroy | `gui_menu_close` |
| 5 | `+0x14` | `Tick` — per-frame update **and** input; returns the activated widget id | `gui_menu_tick` |

**Slot 4 also carries `OnClose` work.** A class that needs teardown work overrides slot 4 and
chains `gui_menu_close` (`gui_race_car_model_close` is the worked example). A class that only
needs widget teardown overrides slot 2 instead — which is what the existing `*_on_close` names
denote.

## 4. Globals

| VA | Name | Meaning |
|---|---|---|
| `0x0067a4d8` | `g_gui_active_screen` | top of the screen stack; 215 references |
| `0x0067a4e8` | `g_gui_blink_phase_ms` | blink phase in ms ¹ |
| `0x00672630` | `g_gui_screen_design_table` | the parsed `tables\menu.def` (§6) |
| `0x0067a4c0` | `g_gui_scale_x` | `screen_width / 800` |
| `0x0067a4c4` | `g_gui_scale_y` | `screen_height / 600` |
| `0x00672618` | `g_gui_mouse_x` | menu cursor X in pixels; seeded to 400.0f |
| `0x0067261c` | `g_gui_mouse_y` | menu cursor Y in pixels; seeded to 300.0f |
| `0x0067a4e0` | `g_gui_menu_open_sound` | `sounds\menuopen.wav` |
| `0x0067a4dc` | `g_gui_menu_change_sound` | `sounds\menuchange.wav`, played on every focus move |

¹ Accumulated by `gui_menu_tick`, wraps at `0x578` = 1400, switches state at 700 — a 50 % duty
cycle. Consumed by the text-input widget `gui_widget_edit_ip_draw` (`0x005753b0`), which draws a
`"\|"` caret.

🔑 **The GUI is authored in an 800x600 design space.** `g_gui_scale_x`/`g_gui_scale_y` are
recomputed from the live resolution at the top of
`gui_menu_build_components_from_design` (`width * 0.00125`, `height * 0.0016666667`), and
every `menu.def` coordinate passes through them. That is also why the cursor is seeded to
400/300 — the centre of that space.

## 5. What this subsystem borrows from `G_IndicatorsClass`

The GUI does not own a text engine. It calls **HUD methods** on the HUD singleton
(`mov ecx, 0x658330` at every site):

* `hud_text_draw_line` — 44 of its 70 call sites are in this `0x0057xxxx` region.
* `hud_text_draw_line_alt_font` and `hud_text_measure_fixed_pitch` — used **only** here,
  by the text-input widget `gui_widget_edit_ip_draw` (caret and space width). See
  `structs/hud.md` §2 for the two-font-table layout those two read.
* `hud_draw_mouse_cursor` — the cursor quad, out of the `1int.tga` sheet at `hud+0x9080`.
* `hud_render_all` — called by `gui_menu_i3d_draw_callback` right after the GUI itself.

## 6. `tables\menu.def` — the UI shipped as data

`g_gui_screen_design_table` (`0x00672630`) is the parsed file: **72 designs**, ids
`0x00..0x47` (the last 16 empty), each a run of 0x24-byte records
`{u32 id, FourCC tag, float x, y, w, h, u32 text_id, u32 flags, u32 colour}` terminated by
a `stop` record.

`gui_menu_build_components_from_design(this, table, design_id, gate)` walks `design_id`
`stop`-terminated groups and then instantiates that group — byte-verified,
`cmp ecx, 0x73746f70` at `0x0056e5c3` — so **the group index in the shipped file IS the
literal each constructor pushes**. Decoding the file therefore gives every screen its
component ids, its geometry and, through `textdb.json`, its labels.

**The loader is `gui_screen_design_table_load` (`0x0056af70`).** It is
`dtaOpen("tables\menu.def", 0)` → `dtaRead(handle, table, 0x7e90)` → `dtaClose`, and on a
missing archive entry it stamps `'stop'` (`0x73746f70`) into the first record's tag so the
whole table reads as empty instead of as garbage. `0x7e90` = 32400 bytes is the capacity;
the shipped file is 29124.

⚠️ **Read the fr/cz textdb columns, not en.** Retail English overwrote several racing
strings (ids 410-412) with debug text; fr/cz still carry the real labels.

## 7. The named API (`g_menu.cpp`)

Registry section: *"GUI menu framework — the `g_menu.cpp` translation unit"*.

| band | functions |
|---|---|
| static init | `gui_scale_static_init` · `gui_mouse_pos_static_init` (+ their two `.CRT$XCU` thunks) |
| lifecycle | `gui_menu_ctor` · `gui_menu_dtor` · `gui_menu_scalar_deleting_dtor` · `gui_menu_free_components` · `gui_menu_destroy` · `gui_menu_close` · `gui_menu_on_close_child` |
| design / data | `gui_screen_design_table_load` · `gui_menu_build_components_from_design` |
| layout | `gui_menu_add_component_from_template` · `gui_menu_layout_add_vertical_space` · `gui_menu_delete_component` · `gui_menu_force_update_component` · `gui_menu_force_update_all` |
| frame | `gui_menu_tick` · `gui_menu_draw` · `gui_menu_i3d_draw_callback` · `gui_menu_tick_components` (broadcasts widget slot 7 `Tick(dt)`) |
| modal loop | `gui_menu_execute_modal` · `gui_notice_panel_present_once` (§9) |
| focus | `gui_menu_update_hover_focus` · `gui_menu_activate_by_id` · `gui_menu_activate_by_id_or_prev` · `gui_menu_clear_focus` · `gui_menu_refocus_if_disabled` · `gui_menu_find_component_by_id` · `..._by_id_prev` · `..._by_id_next` · `gui_menu_find_prev_selectable_in_order` · `gui_menu_find_next_selectable_in_order` |
| widget accessors | `gui_menu_set_component_text` / `_string` / `_text_ptr` / `_text_id` / `_float` / `_dword` / `_pointer` / `_enabled` / `_visible`, and the matching getters |
| widget factory | `gui_widget_create_from_design` (the 53-arm FourCC switch) · `gui_menu_clone_component_under` |
| combo helpers | `gui_menu_set_component_option_list` · `gui_menu_set_component_text_from_option_list` |

**Input map, from `gui_menu_tick`:** scancode `0xc8` up and `0xd0` down drive the ID-order
walkers; `0xcb` left and `0xcd` right, plus `0x0f` tab, drive the LIST-order walkers. A
focus change plays `g_gui_menu_change_sound`.

🔑 **`gui_menu_i3d_draw_callback` has zero call sites.** Its only reference in the whole
decomp is `&gui_menu_i3d_draw_callback`, handed to an I3D object's vtable `+0x50`: it is
only ever taken by address.

## 8. PC vs PS2

**The framework has PS2 twins throughout**, confirmed by body diffs: `__ct__7GM_MenuFv`,
`__dt__7GM_MenuFv`, `Destroy`, `Draw`, `Tick`, `FindComponentByID`, `FindComponentByIDPrev`,
`FindComponentByIDNext`, `SetTextID`, `GetTextID`, `SetDword`, `GetDword`, `SetDisabled`,
`GetDisabled`, `SetHidden`, `DeleteComponent`, `PridejComponent`, `SetText`.
`gui_menu_tick_components` = `GM_Menu::Tick`; `gui_widget_create_from_design` =
`G_Factory<12GM_Component>::Make`; `gui_menu_set_component_option_list` =
`GM_ComboBox::SetPointer`. `gui_menu_execute_modal` folds `ExecuteMenu` + `TickCamera` +
`SetCameraTargetID` into one function.

**PC-only is the pointer:** `gui_menu_update_hover_focus`, the `+0x31`/`+0x32`/`+0x34`/`+0x38`
mouse block, the LIST-order focus walkers behind tab/left/right, `hud_draw_mouse_cursor`, widget
vtable slot `+0x18`, and the two `.CRT$XCU` initialisers. The PS2 has no mouse; its equivalent
in-game inventory UI (`DrawInventory`, `InventoryScroll`, `_InventorySet`) is tagged **PS2-ONLY**
on the HUD class in `structs/hud.md` §4. Widget slot `+0x1c` `Tick(dt)` is the twin of the PS2
`GM_Component::Tick`; `'swnd'` and `'lstc'`, its only overriders, are exactly the two classes with
a PS2 `Tick`.

**Factoring differences worth knowing before diffing anything else here:**

* The PS2 dispatches every widget value through a `GM_Component` virtual
  (`SetDword` → vtable `+0x58`, `GetBool` → `+0x64`, …). The PC writes a **shared field**
  on the record instead (`+0x2c` where the PS2 calls `SetBool`), which is why the PC accessors
  are 48-byte scans and the PS2 ones are forwarders.
* **`SetDisabled` and `SetHidden` are inverted against their own names on the PS2.** At every
  matched callsite both builds pass the identical argument, and PC flag bit 0 is proven
  "selectable" by the focus walkers, so the PC pair is named `_set_component_enabled` and
  `_set_component_visible`.
* The PS2 resolves a widget type through `G_GlobalFactory`; the PC uses a literal FourCC
  switch inside `gui_menu_add_component_from_template`.
* The PS2 `MenuClose` only FLAGS a screen and lets the next `Tick` tear it down; the PC
  destroys inline. The PS2 has a separate `OnClose` virtual; the PC folds it into slot 4 (§3).
* The PS2 `OnCloseChild` base is an empty 8-byte stub; the PC base re-arms input steering.
* **Source files.** The PS2 calls the TU `g_menu.cpp` and its link order puts it after every
  `g_m_*.cpp`, which the PC order matches. The PS2 splits the PC range across FOUR files
  (`g_menu.cpp` + `g_m_comps.cpp` / `_comps2` / `_compsm`).
* **Menu sounds.** The PS2 pair is `menuSoundChange` / `menuSoundClose`;
  `gui_menu_execute_modal` plays its second sound behind a `0x400000` flag test (§10).

### 8a. The two next-TU screens (§9)

Both fold N PS2 functions into one PC function, because the PC has a shipped design table where
the PS2 has code:

| PC function | PS2 functions | grade |
|---|---|---|
| `gui_combo_dropdown_screen_ctor` | `__ct__12GM_ComboMenuFP12GM_Component` + `OnCreate__12GM_ComboMenuFv` | STRUCTURAL ¹ |
| `gui_notice_panel_ctor` | - | twin not searched (§10) |

¹ One PC function against two PS2 functions, so no single-body agreement is assertable. The PS2
`GM_ComboMenu` constructor stores only the owner and pulls the option vector back through the
owner's `GetPointer` virtual (slot `0x6c`); its `OnCreate` clones one row per option with
`PridejComponent` and sets each row's text, and its `OnClick` writes the picked value back through
the owner's `SetDword` virtual (slot `0x58`) before closing. The PC passes the vector and the
write-back slot in as arguments — the same devirtualisation as the widget accessors. The fold site
carries a `// PS2-INLINED HERE:` banner in the annotated decomp.

**The design-0 notice panel is NOT `GM_QuestionBox`.** The auto-fit pass is shared boilerplate
(three PC classes run it), so the discriminator is the component id set:

| evidence | PS2 `GM_QuestionBox` | PC design **1** | PC design **0** |
|---|---|---|---|
| components | `0x82` `0x83` **`0x8a` `0x8b`** | `0x82` `0x83` **`0x8a` Yes `0x8b` No** | `0x82` `0x83`, **no buttons** |
| Yes result | close(1) → **`0xb`** | close(1) → **`0xb`** | - |
| No / ESC | close(0) → **`0xc`** | close(0) → **`0xc`** | - |
| root vs child | forks on `+0x20` | forks on `+0x18` | closes unconditionally |

`OnCreate__14GM_QuestionBoxFv` shifts components `0x8a` and `0x8b`, which design 0 does not ship,
and `GM_QuestionBox` takes a `bool` third constructor argument the design-0 ctor does not. The PC
twin is **`gui_question_box_ctor` / `gui_question_box_tick`** (`0x005764c0` / `0x00576560`,
design 1) — `structs/game_menus.md` §3.

## 9. Two screens in the NEXT translation unit that `g_menu.cpp` constructs

That TU is documented in full in `structs/game_menus.md`. Both screens below are the first two
functions after `[0x0056adf0, 0x00575c20)` and are reached only from widgets or helpers inside the
GUI TU.

| PC function | VA | screen | design | vtable | reached from |
|---|---|---|---|---|---|
| `gui_combo_dropdown_screen_ctor` | `0x00575c50` | the combo drop-down list | `0x13` | `0x0063ddfc` | `gui_widget_combo_key`, `gui_widget_combo_mouse`, `gui_widget_combo_range_key`, `gui_widget_combo_range_mouse` |
| `gui_notice_panel_ctor` | `0x00575f50` | the two-message NOTICE PANEL (no buttons) | `0` | `0x0063de14` | `gui_notice_panel_present_once` |

**`gui_combo_dropdown_screen_ctor`** takes the field's screen rectangle, the option vector and
the owning combo's write-back slot, builds design `0x13`, and sets the modal flag bits `0x8010`.

**`gui_notice_panel_ctor`** takes **two textdb string ids**, pushes them into components
`0x82` and `0x83` of design `0`, then measures each message and — when a message is wider than
its component — widens the frame and both message rows by the overflow and re-centres the
frame around it. That auto-fit pass is why the function is 600+ bytes. Design `0` ships a `vers`
marker, a frame and TWO `ctxt` rows and **no buttons**. A `const char *` overload,
`gui_notice_panel_ctor_from_strings` (`0x005761d0`), shares the vtable and the size.

**`gui_notice_panel_present_once`** (`0x0056e3f0`, inside the GUI TU) is the only caller of the
ctor: it pushes `g_hud_flags` onto the save stack (`g_hud_flags_save_stack_begin`/`_end`), forces
the flags to 1, builds the panel, drives **one** frame, and pops. There is no input loop — it is a
splash, not an interactive modal.

## 10. Open items

| item | state |
|---|---|
| the PC source file name | `g_menu.cpp` is inferred from the PS2 (§8), not measured; whether the PC range, which the PS2 splits across four files and which has a single static-init pair at its head, is one PC translation unit is not measured either. The `.CRT$XCU` table would settle it |
| `g_gui_menu_open_sound` | may be a misnomer: the PS2 pair is change / **close**, and `gui_menu_execute_modal` plays the second sound behind a `0x400000` flag test (§8); not re-graded |
| the design-0 notice panel's PS2 counterpart | not searched (§8a) |
| in game: one cursor per stack? | opening a submenu over a menu should leave exactly one cursor drawn, on the topmost menu |
