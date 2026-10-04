# `ai_group` / `ai_groups` — NPC groups and the group registry

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-09-29 12:39 +0200

Two classes, one subsystem:

- **`ai_group`** (11 methods) — ONE group of NPCs. Holds a vector of `ai_group_member` records and
  acts as a small **event bus**: members cast events into the group, other members receive them, and
  a per-member `role` field decides which direction each member is allowed to use.
- **`ai_groups`** (12 methods) — the **registry** that owns every group, hands out ids, ticks them,
  and serialises them into the savegame.

```
  ai_groups   (registry: ids, Tick, Save/Load)      §2b
      |  owns N of
      v
  ai_group    (one group; event bus + car helpers)  §2a
      |  holds a vector of
      v
  ai_group_member  { C_entity *entity; uint role }  §1a
```

| | |
|---|---|
| PC prefixes | **`ai_group_`** · **`ai_groups_`** (the PS2 mangled tags `__8ai_group` / `__9ai_groups` keep the two apart) |
| PC coverage | `ai_group` **11/11**, `ai_groups` **10/12** ¹ |
| PC cluster | `[0x0043b070, 0x0043bdd0)` for the head of both classes, with `canReceive` / `canCast` at `[0x0043ccf0, 0x0043cd70)` and the compiler-generated members in the container-helper cluster around `0x00441000`..`0x00443b20` |
| `ai_groups` singleton | **`0x006552dc`** — a MEMBER of the `ai_manager` root object at `ai_manager+0xc4` (§1c) |

¹ The two `ai_groups` rows without a PC function are `MemWrite` / `MemRead`, a factoring difference
rather than a gap (§2b).

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Layout

### 1a. `ai_group_member` — 8 bytes

```c
typedef struct ai_group_member {             // 8 bytes on PC
    /* +0x00 */ C_entity *entity;            // [H] the member; also the lookup key
    /* +0x04 */ uint32_t  role;              // [H] see the role table below
} ai_group_member;
```

The element type comes from the PS2 template argument
`erase__Q23std70__vector_imp<>F**P15ai_group_member**Ui`; the PC decomp shows only stride-8 rows.

**The `role` field** [H] — read by exactly two functions, which between them exclude two
different values, and named by the script keywords that set it:

| role | script keyword | `ai_group_can_cast` (`!= 1`) | `ai_group_can_receive` (`!= 2`) | reading |
|---|---|---|---|---|
| 0 | - (default) | may cast | may receive | full participant |
| 1 | `RECEIVE` | - | may receive | **receive-only** — fed group events, may not originate them |
| 2 | `CAST` | may cast | - | **cast-only** — may broadcast, is not sent events itself |

**The writers are a closed set.** `ai_group_add_member` stores its third argument as the role,
and its five callsites pass either the script value or a literal 0:
`entity_process_command` case `0x102b` (the AI instruction `enemy_group_add <group>, <actor>
[, RECEIVE | CAST]`, decoded by `entity_script_decode_params` case `0x102b`: default 0,
`RECEIVE` → 1, `CAST` → 2), `police_board_create_and_engage` ×2 and
`traffic_car_spawn_mafia_pursuit_car` ×2 (both 0). A savegame load restores saved values, so no
role outside 0..2 ever exists. In the shipped scripts `CAST` is used by MISE08-HOTEL `recep` and
MISE18-PRISTAV `City_Guard3`, `City_Guard5` and `Guard_bouda`; `RECEIVE` by MISE07-SARA `Billy`.

### 1b. `ai_group` — the group record (0x60 bytes) and its five containers

```c
    /* +0x14 */ ai_group_member *members_begin;   // [H] count = (end - begin) >> 3
    /* +0x18 */ ai_group_member *members_end;     // [H]
    /* +0x24 */ C_actor       **notified_begin;   // [H] event subjects already announced once;
                                                  //     the de-duplication set of §1e. Also
                                                  //     scrubbed by ai_groups_invalidate_actor
    /* +0x28 */ C_actor       **notified_end;     // [H] count = (end - begin) >> 2
    /* +0x34 */ ai_group_event  *events_begin;    // [H] the event queue, stride 0x14 (§1e)
    /* +0x38 */ ai_group_event  *events_end;      // [H]
    /* +0x44 */ C_car          **cars_begin;      // [H] candidate cars, stride 4
    /* +0x48 */ C_car          **cars_end;        // [H]
    /* +0x54 */ float           *factors_begin;   // [H] one float per car, stride 4
    /* +0x58 */ float           *factors_end;     // [H]
    /* +0x5c */ float           *factors_cap;     // [H]
```

`ai_group` has **five** inner containers, confirmed twice over: the PS2 constructor constructs five,
and `ai_groups_dtor` frees five per element. `ai_group_dtor` releases them in reverse order
(`+0x54`, `+0x44`, `+0x34`, `+0x24`, `+0x14`).

`ai_group_add_member` additionally writes the group's **id** (the record's first dword) into the
member entity at `entity+0x11ac`; every reader resolves it with `ai_groups_get_group(id)`, so an
entity names its group rather than pointing at it. It also empties the group's event queue
(`+0x38 = +0x34`) on every successful append.

The car pair is indexed together: `ai_group_add_car` (`0x0042dcb0`) push-backs into both;
`ai_group_get_car_factor` finds a car in the first and returns the float at the same index in the
second; `ai_group_get_closest_usable_car` iterates the first.

### 1c. `ai_groups` — the registry

Singleton at **`0x006552dc`**, byte-proven twice: of the 30 sites in `.text` that load that address
into ECX, 29 call an `ai_groups_*` method; and the MSVC unwind funclet
`mov ecx,[ebp-0x14] ; add ecx, 0xc4 ; jmp ai_groups_dtor` shows it is `ai_manager+0xc4`
(`structs/ai_manager.md`).

```c
    /* +0x00 */ uint32_t  control_lock_deadline; // [H] absolute ms; the PLAYER's controls stay
                                            //     locked until it passes. 0xffffffff = no lock.
                                            //     Armed/extended by ai_groups_extend_control_lock
                                            //     (max(cur, now+800), called only for kind==2 one
                                            //     instruction before player_lock_controls(1));
                                            //     cleared by ai_groups_tick, which then calls
                                            //     player_lock_controls(0).
    /* +0x04 */ uint32_t  next_id;          // [H] pre-incremented by ai_groups_new_group
    /* +0x0c */ ai_group *entries_begin;    // [H] stride 0x60 per group record (head at +0x08)
    /* +0x10 */ ai_group *entries_end;      // [H] count = (end - begin) / 0x60
    /* +0x1c */ C_actor **tracked_begin;    // [H] the release list: stride-4 pointer vector,
    /* +0x20 */ C_actor **tracked_end;      //     distance-culled by ai_groups_tick beyond 150
                                            //     units (22500.0)
```

**The tracked vector is a release list for disbanded groups.** Its only producer is
`ai_groups_add_tracked_actor`, called three times by `program_process_command` case `0x17b`
(`GROUP_DISBAND`): once for each disbanded member, once for the group's car and once for each of
the car's occupants. `ai_groups_tick` sets act state 2 on any of them further than 150 units
from the player; a car (kind 4) is also hidden, has its dynamic collisions turned off and is
deactivated through its vtable. So a disbanded group is released once the player has left it.

`ai_groups_dtor` (`0x00443910`) corroborates all four: it frees the tracked vector (head `+0x18`,
begin `+0x1c`), then walks the group vector (head `+0x08`, begin `+0x0c`, stride 0x60) freeing all
five inner containers per element — `ai_group_dtor` inlined — then frees the group buffer. It
destroys **both** of the class's members, which is what makes it the class destructor rather than a
folded `std::vector<ai_group>::~vector`. It is reached only through the exception path; the normal
path is inlined into `ai_manager_dtor`.

**`ai_groups_new_group` returns an ID, not a pointer.** Callers follow it with
`ai_groups_get_group(id)` to obtain the record — `police_board_create_and_engage` does exactly that.
The id lives in the record's first dword, which is what `get_group` scans for.

### 1e. `ai_group_event` — the event queue element, 0x14 bytes

```c
typedef struct ai_group_event {           // 0x14 bytes
    /* +0x00 */ uint32_t  kind;           // [H] only kind 1 is dispatched; an entry of any other
                                          //     kind is spliced out without one
    /* +0x04 */ uint32_t  deadline;       // [H] absolute ms; the entry fires once deadline <= now
    /* +0x08 */ float     damage;         // [H] its ADDRESS is the data argument handed to
                                          //     entity_dispatch_event; the RECEIVER reads it as a
                                          //     float and folds it into that target's
                                          //     ai_character_log +0x14 friend_damage
    /* +0x0c */ C_actor  *caster;         // [H] its +0x5d alive byte gates the whole dispatch, and
                                          //     the member equal to it is skipped - never to self.
                                          //     Scrubbed by ai_groups_invalidate_actor
    /* +0x10 */ C_actor  *subject;        // [H] what the event is about: passed to
                                          //     entity_dispatch_event in its sender position and
                                          //     used as the key of the +0x24 set.
                                          //     Scrubbed by ai_groups_invalidate_actor
} ai_group_event;                         // 0x14
```

**How the queue drains** [H], from `ai_group_process_events`. It walks the queue BACKWARDS and, for
each entry whose `deadline` has passed, splices the entry out regardless of what else happens. An
entry of `kind` 1 whose `caster` is still alive additionally dispatches, unless its `subject` is
already in the `+0x24` set — that check is what stops the same subject being announced twice. When
it does dispatch, it delivers `entity_dispatch_event(subject, 1, &damage)` to every member of the
`+0x14` vector that is neither the `caster` nor rejected by `ai_group_can_receive`, and then appends
`subject` to the `+0x24` set. The receiver travels in ECX and is not rendered by the decompiler.

Held in the third container of the group record, at **`+0x34`/`+0x38`** on PC. This is the queue
`ai_group_process_events` drains and `ai_group_push_event` fills.

The type name comes from the PS2 side — `ai_groups_invalidate_actor`'s twin calls
`erase__Q23std68__vector_imp<>F**P14ai_group_event**Ui`, and the copy constructor's twin spells the
same element type out; the PC decomp shows only stride-0x14 rows with two pointer fields.

Both pointer fields are typed `C_actor *` because `ai_groups_invalidate_actor` erases an event when
**either** matches the departing actor.

**Where `damage` ends up** [H]. `entity_dispatch_event` forwards EVERY event to
`ai_logs_event(subject, kind, data)` before reaching its own switch, and its own `case 1` is an
explicit no-op — so the whole meaning of a kind-1 entry lives in `ai_logs`. There `case 1` calls
`ai_character_log_event_friend_hit`, which dereferences the data pointer as a `float`, ADDS it to
the subject's `ai_character_log +0x14` (`friend_damage`, `structs/ai_logs.md`), pushes that log's
`+0x1c` expiry to now + 30000 ms and bumps the `+0x04` counter. So a kind-1 entry means *"the
caster was attacked by the subject for this much damage"*, broadcast to every other member, and
each receiver books it against the subject as damage done to a friend. The three producers pass
`1.0f`, `50.0f` and `100.0f`.

Stride 0x14 is shared with `ai_fight_member`, which is why `fill_n_stride_0x14`,
`copy_element_stride_0x14`, `std_vector20_copy_assign` and `vec_count_stride_0x14` are COMDAT-folded
and keep mechanical names.

## 2. Method families — PC ↔ PS2

Ordered by **PC VA**, PC name first.

### 2a. `ai_group` — 11 of 11

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / evidence |
|---|---|---|---|---|---|
| `0x0042dcb0` | `ai_group_add_car` | - | STRUCTURAL | no | Push-backs into the candidate-car pair (§1b) |
| `0x0043b070` | `ai_group_add_member` | `Add__8ai_groupFP8C_entityUi` | CONFIRMED | yes | Unique-appends a member and stamps the `npc+0x11ac` back-ref |
| `0x0043b100` | `ai_group_del_member` | `Del__8ai_groupFP8C_entity` | CONFIRMED | yes | Removes EVERY record for an entity — the loop does not stop at the first hit. PC inlines the shift where PS2 calls `erase__…<ai_group_member>`. Ratio 96/124 |
| `0x0043b160` | `ai_group_send_sniff_event` | `GetSniff__8ai_groupFP8C_entityP7C_actor` | CONFIRMED | yes | Mints an `ai_blob` and posts it as event `0xb` — §3. Ratio 512/272 = 1.88, entirely PC inlining `Normalize__8S_vector` |
| `0x0043b360` | `ai_group_push_event` | `UnderAttack__8ai_groupFP8C_entityP7C_actorf` | CONFIRMED | yes | The generic event push; PS2 names it `UnderAttack` |
| `0x0043b5c0` | `ai_group_process_events` | `ProcessEvents__8ai_groupFUi` | CONFIRMED | yes | Drains the group's event queue |
| `0x0043b710` | `ai_group_get_car_factor` | `GetCarFactor__8ai_groupFP5C_car` | CONFIRMED | yes | Scores a car's suitability for the group |
| `0x0043b770` | `ai_group_get_closest_usable_car` | `GetCarClosestAndUsable__8ai_groupFP8S_vectorP8S_vectorPUi` | CONFIRMED | yes | The car-commandeering search |
| `0x0043bd00` | `ai_group_dtor` | `__dt__8ai_groupFv` | CONFIRMED | yes | Frees the record's five inner containers in the same reverse order on both builds — PC `+0x54`, `+0x44`, `+0x34`, `+0x24`, `+0x14` ↔ PS2 `+0x40`, `+0x34`, `+0x28`, `+0x1c`, `+0x10`. Consumer-pinned: `ai_groups_clear_all_groups` calls it once per stride-0x60 element exactly where PS2 `Init__9ai_groups` calls `__dt__8ai_group` per stride-0x4c element |
| `0x0043ccf0` | `ai_group_can_receive` | `canReceive__8ai_groupFP8C_entity` | CONFIRMED | yes | `role != 2`; also consumer-pinned by `GetSniff`. Ratio 64/92 |
| `0x0043cd30` | `ai_group_can_cast` | `canCast__8ai_groupFP8C_entity` | CONFIRMED | yes | `role != 1`. Separated from its identical sibling by the **role constant**, a build-invariant. Ratio 64/92 |
| `0x004424e0` | `ai_group_copy_ctor` | `__ct__8ai_groupFRC8ai_group` | CONFIRMED | yes | Statement-for-statement, 480/304 B = 1.58; its twin is where both element types are spelled out (§1a, §1e) |

**Compiler-generated members and container helpers** (no PS2 counterpart — GCC inlines or folds
them):

| PC VA | PC name | note |
|---|---|---|
| `0x004410a0` | `ai_group_destroy_range` | `_Destroy(first,last)`; sole caller `ai_groups_destroy` |
| `0x00441300` | `ai_group_vector_insert_at` | `std::vector<ai_group>::insert` |
| `0x00442910` | `ai_group_copy_assign` | MSVC-generated `operator=` |
| `0x00442f50` | `ai_group_scalar_deleting_dtor` | MSVC artifact |
| `0x004434f0` | `ai_group_member_vector_capacity` | capacity of the `vector<ai_group_member>` |
| `0x00443b20` | `ai_group_vector_destroy` | `std::vector<ai_group>::~vector` — the CONTAINER's dtor, receiver `ai_groups+0x08` |

### 2b. `ai_groups` — 10 of 12

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / evidence |
|---|---|---|---|---|---|
| `0x0043bb30` | `ai_groups_new_group` | `NewGroup__9ai_groupsFv` | CONFIRMED | yes | Allocates + registers a group, returns its **id**. SEH-guarded |
| `0x0043bd70` | `ai_groups_get_group` | `GetGroup__9ai_groupsFUi` | CONFIRMED | yes | Linear scan of the stride-0x60 entry array |
| `0x0043bdd0` | `ai_groups_clear_all_groups` | `Init__9ai_groupsFv` | CONFIRMED | yes | Both open with the identical pair `*this = 0xffffffff` (the §1c control-lock sentinel) and `+0x04 = 0`, then dispose every entry through `ai_group_dtor` (PC stride 0x60 ↔ PS2 0x4c) and reset the tracked vector. Clears without freeing — the family's `Init`-is-a-teardown idiom. Caller `game_init` |
| `0x0043be40` | `ai_groups_destroy` | `Destroy__9ai_groupsFv` | CONFIRMED | yes | The `clear_all_groups` body **plus** a leading REVERSE pass releasing each record's five inner containers — the same build-invariant discriminator that splits `ai_fight_manager_destroy` and `ai_follow_manager_destroy`, with `Init` emitted first on both builds. Re-pinned by `ai_groups_load` calling it first. Caller `game_done` |
| `0x0043bfe0` | `ai_groups_tick` | `Tick__9ai_groupsFv` | CONFIRMED | yes | The registry's per-frame tick, run by `game_tick` ¹ |
| `0x0043c170` | `ai_groups_get_save_size` | `GetSize__9ai_groupsFv` | CONFIRMED | yes | Shared literal **8** base, shared literal **`0x1c`** per-group constant, the same three containers accumulated in order, and the same trailing `+ 4 + tracked_count * K`. Per-element sizes differ only for the build-wide frame-name-vs-GUID reason (§5) |
| `0x0043c250` | `ai_groups_save` | `Save__9ai_groupsFPPv` | CONFIRMED | yes | Identical grammar: `+0x04`, group count, then per group `+0x00`/`+0x04`/`+0x08`/`+0x0c` and each container as count-then-elements, with the same `(1, 4)` element args on both builds |
| `0x0043c590` | `ai_groups_load` | `Load__9ai_groupsFPPv` | CONFIRMED | yes | Opens with the class teardown on **both** builds before reading anything — a consumer-position pin that also re-confirms `ai_groups_destroy` — then mirrors `save`'s order. SEH-guarded |
| `0x0043ca60` | `ai_groups_invalidate_actor` | `InvalidateActor__9ai_groupsFP7C_actor` | CONFIRMED | yes | Three containers scrubbed in the same order with the same geometry (stride 8 keyed on element `+0x00`; stride 4; stride **0x14** matched on element `+0x0c` **OR** `+0x10`), same remember-the-emptied-group-then-splice, same trailing pass over the registry's tracked vector. The structural twin of `ai_fight_manager_invalidate_actor` |
| `0x0043cd70` | `ai_groups_group_has_car` | `hasCarGroup__9ai_groupsFUi` | CONFIRMED | yes | Same `id != 0` guard, same entry scan keyed on element `+0x00`, same "container non-empty" return read from **`+0x44`/`+0x48`** — independently re-confirming §1b's candidate-car vector from a second consumer |
| `0x00443910` | `ai_groups_dtor` | - | STRUCTURAL | no | The class destructor (§1c); PS2 inlines every sub-manager teardown into `__dt__10ai_managerFv` |
| `0x005c8dc0` | `ai_groups_add_tracked_actor` | - | STRUCTURAL | yes (callsites) | The tracked vector's push_back (begin `+0x1c`, end `+0x20`, cap `+0x24`), emitted out of line with the singleton itself as receiver: all 3 callsites load `0x006552dc` into ECX, all in `program_process_command`'s `GROUP_DISBAND` arm (§1c). PS2 counterpart at callsite level (§5) |
| - | *no PC function* | `MemWrite__9ai_groupsFPPvPvUiUi` · `MemRead__9ai_groupsFPPvPvUiUi` | STRUCTURAL | yes (callsites) | **Factoring, not absence.** PS2 wraps the savegame stream primitive in a 72-byte per-class method; PC calls `entity_mem_write` / `entity_mem_read` **directly** at every site. The identical pair exists on `ai_follow_manager` |

¹ (1) Expires the player control lock at `+0x00` → `player_lock_controls(0)`; (2) drains each
group's event queue via `ai_group_process_events` over the stride-0x60 vector at `+0x0c`/`+0x10`;
(3) distance-culls the stride-4 tracked-object vector at `+0x1c`/`+0x20` beyond **150 units**
(22500.0 = 150²). Statement-for-statement with the PS2 body (§5).

## 3. `ai_group_send_sniff_event` — a SECOND scent producer

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
(`structs/ai_sensors.md` §4d). It has nine callers: `entity_action_tick` ×6 (the
steer-to-target arms, a freshness test of under 5000 ms, and the cop chase in case `0x21`),
`ai_logs_reset_stale_logs`, `ai_action_new_hostile_coverage` and `police_board_update_sighting`.
So the lead is a last-known position and heading for the target, used when the member cannot
see it.

## 4. Related

- `structs/ai_manager.md` — the AI root object; this registry is its `+0xc4` member, and the
  `ai_fight_manager` / `ai_follow_manager` singletons sit beside it.
- `structs/ai_fight_manager.md` — the sibling coordination subsystem in the same TU;
  `ai_fight_manager_invalidate_actor` is the structural template for `ai_groups::InvalidateActor`.
- `structs/ai_blob_log.md` — the `ai_blob` record type and the *other* scent producer.
- `structs/ai_sensors.md` — the smell band that consumes event `0xb`, and the read side.
- `structs/ai_logs.md` — the per-target `ai_character_log`; its `+0x14 friend_damage` is where a
  kind-1 entry's `damage` is accumulated.
- `structs/police-board.md` — `police_board_create_and_engage`, the caller that pairs
  `ai_groups_new_group` with `ai_groups_get_group`.

## 5. PC vs PS2

- **Member vector representation differs**: PC keeps `begin`/`end` pointers at `+0x14`/`+0x18`
  (MSVC `std::vector`), PS2 keeps a **count** at `+0x14` and a **data pointer** at `+0x18`
  (`std::__vector_imp`). Every count computation therefore reads `(end-begin)>>3` on PC and a plain
  load on PS2 — expect this shape difference in every method of both classes; it is not evidence of
  a behavioural difference.
- **Group record stride:** PC `0x60` ↔ PS2 `0x4c`; the five inner containers sit at PC `+0x14`,
  `+0x24`, `+0x34`, `+0x44`, `+0x54` ↔ PS2 `+0x10`, `+0x1c`, `+0x28`, `+0x34`, `+0x40`.
- **`ai_group_member` is 8 bytes on both builds** and its two fields are at the same offsets — it
  contains no vector, so there is nothing to pad.
- **`ai_blob` stride: PC `0x24` / PS2 `0x30`** — see `structs/ai_blob_log.md` §5. PS2 allocates it
  through `__nw__7ai_blob` (which tail-calls the PS2-only `ai_spageman` pool); PC uses plain
  `operator_new`. That difference is what TYPES the PC allocation.
- **The mission clock**: PC `*(g_act_mission+0x24)+0x2b0c` ↔ PS2 `*(act_mission+0x20)+0xe54`.
- **Savegame element sizes** differ only for the build-wide frame-name-vs-GUID reason.
- **Every out-of-band size ratio in §2 is a factoring difference in the same direction** — PS2 calls
  `erase__…<ai_group_member>` and `Normalize__8S_vector` out of line where MSVC inlines both, so the
  PC bodies that touch a vector or normalise a vector are the bigger ones.
- **No PS2-only methods** in either class; `ai_groups_dtor` and `ai_groups_add_tracked_actor` are
  PC-only functions, the first because PS2 inlines the teardown.
- **`ai_groups_add_tracked_actor` has a callsite-level counterpart.** The `GROUP_DISBAND` arm of
  PS2 `Process__9C_programFUi` runs `LeaveParty__16ai_fight_manager` and then
  `push_back__Q23gif19vector<P7C_actor,8>(0x509170)` three times, at the same three points as
  the PC calls. `0x509170` is `ai_groups` (`0x50915c`) + `0x14`, the vector `Tick__9ai_groupsFv`
  culls. The PS2 symbol is the generic template, so it is no twin of the PC function, and
  `ai_groups` stays 12/12 accounted.
- **`ai_group_add_member`'s `Ui` parameter** (`Add__8ai_groupFP8C_entityUi`) is the role.
