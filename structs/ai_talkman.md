# `ai_talkman` — the NPC speech scheduler

**Created** 2026-07-26 22:47 +0200 · **Modified** 2026-09-30 23:08 +0200

Static analysis of `Game-ghidra.exe.annotated.c`, cross-checked against the non-stripped PS2 ELF
and the shipped `tables\voices.def`. All cross-build material is in §5; the coverage table in §4
is a PC↔PS2 twin table.

`ai_talkman` decides **when an AI person says something out loud**. It is a four-slot scheduler:
callers bind a *talk group* to a slot with `ai_talkman_say`, and the per-frame tick fires lines from
due slots through `human_new_scream`, re-scheduling each with a randomised delay. `entity_ai` calls
its tick **×28**.

- **PC prefix:** `ai_talkman_*`.
- **PC location:** embedded in every `C_entity` at **`+0x10cc`**, 0x98 bytes, so it ends exactly where
  the `ai_action_manager` at `+0x1164` begins (`structs/entity.md` §6c). Byte-proven by 22
  `lea ecx,[reg+0x10cc]` receivers in six functions (`entity_action_tick` ×12, `entity_ai`,
  `entity_dispatch_event`, `ai_action_new_crouch_fight`, `ai_action_new_normal_fight`,
  `ai_vis_logs_tick`, `entity_cb_hit_info`) and by `entity_game_init`, which initialises it with the
  entity itself as the owner. It is therefore **per person**.
- **Coverage:** 4/4 CONFIRMED, every row body-diffed on both builds.

**Confidence tags:** **[H]** writer and reader both read · **[M]** inferred · **[U]** not observed.

---

## 1. Field layout (PC offsets, relative to the `ai_talkman` base)

```c
typedef struct ai_talkman {          // at C_entity+0x10cc, 0x98 bytes
    /* +0x00 */ uint8_t  enabled;    // [H] ai_talkman_init sets 1. ai_talkman_tick plays no line
                                     //     while this is 0 - the mute for this person.
    /* +0x04 */ talk_slot slots[4];  // [H] four scheduler slots, stride 0x24, +0x04..+0x93:
                                     //     +0x04 / +0x28 / +0x4c / +0x70.
    /* +0x94 */ C_entity *owner;     // [H] stored by ai_talkman_init, passed to human_new_scream.
} ai_talkman;                        // 0x98
```

### 1a. `talk_slot` — the scheduler slot (stride `0x24`)

Offsets below are **relative to the slot**; add `0x04 + N*0x24` for the absolute offset.

```c
typedef struct talk_slot {            // 0x24 bytes
    /* +0x00 */ uint32_t flags;       // [H] bit map below
    /* +0x04 */ uint32_t group;       // [H] talk group, an index into g_ai_talk_line_groups (§3);
                                      //     also the key ai_talkman_say matches slots on.
    /* +0x08 */ uint32_t last_line;   // [H] the line played last; the tick re-rolls once if the new
                                      //     draw repeats it.
    /* +0x0c */ uint32_t delay;       // [H] repeat delay in ms.
    /* +0x10 */ uint32_t jitter;      // [H] jitter window in ms; the tick schedules
                                      //     next_due = now + delay + rand(jitter) - jitter/2.
    /* +0x14 */ uint32_t last_spoke;  // [H] mission clock when the slot last fired.
    /* +0x18 */ uint32_t next_due;    // [H] mission clock at which the slot becomes eligible.
    /* +0x1c */ uint32_t percent;     // [H] fire probability 0..100; the tick draws rand(101) and
                                      //     stays silent when the draw is >= this. Init seeds 100.
    /* +0x20 */ uint32_t priority;    // [H] eviction key (lowest is evicted), see §2a.
} talk_slot;                          // 0x24
```

| flag | meaning | set by | cleared by |
|---|---|---|---|
| `0x01` | armed | `ai_talkman_say` (stores 9 or 0xb) | `ai_talkman_tick` after a one-shot fires |
| `0x02` | one-shot | `ai_talkman_say` (`flags & 0x02`), `ai_talkman_tick` (demotion) | `ai_talkman_say` when it refreshes a demoted slot |
| `0x08` | requested this frame | `ai_talkman_say`, on every request | `ai_talkman_begin_frame` |
| `0x20` | demoted by the tick | `ai_talkman_tick`: `flags \|= 0x22` when neither `0x08` nor `0x02` is set | `ai_talkman_say` when it refreshes the slot |

`entity_ai` clears `0x08` on every slot, then the callers request, then the tick runs. So a
**repeating line keeps repeating only while its caller requests it every frame**: once the requests
stop, the tick demotes the slot to one-shot, it fires at most once more and then disarms.

---

## 2. `ai_talkman_say` — the only entry point

`ai_talkman_say(this, group, delay, jitter, first, spread, percent, flags)`, `ret 0x1c`:

| argument | meaning |
|---|---|
| `group` | talk group (§3), matched against slot `+0x04` |
| `delay` / `jitter` | repeat delay and centred jitter window, both ms, stored at slot `+0x0c` / `+0x10` |
| `first` | delay before the first line, ms: slot `+0x18` = now + `first` |
| `spread` | under flags bit `0x04` the first due time grows by `rand_range(spread) + 2*spread` |
| `percent` | fire probability 0..100, slot `+0x1c` |
| `flags` | `0x02` one-shot · `0x04` randomise the first delay · `0x10` re-bind even when already bound · `0x40` re-bind only when the bound slot is disarmed. No other bit is tested; shipped callers pass `0`, `2`, `0x42` and `0x46` |

**Slot choice.** The slot already bound to `group` is refreshed (`0x08` set, a demoted slot restored to
repeating) and re-bound only as `flags` bits `0x10` / `0x40` allow. Otherwise the first disarmed slot is
taken; with all four armed, the slot with the lowest priority is evicted, but only when that priority is
`<=` column 0 of `g_game_voice_table` row `group - 2` (`-1` when the row does not exist).

### 2a. The priority word

Column 0 of each `voices.def` row is the line's **speech priority**: `human_new_scream` interrupts the
current line only when the new line's priority is higher than the stored one (`human+0x6d0`).
`ai_talkman_say` reads that column with the **talk-group id** minus 2 — the arithmetic
`human_new_scream` applies to a **line id** — so the eviction threshold of group *g* is the priority of
`voices.def` row *g − 2*, not of the group's own lines (§3 lists both). The stored slot priority is that
value on an eviction and the **flags word** on the refresh and free-slot paths.

---

## 3. Talk groups

`g_ai_talk_line_groups` (`0x00646420`) is a statically initialised `S_ai_talk_entry[20]` of 0xc-byte
records `{group, first_line, line_count}` in which each record's `group` equals its index.
`ai_talkman_tick` picks a random line in `[first_line, first_line + line_count)` and passes it to
`human_new_scream`, which plays `voices.def` row `line - 2` in the speaker's voice bank
(`human+0x680`, one of 40 columns; file name `%08lu.wav` from the cell).

The situations below are read from the requesting code (the spoken words: §7). Delays are
`delay +- jitter/2`; "idle-only" = flags bit `0x40`.

| group | lines | `voices.def` rows | line priority | banks | eviction key | requested by, in which situation, and how |
|---|---|---|---|---|---|---|
| 0 | 21..23 | 19..21 | 45 | 20 | -1 | fight start (`ai_action_new_crouch_fight`, `_normal_fight`) when the target is visible and the person is armed: one-shot, 20 % · normal fight, on the state 9 → state 1 transition: one-shot, 25 % · normal fight, state 10 fast path: one-shot after 0.5 s, 25 %, idle-only |
| 1 | 37..41 | 35..39 | 50 | 19 | -1 | fight start when the target is visible (armed or not): one-shot, 20 % |
| 2 | 45..50 | 43..48 | 60 | 25 | 50 | crouch and normal fight while the target is seen: every 20 s +-5 s, first after 5 s, 25 % |
| 3 | 15..16 | 13..14 | 0 | 14 | 50 | `ai_vis_logs_tick`, for an actor in the visual logs without an `ai_logs` record (or with a log whose `+0x04` is not `0x29a`) while the person's AI state (`entity+0xf40`, the 10000 and `0x29a` offsets removed) is neither 3 nor 4: one-shot, 50 % |
| 4 | 33..36 | 31..34 | 40 | 18 | 50 | crouch and normal fight when `entity+0x11e0 > 0.98`: one-shot after 0.4 s, 5 % |
| 5 | 6..8 | 4..6 | 50 | 17 | 50 | `entity_dispatch_event` for type groups 3..6 with a resolved `ai_group`, AI state not 4: one-shot after 5-7 s, 75 %, idle-only |
| 6 | 9..11 | 7..9 | 50 | 15 | 50 | no requester |
| 7 | 2..5 | 0..3 | 50 | 20 | 50 | no requester |
| 8 | 70..74 | 68..72 | 50 | 18 | 50 | normal fight, the fall-through of states 9 and 10: every 12 s +-2.5 s, first after 4 s or 2 s, 50 % |
| 9 | 75..78 | 73..76 | 40 | 16 | 50 | normal fight, right after the fight-mode scores are reset (`ai_fight_modes_finish_mode`): one-shot, 25 % |
| 0xa | 66..69 | 64..67 | 85 | 27-28 | 50 | `entity_ai`, AI state 4 with `entity+0xf44 == 0` — the reaction that queues a run-away job from `entity+0x127c`: every 10 s +-2.5 s, first after 4 s, 75 % |
| 0xb | 61..65 | 59..63 | 70 | 23 | 50 | `entity_ai`, AI state 4 with `entity+0xf44 == 1` — the reaction that queues a freeze job of at least 20 s: every 5 s +-2.5 s, first after 1 s, 60 % · `entity_dispatch_event` for type group 1: one-shot after 1.5-2 s, 50 %, idle-only |
| 0xc | 79..83 | 77..81 | 40 | 19 | 50 | `entity_cb_hit_info` when the attacker's `ai_logs` record has `+0x08 == 1` and the hit value (`param_2`) exceeds 25.0: one-shot after 1 s, 30 %, idle-only |
| 0xd | 94..96 | 92..94 | 90 | 32 | 50 | no requester |
| 0xe | 150..151 | 148..149 | 70 | 2 | 50 | police action (`0x21`): a suspect leaving a car within 3.5 m, at most every 5 s: one-shot, always, idle-only |
| 0xf | 139..144 | 137..142 | 70 | 2 | 0 | police action: a throttled (4 s + random) one-in-N roll: one-shot, always, idle-only |
| 0x10 | 145..146 | 143..144 | 76 | 2 | 0 | police action, unarmed suspect: one-shot after 1 s, always, idle-only |
| 0x11 | 147..149 | 145..147 | 79 | 2 | 60 | police action, armed suspect: one-shot after 1 s, always, idle-only |
| 0x12 | 152..154 | 150..152 | 70-72 | 1-2 | 60 | no requester |
| 0x13 | 155..157 | 153..155 | 73-75 | 1-2 | 60 | no requester |

`banks` counts the speaker columns with a file for the group's lines. Groups `0xe..0x13` exist only in
banks 10 and 11 (files `21940xxx` / `22940xxx`), which agrees with `0xe..0x11` being requested only by
the police action. Groups 6, 7, `0xd`, `0x12` and `0x13` are in the table and in `voices.def` but no
code requests them.

---

## 4. Method family — PC ↔ PS2 (4 methods)

Ordered by PC VA. Every row body-diffed on both builds.

| PC VA | PC name | PS2 twin (mangled) | Status | Role |
|---|---|---|---|---|
| 0x00439ce0 | `ai_talkman_tick` | `Tick__10ai_talkmanFv` | CONFIRMED | Per-frame: demote unrequested repeating slots, test armed-and-due, draw a line (re-rolling one repeat), play it via `human_new_scream` behind the enabled + rand(101) gates, then disarm a one-shot or reschedule. `entity_ai` calls it ×28 |
| 0x00439dd0 | `ai_talkman_say` | `Say__10ai_talkmanFUiUiUiUiUiUiUi` | CONFIRMED | The only entry point, §2 |
| 0x00439f10 | `ai_talkman_init` | `Init__10ai_talkmanFPv` | CONFIRMED | Enable, store the owner at `+0x94`, clear all four slots, seed `percent` = 100 |
| 0x00439f50 | `ai_talkman_begin_frame` | `NewFrame__10ai_talkmanFv` | CONFIRMED | Clear the requested-this-frame bit on all four slots |

---

## 5. PC vs PS2

- **PS2 location:** `C_entity + 0x12b0` (`Tick__10ai_talkmanFv(this + 0x12b0)` inside `AI__8C_entity`).
  PC `+0x10cc`: the delta `0x1e4` is the one the neighbouring substate-lock band and the
  `ai_action_manager` show (`structs/entity.md`).
- **The slot layout has ZERO delta** — every offset in §1a is identical on both builds, and the slot
  stride is 0x24 on both. PS2's `NewFrame` unrolls its four writes at the literal offsets
  `+0x04 / +0x28 / +0x4c / +0x70`, which is exactly `+0x04 + N*0x24`, while PC loops four times at
  stride 0x24.
- **One behavioural difference in `Say`:** on the refresh and free-slot paths PS2 stores priority **0**,
  where the PC stores the flags word (the register that holds it is reused). Every other statement,
  including the `0x29a` search seed, the voice-table eviction key and the `2*spread` term, matches.
- **Rendering differences:** PC folds PS2's `(x & 8) == 0 && (x & 2) == 0` into the single equivalent
  `(x & 0xa) == 0`; PS2 unrolls loops PC keeps rolled. PS2 indexes the talk table through the base
  symbol `ai_talk_table`, the counterpart of PC `g_ai_talk_line_groups`.
- **No PS2-only methods.** All four have PC twins.
- **`NewScream`** is the voice-playback call on both builds and is **not** limited to screaming — it
  plays any voice line. Its PC name `human_new_scream` keeps the developers' word.

---

## 6. Related

- `structs/entity.md` §6c — the C_entity AI sub-object map.
- `structs/ai_action_manager.md` — the fight and police actions that request most groups.
- `basic/artifacts/npc-player-interactions.md` — `voices.def` line groups and the brush-off/greeting
  behaviour of the pedestrian voice path.

---

## 7. Open items

| item | state |
|---|---|
| the spoken words of each talk group (§3) | not transcribed and not verified in game |
