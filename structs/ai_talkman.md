# `ai_talkman` — the NPC speech scheduler

**Created** 2026-07-26 22:47 +0200 · **Modified** 2026-09-17 13:43 +0200

Static analysis of `Game-ghidra.exe.annotated.c`, cross-checked against the non-stripped PS2 ELF.
All cross-build material is in §4; the coverage table in §3 is a PC↔PS2 twin table.

`ai_talkman` decides **when an AI person says something out loud**. It is a small four-slot scheduler:
callers bind a *talk group* to a slot with `ai_talkman_say`, and the per-frame tick fires lines from
due slots through `human_new_scream`, re-scheduling each with a randomised delay. `entity_ai` calls
its tick **×28**.

- **PC prefix:** `ai_talkman_*`.
- **PC location:** owner offset not traced (§6). The object holds its owner at `+0x94` and passes it
  to `human_new_scream`, so the owner is a `C_human`-family object.
- **Coverage:** 4/4 CONFIRMED, every row body-diffed on both builds.

**Confidence tags:** **[H]** writer and reader both read · **[M]** inferred · **[U]** not observed.

---

## 1. Field layout (PC offsets, relative to the `ai_talkman` base)

```c
typedef struct ai_talkman {          // owner offset untraced (§6)
    /* +0x00 */ uint8_t  enabled;    // [H] ai_talkman_init sets 1. ai_talkman_tick refuses to play
                                     //     any line while this is 0 - the global mute for this person.
    /* +0x04 */ talk_slot slots[4];  // [H] four scheduler slots, stride 0x24, spanning +0x04..+0x93.
                                     //     Slot N is at +0x04 + N*0x24, i.e. +0x04 / +0x28 / +0x4c /
                                     //     +0x70.
    /* +0x94 */ C_human *owner;      // [H] stored by ai_talkman_init, passed to human_new_scream.
} ai_talkman;                        // size >= 0x98
```

### 1a. `talk_slot` — the scheduler slot (stride `0x24`)

Offsets below are **relative to the slot**; add `0x04 + N*0x24` for the absolute offset.

```c
typedef struct talk_slot {            // 0x24 bytes
    /* +0x00 */ uint32_t flags;       // [H] bit 0 (0x01) = slot enabled/armed
                                      //     bit 1 (0x02) = one-shot: tick clears bit 0 after firing
                                      //     bit 3 (0x08) = already spoke this frame; cleared by
                                      //                    ai_talkman_begin_frame, set by ai_talkman_say
                                      //     bit 5 (0x20) = re-arm marker; tick ORs in 0x22 when both
                                      //                    bit 3 and bit 1 are clear
    /* +0x04 */ uint32_t group;       // [H] talk-group id; indexes the 0xc-stride global talk table.
                                      //     Also the key ai_talkman_say matches slots on.
    /* +0x08 */ uint32_t last_line;   // [H] the line played last time; tick re-rolls once if the new
                                      //     draw repeats it, so a slot never says the same thing twice
                                      //     in a row.
    /* +0x0c */ uint32_t base_delay;  // [H] base gap in ms before the slot is due again.
    /* +0x10 */ uint32_t jitter;      // [H] randomisation window; tick schedules
                                      //     next_due = now + base_delay + rand(jitter) - jitter/2,
                                      //     i.e. the jitter is CENTRED on the base delay.
    /* +0x14 */ uint32_t last_spoke;  // [H] mission clock when the slot last fired.
    /* +0x18 */ uint32_t next_due;    // [H] mission clock at which the slot becomes eligible.
    /* +0x1c */ uint32_t percent;     // [H] fire probability 0..100; tick draws rand(101) and stays
                                      //     silent when the draw is >= this. ai_talkman_init seeds
                                      //     100 (always speak).
    /* +0x20 */ uint32_t priority;    // [H] eviction key: ai_talkman_say picks the slot with the
                                      //     LOWEST value here, seeded from 0x29a (666, the project
                                      //     "none" sentinel) so any real slot beats "no slot".
} talk_slot;                          // 0x24
```

The `say` flag word (8th parameter of `ai_talkman_say`): bit `0x10` = re-trigger, bit `0x40` =
re-trigger only when idle; the other bits are not decoded (§6).

---

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | enabled | H | `ai_talkman_init` (→1) | `ai_talkman_tick` | global mute for this person |
| slot +0x00 | flags | H | `ai_talkman_init`, `ai_talkman_say`, `ai_talkman_begin_frame`, `ai_talkman_tick` | all four | bit map in §1a |
| slot +0x04 | group | H | `ai_talkman_init`, `ai_talkman_say` | `ai_talkman_tick` (table index), `ai_talkman_say` (match key) | - |
| slot +0x08 | last_line | H | `ai_talkman_tick` | `ai_talkman_tick` (repeat test) | - |
| slot +0x0c / +0x10 | base_delay / jitter | H | `ai_talkman_init`, `ai_talkman_say` | `ai_talkman_tick` | jitter centred on the delay |
| slot +0x14 / +0x18 | last_spoke / next_due | H | `ai_talkman_init`, `ai_talkman_tick` | `ai_talkman_tick` | mission-clock stamps |
| slot +0x1c | percent | H | `ai_talkman_init` (→100), `ai_talkman_say` | `ai_talkman_tick` | rand(101) gate |
| slot +0x20 | priority | H | `ai_talkman_init`, `ai_talkman_say` | `ai_talkman_say` (victim pick) | 0x29a seed |
| +0x94 | owner | H | `ai_talkman_init` | `ai_talkman_tick` | passed to `human_new_scream` |

**The global talk table** is a 0xc-stride array; `ai_talkman_tick` reads its `+4` and `+8` columns
(PC `DAT_00646424` / `DAT_00646428`) as the line-range base and count for a group. Both globals are
unnamed (§6).

---

## 3. Method family — PC ↔ PS2 (4 methods)

Ordered by PC VA. Every row body-diffed on both builds.

| PC VA | PC name | PS2 twin (mangled) | Status | Role |
|---|---|---|---|---|
| 0x00439ce0 | `ai_talkman_tick` | `Tick__10ai_talkmanFv` | CONFIRMED | Per-frame: re-arm, test due-and-enabled, draw a line (re-rolling one repeat), play it via `human_new_scream` behind the enabled + rand(101) gates, then reschedule. `entity_ai` calls it ×28 |
| 0x00439dd0 | `ai_talkman_say` | `Say__10ai_talkmanFUiUiUiUiUiUiUi` | CONFIRMED | The only entry point: bind a talk group to a slot, or refresh the slot already bound to it; evicts the lowest-priority slot on a miss |
| 0x00439f10 | `ai_talkman_init` | `Init__10ai_talkmanFPv` | CONFIRMED | Enable, store owner at `+0x94`, clear all four slots, seed `percent` = 100 |
| 0x00439f50 | `ai_talkman_begin_frame` | `NewFrame__10ai_talkmanFv` | CONFIRMED | Clear the spoke-this-frame bit on all four slots |

---

## 4. PC vs PS2

- **PS2 location:** `C_entity + 0x12b0` (`Tick__10ai_talkmanFv(this + 0x12b0)` inside `AI__8C_entity`).
  The PC offset is untraced, so **no delta is asserted** — C_entity's deltas vary per band, so the PS2
  offset does not transfer.
- **The slot layout has ZERO delta** — every offset in §1a is identical on both builds, and the slot
  stride is 0x24 on both. PS2's `NewFrame` unrolls its four writes at the literal offsets
  `+0x04 / +0x28 / +0x4c / +0x70`, which is exactly `+0x04 + N*0x24`, while PC loops four times at
  stride 0x24.
- **Rendering differences only, no behavioural ones:** PC folds PS2's `(x & 8) == 0 && (x & 2) == 0`
  into the single equivalent `(x & 0xa) == 0`; PS2 unrolls loops PC keeps rolled; PS2 indexes the talk
  table through one base symbol (`ai_talk_table`) where Ghidra splits the PC side into two column
  symbols (`DAT_00646424` / `DAT_00646428`).
- **No PS2-only methods.** All four have PC twins.
- **`NewScream`** is the voice-playback call on both builds and is **not** limited to screaming — it
  plays any voice line. Its PC name `human_new_scream` keeps the developers' word.

---

## 5. Related

- `structs/entity.md` §6c — the C_entity AI sub-object map (this class is not on it; §6).
- `basic/artifacts/npc-player-interactions.md` — `voices.def` line groups and the brush-off/greeting
  behaviour this scheduler ultimately drives.

---

## 6. Open items

1. **Where does this object live on PC?** No receiver trace has been done: find the `lea ecx,[reg+N]`
   that loads ECX before each call to `ai_talkman_tick` (`0x00439ce0`) — the same method pinned
   `ai_logs`, `ai_sensors`, `ai_character`, `ai_fight_modes` and `ai_action_manager`. Once found, add
   the class to `structs/entity.md` §6c.
2. **Name the talk table.** `DAT_00646424` / `DAT_00646428` are the `+4` / `+8` columns of a 0xc-stride
   global. Naming it (and finding its loader) would connect this class to the voice-line data
   (`voices.def` line groups, `basic/artifacts/npc-player-interactions.md`).
3. **What are the four slots FOR?** Four concurrent talk groups per person suggests a fixed role split
   (idle chatter / combat / pain / interaction), but nothing observed proves it. The callers of
   `ai_talkman_say` pass the group id as a literal — enumerating those literals would answer it.
4. **The `say` flag word.** Bits `0x10` and `0x40` are decoded (§1a); the rest of the 8th parameter is
   not.
5. **Is `ai_talkman` per-person or shared?** It is reached from `entity_ai`, and its owner field is a
   `C_human`, which implies per-person — item 1 settles it.
