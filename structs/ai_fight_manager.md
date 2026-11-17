# `ai_fight_manager` / `ai_fight_followship` — fight parties and the party manager

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-11-15 13:43 +0100

How NPCs fight as a group instead of individually. **`ai_fight_manager`** owns the vector of parties, forms and disbands them,
ticks them, and scrubs dead actors out of them. **`ai_fight_followship`** is one party: a commander, a target, a member list, an
optional car and an evaluation time. ("Followship" is the developers' spelling and is kept.)

```
  ai_fight_manager      vector<ai_fight_followship>, stride 0x48   §1c
        |  owns N of
        v
  ai_fight_followship   commander + target + car + members         §1a
        |  holds a vector of
        v
  ai_fight_member       stride 0x14                                §1b
```

| | |
|---|---|
| PC prefix | `ai_fight_manager_`, `ai_fight_followship_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `ai_fight_manager` `0x10`, a member of the root at `ai_manager+0x100` (`0x00655318`); `ai_fight_followship` `0x48`; `ai_fight_member` `0x14` |
| PC constructor | `ai_fight_manager` -, built by `ai_manager_ctor`; `ai_fight_followship_copy_ctor` |
| PC translation unit | `[0x0043b000, 0x00443510)`, 84 starts, shared with `ai_group` and `ai_follow_manager`, hosted by `structs/ai_group.md` §6; `ai_fight_manager_dtor` sits in the `ai_manager` unit |
| PS2 class | `ai_fight_manager` (tag `16ai_fight_manager`), 10 methods; `ai_fight_followship` (tag `19ai_fight_followship`), 8 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `ai_fight_followship` — one party (`0x48`)

```c
typedef struct ai_fight_followship {
    /* +0x00 */ allocator_t      members_alloc;  // [H] the member vector's empty allocator, padded to 4
    /* +0x04 */ ai_fight_member *members_begin;  // [H] the member vector, stride 0x14
    /* +0x08 */ ai_fight_member *members_end;    // [H] count = (end - begin) / 0x14
    /* +0x0c */ ai_fight_member *members_cap;    // [H] one past the last allocated member slot
    /* +0x10 */ S_vector         centre;         // [H] the mean member position, written on each evaluation
    /* +0x1c */ S_vector         stop_position;  // [H] read by entity_ai's car-stop / car-aim actions; only non-zero writer the unreachable mode-2 arm (§4a)
    /* +0x28 */ float            spread_sq;      // [H] the largest squared distance from a member to centre
    /* +0x2c */ uint32_t         next_eval_ms;   // [H] the tick evaluates the party once the clock passes this, then sets now + 2000
    /* +0x30 */ uint32_t         kind;           // [H] 1 = car party, 0 = on-foot party; no other value is written
    /* +0x34 */ uint32_t         car_mode;       // [H] 0 on creation; 1 = board the car, the only value the search can produce; 2 = stop it at stop_position (§4a)
    /* +0x38 */ uint32_t         stuck_ms;       // [H] 4000 while the car moves, + 2000 per evaluation while it stands; 1000000 = given up
    /* +0x3c */ C_entity        *commander;      // [H] elected by ai_fight_followship_set_commander; its group decides who may join
    /* +0x40 */ C_actor         *target;         // [H] the actor the party fights, from the entity's ai_target_angry at +0x1280; the party key
    /* +0x44 */ C_car           *car;            // [H] the party's vehicle when kind == 1
} ai_fight_followship;
```

**How the commander is elected.** `ai_fight_followship_set_commander` takes the first member whose `ai_group` has candidate cars
(`ai_groups_group_has_car`), else the first member with a group at all (`entity+0x11ac`), else the first member. It then rewrites
each member's `group_state` against the commander's group.

**Why `target` is the enemy.** `entity_ai` calls `ai_fight_manager_join_party(this, entity+0x1280)`, the entity's best-by-anger target (`structs/entity.md` §1). The tick hands `+0x40` to
`pursuit_event_dispatch_pursuit_vehicle` as the actor to chase and posts its position to the members as a scent, and
`ai_fight_followship_must_car_hunt` measures the distance from the party to it.

**Who may join.** `ai_fight_manager_join_party` matches a party with the same `target`, a commander whose `+0xf4c` type group equals
the joining entity's (`entity_game_init` sets it from `human_type`), and `kind != 1`.

**`car+0x688` reads as speed [M].** Speed fits both consumers: `ai_fight_followship_add_car_member` requires `< 0.5` before treating a car
as boardable, and `ai_fight_followship_must_car_hunt` requires `> 2.0` before deciding a target's car must be chased by car.

### 1b. `ai_fight_member` — one member (`0x14`)

```c
typedef struct ai_fight_member {
    /* +0x00 */ C_entity *entity;       // [H] the member, also the lookup key
    /* +0x04 */ uint32_t  order;        // [H] the member's order code (§4b); ai_fight_manager_join_party leaves it unset
    /* +0x08 */ uint32_t  unread_08;    // [U] 0 from ai_fight_followship_add_car_member; no reader (class census)
    /* +0x0c */ uint32_t  group_state;  // [H] 1 = in the commander's ai_group, 0 = not, 2 = being moved to a car party
    /* +0x10 */ uint32_t  unread_10;    // [U] 0 from ai_fight_manager_join_party; no reader (class census)
} ai_fight_member;
```

### 1c. `ai_fight_manager` — the party vector (`0x10`)

```c
typedef struct ai_fight_manager {
    /* +0x00 */ allocator_t          parties_alloc; // [H] the party vector's empty allocator, padded to 4
    /* +0x04 */ ai_fight_followship *parties_begin; // [H] the parties, stride 0x48
    /* +0x08 */ ai_fight_followship *parties_end;   // [H] count = (end - begin) / 0x48
    /* +0x0c */ ai_fight_followship *parties_cap;   // [H] one past the last allocated party slot
} ai_fight_manager;
```

The singleton `0x00655318` is a member of the `ai_manager` root at `+0x100`: the unwind funclet `add ecx, 0x100; jmp ai_fight_manager_dtor`
shows it (`structs/ai_manager.md`), and 24 callsites load `0x00655318` into ECX, each calling a method in §3c. The class is one vector,
so it ends at root `+0x110`; the vectors at `ai_manager+0x110` / `+0x120` are the root's next member (`structs/ai_manager.md` §6b).

## 2. Field evidence

The record functions `ai_fight_followship_copy_ctor`, `_copy_assign`, `_assign_range`, `_assign_range_backward`, `_uninit_copy_range`,
`_uninit_fill_n` and `_vector_insert_n` copy every field; the cells leave them out. Manager functions reach a party through a walked
pointer, so the cells of §2a and §2b come from the bodies and a displacement scan of the class's functions.

### 2a. `ai_fight_followship`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | members_alloc | H | `ai_fight_manager_new_car_party` | - | a byte copy of an empty allocator |
| +0x4 | members_begin | H | `ai_fight_followship_add_car_member`, `ai_fight_manager_new_car_party` | `ai_fight_followship_add_car_member`, `ai_fight_followship_compute_position`, `ai_fight_followship_count_free_seats`, `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_dtor`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0x8 | members_end | H | `ai_fight_followship_add_car_member`, `ai_fight_followship_take_car`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_dtor`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_leave_party`, `ai_fight_manager_new_car_party` | `ai_fight_followship_add_car_member`, `ai_fight_followship_compute_position`, `ai_fight_followship_count_free_seats`, `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0xc | members_cap | H | `ai_fight_followship_add_car_member`, `ai_fight_manager_dtor`, `ai_fight_manager_new_car_party` | `ai_fight_followship_add_car_member` | - |
| +0x10 | centre | H | `ai_fight_followship_compute_position` | `ai_fight_followship_must_car_hunt`, `ai_fight_manager_tick` | - |
| +0x1c | stop_position | H | `ai_fight_manager_new_car_party`, `ai_fight_manager_tick` | `entity_ai` | `entity_ai` passes its address to `ai_action_new_car_stop` and `ai_action_new_car_aim`, order codes 9 and 10. `ai_fight_manager_new_car_party` stores 0; the tick's only non-zero store is its `car_mode` 2 arm, which cannot run (§4a), so the field stays 0 |
| +0x28 | spread_sq | H | `ai_fight_followship_compute_position` | `ai_fight_followship_must_car_hunt`, `ai_fight_manager_tick` | - |
| +0x2c | next_eval_ms | H | `ai_fight_manager_adjust_form_time`, `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party`, `ai_fight_manager_tick` | `ai_fight_manager_tick` | - |
| +0x30 | kind | H | `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party` | `ai_fight_manager_adjust_form_time`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick`, `entity_ai`, `entity_update`, `program_process_command` | `ai_fight_manager_join_party` zero-fills a new record |
| +0x34 | car_mode | H | `ai_fight_followship_take_car`, `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party`, `ai_fight_manager_tick` | `ai_fight_manager_tick` | `ai_fight_followship_take_car` writes it on the new party |
| +0x38 | stuck_ms | H | `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party`, `ai_fight_manager_tick` | `ai_fight_manager_tick` | - |
| +0x3c | commander | H | `ai_fight_followship_set_commander` | `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0x40 | target | H | `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party` | `ai_fight_followship_must_car_hunt`, `ai_fight_followship_take_car`, `ai_fight_manager_adjust_form_time`, `ai_fight_manager_disband_followship`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0x44 | car | H | `ai_fight_manager_join_party`, `ai_fight_manager_new_car_party` | `ai_fight_followship_add_car_member`, `ai_fight_manager_disband_followship`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_tick`, `entity_ai`, `program_process_command` | - |

### 2b. `ai_fight_member`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | entity | H | `ai_fight_followship_add_car_member`, `ai_fight_manager_join_party` | `ai_fight_followship_compute_position`, `ai_fight_followship_count_free_seats`, `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_disband_followship`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0x4 | order | H | `ai_fight_followship_add_car_member`, `ai_fight_manager_tick` | `ai_fight_manager_tick` | the tick hands it to `entity_lock_substate`; nothing outside the manager reads the record's field |
| +0x8 | unread_08 | U | `ai_fight_followship_add_car_member` | - | 0 from `ai_fight_followship_add_car_member`, left unset by `ai_fight_manager_join_party`; no reader (class census), Open item 1 |
| +0xc | group_state | H | `ai_fight_followship_add_car_member`, `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_join_party` | `ai_fight_followship_take_car` | the value 2 is the only one a reader tests |
| +0x10 | unread_10 | U | `ai_fight_manager_join_party` | - | 0 when `ai_fight_manager_join_party` creates a party, left unset by `ai_fight_followship_add_car_member`; no reader (class census), Open item 1 |

### 2c. `ai_fight_manager`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | parties_alloc | H | `ai_manager_ctor` | - | the byte at `ai_manager +0x100`; the `inc [ebx]` decodings at `0x00440dc9`..`0x00440dd9` in `ai_fight_manager_tick` lie after its `ret` at `0x00440dc4` and are its jump table, not code |
| +0x4 | parties_begin | H | `ai_fight_followship_vector_insert_at`, `ai_fight_manager_dtor` | `ai_fight_followship_vector_count`, `ai_fight_manager_adjust_form_time`, `ai_fight_manager_clear_all_parties`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_dtor`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0x8 | parties_end | H | `ai_fight_followship_vector_insert_at`, `ai_fight_followship_vector_insert_n`, `ai_fight_manager_clear_all_parties`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_dtor`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | `ai_fight_followship_vector_count`, `ai_fight_manager_adjust_form_time`, `ai_fight_manager_clear_all_parties`, `ai_fight_manager_destroy`, `ai_fight_manager_disband_followship`, `ai_fight_manager_dtor`, `ai_fight_manager_get_party`, `ai_fight_manager_invalidate_actor`, `ai_fight_manager_join_party`, `ai_fight_manager_leave_party`, `ai_fight_manager_tick` | - |
| +0xc | parties_cap | H | `ai_fight_followship_vector_insert_at`, `ai_fight_followship_vector_insert_n`, `ai_fight_manager_dtor` | `ai_fight_followship_vector_insert_at`, `ai_fight_followship_vector_insert_n` | - |

## 3. Method family

18 of 18 PS2 methods mapped; every twin is CONFIRMED by a statement-level diff (§7). The two classes interleave by VA.

### 3a. Construction & destruction

#### `ai_fight_manager`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443a90 | `ai_fight_manager_dtor` | instance | - | - | - | Walks the party vector, drains and frees each party's member vector, frees the buffer; one entry point, the `ai_manager` unwind funclet. No twin: inlined into the PS2 `ai_manager` destructor |

#### `ai_fight_followship`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004427f0 | `ai_fight_followship_copy_ctor` | instance | `__ct__19ai_fight_followshipFRC19ai_fight_followship` | 0x003517e0 | CONFIRMED | Copies the member vector, then every field in order |
| 0x00442cd0 | `ai_fight_followship_copy_assign` | instance | `__as__19ai_fight_followshipFRC19ai_fight_followship` | 0x00351e70 | CONFIRMED | The same order, with assignment instead of construction; 6 callers, all manager paths or this class's helpers |

#### the unit's shared helpers

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443290 | `std_vector20_copy_assign` | a member vector | - | - | - | assigns a stride-0x14 vector; a fold shared with `ai_group_event` |

### 3b. Accessors

No row.

### 3c. Behaviour

#### `ai_fight_manager`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043e610 | `ai_fight_manager_join_party` | `JoinParty__16ai_fight_managerFP8C_entityP7C_actor` | 0x00351460 | CONFIRMED | Enrols an entity in the party with its target, creating the party at `now + 500` if none matches; an existing party's `next_eval_ms` gets + 1000 |
| 0x0043e870 | `ai_fight_manager_leave_party` | `LeaveParty__16ai_fight_managerFP8C_entityP7C_actor` | 0x00351b50 | CONFIRMED | Finds the member (in every party, or only those with the given target), erases it; a car party with no member left is disbanded, an emptied on-foot party is spliced out, otherwise a new commander is elected |
| 0x0043f090 | `ai_fight_manager_new_car_party` | `NewCarParty__16ai_fight_managerFP5C_carP7C_actor` | 0x003525e0 | CONFIRMED | `operator_new(0x48)`; sets `next_eval_ms` = now, `kind` = 1, `target`, `car`, and `0x29a` into `car+0x2150`, which keeps the car out of the car search |
| 0x0043f730 | `ai_fight_manager_disband_followship` | `DisbandCarParty__16ai_fight_managerFP19ai_fight_followshipb` | 0x00352cd0 | CONFIRMED | Snapshots the members, unlocks each and clears its actions, splices the record out, rejoins each member through `ai_fight_manager_join_party`; a non-zero flag also calls `ai_fight_manager_adjust_form_time` |
| 0x0043f950 | `ai_fight_manager_adjust_form_time` | `AdjustFormTime__16ai_fight_managerFP7C_actor` | 0x00352ee0 | CONFIRMED | Adds 10000 to `next_eval_ms` of every on-foot party with the given target |
| 0x0043f9b0 | `ai_fight_manager_get_party` | `GetParty__16ai_fight_managerFP8C_entityP7C_actorPUi` | 0x00352f60 | CONFIRMED | Finds the party of an entity (in every party, or only those with the given target) and writes the member index to an out-parameter |
| 0x0043fac0 | `ai_fight_manager_clear_all_parties` | `Init__16ai_fight_managerFv` | 0x00353070 | CONFIRMED | Teardown to empty, without freeing the buffer; `game_init` |
| 0x0043fb90 | `ai_fight_manager_destroy` | `Destroy__16ai_fight_managerFv` | 0x003530f0 | CONFIRMED | A reverse pass emptying each party's member vector, then the same teardown; `game_done` |
| 0x0043fce0 | `ai_fight_manager_tick` | `Tick__16ai_fight_managerFv` | 0x003531e0 | CONFIRMED | The per-frame party engine (§4); `game_tick` |
| 0x00440e10 | `ai_fight_manager_invalidate_actor` | `Invalidate__16ai_fight_managerFP7C_actor` | 0x003549f0 | CONFIRMED | Scrubs a dead or departing actor (§6); `entity_update`, `game_invalidate_actor` |

#### `ai_fight_followship`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043ebc0 | `ai_fight_followship_set_commander` | `SetCommander__19ai_fight_followshipFv` | 0x00351f40 | CONFIRMED | Elects `commander` (§1a) and rewrites each member's `group_state` |
| 0x0043ecc0 | `ai_fight_followship_count_free_seats` | `CountFreeSeats__19ai_fight_followshipFP5C_car` | 0x00352090 | CONFIRMED | Returns the car's seat count when the party can use all of it: 0 when a seat's occupant is not a member, a seat's attach position fails, or the line from it hits mask `0x140502` |
| 0x0043ee60 | `ai_fight_followship_take_car` | `TakeCar__19ai_fight_followshipFP5C_car` | 0x00352270 | CONFIRMED | Commandeers a car: picks up to the seat count of members with no vehicle or this car, type group not 2, in the commander's group, commander first; builds the new party, adds them, marks them 2 and erases them here |
| 0x0043f110 | `ai_fight_followship_add_car_member` | `AddCarMember__19ai_fight_followshipFP8C_entity` | 0x00352680 | CONFIRMED | Appends a member with an order code from its vehicle: 8 when it owns the matching seat of this car; 6, after `human_use_actor(car, 2)`, when in this car otherwise or in another car below 0.5; else 0 |
| 0x0043f3f0 | `ai_fight_followship_compute_position` | `ComputePosition__19ai_fight_followshipFv` | 0x003527f0 | CONFIRMED | Two passes: the mean member position into `centre`, the largest squared distance to it into `spread_sq` |
| 0x0043f590 | `ai_fight_followship_must_car_hunt` | `mustCarHunt__19ai_fight_followshipFv` | 0x003529b0 | CONFIRMED | Does the party need a vehicle to reach its target (§5) |
| 0x004410d0 | `ai_fight_followship_vector_insert_at` | - | - | - | `std::vector<ai_fight_followship>::insert` |
| 0x004412a0 | `ai_fight_followship_vector_count` | - | - | - | the party vector's count |
| 0x004412d0 | `ai_fight_followship_destroy_range` | - | - | - | disposes a range of parties |
| 0x00441c50 | `ai_fight_followship_vector_insert_n` | - | - | - | `insert_n` on the party vector |
| 0x00441fc0 | `ai_fight_followship_uninit_copy_range` | - | - | - | copy-constructs a range of parties |
| 0x00442000 | `ai_fight_followship_uninit_fill_n` | - | - | - | copy-constructs n parties |
| 0x00442030 | `ai_fight_followship_assign_range` | - | - | - | assigns a range of parties |
| 0x00442280 | `ai_fight_followship_assign_range_backward` | - | - | - | assigns a range of parties from the end |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00441070 | `vec_count_stride_0x14` | - | - | - | stride-0x14 count; `ai_fight_followship_add_car_member` and the assign helpers call it |
| 0x00441950 | `std_vector20_insert` | - | - | - | stride-0x14 insert; `ai_fight_manager_join_party` |
| 0x00441f90 | `fill_n_stride_0x14` | - | - | - | stride-0x14 fill; `ai_fight_followship_add_car_member` |
| 0x004427d0 | `copy_element_stride_0x14` | - | - | - | stride-0x14 element copy; `ai_fight_followship_add_car_member` and the assign helpers |

## 4. `ai_fight_manager_tick`

The per-frame engine walks the parties from the last to the first. A party with no member is spliced out. Otherwise, once the clock passes
`next_eval_ms`, it sets `now + 2000`, runs `ai_fight_followship_compute_position` and evaluates by `kind`; a car party also runs its member
machine (§4b) on the frames between evaluations.

### 4a. The evaluation

**On foot (`kind` 0).** When the target is a human in a car more than 20 units from `centre` (400 squared) and the car moves faster than 3.0, the
party looks for a car through the commander's group: `ai_group_get_closest_usable_car`, then `ai_fight_followship_take_car`. The new car party
gets `car_mode` 1 and is inserted into the vector.

The tick also holds a `car_mode` 2 arm that copies `stop_position`, taken when the search's third argument comes back non-zero. It cannot run: `ai_group_get_closest_usable_car` pops three arguments (`ret 0xc`) and its machine code reads only the first; the second argument's stack slot serves as a scratch float and the other argument is never touched, so the tick's flag stays 0.

**Car party (`kind` 1).** It posts an `ai_blob` with the target's position and forward vector to every member as event `0xb`
(`structs/ai_blob_log.md` §4), and counts the members aboard. `stuck_ms` becomes 4000 unless all are aboard and the car is below 2.0, in which
case it grows by 2000; at 10000 the car's task status is set to 2 and `stuck_ms` to 1000000. Then:

| Situation | Effect |
|---|---|
| all aboard, target a human on foot | within 25 units (625 squared) the car's task is removed; once the car is below 0.5 all members get code 6 |
| all aboard, otherwise, driver seat empty or its occupant alive | `stuck_ms` at 1000000: `car+0x2150` = `0x534`, the task is removed, all get code 6. Else seat 0 gets code 7, the rest 8, the car is dispatched at the target (`pursuit_event_dispatch_pursuit_vehicle`, with the group's `ai_group_get_car_factor`), and the task status decides: 1 and 3 remove the task and give code 6; 2 and 4 also write `0x534`, and 2 sets `stuck_ms` to 1000000 |
| all aboard, driver dead | the task is removed; once the car is below 0.5 all get code 6 and `car+0x2150` = 0 |
| not all aboard, no car hunt needed or given up | the task is removed; below 0.5 members in a vehicle get code 6, on-foot members not at code 5 get code 2, and with no member at code 5 the party is disbanded. This arm's `ret` at `0x00440dc4` ends the walk over the parties for that frame; every other arm, and this arm with the car still moving, jumps to the loop's continuation at `0x00440b1f` |
| not all aboard, otherwise, car at or below 0.5 | `stuck_ms` = 0; members aboard get code 8. With the commander aboard the rest get code 4. When 0.8 of the car's horizontal offset from `centre`, squared, exceeds `spread_sq` or 49, code 2 becomes 1 and code 3 becomes 4. Otherwise the rest get code 4 |

### 4b. The member machine and the order codes

On every frame each member of a car party is locked with `entity_lock_substate(member, 3, 8, code)`: first with 2, then with the value its case picks.
`entity_switch_ai_state` turns the lock into `entity_force_ai(3, 8, 0, code)`, which stores the code at `entity+0xf5c`, and `entity_ai`'s state-3 /
substate-8 branch switches on it. Below, *shot* means the member's log of the target (`ai_logs_get_log_for_actor`, `structs/ai_logs.md`) has `last_shot` under 2000 ms old,
and *seen* that it has `last_seen` under 3000 ms old.

| Code | What `entity_ai` makes the member do | How the tick moves it on |
|---|---|---|
| 0 | start state; the lock gives it code 2 | with `car_mode` set: the first member to 9 (or to 2 when *shot* and the party is under 3); the others to 1 (mode 1) or 2 (mode 2) |
| 1 | walk to the party car (`ai_action_new_move_to_car`) | *shot* and *seen*: to 3 within 5 units of the car, else 2. The commander to 4; within 5 units to 3 |
| 2 | fight its current enemy (`ai_fight_modes_tick`) | stays |
| 3 | crouch-fight (`ai_action_new_crouch_fight`) | stays |
| 4 | walk to the party car | *shot* or mode 2: to 3. No seat position: to 3. Beyond 100 units of the player and within 20 of a free seat: `human_put_in_vehicle_at_seat`, to 8. Within 0.4 horizontally: to 5 with a free seat, else 11 |
| 5 | get into the party car (`human_use_actor(car, 1)`) | aboard: to 8, and the first member starts the engine. In another vehicle: to 6. Over 1 unit from the seat: to 4 |
| 6 | leave its car once the car is nearly stopped (`human_use_actor(car, 2)`) | on foot: to 2 |
| 7 | breathe (idle) | stays |
| 8 | fire at its target (`ai_action_new_fire`) | stays |
| 9 | stop the car (`ai_action_new_car_stop`, at `stop_position`) | *shot*, *seen* and first: to 3. Mode 1: to 4 |
| 10 | aim from the car (`ai_action_new_car_aim`, at `stop_position`) | *shot*, *seen* and first: to 3. No writer in the tick |
| 11 | outside `entity_ai`'s switch | stays |

No code outside the manager reads a member record: `entity_ai`, `entity_update` and `program_process_command` call `ai_fight_manager_get_party` and read the party's
`kind`, `car` and `stop_position` and the member index. The stride `0x14` is shared with `ai_group_event`, which is why the stride-0x14 helpers are COMDAT-folded.

## 5. Why `ai_fight_followship_must_car_hunt` is a hunting decision

The test runs only for a target of kind 2 or `0x1b`:

| The target is… | Returns 1 when |
|---|---|
| on foot | the x/z squared distance from `centre` exceeds **10000.0** |
| in a kind-8 object | the 3D squared distance exceeds **100.0** |
| in a car (kind 4) | the car's speed exceeds **2.0** and the target is beyond `spread_sq`; otherwise the 10000.0 test applies |

The car branch returns 1 for a target that is merely moving at moderate distance. The question is whether the party can still catch the target on foot.

## 6. What `ai_fight_manager_invalidate_actor` does

It walks the parties backwards and dispatches three ways per record:

| The actor is… | Action |
|---|---|
| the party's `target` (`+0x40`) | when `kind` is 1, remove the car from the pursuit-event manager and disband; otherwise splice the record out |
| the party's `car` (`+0x44`) | disband the party |
| anything else | erase its member records; an emptied party is removed as in the first row, a party with members left elects a new commander |

`ai_groups_invalidate_actor` is the parallel scrub in the sibling class (`structs/ai_group.md` §3c).

## 7. PC vs PS2

- **Party record stride: PC `0x48`, PS2 `0x60`** (PS2 `NewCarParty` allocates `__nw__(0x60)`, `JoinParty` issues `memset(…, 0x60)`, and the walk divisors agree). The field map is a
  flat **+4** delta, confirmed by three functions:

  | Field | PC | PS2 |
  |---|---|---|
  | members begin / end | `+0x04` / `+0x08` | count `+0x24`, data `+0x28` |
  | next_eval_ms | `+0x2c` | `+0x30` |
  | kind | `+0x30` | `+0x34` |
  | commander | `+0x3c` | `+0x40` |
  | target | `+0x40` | `+0x44` |
  | car | `+0x44` | `+0x48` |

  The member vector is the exception: PC begin / end pointers (MSVC `std::vector`), PS2 count + data (`std::__vector_imp`). Do not rebase a single offset.
  **Where the `0x18` bytes go:** +4 inside `centre` (a 16-byte PS2 `S_vector`; PS2 `NewCarParty` zeroes the three floats at `+0x10`), plus `0x14` bytes after the car pointer
  (PS2 `+0x4c..+0x5f`) that no PS2 party method reads or writes.
- **`ai_fight_member` is `0x14` bytes on both builds.** The type name comes from `erase__Q23std70__vector_imp<>F**P15ai_fight_member**Ui` in `Invalidate`.
- **Other field deltas:** party car speed `car+0x688`, PS2 `car+0x8f0`; the `0x29a` sentinel `car+0x2150`, PS2 `car+0x23ac`; the seat array `+0xd90` / `+0xd94` stride `0x24`, PS2
  `+0xfd0` / `+0xfd4` stride `0x30`; current vehicle `entity+0x98`, PS2 `+0x224`.
- **The `10000` ms extension is identical on both platforms**, as are the `kind == 1` car test and the three-way `Invalidate` dispatch.
- **`ComputePosition` uses VU ops on the PS2** (`_vadd` / `_vdiv` / `_vmulq`) where the PC uses x87. `AddCarMember` stores the same codes 0 / 6 / 8 on both.
- **PS2 calls `__as__19ai_fight_followship` to shift records down** when splicing a party out; MSVC inlines the shift. That is the whole of `ai_fight_manager_invalidate_actor`'s size difference.
- **The PS2 car search fills neither out-parameter either:** the 169-line body of `GetCarClosestAndUsable__8ai_group` uses neither its third nor its fourth argument register, so the `car_mode` 2 arm (§4a) is unreachable on both builds.
- **`ai_fight_manager_dtor` has no PS2 twin:** the PS2 inlines it into `__dt__10ai_managerFv`.
- **`RemoveCar` is overloaded on the PS2:** `RemoveCar__12C_sledovackaFP5C_car` is the match; `RemoveCar__13G_GarageClassFPc` takes a `char *`.
- **No PS2-only methods** in either class.
- **The AI root object.** PS2 `Init__10ai_managerFv` embeds every AI sub-manager at a set offset in one root at `0x509080`:

  | Sub-manager | PS2 offset | PS2 absolute | PC singleton |
  |---|---|---|---|
  | `ai_groups` | `+0xdc` | `0x50915c` | `0x006552dc` |
  | `ai_follow_manager` | `+0xfc` | `0x50917c` | `0x00655304` |
  | `ai_fight_manager` | `+0x10c` | `0x50918c` | `0x00655318` |
  | `ai_sight_manager` | `+0x118` | `0x509198` | `0x00655328` [M] ¹ |
  | `ai_panic_manager` | `+0x138` | `0x5091b8` | - |
  | `ai_framerater` | `+0x148` | `0x5091c8` | - |
  | `ai_spageman` ×5 | `+0x14c…+0x20c` | - | - |
  | `ai_unlock_manager` | `+0x23c` | `0x5092bc` | - ² |

  `-` = no PC singleton; the sub-manager is PS2-only unless noted. ¹ Two vectors, no code (`structs/ai_manager.md` §6b). ² No PC member: `structs/ai_manager.md` §7 maps it to the `ai_groups+0x00`
  control-lock scalar (`structs/ai_group.md` §1c). The PS2 spacing `root+0x10c` to `+0x118` is `0xc`, one `gif::vector`, which is how `ai_fight_manager` is known to be a single vector.
  **The inter-sibling deltas differ between builds** (PC `ai_groups` to `ai_follow` `0x28`, PS2 `0x20`), so a PC singleton cannot be derived from another by rebasing; each is byte-proven by a census of
  the ECX immediate at its callsites. The PC root ends at `+0x130`; `0x00655348` is the actor possible-collision table (`actor_posscoll_table_ctor`).

### 7a. Emission order

Merged PS2 order across both classes maps monotonically onto merged PC order, with no non-monotone pair:

```
  SetCommander  CountFreeSeats  TakeCar  NewCarParty  AddCarMember  ComputePosition
     0x43ebc0       0x43ecc0    0x43ee60   0x43f090      0x43f110       0x43f3f0
  mustCarHunt  DisbandCarParty  AdjustFormTime  GetParty  Init  Destroy  Tick  Invalidate
     0x43f590      0x43f730        0x43f950     0x43f9b0  0x43fac0 0x43fb90 0x43fce0 0x440e10
```

The four rows in the middle (`AdjustFormTime` … `Destroy`) sit in a forced 4-to-4 gap between `DisbandCarParty` and `Tick`; `Invalidate` falls out of the trailing gap. A gap is forced only when it is enumerated as all
function starts, against the PS2 entries capable of having a PC function: `__ct__` and `__as__`, out of line on the PC (§3a), count in that arithmetic.

### 7b. The twin evidence

| PC name | Decisive evidence |
|---|---|
| `ai_fight_manager_join_party` | the stride proven twice in one body (walk divisor, zero-fill); shared 1000 and 500; ratio 0.68 |
| `ai_fight_manager_leave_party` | same two search arms and erase-and-dispatch ladder; ratio 1.08 |
| `ai_fight_manager_new_car_party` | `operator_new(0x48)` against `__nw__(0x60)`, the same four field writes in the same order, the same `0x29a` into the car; pinned by `ai_fight_followship_take_car` |
| `ai_fight_manager_disband_followship` | `Invalidate` calls `DisbandCarParty` at the same three points; the `b` is the flag, 0 at all three |
| `ai_fight_manager_adjust_form_time` | shared 10000, the same two-condition guard; 96 / 128 |
| `ai_fight_manager_get_party` | identical dual-branch search and inner scan; 272 / 272 = 1.00 |
| `ai_fight_manager_clear_all_parties`, `ai_fight_manager_destroy` | near-identical on both builds: only `Destroy` opens with the reverse pass, and both builds emit `Init` first; 208 / 120 and 336 / 236 |
| `ai_fight_manager_invalidate_actor` | pins three twins by position: `pursuit_event_remove_vehicle`, `ai_fight_manager_disband_followship`, `ai_fight_followship_set_commander` |
| `ai_fight_followship_count_free_seats` | same seat array, same `car_get_owner` per seat, same early `return 0` |
| `ai_fight_followship_take_car` | calls `count_free_seats` first and `new_car_party` last on both; the same 8-slot local index array and leader-first flag |
| `ai_fight_followship_add_car_member` | same vehicle field, same compare against the car, same `kind == 4 && speed < 0.5` branch |
| `ai_fight_followship_must_car_hunt` | shared 100.0, 10000.0, 2.0, kind codes 4 and 8, gate `kind == 2 \|\| 0x1b` |
| `ai_fight_followship_copy_ctor`, `_copy_assign` | same field order; 288 / 236 = 1.22 and 640 / 200: GCC calls `do_assign<>` out of line where MSVC inlines the capacity branches |

## 8. Related

- `structs/ai_manager.md` — the root object.
- `structs/ai_group.md` — the sibling subsystem; §6 hosts the shared translation unit.
- `structs/ai_follow_manager.md` — the third subsystem in the unit; it reuses the car seat-array offsets.
- `structs/pursuit_event.md` — `pursuit_event`, whose `pursuit_event_remove_vehicle` this class calls.
- `structs/ai_fight_modes.md` — the per-character combat-mode scorer, a different class despite the shared stem.
- `structs/entity.md` §7 — `entity_ai`, the caller of `ai_fight_manager_join_party` and `ai_fight_manager_leave_party`.

## 9. Open items

1. `ai_fight_member` `unread_08` and `unread_10`: no reader is located in the tick, the record functions or the three outside callers of `ai_fight_manager_get_party`.
2. What code 10 is for: `entity_ai` has an arm for it, and no writer is located in the tick or `ai_fight_followship_add_car_member`.
