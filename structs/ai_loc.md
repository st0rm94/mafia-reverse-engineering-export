# `ai_loc` — the per-character navigation-locality cache

**Created** 2026-07-30 12:49 +0200 · **Modified** 2026-11-15 23:55 +0100

An `ai_loc` answers one question for one character, cheaply and repeatedly: **"where am I in the
navigation world, and what map geometry is around me?"** It caches the character's resolved
`C_web_node` indices, the web connection its body is standing inside, and a short list of the
`check_seg` **specials** (cover corners, frame-anchored obstacles) whose spheres contain it. Every field
is refreshed lazily, so the AI can ask those questions per frame without running a spatial search per
frame.

```
  ai_wayfind  ->  ai_path              structs/ai_wayfind.md, structs/ai_path.md
        ^
  check_seg_clusters                   structs/check_seg.md: the specials this class records
        ^
  ai_loc   (one per C_human, at +0x724)
```

| | |
|---|---|
| PC prefix | `ai_loc_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x34` (`C_human+0x724..+0x758`) |
| PC constructor | - (`human_ctor` builds the vector head in place) |
| PC translation unit | `[0x0048b5b0, 0x004ae800)`, the `C_human` unit (`structs/human.md` §4); this class is `[0x0048e7d0, 0x0048ee60)` inside it, next to `ai_blob_log` |
| PS2 class | `ai_loc` (tag `6ai_loc`), 4 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The sole `ai_loc_update` callsite, inside `human_update`, loads the receiver as
`lea ecx,[esi+0x724]`. The neighbour `ai_blob_log` sits at `+0x758`, and `0x724 + 0x34 = 0x758` exactly
(`structs/ai_blob_log.md`).

```c
typedef struct ai_loc {                       // 0x34 bytes, at C_human+0x724
    /* +0x00 */ float    cached_pos[3];       // [H] the owner frame's origin at the last refresh
    /* +0x0c */ uint32_t node_a;              // [H] resolved C_web_node array index
    /* +0x10 */ uint32_t node_b;              // [H] the connection's other node
    /* +0x14 */ uint32_t connection_idx;      // [H] index, among node_a's connections, of the corridor the owner stands inside; no reader (.text scan)
    /* +0x18 */ uint32_t flags;               // [H] outcome of the last refresh (§2a)
    /* +0x1c */ uint32_t last_refresh_ms;     // [H] mission clock at the last refresh
    /* +0x20 */ uint8_t  stats_alloc;         // [H] allocator byte of the stats vector
    /* +0x24 */ ai_seg_stat *stats_begin;     // [H] first observed special
    /* +0x28 */ ai_seg_stat *stats_end;       // [H] one past the last observed special
    /* +0x2c */ ai_seg_stat *stats_cap;       // [H] end of the vector's storage
    /* +0x30 */ C_actor *owner;               // [H] the character, set once by ai_loc_init
} ai_loc;
```

**Why `cached_pos` has three entries.** x, y, z of the owner frame's origin. `node_a` and `node_b` hold
`C_web_node` **array indices**, not the stable node ids at `web_node+0x0e` (`structs/web_node.md`);
`web_path_find_node_id` is the converter.

### 1a. `ai_seg_stat` — one observed special, `0x10` bytes

```c
typedef struct ai_seg_stat {                  // 0x10 bytes, the vector's stride
    /* +0x00 */ uint32_t type;                // [H] the special's check_seg type code; no reader (.text scan)
    /* +0x04 */ uint32_t last_seen_ms;        // [H] mission clock of the last observation
    /* +0x08 */ uint32_t hit_count;           // [H] how many observations this record has had; read only by its own increment
    /* +0x0c */ void    *payload;             // [H] the special's payload pointer, the record key
} ai_seg_stat;
```

**The key is the payload pointer**, not the type and not a (segment, special) index pair. A record is
therefore valid only as long as the loaded `clusters.seg` data it points into, which is why
`entity_save_action_man` converts such pointers to index pairs when it writes a savegame
(`structs/check_seg.md` §5).

**Lifetime.** A record expires 4000 ms after its last refresh. `ai_loc_update` refreshes at most every
5000 ms and only after a move of 2 units (§4), so a record always expires before the next refresh can
renew it, and a character standing still ends up with an empty list.

## 2. Field evidence

Measured by a register-tag census of the class's functions, a displacement census of
`C_human+0x724..+0x754` over `.text`, and reading all four bodies. The census finds the four method
receivers (`human_update`, `human_game_init`, `human_activate`, `engine_new_mission`), `human_ctor`,
`human_dtor` and `ai_wayfind_plan_directed_route_v3`, which reads `C_human+0x730`, `+0x734` and `+0x73c`
directly. The record is reached through a walking pointer, so its cells come from the bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | cached_pos | H | `ai_loc_init`, `ai_loc_update` | `ai_loc_update` | the move test compares against it |
| +0xc | node_a | H | `ai_wayfind_get_node_for_position` (through the pointer `ai_loc_update` passes) | `ai_loc_update`, `ai_wayfind_plan_directed_route_v3` | an array index, or `0xffffffff` when no node was found |
| +0x10 | node_b | H | `ai_loc_update` | `ai_loc_update`, `ai_wayfind_plan_directed_route_v3` | valid only with flag `0x10` |
| +0x14 | connection_idx | H | `ai_loc_update` | - | the loop index of the corridor walk over `node_a`'s connections, stored when `ai_path_is_pos_in_connection_corridor` accepts one; no access to `C_human+0x738` with a human base, so no reader (`.text scan`) |
| +0x18 | flags | H | `ai_loc_init`, `ai_loc_update` | `ai_loc_update`, `ai_wayfind_plan_directed_route_v3` | §2a |
| +0x1c | last_refresh_ms | H | `ai_loc_init`, `ai_loc_update` | `ai_loc_update` | - |
| +0x20 | stats_alloc | H | `human_ctor` | `ai_loc_add_or_refresh_seg_stat` | the allocator byte; `mov [esi+0x744],al`; the reader only takes its address for the vector helpers |
| +0x24 | stats_begin | H | `ai_loc_add_or_refresh_seg_stat`, `human_ctor`, `human_dtor` | `ai_loc_add_or_refresh_seg_stat`, `ai_loc_prune_stale_seg_stats`, `human_dtor` | - |
| +0x28 | stats_end | H | `ai_loc_add_or_refresh_seg_stat`, `ai_loc_prune_stale_seg_stats`, `human_ctor`, `human_dtor` | `ai_loc_add_or_refresh_seg_stat`, `ai_loc_prune_stale_seg_stats` | count = (end - begin) >> 4 |
| +0x2c | stats_cap | H | `ai_loc_add_or_refresh_seg_stat`, `human_ctor`, `human_dtor` | `ai_loc_add_or_refresh_seg_stat` | - |
| +0x30 | owner | H | `ai_loc_init` | `ai_loc_update` | typed `C_actor` by the signature, every caller is a `C_human` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | type | H | `ai_loc_add_or_refresh_seg_stat` | - | the special's `check_seg` type code 1 / 2 / 4 / 8, the first value `check_seg_get_special` returns; no reader: the vector has no user outside the class (`.text scan`, §4) |
| +0x4 | last_seen_ms | H | `ai_loc_add_or_refresh_seg_stat` | `ai_loc_prune_stale_seg_stats` | - |
| +0x8 | hit_count | H | `ai_loc_add_or_refresh_seg_stat` | `ai_loc_add_or_refresh_seg_stat` | starts at 1; only its own increment reads it |
| +0xc | payload | H | `ai_loc_add_or_refresh_seg_stat` | `ai_loc_add_or_refresh_seg_stat` | the record key |

### 2a. `flags` — what the last refresh found

`ai_loc_update` zeroes the word, then stores exactly one of `1`, `| 0x11` or `| 3`. The two-way split comes
from the status `ai_wayfind_get_node_for_position` returns (`structs/ai_wayfind.md` §4c): bit 0 set means
the position lies inside no node's radius, so the walk over the node's connections runs.

| Value | Meaning | Evidence |
|---|---|---|
| `0` | not refreshed yet, or the mission has no navigation web | `ai_loc_init`; `ai_loc_update`'s first exit |
| `1` | refreshed; the position is inside a node's radius, or no node was found at all (`node_a` = `0xffffffff`) | status bit 0 clear |
| `3` | refreshed; outside every node's radius and every connection corridor | status bit 0 set, the corridor walk found nothing |
| `0x11` | refreshed; inside a connection corridor, so `node_b` and `connection_idx` are valid | `ai_path_is_pos_in_connection_corridor` returned true |

Bit `0x01` is therefore "refreshed", the only bit the throttle tests. `ai_wayfind_plan_directed_route_v3`
reads bit `0x10` to choose between `node_b` and `node_a`.

## 3. Method family

4 of 4 PS2 methods mapped, all CONFIRMED by body diff. The unit's CRT pair and helpers stay with the host,
`structs/human.md`.

### 3a. Construction & destruction

None: `human_ctor` and `human_dtor` build and free the vector head in place.

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0048e7d0 | `ai_loc_init` | `Init__6ai_locFP7C_actor` | 0x00164d90 | CONFIRMED | binds the cache to its owner, seeds `cached_pos` from the owner's world frame, zeroes `flags` and `last_refresh_ms`, so the first `ai_loc_update` always refreshes; callers `human_game_init`, `human_activate`, `engine_new_mission` |
| 0x0048e810 | `ai_loc_update` | `Update__6ai_locFv` | 0x00164df0 | CONFIRMED | the throttled refresh (§4); sole caller `human_update` |
| 0x0048eb50 | `ai_loc_add_or_refresh_seg_stat` | `AddSegment__6ai_locFUiPv` | 0x00165260 | CONFIRMED | upsert keyed on `payload`: a hit refreshes `last_seen_ms` and bumps `hit_count`, a miss appends |
| 0x0048edb0 | `ai_loc_prune_stale_seg_stats` | `SegListUpdate__6ai_locFv` | 0x00165320 | CONFIRMED | reverse walk, erases every record whose `last_seen_ms` is over 4000 ms old |

### 3d. Free helpers

None: the vector helpers its methods call lie outside the unit.

## 4. What `ai_loc_update` does

The body is a chain of cheap exits before any spatial work:

1. **Gate:** runs only while the owner's `+0x98` is 0. `entity+0x98` is the actor's **current vehicle**
   (`ai_fight_followship_add_car_member` reads it and tests the result's `kind == 4`, car), so the cache
   refreshes **only while the character is on foot**.
2. **Bail:** if the mission has no navigation web (`*(g_act_mission+0x2c) + 0x0c == 0`), clear `flags` and
   return.
3. **Throttle:** take the owner frame's origin. Refresh only when bit `0x01` of `flags` is clear, **or** when
   the owner has moved at least 2 units (4.0 squared) from `cached_pos` **and** 5000 ms have elapsed since
   `last_refresh_ms`. Both conditions must hold: a `jb` at `0x0048e8c8` skips the refresh on a young stamp
   after the distance test has passed. The mission clock is `*(g_act_mission+0x24) + 0x2b0c`.
4. **Resolve:** `ai_wayfind_get_node_for_position` writes `node_a` and a status word; on status bit 0 walk
   the node's connections with `ai_path_is_pos_in_connection_corridor` (§2a).
5. **Rebuild the specials list:** for every `check_seg` segment whose node-index list contains `node_a`, or
   `node_b` under flag `0x10` (`check_seg_has_index`), test each special with
   `check_seg_is_pos_in_any_special` and pass the survivors to `ai_loc_add_or_refresh_seg_stat`.
6. **Expire:** `ai_loc_prune_stale_seg_stats`.

The class is found through the consumers of `check_seg_get_special`: there are seven, six of them
discriminate on the special's type code (`== 1` or `== 2`), and step 5 records all four types
indiscriminately. It is therefore the one place where a type-4 or type-8 special is *recorded*, and
**nothing reads that list**: the census of §2 shows the vector maintained by `ai_loc`'s own methods and
consumed by nothing.

## 5. PC vs PS2

- **Object size: PC `0x34` / PS2 `0x34`** — the same, but the fields are not at the same offsets, because
  the head vector is 3 floats on PC and 4 on PS2:

  | field | PC | PS2 |
  |---|---|---|
  | cached position | `+0x00` (3 floats) | `+0x00` (4 floats) |
  | node_a / node_b | `+0x0c` / `+0x10` | `+0x10` / `+0x14` |
  | connection idx | `+0x14` | `+0x18` |
  | flags | `+0x18` | `+0x1c` |
  | last refresh ms | `+0x1c` | `+0x20` |
  | stats vector | `+0x24` (MSVC `std::vector`, allocator byte at `+0x20`) | `+0x24` (`std::__vector_imp`, count at `+0x28`, data at `+0x2c`) |
  | owner | `+0x30` | `+0x30` |

  The head band is shifted by exactly one float and the tail band has zero delta, the same band-wise
  pattern as `check_seg`. Rebasing a single offset across builds produces nonsense; use the table.
- **Owner offset: PC `C_human+0x724` ↔ PS2 `+0x780`** (delta `0x5c`); PS2 reads it from
  `Init__6ai_locFP7C_actor(iVar2 + 0x780, iVar2)`.
- **Current vehicle:** PC `entity+0x98` ↔ PS2 `entity+0x224`. **Frame field** in `Init`: PC `+0x68` ↔ PS2
  `+0x70`. **Node arithmetic:** PC `/0x1e` ↔ PS2 `>>5`. **Mission clock:** PC
  `*(g_act_mission+0x24) + 0x2b0c` ↔ PS2 `*(act_mission+0x20) + 0xe54`.
- **The timing constants are identical**: 5000 ms refresh, 4000 ms expiry, the 4.0 squared move threshold.
  The two composite flag writes (`| 3`, `| 0x11`) are byte-identical and pin `ai_loc_update` to its twin.
- **`ai_seg_stat` is `0x10` bytes on both builds** with its four fields at the same offsets. Its name comes
  from the PS2 template argument `insert__Q23std62__vector_imp<>FP11ai_seg_statUi…`; on the PS2 the symbol
  occurs only in its `insert` / `erase` instantiations, so the list has no reader there either.
- **The PS2 method names overstate two bodies.** `AddSegment` adds no segment: its element is an observation
  record about one special, and the common path is a refresh. `SegListUpdate` only removes expired entries.
- **Twin evidence.** `Init`: same owner store, same frame field, same `& 0x20` guard + `UpdateWMatrixProc`,
  same two zeroed fields. `Update`: shared 5000 and 4.0, identical flag ors, identical node arithmetic at each
  build's stride. `AddSegment`: same key field, same refresh-or-insert split, same three written fields in
  order. `SegListUpdate`: same iteration direction, same timestamp field, same tail-of-`Update` call position.
- **Size ratios are a factoring difference, all in one direction:** PS2 calls `insert` / `erase` out of line
  where MSVC inlines the whole reallocate-and-shift, so the two PC functions that manipulate the vector are
  bigger than their twins (`ai_loc_add_or_refresh_seg_stat` 0.32, `ai_loc_prune_stale_seg_stats` 0.70) while
  the two that do not are in band (`ai_loc_init` 88 / 64 = 1.375, `ai_loc_update` 1136 / 832 = 1.37).
- **No PS2-only methods**, and no PC-only functions in the cluster.

## 6. Related

- `structs/check_seg.md` — the specials this class records; §1f the `ai_seg_corner` layout.
- `structs/ai_blob_log.md` — the adjacent sub-object at `C_human+0x758`.
- `structs/web_node.md` — what `node_a` / `node_b` index into (stride `0x1e`).
- `structs/ai_path.md` — `ai_path_is_pos_in_connection_corridor`, the corridor test.
- `structs/human.md` — the owner class and its sub-object offsets.

## 7. Open items

None.
