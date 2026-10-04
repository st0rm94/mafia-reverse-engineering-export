# The fake-shadow subsystem — `C_game`'s blob shadows

**Created** 2026-09-02 16:27 +0200 · **Modified** 2026-10-04 11:48 +0200

Part of the `g_game.cpp` translation unit, `[0x005dc190, 0x005ee3d0]`.

Not a class: a **feature of `C_game`**, built from four record types, one LS3D engine class
and six PC functions. It gets its own document because it spans three `C_game` field bands,
two shipped data joins and the largest single PC-vs-PS2 content difference in this TU.

Everything below is a **PC** fact unless it sits in §7, the one dedicated PS2 section. The record
TYPE NAMES come from the PS2 template arguments (the PC decomp cannot supply them); they name PC
structures.

---

## 1. What it is

Mafia does not cast real shadows for actors. It draws a **flat shadow mesh** underneath
them — a "fake shadow" — and the engine class that projects it is **`I3D_shadow`**.

The subsystem has three moving parts:

| part | where | what it holds |
|---|---|---|
| the **buffer table** | `C_game +0x284 .. +0x28c` | every shadow MESH the game preloads, keyed by file basename |
| the **item vectors** | `C_game +0x290` and `+0x2a0` | one record per live caster, split into two vectors by shadow type |
| the **LOD pass** | inlined inside `game_tick` | sorts the items by distance and keeps only the nearest N alive |

A caster registers once (`game_add_frame_for_shadow`), is bound to a mesh by name or index
(`game_bind_fake_shadow_buffer`), and is activated or deactivated every cycle by the LOD
pass according to the display setting.

---

## 2. The record types

```c
/* the buffer table element — stride 8 */
typedef struct {                    /* PS2: C_game::S_shadow_temp       */
/* +0x00 */ I3D_shadow *mesh;       /* created type 0xf, Open()ed from a path */
/* +0x04 */ char       *basename;   /* heap copy, path stripped; operator delete on teardown */
} S_shadow_temp;

/* the per-caster element — stride 0x1c */
typedef struct {                    /* PS2: C_game::S_shadow_item       */
/* +0x00 */ I3D_frame  *caster;     /* the frame that casts; the dedup key */
/* +0x04 */ I3D_shadow *shadow;     /* this item's own I3D_shadow */
/* +0x08 */ I3D_frame  *target;     /* optional target frame, arg 4 of AddFrameForShadow */
/* +0x0c */ char        active;     /* [H] set by the LOD pass when the item is live */
/* +0x10 */ char       *name;       /* [H] mesh basename, or NULL to use the index */
/* +0x14 */ uint        buffer_idx; /* [H] index into the +0x284 buffer table */
/* +0x18 */ float       dist_sq;    /* [H] distance to camera, recomputed each LOD cycle */
} S_shadow_item;
```

Both strides match the PC loop arithmetic exactly (8 and `0x1c`); the template arguments that
supply the names are listed in §7.

---

## 3. The `C_game` fields

| offset | field | evidence |
|---|---|---|
| `+0x284` / `+0x288` / `+0x28c` | buffer table begin / end / cap, stride 8 | filled 43× by `game_add_fake_shadow_buffer`; drained by `game_done_intern` |
| `+0x290` .. `+0x29c` | `S_shadow_item` vector for shadow type 0, stride `0x1c` | selected when the second `E_shadow_type` argument is 0 |
| `+0x2a0` .. `+0x2ac` | `S_shadow_item` vector for every other shadow type | selected otherwise |
| `+0x2b0` | LOD accumulator (ms) | `game_tick` adds the frame time to it |
| `+0x2b4` | LOD period (ms) | compared against `+0x2b0` |
| `+0x2b8` | LOD vector toggle | flipped each cycle, so the two item vectors are serviced alternately |
| `+0x3678` / `+0x367c` | recycled 0x20-byte name buffers | popped by `game_add_frame_for_shadow`, pushed back by `game_del_frame_for_shadow` |

The name free list is a real allocation optimisation, not scaffolding: a name is only
`operator new`'d when the free list is empty, and the size taken is `max(0x20, len + 1)`.

---

## 4. The PC functions

| PC VA | PC name | PS2 twin | status | role |
|---|---|---|---|---|
| `0x005e8b40` | `game_fake_shadow_compare_by_distance` | - | CONFIRMED | qsort comparator on `S_shadow_item +0x18`; only ever taken by address |
| `0x005e8b70` | `game_add_fake_shadow_buffer` | - | CONFIRMED | preload one mesh into the `+0x284` table |
| `0x005e8da0` | `game_bind_fake_shadow_buffer` | - | CONFIRMED | resolve an item's mesh by index or name and `SetTo` it |
| `0x005e8e30` | `game_add_frame_for_shadow` | `AddFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_typeP9I3D_framePc` | CONFIRMED | register a caster, create its `I3D_shadow` |
| `0x005e91d0` | `game_set_frame_for_shadow` | `SetFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_type` | STRUCTURAL | repoint an existing item at another buffer |
| `0x005e9350` | `game_del_frame_for_shadow` | `DelFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type` | STRUCTURAL | unregister a caster, recycle its name buffer |
| inlined in `game_tick` | the LOD pass | - | PC-only | distance sort + activate the nearest N; no PS2 code sorts the shadow items (§7) |
| - | - | `TickShadows__6C_gameFUi` | PS2-only | an empty 8-byte body |

The two STRUCTURAL rows cannot be raised: both PS2 bodies are `return 0`, so there is nothing
to body-diff against (§7).

---

## 5. The 43 preloaded meshes

`game_init` calls `game_add_fake_shadow_buffer` **43 times**, each with a literal path,
gated on the heavy-init flag `C_game+0x2f9c`:

* **4 generic** — `models\ShadowManHIGH.i3d`, `models\ShadowGirlHIGH.i3d`,
  `models\ShadowMan.i3d`, `models\ShadowGirl.i3d`
* **1 default vehicle** — `models\car_shadow.i3d`
* **38 per-car** — `alfaS`, `buick_coupS`, `deliveryS`, `fordAcabS`, `fordAtudS`,
  `fordor_bigS`, `fordorS`, `fordTrunS`, `hudCoupS`, `chryslerS`, `mercedS`, `phantomS`,
  `pickupS`, `pierceS`, `truckAS`, `truckBS`, `bugattiS`, `duesenRacingS`, `milerS`,
  `Fv8SHDW`, `xedosS`, `broukS`, `toyotaS`, `cord_sedanhs`, `flamer00s`, `fordapicks`,
  `fordthd`, `fordtrons`, `hotrodp2s`, `hotrodp3s`, `hotrodp4s`, `hotrodp5s`,
  `chevroletm6hs`, `hotrodp6sh`, `deliverys`, `thunders`, `fordths`, `hotrodp6s`

The trailing `S` / `s` on the car meshes is a shadow suffix, and the `HIGH` pair is a
quality variant of the human shadow.

🔑 **The join key is a shipped-data field.** A car registers its shadow with
`game_add_frame_for_shadow(frame, 1, 4, car+0x2118, name)` where `name` comes from
`car_database_get_shadow_model(car+0x20cc)` — i.e. from the car's `carindex.def` record.
`game_bind_fake_shadow_buffer` then matches that string case-insensitively against the
basenames `game_add_fake_shadow_buffer` stored. So the table, the `.def` field and the bind are one chain.

---

## 6. Lifecycle

```
game_init          -> game_add_fake_shadow_buffer x43        (fills +0x284)
<car spawns>       -> car_database_get_shadow_model
                   -> game_add_frame_for_shadow(...,name)    (adds an S_shadow_item)
game_tick, per LOD -> recompute +0x18 distances
                   -> crt_qsort with game_fake_shadow_compare_by_distance
                   -> nearest N: game_bind_fake_shadow_buffer + LinkTo + show
                      the rest:  detach + release + hide
<car despawns>     -> game_del_frame_for_shadow               (recycles the name buffer)
game_done_intern   -> destroy every buffer, operator delete every basename
```

N is `mission_settings_get_shadow_quality_idx()` — used as N in the sorted list and nowhere else —
so the shadow **count** is the graphics setting, not the shadow resolution. A car whose
`carindex.def` shadow-model field names a mesh outside the 43 preloaded ones gets no shadow:
`game_bind_fake_shadow_buffer` binds nothing when no basename matches.

---

## 7. PC vs PS2

**Record type names.** The PC mesh table's element is `S_shadow_temp`: PS2 `C_mission::Init`
constructs a `vector<S_shadow_temp>` at PS2 `C_game+0x2c4`, and PS2 `DoneIntern` releases it the
way the PC `game_done_intern` releases `+0x284` — stride 8, the shadow object, then a heap name
freed with `delete[]`. `S_shadow_item` comes from `std::__vector_imp<…C_game13S_shadow_item…>::insert`,
and PS2 `Init` constructs two of those vectors, at `+0x2d0` and `+0x2dc`. `S_fake_shadow_buff` and
`S_fake_shadow_hum` belong to the PS2-only person shadows below.

**The mesh table is the difference.** The PS2 ELF contains **zero** occurrences of
`ShadowMan`, `ShadowGirl` or `car_shadow`, and exactly one shadow asset string:
**`shadow.I3D`**. The PC ships 43 meshes and a per-car lookup; the PS2 ships one mesh for
everything. This is a content difference, not a port artifact — the code that would consume
a table is present on both sides.

**The PS2 has a second shadow system, for people only, with no PC counterpart.** PS2
`AddFakeShadow__6C_gameFP9I3D_frameP7C_actor` claims a slot of the stride-8 `S_fake_shadow_buff`
pool at `+0x1678` (a slot whose `+4` byte is 0) and records the caster in the stride-`0xc`
`S_fake_shadow_hum` vector at `+0x1684`; `DelFakeShadow` releases it. `TickFakeShadows` then places
each blob under its person every frame: within 20 units of the camera at the person's frame,
farther out at the centre of a footprint that `cbFindMinMax` gathers from the type-10 frames below
it. Every caller of `AddFakeShadow` / `DelFakeShadow` is a `C_human`, `C_player`,
`C_traffic_element`, `C_traffic_generator` or script method, and each pairs the call with
`AddFrameForShadow` / `DelFrameForShadow` in the next statement. The PC twins keep only
`game_add_frame_for_shadow` / `game_del_frame_for_shadow`: PS2 `Death__7C_humanFv` runs
`DelFrameForShadow`, `DelFakeShadow`, `PoseNullRotate`, `DeactivateColls` where `human_death` runs
`game_del_frame_for_shadow`, `human_pose_null_rotate`, `human_deactivate_colls`, and
`GameInit__7C_humanFv` lines up with `human_game_init` the same way. The four methods and both
record types are PS2-only.

**Field pairs**

| PC | PS2 | what |
|---|---|---|
| `+0x284` | `+0x2c4` (count `+0x2c8`, ptr `+0x2cc`) | the `S_shadow_temp` vector; matched through `DoneIntern` |
| `+0x290` / `+0x2a0` | `+0x2d0` / `+0x2dc` | the two `S_shadow_item` vectors, stride `0x1c` on both |
| `+0x3678` / `+0x367c` | `+0x1560` / `+0x1564` | the recycled-name free list |

**Gutted on the PS2, live on the PC** (the PS2 method count is a lower bound here):
`SetFrameForShadow` and `DelFrameForShadow` are both `return 0`, and `TickShadows` is empty. All
three bodies were read, not inferred from size.

**No PS2 code sorts the shadow items.** Apart from the teardown's `clear`, `AddFrameForShadow` is
the only PS2 code that touches the two `S_shadow_item` vectors, so the PC's distance-sorted LOD pass
in `game_tick` has no PS2 counterpart.

---

## 8. `I3D_shadow` — the engine class behind it

`I3D_shadow` lives in **`ls3df.dll`** on the PC, so `Game.exe` only ever holds pointers to
it. The PS2 compiled the engine in, which is where the API below comes from.
It is created by the driver factory with **object type `0xf`** — the same constant on both
builds, and the only use of that type anywhere in the PC decomp.

| method | used by |
|---|---|
| `Open(const char *, u32)` | `game_add_fake_shadow_buffer` (vtable `+0x50`) |
| `SetTo(I3D_frame *)` | `game_bind_fake_shadow_buffer` (vtable `+0x3c`) |
| `LinkTo(I3D_frame *, u32)` | the LOD pass and `game_del_frame_for_shadow` (vtable `+0x2c`) |
| `Duplicate`, `SetRange`, `SetColor`, `SetTextureSize`, `SetOrthogonal`, `SetMaxTextureDistance`, `SetReverseFactor`, `SetShadowLock`, `DeleteFromAnyShadow`, `UpdateV`, `Close` | not reached from this subsystem |

---

## 9. Related

`structs/game.md` (the owning singleton and its field map) ·
`structs/car_database.md` (`car_database_get_shadow_model`, the `carindex.def` join) ·
`ghidra/Game-ghidra.exe.functions.md` (the function registry).

---

## 10. Open items

| item | state |
|---|---|
| in game: shadow-quality setting | expected to reduce the NUMBER of actors casting a shadow (nearest-first), not the resolution (§6); not observed |
| in game: unmatched shadow-model name | a car whose `carindex.def` shadow model is not among the 43 meshes is expected to get no shadow (§6); not observed |
