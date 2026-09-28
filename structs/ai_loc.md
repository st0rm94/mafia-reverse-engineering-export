# `ai_loc` — the per-character navigation-locality cache

**Created** 2026-07-30 12:49 +0200 · **Modified** 2026-09-17 13:45 +0200

An `ai_loc` answers one question for one character, cheaply and repeatedly: **"where am I in the
navigation world, and what map geometry is around me?"** It caches the character's resolved
`C_web_node` indices, the web connection its body is standing inside, and a short list of the
`check_seg` **specials** (cover corners, frame-anchored obstacles) whose spheres currently contain
it. Every field is refreshed lazily, so the AI can ask those questions per frame without running a
spatial search per frame.

```
  ai_wayfind  ->  ai_path              structs/ai_wayfind.md · structs/ai_path.md
        ^
  check_seg_clusters                   structs/check_seg.md — the specials this class records
        ^
  ai_loc   (one per C_human, @ +0x724) THIS FILE
```

| | |
|---|---|
| PC prefix | `ai_loc_*` |
| Owner | embedded in **`C_human` at `+0x724`** ¹ |
| PC cluster | `[0x0048e7d0, 0x0048ee60)`, one translation unit shared with `ai_blob_log` |
| Methods | 4 PS2 methods, 4 PC functions, all mapped and body-diffed (§2) |

¹ The sole `ai_loc_update` callsite, inside `human_update`, loads the receiver as
`lea ecx,[esi+0x724]`. The neighbour `ai_blob_log` sits at `+0x758` and `ai_loc` is `0x34` bytes —
`0x724 + 0x34 = 0x758` exactly (`structs/ai_blob_log.md`).

The class is found through the consumers of `check_seg_get_special`: there are seven, six of them
discriminate on the special's type code (`== 1` or `== 2`), and the seventh — `ai_loc_update` —
records all four types indiscriminately (§3).

Confidence: **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Object layout (`0x34` bytes)

```c
typedef struct ai_loc {                      // 0x34 bytes; embedded in C_human at +0x724
    /* +0x00 */ float     cached_pos[3];     // [H] the owner frame's origin at the last refresh;
                                             //     the move test compares against this
    /* +0x0c */ uint32_t  node_a;            // [H] resolved C_web_node ARRAY INDEX
    /* +0x10 */ uint32_t  node_b;            // [H] second node index - the other end of the
                                             //     connection when one was resolved
    /* +0x14 */ uint32_t  connection_idx;    // [H] index of the web connection the owner is
                                             //     standing inside, or unset
    /* +0x18 */ uint32_t  flags;             // [H] see §1a
    /* +0x1c */ uint32_t  last_refresh_ms;   // [H] mission clock at the last refresh; drives the
                                             //     5000 ms interval
    /* +0x20 */ uint8_t   _alloc;            // [M] MSVC std::vector empty-allocator member,
                                             //     padded to 4 - the same shape check_seg
                                             //     documents at its own +0x00 (check_seg.md §1a)
    /* +0x24 */ ai_seg_stat *stats_begin;    // [H] vector<ai_seg_stat> - the nearby specials
    /* +0x28 */ ai_seg_stat *stats_end;      // [H] count = (end - begin) >> 4
    /* +0x2c */ ai_seg_stat *stats_cap;      // [H] freed by `vector_dispose_at_0x24`, an ICF
                                             //     fold shared with `ai_logs` that belongs
                                             //     to neither class
    /* +0x30 */ C_actor  *owner;             // [H] set once by ai_loc_init; typed C_actor by the
                                             //     signature though every caller is C_human
} ai_loc;                                    // 0x34
```

**`node_a` / `node_b` hold `C_web_node` ARRAY INDICES, not the stable node IDs** at
`web_node+0x0e` (`structs/web_node.md`); `web_path_find_node_id` is the converter.

### 1a. The flag word at `+0x18`

Assembled by `ai_loc_update` and never read by this class — the consumers are elsewhere.

| bits | written when |
|---|---|
| `0x01` | a node was resolved successfully [H] |
| `0x02` | set together with `0x01` (as `\| 3`) when the position is **not** inside a connection corridor [H] |
| `0x10` | set together with `0x01` (as `\| 0x11`) when the position **is** inside a connection corridor, alongside `connection_idx` [H] |
| others | [U] |

### 1b. `ai_seg_stat` — one observed special (`0x10` bytes)

The element type name comes from the PS2 template argument
`insert__Q23std62__vector_imp<>F**P11ai_seg_stat**Ui…`; the PC shows only `0x10`-stride rows.

```c
typedef struct ai_seg_stat {                 // 0x10 bytes
    /* +0x00 */ uint32_t  type;              // [H] the special's check_seg type code: 1/2/4/8
    /* +0x04 */ uint32_t  last_seen_ms;      // [H] mission clock, refreshed on every re-observation
    /* +0x08 */ uint32_t  hit_count;         // [H] how many refreshes this record has had
    /* +0x0c */ void     *payload;           // [H] the special's payload pointer - THE RECORD KEY
} ai_seg_stat;                               // 0x10
```

**The key is the payload POINTER**, not the type and not a (segment, special) index pair. A record
is therefore valid only as long as the loaded `clusters.seg` data it points into, which is why
`entity_save_action_man` converts such pointers to index pairs when it writes a savegame
(`structs/check_seg.md` §3).

**Lifetime:** a record is created or refreshed by `ai_loc_add_or_refresh_seg_stat` and expires
**4000 ms** after its last refresh (`ai_loc_prune_stale_seg_stats`), against a refresh interval of
**5000 ms** in `ai_loc_update`. Since 4000 < 5000, a character standing still does **not** hold its
list: entries expire in the gap between refreshes and are re-added on the next one. The constants
are the same on both builds, so this is not a port artifact.

## 2. Method family

| PC VA | PC name | PS2 twin | status | role / decisive evidence |
|---|---|---|---|---|
| `0x0048e7d0` | `ai_loc_init` | `Init__6ai_locFP7C_actor` | CONFIRMED | binds the cache to its owner, seeds `cached_pos` from the owner's world frame, zeroes `flags` + `last_refresh_ms` so the first refresh is unconditional ¹ |
| `0x0048e810` | `ai_loc_update` | `Update__6ai_locFv` | CONFIRMED | the throttled refresh (§3) ² |
| `0x0048eb50` | `ai_loc_add_or_refresh_seg_stat` | `AddSegment__6ai_locFUiPv` | CONFIRMED | upsert keyed on `payload` at record `+0x0c` ³ |
| `0x0048edb0` | `ai_loc_prune_stale_seg_stats` | `SegListUpdate__6ai_locFv` | CONFIRMED | reverse walk, expire at **4000 ms** ⁴ |

¹ Same owner store, same frame field, same `& 0x20` cached-matrix guard + `UpdateWMatrixProc`, same
two zeroed fields.
² Shared magics **5000** (interval) and **4.0** (squared move threshold); identical `| 0x11` / `| 3`
flag ors; identical node arithmetic (each build's node stride).
³ Same key field, same refresh-or-insert split, same three written fields in order; `FUiPv` matches
the two visible args. The PS2 name describes an intent the body does not carry out: it adds no
*segment* — its element is an observation record about one **special**, and the common path is a
refresh, not an add.
⁴ Same iteration direction, same timestamp field, same tail-of-`Update` call position. The PS2 name
`SegListUpdate` overstates it: the body only removes expired entries.

## 3. What `ai_loc_update` does

The sole caller is `human_update`. The body is a chain of cheap exits before any spatial work:

1. **Gate:** runs only while the owner's `+0x98` is 0. `entity+0x98` is the actor's **current
   vehicle** (both builds read that field in `ai_fight_followship_add_car_member` and test the
   result's `kind == 4`, car), so the cache refreshes **only while the character is on foot** — a
   passenger's nav-web position is meaningless.
2. **Bail:** if the mission has no navigation web (`*(g_act_mission+0x2c) + 0x0c == 0`), clear
   `flags` and return.
3. **Throttle:** take the owner frame's origin; do nothing further unless the owner has moved more
   than 2 units (**4.0** squared) from `cached_pos` **or 5000 ms** have elapsed since
   `last_refresh_ms`. The mission clock is `*(g_act_mission+0x24) + 0x2b0c`.
4. **Resolve:** `ai_wayfind_get_node_for_position` → `node_a`; on success walk the node's
   connections and use `ai_path_is_pos_in_connection_corridor` to find the containing connection,
   setting `node_b`, `connection_idx` and the `0x11` flag pair; otherwise the `3` pair.
5. **Rebuild the specials list:** for every `check_seg` segment whose node-index list contains the
   resolved node (`check_seg_has_index`), test each special with
   `check_seg_is_pos_in_any_special` and pass the survivors to
   `ai_loc_add_or_refresh_seg_stat`.
6. **Expire:** `ai_loc_prune_stale_seg_stats`.

**Step 5 is the only `check_seg_get_special` consumer that does not discriminate on the special's
type code.** Every other consumer tests `== 1` or `== 2`, so `ai_loc`'s list is the one place where
a type-4 or type-8 special is *recorded* — and no function on either build is known to read that
list (§6).

## 4. Related

- `structs/check_seg.md` — the specials this class records; §1f the `ai_seg_corner` layout.
- `structs/ai_blob_log.md` — the adjacent sub-object at `C_human+0x758`.
- `structs/web_node.md` — what `node_a` / `node_b` index into (stride `0x1e`).
- `structs/ai_path.md` — `ai_path_is_pos_in_connection_corridor`, the corridor test.
- `structs/human.md` — the owner class and its sub-object offsets.

## 5. PC vs PS2

- **Object size: PC `0x34` / PS2 `0x34`** — the same, but the fields are *not* at the same
  offsets, because the head vector is 3 floats on PC and 4 on PS2:

  | field | PC | PS2 |
  |---|---|---|
  | cached position | `+0x00` (3 floats) | `+0x00` (4 floats) |
  | node_a / node_b | `+0x0c` / `+0x10` | `+0x10` / `+0x14` |
  | connection idx | `+0x14` | `+0x18` |
  | flags | `+0x18` | `+0x1c` |
  | last refresh ms | `+0x1c` | `+0x20` |
  | stats vector | `+0x24` (MSVC `std::vector`, allocator byte at `+0x20`) | `+0x24` (`std::__vector_imp`, count at `+0x28`, data at `+0x2c`) |
  | owner | `+0x30` | `+0x30` |

  The head band is shifted by exactly one float and the tail band has **zero** delta — the same
  band-wise pattern as `check_seg`. Rebasing a single offset across builds produces nonsense; use
  the table.
- **Owner offset: PC `C_human+0x724` ↔ PS2 `+0x780`** (delta `0x5c`); PS2 reads it from
  `Init__6ai_locFP7C_actor(iVar2 + 0x780, iVar2)`.
- **Current vehicle:** PC `entity+0x98` ↔ PS2 `entity+0x224`.
- **Frame field** in `Init`: PC `+0x68` ↔ PS2 `+0x70`.
- **Node arithmetic:** PC `/0x1e` ↔ PS2 `>>5`.
- **Mission clock: PC `*(g_act_mission+0x24) + 0x2b0c` ↔ PS2 `*(act_mission+0x20) + 0xe54`.**
- **The timing constants are identical** — 5000 ms refresh, 4000 ms expiry, and the 4.0 squared
  move threshold. The two composite flag writes (`| 3`, `| 0x11`) are byte-identical and pin
  `ai_loc_update` to its twin.
- **`ai_seg_stat` is `0x10` bytes on both builds** with its four fields at the same offsets — it
  contains no vector, so there is nothing to pad.
- **Size ratios are a factoring difference, all in one direction:** PS2 calls
  `insert__…<ai_seg_stat>` / `erase__…<ai_seg_stat>` out of line where MSVC inlines the whole
  reallocate-and-shift, so the two PC functions that manipulate the vector are *bigger* than their
  twins (`ai_loc_add_or_refresh_seg_stat` 0.32, `ai_loc_prune_stale_seg_stats` 0.70) while the two
  that do not are in band (`ai_loc_init` 88/64 = 1.375, `ai_loc_update` 1136/832 = 1.37).
- **No PS2-only methods**, and no PC-only functions in the cluster.

## 6. Open items

| item | state |
|---|---|
| who reads the `ai_seg_stat` list? | nothing found on either build. On PS2 the string `ai_seg_stat` occurs 4 times, all `insert`/`erase` template instantiations. Candidates: an inlined loop in a `C_human` or AI method that walks `this+0x748` (= `+0x724 + 0x24`) directly, or a consumer Ghidra renders through a register. This is the remaining route to the type-4/8 payload meanings |
| `+0x20` allocator byte | [M], inferred from the `check_seg` precedent (MSVC's empty allocator member padded to 4) and the absence of such a gap on PS2; byte-proving it needs the disassembly |
| the 4000-vs-5000 ms asymmetry (§1b) | deliberate hysteresis or not; needs an in-game observation |
