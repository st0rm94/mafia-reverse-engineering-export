# The race and multiplayer session drivers — PS2 `multiloop.cpp`

**Created** 2026-08-27 20:16 +0200 · **Modified** 2026-09-17 13:44 +0200

This is **not a class.** It is a translation unit of **thirteen global free functions**
that drive a game-mode session: the two race frame loops, the race runner, the two menu
loops, the networked-load handshake, and the mission-6 street race. It sits directly
above `g_net_session` (`structs/net_session.md`), which is the object most of it talks to.

## 1. The translation unit

| | |
|---|---|
| Range | `[0x00594740, 0x00596c50)` |
| Starts | 17 — 13 free functions, one COMDAT class ctor, one CRT init/atexit pair |
| Boundary evidence | `.CRT$XCU` slots `0x006461c0` (= `0x00594740`, opens it) and `0x006461c4` (= `0x00596c50`, opens the next TU), each confirmed by its own `crt_static_init_guard` / `crt_atexit_register` pair |
| Prefixes | `race_` (racing-session drivers) and `net_` (PC-only multiplayer drivers) |

**`event_msg_ctor` (`0x00596c20`) is not a member.** It is a COMDAT inline constructor of
the event-message class, emitted into whichever object file first used it; its own prefix
names its class. Likewise the `crt_*` pair is scaffolding.

### 1a. Why the members are free functions, not methods

Three independent measurements:

1. **Symbol shape** — every located twin is mangled `__Fv` / `__FRUi` / `__FUiUiUiUi` /
   `__Fb`: no class tag, `GLOBAL` binding (§5).
2. **Receiver census** — a census of the ECX value loaded before every callsite of nine of
   them finds **no consistent ECX**. The loads are leftovers: `mov ecx,1`,
   `mov ecx,[esp+0x24]`, `lea ecx,[eax-0x35]`, `mov ecx,[0x6f958c]`. Compare
   `net_session_is_active` in the next TU: **43 of 43** callsites load the same singleton.
3. **Field access** — no body touches a `this`-relative field. They reach their state
   through `g_racing`, `g_net_session` and `g_act_mission`.

`race_run`'s callers do `lea ecx,[esp+8]` / `lea ecx,[esp+0x10]`. That is **not** a `this` —
it is the `unsigned int&` out-parameter the PS2 mangling `__FRUi` declares, in which the
function returns the next menu code.

## 2. What it does — the two session shapes

```
engine_main ──► race_menu_run ──┬─ 0x2a/0x2b  set g_racing+0x2c = 1/2  ──► race_run
                              ├─ 0x2c       set g_racing+0x2c = 3    ──► race_run
                              ├─ 0x2d       replay load             ──► race_run
                              └─ 0x2e                          ──► net_multiplayer_run
                                                                        │
engine_main ────────────────────────────────────────────────────────────►─┘
                                                                        │
   net_multiplayer_run ─┬─ 0x36 net_open_session_browser                 │
                        ├─ 0x37 net_host_session_start                   │
                        ├─ 0x42..0xa5 join ─► net_wait_for_session_mode  │
                        └─ 0x3d start ─────► race_run                    │

race_run ─┬─ offline ─► race_game_loop
          └─ networked ─► race_net_load_sync ─► race_game_loop_net
   … then race_open_menu_scene and back to the right menu screen
```

`race_game_loop` also owns the **winners sequence**: when `g_racing+0x84` goes true it
ends the race, fakes the final positions, calls `racing_spawn_winners_scene`, and wraps a
recorded replay in `race_winners_replay_begin` / `_end`. The replay loads the scene-change file
`difrace.chg` / `difracech.chg` and plays `records\recordrace.rep` /
`records\recordracech.rep` through the `C_recorder` system.

## 3. Function map

| PC VA | PC name | PS2 twin | Role |
|---|---|---|---|
| `0x00594740` | `crt_static_init_guard_00594740` | - | TU-opening static init |
| `0x00594760` | `crt_atexit_register_00594760` | - | its atexit partner |
| `0x00594770` | `race_open_menu_scene` | `OpenMenuScene__Fv` | load `00menu`, `game_init`; clears every peer's ready flags first |
| `0x005947e0` | `race_winners_replay_begin` | - | `difrace.chg` + `records\recordrace.rep`, widescreen bars, `g_cutscene_active` |
| `0x005948f0` | `race_winners_replay_end` | - | the exact teardown |
| `0x005949c0` | `race_game_loop` | `GameLoopSingle1__Fv` | the offline race frame loop |
| `0x00594e90` | `race_game_loop_net` | - | the networked race frame loop |
| `0x00595390` | `race_net_load_sync` | - | scene announce, load screens, actor GUID table |
| `0x00595a30` | `race_run` | `RunRacing__FRUi` | load a circuit, run one race, pick the next screen |
| `0x00595d90` | `race_menu_run` | `ExecuteRacingMenu__Fv` | the Racing front-end loop |
| `0x00595f20` | `net_wait_for_session_mode` | - | block ≤ 25 s for the host's mode choice |
| `0x00595fb0` | `net_host_session_start` | - | connect, name, password, create as host, open lobby |
| `0x00596280` | `net_open_session_browser` | - | connect, enumerate, open the join list |
| `0x00596370` | `net_multiplayer_run` | - | the Multiplayer menu loop |
| `0x00596ad0` | `race_mission6_init` | `RacingMisse06__FUiUiUiUi` | script `racing_mission6_init` |
| `0x00596bd0` | `race_mission6_execute_loop` | `RacingMisse06ExecuteLoop__Fv` | script `racing_mission6_start` |
| `0x00596c20` | `event_msg_ctor` | - | not a TU member — a COMDAT ctor of the message class |

## 4. The mission-6 street race

The Mafia Script Data Reference (section R) documents two commands that are this TU's smallest
pair:

| script command | dispatcher case | PC function |
|---|---|---|
| `racing_mission6_init ww, xx, yy, zz` | `0xa6` | `race_mission6_init` |
| `racing_mission6_start xx` | `0xa7` | `race_mission6_execute_loop` (run from `game_tick`) |

The two bodies define the parameters the reference leaves as `???`:

| parameter | what it is |
|---|---|
| `ww` | lap count → `g_racing+0x27c` |
| `xx` | opponent count → `racing_generate_opponents`, passed through unclamped |
| `yy` | **circuit index** into the `g_racing+0x0c` circuit vector |
| `zz` | **the player's car-model id** → `g_racing+0x64` |

`racing_mission6_start` does not run inside the dispatcher. Case `0xa7` parks the script
variable's address at `game+0x2ee0` and sets `game+0x2edc = 2`; `game_tick` then runs the
whole race synchronously and writes the outcome back as a float. The reference's return
codes — 1 won, 2 lost, 0 other, 3 unset — are exactly this body's.

Mission 6 ("Fair Play") is the only consumer of `race_mission6_*`.

## 5. PC vs PS2

### 5a. The twin band

The PS2 counterparts are **seven free functions** at `0x0038f9f0`-`0x00390560`, sitting
between `Mission.cpp`'s container instantiations and `pathweb.cpp`'s `C_web_node` — exactly
where case-insensitive alphabetical link order puts **`multiloop.cpp`**, whose
`__sinit_multiloop.cpp` symbol is in the PS2 source-file manifest.

| PS2 VA | PS2 symbol | size | PC twin |
|---|---|---|---|
| `0x0038f9f0` | `OpenMenuScene__Fv` | 68 | `race_open_menu_scene` |
| `0x0038fa40` | `PlayMovie__Fb` | 128 | - (§5b) |
| `0x0038fac0` | `GameLoopSingle1__Fv` | 1172 | `race_game_loop` |
| `0x0038ff60` | `RunRacing__FRUi` | 780 | `race_run` |
| `0x00390270` | `ExecuteRacingMenu__Fv` | 480 | `race_menu_run` |
| `0x00390450` | `RacingMisse06__FUiUiUiUi` | 268 | `race_mission6_init` |
| `0x00390560` | `RacingMisse06ExecuteLoop__Fv` | 140 | `race_mission6_execute_loop` |

**Six of seven mapped.** The PS2 also has a separate `GameLoopSingle__Fv` at `0x00375090`
in a different TU — that is the MISSION loop, not this one.

### 5b. `PlayMovie` is a platform substitution, not an inlining

At the point where PS2 `GameLoopSingle1` calls `PlayMovie__Fb(uVar4)` — right after
`FakeFinalPositions` — the PC calls `racing_spawn_winners_scene` and then
`race_winners_replay_begin`, and later `race_winners_replay_end`.

* PS2 `PlayMovie__Fb` builds a filename, resolves it with `GetAnimationPath__9C_mission`
  and hands it to **`LSI_PlayPSS__FPcb`** — a `.pss` video stream.
* The PC has no PSS decoder; it plays the recorded in-engine replay (§2).

Nothing here is inlined on the PC, in either direction. The two PC functions are a *different
implementation of the same beat*, which is why they have no PS2 twin rather than `PlayMovie`.

### 5c. Divergences inside the matched bodies

| PC | PS2 | note |
|---|---|---|
| in-race menu codes `0x21` / `0x22` / `0x23` | `0x26` / `0x27` / `0x28` | the uniform **+5** shift measured across the GUI result codes; `race_menu_run`'s `0x2a`..`0x2d` ↔ `0x2f`..`0x32` is the same delta |
| `race_mission6_init` passes the opponent count straight through | `RacingMisse06` clamps it to **5** | a shipped behavioural difference, not a port artifact |
| `race_menu_run` case `0x2e` → `net_multiplayer_run` | absent | the PS2 build has no networking at all |
| `race_run` branches into `race_net_load_sync` + `race_game_loop_net` | absent | same reason |
| - | `ZpicenyPleaseWait`, `LSI_CdTrayCheck`, `LSI_TestOurCD`, `C_cache_base_block::Tick`, `C_silver_screen::Tick`, `PauseAllSounds`/`PlayAllSounds` | PS2-port-only calls inside `GameLoopSingle1` (CD tray, streaming cache, loading screen), not PC inlinings |

## 6. Related

`structs/net_session.md` (the object this TU drives) · `structs/racing.md` (`C_racing`) ·
`structs/racing_menus.md` (the screens `race_menu_run` opens) ·
`structs/multiplayer.md` (the front end `net_multiplayer_run` opens) ·
`script-mapping/artifacts/opcodes-verified.md` (cases `0xa6` / `0xa7`).

## 7. Open items

| item | state |
|---|---|
| does the multiplayer path still reach a race? | `net_multiplayer_run` is reachable from `race_menu_run` case `0x2e` even though the main-menu Multiplayer item was removed (`structs/multiplayer.md` §1a); not tested in game |
| more than five opponents in mission 6 | the PC does not clamp the opponent count (§5c); not tested in game |
