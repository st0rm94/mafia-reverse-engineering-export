# `police_action` — the pursuit squad (0x6C bytes)

**Created** 2026-07-08 22:08 +0200 · **Modified** 2026-11-08 17:23 +0100

One `police_action` is one officer squad chasing one suspect. The `police_board` singleton
(`structs/police-board.md`, VA `0x006552a8`) owns them in a stride-`0x6c` `std::vector` whose object
begins at `board+0x10`; `police_board_create_and_engage` adds one per new chase and stamps it with a
session id from the board's `+0x20` counter, and `police_board_tick` runs `police_action_tick` once
per squad per frame. Reconstructed from `Game.exe` disassembly and the annotated decomp; no field
has been watched in game (Open item 1).

It is not a `police_manager` field: `police_manager` is one instance embedded in the `C_game`
singleton at `C_game+0x2da0` (`structs/police_manager.md`), and every `police_manager_*` call in
`police_action_tick` disassembles to `mov eax,[g_act_mission]; mov eax,[eax+0x24]; lea
ecx,[eax+0x2da0]`. The two structs meet in the police call chain and are unrelated in memory.

| | |
|---|---|
| PC prefix | `police_action_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x6c` (the board's stride-`0x6c` iteration and `police_action_vector_count`'s divisor) |
| PC constructor | `police_action_copy_ctor` @ `0x00449800`, the only constructor: elements are copy-constructed by the vector helpers (§7) |
| PC translation unit | `[0x00443bb0, 0x00449ba0)`, shared with `police_board`, whose doc hosts the unit (`structs/police-board.md` §4); `police_action_destroy_range` sits in the preceding `ai_manager` unit |
| PS2 class | `ai_police_action` (tag `16ai_police_action`), 10 methods: 9 mapped (§3), 1 PS2-only (§8) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct police_action {                    // 0x6c bytes
    /* +0x00 */ std::vector<C_entity*> officers;  // [H] {alloc, begin, end, cap}: the squad roster
    /* +0x10 */ float    anchor_pos[3];           // [H] last-sighting world position
    /* +0x1c */ float    anchor_dir[3];           // [H] last-sighting direction (normalised)
    /* +0x28 */ uint32_t last_sighting_ts_ms;     // [H] 0 = never sighted
    /* +0x2c */ uint32_t tier;                    // [H] the response tier (§4)
    /* +0x30 */ uint32_t session_id;              // [H] from police_board +0x20
    /* +0x34 */ uint32_t on_foot_phase;           // [H] 0 vehicle phase, 1 on-foot phase; the disband stash (§4a)
    /* +0x38 */ uint32_t arrest_grace_deadline;   // [H] creation + 60000 + rand
    /* +0x3c */ uint32_t created_ts_ms;           // [H] game time at creation
    /* +0x40 */ uint32_t los_timestamp_ms;        // [H] last time an officer perceived the suspect
    /* +0x44 */ uint32_t sighting_accum_ms;       // [H] accumulated perceived time
    /* +0x48 */ uint32_t search_rearm_deadline;   // [H] gate of the next event-0x12 dispatch
    /* +0x4c */ uint32_t stuck_vehicle_ms;        // [H] time the chase car has been unusable
    /* +0x50 */ uint32_t flags;                   // [H] §2a
    /* +0x54 */ uint32_t backup_rerequest_deadline; // [H] game time when flags 0x20 | 0x40 are next set
    /* +0x58 */ uint32_t unread_58;               // [U] 0 at creation, copied; no reader (.text scan)
    /* +0x5c */ C_car   *group_vehicle;           // [H] the squad's chase car, 0 on foot / lost
    /* +0x60 */ C_actor *target;                  // [H] the suspect, the key of every *_target op
    /* +0x64 */ C_car   *tracked_target_vehicle;  // [H] the car the suspect was last seen in
    /* +0x68 */ uint8_t  target_on_foot;          // [H] latch: the squad keeps the suspect on foot
} police_action;
```

* **`officers`.** An MSVC `std::vector` of non-owning officer pointers (stride 4). Its allocator
  byte at `+0x00` is written by `police_board_create_and_engage` (`mov [eax], cl`, the force
  argument's byte, `0x00446c4f`) and copied by `police_action_copy_ctor`, and never read. Roster officers are `C_entity` actors (kind `0x1b`); the
  arrest orders of §4 go only to those.
* **The sighting record (`+0x10`..`+0x28`).** `police_board_update_sighting` clears
  `last_sighting_ts_ms`, then in the in-car tiers 0 / 2 / 4 with `on_foot_phase` = 0 copies the
  suspect's own frame position and forward vector and stamps the game clock (`C_game+0x2b0c`); in
  every other case it takes, from the roster's `C_entity` officers, the newest
  `ai_sensors_get_smelled_target_info` report that is newer than `+0x28`. With no such report
  `+0x28` stays 0.

## 2. Field evidence

Measured by a census of every `this+N` access in the `police_action_*` functions and by body reads
of the `police_board_*` functions that write a squad. The four element helpers of §7
(`police_action_copy_ctor`, `police_action_assign`, `police_action_copy_assign_range`,
`police_action_uninit_copy_range`) copy every field; they are not repeated in the cells.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | officers | H | `police_board_create_and_engage` (binds the roster), `police_action_tick` (pops a dead officer ¹), `police_board_invalidate_actor` (removes a dead NPC), `police_action_disband` (`end = begin`) | `police_action_tick`, `police_action_disband`, `police_action_broadcast_event`, `police_action_find_officer_index`, `police_action_dispatch_officer_commands`, `police_action_tick_sighting_timers`, `police_board_is_actor_in_pursuit` | §1 |
| +0x10 | anchor_pos | H | `police_board_create_and_engage` ((0, 0, 0)), `police_board_update_sighting` | `police_action_tick` (the offset of the "go to point" pursuit event in tiers 1 / 3); `entity_action_tick` cases `0x1e` / `0x21` (copied into the task's cached target position when the officer's vehicle state differs from the squad's) | - |
| +0x1c | anchor_dir | H | `police_board_create_and_engage` ((1, 0, 0)), `police_board_update_sighting` | `entity_action_tick` cases `0x1e` / `0x21` (the task's cached forward vector, same gate) | `police_action_tick` does not read it |
| +0x28 | last_sighting_ts_ms | H | `police_board_create_and_engage` (0), `police_board_update_sighting` | `police_action_tick` (0 = never sighted: disband when the suspect leaves the tracked car before the on-foot phase); `entity_action_tick` cases `0x1e` / `0x21` (task slot `[0xb]`, the slot `ai_sensors_get_smelled_target_info` fills; non-zero gates the position copy) | - |
| +0x2c | tier | H | `police_board_create_and_engage` (from its mode argument), `police_board_raise_gun_escalation` (0 / 1), `police_board_raise_handcuffs_escalation` (4 / 5), `police_board_promote_gun_escalation` (0 / 1), `police_board_promote_handcuffs_escalation` (4 / 5), `police_action_tick` and `police_action_on_vehicle_lost` (even → odd), `police_action_disband` (7), `police_action_dispatch_officer_commands` (restores the stash), `police_board_invalidate_actor` | `police_action_tick` (both switches), `police_action_disband`, `police_action_tick_sighting_timers`, `police_action_dispatch_officer_commands`, `police_action_on_vehicle_lost`, `police_board_update_sighting`, `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation`, `police_board_promote_gun_escalation`, `police_board_promote_handcuffs_escalation`, `police_board_invalidate_actor` | §4 |
| +0x30 | session_id | H | `police_board_create_and_engage` (`police_board_increment_counter`) | `police_board_find_group_by_session_id`, `police_board_create_and_engage` (passes it to `entity_spawn_police_response`, which stamps each officer's `+0xf54`) | unique per chase |
| +0x34 | on_foot_phase | H | `police_board_create_and_engage` (1 on foot, 0 in a car), `police_board_invalidate_actor` (1 when the car died), `police_action_tick` (1 on entering the exit-vehicle / arrest phase, 0 when re-issuing the vehicle order), `police_action_on_vehicle_lost` (1), `police_action_dispatch_officer_commands` (1), `police_action_disband` (the with-car path stashes the tier here, §4a) | `police_action_tick`, `police_action_tick_sighting_timers` (0 = the in-car fast path), `police_action_dispatch_officer_commands` (tier 7 reads the stash), `police_board_update_sighting`, `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation` | - |
| +0x38 | arrest_grace_deadline | H | `police_board_create_and_engage`, once: game time + 60000 + `rand_range` | `police_action_tick` (the second switch: past it, `police_manager_enable_roadblock_flag_for_target` and the per-officer `ai_logs_get_log_for_actor` loop ²) | - |
| +0x3c | created_ts_ms | H | `police_board_create_and_engage`, once | `police_action_tick` (tier 1: older than 30000 ms escalates the wanted substate) | never re-armed: the gate means "this squad is older than 30 s" |
| +0x40 | los_timestamp_ms | H | `police_board_create_and_engage` (the game time), `police_board_promote_gun_escalation`, `police_board_promote_handcuffs_escalation` (the game time), `police_action_tick_sighting_timers` (the game time while the suspect is perceivable; in the in-car fast path only while `pursuit_event_record_is_chase_tracking_by_vehicle(+0x5c)` returns 0) | `police_action_tick` (its age gates the 5 / 10 / 15 / 30 / 40 s officer-recall steps) | - |
| +0x44 | sighting_accum_ms | H | `police_board_create_and_engage` (0), `police_board_promote_gun_escalation`, `police_board_promote_handcuffs_escalation` (0), `police_action_tick_sighting_timers` (adds the frame time while perceivable, same condition) | `police_action_tick` (above 40000 / 15000, `police_manager_record_begin_wanted_substate`), `police_action_tick_sighting_timers` | - |
| +0x48 | search_rearm_deadline | H | `police_board_create_and_engage` (0), `police_action_tick` (game time + 3000; + 10000 after a dispatch) | `police_action_tick` (the gate before sending officers to a callable device with event `0x12`) | - |
| +0x4c | stuck_vehicle_ms | H | `police_board_create_and_engage` (0), `police_action_tick` (adds the frame time while `car_is_usable(2)` is false, 0 when true, 20000 when channel 1 is false) | `police_action_tick` (above 10000: cancel the vehicle order and advance the tier) | - |
| +0x50 | flags | H | `police_board_create_and_engage` (0), `police_action_tick`, `police_action_on_vehicle_lost` | `police_action_tick`, `police_action_on_vehicle_lost`, `police_board_has_active_group_for_target` (bit 0), `police_board_target_matches` | §2a |
| +0x54 | backup_rerequest_deadline | H | `police_board_create_and_engage` (0), `police_action_tick` (game time + 3000 when it passes) | `police_action_tick` (when passed, sets flags `0x20 \| 0x40`) | - |
| +0x58 | unread_58 | U | `police_board_create_and_engage` (0, once, `0x00446d25`) | - | copied by the element helpers and by the splice of `police_board_end_pursuit`; no other access (a `.text` scan: the other `+0x58` hits are `police_manager` fields) |
| +0x5c | group_vehicle | H | `police_board_create_and_engage` (the suspect's type-4 vehicle, or 0), `police_action_tick` (0 on drop), `police_action_on_vehicle_lost` (0), `police_board_invalidate_actor` (0 when the vehicle is the dead actor) | `police_action_tick` (the chase-order logic), `police_action_disband` (non-zero: transfer ownership), `police_action_dispatch_officer_commands`, `police_action_tick_sighting_timers`, `police_board_release_all_group_vehicles`, `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation`, `police_board_promote_gun_escalation`, `police_board_promote_handcuffs_escalation`, `police_board_invalidate_actor` | - |
| +0x60 | target | H | `police_board_create_and_engage` | `police_action_tick`, `police_action_disband`, `police_action_tick_sighting_timers`, `police_board_target_matches`, `police_board_is_actor_in_pursuit`, `police_board_invalidate_actor`, `police_board_has_active_group_for_target`, `police_board_broadcast_for_target`, `police_board_disband_by_target`, `police_board_end_pursuit`, `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation`, `police_board_promote_gun_escalation`, `police_board_promote_handcuffs_escalation`, `police_board_update_sighting`, `police_board_refresh_target_vehicle`, `police_board_has_recent_sighting`, `police_board_has_group_for_target`, `police_board_create_and_engage` (the ai-group reuse) | - |
| +0x64 | tracked_target_vehicle | H | `police_board_create_and_engage` (the vehicle, or -1 when no squad covers the suspect yet), `police_board_refresh_target_vehicle` (the suspect's `+0x98`), `police_board_update_sighting` (the forced branch) | `police_action_tick` (equal to the suspect's `+0x98` keeps the chase live), `police_action_disband`, `police_action_tick_sighting_timers`, `police_board_target_matches`, `police_board_end_pursuit`, `police_board_update_sighting` | - |
| +0x68 | target_on_foot | H | `police_board_create_and_engage` (1 on foot, else 0), `police_board_refresh_target_vehicle` (1 when the suspect has no vehicle), `police_board_update_sighting` (the forced branch stores 1 with the suspect in a car) | `police_action_tick`, `police_action_disband`, `police_action_tick_sighting_timers`, `police_board_target_matches`, `police_board_end_pursuit`, `police_board_update_sighting` | a latch, "the squad keeps the suspect on foot": `police_board_target_matches`, the tick's force byte and the sighting timers accept an on-foot suspect only while it is set |

¹ `stl_copy_dwords_fastcall` shifts the tail down, `stub_noop_args8_00575c00` (an empty `ret 8`)
follows, then `end -= 4`.

² `ai_logs_get_log_for_actor` is a pure lookup and its result is discarded, so that loop has no
observable effect.

### 2a. `flags` (`+0x50`) — bit map

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | the squad is active | recomputed each tick by `police_action_tick` from `ai_action_manager_has_action(0x23)`; read by `police_board_has_active_group_for_target` |
| `0x2` | redispatch the orders | cleared by `police_action_tick` after the dispatch; it drives `police_action_dispatch_officer_commands`, whose tier-7 case revives a disbanded squad (§4a) |
| `0x4` | the chase car was lost | set by `police_action_on_vehicle_lost` |
| `0x20 \| 0x40` | backup re-request | set together by `police_action_tick` when `backup_rerequest_deadline` passes |

## 3. Method family

Coverage: 9 of the 10 PS2 `ai_police_action` methods are mapped, all CONFIRMED;
`ProhodteSiHodnosti` has no PC function located (§8). The container template `reserve` is the
STRUCTURAL twin of `police_action_vector_insert_at` (§3d). Six PC functions have no PS2 symbol: the range destroyer and five vector
helpers. The `police_board_*` functions that write a squad take the board as `this` and live in
`structs/police-board.md` §3.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443b60 | `police_action_destroy_range` | a range of instances | - | - | - | destroys `[first, last)` at stride `0x6c` (each roster through `vector_dispose_at_0x04`); callers `ai_manager_dtor` and `police_board_create_and_engage` |
| 0x00449800 | `police_action_copy_ctor` | instance | `__ct__16ai_police_actionFRC16ai_police_action` | 0x0036fd60 | CONFIRMED | deep-copies `officers`, then the dwords `+0x10`..`+0x68` |
| 0x00449970 | `police_action_assign` | instance | `__as__16ai_police_actionFRC16ai_police_action` | 0x00370070 | CONFIRMED | `operator=`: self-assignment guard and a three-way reuse / grow / shrink of the roster |

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00443be0 | `police_action_disband` | `HosiKoncime__16ai_police_actionFP7C_actorb` | 0x0036b120 | CONFIRMED | ends the squad: event `0x11` to every officer except the given one, clears the roster, hands off or tears down the car, `police_board_end_pursuit` |
| 0x00443d00 | `police_action_broadcast_event` | `EventVsem__16ai_police_actionFP7C_actorUiPv` | 0x0036b290 | CONFIRMED | sends an officer event to every roster officer except an excluded one |
| 0x00443d40 | `police_action_tick` | `Tick__16ai_police_actionFUi` | 0x0036b330 | CONFIRMED | the per-frame driver: prunes dead officers, manages the chase car, runs the tier machine (§4), stages arrests |
| 0x004461d0 | `police_action_find_officer_index` | `KolikatejSemSefe__16ai_police_actionFP7C_actor` | 0x0036e290 | CONFIRMED | the officer's roster index, or `0x29a` (666) when absent |
| 0x00446200 | `police_action_tick_sighting_timers` | `PanoveCekamVasNaWCSesynchronizujemSiCas__16ai_police_actionFUi` | 0x0036e2e0 | CONFIRMED | refreshes `los_timestamp_ms` and `sighting_accum_ms` from the officers' perception |
| 0x004466e0 | `police_action_dispatch_officer_commands` | `TakZnovuDoToho__16ai_police_actionFv` | 0x0036e9c0 | CONFIRMED | pushes the current tier to each officer as orders and clears officer flag `+0xf48` bit 3; its tier-7 case revives the squad (§4a) |
| 0x00446930 | `police_action_on_vehicle_lost` | `ZbavSeAutaSmrdi__16ai_police_actionFb` | 0x0036ed00 | CONFIRMED | clears `group_vehicle`, sets flag `0x4`, steps the tier even → odd |

### 3d. Free helpers

The `std::vector<police_action>` instantiation (§7). Each takes a vector or a range, not a squad;
`police_action_vector_count` and `police_action_vector_insert_at` receive the vector object at
`board+0x10` in ECX.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004490b0 | `police_action_vector_count` | - | - | - | `(end - begin) / 0x6c`, guarded against a NULL begin |
| 0x004490e0 | `police_action_vector_insert_at` | `reserve__Q23gif28vector<16ai_police_action,8>Fi` | 0x0036fe50 | STRUCTURAL | `insert(pos, n, value)`; the grow path allocates, copy-constructs prefix, new elements and suffix, then destroys and frees the old buffer (§8) |
| 0x004493a0 | `police_action_uninit_copy_range` | - | - | - | copy-constructs a range into raw storage |
| 0x004493e0 | `police_action_uninit_fill_n` | - | - | - | `n` copy-constructions from one value |
| 0x00449450 | `police_action_copy_assign_range` | - | - | - | `std::copy` over constructed elements |
| 0x00449620 | `police_action_copy_backward_range` | - | - | - | the overlap-safe shift-up of the insert |

## 4. Squad tiers (`+0x2c`)

`tier` encodes the **response tier**, not a chase phase, and pairs the way the HUD police-state
tiers do (`record+0x3c`: 1 fine / 2 handcuffs / 3 gun). **Even = the squad is in a car, odd = on
foot**; the even → odd step happens when the car is lost (`police_action_on_vehicle_lost`, the
stuck-car watchdog of `police_action_tick`).

| Tier | In car | On foot | Set by |
|---|---|---|---|
| **gun** | 0 | 1 | `police_board_raise_gun_escalation`; `police_board_create_and_engage` mode 0 |
| **fine** | 2 | 3 | `police_board_create_and_engage` mode 2, the caller's default, so a fresh dispatch opens here |
| **handcuffs** | 4 | 5 | `police_board_raise_handcuffs_escalation`; `police_board_create_and_engage` mode 4 |
| **roadblock squad** | - | 6 | `police_board_create_and_engage` mode 6; it never reaches either switch of `police_action_tick` |
| **disbanded** | - | 7 | `police_action_disband`, `police_board_invalidate_actor` |

Three independent witnesses fix the mapping: `police_board_raise_gun_escalation` writes 0 / 1 and
gates only on `!= 6`; `police_board_raise_handcuffs_escalation` writes 4 / 5 and refuses tiers 0 and
1, so handcuffs cannot demote the gun tier; `police_board_create_and_engage` maps mode 0 → (1, 0),
2 → (3, 2), 4 → (5, 4), 6 → 6, and `police_manager_try_dispatch_pursuit_for_target` passes mode 2 by
default. That is also why tier 2 is the only tier that raises
`police_board_raise_handcuffs_escalation` from inside the tick: fine → handcuffs is the one
escalation a squad raises on itself.

Every tier case shares one **officer distance-cull idiom**: resolve a reference frame (the suspect's
body frame at `npc+0x68`, refreshed when the dirty bit `frame[0xac] & 0x20` is clear), read its
world position, walk the roster and `entity_dispatch_event(0, <event>, <arg>)` every officer past a
squared-metre gate (`400` = 20 m, `1600` = 40 m, `2500` = 50 m, `6400` = 80 m, `22500` = 150 m,
`40000` = 200 m).

| Tier | Behaviour in `police_action_tick` | Officer events |
|---|---|---|
| 0 | gun tier, in car: escalates the wanted substate after 40 s of accumulated sighting; disbands when the car drifts more than 150 m from an unseen suspect | `0x11` advance, `0x12` call HQ |
| 1 | gun tier, on foot: escalates the wanted substate once the squad is older than 30 s | `0xc` search, `0x11`, `0x12` call HQ |
| 2 | fine tier, in car: **raises the handcuffs tier** when the suspect's car keeps moving; disbands on a drift above 150 m | `0x11` advance |
| 3 | fine tier, on foot | `0xc` search |
| 4 | handcuffs tier, in car; mirrors tier 0 | `0x11` advance, `0x12` call HQ |
| 5 | handcuffs tier, on foot | `0xc` search, `0x11`, `0x12` call HQ |
| 7 | disbanded: recalls officers more than 200 m from the **active camera** ¹ | `0xc` recall |

¹ The reference frame is `(*(g_act_mission+0x10))->+0x17c`, the `I3D_scene`'s camera frame
(`structs/i3d_frame.md`).

**Event `0x12` is "go to a callable device and call HQ".** The list those branches walk is
`C_game+0x1c8` / `+0x1cc`, the stride-`0xc` list of emergency alarms and phone booths that
`game_load_telephones` builds (`structs/game.md`); the officer picks the first device within 40 m
(§6).

The chase car is driven by `pursuit_event_dispatch_pursuit_vehicle` with a tier-keyed event mask
(`0x400` gun / `0x800` fine / `0x200` handcuffs) and a timeout. Every such dispatch first arms the
model-level boost (`pursuit_event_set_change_tables` with `ECX = 1`,
`police-system/artifacts/boosting.md` §6); the only disarm in the family is the no-sighting teardown
branch. The on-foot arrest is staged in the **second** switch (tiers 0 / 2 / 4): once the chase car
stops, the squad broadcasts event `0xf` (exit the vehicle and converge), enters the on-foot phase
and issues arrest orders (event `0x10`) to the officers whose kind `+0x10` is `0x1b`, `C_entity`,
the scriptable-NPC class (`structs/entity.md`).

### 4a. Tier 7 is not always terminal — the `+0x34` stash

The **with-car** path of `police_action_disband` writes `on_foot_phase = tier` before `tier = 7`.
`police_action_dispatch_officer_commands` case 7 reads it back (`tier = on_foot_phase`,
`on_foot_phase = 1`) and broadcasts a final event `0xf`: a squad that disbanded while still holding
a car is **revived at its old tier, on foot**, by the next redispatch request (flag `0x2`). The
no-car path has no stash: it calls `police_board_end_pursuit` and the squad is spliced out for good.

## 5. Cheat Engine — `police_action[0]` (only while a pursuit is active)

Base `[006552A8+14]` (the board's `+0x14` begin). For index N: `[006552A8+14]+<N*0x6c>`. If it reads
garbage, the board pointer indirects once more: try `[[006552A8]+14]`.

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x10 | anchor_pos x | Float | `[006552A8+14]+10` |
| +0x14 | anchor_pos y | Float | `[006552A8+14]+14` |
| +0x18 | anchor_pos z | Float | `[006552A8+14]+18` |
| +0x1C | anchor_dir x | Float | `[006552A8+14]+1C` |
| +0x20 | anchor_dir y | Float | `[006552A8+14]+20` |
| +0x24 | anchor_dir z | Float | `[006552A8+14]+24` |
| +0x28 | last_sighting_ts_ms | 4 Bytes | `[006552A8+14]+28` |
| +0x2C | tier (0/1 gun, 2/3 fine, 4/5 cuffs, 6 roadblock, 7 disbanded) | 4 Bytes | `[006552A8+14]+2C` |
| +0x30 | session_id | 4 Bytes | `[006552A8+14]+30` |
| +0x34 | on_foot_phase | 4 Bytes | `[006552A8+14]+34` |
| +0x38 | arrest_grace_deadline | 4 Bytes | `[006552A8+14]+38` |
| +0x3C | created_ts_ms | 4 Bytes | `[006552A8+14]+3C` |
| +0x40 | los_timestamp_ms | 4 Bytes | `[006552A8+14]+40` |
| +0x44 | sighting_accum_ms | 4 Bytes | `[006552A8+14]+44` |
| +0x48 | search_rearm_deadline | 4 Bytes | `[006552A8+14]+48` |
| +0x4C | stuck_vehicle_ms | 4 Bytes | `[006552A8+14]+4C` |
| +0x50 | flags (bit 0 = active) | 4 Bytes | `[006552A8+14]+50` |
| +0x54 | backup_rerequest_deadline | 4 Bytes | `[006552A8+14]+54` |
| +0x68 | target_on_foot | Byte | `[006552A8+14]+68` |

## 6. Spawn-gating chain for the type-`0x28` action

- `police_action_tick` case 0 sends event `0x12` to one officer when: the squad reaches the 10 s
  mark; a roster officer has `sub_state == 7`; `dist²(officer, suspect) > 400.0` (20 m); and a
  callable device lies within `dist²(officer, device) < 1600.0` (40 m).
- Event `0x12` → `entity_dispatch_event` case `0x12` → `ai_action_new_police_call(device_idx, 20000,
  3, 0, 0)` (the type-`0x28` `ai_action`, `structs/entity.md`) → `ai_action_manager_new_job` on the
  officer's `NPC+0x1164` → the officer latches `sub_state = 9`.
- The officer's line-of-sight flag does not gate this chain.

## 7. The `std::vector<police_action>` instantiation

Eight functions implement the container: `police_action_copy_ctor`, `police_action_assign` (§3a)
and the six helpers of §3d. The instantiation is exclusive: a byte scan of `.text` for `E8` calls
and `E9` jumps to all eight finds every reference in `police_board_create_and_engage` or inside the
cluster, so none of the names is exposed to a COMDAT fold. Six of the eight are free function
templates: read `police_action_uninit_copy_range` as the `police_action` instantiation of
`uninit_copy`, never as a method.

## 8. PC vs PS2

- **Class names.** `ai_police_action`; the owner is `ai_police_nastenka` (`structs/police-board.md`).
- **Object size: PC `0x6c`, PS2 `0x80`.** The roster rebases PC `+0x04` ↔ PS2 `+0x20` (a Metrowerks
  `std::__vector_imp` against an MSVC `std::vector`); the fields after it sit one dword apart (target
  PC `+0x60` ↔ PS2 `+0x64`, flags `+0x50` ↔ `+0x54`, session number `+0x30` ↔ `+0x34`).
- **The container symbols** name the element type where the PC decomp cannot:
  `__ct__16ai_police_actionFRC16ai_police_action` (the copy constructor) and
  `reserve__Q23gif28vector<16ai_police_action,8>Fi` (the element type in the template argument).
  The PS2 `reserve` calls the copy constructor and then `__dt__std::__vector_imp(old + 0x20)` in its
  grow loop, exactly where the PC calls `police_action_copy_ctor` and `vector_dispose_at_0x04(old +
  0x04)`: the same two callees at the documented rebase.
- **Grades.** `police_action_vector_insert_at` is STRUCTURAL: the PS2 `reserve(n)` is only the
  grow-and-move core, the PC function is `insert(pos, n, val)` that contains it, hence the size
  ratio 2.35. `police_action_assign`'s ratio 2.18 has the same cause in reverse (the PS2 delegates the
  grow to `gif::vector::reserve`, MSVC inlines it); `police_action_copy_ctor`'s ratio is 1.33. The
  other five vector helpers have no PS2 symbol: Metrowerks inlined the same loops inside `reserve`.
- **`ai_action_new_police_call`** ↔ `NewPoliceCall__17ai_action_manager` (CONFIRMED), paired 1:1 on
  the `ai_action` type id `0x28` (`structs/entity.md`), the action of §6.

**PS2 methods with no PC function:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0036edf0 | `ProhodteSiHodnosti__16ai_police_actionFv` | when the roster at `+0x20` holds exactly two officers it reinserts them at index 0 (a reorder of the pair); its two callers are both in `AI__8C_entityFUi`, and no PC function with this body was located (Open item 2) |

## 9. Related

- `structs/police-board.md` — the owning board singleton and the unit's host.
- `structs/police_manager.md` — the `police_manager` at `C_game+0x2da0`; a distinct struct.
- `structs/entity.md` — the officers (`C_entity`) and the type-`0x28` action.
- `police-system/artifacts/police-system.md` — the squad life cycle.
- `police-system/artifacts/boosting.md` — the chase-car boost.
- `Game-ghidra.exe.functions.md` — the function registry.

## 10. Open items

1. **Runtime validation.** The layout is static analysis only; no field has been watched in game.
2. **`ProhodteSiHodnosti`.** Whether `entity_ai` reorders a two-officer roster inline at the
   positions of the two PS2 calls in `AI__8C_entityFUi` was not checked.
