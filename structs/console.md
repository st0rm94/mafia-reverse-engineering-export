# `C_console` — the on-screen text console

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-11-16 16:16 +0100

A `0x3c`-byte scrolling text buffer with five methods, whose lines age faster the fuller it is, plus a global registry of live instances, a shared bitmap font and a
1000-vertex quad batch.

**No console line is ever written in the retail build.** The two global instances and the network session's registered one (§5a) are
drawn every frame, but they stay empty: an image-wide `mov ecx, <console>` scan finds only `console_clear`, `console_tick`,
`console_register` and `console_unregister`, no code touches either global's line vector, and the PC has no `OutText` / `Sprintf` (§7).
No input is bound to a console.

Not to be confused with the HUD's own message console (`hud_console_push_message`), a different class in a different translation unit,
nor with `G_ConsoleControls`, where "Console" means the games console (`structs/input.md`).

| | |
|---|---|
| PC prefix | `console_` (with the extensions `console_registry_`, `console_unused_vector_`, `console_log_`, `console_overlay_`) |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x3c` |
| PC constructor | `console_ctor` @ `0x005d3ac0` |
| PC translation unit | `[0x005d3a60, 0x005d4690)`: 19 starts, all named |
| PS2 class | `C_console` (tag `9C_console`), 8 methods, plus 5 free functions |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The Ghidra type behind the decomp spells some of these shorter: `alloc`, `begin`, `end`, `cap`, `def_r`, `def_g`, `def_b`, and
`max_lines` for `nominal_lines`.

```c
typedef struct C_console {                    // 0x3c bytes
    /* +0x00 */ uint8_t  lines_alloc;         // [H] allocator byte of the line vector
    /* +0x04 */ S_console_line *lines_begin;  // [H] the oldest line
    /* +0x08 */ S_console_line *lines_end;    // [H] one past the newest line
    /* +0x0c */ S_console_line *lines_cap;    // [H] end of the vector's storage
    /* +0x10 */ int32_t  default_red;         // [H] the default colour's red
    /* +0x14 */ int32_t  default_green;       // [H] the default colour's green
    /* +0x18 */ int32_t  default_blue;        // [H] the default colour's blue
    /* +0x1c */ float    x;                   // [H] left edge, fraction of screen width
    /* +0x20 */ float    y;                   // [H] first baseline, fraction of screen height
    /* +0x24 */ float    glyph_size;          // [H] glyph height, fraction of screen height
    /* +0x28 */ float    line_step;           // [H] baseline step, fraction of screen height
    /* +0x2c */ int32_t  nominal_lines;       // [H] the line count at which lines age 11x as fast
    /* +0x30 */ int32_t  line_lifetime_ms;    // [H] ms each line is kept
    /* +0x34 */ int32_t  expiry_accum;        // [H] ms accumulated by console_tick
    /* +0x38 */ uint8_t  anchor_bottom;       // [H] the block grows upward from y
    /* +0x39 */ uint8_t  reverse_chars;       // [H] lay each line out right to left
    /* +0x3a */ uint8_t  reverse_lines;       // [H] walk the lines newest first
} C_console;
```

### 1a. `S_console_line` — one line, `0x1c` bytes

```c
typedef struct S_console_line {               // 0x1c bytes, the vector's stride
    /* +0x00 */ char    *text;                // [H] heap copy of the line
    /* +0x04 */ int32_t  red;                 // [H] the line's red, -1 = the default
    /* +0x08 */ int32_t  green;               // [H] the line's green, -1 = the default
    /* +0x0c */ int32_t  blue;                // [H] the line's blue, -1 = the default
    /* +0x10 */ int32_t  group;               // [U] line group id; no PC access (class census), its use is in §7
    /* +0x14 */ float    x_override;          // [H] above 0.0, used instead of the console's x
    /* +0x18 */ float    y_override;          // [H] the matching y for that branch
} S_console_line;
```

The colour is packed for the renderer as `((red - 0x100) * 0x100 + green) * 0x100 + blue`.

### 1b. `S_con_register` — one registry record, `0xc` bytes

```c
typedef struct S_con_register {               // 0xc bytes, the registry's stride
    /* +0x00 */ C_console *console;           // [H] the registered console
    /* +0x04 */ uint32_t flags;               // [H] render and clear policy (§2a)
    /* +0x08 */ uint32_t refcount;            // [H] registrations still held
} S_con_register;
```

## 2. Field evidence

Measured by a register-tag census of the class's functions, an absolute-reference census of the two global instances, and reading the
bodies of the constructor, `console_clear`, `console_tick`, `console_render`, `console_register`, `console_unregister` and
`console_render_all`. Lines and registry records are reached through walking pointers, so their cells come from the bodies. `game_init`
configures the two globals (§5a); `net_session_ctor` constructs two more instances and writes none of these fields itself.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | lines_alloc | H | `console_ctor` | - | the allocator byte |
| +0x4 | lines_begin | H | `console_ctor`, `console_dtor` | `console_clear`, `console_dtor`, `console_render`, `console_tick` | - |
| +0x8 | lines_end | H | `console_clear`, `console_ctor`, `console_dtor`, `console_tick` | `console_clear`, `console_render`, `console_tick` | - |
| +0xc | lines_cap | H | `console_ctor`, `console_dtor` | - | no reader: no PC code inserts a line (§7) |
| +0x10 | default_red | H | `console_ctor`, `game_init` | `console_render` | ctor `0xff`; a line's -1 takes it |
| +0x14 | default_green | H | `console_ctor`, `game_init` | `console_render` | ctor `0xff` |
| +0x18 | default_blue | H | `console_ctor`, `game_init` | `console_render` | ctor `0xff` |
| +0x1c | x | H | `console_ctor`, `game_init` | `console_render` | fraction of screen width; ctor 0.1 |
| +0x20 | y | H | `console_ctor`, `game_init` | `console_render` | fraction of screen height; ctor 0.1 |
| +0x24 | glyph_size | H | `console_ctor`, `game_init` | `console_render` | fraction of screen height; ctor 0.1 |
| +0x28 | line_step | H | `console_ctor`, `game_init` | `console_render` | fraction of screen height; ctor 0.11 |
| +0x2c | nominal_lines | H | `console_ctor`, `game_init` | `console_tick` | ctor 100; only `console_tick`'s ageing rate reads it, so it is no limit on the vector |
| +0x30 | line_lifetime_ms | H | `console_ctor`, `game_init` | `console_tick` | ctor 1000 |
| +0x34 | expiry_accum | H | `console_clear`, `console_tick` | `console_tick` | `console_ctor` does not initialise it; `console_clear` zeroes it |
| +0x38 | anchor_bottom | H | `console_ctor`, `game_init` | `console_render` | `y -= line_step` before the first line |
| +0x39 | reverse_chars | H | `console_ctor`, `game_init` | `console_render` | lays each line out right to left |
| +0x3a | reverse_lines | H | `console_ctor`, `game_init` | `console_render` | walks the line vector newest first |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | text | H | - | `console_clear`, `console_render`, `console_tick` | no PC code writes a line (§7); freed by both, `strlen`'d by the render |
| +0x4 | red | H | - | `console_render` | -1 takes the console's default |
| +0x8 | green | H | - | `console_render` | the same |
| +0xc | blue | H | - | `console_render` | the same |
| +0x10 | group | U | - | - | no PC function reads or writes it (class census); §7 has its only use |
| +0x14 | x_override | H | - | `console_render` | above 0.0 it replaces the console's `x` |
| +0x18 | y_override | H | - | `console_render` | multiplied by the screen height in the `x_override` branch; the decomp hides it as a `__ftol` argument, read at `fmul [edx + ebp + 0x18]` in the disassembly |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | console | H | `console_register` | `console_register`, `console_render_all`, `console_unregister` | the key |
| +0x4 | flags | H | `console_register` | `console_render_all` | §2a |
| +0x8 | refcount | H | `console_register`, `console_unregister` | `console_unregister` | starts at 1, bumped by a repeat registration |

### 2a. `S_con_register.flags`

| Value | Meaning | Evidence |
|---|---|---|
| bit 0 | clear the console after rendering | `console_render_all`: `console_clear` under `flags & 1` |
| bit 1 | render even when `g_console_enabled` is 0 | `console_render_all`: `g_console_enabled != 0 \|\| flags & 2` |

## 3. Method family

10 of 13 PS2 symbols have a PC twin, all CONFIRMED by body diff; 3 PS2 methods have no PC function (§7). `console_registry_`,
`console_unused_vector_`, `console_log_` and `console_overlay_` are proper extensions of the `console_` prefix, the same shape as
`particle_effect_pool_` extending `particle_`, so the sixteen static-init rows are not `C_console` methods.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005d3a60 | `console_registry_static_init` | `g_console_registry` | - | - | STRUCTURAL | `.CRT$XCU` entry |
| 0x005d3a70 | `console_registry_static_ctor` | `g_console_registry` | - | - | STRUCTURAL | zeroes the registry vector |
| 0x005d3a90 | `console_registry_register_atexit_dtor` | `g_console_registry` | - | - | STRUCTURAL | registers the static destructor |
| 0x005d3aa0 | `console_registry_static_dtor` | `g_console_registry` | - | - | STRUCTURAL | frees the registry vector |
| 0x005d3ac0 | `console_ctor` | instance | `__ct__9C_consoleFv` | 0x00274450 | CONFIRMED | the defaults of §1; a zero-sized `operator_new` for the line vector |
| 0x005d3b20 | `console_dtor` | instance | `__dt__9C_consoleFv` | 0x00274540 | CONFIRMED | `console_clear`, then frees the line vector |
| 0x005d4640 | `console_unused_vector_static_init` | `g_console_unused_vector` | - | - | STRUCTURAL | `.CRT$XCU` entry (§6) |
| 0x005d4650 | `console_unused_vector_static_ctor` | `g_console_unused_vector` | - | - | STRUCTURAL | zeroes the vector |
| 0x005d4670 | `console_unused_vector_register_atexit_dtor` | `g_console_unused_vector` | - | - | STRUCTURAL | registers the static destructor |
| 0x005d4680 | `console_unused_vector_static_dtor` | `g_console_unused_vector` | - | - | STRUCTURAL | frees the vector |
| 0x005dc190 | `console_log_static_init` | `g_console_log` | - | - | STRUCTURAL | `.CRT$XCU` entry, in the `C_game` unit (§5) |
| 0x005dc1a0 | `console_log_static_ctor` | `g_console_log` | - | - | STRUCTURAL | `console_ctor` on the global |
| 0x005dc1b0 | `console_log_register_atexit_dtor` | `g_console_log` | - | - | STRUCTURAL | registers the static destructor |
| 0x005dc1c0 | `console_log_static_dtor` | `g_console_log` | - | - | STRUCTURAL | `console_dtor` on the global |
| 0x005dc1d0 | `console_overlay_static_init` | `g_console_overlay` | - | - | STRUCTURAL | `.CRT$XCU` entry, in the `C_game` unit (§5) |
| 0x005dc1e0 | `console_overlay_static_ctor` | `g_console_overlay` | - | - | STRUCTURAL | `console_ctor` on the global |
| 0x005dc1f0 | `console_overlay_register_atexit_dtor` | `g_console_overlay` | - | - | STRUCTURAL | registers the static destructor |
| 0x005dc200 | `console_overlay_static_dtor` | `g_console_overlay` | - | - | STRUCTURAL | `console_dtor` on the global |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d3b70 | `console_clear` | `Clear__9C_consoleFv` | 0x002747c0 | CONFIRMED | frees every line's text, `end = begin`, zeroes `expiry_accum` |
| 0x005d3bf0 | `console_tick` | `Tick__9C_consoleFUi` | 0x00274830 | CONFIRMED | while a line exists, `expiry_accum += elapsed * (1 + 10 * count / nominal_lines)`; drops the oldest line per `line_lifetime_ms` accumulated |
| 0x005d3ce0 | `console_render` | `Render__9C_consoleFv` | 0x00274bf0 | CONFIRMED | a quad batch, 6 vertices per glyph, flushed at 1000 vertices; `console_render_all` is its one caller |
| 0x005d4230 | `console_register` | `ConsoleReg__FP9C_consoleUi` | 0x002751e0 | CONFIRMED | a ref-counted registry insert; `__fastcall` (console, flags) |
| 0x005d4440 | `console_unregister` | `ConsoleUnReg__FP9C_console` | 0x002752a0 | CONFIRMED | a ref-counted release; at zero it clears the console and erases the record |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d44f0 | `console_render_all` | `ConsoleRender__Fv` | 0x00275350 | CONFIRMED | the per-frame pass, last record first: renders while `g_console_enabled` or flag bit 1, clears on flag bit 0; nothing without a font texture |
| 0x005d45c0 | `console_load_font` | - | - | - | the body behind `console_font_init`: loads `editfont.bmp` into the pointer it is given and reads `fnt_matrix.bin` into `g_console_font_matrix` |
| 0x005d4620 | `console_font_init` | `ConsolesInit__Fv` | 0x00275570 | CONFIRMED | a 10-byte forwarder |
| 0x005d4630 | `console_font_release` | `ConsolesDone__Fv` | 0x00275580 | CONFIRMED | releases the font texture |

## 4. The globals, and how they tile

BSS is emitted in link order, and this unit's globals form a **gapless run**, which is what places the otherwise unattributable object at
`0x006efde0` inside it:

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_console_unused_vector` (+ `_begin` / `_end` / `_cap`) | `0x006efde0` | `0x10` bytes, §6 | its static ctor and dtor | its static dtor |
| `g_console_vert_count` | `0x006efdf0` | 4 bytes + 4 pad | `console_render` | `console_render` |
| `g_console_vert_buffer` | `0x006efdf8` | `0x7d00` bytes: 1000 x 32-byte D3D `TLVERTEX` | `console_render` | `console_render` |
| `g_console_font_matrix` | `0x006f7af8` | `0x1400` bytes: 256 x `0x14` | `console_load_font` | `console_render` |
| `g_console_registry` (+ begin / end / cap) | `0x006f8ef8` | `0x10` bytes | its static ctor and dtor, `console_register`, `console_unregister` | `console_register`, `console_unregister`, `console_render_all` |
| `g_console_font_texture` | `0x006f8f08` | 4 bytes | `console_load_font` | `console_render_all`, `console_font_release` |
| `g_console_overlay` + `g_console_log` (`0x006f919c`) | `0x006f9160` | `0x78` bytes, two separate objects (§5) | their static ctors, `game_init` | `game_init`, `game_done`, `game_tick` |
| `g_console_enabled` | `0x00650410` | 4 bytes, shipped as 1 | - | `console_render_all` |

Two of those sizes fall out of the code rather than out of the addresses, and they agree with the addresses exactly:

* the vertex is `{x, y, z, rhw, diffuse, specular, tu, tv}` = 32 bytes, and `console_render` flushes at 1000 vertices; `1000 x 32 = 0x7d00`,
  which is precisely `0x006f7af8 - 0x006efdf8`;
* the glyph record is `{float u0, v0, u1, v1, advance}` = `0x14` bytes and the loader reads exactly `0x1400`; `0x1400 = 256 x 0x14`, which is
  precisely `0x006f8ef8 - 0x006f7af8`. 256 entries, one per value of the `char` the render indexes it with.

`DAT_006f7afc` / `_006f7b00` / `_006f7b04` / `_006f7b08` and `DAT_006efdfc`..`DAT_006efe14` are Ghidra symbols on the fields of record 0 of
those two arrays, not separate globals, and stay unrenamed.

## 5. Two separate objects, not an array

`0x006f9160` and `0x006f919c` are `0x3c` apart, one class size, but they are **two independent file-scope objects** that the linker lays out
adjacently. Each address has its own `.CRT$XCU` dynamic initializer, `0x005dc190` for `g_console_log` and `0x005dc1d0` for `g_console_overlay`,
listed as two separate slots at `.data:0x00646274` and `0x00646278`, and each registers its own `atexit` destructor. MSVC emits a single
initializer for an array. The eight functions (§3a) are file-scope statics of `g_game.cpp`, which is what puts them at `0x005dc190`, not in the
console unit. Both destructors are reached only as the `atexit` argument; nothing calls them.

### 5a. The two instances `game_init` builds

Both are configured field by field and then registered; the registry flags are `__fastcall` arguments, read from the disassembly of the two
callsites.

| | `g_console_overlay` @ `0x006f9160` | `g_console_log` @ `0x006f919c` |
|---|---|---|
| colour | `0x80, 0xff, 0x80`, pale green | `0xff, 0xff, 0xff`, white |
| x / y | 0.01 / 0.45 (mid-screen) | 0.01 / 0.89 (bottom) |
| glyph / step | 0.038 / 0.028 | 0.035 / 0.028 |
| nominal lines | 20 | 20 |
| lifetime | 1000 ms (ctor default, not overwritten) | 10 000 ms |
| anchor_bottom | 0 | 1, grows upward |
| registry flags | 3 = always render + clear every frame | 2 = always render, never auto-clear |
| reads as | a per-frame transient overlay | a scrolling message log |

`g_net_session` embeds two more at `+0x20` and `+0x5c`. Only the one at `+0x5c` is registered (flags 3); the one at `+0x20` is constructed and
destroyed but never registered: `console_register` has exactly one callsite in `net_session_ctor` and it is `mov ecx,edi`, the same register
`console_unregister`'s `lea ecx,[esi+0x5c]` resolves to in `net_session_dtor`.

## 6. `g_console_unused_vector` — constructed, destroyed, never used

The file-scope object at `0x006efde0` is an MSVC `std::vector` (allocator byte plus begin / end / cap) whose four lifecycle functions are named and
whose element type is undetermined.

* A **byte scan of the whole image** for all four addresses returns eight references, four in its constructor and four in its destructor, and
  nothing else. No vtable, pointer table or other function holds any of the addresses.
* It belongs to this unit on two independent arguments: the gapless BSS run in §4, and its `.CRT$XCU` entry being the next one after the console
  unit's own.
* `T` is a POD (the destructor frees the block with no per-element loop), but no insert exists anywhere, so no stride is ever materialised.

The name states the finding rather than a guess. The allocator byte is `g_console_unused_vector` and the three pointers are
`g_console_unused_vector_begin` / `_end` / `_cap`, the same shape as `g_public_physics_list_begin`.

## 7. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002745b0 | `Sprintf__9C_consoleFiPCce` | not in the PC image: dropped as unreferenced, not inlined (below) |
| 0x00274660 | `OutText__9C_consoleFPciiii` | the same |
| 0x002749b0 | `RenderIndicators__9C_consoleFv` | the PC dispatcher has no flag-4 branch, so no PC code path could reach one (below) |

**Layout.** Field for field against the PS2 after the one-dword shift MSVC's three-pointer vector adds over the PS2's count + pointer pair: PS2
`+0x04` count / `+0x08` data, then `+0x0c`..`+0x14` colour, `+0x18`..`+0x20` the 0.1f triple, `+0x24` the 0.11f, `+0x28` = 100, `+0x2c` = 1000,
`+0x30` accumulator, `+0x34` / `+0x35` / `+0x36` the three bools.

**Type names.** `S_console_line` is the template argument of `insert__Q23std68__vector_imp<>FP14S_console_lineUiRC14S_console_line`,
`S_con_register` that of `insert__Q23std68__vector_imp<>FP14S_con_registerUiRC14S_con_register`. On the PS2, `S_con_register.flags` bit 2 selects
`RenderIndicators` instead of `Render`, and `OutText` replaces the last line instead of appending when the incoming `S_console_line.group`
matches it.

**Colour packing.** PS2 packs `b + g*0x100 + r*0x10000 - 0x1000000`, algebraically the same expression as the PC's, and part of the shared evidence
for `console_render`.

**Globals.** `g_console_vert_count` = PS2 `__vert_count`, `g_console_font_matrix` = `fnt_matrix`, `g_console_registry` = PS2 `0x004e4e80`,
`g_console_font_texture` = `fonty`, `g_console_enabled` = `_is_on_`.

**Free functions.** Five of the thirteen PS2 console symbols are free functions with an empty class column (`ConsoleReg`, `ConsoleUnReg`,
`ConsoleRender`, `ConsolesInit`, `ConsolesDone`), so a search by class misses them; a search of the raw symbol names for `Console` finds all five.

**The three methods with no PC function.**

* `RenderIndicators__9C_consoleFv` (568 B): PS2 `ConsoleRender__Fv` tests the registry record's flags a third time and dispatches `flags & 4` to
  it instead of `Render`; that method draws the same lines through the HUD's `OutText__17G_IndicatorsClass`. An inline banner sits at that exact
  statement in `console_render_all`.
* `Sprintf__9C_consoleFiPCce` (168 B) and `OutText__9C_consoleFPciiii` (328 B) are not in the PC image. Three measurements close it: the unit's 19
  defs tile `[0x005d3a60, 0x005d4690)` with no gap longer than 8 bytes; no code touches the line vector of `g_console_overlay` or `g_console_log`; and
  the image's only `0x1c`-stride vector insert, `std_vector1c_insert_n`, has three callers, `car_load_particle_effects`, `vehicle_init` and
  `game_add_frame_for_shadow`, none of them console code. Without `OutText`, the replace-last-line rule on `S_console_line.group` has no PC code.

**Two PC functions have no PS2 counterpart to look for.** `console_load_font` is the out-of-line body of the `ConsolesInit` forwarder: both builds
carry the same forwarder-plus-body pair, and only the forwarder is a symbol on PS2. The `g_console_unused_vector` quartet is in §6; PS2 `console.cpp`
instantiates exactly two vector types (`S_console_line`, `S_con_register`) and no third, so there is no PS2 symbol to name its element type from.

**Where the PS2 writes to a console and the PC does not.** `Tick__6C_gameFUi` calls `Tick__9C_consoleFUi` and then, behind `C_game+0x1338 & 2`, emits
five `OutText` and two `Sprintf` calls printing the mission name and the player's world position. `MessageBoxGraph__FPcPcPcPcPc` emits seven more. The PC
`game_tick` calls `console_tick` at the same position and then stops; an inline banner records that.

**`MessageBoxGraph__FPcPcPcPcPc` is PS2-only.** It is a modal on-screen error box: a stack `C_console`, up to five caller lines, then `"Press any key to
continue..."`, rendered until a key is pressed. Its two PS2 callers are `C_cache_base_block::Open` (`"Chybi model"`, "model missing") and
`GM_Menu::CreateCompsByDef` (`"Nenasel menu:"`, "menu not found"). The PC `cache_base_block_open` reports the same missing model through Win32
`MessageBoxA` with the caption `"Chybi model"`, `gui_menu_build_components_from_design` has no not-found branch, and neither `"Nenasel menu:"` nor `"Press
any key to continue..."` occurs in `Game.exe`.

## 8. Related

`structs/input.md` (the previous unit), `structs/hud.md` (the other console, `hud_console_push_message`), `structs/net_session.md` (two more instances),
`structs/loadsave.md`, `structs/game.md` (the two global instances and `console_tick`'s caller).

## 9. Open items

| Item | State |
|---|---|
| in game | nothing in the retail build writes a console line, so the consoles show nothing without added code that writes one; §5a fixes what they would show: a pale-green block mid-screen that clears every frame, and a white log at the bottom that grows upward and holds each line for ten seconds |
