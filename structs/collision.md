# `g_collision` — the global collision system (PC prefix `collision_*`)

**Created** 2026-07-26 13:53 +0200 · **Modified** 2026-10-12 11:21 +0200

PS2 class `g_collision` (mangled `11g_collision`) is the game's global collision service: a **singleton**
that owns the world collision tree (static geometry) plus a **uniform 2-D spatial grid of dynamic
collision objects** (`tDynamicCollObject`), and exposes the line / sphere / box / curve intersection
queries the rest of the engine calls. On PC the twins carry the `collision_*` prefix.

This doc documents the PC binary. PS2 offsets/names and cross-build differences live in §3 only; the
coverage table in §2 is the PC↔PS2 twin map (PC VA first).

| outcome | count | what it means |
|---|---|---|
| Mapped to a PC twin | **54** | 51 CONFIRMED + 3 STRUCTURAL |
| PS2-port-only | **11** | the code never existed in the PC build (§3) |
| **total** | **65** | 64 class methods + 1 free function |

**The population is a VA RANGE, not a class.** A free function has an **empty class column** in the PS2
symbol table, so any enumeration keyed on `class == g_collision` is structurally incapable of seeing
one. Enumerating the PS2 band `0x00276000..0x0029b000` **with the class column ignored** returns 65
symbols: the 64 class methods plus `GrantCoherence__FP15tCollisionLevel`, the twin of
`collision_classify_static_primitives`. Before quoting any "N/N accounted" figure for a class, ask
whether the enumeration was by CLASS or by VA RANGE — only the second can see a free function, a
file-local static, or a helper the linker placed in another cluster. Complete families: `Dyn*` (9/9),
line-test, **Box**, **Sphere**, the **Line counterparts**, the **collect** family, the character-proxy
pair, the full **lifecycle** chain (ctor / dtor / load / free / teardown) and every public thunk that PC
actually emits.

- 🔑 **The singleton is `g_collision` @ `0x0067A588`.** It is the hidden `__thiscall` **ECX receiver** at
  every callsite — `collision_dyn_insert(obj)` passes `obj` as the only visible argument. `.text`
  contains **514 `mov ecx,0x0067A588`** and **zero** `push 0x0067A588`, i.e. it is *only* ever a
  receiver, never a stack argument. Recovered by ECX back-trace at `collision_construct_singleton`,
  whose whole body is `mov ecx,0x0067A588; jmp collision_ctor`.
- **Four PC functions have no PS2 method and therefore no row in §2:** `collision_ctor` (`0x005800d0`,
  zeroes every field, frees nothing), `collision_construct_singleton` (`0x0057d980`, the 2-line wrapper
  that calls it) — the PS2 build exports no `g_collision` constructor — and the slide-solver pair of
  §2b.

## 1. The dynamic-collision spatial grid (PC field map)

Every confirmed `Dyn*` twin operates on the same sub-object of the `g_collision` singleton — a uniform 2-D
grid keyed by two integer cell coordinates. Offsets are **PC** (`this + N`), byte-proven across the seven
confirmed twins.

| PC offset | field | meaning |
|---|---|---|
| `+0x28c` | `grid` (ptr) | the grid object. `grid+0x10` = **cols** (width), `grid+0x14` = **rows**, `grid+0x18` = **cell array** (one `dword` slot per cell). |
| `+0x290` .. `+0x29c` | `free_list` vector | recycled cell-bucket nodes: allocator byte `+0x290`, begin `+0x294`, end `+0x298`, capacity `+0x29c`. `collision_dyn_optimize_tick` appends to it through `std_vector4_insert_n` with `lea ecx,[esi+0x290]`. |
| `+0x2a0` .. `+0x2ac` | `pending_empty` vector | cells that became empty and are awaiting recycling: allocator byte `+0x2a0`, begin `+0x2a4`, end `+0x2a8`, capacity `+0x2ac`; drained by `collision_dyn_optimize_tick`. |
| `+0x2b0` | `next_owner_callback` | per-hit result callback (`gcCallbackRet`) for the next dynamic query. |
| `+0x2b4` | `next_owner_single` | single-owner storage slot. |
| `+0x2b8` | `next_owner_array` (ptr) | owner-array pointer (points at `+0x2b4` in the single case, or the caller's array in the mult case). |
| `+0x2bc` | `next_owner_count` | owner count (1 for single, N for mult, 0 = none). |

**Cell index formula (identical on both builds):** a cell at (col, row) is
`*(grid+0x18 + (cols*row + col) * 4)`, where `cols = *(grid+0x10)`. Each non-zero slot points at a bucket
`{..., +0xc: begin, +0x10: end}` — a vector of the `tDynamicCollObject*` in that cell. The bucket is a
0x18-byte record `collision_dyn_insert` allocates: the cell's (col, row) in its first two words, an
allocator byte at `+0x08`, and the vector at `+0x0c..+0x14`. A `tDynamicCollObject` keeps its type in
word 0 (§1d) and the back-link to its bucket at `+0xc`.

### 1a. How the `Dyn*` family fits together

- **`collision_dyn_insert`** — register one object: compute its cell, create/reuse the bucket, link in
  (gated by `obj+0xc == 0` = not-already-registered).
- **`collision_dyn_update`** — after the object moves, re-bucket it: remove from the old cell, insert into
  the new; no-op if the cell is unchanged.
- **`collision_dyn_erase`** — unregister one object (does not free it).
- **`collision_dyn_optimize_tick`** — per-tick housekeeping: drain the `pending_empty` list, zero the empty
  grid slots, recycle their buckets onto the free-list.
- **`collision_dyn_clear_all`** — bulk teardown: sweep the whole grid, empty every bucket, recycle all cells.
- **`collision_dyn_set_next_owner` / `collision_dyn_set_next_multiple_owners`** — install the object(s) that
  "own" the next dynamic query (self-exclusion + a hit callback); single vs. array-of-N form.
- **`collision_dyn_take_out_to_list` / `collision_dyn_put_back_from_list`** — temporarily lift an object out
  of the grid into a caller-held vector, and put it back. Used to exclude a moving object from its own
  queries.

### 1b. The query parameter structs (PC field map)

Every query family passes one parameter struct. These offsets are **PC**, byte-proven on the Box and
Sphere workers; they are what separates two workers whose bodies are otherwise identical
(`collision_box_in_tree_level` vs `collision_sphere_in_tree_level` differ *only* in which of these they read).

| struct | PC offset | field | proven by |
|---|---|---|---|
| `tCBoxParam` | `+0x00` / `+0x0c` | box min corner / max corner (the AABB tested) | `collision_box_in_tree_level`, `collision_box_in_primitive_volume` |
| `tCBoxParam` | `+0x08`, `+0x10`, `+0x14` | remaining corner components used by the 6-way overlap test | `collision_box_in_primitive_volume` |
| `tCSphereParam` | `+0x00` | sphere centre (sweep start) | `collision_sphere_vs_triangle`, `collision_sphere_in_dynamics` |
| `tCSphereParam` | `+0x0c` | sweep delta (end = centre + delta) | `collision_sphere_vs_aabb` |
| `tCSphereParam` | `+0x18` | **sphere radius** | `collision_sphere_vs_triangle`, `_vs_edge`, `_vs_aabb` (three independent witnesses) |
| `tCSphereParam` | `+0x1c` / `+0x24` | query bounds used for tree-level rejection | `collision_sphere_in_tree_level` |
| `tCSphereParam` | `+0x38` | sweep direction | `collision_sphere_vs_triangle`, `collision_sphere_vs_edge` |
| `tCSphereParam` | `+0x44` / `+0x48` | layer/flag masks (tree-level vs vehicle paths read different words of this band) | `collision_sphere_in_tree_level`, `collision_sphere_in_vehicle` |
| `tCSphereParam` | `+0x50` | current best-hit squared time — a candidate must beat it | `collision_sphere_vs_edge` |

**Object kind dispatch.** Every dynamic-object query switches on the low byte of the object's first word.
The dynamic-grid workers write it as `switch((*obj & 0xff) - 0x40)`, so PC `case 0` is kind `0x40`.
Static volume primitives (`g_volume_triface`) use a separate range dispatched as
`switch((*vol & 0xff) - 0x80)` over kinds `0x80..0x84`, with `0x80`/`0x81` sharing one branch.

### 1c. The naming grammar — how to read a query function's name

Names are `collision_<query shape>_<relation>_<target>`. **The relation word carries real information and
the two are not interchangeable:**

| relation | meaning | target is… |
|---|---|---|
| **`_in_`** | **search a CONTAINER** — walk a set of candidates and return the first hit | `dynamics`, `vehicle`, `tree_level`, `primitive_volume` |
| **`_vs_`** | **leaf math against ONE primitive** — no iteration, just the intersection test | `aabb`, `edge`, `triangle` |
| **`collect_…_in_sphere`** | **gather ALL hits** into a caller-supplied out-vector instead of returning the first | the collect family |

> **Why the distinction exists.** Without it, "box" sits on both sides of `_in_` with opposite meanings:
> `collision_box_in_tree_level` (box = the **query** being swept) versus a sphere-against-a-box leaf test
> (box = the **target** being tested), and a reader cannot tell which role a shape plays. `_vs_` marks the
> leaf tests, so the query shape is always the word before the relation and the target is always the word
> after.

| target | what it is | tell in the body |
|---|---|---|
| `_in_dynamics` | the dynamic grid | derefs `this+0x28c`, single struct arg |
| `_in_vehicle` | a caller-supplied `std::vector<tDynamicCollObject*>` | walks `param+0x04`/`+0x08`, **never** touches `+0x28c` |
| `_in_tree_level` | one level of the static tree | count at `lvl+0x00`, pointer array at `lvl+0x04`, `&0xa0` skip |
| `_in_primitive_volume` | the non-triangle primitives of a `g_volume_triface` | `switch((*vol & 0xff) - 0x80)` over kinds `0x80..0x84` |
| `_vs_aabb` | one axis-aligned box given by min/max corners | per-axis slab test; the sphere form inflates by the radius first |
| `_vs_edge` | one line segment | quadratic solve + clamp to the segment |
| `_vs_triangle` | one triangle of a `g_volume_triface` | 3 vertex pointers + plane normal/distance |
| `collision_test_*` | a **public entry point** that packs a param struct and delegates to a worker | short; calls exactly one worker |
| `*_thunk` | a **one-line** public entry that only binds the singleton receiver | 2 lines, forwards everything |

**Terminology note.** *AABB* = axis-aligned bounding box. It is what distinguishes `collision_line_vs_aabb` (no orientation anywhere in the body) from
`collision_test_moving_box`, which takes two transform matrices because its box rotates.

### 1d. The dynamic-object type byte

Every object in the grid carries its type in the low byte of word 0 (`*obj & 0xff`).
`collision_line_in_dynamics`, `collision_line_in_dynamics_inner` and `collision_sphere_in_dynamics`
switch on it, and their case labels together close the PC set at eight values; any other value,
`0x45` included, falls through to the next object (`collision_box_in_dynamics` handles `0x40`,
`0x41`, `0x42` and `0x44`). The two line dispatchers share the `0x40` / `0x41` / `0x42` / `0x44`
arms; only the inner one handles `0x47` and only the outer one `0x48`.

| type | line query arm | sphere query arm | inserted by |
|---|---|---|---|
| `0x40` | transforms the ray into the object's frame and tests the box at `+0x24` / `+0x30` (`collision_line_vs_aabb`); the hit normal is rotated back | the same with `collision_sphere_vs_aabb` | `airplane`, `box`, `detector`, `dog`, `door`, `physical_object`, `pumper`, `rail_passenger`, `railway`, `spike_strip`, `vehicle`, `wagon` |
| `0x41` | tests the box at `+0x24` / `+0x30` as stored (no transform) | the same with `collision_sphere_vs_aabb` | - |
| `0x42` | sphere: centre `+0x10`, radius `+0x1c` | sphere against sphere | `airplane`, `bottle`, `dog`, `human_create_dyn_colls`, `pumper`, `railway_add_person`, `railway_switch_game_init` |
| `0x43` | none (skipped) | its own arm, built on the horizontal (x, z) components of the query direction | - |
| `0x44` | shares the `0x42` arm | its own arm, which also takes the dot product of the query direction with the centre offset | - |
| `0x46` | none (skipped) | like `0x40`, with the frame's uniform scale applied | - |
| `0x47` | `bendable_plant_push_element` when the owner at `+0x44` is kind `0x1d`; **never reports a hit** | the same, additionally gated on singleton `+0x2c0`, which only the first sphere of `collision_test_character_proxy` sets (§4) | `bendable_plant_game_init` |
| `0x48` | when `+0x2c4` (`g_collision_next_query_actor`) is set, calls that actor's vtable `+0x90` (cbCollTester) with the object; **never reports a hit** | none (skipped) | `game_create_track_effect` (blood decals, `structs/blood.md` §5a) |

The last column lists the functions that store the value as an immediate and then call
`collision_dyn_insert` (a census of the 51 insert callsites). `-` = no PC code creates an object of
that type, so the arm is unreachable on the PC: a census of every `mov [mem], imm` in `.text` finds
`0x41`, `0x43`, `0x44` and `0x46` only in six functions that build no collision object (the human
animation state at `+0x74`, network-message and script-decoder stack locals, a racing gate field
and the crash reporter's map reader), while the same census returns every inserter listed for the
other four types.
Pedestrians register a type-`0x40` object too: `pedestrian_set_model` and
`pedestrian_create_model` store the type into the object embedded in the pedestrian record, and
`pedestrian_activate_free` and `pedestrian_set_anim` insert it (`structs/pedestrian.md` §2). The
store and the insert sit in different functions, so the column does not list them.
`0x47` and `0x48` are the types whose arms never set the hit
result: the `structs/bendable_plant.md` clones react to a query and never block it, and a blood
decal's sphere only tells the querying actor that it stepped into blood, which arms its bloody
prints (`structs/blood.md` §5e).

## 2. Method coverage — PC ↔ PS2 (65 PS2 symbols)

Row set taken from the PS2 symbol table over the VA band, class column ignored. `not located` = no PC
twin has been body-diffed.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role / notes |
|---|---|---|---|---|---|
| 0x00470880 | `collision_earliest_contact_between_spheres` | `TestSphereSphere__11g_collisionFRC8S_vectorRC8S_vectorfRC8S_vectorRC8S_vectorf` | 0x0015ff20 | CONFIRMED | earliest contact time between two MOVING spheres (pos,vel,radius x2 -> float); lives at 0x0047xxxx, far from the class cluster |
| 0x0057d9b0 | `collision_dtor` | `__dt__11g_collisionFv` | 0x0027a4d0 | CONFIRMED | free the pending-empty vector +0x2a4 then the free-list +0x294, nulling both triples - same order as PS2 |
| 0x0057da00 | `collision_free_level_data` | `DeleteCollsLevel__11g_collisionFb` | 0x002768e0 | CONFIRMED | free static grid +0x284 and dynamic grid +0x28c, null both, rewind the free-list and pending-empty vectors |
| 0x0057dab0 | `collision_classify_static_primitives` | `GrantCoherence__FP15tCollisionLevel` | 0x00276a00 | CONFIRMED | post-load pass over the static level: classify every primitive by dominant normal axis + degeneracy into codes 1..7. FREE function on PS2 (empty class column) - invisible to a class census. Its param type names the PC structure: g_collision+0x284 is a tCollisionLevel. Epsilon 0.0001 PC vs 0.0002 PS2 |
| 0x0057dca0 | `collision_load_level` | `LoadCollision__11g_collisionFPCcb` | 0x00276e10 | CONFIRMED | load the static collision tree from a .dta: dtaOpen + free the old level, then ALLOCATE/assign the static grid at this+0x284 |
| 0x0057e740 | `collision_dyn_optimize_tick` | `DynOptimizeTick__11g_collisionFv` | 0x00277eb0 | CONFIRMED | per-tick housekeeping: drain the pending-empty vector +0x2a0, recycle to the free-list vector +0x290 |
| 0x0057e7f0 | `collision_dyn_insert` | `DynInsert__11g_collisionFP18tDynamicCollObject` | 0x00278150 | CONFIRMED | register ONE tDynamicCollObject into the grid at this+0x28c (entity+0xc==0 gate) |
| 0x0057e9e0 | `collision_dyn_erase` | `DynErase__11g_collisionFP18tDynamicCollObject` | 0x00278340 | CONFIRMED | unregister ONE tDynamicCollObject (does not free it) |
| 0x0057ebf0 | `collision_dyn_update` | `DynUpdate__11g_collisionFP18tDynamicCollObject` | 0x002784c0 | CONFIRMED | re-bucket an object to its current cell after movement |
| 0x0057ee40 | `collision_dyn_set_next_owner` | `DynSetNextCollOwner__11g_collisionFP18tDynamicCollObjectPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet` | 0x002787c0 | CONFIRMED | register the SINGLE owner for the next dynamic query + callback (owner->+0x2b4) |
| 0x0057ee90 | `collision_dyn_set_next_multiple_owners` | `DynSetNextCollOwnerMult__11g_collisionFPP18tDynamicCollObjectUiPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet` | 0x00278870 | CONFIRMED | register an ARRAY of owners for the next dynamic query + callback (array->+0x2b8, count->+0x2bc) |
| 0x0057eed0 | `collision_dyn_clear_all` | `DynWipeOut__11g_collisionFv` | 0x00278910 | CONFIRMED | remove ALL dynamic objects: sweep whole grid, empty every bucket, recycle cells |
| 0x0057f150 | `collision_dyn_take_out_to_list` | `DynGetIn__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>` | 0x00278db0 | CONFIRMED | erase from the grid if registered (obj+0xc), then append the object to the caller's vector |
| 0x0057f300 | `collision_dyn_put_back_from_list` | `DynGetOut__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>` | 0x00278e10 | CONFIRMED | find-and-erase the object from the caller's vector, then re-insert it into the grid |
| 0x0057f360 | `collision_box_in_dynamics` | `BoxInDynamic__11g_collisionFR10tCBoxParam` | 0x00278e90 | CONFIRMED | box vs DYNAMIC objects: +0x28c cell-rect sweep, owner-exclusion +0x2bc/+0x2b8, kind switch (*obj&0xff)-0x40 |
| 0x0057f5e0 | `collision_box_in_primitive_volume` | `BoxInNonFace__11g_collisionFP16g_volume_trifaceR10tCBoxParam` | 0x00279220 | CONFIRMED | box vs ONE non-triangle primitive of a g_volume_triface; switch((*vol&0xff)-0x80) over kinds 0x80..0x84 |
| 0x0057f7f0 | `collision_box_in_tree_level` | `BoxInTreeLevel__11g_collisionFP14tCollTreeLevelR10tCBoxParam` | 0x00279510 | CONFIRMED | box vs one static-tree level (count lvl+0x00, array lvl+0x04, &0xa0 skip); establishes tCBoxParam PC+0x00 <-> PS2+0x50 |
| 0x0057f9e0 | `collision_test_moving_box` | `_TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi` | 0x00279960 | CONFIRMED | swept test for a MOVING oriented box: I3D_bbox + prev/current S_matrix + 3 vectors + 2 uints (exact 9/9 params) |
| 0x00580170 | `collision_line_in_dynamics` | `LineInDynamics__11g_collisionFR11tCLineParam` | 0x0027a580 | CONFIRMED | line vs dynamic objects (+0x28c cell sweep, narrow-phase collision_line_vs_aabb); called by _TestLineH |
| 0x005806b0 | `collision_line_in_dynamics_inner` | `LineInDynamicsInnerDyn__11g_collisionFR11tCLineParam` | 0x0027ada0 | STRUCTURAL | inner-dynamic variant (mask +0x3c, builds S_matrix); PS2 body not line-read |
| 0x00580c50 | `collision_line_vs_aabb` | `LineInAABox__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam` | 0x0027b620 | CONFIRMED | leaf math: line vs ONE axis-aligned box, per-axis slab test picking the NEAR plane; PS2 LineInDynamics calls it x2, PC likewise |
| 0x00580ff0 | `collision_line_vs_aabb_from_inside` | `LineInAABoxInverted__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam` | 0x0027bc20 | CONFIRMED | same slab math but picks the FAR plane = the ray-starts-inside exit test; normal sign flipped |
| 0x00581210 | `collision_line_in_primitive_volume` | `LineInNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam` | 0x0027c380 | CONFIRMED | line vs ONE non-triangle primitive; switch((*vol&0xff)-0x80) kinds 0x80..0x84 - completes the *InNonface trio |
| 0x005817d0 | `collision_line_in_tree_level` | `LineInTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam` | 0x0027ccf0 | CONFIRMED | line vs one static-tree level (count lvl+0x00, array lvl+0x04, &0xa0 skip) - completes the tree-level trio |
| 0x00581c40 | `collision_test_line_vertical` | `_TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027d460 | CONFIRMED | vertical variant of the line test; the horizontal DDA's near-vertical degenerate case |
| 0x00582120 | `collision_test_line_vertical_inner_dyn` | `_TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027d920 | CONFIRMED | inner-dynamic vertical variant |
| 0x00582600 | `collision_test_line_horizontal_static` | `TestLineHStatic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi` | 0x0027dde0 | CONFIRMED | static-only (+0x284) horizontal line test (5-arg) |
| 0x00583570 | `collision_test_line_horizontal_dynamic` | `TestLineHDynamic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027f010 | CONFIRMED | dynamic-only (+0x28c) horizontal line test |
| 0x005838a0 | `collision_test_line_horizontal` | `_TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027f300 | CONFIRMED | HORIZONTAL line/ray test vs BOTH grids (in-game confirmed); delegates dynamic to collision_line_in_dynamics |
| 0x005848e0 | `collision_line_in_vehicle` | `LineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R11tCLineParam` | 0x00280660 | CONFIRMED | line vs a caller-supplied vector<tDynamicCollObject*>; S_matrix per part; never touches +0x28c |
| 0x00584cb0 | `collision_test_line_in_vehicle` | `TestLineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi` | 0x00280c10 | CONFIRMED | public wrapper over collision_line_in_vehicle; calls the worker exactly ONCE on both builds |
| 0x00584fa0 | `collision_collect_primitive_volume_in_sphere` | `LocalityInNonface__11g_collisionFP16g_volume_trifaceRC8S_vectorRC8S_vectorRC8S_vectorfRQ23gif31vector<P18g_collision_header,8>` | 0x00280e90 | CONFIRMED | COLLECT every non-triangle primitive inside a spherical region into the caller's out-vector |
| 0x005854a0 | `collision_collect_tree_level_in_sphere` | `LocalityInTreeLevel__11g_collisionFP14tCollTreeLevelRC8S_vectorRC8S_vectorRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8>` | 0x00281930 | CONFIRMED | descend one static-tree level collecting into the out-vector; the two Ui args are an inclusive kind-range filter |
| 0x00585510 | `collision_collect_static_hits_in_sphere` | `FindStaticCollsInLocality__11g_collisionFRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8>` | 0x00281a20 | CONFIRMED | public entry of the collect family: clear the out-vector, then gather every static primitive within a radius (this+0x284) |
| 0x005856c0 | `collision_find_hits_for_frame` | `FindCollsByFrame__11g_collisionFP9I3D_frameRQ23gif31vector<P18g_collision_header,8>` | 0x00281cb0 | CONFIRMED | collect every static primitive attached to an I3D_frame; inner offsets grid+0x38/+0x3c pin static grid PS2+0x3e4 <-> PC+0x284 |
| 0x00585890 | `collision_find_dynamic_line_hits` | `FindDynamicCollsInLine__11g_collisionFRC8S_vectorRC8S_vectorRQ23gif31vector<P18g_collision_header,8>Ui` | 0x002832e0 | STRUCTURAL | collect ALL dynamic line hits into out-vector param_4, return count; PS2 body not line-read |
| 0x005860b0 | `collision_sphere_in_dynamics` | `SphereInDynamics__11g_collisionFR13tCSphereParam` | 0x00284040 | CONFIRMED | sphere vs DYNAMIC objects: single-struct-arg + this+0x28c, the *InDynamics worker signature |
| 0x00586fd0 | `collision_sphere_vs_edge` | `SphereInEdge__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParamRUiUiUi` | 0x002850a0 | CONFIRMED | swept-sphere vs line SEGMENT: quadratic solve, clamp to segment, must beat best-hit at param+0x50 |
| 0x00587570 | `collision_sphere_vs_triangle` | `SphereInFace__11g_collisionFP16g_volume_trifaceR13tCSphereParamRi` | 0x002856a0 | CONFIRMED | swept-sphere vs ONE triangle (3 vertex ptrs + plane); establishes tCSphereParam radius +0x18 / sweep dir +0x38 |
| 0x00588140 | `collision_sphere_vs_aabb` | `SphereInAABox__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParam` | 0x00286600 | CONFIRMED | swept-sphere vs AABB: inflate min-r/max+r (Minkowski) then per-axis slab reject |
| 0x0058c6a0 | `collision_sphere_in_primitive_volume` | `SphereInNonface__11g_collisionFP16g_volume_trifaceR13tCSphereParam` | 0x0028c530 | CONFIRMED | swept-sphere vs ONE non-triangle primitive; switch(*vol&0xff) over kinds 0x80..0x83 |
| 0x0058e970 | `collision_sphere_in_tree_level` | `SphereInTreeLevel__11g_collisionFP14tCollTreeLevelR13tCSphereParam` | 0x0028ec90 | CONFIRMED | sphere vs one static-tree level; separated from the Box sibling ONLY by param offsets (+0x1c/+0x24 vs +0x00/+0x0c) |
| 0x0058eaa0 | `collision_test_sphere_swept` | `_TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x0028ef90 | CONFIRMED | the swept-sphere WORKER: cast a sphere of radius param_4, report hit pos/normal (exact 8/8 params) |
| 0x0058fb00 | `collision_earliest_contact_between_paths` | `TestCurveCurve__11g_collisionFPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>fPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>f` | 0x0028fa50 | CONFIRMED | earliest contact time between two moving spheres on sampled gif::vector<gcol_CurveSphereStruct> trajectories; FLT_MAX if none |
| 0x0058fd30 | `collision_sphere_in_vehicle` | `SphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R13tCSphereParam` | 0x0028ff10 | CONFIRMED | sphere vs a caller-supplied vector<tDynamicCollObject*>; never touches +0x28c - the *InVehicle discriminator |
| 0x00590590 | `collision_test_sphere_in_vehicle` | `TestSphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x002909d0 | CONFIRMED | public wrapper: packs a tCSphereParam and delegates to collision_sphere_in_vehicle |
| 0x00590a00 | `collision_teardown` | `GameDone__11g_collisionFb` | 0x00299920 | CONFIRMED | shutdown: collision_dyn_clear_all() then collision_free_level_data() (PS2: DynWipeOut(); DeleteCollsLevel();) |
| 0x00590a10 | `collision_test_line_vertical_thunk` | `TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299960 | CONFIRMED | one-line public entry binding the singleton, forwards to collision_test_line_vertical |
| 0x00590a20 | `collision_test_line_vertical_inner_dyn_thunk` | `TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299970 | CONFIRMED | one-line public entry over collision_test_line_vertical_inner_dyn |
| 0x00590a30 | `thunk_collision_test_line_horizontal` | `TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299980 | CONFIRMED | the public forwarding thunk: a real 5-byte `e9` jmp at its own VA, target 0x005838a0 (§3) |
| 0x00590a40 | `collision_test_sphere_swept_thunk` | `TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x002999a0 | CONFIRMED | one-line public entry over collision_test_sphere_swept; 44 callsites pass its 7 args |
| 0x00590ab0 | `thunk_collision_test_moving_box` | `TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi` | 0x002999b0 | STRUCTURAL | the public forwarding thunk: a real 5-byte `e9` jmp at its own VA, target 0x0057f9e0 (§3) |
| 0x00590ac0 | `collision_test_character_proxy` | `TestPanak2__11g_collisionFP9tgcPanak2` | 0x002999d0 | CONFIRMED | sweep a THREE-SPHERE person proxy (tgcPanak2, Czech panak = dummy/figure) via collision_test_sphere_swept x3; consumer human_test_move_coll |
| 0x00591530 | `collision_test_character_proxy_in_vehicle` | `TestPanak2InVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>P9tgcPanak2` | 0x0029a320 | CONFIRMED | same proxy against a vehicle part list via collision_test_sphere_in_vehicle x3; consumer human_test_move_coll_in_owner |
| - | - | `PreallocPrepare__11g_collisionFPCcPCc` | 0x00276610 | PS2-ONLY | its ONLY PS2 caller is InterCity_Prebuffering__9C_mission - the PS2-only city-streaming subsystem |
| - | - | `LineInAACapsule__11g_collisionFffR11tCLineParam` | 0x0027bf40 | PS2-ONLY | five PS2 callsites, all under kinds no PC switch has: dynamic kind 0x4a in LineInDynamics, LineInDynamicsInnerDyn, LineInVehicle and FindDynamicCollsInLine, static primitive kind 0x87 in LineInNonface |
| - | - | `_TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi` | 0x00282340 | PS2-ONLY | the worker of the S-line chain: a pure yes/no line test (no out-params), §3 |
| - | - | `LineInSTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam` | 0x00282630 | PS2-ONLY | static-tree level walker of the S-line chain, §3 |
| - | - | `LineInSNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam` | 0x00282c40 | PS2-ONLY | primitive test of the S-line chain, §3 |
| - | - | `SphereInAACapsule__11g_collisionFffR13tCSphereParam` | 0x0028bc00 | PS2-ONLY | three PS2 callsites, all under kinds no PC switch has: dynamic kind 0x4a in SphereInDynamics and SphereInVehicle, static primitive kind 0x87 in SphereInNonface |
| - | - | `TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi` | 0x00299990 | PS2-ONLY | the public entry of the S-line chain, §3 |
| - | - | `LineInConvexDynamics__11g_collisionFR11tCLineParamP24tDynamicCollObjectConvex` | 0x004896c0 | PS2-ONLY | PC kind switch lacks 0x49/0x4a; 0 'convex' in the PC decomp and 0 `convex` strings in Game.exe |
| - | - | `SphereInConvexDynamics__11g_collisionFR13tCSphereParamP24tDynamicCollObjectConvex` | 0x00489a10 | PS2-ONLY | PC kind switch lacks 0x49/0x4a; no convex primitive exists in the PC build |
| - | - | `LineInConvexStatic__11g_collisionFR11tCLineParamP15g_volume_convex` | 0x00489e00 | PS2-ONLY | no convex primitive in the PC build |
| - | - | `SphereInConvexStatic__11g_collisionFR13tCSphereParamP15g_volume_convex` | 0x0048a150 | PS2-ONLY | no convex primitive in the PC build |

### 2a. The API at a glance (from the mangled signatures)

- **Query families** (the bulk): `Line*` (ray/segment), `Sphere*` (swept sphere), `Box*` (AABB), plus
  `TestCurveCurve`, `TestPanak2` (a `tgcPanak2` shape) and the `*Convex` pair. Each family splits into
  variants by target: `*InDynamics`/`*InVehicle` (dynamic grid), `*Static`/`*Nonface`/`*TreeLevel`
  (static world tree), `*InAABox`/`*InAACapsule` (primitive tests). The bare `TestLine*`/`TestSphere*`
  at `0x00299xxx` are tiny (8–12 bytes) — the public thunks over the `_Test*` workers.
- **Locality / find:** `FindStaticCollsInLocality`, `FindCollsByFrame`, `FindDynamicCollsInLine`,
  `Locality*` — collect candidate primitives into a `gif::vector<g_collision_header*>`.
- **Lifecycle:** `LoadCollision`, `PreallocPrepare`, `DeleteCollsLevel`, `GameDone`, `{dtor}`.
- **Element/param types:** `tDynamicCollObject`,
  `tDynamicCollObjectConvex`, `tCLineParam`, `tCSphereParam`, `tCBoxParam`, `g_volume_triface`,
  `g_volume_convex`, `tCollTreeLevel`, `g_collision_header`, `tgcPanak2`, `gcol_CurveSphereStruct`.

### 2b. The collide-and-slide solver and the contact array (PC-only)

Three things at the front of the singleton: a 20-slot contact array, its count, and the two functions
that fill and publish it.

**The contact record — `0x20` bytes, 20 slots, at `g_collision + 0x00`:**

| offset | field | meaning |
|---|---|---|
| `+0x00` | `S_vector plane_normal` | the contact plane's normal (also returned as the solver's out-normal) |
| `+0x0c` | `float plane_d` | the plane constant, `-dot(normal, impact_point)` |
| `+0x10` | `S_vector contact_point` | the sphere centre at impact |
| `+0x1c` | `tDynamicCollObject *hit_obj` | the hit object, but **only** when its kind byte has bit `0x40` (i.e. a dynamic object); `0` for a static hit |

`g_collision_contact_count` (`+0x280`) holds how many are live. Capacity is `0x280 / 0x20` =
**20**. Neither the array nor the count is touched by `collision_ctor` (§4).

**`collision_slide_sphere` (`0x0058f670`)** is the classic collide-and-slide loop: lift the
start by `+0.01` in Y and drop the end by `-0.02`, then iterate at most **ten** times —
`collision_test_sphere_swept` along the remaining motion, and on a hit build the contact
plane, project what is left of the motion onto it, and re-clip that against every plane
already collected this call, each re-clip damped by `0.95`. One record is appended per
iteration.

**`collision_test_sphere_slide` (`0x00590a60`)** is the public entry (the `collision_test_*`
shape of §1c): it runs the solver and hands the caller `(&g_collision, contact_count)` — the
array base is the singleton's own address, which is why the count appears in the decomp as a
standalone `DAT_` while the array does not.

**Both are PC-only, and that is a measurement over a closed set** rather than a failed
search: the PS2 collision band enumerated with the class column ignored holds 64 class
methods + 1 free function, none of them a slide solver. The sole consumer chain ends in
`box_ai_tick`, a method of `C_box` — itself PC-only (`structs/box.md`).

### 2c. Translation units of the PC cluster

From the MSVC `.CRT$XCU` dynamic-initializer table at `.data:0x00646190..0x006461bc`, which
lists every TU initializer in link order, paired with the code following each:

| PC range | content | PS2 source file |
|---|---|---|
| `[0x0057d970, 0x00580140)` | singleton lifecycle, level load/free, `Dyn*` grid, `Box*` | `g_c_glob.cpp` |
| `[0x00580140, 0x00584f70)` | the `Line*` family | `g_c_line.cpp` |
| `[0x00584f70, 0x00586080)` | the `Locality*` / `Find*Colls*` collect family | `g_c_linebs.cpp` |
| `[0x00586080, 0x005909d0)` | the `Sphere*` family + the slide solver | `g_c_sph.cpp` |
| `[0x005909d0, 0x00591f20)` | teardown, the five public thunks, `TestPanak2` | `g_colls.cpp` |

The PS2 filenames come from its `__sinit_<file>.cpp` manifest, which carries six collision files — the
five above plus the PS2-only `g_c_convex.cpp`. Each PC TU is matched to its file by link order, which
agrees on both builds (§3). The PC scaffolding is named after the PC TU's content
(`crt_static_init_collision_line`, `..._collect`, `..._sphere`, `..._api`) so that a reader
needs no cross-build knowledge to use it.

Each TU also emits an `atexit(...)` one-liner. They are the cluster's OWN scaffolding, not foreign code
— `crt_atexit_register_collision` (`0x0057d990`) is `g_c_glob.cpp`'s registrar and the only one in the
whole range that registers a real destructor instead of the shared no-op; the others are
`crt_static_init_collision_line` / `crt_atexit_register_collision_line` (`0x00580140` / `0x00580160`),
`..._collect` (`0x00584f70` / `0x00584f90`) and `..._sphere` (`0x00586080` / `0x005860a0`). **Subtract
all of them before reading a rank-order comparison against the PS2 symbol order**, since they shift
every rank below them.

## 3. PC vs PS2 — cross-build notes

This is the one section carrying PS2 offsets and behaviour.

- **Field rebase.** The PC grid-owner fields sit ~`0x158`–`0x160` below their PS2 counterparts, and the
  delta is **not** a single constant across the whole object: the grid pointer maps PS2 `+0x3ec` ↔ PC
  `+0x28c` (Δ `0x160`), while the pending/free/owner fields map at Δ `0x158`
  (PS2 pending `+0x3fc/+0x400` ↔ PC `+0x2a4/+0x2a8`; PS2 owner group `+0x408/+0x40c/+0x410/+0x414` ↔ PC
  `+0x2b0/+0x2b4/+0x2b8/+0x2bc`). Match by **structure and the cell-index formula**, never by a fixed delta.
- **The `SetNextCollOwner` bit-8 loops are PS2-only in the decompiled PC bodies.** On PS2 both
  `DynSetNextCollOwner`/`Mult` walk the current owner array clearing flag bit `0x08` (`&0xfffffff7`) on each
  owner's `+8`, install the new owner(s), then set bit `0x08` (`|8`) on the new set. `DynWipeOut` has a
  matching per-owner bit-8 pre-loop. The PC twins install the owner fields but **omit those loops** — a
  build/factoring divergence (§6). The setter structure (owner ptr / count / callback) is byte-exact, which is what makes the twin
  CONFIRMED.
- **PS2 names of the §1 vocabulary.** `collision_dyn_take_out_to_list` /
  `collision_dyn_put_back_from_list` are PS2 `DynGetIn` / `DynGetOut`, which read **backwards**
  relative to the behaviour: "GetIn" means get the object into the caller's *list* (i.e. **out** of
  collision). The collect family is PS2 `Locality*` / `Find*`; the targets `_in_primitive_volume`,
  `_vs_aabb` and `_vs_triangle` are PS2 `NonFace`, `AABox` (Illusion's term for an AABB) and `Face`.
- **The grid-cell bucket is 4 bytes larger on PC** — `operator_new(0x18)` vs PS2 `__nw__(0x14)`, both in
  `DynInsert`: the PS2 bucket is `{col, row, vector}` with the vector at `+0x08`, the PC one puts an
  allocator byte there and the vector at `+0x0c`.
- **The TU file names of §2c come from link order.** The PS2 links its game files in alphabetical
  order: the `__sinit_*` list and the code agree from `C_wagon.cpp` → `console.cpp` →
  `g_c_glob.cpp` through `g_dabing.cpp` → `g_indicators.cpp` → `g_m_carc.cpp` → `g_menu.cpp` →
  `game.cpp`. In the PS2 code, `G_Camera` (`g_camera.cpp`) fills `[0x00290d50, 0x00299920)` after
  the `Sphere*` band; `GameDone`, the public `Test*` thunks, `TestPanak2` and `TestPanak2InVehicle`
  follow at `0x00299920`, and `Dabing*` (`g_dabing.cpp`) at `0x0029ac40`. Since
  `g_camera < g_colls < g_dabing`, that block is `g_colls.cpp`. The `Locality*` / `Find*` block
  `[0x00280e90, 0x00284040)` sits between `g_c_line.cpp` and `g_c_sph.cpp`, so it is
  `g_c_linebs.cpp`. The PC TUs follow the same order: line, collect, sphere, then the public API.
- **Emission order matches within the cluster.** The confirmed PC `Dyn*` cluster `0x0057e740…0x0057eed0`
  is 1:1 with PS2 `0x00277eb0…0x00278910` (`OptimizeTick, Insert, Erase, Update, SetNextCollOwner,
  SetNextCollOwnerMult, WipeOut`) — an emission-order lever.
- **Parameter-struct rebase.** `tCBoxParam` maps PC `+0x00` ↔ PS2 `+0x50` (Δ `0x50`),
  proven on `BoxInTreeLevel` and re-confirmed on `BoxInNonFace`. This is the discriminator that separates
  a Box worker from its identically-shaped Sphere sibling — the Sphere struct's fields sit at different
  offsets entirely (see §1b for the PC map).
- **`gcol_CurveSphereStruct` is 32 bytes on PS2 but 16 on PC** — PS2 pads each `{x, y, z, t}` sample to a
  VU quadword pair, putting the time field at `+0x10` where PC has it at `+0x0c`. The element-count guard
  therefore reads `>> 5` on PS2 and `>> 4` on PC while meaning the same thing (at least two samples).
- **`TestCurveCurve` factoring differs.** PS2 inlines the two-moving-spheres contact solver; PC calls it
  out of line. Size ratio 2.16 is explained entirely by that helper.
- **The PS2 kind codes are the same numbers.** `BoxInNonFace` dispatches over primitive kinds
  `0x80..0x84` on both builds, with `0x80`/`0x81` sharing a branch on both — PC just renders the switch
  rebased (`(*vol & 0xff) - 0x80`) where PS2 compares the signed byte directly (`cVar1 == -0x80` … `-0x7c`).
- 🔴 **ELEVEN methods are PS2-PORT-ONLY — do not hunt them.** Four independent groups, each proven by a
  *positive* absence test rather than a failed search:
  - **`PreallocPrepare`** — its **only** PS2 caller is `InterCity_Prebuffering__9C_missionFPc`, i.e. the
    PS2-only city-part streaming subsystem; a caller that exists only on PS2 is the caller-side tell for
    a PS2-only subsystem. It also takes two `const char*` (filenames), which no PC function in the
    lifecycle band does.
  - **The four `*Convex` methods** (`LineInConvexDynamics`, `SphereInConvexDynamics`,
    `LineInConvexStatic`, `SphereInConvexStatic`). PS2's object-kind switch handles
    `0x40–0x44, 0x46, 0x48, **0x49, 0x4a**`; the PC switches handle the same set **minus
    `0x49`/`0x4a`**, plus the PC-only `0x47` (§1d) — and
    `0x49`/`0x4a` are exactly the cases from which the convex handlers are called. Corroborated: **0**
    occurrences of "convex" anywhere in the PC decomp and **0** `convex`/`Convex`/`CONVEX` byte strings
    in `Game.exe`. The PC build has no convex collision primitive at all. Their PS2 VAs (`0x004896c0`–
    `0x0048a150`) sit ~2 MB from the main cluster, i.e. a separate translation unit.
    ⚠️ `0x0048a…` here is a **PS2** address; PC `0x0048a…` is `driver_*` code.
  - **The four "S"-line methods** (`TestLineS`, `_TestLineS`, `LineInSTreeLevel`, `LineInSNonface`) — a
    self-contained chain whose worker `_TestLineS` takes no output parameters and returns a bool: a
    pure yes/no line-of-sight test. The public `TestLineS` is the PS2's general visibility test, with
    callers in auto-aim (`Can_Aimed__7C_human`, `FindNearestEnemyToTurn__8C_player`), perception
    (`Tick__11ai_vis_logsFv`, `TickCorpses__11ai_vis_logsFv`), the entity script processor, scent
    tracing (`TraceHuman__8C_entity…`), `canAim__8C_entityFUi`, `ai_wayfind`, `ai_path` and the
    police board.
    At the PC twins of those callers that were read — `ai_vis_logs_tick` (twice) and
    `entity_trace_human_scent` — the PC calls `collision_test_line_horizontal`, the full raycast,
    instead. The absence is a closed set: apart from the CRT scaffolding every PC function in the five
    collision translation units of §2c is mapped in §2, so no PC body is left to be the S-line chain.
  - **`LineInAACapsule` and `SphereInAACapsule`** — a census of the `jal` instructions to each finds
    eight callsites in all (five and three), and every one sits under a kind the PC switches lack:
    dynamic kind **`0x4a`** (`LineInDynamics`, `LineInDynamicsInnerDyn`, `LineInVehicle`,
    `FindDynamicCollsInLine`, `SphereInDynamics`, `SphereInVehicle`; the two vehicle walkers test it
    as `== 'J'`) or static primitive kind **`0x87`** (`LineInNonface`, `SphereInNonface`, whose PS2
    switches run `0x80..0x85` and `0x87`). The PC dispatchers are closed sets — dynamic
    `0x40..0x48`, primitive `0x80..0x84`, vehicle `0x40`/`0x41`/`0x42`/`0x44` plus `0x84` (sphere)
    or `0x45` (line) — so a function reachable
    only through those kinds was never in the PC source; there is nothing for MSVC to have inlined. ⚠️ The line band
    being "exactly one function short" is a GAP, and a gap alone is not evidence for a terminal
    *inlined-on-PC* tag — the evidence that settles it is **which case calls it**.
- **Type `0x48` exists on both builds.** The PS2 `LineInDynamics` has the same `case 0x48`, reading
  the actor from PS2 `g_collision + 0x41c`; but the PS2 `CreateTrackEffect` overloads are 8-byte
  stubs, so no `0x48` object is ever inserted there and the arm is unreachable on the PS2.
- **The `TestLineH` and `TestDBox` public thunks ARE emitted on PC.** Both are real 5-byte `e9` jumps,
  16-aligned and NOP-padded, byte-verified at **their own** addresses:

  | thunk VA | bytes | jumps to | symbol dump |
  |---|---|---|---|
  | `0x00590a30` | `e9 6b 2e ff ff` + `90 90 90` | `0x005838a0` | `kind=thunk size=5 target=0x005838a0` |
  | `0x00590ab0` | `e9 2b ef fe ff` + `90 90 90` | `0x0057f9e0` | `kind=thunk size=5 target=0x0057f9e0` |

  ⚠️ The target addresses `0x005838a0` and `0x0057f9e0` begin `81 EC …` (`sub esp, imm32`, a real
  prologue) — those bytes belong to the TARGETS, not to the thunks. Ghidra names a thunk after its
  target, so a name of the form `thunk_<target>` hands you the other function: say which address a byte
  reading is about before saying what it proves. Ghidra also **inlines a thunk's target into the thunk's
  own decompilation**, so a thunk and its target render as near-duplicate bodies by construction.
- **`TestSphereSphere` lives far outside the class cluster** (PC `0x00470880`, PS2 `0x0015ff20`) — a
  reminder that a class's PC code is not one contiguous cluster. It was found from the caller side:
  PC `railway_update` ×2 ↔ PS2 `PreduceMovement__9C_railway` ×2, plus a driver/car-AI consumer and the
  curve test on both builds.
- **The PS2 decompilation renders template arguments as `<>`.** The symbol table carries the full
  mangled name (e.g. `…PQ23std70vector<P18tDynamicCollObject,Q23std32allocator<…>>`), but the
  decompiled `.c` renders the same definition as `…PQ23std70vector<>`. That applies to 1247 of 7788
  PS2 symbols and to **17 of these 65 `g_collision` symbols**; all 17 bodies are present and were
  body-diffed.

## 4. The singleton's own fields

⚠️ **The `collision_ctor` list below is NOT the whole object.** It starts at `+0x284`, so it
misses `+0x00..+0x283` entirely — and that band is real: it is the **20-slot contact array**
described in §2b, scratch storage the constructor has no reason to zero because the solver
rewrites the count on every call. A "the constructor zeroes exactly these, therefore this is
the field list" argument bounds only what the CONSTRUCTOR touches.

`collision_ctor` zeroes exactly these: `+0x284`, `+0x28c`, `+0x290`, `+0x294`, `+0x298`, `+0x29c`,
`+0x2a0`, `+0x2a4`, `+0x2a8`, `+0x2ac`, `+0x2b0`, `+0x2b4`, `+0x2b8`, `+0x2bc`, `+0x2c4` — every field
of §1. `+0x290` and `+0x2a0` are the allocator bytes at the head of the two vectors; the constructor
fills each from an uninitialised stack byte, the way MSVC copies an empty allocator.

**`+0x2c4` is `g_collision_next_query_actor`**, and it scopes ONE dynamic-collision query to ONE actor.
Both writers use the identical shape — set the field to the actor,
`collision_dyn_set_next_multiple_owners(actor_colls, 2, 0)`, cast the ray, then
`collision_dyn_set_next_owner(0, 0)` and clear the field back to 0. The two writers are
`dog_snap_to_ground` and `human_movement_intern`, on both builds (PS2 puts the same field at
`g_collision + 0x41c`). Its reader is the `case 0x48` arm of `collision_line_in_dynamics` (§1d):
when that ray crosses a blood decal's collision sphere, the field names the actor whose vtable
`+0x90` is called - an attribution of the query to the actor, which arms its bloody prints.

**`+0x2c0` is the bendable-plant gate of the character proxy.** `collision_test_character_proxy`
sets the byte to 1 before it sweeps the first of its three spheres and to 0 before the second and
the third; the only reader is the `0x47` arm of `collision_sphere_in_dynamics` (§1d), and no other
instruction or absolute address reaches it. The singleton lives in static storage and
`collision_ctor` leaves the byte alone, so it is 0 outside that sweep: of all sphere queries, only
the character proxy's first sphere pushes bendable plants.

**Static grid (`+0x284`) inner layout**, proven by `collision_find_hits_for_frame` and
`collision_classify_static_primitives`: dims at `grid+0x18`/`grid+0x1c`, cell array at `grid+0x2c`,
entry count at `grid+0x38`, entry array at `grid+0x3c`, and a secondary allocation at `grid+0x2c` freed
by `collision_free_level_data`. Each entry's `+0x04` is its owning `I3D_frame`.

⚠️ **Use the PC cluster extent `[0x0057e000, 0x00590600)`** for any size measurement — plus the outliers
at `0x00470880` (`collision_earliest_contact_between_spheres`) and `0x00590a00`–`0x00591530` (thunks,
character proxy). A size report is CLIPPED to the range it is given, so a size at a range edge is never
trustworthy: a bound of `0x0058a000` clips `collision_sphere_vs_aabb` to 7872 B; its true size is
17760 B.

## 5. Cross-references

- `Game-ghidra.exe.functions.md` — the `collision_*` registry rows (authoritative names + VAs).
- `structs/car.md`, `structs/entity.md` — consumers that register dynamic-collision objects
  (`human_create_dyn_colls` / `human_erase_dyn_colls`).
- `structs/box.md` — `box_ai_tick`, the sole consumer chain of the slide solver.

## 6. Open items

| item | state |
|---|---|
| the owner bit-8 loops of `DynSetNextCollOwner` / `Mult` / `DynWipeOut` | the PC twins omit them (§3); whether they are inlined elsewhere on the PC or the flag is maintained on another path is not traced |
