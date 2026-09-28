# `C_web_node` — one navigation-web node

**Created** 2026-07-27 13:56 +0200 · **Modified** 2026-09-17 13:44 +0200

A `C_web_node` is one position in the navigation web, owning its own outgoing connection list. The
nodes live in a flat array on `C_web_path` (`structs/web_path.md`), kept sorted by X so that
`web_path_find_nearest_node` can binary-search it.

| | |
|---|---|
| PC prefix | **`web_node_`** |
| Record stride | **`0x1e` bytes** |
| PC functions | 7 — 6 mapped to the 8 PS2 methods (all CONFIRMED) + 1 PC-only accessor |

Confidence tags: **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Record layout (0x1e bytes)

```c
typedef struct C_web_node {              // 0x1e bytes; array base at C_web_path+0x04
    /* +0x00 */ float             x;     // [H] the sort key - web_path_sort_nodes_by_x orders on it
                                         //     and web_path_find_nearest_node binary-searches it
    /* +0x04 */ float             y;     // [H]
    /* +0x08 */ float             z;     // [H]
    /* +0x0c */ uint16_t          category; // [H] the node's CATEGORY / state flag word - see 1a.
                                         //     Read by web_path_is_connection_legal to decide
                                         //     whether two nodes may be joined; bit 0x4000 is a
                                         //     node-level SELECTED bit written by the endpoint
                                         //     resolvers. NOT the same word as the per-connection
                                         //     flags at C_web_connection+0x02 (web_path.md §2).
    /* +0x0e */ uint16_t          id;    // [H] the node's stable ID. NOT its array index -
                                         //     web_path_find_node_id converts one to the other, and
                                         //     connections store the neighbour's ID, not its index.
    /* +0x10 */ uint16_t          radius_cm; // [H] the node's ACCEPTANCE RADIUS in hundredths of
                                         //     a unit (a radius, not a squared radius).
                                         //     web_node_ctor seeds it to 0x32 (= 0.5 units) and
                                         //     the endpoint resolvers test
                                         //     dist^2 <= (field & 0xffff)^2 * 0.0001.
    /* +0x12 */ uint16_t          flags; // [M] node filter mask, tested against the `ushort` filter
                                         //     argument every web_path_find_* takes (§6)
    /* +0x14 */ uint8_t           _unk_14[4]; // [U]
    /* +0x18 */ C_web_connection *connections; // [H] this node's outgoing edge array, stride 8
    /* +0x1c */ uint8_t           _unk_1c;     // [U]
    /* +0x1d */ uint8_t           conn_count;  // [H] a BYTE - so a node has at most 255 connections
} C_web_node;                            // 0x1e
```

Everything except `+0x12` is [H], byte-proven on both builds. The `+0x00`/`+0x0e`/`+0x18`/`+0x1d`
group is the load-bearing part.

`web_path_resize_node_array` / `web_node_resize_connection_array` also treat `+0x1c` as the
connection-array **capacity** byte (§2), next to the live count at `+0x1d`.

### 1a. The `category` flag word at `+0x0c`

Settled by a statement-level diff of `web_path_is_connection_legal` against PS2 `LegalConnect`, which
reads this word on both endpoint nodes and decides legality from it.

| Bit | Meaning | Evidence |
|---|---|---|
| `0x0002` | a distinct node class; downgrades the resulting connection kind to **2** | the `(A\|B) & 2` arm of `web_path_is_connection_legal` |
| `0x000c` | a two-bit node category (bits 2\|3); may **not** be joined to a node in the `0x13` category — the sole reject condition. Yields connection kind **4** | the `0x0c` vs `0x13` incompatibility test ¹ |
| `0x0010` | a third node class; yields connection kind **0x10** | same function |
| `0x4000` | **SELECTED** (node-level): set then cleared around every endpoint resolve so the search can exclude nodes it has already tried | `ai_wayfind_resolve_start_node` / `ai_wayfind_resolve_end_node` ² |

¹ MSVC merges PS2's separate `&4`/`&8` tests into one `&0xc`.
² They set `|0x40` on the **byte at `+0x0d`**, the high byte of this ushort; PS2 sets `|0x4000` on
the ushort directly.

**Two different SELECTED bits exist.** This one is on the **node** (`node+0x0c` bit `0x4000`, used
by the route search as a visited marker). The other is on the **connection**
(`C_web_connection+0x02` bit `0x40`, the map editor's selection — `web_path.md` §2). A PC decomp line
reading `*(byte *)(x + 0xd) |= 0x40` is the NODE one; `*(byte *)(x + 3) |= 0x40` is the CONNECTION
one.

The remaining bits of `+0x0c` are unidentified; the `0x13` mask (bits 0\|1\|4) implies at least bit
`0x0001` is also meaningful.

**The connection record** (`C_web_connection`, 8 bytes: neighbour ID `+0x00`, flags `+0x02` with
`0x40` = SELECTED and `0x80` = ONE-WAY, cost `+0x04`) is documented in `structs/web_path.md` §2,
since its flag bits are only ever read and written by `web_path_*` methods.

## 2. Method family — PC ↔ PS2 (6 of 8 mapped, + 1 PC-only)

| PC VA | PC name | PS2 twin | Status | Role / evidence |
|---|---|---|---|---|
| 0x0040f010 | `web_node_get_connection` | - | STRUCTURAL | Bounds-checked connection accessor: `idx < node[0x1d] ? node[0x18] + idx*8 : 0`. PC-only ¹ |
| 0x006083c0 | `web_node_dtor` | `{dtor}__10C_web_node` | CONFIRMED | Destroys the stride-8 connection array at `+0x18` and frees it. It is the exact element-destructor pointer `web_path_load_nodes` and `web_path_dtor` pass to the array helper. Ratio 1.42 |
| 0x006083f0 | `web_node_resize_connection_array` | `Resize__10C_web_nodeFUsP10C_web_path` | CONFIRMED | Grows the connection array: capacity byte `+0x1c`, `operator_new(n*8 + 4)`, copy-and-free; live count `+0x1d` untouched. Ratio 1.18 ² |
| 0x006084d0 | `web_node_add_connection` | `AddConn__10C_web_nodeFUsR8S_vectorR8S_vectorUs` | CONFIRMED | Appends one connection: neighbour id `+0x00`, flags `+0x02`, and the **euclidean distance** at `+0x04` — the proof that `conn+0x04` holds a distance. Exact signature + callee multiset (`Resize`×1 + SQRT). Ratio 1.85 ³ |
| 0x00608590 | `web_node_delete_connection` | `DeleteConn__10C_web_nodeFUs` | CONFIRMED | Removes a connection by index using **swap-with-last**, so connection indices are NOT stable across a delete. Ratio 1.38; caller call-counts match (1 / 2 / 2) |
| 0x006085d0 | `web_node_find_connection_index` | `FindConnIdx__10C_web_nodeFUs` | CONFIRMED | Index of this node's connection to a given neighbour **ID**, or `0xffff`. Zero cross-build delta ⁴ |
| 0x0060b710 | `web_node_ctor` | `__ct__10C_web_nodeFv` | CONFIRMED | Zeroes the whole 0x1e-byte record and seeds `+0x10` to **0x32 (50)**. The ctor argument for the stride-0x1e array in `web_path_resize_node_array`, paired there with `web_node_dtor`; one use (no COMDAT fold) |

¹ GCC inlines it everywhere, so it is not among `C_web_node`'s 8 PS2 symbols.
² PS2 takes a trailing `C_web_path*` the PC build does not (PS2 `MergeNodes` passes 0).
³ Above the usual band — PS2 `S_vector&` copying.
⁴ Same `+0x1d` count, `+0x18` array, stride 8, same sentinel; `web_path_set_connection_one_way` and
`web_path_insert_node_in_connection` each call it twice, in the same positions, on both builds.

**Unmapped PS2 methods (2):** `GetPos`, `ClearConn` (§6).

### 2a. VA proximity is not evidence for this class

`web_node_get_connection` sits at PC `0x0040f010` — inside the **`ai_wayfind`/`ai_path` object
file**, roughly 2 MB below the `web_*` cluster at `0x006081e0`+. An **inline member function** gets
a COMDAT copy in every object file that uses it, `/OPT:ICF` folds them, and the survivor lands
wherever the linker put the winning object. Corroboration: both its callers are in that other
translation unit, and `ai_wayfind_cb_flee_edge_cost_directional` **inlines** the identical lookup
rather than calling it (the `/Ob` heuristic treating one site differently inside a single `.obj`).

So a VA far from a class's cluster is **not** evidence against class membership, and an unnamed
function inside a cluster is not evidence for it. The receiver's field signature decides — here
`+0x18` array / `+0x1d` count / stride 8, which is this record exactly.

## 4. PC vs PS2

- **Record stride: PC `0x1e`, PS2 `0x20`.** Every field offset is identical; the extra 2 bytes are a
  trailing alignment pad for MIPS. A field finding on either build transfers directly, but any code
  that *indexes* the array must use the right stride — that difference shows up in every scan loop
  (`* 0x1e` vs `* 0x20`, or `psVar + 0xf` shorts vs `pfVar[i * 8]` floats).
- `C_web_connection` has **zero delta** — 8 bytes on both builds, same field offsets.
- The `conn_count` byte at `+0x1d` and the array pointer at `+0x18` are identical on both builds,
  which is what made `FindConnIdx` a zero-delta match.
- PS2 confirms the array element type from `C_web_path::LoadNodes`, whose teardown passes
  `C_web_node::~C_web_node` to `__destroy_new_array`.
- **The class is emitted in three places on PS2 as well.** `GetPos` sits far outside the PS2 class
  cluster (`0x0011c210` vs the rest at `0x0039xxxx`), so its PC twin need not be inside the main
  `web_*` cluster either — the same inline-COMDAT effect as §2a.
- `ClearConn` (60 B) belongs to the same second-cluster group as the `C_web_path` `*HlpMem` trio
  (`structs/web_path.md` §5), not to the PC `web_node_*` cluster.

## 5. Cross-references

- `structs/web_path.md` — the owning graph, the connection flag bits, and the route searches.
- `structs/ai_wayfind.md` / `structs/ai_path.md` — the consumers.

## 6. Open items

| item | state |
|---|---|
| `+0x12` filter mask | [M]; not re-derived from a body read on either build |
| PC twin of `ClearConn` | not located; expected outside the `web_node_*` cluster (§4) |
| PC twin of `GetPos` | not located; low priority |
