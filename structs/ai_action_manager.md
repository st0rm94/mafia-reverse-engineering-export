# `ai_action_manager` — the NPC action queue (what an AI person can be told to do)

**Created** 2026-07-16 21:10 +0200 · **Modified** 2026-11-16 09:20 +0100

`ai_action_manager` owns an NPC's **action queue**, embedded in `C_entity`. It is the largest AI class in the game
and the class the whole `ai_action` vocabulary hangs off: 35 factories that build a typed `ai_action`, three parallel
lists (jobs, idles, modifiers) they get filed into, and the machinery that advances and retires them.
`entity_action_tick` then switches on an action's type id to execute it.

| | |
|---|---|
| PC prefix | `ai_action_manager_` for the manager, `ai_action_new_` for the factories, `ai_action_` for operations on one action |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x34`, embedded at `C_entity + 0x1164` |
| PC constructor | - |
| PC translation unit | `[0x0040f700, 0x0043b000)`, the `C_entity` unit, hosted by `structs/entity.md` §8; this class is `[0x0042ef70, 0x004324d0)` inside it |
| PS2 class | `ai_action_manager` (tag `17ai_action_manager`), 58 methods = 35 factories + 23 members |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The object is embedded in `C_entity`: `entity_get_action_man_size`, `entity_save_action_man` and
`entity_load_action_man` read and write its vectors directly as `entity+0x1168` onwards, and `entity_ai` and
`entity_action_tick` pass `entity+0x1164` as the receiver.

```c
typedef struct ai_action_manager {            // 0x34 bytes, embedded at C_entity+0x1164
    /* +0x00 */ allocator_t  jobs_alloc;      // [H] MSVC's empty std::allocator member of the jobs vector, padded to 4
    /* +0x04 */ ai_action  **jobs_begin;      // [H] the jobs list, stride 4; the top job is the last element
    /* +0x08 */ ai_action  **jobs_end;        // [H] count = (end - begin) >> 2
    /* +0x0c */ ai_action  **jobs_cap;        // [H] one past the last allocated jobs slot
    /* +0x10 */ allocator_t  idles_alloc;     // [H] the empty allocator of the idles vector
    /* +0x14 */ ai_action  **idles_begin;     // [H] the idles list, stride 4
    /* +0x18 */ ai_action  **idles_end;       // [H] count = (end - begin) >> 2
    /* +0x1c */ ai_action  **idles_cap;       // [H] one past the last allocated idles slot
    /* +0x20 */ allocator_t  modifiers_alloc; // [H] the empty allocator of the modifiers vector
    /* +0x24 */ ai_action  **modifiers_begin; // [H] the modifiers list, stride 4
    /* +0x28 */ ai_action  **modifiers_end;   // [H] count = (end - begin) >> 2
    /* +0x2c */ ai_action  **modifiers_cap;   // [H] one past the last allocated modifiers slot
    /* +0x30 */ uint32_t     uid_counter;     // [H] the action-uid source: incremented once per factory; only ai_action_manager_clear_all resets it
} ai_action_manager;
```

**Three lists, sixteen bytes each.** Each list is one MSVC `std::vector`, `{allocator, begin, end, cap}`, which is why
the `begin` pointers lie `0x10` apart. There is no fourth list: no member touches anything past `+0x30`.

**The elements.** Each slot holds an `ai_action *`, a heap record with a `0x24`-byte header and a per-type inner payload
(`structs/entity.md` §6a). A child action names its owning job by uid in `+0x08` and its parent by uid in `+0x0c`, which
is what `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children` and the two adopt functions match on.

## 2. Field evidence

Cells come from a census of the class's functions, read against the bodies, plus every access to `entity+0x1164..0x1197`
in the decomp, which is where the three `entity_*_action_man` functions and `entity_actions_tick` appear.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | jobs_alloc | H | `entity_ctor` | - | the allocator byte, stored from a stack byte (`mov byte ptr [edi], dl`); no list method reads it |
| +0x4 | jobs_begin | H | `ai_action_manager_add_job`, `ai_action_manager_new_job`, `entity_ctor`, `entity_dtor` (by address, `vector_dispose_at_0x04`), `entity_load_action_man` | `ai_action_manager_add_job`, `ai_action_manager_clear_all`, `ai_action_manager_get_base_job`, `ai_action_manager_get_job_by_type`, `ai_action_manager_get_top_job`, `ai_action_manager_has_action`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_job`, `ai_action_manager_retire_top_job`, `entity_dtor` (by address, `vector_dispose_at_0x04`), `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | `entity+0x1168` |
| +0x8 | jobs_end | H | `ai_action_manager_add_job`, `ai_action_manager_clear_all`, `ai_action_manager_new_job`, `ai_action_manager_retire_top_job`, `entity_ctor`, `entity_dtor` (by address, `vector_dispose_at_0x04`), `entity_load_action_man` | `ai_action_manager_add_job`, `ai_action_manager_clear_all`, `ai_action_manager_get_base_job`, `ai_action_manager_get_job_by_type`, `ai_action_manager_get_top_job`, `ai_action_manager_has_action`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_job`, `ai_action_manager_retire_top_job`, `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | - |
| +0xc | jobs_cap | H | `ai_action_manager_add_job`, `ai_action_manager_new_job`, `entity_ctor`, `entity_dtor` (by address, `vector_dispose_at_0x04`) | `ai_action_manager_add_job`, `ai_action_manager_new_job`, `entity_load_action_man` | - |
| +0x10 | idles_alloc | H | `entity_ctor` (by address, `std_vector_construct_empty`) | - | the allocator byte; `ai_action_manager_add_idle`, `ai_action_manager_new_idle`, `ai_action_manager_retire_top_job` and `entity_dtor` take its address as the receiver of the vector helpers, none of which reads the byte |
| +0x14 | idles_begin | H | `ai_action_manager_add_idle`, `ai_action_manager_new_idle`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor`, `entity_load_action_man` | `ai_action_manager_add_idle`, `ai_action_manager_adopt_idles`, `ai_action_manager_clear_all`, `ai_action_manager_has_action`, `ai_action_manager_has_idle_linked_to`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_idle`, `ai_action_manager_retire_top_idle`, `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children`, `entity_actions_tick`, `entity_dtor`, `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | `entity+0x1178` |
| +0x18 | idles_end | H | `ai_action_manager_add_idle`, `ai_action_manager_clear_all`, `ai_action_manager_new_idle`, `ai_action_manager_retire_top_idle`, `ai_action_manager_retire_top_job`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor`, `entity_load_action_man` | `ai_action_manager_add_idle`, `ai_action_manager_adopt_idles`, `ai_action_manager_clear_all`, `ai_action_manager_has_action`, `ai_action_manager_has_idle_linked_to`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_idle`, `ai_action_manager_retire_top_idle`, `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children`, `entity_actions_tick`, `entity_dtor`, `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | - |
| +0x1c | idles_cap | H | `ai_action_manager_add_idle`, `ai_action_manager_new_idle`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor` | `ai_action_manager_add_idle`, `ai_action_manager_new_idle`, `entity_load_action_man` | - |
| +0x20 | modifiers_alloc | H | `entity_ctor` (by address, `std_vector_construct_empty`) | - | the allocator byte; `ai_action_manager_add_modifier`, `ai_action_manager_new_modifier`, `ai_action_manager_retire_top_job` and `entity_dtor` take its address as the receiver of the vector helpers, none of which reads the byte |
| +0x24 | modifiers_begin | H | `ai_action_manager_add_modifier`, `ai_action_manager_new_modifier`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor`, `entity_load_action_man` | `ai_action_manager_add_modifier`, `ai_action_manager_adopt_modifiers`, `ai_action_manager_clear_all`, `ai_action_manager_has_action`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_modifier`, `ai_action_manager_retire_top_job`, `ai_action_manager_retire_top_modifier`, `ai_action_manager_signal_children`, `entity_actions_tick`, `entity_dtor`, `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | `entity+0x1188` |
| +0x28 | modifiers_end | H | `ai_action_manager_add_modifier`, `ai_action_manager_clear_all`, `ai_action_manager_new_modifier`, `ai_action_manager_retire_top_job`, `ai_action_manager_retire_top_modifier`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor`, `entity_load_action_man` | `ai_action_manager_add_modifier`, `ai_action_manager_adopt_modifiers`, `ai_action_manager_clear_all`, `ai_action_manager_has_action`, `ai_action_manager_invalidate_actor`, `ai_action_manager_new_modifier`, `ai_action_manager_retire_top_job`, `ai_action_manager_retire_top_modifier`, `ai_action_manager_signal_children`, `entity_actions_tick`, `entity_dtor`, `entity_get_action_man_size`, `entity_load_action_man`, `entity_save_action_man` | - |
| +0x2c | modifiers_cap | H | `ai_action_manager_add_modifier`, `ai_action_manager_new_modifier`, `entity_ctor` (by address, `std_vector_construct_empty`), `entity_dtor` | `ai_action_manager_add_modifier`, `ai_action_manager_new_modifier`, `entity_load_action_man` | - |
| +0x30 | uid_counter | H | `ai_action_generate_UID`, `ai_action_manager_clear_all` | `ai_action_generate_UID` | `ai_action_manager_clear_all` zeroes it only when its argument is non-zero; each of the 35 factories stores the result at `action+0x04` |

## 3. Method family

58 of 58 PS2 methods mapped: 23 members and 35 factories. The factories join their PS2 twins 1:1 on the action's type id
(§7a). The type id and inner size in each factory's Role were read from its body or machine code; the role words are in
`structs/entity.md` §6.

### 3a. Construction & destruction

No row. The manager has no constructor or destructor of its own: it is built and torn down inside `C_entity`.

### 3b. Accessors

No row: `ai_action_manager_get_top_job` and its siblings all branch on the count, and `ai_action_generate_UID` both loads and stores.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0042ef70 | `ai_action_manager_clear_all` | `Clear__17ai_action_managerFb` | 0x0033a8c0 | CONFIRMED | Destroys every action of the three lists in the order jobs, idles, modifiers and collapses the lists; a non-zero argument also zeroes `uid_counter` |
| 0x0042f060 | `ai_action_generate_UID` | `GenUID__17ai_action_managerFv` | 0x0033a9b0 | CONFIRMED | Increments `uid_counter` and returns the new value in EAX, so the first uid is 1 and `+0x0c == 0` can mean no parent; called by all 35 factories |
| 0x0042f070 | `ai_action_destroy_action` | `DestroyAction__17ai_action_managerFP9ai_action` | 0x0033a9d0 | CONFIRMED | Per-type switch on `action+0x00` that frees the inner payload, including the `ai_path` of a movement action (`ai_path_destroy`, `ai_path_free_buffers`), then the action |
| 0x0042f2f0 | `ai_action_invalidate_path_for_actor` | `InvalidatePath__17ai_action_managerFP9ai_actionP7C_actor` | 0x0033b3a0 | CONFIRMED | Switches on the type id to reach the `ai_path *` in the inner payload and scrubs the dying actor out of it; 15 case labels: 0, 1, 8, 0xa, 0x10, 0x11, 0x14, 0x18, 0x1e, 0x21, 0x22, 0x23, 0x26, 0x27, 0x28 |
| 0x0042f4f0 | `ai_action_manager_invalidate_actor` | `Invalidate__17ai_action_managerFP7C_actor` | 0x0033b690 | CONFIRMED | The dead-actor path: calls `ai_action_invalidate_path_for_actor` on every action of the three lists, walking forward |
| 0x0042f570 | `ai_action_new_move` | `NewMove__17ai_action_managerFR8S_vectorUiUiUifP7ai_pathUi` | 0x0033b770 | CONFIRMED | type `0x00`, inner `0x20`: walk or run to a world position |
| 0x0042f620 | `ai_action_new_follow` | `NewFollow__17ai_action_managerFP7C_actorfUiUiP7ai_pathUi` | 0x0033b880 | CONFIRMED | type `0x01`, inner `0x14`: follow an actor |
| 0x0042f6b0 | `ai_action_new_turn_to` | `NewTurnTo__17ai_action_managerFP7C_actorUi` | 0x0033b980 | CONFIRMED | type `0x05`, inner `0x04`: rotate to face an actor |
| 0x0042f710 | `ai_action_new_fire` | `NewFire__17ai_action_managerFP7C_actorfUiUi` | 0x0033ba20 | CONFIRMED | type `0x06`, inner `0x0c`: shoot at an actor |
| 0x0042f7b0 | `ai_action_new_follow_link` | `NewFollowLink__17ai_action_managerFP7C_humanUiUifP7ai_pathUi` | 0x0033bb00 | CONFIRMED | type `0x08`, inner `0x18`: follow along a path link |
| 0x0042f850 | `ai_action_new_blocked_way` | `NewBlockedWay__17ai_action_managerFUiUiUi` | 0x0033bc00 | CONFIRMED | type `0x09`, inner `0x08`: react to a blocked route |
| 0x0042f8c0 | `ai_action_new_runaway_dist` | `NewRunawayDist__17ai_action_managerFP7C_actorfUiUiUiP7ai_pathUi` | 0x0033bcc0 | CONFIRMED | type `0x0a`, inner `0x20`: flee until a distance is reached |
| 0x0042f980 | `ai_action_new_side_look` | `NewSideLook__17ai_action_managerFR8S_vectorfUiUi` | 0x0033bdf0 | CONFIRMED | type `0x0b`, inner `0x18`: glance sideways |
| 0x0042fa10 | `ai_action_new_back_look` | `NewBackLook__17ai_action_managerFR8S_vectorUibUi` | 0x0033bec0 | CONFIRMED | type `0x0c`, inner `0x20`: glance behind |
| 0x0042fbb0 | `ai_action_new_unlink` | `NewUnLink__17ai_action_managerFUiUi` | 0x0033c060 | CONFIRMED | type `0x0d`, inner `0x04`: detach from a path link |
| 0x0042fc20 | `ai_action_new_run_closer_to` | `NewRunCloserTo__17ai_action_managerFP7C_actorffUiUiP7ai_pathUi` | 0x0033c110 | CONFIRMED | type `0x10`, inner `0x24`: approach an actor |
| 0x0042fd00 | `ai_action_new_freeze` | `NewFreeze__17ai_action_managerFUiUiUi` | 0x0033c250 | CONFIRMED | type `0x12`, inner `0x04`: stand still |
| 0x0042fd70 | `ai_action_new_move_to_car` | `NewMoveToCar__17ai_action_managerFP5C_carffUiUiUiP7ai_pathUi` | 0x0033c310 | CONFIRMED | type `0x14`, inner `0x28`: go to a car, the boarding path |
| 0x0042fe10 | `ai_action_new_step_away` | `NewStepAway__17ai_action_managerFP7C_actorUi` | 0x0033c430 | CONFIRMED | type `0x15`, inner `0x14`: step aside |
| 0x0042fe70 | `ai_action_new_strafe` | `NewStrafe__17ai_action_managerFP7C_actorUiUifUi` | 0x0033c4d0 | CONFIRMED | type `0x16`, inner `0x1c`: strafe |
| 0x0042ff30 | `ai_action_new_jump` | `NewJump__17ai_action_managerFUiUi` | 0x0033c5e0 | CONFIRMED | type `0x17`, inner `0x04`: jump |
| 0x0042ffa0 | `ai_action_new_corner_look` | `NewCornerLook__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi` | 0x0033c690 | CONFIRMED | type `0x19`, inner `0x10`: cover: peek from cover |
| 0x00430020 | `ai_action_new_corner_cover_fire` | `NewCornerCoverFire__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi` | 0x0033c770 | CONFIRMED | type `0x1a`, inner `0x10`: cover: suppressing fire from cover |
| 0x004300a0 | `ai_action_new_corner_fire` | `NewCornerFire__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi` | 0x0033c850 | CONFIRMED | type `0x1b`, inner `0x10`: cover: fire from cover |
| 0x00430120 | `ai_action_new_corner_take_pos` | `NewCornerTakePos__17ai_action_managerFP7C_actorP13ai_seg_cornerUi` | 0x0033c930 | CONFIRMED | type `0x18`, inner `0x40`: cover: take a cover position |
| 0x004301a0 | `ai_action_new_corner_fight` | `NewCornerFight__17ai_action_managerFP7C_actorP19ai_corner_hide_infoUi` | 0x0033c9f0 | CONFIRMED | type `0x1c`, inner `0x48`: cover: the full fight from cover |
| 0x00430290 | `ai_action_new_crouch_fight` | `NewCrouchFight__17ai_action_managerFP7C_actorP7C_actorfUiUi` | 0x0033cb70 | CONFIRMED | type `0x1d`, inner `0x40`: fight while crouched |
| 0x00430440 | `ai_action_new_normal_fight` | `NewNormalFight__17ai_action_managerFP7C_actorP7C_actorfUiUi` | 0x0033cd90 | CONFIRMED | type `0x1e`, inner `0x58`: the default stand-up firefight |
| 0x00430590 | `ai_action_new_barricade_fight` | `NewBarricadeFight__17ai_action_managerFP7C_actorP7C_actorffUi` | 0x0033cf70 | CONFIRMED | type `0x1f`, inner `0x1c`: fight from behind a barricade |
| 0x00430650 | `ai_action_new_follow_in_rail` | `NewFollowInRail__17ai_action_managerFP7C_humanP7C_humanUi` | 0x0033d070 | CONFIRMED | type `0x20`, inner `0x14`: follow a human along a rail or track |
| 0x00430710 | `ai_action_new_police_action` | `NewPoliceAction__17ai_action_managerFP7C_actorUifUiUi` | 0x0033d160 | CONFIRMED | type `0x21`, inner `0x60`: police: the fine or arrest action |
| 0x00430810 | `ai_action_new_search` | `NewSearch__17ai_action_managerFP7C_actorfUiUi` | 0x0033d2c0 | CONFIRMED | type `0x22`, inner `0x34`: search for a lost target |
| 0x004308e0 | `ai_action_new_kick_driver` | `NewKickDriver__17ai_action_managerFP7C_humanffUiP7ai_pathUi` | 0x0033d3d0 | CONFIRMED | type `0x23`, inner `0x28`: the carjack eject |
| 0x004309d0 | `ai_action_new_hostile_coverage` | `NewHostileCoverage__17ai_action_managerFP7C_actorUiPQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>Ui` | 0x0033d520 | CONFIRMED | type `0x24`, inner `0x84`: takes a vector of hostile actors |
| 0x00430cc0 | `ai_action_new_car_jump` | `NewCarJump__17ai_action_managerFUiUi` | 0x0033d820 | CONFIRMED | type `0x25`, inner `0x10`: jump out of the way of a car |
| 0x00430d30 | `ai_action_new_car_stop` | `NewCarStop__17ai_action_managerFP7C_actorP5C_carUiR8S_vectorUi` | 0x0033d8d0 | CONFIRMED | type `0x26`, inner `0x20`: make a car stop |
| 0x00430dc0 | `ai_action_new_car_aim` | `NewCarAim__17ai_action_managerFP7C_actorP5C_carUiR8S_vectorUi` | 0x0033d9b0 | CONFIRMED | type `0x27`, inner `0x20`: aim at or from a car |
| 0x00430e50 | `ai_action_new_police_call` | `NewPoliceCall__17ai_action_managerFUiUiUiP7ai_pathUi` | 0x0033da90 | CONFIRMED | type `0x28`, inner `0x30`: police: radio for backup |
| 0x004312d0 | `ai_action_new_strafe_him` | `NewStrafeHim__17ai_action_managerFP7C_humanUiUi` | 0x0033dde0 | CONFIRMED | type `0x29`, inner `0x0c`: strafe relative to a human |
| 0x00431350 | `ai_action_new_strafe_beware` | `NewStrafeBeware__17ai_action_managerFP12ai_fire_areaUiUi` | 0x0033deb0 | CONFIRMED | type `0x2a`, inner `0x30`: strafe while avoiding a friendly line of fire |
| 0x004313d0 | `ai_action_manager_add_job` | `AddJob__17ai_action_managerFP9ai_action` | 0x0033df80 | CONFIRMED | `push_back` onto jobs |
| 0x00431580 | `ai_action_manager_new_job` | `NewJob__17ai_action_managerFP9ai_action` | 0x0033dfd0 | CONFIRMED | `ai_action_manager_clear_all(this, 0)`, then a push onto jobs |
| 0x00431730 | `ai_action_manager_add_modifier` | `AddModifier__17ai_action_managerFP9ai_action` | 0x0033e030 | CONFIRMED | `push_back` onto modifiers |
| 0x004318e0 | `ai_action_manager_new_modifier` | `NewModifier__17ai_action_managerFP9ai_action` | 0x0033e080 | CONFIRMED | Destroys every modifier walking backwards, collapses the list, pushes one |
| 0x00431a90 | `ai_action_manager_add_idle` | `AddIdle__17ai_action_managerFP9ai_action` | 0x0033e110 | CONFIRMED | `push_back` onto idles |
| 0x00431c40 | `ai_action_manager_new_idle` | `NewIdle__17ai_action_managerFP9ai_action` | 0x0033e160 | CONFIRMED | Destroys every idle walking backwards, collapses the list, pushes one |
| 0x00431df0 | `ai_action_manager_adopt_modifiers` | `AdoptModifiers__17ai_action_managerFUiUi` | 0x0033e1f0 | CONFIRMED | Walks modifiers backwards: while `elem+0x08 == old_uid` writes `elem+0x0c = new_uid`; the first mismatch ends the walk |
| 0x00431e30 | `ai_action_manager_adopt_idles` | `AdoptIdles__17ai_action_managerFUiUi` | 0x0033e250 | CONFIRMED | The same on idles |
| 0x00431e70 | `ai_action_manager_derive_job` | `DeriveJob__17ai_action_managerFP9ai_action` | 0x0033e2b0 | CONFIRMED | The new job inherits the top job's children: adopts modifiers and idles, sets `new+0x0c` to the top job's uid outside the null test, then `ai_action_manager_add_job` |
| 0x00431ec0 | `ai_action_manager_get_top_job` | `GetJob__17ai_action_managerFv` | 0x0033e330 | CONFIRMED | Last element of jobs, or 0 |
| 0x00431ee0 | `ai_action_manager_get_base_job` | `GetBaseJob__17ai_action_managerFv` | 0x0033e360 | CONFIRMED | Walks jobs from the top to the first with `+0x0c == 0`: the root of the derivation chain |
| 0x00431f20 | `ai_action_manager_retire_top_job` | `NextJob__17ai_action_managerFv` | 0x0033e3d0 | CONFIRMED | Removes the child idles and child modifiers whose `+0x08` is the retiring job's uid, then disposes the job; the count-below-2 arm and the other arm each hold their own copy of the cascade |
| 0x004321e0 | `ai_action_manager_retire_top_modifier` | `NextModifier__17ai_action_managerFv` | 0x0033e730 | CONFIRMED | Disposes the last modifier and erases it; the count-below-2 arm collapses instead |
| 0x00432260 | `ai_action_manager_retire_top_idle` | `NextIdle__17ai_action_managerFv` | 0x0033e7c0 | CONFIRMED | The same on idles |
| 0x004322e0 | `ai_action_manager_has_idle_linked_to` | `hasIdle__17ai_action_managerFP9ai_action` | 0x0033e850 | CONFIRMED | Scans idles for `arg+0x04 == elem+0x08` or `== elem+0x0c`; not a plain membership test |
| 0x00432330 | `ai_action_manager_has_action` | `hasAction__17ai_action_managerFUiPP9ai_actionb` | 0x0033e8c0 | CONFIRMED | Searches jobs, modifiers, idles in that order for a type id; optional out-pointer; the fourth argument limits the search to jobs |
| 0x00432410 | `ai_action_manager_get_job_by_type` | `GetJob__17ai_action_managerFUi` | 0x0033e9e0 | CONFIRMED | The job nearest the top with the given type id, or 0 |
| 0x00432450 | `ai_action_manager_signal_children` | `SignalToChilds__17ai_action_managerFUiUi` | 0x0033ea40 | CONFIRMED | ORs a bitmask into `elem+0x18` of the trailing modifiers, then idles, whose `+0x08` is the given uid; the first mismatch ends each walk |

### 3d. Free helpers

No row. The class's callees inside the `C_entity` unit are methods of other classes (`ai_sensors`, `ai_space_coverage`,
`ai_talkman`, `ai_fight_modes`, `ai_path`) and of `C_entity`; the unit's free helpers are hosted by `structs/entity.md`.

## 4. The `ai_action` type table

The table is kept in `structs/entity.md` §6, because `entity_action_tick` switches on exactly these ids and
`entity_get_action_size` and `entity_get_target` read them too. This is the NPC action vocabulary. Only types **0x21**
(`_police_action`) and **0x28** (`_police_call`) are police-specific; the **cover system** is `0x18..0x1c`.

## 5. Related functions outside this class

* `entity_action_tick` and `entity_actions_tick` execute the actions (`structs/entity.md` §3).
* `ai_fight_modes_enact_active_mode` (`structs/ai_fight_modes.md`) calls `ai_action_manager_get_base_job`,
  `ai_action_manager_new_job` and two factories.
* `ai_action_destroy_action` cases 0, 1 and `0xa` call `ai_path_destroy` then `ai_path_free_buffers` on the pointer at
  `inner+0x1c` / `+0x10`, the entry into `structs/ai_path.md`.
* The factory parameters of `ai_action_new_corner_take_pos`, `ai_action_new_corner_fight`, `ai_action_new_strafe_beware`
  and the movement factories are typed by `ai_seg_corner`, `ai_corner_hide_info` (`structs/check_seg.md` §1d / §1e),
  `ai_fire_area` (`structs/entity.md` §1a) and `ai_path`.

## 6. The two link fields of a child action

`ai_action_manager_retire_top_job` removes children by `+0x08`, and `ai_action_manager_adopt_modifiers` /
`_adopt_idles` move only `+0x0c`, so after a derive a child still names its original job in `+0x08` and the new top job in
`+0x0c`. Every factory stores its uid argument in both. `ai_action_manager_derive_job` writes `new+0x0c` from the top
job with no null test, so with an empty jobs list it reads through a null pointer.

## 7. PC vs PS2

* **Object base:** PC `C_entity+0x1164`, PS2 `C_entity+0x1348`, delta `0x1e4`.
* **Object size:** PC `0x34`, PS2 `0x28`. The whole difference is the vector: MSVC's `std::vector` is `0x10` bytes,
  PS2's `__vector_pod` `0x0c`. The counter is PC `+0x30`, PS2 `+0x24`.
* **Allocation:** every PS2 factory allocates through `ai_spageman`, a PS2-only pool; the PC uses `operator_new`.
* **The inner payload** of a PS2 action is equal to the PC's, or larger by its 16-byte `S_vector`s.
* **PS2 signatures type what the PC cannot show:** `NewCornerTakePos(C_actor*, ai_seg_corner*, uint)`,
  `NewCornerFight(C_actor*, ai_corner_hide_info*, uint)`, `NewStrafeBeware(ai_fire_area*, uint, uint)`,
  `NewHostileCoverage(C_actor*, uint, vector<C_actor*>*, uint)`, and the `ai_path*` parameter of the movement factories.
* **No PS2-only method and no unlocated member.**

### 7a. How the twins were established

**The factories.** The PS2 `New*` set and the PC `ai_action_new_*` set join on the type id with no leftover on either
side, and the id set equals the case set of `entity_action_tick`. PS2 emission order: `NewMove`, `NewFollow`, `NewTurnTo`,
`NewFire`, `NewFollowLink`, `NewBlockedWay`, `NewRunawayDist`, `NewSideLook`, `NewBackLook`, `NewUnLink`, `NewRunCloserTo`,
`NewFreeze`, `NewMoveToCar`, `NewStepAway`, `NewStrafe`, `NewJump`, `NewCornerLook`, `NewCornerCoverFire`, `NewCornerFire`,
`NewCornerTakePos`, `NewCornerFight`, `NewCrouchFight`, `NewNormalFight`, `NewBarricadeFight`, `NewFollowInRail`,
`NewPoliceAction`, `NewSearch`, `NewKickDriver`, `NewHostileCoverage`, `NewCarJump`, `NewCarStop`, `NewCarAim`,
`NewPoliceCall`, `NewStrafeHim`, `NewStrafeBeware`.

**The members.** Two forced 1:1 order-align gaps carry them, and the `GetJob(FUi)` anchor lands at rank 17 of 18 on both
sides:

| Cluster | PC range | PS2 range | Shape |
|---|---|---|---|
| head | `0x0042ef70`..`0x0042f4f0` | `0x0033a8c0`..`0x0033b690` | 5 to 5 |
| tail | `0x004313d0`..`0x00432450` | `0x0033df80`..`0x0033ea40` | 18 to 18, anchored at rank 17 |

**The list identities.** PS2's three lists sit at `+0x00`, `+0x0c`, `+0x18` (read off `AddJob`, `AddIdle`, `AddModifier`), and
the same three members in the same order land at PC `+0x00`, `+0x10`, `+0x20`. `Clear`, `NextJob`, `Invalidate`,
`hasAction` and `SignalToChilds` each walk more than one list in a set order, and each order matches its PC twin
against `+0x04`, `+0x14`, `+0x24`. `hasAction` and `SignalToChilds` walk modifiers before idles on both builds. The two
`Adopt*` twins carry the same `else i = -1` early break. `entity_spawn_police_response` and `SpawnPolice` call
`ai_action_manager_add_modifier` and `AddModifier` twice each. `Clear` pins PC `+0x30` to PS2 `+0x24`.

**Per-member notes.** `DestroyAction`'s ×3.4 size gap is factoring: the PS2 inlines the whole `ai_path` teardown in each
arm, the PC calls two helpers. `InvalidatePath` has the same 15 case labels on both. `DeriveJob` writes `new+0x0c` outside
the null test on both builds. `NextJob` duplicates the cascade in both count arms on both builds.

## 8. Related

- `structs/entity.md` §6 — the action type table; §6a — the `ai_action` header; §7 — the `C_entity` AI sub-objects.
- `structs/ai_fight_modes.md` — a caller.
- `structs/ai_path.md` — the path object the movement actions carry.
- `structs/check_seg.md` — `ai_seg_corner` and `ai_corner_hide_info`.

## 9. Open items

None.
