# `ai_wayfind` — the route planner (checkpoint-based wayfinding)

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-11-08 21:58 +0100

`ai_wayfind` is the **route planner**: a singleton that turns a movement request into an `ai_path`
(`structs/ai_path.md`) by weighting checkpoints and searching the underlying navigation web
(`structs/web_path.md`). It is the middle of the three navigation layers:

```
  ai_action_manager  (movement factories carry an ai_path*)      structs/ai_action_manager.md
        ^
  ai_wayfind  ->  ai_path                                        this file, structs/ai_path.md
        ^
  C_web_path  (nodes + connections, loaded from check.bin)       structs/web_path.md
```

| | |
|---|---|
| PC prefix | `ai_wayfind_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x84` (singleton at `0x00655224`, `ai_manager+0x0c`; `police_board` follows at `ai_manager+0x90`) |
| PC constructor | - (`ai_manager_ctor` builds `route` in place) |
| PC translation unit | `[0x00402740, 0x0040f700)`, 72 starts, shared with `ai_path`, whose doc hosts the unit (`structs/ai_path.md` §9) |
| PS2 class | `ai_wayfind` (tag `10ai_wayfind`), 19 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Offsets from `0x00655224`. The base is byte-solid: the ECX load before each call is `mov ecx, 0x00655224` at
all 35 callsites of `ai_wayfind_build_optimal_path` and all 32 of the two `blocked_con` accessors.

```c
typedef struct ai_wayfind {                   // 0x84 bytes, singleton at 0x00655224
    /* +0x00 */ float    cb_start_pos[3];     // [H] the query start, published for the callbacks
    /* +0x0c */ float    cb_end_pos[3];       // [H] the query end, published for the callbacks
    /* +0x18 */ float    cb_direction[3];     // [H] the normalised query direction
    /* +0x24 */ float    cb_result[3];        // [H] a position the callbacks hand back
    /* +0x30 */ float    cb_goal_dist_sq;     // [H] the forward strategies' squared goal distance
    /* +0x34 */ uint32_t unused_34;           // [U] no access; no reader (.text scan)
    /* +0x38 */ float    cb_radius_sq;        // [H] the flee radius squared, or the corner distance
    /* +0x3c */ int32_t  cb_forward_mode;     // [H] 0 / 1 / 2, the edge cost's verdict for the goal test
    /* +0x40 */ uint32_t cb_target_node;      // [H] the target's cached node, or -1
    /* +0x44 */ uint32_t cb_target_next_node; // [H] the target's alternate node, or -1
    /* +0x48 */ uint8_t  unread_48;           // [U] the two directed planners store 0; no reader (.text scan)
    /* +0x4c */ uint8_t  route_alloc;         // [H] allocator byte of the route vector
    /* +0x50 */ uint32_t *route_begin;        // [H] first node index of the route
    /* +0x54 */ uint32_t *route_end;          // [H] one past the last node index
    /* +0x58 */ uint32_t *route_cap;          // [H] end of the vector's storage
    /* +0x5c */ vector_blocked_con *blocked_cons; // [H] the installed blocked-connection list
    /* +0x60 */ float    start_pos[3];        // [H] the position the start node was resolved from
    /* +0x6c */ float    end_pos[3];          // [H] the position the end node was resolved from
    /* +0x78 */ uint32_t status;              // [H] resolver outcome bits (§4c)
    /* +0x7c */ uint32_t start_node;          // [H] resolved start node index
    /* +0x80 */ uint32_t end_node;            // [H] resolved end node index
} ai_wayfind;
```

**Why the seven arrays have three entries.** Each is one x, y, z position or direction. `+0x34` (`unused_34`) has no
access at all: no absolute reference to `0x00655258` and no `this`-relative one (.text scan).

**`route` is the only allocation the class owns.** Every strategy clears it and passes it as the output of its
web-path search; `ai_wayfind_build_optimal_path` consumes it. Its node indices index the `C_web_path` node array
(the body multiplies them by the stride `0x1e` against `*(g_act_mission+0x2c)+4`). Strategy 6's candidate flood
goes to a caller-local vector, not here. `ai_wayfind_dtor` frees `(root+0x0c)+0x50` = `root+0x5c`, exactly the
vector `ai_manager_ctor` builds at head `root+0x58`; a whole-class destructor that frees one buffer also says
that nothing else here is owned storage (`structs/ai_manager.md`).

**Two constant up-vectors sit just below this object and are not part of it**: `g_vec_up_00655198_*` and
`g_vec_up_00655208_*`, each a separately static-initialised `(0, 1, 0)` (`vec_up_00655198_static_init` /
`vec_up_00655208_static_init`). The first is the cross-product axis of the `ai_path` corridor tests.

## 2. Field evidence

Measured by a register-tag census of the class's functions plus every absolute reference inside
`[0x00655224, 0x006552a8)` in `.text`, and by reading the small bodies. The writes through a pointer the census
cannot tag (`start_node`, the two position triples) come from the resolver bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | cb_start_pos | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route` | `ai_wayfind_cb_flee_goal_beyond_radius` | `g_ai_wayfind_cb_start`, the object's base address |
| +0xc | cb_end_pos | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_route_via_corner` | `ai_wayfind_cb_corner_edge_cost`, `ai_wayfind_cb_forward_edge_cost` | `g_ai_wayfind_cb_end` |
| +0x18 | cb_direction | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route` | `ai_wayfind_cb_flee_edge_cost_directional`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route` | `g_ai_wayfind_cb_dir`; normalised with the `1e-08` / `+-1.0`-axis ladder |
| +0x24 | cb_result | H | `ai_wayfind_cb_flee_goal_beyond_radius`, `ai_wayfind_cb_forward_edge_cost` | `ai_wayfind_plan_flee_route` | `g_ai_wayfind_cb_out`, written by the callbacks |
| +0x30 | cb_goal_dist_sq | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3` | `ai_wayfind_cb_forward_edge_cost` | `g_ai_wayfind_cb_goal_dist_sq`; v2 stores its float raw, v3 stores `max(dist², param)` |
| +0x34 | unused_34 | U | - | - | no access: no absolute reference to `0x00655258` and no `this+0x34` (.text scan) |
| +0x38 | cb_radius_sq | H | `ai_wayfind_cb_corner_edge_cost`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_route_via_corner` | `ai_wayfind_cb_flee_goal_beyond_radius` | `g_ai_wayfind_cb_radius_sq`; flee: `param²` capped at `400.0` (20 units); corner: 0, then the edge cost stores a squared distance; `ai_wayfind_cb_corner_edge_cost` writes it and does not read it |
| +0x3c | cb_forward_mode | H | `ai_wayfind_cb_forward_edge_cost` | `ai_wayfind_cb_forward_goal_by_mode` | `g_ai_wayfind_cb_forward_mode`; 0 / 1 / 2, set by the edge cost and read by the goal predicate |
| +0x40 | cb_target_node | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3` | `ai_wayfind_cb_forward_edge_cost` | `g_ai_wayfind_cb_target_node`; v3: `target+0x730`; v2: -1 |
| +0x44 | cb_target_next_node | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3` | `ai_wayfind_cb_forward_edge_cost` | `g_ai_wayfind_cb_target_next_node`; v3: `target+0x734` under `target+0x73c & 0x10`, else `+0x730`; v2: -1 |
| +0x48 | unread_48 | U | `ai_wayfind_plan_directed_route_v2` (`0x00406c2e`), `ai_wayfind_plan_directed_route_v3` (`0x004070e8`) | - | `g_ai_wayfind_cb_field_0x48`; byte stores of 0 next to the two node sentinels; no reader (.text scan: these two stores are the only references to `0x0065526c`, and the class has no `this+0x48`) |
| +0x4c | route_alloc | H | `ai_manager_ctor` | `ai_wayfind_build_optimal_path`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_route_via_corner`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route` | the allocator byte, `ai_manager_ctor` stores it |
| +0x50 | route_begin | H | `ai_manager_ctor`, `ai_wayfind_dtor` | `ai_wayfind_build_optimal_path`, `ai_wayfind_dtor`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route` | an empty list makes `ai_wayfind_build_optimal_path` free the fresh path and return NULL |
| +0x54 | route_end | H | `ai_manager_ctor`, `ai_wayfind_dtor` | `ai_wayfind_build_optimal_path`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route` | count = (end - begin) >> 2 |
| +0x58 | route_cap | H | `ai_manager_ctor`, `ai_wayfind_dtor` | - | the vector helpers write it through the `+0x4c` address |
| +0x5c | blocked_cons | H | `ai_wayfind_clear_blocked_cons`, `ai_wayfind_set_blocked_cons`, `dog_replan_path`, `dog_set_destination_and_plan`, `entity_script_tick` | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_route_via_corner`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route`, `ai_wayfind_resolve_start_node` | also the global `g_ai_wayfind_blocked_cons`; §4b |
| +0x60 | start_pos | H | `ai_wayfind_resolve_start_node` | `ai_wayfind_build_optimal_path`, `ai_wayfind_resolve_start_node` | - |
| +0x6c | end_pos | H | `ai_wayfind_resolve_end_node`, `ai_wayfind_plan_flee_route` (the callback's result), `ai_wayfind_plan_directed_route_v2` and `ai_wayfind_plan_directed_route_v3` (the goal or the last node), `ai_wayfind_plan_search_route` (the flood's node) | `ai_wayfind_build_optimal_path`, `ai_wayfind_plan_search_route`, `ai_wayfind_resolve_end_node` | - |
| +0x78 | status | H | `ai_wayfind_get_node_for_position`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_route_via_corner`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route`, `ai_wayfind_resolve_end_node`, `ai_wayfind_resolve_start_node` | `ai_wayfind_build_optimal_path`, `ai_wayfind_get_node_for_position`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_resolve_start_node` | §4c |
| +0x7c | start_node | H | `ai_wayfind_resolve_start_node` | `ai_wayfind_get_node_for_position`, `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_route_via_corner`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route`, `ai_wayfind_resolve_start_node` | `0xffffffff` = unresolved |
| +0x80 | end_node | H | `ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`, `ai_wayfind_plan_flee_route`, `ai_wayfind_plan_search_route`, `ai_wayfind_resolve_end_node` | `ai_wayfind_plan_route_via_corner`, `ai_wayfind_plan_search_route`, `ai_wayfind_plan_shortest_route`, `ai_wayfind_resolve_end_node` | `0xffffffff` = unresolved; strategies 3–6 force 0 |

## 3. Method family

18 of 19 PS2 methods mapped, all CONFIRMED by body diff (the five callbacks are free functions on both builds,
in 3d); `isInBound` is the one with no PC counterpart (§6). The three functions outside the unit are PC-only
because the PS2 inlines them: `ai_wayfind_dtor` in the `ai_manager` unit, the two accessors in the `C_entity` unit.
§3d lists the unit's helpers this class's functions call; the rest of the unit's helpers and its CRT rows stay with
the host, `structs/ai_path.md`, as do the `ai_path_` functions its methods call.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443840 | `ai_wayfind_dtor` | `0x00655224` | - | - | - | `operator_delete` on the route buffer, zeroes `+0x50` / `+0x54` / `+0x58`; reached only from the two unwind funclets `Unwind_006354f0`, `Unwind_00635560`; inlined into `__dt__10ai_managerFv` on the PS2 |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004256e0 | `ai_wayfind_set_blocked_cons` | - | - | - | installs the caller's `blocked_con` list at `+0x5c`; 22 callers, all `entity_action_tick`; inlined on the PS2 |
| 0x004256f0 | `ai_wayfind_clear_blocked_cons` | - | - | - | zeroes `+0x5c`; 10 callers, all `entity_action_tick`; inlined on the PS2 |

The usage shape, from `entity_action_tick` case 0: `ai_wayfind_set_blocked_cons(entity+0x1198)`, then
`ai_wayfind_plan_shortest_route`, then `ai_wayfind_clear_blocked_cons`: the caller installs its own list for the
duration of one plan.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004035e0 | `ai_wayfind_build_path_node_from_nodes` | `GetPathNode__10ai_wayfindFP10C_web_nodeP10C_web_nodeP10C_web_nodeP12ai_path_node` | 0x00357f60 | CONFIRMED | the three-node overload: builds one `ai_path_node` route step from the prev / cur / next node triple; opens `if (prev == NULL) prev = cur`, which only a node-pointer overload can do |
| 0x00403c30 | `ai_wayfind_build_path_node_from_positions` | `GetPathNode__10ai_wayfindFP10C_web_nodeRC8S_vectorRC8S_vectorP12ai_path_node` | 0x00358630 | CONFIRMED | the `S_vector` overload: the body copies two 3-float vectors out of params 2 and 3 into the out-record |
| 0x00403ee0 | `ai_wayfind_build_optimal_path` | `GetOptimalPath__10ai_wayfindFv` | 0x00358a60 | CONFIRMED | allocates a fresh `ai_path` and computes the route for `route`; frees it and returns NULL on an empty list; the sole `ai_path` allocator; 35 callers: `entity_action_tick` 27, `entity_process_command` 4, `entity_script_tick` 2, `dog_replan_path`, `dog_set_destination_and_plan` |
| 0x00405510 | `ai_wayfind_resolve_start_node` | `FindStartCheckpoint__10ai_wayfindFR8S_vectorUib` | 0x0035abd0 | CONFIRMED | resolves a world position to a start node (§4c): 16 attempts, LOS test from `+0.5` to `+0.2` above, SELECTED-bit set / clear bracket |
| 0x004059d0 | `ai_wayfind_resolve_end_node` | `FindEndCheckpoint__10ai_wayfindFR8S_vectorUib` | 0x0035b220 | CONFIRMED | the end-side twin: the same 16-attempt loop without the `blocked_con` sweep, then the neighbour-scan fallback |
| 0x00405dd0 | `ai_wayfind_get_node_for_position` | `GetPosition__10ai_wayfindFR8S_vectorPUiPUiUi` | 0x0035b6e0 | CONFIRMED | four statements: clear `status`, resolve, copy `start_node` and `status` out |
| 0x00405e10 | `ai_wayfind_plan_shortest_route` | `FindWaySHORTEST__10ai_wayfindFR8S_vectorR8S_vectorUib` | 0x0035b740 | CONFIRMED | strategy 1 of 6 (§5a); resolve both ends, validity guard, `0x29a` sentinel test, SELECTED bracket, `web_path_find_path` |
| 0x00406040 | `ai_wayfind_plan_route_via_corner` | `FindWayCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorUiP19ai_corner_hide_infob` | 0x0035b9e0 | CONFIRMED | strategy 2: scans the `check_seg` segment store for type-2 specials (`ai_seg_corner`) and routes via the nearest usable one (`structs/check_seg.md` §4c, §5) |
| 0x00406550 | `ai_wayfind_plan_flee_route` | `FindWayRUNAWAY_V1__10ai_wayfindFR8S_vectorR8S_vectorfUib` | 0x0035c070 | CONFIRMED | strategy 3: start node only, end forced to 0; publishes one position, squares its float and clamps it at `400.0` |
| 0x00406930 | `ai_wayfind_plan_directed_route_v2` | `FindWayFORWARD_V2__10ai_wayfindFR8S_vectorR8S_vectorfUib` | 0x0035c4c0 | CONFIRMED | strategy 4: publishes two positions and stores its float raw at `+0x30`, plus a different callback pair |
| 0x00406de0 | `ai_wayfind_plan_directed_route_v3` | `FindWayFORWARD_V3__10ai_wayfindFR8S_vectorR8S_vectorP7C_humanP7C_humanfUib` | 0x0035ca80 | CONFIRMED | strategy 5: as v2, plus two `C_human*` participants and the `FindPathCompFuncJIPO` search variant |
| 0x004072c0 | `ai_wayfind_plan_search_route` | `FindWaySEARCH__10ai_wayfindFP15ai_check_searchUiUib` | 0x0035d140 | CONFIRMED | strategy 6: start node only, floods the web via `ai_wayfind_score_and_add_checkpoint` and routes with the plain search |
| 0x00407620 | `ai_wayfind_test_corner_hide_side` | `hasCornerVisHide__10ai_wayfindFP13ai_seg_cornerR8S_vectorR8S_vectorR8S_vectorPUibf` | 0x0035d4c0 | CONFIRMED | corner-usability predicate that also writes the 0 / 1 side selector; shared `9.0`, `1.25`, `1e-08`; three signed 2-D angle calls; LOS test |
| 0x0040d4e0 | `ai_wayfind_find_nearest_hide_corner` | `FindCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorP19ai_corner_hide_info` | 0x00366360 | CONFIRMED | scans the map's segment clusters for cover corners and fills the caller's `ai_corner_hide_info`; the helper behind strategy 2 |
| 0x0040d700 | `ai_wayfind_score_and_add_checkpoint` | `WeightCheckpoints__10ai_wayfindFP15ai_check_searchPQ23std32vector<Ui,Q23std13allocator<Ui>>UiUiUiUi` | 0x00366750 | CONFIRMED | the flood step of strategy 6: dedup guard, append to the node vector, gate on the filter mask (against the node's `+0x0c` category), start radius², target radius² and forward-facing, then weight by the two nibbles of the node's `+0x16` byte (`structs/ai_check.md` §5a) |
| 0x0040dc10 | `ai_wayfind_compute_2d_line_intersection` | `Compute2DIntersection__10ai_wayfindFP9S_vector2P9S_vector2P9S_vector2P9S_vector2P9S_vector2` | 0x00366ec0 | CONFIRMED | 2-D line / line intersection; every param indexed `[0]` / `[1]` only, so five `S_vector2*`; `0x322bcc77` is `1e-08` |
| 0x0040dc90 | `ai_wayfind_compute_node_left_right_gate` | `GetLR__10ai_wayfindFP8S_vectorP8S_vectorP8S_vectorP12ai_path_nodefff` | 0x00366f70 | CONFIRMED | builds the left / right gate pair around a path node, the pair a normal waypoint stores at record `+0x00` / `+0x0c`: at a turn sharper than 45 degrees one gate point is the intersection of the two offset corridor edges and the other the node plus the perpendicular times the width, and it also writes `dir`, `pos` and `type` 1 / 2 (`structs/ai_path.md` §2a); otherwise a straight gate and `type` 0; shared `0.707`, `1.2`, `0.5` x6 |
| 0x0040e850 | `ai_wayfind_compute_nearest_point_on_segment` | `ComputeNearestPoint__10ai_wayfindFP8S_vectorP8S_vectorP8S_vectorP8S_vectorff` | 0x00367ce0 | CONFIRMED | projects a point onto an X / Z segment, lerps two offsets by the projection ratio, steps perpendicular; Y forced to 0 |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00402cd4 | `thunk_jmp_via_retaddr_plus_0x953b` | - | - | - | `ai_wayfind_cb_flee_edge_cost_directional`'s one call: pops the return address, adds `0x953b`, jumps through the slot there |
| 0x00402ce0 | `ai_wayfind_cb_corner_edge_cost` | `fxForwardShortest__FUsUsUc` | 0x003572f0 | CONFIRMED | §5b; used by strategy 2 |
| 0x00402dd0 | `ai_wayfind_cb_flee_goal_beyond_radius` | `fxRunawayEndFuncV1__FUs` | 0x00357420 | CONFIRMED | §5b; used by strategy 3 |
| 0x00402e80 | `ai_wayfind_cb_flee_edge_cost_directional` | `fxRunawayFuncV1__FUsUsUc` | 0x00357520 | CONFIRMED | §5b; used by strategy 3 |
| 0x00403080 | `ai_wayfind_cb_forward_goal_by_mode` | `fxForward2EndFuncV1__FUs` | 0x00357780 | CONFIRMED | §5b; used by strategies 4 and 5 |
| 0x004030a0 | `ai_wayfind_cb_forward_edge_cost` | `fxForward2FuncV1__FUsUsUc` | 0x003577d0 | CONFIRMED | §5b; used by strategies 4 and 5 |
| 0x00403530 | `ai_check_desc_unpack_data` | `UNPACKDATA__13ai_check_descFUi` | 0x00357e30 | CONFIRMED | called by `ai_wayfind_score_and_add_checkpoint`; owned by `structs/ai_check.md` |
| 0x00403570 | `ai_check_desc_unpack_ranges` | `UNPACKRANGES__13ai_check_descFUi` | 0x00357e80 | CONFIRMED | called by `ai_wayfind_plan_search_route`; owned by `structs/ai_check.md` |
| 0x00403e90 | `vec3_div_scalar_in_place` | - | - | - | called by `ai_wayfind_cb_forward_edge_cost` |
| 0x00403ec0 | `vec3_dot_product` | - | - | - | called by `ai_wayfind_cb_forward_edge_cost` |
| 0x0040f010 | `web_node_get_connection` | - | - | - | called by `ai_wayfind_cb_forward_edge_cost`; owned by `structs/web_node.md` |
| 0x0040f040 | `vec3_copy_to` | - | - | - | called by `ai_wayfind_build_path_node_from_nodes` |
| 0x0040f060 | `std_vector3c_insert_at` | - | - | - | called by `ai_wayfind_build_optimal_path`; the path's `0x3c` waypoint vector |
| 0x0040f2b0 | `std_vector3c_erase_at` | - | - | - | the same |
| 0x0040f300 | `std_vector18_erase_at` | - | - | - | called by `ai_wayfind_build_optimal_path` |
| 0x0040f350 | `std_vector3c_insert_n` | - | - | - | called by `ai_wayfind_build_optimal_path` |

## 4. The object's three groups of fields

### 4a. Why the object has two copies of the query geometry

`+0x60` / `+0x6c` and `+0x00` / `+0x0c` both hold a start and an end position, which looks redundant until you
see who reads them.

The callback-driven strategies hand **free functions** to `web_path_find_path_to_goal_fn` as the goal predicate
and the per-edge cost function. A free function has no `this`, so it cannot reach the singleton through ECX. The
strategies therefore **publish** the query geometry into the object's first `0x40` bytes, which the callbacks reach
by **absolute address**: the flee goal callback writes `[0x655248]`, `[0x65524c]` and `[0x655250]`, that is
`this+0x24`, `+0x28`, `+0x2c`, with no receiver in sight. `ai_wayfind_plan_shortest_route` writes none of that
block, because it uses the plain graph search and passes no callbacks.

Ghidra renders the head of this object as loose globals and the tail as `this+N`, purely because of how each is
addressed. The head globals are named, so a grep for **`g_ai_wayfind_cb_`** finds the callback-block traffic that a
`this+` grep misses, and vice versa: §2's notes give each field's global. **`g_ai_wayfind_cb_start` names address
`0x00655224`, which is also the singleton base.** The field name was chosen over an object name deliberately (user
decision), so a reader who hits `mov ecx, 0x00655224` will not see a symbol; this section is the record of that.

### 4b. `blocked_cons` — the list swept around every search

A pointer to a caller's 0x10-stride `blocked_con` vector, a different thing from `route`. The resolvers and the
strategies walk it before each search, drop expired records, and set the SELECTED bit (`0x40` of node `+0x0d`) on the
nodes it names, clearing it after. Installers: `entity_action_tick` through `ai_wayfind_set_blocked_cons` (22 calls,
`C_entity+0x1198`; 10 clears); by absolute address `entity_script_tick` (`C_entity+0x1198`), `dog_replan_path` and
`dog_set_destination_and_plan` (`C_dog+0x2fc`).

### 4c. `status` — what the resolvers found

`ai_wayfind_get_node_for_position` and the strategies zero it first. `ai_wayfind_resolve_start_node` ends one of
three ways, read from its body:

| Value | Meaning | Evidence |
|---|---|---|
| bit 0 clear, `start_node` valid | the position lies inside the radius of the nearest visible node or of one of its neighbours | the two `goto LAB_0040596d` paths skip the `or` |
| bit 0 clear, `start_node` = `0xffffffff` | no node: `web_path_find_nearest_node` returned none, or all 16 attempts failed the LOS test | both exits jump past the `or` to `LAB_004059b2` |
| bit 0 set, `start_node` valid | the nearest visible node was kept although the position lies outside the radius of it and of every neighbour | `or al, 1` at `0x00405968` |

So bit 0 means "resolved, but outside every radius", not a failure; a failure is `0xffffffff`, which is what the
strategies test. Bit 1 (`or dword ptr [eax + 0x78], 2` at `0x00405d9d`) is the same for `ai_wayfind_resolve_end_node`.
`ai_loc_update` is a consumer of bit 0 (`structs/ai_loc.md` §2a).

## 5. Routing policy

### 5a. The six route strategies at a glance

| # | PC name | Endpoints resolved | Search used | Distinguishing state |
|---|---|---|---|---|
| 1 | `ai_wayfind_plan_shortest_route` | start + end | plain `web_path_find_path` | publishes nothing |
| 2 | `ai_wayfind_plan_route_via_corner` | start + end | cost-fn search | publishes `cb_end_pos`, zeroes `cb_radius_sq`; scans the `check_seg` segment store for type-2 `ai_seg_corner` cover corners |
| 3 | `ai_wayfind_plan_flee_route` | start only | goal + cost callbacks | one position; radius² clamped at `400.0` |
| 4 | `ai_wayfind_plan_directed_route_v2` | start only | goal + cost callbacks | two positions; raw float |
| 5 | `ai_wayfind_plan_directed_route_v3` | start only | `FindPathCompFuncJIPO` | two positions + two `C_human*` |
| 6 | `ai_wayfind_plan_search_route` | start only | plain `web_path_find_path` | floods the web to discover the end |

Strategies 3–6 force `end_node` because they have no destination node: the goal predicate, or the flood, decides
where to stop.

### 5b. The five route-cost callbacks

These free functions **are** the routing policy: the goal predicate decides when a search stops, the cost function
which way it prefers to go. Each is taken by address by its strategies, which is why a caller shows it as `&LAB_…` and
its E8 count is zero.

| PC name | Used by | Role |
|---|---|---|
| `ai_wayfind_cb_corner_edge_cost` | strategy 2 | cost only: stores the squared distance of the edge's far node from `cb_end_pos` in `cb_radius_sq` and returns the connection record |
| `ai_wayfind_cb_flee_goal_beyond_radius` | strategy 3 | stop once a node is further than `cb_radius_sq` from `cb_start_pos`; writes `cb_result` |
| `ai_wayfind_cb_flee_edge_cost_directional` | strategy 3 | `(2 - dot(cb_direction, edge_dir))² * edge_len`, so aligned x1, perpendicular x4, reversed x9 |
| `ai_wayfind_cb_forward_goal_by_mode` | strategies 4, 5 | returns 0 / 3 / 1 for `cb_forward_mode` 0 / 1 / 2; ignores the node index |
| `ai_wayfind_cb_forward_edge_cost` | strategies 4, 5 | distance-derived cost from `cb_end_pos`, `cb_goal_dist_sq` and the two target nodes; the only writer of `cb_forward_mode`, and writes `cb_result` |

**The flee policy, in full:** run in the published direction (cost bias) until you are further than the radius from
where you started (goal predicate). Two small functions, and that is the whole behaviour.

## 6. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00367e70 | `isInBound__10ai_wayfindFPQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>R8S_vector` | the code that calls it is absent on the PC (below) |

- **`ai_wayfind` and `ai_path` interleave in one PS2 translation unit** (`0x00357f60`–`0x00368940`), so they share one
  PC unit and one emission order. Order-align must treat them as a single sequence.
- **The class anchor is an allocation-uniqueness argument**, not a body diff of an 8.5 KB function: `ai_path` is
  allocated in exactly one place on each build: PC `operator_new(0x9c)` inside `ai_wayfind_build_optimal_path`, PS2
  `__nw__7ai_path(0xa0)` inside `GetOptimalPath__10ai_wayfindFv`. Both then zero three vector triples in the fresh
  object and walk the nav web. Size ratio 8548 / 5680 = 1.50, in band.
- **Receiver shape matches:** PS2 `GetOptimalPath` is `Fv` (implicit `this` only); the PC twin takes its receiver in
  ECX as an absolute global.
- **The nine planner methods were each body-diffed** before being attributed to this class. A forced 10↔10 order-align
  gap between `ai_wayfind_build_optimal_path` (PS2 rank 3) and `ai_path_destroy` (PS2 rank 14) generated the candidates;
  the gap was enumerated as Ghidra `FUN_` ids ∪ named defs and Ghidra's default analysis returned
  the same 11 starts.
- **The tail of the cluster is only half forced.** `ai_wayfind_score_and_add_checkpoint` (PS2 rank 33) splits it: the
  sub-gap below that anchor is a forced 18↔18, the sub-gap above it is non-forced (23 PC starts vs 11 PS2 methods, two
  of them PS2-only).
- **Separators of the look-alike twins.** The two `GetPathNode` overloads split on `if (prev == NULL) prev = cur` and on
  the two copied vectors. `FindWayRUNAWAY_V1` and `FindWayFORWARD_V2` have identical signatures and split on one
  published position plus a clamped square against two published positions plus a raw float. `GetPosition` has four
  statements in the same order and pins the `+0x18` tail rebase. `FindEndCheckpoint`: field rebase PS2 `+0x98` ↔ PC
  `+0x80`. Arity agrees on every row; ratios: `GetPathNode` 1.08 and 0.97, `WeightCheckpoints` 1.40, `Compute2DIntersection`
  1.31, `GetLR` 1.14, `ComputeNearestPoint` 0.86.
- **The callback twins are pinned by their consumers on both builds**, themselves confirmed twins, which is stronger than any
  size ratio. All five are free functions on the PS2 (`fxRunawayFuncV1__FUsUsUc` and siblings), so their class column is
  empty and a class-filtered query misses them. The name `fxForwardShortest` says "ForwardShortest" but SHORTEST passes no
  callbacks: its consumer is the corner strategy.
- **PS2-only, near-certain:** `ai_path`'s class-specific `operator new` / `operator delete` (pool allocation via the
  `ai_spageman` family); PC uses the global `operator_new` here.
- **The element name `blocked_con`** comes from the PS2 type `reserve__gif::vector<blocked_con,2>`.
- **The singleton's PS2 base is `0x00509090`** [H]: all 48 PS2 calls of `ai_wayfind` methods pass `this = 0x509090`, the
  `ai_manager` root (`0x509080`) `+0x10`. Both builds hold the same fields, including the two copies of the query
  geometry; PS2 puts the resolvers' copy first:

  | field | PC | PS2 |
  |---|---|---|
  | resolver start / end position | `+0x60` / `+0x6c` | `+0x00` / `+0x10` |
  | `cb_start` / `cb_end` / `cb_dir` / `cb_result` | `+0x00` / `+0x0c` / `+0x18` / `+0x24` | `+0x20` / `+0x30` / `+0x40` / `+0x50` |
  | `cb_goal_dist_sq` / never accessed / `cb_radius_sq` | `+0x30` / `+0x34` / `+0x38` | `+0x60` / `+0x64` / `+0x68` |
  | `cb_forward_mode` / target nodes / write-only byte | `+0x3c` / `+0x40`, `+0x44` / `+0x48` | `+0x6c` / `+0x70`, `+0x74` / `+0x78` |
  | the route vector | `+0x4c` | `+0x80` |
  | the `blocked_con` list pointer | `+0x5c` | `+0x8c` |
  | status / start node / end node | `+0x78` / `+0x7c` / `+0x80` | `+0x90` / `+0x94` / `+0x98` |

  The PS2 rows are read from `FindStartCheckpoint` (writes `+0x00`, `+0x94`, reads `+0x8c`), `FindEndCheckpoint` (`+0x10`,
  `+0x98`), `GetPosition` (`+0x90`, `+0x94`), `FindWayRUNAWAY_V1` (`0x5090b0`, `0x5090d0`, `0x5090f8`, `0x5090e0`),
  `FindPath`'s route argument `+0x80` and the `C_entity` / `C_dog` writers of `0x50911c`; `0x5090f4` is referenced
  nowhere. The tail band rebases by a constant `0x18`; the rest does not, so map by the table.
- **The callback publication block spacing differs by exactly the `S_vector` padding.** PS2 vectors are 4 floats (16 B),
  PC's are 3 (12 B), so the block strides `b0 / c0 / d0` on PS2 against `224 / 230 / 23c` on PC.
- **`hasCornerVisHide`'s corner field index shifts by one float**: PS2 reads `corner[0xb]` where PC reads `corner[10]`,
  for the same reason.
- **`+0x48` is dead on both builds.** PS2 `FindWayFORWARD_V2` and `_V3` store `AIManager+0x88` (`0x509108`,
  `ai_wayfind+0x78`) = 0 right after the two node sentinels (`AIManager+0x80` / `+0x84` = -1), the same three statements
  as the PC's `+0x40` / `+0x44` / `+0x48`, and nothing on the PS2 reads that byte either.
- **`isInBound` (260 B) has no PC counterpart because the code that calls it is absent on the PC.** It is a convex-polygon
  test over a `std::vector<S_vector2>` parameter (fewer than 3 points returns 0; every edge cross product must share one
  sign); `0x509090` is its `this`. Its 6 calls sit 2x in each of the three spec-node searches, in two loops that run when
  only one of the two colliders was hit: for each pass-side entry (`ai_actor_evading`) whose blocker is that collider's
  owner, blocker kind not 2 or `0x1b` and side 2 (resp. 1), they fetch the blocker's 2-D footprint through its virtual
  `GetBoundPoints2D` and abort the search unless the probe point lies inside. In all three PC searches
  (`ai_path_find_spec_node_from_coll_list` / `_from_plane` / `_from_positions`) the second door block leads directly to
  the "both colliders null" exit, and the counts agree: PS2 7 virtual bound-point calls per search (5 + the 2 in these
  loops) against PC 4 plus 4 `i3d_frame_get_bound_points_2d`, the 5 ↔ 4 being the merged block of `structs/ai_check.md`
  §4. So the PC skips this abort; `ai_path_is_point2d_in_checkpoint_radius` is a genuine radius test and is not its
  counterpart.
- **`FindPathCompFuncJIPO`** exists on both builds as a distinct search entry point used only by `FindWayFORWARD_V3`.
- **`ai_corner_hide_info`'s member order differs between builds**: PC puts the corner pointer first, PS2 puts it after both
  vectors, for VU quadword alignment. That is a reordering, not a rebase; see `structs/check_seg.md`.

## 7. Related

- `structs/ai_path.md` — the object this class produces; same unit.
- `structs/web_path.md` — the graph underneath. `structs/web_node.md` — the node records the indices point at.
- `structs/ai_manager.md` — the root object this singleton is a member of.
- `structs/ai_action_manager.md` — the consumer.
- `structs/check_seg.md` §1d / §1e — `ai_seg_corner` (`0x38`) and `ai_corner_hide_info` (`0x24`), the two records strategy
  2 works with. `ai_seg_corner`'s `radius` sits at PC `+0x24`, the same float `check_seg_is_pos_in_any_special` reads as
  the type-2 sphere radius; two unrelated functions agreeing on one offset is what byte-proved the field.
- `structs/ai_check.md` §7 — the `ai_path_find_spec_node_*` searches and their bound-points calls.

## 8. Open items

None.
