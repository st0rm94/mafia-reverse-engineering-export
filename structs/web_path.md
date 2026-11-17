# `C_web_path` — the navigation web (the game's road and path graph)

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-11-09 19:47 +0100

`C_web_path` is the **bottom layer** of the navigation stack: a flat graph of `C_web_node` positions joined by `C_web_connection`
edges, loaded from `check.bin`, plus the route searches over it. `ai_wayfind` and `ai_path` consume it (`structs/ai_wayfind.md`,
`structs/ai_path.md`).

* **One graph tier** over `C_web_node` / `C_web_connection`. There is no sector tier and no sector class. The long-distance branch
  diverts to a **roads** route (`web_path_find_path_by_roads`), not to a second graph.
* **The root is `g_act_mission+0x2c`** (194 occurrences in the decomp).
* **The 500-unit threshold** (`250000.0`) gates the roads diversion together with the structure at `*(g_act_mission+0x30)`, the
  `C_roads` singleton, whose `+0x08` table is the stride-`0xec` crossing array (`structs/roads.md`).

| | |
|---|---|
| PC prefix | `web_path_`; nodes are `web_node_` (`structs/web_node.md`) |
| PC kind (`+0x10`) | - |
| PC vtable | `g_web_path_vt` @ `0x0063edf8`, 14 slots (§5) |
| PC base class | - |
| PC instance size | `0x1d` (`operator_new` in `mission_init`) |
| PC constructor | inlined into `mission_init` |
| PC translation unit | `[0x00608390, 0x0060b740)`, guard to guard, shared with `C_web_node`; 35 function starts from `0x006083c0` |
| PS2 class | `C_web_path` (tag `10C_web_path`), 33 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_web_path {           // 0x1d bytes, singleton at *(g_act_mission + 0x2c)
    /* +0x00 */ void       *vtable;       // [H] g_web_path_vt, set by mission_init and web_path_dtor
    /* +0x04 */ C_web_node *nodes;        // [H] the node array; record stride 0x1e
    /* +0x08 */ uint32_t   node_capacity; // [H] records the array holds; web_path_resize_node_array grows it
    /* +0x0c */ uint32_t   node_count;    // [H] live records; every scan bounds on this
    /* +0x10 */ uint32_t   grow_step;     // [H] capacity increment; the constructor seeds 0x32
    /* +0x14 */ C_web_path::S_pathfind *search_scratch; // [H] one 6-byte record per node (§1b)
    /* +0x18 */ uint32_t   scratch_capacity; // [H] records in search_scratch; reallocated when != node_count
    /* +0x1c */ uint8_t    search_in_progress; // [H] set around the roads diversion in web_path_find_path
} C_web_path;
```

The constructor is inlined into `mission_init` (`operator_new(0x1d)`, the vtable, `+0x10` = `0x32`, every other field zeroed), and
`mission_close` resets the graph in place: the count `+0x0c` to 0, the scratch `+0x14` freed and zeroed, `+0x18` and the guard
`+0x1c` cleared. The node and connection arrays are freed by `web_path_dtor` and by `web_path_load_nodes`' destroy-and-zero preamble.

### 1a. `C_web_connection`

```c
typedef struct C_web_connection {     // 8 bytes, stored per node (structs/web_node.md +0x18, count +0x1d)
    /* +0x00 */ uint16_t neighbour_id;    // [H] the neighbour's node id, not an index: web_path_find_node_id resolves it
    /* +0x02 */ uint16_t flags;           // [H] the bit map of §2d
    /* +0x04 */ float    cost;            // [H] the euclidean distance between the two node positions
} C_web_connection;
```

### 1b. `C_web_path::S_pathfind`

```c
typedef struct S_pathfind {           // 6 bytes, one per node, in search_scratch
    /* +0x00 */ uint16_t predecessor;     // [H] the node this one was reached from; seeded 0xffff
    /* +0x02 */ float    path_cost;       // [H] cost so far; seeded 0x501502f9, about 1e10
} S_pathfind;
```

## 2. Field evidence

### 2a. `C_web_path`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vtable | H | `mission_init`, `web_path_dtor` | `web_path_insert_node_in_connection` | its call through slot 0 adds the midpoint node (§5) |
| +0x4 | nodes | H | `web_path_load_nodes`, `web_path_resize_node_array` | `web_path_add_node`, `web_path_clear_all_selections`, `web_path_clear_marker_bits`, `web_path_delete_node`, `web_path_dtor`, `web_path_find_nearest_node`, `web_path_find_node_id`, `web_path_find_path`, `web_path_find_path_by_roads`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn`, `web_path_find_selected_connection`, `web_path_insert_node_in_connection`, `web_path_is_connection_legal`, `web_path_is_sorted_by_x`, `web_path_load_nodes`, `web_path_merge_nodes`, `web_path_recompute_connection_costs`, `web_path_resize_node_array`, `web_path_set_connection_one_way`, `web_path_sort_nodes_by_x`, `web_path_toggle_connection`, `web_path_validate_node_connections` | Many outside readers through `g_act_mission+0x2c` |
| +0x8 | node_capacity | H | `web_path_load_nodes`, `web_path_resize_node_array` | `web_path_resize_node_array` | - |
| +0xc | node_count | H | `mission_close`, `web_path_add_node`, `web_path_delete_node`, `web_path_load_nodes`, `web_path_merge_nodes` | `web_path_add_node`, `web_path_clear_all_selections`, `web_path_clear_marker_bits`, `web_path_delete_node`, `web_path_find_nearest_node`, `web_path_find_node_id`, `web_path_find_path`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn`, `web_path_find_selected_connection`, `web_path_insert_node_in_connection`, `web_path_is_sorted_by_x`, `web_path_load_nodes`, `web_path_merge_nodes`, `web_path_resize_node_array`, `web_path_sort_nodes_by_x` | Many outside readers through `g_act_mission+0x2c` |
| +0x10 | grow_step | H | `mission_init` | `web_path_resize_node_array` | - |
| +0x14 | search_scratch | H | `mission_close`, `web_path_find_path`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn` | `mission_close`, `web_path_dtor`, `web_path_find_path`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn` | - |
| +0x18 | scratch_capacity | H | `mission_close`, `web_path_find_path`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn` | `web_path_find_path`, `web_path_find_path_to_goal_fn`, `web_path_find_path_with_cost_fn` | - |
| +0x1c | search_in_progress | H | `mission_close`, `web_path_find_path` | `web_path_find_path` | - |

### 2b. `C_web_connection`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | neighbour_id | H | `web_node_add_connection`, `web_path_load_nodes`, `web_path_merge_nodes` | `web_node_find_connection_index`, `web_path_find_path` | `web_path_merge_nodes` adds the old node count to every incoming id |
| +0x2 | flags | H | `web_path_validate_node_connections`, `web_path_set_connection_one_way`, `web_path_clear_all_selections`, `web_path_clear_marker_bits` | `web_path_find_selected_connection`, `web_path_find_path` | §2d |
| +0x4 | cost | H | `web_path_recompute_connection_costs`, `web_path_load_nodes` | `web_path_find_path` | - |

### 2c. `S_pathfind`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | predecessor | H | `web_path_find_path`, `web_path_find_path_with_cost_fn`, `web_path_find_path_to_goal_fn` | the same three | - |
| +0x2 | path_cost | H | `web_path_find_path`, `web_path_find_path_with_cost_fn`, `web_path_find_path_to_goal_fn` | the same three | - |

### 2d. `C_web_connection.flags` — the bit map

| Value | Meaning | Evidence |
|---|---|---|
| `0x01` / `0x02` / `0x04` / `0x10` | the connection **kind**, exactly one per connection | the value `web_path_is_connection_legal` reports and `web_path_validate_node_connections` rewrites |
| `0x40` | **selected**: the map editor's current selection | `web_path_find_selected_connection` scans for exactly this bit and returns the owning node and neighbour |
| `0x80` | **one-way**: this direction is not traversable | `web_path_set_connection_one_way` is the only writer: it clears `0x80` on both directions, or flips which direction carries it |

In the 71 shipped `check.bin` files (2 405 202 connections) the word uses only these bits: kind `0x01` 435 394, `0x02` 1 875 268,
`0x04` 92 174 and `0x10` 2 366; `0x80` on 45 376 connections and `0x40` on 26, editor selections saved with the map. The search
markers `0x1000` / `0x4000` that `web_path_clear_marker_bits` clears never occur in a shipped file. Every `web_path_find_*` takes a
trailing `ushort` **filter mask**.

**Do not confuse this `0x40` with the node-level bit.** There are two, on two records: this one is `C_web_connection+0x02` bit
`0x40`, the editor's selection; the other is `C_web_node+0x0c` bit `0x4000`, a visited-marker the endpoint resolvers set and clear
around each search (`structs/web_node.md` §2a). In the decomp they look nearly identical: `(x + 3) |= 0x40` is the connection,
`(x + 0xd) |= 0x40` is the node.

## 3. Method family

27 of the 33 `C_web_path` PS2 methods are mapped: 24 own bodies, all body-diffed, and 3 vtable slots that fold onto shared stubs
(§5). The other 6 are in §7a. `web_node_find_connection_index` is a `C_web_node` method both of whose callers are `web_path_*`
(`structs/web_node.md`).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00608610 | `web_path_dtor` | instance | `__dt__10C_web_pathFv` | 0x00390dd0 | CONFIRMED | Vtable `g_web_path_vt`, destroys the stride-0x1e node array, frees the `+0x14` search scratch |

### 3b. Accessors

The class has no one-load or one-store method.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00574a40 | `stub_noop_cdecl_00574a40` | `DrawConnections__10C_web_pathFv` | 0x003912b0 | base-stub | Slot 3, `DrawConnections` on the PS2; a shared stub serving many vtables (§5) |
| 0x00575a60 | `stub_ret0_args8_00575a60` | `SelectLine__10C_web_pathFii` | 0x00391740 | base-stub | Slot 7, `SelectLine` on the PS2; a shared stub with two arguments (§5) |
| 0x00608650 | `web_path_resize_node_array` | `Resize__10C_web_pathFUi` | 0x00390e30 | CONFIRMED | Grows capacity in `+0x10` increments; `operator_new(n*0x1e + 4)`, copies, per-node connection resize. Live count `+0x0c` untouched |
| 0x006087e0 | `web_path_add_node` | `AddNode__10C_web_pathFfffUs` | 0x00391010 | CONFIRMED | Appends a node at x/y/z with a category word; zeroes id `+0x0e` and conn count `+0x1d`. Exact signature + callee multiset (`Resize` ×1) |
| 0x00608840 | `web_path_delete_node` | `DeleteNode__10C_web_pathFUi` | 0x003910c0 | CONFIRMED | Strips inbound connections, then re-points every reference to the moved last node into the freed slot. Callee multiset identical: `DeleteConn`×1 + `FindConnIdx`×2 |
| 0x00608980 | `web_path_toggle_connection` | `CreateConnect__10C_web_pathFUiUi` | 0x003912c0 | CONFIRMED | Editor **toggle** — deletes the A–B connection both ways if present, else validates and adds it both ways. Callee multiset identical: `AddConn`×2 + `DeleteConn`×2 + `LegalConnect`×1 |
| 0x00608b00 | `web_path_is_connection_legal` | `LegalConnect__10C_web_pathFUiUiRUs` | 0x00391490 | CONFIRMED | Legality predicate for joining two nodes; reports the connection kind (1 / 2 / 4 / 0x10) through a `ushort&`, 0 on reject |
| 0x00608bc0 | `web_path_validate_node_connections` | `ControlNodeConn__10C_web_pathFUi` | 0x00391590 | CONFIRMED | Re-validates one node's connections: illegal ⇒ delete both ways, legal ⇒ rewrite the kind preserving ONE-WAY `0x80`. Callee multiset identical: `DeleteConn`×2 + `FindConnIdx`×3 + `LegalConnect`×1 |
| 0x00608cf0 | `web_path_load_nodes` | `LoadNodes__10C_web_pathFP7C_chunk` | 0x00391870 | CONFIRMED | Reads the node array from an open `C_chunk`. Same destroy-and-zero, same two-condition chunk guard, shared callee `dtaSeek(h,0,1)`, same `<4` bail, then `dtaRead(h,&magic,4)`. **Node-chunk magic `0x1abcedf`** |
| 0x00608f70 | `web_path_merge_nodes` | `MergeNodes__10C_web_pathFP7C_chunk` | 0x00391ba0 | CONFIRMED | Merges a second node chunk into the live graph: resize to old+new, append, then **add the old count to every incoming neighbour id** ³ |
| 0x00609210 | `web_path_recompute_connection_costs` | `RecompileDistance__10C_web_pathFUi` | 0x00391ef0 | CONFIRMED | Recomputes each connection's distance into `conn+0x04` and mirrors it onto the reverse connection |
| 0x00609340 | `web_path_clear_all_selections` | `ReleaseSelect__10C_web_pathFv` | 0x003920b0 | CONFIRMED | Clears connection SELECTED bit `0x40` graph-wide, preserving ONE-WAY `0x80`. PC `&0xbf` on the byte = PS2 `&0xffbf` on the ushort |
| 0x00609390 | `web_path_find_node_id` | `FindNodeID__10C_web_pathFUs` | 0x00392120 | CONFIRMED | **Order-align anchor A.** Linear scan for `node+0x0e == id`; `0xffffffff` miss |
| 0x006093d0 | `web_path_find_path_by_roads` | `FindPath_FastByRoads__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>Us` | 0x00392180 | CONFIRMED | The long-distance route. Callee-harvest — called from the identical guarded position on both builds |
| 0x00609970 | `web_path_find_path` | `FindPath__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>Us` | 0x003927d0 | CONFIRMED | The general node-to-node search. **Shared magic `250000.0`** + the `g_act_mission+0x30` co-condition + the same guard/call/guard triple around the roads variant |
| 0x00609f30 | `web_path_find_path_with_cost_fn` | `FindPathCompFunc__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs` | 0x00392f00 | CONFIRMED | End-node overload; caller supplies `float cost_fn(ushort,ushort,uchar)`. Same guard ladder order; **no** roads branch |
| 0x0060a480 | `web_path_find_path_to_goal_fn` | `FindPathCompFunc__10C_web_pathFUiPFUs_iPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs` | 0x00393530 | CONFIRMED | Goal-**predicate** overload. Separated from its identical sibling by two null-pointer guards in the same order and **no end-node range test** |
| 0x0060a9d0 | `web_path_find_selected_connection` | `FindSelectedLine__10C_web_pathFRUiRUi` | 0x00393b80 | CONFIRMED | Editor query for the `0x40` bit; two `uint&` out-params, `0xffffffff/0xffffffff` miss |
| 0x0060aa50 | `web_path_insert_node_in_connection` | `IncludeNode__10C_web_pathFUiUi` | 0x00393c30 | CONFIRMED | Editor op — splits A–B by inserting a node at the **midpoint**, then deletes the original pair. Statement-for-statement match |
| 0x0060ad00 | `web_path_set_connection_one_way` | `FlipOneWay__10C_web_pathFUiUib` | 0x00393f60 | CONFIRMED | Edits `0x80` on both directions; `clear` arm vs `flip` arm |
| 0x0060add0 | `web_path_find_nearest_node` | `GetNearNode__10C_web_pathFRC8S_vectorfUs` | 0x003940f0 | CONFIRMED | **Order-align anchor B.** Shared magic `r*r + 0.001`; binary search on X + two-way sweep |
| 0x0060afd0 | `web_path_sort_nodes_by_x` | `SortNodesByX__10C_web_pathFv` | 0x00394360 | CONFIRMED | Selection sort behind an is-sorted early-out. This ordering is what makes anchor B's binary search valid |
| 0x0060b120 | `web_path_is_sorted_by_x` | `IsSorted__10C_web_pathFv` | 0x00394510 | CONFIRMED | Callee-harvest from the row above; its only caller |
| 0x0060b150 | `web_path_clear_marker_bits` | `DeRiciConns__10C_web_pathFv` | 0x00394580 | CONFIRMED | Clears search marker bits `0x1000`/`0x4000` from every node category word and every connection flag word, preserving ONE-WAY `0x80` and SELECTED `0x40` ⁴ |
| 0x0060b1a0 | `stub_noop_args8_0060b1a0` | `DebugFindPath__10C_web_pathFUsUs` | 0x00394610 | base-stub | Slot 4, `DebugFindPath` on the PS2; a shared stub with two arguments (§5) |
| 0x0060b680 | `web_path_find_path_jipo` | `FindPathCompFuncJIPO__10C_web_pathFUiPFUs_iPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs` | 0x00394a60 | CONFIRMED | Search variant used by `ai_wayfind_plan_directed_route_v3`. **Shared magic float `0x4922c2a0` = 666666.0** seeded as the cost cap on both builds, plus statement-identical flow |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0060b1b0 | `vector_uint_append_all` | `_STDVECTCOPY__FPQ23std32vector<Ui,Q23std13allocator<Ui>>PQ23std32vector<Ui,Q23std13allocator<Ui>>` | 0x00394620 | CONFIRMED | Appends one `vector<uint>` to another; the route output of the searches |
| 0x0060b370 | `web_graph_dfs_expand` | `nodeproc__FUiUiPFUs_iPFUsUsUc_fPff` | 0x003946c0 | CONFIRMED | The depth-first node expansion `web_path_find_path_jipo` runs |
| 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | - | - | - | The no-op destructor nearly every `crt_atexit` registrar passes; it sits at the end of this unit |

Notes on the rows:

* `web_path_is_connection_legal` is near-byte-identical with the PS2 body: same `0x13` mask, same branch order, same out-values; MSVC
  merges the PS2's `&4` / `&8` into `&0xc`. That proves `C_web_node+0x0c` is the node category word.
* `web_path_merge_nodes` reads the same magic `0x1abcedf` as `web_path_load_nodes`; the rebase loop and the absent destroy-and-zero
  preamble separate them.
* `web_path_clear_marker_bits` is a statement-identical backwards double loop; the PS2's `&0xafff` on the ushort is the PC's `&0xaf`
  on its high byte.

**Order-align check:** across the region the 15 confirmed anchors come out monotone (PS2 ranks 1 to 28 against PC indices 1 to 16,
counted from `0x006083c0`). The region is not a forced gap, so every row was established by behaviour plus an exact callee multiset,
never by rank interpolation.

## 4. `check.bin` — the node chunk

The node chunk is tagged `0x1abcedf`, and a shipped `check.bin` opens with that magic directly, with no chunk header. Two readers use
the **same** layout: `web_path_load_nodes` (destroy and replace) and `web_path_merge_nodes` (append and neighbour-id rebase). Record
order: magic, node count, node records (stride `0x1e`), then per-node connection arrays (stride 8, `conn_count` records per node).
That is the whole file: all 71 shipped `check.bin` files end exactly after the last connection array.

## 5. Vtable

Dumped from `.rdata`; the slots are the PS2 table's in order, without its two null header slots and without `SaveNodes`.

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 0 | `0x006087e0` | `web_path_add_node` | `AddNode__10C_web_pathFfffUs` |
| 1 | `0x00608840` | `web_path_delete_node` | `DeleteNode__10C_web_pathFUi` |
| 2 | `0x0060aa50` | `web_path_insert_node_in_connection` | `IncludeNode__10C_web_pathFUiUi` |
| 3 | `0x00574a40` | `stub_noop_cdecl_00574a40` | `DrawConnections__10C_web_pathFv` |
| 4 | `0x0060b1a0` | `stub_noop_args8_0060b1a0` | `DebugFindPath__10C_web_pathFUsUs` |
| 5 | `0x00608980` | `web_path_toggle_connection` | `CreateConnect__10C_web_pathFUiUi` |
| 6 | `0x00608bc0` | `web_path_validate_node_connections` | `ControlNodeConn__10C_web_pathFUi` |
| 7 | `0x00575a60` | `stub_ret0_args8_00575a60` | `SelectLine__10C_web_pathFii` |
| 8 | `0x00609340` | `web_path_clear_all_selections` | `ReleaseSelect__10C_web_pathFv` |
| 9 | `0x0060a9d0` | `web_path_find_selected_connection` | `FindSelectedLine__10C_web_pathFRUiRUi` |
| 10 | `0x0060afd0` | `web_path_sort_nodes_by_x` | `SortNodesByX__10C_web_pathFv` |
| 11 | `0x00608f70` | `web_path_merge_nodes` | `MergeNodes__10C_web_pathFP7C_chunk` |
| 12 | `0x00609210` | `web_path_recompute_connection_costs` | `RecompileDistance__10C_web_pathFUi` |
| 13 | `0x0060ad00` | `web_path_set_connection_one_way` | `FlipOneWay__10C_web_pathFUiUib` |

`web_path_insert_node_in_connection` calls slot 0 through the vtable to add the midpoint node.

## 6. False function starts in this cluster

The aggressive instruction finder created two false function starts at mid-body branch
targets. Neither is in Ghidra's default analysis or `Game-ghidra.exe.annotated.c`.

| Fake start | Really is | Proof |
|---|---|---|
| `0x0060aa25` | the match-found exit path of `web_path_find_selected_connection` | target of `jne 0x60aa25` at `0x0060a9f8`; unaligned (`%16 = 5`); no NOP padding; preceded by `ret 8` |
| `0x0060adb3` | the set-one-way arm of `web_path_set_connection_one_way` | target of `jns 0x60adb3` at `0x0060ad91`; unaligned (`%16 = 3`); no NOP padding; preceded by `ret 0xc` |

Both bodies read live registers they never receive as parameters (`unaff_EBX`, `in_stack_*`, `in_EAX`, `unaff_ESI`). They are proven
from the disassembly of `Game.exe`; a search of the decomp text for split-body markers does not flag either shape.

## 7. PC vs PS2

- **One graph class.** The PS2 symbol table has exactly one graph class — `C_web_path` — over
  `C_web_node` / `C_web_connection`, and no sector class.
- **Node record stride: PC `0x1e`, PS2 `0x20`.** Every *field offset* inside the record is
  identical — the difference is a trailing alignment pad only, so field-level findings transfer
  directly. `C_web_connection` has **zero delta** (8 bytes on both).
- **The search scratch is an array of `C_web_path::S_pathfind`.** The PS2 `FindPath` builds it with
  `__construct_new_array(…, __ct__Q210C_web_path10S_pathfindFv, 0, 8, count)`: 8 bytes there
  (`+0x00` predecessor, `+0x04` cost) against 6 on the PC (`+0x00` predecessor, `+0x02` cost) — the
  same 2-byte packing that makes the PC node stride `0x1e` against `0x20`. Both builds seed
  `0xffff` / `1e10`, reallocate when the capacity differs from the node count, and run the same
  relaxation over it.
- **Container header differs.** PC keeps `nodes` at `+0x04` and `node_count` at `+0x0c`; PS2 keeps
  them at `+0x00` and `+0x08`. PS2's search-in-progress guard is at `+0x2c` (PC `+0x1c`), and its
  scratch buffer pair at `+0x24` / `+0x28` (PC `+0x14` / `+0x18`): `ClearNode` frees and clears it
  exactly as PC `mission_close` does.
- **Shared magic constants that carried the twins:** `250000.0` (500² — the roads threshold in
  `FindPath`, byte-identical), `r*r + 0.001` (the radius guard in `GetNearNode`), `0x1abcedf` (the
  node-chunk magic in `LoadNodes`), `0xffff` (the connection-index miss sentinel).
- **What the PS2 signatures type:** the route output is a `std::vector<uint>` of **node indices**
  (`PQ23std32vector<Ui,…>`); the trailing `Us` on every `Find*` is the **filter mask**; `LoadNodes`
  takes a **`C_chunk*`**; the cost callback is `float(*)(ushort, ushort, uchar)` and the goal
  predicate is `int(*)(ushort)`. None of that is visible from the PC decomp.
- **PS2 names the array element type** inside `LoadNodes`' teardown
  (`__destroy_new_array(..., C_web_node::~C_web_node)`), independently confirming it.
- 🔴 **The `*HlpMem` trio is PS2-only — an arena allocator the PC build does not have.** The
  arena's lifecycle pair belongs to the PS2-only city streaming: `AllocHelpMem`'s only caller is
  `C_mission::InterCity_Prebuffering` and `FreeHelpMem`'s is `C_mission::InterCity_Done`. The PS2
  caller list:

  | PS2 site | What it does | PC equivalent |
  |---|---|---|
  | `Resize__10C_web_node` | `GetNextHlpMemBlock(capacity << 3)` — i.e. `capacity * 8` | `web_node_resize_connection_array` calls **`operator_new(capacity*8 + 4)`** |
  | `Resize__10C_web_path` | `GetNextHlpMemBlock(count << 5)` — i.e. `count * 0x20` | `web_path_resize_node_array` calls **`operator_new(count*0x1e + 4)`** |
  | one call each | `AllocHelpMem` / `FreeHelpMem` — the arena lifecycle pair | - |

  Both PS2 `Resize` methods route their allocation through the arena; **both PC `Resize` methods
  call the global `operator_new` at exactly the same point** (body-read on both). Same pattern as
  `ai_spageman` — memory infrastructure the 32 MB console needed. Do not hunt the trio for a PC
  twin. The PC `+0x14` search scratch is not managed by it.
- **`ClearNode`, `ClearConn` and the `C_web_path` ctor are inlined on the PC** (§7a). The ctor's two
  PS2 callers are `C_mission::Init` and the PS2-only `InterCity_Init`; PC `mission_init` spells the
  same construction inline (§1). `ClearNode` (called from `C_mission::Close` and the destructor)
  runs `ClearConn` per node and frees the arrays only when no helper-memory arena is active; PC
  `mission_close` carries its reset inline (§1), and the arrays are freed where §1 says.
  `ClearConn` is called only from `ClearNode`.
- **`DrawConnections` and `SelectLine` (8 B each)** are stubs on the PS2 and **`DebugFindPath`** a debug entry point; on the PC all
  three are slots folded onto shared stubs (§5).

### 7a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00390a20 | `AllocHelpMem__10C_web_pathFPcPc` | the arena's lifecycle: only caller `C_mission::InterCity_Prebuffering`, PS2-only city streaming |
| 0x00390d90 | `__ct__10C_web_pathFUi` | inlined into `mission_init` |
| 0x00390c40 | `FreeHelpMem__10C_web_pathFv` | the arena's lifecycle: only caller `C_mission::InterCity_Done` |
| 0x00390ca0 | `GetNextHlpMemBlock__10C_web_pathFi` | the arena allocator; at the same point the PC calls `operator_new` |
| 0x00390cd0 | `ClearNode__10C_web_pathFv` | inlined into `mission_close` |
| 0x00391750 | `SaveNodes__10C_web_pathFP7C_chunk` | absent: the one vtable slot the PC table lacks (§5). It is the method that would write `check.bin`, it has no caller on the PS2, and the PC holds the magic `0x1abcedf` only in the two readers of §4 |

## 8. Related

- `structs/web_node.md`: the node record and its connection list.
- `structs/ai_wayfind.md` / `structs/ai_path.md`: the layer above this one.
- `structs/roads.md`: the `C_roads` singleton at `*(g_act_mission+0x30)` that `web_path_find_path_by_roads` routes over.

## 9. Open items

Nothing is open.
