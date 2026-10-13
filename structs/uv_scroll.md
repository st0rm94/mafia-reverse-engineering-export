# `C_tshift` — the UV-scroll animator

**Created** 2026-09-03 19:14 +0200 · **Modified** 2026-10-11 15:07 +0200

The small class that scrolls texture coordinates on named scene objects — in the shipped game
a projector's film reels, a UFO and water and shore surfaces (§3a). Its record builders live
in the `Mission.cpp` translation unit (`[0x005fe550, 0x00607f70]`); its two tick functions sit
at `0x0061df30` and `0x0061e050`, outside it. PC prefix **`uv_scroll_`**.

Related: `structs/mission.md` §3 (the owning member) · `structs/i3d_frame.md` (the
`i3d_frame_get_mesh_object_ptr` accessor at `frame+0x1e0`).

---

## 1. The object

| what | where |
|---|---|
| the object | `C_mission + 0x28`, a pointer to a `0x10`-byte heap allocation |
| allocated by | `mission_init` (`operator_new(0x10)` → `folded_vector_ctor_at_0x04`) |
| destroyed by | `mission_uninit` (`folded_vector_free_at_0x04`) |
| driven by | `mission_tick`, every frame, guarded on the pointer being non-null |
| cleared by | `mission_close` (`uv_scroll_clear_channels`) |
| filled by | `mission_open_bin`, from the `0xAE02` chunk of `scene2.bin` (§2a) |
| accessor | `mission_get_uv_scroll` |

```c
typedef struct C_tshift {          /* 0x10 bytes */
/* +0x00 */ uint8_t alloc;         // [H] the MSVC allocator byte
/* +0x04 */ S_uv_scroll_channel *begin;   // [H] the channel vector
/* +0x08 */ S_uv_scroll_channel *end;     // [H]
/* +0x0c */ S_uv_scroll_channel *cap;     // [H]
} C_tshift;
```

## 2. The channel record — `0x1c` bytes

Type name `C_mission::S_tshift_elem` (the PC decomp gives the stride, only the PS2 symbol gives
the type).

```c
typedef struct S_uv_scroll_channel {   /* 0x1c bytes */
/* +0x00 */ I3D_visual *target;        // [H] from i3d_frame_get_mesh_object_ptr (frame+0x1e0)
/* +0x04 */ I3D_frame  *frame;         // [H] the owning scene frame
/* +0x08 */ float du_per_sec;          // [H] U scroll rate
/* +0x0c */ float dv_per_sec;          // [H] V scroll rate
/* +0x10 */ float u;                   // [H] accumulated U, wrapped into [-1, 1]
/* +0x14 */ float v;                   // [H] accumulated V, wrapped into [-1, 1]
/* +0x18 */ uint8_t channel_mask;      // [H] 0x1 -> visual+0x5c/+0x60
                                       //     0x2 -> visual+0x64/+0x68
} S_uv_scroll_channel;
```

### 2a. The scene-file record

Under the root chunk `0x4c53` of `scene2.bin`, one `0xAE02` chunk holds one `0xAE03` child per
channel (`structs/mission.md` §4a). Each child is the 6-byte chunk header followed by:

| offset | size | field | stored at |
|---|---|---|---|
| `+0x06` | 2 | channel mask | `+0x18` |
| `+0x08` | 4 | U rate, float | `+0x08` |
| `+0x0c` | 4 | V rate, float | `+0x0c` |
| `+0x10` | 4 | name length *n* | - |
| `+0x14` | *n* | frame name, no terminator | resolved to `+0x04` |

The chunk size is exactly `0x14 + n`: all eight shipped records (§3a) decode to it with nothing
left over. The named frame must be a visual (frame type 1) tagged `'LMAP'` or `'OBJ_'` that carries
a mesh object; a record naming any other frame is skipped.

## 3. 🔑 Why the name is `uv_scroll` and not `tshift`

`tshift` is the developers' abbreviation and means nothing to a reader who has not seen the PS2
symbols. The PC name is taken from the mechanism, which `uv_scroll_channel_tick` states in full:

```c
u += dt * du_per_sec;
if (u >  1.0f) u -= floorf(u);         // wrap into one texture period
if (u < -1.0f) u -= ceilf(u);
v += dt * dv_per_sec;                  // same
if (mask & 1) { target->+0x5c = u; target->+0x60 = v; }
if (mask & 2) { target->+0x64 = u; target->+0x68 = v; }
```

Two float pairs, integrated against real time, wrapped to unit period, written into an
`I3D_visual`'s two UV-offset slots. That is a texture-coordinate scroller and nothing else — so
`tshift` decodes as **texture shift**, and the PC name says so in English.

The channels are authored per scene object: `mission_open_bin` builds one per `0xAE03` record
(§2a), reading the two rates and the mask out of the file and resolving the target with
`i3d_frame_get_mesh_object_ptr`.

### 3a. What the two channels are, and which one ships

Channel 1 (mask `0x1`, `+0x5c`/`+0x60`) shifts the mesh's base texture coordinates (§5).
Every channel the shipped scenes author uses it. The `0xAE03` records of every shipped
`scene2.bin` hold eight channels in three scenes:

| scene | frame | mask | U rate | V rate |
|---|---|---|---|---|
| `00MENU` | `9promitac.Cylinder05` | `0x1` | 0.01 | 0 |
| `00MENU` | `9promitac.Cylinder18` | `0x1` | -0.05 | 0 |
| `EXTREME` | `UfoTalir.Cylinder02` | `0x1` | -0.4 | 1.0 |
| `EXTREME` | `UfoTalir.Cylinder01` | `0x1` | 0.5 | 0.8 |
| `MISE14-PARNIK` | `Cylinder01` | `0x1` | -0.05 | 0 |
| `MISE14-PARNIK` | `pobrezi2.Object02` | `0x1` | 0.05 | 0 |
| `MISE14-PARNIK` | `Box488` | `0x1` | 0 | -0.05 |
| `MISE14-PARNIK` | `Object66` | `0x1` | 0.08 | 0 |

`9promitac` is the main menu's film projector ("promitac" = projector). Channel 2 (mask `0x2`,
`+0x64`/`+0x68`) is used by no shipped scene. The `LS3DF.dll` code that consumes `+0x64`/`+0x68`
is not identified; this class only stores the offsets.

## 4. PC functions

| PC VA | PC name | role |
|---|---|---|
| `0x006047a0` | `uv_scroll_channel_init` | fill one channel record |
| `0x006047d0` | `uv_scroll_add_channel` | append it to the vector |
| `0x00607400` | `uv_scroll_clear_channels` | erase a range (empties the vector) |
| `0x0061e050` | `uv_scroll_tick` | per-frame driver, `dt_ms * 0.001` |
| `0x0061df30` | `uv_scroll_channel_tick` | the integrate-wrap-store body above |
| `0x00606390` | `mission_get_uv_scroll` | `C_mission + 0x28` |

🔴 **The class ctor and dtor are NOT in this family.** `0x0061e010` and `0x0061e030` are proven
two-owner folds: each has two callsites with disagreeing receivers — this class at
`C_mission + 0x28`, and `C_blood_manager` at `C_game + 0x2d90`. Both are trivial "zero a vector
triple" bodies, which is exactly what identical-COMDAT folding merges. They therefore carry
`folded_vector_ctor_at_0x04` / `folded_vector_free_at_0x04` and cannot take a class-qualified name.
`0x0061e010` is not even byte-unique — a second copy sits at `0x0044a120`
(`folded_vector_ctor_at_0x04_0044a120`).

## 5. PC vs PS2

PS2 class `C_tshift`, object at `C_mission + 0x24`, with the same lazy allocation inside
`Init__9C_missionFP10I3D_driver` (`__nw__FUi(0xc)` → `__ct__8C_tshiftFv`), the same teardown
inside `UnInit__9C_missionFv`, the same `Tick__8C_tshiftFUi` call from `Tick__9C_missionFi` and
the same `clear` from `Close__9C_missionFv`.

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| `0x0061e050` | `uv_scroll_tick` | `Tick__8C_tshiftFUi` | **CONFIRMED** |
| `0x0061df30` | `uv_scroll_channel_tick` | `Tick__13S_tshift_elemFf` | **CONFIRMED** ¹ |
| all others | - | - | no PS2 counterpart function; the PS2 builds its records inline at the `insert` callsite |

¹ The method is a member of the element type **`S_tshift_elem`**, not of `C_tshift`, so a search
keyed on the class name does not return it.

Matched: same field indices `[2]` du, `[3]` dv, `[4]` u, `[5]` v; same mask byte at `+0x18`; same
integrate-then-wrap against ±1.0 with `floor` on overflow and `ceil` on underflow; same
`dt_ms / 1000.0`; same backward walk at stride `0x1c`.

🔴 **A real build difference — the bodies are 224 B against 760 B.** After the wrap the two
implementations diverge completely:

* the **PC** stores the accumulated offsets into the visual's UV-offset registers at
  `+0x5c`/`+0x60` (mask `0x1`) and `+0x64`/`+0x68` (mask `0x2`) — two channels, and the hardware
  does the rest;
* the **PS2** locks the mesh's vertex buffer (`GetLOD__15I3D_mesh_objectCFi`,
  `LockVertices__14I3D_mesh_levelFUi`, `UnlockVertices__14I3D_mesh_levelFv`) and adds the delta to
  **every vertex's UV pair**, eight vertices per unrolled iteration, then notifies the visual
  through vtable `+0xcc`. It honours **mask `0x1` only**. The vertex it edits is `0x20`
  bytes with a single UV pair at `+0x18` (position, normal, texture coordinate), which is what
  identifies mask `0x1` as the base texture coordinates (§3a).

So the PC has a second scroll channel (mask `0x2`) the PS2 does not, and the PS2 pays per-vertex
for the first. **In game:** a PS2 surface using the second channel would not scroll at all.

The PS2 object is `0xc` bytes to the PC's `0x10`: the PC's leading allocator byte is an MSVC
`std::vector` artifact the Metrowerks STL does not have.

## 6. Open items

| item | state |
|---|---|
| in-game check | any scene object with a `tshift` channel should scroll its texture continuously and seamlessly (the wrap is one period); the shipped channels are the main-menu projector reels, the `EXTREME` UFO and four objects in `MISE14-PARNIK` (§3a) |
