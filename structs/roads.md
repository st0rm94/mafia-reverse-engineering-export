# `C_roads` — the road graph

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-11-09 09:52 +0100

The city's driving graph: a table of **road nodes** and a table of **crossings**, loaded from the
level `.dta` and queried by the traffic AI, the chase driver, the NPC pathfinder and the police
red-light test. The singleton is `*(g_act_mission + 0x30)`, a member of `C_mission`.

| | |
|---|---|
| PC prefix | `roads_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_roads_vt` @ `0x0063ee48`, 4 slots (§4b) |
| PC base class | - |
| PC instance size | `0x24` (`operator_new(0x24)` in `mission_init`) |
| PC constructor | `roads_ctor` @ `0x00617de0` |
| PC translation unit | `[0x00617db0, 0x0061aff0)`, `road_web.cpp`: the unit's CRT pair and 22 `roads_*` functions; two container helpers sit in `Mission.cpp` (§3c) and one header inline in the `C_shvestky` unit (`structs/police_manager.md`) |
| PS2 class | `C_roads` (tag `7C_roads`), 19 methods, all mapped, plus its `S_pathfind` constructor and three container methods of its two vectors (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_roads {                     // 0x24 bytes; at *(C_mission + 0x30)
    /* +0x00 */ void              *vtable;      // [H] g_roads_vt 0x0063ee48
    /* +0x04 */ uint8_t            cross_alloc; // [H] allocator byte of the crossing vector
    /* +0x08 */ S_road_crossing_1 *cross_begin; // [H] crossing table, stride 0xec
    /* +0x0c */ S_road_crossing_1 *cross_end;   // [H] one past the last crossing
    /* +0x10 */ S_road_crossing_1 *cross_cap;   // [H] end of the crossing vector's storage
    /* +0x14 */ uint8_t            node_alloc;  // [H] allocator byte of the node vector
    /* +0x18 */ S_road_node_1     *node_begin;  // [H] road-node table, stride 0x18
    /* +0x1c */ S_road_node_1     *node_end;    // [H] one past the last road node
    /* +0x20 */ S_road_node_1     *node_cap;    // [H] end of the node vector's storage
} C_roads;
```

The layout is settled by three measurements that agree: `roads_ctor` zeroes the two allocator-byte +
vector triples; the receivers of the two `Mission.cpp` container helpers are `C_roads + 0x04` and
`C_roads + 0x14`; and every table read uses `+0x08` / `+0x18`. The element type names come from the
PS2 template arguments (§5). **The PC's in-memory record is the file record**: `roads_save_data`
writes the tables out as they are.

### 1a. The tagged index

Both tables share one 16-bit id space, and bit `0x8000` selects the table:

```
id & 0x8000  set   ->  road node,  index = id & 0x7fff,  base +0x18, stride 0x18
id & 0x8000  clear ->  crossing,   index = id,           base +0x08, stride 0xec
```

`roads_get_node_or_crossing_pos` implements exactly that dispatch, and its caller
`police_manager_tick` corroborates both branches: it reads the two ids from the node record's
`+0x10` / `+0x12`, feeds them here, and separately gates on `(id & 0x8000) == 0` before indexing the
crossing table itself.

### 1b. `S_road_node_1` — one road node, `0x18` bytes

```c
typedef struct S_road_node_1 {               // 0x18 bytes
    /* +0x00 */ float    pos[3];             // [H] world position (x, y, z)
    /* +0x0c */ float    speed_limit;        // [H] m/s; 13.3 in 42 145 of the 45 472 shipped nodes
    /* +0x10 */ uint16_t end_a;              // [H] tagged id of one end of the stop-line edge
    /* +0x12 */ uint16_t end_b;              // [H] tagged id of the other end; -1 when none
    /* +0x14 */ uint16_t crossing[2];        // [H] the crossing at each end of the node's road
} S_road_node_1;
```

**Why two crossings.** A road node lies on one road between two crossings, and the record stores
both crossing indices (plain indices into the crossing table, not tagged ids).
`roads_recompile_distances` (vtable slot 3, which `roads_load_data` calls last) resets both to
`0xffff`, then walks every used slot's node chain and stamps each node with the crossing the walk
starts from (`+0x14`) and the crossing it ends at (`+0x16`). `pursuit_event_init` reads both as
`crossing_table + id * 0xec` and keeps the nearer one for each bridge
(`structs/pursuit_event.md` §1a).

### 1c. `S_road_crossing_1` — one crossing, `0xec` bytes

```c
typedef struct S_road_crossing_1 {           // 0xec bytes
    /* +0x00 */ float       pos[3];          // [H] world position (x, y, z)
    /* +0x0c */ uint8_t     signalled;       // [H] non-zero: the crossing is signal-controlled (§4a)
    /* +0x10 */ float       speed_limit;     // [H] m/s; 13.3 in 7 097 of the 7 136 shipped crossings
    /* +0x14 */ uint16_t    first_node[4];   // [H] tagged id of each road slot's first node; -1 empty
    /* +0x1c */ S_road_slot slot[4];         // [H] the four roads leaving the crossing (§1d)
} S_road_crossing_1;
```

Bytes `+0x0d..+0x0f` are padding. **Why four.** A crossing is a junction of at most four roads;
every walker of the slots (`roads_find_road_slot_to_crossing`, `roads_disconnect_unused_road_links`,
`traffic_car_lock_crossing`) iterates four, and an unused slot carries `-1` in both its
`first_node` entry and its neighbour id.

### 1d. `S_road_slot` — one road leaving a crossing, `0x34` bytes

```c
typedef struct S_road_slot {                 // 0x34 bytes; crossing + 0x1c + 0x34 * k
    /* +0x00 */ uint16_t    neighbour;       // [H] tagged id of the crossing at the far end; -1 unused
    /* +0x04 */ float       length;          // [H] the road length
    /* +0x08 */ float       gap_angle;       // [H] angle from the previous slot's road, radians; the four sum to 2π
    /* +0x0c */ uint8_t     claim_flag;      // [H] traffic claims the crossing only when set
    /* +0x0e */ uint16_t    capacity;        // [H] car capacity; 100 in 98 % of the shipped slots
    /* +0x10 */ uint16_t    car_count;       // [H] traffic cars on the road
    /* +0x14 */ S_road_lane lane[4];         // [H] the road's lanes (§1e)
} S_road_slot;
```

Bytes `+0x02..+0x03`, `+0x0d` and `+0x12..+0x13` are padding. `+0x12..+0x13` is the 2-byte gap
before the 4-aligned lane array: no load reaches it (a `.text` displacement scan finds no load at
crossing `+0x2e`, `+0x62`, `+0x96` or `+0xca`, and every word load at displacement `0x12` in the road
users is a node's `end_b`), and all 20 557 used slots of the 66 shipped `road.bin` files hold
`0xcccc` there, the filler the pad byte `+0x0d` holds too (`0xcc`).

**`gap_angle`.** `roads_recompile_distances` sorts each crossing's slots (used first, then by
direction), then stores in slot `j` the angle from slot `j-1`'s road to slot `j`'s
(`vec2_signed_angle_diff` against slot 0, + π, minus the running sum) and in slot 0 `2π` minus the
sum, so the four add up to 2π. `traffic_car_lock_crossing` compares the sum of two slots' values
with π.

### 1e. `S_road_lane` — one lane, 8 bytes

```c
typedef struct S_road_lane {                 // 8 bytes; slot + 0x14 + 8 * k
    /* +0x00 */ uint8_t turn_reservations;   // [H] cars turning through the lane (§1e)
    /* +0x01 */ uint8_t straight_reservations; // [H] cars going straight through the lane (§1e)
    /* +0x02 */ uint8_t kind;                // [H] 0 = no lane, 2 = a lane traffic may enter; 1, 3 also ship
    /* +0x04 */ float   width;               // [H] 3.0 .. 5.2 in the shipped data
} S_road_lane;
```

Byte `+0x03` is padding. **The two counters.** `traffic_car_lock_crossing` adds to both and
`traffic_car_clear_lane_blocking` subtracts from both. A car's path node points at them
(`structs/driver.md` `S_car_node`): `turn_lane_counter` (`+0x5c`, held by `S_LOCKCROSS` `blocks_b`)
at `turn_reservations`, `straight_lane_counter` (`+0x58`, `blocks_a`) at `straight_reservations`;
`driver_tick` and `driver_advance_checkpoint` wait on the first with driver `flags` `0x300` set and
on the second with them clear. **Why four lanes.** The slot has room for four lanes; the shipped kind patterns
use up to three (per road the common ones are 2-3, 2-0, 1-2, 2-2-3 and 1-2-3).
`roads_disconnect_unused_road_links` drops a slot whose four kinds are all 0;
`traffic_car_prepare_next_direction` skips a road with no kind-2 lane and caps the lane choice at
their count, and `traffic_car_lock_crossing` picks its target lanes among them. No reader tells
kinds 1 and 3 apart. The file stores the in-memory record, so the counters and counts carry
whatever the editor held when it saved: the counter pair is `(0, 0)` in 42 354 shipped lanes with a
kind, `(0xcc, 0xcc)` in 157 and other values in 9.

### 1f. `S_pathfind` — one Dijkstra entry, 8 bytes

```c
typedef struct S_pathfind {                  // 8 bytes; one per crossing, on the stack of the search
    /* +0x00 */ uint16_t prev_id;            // [H] the crossing the best path arrives from
    /* +0x04 */ float    cost;               // [H] the best cost so far; 1e+10 = unreached
} S_pathfind;
```

## 2. Field evidence

Measured by a census of every access in the `roads_*` functions, and for the record fields by body
reads of the `roads_*`, `traffic_car_*`, `pursuit_event_*` and `police_manager_tick` consumers.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `roads_ctor`, `roads_dtor` | the vtable callers (§4b) | - |
| +0x04 | cross_alloc | H | `roads_ctor` | - | the receiver of `roads_crossings_insert_n` and `roads_crossings_erase_range` |
| +0x08 | cross_begin | H | `roads_ctor`, `roads_dtor`, `roads_crossings_insert_n` | every crossing lookup of §3c, `roads_crossings_insert_n` | - |
| +0x0c | cross_end | H | `roads_ctor`, `roads_dtor`, `roads_crossings_insert_n`, `roads_crossings_erase_range` | `roads_save_data`, `roads_load_data`, `roads_merge_data`, `roads_recompile_distances`, `roads_find_path_between_crossings`, `roads_sort_nodes_by_x`, `roads_disconnect_unused_road_links`, the two helpers | - |
| +0x10 | cross_cap | H | `roads_ctor`, `roads_dtor`, `roads_crossings_insert_n` | `roads_crossings_insert_n` | - |
| +0x14 | node_alloc | H | `roads_ctor` | - | the receiver of `roads_nodes_clear`, `stl_insert_n_stride_0x18` and `std_vector18_erase_at` |
| +0x18 | node_begin | H | `roads_ctor`, `roads_dtor`, `stl_insert_n_stride_0x18` | every node lookup of §3c, `roads_nodes_clear` | ¹ |
| +0x1c | node_end | H | `roads_ctor`, `roads_dtor`, `roads_nodes_clear`, `stl_insert_n_stride_0x18`, `std_vector18_erase_at` | `roads_save_data`, `roads_load_data`, `roads_merge_data`, `roads_recompile_distances`, `roads_get_near_node`, `roads_sort_nodes_by_x`, `roads_is_sorted` | ¹ |
| +0x20 | node_cap | H | `roads_ctor`, `roads_dtor`, `stl_insert_n_stride_0x18` | - | ¹ |

¹ `roads_load_data` and `roads_merge_data` size the node vector through two shared `0x18`-stride
vector helpers called with the vector at `C_roads + 0x14`: `stl_insert_n_stride_0x18` grows it
(it writes begin, end and cap) and `std_vector18_erase_at` shrinks it (it writes end).
`roads_load_data` also empties it with `std_vector18_erase_at(begin, end)` before the read. The
crossing vector has its own instantiations, `roads_crossings_insert_n` and
`roads_crossings_erase_range`.

### 2a. `S_road_node_1`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | `roads_load_data`, `roads_merge_data` | `roads_get_near_node`, `roads_sort_nodes_by_x`, `roads_is_sorted`, `roads_get_node_or_crossing_pos` | the table is sorted by X for `roads_get_near_node`'s binary search |
| +0x0c | speed_limit | H | `roads_load_data` (rewrites a value within 1.0 of 16.666 to 13.3), `roads_merge_data` | `pursuit_event_recompile_path` | - |
| +0x10 | end_a | H | `roads_load_data`, `roads_merge_data` (rebased), `roads_sort_nodes_by_x` (repaired) | `police_manager_tick` | - |
| +0x12 | end_b | H | `roads_load_data`, `roads_merge_data` (rebased), `roads_sort_nodes_by_x` (repaired) | `police_manager_tick`, `roads_recompile_distances` | - |
| +0x14 | crossing | H | `roads_load_data`, `roads_merge_data` (file), `roads_recompile_distances` (`0xffff`, then the start and end crossing of the node's road) | `pursuit_event_init` | §1b |

### 2b. `S_road_crossing_1`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | `roads_load_data`, `roads_merge_data` | `roads_get_node_or_crossing_pos` | - |
| +0x0c | signalled | H | `roads_load_data`, `roads_merge_data` | `police_manager_tick`, `pursuit_event_recompile_path` | §4a; Open item 1 |
| +0x10 | speed_limit | H | `roads_load_data` (rewrites a value within 1.0 of 16.666 to 13.3), `roads_merge_data` | `pursuit_event_recompile_path` | - |
| +0x14 | first_node | H | `roads_load_data`, `roads_merge_data` (rebased), `roads_sort_nodes_by_x` (repaired) | `roads_recompile_distances` | - |
| +0x1c | slot | H | `roads_load_data`, `roads_merge_data`, `roads_recompile_distances` (slot order, neighbours, lengths, gap angles) | §2c | - |

### 2c. `S_road_slot`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | neighbour | H | `roads_load_data`; `roads_merge_data` (copied as read: it rebases only `first_node`, `end_a` and `end_b`); `roads_recompile_distances` (the chain walk's end crossing, `0xffff` when the chain ends nowhere) | `roads_find_road_slot_to_crossing`, `traffic_car_lock_crossing` | - |
| +0x04 | length | H | `roads_load_data`, `roads_recompile_distances` | `roads_find_road_slot_to_crossing` (the tie-break) | - |
| +0x08 | gap_angle | H | `roads_load_data`, `roads_recompile_distances` | `traffic_car_lock_crossing` | §1d; the reader compares the sum of two slots' values with π |
| +0x0c | claim_flag | H | `roads_load_data` | `traffic_car_activate_elements` (copied into the car's path node `+0x54`, read as `crossing + 0x28 + k * 0x34`), `traffic_car_ai` (claims the crossing in its `S_LOCKCROSS` list only when set), `pursuit_event_recompile_path` (copied into every path node's `+0x54` through node `+0x64`) | 1 in 9 140 of the 20 557 used shipped slots, 0 in 11 417 |
| +0x0e | capacity | H | `roads_load_data` | `traffic_car_prepare_next_direction` (ranks a road whose cars + 1 exceed it by `(cars + 1) / capacity`), `traffic_car_activate_elements` | - |
| +0x10 | car_count | H | `roads_load_data`, `traffic_car_activate_elements` (+1), `traffic_car_deactivate_element` and `traffic_car_ai` (−1) | `traffic_car_prepare_next_direction`, `traffic_car_activate_elements` | - |
| +0x14 | lane | H | `roads_load_data` | §2d | - |

### 2d. `S_road_lane`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | turn_reservations | H | `roads_load_data`, `traffic_car_lock_crossing` (+), `traffic_car_clear_lane_blocking` (−) | `traffic_car_lock_crossing`; `driver_tick` and `driver_advance_checkpoint` through `S_car_node` `turn_lane_counter` (driver `flags` `0x300` set) | §1e |
| +0x01 | straight_reservations | H | `roads_load_data`, `traffic_car_lock_crossing` (+), `traffic_car_clear_lane_blocking` (−) | `traffic_car_lock_crossing`; `driver_tick` and `driver_advance_checkpoint` through `S_car_node` `straight_lane_counter` (driver `flags` `0x300` clear) | §1e |
| +0x02 | kind | H | `roads_load_data` | `roads_disconnect_unused_road_links`, `traffic_car_prepare_next_direction`, `traffic_car_lock_crossing` | §1e |
| +0x04 | width | H | `roads_load_data` | `pursuit_event_recompile_path`, `traffic_car_activate_elements` (both place the path node's entry points across the road, `offset += width * 0.5` per lane, through node `+0x64`) | 3.0 .. 5.2 in the shipped data |

### 2e. `S_pathfind`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | prev_id | H | `roads_pathfind_entry_ctor` (`0xffff`), `roads_find_path_between_crossings` | `roads_find_path_between_crossings` | - |
| +0x04 | cost | H | `roads_pathfind_entry_ctor` (`1e+10`), `roads_find_path_between_crossings` | `roads_find_path_between_crossings` | `1e+10` is the infinity the relaxation compares against |

## 3. Method family

Coverage: all **19** PS2 `7C_roads` methods are mapped, 18 CONFIRMED and the vtable-proven
`RecompileDistances`; the `S_pathfind` constructor and the crossing vector's `insert` and `erase` are
CONFIRMED too, and the node vector's `clear` is STRUCTURAL. `roads_merge_data` is PC-only (§4b). Six PS2 container helpers have no PC rank (§5a).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00617db0 | `crt_static_init_guard_00617db0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00617dd0 | `crt_atexit_register_00617dd0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x00617de0 | `roads_ctor` | instance | `__ct__7C_roadsFv` | 0x003a70f0 | CONFIRMED | zeroes the two vector triples, installs the vtable; called by `mission_init` |
| 0x00617e10 | `roads_dtor` | instance | `__dt__7C_roadsFv` | 0x003a7120 | CONFIRMED | re-installs the vtable, frees `+0x18` then `+0x08`; called by `mission_uninit` ¹ |
| 0x0061a540 | `roads_pathfind_entry_ctor` | an `S_pathfind` | `__ct__Q27C_roads27S_pathfind$3865road_web_cppFv` | 0x003aba50 | CONFIRMED | `this = S_pathfind` byte-exact: `prev = 0xffff`, `cost = 1e+10` |

¹ The PS2 destructor frees neither table (§5).

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d1420 | `roads_get_node_or_crossing_pos` | - | - | - | the world position of one tagged id (§1a); a header inline in the `C_shvestky` unit, called twice by `police_manager_tick`; the PS2 inlines it (§5) |
| 0x00606590 | `roads_nodes_clear` | `clear__Q23std68__vector_deleter<13S_road_node_1,…>Fv` | 0x003ad440 | STRUCTURAL | `this = +0x14` the node vector's `clear`, receiver `C_roads + 0x14`, in `Mission.cpp`; one callsite, `mission_close`. MSVC builds it as `erase(begin, end)`: the move-down loop (six dwords per record) is dead (`mov eax,ebx; cmp eax,ebx; je`) because the source starts at the end (§5) |
| 0x00606700 | `roads_crossings_erase_range` | `erase__Q23std74__vector_imp<17S_road_crossing_1,…>F…` | 0x003ac5a0 | CONFIRMED | `this = +0x04` the crossing vector's erase-range, receiver `C_roads + 0x04`, in `Mission.cpp`; 4 callsites: `roads_load_data` ×2, `roads_merge_data`, `mission_close` |
| 0x00617e50 | `roads_save_data` | `SaveData__7C_roadsFP7C_chunk` | 0x003a7170 | CONFIRMED | slot 1; version 2, crossing count + table, node count + table |
| 0x00617f70 | `roads_load_data` | `LoadData__7C_roadsFP7C_chunk` | 0x003a7900 | CONFIRMED | reads the graph out of a `.dta` chunk: clear both tables, resize, read; called by `mission_open` and `mission_open_bin` |
| 0x00618320 | `roads_merge_data` | - | - | - | slot 2, PC-only (§4b): appends a second road web and rebases every tagged id in the appended records; no direct caller |
| 0x00618730 | `roads_recompile_distances` | `RecompileDistances__7C_roadsFv` | 0x003a83e0 | CONFIRMED | slot 3 (PS2 slot 2); re-stamps the node chains (each node's two crossings) and every road slot's length, orders each crossing's slots and writes their neighbours and gap angles |
| 0x00619480 | `roads_get_near_node` | `GetNearNode__7C_roadsFRC8S_vector` | 0x003aa290 | CONFIRMED | the nearest node to a world position, binary search on X; 19 callsites |
| 0x00619680 | `roads_find_crossing` | `FindCrossing__7C_roadsFUsUs` | 0x003aa490 | CONFIRMED | follows a node chain until it reaches a crossing |
| 0x006196e0 | `roads_find_road_slot_to_crossing` | `NPC_FindCrossIdxToCross__7C_roadsFUsUsUs` | 0x003aa540 | CONFIRMED | which of a crossing's four road slots leads toward a given neighbour |
| 0x00619820 | `roads_find_car_path_from_node_to_crossing` | `NPC_FindPathFromNodeToCross__7C_roadsF…` | 0x003aa6b0 | CONFIRMED | builds an `S_car_node` path out of a road node |
| 0x00619be0 | `roads_find_car_path_to_next_crossing` | `NPC_FindPathToNextCross1__7C_roadsF…` | 0x003aae40 | CONFIRMED | builds an `S_car_node` path along one road slot |
| 0x00619d20 | `roads_fill_path_from_node_to_crossing` | `FillPathFromNodeToCross__7C_roadsF…` | 0x003ab100 | CONFIRMED | appends a node-chain id list to a `vector<ushort>` |
| 0x00619de0 | `roads_find_crossing_from_node` | `FindCrossingFromNode__7C_roadsFUsUsRf` | 0x003ab220 | CONFIRMED | walks to a crossing accumulating the squared distance |
| 0x00619f20 | `roads_find_near_crossing_from_node` | `FindNearCrossingFromNode__7C_roadsFUsRib` | 0x003ab3a0 | CONFIRMED | the nearer of a road node's two crossings |
| 0x0061a070 | `roads_find_path_between_crossings` | `FindPathBeweenCross__7C_roadsF…` | 0x003ab550 | CONFIRMED | Dijkstra over the crossings (the PS2 symbol spells "Between" without the t) |
| 0x0061a550 | `roads_fill_path_from_crossing_to_crossing` | `FillPathFromCrossToCross__7C_roadsF…` | 0x003aba70 | CONFIRMED | the `NodeToCross` sibling; no pre-insert, cursor rewind before the insert |
| 0x0061a5f0 | `roads_find_path` | `FindPath__7C_roadsFUsUsR…` | 0x003abb70 | CONFIRMED | the public route query; snaps node endpoints to crossings first; 5 callsites (the chase driver and `pedestrian_wave_at_taxi`) |
| 0x0061a9a0 | `roads_sort_nodes_by_x` | `SortNodesByX__7C_roadsFv` | 0x003ac000 | CONFIRMED | slot 0; selection sort on X, then repairs every stored tagged id |
| 0x0061abb0 | `roads_is_sorted` | `IsSorted__7C_roadsFv` | 0x003ac260 | CONFIRMED | is the node table in ascending X order? Called by `roads_sort_nodes_by_x` |
| 0x0061ac10 | `roads_disconnect_unused_road_links` | `DestroyAllOffPaths__7C_roadsFv` | 0x003ac2d0 | CONFIRMED | drops a road slot whose four lane kinds are all 0; called by `roads_load_data` |
| 0x0061aca0 | `roads_crossings_insert_n` | `insert__Q23std74__vector_imp<17S_road_crossing_1,…>…` | 0x003ac710 | CONFIRMED | `this = +0x04` the crossing vector's out-of-line `insert(pos, n, value)`, receiver `C_roads + 0x04`; called by `roads_load_data` and `roads_merge_data` |

### 3d. Free helpers

The translation unit holds no free helper.

## 4. Consumers and the vtable

### 4a. Consumers

* **`police_manager_tick`**, the running-a-red-light test. It snaps the suspect's world position to
  the nearest road node (`roads_get_near_node`), reads that node's two stop-line endpoint ids, turns
  both into world positions (`roads_get_node_or_crossing_pos`), picks whichever lines up better with
  the suspect's heading, and requires the chosen id to be a crossing (`& 0x8000` clear) whose
  `signalled` byte is set before recording the offence.
* **`pursuit_event_recompile_path`**, the chase driver's path: at every path node that sits on a
  `signalled` crossing it links the node to what controls the crossing, the act state of a
  `C_bridge` on that crossing (`C_bridge + 0x138`) when one is listed, otherwise the traffic-light
  phase of the axis the path runs along (`C_game + 0x19c` or `+0x1a0`, `structs/game.md`). So the
  byte marks a crossing controlled by a light or a bridge, not only a red-light site.
* **`mission_open` / `mission_open_bin`** call `roads_load_data` while streaming the level.
* **The chase driver** (`structs/pursuit_event.md`) caches the singleton at its own `+0x20`
  and walks the node and crossing tables directly when it builds a path.
* **`game_collect_road_block_node`**, the `C_game` roadblock loader callback, calls
  `roads_get_near_node` on `*(act_mission + 0x30)` and discards the result on `-1`.

### 4b. The vtable closes the class

The cluster aligns 1:1 with the PS2 emission order with no gaps, including the `S_pathfind`
constructor, which sits at the same rank on both builds. The two vtables anchor both ends:

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 0 | 0x0061a9a0 | `roads_sort_nodes_by_x` | `SortNodesByX` (PS2 slot 0) |
| 1 | 0x00617e50 | `roads_save_data` | `SaveData` (PS2 slot 1) |
| 2 | 0x00618320 | `roads_merge_data` | - |
| 3 | 0x00618730 | `roads_recompile_distances` | `RecompileDistances` (PS2 slot 2) |

The PS2 table `__vt__7C_roads` (`0x004d9930`) is 20 bytes = a 2-word GCC header plus exactly three
pointers. Slots 0 and 1 hold the same two methods on both builds; the PC has an **inserted slot 2**,
so `roads_merge_data` is PC-only, a measurement over a closed set.

## 5. PC vs PS2

* **Element types.** The PS2 template arguments name the vectors `std::vector<S_road_crossing_1>`
  and `std::vector<S_road_node_1>`; the PS2 also carries a plain `S_road_crossing` / `S_road_node`
  pair, the file record its `SaveData` repacks into.
* **Record widths.** The PS2 keeps wider in-memory records, crossing `0xe0` and node `0x20` against
  `0xec` / `0x18` on disk, and `SaveData__7C_roadsFP7C_chunk` spends most of its 1932 bytes
  repacking, which is why that twin's size ratio is 0.15.
* **Object layout.** PS2: vtable at `+0x18`, six dwords zeroed at `+0x00..+0x14`, object `0x1c` bytes
  against the PC's `0x24` (MSVC adds one allocator byte per vector). The node table sits at PS2
  `C_roads + 0x0c` and PC `C_roads + 0x18`; do not carry a PS2 offset into a PC expression.
* **Destructor.** The PS2 `__dt__7C_roadsFv` frees neither table; `Close__9C_missionFv` frees them
  on the mission's member instead. The ctor / dtor pair was placed by rank: the PS2 calls them at the
  same positions as `mission_init` / `mission_uninit`.
* **`roads_load_data` ↔ `LoadData__7C_roadsFP7C_chunk`** is body-exact on the shared magic: the same
  two `C_chunk` flags gate the read, the same `dtaSeek(handle, 0, 1)` plus remaining-bytes `< 4`
  guard, the same 4-byte version read, the same `if (version != 2) return 0`, then the same clear of
  both tables in the same order.
* **`GetNearNode` second witness.** PS2 `cbRoadBlock` calls
  `GetNearNode__7C_roadsFRC8S_vector(act_mission + 0x30, frame + 0x70)` in exactly the statement
  position where `game_collect_road_block_node` calls `roads_get_near_node`, both discarding `-1`.
* **`S_pathfind` size.** Set at 8 bytes by the PS2's `__construct_new_array(..., 0, 8, count)`.
* **The two `Mission.cpp` container helpers.** `erase__Q23std74__vector_imp<17S_road_crossing_1,…>`
  (`0x003ac5a0`, 368 B) copies the tail of `0xec`-byte records down over the erased range, lowers the
  count and returns the first iterator; `roads_crossings_erase_range` does the same with the end
  pointer. Its one caller is the crossing vector's `resize`, which `LoadData` calls, and
  `roads_load_data` calls `roads_crossings_erase_range(begin + n * 0xec, end)` at the statement where
  that `resize` shrinks the table (MSVC inlines the `resize`), so the pair is CONFIRMED.
  `clear__Q23std68__vector_deleter<13S_road_node_1,…>` (`0x003ad440`, 80 B) walks the `0x18`-byte
  records with an empty destroy loop and zeroes the count, the outcome of `roads_nodes_clear`. Its one
  caller is the node vector's destructor, which `LoadData` runs on its temporary vector, a call
  position the PC does not have, so that pair is STRUCTURAL. `Close__9C_missionFv` frees both tables
  with `__dla` and calls neither helper.
* **The four `Max<int>` sites** in each chunk loader are a PC inlining: the PS2 calls
  `Max<i>__FRCiRCi(0x4dc2bc, &remaining)` where the PC has a compare-and-select, `max(0, remaining)`.
* **`roads_get_node_or_crossing_pos` has no PS2 function.** `TestOverRed__10C_shvestkyFv`, whose PC
  counterpart is inlined into `police_manager_tick`, spells the tagged-id dispatch inline at both
  positions where the PC calls this function, with the PS2 strides `0xe0` (crossing) and `0x20`
  (node).
* **The `road_web.cpp` unit is named by a PS2 file-static tag**,
  `Q27C_roads27S_pathfind$3865road_web_cpp`, and confirmed by the `__sinit_road_web.cpp` entry in the
  PS2 source-file manifest. The decomp spells the `$` as `_`, so a raw mangled name pasted into a
  body lookup misses.

### 5a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x003a8380 | `data__Q23std68__vector_deleter<13S_road_node_1,…>Fv` | an 8-byte begin-pointer return of the Metrowerks vector layer; its one caller is `LoadData__7C_roadsFP7C_chunk`, and `roads_load_data` reads the begin pointer inline at that statement |
| 0x003a8390 | `__ct__11S_road_nodeFv` | an 8-byte element constructor with no caller even on the PS2 (no `jal` reaches it) |
| 0x003a83a0 | `data__Q23std76__vector_deleter<17S_road_crossing_1,…>Fv` | the crossing sibling of the first row |
| 0x003a83b0 | `__ct__15S_road_crossingFv` | an 8-byte element constructor with no caller on the PS2 |
| 0x003a83c0 | `__ct__Q310Metrowerks7details60compressed_pair_imp<…S_road_node_1…>FUi` | a 12-byte Metrowerks allocator-pair constructor, called once from `LoadData`; MSVC's vector has no such layer |
| 0x003a83d0 | `__ct__Q310Metrowerks7details64compressed_pair_imp<…S_road_crossing_1…>FUi` | the crossing sibling of the row above |

## 6. Related

- `police-system/artifacts/offences.md` — the red-light offence.
- `structs/game.md` — the traffic-light clock (`C_game + 0x194` .. `+0x1a0`).
- `structs/police_manager.md` — `police_manager_tick`, the red-light consumer.
- `structs/pursuit_event.md` — the chase driver that walks the tables.
- `structs/traffic_car.md` — the traffic consumers of the slots and lanes.
- `structs/chunk.md` — `C_chunk`, what `roads_load_data` reads from.
- `structs/mission.md` — `C_mission + 0x30`, the owner.

## 7. Open items

1. **In game, un-policed crossings:** running a red light should only be recorded at a crossing whose
   `signalled` byte is set, so a crossing without lights should be un-policed; which ones is
   untested.
