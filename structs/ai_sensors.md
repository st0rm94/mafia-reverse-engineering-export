# `ai_sensors` — the three-sense contact system (sight / sound / smell)

**Created** 2026-07-16 10:28 +0200 · **Modified** 2026-11-03 23:04 +0100

`ai_sensors` is a **sub-object EMBEDDED in `C_entity` at `+0xbb4`** (§4a). It is the NPC's *sensory
short-term memory*: a set of **contact** records, one per perceived actor per sense, each with a
strength/intensity that decays and expires. Three parallel senses, each with the identical
**`has` / `New` / `Update` / `Cancel` / `Update<SENSE>`** five-method shape:

| Sense | Contact struct | Notes |
|---|---|---|
| **VISUAL** | `ai_visual_contact` | fed by `ai_vis_logs_tick` (LOS + sight range) |
| **SOUND** | `ai_sound_contact` | fed by `sound_arena_update_entity_sound_contacts` (loudness vs hearing) |
| **SMELL** | `ai_smell_contact` | fed by `ai_sensors_build_smell_contact_from_blob` + `ai_blob`; blobs come from `ai_blob_log_add_blob` (trails) and `ai_group_send_sniff_event` (on demand); read by `ai_sensors_can_smell` and `ai_sensors_get_smelled_target_info` (§6a) |

Every `ai_sensors_*` predicate reaches the NPC's stats through `owner` (`+0x2a0`) → the
`ai_character` at `C_entity+0xebc` (`structs/ai_character.md`). That is why the predicates can read
a **sight** or **hearing** stat without taking it as a parameter.

| | |
|---|---|
| PC prefix | `ai_sensors_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x2d4` — the extrema scalars end at `+0x2d4`, and the next `C_entity` member, `ai_logs`, starts at `C_entity+0xe88` = `0xbb4 + 0x2d4` |
| PC constructor | - |
| PC translation unit | `[0x0040f700, 0x0043b000)`, the `C_entity` unit (`structs/entity.md` §8); this class is `[0x0040f760, 0x00410670)` inside it, with no foreign function |
| PS2 class | `ai_sensors` (tag `10ai_sensors`), 23 methods: 23 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

PC offsets, relative to the `ai_sensors` base. The three contact arrays and all twelve extrema
scalars are proven by `ai_sensors_init` / `ai_sensors_destroy` (§2d).

```c
typedef struct ai_sensors {                 // 0x2d4 bytes, embedded at C_entity+0xbb4
    /* +0x000 */ ai_visual_contact visual[8];  // [H] up to 8 seen actors, stride 0x18 (§1a)
    /* +0x0c0 */ ai_sound_contact  sound[8];   // [H] up to 8 heard sources, stride 0x20 (§1b)
    /* +0x1c0 */ ai_smell_contact  smell[8];   // [H] up to 8 smelled actors, stride 0x1c (§1c)
    /* +0x2a0 */ C_entity *owner;              // [H] owning entity; stored by ai_sensors_init
    /* +0x2a4 */ float     min_value[3];       // [H] visual / sound / smell (§2d)
    /* +0x2b0 */ float     max_value[3];       // [H] visual / sound / smell
    /* +0x2bc */ uint32_t  min_index[3];       // [H] visual / sound / smell; also the append index
    /* +0x2c8 */ uint32_t  max_index[3];       // [H] visual / sound / smell
} ai_sensors;
```

**Why arrays of eight, and what each entry is.** Each array is a pool of eight contacts of one
sense, one entry per perceived actor: the seen actor (visual), the sound's source (sound) or the
blob's owner (smell), which is the key every `ai_sensors_find_*_contact` scans for. A contact for
an actor the pool holds updates its entry in place; a new one is written into the slot `min_index` names
(§2d), which is the **weakest** contact after an insert and the **just-freed** slot after a
cancel. Visual and sound insert only when the newcomer is stronger than that weakest entry
(`min_value`), so a full pool evicts its weakest contact; smell always takes the slot, freeing
the blob it held. Slot 0 of the sound pool is `C_entity+0xc74`, the heard-event ring of §5.

The count is a constant eight: `ai_sensors_init` and `ai_sensors_destroy` walk eight slots of each
array, and every PC scan and extrema sweep stops at eight. Nothing in the binary derives the
number; it is a capacity, so an NPC tracks at most eight actors per sense at once.

The type names of the three contact records are the PS2 signatures'
(`NewVisualCon__10ai_sensorsFP17ai_visual_contact` and its siblings). A contact's stamps are the
mission clock (`*(g_act_mission+0x24)+0x2b0c`); a visual or smell contact's state is 1 on insert and
0 once cancelled, and a visual update sets 2. The smoothed value is the shared EMA
`(old*3.0 + (new - prev)) * 0.25`.

### 1a. `ai_visual_contact`

```c
typedef struct ai_visual_contact {          // 0x18 bytes; built by ai_vis_logs_tick
    /* +0x00 */ uint32_t first_ms;             // [H] = last_ms when the contact is inserted
    /* +0x04 */ uint32_t last_ms;              // [H] refreshed by every update
    /* +0x08 */ uint32_t state;                // [H] 0 / 1 / 2
    /* +0x0c */ C_actor *actor;                // [H] the seen actor; the scan key
    /* +0x10 */ float    strength;             // [H] 0..1, gated by 1.05 - sight
    /* +0x14 */ float    smoothed;             // [H] the EMA
} ai_visual_contact;
```

### 1b. `ai_sound_contact`

The record is measured in the disassembly (§4b); the decompiler renders the same offsets.

```c
typedef struct ai_sound_contact {           // 0x20 bytes; built by sound_arena_update_entity_sound_contacts
    /* +0x00 */ uint32_t time_ms;              // [H] when the sound was registered
    /* +0x04 */ uint32_t type;                 // [H] 2 = gunfire; zeroed to kill the slot
    /* +0x08 */ float    pos[3];               // [H] where the sound was
    /* +0x14 */ C_actor *source;               // [H] the scan key
    /* +0x18 */ float    intensity;            // [H] attenuated loudness
    /* +0x1c */ float    smoothed;             // [H] the EMA
} ai_sound_contact;
```

### 1c. `ai_smell_contact`

```c
typedef struct ai_smell_contact {           // 0x1c bytes; built by ai_sensors_build_smell_contact_from_blob
    /* +0x00 */ uint32_t first_ms;             // [H] = last_ms when the contact is inserted
    /* +0x04 */ uint32_t last_ms;              // [H] mission clock of the blob reading
    /* +0x08 */ uint32_t state;                // [H] 0 / 1 / 2
    /* +0x0c */ ai_blob *blob;                 // [H] OWNED; moved, never copied
    /* +0x10 */ C_actor *actor;                // [H] the blob's owner (ai_blob+0x20); the scan key
    /* +0x14 */ float    strength;             // [H] 1.0 when built; gated by 1.0 - sight
    /* +0x18 */ float    smoothed;             // [H] the EMA
} ai_smell_contact;
```

`ai_blob` itself (0x24 bytes: position, movement direction, emission time, flags, owner) is laid out
in `structs/ai_blob_log.md` §1a; `ai_sensors_get_smelled_target_info` copies its `+0x00` position,
`+0x0c` direction, `+0x18` time and `+0x1c` flags.

## 2. Field evidence

The scalar rows are measured by a census of every `this+N` access in the `ai_sensors_*` methods;
the contact-record rows come from the body-diffs of §3, which name each method's reads and writes.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | visual | H | per field, §2a | per field, §2a | 8 slots; `ai_sensors_init` / `ai_sensors_destroy` walk all eight |
| +0x0c0 | sound | H | per field, §2b | per field, §2b | 8 slots; the `C_entity+0xc74` heard-event ring (§5) |
| +0x1c0 | smell | H | per field, §2c | per field, §2c | 8 slots; base pinned by `(idx+0x10)*0x1c` in `ai_sensors_new_smell_contact` |
| +0x2a0 | owner | H | `ai_sensors_init` | every `ai_sensors_can_*` / `_get_*` | the owning `C_entity`; the route to the `ai_character` stats at `C_entity+0xebc` |
| +0x2a4 | min_value | H | the three `ai_sensors_recompute_*_extrema`, `ai_sensors_init`, `ai_sensors_destroy` | `ai_sensors_new_visual_contact`, `ai_sensors_new_sound_contact`, the three `ai_sensors_recompute_*_extrema` | a new visual contact is inserted only when `this[+0x2a4] < contact.strength`; a new sound contact exits early when `contact.intensity <= this[+0x2a8]` (§2d) |
| +0x2b0 | max_value | H | the three `ai_sensors_recompute_*_extrema`, `ai_sensors_init`, `ai_sensors_destroy` | the three `ai_sensors_recompute_*_extrema` | §2d |
| +0x2bc | min_index | H | the three `ai_sensors_recompute_*_extrema`, `ai_sensors_init`, `ai_sensors_destroy` | the three `ai_sensors_new_*_contact` | also the APPEND index (`this + idx*stride`) |
| +0x2c8 | max_index | H | the three `ai_sensors_recompute_*_extrema`, `ai_sensors_init`, `ai_sensors_destroy` | the three `ai_sensors_recompute_*_extrema` | §2d |

### 2a. `ai_visual_contact`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | first_ms | H | `ai_sensors_new_visual_contact` (= `last_ms`), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `entity_save_sensors` | no reader but the savegame ¹ |
| +0x04 | last_ms | H | `ai_vis_logs_tick` (builds the contact), `ai_sensors_update_visual_contact`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `entity_save_sensors` | `ai_sensors_new_visual_contact` copies the incoming record's `last_ms` to `first_ms`, not the slot's |
| +0x08 | state | H | `ai_sensors_new_visual_contact` (= 1), `ai_sensors_update_visual_contact` (= 2), `ai_sensors_cancel_visual_contact` (= 0), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_visual_contact`, `entity_save_sensors` | the scan skips state 0 |
| +0x0c | actor | H | `ai_vis_logs_tick`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_visual_contact` (the key), `ai_sensors_cancel_visual_contact` (its kind gate), `entity_save_sensors` | - |
| +0x10 | strength | H | `ai_vis_logs_tick`, `ai_sensors_update_visual_contact`, `entity_load_sensors` | `ai_sensors_new_visual_contact` (the extrema gate), `ai_sensors_recompute_visual_extrema`, `ai_sensors_can_see`, `entity_save_sensors` | compared against `1.05 − sight` |
| +0x14 | smoothed | H | `ai_sensors_new_visual_contact` (= 0), `ai_sensors_update_visual_contact` (the EMA), `entity_load_sensors` | `ai_sensors_update_visual_contact`, `entity_save_sensors` | - |

¹ The smell record's `first_ms` has no reader either; both are a per-contact "first seen" stamp
that nothing in the PC build consults but the savegame.

### 2b. `ai_sound_contact`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_ms | H | `sound_arena_update_entity_sound_contacts` (builds the contact), `ai_sensors_new_sound_contact` (copies it into the slot), `ai_sensors_update_sound_contact`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_can_hear` (the 4000 ms staleness test), `entity_has_heard_gunfire`, `ai_sensors_get_heard_sound_info`, `entity_save_sensors` | slot 0 is `C_entity+0xc74` |
| +0x04 | type | H | `sound_arena_update_entity_sound_contacts`, `ai_sensors_new_sound_contact`, `ai_sensors_cancel_sound_contact` (= 0), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_sound_contact`, `ai_sensors_new_sound_contact` (type 1 ⇒ event 6, type 2 ⇒ event 5), `entity_has_heard_gunfire`, `ai_sensors_can_hear` (hands it to the cancel), `entity_save_sensors` | **type 2 = gunfire**; zeroing it kills the slot (§5) |
| +0x08 | pos | H | `sound_arena_update_entity_sound_contacts`, `ai_sensors_new_sound_contact`, `ai_sensors_update_sound_contact`, `entity_load_sensors` | `ai_sensors_get_heard_sound_info`, `entity_save_sensors` | three floats |
| +0x14 | source | H | `sound_arena_update_entity_sound_contacts`, `ai_sensors_new_sound_contact`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_sound_contact` (the key), `entity_save_sensors` | the scan reads it at `+0xd4 + i*0x20` (§4b) |
| +0x18 | intensity | H | `sound_arena_update_entity_sound_contacts`, `ai_sensors_new_sound_contact`, `ai_sensors_update_sound_contact`, `entity_load_sensors` | `ai_sensors_new_sound_contact` (the extrema early-out), `ai_sensors_update_sound_contact` (the EMA's previous value), `ai_sensors_recompute_sound_extrema`, `ai_sensors_can_hear`, `ai_sensors_get_heard_sound_info`, `entity_save_sensors` | compared against `1.05 − hearing` (`ai_sensors_can_hear`) and `1.0 − hearing` (`ai_sensors_get_heard_sound_info`) |
| +0x1c | smoothed | H | `ai_sensors_new_sound_contact` (= 0), `ai_sensors_update_sound_contact` (the EMA), `entity_load_sensors` | `ai_sensors_update_sound_contact`, `entity_save_sensors` | - |

### 2c. `ai_smell_contact`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | first_ms | H | `ai_sensors_new_smell_contact` (= `last_ms`), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `entity_save_sensors` | no reader but the savegame |
| +0x04 | last_ms | H | `ai_sensors_build_smell_contact_from_blob` (= mission clock), `ai_sensors_update_smell_contact`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_new_smell_contact` (copies it to `first_ms`), `ai_sensors_can_smell` (the 1000 ms window), `entity_save_sensors` | - |
| +0x08 | state | H | `ai_sensors_build_smell_contact_from_blob` (= 1), `ai_sensors_cancel_smell_contact` (= 0), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_smell_contact`, `entity_save_sensors` | the scan skips state 0; the smell update leaves it at 1 |
| +0x0c | blob | H | `ai_sensors_build_smell_contact_from_blob`, `ai_sensors_new_smell_contact` and `ai_sensors_update_smell_contact` (both move it: `dst = src->blob; src->blob = 0`), `ai_sensors_cancel_smell_contact`, `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_get_smelled_target_info`, `ai_sensors_cancel_smell_contact`, `ai_sensors_destroy`, `entity_save_sensors` | an **owned `ai_blob *`**, freed by `ai_sensors_cancel_smell_contact` and `ai_sensors_destroy`. Smell is the only sense holding heap memory |
| +0x10 | actor | H | `ai_sensors_build_smell_contact_from_blob` (from `ai_blob+0x20`), `ai_sensors_init`, `ai_sensors_destroy` (= 0), `entity_load_sensors` | `ai_sensors_find_smell_contact` (the key), `entity_save_sensors` | - |
| +0x14 | strength | H | `ai_sensors_build_smell_contact_from_blob` (= 1.0f), `entity_load_sensors` | `ai_sensors_can_smell`, `ai_sensors_get_smelled_target_info`, `ai_sensors_update_smell_contact` (the EMA's previous value), `ai_sensors_recompute_smell_extrema`, `entity_save_sensors` | compared against `1.0 − sight` (§6); the update keeps the stored strength and feeds only the EMA |
| +0x18 | smoothed | H | `ai_sensors_new_smell_contact` (= 0), `ai_sensors_update_smell_contact` (the EMA), `entity_load_sensors` | `ai_sensors_update_smell_contact`, `entity_save_sensors` | - |

### 2d. The extrema scalars are per-sense parallel arrays

`ai_sensors_recompute_visual_extrema` and `ai_sensors_recompute_sound_extrema` are the same function
per sense, and their scalars are 4 bytes apart. `ai_sensors_destroy` zeroes all twelve, in four
groups of three (min value, max value, min index, max index), each group ordered visual / sound /
smell:

| Scalar | Visual | Sound | Smell |
|---|---|---|---|
| min value | `+0x2a4` | `+0x2a8` | `+0x2ac` |
| max value | `+0x2b0` | `+0x2b4` | `+0x2b8` |
| min index | `+0x2bc` | `+0x2c0` | `+0x2c4` |
| max index | `+0x2c8` | `+0x2cc` | `+0x2d0` |

The smell scalars sit at visual+8 for all four — the three senses are one template instantiated
per sense. Each recompute runs in one of three modes: 0 after an insert, 1 after a cancel (mode 1
re-sweeps the max only when the cancelled slot **was** the max), 2 after an update.

## 3. Method family

Coverage: all **23** PS2 `10ai_sensors` methods are in the tables below, **23 CONFIRMED twins**,
every row body-diffed on both builds; none rests on rank interpolation, a name, or a
callee/constant fingerprint alone. The class has no PC-only function.

### 3a. Construction & destruction

The class has no constructor or destructor of its own: `ai_sensors_init` and `ai_sensors_destroy`
(§3c) are the named `Init` / `Destroy` pair the owner calls.

### 3b. Accessors

None.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0040f760 | `ai_sensors_find_visual_contact` | `hasVisualCon__10ai_sensorsFP7C_actor` | 0x0030e720 | CONFIRMED | identical loop; `+0x8` = state, `+0xc` = actor, stride 0x18, same `−1` contract. PC scans 8 slots (§7) |
| 0x0040f790 | `ai_sensors_find_sound_contact` | `hasSoundCon__10ai_sensorsFP7C_actorUib` | 0x0030e770 | CONFIRMED | array base **0xc0 on both builds**; type and actor keys; 8 slots on both; the same `any_type == 1` / `== 0` branch split; same `−1` contract |
| 0x0040f7f0 | `ai_sensors_find_smell_contact` | `hasSmellCon__10ai_sensorsFP7C_actor` | 0x0030e820 | CONFIRMED | identical loop; `+0x8` = state, `+0x10` = actor, stride **0x1c**, 8 slots on both, same `−1` contract |
| 0x0040f820 | `ai_sensors_new_visual_contact` | `NewVisualCon__10ai_sensorsFP17ai_visual_contact` | 0x0030e870 | CONFIRMED | event **3**; identical 3-statement prologue (`[2]=1; [0]=[1]; [5]=0`); extrema gate on `[4]`; tail `UpdateVIS(0, slot_count)` ¹ |
| 0x0040f8b0 | `ai_sensors_update_visual_contact` | `UpdateVisualCon__10ai_sensorsFiP17ai_visual_contact` | 0x0030e940 | CONFIRMED | byte-identical EMA `(old*3.0 + (new−prev))*0.25` — **the filter all three senses use**; `state = 2`; tail `UpdateVIS(2, idx)` |
| 0x0040f900 | `ai_sensors_cancel_visual_contact` | `CancelVisualCon__10ai_sensorsFP7C_actorb` | 0x0030e9c0 | CONFIRMED | event **4**; `state = 0`; `UpdateVIS(1, idx)`; then the kind gate `== 0x1b \|\| == 2` on the actor's kind field (`structs/entity_kind.md` §1) ² |
| 0x0040f9c0 | `ai_sensors_recompute_visual_extrema` | `UpdateVIS__10ai_sensorsFUiUi` | 0x0030eb40 | CONFIRMED | same 0/1/2 mode switch; four extrema scalars (§2d) |
| 0x0040faf0 | `ai_sensors_new_sound_contact` | `NewSoundCon__10ai_sensorsFP16ai_sound_contact` | 0x0030ecb0 | CONFIRMED | **the `C_entity+0xc74` ring writer** (§5). Find-or-update; type 1 ⇒ event 6, type 2 ⇒ event 5; extrema early-out. Sole callsite `lea ecx,[ebp+0xbb4]` — the receiver proof for the class |
| 0x0040fb80 | `ai_sensors_update_sound_contact` | `UpdateSoundCon__10ai_sensorsFiP16ai_sound_contact` | 0x0030edb0 | CONFIRMED | same EMA 3.0/0.25; same slot base 0xc0 on both builds; tail `UpdateSND(2, idx)` |
| 0x0040fbf0 | `ai_sensors_cancel_sound_contact` | `CancelSoundCon__10ai_sensorsFP7C_actorUi` | 0x0030ee30 | CONFIRMED | zeroes the slot's **type** at `+0xc4 + idx*0x20` (= `C_entity+0xc78`) — what makes `entity_has_heard_gunfire`'s type-2 scan miss it |
| 0x0040fc30 | `ai_sensors_recompute_sound_extrema` | `UpdateSND__10ai_sensorsFUiUi` | 0x0030ee90 | CONFIRMED | same 0/1/2 modes; scalars exactly visual+4 on both builds ⇒ per-sense parallel arrays (§2d) |
| 0x0040fd70 | `ai_sensors_new_smell_contact` | `NewSmellCon__10ai_sensorsFP16ai_smell_contact` | 0x0030f000 | CONFIRMED | byte-identical: key at `contact[4]`, `*src = src[1]`, `src[6] = 0`, free the existing blob, copy 0x1c, `src[3] = 0`, `UpdateSMELL(0, slot_count)`. `(idx+0x10)*0x1c` **pins the smell base 0x1c0**; slot count `+0x2c4` = visual+8 ¹ |
| 0x0040fe30 | `ai_sensors_update_smell_contact` | `UpdateSmellCon__10ai_sensorsFiP16ai_smell_contact` | 0x0030f0f0 | CONFIRMED | same EMA 3.0/0.25; identical **ai_blob ownership TRANSFER** (`dst = src->blob; src->blob = 0`); tail `UpdateSMELL(2, idx)` |
| 0x0040feb0 | `ai_sensors_cancel_smell_contact` | `CancelSmellCon__10ai_sensorsFP7C_actor` | 0x0030f1c0 | CONFIRMED | find → `state = 0` → **free the ai_blob** (`operator_delete_wrapper`) → `UpdateSMELL(1, idx)` |
| 0x0040ff10 | `ai_sensors_recompute_smell_extrema` | `UpdateSMELL__10ai_sensorsFUiUi` | 0x0030f250 | CONFIRMED | same 0/1/2 modes and min/max structure, including mode 1 re-sweeping the max *only* when the cancelled slot **was** the max (§2d) |
| 0x00410050 | `ai_sensors_init` | `Init__10ai_sensorsFPv` | 0x0030f3c0 | CONFIRMED | owner backpointer; the distinctive **three-cursors-in-one-loop** shape, 8 iterations; same trailing extrema clears. Proves 8 visual slots are allocated (§7) |
| 0x004100f0 | `ai_sensors_destroy` | `Destroy__10ai_sensorsFv` | 0x0030f460 | CONFIRMED | **the strongest fingerprint in the class:** zeroes the **same twelve** scalars in the same four groups of three, same order (§2d). Same triple-stride loop; blob free |
| 0x004101b0 | `ai_sensors_can_see` | `isVisible__10ai_sensorsFP7C_actor` | 0x0030f560 | CONFIRMED | `ai_sensors_find_visual_contact` → `ai_character_get_sight` → shared `1.05` → field `+0x10 + idx*0x18` |
| 0x00410200 | `ai_sensors_can_hear` | `isHearable__10ai_sensorsFP7C_actorUib` | 0x0030f5f0 | CONFIRMED | `ai_sensors_find_sound_contact` → stale check (4000 ms) → `ai_sensors_cancel_sound_contact`; else `ai_character_get_hearing` + shared `1.05` (§7) |
| 0x00410290 | `ai_sensors_can_smell` | `isSmellable__10ai_sensorsFP7C_actorUi` | 0x0030f6d0 | CONFIRMED | `ai_sensors_find_smell_contact` → **`1.0`** (not 1.05) + a **1000 ms** window. **Reads the SIGHT stat** (§6) |
| 0x00410300 | `ai_sensors_get_smelled_target_info` | `Sniff__10ai_sensorsFP7C_actorP8S_vectorP8S_vectorPUiPUi` | 0x0030f780 | CONFIRMED | shared magic bit `0x4000` on the owner flag word; 6 parameters on both; live branch → `UpdateWMatrixProc` + frame position and direction; cached branch → `ai_sensors_find_smell_contact` + **`1.0 − sight`** + the blob |
| 0x004105a0 | `ai_sensors_get_heard_sound_info` | `Hear__10ai_sensorsFP7C_actorP8S_vectorPUiUi` | 0x0030f970 | CONFIRMED | `ai_sensors_find_sound_contact(…, flag = 0)` exact match; `ai_character_get_hearing` + **`1.0`** (against `ai_sensors_can_hear`'s 1.05); 5 parameters on both |
| 0x00410630 | `ai_sensors_build_smell_contact_from_blob` | `AnalyzeBlob__10ai_sensorsFP7ai_blobP16ai_smell_contact` | 0x0030fa50 | CONFIRMED | field-for-field identical: `+4` = mission clock, `+0xc` = blob, `+0x10` = the blob's owner, `+8` = 1, `+0x14` = **1.0f**. PC `operator_new(0x1c)` = exactly the smell-contact stride ² |

¹ PS2 `memcpy` ↔ PC inlined dword loop (the MSVC CRT intrinsic).
² Signature differences in §7.

Two sibling ticks belong to other sub-objects, not to this class: `ai_vis_logs_tick` (the visual
feed, `structs/entity.md` §7) and `ai_logs_tick` (`structs/ai_logs.md`).

### 3d. Free helpers

None of this class's methods calls a free helper of the shared `C_entity` unit; those helpers are
listed in `structs/entity.md` §3d.

## 4. How the layout is pinned

### 4a. The base `C_entity+0xbb4`

`ai_sensors_cancel_sound_contact` writes `*(this + 0xc4 + idx*0x20) = 0`. With the byte-proven
receiver `this = C_entity + 0xbb4` (`lea ecx, [ebp + 0xbb4]` at the sound-contact callsite in
`sound_arena_update_entity_sound_contacts`), that field is `C_entity + 0xc78 + idx*0x20`.
`structs/entity.md` §1 documents the heard-event ring at **`C_entity+0xc74`, stride 0x20**, with
`entry[0]` = timestamp and `entry[1]` = **type**. `0xc74 + 4 = 0xc78` ⇒ the cancel zeroes exactly
the **type** field. The two independent derivations agree, which pins:

- **`ai_sensors` base = `C_entity + 0xbb4`**;
- **sound-contact array = `ai_sensors + 0xc0`** = `C_entity + 0xc74`.

`structs/entity.md` records "sensors @ `+0xbc0`, serialized size 0x3a0". `+0xbc0` is where
**`entity_save_sensors` begins serializing** (= `ai_sensors + 0xc`); the **object base is
`+0xbb4`**. Both are true; they are different things.

### 4b. The sound-contact record in the disassembly

`ai_sensors_find_sound_contact`'s scan keeps a cursor at `this + 0xc4`, the type of slot 0, and
reads the actor 0x10 bytes past it; the decompiler renders it as `piVar2 = (int*)(this + 0xc4); …
param_2 != piVar2[4]`:

```
  0040f7a3  add  ecx, 0xc4                   ; cursor = this + 0xc4
  0040f7a9  cmp  edx, dword ptr [ecx + 0x10] ; actor  = this + 0xd4 + i*0x20
  0040f7ae  cmp  dword ptr [ecx], 0          ; type   = this + 0xc4 + i*0x20
  0040f7b4  add  ecx, 0x20                   ; stride 0x20, 8 slots
```

The record is the one in §1b.

## 5. The `+0xc74` heard-event ring writer

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
(`ai_character+0x18`, `structs/ai_character.md`). Gunfire is heard when it is loud enough
*relative to that NPC's hearing*, not at a constant radius.

## 6. The thresholds — smell reuses the SIGHT stat

`ai_sensors_can_smell` and `ai_sensors_get_smelled_target_info` both read the **sight** stat
(`ai_character_get_sight`) in a *smell* predicate. This is original behaviour, identical on both
builds: **`ai_character` has only five stats** — morale, intelligence, aggression, sight, hearing
(`structs/ai_character.md`). **There is no smell stat**, so the smell path reuses sight as its
sensitivity input.

The thresholds are not uniform either:

| Predicate | Stat used | Threshold | Extra gate |
|---|---|---|---|
| `ai_sensors_can_see` | sight | `1.05 − stat` | - |
| `ai_sensors_can_hear` | hearing | `1.05 − stat` | staleness 4000 ms |
| `ai_sensors_can_smell` | **sight** | **`1.0` − stat** | age < **1000 ms** |
| `ai_sensors_get_heard_sound_info` | hearing | **`1.0` − stat** | - |
| `ai_sensors_get_smelled_target_info` | **sight** | **`1.0` − stat** | owner flag bit `0x4000` picks live vs cached |

The `1.05` vs `1.0` split is consistent on both builds: the **can-I-perceive predicates** use 1.05,
the **where-is-it queries** use the slightly more permissive 1.0.

### 6a. Who reads smell

Two readers, both measured as `E8` callsite censuses:

| Reader | Callers | What the caller gets |
|---|---|---|
| `ai_sensors_can_smell` | `ai_logs_tick` ×1 | yes / no for one target |
| `ai_sensors_get_smelled_target_info` | `entity_action_tick` ×6, `ai_logs_reset_stale_logs`, `ai_action_new_hostile_coverage`, `police_board_update_sighting` | the target's position, heading, time and flags: live from its frame while the owner flag bit `0x4000` is set, otherwise the blob's `+0x00` position, `+0x0c` direction, `+0x18` time and `+0x1c` flags, while the contact weight exceeds `1.0 − sight` |

In `entity_action_tick` the result drives the steer-to-target arms, a freshness test (under
5000 ms) and the cop chase in case `0x21`. So a smell contact — from a trail blob or from a group's
sniff event (`structs/ai_group.md` §5) — gives an NPC a last-known position and heading for a
target it cannot see.

## 7. PC vs PS2

- **Location:** PS2 `C_entity + 0xd00` (`NewSoundCon__10ai_sensors(entity + 0xd00, ...)`), delta
  **0x14c** from PC `+0xbb4`.
- **Sound array:** PS2's ring sits at `C_entity+0xdd0` = `ai_sensors + 0xd0`. PC `0xc0` vs PS2
  `0xd0` = **0x10 apart**, consistent with the vectors ahead of the array being 16-byte `S_vector`s
  on PS2 and 12-byte vec3s on PC (cf. the head-band drift arithmetic in `structs/entity.md` §9).
  Yet `hasSoundCon` / `UpdateSoundCon` use slot base 0xc0 on both builds.
- **Sound-contact record:** PS2's is stride 0x30 and its **fields are PERMUTED**, not merely
  offset: the S_vector sits at `+0x00` (VU 16-byte alignment) and the scalars follow — PS2 `+0x10`
  timestamp, `+0x14` type, `+0x18` actor, `+0x1c` intensity, `+0x20` EMA. There is no constant delta
  inside this record.
- **Smell base:** PC `0x1c0` ↔ PS2 `0x240` (delta 0x80). **Owner:** PC `+0x2a0` ↔ PS2 `+0x320`.
- **Extrema scalars** — every one at the constant 0x80 delta:

  | Scalar | PS2 visual | PS2 sound | PS2 smell |
  |---|---|---|---|
  | min value | `+0x324` | `+0x328` | `+0x32c` |
  | max value | `+0x330` | `+0x334` | `+0x338` |
  | min index | `+0x33c` | `+0x340` | `+0x344` |
  | max index | `+0x348` | `+0x34c` | `+0x350` |

- **Visual scan bound — a real build difference.** Both builds ALLOCATE 8 visual slots (the
  init/teardown loop runs 8× on both). PS2 `hasVisualCon` / `UpdateVIS` stop at **6**
  (`if (5 < i) return -1;` / `while (i < 6)`) versus **8** on PC, so PS2 leaves 2 slots as dead
  storage. Sound and smell scan 8 on both. This is a tuning constant, and together with the
  round-robin `ai_sight_manager` (`structs/entity.md` §9) it shows the PS2 port throttling
  perception. Consequence: a PS2 NPC tracks at most 6 visible things at once vs 8 on PC.
- **Hearing staleness window:** PC 4000 ms vs PS2 6000 ms — a real tuning difference.
- **Signatures and codegen:** `CancelVisualCon` takes a 3rd `bool` parameter the PC lacks; its kind
  gate reads PC `+0x10` / PS2 `+0x50`. `AnalyzeBlob`: PC allocates and returns (1 parameter), PS2
  fills a caller record (3 parameters). `Hear`: PS2 `memcpy(...,0x10)` S_vector ↔ PC 3-dword vec3
  copy. Blob frees: PS2 `__dl__7ai_blob` ↔ PC `operator_delete_wrapper`. `Sniff`: branch order
  inverted (normal codegen). The sight stat getter is PS2 `GetZrak`.
- **Emission order.** All 23 members line up 1:1 across builds in emission order with zero gaps —
  an independent confirmation of the whole table. The `hasSmellCon` row was generated by that order
  as a forced position and then body-diffed; it is not an interpolation.
- **`UpdateSoundCon`'s PS2 definition line is wrapped** in the PS2 decomp, so a `type name(` search
  misses it.

## 8. Related

- `structs/entity.md` §1/§2 — the `C_entity` field band and the `+0xc74` heard-event ring; §7 —
  the AI sub-object map; §8 — the shared translation unit.
- `structs/ai_character.md` — the stat block at `C_entity+0xebc`; its **hearing** stat is the
  threshold operand for sound contacts, its **sight** stat feeds `entity_get_sight_range`.
- `structs/ai_blob_log.md` §4 — the two producers of scent records: `ai_blob_log_add_blob` lays a
  trail from `human_update`; `ai_group_send_sniff_event` delivers one blob to one entity through
  entity event `0xb` and `ai_sensors_new_smell_contact`.
- `structs/sound_arena.md` — `sound_arena_update_entity_sound_contacts`, the producer of §5.
- `structs/dog.md` §4 — `C_dog` derives from `C_actor`, carries no `ai_sensors`, and is not a smell
  consumer.

## 9. Open items

1. **PS2 crowded-fight tracking** — 🎮 in game: PS2 NPCs should be measurably worse at tracking
   multiple visible targets (§7).
2. **The contact records' readers.** The rows of §2a–§2c come from the body-diffs; a per-field
   access census over every function, including the producers outside the class, has not been run,
   so a reader outside the class may be missing.
