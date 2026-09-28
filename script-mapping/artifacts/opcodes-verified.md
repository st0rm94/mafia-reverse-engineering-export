# Mafia script-VM opcodes — verified

**Created** 2026-09-07 22:05 +0200 · **Modified** 2026-09-17 17:52 +0200

Per-command evidence for the runtime dispatcher `program_process_command` (`0x005bb2d0`, jump
table at `0x005c7700`, opcode range `1..0x186`).

## 1. Scope and inclusion bar

[`opcode-table.md`](./opcode-table.md) is the authority for *which case implements which command*:
all 391 commands, byte-extracted from the game's own keyword table. **This file records what a
command DOES** — its arguments, helpers, and how that was established. The mechanism behind the
generated table is in `basic/artifacts/script-opcode-dispatch.md`.

A command is listed here only when both hold:

1. its case body in `Game-ghidra.exe.annotated.c` was read and its distinctive helpers / behaviour
   identified, AND
2. that behaviour matches the command's description in the **Mafia Script Data Reference**.

Rows that additionally rest on an in-game test say so in their evidence cell.

## 2. The opcode is the case label

The value stored in `program_decode_instruction`'s keyword table **is** the dispatcher case label.
The keyword record is `+0x00 char name[0x20]; +0x20 int opcode` (stride `0x24`). The jump table at
`0x005c7700` is indexed by `opcode - 1` only because opcode 0 is `LABEL`, which has no arm; Ghidra
prints the case by the opcode itself. No offset is applied anywhere.

Reading the record with the fields swapped (`+0x00 uint32 opcode; +0x04 char[] name`) puts the
window four bytes early and pairs every name with **its predecessor's opcode**: 388 of 388 rows of
such a read satisfy `raw == opcode_of(table_index − 1)`. Where the table is locally in opcode order
that looks like a constant `opcode − 1`; where it is not, it gives an unrelated value:

| command | table index | predecessor (index − 1) | real case |
|---|---:|---|---|
| `policeitchforplayer` | 284 | `VLVP` (`0x122`) | `0x12c` |
| `get_pm_humanstate` | 388 | `HUMAN_GETITEMINRHAND` (`0x41`) | `0xb2` |
| `CITYMUSIC_ON` | 306 | – | `0x139` |
| `CITYMUSIC_OFF` | 307 | `CITYMUSIC_ON` (`0x139`) | `0x138` |

The `CITYMUSIC_*` pair is stored out of order in the table (index 306 = `CITYMUSIC_ON`, 307 =
`CITYMUSIC_OFF`), so the swapped read inverts the pair instead of shifting it. The real values
`CITYMUSIC_OFF = 0x138` / `CITYMUSIC_ON = 0x139` match the in-game behaviour (§3.10).

## 3. Verified commands

Column conventions for every table below: **Opcode** = hex / decimal = dispatcher case label;
**Handler** = the case's distinctive callee; `-` = none.

### 3.1 Script core

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x07` / 7 | `dim_flt xx` | - | Reallocates the `flt[]` register file (`param_1[0x12]` / `[0x14]` block), sized by the int arg. | Body reallocs the block; Mafia Script Data Reference: *"reserves xx memory slots for float variables"*. |
| `0x0a` / 10 | `rnd xx, yy, zz` | `rand_range` | Stores a `rand_range` result into the destination `flt[]` slot. | `rand_range` + the `LAB_005c76d4` float-write idiom; Mafia Script Data Reference: *"random integer from an interval"*. |
| `0x0d` / 13 | `findframe` | `I3D_scene` vtable `+0x58` | Looks up a scene frame **by name** into the FRAME register file ¹. | Bounds against `param_1[0x15]`, writes `param_1[0x14] + slot*4`; keyword table places `FINDFRAME` at `0x0d`. |
| `0x11` / 17 | `findactor xx, yy` | `i3d_frame_get_name` | Looks up an actor **by name** into the ACTOR register file ². | Bounds against `param_1[0x17]` (ACTOR table length); Mafia Script Data Reference: *"finds an object by name"*. |

¹ The name resolves through the `I3D_scene` vtable at `+0x58` (`folded_get_dword_0x10()` is the
scene in this function, not an actor kind). An empty name substitutes the current frame.
² Walks a vector with `vec_index_addr_stride_0x4`, comparing `i3d_frame_get_name()` against the
argument. Developer easter egg: when the current frame is named `chcipnuti_chlapka` it calls
`MessageBeep`. `0x0d` and `0x11` are graded together because both are lookup-by-name bodies; the
register-file bound (`[0x15]` frame vs `[0x17]` actor) is what separates them.

### 3.2 HUD: compass, stopwatch, console, fade, damage bars

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x1d` / 29 | `setcompass xx` | `hud_compass_set_destination_frame` | Points the compass at frame `xx`, or hides it for `-1` ¹. | Single int arg, `-1` sentinel, frame-pointer dispatch; Mafia Script Data Reference: *"Sets the compass point to the position of frame xx; -1 turns the compass off."* |
| `0x31` / 49 | `timeron ww, xx, yy, zz` | `hud_timer_set_time`, `hud_timer_set_interval` | Shows the analog stopwatch with hands `ww`/`xx`/`yy` and `zz` seconds remaining ². | 4 int args + analog-clock arithmetic; Mafia Script Data Reference: *"Shows the stopwatch … ww = hour hand (1..12), xx = minute (0..59), yy = second (0..59), zz = remaining time."* |
| `0x32` / 50 | `timeroff` | `hud_hide_element(8)` | Hides the stopwatch; the only statement is the hide ³. | 0 args, pairs with `0x31`; Mafia Script Data Reference: *"Stops the stopwatch and hides its display, see timeron."* |
| `0xd8` / 216 | `console_addtext xx` | `hud_console_print` | Prints textdb string `xx` to the on-screen message console in white (`0xffffff`, hardcoded) ⁴. | Body → `textdb_get_string_by_id_or_null` + `hud_console_push_message`; the Mafia Script Data Reference documents `xx` as a `textdb_xx.def` id. |
| `0xe9` / 233 | `zatmyse xx, yy` | `hud_fade_set` (`0x0054c490`) | Fades the whole screen to black: `xx` direction (0 = clear, non-0 = to black), `yy` duration in ms ⁵. | Only `program_process_command` case calling `hud_fade_set`; shipped easter-egg detector scripts issue `ZatmySe 0, 500` (keyword string `.rodata 0x0064efec`); screen goes black in game. |
| `0xea` / 234 | `cardamagevisible xx` | `hud_show_element` / `hud_hide_element(0x10000)` | Shows (`1`) / hides (`0`) the car-damage bar ⁶. | Used by mission `MISE05-MESTO`, script `detector_bertone`, next to `getcardamage` / `car_switchshowenergy`; verified in game. |
| `0x146` / 326 | `pm_showsymbol nn` | `hud_set_police_symbol_and_alpha` | Shows a police symbol without an offence: `nn` 1..4 = fine / handcuffs / gun / searching at intensity 1.0, 5..8 the same at 0.4, anything else hides ⁸. | 8-branch switch; the 1.0 / 0.4 pair matches the bright / translucent icons seen in game. Used in one mission. |
| `0x147` / 327 | `pm_showprogress nn` | `hud_show_element` / `hud_hide_element(0x10)` | Shows (`nn != 0`) / hides the wanted bar under the police icon ⁸. | Gate-then-toggle body; bit `0x10` of `+0x90a4`. Not used by shipped scripts. |
| `0x148` / 328 | `pm_setprogress xx` | `hud_set_police_wanted_bar` (`0x005c8010`) | Writes `flt[xx]`, clamped to `[0.0, 1.0]` (negative → 0), into the wanted-bar fill at `+0x96e0` ⁸ ⁹. | Gate-then-clamp body; slot bound `param_1[0x13]`. Not used by shipped scripts. |
| `0x14f` / 335 | `airplaneshowdamage xx, yy` | `airplane_hud_damage_toggle` | Shows / hides the airplane-damage gauge (HUD bit `0x40000`) ⁷. | Tier-1 string `s_AIRPLANESHOWDAMAGE_0064e950`; verified in game (§3.7). Mafia Script Data Reference: *"Shows the damage amount of airplane xx"*, `yy = 0 or 1`. |

¹ Bounds-checks `xx` against `param_1[0x15]`. Out of range or `-1` → `hud_hide_element(0x80)` +
`player_set_navi_point()`. In range → `hud_compass_set_destination_frame(frame)` (writes HUD
singleton `+0x9270`) + `hud_show_element(0x80)` + `player_set_navi_point()`. Bit `0x80` of the flag
word at singleton `+0x90a4` is compass visibility.
² `hud_timer_set_time(ww, xx, yy)` writes `+0x9150/+0x9154/+0x9158`; `hud_timer_set_interval(zz)`
writes the ms count at `+0x9168` and recomputes the hand fractions at `+0x915c/+0x9160` with `/60`
arithmetic; then `hud_show_element(8)` and `pedestrian_manager_end_all_taxi_waves()`.
³ No explicit stop call: whether the countdown stops or only its display is hidden is not visible in
the case body.
⁴ The same HUD message ring the police system uses for offence texts (ids 2100–2110, 2200, …).
⁵ Sets fade-overlay bit `0x8000` of `+0x90a4` and the `+0x9468..+0x9474` state block. The same
overlay drives the arrest / mission-fail end screen and the `game_cheat_check` acknowledgement flash.
⁶ HUD singleton at `0x00658330`. The show path fires only when
`actor_get_vehicle(game_get_player_actor(player))` is a type-4 (vehicle) entity (`target+0x98`).
⁷ `xx` = airplane actor id (`-1 < id < param_1[0x17]`, `param_1[0x16]` entry non-null,
`folded_get_dword_0x10() == 0x16`); `yy` = show flag. On show, writes the damage ratio to
`hud_state+0x9454` via `hud_set_airplane_damage_fill`. Used only in mission `MISE16-LETISTE`
(scripts `letadlo start`: `airplaneshowdamage 0, 1`; `load animation 16.2`: `airplaneshowdamage 0, 0`),
where the player shoots down a plane from a car — which is why bit `0x40000` is otherwise never a
visible HUD element.
⁸ All three run only when `police_manager_is_enabled()` is **false**, i.e. on a map where
`policemanager_on` was not used; the actor and police-block fetches at the top only feed that gate.
Receiver is the HUD singleton at `0x00658330`. `pm_showsymbol` sets bit `0x04000000` of `+0x90a4`,
writes the selector to `+0x96d8` and `alpha = intensity * 255` into the high byte of the RGBA word at
`+0x96dc` (RGB white); `nn = 0` or unrecognised clears the bit via `LAB_005c591b`, which
`pm_showprogress` reuses for its hide.
⁹ A visible bar also needs `pm_showsymbol` and `pm_showprogress`. In game the command works only once
after starting the game; first-call fill: `-1.0` empty, `-0.01` / `0.0` / `0.5` / `0.99` full,
`1.0` / `1.5` empty.

### 3.3 Wingman indicators

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x79` / 121 | `wingman_setindicator xx, yy, zz` | `player_set_wingman_indicator` | Pins one of 4 wingman-indicator HUD slots to a (portrait, live-value) display ¹. | 4-slot switch on bits `0x10000000..0x80000000`; confirmed in game ³. |
| `0x7a` / 122 | `wingman_delindicator xx, yy` | `player_set_wingman_indicator(0xffffffff, 0)` | Clears the pinned indicator and hides slot `yy`'s HUD bit ². | Sibling of `0x79`, same actor-table gate; same in-game anchor ³. |

¹ Args: `xx` = npc id, validated against `param_1[0x17]` with `folded_get_dword_0x10() ∈ {2, 0x1b}`
— **`0` = the player** (in-game; the Mafia Script Data Reference's "id 0 = Paulie" does not apply to this command);
`yy` = portrait `0..7` (masked `& 7`; scripts use 1..5, e.g. mission `MISE07-SARA`, script `Sara`:
`wingman_setindicator 1, 5, 0`); `zz` = slot `0..3` (`-1 < slot < 4`). Pipeline:
`player_set_wingman_indicator(slot, portrait)` records the pin on the player (`+0x6d8`/`+0x6dc`,
saved and loaded) → `hud_wingman_set_portrait_for_slot(slot, portrait)` writes `+0x9244 + slot*4`
→ value = the script scalar (`__ftol`), replaced by `*stl_max_int_ptr()` (an engine-live max) when
`actor_is_death_processed()` (the `player+0x5e` byte getter) returns 0 →
`hud_wingman_set_indicator_value_for_slot(slot, value)` writes `+0x9254 + slot*4` →
`hud_show_element(slot bit)` on `DAT_006613d4`. A per-frame player tick (3 sites) replays the pin.
`xx` is consumed only by the bounds check and is **not** stored.
² `player_set_wingman_indicator(0xffffffff, 0)` writes `player+0x6d8 = -1` ("no pin") and
`+0x6dc = 0`, which stops the per-frame replay **for all slots**; only slot `yy`'s bit is cleared
(slots outside `0..3` clear nothing), so other slots' icons go stale.
³ A diagnostic patch of case `0x146` sub-case 2 to `hud_show_element(0x10000000)` made a player health
indicator appear in game; repeated calls stacked to 4 independent indicators. The 4 hide bits in
`0x7a` and an init loop zeroing all 4 slots agree.

### 3.4 Saving

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x80` / 128 | `autosavegame xx, yy` | `loadsave_savegame_from_script(mgr, 0, yy != 0)` | Autosaves to slot `xx`; `yy != 0` also prints textdb 2203 *"The game was saved"* ¹. | Tier-1 literal `hud_console_print(0x89b, 0xffffff)` in the back-end; verified by the user. |
| `0x81` / 129 | `autosavegamefull xx, yy` | `loadsave_savegame_from_script(mgr, 1, yy != 0)` | Same, with `full_flag = 1` ². | Same anchor; `full_flag` is the only difference. |

¹ Arg0 = slot id (stored at `manager+0x1d0`, translated by `slot - 5000`); honours the
`manager+0x1d4` skip-once guard. Pipeline: `loadsave_savegame_create(manager, slot - 5000)` →
`loadsave_savegame_save(manager, full_flag)` → rewind (`*(manager+0x18) = *(manager+0x14)`) →
`loadsave_profile_save(manager)` → optional toast. `loadsave_savegame_save` uses only
`s_savegame_mafia_03d__03d_0064d788` (`savegame/mafia/%03d/%03d`) and the sigil `0x53617647` "SavG";
`s_savegame_mafia_03d_sav_0064d764` (`savegame/mafia/%03d.sav`) and sigil `'Prof'` (`0x50726f66`)
belong to `loadsave_profile_save`. `full_flag` is also passed to `game_save_game_get_size`
(`0x005f0450`).
² `full_flag` selects the scene-object source inside `loadsave_savegame_save`: `0` = the live
world-frame list at `*(g_act_mission+0x24) + 0x144/+0x148`; `1` = the static list at
`&g_actor_global_list_alloc` (via `LAB_0055922e`), a wider object set. The writer runs two passes
for both variants — pass 1 keeps `obj_type ∈ {4, 8}` (type 4 = the actor's vehicle, resolved
against the `manager+0xa0` stride-8 pair table), pass 2 everything else — both gated by
`obj+0x65 == 0`.

### 3.5 Police manager

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x86` / 134 | `policemanager_on [xx, yy]` | `police_manager_apply_state` (enable = 1) | Enables the police system and registers the player ¹. | Body reads both args without an arg-count check; `+0x34` has no reader in the police manager range; the `yy` pool effect is confirmed in game. |
| `0x87` / 135 | `policemanager_add "name"` | `police_manager_register_actor` | Registers an actor, by **name**, for offence detection like the player ². | Name loop: `vec_index_addr_stride_0x4` + `i3d_frame_get_name` + `__strcmpi`. |
| `0x88` / 136 | `policemanager_del "name"` | `police_manager_unregister_actor` | Unregisters an actor added by `policemanager_add` ³. | Same name loop; no second-arg branch. |
| `0x89` / 137 | `policemanager_setspeed xx, yy` | `police_manager_set_speed_thresholds` | Sets the fine (`xx`) and arrest (`yy`) speed thresholds, in km/h ⁴. | `* 0.2777778` (km/h → m/s) fingerprint + `game_get_police_block`. |
| `0x8a` / 138 | `policemanager_forcearrest xx, yy, zz` | `police_manager_escalate_handcuffs_with_optional_wanted` | Not an arrest: forces a handcuffs escalation on actor `xx`, optionally with the wanted substate ⁵. | In game: `0,1,0` → handcuffs, no wanted substate; `0,1,1` → handcuffs with the wanted substate. |
| `0x12c` / 300 | `policeitchforplayer xx` | `police_manager_count_officers_alive_for_target` | Writes 1 when the cops know who the player is, 0 while the player is still anonymous ⁶. | Mission scripts loop `policeitchforplayer 0; if flt[0]=1, -1, 2` to hold a dialog while the cops know the player. |
| `0x15e` / 350 | `police_support ww, xx [, yy]` | `police_manager_spawn_patrol_car_reinforcements_at_radius` | Spawns an armed NPC group around actor `xx` and registers it as a pursuit group on it ⁷. | Own arm `0x005c64a6`. The Mafia Script Data Reference lists the name with the gloss *"police support ???"*; no shipped script calls it. |
| `0x173` / 371 | `police_speed_factor xx, yy` | writes `g_police_speed_factor` (`0x006552d0`) | Sets the global police-car chase speed coefficient, clamped to `[0.0, 1.0]` ⁸. | The `[0, 1]` clamp matches the documented float range `<0,1>`. |
| `0x183` / 387 | `m7_zastavzkurvenepolise` | `police_board_release_all_group_vehicles` | Mission-7 helper (Czech: "stop the fucking police"), no args: halts every pursuit group's chase vehicle ⁹. | Case `0x183` is the **only** caller; verified in game. Used in mission `MISE07B-CHULIGANI`, script `detector_konec` (2 calls) before the ending animation. |

¹ `xx` is stored at `police_manager+0x34` and never read (shipped value 980000). `yy` is booleanized
to `+0x39` and selects the vehicle pool: `0` = `poli00` / `polimFor00` / `polimTud00`, non-zero =
`polimFor00` / `polimTud00` / `polCad00`. That `rand_range()` switch sits inside the wanted-spawn
branch, so `yy` affects only reinforcements that spawn during an active wanted substate; ordinary
patrol traffic always uses `poli00`. The worker is idempotent (bails when already enabled).
² Optional second arg (type 4 = string) → `police_manager_link_target_to_vehicle`, which also sets
`*(second_arg+0x2035) = 1` and calls `actor_get_world_position`. The register walks the pursuit-group
array at `*(this+0x40)..*(this+0x44)`. In the shipped scripts the command is commented out wherever
it targets a non-player actor.
³ Walks `*(this+0x40)..*(this+0x44)` for the target's record, then the officer sub-array
`record+0x74..+0x78`, before dropping the target. Commented out for non-player actors in the shipped
scripts.
⁴ Writes `*(this+0xc)` = fine m/s, `*(this+0x10)` = arrest m/s. Gameplay defaults: ~18 m/s
(65 km/h) → fine, ~28 m/s (100 km/h) → arrest.
⁵ `xx` = actor index (validated against `param_1[0x16]`, `folded_get_dword_0x10()` = 2 or `0x1b`;
always 0 = player in shipped scripts); `yy` = enable; `zz` = also raise the wanted substate (read only
when enabling). The worker finds the record for `target_actor` in `*(this+0x40)..*(this+0x44)`, sets
bit 14 (`0x40` at `record+9`) on enable / clears it on disable, and on enable with `zz != 0` writes 1
to `record+0x50`. The state decays normally and is **not** re-applied by the command: mission
`MISE16-MESTO`, script `detekce playera v kradenym aute` loops the command with `goto` until the
player re-enters the car.
⁶ Branch on the number of officers currently seeing the player (`police_manager_count_officers_alive_for_target`).
**None seeing:** result = `police_board_is_busted_cutscene_active()` = manager `+0x2c`, set only by
the arrest/fine capture cutscene (`entity_action_tick` case `0x21`) after the pursuit group is
disbanded. **At least one seeing:** result = `police_manager_target_identity_pending() == 0`; the
helper returns 1 while identity is pending (record `+0x51 != 0` and manager `+0x38 == 0`; `+0x51` is
written as `(police_board_target_matches == '\0')`). So `flt[xx] = 1` iff (seen ∧ identity known) ∨
(unseen ∧ capture cutscene animating); a player under pursuit who is unseen and not being arrested
reads 0 (§6). The Mafia Script Data Reference cross-references `vlvp` and glosses *"gives 1 if cops know who you are,
even when the icon is transparent"*.
⁷ `ww` = spawn radius (float, passed through); `xx` = target actor (bounds against `param_1[0x17]`,
`folded_get_dword_0x10()` = 2 or `0x1b`); `yy` optional, mapped `(yy != 0) ? 0 : 4` into the
worker's mode parameter. Sequence: `game_get_police_block()` (actor `+0x2da0`) →
`police_manager_spawn_patrol_car_reinforcements_at_radius(target, mode, radius)`, which picks a
navmesh point at least `radius * 0.66` away, spawns an NPC group with weapon id `0x1e` and registers
it as a pursuit group via `police_board_create_and_engage`. Return value discarded. Listed on body
evidence; the Mafia Script Data Reference gives no behaviour to match (§6).
⁸ Pre-clears the global to 0.0. `xx == -1` → `yy` is the literal float; otherwise reads `flt[xx]`
(out-of-range index leaves 0.0). The value is the last argument of
`pursuit_event_dispatch_pursuit_vehicle` (8 callsites).
⁹ For every pursuit group owning a chase vehicle: riding officers → act state 2, the vehicle's
pursuit order is cancelled (`pursuit_event_remove_vehicle`), it is detached from its waypath and its
dynamics are stopped, so it stops ramming the player. It does not despawn cars. Works on any map
where `policemanager_on` was called. Name string `s_M7_ZASTAVZKURVENEPOLISE_0064e4dc`, referenced at
`0x005ba585` in `program_decode_instruction`.

### 3.6 Police queries

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0xae` / 174 | `get_pm_state xx, yy, zz` | `police_manager_get_offense_bits_for_target` | `flt[yy] = 1.0` iff actor `xx` recently committed offence type `zz`, else 0.0 ¹. | In game (mission `MISE02A-TAXI`, no gun): `get_pm_state 1,0,6` detects sidewalk = bit `1<<6 = 0x40`, impossible under a raw `6 & bits`. |
| `0xaf` / 175 | `get_pm_crashtime xx, yy` | `police_manager_get_crashtime_for_target` | `flt[yy]` = time since actor `xx` last crashed in a car (`record+0x1c`); 0 without a record. | Standard 2-arg per-actor query skeleton. |
| `0xb0` / 176 | `get_pm_firetime xx, yy` | `police_manager_get_firetime_for_target` | `flt[yy]` = time since actor `xx` last fired (`record+0x10`); 0 without a record. | Same skeleton as `0xaf`. |
| `0xb1` / 177 | `get_pm_numpredators xx, yy` | `police_manager_count_officers_for_target` | `flt[yy]` = number of cops chasing actor `xx`, with no line-of-sight or identification filter ². | Returns `(record+0x78 − record+0x74) >> 2`; the Mafia Script Data Reference: *"0 → no one is chasing, >0 → at least one predator is chasing"*. |
| `0xb2` / 178 | `get_pm_humanstate xx, yy` | `police_manager_get_committed_police_state` | Reports whether actor `xx` is in any police state ³. | Helper returns the committed police-state level at `record+0x3c`; case sits in the `0xae–0xb2` `get_pm_*` cluster. |
| `0x122` / 290 | `vlvp xx` | `police_board_has_recent_sighting` | 1 if any pursuit group on this target has an officer who saw the player within the last 2000 ms, else 0. | Mafia Script Data Reference: *"Returns whether the police see the player during an offence"*. |

¹ `xx` = actor (0 = player), `yy` = output slot, `zz` = offence type 1..13. The helper returns
`bits_a | bits_b` (`pursuit_record+0x08/+0x0a`); the body pre-clears `flt[yy]`, validates the actor
type (2 or `0x1b`) and tests `script_argv[2] & bits`. The operand reaching the AND is the pre-shifted
mask `1 << zz`. Type map: 1 shooting `0x02`, 2 speeding `0x04`, 3 vehicle contact `0x08`, 4 crash
`0x10`, 5 run-over pedestrian `0x20`, 6 sidewalk `0x40`, 9 assault on police `0x200`, 11 car theft
`0x800`, 12 traffic sign `0x1000`. Each bit decays (timers ~4–60 s): `police-system/artifacts/offences.md`
§3.1.
² The count is 0 iff no pursuit group exists for the target — both with no police state at all and
once the wanted bar has entered its decaying phase (the group is dropped as soon as decay starts).
`flt[yy]` is pre-cleared before the type check, so a failed check leaves 0.
³ Levels: 0 no record, 1 fine, 2 handcuffs, 3 gun; the Mafia Script Data Reference documents a 0/1 result (§6).

### 3.7 Airplane

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x35` / 53 | `airplane_start xx, yy, zz` | `airplane_start_ground_run` (`0x004520e0`) | Starts airplane `xx` moving along route `yy` at `zz` km/h — **on the ground** ¹. | `* 0.2777778` km/h → m/s fingerprint; Mafia Script Data Reference: *"Start of airplane xx, which moves along track yy"*, `zz` = speed. In game (`MISE16-LETISTE`) the plane starts rolling. |
| `0x36` / 54 | `airplane_start2 xx` | `airplane_begin_takeoff` (`0x004522c0`) | Begins the takeoff of a plane already configured by `airplane_start`: motion mode `+0xb4 = 2`, moving anim phase. | 1 actor-ref arg; Mafia Script Data Reference: *"Beginning of the takeoff of airplane xx"*. In game the plane takes off and climbs. |
| `0x90` / 144 | `airplane_isdestroyed xx, yy` | `airplane_is_destroyed` (`0x00453190`) | `flt[yy]` = 1.0 when **both engines** (`+0x1c8`, `+0x1cc`) are 0.0, else 0.0 ². | Boolean-into-slot shape; Mafia Script Data Reference: *"Returns the state of airplane xx into variable yy; 0 = not destroyed, 1 = destroyed"*. |

¹ Validates `xx` as an actor reference of type `0x16`. The worker assigns the nav route
(`+0x1c4`/`+0x110` waypoint table), target speed (`+0xb8`), path-aligned orientation, motion mode
`+0xb4` (1 if speed ≤ 0, else 2) and the moving anim phase via `airplane_play_sound`. Node args:
`pfVar26[1]` = airplane, `pfVar26[2]` = route id, `pfVar26[0]` = speed.
² `yy` bounds `-1 < slot < param_1[0x13]`. The two engines are the ones the player must shoot; the
same pair drives the `airplaneshowdamage` gauge (§3.2).

### 3.8 Vehicles and physics props

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0xc0` / 192 | `carlight_main xx, yy` | `vehicle_set_headlights` (`0x005c8960`) | Main headlights of car `xx`: `yy` = 1 on, 2 on + override bit `0x00800000`, else off ¹. | Mafia Script Data Reference: *"Turns on the main lights of car xx. A night mission must however be started via `game_nightmission 1`"* — the decompiled gate is that flag. Verified in game; shipped use `carlight_main 1, 0`. |
| `0xc2` / 194 | `carlight_indic_l xx, yy` | `vehicle_set_left_indicator` (`0x005c88e0`) | Left indicator: lamp bit `0x1`; `yy` three-way, override bit `0x00100000` ². | Mafia Script Data Reference: *"Turns on the **left** indicator of car xx"*. Verified in game. |
| `0xc3` / 195 | `carlight_indic_r xx, yy` | `vehicle_set_right_indicator` (`0x005c8920`) | Right indicator: mirror of `0xc2`, lamp bit `0x2`. | Name string at `.rdata 0x0064f2cc`; Mafia Script Data Reference: *"Turns on the **right** indicator"*. Verified in game. |
| `0xcc` / 204 | `create_physicalobject ww, xx, yy, zz` | `public_physics_ctor` (`0x004cde20`) | Turns an existing scene frame into a free-falling physics prop ³. | Case body matched helper for helper; the Mafia Script Data Reference's 30-object cap equals the one `public_physics_game_init` enforces. |
| `0xcd` / 205 | `destroy_physicalobject xx` | `game_remove_temporary_actor` (`0x005e3400`) | Removes the prop created from frame `xx`: scans the mission actor list for kind **`0x20`** on that frame. | The `kind == 0x20` literal is unique to `C_public_physics`; Mafia Script Data Reference: *"Removes physical object xx created with create_physicalobject. xx = frame ID"*. |
| `0x134` / 308 | `frameistelephone xx, yy` | `game_telephone_list_contains_key` | `flt[yy] = 1.0` when frame `xx` is a registered callable device; 0.0 on miss or bounds failure ⁴. | Unique callee on a 2-int-in / 1-float-out shape; Mafia Script Data Reference: *"returns into variable yy whether frame xx is a telephone"*. |

¹ `xx` bounds against `C_program+0x5c`, fetched from the actor register file `C_program+0x58`, must be
kind 4 (`C_car`). Lamp = bit `0x10000` of the light word `C_Vehicle+0x19c` (= `car+0x20c`). The off
path also calls `car_set_auto_headlights_suppressed(1)`; without it `car_update` switches the lights
back on next frame, because it re-derives the bit every frame for any driven car while
`C_game+0x2d44` (`game_set_night_mission`) is set.
² Same lookup and kind-4 gate as `carlight_main`. Each branch first calls
`vehicle_set_auto_indicators_from_steering`, because bit `0x8` (indicators driven from the steering
angle) gates the side setter.
³ Allocates `C_public_physics` (kind `0x20`, `operator_new(0x3b0)`), calls `Init(frame)` (vtable slot
18; `Release`, slot 17, on failure), builds the body with `physical_object_init_box`, registers via
`game_add_temporary_actor`. `ww` = frame id (bounds-checked); `xx` = initial motion negated into Y
(`vec3_set_xyz(0, -xx, 0)`), so in this Y-up world **>0 = impulse down, 0 = none, <0 = up**; `yy` =
0 falls only once hit / 1 falls immediately (random-direction impulse scaled by weight); `zz` =
weight in kg, clamped against a floor and used as mass. The Mafia Script Data Reference: *"Changes object ww into a
physical one … At most 30 physical children can be created from one object"*;
`public_physics_game_init` retires the oldest when `0x1e < size`. See `structs/public_physics.md`.
⁴ `xx` bounds against `param_1[0x15]`, resolved via `param_1[0x14] + xx*4`, canonicalised with
`act_mission_get_game`; `yy` bounds against `param_1[0x13]`; failures jump to `LAB_005c7541`. The
helper walks `world+0x1c8..+0x1cc` (stride `0xc`) comparing entry `+0x00` only. The list, filled by
`game_load_telephones`, holds two device types distinguished by entry `+0x08` LSB — 0 alarm
(`alarm01.i3d`), 1 PHbox (`PHbox01.i3d`) — so a registered alarm frame also returns 1 (§6). Shipped
use: 11 calls in `EXTREME`, 1 in `MISE15-MESTO`.

### 3.9 Menus, freeride score, film sequences, racing

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x43` / 67 | `loaddifferences "ssss"` | `game_load_differences` (`0x005ea0c0`) | Loads a scene-difference `.chg` file for a film sequence ¹. | Mafia Script Data Reference: *"Loads difference file ssss for a film sequence … only a \*.chg file, with extension and without path … The data is released with cleardifferences."* |
| `0x61` / 97 | `gunshop_menu` | `gui_gun_shop_ctor` (`0x0057b3a0`) | Opens the weapon shop as a modal screen and blocks the script until it closes ². | Shipped use: twice in mission `MISE19-MESTO` and once in `MISE20-MESTO`, script `1ProdZ` (*prodavač zbraní*, the arms dealer), each right after `dialog_end` and before `inventory_push` / `inventory_clear`; zero args, modal, weapon-model refresh match. |
| `0xa6` / 166 | `racing_mission6_init ww, xx, yy, zz` | `race_mission6_init` | Creates `C_racing` if absent, marks a mission race, selects circuit `yy` (index into `g_racing+0x0c`), sets lap count, player car model `zz`, generates opponents, enables damage. | Mafia Script Data Reference: *"Initialises the race"*, with `yy`/`zz` documented as `???`; a statement-level diff against the PS2 body supplies them (§4). |
| `0xa7` / 167 | `racing_mission6_start xx` | `race_mission6_execute_loop` | Does not race inline: parks `&flt[xx]` at `game+0x2ee0`, sets `game+0x2edc = 2`, returns; `game_tick` then runs the race and writes the result. | Mafia Script Data Reference return map 0 failure / 1 completed / 2 failure / 3 `???` / other = racing — exactly `race_mission6_execute_loop`'s return map. |
| `0x149` / 329 | `setfilmmusic xx` | `game_set_film_music` (`0x005e9e10`) | Adds the track id held in `flt[xx]` to the film-sequence list at `C_game+0x2fc8`, with dedup. | Register-index validation; Mafia Script Data Reference: *"Sets the music track in a film sequence."* |
| `0x14a` / 330 | `getfilmmusic xx, yy` | `game_get_film_music` (`0x005e9fd0`) | Writes film-music handle `xx` into `flt[yy]`, or `-1` when out of range. | Mafia Script Data Reference: *"Returns the music into variable yy."* |
| `0x166` / 358 | `freeride_scoreon xx` | `game_freeride_score_set_on` | Shows / hides the freeride score (booleanized arg; writes player `+0x363e`). | Tier-2 body shape, no string anchor. |
| `0x167` / 359 | `freeride_scoreset xx` | `game_freeride_score_set` | Sets the score to `__ftol(xx)` via the shared tail `LAB_005c69b9` (sentinel `-1` takes the same path). | Tier-2; tail shared with `0x169`. |
| `0x168` / 360 | `freeride_scoreget xx` | `game_freeride_score_get` | Writes the score (player `+0x3640`) as a float into `flt[xx]` (slot bounds-checked). | Tier-2. |
| `0x169` / 361 | `freeride_scoreadd xx` | `game_freeride_score_get` + `game_freeride_score_set` | Adds `__ftol(xx)` to the score through `LAB_005c69b9`. | Tier-2. |

¹ Calls `game_clear_differences`, opens the file from `Mafia\diff` through a stack `C_chunk` (magic
`0x4d2`) and rebuilds its frames and objects into `C_game+0x2ae0` / `+0x2af0`. The engine's other
caller is `race_winners_replay_begin` with `difrace.chg` / `difracech.chg`.
² Design-`0x14` screen on the player's `G_Inventory` (`player+0x480`), `gui_menu_execute_modal(1)`,
console clear, then `human_change_weapon_model`. Left band = what the player carries (frees capacity
via `inventory_remove_slot`); right band = every `g_game_item_table` entry without flag `0x800000`.
Despite the window caption *"Inventory (Drop)"* (textdb 1300) it is a **pickup** screen — textdb 1310
labels the right band *"Select (Pick Up)"*. No money or score is involved. See `structs/game_menus.md`
§4e.

### 3.10 Audio, math, collision, developer commands

| Opcode | Command | Handler | Behaviour | Evidence |
|---|---|---|---|---|
| `0x11e` / 286 | `quat_normalize` | `S_quat::Normalize` | Normalizes a quaternion register slot. | Callee + single-quat arg layout. |
| `0x11f` / 287 | `quat_rotbymatrix` | `S_quat::RotateByMatrix` | `quat ← M * quat`. | Callee + arg layout. |
| `0x120` / 288 | `quat_slerp` | `S_quat::Slerp` | Spherical-linear interpolation between two quats. | Callee matches the 3-arg slerp. |
| `0x121` / 289 | `coll_testline` | `collision_test_line_vertical_thunk` | 5-arg ray test; boolean hit into a `flt[]` slot. | Callee; `VLVP` is the next opcode, `0x122`. |
| `0x138` / 312 | `citymusic_off` | `game_city_music_set_paused(this, 1)` | Disables city music, zero args ¹. | Verified in game with untouched and custom scripts. |
| `0x139` / 313 | `citymusic_on` | `game_city_music_set_paused(this, 0)` | Re-enables city music, zero args ¹. | Verified in game; keyword table stores `CITYMUSIC_ON = 0x139` (§2). |
| `0x185` / 389 | `dan_internal_1` | `car_ctor` | Developer cheat, zero args: spawns a Thunderbird (`thunderbird00.i3d`, label `cheat_car`) at `(-841.0, -6.5, -637.0)` ². | Literals `s_cheat_car_0064ff98` + `s_thunderbird00_i3d_0064ff84`; spawn verified in game. |
| `0x186` / 390 | `dan_internal_2` | `game_play_3d_sound` | Developer cheat, zero args: plays `system\m_pravda.wav` ("pravda" = Czech "truth") ³. | Literal `s_system_m_pravda_wav_0064ff70`; verified in game. |

¹ Both set `script_scratch_u0` (1 / 0) and fall into `LAB_005c5109` →
`game_city_music_set_paused`, which writes `this+0x2fc0`. The per-frame zone-driven ticker reads that
byte to skip track selection; the disable path also calls `game_city_music_off` to stop the active
track with a 5-second fade. After `citymusic_on` a track starts on the next evaluation pass. The
third-party "MafiaScript Extensions v1.1.0" plugin present during testing does not hook either command.
² Factory-allocates a type-9 (vehicle) object via `g_i3d_driver->vt[0x4c](this, 9)`, tags it
`cheat_car` (vt `0x28`), loads the model with `i3d_cache_model_open`, allocates a `0x21f4`-byte
controller (`operator_new` + `car_ctor`), sets the position via `i3d_frame_set_position_xyz_ptrs`
from the literals `0xc4524000` / `0xc0d00000` / `0xc41f4000`, facing via `vec3_set_xyz(1.0, 0, 0)` +
vt `0xc` / `0x18`, registers via `i3d_scene_set_frame_sector_pos`. On controller vt `0x48` success it
runs vt `0x64`, `vehicle_controller_set_field_0x214c(1)`, `act_mission_get_game`,
`game_add_temporary_actor_intern`, `vehicle_settle(100)`, `vehicle_controller_set_field_0x65(1)` (§6).
³ Allocates a type-4 (audio) object through the same `g_i3d_driver->vt[0x4c]` factory, loads the wav
(vt `0x50`), configures it (vt `0x58/0x68/0x64/0x6c/0x24`: channel, volume 1.0, pitch/pan 1.0, no
loop, autoplay), positions it at the player via `act_mission_get_game`, finalises with
`game_play_3d_sound`; on load failure calls vt `0` (release).

### 3.11 Commands whose slot points at the default arm

15 jump-table slots point at the shared `0x005c76da`, which only calls `program_next_command(NULL)`
(advance IP by 1). They are listed in the header of [`opcode-table.md`](./opcode-table.md): `BEEP`,
`DETECTOR_INRANGE`, `DETECTOR_WAITFORUSE`, `DETECTOR_WAITFORHIT`, `PROGRAM_STORAGE`, `DEBUG_TEXT`,
`EVENT`, `[BREAK]`, `[DEBUG_ME]`, `[DEBUG_STOP]`, `DEBUG_GETFRAMEINFO`, `[DAN_DEBUG]`,
`DETECTOR_CREATEDYNCOLL`, `DETECTOR_ERASEDYNCOLL`, `HUMAN_CANDIE`. `END` (`0x6`), `SETEVENT` (`0x49`)
and `DOOR_LOCK` (`0x26`) each have their own arm.

Not listed in §3 because the Mafia Script Data Reference gives no opcode-level match yet: `CLEARDIFFERENCES` (`0x44`,
body `game_clear_differences`) and `RECUNLOAD` (`0x2e`, body tail-calls `game_clear_film_music`).

## 4. PC vs PS2

| PC command | PS2 corroboration |
|---|---|
| `racing_mission6_init` (`0xa6`) | PS2 twin `RacingMisse06__FUiUiUiUi`, reached at PS2 dispatcher `case 0xa9`. The diff supplies the Mafia Script Data Reference's `???` parameters (`yy` circuit, `zz` car model). PS2 clamps `xx` to 5; the PC does not. |
| `racing_mission6_start` (`0xa7`) | PS2 `case 0xaa` + `RacingMisse06ExecuteLoop__Fv` are statement-for-statement identical, at `+0x1304` / `+0x1308`. |
| `m7_zastavzkurvenepolise` (`0x183`) | `police_board_release_all_group_vehicles` is byte-structurally identical to `ZastavMotorizovaneJednotky__18ai_police_nastenkaFv` (PS2 op `0x18b`; "halt the motorized units"). |
| `carlight_indic_l` / `_r` (`0xc2` / `0xc3`) | PS2 `DoLights__9C_VehicleFf` lights bit `0x1` for a negative steering value and bit `0x2` for a positive one. |
| `gunshop_menu` (`0x61`) | The screen's PS2 twin is `GM_GunShop` (`OnCreate__10GM_GunShopFv` / `OnClick__10GM_GunShopFUi`), matched on five shared constants at identical sites: start index 2, the `0x800000` flag test, base row id 1000, textdb base `0xdac`, the `0x44c` template argument. |
| `getfilmmusic` (`0x14a`) | The PS2 twin is gutted to a bare `return -1`, which preserves exactly this function's failure return. |

## 5. How to add a verified entry

1. Pick a case of `program_process_command`, preferably one with a distinctive callee;
   `opcode-table.md` names its command.
2. Read the case body in `Game-ghidra.exe.annotated.c` and identify the distinctive helper(s).
3. Find the command in the Mafia Script Data Reference and check the description matches.
4. Add a row to the matching table in §3.
5. Add a `// NAME` comment on the `case` label in `Game-ghidra.exe.annotated.c`, and add the mapping
   to the docblock above `program_process_command`.
6. Record any helper renamed along the way in `Game-ghidra.exe.functions.md`.

## 6. Open items

| item | state |
|---|---|
| `policeitchforplayer` while pursued, unseen and not arrested | the body gives 0 (manager `+0x2c` is only set by the capture cutscene); the Mafia Script Data Reference's *"even when the icon is transparent"* suggests 1. Test: get chased, break line of sight without being arrested, run `policeitchforplayer 0`, read `flt[0]`. A 1 means `+0x2c` has another writer |
| `get_pm_state` — where `1 << zz` is produced | the AND operand is the shifted mask, so the shift happens earlier (script compile or VM argument decode); the site is not pinned |
| `get_pm_humanstate` result value | the helper returns level 0..3 and the float write is dropped by Ghidra; whether scripts see 0/1 (as the Mafia Script Data Reference documents) or 0..3 is unread |
| `get_pm_crashtime` / `get_pm_firetime` units | ms or game ticks — not established |
| `policemanager_del` with cops already spawned | reported to misbehave; not verified |
| `police_support` mode selector | `(yy != 0) ? 0 : 4` — meaning of 0 / 4 unknown; no in-game test |
| `wingman_setindicator` — which actor the indicator tracks | `xx` is not stored; the live source is elsewhere. The setter of `player+0x5e` is not traced |
| `wingman_delindicator` clearing every slot's pin | intentional or a bug — undetermined |
| `timeroff` | whether hiding the stopwatch also stops the countdown; and why `timeron` calls `pedestrian_manager_end_all_taxi_waves` |
| `frameistelephone` on an alarm frame | the helper ignores the `+0x08` type bit, so alarms should also return 1; whether registration filters them first is not located |
| `dan_internal_1` finalisation | whether the player is placed into the spawned car is not confirmed in game |
| `pm_setprogress` fill | in game it works only once per game start, and the first-call fill does not follow the clamp (§3.2 note ⁹) |
| `CLEARDIFFERENCES` (`0x44`) / `RECUNLOAD` (`0x2e`) | bodies read; Mafia Script Data Reference match not recorded |
