# `police_manager` and `pursuit_record` — the player-side police engine

**Created** 2026-07-20 19:59 +0200 · **Modified** 2026-11-08 21:54 +0100

`police_manager` is the police engine embedded once in the `C_game` singleton at `C_game + 0x2da0` (reached as `*(g_act_mission+0x24)+0x2da0`, `g_act_mission` = `0x0065115c`), not per actor. `pursuit_record` is one heap record per tracked suspect, owned by the manager's record vector. Field names match the rest of the police docs. The squad side is `structs/police-board.md` and `structs/police-action.md`.

| | |
|---|---|
| PC prefix | `police_manager_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `police_manager` `0xb0`, embedded in `C_game`; `pursuit_record` `0x80`, heap |
| PC constructor | `police_manager_ctor`; `pursuit_record` is built in `police_manager_register_actor` |
| PC translation unit | `[0x005cad40, 0x005d14a0)`, 66 starts; three accessors of the manager live outside it (§3b) |
| PS2 class | `C_shvestky` (tag `10C_shvestky`), 65 methods; its record `S_shvestky` = `pursuit_record` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `police_manager`

```c
typedef struct police_manager {      // 0xb0 bytes, embedded at C_game + 0x2da0, no vtable
    /* +0x00 */ uint8_t  enabled; // [H] master switch; every helper returns early when it is 0
    /* +0x04 */ uint32_t tick_acc_inner; // [H] accumulator of the inner tick, ms
    /* +0x08 */ uint32_t tick_period_inner; // [H] inner period, 330 ms; also the step of nine record timers
    /* +0x0c */ float    speed_threshold_fine; // [H] speed above which speeding is a fine, 17 m/s
    /* +0x10 */ float    speed_threshold_arrest; // [H] speed above which speeding is handcuffs, 28 m/s
    /* +0x14 */ uint32_t tick_acc_outer; // [H] accumulator of the outer tick, ms
    /* +0x18 */ uint32_t tick_period_outer; // [H] outer period, 913 ms
    /* +0x1c */ uint32_t tick_acc_visibility; // [H] accumulator of the visibility tick, ms
    /* +0x20 */ uint32_t tick_period_visibility; // [H] visibility period, 217 ms
    /* +0x24 */ uint32_t tick_acc_vestigial; // [M] accumulator of a pair; no reader (.text scan)
    /* +0x28 */ uint32_t tick_period_vestigial; // [M] period of the pair, 4013 ms; no reader (.text scan)
    /* +0x2c */ uint32_t unread_2c; // [U] police_manager_game_init stores 0; no reader (.text scan)
    /* +0x30 */ uint32_t unread_30; // [U] police_manager_game_init stores 0; no reader (.text scan)
    /* +0x34 */ uint32_t arrest_text_id; // [H] textdb id the arrest end-screen shows; policemanager_on xx
    /* +0x38 */ uint8_t  busted_cutscene_lock; // [H] 1 while the player is in a capture cutscene (§7)
    /* +0x39 */ uint8_t  vehicle_tier; // [H] backup vehicle-pool selector; policemanager_on yy
    /* +0x3c */ uint8_t  records_alloc; // [H] allocator byte of the record vector
    /* +0x40 */ pursuit_record ** records_begin; // [H] one record per tracked suspect
    /* +0x44 */ pursuit_record ** records_end; // [H] one past the last record
    /* +0x48 */ pursuit_record ** records_cap; // [H] end of the record vector's storage
    /* +0x4c */ uint8_t  police_vehicles_alloc; // [H] allocator byte of the vehicle pool
    /* +0x50 */ S_police_pool_element * police_vehicles_begin; // [H] reserve pool of patrol cars
    /* +0x54 */ S_police_pool_element * police_vehicles_end; // [H] one past the last element
    /* +0x58 */ S_police_pool_element * police_vehicles_cap; // [H] end of the pool's storage
    /* +0x5c */ uint8_t  spike_strips_alloc; // [H] allocator byte of the spike-strip pool
    /* +0x60 */ S_police_pool_element * spike_strips_begin; // [H] reserve pool of spike strips, barrier.4ds
    /* +0x64 */ S_police_pool_element * spike_strips_end; // [H] one past the last element
    /* +0x68 */ S_police_pool_element * spike_strips_cap; // [H] end of the pool's storage
    /* +0x6c */ uint8_t  foot_cops_alloc; // [H] allocator byte of the foot-cop pool
    /* +0x70 */ S_police_pool_element * foot_cops_begin; // [H] reserve pool of foot cops
    /* +0x74 */ S_police_pool_element * foot_cops_end; // [H] one past the last element
    /* +0x78 */ S_police_pool_element * foot_cops_cap; // [H] end of the pool's storage
    /* +0x7c */ uint8_t  npc_list_alloc; // [H] allocator byte of the despawn list
    /* +0x80 */ C_actor ** npc_list_begin; // [H] temporary cars and NPCs to despawn out of range (§6)
    /* +0x84 */ C_actor ** npc_list_end; // [H] one past the last entry
    /* +0x88 */ C_actor ** npc_list_cap; // [H] end of the list's storage
    /* +0x8c */ uint8_t  tracked_police_cars_alloc; // [H] allocator byte of the tracked-car vector
    /* +0x90 */ C_car   ** tracked_police_cars_begin; // [H] the police cars on the radar; the tick evicts above five
    /* +0x94 */ C_car   ** tracked_police_cars_end; // [H] one past the last car
    /* +0x98 */ C_car   ** tracked_police_cars_cap; // [H] end of the vector's storage
    /* +0x9c */ uint8_t  visibility_cells_alloc; // [H] allocator byte of the visibility cache
    /* +0xa0 */ uint16_t * visibility_cells_begin; // [H] the visibility-cell indices
    /* +0xa4 */ uint16_t * visibility_cells_end; // [H] one past the last index
    /* +0xa8 */ uint16_t * visibility_cells_cap; // [H] end of the cache's storage
} police_manager;
```

`+0x3a`, `+0x3b` and `+0xac..+0xaf` have no access: padding.

### 1b. `pursuit_record`

```c
typedef struct pursuit_record {      // 0x80 bytes, operator_new in police_manager_register_actor
    /* +0x00 */ C_actor * target; // [H] the tracked actor, the match key
    /* +0x04 */ C_car   * linked_vehicle; // [H] the car the suspect drives, as actor+0x98
    /* +0x08 */ uint16_t offense_bits_a; // [H] offence bits, 1 << type id; the short-decay lane
    /* +0x0a */ uint16_t offense_bits_b; // [H] long-decay mirror of collision bits 0x08, 0x10, 0x20
    /* +0x0c */ uint32_t car_theft_timer; // [H] car-theft offence window, ms
    /* +0x10 */ uint32_t last_fire_time_ms; // [H] gunfire countdown, ms
    /* +0x14 */ uint16_t on_sidewalk_timer; // [H] time on the sidewalk, ms, clamped at 5000
    /* +0x18 */ uint32_t collision_short_timer; // [H] short collision-bit decay, about 4 s
    /* +0x1c */ uint32_t last_crash_time_ms; // [H] long collision-bit decay, about 20 s
    /* +0x20 */ uint32_t violence_event_timer; // [H] violence offence decay, ms
    /* +0x24 */ uint32_t running_red_or_sign_damage_timer; // [H] shared decay of the red-light and sign bits
    /* +0x28 */ uint32_t capture_count; // [H] captures of the player; flagged wipes of an NPC record (§2b)
    /* +0x2c */ uint8_t  roadblock_eligible_flag; // [H] the gun-tier roadblock gate
    /* +0x34 */ float    last_seen_x; // [H] the suspect's world X when last linked or seen
    /* +0x38 */ float    last_seen_z; // [H] the suspect's world Z; the record keeps no Y
    /* +0x3c */ uint16_t police_state; // [H] committed level: 1 fine, 2 handcuffs, 3 gun
    /* +0x3e */ uint16_t police_state_pending; // [H] pending level, re-derived each tick from the bits
    /* +0x40 */ int32_t  collision_episode_timer; // [H] window grouping collisions, about 5 s; -1 = idle
    /* +0x44 */ uint32_t wanted_bar_remaining; // [H] wanted-bar countdown, ms; -1 = not running
    /* +0x48 */ uint32_t wanted_bar_max; // [H] wanted-bar maximum: 20000, 40000 or 60000
    /* +0x4c */ uint32_t collision_episode_count; // [M] collisions in the episode; no reader (closed record set)
    /* +0x50 */ uint8_t  wanted_substate_flag; // [H] the wanted substate: all cops alerted
    /* +0x51 */ uint8_t  identity_pending; // [H] 1 = searching, 0 = known
    /* +0x54 */ C_car   * identity_tracked_vehicle; // [H] the vehicle the suspect is identified with
    /* +0x58 */ uint8_t  identity_on_foot_byte; // [H] face-known flag used with no living officer
    /* +0x5c */ uint32_t reinforcement_counter; // [H] backup-wave latch: 0 none, 1 handcuffs, 2 gun
    /* +0x60 */ C_car   * struck_police_car; // [H] the police car hit; the next dispatch takes and clears it
    /* +0x64 */ uint32_t action_code; // [H] offence display code; the string is 2100 + code
    /* +0x68 */ uint32_t run_over_count; // [H] pedestrians run over this episode; 6 or more means gun
    /* +0x6c */ uint8_t  severe_impact; // [H] the police-car hit was above 9.7
    /* +0x70 */ uint8_t  officers_alloc; // [H] allocator byte of the officer vector
    /* +0x74 */ C_actor ** officers_begin; // [H] the cops chasing this suspect
    /* +0x78 */ C_actor ** officers_end; // [H] one past the last officer
    /* +0x7c */ C_actor ** officers_cap; // [H] end of the vector's storage
} pursuit_record;
```

`+0x2d..+0x33`, `+0x52`, `+0x53`, `+0x59..+0x5b` and `+0x6d..+0x6f` have no access: padding. Offence storage, escalation and strings: `police-system/artifacts/offences.md`.

### 1c. `S_police_pool_element`

The stride-8 element of the three reserve pools (§6). The object pointer is a `vehicle_controller *`, a `spike_strip *` or an NPC, by pool.

```c
typedef struct S_police_pool_element { // 8 bytes, inserted by std_vector8_insert_at
    /* +0x00 */ void    *object;          // [H] the pooled object
    /* +0x04 */ uint8_t  available;       // [H] 1 in the pool, 0 handed out
    /* +0x05 */ uint8_t  model_flag;      // [H] vehicle pool only; see §6
} S_police_pool_element;
```

## 2. Field evidence

Confidence: H measured on at least two independent sites, M one site, U no consumer.

### 2a. `police_manager`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | enabled | H | `police_manager_apply_state`; `police_manager_game_init` (→0) | `police_manager_is_enabled` (dedicated getter); plus the inline `if (*this == 0) return;` early-out idiom shared by the `police_manager_*` family (§4) | 0 = disabled, 1 = enabled. |
| +0x4 | tick_acc_inner | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` (acc += dt); `police_manager_register_assault_on_police_officer_for_target` and `police_manager_register_assault_on_police_car_for_target` (each adds `period - acc` when positive) | `police_manager_tick` | Inner tick accumulator. The two assault registrars push it so the inner tick fires on the next frame. |
| +0x8 | tick_period_inner | H | `police_manager_ctor` (200); `police_manager_game_init` (330) | `police_manager_tick` (overflow threshold + per-record timer step) | 330 ms operational; doubles as the decrement step for 9 per-record timers. |
| +0xc | speed_threshold_fine | H | `police_manager_set_speed_thresholds`; `police_manager_game_init` (17.0f) | `police_manager_tick` (speeding check: `vehicle+0x688 > +0x0C` → speeding bit / fine) | ~17 m/s. Set via `policemanager_setspeed`. |
| +0x10 | speed_threshold_arrest | H | `police_manager_set_speed_thresholds`; `police_manager_game_init` (28.0f) | `police_manager_tick` (heavy-speeding check: `speed > +0x10` → handcuffs tier) | ~28 m/s. |
| +0x14 | tick_acc_outer | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` | `police_manager_tick` (first statement, unconditional) | Outer tick accumulator. |
| +0x18 | tick_period_outer | H | `police_manager_ctor` (200); `police_manager_game_init` (913) | `police_manager_tick` | 913 ms; handler = `police_manager_npc_list_tick`. |
| +0x1c | tick_acc_visibility | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` | `police_manager_tick` | Middle tick accumulator (gated; §5). |
| +0x20 | tick_period_visibility | H | `police_manager_ctor` (200); `police_manager_game_init` (217) | `police_manager_tick` | 217 ms; handler = visibility/LOS refresh. |
| +0x24 | tick_acc_vestigial | M | `police_manager_ctor` (→0); `police_manager_game_init` (→0) | - | Unused pair (§5); no reader (.text scan: no load of the manager's `+0x24` or of `C_game +0x2dc4`). |
| +0x28 | tick_period_vestigial | M | `police_manager_ctor` (200); `police_manager_game_init` (4013) | - | ~0.25 Hz if it were read; no reader (.text scan: no load of the manager's `+0x28` or of `C_game +0x2dc8`). |
| +0x2c | unread_2c | U | `police_manager_game_init` (0) | - | the ctor does not touch it; no reader (.text scan: the only `this` accesses are the two stores of `police_manager_game_init`, and `C_game +0x2dcc` has no access) |
| +0x30 | unread_30 | U | `police_manager_game_init` (0) | - | the ctor does not touch it; no reader (.text scan, as `+0x2c`; `C_game +0x2dd0` has no access) |
| +0x34 | arrest_text_id | H | `police_manager_apply_state` | `police_manager_tick` | the third argument of `gui_game_over_ctor` at the arrest end-screen, a textdb id |
| +0x38 | busted_cutscene_lock | H | `police_manager_set_busted_cutscene_lock` (→1 set / →0 clear, in `entity_action_tick` case 0x21 player capture — arrest OR fine); `police_manager_apply_state` (→0 on activate) | `police_manager_target_identity_pending`; `police_manager_tick`; `police_manager_try_dispatch_pursuit_for_target`; `police_manager_record_clear_pursuit_state` | 1 while the player is in a busted cutscene → suppresses the searching HUD icon, identity-pending and pursuit re-dispatch; else 0. §7. |
| +0x39 | vehicle_tier | H | `police_manager_apply_state` | `police_manager_apply_state` (model-pool select) | Backup-vehicle pool selector. |
| +0x3c | records_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x40 | records_begin | H | `police_manager_register_actor` | the `police_manager_*_for_target` record-lookup family (§4; enumerated in the function registry) | `std::vector<pursuit_record*>`. |
| +0x44 | records_end | H | `police_manager_register_actor` | the `police_manager_*_for_target` record-lookup family (§4; enumerated in the function registry) | `std::vector<pursuit_record*>`. |
| +0x48 | records_cap | H | `police_manager_register_actor` | the `police_manager_*_for_target` record-lookup family (§4; enumerated in the function registry) | `std::vector<pursuit_record*>`. |
| +0x4c | police_vehicles_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x50 | police_vehicles_begin | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` (free) | `police_manager_apply_state`; `police_manager_deploy_vehicle_from_pool`; `police_manager_vehicle_pool_pop_random` | Stride-8 element `{vehicle_controller *, available +4, model flag +5}` (§6). |
| +0x54 | police_vehicles_end | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` (free) | `police_manager_apply_state`; `police_manager_deploy_vehicle_from_pool`; `police_manager_vehicle_pool_pop_random` | Stride-8 element `{vehicle_controller *, available +4, model flag +5}` (§6). |
| +0x58 | police_vehicles_cap | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` (free) | `police_manager_apply_state`; `police_manager_deploy_vehicle_from_pool`; `police_manager_vehicle_pool_pop_random` | Stride-8 element `{vehicle_controller *, available +4, model flag +5}` (§6). |
| +0x5c | spike_strips_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x60 | spike_strips_begin | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_temp_spike_strip`; `police_manager_spike_strip_pool_pop` | Stride-8 element `{spike_strip *, available +4}` (§6). spike_strip = PS2 `C_irenka` (kind 0x1f). |
| +0x64 | spike_strips_end | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_temp_spike_strip`; `police_manager_spike_strip_pool_pop` | Stride-8 element `{spike_strip *, available +4}` (§6). spike_strip = PS2 `C_irenka` (kind 0x1f). |
| +0x68 | spike_strips_cap | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_temp_spike_strip`; `police_manager_spike_strip_pool_pop` | Stride-8 element `{spike_strip *, available +4}` (§6). spike_strip = PS2 `C_irenka` (kind 0x1f). |
| +0x6c | foot_cops_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x70 | foot_cops_begin | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_npc_from_pool`; `police_manager_officer_pool_pop_random` | Stride-8 element `{npc *, available +4}` (§6). **Not** cleared by `police_manager_game_init`. |
| +0x74 | foot_cops_end | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_npc_from_pool`; `police_manager_officer_pool_pop_random` | Stride-8 element `{npc *, available +4}` (§6). **Not** cleared by `police_manager_game_init`. |
| +0x78 | foot_cops_cap | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_npc_from_pool`; `police_manager_officer_pool_pop_random` | Stride-8 element `{npc *, available +4}` (§6). **Not** cleared by `police_manager_game_init`. |
| +0x7c | npc_list_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x80 | npc_list_begin | H | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_game_init` (clear) | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_npc_list_tick` | `std::vector<actor*>` of cars (kind 4) and `C_entity` NPCs (kind `0x1b`); elements non-owning. §6. |
| +0x84 | npc_list_end | H | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_game_init` (clear) | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_npc_list_tick` | `std::vector<actor*>` of cars (kind 4) and `C_entity` NPCs (kind `0x1b`); elements non-owning. §6. |
| +0x88 | npc_list_cap | H | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_game_init` (clear) | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_npc_list_tick` | `std::vector<actor*>` of cars (kind 4) and `C_entity` NPCs (kind `0x1b`); elements non-owning. §6. |
| +0x8c | tracked_police_cars_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x90 | tracked_police_cars_begin | H | `police_manager_add_tracked_police_car`; `police_manager_dtor` | `police_manager_apply_state`, `police_manager_game_done`, `police_manager_dtor`, `police_manager_tick` | `police_manager_game_done` walks it calling `hud_radar_remove_car` + `game_remove_temporary_actor`; `police_manager_tick` does the same to the oldest while more than five remain. §12d |
| +0x94 | tracked_police_cars_end | H | `police_manager_add_tracked_police_car`, `police_manager_dtor`, `police_manager_tick` | `police_manager_apply_state`, `police_manager_game_done`, `police_manager_dtor`, `police_manager_tick` | - |
| +0x98 | tracked_police_cars_cap | H | `police_manager_add_tracked_police_car`; `police_manager_dtor` | `police_manager_apply_state`; `police_manager_game_done`; `police_manager_dtor` | The manager's **tracked police cars**: `police_manager_game_done` walks it calling `hud_radar_remove_car` + `game_remove_temporary_actor` on each entry. §12d. |
| +0x9c | visibility_cells_alloc | H | `police_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0xa0 | visibility_cells_begin | H | `police_manager_apply_state` + `police_manager_tick` visibility refresh (via `std_vector2_insert_at`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_tick` visibility refresh; `police_manager_dtor` | `std::vector<uint16_t>` (stride-2) — visibility-cell indices. |
| +0xa4 | visibility_cells_end | H | `police_manager_apply_state` + `police_manager_tick` visibility refresh (via `std_vector2_insert_at`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_tick` visibility refresh; `police_manager_dtor` | `std::vector<uint16_t>` (stride-2) — visibility-cell indices. |
| +0xa8 | visibility_cells_cap | H | `police_manager_apply_state` + `police_manager_tick` visibility refresh (via `std_vector2_insert_at`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_tick` visibility refresh; `police_manager_dtor` | `std::vector<uint16_t>` (stride-2) — visibility-cell indices. |

### 2b. `pursuit_record`

The record set is closed: no `police_manager_*` function hands a record pointer outside the class and every getter returns a field value, so the readers and writers below are those of the 68 bodies of §3, read whole.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | target | H | `police_manager_register_actor` | the `police_manager_*_for_target` family's find-loop (`*(int*)record == target`; §1, enumerated in the function registry) | Match key. |
| +0x4 | linked_vehicle | H | `police_manager_register_actor` (if `actor+0x98` is type 4); `police_manager_link_target_to_vehicle` (car or 0; also sets or clears car `+0x2035`) | `police_manager_register_collision_offence_for_target` and `police_manager_register_sidewalk_offence_for_target` (match key); `police_manager_tick` (speed `+0x688`, the red-light block's frame `+0x68`, heading `+0x1f7c..+0x1f84`, the sidewalk byte `+0x20b0`, the police-car waiver `+0x1ff2`) | The car the suspect drives; pos captured at +0x34/+0x38. |
| +0x8 | offense_bits_a | H | `police_manager_register_collision_offence_for_target` (0x08/0x10/0x20); `police_manager_register_gunfire_offence_for_target` (0x02); `police_manager_register_violence_offence_for_target` (0x200); `police_manager_register_assault_on_police_officer_for_target` (0x2000); `police_manager_register_assault_on_police_car_for_target` (0x8000); `police_manager_escalate_handcuffs_with_optional_wanted` (0x4000); `police_manager_register_traffic_sign_offence_for_target` (0x1000); `police_manager_set_car_theft_offence_for_target` (0x800); `police_manager_register_sidewalk_offence_for_target` (0x40); `police_manager_tick` (live 0x01 weapon / 0x04 speeding) | `police_manager_get_offense_bits_for_target` | Short-decay lane. `bit = 1<<offence type id`. Full per-bit setter map: `police-system/artifacts/offences.md` §2a. |
| +0xa | offense_bits_b | H | `police_manager_register_collision_offence_for_target` (lockstep with bits_a) | `police_manager_get_offense_bits_for_target` (OR with bits_a) | Long-decay (~20 s) collision mirror. `police-system/artifacts/offences.md` §3. |
| +0xc | car_theft_timer | H | `police_manager_set_car_theft_offence_for_target` (5000/0); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (0); `police_manager_tick` (countdown) | `police_manager_tick` (counts it down, clears bit 11 at 0) | Car-theft window. |
| +0x10 | last_fire_time_ms | H | `police_manager_register_actor` (0); `police_manager_register_gunfire_offence_for_target` (+5000, clamp 60000); `police_manager_tick` (minus `+0x08` per inner tick, floor 0) | `police_manager_get_firetime_for_target` | Countdown in ms; 0 = no gunfire window. |
| +0x14 | on_sidewalk_timer | H | `police_manager_register_actor` (0); `police_manager_register_sidewalk_offence_for_target`; `police_manager_tick` (grows on surface, clamp 5000) | `police_manager_tick` (sidewalk fine gate) | Tied to bit 0x40. `police-system/artifacts/offences.md` §2a/§8. |
| +0x18 | collision_short_timer | H | `police_manager_register_collision_offence_for_target` (max of the arm's window, clamp 4000); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (0); `police_manager_tick` (countdown; 2000 cap while draining; 0 when dormant) | `police_manager_tick` (clears bits `0x38` at 0); `police_manager_register_collision_offence_for_target` (its max test) | ~4 s collision window. |
| +0x1c | last_crash_time_ms | H | `police_manager_register_actor` (0); `police_manager_register_collision_offence_for_target` (clamp 20000); `police_manager_tick` (minus `+0x08` per inner tick, floor 0) | `police_manager_get_crashtime_for_target`; `police_manager_tick` | ~20 s long collision window; a countdown in ms. |
| +0x20 | violence_event_timer | H | `police_manager_register_violence_offence_for_target` (+5000, no clamp); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (0); `police_manager_tick` (countdown, 2000 cap) | `police_manager_tick` (clears bit 9 on expiry) | Accumulates. |
| +0x24 | running_red_or_sign_damage_timer | H | `police_manager_register_traffic_sign_offence_for_target` (+4000, clamp 20000); `police_manager_tick` red-light block (+4000, clamp 10000); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (0) | `police_manager_tick` (clears bits 10 + 12 together) | Shared decay for red-light + sign bits. |
| +0x28 | capture_count | H | `police_manager_record_clear_pursuit_state` (`inc [esi+0x28]` at `0x005d0a91`, when its flag is set and the record is not the player's or `busted_cutscene_lock` is set); `police_manager_register_actor` (0) | `police_manager_get_capture_count` (`0x005ce6d7`; callers `human_notify_callback` animation events `0x5f` / `0x60` / `0x61`, which pick a `human_new_scream` line by the value 1 / 2 / 3 on the player's record); `police_manager_tick` (`cmp [eax+0x28],3` at `0x005cc493`: a pending fine becomes handcuffs above 2) | `entity_action_tick` case `0x21` (player capture, arrest or fine) calls `police_manager_set_busted_cutscene_lock(1)` and then `police_manager_record_clear_pursuit_state(player, 1)`, so the player's record counts every capture; a non-player record counts every flagged wipe (the tick's bar-drain clear). |
| +0x2c | roadblock_eligible_flag | H | `police_manager_enable_roadblock_flag_for_target` (1 when substate + state ≥ gun); `police_manager_record_clear_pursuit_state` (→0) | `police_manager_tick` (per-cell roadblock scan, sole reader) | Functional roadblock-eligibility gate. |
| +0x34 | last_seen_x | H | `police_manager_link_target_to_vehicle`, `police_manager_tick` | - | the first component of the position vector; no reader (closed record set) |
| +0x38 | last_seen_z | H | `police_manager_link_target_to_vehicle`, `police_manager_tick` | - | the third component of the position vector, so the Z; `police_manager_tick` stores both at its last-seen tail; no reader (closed record set) |
| +0x3c | police_state | H | `police_manager_record_raise_police_state` (loud raisers); `police_manager_tick` (commit from +0x3e) | `police_manager_get_committed_police_state`; `police_manager_enable_roadblock_flag_for_target`; `police_manager_try_dispatch_pursuit_for_target` (radius select) | Committed icon level. Distinct from the wanted substate. |
| +0x3e | police_state_pending | H | `police_manager_register_collision_offence_for_target`, `police_manager_register_gunfire_offence_for_target`, `police_manager_register_violence_offence_for_target`, `police_manager_register_traffic_sign_offence_for_target` (1/2/3, escalate-only); `police_manager_tick` escalation walk (then zeroed each pass) | `police_manager_tick` (commit + dispatch level); `police_manager_register_collision_offence_for_target` (escalate guard) | Re-derived from current offence bits each inner tick. |
| +0x40 | collision_episode_timer | H | `police_manager_register_actor` (-1); `police_manager_register_collision_offence_for_target` (5000 on first hit) | `police_manager_register_collision_offence_for_target` (`<0` first-vs-repeat); `police_manager_tick` (countdown) | ~5 s collision grouping window. |
| +0x44 | wanted_bar_remaining | H | `police_manager_register_actor` (-1); `police_manager_record_begin_wanted_substate`; `police_manager_tick` (= max / refill / drain) | `police_manager_tick` (bar numerator) | Drains while unseen; `<0` ⇒ clear. |
| +0x48 | wanted_bar_max | H | `police_manager_record_begin_wanted_substate`; `police_manager_tick` | `police_manager_tick` (bar denominator) | State-keyed 20000/40000/60000. |
| +0x4c | collision_episode_count | M | `police_manager_register_collision_offence_for_target` (1 on the first hit of an episode, +1 after); `police_manager_tick` (→0 when `+0x40` closes); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (0) | - | No reader outside its own increment (closed record set). |
| +0x50 | wanted_substate_flag | H | `police_manager_register_actor` (0); `police_manager_record_begin_wanted_substate`; `police_manager_escalate_handcuffs_with_optional_wanted` | `police_manager_is_wanted_substate_active_for_target`; `police_manager_tick` (bar + HUD wrap); `police_manager_enable_roadblock_flag_for_target` | "All cops alerted." Not the icon level (+0x3c). |
| +0x51 | identity_pending | H | `police_manager_register_actor` (0); `police_manager_tick` (identity recompute, four store sites, §8); `police_manager_record_clear_pursuit_state` (0) | `police_manager_target_identity_pending`; `police_manager_tick` (searching override); `police_manager_try_dispatch_pursuit_for_target` | 1 = UNKNOWN/searching, 0 = KNOWN. The writer set is closed: a byte-store census of the whole binary finds exactly these six stores on a `pursuit_record`. |
| +0x54 | identity_tracked_vehicle | H | `police_manager_record_set_tracked_vehicle_for_target` (= squad `police_action+0x64`, handed back when a squad ends: `police_board_end_pursuit`, `police_action_disband` with-car path) | `police_manager_tick` (in-car identity recompute) | Written only when the target's current vehicle differs from the stored `+0x54`. §8. `police_manager_register_actor` initialises neither `+0x54` nor `+0x58` (nor `+0x34`, `+0x38`, `+0x48`, `+0x60`, `+0x64`, `+0x6c`). |
| +0x58 | identity_on_foot_byte | H | `police_manager_record_set_tracked_vehicle_for_target` (= squad `police_action+0x68`, 0 or 1, same guard and callers as `+0x54`) | `police_manager_tick` (on-foot identity recompute) | Face-known flag used while no living officer is assigned. §8. |
| +0x5c | reinforcement_counter | H | `police_manager_register_actor` (0); `police_manager_tick` (1 at handcuffs+wanted, 2 at gun+wanted) | `police_manager_tick` (`<1`/`<2` spawn gates) | Backup-wave latch; gates `police_manager_spawn_patrol_car_reinforcements_at_radius`. |
| +0x60 | struck_police_car | H | `police_manager_register_assault_on_police_car_for_target`, `police_manager_record_clear_pursuit_state`, `police_manager_try_dispatch_pursuit_for_target` | `police_manager_try_dispatch_pursuit_for_target` | the dispatch hands it on and zeroes it |
| +0x64 | action_code | H | `police_manager_register_gunfire_offence_for_target` (1); `police_manager_register_violence_offence_for_target` (5); `police_manager_register_traffic_sign_offence_for_target` (8); `police_manager_register_collision_offence_for_target` (3); `police_manager_tick` escalation (0/2/4/6/7/9/10) | `police_manager_try_dispatch_pursuit_for_target` (string = 2100+code) | `police-system/artifacts/offences.md` §5.2. |
| +0x68 | run_over_count | H | `police_manager_increase_run_over_count_for_target` (+1 per NPC struck); `police_manager_register_actor` (0); `police_manager_record_clear_pursuit_state` (reset) | `police_manager_register_collision_offence_for_target` (`<6` gate) | `>=6` ⇒ straight to gun. |
| +0x6c | severe_impact | H | `police_manager_register_assault_on_police_car_for_target` | `police_manager_tick` | `9.7 < impact`; hits under 2.8 do not reach the store |
| +0x70 | officers_alloc | H | `police_manager_register_actor` | - | The allocator byte that opens the officers `std::vector` `{+0x70 allocator, +0x74 begin, +0x78 end, +0x7c cap}`; the constructor copies an empty allocator object into it, which is why the decompiler shows an arbitrary byte (the low byte of the target pointer). It carries no value. |
| +0x74 | officers_begin | H | `police_manager_register_actor` (0); `police_manager_record_add_officer` (from `police_manager_try_dispatch_pursuit_for_target`, `police_manager_spawn_patrol_car_reinforcements_at_radius`, `police_manager_add_officer_to_target`); `police_manager_record_delete_officer` (via `police_manager_remove_officer_from_all_records`); `police_manager_tick` (prune of death-latched officers) | `police_manager_count_officers_for_target`; `police_manager_count_officers_alive_for_target`; `police_manager_tick` (officer-count gate, via `vec_count_stride_0x4`) | `std::vector<officer*>` of cops assigned to this target; cap inferred. A living cop leaves it only when it dies or is released (`+0xf48` bit 3) and culled by `police_manager_npc_list_tick`. |
| +0x78 | officers_end | H | `police_manager_register_actor` (0); `police_manager_record_add_officer` (from `police_manager_try_dispatch_pursuit_for_target`, `police_manager_spawn_patrol_car_reinforcements_at_radius`, `police_manager_add_officer_to_target`); `police_manager_record_delete_officer` (via `police_manager_remove_officer_from_all_records`); `police_manager_tick` (prune of death-latched officers) | `police_manager_count_officers_for_target`; `police_manager_count_officers_alive_for_target`; `police_manager_tick` (officer-count gate, via `vec_count_stride_0x4`) | `std::vector<officer*>` of cops assigned to this target; cap inferred. A living cop leaves it only when it dies or is released (`+0xf48` bit 3) and culled by `police_manager_npc_list_tick`. |
| +0x7c | officers_cap | H | `police_manager_register_actor` (0); `police_manager_record_add_officer` (from `police_manager_try_dispatch_pursuit_for_target`, `police_manager_spawn_patrol_car_reinforcements_at_radius`, `police_manager_add_officer_to_target`); `police_manager_record_delete_officer` (via `police_manager_remove_officer_from_all_records`); `police_manager_tick` (prune of death-latched officers) | `police_manager_count_officers_for_target`; `police_manager_count_officers_alive_for_target`; `police_manager_tick` (officer-count gate, via `vec_count_stride_0x4`) | `std::vector<officer*>` of cops assigned to this target; cap inferred. A living cop leaves it only when it dies or is released (`+0xf48` bit 3) and culled by `police_manager_npc_list_tick`. |

### 2c. `S_police_pool_element`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | object | H | `police_manager_apply_state` | `police_manager_vehicle_pool_pop_random`, `police_manager_spike_strip_pool_pop`, `police_manager_officer_pool_pop_random`, `police_manager_acquire_from_pool_by_name` | - |
| +0x4 | available | H | `police_manager_apply_state`, the three pops, `police_manager_free_temp_car`, `police_manager_free_temp_spike_strip`, `police_manager_free_temp_enemy` | the three pops, `police_manager_get_free_officers_count` | - |
| +0x5 | model_flag | H | `police_manager_apply_state` | `police_manager_vehicle_pool_pop_random` | read only with a non-zero argument, and the one caller passes 0 |

## 3. Method family

All 65 `C_shvestky` methods are placed: in the tables below, as `inlined` rows, or in §12c. A row with no twin is PC-only. `pedestrian_manager_spawn_cop_from_pedestrian` (`structs/pedestrian.md`) and `traffic_car_activate_or_spawn_pursuit_vehicles` (`structs/traffic_car.md`) are driven by the manager and belong to their own classes.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005cad40 | `crt_static_init_guard_005cad40` | - | - | - | - | opening guard of the unit |
| 0x005cad60 | `crt_atexit_register_005cad60` | - | - | - | - | the guard's `atexit` registrar |
| 0x005cad70 | `police_manager_ctor` | instance | `__ct__10C_shvestkyFv` | 0x001f99b0 | CONFIRMED | ctor — scalar inits + 7 std::vector zero-inits |
| 0x005cae30 | `police_manager_dtor` | instance | `__dt__10C_shvestkyFv` | 0x001f9be0 | CONFIRMED | dtor — reverse teardown of the 7 vectors |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425500 | `police_manager_set_busted_cutscene_lock` | - | - | - | stores `busted_cutscene_lock` (§7) |
| 0x005c8af0 | `police_manager_is_enabled` | - | - | - | getter — returns police_manager+0x00 `enabled` byte |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8b00 | `police_manager_set_speed_thresholds` | - | - | - | set speeding thresholds +0x0c fine / +0x10 arrest (m/s); backs POLICEMANAGER_SETSPEED |
| 0x005caf00 | `police_manager_game_init` | `GameInit__10C_shvestkyFv` | 0x001f9cf0 | CONFIRMED | soft reset keeping the buffers, seeds the periods and speeds |
| 0x005caff0 | `police_manager_game_done` | `GameDone__10C_shvestkyFv` | 0x001f9d80 | CONFIRMED | heavy teardown: records, officers, `npc_list`, tracked cars, cache and pools |
| 0x005cb330 | `police_manager_record_free_officers` | - | - | - | dtor sub-step — free a pursuit_record's officer vector (rec+0x74) + zero the triple |
| 0x005cb350 | `police_manager_add_tracked_police_car` | `DezoSebralGumyIRadio__10C_shvestkyFP5C_car` | 0x001fa940 | CONFIRMED | dedup-append of a police car to the manager's tracked-car vector (+0x90) |
| 0x005cb510 | `police_manager_tick` | `Tick__10C_shvestkyFUi` | 0x001faa30 | STRUCTURAL | per-frame manager tick (PS2 SPLIT across Tick + ai_police_nastenka::Tick + ai_police_action::Tick; PC fuses) |
| - | - | `CreateShvestkyTest__10C_shvestkyFv` | 0x001fb6e0 | inlined | host `police_manager_tick`: the escalation-decision block |
| - | - | `TestOverRed__10C_shvestkyFv` | 0x001fc1c0 | inlined | host `police_manager_tick`: the red-light detector; it detects, it does not register the offence |
| 0x005cc9e0 | `police_manager_apply_state` | `SetActive__10C_shvestkyFbii` | 0x001fcb20 | STRUCTURAL | activate / tear down police presence (4388 B PS2) |
| 0x005cd960 | `police_manager_register_actor` | `AddPerson__10C_shvestkyFP7C_human` | 0x001fdc50 | CONFIRMED | alloc + insert a pursuit_record (PS2 0x7c / PC 0x80) |
| 0x005cdbd0 | `police_manager_unregister_actor` | `DelPerson__10C_shvestkyFP7C_human` | 0x001fddf0 | CONFIRMED | return officers to pool, free vector, swap-remove |
| 0x005cdca0 | `police_manager_register_gunfire_offence_for_target` | `Firing__10C_shvestkyFP7C_human` | 0x001fdf20 | CONFIRMED | gunfire offence: bit 0x2, tier3, fire timer +=5000 |
| 0x005cdd30 | `police_manager_link_target_to_vehicle` | `DriveCar__10C_shvestkyFP7C_humanP5C_car` | 0x001fe010 | CONFIRMED | link/unlink the ridden vehicle (rec+0x04) |
| 0x005cde10 | `police_manager_register_collision_offence_for_target` | `CarCollision__10C_shvestkyFP5C_carPUif` | 0x001fe120 | CONFIRMED | collision offence: key on rec+0x04, bit 0x8 |
| 0x005ce150 | `police_manager_register_sidewalk_offence_for_target` | `CarIllegalPath__10C_shvestkyFP5C_car` | 0x001fe600 | CONFIRMED | sidewalk/illegal-path offence: bit 0x40 |
| 0x005ce1b0 | `police_manager_register_violence_offence_for_target` | `HumanBoxing__10C_shvestkyFP7C_human` | 0x001fe690 | CONFIRMED | melee offence: bit 0x200, tier2 |
| 0x005ce230 | `police_manager_set_car_theft_offence_for_target` | `CarSteal__10C_shvestkyFP7C_humanbb` | 0x001fe750 | CONFIRMED | car-theft offence: bit 0x800 |
| 0x005ce2b0 | `police_manager_escalate_handcuffs_with_optional_wanted` | `ForceArrest__10C_shvestkyFP7C_humanbb` | 0x001fe800 | CONFIRMED | arrest bit 0x4000; optional wanted substate rec+0x50 |
| 0x005ce320 | `police_manager_register_traffic_sign_offence_for_target` | `DestroySignalMark__10C_shvestkyFP7C_human` | 0x001fe8a0 | STRUCTURAL | traffic-sign offence: bit 0x1000 + tier1 + action8 + timer |
| 0x005ce3a0 | `police_manager_increase_run_over_count_for_target` | `NextNaSrackySoudruh__10C_shvestkyFP7C_human` | 0x001fe980 | CONFIRMED | +1 run_over_count (rec+0x68) per NPC struck |
| 0x005ce3e0 | `police_manager_report_police_car_stealing_sighting` | `CorujuPoldumMasinu__10C_shvestkyFP7C_humanP5C_car` | 0x001fe9f0 | CONFIRMED | report sighting of stealing of seated police car (§12a) |
| 0x005ce460 | `police_manager_enable_roadblock_flag_for_target` | `SerSeMuDoCesty__10C_shvestkyFP7C_human` | 0x001fead0 | CONFIRMED | set roadblock flag rec+0x2c (gated wanted substate + gun) |
| 0x005ce4c0 | `police_manager_get_offense_bits_for_target` | `GetHumanStatus__10C_shvestkyFP7C_human` | 0x001feb50 | CONFIRMED | return offense_bits_a OR offense_bits_b (rec+0x08 / rec+0x0a) |
| 0x005ce510 | `police_manager_get_committed_police_state` | `JakPicusNaTomJest__10C_shvestkyFP7C_human` | 0x001febb0 | CONFIRMED | return committed tier rec+0x3c |
| 0x005ce560 | `police_manager_get_crashtime_for_target` | `GetCrashTime__10C_shvestkyFP7C_human` | 0x001fec00 | CONFIRMED | return rec+0x1c |
| 0x005ce5a0 | `police_manager_get_firetime_for_target` | `GetFireTime__10C_shvestkyFP7C_human` | 0x001fec50 | CONFIRMED | return rec+0x10 |
| 0x005ce5e0 | `police_manager_count_officers_for_target` | `PocetNasranych__10C_shvestkyFP7C_human` | 0x001feca0 | CONFIRMED | assigned-officer count (rec+0x74) |
| 0x005ce630 | `police_manager_count_officers_alive_for_target` | `PocetNasranychZivych__10C_shvestkyFP7C_human` | 0x001fecf0 | CONFIRMED | count 'alive' officers in the vector |
| 0x005ce6a0 | `police_manager_get_capture_count` | `PocetPrestupku__10C_shvestkyFP7C_human` | 0x001fedd0 | CONFIRMED | return capture_count rec+0x28 |
| 0x005ce6e0 | `police_manager_target_identity_pending` | `NeviDoKterePiceVlezl__10C_shvestkyFP7C_human` | 0x001fee20 | CONFIRMED | identity-pending rec+0x51 (gated mgr+0x38==0) |
| 0x005ce730 | `police_manager_is_wanted_substate_active_for_target` | `ChceHoOpichatKazdyPoldaNaOkresku__10C_shvestkyFP7C_human` | 0x001fee90 | CONFIRMED | wanted substate rec+0x50 |
| 0x005ce770 | `police_manager_deploy_vehicle_from_pool` | `CreateCar__10C_shvestkyFR8S_vectorR8S_vectorb` | 0x001feee0 | CONFIRMED | deploy a patrol car from the `+0x50` pool |
| 0x005ce8d0 | `police_manager_deploy_temp_spike_strip` | - | - | - | deploy a spike strip from the +0x60 pool |
| 0x005ce9e0 | `police_manager_deploy_npc_from_pool` | - | - | - | deploy a foot cop from the `+0x70` pool |
| 0x005ceb00 | `police_manager_spawn_roadblock_in_neighbor_cells` | - | - | - | spawn a roadblock composite across neighbor cells |
| 0x005cf3c0 | `police_manager_acquire_from_pool_by_name` | `MasVolnouPlecku__10C_shvestkyFPc` | 0x001ff0b0 | CONFIRMED | name-keyed pool acquire + reactivate |
| 0x005cf540 | `police_manager_vehicle_pool_pop_random` | `GetTempCar__10C_shvestkyFb` | 0x001ff2c0 | STRUCTURAL | random-pop from the vehicle pool (+0x50) |
| 0x005cf610 | `police_manager_free_temp_car` | `FreeTempCar__10C_shvestkyFP5C_car` | 0x001ff450 | CONFIRMED | release a temp car (dispatch type 4) |
| 0x005cf6b0 | `police_manager_get_free_officers_count` | `GetFreeEnemy__10C_shvestkyFv` | 0x001ff550 | STRUCTURAL | count +4-flagged officer-pool entries |
| 0x005cf6e0 | `police_manager_spike_strip_pool_pop` | - | - | - | random-pop a spike strip from the +0x60 pool |
| 0x005cf720 | `police_manager_free_temp_spike_strip` | `FreeTempIris__10C_shvestkyFP8C_irenka` | 0x001ff5a0 | CONFIRMED | release a temp spike strip (dispatch type 0x1f) |
| 0x005cf790 | `police_manager_officer_pool_pop_random` | `GetTempEnemy__10C_shvestkyFv` | 0x001ff650 | STRUCTURAL | random-pop from the cop/enemy pool |
| 0x005cf810 | `police_manager_free_temp_enemy` | `FreeTempEnemy__10C_shvestkyFP8C_entity` | 0x001ff730 | CONFIRMED | release temp enemy NPC to ambient traffic (dispatch type 0x1b) |
| 0x005cf960 | `police_manager_free_temp_actor` | `FreeMyTempActor__10C_shvestkyFP7C_actor` | 0x001ff970 | CONFIRMED | kind-dispatch 4/0x1b/0x1f -> the three FreeTemp* |
| 0x005cf9a0 | `police_manager_npc_list_push` | `AddDoRiciActor__10C_shvestkyFP7C_actor` | 0x001ff9e0 | STRUCTURAL | unique-append writer for npc_list (+0x80); PS2 adds an actor to the DoRici squad (polymorphic on receiver) |
| 0x005cfb60 | `police_manager_npc_list_erase` | `DelDoRiciActor__10C_shvestkyFP7C_actor` | 0x001ffa70 | STRUCTURAL | erase-swap helper on npc_list (+0x80); PS2 removes an actor from the DoRici squad (polymorphic on receiver) |
| 0x005cfc00 | `police_manager_npc_list_tick` | `TickDoRiciActors__10C_shvestkyFb` | 0x001ffbf0 | STRUCTURAL | out-of-range despawn of `npc_list` members (outer tick, mode 0); mode 1 empties the list (§6) |
| 0x005cff60 | `police_manager_invalidate_discarded_npcs` | `ZlikvidujVsechnyNasraneVyhozeneCuraky__10C_shvestkyFv` | 0x00200110 | CONFIRMED | purge flagged NPCs from npc_list |
| 0x005d0060 | `police_manager_spawn_patrol_car_reinforcements_at_radius` | `DalsiChalpiDoPartie__10C_shvestkyFP7C_humanUif` | 0x00200240 | CONFIRMED | spawn one patrol car with two cops at a radius and engage it as a squad of the given tier (4 handcuffs / 0 gun); `police_manager_tick` gates it on reinforcement_counter (rec+0x5c) |
| 0x005d06c0 | `police_manager_remove_officer_from_all_records` | `GlobalDelNasrany__10C_shvestkyFP7C_actor` | 0x002009f0 | CONFIRMED | remove the given officer from EVERY pursuit_record's officer list (rec+0x74) |
| 0x005d0700 | `police_manager_add_officer_to_target` | - | - | - | add a cop to a target's keyed record squad (rec+0x74); the receiver is the manager, and no distinct PS2 twin exists |
| 0x005d0750 | `police_manager_record_add_officer` | `AddNasrany__Q210C_shvestky10S_shvestkyFP7C_actor` | 0x00200a60 | CONFIRMED | unique-append officer to rec+0x74 |
| 0x005d0900 | `police_manager_record_delete_officer` | `DelNasrany__Q210C_shvestky10S_shvestkyFP7C_actor` | 0x00200af0 | CONFIRMED | swap-remove officer from rec+0x74 |
| 0x005d0960 | `police_manager_record_raise_police_state` | `DamMuPokuritJesteVice__10C_shvestkyFP7C_humanb` | 0x00200b50 | CONFIRMED | monotonic raise committed tier rec+0x3c |
| 0x005d09d0 | `police_manager_record_clear_pursuit_state` | `DobreMiPokuril_MuzeJitDom__10C_shvestkyFP7C_humanb` | 0x00200c10 | CONFIRMED | de-escalate/release: zero record fields |
| 0x005d0b00 | `police_manager_try_dispatch_pursuit_for_target` | `VasekSelChlastatNaOslavu__10C_shvestkyFP7C_humanRC8S_vectorfbb` | 0x00200df0 | CONFIRMED | dispatch cops/cars that pass range+facing+LOS; rolls back the provisional police-state otherwise |
| 0x005d1100 | `police_manager_record_begin_wanted_substate` | `NabonzovalJsemHo__10C_shvestkyFP7C_human` | 0x00201600 | CONFIRMED | begin the wanted substate on a record; shared textdb id `0x898`, shared colour `0xffff0000` and the same 20000/40000/60000 ladder |
| 0x005d11c0 | `police_manager_register_assault_on_police_officer_for_target` | `DalJsemPochuzkariDoDrzky__10C_shvestkyFP7C_human` | 0x00201710 | CONFIRMED | assault-on-cop offence: bit 13 (`+9 \|= 0x20`) → pending GUN (3), action 9 |
| 0x005d1220 | `police_manager_register_assault_on_police_car_for_target` | `StrcilJsemHoPoldumDoVyfuku__10C_shvestkyFP7C_actorP5C_carf` | 0x002017a0 | CONFIRMED | assault-on-cop-car: bit 0x8000, rec ring-rotate |
| 0x005d12d0 | `police_manager_record_set_tracked_vehicle_for_target` | `ZalozSpisATahniDoRiti_Haleluja__10C_shvestkyFP7C_humanP7C_actorb` | 0x002018a0 | CONFIRMED | set rec+0x54 tracked vehicle + rec+0x58 on-foot byte |
| 0x005d1340 | `police_manager_are_cops_after_player` | `IdePoliciePoPlayeroviBoPumper__10C_shvestkyFv` | 0x00201920 | CONFIRMED | the predicate `pumper_can_pump` asks |
| 0x005d13b0 | `police_manager_rename_police_car` | `RenamePoliceCar__10C_shvestkyFP5C_car` | 0x002019f0 | CONFIRMED | give a pooled patrol car the unique frame name `corepoli_%04i` from `g_police_car_rename_counter`; called right after `police_manager_add_tracked_police_car` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005cc9a0 | `vec3_is_near_zero` | - | - | - | shared math |
| 0x005d1470 | `std_vector4_erase_at` | - | - | - | shared vector helper |


## 4. Where it lives

```
C_game object  (the game-world singleton, *(g_act_mission+0x24))
+----------------------------------------------------------+
| +0x2DA0  police_manager          embedded, single, 0xB0    |
|   +0x00  enabled               master on/off             |
|   +0x40  records[]             vector<pursuit_record*>   |
|   +0x50  police_vehicles[]     reserve pool (cars)       |
|   +0x60  spike_strips[]        reserve pool (barriers)   |
|   +0x70  foot_cops[]           reserve pool (npc*)       |
|   +0x80  npc_list[]            despawn list (actor*)     |
|   +0x90  tracked_police_cars[] cars the manager owns      |
|   +0xA0  visibility_cells[]    vector<uint16_t>          |
+----------------------------------------------------------+

records[] entries are pointers to heap pursuit_record structs:

pursuit_record   (heap, 0x80 bytes, one per tracked suspect)
+----------------------------------------------------------+
|   +0x00  target                the tracked actor         |
|   +0x04  linked_vehicle        car being driven          |
|   +0x08  offense_bits_a        offence bitfield          |
|   +0x0A  offense_bits_b        long-decay mirror         |
|   +0x3C  police_state          1 fine / 2 cuffs / 3 gun  |
|   +0x3E  police_state_pending  pending level             |
|   +0x44  wanted_bar_remaining  wanted-bar countdown      |
|   +0x50  wanted_substate_flag  'all cops alerted'        |
|   +0x51  identity_pending      1 searching / 0 known     |
|   +0x74  officers[]            vector<officer*>          |
+----------------------------------------------------------+
```

- **`police_manager`** is **embedded** (not separately allocated) at `C_game + 0x2DA0`, size
  **0xB0 (176)** — the next C_game member begins at `C_game + 0x2E50`. It is plain (no vtable;
  `+0x00` is the `enabled` byte). It is a **SINGLE instance** on the C_game game-world singleton
  (`*(g_act_mission+0x24)`, `g_act_mission` = `0x0065115c`), **not per-actor**.
- **`pursuit_record`** is heap-allocated by `operator_new(0x80)` in `police_manager_register_actor`.
  The manager owns a `std::vector<pursuit_record*>` at `+0x40/+0x44/+0x48`. One record per tracked
  suspect (the player auto-registers first).
- **Naming policy & what "helper" means:** the struct *type* is `police_manager`; a **helper** is a
  `__thiscall` member function of that class — its `this` (ECX) is a `police_manager*`
  (= `C_game+0x2DA0`). The project names this family `police_manager_*` (matching the
  `s_POLICEMANAGER_*` `.rdata` strings). The **`police_manager_*_for_target`** subfamily additionally
  take a target actor and look up that actor's `pursuit_record` in the records vector (`+0x40`) — so
  by construction every one of them reads `records_begin`/`records_end` and the record's
  `+0x00 target`. `police_manager_*` / `*_for_target` in the evidence tables therefore denote that whole
  defined family (not one function); the **authoritative enumerated list is the function registry**,
  and the specific members that touch each field are the named readers and writers cited in §2a / §2b.
  The separate `police_board_*` / `police_action_*` officer-squad system operates on its own
  objects (§10).

## 5. The per-frame tick scheduler

The eight scalar fields at `+0x04..+0x28` are **four `(accumulator, period_ms)` pairs**, three
active and read by `police_manager_tick`, one vestigial; two words, `unread_2c` and `unread_30`,
follow them. Each active pair runs the frame-rate-independent pattern (so the handler fires at `1000/period` Hz regardless of frame time):

```
acc += dt_ms;  if (period_ms < acc) { acc -= period_ms;  <handler>; }
```

| Pair | acc / period | period | rate | Overflow handler |
|---|---|---|---|---|
| Outer | +0x14 / +0x18 | 913 ms | ~1.1 Hz | `police_manager_npc_list_tick` — distance-cull `npc_list` vs camera. Runs unconditionally. |
| Middle | +0x1C / +0x20 | 217 ms | ~4.6 Hz | Visibility/LOS refresh; sets sighting markers; populates `visibility_cells`. Gated on not-busted-cutscene + render quality ≥ 10. |
| Inner | +0x04 / +0x08 | 330 ms | ~3.0 Hz | Per-record state machine: re-derive live offence bits, decay offence timers, escalate `pending → committed`, drive HUD/bar. `+0x08` also = the decrement step for 9 per-record timers. |

The vestigial pair (`+0x24/+0x28 = 4013 ms`) and the words `+0x2c` / `+0x30` are written at reset
and have no reader (.text scan). The four operational periods (330/913/217/4013) appear in the binary only inside
`police_manager_game_init` — no reload-on-zero re-arm sites, confirming accumulator
(not countdown) timers.

## 6. Reserve pools, npc_list, tracked cars and visibility cache, lifecycle

`police_manager` owns seven `std::vector` triples. Three are **pre-loaded reserve pools** for
pursuit assets — populated up-front by `police_manager_apply_state` (script `POLICEMANAGER_ON`)
and consumed on demand mid-pursuit by a deploy trio. This is a ready-reserve cache: mid-chase spawns
incur no i3d-load / `operator_new` (the heavy work is done at `POLICEMANAGER_ON` time).

| Pool | offset | element | populate model pool (`police_manager_apply_state`) | deploy / pop helpers | `police_manager_game_init` clears? |
|---|---|---|---|---|---|
| police_vehicles | +0x50 | `vehicle_controller*` | 4-way pool gated by `vehicle_tier` | `police_manager_deploy_vehicle_from_pool` / `police_manager_vehicle_pool_pop_random` | yes |
| spike_strips | +0x60 | `spike_strip*` (`barrier.4ds`; PS2 `C_irenka`, kind 0x1f) | single type | `police_manager_deploy_temp_spike_strip` / `police_manager_spike_strip_pool_pop` | yes |
| foot_cops | +0x70 | `npc*` | 3-way pool (`pol11/12/13`) | `police_manager_deploy_npc_from_pool` / `police_manager_officer_pool_pop_random` | **no — survives soft reset** |

All three are stride-8 elements inserted by `std_vector8_insert_at`: the object pointer, then at
`+4` an **available** byte. `police_manager_apply_state` inserts every element with it set to 1;
each pop clears it when it hands the object out (`police_manager_vehicle_pool_pop_random`,
`police_manager_spike_strip_pool_pop`, `police_manager_officer_pool_pop_random`), each
`police_manager_free_temp_*` sets it back to 1 when the object returns, and
`police_manager_get_free_officers_count` counts the elements that have it set. A vehicle element
also carries a **model flag** at `+5`: 1 when the pick was `polimFor00` or `polimTud00` on
`vehicle_tier` 0, or `polCad00` on the other tier. `police_manager_vehicle_pool_pop_random` reads
it only when its argument is non-zero, and its one caller,
`police_manager_deploy_vehicle_from_pool`, passes 0, so the filter never runs. A gameplay
"roadblock" is a *composite* drawn from all three (cars + spike strips + armed foot cops) — see
`police-system/artifacts/police-system.md` "Wanted with Gun". The asymmetry (foot cops survive
`police_manager_game_init`) means after a soft reset, vehicle/spike-strip deploys
return 0 until the script re-issues `POLICEMANAGER_ON`; foot-cop deploys stay available.

- **npc_list** (`+0x80`, `std::vector<actor*>`) — the **despawn list**: temporary cars (kind 4) and
  `C_entity` NPCs (kind `0x1b`) that the engine spawned or released, deleted once they are out of
  range. `police_manager_npc_list_tick` (outer tick, mode 0) removes a member more than 150 m from the
  active camera — a car only when no human or `C_entity` occupies it, and a nearer car whose frame has
  `+0xac` bit 0 clear; a living `C_entity` in state `+0x5fc == 4` goes when it is released
  (`+0xf48` bit 3) instead of by distance — by pruning it from every record, invalidating it and
  freeing it through `police_manager_free_temp_actor` or `game_remove_temporary_actor`; mode 1
  empties the whole list. The closed set of adders (15 `police_manager_npc_list_push` callsites):
  the crews and cars of `police_manager_try_dispatch_pursuit_for_target` and
  `police_manager_spawn_patrol_car_reinforcements_at_radius`, cops promoted by
  `pedestrian_manager_spawn_cop_from_pedestrian`, gangs from `pedestrian_spawn_gang_on_hit`, mafia
  crews in `traffic_car_update`, cars `traffic_car_deactivate_element` could not free, exploded cars
  (`car_explosion`), occupants thrown out (`human_do_throw_occupant_from_car`), officers released by
  `police_manager_tick` or by `police_manager_unregister_actor`, and the script command
  `ACTOR_ADDDORICI` (`program_process_command` case `0x171`). Removed without despawn (5
  `police_manager_npc_list_erase` callsites): the car the player takes (`human_use_actor`), a car
  `traffic_car_adopt_car_with_new_driver` takes over, and `HUMAN_RETURNTOTRAFF` (case `0x16b`).
  Elements are non-owning (`police_manager_dtor` frees the array, not the actors).
- **tracked_police_cars** (`+0x90`) — the police cars the manager must clean up (§2a, §12d); `police_manager_tick` evicts the oldest while more than five remain, so this bounds the radar-tracked police cars.
  **visibility_cells** (`+0xA0`, `std::vector<uint16_t>`) — the visibility cache; it receives
  visibility-cell indices from `police_manager_apply_state` and the `police_manager_tick` middle-tick
  refresh.

**Lifecycle trio.** The manager is the single instance embedded in the C_game object: `game_ctor`
constructs it (`lea ecx,[esi+0x2da0]`, then `police_manager_ctor`) and `game_dtor` destroys it the
same way (§4).

- `police_manager_ctor` — cold init (timer budgets 200; all triples zeroed).
- `police_manager_game_init` — soft reset: clears records / police_vehicles /
  spike_strips / npc_list (`end = begin`, buffers kept), reseeds timer budgets to operational values;
  does **not** touch foot_cops, tracked_police_cars or visibility_cells. Sole caller: `game_init`.
- `police_manager_dtor` — frees all seven triples, each by releasing the backing array and zeroing
  the three pointers. **No triple destroys its elements**: the records vector's only extra step is a
  call to `stub_noop_args8_00575c00`, whose whole body is `return;`, so the heap `pursuit_record`s
  are not freed here.

## 7. `+0x38 busted_cutscene_lock` — the player-only suppression lock

The game sets it to **1** whenever the **player** is captured in a busted
cutscene — **both arrest and fine** — in `entity_action_tick` case 0x21 (via
`police_manager_set_busted_cutscene_lock`, gated on `folded_get_dword_0x10() == 2`, the
`C_actor::GetKind` role of that folded symbol, = player; callsite VA `0x420081` sets 1,
VA `0x421656` clears it), and clears it to 0 on cutscene teardown; `police_manager_apply_state` also
resets it to 0.

The byte only ever holds 0 or 1: the two setter callsites push the literals 1 and 0, and
`police_manager_apply_state` stores 0. It therefore records that a capture cutscene is running and
carries nothing that could tell an arrest from a fine.

While it is 1 it suppresses the searching HUD icon, identity-pending reporting and player-pursuit
re-dispatch. An EXE-patch test that forces the byte to 1 reproduces exactly that: ambient cops ignore
the player and the searching HUD breaks.

The writer in `entity_action_tick` reaches `+0x38` through
`act_mission_get_game → game_get_police_manager → set`, i.e. from outside the police files.

The PS2 build has the same field and gate (§12d).

**Distinct from `police_board+0x2c`** (`busted_cutscene_active`, read by
`police_board_is_busted_cutscene_active`): that is the **any-target** board-side "capture cutscene
animating" flag, set for a player or an NPC perpetrator upstream of the player-kind gate; `+0x38` is
the **player-only** manager-side suppression lock, set inside that gate.

## 8. Identity recompute — foot vs car (the "searching" swap)

`identity_pending` (+0x51) is what the HUD reads to draw the **searching** icon (searching iff
`identity_pending != 0 && police_manager+0x38 (busted_cutscene_lock) == 0`). The icon's transparency
is a separate test (`police_board_has_recent_sighting`: an assigned officer logged a sighting of the
player within the last 2000 ms).

**Where the identity is held.** Every squad (`police_action`, 0x6c bytes, `structs/police-action.md`)
chasing the player keeps two identity fields: `+0x64` = the vehicle the police know him by, and
`+0x68` = the face-known flag. `police_board_target_matches` reports the player as recognised when
some squad targeting him has `+0x64 ==` his current vehicle (`actor+0x98`), or he is on foot and
that squad has `+0x68` set. `record+0x54` / `+0x58` are copies of a squad's `+0x64` / `+0x68`,
handed back by `police_manager_record_set_tracked_vehicle_for_target` when the squad ends.

**The recompute.** `police_manager_tick` rewrites `+0x51` every inner tick for the player record
(only while an icon is shown). The branch selector is `police_manager_count_officers_alive_for_target`
= officers in `record+0x74` whose death latch (`actor+0x5e`) is clear — a count of living ASSIGNED
officers, not a visibility test:

| store VA | condition | `+0x51` |
|---|---|---|
| `0x005cc77b` | living officers, a squad targets the player | `!police_board_target_matches` |
| `0x005cc786` | living officers, no squad targets the player | 0 (known) |
| `0x005cc7a4` | no living officer, player in a car | `+0x54 != actor+0x98` |
| `0x005cc7b1` | no living officer, player on foot | `+0x58 == 0` |

A squad survives a short unseen spell (`police_action_tick` disbands it after more than 30 s without
a sighting with its chase car more than 150 m away, or when all its officers are dead), so right
after the player leaves a car the first row decides.

**Writers of the squad identity fields** (the set is closed: every store to `+0x64` / `+0x68` in
the `police_board` / `police_action` code band, plus the three squad copy routines):

| writer | `+0x64` | `+0x68` | when |
|---|---|---|---|
| `police_board_create_and_engage` | player's current vehicle (`-1` when `force_flag` is clear and no squad recognises him) | 1 iff he is on foot | squad creation; the two dispatch creators (foot path through `pedestrian_manager_spawn_cop_from_pedestrian`, patrol-car path in `police_manager_try_dispatch_pursuit_for_target`) pass `force_flag = 1` in normal play |
| `police_board_refresh_target_vehicle` | player's current vehicle, on every squad targeting him | set to 1 iff he is on foot, never cleared | from `police_board_create_and_engage` (force path, or a recognised player), and from `police_manager_tick` when a pending state is re-derived while identity is unknown and the player was sighted in the last 2000 ms |
| `police_board_update_sighting` identity batch | player's current vehicle, on every squad targeting him | **1 unconditionally, even in a car** | forced (`force = 1`) by `entity_hit` and `police_manager_report_police_car_stealing_sighting`; otherwise only on a vehicle entry/exit that an assigned officer witnessed (sighting within 2000 ms, or clear line within 10 m) and that matches the squad's `+0x64` |

**What this produces.** Seen only in a car, then leaving it unseen: every squad has `+0x64` = that
car and `+0x68` = 0, so the player is unknown on foot (searching) and known again on re-entering the
same car. Seen on foot, then entering a car unseen: `+0x64` = 0 and `+0x68` = 1, so the car is
unknown (searching) and leaving it restores the state. A witnessed exit or entry runs the identity
batch and keeps the player known.

**The assault-on-officer case.** Two paths set `+0x68` without a witnessed transition, and both are
reached by an in-car assault on an officer:

- **`entity_hit`** (C_entity::Hit). When a car hits a police officer (`+0xf4c == 2`, i.e.
  `human_type` 4) whose squad (looked up through the officer's `+0xf54` session id) targets the car's
  driver, it calls `police_board_update_sighting(driver, 0, 1)` (VA `0x00412388`–`0x00412392`), so
  every squad chasing the player records the car he is in AND his face. The on-foot arm
  (VA `0x004122c9`) does the same when the player himself hits such an officer, including by
  shooting from a car. No impact threshold gates the batch; `27.3` gates only the extra handcuffs
  escalation. Ramming an ambient cop dummy promotes it, on the next inner tick, into exactly such an
  officer beside the car (the promotion skips its line-of-sight test inside 1 m).
- **The sticky assault bit.** Bit 13 has no decay, so pending is re-derived to gun on every inner
  tick and `police_manager_try_dispatch_pursuit_for_target` runs every tick with the facing gate off.
  The first cop activated after the player leaves the car creates a squad while he is on foot, and
  `police_board_refresh_target_vehicle` rewrites every squad to `+0x64 = 0`, `+0x68 = 1`. Each such
  activation reprints "Assault on Police!", because the action code is re-derived as 9 every tick.
  The same bit makes any sighting while searching re-identify the player.

Both leave the player known on foot and unknown in any other car. They differ on re-entering the
original car: after the `entity_hit` batch that car is still known; after an on-foot refresh the
squads hold `+0x64 = 0`, so the original car reads as unknown.

## 9. Raise and clear — the inverse pair

`police_manager_register_collision_offence_for_target` (escalate) and
`police_manager_record_clear_pursuit_state` (wipe) are the canonical round-trip on a record, both
keyed by target lookup against `police_manager.records`. Every field the registrar *writes* (offence
bits, collision timers, `police_state_pending`, `action_code`, `collision_episode_count`) is reset by
the clear; the clear additionally zeroes fields the registrar only *reads* (`run_over_count`,
`wanted_substate_flag`, `car_theft_timer`, `roadblock_eligible_flag`), writes the `-1` sentinel to
`collision_episode_timer` / `wanted_bar_remaining`, and increments `capture_count` when its flag is
set (§2b). The registrar fires routinely (every offence);
the clear has two callers (`police_manager_tick`, `entity_action_tick`) — offences accumulate, then
are wiped in bulk on a reset event.

## 10. `police_action`

The per-pursuit officer squad (`police_action`, `0x6c` bytes) is not part of `police_manager`: it belongs to the global `police_board` singleton (`0x006552a8`). Its layout, evidence, the squad state machine (`+0x2c` tiers: 0 and 1 gun, 2 and 3 fine, 4 and 5 handcuffs, 6 roadblock, 7 disbanded), its method family and its Cheat Engine table are in `structs/police-action.md`.

## 11. Cheat Engine address tables (player instance)

`Game.exe` loads at a constant base (no ASLR), so `0x0065115C` and `0x006552A8` are literal. **Type** = the CE
value type to pick; pointer/array/padding fields are omitted.

**Pointer-chain roots:** `[[0065115C]+24]` → the **C_game** game-world singleton (NOT the player
actor — `*(g_act_mission+0x24)` is C_game). `police_manager`
is inline at `C_game+0x2DA0`. `pursuit_record[0]` base = `[[[[0065115C]+24]+2DE0]]` (`0x2DE0 = 0x2DA0+0x40`). For
index N: `record[N]` = `[[[[0065115C]+24]+2DE0]+N*4]`. (The `police_action[0]` CE table lives in
`structs/police-action.md`.)

### 11a. `police_manager` — base `C_game + 0x2DA0`

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x00 | enabled | Byte | `[[0065115C]+24]+2DA0` |
| +0x04 | tick_acc_inner | 4 Bytes | `[[0065115C]+24]+2DA4` |
| +0x08 | tick_period_inner | 4 Bytes | `[[0065115C]+24]+2DA8` |
| +0x0C | speed_threshold_fine | Float | `[[0065115C]+24]+2DAC` |
| +0x10 | speed_threshold_arrest | Float | `[[0065115C]+24]+2DB0` |
| +0x14 | tick_acc_outer | 4 Bytes | `[[0065115C]+24]+2DB4` |
| +0x18 | tick_period_outer | 4 Bytes | `[[0065115C]+24]+2DB8` |
| +0x1C | tick_acc_visibility | 4 Bytes | `[[0065115C]+24]+2DBC` |
| +0x20 | tick_period_visibility | 4 Bytes | `[[0065115C]+24]+2DC0` |
| +0x24 | tick_acc_vestigial | 4 Bytes | `[[0065115C]+24]+2DC4` |
| +0x28 | tick_period_vestigial | 4 Bytes | `[[0065115C]+24]+2DC8` |
| +0x34 | arrest_text_id | 4 Bytes | `[[0065115C]+24]+2DD4` |
| +0x38 | busted_cutscene_lock | Byte | `[[0065115C]+24]+2DD8` |
| +0x39 | vehicle_tier | Byte | `[[0065115C]+24]+2DD9` |

### 11b. `pursuit_record[0]` — base `[[[[0065115C]+24]+2DE0]]`

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x08 | offense_bits_a | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+8` |
| +0x0A | offense_bits_b | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+A` |
| +0x0C | car_theft_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+C` |
| +0x10 | last_fire_time_ms | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+10` |
| +0x14 | on_sidewalk_timer | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+14` |
| +0x18 | collision_short_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+18` |
| +0x1C | last_crash_time_ms | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+1C` |
| +0x20 | violence_event_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+20` |
| +0x24 | running_red_or_sign_damage_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+24` |
| +0x28 | capture_count | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+28` |
| +0x2C | roadblock_eligible_flag | Byte | `[[[[0065115C]+24]+2DE0]]+2C` |
| +0x34 | last_seen_x | Float | `[[[[0065115C]+24]+2DE0]]+34` |
| +0x38 | last_seen_z | Float | `[[[[0065115C]+24]+2DE0]]+38` |
| +0x3C | police_state (1 fine / 2 cuffs / 3 gun) | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+3C` |
| +0x3E | police_state_pending | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+3E` |
| +0x40 | collision_episode_timer (-1 sentinel) | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+40` |
| +0x44 | wanted_bar_remaining | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+44` |
| +0x48 | wanted_bar_max | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+48` |
| +0x4C | collision_episode_count | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+4C` |
| +0x50 | wanted_substate_flag | Byte | `[[[[0065115C]+24]+2DE0]]+50` |
| +0x51 | identity_pending | Byte | `[[[[0065115C]+24]+2DE0]]+51` |
| +0x5C | reinforcement_counter | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+5C` |
| +0x64 | action_code | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+64` |
| +0x68 | run_over_count | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+68` |

## 12. PC vs PS2

The PC translation unit is `[0x005cad40, 0x005d14a0)`, bounded by the `.CRT$XCU` initializers at
`0x005cad40` and `0x005d14a0`. It holds 66 PC defs against 65 PS2 `C_shvestky` methods, and the
alignment is 1:1 all the way down once the three `ChangeScene*` methods (InterCity, PS2-port-only)
are dropped.

### 12a. `police_manager_report_police_car_stealing_sighting` — the semantics are the caller's

The body itself tests neither "police car" nor "stealing"; its sole caller does. `player_try_use_car`
(`C_player::Can_UseCar`) guards the call with

```c
if ((*(char *)(car + 0x2044) != ' ') &&                          // the car IS a police car
   (*(char *)(*(int *)(g_act_mission + 0x24) + 0x2da0) != ' '))   // police system enabled
```

`car+0x2044` is the cop-car flag — the same byte `police_manager_npc_list_erase` tests, and the same
one PS2 spells `car+0x2473`. For a single-callsite function the semantics are the caller's guard plus
the callee's effect, so every word of the name is backed.

### 12b. Cross-build tuning differences

* `police_manager_game_init` seeds the two speed thresholds at `+0x0c`/`+0x10` with
  **17.0 / 28.0** on the PC and **27.0 / 1000.0** on the PS2. Every other constant in that
  function (`0x14a`, `0xd9`, `0xfad`, `0x391`) is identical.
* `police_manager_spawn_patrol_car_reinforcements_at_radius` turns the spawn heading by a random
  angle up to 2·pi on both builds (PC `rand_float() * 6.2831855`, PS2 `rnd() * 3.1415927 * 2.0`).
  The difference is the police-car cap: the PS2 versions of it and of
  `police_manager_try_dispatch_pursuit_for_target` count cars through
  `4 - PocetPolicejnichPlechovekVRici`, and the PC twins have no police-car count at those positions.
* The PC's radar takes a literal ARGB (`0xffff0000`); the PS2 takes a 3-bit palette index
  into `g_radarLimit`.

### 12c. Methods present on one build only

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001fa3f0 | `ChangeSceneInit__10C_shvestkyFv` | InterCity, PS2-port-only |
| 0x001fa8d0 | `ChangeSceneDone__10C_shvestkyFv` | InterCity, PS2-port-only |
| 0x001fa8e0 | `ChangeSceneVystrc__10C_shvestkyFv` | InterCity, PS2-port-only |
| 0x001fa9d0 | `DezoVratilGumyAleRadioSiNechal__10C_shvestkyFP5C_car` | the remove half of the tracked-car list; not located on the PC (Open item 3) |
| 0x001ffb40 | `PocetPolicejnichPlechovekVRici__10C_shvestkyFv` | not located (Open item 3); its only PS2 callers are `DalsiChalpiDoPartie` and `VasekSelChlastatNaOslavu` (3 calls), whose PC twins have no police-car count |
| 0x00201aa0 | `GetPolicePosForKokot__10C_shvestkyFP7C_humanR18S_police_interface` | not located (Open item 3) |
| 0x00201d60 | `BugZamrdanychTesteruZTepleDve_410__10C_shvestkyFP7C_humaniR18S_police_interface` | not located (Open item 3) |
| 0x002023a0 | `JeToDutolebecniAuto__10C_shvestkyFP5C_carb` | temp-car pool membership test; below |

**`JeToDutolebecniAuto__10C_shvestkyFP5C_carb`.** With its bool clear it forwards to
`FreeMyTempActor` (the twin of `police_manager_free_temp_actor`); with it set it reports whether the
car is in the temp-car pool (PS2 count `+0x50`, records `+0x54`, 8 bytes each). Its two PS2
callers are `Garage_DelCar__9C_missionFP5C_car` (bool clear) and
`DestroyCar__13G_GarageClassFP5C_car` (bool set), which skips `InvalidateActor` for a pool car. No
PC function tests pool membership alone: a scan of all of `.text` for code that walks the PC
temp-car pool `+0x50` / `+0x54` (the same scan finds `police_manager_spike_strip_pool_pop` on
`+0x60` / `+0x64`) finds only `police_manager_acquire_from_pool_by_name`,
`police_manager_vehicle_pool_pop_random`, `police_manager_free_temp_car` and
`police_manager_rename_police_car`, and `garage_destroy_car`, the twin of `DestroyCar`,
invalidates the car unconditionally at the position of the PS2 test.

**PC has, PS2 does not:** `police_manager_deploy_temp_spike_strip`,
`police_manager_deploy_npc_from_pool`, `police_manager_spawn_roadblock_in_neighbor_cells`,
`police_manager_spike_strip_pool_pop`, `police_manager_add_officer_to_target`. For the two
spike-strip functions this is measured: the PS2 instantiates no `insert` for
`vector<C_shvestky::S_road_block_iris>`, so its iris pool is never filled
(`structs/spike_strip.md` §8).

### 12d. Field evidence from the PS2 build

* **`+0x90` tracked police cars.** The PS2 twin of the writer
  `police_manager_add_tracked_police_car` is `DezoSebralGumyIRadio__10C_shvestkyFP5C_car`, whose
  parameter is typed `P5C_car`.
* **`+0x38 busted_cutscene_lock`.** The field exists at `C_shvestky+0x38` on PS2 and is read
  identically by `NeviDoKterePiceVlezl__10C_shvestky` (the same `record+0x51 && +0x38 == 0` gate).
  The PS2 writes it inline, as `*(act_mission+0x20) + 0x1218` (`C_game+0x11e0+0x38`), in
  `Tick__11ai_modifierFUi`, the twin of `entity_action_tick`: 1 behind the same player-kind test
  right after `LockControls(1)`, 0 at the cutscene teardown. It has no setter method.
* **Construction.** The PS2 has no `C_game` constructor; `Init__9C_missionFP10I3D_driver` builds the
  0x16a0-byte `C_game` inline and runs `__ct__10C_shvestkyFv` on its `+0x11e0`, and
  `UnInit__9C_missionFv` runs the destructor the same way.
* **The officers vector.** PS2 `AddPerson` constructs a 12-byte `__vector_pod` at `record+0x70`, the
  position of the PC's 16-byte `std::vector`; that 4-byte difference is the whole record-size
  difference (PS2 0x7c, PC 0x80).
* **Pool element types.** The PS2 names the three pool element types
  `C_shvestky::S_road_block_car`, `S_road_block_iris` and `S_road_block_enemy` (their vector helper
  symbols). Both PS2 callers of `GetTempCar__10C_shvestkyFb` pass 0, as the PC's does.
* **`pursuit_record+0x28 capture_count`.** PS2 `PocetPrestupku` ("number of offences") returns the
  same field.

## 13. Related

- `police-system/artifacts/offences.md`: offence bit map, escalation, noticing, strings.
- `police-system/artifacts/police-manager.md`: `police_manager_tick` phases and the noticing pipeline.
- `police-system/artifacts/police-system.md`: runtime behaviour, state machine, call chain.
- `structs/police-board.md` and `structs/police-action.md`: the officer-squad system.
- `structs/pedestrian.md`, `structs/traffic_car.md`: the crews and the cars the manager promotes and drives.
- `ghidra/Game-ghidra.exe.functions.md`: the function registry.

## 14. Open items

1. **Runtime validation:** no field has been confirmed with Cheat Engine or ReClass.
2. **Assault-on-officer identity exception:** the two mechanisms are traced (§8: the `entity_hit` identity batch and the sticky-bit re-dispatch); which one fires in a given in-car assault is not yet measured. Discriminators: a write breakpoint on squad `+0x68` names the writer (`0x00447e66` with return address `0x00412397` / `0x004122d8` = `entity_hit`; `0x00448210` or `0x00446cb9` / `0x00446cdf` = a squad created or refreshed on foot); a reprinted "Assault on Police!" on leaving the car; the state on re-entering the original car unseen.
3. **PS2 methods with no PC function at their rank (§12c):** `DezoVratilGumyAleRadioSiNechal`, `PocetPolicejnichPlechovekVRici`, `GetPolicePosForKokot`, `BugZamrdanychTesteruZTepleDve_410`; none of them is tagged terminal.
