# `ai_path` — one computed route, handed to an NPC action

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-11-17 20:15 +0100

An `ai_path` is a **route object**: the result `ai_wayfind` produces and the thing the movement
`ai_action` factories carry. It sits one layer above the raw graph — `ai_path` consumes
`C_web_path` (`structs/web_path.md`) node indices, and is consumed by `ai_action_manager`
(`structs/ai_action_manager.md`), whose movement factories take an `ai_path*` parameter.

| | |
|---|---|
| PC prefix | `ai_path_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x9c` — `operator_new(0x9c)`, **exactly one allocation site in the PC binary**, inside `ai_wayfind_build_optimal_path` |
| PC constructor | - |
| PC translation unit | `[0x00402740, 0x0040f700)`, shared with `ai_wayfind` (§9) |
| PS2 class | `ai_path` (tag `7ai_path`), 25 methods: 23 CONFIRMED twins in §3, 2 PS2-only in §10 |

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

PC offsets. The three containers are MSVC `std::vector`s — an allocator byte, then begin / end /
capacity — and their element type names come from the PS2 container symbols (§10). The 1.2
executable keeps every field from `+0x58` on at the same offset; its NORMAL container sits at
`+0x1c` and its SPEC container at `+0x2c` (`ai_path_tick` is `0x005026c0` there).

```c
typedef struct ai_path {                    // 0x9c bytes
    /* +0x00 */ uint8_t      _alloc_normal;      // [H] allocator byte of the NORMAL vector
    /* +0x04 */ ai_path_node *normal_begin;      // [H] NORMAL waypoints, stride 0x3c (§1a)
    /* +0x08 */ ai_path_node *normal_end;        // [H] one past the last NORMAL waypoint
    /* +0x0c */ ai_path_node *normal_cap;        // [H] end of the NORMAL vector's storage
    /* +0x10 */ uint8_t      _alloc_spec;        // [H] allocator byte of the SPEC vector
    /* +0x14 */ ai_path_specnode *spec_begin;    // [H] SPEC (detour) waypoints, stride 0x18 (§1b)
    /* +0x18 */ ai_path_specnode *spec_end;      // [H] one past the last SPEC waypoint
    /* +0x1c */ ai_path_specnode *spec_cap;      // [H] end of the SPEC vector's storage
    /* +0x20 */ uint8_t      _alloc_side;        // [H] allocator byte of the PASS-SIDE vector
    /* +0x24 */ ai_actor_evading *side_begin;    // [H] PASS-SIDE records, stride 0x10 (§1d)
    /* +0x28 */ ai_actor_evading *side_end;      // [H] one past the last PASS-SIDE record
    /* +0x2c */ ai_actor_evading *side_cap;      // [H] end of the PASS-SIDE vector's storage
    /* +0x30 */ blocked_con  blocked;            // [H] 0x10 bytes (§1c)
    /* +0x40 */ float        last_actor_pos[3];  // [H] the actor position at the last revalidation
    /* +0x4c */ float        blocker_pos[3];     // [H] the last detour blocker's world position; no reader (class census)
    /* +0x58 */ int32_t      normal_cursor;      // [H] index into the NORMAL vector; -1 = reset
    /* +0x5c */ int32_t      spec_cursor;        // [H] index into the SPEC vector; -1 = reset
    /* +0x60 */ int32_t      delay_until;        // [H] mission-clock deadline of the delay timer
    /* +0x64 */ int32_t      no_progress;        // [H] the blocked / no-progress accumulator
    /* +0x68 */ C_actor     *last_blocker;       // [H] the last detour blocker a search stored (§4)
    /* +0x6c */ uint32_t     spec_corner;        // [H] the blocker corner the last search chose; no reader (class census)
    /* +0x70 */ uint8_t      spec_inserted;      // [H] a search level inserted its spec waypoint
    /* +0x71 */ uint8_t      blocked_flag;       // [H] a blocker is standing in the way
    /* +0x74 */ float        move_target[3];     // [H] the point the walker heads for (§7)
    /* +0x80 */ float        arrival_center[3];  // [H] the arrival test's centre; the walker does not steer to it (§7)
    /* +0x8c */ uint32_t     state;              // [H] 0 none · 1 normal · 2 spec · 3 invalidated · 4 abandoned
    /* +0x90 */ float        tolerance_sq;       // [H] arrival radius^2 for arrival_center
    /* +0x94 */ uint32_t     unused_94;          // [U] no access; no reader (class census)
    /* +0x98 */ void        *cached_blocker;     // [H] the TYPE-1 special of the current segment
} ai_path;
```

### 1a. `ai_path_node` — the NORMAL waypoint (stride `0x3c`)

The route proper.

```c
typedef struct ai_path_node {               // 0x3c bytes
    /* +0x00 */ float     gate_a[3];           // [H] gate point A
    /* +0x0c */ float     gate_b[3];           // [H] gate point B; the A/B midpoint is type 0's arrival centre
    /* +0x18 */ float     dir[3];              // [H] the waypoint's unit direction
    /* +0x24 */ float     pos[3];              // [H] the waypoint's own position
    /* +0x30 */ float     width;               // [H] the corridor width at this end
    /* +0x34 */ uint32_t  web_node;            // [H] C_web_node index
    /* +0x38 */ uint32_t  type;                // [H] arrival centre: 0 gate midpoint · 1 / 2 the web node
} ai_path_node;
```

### 1b. `ai_path_specnode` — the SPEC (detour) waypoint (stride `0x18`)

```c
typedef struct ai_path_specnode {           // 0x18 bytes
    /* +0x00 */ float     point[3];            // [H] the detour point
    /* +0x0c */ uint32_t  unread_0c;           // [U] the three searches store 1; no reader (class census)
    /* +0x10 */ C_actor  *blocker;             // [H] the blocker the detour goes round
    /* +0x14 */ tDynamicCollObject *collider;  // [H] the blocker's collider
} ai_path_specnode;
```

### 1c. `blocked_con` — the embedded blocked-connection record (`+0x30..+0x3f`)

The type name comes from the signature `AddBlockedCon__8C_entityFP11blocked_con`.

```c
typedef struct blocked_con {                // 0x10 bytes, embedded at ai_path+0x30
    /* +0x00 */ uint32_t  node_a;              // [H] the blocked web node, or the connection's start node (§5)
    /* +0x04 */ uint32_t  node_b;              // [H] the current waypoint's web node
    /* +0x08 */ uint32_t  conn_index;          // [H] index of the node_a -> node_b connection; 0x29a = the whole node
    /* +0x0c */ uint32_t  expiry_ms;           // [H] the block's deadline; 0 = no block
} blocked_con;
```

### 1d. `ai_actor_evading` — the pass-side record (stride `0x10`)

```c
typedef struct ai_actor_evading {           // 0x10 bytes
    /* +0x00 */ uint32_t  made_ms;             // [H] mission clock when the record was made
    /* +0x04 */ uint32_t  depth;               // [H] the search depth that made it
    /* +0x08 */ uint32_t  side;                // [H] the side taken: 1 or 2
    /* +0x0c */ C_actor  *blocker;             // [H] the blocker
} ai_actor_evading;
```

## 2. Field evidence

Measured by a census of every `this+N` access in the `ai_path_*` methods, plus the external
writers and readers named in each row. "The three searches" are the
`ai_path_find_spec_node_from_*` trio; "the normal getters" are
`ai_path_advance_to_next_normal_waypoint` and `ai_path_load_normal_waypoint`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | _alloc_normal | H | - | - | the first byte of the NORMAL vector object |
| +0x04 | normal_begin | H | `std_vector3c_insert_at`, `std_vector3c_insert_n` (from `ai_wayfind_build_optimal_path`), `ai_path_free_buffers` | the normal getters, `ai_path_is_last_normal_waypoint`, `ai_path_get_destination`, the corridor tests, `ai_path_is_pos_in_next_connection`, `ai_path_is_turning_at_normal_waypoint`, `ai_path_normal_waypoint_at`, the three searches, `ai_path_update_spec_waypoints`, `ai_path_destroy` | - |
| +0x08 | normal_end | H | the two inserts, `ai_path_destroy` (= begin), the normal getters, `ai_path_free_buffers` | the same readers | the count is `(end - begin) / 0x3c` |
| +0x0c | normal_cap | H | the two inserts, `ai_path_free_buffers` | the two inserts | - |
| +0x10 | _alloc_spec | H | the three searches (by address, their inline insert), `ai_path_update_spec_waypoints` (by address, `std_vector18_erase_at`) | - | the first byte of the SPEC vector object; the byte itself is never stored: the fillers write `spec_begin` / `_end` / `_cap` through the header's address |
| +0x14 | spec_begin | H | the three searches (their insert), `ai_path_free_buffers` | the three searches, `ai_path_update_spec_waypoints`, `ai_path_advance_to_next_spec_waypoint`, `ai_path_tick`, `ai_path_destroy` | - |
| +0x18 | spec_end | H | the three searches, `ai_path_destroy` (= begin), `ai_path_advance_to_next_spec_waypoint`, `ai_path_free_buffers`; `std_vector18_erase_at` (from `ai_path_update_spec_waypoints`); `ai_action_invalidate_path_for_actor` (= begin, when a spec waypoint has a blocker) | the three searches, `ai_path_update_spec_waypoints`, `ai_path_advance_to_next_spec_waypoint` | - |
| +0x1c | spec_cap | H | the three searches, `ai_path_free_buffers` | the three searches | - |
| +0x20 | _alloc_side | H | the three searches (by address, `std_vector10_insert_n`) | - | the first byte of the PASS-SIDE vector object; the byte itself is never stored: the fillers write `side_begin` / `_end` / `_cap` through the header's address |
| +0x24 | side_begin | H | `std_vector10_insert_n` (from the three searches), `ai_path_free_buffers` | the three searches, `ai_path_force_pass_side_right`, `ai_path_destroy`, `ai_action_invalidate_path_for_actor` | - |
| +0x28 | side_end | H | `std_vector10_insert_n`, `ai_path_destroy` (= begin), `ai_path_free_buffers`, `ai_action_invalidate_path_for_actor` (its erase) | the three searches, `ai_path_force_pass_side_right` | - |
| +0x2c | side_cap | H | `std_vector10_insert_n`, `ai_path_free_buffers` | `std_vector10_insert_n` | - |
| +0x30 | blocked | H | per field, §2c | per field, §2c | an embedded `blocked_con` |
| +0x40 | last_actor_pos | H | `ai_path_revalidate_for_actor`, `ai_wayfind_build_optimal_path` (0) | `ai_path_revalidate_for_actor` | a move of more than 4.0 units² since the last call is a teleport ⇒ state 4 (§7); a zero vector means not yet set, and the teleport test is skipped |
| +0x4c | blocker_pos | H | the three searches | - | the blocker's frame `+0x40..+0x48`; no reader (class census: every `ai_path_*` and `ai_wayfind_*` body, the holders and `dog_follow_path`; the `+0x4c` accesses in `entity_action_tick` and `entity_process_command` are action records) |
| +0x58 | normal_cursor | H | `ai_path_destroy` (= -1), the normal getters, `ai_path_update_spec_waypoints` | the normal getters, `ai_path_is_last_normal_waypoint`, `ai_path_is_pos_in_next_connection`, `ai_path_is_turning_at_normal_waypoint`, the three searches, `ai_path_update_spec_waypoints` | `ai_path_destroy` writes `+0x5c` and then `+0x58` |
| +0x5c | spec_cursor | H | `ai_path_destroy` (= -1), `ai_path_update_spec_waypoints`, `ai_path_advance_to_next_spec_waypoint`, `ai_action_invalidate_path_for_actor` (= -1) | `ai_path_advance_to_next_spec_waypoint`, `ai_path_tick` | - |
| +0x60 | delay_until | H | `ai_path_arm_delay_timer` (= clock + 1000), `ai_path_update_spec_waypoints` (= clock + 200 after opening a door, and its other re-arms) | `ai_path_update_spec_waypoints` | - |
| +0x64 | no_progress | H | `ai_path_revalidate_for_actor` | `ai_path_revalidate_for_actor`, `ai_path_tick`, the three searches | +3 below speed 0.1, +1 up to 0.5, −1 above, 0 above 0.9; gates at 4/10/15/25/50/100 (§7) |
| +0x68 | last_blocker | H | the three searches (every detour blocker, of any kind, at their insertion tail; a closed door also earlier, within 3 / 1.5 units), `ai_path_update_spec_waypoints` (clears it), `ai_wayfind_build_optimal_path` (0), `ai_action_invalidate_path_for_actor` (nulls it when it is the departing actor) | the three searches (find the previous blocker's pass-side record), `ai_path_update_spec_waypoints` (opens it only when it is a door, kind 6, and the walker a kind-`0x1b` entity) | the door test is the reader's (§4) |
| +0x6c | spec_corner | H | the three searches (the index of the chosen footprint corner; the coll-list search stores it through a float local) | - | no reader (class census, as `+0x4c`; the `+0x6c` of `ai_path_is_pos_in_next_connection` is the next waypoint's `width`) |
| +0x70 | spec_inserted | H | the three searches (= 1), `ai_path_update_spec_waypoints` (clears it before each search) | the three searches | of the recursion's levels only the first to finish inserts |
| +0x71 | blocked_flag | H | `ai_path_revalidate_for_actor` | `ai_path_tick`, `ai_path_revalidate_for_actor` | set while a blocker stands in the way, cleared when `no_progress` falls back to 0 |
| +0x74 | move_target | H | the normal getters, `ai_path_advance_to_next_spec_waypoint` (the detour point) | `ai_path_tick` (both arms pass its address to `human_move_to` in EDX), `dog_follow_path` (steers the dog at `+0x74..+0x7c` in its states 1 and 2) | the movement target: the current waypoint's `pos`, or the detour point in state 2 (§7) |
| +0x80 | arrival_center | H | the normal getters | `ai_path_tick` (its arrival test only, against `tolerance_sq`) | the arrival-test centre: the gate midpoint (type 0) or the web node position (types 1/2); a detour leaves it unchanged (§7) |
| +0x8c | state | H | the three searches (= 3), the normal getters and `ai_path_advance_to_next_spec_waypoint` (1 / 2), `ai_path_revalidate_for_actor` (= 4), `ai_action_invalidate_path_for_actor` (= 3) | `ai_path_update_spec_waypoints`, `ai_path_tick`, `ai_path_revalidate_for_actor`, `ai_path_is_invalidated`, `ai_path_is_abandoned`, `ai_path_get_state`, `dog_follow_path` | the value set 0..4 is closed by the 29 writes to `+0x8c` in the binary |
| +0x90 | tolerance_sq | H | the normal getters | `ai_path_tick`, `dog_follow_path` (its arrival test against `move_target`) | the arrival radius² |
| +0x94 | unused_94 | U | - | - | no access; no reader (class census: every `ai_path_*`, `ai_wayfind_*`, `ai_action_*` and `dog_*` function, and `entity_action_tick`) |
| +0x98 | cached_blocker | H | `ai_path_find_segment_for_node`, `ai_path_advance_to_next_normal_waypoint` | `ai_path_find_segment_for_node`, `dog_follow_path` (writes the special's `+0x2c` through it) | a piece of geometry bound to a live scene frame (`structs/check_seg.md` §4c, §5) |

### 2a. `ai_path_node`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | gate_a | H | `ai_wayfind_build_path_node_from_nodes`, `ai_wayfind_build_path_node_from_positions`, `ai_wayfind_compute_node_left_right_gate` | the normal getters, `ai_path_is_pos_in_next_connection` | the node position minus the perpendicular times the width |
| +0x0c | gate_b | H | the two builders, `ai_wayfind_compute_node_left_right_gate` | the normal getters, `ai_path_is_pos_in_next_connection` | the node position plus the perpendicular times the width |
| +0x18 | dir | H | the two builders (normalised), `ai_wayfind_compute_node_left_right_gate` (the bisector at a sharp turn) | the same three, to derive the gate perpendicular | read only by its builders; no reader after the element is stored |
| +0x24 | pos | H | the two builders, `ai_wayfind_build_optimal_path` (the last element), `ai_wayfind_compute_node_left_right_gate` (0.5 off the corner at a sharp turn, else `1.2 - dot` along the perpendicular) | `ai_path_get_destination` (the last element), the normal getters | the movement target while the waypoint is current (copied to `move_target`); the route destination is the last element's `pos` |
| +0x30 | width | H | `ai_wayfind_build_optimal_path` (0.5 on the positional end node, the one with `web_node` -1) | `ai_path_is_pos_in_next_connection` | used when either end has no web node; otherwise the web node's `+0x10 & 0xffff` × `0.01` |
| +0x34 | web_node | H | `ai_wayfind_build_optimal_path` (the route's node index before each insert; -1 on the positional end node) | the normal getters (types 1/2), `ai_path_update_spec_waypoints` (`!= -1`), `ai_path_is_pos_in_next_connection`, the three searches (the current and the previous waypoint's node), `entity_action_tick` (through `ai_path_normal_waypoint_at`) | resolved against `*(g_act_mission+0x2c)+4` at stride `0x1e` |
| +0x38 | type | H | the two builders (= 0), `ai_wayfind_compute_node_left_right_gate` (1 or 2 at a turn sharper than 45 degrees, `dot <= 0.707`, by the sign of `vec2_signed_angle_diff`, when the two offset edges intersect; 0 otherwise) | the normal getters | 0 centres the arrival test on the A/B midpoint; 1 and 2 centre it on the web node and give the side the corner is on |

### 2b. `ai_path_specnode`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | point | H | the three searches | `ai_path_advance_to_next_spec_waypoint` | the detour point |
| +0x0c | unread_0c | U | the three searches (= 1) | - | copied only (`copy_elem_stride_0x18`, `std_vector18_erase_at`); no reader (class census: the class, `ai_path_tick`, `dog_follow_path`, `ai_action_invalidate_path_for_actor`) |
| +0x10 | blocker | H | the three searches | `ai_path_tick`, `ai_action_invalidate_path_for_actor` | the owner of the collider below (`collider+0x44`); `ai_path_tick` ghosts it (§4) |
| +0x14 | collider | H | the three searches | `ai_path_tick` (hands it to `ai_ghost_add_or_refresh`: a car blocker for 500 ms, a door for 800 ms) | as `ai_path_find_blocking_dyn_object` reports it |

### 2c. `blocked_con`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | node_a | H | the three searches (six sites) | `entity_add_blocked_con` | the current waypoint's `web_node`; in the connection shape the previous waypoint's (§5) |
| +0x04 | node_b | H | the three searches | `entity_add_blocked_con` | the current waypoint's `web_node`; equal to `node_a` in the node shape |
| +0x08 | conn_index | H | the three searches: `0x29a` in the node shape, the index of the `node_a` -> `node_b` connection in the connection shape | `entity_add_blocked_con`, the `ai_wayfind_plan_*` strategies | `0x29a`, the project-wide 666 marker, blocks the whole node: `entity_add_blocked_con` **gates on `param_2[2] == 0x29a`**; any other value names one connection, whose byte `+3` bit `0x40` the planners set for the search and clear after (§5) |
| +0x0c | expiry_ms | H | the three searches, `ai_wayfind_build_optimal_path` (= 0 on a fresh route) | `ai_path_has_blocked_con`, `entity_add_blocked_con` | `game_frame_time_ms` + 30000 (door arm), + 10000 (the coll-list search's blocker arm), + 10000 / + 15000 (the plane / positions searches' blocker arms); + 1000 instead when the blocker is a kind-`0x1b` entity (§5) |

### 2d. `ai_actor_evading`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | made_ms | H | the three searches | the three searches | the same-frame test of §6 |
| +0x04 | depth | H | the three searches | the three searches | - |
| +0x08 | side | H | the three searches (1, and the 1 ↔ 2 flip), `ai_path_force_pass_side_right` (= 2) | the three searches | - |
| +0x0c | blocker | H | the three searches | the three searches, `ai_action_invalidate_path_for_actor` (erases the departing blocker's records) | - |

## 3. Method family

Coverage: **23** of the 25 PS2 `7ai_path` methods are in the tables below, **23 CONFIRMED twins**,
every one body-diffed on both builds; the two PS2-only members are in §10. Seven PC functions have
no PS2 twin (the destructor pair and five inline helpers), and one `ai_path_node` member is
STRUCTURAL.

### 3a. Construction & destruction

The CRT and static-init rows belong to the shared unit; this doc hosts them (§9).

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00402740 | `crt_static_init_guard_00402740` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00402760 | `crt_atexit_register_00402760` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00402780 | `dir8_table_normalized_static_init` | `g_dir8_table_normalized` | - | - | - | fills the normalised 8-direction table (`structs/ai_check.md` §6) |
| 0x00402890 | `dir8_table_static_init` | `g_dir8_table` | - | - | - | fills the raw 8-direction table (`structs/ai_check.md` §6) |
| 0x004029a0 | `vec_up_00655198_static_init` | `g_vec_up_00655198` | - | - | - | the constant up-vector `(0, 1.0, 0)` |
| 0x0040f6c0 | `ai_path_node_construct_copy` | an `ai_path_node` element | `__ct__12ai_path_nodeFRC12ai_path_node` | 0x00358880 | STRUCTURAL | `this = ai_path_node` null-guarded copy of one `0x3c`-byte record, reached only through `std_vector3c_insert_at` from `ai_wayfind_build_optimal_path` (§10) |
| 0x004257e0 | `ai_path_scalar_deleting_dtor` | instance | - | - | - | MSVC-only: `{ ai_path_free_buffers(this); if (flags & 1) operator_delete(this); return this; }` |
| 0x00425800 | `ai_path_free_buffers` | instance | - | - | - | the `~ai_path` body: `operator_delete` on the three buffers in **reverse** order (`+0x24`, `+0x14`, `+0x04`), zeroing each triple |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004256d0 | `ai_path_get_state` | - | - | - | raw `state` (`+0x8c`) read; 30 references. Inline on PS2 (§8) |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00407920 | `ai_path_destroy` | `Destroy__7ai_pathFv` | 0x0035d8e0 | CONFIRMED | resets the path: collapses the three containers and writes `-1` to the `+0x5c` / `+0x58` pair, high dword first ¹ |
| 0x004079b0 | `ai_path_find_spec_node_from_coll_list` | `GetSpecNodeA__7ai_pathFR8S_vectorR8S_vectorR7S_planeP7C_actorRQ23gif31vector<P18g_collision_header,8>UiP7C_actor` | 0x0035d9f0 | CONFIRMED | widest detour search — clip plane **and** a pre-gathered collision list; `isInConnection` ×2; constants `9.0`×2 + `0.01`×4; ratio 1.57 |
| 0x00408e70 | `ai_path_find_spec_node_from_plane` | `GetSpecNodeB__7ai_pathFR8S_vectorR8S_vectorR7S_planeP7C_actorUiP7C_actor` | 0x0035fd00 | CONFIRMED | middle sibling — clip plane only, gathers its own collisions; neither containment call; constants `2.25`×1 + `0.75`×2; ratio 1.61 |
| 0x0040a0c0 | `ai_path_find_spec_node_from_positions` | `GetSpecNodeC__7ai_pathFR8S_vectorR8S_vectorP7C_actorUiP7C_actor` | 0x00361a90 | CONFIRMED | narrowest — positions only; `isInCheckpoint` ×2; arity `this` + 6 floats + `C_actor*` + `uint` + `C_actor*`; ratio 1.63 |
| 0x0040b390 | `ai_path_update_spec_waypoints` | `UpdateSpec__7ai_pathFR8S_vectorbP7C_actor` | 0x00363940 | CONFIRMED | rebuilds the spec container; callees: the three searches ×1 + `DynSetNextCollOwner` ×2 + `SetState` ×3; returns 0/1/2; ratio 1.18 |
| 0x0040bca0 | `ai_path_advance_to_next_normal_waypoint` | `GetNextNormalWaypoint__7ai_pathFR8S_vectorb` | 0x003643f0 | CONFIRMED | advances the normal cursor then stages; `GetSegments` ×1 + `isInNextConn` ×1; arity 3↔3; ratio 1.43 |
| 0x0040bf30 | `ai_path_load_normal_waypoint` | `GetNormalWaypoint__7ai_pathFR8S_vector` | 0x003647a0 | CONFIRMED | non-advancing sibling: tests `cursor < count`, not `count-1`, never calls `GetSegments`; ratio 1.32 |
| 0x0040c1c0 | `ai_path_advance_to_next_spec_waypoint` | `GetNextSpecWaypoint__7ai_pathFR8S_vector` | 0x00364b00 | CONFIRMED | statement for statement; frees the container and resets the cursor at the end of the list; both builds ignore the declared out-parameter; ratio 1.25 |
| 0x0040c260 | `ai_path_is_last_normal_waypoint` | `isLastNormalWaypoint__7ai_pathFv` | 0x00364bd0 | CONFIRMED | identical predicate and signed-division idiom; ratio 1.69 |
| 0x0040c2a0 | `ai_path_is_pos_in_connection_corridor` | `isInConnection__7ai_pathFRC8S_vectorRC8S_vectorRC8S_vectorff` | 0x00364c40 | CONFIRMED | `this = none` two vectors in ECX and EDX; trapezoid corridor, 4 signed-cross side tests; probe indexed `[2]` = z ⇒ the 3-D overload; shared `1e-08`; ratio 1.31 |
| 0x0040c5c0 | `ai_path_is_point2d_in_connection_corridor` | `isInConnection__7ai_pathFRC8S_vectorRC8S_vectorRC9S_vector2ff` | 0x00365060 | CONFIRMED | `this = none` every argument on the stack; same geometry, probe `[1]` = y ⇒ the `S_vector2` overload, plus endpoint discs (a capsule); `isInCheckpoint` ×2; ratio 1.28 |
| 0x0040c930 | `ai_path_is_point2d_in_checkpoint_radius` | `isInCheckpoint__7ai_pathFRC8S_vectorRC9S_vector2f` | 0x003654d0 | CONFIRMED | `this = none` every argument on the stack; one expression, same 2-D/3-D index asymmetry; ratio 1.00; "checkpoint" is backed by the consumers, not the body |
| 0x0040c970 | `ai_path_tick` | `Tick__7ai_pathFUiP7C_humanbb` | 0x00365510 | CONFIRMED | the per-frame **advance** half (§7); identical 7-callee set; arity 5↔5; ratio 1.57 |
| 0x0040cca0 | `ai_path_revalidate_for_actor` | `Update__7ai_pathFP7C_actor` | 0x00365a20 | CONFIRMED | the per-frame **revalidate** half (§7); exact 4-callee multiset; arity 2↔2; ratio 1.04 |
| 0x0040d2c0 | `ai_path_get_destination` | `GetDestination__7ai_pathFR8S_vector` | 0x00366090 | CONFIRMED | returns `pos` of the last normal element; ratio 1.30 |
| 0x0040d310 | `ai_path_arm_delay_timer` | `DelayTest__7ai_pathFv` | 0x00366100 | CONFIRMED | one statement: `+0x60 = mission_clock + 1000`; the PS2 name says "Test", the body only writes |
| 0x0040d330 | `ai_path_must_evade` | `mustEvade__7ai_pathFP7C_actorR9S_vector2R9S_vector2` | 0x00366120 | CONFIRMED | **constant TRUE on both builds**; arity 3↔3 (`ret 0xc`); called ×1 in each spec-node search |
| 0x0040d340 | `ai_path_find_segment_for_node` | `GetSegments__7ai_pathFUib` | 0x00366130 | CONFIRMED | seven-callee multiset matches 1:1; caches `+0x98`; arity 3↔3; ratio 1.35 |
| 0x0040db30 | `ai_path_do_connections_overlap` | `areOverlaped__7ai_pathFR8S_vectorR8S_vectorR8S_vectorffff` | 0x00366d30 | CONFIRMED | `this = none` two vectors in ECX and EDX; shared `0.088` + `3.1415927` + `1e-08`×2; arity 7↔7; ×1 from `ai_wayfind_build_optimal_path` |
| 0x0040ea20 | `ai_path_force_pass_side_right` | `ForceRight__7ai_pathFv` | 0x00367f80 | CONFIRMED | statement-identical loop writing `2` into `side` of every pass-side record |
| 0x0040ea50 | `ai_path_is_pos_in_next_connection` | `isInNextConn__7ai_pathFR8S_vector` | 0x00367fd0 | CONFIRMED | three confirmed callers agree at ×1 each; arity 2↔2; the same two arms on both builds (§10) |
| 0x0040ebb0 | `ai_path_find_blocking_dyn_object` | `TestDetailLines__7ai_pathFP8S_vectorP8S_vectorP8S_vectorPP18tDynamicCollObject` | 0x003682b0 | CONFIRMED | six horizontal dynamic-collision line probes; reports the blocker through a `tDynamicCollObject**`; forced 1↔1 by position, plus `collision_test_line_horizontal_dynamic` ×6; shared `0.4`×3 / `1.8` / `1.5` / `0.3` / `0.2`; arity 4↔4; ratio 1.23 |
| 0x0040ee50 | `ai_path_is_turning_at_normal_waypoint` | `isTurningNormalNode__7ai_pathFR8S_vector` | 0x003685f0 | CONFIRMED | including the `cursor < count-2` guard; true when two signed angles have a **negative** product (the route zig-zags); ratio 1.87 |
| 0x004256a0 | `ai_path_is_invalidated` | - | - | - | `state == 3`; 46 references. Inline on PS2 (§8) |
| 0x004256b0 | `ai_path_is_abandoned` | - | - | - | `state == 4`; 34 references. Inline on PS2 |
| 0x004256c0 | `ai_path_has_blocked_con` | - | - | - | `blocked.expiry_ms != 0` — this route recorded a blocked connection (§5). Inline on PS2 |
| 0x0043ac60 | `ai_path_normal_waypoint_at` | - | - | - | `*(this+4) + idx*0x3c`, the idx-th NORMAL waypoint; 2 callsites, both in `entity_action_tick`, both on the route just built (`[edi+0x1c]`), reading waypoint 0's `web_node`. Inline on PS2 |

Ratio = PS2 size / PC size.
¹ PS2 also frees here; the PC splits the free into `ai_path_free_buffers`.

### 3d. Free helpers

The helpers of the shared unit that `ai_path`'s methods call, and the two that no class of the
unit calls; the `0x3c`-stride container helpers and the vector math are called by `ai_wayfind`
and belong to its doc (`structs/ai_wayfind.md`).

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00402cd4 | `thunk_jmp_via_retaddr_plus_0x953b` | - | - | - | an indirect jump through the return address; no direct caller |
| 0x0040efe0 | `stl_destroy_n_via_fnptr` | - | - | - | calls a destructor pointer `n` times; its 8 callers are all outside this unit |
| 0x0040f300 | `std_vector18_erase_at` | - | - | - | `std::vector::erase` for `0x18`-byte elements — the spec waypoints; called by `ai_path_update_spec_waypoints` ×2 and, outside the class, by `ai_wayfind_build_optimal_path` and the road loader |
| 0x0040f6e0 | `copy_elem_stride_0x18` | - | - | - | null-guarded copy of one `0x18`-byte record, the spec-waypoint element; called ×3 by each of the three searches |

## 4. Normal and spec waypoints

- **NORMAL waypoints** are the route: full records with a gate pair, a node reference and a type.
  Produced by `ai_wayfind`, consumed by the normal getters, bounded by
  `ai_path_is_last_normal_waypoint`.
- **SPEC waypoints** are *detours* around a blocker, rebuilt on demand by
  `ai_path_update_spec_waypoints` from the three `ai_path_find_spec_node_*` searches when the direct
  line is blocked, and consumed by `ai_path_advance_to_next_spec_waypoint`. While the route follows
  a spec waypoint, `ai_path_tick` ghosts its blocker — a car, a door or an entity — so the walker's
  movement collision ignores it for a short time (`structs/ai_ghost.md` §4).
  `ai_action_invalidate_path_for_actor` invalidates the route (state 3, `spec_cursor` -1,
  `spec_end = spec_begin`) whenever any spec waypoint carries a blocker.
- **Doors on the route.** Each search stores every detour blocker, of any kind, at `+0x68`
  (`last_blocker`) at its insertion tail, and a closed door (kind 6, `door+0x70 == 1`) within 3 /
  1.5 units earlier; a locked door (`door+0x125` set) also gets the 30 s `blocked_con` of §5, and
  for a `C_dog` walker every closed door does. When `+0x68` is a door,
  `ai_path_update_spec_waypoints` calls `door_set_state(2, walker, 1, 1)` for a `C_entity`
  walker, re-arms the delay `+0x60` to now + 200 ms and clears `+0x68`. `door_set_state` refuses a
  locked door (`structs/door.md` §7c), so only an unlocked one opens.
- `ai_path_update_spec_waypoints` returns **0 / 1 / 2**, and that return value is the switch
  between the two modes — both per-frame entry points use it to decide which getter to call next.
  `state` (`+0x8c`) records the answer as 1 or 2.

The three spec-node searches test route candidates against scene objects' **footprints**:
`i3d_frame_get_bound_points_2d` returns the 4 world-space corners of an `I3D_frame`'s 2D bounding
rectangle, inflated `0.7`, at mid-height (`structs/ai_check.md`). Each search calls it four times
(§10).

## 5. The blocked connection

**How it is written.** The three `ai_path_find_spec_node_*` searches fill `blocked` at **six
sites**, in one of two shapes. The door arm writes the **node shape**:

```c
state(+0x8c) = 3;                       // INVALIDATED
+0x3c = 0;
+0x30 = +0x34 = <current web_node id>;
+0x38 = 0x29a;                          // the whole node
+0x3c = game_frame_time_ms + 30000;     // or + 1000
```

The blocker arm of `ai_path_find_spec_node_from_coll_list` writes the **connection shape**:
`node_a` = the previous waypoint's web node, `node_b` = the current one, `conn_index` = the index of
the `node_a` -> `node_b` connection, deadline + 10000. The blocker arms of the plane and positions
searches write the node shape with + 10000 / + 15000. The `+ 1000` variant replaces each deadline
when the blocker is a kind-`0x1b` entity, which is also sent event 8.

**How it is consumed.** In `entity_action_tick` case 0, when the route is invalidated and
`ai_path_has_blocked_con` reports a live record, the block is published with
`entity_add_blocked_con(path + 0x30)` before the route is torn down and replanned — so the NPC
avoids that graph edge until the deadline passes. `entity_add_blocked_con` walks the entity's
`+0x119c` list at **stride `0x10`**, exactly this record's size. The `ai_wayfind_plan_*` strategies
read `conn_index`: when it is not `0x29a` they set bit `0x40` of that connection's byte `+3` for the
search and clear it after; for a whole node the resolvers mark the node's `+0x04`.

## 6. The pass-side records

The three spec-node searches are **recursive**: each calls itself with its depth argument `+ 1`
and the blocker just found. For every blocker the route goes round they keep one
`ai_actor_evading` record. A search that meets a blocker it has a record for, in the same
frame and at a greater depth than the record's, flips the side 1 ↔ 2 — the route tries the other
side of an obstacle it keeps running into. `ai_action_invalidate_path_for_actor` erases the
records whose blocker is leaving.

## 7. The two per-frame entry points

`ai_path` is driven by **two** methods every frame:

| | `ai_path_tick` | `ai_path_revalidate_for_actor` |
|---|---|---|
| job | **advance** along the route | **revalidate** the route |
| arity | `(this, uint, C_human*, bool, bool)` | `(this, C_actor*)` |
| always does | picks the next waypoint and stages it | re-runs `ai_path_update_spec_waypoints` and re-stages |
| can set state 4 | no | **yes** — give-up or teleport |

`ai_path_tick` — move or advance. The walker always heads for `move_target` (`+0x74`): both arms
pass its address to `human_move_to` in EDX, with the walker in ECX. `arrival_center` (`+0x80`) and
`tolerance_sq` (`+0x90`) only decide arrival: outside that radius the walker keeps moving, inside
it the tick advances the cursor instead. For a normal waypoint the two points differ:
`move_target` is the element's `pos`, `arrival_center` the gate midpoint (type 0) or the web node
(types 1/2). During a detour (state 2) `ai_path_advance_to_next_spec_waypoint` replaces only
`move_target` with the detour point, so the arrival centre stays on the normal waypoint.
`ai_path_tick` returns a `char` in AL: the non-zero result of `human_move_to`, or the result of
the advance call on the other paths.

Observed in game (mission 07-02, Sarah walking home): the walker followed the line through the
waypoints' `pos` points, 0.11 units off on average and 0.33 at most over about 80 samples, while
the gate points A and B lay 2 to 5 units apart across the pavement. Also observed: a walking
NPC's `ai_path` is freed and allocated again every few seconds, and the new one often lands at
the old address (same `0x9c` size), so a cached pointer can refer to a new route.

`ai_path_revalidate_for_actor` — the guard ladder in front of its unconditional tail:

1. Bail if `state` ∈ {3, 4}.
2. Maintain `no_progress` (`+0x64`) from the actor's speed.
3. `no_progress > 4` ⇒ swept-sphere probe (r = 0.2) ahead; cache the blocker.
4. With a blocker, escalate **by blocker type**: `>10` force the pass side right · `>15` strafe
   around · type 6 ⇒ `human_use_actor` (open the door) · types 2/`0x15` `>100` ⇒ event 7 · type
   `0x1b` `>25` ⇒ event 8, `>100` ⇒ event 7 + reschedule · default `>50` ⇒ **state 4**.
5. Moved `> 4.0` units² since `last_actor_pos` ⇒ teleport ⇒ **state 4**.
6. Otherwise cache the position and fall through to the tail.

## 8. The inline state accessors

`ai_path_is_invalidated`, `ai_path_is_abandoned`, `ai_path_get_state` and
`ai_path_has_blocked_con` live in the `entity_action_tick` inline-accessor block
(`0x00424e20..0x00425850`, inside the `C_entity` unit). That band is one translation unit's inline
methods, so its members belong to many classes (`ai_path`, `ai_wayfind`, `ai_groups`, `C_game`,
`C_Vehicle`, `C_car`, `C_actor`, `C_entity`) and VA proximity says nothing about the receiver. The
receiver is `ai_path` by two independent measurements:

1. **Callsite receivers** — every callsite loads ECX as `mov ecx,[edi+0x10 / 0x14 / 0x1c / 0x20 /
   0x24 / 0x30 / 0x38 / 0x50 / 0x5c]`, one offset per `ai_action` record type, with an identical
   distribution across all three state accessors.
2. **Control flow** — in `entity_action_tick` case 0, the same pointer field is torn down three
   statements later by `ai_path_destroy()` + `ai_path_scalar_deleting_dtor(1)`.

## 9. The translation unit

`[0x00402740, 0x0040f700)`: it opens with its CRT static-init pair and three static
initializers of the direction tables and an up-vector (§3a), and the `C_entity` unit's guard,
`crt_static_init_guard_0040f700`, closes it. It holds two classes and some foreign code:

* `ai_wayfind` (`structs/ai_wayfind.md`) — the producer — with its search callbacks, path-node
  builders and planners;
* the `ai_path` methods `0x00407920`–`0x0040ee50`, interleaved with `ai_wayfind` helpers;
* `i3d_frame_get_bound_points_2d` and the two `ai_check_desc_unpack_*` functions
  (`structs/ai_check.md`), and `web_node_get_connection` (`structs/web_node.md`);
* the container and vector helpers at the tail, of which §3d lists the ones `ai_path` uses.

The previous unit, `[0x00402290, 0x00402740)`, holds `ai_check_builder`. The rest of the class sits
elsewhere: the destructor pair at `0x004257e0` and the inline accessors of §8 in the `C_entity`
unit, and `ai_path_normal_waypoint_at` at `0x0043ac60` near that unit's tail.

## 10. PC vs PS2

- **Object size: PC `0x9c`, PS2 `0xa0`.** Both allocate it in exactly one place (PC
  `operator_new(0x9c)` in `ai_wayfind_build_optimal_path`; PS2 `__nw__7ai_path(0xa0)` in
  `GetOptimalPath__10ai_wayfindFv`), which anchors the class.
- **Field rebase — band by band, never by a single offset** (the delta is not constant):

  | Field | PC | PS2 |
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
  `isTurningNormalNode` into two argument-less FPU helper calls, hence ratio 1.87. The four state
  accessors and `ai_path_normal_waypoint_at` are inline on PS2, so they have no symbol.
- **Shared magic constants:** `0.088` + `3.1415927` (`areOverlaped`), `1e-08` (the whole normalise
  ladder), `+1000` (`DelayTest`), `9.0`/`0.01`/`0.75`/`2.25`/`0.3` (the per-sibling `GetSpecNode`
  fingerprints), `4.0` (the teleport threshold), `0.5`/`0.9` (the speed gates).
- **The `GetSpecNode` siblings** share a 15-callee skeleton and are separated by exactly one callee
  each (`isInConnection` ×2 / neither / `isInCheckpoint` ×2); because those discriminators are
  themselves confirmed twins, the assignment needs no appeal to size or emission order.
- **`GetBoundPoints2D`** is the free overload (its PS2 class column is empty). Each spec-node
  search calls it five times in the PS2 source against four on the PC, because the PC emits two
  identical call blocks as one shared block (`structs/ai_check.md` §8).
- **Element type names.** The PS2 symbol table names the three containers' element types:
  `ai_path_node` (normal), `ai_path_specnode` (spec) and `ai_actor_evading` (pass-side), matched by
  their `reserve` bodies' element sizes (`0x50`, `0x20` at `0x0035fa90`, `0x10` at `0x0035fbc0`).
- **`__ct__12ai_path_nodeFRC12ai_path_node`** (72 B) is the twin of `ai_path_node_construct_copy`,
  graded STRUCTURAL: `GetOptimalPath` copies each new path node with it after `reserve`, at the
  position where `ai_wayfind_build_optimal_path` inserts through `std_vector3c_insert_at`. The PS2
  caller does the null test that the PC helper does itself.
- **`isInNextConn`** has the PC's two arms — gate positions and widths when either end has no web
  node, web-node positions and widths otherwise — and calls `isInConnection` once in each; MSVC
  routes both arms into one shared call at `0x0040eb90`.
- **No `state` enum symbol exists.** The PS2 export carries no game enum types (only ELF-loader
  ones); the value set 0..4 is closed by the 29 writes to `+0x8c`.

**The PS2-only methods.**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x003688e0 | `__nw__7ai_pathFUi` | the class-specific `operator new`: `ai_path` is pool-allocated on PS2 through the `ai_spageman` family; the PC calls the global `operator_new` |
| 0x00368940 | `__dl__7ai_pathFPv` | the matching class-specific `operator delete`; the PC calls the global `operator_delete` |

## 11. Related

- `structs/ai_wayfind.md` — the producer; same translation unit.
- `structs/web_path.md` — the graph underneath; `structs/web_node.md` — the node at `web_node`.
- `structs/ai_action_manager.md` — the consumer (`ai_path*` on every movement factory).
- `structs/check_seg.md` — the map segments and specials cached at `+0x98`.
- `structs/ai_check.md` — `i3d_frame_get_bound_points_2d` and the direction tables.
- `structs/ai_ghost.md` — the collision exemptions of §4; `structs/door.md` — the doors of §4.

## 12. Open items

1. **No reader found (class census)** for `ai_path_specnode.unread_0c`, `unused_94`, `blocker_pos`
   and `spec_corner`. The census covered every `ai_path_*`, `ai_wayfind_*`, `ai_action_*` and
   `dog_*` function, the holders of an `ai_path` and `entity_action_tick`; a `.text` scan through
   computed addresses outside them has not been run.
