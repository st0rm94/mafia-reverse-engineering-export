# The fake-shadow subsystem — `C_game`'s blob shadows

**Created** 2026-09-02 16:27 +0200 · **Modified** 2026-11-01 19:11 +0100

A **feature of `C_game`**, not a class, built from two record types, one LS3D engine class and six PC
functions. It has its own document because it spans three `C_game` field bands, two shipped data joins and the largest
single PC-vs-PS2 content difference in this translation unit.

Mafia does not cast real shadows for actors. It draws a **flat shadow mesh** underneath them, a "fake shadow", and the
engine class that projects it is **`I3D_shadow`**. The record type names come from the PS2 template arguments (§8), which
the PC decomp cannot supply; they name PC structures.

| | |
|---|---|
| PC prefix | `game_fake_shadow_`, `game_add_fake_shadow_`, `game_bind_fake_shadow_`, `game_add_frame_for_`, `game_set_frame_for_`, `game_del_frame_for_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x005dc160, 0x005f79a0)`, `g_game.cpp`, hosted by `structs/game.md`; the functions here lie in `[0x005e8b40, 0x005e94b0)` |
| PS2 class | `C_game` (tag `6C_game`): 3 methods mapped here, 4 PS2-only (§8); the rest are in `structs/game.md` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

The subsystem has three moving parts:

| Part | Where | What it holds |
|---|---|---|
| the **buffer table** | `C_game.shadow_meshes` (§4) | every shadow MESH the game preloads, keyed by file basename |
| the **item vectors** | `C_game.shadow_items_a` and `shadow_items_b` | one record per live caster, split by shadow type |
| the **LOD pass** | inlined in `game_tick` and once in `game_init` | sorts the items by distance and keeps only the nearest N alive |

A caster registers once (`game_add_frame_for_shadow`), is bound to a mesh by name or index
(`game_bind_fake_shadow_buffer`), and is activated or deactivated by the LOD pass according to the display setting. Neither
record holds an array.

### 1a. `S_shadow_temp` — the buffer table element, stride 8

```c
typedef struct S_shadow_temp {
    /* +0x00 */ I3D_shadow *mesh;    // [H] the source mesh, created as driver type 0xf and opened from a path
    /* +0x04 */ char *basename;      // [H] heap copy of the path's file name, the key the bind compares
} S_shadow_temp;
```

### 1b. `S_shadow_item` — the per-caster element, stride `0x1c`

```c
typedef struct S_shadow_item {
    /* +0x00 */ I3D_frame *caster;       // [H] the frame that casts; the key every lookup compares
    /* +0x04 */ I3D_shadow *shadow;      // [H] this item's own I3D_shadow
    /* +0x08 */ I3D_frame *stand_in;     // [H] optional frame switched off while the item is live, and on otherwise
    /* +0x0c */ char active;             // [H] 1 while the LOD pass keeps the item live
    /* +0x10 */ char *name;              // [H] mesh basename, or NULL to bind by buffer_index
    /* +0x14 */ uint32_t buffer_index;   // [H] index into the buffer table, used when name is NULL
    /* +0x18 */ float distance_sq;       // [H] squared distance the LOD pass sorts by
} S_shadow_item;
```

## 2. Field evidence

The records are reached through the vector begins, so these cells come from reading the named bodies, not from a census.
`game_add_frame_for_shadow` builds the record on its stack and copies it into the vector; the decomp hides those stores, and
they are read from the disassembly at `0x005e8f83`.

### 2a. `S_shadow_temp`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `mesh` | H | `game_add_fake_shadow_buffer` | `game_bind_fake_shadow_buffer`, `game_done_intern`, `game_set_frame_for_shadow` | `SetTo` target of the bind |
| +0x4 | `basename` | H | `game_add_fake_shadow_buffer`, `game_done_intern` | `game_bind_fake_shadow_buffer`, `game_done_intern`, `game_set_frame_for_shadow` | compared with `crt_stricmp`; `game_done_intern` frees and zeroes it |

### 2b. `S_shadow_item`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `caster` | H | `game_add_frame_for_shadow` | `game_add_frame_for_shadow`, `game_del_frame_for_shadow`, `game_done`, `game_init`, `game_set_frame_for_shadow`, `game_tick` | a second add for one caster returns 0 |
| +0x4 | `shadow` | H | `game_add_frame_for_shadow`, `game_del_frame_for_shadow` | `game_bind_fake_shadow_buffer`, `game_del_frame_for_shadow`, `game_done`, `game_init`, `game_set_frame_for_shadow`, `game_tick` | driver type `0xf` |
| +0x8 | `stand_in` | H | `game_add_frame_for_shadow` | `game_add_frame_for_shadow`, `game_del_frame_for_shadow`, `game_init`, `game_tick` | vtable `+0x24` argument 1 on add and on deactivation, 0 on activation and on delete. A car passes `car+0x2118` `shadow_object` |
| +0xc | `active` | H | `game_add_frame_for_shadow`, `game_init`, `game_tick` | `game_init`, `game_set_frame_for_shadow`, `game_tick` | 0 on add |
| +0x10 | `name` | H | `game_add_frame_for_shadow`, `game_del_frame_for_shadow` | `game_bind_fake_shadow_buffer`, `game_del_frame_for_shadow`, `game_done`, `game_set_frame_for_shadow` | §4 for the recycled buffer |
| +0x14 | `buffer_index` | H | `game_add_frame_for_shadow`, `game_set_frame_for_shadow` | `game_bind_fake_shadow_buffer`, `game_set_frame_for_shadow` | the add refuses an index outside the table |
| +0x18 | `distance_sq` | H | `game_init`, `game_tick` | `game_fake_shadow_compare_by_distance` | §6 |

## 3. Method family

3 of the 7 PS2 methods of this feature have a PC function; `AddFakeShadow`, `DelFakeShadow`, `TickFakeShadows` and
`TickShadows` are in §8. The PC-only rows are the mesh table and the LOD pass's comparator.

### 3a. Construction & destruction

The feature has no constructor: `game_ctor` and `game_done_intern` build and drain the vectors (`structs/game.md`).

### 3b. Accessors

The feature has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005e8b70 | `game_add_fake_shadow_buffer` | `Init__6C_gameFbb` | 0x002ea0a0 | CONFIRMED | Creates a type `0xf` shadow, opens the path on it and appends it with its basename to the buffer table |
| 0x005e8da0 | `game_bind_fake_shadow_buffer` | - | - | - | Calls `SetTo` on the item's shadow with the mesh at `buffer_index`, or with the mesh whose basename matches `name` |
| 0x005e8e30 | `game_add_frame_for_shadow` | `AddFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_typeP9I3D_framePc` | 0x002f8ca0 | CONFIRMED | Refuses a bad index, a null frame and a duplicate; takes a name buffer, creates the shadow and appends the item |
| 0x005e91d0 | `game_set_frame_for_shadow` | `SetFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_type` | 0x002f8ec0 | STRUCTURAL | Stores a new `buffer_index`; a live item is detached, rebound and linked again |
| 0x005e9350 | `game_del_frame_for_shadow` | `DelFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type` | 0x002f8ed0 | STRUCTURAL | Detaches and releases the shadow, recycles the name buffer, moves the last item over this one |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005e8b40 | `game_fake_shadow_compare_by_distance` | - | - | - | `crt_qsort` comparator on `distance_sq`; only ever taken by address |

## 4. The `C_game` fields

The typedef and the evidence of these fields are in `structs/game.md`; this table says what each does here.

| Offset | `C_game` field | Role |
|---|---|---|
| `+0x280` | `shadow_meshes` | allocator byte; begin `+0x284`, end `+0x288`, cap `+0x28c`, stride 8. Filled 43 times by `game_add_fake_shadow_buffer`, drained by `game_done_intern` |
| `+0x290` | `shadow_items_a` | allocator byte; begin `+0x294`, stride `0x1c`. Selected when the `E_shadow_type` argument is 0 |
| `+0x2a0` | `shadow_items_b` | allocator byte; begin `+0x2a4`, stride `0x1c`. Selected for every other value |
| `+0x2b0` | `shadow_lod_timer` | the LOD accumulator in ms; `game_tick` adds the frame time |
| `+0x2b4` | `shadow_lod_period` | compared with the accumulator; 200 from `game_ctor`, 823 from `game_init` |
| `+0x2b8` | `shadow_lod_toggle` | flipped each pass, so the two item vectors are serviced alternately |
| `+0x3674` | `name_free_list` | allocator byte; begin `+0x3678`, end `+0x367c`. Recycled name buffers: popped by `game_add_frame_for_shadow` for a name under `0x1f` characters, pushed by `game_del_frame_for_shadow` |

The free list is a real allocation optimisation: a name is `operator new`'d only when the list is empty or the name is too
long, and the size taken is `max(0x20, len + 1)`.

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

## 6. Lifecycle

```
game_init          -> game_add_fake_shadow_buffer x43        (fills the buffer table)
<car spawns>       -> car_database_get_shadow_model
                   -> game_add_frame_for_shadow(...,name)    (adds an S_shadow_item)
game_tick, per LOD -> recompute distance_sq
                   -> crt_qsort with game_fake_shadow_compare_by_distance
                   -> nearest N: game_bind_fake_shadow_buffer + LinkTo, active = 1, stand_in off
                      the rest:  LinkTo(0), active = 0, stand_in on
<car despawns>     -> game_del_frame_for_shadow               (recycles the name buffer)
game_done_intern   -> destroy every buffer, operator delete every basename
```

**The LOD pass.** When the accumulator exceeds the period it subtracts the period, flips the toggle and walks one vector.
The pass adds the camera frame's forward vector scaled by 3.0 to its reference position and stores each caster's squared
distance from that point. One caster gets distance 0 and so sorts first: the player's frame in `shadow_items_a`, and the frame
of the object at the player's `+0x98` in `shadow_items_b`. N is `mission_settings_get_shadow_quality_idx()`, used as N in the
sorted list and nowhere else, so the shadow **count** is the graphics setting, not the shadow resolution. `game_init` runs the
same pass once, inline.

A car whose `carindex.def` shadow-model field names a mesh outside the 43 preloaded ones gets no shadow:
`game_bind_fake_shadow_buffer` binds nothing when no basename matches.

## 7. `I3D_shadow` — the engine class behind it

`I3D_shadow` lives in **`ls3df.dll`**, so `Game.exe` only ever holds pointers to it. It is created by the driver factory with
**object type `0xf`**. The slots below are the ones this subsystem calls; their names come from §8.

| Method | Vtable slot | Used by |
|---|---|---|
| `Open(const char *, u32)` | `+0x50` | `game_add_fake_shadow_buffer` |
| `SetTo(I3D_frame *)` | `+0x3c` | `game_bind_fake_shadow_buffer`, `game_set_frame_for_shadow` |
| `LinkTo(I3D_frame *, u32)` | `+0x2c` | the LOD pass, `game_set_frame_for_shadow`, `game_del_frame_for_shadow` |

## 8. PC vs PS2

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

**The LS3D API.** The PS2 compiled the engine in, which is where the method names of §7 come from. The rest of its
`I3D_shadow` API is `Duplicate`, `SetRange`, `SetColor`, `SetTextureSize`, `SetOrthogonal`, `SetMaxTextureDistance`,
`SetReverseFactor`, `SetShadowLock`, `DeleteFromAnyShadow`, `UpdateV` and `Close`, none of them reached from this subsystem.
Object type `0xf` is the same constant on both builds.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002f8c90 | `TickShadows__6C_gameFUi` | an empty 8-byte body; the PC's LOD pass is its own logic |
| 0x002f7c90 | `AddFakeShadow__6C_gameFP9I3D_frameP7C_actor` | the second shadow system, for people; the PC callers keep only `game_add_frame_for_shadow` |
| 0x002f81c0 | `DelFakeShadow__6C_gameFP9I3D_frame` | the same system; the PC callers keep only `game_del_frame_for_shadow` |
| 0x002f8330 | `TickFakeShadows__6C_gameFUi` | the same system's per-frame placement |

## 9. Related

`structs/game.md` (the owning singleton and its field map) · `structs/car_database.md` (`car_database_get_shadow_model`, the
`carindex.def` join) · `structs/car.md` (`car_set_shadow_enabled`, `shadow_object`) · `Game-ghidra.exe.functions.md` (the function
registry).

## 10. Open items

| Item | State |
|---|---|
| in game: shadow-quality setting | expected to reduce the NUMBER of actors casting a shadow (nearest-first), not the resolution (§6); not observed |
| in game: unmatched shadow-model name | a car whose `carindex.def` shadow model is not among the 43 meshes is expected to get no shadow (§6); not observed |
| `I3D_shadow` slots `+0x54`, caster `+0x18` | called on detach and on link; no name measured |
