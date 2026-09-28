# `ai_sensors` — the three-sense contact system (sight / sound / smell)

**Created** 2026-07-16 10:28 +0200 · **Modified** 2026-09-17 17:58 +0200

Static analysis of `Game-ghidra.exe.annotated.c` + `Game.exe` disassembly, cross-checked against the
non-stripped PS2 ELF (`SLUS_206.71`). This documents the PC build; the PS2 symbol table supplies
the original developer names, and the PC offsets below are byte-derived from `Game.exe`.

`ai_sensors` is a **sub-object EMBEDDED in `C_entity` at `+0xbb4`**. It is the NPC's *sensory
short-term memory*: a set of **contact** records, one per perceived actor per sense, each with a
strength/intensity that decays and expires. Three parallel senses, each with the identical
**`has` / `New` / `Update` / `Cancel` / `Update<SENSE>`** five-method shape:

| sense | contact struct | notes |
|---|---|---|
| **VISUAL** | `ai_visual_contact` | fed by `ai_vis_logs_tick` (LOS + sight range) |
| **SOUND** | `ai_sound_contact` | fed by `sound_arena_update_entity_sound_contacts` (loudness vs hearing) |
| **SMELL** | `ai_smell_contact` | fed by `ai_sensors_get_smelled_target_info` / `ai_sensors_build_smell_contact_from_blob` + `ai_blob`. Almost certainly the **`C_dog`** sense |

| | |
|---|---|
| PC prefix | `ai_sensors_*` (lowercase class ⇒ lowercase prefix) |
| PC location | `C_entity + 0xbb4` — byte-proven by `lea ecx, [ebp + 0xbb4]` at the sound-contact callsite in `sound_arena_update_entity_sound_contacts` |
| PS2 methods | 23 — **23/23 CONFIRMED**, every row body-diffed on both builds; no PS2-only members |
| PC cluster | `0x0040f760 → 0x00410630`, no foreign function inside |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Field layout (PC offsets, relative to the `ai_sensors` base)

The three contact arrays, all twelve extrema scalars (§4b) and the sound-contact record are pinned.

```c
typedef struct ai_sensors {                 // embedded at C_entity+0xbb4
    // ---- THREE PARALLEL CONTACT ARRAYS, 8 slots each (proven by ai_sensors_init/_destroy) ----
    /* +0x000 */ visual_contact visual[8];  // [H] stride 0x18. slot: +0x8 state, +0xc actor,
                                            //     +0x10 strength, +0x14 smoothed (EMA)
    /* +0x0c0 */ sound_contact  sound[8];   // [H] stride 0x20, from the disassembly (§1b). Per slot:
                                            //       +0x00 timestamp   (= C_entity+0xc74 for slot 0)
                                            //       +0x04 TYPE        (**2 = GUNFIRE**; zeroed to kill
                                            //             the slot by ai_sensors_cancel_sound_contact)
                                            //       +0x08 pos.x/y/z   (vec3)
                                            //       +0x14 source actor
                                            //       +0x18 intensity
                                            //       +0x1c smoothed    (the 3.0/0.25 EMA)
    /* +0x1c0 */ smell_contact  smell[8];   // [H] stride 0x1c. slot: +0x4 timestamp, +0x8 state,
                                            //     +0xc ai_blob* (OWNED heap ptr), +0x10 actor,
                                            //     +0x14 strength, +0x18 smoothed (EMA).
                                            //     Base pinned by `(idx+0x10)*0x1c` in new_smell_contact.
    // ---- OWNER + THE 12 EXTREMA SCALARS (all proven by ai_sensors_destroy — see §4b) ----
    /* +0x2a0 */ C_entity *owner;           // [H] owning entity; stored by ai_sensors_init
    /* +0x2a4 */ float min_value[3];        // [H] visual / sound / smell   (+0x2a4 / +0x2a8 / +0x2ac)
    /* +0x2b0 */ float max_value[3];        // [H] visual / sound / smell   (+0x2b0 / +0x2b4 / +0x2b8)
    /* +0x2bc */ uint32_t min_index[3];     // [H] visual / sound / smell   (+0x2bc / +0x2c0 / +0x2c4)
    /* +0x2c8 */ uint32_t max_index[3];     // [H] visual / sound / smell   (+0x2c8 / +0x2cc / +0x2d0)
} ai_sensors;                               // size >= 0x2d4
```

Every `ai_sensors_*` accessor reaches the NPC's stats through `owner` (+0x2a0) → the `ai_character`
at `C_entity+0xebc` (`structs/ai_character.md`). That is why the predicates can read a **sight** or
**hearing** stat without taking it as a parameter.

### 1a. How the PC base `+0xbb4` is pinned

`ai_sensors_cancel_sound_contact` writes `*(this + 0xc4 + idx*0x20) = 0`. With the byte-proven
receiver `this = C_entity + 0xbb4`, that field is `C_entity + 0xc78 + idx*0x20`.
`structs/entity.md` §4 documents the heard-event ring at **`C_entity+0xc74`, stride 0x20**, with
`entry[0]` = timestamp and `entry[1]` = **type**. `0xc74 + 4 = 0xc78` ⇒ the cancel zeroes exactly the
**type** field. The two independent derivations agree, which pins:

- **`ai_sensors` base = `C_entity + 0xbb4`**.
- **sound-contact array = `ai_sensors + 0xc0`** = `C_entity + 0xc74`.

`structs/entity.md` records "sensors @ `+0xbc0`, serialized size 0x3a0". `+0xbc0` is where
**`entity_save_sensors` begins serializing** (= `ai_sensors + 0xc`); the **object base is
`+0xbb4`**. Both are true; they are different things.

### 1b. The sound-contact record comes from the disassembly

Ghidra renders `ai_sensors_find_sound_contact`'s scan as `piVar2 = (int*)(this + 0xc4); … param_2 !=
piVar2[1]` — i.e. actor at +0xc8. The disassembly says otherwise:

```
  0040f7a3  add  ecx, 0xc4                   ; cursor = this + 0xc4
  0040f7a9  cmp  edx, dword ptr [ecx + 0x10] ; actor  = this + 0xd4 + i*0x20   (NOT +0xc8)
  0040f7ae  cmp  dword ptr [ecx], 0          ; type   = this + 0xc4 + i*0x20
  0040f7b4  add  ecx, 0x20                   ; stride 0x20, 8 slots
```

Derivations of this record from decompiler output alone contradict each other; the disassembly
settles it. The record is the one in §1.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xc0 (C_entity+0xc74) | sound_contacts[].t | H | `ai_sensors_new_sound_contact` | `entity_has_heard_gunfire` | timestamp; stride 0x20 |
| +0xc4 (C_entity+0xc78) | sound_contacts[].type | H | `ai_sensors_new_sound_contact` | `entity_has_heard_gunfire` | **TYPE 2 = GUNFIRE.** Zeroed (= slot killed) by `ai_sensors_cancel_sound_contact` |
| +0x2a4 | visual_extrema (min) | H | `ai_sensors_recompute_visual_extrema` | `ai_sensors_new_visual_contact` | new visual contact inserted only when `this[+0x2a4] < contact.strength` (§4b) |
| +0x2a8 | sound_extrema (min) | H | `ai_sensors_recompute_sound_extrema` (resets to 1.0f) | `ai_sensors_new_sound_contact` | early-out when `contact.intensity <= this[+0x2a8]`; visual+4 (§4b) |
| +0x2bc/+0x2c0/+0x2c4 | min_index[visual/sound/smell] | H | the three `ai_sensors_recompute_*_extrema` | the three `ai_sensors_new_*_contact` | also the APPEND index (`this + idx*stride`); proven ×3 by `ai_sensors_destroy` |
| +0x2b0/+0x2b4/+0x2b8 | max_value[visual/sound/smell] | H | the three `ai_sensors_recompute_*_extrema` | - | §4b |
| +0x2c8/+0x2cc/+0x2d0 | max_index[visual/sound/smell] | H | the three `ai_sensors_recompute_*_extrema` | - | §4b |
| +0x2a0 | owner | H | `ai_sensors_init` | every `ai_sensors_can_*` / `_get_*` | the owning C_entity; route to the `ai_character` stats at C_entity+0xebc |
| +0x1c0 +0xc (slot) | smell_contact.blob | H | `ai_sensors_new_smell_contact`, `ai_sensors_update_smell_contact` | `ai_sensors_get_smelled_target_info` | an **owned `ai_blob*`** ¹ |

¹ Freed by `_cancel_smell_contact` / `_destroy`, moved (not copied) by `_update_smell_contact`.
Smell is the ONLY sense holding heap memory.

## 3. The `+0xc74` heard-event ring writer

```
  <a sound is emitted>                          sound_arena_emit_sound
        |  registers the sound into the MISSION-owned soundArena @ mission+0x2b3c
        v
  entity_ai  ---(x19, once per listening entity)-->  sound_arena_update_entity_sound_contacts
        |  for each live sound: intensity = base - (dist*base)/range
        |  audible iff  intensity > 1.05 - entity_get_hearing_sensitivity()
        |
        +-- audible ----> ai_sensors_new_sound_contact   (0x0040faf0)
        |                    -> WRITES the ring slot at ai_sensors+0xc0 (= C_entity+0xc74)
        |                       type 1 -> dispatches entity event 6
        |                       type 2 (GUNFIRE) -> dispatches entity event 5
        |
        +-- inaudible --> ai_sensors_cancel_sound_contact (0x0040fbf0)
                             -> ZEROES the slot's type field (C_entity+0xc78)
```

So `entity_has_heard_gunfire`'s "type 2" entries are written by **`ai_sensors_new_sound_contact`**,
driven by **`sound_arena_update_entity_sound_contacts`**, gated by the NPC's **hearing** stat
(`ai_character+0x18`, see `structs/ai_character.md`). Gunfire is heard when it is loud enough
*relative to that NPC's hearing*, not at a fixed radius.

## 4. Method family — PC ↔ PS2 (23 methods)

Sorted by PS2 VA (= PC VA order). Every row is body-diffed on both builds; none rests on rank
interpolation, a name, or a callee/constant fingerprint alone.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0040f760 | `ai_sensors_find_visual_contact` | `hasVisualCon__10ai_sensorsFP7C_actor` | 0x0030e720 | **CONFIRMED** | Identical loop; `+0x8`=state, `+0xc`=actor, stride 0x18, same `−1` contract. PC scans 8 slots (§5) |
| 0x0040f790 | `ai_sensors_find_sound_contact` | `hasSoundCon__10ai_sensorsFP7C_actorUib` | 0x0030e770 | **CONFIRMED** | Array base **0xc0 both builds**; `+0x4`=type, `+0x8`=actor; 8 slots both; same `any_type==1`/`==0` branch split; same `−1` contract |
| 0x0040f7f0 | `ai_sensors_find_smell_contact` | `hasSmellCon__10ai_sensorsFP7C_actor` | 0x0030e820 | **CONFIRMED** | Identical loop; `+0x8`=state, `+0x10`=actor, stride **0x1c**, 8 slots both, same `−1` contract ¹ |
| 0x0040f820 | `ai_sensors_new_visual_contact` | `NewVisualCon__10ai_sensorsFP17ai_visual_contact` | 0x0030e870 | **CONFIRMED** | Event **3**; identical 3-statement prologue (`[2]=1; [0]=[1]; [5]=0`); extrema gate on `[4]`; tail `UpdateVIS(0, slot_count)` ² |
| 0x0040f8b0 | `ai_sensors_update_visual_contact` | `UpdateVisualCon__10ai_sensorsFiP17ai_visual_contact` | 0x0030e940 | **CONFIRMED** | Byte-identical EMA `(old*3.0 + (new−prev))*0.25` — **the filter all three senses use**. `state=2`; tail `UpdateVIS(2, idx)` |
| 0x0040f900 | `ai_sensors_cancel_visual_contact` | `CancelVisualCon__10ai_sensorsFP7C_actorb` | 0x0030e9c0 | **CONFIRMED** | Event **4**; `state=0`; `UpdateVIS(1, idx)`; then the kind gate `==0x1b \|\| ==2` on the entity kind field (`structs/entity.md` §1) ³ |
| 0x0040f9c0 | `ai_sensors_recompute_visual_extrema` | `UpdateVIS__10ai_sensorsFUiUi` | 0x0030eb40 | **CONFIRMED** | Same 0/1/2 mode switch; four extrema scalars (§4b) |
| 0x0040faf0 | `ai_sensors_new_sound_contact` | `NewSoundCon__10ai_sensorsFP16ai_sound_contact` | 0x0030ecb0 | **CONFIRMED** | **The `C_entity+0xc74` ring writer** (§3). Find-or-update; type 1 ⇒ event 6, type 2 ⇒ event 5; extrema early-out. Sole callsite `lea ecx,[ebp+0xbb4]` — the receiver proof for the class |
| 0x0040fb80 | `ai_sensors_update_sound_contact` | `UpdateSoundCon__10ai_sensorsFiP16ai_sound_contact` | 0x0030edb0 | **CONFIRMED** | Same EMA 3.0/0.25; same slot base 0xc0 both builds; tail `UpdateSND(2, idx)` |
| 0x0040fbf0 | `ai_sensors_cancel_sound_contact` | `CancelSoundCon__10ai_sensorsFP7C_actorUi` | 0x0030ee30 | **CONFIRMED** | Zeroes the slot's **type** at `+0xc4 + idx*0x20` (= `C_entity+0xc78`) — what makes `entity_has_heard_gunfire`'s type-2 scan miss it |
| 0x0040fc30 | `ai_sensors_recompute_sound_extrema` | `UpdateSND__10ai_sensorsFUiUi` | 0x0030ee90 | **CONFIRMED** | Same 0/1/2 modes; scalars exactly visual+4 on both builds ⇒ per-sense parallel arrays (§4b) |
| 0x0040fd70 | `ai_sensors_new_smell_contact` | `NewSmellCon__10ai_sensorsFP16ai_smell_contact` | 0x0030f000 | **CONFIRMED** | Byte-identical: key at `contact[4]`, `*src=src[1]`, `src[6]=0`, free-existing-blob, copy 0x1c, `src[3]=0`, `UpdateSMELL(0, slot_count)`. `(idx+0x10)*0x1c` **pins smell base 0x1c0**; slot_count `+0x2c4` = visual+8 ² |
| 0x0040fe30 | `ai_sensors_update_smell_contact` | `UpdateSmellCon__10ai_sensorsFiP16ai_smell_contact` | 0x0030f0f0 | **CONFIRMED** | Same EMA 3.0/0.25; identical **ai_blob ownership TRANSFER** (`dst=src->blob; src->blob=0`); tail `UpdateSMELL(2, idx)` |
| 0x0040feb0 | `ai_sensors_cancel_smell_contact` | `CancelSmellCon__10ai_sensorsFP7C_actor` | 0x0030f1c0 | **CONFIRMED** | find → `state=0` → **free the ai_blob** (`operator_delete_wrapper`) → `UpdateSMELL(1, idx)` |
| 0x0040ff10 | `ai_sensors_recompute_smell_extrema` | `UpdateSMELL__10ai_sensorsFUiUi` | 0x0030f250 | **CONFIRMED** | Same 0/1/2 modes + min/max structure, incl. mode 1 re-sweeping the max *only* when the cancelled slot **was** the max (§4b) |
| 0x00410050 | `ai_sensors_init` | `Init__10ai_sensorsFPv` | 0x0030f3c0 | **CONFIRMED** | Owner backpointer; the distinctive **three-cursors-in-one-loop** shape, 8 iterations; same trailing extrema clears. Proves 8 visual slots are allocated (§5) |
| 0x004100f0 | `ai_sensors_destroy` | `Destroy__10ai_sensorsFv` | 0x0030f460 | **CONFIRMED** | **The strongest fingerprint in the class:** zeroes the **same twelve** scalars in the same four groups of three, same order (§4b). Same triple-stride loop; blob free |
| 0x004101b0 | `ai_sensors_can_see` | `isVisible__10ai_sensorsFP7C_actor` | 0x0030f560 | **CONFIRMED** | `find_visual_contact` → `get_sight` → shared `1.05` → field `+0x10 + idx*0x18` |
| 0x00410200 | `ai_sensors_can_hear` | `isHearable__10ai_sensorsFP7C_actorUib` | 0x0030f5f0 | **CONFIRMED** | `find_sound_contact` → stale-check (4000 ms) → `cancel_sound_contact`; else `get_hearing` + shared `1.05` (§5) |
| 0x00410290 | `ai_sensors_can_smell` | `isSmellable__10ai_sensorsFP7C_actorUi` | 0x0030f6d0 | **CONFIRMED** | `find_smell_contact` → **`1.0`** (not 1.05) + a **1000 ms** window. **Reads the SIGHT stat** (§4d) |
| 0x00410300 | `ai_sensors_get_smelled_target_info` | `Sniff__10ai_sensorsFP7C_actorP8S_vectorP8S_vectorPUiPUi` | 0x0030f780 | **CONFIRMED** | Shared magic bit `0x4000` on the owner flag word; 6 params both; live branch → `UpdateWMatrixProc` + frame pos/dir; cached branch → `find_smell_contact` + **`1.0 − sight`** + the blob |
| 0x004105a0 | `ai_sensors_get_heard_sound_info` | `Hear__10ai_sensorsFP7C_actorP8S_vectorPUiUi` | 0x0030f970 | **CONFIRMED** | `find_sound_contact(…, flag=0)` exact-match; `get_hearing` + **`1.0`** (vs `can_hear`'s 1.05); 5 params both |
| 0x00410630 | `ai_sensors_build_smell_contact_from_blob` | `AnalyzeBlob__10ai_sensorsFP7ai_blobP16ai_smell_contact` | 0x0030fa50 | **CONFIRMED** | Field-for-field identical: `+4`=mission clock, `+0xc`=blob, `+0x10`=blob field, `+8`=1, `+0x14`=**1.0f**. PC `operator_new(0x1c)` = exactly the smell-contact stride ³ |

¹ Base PC 0x1c0 ↔ PS2 0x240 (§5).
² PS2 `memcpy` ↔ PC inlined dword loop (the MSVC CRT intrinsic).
³ Signature differences in §5.

Two sibling ticks belong to sub-objects, not to `C_entity`: `ai_vis_logs_tick` (the visual feed)
and `ai_logs_tick`.

### 4b. The extrema scalars are per-sense PARALLEL ARRAYS

`UpdateVIS` and `UpdateSND` are the same function per sense, and their scalars are 4 bytes apart.
`ai_sensors_destroy` zeroes all twelve, in four groups of three (min value, max value, min index,
max index), each group ordered visual / sound / smell:

| scalar | visual | sound | **smell** |
|---|---|---|---|
| min value | `+0x2a4` | `+0x2a8` | **`+0x2ac`** |
| max value | `+0x2b0` | `+0x2b4` | **`+0x2b8`** |
| min index | `+0x2bc` | `+0x2c0` | **`+0x2c4`** |
| max index | `+0x2c8` | `+0x2cc` | **`+0x2d0`** |

The smell scalars sit at visual+8 for all four — the three senses are one template instantiated
per sense.

### 4c. Emission order

All 23 members line up 1:1 across builds in emission order with zero gaps — an independent
confirmation of the whole table. The `hasSmellCon` row was generated by that order as a forced
position and then body-diffed; it is not an interpolation.

### 4d. Smell reuses the SIGHT stat — by design

`ai_sensors_can_smell` and `ai_sensors_get_smelled_target_info` both read the **sight** stat
(`ai_character_get_sight`) in a *smell* predicate. This is original behaviour, identical on both
builds: **`ai_character` has only five stats** — morale, intelligence, aggression, sight, hearing
(`structs/ai_character.md`). **There is no smell stat**, so the smell path reuses sight as its
sensitivity input.

The thresholds are not uniform either:

| predicate | stat used | threshold | extra gate |
|---|---|---|---|
| `ai_sensors_can_see` | sight | `1.05 − stat` | - |
| `ai_sensors_can_hear` | hearing | `1.05 − stat` | staleness 4000 ms |
| `ai_sensors_can_smell` | **sight** | **`1.0` − stat** | age < **1000 ms** |
| `ai_sensors_get_heard_sound_info` | hearing | **`1.0` − stat** | - |
| `ai_sensors_get_smelled_target_info` | **sight** | **`1.0` − stat** | owner flag bit `0x4000` picks live vs cached |

The `1.05` vs `1.0` split is consistent on both builds: the **can-I-perceive predicates** use 1.05,
the **where-is-it queries** use the slightly more permissive 1.0.

## 5. PC vs PS2

- **Location:** PS2 `C_entity + 0xd00` (`NewSoundCon__10ai_sensors(entity + 0xd00, ...)`), delta
  **0x14c** from PC `+0xbb4`.
- **Sound array:** PS2's ring sits at `C_entity+0xdd0` = `ai_sensors + 0xd0`. PC `0xc0` vs PS2
  `0xd0` = **0xc apart** — consistent with **three S_vectors** ahead of the array (PS2 vectors are
  16 B, PC vec3s 12 B ⇒ +4 drift each; cf. the head-band drift arithmetic in `structs/entity.md`
  §4). Yet `hasSoundCon`/`UpdateSoundCon` use slot base 0xc0 on both builds.
- **Sound-contact record:** PS2's is stride 0x30 and its **fields are PERMUTED**, not merely
  offset: the S_vector sits at `+0x00` (VU 16-byte alignment) and the scalars follow — PS2 `+0x10`
  timestamp, `+0x14` type, `+0x18` actor, `+0x1c` intensity, `+0x20` EMA. There is no constant delta
  inside this record.
- **Smell base:** PC `0x1c0` ↔ PS2 `0x240` (delta 0x80). **Owner:** PC `+0x2a0` ↔ PS2 `+0x320`.
- **Extrema scalars** — every one at the constant 0x80 delta:

  | scalar | PS2 visual | PS2 sound | PS2 smell |
  |---|---|---|---|
  | min value | `+0x324` | `+0x328` | `+0x32c` |
  | max value | `+0x330` | `+0x334` | `+0x338` |
  | min index | `+0x33c` | `+0x340` | `+0x344` |
  | max index | `+0x348` | `+0x34c` | `+0x350` |

- **Visual scan bound — a real build difference.** Both builds ALLOCATE 8 visual slots (the
  init/teardown loop runs 8× on both). PS2 `hasVisualCon` / `UpdateVIS` stop at **6**
  (`if (5 < i) return -1;` / `while (i < 6)`) versus **8** on PC, so PS2 leaves 2 slots as dead
  storage. Sound and smell scan 8 on both. This is a tuning constant, and together with the
  round-robin `ai_sight_manager` (`structs/entity.md` §6c) it shows the PS2 port throttling
  perception. Consequence: a PS2 NPC tracks at most 6 visible things at once vs 8 on PC.
- **Hearing staleness window:** PC 4000 ms vs PS2 6000 ms — a real tuning difference.
- **Signatures and codegen:** `CancelVisualCon` takes a 3rd `bool` param PC lacks; its kind gate
  reads PC `+0x10` / PS2 `+0x50`. `AnalyzeBlob`: PC allocates and returns (1 param), PS2 fills a
  caller record (3 params). `Hear`: PS2 `memcpy(...,0x10)` S_vector ↔ PC 3-dword vec3 copy. Blob
  frees: PS2 `__dl__7ai_blob` ↔ PC `operator_delete_wrapper`. `Sniff`: branch order inverted
  (normal codegen). The sight stat getter is PS2 `GetZrak`.
- **`UpdateSoundCon`'s PS2 definition line is wrapped** in the PS2 decomp, so a `type name(` search
  misses it.

## 6. Cross-references

- `structs/entity.md` §4/§5 — the C_entity field band; the `+0xc74` heard-event ring.
- `structs/ai_character.md` — the stat block at `C_entity+0xebc`; its **hearing** stat is the
  threshold operand for sound contacts, its **sight** stat feeds `entity_get_sight_range`.
- `structs/dog.md` — the likely consumer of the smell sense.

## 7. Open items

| item | state |
|---|---|
| `ai_visual_contact` / `ai_sound_contact` / `ai_smell_contact` as named structs | layouts known (§1); not yet declared as separate structs |
| `ai_blob` (2 methods) | the scent blob — allocated by `ai_sensors_build_smell_contact_from_blob`, owned by the smell slot; fields `+0x00`/`+0x10`/`+0x20`/`+0x24` (what `Sniff` copies out) unread. `ai_blob_log` has 3 named PC methods |
| who consumes smell | the callers of `ai_sensors_can_smell` / `ai_sensors_get_smelled_target_info` / the `ai_blob` producers; `C_dog` is the prime suspect |
| who emits scent blobs | `sound_arena_emit_sound` is the sound producer; the smell producer (what creates `ai_blob`s in the world) is not located |
| PS2 crowded-fight tracking | in game: PS2 NPCs should be measurably worse at tracking multiple visible targets (§5) |
