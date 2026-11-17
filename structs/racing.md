# The racing classes — `C_racing` and the objects it owns

**Created** 2026-08-16 07:38 +0200 · **Modified** 2026-11-09 15:36 +0100

One translation unit holds **six classes** plus the free comparators; all six are **non-polymorphic on both builds** (`__vt__` count 0
for every one). A second unit holds the optimal-line segments (§12). Subsystem write-up, with what shipped, what was cut, the file
formats and the mod: `basic/artifacts/racing.md`. The entity class that goes with these: `structs/racecam.md`.

**Naming.** `racing_` / `circuit_` / `champ_circuit_` / `championship_` / `opponent_` / `best_time_` map the PS2 class tags
`8C_racing`, `9C_circuit`, `15C_champ_circuit`, `14C_championship`, `10C_opponent`, `11C_best_time` onto lowercase subsystem prefixes.
Free functions of the unit take `race_`. `racing_optimal_segment_` extends `racing_`, so anything that selects `C_racing`'s methods by
`startswith("racing_")` pulls `racing_optimal_segment_ctor` in unless the longer prefix is excluded.

| | |
|---|---|
| PC prefix | `racing_optimal_segment_`, `champ_circuit_`, `championship_`, `best_time_`, `opponent_`, `circuit_`, `racing_`, `race_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `C_racing` `0x428` (`race_mission6_init`, `net_host_session_start`); `C_opponent` `0x198`; `C_circuit` `0xdc`; `C_championship` `0xe0`; `C_champ_circuit` `0x74`; `C_racing_gate` `0x2c`; `C_best_time` `0x28`; `C_racing_optimal_segment` `0xa8` |
| PC constructor | `racing_ctor`, `circuit_ctor`, `championship_ctor`, `champ_circuit_ctor`, `opponent_ctor`, `opponent_ctor_with_id`, `best_time_ctor`, `racing_optimal_segment_ctor` |
| PC translation unit | `[0x0060b740, 0x00617a60)`, guard to guard, 131 starts, all named; the segment unit `[0x005c9720, 0x005ca290)` |
| PS2 class | `C_racing` (tag `8C_racing`), 63 methods; `C_opponent` (`10C_opponent`), 30; `C_circuit` (`9C_circuit`), 14; `C_championship` (`14C_championship`), 11; `C_champ_circuit` (`15C_champ_circuit`), 13; `C_best_time` (`11C_best_time`), 1; `C_racing_osegment` (`17C_racing_osegment`), 1 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Each record holds every offset its class's code accesses; the five with no recovered meaning are listed in Open item 1. The vectors are MSVC `std::vector`:
an allocator byte, then begin, end and capacity.

**Why these counts.** `entrants` and `racers` hold 20 pointers because `C_championship` has 21 standings slots, 20 entrants plus the
player; `net_session_add_ai_racer` refuses at `0x15`. `segment_banks`, `class_time_min` and `class_time_max` hold 7 because
`racing_get_segment_id_for_car` returns a car performance class 0..6. `winners_actors` and `winners_models` hold 6: the top three of
the winners' scene, one car and one driver each (`racing_spawn_winners_scene`, frames `FMVZavod0%i` and `FMVcar0%i`).
`final_positions` has one slot per standings row. `pts` and `speed` of a segment hold 10 points per stretch of racing line.

### 1a. `C_racing`

```c
typedef struct C_racing {  // the singleton, g_racing at 0x006f958c
    /* +0x00 */ uint8_t unread_00; // [U] racing_game_init stores 0; no reader (class census)
    /* +0x01 */ uint8_t race_active; // [H] 1 for the life of the race scene; gates the net handler's racer events
    /* +0x02 */ uint8_t results_visible; // [H] racing_toggle_results_visibility flips it, racing_get_results_visibility reads it
    /* +0x03 */ uint8_t mission6_race; // [H] 1 from race_mission6_init: the mission-6 race rules (§2a)
    /* +0x04 */ uint8_t rerun_race; // [H] race_run reloads the circuit and races again while it is set
    /* +0x08 */ uint8_t circuits_alloc; // [H] allocator byte of the circuit vector
    /* +0x0c */ C_circuit ** circuits_begin; // [H] one per track; 11 ship
    /* +0x10 */ C_circuit ** circuits_end; // [H] one past the last circuit
    /* +0x14 */ C_circuit ** circuits_cap; // [H] end of the vector's storage
    /* +0x18 */ uint8_t championships_alloc; // [H] allocator byte of the championship vector
    /* +0x1c */ C_championship ** championships_begin; // [H] one per championship; 7 ship
    /* +0x20 */ C_championship ** championships_end; // [H] one past the last championship
    /* +0x24 */ C_championship ** championships_cap; // [H] end of the vector's storage
    /* +0x28 */ uint32_t menu_mode; // [H] race_mission6_init writes 2; distinct from race_mode
    /* +0x2c */ uint32_t race_mode; // [H] 1 championship, 2 single race, 3 the third mode mission 6 uses
    /* +0x30 */ C_circuit * current_circuit; // [H] the track being raced
    /* +0x34 */ C_champ_circuit * current_round; // [H] the round being raced
    /* +0x38 */ C_championship * current_championship; // [H] written by racing_set_current_champ
    /* +0x3c */ uint32_t difficulty; // [H] 0 to 4, copied from the championship's +0xd0
    /* +0x40 */ C_opponent player_entrant; // [H] the player's entrant, embedded (§1b)
    /* +0x1d8 */ C_opponent * entrants[20]; // [H] the AI entrants; see the array note below
    /* +0x228 */ C_opponent * racers[20]; // [H] the live racers, in standings order
    /* +0x278 */ uint32_t unread_278; // [U] racing_ctor stores 0; no reader (.text scan)
    /* +0x27c */ uint32_t total_laps; // [H] the race's total lap count; ctor 3
    /* +0x280 */ uint32_t saved_total_laps; // [H] total_laps saved by race_menu_run, restored by racing_end_race
    /* +0x284 */ uint32_t entrant_count; // [H] excludes the player's entrant
    /* +0x288 */ uint32_t racer_count; // [H] live racers; not interchangeable with entrant_count
    /* +0x28c */ uint32_t standings_count; // [H] rows of racers the results tables walk
    /* +0x290 */ uint32_t race_time_ms; // [H] elapsed race time
    /* +0x294 */ uint32_t pause_stamp; // [H] set by racing_pause_clock_begin, read by racing_pause_clock_end
    /* +0x298 */ uint32_t race_start_stamp; // [H] timeGetTime when the countdown ends
    /* +0x29c */ uint32_t tick_accum_ms; // [H] gate-check accumulator
    /* +0x2a0 */ uint32_t tick_period_ms; // [H] ctor 200; 1000 once a race runs
    /* +0x2a4 */ uint32_t phase; // [H] 0 countdown, 1 racing, 2 post-race delay, 3 cooldown
    /* +0x2a8 */ uint32_t countdown_step; // [H] 0 to 3 on the HUD; the race starts at 4
    /* +0x2ac */ uint32_t wrong_way_stamp; // [H] stamped when the player drives away from the gate; HUD bit 0x800000 after 5001 ms
    /* +0x2b0 */ uint32_t bonus_car; // [H] the car a won championship awards, unlocked on CONTINUE
    /* +0x2b4 */ uint8_t mirror_track; // [H] the mirrored track: the %s mirror scene and mrsegm%i.sav
    /* +0x2b5 */ uint8_t replay_playback; // [H] 1 while a replay plays; race_run picks the replay mode from it
    /* +0x2b6 */ uint8_t segment_learn; // [H] gates the commit in racing_finish_segment_and_advance
    /* +0x2b7 */ uint8_t damage_enabled; // [H] ctor 1; racing_set_damage_enabled
    /* +0x2b8 */ uint32_t race_camera_racer; // [H] racing_set_race_camera_index clamps into it
    /* +0x2bc */ uint32_t race_camera_timer; // [H] the cycle timer
    /* +0x2c0 */ uint32_t race_camera_dwell; // [H] 10000 ms for a running racer, 1000 ms for a retired one
    /* +0x2c4 */ S_segment_bank segment_banks[7]; // [H] stride 0x10; one per car performance class (§12a)
    /* +0x334 */ C_racing_optimal_segment segment_scratch; // [H] the record being recorded (§1i)
    /* +0x3dc */ uint32_t segment_index; // [H] the recorder's current segment
    /* +0x3e0 */ uint32_t point_cursor; // [H] the recorder's point write cursor
    /* +0x3e4 */ S_vector last_sample_pos; // [H] the last sampled car position
    /* +0x3f0 */ void * winners_actors[6]; // [H] the winners' scene actors
    /* +0x408 */ I3D_model * winners_models[6]; // [H] their models
    /* +0x420 */ uint32_t power_accum_ms; // [H] driver-power update accumulator
    /* +0x424 */ uint32_t power_period_ms; // [H] ctor 200; 1000 once a race runs
} C_racing;
```

### 1b. `C_opponent`

```c
typedef struct C_opponent {  // 0x198 bytes
    /* +0x00 */ char name[0x20]; // [H] the driver's display name; opponent_ctor copies Tommy Angelo
    /* +0x20 */ uint32_t driver_model; // [H] ctor default 0x3a; racing_add_racer_for_player copies S_net_peer.driver_model here
    /* +0x24 */ int32_t entrant_id; // [H] -1 unassigned; indexes the opponent name and model roster
    /* +0x28 */ C_car * car; // [H] the racer's car
    /* +0x2c */ C_player * player; // [H] set only for the player's entrant
    /* +0x30 */ I3D_model * car_model; // [H] released by opponent_game_done
    /* +0x34 */ I3D_model * driver_model_rig; // [H] released by opponent_game_done
    /* +0x38 */ int32_t qualify_time; // [H] -1 no time
    /* +0x3c */ uint32_t lap_start_ms; // [H] the race time at the line; lap time = now - this
    /* +0x40 */ uint32_t unused_40; // [U] no access; no reader (class census)
    /* +0x44 */ uint8_t place_settled; // [H] retired once; opponent_set_death tests it
    /* +0x45 */ uint8_t start_crossing; // [H] while set, a finish crossing adds no lap
    /* +0x46 */ uint8_t timed_lap; // [H] while set, a finish crossing runs racing_test_best_lap
    /* +0x47 */ uint8_t rebuild_path; // [H] set => opponent_tick_overtake rebuilds the driver path, then clears it
    /* +0x48 */ C_driver * driver; // [H] opponent_tick deletes it when the car dies
    /* +0x4c */ float zone_t; // [H] 0 at the previous gate's line, 1 at the current gate's
    /* +0x50 */ void * session_player; // [H] a human player's session record; PC only
    /* +0x54 */ void * session_ai_slot; // [H] an AI opponent slot; PC only
    /* +0x58 */ int32_t unread_58; // [U] the ctors and opponent_game_init store -1; no reader (class census)
    /* +0x5c */ C_racing_gate * current_gate; // [H] the gate the racer is heading for
    /* +0x60 */ uint8_t is_player; // [H] 1 on the player's entrant: opponent_game_init builds a C_player
    /* +0x64 */ uint32_t position; // [H] 1-based race position after each sort
    /* +0x68 */ uint32_t finish_place; // [H] written by opponent_set_death
    /* +0x6c */ uint32_t lap; // [H] the lap counter
    /* +0x70 */ C_best_time * new_lap_record; // [H] the record this racer added to the best-lap table
    /* +0x74 */ C_best_time * new_race_record; // [H] the record this racer added to the best-race table
    /* +0x78 */ uint32_t best_lap_ms; // [H] the racer's own best lap
    /* +0x7c */ uint32_t lap_time; // [H] current lap or finish time; zeroed by opponent_set_death
    /* +0x80 */ uint32_t retired; // [H] both comparators sort on it
    /* +0x84 */ uint32_t stuck_ms; // [H] accumulated while car_is_usable(2) is false
    /* +0x88 */ uint32_t recover_timer; // [H] both constructors seed 0
    /* +0x8c */ uint32_t recover_period; // [H] 200, then 1000
    /* +0x90 */ uint8_t recover_step; // [H] shown on the HUD start flag for the player's entrant; step 4 restores collisions
    /* +0x94 */ S_car_node overtake_node_a; // [H] the first overtaking path node, 0x70 bytes
    /* +0x104 */ S_car_node overtake_node_b; // [H] the second overtaking path node, 0x70 bytes
    /* +0x174 */ C_car * overtake_target; // [H] the car being overtaken
    /* +0x178 */ uint32_t overtake_dir_bit; // [H] the current overtaking direction bit (§2b)
    /* +0x17c */ uint32_t overtake_reject_mask; // [H] the accumulated rejection mask, the same bits
    /* +0x180 */ uint8_t overtake_advance_count; // [H] counts to 2 during an overtake
    /* +0x181 */ uint8_t overtake_skip_count; // [H] path nodes to search back on ending
    /* +0x184 */ uint32_t overtake_rejoin_index; // [H] the route node after the inserted pair
    /* +0x188 */ uint32_t overtake_saved_index; // [H] the driver route index restored on ending
    /* +0x18c */ float overtake_start_dist_sq; // [M] squared distance to the target at the start; no reader (class census)
    /* +0x190 */ uint32_t overtake_scan_index; // [H] route cursor of the forward scan
    /* +0x194 */ I3D_frame * start_frame; // [H] the grid frame start%i
} C_opponent;
```

### 1c. `C_circuit`

```c
typedef struct C_circuit {  // 0xdc bytes, one per record of tables/circuits.def
    /* +0x00 */ char name[0x20]; // [H] the display name
    /* +0x20 */ char mission_name[0x20]; // [H] the mission folder: mission_open, the mirror scene, tree.klz
    /* +0x40 */ uint8_t gates_alloc; // [H] allocator byte of the gate vector
    /* +0x44 */ C_racing_gate ** gates_begin; // [H] element 0 is the start and finish line
    /* +0x48 */ C_racing_gate ** gates_end; // [H] one past the last gate
    /* +0x4c */ C_racing_gate ** gates_cap; // [H] end of the vector's storage
    /* +0x50 */ uint32_t lock_state; // [H] written by champ_circuit_set_lock_state: the track-unlock mechanism
    /* +0x54 */ uint8_t best_times_alloc; // [H] allocator byte of the best-lap vector
    /* +0x58 */ C_best_time ** best_times_begin; // [H] the best-lap table, cap 12
    /* +0x5c */ C_best_time ** best_times_end; // [H] one past the last record
    /* +0x60 */ C_best_time ** best_times_cap; // [H] end of the vector's storage
    /* +0x64 */ uint32_t unused_64; // [U] no access; no reader (class census)
    /* +0x68 */ uint32_t lap_count; // [H] from record +0x40 of circuits.def
    /* +0x6c */ uint32_t class_time_min[7]; // [H] qualifying time per car class, MIN
    /* +0x8c */ uint32_t class_time_max[7]; // [H] qualifying time per car class, MAX
    /* +0xac */ uint8_t left_line_alloc; // [H] allocator byte of the left racing line
    /* +0xb0 */ C_web_node ** left_line_begin; // [H] the left racing line, web-path type 2
    /* +0xb4 */ C_web_node ** left_line_end; // [H] one past the last node
    /* +0xb8 */ C_web_node ** left_line_cap; // [H] end of the vector's storage
    /* +0xbc */ uint8_t right_line_alloc; // [H] allocator byte of the right racing line
    /* +0xc0 */ C_web_node ** right_line_begin; // [H] the right racing line, web-path type 3
    /* +0xc4 */ C_web_node ** right_line_end; // [H] one past the last node
    /* +0xc8 */ C_web_node ** right_line_cap; // [H] end of the vector's storage
    /* +0xcc */ uint8_t race_cams_alloc; // [H] allocator byte of the race-camera vector
    /* +0xd0 */ race_cam ** race_cams_begin; // [H] the camera nodes
    /* +0xd4 */ race_cam ** race_cams_end; // [H] one past the last camera
    /* +0xd8 */ race_cam ** race_cams_cap; // [H] end of the vector's storage
} C_circuit;
```

### 1d. `C_racing_gate`

```c
typedef struct C_racing_gate {  // 0x2c bytes, linked into a ring
    /* +0x00 */ C_web_node *track_node;   // [H] one end of the gate line
    /* +0x04 */ C_web_node *outer_marker; // [H] the node with one connection, the other end
    /* +0x08 */ S_vector midpoint;        // [H] the midpoint of the two nodes
    /* +0x14 */ C_racing_gate *next;      // [H] ring link
    /* +0x18 */ C_racing_gate *previous;  // [H] ring link
    /* +0x1c */ uint32_t number;          // [H] the index in the gate vector
    /* +0x20 */ uint8_t  prints_time;     // [H] prints the checkpoint time
    /* +0x24 */ uint32_t split_time;      // [H] the stored split time
    /* +0x28 */ float    to_next;         // [H] the distance to the next gate's midpoint
} C_racing_gate;
```

### 1e. `C_championship`

```c
typedef struct C_championship {  // 0xe0 bytes, one per record of tables/championship.def
    /* +0x00 */ S_standing standings[21]; // [H] {C_opponent *, points}, stride 8
    /* +0xa8 */ uint32_t index; // [H] the constructor's argument; no reader (class census)
    /* +0xac */ uint8_t rounds_alloc; // [H] allocator byte of the round vector
    /* +0xb0 */ C_champ_circuit ** rounds_begin; // [H] the rounds
    /* +0xb4 */ C_champ_circuit ** rounds_end; // [H] one past the last round
    /* +0xb8 */ C_champ_circuit ** rounds_cap; // [H] end of the vector's storage
    /* +0xbc */ uint8_t roster_alloc; // [H] allocator byte of the roster
    /* +0xc0 */ void ** roster_begin; // [H] the opponent roster, {u32, u32} records
    /* +0xc4 */ void ** roster_end; // [H] one past the last record
    /* +0xc8 */ void ** roster_cap; // [H] end of the vector's storage
    /* +0xcc */ uint32_t progress; // [H] 0 open, 1 shown, 2 locked
    /* +0xd0 */ uint32_t difficulty; // [H] ctor 1; racing_set_current_champ copies it to C_racing +0x3c
    /* +0xd4 */ uint32_t opponent_count; // [H] from record +0x04 of championship.def
    /* +0xd8 */ uint32_t car_mask; // [H] the cars allowed, record +0x0c
    /* +0xdc */ uint32_t bonus_car; // [H] the car awarded on winning, record +0x10
} C_championship;
```

### 1f. `C_champ_circuit`

```c
typedef struct C_champ_circuit {  // 0x74 bytes, one per round
    /* +0x00 */ C_circuit * circuit; // [H] the track; not owned, the circuits live in C_racing's vector
    /* +0x04 */ uint8_t positions_valid; // [H] the positions-valid flag
    /* +0x08 */ uint32_t lock_state; // [H] ctor 2; champ_circuit_set_lock_state mirrors it into the C_circuit
    /* +0x0c */ uint32_t final_positions[21]; // [H] the final positions
    /* +0x60 */ uint32_t lap_count; // [H] the round's lap count; ctor 3
    /* +0x64 */ uint8_t best_races_alloc; // [H] allocator byte of the best-race table
    /* +0x68 */ void ** best_races_begin; // [H] the best-race table, cap 10
    /* +0x6c */ void ** best_races_end; // [H] one past the last record
    /* +0x70 */ void ** best_races_cap; // [H] end of the vector's storage
} C_champ_circuit;
```

### 1g. `C_best_time`

```c
typedef struct C_best_time {  // 0x28 bytes
    /* +0x00 */ char     name[0x20];      // [H] the holder; default mafia test driver
    /* +0x20 */ uint32_t time;            // [H] the lap time
    /* +0x24 */ uint32_t car_id;          // [H] the car
} C_best_time;
```

### 1h. `race_cam`

```c
typedef struct race_cam {  // 12 bytes
    /* +0x00 */ C_web_node *node;         // [H] the camera node
    /* +0x04 */ C_racing_gate *gate_a;    // [H] the first of its two gates
    /* +0x08 */ C_racing_gate *gate_b;    // [H] the second of its two gates
} race_cam;
```

### 1i. `C_racing_optimal_segment`

```c
typedef struct C_racing_optimal_segment {  // 0xa8 bytes
    /* +0x00 */ S_vector pts[10];         // [H] gate-crossing points along the racing line
    /* +0x78 */ float    speed[10];       // [H] the car's speed at each point; seeded 1000.0
    /* +0xa0 */ uint32_t count;           // [H] points actually recorded
    /* +0xa4 */ int32_t  time_ms;         // [H] elapsed ms for this stretch; -1 never recorded
} C_racing_optimal_segment;
```

## 2. Field evidence

Offsets of an embedded record are in the record's own table. The cells come from a register-tag census of each class's functions, plus the outside users named in the old witness notes.

### 2a. `C_racing`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | unread_00 | U | `racing_game_init` | - | stores 0; no reader (class census, and no `g_racing + 0` load in the decomp); the `gif_vector_ctor` stores at `+0x00` are `segment_banks` elements (§3a) |
| +0x1 | race_active | H | `racing_game_done`, `racing_game_init`, `net_session_handle_event_msg` | `net_session_handle_event_msg` | `racing_game_init` 1, `racing_game_done` 0, the handler's setting arm 0 clears it; the handler applies its four racer-event arms only while it is set |
| +0x2 | results_visible | H | `racing_game_init`, `racing_toggle_results_visibility` | `racing_get_results_visibility`, `racing_toggle_results_visibility` | - |
| +0x3 | mission6_race | H | `racing_ctor`, `race_mission6_init` | `game_done`, `game_tick`, `gui_race_results_tick`, `race_game_loop`, `race_mission6_execute_loop`, `racing_generate_opponents`, `racing_init_racers`, `racing_tick` | ctor 0, `race_mission6_init` 1. While set: car `0x2b` for entrant 0, the grid by index, the player's qualifying time 5, the player's driver dropped at the start, no winners scene, RETRY RACE disabled, and `game_done` frees `g_racing` |
| +0x4 | rerun_race | H | `racing_ctor`, `racing_end_race`, `racing_game_init`, `race_run` | `race_game_loop`, `race_run` | 0 from the ctor and `racing_game_init`; `racing_end_race` sets 1 on RETRY RACE, SHOW REPLAY and a finished qualifying; `race_run` reloads the circuit and runs the race again while it is set |
| +0x8 | circuits_alloc | H | `racing_ctor` | `racing_dtor`, `racing_load_circuits` | - |
| +0xc | circuits_begin | H | `racing_ctor` | `racing_ctor`, `racing_dtor`, `racing_get_circuit`, `racing_get_current_circuit_id`, `racing_load_best_times`, `racing_save_best_times` | `racing_load_circuits` fills it, `racing_get_circuit` indexes it, `racing_dtor` frees it |
| +0x10 | circuits_end | H | `racing_ctor` | `racing_ctor`, `racing_get_circuit`, `racing_get_current_circuit_id`, `racing_load_best_times`, `racing_save_best_times` | - |
| +0x14 | circuits_cap | H | `racing_ctor` | - | - |
| +0x18 | championships_alloc | H | `racing_ctor` | - | - |
| +0x1c | championships_begin | H | `racing_ctor`, `racing_dtor` | `racing_ctor`, `racing_dtor`, `racing_end_race`, `racing_get_current_champ_id`, `racing_load_champs_profile`, `racing_save_champs_profile` | `racing_load_champs` fills it, `racing_dtor` frees it |
| +0x20 | championships_end | H | `racing_ctor`, `racing_dtor` | `racing_ctor`, `racing_dtor`, `racing_end_race`, `racing_get_current_champ_id`, `racing_load_champs_profile`, `racing_save_champs_profile` | - |
| +0x24 | championships_cap | H | `racing_ctor`, `racing_dtor` | - | - |
| +0x28 | menu_mode | H | `race_mission6_init` | `racing_can_run_race`, `racing_do_gates`, `racing_end_race`, `racing_tick` | distinguished by `racing_can_run_race`; `racing_tick` branches on race mode `== 2` to tick a single racer instead of all of them |
| +0x2c | race_mode | H | `race_menu_run`, `race_mission6_init`, `racing_ctor`, `racing_end_race`, `racing_load_replay_setup` | `racing_do_gates`, `racing_end_race`, `racing_init_racers`, `racing_save_replay_setup`, `racing_tick` | - |
| +0x30 | current_circuit | H | `racing_ctor`, `racing_set_current_champ_circuit`, `racing_set_current_circuit` | `racing_activate_racers`, `racing_can_run_race`, `racing_create_segments`, `racing_do_gates`, `racing_game_done`, `racing_game_init`, `racing_generate_opponents`, `racing_generate_qualification`, `racing_get_current_circuit`, `racing_get_current_circuit_id`, `racing_get_distance_by_gates`, `racing_init_racers`, `racing_load_segments`, `racing_record_segment_points`, `racing_save_segments`, `racing_test_all_gates`, `racing_test_best_lap` | `racing_get_current_circuit` returns it; `racing_do_gates` and `racing_get_distance_by_gates` index its gate vector through it |
| +0x34 | current_round | H | `racing_ctor`, `racing_set_current_champ_circuit` | `racing_can_run_race`, `racing_end_race`, `racing_fill_champ_results`, `racing_is_last_champ_circuit`, `racing_test_best_race` | `racing_fill_champ_results` stops summing points at it; `racing_tick` reads its `+0x04`; the championship launch and results screens list its `+0x68` best-race table (`structs/racing_menus.md` §9) |
| +0x38 | current_championship | H | `racing_ctor`, `racing_set_current_champ` | `racing_can_run_race`, `racing_end_race`, `racing_fill_champ_results`, `racing_get_current_champ_id`, `racing_is_last_champ_circuit`, `racing_set_current_champ` | written by `racing_set_current_champ`; read by `racing_fill_champ_results` and `race_compare_champ_points` |
| +0x3c | difficulty | H | `racing_ctor`, `racing_set_current_champ` | `racing_get_difficulty_scale`, `racing_get_power_coeff_by_difficulty` | the switch selector in `racing_get_power_coeff_by_difficulty`; copied from championship `+0xd0` |
| +0x40 | player_entrant | H | `racing_ctor` (by address, `opponent_ctor`), `racing_bind_local_player_racer`, `racing_end_race`, `racing_init_racers`, `racing_load_replay_setup`, `racing_set_current_champ`, `racing_set_current_champ_circuit`, `racing_set_current_circuit`, `racing_set_player_racer_model`, `racing_tick` | `racing_bind_local_player_racer`, `racing_ctor`, `racing_do_gates`, `racing_dtor`, `racing_end_race`, `racing_fill_champ_results`, `racing_game_done`, `racing_init_racers`, `racing_test_best_lap`, `racing_tick`, `racing_update_driver_power_coeff` | `racing_tick` ticks it alone in race mode 2; `racing_fill_champ_results` appends it as the last result row; `racing_update_driver_power_coeff` measures every AI racer's distance from it |
| +0x1d8 | entrants[20] | H | `racing_ctor` (0), `racing_add_opponent`, `racing_add_opponents`, `racing_add_racer_for_ai_slot`, `racing_add_racer_for_player`, `racing_clear_opponents`, `racing_generate_opponents`, `racing_load_replay_setup`, `racing_remove_opponent_at` | `racing_clear_opponents`, `racing_fill_champ_results`, `racing_fill_opponent_name_and_model`, `racing_find_opponent_by_ai_slot`, `racing_find_opponent_by_player`, `racing_game_done`, `racing_generate_qualification`, `racing_load_replay_setup`, `racing_remove_racer_for_ai_slot`, `racing_remove_racer_for_player`, `racing_save_replay_setup`, `racing_set_current_champ_circuit`, `racing_set_player_racer_model` | `racing_generate_qualification` walks it |
| +0x228 | racers[20] | H | `racing_ctor` (0), `racing_generate_qualification`, `racing_init_racers`, `racing_remove_opponent_at`; the reorders `opponent_set_death`, `racing_end_race`, `racing_place_cars`, `racing_tick` (by address, `crt_qsort`) | `racing_activate_racers`, `racing_ctor`, `racing_end_race`, `racing_fake_final_positions`, `racing_get_racer_array_ptr`, `racing_init_racers`, `racing_place_cars`, `racing_remove_opponent_at`, `racing_tick`, `racing_update_driver_power_coeff` | `racing_tick` and `opponent_set_death` walk it; the accessor is `racing_get_racer_array_ptr` (`0x005c8270`) |
| +0x278 | unread_278 | U | `racing_ctor` | - | stores 0; no reader (`.text scan`: of the 10 accesses at displacement `+0x278`, the only `C_racing` one is this store) |
| +0x27c | total_laps | H | `racing_ctor`, `racing_end_race`, `racing_load_replay_setup`, `racing_set_current_champ_circuit` | `racing_activate_racers`, `racing_do_gates`, `racing_save_replay_setup`, `racing_test_best_lap` | `racing_do_gates` pushes it to the HUD with the current lap |
| +0x280 | saved_total_laps | H | `race_menu_run`, `racing_ctor` | `racing_end_race` | ctor 3; `race_menu_run` saves `total_laps` before forcing 99 or 1; `racing_end_race` restores it |
| +0x284 | entrant_count | H | `racing_add_opponent`, `racing_add_opponents`, `racing_add_racer_for_ai_slot`, `racing_add_racer_for_player`, `racing_clear_opponents`, `racing_ctor`, `racing_generate_opponents`, `racing_load_replay_setup`, `racing_remove_opponent_at` | `racing_add_opponent`, `racing_add_opponents`, `racing_add_racer_for_ai_slot`, `racing_add_racer_for_player`, `racing_can_run_race`, `racing_end_race`, `racing_fill_champ_results`, `racing_fill_opponent_name_and_model`, `racing_find_opponent_by_ai_slot`, `racing_find_opponent_by_player`, `racing_game_done`, `racing_generate_qualification`, `racing_get_opponent_count`, `racing_get_replay_setup_size`, `racing_init_racers`, `racing_load_replay_setup`, `racing_remove_opponent_at`, `racing_remove_racer_for_ai_slot`, `racing_remove_racer_for_player`, `racing_save_replay_setup`, `racing_set_current_champ_circuit`, `racing_set_player_racer_model` | `racing_get_opponent_count` (`0x00610870`) reads it; it excludes the player entrant |
| +0x288 | racer_count | H | `racing_clear_opponents`, `racing_ctor`, `racing_init_racers`, `racing_remove_opponent_at` | `racing_activate_racers`, `racing_fake_final_positions`, `racing_get_racer_count`, `racing_init_racers`, `racing_place_cars`, `racing_predict_car_collision`, `racing_remove_opponent_at`, `racing_set_race_camera_index`, `racing_tick`, `racing_update_driver_power_coeff` | `racing_get_racer_count` (`0x005c8280`) reads it. Two adjacent counts, two accessors — `+0x284` and `+0x288` are not interchangeable |
| +0x28c | standings_count | H | `racing_clear_opponents`, `racing_ctor`, `racing_end_race`, `racing_init_racers`, `racing_remove_opponent_at` | `racing_end_race`, `racing_remove_opponent_at` | `racing_init_racers` sets it to 1, then to `+0x284` + 1 in race modes 2 and 3; `racing_end_race` sets it to `+0x284` + 1 before sorting `+0x228` by qualifying time; `racing_remove_opponent_at` decrements it with `+0x288`; read by `racing_next_finish_place`, both results screens, `camera_tick` and `net_session_handle_event_msg` |
| +0x290 | race_time_ms | H | `racing_ctor`, `racing_game_init`, `racing_tick` | `racing_activate_racers`, `racing_do_gates`, `racing_test_best_race`, `racing_tick` | `racing_tick` computes `now - +0x298` and feeds `hud_set_race_time_text` |
| +0x294 | pause_stamp | H | `racing_ctor`, `racing_pause_clock_begin`, `racing_pause_clock_end` | `racing_pause_clock_end` | the two `racing_pause_clock_*` helpers |
| +0x298 | race_start_stamp | H | `racing_pause_clock_end`, `racing_tick` | `racing_pause_clock_end`, `racing_tick` | `racing_tick` stamps it with `timeGetTime()` when the countdown ends |
| +0x29c | tick_accum_ms | H | `racing_ctor`, `racing_game_init`, `racing_reset_phase_timer`, `racing_tick` | `racing_tick` | `racing_tick`; the period is **1000 ms** in both states. Reset together by `racing_reset_phase_timer` (`0x005a14d0`), a `C_racing` setter emitted inside the `g_net_session` translation unit and called from the type-`0xe` arm of `net_session_handle_event_msg` right after `racing_set_phase(0)`. In race phase 1 `racing_do_gates` runs only when the accumulator passes the period, so every racer's gates are checked about once per second (§6) |
| +0x2a0 | tick_period_ms | H | `racing_ctor`, `racing_game_init`, `racing_reset_phase_timer`, `racing_tick` | `racing_tick` | - |
| +0x2a4 | phase | H | `racing_game_init`, `racing_set_phase`, `racing_tick` | `racing_tick` | `racing_tick`'s switch: 0 countdown / start lights, 1 racing, 2 post-race delay, 3 cooldown. 2 and 3 are the networked pre-start phases |
| +0x2a8 | countdown_step | H | `racing_game_init`, `racing_tick` | `racing_tick` | `racing_tick` feeds it to `hud_set_race_start_flag` as 0,1,2,3 and starts the race at 4 |
| +0x2ac | wrong_way_stamp | H | `racing_init_racers`, `racing_tick` | `racing_tick` | `racing_tick` stamps it with the race time when the car's velocity points away from the current gate's `midpoint`, and raises HUD bit `0x800000` after 5001 ms |
| +0x2b0 | bonus_car | H | `racing_end_race`, `racing_game_init` | `racing_end_race` | 0 from `racing_game_init`; `racing_end_race` copies the championship's `bonus_car` (`+0xdc`) on winning, and on CONTINUE (`0x33`) a value other than 0 and -1 is unlocked through `loadsave_race_cars_unlock_from_car` and shown on the bonus-car screen |
| +0x2b4 | mirror_track | H | `gui_race_setup_tick`, `racing_ctor` | `gui_race_setup_ctor`, `race_run`, `racing_load_segments`, `racing_save_segments` | the setup screen's Track Mode toggle; selects the `%s mirror` scene, `mirror tree.klz` and `mrsegm%i.sav` over `mrseg%i.sav` (§12c); also posted as type-`0x31` selector 4 (§10) |
| +0x2b5 | replay_playback | H | `race_menu_run`, `racing_ctor`, `racing_end_race` | `race_run` | 0 from the ctor and `racing_end_race`; 1 from `racing_end_race` on SHOW REPLAY and from `race_menu_run` for a replay load; `race_run` calls `loadsave_replay_set_mode(!flag, flag)` |
| +0x2b6 | segment_learn | H | `racing_ctor` | `racing_finish_segment_and_advance` | gates the commit in `racing_finish_segment_and_advance` (§12b); the only store is the ctor's 0 (`.text scan`, 2 accesses), so the commit branch never runs (Open item 3) |
| +0x2b7 | damage_enabled | H | `racing_ctor`, `racing_set_damage_enabled` | `gui_race_champ_launch_ctor`, `gui_race_launch_ctor`, `opponent_game_init`, `racing_activate_racers`, `racing_set_damage_enabled`, `racing_tick` | `racing_ctor` seeds it to 1 |
| +0x2b8 | race_camera_racer | H | `racing_set_race_camera_index` | `racing_set_race_camera_index` | `racing_set_race_camera_index` clamps into it. Not a lap count |
| +0x2bc | race_camera_timer | H | `race_game_loop`, `racing_ctor`, `racing_game_init`, `racing_set_race_camera_index`, `racing_tick` | `racing_tick` | 10000 ms for a running racer, 1000 ms for a retired one |
| +0x2c0 | race_camera_dwell | H | `race_game_loop`, `racing_ctor`, `racing_game_init`, `racing_set_race_camera_index` | `racing_tick` | - |
| +0x2c4 | segment_banks[7] | H | `racing_ctor` (by address, `crt_ehvec_ctor`), `racing_clear_segments`, `racing_create_segments`, `racing_game_done`, `racing_load_segments` | `racing_clear_segments`, `racing_create_segments`, `racing_ctor`, `racing_dtor`, `racing_game_done` | `racing_ctor` builds them with `crt_ehvec_ctor(…, 0x10, 7, …)`; one bank per car performance class (§12a) |
| +0x334 | segment_scratch | H | `racing_ctor` (by address, `racing_optimal_segment_ctor`), `racing_finish_segment_and_advance`, `racing_game_init`, `racing_record_segment_points` | `racing_ctor`, `racing_finish_segment_and_advance`, `racing_game_init` | byte-proven by `lea ecx,[esi+0x334]` in `racing_ctor` calling `racing_optimal_segment_ctor` |
| +0x3dc | segment_index | H | `racing_finish_segment_and_advance`, `racing_game_init` | `racing_finish_segment_and_advance`, `racing_record_segment_points` | `racing_record_segment_points` |
| +0x3e0 | point_cursor | H | `racing_finish_segment_and_advance`, `racing_game_init`, `racing_record_segment_points` | `racing_record_segment_points` | - |
| +0x3e4 | last_sample_pos | H | `racing_game_init`, `racing_tick` | `racing_tick` | - |
| +0x3f0 | winners_actors[6] | H | `racing_game_init` (0), `racing_release_winners_scene` (0), `racing_spawn_winners_scene` | `racing_release_winners_scene` | `racing_spawn_winners_scene` / `racing_release_winners_scene` |
| +0x408 | winners_models[6] | H | `racing_game_init` (0), `racing_release_winners_scene` (0), `racing_spawn_winners_scene` | `racing_game_init` | - |
| +0x420 | power_accum_ms | H | `racing_ctor`, `racing_game_init`, `racing_tick` | `racing_tick` | `racing_tick` drives `racing_update_driver_power_coeff` off it |
| +0x424 | power_period_ms | H | `racing_ctor`, `racing_game_init`, `racing_tick` | `racing_tick` | - |

### 2b. `C_opponent`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name[0x20] | H | `opponent_ctor`, `opponent_ctor_with_id` | `opponent_get_race_points` | - |
| +0x20 | driver_model | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_add_racer_for_player` | `opponent_game_init` | `0x3a` is that field's default (`net_session.md` §1d) |
| +0x24 | entrant_id | H | `opponent_ctor`, `opponent_ctor_with_id` | `opponent_create_driver_path`, `opponent_game_init` | `opponent_ctor_with_id` stores it; indexes the opponent name/model roster |
| +0x28 | car | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_done`, `opponent_game_init` | `opponent_create_driver`, `opponent_create_driver_path`, `opponent_do_recover`, `opponent_end_overtake`, `opponent_game_init`, `opponent_overtake`, `opponent_recover`, `opponent_test_node_vs_cars`, `opponent_tick`, `opponent_tick_overtake`, `opponent_tick_recover` | `opponent_tick`, `opponent_game_init`, `opponent_create_driver` |
| +0x2c | player | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_done`, `opponent_game_init` | `opponent_play_death` | `opponent_game_init` stores it, `opponent_game_done` clears it |
| +0x30 | car_model | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_done`, `opponent_game_init` | `opponent_game_done`, `opponent_game_init` | both released by `opponent_game_done` |
| +0x34 | driver_model_rig | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_done`, `opponent_game_init` | `opponent_game_done` | - |
| +0x38 | qualify_time | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_end_race`, `racing_generate_qualification`, `racing_init_racers` | `race_compare_qualification_time` | the ctors write 0; `racing_generate_qualification` writes it, `racing_init_racers` the grid index in the mission-6 race, `race_compare_qualification_time` reads it |
| +0x3c | lap_start_ms | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init`, `racing_do_gates`, `racing_init_racers` | `racing_activate_racers`, `racing_do_gates`, `racing_test_best_lap`, `racing_tick` | `racing_init_racers` writes 0, `racing_do_gates` the race time at the line and reads it for the split; the lap time is now minus this; `racing_tick` and `racing_activate_racers` show the player's on the HUD through `C_racing +0x7c` |
| +0x40 | unused_40 | U | - | - | no access; no reader (class census) |
| +0x44 | place_settled | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_finish`, `opponent_set_death` | `opponent_recover`, `opponent_set_death` | `opponent_set_death` |
| +0x45 | start_crossing | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_do_gates`, `racing_init_racers` | `opponent_tick_overtake`, `race_compare_position`, `racing_do_gates` | set by both constructors and `racing_init_racers`; `racing_do_gates` clears it when gate 2 becomes current and on a backward finish crossing; `race_compare_position` ranks a racer heading for the finish gate with it clear ahead of one with it set |
| +0x46 | timed_lap | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init`, `racing_do_gates` | `racing_do_gates` | `racing_do_gates` sets it when gate 2 becomes current or on an untimed lap completion; while set, a finish crossing runs `racing_test_best_lap`; cleared by a backward finish crossing and by `opponent_game_init` |
| +0x47 | rebuild_path | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_tick_overtake`, `racing_do_gates` | `opponent_tick_overtake` | ctors 0; `racing_do_gates` sets 1 past gate 2 when the racer has a driver; `opponent_tick_overtake` then calls `opponent_create_driver_path` and clears it |
| +0x48 | driver | H | `opponent_create_driver`, `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_done`, `opponent_tick` | `opponent_create_driver`, `opponent_create_driver_path`, `opponent_end_overtake`, `opponent_game_done`, `opponent_game_init`, `opponent_get_next_driver_route`, `opponent_get_next_driver_route_from`, `opponent_get_prev_driver_route`, `opponent_get_prev_driver_route_from`, `opponent_overtake`, `opponent_test_node_vs_track_edges`, `opponent_tick`, `opponent_tick_overtake` | `opponent_tick` deletes it when the car dies |
| +0x4c | zone_t | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_test_gate_crossing` | `race_compare_position` | written by `racing_test_gate_crossing`; the final tie-break in `race_compare_position` |
| +0x50 | session_player | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_dtor`, `racing_add_racer_for_player` | `opponent_do_recover`, `opponent_game_init`, `race_compare_participant_slot`, `racing_activate_racers`, `racing_find_opponent_by_player` | **PC-only.** Written by `racing_add_racer_for_player`; read by `racing_activate_racers`, `racing_find_opponent_by_player` and `race_compare_participant_slot`, which orders on the id byte at `player+0x2c` (`structs/net_session.md`) |
| +0x54 | session_ai_slot | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_dtor`, `racing_add_racer_for_ai_slot` | `opponent_game_init`, `race_compare_participant_slot` | **PC-only.** Written by `racing_add_racer_for_ai_slot`; `race_compare_participant_slot` orders on the slot index at `slot+0x28` |
| +0x58 | unread_58 | U | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init` | - | -1 from all three; no reader (class census; a base trace of the `opponent_` functions finds no `C_opponent` load) |
| +0x5c | current_gate | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_do_gates`, `racing_init_racers` | `opponent_recover`, `opponent_tick_overtake`, `race_compare_position` | seeded with the finish gate by `racing_init_racers`, moved by `racing_do_gates`; `race_compare_position` reads its gate number `+0x1c` |
| +0x60 | is_player | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_init_racers` | `opponent_game_init` | ctors 0, `racing_init_racers` 1 on the embedded entrant; a byte read in `opponent_game_init` selects the `C_player` branch (`player_ctor`) over an AI driver |
| +0x64 | position | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_activate_racers`, `racing_end_race`, `racing_init_racers`, `racing_tick` | `racing_activate_racers`, `racing_do_gates`, `racing_tick` | the 1-based race position after each sort; the HUD shows the player's through `C_racing +0xa4`; `racing_do_gates` reads it as the finish place |
| +0x68 | finish_place | H | `opponent_finish`, `opponent_set_death` | `opponent_get_race_points` | `opponent_set_death` |
| +0x6c | lap | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_do_gates` | `race_compare_position` | `race_compare_position` |
| +0x70 | new_lap_record | H | `opponent_ctor`, `opponent_ctor_with_id`, `racing_init_racers`, `racing_test_best_lap` | `gui_race_results_ctor`, `racing_test_best_lap` | the `C_best_time` this racer added to the circuit's best-lap table, 0 for none; the results screen highlights it |
| +0x74 | new_race_record | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init`, `racing_init_racers`, `racing_test_best_race` | `gui_race_champ_results_ctor` | the same for the round's best-race table |
| +0x78 | best_lap_ms | H | `opponent_game_init`, `racing_test_best_lap` | `gui_race_champ_results_ctor`, `gui_race_results_ctor`, `racing_test_best_lap` | the racer's own best lap; `opponent_game_init` writes 0; the championship results show it in widget `0x69` |
| +0x7c | lap_time | H | `opponent_finish`, `opponent_game_init`, `opponent_set_death`, `racing_generate_qualification` | - | zeroed by `opponent_set_death` |
| +0x80 | retired | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init`, `opponent_set_death` | `opponent_get_race_points` | `opponent_set_death` writes it, both comparators sort on it |
| +0x84 | stuck_ms | H | `opponent_game_init`, `opponent_tick` | `opponent_tick` | `opponent_tick` accumulates it while `car_is_usable(2)` is false |
| +0x88 | recover_timer | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_do_recover`, `opponent_tick_recover` | `opponent_tick_recover` | both constructors seed `0` / `200`; `opponent_tick_recover` adds the frame time to `+0x88` and advances the step counter every `+0x8c` ms (200, then 1000) |
| +0x8c | recover_period | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_do_recover`, `opponent_tick_recover` | `opponent_tick_recover` | - |
| +0x90 | recover_step | H | `opponent_do_recover`, `opponent_game_init`, `opponent_tick_recover` | `opponent_recover`, `opponent_tick`, `opponent_tick_recover` | a byte: every store and load is one byte; 0..4 the steps, 5 idle. `opponent_tick_recover`: the player's entrant shows it on the HUD start flag; the car fades in from 0.2 opacity over the steps, and at step 4 it gets its dynamic collisions back and full opacity |
| +0x94 | overtake_node_a | H | `opponent_overtake` (by address, `opponent_fill_path_node`) | `opponent_overtake` | an `S_car_node` (`0x70` B, `structs/driver.md`); inserted into the driver path and tested by `opponent_test_node_vs_cars` and `opponent_test_node_vs_track_edges`; `+0x94..+0x9c` is its position |
| +0x104 | overtake_node_b | H | `opponent_overtake` (by address, `opponent_fill_path_node`) | `opponent_overtake` | the second node, used as the first; ends at `+0x173` |
| +0x174 | overtake_target | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_end_overtake`, `opponent_overtake`, `opponent_tick_overtake` | `opponent_overtake`, `opponent_test_node_vs_cars`, `opponent_tick_overtake` | the car being overtaken; `opponent_end_overtake` clears it |
| +0x178 | overtake_dir_bit | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_end_overtake`, `opponent_overtake` | `opponent_overtake`, `opponent_test_node_vs_cars`, `opponent_test_node_vs_track_edges`, `opponent_tick_overtake` | the bits: `0x1` / `0x20` ahead, `0x40` / `0x80` behind, `0x100` / `0x400` and `0x200` / `0x800` the two sides (which side is which is not asserted) |
| +0x17c | overtake_reject_mask | H | `opponent_overtake`, `opponent_test_node_vs_cars`, `opponent_test_node_vs_track_edges` | `opponent_overtake`, `opponent_test_node_vs_cars`, `opponent_test_node_vs_track_edges` | the bits of `+0x178`, accumulated over the rejected directions; `opponent_overtake` clears it |
| +0x180 | overtake_advance_count | H | `opponent_ctor`, `opponent_overtake` | `opponent_overtake`, `opponent_tick_overtake` | `opponent_overtake` resets it to 0; it counts to 2 |
| +0x181 | overtake_skip_count | H | `opponent_ctor`, `opponent_overtake`, `opponent_tick_overtake` | `opponent_end_overtake`, `opponent_overtake` | `opponent_overtake` increments it, `opponent_tick_overtake` clears it; `opponent_end_overtake` searches that many path nodes back |
| +0x184 | overtake_rejoin_index | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_overtake` | `opponent_overtake` | the route index + 2: the node after the inserted pair |
| +0x188 | overtake_saved_index | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_overtake`, `opponent_tick_overtake` | `opponent_end_overtake`, `opponent_overtake`, `opponent_tick_overtake` | the driver route index `opponent_end_overtake` restores |
| +0x18c | overtake_start_dist_sq | M | `opponent_tick_overtake` | - | the squared distance to the target when an overtake starts; no reader (class census); the name rests on the writer's formula |
| +0x190 | overtake_scan_index | H | `opponent_overtake`, `opponent_tick_overtake` | `opponent_overtake` | the route cursor of the forward scan |
| +0x194 | start_frame | H | `opponent_ctor`, `opponent_ctor_with_id`, `opponent_game_init` | `opponent_create_driver_path`, `opponent_game_init` | `opponent_game_init` resolves it by name and copies its transform onto the car |

### 2c. `C_circuit`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name[0x20] | H | `circuit_ctor` | `gui_mp_lobby_ctor`, `gui_race_champ_launch_ctor`, `gui_race_launch_ctor`, `gui_race_setup_ctor`, `gui_race_setup_on_close_child` | the display name, record `+0x00` of `circuits.def`; the screens pass the record's address as a label's text |
| +0x20 | mission_name[0x20] | H | `circuit_ctor` | `race_run` | record `+0x20`: the mission folder for `mission_open`, the `%s mirror` scene and the `tree.klz` paths |
| +0x40 | gates_alloc | H | `circuit_ctor` | `circuit_dtor`, `circuit_load_gates_from_scene` | `circuit_get_first_gate` returns element 0 = the start/finish line |
| +0x44 | gates_begin | H | `circuit_ctor` | `circuit_clear_gates`, `circuit_determine_race_cams`, `circuit_dtor`, `circuit_get_first_gate`, `circuit_load_gates_from_scene` | - |
| +0x48 | gates_end | H | `circuit_clear_gates`, `circuit_ctor` | `circuit_clear_gates`, `circuit_determine_race_cams`, `circuit_get_first_gate`, `circuit_load_gates_from_scene` | - |
| +0x4c | gates_cap | H | `circuit_ctor` | - | - |
| +0x50 | lock_state | H | `champ_circuit_load_profile`, `champ_circuit_set_lock_state`, `circuit_ctor` | `gui_race_circuit_select_ctor` | ctor 2; written by `champ_circuit_set_lock_state` — this is the track-unlock mechanism; `!= 2` makes the circuit selectable |
| +0x54 | best_times_alloc | H | `circuit_ctor` | `circuit_dtor` | Serialised by `circuit_save_best_times` |
| +0x58 | best_times_begin | H | `circuit_ctor`, `circuit_load_best_times` | `circuit_dtor`, `circuit_save_best_times` | - |
| +0x5c | best_times_end | H | `circuit_ctor`, `circuit_load_best_times` | `circuit_save_best_times` | - |
| +0x60 | best_times_cap | H | `circuit_ctor`, `circuit_load_best_times` | `circuit_load_best_times` | - |
| +0x64 | unused_64 | U | - | - | no access; no reader (class census) |
| +0x68 | lap_count | H | `racing_load_circuits` | `racing_game_init` | copied from record `+0x40` by `racing_load_circuits`; 7 for every shipped circuit except `trat12`/`trat13` (5) in the mod pack |
| +0x6c | class_time_min[7] | H | `racing_load_circuits` | `circuit_get_class_qualify_time_min` | from record `+0x48`; read by `circuit_get_class_qualify_time_min`. Order proven by the lerp `t*(max-min)+min` in `racing_generate_qualification` |
| +0x8c | class_time_max[7] | H | `racing_load_circuits` | `circuit_ctor`, `circuit_get_class_qualify_time_max` | from record `+0x68`; read by `circuit_get_class_qualify_time_max` |
| +0xac | left_line_alloc | H | `circuit_ctor` | - | loaded as web-path type 2; the side is pinned in §13a |
| +0xb0 | left_line_begin | H | `circuit_ctor`, `circuit_dtor` | `circuit_clear_paths`, `circuit_dtor` | - |
| +0xb4 | left_line_end | H | `circuit_clear_paths`, `circuit_ctor`, `circuit_dtor` | `circuit_clear_paths` | - |
| +0xb8 | left_line_cap | H | `circuit_ctor`, `circuit_dtor` | - | - |
| +0xbc | right_line_alloc | H | `circuit_ctor` | - | web-path type 3; the side is pinned in §13a |
| +0xc0 | right_line_begin | H | `circuit_ctor`, `circuit_dtor` | `circuit_clear_paths`, `circuit_dtor` | - |
| +0xc4 | right_line_end | H | `circuit_clear_paths`, `circuit_ctor`, `circuit_dtor` | `circuit_clear_paths` | - |
| +0xc8 | right_line_cap | H | `circuit_ctor`, `circuit_dtor` | - | - |
| +0xcc | race_cams_alloc | H | `circuit_ctor` | - | 12-byte records `{C_web_node* node, C_racing_gate* a, C_racing_gate* b}` |
| +0xd0 | race_cams_begin | H | `circuit_ctor`, `circuit_dtor` | `circuit_clear_race_cams`, `circuit_determine_race_cams`, `circuit_dtor`, `circuit_load_gates_from_scene` | - |
| +0xd4 | race_cams_end | H | `circuit_clear_race_cams`, `circuit_ctor`, `circuit_dtor` | `circuit_clear_race_cams`, `circuit_determine_race_cams` | - |
| +0xd8 | race_cams_cap | H | `circuit_ctor`, `circuit_dtor` | - | - |

### 2d. `C_racing_gate`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | track_node | H | `circuit_load_gates_from_scene` | `racing_test_gate_crossing` | - |
| +0x4 | outer_marker | H | `circuit_load_gates_from_scene` | `racing_test_gate_crossing` | - |
| +0x8 | midpoint | H | `circuit_load_gates_from_scene` | `racing_tick` | `racing_tick` tests the car's velocity against the direction to it for the wrong-way stamp (`C_racing +0x2ac`) |
| +0x14 | next | H | `circuit_load_gates_from_scene` | `racing_test_gate` | - |
| +0x18 | previous | H | `circuit_load_gates_from_scene` | `racing_test_gate`, `racing_test_gate_crossing` | - |
| +0x1c | number | H | `circuit_load_gates_from_scene` | `race_compare_position`, `racing_get_distance_by_gates` | - |
| +0x20 | prints_time | H | `circuit_load_gates_from_scene` | `racing_do_gates` | the outer marker's connection bit `0x80` |
| +0x24 | split_time | H | `racing_do_gates` | `racing_do_gates` | cleared on every gate at each forward step, so the compare-with-split branch never runs |
| +0x28 | to_next | H | `circuit_load_gates_from_scene` | `racing_get_distance_by_gates` | - |

### 2e. `C_championship`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | standings[21] | H | `racing_fill_champ_results` | `championship_get_results` | 20 entrants plus the player |
| +0xa8 | index | H | `championship_ctor` | - | no reader (class census, Open item 2) |
| +0xac | rounds_alloc | H | `championship_ctor` | `championship_add_circuit`, `championship_dtor` | - |
| +0xb0 | rounds_begin | H | `championship_ctor` | `championship_dtor`, `championship_load_profile`, `championship_save_profile` | - |
| +0xb4 | rounds_end | H | `championship_ctor` | `championship_add_circuit`, `championship_load_profile`, `championship_save_profile` | - |
| +0xb8 | rounds_cap | H | `championship_ctor` | - | - |
| +0xbc | roster_alloc | H | `championship_ctor` | `championship_set_opponents` | - |
| +0xc0 | roster_begin | H | `championship_ctor`, `championship_dtor` | `championship_dtor` | - |
| +0xc4 | roster_end | H | `championship_ctor`, `championship_dtor` | `championship_dtor` | - |
| +0xc8 | roster_cap | H | `championship_ctor`, `championship_dtor` | - | - |
| +0xcc | progress | H | `championship_ctor`, `championship_load_profile` (by address, `dtaRead`), `racing_ctor`, `racing_end_race` | `championship_save_profile`, `gui_race_championship_ctor`, `race_run` | 0 open, 1 shown, 2 locked: ctor 2, `racing_ctor` 1 for the first championship, `racing_end_race` 0 or 1 |
| +0xd0 | difficulty | H | `championship_ctor` | `racing_set_current_champ` | - |
| +0xd4 | opponent_count | H | `racing_load_champs` | `championship_load_profile`, `championship_save_profile`, `championship_set_opponents`, `gui_race_champ_launch_ctor`, `racing_set_current_champ` | record `+0x04` of `championship.def` |
| +0xd8 | car_mask | H | `racing_load_champs` | `gui_race_car_maker_ctor`, `gui_race_car_model_ctor` | record `+0x0c` |
| +0xdc | bonus_car | H | `racing_load_champs` | `racing_end_race` | record `+0x10`; copied to `C_racing +0x2b0` on winning |

### 2f. `C_champ_circuit`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | circuit | H | `champ_circuit_ctor` | `champ_circuit_load_profile`, `champ_circuit_set_lock_state` | - |
| +0x4 | positions_valid | H | `champ_circuit_ctor`, `champ_circuit_final_positions_set`, `racing_set_current_champ_circuit` | `racing_tick` | a byte: every store and load is one byte |
| +0x8 | lock_state | H | `champ_circuit_ctor`, `champ_circuit_set_lock_state` | `champ_circuit_load_profile`, `champ_circuit_save_profile` | - |
| +0xc | final_positions[21] | H | `champ_circuit_ctor` (0), `champ_circuit_final_positions_set`, `champ_circuit_load_profile` (by address, `dtaRead`), `racing_set_current_champ_circuit` (0 in a session) | `champ_circuit_ctor`, `champ_circuit_load_profile` | - |
| +0x60 | lap_count | H | `champ_circuit_ctor`, `racing_load_champs` | `racing_set_current_champ_circuit` | ctor 3; `racing_load_champs` copies record `+0x40 + round * 4` |
| +0x64 | best_races_alloc | H | `champ_circuit_ctor` | - | - |
| +0x68 | best_races_begin | H | `champ_circuit_ctor`, `champ_circuit_dtor` | `champ_circuit_dtor` | - |
| +0x6c | best_races_end | H | `champ_circuit_ctor`, `champ_circuit_dtor` | `champ_circuit_dtor` | - |
| +0x70 | best_races_cap | H | `champ_circuit_ctor`, `champ_circuit_dtor` | - | - |

### 2g. `C_best_time`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `best_time_ctor`, `circuit_load_best_times` | `circuit_save_best_times` | - |
| +0x20 | time | H | `best_time_ctor`, `circuit_load_best_times` | `circuit_save_best_times` | - |
| +0x24 | car_id | H | `best_time_ctor`, `circuit_load_best_times` | `circuit_save_best_times` | - |

### 2h. `race_cam`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | node | H | `circuit_determine_race_cams` | `camera_tick` | - |
| +0x4 | gate_a | H | `circuit_determine_race_cams` | `camera_tick` | - |
| +0x8 | gate_b | H | `circuit_determine_race_cams` | `camera_tick` | - |

### 2i. `C_racing_optimal_segment`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | pts | H | `racing_record_segment_points`, `racing_finish_segment_and_advance`, `racing_load_segments` | `opponent_create_driver_path`, `racing_save_segments` | - |
| +0x78 | speed | H | `racing_optimal_segment_ctor`, `racing_record_segment_points`, `racing_load_segments` | `opponent_create_driver_path`, `racing_save_segments` | - |
| +0xa0 | count | H | `racing_optimal_segment_ctor`, `racing_finish_segment_and_advance`, `racing_game_init`, `racing_load_segments` | `racing_record_segment_points`, `racing_save_segments` | - |
| +0xa4 | time_ms | H | `racing_optimal_segment_ctor`, `racing_finish_segment_and_advance`, `racing_game_init`, `racing_load_segments` | `racing_finish_segment_and_advance`, `racing_save_segments` | - |

## 3. Method family

Every PC start of both units has a row; the per-class counts are in §11a. Role cells are taken from each function's docblock.

### 3a. Construction & destruction

#### `C_racing`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060f910 | `racing_ctor` | instance | `__ct__8C_racingFv` | 0x0039a2e0 | CONFIRMED | initialises the circuit vector (`+0x0c`) and the championship vector (`+0x1c`), seeds the two `{0, 200}` timer pairs and the 7-slot round array at `+0x2c4` (`crt_ehvec_ctor`, stride 0x10), then loads |
| 0x0060fb50 | `racing_dtor` | instance | `__dt__8C_racingFv` | 0x0039a6f0 | CONFIRMED | destroys and frees every `C_circuit` in the vector at `+0x0c`, then every `C_championship` in the vector at `+0x1c`, then clears the opponent roster, tears down the 7-slot round array at `+0x2c4` |

#### `C_racing_optimal_segment`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005c9750 | `racing_optimal_segment_ctor` | instance | `__ct__17C_racing_osegmentFv` | 0x001db9d0 | CONFIRMED | zeroes the point count at `+0xa0` and stamps the segment time at `+0xa4` with -1 (= not yet recorded) |

#### `C_circuit`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060b7a0 | `circuit_ctor` | instance | `__ct__9C_circuitFPcPcPUiPUi` | 0x00394b40 | CONFIRMED | one per record of `tables/circuits.def`: empties the vectors and copies the constructor's arguments |
| 0x0060b880 | `circuit_dtor` | instance | `__dt__9C_circuitFv` | 0x00394d00 | CONFIRMED | deletes every gate, then every best-time record, then releases the storage of all five vectors (gates, best times, the two racing lines, the race cams) |

#### `C_championship`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060c640 | `championship_ctor` | instance | `__ct__14C_championshipFUi` | 0x00395f60 | CONFIRMED | initialises the rounds vector at `+0xb0` and the roster vector at `+0xc0`, stores the record id at `+0xa8` (and its low byte at `+0xac`/`+0xbc`), and seeds the two state dwords at `+0xcc`/`+0xd0` |
| 0x0060c6a0 | `championship_dtor` | instance | `__dt__14C_championshipFv` | 0x00395fe0 | CONFIRMED | destroys every `C_champ_circuit` in the rounds vector at `+0xb0` and frees it, then frees the 8-byte opponent pairs in the roster vector at `+0xc0`, then releases both vectors' storage |

#### `C_champ_circuit`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060c3a0 | `champ_circuit_ctor` | instance | `__ct__15C_champ_circuitFP9C_circuitUi` | 0x003959f0 | CONFIRMED | binds the round to a `C_circuit` and records the owning championship's id |
| 0x0060c3e0 | `champ_circuit_dtor` | instance | `__dt__15C_champ_circuitFv` | 0x00395a90 | STRUCTURAL | releases the round's owned storage |

#### `C_opponent`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060cda0 | `opponent_ctor` | instance | `__ct__10C_opponentFv` | 0x003964c0 | CONFIRMED | the default constructor: seeds `+0x88 = 0`, `+0x8c = 200`, the entrant id at `+0x24` to `-1`, copies the default driver name **"Tommy Angelo"** into the `char[0x20]` at `+0x00`, sets `+0x20 = 0x3a` |
| 0x0060ce50 | `opponent_ctor_with_id` | instance | `__ct__10C_opponentFUi` | 0x003965e0 | CONFIRMED | the id-taking overload: stores the id at `+0x24` and copies a different default name |
| 0x0060cf00 | `opponent_dtor` | instance | `__dt__10C_opponentFv` | 0x003966e0 | STRUCTURAL | clears the two identity-record pointers at `+0x50` (car colour) and `+0x54` (player model) |

#### `C_best_time`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0060b770 | `best_time_ctor` | instance | `__ct__11C_best_timeFv` | 0x00394b00 | CONFIRMED | copies the default holder name `"mafia test driver"` into the record's 0x20-byte name field and seeds `+0x20` with `100` and `+0x24` with `0` |

#### the units' CRT pairs and templates

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005c9720 | `crt_static_init_guard_005c9720` | the file scope | - | - | - | sets bit 0 of the module init flag at 0x006f9ae0 if it is clear, then TAIL-JUMPS to `crt_atexit_register_005c9740`, which registers this unit's static destructor |
| 0x005c9740 | `crt_atexit_register_005c9740` | the file scope | - | - | - | `push 0x0060b700 / call atexit / pop ecx / ret` |
| 0x0060b740 | `crt_static_init_guard_0060b740` | the file scope | - | - | - | sets bit 0 of the module init flag at 0x006f9ae0 if it is clear, then TAIL-JUMPS to `crt_atexit_register_0060b760`, which registers this unit's static destructor |
| 0x0060b760 | `crt_atexit_register_0060b760` | the file scope | - | - | - | `push 0x0060b700 / call atexit / pop ecx / ret` |
| 0x0060fb30 | `gif_vector_ctor` | the file scope | - | - | - | `this = +0x2c4` one `segment_banks` element per call from `crt_ehvec_ctor`; writes an uninitialised byte to `+0x00` and zeroes the three pointers at `+0x04`, `+0x08` and `+0x0c` |

### 3b. Accessors

#### `C_racing`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8270 | `racing_get_racer_array_ptr` | - | - | - | returns `this + 0x228`, the base of the 20-slot `C_opponent *` live-racer array whose count lives at `+0x288` |
| 0x005c8280 | `racing_get_racer_count` | - | - | - | returns `*(this + 0x288)`, the live-racer count |
| 0x00610260 | `racing_get_current_circuit` | `GetCurrentCircuit__8C_racingFv` | 0x0039ac90 | CONFIRMED | one-line accessor: `return this->+0x30` |
| 0x00610870 | `racing_get_opponent_count` | `GetOpponentCount__8C_racingFv` | 0x0039b3d0 | CONFIRMED | returns `+0x284` |
| 0x00614280 | `racing_set_phase` | - | - | - | writes `+0x2a4`, the state `racing_tick` switches on |

### 3c. Behaviour

#### `C_racing`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005a14d0 | `racing_reset_phase_timer` | - | - | - | zeroes the tick accumulator at `C_racing+0x29c` and stores the new period in `+0x2a0` |
| 0x005c9770 | `racing_record_segment_points` | - | - | - | per frame: for every circuit cross-line the movement crossed, stores the crossing point and the speed in the scratch record (§12b) |
| 0x005c99a0 | `racing_create_segments` | `CreateSegments__8C_racingFv` | 0x001dba10 | CONFIRMED | for each of the seven car-class banks at `C_racing+0x2c4` (stride 0x10), walks the circuit's left-path and right-path node vectors, takes the midpoint of each node pair as one segment point, seeds |
| 0x005c9c90 | `racing_save_segments` | - | - | - | opens `savegame\mrseg%i.sav` (or `mrsegm%i.sav` when `C_racing+0x2b4` is set) for the current circuit id and writes all seven banks: per bank a segment count, then per segment the point count, the |
| 0x005c9e30 | `racing_load_segments` | `LoadSegments__8C_racingFbPcUi` | 0x001dbe70 | CONFIRMED | opens `system\mrseg%i.sav` (or `mrsegm%i.sav` when `C_racing+0x2b4` is set) for the current circuit id and reads all seven banks, allocating a record per segment and reading count, time, points and |
| 0x005ca040 | `racing_clear_segments` | `ClearSegments__8C_racingFUi` | 0x001dc3b0 | CONFIRMED | walks each of the seven banks at `C_racing+0x2c4`, deletes every segment record with a bare `operator delete`, and resets the bank's end pointer to its begin pointer without freeing the bank's own |
| 0x005ca0a0 | `racing_get_segment_bank_for_car` | - | - | - | reads the packed car id the player picked (`C_racing+0x64`), asks the car database for that record's top speed and model index, and passes both to the free function `racing_get_segment_id_for_car`, |
| 0x005ca0f0 | `racing_finish_segment_and_advance` | - | - | - | resolves the player's bank, and if the just-recorded time (`+0x3d8`) is LOWER than the stored segment's time and the learn flag at `C_racing+0x2b6` is set, copies the 0xa8-byte scratch record over |
| 0x0060cc60 | `racing_fill_champ_results` | `FillChampResults__8C_racingFP14C_championship` | 0x00396330 | CONFIRMED | walks the entrant array at `+0x1d8` (count `+0x284`) and, for each entrant, writes a `{C_opponent*, points}` pair into the current championship's result table at `champ+0x00`, summing |
| 0x0060f6c0 | `racing_set_race_camera_index` | `SetRCamIndex__8C_racingFUi` | 0x00399e90 | CONFIRMED | clamps the index against the live racer count (`+0x288`), storing it at `+0x2b8` or falling back to 0; resets the cycle timer `+0x2bc` and sets the dwell period `+0x2c0` to 10000 ms for a running |
| 0x0060fc90 | `racing_load_circuits` | `LoadCircuits__8C_racingFv` | 0x0039a850 | CONFIRMED | reads the whole file, refuses it unless the length is a whole number of **0x88-byte** records, and for each record constructs a `C_circuit` (`operator_new(0xdc)`) from the two 0x20-byte name fields |
| 0x0060ff10 | `racing_load_champs` | `LoadChamps__8C_racingFv` | 0x0039a9d0 | CONFIRMED | reads the whole file through `dtaOpen`/`dtaRead`, refuses it unless the length is a whole number of **0xb8-byte** records, and for each record constructs a `C_championship` (`operator_new(0xe0)`), |
| 0x00610230 | `racing_get_circuit` | `GetCircuit__8C_racingFUi` | 0x0039ac50 | CONFIRMED | bounds-checked index into the circuit vector at `+0x0c` |
| 0x00610270 | `racing_set_current_champ` | `SetCurrentChamp__8C_racingFP14C_championship` | 0x0039aca0 | CONFIRMED | stores the championship in `+0x38` and copies its difficulty from `championship+0xd0` into `+0x3c` |
| 0x006102f0 | `racing_set_current_champ_circuit` | `SetCurrentChampCircuit__8C_racingFP15C_champ_circuit` | 0x0039ad30 | CONFIRMED | stores the round in `+0x34` and its circuit in `+0x30`, copies the round's lap count from `+0x60` into `+0x27c`, clears every entrant's place and the player's own place counter, and in a networked |
| 0x00610370 | `racing_add_opponents` | `AddOpponents__8C_racingFPUiUi` | 0x0039adc0 | CONFIRMED | resets the opponent count, then for every non-zero model id in the array constructs a `C_opponent`, gives it a random unused driver name and stores it in the next slot |
| 0x00610400 | `racing_add_racer_for_player` | - | - | - | destroys whatever occupies the next free slot, constructs a `C_opponent` with no model of its own, copies the player's typed name out of the record (`player+4`, 0x20 bytes) instead of drawing one |
| 0x00610480 | `racing_add_racer_for_ai_slot` | - | - | - | destroys whatever occupies the next free slot, constructs a `C_opponent` from the slot's car model (`slot+0x24`), and either applies the slot's stored driver-name index (`slot+0x20`) or draws a fresh |
| 0x00610510 | `racing_bind_local_player_racer` | - | - | - | stores the player record in `+0x90`, copies the player's typed name into the entrant's name field at `+0x40` and the record's car id into `+0x60` |
| 0x00610540 | `racing_add_opponent` | `AddOpponent__8C_racingFUiUi` | 0x0039aec0 | CONFIRMED | destroys whatever occupies the next free slot, constructs a `C_opponent` (0x198 bytes) around the model id, applies the requested name-table index, stores it at `+0x1d8 + count*4` and increments |
| 0x006105b0 | `racing_generate_opponents` | `GenerateOpponents__8C_racingFUi` | 0x0039af70 | CONFIRMED | loads the car-database detail tables, resolves the selected class `+0x64` to a model bitmask and collects every model in it — skipping model 0x2b when the single-model flag `+0x03` is set — then |
| 0x00610810 | `racing_clear_opponents` | `ClearOpponents__8C_racingFv` | 0x0039b350 | CONFIRMED | walks all **twenty** opponent slots at `+0x1d8`, destroys and frees each live entrant, and zeroes the three counts `+0x284`, `+0x288` and `+0x28c` |
| 0x00610880 | `racing_fill_opponent_name_and_model` | `FillOpponentNameModel__8C_racingFP10C_opponentUi` | 0x0039b6e0 | CONFIRMED | holds a table of fourteen driver names with a car model id for each (0x39..0x3d) |
| 0x00610a10 | `racing_place_cars` | `PlaceCars__8C_racingFv` | 0x0039ba00 | CONFIRMED | acquires the two shared animation sets, then runs `opponent_game_init` for each entrant |
| 0x00610c00 | `racing_init_racers` | `InitRacers__8C_racingFv` | 0x0039bf90 | CONFIRMED | starts the racer array at `+0x228` with the embedded player entrant at `+0x40`, counts of 1, and the player un-retired |
| 0x00610fc0 | `racing_load_champs_profile` | `LoadChampsProfile__8C_racingFv` | 0x0039c4a0 | CONFIRMED | opens `savegame\mr%03u.sav` for the current profile slot and streams each loaded championship through `championship_load_profile` |
| 0x00611050 | `racing_save_champs_profile` | `SaveChampsProfile__8C_racingFv` | 0x0039c600 | CONFIRMED | opens `savegame\mr%03u.sav` for the current profile slot and streams each loaded championship through `championship_save_profile` |
| 0x006110e0 | `racing_load_best_times` | `LoadBestTimes__8C_racingFb` | 0x0039cd20 | CONFIRMED | opens `savegame\mrtimes.sav` and streams each loaded circuit's table through `circuit_load_best_times`, stopping at the first failure |
| 0x00611140 | `racing_save_best_times` | `SaveBestTimes__8C_racingFb` | 0x0039cfa0 | CONFIRMED | opens `savegame\mrtimes.sav` for writing and streams each loaded circuit's table through `circuit_save_best_times`, stopping at the first failure |
| 0x006111a0 | `racing_can_run_race` | `CanRunRace__8C_racingFv` | 0x0039d140 | CONFIRMED | for a single race (`+0x28` == 2) it needs a car class chosen (`+0x64` != -1), a circuit (`+0x30`) and at least one opponent (`+0x284`); for a championship (`+0x28` == 1) it needs the car class, a |
| 0x006111e0 | `racing_game_init` | `GameInit__8C_racingFv` | 0x0039d1f0 | CONFIRMED | resets the per-race timers and flags, grows the web-path node pool to 15000, then loads the circuit's gates from the scene and its **left and right track paths as web-path types 2 and 3** into |
| 0x00611380 | `racing_game_done` | `GameDone__8C_racingFv` | 0x0039d3d0 | CONFIRMED | clears the recorded track trail, then the circuit's gates and race cameras, runs `opponent_game_done` for the player entrant and for every opponent, frees the **seven segment containers** at `+0x2c4` |
| 0x00611430 | `racing_activate_racers` | `ActivateRacers__8C_racingFv` | 0x0039d500 | CONFIRMED | the last step of `racing_game_init`, run ONCE per race |
| 0x006139c0 | `racing_tick` | `Tick__8C_racingFUi` | 0x003a0f90 | CONFIRMED | ticks every racer (`opponent_tick`) — one racer in mode 2, otherwise all `+0x288` of them — then runs the race state machine at `+0x2a4`: state 0  the START COUNTDOWN. Every 1000 ms it calls |
| 0x006141e0 | `racing_next_finish_place` | - | - | - | the place the next racer to cross the line will be given |
| 0x00614290 | `racing_show_time_banner` | `FlushText__8C_racingFPcUiUi` | 0x003a19b0 | CONFIRMED | renders `mm:ss:mmm` from a millisecond count, prefixes it with a textdb string and — when a prefix string is supplied — that too, then hands the result to the HUD banner with a 2.0 s hold |
| 0x00614340 | `racing_test_best_lap` | `TestBestLap__8C_racingFP10C_opponentUi` | 0x003a1b40 | CONFIRMED | computes the lap time as `now - opponent+0x3c`, keeps it in `opponent+0x78` if it is the racer's own best, and announces the last lap (textdb **800**) when the racer enters the final one |
| 0x00614660 | `racing_test_best_race` | `TestBestRace__8C_racingFP10C_opponent` | 0x003a1ed0 | CONFIRMED | walks the current `C_champ_circuit`'s best-time vector (`+0x68`/`+0x6c`), and if the racer's finishing time `+0x290` beats an existing entry — or the table is shorter than ten — inserts a new |
| 0x00614820 | `racing_end_race` | `EndRace__8C_racingFv` | 0x003a20e0 | CONFIRMED | switches the race camera back to the leader, rebuilds the player's AI driver, locks the player's controls and releases the car, clears `+0x2b5`, and saves the best-time file |
| 0x00614db0 | `racing_set_current_circuit` | `SetCurrentCircuit__8C_racingFP9C_circuit` | 0x003a2ae0 | CONFIRMED | clears the qualification marker at `+0x78` to -1 and stores the circuit pointer in `+0x30` |
| 0x00614dd0 | `racing_generate_qualification` | `GenerateQualification__8C_racingFv` | 0x003a2b20 | CONFIRMED | opens the car-detail database (`car_database_load_details`), walks the entrant array at `+0x1d8` (count `+0x284`), reads each racer's car class from the car-database entry's `+0x88` bitmask (first |
| 0x00614ee0 | `racing_get_difficulty_scale` | `GetDifficulty2__8C_racingFv` | 0x003a2cc0 | CONFIRMED | switches on the difficulty stored at `+0x3c` and returns 0.0, 0.2, 0.5, 0.8 or 1.0 |
| 0x00614f30 | `racing_get_power_coeff_by_difficulty` | `GetPowerCoeffByDifficulty__8C_racingFRfRf` | 0x003a2d40 | CONFIRMED | switches on the difficulty level at `+0x3c` and writes a pair of float multipliers to the two out-parameters: 0.9/0.9, 0.93/0.85, 1.0/1.0, 1.2/1.3, 1.3/1.4 for levels 0-4 |
| 0x00614fd0 | `racing_pause_clock_begin` | - | - | - | stores `timeGetTime()` into `+0x294`, to be consumed by `racing_pause_clock_end` |
| 0x00614ff0 | `racing_pause_clock_end` | - | - | - | computes `timeGetTime() - +0x294`, stores that duration back into `+0x294`, and **adds it to `+0x298`** — the race start stamp — so the elapsed race time `+0x290 = now - +0x298` never counts time |
| 0x00615020 | `racing_toggle_results_visibility` | `GMenuTogleResultsVisibility__8C_racingFv` | 0x003a2dc0 | CONFIRMED | flips `results_visible` |
| 0x00615030 | `racing_get_results_visibility` | `GMenuGetResultsVisibility__8C_racingFv` | 0x003a2de0 | CONFIRMED | returns `results_visible` |
| 0x006150a0 | `racing_get_replay_setup_size` | - | - | - | returns `0x2c + opponentCount * 0x24` — a four-dword header plus one 0x24-byte record per entrant |
| 0x006150c0 | `racing_save_replay_setup` | - | - | - | writes `+0x27c`, `+0x2c`, `+0x64` and `+0x60` as the first four dwords, then one 0x24-byte record per opponent carrying the car model (`opponent+0x24`) and the drawn name index (`opponent+0x20`) |
| 0x00615120 | `racing_load_replay_setup` | `LoadReplay__8C_racingFUiPv` | 0x003a2e80 | CONFIRMED | reads the four header dwords back into `+0x27c` (lap count), `+0x2c` (race mode), `+0x64` (car class) and `+0x60`; derives the entrant count from the blob length as `(len - 8) / 0x24 - 1`; clears the |
| 0x006151d0 | `racing_get_current_circuit_id` | `GetCurrentCircuitID__8C_racingFv` | 0x003a2fe0 | CONFIRMED | linear-scans the circuit vector at `+0x0c`/`+0x10` for the pointer held in `+0x30` and returns its index |
| 0x00615200 | `racing_get_current_champ_id` | `GetCurrentChampID__8C_racingFv` | 0x003a3030 | CONFIRMED | linear-scans the championship vector at `+0x1c`/`+0x20` for the pointer held in `+0x38` and returns its index |
| 0x00615230 | `racing_get_segment_id_for_car` | `GetSegmentIdFromCarIdG__FUi` | 0x003a3080 | CONFIRMED | four car models are special-cased outright — 0x2b→3, 0x2c→4, 0x2d→5, 0x3b→6 — and everything else is classified by top speed: below 140 → 0, below 170 → 1, otherwise 2 |
| 0x006152d0 | `racing_set_player_racer_model` | - | - | - | if the player is the local one (`+0x90` matches) it also updates the embedded player entrant's model at `+0x60`; then finds the racer whose `+0x50` is that player and writes the new model id into |
| 0x00615330 | `racing_remove_racer_for_player` | - | - | - | walks the opponent array looking for `opponent+0x50` equal to the given player record and calls `racing_remove_opponent_at` for each match |
| 0x00615370 | `racing_remove_racer_for_ai_slot` | - | - | - | walks the opponent array looking for `opponent+0x54` equal to the given AI slot record and calls `racing_remove_opponent_at` for each match |
| 0x006153b0 | `racing_remove_opponent_at` | - | - | - | swap-erases the opponent at the given index out of the array at `+0x1d8` and decrements `+0x284`; then finds the same object in the live racer array `+0x228`, swap-erases it there too, decrements |
| 0x006154c0 | `racing_find_opponent_by_player` | - | - | - | scans the opponent array `+0x1d8` (count `+0x284`) for the racer whose `+0x50` back-pointer is the given player record and returns it |
| 0x00615500 | `racing_find_opponent_by_ai_slot` | - | - | - | scans the opponent array `+0x1d8` (count `+0x284`) for the racer whose `+0x54` back-pointer is the given AI slot record and returns it |
| 0x00615710 | `racing_set_damage_enabled` | `SetDamage__8C_racingFb` | 0x003a3360 | CONFIRMED | writes the flag to `+0x2b7` — the byte `racing_activate_racers` reads to choose between the two HUD flag words, i.e |
| 0x00615810 | `racing_spawn_winners_scene` | - | - | - | runs two passes over places 1..3 |
| 0x00616070 | `racing_release_winners_scene` | - | - | - | walks the six-entry actor array at `+0x3f0`, and for each live entry invalidates the actor, purges it from the entity reference vectors, runs its vtable teardown and frees it; then releases the |
| 0x006160e0 | `racing_fake_final_positions` | `FakeFinalPositions__8C_racingFv` | 0x003a3370 | CONFIRMED | hands a finishing place to every racer that never crossed the line |
| 0x00616150 | `racing_test_gate` | `TestGate__8C_racingFP10C_opponent` | 0x003a3420 | CONFIRMED | takes the car's AI probe point (`car_get_ai_probe_point(1)`) and tests it with `racing_test_gate_crossing` against three gate zones in a FIXED order: the racer's current gate (`opponent+0x5c`), its |
| 0x00616200 | `racing_test_all_gates` | `TestAllGates__8C_racingFP10C_opponent` | 0x003a3550 | CONFIRMED | takes the car's AI probe point once, then sweeps the circuit's whole gate vector (`circuit+0x44`/`+0x48`) through `racing_test_gate_crossing`, returning the index of the first gate that reports a |
| 0x00616280 | `racing_test_gate_crossing` | `InternTestGate__8C_racingFP13C_racing_gateR8S_vectorRf` | 0x003a3600 | CONFIRMED | the geometric core of the lap gates, read from the disassembly (the body below loses the `vec3_set_xyz` results) |
| 0x00616510 | `racing_do_gates` | `DoGates__8C_racingFP10C_opponent` | 0x003a3910 | CONFIRMED | one gate check for one racer; `racing_tick` runs it every 1000 ms (accumulator `C_racing+0x29c` against the period `+0x2a0`) |
| 0x00616ce0 | `racing_is_last_champ_circuit` | `IsLastChampCircuit__8C_racingFv` | 0x003a3df0 | CONFIRMED | reads the current championship from `+0x38`, and compares the current `C_champ_circuit` at `+0x34` against the last element of the championship's round vector (`champ+0xb0`/`+0xb4`) |
| 0x00616d30 | `racing_update_driver_power_coeff` | `UpdateDriverPowerC__8C_racingFv` | 0x003a3e40 | CONFIRMED | walks the live racer array at `+0x228` (count `+0x288`); for each racer that has a `C_driver` at `+0x48` it fetches the difficulty's two power coefficients (`racing_get_power_coeff_by_difficulty`), |
| 0x00616e10 | `racing_get_distance_by_gates` | `GetDistanceByGates__8C_racingFP10C_opponentP10C_opponent` | 0x003a41e0 | CONFIRMED | with the current circuit taken from `+0x30`, compares the two racers' lap counters (`+0x6c`) and their current gate indices (`opponent+0x5c` -> `+0x1c`), then sums the per-gate lengths (`gate+0x28`) |
| 0x00616f20 | `racing_predict_car_collision` | `TestniPredvidaciKolize__8C_racingFP5C_carP8C_driver` | 0x003a4530 | CONFIRMED | clears the car's `+0x200c` predicted-contact slot, builds its swept curve with `racing_fill_driver_curve`, then walks the mission's live actor vector (`mission+0xf0`) for every other kind-4 actor |
| 0x00617430 | `racing_fill_driver_curve` | `FillDriverCurve__8C_racingFP5C_carP8C_driverPQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>bf` | 0x003a4f10 | CONFIRMED | zeroes the caller's curve container, then walks the driver's path-node vector (`C_driver+0x164`, stride 0x70) from the current index `+0x170`, emitting a sphere per node scaled by the car's speed |

#### `C_circuit`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0060ba00 | `circuit_load_gates_from_scene` | `LoadGatesFromScene__9C_circuitFv` | 0x00394e50 | CONFIRMED | seeds a graph walk at node id **`0x27c`**, and for every node it reaches: a node whose flag word has bit `0x8` becomes a **race camera** — a 12-byte `{node, gate_a, gate_b}` record appended to |
| 0x0060bda0 | `circuit_get_first_gate` | `GetFirstGate__9C_circuitFv` | 0x00395300 | CONFIRMED | returns the first element of the gate vector at `+0x40`, or 0 when there are no gates |
| 0x0060bdc0 | `circuit_clear_gates` | `ClearGates__9C_circuitFv` | 0x00395320 | CONFIRMED | frees each element of the gate vector at `+0x40` and resets its end pointer |
| 0x0060be10 | `circuit_clear_paths` | `ClearPaths__9C_circuitFv` | 0x00395380 | CONFIRMED | resets the end pointers of the two `C_web_node*` path vectors at `+0xac` and `+0xbc` |
| 0x0060be70 | `circuit_clear_race_cams` | `ClearRaceCams__9C_circuitFv` | 0x00395390 | CONFIRMED | frees each element of the race-cam vector at `+0xcc` and resets its end pointer |
| 0x0060bed0 | `circuit_load_path` | `LoadPaths__9C_circuitFUiPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>` | 0x003953f0 | CONFIRMED | resolves a start node by id, then follows the graph node-by-node — always taking the first neighbour that is not the node it just came from — appending each node to the supplied vector until the walk |
| 0x0060bfb0 | `circuit_determine_race_cams` | `DetermineRaceCams__9C_circuitFv` | 0x00395510 | CONFIRMED | for every race-camera node of web-path type 2, finds the gate whose node is the camera's first neighbour and the gate whose node is its fifth, orders the two by gate index and stores them in the |
| 0x0060c0a0 | `circuit_save_best_times` | `SaveBestTimes__9C_circuitFR8G_Stream` | 0x00395660 | CONFIRMED | writes the entry count, then per entry the 0x20-byte driver name, the value at `+0x24` and the value at `+0x20` |
| 0x0060c130 | `circuit_load_best_times` | `LoadBestTimesDta__9C_circuitFi` | 0x00395890 | CONFIRMED | reads the record count, then for each one allocates a 0x28-byte `C_best_time`, reads the driver name (0x20 bytes), the car id at `+0x24` and the time at `+0x20`, and appends it to the circuit's table |
| 0x0060c360 | `circuit_get_class_qualify_time_min` | - | - | - | returns `*(this + 0x6c + class*4)` for a class below 7, and -1 otherwise |
| 0x0060c380 | `circuit_get_class_qualify_time_max` | - | - | - | returns `*(this + 0x8c + class*4)` for a class below 7, and -1 otherwise |

#### `C_championship`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0060c7d0 | `championship_add_circuit` | `AddCircuit__14C_championshipFP15C_champ_circuit` | 0x003960e0 | STRUCTURAL | push_back of one `C_champ_circuit*` onto the vector at `+0xb0` |
| 0x0060c980 | `championship_load_profile` | `LoadProfile__14C_championshipFR8G_Stream` | 0x00396130 | CONFIRMED | reads the championship's progress dword into `+0xcc`, then streams each of its rounds through `champ_circuit_load_profile`, passing the entrant count at `+0xd4` so the final-positions array is read |
| 0x0060c9f0 | `championship_save_profile` | `SaveProfile__14C_championshipFR8G_Stream` | 0x003961d0 | CONFIRMED | writes the championship's progress dword at `+0xcc`, then streams each of its rounds through `champ_circuit_save_profile`, passing the entrant count so the final-positions array is sized correctly |
| 0x0060ca60 | `championship_set_opponents` | `SetOpponents__14C_championshipFPUiPUi` | 0x00396270 | STRUCTURAL | for each of the championship's `+0xd4` entries, allocates an 8-byte `{model_id, colour_or_variant}` pair from the two parallel arrays it is handed and appends it to the roster vector at |
| 0x00615060 | `championship_get_results` | `GetResults__14C_championshipFP10C_opponent` | 0x003a2df0 | CONFIRMED | linear-scans the championship's result table — 8-byte `{C_opponent*, points}` pairs starting at `this+0x00` — for the given racer and returns its point total; returns 0 when the racer is not entered |
| 0x00615080 | `championship_get_position` | `GetChampionshipPos__14C_championshipFP10C_opponent` | 0x003a2e40 | CONFIRMED | scans the 21-entry `{C_opponent*, points}` table embedded at the head of the championship (`+0x00`, stride 8) for the given racer and returns its index plus one; 0 when the racer is not in the table |

#### `C_champ_circuit`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0060c450 | `champ_circuit_load_profile` | `LoadProfile__15C_champ_circuitFR8G_StreamUi` | 0x00395b30 | CONFIRMED | reads the lock state into `+0x08`, and when it is 0 or 1 also the `entrants + 1` dwords of the final-positions array at `+0x0c`; then pushes the lock state down into the round's circuit at |
| 0x0060c4c0 | `champ_circuit_save_profile` | `SaveProfile__15C_champ_circuitFR8G_StreamUi` | 0x00395bf0 | CONFIRMED | writes the lock state at `+0x08`, and when it is 0 or 1 also the `entrants + 1` dwords of the final-positions array at `+0x0c` |
| 0x0060c520 | `champ_circuit_set_lock_state` | `SetLockState__15C_champ_circuitF13ER_lock_state` | 0x00395ca0 | CONFIRMED | stores the state at `+0x08` and, when the state is `1` (unlocked) — or `0` with a circuit attached — writes the same value into the owning `C_circuit`'s `+0x50` |
| 0x0060c540 | `champ_circuit_final_positions_set` | `FinalPositionsSet__15C_champ_circuitFv` | 0x00395dc0 | CONFIRMED | stores one final position; the four small `FinalPositions*` siblings are inlined here |

#### `C_opponent`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0060cf10 | `opponent_game_done` | `GameDone__10C_opponentFv` | 0x00396730 | CONFIRMED | clears the car pointer at `+0x28`; releases the driver-man model at `+0x34` (unhooking the rig with `game_del_driver_man`, dropping its frame placement with `game_del_frame_placement`, detaching it |
| 0x0060cfa0 | `opponent_game_init` | `GameInit__10C_opponentFP17I3D_animation_setUi` | 0x00396800 | CONFIRMED | builds the car model's file name from the car database, resolves the grid frame and copies its transform onto the car |
| 0x0060e000 | `opponent_create_driver` | `CreateDriver__10C_opponentFb` | 0x00397f60 | CONFIRMED | creates the racer's `C_driver` |
| 0x0060e0d0 | `opponent_recover` | `Recover__10C_opponentFv` | 0x00398010 | CONFIRMED | only runs once the respawn phase has parked at 5, the race phase is 1 and the racer has not finished |
| 0x0060eb40 | `opponent_do_recover` | `DoRecover__10C_opponentFR8S_vectorR8S_vector` | 0x00398af0 | CONFIRMED | moves the car's frame to the given position and direction, mirrors both into the car's own transform, resets the vehicle and drops it onto the ground, then makes it non-solid, detaches it from its |
| 0x0060ed10 | `opponent_tick_recover` | `TickRecover__10C_opponentFUi` | 0x00398c50 | CONFIRMED | accumulates the frame delta into `+0x88` against the 1000 ms period at `+0x8c`; each time it wraps, the player's respawn indicator advances through `hud_set_race_start_flag` and the phase counter |
| 0x0060ee40 | `opponent_create_driver_path` | `CreateDriverPath__10C_opponentFb` | 0x00399000 | CONFIRMED | clears the driver's path-node vector, picks the racer's segment class from its car and walks the circuit's left and right track paths in step, emitting one 0x70-byte node per pair — the racing line |
| 0x0060f750 | `opponent_get_race_points` | `GetRacePoints__10C_opponentFv` | 0x0039a000 | CONFIRMED | returns 0 for a retired racer (`+0x80`), otherwise maps its finishing place (`+0x68`) through the same 10/8/7/5/4/3/2/1 ladder |
| 0x0060f7c0 | `opponent_get_race_points_for_place` | `GetRacePoints__10C_opponentFUi` | 0x0039a090 | CONFIRMED | points for a place given as the argument |
| 0x0060f830 | `opponent_set_death` | `SetDeath__10C_opponentFb` | 0x0039a110 | CONFIRMED | retires the racer: `+0x44`, the finishing place, `+0x80`, and clears the lap time |
| 0x00611630 | `opponent_play_death` | `PlayDeath__10C_opponentFv` | 0x0039d7e0 | CONFIRMED | applies a fatal hit to the human at `+0x2c` with damage set one above his current health, of hit type 5, then advances his animation machine and pushes the resulting pose onto the car's frame |
| 0x00611700 | `opponent_tick` | `Tick__10C_opponentFUi` | 0x0039d8a0 | CONFIRMED | pushes the car's fuel value into its `C_Vehicle`, runs the overtaking tick, ticks the AI `C_driver`, and then two usability tests on the car: `car_is_usable(1)` failing destroys the driver and |
| 0x006117e0 | `opponent_tick_overtake` | `TickObjizdecka__10C_opponentFUi` | 0x0039dae0 | CONFIRMED | picks up the car the driver reports ahead (`C_driver+0x1a0`) into `+0x174`, bails out while the driver is flagged busy, then switches on the direction bit `+0x178` |
| 0x00611d10 | `opponent_end_overtake` | `EndOjed__10C_opponentFv` | 0x0039e2f0 | CONFIRMED | clears the direction bit `+0x178` and the target `+0x174`, restores the driver's path index from the saved `+0x188` and re-arms the driver (`C_driver+0x214`) |
| 0x00611f90 | `opponent_test_node_vs_cars` | `ObjzTestNodeValidCars__10C_opponentFR10S_car_node` | 0x0039e5d0 | CONFIRMED | walks the global actor list for every kind-4 actor that is neither the racer's own car nor the car being overtaken, transforms the candidate point into that car's local space, and tests it against |
| 0x00612100 | `opponent_test_node_vs_track_edges` | `ObjzTestNodeValid__10C_opponentFR10S_car_nodeUi` | 0x0039e5e0 | CONFIRMED | takes the driver path node at the given index, forms the segment from the racer's current position to it, and intersects that against both the left and the right track path (`circuit+0xb0` and |
| 0x00612290 | `opponent_overtake` | `Ojed__10C_opponentFP5C_car` | 0x0039f4b0 | CONFIRMED | records the target in `+0x174` and clears the rejection mask `+0x17c`, then takes the target's bounding box into world space |
| 0x006137c0 | `opponent_fill_path_node` | `FillObjzNode__10C_opponentFR8S_vectorfR8S_vectorP10S_car_node` | 0x003a0cd0 | CONFIRMED | initialises one driver path node at a position with a given radius |
| 0x00613880 | `opponent_get_next_driver_route` | `GetNextDriverRoute__10C_opponentFv` | 0x003a0e10 | CONFIRMED | reads the driver's current node index (`C_driver+0x170`), steps forward one and wraps to 0 at the end of the path-node vector |
| 0x006138d0 | `opponent_get_prev_driver_route` | `GetPrevDriverRoute__10C_opponentFv` | 0x003a0e70 | CONFIRMED | reads the driver's current node index (`C_driver+0x170`), steps back one and wraps to the last node when it would go below zero |
| 0x00613920 | `opponent_get_next_driver_route_from` | `GetNextDriverRoute__10C_opponentFUi` | 0x003a0ed0 | CONFIRMED | returns `index + 1`, wrapping to 0 when it reaches the end of the driver's path-node vector |
| 0x00613970 | `opponent_get_prev_driver_route_from` | `GetPrevDriverRoute__10C_opponentFUi` | 0x003a0f30 | CONFIRMED | returns `index - 1`, wrapping to the last node of the driver's path-node vector (`C_driver+0x164`/`+0x168`, stride 0x70) when it would go below zero |
| 0x00614220 | `opponent_finish` | `Finish__10C_opponentFUiUi` | 0x003a1860 | CONFIRMED | stores the place in `+0x68`, marks the racer settled (`+0x44`), keeps the finishing time in `+0x7c`, and banners `"%u %s"` — the place and the racer's name — for two seconds unless the race is |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00610d40 | `race_compare_qualification_time` | `compareQtime__FPCvPCv` | 0x0039c1f0 | CONFIRMED | orders two `C_opponent*` by the qualification time at `+0x38`, with the retired flag `+0x80` sorting last and `-1` meaning "no time set" (a racer without a time sorts behind one with any time) |
| 0x00610db0 | `race_compare_result_points` | - | - | - | orders 8-byte `{C_opponent*, points}` pairs descending on the CACHED point total in `pair[1]`, without a database lookup; returns 0 when either argument is null |
| 0x00610dd0 | `race_compare_champ_points` | `compareChampPoint__FPCvPCv` | 0x0039c2b0 | CONFIRMED | dereferences each `{C_opponent*, points}` pair, looks the racer's point total up through `championship_get_results` on the current championship (`g_racing+0x38`), and orders descending; a tie falls |
| 0x00610e30 | `race_compare_participant_slot` | - | - | - | a deterministic ordering that looks at no race data at all |
| 0x00610eb0 | `race_compare_position` | `comparePos__FPCvPCv` | 0x0039c350 | CONFIRMED | orders two `C_opponent*` by: retired-flag first (a retired racer sorts by its recorded finishing place), then lap counter `+0x6c`, then the gate index reached within the lap, then the remaining |
| 0x00615540 | `line2d_intersect_params` | `PrusecikPrimek__FR9S_vector2R9S_vector2R9S_vector2R9S_vector2R9S_vector2RfRf` | 0x003a30b0 | CONFIRMED | intersects two infinite 2-D lines and returns both line parameters |
| 0x00615610 | `seg2d_intersect_in_range` | `PrusecikUsecek__FRC9S_vector2RC9S_vector2RC9S_vector2RC9S_vector2R9S_vector2ffff` | 0x003a31e0 | CONFIRMED | intersects two 2-D segments and reports the hit only inside given parameter bands |

Notes on the rows: `racing_get_difficulty_scale` is 80 B against 124 B, and two PC-only helpers sit between `racing_tick` and it; with them placed, the rank matches. `racing_activate_racers` is statement for statement with exactly one callsite, in `racing_game_init`. `circuit_load_best_times` serves both PS2 loaders. `opponent_dtor` is STRUCTURAL because the PS2's is the empty delete-flag destructor; `opponent_game_done` is statement-identical; `opponent_tick_recover` has size ratio 0.32. `opponent_get_race_points` takes no place and reads the racer's own `+0x68`, `opponent_get_race_points_for_place` takes the place as an argument. `racing_get_segment_bank_for_car` is a wrapper, `racing_finish_segment_and_advance` absorbs `GetSegments`, and `racing_get_segment_id_for_car` is a free function on the PS2.

## 4. The object graph

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
                             +-- C_opponent*[20]          +0x1d8  entrants   (count +0x284)
                             +-- C_opponent*[20]          +0x228  live racers (count +0x288)
                             +-- C_opponent               +0x40   the player's entrant, embedded
```

`C_champ_circuit` points at a `C_circuit` but does **not** own it: the circuits live in
`C_racing`'s own vector and outlive every round.

## 5. `C_racing` — the singleton

The root object of the whole mode. `g_racing` (`0x006f958c`, PS2 `CRacing`) is the only
pointer to it. **In the retail PC build it is constructed in mission 6 only**: the mission's
`GameInitStart` script runs `racing_mission6_init`, and `race_mission6_init` builds the object,
writes `+0x28 = 2` and `+0x2c = 3`, and selects circuit 0 (`basic/artifacts/racing.md` §8a,
`structs/race_session.md` §6). In every other mission it is NULL, which is why ~310 references to
it in ordinary gameplay code are `== 0` tests: they are "are we in a race?" gates.

The menu front end is `race_menu_run` (`0x00595d90`, PS2 `ExecuteRacingMenu`): it builds the
design-`0x16` Racing menu and dispatches championship (`g_racing+0x2c` = 1), single race (= 2), a
third mode (= 3, the value the mission 6 race also uses), replay load, multiplayer and exit. Its
sole caller is `engine_main` case `0x15`, immediately after `racing_ctor`; that path is reached
only through the `Racing` main-menu item the shipped executable deletes.

## 6. The lap gates

`circuit_load_gates_from_scene` builds the gates from the mission's navigation web (`check.bin`).
Starting at the web node with ID `0x27c`, each node of the walk becomes a gate's **track node**;
its neighbour with exactly one connection is the gate's **outer marker**, the neighbour with more
connections is the next node of the walk, and neighbours with node category bit `0x8`
(category value 2, `structs/racecam.md` §4b) are race-camera nodes collected for `+0xcc`. The gates are linked into a ring. A gate prints the
checkpoint time when the first connection of its outer marker carries flag bit `0x80`. Circuit 0
(mission 6) has 27 gates; gates 9, 14 and 20 print.

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

## 7. `C_championship` and `C_champ_circuit`

One championship per record of `tables/championship.def`; one `C_champ_circuit` per round. `C_championship+0x00` is the standings table, scanned by `championship_get_results` and rebuilt by `racing_fill_champ_results`.

`champ_circuit_set_lock_state` is the **track-unlock mechanism**: winning a round writes the unlocked state into the round and mirrors it into the `C_circuit`, which is what makes the track selectable in single-race mode. The racing mod's readme describes exactly this behaviour from the player's side ("Races can be played in reverse from single race menu once you unlock them through winning in championship").

## 8. `C_opponent` — one racer

The heaviest class of the family: 30 PS2 methods, of which the overtaking AI (`Ojed*`, `TickObjizdecka`, `ObjzTestNodeValid*`) is more than half.

**An entrant's look is its car.** `opponent_game_init` builds the car model's file name from the car database record selected by `+0x24` (`car_database_get_car_model`, `"%s%02d.i3d"`, whose two digits are that record's colour variant). No other `C_opponent` field carries a colour or a skin.

## 9. The free comparators

Three of the comparators are **taken by address only** (`crt_qsort(…, 4, &LAB_…)`), so they have no `E8` caller at all; every reference to them is a label operand. `race_compare_participant_slot` is the exception: `race_compare_champ_points` calls it normally. The two qsort-only comparators are `race_compare_position` (272 B) and `race_compare_qualification_time` (112 B), against 328 B / 184 B PS2 methods.

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

## 10. Racing over the network

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
written back to `g_racing+0x60`), 3 = `g_racing+0x27c`, 4 = `mirror_track` (`g_racing+0x2b4`),
5 = `racing_get_current_champ_id`. Selector 5 additionally prunes every racer whose car fails
`car_database_is_enabled_for_championship`.

**A 21-slot cap.** `net_session_add_ai_racer` refuses once peers + AI racers reach `0x15`.

The type-`0x3a` producer sits inside `camera_tick` (`Tick__8G_CameraFUi`), the `G_Camera`
per-frame driver: it picks `g_active_race_camera` from a range list and writes `g_racing+0x98`.
See `structs/camera.md`.

Full write-up of the message layer: `structs/net_event_msg.md`.

## 11. Coverage of the translation unit

`[0x0060c130, 0x00617a60)` enumerates to **120 real starts** — a scan for the decompiler's
Aggressive-Instruction-Finder artifact shapes finds none left in that range — and every body has
been read. The TU as a whole is 131/131.

### 11a. Measured coverage

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

### 11b. Twelve PC-only functions are the multiplayer half of `C_racing`

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

### 11c. The segment subsystem is on the PC too — in a second translation unit

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

The full write-up of that cluster is §12.

### 11d. PS2 methods with no PC start in this TU

Each was checked at the position of its PS2 callers inside the matching PC function.

Inlined, and marked in the PC decomp with `//` banners at the exact statement:
`GetLeftPath` / `GetRightPath` (in `racing_game_init` and `racing_activate_racers`),
`GetOpponentName` / `GetOpponentNameIdx` (in `racing_fill_opponent_name_and_model`),
`GetOpponentFromCar` (twice in `racing_predict_car_collision`), the four `OjedMe*` quadrant
arms (in `opponent_overtake`), and the four small `FinalPositions*` siblings (in
`champ_circuit_final_positions_set`).

Absent, with the PC doing something else at the caller's position (§13):
`SetRCamIndexFirstRacer`, `GetSegmentTotalTime`, `GetRecoverTime`, `PreloadWrecks`,
`CreateBestTimes` with its comparator `compSegID`, and the six `C_championship` /
`C_champ_circuit` best-time streamers. The two `C_circuit` loaders are one PC function,
`circuit_load_best_times` (§3).

## 12. The optimal-line SEGMENT system

The AI racing-line database, its own translation unit at `[0x005c9720, 0x005ca290)` — the second
PC cluster §11c places.

### 12a. What it stores

`C_racing` keeps **seven banks** of segments at `+0x2c4` (stride `0x10`), one per car
performance class; `racing_get_segment_id_for_car` (`0x00615230`) returns the bank index
0..6 from the car's top speed and model id, and `racing_get_segment_bank_for_car` is the
`C_racing` method that resolves it for the player's chosen car (`+0x64`).

Each bank holds pointers to `C_racing_optimal_segment` records:

(the record is §1i)

The class name `C_racing_osegment` does not settle whether its "o" stands for *optimal* or
*opponent*: the records are written by the best-time learner (§12b) and read by the
opponents' path builder (§12d), and no symbol or string of either build spells the word out.
The PC prefix follows the learner.

**`C_racing` EMBEDS one at `+0x334`** as the scratch record being recorded — byte-proven
by `lea ecx,[esi+0x334]` in `racing_ctor` calling `racing_optimal_segment_ctor`. The three
fields after it are the recorder's own cursors: `+0x3dc` current segment index, `+0x3e0`
point write cursor, `+0x3e4..+0x3ef` the last sampled car position.

**The class has NO destructor on either build.** No `__dt__17C_racing_osegment` symbol
exists in the PS2 ELF, and both builds free a record with a bare `operator delete`. The
record owns no pointer, so there is nothing to destroy — this is a measurement, not a gap.

### 12b. How it is filled — a best-time learner, not a recording

`racing_tick` calls `racing_record_segment_points` every frame with the car's position, its
displacement since the last frame and its speed. For every circuit cross-line
(left-path node *i* to right-path node *i*) the movement crossed, the crossing point and the
speed go into the scratch record. When the scratch fills,
`racing_finish_segment_and_advance` commits it into the bank **only if this lap's time for
that stretch beat the stored one** and the learn flag at `C_racing+0x2b6` is set, drops each
point onto the ground with a downward ray, then advances to the next segment.

That min-time gate is what makes the stored data an *optimal* line rather than a trace.

### 12c. Files

| file | direction | when |
|---|---|---|
| `system\mrseg%i.sav` / `system\mrsegm%i.sav` | read | `racing_game_init`, per circuit id |
| `savegame\mrseg%i.sav` / `savegame\mrsegm%i.sav` | write | `racing_game_done` |

The `m` variant is selected by `mirror_track` (`C_racing+0x2b4`). Reading from `system\` and writing to
`savegame\` is a record-then-ship developer workflow: the shipped data lives in `system\`,
and anything learned during play lands in `savegame\`.

Per bank the format is a segment count, then per segment: point count, segment time, the
points, the speeds.

### 12d. Who reads it — the AI racers' driving line

`opponent_create_driver_path` resolves the bank of the racer's car class
(`racing_get_segment_id_for_car`, then `C_racing+0x2c4 + class * 0x10`) and copies each
segment's points and per-point speeds into the racer's `C_driver` path (`structs/driver.md`
§3). The stored line is what the AI racers drive along, at the speeds recorded with it.

## 13. PC vs PS2

All cross-build material for these classes lives in this section.

| Area | PC | PS2 |
|---|---|---|
| the optimal-line segment subsystem | present, but in a **second translation unit** at `[0x005c9720, 0x005ca290)` (§12) | in the `C_racing` run, as `C_racing_osegment` + `CreateSegments` / `LoadSegments` / `ClearSegments` / `GetSegments` / `GetSegmentTotalTime` |
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
| session back-pointers on `C_opponent` (`+0x50` / `+0x54`) | present — the multiplayer half of the class (§11b) | no trace |
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
of a *different placement* — a second translation unit (§12) — not of absence.

### 13a. Field pairs

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
| `C_circuit` | `+0x44` | `+0x4c` | gate vector |
| `C_circuit` | `+0x50` | `+0x00` | lock state (PS2 mirrors it at circuit `+0x00`) |
| `C_circuit` | `+0x58` | `+0x58` | best-lap table |
| `C_circuit` | `+0xb0` | `+0x64` | left racing line ¹ |
| `C_circuit` | `+0xc0` | `+0x70` | right racing line ¹ |
| `C_circuit` | `+0xd0` | `+0x7c` | race-camera nodes |
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

¹ The sides are pinned by `RadarSetLeftEdge` taking `circuit+0x64` on PS2 and the vector at `+0xac` on PC, and
by `RadarSetRightEdge` for the right line.

**The PC↔PS2 delta is NOT uniform** — `+0x228`↔`+0xa3c` is `0x814`, `+0x288`↔`+0xac4` is
`0x83c`, `+0x2b7`↔`+0xb07` is `0x850`. Do not rebase band-wise; the two builds are different
revisions of the class, not a shifted copy (the same holds for `structs/detector.md` §7 and
`structs/pumper.md` §8).

### 13b. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001dc4b0 | `GetSegments__8C_racingFUiUi` | absorbed by `racing_finish_segment_and_advance` |
| 0x001dc500 | `GetSegmentTotalTime__8C_racingFUiUi` | absent, with the PC doing something else at the caller's position: qualifying times interpolate the circuit tables |
| 0x001dbe40 | `GetRightPath__9C_circuitFv` | inlined into `racing_game_init` and `racing_activate_racers` |
| 0x001dbe60 | `GetLeftPath__9C_circuitFv` | inlined into `racing_game_init` and `racing_activate_racers` |
| 0x00395730 | `LoadBestTimes__9C_circuitFR8G_Stream` | one PC function, `circuit_load_best_times`, serves both PS2 loaders |
| 0x00395ce0 | `SetFinalPos__15C_champ_circuitFUiUi` | inlined into `champ_circuit_final_positions_set` |
| 0x00395d10 | `FinalPositionsClear__15C_champ_circuitFv` | inlined into `champ_circuit_final_positions_set` |
| 0x00395d80 | `FinalPositionsRestore__15C_champ_circuitFv` | inlined into `champ_circuit_final_positions_set` |
| 0x00395da0 | `FinalPositionsStore__15C_champ_circuitFv` | inlined into `champ_circuit_final_positions_set` |
| 0x00399f70 | `SetRCamIndexFirstRacer__8C_racingFv` | absent, with the PC doing something else at the caller's position: `racing_tick` calls `racing_set_race_camera_index` with current + 1 |
| 0x0039b3e0 | `GetOpponentName__8C_racingFUi` | inlined into `racing_fill_opponent_name_and_model` |
| 0x0039b4d0 | `GetOpponentNameIdx__8C_racingFPc` | inlined into `racing_fill_opponent_name_and_model` |
| 0x0039c760 | `compSegID__FP11C_best_timeP11C_best_time` | the comparator of `CreateBestTimes`, absent with it |
| 0x0039c770 | `CreateBestTimes__8C_racingFv` | absent, with the PC doing something else at the caller's position: on a load failure `racing_ctor` saves the defaults the constructors seeded |
| 0x0039e870 | `OjedMeZePredu__10C_opponentFP5C_car` | inlined into `opponent_overtake` |
| 0x0039eb80 | `OjedMeZleva__10C_opponentFP5C_car` | inlined into `opponent_overtake` |
| 0x0039ee90 | `OjedMeZprava__10C_opponentFP5C_car` | inlined into `opponent_overtake` |
| 0x0039f1a0 | `OjedMeZeZadu__10C_opponentFP5C_car` | inlined into `opponent_overtake` |
| 0x003a44d0 | `GetOpponentFromCar__8C_racingFP5C_car` | inlined into `racing_predict_car_collision`, twice |
| 0x003a59f0 | `GetRecoverTime__8C_racingFv` | absent, with the PC doing something else at the caller's position: `opponent_tick_recover` runs 4 steps of 1000 ms, constant |
| 0x003a5a60 | `SaveBestTimes__14C_championshipFR8G_Stream` | absent: the PC's best-time file holds circuits only |
| 0x003a5ae0 | `LoadBestTimes__14C_championshipFR8G_Stream` | absent: the PC's best-time file holds circuits only |
| 0x003a5b60 | `LoadBestTimesDta__14C_championshipFi` | absent: the PC's best-time file holds circuits only |
| 0x003a5be0 | `SaveBestTimes__15C_champ_circuitFR8G_Stream` | absent: the PC's best-time file holds circuits only |
| 0x003a5cb0 | `LoadBestTimes__15C_champ_circuitFR8G_Stream` | absent: the PC's best-time file holds circuits only |
| 0x003a5e10 | `LoadBestTimesDta__15C_champ_circuitFi` | absent: the PC's best-time file holds circuits only |
| 0x003a5f70 | `PreloadWrecks__8C_racingFv` | absent, with the PC doing something else at the caller's position: no preload step in `racing_game_init` |

## 14. Related

`basic/artifacts/racing.md` (the subsystem) · `basic/artifacts/racing/lap-gates-and-lap-glitch.md` (the gates in play) · `structs/racecam.md` · `structs/race_session.md` · `structs/net_session.md` · `structs/driver.md` (`S_car_node`, stride `0x70`) · `structs/racing_menus.md`.

## 15. Open items

1. **Offsets with no recovered meaning**, each with its store and the scope of the absence:

| Class | Offset | Field | Store | Scope |
|---|---|---|---|---|
| `C_racing` | +0x0 | `unread_00` | `racing_game_init` 0 | class census; offset 0 cannot be base-traced |
| `C_racing` | +0x278 | `unread_278` | `racing_ctor` 0 | `.text scan` |
| `C_opponent` | +0x40 | `unused_40` | - | class census |
| `C_opponent` | +0x58 | `unread_58` | the ctors and `opponent_game_init` -1 | class census |
| `C_circuit` | +0x64 | `unused_64` | - | class census |

2. **`C_championship` `index`** has no reader (class census).
3. **In game (needs the racing mod):** drive a circuit twice, improving on one stretch, and check that `savegame\mrseg<circuit>.sav` grows or changes; that would confirm the learner runs in the shipped build rather than only in a developer configuration.
