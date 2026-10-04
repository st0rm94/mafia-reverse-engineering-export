# `check_seg` / `check_seg_cluster` / `check_seg_clusters` — the map segment index

**Created** 2026-05-27 09:13 +0200 · **Modified** 2026-10-01 08:56 +0200

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

- **PC prefixes:** `check_seg_*` · `check_seg_cluster_*` · `check_seg_clusters_*` — one per
  class, deliberately: the three PS2 tags differ by a single letter, so the prefix is what
  carries the class identity.
- **Singleton:** `check_seg_clusters` lives at **`0x0065521c`**, established by a census of the
  last ECX write before each callsite of its two accessors (`mov ecx, 0x0065521c` at 7/8 and
  12/12 callsites). It has **zero textual occurrences** in the decomp — the receiver lives in
  ECX, which Ghidra hides — so there is no global symbol to rename and a grep for the address
  finds nothing. It sits 8 bytes below the unrelated `ai_wayfind` singleton at `0x00655224`, and
  it is a **member of the `ai_manager` root object at `ai_manager+0x04`** (§1h).
- **PS2 method count:** 15 across the three classes. **PC mapped: 14 · accounted: 15.**
- **PC cluster:** `[0x00449bd0, 0x0044a620)` — one translation unit, plus the folded destructor
  at `0x00566200`.

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. Object layout (PC offsets)

### 1a. `check_seg` — one map segment (`0x2c` bytes)

```c
typedef struct check_seg {                   // 0x2c bytes; 4 of them per check_seg_cluster
    /* +0x00 */ uint8_t   _alloc;            // [H] MSVC std::vector empty-allocator member
    /* +0x04 */ uint32_t *nodes_begin;       // [H] sorted C_web_node ARRAY INDICES - see §1b
    /* +0x08 */ uint32_t *nodes_end;         // [H] count = (end - begin) >> 2
    /* +0x0c */ uint32_t *nodes_cap;         // [H] capacity pointer
    /* +0x10 */ uint32_t  packed;            // [H] type nibbles + special count - see §1c.
                                             //     ALSO the empty-slot marker: 0 = free
    /* +0x14 */ float     centre[3];         // [H] the segment's world-space centre, file data
                                             //     copied by check_seg_clusters_add_segment;
                                             //     no function reads it (section 4a)
    /* +0x20 */ float     radius;            // [H] the segment's radius, 2.7..10.6 in the
                                             //     shipped files; no function reads it
    /* +0x24 */ void     *own_payload;       // [H] the segment's own typed record; its size is
                                             //     0x28/0x34/0x10 for type 1/2/4. Type 8
                                             //     segments own nothing here
    /* +0x28 */ void    **specials;          // [H] array of special payload pointers, stride 4,
                                             //     length = packed >> 0x1c
} check_seg;                                 // 0x2c
```

### 1b. The index vector holds `C_web_node` **array indices** — byte-proven

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

### 1c. The packed word at `+0x10`

One dword carries the whole type table:

| Bits | Meaning |
|---|---|
| `0..3` | the segment's **own** type code — read by `check_seg_get_type_and_data` |
| `4*(i+1) .. 4*(i+1)+3` | **special `i`'s** type code, for i in 0..5 — read by `check_seg_get_special` |
| `28..31` | the **special count** — read by `check_seg_get_special_count` |

So a segment carries at most **7 type nibbles** and the count is capped at 15 by the field width.
**A zero word means the slot is free** — `check_seg_clusters_get_segment` returns NULL on it,
`check_seg_clusters_get_segment_count` skips it and `check_seg_clusters_add_segment` claims it.

### 1d. Type codes and the special record

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

**Type 2 is the cover corner — `ai_seg_corner`**, byte-proven on both builds; full layout in §1f.
`ai_wayfind_find_nearest_hide_corner` scans every segment for `type == 2` and passes that payload
straight into `hasCornerVisHide__10ai_wayfindF**P13ai_seg_corner**…` — the PS2 signature types the
PC payload directly. `ai_wayfind_plan_route_via_corner` (route strategy 2) does the same, and
`entity_save_action_man` / `entity_load_action_man` serialise type-2 payloads for savegames:
**four independent consumers, all testing `== 2`, all treating the payload as a corner.**

**Types 4 and 8 have no consumer.** In the 47 shipped files the PC loads, specials are only ever
type 1 or 2, type 4 occurs only as a segment's own type, and type 8 never occurs (§4a). A
segment's own type and own payload are read only by `check_seg_clusters_load` and
`check_seg_clusters_clear` — a closed set of two `E8` calls to `check_seg_get_type_and_data` —
so the PC loads and frees them without using them.

### 1e. `check_seg_cluster` (`0xb0`) and `check_seg_clusters` (the singleton)

```c
typedef struct check_seg_cluster {           // 0xb0 bytes
    check_seg seg[4];                        // [H] exactly four - proven by check_seg_cluster_ctor
} check_seg_cluster;

typedef struct check_seg_clusters {          // singleton at 0x0065521c
    /* +0x00 */ uint32_t            count;   // [H] number of clusters
    /* +0x04 */ check_seg_cluster  *blocks;  // [H] array; the 4 bytes BELOW it hold the element
                                             //     count for the MSVC array-teardown helper
} check_seg_clusters;
```

The array is **count-prefixed**: `check_seg_clusters_init` and `_add_segment` allocate
`n * 0xb0 + 4`, write `n` into the first dword and hand back `base + 4` — the same idiom
`web_node_resize_connection_array` uses (`structs/web_node.md`).

**Segment capacity only ever grows, and only by one cluster at a time.** `_add_segment` fills
the last cluster's four slots first and reallocates the whole array when they are taken, so a
mission with 400 segments performs 100 reallocations at load. That is a real (small) load-time
cost.

### 1f. `ai_seg_corner` — the type-2 special payload (`0x38`)

Decoded from the `ai_wayfind_test_corner_hide_side` ↔ `hasCornerVisHide` body diff, which agrees
field for field with a one-float shift throughout (§5).

```c
typedef struct ai_seg_corner {               // 0x38 bytes (PC); the payload of a type-2 special
    /* +0x00 */ float pos[3];                // [H] the corner position - the LOS ray's origin and
                                             //     the base the hide position is offset from
    /* +0x0c */ float dir_a[2];              // [H] a 2D (x, z) direction
    /* +0x14 */ float dir_b[2];              // [H] a 2D (x, z) direction
    /* +0x1c */ float dir_c[2];              // [H] a 2D (x, z) direction; NEGATED before its
                                             //     angle test
    /* +0x24 */ float radius;                // [H] the same float check_seg_is_pos_in_any_special
                                             //     reads as the type-2 sphere radius - which is
                                             //     what confirms the field is at +0x24
    /* +0x28 */ float angle_tolerance;       // [H] scaled x1.25 before every angle comparison
    /* +0x2c */ uint32_t _unread[3];         // [U] never touched by either build's corner code
} ai_seg_corner;                             // 0x38
```

**All three directions are 2D `(x, z)` pairs** [H] — the y component is never read, and the hide
position the test produces copies the corner's y through unchanged. The predicate's shape is:

1. reject unless the query point is within the caller's range **and** at least `9.0` (squared)
   away — so a corner is unusable if you are already on top of it;
2. compare the normalised 2D approach direction against the corner's directions with
   `vec2_signed_angle_diff`, all against `angle_tolerance * 1.25`;
3. the **side selector** (0 or 1) is decided by which comparison passed, and it selects which of
   `dir_a`/`dir_b` becomes the offset and which becomes the alignment vector;
4. a horizontal LOS test (`collision_test_line_horizontal`, mask `0x348d02`) must be clear;
5. the hide position is `pos + chosen_dir * (bias + 1 − dot)` stepped back along the approach
   direction, where `bias` is `0.6` or `0.4` depending on the caller's bool.

### 1g. `ai_corner_hide_info` — the result record (`0x24`)

The out-param of both `ai_wayfind_find_nearest_hide_corner` and
`ai_wayfind_plan_route_via_corner`.

```c
typedef struct ai_corner_hide_info {         // 0x24 bytes (PC)
    /* +0x00 */ ai_seg_corner *corner;       // [H] the chosen corner; NULL/0 = nothing found,
                                             //     which is the function's return test
    /* +0x04 */ float hide_pos[3];           // [H] where to stand
    /* +0x10 */ float hide_dir[3];           // [H] which way to face (y forced to 0)
    /* +0x1c */ uint32_t side;               // [H] 0 or 1 - which side of the corner
    /* +0x20 */ float distance;              // [H] distance from the query position; the scan
                                             //     keeps the smallest, seeded at 10.0
} ai_corner_hide_info;                       // 0x24
```

**Independently corroborated by the savegame code.** `entity_save_action_man` cases `0x18` and
`0x1c` write exactly these nine dwords — and, because a raw payload pointer cannot be persisted,
they convert `corner` into a **(segment index, special index) pair** by scanning the whole store
for the matching type-2 payload, writing `0xffffffff / 0xffffffff` when it is not found.
`entity_load_action_man` reverses it. Two AI action types therefore embed an
`ai_corner_hide_info` at offset 0 of their record, which is consistent with the PS2 factory
signature `NewCornerFight(…, ai_corner_hide_info *, …)`.

### 1h. The store is an `ai_manager` member

The `check_seg_clusters` singleton is the object at **`ai_manager+0x04`**
(`structs/ai_manager.md`), and `+0x04` holds on both builds — it sits before any vector member,
so the usual PC/PS2 drift does not reach it (§5).

| phase | PC | PS2 |
|---|---|---|
| load | `check_seg_clusters_load`, from `game_init` | `Load__18check_seg_clustersFv`, from `Init__6C_gameFbb` |
| clear | `check_seg_clusters_clear`, inlined into `game_done` as the last statement of the `ai_manager::Destroy` block | `Clear__18check_seg_clustersFv(root + 4)`, from `Destroy__10ai_managerFv` |

---

## 2. Method family — PC ↔ PS2 (14 of 15 mapped, 15 accounted)

Ordered by **PC VA**, PC name first. **Every row is body-diffed on both builds.**

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| 0x00449bd0 | `check_seg_has_index` | `existIndex__9check_segFUiUiUi` | CONFIRMED | Binary-search membership test over the sorted node-index list. Same algorithm, both endpoint pre-tests, both boundary guards; PS2 recursive, PC the same tail recursion turned into a loop by MSVC. Arity 4↔4 |
| 0x00449c30 | `check_seg_get_special_count` | `NumSpecials__9check_segFv` | CONFIRMED | Top nibble of the packed word. One statement, identical on both builds. Ratio **1.00** |
| 0x00449c40 | `check_seg_get_type_and_data` | `GetType__9check_segFPUiPPv` | CONFIRMED | Nibble 0 + the `+0x24` pointer. Two statements, identical order; the PS2 signature `FPUiPPv` decomposes exactly to the two out-params. Ratio 0.75 |
| 0x00449c60 | `check_seg_get_special` | `GetSpecial__9check_segFUiPUiPPv` | CONFIRMED | The indexed sibling. PC's shift `idx*4 + 4` and PS2's `(idx+1)*4` are **the same expression re-factored**; `+0x28` identical on both. Ratio **1.00** |
| 0x00449c90 | `check_seg_is_pos_in_any_special` | `isInSpecial__9check_segFR8S_vector` | CONFIRMED | Point-in-sphere over every special. Same four type arms, same `d² ≤ r²`, and the same quirk of **re-reading the count at the bottom of each iteration**. Ratio 2.72 (§5) |
| 0x00449d80 | `check_seg_clusters_get_segment` | `GetSegment__18check_seg_clustersFUi` | CONFIRMED | Flat index → segment, NULL when out of range **or empty**. Same two-level arithmetic and guard. Arity 2↔2, ratio 1.44 |
| 0x00449dc0 | `check_seg_clusters_get_segment_count` | `NumSegments__18check_seg_clustersFv` | CONFIRMED | `(n-1)*4` + a scan of the last cluster's four slots. The scan bases differ by exactly the field rebase. Arity 1↔1, ratio 1.63 |
| 0x00449e00 | `check_seg_clusters_add_segment` | `AddSegment__18check_seg_clustersFP9check_seg` | CONFIRMED | Step-for-step: free-slot scan, count-prefixed grow-by-one, array-construct, field-by-field copy with per-element vector re-insert, **source teardown**. Arity 2↔2, ratio 1.42 |
| 0x0044a0f0 | `check_seg_cluster_ctor` | `__ct__17check_seg_clusterFv` | CONFIRMED | **The row the container model rests on** — it vector-constructs **4 elements of 0x2c**. Pinned by argument position in the array-construct call on both builds, then body-read |
| 0x0044a110 | `check_seg_cluster_dtor` | `__dt__17check_seg_clusterFv` | CONFIRMED | Destroys the four segments. Same array-destroy call, same element dtor in the matching slot. Ratio 6.0 — the consumer is the evidence, not the ratio |
| 0x0044a120 | `folded_vector_ctor_at_0x04_0044a120` | - | **STRUCTURAL** | **Not a `check_seg` method** ¹ |
| 0x0044a140 | `check_seg_clusters_init` | `Init__18check_seg_clustersFv` | CONFIRMED | Resets the store to one empty cluster. Same allocation shape, single-element array-construct, `count = 1`, same per-segment zeroing loop. Arity 1↔1, ratio **1.19** |
| 0x0044a200 | `check_seg_clusters_clear` | `Clear__18check_seg_clustersFv` | CONFIRMED | Full teardown; frees the own payload only for types 1/2/4. Identical nested walk and header reset. Ratio 1.74 |
| 0x0044a330 | `check_seg_clusters_load` | `Load__18check_seg_clustersFv` | CONFIRMED | The `clusters.seg` reader — §4. The **read sizes are byte-identical across builds**, which is the format itself. Ratio 2.27 |
| 0x00566200 | *(folded into `vector_dispose_at_0x04`)* | `__dt__9check_segFv` | **STRUCTURAL** | PC has **no distinct `check_seg` destructor**: the body is exactly "dispose the std::vector at `+0x04`", and MSVC folded it with ~300 identical destructors binary-wide. Accounted, not renameable |

¹ The 22-byte body constructs `{allocator byte @+0x00, std::vector @+0x04}` and is reached from
three sites with **two element strides**: `check_seg_cluster_ctor` (`crt_ehvec_ctor`, stride
`0x2c`), `glass_shatter_init` (stride `0x18`) and a direct `E8` call in
`program_process_command` on a single `operator_new(0x18)` object — the third only visible in an
image-wide reference scan, not in the stride test. Two strides mean two element types mean two
owners, so no `PS2 twin:` line may sit on it; PS2 `__ct__9check_segFv` builds its vector at
`this+0x10` through a Metrowerks `compressed_pair_imp`, not at `+0x04`. This does not disturb
§1a's "a PC `check_seg` is `0x2c` bytes", which rests on `check_seg_cluster_ctor`'s own `push
0x2c` literal.

**All 15 PS2 methods are accounted for; all three classes are closed.**

---

## 3. Who uses the store

| Consumer | Type filter | What it wants |
|---|---|---|
| `ai_path_find_segment_for_node` | `== 1` | the segment containing a given `C_web_node` index, then that segment's **type-1** special — the frame-anchored obstacle — cached at `ai_path+0x98` ¹ |
| `ai_wayfind_find_nearest_hide_corner` | `== 2` | scans **every** segment for cover corners, testing each with `ai_wayfind_test_corner_hide_side`, and fills an `ai_corner_hide_info` (§1g) |
| `ai_wayfind_plan_route_via_corner` | `== 2` | route strategy 2 — the same corner scan, but it then plans a route to the corner it picked |
| `entity_save_action_man` (×2 loops) | `== 2` | converts an action's `ai_seg_corner *` into a **(segment, special) index pair** so it can be persisted; writes `0xffffffff/0xffffffff` when the corner is not found. Cases `0x18` and `0x1c` |
| `entity_load_action_man` (×2 loops) | `== 2` | the reverse — resolves a saved index pair back to a live payload pointer |
| `ai_loc_update` | none | records **every** special whose sphere contains the owner, of **all four types**, into its `vector<ai_seg_stat>` (`structs/ai_loc.md`) |

¹ `ai_path+0x98` caches the **type-1 special payload**, not the segment.

Every one of them walks the store the same way: `check_seg_clusters_get_segment_count` as the loop
bound, `check_seg_clusters_get_segment` per index (skipping NULLs), then
`check_seg_get_special_count` / `check_seg_get_special` over the specials.

**The type-agnostic consumer is the interesting one.** Every consumer except `ai_loc_update`
tests `== 1` or `== 2`; only `ai_loc_update` records types 4 and 8 at all, and nothing reads the
`vector<ai_seg_stat>` it records them into (`structs/ai_loc.md` §3). So no code on the PC
consumes a type-4 or type-8 special beyond the sphere test that records it, and the shipped
files contain no special of either type (§4a).

---

## 4. The `clusters.seg` file format

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
| 4 | the packed type word → `+0x10` (§1c) |
| 4 | the radius → `+0x20` |
| 0xc | the centre → `+0x14` |
| 4 | `node_count` |
| 4 × `node_count` | the sorted `C_web_node` array indices (§1b) |
| 0x28 / 0x34 / 0x10 | the segment's **own payload**, sized by its type code (1 / 2 / 4). Type 8 has none |
| 0x38 / 0x38 / 0x18 / 0x2c | one payload **per special**, sized by that special's type code (1 / 2 / 4 / 8) |

The special count is **not stored separately** — it is already in the packed word that opens the
record, and `check_seg_clusters_load` calls `check_seg_get_special_count` on the partly-filled
stack record to drive the loop.

**Loader behaviour worth knowing:** `Load` calls `check_seg_clusters_clear` then
`check_seg_clusters_init` before opening the file, and builds every record in **one reused
stack-local `check_seg`**, handing it to `check_seg_clusters_add_segment` — which *moves* the
contents out and leaves the local empty for the next record. That is why `_add_segment` empties
its source, and it is visible from outside as two `lea ecx,[esp+…]` receivers in an ECX census.

### 4a. The shipped files

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

---

## 5. PC vs PS2

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
  also why the `ai_manager+0x04` anchor (§1h) holds on both builds.
- **The specials' internal offsets shift by exactly one float on PS2** (PC `[4]`/`[9]`/`[0xc]` ↔
  PS2 `[5]`/`[10]`/`[0xd]`), because a PS2 `S_vector` is 4 floats and a PC one is 3 — the same
  padding effect measured on `ai_wayfind_test_corner_hide_side`'s corner fields.
- **`ai_seg_corner` (§1f) confirms that shift across the whole record**, not just one field:
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
- **A real behavioural difference (§7):** `isInSpecial` reads a **different radius for
  type 8** on the two builds. PS2 uses `[0xc]` where type 2 uses `[10]`; PC merges the two arms
  and reads `[9]` for both — and MSVC only merges arms that are *identical in the source*, so
  this is a source-level difference, not a compiler artifact.
- **Factoring differences** account for every out-of-band ratio in §2: GCC duplicates the whole
  distance expression once per type arm in `isInSpecial` (652 B vs 240 B) and duplicates the
  per-type arms in `Load` (1708 B vs 752 B); it emits three separate `delete` calls in `Clear`
  where MSVC merges them into one `if ((t==1)||(t==2)||(t==4))`.
- **The file format itself has zero delta** — every `dtaRead` size in §4 is byte-identical on
  both builds, which is expected (the two builds read the same shipped asset) and is what made
  `Load` an unambiguous match.
- **Mission name:** PC `g_act_mission+0x68` ↔ PS2 `act_mission+0x5c`.
- **The array helpers can never have twins.** PC links the MSVC CRT (`crt_ehvec_ctor` /
  `crt_ehvec_dtor`), PS2 links Metrowerks (`__construct_new_array` / `__destroy_new_array`).
  Their `-` is permanent.
- **No PS2-only methods in any of the three classes**, and only one PC-side absence — the folded
  `check_seg` destructor (§2).

---

## 6. Cross-references

- `structs/web_node.md` — the record the index vector points into (stride `0x1e`).
- `structs/ai_path.md` — `ai_path_find_segment_for_node`, the per-node consumer.
- `structs/ai_wayfind.md` — `ai_wayfind_find_nearest_hide_corner`, the whole-store scan.
- `structs/ai_loc.md` — the type-agnostic consumer and its `vector<ai_seg_stat>`.
- `structs/ai_manager.md` — the root object this store hangs off (§1h).
- `structs/web_path.md` §4 — `check.bin`, the *other* navigation file, still open.

---

## 7. Open items

| item | state |
|---|---|
| the type-8 radius difference (§5) | a source-level difference between the builds; its cause is not explained |
