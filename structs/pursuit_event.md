# `pursuit_event` — the chase driving system

**Created** 2026-07-09 20:19 +0200 · **Modified** 2026-11-08 22:21 +0100

The engine-wide system that drives a car physically chasing, fleeing or heading for a target: police
pursuits, AI combat chases and scripted mission chases alike. Distinct from the police squad
bookkeeping in `structs/police-board.md` / `structs/police-action.md`: those decide *who* chases;
`pursuit_event` executes the *driving*. One instance is embedded in
`C_game` at `+0x2b18`. Reconstructed from static analysis; §7 gives the addresses for a Cheat Engine
check at run time.

| | |
|---|---|
| PC prefix | `pursuit_event_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x24` (the next `C_game` member, `soundArena`, starts at `+0x2b3c`) |
| PC constructor | `pursuit_event_ctor` @ `0x004ec1b0` |
| PC translation unit | `[0x004ec180, 0x004f60b0)`: 53 starts, all named: the unit's CRT pair, the 21 `pursuit_event_*` functions, 17 container helpers and 13 header inlines of other classes (§6); the next unit's guard `crt_static_init_guard_004f60b0` follows |
| PS2 class | `C_sledovacka` (tag `12C_sledovacka`), 22 methods: 21 CONFIRMED twins, the destructor STRUCTURAL on a shared PC body (§3a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct pursuit_event {                 // 0x24 bytes; at C_game + 0x2b18
    /* +0x00 */ uint8_t    bridges_alloc;      // [H] allocator byte of the bridge vector
    /* +0x04 */ S_BRIDGE  *bridges_begin;      // [H] std::vector<S_BRIDGE>, stride 8
    /* +0x08 */ S_BRIDGE  *bridges_end;        // [H] one past the last bridge entry
    /* +0x0c */ S_BRIDGE  *bridges_cap;        // [H] end of the bridge vector's storage
    /* +0x10 */ uint8_t    records_alloc;      // [H] allocator byte of the chase-record vector
    /* +0x14 */ S_ELEMENT *records_begin;      // [H] std::vector<S_ELEMENT>, stride 0x44
    /* +0x18 */ S_ELEMENT *records_end;        // [H] one past the last chase record
    /* +0x1c */ S_ELEMENT *records_cap;        // [H] end of the record vector's storage
    /* +0x20 */ C_roads   *roads;              // [H] the road graph, cached on first use
} pursuit_event;
```

Every `AddCarHunt` / `RemoveCar` callsite loads `this` with
`mov eax,[0x0065115C]; mov ecx,[eax+0x24]; add ecx,0x2B18`. `+0x20` is copied once from
`*(g_act_mission + 0x30)`, the `C_roads` singleton (`structs/roads.md`), by whichever of the three
dispatchers or `pursuit_event_save_game_load` runs first; the path builders then index the crossing
table (`roads + 0x08`, stride `0xec`) and the node table (`roads + 0x18`, stride `0x18`) through it.

The class also owns a **file-static** byte outside the instance: the boost flag
`g_pursuit_change_tables` at `0x0064C6D8` (§5c).

The manager has no destructor of its own: `game_dtor` destroys it with the body it shares with the
force-feedback manager, `folded_vector_pair_free_at_0x04_0x14` (§3a).

### 1a. `S_BRIDGE` — one bridge on the road graph, 8 bytes

```c
typedef struct S_BRIDGE {                       // 8 bytes; one per C_bridge actor (kind 0x14)
    /* +0x00 */ C_actor  *bridge;              // [H] the bridge actor
    /* +0x04 */ uint16_t  crossing;            // [H] the crossing the bridge controls
} S_BRIDGE;
```

`pursuit_event_init` walks every mission actor of kind `0x14`, snaps its position to the nearest
road node (`roads_get_near_node`), and stores whichever of that node's two crossings
(`S_road_node_1 + 0x14`, `structs/roads.md` §1b) lies nearer the bridge. `pursuit_event_recompile_path`
then matches each path node that sits on a signal-controlled crossing against this list, so a chase
car waits on the bridge's act state (`C_bridge + 0x138`) instead of the traffic light.

### 1b. `S_ELEMENT` — one chase record, `0x44` bytes

One record per car the system is driving. Built by the three `AddCar*` entry points (§5a); the
dispatch keeps one record per vehicle and requires any target to be entity type 2 (player actor) or
`0x1b` (NPC).

```c
typedef struct S_ELEMENT {                      // 0x44 bytes; element of the +0x14 vector
    /* +0x00 */ C_car    *vehicle;             // [H] the chase car; the record's key
    /* +0x04 */ C_driver *driver;              // [H] the 0x29c-byte driver (structs/driver.md)
    /* +0x08 */ C_actor  *chase_object;        // [H] what the car drives at: the target's car, or the target on foot (§2b)
    /* +0x0c */ C_actor  *target;              // [H] the pursued actor; 0 = drive to destination
    /* +0x10 */ float     destination[3];      // [H] the set position when there is no target; not saved
    /* +0x1c */ uint32_t  event_flags;         // [H] phase, mode and side bits (§4b)
    /* +0x20 */ uint32_t  action_status;       // [H] E_STATUS; 0 at build, getter default 1
    /* +0x24 */ uint32_t  mode;                // [H] the chase type ("VoleMode"), set at build (§4)
    /* +0x28 */ uint32_t  mode_timer_ms;       // [H] armed, stepped and reset by the tick
    /* +0x2c */ uint32_t  cadence_timer_ms;    // [H] the re-evaluation and stall timer
    /* +0x30 */ uint32_t  unread_30;           // [U] zero-filled by the dispatchers and the loader; no reader (.text scan)
    /* +0x34 */ C_car    *avoid_car;           // [H] the car the driver is steering around
    /* +0x38 */ float     mode_limit;          // [H] hunt: duration in ms; escape: distance squared
    /* +0x3c */ float     mode_limit_b;        // [H] escape: speed cap, m/s; hunt: path-cursor threshold (§2b)
    /* +0x40 */ int32_t   horn_timer_ms;       // [H] horn on while > 0; -1 = no car close ahead
} S_ELEMENT;
```

**Why `destination[3]`.** It is one `S_vector` (x, y, z): `AddCarHunt` copies its position
argument there only when no target actor is given.

**What the dispatch writes outside the record.** On the `C_driver` (layout in `structs/driver.md`):
`+0x84` = the speed factor argument (`g_police_speed_factor` from the police callers), `+0x170` = 0,
`+0x1fc` = target, `+0x200` = target vehicle, `+0x208` = 1, `+0x218` = 1000.0f. On the chase
`C_car`: `+0x2038` = `0xc00` and `+0x1ac` = 1 (the under-pursuit flag), and the radar blip is
repainted red. `pursuit_event_remove_vehicle` clears `+0x1ac`, `+0x51c` and `+0x215a`, writes
`+0x2038` = 0 and `+0x2040` = 1.0f; the tick toggles `+0x215a`.

## 2. Field evidence

Measured by body reads of every `pursuit_event_*` function and a `.text` displacement census of the
manager offsets inside the unit.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | bridges_alloc | H | `pursuit_event_ctor` | - | allocator byte |
| +0x04 | bridges_begin | H | `pursuit_event_ctor` (0), `pursuit_event_init` (grow) | `pursuit_event_init`, `pursuit_event_done`, `pursuit_event_recompile_path` | - |
| +0x08 | bridges_end | H | `pursuit_event_ctor` (0), `pursuit_event_init`, `pursuit_event_done` (= begin) | `pursuit_event_init`, `pursuit_event_recompile_path` | - |
| +0x0c | bridges_cap | H | `pursuit_event_ctor` (0), `pursuit_event_init` | `pursuit_event_init` | - |
| +0x10 | records_alloc | H | `pursuit_event_ctor` | - | allocator byte |
| +0x14 | records_begin | H | `pursuit_event_ctor` (0), the three dispatchers (insert) | every record lookup; `traffic_car_activate_elements` (as `C_game +0x2b2c`: is a chase car near the spawn point) | - |
| +0x18 | records_end | H | `pursuit_event_ctor` (0), the three dispatchers (insert), `pursuit_event_remove_vehicle` (erase-shift), `pursuit_event_done` (= begin) | every record lookup; `traffic_car_activate_elements` (as `C_game +0x2b30`) | - |
| +0x1c | records_cap | H | `pursuit_event_ctor` (0), the insert helpers | the insert helpers | - |
| +0x20 | roads | H | `pursuit_event_ctor` (0), the three dispatchers and `pursuit_event_save_game_load` (from `C_mission + 0x30` while 0) | the three dispatchers, `pursuit_event_recompile_path`, `pursuit_event_prepare_path`, `pursuit_event_save_game_load` | §1 |

### 2a. `S_BRIDGE`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | bridge | H | `pursuit_event_init` | `pursuit_event_recompile_path` | read for `bridge + 0x138` |
| +0x04 | crossing | H | `pursuit_event_init` | `pursuit_event_recompile_path` | the match key |

### 2b. `S_ELEMENT`

The record readers are a closed set. The record vector is reached only through `C_game +0x2b18`:
each of its 85 address formations in `.text` leads to a `pursuit_event_*` method, to
`folded_vector_pair_free_at_0x04_0x14`, to an unwind funclet, or to the getter
`game_get_pursuit_event_manager`, whose two callers call `pursuit_event_remove_vehicle` and
`pursuit_event_record_set_action_status_by_vehicle`. The one direct read of the vector from outside,
`traffic_car_activate_elements`, loads record `+0x00` only.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vehicle | H | the three dispatchers | every `pursuit_event_record_*_by_vehicle` lookup, `pursuit_event_is_car_chased`, `pursuit_event_remove_vehicle`, `pursuit_event_drop_actor_references`, the savegame triple, `traffic_car_activate_elements` (then the car's `+0x24..+0x2c`) | the de-dup and match key |
| +0x04 | driver | H | the three dispatchers (`operator_new(0x29c)` + `driver_ctor`) | `pursuit_event_tick`, `pursuit_event_remove_vehicle` and `pursuit_event_done` (`driver_dtor` + free), `pursuit_event_drop_actor_references`, the savegame triple | - |
| +0x08 | chase_object | H | `pursuit_event_dispatch_pursuit_vehicle` and `pursuit_event_dispatch_escape_vehicle` (`target + 0x98`); `pursuit_event_tick` (every tick of a record with a target: the target's car, or the target itself when it has none); `pursuit_event_save_game_load` (from its id) | `pursuit_event_tick` (the object driven at: car `+0x484` when its kind is 4, else `actor_get_world_position`; equal to `+0x0c` means the target is on foot and selects driver `+0x204`; other records with the same object in the side-by-side logic), `pursuit_event_record_is_chase_tracking_by_vehicle` (`!= 0`), `pursuit_event_drop_actor_references`, the savegame triple | a `C_car` or the target itself; 0 after a dispatch whose target is on foot, until the first tick |
| +0x0c | target | H | the three dispatchers | `pursuit_event_dispatch_pursuit_vehicle` (the close-and-visible test), `pursuit_event_tick` (against `+0x08`), `pursuit_event_drop_actor_references`, the savegame triple | type 2 or `0x1b` |
| +0x10 | destination | H | `pursuit_event_dispatch_pursuit_vehicle` (when `target == 0`) only | `pursuit_event_tick`, through the walking record pointer: the end of the road path in the road-approach phase, the breadcrumb node appended when the last node is more than 5 units off, the 100-unit test that switches `vehicle_set_dynamic_colls_off` and car `+0x215a`, the 50-unit test of the status code, the direct-follow end point and the 120-unit line-of-sight test | not saved, so a restored mode-2 record has 0, 0, 0 (§5b) |
| +0x1c | event_flags | H | `pursuit_event_dispatch_pursuit_vehicle` (`(argument & 0x201f00) \| 1`), `pursuit_event_dispatch_escape_vehicle`, `pursuit_event_dispatch_move_to_vehicle`, `pursuit_event_tick`, `pursuit_event_save_game_load` | `pursuit_event_tick`, `pursuit_event_record_is_chase_tracking_by_vehicle` (bit `0x4`), `pursuit_event_save_game_save`, `pursuit_event_save_game_load` | bit table §4b |
| +0x20 | action_status | H | the dispatchers (0; `AddCarHunt` 1 when the car starts within 12 units of an unblocked target that is not a moving car), `pursuit_event_record_set_action_status_by_vehicle`, `pursuit_event_tick`, `pursuit_event_drop_actor_references` (1 when the target or its vehicle is dropped) | `pursuit_event_record_get_action_status_by_vehicle` (default 1), `pursuit_event_remove_vehicle` (`!= 3` resets power and brake), `pursuit_event_tick`, `pursuit_event_save_game_save` | E_STATUS |
| +0x24 | mode | H | the dispatchers only: hunt 1 / 2, escape 3, move-to 4 | `pursuit_event_record_get_mode_by_vehicle` (default 0), `pursuit_event_remove_vehicle` (3 / 4 un-boost), `pursuit_event_record_is_chase_tracking_by_vehicle` (== 1), `pursuit_event_save_game_save` | §4 |
| +0x28 | mode_timer_ms | H | the dispatchers (0); `pursuit_event_tick` (armed to 1, frame time added, reset to 0) | `pursuit_event_tick` (against `+0x38`, 2000 ms and 15000 ms), `pursuit_event_record_is_chase_tracking_by_vehicle`, `pursuit_event_save_game_save` | non-zero while the timer runs |
| +0x2c | cadence_timer_ms | H | `pursuit_event_tick` (frame time added; reset after 200 ms or 500 ms) | `pursuit_event_tick` (10000 ms below speed 3.0 while turned more than 0.52 rad off the heading sets `action_status` 1 or 2), `pursuit_event_save_game_save` | the re-evaluation period |
| +0x30 | unread_30 | U | the three dispatchers and `pursuit_event_save_game_load` (zero fill of the record built on the stack) | - | copied with the whole record; no reader (.text scan: the closed set above, and none of the class's bodies loads it) |
| +0x34 | avoid_car | H | `pursuit_event_tick_driver` (from driver `+0x1a0` while driver `+0x19c` bit `0x40000000` is set; 0 once the path cursor passes the inserted detour, and 0 when the next node lies behind the car), `pursuit_event_drop_actor_references` (0) | `pursuit_event_tick_driver` (`car_get_navigation_corners`) | a `C_car` |
| +0x38 | mode_limit | H | `pursuit_event_dispatch_pursuit_vehicle` (argument × 1000, ms), `pursuit_event_dispatch_escape_vehicle` (argument squared) | `pursuit_event_tick` (against `+0x28` and against a squared distance), `pursuit_event_save_game_save` | - |
| +0x3c | mode_limit_b | H | `pursuit_event_dispatch_escape_vehicle` (argument × 0.2777778, km/h to m/s); `pursuit_event_tick` in a hunt (10000.0 on the first route, 100000.0 while the target is within 100 units and in line of sight, -1.0 when not, and 1.0 less for each path node it drops); `pursuit_event_save_game_load` | `pursuit_event_tick`: in an escape against the car's speed; in a hunt the give-up timer `+0x28` runs only while the driver's path cursor (driver `+0x170`) is above it; `pursuit_event_save_game_save` | two uses, like `mode_limit`: a speed cap in an escape, a path-cursor threshold in a hunt; the savegame stores it (save-record dword `0x2d`) |
| +0x40 | horn_timer_ms | H | `pursuit_event_tick` (frame time subtracted; -1 when no traffic car is close ahead; re-armed to 200..799 ms at random above 4 m/s) | `pursuit_event_tick` (horn on while > 0) | runs only while the siren is off |

## 3. Method family

Coverage: all **22** PS2 `12C_sledovacka` methods are mapped: 21 CONFIRMED twins on the class's own
functions, and the destructor STRUCTURAL on a body the PC shares with the force-feedback manager.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004ec180 | `crt_static_init_guard_004ec180` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004ec1a0 | `crt_atexit_register_004ec1a0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x004ec1b0 | `pursuit_event_ctor` | instance | `__ct__12C_sledovackaFv` | 0x00202ee0 | CONFIRMED | zeroes both vectors and `roads`; called by `game_ctor` |
| 0x005ae6f0 | `folded_vector_pair_free_at_0x04_0x14` | instance | `__dt__12C_sledovackaFv` | 0x00202f40 | STRUCTURAL | the destructor: frees the `+0x14` vector, then the `+0x04` one. A two-owner fold: the same body destroys the force-feedback manager (`structs/force_feedback.md`). Reached from `game_dtor` (`lea ecx,[esi+0x2b18]`) and two unwind funclets of `game_ctor` |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004ec1e0 | `pursuit_event_set_change_tables` | `EnableChangeTables__12C_sledovackaFb` | 0x00202fd0 | CONFIRMED | writes the boost flag `g_pursuit_change_tables` (§5c); 18 callsites, mostly the police squads |
| 0x004ec1f0 | `pursuit_event_init` | `Init__12C_sledovackaFv` | 0x00202fe0 | CONFIRMED | builds the bridge list (§1a); called by `game_init` |
| 0x004ec4f0 | `pursuit_event_done` | `Done__12C_sledovackaFv` | 0x00203230 | CONFIRMED | destroys every record's driver and empties both vectors; called by `game_done` |
| 0x004ec590 | `pursuit_event_is_car_chased` | `HoniMiHoNekdoZKanclu__12C_sledovackaFP5C_car` | 0x00203330 | CONFIRMED | has this car a record? Called by `pumper_can_pump` |
| 0x004ec5f0 | `pursuit_event_dispatch_pursuit_vehicle` | `AddCarHunt__12C_sledovackaFP5C_carP7C_actorUiUiRC8S_vectorf` | 0x002033b0 | CONFIRMED | enrolls a car with a target actor (mode 1) or a position (mode 2); applies the model-level boost when the flag is set; 13 callsites |
| 0x004ec980 | `pursuit_event_dispatch_escape_vehicle` | `AddCarEscape__12C_sledovackaFP5C_carP7C_actorRC8S_vectorbfff` | 0x00203c10 | CONFIRMED | enrolls a car fleeing a target actor (mode 3) ¹ |
| 0x004ed0b0 | `pursuit_event_dispatch_move_to_vehicle` | `AddCarMoveTo__12C_sledovackaFP5C_carRC8S_vectorbbUif` | 0x00204b00 | CONFIRMED | enrolls a car driving to a destination (mode 4) |
| 0x004edac0 | `pursuit_event_remove_vehicle` | `RemoveCar__12C_sledovackaFP5C_car` | 0x00206030 | CONFIRMED | cancels a car's record, resets its pursuit bytes and model level, frees the driver, erase-shifts; 45 callsites |
| 0x004edc20 | `pursuit_event_recompile_path` | `RecompilePath__12C_sledovackaFi` | 0x002062f0 | CONFIRMED | rebuilds the driver path from the road graph and links signal-controlled crossings to their light or bridge (§1a) |
| 0x004ee740 | `pursuit_event_prepare_path` | `PreparePath__12C_sledovackaFi` | 0x00207310 | CONFIRMED | prepares the path ahead of the recompile |
| 0x004ee900 | `pursuit_event_tick_driver` | `TickDriver__12C_sledovackaFiUi` | 0x002075b0 | CONFIRMED | the per-record driving tick; called three times by the tick |
| 0x004efcf0 | `pursuit_event_tick` | `Tick__12C_sledovackaFUi` | 0x002095a0 | CONFIRMED | the per-frame tick over every record; called by `game_tick` |
| 0x004f4470 | `pursuit_event_record_get_action_status_by_vehicle` | `ActionStatus__12C_sledovackaFP5C_car` | 0x0020fcf0 | CONFIRMED | `action_status` of the car's record, default 1 |
| 0x004f44d0 | `pursuit_event_record_set_action_status_by_vehicle` | `SetActionStatus__12C_sledovackaFP5C_car8E_STATUS` | 0x0020fd80 | CONFIRMED | sets it; called by `ai_fight_manager_tick` and a `program_process_command` arm |
| 0x004f4530 | `pursuit_event_record_get_mode_by_vehicle` | `GetVoleMode__12C_sledovackaFP5C_car` | 0x0020fe10 | CONFIRMED | `mode` of the car's record, default 0; called by `police_action_tick` |
| 0x004f4590 | `pursuit_event_drop_actor_references` | `VypicujMiToyotu__12C_sledovackaFP7C_actor` | 0x0020fea0 | CONFIRMED | removes the actor's own record, or clears it from every record and driver that tracks it; called by `game_invalidate_actor` |
| 0x004f46e0 | `pursuit_event_record_is_chase_tracking_by_vehicle` | `DevaaateeeZasNeznaCas__12C_sledovackaFP5C_car` | 0x002100b0 | CONFIRMED | the mode timer while an actor hunt runs (§4a) |
| 0x004f4760 | `pursuit_event_save_game_get_size` | `SaveGameGetSize__12C_sledovackaFv` | 0x00210170 | CONFIRMED | §5b |
| 0x004f4870 | `pursuit_event_save_game_save` | `SaveGameSave__12C_sledovackaFPPv` | 0x00210300 | CONFIRMED | §5b |
| 0x004f4dc0 | `pursuit_event_save_game_load` | `SaveGameLoad__12C_sledovackaFPPv` | 0x00210a80 | CONFIRMED | §5b |

¹ The same de-dup, the same `{2, 0x1b}` gate, `vehicle_set_engine_on` and `C_driver` sub-record as
`AddCarHunt`; the km/h to m/s factor `× 0.2777778` (= /3.6), a squared distance, and a bool to
1000.0f.

`ai_fight_manager_join_party` (`0x0043E610`) and `ai_fight_manager_adjust_form_time`
(`0x0043F950`) are not members: their receiver is the `ai_fight_manager` singleton `0x00655318`, and
the stride-`0x48` list they walk is a fight party (`structs/ai_fight_manager.md` §3c).

### 3d. Free helpers

The unit's container helpers, emitted here because the class is their only user in this object
file. No class but `pursuit_event` shares the unit, so all are listed here.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004f5570 | `std_vector2_insert_one` | - | - | - | insert one 2-byte element; `pursuit_event_dispatch_escape_vehicle` |
| 0x004f5710 | `std_vector2_size` | - | - | - | element count of a 2-byte vector |
| 0x004f5730 | `std_vector2_erase_one` | - | - | - | erase one 2-byte element; `pursuit_event_dispatch_move_to_vehicle` (4) |
| 0x004f5770 | `std_vector44_size` | - | - | - | record count; `pursuit_event_tick` |
| 0x004f57a0 | `std_vector44_at` | - | - | - | record by index; `pursuit_event_tick` |
| 0x004f5800 | `std_vector70_insert_one` | - | - | - | insert one driver path node; the move-to dispatcher and the tick |
| 0x004f5af0 | `std_vector2_insert_at` | - | - | - | insert at a position; `pursuit_event_dispatch_escape_vehicle` |
| 0x004f5d00 | `std_vector44_insert_one` | - | - | - | insert one record; the escape and move-to dispatchers ² |
| 0x004f5f10 | `std_vector70_erase_range` | - | - | - | erase driver path nodes; the move-to dispatcher, the tick and the savegame loader |
| 0x004f5f60 | `stl_copy_range_stride2` | - | - | - | copy 2-byte elements |
| 0x004f5f90 | `stl_fill_n_stride2` | - | - | - | fill 2-byte elements |
| 0x004f5fc0 | `stl_copy_range_stride8` | - | - | - | copy bridge entries; `pursuit_event_init` |
| 0x004f5ff0 | `stl_fill_n_stride8` | - | - | - | fill bridge entries; `pursuit_event_init` |
| 0x004f6020 | `stl_fill_n_stride44` | - | - | - | fill records |
| 0x004f6050 | `array_copy_u16` | - | - | - | copy `u16` ids; `pursuit_event_dispatch_escape_vehicle` |
| 0x004f6070 | `stl_copy_range_stride70` | - | - | - | copy driver path nodes; both dispatchers and the tick ² |
| 0x004f60a0 | `stl_copy_one_stride2` | - | - | - | copy one 2-byte element |

² The PS2 copy constructor of `S_ELEMENT` and the vector's `reserve` are inlined into
`std_vector44_insert_one`; the `S_car_node` copy constructor into `stl_copy_range_stride70` (§8).

## 4. `mode` — the chase type ("VoleMode", record `+0x24`)

The developers' name for the field is `VoleMode` (*vole* is slang, about "dude"; its siblings include
`VypicujMiToyotu` and `HoniMiHoNekdoZKanclu`). It records what kind of driving the record does, and
it is the field the police squad polls each tick to decide whether to re-issue the chase order. It
is distinct from `action_status` (`+0x20`).

| Value | Meaning | Evidence |
|---|---|---|
| 0 | no record for this vehicle | the getter's miss default |
| 1 | hunt a target actor | `AddCarHunt` with a target; the gate of `pursuit_event_record_is_chase_tracking_by_vehicle` |
| 2 | drive to a set position | `AddCarHunt` with `target == 0` |
| 3 | flee a target | `pursuit_event_dispatch_escape_vehicle` |
| 4 | scripted move to a position | `pursuit_event_dispatch_move_to_vehicle` |

**The mode is stamped at build time, never changed by the tick.** No `pursuit_event` method writes it
after the record is built. `pursuit_event_remove_vehicle` restores model level 0 when the mode is 3 or
4 even if the boost flag is clear, while a hunt (1 / 2) defers to the flag.

**The consumer** is the police squad tick: it reads the mode of its chase car and, when it is not 1,
corrects the chase: it sets a car flag, toggles `pursuit_event_set_change_tables`, then removes and
re-dispatches the car with a fresh event id chosen from the squad's own state.

### 4a. The chase-tracking predicate

`pursuit_event_record_is_chase_tracking_by_vehicle` returns the mode timer `+0x28` only while
`mode == 1`, `event_flags` bit `0x4` is clear and `chase_object` is not 0, so it is non-zero while an
actor hunt runs its timer. Its one caller, `police_action_tick_sighting_timers`, skips the squad's
sighting refresh while it is non-zero.

### 4b. `event_flags` — the record's bits (`+0x1c`)

The tick's state machine runs on the phase bits: a hunt moves `0x1` -> `0x2` -> `0x4` and back as the
distance to its object changes.

| Bit | Meaning | Set by | Tested by |
|---|---|---|---|
| `0x1` | road approach: the car follows a path built on the road graph | `pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_tick` | `pursuit_event_tick` |
| `0x2` | direct follow on a breadcrumb path | `pursuit_event_tick` | `pursuit_event_tick`, `pursuit_event_save_game_load` (`& 0x26` skips the lane re-link) |
| `0x4` | close range | `pursuit_event_tick` | `pursuit_event_tick`, `pursuit_event_record_is_chase_tracking_by_vehicle` (clear), `pursuit_event_save_game_load` (`& 0x26`) |
| `0x10` | escape | `pursuit_event_dispatch_escape_vehicle` | `pursuit_event_tick` |
| `0x20` | move-to on a straight two-node route | `pursuit_event_dispatch_move_to_vehicle` | `pursuit_event_tick`, `pursuit_event_save_game_load` (`& 0x26`) |
| `0x40` | move-to on a routed path | `pursuit_event_dispatch_move_to_vehicle` | `pursuit_event_tick` |
| `0x100000` | stalled | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x1000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x2000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x8000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x20000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x40000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |
| `0x80000000` | side-by-side state | `pursuit_event_tick` | `pursuit_event_tick` |

`pursuit_event_dispatch_pursuit_vehicle` also copies its argument's bits `0x1f00` and `0x200000`, and
`pursuit_event_dispatch_move_to_vehicle` its argument's `0x200000` (Open item 1).

## 5. Entry points, savegame and the boost flag

### 5a. The three public entry points

| PC | Developer name | The argument that distinguishes it |
|---|---|---|
| `pursuit_event_dispatch_pursuit_vehicle` | `AddCarHunt` | a `C_actor *` to hunt (or a position) |
| `pursuit_event_dispatch_escape_vehicle` | `AddCarEscape` | a `C_actor *` to flee |
| `pursuit_event_dispatch_move_to_vehicle` | `AddCarMoveTo` | no actor: a destination `S_vector` |

All three share one callee spine: `vehicle_set_engine_on` → `operator_new` → `driver_ctor` →
`driver_set_car` → record insert → `roads_get_near_node` ×2 → … → `pursuit_event_prepare_path` →
`pursuit_event_recompile_path`. The script side reaches them at `entity_process_command` cases
`0x100c` / `0x100d` / `0x100e`.

### 5b. The savegame chunk

`pursuit_event_save_game_get_size` / `_save` / `_load` are a closed triple. The chunk tag is **6**;
each surviving record costs **`0xb8` bytes** plus `0x70` per driver path node, after a 4-byte header.
A record survives an eight-clause guard: the car is not save-excluded (`actor + 0x65`,
`structs/actor.md` §2) and its `+0x1c` is not 2, and none of the target vehicle, the target and the
driver's four tracked actors (`+0x1a0`, `+0x1a4`, `+0x1fc`, `+0x200`) is save-excluded. The saver
stores the actor ids, record `+0x1c..+0x2c`, `+0x38` and `+0x3c` (save-record dwords `0x2c`, `0x2d`)
and the driver state, and not `+0x10`, `+0x30`, `+0x34` or `+0x40`; `_load` removes every
live record through `pursuit_event_remove_vehicle` before rebuilding.

### 5c. The boost flag (`g_pursuit_change_tables`)

`pursuit_event_set_change_tables(bool)` writes the file-static byte at **`0x0064C6D8`**. When it is
set, `AddCarHunt` switches the chase car to its **top model level** (`car_set_model_level(count − 1)`),
which re-applies that variant's CARINIT physics table: faster and better handling, "boosting".
`RemoveCar` reverts to level 0. Only cars with more than one model level can be boosted; the level
count comes from the car's record in `tables/vehicles.bin` (14 cars carry a second, boosted CARINIT
block). Full mechanism, toggle map and boostable-car list: `police-system/artifacts/boosting.md`.

## 6. The translation unit

The range `[0x004ec180, 0x004f60b0)` holds **53 PC function starts**, bounded by the CRT
static-init / atexit pairs `0x004ec180` / `0x004ec1a0` and `0x004f60b0` / `0x004f60d0`. None is a
phantom start. The 21 class functions with a body of their own are §3c plus the constructor; the
destructor is the shared body of §3a, outside the unit.

**Thirteen header inlines of other classes** sit in the range and keep their own prefix:
`vehicle_get_engine_on`, `vehicle_set_automatic_gearbox`, `vehicle_set_horn`, `vehicle_get_siren`,
`vehicle_is_throttle_applied`, `vehicle_get_up_axis`, `vehicle_get_wheel_count`,
`vehicle_get_wheel_at` (all `C_Vehicle`), `car_set_condition_channel` (`C_car`),
`actor_is_death_processed` and `actor_is_in_camera_sector` (`C_actor`), `driver_path_node_at`
(`C_driver`) and `i3d_frame_get_world_matrix_ptr` (shared). Inside the unit, `pursuit_event_tick`
calls twelve of them.

**How the `vehicle_` / `car_` receiver split is proved:**

1. The same caller distinguishes the two receivers. Inside `pursuit_event_tick`, `vehicle_set_power`
   is reached as `mov ecx,[esi]; add ecx,0x70`, while `car_get_condition_fraction` and
   `car_get_owner` are reached as `mov ecx,[ebp]` with no `+0x70`. The `+0x70` step is the compiler's
   own discriminator.
2. Eight of the nine vehicle / car inlines carry that step; `car_set_condition_channel` does not.
3. A field-offset census over the whole binary agrees: `+0xca8`, `+0x548`, `+0x540`, `+0xcb4`,
   `+0xcd0`, `+0x5bc` / `+0x5c0` and `+0x4ae` are read only by named `vehicle_*` functions, while
   `+0x20b8` is read only by `car_get_condition_fraction`. `box_game_init` /
   `box_write_state_to_frame` / `box_apply_net_event_msg` also touch `+0xcb4` and `+0xcd0`; those are
   `C_box` at equal offsets, not evidence for either receiver.

## 7. Cheat Engine

- **Boost flag:** 1 byte at `Game.exe+0x24C6D8`, absolute `0x0064C6D8`. 1 = a hunt boosts, 0 = not.
- **Chase record N** (only while a chase is active): base `[[[0x0065115C]+0x24]+0x2B18+0x14]`;
  record N at `base + N*0x44`. Useful fields: `+0x00` chase car, `+0x0C` target, `+0x20`
  `action_status`, `+0x24` `mode` (§4).

## 8. PC vs PS2

**Class name.** The PS2 class is `C_sledovacka` (*sledovacka* = a chase / tail); the PC
class is named `pursuit_event` after its function prefix.

**Record layout.** PC `S_ELEMENT` is stride `0x44`, PS2 `0x50`: the PS2 record carries an extra
`+0x1c` position word, so every field from `event_flags` on sits 4 higher on the PS2 (`event_flags`
`+0x20`, `action_status` `+0x24`, `mode` `+0x28`, `mode_timer_ms` `+0x2c`, `cadence_timer_ms`
`+0x30`). The PS2 `Tick` runs the mode timer with the same 2000 ms and 15000 ms thresholds. The
savegame record follows: `0xb8` on the PC, `0xbc` on the PS2; the driver path node is `0x70` on the
PC and `0x90` on the PS2.

**`GetVoleMode` ↔ `+0x24`, `DevaaateeeZasNeznaCas` ↔ `+0x28`.** PS2 `+0x28` (VoleMode) is PC `+0x24`.
The PC `+0x28` accessor reads PS2 `+0x2c`, and its twin is `DevaaateeeZasNeznaCas__12C_sledovacka`
(byte-exact).

**Mode writers on PS2:** `AddCarEscape` sets the mode = 3 and `AddCarMoveTo` = 4. The PS2
`ai_police_action` tick reads `GetVoleMode` and branches on `!=1 / ==0 / {2,3,4}`.

**`C_driver` twin:** PS2 `__ct__8C_driver` after `__nw__(0x320)` is PC `driver_ctor` after
`operator_new(0x29C)`, settled by the `AddCarEscape` body-diff.

**Size-ratio band.** This unit runs 0.48–0.71 (PC ÷ PS2). `AddCarEscape` 0.481 and `AddCarMoveTo`
0.475 are the two extremes and they agree with each other.

**The destructor.** PS2 `__dt__12C_sledovackaFv` frees the `gif::vector<S_ELEMENT>` at `+0x0c` and
then the `std::vector<S_BRIDGE>` at `+0x00`. The PC body `folded_vector_pair_free_at_0x04_0x14` frees
the same two vectors in the same order, at `+0x14` and `+0x04`; two different Standard Libraries, so
the twin is STRUCTURAL.

**Proven PC inlinings**, each the only PS2 call missing at its mirrored position while everything
around it matches:

| PS2 symbol | Folded into (PC) |
|---|---|
| `__as__Q212C_sledovacka9S_ELEMENT` (operator=) | the erase-shift loop in `pursuit_event_remove_vehicle` |
| `__ct__Q212C_sledovacka9S_ELEMENT` (copy ctor) | `std_vector44_insert_one` |
| `reserve__Q23gif36vector<S_ELEMENT,8>` | `std_vector44_insert_one`'s grow path |
| `__ct__10S_car_nodeFRC10S_car_node` | `stl_copy_range_stride70` |

**`RemoveCar`'s vehicle calls.** PS2 `RemoveCar` calls `SetDynamicCollsOn__9C_Vehicle(1, …)` and
then `SetClutch__9C_Vehicle(1.0)`. The PC calls `vehicle_set_dynamic_colls_off(0)` at the same
position (the PC parameter is the inverse of the PS2's `on`), and has no clutch call and no inline
write to the clutch field `C_Vehicle + 0x65c`: the PC leaves the clutch where the driver left it.

**`AddCarEscape` lockstep:** PS2 `GetAIPos__5C_carFb` sits at the position where the PC calls
`roads_find_path`, a positional coincidence, not a twin.

## 9. Related

- `police-system/artifacts/boosting.md` — the boost mechanism this system implements.
- `structs/police-board.md` / `structs/police-action.md` — the police squads that call `AddCarHunt` /
  `RemoveCar`.
- `structs/driver.md` — `C_driver`, the per-record driver and its path nodes.
- `structs/roads.md` — the road graph the paths are built on.
- `structs/ai_fight_manager.md` — the fight-party manager (not part of this class).
- `structs/force_feedback.md` — the other owner of the destructor body.
- `structs/game.md` — the owning `C_game` (`+0x2b18`) and the traffic-light clock.

## 10. Open items

1. **`event_flags` bits `0x100`..`0x1000` and `0x200000`**, copied from the dispatchers' arguments
   (§4b): their testers and meanings are not mapped.
2. **Run-time check:** none of this has been validated in a running game; §7 gives the addresses.
