# `C_web_node` — one navigation-web node

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-11-09 21:00 +0100

A `C_web_node` is one position in the navigation web, owning its own outgoing connection list. The
nodes live in a flat array on `C_web_path` (`structs/web_path.md`), kept sorted by X so that
`web_path_find_nearest_node` can binary-search it.

| | |
|---|---|
| PC prefix | `web_node_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x1e` (the record stride) |
| PC constructor | `web_node_ctor` @ `0x0060b710` |
| PC translation unit | `[0x00608390, 0x0060b740)`, the `web_path` unit (`structs/web_path.md`), guard to guard; `web_node_get_connection` is emitted in another unit (§4) |
| PS2 class | `C_web_node`, 8 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_web_node {              // 0x1e bytes; array base at C_web_path+0x04
    /* +0x00 */ float    x;              // [H] world position; the sort key of the node array (§2)
    /* +0x04 */ float    y;              // [H] world position, height
    /* +0x08 */ float    z;              // [H] world position, depth
    /* +0x0c */ uint16_t category;       // [H] the node's category / state flag word (§2a)
    /* +0x0e */ uint16_t id;             // [H] the stable ID, not the array index (§2)
    /* +0x10 */ uint16_t radius_cm;      // [H] acceptance radius in hundredths of a unit (§2, §2b)
    /* +0x12 */ uint8_t  range_b;        // [H] quarter units; unpacked, never read (§2b)
    /* +0x13 */ uint8_t  search_range_q; // [H] quarter units; the strategy-6 search range factor (§2b)
    /* +0x14 */ uint8_t  unread_14;      // [U] descriptor byte 0, unpacked; no reader (class census, §2b)
    /* +0x15 */ uint8_t  unread_15;      // [U] descriptor byte 1, unpacked; no reader (class census, §2b)
    /* +0x16 */ uint8_t  weight_codes;   // [H] two 4-bit checkpoint weighting codes (§2b)
    /* +0x17 */ uint8_t  unread_17;      // [U] descriptor byte 3, unpacked; no reader (class census, §2b)
    /* +0x18 */ C_web_connection *connections; // [H] this node's outgoing edge array, stride 8
    /* +0x1c */ uint8_t  conn_cap;       // [H] connection-array capacity, grown one element at a time
    /* +0x1d */ uint8_t  conn_count;     // [H] live connections; a byte, so at most 255
} C_web_node;                            // 0x1e
```

The filter argument every `web_path_find_*` takes is tested against the `+0x0c` category word (§2a), not against
any of the bytes at `+0x12..+0x17`.

**The connection record** (`C_web_connection`, 8 bytes: neighbour ID `+0x00`, flags `+0x02` with
`0x40` = SELECTED and `0x80` = ONE-WAY, cost `+0x04`) is documented in `structs/web_path.md` §1a,
since its flag bits are only ever read and written by `web_path_*` methods.

## 2. Field evidence

The whole record is read from `check.bin` by `web_path_load_nodes` and copied by `web_path_resize_node_array`,
which is every field's second writer; the cells below name those two only where nothing else writes the field.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | x | H | `web_node_ctor`, `web_path_load_nodes`, `web_path_resize_node_array` | `web_path_find_nearest_node`, `web_path_sort_nodes_by_x` | the sort key: `web_path_sort_nodes_by_x` orders on it and `web_path_find_nearest_node` binary-searches it |
| +0x4 | y | H | `web_node_ctor`, `web_path_load_nodes`, `web_path_resize_node_array` | `web_path_find_nearest_node` | - |
| +0x8 | z | H | `web_node_ctor`, `web_path_load_nodes`, `web_path_resize_node_array` | `web_path_find_nearest_node` | - |
| +0xc | category | H | `ai_wayfind_resolve_end_node`, `ai_wayfind_resolve_start_node`, `railway_game_init`, `web_node_ctor`, `web_path_clear_marker_bits`, `web_path_load_nodes` | `circuit_load_gates_from_scene`, `pedestrian_tick`, `railway_is_semaphore_point`, `web_path_find_nearest_node`, `web_path_is_connection_legal` | §2a; not the per-connection flags at `C_web_connection+0x02` |
| +0xe | id | H | `web_node_ctor`, `web_path_load_nodes` | `web_path_find_node_id` | `web_path_find_node_id` converts an ID to the array index, and connections store the neighbour's ID |
| +0x10 | radius_cm | H | `web_node_ctor`, `web_path_load_nodes` | `ai_check_desc_unpack_ranges`, `ai_wayfind_resolve_end_node`, `ai_wayfind_resolve_start_node` | the ctor seeds `0x32` (0.5 units); the resolvers test `dist^2 <= (field & 0xffff)^2 * 0.0001`; also the low half of the first descriptor dword (§2b) |
| +0x12 | range_b | H | `web_path_load_nodes` | `ai_check_desc_unpack_ranges` | no reader after the unpacking |
| +0x13 | search_range_q | H | `web_path_load_nodes` | `ai_check_desc_unpack_ranges` | reaches `ai_wayfind_plan_search_route` (§2b) |
| +0x14 | unread_14 | U | `web_node_ctor`, `web_path_load_nodes` | `ai_check_desc_unpack_data` | the ctor zeroes the dword; the unpacker's one caller, `ai_wayfind_score_and_add_checkpoint`, reads only `+0x16`, so no reader after the unpacking (class census) |
| +0x15 | unread_15 | U | `web_path_load_nodes` | `ai_check_desc_unpack_data` | no reader after the unpacking (class census) |
| +0x16 | weight_codes | H | `web_path_load_nodes` | `ai_check_desc_unpack_data` | reaches `ai_wayfind_score_and_add_checkpoint` (§2b) |
| +0x17 | unread_17 | U | `web_path_load_nodes` | `ai_check_desc_unpack_data` | no reader after the unpacking (class census) |
| +0x18 | connections | H | `web_node_ctor`, `web_node_resize_connection_array`, `web_path_load_nodes` | `web_node_add_connection`, `web_node_delete_connection`, `web_node_dtor`, `web_node_find_connection_index`, `web_node_get_connection`, `web_node_resize_connection_array` | stride 8 |
| +0x1c | conn_cap | H | `web_node_ctor`, `web_node_resize_connection_array`, `web_path_load_nodes` | `web_node_resize_connection_array` | - |
| +0x1d | conn_count | H | `web_node_add_connection`, `web_node_ctor`, `web_node_delete_connection`, `web_path_load_nodes` | `web_node_add_connection`, `web_node_delete_connection`, `web_node_find_connection_index`, `web_node_get_connection`, `web_node_resize_connection_array` | - |

### 2a. The `category` flag word at `+0x0c`

Settled by `web_path_is_connection_legal`, which reads this word on both endpoint nodes and decides
legality from it; the cross-build diff that confirms it is in §5.

| Bit | Meaning | Evidence |
|---|---|---|
| `0x0001` | the pedestrian WALKWAY class: crowd pedestrians search with the filter `1` (`pedestrian_activate`) or `3` (`pedestrian_activate_free`), held at `pedestrian+0x1f4` and passed to `web_path_find_nearest_node` | the three pedestrian callers of `web_path_find_nearest_node` |
| `0x0002` | a distinct node class; downgrades the resulting connection kind to **2** | the `(A\|B) & 2` arm of `web_path_is_connection_legal` |
| `0x000c` | a two-bit node category (bits 2\|3); may **not** be joined to a node in the `0x13` category — the sole reject condition. Yields connection kind **4** | the `0x0c` vs `0x13` incompatibility test ¹ |
| `0x0010` | a third node class; yields connection kind **0x10** | same function |
| `0x0200` | pedestrian TURN-BACK node: a walker that reaches one swaps its current and previous node and walks back | `pedestrian_tick` |
| `0x0800` | railway SEMAPHORE node: set by `railway_game_init`, which allocates an 8-byte record for the node; `railway_is_semaphore_point` tests the bit, then searches `g_rail_semaphore_list` for a record whose `+0x04` is the node | `railway_game_init`, `railway_is_semaphore_point` |
| `0x1000` | railway-pass marker: `railway_game_init` sets it on the nodes its pass walks and skips the pass when a railway's first node carries it; `game_init` clears it on every node through `web_path_clear_marker_bits` | the three functions |
| `0x4000` | **SELECTED** (node-level): set then cleared around every endpoint resolve so the search can exclude nodes it has tried | `ai_wayfind_resolve_start_node` / `ai_wayfind_resolve_end_node` ² |
| `0x8000` | no reader: no instruction tests, sets or clears it on its own; it can only match a caller's filter that includes it (the dog code passes `0xffff`) | a census of every bit operation on `+0x0c` and `+0x0d` in `.text` |

¹ The PC tests bits 2 and 3 as one `&0xc` (§5).
² They set `|0x40` on the **byte at `+0x0d`**, the high byte of this ushort (§5).

**Two different SELECTED bits exist.** This one is on the **node** (`node+0x0c` bit `0x4000`, used
by the route search as a visited marker). The other is on the **connection**
(`C_web_connection+0x02` bit `0x40`, the map editor's selection — `web_path.md` §1a). A PC decomp line
reading `*(byte *)(x + 0xd) |= 0x40` is the NODE one; `*(byte *)(x + 3) |= 0x40` is the CONNECTION
one.

The `0x13` mask covers bits 0\|1\|4. `web_path_find_nearest_node` accepts a node only when its
word shares a bit with the caller's filter and `0x4000` is clear.

**The word in the shipped webs.** In the node chunk of all 71 shipped `check.bin` files the low
byte is one-hot: every node carries exactly one of `0x0001`, `0x0002`, `0x0004`, `0x0008` and
`0x0010`. The high byte carries `0x0200` (1 708 nodes, all `0x0001`), `0x0800` (282, all
category 1 or 2), `0x1000` (20 135, all category 1 or 2) and `0x8000` (6 402, with `0x0001`,
`0x0002`, `0x0004` or `0x0010`). The shipped `0x1000` bits do not survive into play: `game_init`
clears the bit on every node.

**The two-bit category in the shipped webs.** A census of the node chunk of all 71 shipped
`check.bin` files finds the values 0, 1 (`0x0004`) and 2 (`0x0008`); value 3 occurs nowhere.
Value 2 is carried by 32 webs, and every value-2 node has exactly two connections:

| web | value-2 nodes | their connections |
|---|---|---|
| `MISE06-AUTODROM` | 26 of 860 | two each, neither one-way; one per race camera (`structs/racecam.md` §4b) |
| the 31 other webs | 23 to 79 | two each, exactly one of them one-way |

`circuit_load_gates_from_scene` is the one live reader that tests value 2 on its own. Outside the
race track nothing distinguishes value 2 from value 1 at run time: the railway code
(`railway_game_init`, `railway_save_game_load`, `wagon_ai`) searches with the filter `0xc` and
tests `0xc`, accepting both values alike, and the only other reader of value 2 alone is
`seagull_flock_game_init` (filter `8`), whose class is never constructed
(`structs/seagull_flock.md` §6).

### 2b. The two descriptor dwords at `+0x10` and `+0x14`

`web_path_load_nodes` reads the records from `check.bin` byte for byte, `web_path_resize_node_array`
copies `+0x10` and `+0x14` as two whole dwords, and `web_node_ctor` zeroes both except the 0.5-unit
radius. No code reads the bytes `+0x12..+0x17` one by one: they are read only as two packed words,
through the two `ai_check_desc` unpackers (`structs/ai_check.md` §5).

| Byte | Unpacked by | Read by | Meaning |
|---|---|---|---|
| `+0x10..+0x11` | `ai_check_desc_unpack_ranges`, `[15:0]` × 0.01 | the endpoint resolvers, directly | the acceptance radius |
| `+0x12` | `ai_check_desc_unpack_ranges`, `[23:16]` × 0.25 | nothing | a quarter-unit value |
| `+0x13` | `ai_check_desc_unpack_ranges`, `[31:24]` × 0.25 | `ai_wayfind_plan_search_route` | multiplied into the strategy-6 search range, whose square it stores back |
| `+0x14` | `ai_check_desc_unpack_data`, out[4] | nothing | - |
| `+0x15` | `ai_check_desc_unpack_data`, out[3] | nothing | - |
| `+0x16` | `ai_check_desc_unpack_data`, out[1] (high nibble) and out[2] (low nibble) | `ai_wayfind_score_and_add_checkpoint` | two codes 0..9, each mapped to a weight (0.25 / 1.0 / 1.5 / 3.0 / 4.0); their product, randomised, is the node's desirability as a search checkpoint |
| `+0x17` | `ai_check_desc_unpack_data`, out[0] | nothing | - |

**What the shipped maps carry.** A census of the node chunk of all 71 shipped `check.bin` files
(494 611 nodes): 259 795 nodes (53 %) have all six bytes `+0x12..+0x17` zero. Where they are set,
`+0x12` holds 0..7, `+0x13` holds `0x1a..0x28` (6.5–10.0 units), both nibbles of `+0x16` hold
0..9, `+0x17` is almost always 0, and `+0x14` / `+0x15` hold mask-like values (`0xff`, `0xef`,
`0xfb`, `0xfe`, …). What `+0x12`, `+0x14`, `+0x15` and `+0x17` were for is not recoverable from
the binary: nothing reads them.

## 3. Method family

6 of the 8 PS2 methods mapped, all CONFIRMED, plus 1 PC-only function; the other 2 are in §5a. Ratios are PC / PS2 body size.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x006083c0 | `web_node_dtor` | instance | `__dt__10C_web_nodeFv` | 0x003905f0 | CONFIRMED | destroys the stride-8 connection array at `+0x18` and frees it; the exact element-destructor pointer `web_path_load_nodes` and `web_path_dtor` pass to the array helper. Ratio 1.42 |
| 0x0060b710 | `web_node_ctor` | instance | `__ct__10C_web_nodeFv` | 0x00390fd0 | CONFIRMED | zeroes the whole `0x1e`-byte record and seeds `+0x10` to `0x32`; the ctor argument for the stride-`0x1e` array in `web_path_resize_node_array`, paired there with `web_node_dtor`; one use (no COMDAT fold) |

### 3b. Accessors

None: `web_node_get_connection` bounds-checks its index.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0040f010 | `web_node_get_connection` | - | - | - | bounds-checked connection accessor: `idx < node[0x1d] ? node[0x18] + idx*8 : 0`. PC-only ¹ |
| 0x006083f0 | `web_node_resize_connection_array` | `Resize__10C_web_nodeFUsP10C_web_path` | 0x00390640 | CONFIRMED | grows the connection array: capacity byte `+0x1c`, `operator_new(n*8 + 4)`, copy-and-free; live count `+0x1d` untouched. Ratio 1.18 ² |
| 0x006084d0 | `web_node_add_connection` | `AddConn__10C_web_nodeFUsR8S_vectorR8S_vectorUs` | 0x003907b0 | CONFIRMED | appends one connection: neighbour id `+0x00`, flags `+0x02`, and the **euclidean distance** at `+0x04`, the proof that `conn+0x04` holds a distance. Exact signature + callee multiset (`Resize`×1 + SQRT). Ratio 1.85 ³ |
| 0x00608590 | `web_node_delete_connection` | `DeleteConn__10C_web_nodeFUs` | 0x00390920 | CONFIRMED | removes a connection by index using **swap-with-last**, so connection indices are not stable across a delete. Ratio 1.38; caller call-counts match (1 / 2 / 2) |
| 0x006085d0 | `web_node_find_connection_index` | `FindConnIdx__10C_web_nodeFUs` | 0x00390980 | CONFIRMED | index of this node's connection to a given neighbour **ID**, or `0xffff`. Zero cross-build delta ⁴ |

### 3d. Free helpers

None; the `web_path` unit's helpers are listed in `structs/web_path.md`.

¹ GCC inlines it everywhere, so it is not among `C_web_node`'s 8 PS2 symbols.
² PS2 takes a trailing `C_web_path*` the PC build does not (PS2 `MergeNodes` passes 0).
³ Above the usual band — PS2 `S_vector&` copying.
⁴ Same `+0x1d` count, `+0x18` array, stride 8, same sentinel; `web_path_set_connection_one_way` and
`web_path_insert_node_in_connection` each call it twice, in the same positions, on both builds.

## 4. VA proximity is not evidence for this class

`web_node_get_connection` sits at PC `0x0040f010` — inside the **`ai_wayfind`/`ai_path` object
file**, roughly 2 MB below the `web_*` cluster at `0x006081e0`+. An **inline member function** gets
a COMDAT copy in every object file that uses it, `/OPT:ICF` folds them, and the survivor lands
wherever the linker put the winning object. Corroboration: both its callers are in that other
translation unit, and `ai_wayfind_cb_flee_edge_cost_directional` **inlines** the identical lookup
rather than calling it (the `/Ob` heuristic treating one site differently inside a single `.obj`).

So a VA far from a class's cluster is **not** evidence against class membership, and an unnamed
function inside a cluster is not evidence for it. The receiver's field signature decides — here
`+0x18` array / `+0x1d` count / stride 8, which is this record exactly.

## 5. PC vs PS2

- **Record stride: PC `0x1e`, PS2 `0x20`.** Every field offset is identical; the extra 2 bytes are a
  trailing alignment pad for MIPS. A field finding on either build transfers directly, but any code
  that *indexes* the array must use the right stride — that difference shows up in every scan loop
  (`* 0x1e` vs `* 0x20`, or `psVar + 0xf` shorts vs `pfVar[i * 8]` floats).
- `C_web_connection` has **zero delta** — 8 bytes on both builds, same field offsets.
- **The `category` word (§2a)** is settled by a statement-level diff of `web_path_is_connection_legal`
  against PS2 `LegalConnect`. MSVC merges PS2's separate `&4`/`&8` tests into one `&0xc`; where the
  PC endpoint resolvers set `|0x40` on the byte at `+0x0d`, the PS2 sets `|0x4000` on the ushort
  directly.
- The `conn_count` byte at `+0x1d` and the array pointer at `+0x18` are identical on both builds,
  which is what made `FindConnIdx` a zero-delta match.
- PS2 confirms the array element type from `C_web_path::LoadNodes`, whose teardown passes
  `C_web_node::~C_web_node` to `__destroy_new_array`.
- **The class is emitted in three places on PS2 as well.** `GetPos` sits far outside the PS2 class
  cluster (`0x0011c210` vs the rest at `0x0039xxxx`), so its PC twin need not be inside the main
  `web_*` cluster either — the same inline-COMDAT effect as §4. On the PC it is inlined (§5a).

### 5a. PS2 methods without a PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0011c210 | `GetPos__10C_web_nodeFv` | inlined: `airplane_fly_along_route`, `racing_create_segments` and `opponent_create_driver_path`, the twins of its three callers, read the node fields in place and call no `web_node_*` function |
| 0x003909e0 | `ClearConn__10C_web_nodeFb` | 60 B; its only caller, `C_web_path::ClearNode`, runs it per node; the PC inlines the non-arena part of `ClearNode` into `mission_close` (`structs/web_path.md` §7), and the connection arrays are released by `web_node_dtor` through `crt_ehvec_dtor` |

## 6. Related

- `structs/web_path.md` — the owning graph, the connection flag bits, and the route searches.
- `structs/ai_wayfind.md` / `structs/ai_path.md` — the consumers.

## 7. Open items

Nothing is open.
