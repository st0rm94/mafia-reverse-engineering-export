# `ai_check_builder` / `ai_check_desc` — the 8-direction space test and the descriptor unpackers

**Created** 2026-07-30 13:41 +0200 · **Modified** 2026-09-28 19:40 +0200

Two small classes (PS2 class names) that both serve the navigation searches, and that both live in
the translation unit immediately *below* the `ai_wayfind` / `ai_path` cluster:

- **`ai_check_builder`** (3 methods) — asks *"how much open space is there around this point?"* by
  probing 8 compass directions with collision rays, plus the two helpers that make that possible.
- **`ai_check_desc`** (2 methods) — unpacks a **packed descriptor dword** into fields, one method
  for integers and one for scaled float ranges. Its consumers are route strategy 6
  (`ai_wayfind_plan_search_route`) and the checkpoint scorer.

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
                          g_dir8_table_normalized / g_dir8_table   (§3)
```

- **PC prefixes:** `ai_check_builder_*` · `ai_check_desc_*`.
- **Coverage:** 5 of 5 methods mapped, both classes complete, every row body-diffed (§5).
- **PC cluster:** `[0x004022c0, 0x004035e0)`, interleaved with the `vec3_*` math helpers, two CRT
  static-init bodies and the five `ai_wayfind` route-cost callbacks (§6).
- **No singleton, no owner offset.** Every method is a free function or takes its receiver as an
  out-parameter; neither class has instance state.

The classes are identified through the **data** they touch: the two static direction tables (§3)
carry named PS2 data symbols, and once a function is known to walk an 8-entry direction table, its
PS2 method names follow.

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. `ai_check_builder` — the 8-direction space test

There is no object. The class is three free functions over the direction tables.

### 1a. What the space test does

`ai_check_builder_test_space_8dir(pos, radius, out_threshold)` -> uint bitmask:

1. for each of the **8 unit vectors** in `g_dir8_table_normalized` (45° apart, y = 0):
2. scale it by `radius` and cast a **vertical** collision line from `pos` along it, taking the
   distance to the hit — **or `radius` itself when the probe is clear**, so "no obstacle" and
   "obstacle exactly at the radius" produce the same value;
3. re-rotate the ray by **`0.25`** and probe again; re-rotate by **`−0.5`** and probe a third time;
4. store `(probe1 + probe2 + probe3) × 0.3334` — the **mean of the three probes** — as that
   direction's clearance, in a **caller-invisible local array** (`local_20[]`).

So the scan is **24 collision queries** and yields 8 clearances. The tail:

```c
fVar1 = local_54 * 0.125 - 0.25;                       // mean of the 8 clearances, minus 0.25
do { if (fVar1 <= local_20[i]) local_60 |= 1 << i; } while (i < 8);   // x87 fcom / test ah,0x41
if (param_3 == (float *)0x0) return local_60;
*param_3 = fVar1;                                       // ONE store, NULL-guarded
```

* **The third parameter is one optional float**, not an array: it receives the computed threshold,
  and NULL is tolerated.
* **The return value is a bitmask**: bit *i* is set iff direction *i*'s clearance is at least
  the 8-direction **mean − 0.25**, i.e. *"directions with above-average clearance"*. Its consumer is
  `ai_space_coverage_compute`, which stores it at `coverage+0x50` as the direction-permission mask,
  and `ai_space_coverage_tick` tests it bit by bit with `1 << i`. `ai_space_coverage` is `0x5c`
  bytes, with the threshold float at `+0x54`.

The loop's cursor starts at `g_dir8_table_normalized[0].z` and reads each entry as `[-2]`, `[-1]`,
`[0]` (§3a).

### 1b. The direction→index helper

`ai_check_builder_get_dir8_index(dir)` quantises a direction to one of the 8 slots: normalise the
`(x, z)` pair (guarded by `1e-08`), measure the signed angle against the table's entry 0 — the
reference direction `(0, 0, 1)` — and round to the nearest 45°. It returns a table **index**, not an
orientation.

**The PC decomp does not show the rounding arithmetic.** It runs on the x87 stack and Ghidra drops
it, leaving only the trailing `__ftol` — the same FPU render artifact that makes `vec3_dist_sq`
render as `return;` despite 91 callers. The PC body is not a mismatch (the rounding formula is in
§5).

### 1c. The third method keeps its own name

`vec3_rotate_by_angle_in_place` (`0x004025c0`) is `ai_check_builder`'s third method, and it
deliberately carries no `ai_check_builder_*` prefix: it is a pure vector rotation, and the name says
what it is. The class membership is recorded in its docblock and its twin cell.

---

## 2. `ai_check_desc` — the packed descriptor

Both methods take **one packed dword** and expand it. They are the integer and the scaled-float
halves of the same descriptor word.

| Bits | `ai_check_desc_unpack_data` (integers) | `ai_check_desc_unpack_ranges` (floats) |
|---|---|---|
| `[31:24]` | out[0] | `this+0x14 = value * 0.25` |
| `[23:20]` | out[1] (4 bits) | *(part of the `[23:16]` byte below)* |
| `[19:16]` | out[2] (4 bits) | *(part of the `[23:16]` byte below)* |
| `[23:16]` | *(split into the two nibbles above)* | `this+0x18 = value * 0.25` |
| `[15:8]` | out[3] | *(part of the `[15:0]` word below)* |
| `[7:0]` | out[4] | *(part of the `[15:0]` word below)* |
| `[15:0]` | *(split into the two bytes above)* | `this+0x1c = value * 0.01` |

**The `* 0.01` is the project's hundredths fixed-point convention** — the same one
`ai_path_is_pos_in_connection_corridor` uses for web-connection widths, and the same one
`ai_loc_update` passes through. So the `[15:0]` field is a **world-unit distance stored as
hundredths**, and the two `* 0.25` fields are quarter-unit quantities. What the descriptor as a
whole describes is open (§4).

### 2a. Why both mappings are confirmed

Neither is matched on size. Each has **exactly one PC caller**, and that caller's confirmed PS2
twin calls exactly that unpacker:

| PC function | its sole PC caller | that caller's PS2 twin | which calls |
|---|---|---|---|
| `ai_check_desc_unpack_data` | `ai_wayfind_score_and_add_checkpoint` | `WeightCheckpoints__10ai_wayfind` | `UNPACKDATA` |
| `ai_check_desc_unpack_ranges` | `ai_wayfind_plan_search_route` | `FindWaySEARCH__10ai_wayfind` | `UNPACKRANGES` |

---

## 3. The direction tables

Each is 8 × vec3 = 24 floats, filled at start-up by its own C++ static initialiser.

| PC base | PC symbol | Written by | Contents |
|---|---|---|---|
| `0x006550d8` | `g_dir8_table_normalized` | `crt_init_dir8_table_normalized_006550d8` | **unit** vectors, 45° apart, y = 0; diagonals `0x3f3504f3` = **0.70710677** |
| `0x00655138` | `g_dir8_table` | `crt_init_dir8_table_unnormalized_00655138` | the same 8 compass directions with **±1.0** diagonals — length √2, deliberately **un-normalised**: a grid-step / neighbour-offset table |
| `0x006551a8` | `g_dir8_table_normalized_006551a8` | `crt_init_dir8_table_normalized_006551a8` | a **byte-identical second copy** of the normalised table ¹ |

¹ Same 24 floats, same order, same `0x3f3504f3` diagonals. `ai_check_builder_test_space_8dir` reads
the `0x006550d8` copy and `ai_space_coverage_get_look_dir` reads the `0x006551a8` copy: MSVC emits
the table once per translation unit and the linker keeps both copies.

Entry order is the same in all three: `(0,0,1)`, `(d,0,d)`, `(1,0,0)`, `(d,0,−d)`, `(0,0,−1)`,
`(−d,0,−d)`, `(−1,0,0)`, `(−d,0,d)` — i.e. **starting at +Z and turning clockwise**, which is what
makes `ai_check_builder_get_dir8_index`'s "angle from entry 0" the angle from +Z.

**Initialisers.** All three live in writable data and are **zero in the file on disk**, so the
initialiser bodies are the only static evidence of each table's contents. The first two are
NOP-padded real function starts reached by a `jmp` from the static-init chain, with zero datarefs —
the same shape as `crt_init_vec_up_00655198`, which initialises the `(0,1,0)` up-vector `0x220`
bytes further on (`structs/ai_wayfind.md` §1a); those up-vectors are **not** part of any table.
`crt_init_dir8_table_normalized_006551a8` has 0 direct `E8` callers and 0 datarefs, and ONE `E9`
jmp from the thunk `thunk_crt_init_dir8_table_normalized_006551a8` (`0x00425590`), whose address
sits in the CRT `_initterm` table in `.data` **20 bytes from the entry pointing at
`crt_init_dir8_table_normalized_006550d8`'s thunk**. Its VA inside the `C_game` code region is a
linker artifact (MSVC emits a file-scope object's initialiser into the defining TU's .obj), not
ownership.

### 3a. All three tables are typed

Each table is one `S_vector[8]` symbol, so every access reads as `<table>[i].x` / `.y` / `.z`.
Real code reads `g_dir8_table_normalized` in two places: `ai_check_builder_get_dir8_index`
(entry 0's `x` and `z`) and `ai_check_builder_test_space_8dir` (a cursor that starts at
`&g_dir8_table_normalized[0].z`). `g_dir8_table`'s base is referenced only by its own
initialiser. `g_dir8_table_normalized_006551a8` is indexed by `ai_space_coverage_get_look_dir`.

---

## 4. Open items

| item | state |
|---|---|
| what the `ai_check_desc` descriptor word means (§2) | both methods are pure bit arithmetic; the meaning has to come from the `ai_check_search` structure that `ai_wayfind_plan_search_route` takes |
| why 3 probes per direction, at `0.25` and `−0.5` | the net effect reads as samples at the direction itself, `+0.25` and `−0.25` — a ±0.25 rad fan; needs the rotation helper's angle convention from the disassembly |
| where `g_dir8_table` (un-normalised) is read | no PC element symbol is read and its base is referenced only by its initialiser, so consumers reach it by computed addressing; the PS2 references (§5) are the way in |
| `i3d_frame_get_bound_points_2d` call count | PC calls it ×4 per `ai_path_find_spec_node_*` search, PS2 ×5 (§6); the mismatch of one is unexplained |

---

## 5. PC vs PS2

- **Method-for-method table** (PC VA order, PC name first):

  | PC VA | PC name | PS2 twin | Status | Ratio |
  |---|---|---|---|---|
  | 0x004022c0 | `ai_check_builder_test_space_8dir` | `TestSpace8RExt__16ai_check_builderFR8S_vectorfPf` | CONFIRMED | 1280/768 = 1.67 |
  | 0x004025c0 | `vec3_rotate_by_angle_in_place` | `RotateVect__16ai_check_builderFR8S_vectorf` | CONFIRMED | 116/96 = 1.21 |
  | 0x00402620 | `ai_check_builder_get_dir8_index` | `GetOrientation__16ai_check_builderFR8S_vector` | CONFIRMED | 300/160 = 1.875 |
  | 0x00403530 | `ai_check_desc_unpack_data` | `UNPACKDATA__13ai_check_descFUi` | CONFIRMED | 72/64 = 1.125 |
  | 0x00403570 | `ai_check_desc_unpack_ranges` | `UNPACKRANGES__13ai_check_descFUi` | CONFIRMED | 212/112 = 1.89 |

- **Table symbols.** `g_dir8_table_normalized` is PS2 **`Dir8TableN`** (103 refs); `g_dir8_table` is
  PS2 **`Dir8Table`** (99 refs). The tables are byte-identical in content and order across builds,
  which is what let the PS2 data symbols name the PC globals. On the PS2, `GetLook__17ai_space_coverage`
  and `TestSpace8RExt__16ai_check_builder` both read the single `Dir8TableN`; on the PC the two
  readers use the two copies (§3). Enumerating the PS2 `Dir8Table` references would name the PC
  readers of `g_dir8_table`.
- **PS2 emission order matches PC exactly** in both classes (`TestSpace8RExt` < `RotateVect` <
  `GetOrientation`; `UNPACKDATA` < `UNPACKRANGES`), giving a forced 3↔3 and 2↔2.
- **`GetOrientation` computes `(θ + π/8) / (π/4)` explicitly**; the PC's x87 block that does the same
  is dropped from the Ghidra render (§1b), which is the whole of that row's 1.875 ratio.
- **`UNPACKRANGES`' 1.89** is MIPS integer→float conversion sequences that x86 does in one
  instruction each; the mapping rests on the sole-caller evidence of §2a, not on the ratio.
- **`TestSpace8RExt`'s 1.67** is a real factoring difference: PS2 *calls* `RotateVect` where PC
  inlines the rotation at two of its three probe sites.
- **No PS2-only methods** in either class, and **no PC-only functions**; PS2 has no
  `ai_check_builder` instance state either.
- **`i3d_frame_get_bound_points_2d`** (§6) has a standalone PS2 counterpart with an empty `class`
  column, next to five per-class `GetBoundPoints2D` overloads (`C_actor`, `C_human`, `C_door`,
  `C_railway`, `C_don_ocas`). PS2 calls it ×5 per search where PC calls it ×4.

---

## 6. The rest of the VA range `[0x004022c0, 0x00403570]`

Besides the five class methods above, the range holds:

| PC name | What it is |
|---|---|
| `i3d_frame_get_bound_points_2d` | a **free** geometry helper — the 4 world-space corners of an `I3D_frame`'s 2D bounding rectangle, inflated `0.7`, at mid-height. Used ×4 by each of the three `ai_path_find_spec_node_*` searches |
| `crt_init_dir8_table_normalized_006550d8` · `crt_init_dir8_table_unnormalized_00655138` | the two static-init bodies (§3) |

### 6a. `0x00403360` is not a function

`0x00403360` is an **alternate exit block** of `ai_wayfind_cb_forward_edge_cost`
(`0x004030a0`–`0x00403530`), not a function start:

* 0 `E8` callers, 0 datarefs;
* an earlier `jcc rel8` at `0x00403346` targets it;
* no 16-aligned NOP boundary before it;
* its entry bytes `33 c0 5f 5e 5b` = `xor eax,eax; pop edi; pop esi; pop ebx` are a teardown
  fragment, and decompiled alone it reads `return (float10)_DAT_00000004;` — a read from address 4.

It is absent from Ghidra's default analysis. Two other exit arms of the same function, `0x0040331d` and
`0x0040342e`, have the same shape and are likewise not function starts.

---

## 7. Cross-references

- `structs/ai_wayfind.md` — the route planner; strategy 6 and the checkpoint scorer are
  `ai_check_desc`'s only consumers, and the five route-cost callbacks share this TU.
- `structs/ai_space_coverage.md` — the consumer of the space test and reader of the second
  normalised table.
- `structs/ai_path.md` — `ai_path_find_spec_node_*`, the callers of
  `i3d_frame_get_bound_points_2d`.
