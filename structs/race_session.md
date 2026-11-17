# The race and multiplayer session drivers

**Created** 2026-08-27 20:16 +0200 · **Modified** 2026-11-17 10:40 +0100

This is **not a class.** It is a translation unit of **fourteen global free functions** that drive a game-mode session: the two race frame loops,
the race runner, the two menu loops, the networked-load handshake, the winners replay and the mission-6 circuit race. It sits directly above
`g_net_session` (`structs/net_session.md`), which is the object most of it talks to.

| | |
|---|---|
| PC prefix | `race_`, `net_multiplayer_`, `net_host_`, `net_open_`, `net_wait_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x00594740, 0x00596c50)` |
| PS2 class | free functions of `multiloop.cpp`: 6 mapped, 1 PS2-only (§7) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

The unit owns no global and no record. Everything it touches belongs to another document, so there is no typedef here and this section lists what
the drivers write and test, under the owners' names. §2 gives the functions.

| Owner | Doc | What the unit uses it for |
|---|---|---|
| `C_racing`, `g_racing` | `structs/racing.md` | creates it for mission 6 and for a hosted race, selects the mode and the laps, and reads the finish state |
| `g_net_session` and its peer records | `structs/net_session.md` | session state, the error id, the load handshake |
| `C_game` | `structs/game.md` | `running`, `player_in_death_volume`, `clear_differences_request` |
| `C_mission`, `g_act_mission` | `structs/mission.md` | `+0x70` / `+0x74` around the circuit load, `+0x78` the count of frames that skip the engine's vtable `+0x3c` call |
| the HUD | `structs/hud.md` | `g_hud_flags` and its save stack, around the winners replay |

## 2. Field evidence

The cells list only this unit's functions; the full census of each field is in its owner's document.

### 2a. `C_racing`

Fields inside the embedded `player_entrant` (`+0x40`) carry the `C_opponent` name.

| Owner offset | Field | Conf | Written here by | Read here by | Notes |
|---|---|---|---|---|---|
| +0x03 | `mission6_race` | H | `race_mission6_init` | `race_game_loop`, `race_mission6_execute_loop` | 1 from `race_mission6_init`; `race_mission6_execute_loop` returns 3 while it is clear |
| +0x04 | `rerun_race` | H | `race_run` | `race_game_loop`, `race_run` | `race_run` repeats the load and the loop while it is set |
| +0x28 | `menu_mode` | H | `net_host_session_start`, `race_menu_run`, `race_mission6_init` | `race_game_loop`, `race_run` | 1 from a hosted championship, 2 from a hosted single race and from mission 6, 3 from a replay load |
| +0x2c | `race_mode` | H | `net_host_session_start`, `race_menu_run`, `race_mission6_init` | `race_game_loop`, `race_run` | picks the next screen in `race_run` |
| +0x34 | `current_round` | H | - | `net_multiplayer_run`, `race_run` | - |
| +0x38 | `current_championship` | H | - | `net_multiplayer_run`, `race_run` | - |
| +0x3c | `difficulty` | H | `race_mission6_init` | - | 1 |
| +0x64 | `player_entrant.entrant_id` | H | `race_mission6_init` | - | the script's `zz` (§6) |
| +0x84 | `player_entrant.place_settled` | H | - | `race_game_loop`, `race_game_loop_net` | true ends the race |
| +0xa8 | `player_entrant.finish_place` | H | - | `race_mission6_execute_loop` | §6 |
| +0x27c | `total_laps` | H | `net_host_session_start`, `race_menu_run`, `race_mission6_init` | `race_menu_run` | 99 in case `0x2a`, 1 in case `0x2b`, each saving the old value first; 3 from a hosted single race |
| +0x280 | `saved_total_laps` | H | `race_menu_run` | - | the saved `total_laps`, restored by `racing_end_race` |
| +0x2b4 | `mirror_track` | H | - | `race_run` | non-zero loads the `%s mirror` scene |
| +0x2b5 | `replay_playback` | H | `race_menu_run` | `race_run` | 1 for a replay; chooses `loadsave_replay_set_mode`'s two arguments |
| +0x2bc | `race_camera_timer` | H | `race_game_loop` | - | 0 |
| +0x2c0 | `race_camera_dwell` | H | `race_game_loop` | - | 0 |

### 2b. `g_net_session`

| Owner offset | Field | Conf | Written here by | Read here by | Notes |
|---|---|---|---|---|---|
| +0x98 | `frame_ms` | H | `race_game_loop_net` | - | the frame delta |
| +0xa5 | `send_paused` | H | `race_game_loop_net` | - | cleared when the timer below runs out |
| +0xa8 | `pause_elapsed_ms` | H | `race_game_loop_net` | `race_game_loop_net` | - |
| +0xac | `pause_timeout_ms` | H | `race_game_loop_net` | `race_game_loop_net` | 0 = no timer |
| +0x100 | `state` | H | - | `net_host_session_start`, `net_multiplayer_run`, `net_open_session_browser`, `net_wait_for_session_mode`, `race_game_loop_net`, `race_net_load_sync` | 0 ends every wait |
| +0x104 | `load_wait_state` | H | `race_net_load_sync` | - | 6 and 7 for the two load screens, 8 when done |
| +0x1fc | `last_error_id` | H | - | `net_multiplayer_run` | the message box's text id |
| +0x200 | `is_host` | H | - | `net_multiplayer_run`, `race_net_load_sync` | - |
| +0x250 | `local_player.guid_table_acked` | H | `race_net_load_sync` | - | set to 1 after the GUID table goes out |
| +0x251 | `local_player.scene_loaded` | H | `race_net_load_sync` | - | set to 1 by the host on entry |
| +0x268 | `players_begin` | H | - | `race_net_load_sync`, `race_open_menu_scene` | the peer walks below |
| +0x274 | `host_peer_id` | H | - | `race_game_loop_net`, `race_net_load_sync` | the sender of the posted messages |

### 2c. The peer record

| Owner offset | Field | Conf | Written here by | Read here by | Notes |
|---|---|---|---|---|---|
| +0x48 | `load_progress` | H | `race_net_load_sync`, `race_open_menu_scene` | - | cleared with `scene_loaded` |
| +0x4c | `guid_table_acked` | H | - | `race_net_load_sync` | all set sends message `0xe` and state 7 |
| +0x4d | `scene_loaded` | H | `race_net_load_sync`, `race_open_menu_scene` | `race_net_load_sync` | all set lets the host announce the scene |

## 3. Method family

6 of the 7 PS2 functions of `multiloop.cpp` are mapped. The members are free functions (§4a), so 3d is the main table. The `race_compare_*` comparators
and the other `net_` helpers report with T2's prefixes and are hosted by `structs/racing.md` and `structs/net_session.md`.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00594740 | `crt_static_init_guard_00594740` | the translation unit | - | - | - | the unit's `.CRT$XCU` entry, slot `0x006461c0` |
| 0x00594760 | `crt_atexit_register_00594760` | the translation unit | - | - | - | registers the no-op `crt_atexit_noop_dtor_0060b700` |

### 3b. Accessors

The unit has no accessor.

### 3c. Behaviour

The unit has no function with a `this`.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00594770 | `race_open_menu_scene` | `OpenMenuScene__Fv` | 0x0038f9f0 | CONFIRMED | opens `00menu`, clears every peer's `scene_loaded` and `load_progress` when a session is active, then `game_init` |
| 0x005947e0 | `race_winners_replay_begin` | - | - | - | loads the difference file and the replay (the `ch` pair for a last championship round), widescreen bars, `g_cutscene_active` |
| 0x005948f0 | `race_winners_replay_end` | - | - | - | undoes the begin, and sets `clear_differences_request` |
| 0x005949c0 | `race_game_loop` | `GameLoopSingle1__Fv` | 0x0038fac0 | CONFIRMED | the offline race frame loop, including the winners sequence |
| 0x00594e90 | `race_game_loop_net` | - | - | - | the networked race frame loop; returns whether to go on |
| 0x00595390 | `race_net_load_sync` | - | - | - | scene announce, load screens, actor GUID table |
| 0x00595a30 | `race_run` | `RunRacing__FRUi` | 0x0038ff60 | CONFIRMED | loads a circuit, runs one race, returns the next screen |
| 0x00595d90 | `race_menu_run` | `ExecuteRacingMenu__Fv` | 0x00390270 | CONFIRMED | the Racing front-end loop |
| 0x00595f20 | `net_wait_for_session_mode` | - | - | - | waits up to 25000 ms for the host's `game_mode`, or until the session ends |
| 0x00595fb0 | `net_host_session_start` | - | - | - | connect, name by mode, password, create as host, open the lobby |
| 0x00596280 | `net_open_session_browser` | - | - | - | connect, enumerate, open the join list |
| 0x00596370 | `net_multiplayer_run` | - | - | - | the Multiplayer menu loop; frees `g_racing` on exit |
| 0x00596ad0 | `race_mission6_init` | `RacingMisse06__FUiUiUiUi` | 0x00390450 | CONFIRMED | script `racing_mission6_init` (§6) |
| 0x00596bd0 | `race_mission6_execute_loop` | `RacingMisse06ExecuteLoop__Fv` | 0x00390560 | CONFIRMED | runs the race for script `racing_mission6_start` (§6) |

## 4. The translation unit

17 starts lie in the range: the 14 functions, the CRT pair and `event_msg_ctor` (`0x00596c20`). Its boundaries are the `.CRT$XCU` slots `0x006461c0`
(= `0x00594740`, which opens it) and `0x006461c4` (= `0x00596c50`, which opens the next unit), each confirmed by its own guard and `atexit` pair.
`event_msg_ctor` is not a member: it is a COMDAT inline constructor of the event-message class, listed in `structs/net_event_msg.md`.

### 4a. Why the members are free functions, not methods

1. **Symbol shape:** every located twin is a free function with no class tag (§7d).
2. **Receiver census:** a census of the ECX value loaded before every callsite of nine of them finds no consistent ECX. The loads are leftovers:
   `mov ecx,1`, `mov ecx,[esp+0x24]`, `lea ecx,[eax-0x35]`, `mov ecx,[0x6f958c]`. Compare `net_session_is_active` in the next unit: **43 of 43**
   callsites load the same singleton.
3. **Field access:** no body touches a `this`-relative field. They reach their state through `g_racing`, `g_net_session` and `g_act_mission`.

`race_run`'s callers do `lea ecx,[esp+8]` and `lea ecx,[esp+0x10]`. That is **not** a `this`: it is an `unsigned int&` out-parameter (§7d). `race_run`
stores `0x100001` or 2 in it and **returns** the next screen; the callers keep that return value and do not read the out-parameter (Open item 3).

## 5. The two session shapes

```
engine_main --> race_menu_run --+-- 0x2a  race_mode = 1, laps 99 --> race_run
                                +-- 0x2b  race_mode = 2, laps 1  --> race_run
                                +-- 0x2c  race_mode = 3          --> race_run
                                +-- 0x2d  replay load            --> race_run
                                +-- 0x2e                         --> net_multiplayer_run

engine_main --> net_multiplayer_run --+-- 0x36 net_open_session_browser
                                      +-- 0x37 net_host_session_start
                                      +-- 0x42..0xa5 join --> net_wait_for_session_mode
                                      +-- 0x3d start, game_mode 3 or 4 --> race_run

race_run --+-- offline   --> race_game_loop
           +-- networked --> race_net_load_sync --> race_game_loop_net
   then race_open_menu_scene and back to the right menu screen
```

`net_host_session_start` names the session by `game_mode`, 1 Deathmatch, 2 Team Deathmatch, 3 Single Race, 4 Championship, and only the last two have
a start arm here.

`race_game_loop` also owns the **winners sequence**: when `player_entrant.place_settled` goes true it ends the race, fakes the final positions, calls
`racing_spawn_winners_scene`, and wraps a recorded replay in `race_winners_replay_begin` / `_end`. The replay loads the scene-change file `difrace.chg`
or, for the last round of a championship, `difracech.chg`, and plays `records\recordrace.rep` or `records\recordracech.rep` through the `C_recorder`
system.

## 6. The mission-6 circuit race

The Mafia Script Data Reference (section R) documents two commands that are this unit's smallest pair:

| Script command | Dispatcher case | PC function |
|---|---|---|
| `racing_mission6_init ww, xx, yy, zz` | `0xa6` | `race_mission6_init` |
| `racing_mission6_start xx` | `0xa7` | `race_mission6_execute_loop` (run from `game_tick`) |

The two bodies define the parameters the reference leaves as `???`:

| Parameter | What it is |
|---|---|
| `ww` | lap count, `total_laps` |
| `xx` | opponent count, to `racing_generate_opponents`, passed through unclamped |
| `yy` | **circuit index** into the `g_racing+0x0c` circuit vector; an index outside it keeps the current circuit |
| `zz` | **the player's car-model id**, `g_racing+0x64` |

`race_mission6_init` does its work only while `g_racing` is 0: it allocates the `0x428`-byte object, and a second call finds it and does nothing. Mission 6
("Fair Play", `MISE06-AUTODROM`) is the only shipped consumer; its `GameInitStart` script runs `racing_mission6_init 5, 12, 0, 44`.

`racing_mission6_start` does not run inside the dispatcher. Case `0xa7` stores the script variable's address through `game_set_race_result_register` and sets
`race_state` to 2 through `game_set_race_state`, then returns from the dispatcher so the script waits; `game_tick` runs the whole race synchronously, writes
the outcome to the variable as a float and sets `race_state` to 10. The value `race_mission6_execute_loop` returns:

| Return | When |
|---|---|
| 1 | `player_entrant.finish_place` is 1 |
| 2 | `finish_place` is 0 |
| 0 | any other `finish_place` |
| 3 | `g_racing+0x03` is clear |

The reference reads these as 1 won, 2 lost, 0 other and 3 unset.

## 7. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0038fa40 | `PlayMovie__Fb` | replaced: the PC plays a recorded replay instead (§7b) |

### 7a. The twin band

The PS2 counterparts are **seven free functions** at `0x0038f9f0`-`0x00390560`, between `Mission.cpp`'s container instantiations and `pathweb.cpp`'s
`C_web_node`, which is where case-insensitive alphabetical link order puts **`multiloop.cpp`**, whose `__sinit_multiloop.cpp` symbol is in the PS2 source-file
manifest. The sizes are `OpenMenuScene` 68, `PlayMovie` 128, `GameLoopSingle1` 1172, `RunRacing` 780, `ExecuteRacingMenu` 480, `RacingMisse06` 268 and
`RacingMisse06ExecuteLoop` 140. The PS2 also has a separate `GameLoopSingle__Fv` at `0x00375090` in a different unit, the MISSION loop.

### 7b. `PlayMovie` is a platform substitution, not an inlining

At the point where PS2 `GameLoopSingle1` calls `PlayMovie__Fb(uVar4)`, right after `FakeFinalPositions`, the PC calls `racing_spawn_winners_scene` and then
`race_winners_replay_begin`, and later `race_winners_replay_end`. PS2 `PlayMovie__Fb` builds a filename, resolves it with `GetAnimationPath__9C_mission` and hands
it to **`LSI_PlayPSS__FPcb`**, a `.pss` video stream. The PC has no PSS decoder; it plays the recorded in-engine replay (§5). The two PC functions are a different
implementation of the same beat, which is why they have no PS2 twin.

### 7c. Divergences inside the matched bodies

| PC | PS2 | Note |
|---|---|---|
| in-race menu codes `0x21` / `0x22` / `0x23` | `0x26` / `0x27` / `0x28` | the uniform **+5** shift measured across the GUI result codes; `race_menu_run`'s `0x2a`..`0x2d` against `0x2f`..`0x32` is the same delta |
| `race_mission6_init` passes the opponent count straight through | `RacingMisse06` clamps it to **5** | a shipped behavioural difference |
| `race_menu_run` case `0x2e` calls `net_multiplayer_run` | - | the PS2 build has no networking at all |
| `race_run` branches into `race_net_load_sync` and `race_game_loop_net` | - | the same reason |
| - | `ZpicenyPleaseWait`, `LSI_CdTrayCheck`, `LSI_TestOurCD`, `C_cache_base_block::Tick`, `C_silver_screen::Tick`, `PauseAllSounds`, `PlayAllSounds` | PS2-port-only calls inside `GameLoopSingle1` (CD tray, streaming cache, loading screen) |

### 7d. The symbol evidence for free functions

Every located PS2 twin is mangled `__Fv` / `__FRUi` / `__FUiUiUiUi` / `__Fb`: no class tag, `GLOBAL` binding. The `unsigned int&` out-parameter of `race_run` is the one
the mangling `RunRacing__FRUi` declares.

## 8. Related

`structs/net_session.md` (the object this unit drives) · `structs/racing.md` (`C_racing`) · `structs/racing_menus.md` (the screens `race_menu_run` opens) ·
`structs/multiplayer.md` (the front end `net_multiplayer_run` opens) · `script-mapping/artifacts/opcodes-verified.md` (cases `0xa6` / `0xa7`).

## 9. Open items

| Item | State |
|---|---|
| in game: the multiplayer path | `net_multiplayer_run` is reachable from `race_menu_run` case `0x2e` even though the main-menu Multiplayer item was removed (`structs/multiplayer.md` §4a); whether it still reaches a race is not tested |
| in game: more than five opponents in mission 6 | the PC does not clamp the opponent count (§7c); not tested |
| `race_run`'s out-parameter | written `0x100001` or 2; `race_menu_run` passes the slot's address and never reads it. What the value means was not read |
