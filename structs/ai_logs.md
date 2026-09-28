# `ai_logs` — the NPC's per-target attitude log

**Created** 2026-07-16 21:10 +0200 · **Modified** 2026-09-27 13:44 +0200

Static analysis of `Game-ghidra.exe.annotated.c` + `Game.exe` disassembly, cross-checked against the
non-stripped PS2 ELF. PS2 material is in §5; the coverage tables in §3 are PC↔PS2 twin tables.

Where `ai_sensors` (`structs/ai_sensors.md`) answers *"can I perceive X?"*, `ai_logs` answers
**"how do I FEEL about X?"** — it keeps one `ai_character_log` per actor the NPC has an opinion about,
scored by fear / anger / threat / death, and hands the AI the current worst offender.
`entity_switch_ai_state` reads those picks to choose a fight state.

- **PC prefix:** `ai_logs_*`. Its record's methods use the sibling prefix **`ai_character_log_*`**.
- **PC location:** **`C_entity + 0xe88`** — byte-proven: `lea ecx,[esi+0xe88]` at all 20 callsites of
  `ai_logs_tick`.
- **Coverage: 14/14 `ai_logs` methods CONFIRMED + 7/7 `ai_character_log`, every row body-diffed on
  both builds. Zero CANDIDATE rows.**

**Confidence tags:** **[H]** writer and reader both read · **[M]** inferred from one side · **[U]** not observed.

---

## 1. `ai_logs` field layout (PC offsets, relative to the `ai_logs` base)

```c
typedef struct ai_logs {                 // embedded at C_entity+0xe88
    /* +0x00 */ uint32_t hdr[9];         // [M] the first 0x24 bytes, zeroed wholesale by ai_logs_init
    /* +0x24 */ ai_character_log *begin; // [H] the LOG VECTOR (begin) = C_entity+0xeac, the
                                         //     "ai_logs list" of structs/entity.md. Freed by the
                                         //     shared fold vector_dispose_at_0x24 ¹
    /* +0x28 */ ai_character_log *end;   // [H] one past the last log. Count = (end - begin) / 0x54.
                                         //     ai_logs_init empties the vector with `end = begin`.
    /* +0x30 */ C_entity *owner;         // [H] owner backpointer, stored by ai_logs_init. Every method
                                         //     that needs the owner's stats, frame or sub-objects
                                         //     reaches them through here.
} ai_logs;                               // partial — only the init/teardown band is pinned
```

¹ `vector_dispose_at_0x24` is an identical-COMDAT fold shared with `ai_loc`'s own `+0x24` vector, so
it belongs to neither class. The `ai_logs` ownership is byte-proven by this owner's two unwind
funclets, `0x00635136` and `0x006351a1`, both `add ecx, 0xe88`.

### 1a. The `ai_character_log` record — the element type (stride `0x54`)

Recovered from the six `ai_character_log_*` methods plus `ai_logs_tick`, `ai_logs_compute_attitude`
and `ai_logs_reset_stale_logs`.

```c
typedef struct ai_character_log {       // 0x54 bytes; element of the ai_logs vector
    /* +0x00 */ C_actor *target;        // [H] who this log is about; the vector's scan key
    /* +0x04 */ uint32_t death_mark;    // [H] 0x29a marks a DEATH-witnessed log. The two threat
                                        //     pickers split on exactly this field, partitioning the
                                        //     vector between them
    /* +0x08 */ uint32_t relation;      // [H] relationship class 0..3, assigned once at creation by
                                        //     ai_logs_event's owner-mode x target-mode matrix; selects
                                        //     the row of g_ai_logs_exposure_table (§4c)
    /* +0x0c */ float    enemy_damage;  // [H] damage this target dealt to MY ENEMIES
    /* +0x10 */ float    own_damage;    // [H] damage this target dealt to ME
    /* +0x14 */ float    friend_damage; // [H] damage this target dealt to MY FRIENDS; folded into
                                        //     anger at 0.05 weight. Its producer is the ai_group
                                        //     event bus - see the note under this block
    /* +0x18 */ uint32_t contact_ts;    // [H] contact-window open stamp; 0 = no live contact, and the
                                        //     gate both max-pickers require
    /* +0x1c */ uint32_t expire_ts;     // [H] contact-window expiry deadline
    /* +0x20 */ uint32_t event_ts;      // [H] last event stamp, written by ai_logs_event
    /* +0x24 */ uint32_t last_seen;     // [H] ai_logs_tick, when ai_sensors_can_see fires
    /* +0x28 */ uint32_t last_heard;    // [H] ai_logs_tick, when ai_sensors_can_hear fires
    /* +0x2c */ uint32_t last_smelled;  // [H] ai_logs_tick, when ai_sensors_can_smell fires
    /* +0x30 */ uint32_t last_aimed;    // [H] visible AND target armed (+0x1e5); the 6250 ms weight
    /* +0x34 */ uint32_t last_shot;     // [H] heard AND target fired <250 ms ago; the 25000 ms weight
    /* +0x38 */ uint32_t next_poll_ts;  // [H] sense re-poll gate; ai_logs_tick pushes it +250 ms
    /* +0x3c */ float    attitude_bias; // [H] input bias; snapshotted from +0x40 on each new contact
    /* +0x40 */ float    attitude_ema;  // [H] smoothed attitude, folded (3*bias + ema) * 0.25 on reset
    /* +0x44 */ float    attitude;      // [H] morale-shaped attitude, clamped at 0
    /* +0x48 */ float    threat;        // [H] exposure-weighted threat — what BOTH max-threat pickers
                                        //     rank, and what the death event overwrites with 2/0/1
    /* +0x4c */ float    anger;         // [H] (aggression + 0.5) scaled; ranked by ai_logs_get_max_angry
    /* +0x50 */ float    fear;          // [H] (1 - morale) scaled; ranked by ai_logs_get_max_fear
} ai_character_log;                     // 0x54 — complete
```

**Two independent derivations agree on the stride and the extent:** `ai_character_log_new_contact`
writes exactly indices `0 .. 0x14` (= `+0x00 .. +0x50`), and every scan in the class advances by
`0x54`. **`+0x24` is doubly confirmed:** `ai_logs_init` clears the vector there, and
`structs/entity.md` independently documents the list at `C_entity+0xeac` — `0xe88 + 0x24 = 0xeac`.

---

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00..+0x23 | hdr | M | `ai_logs_init` | - | not decoded |
| +0x24 / +0x28 | log vector begin/end | H | `ai_logs_init`, `ai_logs_event`, `ai_logs_destroy_log_for_actor`, `ai_logs_tick` | `ai_logs_has_log`, `ai_logs_get_log_for_actor`, `ai_logs_get_max_angry`, `ai_logs_get_max_fear`, `ai_logs_get_max_death`, `ai_logs_get_max_threat_score`, `ai_logs_reset_stale_logs`, `entity_save_logs`, `entity_load_logs` | = `C_entity+0xeac`. Elements are `ai_character_log` (§1a) |
| +0x30 | owner | H | `ai_logs_init` | `ai_logs_event`, `ai_logs_compute_attitude`, `ai_logs_reset_stale_logs` | the owning C_entity; how the class reaches morale/aggression, the I3D_frame and `ai_fight_modes` |

**Serialized size is VARIABLE** (`entity_get_logs_size` loops the list) — unlike the other three
C_entity sub-blocks, whose sizes are constants (`structs/entity.md` §4).

---

## 3. Method family — PC ↔ PS2 (14 methods)

**PC-first, ordered by PC VA.** Every row is body-diffed on BOTH builds.

| PC VA | PC name | PS2 twin (mangled) | Status | Role |
|---|---|---|---|---|
| 0x00401060 | `ai_logs_init` | `Init__7ai_logsFPv` | **CONFIRMED** | Owner backpointer, empty the vector, zero the header |
| 0x004010b0 | `ai_logs_tick` | `Tick__7ai_logsFv` | **CONFIRMED** | Per-frame maintenance: evict dead/expired logs, stamp the per-sense timestamps behind a 250 ms poll gate, then `ai_logs_compute_attitude` on every live log |
| 0x00401440 | `ai_logs_event` | `Event__7ai_logsFP7C_actorUiPv` | **CONFIRMED** | 🔑 **The only WRITER** — find-or-create a log, then route 9 event ids to the five `ai_character_log_event_*` mutators |
| 0x00401810 | `ai_logs_get_aiming_weight` | `GetAimingWeight__7ai_logsFP16ai_character_log` | **CONFIRMED** | Recency weight of `last_aimed`: 1.25 decaying over 6250 ms |
| 0x00401880 | `ai_logs_get_shooting_weight` | `GetShootingWeight__7ai_logsFP16ai_character_log` | **CONFIRMED** | Recency weight of `last_shot`: 1.25 decaying over 25000 ms |
| 0x004018f0 | `ai_logs_compute_attitude` | `ComputeAttitude__7ai_logsFP16ai_character_log` | **CONFIRMED** | 🔑 **The scorer** — morale/aggression + the two weights + damage → attitude/threat/anger/fear |
| 0x00401d20 | `ai_logs_has_log` | `hasLog__7ai_logsFP7C_actorPUi` | **CONFIRMED** | Membership test; optionally outputs the relationship class |
| 0x00401d90 | `ai_logs_get_log_for_actor` | `GetLog__7ai_logsFP7C_actor` | **CONFIRMED** | Find the log for one actor |
| 0x00401df0 | `ai_logs_get_max_angry` | `GetMaxAngryCharacter__7ai_logsFv` | **CONFIRMED** | Worst offender by ANGER (+0x4c). Cached at `C_entity+0x1280` |
| 0x00401e80 | `ai_logs_get_max_fear` | `GetMaxFearCharacter__7ai_logsFv` | **CONFIRMED** | Worst offender by FEAR (+0x50). Cached at `C_entity+0x127c` |
| 0x00401f10 | `ai_logs_get_max_death` | `GetMaxDeathCharacter__7ai_logsFv` | **CONFIRMED** | Worst by THREAT among **death-marked** logs (`+0x04 == 0x29a`). Returns the RECORD |
| 0x00401fa0 | `ai_logs_get_max_threat_score` | `GetMaxOhrozeni__7ai_logsFv` | **CONFIRMED** | Worst by THREAT among **living** logs (`+0x04 != 0x29a`). Returns the **SCORE** — the only picker that does |
| 0x00402030 | `ai_logs_destroy_log_for_actor` | `Destroy__7ai_logsFP7C_actor` | **CONFIRMED** | Drop one actor's log |
| 0x004020d0 | `ai_logs_reset_stale_logs` | `LooseDistant__7ai_logsFv` | **CONFIRMED** | Wipe the contact window + attitude of any log tripping ≥3 of 4 staleness gates |

### 3a. `ai_character_log` — the record's own methods (7/7)

| PC VA | PC name | PS2 twin (mangled) | Status | Role |
|---|---|---|---|---|
| 0x00401000 | `ai_character_log_new_contact` | `NewContact__16ai_character_logFP7C_actorUi` | **CONFIRMED** | Initialise a fresh log; called on the stack, then copied into the vector |
| 0x00401aa0 | `ai_character_log_event_hit` | `EventHit__16ai_character_logFP12ai_event_hit` | **CONFIRMED** | Target hit ME → `own_damage`; can promote a neutral target to hostile |
| 0x00401b50 | `ai_character_log_event_enemy_hit` | `EventEnemyHit__16ai_character_logFP12ai_event_hit` | **CONFIRMED** | Target hit MY ENEMY → `enemy_damage`; never touches `relation` |
| 0x00401bc0 | `ai_character_log_event_friend_hit` | `EventFriendHit__16ai_character_logFPf` | **CONFIRMED** | Target hit MY FRIEND → `friend_damage`; hardens an existing grudge, cannot create one. Reached only through event 1 — see §"Who raises event 1" |
| 0x00401c50 | `ai_character_log_event_visual_contact` | `EventVisualCon__16ai_character_logFb` | **CONFIRMED** | Sight contact; 20000 ms window, and contact LOSS shortens rather than closes |
| 0x00401cd0 | `ai_character_log_event_sound_contact` | `EventSoundCon__16ai_character_logFv` | **CONFIRMED** | Sound contact; 10000 ms window — half of sight's |
| - | - (inlined) | `__as__16ai_character_logFRC16ai_character_log` | **INLINED** | Compiler-generated `operator=` ¹ |

¹ The PS2 body is a flat copy of **exactly 0x15 dwords (0x54 bytes)**, byte-for-byte the
`for (i = 0x15; i != 0; i--)` dword loop MSVC emits **inline** at every PC site that copies a log —
the vector insert, `ai_logs_destroy_log_for_actor`'s shift-down erase, and `ai_logs_tick`'s swap-pop.
No standalone PC function performs this copy.

---

## 4. How the system behaves

1. **Creation.** `ai_logs_event` is the only way a log is born. On first contact it computes a
   **relationship class 0..3** from the owner's AI mode crossed with the target's kind, and that class
   is fixed for the life of the log — it selects which row of `g_ai_logs_exposure_table` (§4c) the
   threat blend uses.
2. **Accumulation.** Nine event ids feed the log: three damage channels (hit me / hit my enemy / hit
   my friend), two contact channels (sight / sound), a friend-damage bump, a force-hostile flag, an
   escalation broadcast, and the death event.
3. **Perception refresh.** `ai_logs_tick` re-polls the three senses at most every 250 ms and stamps
   `last_seen` / `last_heard` / `last_smelled`, plus the two derived stamps `last_aimed` (visible AND
   armed) and `last_shot` (heard AND fired within 250 ms).
4. **Scoring.** `ai_logs_compute_attitude` runs on every live log every tick and produces four numbers:
   `attitude` (morale-shaped), `threat` (the exposure blend), `anger` (aggression-driven) and `fear`
   (low-morale and own-wounds-driven). **Anger and fear come from different personality stats** — an
   aggressive NPC escalates, a low-morale or wounded one panics.
5. **Selection.** The four max-pickers hand the AI the current worst offender per axis;
   `entity_switch_ai_state` turns those into a fight state, and `ai_fight_modes` scores the tactic.
6. **Decay.** `ai_logs_reset_stale_logs` wipes a log's accumulated state once three of four staleness
   gates fire, but **keeps the log** — the NPC still recognises the actor, it just stops holding a
   grudge. `ai_logs_tick` and `ai_logs_destroy_log_for_actor` do the actual removals.

### 4a. The stride-0x54 insert helper is shared

The PC's stride-0x54 `std::vector::insert` helper **`vector_insert_stride_0x54`** (`0x00561170`) is an
identical-COMDAT fold: its three callsites cover two unrelated element types — the ai_logs log vector
(`ai_logs_event`, `entity_load_logs`) and a **savegame slot descriptor** (31-char name at record
`+0x10`, slot index at `+0x08`, written alongside `savegame/mafia%03d.sav`). Its name therefore stays
mechanical; a name after `ai_character_log` would be false for the savegame caller.

### 4b. Who raises event 1 — the `ai_group` event bus

`ai_character_log_event_friend_hit` has no direct caller: it is reached only as
`ai_logs_event`'s `case 1`, and the chain that gets there starts one subsystem away.

1. `ai_group_push_event(caster, subject, damage)` appends an `ai_group_event` to the group's
   queue. The `float` third argument lands at `event+0x08`; `structs/ai_group.md` §1e has the
   record.
2. `ai_group_process_events` drains the queue and, for each live entry, calls
   `entity_dispatch_event(subject, 1, &event->damage)` on every OTHER member of the group.
3. `entity_dispatch_event` forwards EVERY event to `ai_logs_event` before reaching its own
   switch, and its own `case 1` is an explicit no-op. So the entire effect of a kind-1 group
   event is the `ai_logs` arm.
4. `ai_logs_event`'s `case 1` dereferences the data pointer as a `float` and hands it to
   `ai_character_log_event_friend_hit`, which ADDS it to that subject's `friend_damage`,
   pushes `expire_ts` to now + 30000 ms and bumps `+0x04`.

So `friend_damage` is not accumulated by anything watching a hit directly — it is accumulated
because a GROUP MEMBER broadcast that it was attacked, and every other member books the damage
against the attacker. That is also why the function "hardens an existing grudge but cannot
create one": a bystander who never had a log for the attacker gets no new record out of a
friend's report. The three producers pass `1.0f`, `50.0f` and `100.0f`.

### 4c. The exposure table — `g_ai_logs_exposure_table` (`0x0063b260`, `.rdata`)

Four `{rest, aiming, shooting}` float rows, one per relationship class. `ai_logs_compute_attitude`
splits the log's exposure into three shares, each clamped at 0: **shooting** = the shooting weight
(`last_shot` recency) × `attitude`; **aiming** = the aiming weight (`last_aimed` recency) ×
`attitude` × (1 − shooting); **rest** = 1 − shooting − aiming. It multiplies each share by its
column of row `relation`, stores the sum to `threat` (`+0x48`), and scales it by the owner's tuning
float at `C_entity+0xf34`.

| `relation` | `rest` | `aiming` | `shooting` |
|---|---|---|---|
| 0 | 0.0 | 0.0 | 0.0 |
| 1 | 0.75 | 1.0 | 1.25 |
| 2 | 0.5 | 0.75 | 1.0 |
| 3 | 0.0 | 0.25 | 0.5 |

A class-0 log never builds threat, whatever the target does; class 1 weighs every share the most,
and class 3 ignores the remainder and discounts aiming and shooting.

---

## 5. PC vs PS2

- **`ai_logs` PS2 base:** `C_entity + 0x1060` (PC `+0xe88`), delta `0x1d8`.
- **The record layout has ZERO delta** — every offset in §1a is identical on both builds. Only the
  *container header* differs: PC holds a `std::vector` begin/end pair at `+0x24/+0x28` and computes
  `(end - begin) / 0x54`; PS2 holds a count at `+0x24` and a data pointer at `+0x28` and reads the
  count directly. The owner sits at PS2 `+0x2c` vs PC `+0x30`.
- **Container lever:** `insert__…<16ai_character_log,…>` and `erase__…<16ai_character_log,…>` in the
  PS2 symbol table **name the element type** of the PC vector at `C_entity+0xeac`; the PC decomp
  shows only "0x54-byte rows".
- **Emission order:** `ai_logs` and `ai_character_log` interleave in PS2 VA order, i.e. they come from
  the **same translation unit** — which is why both classes share one PC cluster and why the gap
  between `ai_logs_tick` and `ai_logs_get_log_for_actor` is a forced 10-for-10 match.
- **Factoring differences:** PC inlines `std::vector::erase` into `ai_logs_destroy_log_for_actor`
  (making it *larger* than its twin, 160 B vs 116 B — the one inverted size ratio); PC inlines
  `operator=`; PC inlines the `memset` and the `clear__vector_deleter<>` that `Init` calls on PS2
  (MSVC CRT intrinsics).
- **The exposure table** is PS2 `EXPOSURE_TABLE` (`undefined1[48]`, 4 × 3 floats); `ComputeAttitude`
  reads it in the same statement, row `relation * 0xc`, with the same column-to-share mapping.
- **`ai_logs_event`'s relationship matrix differs in two cells.** PS2 handles owner modes `1` and
  `2` explicitly; the PC build folds them into the default arm. Both builds agree on modes 0 and 3–6.
- **No PS2-only methods in either class.** All 14 `ai_logs` methods and 6 of 7 `ai_character_log`
  methods have PC twins; the seventh is the inlined `operator=` (§3a).
- **`vector_insert_stride_0x54`'s PS2 twin is [STRUCTURAL]**, not CONFIRMED: PS2 emits one
  instantiation per element type, PC folded them (§4a), so the mapping is one-to-many on the PC side.

---

## 6. Cross-references

- `structs/entity.md` §6c — the C_entity AI sub-object map (`ai_logs` @ `+0xe88`).
- `structs/ai_sensors.md` — the "can I perceive X?" sibling that feeds this class's timestamps.
- `structs/ai_character.md` — the stat block `ai_logs_compute_attitude` reads.
- `structs/ai_fight_modes.md` — `ai_logs_event` credits/debits it directly; `+0xf64`.

---

## 7. Open items

| item | state |
|---|---|
| the `ai_logs` header `+0x00..+0x23` | undecoded — `ai_logs_init` zeroes it wholesale and no method in the class reads it; likely a base or the vector's allocator block |
| what each relationship class `0..3` is | the exposure rows (§4c) rank them 1 > 2 > 3 > 0 and class 0 never builds threat; naming them needs `ai_logs_event`'s owner-mode × target-kind matrix decoded cell by cell |
| `ai_logs_event`'s two divergent matrix cells | PC folds owner modes 1 and 2 into the default arm (§5); the gameplay effect is not established |
| the target's `+0x1e5` armed byte | used by `ai_logs_tick`'s `last_aimed` gate; not documented in `structs/human.md` |
