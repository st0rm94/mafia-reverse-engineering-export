# Mafia: The City of Lost Heaven (PC, 2002) — reverse-engineering export

<!-- version -->
**Export version:** `2026.09.28`
<!-- /version -->

Created 2026-09-28 18:41 +0200 · history in `CHANGELOG.md`

## What this is

- A reverse-engineering study of the retail PC `Game.exe` (image base `0x00400000`, no symbols, no RTTI).
- **Game version: 1.0** — version resource FileVersion `Master 1.0`, PE build timestamp `0x3d51286b` =
  **2002-08-07 14:02:19 UTC** (read from the retail executable).
- The analysed file is a cracked no-CD copy of that build, because the retail code is SafeDisc 2.70.030
  encrypted: 2,486,272 bytes, MD5 `92419435416499862cc981199566c14b`. Its PE timestamp field was
  overwritten by the crack (`GiMP`); details in `basic/artifacts/crack.md`.
- Goal: a readable, fully named decompilation plus documentation of the game's systems
  (police/pursuit, script VM, AI, vehicles, HUD, menus, racing, save/load, CRT/runtime).
- This directory is a partial copy of the project: the documentation set and the decompiler outputs.
  Tools, trackers and working notes are not included.
- **AI-assisted:** the whole reverse-engineering effort is AI-assisted. The analysis, the names,
  the comments in the annotated decompilation and all documentation were produced with
  [Claude Code](https://claude.com/claude-code) (Anthropic), directed and reviewed by the author,
  with in-game verification by the author.

## Contents

### Documentation (`.md` files cross-link each other by relative path)

| directory | files | what |
|---|---:|---|
| `basic/artifacts/` | 11 | gameplay and engine write-ups |
| `police-system/artifacts/` | 5 | the police and offence system |
| `structs/` | 116 | one doc per class/subsystem: layout, fields, methods, PC↔PS2 twin tables |
| `def-files-docs/` | 5 | shipped data formats and tables |
| `script-mapping/artifacts/` | 3 | the mission-script VM: opcode ↔ command ↔ handler |

- **basic/artifacts:** `cheats.md`, `crack.md`, `dll.md`, `freeride-score.md`, `game-loop.md`,
  `hud_elements.md`, `npc-player-interactions.md`, `player-interactions.md`, `racing.md`,
  `script-opcode-dispatch.md`, `vehicle-movement.md`
- **police-system/artifacts:** `boosting.md`, `offences.md`, `police-board-action.md`,
  `police-manager.md`, `police-system.md`
- **structs:** `actor.md`, `actor_shot.md`, `actor_track.md`, `ai_action_manager.md`, `ai_blob_log.md`,
  `ai_character.md`, `ai_check.md`, `ai_fight_manager.md`, `ai_fight_modes.md`, `ai_follow_manager.md`,
  `ai_ghost.md`, `ai_group.md`, `ai_loc.md`, `ai_logs.md`, `ai_manager.md`, `ai_path.md`, `ai_sensors.md`,
  `ai_space_coverage.md`, `ai_talkman.md`, `ai_wayfind.md`, `airplane.md`, `anim_machine.md`,
  `app_bootstrap.md`, `bendable_plant.md`, `blood.md`, `bottle.md`, `box.md`, `box_new.md`, `bridge.md`,
  `cache_base_block.md`, `camera.md`, `car.md`, `car_database.md`, `car_extended.md`, `carcyclopedia.md`,
  `cartridge.md`, `check_seg.md`, `chunk.md`, `clock.md`, `collision.md`, `console.md`, `crash_obj.md`,
  `crash_report.md`, `crt_runtime_tail.md`, `debris.md`, `def_tables.md`, `detector.md`, `dog.md`,
  `door.md`, `driver.md`, `drop_in_weapon.md`, `dropout.md`, `entity.md`, `fake_shadow.md`, `fire.md`,
  `force_feedback.md`, `game.md`, `game_menus.md`, `garage.md`, `glass_shatter.md`, `grenade.md`,
  `gui_screen.md`, `gui_widget.md`, `hud.md`, `human.md`, `i3d_cache.md`, `i3d_frame.md`, `input.md`,
  `inventory.md`, `lightness.md`, `line_batch.md`, `loadsave.md`, `mission.md`, `multiplayer.md`,
  `net_event_msg.md`, `net_session.md`, `particle.md`, `pedestrian.md`, `physical_object.md`, `player.md`,
  `police-action.md`, `police-board.md`, `police_manager-reconstruction-notes.md`, `police_manager.md`,
  `program.md`, `public_physics.md`, `pumper.md`, `pursuit-event-manager.md`, `pursuit_group_manager.md`,
  `race_session.md`, `racecam.md`, `racing.md`, `racing_menus.md`, `railway.md`, `railway_switch.md`,
  `random_morph.md`, `recorder.md`, `roads.md`, `safedisc_stub.md`, `screen_fade.md`, `seagull_flock.md`,
  `shot_skeleton.md`, `smoke_trail.md`, `soundArena.md`, `sound_table.md`, `speech.md`, `spike_strip.md`,
  `string.md`, `textdb.md`, `traffic_car.md`, `using_object.md`, `uv_scroll.md`, `vehicle.md`, `wagon.md`,
  `web_node.md`, `web_path.md`
- **def-files-docs:** `materialy.md`, `predmety.md`, `textdb.csv`, `textdb.json`, `vehicles-bin.md`
- **script-mapping/artifacts:** `dispatcher-symbols.md`, `opcode-table.md`, `opcodes-verified.md`

### Decompiler outputs (`ghidra/`)

| file | what |
|---|---|
| `Game-ghidra.exe.annotated.c` | **the annotated PC decompilation** — the main deliverable (19 MB, VA-sorted) |
| `Game-ghidra.exe.functions.md` | function registry: VA · Ghidra id · project name · PS2 twin (5,098 rows) |
| `Game.exe.gzf` | Ghidra 11.4.2 database with all project names and data types (File → Import File) |
| `Game-racing-mod.exe.c` | raw export of the community racing-mod executable (comparison binary) |
| `SLUS_206.71.c` | raw export of the PS2 build (unstripped — carries the developers' symbols) |
| `SLUS_206.71.functions.md` | PS2 symbol table: mangled + demangled names |

## What changed in the annotated source (vs Ghidra's default auto-analysis of `Game.exe`)

<!-- stats -->
| change | result |
|---|---|
| function names | 5,098 functions named, **0 `FUN_` left**; each docblock keeps `Renamed from FUN_xxxxxxxx` |
| compiler glue | 385 `EHHandler_*` and 1,290 `Unwind_*` funclets named by owner |
| PS2 twins | 2,651 functions mapped to their original PS2 developer method (`PS2 twin:` line + registry column) |
| docblocks | ~5,400: what it does, parameters, return value, evidence |
| inline comments | ~9,800 `//` lines inside bodies; ~400 per-`case` docblocks in the script/AI dispatchers |
| globals and data | ~1,100 `g_*` globals + ~1,700 `s_*` strings named; raw `DAT_` tokens 1,253 → 14 |
| data types | arrays, vectors, tables and structs typed in Ghidra and spliced back into the source |
| function set | missing function starts recovered (e.g. the CRT tail, callbacks); a lost jump table restored; ~300 phantom functions invented by the Aggressive Instruction Finder removed |
| cleanup | Ghidra `WARNING` lines removed (2,688 → 3); records sorted by VA |
<!-- /stats -->

## Approach

- **PS2 cross-build matching:** the PS2 build keeps its symbols; PC functions are matched to PS2 methods by
  diffing both bodies (callees, constants, field offsets, vtable slots, class emission order).
- **Byte-level probes of `Game.exe`:** disassembly of call sites, ECX receiver analysis for `__thiscall`,
  vtable and `.rdata` table dumps, field read/write censuses.
- **Shipped game data:** `.def` tables, mission scripts, `textdb` strings, scene files and asset names
  used to name tables, commands and features.
- **Headless Ghidra passes:** function-start recovery, data typing, jump-table override, name import;
  each re-export is spliced into the annotated file surgically so existing comments survive.
- **In-game verification:** Cheat Engine watches and play tests; an in-game observation outranks static reading.
- **Evidence grades:** names and twins are tagged CONFIRMED / STRUCTURAL / CANDIDATE by the strength of the evidence.
