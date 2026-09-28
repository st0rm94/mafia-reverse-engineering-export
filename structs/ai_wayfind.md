# `ai_wayfind` — the route planner (checkpoint-based wayfinding)

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-09-28 19:40 +0200

PC prefix **`ai_wayfind_*`**.

| | |
|---|---|
| PC singleton | **`0x00655224`**, a member of the `ai_manager` root at `ai_manager+0x0c` |
| PC cluster | `[0x00403ee0, 0x0040f760)` — shared with `ai_path`, one PS2 translation unit |
| PS2 class | `ai_wayfind`, 19 methods, **18 mapped**, all CONFIRMED by body-diff (§2) |
| Route strategies | 6, all mapped (§3a) |
| Route-cost callbacks | 5, all named (§3b) |

`ai_wayfind` is the **route planner**: a singleton that turns a movement request into an `ai_path`
(`structs/ai_path.md`) by weighting checkpoints and searching the underlying navigation web
(`structs/web_path.md`). It is the middle of the three navigation layers:

```
  ai_action_manager  (movement factories carry an ai_path*)      structs/ai_action_manager.md
        ^
  ai_wayfind  ->  ai_path                                        THIS FILE / structs/ai_path.md
        ^
  C_web_path  (nodes + connections, loaded from check.bin)       structs/web_path.md
```

Confidence tags: **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Singleton layout (PC offsets from `0x00655224`)

The base address is byte-solid. `mov ecx, 0x00655224` appears at 34 of 35 callsites of
`ai_wayfind_build_optimal_path`, and a receiver probe that reads the ECX write preceding each
reference finds the same constant at all 43 references of the two checkpoint-container accessors
(29 set, 14 clear). Every absolute `_DAT_006552xx` address observed in the strategy bodies resolves
to a distinct `this+N` below, with no collision — see §1a.

```c
typedef struct ai_wayfind {                  // singleton at 0x00655224
    // ---- the CALLBACK PUBLICATION BLOCK (see §1a) ----
    /* +0x00 */ float      cb_start_pos[3];  // [H] published by the callback strategies
    /* +0x0c */ float      cb_end_pos[3];    // [H] published by the two directed strategies only
    /* +0x18 */ float      cb_direction[3];  // [H] normalised; the 1e-08 / +-1.0-axis ladder
    /* +0x24 */ float      cb_result[3];     // [H] written BY the goal callback
                                             //     ai_wayfind_cb_flee_goal_beyond_radius
    /* +0x38 */ float      cb_range;         // [H] flee: radius^2 clamped at 400.0 (= 20 units);
                                             //     directed: the raw float, unclamped
    /* +0x3c */ int32_t    cb_forward_mode;  // [H] 0/1/2, read by both forward callbacks
    /* +0x40 */ uint32_t   cb_target_node;   // [H] v3: the target's cached node; -1 otherwise
    /* +0x44 */ uint32_t   cb_target_next;   // [H] v3: the target's alternate node; -1 otherwise
    /* +0x48 */ uint8_t    cb_field_48;      // [H] stored 0 by v2 and v3 before the search; no reader
    /* +0x4c */ std_vector route;            // [H] the class's own std::vector<uint> of node
                                             //     indices: allocator byte +0x4c, begin +0x50,
                                             //     end +0x54, cap +0x58; count = (end-begin) >> 2.
                                             //     An empty list (+0x50 == 0) makes
                                             //     ai_wayfind_build_optimal_path free the fresh
                                             //     ai_path and return NULL. The ONLY allocation
                                             //     the class owns - ai_wayfind_dtor frees this
                                             //     buffer and zeroes +0x50/+0x54/+0x58.
    /* +0x5c */ void      *checkpoint_ctr;   // [H] a DIFFERENT container: the 0x10-stride
                                             //     blocked_con list swept before and after every
                                             //     search. Also reachable as the global
                                             //     g_ai_wayfind_checkpoint_container (0x00655280)
                                             //     - four callers write it by absolute address.
                                             //     The 0x10 stride is a `blocked_con` record, from
                                             //     the PS2 type reserve__gif::vector<blocked_con,2>;
                                             //     installers seen so far: entity_script_tick
                                             //     (C_entity+0x1198), dog_replan_path and
                                             //     dog_set_destination_and_plan (C_dog+0x2fc)
    /* +0x60 */ float      start_pos[3];     // [H] written by ai_wayfind_resolve_start_node
    /* +0x6c */ float      end_pos[3];       // [H] written by ai_wayfind_resolve_end_node
    /* +0x78 */ uint32_t   status;           // [H] bit 0 = start resolve failed, bit 1 = end
    /* +0x7c */ uint32_t   start_node;       // [H] node index, 0xffffffff = unresolved
    /* +0x80 */ uint32_t   end_node;         // [H] node index, 0xffffffff = unresolved
} ai_wayfind;
```

The node indices in `route` index the `C_web_path` node array (the body multiplies them by the node
stride `0x1e` against `*(g_act_mission+0x2c)+4`).

**`+0x4c` is byte-proven a member of this class, not of the root.** `ai_wayfind_dtor` frees
`(root+0x0c)+0x50` = `root+0x5c`, which is exactly the vector `ai_manager_ctor`
default-constructs at head `root+0x58`. A whole-class destructor that frees one buffer is also a
strong statement about the rest of the layout: nothing else here is owned storage. Full picture of
the root: `structs/ai_manager.md`.

**Two constant up-vectors sit just below this object and are NOT part of it** —
`g_vec_up_00655198_*` and `g_vec_up_00655208_*`, each a separately static-initialised `(0, 1, 0)`
(`crt_init_vec_up_00655198` / `..._00655208`). The first is the cross-product axis of the
`ai_path` corridor tests. Do not fold them into the singleton or into each other.

### 1a. Why the object has TWO copies of the query geometry

`+0x60`/`+0x6c` and `+0x00`/`+0x0c` both hold a start and an end position, which looks redundant
until you see who reads them.

The three callback-driven strategies (`ai_wayfind_plan_flee_route`,
`ai_wayfind_plan_directed_route_v2`, `ai_wayfind_plan_directed_route_v3`) hand **free functions** to
`web_path_find_path_to_goal_fn` as the goal predicate and the per-edge cost function — for the flee
route those are `ai_wayfind_cb_flee_goal_beyond_radius` (`0x00402dd0`) and
`ai_wayfind_cb_flee_edge_cost_directional` (`0x00402e80`). A free function has no `this`, so it
cannot reach the singleton through ECX. The strategies therefore **publish** the query geometry into
the object's first `0x40` bytes, which the callbacks reach by **absolute address**. That is directly
observable: the goal callback writes `[0x655248]`, `[0x65524c]` and `[0x655250]` — i.e. `this+0x24`,
`+0x28`, `+0x2c` — with no receiver in sight.

`ai_wayfind_plan_shortest_route` writes none of that block, because it uses the plain graph search
and passes no callbacks.

Ghidra renders the head of this object as loose globals and the tail as `this+N`, purely because of
how each is addressed. They are one object. The head globals are **named**, so a grep for
**`g_ai_wayfind_cb_`** finds the callback-block traffic that a `this+` grep misses, and vice versa:

| Field | Global | Written by |
|---|---|---|
| `+0x00`/`+0x04`/`+0x08` | `g_ai_wayfind_cb_start` | flee, directed v2, directed v3 |
| `+0x0c`/`+0x10`/`+0x14` | `g_ai_wayfind_cb_end` | directed v2 and v3 **only** |
| `+0x18`/`+0x1c`/`+0x20` | `g_ai_wayfind_cb_dir` | all three callback strategies |
| `+0x24`/`+0x28`/`+0x2c` | `g_ai_wayfind_cb_out` | **the goal callback itself** |
| `+0x30` | `g_ai_wayfind_cb_goal_dist_sq` **[H]** | directed v3 — `max(dist², param)`, a floor |
| `+0x38` | `g_ai_wayfind_cb_radius_sq` **[H]** | flee — `param²` capped at `400.0`, a ceiling · **also written** by `ai_wayfind_cb_corner_edge_cost` as a corner heuristic |
| `+0x3c` | `g_ai_wayfind_cb_forward_mode` **[H]** | mode selector, values 0/1/2; read by both forward callbacks |
| `+0x40` | `g_ai_wayfind_cb_target_node` **[H]** | directed v3 (the target's cached node `target+0x730`); `-1` from v2. Read by `ai_wayfind_cb_forward_edge_cost` |
| `+0x44` | `g_ai_wayfind_cb_target_next_node` **[H]** | directed v3 (`target+0x734` when `target+0x73c & 0x10`, else `+0x730`); `-1` from v2. Read by `ai_wayfind_cb_forward_edge_cost` |
| `+0x48` | `g_ai_wayfind_cb_field_0x48` **[H]** | a byte stored 0 by v2 and v3 next to the two node sentinels; no reader in `.text` |

**`g_ai_wayfind_cb_start` names address `0x00655224`, which is simultaneously the singleton
BASE.** The field name was chosen over an object name deliberately (user decision), so a reader who
hits `mov ecx, 0x00655224` will not see a symbol — this section is the record of that fact.

Both range scalars are **[H]**: both writers and both consumers are read.
`ai_wayfind_cb_flee_goal_beyond_radius` tests against `cb_radius_sq` and writes `cb_out_*`;
`ai_wayfind_cb_forward_edge_cost` reads `cb_goal_dist_sq`. That independently confirms the whole
callback-publication model — the free callbacks really do reach this block by absolute address, and
`cb_end_*` really is consumed only by the V2/V3 pair.

## 2. Method family — PC ↔ PS2 (18 of 19 mapped)

Ordered by **PC VA**, PC name first. Every row body-diffed on both builds.

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| `0x004035e0` | `ai_wayfind_build_path_node_from_nodes` | `GetPathNode__10ai_wayfindFP10C_web_nodeP10C_web_nodeP10C_web_nodeP12ai_path_node` (`0x00357f60`) | CONFIRMED | The three-node overload: builds one `ai_path_node` route step from the prev/cur/next node triple. Opens `if (prev == NULL) prev = cur;` — only a node-pointer overload can do that. Ratio 1.08 |
| `0x00403c30` | `ai_wayfind_build_path_node_from_positions` | `GetPathNode__10ai_wayfindFP10C_web_nodeRC8S_vectorRC8S_vectorP12ai_path_node` (`0x00358630`) | CONFIRMED | The `S_vector` overload. Decisive: the body **copies two 3-float vectors** out of params 2 and 3 into the out-record (`[9..0xb]`, `[6..8]`), which the node-pointer sibling cannot. Ratio 0.97 — below band; the param-type behaviour settles it |
| `0x00403ee0` | `ai_wayfind_build_optimal_path` | `GetOptimalPath__10ai_wayfindFv` (`0x00358a60`) | CONFIRMED | Allocates a fresh `ai_path` and computes the route for the node list held on the singleton; frees it and returns NULL when there is no list. **Sole `ai_path` allocator on each build**; ratio 1.50 |
| `0x00405510` | `ai_wayfind_resolve_start_node` | `FindStartCheckpoint__10ai_wayfindFR8S_vectorUib` (`0x0035abd0`) | CONFIRMED | Resolves a world position to a start node: 16 attempts, `-1.0f` radius, `+0.5`/`+0.2` probe offsets, LOS test, SELECTED-bit set/clear bracket, `0.0001` radius scale |
| `0x004059d0` | `ai_wayfind_resolve_end_node` | `FindEndCheckpoint__10ai_wayfindFR8S_vectorUib` (`0x0035b220`) | CONFIRMED | The end-side twin: same 16-attempt loop **without** the checkpoint-sweep preamble, then the neighbour-scan fallback. Field rebase PS2 `+0x98` ↔ PC `+0x80` |
| `0x00405dd0` | `ai_wayfind_get_node_for_position` | `GetPosition__10ai_wayfindFR8S_vectorPUiPUiUi` (`0x0035b6e0`) | CONFIRMED | Four statements, identical and in the same order on both builds. Pins the `+0x18` class-wide field rebase |
| `0x00405e10` | `ai_wayfind_plan_shortest_route` | `FindWaySHORTEST__10ai_wayfindFR8S_vectorR8S_vectorUib` (`0x0035b740`) | CONFIRMED | **Strategy 1/6.** The only strategy using the plain graph search. Statement-for-statement: resolve both ends → validity guard → `0x29a` sentinel test → SELECTED bracket → `web_path_find_path` |
| `0x00406040` | `ai_wayfind_plan_route_via_corner` | `FindWayCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorUiP19ai_corner_hide_infob` (`0x0035b9e0`) | CONFIRMED | **Strategy 2/6.** Scans the **`check_seg` segment store** for **type-2 specials (`ai_seg_corner` cover corners)** and routes via the nearest usable one, tested by `ai_wayfind_test_corner_hide_side`. The loop is `check_seg_clusters_get_segment*` → `check_seg_get_special*` → `if (type == 2)` (`structs/check_seg.md` §1d/§3). Identical normalise ladder; arity 5↔5 |
| `0x00406550` | `ai_wayfind_plan_flee_route` | `FindWayRUNAWAY_V1__10ai_wayfindFR8S_vectorR8S_vectorfUib` (`0x0035c070`) | CONFIRMED | **Strategy 3/6 — the flee route.** Start node only, end forced to 0, direction published, goal+cost callbacks. Separated from the V2 sibling by publishing **ONE** position and **squaring then clamping** its float at `400.0` |
| `0x00406930` | `ai_wayfind_plan_directed_route_v2` | `FindWayFORWARD_V2__10ai_wayfindFR8S_vectorR8S_vectorfUib` (`0x0035c4c0`) | CONFIRMED | **Strategy 4/6.** Separated from the flee route (identical PS2 signature) by publishing **TWO** positions and storing its float **raw**, plus a different callback pair |
| `0x00406de0` | `ai_wayfind_plan_directed_route_v3` | `FindWayFORWARD_V3__10ai_wayfindFR8S_vectorR8S_vectorP7C_humanP7C_humanfUib` (`0x0035ca80`) | CONFIRMED | **Strategy 5/6.** Same publication + callback pair as V2, plus two `C_human*` participants and the `FindPathCompFuncJIPO` search variant. Arity 7↔7, unique in the gap, positional types exact |
| `0x004072c0` | `ai_wayfind_plan_search_route` | `FindWaySEARCH__10ai_wayfindFP15ai_check_searchUiUib` (`0x0035d140`) | CONFIRMED | **Strategy 6/6.** Start node only, then flood the web via `ai_wayfind_score_and_add_checkpoint` and route with the **plain** search — that is what separates it from strategies 3–5. Arity 4↔4 |
| `0x00407620` | `ai_wayfind_test_corner_hide_side` | `hasCornerVisHide__10ai_wayfindFP13ai_seg_cornerR8S_vectorR8S_vectorR8S_vectorPUibf` (`0x0035d4c0`) | CONFIRMED | Corner-usability predicate that also writes the 0/1 side selector. Shared magic `9.0`, `1.25`, `1e-08`; three signed-2D-angle calls; LOS test. Arity 7↔7 |
| `0x0040d4e0` | `ai_wayfind_find_nearest_hide_corner` | `FindCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorP19ai_corner_hide_info` (`0x00366360`) | CONFIRMED | Scans the map's segment clusters for cover corners, tests each with `ai_wayfind_test_corner_hide_side` and fills the caller's `ai_corner_hide_info`. The helper behind strategy 2. Callee set exact incl. `hasCornerVisHide` ×1 ↔ the confirmed anchor ×1; 3 args ↔ 3 |
| `0x0040d700` | `ai_wayfind_score_and_add_checkpoint` | `WeightCheckpoints__10ai_wayfindFP15ai_check_search…UiUiUiUi` (`0x00366750`) | CONFIRMED | The flood-fill step of strategy 6: dedup-guard, append to the node vector, then gate on filter mask / start radius² / target radius² / forward-facing. Arity 6↔6, ratio 1.40. Free callee-harvest |
| `0x0040dc10` | `ai_wayfind_compute_2d_line_intersection` | `Compute2DIntersection__10ai_wayfindFP9S_vector2×5` (`0x00366ec0`) | CONFIRMED | Statement-for-statement 2-D line/line intersection. Every param indexed `[0]`/`[1]` only ⇒ five `S_vector2*`. PC's `0x322bcc77` **is** `1e-08` as a float. Arity 5↔5, ratio 1.31 |
| `0x0040dc90` | `ai_wayfind_compute_node_left_right_gate` | `GetLR__10ai_wayfindF…P12ai_path_nodefff` (`0x00366f70`) | CONFIRMED | Builds the left/right gate pair around a path node — the pair a normal waypoint stores at record `+0x00`/`+0x0c`. `Compute2DIntersection` ×2 + `AngleTo2` ×1 exact; shared `0.707`, `1.2`, `0.5`×6. Arity 7↔7, ratio 1.14. The "gate" noun is inferred: the 3008 B body was fingerprinted, not statement-read |
| `0x0040e850` | `ai_wayfind_compute_nearest_point_on_segment` | `ComputeNearestPoint__10ai_wayfindF…ff` (`0x00367ce0`) | CONFIRMED | Projects a point onto an X/Z segment, lerps two offsets by the projection ratio, steps perpendicular via an identical sign cascade; Y forced to 0. Arity 6↔6, ratio 0.86 — below band |

The one unmapped PS2 method is **`isInBound`** (260 B) — an open hunt, §6.

### 2a. PC-only members

Three PC functions belong to the class and have no PS2 twin, because the PS2 build inlines all
three. Their empty twin cell is a RESULT, not a gap: none of the remaining unmapped PS2 methods is
an accessor or a destructor.

| PC VA | PC name | PS2 twin | What it does |
|---|---|---|---|
| `0x004256e0` | `ai_wayfind_set_checkpoint_container` | - | one-line accessor: installs the caller's `blocked_con` list at `+0x5c` |
| `0x004256f0` | `ai_wayfind_clear_checkpoint_container` | - | one-line accessor: releases `+0x5c` |
| `0x00443840` | `ai_wayfind_dtor` | - | inlined into `__dt__10ai_managerFv`; `operator_delete` on the buffer at `+0x50`, then zeroes `+0x50`/`+0x54`/`+0x58` |

The accessor usage shape comes from `entity_action_tick` case 0:
**set(`entity+0x1198`) → `ai_wayfind_plan_shortest_route` → clear()** — the caller installs its own
blocked-connection list into the planner for the duration of one plan, then releases it.

## 3. Routing policy

### 3a. The six route strategies at a glance

| # | PC name | Endpoints resolved | Search used | Distinguishing state |
|---|---|---|---|---|
| 1 | `ai_wayfind_plan_shortest_route` | start + end | plain `web_path_find_path` | publishes nothing |
| 2 | `ai_wayfind_plan_route_via_corner` | start + end | cost-fn search | scans the `check_seg` segment store for type-2 `ai_seg_corner` cover corners |
| 3 | `ai_wayfind_plan_flee_route` | start only | goal + cost callbacks | ONE position; radius² clamped at `400.0` |
| 4 | `ai_wayfind_plan_directed_route_v2` | start only | goal + cost callbacks | TWO positions; raw float |
| 5 | `ai_wayfind_plan_directed_route_v3` | start only | `FindPathCompFuncJIPO` | TWO positions + two `C_human*` |
| 6 | `ai_wayfind_plan_search_route` | start only | plain `web_path_find_path` | floods the web to *discover* the end |

Strategies 3–5 force `end_node = 0` because they have no destination node — the goal predicate
decides when to stop.

### 3b. The five route-cost callbacks

These free functions **are** the routing policy — the goal predicate decides when a search stops,
the cost function decides which way it prefers to go. Every one is pinned by **its consumer on
both builds**, and the consumers are already-confirmed twins; that is stronger than any size ratio.
Ghidra renders each **definition** as a function and each **reference in a caller** as `&LAB_…`,
which is why a callsite alone can look like a missing function.

| PC VA | PC name | PS2 twin | Used by | Role |
|---|---|---|---|---|
| `0x00402ce0` | `ai_wayfind_cb_corner_edge_cost` | `fxForwardShortest__FUsUsUc` | corner | cost only. The PS2 name says "ForwardShortest" but SHORTEST passes **no** callbacks |
| `0x00402dd0` | `ai_wayfind_cb_flee_goal_beyond_radius` | `fxRunawayEndFuncV1__FUs` | flee | stop once a node is further than `cb_radius_sq` from the published start |
| `0x00402e80` | `ai_wayfind_cb_flee_edge_cost_directional` | `fxRunawayFuncV1__FUsUsUc` | flee | `(2 − dot(cb_dir, edge_dir))² × edge_len` ⇒ aligned ×1, perpendicular ×4, reversed ×9 |
| `0x00403080` | `ai_wayfind_cb_forward_goal_by_mode` | `fxForward2EndFuncV1__FUs` | v2 **and** v3 | returns 0/3/1 from `g_ai_wayfind_cb_forward_mode`; **ignores the node index** |
| `0x004030a0` | `ai_wayfind_cb_forward_edge_cost` | `fxForward2FuncV1__FUsUsUc` | v2 **and** v3 | distance-derived cost; consumes `cb_end_*` and `cb_goal_dist_sq` |

All five are named on PS2 as FREE functions (`fxRunawayFuncV1__FUsUsUc` and siblings), so their
`class` column is empty and a class-filtered query over the PS2 symbols will not return them.

**The flee policy, in full:** run in the published direction (cost bias) until you are further than
the radius from where you started (goal predicate). Two small functions, and that is the whole
behaviour.

## 4. PC vs PS2

- **`ai_wayfind` and `ai_path` interleave in ONE PS2 translation unit** (`0x00357f60`–`0x00368940`),
  so they share one PC cluster and one emission order. Order-align must treat them as a single
  sequence, not two.
- **The class anchor is an allocation-uniqueness argument**, not a body diff of an 8.5 KB function:
  `ai_path` is allocated in **exactly one place on each build** — PC `operator_new(0x9c)` inside
  `ai_wayfind_build_optimal_path`, PS2 `__nw__7ai_path(0xa0)` inside `GetOptimalPath__10ai_wayfindFv`.
  One callsite each, whole binary. Both then zero three vector triples in the fresh object and walk
  the nav web. Size ratio 8548 / 5680 = **1.50**, in band.
- **Receiver shape matches:** PS2 `GetOptimalPath` is `Fv` (implicit `this` only); the PC twin takes
  its receiver in ECX as an absolute global, which is what a singleton method looks like.
- **The nine `ai_wayfind_*` planner methods were each body-diffed** before being attributed to this
  class. A forced 10↔10 order-align gap between `ai_wayfind_build_optimal_path` (PS2 rank 3) and
  `ai_path_destroy` (PS2 rank 14) generated the candidates; the gap was enumerated as Ghidra
  `FUN_` ids ∪ named defs and Ghidra's default analysis returned exactly the same 11 starts, so no foreign function
  shifts the ranks.
- **The tail of the cluster is only half forced.** `ai_wayfind_score_and_add_checkpoint`
  (PS2 rank 33) sits inside it and splits it: the sub-gap **below** that anchor is a forced 18↔18,
  while the sub-gap **above** it is genuinely non-forced (23 PC starts vs 11 PS2 methods, two of
  them PS2-only). Anything still open up there needs callee-harvest, never interpolation.
- **PS2-only, near-certain:** `ai_path`'s class-specific `operator new` / `operator delete` (pool
  allocation via the `ai_spageman` family) — PC uses the global `operator_new` here.
- **The singleton's PS2 base is `0x005090b0`** [M]. Inferred, not probed: `FindStartCheckpoint`
  writes `this+0x00`, `FindEndCheckpoint` writes `this+0x10`, and `FindWayRUNAWAY_V1` /
  `FindWayFORWARD_V2` write the absolute globals `DAT_005090b0` / `DAT_005090c0` respectively — the
  same two fields. `GetPosition` writes `this+0x90`/`+0x94` ↔ PC `+0x78`/`+0x7c`, giving the class
  rebase delta of **`0x18`** for the tail band.
- **The callback publication block spacing differs by exactly the `S_vector` padding.** PS2 vectors
  are 4 floats (16 B), PC's are 3 (12 B), so the block strides `b0 / c0 / d0` on PS2 against
  `224 / 230 / 23c` on PC. That the deltas shrink by 4 per successive vector is an independent
  cross-check on the whole block and on the base addresses.
- **A real factoring difference:** on PS2 the resolvers write the SAME fields the callbacks read
  (`this+0x00` / `this+0x10`); on PC the resolvers keep their own copy at `this+0x60` / `this+0x6c`
  and the strategies publish a second copy into the callback block. PC therefore stores the query
  geometry twice where PS2 stores it once. Recorded, not explained.
- **`hasCornerVisHide`'s corner field index shifts by one float** — PS2 reads `corner[0xb]` where PC
  reads `corner[10]`, because a leading `S_vector` is 4 floats on PS2 and 3 on PC. Same cause as the
  block spacing above.
- **`+0x48` is dead on both builds.** PS2 `FindWayFORWARD_V2` and `_V3` store `AIManager+0x88` = 0
  right after the two node sentinels (`AIManager+0x80`/`+0x84` = -1), the same three statements as
  the PC's `+0x40`/`+0x44`/`+0x48`, and nothing on the PS2 reads that byte either.
- **`FindPathCompFuncJIPO`** exists on both builds as a distinct search entry point used only by
  `FindWayFORWARD_V3`; it is not a PS2-only symbol.
- **`ai_corner_hide_info`'s member order differs between builds** — PC puts the corner pointer
  first, PS2 puts it after both vectors, for VU quadword alignment. That is a reordering, not a
  rebase; see `structs/check_seg.md`.

## 5. Related

- `structs/ai_path.md` — the object this class produces; same cluster.
- `structs/web_path.md` — the graph underneath.
- `structs/web_node.md` — the node records the indices point at.
- `structs/ai_manager.md` — the root object this singleton is a member of.
- `structs/ai_action_manager.md` — the consumer.
- `structs/check_seg.md` §1f / §1g — `ai_seg_corner` (`0x38`) and `ai_corner_hide_info` (`0x24`),
  the two records strategy 2 works with. `ai_seg_corner`'s `radius` sits at PC `+0x24`, the same
  float `check_seg_is_pos_in_any_special` reads as the type-2 sphere radius — two unrelated
  functions agreeing on one offset is what byte-proved the field.
- `structs/ai_check.md` §6 — the `ai_path_find_spec_node_*` searches and their bound-points calls.

## 6. Open items

| item | state |
|---|---|
| `isInBound` (260 B), the one unmapped PS2 method | see the evidence block below; recorded as open, **not** tagged inlined |
| is the `+0x4c` vector the request's checkpoint list or the resulting route? | one `std::vector<uint>` of node indices serves both roles in the observed traffic: the plan strategies fill it (strategy 6 through `ai_wayfind_score_and_add_checkpoint`) and `ai_wayfind_build_optimal_path` consumes it. Which name is the developers' is not established |
| singleton interior `+0x28..+0x38`, and everything past `+0x84` | not mapped |
| `+0x48` | written 0 before every directed search and never read, on both builds (§4) |

**`isInBound` — what is known and what is ruled out.**

On PS2 it is a **point-in-polygon test** over a `std::vector<S_vector2>` global (`0x509090`,
requires ≥3 points) with **exactly 6 callsites — ×2 inside each of the three
`ai_path_find_spec_node_*` searches**. Its signature,
`isInBound__10ai_wayfindFPQ23std48vector<9S_vector2,…>R8S_vector`, takes **exactly the vector that
`GetBoundPoints2D` fills**, and that free function is named on PC:
**`i3d_frame_get_bound_points_2d`** (`0x004029c0`), called ×4 by each of the three
`ai_path_find_spec_node_*` searches. So the PS2 idiom is a *pair* — build the 4 bound points of a
frame's footprint, then test a position against them.

| candidate / test | outcome |
|---|---|
| a PC callee appearing ×2 in the three `ai_path_find_spec_node_*` bodies | none |
| an inlined equivalent with a `< 3` guard or a polygon global on PC | none. That fingerprint could not match: on PS2 the point list is a global, on PC it is a **caller local** filled by `i3d_frame_get_bound_points_2d` |
| `ai_path_is_point2d_in_checkpoint_radius` — ×2 per spec-node search, sits right after the bound-points build, indexes its second argument with stride 8 (`S_vector2`) | **refuted by the body**: it computes `dx² + dz²` against `param_3²`, a genuine **radius** test, and it has 10 callers. Its PC name is accurate and must not be changed |
| PS2-port-only subsystem | does not apply: its callers are ordinary `ai_path` methods |

Next step: re-search the three PC `ai_path_find_spec_node_*` bodies for an inline point-in-quad test
over the local vector, immediately after each `i3d_frame_get_bound_points_2d` call. That would also
explain the **×5 PS2 vs ×4 PC** call-count mismatch on `i3d_frame_get_bound_points_2d`
(`structs/ai_check.md` §6): if PC merged one bound-points build with the containment test, one call
would disappear. Still a hypothesis.
