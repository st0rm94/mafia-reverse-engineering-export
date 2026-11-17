# `ai_group` / `ai_groups` — NPC groups and the group registry

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-11-09 21:00 +0100

**`ai_group`** is one group of NPCs: it holds a vector of `ai_group_member` records and acts as a small **event bus**, where members cast events into the group, other members receive them, and a per-member `role` decides which direction each
member may use. **`ai_groups`** is the **registry** that owns every group, hands out ids, ticks them and serialises them into the savegame.

```
  ai_groups   (registry: ids, Tick, Save/Load)      §1c
      |  owns N of
      v
  ai_group    (one group; event bus + car helpers)  §1b
      |  holds a vector of
      v
  ai_group_member  { C_entity *entity; uint role }  §1a
```

| | |
|---|---|
| PC prefix | `ai_group_`, `ai_groups_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `ai_groups` `0x28`, a member of the root at `ai_manager+0xc4` (`0x006552dc`); `ai_group` `0x60`; `ai_group_member` `0x08`; `ai_group_event` `0x14` |
| PC constructor | `ai_groups` -, built by `ai_manager_ctor`; `ai_group_copy_ctor` |
| PC translation unit | `[0x0043b000, 0x00443510)`, guard to guard, 84 starts, shared with `ai_follow_manager` and `ai_fight_manager`; this doc hosts it (§6) |
| PS2 class | `ai_group` (tag `8ai_group`), 11 methods; `ai_groups` (tag `9ai_groups`), 12 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `ai_group_member` (`0x08`)

```c
typedef struct ai_group_member {
    /* +0x00 */ C_entity *entity; // [H] the member, also the lookup key
    /* +0x04 */ uint32_t  role;   // [H] 0 full participant, 1 receive-only, 2 cast-only (§2e)
} ai_group_member;
```

The element type comes from the PS2 template argument (§7); the PC decomp shows only stride-8 rows.

### 1b. `ai_group` — one group (`0x60`)

```c
typedef struct ai_group {
    /* +0x00 */ uint32_t         id;              // [H] the group's id, the key of ai_groups_get_group; stored in entity+0x11ac
    /* +0x04 */ uint32_t         unread_04;       // [U] 0 from ai_groups_new_group; saved and loaded; no reader (class census)
    /* +0x08 */ uint32_t         unread_08;       // [U] 0 from ai_groups_new_group; saved and loaded; no reader (class census)
    /* +0x0c */ uint32_t         unread_0c;       // [U] stack garbage from ai_groups_new_group; saved and loaded; no reader (class census)
    /* +0x10 */ allocator_t      members_alloc;   // [H] the member vector's empty allocator
    /* +0x14 */ ai_group_member *members_begin;   // [H] the members, stride 8
    /* +0x18 */ ai_group_member *members_end;     // [H] count = (end - begin) >> 3
    /* +0x1c */ ai_group_member *members_cap;     // [H] one past the last allocated member slot
    /* +0x20 */ allocator_t      notified_alloc;  // [H] the notified vector's empty allocator
    /* +0x24 */ C_actor        **notified_begin;  // [H] event subjects announced once, the de-duplication set of §4; stride 4
    /* +0x28 */ C_actor        **notified_end;    // [H] count = (end - begin) >> 2
    /* +0x2c */ C_actor        **notified_cap;    // [H] one past the last allocated slot
    /* +0x30 */ allocator_t      events_alloc;    // [H] the event vector's empty allocator
    /* +0x34 */ ai_group_event  *events_begin;    // [H] the event queue, stride 0x14
    /* +0x38 */ ai_group_event  *events_end;      // [H] count = (end - begin) / 0x14
    /* +0x3c */ ai_group_event  *events_cap;      // [H] one past the last allocated slot
    /* +0x40 */ allocator_t      cars_alloc;      // [H] the candidate-car vector's empty allocator
    /* +0x44 */ C_car          **cars_begin;      // [H] the candidate cars, stride 4
    /* +0x48 */ C_car          **cars_end;        // [H] count = (end - begin) >> 2
    /* +0x4c */ C_car          **cars_cap;        // [H] one past the last allocated slot
    /* +0x50 */ allocator_t      factors_alloc;   // [H] the factor vector's empty allocator
    /* +0x54 */ float           *factors_begin;   // [H] one float per candidate car, same index, stride 4
    /* +0x58 */ float           *factors_end;     // [H] count = (end - begin) >> 2
    /* +0x5c */ float           *factors_cap;     // [H] one past the last allocated slot
} ai_group;
```

**Five vectors.** `ai_group_dtor` frees them in reverse order (`+0x54`, `+0x44`, `+0x34`, `+0x24`, `+0x14`). `ai_group_add_member` writes the group's id into `entity+0x11ac`, so an entity names its group and every reader resolves it with
`ai_groups_get_group`; it also empties the event queue on every successful append. The car pair is indexed together: `ai_group_add_car` pushes into both, with the script's value `* 0.01` as the factor, `ai_group_get_car_factor` finds a car in
the first and returns the float at the same index, 1.0 when absent, and `ai_group_get_closest_usable_car` walks the first. The queue is not saved: `ai_groups_save` and `ai_groups_get_save_size` cover members, notified subjects and the car pair.

### 1c. `ai_groups` — the registry (`0x28`)

Singleton at `0x006552dc`: of the 30 sites in `.text` that load that address into ECX, 29 call an `ai_groups_*` method, and the unwind funclet `add ecx, 0xc4; jmp ai_groups_dtor` shows it is `ai_manager+0xc4` (`structs/ai_manager.md`).

```c
typedef struct ai_groups {
    /* +0x00 */ uint32_t    control_lock_deadline; // [H] absolute ms; the player's controls stay locked until it passes; 0xffffffff = no lock
    /* +0x04 */ uint32_t    next_id;               // [H] pre-incremented by ai_groups_new_group
    /* +0x08 */ allocator_t entries_alloc;         // [H] the group vector's empty allocator
    /* +0x0c */ ai_group   *entries_begin;         // [H] the groups, stride 0x60
    /* +0x10 */ ai_group   *entries_end;           // [H] count = (end - begin) / 0x60
    /* +0x14 */ ai_group   *entries_cap;           // [H] one past the last allocated group slot
    /* +0x18 */ allocator_t tracked_alloc;         // [H] the release list's empty allocator
    /* +0x1c */ C_actor   **tracked_begin;         // [H] the release list for disbanded groups, stride 4
    /* +0x20 */ C_actor   **tracked_end;           // [H] count = (end - begin) >> 2
    /* +0x24 */ C_actor   **tracked_cap;           // [H] one past the last allocated slot
} ai_groups;
```

**The control lock.** `entity_action_tick` calls `ai_groups_extend_control_lock(now + 800)` (the deadline becomes the larger value, or the argument when no lock runs) one instruction before `player_lock_controls(1)`. `ai_groups_tick`
clears it once it passes and calls `player_lock_controls(0)`.

**The release list.** Its only producer is `ai_groups_add_tracked_actor`, called three times by `program_process_command` case `0x17b` (`GROUP_DISBAND`): for each disbanded member, for the group's car and for each of the car's occupants.
`ai_groups_tick` sets act state 2 on any of them further than 150 units from the player (22500.0 squared), and a car (kind 4) is also hidden, has its dynamic collisions turned off and is deactivated through its vtable. So a disbanded group is released once
the player has left it.

**`ai_groups_new_group` returns an id, not a pointer.** Callers follow it with `ai_groups_get_group(id)`.

### 1d. `ai_group_event` — the event queue element (`0x14`)

```c
typedef struct ai_group_event {
    /* +0x00 */ uint32_t kind;     // [H] always 1 from ai_group_push_event; only kind 1 is dispatched
    /* +0x04 */ uint32_t deadline; // [H] now + 1000 ms; the entry fires once deadline <= now
    /* +0x08 */ float    damage;   // [H] its address is the data argument of entity_dispatch_event; the receiver reads it as a float
    /* +0x0c */ C_actor *caster;   // [H] its +0x5d alive byte gates the dispatch; the member equal to it is skipped
    /* +0x10 */ C_actor *subject;  // [H] what the event is about: the sender position of the dispatch and the key of the notified set
} ai_group_event;
```

## 2. Field evidence

`ai_group_copy_ctor` and `ai_group_copy_assign` copy every field of a group and are left out of §2b, except as the only writer of the five allocator bytes. Group functions and the registry reach a group through a pointer, so §2a, §2b and §2d come from the bodies; `ai_groups_new_group` and `ai_groups_load` fill a stack record, then insert it.

### 2a. `ai_group_member`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | entity | H | `ai_group_add_member`, `ai_groups_load` | `ai_group_add_member`, `ai_group_can_cast`, `ai_group_can_receive`, `ai_group_del_member`, `ai_group_process_events`, `ai_group_send_sniff_event`, `ai_groups_invalidate_actor`, `ai_groups_save` | - |
| +0x4 | role | H | `ai_group_add_member`, `ai_groups_load` | `ai_group_can_cast`, `ai_group_can_receive`, `ai_groups_save` | - |

### 2b. `ai_group`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | id | H | `ai_groups_load`, `ai_groups_new_group` | `ai_group_add_member`, `ai_groups_get_group`, `ai_groups_group_has_car`, `ai_groups_save`, `entity_process_command` | - |
| +0x4 | unread_04 | U | `ai_groups_load`, `ai_groups_new_group` | `ai_groups_save` | 0 from `ai_groups_new_group`; no reader (class census), Open item 1 |
| +0x8 | unread_08 | U | `ai_groups_load`, `ai_groups_new_group` | `ai_groups_save` | 0 from `ai_groups_new_group`; no reader (class census), Open item 1 |
| +0xc | unread_0c | U | `ai_groups_load` | `ai_groups_save` | `ai_groups_new_group` leaves it as stack garbage in the record it inserts; no reader (class census), Open item 1 |
| +0x10 | members_alloc | H | `ai_group_copy_ctor` | - | a byte copy of the source's empty allocator; its address is the vector argument of the insert in `ai_group_add_member` |
| +0x14 | members_begin | H | `ai_group_dtor`, `ai_groups_dtor` | `ai_group_add_member`, `ai_group_can_cast`, `ai_group_can_receive`, `ai_group_del_member`, `ai_group_dtor`, `ai_group_process_events`, `ai_group_send_sniff_event`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_save` | - |
| +0x18 | members_end | H | `ai_group_del_member`, `ai_group_dtor`, `ai_groups_destroy`, `ai_groups_dtor`, `ai_groups_invalidate_actor` | `ai_group_add_member`, `ai_group_can_cast`, `ai_group_can_receive`, `ai_group_del_member`, `ai_group_process_events`, `ai_group_send_sniff_event`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_save` | - |
| +0x1c | members_cap | H | `ai_group_dtor`, `ai_groups_dtor` | - | - |
| +0x20 | notified_alloc | H | `ai_group_copy_ctor` | - | a byte copy of the source's empty allocator; its address is the vector argument of the insert in `ai_group_process_events` |
| +0x24 | notified_begin | H | `ai_group_dtor`, `ai_groups_dtor` | `ai_group_dtor`, `ai_group_process_events`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_save` | - |
| +0x28 | notified_end | H | `ai_group_dtor`, `ai_groups_destroy`, `ai_groups_dtor`, `ai_groups_invalidate_actor` | `ai_group_process_events`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_save` | - |
| +0x2c | notified_cap | H | `ai_group_dtor`, `ai_groups_dtor` | - | - |
| +0x30 | events_alloc | H | `ai_group_copy_ctor` | - | a byte copy of the source's empty allocator; `ai_group_copy_assign` passes its address to `std_vector20_copy_assign` |
| +0x34 | events_begin | H | `ai_group_dtor`, `ai_group_push_event`, `ai_groups_dtor` | `ai_group_add_member`, `ai_group_dtor`, `ai_group_process_events`, `ai_group_push_event`, `ai_groups_destroy`, `ai_groups_invalidate_actor` | - |
| +0x38 | events_end | H | `ai_group_add_member`, `ai_group_dtor`, `ai_group_process_events`, `ai_group_push_event`, `ai_groups_destroy`, `ai_groups_dtor`, `ai_groups_invalidate_actor` | `ai_group_add_member`, `ai_group_process_events`, `ai_group_push_event`, `ai_groups_destroy`, `ai_groups_invalidate_actor`, `ai_groups_tick` | - |
| +0x3c | events_cap | H | `ai_group_dtor`, `ai_group_push_event`, `ai_groups_dtor` | `ai_group_push_event` | - |
| +0x40 | cars_alloc | H | `ai_group_copy_ctor` | - | a byte copy of the source's empty allocator; its address is the vector argument of `std_vector4_insert_n` in `ai_group_add_car` and of `std_vector4_copy_assign` in `ai_group_copy_assign` |
| +0x44 | cars_begin | H | `ai_group_dtor`, `ai_groups_dtor` | `ai_group_dtor`, `ai_group_get_car_factor`, `ai_group_get_closest_usable_car`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_group_has_car`, `ai_groups_save` | - |
| +0x48 | cars_end | H | `ai_group_dtor`, `ai_groups_destroy`, `ai_groups_dtor` | `ai_group_add_car`, `ai_group_get_car_factor`, `ai_group_get_closest_usable_car`, `ai_groups_destroy`, `ai_groups_get_save_size`, `ai_groups_group_has_car`, `ai_groups_save` | - |
| +0x4c | cars_cap | H | `ai_group_dtor`, `ai_groups_dtor` | - | - |
| +0x50 | factors_alloc | H | `ai_group_copy_ctor` (by address, `dword_vec_copy_assign`) | - | `ai_group_copy_assign` passes its address to `std_vector4_copy_assign` |
| +0x54 | factors_begin | H | `ai_group_add_car`, `ai_group_dtor`, `ai_groups_dtor` | `ai_group_add_car`, `ai_group_dtor`, `ai_group_get_car_factor`, `ai_groups_save` | - |
| +0x58 | factors_end | H | `ai_group_add_car`, `ai_group_dtor`, `ai_groups_destroy`, `ai_groups_dtor` | `ai_group_add_car`, `ai_group_get_car_factor`, `ai_groups_destroy` | - |
| +0x5c | factors_cap | H | `ai_group_add_car`, `ai_group_dtor`, `ai_groups_dtor` | `ai_group_add_car` | - |

### 2c. `ai_groups`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | control_lock_deadline | H | `ai_groups_clear_all_groups`, `ai_groups_destroy`, `ai_groups_extend_control_lock`, `ai_groups_tick` | `ai_groups_extend_control_lock`, `ai_groups_tick` | the literal `0xffffffff` from the first two and from the tick |
| +0x4 | next_id | H | `ai_groups_clear_all_groups`, `ai_groups_destroy`, `ai_groups_load`, `ai_groups_new_group` | `ai_groups_new_group`, `ai_groups_save` | - |
| +0x8 | entries_alloc | H | `ai_manager_ctor` | - | the byte at `ai_manager +0xcc`; its address is the vector argument of `ai_group_vector_insert_at` in `ai_groups_new_group` and `ai_groups_load` |
| +0xc | entries_begin | H | `ai_group_vector_insert_at`, `ai_groups_dtor` | `ai_groups_clear_all_groups`, `ai_groups_destroy`, `ai_groups_dtor`, `ai_groups_get_group`, `ai_groups_get_save_size`, `ai_groups_group_has_car`, `ai_groups_invalidate_actor`, `ai_groups_save`, `ai_groups_tick` | - |
| +0x10 | entries_end | H | `ai_group_vector_insert_at`, `ai_groups_clear_all_groups`, `ai_groups_destroy`, `ai_groups_dtor`, `ai_groups_invalidate_actor` | `ai_groups_clear_all_groups`, `ai_groups_dtor`, `ai_groups_get_group`, `ai_groups_get_save_size`, `ai_groups_group_has_car`, `ai_groups_invalidate_actor`, `ai_groups_load`, `ai_groups_tick` | - |
| +0x14 | entries_cap | H | `ai_group_vector_insert_at`, `ai_groups_dtor` | `ai_group_vector_insert_at` | - |
| +0x18 | tracked_alloc | H | `ai_manager_ctor` | - | the byte at `ai_manager +0xdc`; its address reaches `ai_groups_add_tracked_actor`, the destroy and the load |
| +0x1c | tracked_begin | H | `ai_groups_add_tracked_actor`, `ai_groups_dtor` | `ai_groups_add_tracked_actor`, `ai_groups_clear_all_groups`, `ai_groups_dtor`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_load`, `ai_groups_save`, `ai_groups_tick` | - |
| +0x20 | tracked_end | H | `ai_groups_add_tracked_actor`, `ai_groups_clear_all_groups`, `ai_groups_dtor`, `ai_groups_invalidate_actor`, `ai_groups_tick` | `ai_groups_add_tracked_actor`, `ai_groups_clear_all_groups`, `ai_groups_get_save_size`, `ai_groups_invalidate_actor`, `ai_groups_load`, `ai_groups_save`, `ai_groups_tick` | - |
| +0x24 | tracked_cap | H | `ai_groups_add_tracked_actor`, `ai_groups_dtor` | `ai_groups_add_tracked_actor` | - |

### 2d. `ai_group_event`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | kind | H | `ai_group_push_event` | `ai_group_process_events` | - |
| +0x4 | deadline | H | `ai_group_push_event` | `ai_group_process_events` | - |
| +0x8 | damage | H | `ai_group_push_event` | `ai_group_process_events` | the three producers pass 1.0 (`entity_force_ai`), 50.0 (`entity_dispatch_event`), 100.0 (`entity_process_command`) |
| +0xc | caster | H | `ai_group_push_event` | `ai_group_process_events`, `ai_groups_invalidate_actor` | - |
| +0x10 | subject | H | `ai_group_push_event` | `ai_group_process_events`, `ai_groups_invalidate_actor` | - |

### 2e. `role`

Read by exactly two functions, which between them exclude two different values, and named by the script keywords that set it:

| Value | Meaning | Evidence |
|---|---|---|
| 0 | full participant: may cast, may receive | the script default; `ai_group_can_cast` is `role != 1`, `ai_group_can_receive` is `role != 2` |
| 1 | receive-only (script `RECEIVE`): fed group events, may not originate them | `ai_group_can_cast` refuses it |
| 2 | cast-only (script `CAST`): may broadcast, is not sent events | `ai_group_can_receive` refuses it |

**The writers are a closed set.** `ai_group_add_member` stores its third argument, and its five callsites pass the script value or 0: `entity_process_command` case `0x102b` (the AI instruction `enemy_group_add <group>, <actor> [, RECEIVE | CAST]`, decoded by
`entity_script_decode_params` case `0x102b`), `police_board_create_and_engage` ×2 and `traffic_car_spawn_mafia_pursuit_car` ×2 (both 0). A savegame load restores saved values. In the shipped scripts `CAST` is used by MISE08-HOTEL `recep` and MISE18-PRISTAV `City_Guard3`,
`City_Guard5` and `Guard_bouda`; `RECEIVE` by MISE07-SARA `Billy`.

## 3. Method family

`ai_group` 11 of 11 PS2 methods mapped; `ai_groups` 10 of 12, the other two a factoring (§7a).

### 3a. Construction & destruction

#### the translation unit

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0043b000 | `crt_static_init_guard_0043b000` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0043b020 | `crt_atexit_register_0043b020` | the translation unit | - | - | - | registers the shared no-op static destructor |

#### `ai_group`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0043bd00 | `ai_group_dtor` | instance | `__dt__8ai_groupFv` | 0x0034ca40 | CONFIRMED | Frees the five vectors in reverse order; `ai_groups_clear_all_groups` calls it once per record |
| 0x004424e0 | `ai_group_copy_ctor` | instance | `__ct__8ai_groupFRC8ai_group` | 0x0034c500 | CONFIRMED | Copies the id, the three unread dwords and the five vectors |
| 0x00442910 | `ai_group_copy_assign` | instance | - | - | - | MSVC-generated `operator=` |
| 0x00442f50 | `ai_group_scalar_deleting_dtor` | instance | - | - | - | MSVC artifact |

#### `ai_groups`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443910 | `ai_groups_dtor` | instance | - | - | - | Frees the release list, then the group vector with all five inner vectors per record, then the buffer; reached only through the exception path, the normal path is inlined into `ai_manager_dtor` |

#### the unit's shared helpers

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00442f70 | `std_vector4_copy_assign` | a notified, car or factor vector | - | - | - | assigns a stride-4 vector |
| 0x004430f0 | `std_vector8_copy_assign` | a member vector | - | - | - | assigns a stride-8 vector |
| 0x00443290 | `std_vector20_copy_assign` | an event vector | - | - | - | assigns a stride-0x14 vector; a fold shared with `ai_fight_member` |

### 3b. Accessors

No row.

### 3c. Behaviour

#### `ai_group`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0042dcb0 | `ai_group_add_car` | - | - | - | Pushes a car and its factor into the candidate pair; `entity_process_command` |
| 0x0043b070 | `ai_group_add_member` | `Add__8ai_groupFP8C_entityUi` | 0x0034b8e0 | CONFIRMED | Unique-appends a member, stores the role, empties the event queue, stamps `entity+0x11ac` |
| 0x0043b100 | `ai_group_del_member` | `Del__8ai_groupFP8C_entity` | 0x0034b980 | CONFIRMED | Removes every record of an entity; the loop does not stop at the first hit |
| 0x0043b160 | `ai_group_send_sniff_event` | `GetSniff__8ai_groupFP8C_entityP7C_actor` | 0x0034ba00 | CONFIRMED | Mints an `ai_blob` and posts it as event `0xb` (§5) |
| 0x0043b360 | `ai_group_push_event` | `UnderAttack__8ai_groupFP8C_entityP7C_actorf` | 0x0034bb10 | CONFIRMED | Queues a kind-1 event when `ai_group_can_cast` allows it, deadline now + 1000 |
| 0x0043b5c0 | `ai_group_process_events` | `ProcessEvents__8ai_groupFUi` | 0x0034bbb0 | CONFIRMED | Drains the event queue (§4); `ai_groups_tick` |
| 0x0043b710 | `ai_group_get_car_factor` | `GetCarFactor__8ai_groupFP5C_car` | 0x0034bd70 | CONFIRMED | The factor stored with a car, 1.0 when the car is not a candidate |
| 0x0043b770 | `ai_group_get_closest_usable_car` | `GetCarClosestAndUsable__8ai_groupFP8S_vectorP8S_vectorPUi` | 0x0034bde0 | CONFIRMED | The nearest usable car within 50 units: the group's candidates first, then the world lists and the parked cars of `traffic_car_get_parking_cars`; `ai_fight_manager_tick` |
| 0x0043ccf0 | `ai_group_can_receive` | `canReceive__8ai_groupFP8C_entity` | 0x0034e250 | CONFIRMED | `role != 2`; false for a non-member |
| 0x0043cd30 | `ai_group_can_cast` | `canCast__8ai_groupFP8C_entity` | 0x0034e2b0 | CONFIRMED | `role != 1`; false for a non-member |
| 0x004410a0 | `ai_group_destroy_range` | - | - | - | `_Destroy(first, last)`; sole caller `ai_groups_destroy` |
| 0x00441300 | `ai_group_vector_insert_at` | - | - | - | `std::vector<ai_group>::insert` |
| 0x004434f0 | `ai_group_member_vector_capacity` | - | - | - | capacity of a member vector |
| 0x00443b20 | `ai_group_vector_destroy` | - | - | - | `std::vector<ai_group>::~vector`, the container's destructor, receiver `ai_groups+0x08` |

#### `ai_groups`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004257c0 | `ai_groups_extend_control_lock` | - | - | - | Moves the control-lock deadline later, or sets it when no lock runs; `entity_action_tick` |
| 0x0043bb30 | `ai_groups_new_group` | `NewGroup__9ai_groupsFv` | 0x0034c2a0 | CONFIRMED | Allocates and registers a group, returns its id; SEH-guarded |
| 0x0043bd70 | `ai_groups_get_group` | `GetGroup__9ai_groupsFUi` | 0x0034cc20 | CONFIRMED | Linear scan of the groups by id |
| 0x0043bdd0 | `ai_groups_clear_all_groups` | `Init__9ai_groupsFv` | 0x0034cca0 | CONFIRMED | Control lock `0xffffffff`, `next_id` 0, disposes every group, resets the release list; clears without freeing; `game_init` |
| 0x0043be40 | `ai_groups_destroy` | `Destroy__9ai_groupsFv` | 0x0034cd60 | CONFIRMED | The same, with a leading reverse pass releasing each record's five vectors; `game_done`, and first in `ai_groups_load` |
| 0x0043bfe0 | `ai_groups_tick` | `Tick__9ai_groupsFv` | 0x0034ced0 | CONFIRMED | The per-frame tick: expires the control lock, drains each group's events, culls the release list at 150 units; `game_tick` |
| 0x0043c170 | `ai_groups_get_save_size` | `GetSize__9ai_groupsFv` | 0x0034d140 | CONFIRMED | `8 + sum(0x1c + members * 0x24 + notified * 0x20 + cars * 0x24) + 4 + tracked * 0x20` |
| 0x0043c250 | `ai_groups_save` | `Save__9ai_groupsFPPv` | 0x0034d210 | CONFIRMED | `next_id`, the group count, then per group `+0x00..+0x0c` and each saved vector as count then elements |
| 0x0043c590 | `ai_groups_load` | `Load__9ai_groupsFPPv` | 0x0034d640 | CONFIRMED | Opens with `ai_groups_destroy`, then mirrors the save; SEH-guarded |
| 0x0043ca60 | `ai_groups_invalidate_actor` | `InvalidateActor__9ai_groupsFP7C_actor` | 0x0034de70 | CONFIRMED | Scrubs three vectors per group (stride 8 on `+0x00`; stride 4; stride `0x14` on `+0x0c` or `+0x10`), splices an emptied group, then scrubs the release list |
| 0x0043cd70 | `ai_groups_group_has_car` | `hasCarGroup__9ai_groupsFUi` | 0x0034e310 | CONFIRMED | True when the group with that non-zero id has a candidate car |
| 0x005c8dc0 | `ai_groups_add_tracked_actor` | - | - | - | `push_back` on the release list, out of line, with the singleton as receiver; all 3 callsites are in `program_process_command`'s `GROUP_DISBAND` arm |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00441f90 | `fill_n_stride_0x14` | - | - | - | stride-0x14 fill; `ai_group_push_event` |
| 0x004426c0 | `stl_copy_dwords` | - | - | - | copies a dword range; `ai_group_add_car`, the group's copies |
| 0x004427d0 | `copy_element_stride_0x14` | - | - | - | stride-0x14 element copy; `ai_group_push_event` |

## 4. The event queue

`ai_group_process_events` walks the queue backwards and, for each entry whose `deadline` has passed, splices the entry out whatever else happens. An entry of `kind` 1 whose `caster` is alive also dispatches, unless its `subject` is in the notified set, which is what stops a subject being announced
twice. A dispatch delivers `entity_dispatch_event(subject, 1, &damage)` to every member that is neither the `caster` nor refused by `ai_group_can_receive`, then appends `subject` to the set. The receiver travels in ECX and is not rendered by the decompiler. Both pointers are typed
`C_actor *` because `ai_groups_invalidate_actor` erases an event when either matches the departing actor.

**Where `damage` ends up.** `entity_dispatch_event` forwards every event to `ai_logs_event(subject, kind, data)` before its own switch, and its own `case 1` is a no-op, so the meaning of a kind-1 entry lives in `ai_logs`. There `case 1` calls `ai_character_log_event_friend_hit`, which reads the data
pointer as a `float`, adds it to the subject's `ai_character_log +0x14` (`friend_damage`, `structs/ai_logs.md`), pushes that log's `+0x1c` expiry to now + 30000 ms and bumps its `+0x04` counter. So a kind-1 entry means *the caster was attacked by the subject for this much damage*, broadcast to the other
members, each of which books it against the subject as damage done to a friend.

Stride `0x14` is shared with `ai_fight_member`, which is why `fill_n_stride_0x14`, `copy_element_stride_0x14`, `std_vector20_copy_assign` and `vec_count_stride_0x14` are COMDAT-folded and keep mechanical names.

## 5. `ai_group_send_sniff_event` — a SECOND scent producer

The function:

1. checks `ai_group_can_receive(entity)`;
2. walks the members until `entity_get_group_position(actor, &pos)` succeeds;
3. allocates a **0x24-byte `ai_blob`** — the scent record type documented in
   `structs/ai_blob_log.md` §1a — and fills it with that position, the actor's normalised forward
   vector, the mission clock, a zero flag word, and the actor as owner;
4. posts it as `entity_dispatch_event(entity, 0, **0xb**, blob)`.

The consumer chain is verified: `entity_dispatch_event` case `0xb` wraps the record in a 0x1c-byte
entry — timestamp `+0x04`, flag = 1 `+0x08`, blob pointer `+0x0c`, back-ref actor `+0x10`,
weight = 1.0f `+0x14` — and upserts it into the target-observation table through
**`ai_sensors_new_smell_contact`**.

**So Mafia has two ways a scent record enters the world:**

| producer | trigger | destination |
|---|---|---|
| `ai_blob_log_add_blob` | passive, one blob per human per 500 ms | that human's own 16-slot trail (`ai_blob_log`) |
| `ai_group_send_sniff_event` | on demand, one blob per call | **one specific entity's smell-contact table**, via event `0xb` |

The second is a *communication* mechanism, not a trail: a group hands one member a scent lead
directly. Its caller is `entity_action_tick`, which sends it only while the entity's group id
at `+0x11ac` resolves.

**What the receiver does with it.** The smell contact is read back by
`ai_sensors_get_smelled_target_info`, which returns the blob's position (`+0x00`), direction
(`+0x0c`), time (`+0x18`) and flags (`+0x1c`) while the contact's weight exceeds `1.0 − sight`
(`structs/ai_sensors.md` §6). It has nine callers: `entity_action_tick` ×6 (the
steer-to-target arms, a freshness test of under 5000 ms, and the cop chase in case `0x21`),
`ai_logs_reset_stale_logs`, `ai_action_new_hostile_coverage` and `police_board_update_sighting`.
So the lead is a last-known position and heading for the target, used when the member cannot
see it.

## 6. The translation unit

`[0x0043b000, 0x00443510)`, guard to guard, 84 starts, all named. The band holds, in VA order: the CRT pair, the four `program_base_thunk_entity_*` thunks of `C_entity`'s `C_program` base (`structs/entity.md`), `ai_group` and `ai_groups` to `0x0043cdf0`, `ai_follow_manager` to
`0x0043e610`, the `ai_fight_*` methods to `0x00441070`, then the compiler-emitted container helpers of all three. Each helper is listed in the doc of every class whose methods call it: `vec_count_stride_0x14` and `std_vector20_insert` only in `structs/ai_fight_manager.md`, and `stub_retfalse_args4_0043e0a0`, the folded `mustEnter`,
in `structs/ai_follow_manager.md`. No helper of the unit is left uncalled by all three.

Outside the unit: `ai_groups_extend_control_lock` and `ai_group_add_car` sit in the `C_entity` unit (`structs/entity.md` §8), `ai_groups_add_tracked_actor` in the `C_program` unit (`structs/program.md`), and the destructors `ai_groups_dtor`, `ai_group_vector_destroy`, `ai_follow_manager_dtor` and `ai_fight_manager_dtor` in the `ai_manager` unit (`structs/ai_manager.md`).

## 7. PC vs PS2

- **The member vector differs:** PC `begin` / `end` pointers at `+0x14` / `+0x18`, PS2 a count at `+0x14` and a data pointer at `+0x18` (`std::__vector_imp`). Expect this shape difference in every method of both classes.
- **Group record stride:** PC `0x60`, PS2 `0x4c`; the five vectors at PC `+0x14`, `+0x24`, `+0x34`, `+0x44`, `+0x54`, PS2 `+0x10`, `+0x1c`, `+0x28`, `+0x34`, `+0x40`.
- **`ai_group_member` is 8 bytes on both builds**, same offsets; `erase__Q23std70__vector_imp<>F**P15ai_group_member**Ui` names it.
- **`ai_group_event`:** `ai_groups_invalidate_actor`'s twin calls `erase__Q23std68__vector_imp<>F**P14ai_group_event**Ui`, and the copy constructor's twin spells the same element type.
- **`ai_blob` stride: PC `0x24`, PS2 `0x30`** (`structs/ai_blob_log.md` §5). The PS2 allocates it through `__nw__7ai_blob`, which tail-calls the PS2-only `ai_spageman` pool; the PC uses `operator_new`.
- **The mission clock:** PC `*(g_act_mission+0x24)+0x2b0c`, PS2 `*(act_mission+0x20)+0xe54`.
- **Savegame element sizes** differ only for the build-wide frame-name-against-GUID reason (`structs/ai_follow_manager.md` §5).
- **Out-of-band size ratios are factoring in one direction:** the PS2 calls `erase__…<ai_group_member>` and `Normalize__8S_vector` out of line where MSVC inlines both.
- **`ai_groups_dtor` and `ai_groups_add_tracked_actor` have no twin:** the PS2 inlines every sub-manager teardown into `__dt__10ai_managerFv`. The `GROUP_DISBAND` arm of PS2 `Process__9C_programFUi` runs `LeaveParty__16ai_fight_manager` and then `push_back__Q23gif19vector<P7C_actor,8>(0x509170)` three times, at the same
  three points as the PC; `0x509170` is `ai_groups` (`0x50915c`) + `0x14`, the vector `Tick__9ai_groupsFv` culls. The PS2 symbol is the generic template, so it is no twin of the PC function.
- **`ai_group_add_car` and `ai_groups_extend_control_lock` have no PS2 method of their own:** all 11 + 12 PS2 methods are accounted for without them.
- **`ai_group_add_member`'s `Ui` parameter** (`Add__8ai_groupFP8C_entityUi`) is the role.

### 7a. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0034e3a0 | `MemWrite__9ai_groupsFPPvPvUiUi` | factoring: the PS2 wraps the savegame stream primitive in a 72-byte per-class method; the PC calls `entity_mem_write` at every site. The identical pair exists on `ai_follow_manager` |
| 0x0034e3f0 | `MemRead__9ai_groupsFPPvPvUiUi` | the same, against `entity_mem_read` |

### 7b. The twin evidence

| PC name | Decisive evidence |
|---|---|
| `ai_group_del_member` | PC inlines the shift where the PS2 calls `erase__…<ai_group_member>`; 96 / 124 |
| `ai_group_send_sniff_event` | 512 / 272 = 1.88, entirely the PC inlining `Normalize__8S_vector` |
| `ai_group_push_event` | the PS2 names the generic push `UnderAttack` |
| `ai_group_dtor` | the same reverse free order, PC `+0x54`, `+0x44`, `+0x34`, `+0x24`, `+0x14`, PS2 `+0x40`, `+0x34`, `+0x28`, `+0x1c`, `+0x10`; `Init__9ai_groups` calls `__dt__8ai_group` per stride-0x4c element at the same point |
| `ai_group_can_receive`, `ai_group_can_cast` | split by the role constant; 64 / 92 each. `canReceive` is pinned by `GetSniff` too |
| `ai_group_copy_ctor` | statement-for-statement, 480 / 304 = 1.58; spells out both element types |
| `ai_groups_clear_all_groups` | the same opening pair, `*this = 0xffffffff` and `+0x04 = 0`, then the dispose through the destructor, stride PC `0x60`, PS2 `0x4c` |
| `ai_groups_destroy` | the extra leading reverse pass, as in `ai_fight_manager_destroy`, `Init` emitted first; re-pinned by `ai_groups_load` calling it first |
| `ai_groups_tick` | statement-for-statement |
| `ai_groups_get_save_size` | shared 8 base, shared `0x1c` per group, the same three containers in order, the same trailing `+ 4 + tracked * K` |
| `ai_groups_save`, `ai_groups_load` | identical grammar, the same `(1, 4)` element arguments |
| `ai_groups_invalidate_actor` | three containers in the same order with the same geometry; the structural twin of `ai_fight_manager_invalidate_actor` |
| `ai_groups_group_has_car` | same `id != 0` guard, same scan, the same container read from `+0x44` / `+0x48` |

## 8. Related

- `structs/ai_manager.md` — the AI root object; this registry is its `+0xc4` member.
- `structs/ai_fight_manager.md`, `structs/ai_follow_manager.md` — the sibling subsystems in the unit.
- `structs/ai_blob_log.md` — the `ai_blob` record and the other scent producer.
- `structs/ai_sensors.md` — the smell band that consumes event `0xb`, and the read side.
- `structs/ai_logs.md` — the per-target `ai_character_log`, where a kind-1 entry's `damage` accumulates.
- `structs/police-board.md` — `police_board_create_and_engage`, which pairs `ai_groups_new_group` with `ai_groups_get_group`.

## 9. Open items

1. `unread_04`, `unread_08`, `unread_0c`: saved, loaded and copied, and no reader is located in the group methods, the registry methods or the 14 callers of `ai_groups_get_group` read for this doc.
