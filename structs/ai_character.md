# `ai_character` — the NPC personality / stat block

**Created** 2026-07-16 10:28 +0200 · **Modified** 2026-09-30 23:08 +0200

Static analysis of `Game-ghidra.exe.annotated.c` and `Game.exe` disassembly, cross-checked
against the non-stripped PS2 ELF (`SLUS_206.71`). This documents the PC build; the PS2 side
supplies the original developer names (§6), and every PC offset below is byte-derived from
`Game.exe`, not translated.

`ai_character` is a small **sub-object EMBEDDED in `C_entity` at `+0xebc`** holding the NPC's
personality/sense stats. It is the "how good is this NPC at X" record: each stat is
`owner_property[K] + per_instance_delta`, where the property block is the owning entity's own
(§1).

| | |
|---|---|
| PC prefix | `ai_character_` (lowercase class ⇒ lowercase prefix) |
| PC location | `C_entity + 0xebc` — byte-proven: both `C_entity` wrappers compile to `add ecx, 0xebc` before calling into this class |
| Size | `0x28` — nine stat dwords (`+0x00..+0x20`) + the config backpointer (`+0x24`) |
| PS2 class | `ai_character`, 6 methods, **6/6 located** |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

---

## 1. Field layout (PC offsets, relative to the `ai_character` base)

```c
typedef struct ai_character {           // embedded at C_entity+0xebc
    /* +0x00 */ float    stat_delta_0;      // [H] zeroed by ai_character_init; never read (§2)
    /* +0x04 */ float    intelligence_delta;// [H] per-instance delta; ai_character_get_intelligence
    /* +0x08 */ float    morale_delta;      // [H] per-instance delta; ai_character_get_morale
    /* +0x0c */ float    stat_delta_3;      // [H] zeroed by ai_character_init; never read (§2)
    /* +0x10 */ float    aggression_delta;  // [H] per-instance delta; ai_character_get_aggression
    /* +0x14 */ float    sight_delta;       // [H] per-instance delta; ai_character_get_sight
    /* +0x18 */ float    hearing_delta;     // [H] per-instance delta; ai_character_get_hearing
    /* +0x1c */ float    stat_delta_7;      // [H] zeroed by ai_character_init; never read (§2)
    /* +0x20 */ float    stat_delta_8;      // [H] zeroed by ai_character_init; never read (§2)
    /* +0x24 */ void    *cfg;               // [H] -> the OWNING C_entity (NOT serialized).
                                            //     Stored by ai_character_init(this, cfg).
} ai_character;                             // size 0x28
```

**Why `+0x24` is excluded from the savegame:** `ai_character_init` zeroes exactly the nine dwords
`+0x00..+0x20` = **0x24 bytes**, and stores `cfg` at `+0x24` afterwards. That is precisely the
constant `entity_get_character_size` returns (**0x24**) and the block `entity_save_character`
writes — the config pointer is a live pointer and is restored by re-`Init` on load, not saved.
This independently corroborates both the sub-object identification and the savegame chain
documented in `structs/entity.md` §4.

### The config record (`*(this+0x24)`) is the owning entity

Both callers of `ai_character_init` — `entity_game_init` and `entity_reset` — push the entity itself
and set ECX to `entity+0xebc`, so `cfg` is the owning `C_entity`, and each getter reads a slot of the
entity's **current-value property block** (`prop_cur`, `structs/human.md` §2b). That block is loaded
from the scene by `human_load_stats_block_plain` / `_scaled` and changed by the script's
`human_setproperty`.

| cfg offset | stat | property slot |
|---|---|---|
| `+0x660` | aggression | 8 `Aggresivity` |
| `+0x664` | intelligence | 9 `Intelligence` |
| `+0x66c` | sight | 11 `Sight` |
| `+0x670` | hearing | 12 `Hearing` |
| `+0x67c` | morale | 15 — past the 15 named slots, so no script name reaches it; its maximum-side twin `+0x63c` is the constant 0.5 |

No store with a literal displacement writes `C_entity+0x67c`.

---

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x04 | intelligence_delta | H | `ai_character_init` (zero) | `ai_character_get_intelligence` | consumer: `entity_ai` → `ai_fight_modes_tick` (§4) |
| +0x08 | morale_delta | H | `ai_character_init` (zero) | `ai_character_get_morale` | **consumer-proven** by `ai_logs_compute_attitude` (§4) |
| +0x10 | aggression_delta | H | `ai_character_init` (zero) | `ai_character_get_aggression` | **consumer-proven** by `ai_logs_compute_attitude` (§4) |
| +0x14 | sight_delta | H | `ai_character_init` (zero) | `ai_character_get_sight` | **consumer-proven**: `entity_get_sight_range` = this × 200.0, compared against inter-entity distance by `ai_vis_logs_tick` |
| +0x18 | hearing_delta | H | `ai_character_init` (zero) | `ai_character_get_hearing` | **consumer-proven**: `sound_arena_update_entity_sound_contacts` registers a sound contact iff attenuated loudness > `1.05 − this` |
| +0x00, +0x0c, +0x1c, +0x20 | 4 unused stat deltas | H | `ai_character_init` (zero), `entity_load_character` (block restore) | - | never read and never set to anything but zero: all 16 pointers the image forms to `entity+0xebc` go to `ai_character_init`, a getter, or the `entity_save_character` / `entity_load_character` block copy, and no instruction addresses the four fields from the entity base. No getter on either build |
| +0x24 | cfg | H | `ai_character_init` | every `ai_character_get_*` | the owning `C_entity`; **not serialized** (§1) |

---

## 3. Method family — PC ↔ PS2 (all 6 methods)

PC-first, sorted by VA; all six body-diffed.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004324d0 | `ai_character_init` | `Init__12ai_characterFPv` | 0x0033ead0 | CONFIRMED | **byte-identical**: zeroes 9 dwords, stores `cfg` at `[9]` = `+0x24`. Unrelated to the `ai_action_new_*` factories it sits beside |
| 0x00432500 | `ai_character_get_morale` | `GetMoralka__12ai_characterFv` | 0x0033eb00 | CONFIRMED | `cfg[0x67c] + this[+0x08]` |
| 0x00432510 | `ai_character_get_intelligence` | `GetInteligence__12ai_characterFv` | 0x0033eb20 | CONFIRMED | `cfg[0x664] + this[+0x04]` |
| 0x00432520 | `ai_character_get_aggression` | `GetAgresivita__12ai_characterFv` | 0x0033eb40 | CONFIRMED | `cfg[0x660] + this[+0x10]` |
| 0x00432530 | `ai_character_get_sight` | `GetZrak__12ai_characterFv` | 0x0033eb60 | CONFIRMED | `cfg[0x66c] + this[+0x14]` |
| 0x00432540 | `ai_character_get_hearing` | `GetSluch__12ai_characterFv` | 0x0033eb80 | CONFIRMED | `cfg[0x670] + this[+0x18]` |

**Coverage: 6/6 CONFIRMED.** The evidence is in §6.

---

## 4. Consumers

| PC consumer | uses | via |
|---|---|---|
| `entity_get_sight_range` | sight | `add ecx,0xebc` → `ai_character_get_sight` × 200.0 |
| `entity_get_hearing_sensitivity` | hearing | `add ecx,0xebc` → `ai_character_get_hearing` |
| `ai_vis_logs_tick` | sight | calls `ai_character_get_sight` directly once, plus `entity_get_sight_range` ×4 |
| `ai_logs_compute_attitude` | morale + aggression | calls `ai_character_get_morale` then `ai_character_get_aggression` on the owning `C_entity` (`ai_logs+0x30` → `+0xebc`) ¹ |
| `entity_ai` | intelligence | 3 calls; each stores the value as the 4th float of the input block passed to `ai_fight_modes_tick` ² |

¹ **morale** picks the attitude-shaping branch (`<0.5` amplifies a negative bias, `≥0.5` damps it)
and scales **FEAR** as `fear = (1 − morale) · (...) · 2` — low morale ⇒ more fear. **aggression**
scales **ANGER** as `anger = (aggression + 0.5) · (...)` — high aggression ⇒ more anger. That the
two feed different outputs, each with the expected polarity, corroborates both glosses. See
`structs/ai_logs.md` §3/§4.

² The block is `{anger (log+0x4c), fear (log+0x50), threat (log+0x48), intelligence}`, built from
the `ai_logs` record of the current enemy. `ai_fight_modes_tick` weights intelligence by each mode's
`+0x0c` constant: 0.1 for the normal fight, 0.75 for the crouch fight, 1.0 for the corner (cover)
fight, 0 for the reset mode. A higher intelligence therefore shifts the NPC from the normal fight
towards crouch and cover fighting (`structs/ai_fight_modes.md` §2a, §4).

---

## 5. Cross-references

- `structs/entity.md` §4 — the `C_entity` field band; `ai_character` is the "character" savegame
  sub-block at `+0xebc`.
- `structs/ai_sensors.md` — the sibling three-sense contact system at `C_entity+0xbb4` (the
  hearing stat feeds its sound-contact threshold).
- `structs/ai_logs.md` — `ai_logs` at `C_entity+0xe88`. `ai_character_log` (PS2 class, 7
  methods) is a different class: the per-target log the AI scores attitudes from; its PC functions
  are `ai_character_log_new_contact` and the `ai_character_log_event_*` family.

---

## 6. PC vs PS2

* **Location.** PS2 embeds the block at `C_entity + 0x1090` (`GetZrak__12ai_character(this + 0x1090)`).
* **Instance offsets are identical** on both builds (`+0x04/+0x08/+0x10/+0x14/+0x18`, cfg pointer
  at `+0x24`). Only the config-record offsets differ, by a **constant `0x64`** across all five
  getters:

| stat | PC cfg | PS2 cfg |
|---|---|---|
| aggression | `+0x660` | `+0x6c4` |
| intelligence | `+0x664` | `+0x6c8` |
| sight | `+0x66c` | `+0x6d0` |
| hearing | `+0x670` | `+0x6d4` |
| morale | `+0x67c` | `+0x6e0` |

* **Czech method names:** *moralka* = morale, *agresivita* = aggressiveness, *zrak* = sight,
  *sluch* = hearing.
* **How the family was proven:**
  1. **The lever.** PS2 `GetVisibleRange__8C_entity` = `GetZrak__12ai_character(this + 0x1090) * 200.0`.
     PC disassembly of `entity_get_sight_range`: `add ecx, 0xebc; call 0x00432530; fmul [0x0063b5f4]`
     (= 200.0).
  2. **The 5/5 offset lock.** Predicting each PC cfg offset as `PS2 − 0x64` hit 5/5.
  3. **Emission order.** The 6 PC functions and the 6 PS2 methods are in the same order, and both
     sequences end at the confirmed `WeaponChoose` / `entity_weapon_choose` anchor.
  4. **Receiver fingerprint.** `add ecx, 0xebc` in the `C_entity` wrappers = an **embedded**
     sub-object, not a pointer — matching PS2's `this + 0x1090`.
  5. **Structural corroboration:** `entity_get_character_size` == `0x24` == the nine zeroed dwords.
* `GetMoralka__12ai_character` and `GetAgresivita__12ai_character` are called in that order at the
  head of `ComputeAttitude__7ai_logs`, matching `ai_logs_compute_attitude`.

