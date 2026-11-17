# `ai_check_builder` / `ai_check_desc` — the 8-direction space test and the descriptor unpackers

**Created** 2026-07-30 13:41 +0200 · **Modified** 2026-11-15 23:54 +0100

Two small classes that serve the navigation searches, in the translation units immediately below the `ai_wayfind` /
`ai_path` cluster. **`ai_check_builder`** asks how much open space there is around a point by probing 8 compass directions with
collision rays. **`ai_check_desc`** unpacks a packed descriptor dword of a navigation-web node into a small stack record, one
method for integers and one for scaled floats.

```
  ai_wayfind  (strategy 6 = SEARCH, and the checkpoint scorer)   structs/ai_wayfind.md
        |                                    |
        v                                    v
  ai_check_desc  (unpack the descriptor)   ai_space_coverage  (structs/ai_space_coverage.md)
                                             |
                                             v
                                      ai_check_builder  (probe 8 directions)
                                             |
                                             v
                          g_dir8_table_normalized / g_dir8_table   (§6)
```

| | |
|---|---|
| PC prefix | `ai_check_builder_`, `ai_check_desc_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `ai_check_builder` no field, the member at `ai_manager+0x00`; `ai_check_desc` `0x20`, a stack local |
| PC constructor | - |
| PC translation unit | `[0x00402290, 0x00402740)` for `ai_check_builder`, guard to guard, 8 starts; the two `ai_check_desc` methods sit in the next unit, hosted by `structs/ai_path.md` §9 |
| PS2 class | `ai_check_builder` (tag `16ai_check_builder`), 3 methods; `ai_check_desc` (tag `13ai_check_desc`), 2 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `ai_check_builder`

No field. Its object is the empty member at `ai_manager+0x00` (`0x00655218`, `structs/ai_manager.md`): the six outside callsites
load `mov ecx, 0x655218`, and no method reads `this` except to pass it on to `vec3_rotate_by_angle_in_place`.

### 1b. `ai_check_desc`

A `0x20`-byte record on the caller's stack: `lea ecx, [esp + 0x30]` before `ai_check_desc_unpack_data` in
`ai_wayfind_score_and_add_checkpoint`, `lea ecx, [esp + 0x34]` before `ai_check_desc_unpack_ranges` in
`ai_wayfind_plan_search_route`. Each caller fills only its own half. The two halves hold the two descriptor dwords of a
`C_web_node` (§5a).

```c
typedef struct ai_check_desc {       // 0x20 bytes, a stack local of each caller
    /* +0x00 */ uint32_t unread_00;    // [U] bits [31:24] of the data dword, node byte +0x17; no reader (class census, §2)
    /* +0x04 */ uint32_t weight_code_a; // [H] bits [23:20]: the first checkpoint weighting code
    /* +0x08 */ uint32_t weight_code_b; // [H] bits [19:16]: the second checkpoint weighting code
    /* +0x0c */ uint32_t unread_0c;    // [U] bits [15:8] of the data dword, node byte +0x15; no reader (class census, §2)
    /* +0x10 */ uint32_t unread_10;    // [U] bits [7:0] of the data dword, node byte +0x14; no reader (class census, §2)
    /* +0x14 */ float    search_range; // [H] bits [31:24] of the range dword * 0.25: the strategy-6 search range factor
    /* +0x18 */ float    range_b;      // [H] bits [23:16] * 0.25: node byte +0x12; no reader
    /* +0x1c */ float    radius;       // [H] bits [15:0] * 0.01: the node's radius_cm in world units; no reader through this record
} ai_check_desc;
```

## 2. Field evidence

`ai_check_builder` has no field. The readers below were located by disassembly at each of the two callsites. The record
exists only in those two stack frames, so the two callers are the closed set of its readers: after the
`ai_check_desc_unpack_data` call its one caller loads `+0x04` and `+0x08` only.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | unread_00 | U | `ai_check_desc_unpack_data` | - | node byte `+0x17` (`unread_17` in `structs/web_node.md`); unpacked and not read |
| +0x4 | weight_code_a | H | `ai_check_desc_unpack_data` | `ai_wayfind_score_and_add_checkpoint` | `[esp + 0x30]` after the call |
| +0x8 | weight_code_b | H | `ai_check_desc_unpack_data` | `ai_wayfind_score_and_add_checkpoint` | - |
| +0xc | unread_0c | U | `ai_check_desc_unpack_data` | - | node byte `+0x15` (`unread_15` in `structs/web_node.md`); unpacked and not read |
| +0x10 | unread_10 | U | `ai_check_desc_unpack_data` | - | node byte `+0x14` (`unread_14` in `structs/web_node.md`); unpacked and not read |
| +0x14 | search_range | H | `ai_check_desc_unpack_ranges` | `ai_wayfind_plan_search_route` | `[esp + 0x44]` after the call |
| +0x18 | range_b | H | `ai_check_desc_unpack_ranges` | - | unpacked and not read |
| +0x1c | radius | H | `ai_check_desc_unpack_ranges` | - | the endpoint resolvers read the node's `radius_cm` directly (`structs/web_node.md` §2b) |

## 3. Method family

5 of 5 PS2 methods mapped; no PS2-only method.

### 3a. Construction & destruction

#### the translation unit

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00402290 | `crt_static_init_guard_00402290` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004022b0 | `crt_atexit_register_004022b0` | the translation unit | - | - | - | registers the shared no-op static destructor |

### 3b. Accessors

No row.

### 3c. Behaviour

#### `ai_check_builder`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004022c0 | `ai_check_builder_test_space_8dir` | `TestSpace8RExt__16ai_check_builderFR8S_vectorfPf` | 0x00356a80 | CONFIRMED | The 8-direction space test (§4a); called by `ai_space_coverage_compute` |
| 0x004025c0 | `vec3_rotate_by_angle_in_place` | - | - | - | Rotates a vector in place by an angle in radians; called by the space test and 3× by `entity_action_tick`. It keeps a name that says what it is (§4c) |
| 0x00402620 | `ai_check_builder_get_dir8_index` | `GetOrientation__16ai_check_builderFR8S_vector` | 0x00357000 | CONFIRMED | Quantises a direction to one of the 8 slots (§4b); called by `ai_space_coverage_tick`, `ai_space_coverage_prefer_dir` |

#### `ai_check_desc`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00403530 | `ai_check_desc_unpack_data` | `UNPACKDATA__13ai_check_descFUi` | 0x00357e30 | CONFIRMED | Unpacks a data dword into `+0x00..+0x10`; one caller, `ai_wayfind_score_and_add_checkpoint` |
| 0x00403570 | `ai_check_desc_unpack_ranges` | `UNPACKRANGES__13ai_check_descFUi` | 0x00357e80 | CONFIRMED | Unpacks a range dword into `+0x14..+0x1c`; one caller, `ai_wayfind_plan_search_route` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004026c0 | `vec3_set_xyz` | - | - | - | vector helper of the unit, emitted here and used across the binary |
| 0x004026e0 | `vec3_sub` | - | - | - | vector helper of the unit, emitted here and used across the binary |
| 0x00402710 | `vec3_make_scaled` | - | - | - | vector helper of the unit; one caller, `circuit_load_gates_from_scene` |

## 4. The space test

### 4a. `ai_check_builder_test_space_8dir`

`ai_check_builder_test_space_8dir(pos, radius, out_threshold)` returns a bitmask:

1. for each of the **8 unit vectors** in `g_dir8_table_normalized` (45° apart, y = 0):
2. scale it by `radius` and test a line from `pos` along it through `collision_test_line_vertical_thunk`, taking the distance to the
   hit, or `radius` itself when the probe is clear, so "no obstacle" and "obstacle exactly at the radius" give the same value;
3. rotate the ray by **`0.25`** and probe again; rotate by **`-0.5`** and probe a third time. `vec3_rotate_by_angle_in_place` feeds its
   argument to `fsin` / `fcos` (radians), so the three probes lie at the direction, **+0.25 rad** and **-0.25 rad**, a ±14.3° fan;
4. store `(probe1 + probe2 + probe3) * 0.3334`, the mean of the three, as that direction's clearance in a local array of 8.

So the scan is **24 collision queries** and yields 8 clearances. The tail:

```
threshold = sum(clearance) * 0.125 - 0.25
bit i is set when threshold <= clearance[i]
if out_threshold != NULL: *out_threshold = threshold
```

* The third parameter is one optional float, not an array.
* Bit *i* means *direction i has above-average clearance*. The consumer is `ai_space_coverage_compute`, which stores the mask at
  `coverage+0x50` and the threshold at `+0x54`; `ai_space_coverage_tick` tests the mask bit by bit.

The loop's cursor starts at `g_dir8_table_normalized[0].z` and reads each entry as `[-2]`, `[-1]`, `[0]`.

### 4b. The direction-to-index helper

`ai_check_builder_get_dir8_index(dir)` normalises the `(x, z)` pair (guarded by `1e-08`), measures the signed angle against the table's
entry 0, the reference direction `(0, 0, 1)`, and rounds to the nearest 45°. It returns a table index. The decomp does not show the
rounding: it runs on the x87 stack and only the trailing `__ftol` renders (the formula is in §8).

### 4c. The third method keeps its own name

`vec3_rotate_by_angle_in_place` (`0x004025c0`) is `ai_check_builder`'s third method and carries no class prefix: it is a pure vector
rotation. The class membership is recorded in its docblock and its twin cell.

## 5. The packed descriptor

Both methods take one packed dword and expand it. They are the integer and the scaled-float halves of the same word shape.

| Bits | `ai_check_desc_unpack_data` (integers) | `ai_check_desc_unpack_ranges` (floats) |
|---|---|---|
| `[31:24]` | `+0x00` | `+0x14 = value * 0.25` |
| `[23:20]` | `+0x04` | part of `[23:16]` |
| `[19:16]` | `+0x08` | part of `[23:16]` |
| `[23:16]` | split into the two nibbles above | `+0x18 = value * 0.25` |
| `[15:8]` | `+0x0c` | part of `[15:0]` |
| `[7:0]` | `+0x10` | part of `[15:0]` |
| `[15:0]` | split into the two bytes above | `+0x1c = value * 0.01` |

The `* 0.01` is the project's hundredths convention, the one `ai_path_is_pos_in_connection_corridor` uses for
web-connection widths and `ai_loc_update` passes through: `[15:0]` is a world-unit distance stored as hundredths, and the two `* 0.25`
fields are quarter-unit quantities.

### 5a. Both words are fields of a navigation-web node

Both callers read the word out of a `C_web_node` (`structs/web_node.md`): the node array at `*(g_act_mission+0x2c)+4`, stride `0x1e`,
indexed by the route's node index.

| Unpacker | Node dword | Fields, and what reads them |
|---|---|---|
| `ai_check_desc_unpack_ranges` | `+0x10` | `ai_wayfind_plan_search_route` reads only `search_range` (node byte `+0x13`): it multiplies it into the search range and squares the result |
| `ai_check_desc_unpack_data` | `+0x14` | `ai_wayfind_score_and_add_checkpoint` reads only the two nibbles of node byte `+0x16` |

**The weights.** Each code maps to a weight, and the product is the checkpoint's desirability, multiplied by 0.25 within squared distance
100 and by a random factor when it beats the best so far:

| Code | `weight_code_a` | `weight_code_b` |
|---|---|---|
| 0 | 1.5 | 1.0 |
| 1 | 3.0 | 4.0 |
| 2 | 4.0 | 4.0 |
| 3 | 4.0 | 4.0 |
| 4 | 1.5 | 0.25 |
| 5 | 1.5 | 0.25 |
| 6 | 4.0 | 3.0 |
| 7 | 1.0 | 0.25 |
| 8 | 4.0 | 4.0 |
| 9 | 0.25 | 1.0 |

## 6. The direction tables

Each is 8 × `S_vector` = 24 floats, filled at start-up by its own static initialiser.

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_dir8_table_normalized` | `0x006550d8` | unit vectors 45° apart, y = 0; diagonals `0x3f3504f3` = 0.70710677 | `dir8_table_normalized_static_init` | `ai_check_builder_get_dir8_index`, `ai_check_builder_test_space_8dir` |
| `g_dir8_table` | `0x00655138` | the same 8 directions with ±1.0 diagonals, length √2, not normalised | `dir8_table_static_init` | - |
| `g_dir8_table_normalized_006551a8` | `0x006551a8` | a byte-identical second copy of the normalised table | `dir8_table_normalized_006551a8_static_init` | `ai_space_coverage_get_look_dir` |

MSVC emits the table once per translation unit and the linker keeps both copies. Entry order in all three: `(0,0,1)`, `(d,0,d)`, `(1,0,0)`,
`(d,0,-d)`, `(0,0,-1)`, `(-d,0,-d)`, `(-1,0,0)`, `(-d,0,d)`, starting at +Z and turning clockwise, which is what makes
`ai_check_builder_get_dir8_index`'s angle from entry 0 the angle from +Z.

**Initialisers.** All three tables live in writable data and are zero on disk, so the initialiser bodies are the only static evidence of their
contents. The first two are real function starts reached by a `jmp` from the static-init chain, with no dataref, the same shape as
`vec_up_00655198_static_init`, which fills the `(0,1,0)` up-vector `0x220` bytes further on (`structs/ai_wayfind.md` §4a); the up-vectors are
not part of any table. `dir8_table_normalized_006551a8_static_init` has no `E8` caller and no dataref, and one `E9` jump from
`thunk_dir8_table_normalized_006551a8_static_init` (`0x00425590`), whose address sits in the `_initterm` table in `.data` 20 bytes from the entry
of `dir8_table_normalized_static_init`'s thunk. Its VA inside the `C_game` code region is a linker artifact, not ownership.

### 6a. The tables are typed

Each table is one `S_vector[8]` symbol. **Nothing reads `g_dir8_table`:** every dword in every section of the image was checked for a value inside
`0x00655138..0x00655197`, and the only ones are the 24 stores of its own initialiser; the same scan finds every reader of the two normalised copies.

## 7. The rest of the VA range

Beside the class methods, `[0x00402740, 0x00403570]` holds `i3d_frame_get_bound_points_2d`, a free geometry helper hosted by `structs/ai_path.md`
(the 4 world-space corners of an `I3D_frame`'s 2D bounding rectangle, inflated `0.7`, at mid-height; each of the three `ai_path_find_spec_node_*`
searches calls it 4×), the two table initialisers, and the five `ai_wayfind` route-cost callbacks.

### 7a. `0x00403360` is not a function

`0x00403360` is an alternate exit block of `ai_wayfind_cb_forward_edge_cost` (`0x004030a0`..`0x00403530`):

* no `E8` caller and no dataref;
* an earlier `jcc rel8` at `0x00403346` targets it;
* no 16-aligned NOP boundary before it;
* its entry bytes `33 c0 5f 5e 5b` = `xor eax,eax; pop edi; pop esi; pop ebx` are a teardown fragment.

Two other exit arms of the same function, `0x0040331d` and `0x0040342e`, have the same shape.

## 8. PC vs PS2

- **Size ratios:** `TestSpace8RExt` 1280 / 768 = 1.67, with the same two `RotateVect` calls on both builds. `RotateVect` 116 / 96 = 1.21. `GetOrientation` 300 / 160 = 1.875:
  it computes `(theta + pi/8) / (pi/4)` explicitly, and the PC's x87 block that does the same is dropped from the render (§4b). `UNPACKDATA` 72 / 64 = 1.125.
  `UNPACKRANGES` 212 / 112 = 1.89: MIPS integer-to-float sequences that x86 does in one instruction each.
- **The line test differs.** The PC space test calls `collision_test_line_vertical_thunk` 3× (a byte scan finds the three `E8` calls), whose twin is
  `TestLineV__11g_collision`; `TestSpace8RExt__16ai_check_builder` calls `TestLineH__11g_collision` at the same three points, with the same argument shape (Open item 1).
- **Table symbols.** `g_dir8_table_normalized` is PS2 **`Dir8TableN`** (103 refs); `g_dir8_table` is PS2 **`Dir8Table`** (99 refs). The tables are byte-identical
  in content and order across builds, which is what let the PS2 data symbols name the PC globals. On the PS2, `GetLook__17ai_space_coverage` and
  `TestSpace8RExt__16ai_check_builder` both read the single `Dir8TableN`; on the PC the two readers use the two copies. The PS2 `Dir8Table` references have no PC
  counterpart (§6a).
- **PS2 emission order matches the PC's** in both classes, giving a forced 3-to-3 and 2-to-2.
- **`ai_check_builder` has no instance state on the PS2 either:** its callers pass the `ai_manager` root `0x509080` as `this`.
- **`UNPACKDATA` and `UNPACKRANGES` write the same record offsets on the PS2** (`auStack_b0` in `WeightCheckpoints__10ai_wayfind`).
- **`i3d_frame_get_bound_points_2d`** has a standalone PS2 counterpart with an empty class column, next to five per-class `GetBoundPoints2D` overloads (`C_actor`,
  `C_human`, `C_door`, `C_railway`, `C_don_ocas`). The PS2 calls it ×5 per search where the PC calls it ×4: the PS2's `a == b` branch and its "a is nearer" branch
  each build the same frame's bound points, and the PC emits that block once (`LAB_0040a7e5` in `ai_path_find_spec_node_from_positions`).

### 8a. Why the two `ai_check_desc` twins are confirmed

Neither is matched on size. Each has exactly one PC caller, and that caller's confirmed PS2 twin calls exactly that unpacker:

| PC function | Its sole PC caller | That caller's PS2 twin | Which calls |
|---|---|---|---|
| `ai_check_desc_unpack_data` | `ai_wayfind_score_and_add_checkpoint` | `WeightCheckpoints__10ai_wayfind` | `UNPACKDATA` |
| `ai_check_desc_unpack_ranges` | `ai_wayfind_plan_search_route` | `FindWaySEARCH__10ai_wayfind` | `UNPACKRANGES` |

## 9. Related

- `structs/ai_wayfind.md` — the route planner; strategy 6 and the checkpoint scorer are `ai_check_desc`'s consumers.
- `structs/ai_space_coverage.md` — the consumer of the space test and reader of the second normalised table.
- `structs/ai_path.md` — hosts the next unit, and `ai_path_find_spec_node_*` call `i3d_frame_get_bound_points_2d`.
- `structs/web_node.md` §2b — the node dwords.
- `structs/ai_manager.md` — the root that holds the `ai_check_builder` member.

## 10. Open items

1. Whether the PC's `TestLineV` route gives the space test a different result from a horizontal line test (§8): the call is measured, its effect on a ray with y = 0 is not. `structs/collision.md` describes the vertical test as the horizontal walk's near-vertical case.
