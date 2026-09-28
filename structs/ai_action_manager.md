# `ai_action_manager` — the NPC action queue (what an AI person can be told to do)

**Created** 2026-07-16 21:10 +0200 · **Modified** 2026-09-18 05:52 +0200

`ai_action_manager` owns an NPC's **action queue**. It is the largest AI class in the game
(**58 PS2 methods**) and it is the class the whole `ai_action` vocabulary hangs off: 35 factories
that mint a typed `ai_action`, three parallel lists (jobs / modifiers / idles) they get filed into,
and the cursor/lifetime machinery that advances them. `entity_action_tick` then switches on an
action's type id to execute it.

| | |
|---|---|
| PC prefix | `ai_action_manager_*` for the manager · `ai_action_new_*` for the factories · `ai_action_*` for operations on one action |
| PC object | **embedded** at `C_entity + 0x1164`, `0x34` bytes |
| PS2 class | `ai_action_manager`, 58 methods = **35 factories + 23 members** |
| Coverage | 58/58 accounted — 23 members CONFIRMED by body-diff (§3b), 35 factories joined 1:1 by type id and individually twinned (§3a) |
| Field layout | fully resolved (§1) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Field layout (PC offsets, relative to the `ai_action_manager` base)

The object is **embedded** in `C_entity`, not pointed to: `lea ecx,[reg+0x1164]` appears at more than
twelve callsites of `ai_action_manager_clear_all` and `ai_action_manager_get_base_job`, measured by
reading the receiver expression at every callsite of both. `lea` means embedded. The independent
corroboration is `entity_get_action_man_size`, which reads `entity+0x1168` — exactly this object's
`+0x04`.

Three parallel `std::vector<ai_action*>` lists plus a counter, and nothing else.

```c
typedef struct ai_action_manager {            // embedded at C_entity+0x1164; sizeof == 0x34
    /* ---- list 1: JOBS — std::vector<ai_action*> ---------------------------------- */
    /* +0x00 */ allocator_t   jobs_alloc;      // [H] MSVC's empty std::allocator member, padded to 4.
                                               //     NOT an unknown field and NOT a capacity - see §1b.
    /* +0x04 */ ai_action  **jobs_begin;       // [H] = C_entity+0x1168, which is exactly what
                                               //     entity_get_action_man_size reads - an independent
                                               //     confirmation of the base. Stride 4.
    /* +0x08 */ ai_action  **jobs_end;         // [H] count = (end - begin) >> 2. Collapsed to begin by
                                               //     ai_action_manager_clear_all.
    /* +0x0c */ ai_action  **jobs_cap_end;     // [H] one past the last ALLOCATED slot; the realloc
                                               //     path in ai_action_manager_add_job writes it.
    /* ---- list 2: IDLES — std::vector<ai_action*> --------------------------------- */
    /* +0x10 */ allocator_t   idles_alloc;     // [H]
    /* +0x14 */ ai_action  **idles_begin;      // [H] PS2 AddIdle / NewIdle / AdoptIdles / NextIdle /
                                               //     hasIdle all target this list.
    /* +0x18 */ ai_action  **idles_end;        // [H]
    /* +0x1c */ ai_action  **idles_cap_end;    // [H]
    /* ---- list 3: MODIFIERS — std::vector<ai_action*> ----------------------------- */
    /* +0x20 */ allocator_t   modifiers_alloc; // [H]
    /* +0x24 */ ai_action  **modifiers_begin;  // [H] PS2 AddModifier / NewModifier / AdoptModifiers /
                                               //     NextModifier all target this list.
    /* +0x28 */ ai_action  **modifiers_end;    // [H]
    /* +0x2c */ ai_action  **modifiers_cap_end;// [H]
    /* ---- tail ---------------------------------------------------------------------- */
    /* +0x30 */ uint32_t      uid_counter;     // [H] the monotonically increasing action-UID source.
                                               //     ai_action_generate_UID post-increments it and the
                                               //     result becomes the new action's uid (action+0x04);
                                               //     every parent/child link in the three lists is a
                                               //     comparison against one of these. Zeroed by
                                               //     ai_action_manager_clear_all only on a HARD reset.
} ai_action_manager;                           // 0x34 bytes. There is NO fourth list - see §1b.
```

**Each list is one MSVC `std::vector`, 0x10 bytes: `{allocator, begin, end, cap_end}`.** That is what
produces the 0x10 stride between the three `begin` pointers, and it is why `+0x00/+0x10/+0x20` are not
mysterious (allocator members) and `+0x0c/+0x1c/+0x2c` are not padding (capacity pointers). The
declaration order is **jobs, idles, modifiers**.

### 1a. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 / +0x10 / +0x20 | the three `std::allocator` members | H | - | - | empty class, padded to 4 bytes; explains the 0x10 list stride |
| +0x04 / +0x08 / +0x0c | jobs begin/end/cap | H | `ai_action_manager_clear_all`, `ai_action_manager_new_job`, `ai_action_manager_add_job`, `ai_action_manager_derive_job` | `ai_action_manager_has_action`, `ai_action_manager_get_base_job`, `ai_action_manager_get_top_job`, `ai_action_manager_get_job_by_type`, `ai_action_manager_retire_top_job`, `entity_get_action_man_size` | `+0x04` = `C_entity+0x1168` |
| +0x14 / +0x18 / +0x1c | idles begin/end/cap | H | `ai_action_manager_clear_all`, `ai_action_manager_new_idle`, `ai_action_manager_add_idle`, `ai_action_manager_adopt_idles`, `ai_action_manager_retire_top_idle` | `ai_action_manager_has_action`, `ai_action_manager_has_idle_linked_to`, `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children` | stride 4 |
| +0x24 / +0x28 / +0x2c | modifiers begin/end/cap | H | `ai_action_manager_clear_all`, `ai_action_manager_new_modifier`, `ai_action_manager_add_modifier`, `ai_action_manager_adopt_modifiers`, `ai_action_manager_retire_top_modifier` | `ai_action_manager_has_action`, `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children` | stride 4 |
| +0x30 | uid_counter | H | `ai_action_generate_UID` (post-increment), `ai_action_manager_clear_all` (zeroed only when arg2 ≠ 0) | every `ai_action_new_*` factory (stores the result at `action+0x04`) | PS2 `+0x24` |
| +0x34 | *(nothing — end of object)* | H | - | - | the claimed 4th list is refuted, see §1b |

**Element type:** each slot holds an `ai_action*` — a heap-allocated, polymorphic record whose **type
id sits at offset 0** of a **0x24-byte header** (proven by `entity_get_action_size` switching on it).
The 35-value type vocabulary lives in **`structs/entity.md` §6b** and is deliberately not duplicated
here.

### 1b. Why the list identities are certain, and why there is no fourth list

Eleven independent proofs:

1. **Declaration order.** PS2's `__vector_pod` is 0x0c bytes, so its three lists sit at `+0x00`
   (jobs), `+0x0c` (idles) and `+0x18` (modifiers) — read straight off `AddJob`, `AddIdle` and
   `AddModifier`. PC's MSVC vector is 0x10 bytes, so the same three members in the same declaration
   order land at `+0x00`, `+0x10`, `+0x20`, with `begin` at `+0x04`, `+0x14`, `+0x24`.
2–6. **Intra-body sweep order, five times over.** `ai_action_manager_clear_all` ↔ `Clear`,
   `..._retire_top_job` ↔ `NextJob`, `..._has_action` ↔ `hasAction`, `..._signal_children` ↔
   `SignalToChilds` and `..._invalidate_actor` ↔ `Invalidate` each walk more than one list in a fixed
   order, and every one of them matches PS2's jobs → idles → modifiers against PC's `+0x04` → `+0x14`
   → `+0x24`. (`hasAction` and `SignalToChilds` walk modifiers before idles on *both* builds — the
   orders agree even where they are not alphabetical.)
7–9. **The three `Add*`/`New*` pairs** are byte-identical to each other apart from the base offset,
   and each PC base matches its PS2 sibling's list.
10. **The two `Adopt*` twins** carry the same distinctive `else i = -1` early break and target
    `+0x24`/`+0x14` exactly as PS2 targets modifiers/idles.
11. **A call-count match in a confirmed caller pair:** inside `entity_spawn_police_response` ↔
    `SpawnPolice`, `ai_action_manager_add_modifier` is called ×2 against `AddModifier` ×2.

**There is no fourth list.** `ai_action_manager_clear_all` ↔ `Clear` byte-pins the counter at PC
`+0x30` ↔ PS2 `+0x24`, so PS2's object ends at `0x28` and PC's at `0x34`: no PS2 method touches
anything past its counter, and no PC member touches `+0x34`. A list at `+0x34/+0x38` with stride
0x14, walked backwards by `ai_group_process_events`, does not belong to this object — whatever that
function walks, its receiver is something else.

## 2. The `ai_action` type table (35 types)

This is the class's real payload and it is fully mapped — the PC and PS2 factory sets join **1:1 on
the type id**. The table is maintained in **`structs/entity.md` §6b**, because `entity_action_tick`
switches on exactly these ids and `entity_get_action_size` / `entity_get_target` independently
cross-validate them. It is not duplicated here, to keep the two copies from drifting.

Summary: this is the NPC AI action vocabulary — what an AI-driven person in Mafia can be told to do.
Only types **0x21** (`_police_action`) and **0x28** (`_police_call`) are police-specific; the **cover
system** is 0x18–0x1c.

## 3. Method family — PC ↔ PS2 (58 methods)

### 3a. The 35 factories

Every PS2 `New*__17ai_action_manager` has a PC `ai_action_new_*` counterpart, joined on the action's
type id (`structs/entity.md` §6b): the factory stamps the id at `*outer`, the id set equals the case
set of `entity_action_tick`, and the join is 1:1 with no leftovers on either side. The join is by
type id and inner-payload size rather than 35 separate body-diffs (§6).

PS2 factories, in emission order: `NewMove`, `NewFollow`, `NewTurnTo`, `NewFire`, `NewFollowLink`,
`NewBlockedWay`, `NewRunawayDist`, `NewSideLook`, `NewBackLook`, `NewUnLink`, `NewRunCloserTo`,
`NewFreeze`, `NewMoveToCar`, `NewStepAway`, `NewStrafe`, `NewJump`, `NewCornerLook`,
`NewCornerCoverFire`, `NewCornerFire`, `NewCornerTakePos`, `NewCornerFight`, `NewCrouchFight`,
`NewNormalFight`, `NewBarricadeFight`, `NewFollowInRail`, `NewPoliceAction`, `NewSearch`,
`NewKickDriver`, `NewHostileCoverage`, `NewCarJump`, `NewCarStop`, `NewCarAim`, `NewPoliceCall`,
`NewStrafeHim`, `NewStrafeBeware`.

### 3b. The 23 manager members

Ordered by **PC VA** with the **PC name first**. Every row is body-diffed on both builds.

Two forced 1:1 order-align gaps carry the table, and the `GetJob(FUi)` anchor lands at **rank 17 of
18 on both sides**, which is what makes the tail cluster safe to work as a block:

| cluster | PC range | PS2 range | shape |
|---|---|---|---|
| head | `0x0042ef70`-`0x0042f4f0` | `0x0033a8c0`-`0x0033b690` | 5 ↔ 5 |
| tail | `0x004313d0`-`0x00432450` | `0x0033df80`-`0x0033ea40` | 18 ↔ 18, anchored at rank 17 |

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role / decisive evidence |
|---|---|---|---|---|---|
| 0x0042ef70 | `ai_action_manager_clear_all` | `Clear__17ai_action_managerFb` | 0x0033a8c0 | CONFIRMED | Dispose all three lists, collapse, then `if (hard_reset) uid_counter = 0`. Sweep order jobs → idles → modifiers on both. **Pins PS2 `+0x24` to PC `+0x30`**, hence the 0x34 object size |
| 0x0042f060 | `ai_action_generate_UID` | `GenUID__17ai_action_managerFv` | 0x0033a9b0 | CONFIRMED | Post-increments `+0x30`, returns it in EAX. Called by all 35 factories. A UID source, never decremented |
| 0x0042f070 | `ai_action_destroy_action` | `DestroyAction__17ai_action_managerFP9ai_action` | 0x0033a9d0 | CONFIRMED | Same per-type switch on `action+0x00`. The ×3.4 size gap is factoring: PS2 inlines the whole `ai_path` teardown in every case arm, PC calls two shared helpers |
| 0x0042f2f0 | `ai_action_invalidate_path_for_actor` | `InvalidatePath__17ai_action_managerFP9ai_actionP7C_actor` | 0x0033b3a0 | CONFIRMED | Switches on the type id to reach the `ai_path*` in the inner payload, then scrubs the dying actor out of it. **Case sets identical 15/15**: {0,1,8,0xa,0x10,0x11,0x14,0x18,0x1e,0x21,0x22,0x23,0x26,0x27,0x28} |
| 0x0042f4f0 | `ai_action_manager_invalidate_actor` | `Invalidate__17ai_action_managerFP7C_actor` | 0x0033b690 | CONFIRMED | The dead-actor path: calls the row above on every action in all three lists. Three **forward** loops — unique in this class |
| 0x004313d0 | `ai_action_manager_add_job` | `AddJob__17ai_action_managerFP9ai_action` | 0x0033df80 | CONFIRMED | `push_back` onto jobs |
| 0x00431580 | `ai_action_manager_new_job` | `NewJob__17ai_action_managerFP9ai_action` | 0x0033dfd0 | CONFIRMED | `Clear(this,false)` **then** push onto jobs — same soft-clear argument on both |
| 0x00431730 | `ai_action_manager_add_modifier` | `AddModifier__17ai_action_managerFP9ai_action` | 0x0033e030 | CONFIRMED | `add_job` rebased to `+0x24` |
| 0x004318e0 | `ai_action_manager_new_modifier` | `NewModifier__17ai_action_managerFP9ai_action` | 0x0033e080 | CONFIRMED | Backwards dispose-all over the modifiers, collapse, single push |
| 0x00431a90 | `ai_action_manager_add_idle` | `AddIdle__17ai_action_managerFP9ai_action` | 0x0033e110 | CONFIRMED | `add_job` rebased to `+0x14` |
| 0x00431c40 | `ai_action_manager_new_idle` | `NewIdle__17ai_action_managerFP9ai_action` | 0x0033e160 | CONFIRMED | Sibling of `new_modifier` on the idles list |
| 0x00431df0 | `ai_action_manager_adopt_modifiers` | `AdoptModifiers__17ai_action_managerFUiUi` | 0x0033e1f0 | CONFIRMED | Re-parents modifiers: rewrite `elem+0x0c` where `elem+0x08 == old_uid`. Both builds carry the same `else i = -1` early break |
| 0x00431e30 | `ai_action_manager_adopt_idles` | `AdoptIdles__17ai_action_managerFUiUi` | 0x0033e250 | CONFIRMED | Identical, on the idles list |
| 0x00431e70 | `ai_action_manager_derive_job` | `DeriveJob__17ai_action_managerFP9ai_action` | 0x0033e2b0 | CONFIRMED | New job inherits the top job's children: adopt modifiers, adopt idles, link parent, `AddJob`. **Both builds write `new+0x0c` outside the null guard** — an original-source bug |
| 0x00431ec0 | `ai_action_manager_get_top_job` | `GetJob__17ai_action_managerFv` | 0x0033e330 | CONFIRMED | Last element of the jobs list, or 0. The `Fv` overload |
| 0x00431ee0 | `ai_action_manager_get_base_job` | `GetBaseJob__17ai_action_managerFv` | 0x0033e360 | CONFIRMED | Walks jobs top-down to the first with `parent(+0x0c) == 0` — the ROOT of the derivation chain, not a pending task |
| 0x00431f20 | `ai_action_manager_retire_top_job` | `NextJob__17ai_action_managerFv` | 0x0033e3d0 | CONFIRMED | Cascade-removes child idles then child modifiers keyed on the retiring job's uid, then disposes the job. Both builds duplicate the cascade in both count arms |
| 0x004321e0 | `ai_action_manager_retire_top_modifier` | `NextModifier__17ai_action_managerFv` | 0x0033e730 | CONFIRMED | Dispose last modifier + erase; the `count < 2` arm collapses instead |
| 0x00432260 | `ai_action_manager_retire_top_idle` | `NextIdle__17ai_action_managerFv` | 0x0033e7c0 | CONFIRMED | Byte-identical sibling of the row above on `+0x14` |
| 0x004322e0 | `ai_action_manager_has_idle_linked_to` | `hasIdle__17ai_action_managerFP9ai_action` | 0x0033e850 | CONFIRMED | Scans idles for `arg+0x04 == elem+0x08` **or** `== elem+0x0c`. Not a plain membership test |
| 0x00432330 | `ai_action_manager_has_action` | `hasAction__17ai_action_managerFUiPP9ai_actionb` | 0x0033e8c0 | CONFIRMED | Three buckets in the order jobs → modifiers → idles, optional out-pointer, 4th arg = "jobs only" |
| 0x00432410 | `ai_action_manager_get_job_by_type` | `GetJob__17ai_action_managerFUi` | 0x0033e9e0 | CONFIRMED | The id-taking overload; arrived as a corollary of the `entity_is_in_danger` / `IsInDanger` diff (shared type ids 0x1e / 0x1d) |
| 0x00432450 | `ai_action_manager_signal_children` | `SignalToChilds__17ai_action_managerFUiUi` | 0x0033ea40 | CONFIRMED | ORs a bitmask into `elem+0x18` of every modifier then every idle whose parent uid matches. Both share the same fused `while` condition. The key is a parent uid, not an actor |

**No PS2-only members and no unlocated members.** All 23 have PC twins.

### 3c. Related functions outside this class

* `entity_action_tick` ↔ `actionTick__8C_entity` and `entity_actions_tick` ↔ `actionsTick` are both
  CONFIRMED (`structs/entity.md` §6). PS2's `actionTick_2` is a source split with no PC counterpart.
* `ai_fight_modes_enact_active_mode` (`structs/ai_fight_modes.md`) is one of this class's callers —
  `get_base_job`, `new_job` and two factories.
* `ai_action_destroy_action` cases 0/1/0x0a call `ai_path_destroy` then `ai_path_free_buffers` on the
  pointer at `inner+0x1c`/`+0x10`, where PS2's `DestroyAction` calls `Destroy__7ai_pathFv` followed
  by the three `gif::vector` dtors and `__dl__7ai_path`. That is the entry point into the `ai_path`
  class (`structs/ai_path.md`).

## 4. PC vs PS2

* **Object base:** PC `C_entity+0x1164` ↔ PS2 `C_entity+0x1348`, delta `0x1e4` — one of the
  documented `C_entity` band deltas, and it lands on `+0x1164` exactly.
* **Object size:** PC `0x34`, PS2 `0x28`. The whole difference is the vector representation: MSVC's
  `std::vector` is 0x10 bytes (allocator member included), PS2's `__vector_pod` is 0x0c. The counter
  is PC `+0x30` ↔ PS2 `+0x24`.
* **Allocation:** PS2 allocates `ai_action` records through `ai_spageman`, a PS2-only pool allocator
  every PS2 factory uses; PC uses plain `operator_new`.
* **The PS2 factory signatures carry type information the PC side cannot show** —
  `NewCornerTakePos(C_actor*, ai_seg_corner*, uint)`, `NewCornerFight(C_actor*,
  ai_corner_hide_info*, uint)`, `NewStrafeBeware(ai_fire_area*, uint, uint)`,
  `NewHostileCoverage(C_actor*, uint, vector<C_actor*>*, uint)`, and the `ai_path*` parameter on the
  movement factories. Those name four otherwise-anonymous structs: `ai_seg_corner` and
  `ai_corner_hide_info` (`structs/check_seg.md` §1f / §1g), `ai_fire_area` (`structs/entity.md` §6)
  and `ai_path` (`structs/ai_path.md`).

## 5. Related

- `structs/entity.md` §6b — the 35-type action table (authoritative; not duplicated here); §6c — the
  `C_entity` AI sub-object map. The manager is one of that map's sub-objects, alongside `ai_vis_logs`
  (`+0xb04`), `ai_sensors` (`+0xbb4`), `ai_logs` (`+0xe88`), `ai_character` (`+0xebc`),
  `ai_fight_modes` (`+0xf64`) and `ai_ghost` (`+0x126c`).
- `structs/ai_fight_modes.md` — sibling class and a caller.
- `structs/ai_path.md` — the path object the movement actions carry.
- `structs/check_seg.md` — `ai_seg_corner` and `ai_corner_hide_info`, the cover-system payloads the
  corner factories take.

## 6. Open items

| item | state |
|---|---|
| the 35 factory twins | joined as a closed 1:1 set by type id and inner-payload size, not by 35 individual body-diffs; no row contradicts the join, and 16 of the 35 have no decompiled body available to diff |
