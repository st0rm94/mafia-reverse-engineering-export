# The 3-D line batch — `[0x00591f20, 0x00592050)` (PC prefix `line_batch_*`)

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-09-17 13:43 +0200

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
    /* +0x10 */ uint8_t  pad[8];   // unread by this TU; the FVF's texcoord slot
} S_line_vertex;
```

`0x18` = 12 + 4 + 8 is the classic D3D `XYZ | DIFFUSE | TEX1` vertex, consistent with the
trailing eight bytes never being written here.

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
g_engine->vt[0x0c](0)                          // drop the current texture / material
g_engine->vt[0x54](&g_line_batch_world_matrix) // install the IDENTITY world matrix
g_engine->vt[0x58](1, count, &vertices, 0)     // draw primitive type 1 = line list
```

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

**The located producer: `debris_tick` (`0x006223a0`), `C_Debris::Tick`.** It advances a
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

**No PS2 twin is located for any of the three `line_batch_*` functions, and none is tagged
terminal.** Searched:

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

A plausible reason for a genuine absence is that the PS2 draws through the GS directly rather
than through a D3D-style FVF vertex buffer, so a straight port of this TU would not survive.
That is a hypothesis; the status is *searched, not located*.

## 5. Related

* `Game-ghidra.exe.functions.md` — the function registry.
* `structs/collision.md` §2c — the TU map this range sits in.
* `structs/debris.md` — the producer's class.

## 6. Open items

* The PS2 side (§4).
* `g_engine` vtable slots `+0x0c`, `+0x54` and `+0x58` are read from their use here; none is
  confirmed against the `ls3df.dll` export list, which would name them properly.
* The trailing 8 bytes of the vertex are never written by this TU. If the FVF really declares
  a texcoord, the renderer reads uninitialised BSS for it — harmless with no texture bound, but
  unconfirmed.
* Producers other than `debris_tick` are not enumerated; `line_batch_add` has one textual
  callsite, but a `__fastcall` helper reached through a wrapper would not show.
