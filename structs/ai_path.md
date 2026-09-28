# `ai_path` — one computed route, handed to an NPC action

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-09-17 17:54 +0200

An `ai_path` is a **route object**: the result `ai_wayfind` produces and the thing the movement
`ai_action` factories carry. It sits one layer above the raw graph — `ai_path` consumes
`C_web_path` (`structs/web_path.md`) node indices, and is consumed by `ai_action_manager`
(`structs/ai_action_manager.md`), whose movement factories take an `ai_path*` parameter.

| | |
|---|---|
| PC prefix | `ai_path_*` |
| Allocated | `operator_new(0x9c)` — **exactly one allocation site in the PC binary**, inside `ai_wayfind_build_optimal_path` |
| Methods | 25 PS2 methods: 23 mapped, 2 PS2-only — all accounted (§2) |
| PC clusters | `[0x004035e0, 0x0040f760)` (shared with `ai_wayfind`, one PS2 translation unit) · the destructor helpers at `0x004257e0`+ · four inline accessors in the `entity_action_tick` inline-accessor block `0x00424e20..0x00425850` (§2a) |

Confidence: **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Object layout (object size `0x9c`)

```c
typedef struct ai_path {                     // 0x9c bytes
    /* +0x00 */ uint32_t    _unk_00;         // [U]
    /* +0x04 */ waypoint   *normal_begin;    // [H] NORMAL waypoint container, stride 0x3c.
    /* +0x08 */ waypoint   *normal_end;      //     The route proper - see §1a.
    /* +0x0c */ waypoint   *normal_cap;      // [H]
    /* +0x10 */ uint32_t    _unk_10;         // [U]
    /* +0x14 */ float      *spec_begin;      // [H] SPEC waypoint container, stride 0x18.
    /* +0x18 */ float      *spec_end;        //     Detour points; rebuilt by
    /* +0x1c */ float      *spec_cap;        // [H] ai_path_update_spec_waypoints.
    /* +0x20 */ uint32_t    _unk_20;         // [U]
    /* +0x24 */ passside   *side_begin;      // [H] PASS-SIDE container, stride 0x10.
    /* +0x28 */ passside   *side_end;        //     Element +8 = the side selector;
    /* +0x2c */ passside   *side_cap;        // [H] ai_path_force_pass_side_right sets them all
                                             //     to 2.
    /* +0x30 */ blocked_con blocked;         // [H] 0x10 bytes, see §1c
    /* +0x40 */ float       last_actor_pos[3]; // [H] the actor position cached at the end of the
                                             //     previous ai_path_revalidate_for_actor. A move
                                             //     of >4.0 units^2 since then = teleport =>
                                             //     state 4.
    /* ...  */
    /* +0x58 */ int32_t     normal_cursor;   // [H] index into the +0x04 container; -1 = reset
    /* +0x5c */ int32_t     spec_cursor;     // [H] index into the +0x14 container; -1 = reset
    /* +0x60 */ int32_t     delay_until;     // [H] mission_clock + 1000 ms, armed by
                                             //     ai_path_arm_delay_timer
    /* +0x64 */ int32_t     no_progress;     // [H] the blocked/no-progress accumulator. +3 below
                                             //     speed 0.1, +1 up to 0.5, -1 above, reset to 0
                                             //     above 0.9. Gates at 4/10/15/25/50/100.
    /* +0x68 */ C_actor    *owner;           // [H] cleared by ai_action_invalidate_path_for_actor
                                             //     when it matches the actor being destroyed
    /* ...  */
    /* +0x71 */ uint8_t     blocked_flag;    // [H] set when a blocker is standing in the way,
                                             //     cleared when no_progress falls back to 0
    /* +0x74 */ float       staged_point[3]; // [H] the current waypoint's own position
                                             //     (waypoint +0x24), staged by every getter
    /* +0x80 */ float       staged_target[3];// [H] where the actor is actually steering: the
                                             //     midpoint of the waypoint's gate pair (type 0)
                                             //     or the web node position (types 1/2)
    /* +0x8c */ uint32_t    state;           // [H] 0 = none · 1 = following a normal waypoint ·
                                             //     2 = following a spec waypoint ·
                                             //     3 = invalidated · 4 = abandoned
    /* +0x90 */ float       tolerance_sq;    // [H] arrival radius^2 for staged_target
    /* ...  */
    /* +0x98 */ void       *cached_blocker;  // [H] the TYPE-1 SPECIAL of the map segment
                                             //     covering the current node, cached by
                                             //     ai_path_find_segment_for_node - a piece of
                                             //     geometry bound to a live scene frame
                                             //     (structs/check_seg.md §1d, §3)
} ai_path;                                   // 0x9c
```

### 1a. The NORMAL waypoint record (stride `0x3c`)

The route proper. Offsets are inside the element.

| offset | type | purpose | conf |
|---|---|---|---|
| `+0x00` | `float[3]` | gate point **A** | H |
| `+0x0c` | `float[3]` | gate point **B** — the midpoint of A/B is the steering target for type 0 | H |
| `+0x24` | `float[3]` | the waypoint's own position; `ai_path_get_destination` returns this field of the **last** element as the route destination | H |
| `+0x34` | `uint32` | **`C_web_node` index** — resolved against `*(g_act_mission+0x2c)+4` at stride `0x1e` | H |
| `+0x38` | `uint32` | **type discriminator**: `0` = free midpoint between the gate pair · `1`/`2` = snap to the web node at `+0x34` | H |

### 1b. The normal-vs-spec split

- **NORMAL waypoints** are the route: full records with a gate pair, a node reference and a type.
  Produced by `ai_wayfind`, consumed by `ai_path_advance_to_next_normal_waypoint` /
  `ai_path_load_normal_waypoint`, bounded by `ai_path_is_last_normal_waypoint`.
- **SPEC waypoints** are *detours*: bare position triples, rebuilt on demand by
  `ai_path_update_spec_waypoints` from the three `ai_path_find_spec_node_*` searches when the
  direct line is blocked. Consumed by `ai_path_advance_to_next_spec_waypoint`.
- `ai_path_update_spec_waypoints` returns **0 / 1 / 2**, and that return value is the switch
  between the two modes — both per-frame entry points use it to decide which getter to call next.
  `state` (`+0x8c`) records the answer as 1 or 2.

The three spec-node searches test route candidates against scene objects' **footprints**:
`i3d_frame_get_bound_points_2d` returns the 4 world-space corners of an `I3D_frame`'s 2D bounding
rectangle, inflated `0.7`, at mid-height (`structs/ai_check.md`).

### 1c. `+0x30..+0x3f` — an embedded `blocked_con` record

The element type name comes from the signature `AddBlockedCon__8C_entityFP11blocked_con`.

| offset | field | conf | purpose |
|---|---|---|---|
| `+0x30` | `node_a` | H | a `C_web_node` id, taken from the current waypoint's `+0x34` |
| `+0x34` | `node_b` | H | the same id — the searches write both slots from one source |
| `+0x38` | `0x29a` sentinel | H | the project-wide 666 marker; `entity_add_blocked_con` **gates on `param_2[2] == 0x29a`**, exactly this slot, to tell a node-pair block from a positional one |
| `+0x3c` | `expiry_ms` | H | `game_frame_time_ms + 30000`, or `+ 1000` when the blocker is a kind-`0x1b` entity (which is also sent event 8); `ai_wayfind_build_optimal_path` zeroes it on a fresh route |

**How it is written.** The three `ai_path_find_spec_node_*` searches fill it at **six sites** with
one identical sequence:

```c
state(+0x8c) = 3;                       // INVALIDATED
+0x3c = 0;
+0x30 = +0x34 = <web_node id>;
+0x38 = 0x29a;
+0x3c = game_frame_time_ms + 30000;     // or + 1000
```

**How it is consumed.** In `entity_action_tick` case 0, when the route is invalidated and
`ai_path_has_blocked_con` reports a live record, the block is published with
`entity_add_blocked_con(path + 0x30)` before the route is torn down and replanned — so the NPC
avoids that graph edge until the deadline passes. `entity_add_blocked_con` walks the entity's
`+0x119c` list at **stride `0x10`**, exactly this record's size.

## 2. Method family

Every row with a PS2 twin is body-diffed on both builds. Ratio = PS2 size / PC size.

| PC VA | PC name | PS2 twin | status | role / decisive evidence |
|---|---|---|---|---|
| `0x00407920` | `ai_path_destroy` | `Destroy__7ai_pathFv` | CONFIRMED | resets the path: collapses the three containers and writes `-1` to the `+0x5c`/`+0x58` pair, high dword first ¹ |
| `0x004079b0` | `ai_path_find_spec_node_from_coll_list` | `GetSpecNodeA__7ai_pathF…RQ23gif31vector<P18g_collision_header,8>…` | CONFIRMED | widest detour search — clip plane **and** a pre-gathered collision list; `isInConnection` ×2; constants `9.0`×2 + `0.01`×4; ratio 1.57 |
| `0x00408e70` | `ai_path_find_spec_node_from_plane` | `GetSpecNodeB__7ai_pathF…R7S_planeP7C_actorUiP7C_actor` | CONFIRMED | middle sibling — clip plane only, gathers its own collisions; neither containment call; constants `2.25`×1 + `0.75`×2; ratio 1.61 |
| `0x0040a0c0` | `ai_path_find_spec_node_from_positions` | `GetSpecNodeC__7ai_pathFR8S_vectorR8S_vectorP7C_actorUiP7C_actor` | CONFIRMED | narrowest — positions only; `isInCheckpoint` ×2; arity `this` + 6 floats + `C_actor*` + `uint` + `C_actor*`; ratio 1.63 |
| `0x0040b390` | `ai_path_update_spec_waypoints` | `UpdateSpec__7ai_pathFR8S_vectorbP7C_actor` | CONFIRMED | rebuilds the spec container; callees: the three searches ×1 + `DynSetNextCollOwner` ×2 + `SetState` ×3; returns 0/1/2; ratio 1.18 |
| `0x0040bca0` | `ai_path_advance_to_next_normal_waypoint` | `GetNextNormalWaypoint__7ai_pathFR8S_vectorb` | CONFIRMED | advances the normal cursor then stages; `GetSegments` ×1 + `isInNextConn` ×1; arity 3↔3; ratio 1.43 |
| `0x0040bf30` | `ai_path_load_normal_waypoint` | `GetNormalWaypoint__7ai_pathFR8S_vector` | CONFIRMED | non-advancing sibling: tests `cursor < count`, not `count-1`, never calls `GetSegments`; ratio 1.32 |
| `0x0040c1c0` | `ai_path_advance_to_next_spec_waypoint` | `GetNextSpecWaypoint__7ai_pathFR8S_vector` | CONFIRMED | statement for statement; frees the container and resets the cursor at the end of the list; both builds ignore the declared out-param; ratio 1.25 |
| `0x0040c260` | `ai_path_is_last_normal_waypoint` | `isLastNormalWaypoint__7ai_pathFv` | CONFIRMED | identical predicate and signed-division idiom; ratio 1.69 |
| `0x0040c2a0` | `ai_path_is_pos_in_connection_corridor` | `isInConnection__7ai_pathF…RC8S_vectorff` | CONFIRMED | trapezoid corridor, 4 signed-cross side tests; probe indexed `[2]` = z ⇒ the 3-D overload; shared `1e-08`; ratio 1.31 |
| `0x0040c5c0` | `ai_path_is_point2d_in_connection_corridor` | `isInConnection__7ai_pathF…RC9S_vector2ff` | CONFIRMED | same geometry, probe `[1]` = y ⇒ the `S_vector2` overload, plus endpoint discs (a capsule); `isInCheckpoint` ×2; ratio 1.28 |
| `0x0040c930` | `ai_path_is_point2d_in_checkpoint_radius` | `isInCheckpoint__7ai_pathFRC8S_vectorRC9S_vector2f` | CONFIRMED | one expression, same 2-D/3-D index asymmetry; ratio 1.00 ² |
| `0x0040c970` | `ai_path_tick` | `Tick__7ai_pathFUiP7C_humanbb` | CONFIRMED | the per-frame **advance** half (§3); identical 7-callee set; arity 5↔5; ratio 1.57 |
| `0x0040cca0` | `ai_path_revalidate_for_actor` | `Update__7ai_pathFP7C_actor` | CONFIRMED | the per-frame **revalidate** half (§3); exact 4-callee multiset; arity 2↔2; ratio 1.04 |
| `0x0040d2c0` | `ai_path_get_destination` | `GetDestination__7ai_pathFR8S_vector` | CONFIRMED | returns waypoint `+0x24` of the last normal element; ratio 1.30 |
| `0x0040d310` | `ai_path_arm_delay_timer` | `DelayTest__7ai_pathFv` | CONFIRMED | one statement: `+0x60 = mission_clock + 1000`; the PS2 name says "Test", the body only writes |
| `0x0040d330` | `ai_path_must_evade` | `mustEvade__7ai_pathFP7C_actorR9S_vector2R9S_vector2` | CONFIRMED | **constant TRUE on both builds**; arity 3↔3 (`ret 0xc`); called ×1 in each spec-node search |
| `0x0040d340` | `ai_path_find_segment_for_node` | `GetSegments__7ai_pathFUib` | CONFIRMED | seven-callee multiset matches 1:1; caches `+0x98`; arity 3↔3; ratio 1.35 |
| `0x0040db30` | `ai_path_do_connections_overlap` | `areOverlaped__7ai_pathF…ffff` | CONFIRMED | shared `0.088` + `3.1415927` + `1e-08`×2; arity 7↔7; ×1 from `ai_wayfind_build_optimal_path` |
| `0x0040ea20` | `ai_path_force_pass_side_right` | `ForceRight__7ai_pathFv` | CONFIRMED | statement-identical loop writing `2` into element `+8` of the third container, stride `0x10` |
| `0x0040ea50` | `ai_path_is_pos_in_next_connection` | `isInNextConn__7ai_pathFR8S_vector` | CONFIRMED | three confirmed callers agree at ×1 each; arity 2↔2 ³ |
| `0x0040ebb0` | `ai_path_find_blocking_dyn_object` | `TestDetailLines__7ai_pathF…PP18tDynamicCollObject` | CONFIRMED | six horizontal dynamic-collision line probes; reports the blocker through a `tDynamicCollObject**`; forced 1↔1 by position, plus `collision_test_line_horizontal_dynamic` ×6; shared `0.4`×3 / `1.8` / `1.5` / `0.3` / `0.2`; arity 4↔4; ratio 1.23 |
| `0x0040ee50` | `ai_path_is_turning_at_normal_waypoint` | `isTurningNormalNode__7ai_pathFR8S_vector` | CONFIRMED | incl. the `cursor < count-2` guard; true when two signed angles have a **negative** product (the route zig-zags); ratio 1.87 |
| `0x004256a0` | `ai_path_is_invalidated` | - | STRUCTURAL ⁴ | `state (+0x8c) == 3`; 46 references |
| `0x004256b0` | `ai_path_is_abandoned` | - | STRUCTURAL ⁴ | `state (+0x8c) == 4`; 34 references |
| `0x004256c0` | `ai_path_has_blocked_con` | - | STRUCTURAL ⁴ | `+0x3c != 0` — this route recorded a blocked connection (§1c) |
| `0x004256d0` | `ai_path_get_state` | - | STRUCTURAL ⁴ | raw `state (+0x8c)` read; 30 references |
| `0x004257e0` | `ai_path_scalar_deleting_dtor` | - | STRUCTURAL | MSVC-only: `{ ai_path_free_buffers(this); if (flags & 1) operator_delete(this); return this; }` |
| `0x00425800` | `ai_path_free_buffers` | - | STRUCTURAL ⁴ | the `~ai_path` body: `operator_delete` on the three buffers in **reverse** order (`+0x24`, `+0x14`, `+0x04`), zeroing each triple |

¹ PS2 also frees here; the PC splits the free into `ai_path_free_buffers`.
² "checkpoint" is backed by the consumers, not the body.
³ It calls the corridor test ×1 on PC and ×2 on PS2 (§6).
⁴ Inline on PS2, so no symbol.

`copy_elem_stride_0x18` (`0x0040f6e0`, 32 bytes) is the spec-waypoint element copy, not a class
method.

### 2a. The four inline state accessors

`ai_path_is_invalidated`, `ai_path_is_abandoned`, `ai_path_get_state` and
`ai_path_has_blocked_con` live in the `entity_action_tick` inline-accessor block
(`0x00424e20..0x00425850`). That band is one translation unit's inline methods, so its members
belong to many classes (`ai_path`, `ai_wayfind`, `ai_groups`, `C_game`, `C_Vehicle`, `C_car`,
`C_actor`, `C_entity`) and VA proximity says nothing about the receiver. The receiver is `ai_path`
by two independent measurements:

1. **Callsite receivers** — every callsite loads ECX as `mov ecx,[edi+0x10 / 0x14 / 0x1c / 0x20 /
   0x24 / 0x30 / 0x38 / 0x50 / 0x5c]`, one offset per `ai_action` record type, with an identical
   distribution across all three state accessors.
2. **Control flow** — in `entity_action_tick` case 0, the same pointer field is torn down three
   statements later by `ai_path_destroy()` + `ai_path_scalar_deleting_dtor(1)`.

## 3. The two per-frame entry points

`ai_path` is driven by **two** methods every frame:

| | `ai_path_tick` | `ai_path_revalidate_for_actor` |
|---|---|---|
| job | **advance** along the route | **revalidate** the route |
| arity | `(this, uint, C_human*, bool, bool)` | `(this, C_actor*)` |
| always does | picks the next waypoint and stages it | re-runs `ai_path_update_spec_waypoints` and re-stages |
| can set state 4 | no | **yes** — give-up or teleport |

`ai_path_revalidate_for_actor` — the guard ladder in front of its unconditional tail:

1. Bail if `state` ∈ {3, 4}.
2. Maintain `no_progress` (`+0x64`) from the actor's speed.
3. `no_progress > 4` ⇒ swept-sphere probe (r = 0.2) ahead; cache the blocker.
4. With a blocker, escalate **by blocker type**: `>10` force the pass side right · `>15` strafe
   around · type 6 ⇒ `human_use_actor` (open the door) · types 2/`0x15` `>100` ⇒ event 7 · type
   `0x1b` `>25` ⇒ event 8, `>100` ⇒ event 7 + reschedule · default `>50` ⇒ **state 4**.
5. Moved `> 4.0` units² since `last_actor_pos` ⇒ teleport ⇒ **state 4**.
6. Otherwise cache the position and fall through to the tail.

## 4. Related

- `structs/ai_wayfind.md` — the producer; same PS2 translation unit and PC cluster.
- `structs/web_path.md` — the graph underneath.
- `structs/ai_action_manager.md` — the consumer (`ai_path*` on every movement factory).
- `structs/check_seg.md` — the map segments and specials cached at `+0x98`.
- `structs/ai_check.md` — `i3d_frame_get_bound_points_2d`.

## 5. PC vs PS2

- **Object size: PC `0x9c`, PS2 `0xa0`.** Both allocate it in exactly one place (PC
  `operator_new(0x9c)` in `ai_wayfind_build_optimal_path`; PS2 `__nw__7ai_path(0xa0)` in
  `GetOptimalPath__10ai_wayfindFv`), which anchors the class.
- **Field rebase — band by band, never by a single offset** (the delta is not constant):

  | field | PC | PS2 |
  |---|---|---|
  | normal waypoints | `+0x04`, stride `0x3c` | `+0x40`, stride `0x50` |
  | spec waypoints | `+0x14`, stride `0x18` | `+0x4c`, stride `0x20` |
  | pass-side entries | `+0x24`, stride `0x10` | `+0x58`, stride `0x10` |
  | normal cursor | `+0x58` | `+0x74` |
  | spec cursor | `+0x5c` | `+0x78` |
  | delay timer | `+0x60` | `+0x7c` |
  | state | `+0x8c` | `+0x90` |
  | waypoint position (in-element) | `+0x24` | `+0x30` |

  PC `+0x58` is the normal cursor while PS2 `+0x58` is the pass-side container; use the table.
- **Only the third container has the same stride on both builds** (`0x10`). The first two differ
  because a PS2 `S_vector` is 4 floats and a PC one is 3.
- **Factoring differences:** PS2's `Destroy` both clears and frees; PC splits it into
  `ai_path_destroy` + `ai_path_free_buffers`. PS2's `Tick` issues ~2 more calls than PC's on three
  of its seven callees (GCC duplicating switch-arm tails). PC collapses the vector arithmetic in
  `isTurningNormalNode` into two argument-less FPU helper calls, hence ratio 1.87.
- **PS2-only (2):** the class-specific `operator new` / `operator delete` — `ai_path` is
  pool-allocated on PS2 via the `ai_spageman` family; PC calls the global `operator_new`.
- **Shared magic constants:** `0.088` + `3.1415927` (`areOverlaped`), `1e-08` (the whole normalise
  ladder), `+1000` (`DelayTest`), `9.0`/`0.01`/`0.75`/`2.25`/`0.3` (the per-sibling `GetSpecNode`
  fingerprints), `4.0` (the teleport threshold), `0.5`/`0.9` (the speed gates).
- **The `GetSpecNode` siblings** share a 15-callee skeleton and are separated by exactly one callee
  each (`isInConnection` ×2 / neither / `isInCheckpoint` ×2); because those discriminators are
  themselves confirmed twins, the assignment needs no appeal to size or emission order.
- **`GetBoundPoints2D`** is the free overload (its PS2 class column is empty).

## 6. Open items

| item | state |
|---|---|
| `i3d_frame_get_bound_points_2d` call count in the three spec-node searches | ×4 on PC vs ×5 on PS2, unexplained. On PS2 `isInBound` consumes exactly the vector `GetBoundPoints2D` fills; if the PC merged one build-and-test pair inline, one call would vanish — search the three bodies for an inline point-in-quad test over the local vector right after each call |
| pass-side element layout | only `+8` (the selector, values seen: 2) is known; stride `0x10` on both builds, the other 12 bytes are unread |
| `state` enum | values 1/2/3/4 are proven; no enum symbol recovered |
| `ai_path_is_pos_in_next_connection` | calls the corridor test ×1 on PC and ×2 on PS2, unexplained |
| `_unk_00` / `_unk_10` / `_unk_20` and the gaps after `+0x40`, `+0x68`, `+0x90` | not read |
