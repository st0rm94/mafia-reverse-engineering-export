# The racing classes — `C_racing` and the five objects it owns

**Created** 2026-08-16 07:38 +0200 · **Modified** 2026-10-07 21:57 +0200

One translation unit, **`[0x0060b770, 0x00617a60)`**, **131 starts**, holding **six classes** plus
the free comparators. **131 of 131 PC starts are named — the translation unit is CLOSED.**

Subsystem write-up — what shipped, what was cut, the file formats, the mod:
`basic/artifacts/racing.md` · the entity class that goes with these: `structs/racecam.md`.

> **Why one file for six classes.** All six are **non-polymorphic on both builds** (`__vt__` count
> 0 for every one), they live in a single TU, and five of them are mostly unmapped. Six separate
> stubs would be worse than one sectioned doc. `C_racecam` is the exception and keeps its own file,
> because it is an entity kind with a vtable and it is complete.

> **Naming.** `racing_` / `circuit_` / `champ_circuit_` / `championship_` / `opponent_` /
> `best_time_` map the PS2 class tags `8C_racing`, `9C_circuit`, `15C_champ_circuit`,
> `14C_championship`, `10C_opponent`, `11C_best_time` onto lowercase subsystem prefixes. Free
> functions of the TU take `race_`. The prefixes were checked pairwise in both directions before
> any was picked: the two near-misses (`circuit_` vs `champ_circuit_`, `race_` vs
> `racecam_`/`racing_`) are **not** collisions.

---

## 1. The object graph

```
g_racing  (0x006f958c)  ->  C_racing                     the singleton; in retail built by mission 6 only
                             |
                             +-- vector<C_circuit*>       +0x0c   one per track   (11 ship)
                             |     |
                             |     +-- vector<C_racing_gate*>  +0x40   the lap gates
                             |     +-- vector<C_best_time*>    +0x54   the record table
                             |     +-- vector<C_web_node*>     +0xac   left racing line
                             |     +-- vector<C_web_node*>     +0xbc   right racing line
                             |     +-- vector<race_cam*>       +0xcc   camera nodes + their 2 gates
                             |
                             +-- vector<C_championship*>  +0x1c   one per championship (7 ship)
                             |     |
                             |     +-- vector<C_champ_circuit*>  +0xb0   the rounds
                             |     +-- vector<{u32,u32}*>        +0xc0   the opponent roster
                             |
                             +-- C_opponent[20]           +0x1d8  entrants   (count +0x284)
                             +-- C_opponent*[20]          +0x228  live racers (count +0x288)
```

`C_champ_circuit` points at a `C_circuit` but does **not** own it: the circuits live in
`C_racing`'s own vector and outlive every round.

---

## 2. `C_racing` — the singleton

The root object of the whole mode. `g_racing` (`0x006f958c`, PS2 `CRacing`) is the only
pointer to it. **In the retail PC build it is constructed in mission 6 only**: the mission's
`GameInitStart` script runs `racing_mission6_init`, and `race_mission6_init` builds the object,
writes `+0x28 = 2` and `+0x2c = 3`, and selects circuit 0 (`basic/artifacts/racing.md` §8a,
`structs/race_session.md` §4). In every other mission it is NULL, which is why ~310 references to
it in ordinary gameplay code are `== 0` tests: they are "are we in a race?" gates.

The menu front end is `race_menu_run` (`0x00595d90`, PS2 `ExecuteRacingMenu`): it builds the
design-`0x16` Racing menu and dispatches championship (`g_racing+0x2c` = 1), single race (= 2), a
third mode (= 3, the value the mission 6 race also uses), replay load, multiplayer and exit. Its
sole caller is `engine_main` case `0x15`, immediately after `racing_ctor`; that path is reached
only through the `Racing` main-menu item the shipped executable deletes.

### 2a. Known fields

| Offset | Field | Conf | Witness |
|---|---|---|---|
| +0x0c / +0x10 / +0x14 | `vector<C_circuit*>` | H | `racing_load_circuits` fills it, `racing_get_circuit` indexes it, `racing_dtor` frees it |
| +0x1c / +0x20 / +0x24 | `vector<C_championship*>` | H | `racing_load_champs` fills it, `racing_dtor` frees it |
| +0x28 vs +0x2c | menu mode vs race mode — two different fields | H | distinguished by `racing_can_run_race`; `racing_tick` branches on race mode `== 2` to tick a single racer instead of all of them |
| +0x30 | current `C_circuit` | H | `racing_get_current_circuit` returns it; `racing_do_gates` and `racing_get_distance_by_gates` index its gate vector through it |
| +0x34 | current `C_champ_circuit` (the round being raced) | H | `racing_fill_champ_results` stops summing points at it; `racing_tick` reads its `+0x04`; the championship launch and results screens list its `+0x68` best-race table (`structs/racing_menus.md` §6) |
| +0x38 | current `C_championship` | H | written by `racing_set_current_champ`; read by `racing_fill_champ_results` and `race_compare_champ_points` |
| +0x3c | difficulty level, 0-4 | H | the switch selector in `racing_get_power_coeff_by_difficulty`; copied from championship `+0xd0` |
| +0x40 | an embedded `C_opponent` — **the player's entrant** | H | `racing_tick` ticks it alone in race mode 2; `racing_fill_champ_results` appends it as the last result row; `racing_update_driver_power_coeff` measures every AI racer's distance from it |
| +0x1d8 | entrant array (20 slots) | H | `racing_generate_qualification` walks it |
| +0x228 | live racer array (20 slots) | H | `racing_tick` and `opponent_set_death` walk it; the accessor is `racing_get_racer_array_ptr` (`0x005c8270`) |
| +0x27c | the race's total lap count | H | `racing_do_gates` pushes it to the HUD with the current lap |
| +0x284 | entrant count | H | `racing_get_opponent_count` (`0x00610870`) reads it; it excludes the player entrant |
| +0x288 | live racer count | H | `racing_get_racer_count` (`0x005c8280`) reads it. Two adjacent counts, two accessors — `+0x284` and `+0x288` are not interchangeable |
| +0x28c | standings row count: how many rows of `+0x228` the results tables and the finish-place count walk | H | `racing_init_racers` sets it to 1, then to `+0x284` + 1 in race modes 2 and 3; `racing_end_race` sets it to `+0x284` + 1 before sorting `+0x228` by qualifying time; `racing_remove_opponent_at` decrements it with `+0x288`; read by `racing_next_finish_place`, both results screens, `camera_tick` and `net_session_handle_event_msg` |
| +0x290 | elapsed race time | H | `racing_tick` computes `now - +0x298` and feeds `hud_set_race_time_text` |
| +0x294 | pause-clock stamp | H | the two `racing_pause_clock_*` helpers |
| +0x298 | race start timestamp | H | `racing_tick` stamps it with `timeGetTime()` when the countdown ends |
| +0x29c / +0x2a0 | tick accumulator / period | H | `racing_tick`; the period is **1000 ms** in both states. Reset together by `racing_reset_phase_timer` (`0x005a14d0`), a `C_racing` setter emitted inside the `g_net_session` translation unit and called from the type-`0xe` arm of `net_session_handle_event_msg` right after `racing_set_phase(0)`. In race phase 1 `racing_do_gates` runs only when the accumulator passes the period, so every racer's gates are checked about once per second (§3b) |
| +0x2a4 | race phase | H | `racing_tick`'s switch: 0 countdown / start lights, 1 racing, 2 post-race delay, 3 cooldown. 2 and 3 are the networked pre-start phases |
| +0x2a8 | countdown step | H | `racing_tick` feeds it to `hud_set_race_start_flag` as 0,1,2,3 and starts the race at 4 |
| +0x2ac | wrong-way timer | M | `racing_tick` stamps it when the player faces backwards and raises HUD bit `0x800000` after 5001 ms |
| +0x2b4 | segment-file variant selector | H | picks `mrseg%i.sav` vs `mrsegm%i.sav` (§10c); also posted as type-`0x31` selector 4 (§8) |
| +0x2b6 | segment learn flag | H | gates the commit in `racing_finish_segment_and_advance` (§10b) |
| +0x2b7 | damage enabled | M | `racing_ctor` seeds it to 1 |
| +0x2b8 | **race-camera racer index** | H | `racing_set_race_camera_index` clamps into it. Not a lap count |
| +0x2bc / +0x2c0 | race-camera cycle timer / dwell period | H | 10000 ms for a running racer, 1000 ms for a retired one |
| +0x2c4 | 7 × 0x10 segment banks | H | `racing_ctor` builds them with `crt_ehvec_ctor(…, 0x10, 7, …)`; one bank per car performance class (§10) |
| +0x334 | an embedded `C_racing_optimal_segment` — the scratch record (§10a) | H | byte-proven by `lea ecx,[esi+0x334]` in `racing_ctor` calling `racing_optimal_segment_ctor` |
| +0x3ac / +0x3d4 | the live position trail: 10 × tag, capacity | H | `racing_game_init` copies a 42-dword record out of a bank into it |
| +0x3dc / +0x3e0 / +0x3e4 | segment recorder cursors: segment index, point write cursor, last sampled car position | H | `racing_record_segment_points` |
| +0x3f0[6] / +0x408[6] | the winners' scene actors and their I3D models | H | `racing_spawn_winners_scene` / `racing_release_winners_scene` |
| +0x420 / +0x424 | driver-power update accumulator / period (1000 ms) | H | `racing_tick` drives `racing_update_driver_power_coeff` off it |

The PS2 offsets of these fields are in §7a.

### 2b. Named methods — **69 `racing_*` functions for 63 PS2 methods**

The PC count exceeds the PS2 method count because twelve `racing_*` functions are
PC-only (§9b) while several PS2 methods are unmatched (§9d). The per-function list lives in the
function registry `Game-ghidra.exe.functions.md`; the table below keeps the rows whose evidence
notes are not reproducible from the registry.

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x0060f910 | `racing_ctor` | `__ct__8C_racingFv` | CONFIRMED |
| 0x0060fb50 | `racing_dtor` | `__dt__8C_racingFv` | CONFIRMED |
| 0x0060fc90 | `racing_load_circuits` | `LoadCircuits__8C_racingFv` | CONFIRMED |
| 0x0060ff10 | `racing_load_champs` | `LoadChamps__8C_racingFv` | CONFIRMED |
| 0x00610230 | `racing_get_circuit` | `GetCircuit__8C_racingFUi` | CONFIRMED |
| 0x00610260 | `racing_get_current_circuit` | `GetCurrentCircuit__8C_racingFv` | CONFIRMED |
| 0x00610270 | `racing_set_current_champ` | `SetCurrentChamp__8C_racingFP14C_championship` | CONFIRMED |
| 0x0060cc60 | `racing_fill_champ_results` | `FillChampResults__8C_racingFP14C_championship` | CONFIRMED |
| 0x0060f6c0 | `racing_set_race_camera_index` | `SetRCamIndex__8C_racingFUi` | CONFIRMED |
| 0x006139c0 | `racing_tick` | `Tick__8C_racingFUi` | CONFIRMED |
| 0x00614dd0 | `racing_generate_qualification` | `GenerateQualification__8C_racingFv` | CONFIRMED |
| 0x00614ee0 | `racing_get_difficulty_scale` | `GetDifficulty2__8C_racingFv` | CONFIRMED ¹ |
| 0x00614f30 | `racing_get_power_coeff_by_difficulty` | `GetPowerCoeffByDifficulty__8C_racingFRfRf` | CONFIRMED |
| 0x00616510 | `racing_do_gates` | `DoGates__8C_racingFP10C_opponent` | CONFIRMED |
| 0x00616d30 | `racing_update_driver_power_coeff` | `UpdateDriverPowerC__8C_racingFv` | CONFIRMED |
| 0x00616e10 | `racing_get_distance_by_gates` | `GetDistanceByGates__8C_racingFP10C_opponentP10C_opponent` | CONFIRMED |
| 0x00611430 | `racing_activate_racers` | `ActivateRacers__8C_racingFv` | CONFIRMED ² |

¹ 80 B against 124 B, ratio 0.65. Two PC-only helpers sit between `racing_tick` and this
function; with them placed, the rank matches.
² Statement-for-statement, and **exactly one callsite**, in `racing_game_init`.

---

## 3. `C_circuit` — one race track

Instance **0xdc B**. Constructed one per record of
`tables/circuits.def`.

| Offset | Field | Conf | Notes |
|---|---|---|---|
| +0x40.. | `vector<C_racing_gate*>` gates | H | `circuit_get_first_gate` returns element 0 = the start/finish line |
| +0x50 | lock state | M | written by `champ_circuit_set_lock_state` — this is the track-unlock mechanism |
| +0x54.. | `vector<C_best_time*>` — the best-lap table, cap 12 | H | Serialised by `circuit_save_best_times` |
| +0x68 | lap count | H | copied from record `+0x40` by `racing_load_circuits`; 7 for every shipped circuit except `trat12`/`trat13` (5) in the mod pack |
| +0x6c[7] | per-car-class qualifying time MIN | H | from record `+0x48`; read by `circuit_get_class_qualify_time_min`. Order proven by the lerp `t*(max-min)+min` in `racing_generate_qualification` |
| +0x8c[7] | per-car-class qualifying time MAX | H | from record `+0x68`; read by `circuit_get_class_qualify_time_max` |
| +0xac.. | `vector<C_web_node*>` **left** racing line | H | loaded as web-path type 2; the side is pinned in §7a |
| +0xbc.. | `vector<C_web_node*>` **right** racing line | H | web-path type 3; the side is pinned in §7a |
| +0xcc.. | `vector<race_cam*>` | H | 12-byte records `{C_web_node* node, C_racing_gate* a, C_racing_gate* b}` |

### 3a. Named methods

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x0060b7a0 | `circuit_ctor` | `__ct__9C_circuitFPcPcPUiPUi` | CONFIRMED |
| 0x0060b880 | `circuit_dtor` | `__dt__9C_circuitFv` | CONFIRMED |
| 0x0060ba00 | `circuit_load_gates_from_scene` | `LoadGatesFromScene__9C_circuitFv` | CONFIRMED |
| 0x0060bda0 | `circuit_get_first_gate` | `GetFirstGate__9C_circuitFv` | CONFIRMED |
| 0x0060bdc0 | `circuit_clear_gates` | `ClearGates__9C_circuitFv` | CONFIRMED |
| 0x0060be10 | `circuit_clear_paths` | `ClearPaths__9C_circuitFv` | CONFIRMED |
| 0x0060be70 | `circuit_clear_race_cams` | `ClearRaceCams__9C_circuitFv` | CONFIRMED |
| 0x0060bed0 | `circuit_load_path` | `LoadPaths__9C_circuitFUiP…vector<P10C_web_node…>` | CONFIRMED |
| 0x0060bfb0 | `circuit_determine_race_cams` | `DetermineRaceCams__9C_circuitFv` | CONFIRMED |
| 0x0060c0a0 | `circuit_save_best_times` | `SaveBestTimes__9C_circuitFR8G_Stream` | CONFIRMED |
| 0x0060c130 | `circuit_load_best_times` | `LoadBestTimes__9C_circuitFR8G_Stream` + `LoadBestTimesDta__9C_circuitFi` | CONFIRMED ² |
| 0x0060c360 | `circuit_get_class_qualify_time_min` | - | PC-only ¹ |
| 0x0060c380 | `circuit_get_class_qualify_time_max` | - | PC-only ¹ |

¹ An accessor the PS2 inlines.
² One PC function serves both PS2 loaders (§7).

### 3b. `C_racing_gate` — one lap gate (0x2c bytes)

`circuit_load_gates_from_scene` builds the gates from the mission's navigation web (`check.bin`).
Starting at the web node with ID `0x27c`, each node of the walk becomes a gate's **track node**;
its neighbour with exactly one connection is the gate's **outer marker**, the neighbour with more
connections is the next node of the walk, and neighbours with node category bit `0x8`
(category value 2, `structs/racecam.md` §2b) are race-camera nodes collected for `+0xcc`. The gates are linked into a ring. A gate prints the
checkpoint time when the first connection of its outer marker carries flag bit `0x80`. Circuit 0
(mission 6) has 27 gates; gates 9, 14 and 20 print.

| Offset | Field | Conf | Witness |
|---|---|---|---|
| +0x00 | `C_web_node*` track node | H | `circuit_load_gates_from_scene`; one end of the gate line |
| +0x04 | `C_web_node*` outer marker | H | the node with one connection; the other end of the gate line |
| +0x08 | `S_vector` midpoint of the two nodes | H | `circuit_load_gates_from_scene` |
| +0x14 | `C_racing_gate*` next gate | H | ring link; `racing_test_gate` |
| +0x18 | `C_racing_gate*` previous gate | H | ring link; `racing_test_gate`, `racing_test_gate_crossing` |
| +0x1c | gate number (index in the gate vector) | H | `race_compare_position`, `racing_get_distance_by_gates` |
| +0x20 | prints checkpoint time (bool) | H | the outer marker's connection bit `0x80`; read by `racing_do_gates` |
| +0x24 | stored split time | H | `racing_do_gates` clears it on every gate at each forward step, so the compare-with-split branch never runs |
| +0x28 | distance to the next gate's midpoint | H | summed by `racing_get_distance_by_gates` |

**The zone test.** `racing_test_gate_crossing` puts a point inside gate *G*'s zone when, on the
X/Z plane, it lies between the infinite line of the previous gate and the infinite line of *G*:
`t = d_prev / (d_prev + d_cur)`, inside when `0 <= t <= 1`. The lines are unbounded, so a zone
has no side limit and appears again, mirrored, past the crossing point of two non-parallel
lines. Zones of different gates therefore overlap widely.

**The gate check.** Every 1000 ms (`C_racing+0x29c`) `racing_do_gates` runs for each racer.
`racing_test_gate` tests the current gate's zone, then the next gate's, then the previous gate's,
always in that order; when none holds the car, `racing_test_all_gates` hands back the
lowest-numbered gate whose zone does. Stepping forward off the finish gate adds a lap unless
`C_opponent+0x45` is set; stepping back onto it subtracts one.

---

## 4. `C_championship` and `C_champ_circuit`

`C_championship` is 0xe0 B; `C_champ_circuit` is 0x74 B (§7). One
championship per record of `tables/championship.def`; one `C_champ_circuit` per round.

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x0060c640 | `championship_ctor` | `__ct__14C_championshipFUi` | CONFIRMED |
| 0x0060c6a0 | `championship_dtor` | `__dt__14C_championshipFv` | CONFIRMED |
| 0x0060c7d0 | `championship_add_circuit` | `AddCircuit__14C_championshipFP15C_champ_circuit` | STRUCTURAL |
| 0x0060ca60 | `championship_set_opponents` | `SetOpponents__14C_championshipFPUiPUi` | STRUCTURAL |
| 0x00615060 | `championship_get_results` | `GetResults__14C_championshipFP10C_opponent` | CONFIRMED |
| 0x0060c3a0 | `champ_circuit_ctor` | `__ct__15C_champ_circuitFP9C_circuitUi` | CONFIRMED |
| 0x0060c3e0 | `champ_circuit_dtor` | `__dt__15C_champ_circuitFv` | STRUCTURAL |
| 0x0060c4c0 | `champ_circuit_save_profile` | `SaveProfile__15C_champ_circuitFR8G_StreamUi` | CONFIRMED |
| 0x0060c520 | `champ_circuit_set_lock_state` | `SetLockState__15C_champ_circuitF13ER_lock_state` | CONFIRMED |

**`C_championship+0x00` is the standings table**: `{C_opponent*, points}` pairs, stride 8,
**21 slots**, scanned by `championship_get_results` and rebuilt by
`racing_fill_champ_results`. That is a 20-entrants-plus-player field (§7). `+0xd0` holds the championship's
difficulty, which `racing_set_current_champ` copies into `C_racing+0x3c`.

`C_champ_circuit+0x04` is the positions-valid flag, `+0x0c[21]` the final positions, `+0x68`
the best-race table (cap 10).

`champ_circuit_set_lock_state` is the **track-unlock mechanism**: winning a round writes
the unlocked state into the round and mirrors it into the `C_circuit`, which is what makes
the track selectable in single-race mode. The racing mod's readme describes exactly this
behaviour from the player's side ("Races can be played in reverse from single race menu
once you unlock them through winning in championship").

---

## 5. `C_opponent` — one racer

The heaviest class of the family: 30 PS2 methods, of which the overtaking AI
(`Ojed*`, `TickObjizdecka`, `ObjzTestNodeValid*`) is more than half. **26 of 30 accounted for.**

**The record opens with `char name[0x20]`** — `opponent_ctor` seeds it with the literal
**"Tommy Angelo"**, which is what identified the field.

| Offset | Field | Conf | Witness |
|---|---|---|---|
| +0x00 | `char name[0x20]` — the driver's display name | H | `opponent_ctor` copies **"Tommy Angelo"** in; the id-taking overload copies a different default |
| +0x20 | `0x3a` seed | U | both constructors write it (§11) |
| +0x24 | entrant id (`-1` = unassigned) | H | `opponent_ctor_with_id` stores it; indexes the opponent name/model roster |
| +0x28 | `C_car*` | H | `opponent_tick`, `opponent_game_init`, `opponent_create_driver` |
| +0x2c | `C_player*` — set only for the player entrant | H | `opponent_game_init` stores it, `opponent_game_done` clears it |
| +0x30 / +0x34 | the two `I3D_model`s: the car model and the driver-man rig | H | both released by `opponent_game_done` |
| +0x38 | qualification time (`-1` = no time) | H | `racing_generate_qualification` writes it, `race_compare_qualification_time` reads it |
| +0x44 | retired-once / place-settled guard | H | `opponent_set_death` |
| +0x45 | start-crossing flag: while set, a finish crossing adds no lap | H | set by both constructors and `racing_init_racers`; `racing_do_gates` clears it when gate 2 becomes current and on a backward finish crossing; `race_compare_position` ranks a racer heading for the finish gate with it clear ahead of one with it set |
| +0x46 | timed-lap flag | H | `racing_do_gates` sets it when gate 2 becomes current or on an untimed lap completion; while set, a finish crossing runs `racing_test_best_lap`; cleared by a backward finish crossing and by `opponent_game_init` |
| +0x48 | `C_driver*` | H | `opponent_tick` deletes it when the car dies |
| +0x4c | zone parameter `t`: 0 at the previous gate's line, 1 at the current gate's line | H | written by `racing_test_gate_crossing`; the final tie-break in `race_compare_position` |
| +0x50 | back-pointer to a **human player's session record** | H | **PC-only.** Written by `racing_add_racer_for_player`; read by `racing_activate_racers`, `racing_find_opponent_by_player` and `race_compare_participant_slot`, which orders on the id byte at `player+0x2c` (`structs/net_session.md`) |
| +0x54 | back-pointer to an **AI opponent slot** | H | **PC-only.** Written by `racing_add_racer_for_ai_slot`; `race_compare_participant_slot` orders on the slot index at `slot+0x28` |
| +0x5c | `C_racing_gate*` current gate: the gate the racer is heading for | H | seeded with the finish gate by `racing_init_racers`, moved by `racing_do_gates`; `race_compare_position` reads its gate number `+0x1c` |
| +0x68 | finishing place | H | `opponent_set_death` |
| +0x6c | lap counter | H | `race_compare_position` |
| +0x7c | current lap / finish time | H | zeroed by `opponent_set_death` |
| +0x80 | retired flag | H | `opponent_set_death` writes it, both comparators sort on it |
| +0x84 | stuck timer | H | `opponent_tick` accumulates it while `car_is_usable(2)` is false |
| +0x88 / +0x8c | recovery step timer / period | H | both constructors seed `0` / `200`; `opponent_tick_recover` adds the frame time to `+0x88` and advances the step counter every `+0x8c` ms (200, then 1000) |
| +0x90 | recovery step counter | H | `opponent_tick_recover`: the player's entrant shows it on the HUD start flag; the car fades in from 0.2 opacity over the steps, and at step 4 it gets its dynamic collisions back and full opacity |
| +0x178 / +0x17c | current overtaking direction bit; accumulated rejection mask | M | the overtaking AI |
| +0x194 | the grid start frame (`"start%i"`) | H | `opponent_game_init` resolves it by name and copies its transform onto the car |

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x0060cda0 | `opponent_ctor` | `__ct__10C_opponentFv` | CONFIRMED |
| 0x0060ce50 | `opponent_ctor_with_id` | `__ct__10C_opponentFUi` | CONFIRMED |
| 0x0060cf00 | `opponent_dtor` | `__dt__10C_opponentFv` | STRUCTURAL ¹ |
| 0x0060cf10 | `opponent_game_done` | `GameDone__10C_opponentFv` | CONFIRMED ² |
| 0x0060cfa0 | `opponent_game_init` | `GameInit__10C_opponentFP17I3D_animation_setUi` | CONFIRMED |
| 0x0060e000 | `opponent_create_driver` | `CreateDriver__10C_opponentFb` | CONFIRMED |
| 0x0060e0d0 | `opponent_recover` | `Recover__10C_opponentFv` | CONFIRMED |
| 0x0060eb40 | `opponent_do_recover` | `DoRecover__10C_opponentFv` | CONFIRMED |
| 0x0060ed10 | `opponent_tick_recover` | `TickRecover__10C_opponentFUi` | CONFIRMED ³ |
| 0x0060ee40 | `opponent_create_driver_path` | `CreateDriverPath__10C_opponentFb` | CONFIRMED |
| 0x0060f750 | `opponent_get_race_points` | `GetRacePoints__10C_opponentFv` | CONFIRMED ⁴ |
| 0x0060f7c0 | `opponent_get_race_points_for_place` | `GetRacePoints__10C_opponentFUi` | CONFIRMED ⁵ |
| 0x0060f830 | `opponent_set_death` | `SetDeath__10C_opponentFb` | CONFIRMED |
| 0x00611630 | `opponent_play_death` | `PlayDeath__10C_opponentFv` | CONFIRMED |
| 0x00611700 | `opponent_tick` | `Tick__10C_opponentFUi` | CONFIRMED |
| 0x006117e0 | `opponent_tick_overtake` | `TickObjizdecka__10C_opponentFUi` | CONFIRMED |

¹ The PS2's is the empty GCC delete-flag dtor.
² Statement-identical.
³ Size ratio 0.32.
⁴ Takes no place; reads the racer's own `+0x68`.
⁵ Takes the place as an argument.

`opponent_set_death` stops after clearing the lap time (§7).

**An entrant's look is its car.** `opponent_game_init` builds the car model's file name from
the car database record selected by `+0x24` (`car_database_get_car_model`, `"%s%02d.i3d"`,
whose two digits are that record's colour variant). No other `C_opponent` field carries a
colour or a skin.

---

## 6. `C_best_time` and the free comparators

`C_best_time` has exactly **one** PS2 symbol, its constructor, and the PC has exactly one
matching function immediately before `circuit_ctor` on both builds. The record is 0x28 B:
`{char name[0x20]; uint time +0x20; uint carId +0x24}`, default holder
**`"mafia test driver"`**.

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x0060b770 | `best_time_ctor` | `__ct__11C_best_timeFv` | CONFIRMED |
| 0x00610d40 | `race_compare_qualification_time` | `compareQtime__FPCvPCv` | CONFIRMED |
| 0x00610db0 | `race_compare_result_points` | - *(PC-only)* | CONFIRMED |
| 0x00610dd0 | `race_compare_champ_points` | `compareChampPoint__FPCvPCv` | CONFIRMED |
| 0x00610e30 | `race_compare_participant_slot` | - *(PC-only)* | CONFIRMED |
| 0x00610eb0 | `race_compare_position` | `comparePos__FPCvPCv` | CONFIRMED |

Three of the comparators are **taken by address only** — `crt_qsort(…, 4, &LAB_…)` — so they have
no `E8` caller at all; every reference to them is a label operand.
`race_compare_participant_slot` is the exception: `race_compare_champ_points` calls it normally.

The two qsort-only comparators are `race_compare_position` (**272 B**) and
`race_compare_qualification_time` (**112 B**), against 328 B / 184 B PS2 methods.

### 6a. Five comparators on PC, three on PS2

The PS2 has `compareQtime`, `compareChampPoint` and `comparePos`. The PC has those three plus
**two file-local helpers with no PS2 twin**, interleaved in the same source block:

* `race_compare_result_points` (`0x00610db0`) orders the results table on the **cached** total in
  `pair[1]`. `racing_fill_champ_results` sums the points into the pairs and then sorts with this;
  the PS2 sorts the same table with `compareChampPoint`, which re-derives every total through
  `GetResults`. The PC authored a cheaper one.
* `race_compare_participant_slot` (`0x00610e30`) orders by **participant slot**: two human
  entrants by the player id byte at `player+0x2c`, two AI entrants by the slot index at
  `slot+0x28`, and a human always sorts before an AI entrant; entrants with neither back-pointer
  compare equal. It is used two ways — `race_compare_champ_points` delegates to it when the points
  are level, and `racing_place_cars` sorts the grid with it directly. It cannot have a PS2 twin,
  because the session slots it orders by exist only on the PC.

---

## 7. PC vs PS2

All cross-build material for these classes lives in this section.

| Area | PC | PS2 |
|---|---|---|
| the optimal-line segment subsystem | present, but in a **second translation unit** at `[0x005c9720, 0x005ca290)` (§10) | in the `C_racing` run, as `C_racing_osegment` + `CreateSegments` / `LoadSegments` / `ClearSegments` / `GetSegments` / `GetSegmentTotalTime` |
| how a qualifying time is derived | interpolates between the circuit's per-class MIN/MAX tables from `circuits.def` | scales `GetSegmentTotalTime(segId, 0)` by a difficulty term |
| `championship.def` record | **0xb8 bytes** | 0xdc bytes |
| `carcyclopedia.def` record | **200 bytes** | 204 bytes |
| `circuits.def` record | 0x88 bytes | 0x88 bytes — the one table that is portable |
| best-times serialisation | writes the driver **name** as a 0x20-byte string via `dtaWrite` | writes a 0x10 record `{name INDEX, +0x20, +0x24}` via `G_Stream::WriteAdr` |
| `C_championship` instance | 0xe0 B | 0x5c B |
| `C_champ_circuit` instance | 0x74 B | 0x50 B |
| `C_circuit` instance | 0xdc B | 0xc8 B |
| championship ctor seed at `+0xcc`/`+0xd0` | `{2, 1}` | `{2, 2}` |
| `SetDeath` tail | stops after clearing the lap time | also writes the championship final position and refills the results |
| best-time loading | `racing_load_best_times` reads `savegame\mrtimes.sav` circuit by circuit through `circuit_load_best_times`; on failure `racing_ctor` saves the defaults the constructors seeded | `LoadBestTimes(0)`; on failure `LoadBestTimes(1)`, which reads the `.dta` archive through `LoadBestTimesDta` (×3 classes); on failure again `CreateBestTimes` (sorted with `compSegID`); then `SaveBestTimes` |
| circuit best-time loaders | one function, `circuit_load_best_times` | two, `LoadBestTimes` and `LoadBestTimesDta` |
| best-time file scope | circuits only | circuits and championships (`C_championship` / `C_champ_circuit` `SaveBestTimes`, `LoadBestTimes`, `LoadBestTimesDta`) |
| next race-camera racer | `racing_tick` calls `racing_set_race_camera_index(current + 1)`, `racing_end_race` calls it with 0 | both sites call `SetRCamIndexFirstRacer`: the first racer not yet placed, else the next one |
| segment completeness | no total-time test anywhere: qualifying times interpolate the circuit tables, `racing_init_racers` derives none, `opponent_create_driver_path` reads the bank directly | `GetSegmentTotalTime` in `GenerateQualification`, `InitRacers` and `CreateDriverPath` |
| recovery length | 4 steps of 1000 ms in `opponent_tick_recover`, constant | `GetRecoverTime` (0..3, from the difficulty) in `TickRecover` |
| wreck models | no preload step in `racing_game_init` (`racing_init_racers`, `racing_place_cars`, `racing_activate_racers`) | `PreloadWrecks` between `InitRacers` and `PlaceCars` loads every racer's wreck model |
| scene web-path node stride | 0x1e B | 0x20 B |
| the class emission run | a run of its own around `0x0060b770`, reached from the `.text` string xrefs | run 1, immediately after `C_racecam.cpp` |
| championship standings table (`C_championship+0x00`) | **21** `{C_opponent*, points}` slots ⇒ 20 entrants + the player | **6** slots ⇒ 5 entrants + the player |
| difficulty levels | **5** (`racing_get_power_coeff_by_difficulty` 0.9/0.9 · 0.93/0.85 · 1.0/1.0 · 1.2/1.3 · 1.3/1.4) | **4** (0.7/0.7 · 0.8/0.8 · 1.0/1.0 · 1.0/1.1) |
| AI rubber-banding law | raw distance: ×0.7 past 100 units, ×0.6 past 150 | difficulty-graded over a 300-unit ramp, and only for racers BEHIND the player |
| grid placement | copies the `"start%i"` frame transform straight onto the car | additionally derives a lateral offset with `PrusecikPrimek` against the circuit's left/right racing lines |
| session back-pointers on `C_opponent` (`+0x50` / `+0x54`) | present — the multiplayer half of the class (§9b) | no trace |
| free comparators | **5** (adds `race_compare_result_points` + `race_compare_participant_slot`) | 3 (+ `compSegID`) |
| `C_game` frame-placement record | **0x34 bytes**, key at `+0x00` | 0x40 bytes, key at `+0x30` |
| `G_CarDatabase` car-list record | **0xa4 bytes** | 0xa8 bytes |
| camera-mode latches | **two** — `g_camera_last_vehicle_mode` (modes 7-15) and `g_camera_last_onfoot_mode` (modes 1-3) | one (`DAT_004f6290`, modes 7-15 only) |
| race phases in `Tick` | **four** — 0 countdown, 1 racing, **2 post-race delay (posts event `0xf`)**, **3 cooldown** | two (0 and 1); anything else falls through |
| gate check period in `Tick` | **1000 ms** | 500 ms |
| neighbour order in `TestGate` | current, next, previous, always | current, then previous first when the racer's last zone parameter is below 0.5, otherwise next first |
| backward crossing of the finish gate in `DoGates` | lap counter −1, start-crossing flag cleared | lap counter kept, start-crossing flag set so the next finish crossing adds nothing |
| circuit 0 gates (`MISE06-AUTODROM/check.bin`) | 27 gates from 860 nodes | the same 27 gates, identical coordinates, from 409 nodes |

**The divergence runs BOTH ways.** The PS2 has the extra championship record fields, the
memory-card profile path and the champ-results tail; the PC has race phases 2 and 3 that the PS2
`Tick` does not have, two comparator helpers the PS2 lacks, the whole session back-pointer layer,
and a championship that holds four times as many entrants. Read this as "a different branch that
the PS2 later extended in some places and the PC extended in others", not as a strict ordering.

**Do not write "the PC inlined it" for the segment methods.** There is positive PC-side evidence
of a *different placement* — a second translation unit (§10) — not of absence.

### 7a. Field pairs

| class | PC | PS2 | field |
|---|---|---|---|
| `C_racing` | `+0x0c` | `+0x5f0` | `vector<C_circuit*>` |
| `C_racing` | `+0x1c` | `+0x5fc` | `vector<C_championship*>` |
| `C_racing` | `+0x30` | `+0xa54` | current `C_circuit` |
| `C_racing` | `+0x34` | `+0xa58` | current `C_champ_circuit` |
| `C_racing` | `+0x38` | `+0xa5c` | current `C_championship` |
| `C_racing` | `+0x3c` | `+0xab0` | difficulty level (5 levels on PC, 4 on PS2) |
| `C_racing` | `+0x40` | `+0x400` | the player's embedded entrant |
| `C_racing` | `+0x1d8` | `+0xa28` | entrant array |
| `C_racing` | `+0x228` | `+0xa3c` | live racer array |
| `C_racing` | `+0x27c` | `+0xab8` | total lap count |
| `C_racing` | `+0x284` | `+0xac0` | entrant count |
| `C_racing` | `+0x288` | `+0xac4` | live racer count |
| `C_racing` | `+0x2b7` | `+0xb07` | damage enabled (PS2 copies it into `zmisse6damage`) |
| `C_racing` | `+0x2b8` | `+0xaf0` | race-camera racer index |
| `C_racing` | `+0x2bc` / `+0x2c0` | `+0xaa0` / `+0xaa4` | race-camera cycle timer / dwell period (PS2 50000 ms) |
| `C_racing` | `+0x420` / `+0x424` | `+0xa90` / `+0xa94` | driver-power update accumulator / period (PS2 500 ms) |
| `C_circuit` | `+0x40` | `+0x4c` | gate vector |
| `C_circuit` | `+0x50` | `+0x00` | lock state (PS2 mirrors it at circuit `+0x00`) |
| `C_circuit` | `+0x54` | `+0x58` | best-lap table |
| `C_circuit` | `+0xac` | `+0x64` | left racing line ¹ |
| `C_circuit` | `+0xbc` | `+0x70` | right racing line ¹ |
| `C_circuit` | `+0xcc` | `+0x7c` | race-camera nodes |
| `C_opponent` | `+0x00` | `+0x15c` | driver name |
| `C_opponent` | `+0x24` | `+0x180` | entrant id |
| `C_opponent` | `+0x28` | `+0x134` | `C_car*` |
| `C_opponent` | `+0x2c` | `+0x140` | `C_player*` |
| `C_opponent` | `+0x30` / `+0x34` | `+0x148` / `+0x14c` | car model / driver rig |
| `C_opponent` | `+0x38` | `+0x188` | qualification time |
| `C_opponent` | `+0x44` | `+0x1df` | place-settled guard |
| `C_opponent` | `+0x45` | `+0x1e0` | start-crossing flag |
| `C_opponent` | `+0x46` | `+0x1e1` | timed-lap flag |
| `C_opponent` | `+0x48` | `+0x144` | `C_driver*` |
| `C_opponent` | `+0x4c` | `+0x1cc` | zone parameter `t` |
| `C_opponent` | `+0x5c` | `+0x13c` | current gate |
| `C_opponent` | `+0x68` | `+0x198` | finishing place |
| `C_opponent` | `+0x6c` | `+0x19c` | lap counter |
| `C_opponent` | `+0x7c` | `+0x1a4` | current lap / finish time |
| `C_opponent` | `+0x80` | `+0x1e4` | retired flag |
| `C_opponent` | `+0x84` | `+0x1a8` | stuck timer |
| `C_opponent` | `+0x88` / `+0x8c` | `+0x154` / `+0x158` | recovery step timer / period |
| `C_opponent` | `+0x90` | `+0x1dc` | recovery step counter |
| `C_opponent` | `+0x194` | `+0x150` | grid start frame |
| `C_racing_gate` | `+0x00` / `+0x04` | `+0x10` / `+0x14` | track node / outer marker |
| `C_racing_gate` | `+0x14` / `+0x18` | `+0x18` / `+0x1c` | next / previous gate |
| `C_racing_gate` | `+0x1c` | `+0x20` | gate number |
| `C_racing_gate` | `+0x20` | `+0x2c` | prints checkpoint time |

¹ The sides are pinned by `RadarSetLeftEdge` taking `circuit+0x64` on PS2 and `+0xac` on PC, and
by `RadarSetRightEdge` for the right line.

**The PC↔PS2 delta is NOT uniform** — `+0x228`↔`+0xa3c` is `0x814`, `+0x288`↔`+0xac4` is
`0x83c`, `+0x2b7`↔`+0xb07` is `0x850`. Do not rebase band-wise; the two builds are different
revisions of the class, not a shifted copy (the same holds for `structs/detector.md` §7 and
`structs/pumper.md` §6).

---

## 8. Racing over the network

**Six message types belong to racing**, which makes the multiplayer racing mode the single
heaviest consumer of the event-message layer after `C_car`:

| type | what | producer |
|---|---|---|
| `0x31` | a lobby setting changed | `net_session_broadcast_setting` |
| `0x32` | an AI opponent added | `net_session_add_ai_racer` |
| `0x33` | an AI opponent removed | `net_session_remove_ai_racer` |
| `0x37` / `0x38` | a racer finished | `racing_do_gates` |
| `0x3a` | the race camera changed | `camera_tick` |

**The type-`0x31` selector map is a free field decode.** Its producer resolves the selector before
posting: 0 = `racing_get_current_circuit_id`, 1 = `g_racing+0x64`, 2 = `session+0x228` (also
written back to `g_racing+0x60`), 3 = `g_racing+0x27c`, 4 = the byte at `g_racing+0x2b4`,
5 = `racing_get_current_champ_id`. Selector 5 additionally prunes every racer whose car fails
`car_database_is_enabled_for_championship`.

**A 21-slot cap.** `net_session_add_ai_racer` refuses once peers + AI racers reach `0x15`.

The type-`0x3a` producer sits inside `camera_tick` (`Tick__8G_CameraFUi`), the `G_Camera`
per-frame driver: it picks `g_active_race_camera` from a range list and writes `g_racing+0x98`.
See `structs/camera.md`.

Full write-up of the message layer: `structs/net_event_msg.md`.

---

## 9. Coverage of the translation unit

`[0x0060c130, 0x00617a60)` enumerates to **120 real starts** — a scan for the decompiler's
Aggressive-Instruction-Finder artifact shapes finds none left in that range — and every body has
been read. The TU as a whole is 131/131.

### 9a. Measured coverage

| class | PC prefix | PC named in TU | PS2 methods | PS2 accounted |
|---|---|---|---|---|
| `C_racing` | `racing_` | 69 | 63 | 46 |
| `C_opponent` | `opponent_` | 26 | 30 | 26 |
| `C_circuit` | `circuit_` | 13 | 14 | 11 |
| `C_championship` | `championship_` | 8 | 11 | 8 |
| `C_champ_circuit` | `champ_circuit_` | 6 | 13 | 6 |
| `C_best_time` | `best_time_` | 1 | 1 | 1 |
| `C_racing_osegment` | - | 0 | 1 | 0 |
| *free functions of the TU* | `race_` / `gif_` / `line2d_` / `seg2d_` | 8 | 7 | 5 |

The figures are counted from the function registry `Game-ghidra.exe.functions.md` and the PS2
symbol table, never asserted.

### 9b. Twelve PC-only functions are the multiplayer half of `C_racing`

`racing_add_racer_for_player`, `racing_add_racer_for_ai_slot`,
`racing_bind_local_player_racer`, `racing_set_player_racer_model`,
`racing_remove_racer_for_player`, `racing_remove_racer_for_ai_slot`,
`racing_remove_opponent_at`, `racing_find_opponent_by_player`,
`racing_find_opponent_by_ai_slot`, plus `race_compare_participant_slot` and the two
`racing_pause_clock_*` helpers.

They all reach into the session object documented in `structs/net_session.md`. A `C_opponent`
carries exactly one back-pointer: `+0x50` to a human player record, `+0x54` to an AI opponent
slot. The distinguishing detail between the two adders is what the entrant brings: a human player
brings a **name string**, an AI slot brings an **index into the driver-name table**.

The other PC-only rows: the replay triple (`racing_get_replay_setup_size` /
`racing_save_replay_setup`, feeding `"savegame\reply%03d\r%02d"`), the winners' scene pair
(`racing_spawn_winners_scene` / `racing_release_winners_scene`, placing the top three at
the `"FMVZavod0%i"` / `"FMVcar0%i"` frames), `racing_next_finish_place`,
`racing_set_phase`, and the two per-class qualifying-time accessors.

### 9c. The segment subsystem is on the PC too — in a second translation unit

Three PC-side facts place it:

1. **`racing_get_segment_id_for_car`** (`0x00615230`) is `GetSegmentIdFromCarIdG__FUi`, at the
   exact matching rank — one of the three free functions in the PS2 gap at `0x003a3080`. It
   returns 0..6.
2. **`C_racing+0x2c4` is an array of seven segment banks.** `racing_ctor` builds them with
   `crt_ehvec_ctor(this+0x2c4, 0x10, 7, gif_vector_ctor, vector_dispose_at_0x04)`,
   `racing_game_done` frees all seven element by element, and `racing_game_init` copies a
   42-dword record out of `[i]` into the live trail at `+0x334`.
3. **The out-of-line segment methods sit at `[0x005c9720, 0x005ca290)`**, called from
   `racing_game_init` / `racing_game_done`. That mirrors the PS2 exactly: the PS2 emits
   `CreateSegments` / `LoadSegments` / `ClearSegments` / `GetSegments` / `GetSegmentTotalTime` in
   a **different link run**, next to `C_racecam`, not in the `C_racing` run.

The full write-up of that cluster is §10.

### 9d. PS2 methods with no PC start in this TU

Each was checked at the position of its PS2 callers inside the matching PC function.

Inlined, and marked in the PC decomp with `//` banners at the exact statement:
`GetLeftPath` / `GetRightPath` (in `racing_game_init` and `racing_activate_racers`),
`GetOpponentName` / `GetOpponentNameIdx` (in `racing_fill_opponent_name_and_model`),
`GetOpponentFromCar` (twice in `racing_predict_car_collision`), the four `OjedMe*` quadrant
arms (in `opponent_overtake`), and the four small `FinalPositions*` siblings (in
`champ_circuit_final_positions_set`).

Absent, with the PC doing something else at the caller's position (§7):
`SetRCamIndexFirstRacer`, `GetSegmentTotalTime`, `GetRecoverTime`, `PreloadWrecks`,
`CreateBestTimes` with its comparator `compSegID`, and the six `C_championship` /
`C_champ_circuit` best-time streamers. The two `C_circuit` loaders are one PC function,
`circuit_load_best_times` (§3a).

### 9e. Field facts

| Where | Offset | Meaning |
|---|---|---|
| `C_circuit` | `+0xac` / `+0xbc` | **left** and **right** track path containers (§3) |
| `C_circuit` | `+0x6c[7]` / `+0x8c[7]` | per-car-class qualifying time **min** and **max** |
| `C_circuit` | `+0x44` / `+0x58` | the gate vector; the best-lap table (cap 12) |
| `C_racing` | `+0x2c4[7]` | the seven segment banks (§10) |
| `C_racing` | `+0x334` / `+0x3ac` / `+0x3d4` | the live position trail: 10 × vec3, 10 × tag, capacity |
| `C_racing` | `+0x3f0[6]` / `+0x408[6]` | the winners' scene actors and their I3D models |
| `C_racing` | `+0x294` / `+0x298` | pause-clock stamp; **race start stamp** (`+0x290 = now - +0x298`) |
| `C_racing` | `+0x2a4` | **race phase**: 0 start lights, 1 racing, 2/3 networked pre-start |
| `C_racing` | `+0x28` vs `+0x2c` | menu mode vs race mode — two different fields |
| `C_opponent` | `+0x38` / `+0x44` / `+0x68` / `+0x7c` / `+0x80` | qualifying time; place-settled flag; finishing place; finish time; retired flag |
| `C_opponent` | `+0x178` / `+0x17c` | current overtaking direction bit; accumulated rejection mask |
| `C_championship` | `+0x00[21]` stride 8 | the `{C_opponent*, points}` results table |
| `C_champ_circuit` | `+0x04` / `+0x0c[21]` / `+0x68` | positions-valid flag; final positions; best-race table (cap 10) |
| `C_best_time` | 0x28 B | `char name[0x20]`, `uint time` at `+0x20`, `uint carId` at `+0x24` |
| `S_car_node` | stride 0x70 | see `structs/driver.md` §3 |

---

## 10. The optimal-line SEGMENT system

The AI racing-line database, its own translation unit at `[0x005c9720, 0x005ca290)` — the second
PC cluster §9c places.

### 10a. What it stores

`C_racing` keeps **seven banks** of segments at `+0x2c4` (stride `0x10`), one per car
performance class; `racing_get_segment_id_for_car` (`0x00615230`) returns the bank index
0..6 from the car's top speed and model id, and `racing_get_segment_bank_for_car` is the
`C_racing` method that resolves it for the player's chosen car (`+0x64`).

Each bank holds pointers to `C_racing_optimal_segment` records:

```c
typedef struct C_racing_optimal_segment {   /* 0xa8 bytes */
/* +0x00 */ S_vector pts[10];    // [H] gate-crossing points along the racing line
/* +0x78 */ float    speed[10];  // [H] the car's speed at each point; seeded 1000.0f
/* +0xa0 */ int      count;      // [H] points actually recorded
/* +0xa4 */ int      time_ms;    // [H] elapsed ms for this stretch; -1 = never recorded
} C_racing_optimal_segment;
```

The class name `C_racing_osegment` does not settle whether its "o" stands for *optimal* or
*opponent*: the records are written by the best-time learner (§10b) and read by the
opponents' path builder (§10e), and no symbol or string of either build spells the word out.
The PC prefix follows the learner.

**`C_racing` EMBEDS one at `+0x334`** as the scratch record being recorded — byte-proven
by `lea ecx,[esi+0x334]` in `racing_ctor` calling `racing_optimal_segment_ctor`. The three
fields after it are the recorder's own cursors: `+0x3dc` current segment index, `+0x3e0`
point write cursor, `+0x3e4..+0x3ef` the last sampled car position.

**The class has NO destructor on either build.** No `__dt__17C_racing_osegment` symbol
exists in the PS2 ELF, and both builds free a record with a bare `operator delete`. The
record owns no pointer, so there is nothing to destroy — this is a measurement, not a gap.

### 10b. How it is filled — a best-time learner, not a recording

`racing_tick` calls `racing_record_segment_points` every frame with the car's position, its
displacement since the last frame and its speed. For every circuit cross-line
(left-path node *i* to right-path node *i*) the movement crossed, the crossing point and the
speed go into the scratch record. When the scratch fills,
`racing_finish_segment_and_advance` commits it into the bank **only if this lap's time for
that stretch beat the stored one** and the learn flag at `C_racing+0x2b6` is set, drops each
point onto the ground with a downward ray, then advances to the next segment.

That min-time gate is what makes the stored data an *optimal* line rather than a trace.

### 10c. Files

| file | direction | when |
|---|---|---|
| `system\mrseg%i.sav` / `system\mrsegm%i.sav` | read | `racing_game_init`, per circuit id |
| `savegame\mrseg%i.sav` / `savegame\mrsegm%i.sav` | write | `racing_game_done` |

The `m` variant is selected by `C_racing+0x2b4`. Reading from `system\` and writing to
`savegame\` is a record-then-ship developer workflow: the shipped data lives in `system\`,
and anything learned during play lands in `savegame\`.

Per bank the format is a segment count, then per segment: point count, segment time, the
points, the speeds.

### 10d. Coverage

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x005c9750 | `racing_optimal_segment_ctor` | `__ct__17C_racing_osegmentFv` | CONFIRMED |
| 0x005c9770 | `racing_record_segment_points` | - | PC-only |
| 0x005c99a0 | `racing_create_segments` | `CreateSegments__8C_racingFv` | CONFIRMED |
| 0x005c9c90 | `racing_save_segments` | - | PC-only |
| 0x005c9e30 | `racing_load_segments` | `LoadSegments__8C_racingFbPcUi` | CONFIRMED |
| 0x005ca040 | `racing_clear_segments` | `ClearSegments__8C_racingFUi` | CONFIRMED |
| 0x005ca0a0 | `racing_get_segment_bank_for_car` | - | PC-only ¹ |
| 0x005ca0f0 | `racing_finish_segment_and_advance` | - | PC-only ² |
| 0x00615230 | `racing_get_segment_id_for_car` | `GetSegmentIdFromCarIdG__FUi` | CONFIRMED ³ |

¹ A wrapper. ² Absorbs `GetSegments`. ³ A free function.

**`racing_optimal_segment_` is a proper extension of the `racing_` class prefix.** Anything that
selects `C_racing`'s methods by `startswith("racing_")` pulls `racing_optimal_segment_ctor` in
unless the longer prefix is excluded — the same situation `car_extended_` and `car_database_`
create for `car_`.

### 10e. Who reads it — the AI racers' driving line

`opponent_create_driver_path` resolves the bank of the racer's car class
(`racing_get_segment_id_for_car`, then `C_racing+0x2c4 + class * 0x10`) and copies each
segment's points and per-point speeds into the racer's `C_driver` path (`structs/driver.md`
§3). The stored line is what the AI racers drive along, at the speeds recorded with it.

---

## 11. Open items

| item | state |
|---|---|
| `C_opponent+0x20` | a constructor seed (`0x3a`) written by both constructors; no reader located (§5) |
| in game (needs the racing mod) | drive a circuit twice, improving on one stretch, and check that `savegame\mrseg<circuit>.sav` grows or changes — that would confirm the learner runs in the shipped build rather than only in a developer configuration |
