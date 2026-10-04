# Game loop — from the CRT entry point to player interaction

**Created** 2026-07-19 11:24 +0200 · **Modified** 2026-09-30 23:08 +0200

Two nested loops run the whole game: `engine_main`, the outer menu / mission state machine, and
`engine_game_loop`, the inner per-frame loop a mission runs inside. Every function named below has
its definition in the annotated decomp under that name; `Game-ghidra.exe.functions.md` is the
registry that maps it to its VA.

## 1. Process entry → `engine_main` → `engine_game_loop`

```
crt_startup              entry, 0x00626f1a   CRT bootstrap
  └─ winmain             0x00623e40          GetModuleFileNameA → crt_chdir(exe_dir) → engine_main
      └─ engine_main     0x005f99b0          OUTER menu / mission state machine
           ├─ CreateMutexA("Mafia Launcher Super Mutex Shit")   single-instance guard
           ├─ the TEV and CD checks, then the exception handlers
           ├─ reads HKLM\Software\Illusion Softworks\Mafia → "LS3D_setup" over the defaults;
           │  a blob whose version tag is not 0x14 sends the user to setup.exe
           ├─ engine_init, then six persisted quality settings into C_mission+0x90
           └─ do { gui_menu_execute_modal(1) → switch(result) } while (no exit code)
                  case 0x12 → intro, then mise01
                      └─ engine_game_loop   0x005f9540   INNER per-frame loop (§3)
```

`engine_main` pumps the front-end menu through `gui_menu_execute_modal` and launches a mission by
calling `engine_game_loop`, which returns only when the mission ends or the ESC menu quit is taken.
On the way out `engine_main` copies the live settings back into `g_ls3d_setup`, writes the registry
value and calls `engine_shutdown`.

| menu result | what it starts |
|---|---|
| `0x11` | tutorial |
| `0x12` | intro cutscene, then `mise01` |
| `0x13` | load game |
| `0x14` | multiplayer |
| `0x15` | racing |
| `0x16` | intro cutscene on its own |
| `0x17` | credits |
| `0x18` | leaves the menu loop |
| `0x19` | Carcyclopedia (`autosalon`) |
| `0x1a` | leaves the menu loop |
| `0x1b` | extreme |
| `0x1c` – `0x20` | the five free-ride variants, in order: `freeitaly`, `freeride`, `freeridenoc`, `freekrajina`, `freekrajinanoc` |

## 2. Mission startup

`engine_main` sets a story mission up in two phases. Menu result `0x12` is the worked example.

**Phase 1 — the cutscene:**

```
mission_close()
mission_open("intro", 0, 0xffffffff, 1)
collision_load_level("missions\intro\tree.klz", 1)
g_act_mission+0x70 = 0
g_act_mission+0x74 = 0
game_init()
g_in_cutscene = 1
engine_game_loop()                  ← runs until the intro ends
g_in_cutscene = 0
mission_close()
```

**Phase 2 — gameplay:**

```
mission_open("mise01", 0, 0xffffffff, 1)
collision_load_level("missions\mise01\tree.klz", 1)
g_act_mission+0x70 = 0
g_act_mission+0x74 = 0
loadsave_savegame_new_game()
game_init()
engine_game_loop()                  ← the player plays in here
```

`tree.klz` is the level's **collision tree**, not a script: `collision_load_level` is the same entry
point the script command `loadcolltree` reaches, and a `loadcolltree` call replaces the level
collision wholesale. Layout in `structs/collision.md`.

## 3. `engine_game_loop` — the frame

`engine_game_loop` returns immediately unless the `C_game` running flag
`*(*(g_act_mission + 0x24) + 0x40)` is set, then spins as a `do { } while (true)` until that same
flag clears. It IS the running game — every frame of gameplay happens inside it.

```c
(*g_engine + 0xec)(g_engine);                    // one-time pre-loop engine init
if (!*(*(g_act_mission + 0x24) + 0x40)) return;

do {
    (*g_engine + 0x2c)(g_engine);                // begin frame: poll input, advance the timer
    (*g_engine + 0xcc)(g_engine);                // world update: mission script, AI, player, crowd
    (*g_engine + 0x94)(g_engine);                // physics and collision
    mission_state = (*g_engine + 0x98)(g_engine);
    (*g_engine + 0xf0)(g_engine, 0x10, 0x53);    // prepare render
    for (d in g_joy_device_list) (*d + 0x58)(1); // pump every ijoy.dll device
    input_update(0);                             // resolve every binding into the action tables
    state_recheck = (*g_engine + 0x98)(g_engine);
    if (state_recheck) { /* the cheat check, below */ }
    if (running) recorder_advance_clock(&delta);
    mission_tick(delta_ms);                      // sound mgr, renderer, display sub-objects
    recorder_tick_tracks(delta_ms);              // C_recorder keyframe playback
    if (g_cache_base_block_loaded) cache_base_block_tick(delta_ms);
    /* mission-end and ESC menu — §4 */
    /* display-state word — §5 */
    hud_tick(delta_ms);                          // minimap, race/mission timer
    (**(g_act_mission + 0x10) + 0x54)(...);      // present / swap buffers
    if (*(g_act_mission + 0x78) < 1) (*g_engine + 0x3c)(g_engine);
    else                            (*(g_act_mission + 0x78))--;
    if (g_fade_active)    screen_fade_tick(delta_ms);
    if (g_reload_pending) { g_reload_pending = 0; engine_new_mission(); }
} while (*(*(g_act_mission + 0x24) + 0x40));
```

### 3a. Engine vtable calls (on `g_engine`)

| offset | called when | role |
|---|---|---|
| `+0xec` | once before the loop | pre-loop engine init |
| `+0x2c` | every frame | begin frame: poll input, advance the timer, set `delta_ms` |
| `+0xcc` | every frame | world update: mission script, AI, player movement |
| `+0x94` | every frame | physics and collision |
| `+0x98` | twice per frame | mission state (0 running, 1 ended normally, `0x39` / `0x1c` other terminal states) |
| `+0xf0` | every frame | prepare render, arguments `0x10` and `0x53` |
| `+0xb8` | when `state_recheck` ≠ 0 | active screen flags |
| `+0xbc` | when `state_recheck` ≠ 0 | active screen object, index `flags & 0xf` |
| `+0x30` | inside `engine_new_mission` | engine pre-reload step |
| `+0x3c` | end of frame | end frame; skipped while `g_act_mission+0x78` > 0 |

### 3b. The cheat check

When `state_recheck` is non-zero the loop reads the active screen flags (`+0xb8`) and the screen
object at index `flags & 0xf` (`+0xbc`) and passes the object to `game_cheat_check`, which
accumulates keypresses and fires the dev hotkeys. On a match it forces `mission_state = 0` — the
mission keeps running — then takes both per-action value tables with
`input_get_action_values(&t, 0)` and `input_get_action_values(&t, 1)` and zeroes
`(g_input_bindings_end − g_input_bindings_begin) / 0xc` entries in each, so a cheat's keystrokes
cannot leave an action stuck on. Cheat vocabulary: `basic/artifacts/cheats.md`.

### 3c. Loop locals

| name | type | meaning |
|---|---|---|
| `delta_ms` | `undefined4` | elapsed milliseconds for this frame, preserved in EBX from the `+0x2c` begin-frame call |
| `mission_state` | `char` | the first `g_engine+0x98` result — the mission running/ended state |
| `state_recheck` | `char` | the second `g_engine+0x98` result, after the device pump and `input_update`; it gates the cheat check |
| `screen_flags` | `uint` | flags word from `g_engine+0xb8`; the low nibble indexes the active screen |
| `screen_obj` | `undefined4` | screen object from `g_engine+0xbc`, passed to `game_cheat_check` |
| `tmp` | `int` | scratch — device count, vtable base, menu result, display state |
| `layer0_ptr` / `layer1_ptr` | `int` | the input manager's two per-action value tables, the sampled one and the one-shot one, from `input_get_action_values`; `structs/input.md` |
| `esc_menu_obj` | `void *` | the `0x48`-byte ESC/pause menu allocation; the compiler reuses the slot as an SEH scope marker (value `0x3f`) |
| `display_ptr` | `undefined *` | local copy of `g_act_mission` for the paired `+0x70` / `+0x74` write |

## 4. Mission end and the ESC menu

Three `mission_state` values leave the running case: `0x01` (ended normally), `0x39` (`'9'`) and
`0x1c`.

`recorder_is_sequence_running()` gates all three. It is true while `C_recorder+0xe4` is a mode other
than 0 or 4, and during the five-frame startup hold after a `.rep` is loaded — i.e. while a recorded
sequence is playing or recording. Every one of its consumers refuses an action while it is true.

**`mission_state == 0x01`.** With no sequence running, **or** with `g_in_cutscene == 0`, the ESC
menu opens: `game_pause_all_sounds()` → `operator_new(0x48)` + `gui_game_menu_ctor(0, 0)` →
`gui_menu_execute_modal(1)` → `game_play_all_sounds(choice != 0x21)`.

| menu choice | action |
|---|---|
| `0x22` | `game_update_controls()` |
| `0x21` | quit: with no sequence running, return from `engine_game_loop`; with one running, `recorder_stop()` + `recorder_end_sequence_and_restore_scene()`, then return |
| `0x13` | `loadsave_savegame_load()` |
| anything else | fall through into the rest of the frame |

With a sequence running **and** `g_in_cutscene != 0` the menu is suppressed and only
`recorder_stop()` runs, so the cutscene finishes.

**`mission_state == 0x39` or `0x1c`** take that same `recorder_stop()` shortcut, and only when a
sequence is running.

## 5. Display state and the frame-skip counter

`*(*(g_act_mission + 0x24) + 0x48)` is read once per frame, right after the ESC-menu block:

| value | effect |
|---|---|
| `0` | continue normally |
| `1` | return from `engine_game_loop` immediately |
| `3` | zero `g_act_mission+0x70` and `+0x74`, call `loadsave_savegame_load()`, continue |

`g_act_mission+0x78` is the frame-skip counter. `recorder_stop` raises it to `4` and
`recorder_end_sequence_and_restore_scene` to `2`, each only when the counter is currently lower.
While it is above 0 the `g_engine+0x3c` end-frame call is skipped and the counter is decremented
instead, which gives rendering a few frames to settle after a state change.

## 6. Globals the loop touches

| global | set by | cleared by | effect |
|---|---|---|---|
| `g_in_cutscene` | `engine_main`, around a cutscene `engine_game_loop` | `engine_main`, right after it | suppresses the ESC menu while a sequence runs |
| `g_cache_base_block_loaded` | the `cache.bin` loader | - | gates `cache_base_block_tick` |
| `g_fade_active` | fade start | `screen_fade_tick`, at either end of the lerp | gates `screen_fade_tick` |
| `g_reload_pending` | a queued mission change | `engine_game_loop`, before `engine_new_mission` | one-shot mission reload |
| `g_recorder_startup_hold_frames` | a `.rep` load | after five frames | keeps `recorder_is_sequence_running` true |

`cache_base_block_tick` retires one streaming instance per frame through a round-robin cursor and,
every 100 ms or whenever the reference frame has moved more than 100 units, re-evaluates every
block against the mission scene's `+0x17c` reference frame. `structs/cache_base_block.md`.

## 7. When the player gains control

**Engine code does not grant player control.** The loop runs identically every frame whether or not
Tommy is controllable: `+0x2c` reads input every frame and `input_update` resolves the bindings
regardless. Whether the result reaches Tommy is decided by the mission's script, which executes
inside the `+0xcc` world update. The script's command set and its dispatcher are documented in
`basic/artifacts/script-opcode-dispatch.md` and `script-mapping/artifacts/opcode-table.md`.

## 8. Per-frame actor update and player interaction

Every live actor (the on-foot character class, vtable `0x0063c908`, ctor `actor_ctor` at
`0x004c6b90`) is ticked through its virtual update at vtable slot 13 (`+0x34`) = `player_ai`
(`0x004cb1b0`). It is reached only by indirect dispatch, `(*(*actor + 0x34))(actor, dt)`, so it has
no textual caller.

Per frame, `player_ai`:

- decays a fade/hint timer (`actor+0xadc` / `+0xae0`);
- lerps an animation-blend weight (`actor+0xab8` toward `+0xabc`);
- decays the accumulated shot spread `actor+0xad0` toward 0 and pushes `spread * 3 + 1` to
  `hud_set_player_accuracy` — the aim crosshair, gated by `hud_state+0x90a4` bit `0x20`
  (`basic/artifacts/hud_elements.md`). `player_add_shot_accuracy_penalty` (`0x004ccd20`, called from
  `human_shooting`) raises the same `+0xad0` on every shot the player fires;
- when the actor is on foot (`actor+0x98 == 0`), calls `player_do_action_button_on_foot`
  (`0x004cbd00`), the context-interaction handler. On a use/interact press it gathers the nearby
  options with `using_object_find_near_objects`; more than one available builds a modal choose-action
  menu (`gui_action_menu_ctor` + `gui_menu_execute_modal(1)`, the same modal driver the ESC menu
  uses, not a `hud_state` element), then runs the picked action through a switch keyed on the action
  code, which is the textdb string id in the 2000..2017 range. Full write-up:
  `basic/artifacts/player-interactions.md`.

## 9. Per-frame ambient pedestrian tick

An ambient pedestrian is a `C_traffic_element` — a `0x200`-byte record with no vtable and no entity
kind, held in the `std::vector<C_traffic_element*>` at `g_pedestrian_list` (`0x006560c4`). Each one
is driven by `pedestrian_tick` (`0x004b7520`):

- **Movement.** `pedestrian_advance_to_next_node` (`0x004b60b0`) pops the next node off the
  pedestrian's path and copies it into the target slot, choosing a neighbour by random wander
  (`+0x19e == 0`, avoiding the previous node `+0x7c`) or by minimising `S_vector::AngleTo` toward a
  target. It caches the current node `+0x60`, the previous `+0x7c`, the node position
  `+0x64`/`+0x68`/`+0x6c` and the per-node speed `+0x70` = `(node+0x10 & 0xffff) * 0.01`, then
  advances the world position along the jittered heading.
  When the path is exhausted and the pedestrian is a witness it writes the 3000 ms report delay,
  calls `police_manager_spawn_patrol_car_reinforcements_at_radius` with `100.0`, prints console
  string `0x836` (textdb 2102, the speeding line) and plays the report bark.
  `police-system/artifacts/offences.md` §5.4.
- **Ambient voice.** `pedestrian_manager_play_pedestrian_scream` (`0x004b58c0`) plays one
  `tables\voices.def` entry as a positional, distance-attenuated one-shot WAV. The table
  (`g_game_voice_table`, `0xa4` bytes = 41 dwords per row) is indexed `row = line_id - 2`,
  `col = the speaker's voice bank`; a slot value of 0 means no sound. It is gated on
  `g_pedestrian_scream_enabled`, on `g_sound_mgr`, on the per-frame budget
  `g_traffic_scream_budget < 0x65` (`+500` per play) and on a 2-D listener-distance cull against the
  camera frame at `*(g_act_mission+0x10) + 0x17c`. Ten callers, five of them inside
  `pedestrian_tick`.
- **Recognition greeting.** `pedestrian_greet_player` (`0x004be0d0`) runs behind the same ~1.5 s
  countdown. It fires when the pedestrian shares the player's model id, the player is not under
  pursuit and has no officers assigned, and the player is on foot; it sets the greeted flag `+0x1fd`,
  plays a greeting line (voices.def rows 168-173) and head-tracks the player through the frame
  vtable `+0x38` on the `neck` bone.
- **Interaction reply.** `pedestrian_on_player_talk` (`0x004be050`) is the interaction-button
  counterpart, reached from the player-command dispatcher case `0x7de`. Its friendly branch is
  unlocked by `+0x1fd`; without it only the brush-off branch fires.
  `basic/artifacts/npc-player-interactions.md`.

## 10. The crowd's damage path

`pedestrian_hit` (`C_traffic_element::Hit`, `0x004bd0d0`) is the ambient pedestrian's damage entry
point. It bails when the pedestrian is already dead, hands a **type-4 (gang)** pedestrian
(`+0x15a`) to `pedestrian_spawn_gang_on_hit` and returns, and otherwise subtracts the damage from
the health float at `+0x15c` — skipping the subtraction for hit type 5 unless the attacker is a car
the player is driving. Below `0.5` it stamps `-1.0` = dead, detaches the pedestrian from its
waypath and runs the death reaction. A car killing a pedestrian is what drives
`police_manager_increase_run_over_count_for_target`.

| source | route | what reaches `pedestrian_hit` |
|---|---|---|
| a burning area | `fire_tick` → `pedestrian_manager_fire` | every live pedestrian inside the squared radius takes `(3, dir, 5, 0, source)`; the sweep refuses to run while `g_traffic_all_visible` is clear |
| an explosion | `game_new_explosion` → `pedestrian_manager_explosion` | signature `(position, radius, force)`; each pedestrian in range passes a horizontal line-of-sight test (`collision_test_line_horizontal_static`) before taking `(2, dir, 5, 0, force·distance)` |
| a collision | `pedestrian_crowd_collision` | resolves the struck pedestrian through `pedestrian_manager_get_pedestrian_from_frame`, then `(elem, 5, vector, 5, actor, abs(v) · 4.2)` |

**`pedestrian_spawn_gang_on_hit`** (`C_traffic_element::CreateMafianos`, `0x004be390`) promotes the
`0x200`-byte crowd record into a real armed actor:

1. `g_i3d_driver+0x4c` yields a new scene frame; `pedestrian_set_alpha` runs on it.
2. `game_del_frame_for_shadow` / `game_add_frame_for_shadow` move the shadow registration from the
   pedestrian's frame to the new one.
3. The pedestrian's world position (`+0x10`/`+0x14`/`+0x18`) is copied into the frame
   (`+0x80`/`+0x84`/`+0x88`).
4. The animation set comes from the gender byte `+0x158`: `1` selects set 2, anything else set 3.
5. `operator_new(0x128c)` + `entity_ctor` build the actor. Measured stamps: `+0x680` =
   `rand_range() + 0xc` (the idle animation), `+0x5fc` = `0x10` (the human type), `+0x604` = `200.0`,
   `+0x61c` = `1.1`, `+0x620` = `1.0`, `+0x63c` = `1.0`.
6. `game_add_temporary_actor` adds it to the scene, `inventory_insert_item` arms it with weapon id
   `0xa` and `0x32` rounds, `human_change_weapon_model` puts the model in its hands.
7. The actor is linked back to the source record through the `+0x6f4` pair, the record is marked
   `+0x1a5 = 1` and `pedestrian_deactivate` retires it.
8. `police_manager_npc_list_push` registers the new actor with the police manager, and the actor is
   ordered to move away from the hit direction.

Class layout and the full method table: `structs/pedestrian.md`.

## 11. Per-frame vehicle tick & how cars move

A vehicle is a C++ object built on an LS3DF `I3D_frame` scene node, with multiple inheritance — its
constructor `car_ctor` installs two vtable pointers:

| object offset | vtable VA | role |
|---|---|---|
| `+0x00` | `0x0063bc08` | primary game-class vtable, 41 slots |
| `+0x70` | `0x0063bcac` | the `C_Vehicle` second base; its one slot is an adjustor to `vehicle_open_sound` (`0x0053b850`) |

The subclass `C_car_ex` uses `car_extended_ctor`, which installs the derived pair
`0x0063bd68` / `0x0063bd64` and the flavour tag `+0x14 = 3`. Both tables are **41 slots,
slot-for-slot aligned with no shift**, and exactly **8** slots differ: 14, 16, 18, 19, 20, 21, 25,
26. `structs/car_extended.md`.

### 11a. The per-frame virtual cluster (primary vtable `0x0063bc08`)

There is no single "Update": the vehicle is ticked through a cluster of virtuals dispatched
polymorphically from the per-frame world update (`g_engine+0xcc`, and the `mission_tick` →
`game_tick` subsystem passes). None of them has a static caller — they are reached only by vtable
dispatch, which is why a text search of the decomp for a caller dead-ends. The cluster is the
vtable order read out of `.rdata`.

| slot | offset | function | role |
|---|---|---|---|
| 0 | `+0x00` | `car_track_record_key` | record a position/quaternion/wheel keyframe, gated to 50 ms |
| 2 | `+0x08` | `car_track_apply_pose_at_clock` | playback: lerp position, Slerp quaternion between two keyframes, write the transform, derive velocity |
| 3 | `+0x0c` | `car_track_apply_pose_at_time` | playback over a streamed keyframe list |
| 4 | `+0x10` | `car_track_get_pos_at_time` | sample the recorded motion vector at a time — read-only query |
| 13 | `+0x34` | `car_ai_tick` | engine audio (RPM to pitch and volume), flasher, horn, and the sidewalk-offence check |
| 14 | `+0x38` | `car_update` | camera-distance LOD/cull flags (`+0xd54`), path-node lifecycle (`+0x210c`), `I3D_frame::UpdateWMatrixProc`, motion cues off the `+0x1f7c` velocity |
| 14 | `+0x38` | `car_extended_update` | the `C_car_ex` override of slot 14 — five blocks, of which the burnt-corpse spawn (`ohorelec01.i3d`) is only the first; `structs/car_extended.md` §3 |

### 11b. Two ways a car's transform is set

1. **Record / playback.** Slots 0, 2, 3 and 4 record and replay transform keyframes against the
   playback clock `DAT_00656488`. On this path the world transform is *interpolated* from keyframes
   and the velocity `+0x1f7c` is *derived* as `(pos_now − pos_prev) / dt` — the reverse of
   integration. This is how scripted and replayed vehicles move.
2. **Live driving physics.** Throttle and steering become forces, velocity and position in a
   separate subsystem: the wheel-contact / suspension solver over the vehicle's ground-contact list
   (`+0xd24`, count `+0x5b0`; the `vehicle_wheel_contact_*` helpers and the `LAB_005341b4`
   ground update). It runs per frame but is not one of the class vtable slots above. `car_update`
   only READS the resulting velocity to drive LOD and audio cues; it never writes the transform.

`police-system/artifacts/police-system.md` covers how vehicle motion that strikes a cop or a cop car
turns into police offences.

## 12. Related

`structs/game.md` (`C_game`, the running flag and the display word) · `structs/mission.md`
(`C_mission`, `g_act_mission`) · `structs/recorder.md` (the `.rep` sequence machinery) ·
`structs/input.md` (the two action-value tables) · `structs/cache_base_block.md` ·
`structs/pedestrian.md` · `structs/car.md` · `structs/car_extended.md` · `structs/app_bootstrap.md`
(the bootstrap and the registry blob) · `basic/artifacts/vehicle-movement.md` ·
`basic/artifacts/hud_elements.md` · `basic/artifacts/cheats.md`.

## 13. Open items

| item | state |
|---|---|
| what releases player control at mission start | the loop reads input every frame and the script decides; which script construct flips it has not been located in the dispatcher |
| other writers of `g_act_mission+0x78` | besides the two recorder paths, `.text` holds writes of `1` and `2` to the counter from functions not enumerated here |
| `game_update_controls` on ESC menu choice `0x22` | called, but its body has not been read |
| is `pedestrian_greet_player`'s gate satisfiable in shipped play | the greeting is not observed in game; the gate's terms are measured but the reachability question belongs to `basic/artifacts/npc-player-interactions.md` |
| how a civilian model acquires pedestrian type `4` | `civil05` has been seen spawning armed mafiosi, which needs `+0x15a == 4` at `pedestrian_hit`; nothing measured here shows how a civilian record gets that value. The `back1` frame-name tag that `pedestrian_create_model` parses — a 7-character name, uppercased, `atoi` of the characters from index 3 — sets the model scale at `+0x154`, not the type |
