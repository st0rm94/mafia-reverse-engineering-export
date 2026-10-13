# Police System — Mafia: The City of Lost Heaven

**Created** 2026-08-02 11:41 +0200 · **Modified** 2026-10-12 21:00 +0200

The police / pursuit system as observed in gameplay, and the engine code behind it. The per-bit
offence table is `police-system/artifacts/offences.md`; the struct layouts are
`structs/police_manager.md` (a single instance embedded in the `C_game` world object at
`C_game+0x2DA0`, holding one `pursuit_record` per tracked suspect) and `structs/police-board.md` (the
global singleton at `0x006552a8`).

## 1. Police states

There are four police states. Each is shown by an icon at the top of the screen.

### 1a. Fine

- Triggered by minor offences. The police want to give the player a fine.
- If the player is on foot next to a police officer, an animation plays in which the officer issues
  the fine.
- If the player is stationary in a car, the police wait next to the car and do not pull the player
  out.
- The player can receive a maximum of 3 fines in a single map / mission (loaded `scene2.bin`). On the
  4th offence, the handcuffs state is triggered automatically.

### 1b. Handcuffs

- Triggered by a bigger offence. The police want to arrest the player.
- A patrol car will try to ram the player's car to stop it.
- Officers will pull the player out of the car when stationary.

### 1c. Gun

- Police will hunt and shoot the player on sight, including shooting from patrol cars.
- Officers will pull the player out of the car when stationary.

### 1d. Searching

- Icon: a confused cop with question marks.
- The police are searching for the player but do not know their identity.

The stored encoding is **1 = fine, 2 = handcuffs, 3 = gun**, anchored to `PM_SHOWSYMBOL`
(`program_process_command` case `0x146` → `hud_set_police_symbol_and_alpha`). **4 = searching** is a
HUD override applied in `police_manager_tick`, not a stored level.

## 2. Wanted substate

Indicated by an additional bar shown under the icon.

- All officers have been alerted.
- Pedestrian cops can raise the alarm by using a phone booth or alarm phones.
- On start, a single police car is always spawned and will arrive at the player's location.
- Can occur for the **handcuffs**, **gun**, and **searching** states.
- Can very rarely occur for the **fine** state — probably a bug / unintended occurrence.
- When the substate starts, the string with id `2200` from `textdb` is written on the screen.
- If the player has not been seen by a cop / patrol car for some time, the bar starts to decrease.
  - When the bar empties, the police state is completely lost.
  - If the player then encounters a cop / patrol car, the state is reinstated.

The bar is `pursuit_record+0x44` (`wanted_bar_remaining`) over `+0x48` (`wanted_bar_max`); the
on-screen fill is `g_police_wanted_bar = remaining / max`, written by `police_manager_tick` while the
player is unseen. Measured drain: **40 s handcuffs / 60 s gun**, verified in game.

### 2a. Wanted with gun

- Police create a roadblock with cars and spike strips (`barrier.4ds`) to stop the player's car when
  driving.
- Patrolling cops can throw a spike strip (`barrier.4ds`) at the player.
- Cops in patrol cars spawned during the substate, or pedestrian cops spawned next to roadblocks,
  carry different weapons:
  - Weapon id `10` — Thompson 1928
  - Weapon id `11` — Pump action shotgun
  - (See `def-files-docs/predmety.md`.)
- Roadblock composition, as observed:
  - 0 or more spike strips (`barrier.4ds`)
  - patrol cars
  - officers carrying a pump action shotgun or a Thompson 1928
  - the places are predefined

## 3. Transparent (Faded) Substate

When the player is not in the direct sight of a cop / patrol car, the icon becomes transparent.

This substate is what drives the transitions between **fine / handcuffs / gun** and the **searching**
state:

- If the player was seen committing a crime **only while in a car**, and leaves the car during the
  transparent substate, the state changes to **searching**.
  - Exception: if the player hit a pedestrian cop with a car and then leaves it, the searching state
    is **not** given.
  - If the player re-enters the same car, the previous state is restored.
- If the player was seen committing a crime **on foot**, and enters a car during the transparent
  substate, the state changes to **searching**.
  - If the player then leaves the car, the previous state is restored.
- While in the **searching** state, if the player is noticed committing a crime again, the previous
  state is restored.

In code the two dimensions are separate. **Transparency** is `police_board_has_recent_sighting`: no
officer assigned to the player has logged a sighting of him in the last 2000 ms. **Searching** is
`pursuit_record+0x51`, recomputed by `police_manager_tick` from what the chasing squads know: each
squad stores the vehicle it knows the player by (`police_action+0x64`) and a face-known flag
(`+0x68`), and the player is recognised in that vehicle, or on foot when the flag is set. Each rule
above follows from those two fields; the full recompute, the writers and the two paths behind the
pedestrian-cop exception are in `structs/police_manager.md` §3.2.

## 4. Transitions

- De-escalation is **not** present — e.g. handcuffs never change back to fine.
- The transition table is in `police-system/artifacts/offences.md`.

## 5. Models and spawn pools

### 5a. Police NPC models

- In the `/models` directory, matching `pol%d%d.d4s`.
- Normal / free-roam pedestrian officers are determined by Actor type 18 (Pedestrian) stored in
  `scene2.bin` for each mission.

### 5b. Police car models

| Model | In-game name | Real manufacturer | Notes |
|---|---|---|---|
| `poli00` / `poli10` | Schubert Six Police | Chevrolet | older and less powerful than the two Extra Six models |
| `polimFor00` / `polimFor10` | Schubert Extra Six Fordor Police | Chevrolet | - |
| `polimTud00` / `polimTud10` | Schubert Extra Six Tudor Police | Chevrolet | - |
| `polCad00` / `polCad10` | Lassister V16 Police | Cadillac | highway patrol heavy vehicle |

Cars used for normal / free-roam patrols are determined by Actor type 12 (Traffic) stored in
`scene2.bin` for each mission; `polCad` is never used there.

### 5c. Vehicle spawn pools and `policemanager_on yy`

The `yy` parameter of `policemanager_on` (script command, case `0x86`) selects the **backup spawn
pool** used during the wanted substate, not the normal patrol pool:

- Normal free-roam / patrol traffic always spawns `poli00`-class cars regardless of `yy`.
- The heavy pool selected by `yy` is **only** consulted when the police spawn reinforcement / backup
  cars during an active wanted substate.
  - `yy == 0` (or omitted) → backup cars are drawn from the basic pool: `poli00` dominant, with
    occasional `polimFor00` / `polimTud00`.
  - `yy != 0` (scripts use `1` or `2`; the engine booleanizes both) → backup cars come from the heavy
    pool only: `polimFor00`, `polimTud00`, `polCad00`. This is the only way `polCad00` ever appears,
    which matches the in-game rule that the Cadillac is exclusive to the **gun** state.

Confirmed by gameplay observation and by the code: the vehicle-pool `switch (rand_range())` inside
`police_manager_apply_state` reads `*(this+0x39)` (the booleanized `yy`) to pick the model name, but
that site is reached only from the wanted-substate spawn branch (gated by the `cStack_10 != '\0'`
flag); ordinary patrol spawns never enter it.

## 6. Radar / minimap vehicle colours

While the player is in a car, the top-left radar shows nearby vehicles as coloured blips (colours
user-confirmed in game; constants verified at the callsites):

| Colour | RGBA | Vehicle |
|---|---|---|
| **White** | `0xffffffff` | the player's **own** car (the one he is driving) |
| **Pale green** | `0xffb0e0b0` | ordinary traffic car |
| **Orange** | `0xffff9f00` | gangster car (Free Ride) |
| **Light blue** | `0xff009fff` | police car (**not** chasing) |
| **Red** | `0xffff0000` | a car **chasing the player** |
| **Light grey** | `0xffc0c0c0` | a **parked** car (no driver / unoccupied) |

A patrol car's blip goes **light blue → red** the moment it begins a pursuit, and reverts to light
blue when the chase ends. **Red is not police-exclusive** — mission-related **gangster cars** chasing
the player are also red.

Implementation: blip colours are written by `hud_radar_add_car_with_color(vehicle, rgba)`
(VA `0x0054c630`), which updates a stride-8 `(entity, colour)` list on the HUD singleton at `+0x9494`;
`hud_radar_remove_car` removes a vehicle's entry, reverting it to its default colour. Callers: the
traffic-vehicle spawn/streaming routine (VA `0x005012e0`) sets the per-type base colour (pale green /
orange / light blue); the player-car path sets white; the pursuit/spawn paths
(`police_manager_try_dispatch_pursuit_for_target`, `police_manager_deploy_vehicle_from_pool`) set red
on chase start. A separate vehicle-cache site sets `0xffc0c0c0` light grey for parked / unoccupied
cars. Full HUD field map: `structs/hud.md`.

## 7. Offence bits — the `get_pm_state` in-game sweep

The script command `get_pm_state xx, yy, zz` writes `flt[yy] = 1.0` iff actor `xx` has offence bit
`1 << zz` set (full storage model + per-bit code evidence in
`police-system/artifacts/offences.md` §2–§4). The bits were swept one by one in normal play with a
debug-print test script, confirmed not to interfere with the result. Observed:

| `zz` | Offence (observed) | Notes |
|---|---|---|
| 0 | **gun or throwable drawn** | live flag, no decay: `police_manager_tick` mirrors equipped-weapon id `+0x4a0` ∈ [5,15] (guns + throwables); melee weapons do not set it. `offences.md` §2.4 |
| 1 | shooting | - |
| 2 | speeding | set above **17 m/s** |
| 3 | **world / environment contact** | car touching road edges, tram tracks, scenery. Very buggy — set almost constantly while driving, decays slowly (stays set after leaving the car). **Police ignore it** |
| 4 | crashing into any car | - |
| 5 | hitting a pedestrian with a car | - |
| 6 | driving on the sidewalk | **Police ignore it** — the escalation/print is unreachable in the shipped EXE (the off-by-800 sidewalk bug; `offences.md` §8) |
| 7 | - | never observed set; no code sets it (unused) |
| 8 | - | never observed set; no code sets it (unused) |
| 9 | **melee / violence vs any pedestrian** (incl. cops) | punches and melee weapons set it; **gunfire does NOT** (gunfire sets bit 1). The Mafia Script Data Reference's "attack on a policeman" label is just the police-relevant case |
| 10 | running a red light in a car | - |
| 11 | stealing a car | - |
| 12 | traffic-sign damage | any form — car-hit **or** gunfire both set it |

**`zz ≥ 13` reads bit 12.** The script loader clamps the operand: `program_decode_params` case
`0xae` (`0x005b0428`..`0x005b043d`) stores the mask `1 << min(zz, 12)`, and a negative `zz` gives mask
1. So `get_pm_state zz = 13/14/15/16` all read the traffic-sign bit (12) and return 1 exactly when it
is set; the command is trustworthy for `zz = 0..12` only. This is also why the Mafia Script Data
Reference documents bit 13 as a duplicate "traffic sign" type. The high escalation bits 13 and 15
*are* real police-internal flags (assault on officer / assault on car) but cannot be queried this
way; see `offences.md` §2.2 and §4.

## 8. Script commands

**Non-player actors are unused.** Although several commands below suggest the police system should be
able to react to actors other than the player, this never happens in the final version of the game.
Mission scripts sometimes contain `policemanager_add` for non-player actors, but in every observed
case the call is commented out. `policemanager_forcearrest`, `policemanager_add` and
`policemanager_del` are also buggy: cops already spawned before the call keep reacting to the player
even after he no longer has an active police state (no icon on screen).

- `police_speed_factor nn` — speed coefficient of police cars during a chase (cops get faster than
  normal). `nn` is a float in `<0,1>`. Example: `police_speed_factor 0.8`.
- `police_support ww, xx [, yy]` — spawns one patrol car crewed by two cops at radius `ww` around
  actor `xx` and engages it as a squad on him: `yy != 0` opens the squad at the gun tier, otherwise
  at the handcuffs tier (`structs/police-action.md` §3). Engine entry
  `police_manager_spawn_patrol_car_reinforcements_at_radius`. No shipped script calls it.
- `policeitchforplayer xx` — writes to `flt[xx]`: **1** if police are chasing the player AND know his
  identity (fine / handcuffs / gun state), **0** if there are no police or police are only in the
  searching state (identity not yet known). Opcode **300** (`0x12c`). Implementation: the case body
  calls `police_manager_count_officers_alive_for_target` and
  `police_manager_target_identity_pending`; `police_manager_count_officers_alive_for_target` counts
  the living officers assigned to the player (`pursuit_record+0x74`), not officers that currently
  see him. The result is 1 iff **(a)** no living officer is assigned while a pursuit on him is
  active (gated by `police_board_is_busted_cutscene_active`), OR **(b)** living officers are
  assigned AND identity is already locked in (`police_manager_target_identity_pending` returns 0).
  It writes **0** otherwise: no pursuit at all, or officers are assigned but the identity is still
  pending (the searching state). The case body's `script_pred_c = (script_pred_c == '\0')` is
  **not** a logical inversion of the script-side meaning — it converts "pending" into "known" before
  writing the result. Corroboration: with living officers and a chasing squad,
  `police_manager_tick` sets `+0x51` as `(police_board_target_matches == '\0')`, i.e. set while no
  squad recognises the player, which makes `+0x51` a "not recognised" flag; and `scene2.bin` scripts use
  `policeitchforplayer 0; if flt[0]=1, -1, 2` to hold the player in a dialog loop while the cops
  still know him, releasing on `flt[0] == 0`.
- `policemanager_add "ssss"` — police start reacting to the named actor's behaviour. `ssss` is the
  actor name (not id). Example: `policemanager_add "Tommy"`.
- `policemanager_del "ssss"` — police stop reacting to the named actor's behaviour. Example:
  `policemanager_del "Tommy"`.
- `policemanager_forcearrest xx, yy, zz` — despite the name this does **not** force an arrest; it
  applies a **forced handcuffs escalation** to actor `xx`, with an optional wanted substate. A
  **single** enabling call produces a normal handcuffs (+ optional wanted substate) that decays
  normally; it is not a self-reinstating toggle. If the actor is not the player, they must first be
  registered via `policemanager_add`. Engine entry
  `police_manager_escalate_handcuffs_with_optional_wanted`. Arguments: `xx` = target actor (always
  `0` = player in shipped scripts); `yy` = handcuffs-escalation enable (`1` sets offence bit 14 →
  `police_manager_tick` raises pending to handcuffs; `0` clears it); `zz` = the also-raise-wanted-
  substate lever, read **only when `yy != 0`** (`1` additionally sets the wanted-substate flag
  `record+0x50`; `0` or omitted = handcuffs only). Verified in game: `forcearrest 0,1,0` → handcuffs
  without the wanted substate; `forcearrest 0,1,1` → handcuffs with it. The handcuffs **icon is not
  instant** — bit 14 only sets *pending* handcuffs, and `police_manager_tick` commits it (the icon
  appears) **only once a cop sees the player**. `forcearrest 0,1,0` therefore shows handcuffs only
  after the player is sighted; `forcearrest 0,1,1` tends to show sooner because the wanted substate
  brings cops that sight him. Examples: `policemanager_forcearrest 0, 1, 1` (handcuffs + wanted
  substate on the player); `policemanager_forcearrest 0, 0` (cancel).
- `policemanager_on [xx], [yy]` — enables the police system. Both parameters optional (default 0).
  `xx` is vestigial: stored at `police_manager+0x34` but never read by runtime code; always 980000
  when supplied. `yy` = the backup-vehicle severity tier (§5c). Auto-registers the player actor.
  Example: `policemanager_on 980000, 2`.
- `policemanager_setspeed xx, yy` — sets the speeding thresholds in km/h: `xx` = fine, `yy` =
  handcuffs, stored as m/s at `police_manager+0x0c` / `+0x10` (defaults 17 / 28 m/s). Engine entry
  `police_manager_set_speed_thresholds`.
- `vlvp xx` — writes to `flt[xx]`: **1** if there is an active police state (searching / fine /
  handcuffs / gun) AND police can currently see the player, **0** if there are no police or police
  cannot see the player. Equivalently: 1 iff the on-screen icon is opaque, 0 if transparent or
  absent. Opcode **`0x122`** (= 290); the case body calls `police_board_has_recent_sighting`.
- `get_pm_numpredators xx, yy` — writes to `flt[yy]` the number of cops chasing actor `xx`,
  regardless of whether they are confused/searching or actively seeing the player. 0 means no one is
  chasing (no police state at all, or the wanted bar has fully drained).
- `get_pm_humanstate xx, yy` — writes to `flt[yy]` the **committed police-state level** of actor
  `xx`: **0** none / **1** fine / **2** handcuffs / **3** gun — the same `pursuit_record+0x3c` value
  the HUD icon and `pm_showsymbol` use. `program_process_command` case `0xb2` calls
  `police_manager_get_committed_police_state` (VA `0x005ce510`), which finds the actor's
  `pursuit_record` and returns `record+0x3c`; the case casts it straight to float, so the script gets
  the **full tier (0/1/2/3), not a strict 0/1**. `xx` = actor id, `yy` = destination `flt[]` slot.
- `get_pm_crashtime xx, yy` — writes to `flt[yy]` the time since actor `xx` last crashed a car.
- `get_pm_firetime xx, yy` — writes to `flt[yy]` the time since actor `xx` last fired a weapon.
- `get_pm_state xx, yy, zz` — writes to `flt[yy]` **1.0** iff actor `xx` has the offence-type bit
  `1 << zz` set, else **0.0**. `xx` = actor index (0 = player), `yy` = destination `flt[]` slot,
  `zz` = offence type. `program_process_command` case `0xae` (opcode 174) ANDs the `1 << zz` mask
  with `police_manager_get_offense_bits_for_target` (= `bits_a | bits_b`). Reliable for
  `zz = 1..12` (§7). Full bit↔offence map and the `(1<<zz) & bits` model: `offences.md` §2.1/§4.
  Only shipped user: mission `MISE02A-TAXI`.
- `pm_showsymbol nn` — shows the police symbol on screen, simulating a police level without an actual
  offence. Works only when `policemanager_on` was **not** used on the current mission (used in one
  mission). `nn`: 0 none / 1 fine / 2 handcuffs / 3 gun / 4 searching / 5 transparent fine /
  6 transparent handcuffs / 7 transparent gun / 8 transparent searching.
- `pm_showprogress nn` — shows the wanted bar under the police icon. Never used in unmodified game
  scripts; works only when `policemanager_on` was not used on the current mission. `nn`: 0 = no
  visible bar, > 0 = bar visible.
- `pm_setprogress nn` — sets the wanted bar (used together with `pm_showsymbol` +
  `pm_showprogress`). `nn` = float `<0.0, 1.0>`, out-of-range sanitized. Never used in unmodified
  game scripts and buggy — appears to work only once after starting the game; works only when
  `policemanager_on` was not used on the current mission.
- `m7_zastavzkurvenepolise` — despawns the police cars chasing the player and respawns them at the
  same positions (presumably to stop them crashing into the player). Used only in Mission 7,
  script `detector_konec`, before one of the two ending animations.

### 8a. Runtime dispatcher

`program_process_command` is at **VA `0x005bb2d0`**, decompiled in full; every command's arm carries
its own case docblock. Its dispatch is a classic bytecode VM step:

```asm
PUSH this->ip                     ; current script-tree node
CALL <node_get_opcode_ptr>
MOV  EAX, [EAX]                   ; raw opcode (1..0x186)
DEC  EAX                          ; index = opcode - 1
CMP  EAX, 0x185
JA   <default_handler>
JMP  [0x005c7700 + EAX*4]         ; <-- the dispatch table
```

- **Dispatch table** lives at VA `0x005c7700` with **`0x186` (390) dword entries**
  (`0x005c7700..0x005c7d18`), indexed by `opcode - 1`, each the address of one command's arm or of
  the default arm, all inside `program_process_command`.
- **Opcode space**: `0x000..0x186`, **391 commands**. Opcode 0 is `LABEL`, which the compiler
  consumes, so it has no arm; out-of-range values fall through to the default.

So for `vlvp`: at load time `program_decode_instruction` resolves the name to opcode `0x122`; at
runtime the script-tree walker enters `program_process_command`, reads the opcode from the current
node, and `JMP [0x005c7700 + (0x122 - 1) * 4]` lands at the case body that queries police state and
stores 0/1 into the destination `flt` slot. `policeitchforplayer` is opcode `0x12c`, ten opcodes
after `vlvp`.

The full opcode-to-name mapping is `script-mapping/artifacts/opcode-table.md` — all 391 commands,
`0x000`..`0x186`, byte-extracted from the game's own keyword table. Per-command evidence is in
`script-mapping/artifacts/opcodes-verified.md`. The 15 commands that share the "just advance IP"
default arm are named in the generated table's header; `END`, `SETEVENT` and `VERSION_IS_GERMANY` are
**not** among them — each has its own arm (`0x6`, `0x49`, `0x15b`). Of the commonly assumed members
only `DETECTOR_WAITFORHIT` belongs there. The one-record read offset that can mis-pair a command name
with its predecessor's opcode is described in `basic/artifacts/script-opcode-dispatch.md` §4.

## 9. Code layout in `Game.exe`

The system is split across three layers. Address table: `Game-ghidra.exe.functions.md`.

| Layer | Owner | What it tracks |
|---|---|---|
| **Per-officer** | NPC entity (`officer + 0xf40..+0x12xx`) | the officer's individual state machine: 0 = idle, 2 = warn, 3 = chase-foot, 4 = arrest (handcuffs), 6 = vehicle-pursuit. `+0xf44` is the substate (faded toggle bit, vehicle action 1..7). `+0xafc` is visibility — 0.75 is the cutoff above which `entity_force_ai` refuses any new state |
| **Pursuit squad** (`police_action`, `0x6c` bytes) | `police_board` array `+0x14..+0x18` (`structs/police-action.md`) | the chase team for one suspect. Holds the officer list, the suspect (`+0x60`), the chase vehicle (`+0x5c`), the response tier (`+0x2c`) and the sighting timers (`+0x40`/`+0x44`) |
| **Follow group** (`ai_follow_group`, `0x38` bytes, machine 0..5) | `ai_follow_manager` group vector `+0x04..+0x08` (`structs/ai_follow_manager.md`) | per-target arrest/escort timing. State 0 → 2 → 5 (faded) → 1 (back to idle); state 4 = enforced-pursuit window; state 3 = approach |

**`police_board+0x24` is the arrest-settle timer**, not a wanted-bar field. `police_board_tick`
decrements this uint (ms) each frame; `police_board_arrest_settle_is_zero` reports zero; it is
clamped DOWN to 12 000 ms = 12 s **on the ARREST branch** of the arrest/fine cutscene only. It is the
post-bust settling grace timer: after an arrest it counts 12 s → 0, gating when the arrest end-screen
menu appears (also read by `program_process_command` case `0x33`). The on-screen wanted bar is a
different thing entirely — `g_police_wanted_bar`, from `pursuit_record+0x44`/`+0x48` (§2).

## 10. Per-frame call chains

Two independent per-frame engines run in parallel out of `game_tick`.

```
engine_game_loop                      (engine vtable, every frame)
  └─ +0xcc world update
       └─ mission_tick
            └─ game_tick
                 ├─ police_board_tick(dt_ms)      — police_action_tick × N + decrement the
                 │     │                            arrest-settle timer (police_board+0x24)
                 │     └─ police_action_tick      — per officer in group, dispatches 0xd/0xe events
                 │           ├─ ai_action_manager_has_action
                 │           └─ entity_spawn_police_response   (if vehicle action chosen)
                 │                 ├─ entity_dispatch_event(target, 0x14, 0)
                 │                 └─ entity_force_ai(officer, 6, sub, …)
                 │                       └─ human_pose_null_rotate
                 ├─ ai_groups_tick                — per-target line-of-sight prune (radius 150²)
                 ├─ ai_follow_manager_update      — drives the 0..5 pursuit-table state machine
                 ├─ ai_fight_manager_tick         — 2 s heartbeat over fight followships: member
                 │                                  state machine, AI car commandeering
                 │                                  (ai_group_get_closest_usable_car →
                 │                                  ai_fight_followship_take_car), pursuit-event
                 │                                  dispatch. Never reaches the police board
                 └─ police_manager_tick(this = C_game+0x2DA0, dt_ms)   — §10a
```

`police_action_broadcast_event` and `ai_follow_manager_event` are reached on demand, not from the
per-frame chain, when a state-changing event happens.

### 10a. `police_manager_tick` — the player-side engine

`police_manager_tick` is the per-frame state machine for the `police_manager`, distinct from
`police_board_tick` which drives the per-officer / per-squad AI.

```
police_manager_tick(this = C_game+0x2DA0, dt_ms)
     ├─ tick_acc_outer += dt; if (913 < acc) {
     │    acc -= 913;
     │    police_manager_npc_list_tick(this, 0)
     │    └─ walks npc_list at +0x80/+0x84 (the
     │       despawn list, §15), computes the 2D
     │       camera distance² per member and despawns
     │       the cars and NPCs that are out of range.
     │  }                    — ~1.1 Hz, ALWAYS RUNS
     │
     ├─ tracked-car cap     — keep the +0x90 police cars ≤ 5
     │
     ├─ end_screen_path     — if police_board_arrest_settle_is_zero()
     │                        && hud_fade_get_state() → spawn
     │                        end-screen overlay, gui_menu_execute_modal
     │
     ├─ (gated on enabled + !engine_paused)
     │
     ├─ if (g_act_mission.+0x2b14 < 10): dormant — clear
     │      certain record bits, no timer ticks
     │
     ├─ tick_acc_visibility += dt; if (217 < acc &&
     │    !police_board_is_busted_cutscene_active()) {
     │    acc -= 217;
     │    <visibility / LOS refresh>
     │    └─ walks the records vector (size via
     │       vec_count_stride_0x4 on records), does
     │       per-record world-position + view-direction
     │       + nearest-cell lookup via roads_get_near_node, sets
     │       sighting markers (record+0x24 += 4000 clamp
     │       10000, record+0x3e = 1, record+0x64 = 6),
     │       populates rndr_arr_b at +0xA0.
     │  }                    — ~4.6 Hz
     │
     ├─ tick_acc_inner += dt; if (330 < acc) {
     │    acc -= 330;
     │    <per-record state machine + commit>
     │    ├─ walk records vector at +0x40
     │    ├─ per record: read *(*record+0x4a0) for state
     │    │   gating; bit-flag updates at *(record+8);
     │    │   speed/threshold checks
     │    ├─ TICK DOWN 9 per-record timer fields by the
     │    │   330ms step value: record+0x14, +0x18,
     │    │   +0x1c, +0x20, +0x24, +0xc, +0x10, +0x40,
     │    │   +0x44 (the canonical step is the +0x08
     │    │   period itself — coupled design)
     │    └─ POLICE-STATE COMMIT: per record copy
     │       *(record+0x3c) = *(record+0x3e) (pending→
     │       committed u16), gated on visibility:
     │       records count == 0 → unconditional
     │       commit; > 0 → conditional on
     │       police_board_has_recent_sighting AND
     │       record.identity_pending. At level 2
     │       (handcuffs), police_board_promote_handcuffs_escalation
     │       runs first; at level 3 (gun),
     │       police_board_promote_gun_escalation runs first.
     │  }                    — ~3.0 Hz
     │
     └─ scene_cleanup_path  — when the player actor goes
           invalid: walk tracked_police_cars, finalize entries,
           free backing, call
           police_manager_npc_list_tick(this, 1) — the
           mode=1 "tick + wipe entire list" variant
           (the only mode-1 callsite in the binary).
```

**Tick rates are structural, not nested by code** — each accumulator advances independently. The
middle tier is gated by `!police_board_is_busted_cutscene_active()`, not by the outer tier's
overflow; the inner tier is unconditional within the per-frame body. So during an active pursuit
(with a squad spun up) the middle-tier visibility refresh is dormant and only the inner and
outer tiers tick.

**Magic literals.** The four operational period defaults (330 / 913 / 217 / 4013) are byte-verified
by `police_manager_game_init`. The 4013 ms period at `+0x28` and the (`+0x2c`, `+0x30`) padding pair
are **vestigial** — written by `police_manager_ctor` and `police_manager_game_init` but never read;
most likely removed-feature remnants. Full analysis: `structs/police_manager.md` §2.2.

**How the two engines interact.** `police_manager_tick` drives the `police_manager`: its
records vector (the pursued targets, including the player), its npc_list (the despawn list of
temporary cars and NPCs, §15), its tracked police cars (`+0x90`) and `rndr_arr_b` (`+0xA0`). `police_board_tick` drives `police_action` objects
and the per-officer state machines. They meet at the
`police_board_promote_handcuffs_escalation` / `police_board_promote_gun_escalation` calls made from
the commit step, and at the `police_board_has_recent_sighting` /
`police_board_is_busted_cutscene_active` guards.

## 11. Vehicle movement → collision → offence bits

The per-frame vehicle tick (`basic/artifacts/game-loop.md`, "Per-frame vehicle tick & how cars move")
is what *moves* cars each frame. When a moving car strikes something, a separate, event-driven
collision-response path turns that contact into police offence bits on the player's record, which the
`police_manager_tick` inner tier then commits into a police state. Movement is per frame; offence
registration is per contact; consumption is per frame again.

### 11a. The collision-response path

Each vehicle registers a collision-response handler table during construction via `car_init`
(`0x00462d30`) — entry #2 is `car_react_to_and_report_collision` (`0x0046c6e0`), the per-contact
dispatcher. On each impact it switches on the struck entity's type tag:

| struck entity | branch | offence registered |
|---|---|---|
| cop NPC on foot (kinds 2 / 0x13 / 0x19 / 0x1b) | struck → vtable #30/#31 (`pedestrian_crowd_collision`), and only when the cop's officer-attacked state byte `+0x15a` ∈ {1,3} | `police_manager_register_assault_on_police_officer_for_target` (`0x005d11c0`) → **bit 13** (`record+9 \|= 0x20`), action 9 **"Assault on Police!"**, pending GUN (3) |
| cop **car** (type 4 **and** the operational-vehicle flag `+0x20bd` set) | car-assault registrars; the COP identity is confirmed downstream by scanning `g_traffic_car_registry` (`traffic_car_on_collision_try_register_assault_on_police_car`, gate `+0x150`) | `police_manager_register_assault_on_police_car_for_target` (`0x005d1220`) → **bit 15** (`record+9 \|= 0x80`) + severe-impact flag `+0x6c` = (9.7 < severity) + alert payload `+0x60`, gated severity ≥ 2.8, action 10 **"Assault on Police car!"** |
| any entity (generic crash / contact / run-over) | always, after the type-specific branch | `police_manager_register_collision_offence_for_target` (`0x005cde10`) |

`+0x20bd` is an **operational / live-vehicle** flag (setter `car_set_is_traffic_car`), NOT a police
marker — it is set `= 1` even for ambient civilian traffic when an `s_traffic_element` is promoted to
a real simulated vehicle, and it also gates engine audio in `car_ai_tick`. So the car-assault branch
gates on "the struck car is a real operational vehicle"; the COP-car identity is decided by the
`g_traffic_car_registry` scan.

**Cop-car vs gang-car routing (the `C_traffic_car` collision adapters).** When the struck entity is a
real operational vehicle (type 4 + `+0x20bd`), the dispatcher routes through two sibling adapters
that both scan `g_traffic_car_registry` — the faction-car registry of cruising cop **and** gang cars
— for the entry whose car (`+0xc`) is the struck car, and split on the entry's faction flag. The
`+0x152` not-already-processed guard applies to both. Confirmed in game with a memory watcher:

- **cop car** (`+0x150` set) → `traffic_car_on_collision_try_register_assault_on_police_car` →
  `police_manager_register_assault_on_police_car_for_target` (the bit-15 car assault above). The
  registrar fires **only** for a police car.
- **gang car** (`+0x155` set) → `traffic_car_on_collision_try_register_assault_on_gang_car` (not
  police), and **only on a bigger hit** (`severity > 5.0`) → `traffic_car_spawn_mafia_pursuit_car`,
  which **activates the dummy gangster car that was hit, in place**: it becomes a full pursuer with a
  `mafian1` crew, and **no new car spawns** — the same realize-from-dummy mechanism as
  `traffic_car_take_over_cruising_car`. Hitting a pedestrian gangster never reaches this (car only).

So `+0x150` = **cop-car** flag and `+0x155` = **gang-car** flag; neither is an in-vehicle / on-foot
posture bit.

**Bit 13 persists, bit 15 is transient** (confirmed in game with a memory watcher). **Bit 13**
(assault on officer) stays set on the record until it decays — a watch on `record+0x09` shows it
(punching or ramming a cop on foot leaves `0x2200` = bit 9 violence + bit 13 assault). **Bit 15**
(assault on car) is an **alert-pending / needs-dispatch** signal:
`police_manager_try_dispatch_pursuit_for_target` consumes it on the next pass
(`record+9 &= 0x7f`, VA `0x005d0d12`) right after it spawns cops and prints "Assault on Police car!",
so a watch on `record+0x09` reads 0 even immediately after the hit — set and cleared within one
frame. The lasting effect of a car assault is the **committed police state** (e.g. a hard hit →
handcuffs via the bit-15 severe `+0x6c` branch), not a standing offence bit. Full set/consume
mechanism: `offences.md` §2.2 (bit 15) and §5.3.

A **fourth** vehicle-movement offence comes from the per-frame tick itself, not from a collision:
`car_ai_tick`'s sidewalk-check tail scans the wheel ground-contact list (`+0xd24`, count `+0x5b0`)
and, when a wheel sits on surface material id 3 (Chodník = sidewalk), calls
`police_manager_register_sidewalk_offence_for_target` (`0x005ce150`) → **bit 0x40**
(drive on pavement).

### 11b. Hand-off to `police_manager_tick`

All of these registrars write **into the player's `police_manager` record** — the records vector at
`police_manager+0x40`: they set offence/escalation bits in `record+9` and the pending state
`record+0x3e`. They do NOT themselves change the visible police state. The `police_manager_tick`
inner tier (~3 Hz, §10a) is the consumer: each tick it walks the records, reads the bit flags at
`record+8`/`+9`, and runs the police-state commit (`*(record+0x3c) = *(record+0x3e)`, gated on
visibility / recent sighting). The full loop:

```
per-frame vehicle tick                  moves the car
  └─ (on impact) collision dispatch     car_react_to_and_report_collision
       └─ offence registrar             sets record+9 bit + pending record+0x3e
            └─ police_manager_tick      commits pending → police state, escalates pursuit
```

This is why ramming a cop or cop car raises the police state a frame or two *after* the hit —
register on contact, commit on the next `police_manager_tick` inner-tier overflow — and why it is
gated by line of sight / recent sighting just like the on-foot offences. Per-bit offence table:
`police-system/artifacts/offences.md`; record-field layout: `structs/police_manager.md`.

## 12. The pursuit-asset pool — producer and consumer

The `police_manager` holds three pre-loaded **ready-reserve pools** of pursuit assets:
`police_vehicles` (vec2 @ `+0x50`), `spike_strips` (vec3 @ `+0x60`), `foot_cops`
(vec4 @ `+0x70`). These pools are populated up front by a script-driven producer and
drained on demand by a runtime consumer, so mid-pursuit asset deployment happens with zero i3d-load
and zero `operator_new` cost.

**Producer — `police_manager_apply_state`.** Invoked by script-VM case `0x86` `POLICEMANAGER_ON`,
typically once per mission segment. It walks three sequential allocation blocks, one per bucket,
loading the appropriate `*.i3d` assets and constructing entities into the three vectors via
`std_vector8_insert_at`:

| Bucket | Count | Assets | Element constructor |
|---|---|---|---|
| vec2 `police_vehicles` | up to 16 | i3d pool gated by `vehicle_tier` (`+0x39`, set from script arg `yy`): tier 0 picks `poli00` / `polimFor00`; tier non-zero picks `polimFor00` / `polimTud00` / `polCad00` | `car_ctor` on a `0x21f4`-byte `operator_new` block |
| vec3 `spike_strips` | 8 | single i3d (`barrier.4ds`) | `spike_strip_ctor` on a `0x10c`-byte block |
| vec4 `foot_cops` | 36 | i3d pool `pol11` / `pol12` / `pol13`, uniform random | `entity_init` on a `0x128c`-byte block |

Per-bucket spawn-pipeline mechanics: `structs/police_manager.md` §2.3.

**Consumer — the deploy trio.** One per bucket:

| Bucket | Deploy wrapper | Pool-pop helper (called inside) | Per-wrapper specialisation |
|---|---|---|---|
| vec2 | `police_manager_deploy_vehicle_from_pool` (VA `0x005ce770`) | `police_manager_vehicle_pool_pop_random` — uniform random + post-pop hooks | Cycles a 3-step vehicle-controller state machine on `entity+0x20c`; the `vt[+0x3f8]` hook fires after each transition |
| vec3 | `police_manager_deploy_temp_spike_strip` (VA `0x005ce8d0`) | `police_manager_spike_strip_pool_pop` — LIFO, first-claimed walking backward | Navmesh-snaps the spawn position with a downward `collision_test_line_vertical_thunk`; the caller's position vec3 is overwritten with the snapped point. The spike strip is `C_irenka`, entity kind `0x1f` |
| vec4 | `police_manager_deploy_npc_from_pool` (VA `0x005ce9e0`) | `police_manager_officer_pool_pop_random` — uniform random | Equips ONE random weapon, 50/50 between weapon id `0x0b` (pump-action shotgun, byte 4 = 8 = 8-shell magazine) and `0x0a` (Thompson 1928 SMG, byte 4 = `0x32` = 50-round drum). Mapping per `def-files-docs/predmety.md`. Calls `entity_reset_state` after equipping |

Each deploy wrapper follows the same skeleton: pop one entity from the bucket (clearing the entry's
`+4` claim flag), place it at `(pos, dir)` via the I3D frame's vtable slots (`+0x80`/`+0x84`/`+0x88`
translation write, `vt[+0xc]` orient, `vt[+0x18]` finalise), run the per-bucket specialisation,
register via `game_add_temporary_actor_intern`, and return the entity — or 0 if the pool was empty.

**Why it matters for gameplay.** This is a **pre-load ready-reserve cache**. The cost of i3d loading
plus entity construction is paid up front when the script issues `POLICEMANAGER_ON`. Subsequent
mid-pursuit spawn events — a chase patrol car materialising near the player — just pop a
pre-constructed entity and slot it in: no frame-time spike, no i3d cache miss, no constructor call.
Heavy script segments with many waves of reinforcement keep a flat per-frame cost.

**Lifecycle interaction with `police_manager_game_init`.** A soft scene reset clears vec2 and vec3
(sets `end = begin`, keeps the backing buffer) but does NOT touch vec4. So after a soft reset,
deploying a vehicle or a spike strip returns 0 until the script re-issues `POLICEMANAGER_ON`, while
deploying a foot cop remains immediately available. The asymmetry is intentional — foot cops carry
persistent AI state that scripts depend on across scene transitions.

**The composite "spawn a chase patrol car" workflow** — `police_manager_spawn_patrol_car_reinforcements_at_radius`,
one of the two callers of the vehicle deploy (the other, `police_manager_spawn_roadblock_in_neighbor_cells`,
passes flag 0):

1. Compute a spawn pose around the target: a 10-iteration retry loop with navmesh snap and a
   distance² threshold against the target's current position.
2. Call `police_manager_deploy_vehicle_from_pool(&pos, &dir, 1)` → the vehicle entity, the chase
   patrol-car host.
3. Call `police_manager_officer_pool_pop_random` twice to pull two cop NPCs from vec4 — the raw pop
   helper, not `police_manager_deploy_npc_from_pool`, because this workflow wants its own two-weapon
   loadout.
4. Equip each NPC via `inventory_insert_item`: pump shotgun (weapon id `0x0b`, mag 8) + S&W M&P
   revolver (weapon id `0x08`, capacity 6).
5. Register each NPC and bind it to the vehicle's occupant slots.

So the runtime event is one vec2 deploy + two vec4 pops + 4 weapon equips, all served from pre-loaded
pools with no i3d or heap activity at deploy time.

## 13. State-change paths

```
                     ┌──────────── player commits an offence ────────────┐
                     │                                                    │
                     ▼                                                    ▼
         (mission script / scene)                              (engine event ─ pursuit_event_dispatch_pursuit_vehicle
            console_addtext(2100..2110)                         "siren" sound on group spawn)
                     │
                     │  game code reacts
                     ▼
        ┌──────────────────────────────┐
        │ entity_force_ai /            │  (per-officer)
        │ entity_spawn_police_response │
        └──────────────┬───────────────┘
                       │ writes officer+0xf40 = state
                       │
        ┌──────────────┴──────────────────┐
        │                                  │
        ▼                                  ▼
 state 2 / 3  (foot)                state 6  (vehicle)
   "fine / handcuffs"                  pursuit
   on-foot animation                   substates 1..7 = ram / approach / fire / block
                       │
                       │ if visibility (officer+0xafc) drops below +0xf58
                       │    → entity_script_switch toggles
                       │      officer+0xf44 between 0 (visible) and 1 (faded)
                       ▼
            ┌──────────────────────────────────────┐
            │ police_action_tick (every frame)     │
            │   ├─ if alarm window expired:        │
            │   │     dispatch event 0x0D / 0x0E   │
            │   │     to every officer except      │
            │   │     sender via                   │
            │   │     police_action_broadcast_event│
            │   └─ if all officers lose target:    │
            │         entity_force_script          │
            └──────────────────────────────────────┘
                       │
                       │ arrest-settle grace (NOT wanted-bar drain)
                       ▼
            ┌──────────────────────────────────────┐
            │ police_board_tick decrements         │
            │   police_board+0x24                  │
            └──────────────────────────────────────┘
```

### 13a. Event lookup — `entity_dispatch_event`

The per-officer event dispatcher is the shared backbone. Event-type codes observed:

| Event | Effect |
|---|---|
| `0x00` | weapon-seen ping; arms officer or queues warning |
| `0x07`/`0x08` | sight pings (NPC seen) |
| `0x0B` | spawn an arrest-progress object (28 bytes) |
| `0x0C` | vehicle event; reads `police_board_find_group_by_officer`, may clear pursuit |
| `0x0D` | vehicle pursuit transition → calls `entity_spawn_police_response` (action 2 or 3) |
| `0x0E` | vehicle pursuit transition → calls `entity_spawn_police_response` (action 7 or 6) |
| `0x0F` | substate 5 (in state 6) |
| `0x10` | substate 4 (in state 6) |
| `0x12` | start the 20 s arrest timer |
| `0x14` | sent by `entity_spawn_police_response` *before* setting state 6 |
| `0x16`/`0x17`/… | misc broadcast variants |

`police_action_broadcast_event(group, evt_type, …)` rebroadcasts these to every officer of the group
except the sender — which is how a single sighting escalates the whole group.

### 13b. Sequence: handcuffs (state 4) → arrest

```
script: console_addtext(2100)       offence: "Police! Weapon seen!"
        └─ raises the player's police state (record+0x3c); police_board+0x24 is NOT bumped here
                ↓
entity_dispatch_event(officer, target, 0x0D, …)
        └─ police_board_find_group_by_officer → squad
        └─ entity_spawn_police_response(target, 2|3, …)
                └─ entity_dispatch_event(target, 0x14, 0)
                └─ entity_force_ai(officer, 6, sub=2|3, …)
                ↓ (player car stationary, officer drives in)
entity_dispatch_event(officer, target, 0x12, &cuff_data)
        └─ creates the 20 s arrest-deadline object
                ↓ (timeout reached without escape)
entity_force_ai(officer, 4, …)
        └─ human_free_hand()                the handcuff-prop spawner (§16h)
        └─ +0xf50 |= 1                      "real arrest"
        └─ hud_fade_set …                   (game-over screen scheduled by game_tick)
```

### 13c. Sequence: gun (state 6 + sub 4/7) → de-escalate

```
entity_dispatch_event(officer, target, 0x0E, …)
        └─ entity_spawn_police_response(target, 7|6, …)
                └─ entity_force_ai(officer, 6, sub=7|6, …)
                ↓ (player flees, no sight for ≥ +0xf58 seconds)
entity_script_switch(officer)
        └─ +0xf44 ← 1   (faded substate)
                ↓ (continued evasion)
entity_switch_ai_state(officer)           (called from the engine)
        └─ ai_logs_get_max_angry / ai_logs_get_max_fear ≈ 0
        └─ entity_force_script(officer, 1)
                ├─ ai_follow_manager_event(3, officer)
                ├─ ai_action_manager_clear_all(0)
                └─ human_do_breath()
                ↓
wanted bar (record+0x44) drains in police_manager_tick while unseen
   → on full drain: police_manager_record_clear_pursuit_state → idle
   (police_board+0x24 is the arrest-settle grace; it only gates the arrest end-screen)
```

### 13d. Sequence: searching ↔ police state swap on car/foot transition

The searching swap (§3) is decided by the chasing squads' identity fields (`police_action+0x64`
known vehicle, `+0x68` face known) and read back by `police_manager_tick`:

```
player seen committing an offence on foot
        └─ police_board_create_and_engage          squad +0x64 = 0, +0x68 = 1
                ↓ player enters a car, no officer sighting in the last 2000 ms
human_intern_use_car → police_board_update_sighting(player, 0, 0)
        └─ not witnessed → squad fields unchanged
police_manager_tick (inner tick)
        └─ police_board_target_matches: car != +0x64  → record+0x51 = 1 → icon "searching"
                ↓ player leaves the car
police_manager_tick
        └─ on foot and +0x68 set → record+0x51 = 0 → icon restored
```

The mirror case (seen only in a car, leaves it unseen) runs the same way with `+0x64` = that car and
`+0x68` = 0. A witnessed entry or exit, an officer struck by the player or his car (`entity_hit`),
and any squad created or refreshed while the player is on foot all set `+0x68`; the complete writer
table and the recompute branches are in `structs/police_manager.md` §3.2.

## 14. Escalation bridge and squad lifecycle

Mission-script offence triggers do not poke the group state directly. They go through a two-step
queue:

```
script signal (offence committed)
        │
        ▼
police_manager_record_raise_police_state(threat_obj, target, flag)   raises the COMMITTED level
        │                                             at threat+0x3c
        │                                             (flag 0 → 2 = handcuffs, flag 1 → 3 = gun)
        ▼
police_manager_tick (commit phase, every frame)
        │
        ├─ committed 2 (handcuffs):  police_board_promote_handcuffs_escalation(manager, target)
        │                            → group state 4/5
        ├─ committed 3 (gun):        police_board_promote_gun_escalation(manager, target)
        │                            → group state 0/1
        │
        └─ fine path:  threat+0x3c ← threat+0x3e   (police_manager_tick latches the pending level)
```

`police_manager_record_raise_police_state` raises the committed level at `threat+0x3c` directly and
monotonically, as `level = (flag != 0) + 2`.

The two families differ in job:

| Family | Members | What it does |
|---|---|---|
| raisers | `police_board_raise_handcuffs_escalation` (writes `+0x3c = 2`) · `police_board_raise_gun_escalation` (writes `+0x3c = 3`) | used by gameplay event paths; they additionally dispatch the chase vehicle (dispatch constants `0x400,0x3c` for **gun**, `0x200,0x1e` for **handcuffs**) and refresh the last-seen vector at `+0x10..+0x1c` on the matching squad |
| promoters | `police_board_promote_handcuffs_escalation` (sets the handcuffs group pair 4,5) · `police_board_promote_gun_escalation` (sets the gun group pair 0,1) | mirror the escalation into the group FSM: state pair + sighting refresh |
| pending setters | `police_manager_register_violence_offence_for_target` · `police_manager_register_gunfire_offence_for_target` | set the pending tier that the commit step later latches |

### 14a. Lifecycle

```
spawn (mission script / radio event)
   ↓
police_board_raise_gun_escalation (gun → state 0/1) /
police_board_raise_handcuffs_escalation (handcuffs → state 4/5)   — dispatches the chase vehicle
   ↓
police_action_tick (every frame from police_board_tick)
   ↓ on each new sighting:
police_board_update_sighting(manager, target, vehicle, force)
   ↓ on lost target:
police_action_disband(group, sender, exclude)
   ↓ on cleanup:
police_board_end_pursuit(manager, group_ptr)
```

The wanted bar (`pursuit_record+0x44`, drained by `police_manager_tick` while the player is unseen)
is independent of the group lifecycle: even after all groups are disbanded, the bar still has to
drain before the pursuit record fully clears
(`police_manager_record_clear_pursuit_state`). `police_board+0x24` is the separate arrest-settle
grace (§9).

## 15. `police_manager.npc_list` at `+0x80`/`+0x84`/`+0x88`

| Offset | Type | Role |
|---|---|---|
| `police_manager + 0x80` | `actor**` | `npc_list_begin` — `std::vector<actor*>::_begin` |
| `police_manager + 0x84` | `actor**` | `npc_list_end` — `std::vector<actor*>::_end` |
| `police_manager + 0x88` | `actor**` | `npc_list_cap` — `std::vector<actor*>::_cap`, read by the writer `police_manager_npc_list_push` (the eraser does not read it) |

**The list is the despawn list.** It holds temporary cars (kind 4) and `C_entity` NPCs (kind
`0x1b`) that the engine spawned or released; `police_manager_npc_list_tick` deletes each one once it
is more than 150 m from the active camera. Membership and the removal rules are measured in
`structs/police_manager.md` §2.3. The functions that touch it:

| Function | Role | Receiver verification |
|---|---|---|
| `police_manager_npc_list_push(this, npc)` | Unique-append inserter for the npc_list at `+0x80`: uniqueness scan, then append or realloc. Generic vector code, but every located caller passes the `police_manager` | Multiple callsites in police code; in `program_process_command` case `0x171` the receiver is byte-recovered as the `police_manager` (`C_game+0x2DA0`), via the same `act_mission_get_game` → `game_get_police_manager` chain as case `0x16b` |
| `police_manager_npc_list_erase(this, npc)` | Erase-by-value swap-pop counterpart to the push, polymorphic on receiver layout | Receiver byte-recovered as the `police_manager` in case `0x16b` |
| `police_manager_npc_list_tick(this, mode)` | The consumer: mode 0 despawns the out-of-range members (outer tick), mode 1 empties the list | Called from `police_manager_tick` with its own `this` |
| `police_manager_game_init(this)` | Partial in-place reset: clears `enabled`, resets `speed_threshold_fine` to `17.0f` and `speed_threshold_arrest` to `28.0f`, drains the records vector at `+0x40` by setting `end = begin` (buffer kept), and clears vec2 / vec3 / npc_list the same way, preserving the buffer pointers. The `end = begin` write is the entire drain — the empty `ret 8` stub `stub_noop_args8_00575c00` that sits next to it does nothing. vec4 at `+0x70`, `tracked_police_cars` (`+0x90`) and `rndr_arr_b` (`+0xA0`) are NOT touched | Receiver verified as `police_manager` via disasm at VA `0x005dc670`: `lea ecx, [esi + 0x2da0]; call police_manager_game_init` |

**Script-side callers.** `program_process_command` case `0x171` (`ACTOR_ADDDORICI`) is ADD —
`police_manager_npc_list_push(slot_npc)` with `this` = the `police_manager`; case `0x16b`
(`HUMAN_RETURNTOTRAFF`) is REMOVE — `police_manager_npc_list_erase(slot_npc)` with the same receiver.
Both names come from `script-mapping/artifacts/opcode-table.md`. The two form a paired
add/remove command. Both validate the slot object with `folded_get_dword_0x10 == 0x1b`, the
`C_actor::GetKind` role of that folded symbol.

**Engine-side adders.** An `E8` census finds 15 `police_manager_npc_list_push` callsites and 5
`police_manager_npc_list_erase` callsites, and no tail jump to either. The adders are the pursuit
dispatch and reinforcement spawners, the cop promotion from a pedestrian, the gang spawn, traffic
code, exploded cars, thrown-out occupants, and the officer releases in `police_manager_tick` and
`police_manager_unregister_actor` (the loop that pushes every officer of a record); the full list is
in `structs/police_manager.md` §2.3. `police_manager_apply_state` makes no call to either.

**Receiver discipline.** At all three identified call sites — cases `0x16b` and `0x171` in
`program_process_command`, and `police_manager_game_init` from a `game_init` context — Ghidra's C
decomp hides the `__thiscall` receiver. Recovering the true call shape requires reading the disasm.

## 16. Engine-context findings — per-officer state, scoring, and timing

Grouped by struct receiver. All claims here are body-derived; cross-references point at function
names and struct docs.

### 16a. Per-officer status fields

Two per-officer fields are read by the police code as officer status:

| Field | Meaning |
|---|---|
| `officer + 0x5e` | the `C_actor` **death-processed latch** (`structs/actor.md`): set once by `actor_tick` when the death has been handled. `police_manager_count_officers_alive_for_target` counts assigned officers with `+0x5e == 0`, and `police_manager_tick` prunes the ones with it set |
| `officer + 0x11ec` | **"a weapon is drawn"** — the weapon-armed flag, a member of the `C_entity` weapon-AI band `+0x11d0..+0x11ec` (`structs/entity.md`). Read by `entity_weapon_is_armed`; written by `entity_weapon_get_info` (set iff the weapon slot `+0x1e8` ∈ 1..3) and by the `+0x11e8` weapon-state FSM. Not part of the LOS pipeline |

### 16b. `entity_can_safe_shoot` — an occluder walk, not raw LOS

The predicate is **not** a raw line-of-sight test. It walks the officer's per-target visibility-slot
table at `officer + 0xb14` (8 slots × `0x14` bytes = 5 dwords per slot) and returns 1 only when no
occluder in the cone at sufficient range blocks the view. So a hard wall between officer and target
reads as "cannot see" even when the line is geometrically clear; the slots track per-target occlusion
history.

### 16c. Cone-angle distance attenuation

The cone-acceptance threshold is distance-attenuated rather than fixed:

```c
if (distance > 3.0f)
    cos_threshold = cos(entity_weapon_get_spread() * 0.0002546479 + 0.17);
else
    cos_threshold = 0;   // 90 degree cone (effectively omnidirectional) at close range
```

`entity_weapon_get_spread` is the `+0x11dc` float getter on the officer. The magic constants are
byte-verified; the 90° fallback under 3.0 units is what makes close-range cops "feel everything"
regardless of facing.

### 16d. `entity_force_script` flag-flip semantics

When called with `force_if_idle == 0` AND `officer + 0xf08 & 0x1000`, the function transitions
`0x1000 → 0x2000` (soft reset). The hard-reset path skips the flag flip entirely.

`0x1000` and `0x2000` form a **paired pending-action sentinel**: `0x1000` is "soft reset requested but
not yet acknowledged", `0x2000` is "soft reset acknowledged, the action consumer should process it on
the next tick". The bits are mutually exclusive within this pair.

### 16e. The `ai_fight_modes` sub-object — `C_entity + 0xf64`

Every NPC officer carries the combat-mode scorer `ai_fight_modes`, embedded at `C_entity + 0xf64`
and reached through `entity_get_ai_fight_modes`. The full field map, the per-mode constants and the
decide/act split are in `structs/ai_fight_modes.md`; offsets below are relative to that object.

- Six mode slots of `0x34` bytes at `+0x00..+0x137`. Per slot: `+0x00..+0x0c` input weights,
  `+0x10` hit weight, `+0x14` the mode's total, `+0x18` accumulated hit statistic, `+0x1c`
  viability, `+0x20` last-used time, `+0x24` preference score, `+0x28` the time of that write,
  `+0x2c` the finish-request byte.
- `+0x138` active mode, `+0x13c` previous mode, `+0x140` round-robin refresh cursor — all three
  reset to **5** ("no mode") by `ai_fight_modes_reset`; `+0x144` the refresh deadline; `+0x148` the
  current enemy.
- `ai_fight_modes_set_mode_score(slot_idx, value)` writes the value and the mission clock to
  `+0x24` / `+0x28` of that slot and zeroes `+0x144`, forcing a full rescore. Heavy use — about 28
  callsites inside `entity_action_tick` alone.
- `ai_fight_modes_set_enemy` seeds mode 2 (cover-seeking) to `5.0` and mode 1 to `1.5` on every
  enemy change.

### 16f. Pursuit-event timing block — `officer + 0x11f0..+0x1204`

A 4-dword timing block plus a 2-dword next-event-timestamp pair drives audio/event dispatch during
pursuit:

| Field | Type | Role |
|---|---|---|
| `+0x11f0` | uint | base period for the next primary event (ms) |
| `+0x11f4` | uint | jitter for the primary |
| `+0x11f8` | uint | base period for the next secondary event |
| `+0x11fc` | uint | jitter for the secondary |
| `+0x1200` | uint | next primary event timestamp, seeded as `now + base + rand(0..jitter)` |
| `+0x1204` | uint | next secondary event timestamp, same formula |

The base/jitter pairs are seeded from one of **5 presets**, keyed by the officer's pursuit-level enum
at `officer + 0x1e8`. The per-tick consumer that reads the timestamps and dispatches when they are
reached is `entity_do_shoot_timed`.

### 16g. The keyed param table — `(officer / cop NPC) + 0x10b8..+0x10c0`

A `std::vector`-shaped header at `+0x10b8` (begin) / `+0x10bc` (end) / `+0x10c0` (cap), stride
`0x0c`: rows are `{int key, int p1, int p2}`, a small **id-keyed map** (keys 2/3/4 observed). It is a
generic keyed param store, not a patrol-mode structure. The receiver is the officer / NPC, not
`police_board`. The API:

| Function | Role |
|---|---|
| `entity_lock_substate` | writer — erases any row whose first dword equals `key`, then inserts a fresh `(key, p1, p2)` row; also stores `aux` → `+0x10c8` and, when its flag argument is set, `key` → `+0x10c4` |
| `entity_has_locked_substate` | reader |
| `entity_unlock_substate` | remover — removes the **FIRST** match only; forward scan, break on match |

The sole runtime writer is `ai_fight_manager_tick`, which publishes its current per-NPC
pursuit-approach sub-state into key 3: every call is `(3, 8, (float)substate, 0, 0)` with substate
`1.0..11.0`. Other keys are populated from save-restored rows. The table is **persistent** —
serialized per officer by `entity_save_state` and restored by `entity_load_state`, which also
corroborates the per-officer receiver: those functions serialize `+0xf40`/`+0xf44` alongside
`+0x10b8`.

For contrast, `ai_ghost_invalidate_actor` removes **ALL** matches with a backward scan. It is not a
police function — it purges an entity's remembered-obstacle records — and is mentioned only for the
contrasting shape. The asymmetry is intentional: "remove a single keyed row" is single-shot, while
"evict every record for a destroyed actor" must purge everything.

### 16h. The `ai_action` task struct

The `ai_action_manager`'s 3 task lists hold pointers to heap-allocated task objects (`0x24` bytes)
with a polymorphic per-type inner buffer:

| Field | Type | Purpose |
|---|---|---|
| `task + 0x00` | uint | type id (range `0..0x2A`) |
| `task + 0x04` | int | **post-increment UID snapshot** stored at construction — not a pointer back to the creator |
| `task + 0x08`, `+0x0c` | dword | caller-supplied tags, mirrored from the ctor arguments |
| `task + 0x10` | uint | creation timestamp from `g_act_mission+0x24+0x2b0c`, or a 0 sentinel |
| `task + 0x14` | uint | `sizeof` the inner payload buffer |
| `task + 0x18` | uint | flag word, OR'd by `ai_action_manager_signal_children` |
| `task + 0x20` | `void*` | pointer to the per-type inner buffer, from `operator_new(<size at +0x14>)` |

The full type inventory is the **35 `ai_action` factories** (`ai_action_new_*`) — the generic NPC AI
action vocabulary consumed by `entity_action_tick`.
**Only 2 of the 35 are police-specific**: type `0x21` `ai_action_new_police_action` (the fine/arrest
action) and type `0x28` `ai_action_new_police_call` (radio for backup). Full inventory:
`structs/entity.md` §6b. The inner-buffer size per type is captured by each factory's `operator_new`
argument.

`ai_action_generate_UID` post-increments the manager's UID counter `ai_action_manager+0x30`
(`structs/ai_action_manager.md` §1) and returns it; the factory stores it in the new task's `+0x04`.
The counter is never decremented: `ai_action_manager_clear_all` zeroes it on a hard reset. `ai_action_manager_has_idle_linked_to`
matches that stored snapshot against `task[2]` / `task[3]`, the caller-supplied tags — so linkage
back to the creator is carried by the **UID value**, not by a pointer.

### 16i. The inner subscriber-list block

A sub-struct embedded inside some task types' inner buffer (the buffer at `task+0x20`), used to track
subscribers to a per-task event, e.g. officers attached to a list_c task:

| Inner offset | Type | Role |
|---|---|---|
| `inner + 0x14` / `+0x18` | `std::vector` header | primary array, stride **0x18**, per-entry `+0x10` = valid flag |
| `inner + 0x24` / `+0x28` | `std::vector` header | secondary array, stride **0x10**, per-entry `+0x0c` = target key |
| `inner + 0x5c` | dword | empty sentinel |
| `inner + 0x68` | dword | direct slot |
| `inner + 0x8c` | dword | state marker |

The primary/secondary pair is the same general layout as the cross-task subscriber-list cluster;
wrappers for the inner block carry the `subscriber_list_*` prefix.

### 16j. The player pointer — `C_game + 0xe4`

`game_get_player_actor` (`0x00425530`) reads `C_game + 0xe4`, the single pointer to the player's
`C_actor` in the world object (`structs/game.md` §2). It is not a per-actor slot. It is distinct from
`officer + 0x127c` / `+0x1280`, which hold the per-entity `ai_logs` maxima: the greatest fear
(`ai_logs_get_max_fear`, max of `+0x50`) and anger (`ai_logs_get_max_angry`, max of `+0x4c`) among
non-suppressed log entries, whose receiver is the per-entity `ai_logs` sub-object rather than a
police manager. The suspect a pursuit squad chases is `police_action + 0x60` (§16l).

### 16k. `police_board_invalidate_actor` state transitions

When an NPC tracked by a squad is destroyed, the squad's tier advances:

- **Vehicle-lost path**: applies `(0, 2, 4) → (1, 3, 5)` — the same tier pairing as
  `police_board_raise_gun_escalation` / `police_board_promote_handcuffs_escalation`, where the pairs
  are `(0,1)` = gun, `(2,3)` = fine and `(4,5)` = handcuffs.
- **Tier 7** (disbanded) gets no step on vehicle loss.

The even/odd pairing throughout the squad tier encodes the squad's posture: **even = in a car, odd =
on foot** (`structs/police-action.md` §3). Tier 7 is not always terminal: a squad disbanded while
still holding a car is revived at its old tier, on foot, by the next redispatch
(`structs/police-action.md` §3a).

### 16l. Squad (`police_action`) struct fields (`0x6c` bytes)

| Field | Type | Role |
|---|---|---|
| `+0x2c` | uint | response tier (`structs/police-action.md` §3) |
| `+0x30` | uint | pursuit-session ID, stamped at construction by `police_board_increment_counter`; propagates to every group officer's `+0xf54` via `entity_spawn_police_response` |
| `+0x40` / `+0x44` | uint | last-perceived timestamp / sighting accumulator (`police_action_tick_sighting_timers`) |
| `+0x5c` | ptr | group vehicle |
| `+0x60` | `npc*` | target NPC pointer, written from the second parameter of `police_board_create_and_engage` |

Lookup helpers: `police_board_find_group_by_session_id` looks up by `+0x30`;
`police_board_has_group_for_target` looks up by `+0x60`.

### 16m. Bone-string anchors — handcuff geometry

From the body of `human_free_hand`, the handcuff-prop spawner used during an arrest:

- `s_l_hand_0064bcf0` and `s_r_hand_0064bcf8` are 8-byte adjacent string literals (`"l hand"` and
  `"r hand"`, each null-terminated to 8 bytes including terminator and padding).
- They are used as **attach-point bone names** when binding handcuff geometry to the player skeleton.
- `human_free_hand` constructs a frame of type 9 for each handcuff loop; type 9 is **MODEL** in the
  frame-type names of `ls3df.dll` (`structs/i3d_frame.md` §3a).

## 17. PC vs PS2

- `ai_fight_manager_tick` is the PC twin of PS2 `Tick__16ai_fight_managerFv`. Its 2 s heartbeat
  walks only the fight followships; it never reaches the police board.
- `entity_weapon_is_armed` is the PC twin of PS2 `WeaponIsArmed__8C_entityFv` (CONFIRMED).
- The 35 `ai_action_new_*` factories are the PC twins of the PS2 `ai_action_manager::New*` family.
- The three `C_traffic_car` collision adapters have the PS2 twins `CoDelasTyCuraku`,
  `CoDelasTyCuraku1` and `Krles`.
- The spike strip is the PS2 class `C_irenka` (entity kind `0x1f`); the shipped asset `barrier.4ds`
  is specifically a spike strip, not a whole roadblock. A roadblock in gameplay is a composite of
  cars, spike strips and foot cops (§2a).

## 18. Open items

Nothing is open.
