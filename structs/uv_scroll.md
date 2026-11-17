# `C_tshift` — the UV-scroll animator

**Created** 2026-09-03 19:14 +0200 · **Modified** 2026-11-09 09:53 +0100

The small class that scrolls texture coordinates on named scene objects; in the shipped game a
projector's film reels, a UFO and water and shore surfaces (§4a). One instance hangs off
`C_mission + 0x28`. Its record builders live in the `Mission.cpp` unit; its two tick functions and
its two folded construction helpers sit in the class's own unit, `tshift.cpp`.

| | |
|---|---|
| PC prefix | `uv_scroll_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x10` (`operator_new(0x10)` in `mission_init`) |
| PC constructor | `folded_vector_ctor_at_0x04` @ `0x0061e010`, a body shared with `C_blood_manager` (§3a) |
| PC translation unit | `[0x0061df00, 0x0061e0d0)`, `tshift.cpp`: the unit's CRT pair, `uv_scroll_channel_tick`, the two folds and `uv_scroll_tick`; the three record builders sit in `Mission.cpp` (`structs/mission.md`) |
| PS2 class | `C_tshift` (tag `8C_tshift`), 3 methods, and its element `S_tshift_elem` (tag `13S_tshift_elem`), 1 method with a body; all mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_tshift {                  // 0x10 bytes; at *(C_mission + 0x28)
    /* +0x00 */ uint8_t        alloc;      // [H] the MSVC allocator byte of the vector
    /* +0x04 */ S_tshift_elem *begin;      // [H] the channel vector
    /* +0x08 */ S_tshift_elem *end;        // [H] one past the last channel
    /* +0x0c */ S_tshift_elem *cap;        // [H] end of the vector's storage
} C_tshift;
```

The object is the vector and nothing else. Who owns it:

| what | where |
|---|---|
| allocated by | `mission_init` (`operator_new(0x10)`, then `folded_vector_ctor_at_0x04`) |
| destroyed by | `mission_uninit` (`folded_vector_free_at_0x04`) |
| driven by | `mission_tick`, every frame, while the pointer is non-null |
| emptied by | `mission_close` (`uv_scroll_clear_channels`) |
| filled by | `mission_open_bin`, from the `0xAE02` chunk of `scene2.bin` (§4) |
| read through | `mission_get_uv_scroll` (`structs/mission.md`) |

### 1a. `S_tshift_elem` — one scroll channel, `0x1c` bytes

The PC decomp gives the stride; the PS2 symbol `Tick__13S_tshift_elemFf` gives the type name.

```c
typedef struct S_tshift_elem {             // 0x1c bytes
    /* +0x00 */ I3D_visual *target;        // [H] the mesh visual whose UV offsets are written
    /* +0x04 */ I3D_frame  *frame;         // [H] the scene frame named by the record
    /* +0x08 */ float       du_per_sec;    // [H] U scroll rate
    /* +0x0c */ float       dv_per_sec;    // [H] V scroll rate
    /* +0x10 */ float       u;             // [H] accumulated U, wrapped into [-1, 1]
    /* +0x14 */ float       v;             // [H] accumulated V, wrapped into [-1, 1]
    /* +0x18 */ uint16_t    channel_mask;  // [H] bit 0x1 -> visual +0x5c/+0x60, bit 0x2 -> visual +0x64/+0x68
} S_tshift_elem;
```

Bytes `+0x1a..+0x1b` are padding. `uv_scroll_channel_init` writes the mask as a 16-bit value and
`uv_scroll_channel_tick` tests its low byte.

## 2. Field evidence

Measured by a census of the decomp's accesses in the `uv_scroll_*` functions and the two folds, and
a body read of each.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | alloc | H | `folded_vector_ctor_at_0x04` | - | allocator byte |
| +0x04 | begin | H | `folded_vector_ctor_at_0x04`, `folded_vector_free_at_0x04`, `uv_scroll_add_channel` | `uv_scroll_add_channel`, `uv_scroll_tick`, `mission_close` | - |
| +0x08 | end | H | `folded_vector_ctor_at_0x04`, `folded_vector_free_at_0x04`, `uv_scroll_add_channel`, `uv_scroll_clear_channels` | `uv_scroll_add_channel`, `uv_scroll_clear_channels`, `uv_scroll_tick`, `mission_close` | - |
| +0x0c | cap | H | `folded_vector_ctor_at_0x04`, `folded_vector_free_at_0x04`, `uv_scroll_add_channel` | `uv_scroll_add_channel` | - |

### 2a. `S_tshift_elem`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | target | H | `uv_scroll_channel_init` | `uv_scroll_channel_tick` | from `i3d_frame_get_mesh_object_ptr` (`frame + 0x1e0`) in `mission_open_bin` |
| +0x04 | frame | H | `uv_scroll_channel_init` | - | stored for the record; the tick writes through `target` only |
| +0x08 | du_per_sec | H | `uv_scroll_channel_init` | `uv_scroll_channel_tick` | from the file record |
| +0x0c | dv_per_sec | H | `uv_scroll_channel_init` | `uv_scroll_channel_tick` | from the file record |
| +0x10 | u | H | `uv_scroll_channel_init` (0), `uv_scroll_channel_tick` | `uv_scroll_channel_tick` | §4b |
| +0x14 | v | H | `uv_scroll_channel_init` (0), `uv_scroll_channel_tick` | `uv_scroll_channel_tick` | §4b |
| +0x18 | channel_mask | H | `uv_scroll_channel_init` | `uv_scroll_channel_tick` | §4a |

## 3. Method family

Coverage: all **4** PS2 methods with a body are mapped: `C_tshift::Tick` and `S_tshift_elem::Tick`
CONFIRMED, the `C_tshift` constructor and destructor STRUCTURAL on the two folds (§5b). The three
record builders are PC-only shapes: the PS2 builds each record inline at its insert site.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x006047a0 | `uv_scroll_channel_init` | a channel record | - | - | - | `this = S_tshift_elem` fills one `S_tshift_elem` (zeroes `u` / `v`); called once, by `mission_open_bin` |
| 0x0061df00 | `crt_static_init_guard_0061df00` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0061df20 | `crt_atexit_register_0061df20` | the translation unit | - | - | - | registers `crt_atexit_noop_dtor_0060b700` |
| 0x0061e010 | `folded_vector_ctor_at_0x04` | instance | `__ct__8C_tshiftFv` | 0x003b6150 | STRUCTURAL | the constructor: allocator byte + empty vector. A two-owner fold: the same body constructs `C_blood_manager` at `C_game + 0x2d90` (`structs/blood.md`); a byte copy sits at `0x0044a120` |
| 0x0061e030 | `folded_vector_free_at_0x04` | instance | `__dt__8C_tshiftFv` | 0x003b61a0 | STRUCTURAL | the destructor: frees `begin` and zeroes the triple; the same two owners |

### 3b. Accessors

The class has no accessor; `mission_get_uv_scroll` is a `C_mission` accessor (`structs/mission.md`).

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x006047d0 | `uv_scroll_add_channel` | - | - | - | appends one record (`push_back`), growing the vector; called once, by `mission_open_bin` |
| 0x00607400 | `uv_scroll_clear_channels` | - | - | - | erases a range (`mission_close` passes the whole vector) |
| 0x0061df30 | `uv_scroll_channel_tick` | `Tick__13S_tshift_elemFf` | 0x003b5e50 | CONFIRMED | `this = S_tshift_elem` integrate, wrap and store one channel (§4b) ¹ |
| 0x0061e050 | `uv_scroll_tick` | `Tick__8C_tshiftFUi` | 0x003b6210 | CONFIRMED | walks the records with `dt_ms * 0.001`; called by `mission_tick` |

¹ A member of the element type `S_tshift_elem`, not of `C_tshift`, so a search keyed on the class
name does not return it.

### 3d. Free helpers

The unit holds no free helper. The `Mission.cpp` builders call the generic stride-`0x1c` container
helpers (`stl_fill_n_stride_0x1c`, `stl_uninit_copy_stride_0x1c`, `stl_construct_stride_0x1c`),
which serve many classes and are listed with their own units.

## 4. The scene records and the two channels

### 4a. The scene-file record

Under the root chunk `0x4c53` of `scene2.bin`, one `0xAE02` chunk holds one `0xAE03` child per
channel (`structs/mission.md` §6a). Each child is the 6-byte chunk header followed by:

| Offset | Size | Field | Stored at |
|---|---|---|---|
| `+0x06` | 2 | channel mask | `+0x18` |
| `+0x08` | 4 | U rate, float | `+0x08` |
| `+0x0c` | 4 | V rate, float | `+0x0c` |
| `+0x10` | 4 | name length *n* | - |
| `+0x14` | *n* | frame name, no terminator | resolved to `+0x04` |

The chunk size is exactly `0x14 + n`: all eight shipped records decode to it with nothing left over.
The named frame must be a visual (frame type 1) tagged `'LMAP'` or `'OBJ_'` that carries a mesh
object; a record naming any other frame is skipped.

Channel 1 (mask `0x1`, `+0x5c` / `+0x60`) shifts the mesh's base texture coordinates (§5a). Every
channel the shipped scenes author uses it. The `0xAE03` records of every shipped `scene2.bin` hold
eight channels in three scenes:

| Scene | Frame | Mask | U rate | V rate |
|---|---|---|---|---|
| `00MENU` | `9promitac.Cylinder05` | `0x1` | 0.01 | 0 |
| `00MENU` | `9promitac.Cylinder18` | `0x1` | -0.05 | 0 |
| `EXTREME` | `UfoTalir.Cylinder02` | `0x1` | -0.4 | 1.0 |
| `EXTREME` | `UfoTalir.Cylinder01` | `0x1` | 0.5 | 0.8 |
| `MISE14-PARNIK` | `Cylinder01` | `0x1` | -0.05 | 0 |
| `MISE14-PARNIK` | `pobrezi2.Object02` | `0x1` | 0.05 | 0 |
| `MISE14-PARNIK` | `Box488` | `0x1` | 0 | -0.05 |
| `MISE14-PARNIK` | `Object66` | `0x1` | 0.08 | 0 |

`9promitac` is the main menu's film projector (*promitac* = projector). Channel 2 (mask `0x2`,
`+0x64` / `+0x68`) is used by no shipped scene. The `LS3DF.dll` code that consumes `+0x64` / `+0x68`
is not identified; this class only stores the offsets.

### 4b. Why the name is `uv_scroll`

`tshift` is the developers' abbreviation (*texture shift*). The PC name is taken from the mechanism,
which `uv_scroll_channel_tick` states in full:

```c
u += dt * du_per_sec;
if (u >  1.0f) u -= floorf(u);         // wrap into one texture period
if (u < -1.0f) u -= ceilf(u);
v += dt * dv_per_sec;                  // same
if (mask & 1) { target->+0x5c = u; target->+0x60 = v; }
if (mask & 2) { target->+0x64 = u; target->+0x68 = v; }
```

Two float pairs, integrated against real time, wrapped to unit period and written into an
`I3D_visual`'s two UV-offset slots: a texture-coordinate scroller and nothing else.

## 5. PC vs PS2

PS2 class `C_tshift`, object at `C_mission + 0x24`, with the same lazy allocation inside
`Init__9C_missionFP10I3D_driver` (`__nw__FUi(0xc)` → `__ct__8C_tshiftFv`), the same teardown inside
`UnInit__9C_missionFv`, the same `Tick__8C_tshiftFUi` call from `Tick__9C_missionFi` and the same
`clear` from `Close__9C_missionFv`. The PS2 object is `0xc` bytes to the PC's `0x10`: the PC's
leading allocator byte is an MSVC `std::vector` artifact the Metrowerks STL does not have.

### 5a. The element tick: 224 B against 760 B

Matched: the same field indices `[2]` du, `[3]` dv, `[4]` u, `[5]` v; the same mask at `+0x18`; the
same integrate-then-wrap against ±1.0 with `floor` on overflow and `ceil` on underflow; the same
`dt_ms / 1000.0`; the same walk at stride `0x1c`. After the wrap the two implementations differ:

* the **PC** stores the offsets into the visual's UV-offset registers at `+0x5c` / `+0x60` (mask
  `0x1`) and `+0x64` / `+0x68` (mask `0x2`), two channels, and the hardware does the rest;
* the **PS2** locks the mesh's vertex buffer (`GetLOD__15I3D_mesh_objectCFi`,
  `LockVertices__14I3D_mesh_levelFUi`, `UnlockVertices__14I3D_mesh_levelFv`) and adds the delta to
  **every vertex's UV pair**, eight vertices per unrolled iteration, then notifies the visual through
  vtable `+0xcc`. It honours **mask `0x1` only**. The vertex it edits is `0x20` bytes with a single
  UV pair at `+0x18` (position, normal, texture coordinate), which is what identifies mask `0x1` as
  the base texture coordinates.

So the PC has a second scroll channel (mask `0x2`) the PS2 does not, and the PS2 pays per vertex for
the first.

### 5b. The constructor and destructor on the folds

`__ct__8C_tshiftFv` constructs the Metrowerks allocator pair and zeroes the two remaining words of
the vector; `folded_vector_ctor_at_0x04` writes the allocator byte and zeroes the triple.
`__dt__8C_tshiftFv` runs the vector deleter and deletes on its flag; `folded_vector_free_at_0x04`
frees `begin` and zeroes the triple. The statements agree one for one across two different Standard
Libraries, so the twins are STRUCTURAL. `C_blood_manager`'s constructor and destructor have the same
bodies on the PS2 and are the same two PC folds.

## 6. Related

- `structs/mission.md` — the owner (`C_mission + 0x28`), the scene loader and the three builders.
- `structs/blood.md` — the other owner of the two folds.
- `structs/i3d_frame.md` — `i3d_frame_get_mesh_object_ptr` (`frame + 0x1e0`).

## 7. Open items

1. **In game:** a scene object with a channel should scroll its texture continuously and seamlessly
   (the wrap is one period); the shipped channels are the main-menu projector reels, the `EXTREME`
   UFO and four objects in `MISE14-PARNIK` (§4a).
2. **In game:** a PS2 surface using mask `0x2` would not scroll at all (§5a); no shipped scene uses
   it, so only an edited scene can show it.
