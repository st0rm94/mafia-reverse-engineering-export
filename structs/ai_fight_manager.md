# `ai_fight_manager` / `ai_fight_followship` — fight parties and the party manager

**Created** 2026-07-30 16:45 +0200 · **Modified** 2026-10-04 20:16 +0200

Two classes, one translation unit, one subsystem — **how NPCs fight as a group instead of
individually**:

- **`ai_fight_manager`** (10 methods) — owns the vector of parties, forms and disbands them,
  ticks them, and scrubs dead actors out of them.
- **`ai_fight_followship`** (8 methods) — ONE party: a commander, a member list, an optional car,
  and a formation deadline. ("Followship" is the developers' spelling and is kept.)

```
  ai_fight_manager      vector<ai_fight_followship>, stride 0x48   §2a
        |  owns N of
        v
  ai_fight_followship   commander + car + members + form deadline  §2b
        |  holds a vector of
        v
  ai_fight_member       stride 0x14                                §1b
```

- **PC prefixes:** `ai_fight_manager_*` · `ai_fight_followship_*`.
- **Coverage: 18 of 18** — `ai_fight_manager` 10/10, `ai_fight_followship` 8/8. Every row is
  CONFIRMED by a statement-level diff of both bodies.
- **PC singleton `0x00655318`**, a member of the `ai_manager` root object at `ai_manager+0x100`
  (§1c).
- **PC cluster:** `[0x0043e610, 0x0043fce0)` plus `[0x00440e10, …)` and the container helpers at
  `[0x004410d0, 0x00442cd0]`. The two classes **interleave by VA on both builds**, which is what
  makes a merged order-align possible (§2c).

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

## 1. Layout

### 1a. `ai_fight_followship` — the party record, 0x48 bytes (PC)

```c
typedef struct ai_fight_followship {          // stride 0x48 on PC
    /* +0x04 */ ai_fight_member *members_begin;  // [H]
    /* +0x08 */ ai_fight_member *members_end;    // [H] count = (end - begin) / 0x14
    /* +0x10 */ float     centre[3];             // [H] the party's mean member position -
                                                 //     written by compute_position, read by
                                                 //     must_car_hunt
    /* +0x28 */ float     spread_sq;             // [H] MAX squared distance from any member to
                                                 //     `centre` - a bounding radius^2
    /* +0x2c */ uint32_t  form_deadline_ms;      // [H] mission clock; set to the CURRENT clock by
                                                 //     new_car_party, += 10000 by adjust_form_time,
                                                 //     += 1000 by join_party
    /* +0x30 */ uint32_t  kind;                  // [H] 1 = CAR party (set by new_car_party);
                                                 //     0 = on-foot party (join_party's zero-filled
                                                 //     new record). No other value is written
    /* +0x3c */ C_entity *key_holder;            // [H] a representative member; both join_party
                                                 //     and leave_party dereference it and compare
                                                 //     its +0xf4c group key against the joining /
                                                 //     leaving entity's, so a party only accepts
                                                 //     members of the same group. The key is the
                                                 //     entity's type group (entity+0xf4c), set by
                                                 //     entity_game_init from human_type
    /* +0x40 */ C_actor  *commander;             // [H] the party leader
    /* +0x44 */ C_car    *car;                   // [H] the party's vehicle (kind == 1)
} ai_fight_followship;                           // 0x48
```

The three offsets `+0x2c` / `+0x30` / `+0x40` are proven together by
`ai_fight_manager_adjust_form_time`, and `+0x40` / `+0x44` / `+0x30` again independently by
`ai_fight_manager_invalidate_actor` and `ai_fight_manager_get_party` — three functions agreeing on
the same field map. **`ai_fight_manager_new_car_party` makes it four**: it writes the clock, `1`,
the commander and the car into exactly those slots, in that order, and allocates `0x48`, which is
the stride claim itself made falsifiable. `+0x10` and `+0x28` are proven by the
`compute_position` / `must_car_hunt` pair: one writes them, the other reads both.

* **`car+0x688` is read as SPEED [M].** Speed fits both consumers — `add_car_member` requires
  `< 0.5` before treating a car as boardable (nearly stopped) and `must_car_hunt` requires `> 2.0`
  before deciding a target's car must be chased by car (moving).
* **`kind == 1` is read as "car party" [H]** because both functions that test it call the
  **car**-removal path on the true branch (`pursuit_event_remove_vehicle(party+0x44)`).

### 1b. `ai_fight_member` — 0x14 bytes, the container lever

```c
typedef struct ai_fight_member {              // 0x14 bytes, both builds
    /* +0x00 */ C_entity *entity;             // [H] the member; also the lookup key
    /* +0x04 */ uint32_t  _unk_04;            // [U] not written by join_party
    /* +0x08 */ uint32_t  _unk_08;            // [U] not written by join_party
    /* +0x0c */ uint32_t  state;              // [M] join_party: 1 if the entity belongs to an
                                              //     ai_group (entity+0x11ac != 0), else 0;
                                              //     take_car sets 2 on the members it seats
    /* +0x10 */ uint32_t  _unk_10;            // [U] 0 when join_party creates the record
} ai_fight_member;
```

`join_party` builds the record as `{entity, -, -, in_ai_group, 0}`; the two words it leaves unset
and the readers of `+0x0c` are open (§7). The
stride `0x14` is shared with `ai_group_event`, which is why the stride-0x14 container helpers are
COMDAT-folded and must stay mechanical.

### 1c. `ai_fight_manager` — the party vector, inside the AI root object

```c
    /* +0x04 */ ai_fight_followship *parties_begin;  // [H]
    /* +0x08 */ ai_fight_followship *parties_end;    // [H] count = (end - begin) / 0x48
```

The singleton `0x00655318` is a **member of the `ai_manager` root object at `ai_manager+0x100`**,
byte-proven by the unwind funclet `add ecx, 0x100 ; jmp ai_fight_manager_dtor`
(`structs/ai_manager.md`). The singleton is also byte-proven directly: 24 callsites load
`0x00655318` into ECX and every one calls a method in §2.

**The class is ONE vector**, so `ai_fight_manager` ends at PC `+0x110`; the vectors at
`ai_manager+0x110` / `+0x120` are the root's next member (`structs/ai_manager.md` §3b), not this
class, and that is why its destructor covers only one vector.

## 2. Method families

### 2a. `ai_fight_manager` — 10 of 10, plus the destructor

| PC VA | PC name | PS2 twin | status | role / decisive evidence |
|---|---|---|---|---|
| `0x0043e610` | `ai_fight_manager_join_party` | `JoinParty__16ai_fight_managerFP8C_entityP7C_actor` | CONFIRMED | Enrols an entity in the party commanded by `param_3`, creating it if none matches. The party stride is proven twice inside this one body — by the walk divisor and, independently, by the zero-fill (PC clears 0x12 dwords = 0x48 B). Shared magics **1000** (deadline bump) and **500** (new-party expiry). Ratio 0.68 |
| `0x0043e870` | `ai_fight_manager_leave_party` | `LeaveParty__16ai_fight_managerFP8C_entityP7C_actor` | CONFIRMED | The mirror: two search arms on `param_3 == 0`, then erase-and-dispatch — kind-1 party → `disband_followship(…, 0)`; emptied party → splice the 0x48-byte record out; members remain → `set_commander` re-elects. Ratio 1.08 |
| `0x0043f090` | `ai_fight_manager_new_car_party` | `NewCarParty__16ai_fight_managerFP5C_carP7C_actor` | CONFIRMED | `operator_new(0x48)` and the same four field writes in the same order (clock→`+0x2c`, `1`→`+0x30`, commander→`+0x40`, car→`+0x44`), plus the same `0x29a` sentinel into the CAR (PC `car+0x2150`). Consumer-pinned by `take_car` |
| `0x0043f730` | `ai_fight_manager_disband_followship` | `DisbandCarParty__16ai_fight_managerFP19ai_fight_followshipb` | CONFIRMED | Consumer-pinned: `invalidate_actor` calls it at all three points where PS2 `Invalidate` calls `DisbandCarParty`; the `b` is the do_notify flag, 0 at all three sites on both builds |
| `0x0043f950` | `ai_fight_manager_adjust_form_time` | `AdjustFormTime__16ai_fight_managerFP7C_actor` | CONFIRMED | Shared magic **10000**, same two-condition guard. Ratio 96/128 |
| `0x0043f9b0` | `ai_fight_manager_get_party` | `GetParty__16ai_fight_managerFP8C_entityP7C_actorPUi` | CONFIRMED | Identical dual-branch search + inner stride-0x14 scan, index via out-param. Ratio **272/272 = 1.00** |
| `0x0043fac0` | `ai_fight_manager_clear_all_parties` | `Init__16ai_fight_managerFv` | CONFIRMED | Initialises nothing: it is a teardown to empty. Separated from `Destroy` by a build invariant (§2d). Ratio 208/120 |
| `0x0043fb90` | `ai_fight_manager_destroy` | `Destroy__16ai_fight_managerFv` | CONFIRMED | `clear_all_parties` **plus** a leading reverse pass emptying each party's member vector. Ratio 336/236 |
| `0x0043fce0` | `ai_fight_manager_tick` | `Tick__16ai_fight_managerFv` | CONFIRMED | The per-frame party engine: 2 s heartbeat, member state machine, car commandeering, pursuit dispatch |
| `0x00440e10` | `ai_fight_manager_invalidate_actor` | `Invalidate__16ai_fight_managerFP7C_actor` | CONFIRMED | Three-way scrub, §3. Consumer-pinned three times in one body |
| `0x00443a90` | `ai_fight_manager_dtor` | - ¹ | STRUCTURAL | Walks the party vector (stride 0x48); per party drains and frees its inner `vector<ai_fight_member>` (element `+0x04`, stride 0x14); then frees the buffer. One entry point ⇒ no fold |

¹ No PS2 symbol: inlined into `__dt__10ai_managerFv` (§5).

### 2b. `ai_fight_followship` — 8 of 8

| PC VA | PC name | PS2 twin | status | role / evidence |
|---|---|---|---|---|
| `0x0043ebc0` | `ai_fight_followship_set_commander` | `SetCommander__19ai_fight_followshipFv` | CONFIRMED | Consumer-pinned by `invalidate_actor`'s "members remain, elect a replacement" arm |
| `0x0043ecc0` | `ai_fight_followship_count_free_seats` | `CountFreeSeats__19ai_fight_followshipFP5C_car` | CONFIRMED | Seat array PC `+0xd90`/`+0xd94` stride `0x24`; same `car_get_owner` per-seat call; same member scan + early `return 0`. A seat that is free but fails the reachability raycast (mask **0x140502**) disqualifies the whole car |
| `0x0043ee60` | `ai_fight_followship_take_car` | `TakeCar__19ai_fight_followshipFP5C_car` | CONFIRMED | Calls `count_free_seats` first and `new_car_party` last on both builds — one diff that pins three rows. Same 8-slot local index array, same leader-first flag, same three member qualifying tests |
| `0x0043f110` | `ai_fight_followship_add_car_member` | `AddCarMember__19ai_fight_followshipFP8C_entity` | CONFIRMED | Same current-vehicle field (PC `entity+0x98`), same compare against the party car, same `kind==4 && speed<0.5` branch, same 0/1/2 boarding codes |
| `0x0043f3f0` | `ai_fight_followship_compute_position` | `ComputePosition__19ai_fight_followshipFv` | CONFIRMED | Two passes: mean member position → `+0x10`, then max squared distance to it → `+0x28` |
| `0x0043f590` | `ai_fight_followship_must_car_hunt` | `mustCarHunt__19ai_fight_followshipFv` | CONFIRMED | §2e. Shared magics **100.0** / **10000.0** / **2.0**, kind codes 4 and 8, gate `kind == 2 \|\| 0x1b`, the `+0x28` spread threshold, the `car+0x688` speed field |
| `0x004427f0` | `ai_fight_followship_copy_ctor` | `__ct__19ai_fight_followshipFRC19ai_fight_followship` | CONFIRMED | Same field order, 288/236 B = 1.22 |
| `0x00442cd0` | `ai_fight_followship_copy_assign` | `__as__19ai_fight_followshipFRC19ai_fight_followship` | CONFIRMED | Same order as the copy ctor, `do_assign<>` where the ctor has `init<>`. 6 callers, every one an `ai_fight_manager` path or this class's own helpers |

`new_car_party` (`0x0043f090`) sits inside this VA range because the two classes interleave.

The class's container helpers, all single-owner: `ai_fight_followship_vector_insert_at` ·
`_vector_insert_n` · `_vector_count` · `_destroy_range` · `_uninit_copy_range` · `_uninit_fill_n` ·
`_assign_range` · `_assign_range_backward` (`[0x004410d0, 0x00442280]`).

### 2c. Emission order (the anchor chain)

Merged PS2 order across both classes maps monotonically onto merged PC order:

```
  SetCommander  CountFreeSeats  TakeCar  NewCarParty  AddCarMember  ComputePosition
     0x43ebc0       0x43ecc0    0x43ee60   0x43f090      0x43f110       0x43f3f0
  mustCarHunt  DisbandCarParty  AdjustFormTime  GetParty  Init  Destroy  Tick  Invalidate
     0x43f590      0x43f730        0x43f950     0x43f9b0  0x43fac0 0x43fb90 0x43fce0 0x440e10
```

Zero non-monotone pairs. The four rows in the middle (`AdjustFormTime` … `Destroy`) sit in a
forced 4:4 gap between the `DisbandCarParty` and `Tick` anchors; `Invalidate` falls out of the
**trailing** gap after the last anchor.

A gap is only forced when it is enumerated as **Ghidra `FUN_` ids ∪ already-named defs**, and
against the PS2 entries **capable of having a PC function** — a method already established as
inlined (here `ai_fight_followship`'s `__ct__` / `__as__` in the `JoinParty` gap) is an input to
that arithmetic, not a footnote.

### 2d. The identical-sibling resolution

`Init` and `Destroy` have near-identical bodies on both builds and cannot be separated by size,
arity or callee set. What separates them is a **build invariant**: `Destroy` alone is preceded by
an extra REVERSE pass that empties each party's member vector, on both builds, and both builds
emit `Init` first — a derived property that survives recompilation.

### 2e. Why `must_car_hunt` is a hunting decision, not a range test

Its three branches:

| target is… | returns 1 when |
|---|---|
| on foot | squared x/z distance from `party+0x10` exceeds **10000.0** |
| in a kind-8 object | 3D squared distance exceeds **100.0** |
| **in a car** (`kind == 4`) | the car's **speed** (`car+0x688`) exceeds **2.0** *and* the target is beyond the party's own spread `party+0x28` — otherwise it still applies the 10000.0 test |

The car branch returns 1 for a target that is merely **moving**, at only moderate distance. That is
not a range test. The question being answered is *"can we still catch this on foot, or do we need a
vehicle?"* — exactly `mustCarHunt`. `ai_fight_manager_tick` keys its keep-pursuing
vs. commandeer-a-vehicle decision on the answer.

## 3. What `ai_fight_manager_invalidate_actor` does

Called when an actor dies or leaves the world. It reverse-walks the parties and dispatches three
ways per record:

| the actor is… | action |
|---|---|
| the party **commander** (`+0x40`) | if `kind == 1`, remove the car from the pursuit-event manager and disband; else splice the record out of the party vector |
| the party's **car** (`+0x44`) | disband the party |
| an ordinary **member** | erase its `ai_fight_member` records; if the party is now empty, disband-or-splice as above; if members remain, **elect a new commander** |

This one body pins **three separate twins at once** by callsite position —
`pursuit_event_remove_vehicle ↔ RemoveCar__12C_sledovacka`,
`ai_fight_manager_disband_followship ↔ DisbandCarParty`, and
`ai_fight_followship_set_commander ↔ SetCommander`. A caller that dispatches N ways is N
consumer-pins in one read.

`ai_groups_invalidate_actor` (`0x0043ca60`) is the structurally parallel scrub in the sibling
class: same reverse walk, per-record multi-container scrub, and remember-then-splice of an emptied
record. It scrubs **three** containers per record where this class scrubs one, and its third
container is matched on **two** actor fields rather than one — which is what surfaced the
`ai_group_event` type (`structs/ai_group.md` §1e).

## 5. PC vs PS2

- **Party record stride: PC `0x48` / PS2 `0x60`** (PS2 `NewCarParty` allocates `__nw__(0x60)` and
  `JoinParty` issues `memset(…, 0x60)`; the walk divisors agree). The field map is otherwise a
  clean flat **+4** delta, confirmed independently by three functions:

  | field | PC | PS2 |
  |---|---|---|
  | members begin / end | `+0x04` / `+0x08` | count `+0x24`, data `+0x28` |
  | form deadline | `+0x2c` | `+0x30` |
  | kind | `+0x30` | `+0x34` |
  | commander | `+0x40` | `+0x44` |
  | car | `+0x44` | `+0x48` |

  The **member vector is the exception** — PC keeps begin/end pointers (MSVC `std::vector`),
  PS2 keeps count + data (`std::__vector_imp`), so that field pair does not follow the +4 rule.
  Do not rebase a single offset; use the table.
  **Where the `0x18` bytes go:** +4 inside the party centre (a 16-byte PS2 `S_vector` against 12 PC
  bytes; PS2 `NewCarParty` zeroes the three centre floats at `+0x10`), plus `0x14` bytes after the
  car pointer (PS2 `+0x4c..+0x5f`) that no PS2 party method reads or writes.
- **`ai_fight_member` is `0x14` bytes on both builds.** The type name is a PS2 gift —
  `erase__Q23std70__vector_imp<>F**P15ai_fight_member**Ui` in `Invalidate`; the PC decomp shows
  only "stride-0x14 rows".
- **Other cross-build field deltas:** party car `car+0x688` ↔ PS2 `car+0x8f0`; the `0x29a`
  sentinel PC `car+0x2150` ↔ PS2 `car+0x23ac`; the seat array PC `+0xd90`/`+0xd94` stride `0x24` ↔
  PS2 `+0xfd0`/`+0xfd4` stride `0x30`; current vehicle PC `entity+0x98` ↔ PS2 `+0x224`.
- **`10000` ms formation extension is identical on both platforms**, as is the `kind == 1` car
  test and the three-way `Invalidate` dispatch.
- **`ComputePosition` uses VU ops on PS2** (`_vadd`/`_vdiv`/`_vmulq`) where the PC uses x87 — a
  rendering difference, not a factoring one. `AddCarMember`'s boarding codes are 0/1/2 on PC and
  0/6/8 on PS2.
- **PS2 calls `__as__19ai_fight_followship` (`operator=`) to shift records down** when splicing a
  party out; MSVC inlines the same shift as a 0x48-byte field-by-field copy. That is the whole of
  `invalidate_actor`'s size difference.
- **`ai_fight_manager_dtor` has no PS2 twin:** the PS2 build inlines it into `__dt__10ai_managerFv`.
- **Size ratios run 0.68–1.73**, all in or near band; the one exact hit is `GetParty` at
  272/272 = 1.00. `ai_fight_followship_copy_assign` is 640/200 B — a FACTORING difference: GCC
  calls `do_assign<>__…<15ai_fight_member>` out of line where MSVC inlines all three capacity
  branches. A ratio outside the 1.0–1.6 band is a prompt to look for the factoring, not a
  refutation.
- **`RemoveCar` is OVERLOADED on PS2** — `RemoveCar__12C_sledovackaFP5C_car` (the pursuit/chase
  manager, the one that matches here) and `RemoveCar__13G_GarageClassFPc` (the garage, takes a
  `char*`). The PC caller passes a car pointer and the `pursuit_event_*` family IS `C_sledovacka`,
  so the first is correct.
- **No PS2-only methods** in either class.
- **The AI root object.** PS2 `Init__10ai_managerFv` embeds every AI sub-manager at a fixed offset
  in one root object at `0x509080`, which is what makes the PS2 singleton addresses `root + N`:

  | sub-manager | PS2 offset | PS2 absolute | PC singleton |
  |---|---|---|---|
  | `ai_groups` | `+0xdc` | `0x50915c` | `0x006552dc` |
  | `ai_follow_manager` | `+0xfc` | `0x50917c` | `0x00655304` |
  | **`ai_fight_manager`** | **`+0x10c`** | **`0x50918c`** | **`0x00655318`** |
  | `ai_sight_manager` | `+0x118` | `0x509198` | `0x00655328` [M] ¹ |
  | `ai_panic_manager` | `+0x138` | `0x5091b8` | - |
  | `ai_framerater` | `+0x148` | `0x5091c8` | - |
  | `ai_spageman` ×5 | `+0x14c…+0x20c` | - | - |
  | `ai_unlock_manager` | `+0x23c` | `0x5092bc` | - ² |

  `-` = no PC singleton; the sub-manager is PS2-only unless noted. ¹ Two vectors, no code
  (`structs/ai_manager.md` §3b). ² No PC member: `structs/ai_manager.md` §4 maps it to the
  `ai_groups+0x00` control-lock scalar (`structs/ai_group.md` §1c).

  The PS2 spacing `root+0x10c → +0x118` is `0xc` — exactly one `gif::vector` — which is how
  `ai_fight_manager` is known to be a single vector (§1c).

  **The inter-sibling deltas are NOT the same on both builds** (PC `ai_groups`→`ai_follow` is
  `0x28` against PS2's `0x20`) because the PC sub-objects are larger — MSVC `std::vector` keeps
  begin/end/cap where PS2's `gif::vector` keeps count/data. **Do not derive one PC singleton from
  another by rebasing;** each is byte-proven independently by a census of the ECX immediate at its
  callsites. The PC root holds **one more member after `ai_fight_manager`** — the code-less
  two-vector member at `+0x110` — and ends at `+0x130`. The next referenced addresses are not
  managers: `0x00655348` is the actor possible-collision table (`actor_posscoll_table_ctor`),
  then `0x0065535c`.

## 6. Related

- `structs/ai_manager.md` — the root object this class is a member of.
- `structs/ai_group.md` — the sibling coordination subsystem sharing this TU.
- `structs/ai_follow_manager.md` — the third subsystem in the same TU. It reuses this class's
  proven car seat-array offsets in `ai_follow_manager_group_fits_leader_vehicle`, and its
  `Init`/`Destroy` pair is split by the same build-invariant discriminator recorded in §2d.
- `structs/pursuit-event-manager.md` — `C_sledovacka`, whose `RemoveCar` this class calls when
  disbanding a car party.
- `structs/ai_fight_modes.md` — the per-character combat-mode scorer; a different class despite
  the shared `ai_fight` stem.
- `structs/entity.md` §6c — `entity_ai`, the caller of `JoinParty` / `LeaveParty`.

## 7. Open items

| item | state |
|---|---|
| `ai_fight_member` `+0x04` / `+0x08` / `+0x0c` / `+0x10` (§1b) | `join_party` writes `+0x0c` (in an `ai_group` or not) and `+0x10` (0), `take_car` sets `+0x0c = 2`; `+0x04` / `+0x08` are not written on the join path, and the readers of all four are not traced |
