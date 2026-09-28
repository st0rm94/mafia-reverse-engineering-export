# `ai_space_coverage` — the 8-direction "where should I go next" scorer

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-09-28 19:37 +0200

Static analysis of `Game-ghidra.exe.annotated.c`, cross-checked against the non-stripped PS2 ELF
(`SLUS_206.71`). This documents the PC build; cross-build material is confined to §5.

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
  g_dir8_table_normalized                                     structs/ai_check.md §3
```

| | |
|---|---|
| PC prefix | `ai_space_coverage_*` |
| PS2 methods | 4 — **4/4 mapped**, every row body-diffed (§2) |
| PC cluster | `[0x00439f70, 0x0043a150)` — four consecutive functions, contiguous |
| Singleton | none — the object is caller-owned and reached through a register at every callsite (§1b) |

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Object layout (PC offsets)

```c
typedef struct ai_space_coverage {        /* 0x5c bytes */
    /* +0x00 */ float     score[8];       // [H] live per-direction scores, dir 0..7
    /* +0x20 */ float     bias[8];        // [H] persistent per-direction biases (see §3)
    /* +0x40 */ float     anchor[3];      // [H] the world position the evaluation is pinned to
    /* +0x4c */ uint32_t  cooldown_ms;    // [H] mission clock; also the tick gate (+1000 ms)
    /* +0x50 */ uint32_t  permit_mask;    // [H] 1 bit per direction, from the space probe
    /* +0x54 */ float     probe_threshold;// [H] the probe's optional out-float: the mean of the
                                          //     8 clearances minus 0.25 (see §1a)
    /* +0x58 */ uint32_t  chosen_dir;     // [H] 0..7, or 0x29a = none/unset
} ai_space_coverage;
```

### 1a. The space probe's out-parameter is one float

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
  chosen-direction sentinel;
* the return value is the **permit mask**: bit *i* is set iff direction *i*'s clearance is at
  least the 8-direction mean minus `0.25` — **"directions with above-average clearance"**, slot `i` ↔
  bit `i` as `tick` assumes.

The out-parameter is decided by these four lines at the *end* of a 1280-byte function, not by its
loop.

### 1b. Ownership [U]

At all 8 callsites the receiver is loaded as `mov ecx, <reg>`, so the callsite alone does not name
an owner. The disassembly at the `ai_space_coverage_compute` callsite in the `0x00430xxx` caller
shows that the receiver register also carries fields at `+0x68`, `+0x6c`, `+0x70` (another
`0x29a` sentinel) and `+0x74`, and that the enclosing action object stores it at `action+0x20`. So
the coverage object is **embedded at offset 0 of a larger object** that the action holds a pointer
to. Because this object ends at `0x5c`, `+0x68`…`+0x74` are fields of that larger object. The
enclosing type is unidentified (§4).

## 2. Method family — PC ↔ PS2 (4 of 4)

Ordered by **PC VA**, PC name first. **Every row is body-diffed on both builds.**

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| 0x00439f70 | `ai_space_coverage_compute` | `Compute__17ai_space_coverageFR8S_vector` | CONFIRMED | Anchors the evaluation, runs the space probe, zeroes both arrays. Shared **10.0f** radius, **0x29a** sentinel, clock field, interleaved zeroing loop ¹ |
| 0x00439fe0 | `ai_space_coverage_tick` | `Tick__17ai_space_coverageFR8S_vectorR8S_vector` | CONFIRMED | The per-frame drift, §3. Shared **0.5** / **0.1** / **−0.05** ×2 / **1000**; identical `&7` wraparound and trailing writes |
| 0x0043a0d0 | `ai_space_coverage_get_look_dir` | `GetLook__17ai_space_coverageFR8S_vector` | CONFIRMED | Reads out the winner. Same scan (seed −1.0, strict `>`, lowest index wins ties) and table lookup |
| 0x0043a120 | `ai_space_coverage_prefer_dir` | `Preffer__17ai_space_coverageFR8S_vectorfb` | CONFIRMED | Writes/accumulates a bias. The `(vec3&, float, bool)` push order identifies the class ² |

¹ Size ratios for all four rows are in §5.
² At the `prefer_dir` callsite the argument push order is `push 1` (a bool), then a float, then a
`lea` of a vec3 — `(S_vector&, float, bool)`, which matches `FR8S_vectorfb` and no other method in
the family. The enclosing caller writes `0x24` as the action type, the `_hostile_coverage` action.
The class is pinned by argument shape where the receiver load is ambiguous.

## 3. How the scoring works

1. **`compute(pos)`** pins the anchor, probes the 8 directions once for open space (radius
   `10.0`), and clears both arrays. The probe result becomes the **permit mask**.
2. **`prefer_dir(dir, weight, accumulate)`** lets the caller push a thumb on the scale. A bias
   `>= 0.1` **also overrides the permit mask** — a direction the probe rejected can be forced
   back into play.
3. **`tick(pos, current_dir)`**, per frame, but only while the character is still within `0.5`
   squared units of the anchor and only once per `1000` ms:
   for every eligible direction that is **not** the one currently taken,
   `score[i] += bias[i] + 0.1`, with a further `−0.05` if `i` is one of the two neighbours of the
   taken direction.
4. **`get_look_dir(out)`** returns the highest-scoring direction as a unit vector.

**The gameplay reading:** this is what makes a hostile NPC taking cover, or circling, not simply
oscillate. Standing still and facing one way steadily raises the appeal of every other open
direction, with the two nearest directions raised least — so when the character does move, it
tends to move *decisively* rather than by one step.

⚠️ **Default answer:** a freshly computed object that has never been ticked or biased has all
scores at 0 against the `−1.0` seed, so `get_look_dir` returns **direction 0** — the `+Z` axis.

## 5. PC vs PS2

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
  | chosen direction | `+0x58` | `+0x5c` |

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

  | PC name | bytes | ratio |
  |---|---|---|
  | `ai_space_coverage_compute` | 112/204 | 0.55 |
  | `ai_space_coverage_tick` | 240/400 | 0.60 |
  | `ai_space_coverage_get_look_dir` | 80/92 | 0.87 |
  | `ai_space_coverage_prefer_dir` | 48/116 | 0.41 (five-statement body) |

  Two causes: PS2 writes 4-float vectors where PC writes 3, and MIPS needs more instructions per
  float compare/branch than x87 does. It is not a factoring difference — the call structure
  matches one-for-one.
- **`get_look_dir` reads a SECOND COPY of the normalised direction table** at PC `0x006551a8`
  where PS2 has the single `Dir8TableN`. See `structs/ai_check.md` §3a.
- **No PS2-only methods** and no PC-only functions in the cluster.
- **Naming.** `GetLook` returns a direction, so the PC name keeps the PS2 verb and adds `_dir`.
  `Preffer` is a PS2 misspelling of "prefer", rendered `prefer_dir`. The sibling
  `PrefferMode__14ai_fight_modesFUifb` — same verb, same `(id, float, bool)` shape — is rendered
  `ai_fight_modes_set_mode_score`, so the two renderings of the same verb differ.

## 6. Cross-references

- `structs/ai_check.md` — `ai_check_builder`, the space probe this class delegates to, and §3a
  for the duplicated direction table.
- `structs/ai_action_manager.md` — action type `0x24` `_hostile_coverage`, the consumer.

## 7. Open items

| item | state |
|---|---|
| what owns the object (§1b) | the larger structure carrying `+0x68`..`+0x74` is unidentified; the functions containing the two `compute` callsites (`0x00422989`, `0x00430abe`) are the way in |
