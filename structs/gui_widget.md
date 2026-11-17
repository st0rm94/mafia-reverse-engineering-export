# `GM_Component` — the GUI widget base and its 53 subclasses

**Created** 2026-08-24 22:00 +0200 · **Modified** 2026-11-16 10:48 +0100

Companion to `structs/gui_screen.md`, which documents the *screen* class `GM_Menu`; this file documents
the *widgets* a screen owns. The unit that follows this one, the ~30 `GM_Menu` screen classes that use
these widgets, is documented in `structs/game_menus.md`.

| | |
|---|---|
| PC prefix | `gui_widget_`, `gui_draw_` |
| PC kind (`+0x10`) | - |
| PC vtable | one per class, 10 slots each, in the run `0x0063d59c`..`0x0063ddfc`; the base's is `g_gui_widget_vt` @ `0x0063dbdc` |
| PC base class | - |
| PC instance size | `0x38` for the base; 0x38 to 0x1038 per class (§6) |
| PC constructor | `gui_widget_ctor` |
| PC translation unit | `[0x0056adc0, 0x00575c20)`, guard to guard, every start named; the host doc is `structs/gui_screen.md`; the widget half starts about `0x0056f680` |
| PS2 class | `GM_Component` (tag `12GM_Component`), 36 methods, and 52 widget classes with 402 methods (§8) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct GM_Component {                 // 0x38 bytes
    /* +0x00 */ void *vtable;                     // [H] 10 slots (§5)
    /* +0x04 */ uint32_t id;                      // [H] the menu.def record's id
    /* +0x08 */ uint32_t flags;                   // [H] §2a
    /* +0x0c */ uint32_t text_id;                 // [H] textdb id, resolved into text
    /* +0x10 */ float x;                          // [H] left edge in pixels (see below)
    /* +0x14 */ float y;                          // [H] top edge in pixels (see below)
    /* +0x18 */ float width;                      // [H] in pixels; the right edge is x + width
    /* +0x1c */ float height;                     // [H] in pixels; the bottom edge is y + height
    /* +0x20 */ uint32_t record_flags;            // [H] reused per class as a colour or a boolean
    /* +0x24 */ uint32_t record_color;            // [H] reused per class as an integer value
    /* +0x28 */ float record_color_f;             // [H] the same word as a float
    /* +0x2c */ float value;                      // [H] the sliders' 0..1 value, the bars' fill
    /* +0x30 */ const char *text;                 // [H] what the widget renders
    /* +0x34 */ GM_Component *parent;             // [H] offsets draw and hit test; 0 = none
} GM_Component;
```

**The four geometry fields** come from the `menu.def` record. `gui_widget_ctor` multiplies them by
`g_gui_scale_x` / `g_gui_scale_y` unless `g_gui_widget_use_raw_coords` is set. When `parent` is set, `x`
and `y` are relative to the parent's client origin: `gui_widget_is_point_inside` subtracts it from the
point, and every draw adds it.

`+0x38` is the first subclass field. It is genuinely per-class: do not read a `+0x38` claim from one
widget family as a statement about another.

### 1a. The 53 widget classes

One typedef per factory tag (§6), named `W_<tag>`. A class of `0x38` bytes adds no field; its evidence
row lists which of the reused base fields its own functions touch. `'ltxt'` shares `'stxt'`'s
constructor and vtable.

```c
typedef struct W_stxt {  // 'stxt', 0x38 bytes: static text, left-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_stxt;
```

```c
typedef struct W_item {  // 'item', 0x38 bytes: menu item
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_item;
```

```c
typedef struct W_vidc {  // 'vidc', 0x138 bytes: key-binding row
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x100];  // [H] the key names of the row
} W_vidc;
```

`text[0x100]`: `gui_widget_key_binding_force_update` formats the row's key names into it.

```c
typedef struct W_wint {  // 'wint', 0x48 bytes: titled window — 40 px title bar + caption
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
    /* +0x40 */ int32_t drag_last_x;  // [H] last mouse x; dragging moves x and client_x
    /* +0x44 */ int32_t drag_last_y;  // [H] last mouse y
} W_wint;
```

```c
typedef struct W_ctxt {  // 'ctxt', 0x38 bytes: static text, centre-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_ctxt;
```

```c
typedef struct W_butt {  // 'butt', 0x38 bytes: button, centre-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_butt;
```

```c
typedef struct W_seli {  // 'seli', 0x38 bytes: selectable list item
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_seli;
```

```c
typedef struct W_butl {  // 'butl', 0x38 bytes: button, left-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_butl;
```

```c
typedef struct W_scrl {  // 'scrl', 0x38 bytes: float slider (0..1)
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_scrl;
```

```c
typedef struct W_mtxt {  // 'mtxt', 0x138 bytes: static text with its own buffer
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x100];  // [H] the widget's own copy of its text
} W_mtxt;
```

`text[0x100]`: `gui_widget_text_buffered_set_text` copies 255 characters and writes the NUL at `+0x137`.

```c
typedef struct W_ntxt {  // 'ntxt', 0x58 bytes: numeric static text
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] the formatted integer
} W_ntxt;
```

`text[0x20]`: `gui_widget_text_num_force_update` formats one integer into it, through `+0x30` when set.

```c
typedef struct W_htxt {  // 'htxt', 0x58 bytes: time static text (m:ss:mmm)
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] `m:ss:mmm`
} W_htxt;
```

`text[0x20]`: `gui_widget_text_time_force_update` formats the value of `+0x24` as `m:ss:mmm`.

```c
typedef struct W_rect {  // 'rect', 0x38 bytes: rectangle outline
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_rect;
```

```c
typedef struct W_line {  // 'line', 0x38 bytes: line
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_line;
```

```c
typedef struct W_netp {  // 'netp', 0xf4 bytes: multiplayer player row
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ uint32_t status_flags;  // [H] bit 0 ready, bit 1 AI racer (§2b)
    /* +0x3c */ char player_name[0x50];  // [H] strncpy 0x50
    /* +0x8c */ char car_name[0x50];  // [H] strncpy 0x50
    /* +0xdc */ uint8_t removable;  // [H] the host may remove this row: the right box shows an X (§2b)
    /* +0xe0 */ void *user_pointer;  // [H] slot 9 returns it
    /* +0xe4 */ const char *caption_player;  // [H] textdb 0x2c0
    /* +0xe8 */ const char *caption_car;  // [H] textdb 0x2c1
    /* +0xec */ const char *caption_ready;  // [H] textdb 0x2c2
    /* +0xf0 */ const char *caption_ai;  // [H] textdb 0x2c3
} W_netp;
```

`player_name` and `car_name` are `0x50` bytes because `gui_widget_net_player_set_row` uses `strncpy(.., 0x50)` and terminates the last byte.

```c
typedef struct W_comb {  // 'comb', 0x3c bytes: combo box (list supplied externally)
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ void *option_list;  // [H] supplied by the screen; ctor arm 0
} W_comb;
```

```c
typedef struct W_rtxt {  // 'rtxt', 0x38 bytes: static text, right-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_rtxt;
```

```c
typedef struct W_rctt {  // 'rctt', 0x3c bytes: render-target texture rect
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ void *texture;  // [H] released on a new SetText; ctor arm 0
} W_rctt;
```

```c
typedef struct W_ednu {  // 'ednu', 0x58 bytes: numeric edit box
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] the edited digits
} W_ednu;
```

`text[0x20]`: the same buffer as `'edit'`, read back through `crt_atoi`.

```c
typedef struct W_edit {  // 'edit', 0x58 bytes: text edit box, max 10 characters
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] the edited text; max 10 characters
} W_edit;
```

`text[0x20]`: the setter copies 31 characters and writes the NUL at `+0x57`.

```c
typedef struct W_wind {  // 'wind', 0x40 bytes: plain window — the family base, no chrome
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
} W_wind;
```

```c
typedef struct W_rctf {  // 'rctf', 0x3c bytes: file-textured rect, load mode 0
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ void *texture;  // [H] released on a new SetText; ctor arm 0
} W_rctf;
```

```c
typedef struct W_winc {  // 'winc', 0x40 bytes: window that draws nothing
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
} W_winc;
```

```c
typedef struct W_frac {  // 'frac', 0x48 bytes: framed panel with caption
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
    /* +0x40 */ int32_t drag_last_x;  // [H] last mouse x; dragging moves x and client_x
    /* +0x44 */ int32_t drag_last_y;  // [H] last mouse y
} W_frac;
```

```c
typedef struct W_trib {  // 'trib', 0x38 bytes: triangle arrow
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_trib;
```

```c
typedef struct W_frmt {  // 'frmt', 0x48 bytes: framed panel, no caption
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
    /* +0x40 */ int32_t drag_last_x;  // [H] last mouse x; dragging moves x and client_x
    /* +0x44 */ int32_t drag_last_y;  // [H] last mouse y
} W_frmt;
```

```c
typedef struct W_butr {  // 'butr', 0x38 bytes: button, right-aligned
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_butr;
```

```c
typedef struct W_rcta {  // 'rcta', 0x3c bytes: file-textured rect, load mode 2
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ void *texture;  // [H] released on a new SetText; ctor arm 0
} W_rcta;
```

```c
typedef struct W_swnd {  // 'swnd', 0x48 bytes: scrolling window
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
    /* +0x40 */ float scroll_current;  // [H] eased toward scroll_target by 2 x dt
    /* +0x44 */ float scroll_target;  // [H] clamped into [-(+0x28), 0] each tick
} W_swnd;
```

```c
typedef struct W_orez {  // 'orez', 0x40 bytes: clip region (Czech orezavac)
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
} W_orez;
```

```c
typedef struct W_view {  // 'view', 0x40 bytes: 3-D viewport marker — draws nothing
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float client_x;  // [H] x + parent's client_x
    /* +0x3c */ float client_y;  // [H] y + parent's client_y
} W_view;
```

```c
typedef struct W_rari {  // 'rari', 0x40 bytes: race-round item
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ const char *round_text;  // [H] chosen by the force update
} W_rari;
```

```c
typedef struct W_memi {  // 'memi', 0x138 bytes: menu item with its own buffer
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x100];  // [H] the widget's own copy of its text
} W_memi;
```

`text[0x100]`: the same as `'mtxt'`.

```c
typedef struct W_edtl {  // 'edtl', 0x58 bytes: text edit box, max 31 characters
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] the edited text; max 31 characters
} W_edtl;
```

`text[0x20]`: the same buffer as `'edit'`; only the key handler's cap differs.

```c
typedef struct W_list {  // 'list', 0x1038 bytes: wrapped multi-line text block
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x1000];  // [H] the wrapped lines; gui_widget_text_block_add_text stops at +0x808
} W_list;
```

`text[0x1000]` is the whole object past the base (`operator_new(0x1038)`), but `gui_widget_text_block_add_text` refuses to append past `+0x808`, so only the first `0x7d0` characters are ever used.

```c
typedef struct W_pbrt {  // 'pbrt', 0x38 bytes: progress bar with a visible track
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_pbrt;
```

```c
typedef struct W_lstc {  // 'lstc', 0x48 bytes: car carousel
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ void *car_list;  // [H] gui_widget_car_carousel_set_list
    /* +0x3c */ uint32_t index;  // [H] the selected row
    /* +0x40 */ float scroll_pos;  // [H] eased toward index x row height
    /* +0x44 */ uint32_t unread_44;  // [U] the ctor stores 0; no reader (.text scan)
} W_lstc;
```

```c
typedef struct W_iten {  // 'iten', 0x38 bytes: numeric menu item
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_iten;
```

```c
typedef struct W_butw {  // 'butw', 0x38 bytes: button, right-aligned, white
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_butw;
```

```c
typedef struct W_pull {  // 'pull', 0x48 bytes: resize grip
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ int32_t drag_last_x;  // [H] last mouse x
    /* +0x3c */ int32_t drag_last_y;  // [H] last mouse y
    /* +0x40 */ uint8_t unused_40[8];  // [U] no access; no reader (class census)
} W_pull;
```

The arm allocates `0x48` bytes; no function touches `unused_40`, the eight bytes after `drag_last_y`.

```c
typedef struct W_ltxt {  // 'ltxt', 0x38 bytes: alias of 'stxt' — same ctor, same vtable
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_ltxt;
```

```c
typedef struct W_butu {  // 'butu', 0x38 bytes: button, centre, white-on-red when focused
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_butu;
```

```c
typedef struct W_chck {  // 'chck', 0x38 bytes: check box
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_chck;
```

```c
typedef struct W_cmbs {  // 'cmbs', 0x44 bytes: combo box, options from a textdb id range
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ S_gui_option_list *option_list;  // [H] the widget's own rows, allocated by the ctor (§2b)
    /* +0x3c */ uint32_t first_text_id;  // [H] copied from the record's +0x20
    /* +0x40 */ uint32_t last_text_id;  // [H] copied from the record's +0x24
} W_cmbs;
```

```c
typedef struct W_scls {  // 'scls', 0x38 bytes: integer slider
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_scls;
```

```c
typedef struct W_scln {  // 'scln', 0x5c bytes: float slider showing its value
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char value_text[0x20];  // [H] the formatted value
    /* +0x58 */ float shown_value;  // [H] the value value_text holds; ctor arm -123.0 forces the first format
} W_scln;
```

`value_text[0x20]`: `gui_widget_slider_num_draw` reformats it when the value differs from `shown_value`.

```c
typedef struct W_sbig {  // 'sbig', 0x3c bytes: vertical scrollbar
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ int32_t drag_last_y;  // [H] last mouse y
} W_sbig;
```

```c
typedef struct W_joys {  // 'joys', 0x40 bytes: two-handle range on one axis — never built
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ int32_t drag_last_x;  // [H] last mouse x
    /* +0x3c */ float range_high;  // [H] the upper handle, 0..1; the lower is base +0x28
} W_joys;
```

```c
typedef struct W_rctv {  // 'rctv', 0x38 bytes: vertical two-colour gradient rect
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_rctv;
```

```c
typedef struct W_rctg {  // 'rctg', 0x38 bytes: horizontal two-colour gradient rect
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_rctg;
```

```c
typedef struct W_pbar {  // 'pbar', 0x38 bytes: progress bar
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
} W_pbar;
```

```c
typedef struct W_load {  // 'load', 0x174 bytes: savegame slot panel — never built
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ float thumb_rect[4];  // [H] x, y, width, height: 0, 8, 124, 94 scaled
    /* +0x48 */ float caption_pos[2];  // [H] 138, 8 scaled
    /* +0x50 */ float name_pos[2];  // [H] 138, 22 scaled
    /* +0x58 */ float line1_pos[2];  // [H] 138, 46 scaled
    /* +0x60 */ float line2_pos[2];  // [H] 138, 64 scaled
    /* +0x68 */ float line3_pos[2];  // [H] 138, 82 scaled
    /* +0x70 */ float caption_size;  // [H] 12 scaled
    /* +0x74 */ float name_size;  // [H] 22 scaled
    /* +0x78 */ float line_size;  // [H] 20 scaled
    /* +0x7c */ const char *caption;  // [H] textdb 0xad
    /* +0x80 */ char line1[0x50];  // [H] first text row under the name, drawn at line1_pos; no text writer (§6)
    /* +0xd0 */ char line2[0x50];  // [H] second text row, drawn at line2_pos; no text writer (§6)
    /* +0x120 */ char line3[0x50];  // [H] third text row, drawn at line3_pos; no text writer (§6)
    /* +0x170 */ void *thumbnail;  // [H] the render-target interface; ctor 0
} W_load;
```

`thumb_rect[4]` is one rectangle; the five `_pos[2]` are x, y pairs of the panel's text lines, in the order the draw uses them. `line1[0x50]`, `line2[0x50]` and `line3[0x50]` are the panel's three lower text rows: `gui_widget_save_slot_draw` renders them under the caption and the name, each at its own `_pos` pair and at `line_size`. Each is `0x50` bytes because the blocks abut at that stride, and both constructors store a NUL at `+0x80`, `+0xd0` and `+0x120`. No `'load'` widget is ever built (§6), so nothing writes their text.

```c
typedef struct W_edip {  // 'edip', 0x60 bytes: IPv4 address entry — never built
    /* +0x00 */ GM_Component base;  // [H] 0x38 bytes
    /* +0x38 */ char text[0x20];  // [H] twelve digit cells, ctor `*`
    /* +0x58 */ uint32_t unused_58;  // [U] no access; no reader (class census)
    /* +0x5c */ uint32_t cursor;  // [H] the next digit cell; ctor 0xc
} W_edip;
```

`text[0x20]` holds twelve digit cells (`cursor < 0xc`) and the ctor seeds fifteen `*`; the rest of the array is spare.

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the `gui_widget_` functions (ECX on
entry, followed through moves and stack slots), per class over the functions of its vtable and its
constructor; the `+0x38` fields of the window family come from the factory arms of
`gui_widget_create_from_design`. `every draw` means every slot-2 function that draws through the
parent's client origin.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | every arm of `gui_widget_create_from_design`, `gui_widget_ctor` | `gui_menu_tick`, `gui_menu_draw`, `gui_menu_tick_components`, `gui_menu_force_update_component` | - |
| +0x04 | id | H | `gui_widget_ctor` | `gui_menu_find_component_by_id`, `gui_widget_key_activate`, `gui_widget_click_activate` | returned on activation |
| +0x08 | flags | H | `gui_widget_ctor` (0), the factory arms | `gui_menu_set_component_enabled`, `gui_menu_set_component_visible`, `gui_menu_tick`, `gui_menu_draw` | §2a |
| +0x0c | text_id | H | `gui_widget_ctor`, `gui_menu_set_component_text_id` | `gui_menu_get_component_text_id`, `gui_widget_combo_key` | - |
| +0x10 | x | H | `gui_widget_ctor`, `gui_widget_window_drag_mouse`, `gui_widget_resize_grip_mouse` | `gui_widget_is_point_inside`, every draw | - |
| +0x14 | y | H | `gui_widget_ctor`, `gui_widget_window_drag_mouse`, `gui_widget_resize_grip_mouse` | `gui_widget_is_point_inside`, every draw | - |
| +0x18 | width | H | `gui_widget_ctor`, `gui_widget_resize_grip_mouse` | `gui_widget_is_point_inside`, every draw | - |
| +0x1c | height | H | `gui_widget_ctor`, `gui_widget_resize_grip_mouse` | `gui_widget_is_point_inside`, every draw | - |
| +0x20 | record_flags | H | `gui_widget_ctor`, `gui_widget_checkbox_key`, `gui_widget_combo_draw` | the draws of the text, rect and gradient classes (colour0) | per-class table below |
| +0x24 | record_color | H | `gui_widget_ctor`, `gui_widget_slider_int_key`, `gui_widget_item_num_key`, `gui_menu_set_component_dword` | the text and gradient draws (colour1), `gui_menu_get_component_dword` | - |
| +0x28 | record_color_f | H | `gui_widget_ctor`, `gui_widget_axis_range_mouse` | `gui_widget_rect_outline_draw`, `gui_widget_window_scroll_tick`, `gui_widget_car_carousel_draw` | `'rect'` reads it as its double-the-outline switch |
| +0x2c | value | H | `gui_widget_ctor`, the slider key and mouse handlers, `gui_menu_set_component_float` | the slider and bar draws, `gui_menu_get_component_float` | also the widget's boolean state |
| +0x30 | text | H | `gui_widget_ctor`, the text setters | every text draw | - |
| +0x34 | parent | H | `gui_widget_ctor` | `gui_widget_is_point_inside`, every draw | - |

### 2a. `flags` — the bits

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | selectable / enabled | the factory arms OR it in with `0x2` per class |
| `0x2` | takes the mouse | the factory arms of the sliders and the grip |
| `0x10` | focused | `gui_menu_tick` |
| `0x20` | wants a layout pass | set by `gui_widget_key_binding_key`, `gui_widget_key_binding_mouse`, `gui_widget_combo_range_key` and `gui_widget_combo_range_mouse`, cleared by `gui_widget_key_binding_force_update` and `gui_widget_combo_range_force_update`; tested by `gui_menu_tick` |
| `0x100` | hidden | `gui_menu_set_component_visible` |
| `0x1000` / `0x2000` | `'joys'`: which handle is being dragged | `gui_widget_axis_range_mouse` |
| `0x8000` | set on a `'frac'` panel: a text widget whose parent carries it draws in `0xffffff7f` with style 3 | set by the `'frac'` factory arm; tested on `parent->flags` by `gui_widget_text_left_draw`, `gui_widget_text_center_draw`, `gui_widget_text_right_draw`, `gui_widget_text_buffered_draw`, `gui_widget_text_num_draw` and `gui_widget_text_time_draw`; `gui_widget_frame_draw` tests no flag |
| `0x10000` | takes the mouse wheel: with a non-zero wheel delta `gui_menu_tick` runs the focused widget's slot 6 without a button | set by `gui_widget_car_carousel_ctor` (`flags \| 0x10001`); tested by `gui_menu_tick` |

### 2b. The evidence per class

**The client origin of the window family** (`'wint'`, `'wind'`, `'winc'`, `'frac'`, `'frmt'`, `'swnd'`, `'orez'`, `'view'`). `client_x` / `client_y` are read through `parent` by every child's draw, by `gui_widget_is_point_inside` and by the factory arms of nested windows. Besides the factory, the screen constructors that resize a frame write them: `gui_notice_panel_ctor`, `gui_message_box_ctor`, `gui_combo_dropdown_screen_ctor`, `gui_inventory_ctor`, `gui_action_menu_ctor`, `gui_profile_select_build_rows`, `gui_profile_delete_ctor`, `gui_session_list_ctor` and `gui_game_over_ctor`. Only the `'frac'` arm calls `gui_widget_frame_ctor`; the `'frmt'` arm calls `gui_widget_ctor` and seeds the origin inline.

`'stxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_left_draw` | own functions use `+0x20` read, `+0x24` written and read, `+0x30` read |

`'item'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_item_draw` | own functions use `+0x30` read |

`'vidc'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_key_binding_draw` | own functions use `+0x20` read, `+0x24` read, `+0x30` written and read; `+0x20` = the binding slot, `+0x24` non-zero = an axis row |
| +0x38 | text | H | `gui_widget_key_binding_ctor`, `gui_widget_key_binding_force_update` (by address) | `gui_widget_key_binding_ctor`, `gui_widget_key_binding_draw` (through `text`), `gui_widget_key_binding_force_update` | - |

`'wint'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_window_titled_draw` | own functions use `+0x20` read, `+0x30` read |
| +0x38 | client_x | H | `gui_widget_create_from_design`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x40 | drag_last_x | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |
| +0x44 | drag_last_y | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |

`'ctxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_center_draw` | own functions use `+0x20` read, `+0x24` written and read, `+0x30` read |

`'butt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_button_center_draw` | own functions use `+0x30` read |

`'seli'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_select_item_draw` | own functions use `+0x20` read, `+0x30` read |

`'butl'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_button_left_draw` | own functions use `+0x30` read |

`'scrl'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_slider_float_draw` | own functions use `+0x2c` written and read |

`'mtxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_buffered_draw` | own functions use `+0x20` read, `+0x24` read |
| +0x38 | text | H | `gui_widget_text_buffered_set_text` | `gui_widget_text_buffered_draw` (by address, `hud_text_draw_multiline`), `gui_widget_text_buffered_set_text` | - |

`'ntxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_num_draw` | own functions use `+0x20` read, `+0x24` read, `+0x30` read |
| +0x38 | text | H | `gui_widget_text_num_force_update` (by address, `crt_sprintf`) | `gui_widget_text_num_draw` (by address, `hud_text_draw_line`) | - |

`'htxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_time_draw` | own functions use `+0x20` read, `+0x24` read, `+0x30` read |
| +0x38 | text | H | `gui_widget_text_time_force_update` (by address, `crt_sprintf`) | `gui_widget_text_time_draw` (by address, `hud_text_draw_line`) | - |

`'rect'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_outline_draw` | own functions use `+0x20` read, `+0x28` read |

`'line'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_line_draw` | own functions use `+0x20` read |

`'netp'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_net_player_draw` | own functions use `+0x20` read: bit `0x2` = the header row (captions only), bit `0x1` = filled background |
| +0x38 | status_flags | H | `gui_widget_net_player_ctor`, `gui_widget_net_player_set_row` | `gui_widget_net_player_draw` | bit 0 draws `caption_ready` ("R", `0xff00ff00`), bit 1 `caption_ai` ("AI", `0xff1f1fff`); `gui_mp_lobby_tick` passes the peer's ready flag for a human row, 2 for an AI racer, 0 for an empty row |
| +0x3c | player_name | H | `gui_widget_net_player_ctor`, `gui_widget_net_player_set_row` | `gui_widget_net_player_draw`, `gui_widget_net_player_set_row` | - |
| +0x8c | car_name | H | `gui_widget_net_player_ctor`, `gui_widget_net_player_set_row` | `gui_widget_net_player_draw`, `gui_widget_net_player_set_row` | - |
| +0xdc | removable | H | `gui_widget_net_player_ctor`, `gui_widget_net_player_set_row` | `gui_widget_net_player_draw` | 1 only when this machine is the host and the row is not its own player (every AI row on the host): the draw puts an "X" and a double bevel into the right box, and a click there makes `gui_mp_lobby_tick` fetch `user_pointer` and call `net_session_remove_peer` or `net_session_remove_ai_racer` |
| +0xe0 | user_pointer | H | `gui_widget_net_player_ctor`, `gui_widget_net_player_set_row` | `gui_widget_net_player_get_pointer` | - |
| +0xe4 | caption_player | H | `gui_widget_net_player_ctor` | `gui_widget_net_player_draw` | - |
| +0xe8 | caption_car | H | `gui_widget_net_player_ctor` | `gui_widget_net_player_draw` | - |
| +0xec | caption_ready | H | `gui_widget_net_player_ctor` | `gui_widget_net_player_draw` | - |
| +0xf0 | caption_ai | H | `gui_widget_net_player_ctor` | `gui_widget_net_player_draw` | - |

`'comb'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_combo_draw` | own functions use `+0x20` written and read, `+0x30` read |
| +0x38 | option_list | H | `gui_menu_set_component_option_list`, `gui_widget_create_from_design` | `gui_widget_combo_key`, `gui_widget_combo_mouse` | - |

`'rtxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_right_draw` | own functions use `+0x20` read, `+0x24` written and read, `+0x30` read |

`'rctt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_tex_dynamic_draw` | own functions use `+0x30` read |
| +0x38 | texture | H | `gui_widget_create_from_design`, `gui_widget_rect_tex_dynamic_set_text` | `gui_widget_rect_tex_dynamic_draw`, `gui_widget_rect_tex_dynamic_force_update`, `gui_widget_rect_tex_dynamic_set_text` | - |

`'ednu'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_edit_number_draw` | own functions use `+0x30` written and read |
| +0x38 | text | H | `gui_widget_edit_number_set_value` | `gui_widget_edit_number_ctor`, `gui_widget_edit_number_key`, `gui_widget_edit_number_set_value` | - |

`'edit'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_edit_text_draw` | own functions use `+0x30` written and read |
| +0x38 | text | H | `gui_widget_edit_text_set_text` | `gui_widget_edit_text_ctor`, `gui_widget_edit_text_draw`, `gui_widget_edit_text_get_text`, `gui_widget_edit_text_key_max10`, `gui_widget_edit_text_set_text` | - |

`'wind'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_window_draw` | own functions read only the geometry and parent |
| +0x38 | client_x | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |

`'rctf'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_tex_draw` | own functions use `+0x20` read |
| +0x38 | texture | H | `gui_widget_create_from_design`, `gui_widget_rect_tex_file_dtor` | `gui_widget_rect_tex_draw`, `gui_widget_rect_tex_file_dtor`, `gui_widget_rect_tex_file_set_text` | - |

`'winc'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | - | own functions read only the geometry and parent |
| +0x38 | client_x | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |

`'frac'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_frame_draw` | own functions use `+0x20` read, `+0x30` read |
| +0x38 | client_x | H | `gui_widget_create_from_design`, `gui_widget_frame_ctor`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design`, `gui_widget_frame_ctor`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x40 | drag_last_x | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |
| +0x44 | drag_last_y | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |

`'trib'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_triangle_draw` | own functions use `+0x20` read |

`'frmt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_frame_draw` | own functions use `+0x20` read, `+0x30` read |
| +0x38 | client_x | H | `gui_widget_create_from_design`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design`, `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x40 | drag_last_x | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |
| +0x44 | drag_last_y | H | `gui_widget_window_drag_mouse` | `gui_widget_window_drag_mouse` | - |

`'butr'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_button_right_draw` | own functions use `+0x30` read |

`'rcta'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_tex_draw` | own functions use `+0x20` read |
| +0x38 | texture | H | `gui_widget_create_from_design` | `gui_widget_rect_tex_alpha_set_text`, `gui_widget_rect_tex_draw` | - |

`'swnd'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_window_scroll_draw` | own functions use `+0x28` read |
| +0x38 | client_x | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design`, `gui_widget_window_scroll_mouse`, `gui_widget_window_scroll_tick` | `gui_widget_window_scroll_mouse`, `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x40 | scroll_current | H | `gui_widget_window_scroll_tick` | `gui_widget_window_scroll_page_down`, `gui_widget_window_scroll_page_up`, `gui_widget_window_scroll_tick` | - |
| +0x44 | scroll_target | H | `gui_widget_window_scroll_by`, `gui_widget_window_scroll_page_down`, `gui_widget_window_scroll_page_up`, `gui_widget_window_scroll_tick`, `gui_widget_window_scroll_to_show` | `gui_widget_window_scroll_by`, `gui_widget_window_scroll_tick`, `gui_widget_window_scroll_to_show` | - |

`'orez'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_clip_draw` | own functions read only the geometry and parent |
| +0x38 | client_x | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |

`'view'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_viewport_draw` | own functions read only the geometry and parent |
| +0x38 | client_x | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |
| +0x3c | client_y | H | `gui_widget_create_from_design` | `gui_widget_create_from_design`, `gui_widget_is_point_inside`, every child's draw (through `parent`) | - |

`'rari'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_race_round_draw` | own functions use `+0x2c` read, `+0x30` read |
| +0x38 | round_text | H | `gui_widget_race_round_ctor`, `gui_widget_race_round_force_update` | `gui_widget_race_round_draw` | - |

`'memi'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_item_buffered_draw` | own functions use `+0x20` read |
| +0x38 | text | H | `gui_widget_text_buffered_set_text` | `gui_widget_item_buffered_draw` (by address, `hud_text_draw_line`), `gui_widget_text_buffered_set_text` | - |

`'edtl'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_edit_text_draw` | own functions use `+0x30` written and read |
| +0x38 | text | H | `gui_widget_edit_text_key_max31`, `gui_widget_edit_text_set_text` | `gui_widget_edit_text_ctor`, `gui_widget_edit_text_draw`, `gui_widget_edit_text_get_text`, `gui_widget_edit_text_key_max31`, `gui_widget_edit_text_set_text` | - |

`'list'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_block_draw` | own functions use `+0x20` written and read, `+0x24` read, `+0x30` read |
| +0x38 | text | H | `gui_widget_text_block_clear` | `gui_widget_text_block_add_text` | - |

`'pbrt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_progress_bar_track_draw` | own functions use `+0x20` read, `+0x2c` read |

`'lstc'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_car_carousel_draw` | own functions use `+0x20` written and read, `+0x28` written and read; `+0x20` = the shown row, which `gui_widget_car_carousel_tick` stores |
| +0x38 | car_list | H | `gui_widget_car_carousel_ctor`, `gui_widget_car_carousel_set_list` | `gui_widget_car_carousel_draw`, `gui_widget_car_carousel_mouse` | - |
| +0x3c | index | H | `gui_widget_car_carousel_ctor`, `gui_widget_car_carousel_mouse`, `gui_widget_car_carousel_set_index` | `gui_widget_car_carousel_mouse`, `gui_widget_car_carousel_tick` | - |
| +0x40 | scroll_pos | H | `gui_widget_car_carousel_ctor`, `gui_widget_car_carousel_tick` | `gui_widget_car_carousel_draw`, `gui_widget_car_carousel_mouse`, `gui_widget_car_carousel_tick` | - |
| +0x44 | unread_44 | U | `gui_widget_car_carousel_ctor` | - | the ctor stores 0; no reader (`.text scan`: of the 1,515 accesses at the displacement, the carousel's only one is that store, and no function that calls a carousel method touches `+0x44` of it) |

`'iten'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_item_num_draw` | own functions use `+0x24` written and read, `+0x30` read |

`'butw'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_button_right_white_draw` | own functions use `+0x30` read |

`'pull'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_resize_grip_draw` | own functions read only the geometry and parent |
| +0x38 | drag_last_x | H | `gui_widget_resize_grip_mouse` | `gui_widget_resize_grip_mouse` | - |
| +0x3c | drag_last_y | H | `gui_widget_resize_grip_mouse` | `gui_widget_resize_grip_mouse` | - |
| +0x40 | unused_40 | U | - | - | no access; no reader (class census) |

`'ltxt'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_text_left_draw` | own functions use `+0x20` read, `+0x24` written and read, `+0x30` read |

`'butu'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_button_highlight_draw` | own functions use `+0x30` read |

`'chck'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_checkbox_draw` | own functions use `+0x20` written and read, `+0x30` read |

`'cmbs'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_combo_draw` | own functions use `+0x20` written and read, `+0x24` read, `+0x30` written and read |
| +0x38 | option_list | H | `gui_widget_combo_range_ctor` | `gui_widget_combo_range_ctor`, `gui_widget_combo_range_dtor`, `gui_widget_combo_range_key`, `gui_widget_combo_range_mouse` | the ctor allocates the `0x10`-byte list (`structs/multiplayer.md` §1d) and pushes one 8-byte row `{label, text id}` per id from `first_text_id` to `last_text_id`; the dtor frees the rows and the list; key and mouse hand it to the drop-down |
| +0x3c | first_text_id | H | `gui_widget_combo_range_ctor` | `gui_widget_combo_range_ctor` | - |
| +0x40 | last_text_id | H | `gui_widget_combo_range_ctor` | `gui_widget_combo_range_ctor` | - |

`'scls'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_slider_int_draw` | own functions use `+0x20` read, `+0x24` written and read, `+0x2c` written and read |

`'scln'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_slider_num_draw` | own functions use `+0x2c` written and read |
| +0x38 | value_text | H | `gui_widget_slider_num_draw` (by address, `crt_sprintf`) | `gui_widget_slider_num_draw` | - |
| +0x58 | shown_value | H | `gui_widget_create_from_design`, `gui_widget_slider_num_draw` | `gui_widget_slider_num_draw` | - |

`'sbig'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_scrollbar_draw` | own functions use `+0x2c` written and read |
| +0x38 | drag_last_y | H | `gui_widget_scrollbar_mouse` | `gui_widget_scrollbar_mouse` | - |

`'joys'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_axis_range_draw` | own functions use `+0x28` written and read, `+0x2c` read |
| +0x38 | drag_last_x | H | `gui_widget_axis_range_mouse` | `gui_widget_axis_range_mouse` | - |
| +0x3c | range_high | H | `gui_widget_axis_range_mouse`, `gui_widget_create_from_design` | `gui_widget_axis_range_draw`, `gui_widget_axis_range_mouse` | - |

`'rctv'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_gradient_v_draw` | own functions use `+0x20` read, `+0x24` read |

`'rctg'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_rect_gradient_h_draw` | own functions use `+0x20` read, `+0x24` read |

`'pbar'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_progress_bar_draw` | own functions use `+0x2c` read |

`'load'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_save_slot_draw` | own functions use `+0x30` read |
| +0x38 | thumb_rect | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x48 | caption_pos | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x50 | name_pos | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x58 | line1_pos | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x60 | line2_pos | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x68 | line3_pos | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x70 | caption_size | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x74 | name_size | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x78 | line_size | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x7c | caption | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x80 | line1 | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0xd0 | line2 | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x120 | line3 | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw` | - |
| +0x170 | thumbnail | H | `gui_widget_save_slot_copy_ctor`, `gui_widget_save_slot_ctor` | `gui_widget_save_slot_draw`, `gui_widget_save_slot_dtor`, `gui_widget_save_slot_set_text` | - |

`'edip'`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `gui_widget_ctor`, `gui_widget_create_from_design` | `gui_widget_edit_ip_draw` | own functions use `+0x30` written |
| +0x38 | text | H | `gui_widget_edit_ip_ctor`, `gui_widget_edit_ip_key` | `gui_widget_edit_ip_ctor`, `gui_widget_edit_ip_draw` | - |
| +0x58 | unused_58 | U | - | - | no access; no reader (class census) |
| +0x5c | cursor | H | `gui_widget_edit_ip_ctor`, `gui_widget_edit_ip_key` | `gui_widget_edit_ip_draw`, `gui_widget_edit_ip_key` | - |

## 3. Method family

The `153` rows below hold every PC function of the prefixes; twins come from the registry, each re-checked
against the PS2 symbol table. The PS2's 402 widget methods have no one-to-one partner on the PC: §8a
gives the causes, §8e the classes with no counterpart, as groups rather than rows.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0056c080 | `gui_widget_scalar_deleting_dtor` | instance | `__dt__12GM_ComponentFv` | 0x002c3360 | CONFIRMED | base 0; base-class vtable slot 0 |
| 0x0056c0a0 | `gui_widget_frame_ctor` | instance | - | - | - | construct a framed panel and seed its client origin |
| 0x0056c0f0 | `gui_widget_rect_tex_file_scalar_deleting_dtor` | instance | - | - | - | `'rctf'` 0; vtable slot 0 for tag `'rctf'` |
| 0x0056c110 | `gui_widget_rect_tex_alpha_scalar_deleting_dtor` | instance | - | - | - | `'rcta'` 0; vtable slot 0 for tag `'rcta'` |
| 0x0056c140 | `gui_widget_rect_tex_dynamic_scalar_deleting_dtor` | instance | `__dt__19GM_RectangleTextureFv` | 0x002c7ee0 | CONFIRMED | `'rctt'` 0; vtable slot 0 for the `'rctt'` class |
| 0x0056c160 | `gui_widget_rect_tex_dynamic_dtor` | instance | `__dt__19GM_RectangleTextureFv` | 0x002c7ee0 | CONFIRMED | release the widget's texture and unwind the vtable |
| 0x0056c1b0 | `gui_widget_scalar_deleting_dtor_common` | instance | `__dt__12GM_ComponentFv` | 0x002c3360 | STRUCTURAL | the shared slot 0 of 46 widget vtables |
| 0x005704b0 | `gui_widget_ctor` | instance | `__ct__12GM_ComponentFP15GM_DesignStructP12GM_Component` | 0x002e3b20 | CONFIRMED | construct a widget from a `menu.def` record and a parent |
| 0x00570580 | `gui_widget_copy_ctor_below` | instance | `__ct__12GM_ComponentFP12GM_Component` | 0x002e3c40 | CONFIRMED | copy a widget and place the copy one row lower |
| 0x00570ec0 | `gui_widget_rect_tex_file_dtor` | instance | - | - | - | release a file-textured rectangle's texture |
| 0x005715a0 | `gui_widget_select_item_ctor` | instance | - | - | - | the `'seli'` selectable list row |
| 0x00571700 | `gui_widget_race_round_ctor` | instance | `__ct__16GM_RaceRoundItemFP15GM_DesignStructP12GM_Component` | 0x002c3cd0 | CONFIRMED | the `'rari'` championship-round row |
| 0x005718d0 | `gui_widget_text_ctor` | instance | `__ct__13GM_StaticTextFP15GM_DesignStructP12GM_Component` | 0x002c4590 | CONFIRMED | shared constructor of `'stxt'`, `'ltxt'`, `'ctxt'` and `'rtxt'` |
| 0x00572430 | `gui_widget_combo_range_ctor` | instance | - | - | - | `'cmbs'`: a combo whose options are a textdb id range |
| 0x00572690 | `gui_widget_combo_range_scalar_deleting_dtor` | instance | - | - | - | `'cmbs'` 0; `'cmbs'` vtable slot 0 |
| 0x005726b0 | `gui_widget_combo_range_dtor` | instance | - | - | - | `'cmbs'`: free the option vector and its holder |
| 0x005739f0 | `gui_widget_key_binding_ctor` | instance | - | - | - | `'vidc'`: one row of the *"Player Controls:"* screen |
| 0x00573f80 | `gui_widget_save_slot_ctor` | instance | - | - | - | `'load'`: one savegame slot panel |
| 0x00574180 | `gui_widget_save_slot_scalar_deleting_dtor` | instance | - | - | - | `'load'` 0; `'load'` vtable slot 0 |
| 0x005741a0 | `gui_widget_save_slot_copy_ctor` | instance | - | - | - | `'load'`: clone a slot panel from a template row |
| 0x00574350 | `gui_widget_save_slot_dtor` | instance | - | - | - | `'load'`: release the thumbnail, then chain to the base |
| 0x005745b0 | `gui_widget_net_player_ctor` | instance | `__ct__12GM_NetPlayerFP15GM_DesignStructP12GM_Component` | 0x002c3840 | CONFIRMED | `'netp'`: one row of the multiplayer player list |
| 0x00574680 | `gui_widget_net_player_scalar_deleting_dtor` | instance | - | - | - | `'netp'` 0; `'netp'` vtable slot 0 |
| 0x005746a0 | `gui_widget_net_player_dtor` | instance | - | - | - | `'netp'`: install the base vtable and stop |
| 0x00574b10 | `gui_widget_edit_text_ctor` | instance | `__ct__11GM_TextEditFP15GM_DesignStructP12GM_Component` | 0x002c3790 | CONFIRMED | shared constructor of `'edit'` and `'edtl'` |
| 0x00574fa0 | `gui_widget_edit_number_ctor` | instance | `__ct__10GM_EditNumFP15GM_DesignStructP12GM_Component` | 0x002c3500 | STRUCTURAL | `'ednu'`: the numeric entry box |
| 0x00575350 | `gui_widget_edit_ip_ctor` | instance | - | - | - | `'edip'`: the IPv4 address entry field |
| 0x005756f0 | `gui_widget_text_block_ctor` | instance | `__ct__16GM_StaticTextBigFP15GM_DesignStructP12GM_Component` | 0x002cbe50 | CONFIRMED | `'list'`: a word-wrapped multi-line text block |
| 0x005758d0 | `gui_widget_car_carousel_ctor` | instance | `__ct__11GM_ListCarsFP15GM_DesignStructP12GM_Component` | 0x002cc000 | CONFIRMED | `'lstc'`: the vertically scrolling car picker |
| 0x00575920 | `gui_widget_dtor` | instance | `__dt__12GM_ComponentFv` | 0x002c3360 | CONFIRMED | the widget base destructor |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00575800 | `gui_widget_text_block_clear` | `Clear__16GM_StaticTextBigFv` | 0x002cbe40 | CONFIRMED | `'list'`: empty the wrapped-text buffer |
| 0x00575be0 | `gui_widget_car_carousel_set_list` | `SetPointer__11GM_ListCarsFPv` | 0x002cd540 | CONFIRMED | `'lstc'`: hand the widget its vector of car names |
| 0x00575bf0 | `gui_widget_car_carousel_set_index` | `SetDword__11GM_ListCarsFUi` | 0x002cd550 | CONFIRMED | `'lstc'`: choose which row scrolls into view |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056afb0 | `gui_widget_create_from_design` | - | - | - | the factory: one arm per FourCC (§6); the other build's `G_Factory<T>` has one `Make` per class, so there is no single twin |
| 0x0056c060 | `stub_ret0_args8_0056c060` | - | - | - | the slot-5 default of the widget vtables: no key handled, returns 0 |
| 0x0056c070 | `stub_ret0_args20_0056c070` | - | - | - | the slot-6 default of the widget vtables: no mouse handled, returns 0 |
| 0x005705f0 | `gui_widget_is_point_inside` | `IsOverActive__12GM_ComponentFff` | 0x002c16b0 | CONFIRMED | hit-test a screen point against the widget rect |
| 0x00570680 | `gui_widget_window_draw` | - | - | - | `'wind'` 2; the chrome-less `'wind'` window: fill the client rect |
| 0x00570740 | `gui_widget_window_titled_draw` | `Draw__9GM_WindowFUi` | 0x002c6150 | CONFIRMED | `'wint'` 2; a window with a 40-pixel title bar and a caption |
| 0x005708f0 | `gui_widget_window_drag_mouse` | - | - | - | `'wint'` 6, `'frac'` 6, `'frmt'` 6 |
| 0x00570950 | `gui_widget_frame_draw` | - | - | - | `'frac'` 2, `'frmt'` 2; the `'frac'`/`'frmt'` framed panel with a corner ornament |
| 0x00570a90 | `gui_widget_window_scroll_draw` | `Draw__15GM_WindowScrollFUi` | 0x002c6830 | CONFIRMED | `'swnd'` 2; clip drawing to a scrolling window's client area |
| 0x00570b10 | `gui_widget_window_scroll_mouse` | - | - | - | `'swnd'` 6 |
| 0x00570b50 | `gui_widget_window_scroll_tick` | `Tick__15GM_WindowScrollFUi` | 0x002c6a60 | CONFIRMED | `'swnd'` 7; ease the scroll offset toward its target (slot 7) |
| 0x00570c10 | `gui_widget_window_scroll_page_up` | - | - | - | `'swnd'`: half of the split `Key__15GM_WindowScrollFUiUi` (§8d); the screens call it, slot 5 is the framework stub |
| 0x00570c20 | `gui_widget_window_scroll_page_down` | - | - | - | `'swnd'`: the other half of that split |
| 0x00570c30 | `gui_widget_window_scroll_by` | - | - | - | scroll a `'swnd'` window by a relative delta |
| 0x00570c40 | `gui_widget_window_scroll_to_show` | `Tick__15GM_WindowScrollFUi` | 0x002c6a60 | STRUCTURAL | nudge the scroll offset so a row is fully visible |
| 0x00570c90 | `gui_widget_clip_draw` | `Draw__11GM_OrezavacFUi` | 0x002c7ea0 | CONFIRMED | `'orez'` 2; set or reset the scissor rectangle (the clipper widget) |
| 0x00570d40 | `gui_widget_viewport_draw` | - | - | - | `'view'` 2; publish the widget's rect to the screen as the 3-D viewport |
| 0x00570d80 | `gui_widget_rect_outline_draw` | `Draw__12GM_RectangleFUi` | 0x002c7b40 | STRUCTURAL | `'rect'` 2; the rectangle widget's draw |
| 0x00570e30 | `gui_widget_rect_tex_draw` | - | - | - | `'rctf'` 2, `'rcta'` 2; shared draw of the two file-textured rectangle widgets |
| 0x00570e90 | `gui_widget_rect_tex_file_set_text` | - | - | - | `'rctf'` 8; load a named texture in mode 0 |
| 0x00570f20 | `gui_widget_rect_tex_alpha_set_text` | - | - | - | `'rcta'` 8; load a named texture in mode 2 |
| 0x00570f50 | `gui_widget_rect_gradient_v_draw` | - | - | - | `'rctv'` 2; the vertical-gradient rectangle widget's draw |
| 0x00570fa0 | `gui_widget_rect_gradient_h_draw` | - | - | - | `'rctg'` 2; the horizontal-gradient rectangle widget's draw |
| 0x00570ff0 | `gui_widget_rect_tex_dynamic_draw` | `Draw__19GM_RectangleTextureFUi` | 0x002c7ff0 | CONFIRMED | `'rctt'` 2; draw a render-target texture into the widget rect |
| 0x00571030 | `gui_widget_rect_tex_dynamic_force_update` | - | - | - | `'rctt'` 3; drop and reload the widget's texture (slot 3) |
| 0x00571070 | `gui_widget_rect_tex_dynamic_set_text` | `SetString__19GM_RectangleTextureFPUc` | 0x002c7f60 | CONFIRMED | `'rctt'` 8; create the widget's render target and load into it |
| 0x005710b0 | `gui_widget_line_draw` | `Draw__7GM_LineFUi` | 0x002c7a80 | CONFIRMED | `'line'` 2; the line widget's draw |
| 0x005710f0 | `gui_widget_resize_grip_draw` | - | - | - | `'pull'` 2; a window outline plus a corner resize handle |
| 0x00571170 | `gui_widget_resize_grip_mouse` | - | - | - | `'pull'` 6 |
| 0x00571250 | `gui_widget_item_draw` | `Draw__11GM_MenuItemFUi` | 0x002c8170 | CONFIRMED | `'item'` 2; the `'item'` menu row: centred label, grey when disabled |
| 0x005712c0 | `gui_widget_item_buffered_draw` | `Draw__14GM_MenuItemMemFUi` | 0x002c8340 | CONFIRMED | `'memi'` 2; the `'memi'` menu row, drawn from its own buffer |
| 0x00571330 | `gui_widget_item_num_draw` | `Draw__14GM_MenuItemNumFUi` | 0x002c8580 | CONFIRMED | `'iten'` 2; the `'iten'` numeric row: label, value, and minus/plus |
| 0x00571510 | `gui_widget_item_num_key` | `Key__14GM_MenuItemNumFUiUi` | 0x002c87e0 | CONFIRMED | `'iten'` 5; `'iten'` slot 5: left decrements by 2, everything else signals |
| 0x00571560 | `gui_widget_item_num_mouse` | - | - | - | `'iten'` 6 |
| 0x005715d0 | `gui_widget_select_item_draw` | - | - | - | `'seli'` 2; `'seli'` draw: skip a `~...~` prefix, highlight when focused |
| 0x005716a0 | `gui_widget_select_item_key` | - | - | - | `'seli'` 5; `'seli'` slot 5: activate on SPACE or ENTER when selectable |
| 0x005716d0 | `gui_widget_select_item_mouse` | - | - | - | `'seli'` 6; `'seli'` slot 6: click activates; id 0x7d3 also hovers |
| 0x00571770 | `gui_widget_race_round_force_update` | `SetBool__16GM_RaceRoundItemFb` | 0x002c8970 | CONFIRMED | `'rari'` 3; `'rari'` slot 3: recompute the round's status text |
| 0x005717c0 | `gui_widget_race_round_draw` | `Draw__16GM_RaceRoundItemFUi` | 0x002c8ad0 | CONFIRMED | `'rari'` 2; `'rari'` draw: label on the left, status on the right |
| 0x00571910 | `gui_widget_text_left_draw` | `Draw__13GM_StaticTextFUi` | 0x002c7080 | STRUCTURAL | `'stxt'` 2; `'stxt'`/`'ltxt'` draw, alignment constant 0 |
| 0x00571960 | `gui_widget_text_center_draw` | `Draw__13GM_StaticTextFUi` | 0x002c7080 | STRUCTURAL | `'ctxt'` 2; `'ctxt'` draw, alignment constant 2 |
| 0x005719b0 | `gui_widget_text_right_draw` | `Draw__13GM_StaticTextFUi` | 0x002c7080 | STRUCTURAL | `'rtxt'` 2; `'rtxt'` draw, alignment constant 4 |
| 0x00571a00 | `gui_widget_text_buffered_draw` | `Draw__16GM_StaticTextMemFUi` | 0x002c7380 | CONFIRMED | `'mtxt'` 2; `'mtxt'` draw, from the class's own 0x100-byte buffer |
| 0x00571ad0 | `gui_widget_text_buffered_set_text` | `SetText__16GM_StaticTextMemFPUc` | 0x002c7340 | CONFIRMED | `'mtxt'` 8, `'memi'` 8; slot 8 of `'mtxt'` and `'memi'`: copy into the buffer |
| 0x00571b00 | `gui_widget_text_num_force_update` | `SetDword__16GM_StaticTextNumFUi` | 0x002c7680 | CONFIRMED | `'ntxt'` 3; `'ntxt'` slot 3: reformat the stored number |
| 0x00571b30 | `gui_widget_text_num_draw` | `Draw__16GM_StaticTextNumFUi` | 0x002c76d0 | CONFIRMED | `'ntxt'` 2; `'ntxt'` draw: one line from the formatted buffer |
| 0x00571c00 | `gui_widget_text_time_force_update` | `SetDword__17GM_StaticTextTimeFUi` | 0x002c7830 | CONFIRMED | `'htxt'` 3; `'htxt'` slot 3: reformat the stored millisecond count |
| 0x00571c70 | `gui_widget_text_time_draw` | `Draw__17GM_StaticTextTimeFUi` | 0x002c7920 | CONFIRMED | `'htxt'` 2; `'htxt'` draw: one right-aligned line from the buffer |
| 0x00571d00 | `gui_widget_button_center_draw` | `Draw__9GM_ButtonFUi` | 0x002c8d00 | STRUCTURAL | `'butt'` 2; `'butt'` draw, alignment constant 2 |
| 0x00571d70 | `gui_widget_button_left_draw` | `Draw__9GM_ButtonFUi` | 0x002c8d00 | STRUCTURAL | `'butl'` 2; `'butl'` draw, alignment constant 0 |
| 0x00571de0 | `gui_widget_button_right_draw` | `Draw__9GM_ButtonFUi` | 0x002c8d00 | STRUCTURAL | `'butr'` 2; `'butr'` draw, alignment constant 4 |
| 0x00571e50 | `gui_widget_button_highlight_draw` | - | - | - | `'butu'` 2; `'butu'` draw: white, turning red when focused |
| 0x00571ec0 | `gui_widget_button_right_white_draw` | - | - | - | `'butw'` 2; `'butw'` draw: right-aligned, always white |
| 0x00571f20 | `gui_widget_triangle_draw` | `Draw__11GM_TriangleFUi` | 0x002c7d80 | STRUCTURAL | `'trib'` 2; the triangle-arrow widget's draw |
| 0x00571f60 | `gui_widget_activate_on_hover_mouse` | - | - | - | `'trib'` 6, `'memi'` 6; slot 6 variant: hovering is enough to activate |
| 0x00571f80 | `gui_widget_checkbox_draw` | `Draw__11GM_CheckBoxFUi` | 0x002ca520 | CONFIRMED | `'chck'` 2; `'chck'` draw: an `h*0.8` box, an `X`, and the label |
| 0x005720f0 | `gui_widget_checkbox_key` | `Key__11GM_CheckBoxFUiUi` | 0x002ca7d0 | CONFIRMED | `'chck'` 5; `'chck'` slot 5: SPACE toggles the state |
| 0x00572110 | `gui_widget_checkbox_mouse` | - | - | - | `'chck'` 6; `'chck'` slot 6: a click toggles the state |
| 0x00572130 | `gui_widget_combo_draw` | `Draw__11GM_ComboBoxFUi` | 0x002cc420 | CONFIRMED | `'comb'` 2, `'cmbs'` 2; shared draw of `'comb'` and `'cmbs'`: field, box and arrow |
| 0x005722e0 | `gui_widget_combo_key` | `Key__11GM_ComboBoxFUiUi` | 0x002cc650 | CONFIRMED | `'comb'` 5; `'comb'` slot 5: SPACE or ENTER opens the drop-down screen |
| 0x00572390 | `gui_widget_combo_mouse` | - | - | - | `'comb'` 6; `'comb'` slot 6: a click opens the same drop-down |
| 0x00572700 | `gui_widget_combo_range_force_update` | - | - | - | `'cmbs'` 3; `'cmbs'` slot 3: re-resolve the displayed string |
| 0x00572720 | `gui_widget_combo_range_key` | `Key__11GM_ComboBoxFUiUi` | 0x002cc650 | CONFIRMED | `'cmbs'` 5; `'cmbs'` slot 5: open the drop-down, then request a layout |
| 0x005727d0 | `gui_widget_combo_range_mouse` | - | - | - | `'cmbs'` 6; `'cmbs'` slot 6: a click opens the drop-down |
| 0x00572870 | `gui_widget_slider_float_draw` | `Draw__9GM_SliderFUi` | 0x002c9360 | CONFIRMED | `'scrl'` 2; `'scrl'` draw: two arrows, a fill bar and two outlines |
| 0x00572b10 | `gui_widget_slider_float_key` | `Key__9GM_SliderFUiUi` | 0x002c9620 | CONFIRMED | `'scrl'` 5; `'scrl'` slot 5: arrow keys nudge the value by 0.1 |
| 0x00572b80 | `gui_widget_slider_float_mouse` | - | - | - | `'scrl'` 6; `'scrl'` slot 6: click-and-drag, with an arrow hold-repeat |
| 0x00572c90 | `gui_widget_slider_num_draw` | `Draw__12GM_SliderNumFUi` | 0x002c97a0 | CONFIRMED | `'scln'` 2; `'scln'` draw: the slider plus its value as text |
| 0x00572e70 | `gui_widget_slider_num_key` | `Key__12GM_SliderNumFUiUi` | 0x002c9b30 | CONFIRMED | `'scln'` 5; `'scln'` slot 5: arrow keys nudge by 0.1, with re-clamping |
| 0x00572f10 | `gui_widget_slider_num_mouse` | - | - | - | `'scln'` 6; `'scln'` slot 6: click-and-drag with hold-repeat |
| 0x00572ff0 | `gui_widget_slider_int_draw` | `Draw__16GM_SliderIntegerFUi` | 0x002c9d50 | CONFIRMED | `'scls'` 2; `'scls'` draw: the slider snapped to an integer index |
| 0x005731b0 | `gui_widget_slider_int_key` | `Key__16GM_SliderIntegerFUiUi` | 0x002ca010 | CONFIRMED | `'scls'` 5; `'scls'` slot 5: arrow keys step the index by one, clamped |
| 0x00573230 | `gui_widget_slider_int_mouse` | - | - | - | `'scls'` 6; `'scls'` slot 6: click-and-drag over the integer range |
| 0x005733d0 | `gui_widget_scrollbar_draw` | - | - | - | `'sbig'` 2; `'sbig'` draw: a VERTICAL bar with a square knob |
| 0x00573490 | `gui_widget_scrollbar_mouse` | - | - | - | `'sbig'` 6 |
| 0x00573510 | `gui_widget_progress_bar_draw` | - | - | - | `'pbar'` 2; `'pbar'` draw: a value fill under a double bevel |
| 0x00573580 | `gui_widget_progress_bar_track_draw` | `Draw__14GM_ProgressBarFUi` | 0x002c80a0 | CONFIRMED | `'pbrt'` 2; `'pbrt'` draw: a full track, then the value over it |
| 0x00573600 | `gui_widget_axis_range_draw` | - | - | - | `'joys'` 2; `'joys'` draw: a symmetric two-handle range control |
| 0x00573860 | `gui_widget_axis_range_mouse` | - | - | - | `'joys'` 6 |
| 0x00573b50 | `gui_widget_key_binding_draw` | - | - | - | `'vidc'` 2; `'vidc'` draw: outline, label, and an arrow when focused |
| 0x00573c70 | `gui_widget_key_binding_force_update` | - | - | - | `'vidc'` 3; `'vidc'` slot 3: rebuild the label after a rebind |
| 0x00573da0 | `gui_widget_key_binding_key` | - | - | - | `'vidc'` 5; `'vidc'` slot 5: ENTER rebinds, BACKSPACE unbinds |
| 0x00573f00 | `gui_widget_key_binding_mouse` | - | - | - | `'vidc'` 6; `'vidc'` slot 6: a click opens the key-capture screen |
| 0x005743a0 | `gui_widget_save_slot_draw` | - | - | - | `'load'` 2; `'load'` draw: thumbnail box plus five text rows |
| 0x00574560 | `gui_widget_save_slot_set_text` | - | - | - | `'load'` 8; `'load'` slot 8: forward to the thumbnail interface |
| 0x005746b0 | `gui_widget_net_player_draw` | `Draw__12GM_NetPlayerFUi` | 0x002ca150 | CONFIRMED | `'netp'` 2; `'netp'` draw: the four-column multiplayer row |
| 0x005749f0 | `gui_widget_net_player_is_point_inside` | - | - | - | `'netp'` 4; `'netp'` slot 4: row test plus a column flag |
| 0x00574a40 | `stub_noop_cdecl_00574a40` | - | - | - | the no-op in slot 3 of `'netp'` and `'load'` and slot 1 of `'view'`; shared with other classes |
| 0x00574a50 | `gui_widget_key_activate` | `Key__9GM_ButtonFUiUi` | 0x002c8f40 | CONFIRMED | `'item'` 5, `'butt'` 5, `'butl'` 5, `'netp'` 5, `'trib'` 5, `'butr'` 5, `'rari'` 5, `'memi'` 5, `'butw'` 5, `'butu'` 5, `'load'` 5; the shared slot-5 handler: ENTER or SPACE activates |
| 0x00574a70 | `gui_widget_click_activate` | - | - | - | `'item'` 6, `'butt'` 6, `'butl'` 6, `'netp'` 6, `'butr'` 6, `'rari'` 6, `'butw'` 6, `'butu'` 6, `'load'` 6; the shared slot-6 handler: a left click activates |
| 0x00574a90 | `gui_widget_net_player_set_row` | - | - | - | `'netp'`: stores the status bits, the two names, the remove flag and the user pointer |
| 0x00574b00 | `gui_widget_net_player_get_pointer` | `GetPointer__12GM_NetPlayerFv` | 0x002ca140 | CONFIRMED | `'netp'` 9; `'netp'` slot 9: hand back the row's opaque pointer |
| 0x00574b70 | `gui_widget_edit_text_key_max10` | `Key__11GM_TextEditFUiUi` | 0x002cb0a0 | CONFIRMED | `'edit'` 5; `'edit'` slot 5: text entry capped at 10 characters |
| 0x00574cf0 | `gui_widget_edit_text_set_text` | `SetText__11GM_TextEditFPUc` | 0x002cb010 | CONFIRMED | `'edit'` 8, `'edtl'` 8; slot 8 of `'edit'`/`'edtl'`: replace the buffer |
| 0x00574d30 | `gui_widget_edit_text_get_text` | `GetText__11GM_TextEditFv` | 0x002cb060 | STRUCTURAL | `'edit'` 9, `'edtl'` 9; slot 9 of `'edit'` and `'edtl'`: copy the buffer out |
| 0x00574d60 | `gui_widget_edit_text_draw` | `Draw__11GM_TextEditFUi` | 0x002cb160 | CONFIRMED | `'edit'` 2, `'edtl'` 2; shared draw of `'edit'` and `'edtl'`, with a blinking caret |
| 0x00574e20 | `gui_widget_edit_text_key_max31` | `Key__11GM_TextEditFUiUi` | 0x002cb0a0 | CONFIRMED | `'edtl'` 5; `'edtl'` slot 5: text entry capped at 31 characters |
| 0x00575000 | `gui_widget_edit_number_draw` | `Draw__10GM_EditNumFUi` | 0x002cadb0 | STRUCTURAL | `'ednu'` 2; `'ednu'` draw: a filled, outlined field with the number in it |
| 0x005750a0 | `gui_widget_edit_number_key` | `Key__10GM_EditNumFUiUi` | 0x002caf00 | STRUCTURAL | `'ednu'` 5; `'ednu'` slot 5: type through the key-NAME table |
| 0x00575270 | `gui_widget_edit_number_set_value` | `SetDword__10GM_EditNumFUi` | 0x002cad50 | CONFIRMED | `'ednu'` 8; `'ednu'` slot 8: format an integer into the buffer |
| 0x005752b0 | `gui_widget_edit_number_get_value` | `GetDword__10GM_EditNumFv` | 0x002cada0 | STRUCTURAL | `'ednu'` 9; `'ednu'` slot 9: parse the buffer back to an integer |
| 0x005753b0 | `gui_widget_edit_ip_draw` | - | - | - | `'edip'` 2; `'edip'` draw: the dotted address, with a caret when focused |
| 0x00575540 | `gui_widget_edit_ip_key` | - | - | - | `'edip'` 5; `'edip'` slot 5: digits only, twelve of them |
| 0x00575780 | `gui_widget_text_block_draw` | `Draw__16GM_StaticTextBigFUi` | 0x002cdb60 | CONFIRMED | `'list'` 2; `'list'` draw: the wrapped buffer, red when flagged |
| 0x005757f0 | `stub_ret0_args20_005757f0` | - | - | - | slot 6 of the edit boxes and text blocks: they have no mouse handler |
| 0x00575810 | `gui_widget_text_block_add_text` | `AddText__16GM_StaticTextBigFPUc` | 0x002cbd30 | CONFIRMED | `'list'`: append text, word-wrapped, `~`-separated |
| 0x00575930 | `gui_widget_car_carousel_draw` | `Draw__11GM_ListCarsFUi` | 0x002cd650 | CONFIRMED | `'lstc'` 2; `'lstc'` draw: one arrow, then the clipped list of names |
| 0x00575a60 | `stub_ret0_args8_00575a60` | - | - | - | slot 5 of the scrollbars, ranges and the text block: no key handled |
| 0x00575a70 | `gui_widget_car_carousel_mouse` | - | - | - | `'lstc'` 6 |
| 0x00575b50 | `gui_widget_car_carousel_tick` | `Tick__11GM_ListCarsFUi` | 0x002cd570 | CONFIRMED | `'lstc'` 7; `'lstc'` slot 7: ease the scroll toward the target row |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056f680 | `gui_draw_line` | `Line__FffffUi` | 0x002e2cc0 | CONFIRMED | one 2-D line from (x, y) to (x + w, y + h) |
| 0x0056f720 | `gui_draw_rect_outline` | `Rectangle__FffffUi` | 0x002e2d50 | CONFIRMED | an unfilled rectangle drawn as a closed line strip |
| 0x0056f880 | `gui_draw_rect_filled` | `RectangleFilled__FffffUiP8ITexture` | 0x002e2e40 | CONFIRMED | primitive type 5, count 2, 4 verts; `(ECX = colour, EDX = texture, x, y, w, h)` (§7) |
| 0x0056f9a0 | `gui_draw_rect_filled_partial` | `RectangleProgress__FffffUifP8ITexture` | 0x002e2f10 | CONFIRMED | as `gui_draw_rect_filled`, right edge at `x + w * frac` |
| 0x0056fad0 | `gui_draw_rect_gradient_h` | - | - | - | two colours; colour1 on the right verts |
| 0x0056fbf0 | `gui_draw_rect_gradient_v` | - | - | - | two colours; colour1 on the bottom verts |
| 0x0056fd10 | `gui_draw_rect_double_bevel` | `RectangleBB__FffffUi` | 0x002e2fe0 | CONFIRMED | two nested 3-D bevel frames around a rectangle |
| 0x005701f0 | `gui_draw_triangle_dir` | `Triangle__FffffUiUi` | 0x002e34d0 | CONFIRMED | primitive type 3, count 1; direction 0..3, 0 = apex up, 1 = apex down |
| 0x00570400 | `gui_set_clip_rect` | `SetViewport__Fiiii` | 0x002e3710 | CONFIRMED | clamp a rect to the current mode and install it as the scissor |
| 0x005752d0 | `gui_format_ip_address` | - | - | - | expand a 12-digit buffer into dotted-quad form |

## 4. What it is

A `GM_Component` is one control inside a screen: a label, a button, a slider, a window
frame, a clip region. A screen (`GM_Menu`) owns a flat vector of them at `+0x20`/`+0x24`
and dispatches draw, layout and input down to each.

**53 concrete widget classes exist**, one per arm of the factory switch in
`gui_widget_create_from_design`, and each has its own 10-slot vtable in the contiguous
`.rdata` run `0x0063d59c … 0x0063ddfc`. The two counts match exactly, with no leftover on
either side — that is what makes the roster in §6 complete rather than merely long.

A widget is *not* a container on the PC. It has a `parent` pointer at `+0x34`, used only to
offset drawing and hit-testing by the parent's client origin.

## 5. The 10-slot vtable

Read each slot from the vtable in `.rdata`, never from the order of functions in the decomp (§5c).

| slot | byte offset | role | framework default |
|---|---|---|---|
| 0 | `+0x00` | scalar deleting destructor | `gui_widget_scalar_deleting_dtor_common` — shared by **46** of the 53 vtables |
| 1 | `+0x04` | rarely overridden; a no-op almost everywhere | an 8-byte return |
| 2 | `+0x08` | **Draw** | every class overrides it (`'winc'` deliberately with a no-op) |
| 3 | `+0x0c` | **ForceUpdate** — recompute derived text/geometry | no-op; reached from `gui_menu_force_update_component` |
| 4 | `+0x10` | **IsOverActive(x, y)** | `gui_widget_is_point_inside` in **52 of 53** — `'netp'` overrides it (§5a) |
| 5 | `+0x14` | **Key(scancode, ch)** → the widget id when activated, else 0 | a stub returning 0 |
| 6 | `+0x18` | **mouse(x, y, buttons, …)** — five arguments | a stub returning 0 |
| 7 | `+0x1c` | **Tick(dt)** — per-frame animation ¹ | a no-op |
| 8 | `+0x20` | SetText | a no-op |
| 9 | `+0x24` | **a GET accessor with an out-parameter** — see §5b | a no-op |

¹ Only `'swnd'` and `'lstc'` override slot 7; §8a gives what separates slot 7 (Tick) from
slot 6 (mouse).

**Two shared activation handlers** cover most classes: `gui_widget_key_activate` (slot 5,
scancodes `0x39` / `0x1c` / `0x9c`, 11 vtables) and `gui_widget_click_activate` (slot 6,
mouse bit `0x100`, 9 vtables). `gui_widget_activate_on_hover_mouse` is the slot-6 variant
that fires on the hover bit instead.

**Who drives each slot:** `gui_menu_tick` calls slot 6 on every widget with the mouse state
and slot 5 on the focused widget only; `gui_menu_tick_components` broadcasts slot 7 with the
frame delta; `gui_menu_draw` walks slot 2; `gui_menu_force_update_component` calls slot 3.

### 5a. One class overrides slot 4

`'netp'` (vtable `0x0063dd0c`) puts `gui_widget_net_player_is_point_inside` in slot 4. It calls
the shared `gui_widget_is_point_inside` first and then reports, in a second byte, which side of
`x + width - height` the point fell on — so the screen can tell a click on the row apart from a click on
the remove box. Every other vtable carries `0x005705f0`.

### 5b. Slot 9 is a getter, not `SetString`

The three classes that override it all put a **get** accessor there and the corresponding
**set** in slot 8:

| tag | slot 8 (set) | slot 9 (get) |
|---|---|---|
| `'edit'` / `'edtl'` | `gui_widget_edit_text_set_text` | `gui_widget_edit_text_get_text` |
| `'ednu'` | `gui_widget_edit_number_set_value` | `gui_widget_edit_number_get_value` |
| `'netp'` | - | `gui_widget_net_player_get_pointer` |

MSVC passes the result through an out-parameter, which is why these read as `void` functions
taking a pointer.

### 5c. Slot assignments that address order does not show

* `gui_widget_save_slot_set_text` sits in **slot 8**, not slot 3 — `'load'` slot 3 is
  `stub_noop_cdecl_00574a40`, so that class has no `ForceUpdate` at all. It asks the text
  database for textdb `0xae`, `0xaf` and `0xb0` and drops all three results before tail-jumping
  to the thumbnail interface. The lookup is a pure binary search with no side effect; `0xaf` is
  formatted by `gui_load_game_tick`, and `0xae` and `0xb0` are referenced nowhere else in the
  binary.
* `'pbar'` and `'pbrt'` carry stubs in slots 5 and 6, so the progress bars are inert.
* The edit boxes carry `stub_ret0_args20_005757f0` in slot 6, so they have **no mouse handler**.
* `'list'` carries a *different* no-op symbol in slot 7 (`stub_noop_args4_00461180`) — an ICF
  artifact, not an override, so "only `'swnd'` and `'lstc'` override slot 7" holds.

## 6. The class roster — the FourCC factory

`gui_widget_create_from_design` switches on the `menu.def` record's four-character tag. The
tag is the class identity and it is a **PC fact**: it is a literal in the switch and a field
in the shipped `tables/menu.def`.

**Uses** = occurrences in the shipped `tables/menu.def` (809 records, 72 designs). A count
of 0 means the shipped file never builds the class. Code builds ten tags through
`gui_menu_add_component_from_template` — `'htxt'`, `'item'`, `'memi'`, `'mtxt'`, `'ntxt'`,
`'pbar'`, `'pbrt'`, `'rari'`, `'seli'`, `'stxt'` — and nothing builds `'joys'`, `'edip'` or
`'load'` at all. `'joys'` and `'edip'` occur once in the image, as their factory compare, and each
vtable once, in its factory arm. `'load'` occurs three times, all compares: the factory arm and the
arms of `gui_menu_add_component_from_template` and `gui_menu_clone_component_under`; no caller
passes it and the shipped file has no such record.

| tag | size | vtable | what it is | uses |
|---|---|---|---|---|
| `'stxt'` | 0x38 | `0x0063dc54` | static text, **left**-aligned | 146 |
| `'item'` | 0x38 | `0x0063d9fc` | menu item | 132 |
| `'vidc'` | 0x138 | `0x0063dca4` | key-binding row | 59 |
| `'wint'` | 0x48 | `0x0063d59c` | **titled window** — 40 px title bar + caption | 53 |
| `'ctxt'` | 0x38 | `0x0063dac4` | static text, **centre**-aligned | 44 |
| `'butt'` | 0x38 | `0x0063dbb4` | button, centre-aligned | 29 |
| `'seli'` | 0x38 | `0x0063dc04` | selectable list item | 26 |
| `'butl'` | 0x38 | `0x0063db64` | button, **left**-aligned | 25 |
| `'scrl'` | 0x38 | `0x0063d6b4` | float slider (0..1) | 21 |
| `'mtxt'` | 0x138 | `0x0063d90c` | static text with its own buffer | 16 |
| `'ntxt'` | 0x58 | `0x0063d86c` | numeric static text | 12 |
| `'htxt'` | 0x58 | `0x0063d9d4` | **time** static text (`m:ss:mmm`) | 11 |
| `'rect'` | 0x38 | `0x0063d704` | rectangle outline | 10 |
| `'line'` | 0x38 | `0x0063d95c` | line | 10 |
| `'netp'` | 0xf4 | `0x0063dd0c` | multiplayer player row | 9 |
| `'comb'` | 0x3c | `0x0063da9c` | combo box (list supplied externally) | 9 |
| `'rtxt'` | 0x38 | `0x0063d72c` | static text, **right**-aligned | 8 |
| `'rctt'` | 0x3c | `0x0063d7cc` | render-target texture rect | 8 |
| `'ednu'` | 0x58 | `0x0063dd5c` | numeric edit box | 8 |
| `'edit'` | 0x58 | `0x0063dd34` | text edit box, max 10 characters ¹ | 7 |
| `'wind'` | 0x40 | `0x0063d9ac` | **plain window** — the family base, no chrome | 6 |
| `'rctf'` | 0x3c | `0x0063d77c` | file-textured rect, load mode 0 | 6 |
| `'winc'` | 0x40 | `0x0063d5c4` | window that draws nothing | 5 |
| `'frac'` | 0x48 | `0x0063da24` | framed panel with caption | 5 |
| `'trib'` | 0x38 | `0x0063d614` | triangle arrow | 4 |
| `'frmt'` | 0x48 | `0x0063da4c` | framed panel, no caption | 4 |
| `'butr'` | 0x38 | `0x0063db8c` | button, **right**-aligned | 3 |
| `'rcta'` | 0x3c | `0x0063d7f4` | file-textured rect, load mode 2 | 3 |
| `'swnd'` | 0x48 | `0x0063d63c` | scrolling window | 2 |
| `'orez'` | 0x40 | `0x0063d894` | clip region | 2 |
| `'view'` | 0x40 | `0x0063d5ec` | **3-D viewport marker** — draws nothing | 2 |
| `'rari'` | 0x40 | `0x0063dc2c` | race-round item | 2 |
| `'memi'` | 0x138 | `0x0063d8e4` | menu item with its own buffer | 2 |
| `'edtl'` | 0x58 | `0x0063da74` | text edit box, max 31 characters ¹ | 2 |
| `'list'` | 0x1038 | `0x0063ddac` | wrapped multi-line text block | 2 |
| `'pbrt'` | 0x38 | `0x0063d844` | progress bar with a visible track | 2 |
| `'lstc'` | 0x48 | `0x0063ddd4` | car carousel | 1 |
| `'iten'` | 0x38 | `0x0063d984` | numeric menu item | 1 |
| `'butw'` | 0x38 | `0x0063daec` | button, right-aligned, white | 1 |
| `'pull'` | 0x48 | `0x0063d81c` | resize grip | 1 |
| `'ltxt'` | 0x38 | `0x0063dc54` | **alias of `'stxt'`** — same ctor, same vtable | 0 |
| `'butu'` | 0x38 | `0x0063db3c` | button, centre, white-on-red when focused | 0 |
| `'chck'` | 0x38 | `0x0063db14` | check box | 0 |
| `'cmbs'` | 0x44 | `0x0063dc7c` | combo box, options from a textdb id range | 0 |
| `'scls'` | 0x38 | `0x0063d68c` | integer slider | 0 |
| `'scln'` | 0x5c | `0x0063d664` | float slider showing its value | 0 |
| `'sbig'` | 0x3c | `0x0063d6dc` | vertical scrollbar | 0 |
| `'joys'` | 0x40 | `0x0063d934` | two-handle range on one axis — **never built** | 0 |
| `'rctv'` | 0x38 | `0x0063d754` | **vertical** two-colour gradient rect | 0 |
| `'rctg'` | 0x38 | `0x0063d7a4` | **horizontal** two-colour gradient rect | 0 |
| `'pbar'` | 0x38 | `0x0063d8bc` | progress bar | 0 |
| `'load'` | 0x174 | `0x0063dce4` | savegame slot panel — **never built** | 0 |
| `'edip'` | 0x60 | `0x0063dd84` | IPv4 address entry — **never built** | 0 |
| - | 0x38 | `0x0063dbdc` | the `GM_Component` base itself | - |

¹ `gui_widget_edit_text_key_max10` caps at 10 characters, `gui_widget_edit_text_key_max31` at 31
(`0x1f`, the whole buffer); everything else — the strlen scan, backspace `0x0e`/`0xd3`, SPACE
`0x39`, the `char > 0x1f` append — is identical. `'edtl'` additionally swallows TAB (`0x0f`).

**Two tags in the shipped file have no factory arm**: `'gate'` (37 uses) and `'vers'`
(1 use). They are consumed by `gui_menu_build_components_from_design`, not instantiated —
`'gate'` is the conditional-inclusion marker that lets one design serve several screen
states (design `0x2a` repeats its `'butl'` rows behind four different gates).

**The text-alignment family is one class body, four tags.** `'stxt'`, `'ctxt'` and
`'rtxt'` share a constructor and differ only in the alignment constant they pass to the HUD
text renderer — **0 = left, 2 = centre, 4 = right**. `'ltxt'` shares `'stxt'`'s constructor
*and* vtable outright. The same 0/2/4 constants separate `'butl'`/`'butt'`/`'butr'`.

## 7. The drawing primitives

A contiguous block of nine free functions the widgets share, emitted in source order:

| function | primitive | notes |
|---|---|---|
| `gui_draw_line` | type 1, count 1 | 2 verts, `(x, y)` to `(x + width, y + height)` |
| `gui_draw_rect_outline` | type 2, 4 segments | 5 verts, closed strip |
| `gui_draw_rect_filled` | type 5, count 2 | 4 verts. **`(ECX = colour, EDX = texture, x, y, width, height)`**; the texture (0 = none) goes to `g_engine` slot `0xc` before the draw |
| `gui_draw_rect_filled_partial` | type 5, count 2 | right edge at `x + width * frac` |
| `gui_draw_rect_gradient_h` | type 5, count 2 | two colours; colour1 on the **right** verts |
| `gui_draw_rect_gradient_v` | type 5, count 2 | two colours; colour1 on the **bottom** verts |
| `gui_draw_rect_double_bevel` | type 1, count 4 — **twice** | shadow `0xaf000000`, highlight `0x3fffffff`; bit 0 = outer polarity, bit 1 = inner ¹ |
| `gui_draw_triangle_dir` | type 3, count 1 | direction 0..3; 0 = apex up, 1 = apex down |
| `gui_set_clip_rect` | - | clamp to the mode, then install as the scissor |

¹ Both polarity branches are live: `gui_widget_net_player_draw` has a three-way branch over the
polarity, which travels in ECX (§8f).

**The colour travels in ECX and the texture in EDX**, neither of which Ghidra renders.
Byte-proven: `mov ecx, 0xffbe002c` immediately precedes the `gui_draw_rect_filled` call inside
the progress-bar draw, with `xor edx, edx` (no texture) before it; the window draws load
`mov edx, [g_gui_papir3_texture]` and `or ecx, 0xffffffff` (white).

**The four window-art textures.** `hud_create` loads them and `hud_destroy` releases them, but
only the widgets draw them:

| global | file | drawn by |
|---|---|---|
| `g_gui_papir5a_texture` | `papir5a.tga` | `gui_widget_window_titled_draw`, `gui_widget_frame_draw` |
| `g_gui_papir3_texture` | `papir3.tga` | `gui_widget_window_draw`, `gui_widget_window_titled_draw`, `gui_widget_frame_draw` |
| `g_gui_sponka_texture` | `sponka.tga` (a paper clip) | `gui_widget_window_titled_draw`, `gui_widget_frame_draw` |
| `g_gui_kulicky_texture` | `kulicky.tga` (beads) | `gui_widget_progress_bar_track_draw` |

Every read is the EDX operand of a `gui_draw_rect_filled` call (`gui_draw_rect_filled_partial`
for the beads), so the decomp shows none of them;
they were located by a scan of `.text` for the four addresses.

## 8. PC vs PS2

The widget layer exists on both builds as `GM_Component`, but the PS2 splits it across three
source files (`g_m_comps.cpp`, `g_m_comps2.cpp`, `g_m_compsm.cpp`) where the PC has one.

### 8a. Why the PS2 has ~2.9x more functions

**402 widget-class methods across 52 classes on the PS2**, plus **156** `G_Factory<T>`
members; **~140 real functions for 53 classes** on the PC. None of the gap is a feature
difference:

| cause | PS2 cost | PC cost |
|---|---|---|
| two constructors per class (design-record + copy) | ~104 | 2 shared |
| `GetType` per class, 12 bytes each, returning the FourCC | 52 | 0 — the tag is a literal in the switch |
| a destructor per class | ~52 | 2 shared across 46 vtables (ICF) |
| `G_Factory<T>::{ctor,Make,MakeDup}` per class | 156 | 3 (the factory + the two `Pridej*` twins) |
| every value accessor a per-class virtual (`SetDword`, `GetBool`, `SetRangeFloat`, …) | large | 0 — the PC writes a shared field, so `GM_Menu`'s ~20 accessors serve all 53 classes |
| `CanBeParent` / `GetClientOrigin` overrides | ~10 | 0 |

The slot-9 getter / slot-8 setter pairs of §5b are spelled `SetText`/`GetText`,
`SetDword`/`GetDword`, `SetPointer`/`GetPointer` on the PS2.

The two slot-7 overriders of §5, `'swnd'` and `'lstc'`, are exactly the two classes that have a
`Tick` on the PS2, which is what separates slot 7 (Tick) from slot 6 (mouse).

### 8b. The three real behavioural differences

1. **The PS2 widget is a composite; the PC's is a leaf.** `__ct__12GM_ComponentFP15GM_DesignStructP12GM_Component` shows a **child vector at PS2 `+0x2c`/`+0x30`/`+0x34`** that the widget self-inserts into on construction, driving a recursive layer — `BaseDraw`, `BaseAfterMove`, `InsertComponent`, `InsertComponentAfter`, `AfterMove`, `CanBeParent` and per-class `GetClientOrigin`. The PC has **no child vector**: one flat vector on the screen, and `+0x34` used only to offset by the parent's client origin. That whole PS2 layer has no PC counterpart.
2. **The scrollbar is a different object.** PS2 `GM_WindowScroll::Draw` draws its own bar — knob at `(cur/range) * (height - 3*barwidth)`, track outline and two arrow triangles — then sets the viewport. The PC's `'swnd'` draw is *only* the viewport call: the bar is composed from separate `'sbig'` and two `'trib'` widgets the designer places, visible in `menu.def` design `0x0c`.
3. **The pointer.** The PS2 has pointer plumbing at *screen* level (`GM_Menu::Tick(x, y, …)`, `OnMouse`) but `GM_Component`'s complete 36-method roster has **no mouse entry point**. Slot 6 and every handler in it is a PC addition.

### 8c. Three tag inversions — a FourCC is per-build vocabulary

**The two builds give the same two tags opposite meanings.** PC `'wint'` is the twin of PS2
`GM_Window` — proven by the shared `(y + 40*scale) - 1.0` body split, the `width` / `height - 40`
strips, the centred caption and the `0xff7f7f7f`/`0xffffffff` colour pair, plus the
identical `y + scaleY*40.0` client origin. The PS2's own `'wint'` is `GM_WindowTutorial`, a
different class. The PC **split the PS2 base in two**: a chrome-less `'wind'` (0x40) and a titled
`'wint'` (0x48), so matching by tag here produces two wrong twins.

Two more inversions are visible only by reading the other build's body:

| tag | PC class | PS2 class wearing the same tag | the PS2 class that actually matches |
|---|---|---|---|
| `'pbar'` | value fill under a double bevel — **PC-only** | `GM_ProgressBar` | - |
| `'pbrt'` | track at 1.0, then the value | - | **`GM_ProgressBar`** |
| `'list'` | word-wrapped text block | `GM_ListBox` ¹ | **`GM_StaticTextBig`** (`'stxb'`) ² |

¹ A scrolling row list with its own scrollbar and page keys.
² Matched by shape — a class with `Clear` + `AddText` — not by tag.

⇒ **Enumerate by tag; decide the twin by behaviour.** Eight further pairs are the same class
under different tags (`trib`/`tria`, `ednu`/`edin`, `swnd`/`wins`, `scrl`/`slid`, `scls`/`sldi`,
`scln`/`slin`, `memi`/`itmm`, `list`/`stxb`), so a tag join alone also produces false absences.

### 8d. Folds, in both directions

| direction | what |
|---|---|
| 2 → 1 | `gui_widget_clip_draw` absorbs `GM_Orezavac::Draw` **and** `GM_UpUp::Draw` ¹ behind a `width < 0` sentinel |
| 3 → 1 | `gui_menu_execute_modal` absorbs `ExecuteMenu` + `TickCamera` + `SetCameraTargetID` |
| 2 → 1 | `gui_widget_is_point_inside` absorbs `IsOverClientArea` and `GetClientOrigin` |
| 1 → 2 | PS2 `Key__15GM_WindowScrollFUiUi` becomes `gui_widget_window_scroll_page_up` / `_page_down`; PC `'swnd'` slot 5 is the framework stub, so the screens drive scroll keys |
| 1 → 2 | the tail of PS2 `Tick__15GM_WindowScrollFUi` becomes `gui_widget_window_scroll_to_show` |
| 1 → 2 | every GCC dtor-with-delete-flag becomes an MSVC dtor + scalar-deleting-dtor pair |

¹ An entire PS2 class whose only body is "reset the clip".

Each of the first three is marked with a `// PS2-INLINED HERE:` comment at the fold site in
the annotated decomp.

### 8e. Classes with no counterpart

**PS2-only:** the console families (`GM_PSX*` memory card, `GM_PS2Help`/`Help2`,
`GM_Karusel`, `GM_ColorSel`, `GM_MultiButton`, `GM_TextInput`, `GM_WindowM`,
`GM_StaticTextBig2Color`/`Fit`/`Mem2`, `GM_ButtonClean`/`Shrink`/`Select`/`SelectBool`,
`GM_ComboWindow`, `GM_ViewScroll`, `GM_PSXTextEdit`, `GM_RectangleB`, `GM_ListBox`,
`GM_WindowTutorial`) plus two that explain themselves: **`GM_Nahrada`** (tag `'xxxx'`, a substitute) draws a green outline and a green label — the placeholder for an unknown
design tag, where the PC switch simply returns 0; and **`GM_Mapa`** (`'mapa'`) forwards to
`DrawMap__17G_IndicatorsClass`, the PS2 pause map. `GM_StaticTextBig` is not PS2-only — it is
the twin of PC `'list'` (§8c).

**PC-only**, each measured against the PS2's complete 52-entry `GetType` roster rather than
by a failed grep: `'edip'` (and `gui_format_ip_address` — the PS2 decomp contains **zero**
`%d.%d.%d.%d` format strings), `'joys'`, `'sbig'`, `'vidc'`, `'pull'`, `'view'`, `'load'`,
`'cmbs'`, `'pbar'`, `'seli'`, the four texture/gradient rect variants
`'rctf'`/`'rcta'`/`'rctg'`/`'rctv'`, the framed panels `'frac'`/`'frmt'`, the chrome-less
`'wind'`, and every slot-6 mouse handler. All five PS2 window classes' `Draw` are accounted for,
so `'wind'`, `'frac'` and `'frmt'` having no PS2 twin is a measurement (`GM_WindowM` was read
and rejected).

**PC-only classes whose code is a PS2 body** — the 1 → N alignment fold.
`Draw__13GM_StaticTextFUi` and `Draw__9GM_ButtonFUi` pick align 0/2/4 at runtime from flag bits
`0x40`/`0x80`; the PC compiled the constant in and shipped one class per alignment. So
`'ctxt'`, `'rtxt'`, `'ltxt'`, `'butl'`, `'butr'`, `'butu'` and `'butw'` have no PS2 class and no
own PS2 body — the twin line sits on the `'stxt'` and `'butt'` realisations. The same shape gives
`'edit'`/`'edtl'` (one PS2 class with the length cap in a field at `+0x7c`, two PC classes with
the cap compiled in) and `'comb'`/`'cmbs'`.

**How the absences are measured.** Not by a name search. All 1153 PS2 symbols in
`[0x002b8000, 0x002f0000)` are enumerated, **including the 21 with an empty class column**; the
only widget-relevant free functions there are the eight drawing primitives of §7 plus
`g_InitCompsm__Fv`. The body of every plausible alternative class is read:

| PC tag | PS2 classes read and rejected (or matched) |
|---|---|
| `'seli'`, `'cmbs'` | `GM_ButtonSelect`, `GM_ButtonSelectBool`, `GM_ButtonClean`, `GM_ButtonShrink`, `GM_MultiButton` |
| `'list'` | `GM_StaticTextFit`, `GM_StaticTextBig` (the twin), `GM_StaticTextMem2` |
| `'joys'` | `GM_ColorSel`, `GM_Karusel` |
| `'sbig'` | `GM_ViewScroll`, `GM_WindowScroll` |
| `'wind'`, `'frac'`, `'pbar'` | `GM_WindowM`, `GM_WindowTutorial`, `GM_Rectangle`, `GM_RectangleB` |
| `'ednu'`, `'edip'` | `GM_PSXTextEdit`, `GM_TextInput` |
| `'view'` | `GM_ViewScroll` (`'vscr'`) ¹ |

¹ It has **no `Draw` at all** — only a constructor, destructor, `GetType` and `GetRangeFloat` —
so it is not the viewport marker.

`'pbar'` additionally rests on a whole-binary callsite census of the bevel primitive: PS2
`RectangleBB` has exactly three callsites, all inside `Draw__12GM_NetPlayerFUi`.

### 8f. The double bevel and the scroll window

**`gui_draw_rect_double_bevel`'s polarity.** PS2 `RectangleBB`'s three callsites in
`Draw__12GM_NetPlayerFUi` pass **1, 0 and 3**, and `gui_widget_net_player_draw` has the same
three-way branch. The PC polarity travels in ECX, which Ghidra does not render, so the five PC
callsites do not show it in the decomp.

**`gui_widget_window_scroll_by` (`0x00570c30`) — CONFIRMED, PC-only.** It has **zero vtable
references and exactly one callsite**: the mouse arm of `gui_load_game_tick`, passing the
**mouse-wheel delta**. The PS2 has no wheel and `GM_WindowScroll`'s complete 14-method roster has
no relative adjust, so the missing twin is a measurement. It does not clamp itself:
`gui_widget_window_scroll_tick` clamps `+0x44` into `[-(+0x28), 0]` at the top of every frame.
`SetFloat__15GM_WindowScrollFf` is not its counterpart — it is an absolute setter that the PS2
reaches from `Tick`'s tail through vtable `+0x50`, the half the PC out-lines as
`gui_widget_window_scroll_to_show`. PS2 `Tick__15GM_WindowScrollFUi` eases its own `+0x44`
toward `+0x40` and then runs the keep-visible guards through `SetFloat`, which also confirms the
`gui_widget_window_scroll_tick` twin — the 1 → 2 split of §8d.

**The scroll-window field roles are swapped across the builds:**

| | PS2 `GM_WindowScroll` | PC `'swnd'` |
|---|---|---|
| range | `+0x3c` (`SetRangeFloat`) | **`+0x28`** |
| target | `+0x40` | **`+0x44`** |
| current | `+0x44` | **`+0x40`** |
| key step | `+0x48` | compiled-in `10.0` |

`+0x28` is proven independently by `gui_load_game_ctor`, which writes it from the last row's
geometry exactly where the PS2 calls the widget's `SetRangeFloat`.

### 8g. The car carousel's fourth word

`__ct__11GM_ListCarsFP15GM_DesignStructP12GM_Component` zeroes four words: the list, the target
row, the shown row and the scroll position. The PC keeps the shown row in the base's `+0x20`
(`gui_widget_car_carousel_tick` stores it there), so its fourth word, `unread_44`, has no use.

## 9. Related

`structs/gui_screen.md` (the screen class, the unit's host) · `structs/game_menus.md`,
`structs/multiplayer.md`, `structs/racing_menus.md` (the screens) · `structs/hud.md` (the text renderer)
· `def-files-docs` (`tables/menu.def`).

## 10. Open items

1. **In game: the hover-activated `'seli'` row.** `gui_widget_select_item_mouse` activates a row whose
   textdb id is `0x7d3` on the hover bit `0x10` instead of the click bit `0x100`. No shipped design
   carries that id; `gui_action_menu_ctor` gives each option row its action's textdb id, and `0x7d3` is
   an action code of the car seats (`car_set_owner`), so the row is that action in the action menu
   (`structs/game_menus.md` §7d). What the player sees when hovering it is not observed.
