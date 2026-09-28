# Car "Boosting" — the pursuit model-level / CARINIT swap

**Created** 2026-07-09 20:19 +0200 · **Modified** 2026-09-17 17:58 +0200

**Status:** static analysis complete, byte-verified against `Game.exe`. The in-game Cheat Engine
confirmation is open (§10).

**What "boosting" is (speedrun term):** when a car starts chasing the player it becomes noticeably
faster and handles better than the same model normally does. This document describes the mechanism,
the enabling flag, which cars can be boosted and why, and every place the flag is toggled.

**One-line answer:** boosting is the pursuit system swapping the chase car to its **top model
level**, which re-loads that variant's **CARINIT physics table** (higher acceleration, sharper
gears/accel/brake/handling). It is gated by a single file-static byte, `g_pursuit_change_tables`,
whose **only writer is `pursuit_event_set_change_tables()`**. That byte is exactly what the
autosplitter reads as `boostEnabled`.

---

## 1. The boost flag = the autosplitter's `boostEnabled`

| | |
|---|---|
| Flag | `g_pursuit_change_tables` — a 1-byte file-static in the pursuit-event translation unit |
| VA | **0x0064C6D8** |
| Sole writer | `pursuit_event_set_change_tables(bool)` @ **0x004EC1E0** (body is literally `mov byte [0x0064C6D8], cl ; ret`) |
| Readers | `pursuit_event_dispatch_pursuit_vehicle` (apply boost) + `pursuit_event_remove_vehicle` (remove boost) |

**Identity with the autosplitter (`boostEnabled : 0xE7D64, 0x18`) — proven by arithmetic:**
- The autosplitter path is `[[module_base + 0xE7D64]] + 0x18`. With Mafia's non-relocated base
  `0x00400000`, `module_base + 0xE7D64 = 0x004E7D64`.
- The dword statically stored at `0x004E7D64` is **0x0064C6C0** (a static pointer to the
  pursuit-event static-data block; `0x004E7D64` has zero code xrefs — it is a data anchor the
  autosplitter piggy-backs on).
- `0x0064C6C0 + 0x18 = 0x0064C6D8 = g_pursuit_change_tables`.

So **`boostEnabled` and `g_pursuit_change_tables` are the same byte.** Ghidra hides the byte
argument at every `pursuit_event_set_change_tables()` callsite, so the decompile alone cannot tell
enable from disable — §6 recovers the true values from the raw bytes.

## 2. How the boost is applied and removed

`g_pursuit_change_tables` is read at exactly two pursuit-event methods:

**Apply — `pursuit_event_dispatch_pursuit_vehicle` (`AddCarHunt`, @ 0x004EC5F0):**
```c
if ((g_pursuit_change_tables != 0) && (car_get_model_level_count() > 1))
    car_set_model_level(car_get_model_level_count() - 1);   // → top model level
```

**Remove — `pursuit_event_remove_vehicle` (`RemoveCar`, @ 0x004EDAC0):**
```c
if (((g_pursuit_change_tables != 0) || record_state in {3,4}) && car_get_model_level() > 0)
    car_set_model_level(0);                                     // → back to level 0 (stock)
```

`car_set_model_level(idx)` (@ 0x00465AD0) activates slot `idx` of the vehicle's stride-0x113c
model-level array and, crucially, calls:
```c
vehicle_apply_carinit_params(slot, 2);   // 0x005123D0
```
which **loads that slot's CARINIT physics block into the live car** — engine power/torque, gear
ratios (`1.0/x`), accel/brake limits and grip, i.e. the full handling profile. It also copies the
slot's 6-dword matrix/projection block into the active block and recomputes a projection scale. So a
"model level" is a complete car variant (visual + physics), and activating the top level is the
boost.

**The boost does not raise a stored top speed.** The block contains one nominal km/h speed field —
`vehicle_apply_carinit_params` loads CARINIT `+0x604` into car `+0x4e4` through the
`*0.2777778`/`*3.6` (km/h↔m/s) conversions — but that field is **byte-identical between level 0 and
level 1** (§3). The *effective* top speed is **derived** from the power/RPM/gearing, and in practice
stays about the same while acceleration and grip rise.

```
chase starts → g_pursuit_change_tables = 1
             → AddCarHunt: model_level_count>1 ? set car to top level
             → car_set_model_level → vehicle_apply_carinit_params(top_slot)
             → car now runs the top variant's faster CARINIT table
chase ends   → RemoveCar: set car back to level 0 → stock CARINIT reloaded
```

### 2a. Script-side entry point

The same CARINIT level swap is reachable from the script language: **`car_setactlevel xx, yy`**,
dispatcher case `0x165`. It gates on the standard actor guard plus kind 4, accepts either a literal
or an `flt[]` register for the level (the `-1` immediate sentinel, `structs/program.md` §1c-v), and
calls `car_set_model_level`, which bounds-checks against the record count and re-applies the
selected record through `vehicle_apply_carinit_params(..., 2)`.

Two caveats for anyone testing this from a script:

* an out-of-range register index does NOT abort the case — it leaves the value at 0 and still calls
  the setter, so a bad argument selects **level 0** rather than being ignored;
* the two READ-BACK commands are unusable. `car_getmaxlevels` (case `0x163`) and `car_getactlevel`
  (case `0x164`) gate on `GetKind == 6` — `C_door` — not 4, so neither can fire for a car. Both have
  zero shipped call sites. To observe the level, read `C_car+0x20ec` directly.

The Mafia Script Data Reference documents all three as fuel-tank commands; the bodies show no fuel
model anywhere.

### 2b. Boosting vs. `g_police_speed_factor` — capability vs. utilization

Two independent knobs, differing in **who they affect**:

- **boosting (`g_pursuit_change_tables`)** picks the car's CARINIT **variant** (level 0 stock vs
  level 1 boosted) → the car's physical **capability** (accel/grip/gearing ceiling). It is a
  process-global **read by every chase dispatch**, police *and* non-police (§4).
- **`g_police_speed_factor`** (VA `0x006552D0`, script cmd `POLICE_SPEED_FACTOR` = case 0x173,
  clamped `[0,1]`, default `1.0`) is the AI driver's **throttle cap** — how hard it presses the gas.
  It is **not** read by all pursuits: only the **police** chase-dispatch path passes it.

**Propagation of the limit → `vehicle_set_power` (police path):**
1. script `POLICE_SPEED_FACTOR` (case 0x173) writes `g_police_speed_factor` @0x006552D0, clamped `[0,1]`.
2. `police_action_tick`'s chase dispatch passes it as the **last arg** (`param_7`) of
   `pursuit_event_dispatch_pursuit_vehicle` — the only callsites that read the global.
3. `AddCarHunt` stores it into the chase driver's `driver+0x84`.
4. each frame `driver_tick`'s tail computes the pedal `= driver+0x84 (speed_factor) × driver+0x104`,
   where `+0x104` is the AI's per-frame throttle demand (`1.0` on a clear straight, pulled down by
   the inlined corner/collision helpers), then calls `vehicle_set_power(pedal, 1)`.
5. `vehicle_set_power` writes engine power `vehicle+0x634 = (ramp_accum/ramp_max) × pedal`.

So `g_police_speed_factor = 0.8` = the police AI throttle is **hard-capped at 80%** (and scaled to
80% of its demand everywhere) → slower accel, lower sustained speed, easier to outrun.

**Only police cars are limited by it.** Other pursuers pass their own `param_7`:

| Pursuer | Throttle passed |
|---|---|
| police (`police_action_tick`) | `g_police_speed_factor` |
| gang hit cars (Free Ride; `traffic_car_spawn_mafia_pursuit_car`) | hard `1.0` (`0x3f800000`) literal ¹ |
| mission-script chases (`entity_process_command` opcode `0x100c`) | scripted speed, `arg × 0.01` (`1.0` only when `arg == 100`) |
| ai_group / ai_fight combat chases | `ai_group_get_car_factor(vehicle)`, default `1.0` |
| position/hold dispatches (e.g. the `target==0` callsites) | `1.0` (`0x3f800000`) |
| player and traffic cars | - (they do not use the driver system: pedal = player input / traffic-cruise AI) |

¹ A Free-Ride gang chase therefore runs full-throttle regardless of `POLICE_SPEED_FACTOR` (§7).

Effective chase pace = *capability* (boosted-or-not physics) × *utilization* (throttle). Boosting
feeds every chase; `g_police_speed_factor` gates only police chases.

## 3. Which cars are boostable — and why it is defined in the game files

The runtime boostability test is **`car_get_model_level_count() > 1`** (@ 0x00465A90): the count of
stride-0x113c descriptor slots in the car's `+0x20F4`(begin)/`+0x20F8`(end) array. Each slot carries
its own CARINIT block (at slot `+0x1124`), loaded via `vehicle_apply_carinit_params`. The number of
slots is **not tied to model files** — it comes straight from the car's record in
**`tables/vehicles.bin`**, which defines one or more CARINIT "levels" per car.

### vehicles.bin format (verified against the file)

```
u32 version = 1
u32 car_count = 89
repeat car_count times:
    u32 record_size            // 0x1149 (1 level) or 0x226E (2 levels)
    record:
        char  name[0x20]       // "blackha", "poli", "ambulance", …
        u32   level_count      // 1 = normal car, 2 = boostable (base + boosted)
        repeat level_count:
            u32   level_size = 0x1121
            float carinit[...]  // the CARINIT block (MCVE fields; layout below)
```
The container is a straight array of 89 records; the parse ends exactly at EOF. `vehicles.dat`
(3856 B) is the companion **field-schema** table: per-field `{byte-offset (steps by 4), type
(5=float / 0x11=other)}` descriptors that map named CARINIT fields to record offsets.

### Boostable = `level_count == 2`, verified

Walking all 89 records, **exactly 14 have `level_count = 2`** and every other car has 1. The 14 are
precisely the boostable set:

> `blackha` (Falconer), `buikFor` (Wright Fordor), `cad_ford` (Lassiter V16 Fordor), `forVfor`
> (Bolt V8 Fordor), `hudfor` (Guardian Terraplane Fordor), `arrow` (Silver Fletcher), `polCad`
> (Lassiter V16 Police), `poli` (Schubert Six Police), `polimFor` (Schubert Extra Six Fordor
> Police), `polimTud` (Schubert Extra Six Tudor Police), `chuligani` (BGUTI custom) — every entry in
> `boosting-debug/Boostable cars in Mafia.csv` — plus `bugatti`, `black`, `buigang` (mission/gang
> chase cars not fully listed in the CSV).

This also explains the CSV note "Guardian Terraplane Tudor is not boosted despite both Wrights
are": the **Fordor** body `hudfor` is a `level_count = 2` record, but the **Tudor** body `hudtu` is
a separate `level_count = 1` record — boostability is per vehicles.bin record (per body variant),
and has nothing to do with model files.

### Level 0 vs Level 1 — the boost is acceleration, not top speed (`blackha` / Falconer)

Field names below are from **RHAM 3.0** (the authoritative editor; `boosting-debug/RHAM.png`).
Level 0 is the base block; level 1 is the boosted block. Full field/offset map:
`def-files-docs/vehicles-bin.md`.

| CARINIT field (RHAM) | Level 0 | Level 1 | effect |
|---|---|---|---|
| Power | 58.8 | **160** | engine power → acceleration |
| Max Power at idling RPM | 150 | **450** | low-end torque → acceleration |
| Min Power at redline RPM | 125 | **200** | high-end torque |
| PowerBandKwMax | 90 | **200** | power-band peak |
| GearSpeedFactor | 78 | **88** | gearing responsiveness |
| SkidFactor | 250 | **550** | grip → handling |
| Motor time | 0.28 | **0.4** | throttle/engine response |
| GEAR 2/3/4 shift-down | 15/34/50 | **12/28/43** | earlier downshifts |
| RedLineRPM, maximum rpm, GEARS, gear ratios, Weight, DownForce | identical | identical | - |

**Top speed is NOT changed by the boost.** The fields that set peak velocity — RPM ceiling
(`RedLineRPM` 3000 / `maximum rpm` 3300), `GEARS` (4) and the gear ratios — are identical in both
levels; top speed is a *derived* result of those, not a stored value. The boost raises
**acceleration, grip and drivetrain response** only. (RHAM labels field #2 "Power"; the older
`MCVE.exe` mislabels it "MaxSpeedKMH" — it is not the top-speed value.) **Both editors edit level 0
only**; the boosted level 1 lives in the same record but neither editor surfaces it.

**Load → boost chain:** at vehicle load the engine reads all `level_count` CARINIT blocks from
vehicles.bin into the car's `+0x20F4` slot array (one slot per level); `AddCarHunt` with the flag
set calls `car_set_model_level(level_count - 1)` → activates the top (level 1) block →
`vehicle_apply_carinit_params` loads the boosted physics live.

## 4. The pursuit-event system and the "actor" requirement

Boosting is **only** applied inside `pursuit_event_dispatch_pursuit_vehicle` (`AddCarHunt`), which
is only reached through the pursuit-event chase system. A car boosts **iff an actor enrolls it into a
sledovačka hunt while the flag is on.** That enrollment is NOT police-specific — the dispatch callers
are:

| Caller | Kind |
|---|---|
| `police_board_create_and_engage`, `police_action_tick`, `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation`, `ai_follow_manager_update` | police pursuit |
| `ai_fight_manager_tick` | AI combat chase |
| `traffic_car_spawn_mafia_pursuit_car` | mission/gang chase — realizes a rammed gang car, builds an ai_group, dispatches it into the pursuit |
| `entity_process_command` (the entity mission-script command interpreter) | **mission-script chase command**: its `case 0x100c` reads two entity indices (car + target) from the program args and calls `AddCarHunt(car, target, mask, sub, vec, speed×0.01)` |

This is why the **non-police cars** in the CSV boost: they are mission/scripted chase cars enrolled
by mission actors — most directly via the `entity_process_command` `case 0x100c` program command (a
mission-script opcode that makes one entity's car chase another), plus the gang/AI-fight paths.
Boosting is a general chase property of the sledovačka system, not a police-only effect.

The sibling dispatcher `pursuit_event_dispatch_escape_vehicle` (`AddCarEscape`, @ 0x004EC980,
immediately after `AddCarHunt`) enrolls flee-type chases; its `0x29C` sub-record is a driver
(`driver_ctor`/`driver_dtor`).

**The MISE03-MORELLO exception (observed in game):** there, mission gangsters (scene2.bin actors)
can steal a car and chase the player via ordinary driving AI **without** being enrolled into a
pursuit-event hunt (no `AddCarHunt` call for them). Because `car_set_model_level` is therefore never
invoked for their car, it stays at level 0 and does not boost **even if it is a boostable model and
even with the flag on**. The requirement is enrollment in the sledovačka chase system; no
intrinsic per-actor "can boost" bit exists. (`AddCarHunt` binds a car **and** an actor; that hunt
binding is the "something set on the actor".)

The pursuit-event manager itself is a sub-object of `C_game` at `[[0x0065115C] + 0x24] + 0x2B18`
(its S_ELEMENT record vector lives at manager `+0x14`/`+0x18`). See
`structs/pursuit-event-manager.md` for the full struct.

## 5. Why boost turns off when the player leaves the car / on escalation

The flag has one writer, so every enable/disable is one of the 18 calls in §6. The engine's own
intent, visible in the code:
- On a **fresh pursuit** and during **normal active re-dispatch**, the flag is set to **1** (boost
  the chase car). → matches "always enabled when a new cop car starts chasing."
- On a **police-state escalation** (fine→handcuffs, →gun) the escalation re-dispatches the chase
  vehicle with the flag flipped to **0** (boost off at that transition). → matches "when fine
  changes to handcuffs, boosting is disabled."
- On **pursuit teardown / re-evaluation** the flag is set to **0**. The observed "player leaves the
  car with a police record → identity pending → boost off" routes through the teardown path (the
  squad loses its tracked vehicle and re-evaluates); this is the only family of disable sites (§6),
  plausibly guarding against boosted cars slamming into the now on-foot player.

### Speedrun relevance — one global, written only by police, read by ALL pursuits

`g_pursuit_change_tables` is a **single process-global**. Every writer is in the **police flow**
(all 18 callers in §6 are `police_board_*` / `police_action_*` / police tick paths), but the flag is
**read by every chase dispatch** — police *and* non-police (mission/gang via
`entity_process_command` case 0x100c, `traffic_car_spawn_mafia_pursuit_car`,
`ai_fight_manager_tick`; §4). So a boost state set by the police flow **leaks into non-police
pursuits**: nothing in a mission/gang chase ever flips this flag, yet those cars boost or not
according to whatever the police flow last left it at. **This cross-contamination is exactly why
boosting matters for the speedrun.**

The problematic sequence:
1. End a segment by leaving the car **unseen while under an active pursuit with a police record**
   (police_state → identity pending). The teardown path sets the flag to **0** → boosting disabled.
2. In the **next segment** the flag is *still* `0` (it is global and only the police flow writes
   it). Non-police chase cars pursuing you are therefore **not boosted** → slower, and take much
   longer to catch you.
3. In that segment the segment only ends once those chase cars are **destroyed**, which now takes
   far longer — undesirable for the run.

**The fix (as used in the run):** load a segment whose mission initialization calls
`policemanager_forcearrest` with the **wanted substate** — it spawns a police car that starts
chasing you, which runs the police flow and flips `g_pursuit_change_tables` back to **1** (boost
re-enabled). Then load back into the target segment, where the non-police chase cars are now boosted
(fast) so the pursuit resolves quickly. (Watching/forcing this flag is exactly what the §8 Cheat
Engine guide sets up.)

## 6. Authoritative flag-toggle map (recovered from raw bytes)

Ghidra drops the `CL` argument, so the decompile cannot show polarity. Recovered `CL` at all **18**
direct callsites of `pursuit_event_set_change_tables`:

**ENABLE (CL = 1) — 15 sites:**

| Function | Count | Role |
|---|---|---|
| `police_board_create_and_engage` | 3 | fresh pursuit creation (new cop chase begins) |
| `police_action_tick` | 8 | normal active re-dispatch branches |
| `police_action_dispatch_officer_commands` | 1 | (re)issue officer orders |
| `police_action_disband` (with-vehicle path) | 1 | set just before removing the vehicle |
| `entity_dispatch_event` | 1 | event-driven dispatch |
| `police_board_raise_gun_escalation` | 1 | (a non-dispatch call in the raiser) |

**DISABLE (CL = 0) — 3 sites:**

| Function | VA | Maps to |
|---|---|---|
| `police_board_raise_handcuffs_escalation` | 0x004479EB | **fine → handcuffs disables boost** |
| `police_board_raise_gun_escalation` | 0x004478B8 | escalation to gun tier |
| `police_action_tick` (teardown branch) | 0x00444121 | pursuit collapses / re-evaluates (exit-vehicle path) |

The escalation raisers each contain both an enable and the disable — the disable is the one that
fires on the fresh re-dispatch path, VAs above.

The `pursuit_event_set_change_tables` and `car_set_model_level` docblocks state this polarity. The
inline comments at the `police_action_tick` (8 enable + 1 disable), `police_action_disband` (1
enable) and `police_action_dispatch_officer_commands` (1 enable) sites state the recovered `CL`
value, re-derived from the disassembly and agreeing with the tables above in every case.

## 7. PC vs PS2

| PC VA | PC name | PS2 twin |
|---|---|---|
| `0x00465AD0` | `car_set_model_level` | `C_car::SetActualLevel` |
| `0x004EC1E0` | `pursuit_event_set_change_tables` | `EnableChangeTables__12C_sledovacka` (SLUS_206.71 @ 0x00202FD0) |
| `0x004EC5F0` | `pursuit_event_dispatch_pursuit_vehicle` | `C_sledovacka::AddCarHunt` |
| `0x004EC980` | `pursuit_event_dispatch_escape_vehicle` | `C_sledovacka::AddCarEscape` (CONFIRMED by body diff) |
| `0x004EDAC0` | `pursuit_event_remove_vehicle` | `C_sledovacka::RemoveCar` |
| `0x005053E0` | `traffic_car_spawn_mafia_pursuit_car` | `Krles__13C_traffic_car` |
| - | `entity_process_command` | `Process__8C_entityFUi` (CONFIRMED) |

**Gang-car throttle.** PS2 `Krles` calls `AddCarHunt(car, target, 0x200, 0x14, &zero_vec,
0x3f800000)` — the hard `1.0` throttle of §2b. The PC callsite is Ghidra-mangled but matches.

## 8. Cheat Engine — testing the boost trigger (police vs non-police) & watching the flag

Fixed image base `0x00400000`, no ASLR. **Breakpoints use `Game.exe+X` (= `0x400000+X`); watch
entries are "Add Address Manually" with the pointer chain + Type shown.**

### 8.1 Watch table (Add Address Manually)

Root: `[0065115C]` = `g_act_mission` → `+24` deref → the `C_game` singleton. The **pursuit-event
manager** is inline at **`C_game+0x2B18`**; its S_ELEMENT chase-record vector begins at manager
`+0x14` (stride `0x44`; record `+0` = chase car, `+4` = its driver). Chains below resolve the
**first** chase record (index 0; car N = append `+ N*44` after the `+14`). They only resolve while a
chase is active.

| Watch | Address | Type | Meaning |
|---|---|---|---|
| **boost flag** `g_pursuit_change_tables` | `0064C6D8` | Byte | `1` = boost on, `0` = off. Same byte the autosplitter reads (`[[Game.exe+E7D64]]+18`). |
| chase car[0] model level | `[[[[0065115C]+24]+2B18+14]]+20EC` | 4 Bytes | active CARINIT variant: **0 = stock, 1 = boosted**. Flips 0→1 the instant the car is boosted. |
| chase car[0] pointer | `[[[[0065115C]+24]+2B18+14]]` | 4 Bytes | the chase car pointer (sanity: non-zero during a chase). |
| chase driver[0] speed factor | `[[[[0065115C]+24]+2B18+14]+4]+84` | Float | the `g_police_speed_factor` in effect for this chase (default `1.0`; `POLICE_SPEED_FACTOR` lowers it). |
| record-vector begin / end | `[[0065115C]+24]+2B18+14` / `+2B18+18` | 4 Bytes | `(end-begin)/0x44` = number of active chase records. |

### 8.2 Breakpoints (`Game.exe+X`)

| BP | address | catches |
|---|---|---|
| **writer** `pursuit_event_set_change_tables` | `Game.exe+EC1E0` | every flag write. Read **CL** = value (`1` enable / `0` disable); read **`[ESP]`** (return addr) to see the caller → §8.3. |
| enable path (fresh chase) | `Game.exe+46C00` | `police_board_create_and_engage` (CL will be 1) |
| **disable** – exit-car / lost | `Game.exe+44121` | `police_action_tick` teardown (CL=0) |
| **disable** – gun escalation | `Game.exe+478B8` | `police_board_raise_gun_escalation` (CL=0) |
| **disable** – handcuffs escalation | `Game.exe+479EB` | `police_board_raise_handcuffs_escalation` (CL=0) |
| **apply-boost reader** `AddCarHunt` | `Game.exe+EC5F0` | any car being enrolled+boosted; step to its `car_set_model_level(count-1)` branch |
| **mission-script** enroll (case 0x100c) | `Game.exe+2B016` | `entity_process_command` dispatching a scripted car chase (non-police) |
| **gang** enroll | `Game.exe+1053E0` | `traffic_car_spawn_mafia_pursuit_car` (rammed-gang realize) |
| escape dispatch `AddCarEscape` | `Game.exe+EC980` | flee-type enrollment |
| un-boost `RemoveCar` | `Game.exe+EDAC0` | car leaving the chase (resets model level → 0) |

### 8.3 Which path fired

Break at the writer `Game.exe+EC1E0`; the return address on the stack lands in one of these (all in
the **police flow** — the flag is only written here):

| return addr in | Game.exe+ | CL | meaning |
|---|---|---|---|
| `police_board_create_and_engage` | `+46C00` | 1 | new cop chase → boost ON |
| `police_action_tick` (re-dispatch) | `+43D40` | 1 | ongoing chase keeps boost ON |
| `police_action_tick` teardown | `+44121` | 0 | exit-car / lost-vehicle → boost OFF |
| `police_board_raise_gun_escalation` | `+478B8` | 0 | →gun → boost OFF |
| `police_board_raise_handcuffs_escalation` | `+479EB` | 0 | fine→handcuffs → boost OFF |

**Non-police paths never hit the writer** — `entity_process_command` case 0x100c,
`traffic_car_spawn_mafia_pursuit_car`, `ai_fight_manager_tick` only `AddCarHunt` the car and *read*
the police-set flag. To see a non-police car boost or not, watch its `+20EC` model level (§8.1) or
break `Game.exe+EC5F0`/`+2B016`.

### 8.4 Reproduce & force
- **Police:** commit an offence → cop engages (`+46C00`, flag→1, car boosts). Leave the car unseen
  with the record → teardown (`+44121`, flag→0).
- **Non-police:** free-ride gangster or scripted mission chase enrolls via `+2B016`/`+1053E0`; the
  car boosts **only if the police flow last left `0064C6D8` at 1**.
- **Force the segment fix:** set `0064C6D8` = `1` (freeze) and confirm the next non-police chase
  car's `+20EC` flips to `1` — the CE equivalent of the `policemanager_forcearrest`
  (wanted-substate) reload in §5.

## 9. Cross-references

- `structs/pursuit-event-manager.md` — the pursuit-event manager + S_ELEMENT record struct.
- `structs/police-board.md` / `structs/police-action.md` — the police pursuit board/squad that drives most
  enable/disable calls.
- `ghidra/Game-ghidra.exe.functions.md` — `pursuit_event_*`, `car_*_model_level`,
  `vehicle_apply_carinit_params` rows + PS2-twin columns.
- `def-files-docs/vehicles-bin.md` — the CARINIT field/offset map.
- `boosting-debug/Boostable cars in Mafia.csv` — the observed boostable-car list (matches the 14
  `level_count == 2` records in vehicles.bin, §3).
- `exe-re/game-sources/tables/vehicles.bin` / `vehicles.dat` — per-car CARINIT data (levels) +
  field-schema; `programs/mafia_car_value_editor/MCVE.exe` edits level 0 only.
- `boosting-debug/mafia-autosplitter/…/autosplitter-sounds.asl` — reads `boostEnabled`
  (`0xE7D64, 0x18` → `g_pursuit_change_tables`).

## 10. Open items

| item | state |
|---|---|
| which disable site fires on "exit car → identity pending" | needs an in-game break-on-write at `0x0064C6D8` while leaving a car with a police record; expected: the `police_action_tick` teardown site `0x00444121` |
| full CARINIT offset map | only the MCVE-named fields are pinned; the exact `vehicles.dat` schema semantics are not decoded |
| inline `CL` comments at the remaining sites | `police_board_create_and_engage` (3), `entity_dispatch_event` (1) and the two escalation raisers' sites do not yet state the recovered value |
