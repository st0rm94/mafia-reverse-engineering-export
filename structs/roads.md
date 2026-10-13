# `C_roads` — the road graph

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-10-07 21:57 +0200

PC prefix **`roads_`**. The singleton is `*(g_act_mission + 0x30)`.

The city's driving graph: a table of **road nodes** and a table of **crossings**, loaded
from the level `.dta` and queried by the traffic AI, the NPC pathfinder and the police
red-light test. The class is closed (§2, §2b).

---

## 1. What is measured

The receiver is byte-proven by the `roads_get_near_node` body diff, which identifies
`*(g_act_mission + 0x30)` as the `C_roads` singleton. `roads_get_node_or_crossing_pos` loads it the
same way (`mov ecx,[reg+0x30]`).

```c
typedef struct C_roads {              /* 0x24 bytes */
/* +0x00 */ void       *vtable;       // g_roads_vt 0x0063ee48
/* +0x04 */ byte        cross_alloc;  // allocator byte
/* +0x08 */ S_crossing *cross_begin;  // [H] crossing table, STRIDE 0xec
/* +0x0c */ S_crossing *cross_end;
/* +0x10 */ S_crossing *cross_cap;
/* +0x14 */ byte        node_alloc;   // allocator byte
/* +0x18 */ S_road_node *node_begin;  // [H] road-node table, STRIDE 0x18
/* +0x1c */ S_road_node *node_end;
/* +0x20 */ S_road_node *node_cap;
} C_roads;
```

The layout is fixed by three independent measurements that agree: `roads_ctor` zeroes the two
allocator-byte + vector triples at `+0x04`/`+0x08..+0x10` and `+0x14`/`+0x18..+0x20`; the receivers
of the two container helpers (§2a) are `C_roads + 0x04` and `C_roads + 0x14`; and every table read
uses `+0x08` / `+0x18`.

The element types are `S_road_crossing_1` (crossing vector) and `S_road_node_1` (node vector),
from the PS2 template arguments (§4). **The PC's in-memory record is the file record**: the PC
writes its tables straight out.

| record | stride | fields decoded |
|---|---|---|
| road node (`+0x18`) | `0x18` | `+0x00..+0x0b` = the world position (3 floats); `+0x0c` = the speed limit in m/s (13.3 in 42 145 of the 45 472 shipped nodes); `+0x10` and `+0x12` = two 16-bit **tagged ids** naming the endpoints of this node's stop-line edge, `-1` when it has none |
| crossing (`+0x08`) | `0xec` | `+0x00..+0x0b` = the world position (3 floats); `+0x0c` = a non-zero byte flag `police_manager_tick` requires before it will treat the crossing as a red-light site; `+0x10` = the speed limit in m/s (13.3 in 7 097 of the 7 136 shipped crossings); `+0x14` = four 16-bit tagged ids, the first road node of each road slot, `-1` for an empty slot; `+0x1c` = four road slots ¹ |

¹ Each road slot is `0x34` bytes:

| slot offset | field | consumers |
|---|---|---|
| `+0x00` | u16 tagged id of the neighbour crossing, `-1` for an unused slot | `roads_find_road_slot_to_crossing`, `traffic_car_lock_crossing` |
| `+0x04` | f32 road length | written by `roads_recompile_distances` |
| `+0x08` | f32 the road's angle, radians | `traffic_car_lock_crossing` compares the sum of two slots' angles with pi |
| `+0x0c` | u8 crossing-claim flag | `traffic_car_activate_elements` copies it into the car's path node `+0x54`; `traffic_car_ai` claims the crossing (its `S_LOCKCROSS` list) only when it is set |
| `+0x0e` | u16 car capacity, 100 in 98 % of the shipped slots | `traffic_car_prepare_next_direction` ranks a road whose cars + 1 exceed it by `(cars + 1) / capacity`; `traffic_car_activate_elements` compares the live count with it |
| `+0x10` | u16 the number of traffic cars on the road | `traffic_car_activate_elements` adds 1; `traffic_car_deactivate_element` and `traffic_car_ai` subtract 1 |
| `+0x14` | four lanes of 8 bytes: u8, u8, u8 kind, a pad byte, f32 width (3.0 .. 5.2) | the two leading u8 are per-lane reservation counters: `traffic_car_lock_crossing` adds, `traffic_car_clear_lane_blocking` subtracts |

Lane kind 0 is no lane: `roads_disconnect_unused_road_links` drops a slot whose four kinds are
all 0. Kind 2 is a lane traffic may drive into: `traffic_car_prepare_next_direction` skips a road
with none and caps the lane choice at their count, and `traffic_car_lock_crossing` picks its
target lanes among them. Kinds 1 and 3 also ship (per road the common patterns are 2-3, 2-0,
1-2, 2-2-3 and 1-2-3), and no reader tells them apart from each other. The file stores the
in-memory record, so the counters and counts carry whatever the editor held when it saved.

**`S_pathfind` is `{u16 prev_id; float cost}`** and nothing else — 8 bytes. `roads_pathfind_entry_ctor`
initialises `prev = 0xffff`, `cost = 1e+10`; the `1e+10` is the infinity substitute the Dijkstra
relaxation compares against.

### 1a. The tagged index

Both tables share one 16-bit id space, and bit `0x8000` selects the table:

```
id & 0x8000  set   ->  road node,  index = id & 0x7fff,  base +0x18, stride 0x18
id & 0x8000  clear ->  crossing,   index = id & 0xffff,  base +0x08, stride 0xec
```

`roads_get_node_or_crossing_pos` implements exactly that dispatch, and its caller
`police_manager_tick` corroborates **both** branches independently — it reads the two ids
from the node record's `+0x10`/`+0x12`, feeds them here, and separately gates on
`(id & 0x8000) == 0` before indexing the crossing table itself.

---

## 2. PC coverage — 21 of 21

The PC cluster is `[0x00617db0, 0x0061aff0)`, one contiguous run inside the `road_web.cpp`
translation unit: the rows below from `roads_ctor` to `roads_crossings_insert_n`, preceded by the
two MSVC static-init stubs `crt_static_init_guard_00617db0` and `crt_atexit_register_00617dd0`.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d1420 | `roads_get_node_or_crossing_pos` | - | - | CONFIRMED (PC-side) | world position of one tagged id; the PS2 inlines it (§4) |
| 0x00606590 | `roads_nodes_clear` | - | - | CONFIRMED (PC-side) | the node vector's clear; hosted by `Mission.cpp` (§2a) |
| 0x00606700 | `roads_crossings_erase_range` | - | - | CONFIRMED (PC-side) | the crossing vector's erase-range; hosted by `Mission.cpp` (§2a) |
| 0x00617de0 | `roads_ctor` | `__ct__7C_roadsFv` | 0x003a70f0 | CONFIRMED | zero the two vector triples, install the vtable |
| 0x00617e10 | `roads_dtor` | `__dt__7C_roadsFv` | 0x003a7120 | CONFIRMED | re-install the vtable, free `+0x18` then `+0x08` ¹ |
| 0x00617e50 | `roads_save_data` | `SaveData__7C_roadsFP7C_chunk` | 0x003a7170 | CONFIRMED | vtable slot 1; version 2, crossing count + table, node count + table |
| 0x00617f70 | `roads_load_data` | `LoadData__7C_roadsFP7C_chunk` | 0x003a7900 | CONFIRMED | read the graph out of a `.dta` chunk: clear both tables, resize, read |
| 0x00618320 | `roads_merge_data` | - | - | STRUCTURAL | PC-only, vtable slot 2 ² |
| 0x00618730 | `roads_recompile_distances` | `RecompileDistances__7C_roadsFv` | 0x003a83e0 | CONFIRMED | vtable slot 3; re-stamp every node's owning crossing and every road slot's length |
| 0x00619480 | `roads_get_near_node` | `GetNearNode__7C_roadsFRC8S_vector` | 0x003aa290 | CONFIRMED | nearest node to a world position; binary search on X |
| 0x00619680 | `roads_find_crossing` | `FindCrossing__7C_roadsFUsUs` | 0x003aa490 | CONFIRMED | follow a node chain until it reaches a crossing |
| 0x006196e0 | `roads_find_road_slot_to_crossing` | `NPC_FindCrossIdxToCross__7C_roadsFUsUsUs` | 0x003aa540 | CONFIRMED | which of a crossing's four road slots leads toward a given neighbour |
| 0x00619820 | `roads_find_car_path_from_node_to_crossing` | `NPC_FindPathFromNodeToCross__7C_roadsF…` | 0x003aa6b0 | CONFIRMED | build an `S_car_node` path out of a road node |
| 0x00619be0 | `roads_find_car_path_to_next_crossing` | `NPC_FindPathToNextCross1__7C_roadsF…` | 0x003aae40 | CONFIRMED | build an `S_car_node` path along one road slot |
| 0x00619d20 | `roads_fill_path_from_node_to_crossing` | `FillPathFromNodeToCross__7C_roadsF…` | 0x003ab100 | CONFIRMED | append a node-chain id list to a `vector<ushort>` |
| 0x00619de0 | `roads_find_crossing_from_node` | `FindCrossingFromNode__7C_roadsFUsUsRf` | 0x003ab220 | CONFIRMED | walk to a crossing accumulating the squared distance |
| 0x00619f20 | `roads_find_near_crossing_from_node` | `FindNearCrossingFromNode__7C_roadsFUsRib` | 0x003ab3a0 | CONFIRMED | the nearer of a road node's two crossings |
| 0x0061a070 | `roads_find_path_between_crossings` | `FindPathBeweenCross__7C_roadsF…` | 0x003ab550 | CONFIRMED | Dijkstra over the crossings (sic: the PS2 symbol misspells "Between") |
| 0x0061a540 | `roads_pathfind_entry_ctor` | `__ct__Q27C_roads27S_pathfind$3865road_web_cppFv` | 0x003aba50 | CONFIRMED | byte-exact; `{u16 prev = 0xffff; float cost = 1e+10}` |
| 0x0061a550 | `roads_fill_path_from_crossing_to_crossing` | `FillPathFromCrossToCross__7C_roadsF…` | 0x003aba70 | CONFIRMED | the `NodeToCross` sibling; no pre-insert, cursor rewind before the insert |
| 0x0061a5f0 | `roads_find_path` | `FindPath__7C_roadsFUsUsR…` | 0x003abb70 | CONFIRMED | the public route query; snaps node endpoints to crossings first |
| 0x0061a9a0 | `roads_sort_nodes_by_x` | `SortNodesByX__7C_roadsFv` | 0x003ac000 | CONFIRMED | vtable slot 0; selection sort on X, then repair every stored tagged id |
| 0x0061abb0 | `roads_is_sorted` | `IsSorted__7C_roadsFv` | 0x003ac260 | CONFIRMED | is the node table in ascending X order? |
| 0x0061ac10 | `roads_disconnect_unused_road_links` | `DestroyAllOffPaths__7C_roadsFv` | 0x003ac2d0 | CONFIRMED | drop a crossing road slot whose four lane bytes are all zero |
| 0x0061aca0 | `roads_crossings_insert_n` | `insert__Q23std74__vector_imp<17S_road_crossing_1,…>…` | 0x003ac710 | CONFIRMED | the crossing vector's out-of-line `insert(pos, n, value)`; receiver `C_roads + 0x04` |

¹ The PS2 body differs (§4). ² APPENDs a second road web and rebases every tagged id in the
appended records; the PS2 vtable has no such slot (§2b).

### 2a. The two `Mission.cpp` container helpers

Both are compiler-emitted `std::vector` members in the `Mission.cpp` helper tail. A receiver
census gives each a single owner, and the offsets agree with §1 exactly.

| PC VA | PC name | receiver | which table |
|---|---|---|---|
| 0x00606700 | `roads_crossings_erase_range` | `C_roads + 0x04` | the `+0x08` stride-`0xec` **crossing** table; 4 callsites — `roads_load_data` ×2, `mission_close`, one more inside the roads cluster |
| 0x00606590 | `roads_nodes_clear` | `C_roads + 0x14` | the `+0x18` stride-`0x18` **node** table; 1 callsite, `mission_close`. Its element-destroy loop is provably dead (`mov eax,ebx; cmp eax,ebx; je`) because the element is POD |

**Construction.** `mission_init` does `operator_new(0x24)` then `roads_ctor` into
`C_mission + 0x30`; `mission_uninit` calls `roads_dtor` on the same member.

### 2b. The vtable closes the class

The cluster aligns 1:1 with the PS2 emission order with no gaps — **including the `S_pathfind`
constructor**, which sits at the same rank on both builds. Two independent anchors confirm the
alignment:

| | slot 0 | slot 1 | slot 2 | slot 3 |
|---|---|---|---|---|
| **PC** `g_roads_vt` `0x0063ee48` | `roads_sort_nodes_by_x` | `roads_save_data` | `roads_merge_data` | `roads_recompile_distances` |
| **PS2** `__vt__7C_roads` `0x004d9930` | `SortNodesByX` | `SaveData` | `RecompileDistances` | - |

The PS2 table is 20 bytes = a 2-word GCC header plus exactly three pointers. Slots 0 and 1 hold the
same two methods on both builds; the PC has an **inserted slot 2**, so every slot below it shifts
by one. That settles two things a body-read could not: `roads_recompile_distances` is PC slot 3
against PS2 slot 2, and `roads_merge_data` is **PC-only** — a measurement over a closed set, not a
gap.

---

## 3. Consumers

* **`police_manager_tick`** — the running-a-red-light test. It snaps the suspect's world
  position to the nearest road node (`roads_get_near_node`), reads that node's two
  stop-line endpoint ids, turns both into world positions
  (`roads_get_node_or_crossing_pos`), picks whichever lines up better with the suspect's
  heading, and requires the chosen id to be a crossing (`& 0x8000` clear) whose `+0x0c`
  flag is set before recording the offence.
* **`mission_open` / `mission_open_bin`** — call `roads_load_data` while streaming the level.
* **`game_collect_road_block_node`** — the `C_game` roadblock loader callback; calls
  `roads_get_near_node` on `*(act_mission+0x30)` and discards the result on `-1`.

---

## 4. PC vs PS2

* **Element types.** The PS2 template arguments name the vectors `std::vector<S_road_crossing_1>`
  and `std::vector<S_road_node_1>`; the PS2 also carries a plain `S_road_crossing` / `S_road_node`
  pair, the FILE record its `SaveData` repacks into.
* **Record widths.** The PS2 keeps wider in-memory records — crossing `0xe0` and node `0x20`
  against `0xec` / `0x18` on disk — and `SaveData__7C_roadsFP7C_chunk` spends most of its 1932
  bytes repacking, which is why that twin's size ratio is 0.15.
* **Object layout.** PS2: vtable at `+0x18`, six dwords zeroed at `+0x00..+0x14`, object `0x1c`
  bytes against the PC's `0x24` (MSVC adds one allocator byte per vector). The node table sits at
  PS2 `C_roads+0x0c` and PC `C_roads+0x18`; do not carry a PS2 offset into a PC expression.
* **Destructor.** The PS2 `__dt__7C_roadsFv` is a stub that frees neither table —
  `Close__9C_missionFv` frees them on the mission's member instead. A factoring difference, not a
  mismatch. The ctor/dtor pair was placed by rank: the PS2 calls `__ct__7C_roadsFv` /
  `__dt__7C_roadsFv` at the same positions as `mission_init` / `mission_uninit`.
* **`roads_load_data` ↔ `LoadData__7C_roadsFP7C_chunk`** is body-exact on the shared magic:
  the same two `C_chunk` flags gate the read, the same `dtaSeek(handle, 0, 1)` plus
  remaining-bytes `< 4` guard, the same 4-byte version read, the same
  `if (version != 2) return 0`, then the same clear of **both** tables in the same order.
  Identical signature `(this, C_chunk*)`.
* **`GetNearNode` second witness.** PS2 `cbRoadBlock` calls
  `GetNearNode__7C_roadsFRC8S_vector(act_mission+0x30, frame+0x70)` in exactly the statement
  position where `game_collect_road_block_node` calls `roads_get_near_node`, both discarding `-1` —
  an independent witness for the mapping from a different translation unit.
* **`S_pathfind` size.** Fixed at 8 bytes by the PS2's `__construct_new_array(..., 0, 8, count)`.
* **Method mapping.** 21 of 21 PS2 methods are mapped, on a forced gapless order-align whose two
  ends are independently confirmed by the vtables of both builds (§2b).
* **The four `Max<int>` sites** in each chunk loader are a PC inlining: the PS2 calls
  `Max<i>__FRCiRCi(0x4dc2bc, &remaining)` where the PC has a compare-and-select that is
  plain `max(0, remaining)`. Bannered in the decomp at the first site of
  `roads_load_data` and of `roads_merge_data`.
* **Six tiny PS2 container helpers have no PC rank**: `data__vector_deleter<S_road_node_1>` and
  its crossing sibling, `__ct__11S_road_nodeFv`, `__ct__15S_road_crossingFv` and the two
  `compressed_pair_imp` constructors, all 8–12 byte identity or offset returns. The two element
  constructors have no caller even on the PS2 (no `jal` reaches either address). The other four
  are each called once, all from `LoadData__7C_roadsFP7C_chunk`: they are the Metrowerks vector
  layer, which MSVC's vector does not have, and `roads_load_data` reads the table's begin
  pointer inline at the matching statement.
* **`roads_get_node_or_crossing_pos` has no PS2 function.** `TestOverRed__10C_shvestkyFv`, whose
  PC counterpart is inlined into `police_manager_tick`, spells the tagged-id dispatch inline at
  both positions where the PC calls this function, with the PS2 strides `0xe0` (crossing) and
  `0x20` (node).
* **The `road_web.cpp` translation unit is named by a PS2 file-static tag**,
  `Q27C_roads27S_pathfind$3865road_web_cpp`, and confirmed by the `__sinit_road_web.cpp`
  entry in the PS2 source-file manifest. The decomp spells the `$` as `_`, so a raw mangled name
  pasted into a body lookup misses.

---

## 5. Related

`police-system/artifacts/offences.md` (the red-light offence) · `structs/police_manager.md`
(`police_manager_tick`, the only gameplay consumer) · `structs/chunk.md` (`C_chunk`, what
`roads_load_data` reads from) · `structs/mission.md` (`C_mission+0x30`).

---

## 6. Open items

| item | state |
|---|---|
| in game: un-policed crossings | running a red light should only be recorded at a crossing whose `+0x0c` flag is set, so some intersections should be un-policed; which ones is untested |
