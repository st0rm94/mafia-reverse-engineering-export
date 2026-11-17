# `ai_space_coverage` — the 8-direction "where should I go next" scorer

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-11-04 09:23 +0100

An `ai_space_coverage` answers one question for one AI decision: **"of the eight compass
directions, which one should I take?"** It combines three inputs into a per-direction score —
how much open space the direction has (measured once, by a collision probe), how strongly the
caller has asked for it (a bias the caller writes), and how long it has been since the character
last went that way (accumulated per tick) — and hands back the winner as a unit vector.

The third input is the interesting one: scores drift **away** from the direction currently being
taken, so a character that keeps choosing the same way gradually makes every other way more
attractive. It is a novelty accumulator, not a pathfinder.

```
  ai_action_manager  (action type 0x24 = _hostile_coverage)   structs/ai_action_manager.md
        |
        v
  ai_space_coverage   (score 8 directions, pick one)          THIS FILE
        |
        v
  ai_check_builder    (probe 8 directions for open space)     structs/ai_check.md
        |
        v
  g_dir8_table_normalized                                     structs/ai_check.md §6
```

| | |
|---|---|
| PC prefix | `ai_space_coverage_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x5c` (§2a) |
| PC constructor | - |
| PC translation unit | `[0x0040f700, 0x0043b000)`, the `C_entity` unit (`structs/entity.md` §8); this class is `[0x00439f70, 0x0043a150)` inside it, four consecutive functions |
| PS2 class | `ai_space_coverage` (tag `17ai_space_coverage`), 4 methods: 4 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The object is the first member of a `_hostile_coverage` action's payload (§4).

```c
typedef struct ai_space_coverage {          // 0x5c bytes
    /* +0x00 */ float     score[8];            // [H] live per-direction scores, dir 0..7
    /* +0x20 */ float     bias[8];             // [H] persistent per-direction biases (§5)
    /* +0x40 */ float     anchor[3];           // [H] the world position the evaluation is pinned to
    /* +0x4c */ uint32_t  cooldown_ms;         // [H] mission clock; the tick gate (+1000 ms)
    /* +0x50 */ uint32_t  permit_mask;         // [H] 1 bit per direction, from the space probe
    /* +0x54 */ float     probe_threshold;     // [H] the probe's out-float: mean clearance - 0.25
    /* +0x58 */ uint32_t  chosen_dir;          // [H] the direction taken at the last scoring tick; 0x29a = none
} ai_space_coverage;
```

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | score | H | `ai_space_coverage_compute` (= 0), `ai_space_coverage_tick` | `ai_space_coverage_tick`, `ai_space_coverage_get_look_dir` | eight floats; the tick adds `bias + 0.1` to every eligible direction except the one being taken, with a further `−0.05` on its two neighbours |
| +0x20 | bias | H | `ai_space_coverage_compute` (= 0), `ai_space_coverage_prefer_dir` | `ai_space_coverage_tick`, `ai_space_coverage_prefer_dir` (when accumulating) | eight floats; a bias `>= 0.1` makes its direction eligible even when the permit mask excludes it |
| +0x40 | anchor | H | `ai_space_coverage_compute` | `ai_space_coverage_tick` (x and z only) | the tick runs only while the character is within `0.5` squared units of it, measured on the horizontal plane |
| +0x4c | cooldown_ms | H | `ai_space_coverage_compute` (= mission clock), `ai_space_coverage_tick` (= mission clock + 1000) | `ai_space_coverage_tick` | the tick scores at most once per 1000 ms |
| +0x50 | permit_mask | H | `ai_space_coverage_compute` (the probe's return value) | `ai_space_coverage_tick` | bit *i* = direction *i* has above-average clearance (§2a) |
| +0x54 | probe_threshold | H | `ai_check_builder_test_space_8dir`, through the out-pointer `ai_space_coverage_compute` passes | - | written, never read by this class (§2a) |
| +0x58 | chosen_dir | H | `ai_space_coverage_compute` (= `0x29a`), `ai_space_coverage_tick` (the direction being taken) | - | written, never read by this class; `ai_space_coverage_get_look_dir` returns its winner instead of storing it |

### 2a. The space probe's out-parameter is one float

`ai_check_builder_test_space_8dir` receives `+0x54` as its third parameter. Its tail:

```c
fVar1 = local_54 * 0.125 - 0.25;                  // mean of the 8 clearances, minus 0.25
do { if (fVar1 <= local_20[i]) local_60 |= 1 << i; } while (i < 8);   // x87 fcom / test ah,0x41
if (param_3 == (float *)0x0) return local_60;     // the out-param is OPTIONAL
*param_3 = fVar1;                                 // and receives exactly ONE float
```

The 8 clearances never leave the probe — they live in `local_20[]`, a caller-invisible local. The
third parameter takes **one float**, the computed threshold, and tolerates NULL. Hence:

* the object is **`0x5c` bytes**, `probe_threshold` at `+0x54` is one float, and `+0x58` is the
  direction field;
* the return value is the **permit mask**: bit *i* is set iff direction *i*'s clearance is at
  least the 8-direction mean minus `0.25` — **"directions with above-average clearance"**, slot `i` ↔
  bit `i` as `ai_space_coverage_tick` assumes.

The out-parameter is decided by these four lines at the *end* of a 1280-byte function, not by its
loop.

## 3. Method family

Coverage: all **4** PS2 `17ai_space_coverage` methods are in the tables below, **4 CONFIRMED
twins**, every row body-diffed on both builds; the class has no PC-only function.

### 3a. Construction & destruction

The class has no constructor or destructor; `ai_space_coverage_compute` (§3c) initialises every
field it reads.

### 3b. Accessors

None.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00439f70 | `ai_space_coverage_compute` | `Compute__17ai_space_coverageFR8S_vector` | 0x00349260 | CONFIRMED | anchors the evaluation, runs the space probe, zeroes both arrays. Shared **10.0f** radius, **0x29a** sentinel, clock field, interleaved zeroing loop ¹ |
| 0x00439fe0 | `ai_space_coverage_tick` | `Tick__17ai_space_coverageFR8S_vectorR8S_vector` | 0x00349330 | CONFIRMED | the per-frame drift (§5). Shared **0.5** / **0.1** / **−0.05** ×2 / **1000**; identical `&7` wraparound and trailing writes |
| 0x0043a0d0 | `ai_space_coverage_get_look_dir` | `GetLook__17ai_space_coverageFR8S_vector` | 0x003494c0 | CONFIRMED | reads out the winner. Same scan (seed −1.0, strict `>`, lowest index wins ties) and table lookup; writes the unit vector from `g_dir8_table_normalized_006551a8` and returns the index |
| 0x0043a120 | `ai_space_coverage_prefer_dir` | `Preffer__17ai_space_coverageFR8S_vectorfb` | 0x00349520 | CONFIRMED | writes a bias, or adds to it when the bool is set. The `(vec3&, float, bool)` push order identifies the class ² |

¹ Size ratios for all four rows are in §6.
² At the `ai_space_coverage_prefer_dir` callsite the argument push order is `push 1` (a bool),
then a float, then a `lea` of a vec3 — `(S_vector&, float, bool)`, which matches `FR8S_vectorfb`
and no other method in the family. The enclosing caller writes `0x24` as the action type, the
`_hostile_coverage` action. The class is pinned by argument shape where the receiver load is
ambiguous.

### 3d. Free helpers

None of this class's methods calls a free helper of the shared `C_entity` unit; those helpers are
listed in `structs/entity.md` §3d.

## 4. Ownership — the `_hostile_coverage` payload

The object is the first member of the **payload of a `_hostile_coverage` action** (type `0x24`,
`structs/ai_action_manager.md`). `ai_action_new_hostile_coverage` allocates the 0x24-byte action
record and a separate 0x84-byte payload, stores the payload size `0x84` at action `+0x14` and the
payload pointer at action `+0x20`, and runs `ai_space_coverage_compute` on payload `+0x00`. Only two
functions reach the class: that factory (`compute` ×1, `prefer_dir` ×2, one bias per smelled hostile)
and `entity_action_tick` (`compute`, `tick`, `get_look_dir`, `prefer_dir` ×2), 8 callsites in all,
each with the payload in the receiver register.

The payload fields after the coverage object, as the factory seeds them:

| Payload offset | Seeded with |
|---|---|
| `+0x00..+0x5b` | the `ai_space_coverage` object |
| `+0x5c` / `+0x60` / `+0x64` | 0 |
| `+0x68` | mission clock + 500 ms |
| `+0x6c` | the factory's second argument (`uint`) |
| `+0x70` | `0x29a` (the project-wide "none") |
| `+0x74` | 4 |
| `+0x7c` | the acting person |
| `+0x80` | the `vector<C_actor*>` of hostiles |

## 5. How the scoring works

1. **`ai_space_coverage_compute(pos)`** pins the anchor, probes the 8 directions once for open
   space (radius `10.0`), and clears both arrays. The probe result becomes the **permit mask**.
2. **`ai_space_coverage_prefer_dir(dir, weight, accumulate)`** lets the caller push a thumb on the
   scale. A bias `>= 0.1` **also overrides the permit mask** in the tick — a direction the probe
   rejected can be forced back into play.
3. **`ai_space_coverage_tick(pos, current_dir)`**, per frame, but only while the character is still
   within `0.5` squared units of the anchor and only once per `1000` ms: for every eligible
   direction that is **not** the one currently taken, `score[i] += bias[i] + 0.1`, with a further
   `−0.05` if `i` is one of the two neighbours of the taken direction.
4. **`ai_space_coverage_get_look_dir(out)`** returns the highest-scoring direction as a unit vector.

**The gameplay reading:** this is what makes a hostile NPC taking cover, or circling, not simply
oscillate. Standing still and facing one way steadily raises the appeal of every other open
direction, with the two nearest directions raised least — so when the character does move, it
tends to move *decisively* rather than by one step.

⚠️ **Default answer:** a freshly computed object that has never been ticked or biased has all
scores at 0 against the `−1.0` seed, so `ai_space_coverage_get_look_dir` returns **direction 0** —
the `+Z` axis.

## 6. PC vs PS2

- **Field offsets differ by band**, because the PS2 anchor is a 4-float `S_vector` where PC uses
  3 floats:

  | Field | PC | PS2 |
  |---|---|---|
  | anchor position | `+0x40` (3 floats) | `+0x00` (4 floats) |
  | scores | `+0x00` | `+0x10` |
  | biases | `+0x20` | `+0x30` |
  | cooldown / clock | `+0x4c` | `+0x50` |
  | permit mask | `+0x50` | `+0x54` |
  | probe out-param | `+0x54` | `+0x58` |
  | direction field | `+0x58` | `+0x5c` |

  ⚠️ The PS2 build puts the anchor **first** and the arrays after; PC puts the arrays first. This
  is a REORDER, not a rebase — do not translate a single offset across builds by adding a delta.
  The PS2 shows the same probe-parameter layout at `+0x58`/`+0x5c`.
- **Every timing and scoring constant is identical**: `10.0` probe radius, `0.5` anchor
  proximity, `0.1` eligibility gate and increment, `−0.05` neighbour discount, `1000` ms
  cooldown, `0x29a` sentinel. The shipped behaviour is the same on both platforms.
- 🔑 **PS2 `Compute` opens with a call to `GetFPS__13ai_frameraterFv` and, below 2 FPS, skips the
  space probe entirely — writing `0xffffffff` to the permit mask instead.** The PC build has no
  such call, no `ai_framerater` class, and no equivalent concept anywhere in the body. This is
  behavioural evidence that `ai_framerater` is a PS2-port-only frame-budgeting layer: the PS2
  build measurably degrades AI quality under load, and the PC build has no code path that can.
- **Size ratios (PC/PS2), all below 1.0:**

  | PC name | Bytes | Ratio |
  |---|---|---|
  | `ai_space_coverage_compute` | 112/204 | 0.55 |
  | `ai_space_coverage_tick` | 240/400 | 0.60 |
  | `ai_space_coverage_get_look_dir` | 80/92 | 0.87 |
  | `ai_space_coverage_prefer_dir` | 48/116 | 0.41 (five-statement body) |

  Two causes: PS2 writes 4-float vectors where PC writes 3, and MIPS needs more instructions per
  float compare/branch than x87 does. It is not a factoring difference — the call structure
  matches one-for-one.
- **`ai_space_coverage_get_look_dir` reads a SECOND COPY of the normalised direction table** at
  PC `0x006551a8` where PS2 has the single `Dir8TableN` (`structs/ai_check.md` §6a).
- **No PS2-only methods** and no PC-only functions in the cluster.
- **The owning payload** is 0x90 bytes on PS2 (`NewHostileCoverage` allocates `__nw__FUi(0x90)`)
  against the PC's 0x84, the usual 4-float `S_vector` padding.
- **Naming.** `GetLook` returns a direction, so the PC name keeps the PS2 verb and adds `_dir`.
  `Preffer` is a PS2 misspelling of "prefer", rendered `prefer_dir`. The sibling
  `PrefferMode__14ai_fight_modesFUifb` — same verb, same `(id, float, bool)` shape — is rendered
  `ai_fight_modes_set_mode_score`, so the two renderings of the same verb differ.

## 7. Related

- `structs/ai_check.md` — `ai_check_builder`, the space probe this class delegates to, and §3a
  for the duplicated direction table.
- `structs/ai_action_manager.md` — action type `0x24` `_hostile_coverage`, the consumer.
- `structs/entity.md` §8 — the shared translation unit.

## 8. Open items

None.
