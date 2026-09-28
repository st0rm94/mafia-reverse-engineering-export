# `C_web_path` — the navigation web (the game's road/path graph)

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-09-28 19:12 +0200

Static analysis of `Game-ghidra.exe.annotated.c`, cross-checked against the non-stripped PS2 ELF
(`SLUS_206.71`). This documents the PC build; the PS2 symbol table supplies the original developer
names, and all cross-build material is confined to §5.

`C_web_path` is the **bottom layer** of Mafia's navigation stack: a flat graph of `C_web_node`
positions joined by `C_web_connection` edges, loaded from `check.bin`, plus the route searches over
it. It is what `ai_wayfind` and `ai_path` (the layer above — `structs/ai_wayfind.md`,
`structs/ai_path.md`) consume.

| | |
|---|---|
| PC prefix | `web_path_*`; nodes are `web_node_*` (`structs/web_node.md`) |
| Singleton | `*(g_act_mission + 0x2c)` |
| PS2 methods | 33 — **24 mapped**, all CONFIRMED by body-diff (§3) |
| PC cluster | `[0x006081e0, 0x0060b710]`, 43 aligned function starts |

**What the graph is, and is not:**

- **One graph tier.** The PS2 symbol table has exactly one graph class — this one — over
  `C_web_node` / `C_web_connection`. There is no sector tier and no sector class. The
  long-distance branch diverts to a **ROADS** route (`web_path_find_path_by_roads`), not to a
  second graph.
- **The root is `g_act_mission+0x2c`** (194 occurrences in the decomp); `g_display+0x2c` occurs
  zero times.
- **The 500-unit threshold** (`250000.0`) and the **structure at `*(g_act_mission+0x30)`**
  consulted alongside it are real. That structure is the **road network** (§7).
- **`0x40` on a connection is the editor SELECTED bit** (§2).

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Container layout (PC offsets from the `C_web_path` base)

```c
typedef struct C_web_path {          // singleton at *(g_act_mission + 0x2c)
    /* +0x00 */ uint32_t   _unk_00;  // [U]
    /* +0x04 */ C_web_node *nodes;   // [H] node array base. Record stride 0x1e.
    /* +0x08 */ uint32_t   _unk_08;  // [U]
    /* +0x0c */ uint32_t   node_count; // [H] every scan bounds on this
    /* ... */
    /* +0x14 */ void      *search_scratch;   // [M] parallel per-node search state, 6 bytes/node:
                                             //     ushort predecessor + float cost. Seeded to
                                             //     0xffff / 0x501502f9 (~1e10) before each search.
    /* +0x18 */ uint32_t   scratch_capacity; // [M] reallocated when != node_count
    /* +0x1c */ uint8_t    search_in_progress; // [H] re-entry guard; set around the roads diversion
                                               //     in web_path_find_path, cleared after
} C_web_path;
```

`+0x14`/`+0x18`/`+0x1c` are graded from the PC search bodies only. `web_path_find_path` inlines its
scratch handling and calls no helper.

## 2. `C_web_connection` — the edge record (8 bytes)

Stored per node, not centrally: the array pointer is at `node+0x18` and the count at `node+0x1d`
(see `structs/web_node.md`).

| Offset | Type | Purpose | Conf |
|---|---|---|---|
| `+0x00` | `ushort` | neighbour node **ID** (not an index — resolve with `web_path_find_node_id`) | H |
| `+0x02` | `ushort` | flag word — see the bit map below | H |
| `+0x04` | `float` | traversal cost = the **euclidean distance** between the two node positions | H |

### Connection flag bits (`+0x02`)

| Bit | Meaning | Evidence |
|---|---|---|
| `0x40` | **SELECTED** — the map editor's current selection | H — `web_path_find_selected_connection` scans for exactly this bit and returns the owning node + neighbour; pairs with PS2 `SelectLine` / `ReleaseSelect` |
| `0x80` | **ONE-WAY** — this direction is not traversable | H — `web_path_set_connection_one_way` is the only writer: it clears `0x80` on both directions, or flips which direction carries it |

Other bits are unidentified. Every `web_path_find_*` takes a trailing `ushort` **filter mask** tested
against this word.

⚠️ **Do not confuse this `0x40` with the node-level bit.** There are two, on two different
records: this one is `C_web_connection+0x02` bit `0x40` (the map editor's selection); the other is
`C_web_node+0x0c` bit `0x4000`, a visited-marker the endpoint resolvers set and clear around each
search (`structs/web_node.md` §1a). In the PC decomp they look nearly identical — `(x + 3) |= 0x40`
is the **connection**, `(x + 0xd) |= 0x40` is the **node**.

## 3. Method family — PC ↔ PS2 (24 of 33 mapped, all body-diffed)

Ordered by **PC VA**, PC name first.

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| 0x006085d0 | `web_node_find_connection_index` | `FindConnIdx__10C_web_nodeFUs` | CONFIRMED | ⚠️ a **`C_web_node`** method — see `structs/web_node.md`. Listed here because both its callers are `web_path_*` |
| 0x00608610 | `web_path_dtor` | `{dtor}__10C_web_path` | CONFIRMED | Vtable `g_web_path_vt`, destroys the stride-0x1e node array, frees the `+0x14` search scratch. Ratio 1.50 |
| 0x00608650 | `web_path_resize_node_array` | `Resize__10C_web_pathFUi` | CONFIRMED | Grows capacity in `+0x10` increments; `operator_new(n*0x1e + 4)`, copies, per-node connection resize. Live count `+0x0c` untouched. **Ratio 1.01 — the tightest match in the family** |
| 0x006087e0 | `web_path_add_node` | `AddNode__10C_web_pathFfffUs` | CONFIRMED | Appends a node at x/y/z with a category word; zeroes id `+0x0e` and conn count `+0x1d`. Exact signature + callee multiset (`Resize` ×1). Ratio 1.71 ¹ |
| 0x00608840 | `web_path_delete_node` | `DeleteNode__10C_web_pathFUi` | CONFIRMED | Strips inbound connections, then re-points every reference to the moved last node into the freed slot. Callee multiset identical: `DeleteConn`×1 + `FindConnIdx`×2. Ratio 1.55 |
| 0x00608980 | `web_path_toggle_connection` | `CreateConnect__10C_web_pathFUiUi` | CONFIRMED | Editor **toggle** — deletes the A–B connection both ways if present, else validates and adds it both ways. Callee multiset identical: `AddConn`×2 + `DeleteConn`×2 + `LegalConnect`×1. Ratio 1.19. Named for behaviour; the PS2 name says only "create" |
| 0x00608b00 | `web_path_is_connection_legal` | `LegalConnect__10C_web_pathFUiUiRUs` | CONFIRMED | Legality predicate for joining two nodes; reports the connection kind (1 / 2 / 4 / 0x10) through a `ushort&`, 0 on reject ² |
| 0x00608bc0 | `web_path_validate_node_connections` | `ControlNodeConn__10C_web_pathFUi` | CONFIRMED | Re-validates one node's connections: illegal ⇒ delete both ways, legal ⇒ rewrite the kind preserving ONE-WAY `0x80`. Callee multiset identical: `DeleteConn`×2 + `FindConnIdx`×3 + `LegalConnect`×1. Ratio 1.38 |
| 0x00608cf0 | `web_path_load_nodes` | `LoadNodes__10C_web_pathFP7C_chunk` | CONFIRMED | Reads the node array from an open `C_chunk`. Same destroy-and-zero, same two-condition chunk guard, shared callee `dtaSeek(h,0,1)`, same `<4` bail, then `dtaRead(h,&magic,4)`. **Node-chunk magic `0x1abcedf`** |
| 0x00608f70 | `web_path_merge_nodes` | `MergeNodes__10C_web_pathFP7C_chunk` | CONFIRMED | Merges a second node chunk into the live graph: resize to old+new, append, then **add the old count to every incoming neighbour id** ³. Ratio 1.26 |
| 0x00609210 | `web_path_recompute_connection_costs` | `RecompileDistance__10C_web_pathFUi` | CONFIRMED | Recomputes each connection's distance into `conn+0x04` and mirrors it onto the reverse connection. Ratio 1.47 |
| 0x00609340 | `web_path_clear_all_selections` | `ReleaseSelect__10C_web_pathFv` | CONFIRMED | Clears connection SELECTED bit `0x40` graph-wide, preserving ONE-WAY `0x80`. PC `&0xbf` on the byte = PS2 `&0xffbf` on the ushort. Ratio 1.40 |
| 0x00609390 | `web_path_find_node_id` | `FindNodeID__10C_web_pathFUs` | CONFIRMED | **Order-align anchor A.** Linear scan for `node+0x0e == id`; `0xffffffff` miss |
| 0x006093d0 | `web_path_find_path_by_roads` | `FindPath_FastByRoads__10C_web_path…Us` | CONFIRMED | The long-distance route. Callee-harvest — called from the identical guarded position on both builds |
| 0x00609970 | `web_path_find_path` | `FindPath__10C_web_path…Us` | CONFIRMED | The general node-to-node search. **Shared magic `250000.0`** + the `g_act_mission+0x30` co-condition + the same guard/call/guard triple around the roads variant |
| 0x00609f30 | `web_path_find_path_with_cost_fn` | `FindPathCompFunc__10C_web_pathFUiUi…` | CONFIRMED | End-node overload; caller supplies `float cost_fn(ushort,ushort,uchar)`. Same guard ladder order; **no** roads branch |
| 0x0060a480 | `web_path_find_path_to_goal_fn` | `FindPathCompFunc__10C_web_pathFUiPFUs_i…` | CONFIRMED | Goal-**predicate** overload. Separated from its identical sibling by two null-pointer guards in the same order and **no end-node range test** |
| 0x0060a9d0 | `web_path_find_selected_connection` | `FindSelectedLine__10C_web_pathFRUiRUi` | CONFIRMED | Editor query for the `0x40` bit; two `uint&` out-params, `0xffffffff/0xffffffff` miss |
| 0x0060aa50 | `web_path_insert_node_in_connection` | `IncludeNode__10C_web_pathFUiUi` | CONFIRMED | Editor op — splits A–B by inserting a node at the **midpoint**, then deletes the original pair. Statement-for-statement match |
| 0x0060ad00 | `web_path_set_connection_one_way` | `FlipOneWay__10C_web_pathFUiUib` | CONFIRMED | Edits `0x80` on both directions; `clear` arm vs `flip` arm |
| 0x0060add0 | `web_path_find_nearest_node` | `GetNearNode__10C_web_pathFRC8S_vectorfUs` | CONFIRMED | **Order-align anchor B.** Shared magic `r*r + 0.001`; binary search on X + two-way sweep |
| 0x0060afd0 | `web_path_sort_nodes_by_x` | `SortNodesByX__10C_web_pathFv` | CONFIRMED | Selection sort behind an is-sorted early-out. This ordering is what makes anchor B's binary search valid |
| 0x0060b120 | `web_path_is_sorted_by_x` | `IsSorted__10C_web_pathFv` | CONFIRMED | Callee-harvest from the row above; its only caller |
| 0x0060b150 | `web_path_clear_marker_bits` | `DeRiciConns__10C_web_pathFv` | CONFIRMED | Clears search marker bits `0x1000`/`0x4000` from every node category word and every connection flag word, preserving ONE-WAY `0x80` and SELECTED `0x40` ⁴. Ratio 1.80 ¹ |
| 0x0060b680 | `web_path_find_path_jipo` | `FindPathCompFuncJIPO__10C_web_pathFUiPFUs_i…` | CONFIRMED | Search variant used by `ai_wayfind_plan_directed_route_v3`. 🔑 **Shared magic float `0x4922c2a0` = 666666.0** seeded as the cost cap on both builds, plus statement-identical flow. Ratio 1.36 |

¹ Above the usual ratio band — PS2 `S_vector&` copying.
² Near-byte-identical — same `0x13` mask, same branch order, same out-values; MSVC merges PS2's
`&4`/`&8` into `&0xc`. **Proves `C_web_node+0x0c` is the node category word.**
³ Reads the SAME magic `0x1abcedf` as `web_path_load_nodes`; separated by that rebase loop and by
the absence of the destroy-and-zero preamble.
⁴ Statement-identical backwards double loop; PS2 `&0xafff` on the ushort = PC `&0xaf` on its high
byte.

**Order-align check:** across the region the 15 confirmed anchors come out **monotone** (PS2 ranks
1→28 against PC indices 9→24). The region is not a forced gap, so every row was established by
behaviour plus an exact callee multiset, never by rank interpolation.

**Unmapped PS2 methods (10):** `AllocHelpMem`, `FreeHelpMem`, `GetNextHlpMemBlock`, `ClearNode`,
`ClearConn` *(a `C_web_node` method)*, `SelectLine`, `DrawConnections`, `SaveNodes`, `DebugFindPath`,
and the `C_web_path` **ctor**. §5 covers each.

### 3b. False function starts in this cluster

The aggressive instruction finder created two false function starts at mid-body branch
targets. Neither is in Ghidra's default analysis or the annotated decomp.

| Fake start | Really is | Proof |
|---|---|---|
| `0x0060aa25` | the match-found exit path of **`web_path_find_selected_connection`** (`0x0060a9d0`–`0x0060aa50`) | target of `jne 0x60aa25` at `0x0060a9f8`; unaligned (`%16 = 5`); no NOP padding; preceded by `ret 8` |
| `0x0060adb3` | the set-ONE-WAY arm of **`web_path_set_connection_one_way`** (`0x0060ad00`–`0x0060add0`) | target of `jns 0x60adb3` at `0x0060ad91`; unaligned (`%16 = 3`); no NOP padding; preceded by `ret 0xc` |

Both bodies read live registers they never receive as parameters (`unaff_EBX`, `in_stack_*`,
`in_EAX`, `unaff_ESI`). They are proven from the disassembly of `Game.exe`; a search of the decomp
text for split-body markers does not flag either shape.

## 4. `check.bin` — the node chunk

The node chunk is tagged `0x1abcedf`. Two readers use the **same** chunk layout:
`web_path_load_nodes` (destroy-and-replace) and `web_path_merge_nodes` (append + neighbour-id
rebase). Record order: magic, node count, node records (stride `0x1e`), then per-node connection
arrays (stride 8). The rest of the file format is undocumented (§7).

## 5. PC vs PS2

- **Node record stride: PC `0x1e`, PS2 `0x20`.** Every *field offset* inside the record is
  identical — the difference is a trailing alignment pad only, so field-level findings transfer
  directly. `C_web_connection` has **zero delta** (8 bytes on both).
- **Container header differs.** PC keeps `nodes` at `+0x04` and `node_count` at `+0x0c`; PS2 keeps
  them at `+0x00` and `+0x08`. PS2's search-in-progress guard is at `+0x2c` (PC `+0x1c`).
- **Shared magic constants that carried the twins:** `250000.0` (500² — the roads threshold in
  `FindPath`, byte-identical), `r*r + 0.001` (the radius guard in `GetNearNode`), `0x1abcedf` (the
  node-chunk magic in `LoadNodes`), `0xffff` (the connection-index miss sentinel).
- **What the PS2 signatures type:** the route output is a `std::vector<uint>` of **node indices**
  (`PQ23std32vector<Ui,…>`); the trailing `Us` on every `Find*` is the **filter mask**; `LoadNodes`
  takes a **`C_chunk*`**; the cost callback is `float(*)(ushort, ushort, uchar)` and the goal
  predicate is `int(*)(ushort)`. None of that is visible from the PC decomp.
- **PS2 names the array element type** inside `LoadNodes`' teardown
  (`__destroy_new_array(..., C_web_node::~C_web_node)`), independently confirming it.
- 🔴 **The `*HlpMem` trio is PS2-only — an arena allocator the PC build does not have** (graded
  HIGH: an allocation-strategy inference from callsites, not a body diff of the trio). The PS2
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
- **`ClearNode`, `ClearConn` and the `C_web_path` ctor** are ordinary methods (`ClearNode` calls
  `ClearConn` per node) and are not in this PC cluster; they may have PC twins elsewhere (§7).
- **`SaveNodes` (284 B) has no PC counterpart anywhere in the region** — every other editing
  method is present, and `SaveNodes` is the one that *writes* `check.bin` (§7).
- **`SelectLine` (8 B) and `DrawConnections` (8 B)** are stubs on PS2, almost certainly inlined or
  compiled out on PC; **`DebugFindPath`** is a debug entry point. None is worth hunting.

## 6. Cross-references

- `structs/web_node.md` — the node record and its connection list.
- `structs/ai_wayfind.md` / `structs/ai_path.md` — the layer above this one.

## 7. Open items

| item | state |
|---|---|
| the road structure at `*(g_act_mission+0x30)` | real, present on both builds, gated by the 500-unit threshold, undocumented anywhere; PC reads it with a stride of `0xec`; `web_path_find_path_by_roads` is the way in |
| the `+0x14` search scratch | graded [M] from PC only; confirm against PS2 |
| remaining connection flag bits | unidentified beyond `0x40` / `0x80` |
| `*HlpMem` trio PS2-only grade | identify the single PS2 caller of `AllocHelpMem`; if it belongs to level load/teardown, the grade is settled |
| `ClearNode` / `ClearConn` / `C_web_path` ctor | not located on PC |
| `SaveNodes` | plausibly absent because retail ships without the map editor's save path — a hypothesis, not graded PS2-only |
| rest of `check.bin` | only the node chunk is documented (§4) |
