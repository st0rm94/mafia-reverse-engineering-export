# `C_bridge` — the city's two drawbridges

**Created** 2026-08-06 14:15 +0200 · **Modified** 2026-11-09 18:20 +0100

The city's two lifting road bridges: the **West Marshall Bridge** (Central Island - Little Italy) and the **East
Marshall Bridge** (Central Island - Downtown). Each is one `C_bridge` entity that drives both leaves of its deck: the
frame it is bound to and a second leaf it finds at `Init` time. The two instances share a global bank (§4), and the East
Marshall instance also runs the steamboats whose approach makes either bridge rise (§4a, §4b).

| | |
|---|---|
| PC prefix | `bridge_` |
| PC kind (`+0x10`) | `0x14` (`structs/entity_kind.md`) |
| PC vtable | `g_bridge_vt` @ `0x0063bb40` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x194` (the last field ends there; `operator_new` in `mission_create_actor`) |
| PC constructor | `bridge_ctor` @ `0x0045f980` |
| PC translation unit | `[0x0045f930, 0x00462a90)`, guard to guard: 23 starts, all named |
| PS2 class | `C_bridge` (tag `8C_bridge`), 16 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

One consumer outside the class: `human_find_blocking_drawbridge` refuses to let the player leave a car when a `C_bridge`
in state 3-5 is within 100 units, exactly the raising / up / lowering states of §2a.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base. The class also uses the kind `+0x10` and the frame
`+0x68`, the first leaf.

```c
typedef struct C_bridge {                     // C_actor base +0x00..+0x70
    /* +0x70 */ uint8_t  panels_a_alloc;      // [H] allocator byte of the first leaf's panels
    /* +0x74 */ S_BridgeCol *panels_a_begin;  // [H] first panel of the first leaf
    /* +0x78 */ S_BridgeCol *panels_a_end;    // [H] one past its last panel
    /* +0x7c */ S_BridgeCol *panels_a_cap;    // [H] end of its storage
    /* +0x80 */ uint8_t  panels_b_alloc;      // [H] allocator byte of the second leaf's panels
    /* +0x84 */ S_BridgeCol *panels_b_begin;  // [H] first panel of the second leaf
    /* +0x88 */ S_BridgeCol *panels_b_end;    // [H] one past its last panel
    /* +0x8c */ S_BridgeCol *panels_b_cap;    // [H] end of its storage
    /* +0x90 */ I3D_frame *second_leaf;       // [H] the deck's other leaf
    /* +0x94 */ I3D_sound *machinery_sound;   // [H] the emitter for the deck's motion sounds
    /* +0x98 */ I3D_sound *siren_sound;       // [H] the emitter for the siren
    /* +0x9c */ I3D_frame *lights[6];         // [H] the warning-light frames
    /* +0xb4 */ tDynamicCollObject *dyn_coll; // [H] collision record, in the world while cycling
    /* +0xb8 */ S_matrix dyn_coll_world;      // [H] the record's world matrix
    /* +0xf8 */ S_matrix dyn_coll_inverse;    // [H] the record's inverse matrix
    /* +0x138 */ int32_t cycle_marker;        // [H] 1 during a raise cycle, 7 outside; no reader (.text scan)
    /* +0x13c */ uint8_t frozen;              // [H] holds or ends the cycle (§2a)
    /* +0x140 */ int32_t bridge_index;        // [H] 0 or 1, the slot in the shared bank
    /* +0x144 */ int32_t state;               // [H] the six-state cycle (§2a)
    /* +0x148 */ float   angle;               // [H] current deck angle, radians
    /* +0x14c */ float   angle_applied;       // [H] last angle written to the frames
    /* +0x150 */ int32_t phase_timer_ms;      // [H] counts down within the current state
    /* +0x154 */ int32_t blink_timer_ms;      // [H] counts down to the next light toggle
    /* +0x158 */ uint8_t blink_on;            // [H] which light group shows
    /* +0x15c */ uint32_t hold_ms;            // [H] bridge_ai returns while non-zero; no non-zero writer (.text scan)
    /* +0x160 */ int32_t idle_period_ms;      // [H] the wait after a cycle before state 0 may trigger
    /* +0x164 */ int32_t warn_ms;             // [H] duration of the warning state
    /* +0x168 */ int32_t raise_ms;            // [H] duration of the raise ramp
    /* +0x16c */ int32_t up_dwell_ms;         // [H] minimum time held up
    /* +0x170 */ int32_t lower_ms;            // [H] duration of the lowering ramp
    /* +0x174 */ uint8_t reverse_direction;   // [H] which leaf's angle bridge_ai negates; only the ctor's 0 (.text scan)
    /* +0x178 */ float   angle_max;           // [H] the raised angle, radians
    /* +0x17c */ int32_t sound_id_raise;      // [H] sound played as the raise ramp starts
    /* +0x180 */ int32_t sound_id_raised;     // [H] sound played when fully raised
    /* +0x184 */ int32_t sound_id_lower;      // [H] sound played as the lowering starts
    /* +0x188 */ int32_t sound_id_lowered;    // [H] sound played when fully down
    /* +0x18c */ int32_t sound_id_clear;      // [H] sound played when the deck is clear of cars
    /* +0x190 */ int32_t sound_id_siren;      // [H] sound played by the siren
} C_bridge;
```

**Why `lights` has six entries.** `bridge_game_init` looks up six named frames, by the leaf's first letter:
`watercity_RED11`, `10`, `8`, `9`, `7`, `12` for a name starting with `L`, `watercity_RED5`, `4`, `2`, `3`, `1`, `6`
otherwise. Each may be absent, `bridge_update` tests every pointer. They blink in two groups: entries 0..2 show while
`blink_on` is clear and entries 3..5 while it is set, so the two groups alternate every 600 ms while a cycle runs; state 0
turns all six off once `blink_timer_ms` runs out.

**The durations.** `bridge_ctor` seeds them and `bridge_game_init` clamps four of them (`idle_period_ms`, `up_dwell_ms`,
`lower_ms`, `raise_ms`) to a minimum of 1 ms. No other function writes any of the five (a census of every store at those
displacements in `.text`), so the shipped values are the ctor's.

### 1a. `S_BridgeCol` — one deck collision panel, `0x84` bytes

Two vectors, one per leaf, filled by `bridge_game_init` from `collision_find_hits_for_frame` and re-transformed by
`bridge_update` every tick. That is how the road surface keeps colliding correctly while it tilts.

```c
typedef struct S_BridgeCol {                  // 0x84 bytes, the vector's stride
    /* +0x00 */ void    *record;              // [H] the static collision record of this panel
    /* +0x04 */ S_matrix local_matrix;        // [H] the record relative to its leaf
    /* +0x44 */ S_matrix local_inverse;       // [H] the inverse of that matrix
} S_BridgeCol;
```

## 2. Field evidence

Measured by a register-tag census of the class's functions and by reading `bridge_ctor`, `bridge_init`, `bridge_ai`,
`bridge_game_done`, `bridge_save_game_save`, `bridge_shut_down` and the field traffic of `bridge_update` and
`bridge_game_init`. A panel is reached through a walking pointer, so its cells come from the bodies; `bridge_update`'s
matrix arguments are read from its disassembly at `0x00460294..0x004602d3`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | panels_a_alloc | H | `bridge_ctor` | - | the allocator byte |
| +0x74 | panels_a_begin | H | `bridge_ctor`, `bridge_dtor` | `bridge_dtor`, `bridge_game_init`, `bridge_update` | stride `0x84` |
| +0x78 | panels_a_end | H | `bridge_ctor`, `bridge_dtor` | `bridge_game_init`, `bridge_update` | - |
| +0x7c | panels_a_cap | H | `bridge_ctor`, `bridge_dtor` | - | `bridge_col_vector_insert_n` reaches it through `+0x70` |
| +0x80 | panels_b_alloc | H | `bridge_ctor` | - | the allocator byte |
| +0x84 | panels_b_begin | H | `bridge_ctor`, `bridge_dtor` | `bridge_dtor`, `bridge_game_init`, `bridge_update` | stride `0x84` |
| +0x88 | panels_b_end | H | `bridge_ctor`, `bridge_dtor` | `bridge_game_init`, `bridge_update` | - |
| +0x8c | panels_b_cap | H | `bridge_ctor`, `bridge_dtor` | - | as `panels_a_cap` |
| +0x90 | second_leaf | H | `bridge_ctor`, `bridge_init` | `bridge_ai`, `bridge_game_init`, `bridge_update` | `bridge_init` finds it by name |
| +0x94 | machinery_sound | H | `bridge_ctor`, `bridge_game_done`, `bridge_game_init` | `bridge_ai`, `bridge_game_done`, `bridge_game_init` | placed at the midpoint of the two leaves; range 20..50 |
| +0x98 | siren_sound | H | `bridge_game_done`, `bridge_game_init` | `bridge_ai`, `bridge_game_done`, `bridge_game_init` | placed at the same midpoint |
| +0x9c | lights | H | `bridge_game_init` | `bridge_update` | §1 |
| +0xb4 | dyn_coll | H | `bridge_game_init` | `bridge_ai`, `bridge_game_done`, `bridge_game_init`, `bridge_shut_down` | `operator_new(0x48)`, type `0x40`, flags `0x200000`, owner this, over the frame `watercity_nadparnik1` or `2`; inserted while the bridge cycles (§2a) |
| +0xb8 | dyn_coll_world | H | `bridge_game_init` | `bridge_game_init` | `dyn_coll+0x3c` points at it; `bridge_game_init` copies that frame's world matrix into it, 16 dwords by `rep movsd` |
| +0xf8 | dyn_coll_inverse | H | `bridge_game_init` | `bridge_game_init` | `dyn_coll+0x40` points at it; `S_matrix::Inverse` |
| +0x138 | cycle_marker | H | `bridge_ai`, `bridge_ctor`, `bridge_game_init` | - | 7 from the ctor, `bridge_game_init` and the end of a cycle, 1 at its start; no reader (.text scan: stores only, in the ctor, `bridge_game_init` and twice in `bridge_ai`; no load) |
| +0x13c | frozen | H | `bridge_ctor`, `bridge_game_init`, `bridge_shut_down` | `bridge_ai` | §2a |
| +0x140 | bridge_index | H | `bridge_init` | `bridge_ai`, `bridge_game_done`, `bridge_game_init`, `bridge_init`, `bridge_set_steamboat_figures`, `bridge_update` | 0 or 1, indexes the bank (§4) |
| +0x144 | state | H | `bridge_ai`, `bridge_game_init` | `bridge_ai`, `bridge_save_game_save`, `bridge_update`, `human_find_blocking_drawbridge` | §2a; also read by `human_find_blocking_drawbridge` |
| +0x148 | angle | H | `bridge_ai`, `bridge_game_init` | `bridge_ai`, `bridge_game_init` | radians |
| +0x14c | angle_applied | H | `bridge_ai`, `bridge_game_init` | `bridge_ai` | -1.0 from `bridge_game_init`, so the first tick writes the frames |
| +0x150 | phase_timer_ms | H | `bridge_ai`, `bridge_game_init`, `bridge_init` | `bridge_ai`, `bridge_save_game_save` | `bridge_game_init` sets it to `idle_period_ms` |
| +0x154 | blink_timer_ms | H | `bridge_ai`, `bridge_game_init`, `bridge_init`, `bridge_update` | `bridge_update` | 1 from `bridge_game_init`, 600 entering state 1; re-armed by `bridge_update` until state 0, which turns the lights off |
| +0x158 | blink_on | H | `bridge_game_init`, `bridge_update` | `bridge_update` | toggled by `bridge_update` |
| +0x15c | hold_ms | H | `bridge_ai`, `bridge_game_init` | `bridge_ai`, `bridge_update` | `bridge_ai` returns while it is non-zero and `bridge_update` holds both boats on it; **no non-zero writer** (.text scan): `bridge_game_init` stores 0, `bridge_ai` only decrements, so both arms never run as shipped; the in-game half is Open item 1 |
| +0x160 | idle_period_ms | H | `bridge_ctor`, `bridge_game_init` | `bridge_ai`, `bridge_game_init` | ctor 120000; `bridge_game_init` clamps a zero to 1 |
| +0x164 | warn_ms | H | `bridge_ctor` | `bridge_ai` | ctor 4000; not clamped |
| +0x168 | raise_ms | H | `bridge_ctor`, `bridge_game_init` | `bridge_ai`, `bridge_game_init` | ctor 30000; clamped |
| +0x16c | up_dwell_ms | H | `bridge_ctor`, `bridge_game_init` | `bridge_ai`, `bridge_game_init` | ctor 10000; clamped |
| +0x170 | lower_ms | H | `bridge_ctor`, `bridge_game_init` | `bridge_ai`, `bridge_game_init` | ctor 20000; clamped |
| +0x174 | reverse_direction | H | `bridge_ctor` | `bridge_ai` | selects which leaf's angle `bridge_ai` negates; the ctor's 0 is the only store (.text scan), so `bridge_ai` always negates the first leaf's angle and the other arm never runs as shipped; the in-game half is Open item 1 |
| +0x178 | angle_max | H | `bridge_ctor` | `bridge_ai` | `0x3f5f66f3` = 0.8727 rad = 50 degrees; the ctor's store is the only one |
| +0x17c | sound_id_raise | H | `bridge_ctor` | `bridge_ai` | `0x71`, played entering state 3 |
| +0x180 | sound_id_raised | H | `bridge_ctor` | `bridge_ai` | `0x72`, played entering state 4 |
| +0x184 | sound_id_lower | H | `bridge_ctor` | `bridge_ai` | `0x73`, played entering state 5 |
| +0x188 | sound_id_lowered | H | `bridge_ctor` | `bridge_ai` | `0x74`, played entering state 0 |
| +0x18c | sound_id_clear | H | `bridge_ctor` | `bridge_ai` | `0xb6`, played entering state 2 |
| +0x190 | sound_id_siren | H | `bridge_ctor` | `bridge_ai` | `0xb5`, played on the siren emitter entering state 1 |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | record | H | `bridge_game_init` | `bridge_update` | from `collision_find_hits_for_frame`; its type byte picks the matrix offsets |
| +0x4 | local_matrix | H | `bridge_game_init` | `bridge_update` | times the leaf's world matrix gives the record's world matrix: `record+0x38` for type `0x80`, `+0x20` for type `0x83` |
| +0x44 | local_inverse | H | `bridge_game_init` | `bridge_update` | the leaf's inverse times it gives the record's inverse: `record+0x78` for type `0x80`, `+0x60` for type `0x83` |

### 2a. `state` — the cycle

| Value | Meaning | Evidence |
|---|---|---|
| 0 | idle: on `g_bridge_raise_request_mask[idx]` non-zero and not `frozen`, arm `warn_ms`, blink 600 ms, play `sound_id_siren` on the siren emitter, `cycle_marker` 1, go to 1 | `bridge_ai` default arm |
| 1 | warn: once the timer is below 0, with no car on the deck (`traffic_car_get_bridge_car_count` < 1) and not `frozen`, play `sound_id_clear`, go to 2; otherwise re-check in 1000 ms | `bridge_ai` case 1 |
| 2 | wait: once the machinery emitter reports idle, or `frozen`, arm `raise_ms`, play `sound_id_raise`, `collision_dyn_insert(dyn_coll)`, go to 3 | case 2 |
| 3 | raising: ramps `angle` 0 → `angle_max`; on expiry or `frozen`, arm `up_dwell_ms`, play `sound_id_raised`, go to 4 | case 3 |
| 4 | up: publishes `g_bridge_is_up[idx]` = 1; on expiry or `frozen`, stays while `g_bridge_raise_request_mask[idx]` is non-zero, otherwise arm `lower_ms`, play `sound_id_lower`, go to 5 | case 4 |
| 5 | lowering: ramps `angle` → 0; on expiry or `frozen`, arm `idle_period_ms`, play `sound_id_lowered`, silence the siren, `cycle_marker` 7, `collision_dyn_erase(dyn_coll)`, go to 0 | case 5 |

**`frozen` does not end every state.** States 2..5 take their exit branch at once, and states 0 and 1 refuse to advance: a
frozen state 0 stops counting and ignores requests, and a frozen state 1 re-arms 1000 ms for ever. So a frozen bridge runs
down to the flat deck and stays there. `g_bridge_is_up` is 0 in every state but 4.

**The collision record is in the world only during a cycle** (inserted entering state 3, erased leaving state 5). While the
deck is flat it is ordinary static world geometry. `traffic_car_get_bridge_car_count` (`0x00506760`), state 1's gate, has
this one caller and is a `C_traffic_car` static.

## 3. Method family

16 of 16 PS2 methods mapped: 11 with a PC function, 5 on shared base stubs (the bodies are in §6f); the PC-only rows are
the unit's CRT rows, the slot-16 destructor, `bridge_set_steamboat_figures` and the two container helpers. Every override of
the `C_actor` vtable has a row: slots 13–16, 18–26 and 30.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0045f930 | `crt_static_init_guard_0045f930` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0045f950 | `crt_atexit_register_0045f950` | the translation unit | - | - | - | registers the unit's no-op static destructor |
| 0x0045f960 | `thunk_stub_noop_cdecl_0045f970` | the translation unit | - | - | - | a jump to `stub_noop_cdecl_0045f970` |
| 0x0045f980 | `bridge_ctor` | instance | `__ct__8C_bridgeFv` | 0x00125cb0 | CONFIRMED | `actor_ctor`, empties both panel vectors, vtable, kind `0x14`, seeds the durations, the limit and the sound ids; its one callsite is `mission_create_actor` |
| 0x0045fa70 | `bridge_scalar_deleting_dtor` | instance | - | - | STRUCTURAL | slot 16; an MSVC-only construct |
| 0x0045fa90 | `bridge_dtor` | instance | `__dt__8C_bridgeFv` | 0x00125d90 | CONFIRMED | sets `g_bridge_first`, frees both panel vectors, `actor_dtor` |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0045fae0 | `bridge_init` | `Init__8C_bridgeFP9I3D_frame` | 0x00125e50 | CONFIRMED | slot 18; `actor_init`, finds the second leaf, assigns `bridge_index` (§4) |
| 0x0045fc20 | `bridge_ai` | `AI__8C_bridgeFUi` | 0x00125fb0 | CONFIRMED | slot 13; the six-state cycle (§2a), then writes a changed angle to both leaves |
| 0x004601b0 | `bridge_update` | `Update__8C_bridgeFUi` | 0x00126670 | CONFIRMED | slot 14; re-transforms the panels, blinks the lights (§1), and on the index-1 bridge runs the steamboats (§4b) |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__8C_bridgeFUi` | 0x00127dc0 | base-stub | slot 15; a shared stub with 30 vtable users |
| 0x00461190 | `bridge_game_init` | `GameInit__8C_bridgeFv` | 0x00127dd0 | CONFIRMED | slot 25; seeds the cycle, clamps four durations, builds panels, emitters, lights and collision record; on the index-1 bridge builds the boats and warms up (§4c) |
| 0x00462280 | `bridge_game_done` | `GameDone__8C_bridgeFv` | 0x00129960 | CONFIRMED | slot 26; erases and frees the collision record, `actor_game_done`, releases the emitters and on the index-1 bridge the boats' frames and figures |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveData__8C_bridgeFP7C_chunk` | 0x00129a80 | base-stub | slot 19; a shared stub |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameLoad__8C_bridgeFPPv` | 0x00129be0 | base-stub | slot 24; the same stub |
| 0x00462340 | `bridge_load_data` | `LoadData__8C_bridgeFP7C_chunk` | 0x00129a90 | CONFIRMED | slot 20; loads `tables\parnik.def`, not bridge data (§5) |
| 0x00462420 | `stub_ret0_cdecl_00462420` | `SaveGameGetSize__8C_bridgeFv` | 0x00129bb0 | base-stub | slot 22; a shared stub |
| 0x00462430 | `bridge_save_game_save` | `SaveGameSave__8C_bridgeFPPv` | 0x00129bc0 | CONFIRMED | slot 23; stores `state` and `phase_timer_ms` |
| 0x00462450 | `bridge_shut_down` | `ShutDown__8C_bridgeFb` | 0x00129bf0 | CONFIRMED | not a slot; stores `frozen` and erases the collision record; one callsite, `program_process_command` |
| 0x00462480 | `bridge_collision` | `Collision__8C_bridgeFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00129c20 | CONFIRMED | slot 30; for a kind-2 actor clears its `+0x644` and calls its slot 31 (`Hit`) with hit type 6 |
| 0x004624e0 | `bridge_set_steamboat_figures` | - | - | STRUCTURAL | the PARNIK cheat's figures (§5a); PC-only, the PS2 has no cheat system |
| 0x00462730 | `bridge_col_vector_insert_n` | - | - | STRUCTURAL | `_Insert_n` of the `S_BridgeCol` vector (§6c); 2 callsites, both in `bridge_game_init` |
| 0x00462a60 | `bridge_col_vector_copy` | - | - | STRUCTURAL | `std::copy` of the same (§6c); 2 callsites, both in `bridge_game_init` |
| 0x004776f0 | `stub_noop_args4_004776f0` | `DuplicateFrom__8C_bridgeFP7C_actor` | 0x00129cb0 | base-stub | slot 21; a shared stub outside the unit |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0045f970 | `stub_noop_cdecl_0045f970` | - | - | - | the unit's no-op, a shared stub and not a class member |
| 0x004626b0 | `string_assign_cstr` | - | - | - | a `C_string` method emitted in this unit; `bridge_game_init` uses it for the boats' model names |

## 4. The shared global bank

Two `C_bridge` instances exist, one per drawbridge, and they coordinate through file-scope globals rather than pointers to
each other. `bridge_init` finds each bridge's second leaf by incrementing the last character of its frame name and assigns
the index: **frame name starting with `'L'` means index 0** (the West Marshall Bridge, leaves `LLsklap01` / `LLsklap02`),
**otherwise index 1** (the East Marshall Bridge, `sklapx01` / `sklapx02`); the index-1 instance clears the whole bank. Which
frame is which bridge is the user's identification; the steamboat routes of §4b agree, as index 0 is requested only by routes
0 and 1 and index 1 only by routes 2 and 3, at crossing points about 870 units apart. Every named global below is used only
by `bridge_*` functions.

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_bridge_first` | `0x0064b5ec` | a byte set to 1 | `bridge_dtor`, `bridge_game_done` | - (a byte scan of `Game.exe` for the address finds these two stores and no load) |
| `g_steamboat_shared_section_claim` | `0x00655730` | int[3]: `[0]` the boat (index + 1) in the channel under Terranova Bridge; `[1]` and `[2]` only ever cleared (§4b) | `bridge_game_init`, `bridge_init`, `bridge_update` | `bridge_update` |
| `g_bridge_passage_claim` | `0x0065573c` | int[2]: the boat (index + 1) in the channel under drawbridge `i` (§4b) | `bridge_game_init`, `bridge_init`, `bridge_update` | `bridge_update` |
| `g_bridge_is_up` | `0x00655744` | int[2]: 1 in state 4, otherwise 0 | `bridge_ai`, `bridge_game_init`, `bridge_init` | `bridge_update` |
| `g_bridge_raise_request_mask` | `0x0065574c` | int[2]: a bitmask of requesting steamboats, not a flag | `bridge_game_init`, `bridge_init`, `bridge_update` | `bridge_ai`, `bridge_update` |
| `g_steamboat_slots` | `0x00655758` | record[2], stride `0x24` (§4a) | `bridge_game_init`, `bridge_init`, `bridge_update` | `bridge_update` |
| `g_steamboat_segment_time` | `0x006557a0` | float[28], per-segment traversal time, indexed `route*7 + segment` | `bridge_game_init` | `bridge_update` |
| `g_steamboat_nodes` | `0x00655810` | node[4][7], stride `0x18`, the route table (§5) | `bridge_game_init`, `bridge_load_data` | `bridge_game_init`, `bridge_update` |

`0x006557a0 + 28*4 == 0x00655810` exactly: the segment-time table and the node table are contiguous, and that arithmetic
fixes both element counts. The counts are the routes' shape: 4 routes of 7 nodes. `g_steamboat_shared_section_claim` holds
three ints because that is the room up to `g_bridge_passage_claim`.

### 4a. The steamboat slot record (stride `0x24`, 2 entries)

| Offset | Global | Meaning |
|---|---|---|
| `+0x00` | `g_steamboat_slots` | route id; seeded `0xffff` by `bridge_init`; the `switch` subject in `bridge_update`, and `id * 7` indexes `g_steamboat_nodes` |
| `+0x04` | `g_steamboat_slot_segment` | current segment index, 0..5 (6 segments over 7 nodes); seeded by `rand_range` |
| `+0x08` | `g_steamboat_slot_segment_prev` | written from `+0x04` each tick |
| `+0x0c` | `g_steamboat_slot_t` | float, 0..1 along the current segment |
| `+0x10` | `g_steamboat_slot_frame` | the boat's `I3D_frame`; parent of the figures, released by `bridge_game_done` |
| `+0x14` | `g_steamboat_slot_speed_scale` | float, 5.0 at full speed; a held boat loses the elapsed time from it each tick, down to 0 |
| `+0x18` | `g_steamboat_slot_figure_a` | `Tommy.i3d` / `Leonardo`, §5a |
| `+0x1c` | `g_steamboat_slot_figure_b` | `czena06.i3d` / `Kate`, §5a |
| `+0x20` | `g_steamboat_slot_sound` | the boat's sound object: `bridge_game_init` creates it (`sound_table_construct_sound(..., 0x146)`, range 20..50); `bridge_update` drives it |

The two boats keep to their own routes: slot 0 starts on route 0 and slot 1 on route 2, and when a boat passes segment 5 it
draws a new route with `rand_range`, adding 2 for slot 1.

### 4b. The raise windows and the channel claims

**The writer of the raise request is `bridge_update`.** Its second half, gated `bridge_index != 0`, **is the steamboat
simulation**: it snapshots both mask elements, clears them, then per boat advances it along its route and sets
`1 << slot_index` in the mask of the drawbridge ahead. So **the bridge raises itself because a boat is approaching**, and the
whole mechanism is inside this translation unit.

A boat's position is `segment + t` along its route (7 nodes, so 0.0 .. 6.0). Each tick `bridge_update` first drops every claim
the boat holds, then, per route:

| Route | Raises | Raise-request window | `g_bridge_passage_claim` window | Terranova claim window |
|---|---|---|---|---|
| 0 | index 0 (West Marshall) | 0.72 .. 1.3 | `[0]` 0.6 .. 2.0 | 4.0 .. 5.0 |
| 1 | index 0 (West Marshall) | 4.0 .. 5.2 | `[0]` 4.0 .. 5.3 | - |
| 2 | index 1 (East Marshall) | 4.4 .. 5.2 | `[1]` 4.2 .. 5.4 | 0.5 .. 1.5 |
| 3 | index 1 (East Marshall) | 0.3 .. 1.3 | `[1]` 0.3 .. 2.0 | 4.0 .. 5.5 |

A claim is taken only when it is free; a boat inside the first part of a window whose claim another boat holds, or whose
drawbridge has not published `g_bridge_is_up` yet near the crossing (0.92 .. 1.0, 4.79 .. 5.0, 4.85 .. 5.0, 0.75 .. 1.0 for
routes 0..3), is held back. So `g_bridge_passage_claim[i]` keeps the channel under drawbridge `i` to one boat, and
`g_steamboat_shared_section_claim[0]` does the same for a third channel shared by routes 0, 2 and 3. Mapped through the node
positions of `tables\parnik.def`, that third window lies north of both drawbridges (x about -1200, z about 450 .. 1000), which
is the channel under **Terranova Bridge** (user identification). Route 1 passes the same channel between its nodes 1 and 2 and
takes no claim.

### 4c. The warm-up

`bridge_game_init` ends, on the index-1 bridge only, with a loop of 60 passes that call its own slot 13 and slot 14 with
1000 ms. So the mission starts with a simulated minute of boats and of this bridge's cycle behind it. The index-0 bridge's
`bridge_ai` does not run in those passes.

## 5. The steamboat rides in this class's translation unit

`bridge_load_data` does not load bridge data. It loads **`tables\parnik.def`**, a `4 routes x 7 nodes x 0x18 bytes` table at
`0x00655810`, each node a position vec3 followed by a direction vec3, defaulting to `{(0,0,0), (1,0,0)}` when the file is
absent. `bridge_game_init` on the index-1 bridge opens the same file and derives `g_steamboat_segment_time` from it.

The node element type is `G_ParnikUzel` (*parnik uzel* = steamboat node), a record type declared in the same source file as
`C_bridge` (§6a). A class defined in another class's source file links with that file, the same mechanism that puts `ai_loc` /
`ai_blob` / `ai_blob_log` next to `C_human`.

### 5a. `bridge_set_steamboat_figures` — the PARNIK cheat's figures

The PARNIK cheat (`game_cheat_check` case `0xb`) decorates the steamboats with the Titanic easter egg, and it does so **through
this class**.

**Receiver, byte-proven.** `game_cheat_check` walks the `C_game` entity vector and filters on the entity kind before calling:

```
mov ecx, [edi+4]                 ; entity vector begin
mov ecx, [ecx + esi*4]           ; element
cmp dword ptr [ecx + 0x10], 0x14 ; kind == C_bridge
jne skip
push edx                         ; the cheat's toggle bit
call bridge_set_steamboat_figures
```

A kind gate is the caller stating the receiver class outright, stronger evidence than an ECX fingerprint. The body agrees: it
opens with `if (bridge_index != 0)`, so only the index-1 bridge acts.

**What it does.** For each of the 2 records in the bank, with `enable != 0` it loads `Tommy.i3d` textured **`Leonardo`** into
`record+0x18` and `czena06.i3d` textured **`Kate`** into `record+0x1c`, parents both to the steamboat frame at `record+0x10`, and
places them at local `(0, 4.7, 37.0)` and `(0, 4.7, 37.4)`: the bow, one just behind the other. With `enable == 0` it releases
both and zeroes the slots. The retextured Leonardo (DiCaprio) and Kate (Winslet) in the bow pose parody the Titanic "I'm flying"
scene; user-confirmed in game.

## 6. PC vs PS2

### 6a. Link order and the source file

PS2 emits classes alphabetically by source file, and the sequence around this class is `C_bottle → C_box_new → C_bridge →
G_ParnikUzel → C_car`. The PC `.rdata` vtable sequence carries the same order, which places this vtable's owner before any body
is read. A PS2 `__sinit_` symbol confirms `C_bridge.cpp` is compiled into the PS2 build.

`G_ParnikUzel` immediately follows `C_bridge` and has only an 8-byte `{ctor}`: a plain record type declared inside `C_bridge.cpp`,
which is why the steamboat node table loads from a `C_bridge` method on both builds (§5).

`bridge_set_steamboat_figures` has no PS2 twin: the PS2 build has no cheat system, and nothing in its `C_bridge.cpp` unit
corresponds. It is therefore graded STRUCTURAL: with no PS2 body in existence, CONFIRMED would assert a body agreement that cannot
exist.

### 6b. The field delta is a flat +8

Across the whole `+0x13c`..`+0x190` band the PS2 offset is the PC offset **+ 8**:

| what | PC | PS2 |
|---|---|---|
| `frozen` | +0x13c | +0x144 |
| `bridge_index` | +0x140 | +0x148 |
| `state` | +0x144 | +0x14c |
| `phase_timer_ms` | +0x150 | +0x158 |
| `idle_period_ms` | +0x160 | +0x168 |
| `angle_max` | +0x178 | +0x180 |

Twelve twins diffed independently all land on that delta, so they corroborate each other. The twins of `bridge_ctor` and
`bridge_game_init` seed the same values at the same relative offsets, in the same order.

**Do not rebase other offsets off it.** In the low band the bases differ: PC `+0x74` (panels A) ↔ PS2 `+0x80`; PC `+0x90` (second
leaf) ↔ PS2 `+0x98`; PC `+0xb4` (collision record) ↔ PS2 `+0xbc`.

### 6c. What `S_BridgeCol` is

The PC decomp gives a stride and an access pattern, never an element type. The PS2 symbol
`reserve__Q23gif34vector<Q28C_bridge11S_BridgeCol,8>Fi` gives it directly: **`gif::vector<C_bridge::S_BridgeCol, 8>`**. Its body
copies 9 quadwords = `0x90` per element against the PC's `0x84`, the usual `S_vector` 16-vs-12 padding, ratio 0.92.

MSVC and GCC out-lined **different members** of that template: the PS2 has `reserve`, the PC has `_Insert_n` and `std::copy`. That
is why `bridge_col_vector_insert_n` / `bridge_col_vector_copy` are graded STRUCTURAL: there is no counterpart body to diff, only a
confirmed element type plus a callsite census, found by a byte scan of `.text` for `E8` calls since Ghidra does not render them
textually.

### 6d. PS2 names this class's globals in the clear

`globalFirstBridge`, `bridgeIsUp`, `bridgeNeedUp`, `bridgeObsazen`, `mostObsazen` (*obsazen* = occupied, *most* = bridge), `parnik`,
`uzly` (= nodes). Those symbols identify the PC global bank in §4: `bridgeObsazen` is `g_bridge_passage_claim` and `mostObsazen` is
`g_steamboat_shared_section_claim`, and the PS2 `Update__8C_bridgeFUi` uses the same route windows as §4b, value for value.

### 6e. Cross-build evidence that settles PC-side questions

| PC question | settled by |
|---|---|
| what the `0x84`-stride vector holds | the `gif::vector<C_bridge::S_BridgeCol,8>` template arg |
| what the `0x00655744` / `0x0065574c` arrays are | PS2 `bridgeIsUp` / `bridgeNeedUp`, indexed identically |
| what `bridge_load_data` loads | PS2's destination array is named `uzly`, and `G_ParnikUzel` links right after this class |
| that `bridge_ai` state 1's gate is a traffic query | PS2 calls `GetBridgeCarCount__13C_traffic_car` (twin of `traffic_car_get_bridge_car_count`, CONFIRMED) at that position |

### 6f. The five base stubs

`NetDirect`, `SaveData`, `SaveGameGetSize`, `SaveGameLoad` and `DuplicateFrom` all have **8-byte (empty) PS2 bodies**. The PC fold is
read at the vtable slots, not inferred from those empty bodies: an empty-looking slot can hold real code in another class (`C_PhThing`'s
slot 15 holds 1696 bytes of dead-reckoning code). A static drawbridge has no per-instance state to replicate, so an
emptied-replication reading does not apply here.

## 7. Related

- `structs/actor.md` — the base. `structs/entity_kind.md` — the kind enum.
- `structs/collision.md` — `collision_find_hits_for_frame`, `collision_dyn_insert`, `collision_dyn_erase`.
- `structs/traffic_car.md` — `traffic_car_get_bridge_car_count`.
- `basic/artifacts/cheats.md` — the cheat table holding PARNIK.

## 8. Open items

| Item | State |
|---|---|
| 1. in game: `hold_ms`, `reverse_direction` | no non-zero writer in `.text`, so their dependent arms never run as shipped; not tested in game |
