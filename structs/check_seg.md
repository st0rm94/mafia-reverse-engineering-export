# `check_seg` / `check_seg_cluster` / `check_seg_clusters` — the map segment index

**Created** 2026-05-27 09:13 +0200 · **Modified** 2026-11-09 15:07 +0100

A **`check_seg`** is one **map segment**: a spatial bucket that owns a sorted list of
`C_web_node` array indices plus a handful of typed **"specials"** — navigation geometry (cover
corners and similar) attached to that part of the map. Segments are stored four at a time in a
**`check_seg_cluster`**, and the clusters live in a growable array owned by the singleton
**`check_seg_clusters`**. The whole store is loaded from `Missions\<mission>\clusters.seg` (§4).

It sits *beside* the navigation web rather than above or below it — the web says where you can
walk, the segment index says what geometry is present near a given web node:

```
  ai_wayfind  ->  ai_path                 structs/ai_wayfind.md · structs/ai_path.md
        |               |
        +-------+-------+
                v
        check_seg_clusters                THIS FILE — "what is near web node N?"
                ^
  C_web_path  (nodes + connections)       structs/web_path.md · structs/web_node.md
```

| | |
|---|---|
| PC prefix | `check_seg_clusters_`, `check_seg_cluster_`, `check_seg_`: one per class, because the three PS2 tags differ by a single letter |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `check_seg` `0x2c`; `check_seg_cluster` `0xb0`; `check_seg_clusters` 8, the singleton at `0x0065521c` |
| PC constructor | `check_seg_cluster_ctor`; `folded_vector_ctor_at_0x04_0044a120` for one segment |
| PC translation unit | `[0x00449ba0, 0x0044a620)`, guard to guard |
| PS2 class | `check_seg` (tag `9check_seg`), `check_seg_cluster` (tag `17check_seg_cluster`), `check_seg_clusters` (tag `18check_seg_clusters`), 15 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

**Singleton:** `check_seg_clusters` lives at **`0x0065521c`**, established by a census of the last ECX write before each callsite
of its two accessors (`mov ecx, 0x0065521c` at 7/8 and 12/12 callsites). It has **zero textual occurrences** in the decomp: the
receiver lives in ECX, which Ghidra hides, so there is no global symbol to rename and a grep for the address finds nothing. It sits 8
bytes below the unrelated `ai_wayfind` singleton at `0x00655224`, and it is a **member of the `ai_manager` root object at
`ai_manager+0x04`** (§4e).

## 1. Base field layout

### 1a. `check_seg` — one map segment

```c
typedef struct check_seg {                   // 0x2c bytes; 4 of them per check_seg_cluster
    /* +0x00 */ uint8_t   alloc;             // [H] MSVC's empty allocator member of the node vector
    /* +0x04 */ uint32_t *nodes_begin;       // [H] sorted C_web_node ARRAY INDICES, §4a
    /* +0x08 */ uint32_t *nodes_end;         // [H] count = (end - begin) >> 2
    /* +0x0c */ uint32_t *nodes_cap;         // [H] one past the allocation
    /* +0x10 */ uint32_t  packed;            // [H] type nibbles + special count, §4b; 0 = the slot is free
    /* +0x14 */ float     centre[3];         // [H] the segment's world-space centre, file data; no function reads it (§6a)
    /* +0x20 */ float     radius;            // [H] the segment's radius, 2.7..10.6 in the shipped files; no function reads it
    /* +0x24 */ void     *own_payload;       // [H] the segment's own typed record, 0x28 / 0x34 / 0x10 B for type 1 / 2 / 4; type 8 owns nothing
    /* +0x28 */ void    **specials;          // [H] special payload pointers, stride 4, length = packed >> 0x1c
} check_seg;
```

`centre` is one world vector, x y z.

### 1b. `check_seg_cluster`

```c
typedef struct check_seg_cluster {           // 0xb0 bytes
    /* +0x00 */ check_seg seg[4];            // [H] exactly four, proven by check_seg_cluster_ctor
} check_seg_cluster;
```

Four because `check_seg_cluster_ctor` passes count 4 and stride `0x2c` to `crt_ehvec_ctor`; a flat segment index splits as
`index >> 2` for the cluster and `index & 3` for the slot (`check_seg_clusters_get_segment`).

### 1c. `check_seg_clusters` — the singleton

```c
typedef struct check_seg_clusters {          // 8 bytes, singleton at 0x0065521c
    /* +0x00 */ uint32_t            count;   // [H] number of clusters
    /* +0x04 */ check_seg_cluster  *blocks;  // [H] the array; the 4 bytes BELOW it hold the count for the MSVC array teardown
} check_seg_clusters;
```

The array is **count-prefixed**: `check_seg_clusters_init` and `_add_segment` allocate `n * 0xb0 + 4`, write `n` into the first dword
and hand back `base + 4`, the same idiom `web_node_resize_connection_array` uses (`structs/web_node.md`).

**Segment capacity only ever grows, and only by one cluster at a time.** `_add_segment` fills the last cluster's four slots first and
reallocates the whole array when they are taken, so a mission with 400 segments performs 100 reallocations at load.

### 1d. `ai_seg_corner` — the type-2 special payload

Decoded from the `ai_wayfind_test_corner_hide_side` ↔ `hasCornerVisHide` body diff, which agrees field for field with a one-float
shift throughout (§7).

```c
typedef struct ai_seg_corner {               // 0x38 bytes
    /* +0x00 */ float    pos[3];             // [H] the corner position: the LOS ray's origin and the base of the hide position
    /* +0x0c */ float    dir_a[2];           // [H] a 2D (x, z) direction
    /* +0x14 */ float    dir_b[2];           // [H] a 2D (x, z) direction
    /* +0x1c */ float    dir_c[2];           // [H] a 2D (x, z) direction; negated before its angle test
    /* +0x24 */ float    radius;             // [H] the float check_seg_is_pos_in_any_special reads as the type-2 sphere radius
    /* +0x28 */ float    angle_tolerance;    // [H] scaled x1.25 before every angle comparison
    /* +0x2c */ uint32_t unread_2c;          // [U] file data; no reader (class census)
    /* +0x30 */ uint32_t unread_30;          // [U] check_seg_clusters_load stores 0 after the read; no reader (class census)
    /* +0x34 */ uint32_t unread_34;          // [U] check_seg_clusters_load stores 0 after the read; no reader (class census)
} ai_seg_corner;
```

`pos` is one world vector. Each `dir_*` is **two floats, `(x, z)`** [H]: the y component is never read, and the hide position the test
produces copies the corner's y through unchanged.

### 1e. `ai_corner_hide_info` — the result record

The out-param of both `ai_wayfind_find_nearest_hide_corner` and `ai_wayfind_plan_route_via_corner`.

```c
typedef struct ai_corner_hide_info {         // 0x24 bytes
    /* +0x00 */ ai_seg_corner *corner;       // [H] the chosen corner; 0 = nothing found, the function's return test
    /* +0x04 */ float    hide_pos[3];        // [H] where to stand
    /* +0x10 */ float    hide_dir[3];        // [H] which way to face, y forced to 0
    /* +0x1c */ uint32_t side;               // [H] 0 or 1: which side of the corner
    /* +0x20 */ float    distance;           // [H] distance from the query position; the scan keeps the smallest, seeded at 10.0
} ai_corner_hide_info;
```

`hide_pos` and `hide_dir` are one vector each, x y z.

## 2. Field evidence

Cells come from a census of the three classes' functions, read against the bodies; the payload records from their consumers' bodies.

### 2a. `check_seg`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | alloc | H | `folded_vector_ctor_at_0x04_0044a120` | - | an empty class |
| +0x4 | nodes_begin | H | `check_seg_clusters_add_segment`, `folded_vector_ctor_at_0x04_0044a120` | `ai_loc_update`, `ai_path_find_segment_for_node`, `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init`, `check_seg_has_index` | - |
| +0x8 | nodes_end | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init`, `check_seg_clusters_load`, `folded_vector_ctor_at_0x04_0044a120` | `ai_loc_update`, `ai_path_find_segment_for_node`, `check_seg_clusters_add_segment` | these two pass the range to `check_seg_has_index` |
| +0xc | nodes_cap | H | `folded_vector_ctor_at_0x04_0044a120` | `check_seg_clusters_add_segment` | - |
| +0x10 | packed | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init`, `check_seg_clusters_load` | `check_seg_clusters_add_segment`, `check_seg_clusters_get_segment`, `check_seg_clusters_get_segment_count`, `check_seg_get_special`, `check_seg_get_special_count`, `check_seg_get_type_and_data` | §4b |
| +0x14 | centre | H | `check_seg_clusters_add_segment`, `check_seg_clusters_load` | `check_seg_clusters_add_segment` | only copied |
| +0x20 | radius | H | `check_seg_clusters_add_segment`, `check_seg_clusters_load` | `check_seg_clusters_add_segment` | only copied |
| +0x24 | own_payload | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init`, `check_seg_clusters_load` | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_get_type_and_data` | §4c |
| +0x28 | specials | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init`, `check_seg_clusters_load` | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_get_special` | - |

### 2b. `check_seg_cluster`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | seg | H | `check_seg_cluster_ctor`, `check_seg_cluster_dtor` | `check_seg_clusters_get_segment`, `check_seg_clusters_get_segment_count` | the writers construct and destroy the four segments |

### 2c. `check_seg_clusters`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | count | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init` | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_get_segment`, `check_seg_clusters_get_segment_count` | - |
| +0x4 | blocks | H | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_init` | `check_seg_clusters_add_segment`, `check_seg_clusters_clear`, `check_seg_clusters_get_segment`, `check_seg_clusters_get_segment_count`, `check_seg_clusters_init` | - |

### 2d. `ai_seg_corner`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | pos | H | `check_seg_clusters_load` | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `ai_wayfind_test_corner_hide_side`, `check_seg_is_pos_in_any_special` | - |
| +0xc | dir_a | H | `check_seg_clusters_load` | `ai_wayfind_test_corner_hide_side` | the offset or the alignment vector, by side |
| +0x14 | dir_b | H | `check_seg_clusters_load` | `ai_wayfind_test_corner_hide_side` | the other of the pair |
| +0x1c | dir_c | H | `check_seg_clusters_load` | `ai_wayfind_test_corner_hide_side` | - |
| +0x24 | radius | H | `check_seg_clusters_load` | `check_seg_is_pos_in_any_special` | the corner test itself does not read it |
| +0x28 | angle_tolerance | H | `check_seg_clusters_load` | `ai_wayfind_test_corner_hide_side` | - |
| +0x2c | unread_2c | U | `check_seg_clusters_load` | - | file data; no reader (class census, Open item 2) |
| +0x30 | unread_30 | U | `check_seg_clusters_load` | - | the zero store is in the loader's type-2 arm; no reader (class census, Open item 2) |
| +0x34 | unread_34 | U | `check_seg_clusters_load` | - | as `unread_30` |

### 2e. `ai_corner_hide_info`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | corner | H | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `entity_load_action_man` | `entity_save_action_man` | saved as a segment and special index pair |
| +0x4 | hide_pos | H | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `entity_load_action_man` | `entity_save_action_man` | - |
| +0x10 | hide_dir | H | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `entity_load_action_man` | `entity_save_action_man` | - |
| +0x1c | side | H | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `entity_load_action_man` | `entity_save_action_man` | - |
| +0x20 | distance | H | `ai_wayfind_find_nearest_hide_corner`, `ai_wayfind_plan_route_via_corner`, `entity_load_action_man` | `entity_save_action_man` | - |

## 3. Method family

14 of 15 PS2 methods are mapped and the 15th is accounted for. **Every twin row is body-diffed on both builds.**

### 3a. Construction & destruction

#### `check_seg`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00449ba0 | `crt_static_init_guard_00449ba0` | - | - | - | - | The unit's opening guard |
| 0x00449bc0 | `crt_atexit_register_00449bc0` | - | - | - | - | The guard's `atexit` helper |
| 0x0044a120 | `folded_vector_ctor_at_0x04_0044a120` | instance | - | - | - | Builds `{allocator, vector at +0x04}`; two element strides reach it, so no twin (¹) |
| 0x00566200 | `vector_dispose_at_0x04` | instance | `__dt__9check_segFv` | 0x00372310 | STRUCTURAL | The realisation of the segment's destructor, a fold of about 300 identical destructors |

#### `check_seg_cluster`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0044a0f0 | `check_seg_cluster_ctor` | instance | `__ct__17check_seg_clusterFv` | 0x00372380 | CONFIRMED | Vector-constructs 4 elements of `0x2c`; the row the container model rests on |
| 0x0044a110 | `check_seg_cluster_dtor` | instance | `__dt__17check_seg_clusterFv` | 0x003722b0 | CONFIRMED | Destroys the four segments |

¹ The 22-byte body constructs `{allocator byte @+0x00, std::vector @+0x04}` and is reached from three sites with **two element
strides**: `check_seg_cluster_ctor` (`crt_ehvec_ctor`, stride `0x2c`), `glass_shatter_init` (stride `0x18`) and a direct `E8` call in
`program_process_command` on a single `operator_new(0x18)` object, the third only visible in an image-wide reference scan. Two strides
mean two element types mean two owners, so no `PS2 twin:` line may sit on it (§7). This does not disturb §1a's size, which rests on
`check_seg_cluster_ctor`'s own `push 0x2c` literal.

### 3b. Accessors

No function here is a single load or store of one field.

### 3c. Behaviour

#### `check_seg`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00449bd0 | `check_seg_has_index` | `existIndex__9check_segFUiUiUi` | 0x00371a00 | CONFIRMED | Binary-search membership test over the sorted node list |
| 0x00449c30 | `check_seg_get_special_count` | `NumSpecials__9check_segFv` | 0x00371ac0 | CONFIRMED | The top nibble of the packed word |
| 0x00449c40 | `check_seg_get_type_and_data` | `GetType__9check_segFPUiPPv` | 0x00371ad0 | CONFIRMED | Nibble 0 and the `+0x24` pointer |
| 0x00449c60 | `check_seg_get_special` | `GetSpecial__9check_segFUiPUiPPv` | 0x00371af0 | CONFIRMED | A special's type nibble and payload pointer |
| 0x00449c90 | `check_seg_is_pos_in_any_special` | `isInSpecial__9check_segFR8S_vector` | 0x00371b20 | CONFIRMED | Point-in-sphere over every special, re-reading the count each iteration |

#### `check_seg_clusters`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00449d80 | `check_seg_clusters_get_segment` | `GetSegment__18check_seg_clustersFUi` | 0x00371db0 | CONFIRMED | Flat index to segment; null when out of range or empty |
| 0x00449dc0 | `check_seg_clusters_get_segment_count` | `NumSegments__18check_seg_clustersFv` | 0x00371e10 | CONFIRMED | `(n - 1) * 4` plus a scan of the last cluster |
| 0x00449e00 | `check_seg_clusters_add_segment` | `AddSegment__18check_seg_clustersFP9check_seg` | 0x00371e80 | CONFIRMED | Free-slot scan, grow by one cluster, field copy, empties its source |
| 0x0044a140 | `check_seg_clusters_init` | `Init__18check_seg_clustersFv` | 0x00372410 | CONFIRMED | Resets the store to one empty cluster |
| 0x0044a200 | `check_seg_clusters_clear` | `Clear__18check_seg_clustersFv` | 0x00372500 | CONFIRMED | Full teardown; frees the own payload only for types 1 / 2 / 4 |
| 0x0044a330 | `check_seg_clusters_load` | `Load__18check_seg_clustersFv` | 0x00372710 | CONFIRMED | The `clusters.seg` reader (§6) |

### 3d. Free helpers

The unit holds no free helper.

## 4. What the fields mean

### 4a. The index vector holds `C_web_node` **array indices** — byte-proven

`ai_path_find_segment_for_node` takes a node index, calls `check_seg_has_index` to find the
segment that contains it, and then uses **that same value** to index the web-node array:

```c
node_array = *(g_act_mission + 0x2c) + 4;          // C_web_path's node array
x = *(float *)(node_array + idx * 0x1e);           // stride 0x1e = the C_web_node record
```

`0x1e` is the `C_web_node` stride (`structs/web_node.md`) and `+0x00`/`+0x04`/`+0x08` are its
position — so the stored values index that array directly. They are **array indices, not the
stable node IDs** at `web_node+0x0e`; `web_path_find_node_id` is the function that converts one
to the other, and confusing them is the standing trap in that file.

The list is kept **sorted**, which is what makes `check_seg_has_index` a binary search.

### 4b. The packed word at `+0x10`

One dword carries the whole type table:

| Bits | Meaning |
|---|---|
| `0..3` | the segment's **own** type code — read by `check_seg_get_type_and_data` |
| `4*(i+1) .. 4*(i+1)+3` | **special `i`'s** type code, for i in 0..5 — read by `check_seg_get_special` |
| `28..31` | the **special count** — read by `check_seg_get_special_count` |

So a segment carries at most **7 type nibbles** and the count is capped at 15 by the field width.
**A zero word means the slot is free** — `check_seg_clusters_get_segment` returns NULL on it,
`check_seg_clusters_get_segment_count` skips it and `check_seg_clusters_add_segment` claims it.

### 4c. Type codes and the special record

Four type codes are used: **1, 2, 4, 8**. The code selects the payload size at load time, whether
the segment owns a payload at all, and **which float holds the radius**:

| Type | Own payload | Special payload | Radius offset in the special | Conf |
|---|---|---|---|---|
| `1` | 0x28 B | 0x38 B | `+0x30` (float index `0xc`) | H |
| `2` | 0x34 B | 0x38 B | `+0x24` (float index `9`) | H |
| `4` | 0x10 B | 0x18 B | `+0x10` (float index `4`) | H |
| `8` | none ¹ | 0x2c B | `+0x24`, shared with type 2 | H |

¹ `check_seg_clusters_clear` frees `+0x24` only for types 1/2/4.

Every special begins with its **centre position** at `+0x00`/`+0x04`/`+0x08` (all four types —
`check_seg_is_pos_in_any_special` reads `[0..2]` before the type switch).

**Type 1 is frame-anchored** [H]: `ai_path_find_segment_for_node` reads a name at `special+0x0c`,
resolves it through `mission_find_actor_by_name`, caches the result at `special+0x2c`, forces
the frame's world matrix up to date and then tests the node against `special+0x30` as a radius.
So a type-1 special is a piece of geometry bound to a live scene frame — a door or similar
movable — rather than a static point.

**Type 2 is the cover corner — `ai_seg_corner`**, byte-proven on both builds; full layout in §1d.
`ai_wayfind_find_nearest_hide_corner` scans every segment for `type == 2` and passes that payload
straight into `hasCornerVisHide__10ai_wayfindF**P13ai_seg_corner**…` — the PS2 signature types the
PC payload directly. `ai_wayfind_plan_route_via_corner` (route strategy 2) does the same, and
`entity_save_action_man` / `entity_load_action_man` serialise type-2 payloads for savegames:
**four independent consumers, all testing `== 2`, all treating the payload as a corner.**

**Types 4 and 8 have no consumer.** In the 47 shipped files the PC loads, specials are only ever
type 1 or 2, type 4 occurs only as a segment's own type, and type 8 never occurs (§6a). A
segment's own type and own payload are read only by `check_seg_clusters_load` and
`check_seg_clusters_clear` — a closed set of two `E8` calls to `check_seg_get_type_and_data` —
so the PC loads and frees them without using them.

### 4d. The corner test and its result

**The corner test's shape** (`ai_wayfind_test_corner_hide_side`):

1. reject unless the query point is within the caller's range **and** at least `9.0` (squared)
   away — so a corner is unusable if you stand on top of it;
2. compare the normalised 2D approach direction against the corner's directions with
   `vec2_signed_angle_diff`, all against `angle_tolerance * 1.25`;
3. the **side selector** (0 or 1) is decided by which comparison passed, and it selects which of
   `dir_a`/`dir_b` becomes the offset and which becomes the alignment vector;
4. a horizontal LOS test (`collision_test_line_horizontal`, mask `0x348d02`) must be clear;
5. the hide position is `pos + chosen_dir * (bias + 1 − dot)` stepped back along the approach
   direction, where `bias` is `0.6` or `0.4` depending on the caller's bool.

**Independently corroborated by the savegame code.** `entity_save_action_man` cases `0x18` and
`0x1c` write exactly these nine dwords — and, because a raw payload pointer cannot be persisted,
they convert `corner` into a **(segment index, special index) pair** by scanning the whole store
for the matching type-2 payload, writing `0xffffffff / 0xffffffff` when it is not found.
`entity_load_action_man` reverses it. Two AI action types therefore embed an
`ai_corner_hide_info` at offset 0 of their record, which is consistent with the PS2 factory
signature `NewCornerFight(…, ai_corner_hide_info *, …)`.

### 4e. The store is an `ai_manager` member

The `check_seg_clusters` singleton is the object at **`ai_manager+0x04`**
(`structs/ai_manager.md`), and `+0x04` holds on both builds — it sits before any vector member,
so the usual PC/PS2 drift does not reach it (§7).

| phase | PC | PS2 |
|---|---|---|
| load | `check_seg_clusters_load`, from `game_init` | `Load__18check_seg_clustersFv`, from `Init__6C_gameFbb` |
| clear | `check_seg_clusters_clear`, inlined into `game_done` as the last statement of the `ai_manager::Destroy` block | `Clear__18check_seg_clustersFv(root + 4)`, from `Destroy__10ai_managerFv` |

## 5. Who uses the store

| Consumer | Type filter | What it wants |
|---|---|---|
| `ai_path_find_segment_for_node` | `== 1` | the segment containing a given `C_web_node` index, then that segment's **type-1** special — the frame-anchored obstacle — cached at `ai_path+0x98` ¹ |
| `ai_wayfind_find_nearest_hide_corner` | `== 2` | scans **every** segment for cover corners, testing each with `ai_wayfind_test_corner_hide_side`, and fills an `ai_corner_hide_info` (§1e) |
| `ai_wayfind_plan_route_via_corner` | `== 2` | route strategy 2 — the same corner scan, but it then plans a route to the corner it picked |
| `entity_save_action_man` (×2 loops) | `== 2` | converts an action's `ai_seg_corner *` into a **(segment, special) index pair** so it can be persisted; writes `0xffffffff/0xffffffff` when the corner is not found. Cases `0x18` and `0x1c` |
| `entity_load_action_man` (×2 loops) | `== 2` | the reverse — resolves a saved index pair back to a live payload pointer |
| `ai_loc_update` | - | records **every** special whose sphere contains the owner, of **all four types**, into its `vector<ai_seg_stat>` (`structs/ai_loc.md`) |

¹ `ai_path+0x98` caches the **type-1 special payload**, not the segment.

Every one of them walks the store the same way: `check_seg_clusters_get_segment_count` as the loop
bound, `check_seg_clusters_get_segment` per index (skipping NULLs), then
`check_seg_get_special_count` / `check_seg_get_special` over the specials.

**The type-agnostic consumer is the interesting one.** Every consumer except `ai_loc_update`
tests `== 1` or `== 2`; only `ai_loc_update` records types 4 and 8 at all, and nothing reads the
`vector<ai_seg_stat>` it records them into (`structs/ai_loc.md` §4). So no code on the PC
consumes a type-4 or type-8 special beyond the sphere test that records it, and the shipped
files contain no special of either type (§6a).

## 6. The `clusters.seg` file format

Read by `check_seg_clusters_load`; path **`Missions\<mission>\clusters.seg`**, built from the
mission name at `g_act_mission+0x68`. All integers are little-endian on PC.

This is **not** `check.bin`: `clusters.seg` is a separate file shipped alongside it, and
`check.bin`'s remaining format is still open (`structs/web_path.md` §4).

| Offset | Size | Field |
|---|---|---|
| 0x00 | 0x10 | magic **`AISEG-V1.1-JIPO`**, compared case-insensitively. A mismatch closes the file and returns 0 |
| 0x10 | 4 | `record_count` |
| 0x14 | … | `record_count` × segment record |

**Segment record:**

| Size | Field |
|---|---|
| 4 | the packed type word → `+0x10` (§4b) |
| 4 | the radius → `+0x20` |
| 0xc | the centre → `+0x14` |
| 4 | `node_count` |
| 4 × `node_count` | the sorted `C_web_node` array indices (§4a) |
| 0x28 / 0x34 / 0x10 | the segment's **own payload**, sized by its type code (1 / 2 / 4). Type 8 has none |
| 0x38 / 0x38 / 0x18 / 0x2c | one payload **per special**, sized by that special's type code (1 / 2 / 4 / 8) |

The special count is **not stored separately** — it is in the packed word that opens the
record, and `check_seg_clusters_load` calls `check_seg_get_special_count` on the partly-filled
stack record to drive the loop.

**Loader behaviour worth knowing:** `Load` calls `check_seg_clusters_clear` then
`check_seg_clusters_init` before opening the file, and builds every record in **one reused
stack-local `check_seg`**, handing it to `check_seg_clusters_add_segment` — which *moves* the
contents out and leaves the local empty for the next record. That is why `_add_segment` empties
its source, and it is visible from outside as two `lea ecx,[esp+…]` receivers in an ECX census.

### 6a. The shipped files

53 missions ship a `clusters.seg`. **Six carry the magic `AISEG-V1.0-JIPO`** — MISE09-SALIERY,
MISE10-SALIERY, MISE11-SALIERY, MISE14-PARNIK, MISE14-SALIERY and MISE17-MESTO — so the loader's
magic compare fails and those scenes run with an empty segment store. The other 47 parse to the
exact file size:

| Type | as a segment's own type | as a special |
|---|---:|---:|
| none (0) | 2 130 | - |
| 1 | 1 819 | 545 |
| 2 | 2 386 | 4 266 |
| 4 | 3 479 | 0 |
| 8 | 0 | 0 |

Every own-type-4 payload is one horizontal vector `(x, 0, z, 0)` of length 0.92 to 1.41 (1.0 on
2 241 of them). Typed segments carry a world position and a radius in `+0x14..+0x23`; untyped
segments carry the MSVC debug fill `0xcdcdcdcd` there, which says the level tool never wrote those
fields for them.

## 7. PC vs PS2

- **Record sizes: `check_seg` PC `0x2c` / PS2 `0x30`; the cluster PC `0xb0` / PS2 `0xc0`.** The
  cluster is 4× the segment on both builds.
- **Field rebase — band by band, NOT a constant delta**, because one band has a delta of zero:

  | Field | PC | PS2 |
  |---|---|---|
  | index vector | `+0x00` (`std::vector`) | `+0x10` (Metrowerks `__vector_pod`) |
  | packed word | `+0x10` | `+0x1c` |
  | centre + radius | `+0x14` | `+0x00` |
  | own payload / specials array | `+0x24` / `+0x28` | `+0x24` / `+0x28` — **identical** |

  Rebasing a single offset across builds here produces nonsense: PC `+0x10` is the packed
  word while PS2 `+0x10` is the vector. Use the table.
- **The container header has zero delta** — count at `+0x00`, array at `+0x04` on both, which is
  also why the `ai_manager+0x04` anchor (§4e) holds on both builds.
- **The specials' internal offsets shift by exactly one float on PS2** (PC `[4]`/`[9]`/`[0xc]` ↔
  PS2 `[5]`/`[10]`/`[0xd]`), because a PS2 `S_vector` is 4 floats and a PC one is 3 — the same
  padding effect measured on `ai_wayfind_test_corner_hide_side`'s corner fields.
- **`ai_seg_corner` (§1d) confirms that shift across the whole record**, not just one field:
  PC `pos[0..2]` / `dir_a[3,4]` / `dir_b[5,6]` / `dir_c[7,8]` / `radius[9]` / `tolerance[10]` ↔
  PS2 `[0..3]` / `[4,5]` / `[6,7]` / `[8,9]` / `[10]` / `[0xb]`. Every field is exactly one float
  higher on PS2 and the three 2D direction pairs are **not** padded — only the leading `S_vector`
  is. That is why the record is `0x38` on PC and `0x3c` on PS2, a single-float difference rather
  than three.
- **`ai_corner_hide_info`'s member order differs between builds** — not a rebase, the fields are
  in a different sequence:

  | Field | PC | PS2 |
  |---|---|---|
  | hide position | `+0x04` | `+0x00` |
  | hide direction | `+0x10` | `+0x10` |
  | `ai_seg_corner *` | **`+0x00`** | **`+0x20`** |
  | side selector | `+0x1c` | `+0x24` |
  | distance | `+0x20` | `+0x28` |

  MSVC does not reorder members, so the two source declarations genuinely differ. The likely reason
  is visible in the PS2 body: it loads and stores those vectors with **quadword VU instructions**
  (`lqc2`/`sqc2`/`vadd`), which need 16-byte alignment, and putting both vectors first at offset 0
  guarantees it. ⇒ Never carry this record's offsets across builds; and treat a PS2 struct whose
  vectors sit at the front as a hint that the port reordered it for the VU.
- **A real behavioural difference (Open item 1):** `isInSpecial` reads a **different radius for
  type 8** on the two builds. PS2 uses `[0xc]` where type 2 uses `[10]`; PC merges the two arms
  and reads `[9]` for both — and MSVC only merges arms that are *identical in the source*, so
  this is a source-level difference, not a compiler artifact.
- **Factoring differences** account for every out-of-band ratio of the bodies: GCC duplicates the whole
  distance expression once per type arm in `isInSpecial` (652 B vs 240 B) and duplicates the
  per-type arms in `Load` (1708 B vs 752 B); it emits three separate `delete` calls in `Clear`
  where MSVC merges them into one `if ((t==1)||(t==2)||(t==4))`.
- **The file format itself has zero delta** — every `dtaRead` size in §6 is byte-identical on
  both builds, which is expected (the two builds read the same shipped asset) and is what made
  `Load` an unambiguous match.
- **Mission name:** PC `g_act_mission+0x68` ↔ PS2 `act_mission+0x5c`.
- **The array helpers can never have twins.** PC links the MSVC CRT (`crt_ehvec_ctor` /
  `crt_ehvec_dtor`), PS2 links Metrowerks (`__construct_new_array` / `__destroy_new_array`).
  Their `-` is permanent.
- **No PS2-only methods in any of the three classes**, and only one PC-side absence — the folded
  `check_seg` destructor (§3a).
- **Body-size ratios of the twins** (PC / PS2): `check_seg_get_special_count` 1.00, `check_seg_get_type_and_data` 0.75,
  `check_seg_get_special` 1.00, `check_seg_is_pos_in_any_special` 2.72, `check_seg_clusters_get_segment` 1.44,
  `check_seg_clusters_get_segment_count` 1.63, `check_seg_clusters_add_segment` 1.42, `check_seg_cluster_dtor` 6.0 (the consumer is the
  evidence), `check_seg_clusters_init` 1.19, `check_seg_clusters_clear` 1.74, `check_seg_clusters_load` 2.27.
- **`check_seg_has_index`** is recursive on the PS2 and the same tail recursion turned into a loop by MSVC on the PC. The PC shift
  `idx*4 + 4` in `check_seg_get_special` and the PS2 `(idx+1)*4` are one expression re-factored.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x003723c0 | `__ct__9check_segFv` | Builds its vector at `this+0x10` through a Metrowerks `compressed_pair_imp`, not at `+0x04`; the PC body that constructs a segment is the fold of §3a |

## 8. Related

- `structs/web_node.md`: the record the index vector points into (stride `0x1e`).
- `structs/ai_path.md`: `ai_path_find_segment_for_node`, the per-node consumer.
- `structs/ai_wayfind.md`: `ai_wayfind_find_nearest_hide_corner`, the whole-store scan.
- `structs/ai_loc.md`: the type-agnostic consumer and its `vector<ai_seg_stat>`.
- `structs/ai_manager.md`: the root object this store hangs off (§4e).
- `structs/web_path.md` §4: `check.bin`, the *other* navigation file, still open.

## 9. Open items

| # | Item | State |
|---|---|---|
| 1 | The type-8 radius difference (§7) | a source-level difference between the builds; its cause is not explained |
| 2 | `ai_seg_corner` `unread_2c`, `unread_30`, `unread_34` | no reader in the class census; why the loader zeroes the last two is not explained |
| 3 | The other payload records | the own payloads (`0x28` / `0x34` / `0x10` B) and the type-1, type-4 and type-8 specials (`0x38` / `0x18` / `0x2c` B): only the fields §4c names are decoded |
