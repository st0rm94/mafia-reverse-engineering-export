# `GM_Component` — the GUI widget base and its 53 subclasses

**Created** 2026-08-24 22:00 +0200 · **Modified** 2026-09-24 21:26 +0200

Companion to `structs/gui_screen.md`, which documents the *screen* class `GM_Menu`; this file
documents the *widgets* a screen owns. The TU that follows this one — the ~30 `GM_Menu` screen
classes that use these widgets — is documented in `structs/game_menus.md`.

**Prefix:** `gui_widget_` for the base class and every subclass method
(`gui_widget_ctor`, `gui_widget_window_titled_draw`, …). The free drawing primitives the
widgets share use `gui_draw_*` / `gui_set_clip_rect`.

**Where it lives:** the same translation unit as the framework (`g_menu.cpp`) —
`[0x0056adf0, 0x00575c20)`, every function start named. The widget half runs from about
`0x0056f680` to the end of the range.

---

## 1. What it is

A `GM_Component` is one control inside a screen: a label, a button, a slider, a window
frame, a clip region. A screen (`GM_Menu`) owns a flat vector of them at `+0x20`/`+0x24`
and dispatches draw, layout and input down to each.

**53 concrete widget classes exist**, one per arm of the factory switch in
`gui_widget_create_from_design`, and each has its own 10-slot vtable in the contiguous
`.rdata` run `0x0063d59c … 0x0063ddfc`. The two counts match exactly, with no leftover on
either side — that is what makes the roster in §4 complete rather than merely long.

A widget is *not* a container on the PC. It has a `parent` pointer at `+0x34`, used only to
offset drawing and hit-testing by the parent's client origin.

## 2. Field map (measured)

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | vtable — 10 slots, see §3 | written by every factory arm |
| `+0x04` | the widget **id** — the `menu.def` record's id | matched by every accessor; returned by the key/mouse handlers to signal activation |
| `+0x08` | **flags**: bit 0 selectable/enabled · `0x10` focused · `0x100` hidden · `0x20` wants a layout pass | zeroed by `gui_widget_ctor`; the factory arms OR in `1` or `3` per class |
| `+0x0c` | textdb string id | copied from the record; resolved into `+0x30` |
| `+0x10` / `+0x14` | x / y in screen pixels | scaled by `gui_widget_ctor` |
| `+0x18` / `+0x1c` | w / h in screen pixels | scaled by `gui_widget_ctor`; `gui_widget_is_point_inside` tests all four |
| `+0x20` | the record's flags word, reused per class as a colour or a boolean | `gui_widget_rect_gradient_h_draw` reads it as colour0 |
| `+0x24` | the record's colour word, reused per class as an integer value | the numeric static text formats it; the gradients read it as colour1 |
| `+0x28` | the same colour word as a float | written by `gui_widget_ctor`; `'rect'` uses it as its "double the outline" switch |
| `+0x2c` | a float value — and the widget's boolean state | the sliders' 0..1 value; the progress bars' fill |
| `+0x30` | the resolved `const char *` the widget renders | written by the ctor and by the text setters |
| `+0x34` | **the parent widget**, or 0 | `gui_widget_is_point_inside` and every draw add the parent's client origin |
| `+0x38` | **first subclass field.** The base class is exactly 0x38 bytes | the window family puts its client-origin x here; the texture classes their texture; the combos their option list |

`+0x38` is genuinely per-class. Do not read a `+0x38` claim from one widget family as a
statement about another.

## 3. The 10-slot vtable

Read each slot from the vtable in `.rdata`, never from the order of functions in the decomp (§3c).

| slot | byte offset | role | framework default |
|---|---|---|---|
| 0 | `+0x00` | scalar deleting destructor | `gui_widget_scalar_deleting_dtor_common` — shared by **46** of the 53 vtables |
| 1 | `+0x04` | rarely overridden; a no-op almost everywhere | an 8-byte return |
| 2 | `+0x08` | **Draw** | every class overrides it (`'winc'` deliberately with a no-op) |
| 3 | `+0x0c` | **ForceUpdate** — recompute derived text/geometry | no-op; reached from `gui_menu_force_update_component` |
| 4 | `+0x10` | **IsOverActive(x, y)** | `gui_widget_is_point_inside` in **52 of 53** — `'netp'` overrides it (§3a) |
| 5 | `+0x14` | **Key(scancode, ch)** → the widget id when activated, else 0 | a stub returning 0 |
| 6 | `+0x18` | **mouse(x, y, buttons, …)** — five arguments | a stub returning 0 |
| 7 | `+0x1c` | **Tick(dt)** — per-frame animation ¹ | a no-op |
| 8 | `+0x20` | SetText | a no-op |
| 9 | `+0x24` | **a GET accessor with an out-parameter** — see §3b | a no-op |

¹ Only `'swnd'` and `'lstc'` override slot 7 — exactly the two classes that have a `Tick` on the
PS2 (§6), which is what separates slot 7 (Tick) from slot 6 (mouse).

**Two shared activation handlers** cover most classes: `gui_widget_key_activate` (slot 5,
scancodes `0x39` / `0x1c` / `0x9c`, 11 vtables) and `gui_widget_click_activate` (slot 6,
mouse bit `0x100`, 9 vtables). `gui_widget_activate_on_hover_mouse` is the slot-6 variant
that fires on the hover bit instead.

**Who drives each slot:** `gui_menu_tick` calls slot 6 on every widget with the mouse state
and slot 5 on the focused widget only; `gui_menu_tick_components` broadcasts slot 7 with the
frame delta; `gui_menu_draw` walks slot 2; `gui_menu_force_update_component` calls slot 3.

### 3a. One class overrides slot 4

`'netp'` (vtable `0x0063dd0c`) puts `gui_widget_net_player_is_point_inside` in slot 4. It calls
the shared `gui_widget_is_point_inside` first and then reports, in a second byte, which side of
`x + w - h` the point fell on — so the screen can tell a click on the row apart from a click on
the ready box. Every other vtable carries `0x005705f0`.

### 3b. Slot 9 is a getter, not `SetString`

The three classes that override it all put a **get** accessor there and the corresponding
**set** in slot 8:

| tag | slot 8 (set) | slot 9 (get) |
|---|---|---|
| `'edit'` / `'edtl'` | `gui_widget_edit_text_set_text` | `gui_widget_edit_text_get_text` |
| `'ednu'` | `gui_widget_edit_number_set_value` | `gui_widget_edit_number_get_value` |
| `'netp'` | - | `gui_widget_net_player_get_pointer` |

MSVC passes the result through an out-parameter, which is why these read as `void` functions
taking a pointer.

### 3c. Slot assignments that address order does not show

* `gui_widget_save_slot_set_text` sits in **slot 8**, not slot 3 — `'load'` slot 3 is
  `stub_noop_cdecl_00574a40`, so that class has no `ForceUpdate` at all.
* `'pbar'` and `'pbrt'` carry stubs in slots 5 and 6, so the progress bars are inert.
* The edit boxes carry `stub_ret0_args20_005757f0` in slot 6, so they have **no mouse handler**.
* `'list'` carries a *different* no-op symbol in slot 7 (`stub_noop_args4_00461180`) — an ICF
  artifact, not an override, so "only `'swnd'` and `'lstc'` override slot 7" holds.

## 4. The class roster — the FourCC factory

`gui_widget_create_from_design` switches on the `menu.def` record's four-character tag. The
tag is the class identity and it is a **PC fact**: it is a literal in the switch and a field
in the shipped `tables/menu.def`.

**Uses** = occurrences in the shipped `tables/menu.def` (809 records, 72 designs). A count
of 0 means the class is only ever built from code.

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
| `'orez'` | 0x40 | `0x0063d894` | clip region (Czech *orezavac*) | 2 |
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
| `'joys'` | 0x40 | `0x0063d934` | 2-axis pad | 0 |
| `'rctv'` | 0x38 | `0x0063d754` | **vertical** two-colour gradient rect | 0 |
| `'rctg'` | 0x38 | `0x0063d7a4` | **horizontal** two-colour gradient rect | 0 |
| `'pbar'` | 0x38 | `0x0063d8bc` | progress bar | 0 |
| `'load'` | 0x174 | `0x0063dce4` | savegame slot panel | 0 |
| `'edip'` | 0x60 | `0x0063dd84` | IPv4 address entry | 0 |
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

## 5. The drawing primitives

A contiguous block of nine free functions the widgets share, emitted in source order:

| function | primitive | notes |
|---|---|---|
| `gui_draw_line` | type 1, count 1 | 2 verts, `(x, y)` to `(x + w, y + h)` |
| `gui_draw_rect_outline` | type 2, 4 segments | 5 verts, closed strip |
| `gui_draw_rect_filled` | type 5, count 2 | 4 verts. **`(ECX = colour, EDX = texture, x, y, w, h)`**; the texture (0 = none) goes to `g_engine` slot `0xc` before the draw |
| `gui_draw_rect_filled_partial` | type 5, count 2 | right edge at `x + w * frac` |
| `gui_draw_rect_gradient_h` | type 5, count 2 | two colours; colour1 on the **right** verts |
| `gui_draw_rect_gradient_v` | type 5, count 2 | two colours; colour1 on the **bottom** verts |
| `gui_draw_rect_double_bevel` | type 1, count 4 — **twice** | shadow `0xaf000000`, highlight `0x3fffffff`; bit 0 = outer polarity, bit 1 = inner ¹ |
| `gui_draw_triangle_dir` | type 3, count 1 | direction 0..3; 0 = apex up, 1 = apex down |
| `gui_set_clip_rect` | - | clamp to the mode, then install as the scissor |

¹ Both polarity branches are live: `gui_widget_net_player_draw` has a three-way branch over the
polarity, which travels in ECX (§6f).

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
| `g_gui_sponka_texture` | `sponka.tga` (*sponka* = paper clip) | `gui_widget_window_titled_draw`, `gui_widget_frame_draw` |
| `g_gui_kulicky_texture` | `kulicky.tga` (*kuličky* = beads) | `gui_widget_progress_bar_track_draw` |

Every read is the EDX operand of a `gui_draw_rect_filled` call (`gui_draw_rect_filled_partial`
for the beads), so the decomp shows none of them;
they were located by a scan of `.text` for the four addresses.

## 6. PC vs PS2

The widget layer exists on both builds as `GM_Component`, but the PS2 splits it across three
source files (`g_m_comps.cpp`, `g_m_comps2.cpp`, `g_m_compsm.cpp`) where the PC has one.

### 6a. Why the PS2 has ~2.9x more functions

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

The slot-9 getter / slot-8 setter pairs of §3b are spelled `SetText`/`GetText`,
`SetDword`/`GetDword`, `SetPointer`/`GetPointer` on the PS2.

### 6b. The three real behavioural differences

1. **The PS2 widget is a composite; the PC's is a leaf.** `__ct__12GM_ComponentFP15GM_DesignStructP12GM_Component` shows a **child vector at PS2 `+0x2c`/`+0x30`/`+0x34`** that the widget self-inserts into on construction, driving a recursive layer — `BaseDraw`, `BaseAfterMove`, `InsertComponent`, `InsertComponentAfter`, `AfterMove`, `CanBeParent` and per-class `GetClientOrigin`. The PC has **no child vector**: one flat vector on the screen, and `+0x34` used only to offset by the parent's client origin. That whole PS2 layer has no PC counterpart.
2. **The scrollbar is a different object.** PS2 `GM_WindowScroll::Draw` draws its own bar — knob at `(cur/range) * (h - 3*barw)`, track outline and two arrow triangles — then sets the viewport. The PC's `'swnd'` draw is *only* the viewport call: the bar is composed from separate `'sbig'` and two `'trib'` widgets the designer places, visible in `menu.def` design `0x0c`.
3. **The pointer.** The PS2 has pointer plumbing at *screen* level (`GM_Menu::Tick(x, y, …)`, `OnMouse`) but `GM_Component`'s complete 36-method roster has **no mouse entry point**. Slot 6 and every handler in it is a PC addition.

### 6c. Three tag inversions — a FourCC is per-build vocabulary

**The two builds give the same two tags opposite meanings.** PC `'wint'` is the twin of PS2
`GM_Window` — proven by the shared `(y + 40*scale) - 1.0` body split, the `w`/`h - 40`
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
under renamed tags (`trib`/`tria`, `ednu`/`edin`, `swnd`/`wins`, `scrl`/`slid`, `scls`/`sldi`,
`scln`/`slin`, `memi`/`itmm`, `list`/`stxb`), so a tag join alone also produces false absences.

### 6d. Folds, in both directions

| direction | what |
|---|---|
| 2 → 1 | `gui_widget_clip_draw` absorbs `GM_Orezavac::Draw` **and** `GM_UpUp::Draw` ¹ behind a `w < 0` sentinel |
| 3 → 1 | `gui_menu_execute_modal` absorbs `ExecuteMenu` + `TickCamera` + `SetCameraTargetID` |
| 2 → 1 | `gui_widget_is_point_inside` absorbs `IsOverClientArea` and `GetClientOrigin` |
| 1 → 2 | PS2 `Key__15GM_WindowScrollFUiUi` becomes `gui_widget_window_scroll_page_up` / `_page_down`; PC `'swnd'` slot 5 is the framework stub, so the screens drive scroll keys |
| 1 → 2 | the tail of PS2 `Tick__15GM_WindowScrollFUi` becomes `gui_widget_window_scroll_to_show` |
| 1 → 2 | every GCC dtor-with-delete-flag becomes an MSVC dtor + scalar-deleting-dtor pair |

¹ An entire PS2 class whose only body is "reset the clip".

Each of the first three is marked with a `// PS2-INLINED HERE:` comment at the fold site in
the annotated decomp.

### 6e. Classes with no counterpart

**PS2-only:** the console families (`GM_PSX*` memory card, `GM_PS2Help`/`Help2`,
`GM_Karusel`, `GM_ColorSel`, `GM_MultiButton`, `GM_TextInput`, `GM_WindowM`,
`GM_StaticTextBig2Color`/`Fit`/`Mem2`, `GM_ButtonClean`/`Shrink`/`Select`/`SelectBool`,
`GM_ComboWindow`, `GM_ViewScroll`, `GM_PSXTextEdit`, `GM_RectangleB`, `GM_ListBox`,
`GM_WindowTutorial`) plus two that explain themselves: **`GM_Nahrada`** (tag `'xxxx'`, Czech
*náhrada* = substitute) draws a green outline and a green label — the placeholder for an unknown
design tag, where the PC switch simply returns 0; and **`GM_Mapa`** (`'mapa'`) forwards to
`DrawMap__17G_IndicatorsClass`, the PS2 pause map. `GM_StaticTextBig` is not PS2-only — it is
the twin of PC `'list'` (§6c).

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
only widget-relevant free functions there are the eight drawing primitives of §5 plus
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

### 6f. The double bevel and the scroll window

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
`gui_widget_window_scroll_tick` twin — the 1 → 2 split of §6d.

**The scroll-window field roles are swapped across the builds:**

| | PS2 `GM_WindowScroll` | PC `'swnd'` |
|---|---|---|
| range | `+0x3c` (`SetRangeFloat`) | **`+0x28`** |
| target | `+0x40` | **`+0x44`** |
| current | `+0x44` | **`+0x40`** |
| key step | `+0x48` | compiled-in `10.0` |

`+0x28` is proven independently by `gui_load_game_ctor`, which writes it from the last row's
geometry exactly where the PS2 calls the widget's `SetRangeFloat`.

## 7. Open items

* `gui_widget_save_slot_set_text` evaluates textdb `0xae`, `0xaf` and `0xb0` and **discards all
  three results** before tail-jumping to the thumbnail interface. Verified by disassembly, so it
  is not a decompiler artifact. Either the caption build was compiled out or it lives behind the
  jump — unexplained.
* `gui_widget_select_item_mouse` hover-activates when the widget's textdb id is exactly `0x7d3`.
  Which screen relies on it is not observed in game.
* `'joys'` (`gui_widget_axis_range_draw`) has **0 uses in `menu.def`**, so it is built from code
  only and nothing traced constructs it. Its two handles behave exactly like a dead-zone and a
  saturation bound, but the gameplay noun is unproven — hence the neutral name.
* `DAT_006f9528` is the global `g_engine`, but **23 callsites in the decomp still carry the
  `DAT_` spelling**, one of them in this TU (`gui_widget_edit_text_key_max10`, whose sibling
  `gui_widget_edit_text_key_max31` uses `g_engine`).
