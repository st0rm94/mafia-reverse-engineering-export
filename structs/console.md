# `C_console` — the on-screen text console

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-09-27 17:33 +0200

| | |
|---|---|
| PC prefix | **`console_`** (with the extensions `console_registry_` and `console_unused_vector_`) |
| PC translation unit | `[0x005d3a60, 0x005d4640)` — 15 functions, all named |
| Class size | `0x3c` bytes |
| PS2 symbols | 8, all accounted for; 5 body-diffed CONFIRMED twins (§4, §5) |

A 0x3c-byte scrolling text buffer with five methods, plus a global registry of live
instances, a shared bitmap font and a 1000-vertex quad batch.

> 🔴 **The console is not reachable in the retail build.** No input is bound to it; per the
> user it is exposed by the separate `MafiaCon.exe` launcher, which starts the game in place of
> `Game.exe`. That is consistent with the measurement: an image-wide `mov ecx, <console>` scan
> finds only `console_clear`, `console_tick`, `console_register` and `console_unregister` —
> **nothing in `Game.exe` ever writes a console LINE.**

> ⚠️ Not to be confused with the HUD's own message console (`hud_console_push_message`), which is
> a different class in a different translation unit, nor with `G_ConsoleControls`, where
> "Console" means the games console — see [input.md](input.md).

## 1. Layout

```c
typedef struct C_console {              /* 0x3c bytes */
/* +0x00 */ unsigned char   alloc;      // [H] std::vector allocator byte
/* +0x04 */ S_console_line *begin;      // [H] the line vector
/* +0x08 */ S_console_line *end;
/* +0x0c */ S_console_line *cap;
/* +0x10 */ int   def_r;                // [H] default colour, ctor 0xff
/* +0x14 */ int   def_g;                // [H] ctor 0xff
/* +0x18 */ int   def_b;                // [H] ctor 0xff
/* +0x1c */ float x;                    // [H] left edge, fraction of screen width; ctor 0.1f
/* +0x20 */ float y;                    // [H] first baseline, fraction of height; ctor 0.1f
/* +0x24 */ float glyph_size;           // [H] fraction of screen height;          ctor 0.1f
/* +0x28 */ float line_step;            // [H] fraction of height;                 ctor 0.11f
/* +0x2c */ int   max_lines;            // [H] ctor 100
/* +0x30 */ int   line_lifetime_ms;     // [H] ctor 1000
/* +0x34 */ int   expiry_accum;         // [H] ms accumulated by console_tick
/* +0x38 */ char  anchor_bottom;        // [H] y -= line_step before the first line
/* +0x39 */ char  reverse_chars;        // [H] lay each line out right-to-left
/* +0x3a */ char  reverse_lines;        // [H] walk the line vector newest-first
} C_console;
```

### 1a. `S_console_line` — 0x1c bytes

The type name comes from the PS2 vector template instantiation (§5).

```c
typedef struct S_console_line {   /* 0x1c bytes */
/* +0x00 */ char *text;           // strdup'd; freed by console_clear / console_tick
/* +0x04 */ int   r;              // -1 => use the console's default
/* +0x08 */ int   g;              // -1 => default
/* +0x0c */ int   b;              // -1 => default
/* +0x10 */ int   group;          // line group id (§7)
/* +0x14 */ float x_override;     // > 0.0 => use instead of the console's x
/* +0x18 */ float y_override;
} S_console_line;
```

The colour is packed for the renderer as `((r - 0x100) * 0x100 + g) * 0x100 + b`.

### 1b. `S_con_register` — 0xc bytes, the global registry record

Also named from a PS2 vector template instantiation (§5).

```c
typedef struct S_con_register {   /* 0xc bytes */
/* +0x00 */ C_console *console;
/* +0x04 */ unsigned int flags;   // bit 0 = clear after render
                                  // bit 1 = render even when g_console_enabled == 0
/* +0x08 */ unsigned int refcount;
} S_con_register;
```

## 2. The globals, and how they tile

BSS is emitted in link order, and this TU's globals form a **gapless run** — which is what places
the otherwise unattributable object at `0x006efde0` inside it:

| address | size | symbol |
|---|---|---|
| `0x006efde0` | `0x10` | `g_console_unused_vector` (+ `_begin` / `_end` / `_cap`) — see §6 |
| `0x006efdf0` | `4` (+4 pad) | `g_console_vert_count` |
| `0x006efdf8` | `0x7d00` | `g_console_vert_buffer` — **1000 × 32-byte D3D `TLVERTEX`** |
| `0x006f7af8` | `0x1400` | `g_console_font_matrix` — **256 × 0x14** |
| `0x006f8ef8` | `0x10` | `g_console_registry` (+ begin/end/cap) |
| `0x006f8f08` | `4` | `g_console_font_texture` |
| `0x006f9160` | `0x78` | `g_console_overlay` + `g_console_log` (`0x006f919c`) — two separate objects (§3) |
| `0x00650410` | `4` | `g_console_enabled` — **shipped as 1**, no writer |

Two of those sizes fall out of the code rather than out of the addresses, and they agree
with the addresses exactly:

* the vertex is `{x, y, z, rhw, diffuse, specular, tu, tv}` = **32 bytes**, and
  `console_render` flushes at **1000** vertices; `1000 × 32 = 0x7d00`, which is precisely
  `0x006f7af8 − 0x006efdf8`;
* the glyph record is `{float u0, v0, u1, v1, advance}` = **0x14 bytes** and the loader
  reads exactly **0x1400**; `0x1400 = 256 × 0x14`, which is precisely
  `0x006f8ef8 − 0x006f7af8`.

⚠️ `DAT_006f7afc`/`_006f7b00`/`_006f7b04`/`_006f7b08` and `DAT_006efdfc`..`DAT_006efe14`
are Ghidra symbols on the **fields of record 0** of those two arrays, not separate globals,
and stay unrenamed.

## 3. Two separate objects, not an array

`0x006f9160` and `0x006f919c` are `0x3c` apart — one class size — but they are **two independent
file-scope objects** that the linker lays out adjacently. Each address has its **own `.CRT$XCU`
dynamic initializer** — `0x005dc190` for `g_console_log` and `0x005dc1d0` for `g_console_overlay`,
listed as two separate slots at `.data:0x00646274` and `0x00646278` — and each registers its
**own `atexit` destructor**. MSVC emits a *single* initializer for an array.

The eight functions are file-scope statics of `g_game.cpp`, which is what puts them at
`0x005dc190`, not in the console TU:

| object | `_static_init` | `_static_ctor` | `_register_atexit_dtor` | `_static_dtor` |
|---|---|---|---|---|
| `g_console_overlay` @ `0x006f9160` | `0x005dc1d0` | `0x005dc1e0` | `0x005dc1f0` | `0x005dc200` |
| `g_console_log` @ `0x006f919c` | `0x005dc190` | `0x005dc1a0` | `0x005dc1b0` | `0x005dc1c0` |

Both destructors are reached ONLY as the `atexit` argument (`&console_log_static_dtor` /
`&console_overlay_static_dtor`); nothing calls them.

## 3a. The two instances `game_init` builds

Both are configured field-by-field and then registered; the registry flags are `__fastcall`
arguments, read from the disassembly of the two callsites.

| | `g_console_overlay` @ `0x006f9160` | `g_console_log` @ `0x006f919c` |
|---|---|---|
| colour | `0x80, 0xff, 0x80` — pale green | `0xff, 0xff, 0xff` — white |
| x / y | 0.01 / **0.45** (mid-screen) | 0.01 / **0.89** (bottom) |
| glyph / step | 0.038 / 0.028 | 0.035 / 0.028 |
| max lines | 20 | 20 |
| lifetime | **1000 ms** (ctor default, not overwritten) | **10 000 ms** |
| anchor_bottom | 0 | **1** — grows upward |
| registry flags | **3** = always render **+ clear every frame** | **2** = always render, never auto-clear |
| reads as | a per-frame transient overlay | a scrolling message log |

`g_net_session` embeds two more at `+0x20` and `+0x5c`. Only the one at `+0x5c` is
registered (flags **3**); the one at `+0x20` is constructed and destroyed but never
registered: `console_register` has exactly one callsite in `net_session_ctor` and it is
`mov ecx,edi`, the same register `console_unregister`'s `lea ecx,[esi+0x5c]` resolves to in
`net_session_dtor`.

## 4. PC coverage

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d3a60 | `console_registry_static_init` | - | - | STRUCTURAL | `.CRT$XCU` entry |
| 0x005d3a70 | `console_registry_static_ctor` | - | - | STRUCTURAL | zero the registry vector |
| 0x005d3a90 | `console_registry_register_atexit_dtor` | - | - | STRUCTURAL | - |
| 0x005d3aa0 | `console_registry_static_dtor` | - | - | STRUCTURAL | free the registry vector |
| 0x005d3ac0 | `console_ctor` | `__ct__9C_consoleFv` | 0x00274450 | **CONFIRMED** | defaults |
| 0x005d3b20 | `console_dtor` | `__dt__9C_consoleFv` | 0x00274540 | **CONFIRMED** | clear then release |
| 0x005d3b70 | `console_clear` | `Clear__9C_consoleFv` | 0x002747c0 | **CONFIRMED** | free every line's text |
| 0x005d3bf0 | `console_tick` | `Tick__9C_consoleFUi` | 0x00274830 | **CONFIRMED** | expire the oldest lines |
| 0x005d3ce0 | `console_render` | `Render__9C_consoleFv` | 0x00274bf0 | **CONFIRMED** | quad batch, 6 verts/glyph |
| 0x005d4230 | `console_register` | `ConsoleReg__FP9C_consoleUi` | 0x002751e0 | **CONFIRMED** | ref-counted registry insert |
| 0x005d4440 | `console_unregister` | `ConsoleUnReg__FP9C_console` | 0x002752a0 | **CONFIRMED** | ref-counted release |
| 0x005d44f0 | `console_render_all` | `ConsoleRender__Fv` | 0x00275350 | **CONFIRMED** | per-frame pass over the registry |
| 0x005d45c0 | `console_load_font` | *(the body behind the PS2 forwarder)* | - | **CONFIRMED** | `editfont.bmp` + `fnt_matrix.bin` |
| 0x005d4620 | `console_font_init` | `ConsolesInit__Fv` | 0x00275570 | **CONFIRMED** | 10-byte forwarder |
| 0x005d4630 | `console_font_release` | `ConsolesDone__Fv` | 0x00275580 | **CONFIRMED** | release the font texture |
| 0x005d4640 | `console_unused_vector_static_init` | - | - | STRUCTURAL | `.CRT$XCU` entry, §6 |
| 0x005d4650 | `console_unused_vector_static_ctor` | - | - | STRUCTURAL | zero the unused vector |
| 0x005d4670 | `console_unused_vector_register_atexit_dtor` | - | - | STRUCTURAL | - |
| 0x005d4680 | `console_unused_vector_static_dtor` | - | - | STRUCTURAL | free the unused vector |
| - | - | `Sprintf__9C_consoleFiPCce` | 0x002745b0 | **not located** | see §5 |
| - | - | `OutText__9C_consoleFPciiii` | 0x00274660 | **not located** | see §5 |
| - | - | `RenderIndicators__9C_consoleFv` | 0x002749b0 | **not located** | see §5 |

`console_registry_` and `console_unused_vector_` are proper EXTENSIONS of the `console_` class
prefix — the same shape as `particle_effect_pool_` extending `particle_` — so the eight CRT
scaffolding rows are not `C_console` methods.

## 5. PC vs PS2

**Layout.** Field for field against the PS2 after the one-dword shift MSVC's three-pointer vector
adds over the PS2's count+pointer pair: PS2 `+0x04` count / `+0x08` data, then `+0x0c`..`+0x14`
colour, `+0x18`..`+0x20` the 0.1f triple, `+0x24` the 0.11f, `+0x28` = 100, `+0x2c` = 1000,
`+0x30` accumulator, `+0x34`/`+0x35`/`+0x36` the three bools.

**Type names.** `S_console_line` is the template argument of
`insert__Q23std68__vector_imp<>FP14S_console_lineUiRC14S_console_line`, `S_con_register` that of
`insert__Q23std68__vector_imp<>FP14S_con_registerUiRC14S_con_register`. On the PS2,
`S_con_register.flags` bit 2 selects `RenderIndicators` instead of `Render`, and `OutText`
replaces the last line instead of appending when the incoming `S_console_line.group` matches it.

**Colour packing.** PS2 packs `b + g*0x100 + r*0x10000 - 0x1000000` — algebraically the same
expression as the PC's, and part of the shared evidence for `console_render`.

**Globals.** `g_console_vert_count` = PS2 `__vert_count`, `g_console_font_matrix` = `fnt_matrix`,
`g_console_registry` = PS2 `0x004e4e80`, `g_console_font_texture` = `fonty`, `g_console_enabled`
= `_is_on_`.

**Free functions.** Five of the eight PS2 console symbols are free functions with an empty class
column (`ConsoleReg`, `ConsoleUnReg`, `ConsoleRender`, `ConsolesInit`, `ConsolesDone`), so a
search by class misses them; a search of the raw symbol names for `Console` finds all five.

**Three PS2 methods have no located PC function.**

* `RenderIndicators__9C_consoleFv` (568 B) — measured cause: PS2 `ConsoleRender__Fv` tests the
  registry record's flags a third time and dispatches `flags & 4` to `RenderIndicators` instead
  of `Render`; that method draws the same lines through the HUD's
  `OutText__17G_IndicatorsClass`. **The PC dispatcher has no flag-4 branch at all**, so no PC
  code path could reach a PC `RenderIndicators`. An inline banner sits at that exact statement
  in `console_render_all`.
* `Sprintf__9C_consoleFiPCce` (168 B) and `OutText__9C_consoleFPciiii` (328 B) — all 15 defs of
  the PC TU were read and neither is among them, and the image-wide `mov ecx, <console>` scan
  shows nothing writes a console line through an immediate receiver. With the console
  unreachable in retail, `/OPT:REF` stripping is at least as likely as inlining (§7).

**Two PC functions have no PS2 counterpart to look for.** `console_load_font` is the out-of-line
body of the `ConsolesInit` forwarder — both builds carry the same forwarder-plus-body pair, and
only the forwarder is a symbol on PS2. The `g_console_unused_vector` quartet is in §6; PS2
`console.cpp` instantiates exactly two vector types (`S_console_line`, `S_con_register`) and no
third, so there is no PS2 symbol to name its element type from.

**Where the PS2 writes to a console and the PC does not.** `Tick__6C_gameFUi` calls
`Tick__9C_consoleFUi` and then, behind `C_game+0x1338 & 2`, emits five `OutText` and two
`Sprintf` calls printing the mission name and the player's world position.
`MessageBoxGraph__FPcPcPcPcPc` emits seven more. The PC `game_tick` calls `console_tick`
at exactly the same position and then stops; an inline banner records that.

## 6. `g_console_unused_vector` — constructed, destroyed, never used

The file-scope object at `0x006efde0` is an MSVC `std::vector` (allocator byte plus
begin/end/cap) whose four lifecycle functions are named and whose element type is
**undetermined**.

* A **displacement byte scan of the whole `.text`** for all four addresses returns **five
  references image-wide, and every one is inside its own constructor or destructor**. A scan of
  `.rdata`/`.data` for the addresses as pointers finds nothing, so it is not in a vtable or a
  pointer table.
* It belongs to this TU on two independent arguments: the gapless BSS run in §2, and its
  `.CRT$XCU` entry being the next one after the console TU's own.
* `T` is a POD — the destructor frees the block with no per-element loop — but no insert
  exists anywhere, so no stride is ever materialised and the type cannot be recovered (§5).

The name states the finding rather than a guess. The allocator byte is `g_console_unused_vector`
and the three pointers are `g_console_unused_vector_begin` / `_end` / `_cap`, the same shape as
`g_public_physics_list_begin`.

## 7. Open items

| item | state |
|---|---|
| what `MafiaCon.exe` patches to make the console reachable | the repo ships it (`mods/Editors/Editors/Mafia Console/MafiaCon.exe`, plus an x64 update and two archives under `programs/`); a diff against `Game.exe` would show whether `OutText`/`Sprintf` were stripped or inlined, and what input it binds |
| `Sprintf` / `OutText` on PC | alternatively settled by a byte scan of `.text` for the `strlen` + `operator new[]` + insert shape |
| `S_console_line.group` (`+0x10`) | no PC writer exists to confirm the PC keeps the PS2 replace-last-line behaviour |
| `g_console_enabled` | no writer, 1 in the shipped image; whether anything was meant to clear it is unknown |
| `MessageBoxGraph__FPcPcPcPcPc` | PS2 free function, 652 B, seven console writes; no PC counterpart searched for |
| in game (needs MafiaCon) | the two consoles should be distinguishable — a pale-green block mid-screen that vanishes every frame, and a white log at the bottom that grows upward and holds each line for ten seconds |

**Related:** [input.md](input.md) (the previous TU) · [hud.md](hud.md) (the *other*
console — `hud_console_push_message`) · [net_session.md](net_session.md) (two more
instances) · [loadsave.md](loadsave.md).
