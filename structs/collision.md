# `g_collision` — the global collision system

**Created** 2026-07-26 13:53 +0200 · **Modified** 2026-11-08 12:38 +0100

The game's collision service: one **singleton** in static storage that owns the world collision tree
(static geometry) and a **uniform 2-D grid of dynamic collision objects** (`tDynamicCollObject`),
and answers the line / sphere / box / curve intersection queries the rest of the engine asks. It is
the hidden `__thiscall` receiver of every query: `.text` holds **514** `mov ecx, 0x0067a588` and
**no** `push 0x0067a588`, so the object is only ever a receiver and no callsite shows it.

| | |
|---|---|
| PC prefix | `collision_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x2e8` (the last accessed field ends at `+0x2e8`; static storage, no allocation) |
| PC constructor | `collision_ctor` @ `0x005800d0`, run once by `collision_construct_singleton` |
| PC translation unit | `[0x0057d970, 0x00591f20)`, five units guard to guard (§8), plus `collision_earliest_contact_between_spheres` at `0x00470880` |
| PS2 class | `g_collision` (tag `11g_collision`), 64 methods + 1 free function |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The singleton is `g_collision` @ `0x0067a588`; recovered by an ECX back-trace at
`collision_construct_singleton`, whose whole body is `mov ecx, 0x0067a588; jmp collision_ctor`.

```c
typedef struct g_collision {                   // 0x2e8 bytes; the singleton @ 0x0067a588
    /* +0x000 */ S_collision_contact contacts[20];    // [H] the slide solver's contacts (§1a)
    /* +0x280 */ int32_t             contact_count;   // [H] live entries of contacts[]
    /* +0x284 */ tCollisionLevel    *static_level;    // [H] the static collision tree (§7)
    /* +0x288 */ uint32_t            level_data_size; // [M] byte size of the static level block; no reader (.text scan)
    /* +0x28c */ void               *dyn_grid;        // [H] the dynamic-object grid (§4)
    /* +0x290 */ uint8_t             free_cells_alloc;   // [H] allocator byte, free-cell vector
    /* +0x294 */ void              **free_cells_begin;   // [H] recycled cell buckets
    /* +0x298 */ void              **free_cells_end;     // [H] one past the last recycled bucket
    /* +0x29c */ void              **free_cells_cap;     // [H] end of the free-cell vector's storage
    /* +0x2a0 */ uint8_t             pending_alloc;      // [H] allocator byte, pending-empty vector
    /* +0x2a4 */ void              **pending_begin;      // [H] cells that became empty
    /* +0x2a8 */ void              **pending_end;        // [H] one past the last pending cell
    /* +0x2ac */ void              **pending_cap;        // [H] end of the pending-empty vector's storage
    /* +0x2b0 */ gcCallbackRet     (*next_owner_callback)(tDynamicCollObject *, tCSphereParam *); // [H] hit callback of the next query
    /* +0x2b4 */ tDynamicCollObject *next_owner_single;  // [H] storage for the one-owner form
    /* +0x2b8 */ tDynamicCollObject **next_owners;       // [H] -> +0x2b4, or the caller's array
    /* +0x2bc */ uint32_t            next_owner_count;   // [H] 0 = no owner
    /* +0x2c0 */ uint8_t             push_bendable_plants; // [H] set for one sphere only (§5c)
    /* +0x2c4 */ C_actor            *next_query_actor;   // [H] the actor one query is credited to
    /* +0x2c8 */ uint32_t            unused_2c8[2];      // [U] no access; no reader (.text scan)
    /* +0x2d0 */ float               last_vehicle_hit_pos[3]; // [M] hit position of collision_test_sphere_in_vehicle; no reader (.text scan)
    /* +0x2dc */ void               *last_static_cell;   // [M] last cell the static walk visited; no reader (.text scan)
    /* +0x2e0 */ int32_t             last_static_col;    // [M] its column; no reader (.text scan)
    /* +0x2e4 */ int32_t             last_static_row;    // [M] its row; no reader (.text scan)
} g_collision;
```

**Why twenty contacts.** The array fills the `0x280` bytes before `contact_count`: `0x280 / 0x20` =
20 records. `collision_slide_sphere` resets the count on entry and runs at most ten iterations
(its loop counter starts at 10), appending one record per hit, so a call uses at most ten of the
twenty slots. Nothing in the binary derives the twenty; it is the room the layout gives.

**The ctor's field list is not the object.** `collision_ctor` zeroes `+0x284`, `+0x28c`,
`+0x294..+0x29c`, `+0x2a4..+0x2bc` and `+0x2c4`, and fills the two allocator bytes `+0x290` / `+0x2a0`
from an uninitialised stack byte (the way MSVC copies an empty allocator). It leaves the contact
array, the count, `+0x288`, `+0x2c0` and `+0x2d0..+0x2e4` alone: the singleton sits in zeroed
static storage, and each of those is rewritten before it is read.

**The two owner forms.** `collision_dyn_set_next_owner` stores one owner in `next_owner_single` and
points `next_owners` at it (count 1); `collision_dyn_set_next_multiple_owners` points `next_owners`
at the caller's array (count N). Every dynamic query skips the owners, and a hit on a dynamic object
calls `next_owner_callback`.

**The write-only tail.** `collision_load_level` stores in `level_data_size` the byte size of the
file's third block, the buffer that becomes `static_level`. `collision_test_sphere_in_vehicle` stores
its hit position (`tCSphereParam +0x68`) in `last_vehicle_hit_pos` as well as in its out-parameter,
through `add esi, 0x2d0`. `collision_test_line_vertical`, `collision_test_line_vertical_inner_dyn` and
`collision_test_sphere_swept` store the static cell their cell walk visits, with its column and row,
for every visited cell, hit or not. No instruction reads any of them, and the set of places that
could is closed: the singleton is reached only as ECX by the 514 `mov ecx, 0x0067a588` sites, each
followed by a `collision_*` method (one, before `railway_sweep_sphere_along_curve`, is overwritten
unused), plus the contact-array pointer `collision_test_sphere_slide` hands to `box_ai_tick`; the
absolute references into the object hit only `+0x280` and `+0x2c4`, besides the statics after it
(§7). `+0x2c8..+0x2cf` has no access at all.

### 1a. `S_collision_contact` — one contact of the slide solver, `0x20` bytes

A project name; the slide solver has no PS2 counterpart (§6).

```c
typedef struct S_collision_contact {          // 0x20 bytes
    /* +0x00 */ float               normal[3];     // [H] the contact plane's normal
    /* +0x0c */ float               plane_d;       // [H] -dot(normal, contact point)
    /* +0x10 */ float               point[3];      // [H] the sphere centre at impact
    /* +0x1c */ tDynamicCollObject *hit_obj;       // [H] the hit object if dynamic, else 0
} S_collision_contact;
```

`hit_obj` is set only when the hit object's type byte has bit `0x40` (a dynamic object, §5c); a
static hit stores 0.

## 2. Field evidence

Measured by a census of every `this + N` access in the `collision_*` functions, a census of every
absolute reference into `[0x0067a588, 0x0067a8a0)` in `.text`, and body reads.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | contacts | H | `collision_slide_sphere` | `box_ai_tick` (through `collision_test_sphere_slide`) | §6 |
| +0x280 | contact_count | H | `collision_slide_sphere` | `collision_slide_sphere`, `collision_test_sphere_slide` (by its absolute address) | - |
| +0x284 | static_level | H | `collision_ctor`, `collision_load_level`, `collision_free_level_data` | `collision_load_level`, `collision_free_level_data`, the static queries (`collision_test_line_*`, `collision_test_moving_box`, `collision_test_sphere_swept`, `collision_collect_static_hits_in_sphere`, `collision_find_hits_for_frame`) | §7 |
| +0x288 | level_data_size | M | `collision_load_level` | - | the file size minus the position after the two index blocks; no reader (.text scan), §1 |
| +0x28c | dyn_grid | H | `collision_ctor`, `collision_load_level`, `collision_free_level_data` | the `collision_dyn_*` family and every dynamic query | §4 |
| +0x290 | free_cells_alloc | H | `collision_ctor` | - | the allocator byte `std_vector4_insert_n` receives (`lea ecx, [esi+0x290]` in `collision_dyn_optimize_tick`) |
| +0x294 | free_cells_begin | H | `collision_ctor`, `collision_dtor` | `collision_dtor`, `collision_free_level_data`, `collision_load_level`, `collision_dyn_insert`, `collision_dyn_update`, `collision_dyn_clear_all` | - |
| +0x298 | free_cells_end | H | `collision_ctor`, `collision_dtor`, `collision_free_level_data`, `collision_load_level`, `collision_dyn_insert`, `collision_dyn_update`, `collision_dyn_clear_all` | `collision_dyn_optimize_tick`, `collision_dyn_insert`, `collision_dyn_update`, `collision_dyn_clear_all` | - |
| +0x29c | free_cells_cap | H | `collision_ctor`, `collision_dtor` | - | the vector-insert helper reads it through the allocator byte |
| +0x2a0 | pending_alloc | H | `collision_ctor` | - | as `+0x290` |
| +0x2a4 | pending_begin | H | `collision_ctor`, `collision_dtor`, `collision_dyn_erase` | `collision_dtor`, `collision_free_level_data`, `collision_load_level`, `collision_dyn_optimize_tick`, `collision_dyn_erase`, `collision_dyn_clear_all` | - |
| +0x2a8 | pending_end | H | `collision_ctor`, `collision_dtor`, `collision_free_level_data`, `collision_load_level`, `collision_dyn_optimize_tick`, `collision_dyn_erase`, `collision_dyn_clear_all` | `collision_dyn_optimize_tick`, `collision_dyn_erase`, `collision_dyn_update`, `collision_dyn_clear_all` | - |
| +0x2ac | pending_cap | H | `collision_ctor`, `collision_dtor`, `collision_dyn_erase` | `collision_dyn_erase` | - |
| +0x2b0 | next_owner_callback | H | `collision_ctor`, `collision_dyn_set_next_owner`, `collision_dyn_set_next_multiple_owners` | `collision_test_sphere_in_vehicle` | called with ECX = the hit object, only on a hit whose object has type bit `0x40`; the setters store the callback even with owner 0 |
| +0x2b4 | next_owner_single | H | `collision_ctor`, `collision_dyn_set_next_owner` | the owner-skip loops, through `next_owners` | - |
| +0x2b8 | next_owners | H | `collision_ctor`, `collision_dyn_set_next_owner`, `collision_dyn_set_next_multiple_owners` | `collision_box_in_dynamics`, `collision_line_in_dynamics`, `collision_line_in_dynamics_inner`, `collision_line_in_vehicle`, `collision_sphere_in_dynamics`, `collision_sphere_in_vehicle` | - |
| +0x2bc | next_owner_count | H | `collision_ctor`, `collision_dyn_set_next_owner`, `collision_dyn_set_next_multiple_owners` | the same six queries | - |
| +0x2c0 | push_bendable_plants | H | `collision_test_character_proxy` | `collision_sphere_in_dynamics` | §5c |
| +0x2c4 | next_query_actor | H | `collision_ctor`, `collision_load_level` (0), `dog_snap_to_ground`, `human_movement_intern` (by its absolute address) | `collision_line_in_dynamics` | §5c |
| +0x2c8 | unused_2c8 | U | - | - | no access; no reader (.text scan), §1 |
| +0x2d0 | last_vehicle_hit_pos | M | `collision_test_sphere_in_vehicle` (through `add esi, 0x2d0`) | - | `tCSphereParam +0x68`; no reader (.text scan), §1 |
| +0x2dc | last_static_cell | M | `collision_test_line_vertical`, `collision_test_line_vertical_inner_dyn`, `collision_test_sphere_swept` | - | written for every visited cell, hit or not; no reader (.text scan), §1 |
| +0x2e0 | last_static_col | M | the same three | - | no reader (.text scan), §1 |
| +0x2e4 | last_static_row | M | the same three | - | no reader (.text scan), §1 |

### 2a. `S_collision_contact`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | normal | H | `collision_slide_sphere` | `collision_slide_sphere` (the re-clip), `box_ai_tick` | - |
| +0x0c | plane_d | H | `collision_slide_sphere` | `box_ai_tick` | - |
| +0x10 | point | H | `collision_slide_sphere` | `box_ai_tick` | - |
| +0x1c | hit_obj | H | `collision_slide_sphere` | `box_ai_tick` | - |

## 3. Method family

54 of the 65 PS2 symbols mapped (51 CONFIRMED, 3 STRUCTURAL); 11 are PS2-only (§9b). PC-only:
`collision_ctor`, `collision_construct_singleton`, the slide-solver pair (§6) and the units'
static-init scaffolding. The three `collision_event_payload_*` functions (`0x00593950..0x00593a30`)
carry the prefix but are the payload helpers of the network collision event, in the event-message
unit, and are documented with it (`structs/net_event_msg.md`).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057d970 | `crt_static_init_collision` | `g_collision` | - | - | - | the first unit's `.CRT$XCU` initializer: constructs the singleton and registers its destructor |
| 0x0057d980 | `collision_construct_singleton` | `g_collision` | - | - | - | `mov ecx, 0x0067a588; jmp collision_ctor`; the PS2 exports no `g_collision` ctor |
| 0x0057d990 | `crt_atexit_register_collision` | `g_collision` | - | - | - | `crt_atexit(&crt_atexit_free_collision)`, the only registrar in the five units with a real destructor |
| 0x0057d9a0 | `crt_atexit_free_collision` | `g_collision` | - | - | - | `mov ecx, 0x0067a588; jmp collision_dtor` |
| 0x0057d9b0 | `collision_dtor` | instance | `__dt__11g_collisionFv` | 0x0027a4d0 | CONFIRMED | frees the pending-empty vector, then the free-cell vector, nulling both triples |
| 0x005800d0 | `collision_ctor` | instance | - | - | - | zeroes the fields listed in §1 |
| 0x00580140 | `crt_static_init_collision_line` | the translation unit | - | - | - | the line unit's initializer |
| 0x00580160 | `crt_atexit_register_collision_line` | the translation unit | - | - | - | registers the shared no-op destructor |
| 0x00584f70 | `crt_static_init_collision_collect` | the translation unit | - | - | - | the collect unit's initializer |
| 0x00584f90 | `crt_atexit_register_collision_collect` | the translation unit | - | - | - | registers the shared no-op destructor |
| 0x00586080 | `crt_static_init_collision_sphere` | the translation unit | - | - | - | the sphere unit's initializer |
| 0x005860a0 | `crt_atexit_register_collision_sphere` | the translation unit | - | - | - | registers the shared no-op destructor |
| 0x005909d0 | `crt_static_init_collision_api` | the translation unit | - | - | - | the API unit's initializer |
| 0x005909f0 | `crt_atexit_register_collision_api` | the translation unit | - | - | - | registers the shared no-op destructor |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00470880 | `collision_earliest_contact_between_spheres` | `TestSphereSphere__11g_collisionFRC8S_vectorRC8S_vectorfRC8S_vectorRC8S_vectorf` | 0x0015ff20 | CONFIRMED | earliest contact time of two moving spheres (position, velocity, radius each) |
| 0x0057da00 | `collision_free_level_data` | `DeleteCollsLevel__11g_collisionFb` | 0x002768e0 | CONFIRMED | frees the static level and the dynamic grid, rewinds the two vectors |
| 0x0057dca0 | `collision_load_level` | `LoadCollision__11g_collisionFPCcb` | 0x00276e10 | CONFIRMED | loads the static tree from a `.dta` (magic `0x43666947`), builds `static_level` and `dyn_grid`; 14 callsites |
| 0x0057e740 | `collision_dyn_optimize_tick` | `DynOptimizeTick__11g_collisionFv` | 0x00277eb0 | CONFIRMED | per-tick: drains the pending-empty vector into the free-cell vector; called by `game_tick` |
| 0x0057e7f0 | `collision_dyn_insert` | `DynInsert__11g_collisionFP18tDynamicCollObject` | 0x00278150 | CONFIRMED | registers one object in its cell (gate: `obj+0x0c == 0`); 51 callsites |
| 0x0057e9e0 | `collision_dyn_erase` | `DynErase__11g_collisionFP18tDynamicCollObject` | 0x00278340 | CONFIRMED | unregisters one object; does not free it |
| 0x0057ebf0 | `collision_dyn_update` | `DynUpdate__11g_collisionFP18tDynamicCollObject` | 0x002784c0 | CONFIRMED | re-buckets an object after it moved; no-op when the cell is unchanged |
| 0x0057ee40 | `collision_dyn_set_next_owner` | `DynSetNextCollOwner__11g_collisionFP18tDynamicCollObjectPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet` | 0x002787c0 | CONFIRMED | one owner and a callback for the next dynamic query; with owner 0 it sets the count to 0 and still stores the callback |
| 0x0057ee90 | `collision_dyn_set_next_multiple_owners` | `DynSetNextCollOwnerMult__11g_collisionFPP18tDynamicCollObjectUiPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet` | 0x00278870 | CONFIRMED | an owner array and a callback for the next dynamic query |
| 0x0057eed0 | `collision_dyn_clear_all` | `DynWipeOut__11g_collisionFv` | 0x00278910 | CONFIRMED | empties every bucket of the grid and recycles every cell |
| 0x0057f150 | `collision_dyn_take_out_to_list` | `DynGetIn__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>` | 0x00278db0 | CONFIRMED | erases a registered object from the grid and appends it to the caller's vector |
| 0x0057f300 | `collision_dyn_put_back_from_list` | `DynGetOut__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>` | 0x00278e10 | CONFIRMED | finds and erases the object in the caller's vector, re-inserts it into the grid |
| 0x0057f360 | `collision_box_in_dynamics` | `BoxInDynamic__11g_collisionFR10tCBoxParam` | 0x00278e90 | CONFIRMED | box vs the dynamic grid: cell-rectangle sweep, owner skip, type switch |
| 0x0057f5e0 | `collision_box_in_primitive_volume` | `BoxInNonFace__11g_collisionFP16g_volume_trifaceR10tCBoxParam` | 0x00279220 | CONFIRMED | box vs one non-triangle primitive (kinds `0x80..0x84`) |
| 0x0057f7f0 | `collision_box_in_tree_level` | `BoxInTreeLevel__11g_collisionFP14tCollTreeLevelR10tCBoxParam` | 0x00279510 | CONFIRMED | box vs one static-tree level |
| 0x0057f9e0 | `collision_test_moving_box` | `_TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi` | 0x00279960 | CONFIRMED | swept test of an oriented box between two transforms |
| 0x00580170 | `collision_line_in_dynamics` | `LineInDynamics__11g_collisionFR11tCLineParam` | 0x0027a580 | CONFIRMED | line vs the dynamic grid; the narrow phase is `collision_line_vs_aabb` |
| 0x005806b0 | `collision_line_in_dynamics_inner` | `LineInDynamicsInnerDyn__11g_collisionFR11tCLineParam` | 0x0027ada0 | STRUCTURAL | the walk of `collision_line_in_dynamics`, except that a ray starting inside a type `0x42` / `0x44` sphere hits at t = 0 and type `0x47` replaces `0x48`; both read the mask at param `+0x3c` |
| 0x00580c50 | `collision_line_vs_aabb` | `LineInAABox__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam` | 0x0027b620 | CONFIRMED | line vs one AABB, per-axis slabs, the near plane |
| 0x00580ff0 | `collision_line_vs_aabb_from_inside` | `LineInAABoxInverted__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam` | 0x0027bc20 | CONFIRMED | the same slab test picking the far plane (the ray starts inside); normal negated |
| 0x00581210 | `collision_line_in_primitive_volume` | `LineInNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam` | 0x0027c380 | CONFIRMED | line vs one non-triangle primitive (kinds `0x80..0x84`) |
| 0x005817d0 | `collision_line_in_tree_level` | `LineInTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam` | 0x0027ccf0 | CONFIRMED | line vs one static-tree level |
| 0x00581c40 | `collision_test_line_vertical` | `_TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027d460 | CONFIRMED | the vertical line test, the horizontal walk's near-vertical case |
| 0x00582120 | `collision_test_line_vertical_inner_dyn` | `_TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027d920 | CONFIRMED | its inner-dynamic variant |
| 0x00582600 | `collision_test_line_horizontal_static` | `TestLineHStatic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi` | 0x0027dde0 | CONFIRMED | the horizontal line test against the static level only |
| 0x00583570 | `collision_test_line_horizontal_dynamic` | `TestLineHDynamic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027f010 | CONFIRMED | the horizontal line test against the dynamic grid only |
| 0x005838a0 | `collision_test_line_horizontal` | `_TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x0027f300 | CONFIRMED | the general raycast against both (in-game confirmed) |
| 0x005848e0 | `collision_line_in_vehicle` | `LineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R11tCLineParam` | 0x00280660 | CONFIRMED | line vs a caller's object vector; never touches `dyn_grid` |
| 0x00584cb0 | `collision_test_line_in_vehicle` | `TestLineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi` | 0x00280c10 | CONFIRMED | its public wrapper; calls the worker once |
| 0x00584fa0 | `collision_collect_primitive_volume_in_sphere` | `LocalityInNonface__11g_collisionFP16g_volume_trifaceRC8S_vectorRC8S_vectorRC8S_vectorfRQ23gif31vector<P18g_collision_header,8>` | 0x00280e90 | CONFIRMED | collects the kind `0x80`, `0x81` and `0x83` primitives whose bounding sphere meets the query sphere; `0x82` and `0x84` are never collected |
| 0x005854a0 | `collision_collect_tree_level_in_sphere` | `LocalityInTreeLevel__11g_collisionFP14tCollTreeLevelRC8S_vectorRC8S_vectorRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8>` | 0x00281930 | CONFIRMED | collects from one static-tree level; the two `Ui` are an inclusive kind range |
| 0x00585510 | `collision_collect_static_hits_in_sphere` | `FindStaticCollsInLocality__11g_collisionFRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8>` | 0x00281a20 | CONFIRMED | the collect family's public entry: clears the out-vector, gathers the static primitives in a radius |
| 0x005856c0 | `collision_find_hits_for_frame` | `FindCollsByFrame__11g_collisionFP9I3D_frameRQ23gif31vector<P18g_collision_header,8>` | 0x00281cb0 | CONFIRMED | collects every non-triangle primitive (kinds `0x80..0x84`, the five arrays at level `+0x38..+0x5c`) whose frame is the given one; triangles carry no frame |
| 0x00585890 | `collision_find_dynamic_line_hits` | `FindDynamicCollsInLine__11g_collisionFRC8S_vectorRC8S_vectorRQ23gif31vector<P18g_collision_header,8>Ui` | 0x002832e0 | STRUCTURAL | collects every dynamic line hit into an out-vector, returns the count |
| 0x005860b0 | `collision_sphere_in_dynamics` | `SphereInDynamics__11g_collisionFR13tCSphereParam` | 0x00284040 | CONFIRMED | sphere vs the dynamic grid |
| 0x00586fd0 | `collision_sphere_vs_edge` | `SphereInEdge__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParamRUiUiUi` | 0x002850a0 | CONFIRMED | swept sphere vs one segment; must beat the best hit at param `+0x50` |
| 0x00587570 | `collision_sphere_vs_triangle` | `SphereInFace__11g_collisionFP16g_volume_trifaceR13tCSphereParamRi` | 0x002856a0 | CONFIRMED | swept sphere vs one triangle |
| 0x00588140 | `collision_sphere_vs_aabb` | `SphereInAABox__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParam` | 0x00286600 | CONFIRMED | swept sphere vs one AABB, inflated by the radius |
| 0x0058c6a0 | `collision_sphere_in_primitive_volume` | `SphereInNonface__11g_collisionFP16g_volume_trifaceR13tCSphereParam` | 0x0028c530 | CONFIRMED | swept sphere vs one non-triangle primitive (kinds `0x80..0x83`) |
| 0x0058e970 | `collision_sphere_in_tree_level` | `SphereInTreeLevel__11g_collisionFP14tCollTreeLevelR13tCSphereParam` | 0x0028ec90 | CONFIRMED | sphere vs one static-tree level; differs from the box sibling only in the param offsets |
| 0x0058eaa0 | `collision_test_sphere_swept` | `_TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x0028ef90 | CONFIRMED | the swept-sphere worker: hit position and normal; returns the hit object (`tCSphereParam +0x80`), 0 when nothing was hit |
| 0x0058f670 | `collision_slide_sphere` | - | - | - | the collide-and-slide solver (§6) |
| 0x0058fb00 | `collision_earliest_contact_between_paths` | `TestCurveCurve__11g_collisionFPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>fPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>f` | 0x0028fa50 | CONFIRMED | earliest contact of two spheres on sampled paths; `FLT_MAX` if none |
| 0x0058fd30 | `collision_sphere_in_vehicle` | `SphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R13tCSphereParam` | 0x0028ff10 | CONFIRMED | sphere vs a caller's object vector |
| 0x00590590 | `collision_test_sphere_in_vehicle` | `TestSphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x002909d0 | CONFIRMED | its public wrapper; packs a `tCSphereParam` |
| 0x00590a00 | `collision_teardown` | `GameDone__11g_collisionFb` | 0x00299920 | CONFIRMED | `collision_dyn_clear_all` then `collision_free_level_data`; called by `game_done` |
| 0x00590a10 | `collision_test_line_vertical_thunk` | `TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299960 | CONFIRMED | public entry: binds the singleton, forwards to `collision_test_line_vertical` |
| 0x00590a20 | `collision_test_line_vertical_inner_dyn_thunk` | `TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299970 | CONFIRMED | public entry over the inner-dynamic vertical test |
| 0x00590a30 | `thunk_collision_test_line_horizontal` | `TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi` | 0x00299980 | CONFIRMED | a 5-byte `jmp` to `collision_test_line_horizontal` (§9) |
| 0x00590a40 | `collision_test_sphere_swept_thunk` | `TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi` | 0x002999a0 | CONFIRMED | public entry over `collision_test_sphere_swept`; 44 callsites |
| 0x00590a60 | `collision_test_sphere_slide` | - | - | - | the solver's public entry; returns the contact array and its count (§6) |
| 0x00590ab0 | `thunk_collision_test_moving_box` | `TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi` | 0x002999b0 | STRUCTURAL | a 5-byte `jmp` to `collision_test_moving_box` (§9) |
| 0x00590ac0 | `collision_test_character_proxy` | `TestPanak2__11g_collisionFP9tgcPanak2` | 0x002999d0 | CONFIRMED | sweeps a three-sphere person proxy (`tgcPanak2`); consumer `human_test_move_coll` |
| 0x00591530 | `collision_test_character_proxy_in_vehicle` | `TestPanak2InVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>P9tgcPanak2` | 0x0029a320 | CONFIRMED | the same proxy against an object vector; consumer `human_test_move_coll_in_owner` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057dab0 | `collision_classify_static_primitives` | `GrantCoherence__FP15tCollisionLevel` | 0x00276a00 | CONFIRMED | after a load, tags every static primitive with a dominant-axis and degeneracy code; a free function on the PS2, whose parameter type names `static_level` a `tCollisionLevel` |

## 4. The dynamic-object grid

`dyn_grid` (`+0x28c`) is a uniform 2-D grid keyed by two integer cell coordinates, held in a
`0x1c`-byte record that `collision_load_level` allocates: `grid+0x00` / `+0x04` = the origin x / z
(the level's minimum), `grid+0x08` / `+0x0c` = the cell width / depth (the level extent divided by
the columns / rows), `grid+0x10` = columns, `grid+0x14` = rows, `grid+0x18` = the cell array, one
dword per cell. The cell at
(col, row) is `*(grid+0x18 + (cols*row + col) * 4)`. A non-zero cell points at a `0x18`-byte bucket
that `collision_dyn_insert` allocates: the cell's (col, row) in its first two words, an allocator
byte at `+0x08`, and a vector of `tDynamicCollObject *` at `+0x0c..+0x14`. A `tDynamicCollObject`
keeps its type in word 0 (§5c) and the back-link to its bucket at `+0x0c`.

### 4a. How the `Dyn*` family fits together

* **`collision_dyn_insert`** registers one object: computes its cell, creates or reuses the bucket,
  links it in (gated by `obj+0x0c == 0`, the object is not in a bucket).
* **`collision_dyn_update`** re-buckets an object after it moved: removes it from the old cell and
  inserts it into the new one; a no-op when the cell is unchanged.
* **`collision_dyn_erase`** unregisters one object (does not free it).
* **`collision_dyn_optimize_tick`** drains the pending-empty vector each tick, zeroes the empty
  cells and recycles their buckets onto the free-cell vector.
* **`collision_dyn_clear_all`** sweeps the whole grid, empties every bucket and recycles every cell.
* **`collision_dyn_set_next_owner` / `collision_dyn_set_next_multiple_owners`** install the
  object(s) the next dynamic query skips, and its hit callback (§1).
* **`collision_dyn_take_out_to_list` / `collision_dyn_put_back_from_list`** lift an object out of
  the grid into a caller-held vector and put it back, so a moving object can be kept out of its own
  queries.

## 5. The queries

### 5a. The query parameter structs

Every query family passes one parameter struct. The offsets below are byte-proven on the box,
sphere, line and character-proxy workers; they are what separates two workers whose bodies are otherwise identical
(`collision_box_in_tree_level` vs `collision_sphere_in_tree_level` differ only in which of these
they read).

| Struct | Offset | Field | Proven by |
|---|---|---|---|
| `tCBoxParam` | +0x00 / +0x0c | box min corner / max corner (the AABB tested) | `collision_box_in_tree_level`, `collision_box_in_primitive_volume` |
| `tCBoxParam` | +0x08, +0x10, +0x14 | the corner components the 6-way overlap test reads | `collision_box_in_primitive_volume` |
| `tCLineParam` | +0x00 | line start | the line testers (`collision_test_line_*`), `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x0c | line delta | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x1c..+0x28 | x / z bounds (min x, min z, max x, max z) | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x2c | line end | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x38 | static mask (the caller's mask `\| 0x20`) | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x3c | dynamic mask | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x40 | best t so far (1.0 at entry) | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x44 | \|delta\|² | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x48 | hit normal | the line testers, `collision_find_dynamic_line_hits` |
| `tCLineParam` | +0x54 | hit object; the `0x40` arms copy the struct's first `0x5c` bytes | the line testers, `collision_find_dynamic_line_hits` |
| `tCSphereParam` | +0x00 | sphere centre (sweep start) | `collision_sphere_vs_triangle`, `collision_sphere_in_dynamics` |
| `tCSphereParam` | +0x0c | sweep delta (end = centre + delta) | `collision_sphere_vs_aabb` |
| `tCSphereParam` | +0x18 | sphere radius | `collision_sphere_vs_triangle`, `collision_sphere_vs_edge`, `collision_sphere_vs_aabb` |
| `tCSphereParam` | +0x1c / +0x24 | query bounds of the tree-level rejection | `collision_sphere_in_tree_level` |
| `tCSphereParam` | +0x38 | sweep direction | `collision_sphere_vs_triangle`, `collision_sphere_vs_edge` |
| `tCSphereParam` | +0x44 | static mask | `collision_sphere_in_tree_level` |
| `tCSphereParam` | +0x48 | dynamic mask | `collision_sphere_in_dynamics`, `collision_sphere_in_vehicle` |
| `tCSphereParam` | +0x50 | the best hit's squared time so far; a candidate must beat it | `collision_sphere_vs_edge` |
| `tCSphereParam` | +0x68 | hit position | `collision_test_sphere_swept`, `collision_test_sphere_in_vehicle` |
| `tCSphereParam` | +0x74 | hit normal | `collision_test_sphere_swept`, `collision_test_sphere_in_vehicle` |
| `tCSphereParam` | +0x80 | hit object, which `collision_test_sphere_swept` returns; the `0x40` arms copy the struct's first `0x88` bytes | `collision_test_sphere_swept` |
| `tgcPanak2` | +0x00..+0x08 | the three sphere radii | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x0c / +0x18 / +0x24 | the three sphere centres | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x30 | motion | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x3c | static mask | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x40 | dynamic mask | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x44 | result position | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x50 | hit normal | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |
| `tgcPanak2` | +0x60 | one hit object per iteration; the vehicle form writes none | `collision_test_character_proxy` |
| `tgcPanak2` | +0x88 | iterations minus 1 (at most 10; 4 in the vehicle form) | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` |

**Kind dispatch.** Every dynamic-object query switches on the low byte of the object's first word,
written as `switch((*obj & 0xff) - 0x40)`, so `case 0` is type `0x40`. The static volume primitives
(`g_volume_triface`) use a separate range dispatched as `switch((*vol & 0xff) - 0x80)` over kinds
`0x80..0x84`, with `0x80` / `0x81` sharing one branch.

### 5b. How to read a query function's name

Names are `collision_<query shape>_<relation>_<target>`. The relation word carries the
information:

| Relation | Meaning | Target is |
|---|---|---|
| `_in_` | search a container: walk a set of candidates and return the first hit | `dynamics`, `vehicle`, `tree_level`, `primitive_volume` |
| `_vs_` | leaf math against one primitive: no iteration, just the intersection test | `aabb`, `edge`, `triangle` |
| `collect_…_in_sphere` | gather all hits into a caller-supplied out-vector | the collect family |

Without the split, "box" would sit on both sides of `_in_` with opposite meanings: the query being
swept in `collision_box_in_tree_level`, the target being tested in a sphere-against-box leaf test.
With it, the query shape is always the word before the relation and the target the word after.

| Target | What it is | Tell in the body |
|---|---|---|
| `_in_dynamics` | the dynamic grid | reads `this+0x28c`; single struct argument |
| `_in_vehicle` | a caller-supplied `std::vector<tDynamicCollObject *>` | walks `param+0x04` / `+0x08`; never touches `+0x28c` |
| `_in_tree_level` | one level of the static tree | count at `lvl+0x00`, pointer array at `lvl+0x04`, `& 0xa0` skip |
| `_in_primitive_volume` | the non-triangle primitives of a `g_volume_triface` | `switch((*vol & 0xff) - 0x80)` over kinds `0x80..0x84` |
| `_vs_aabb` | one axis-aligned bounding box given by its corners | per-axis slab test; the sphere form inflates by the radius first |
| `_vs_edge` | one line segment | quadratic solve clamped to the segment |
| `_vs_triangle` | one triangle of a `g_volume_triface` | three vertex pointers plus the plane normal and distance |
| `collision_test_*` | a public entry point that packs a param struct and delegates to one worker | short; calls exactly one worker |
| `*_thunk` | a one-line public entry that only binds the singleton receiver | two lines, forwards everything |

`collision_line_vs_aabb` has no orientation anywhere in its body; `collision_test_moving_box` takes
two transform matrices because its box rotates.

### 5c. The dynamic-object type byte

Every object in the grid carries its type in the low byte of word 0 (`*obj & 0xff`).
`collision_line_in_dynamics`, `collision_line_in_dynamics_inner` and `collision_sphere_in_dynamics`
switch on it, and their case labels close the set at eight values; any other value, `0x45`
included, falls through to the next object (`collision_box_in_dynamics` handles `0x40`, `0x41`,
`0x42` and `0x44`). The two line dispatchers share the `0x40` / `0x41` / `0x42` / `0x44` arms; only
the inner one handles `0x47` and only the outer one `0x48`.

| Type | Line query arm | Sphere query arm | Inserted by |
|---|---|---|---|
| `0x40` | transforms the ray into the object's frame and tests the box at `+0x24` / `+0x30` (`collision_line_vs_aabb`); the hit normal is rotated back | the same with `collision_sphere_vs_aabb` | `airplane`, `box`, `detector`, `dog`, `door`, `physical_object`, `pumper`, `rail_passenger`, `railway`, `spike_strip`, `vehicle`, `wagon` |
| `0x41` | tests the box at `+0x24` / `+0x30` as stored (no transform) | the same with `collision_sphere_vs_aabb` | - |
| `0x42` | sphere: centre `+0x10`, radius `+0x1c` | sphere against sphere | `airplane`, `bottle`, `dog`, `human_create_dyn_colls`, `pumper`, `railway_add_person`, `railway_switch_game_init` |
| `0x43` | none (skipped) | its own arm, on the horizontal (x, z) components of the query direction | - |
| `0x44` | shares the `0x42` arm | its own arm, which also takes the dot product of the query direction with the centre offset | - |
| `0x46` | none (skipped) | like `0x40`, with the frame's uniform scale applied | - |
| `0x47` | `bendable_plant_push_element` when the owner at `+0x44` is kind `0x1d`; never reports a hit | the same, gated on `push_bendable_plants` | `bendable_plant_game_init` |
| `0x48` | when `next_query_actor` is set, calls that actor's vtable `+0x90` (cbCollTester) with the object; never reports a hit | none (skipped) | `game_create_track_effect` (blood decals, `structs/blood.md` §7a) |

The last column lists the functions that store the value as an immediate and then call
`collision_dyn_insert` (a census of its 51 callsites). `-` = no PC code creates an object of that
type, so the arm is unreachable on the PC: a census of every `mov [mem], imm` in `.text` finds
`0x41`, `0x43`, `0x44` and `0x46` only in six functions that build no collision object (the human
animation state at `+0x74`, network-message and script-decoder stack locals, a racing gate field and
the crash reporter's map reader), while the same census returns every inserter listed for the other
four types. Pedestrians register a type-`0x40` object too: `pedestrian_set_model` and
`pedestrian_create_model` store the type into the object embedded in the pedestrian record, and
`pedestrian_activate_free` and `pedestrian_set_anim` insert it (`structs/pedestrian.md` §1a); the
store and the insert sit in different functions, so the column does not list them.

`0x47` and `0x48` are the types whose arms never set the hit result: the `structs/bendable_plant.md`
clones react to a query and never block it, and a blood decal's sphere only tells the querying actor
that it stepped into blood, which arms its bloody prints (`structs/blood.md` §7e).

**`push_bendable_plants` (`+0x2c0`).** `collision_test_character_proxy` sets it to 1 before the
first of its three spheres and to 0 before the second and the third; the `0x47` arm of
`collision_sphere_in_dynamics` is its only reader, and no other instruction or absolute address
reaches it. Of all sphere queries, only the character proxy's first sphere pushes plants.

**`next_query_actor` (`+0x2c4`)** credits one dynamic query to one actor. Both outside writers use
the same shape — set the field to the actor, `collision_dyn_set_next_multiple_owners(actor_colls, 2,
0)`, cast the ray, then `collision_dyn_set_next_owner(0, 0)` and clear the field — in
`dog_snap_to_ground` and `human_movement_intern`. Its reader is the `0x48` arm: when the ray crosses
a blood decal's sphere, the field names the actor whose vtable `+0x90` is called.

## 6. The collide-and-slide solver

**`collision_slide_sphere` (`0x0058f670`)** is the classic collide-and-slide loop: it lifts the start
by `+0.01` in Y and drops the end by `-0.02`, then iterates at most **ten** times —
`collision_test_sphere_swept` along the remaining motion, and on a hit builds the contact plane,
projects what is left of the motion onto it, and re-clips that against every plane collected earlier
in this call, each re-clip damped by `0.95`. Each hit appends one `S_collision_contact` (§1a).

**`collision_test_sphere_slide` (`0x00590a60`)** is the public entry (the `collision_test_*` shape of
§5b): it runs the solver and hands the caller `(&g_collision, contact_count)`. The array base is the
singleton's own address, which is why the count appears in the decomp as a standalone global and the
array does not. Its one caller is `box_ai_tick`, a method of the PC-only `C_box` (`structs/box.md`),
which copies each contact into its own contact list.

## 7. The static level and the file-scope globals

**`static_level` (`+0x284`)** is a `tCollisionLevel`. `collision_load_level`, the line, sphere and
box walkers, `collision_find_hits_for_frame` and `collision_classify_static_primitives` prove its
header:

| Offset | Field |
|---|---|
| +0x00..+0x0c | x / z bounds (min x, min z, max x, max z) |
| +0x18 / +0x1c | columns / rows |
| +0x24 / +0x28 | the column and row boundary arrays (columns + 1 and rows + 1 floats, stored from `+0x68` on) |
| +0x2c | the cell array |
| +0x30 / +0x34 | triangle count / array, `0x20` bytes each |
| +0x38 / +0x3c | kind `0x80` count / array, `0xb8` bytes each |
| +0x40 / +0x44 | kind `0x81` count / array, `0x20` bytes each |
| +0x48 / +0x4c | kind `0x82` count / array, `0x18` bytes each |
| +0x50 / +0x54 | kind `0x83` count / array, `0xa0` bytes each |
| +0x58 / +0x5c | kind `0x84` count / array, `0x14` bytes each |
| +0x60 / +0x64 | vertex count / array, `0xc` bytes each |

Each non-triangle primitive's `+0x04` is its owning `I3D_frame`; a triangle has none. `collision_free_level_data` frees the `+0x2c` allocation and then the level.

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_collision` | 0x0067a588 | the singleton (§1) | `collision_ctor` | the 514 receiver sites |
| `g_collision_contact_count` | 0x0067a808 | `g_collision + 0x280`, the field of §1 under its absolute address | `collision_slide_sphere` | `collision_test_sphere_slide` |
| `g_collision_next_query_actor` | 0x0067a84c | `g_collision + 0x2c4`, the field of §1 under its absolute address | `dog_snap_to_ground`, `human_movement_intern` | `collision_line_in_dynamics` |
| `g_collision_level_loaded` | 0x0067a878 | set to 1 when a level loads | `collision_load_level` | - |
| `g_collision_last_sphere_distance` | 0x0067a880 | the last swept-sphere hit's squared distance | `collision_test_sphere_swept`, `collision_test_sphere_in_vehicle` | `collision_test_character_proxy`, `collision_test_character_proxy_in_vehicle` (keep the nearest of their three spheres) |

The last two sit right after the singleton and are reached only by absolute address, never through
`this`: they are file-scope statics, not fields. `0x0067a888` belongs to `line_batch`
(`structs/line_batch.md`).

**Size measurements.** Use the PC cluster extent `[0x0057e000, 0x00590600)` plus the outliers at
`0x00470880` and `0x00590a00..0x00591530`. A size report is clipped to the range it is given, so a
size at a range edge is never trustworthy: a bound of `0x0058a000` clips `collision_sphere_vs_aabb`
to 7872 bytes; its true size is 17760.

## 8. The translation units

From the MSVC `.CRT$XCU` initializer table at `.data:0x00646190..0x006461bc`, which lists every
unit's initializer in link order, paired with the code that follows each:

| PC range | Content | Source file |
|---|---|---|
| `[0x0057d970, 0x00580140)` | the singleton's lifecycle, level load / free, the `Dyn*` grid, `Box*` | `g_c_glob.cpp` |
| `[0x00580140, 0x00584f70)` | the `Line*` family | `g_c_line.cpp` |
| `[0x00584f70, 0x00586080)` | the collect family | `g_c_linebs.cpp` |
| `[0x00586080, 0x005909d0)` | the `Sphere*` family and the slide solver | `g_c_sph.cpp` |
| `[0x005909d0, 0x00591f20)` | the teardown, the public thunks, the character proxy | `g_colls.cpp` |

The file names are the PS2 build's (§9). The PC scaffolding is named after each unit's content
(`crt_static_init_collision_line`, `_collect`, `_sphere`, `_api`) so that no cross-build knowledge is
needed to use it. Each unit emits one `atexit` registrar; `crt_atexit_register_collision` is the only
one that registers a real destructor instead of the shared no-op. Subtract all of the scaffolding
before reading a rank-order comparison against the PS2 symbol order: it shifts every rank below it.

## 9. PC vs PS2

* **The population is a VA range, not a class.** A free function has an empty class column in the
  PS2 symbol table, so an enumeration keyed on `class == g_collision` cannot see one. Enumerating the
  PS2 band `0x00276000..0x0029b000` with the class column ignored returns 65 symbols: the 64 class
  methods plus `GrantCoherence__FP15tCollisionLevel`, the twin of
  `collision_classify_static_primitives`. Complete families: `Dyn*` (9/9), line test, `Box`,
  `Sphere`, the `Line` counterparts, the collect family, the character-proxy pair, the lifecycle chain
  (dtor / load / free / teardown) and every public thunk the PC emits. 17 of the 65 symbols carry
  template arguments that the PS2 decompilation renders as `<>`; all 17 bodies are present and were
  body-diffed.
* **The API from the mangled signatures.** Query families: `Line*` (ray / segment), `Sphere*`
  (swept sphere), `Box*` (AABB), plus `TestCurveCurve`, `TestPanak2` and the `*Convex` pair, each
  split by target: `*InDynamics` / `*InVehicle` (dynamic), `*Static` / `*Nonface` / `*TreeLevel`
  (static tree), `*InAABox` / `*InAACapsule` (primitive tests). The bare `TestLine*` / `TestSphere*`
  at `0x00299xxx` are 8–12 byte public thunks over the `_Test*` workers. Element and parameter types:
  `tDynamicCollObject`, `tDynamicCollObjectConvex`, `tCLineParam`, `tCSphereParam`, `tCBoxParam`,
  `g_volume_triface`, `g_volume_convex`, `tCollTreeLevel`, `g_collision_header`, `tgcPanak2`
  (a three-sphere person figure), `gcol_CurveSphereStruct`.
* **PS2 names of the §4 / §5 vocabulary.** `collision_dyn_take_out_to_list` /
  `collision_dyn_put_back_from_list` are `DynGetIn` / `DynGetOut`, which read backwards: "GetIn" gets
  the object into the caller's list, out of collision. The collect family is `Locality*` / `Find*`;
  the targets `_in_primitive_volume`, `_vs_aabb` and `_vs_triangle` are `NonFace`, `AABox` and `Face`.
* **Field rebase.** The PC grid-owner fields sit `0x158`–`0x160` below their PS2 counterparts, and
  the delta is not one constant: `dyn_grid` maps PS2 `+0x3ec` ↔ PC `+0x28c` (Δ `0x160`), the pending
  and owner fields map at Δ `0x158` (PS2 pending `+0x3fc` / `+0x400` ↔ PC `+0x2a4` / `+0x2a8`; PS2
  owners `+0x408..+0x414` ↔ PC `+0x2b0..+0x2bc`), and `next_query_actor` is PS2 `+0x41c`. The static
  level is PS2 `+0x3e4` ↔ PC `+0x284`, pinned by the inner offsets `+0x38` / `+0x3c` of
  `FindCollsByFrame`. Match by structure and the cell-index formula, never by one constant delta.
  `LoadedLevel` and `lastSphereDistance` are unmangled file-scope globals on the PS2 too.
* **Parameter-struct rebase.** `tCBoxParam` maps PC `+0x00` ↔ PS2 `+0x50`, proven on `BoxInTreeLevel`
  and `BoxInNonFace`; the sphere struct's fields sit at different offsets entirely.
* **The grid bucket is 4 bytes larger on PC** — `operator_new(0x18)` vs `__nw__(0x14)`, both in
  `DynInsert`: the PS2 bucket is `{col, row, vector}` with the vector at `+0x08`; the PC one puts an
  allocator byte there and the vector at `+0x0c`.
* **The owner bit-8 loops are PS2-only in the decompiled bodies.** The PS2 `DynSetNextCollOwner` /
  `Mult` walk the current owner array clearing bit `0x08` of each owner's `+8`, install the new
  owners, then set the bit on the new set; `DynWipeOut` has a matching pre-loop. The PC twins install
  the owner fields without those loops (Open item 1); the setter structure is byte-exact, which makes
  the twins CONFIRMED.
* **`gcol_CurveSphereStruct` is 32 bytes on PS2 and 16 on PC** — PS2 pads each `{x, y, z, t}` sample
  to two quadwords, putting the time at `+0x10` against the PC's `+0x0c`; the element-count guard reads
  `>> 5` on PS2 and `>> 4` on PC with the same meaning (at least two samples). `TestCurveCurve`
  inlines the two-moving-spheres solver on PS2 and calls it out of line on PC (size ratio 2.16).
* **The classify epsilon** is 0.0001 on PC against 0.0002 on PS2.
* **The kind codes are the same numbers.** `BoxInNonFace` dispatches over `0x80..0x84` on both builds
  with `0x80` / `0x81` sharing a branch; the PC renders the switch rebased, the PS2 compares the signed
  byte (`-0x80` … `-0x7c`). Type `0x48` exists on both builds (the PS2 reads the actor from
  `+0x41c`), but the PS2 `CreateTrackEffect` overloads are 8-byte stubs, so no `0x48` object is ever
  inserted there.
* **Link order.** The PS2 links its game files alphabetically; the `__sinit_*` list and the code
  agree from `C_wagon.cpp` → `console.cpp` → `g_c_glob.cpp` through `g_dabing.cpp`. `G_Camera`
  (`g_camera.cpp`) fills `[0x00290d50, 0x00299920)` after the `Sphere*` band; `GameDone`, the public
  `Test*` thunks and the two `TestPanak2*` follow at `0x00299920`, and `Dabing*` at `0x0029ac40`, so
  that block is `g_colls.cpp`; the `Locality*` / `Find*` block `[0x00280e90, 0x00284040)` sits between
  `g_c_line.cpp` and `g_c_sph.cpp`, so it is `g_c_linebs.cpp`. The PS2 manifest carries a sixth file,
  the PS2-only `g_c_convex.cpp`. Within the cluster the `Dyn*` emission order is 1:1
  (`OptimizeTick, Insert, Erase, Update, SetNextCollOwner, SetNextCollOwnerMult, WipeOut`).
* **`TestSphereSphere` lives outside the cluster** (PC `0x00470880`, PS2 `0x0015ff20`); it was found
  from the callers: PC `railway_update` ×2 ↔ PS2 `PreduceMovement__9C_railway` ×2.
* **The `TestLineH` and `TestDBox` public thunks are emitted on PC** — real 5-byte `e9` jumps,
  16-aligned and NOP-padded, byte-verified at their own addresses:

  | Thunk VA | Bytes | Jumps to |
  |---|---|---|
  | `0x00590a30` | `e9 6b 2e ff ff` + `90 90 90` | `0x005838a0` |
  | `0x00590ab0` | `e9 2b ef fe ff` + `90 90 90` | `0x0057f9e0` |

  The targets begin `81 ec …` (`sub esp, imm32`, a real prologue); those bytes belong to the targets.
  Ghidra names a thunk after its target and inlines the target into the thunk's decompilation, so a
  thunk and its target render as near-duplicates by construction.

### 9a. The PC-only functions

`collision_ctor` and `collision_construct_singleton` have no PS2 counterpart: the PS2 exports no
`g_collision` constructor. The slide-solver pair is PC-only by a closed-set measurement: the PS2
collision band, enumerated with the class column ignored, holds 64 methods and 1 free function, none
of them a slide solver.

### 9b. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00276610 | `PreallocPrepare__11g_collisionFPCcPCc` | its only PS2 caller is `InterCity_Prebuffering__9C_missionFPc`, the PS2-only city-streaming subsystem; no PC lifecycle function takes two file names |
| 0x0027bf40 | `LineInAACapsule__11g_collisionFffR11tCLineParam` | its five callsites all sit under kinds no PC switch has (dynamic `0x4a`, static primitive `0x87`) |
| 0x00282340 | `_TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi` | the worker of the S-line chain, a yes/no line-of-sight test ¹ |
| 0x00282630 | `LineInSTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam` | the S-line chain's tree-level walker ¹ |
| 0x00282c40 | `LineInSNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam` | the S-line chain's primitive test ¹ |
| 0x0028bc00 | `SphereInAACapsule__11g_collisionFffR13tCSphereParam` | its three callsites all sit under kinds no PC switch has (dynamic `0x4a`, static primitive `0x87`) |
| 0x00299990 | `TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi` | the S-line chain's public entry ¹ |
| 0x004896c0 | `LineInConvexDynamics__11g_collisionFR11tCLineParamP24tDynamicCollObjectConvex` | no convex primitive in the PC build ² |
| 0x00489a10 | `SphereInConvexDynamics__11g_collisionFR13tCSphereParamP24tDynamicCollObjectConvex` | no convex primitive in the PC build ² |
| 0x00489e00 | `LineInConvexStatic__11g_collisionFR11tCLineParamP15g_volume_convex` | no convex primitive in the PC build ² |
| 0x0048a150 | `SphereInConvexStatic__11g_collisionFR13tCSphereParamP15g_volume_convex` | no convex primitive in the PC build ² |

¹ The public `TestLineS` is the PS2's general visibility test, called from auto-aim
(`Can_Aimed__7C_human`, `FindNearestEnemyToTurn__8C_player`), perception (`Tick__11ai_vis_logsFv`,
`TickCorpses__11ai_vis_logsFv`), the entity script processor, scent tracing, `canAim__8C_entityFUi`,
`ai_wayfind`, `ai_path` and the police board. At the PC twins of those callers that were read —
`ai_vis_logs_tick` (twice) and `entity_trace_human_scent` — the PC calls
`collision_test_line_horizontal`, the full raycast, instead. The absence is a closed set: apart from
the scaffolding every PC function in the five units of §8 is mapped in §3, so no PC body is left to be
the S-line chain.

² The PS2 object-kind switch handles `0x40–0x44, 0x46, 0x48, 0x49, 0x4a`; the PC switches handle
the same set minus `0x49` / `0x4a`, plus the PC-only `0x47` (§5c), and `0x49` / `0x4a` are exactly
the cases that call the convex handlers. The PC decomp has no "convex" anywhere and `Game.exe` no
`convex` string in any case. The four PS2 addresses sit about 2 MB from the cluster, in the separate
`g_c_convex.cpp` (a PS2 `0x0048a…` address is not the PC's `0x0048a…`, which is `drop_in_weapon` code).

The capsule pair is not "inlined on PC": the PC dispatchers are closed sets — dynamic `0x40..0x48`,
primitive `0x80..0x84`, vehicle `0x40` / `0x41` / `0x42` / `0x44` plus `0x84` (sphere) or `0x45`
(line) — so a function reachable only through `0x4a` or `0x87` was never in the PC source.

## 10. Related

`Game-ghidra.exe.functions.md` (the `collision_*` rows) · `structs/box.md` (`box_ai_tick`, the slide
solver's consumer) · `structs/bendable_plant.md` (type `0x47`) · `structs/blood.md` (type `0x48`) ·
`structs/human.md` (`human_create_dyn_colls`, the character proxy's consumer) · `structs/car.md` and
`structs/vehicle.md` (the vehicle object vectors) · `structs/net_event_msg.md` (the collision event
payload) · `structs/line_batch.md` (the next block of statics).

## 11. Open items

1. **The owner bit-8 loops.** The PC twins of `DynSetNextCollOwner` / `Mult` / `DynWipeOut` omit
   them (§9); whether the PC maintains that owner flag on another path is not traced.
