# `ai_follow_manager` / `ai_follow_group` — escort and follower behaviour

**Created** 2026-08-01 20:14 +0200 · **Modified** 2026-11-09 21:00 +0100

How an NPC follows, escorts or is escorted by another actor, and how a group of followers gets into their leader's car.
**`ai_follow_manager`** is a singleton holding the vector of follow groups: it enrols and removes followers, runs the per-frame state
machine, resolves car seats and serialises everything into the savegame. **`ai_follow_group`** is one group: a leader, a follower list, an
optional destination, an optional linked vehicle and a state.

```
  ai_follow_manager    vector<ai_follow_group>, stride 0x38     §1c
        |  owns N of
        v
  ai_follow_group      leader + followers + destination + state §1a
        |  holds a vector of
        v
  C_actor *            stride 4                                 §1b
```

| | |
|---|---|
| PC prefix | `ai_follow_manager_`, `ai_follow_group_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `ai_follow_manager` `0x14`, a member of the root at `ai_manager+0xec` (`0x00655304`); `ai_follow_group` `0x38` |
| PC constructor | `ai_follow_manager` -, built by `ai_manager_ctor`; `ai_follow_group_copy_ctor` |
| PC translation unit | `[0x0043b000, 0x00443510)`, 84 starts, shared with `ai_group` and `ai_fight_manager`, hosted by `structs/ai_group.md` §6; this class's band is `[0x0043cdf0, 0x0043e610)`, and `ai_follow_manager_dtor` sits in the `ai_manager` unit |
| PS2 class | `ai_follow_manager` (tag `17ai_follow_manager`), 17 methods; `ai_follow_group` (tag `15ai_follow_group`), 1 method |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `ai_follow_group` — one group (`0x38`)

```c
typedef struct ai_follow_group {
    /* +0x00 */ allocator_t followers_alloc; // [H] the follower vector's empty allocator: the vector object spans +0x00..+0x0f
    /* +0x04 */ C_actor **followers_begin;   // [H] the followers, stride 4
    /* +0x08 */ C_actor **followers_end;     // [H] count = (end - begin) >> 2
    /* +0x0c */ C_actor **followers_cap;     // [H] one past the last allocated follower slot
    /* +0x10 */ S_vector  destination;       // [H] where the group's car is sent; meaningful only when flags bit 0x01 is set
    /* +0x1c */ uint32_t  unread_1c;         // [U] no code path sets it; copied, saved, loaded; no reader (class census)
    /* +0x20 */ uint32_t  flags;             // [H] 0x01 destination set, 0x10 dispatched task completed, 0x20 followers leave this car
    /* +0x24 */ uint32_t  state;             // [H] 0..5, the state machine of §4
    /* +0x28 */ uint32_t  retry_deadline;    // [H] mission clock + 7000 ms, set by event 4; state 4 re-dispatches past it
    /* +0x2c */ uint32_t  task_deadline;     // [H] mission clock + 6000 ms, armed on dispatch; no reader but the savegame
    /* +0x30 */ C_actor  *leader;            // [H] the group key: every lookup matches on it
    /* +0x34 */ C_car    *linked_car;        // [H] the car whose dispatched task failed (bit 0x20); cleared when the leader boards another car
} ai_follow_group;
```

The `0x38` stride is proven by the allocation, `operator_new(0x38)`. The record's full field order is visible in one body,
`ai_follow_group_copy_ctor`. `+0x00` is the vector's allocator byte: both add functions copy an empty allocator into it (`mov [eax], cl` at
`0x0043cffd`), and every vector insert takes the record base as the vector (`lea ecx, [edi + eax*8]` at `0x0043cfd9`, `mov ecx, esi` at `0x0043d03e`).
The same shape sits at `ai_ghost+0x00` and `ai_loc+0x20`.

`+0x1c` is a member nothing sets: neither add function writes it and the record comes straight from `operator_new`, so a savegame carries whatever the
heap held there.

### 1b. The follower vector

Stride 4. The PC decomp shows only a vector of dwords at `group+0x04`; the element type `C_actor *` comes from the PS2 template instantiation (§5).

### 1c. `ai_follow_manager` — the group vector (`0x14`)

A singleton, a member of the `ai_manager` root at `+0xec` = `0x00655304`: the unwind funclet `add ecx, 0xec; jmp ai_follow_manager_dtor` shows it
(`structs/ai_manager.md`). All 34 callsites in `.text` that load `0x00655304` into ECX resolve to the 15 functions of §3c.

```c
typedef struct ai_follow_manager {
    /* +0x00 */ allocator_t      groups_alloc; // [H] the group vector's empty allocator, padded to 4
    /* +0x04 */ ai_follow_group *groups_begin; // [H] the groups, stride 0x38
    /* +0x08 */ ai_follow_group *groups_end;   // [H] count = (end - begin) / 0x38
    /* +0x0c */ ai_follow_group *groups_cap;   // [H] one past the last allocated group slot
    /* +0x10 */ uint32_t         heartbeat;    // [H] mission clock + 2000 ms; refreshed by the update, zeroed on clear, destroy and invalidate
} ai_follow_manager;
```

## 2. Field evidence

`ai_follow_group_copy_ctor`, `ai_follow_group_copy_assign` and `ai_follow_group_vector_insert_at` copy every field of a group, and the erase shifts inside
`ai_follow_manager_update` and `ai_follow_manager_del_follower` move whole records; the cells leave them out. The manager reaches a group through a walked pointer, so the cells of §2a come from the bodies.

### 2a. `ai_follow_group`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | followers_alloc | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination` | - | - |
| +0x4 | followers_begin | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_load` | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_dtor`, `ai_follow_manager_event`, `ai_follow_manager_get_car_seat_for_follower`, `ai_follow_manager_get_save_size`, `ai_follow_manager_group_fits_leader_vehicle`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | - |
| +0x8 | followers_end | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_dtor`, `ai_follow_manager_event`, `ai_follow_manager_get_car_seat_for_follower`, `ai_follow_manager_get_save_size`, `ai_follow_manager_group_fits_leader_vehicle`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | - |
| +0xc | followers_cap | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_dtor` | `ai_follow_group_copy_assign`, `std_vector4_copy_assign` | their capacity tests |
| +0x10 | destination | H | `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_load` | `ai_follow_manager_save`, `ai_follow_manager_update` | `ai_follow_manager_update` passes its address to the dispatch |
| +0x1c | unread_1c | U | `ai_follow_manager_load` | `ai_follow_manager_save` | no code path sets it: copied, saved and loaded only; no reader (class census), Open item 1 |
| +0x20 | flags | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | bit `0x10` has no reader: Open item 1 |
| +0x24 | state | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_event`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_event`, `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | - |
| +0x28 | retry_deadline | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_event`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_save`, `ai_follow_manager_update` | - |
| +0x2c | task_deadline | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_save` | Open item 1 |
| +0x30 | leader | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_load` | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_event`, `ai_follow_manager_get_car_seat_for_follower`, `ai_follow_manager_group_fits_leader_vehicle`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_update` | - |
| +0x34 | linked_car | H | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_update` | the load always stores 0 (§3c note) |

### 2b. `ai_follow_manager`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | groups_alloc | H | `ai_manager_ctor` | - | the byte at `ai_manager +0xec`; the `lea ecx, [ecx]` at `0x0043e129`, after the return of `ai_follow_manager_should_follower_leave_group`, is an alignment filler before its jump table |
| +0x4 | groups_begin | H | `ai_follow_group_vector_insert_at`, `ai_follow_manager_dtor` | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_clear_all_groups`, `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_dtor`, `ai_follow_manager_event`, `ai_follow_manager_get_car_seat_for_follower`, `ai_follow_manager_get_save_size`, `ai_follow_manager_group_fits_leader_vehicle`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | - |
| +0x8 | groups_end | H | `ai_follow_group_vector_insert_at`, `ai_follow_manager_clear_all_groups`, `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_dtor`, `ai_follow_manager_invalidate_actor` | `ai_follow_manager_add_follower`, `ai_follow_manager_add_follower_with_destination`, `ai_follow_manager_clear_all_groups`, `ai_follow_manager_del_follower`, `ai_follow_manager_destroy`, `ai_follow_manager_dtor`, `ai_follow_manager_event`, `ai_follow_manager_get_car_seat_for_follower`, `ai_follow_manager_get_save_size`, `ai_follow_manager_group_fits_leader_vehicle`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_save`, `ai_follow_manager_should_follower_leave_group`, `ai_follow_manager_update` | - |
| +0xc | groups_cap | H | `ai_follow_group_vector_insert_at`, `ai_follow_manager_dtor` | `ai_follow_group_vector_insert_at` | - |
| +0x10 | heartbeat | H | `ai_follow_manager_clear_all_groups`, `ai_follow_manager_destroy`, `ai_follow_manager_invalidate_actor`, `ai_follow_manager_load`, `ai_follow_manager_update` | `ai_follow_manager_save`, `ai_follow_manager_update` | - |

## 3. Method family

16 of 18 PS2 methods have a PC function or a folded slot; the other two are the factoring of §5b.

### 3a. Construction & destruction

#### `ai_follow_manager`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443a20 | `ai_follow_manager_dtor` | instance | - | - | - | Walks the groups, drains and frees each follower vector, frees the buffer; one entry point, the `ai_manager` unwind funclet. No twin: inlined into the PS2 `ai_manager` destructor |

#### `ai_follow_group`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004426f0 | `ai_follow_group_copy_ctor` | instance | `__ct__15ai_follow_groupFRC15ai_follow_group` | 0x0034ec30 | CONFIRMED | Copies the follower vector and every field; shows the record's field order |
| 0x00442b30 | `ai_follow_group_copy_assign` | instance | - | - | - | `operator=` |

#### the unit's shared helpers

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00442f70 | `std_vector4_copy_assign` | a follower vector | - | - | - | assigns a stride-4 vector; a fold shared with `ai_group` |

### 3b. Accessors

No row.

### 3c. Behaviour

#### `ai_follow_manager`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043cdf0 | `ai_follow_manager_del_follower` | `DelFollower__17ai_follow_managerFP7C_actor` | 0x0034e440 | CONFIRMED | Removes an actor from every group, walking both levels backwards; a group whose follower vector empties is spliced out |
| 0x0043cf80 | `ai_follow_manager_add_follower` | `AddFollower__17ai_follow_managerFP7C_actorP7C_actor` | 0x0034e830 | CONFIRMED | `ai_follow_manager_del_follower` first, then find-or-create by `leader`; a new group starts in state 5 |
| 0x0043d090 | `ai_follow_manager_add_follower_with_destination` | `AddFollower__17ai_follow_managerFP7C_actorP7C_actorR8S_vector` | 0x0034f370 | CONFIRMED | The destination overload: stores the vector at `+0x10`, sets bit `0x01`; a new group starts in state 0 when leader and follower share a vehicle, else 5 |
| 0x0043d200 | `ai_follow_manager_update` | `Update__17ai_follow_managerFv` | 0x0034f7d0 | CONFIRMED | The per-frame engine: 2000 ms heartbeat, 6-case switch (§4); `game_tick`, and a tail call from `ai_follow_manager_invalidate_actor` |
| 0x0043da00 | `ai_follow_manager_clear_all_groups` | `Init__17ai_follow_managerFv` | 0x00350110 | CONFIRMED | Clears to empty without freeing; `Init`-is-a-teardown (`structs/ai_fight_manager.md` §3c) |
| 0x0043da90 | `ai_follow_manager_destroy` | `Destroy__17ai_follow_managerFv` | 0x00350190 | CONFIRMED | A leading reverse pass emptying each follower vector, then the same teardown |
| 0x0043db90 | `ai_follow_manager_group_fits_leader_vehicle` | `CanFolloshipFollow__17ai_follow_managerFP7C_actor` | 0x003502b0 | CONFIRMED | True when the group fits the leader's vehicle, by the car's seat array `+0xd90` / `+0xd94`, stride `0x24` |
| 0x0043dcd0 | `ai_follow_manager_get_car_seat_for_follower` | `GetCarSeat__17ai_follow_managerFP7C_actorP7C_actor` | 0x00350470 | CONFIRMED | Counts the not-yet-aboard predecessors of a follower, then returns the Nth free seat |
| 0x0043de20 | `ai_follow_manager_invalidate_actor` | `Invalidate__17ai_follow_managerFP7C_actor` | 0x003506b0 | CONFIRMED | Three-way scrub on `leader`, `linked_car` and the followers, zeroes the heartbeat, tail-calls the update |
| 0x0043df40 | `ai_follow_manager_event` | `Event__17ai_follow_managerFUiP7C_actor` | 0x00350870 | CONFIRMED | The group-event dispatcher, events 2, 3, 4 (§4b) |
| 0x0043e0a0 | `stub_retfalse_args4_0043e0a0` | `mustEnter__17ai_follow_managerFP7C_actor` | 0x00350a60 | STRUCTURAL | A COMDAT-shared `return 0` that serves as the method at its one callsite in `entity_action_tick` (§5c) |
| 0x0043e0b0 | `ai_follow_manager_should_follower_leave_group` | `mustLeave__17ai_follow_managerFP7C_actor` | 0x00350a70 | CONFIRMED | State 1 answers yes; states 0, 2, 3, 4, 5 answer the `flags & 0x20` test |
| 0x0043e150 | `ai_follow_manager_get_save_size` | `GetSize__17ai_follow_managerFv` | 0x00350c60 | CONFIRMED | `8 + sum(4 + (followers + 3) * 0x20)`: the name slots of `+3` hold the leader and the linked car, and the third pays for the destination and the five dwords |
| 0x0043e1c0 | `ai_follow_manager_save` | `Save__17ai_follow_managerFPPv` | 0x00350ce0 | CONFIRMED | Heartbeat, group count, then per group the follower names, destination, `+0x1c`, `+0x20`, `+0x24`, `+0x28`, `+0x2c`, leader name, car name or `none` |
| 0x0043e3d0 | `ai_follow_manager_load` | `Load__17ai_follow_managerFPPv` | 0x00350f40 | CONFIRMED | `ai_follow_manager_destroy`, then the same order; followers must be kind `0x1b`, leaders 2 or `0x1b`. The linked car is never restored ¹ |

#### `ai_follow_group`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00441620 | `ai_follow_group_vector_insert_at` | - | - | - | `std::vector<ai_follow_group>::insert`; 3 callers, all `ai_follow_manager_*` |

¹ The car's name is read (`0x0043e585`) but never looked up: the kind-4 test (`0x0043e596`) runs on the leader pointer from the previous lookup, so `+0x34` always loads as 0 while `+0x20`
bit `0x20` is restored. The only clear of that bit on boarding a different car tests `+0x34 != 0`, so after a load it cannot fire and `ai_follow_manager_should_follower_leave_group` keeps answering
yes for that group until another path clears the bit (§4).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004426c0 | `stl_copy_dwords` | - | - | - | copies a dword range; `ai_follow_group_copy_assign` |

## 4. What `ai_follow_manager_update` actually does

The per-frame engine. Its four worker calls are all `pursuit_event_*`.

Each group runs a six-state machine on `+0x24`:

| state | meaning | transitions [H] |
|---|---|---|
| 0 | the leader is in a vehicle, followers boarding | leader on foot → 1 · all followers in his vehicle, a destination set (bit 1), the vehicle a car and its owner a `C_entity` → dispatch → 2 · all aboard otherwise → 3 |
| 1 | the leader is on foot | he gets a vehicle → 0 (a car other than `+0x34` clears `+0x34` and bit `0x20`) · every follower on foot too → 5 |
| 2 | a vehicle task is running | status 1 → bit `0x10`, task removed, → 5 · status 2..4 → task removed, bit `0x20`, `+0x34` = the car, → 1 · a follower off → 0 · vehicle not a car → 3 · leader on foot → 1 |
| 3 | all aboard, nothing dispatched | a follower off → 0 · leader on foot → 1 · also entered by event 3 from states 2 and 4 |
| 4 | the task is held until `+0x28` | only entered by event 4 from state 2. Past the deadline → re-dispatch, → 2 · before it the task is removed each tick; a follower off → 0 · leader on foot → 1 · vehicle not a car → 3 · event 4 again → 1 |
| 5 | settled | a follower's vehicle differs from the leader's → 0, or 1 if the leader is on foot |

The add overloads create a group in state 5, or in state 0 when the destination overload finds
the leader and the follower in the same vehicle. On each 2000 ms heartbeat the update
also drops `C_entity` followers that have no `ai_action` of type 1 or have one of type 6, and
erases a group whose follower vector empties.

On dispatch it arms `+0x2c` with `now + 6000` and sends the car to the destination at `+0x10`:
**`pursuit_event_dispatch_pursuit_vehicle`** if any follower is in danger (`entity_is_in_danger`),
otherwise **`pursuit_event_dispatch_move_to_vehicle`**. State 2 polls
**`pursuit_event_record_get_action_status_by_vehicle`** and tears down with
**`pursuit_event_remove_vehicle`**. Bit `0x20` is what
`ai_follow_manager_should_follower_leave_group` tests: once a task has failed, the followers are
told to leave that car.

The destination comes only from `ai_follow_manager_add_follower_with_destination`, whose single
caller is `entity_process_command` case `0x1012` (`enemy_action_follow`) with its `TARGET <frame>`
keyword: the destination is that frame's world position. No shipped script uses `TARGET` (its
one occurrence, in MISE04-MESTO `Paulie`, is commented out). Bit 1 has no other writer, so in
shipped play no group carries a destination: the dispatch arm of state 0 is not reached, and
states 2 and 4 with it.

**None of that is police.** The `pursuit_event_*` family is the `pursuit_event` class, the **generic
vehicle-task dispatcher** — the same one the police use, and named for that dominant consumer.
A group uses it to have an NPC-driven car carry the leader and his followers to the group's
destination (the four calls' PS2 twins: §5).

### 4a. `ai_follow_manager_should_follower_leave_group`

Bit `0x20` is what it tests: once a task has failed, the followers are told to leave that car. `entity_action_tick` calls it twice.

### 4b. The event ids, and the two that do nothing

`ai_follow_manager_event` is a chain `if (event == 2) … else if (== 3) … else if (== 4) …` with no trailing else, so any other id is a no-op. The five live callsites pass:

| Event | Passed by | Effect |
|---|---|---|
| 0 | `entity_action_tick` | none: no arm matches |
| 1 | `entity_action_tick` | none: no arm matches |
| 2 | `entity_death` | forwards to `ai_follow_manager_del_follower(target)` |
| 3 | `entity_force_script` | keyed on a follower: the first group in state 2 or 4 holding it, whose leader sits in a car, has its vehicle task removed and moves to state 3; the follower stays |
| 4 | `player_do_action_button_in_car` | keyed on the leader: state 2 to 4 with `+0x28` = now + 7000 ms, state 4 to 1 with `+0x28` = 0. Sent only while the player's seat index `+0xac` is not 0, a passenger seat |

Two of the callsites are inert as the code stands, and event 4 needs the player riding as a passenger in a group whose car an NPC drives. By the user's in-game observation the shipped game never seats the
player as a passenger, so event 4 has no effect in shipped play; whether that was ever intended is not recoverable from the binary. State 4 is unreachable in shipped play for a second reason: no group ever gets a
destination (§4).

## 5. PC vs PS2

- **Group record stride: PC `0x38`, PS2 `0x40`**, from the allocation pair `operator_new(0x38)` and `__nw__(0x40)`. The divergence lies entirely inside the follower vector: `+0x30` and `+0x34` are at identical offsets on both
  builds. Do not rebase the record wholesale.
- **Manager fields carry a flat +4 delta:** PC `+0x04` / `+0x08` begin / end and `+0x10` heartbeat, PS2 `+0x00` / `+0x04` and `+0x0c`.
- **The follower vector differs:** PC begin / end / cap pointers, PS2 a `gif::vector` at `group+0x10`. Expect this shape difference in every method.
- **Element types come from PS2 templates:** `gif::vector<15ai_follow_group, 1>` types the manager's vector and `gif::vector<P7C_actor, 8>` the follower vector, a `C_actor *` vector with an initial reserve of 8.
- **Serialisation differs in one respect:** the PC writes each entity reference as a `0x20`-byte frame-name string resolved by `mission_find_actor_by_name`; the PS2 writes a 4-byte GUID resolved by `GetActorFromGUID__6C_gameFUi`.
  That explains every differing constant of the `GetSize` pair (PC `4 + (n+3) * 0x20`, PS2 `n * 4 + 0x2c`) and the 0.44 size ratio on `Load`. It also holds for `ai_groups` (`structs/ai_group.md` §7).
- **`Load` restores the linked car on the PS2 only:** it resolves it with a third `GetActorFromGUID__6C_gameFUi` before its kind-4 test. A PC-only defect (§3c note).
- **The record's `+0x1c` is unset on the PS2 too:** neither `AddFollower` writes `puVar5[7]`, while `Save` writes `puVar4 + 7` and the copy constructor copies it.
- **The event arms and the state machine are the same on both builds.**
- **The four `pursuit_event_*` calls of the update map 1:1 by body-diff:** `pursuit_event_dispatch_pursuit_vehicle` to `AddCarHunt`, `pursuit_event_dispatch_move_to_vehicle` to `AddCarMoveTo`,
  `pursuit_event_record_get_action_status_by_vehicle` to `ActionStatus`, `pursuit_event_remove_vehicle` to `RemoveCar`, all `__12C_sledovacka`.
- **The mission clock:** PC `*(g_act_mission+0x24)+0x2b0c`, PS2 `*(act_mission+0x20)+0xe54`.
- **The 2000 ms heartbeat and the 6000 ms task deadline are identical**, as are the state codes, the flag bits and the `mustLeave` truth table.
- **`ai_follow_manager` is a singleton on the PS2 at `0x50917c`**, a sub-object of the AI root (`structs/ai_fight_manager.md` §7). The census of §1c corroborates the folded `mustEnter` stub (`0x0043e0a0` is in it) and makes §5b a
  measurement: there is no further PC method to find. `0x0043e610` and `0x0043e870` are absent from it: they are `ai_fight_manager`'s `JoinParty` and `LeaveParty`.
- **`ai_follow_group::__ct__` is out of line on the PC**, in the container-helper cluster at `0x0044xxxx`; `ai_follow_group_copy_assign` is inlined by GCC on the PS2.
- **The twin evidence, per method:** `AddFollower`: `operator_new(0x38)` against `__nw__(0x40)`, five identical field writes in identical order with the shared 5 into `+0x24`. `AddFollower` with vector: the same state selector, `leader+0x98` against `follower+0x98` (PS2 `+0x224`).
  `Update`: statement-for-statement, same 2000, 6000, 6-case switch, flag bits, `& 0xffffffdf`, `1.0f`. `Destroy`: split from `Init` by the extra reverse pass, on both builds, with `Init` emitted first. `CanFolloshipFollow`: seat array PC `+0xd90` / `+0xd94` stride `0x24`, PS2 `+0xfd0` / `+0xfd4` stride `0x30`, proven by `ai_fight_followship_count_free_seats`.
  `GetCarSeat`: the field pair PC `+0x98` / `+0x9c`, PS2 `+0x224` / `+0x228`. `Invalidate`: tail-calls `Update` on both, the pin that identified `ai_follow_manager_update`. `Save`: five consecutive identical field offsets in identical order. `ai_follow_group_copy_ctor`: 224 / 204 = 1.10.

### 5a. Emission order: one translation unit, three subsystems

The merged PS2 order across `ai_group`, `ai_groups`, `ai_follow_manager` and `ai_follow_group` maps monotonically onto merged PC order, with no non-monotone pair:

```
  ai_group      Add Del GetSniff UnderAttack ProcessEvents GetCarFactor GetCarClosestAndUsable
  ai_groups     NewGroup                                  ai_group  __ct__  __dt__
  ai_groups     GetGroup Init Destroy Tick GetSize Save Load InvalidateActor
  ai_group      canReceive canCast
  ai_groups     hasCarGroup MemWrite MemRead
  ai_follow_*   DelFollower AddFollower [group __ct__] AddFollower(vec) Update Init Destroy
                CanFolloshipFollow GetCarSeat Invalidate Event mustEnter mustLeave
                MemWrite MemRead GetSize Save Load
```

The class range begins only after `ai_group_can_receive` and `ai_group_can_cast`; the merged chain has 13 anchors.

### 5b. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00350bc0 | `MemWrite__17ai_follow_managerFPPvPvUiUi` | factoring: the PS2 wraps the savegame stream primitive in a 72-byte per-class method; the PC calls `entity_mem_write` at every site |
| 0x00350c10 | `MemRead__17ai_follow_managerFPPvPvUiUi` | the same, against `entity_mem_read` |

### 5c. The `mustEnter` callsite

PS2 `AI__8C_entity` runs four consecutive follow arms; PC `entity_action_tick` runs the same four
in the same order:

| PS2 | PC |
|---|---|
| `mustLeave == 0 && CanFolloshipFollow != 0` → `NewFollowLink` | the arm above |
| `vehicle != 0 && mustLeave != 0` → clear bit 0, unlink | `actor_get_vehicle() != 0 && ai_follow_manager_should_follower_leave_group() != 0` → same |
| `vehicle != 0 && other != vehicle && **mustEnter == 0**` → same | `actor_get_vehicle() != 0 && iVar18 != iVar20 && **stub_retfalse_args4_0043e0a0() == 0**` → same |
| `vehicle != 0 && other == vehicle` → clear bit 0 | the same-vehicle arm below |

PS2's `param_1[0x89]` is offset `0x224` — the current-vehicle field whose PC twin `+0x98` is
proven by `ai_follow_manager_add_follower_with_destination` — so PS2's `fVar28 != param_1[0x89]` **is** the PC `iVar18 != iVar20` test.

Both `mustEnter` bodies are `return 0`, so nothing separates the symbols, and MSVC folded the PC one onto `stub_retfalse_args4_0043e0a0` with every identical `return 0`. The mapping is true at this callsite and false for the symbol, so it is recorded as an inline comment at the callsite and a note
on the stub's docblock, and the symbol keeps its stub name.

## 6. Related

- `structs/ai_manager.md` — the root object that holds `ai_follow_manager` at `+0xec`.
- `structs/ai_group.md` — `ai_group` / `ai_groups`, the neighbours in this unit; §6 hosts it.
- `structs/ai_fight_manager.md` — the third subsystem in the unit.
- `structs/pursuit_event.md` — `pursuit_event`, the generic vehicle-task dispatcher the update drives.
- `structs/entity.md` §7 — `entity_action_tick`, which calls `ai_follow_manager_should_follower_leave_group` and the folded `mustEnter` stub.

## 7. Open items

1. Fields with no consumer: `unread_1c`, `task_deadline` and `flags` bit `0x10` are written and saved, and no reader is located.
