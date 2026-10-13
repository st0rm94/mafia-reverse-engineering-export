# The 3-D line batch — `[0x00591f20, 0x00592050)` (PC prefix `line_batch_*`)

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-10-05 11:17 +0200

A translation unit, not a class (like `structs/app_bootstrap.md`). Three functions and one
fixed buffer implement a **world-space line renderer**: gameplay code calls
`line_batch_add(a, b, colour_a, colour_b)` at any point during a frame, and the overlay pass
draws the whole accumulated set in one call and empties it. PS2 material is confined to §4.

---

## 1. The buffer, and why its geometry is certain

| symbol | VA | what it is |
|---|---|---|
| `g_line_batch_vertices` | `0x0067a888` | the vertex array, `0x75300` bytes |
| `g_line_batch_line_count` | `0x006efb88` | how many LINES are queued |
| `g_line_batch_write_ptr` | `0x0064ddf0` | bump pointer into the array |
| `g_line_batch_world_matrix` | `0x0063e0f0` | a 4×4 identity in `.rdata` |

One arithmetic identity settles the cap, the stride and the base at once:

```
0x0067a888 + 10000 * 2 * 0x18  =  0x006efb88
   base       cap    2/line  vtx      the counter's address
```

The counter sits **exactly** at the end of the array. Three independently read facts — the
`< 10000` gate in `line_batch_add`, the two `0x18`-byte writes per call, and the two symbol
addresses — agree without any of them being derived from another. A vertex is:

```c
typedef struct S_line_vertex {   /* 0x18 bytes */
    /* +0x00 */ float    x, y, z;
    /* +0x0c */ uint32_t colour;   // ARGB
    /* +0x10 */ float    u, v;     // texture coordinate 0; never written, so (0, 0)
} S_line_vertex;
```

`LS3DF.dll` builds its stream-0 vertex buffer with FVF `0x142` (`XYZ | DIFFUSE | TEX1`, stride
`0x18`) and the matching vertex declaration (position FLOAT3, diffuse D3DCOLOR, texture
coordinate 0 FLOAT2), so the trailing eight bytes are a UV pair. This TU never writes them, and
nothing outside it references the array: every reference to `g_line_batch_vertices` and
`g_line_batch_write_ptr` lies in the three functions below. The array is zero-filled BSS, so
every UV reads `(0, 0)`, and `line_batch_draw` unbinds the texture before drawing, so the UV has
no effect on the result.

## 2. The three functions

| PC VA | PC name | role |
|---|---|---|
| `0x00591f50` | `line_batch_add` | append one line: two vertices, count += 1 |
| `0x00591fe0` | `line_batch_draw` | flush the batch to the renderer |
| `0x00592030` | `line_batch_reset` | count = 0, write pointer back to the base |

**`line_batch_add(a, b, colour_a, colour_b)`** writes `a` then `b`, giving each its own
colour, so a line can fade along its length. Passing `0` for `colour_b` reuses `colour_a` —
that is how a solid line is drawn. Past 10000 lines the call is silently dropped; there is no
error path and no caller checks.

⚠️ **It is `__fastcall`.** The two `S_vector *` endpoints travel in ECX and EDX and are
**invisible** in every Ghidra call expression — the only arguments shown are the two colours. A
census of "what does every caller pass" is therefore a census of the COLOURS, not the geometry.

**`line_batch_draw()`** does nothing when the batch is empty. Otherwise, three engine calls:

```
g_engine->vt[0x0c](0)                          // unbind the texture
g_engine->vt[0x54](&g_line_batch_world_matrix) // install the IDENTITY world matrix
g_engine->vt[0x58](1, count, &vertices, 0)     // primitive type 1 = line list, stream 0
```

`g_engine` is `LS3DF.dll`'s `IGraph` interface (`structs/app_bootstrap.md` §3a), and the three
slots are read off the DLL's own bodies:

| slot | what the `LS3DF.dll` body does |
|---|---|
| `+0x0c` | sets a texture; with a null argument it calls the D3D8 device's `SetTexture(0, NULL)` |
| `+0x54` | calls the device's `SetTransform(D3DTS_WORLD, m)` |
| `+0x58` | maps the LS3D primitive type through a seven-entry table (type 1 = `D3DPT_LINELIST`), takes the vertex format from the stream type (0: stride `0x18` with the stored declaration, 1: stride `0x20` with FVF `0x1c4`) and calls the device's `DrawPrimitiveUP` |

The identity matrix is what makes this a **world-space** renderer: the caller supplies
absolute positions and nothing further transforms them. `g_line_batch_world_matrix`, read byte
by byte out of `.rdata`, is the literal identity; it has exactly **one** code reference in the
whole image, so it belongs to this TU and is not a general-purpose identity constant — hence
the qualified name.

## 3. Lifetime, and who uses it

The batch lives for exactly one frame. Its only flush site is
`engine_draw_overlay_callback` (`0x005f86c0`), the engine callback registered under id **6**
on the display singleton `[0x0065115c]` by `engine_init` (byte-verified at `0x005fa9b0`:
`push 6; push 0x005f86c0`). On message 2 that callback runs, in order:

```
line_batch_draw();  line_batch_reset();
hud_render_all();
if (g_gui_active_screen) gui_menu_draw(-1);
console_render_all();
```

so the lines are the **first** thing composited over the 3-D scene, under the HUD and the GUI.

**The only producer: `debris_tick` (`0x006223a0`), `C_Debris::Tick`.** `line_batch_add` has
exactly one reference in `Game.exe`, a direct call inside `debris_tick`: no tail jump reaches it
and no stored pointer holds its address, so no wrapper or callback adds lines. `line_batch_draw`
and `line_batch_reset` each have the one caller `engine_draw_overlay_callback`. `debris_tick`
advances a
`0x1c`-stride vector of debris pieces, each holding an `I3D_frame *`, integrating a per-record
velocity against a gravity vector on a `dt = clamp(ms, 1, 83) * 0.001` clock, and emits one line
per record from the previous position to the current one with a fading pale-yellow gradient
(`0xbfffffae` → `0x9fffffae`). A second `0x44`-stride vector — the spark flashes — is retired by
a swap-and-pop on a lifetime compare. Receiver byte-proven `C_game + 0x240`; see
[`debris.md`](debris.md).

## 4. PC vs PS2

`debris_tick` is `Tick__8C_DebrisFUi`, CONFIRMED on four constants shared with the twin in the
same order: the `0x53` ms clamp, the `6.2831855` spin wrap in both directions, the `-20.0` kill
and the `0.05` flash-arm gate.

**The line batch is PC-only**, measured at its only flush site and its only producer:

* `engine_draw_overlay_callback` is CONFIRMED against `cbScene__F20SCN_CALLBACK_MESSAGEUiUi`.
  On message 2 both draw the HUD, then the active menu, then the console, in the same order.
  The PS2 callback flushes no line batch; it draws the inventory panel
  (`DrawInventory__17G_IndicatorsClassFv`) and brackets the menu with `LSI_Setup` instead.
* `Tick__8C_DebrisFUi` makes no call at the position where `debris_tick` adds a line; its
  callees are vector and quaternion math only.

The PS2 `IGraph` vtable names the three engine slots of §2 `SetTexture`, `SetWorldMatrix` and
`DrawPrimitiveList` (PS2 offsets `+0x48`, `+0x88`, `+0x8c`). The two vtables are laid out
differently, so the slots match by body, not by offset.

A name search finds no twin either:

1. The whole PS2 symbol table by name — `Line`, `line`, `Cara`, `Usecka`, `Segment`, `Prid`,
   `Draw`, `Prim`, `Batch`, `Flush`.
2. The VA band `0x002e2000..0x002e8200` enumerated with the class column ignored, which is where
   the PS2 keeps its free-function drawing backend.

That band holds a contiguous drawing block — `Line__FffffUi`, `Rectangle__FffffUi`,
`RectangleFilled__FffffUiP8ITexture`, `RectangleProgress__FffffUifP8ITexture`,
`RectangleBB__FffffUi`, `Triangle__FffffUiUi`, `SetViewport__Fiiii` — and it is **not** this
subsystem: every one of those takes **four floats**, i.e. 2-D screen-space corners, while
`line_batch_add` takes two `S_vector *`. They are the GUI backend mapped in
`structs/gui_widget.md`: a name/shape collision, not a twin.

## 5. Related

* `Game-ghidra.exe.functions.md` — the function registry.
* `structs/collision.md` §2c — the TU map this range sits in.
* `structs/debris.md` — the producer's class.
* `structs/app_bootstrap.md` §3a — the `IGraph` interface behind `g_engine`.
