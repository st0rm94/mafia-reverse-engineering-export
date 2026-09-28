# Police Offences — storage, bit mapping, and on-screen text

**Created** 2026-06-03 16:00 +0200 · **Modified** 2026-09-18 10:32 +0200

This document covers: how an offence is recorded, how `get_pm_state` reads it, how the police
come to *notice* it, and how the offence text reaches the HUD. It is built from static analysis
of `Game-ghidra.exe.annotated.c`, cross-checked against the Mafia Script Data Reference and
against in-game observation with a debug script and live memory inspection. Everything is
anchored by function name, VA or bit value.

---

## 1. The three distinct enumerations

The police code uses **three different numbering schemes** for "what offence happened". Keeping
them apart is the key to the whole subsystem:

| Enum | Where it lives | Values | Purpose |
|---|---|---|---|
| **offence type** (`zz`) | bit position in `offense_bits` (`1 << type`) | 1..13 (Mafia Script Data Reference) | what `get_pm_state zz` queries |
| **action_code** | `pursuit_record + 0x64` | 0..10 | picks the on-screen string (`+ 0x834`) |
| **police-state level** | `pursuit_record + 0x3c` (committed) / `+0x3e` (pending) | 1=fine, 2=handcuffs, 3=gun | the HUD icon + escalation |

They are **not** the same numbers. Example: a fired shot sets offence-type bit `1<<1` (`zz`=1),
writes `action_code` = 1 (string "Shots fired!"), and raises police-state to 3 (gun). A fine for
a traffic sign sets bit `1<<12` (`zz`=12), `action_code` = 8 ("traffic sign damaged"), and
police-state 1 (fine).

---

## 2. Storage: `bit = 1 << offence_type`

Each pursued target owns a heap `pursuit_record` (0x80 bytes; see `structs/police_manager.md`
§4). Offences are stored as **two adjacent `uint16_t` bitfields**:

- `offense_bits_a` at `pursuit_record + 0x08`
- `offense_bits_b` at `pursuit_record + 0x0a`

A committed offence sets **bit `1 << offence_type`** (`offence_type` = the `zz` code of the Mafia
Script Data Reference). Reader `police_manager_get_offense_bits_for_target` (VA `0x005ce4c0`)
returns **`bits_a | bits_b`**.

Both `bits_a` (`+0x08`) and `bits_b` (`+0x0a`) are 16-bit words: bits 0–7 live in the low byte
(`+0x08` / `+0x0a`), bits 8–15 in the high byte (`+0x09` / `+0x0b`).

### 2.1 Per-bit meaning found in the decomp (evidence) — source of truth

What the code actually writes into each bit of the `bits_a` word. **This is the byte-traced reality,
cross-checked against gameplay — the source of truth**. `+0x08` = low byte, `+0x09` =
high byte (one shared word — there is no "separate band"). The `set` / `clear` columns give the
**full function name** (+ the bit op) as it appears in the annotated source (no VA); the
`live flag / decay window` column says whether the bit is a per-frame live flag (re-derived every
inner tick, e.g. speeding / weapon-seen) or sticky with an associated decay timer.
All bits are **in-game confirmed** with `get_pm_state` plus live memory reads.

| word bit | mask | byte | offence | set | clear | live flag / decay window | Notes |
|---|---|---|---|---|---|---|---|
| 0 | `0x0001` | `+0x08` | weapon equipped | `police_manager_tick` inner-tick mirror (`+8 \|= 1` iff equipped-weapon-id `+0x4a0` ∈ [5,15]) | `police_manager_tick` (`+8 &= 0xFE` iff `+0x4a0` ∉ [5,15]) | **live flag** — re-derived each inner tick, no decay | guns + throwables (predmety.def ids 5–15); melee/tools do NOT set it; See §2.4 |
| 1 | `0x0002` | `+0x08` | shooting | `police_manager_register_gunfire_offence_for_target` (`+8 \|= 2`) | `police_manager_tick` (`+8 &= 0xFD` on `+0x10` expiry) | decay `+0x10` (≤60 s) | — |
| 2 | `0x0004` | `+0x08` | speeding | `police_manager_tick` (`+8 \|= 4` when speed > fine threshold `+0x0c`) | `police_manager_tick` (`+8 &= 0xFB` when speed ≤ `+0x0c` / target gone) | **live flag** — re-derived each inner tick, no decay | above ~17 m/s |
| 3 | `0x0008` | `+0x08` | car contact | `police_manager_register_collision_offence_for_target` (`+8 \|= 0x08`, + `+0xa` mirror) | `police_manager_tick` (`+8 &= 0xC7` on `+0x18`; `+0xa &= 0xC7` on `+0x1c`) | decay `+0x18` (≤4 s) + `+0x1c` (≤20 s) | two paths: destroyable prop → fine / static world → silent. See §2.2, §2.3 |
| 4 | `0x0010` | `+0x08` | car crash | `police_manager_register_collision_offence_for_target` (`+8 \|= 0x10`, + `+0xa` mirror) | `police_manager_tick` (`+8 &= 0xC7` on `+0x18`; `+0xa &= 0xC7` on `+0x1c`) | decay `+0x18` (≤4 s) + `+0x1c` (≤20 s) | escalation → §2.3 |
| 5 | `0x0020` | `+0x08` | run over pedestrian | `police_manager_register_collision_offence_for_target` (`+8 \|= 0x20`, + `+0xa` mirror) | `police_manager_tick` (`+8 &= 0xC7` on `+0x18`; `+0xa &= 0xC7` on `+0x1c`) | decay `+0x18` (≤4 s) + `+0x1c` (≤20 s) | escalation → §2.3 |
| 6 | `0x0040` | `+0x08` | driving on sidewalk | `police_manager_register_sidewalk_offence_for_target` (`+8 \|= 0x40`) | `police_manager_tick` (`+8 &= 0xBF` on leaving surface / speed ≤ 11) | on-sidewalk timer `+0x14` (clamp 5 s); cleared on leaving surface | surface id 3; print **unreachable** (§8) |
| 7 | `0x0080` | `+0x08` | unused | — | — | — | — |
| 8 | `0x0100` | `+0x09` | unused | — | — | — | — |
| 9 | `0x0200` | `+0x09` | violence | `police_manager_register_violence_offence_for_target` (`+9 \|= 2`) | `police_manager_tick` (`+9 &= 0xFD` on `+0x20` expiry) | decay `+0x20` (violence-event window) | melee / violence vs any pedestrian incl. cops; gunfire does NOT (bit 1) |
| 10 | `0x0400` | `+0x09` | running a red light | `police_manager_tick` red-light block (`+9 \|= 4`) | `police_manager_tick` (`+9 &= 0xEB` on `+0x24` expiry — shared w/ bit 12) | decay `+0x24` (≤10 s) | not an offence type in the Mafia Script Data Reference |
| 11 | `0x0800` | `+0x09` | car theft | `police_manager_set_car_theft_offence_for_target` (`+9 \|= 8`) | `police_manager_tick` (`+9 &= 0xF7` on `+0x0c` expiry) | decay `+0x0c` (car-theft window) | — |
| 12 | `0x1000` | `+0x09` | traffic sign damaged | `police_manager_register_traffic_sign_offence_for_target` (`+9 \|= 0x10`) | `police_manager_tick` (`+9 &= 0xEB` on `+0x24` expiry — shared w/ bit 10) | decay `+0x24` (≤20 s) | car-hit or gunfire both set it |
| 13 | `0x2000` | `+0x09` | assault on a police officer | `police_manager_register_assault_on_police_officer_for_target` (`+9 \|= 0x20`) | `police_manager_record_clear_pursuit_state` (full record reset; survives dormant reset) | **no decay** — persists until record reset | `get_pm_state zz=13` reads spurious (§9) |
| 14 | `0x4000` | `+0x09` | force handcuffs | `police_manager_escalate_handcuffs_with_optional_wanted` (`+9 \|= 0x40`) | `police_manager_record_clear_pursuit_state` (full record reset; survives dormant reset) | **no decay** — persists until record reset | script-only (`POLICEMANAGER_FORCEARREST`) |
| 15 | `0x8000` | `+0x09` | assault on a police car | `police_manager_register_assault_on_police_car_for_target` (`+9 \|= 0x80`) | `police_manager_try_dispatch_pursuit_for_target` (`+9 &= 0x7f`) | **transient** — consumed on the next dispatch pass (same frame) | also sets `+0x6c` severe-impact flag (severity > 9.7) |

Notes:
- **The `+0x09` high byte is one field**, shared between the get_pm_state bits (9/11/12) and the
  escalation-only bits (13/14/15) — not two separate fields.
- **Bits 7, 8 have no writer** — in-game `get_pm_state zz=7`/`zz=8` never returned 1 (confirms unused).
- `+0x6c` (set by `police_manager_register_assault_on_police_car_for_target`) is the **severe-impact** flag
  (`impact-severity > 9.7`)
- **Bit 15 is a transient "alert-pending / needs-dispatch" bit, not a persistent offence.** The
  assault-car registrar sets it (`+9 \|= 0x80`); the very next `police_manager_try_dispatch_pursuit_for_target`
  pass uses it (dispatch nearby cops + print the red line) and then **consumes** it (`+9 &= 0x7f`, which clears
  **only** bit 15). Confirmed with a live value-watch: `+0x09` reads 0 even right
  after ramming/punching a cop car, because the bit is set and cleared within one frame. Contrast
  bits **9** (violence) and **13** (assault-officer): the `& 0x7f` clear leaves them intact, so they
  **persist** in a value-watch — punching a *cop* shows `0x2200` (bits 9+13), punching a *cop car*
  shows nothing, same dispatch mechanism, different bit.
- **Bit 6's writer chain.** `police_manager_register_sidewalk_offence_for_target` (VA `0x005ce150`)
  has the sole caller `car_ai_tick` (VA `0x00466840`), which scans the vehicle's wheel/collision
  contact list (`+0xd24`, count `+0x5b0`); a contact with surface-material id 3 sets bit `0x40` plus
  the on-surface flag `vehicle+0x20b0`. Surface id 3 = **sidewalk** per `tables\materials.def`
  (`def-files-docs/materialy.md`). `Game.exe` only does the numeric `== 3` compare.
- **Bit 13's writer** `police_manager_register_assault_on_police_officer_for_target` is
  `__thiscall(police_manager, target)`; its two NPC move/tick callsites fire only when the involved
  cop NPC's state byte `+0x15a` is 1 or 3 (officer-attacked flag), which is what makes the offence
  "assault on an officer" → action 9, pending gun.
- **Bit 15's writer** `police_manager_register_assault_on_police_car_for_target` takes
  `(police_manager, target, payload, severity ≥ 2.8)`, sets the severe-impact flag
  `+0x6c = (9.7 < severity)` and the payload `+0x60`; its callsite scans `g_traffic_car_registry`
  for the matching cop-in-vehicle entry → action 10.

### 2.2 `register collision offense` origin → bit

`police_manager_register_collision_offence_for_target` (VA `0x005cde10`) is the central collision/contact
handler. The bit is chosen by the **type tag of the object hit** (`*(*(origin + 0x44) + 0x10)`):

- **vehicle** (type 4) → `0x10` crash (`zz`=4)
- **character** (type 2 / 0x13 / 0x1b) → `0x20` run-over (`zz`=5)
- **destroyable prop** (any *other* struck type — trash can / barrel / hydrant / phone booth /
  box) → `0x08` contact (`zz`=3), via the `switch` **DEFAULT** branch — **escalates** (fine)
- **no struck entity** (`param_3 == NULL`, or origin without a struck object — road edges, tram
  tracks, scenery) → `0x08` contact (`zz`=3), but the **SILENT** branches — bit only

It writes `action_code` = 3 ("crash" — the shared collision display string) **only on the
escalating branches**: run-over, crash, and the **destroyable-prop DEFAULT branch**. The SILENT
no-struck-entity branches set bit 3 but write **no** action_code and do **not** escalate (which
is why constant road-edge / tram-track contact is ignored). The chosen bit is written into
**both** `bits_a` **and** `bits_b` (the long-decay mirror — §3). Severity (the `param_4` velocity
proxy) decides whether `police_state_pending` (`+0x3e`) escalates to 1/2/3.

Hitting a **destroyable object** (trash can / barrel / hydrant / phone booth / box) **gives a fine**
— these fall into the DEFAULT branch (struck type ≠ vehicle/character), which sets bit 3 **and**
escalates (pending=fine VA `0x005ce081`, bits_a bit 3 @VA `0x005ce03e`). Only the SILENT
no-struck-entity path (road edges, tram tracks) is ignored.

### 2.3 Escalation map — offence bit → action_code → pending police-state

For each offence bit, this is what it escalates to: the **`action_code`** (display selector;
printed string = `2100 + action_code`, §5.1), the resulting **pending police-state**
`pursuit_record + 0x3e` (**1 = fine, 2 = handcuffs, 3 = gun**), the condition, and the
function(s) in the annotated source that write the `action_code` and the pending state.

Notes:
- **Recording vs noticing (§6).** Setting the bit needs no witness. But cops only **notice** it —
  dispatch at you and print the line — when a cop is **in range** and, **below gun**, also **facing
  you with clear line-of-sight** (§6). At **gun-tier** noticing is **omnidirectional** (no facing,
  no LOS), which is why **shooting** still draws cops that cannot see you: it jumps straight to gun,
  so any cop in range notices regardless of sight. The condition column lists the *escalation*
  trigger; the §6 noticing gate applies on top for every dispatch-printed row.
- **Immediate vs deferred.** "Immediate" registrars write the `action_code` + pending at the event.
  "Deferred" bits are only set by their registrar; `police_manager_tick` re-derives `action_code` +
  pending every inner tick (~3 Hz) from the bit. Where the bit is set by a *different* function than
  the one that escalates, the bit-setter is noted in parentheses.
- One bit gets **multiple rows** when it produces different pending states (e.g. by impact severity)
  or is set in different contexts.

| Offence bit | action_code | Pending state | Condition / escalation trigger | Function(s) — action_code + pending |
|---|---|---|---|---|
| 0 weapon equipped (on foot) | 0 "Weapon seen!" | handcuffs (2) | bit 0 set (gun/throwable drawn, equipped-weapon-id `+0x4a0` ∈ [5,15]) + pending 0; target on foot | `police_manager_tick` |
| 0 weapon equipped (from car window)| 0 "Weapon seen!" | handcuffs (2) | bit 0 set (handgun drawn, equipped-weapon-id `+0x4a0` ∈ [6,9]) + pending 0; target aiming from a car window | `police_manager_tick` |
| 1 shooting | 1 "Shots fired!" | gun (3) | weapon fired; pending < 3. Jumps straight to gun → omnidirectional, no LOS needed (§6) | `police_manager_register_gunfire_offence_for_target` |
| 2 speeding (moderate) | 2 "Speeding!" | fine (1) | speeding bit + live target; speed ≤ arrest threshold (`police_manager+0x10`); pending 0 | `police_manager_tick` |
| 2 speeding (heavy) | 2 "Speeding!" | handcuffs (2) | speeding bit + live target; speed > arrest threshold; pending < 2 | `police_manager_tick` |
| 3 contact — destroyable prop (light) | 3 "Crash!" | fine (1) | struck a destroyable prop (DEFAULT branch); severity > 8.3 (≤ 19.4); pending 0 | `police_manager_register_collision_offence_for_target` |
| 3 contact — destroyable prop (hard) | 3 "Crash!" | handcuffs (2) | destroyable prop; severity > 19.4; pending < 2 | `police_manager_register_collision_offence_for_target` |
| 3 contact — static world | — | — (none) | NULL / no struck entity (road edges, tram tracks, scenery) → **SILENT** branches: bit set, no escalation | — (bit only — `police_manager_register_collision_offence_for_target`) |
| 4 car crash (light) | 3 "Crash!" | fine (1) | struck a vehicle (type 4); severity > 4.2 (≤ 13.9); pending 0 | `police_manager_register_collision_offence_for_target` |
| 4 car crash (hard) | 3 "Crash!" | handcuffs (2) | struck a vehicle; severity > 13.9; pending < 2 | `police_manager_register_collision_offence_for_target` |
| 5 run-over (light) | 3 "Crash!" | fine (1) | struck a character/NPC (type 2/0x13/0x1b); severity > 2.8 and < 8.3; run_over_count < 6; pending < 3 | `police_manager_register_collision_offence_for_target` |
| 5 run-over (hard) | 3 "Crash!" | handcuffs (2) | character; severity ≥ 8.3; run_over_count < 6; pending < 3 | `police_manager_register_collision_offence_for_target` |
| 5 run-over (repeat ≥6) | 3 "Crash!" | gun (3) | character; run_over_count ≥ 6 (over-escalation cap); pending < 3 | `police_manager_register_collision_offence_for_target` |
| 6 sidewalk | 4 "Driving on sidewalk!" | fine (1) | sidewalk bit + on-sidewalk timer `+0x14` > 5800 ms; pending 0. **UNREACHABLE** — timer hard-clamped to 5000 (§8) | `police_manager_tick` |
| 9 violence | 5 "Violence seen!" | handcuffs (2) | melee / punch hits a pedestrian; pending < 2 | `police_manager_register_violence_offence_for_target` |
| 10 running a red light | 6 "Running red lights!" | fine (1) | crosses a controlled road-edge while signal `g_traffic_light_states[idx] == 1` (first sighting); pending 0 | `police_manager_tick` (red-light / road-edge block) |
| 11 car theft | 7 "Car theft!" | handcuffs (2) | car-theft (arrest-assist) bit `0x08@+9` set; pending < 2 | `police_manager_tick` (bit set by `police_manager_set_car_theft_offence_for_target`) |
| 12 traffic-sign damage | 8 "Traffic sign damaged!" | fine (1) | traffic-sign damage (car-hit **or** gunfire); pending 0 | `police_manager_register_traffic_sign_offence_for_target` |
| 13 assault on a police officer | 9 "Assault on Police!" | gun (3) | assault-on-officer bit `0x20@+9` set; pending < 3 | `police_manager_tick` (bit set by `police_manager_register_assault_on_police_officer_for_target`) |
| 14 force handcuffs | — | handcuffs (2) | force-handcuffs bit `0x40@+9` set (script `POLICEMANAGER_FORCEARREST`); pending < 2 | `police_manager_tick` (bit set by `police_manager_escalate_handcuffs_with_optional_wanted`) — no action_code |
| 15 assault on a police car (light) | 10 "Assault on Police car!" | fine (1) | assault-on-car bit `0x80@+9` set; **not** severe (`+0x6c` == 0); pending 0 | `police_manager_tick` (bit set by `police_manager_register_assault_on_police_car_for_target`) |
| 15 assault on a police car (severe) | 10 "Assault on Police car!" | handcuffs (2) | assault-on-car bit set; **severe** (`+0x6c` set, severity > 9.7); pending < 2 | `police_manager_tick` (bit set by `police_manager_register_assault_on_police_car_for_target`) |

(Bits 7 and 8 are unused — no writer, no escalation.) The `action_code` arbitration when several
offences land together (which string wins) is the monotonic pending gate — see §5.3.

### 2.4 Bit 0 — weapon equipped

`get_pm_state zz=0` returns **1 while the player has a gun or throwable weapon drawn**, 0 otherwise.
Per the §4 model this is **bit 0 of `record+0x08`** (the offence-bits low byte) — real, stored,
queryable state.

**Writer: `police_manager_tick`** (VA `0x005cb510`), in its inner-tick (~3 Hz) per-suspect loop
(set/clear at VA `0x005cbce0`). Bit 0 is a **live mirror** of the suspect's **equipped-weapon-id**
`+0x4a0`: set when the id is in the range **`[5, 15]`** (inclusive), cleared otherwise. That range is
exactly the **guns + throwables** in `tables\predmety.def` — Molotov cocktail (5), the nine firearms
(6–14), and Grenade (15); melee weapons, tools and unarmed all fall outside it (see
`def-files-docs/predmety.md`). Because it is re-derived every tick it is a **live flag with no
decay timer** — it clears the instant the player holsters (the "2 → 0" seen when holstering after a
shot: bit 0 drops immediately while the shooting bit lingers on its own timer). This is why the
offence-**registrar** list (§2.1) misses it: registrars set *event* bits (1/3/4/5/6); bit 0 comes
from the per-frame **state mirror**.

**The check covers two cases — both set the bit.** A gun/throwable counts as equipped whether it is
drawn **on foot** or **aimed out of a car window while driving**, so bit 0 is set in both. The two
cases differ only in a side effect, gated on the suspect's linked-vehicle pointer `+0x98`:

- **On foot** (`+0x98 == 0`): besides setting the bit, `police_manager_tick` calls
  `pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn` — a proximity marker on enabled
  NPC zones within ~25 units of the player; the downstream effect is unconfirmed.
- **In a car / window-aim** (`+0x98 != 0`): the bit is set, but the NPC-zone marker is **not** placed.

**Guns and throwables only — not melee.** A **melee** weapon (e.g. baseball bat, id 4) has a
weapon-id below the range, so it does **not** set bit 0 and the police ignore it; a melee weapon only
registers as bit 9 "violence" when it actually *hits* someone (§2.1). So `get_pm_state zz=0` means
"a gun or throwable is drawn", not "any weapon equipped".

---

## 3. `offense_bits_a` vs `offense_bits_b` — short vs long decay

Both fields use the **same `1<<type` layout**; the difference is the **decay window**:

- **`bits_a` (+0x08)** — primary short-window lane, holds *all* offence bits. Collision bits
  (`0x08`/`0x10`/`0x20`) clear when the `+0x18` "last-event" timer expires (≤4000 ms). Each
  non-collision bit has its own decay timer (§3.1).
- **`bits_b` (+0x0a)** — long-window **mirror of the collision/contact bits only**
  (`0x08`/`0x10`/`0x20`), written in lockstep by
  `police_manager_register_collision_offence_for_target`, cleared on the `+0x1c` "crash" timer
  (≤20000 ms).

Since the getter returns `bits_a | bits_b`, a collision offence stays "true" for `get_pm_state`
for the **longer (~20 s)** window while the short lane drives the faster state machine. `bits_b`
is the extended-grace copy for collision offences.

**`bits_b` has NO role in escalation** — it exists only for `get_pm_state`. The escalation /
police-state commit walk in `police_manager_tick` reads offence bits **only** from `bits_a`
(`+0x08` low byte / `+0x09` high byte); it never reads `+0x0a`.
The **sole runtime reader** of `bits_b` is `police_manager_get_offense_bits_for_target` (VA
`0x005ce4c0`), which returns `bits_b | bits_a`, and that getter's **only caller is
`get_pm_state` (`program_process_command` case 0xae)**. So `bits_b` plays no part in the
police-state level, pursuit dispatch, the HUD, or the wanted bar — its entire purpose is to widen
the `get_pm_state` detection window for the three collision bits (contact `0x08` / crash `0x10` /
run-over `0x20`) from the engine's ~4 s reaction window (`bits_a` / `+0x18`) to the ~20 s
script-query window (`bits_b` / `+0x1c`). This matches the only shipped consumer: mission
`MISE02A-TAXI`, script `Delnik`, queries exactly those three collision types (§4.1).

### 3.1 Per-bit decay timers (ticked in `police_manager_tick`'s ~3 Hz inner walk)

Only **sticky** bits have a decay timer. The **live flags** (bit 0 weapon-equipped, bit 2 speeding)
have none — they are re-derived from current state every inner tick — so they are not listed here.
Each timer is decremented by the frame dt; when it reaches 0 the listed bit(s) clear.

| Bit(s) cleared | When | Timer field | Window length / how it is set |
|---|---|---|---|
| `bits_a` `0x02` — shooting (bit 1) | `+0x10` reaches 0 | `last_fire_time` | **+5000 ms per shot, clamp ≤ 60000 ms** — `police_manager_register_gunfire_offence_for_target` (sustained fire builds up to 60 s) |
| `bits_a` `0x08`/`0x10`/`0x20` — contact 3 / crash 4 / run-over 5 | `+0x18` reaches 0 | `collision_short_timer` | **≤ 4000 ms** — `police_manager_register_collision_offence_for_target` raises it to a severity-scaled value, clamped to 4000 (short collision window) |
| `bits_b` `0x08`/`0x10`/`0x20` — 3 / 4 / 5 (long-decay mirror) | `+0x1c` reaches 0 | `last_crash_time` | **≤ 20000 ms** — same registrar, clamp 20000 (long collision window) |
| `bits_a` `0x0200` — violence (bit 9) | `+0x20` reaches 0 | `violence_event_timer` | **+5000 ms per hit, NO clamp — accumulates** — `police_manager_register_violence_offence_for_target`. A single punch = 5 s; sustained melee keeps extending it by 5 s per landed hit. The **only** offence timer without a ceiling. |
| `bits_a` `0x0400` + `0x1000` — red light (bit 10) **and** traffic sign (bit 12) | `+0x24` reaches 0 | `running_red_timer` | **+4000 ms per event** — the red-light block clamps to 10000 ms, `police_manager_register_traffic_sign_offence_for_target` clamps to 20000 ms. **Shared timer:** the `& 0xEB` expiry clears **both** bits at once |
| `bits_a` `0x0800` — car theft (bit 11) | `+0x0c` reaches 0 | `car_theft_timer` | **fixed 5000 ms** — `police_manager_set_car_theft_offence_for_target` sets it to the constant 5000 (NOT per-car). The different *steal times* per car are the steal **animation/process** re-arming the bit each tick while you break in; this 5 s is only the post-steal decay |

Each sticky bit auto-expires after its window, so `get_pm_state` reflects **recent** offences.

**Note — two offence bits have NO decay timer.** There is a
third category beyond "live flag" and "sticky-with-timer": **bit 13 (assault on a police officer)
and bit 14 (force handcuffs) are sticky but have no decay timer at all** — which is why they are
absent from the table above. Nothing in `police_manager_tick` ever clears them; the *only* thing
that clears them is `police_manager_record_clear_pursuit_state` (full record wipe, which fires
only when the wanted bar fully drains to its `-1` "not running" sentinel). Contrast bit 15
(assault on a police *car*), which is **transient** — consumed in a single
`police_manager_try_dispatch_pursuit_for_target` pass (§2.1).

The gameplay consequence is that **assault on an officer behaves as a permanent gun-tier offence
for the life of the pursuit**. For as long as the record lives, `police_manager_tick`'s escalation
walk keeps re-deriving `pending → gun (3)` from the still-set bit (the `+9 & 0x20` block, gated
`pending < 3`, §2.3), and its `action_code 9` stays in `record+0x64`. Because
`police_manager_try_dispatch_pursuit_for_target` reprints `action_code + 0x834` to **every fresh
foot-cop / patrol-car dispatch wave** (§5), the "Assault on Police!" line is re-announced each time
new cops are pulled in — e.g. when you break line-of-sight (icon fades) and then come within range
of a previously-uninvolved patrol car: at gun the activation is omnidirectional (facing cone and LOS
gates both off — §6), those cops activate, and the *same* assault line prints again. This is the
structural reason a previously-shown assault line reprints with no new offence committed.

---

## 4. `get_pm_state` — effective behaviour is `(1 << zz) & bits`

Script command (`program_process_command` case **0xae**, opcode 174):

```
get_pm_state xx, yy, zz
  flt[yy] = 1.0  iff actor xx has offence-type bit (1<<zz) set, else 0.0
```

`xx` = actor index (0 = player), `yy` = destination `flt[]` slot, `zz` = the offence type of the
Mafia Script Data Reference. The case-0xae decomp ANDs `argv[2]` with the bits directly; the
operand reaching the AND is **already the mask `1 << zz`** (the shift is produced before this
point — see §9 for the one unresolved mechanism detail). The bit↔offence map is in §2.1
(byte-traced).

**The mask is `1 << zz`, not a raw bitmask.** Reading the literal `argv[2] & bits` as "`zz` is a
raw bitmask" (so `zz`=6 → mask `0x06`) contradicts the only shipped consumer: mission
`MISE02A-TAXI` is a gun-less mission in which `get_pm_state 1,0,6` correctly detects sidewalk
driving, which only `(1<<zz) & bits` (zz=6 → `0x40`) produces. Every observed call fits that model.

**In-game `get_pm_state` sweep.** Bits 0–12 were verified one-by-one in normal play with a
purpose-written test script that prints each result. **`zz ≥ 13` is unreliable** — once the traffic
sign damage bit (12) is set, `get_pm_state zz=13/14/15/16` *also* return 1, so the command is only
trustworthy for `zz = 1..12`. The command was evidently not intended to be used past bit 12; this is
also why the Mafia Script Data Reference documents bit 13 as a duplicate "traffic sign" type — it is
reading spurious state, not a distinct offence. See §9.

### 4.1 Real script usage — mission `MISE02A-TAXI` is the only mission that uses `get_pm_state`

| Script | Call | `1<<zz` | Detects |
|---|---|---|---|
| `Delnik.script` | `get_pm_state 1, 0, 5` | `0x20` | run over pedestrian |
| `Delnik.script` | `get_pm_state 1, 0, 4` | `0x10` | vehicle crash |
| `Delnik.script` | `get_pm_state 1, 0, 3` | `0x08` | vehicle contact |
| `Delnik.script` | `get_pm_state 1, 0, 6` | `0x40` | driving on sidewalk |
| `detector_tommyout.script` | `get_pm_state 0, 0, 9` | `0x200` | assault on police |

---

## 5. How the offence text is shown

Offence/wanted strings reach the HUD message console via `hud_console_print(id, color)` (textdb
lookup + 5-slot ring):

- **Engine-driven** — the police system prints **red** (`0xffff0000`) from two live sites:
  - `police_manager_try_dispatch_pursuit_for_target` (VA `0x005d0b00`): foot officers →
    `action_code + 0x834` (ids 2100–2110); patrol-**car** reinforcements → `action_code + 0x866`
    (ids 2150–2160). `action_code` = `pursuit_record + 0x64`, reset to −1 after printing. **Player-only.**
    The **speeding** line 2102 / 2152 is printed here too (`action_code` 2), like every other offence.
  - `police_manager_record_begin_wanted_substate` (VA `0x005d1100`): the "all cops alerted" line.

  (A third, hard-coded, print of 2102 exists in `pedestrian_advance_to_next_node`, but it is **dead code** —
  nothing arms its trigger at runtime; see §5.4.)

The "every offence string appears twice" pattern is the two reinforcement contexts: 2100–2110 =
**foot** cops dispatched, 2150–2160 = patrol **car** joins (same offence text, `+0x834` vs `+0x866`).

The string is selected by **`action_code`** (`+0x64`), a *display* enum independent of the offence
bits (§2) and of `get_pm_state`'s `zz`. So a string can appear with no queryable offence bit
behind it.

### 5.1 String-print table (one row per string id)

| Id | Hex | Text | Shown when | Printer | VA | Formula |
|---|---|---|---|---|---|---|
| 2100 | 0x834 | Police! Weapon seen! | weapon-seen → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(0)+0x834` |
| 2101 | 0x835 | Police! Shots fired! | shots fired → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(1)+0x834` |
| 2102 | 0x836 | Police alarm - speeding! | speeding → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(2)+0x834` |
| 2103 | 0x837 | Police alarm - crash! | collision → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(3)+0x834` |
| 2104 | 0x838 | Police alarm - driving on sidewalk! | sidewalk offence (**never shown in shipped play — unreachable; §8**) | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(4)+0x834` |
| 2105 | 0x839 | Police! Violence seen! | violence → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(5)+0x834` |
| 2106 | 0x83a | Police alarm - running red lights! | crossing a controlled road-edge on red → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(6)+0x834` |
| 2107 | 0x83b | Police! Car theft! | car theft → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(7)+0x834` |
| 2108 | 0x83c | Police alarm - traffic sign damaged! | traffic-sign → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(8)+0x834` |
| 2109 | 0x83d | Police! Assault on Police! | assault → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(9)+0x834` |
| 2110 | 0x83e | Police! Assault on Police car! | car-assault → foot cops | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(10)+0x834` |
| 2150 | 0x866 | Police! Weapon seen! | weapon-seen → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(0)+0x866` |
| 2151 | 0x867 | Police! Shots fired! | shots fired → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(1)+0x866` |
| 2152 | 0x868 | Police alarm - speeding! | speeding → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(2)+0x866` |
| 2153 | 0x869 | Police alarm - crash! | collision → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(3)+0x866` |
| 2154 | 0x86a | Police alarm - driving on sidewalk! | sidewalk offence (**never shown in shipped play — unreachable; §8**) | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(4)+0x866` |
| 2155 | 0x86b | Police! Violence seen! | violence → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(5)+0x866` |
| 2156 | 0x86c | Police alarm - running red lights! | crossing a controlled road-edge on red → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(6)+0x866` |
| 2157 | 0x86d | Police! Car theft! | car theft → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(7)+0x866` |
| 2158 | 0x86e | Police alarm - traffic sign damaged! | traffic-sign → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(8)+0x866` |
| 2159 | 0x86f | Police! Assault on Police! | assault → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(9)+0x866` |
| 2160 | 0x870 | Police! Assault on Police car! | car-assault → patrol **car** reinforcements | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | `action(10)+0x866` |
| 2200 | 0x898 | All Cops have been alerted! | wanted substate begins (player) | `police_manager_record_begin_wanted_substate` | 0x005d1100 | hard-coded |
| 2201 | 0x899 | Another Cop has joined the pursuit. | alternate-mode dispatch (**never shown; §8**) | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | hard-coded |
| 2202 | 0x89a | Another Patrol Car has joined the pursuit. | alternate-mode dispatch (**never shown; §8**) | `police_manager_try_dispatch_pursuit_for_target` | 0x005d0b00 | hard-coded |

Three lines exist in textdb but are **never printed in normal play** (user-confirmed): **2104**
(sidewalk — structurally unreachable) and **2201 / 2202** (alternate-mode dispatch branch never
taken). Mechanisms in §8.

### 5.2 `action_code` values (row = code)

`action_code` (`+0x64`) is the **display** selector; the printed string is `2100 + action_code`.
Two sources write it: the **`police_manager` family** (codes 1/3/5/8, at offence registration) and
**`police_manager_tick`'s escalation** (codes 0/2/4/6/7/9/10, per-frame from the offence bits +
target state). Every code has a located writer. Code 6 (red lights) comes from the
road-edge-crossing block in `police_manager_tick`: the player is within 15 units of a controlled
edge, straddling it, with signal `g_traffic_light_states[idx] == 1`.

| code | id | String | `action_code` writer | Trigger |
|---|---|---|---|---|
| 0 | 2100 | Weapon seen! | `police_manager_tick` escalation | weapon-equipped bit 0 set + pending 0 **and the weapon is visible**: on foot or when driving — the gun is held out the window, handgun only) |
| 1 | 2101 | Shots fired! | `police_manager_register_gunfire_offence_for_target` | weapon fired (also bit `0x02`, zz 1) |
| 2 | 2102 | Speeding! | `police_manager_tick` | speeding bit `0x04` + live target |
| 3 | 2103 | Crash! | `police_manager_register_collision_offence_for_target` | collision (bits `0x08`/`0x10`/`0x20`) |
| 4 | 2104 | Driving on sidewalk! | `police_manager_tick` escalation | sidewalk timer `+0x14 > 5800` — **unreachable (§8)** |
| 5 | 2105 | Violence seen! | `police_manager_register_violence_offence_for_target` | violence/punch (also bit `0x200`, zz 9) |
| 6 | 2106 | Running red lights! | `police_manager_tick` escalation | player crosses a controlled road-edge while signal `g_traffic_light_states[idx] == 1` |
| 7 | 2107 | Car theft! | `police_manager_tick` escalation | bit `1<<11` (`set_arrest_assist`) → pending handcuffs |
| 8 | 2108 | Traffic sign damaged! | `police_manager_register_traffic_sign_offence_for_target` | fine offence (also bit `0x1000`, zz 12) |
| 9 | 2109 | Assault on Police! | `police_manager_tick` escalation | escalation bit 13 (`+9 & 0x20`, writer `police_manager_register_assault_on_police_officer_for_target`) |
| 10 | 2110 | Assault on Police car! | `police_manager_tick` escalation | escalation bit 15 (`+9 & 0x80`, writer `police_manager_register_assault_on_police_car_for_target`) |

The displayed string, the `get_pm_state` `zz`, and the police-state level do **not** line up 1:1:
the action-5 path shows "Violence seen!" yet sets the `zz`=9 (assault) bit; contact/crash/run-over
are three `zz` bits but all print the single action-3 "crash!" string.

### 5.3 Which offence string wins — `action_code` arbitration

When several offences land close together, only **one** string prints (the dispatcher reads
`action_code`, prints `2100`/`2150 + action_code`, then resets it to −1 — §5; and a given cop
prints its line only on the **first** offence it registers — user-confirmed). Which string wins is
decided **deterministically by the monotonic pending level `record+0x3e`**, not by a print race.

`action_code` (`record+0x64`) has two writer classes:

- **Immediate** — written at the event by the registrar itself, which *also* pre-raises pending:
  - **1 "Shots fired!"** — `police_manager_register_gunfire_offence_for_target` (pending → 3 = gun).
  - **3 "Crash!"** — `police_manager_register_collision_offence_for_target` (pending → 1 or 2 by impact).
- **Deferred** — the registrar sets only the offence *bit*; `police_manager_tick` re-derives the
  `action_code` every inner tick (~3 Hz), **gated on pending**:
  - **9 "Assault on Police!"** ← bit 13 (`police_manager_register_assault_on_police_officer_for_target`):
    fires only while **pending < 3**.
  - **10 "Assault on Police car!"** ← bit 15 (`police_manager_register_assault_on_police_car_for_target`):
    fires while **pending < 2** when the severe-impact flag `+0x6c` (`9.7 < severity`) is set, or
    while **pending == 0** when it is not.

The two assault registrars set **only their bit, never `action_code`** (byte-verified) — the
"Assault" strings exist solely because `police_manager_tick` derives them. Because a crash pre-raises
pending and shooting pre-raises it to gun (3), the **immediate** offence **gates out** the deferred
assault string whenever pending has already passed that bit's threshold. This reproduces every
in-game observation:

| Player action | Immediate write (pending after) | Assault bit + its gate | Printed | State |
|---|---|---|---|---|
| Crash into police car (driving) | action 3, pending → **2** | bit 15, `+0x6c` severe → needs pending < 2 → **blocked** | **Crash!** | handcuffs |
| Hit a foot cop with a car | action 3, pending → 1 / 2 | bit 13 → needs pending < 3 → **fires** (action 9) | **Assault on Police!** | gun |
| Punch a foot cop | *(none)* pending 0 | bit 13 → fires | **Assault on Police!** | gun |
| Punch a patrol car | *(none)* pending 0 | bit 15 set; severe punch (`+0x6c` set) → pending < 2 → fires (action 10, pending 2) | **Assault on Police car!** | handcuffs |
| Shoot a cop | action 1, pending → **3** | bit 13 → needs pending < 3 → **blocked** | **Shots fired!** | gun |
| Shoot a patrol car | action 1, pending → **3** | bit 15 → **blocked** | **Shots fired!** | gun |

So "Assault on Police car!" is **not** lost to a timing race when you ram a cop car while driving —
it is **gated out**: the crash raises pending to handcuffs (2), and the severe car-assault
escalation requires pending < 2. The on-foot officer assault survives the same crash because *its*
gate is pending < 3. Shooting raises pending straight to gun (3) and therefore suppresses **both**
assault strings.

**Punch-patrol-car detail.** Live write-breakpoints confirm the full path: punching a cop car
**does** set bit 15 (writer `police_manager_register_assault_on_police_car_for_target`, instr VA
`0x005d1293`), but the bit is **transient** — `police_manager_try_dispatch_pursuit_for_target`
consumes it on the next pass (`+9 &= 0x7f`, VA `0x005d0d12`) after printing the line, so a
value-watch on `+0x09` samples 0. The **handcuffs** result is the lasting state: the bit-15
**severe** branch (`+0x6c` set → `pending < 2` → `pending = 2`) commits to handcuffs, on a single
hard punch. See the §2.1 bit-15 note for the set/consume mechanism.

### 5.4 The `pedestrian_advance_to_next_node` speeding path — **dead code**

`pedestrian_advance_to_next_node` (VA `0x004b60b0`) contains a *second*, self-contained way to react to a
speeder that is **never reached in the shipped game**. It is the per-tick navmesh mover for ambient
NPCs; when an NPC's re-path countdown (`+0xb0`) expires it runs whichever "scheduled action" field is
set. One of those actions is `+0x1b8` (a target actor): if set, it
`police_manager_spawn_patrol_car_reinforcements_at_radius(target, 0, 200.0)` (spawn a crewed patrol
car ~130 m away), and **if** that succeeds **and** the target is the player, it prints **2102**
("speeding") via a **hard-coded** `hud_console_print(0x836)` and plays a "call the police, a crime
happened" voice bark (`pedestrian_manager_play_pedestrian_scream`, `voices.def` rows **174–176**,
voice-banks **26–31** = ids `27940016`…`32940018`). It then clears `+0x1b8`.

**Why it is dead:** nothing in the runtime ever arms an NPC's `+0x1b8` with a live target. Every
writer of that field is either a zero-clear or the NPC save/load round-trip
(serialise to packed `+0x61` / deserialise back) — there is no equivalent of a runtime setter.
So the police-call slot is wired but never triggered. This matches in-game observation: the barks are
never heard and NPCs never phone in a crime.

Two further tells that this is cut / early-design content: (1) the print is **hard-wired to the
speeding string** regardless of what was "witnessed" — it never reads `action_code`, so it can't
represent any other crime; (2) the shipped game handles all of this through the live
`police_manager_tick` → `police_manager_try_dispatch_pursuit_for_target` proximity + line-of-sight
detection (§6), which supersedes it. The voices.def "call police" barks (rows 174–176) are unused
assets left behind by this path.

**Revived-but-non-functional.** The path *can* be force-enabled by a byte patch that hooks the
`+0x1b8` load and arms it with the player, after which blocking an on-foot pedestrian fires the
branch. When triggered it does produce all the cosmetics: the 2102 line prints, the gendered "call
the police" bark plays from the witnessing NPC's voice bank, and a real crewed patrol car is
deployed from the pool (`police_manager_deploy_vehicle_from_pool` returns non-zero, which is the
only thing gating the line). **But the car never pursues.** User-confirmed in-game: with the spawn
radius lowered so the car materialises in view, it spawns as a normal **cruising** patrol car and
simply drives away on ordinary traffic AI — even when a pursuit is already active. The
`police_board_create_and_engage` call inside the spawner has no live pursuit context to attach to
from this cold path, so the "engage" never bites. The feature is therefore **revivable but
functionally incomplete** — it yields the announcement + bark + a cruising car, never an actual
chase, which is consistent with cut / early-design content that was never finished rather than a
working system that merely lost its trigger.

---

## 6. How the police *notice* an offence (range + line-of-sight + facing)

**Recording** an offence bit needs no witness: a pursuit record for the player exists whenever the
police manager is enabled (it auto-registers the player), so the bit is set the instant the offence
happens. **Being noticed** — cops/cars dispatched at you and the red line printed — is a separate,
gated step.

**The noticing chain.** `police_manager_tick` (VA `0x005cb510`) runs the per-suspect state machine
every ~330 ms. Once a **committed police-state** (`record+0x3c`: 1 fine / 2 handcuffs / 3 gun) is
set, every tick it calls `police_manager_try_dispatch_pursuit_for_target` (VA `0x005d0b00`) centered on
the suspect. Dispatch then runs **two independent activation paths**, each with its own detection
gates:

1. **Foot path — `pedestrian_manager_spawn_cop_from_pedestrian`** (VA `0x004b45f0`). Scans the
   ambient NPC zone array for dormant cop dummies (category `+0x15a ∈ {1,3}`; a normal foot cop is
   category 1, confirmed by live memory inspection) and promotes one into a real foot officer only
   if it is **in range** and — **below gun** — both **within a ~90° facing cone** and with **clear
   line-of-sight** (a raycast from the cop's eye (`cop_pos + 1.2`) toward the suspect against world
   geometry, layer mask `0x100000`; blocked ⇒ that cop is skipped). **At gun both gates are bypassed
   (omnidirectional — see below).**
2. **Vehicle path — `traffic_car_activate_or_spawn_pursuit_vehicles`** (VA `0x00504780`). Scans
   `g_traffic_car_registry` for a patrol-car candidate and reactivates a pooled car (or
   constructs one on a pool miss), crewed by 2 cops, only if it is **in range** (squared distance ≤
   radius²) and — **below gun** — both **within the car's ~180° forward hemisphere** (dot test
   `(suspect − car) · car_forward < 0 ⇒ skip`; car_forward at `+0xd28/+0xd2c/+0xd30`) and with
   **clear line-of-sight** (segment query from the candidate to the suspect not blocked). **At gun
   both gates are bypassed (omnidirectional — see below).**

Cop **cars do NOT go through the foot path** (verified by live memory inspection) — approaching a
cruising patrol car never hits the foot-activation marker (VA `0x004b4bbc`); car crews come from the
vehicle path.

**The police-state radius.** Both paths size their range from the committed police-state
`record+0x3c`: **25 m fine / 45 m handcuffs / 65 m gun / 50 m default** (the dispatcher computes this
once and forwards it to both paths).

**The detection gates depend on police-state.** Below gun a cop/car notices you only if it is both
*facing* you and has clear line-of-sight; at gun both gates drop and noticing is **omnidirectional**.
The **facing** gate is driven by the **below-gun flag** (`police_state < gun`, byte-confirmed):
`police_manager_try_dispatch_pursuit_for_target` packs it into a local flag word and passes it to the foot
path as `directional` and to the vehicle path as `param_4` (call site VA `0x005d0d7f`).

- **Below gun (fine / handcuffs — e.g. speeding):** facing + LOS gates are **ON**. A cop/car notices a
  suspect only when *facing* it (foot ~90° cone; car ~180° forward hemisphere) **and** with clear,
  occlusion-free line-of-sight. This is why, when speeding, a cruising patrol car doesn't react until
  you **overtake it / cross into its front**.
- **At gun (e.g. shooting, assault on an officer):** the gates are **OFF** → noticing is
  **omnidirectional**: cops/cars within range notice you from **any direction (including from behind)
  and through occlusion — no facing cone and no line-of-sight**. This is why approaching a patrol car
  from the rear at gun gets you spotted, and why firing a gun draws even cops that cannot see you.
  There is no separate "sound" or hearing detection; this is the whole mechanism.

The red line prints only if **≥1 cop/car passes its gates** — foot → ids 2100–2110 (`action_code +
0x834`); car → the identical-text ids 2150–2160 (`action_code + 0x866`), see §5.1. So weapon-seen,
crash, violence, traffic-sign, red-lights, etc. all require a cop **near you** and — **below gun** —
also **facing you with clear line-of-sight**; at gun a nearby cop notices from any direction without
sight. The sidewalk line shares this gate too; it is additionally blocked by a separate shipped bug (§8).

**Exceptions:**
- **Shooting** is the **only** offence that jumps **straight to gun** the instant it is committed,
  and the police officer might be far away from the player.
  Because gun-tier noticing is omnidirectional (above), any cop within the gun radius then notices
  regardless of facing or sight, so the shooter can be far away and out of view. (Assault on a police
  officer also reaches gun, but is inherently close-range: you must be next to the officer to commit
  it.) The exact code site that drops the LOS raycast at gun is not fully traced — §9.
- **Speeding** prints through the normal dispatch like every other offence (`police_manager_tick`
  sets `action_code` 2 → `police_manager_try_dispatch_pursuit_for_target` prints 2102 foot / 2152 car).
  A *separate* hard-coded 2102 print exists in `pedestrian_advance_to_next_node`, but it is **dead code** (§5.4).

---

## 7. Cross-references

- `structs/police_manager.md` §4 — `pursuit_record` field layout.
- `police-system/artifacts/police-system.md` — runtime behaviour, textdb string list, call chain.
- `police-system/artifacts/police-manager.md` — the pursuit state machine and dispatch sequence.
- `script-mapping/artifacts/opcodes-verified.md` — `get_pm_state` (case 0xae) + the `get_pm_*` cluster.
- `def-files-docs/materialy.md` — surface-material table (id 3 = sidewalk).
- `def-files-docs/predmety.md` — weapon ids 5–15 (guns + throwables).
- The Mafia Script Data Reference — the `zz` offence-type list.

---

## 8. Strings that never print in shipped play

Three textdb lines exist but are never shown in normal play, for two independent structural reasons.

**2104 / 2154 — driving on sidewalk, unreachable by an off-by-800-ms bug.**
`police_manager_tick`'s `action_code = 4` gate needs `record+0x14 > 5800`
(`CMP word [eax+0x14], 0x16A8` @ VA `0x005cc43f`), but the **same function hard-clamps that timer to
5000** (`if (5000 < record+0x14) record+0x14 = 5000`). `5000 < 5800` always ⇒ the line is
**unreachable by design**, a shipped bug rather than a merely hard condition. The timer growth and
the pending / `action_code = 4` set have no line-of-sight check, but the **print** is LOS-gated like
every other line (§6). A byte patch that lowers the gate (and optionally the speed gate, repointed
to a private constant so the shared `11.0` is untouched) makes the line reachable in
`Game-sidewalk-patched.exe`; the cop-must-see-you gate remains, as intended.

**2201 / 2202 — "Another cop / patrol car joined the pursuit", an alternate-mode branch never taken.**
They print only from the alternate-mode (`param_6 != 0`) dispatch branch, so only **2200** is ever
shown. `police_manager_try_dispatch_pursuit_for_target` has exactly two callers, and `param_6` is the
alternate-mode selector (0 → normal strings 2100–2110 / 2150–2160; non-0 → alternate strings 2201 /
2202):

- the **main pursuit dispatch in `police_manager_tick`** (the `sVar4 != 0` "a police level is set"
  branch) passes **`param_6 = 0`** → the path actually exercised in every pursuit can only ever print
  the normal strings;
- the **cop pursuit-broadcast routine** (the per-cop radio/voice path that re-broadcasts at the cop's
  position, ~30 m radius) is the *only* caller passing **`param_6 = 1`**, but it also passes
  **`param_5 = 1`** (identity-pending gate ON, requires `record+0x51 == 0`) and matches the record by
  the *officer's* target field rather than the player directly. For 2201 / 2202 to fire it would
  additionally need: target == player, identity already locked in, a valid `action_code`
  (`record+0x64 ≥ 0`), AND a fresh foot-cop / patrol-car spawn in that same call
  (`police_manager_get_free_officers_count() > 1`). That confluence is not reached in normal play —
  hence the lines stay dead, matching the user-confirmed observation.

---

## 9. Open items

1. **The `1 << zz` shift site.** Case 0xae ANDs `argv[2]` directly, so the operand is the
   pre-shifted mask. Whether the script compiler emits `1<<zz` into the `.klz`/`.def` or the VM
   argument-decode shifts it is **not determinable from `Game.exe` alone** (it needs the offline
   compiler or the raw dispatch asm). The storage model and bit map are solid regardless.
   In game, the `(1<<zz) & bits` model is confirmed for `zz = 1..12`; for `zz ≥ 13` the read is
   corrupted — querying 13/14/15/16 returns 1 as soon as the traffic sign damaged bit (12) is set.
   This is consistent with an as-yet-unidentified mask layer on `argv[2]` that only behaves for the
   designed `zz` range.
2. **The code site that drops the LOS raycast at gun** is not fully traced (§6). The behaviour is
   in-game confirmed and the below-gun flag is byte-confirmed; the exact raycast bypass is not.
3. **Identity retention after an in-car assault on an officer.** Committing the offence while in a
   car and then leaving that car unseen (transparent gun state) leaves the identity known — no
   searching icon appears. Stealing another car after that does make the searching icon appear
   (identity pending). The mechanism is not traced; pending in-game testing.
