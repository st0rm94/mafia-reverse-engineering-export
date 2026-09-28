# `ai_follow_manager` / `ai_follow_group` — escort and follower behaviour

**Created** 2026-08-01 20:14 +0200 · **Modified** 2026-09-21 23:38 +0200

Cross-build material is in §5; the method tables in §2 are PC↔PS2 twin tables.

This is **how an NPC follows, escorts, or is escorted by another actor — and how a group of
followers gets into their leader's car.** Two classes, one subsystem:

- **`ai_follow_manager`** (17 PS2 methods) — a singleton holding the vector of follow groups.
  Enrols and removes followers, runs the per-frame state machine, resolves car seats, and
  serialises the whole thing into the savegame.
- **`ai_follow_group`** (1 PS2 method, a copy constructor) — ONE group: a leader, a follower
  list, an optional destination point, an optional linked vehicle, and a state.

```
  ai_follow_manager    vector<ai_follow_group>, stride 0x38     THIS FILE §1c
        |  owns N of
        v
  ai_follow_group      leader + followers + destination + state THIS FILE §1a
        |  holds a vector of
        v
  C_actor *            stride 4                                 THIS FILE §1b
```

- **PC prefixes:** `ai_follow_manager_*`, `ai_follow_group_*`.
- **PS2 method count:** 17 + 1 = 18. **PC mapped: 15** — `ai_follow_manager` **14/17** (the
  remaining 3 accounted for, §2c) and `ai_follow_group` **1/1** (§2b).
- **PC cluster:** `[0x0043cdf0, 0x0043e610)`, immediately above the `ai_fight_*` translation unit
  and immediately below `ai_group`/`ai_groups` — the three subsystems share one TU (§2d).
  `ai_follow_group`'s members and `ai_follow_manager_dtor` sit outside it, in the container-helper
  cluster at `0x0044xxxx`.

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. Layout

### 1a. `ai_follow_group` — the group record, 0x38 bytes (PC)

```c
typedef struct ai_follow_group {          // stride 0x38 on PC (0x40 on PS2)
    /* +0x00 */ uint8_t   _b0;            // [U] written from the low byte of the first follower
    /* +0x04 */ C_actor **followers_begin;// [H]
    /* +0x08 */ C_actor **followers_end;  // [H] count = (end - begin) >> 2
    /* +0x0c */ C_actor **followers_cap;  // [H]
    /* +0x10 */ float     destination[3]; // [H] an S_vector; only meaningful when flag 1 is set.
                                          //     Written by add_follower_with_destination, read by
                                          //     update when it dispatches the vehicle task
    /* +0x1c */ uint32_t  _u1c;           // [U] serialised, no reader located
    /* +0x20 */ uint32_t  flags;          // [H] bit 0x01 = a destination is set at +0x10
                                          //     bit 0x20 = the leader's car was handed to the
                                          //                vehicle-task dispatcher
                                          //     bit 0x10 = the dispatched task completed
    /* +0x24 */ uint32_t  state;          // [H] 0..5, the state machine in §3
    /* +0x28 */ uint32_t  retry_deadline; // [H] mission clock; state 4 re-dispatches past it
    /* +0x2c */ uint32_t  task_deadline;  // [H] mission clock + 6000 ms, armed on dispatch
    /* +0x30 */ C_actor  *leader;         // [H] the group key - every lookup matches on this
    /* +0x34 */ C_car    *linked_car;     // [H] the car the dispatcher was asked to bring
} ai_follow_group;                        // 0x38
```

**`+0x30` and `+0x34` sit at the SAME offsets on both builds** despite the 0x38-vs-0x40 stride
difference — the divergence is entirely inside the follower vector (§5). That coincidence is what
made `ai_follow_manager_invalidate_actor`'s three-way dispatch diff so clean.

`+0x24` and `+0x20` are proven together by `ai_follow_manager_should_follower_leave_group`, which
switches on the first and masks the second with **0x20**; `+0x2c`, `+0x28` and `+0x34` by
`ai_follow_manager_update`; `+0x10`/`+0x20 bit 1` by the
`ai_follow_manager_add_follower_with_destination` / `ai_follow_manager_update` writer/reader pair.
The **0x38 stride** is proven outright by the allocation itself — `operator_new(0x38)` against
PS2's `__nw__(0x40)`.
The record's full field order is also visible in one body, `ai_follow_group_copy_ctor` (§2b).

**`+0x00` is [U].** Both `add_follower` overloads write the low BYTE of the follower pointer
there and nothing reads it — most likely a compiler-materialised temporary rather than a field
(§7).

### 1b. The follower vector — `C_actor *`, stride 4

The PC decomp shows only "a vector of dwords at `group+0x04`"; the element type `C_actor *` comes
from the PS2 template instantiation in the same translation unit (§5), as does the outer
container's element type `ai_follow_group`.

### 1c. `ai_follow_manager` — the group vector

A singleton, and a MEMBER of the `ai_manager` root object at **`ai_manager+0xec`** =
**`0x00655304`** (byte-proven by the unwind funclet `add ecx, 0xec ; jmp ai_follow_manager_dtor`;
full picture in `structs/ai_manager.md`). Its entire state is one
`std::vector<ai_follow_group>` plus a heartbeat:

```c
    /* +0x04 */ ai_follow_group *groups_begin;  // [H]
    /* +0x08 */ ai_follow_group *groups_end;    // [H] count = (end - begin) / 0x38
    /* +0x0c */ ai_follow_group *groups_cap;    // [H]
    /* +0x10 */ uint32_t         heartbeat;     // [H] mission clock + 2000 ms. Refreshed by
                                                //     update; zeroed by clear_all_groups,
                                                //     destroy and invalidate_actor
```

---

## 2. Method families — PC ↔ PS2

### 2a. `ai_follow_manager` — 14 of 17

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / decisive evidence |
|---|---|---|---|---|---|
| 0x0043cdf0 | `ai_follow_manager_del_follower` | `DelFollower__17ai_follow_managerFP7C_actor` | **CONFIRMED** | yes | Reverse-outer / reverse-inner erase of an actor from every group, **cascading**: a group whose follower vector empties is spliced out entirely. PC inlines the record shift where PS2 calls `__as__Q23gif19vector<>` |
| 0x0043cf80 | `ai_follow_manager_add_follower` | `AddFollower__17ai_follow_managerFP7C_actorP7C_actor` | **CONFIRMED** | yes | `DelFollower` first, then find-or-create by `+0x30`. Decisive: **`operator_new(0x38)` ↔ `__nw__(0x60→0x40)`** and five identical field writes in identical order including the shared magic **5** into `+0x24` |
| 0x0043d090 | `ai_follow_manager_add_follower_with_destination` | `AddFollower__17ai_follow_managerFP7C_actorP7C_actorR8S_vector` | **CONFIRMED** | yes | The 3-arg overload: stores the caller's `S_vector` at `+0x10` and sets flag bit **1**. Same state selector on both builds — `leader+0x98` vs `follower+0x98` (PS2 `+0x224`) picks state 0 over 5 |
| 0x0043d200 | `ai_follow_manager_update` | `Update__17ai_follow_managerFv` | **CONFIRMED** | yes | Statement-for-statement: same **2000** ms heartbeat, same 6-case switch, same **6000** ms deadline, same flag bits 1/0x10/0x20 and `& 0xffffffdf`, same `1.0f`. §3 |
| 0x0043da00 | `ai_follow_manager_clear_all_groups` | `Init__17ai_follow_managerFv` | **CONFIRMED** | yes | Clears to empty without freeing — the family's `Init`-is-a-teardown idiom (`structs/ai_fight_manager.md` §2a) |
| 0x0043da90 | `ai_follow_manager_destroy` | `Destroy__17ai_follow_managerFv` | **CONFIRMED** | yes | Separated from `Init` by the **build-invariant** discriminator: an extra leading REVERSE pass emptying each record's follower vector, on both builds, with `Init` emitted first |
| 0x0043db90 | `ai_follow_manager_group_fits_leader_vehicle` | `CanFolloshipFollow__17ai_follow_managerFP7C_actor` | **CONFIRMED** | yes | Seat array PC `+0xd90`/`+0xd94` stride 0x24 ↔ PS2 `+0xfd0`/`+0xfd4` stride 0x30 — the pair already proven by `ai_fight_followship_count_free_seats`; returns true when the group fits |
| 0x0043dcd0 | `ai_follow_manager_get_car_seat_for_follower` | `GetCarSeat__17ai_follow_managerFP7C_actorP7C_actor` | **CONFIRMED** | yes | Counts not-yet-aboard predecessors using the **field pair** PC `+0x98`/`+0x9c` ↔ PS2 `+0x224`/`+0x228`, then returns the Nth free seat |
| 0x0043de20 | `ai_follow_manager_invalidate_actor` | `Invalidate__17ai_follow_managerFP7C_actor` | **CONFIRMED** | yes | Three-way scrub on leader `+0x30` / linked car `+0x34` / follower, then **tail-calls `update`** on both builds — the consumer-position pin that identified `ai_follow_manager_update` |
| 0x0043df40 | `ai_follow_manager_event` | `Event__17ai_follow_managerFUiP7C_actor` | **CONFIRMED** | yes | The group-event dispatcher (events 2/3/4); see the event note below |
| 0x0043e0a0 | `stub_retfalse_args4_0043e0a0` (§2c) | `mustEnter__17ai_follow_managerFP7C_actor` | **STRUCTURAL** | n/a | a COMDAT-shared `return 0` |
| 0x0043e0b0 | `ai_follow_manager_should_follower_leave_group` | `mustLeave__17ai_follow_managerFP7C_actor` | **CONFIRMED** | yes | Identical truth table on `+0x24`: state 1 unconditional, states 0/2/3/4/5 gated by `+0x20 & 0x20` |
| 0x0043e150 | `ai_follow_manager_get_save_size` | `GetSize__17ai_follow_managerFv` | **CONFIRMED** | yes | `8 + Σ(4 + (followers + 3) * 0x20)` — the `+3` is the leader and the linked car alongside the followers |
| 0x0043e1c0 | `ai_follow_manager_save` | `Save__17ai_follow_managerFPPv` | **CONFIRMED** | yes | **Five consecutive identical field offsets in identical order** (`+0x1c`, `+0x20`, `+0x24`, `+0x28`, `+0x2c`) after the 0xc-byte destination vector |
| 0x0043e3d0 | `ai_follow_manager_load` | `Load__17ai_follow_managerFPPv` | **CONFIRMED** | yes | Opens with `destroy` on both builds, mirrors `save`'s order, kind-checks each restored reference (follower `0x1b`, leader `2`/`0x1b`, car `4`) |

PC-only: `ai_follow_manager_dtor` (`0x00443a20`) walks the follow-group vector (stride `0x38`),
drains and frees each group's inner `vector<C_actor*>` (element `+0x04`, stride 4), then frees the
buffer. Exactly one entrypoint (the `ai_manager` unwind funclet), so it is the class destructor,
not a folded container helper; the PS2 inlines it into `__dt__10ai_managerFv`.

**The event ids, and the two that do nothing** [H]. `ai_follow_manager_event` is a chain of
`if (event == 2) … else if (== 3) … else if (== 4) …` with **no trailing else**, so any other id
falls straight through and the call is a no-op. The six live callsites pass five distinct ids:

| event | passed by | effect |
|---|---|---|
| 0 | `entity_action_tick` | none — no arm matches |
| 1 | `entity_action_tick` | none — no arm matches |
| 2 | `entity_death` | forwards to `ai_follow_manager_del_follower(target)` and returns |
| 3 | `entity_force_script` | cancel-by-target: drops the matching follower and moves its group to state 3 |
| 4 | `player_do_action_button_in_car` | flips the matched group between states 2 and 4, restamping `+0x04` |

So two of the six callsites are inert as the code stands.

### 2b. `ai_follow_group` — 1 of 1

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / decisive evidence |
|---|---|---|---|---|---|
| 0x00441620 | `ai_follow_group_vector_insert_at` | - | - | n/a | `std::vector<ai_follow_group>::insert`; 3 callers, all `ai_follow_manager_*` |
| 0x004426f0 | `ai_follow_group_copy_ctor` | `__ct__15ai_follow_groupFRC15ai_follow_group` | **CONFIRMED** | yes | Same member set, same record size, 224/204 B = 1.10 |
| 0x00442b30 | `ai_follow_group_copy_assign` | - | **STRUCTURAL** | n/a | GCC inlines it on the PS2 |

### 2c. The three `ai_follow_manager` methods without a PC function — none PS2-only

| PS2 method | Why there is no PC function |
|---|---|
| `MemWrite__17ai_follow_managerFPPvPvUiUi` | **Factoring.** PS2 wraps the savegame stream primitive in a 72-byte per-class method; PC calls `entity_mem_write` **directly** at every site |
| `MemRead__17ai_follow_managerFPPvPvUiUi` | Same, against `entity_mem_read` |

| `mustEnter__17ai_follow_managerFP7C_actor` | **COMDAT fold.** It HAS a PC body, but the body is `return 0` and MSVC folded it onto `stub_retfalse_args4_0043e0a0` together with every other identical `return 0` in the image ¹ |

¹ The mapping is real at one callsite and false for the symbol, so it is recorded as an inline
comment at that callsite plus a note on the stub's own docblock, and the symbol keeps its stub
name. Two empty bodies cannot be separated
by a body-diff; the callsite was pinned by consumer position instead (§3a).

### 2d. Emission order — one translation unit, three subsystems

The merged PS2 order across `ai_group`, `ai_groups`, `ai_follow_manager` and `ai_follow_group`
maps monotonically onto merged PC order, with **zero** non-monotone pairs:

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

The class range begins only after `ai_group_can_receive` (`0x0043ccf0`) and `ai_group_can_cast`
(`0x0043cd30`); the merged chain has 13 anchors. That interleaving is the signature of one shared TU, and it is why `structs/ai_group.md`, this file and
`structs/ai_fight_manager.md` describe neighbours rather than unrelated classes.

---

## 3. What `ai_follow_manager_update` actually does

The per-frame engine. Its four worker calls are all `pursuit_event_*`.

Each group runs a six-state machine on `+0x24`:

| state | meaning | leaves when |
|---|---|---|
| 0 | followers are outside the leader's vehicle | all aboard → 2 · leader on foot → 1 · not boardable → 3 |
| 1 | the leader has no vehicle | he acquires one → 0 |
| 2 | a vehicle task is dispatched and running | task completes → 5 · task fails → 1 · a follower drops out → 0 |
| 3 | the group cannot use the leader's vehicle | a follower's situation changes → 0 |
| 4 | waiting on `+0x28` before re-dispatching | deadline passes → re-dispatch, → 2 |
| 5 | settled | — |

On dispatch it arms `+0x2c` with `now + 6000` and calls **`pursuit_event_dispatch_pursuit_vehicle`**
if any follower is in danger (`entity_is_in_danger`), otherwise
**`pursuit_event_dispatch_move_to_vehicle`**. State 2 polls
**`pursuit_event_record_get_action_status_by_vehicle`** and tears down with
**`pursuit_event_remove_vehicle`**.

🔴 **None of that is police.** The `pursuit_event_*` family is `C_sledovacka`, the **generic
vehicle-task dispatcher** — the same one the police use, and named for that dominant consumer.
The body-diff maps those four calls 1:1 onto PS2 `AddCarHunt` / `AddCarMoveTo` / `ActionStatus` /
`RemoveCar`, all `__12C_sledovacka`. An escorting NPC uses it to make a car come to him.

### 3a. The `mustEnter` callsite

PS2 `AI__8C_entity` runs four consecutive follow arms; PC `entity_action_tick` runs the same four
in the same order:

| PS2 | PC |
|---|---|
| `mustLeave == 0 && CanFolloshipFollow != 0` → `NewFollowLink` | the arm above |
| `vehicle != 0 && mustLeave != 0` → clear bit 0, unlink | `actor_get_vehicle() != 0 && ai_follow_manager_should_follower_leave_group() != 0` → same |
| `vehicle != 0 && other != vehicle && **mustEnter == 0**` → same | `actor_get_vehicle() != 0 && iVar18 != iVar20 && **stub_retfalse_args4_0043e0a0() == 0**` → same |
| `vehicle != 0 && other == vehicle` → clear bit 0 | the same-vehicle arm below |

PS2's `param_1[0x89]` is offset `0x224` — the current-vehicle field whose PC twin `+0x98` is
already proven — so PS2's `fVar28 != param_1[0x89]` **is** the PC `iVar18 != iVar20` test.

Both `mustEnter` bodies are `return 0`, so nothing separates the symbols. The evidence lives where
it is true: at the callsite.

---

## 5. PC vs PS2

- **Group record stride: PC `0x38` / PS2 `0x40`**, proven by the allocation pair
  `operator_new(0x38)` ↔ `__nw__(0x40)` rather than inferred from the walk stride.
  ⚠️ The divergence is **entirely inside the follower vector** — `+0x30` (leader) and `+0x34`
  (linked car) are at *identical* offsets on both builds. Do not rebase the record wholesale.
- **Manager fields carry a flat +4 delta**: PC `+0x04`/`+0x08` begin/end and `+0x10` heartbeat
  against PS2 `+0x00`/`+0x04` and `+0x0c`.
- **The follower vector representation differs**: PC keeps begin/end/cap pointers (MSVC
  `std::vector`), PS2 keeps a `gif::vector` at `group+0x10`. Every count is `(end-begin)>>2` on PC
  against a plain load on PS2 — expect this shape difference in every method; it is not a
  behavioural difference.
- 🔑 **Element types come from PS2 templates, not from the PC decomp**:
  `gif::vector<15ai_follow_group, 1>` types the manager's vector and `gif::vector<P7C_actor, 8>`
  the follower vector (a `C_actor*` vector with an initial reserve of 8).
- **Serialisation differs in ONE respect, consistently**: PC writes each entity reference as a
  **0x20-byte frame-name string** and resolves it back with `mission_find_actor_by_name`; PS2
  writes a **4-byte GUID** and resolves it with `GetActorFromGUID__6C_gameFUi`. That single
  difference explains every constant that differs between the paired `GetSize` bodies
  (PC `4 + (n+3) * 0x20` vs PS2 `n * 4 + 0x2c`) and the 0.44 size ratio on `Load`. It also holds
  for `ai_groups` (`structs/ai_group.md` §5), so it is a build-wide convention, not a per-class choice.
- **`MemWrite` / `MemRead` are a factoring difference, not PS2-only** — see §2b.
- **The mission clock**: PC `*(g_act_mission+0x24)+0x2b0c` ↔ PS2 `*(act_mission+0x20)+0xe54`.
- **The `2000` ms heartbeat and the `6000` ms task deadline are identical on both platforms**, as
  are the state codes, the flag bits and the `mustLeave` truth table.
- **`ai_follow_manager` is a singleton on PS2 at `0x50917c`** and on PC at `0x00655304`; both
  are sub-objects of the AI root object (`structs/ai_fight_manager.md` §5 for the root layout).
  🔑 All **34** callsites in `.text` that load `0x00655304` into ECX (`mov ecx, 0x655304`) resolve
  to exactly the **15** functions in §2a and nothing else. That confirms the §2a mapping without
  any body-diff, corroborates the COMDAT-folded `mustEnter` stub (`0x0043e0a0` is in the census,
  so it really is reached as a method call on this singleton), and makes §2c's accounting a
  measurement: there is no further PC method to find. `0x0043e610` and `0x0043e870` are absent
  from the census — they are `ai_fight_manager`'s `JoinParty` and `LeaveParty`
  (`structs/ai_fight_manager.md` §2a).
- **`ai_follow_group::__ct__` is out of line on the PC** (§2b), in the container-helper cluster at
  `0x0044xxxx` rather than in the class cluster.
- **No PS2-only methods** identified in either class.

---

## 6. Related

- `structs/ai_manager.md` — the root object that holds `ai_follow_manager` at `+0xec`.
- `structs/ai_group.md` — `ai_group` / `ai_groups`, the neighbours in this translation unit.
- `structs/ai_fight_manager.md` — the third subsystem in the TU. Its `count_free_seats` proved
  the car seat-array offsets this class reuses, and its `Init`/`Destroy` split is the same
  build-invariant discriminator.
- `structs/pursuit-event-manager.md` — `C_sledovacka`, the generic vehicle-task dispatcher
  `ai_follow_manager_update` drives (§3).
- `structs/entity.md` §6c — `entity_action_tick`, which calls
  `ai_follow_manager_should_follower_leave_group` and the folded `mustEnter` stub.

---

## 7. Open items

| item | state |
|---|---|
| `ai_follow_group+0x1c` [U] | serialised by `ai_follow_manager_save` and restored by `ai_follow_manager_load`, so real state, but no reader located on either build; lever: the PS2 `Save` body's `puVar4 + 7` slot |
| `ai_follow_group+0x00` [U] | probably not a field at all (§1a); the only byte-sized write in the record, settled by one disassembly read |
| states 3, 4 and 5 [M] | inferred from which arms write them, not from a consumer; lever: `ai_follow_manager_event`, the only other writer |
| callers of `ai_follow_manager_add_follower_with_destination` | not enumerated; this is how a script or mission tells an escort *where* to go |
