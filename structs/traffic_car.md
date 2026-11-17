# `C_traffic_car` — entity kind `0xc`, the ambient-traffic MANAGER

**Created** 2026-08-17 15:33 +0200 · **Modified** 2026-11-09 09:37 +0100

It is not "an ambient car". The class is one actor, and that actor is the **manager of the whole cruising
population**, so one instance owns the car elements, the gangster crews, the adopted elements, the traffic-side view of
the drawbridges and the parked and stolen car lists. Ambient **pedestrians** are a different class (`C_traffic_element`,
PC prefix `pedestrian_`, `structs/pedestrian.md`); the two meet in `traffic_car_seat_pedestrian_as_driver`. It is also where
other subsystems' cars come from: police reinforcement cars, mafia hit-cars and replacement cruisers are all spawned here, so
`traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_spawn_mafia_pursuit_car`, `traffic_car_adopt_car_with_new_driver`
and `traffic_car_destroy_crew_for_car` carry this class's prefix, not the consumer's.

| | |
|---|---|
| PC prefix | `traffic_car_` |
| PC kind (`+0x10`) | `0xc` |
| PC vtable | `g_traffic_car_vt` @ `0x0063ce20`, 40 slots |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x26c` (`operator_new(0x26c)` in `mission_create_actor` `case 0xc`; the last field ends there) |
| PC constructor | `traffic_car_ctor` |
| PC translation unit | `[0x004f62a0, 0x00508120)`, `C_trf_c.cpp`, guard to guard, 77 function starts, all named |
| PS2 class | `C_traffic_car` (tag `13C_traffic_car`), 49 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class's own band starts after the `C_actor` base.

```c
typedef struct C_traffic_car {              // : C_actor
    /* +0x070 */ uint8_t      game_inited;       // [H] 1 from the start of traffic_car_game_init; no other writer
    /* +0x074 */ S_ELEMENT  **elements;          // [H] one pointer per cruising element, element_count entries
    /* +0x078 */ uint32_t     unread_78;         // [U] 0 from traffic_car_game_init; no reader (class census)
    /* +0x07c */ S_ELEMENT  **type_pools;        // [H] per type: its array of 0x158-byte elements, type_count entries
    /* +0x080 */ int         *type_pool_counts;  // [H] per type: the element count of its pool
    /* +0x084 */ int          type_cycle[100];   // [H] the type drawn for each activation, indexed by type_cycle_index
    /* +0x214 */ int          type_cycle_index;  // [H] the read position of type_cycle; game_init seeds 0
    /* +0x218 */ int          active_count;      // [H] elements live now; activation stops at element_count
    /* +0x21c */ int          designer_count;    // [H] the element count stored in the scene data, before the scale
    /* +0x220 */ int          element_count;     // [H] designer_count * g_traffic_car_elems_coef, at least 1 above 0.05
    /* +0x224 */ float        despawn_distance;  // [H] designer value, seeded 180.0
    /* +0x228 */ float        near_distance;     // [H] designer value, seeded 150.0; the near end of the spawn distance
    /* +0x22c */ float        far_distance;      // [H] designer value, seeded 170.0; the far end of the spawn distance
    /* +0x230 */ float        spawn_distance;    // [H] the spawn distance of the current activation
    /* +0x234 */ S_traffic_car_type *types;      // [H] the designer's type table, type_count entries
    /* +0x238 */ int          type_count;        // [H] entries in types, stored in the scene data
    /* +0x23c */ S_vector     query_position;    // [H] the point activation and routing measure from, set by traffic_car_ai
    /* +0x248 */ S_vector     query_direction;   // [H] the direction routing measures its angles from
    /* +0x254 */ C_roads     *roads;             // [H] the road graph; node table at its +0x08, stride 0xec
    /* +0x258 */ C_actor     *player;            // [H] C_game.player, copied by game_init
    /* +0x25c */ uint8_t      unread_25c;        // [U] 1 from traffic_car_game_init; no reader (class census)
    /* +0x260 */ uint32_t     activation_timer;  // [H] ms accumulator of traffic_car_ai; acts above 0x15d, then resets
    /* +0x268 */ uint32_t     routing_timer;     // [H] ms accumulator of traffic_car_ai, reset when it acts
} C_traffic_car;
```

**Why these arrays.** `elements` has one entry per cruising car and `element_count` entries, the designer's count scaled by
`g_traffic_car_elems_coef`. `type_pools` and `type_pool_counts` have one entry per row of the designer's type table, and each pool
holds that type's share of the elements: `traffic_car_game_init` weights each type by `+0x14` (scaled by the mafia or police
coefficient) and hands out `element_count`. `type_cycle` holds 100 entries: the fill loop of `traffic_car_game_init` stops below 100,
`traffic_car_activate_elements` wraps the index at 99, and the gap up to `+0x214` agrees.

### 1a. `S_ELEMENT` — one cruising car (`0x158`)

`operator_new(0x158)` for an adopted element; `crt_ehvec_ctor(base, 0x158, n, ...)` with `traffic_car_element_ctor` for a pool.

```c
typedef struct S_ELEMENT {
    /* +0x000 */ uint8_t    active;           // [H] the element is in the world
    /* +0x004 */ I3D_frame *model;            // [H] the frame it uses now
    /* +0x008 */ I3D_frame *pool_model;       // [M] its own frame; no non-zero writer (.text scan), its reader is a dead branch
    /* +0x00c */ C_car     *car;              // [H] the car it uses now; the key of every per-car lookup
    /* +0x010 */ C_car     *pool_car;         // [M] its own car; no non-zero writer (.text scan), its readers are dead branches
    /* +0x014 */ C_driver  *driver;           // [H] the AI driver; its +0x1ac vector holds the S_LOCKCROSS records
    /* +0x018 */ float      query_distance;   // [H] the element's distance to query_position; no reader (class census)
    /* +0x01c */ I3D_model *driver_model;     // [H] the occupant model at the wheel
    /* +0x020 */ I3D_model *passenger_model;  // [H] the second occupant model, or 0
    /* +0x024 */ C_animations_machine_de_luxe anim_machine; // [H] embedded, up to +0x14c
    /* +0x14c */ uint8_t    door_anim_running; // [H] a door animation runs: the driver getting out, or a pedestrian getting in
    /* +0x14d */ uint8_t    taken_from_driver; // [H] the car was taken from its driver; the AI only measures its distance
    /* +0x14e */ uint8_t    pedestrian_boarding; // [H] the running door animation is a pedestrian boarding
    /* +0x14f */ uint8_t    no_parking;       // [H] copy of the type's no_parking; never picked to park
    /* +0x150 */ uint8_t    is_police;        // [H] copy of the type's +0x1c
    /* +0x151 */ uint8_t    taken_over;       // [M] no non-zero writer (.text scan); its readers are dead branches
    /* +0x152 */ uint8_t    in_pursuit;       // [H] set by the two pursuit spawners and the take-over; clears on deactivation
    /* +0x153 */ uint8_t    used_since_reset; // [H] round-robin marker: used since the pool's last reset
    /* +0x154 */ uint8_t    adopted;          // [H] §2a
    /* +0x155 */ uint8_t    is_gang;          // [H] the type's +0x1e when the type is not police
} S_ELEMENT;
```

### 1b. `S_traffic_car_type` — one row of the designer's table (`0x20`)

Read and written whole by the scene data functions (`0x20` bytes per row), `type_count` rows.

```c
typedef struct S_traffic_car_type {
    /* +0x00 */ char    model_name[0x14]; // [H] the car model, resolved by car_database_pack_id_by_model
    /* +0x14 */ float   weight;           // [H] share of the elements, scaled by the mafia or police coefficient
    /* +0x18 */ int     color_variant_count; // [H] the colour variants (%s%d.i3d, 0..9) the type's pool cycles through
    /* +0x1c */ uint8_t is_police;        // [H] copied to S_ELEMENT.is_police
    /* +0x1d */ uint8_t no_parking;       // [H] copied to S_ELEMENT.no_parking
    /* +0x1e */ uint8_t is_gang;          // [H] copied to S_ELEMENT.is_gang when is_police is clear
} S_traffic_car_type;
```

### 1c. `S_MAFIAN` — one spawned gangster crew (`0x0c`)

Elements of the vector at `0x0065616c` (`g_traffic_mafia_crews_begin` / `_end` / `_cap`); no set count.

```c
typedef struct S_MAFIAN {
    /* +0x00 */ C_car   *car;        // [H] the crew's car
    /* +0x04 */ C_human *driver;     // [H] the man at the wheel
    /* +0x08 */ C_human *passenger;  // [H] the second man
} S_MAFIAN;
```

### 1d. `S_BRIDGE` — one drawbridge as traffic sees it (`0x08`)

Elements of `g_traffic_bridge_slots`; no set count.

```c
typedef struct S_BRIDGE {
    /* +0x00 */ C_actor *actor;  // [H] the bridge
    /* +0x04 */ uint16_t node;   // [H] the road node it blocks
} S_BRIDGE;
```

### 1e. `S_LOCKCROSS` — one crossing an element has locked (`0x22`)

Elements of the vector at `C_driver+0x1ac`, the driver of the element: `traffic_car_lock_crossing` appends, `vec_erase_at_stride_0x22` removes.

```c
typedef struct S_LOCKCROSS {
    /* +0x00 */ uint8_t  blocks_a[4][4]; // [H] per direction and lane: this element holds one count of the node's +0x31 counter
    /* +0x10 */ uint8_t  blocks_b[4][4]; // [H] the same for the node's +0x30 counter
    /* +0x20 */ uint16_t node;           // [H] the road node, index into the 0xec-stride table
} S_LOCKCROSS;
```

**Why four by four.** `traffic_car_clear_lane_blocking` walks exactly that nest over the node record: four approaches at stride `0x34`, four
lanes at stride `8`.

## 2. Field evidence

The element, type, crew, bridge and lock records are walked through pointers, which the census of `this` cannot see: those
cells were read from the bodies.

### 2a. `C_traffic_car`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | game_inited | H | `traffic_car_game_init` | `traffic_car_ai` | - |
| +0x74 | elements | H | `traffic_car_game_done`, `traffic_car_game_init` | `traffic_car_ai`, `traffic_car_change_state`, `traffic_car_clear_lane_blocking`, `traffic_car_deactivate_adopted_elements`, `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_update` | - |
| +0x78 | unread_78 | U | `traffic_car_game_init` (dword 0) | - | no load anywhere (a trace over the kind-`0xc` functions and the `traffic_car_` prefix) |
| +0x7c | type_pools | H | `traffic_car_game_done`, `traffic_car_game_init` | `traffic_car_game_done`, `traffic_car_game_init` | - |
| +0x80 | type_pool_counts | H | `traffic_car_game_done`, `traffic_car_game_init` | `traffic_car_game_done`, `traffic_car_game_init` | - |
| +0x84 | type_cycle | H | `traffic_car_game_init` | `traffic_car_activate_elements` | 100 entries: the fill loop's bound is `< 100`, and the activation wraps the index at 99 |
| +0x214 | type_cycle_index | H | `traffic_car_game_init` | `traffic_car_activate_elements` | - |
| +0x218 | active_count | H | `traffic_car_deactivate_element`, `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_deactivate_element`, `traffic_car_update` | - |
| +0x21c | designer_count | H | `traffic_car_ctor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_game_init`, `traffic_car_save_data` | - |
| +0x220 | element_count | H | `traffic_car_ctor`, `traffic_car_game_init` | `traffic_car_ai`, `traffic_car_activate_elements`, `traffic_car_change_state`, `traffic_car_deactivate_adopted_elements`, `traffic_car_destroy_crew_for_car`, `traffic_car_game_done`, `traffic_car_update` | - |
| +0x224 | despawn_distance | H | `traffic_car_ctor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_ai`, `traffic_car_save_data` | - |
| +0x228 | near_distance | H | `traffic_car_ctor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_activate_elements`, `traffic_car_save_data` | the spawn distance is near + rand x (far - near) |
| +0x22c | far_distance | H | `traffic_car_ctor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_activate_elements`, `traffic_car_save_data` | as `+0x228` |
| +0x230 | spawn_distance | H | `traffic_car_activate_elements` | `traffic_car_activate_elements`, `traffic_car_get_next_pos_info`, `traffic_car_find_last_crossing` | the spawn point must lie farther than this x 0.7..1.0 (by the angle to `query_direction`) from `query_position` |
| +0x234 | types | H | `traffic_car_ctor`, `traffic_car_dtor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_dtor`, `traffic_car_duplicate_from`, `traffic_car_game_init`, `traffic_car_load_data`, `traffic_car_save_data` | - |
| +0x238 | type_count | H | `traffic_car_ctor`, `traffic_car_duplicate_from`, `traffic_car_load_data` | `traffic_car_duplicate_from`, `traffic_car_game_done`, `traffic_car_game_init`, `traffic_car_load_data`, `traffic_car_save_data` | - |
| +0x23c | query_position | H | `traffic_car_ai` | `traffic_car_ai`, `traffic_car_find_last_crossing`, `traffic_car_get_next_pos_info` | - |
| +0x248 | query_direction | H | `traffic_car_ai` | `traffic_car_ai`, `traffic_car_find_last_crossing`, `traffic_car_get_next_pos_info` | - |
| +0x254 | roads | H | `traffic_car_game_init` | `traffic_car_ai`, `traffic_car_clear_lane_blocking`, `traffic_car_deactivate_element`, `traffic_car_find_last_crossing`, `traffic_car_game_init`, `traffic_car_get_next_pos_info`, `traffic_car_lock_crossing` | - |
| +0x258 | player | H | `traffic_car_ai`, `traffic_car_game_init` | `traffic_car_ai` | - |
| +0x25c | unread_25c | U | `traffic_car_game_init` (1 at both exits) | - | stored unconditionally and never loaded (a trace over the kind-`0xc` functions and the `traffic_car_` prefix) |
| +0x260 | activation_timer | H | `traffic_car_ai` | `traffic_car_ai` | - |
| +0x268 | routing_timer | H | `traffic_car_ai` | `traffic_car_ai` | - |

**Owned by another object:** `+0xd90` -> `structs/vehicle.md` (`seats` begin, `C_Vehicle +0xd20` of a
car) and `+0xd94` -> `structs/vehicle.md` (`seats` end, `+0xd24`), read by
`traffic_car_adopt_car_with_new_driver`, whose ECX is the car to adopt (its one caller,
`police_action_tick`, passes the squad's chase car, `police_action +0x5c`); `+0x1f8c` ->
`structs/vehicle.md` (`fuel_capacity`, `C_Vehicle +0x1f1c` of a car), read by
`traffic_car_take_over_cruising_car`, whose ECX is the car taken over.

### 2b. `S_ELEMENT`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | active | H | `traffic_car_activate_elements`, `traffic_car_deactivate_element`, `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_deactivate_adopted_elements`, `traffic_car_update` | - |
| +0x4 | model | H | `traffic_car_deactivate_element`, `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_change_state`, `traffic_car_game_init` | - |
| +0x8 | pool_model | M | - | `traffic_car_deactivate_element` (behind `+0x151`) | no non-zero writer (`.text` scan): zeroed only with the record; the read sits behind `taken_over` and is a dead branch |
| +0xc | car | H | `traffic_car_deactivate_element`, `traffic_car_game_init` | `traffic_car_change_state`, `traffic_car_deactivate_element`, `traffic_car_release_stolen_car`, `traffic_car_take_over_cruising_car`, `traffic_car_update` | - |
| +0x10 | pool_car | M | - | `traffic_car_deactivate_element` (behind `+0x151`), `traffic_car_adopt_car_with_new_driver` (`+0x10 == car`) | no non-zero writer (`.text` scan): zeroed only with the record, so neither test can succeed |
| +0x14 | driver | H | `traffic_car_destroy_crew_for_car`, `traffic_car_game_init` | `traffic_car_clear_lane_blocking`, `traffic_car_deactivate_element`, `traffic_car_game_done` | - |
| +0x18 | query_distance | H | `traffic_car_ai` (`0x004fc76c`, `0x004fee5f`) | - | the element's distance to `query_position`. No reader among the `traffic_car_*` functions and the two parking-list readers, `pedestrian_manager_try_send_pedestrian_to_parked_car` and `ai_group_get_closest_usable_car` (they read `+0x04` and `+0x0c`); the `call [reg+0x18]` of `traffic_car_release_stolen_car` and `traffic_car_seat_pedestrian_as_driver` is a vtable call on the human model |
| +0x1c | driver_model | H | `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_game_init` | `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_game_done`, `traffic_car_release_stolen_car` | - |
| +0x20 | passenger_model | H | `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_release_stolen_car`, `traffic_car_seat_pedestrian_as_driver` | `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_game_done`, `traffic_car_release_stolen_car` | - |
| +0x24 | anim_machine | H | `traffic_car_element_ctor` | `traffic_car_deactivate_element`, `traffic_car_destroy_crew_for_car`, `traffic_car_kill_occupant_on_hit` | `structs/anim_machine.md` |
| +0x14c | door_anim_running | H | `traffic_car_activate_elements` (0), `traffic_car_ai`, `traffic_car_seat_pedestrian_as_driver` | `traffic_car_ai`, `traffic_car_deactivate_element` | a door animation is running, the driver getting out to park or a pedestrian getting in; ticked until it ends, then cleared |
| +0x14d | taken_from_driver | H | `traffic_car_activate_elements`, `traffic_car_seat_pedestrian_as_driver` (0), `traffic_car_release_stolen_car` (1) | `traffic_car_ai` | the car was taken from its driver: the AI only measures its distance and deactivates it beyond `despawn_distance` |
| +0x14e | pedestrian_boarding | H | `traffic_car_activate_elements` (0), `traffic_car_seat_pedestrian_as_driver` (1) | `traffic_car_ai`, `traffic_car_deactivate_element` | the running animation is a pedestrian boarding: when it ends, the passenger model `+0x20` becomes the driver man; such an element is not parked |
| +0x14f | no_parking | H | `traffic_car_game_init` (the type's `+0x1d`), `traffic_car_adopt_car_with_new_driver` (1) | `traffic_car_ai` | the element is never picked to park |
| +0x150 | is_police | H | `traffic_car_adopt_car_with_new_driver`, `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_deactivate_element`, `traffic_car_game_init`, `traffic_car_on_collision_try_register_assault_on_police_car` | - |
| +0x151 | taken_over | M | `traffic_car_deactivate_element` (0) | `traffic_car_deactivate_element`, `traffic_car_take_over_cruising_car`, `traffic_car_update` | no non-zero writer (`.text` scan): 5 accesses in the image, the only store 0; its readers are dead branches |
| +0x152 | in_pursuit | H | `traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_deactivate_element`, `traffic_car_spawn_mafia_pursuit_car`, `traffic_car_take_over_cruising_car` | `traffic_car_ai`, `traffic_car_on_collision_try_register_assault_on_gang_car`, `traffic_car_on_collision_try_register_assault_on_police_car` | - |
| +0x153 | used_since_reset | H | `traffic_car_activate_elements` (1; 0 on the pool reset) | `traffic_car_activate_elements` (`[ecx-5]` walking pointer, `0x004f7812`) | the round-robin marker: an activation takes an inactive element not used since the pool's last reset |
| +0x154 | adopted | H | `traffic_car_adopt_car_with_new_driver`, `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_deactivate_adopted_elements`, `traffic_car_deactivate_element`, `traffic_car_update` | §4 |
| +0x155 | is_gang | H | `traffic_car_game_init` | `traffic_car_activate_elements`, `traffic_car_ai`, `traffic_car_deactivate_element`, `traffic_car_on_collision_try_register_assault_on_gang_car` | - |

### 2c. `S_traffic_car_type`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | model_name | H | `traffic_car_load_data` | `traffic_car_game_init` | - |
| +0x14 | weight | H | `traffic_car_load_data` | `traffic_car_game_init` | - |
| +0x18 | color_variant_count | H | `traffic_car_load_data` | `traffic_car_game_init` | limits the colour variants (`%s%d.i3d`, 0..9) the type's pool cycles through |
| +0x1c | is_police | H | `traffic_car_load_data` | `traffic_car_game_init` | - |
| +0x1d | no_parking | H | `traffic_car_load_data` | `traffic_car_game_init` | copied to `S_ELEMENT` `no_parking` |
| +0x1e | is_gang | H | `traffic_car_load_data` | `traffic_car_game_init` | - |

### 2d. `S_MAFIAN`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | car | H | `traffic_car_spawn_mafia_pursuit_car` | `traffic_car_update` | - |
| +0x4 | driver | H | `traffic_car_spawn_mafia_pursuit_car` | `traffic_car_update` | - |
| +0x8 | passenger | H | `traffic_car_spawn_mafia_pursuit_car` | `traffic_car_update` | - |

### 2e. `S_BRIDGE`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | actor | H | `traffic_car_game_init` | `driver_tick` | `structs/driver.md` |
| +0x4 | node | H | `traffic_car_game_init` | `driver_tick` | `structs/driver.md` |

### 2f. `S_LOCKCROSS`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | blocks_a | H | `traffic_car_lock_crossing` | `traffic_car_clear_lane_blocking` | - |
| +0x10 | blocks_b | H | `traffic_car_lock_crossing` | `traffic_car_clear_lane_blocking` | - |
| +0x20 | node | H | `traffic_car_lock_crossing` | `traffic_car_clear_lane_blocking`, `traffic_car_lock_crossing` | - |

## 3. Method family

49 PS2 methods: 41 have a PC function here, 2 are base stubs on shared PC stubs, 1 is inlined, and 5 are in the PS2-only table of §9. `traffic_car_scalar_deleting_dtor` is PC-only.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004f62a0 | `crt_static_init_004f62a0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f62d0 | `crt_atexit_register_004f62d0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f62e0 | `traffic_adopted_elements_static_dtor` | `g_traffic_adopted_elements_*` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6320 | `crt_static_init_004f6320` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6350 | `crt_atexit_register_004f6350` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6360 | `traffic_bridge_slots_static_dtor` | `g_traffic_bridge_slots` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f63a0 | `crt_static_init_004f63a0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f63d0 | `crt_atexit_register_004f63d0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f63e0 | `traffic_car_registry_static_dtor` | `g_traffic_car_registry` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6420 | `crt_static_init_004f6420` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6450 | `crt_atexit_register_004f6450` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6460 | `traffic_parking_cars_static_dtor` | `g_traffic_parking_cars` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f64a0 | `crt_static_init_004f64a0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f64d0 | `crt_atexit_register_004f64d0` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f64e0 | `traffic_stolen_cars_static_dtor` | `g_traffic_stolen_cars` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6520 | `crt_static_init_004f6520` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6550 | `crt_atexit_register_004f6550` | file-scope vector | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f6560 | `traffic_mafia_crews_static_dtor` | `g_traffic_mafia_crews_*` | - | - | - | MSVC static-init step of the six file-scope vectors (§5) |
| 0x004f65a0 | `traffic_car_ctor` | instance | `__ct__13C_traffic_carFv` | 0x002121b0 | CONFIRMED | installs the vtable, kind `0xc`, the three distances, seeds the three coefficients to 1.0 |
| 0x004f6610 | `traffic_car_scalar_deleting_dtor` | instance | - | - | - | MSVC-only |
| 0x004f6630 | `traffic_car_dtor` | instance | `__dt__13C_traffic_carFv` | 0x00212230 | CONFIRMED | frees `types` |
| 0x00503e40 | `traffic_car_element_ctor` | instance | - | - | - | `S_ELEMENT` default constructor: only the embedded anim machine; handed to `crt_ehvec_ctor` by address |

### 3b. Accessors

None.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004254e0 | `traffic_car_get_registry_header_addr` | - | - | - | CONFIRMED `dword_vec_copy_assign` and then counts it with `vec_count_stride_0x4`, which reads `obj+4`/`obj+8` |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__13C_traffic_carFUi` | 0x00212d70 | base-stub | slot 15, `NetDirect`, shared by 30 vtables (§6) |
| 0x004f6660 | `traffic_car_init` | `Init__13C_traffic_carFP9I3D_frame` | 0x002122a0 | CONFIRMED | Slot 18 — this class overrides it rather than folding onto the shared default |
| 0x004f66b0 | `traffic_car_change_state` | `ChangeState__13C_traffic_carFv` | 0x00212300 | CONFIRMED | Slot 27 |
| 0x004f67e0 | `traffic_car_registry_set_enabled` | `Scipni__13C_traffic_carFb` | 0x00212510 | CONFIRMED | - |
| 0x004f6900 | `traffic_car_duplicate_from` | `DuplicateFrom__13C_traffic_carFP7C_actor` | 0x00212700 | CONFIRMED | Slot 21 |
| 0x004f69c0 | `traffic_car_save_data` | `SaveData__13C_traffic_carFP7C_chunk` | 0x002127e0 | CONFIRMED | Slot 19 |
| 0x004f6af0 | `traffic_car_load_data` | `LoadData__13C_traffic_carFP7C_chunk` | 0x00212960 | CONFIRMED | Slot 20 |
| 0x004f6ef0 | `traffic_car_collision` | `Collision__13C_traffic_carFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00212d60 | CONFIRMED | slot 39, exactly 1 dataref: returns false |
| 0x004f6f00 | `traffic_car_deactivate_element` | `DeactivateElem__13C_traffic_carFib` | 0x00212d80 | CONFIRMED | Called x3 by the AI tick, and by Update, GameDone and the deactivate-all method — the same caller set as on PS2 |
| 0x004f76d0 | `traffic_car_activate_elements` | `ActivateElems__13C_traffic_carFbi` | 0x00213610 | CONFIRMED | 🔑 The function no Ghidra export could decompile; body recovered `0x00624fa0` `__chkstk` |
| 0x004fb000 | `traffic_car_get_next_pos_info` | `GetNewPosInfo__13C_traffic_carFRUsRUs` | 0x00218030 | CONFIRMED | Its only caller is `ActivateElems`, which has no decompiled body — which is why this function appears caller-less in the decomp |
| - | - | `FindNodesIndex__13C_traffic_carFRUsRUsRi` | 0x00218c90 | inlined | inlined here; five `S_vector::AngleTo` calls against the PS2's three plus two (§9) |
| 0x004fb4e0 | `traffic_car_find_last_crossing` | `FindLastCrossing__13C_traffic_carFRUsRi` | 0x00218500 | CONFIRMED | Sole caller is `traffic_car_get_next_pos_info` on both builds |
| 0x004fba80 | `traffic_car_ai` | `AI__13C_traffic_carFUi` | 0x00219400 | CONFIRMED | Vtable slot 13 |
| 0x004feef0 | `traffic_car_prepare_next_direction` | `PrepareNextDir__13C_traffic_carFib` | 0x00216e20 | CONFIRMED | 🔑 Callers DIFFER by build: PS2 `ActivateElems` calls it out of line, while MSVC INLINED a copy into `ActivateElems` (two `vec2_signed_angle_diff` calls in its byte scan) |
| 0x004ffc50 | `traffic_car_clear_lane_blocking` | `ClearBlocking__13C_traffic_carFii` | 0x002192f0 | CONFIRMED | Called x2 from the AI tick and x1 from `traffic_car_deactivate_element`, matching PS2 exactly |
| 0x004ffce0 | `traffic_car_lock_crossing` | `LockCrossing__13C_traffic_carFii` | 0x0021eb60 | CONFIRMED | Sole caller is the AI tick, x2 |
| 0x005010b0 | `traffic_car_update` | `Update__13C_traffic_carFUi` | 0x00221020 | CONFIRMED | Slot 14 |
| 0x005012e0 | `traffic_car_game_init` | `GameInit__13C_traffic_carFv` | 0x00221380 | CONFIRMED | Slot 25; the largest method with a body |
| 0x00503e50 | `traffic_car_game_done` | `GameDone__13C_traffic_carFv` | 0x00224fc0 | CONFIRMED | Slot 26 |
| 0x005042c0 | `traffic_car_release_stolen_car` | `Corovacka__13C_traffic_carFP5C_car` | 0x00225570 | CONFIRMED | Called when the player throws the occupant out of a cruising car |
| 0x00504380 | `traffic_car_get_parking_cars` | `GetParkingCars__13C_traffic_carFv` | 0x002256c0 | - | CONFIRMED twin |
| 0x00504390 | `traffic_car_seat_pedestrian_as_driver` | `ChodDoRici__13C_traffic_carFP9I3D_modelP5C_car` | 0x002256d0 | CONFIRMED | The PS2 name is crude Brno slang and describes nothing — the name here is measured from the body, which ends in `car_set_has_driver(1)` |
| 0x00504780 | `traffic_car_activate_or_spawn_pursuit_vehicles` | `HulaaaaaMuf__13C_traffic_carFP7C_actorRQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>ibP5C_carf` | 0x00225c30 | CONFIRMED | iterates `g_traffic_car_registry_begin`..`_end`; for each candidate passing the radius²/facing/LOS gates it REACTIVATES a pooled car by name |
| 0x005052a0 | `traffic_car_on_collision_try_register_assault_on_police_car` | `CoDelasTyCuraku__13C_traffic_carFP5C_carP7C_actorf` | 0x00226b90 | - | `__fastcall` |
| 0x00505310 | `traffic_car_on_collision_try_register_assault_on_gang_car` | `CoDelasTyCuraku1__13C_traffic_carFP5C_carP7C_actorf` | 0x00226c20 | - | `__fastcall` |
| 0x005053a0 | `traffic_car_release_cop_car_slot` | `ChlupatejVystrik__13C_traffic_carFv` | 0x00226d00 | CONFIRMED | - |
| 0x005053c0 | `traffic_car_release_mafia_car_slot` | `MafianVystrik__13C_traffic_carFv` | 0x00226d30 | CONFIRMED | Paired with `traffic_car_release_cop_car_slot`; the two PS2 names (*mafian* / *chlupatej* = cop) are what identified both counters |
| 0x005053e0 | `traffic_car_spawn_mafia_pursuit_car` | `Krles__13C_traffic_carFP7C_actorP7C_actor` | 0x00226d60 | - | `__fastcall` |
| 0x00506130 | `traffic_car_take_over_cruising_car` | `SebraliMiKaaaruSvine__13C_traffic_carFP5C_car` | 0x00227e30 | - | CONFIRMED twin; ECX is the car taken over, not the manager |
| 0x00506760 | `traffic_car_get_bridge_car_count` | `GetBridgeCarCount__13C_traffic_carFP7C_actor` | 0x002288a0 | CONFIRMED | **The `bridge_ai` case-1 gate** — the drawbridge refuses to raise while this returns >= 1 |
| 0x005067f0 | `traffic_car_is_possible_coll` | `IsPossibleColl__13C_traffic_carFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x002289a0 | CONFIRMED | Slot 33 |
| 0x00506900 | `traffic_car_hit` | `Hit__13C_traffic_carF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00228b20 | CONFIRMED | Slot 31, own override |
| 0x00506b60 | `traffic_car_find_car_heading_at_point` | `ChcesMeSejmout__13C_traffic_carFRC8S_vector` | 0x00228e50 | CONFIRMED | This is a pedestrian asking *is a car about to run me over?* — the caller is the pedestrian tick |
| 0x00506bb0 | `traffic_car_deactivate_adopted_elements` | `TakATedSiVasKupim_Vsecky__13C_traffic_carFb` | 0x00228ee0 | CONFIRMED | 🔑 `S_ELEMENT+0x154` = record is heap-owned (an ADOPTED car) |
| 0x00506c00 | `traffic_car_kill_occupant_on_hit` | `DostalJsemRanuKachlou__13C_traffic_carFP5C_carP9I3D_frame` | 0x00228fe0 | CONFIRMED | Sole caller `car_hit` <-> `Hit__5C_car` |
| 0x00506cf0 | `traffic_car_adopt_car_with_new_driver` | `NoTakMiTedaDajTuPleckuJaJuOdvezu__13C_traffic_carFP5C_car` | 0x00229260 | CONFIRMED | pursuit chase records; ECX is the car to adopt, not the manager |
| 0x00507cc0 | `traffic_car_destroy_crew_for_car` | `VypraskeeeKaaaru__13C_traffic_carFP7C_actor` | 0x0022a340 | CONFIRMED | - |
| 0x00507f80 | `traffic_car_save_game_get_size` | `SaveGameGetSize__13C_traffic_carFv` | 0x0022ab10 | CONFIRMED | Savegame triple, slot 22 |
| 0x00507f90 | `traffic_car_save_game_save` | `SaveGameSave__13C_traffic_carFPPv` | 0x0022ab30 | CONFIRMED | Savegame triple, slot 23 |
| 0x00507fc0 | `traffic_car_save_game_load` | `SaveGameLoad__13C_traffic_carFPPv` | 0x0022ab80 | CONFIRMED | Savegame triple, slot 24 |
| 0x00574a40 | `stub_noop_cdecl_00574a40` | `Death__13C_traffic_carFv` | 0x002127d0 | base-stub | slot 29, `Death`, shared by 36 vtables (§6) |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004f62b0 | `crt_static_ctor_004f62b0` | - | - | - | Static-init scaffolding |
| 0x004f6330 | `crt_static_ctor_004f6330` | - | - | - | Static-init scaffolding |
| 0x004f63b0 | `crt_static_ctor_004f63b0` | - | - | - | Static-init scaffolding |
| 0x004f6430 | `crt_static_ctor_004f6430` | - | - | - | Static-init scaffolding |
| 0x004f64b0 | `crt_static_ctor_004f64b0` | - | - | - | Static-init scaffolding |
| 0x004f6530 | `crt_static_ctor_004f6530` | - | - | - | Static-init scaffolding |
| 0x004f76c0 | `thunk_stub_noop_cdecl_00574a40` | - | - | - | Named `thunk_<target>` by its target; the name was verified unique (0 prior occurrences) |
| 0x004faf30 | `vec2_scale_to` | - | - | - | `S_vector` or vector helper of the unit; `structs/vehicle.md` and the unit's callers |
| 0x004faf50 | `vec2_add_in_place` | - | - | - | `S_vector` or vector helper of the unit; `structs/vehicle.md` and the unit's callers |
| 0x004faf70 | `vec2_sub_to` | - | - | - | `S_vector` or vector helper of the unit; `structs/vehicle.md` and the unit's callers |
| 0x004faf90 | `vec2_line_intersect` | - | - | - | `S_vector` or vector helper of the unit; `structs/vehicle.md` and the unit's callers |
| 0x00507ff0 | `vec_count_stride_0xec` | - | - | - | Bare vector size, stride 0xec |
| 0x00508020 | `vec_count_stride_0x22` | - | - | - | Bare vector size, stride 0x22 (the PC `S_LOCKCROSS` record) |
| 0x00508050 | `vec_erase_at_stride_0x22` | - | - | - | Bare vector erase, stride 0x22 |
| 0x005080b0 | `vec_clear_keep_capacity` | - | - | - | Bare vector clear |
| 0x005080f0 | `stl_copy_range_stride_0x22` | - | - | - | Bare range copy, stride 0x22 |

## 4. The ADOPTED flag

Decoded from its setter and its consumer:

* **Set** by `traffic_car_adopt_car_with_new_driver` immediately after
  `operator_new(0x158)` — the path that takes over a car standing in the world and gives it
  a fresh `C_driver` and anim machine.
* **Read** by `traffic_car_deactivate_element` as the last thing it does: clear ⇒ null the
  slot and return, leaving the pooled record alone; set ⇒ delete the driver and both
  driver-man models, `car_set_owner(0,0)` / `(0,1)`, `car_set_is_traffic_car(0)`, hand the
  car back to the police pool, run the `anim_machine` destructor and
  `operator_delete` the record.

⇒ **`adopted` means "this record is heap-owned — the manager ADOPTED this car rather than
drawing it from a pool"**, and deactivation must free the record and give the car back.
`traffic_car_deactivate_adopted_elements` (`0x00506bb0`) is the scene-teardown pass that
retires exactly those: it deactivates each element with `active` and `adopted` both set, and takes
no parameter (it `ret`s popping 0). `traffic_car_update` guards on the same pair,
`active != 0 && adopted == 0`.

## 5. Global state

**Six file-scope `std::vector` statics**, each with its own MSVC init/ctor/atexit/dtor
quartet and one bit of the shared run-once word `g_traffic_car_static_dtor_guard`:

| guard bit | address | what |
|---|---|---|
| `0x01` | `0x006561c0` | the adopted `S_ELEMENT *` records (`g_traffic_adopted_elements_*`) |
| `0x02` | `0x006561ac` | `g_traffic_bridge_slots` |
| `0x04` | `0x0065619c` | `g_traffic_car_registry` |
| `0x08` | `0x0065618c` | `g_traffic_parking_cars` |
| `0x10` | `0x0065617c` | the stolen cars, `C_car *` (`g_traffic_stolen_cars`) |
| `0x20` | `0x0065616c` | the crew records, `S_MAFIAN` (`g_traffic_mafia_crews_*`) |

Two counters, each with exactly one incrementer and one release:

| global | incremented by | released by |
|---|---|---|
| `g_traffic_cop_car_count` | `traffic_car_activate_or_spawn_pursuit_vehicles` | `traffic_car_release_cop_car_slot` |
| `g_traffic_mafia_car_count` | `traffic_car_spawn_mafia_pursuit_car` | `traffic_car_release_mafia_car_slot` |

## 6. Vtable `0x0063ce20`

Slots 0–12 are the `C_actor_track` base (`structs/actor_track.md`). The class's own:

| slot | function | note |
|---|---|---|
| 13 | `traffic_car_ai` | - |
| 14 | `traffic_car_update` | - |
| 15 | `stub_noop_args4_00461180` | `NetDirect`, shared by 30 vtables — no twin stamp |
| 16 | `traffic_car_scalar_deleting_dtor` | MSVC-only |
| 18 | `traffic_car_init` | own override, not the shared default |
| 19–21 | `traffic_car_save_data` / `_load_data` / `_duplicate_from` | - |
| 22–24 | the `traffic_car_save_game_*` triple | - |
| 25 / 26 | `traffic_car_game_init` / `_game_done` | - |
| 27 | `traffic_car_change_state` | - |
| 29 | `stub_noop_cdecl_00574a40` | `Death`, shared by 36 vtables — no twin stamp |
| 31 | `traffic_car_hit` | own override |
| 33 | `traffic_car_is_possible_coll` | - |
| 39 | `traffic_car_collision` | 🔑 **exactly 1 dataref** — this class's own, not a shared stub |

## 7. The routing half

`traffic_car_ai` is the engine. Per frame it activates and retires elements by distance,
drives each along its path, and claims/releases crossings:

```
traffic_car_ai
  ├─ traffic_car_activate_elements   (0x004f76d0, §8)
  │    ├─ traffic_car_get_next_pos_info  ── chooses the next node + lane
  │    │    └─ traffic_car_find_last_crossing
  │    │       (a node-index lookup is INLINED here — marked in the decomp, §9)
  │    └─ traffic_car_prepare_next_direction   (also inlined a second time here)
  ├─ traffic_car_deactivate_element  ×3
  ├─ traffic_car_lock_crossing       ×2
  └─ traffic_car_clear_lane_blocking ×2
```

`traffic_car_clear_lane_blocking` is what byte-proves the `S_LOCKCROSS` shape (§1e): a nested
4×4 walk decrementing per-lane blocking counters on the node the element is leaving.

### 7a. Inlining on the PC

* **`traffic_car_prepare_next_direction` is inlined once and called once.** MSVC spliced a copy
  into `traffic_car_activate_elements` (whose body makes two `vec2_signed_angle_diff` calls where
  the out-of-line copy makes one) and kept the out-of-line body for `traffic_car_ai`. That is why
  the out-of-line copy's caller is `traffic_car_ai`.
* **A node-index lookup is inlined into `traffic_car_get_next_pos_info`**, marked in the decomp
  with a `// ===== INLINED ON PC:` banner (the inlined method and the call-count proof are in §9).

## 8. `traffic_car_activate_elements` — the activation pass

**14 429 bytes**, the second-largest function in the TU. It walks the `S_ELEMENT` vector
(`elements`, `element_count`, stopping at `active_count`) and brings cruising cars in and out of
existence around the query point: per element it picks the next road node and lane, seats a
driver, applies the car's CARINIT parameters, steers, starts the engine and fuels it, and retires
out-of-range elements through `traffic_car_deactivate_element` (9 callsites). It also claims
bridge slots and consults the railway list, so a spawn never lands on a drawbridge or a level
crossing. Signature `(this, char mode, int index)`; the only caller, `traffic_car_ai`, passes
`(0, -1)` — `-1` = every element.

**Why the stock Ghidra export has no body.** The prologue is `and esp,0xfffffff8` (stack
realignment) + an SEH frame + `mov eax,0x125c; call __chkstk`. `__chkstk` (`crt_chkstk`,
`0x00624fa0`) page-probes and then does `mov esp,ecx`, so ESP changes inside a callee by a
register-supplied amount on a base that is no longer an affine function of the entry ESP; Ghidra
models a call as a constant stack purge and fails with *"Low-level Error: Cannot properly adjust
input varnodes"*. 17 functions call `__chkstk` and all others decompile; this is the only one
that also realigns ESP (`and esp,0xfffffff8` occurs 5 times in the image, no other near a probe
call). The body in the annotated decomp was produced by naming `0x00624fa0` `__chkstk` and marking
it inline, which enables Ghidra's built-in `alloca_probe` p-code injection.

### 8a. Globals near the class

| address | what |
|---|---|
| `0x00656154` / `0x00656158` / `0x0065615c` | `g_traffic_car_elems_coef` / `g_traffic_car_mafia_coef` / `g_traffic_car_police_coef`, seeded to `1.0f` by the ctor; `traffic_car_game_init` scales the element count by the first and weights the mafia / police spawn entries by the other two |
| `0x00656160` | `g_traffic_car_activation_suspended` — `traffic_car_ai` skips `traffic_car_activate_elements` while it is set; `traffic_car_game_init` clears it twice and nothing on the PC sets it |
| `0x00656161` | `g_traffic_cars_enabled` |

## 9. PC vs PS2

**Method accounting — 49 PS2 methods:**

| outcome | count | which |
|---|---|---|
| own PC body, body-diffed | 41 | everything named in §3 |
| folds onto a shared stub | 2 | `Death` (36 vtables), `NetDirect` (30) |
| inlined on the PC | 1 | `FindNodesIndex`, into `GetNewPosInfo` (§7a) |
| PS2-port-only | 3 | `ChangeSceneInit`, `ChangeSceneDone` (city streaming, `InterCity` occurs 0× on PC), `DejSiZatimSlofiiika` (its caller is PS2-only) |
| absent on the PC | 1 | `InvalidateActor`: `game_invalidate_actor` has no kind-`0xc` arm (below) |
| second PS2 cluster | 1 | `DajDevatemuTyZkurvenyZmrdyAtFurtNevotravuje` — 12 B at PS2 `0x0032ea90`, sole caller `actionTick_2__8C_entity`; its PC twin is in the `C_entity` band, not this TU |
| PC-only | 1 | the MSVC scalar deleting destructor |

`traffic_car_element_ctor` is also PC-only: the PS2 holds the anim machine as a POINTER at
element `+0x0`, so no element constructor was emitted there.

**`InvalidateActor` has no PC counterpart.** The PS2 `C_game::InvalidateActor` calls
`InvalidateActor__13C_traffic_car` for every kind-`0xc` actor and the pedestrian generator's for
kind `0x12`; the PC `game_invalidate_actor` sends both kinds to the exit of its kind switch, and the
kinds it does handle match the PS2 set.

**`ActivateElems__13C_traffic_carFbi` ↔ `traffic_car_activate_elements` (CONFIRMED).** An `E8`
scan of the PC byte range reproduces the PS2 callee multiset: `DeactivateElem` ×9↔×11,
`GetNewPosInfo` ×1, `GetNearNode` ×1, plus `car_deactivate`, `car_reset`, `car_set_owner`,
`car_set_enter_protect`, `driver_reset`, `vehicle_set_steer`, `vehicle_set_fuel`,
`vehicle_apply_carinit_params`, `car_set_has_driver` ×2, `UpdateWMatrixProc` ×1,
`S_vector::operator*` ×2, `GetWorldPos` ×1 — every one 1:1. The body diff agrees: same
`count <= cap` early-out, the element-pointer vector (PC `+0x74` / PS2 `+0xa0`), the counters
(PC `+0x220` / `+0x218`, PS2 `+0x24c` / `+0x244`), the `param < 0` = all-elements branch, the
`GetNewPosInfo` call at the same position, and the sole PC callsite's `(0, 0xffffffff)` against
the PS2 `(bool, int)` signature.

**`FindNodesIndex` inlining proof.** The PC `traffic_car_get_next_pos_info` makes **five**
`S_vector::AngleTo` calls — exactly PS2 `GetNewPosInfo`'s three plus `FindNodesIndex`'s two. On the
PS2, `PrepareNextDir` has a single caller, `ActivateElems`.

**The ADOPTED flag on the PS2.** `NoTakMiTedaDajTuPleckuJaJuOdvezu` (↔
`traffic_car_adopt_car_with_new_driver`) sets `+0x33` after `__nw__FUi(0x38)`; `DeactivateElem`'s
`+0x33` arm is the same free-and-return sequence, `__dl__FPv` included. `Update` guards on
`+0x34 != 0 && +0x33 == 0`, which pins **PC `S_ELEMENT+0x0` = PS2 `+0x34` = the active flag**.
`TakATedSiVasKupim_Vsecky__13C_traffic_carFb` ↔ `traffic_car_deactivate_adopted_elements` matches
statement for statement, but the PS2 takes a bool choosing between "elements with `+0x34`" and
"elements with `+0x34` and `+0x33`", and the PC implements only the `param == 0` arm. The
other arm has no PC counterpart to implement: its only caller is `G_GarageClass::UdelejBum`, the
garage explosion, which is reached only from the PS2-only garage menu (`ChangeCar`, `TakeCar`,
`DoGarageMenu`; `structs/garage.md` §6b), and none of that function's four position constants
(`-1757.0`, `-1758.0`, `-5.42`, `2.44`) occurs anywhere in the PC image. `Done__6C_game` calls the
`0` arm, as PC `game_done` does.

**Record-layout differences, measured:**

* `S_ELEMENT` is `0x158` B on the PC against `0x38` B on the PS2, the difference being the
  **embedded** `C_animations_machine_de_luxe`. The flag band is not a uniform shift:
  `+0x30`↔`+0x150` is `+0x120`, `+0x33`↔`+0x154` is `+0x121`.
* `S_LOCKCROSS` is **`0x22` bytes on the PC and `0x32` on the PS2**, and `ClearBlocking`
  decrements **two** per-lane counters on the PC against **three** on the PS2. The road-node
  record is `0xec` on the PC and `0xe0` on the PS2.
* The path-node array is stride `0x70` on the PC and `0x90` on the PS2; the crossing table
  entry is `0x18` PC / `0x20` PS2.
* The ctor's kind write is byte-identical on both builds.

**The dev-humour names, decoded from behaviour rather than transliterated.** Two of them name a
global: *chlupatej* = a cop → `g_traffic_cop_car_count`; *mafian* = a gangster →
`g_traffic_mafia_car_count`. *Corovacka* = a theft, which corroborates
`traffic_car_release_stolen_car`. The rest (`HulaaaaaMuf`, `Krles`,
`NoTakMiTedaDajTuPleckuJaJuOdvezu`, `SebraliMiKaaaruSvine`, `ChodDoRici`,
`DostalJsemRanuKachlou`, `TakATedSiVasKupim_Vsecky`, `ChcesMeSejmout`,
`VypraskeeeKaaaru`, `DejSiZatimSlofiiika`) carry no behaviour at all, and every PC name was
derived from the body and the caller/callee pairs.

**Caller pairs that match one-for-one across builds:**

| PC | PS2 |
|---|---|
| `game_set_traffic_visible` | `SetTrafficVisible__6C_game` |
| `human_do_throw_occupant_from_car` | `Do_ThrowCocotFromCar__7C_human` |
| `pedestrian_advance_to_next_node` | `SetNewNode__17C_traffic_element` |
| `pedestrian_tick` | `Tick__17C_traffic_element` |
| `car_hit` | `Hit__5C_car` |
| `car_game_done` | `GameDone__5C_car` |
| `game_done` | `Done__6C_game` |
| `ai_group_get_closest_usable_car` | `GetCarClosestAndUsable__8ai_group` |
| `pedestrian_manager_try_send_pedestrian_to_parked_car` | `TryGoToAssHole__19C_traffic_generator` |
| `police_manager_npc_list_push` / `_erase` / `_tick` | `AddDoRiciActor` / `DelDoRiciActor` / `TickDoRiciActors__10C_shvestky` |

**PS2-only methods.**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0022a770 | `ChangeSceneInit__13C_traffic_carFv` | PS2-port city streaming: `InterCity` occurs 0 times on the PC |
| 0x0022a800 | `ChangeSceneDone__13C_traffic_carFv` | PS2-port city streaming: `InterCity` occurs 0 times on the PC |
| 0x0022a8e0 | `InvalidateActor__13C_traffic_carFP7C_actor` | `game_invalidate_actor` has no kind-`0xc` arm (above) |
| 0x00228890 | `DejSiZatimSlofiiika__13C_traffic_carFb` | called only by `G_GarageClass::UdelejBum`, PS2-only (`structs/garage.md` §6c) |
| 0x0032ea90 | `DajDevatemuTyZkurvenyZmrdyAtFurtNevotravuje__13C_traffic_carFv` | 12 B in a second PS2 cluster; sole caller `actionTick_2__8C_entity` |

## 10. Related

`structs/actor.md` · `structs/pedestrian.md` · `structs/driver.md` · `structs/anim_machine.md` · `structs/roads.md` · `structs/bridge.md` · `structs/car.md`.

## 11. Open items

1. In game: does the cop-car counter cap reinforcements? `g_traffic_cop_car_count` is watchable in Cheat Engine, and the deploy path increments it once per car.
