# police_manager family — the player-side police engine

**Created** 2026-06-21 19:30 +0200 · **Modified** 2026-09-30 23:08 +0200

Static analysis grounded in the body of `police_manager_tick`. Field names follow
`structs/police_manager.md`; offence storage, escalation and noticing geometry are in
`police-system/artifacts/offences.md` and are not repeated here.

**Scope.** Only the `police_manager_*` family — the helpers whose `__thiscall` `this` is the
`police_manager` (embedded once, at `C_game+0x2da0`), centred on `police_manager_tick`. The
`police_board_*` / `police_action_*` family (per-officer squad AI) is a **separate system**
(`structs/police-board.md`, `police-system/artifacts/police-board-action.md`); where
`police_manager_tick` calls into it, this doc stops at the callee's base behaviour.

**Two parallel per-frame engines.** Each frame `game_tick` runs both:
- `police_board_tick` — per-officer / per-squad AI (out of scope).
- `police_manager_tick(this = C_game+0x2da0, dt_ms)` — **this doc**: the per-frame state
  machine for the `police_manager` and its `records` vector (`+0x40`, a
  `std::vector<pursuit_record*>`). One `pursuit_record` = one tracked **suspect** (the player, or a
  pursued NPC). The player has exactly one record; only the player gets a HUD icon.

`police_manager_tick` VA `0x005cb510`, signature `void __thiscall (police_manager*, int dt_ms)`.

## 1. The three police states — fine / handcuffs / gun

The committed police-state level lives at `pursuit_record+0x3c` (`police_state`):
**1 = fine, 2 = handcuffs, 3 = gun** (4 = searching is a HUD-only override, not stored — §6).
Triggers, per-offence escalation and on-screen strings are in `offences.md` (§2.3 escalation map,
§5 strings). This section covers only what the `police_manager` family does with the level.

| Level | `record+0x3c` | Set/raised by | Notice radius (§2 / offences.md §6) |
|---|---|---|---|
| fine | 1 | pending→commit in `police_manager_tick`; `police_board_raise_*_escalation` (gameplay paths) | 25 m |
| handcuffs | 2 | same | 45 m |
| gun | 3 | same | 65 m |

The level is **monotonic** while a pursuit lives — `police_manager_tick` only ever commits a
*higher* pending level (`+0x3e`) into `+0x3c`; it never de-escalates handcuffs back to fine. The
state is lost only by clearing the whole record (§4).

**Police-car forgiveness.** In the escalation walk, a pending **fine (1)** is dropped back to 0 when
the suspect's linked vehicle is itself a police car (`vehicle+0x1ff2` set). Pending handcuffs/gun
are unaffected — matching the in-game rule that driving a cop car waives minor offences only.

**End of a fine vs end of an arrest.** Both run through the *busted cutscene*, a
`police_action`-owned animation flagged by `police_board_is_busted_cutscene_active`
(`police_board+0x2c`):
- **Fine cutscene ends → cops are lost.** While the busted-cutscene flag is set,
  `police_manager_tick` skips its whole live block and instead calls
  `police_manager_record_clear_pursuit_state(player, 0)` — the fine path wipes the player's pursuit
  state, so after the officer finishes writing the ticket the player walks free.
- **Arrest cutscene → game over.** The arrest path does not return control. Once the post-arrest
  settle timer (`police_board+0x24`, the *arrest-settle* grace, clamped to 12 s on the arrest branch
  only) reaches 0, `police_manager_tick`'s top-of-function **arrest end-screen** block fires: it
  fades in, silences the world (`game_stop_all_sounds`), builds the end-screen UI
  (`gui_game_over_ctor`), and runs the modal `gui_menu_execute_modal` — Load (world action 2) /
  Restart mission (1) / Quit (3). This is the "busted" game-over screen; there is no return to
  gameplay.

## 2. Noticing a crime — `police_manager_try_dispatch_pursuit_for_target`

**Recording ≠ noticing.** An offence *bit* is set the instant it happens, with no witness needed
(the player record exists whenever the manager is enabled), and the bits + police state are driven
entirely inside `police_manager_tick`. **Being noticed** — a real cop/car activated and the red line
printed — is a separate, gated step the dispatcher *attempts* each tick: it tries to find a witness,
and if none qualifies the provisional state is rolled back. The detection *geometry* (foot vs
vehicle path, 25/45/65/50 m radius, below-gun facing cone + LOS raycast, gun-tier omnidirectional,
which string prints) is in **offences.md §6 / §5**; this section is the **sequence**
`police_manager_tick` runs around it.

**The officer list drives the logic.** Throughout `police_manager_tick`, `vec_count_stride_0x4()`
(the generic stride-4 `std::vector` size helper, `this = record+0x74`) returns the **count of
officers assigned to this suspect**. Every gate below keys on that count.

### The per-tick noticing pipeline (one suspect, inner tick ≈ 3 Hz)

1. **Derive `pending` (`+0x3e`).** The escalation walk re-derives the pending police state
   (1 fine / 2 handcuffs / 3 gun) from the suspect's **current** offence bits. `pending` is
   **zeroed at the end of every pass** — it always reflects *this* tick's offences (escalation map:
   offences.md §2.3).

2. **Commit `pending → committed` (`+0x3c`) — provisionally.** Only ever escalates
   (`committed < pending`):
   - **No officer assigned yet** (`record+0x74` empty) → commit **unconditionally**. This is
     provisional — it arms the radius so step 3 can *try* to find a witness; step 5 rolls it back if
     no cop qualifies.
   - **Officers already assigned** → commit only if the suspect was **seen recently**
     (`police_board_has_recent_sighting`, or identity already known), so losing sight of the suspect
     cannot keep escalating the level. Handcuffs/gun first promote the matching squads.

3. **Pick the dispatch level, then try-dispatch.**
   - `wanted_substate_flag` (`+0x50`) clear → dispatch on **pending** (`+0x3e`).
   - `+0x50` set **and** identity known (`+0x51 == 0`) → dispatch on **committed** (`+0x3c`).
   - `+0x50` set **and** identity unknown → a wanted-substate variant (only when both `+0x3e` and
     `+0x3c` are non-zero) = "caught offending again while searching".

   If the chosen level is non-zero, call `police_manager_try_dispatch_pursuit_for_target`. Its search
   **radius comes from the committed state** (`+0x3c`): 25 m fine / 45 m handcuffs / 65 m gun /
   50 m default.

4. **Activate cops/cars — the actual "noticing".** The dispatcher runs two independent paths, each
   gated by range + (below gun) facing + LOS (geometry: offences.md §6):
   - **Foot — `pedestrian_manager_spawn_cop_from_pedestrian`:** promote a dormant ambient cop dummy
     (category `npc+0x15a ∈ {1,3}`) into a real officer — alloc a `0x128c` NPC, equip weapon id 8
     (S&W M10 revolver), mark `npc+0x1a5=1`, despawn the dummy, push onto `npc_list` (`+0x80`) — and
     collect it into an out-list.
   - **Vehicle — `traffic_car_activate_or_spawn_pursuit_vehicles`** (only if ≥2 officers are free in
     the pool): reactivate a pooled patrol car (construct only on a pool miss) and crew it with 2
     cops.

   The dispatcher then **binds each activated cop to `record+0x74`**
   (`police_manager_record_add_officer`), prints the red offence line (`action_code + 0x834` foot /
   `+0x866` car), and forms a squad. **An offence is noticed only if ≥1 cop/car passed its
   gates.**

5. **Reconcile by officer-count delta.** The tick snapshots `record+0x74`'s count *before* and
   *after* the call:
   - **Count unchanged (nobody activated):** restore the pre-dispatch `wanted_bar_remaining`
     (`+0x44`) — the bar keeps draining. If the suspect is **still untracked** (count 0) and its bar
     was already drained (`+0x44 < 0`), **roll the provisional `committed` back to 0** and set
     `+0x44 = -1`. With `committed == 0` the HUD block hides the icon — **nothing sticks**; the next
     tick re-derives and tries again.
   - **Count rose (a cop newly saw the suspect):** snap `g_police_wanted_bar` to `1.0`; `committed`
     persists → the police-state icon shows (§6).

**Net result.** The police-state icon appears — and stays — **only** once
`police_manager_try_dispatch_pursuit_for_target` activates at least one cop/car for this suspect.
Until then the committed state is provisional and is rolled back every tick. The officer-count delta
in step 5 (not a raw LOS query) is also the wanted bar's refill trigger.

**Two details handled inside the dispatcher:**
- The suspect-position and `150.0f` arguments the tick passes are **vestigial** — passed but never
  read; all detection geometry is in the callee (offences.md §6). `150°` is *not* a vision cone.
- Bit 15 (assault on a police car) is **consumed** inside the dispatcher (offences.md §2.1 / §5.3).

## 3. `police_manager_tick` — phase map

The function is structured as independent rate-divided accumulators (period/accumulator pairs on the
`police_manager`; defaults from `police_manager_ctor` / `police_manager_game_init`), not nested
timers. In execution order. The first three phases always run; the rest need `enabled` and an
unpaused world, and the phases marked *live* also need world load phase ≥ 10 and no active busted
cutscene (`!busted_cutscene_active`):

| Phase | Gate / rate | What it does | Key callee |
|---|---|---|---|
| OUTER NPC cull | `+0x14/+0x18` ≈ 1.1 Hz; **always** | despawn the far members of `npc_list` | `police_manager_npc_list_tick(0)` |
| radar-vehicle cap | every frame | keep `tracked_police_cars` (`+0x90`) ≤ 5 radar-tracked cars | `hud_radar_remove_car` |
| arrest end-screen | settle timer == 0 | the game-over menu (§1) | `gui_menu_execute_modal`, `gui_game_over_ctor` |
| dormant scrub | world load phase < 10 | clear transient offence bits per record (keep sticky 13/14) | - |
| VISIBILITY refresh | *live*; `+0x1c/+0x20` ≈ 4.6 Hz | per-suspect **red-light** detection; last-seen pos | `roads_get_near_node` |
| INNER state machine | *live*; `+0x04/+0x08` ≈ 3 Hz | re-derive live bits, decay timers, prune officers, drive bar | (see §4) |
| roadblock scan | *live*; inside inner overflow | per visibility-cell roadblock deploy (§4) | `police_manager_spawn_roadblock_in_neighbor_cells` |
| escalation + commit | *live*; inside inner overflow | offence bits → pending → committed; dispatch (§2) | `police_board_promote_*_escalation` |
| HUD icon + reinforcement | *live*; player record only | icon symbol/alpha + backup waves (§4, §6) | `hud_set_police_symbol_and_alpha` |
| WANTED-BAR fill | *live*; end, player record | smooth `g_police_wanted_bar` fraction (§6) | - |

The **INNER tick** is the heart: per suspect it re-derives the live bits (weapon-equipped bit 0 from
equipped-weapon id `+0x4a0` ∈ [5,15]; speeding bit 2 from vehicle speed vs `police_manager+0x0c`),
counts every offence-decay timer down and clears its bit on expiry, prunes officers whose death latch
(`officer+0x5e`) is set, and then runs the wanted-bar / escalation / HUD blocks below. The full bit map and
decay windows are offences.md §2–§3.

## 4. Wanted substate — what `police_manager_tick` handles

The wanted substate ("all cops alerted") is `pursuit_record+0x50` (`wanted_substate_flag`) plus the
on-screen `g_police_wanted_bar`. The flag is *set* elsewhere
(`police_manager_record_begin_wanted_substate` on sustained pursuit, showing string 2200; or
`police_manager_escalate_handcuffs_with_optional_wanted` for the `forcearrest` command).
`police_manager_tick` is what *runs* it each frame:

**Bar drain / refill** (inner tick, per suspect, only while `record+0x3c != 0`):
- **No officer assigned** (`record+0x74` empty — assigned-to-pursue, *not* merely has-LOS; a
  searching-state cop can see you with nobody assigned) → **drain** `wanted_bar_remaining` (`+0x44`)
  by `dt`. If `wanted_substate_flag` (`+0x50`) is clear, the bar is forced to the `-1` sentinel (no
  countdown outside the substate). While draining, the shooting/collision/violence decay timers are
  capped to 2000 ms so a brief re-sighting during the getaway cannot resurrect a stale offence.
- **Officer assigned and `+0x50` set** → **refill** `+0x44` to the state-keyed max (`+0x48`):
  fine 20 000 / handcuffs 40 000 / gun 60 000 ms.

**Cops lost when the bar drains.** When `+0x44` goes negative (fully drained while unseen), the tick
calls `police_manager_record_clear_pursuit_state(record, 1)` — wiping the offence bits, both
police-state slots (`+0x3c`/`+0x3e`), the substate flag, the run-over count, etc. The pursuit fully
clears and both the icon and bar disappear; a later re-sighting re-arms from scratch.

**Reinforcement patrol cars** (HUD-icon block, player only — gated on the icon index, §6):
- icon index 5 (handcuffs + substate) → if `reinforcement_counter` (`record+0x5c`) < 1, set it to 1
  and spawn one patrol car crewed by two cops as a handcuffs-tier squad via
  `police_manager_spawn_patrol_car_reinforcements_at_radius(player, 4, 200.0f)`; the `4` is the
  squad's opening tier (`structs/police-action.md` §3).
- icon index 6 (gun + substate) → if `+0x5c` < 2, set it to 2 and spawn two such cars as gun-tier
  squads (mode 0) at radius 200.
  Waves escalate once per tier; `+0x5c` is the wave latch (0/1/2).

**Roadblocks** (inner overflow, per world visibility cell at `world+0x1b8`, stride 0x14): the tick
finds the nearest roadblock-eligible suspect — `linked_vehicle != 0` **and**
`roadblock_eligible_flag` (`record+0x2c`, set by `police_manager_enable_roadblock_flag_for_target`
only when substate-active **and** police state ≥ gun) **and** identity known (`+0x51 == 0`). If such
a suspect sits within ~150 u² of an un-engaged cell, the cell is engaged and
`police_manager_spawn_roadblock_in_neighbor_cells` deploys cops + patrol cars + spike strips in the
neighbour cells (a roadblock is the composite — `structs/police_manager.md`). When the suspect
leaves, the cell disengages and its queued cops are released back to `npc_list`. Roadblocks are
therefore **gun-tier only**.

## 5. Identity-pending (the searching swap)

`identity_pending` (`record+0x51`): **1 = identity UNKNOWN (searching), 0 = KNOWN**. It is the flag
the HUD reads to draw the searching icon. `police_manager_tick`'s HUD-icon block recomputes it every
inner tick for the **player record**, branching on the number of LIVING officers assigned to the
player (`police_manager_count_officers_alive_for_target`: the `record+0x74` entries whose death latch
`+0x5e` is clear). The selector is an assignment count, not a visibility test:

- **Living officers assigned** → identity from the chasing squads (`structs/police-action.md`):
  - no squad targets the player (`police_board_has_group_for_target` is false) → KNOWN (0).
  - a squad targets him → `+0x51 = !police_board_target_matches(...)`: KNOWN when some squad holds
    his current vehicle in `+0x64`, or he is on foot and some squad has `+0x68` set.
- **No living officer assigned** → identity from the copies an ended squad handed back, and the test
  differs by foot vs car:
  - **On foot** (`actor+0x98 == 0`): `+0x51 = (identity_on_foot_byte (record+0x58) == 0)`.
  - **In a car**: `+0x51 = (identity_tracked_vehicle (record+0x54) != actor+0x98 current car)`.

`+0x54` and `+0x58` are copies of a squad's `+0x64` / `+0x68` (the latter 0 or 1), written by
`police_manager_record_set_tracked_vehicle_for_target` when the squad ends (`police_board_end_pursuit`,
the with-car path of `police_action_disband`). The squad fields themselves are written by
`police_board_create_and_engage`, `police_board_refresh_target_vehicle` and the identity batch of
`police_board_update_sighting` (`structs/police_manager.md` §3.2). This reproduces the documented
transitions (`police-system/artifacts/police-system.md` §"Transparent (Faded) Substate"):

- **identity_pending → 1 (UNKNOWN / searching):**
  - Player **seen only in a car, then leaves it unseen** → the squads hold that car in `+0x64` and
    `+0x68 = 0` → unknown on foot.
  - Player **seen on foot, then enters a car unseen** → no squad holds the new car → 1.
  - Player **switches cars unseen** → the held vehicle is not the current car → 1.
- **identity_pending → 0 (KNOWN / identity established):**
  - a witnessed vehicle entry or exit runs the identity batch, so the squads hold the current state;
  - on foot, a squad (or, with no living officer, `+0x58`) carries the face-known flag;
  - in a car, the current car is the held one.

The `+0x51` writers form a closed set — a byte-store census of the whole binary finds six stores on
a `pursuit_record`: the four in `police_manager_tick` (`0x005cc77b`, `0x005cc786`, `0x005cc7a4`,
`0x005cc7b1`), `police_manager_register_actor` and `police_manager_record_clear_pursuit_state` (both
write 0). An in-car assault on an officer keeps the player known through two traced paths, the
`entity_hit` identity batch and the sticky bit-13 re-dispatch (`structs/police_manager.md` §3.2).

## 6. Police HUD update (icon + wanted bar)

All HUD writes are **player-record only**, in the HUD-icon block of the inner tick.

**Police icon — two independent dimensions:**
1. **Symbol = identity / level.** The index starts from committed `record+0x3c` (1/2/3); the
   force-handcuffs bit (offence bit 14) can promote a sub-handcuffs icon to 2, the assault-officer bit
   (13) to 3. Index 0 → icon off (`g_hud_flags &= ~0x04000000`), skip the rest. Otherwise
   `g_hud_flags |= 0x04000000` (icon on). The displayed symbol is then `local_a8`: normally 1/2/3 =
   fine/handcuffs/gun, **but** if `identity_pending` (`+0x51`) is set **and** the
   `busted_cutscene_lock` (`police_manager+0x38`, 1 while the player is in a busted cutscene) is
   clear, the symbol is forced to **4 = searching** (a pursued suspect is visible but cops cannot tell
   which body is the player). Searching is a render-time override, never stored in `+0x3c`.
2. **Intensity / alpha = visibility.** Default `1.0f` (opaque = player currently seen). If
   `police_board_has_recent_sighting` returns 0 (no LOS within the ~2 s window), alpha is dimmed to
   `0.4f` (transparent = not recently seen).

Both are pushed with `hud_set_police_symbol_and_alpha(symbol, alpha)`.

To drive the reinforcement waves (§4), the same block computes a combined index: the 1/2/3 level plus
**+3 when `wanted_substate_flag` (`+0x50`) is active**, giving 4/5/6 = the substate variants
(5 = handcuffs + substate, 6 = gun + substate) that gate the backup spawns.

**Wanted bar:**
- **Show/hide** (inner tick, earlier block): for the player record, if police state is 0 **or** the
  substate flag is clear → `g_hud_flags &= ~0x10` (hide bar); else `g_hud_flags |= 0x10` (show bar).
  Bit 0x10 of `g_hud_flags` is the police progress-bar enable.
- **Value** (final phase of the function): the smooth `g_police_wanted_bar ∈ [0,1]` is computed for
  the player record only, gated on `+0x50`:
  `g_police_wanted_bar ≈ (wanted_bar_remaining − police_manager.tick_acc_inner) /
  (wanted_bar_max − police_manager.tick_period_inner)`, clamped to [0,1] (the `−acc`/`−period` terms
  are sub-tick smoothing). It returns the full `1.0` when remaining ≥ max. Because the tick rewrites
  this global every frame, a script's `pm_setprogress` write is clobbered on the next tick.
  `g_police_wanted_bar` = `_DAT_00661a10` = hud-singleton `+0x96e0`.

## 7. Cross-references

- `police-system/artifacts/offences.md` — offence storage (`1<<type`), escalation map (§2.3),
  noticing gates (§6), string printing (§5). The detection black box this doc wraps.
- `structs/police_manager.md` — `police_manager` (§2) + `pursuit_record` (§3, incl. §3.2 identity
  recompute) field layout.
- `structs/police-board.md` — the out-of-scope squad-AI singleton (busted cutscene flag,
  arrest-settle timer).
- `police-system/artifacts/police-system.md` — gameplay-level behaviour, the wanted substate, tick
  diagram, textdb strings.

## 8. Open items

- **Which identity path an in-car assault takes.** Leaving the car after assaulting an officer
  keeps the player KNOWN. Both mechanisms are traced (§5); which one fires in a given assault needs
  an in-game check, with the discriminators listed in `structs/police_manager.md` §9.
