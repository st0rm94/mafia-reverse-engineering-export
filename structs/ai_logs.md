# `ai_logs` — the NPC's per-target attitude log

**Created** 2026-07-16 21:10 +0200 · **Modified** 2026-11-03 23:17 +0100

Where `ai_sensors` (`structs/ai_sensors.md`) answers *"can I perceive X?"*, `ai_logs` answers
**"how do I FEEL about X?"** — it keeps one `ai_character_log` per actor the NPC has an opinion about,
scored by fear / anger / threat / death, and hands the AI the current worst offender.
`entity_switch_ai_state` reads those picks to choose a fight state. The file documents two
classes: the container `ai_logs`, embedded in `C_entity` at `+0xe88` (byte-proven:
`lea ecx,[esi+0xe88]` at all 20 callsites of `ai_logs_tick`), and its element `ai_character_log`.

| | |
|---|---|
| PC prefix | `ai_logs_`, `ai_character_log_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `ai_logs` `0x34` (the next `C_entity` member, `ai_character`, starts at `C_entity+0xebc` = `0xe88 + 0x34`); `ai_character_log` `0x54` |
| PC constructor | - |
| PC translation unit | `[0x00401000, 0x00402290)` (§5) |
| PS2 class | `ai_logs` (tag `7ai_logs`), 14 methods, and `ai_character_log` (tag `16ai_character_log`), 7 methods: 20 CONFIRMED twins in §3, the inlined `operator=` in §6 |

**Confidence tags:** **[H]** writer and reader both read · **[M]** inferred from one side · **[U]** not observed.

## 1. Base field layout

### 1a. `ai_logs`

PC offsets, relative to the `ai_logs` base.

```c
typedef struct ai_logs {                    // 0x34 bytes, embedded at C_entity+0xe88
    /* +0x00 */ uint32_t unread_00[8];         // [U] zeroed by ai_logs_init, saved and restored by the savegame pair; no reader (class census)
    /* +0x20 */ uint8_t  _alloc;               // [H] the vector's MSVC allocator byte
    /* +0x24 */ ai_character_log *begin;       // [H] the LOG VECTOR begin = C_entity+0xeac
    /* +0x28 */ ai_character_log *end;         // [H] one past the last log; count = (end-begin)/0x54
    /* +0x2c */ ai_character_log *cap;         // [H] the vector's capacity end
    /* +0x30 */ C_entity *owner;               // [H] owner backpointer, stored by ai_logs_init
} ai_logs;
```

The vector object starts at `+0x20`: `ai_logs_event` and `entity_load_logs` pass `this+0x20` to the
insert helper, which keeps begin / end / capacity at its own `+0x04` / `+0x08` / `+0x0c`. The vector
is freed by `vector_dispose_at_0x24`, an identical-COMDAT fold shared with `ai_loc`'s own `+0x24`
vector, so it belongs to neither class; the `ai_logs` ownership is byte-proven by this owner's two
unwind funclets, `0x00635136` and `0x006351a1`, both `add ecx, 0xe88`.

### 1b. `ai_character_log` — the element type (stride `0x54`)

```c
typedef struct ai_character_log {           // 0x54 bytes; element of the ai_logs vector
    /* +0x00 */ C_actor *target;               // [H] who this log is about; the vector's scan key
    /* +0x04 */ uint32_t event_count;          // [H] +1 per contact opened; 0x29a = death-marked
    /* +0x08 */ uint32_t relation;             // [H] relationship class 0..3 (§4d)
    /* +0x0c */ float    enemy_damage;         // [H] damage this target dealt to MY ENEMIES
    /* +0x10 */ float    own_damage;           // [H] damage this target dealt to ME
    /* +0x14 */ float    friend_damage;        // [H] damage this target dealt to MY FRIENDS
    /* +0x18 */ uint32_t contact_ts;           // [H] contact-window open stamp; 0 = no live contact
    /* +0x1c */ uint32_t expire_ts;            // [H] contact-window expiry deadline
    /* +0x20 */ uint32_t event_ts;             // [H] last event stamp
    /* +0x24 */ uint32_t last_seen;            // [H] ai_logs_tick, when ai_sensors_can_see fires
    /* +0x28 */ uint32_t last_heard;           // [H] ai_logs_tick, when ai_sensors_can_hear fires
    /* +0x2c */ uint32_t last_smelled;         // [H] ai_logs_tick, when ai_sensors_can_smell fires; no reader but the savegame
    /* +0x30 */ uint32_t last_aimed;           // [H] visible AND the target aiming; 6250 ms weight
    /* +0x34 */ uint32_t last_shot;            // [H] heard AND the target fired < 250 ms ago
    /* +0x38 */ uint32_t next_poll_ts;         // [H] sense re-poll gate, pushed +250 ms
    /* +0x3c */ float    attitude_bias;        // [H] input bias: the damage balance ai_logs_tick writes each tick (§2b)
    /* +0x40 */ float    attitude_ema;         // [H] smoothed attitude
    /* +0x44 */ float    attitude;             // [H] morale-shaped attitude, clamped at 0
    /* +0x48 */ float    threat;               // [H] exposure-weighted threat
    /* +0x4c */ float    anger;                // [H] (aggression + 0.5) scaled
    /* +0x50 */ float    fear;                 // [H] (1 - morale) scaled
} ai_character_log;
```

**Two independent derivations agree on the stride and the extent:** every scan in the class
advances by `0x54`, and `ai_character_log_new_contact` writes every dword index `0 .. 0x14` except
index 8 (`event_ts`), which `ai_logs_event` stamps right after the find-or-create. **`+0x24` of
`ai_logs` is doubly confirmed:** `ai_logs_init` empties the vector there, and `structs/entity.md`
independently documents the list at `C_entity+0xeac` — `0xe88 + 0x24 = 0xeac`.

## 2. Field evidence

Measured by a census of every `this+N` access in the `ai_logs_*` and `ai_character_log_*` methods
and in the vector helpers, plus a read of each `ai_logs_*` body for the accesses it makes through a
log pointer.

### 2a. `ai_logs`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | unread_00 | U | `ai_logs_init` (= 0), `entity_load_logs` | `entity_save_logs` | 8 dwords, saved and loaded as a block; no other reader (class census) |
| +0x20 | _alloc | H | `entity_ctor` (the byte store) | - | the MSVC allocator member; its address is the vector object `ai_logs_event` and `entity_load_logs` pass to `vector_insert_stride_0x54` |
| +0x24 | begin | H | `vector_insert_stride_0x54` (on reallocation) | every `ai_logs_*` scan, `entity_save_logs` | = `C_entity+0xeac`; elements are `ai_character_log` (§1b) |
| +0x28 | end | H | `vector_insert_stride_0x54`, `ai_logs_init` (= begin), `ai_logs_tick` (swap-pop), `ai_logs_destroy_log_for_actor`, `ai_logs_clear_all_logs` (= begin), `entity_load_logs` | every `ai_logs_*` scan, `entity_save_logs` | - |
| +0x2c | cap | H | `vector_insert_stride_0x54` | `vector_insert_stride_0x54` | the capacity end, touched only by the insert helper |
| +0x30 | owner | H | `ai_logs_init` | `ai_logs_event`, `ai_logs_compute_attitude`, `ai_logs_reset_stale_logs` | the owning `C_entity`; how the class reaches morale/aggression, the frame and `ai_fight_modes` |

**Serialized size is VARIABLE** (`entity_get_logs_size` loops the list) — unlike the other three
`C_entity` sub-blocks, whose sizes are constants (`structs/entity.md` §1). It is `0x24` — a 4-byte
count and the 0x20-byte header — plus `0x70` per log whose target is not save-excluded
(`C_actor+0x65`): a 0x20-byte target name and 20 dwords.

### 2b. `ai_character_log`

"The five events" are `ai_character_log_event_hit`, `_enemy_hit`, `_friend_hit`,
`_visual_contact` and `_sound_contact`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | target | H | `ai_character_log_new_contact` | the scans of `ai_logs_event`, `ai_logs_has_log`, `ai_logs_get_log_for_actor`, `ai_logs_destroy_log_for_actor`; `ai_logs_tick` (its alive byte), `ai_logs_reset_stale_logs` (its frame), `entity_save_logs` (its name), `entity_switch_ai_state`, `entity_ai`, `human_is_hit_allowed_between_types` | the vector's scan key |
| +0x04 | event_count | H | `ai_character_log_new_contact` (= 0), the five events (+1 when they open a contact window), `ai_logs_event` (the death event, = `0x29a`), `ai_logs_tick` (`0x29a` on a dead target), `entity_load_logs` | the five events, the four `ai_logs_get_max_*` pickers, `ai_logs_tick`, `ai_vis_logs_tick`, `entity_save_logs` | `0x29a` marks a death-witnessed log; `ai_logs_get_max_death` takes only those and `ai_logs_get_max_threat_score` only the others, so the two partition the vector |
| +0x08 | relation | H | `ai_character_log_new_contact` (the class `ai_logs_event` computes), `ai_character_log_event_hit`, `ai_character_log_event_friend_hit` (both promote to 1), `ai_logs_event` (event `0x14`, = 1) | `ai_character_log_event_hit`, `ai_character_log_event_friend_hit`, `ai_logs_compute_attitude`, `ai_logs_has_log` (its optional output), `ai_logs_event`, `entity_switch_ai_state`, `entity_cb_hit_info`, `entity_force_ai`, `entity_hit`, `ai_vis_logs_tick`, `entity_beware_cross_fire` | selects the row of `g_ai_logs_exposure_table` (§4c) |
| +0x0c | enemy_damage | H | `ai_character_log_new_contact` (= 0), `ai_character_log_event_enemy_hit` (accumulates), the other four events (= 0 when they open a contact) | `ai_character_log_event_enemy_hit`, `ai_logs_tick` | with `own_damage`, the balance `ai_logs_tick` writes to `attitude_bias` |
| +0x10 | own_damage | H | `ai_character_log_new_contact` (= 0), `ai_character_log_event_hit` (accumulates), the other four events (= 0 when they open a contact) | `ai_character_log_event_hit`, `ai_logs_tick` | with `enemy_damage`, the balance `ai_logs_tick` writes to `attitude_bias` |
| +0x14 | friend_damage | H | `ai_character_log_new_contact` (= 0), `ai_character_log_event_friend_hit` (adds the reported damage), `ai_logs_event` (+5.0; event 7 +500 for a kind-`0x15` target), `ai_vis_logs_tick` (+50 for a corpse's killer seen less than 3 s or heard less than 2 s ago), `entity_ai` (+20), `ai_logs_tick` and `ai_logs_reset_stale_logs` (= 0) | `ai_character_log_event_friend_hit`, `ai_logs_event`, `ai_logs_compute_attitude` | folded into anger at 0.05 weight; its producer is the `ai_group` event bus (§4b) |
| +0x18 | contact_ts | H | `ai_character_log_new_contact` (= 0), the five events (= mission clock on open), `ai_logs_event` (= `event_ts`), `ai_logs_tick` (= clock, or 0 on expiry), `ai_logs_reset_stale_logs` (= 0) | the five events (the open test), the four `ai_logs_get_max_*` pickers, `ai_logs_tick`, `entity_beware_cross_fire`, `entity_action_tick` (case `0x1c`) | 0 = no live contact; every picker requires it |
| +0x1c | expire_ts | H | `ai_character_log_new_contact` (= 0), the five events, `ai_logs_tick`, `ai_logs_reset_stale_logs` (= 0) | `ai_character_log_event_visual_contact`, `ai_character_log_event_sound_contact`, `ai_logs_tick` | the window is 30 000 ms for the enemy-hit and friend-hit events, 20 000 ms for sight and 10 000 ms for sound |
| +0x20 | event_ts | H | `ai_logs_event` (= mission clock) | `ai_logs_event` | copied into `contact_ts` by the arms that open a contact; the only field `ai_character_log_new_contact` does not write |
| +0x24 | last_seen | H | `ai_character_log_new_contact` (= 0), `ai_logs_tick` | `ai_logs_reset_stale_logs`, `ai_vis_logs_tick`, `entity_ai`, `ai_fight_manager_tick`, `police_board_update_sighting`, `police_board_has_recent_sighting`, `entity_action_tick` (cases 6, `0x1c`, `0x1e`, `0x21`, `0x23`) | one of the staleness gates of `ai_logs_reset_stale_logs`: more than 15 000 ms ago |
| +0x28 | last_heard | H | `ai_character_log_new_contact` (= 0), `ai_logs_tick` | `ai_logs_reset_stale_logs`, `ai_vis_logs_tick`, `entity_ai` | the gate of `ai_logs_reset_stale_logs`: more than 5000 ms ago |
| +0x2c | last_smelled | H | `ai_character_log_new_contact` (= 0), `ai_logs_tick` | `entity_save_logs` | no reader but the savegame, over the class, the callers of the five log getters, the log cached in the type-`0x1c` action's payload and the seven log lookups of `entity_action_tick`; the smell gate of `ai_logs_reset_stale_logs` asks `ai_sensors_get_smelled_target_info` instead |
| +0x30 | last_aimed | H | `ai_character_log_new_contact` (= 0), `ai_logs_tick` | `ai_logs_get_aiming_weight`, `entity_beware_cross_fire` | visible AND the target's aimed-stance flag `C_human+0x1e5` |
| +0x34 | last_shot | H | `ai_character_log_new_contact` (= 0), `ai_logs_tick` | `ai_logs_get_shooting_weight`, `entity_beware_cross_fire`, `ai_fight_manager_tick`, `entity_action_tick` (case `0x21`) | heard AND the target fired less than 250 ms ago; in case `0x21`, a shot less than 2000 ms ago raises the gun escalation |
| +0x38 | next_poll_ts | H | `ai_character_log_new_contact` (= mission clock), `ai_logs_tick` (= clock + 250) | `ai_logs_tick` | the 250 ms sense re-poll gate |
| +0x3c | attitude_bias | H | `ai_character_log_new_contact` (= 0), `ai_character_log_event_visual_contact` and `ai_character_log_event_sound_contact` (= `attitude_ema` on open), `ai_logs_tick` (each tick of a live target: `(own_damage − enemy_damage) × 10 /` the ms since `contact_ts`, 10 when that is 0) | `ai_logs_compute_attitude`, `ai_logs_reset_stale_logs` | the sign picks the attitude-shaping branch; the copy from `attitude_ema` holds until the next tick |
| +0x40 | attitude_ema | H | `ai_character_log_new_contact` (= 0), `ai_logs_reset_stale_logs` (= `(3*bias + ema) * 0.25`), `ai_logs_tick` (the same average at window expiry) | `ai_character_log_event_visual_contact`, `ai_character_log_event_sound_contact`, `ai_logs_reset_stale_logs`, `ai_logs_tick` | - |
| +0x44 | attitude | H | `ai_character_log_new_contact` (= 0), `ai_logs_compute_attitude`, `ai_logs_tick` and `ai_logs_reset_stale_logs` (= 0) | `ai_logs_compute_attitude`, `ai_character_log_event_hit` | a hit on a class-0 log creates a grudge only above 1.25 / 1.125 (§4d) |
| +0x48 | threat | H | `ai_character_log_new_contact` (= 0), `ai_logs_compute_attitude`, `ai_logs_event` (the death event: 2 / 0 / 1), `ai_logs_tick` and `ai_logs_reset_stale_logs` (= 0) | `ai_logs_get_max_death`, `ai_logs_get_max_threat_score`, `entity_ai`, `ai_logs_compute_attitude`, `entity_switch_ai_state`, `ai_action_new_hostile_coverage`, `entity_action_tick` (case `0x24`: the target's weight in `ai_space_coverage_prefer_dir`) | what both threat pickers rank; the third scoring input of `ai_fight_modes_tick` |
| +0x4c | anger | H | `ai_character_log_new_contact` (= 0), `ai_logs_compute_attitude`, `ai_logs_tick` and `ai_logs_reset_stale_logs` (= 0) | `ai_logs_get_max_angry`, `entity_ai`, `entity_switch_ai_state`, `entity_action_tick` (case `0x1c`) | `(aggression + 0.5) * (friend_damage * 0.05 + threat)`, scaled by `C_entity+0xf1c` |
| +0x50 | fear | H | `ai_character_log_new_contact` (= 0), `ai_logs_compute_attitude`, `ai_logs_tick` and `ai_logs_reset_stale_logs` (= 0) | `ai_logs_get_max_fear`, `entity_ai`, `entity_switch_ai_state`, `entity_action_tick` (case `0x1c`) | `2 * (1 − morale) * ((1 − health fraction) * 0.5 + threat)`, scaled by `C_entity+0xf28` |

## 3. Method family

Coverage: all **14** PS2 `7ai_logs` methods and **6 of the 7** `16ai_character_log` methods are in
the tables below, **20 CONFIRMED twins**, every row body-diffed on both builds; the seventh,
`operator=`, is inlined at three sites (§6). Two PC functions have no PS2 twin:
`ai_logs_clear_all_logs` and `ai_character_log_copy_range`.

### 3a. Construction & destruction

Neither class has a constructor or destructor of its own: `ai_logs_init` and
`ai_character_log_new_contact` (§3c) initialise them.

### 3b. Accessors

None.

### 3c. Behaviour

#### `ai_logs`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00401060 | `ai_logs_init` | `Init__7ai_logsFPv` | 0x0030c700 | CONFIRMED | owner backpointer, empty the vector, zero the header |
| 0x004010b0 | `ai_logs_tick` | `Tick__7ai_logsFv` | 0x0030c740 | CONFIRMED | per-frame maintenance: evict dead or expired logs, stamp the per-sense timestamps behind a 250 ms poll gate, then `ai_logs_compute_attitude` on every live log |
| 0x00401440 | `ai_logs_event` | `Event__7ai_logsFP7C_actorUiPv` | 0x0030cbb0 | CONFIRMED | 🔑 **the only creator** — find-or-create a log, stamp `event_ts`, then route 9 event ids to the five `ai_character_log_event_*` mutators |
| 0x00401810 | `ai_logs_get_aiming_weight` | `GetAimingWeight__7ai_logsFP16ai_character_log` | 0x0030d1c0 | CONFIRMED | recency weight of `last_aimed`: 1.25 decaying over 6250 ms; takes the log, not the container |
| 0x00401880 | `ai_logs_get_shooting_weight` | `GetShootingWeight__7ai_logsFP16ai_character_log` | 0x0030d260 | CONFIRMED | recency weight of `last_shot`: 1.25 decaying over 25 000 ms; takes the log |
| 0x004018f0 | `ai_logs_compute_attitude` | `ComputeAttitude__7ai_logsFP16ai_character_log` | 0x0030d300 | CONFIRMED | 🔑 **the scorer** — morale/aggression + the two weights + damage → attitude/threat/anger/fear |
| 0x00401d20 | `ai_logs_has_log` | `hasLog__7ai_logsFP7C_actorPUi` | 0x0030d920 | CONFIRMED | membership test; optionally outputs the relationship class |
| 0x00401d90 | `ai_logs_get_log_for_actor` | `GetLog__7ai_logsFP7C_actor` | 0x0030d9a0 | CONFIRMED | find the log for one actor |
| 0x00401df0 | `ai_logs_get_max_angry` | `GetMaxAngryCharacter__7ai_logsFv` | 0x0030da00 | CONFIRMED | worst offender by ANGER (`+0x4c`); cached at `C_entity+0x1280` |
| 0x00401e80 | `ai_logs_get_max_fear` | `GetMaxFearCharacter__7ai_logsFv` | 0x0030dab0 | CONFIRMED | worst offender by FEAR (`+0x50`); cached at `C_entity+0x127c` |
| 0x00401f10 | `ai_logs_get_max_death` | `GetMaxDeathCharacter__7ai_logsFv` | 0x0030db60 | CONFIRMED | worst by THREAT among **death-marked** logs (`+0x04 == 0x29a`); returns the RECORD |
| 0x00401fa0 | `ai_logs_get_max_threat_score` | `GetMaxOhrozeni__7ai_logsFv` | 0x0030dc10 | CONFIRMED | worst by THREAT among **living** logs (`+0x04 != 0x29a`); returns the **SCORE** — the only picker that does |
| 0x00402030 | `ai_logs_destroy_log_for_actor` | `Destroy__7ai_logsFP7C_actor` | 0x0030dcd0 | CONFIRMED | drop one actor's log |
| 0x004020d0 | `ai_logs_reset_stale_logs` | `LooseDistant__7ai_logsFv` | 0x0030dd50 | CONFIRMED | wipe the contact window and the attitude of any log tripping ≥ 3 of 4 staleness gates — seen > 15 s, heard > 5 s, smelled > 10 s, farther than 200 units |
| 0x005c8f70 | `ai_logs_clear_all_logs` | - | - | - | empties the log vector (`end = begin`) and leaves the header untouched. One caller: `GROUP_DISBAND` phase 2 (`program_process_command` case `0x17b`), for every member of the group, next to `ai_sensors_init` and `ai_vis_logs_init` ¹ |

¹ No PS2 method corresponds: the PS2 `GROUP_DISBAND` clears the vector directly (§6).

#### `ai_character_log`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00401000 | `ai_character_log_new_contact` | `NewContact__16ai_character_logFP7C_actorUi` | 0x0030c6a0 | CONFIRMED | initialise a fresh log on the stack (it is then copied into the vector): target, class, every other field 0 except `next_poll_ts` = mission clock, and `event_ts` left unwritten |
| 0x00401aa0 | `ai_character_log_event_hit` | `EventHit__16ai_character_logFP12ai_event_hit` | 0x0030d5d0 | CONFIRMED | the target hit ME → `own_damage`; can promote a neutral target to hostile |
| 0x00401b50 | `ai_character_log_event_enemy_hit` | `EventEnemyHit__16ai_character_logFP12ai_event_hit` | 0x0030d6d0 | CONFIRMED | the target hit MY ENEMY → `enemy_damage`; never touches `relation` |
| 0x00401bc0 | `ai_character_log_event_friend_hit` | `EventFriendHit__16ai_character_logFPf` | 0x0030d750 | CONFIRMED | the target hit MY FRIEND → `friend_damage`; hardens an existing grudge, cannot create one. Reached only through event 1 (§4b) |
| 0x00401c50 | `ai_character_log_event_visual_contact` | `EventVisualCon__16ai_character_logFb` | 0x0030d800 | CONFIRMED | sight contact; 20 000 ms window, and contact LOSS shortens rather than closes it |
| 0x00401cd0 | `ai_character_log_event_sound_contact` | `EventSoundCon__16ai_character_logFv` | 0x0030d8b0 | CONFIRMED | sound contact; 10 000 ms window — half of sight's |

### 3d. Free helpers

#### `ai_character_log`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043afb0 | `ai_character_log_copy_range` | - | - | - | copies `[first, last)` of logs to a destination, 21 dwords each — the copy step of the log vector's reallocation. Emitted in the `C_entity` unit (`structs/entity.md` §8); its one caller is `entity_game_init`, on `C_entity+0xeac`. GCC inlines the copy, so there is no PS2 counterpart |

## 4. How the system behaves

1. **Creation.** `ai_logs_event` is the only way a log is born. On first contact it computes a
   **relationship class 0..3** from the owner's type group crossed with the target (§4d). A hit, a
   friend's reported hit or event `0x14` can change it later — it selects which row of
   `g_ai_logs_exposure_table` (§4c) the threat blend uses.
2. **Accumulation.** Nine event ids feed the log: three damage channels (hit me / hit my enemy / hit
   my friend), two contact channels (sight / sound), a friend-damage bump, a force-hostile flag, an
   escalation broadcast, and the death event.
3. **Perception refresh.** `ai_logs_tick` re-polls the three senses at most every 250 ms and stamps
   `last_seen` / `last_heard` / `last_smelled`, plus the two derived stamps `last_aimed` (visible AND
   aiming) and `last_shot` (heard AND fired within 250 ms).
4. **Scoring.** `ai_logs_compute_attitude` runs on every live log every tick and produces four
   numbers: `attitude` (morale-shaped), `threat` (the exposure blend), `anger` (aggression-driven)
   and `fear` (low morale and the owner's own wounds). **Anger and fear come from different
   personality stats** — an aggressive NPC escalates, a low-morale or wounded one panics.
5. **Selection.** The four max-pickers hand the AI the current worst offender per axis;
   `entity_switch_ai_state` turns those into a fight state, and `ai_fight_modes` scores the tactic.
6. **Decay.** `ai_logs_reset_stale_logs` wipes a log's accumulated state once three of four
   staleness gates fire, but **keeps the log** — the NPC still recognises the actor, it just stops
   holding a grudge. `ai_logs_tick` and `ai_logs_destroy_log_for_actor` do the actual removals.

### 4a. The stride-0x54 insert helper is shared

The PC's stride-0x54 `std::vector::insert` helper **`vector_insert_stride_0x54`** (`0x00561170`) is
an identical-COMDAT fold: its three callsites cover two unrelated element types — the `ai_logs` log
vector (`ai_logs_event`, `entity_load_logs`) and a **savegame slot descriptor** (31-char name at
record `+0x10`, slot index at `+0x08`, written alongside `savegame/mafia%03d.sav`). Its name
therefore stays mechanical; a name after `ai_character_log` would be false for the savegame caller.

### 4b. Who raises event 1 — the `ai_group` event bus

`ai_character_log_event_friend_hit` has no direct caller: it is reached only as
`ai_logs_event`'s `case 1`, and the chain that gets there starts one subsystem away.

1. `ai_group_push_event(caster, subject, damage)` appends an `ai_group_event` to the group's
   queue. The `float` third argument lands at `event+0x08`; `structs/ai_group.md` §1d has the
   record.
2. `ai_group_process_events` drains the queue and, for each live entry, calls
   `entity_dispatch_event(subject, 1, &event->damage)` on every OTHER member of the group.
3. `entity_dispatch_event` forwards EVERY event to `ai_logs_event` before reaching its own
   switch, and its own `case 1` is an explicit no-op. So the entire effect of a kind-1 group
   event is the `ai_logs` arm.
4. `ai_logs_event`'s `case 1` dereferences the data pointer as a `float` and hands it to
   `ai_character_log_event_friend_hit`, which ADDS it to that subject's `friend_damage`,
   pushes `expire_ts` to now + 30 000 ms and bumps `event_count`.

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

### 4d. The relationship matrix — how a log gets its class

`ai_logs_event` picks the class from the owner's type group (`C_entity+0xf4c`, `structs/entity.md`
§2c) and a target selector: `0` for the player (kind 2), the target's own `+0xf4c` group for a
`C_entity` (kind `0x1b`), and for any other kind the target pointer itself, which is out of range
and takes the default arm. Decoded from the five jump tables at `0x0040173c`..`0x004017ac`:

| Owner group | Target 0 | 1 | 2 | 3 | 4 | 5 | 6 | Any other kind |
|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 3 | 3 | 1 | 2 | 2 | 3 |
| 1 | 3 | 3 | 3 | 3 | 3 | 3 | 3 | 3 |
| 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| 3 | 3 | 3 | 0 | 0 | 3 | 3 | 3 | 3 |
| 4 | 1 | 3 | 2 | 3 | 0 | 1 | 2 | 3 |
| 5 | 3 | 3 | 2 | 3 | 1 | 0 | 2 | 3 |
| 6 | 2 | 3 | 2 | 3 | 2 | 2 | 0 | 3 |

An owner group above 6 gives class 3 for every target. Who is in each group, measured from the
shipped scenes (group 1 the civilians, group 2 the police, and so on), is in `structs/entity.md`
§2c.

What the classes do, measured: **class 0** builds no threat (§4c) and sits on the diagonal of every
row but 1 and 2. **Class 1** is the top tier: event `0x14` forces it, and a hit
(`ai_character_log_event_hit`) or a friend's reported hit (`ai_character_log_event_friend_hit`)
promotes a class 2 or 3 log to it. A hit on a class 0 log creates a grudge only when the log's
attitude exceeds `1.25` (class 1) or `1.125` (class 2). **Class 3** is the default for every pair
the matrix does not list.

## 5. The translation unit

`[0x00401000, 0x00402290)` is the first unit of `.text`: it has no static initializer of its own,
holds the 20 `ai_logs_*` / `ai_character_log_*` methods `0x00401000`–`0x004020d0` with the two
classes interleaved and nothing else, and the next unit's guard, `crt_static_init_guard_00402290`,
closes it (that unit holds `ai_check_builder`, `structs/ai_check.md`). Two functions of these
classes sit elsewhere: `ai_character_log_copy_range` in the `C_entity` unit and
`ai_logs_clear_all_logs` in the band of the `C_program` unit (`structs/program.md`).

## 6. PC vs PS2

- **`ai_logs` PS2 base:** `C_entity + 0x1060` (PC `+0xe88`), delta `0x1d8`.
- **The record layout has ZERO delta** — every offset in §1b is identical on both builds. Only the
  *container header* differs: PC holds a `std::vector` begin/end pair at `+0x24/+0x28` and computes
  `(end - begin) / 0x54`; PS2 holds a count at `+0x24` and a data pointer at `+0x28` and reads the
  count directly. The owner sits at PS2 `+0x2c` vs PC `+0x30`.
- **Container lever:** `insert__…<16ai_character_log,…>` and `erase__…<16ai_character_log,…>` in the
  PS2 symbol table **name the element type** of the PC vector at `C_entity+0xeac`; the PC decomp
  shows only "0x54-byte rows".
- **Emission order:** `ai_logs` and `ai_character_log` interleave in PS2 VA order, i.e. they come
  from the **same translation unit** — which is why both classes share one PC cluster and why the
  gap between `ai_logs_tick` and `ai_logs_get_log_for_actor` is a forced 10-for-10 match.
- **Factoring differences:** PC inlines `std::vector::erase` into `ai_logs_destroy_log_for_actor`
  (making it *larger* than its twin, 160 B vs 116 B — the one inverted size ratio); PC inlines
  `operator=`; PC inlines the `memset` and the `clear__vector_deleter<>` that `Init` calls on PS2
  (MSVC CRT intrinsics).
- **The exposure table** is PS2 `EXPOSURE_TABLE` (`undefined1[48]`, 4 × 3 floats); `ComputeAttitude`
  reads it in the same statement, row `relation * 0xc`, with the same column-to-share mapping.
- **`ai_logs_event`'s relationship matrix differs from the PC's (§4d) in three ways.** PS2 gives a
  target in group 1 class 0 for every owner, where the PC gives 3 for owners 1 and 3–6. PS2's
  owner-1 row gives groups 1 and 2 class 0 (PC: 3 throughout), and its owner-2 row gives the player
  class 2 and every other target class 0 (PC: 2 throughout). And PS2 creates **no log** for a
  target that is neither the player nor a `C_entity`: its selector starts at `0x29a` and the event
  returns, where the PC's selector is the target pointer and the default arm logs it. Every other
  cell of owner rows 0 and 3–6 agrees.
- **The header is 0x20 bytes on both builds.** PS2 `Init` does `memset(this, 0, 0x20)`, keeps its
  vector at `+0x20` and `SaveLogs` writes the same 8 dwords at `ai_logs+0x00`.
- **`last_aimed` reads the same aimed stance flag**, PS2 `C_human+0x391` (PC `+0x1e5`).
- **`ai_logs_clear_all_logs` has no PS2 counterpart method.** At the same point of `GROUP_DISBAND`
  the PS2 calls `clear__Q23std74__vector_deleter<>Fv` on `ai_logs+0x20` itself; the PC emits a
  one-line `ai_logs` function for it.
- **`vector_insert_stride_0x54`'s PS2 twin is [STRUCTURAL]**, not CONFIRMED: PS2 emits one
  instantiation per element type, PC folded them (§4a), so the mapping is one-to-many on the PC side.

**The PS2-only method.**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0030e0d0 | `__as__16ai_character_logFRC16ai_character_log` | the compiler-generated `operator=`: the PS2 body is a flat copy of exactly 0x15 dwords (0x54 bytes), and MSVC emits the same `for (i = 0x15; i != 0; i--)` dword loop **inline** at every PC site that copies a log — the vector insert, `ai_logs_destroy_log_for_actor`'s shift-down erase and `ai_logs_tick`'s swap-pop |

## 7. Related

- `structs/entity.md` §7 — the `C_entity` AI sub-object map (`ai_logs` at `+0xe88`); §2c — the
  type group of §4d.
- `structs/ai_sensors.md` — the "can I perceive X?" sibling that feeds this class's timestamps.
- `structs/ai_character.md` — the stat block `ai_logs_compute_attitude` reads.
- `structs/ai_fight_modes.md` — `ai_logs_event` credits and debits it directly; `+0xf64`.
- `structs/ai_group.md` — the event bus of §4b.

## 8. Open items

None.
