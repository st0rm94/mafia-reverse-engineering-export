# `ai_fight_modes` — the NPC combat-mode scorer

**Created** 2026-07-16 20:30 +0200 · **Modified** 2026-09-30 23:08 +0200

| | |
|---|---|
| PC prefix | **`ai_fight_modes_*`** |
| Lives at | **`C_entity + 0xf64`**, embedded (§1a) |
| PC instance size | at least `0x150` (owner backpointer at `+0x14c`) |
| PC method count | **13**, every one body-diffed against the PS2 build |
| PS2 class | `ai_fight_modes` (class tag `14ai_fight_modes`), 13 methods, 13 mapped |

## 1. What it is

`ai_fight_modes` answers *"how should I fight right now?"*. It is a **6-slot table of fight modes**,
stride `0x34`, each slot carrying a set of designer constants, a running total, an accumulated hit
statistic, an externally seeded preference score and a finish-request byte.

The class is split in two halves. `ai_fight_modes_tick` **decides**: it refreshes one slot per visit,
recomputes each mode's total and commits the highest-scoring index to `+0x138`.
`ai_fight_modes_enact_active_mode` **acts**: it reads that index and installs the matching
`ai_action_*` on the owner. Damage events feed back in through `ai_fight_modes_apply_hit`, so a
tactic that keeps getting the NPC hurt bleeds score and the scorer drifts off it. The whole table is
reset whenever the enemy changes.

**Confidence tags:** **[H]** body-read, with writer and reader · **[M]** inferred · **[U]** not
observed.

### 1a. Where it lives — `C_entity + 0xf64`

The owner offset is byte-proven two independent ways by reading the ECX receiver expression at each
callsite in the disassembly:

1. **`lea ecx,[esi+0xf64]`** immediately before the call at four separate callsites — three of
   `ai_fight_modes_set_enemy` and one of `ai_fight_modes_reset`. A `lea` means the sub-object is
   **embedded**, not reached through a pointer field.
2. The `ai_fight_modes_apply_hit` callsite inside `ai_logs_event` builds the receiver in two steps:
   `mov ecx,[edi+0x30]` (the `ai_logs` owner backpointer, i.e. the `C_entity`) then `add ecx,0xf64`.
   That reaches the same object from a different base, through a field whose meaning is already
   independently documented.

Corroboration: a one-line PC accessor returns `this+0xf64` (the PS2 placement: §6).

The object is one of the `C_entity` AI sub-objects listed in `structs/entity.md` §6c, alongside
`ai_character` (`+0xebc`), `ai_sensors` (`+0xbb4`), `ai_logs` (`+0xe88`) and `ai_vis_logs` (`+0xb04`).

## 2. Field layout

The six mode slots start at **`+0x00`** — `ai_fight_modes_init` addresses slot *N* as
`this + N*0x34`, so the table spans `+0x00 .. +0x137` and the scalars at `+0x138` follow it exactly.

```c
typedef struct ai_fight_modes {
    struct fight_mode {                     /* slot N at this + N*0x34 */
    /* +0x00 */ float    w_in0;      // [H] weight on scoring input 0   \ set once by
    /* +0x04 */ float    w_in1;      // [H] weight on scoring input 1    | ai_fight_modes_init,
    /* +0x08 */ float    w_in2;      // [H] weight on scoring input 2    | per mode (§2a)
    /* +0x0c */ float    w_in3;      // [H] weight on scoring input 3   /
    /* +0x10 */ float    w_hit;      // [H] weight on damage credit (ai_fight_modes_apply_hit)
    /* +0x14 */ float    total;      // [H] the mode's current total; the tick picks the max
    /* +0x18 */ float    hit_stat;   // [H] accumulated damage credit/debit; subtracted from
                                     //     total; read out by ai_fight_modes_get_hit_stat
    /* +0x1c */ float    viability;  // [H] per-mode gain, refreshed by the tick (§4);
                                     //     init 1.0f, clamped >= 0; <= 0.0001 zeroes the total
    /* +0x20 */ uint32_t last_used;  // [H] mission clock when the mode stopped being active
    /* +0x24 */ float    pref_score; // [H] external preference, ai_fight_modes_set_mode_score
    /* +0x28 */ uint32_t pref_at;    // [H] mission clock of that write; pref decays over 10 s
    /* +0x2c */ uint8_t  finish_req; // [H] finish REQUEST byte: set by
                                     //     ai_fight_modes_request_finish_mode, test-and-cleared
                                     //     by ai_fight_modes_consume_finish_request
    /* +0x30 */ void    *work;       // [H] slot 2 only: a 0x24-byte block from init; every other
                                     //     slot holds 0. Freed by ai_fight_modes_destroy (+0x98)
    } modes[6];                             /* +0x00 .. +0x137 */
    /* +0x138 */ uint32_t active_mode;  // [H] the mode the NPC is in; 5 = none
    /* +0x13c */ uint32_t prev_mode;    // [H] the previous active_mode; 5 = none
    /* +0x140 */ uint32_t cursor;       // [H] round-robin refresh cursor, advanced (n+1) % 6;
                                        //     reset to 5, so the first refresh lands on mode 0
    /* +0x144 */ uint32_t next_update;  // [H] mission clock deadline; the tick refreshes one
                                        //     slot when it expires and re-arms it at +500 ms.
                                        //     0 = refresh all six now (§4)
    /* +0x148 */ C_actor *enemy;        // [H] current enemy; ai_fight_modes_set_enemy
    /* +0x14c */ C_entity *owner;       // [H] owner backpointer, stored by init
} ai_fight_modes;                       /* size >= 0x150 */
```

**`5` is the "no mode" sentinel** — `ai_fight_modes_finish_mode` writes 5 into `+0x13c`, and
`ai_fight_modes_reset` restores all three of `+0x138`/`+0x13c`/`+0x140` to 5. (It is a local
sentinel, unrelated to the `666` "none" value other subsystems use.)

### 2a. The per-mode constants

`ai_fight_modes_init` writes five floats into every slot — the four input weights and the damage
weight — plus `viability = 1.0f`, and zeroes the rest of the slot:

| mode | `+0x00` | `+0x04` | `+0x08` | `+0x0c` | `+0x10` |
|---:|---:|---:|---:|---:|---:|
| 0 | 0.5 | 0.1 | 0.5 | 0.1 | −0.5 |
| 1 | 0.35 | 0.35 | 0.5 | 0.75 | −0.5 |
| 2 | 0.4 | 0.9 | 0.9 | 1.0 | −0.45 |
| 3 | −0.5 | −1.0 | 0.5 | 1.0 | −0.5 |
| 4 | 0.0 | 0.0 | −100.0 | −1000.0 | 0.0 |
| 5 | 0.1 | 0.1 | 0.1 | 0.0 | 0.0 |

Mode 2 is also the only slot that gets a heap block: `operator_new(0x24)` stored at slot `+0x30`,
absolute `+0x98`, which the tick hands to `ai_wayfind_find_nearest_hide_corner` — so mode 2 is the
cover-seeking mode. The block is an **`ai_corner_hide_info`** (`structs/check_seg.md` §1g: corner,
approach position, facing, side, distance), and the tick reads its `+0x20` distance.

**The four scoring inputs** are built by `entity_ai`, the tick's only caller (3 callsites): `{anger
(log+0x4c), fear (log+0x50), threat (log+0x48), intelligence}`, from the `ai_logs` record of the
current enemy plus `ai_character_get_intelligence` (`structs/ai_logs.md`,
`structs/ai_character.md`). So the columns of the table above weight anger, fear, threat and
intelligence in that order.

**What the modes are**, from `ai_fight_modes_enact_active_mode`'s switch:

| mode | action installed |
|---:|---|
| 0 | normal fight (type `0x1e`), aggression 2.0 unarmed / 5.0 armed, plus a car-jump modifier |
| 1 | crouch fight (type `0x1d`), `15.0`, with `8` when the enemy is in a car, plus a car-jump modifier |
| 2 | corner fight (type `0x1c`) on the slot's `ai_corner_hide_info`, plus a car-jump modifier |
| 3, 4 | no arm — nothing is installed, and their totals are always 0 on the PC (§4) |
| 5 | no action: clears the job queue and every mode's hit statistic, re-seeds the preferences (mode 1 = 10.0, mode 2 = −10.0) and writes the owner's ANGER ramp (`C_entity+0xf1c` = 2.0, rate −0.0001/ms, for 10 000 ms; `structs/entity.md`) |

## 3. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| slot `+0x00`..`+0x0c` | input weights | H | `ai_fight_modes_init` | `ai_fight_modes_tick` | multiply the four floats of the tick's input block |
| slot `+0x10` | hit weight | H | `ai_fight_modes_init` | `ai_fight_modes_apply_hit` | scales the damage credit |
| slot `+0x14` | total | H | `ai_fight_modes_tick` | `ai_fight_modes_tick` | the value the winner scan compares |
| slot `+0x18` | hit_stat | H | `ai_fight_modes_apply_hit` | `ai_fight_modes_tick`, `ai_fight_modes_get_hit_stat` | subtracted from the total, so damage taken lowers it |
| slot `+0x1c` | viability | H | `ai_fight_modes_init`, `ai_fight_modes_tick` | `ai_fight_modes_tick` | zero ⇒ the mode's total collapses to 0 |
| slot `+0x20` | last_used | H | `ai_fight_modes_tick` | `ai_fight_modes_tick` | drives the 0..1 "time since last used" ramp |
| slot `+0x24` | pref_score | H | `ai_fight_modes_set_mode_score` | `ai_fight_modes_tick` | `ai_fight_modes_set_enemy` seeds mode 2 = **5.0f**, mode 1 = **1.5f** |
| slot `+0x28` | pref_at | H | `ai_fight_modes_set_mode_score` | `ai_fight_modes_tick` | mission clock; the preference fades to 0 over 10 000 ms |
| slot `+0x2c` | finish_req | H | `ai_fight_modes_request_finish_mode` | `ai_fight_modes_consume_finish_request` | a request/consume pair on one byte |
| slot `+0x30` | work | H | `ai_fight_modes_init` (slot 2 only) | `ai_fight_modes_tick` (mode 2 arm) | freed by `ai_fight_modes_destroy` |
| `+0x138` | active_mode | H | `ai_fight_modes_tick`, `ai_fight_modes_force_mode`, `ai_fight_modes_reset` (→ 5) | `ai_fight_modes_apply_hit`, `ai_fight_modes_enact_active_mode` | the index everything else calls "the mode" |
| `+0x13c` | prev_mode | H | `ai_fight_modes_tick`, `ai_fight_modes_force_mode`, `ai_fight_modes_finish_mode` (→ 5), `ai_fight_modes_reset` (→ 5) | `ai_fight_modes_tick` | differs from `+0x138` ⇒ a transition happened |
| `+0x140` | cursor | H | `ai_fight_modes_tick`, `ai_fight_modes_reset` (→ 5) | `ai_fight_modes_tick` | selects the one slot refreshed this visit |
| `+0x144` | next_update | H | `ai_fight_modes_tick` (→ clock + 500), `ai_fight_modes_set_mode_score`, `ai_fight_modes_finish_mode`, `ai_fight_modes_reset` (all → 0) | `ai_fight_modes_tick` | 0 forces a full six-slot sweep |
| `+0x148` | enemy | H | `ai_fight_modes_set_enemy`, `ai_fight_modes_reset` (→ 0) | `ai_fight_modes_set_enemy` (the change guard), `ai_fight_modes_tick`, `ai_fight_modes_enact_active_mode` | a `C_actor *` |
| `+0x14c` | owner | H | `ai_fight_modes_init` | `ai_fight_modes_tick`, `ai_fight_modes_enact_active_mode` | the `C_entity` this is embedded in |

## 4. The decide / act split

**`ai_fight_modes_tick` decides.** Unarmed owner (`entity_weapon_is_armed` is its first call) ⇒ it
shifts `+0x138` into `+0x13c`, forces mode **0** and returns. Otherwise:

1. **Refresh.** While `next_update` has expired, advance `cursor = (cursor + 1) % 6`, re-arm
   `next_update = clock + 500` and rewrite that one mode's `viability`. When `next_update` is 0 the
   loop runs six times instead of once, refreshing the whole table immediately.

   | mode | what the refresh arm writes into `viability` |
   |---:|---|
   | 0 | `1.0` |
   | 1 | weapon readiness + 1.0 |
   | 2 | 0 when the enemy is within 4 m; otherwise, if `ai_wayfind_find_nearest_hide_corner` finds a corner for the slot's work block, `(10.0 − corner distance) * 0.1`, scaled by readiness + 1.0 and clamped at 0 |
   | 3 | `0` |
   | 4 | `0` |
   | 5 | `0.1` |

2. **Score the refreshed mode.** `viability *= 1.0 + pref_score * fade`, where `fade` is
   `1 − (clock − pref_at) * 0.0001` clamped to 0..1 — a seeded preference that decays over 10 s.
   With `viability <= 0.0001` the total is set to 0; otherwise the total is built from the four
   input floats through the slot's weights, scaled by `viability` and by a second 0..1 ramp
   `(clock − last_used) * 0.0001` (the longer since the mode was used, the higher it ramps).
   Finally `total -= hit_stat`, clamped at 0.
3. **Commit.** Scan the six totals from a **−1000.0f** seed, take the highest, stamp `last_used` on
   the outgoing mode when `+0x13c != +0x138`, then shift `+0x138` into `+0x13c` and write the
   winner into `+0x138`.

Modes **3 and 4 can never win in the PC build**: their refresh arms write `viability = 0` and the
scoring step is multiplicative, so their totals stay 0.

**`ai_fight_modes_enact_active_mode` acts.** It reads the owner (`+0x14c`) and the enemy (`+0x148`),
no-ops without an enemy, and switches on `+0x138`. Each arm first checks whether the pending task is
already the right one for this enemy, so it is idempotent per frame; otherwise it builds and installs
the matching `ai_action_*`. Mode 0 installs a normal-fight action with aggression **2.0f unarmed /
5.0f armed** as the sole task, plus a car-jump modifier.

## 5. Method family — PC ↔ PS2

All 13 methods are **CONFIRMED**: each was body-diffed on both builds, and there are no CANDIDATE
rows. Rows are ordered by PC VA, which is also PS2 emission order.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | What it does |
|---|---|---|---|---|
| `0x00439320` | `ai_fight_modes_init` | `Init__14ai_fight_modesFPv` | `0x00347da0` | Stores the owner at `+0x14c`, calls `ai_fight_modes_reset`, then walks the six slots writing the per-mode constants (§2a), `viability = 1.0f` and zeroes; allocates mode 2's `0x24`-byte work block. Identical first two statements, same loop, same seven writes in order, same per-slot constants |
| `0x00439460` | `ai_fight_modes_destroy` | `Destroy__14ai_fight_modesFv` | `0x00347f60` | Frees the one heap pointer at `+0x98` and nulls it; nothing else needs teardown. Identical null-check / free / null-out on the same offset; PC `operator_delete_wrapper` ↔ PS2 `__dl__FPv` |
| `0x00439490` | `ai_fight_modes_set_enemy` | `SetEnemy__14ai_fight_modesFP7C_actor` | `0x00347fa0` | On an enemy CHANGE (`param != +0x148`): reset the table, store the enemy, seed mode **2 = 5.0f** and mode **1 = 1.5f**. **Byte-identical**, shared magic `0x40a00000` / `0x3fc00000` with the same mode arguments |
| `0x004394d0` | `ai_fight_modes_reset` | `Invalidate__14ai_fight_modesFv` | `0x00348010` | Zeroes each slot's `+0x14`/`+0x18`/`+0x1c`, clears the enemy, restores `+0x138`/`+0x13c`/`+0x140` to **5** and `+0x144` to 0. PS2 unrolls the six slots, PC keeps a loop at stride `0x34`; same offsets |
| `0x00439510` | `ai_fight_modes_tick` | `Tick__14ai_fight_modesFPf` | `0x00348080` | The scorer (§4). Shared magic −1000.0f; `entity_weapon_is_armed` ↔ `WeaponIsArmed__8C_entity` as the first call gating the same path; same `+0x13c = +0x138` shift; owner via `+0x14c` |
| `0x00439980` | `ai_fight_modes_apply_hit` | `Hit__14ai_fight_modesFfb` | `0x003487f0` | Credits or debits the active mode (`+0x138`): `slot+0x18 ±= damage * slot+0x10 * 0.01`. **Byte-identical.** Polarity is the caller's: `ai_logs_event` passes 1 (debit) when the owner was hit, 0 (credit) when the owner's enemy was hit |
| `0x004399c0` | `ai_fight_modes_enact_active_mode` | `Select__14ai_fight_modesFv` | `0x003488a0` | The half that acts (§4). Same guard, same switch, same magic pair `0x40000000` / `0x40a00000` on the same armed test; exact callee correspondence for all four action-manager calls. Named for its behaviour rather than after the PS2 `Select` — the tick chooses, this one enacts |
| `0x00439bf0` | `ai_fight_modes_get_hit_stat` | `GetHitStat__14ai_fight_modesFUi` | `0x00348be0` | **Byte-identical**: `return *(this + 0x18 + N*0x34)` |
| `0x00439c10` | `ai_fight_modes_set_mode_score` | `PrefferMode__14ai_fight_modesFUifb` | `0x00348c10` | Stamps mode N: `+0x28` = mission clock, `+0x24` = score, `+0x144` = 0. Same three writes in the same order and the same mission-clock source; **build difference in §6** |
| `0x00439c50` | `ai_fight_modes_finish_mode` | `FinishMode__14ai_fight_modesFv` | `0x00348cc0` | **Byte-identical**: `*(this+0x13c) = 5; *(this+0x144) = 0` — drop the previous mode and force an immediate full rescore |
| `0x00439c70` | `ai_fight_modes_request_finish_mode` | `RequestFinishMode__14ai_fight_modesFUi` | `0x00348cd0` | **Byte-identical**: `*(byte*)(this + 0x2c + N*0x34) = 1`. The request half |
| `0x00439c90` | `ai_fight_modes_consume_finish_request` | `ModeMustFinish__14ai_fight_modesFUi` | `0x00348d00` | **Byte-identical**: test the byte, clear it if set, return whether it was. The consume half |
| `0x00439cb0` | `ai_fight_modes_force_mode` | `ForceMode__14ai_fight_modesFUiP7C_actor` | `0x00348d40` | Override the scorer: adopt the enemy through `ai_fight_modes_set_enemy`, then pin the mode by writing it to **both** `+0x138` and `+0x13c`, so the next tick sees no transition. Statement-for-statement identical |

## 6. PC vs PS2

* **Zero field delta inside the class.** Every offset above is identical on both builds — `+0x18`,
  `+0x24`, `+0x28`, `+0x2c`, the `0x34` stride, `+0x138`/`+0x13c`/`+0x140`, `+0x144`, `+0x148`. That
  is unusual here (the `C_entity` bands drift by `0x14c` / `0x1d4` / `0x1e4` / `0x1e8`), and seven
  independent body-diffs all landing on identical offsets is itself corroboration.
* **The owner offset does differ**, like the rest of the `C_entity` bands: PC `+0xf64` ↔ PS2
  `+0x114c`, delta `0x1e8` — exactly the `C_entity` band delta that `ai_logs_compute_attitude`'s
  three tuning fields confirm independently (PC `+0xf34`/`+0xf1c`/`+0xf28` ↔ PS2
  `+0x111c`/`+0x1104`/`+0x1110`).
* **PS2 guards the score write.** `PrefferMode__14ai_fight_modes` takes a 4th `bool` parameter the PC
  signature lacks and only stamps a mode when
  `(bool_param == 0) || (0.0 <= current_score) || (mission_clock − pref_at > 9999)`.
  `ai_fight_modes_set_mode_score` writes unconditionally. Predicted consequence: on PS2 a
  freshly-written **negative** mode score is sticky for ~10 s and cannot be re-seeded, while on PC it
  can be overwritten immediately — and since `ai_fight_modes_set_enemy` seeds modes 2 and 1 on
  every enemy change, PS2 NPCs may be slower to re-weight their fight mode when the enemy switches
  rapidly.
* **The tick paces itself differently.** PS2 consults `GetFPS__13ai_framerater` to degrade its update
  rate; PC uses the `+0x144` deadline with its fixed 500 ms cadence. That corroborates
  `ai_framerater` being PS2-port-only.
* **Sibling AI classes show the same kind of behavioural delta**: `ai_sensors_can_hear`'s staleness
  window is PC 4000 ms / PS2 6000 ms, and the visual-contact scan bound is PC 8 slots / PS2 6.
* **`C_entity`'s trailing PC↔PS2 gap contains this class on the PC side**, not `C_entity` methods.
  The size ratios across that gap (×7.5 / ×9.5 / ×22 / ×49) are the signature of a foreign class, not
  of shifted ranks, so rank-matching across it cannot finish `C_entity`'s remaining methods.
* **The PS2 name `PrefferMode` is a developer typo for "prefer"** and is not propagated into the PC
  name: "prefer" would imply the guard the PC build does not have.

## 7. Related

* `structs/entity.md` §6c — the `C_entity` AI sub-object map, including `ai_fight_modes` at `+0xf64`.
* `structs/ai_logs.md` — `ai_logs_event` calls `ai_fight_modes_apply_hit` directly, and the `ai_logs`
  owner backpointer is one of the two paths that byte-prove `+0xf64`.
* `structs/ai_action_manager.md` — the action queue `ai_fight_modes_enact_active_mode` installs into,
  embedded at `C_entity+0x1164`.
* `structs/ai_space_coverage.md` — the four functions at `0x00439f70`..`0x0043a150`, in the same PC
  region after `ai_talkman`, belong to `ai_space_coverage`, not to this class.

## 8. Open items

| item | state |
|---|---|
| the PS2 sticky-score prediction (§6) | 🎮 **IN GAME** (PS2) — derived from the guard in `PrefferMode__14ai_fight_modes`; not observed in play |
