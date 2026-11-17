# `C_entity` — the scriptable-NPC actor and its AI brain

**Created** 2026-07-10 22:46 +0200 · **Modified** 2026-11-09 10:34 +0100

`C_entity` derives from `C_human` (kind `0x1b` over `C_human`'s 3). It is the scriptable-NPC
**AI/perception brain**: it owns a `C_program` (the script VM), perception logs / sensors /
attention, target selection, weapon-choice AI, the action-tick state machine, and the per-subsystem
SaveGame chain.

| | |
|---|---|
| PC prefix | `entity_` |
| PC kind (`+0x10`) | `0x1b`, stamped by `entity_ctor` over `C_human`'s 3 (`structs/entity_kind.md` §1) |
| PC vtable | `g_entity_vt` @ `0x0063b488`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` virtual order; the `C_program` base-subobject vtable @ `0x0063b45c` has 10 slots (§4) |
| PC base class | `C_human` (`structs/human.md`); the `C_program` subobject sits at `+0xa9c` (§4) |
| PC instance size | `0x128c` |
| PC constructor | `entity_ctor` @ `0x004114d0` |
| PC translation unit | `[0x0040f700, 0x0043b000)` (§8) |
| PS2 class | `C_entity` (tag `8C_entity`), 110 methods: 102 in §3, 8 with no PC function in §9 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

C_entity-specific fields; the object also carries the full `C_actor` + `C_human` bands
(`structs/actor.md`, `structs/human.md`). PC offsets. The PS2 offsets and the band deltas are in §9.
`+0x78` is `C_human`'s `act_state` (`structs/human.md`), which `entity_clear_act_state`,
`entity_set_act_state_shock`, `entity_force_ai` and `entity_force_script` reach from this class.

```c
typedef struct C_entity {                   // extends C_human; PC offsets, C_entity band only
    /* +0xa9c */ struct C_program program;  // [H] THE SCRIPT VM sub-object. See §4.
    /* +0xafc */ float    ai_priority;      // [H] = program+0x60, set by the script's setaipriority (§2)
    // ---- THE SAVEGAME SUB-BLOCKS ---------------------------------------------------------
    /* +0xb04 */ struct ai_vis_logs vis_logs;  // [H] the ai_vis_logs sub-object base, ticked by entity_ai
    /* +0xb14 */ /* vis_logs list */        // [H] = vis_logs + 0x10; serialized size constant 0x184
    /* +0xbb4 */ struct ai_sensors sensors; // [H] the ai_sensors sub-object base (structs/ai_sensors.md)
    /* +0xbc0 */ /* sensors serialized */   // [H] = sensors + 0xc, where serialization begins; constant 0x3a0
    /* +0xc74 */ struct { uint32_t t; uint32_t type; } heard_events[]; // [H] the sound-contact ring, stride 0x20; type 2 = gunfire
    /* +0xe88 */ struct ai_logs   ai_logs;  // [H] the ai_logs sub-object base, ticked by entity_ai
    /* +0xeac */ /* ai_logs list */         // [H] = ai_logs + 0x24; serialized size variable (a list)
    /* +0xebc */ struct ai_character character; // [H] the ai_character sub-object base, 0x28 bytes (structs/ai_character.md)
    // ---- SCRIPT-COMMAND BAND (+0xee4..+0xf1b) ---------------------------------------------
    /* +0xee4 */ float    script_look_target[3]; // [H] the point LOOKTO / LOOK / ENTER_DOOR turn the head to (§2)
    /* +0xef0 */ float    script_move_target[3]; // [H] the point USE_RAILWAY / EXIT_RAILWAY walk to; the script route's destination
    /* +0xefc */ float    railway_exit_dir[3];   // [H] the station entry's XZ direction from EXIT_RAILWAY; no reader but the savegame (class census)
    /* +0xf08 */ uint32_t entity_flags;     // [H] the script/AI flag word, also reached as program+0x46c; bits in §2b
    /* +0xf0c */ uint32_t script_cmd_step;  // [H] the running script command's progress word and step counter (§2)
    /* +0xf10 */ uint32_t script_move_mode; // [H] entity_script_tick's mode: 0 head turn, 1 follow script_route
    /* +0xf14 */ uint32_t script_replan_ms; // [H] no new script route before this time, ms
    /* +0xf18 */ C_actor *script_train;     // [H] the train USE_RAILWAY boards and EXIT_RAILWAY leaves
    /* +0xf1c */ struct ai_modifier anger;  // [H] {value, rate_per_ms, remaining_ms}, stride 0xc
    /* +0xf28 */ struct ai_modifier fear;   // [H] same triple
    /* +0xf34 */ struct ai_modifier threat; // [H] same triple
    /* +0xf40 */ uint32_t ai_state;         // [H] the entity's AI state enum; the value 4 is special-cased (§2)
    /* +0xf44 */ uint32_t ai_substate;      // [H] the sub-state inside ai_state; in AI state 0, 1 = the AI phase holds the entity
    /* +0xf48 */ uint32_t flags;            // [H] flag word: bit 3, bit 0x10 in a line of fire, bit 0x20 a shooter's warning (§2)
    /* +0xf4c */ uint32_t human_type_group; // [H] set once by entity_game_init from human_type (+0x5fc); groups in §2c
    /* +0xf50 */ uint32_t ai_flags;         // [H] bit 0 lets a group-1 entity in AI state 4 refuse entity_force_script, bit 1 suspends the state switcher
    /* +0xf54 */ uint32_t police_session_id; // [H] the police squad session the entity serves (= squad +0x30)
    /* +0xf58 */ float    ai_demand;        // [H] the AI phase's claim against ai_priority (entity_script_switch, §2)
    /* +0xf5c */ float    force_range;      // [H] the forced action's range, the fifth argument of entity_force_ai
    /* +0xf60 */ uint32_t cross_fire_next_ms; // [H] the next entity_beware_cross_fire check, ms
    /* +0xf64 */ struct ai_fight_modes fight_modes; // [H] the ai_fight_modes sub-object base, 6 slots of 0x34 (structs/ai_fight_modes.md)
    // ---- THE SUBSTATE-LOCK TABLE (+0x10b4..+0x10c8) -----------------------------------------
    /* +0x10b4 */ uint8_t  lock_vec_alloc;  // [H] the lock vector's allocator byte, stored by entity_ctor
    /* +0x10b8 */ struct ai_substate_lock *lock_vec_begin; // [H] vector<ai_substate_lock>, stride 0xc, rows {key, p1 (uint), p2 (float)}
    /* +0x10bc */ struct ai_substate_lock *lock_vec_end;   // [H] one past the last lock row; (end - begin) / 0xc is the row count
    /* +0x10c0 */ struct ai_substate_lock *lock_vec_cap;   // [H] the end of the allocated rows
    /* +0x10c4 */ uint32_t locked_state;     // [H] the locked state; 0x29a (= 666) means no lock
    /* +0x10c8 */ uint32_t lock_aux;         // [H] write-only: entity_lock_substate's aux; only the savegame reads it
    /* +0x10cc */ struct ai_talkman talk;    // [H] the NPC speech scheduler, 0x98 bytes (structs/ai_talkman.md)
    /* +0x1164 */ struct ai_action_manager actions; // [H] the action-queue sub-object (structs/ai_action_manager.md)
    /* +0x1198 */ uint8_t  bc_alloc;        // [H] the blocked-con vector's allocator byte; its address is the vector
    /* +0x119c */ struct blocked_con *bc_begin; // [H] std::vector<blocked_con> (head +0x1198), stride 0x10: nav-graph edges the entity may not use
    /* +0x11a0 */ struct blocked_con *bc_end;   // [H] one past the last blocked connection; (end - begin) >> 4 is the count
    /* +0x11a4 */ struct blocked_con *bc_cap;   // [H] the end of the allocated elements
    /* +0x11a8 */ uint32_t action_tick_flags; // [H] the flags word entity_actions_tick hands to entity_action_tick (§2)
    /* +0x11ac */ uint32_t ai_group_id;     // [H] the ai_group the entity belongs to, 0 = none
    /* +0x11b0 */ struct ai_path *script_route; // [H] the route entity_script_tick follows in script_move_mode 1
    // ---- HEAD / PERCEPTION-ORIGIN BAND (+0x11b4..+0x11cb) -----------------------------------
    /* +0x11b4 */ float    head_dir[3];     // [H] head/look DIRECTION (entity_get_head_dir)
    /* +0x11c0 */ float    head_pos[3];     // [H] head POSITION, the eye/ear point perception is measured from (entity_get_head_pos)
    /* +0x11cc */ struct I3D_frame *neck_frame; // [H] the model's child frame "Neck" (else "neck"); 0 when there is none
    // ---- WEAPON-AI BAND (+0x11d0..+0x11ec) --------------------------------------------------
    /* +0x11d0 */ float    weapon_def_param;  // [H] write-only copy of weapon_def+0x6c (§2)
    /* +0x11d4 */ float    weapon_max_range;  // [H] from weapon_def+0x74; entity_weapon_get_max_range
    /* +0x11d8 */ float    weapon_best_range; // [H] from weapon_def+0x70; entity_weapon_get_best_range
    /* +0x11dc */ float    weapon_spread;     // [H] from weapon_def+0x88; entity_weapon_get_spread
    /* +0x11e0 */ float    weapon_readiness;  // [H] 0->1 fraction; reset to 0 on weapon change; ramped by entity_weapon_choose
    /* +0x11e4 */ uint32_t weapon_ready_ms;   // [H] ms of the last readiness update; top nibble 0xf after a weapon switch (§2)
    /* +0x11e8 */ uint32_t weapon_state;      // [H] weapon-state FSM, values 1..6; state 4 raises weapon_is_armed
    /* +0x11ec */ uint8_t  weapon_is_armed;   // [H] 1 = a weapon is drawn; entity_weapon_is_armed
    // ---- FIRE TIMING (+0x11f0..+0x1207) -----------------------------------------------------
    /* +0x11f0 */ uint32_t fire_burst_ms;      // [H] the constant part of a burst, ms
    /* +0x11f4 */ uint32_t fire_pause_rand_ms; // [H] the random part of a pause, ms
    /* +0x11f8 */ uint32_t fire_pause_ms;      // [H] the constant part of a pause, ms
    /* +0x11fc */ uint32_t fire_burst_rand_ms; // [H] the random part of a burst, ms
    /* +0x1200 */ uint32_t fire_burst_end_ms;  // [H] the end of the current burst, ms
    /* +0x1204 */ uint32_t fire_pause_end_ms;  // [H] the end of the pause after it, ms
    /* +0x1208 */ struct ai_fire_area fire_area; // [H] 0x28 bytes, filled and tested per candidate shooter by entity_beware_cross_fire (§1a)
    /* +0x1230 */ struct S_using_object use_record; // [H] 0x38 bytes: the entity's own usable-object record (structs/using_object.md §1a)
    // ---- ai_ghost COLLISION EXEMPTIONS (+0x1268) --------------------------------------------
    /* +0x1268 */ struct ai_ghost ghost;    // [H] the object base (structs/ai_ghost.md); vector begin / end at +0x126c / +0x1270
    /* +0x127c */ void  *ai_target_fear;    // [H] best-by-fear target slot (ai_logs_get_max_fear)
    /* +0x1280 */ void  *ai_target_angry;   // [H] best-by-anger target slot (ai_logs_get_max_angry)
    /* +0x1284 */ void  *fight_party_target; // [H] the target whose fight party entity_ai joined
    /* +0x1288 */ uint32_t fight_party_ms;  // [H] ms of entity_ai's last fight-branch tick
} C_entity;                                 // partial - C_entity AI band only; instance 0x128c
```

**The bands.**

* **The savegame sub-blocks.** `C_entity`'s savegame is the `C_human` parent, the `C_program` and six
  sub-blocks. Each sub-block has a `Get<X>Size` / `Save<X>` / `Load<X>` triple, and
  `entity_save_game_get_size` sums them, which is how the whole chain is proven: a size function names
  its own members. Four are the objects at `+0xb04`, `+0xbb4`, `+0xe88` and `+0xebc`; "State" is the
  blocked-con vector at `+0x119c` / `+0x11a0` plus the substate-lock table, and "ActionMan" is the
  `ai_action_manager` at `+0x1164` (`entity_get_action_man_size` reads `+0x1168`).
* **The script-command band** is written by `entity_process_command`'s command arms as
  `program+0x448..+0x47c` and driven each frame by `entity_script_tick` (§2).
* **The head band** holds the two vectors perception is computed from, read by `ai_vis_logs_tick`
  together with `entity_get_sight_range`; `entity_ai` writes both each frame from `neck_frame`.
* **The weapon-AI band** is a cached copy of the held weapon's stats, refreshed by
  `entity_weapon_get_info` from the weapon-definition record, so the AI reads weapon parameters every
  tick without walking the item table.
* **The fire timing** is seeded by `entity_weapon_timing_setup` per held slot; `entity_do_shoot_timed`
  fires while now ≤ `fire_burst_end_ms`, holds until `fire_pause_end_ms`, then draws both ends again.
* **The `ai_ghost` collision exemptions.** The head is `{+0x00 allocator, +0x04 begin, +0x08 end,
  +0x0c cap, +0x10 changed-flag byte}`; the records are `ai_ghost_actor`, stride `0x1c`:
  `{vec3 pos; uint32 expiry @+0x0c; float drift_radius2 @+0x10; C_actor * @+0x14;
  tDynamicCollObject * @+0x18}`. `ai_path_tick` and `ai_path_revalidate_for_actor` ghost the blocker of
  the current detour waypoint, with kind-dependent expiry (500 / 800 / 2000 ms) and radius² (0.09 /
  0.25 / 1.0); `entity_tick_ghosts` walks the vector backwards once per frame, erases every record
  expired by time or drift, and rebuilds the collision ignore list at `C_human+0x698` from the
  survivors' colliders, so the entity's movement collision skips them.

The three emotion ramps at `+0xf1c` / `+0xf28` / `+0xf34` are three embedded **`ai_modifier`**
instances, stride `0xc`; there is no modifier *list*. `ai_modifier_tick` (`0x00425850`) does
`value += min(dt, remaining) * rate; remaining -= min(dt, remaining)`. `entity_ai` inlines the ANGER
ramp and calls `ai_modifier_tick` for the other two. The `ai_logs` target scorer multiplies its
derived anger (`log+0x4c`) by `+0xf1c`, its derived fear (`log+0x50`) by `+0xf28`, and scales its
threat term by `+0xf34` — so a script or event can push a temporary emotional multiplier with a decay
rate and a time budget, and it ramps back on its own.

**The array fields.** `heard_events[]` is the `sound[8]` array of the embedded `ai_sensors`: eight slots of
stride `0x20`, because `ai_sensors_init` and `ai_sensors_destroy` walk eight, each slot one heard sound source
(`structs/ai_sensors.md` §1). `head_dir[3]` and `head_pos[3]` are one 3D vector each, x, y, z, and so are
`shooter_pos[3]` and `aim_dir[3]` of `ai_fire_area`.

### 1a. `ai_fire_area` — the out-struct of the fire-metrics pair

`entity_get_fire_metrics` / `entity_get_player_fire_metrics` both FILL an `ai_fire_area`:

```c
typedef struct ai_fire_area {               // 0x28 bytes
    /* +0x00 */ float shooter_pos[3];        // [H] the shooter's position
    /* +0x0c */ float aim_dir[3];            // [H] the unit aim direction, shooter -> target
    /* +0x18 */ float range;                 // [H] engagement range
    /* +0x1c */ float dodge_cos;             // [H] the dodge cone, a cosine
    /* +0x20 */ float line_of_fire_cos;      // [H] the line-of-fire cone, a cosine
    /* +0x24 */ float measured_cos;          // [H] the measured cosine
} ai_fire_area;
```

The area lives at `C_entity+0x1208`, filled and tested by `entity_beware_cross_fire` for each candidate
shooter it walks. Inside the `+0x20` cone and within range the entity marks itself in the line of fire
(`+0xf48` bit `0x10`); only when the measured cosine also exceeds `+0x1c` does it queue
`ai_action_new_strafe_beware`, which copies the whole `0x28`-byte area into its payload. A cosine of
0 makes the cone a half-space, so for a melee weapon anyone ahead and in range counts.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xa9c | **`C_program` sub-object** | H | `entity_game_init` (the `C_program` init), `entity_duplicate_from` (`program_duplicate_from`) | `entity_death` (calls its vtable through `*(this+0xa9c)`), `entity_script_is_fake` (`program+0x10`/`+0x14`), `entity_get_program` (returns `program+0xc`) | the embedded script VM (§4); `entity_process_command` recovers the entity from it as `lea ecx,[ebp-0xa9c]` |
| +0xafc | ai_priority | H | `program_ctor`, `program_init`, `program_set_ai_priority`, `entity_script_is_fake` (clears) | `entity_force_ai`, `entity_script_switch` | = `program+0x60` (`structs/program.md` §1); the script's `setaipriority xx` stores `(100 - xx) * 0.01`, so a high xx gives a low value. `entity_force_ai` bails when ≥ 0.75; `entity_script_switch` compares it with `ai_demand` `+0xf58` |
| +0xb04 | **`ai_vis_logs` sub-object** | H | `ai_vis_logs_tick` | `entity_ai` | the OBJECT BASE (`lea ecx,[esi+0xb04]` at all 19 `entity_ai` callsites); `+0xb14` is its internal list. Address accessor `entity_get_ai_vis_logs` (`0x00425b70`) |
| +0xb14 | vis_logs (list) | H | `entity_load_vis_logs`, the `ai_vis_logs_*` methods | `entity_save_vis_logs`, `entity_can_safe_shoot` | = `ai_vis_logs + 0x10`, slot 0's actor. Serialized size = constant 0x184; `entity_save_vis_logs` also touches `+0xba4` |
| +0xbb4 | **`ai_sensors` sub-object** | H | `ai_sensors_new_sound_contact` | `sound_arena_update_entity_sound_contacts` | the OBJECT BASE (`lea ecx,[ebp+0xbb4]`). The three-sense contact system: `structs/ai_sensors.md`. Address accessor `entity_get_ai_sensors` (`0x00425b80`) |
| +0xbc0 | sensors (serialized start) | H | `entity_load_sensors` | `entity_save_sensors` | = `ai_sensors + 0xc` = `visual[0].actor`, where serialization begins — **not** the object base. Serialized size = constant 0x3a0 |
| +0xc74 | **heard_events[]** (= `ai_sensors + 0xc0`) | H | `ai_sensors_new_sound_contact` (`0x0040faf0`), driven by `sound_arena_update_entity_sound_contacts`; killed by `ai_sensors_cancel_sound_contact` | `entity_has_heard_gunfire` | ring of sound CONTACTS, stride 0x20: `entry[0]` = timestamp, `entry[1]` = type, **type 2 = GUNFIRE**; `entity_has_heard_gunfire` looks for a type-2 entry still inside its deadline (`t + window > now`). Audibility is gated by the NPC's hearing stat (`intensity > 1.05 − entity_get_hearing_sensitivity()`), not a constant radius. Cancelling ZEROES the type field at `+0xc78`, which is what makes the type-2 scan miss it. Full chain: `structs/ai_sensors.md` §5 |
| +0xe88 | **`ai_logs` sub-object** | H | `ai_logs_tick` | `entity_ai` | the OBJECT BASE (`lea ecx,[esi+0xe88]` at all 19 `entity_ai` callsites); `+0xeac` is its internal list (`ai_logs + 0x24`). Address accessor `entity_get_ai_logs` (`0x00425b90`) |
| +0xeac | ai_logs (list) | H | `entity_ctor`, `entity_load_logs` (through the insert) | `entity_save_logs`, `entity_get_logs_size`, `entity_beware_cross_fire`, `ai_logs_get_max_*`, `ai_logs_get_log_for_actor` | the AI logs the AI scores targets from. Serialized size is VARIABLE (a list) |
| +0xebc | **`ai_character` sub-object** | H | `ai_character_init`; serialized by `entity_save_character` | all `ai_character_get_*`, `entity_get_sight_range`, `entity_get_hearing_sensitivity`; restored by `entity_load_character` | the OBJECT BASE (`add ecx,0xebc`), size 0x28. The NPC personality/stat block: `structs/ai_character.md`. Its serialized part `+0xebc..+0xec8` is the nine stat deltas. Serialized size = constant 0x24: `ai_character_init` zeroes exactly nine dwords (`+0x00..+0x20`) and stores the cfg backpointer at `+0x24`, which is a live pointer and is deliberately not saved |
| +0xee4 | script_look_target | H | `entity_game_init` (the normalised frame row `+0x70..+0x78`), `entity_load_state`, `entity_process_command` (`LOOKTO`: the web node's position; `LOOK`: the actor's frame position `+0x40..+0x48`, +1.5 in Y for kinds 2, 3, `0x1b`; `ENTER_DOOR` steps 0 and 2: the door's frame position) | `entity_script_tick` (passed to `human_look_to`), `entity_save_state` | the script's head-turn target, three floats; each command writer sets `entity_flags` bit `0x8` and clears `0x10`. Reached as `program+0x448` |
| +0xef0 | script_move_target | H | `entity_game_init` (0), `entity_process_command` (`USE_RAILWAY`: the outer entry from `railway_get_outer_entry_near`, at step 1 the own frame's `+0x80..+0x88`; `EXIT_RAILWAY`: the inner entry from `railway_get_inner_entry_near`) | `entity_script_tick` (the route destination of `ai_wayfind_plan_shortest_route`), `entity_process_command` (`USE_RAILWAY`, `EXIT_RAILWAY`: the squared-distance tests 0.25, 1.0, 0.04) | the script's move target point, three floats. Reached as `program+0x454` |
| +0xefc | railway_exit_dir | H | `entity_game_init` (0), `entity_load_state`, `entity_process_command` (`EXIT_RAILWAY`: the second out-argument of `railway_get_inner_entry_near`, the entry's normalised XZ direction) | `entity_save_state` | no reader but the savegame: searched over the class's methods, every access of the program band in `entity_process_command` (`program+0x460`) and the whole of `entity_action_tick` |
| +0xf08 | entity_flags | H | `entity_game_init`, `entity_script_tick`, `entity_script_switch`, `entity_force_script`, `entity_suspend_ai_and_hide`, `entity_resume_ai_and_show`; as `program+0x46c`: `entity_script_callback`, `entity_process_command` | `entity_ai`, `entity_action_tick`, `entity_script_tick`, `entity_script_switch`, `entity_force_ai`, `entity_force_script`, `entity_switch_ai_state`, `ai_sensors_get_smelled_target_info`, `entity_resume_ai_and_show`, `entity_process_command` | §2b |
| +0xf0c | script_cmd_step | H | `entity_game_init`, `entity_force_script` (0), `entity_process_command` | `entity_process_command` | the running command's progress word: 1 once a command has queued its action or dispatched its pursuit event, 0 when it ends; the step counter of `ENTER_DOOR` (0..4), `USE_RAILWAY` (0..5), `EXIT_RAILWAY` (1..3). `TALK` stores the speech channel here and `WAIT` passes it to `game_human_talk_del`. Reached as `program+0x470` |
| +0xf10 | script_move_mode | H | `entity_game_init`, `entity_script_tick` (0 when done), `entity_process_command` (`STOP` 0, `FINDCORNER` 1, `ENTER_DOOR` 1 with its route and 0 at the door, `USE_RAILWAY` 1 and 0 on arrival) | `entity_script_tick` | 0 = turn the head to `script_look_target`, 1 = follow `script_route`. Reached as `program+0x474` |
| +0xf14 | script_replan_ms | H | `entity_game_init`, `entity_script_tick` (now + 5500 after a failed route) | `entity_script_tick` | no new route is planned before this time, ms |
| +0xf18 | script_train | H | `entity_game_init`, `entity_load_state` (by frame name), `entity_process_command` (`USE_RAILWAY`: a kind-8 actor standing in a station within the parameter's squared range, 0 at its step 5; `EXIT_RAILWAY`: the vehicle link when its kind is 8 and it stands in a station, 0 at its step 3) | `entity_process_command` (`human_use_actor(+0xf18, 1)` in `USE_RAILWAY` step 0, `human_use_actor(+0xf18, 2)` in `EXIT_RAILWAY` step 2), `entity_save_state` | the train the script boards or leaves. Reached as `program+0x47c` |
| +0xf1c | **ANGER ramp** `{value, rate_per_ms, remaining_ms}` `+0xf1c` / `+0xf20` / `+0xf24` | H | `entity_game_init` (seeds 1.0 / 0 / 0), `entity_ai` (open-codes the advance on this slot), `ai_fight_modes_enact_active_mode` mode 5 (2.0, −0.0001 per ms, 10 000 ms) | the `ai_logs` target scorer, which multiplies its derived anger (`log+0x4c`) by `+0xf1c` | an embedded `ai_modifier` |
| +0xf28 | **FEAR ramp** `+0xf28` / `+0xf2c` / `+0xf30` | H | `entity_ai` via `ai_modifier_tick` | the scorer multiplies its derived fear (`log+0x50`) by `+0xf28` | an embedded `ai_modifier` |
| +0xf34 | **THREAT ramp** `+0xf34` / `+0xf38` / `+0xf3c` | H | `entity_ai` via `ai_modifier_tick`, `ai_vis_logs_tick` (2.0, −0.0001 per ms, 10 000 ms on a corpse it holds no log for) | the scorer scales its threat term by `+0xf34` | an embedded `ai_modifier`; `ai_modifier_tick` (`0x00425850`) is class-less, and these three slots are its only known users |
| +0xf40 | ai_state | H | `entity_force_ai` | `entity_force_script`, `entity_force_ai`, `entity_script_switch`, `entity_update`, `entity_process_command` (`GETSTATE`) | the AI state enum; value **4** is special-cased, and `entity_force_script` gates on `ai_state != 0 \|\| !force_if_idle`. `entity_update`: `== 3` keeps `fight_party_target`, `< 0x29a` gates the actor-invalidation loop. `GETSTATE` waits while the state is above 6, so its 666 / 10000 strip branches never run |
| +0xf44 | ai_substate | H | `entity_game_init`, `entity_reset`, `entity_reset_state`, `entity_force_ai` (its argument), `entity_force_script`, `entity_ai`, `entity_script_switch` (0 / 1), `entity_dispatch_event` (0, 4, 5, 8, 9) | `entity_ai`, `entity_script_switch`, `entity_force_ai`, `police_action_tick`, `police_action_dispatch_officer_commands` | the sub-state inside `ai_state`; in AI state 0, 1 means the AI phase holds the entity (`entity_script_switch`) |
| +0xf48 | flags | H | `entity_ai`, `entity_dispatch_event` (bit 3 on events `0x13`, `0xc`, cleared by `0x15`; bit `0x20` on event `0x1b`), `police_action_dispatch_officer_commands` (clears bit 3), `entity_beware_cross_fire` (bit `0x10`; clears `0x20`), `entity_force_ai` (the whole word, its fourth argument), `entity_game_init`, `entity_reset`, `entity_reset_state`, `entity_force_script` (0), `entity_death` (bit 3), `entity_action_tick` case `0x1f` (bit 3 once the action is 10 s old) | `police_manager_npc_list_tick` (bit 3), `entity_beware_cross_fire` (bit `0x20`), `entity_action_tick` cases `0x1e` (bit `0x10`: close-engage) and `0x1f` (bit 3), `entity_death` (bit 3) | flag word. Bit `0x10`: in a shooter's line of fire (§1a). Bit `0x20`: `entity_can_safe_shoot` sends event `0x1b` to a friendly in its line of fire |
| +0xf4c | human_type_group | H | `entity_game_init` (from `human_type` `+0x5fc`), `entity_load_state` | `entity_force_script`, `entity_force_ai`, `entity_switch_ai_state`, `entity_action_tick`, `entity_hit`, `entity_dispatch_event`, `ai_logs_event`, `ai_fight_manager_join_party` / `_leave_party`, `ai_fight_followship_take_car`, `police_board_suspend_npcs` / `_resume_npcs`; getter `entity_get_human_type_group` (`0x00425b60`) | a per-entity group, set once: `human_type` `0x2`→0, `0x4`→2, `0x8`→3, `0x10`→4, `0x20`→5, `0x40`→6, any other bit→1. Parties accept only members of one group; `ai_logs_event` switches on it for owner and target; `police_board_suspend_npcs` skips group 2. Tested `== 1` as the arrest-sound gate. Who is in each group: §2c |
| +0xf50 | ai_flags | H | `entity_game_init` / `entity_reset` / `entity_reset_state` (= 1), `entity_set_ai_state_resettable` (bit 0, script `HUMAN_RETURNFROMPANIC`), `entity_set_state_switcher_suspended` (bit 1); `VIDIM`, `SETSTATE`, `ARREST_PLAYER` (§5) | `entity_force_script` (bit 0), `entity_switch_ai_state` (bit 1) | only bits 0 and 1 exist. Bit 0: a group-1 entity in AI state 4 refuses `entity_force_script`. Bit 1: `entity_switch_ai_state` early-returns, freezing the AI state and the target slots `+0x127c` / `+0x1280` |
| +0xf54 | police_session_id | H | `entity_game_init`, `entity_ai` (= the squad's `+0x30`), `entity_spawn_police_response` | `entity_hit`, `entity_ai`, `entity_action_tick` (cases 6, `0x1c`, `0x1d`, `0x1e`, `0x21`) | the police squad session; with `human_type_group` 2, `entity_action_tick` finds the board group by it (`police_board_find_group_by_session_id`) |
| +0xf58 | ai_demand | H | `entity_force_ai` (1.0 on every call), `entity_actions_tick` (0 when the job queue is empty), `entity_game_init`, `entity_reset`, `entity_reset_state`, `entity_force_script` (0) | `entity_script_switch` | the AI phase's claim against `ai_priority`: in AI state 0, `ai_priority < ai_demand` hands the entity to the AI (only `0x1c000` of `entity_flags` kept, `program_leave_compact_block`, `ai_substate` 1); with `ai_substate` 1, `ai_demand < ai_priority` hands it back (`ai_substate` 0, `entity_flags \|= 0x400`) |
| +0xf5c | force_range | H | `entity_force_ai` (its fifth argument), the init functions (0) | `entity_ai` | the forced action's range: the `ai_action_new_search` radius (100.0 when ≤ 0), the `ai_action_new_crouch_fight` distance (5.0 or 6.0 when ≤ 0), the `ai_action_new_runaway_dist` distance |
| +0xf60 | cross_fire_next_ms | H | `entity_game_init` (0), `entity_beware_cross_fire` (now + 500) | `entity_beware_cross_fire` | the time of the next cross-fire check, ms |
| +0xf64 | **`ai_fight_modes` sub-object** | H | `ai_fight_modes_set_enemy`, `ai_fight_modes_reset`, `ai_fight_modes_apply_hit` | `ai_fight_modes_get_hit_stat`, `ai_fight_modes_consume_finish_request`, `entity_get_ai_fight_modes` (address accessor, `0x00425ba0`) | the OBJECT BASE, byte-proven two independent ways: `lea ecx,[esi+0xf64]` at four `set_enemy` / `reset` callsites, and `mov ecx,[edi+0x30]` + `add ecx,0xf64` at the `apply_hit` callsite inside `ai_logs_event` (`edi+0x30` is the `ai_logs` owner backpointer, i.e. this). The 6-slot combat-mode scorer (stride 0x34): `structs/ai_fight_modes.md` |
| +0x10b4 | lock_vec_alloc | H | `entity_ctor` (the byte) | - | the substate-lock vector's allocator byte; `entity_lock_substate` and `entity_load_state` take the vector's address here |
| +0x10b8 | lock_vec_begin — the **substate-lock vector** | H | `entity_lock_substate`, `entity_unlock_substate` | `entity_has_locked_substate`, `entity_get_state_size`, `entity_save_state`, `entity_load_state` | `std::vector<ai_substate_lock>`, stride 0xc, rows `{key, p1 (uint), p2 (float)}` (row type: §9). `entity_get_state_size` sums `nlocks * 0xc` into the savegame "State" block, which confirms the member independently |
| +0x10bc | lock_vec_end | H | `entity_lock_substate`, `entity_unlock_substate` | `entity_has_locked_substate`, `entity_get_state_size`, `entity_save_state`, `entity_load_state` | the end pointer of the vector at `+0x10b8` |
| +0x10c0 | lock_vec_cap | H | `entity_ctor`, `entity_dtor`, `entity_lock_substate`, `entity_load_state` (on growth) | `entity_lock_substate` | the capacity pointer of the vector at `+0x10b8`; `entity_lock_substate` reaches it as `[esi+0xc]` after `lea esi,[ecx+0x10b4]` |
| +0x10c4 | **locked_state** | H | `entity_lock_substate` (when its trailing bool is set) | `entity_has_locked_state` | **0x29a (= 666) = NO LOCK**, the project-wide "none" sentinel, in both int and float form (`ai_vis_logs_init` uses 0x44268000 = 666.0f). Generic substate locking, not pursuit-specific |
| +0x10c8 | lock_aux | H | `entity_lock_substate` (the `aux` param), `entity_game_init`, `BRAINWASH` | the savegame pair only | write-only: no game logic reads it |
| +0x10cc | `ai_talkman` sub-object | H | `ai_talkman_init` (from `entity_game_init`, owner = the entity), `ai_talkman_say` | `ai_talkman_tick`, `ai_talkman_begin_frame` (both from `entity_ai`) | the speech scheduler, 0x98 bytes, ending at `+0x1164`; `lea ecx,[reg+0x10cc]` at 22 callsites: `structs/ai_talkman.md` |
| +0x1164 | **`ai_action_manager` sub-object** | H | `ai_action_manager_clear_all`, `ai_action_manager_new_job` | `ai_action_manager_get_base_job`, `ai_action_manager_has_action`, `entity_get_action_man_size` | the OBJECT BASE (`lea ecx,[reg+0x1164]` at 12+ callsites). Three `ai_action*` lists — jobs `+0x04`, idles `+0x14`, modifiers `+0x24` — plus the action-UID counter at `+0x30`: `structs/ai_action_manager.md` |
| +0x1198 | bc_alloc | H | `entity_ctor` (the byte) | - | the blocked-con vector's allocator byte; its address is the vector `entity_script_tick` and `entity_action_tick` (×22) hand to `ai_wayfind_set_blocked_cons`, and `entity_add_blocked_con` takes it too |
| +0x119c | bc_begin — the **`vector<blocked_con>`** | H | `entity_add_blocked_con`, `entity_load_state` (restores), `entity_game_done` (clears) | the route planner, installed with the vector head `+0x1198` by `entity_action_tick` (×22) and `entity_script_tick`; `entity_get_state_size`, `entity_save_state` | blocked connections — nav-graph edges the entity may not use (element type: §9). Stride 0x10; `row[2]` carries the 0x29a (= 666) sentinel |
| +0x11a0 | bc_end | H | `entity_add_blocked_con`, `entity_load_state` (restores), `entity_game_done` (clears) | the route planner, installed with the vector head `+0x1198` by `entity_action_tick` (×22) and `entity_script_tick`; `entity_get_state_size`, `entity_save_state` | the end pointer of the vector at `+0x119c` |
| +0x11a4 | bc_cap | H | `entity_ctor`, `entity_dtor`, `entity_add_blocked_con` (on growth) | `entity_add_blocked_con` (the growth test) | the capacity pointer of the vector at `+0x119c` |
| +0x11a8 | action_tick_flags | H | `entity_game_init`, `entity_reset` (0), `entity_load_state`, `entity_actions_tick`, `entity_action_tick` (through its fifth argument) | `entity_actions_tick`, `entity_action_tick` (the prologue's bit-4 test), `entity_save_state` | `entity_actions_tick` passes its address as `entity_action_tick`'s last argument. Bit `0x1`: set before the top job's tick, cleared by the arms still turning or moving; when it stays set the driver runs `human_do_breath`. Bit `0x2`: cleared before the tick, never set or read. Bit `0x4`: set by cases `0x19`..`0x1b` while their corner animation starts and runs; the prologue then skips `entity_is_tickable`, and the driver sets `0x8`. At entry a set `0x8` clears `0x4` and `0x8` |
| +0x11ac | ai_group_id | H | `entity_game_init`, `entity_reset`, `ai_group_add_member`, `entity_process_command` (`GROUP_DEL`: 0 on the removed member) | `entity_force_ai`, `entity_dispatch_event`, `ai_fight_manager_join_party`, `ai_fight_followship_set_commander`, `ai_fight_followship_take_car`, `ai_fight_manager_tick`, `police_board_create_and_engage`, `entity_action_tick` (case `0x1e`: `ai_group_send_sniff_event`) | the `ai_group` the entity belongs to, 0 = none |
| +0x11b0 | script_route | H | `entity_game_init` (0), `entity_script_tick` (builds and frees it), `entity_process_command` (`FINDCORNER`, `ENTER_DOOR`, `USE_RAILWAY` free it and build a new one) | `entity_script_tick` (×21), `entity_process_command` | an `ai_path *`, followed while `script_move_mode` is 1. Reached as `program+0x714` |
| +0x11b4 | head_dir | H | `entity_ai` (each frame: the neck frame's forward row `+0x30..+0x38`, normalised, else (0, 0, 1)), `entity_load_state` (restores the 12 bytes) | `entity_get_head_dir`, `entity_save_state` | the head/look direction; `ai_vis_logs_tick` reads it through `entity_get_head_dir` together with `entity_get_sight_range` |
| +0x11c0 | head_pos | H | `entity_ai` (each frame: the neck frame's world position `+0x40..+0x48`, else the body frame's) | `entity_get_head_pos` | the eye/ear point perception is measured from |
| +0x11cc | neck_frame | H | `entity_rebind_neck_frame` (the model's child `"Neck"`, else `"neck"`; 0 when neither is found), `entity_game_init` | `entity_ai`, `entity_action_tick` (cases `0xb`, `0xc`, `0x24`: its world position and forward) | an `I3D_frame *`, looked up again after a model swap; case `0x24` uses it with no null test |
| +0x11d0 | weapon_def_param | H | `entity_weapon_get_info`, `entity_weapon_choose` | the savegame pair only | a write-only copy of `predmety.def` `+0x6c` for the held weapon: 2 for every melee item, 3 Colt Detective Special, 4 Colt 1911, 5 the other guns, 0 the Mosin-Nagant and every non-weapon |
| +0x11d4 | weapon_max_range | H | `entity_weapon_get_info`, `entity_weapon_choose` | `entity_weapon_get_max_range` | from weapon_def+0x74. Consumers compute `radius² − dist² > 0` ⇒ target in range, so it stores a LINEAR range, squared at the callsite |
| +0x11d8 | weapon_best_range | H | `entity_weapon_get_info`, `entity_weapon_choose` | `entity_weapon_get_best_range` | from weapon_def+0x70 |
| +0x11dc | weapon_spread | H | `entity_weapon_get_info`, `entity_weapon_choose` | `entity_weapon_get_spread`, `entity_can_safe_shoot` | from weapon_def+0x88; feeds the safe-shot cone `cos(spread*0.0002546479 + 0.17)` |
| +0x11e0 | weapon_readiness | H | `entity_weapon_choose`, `entity_weapon_get_info`, `entity_game_init`, `entity_reset`, `entity_equip_weapon_by_id` | `entity_weapon_get_readiness`, `entity_action_tick` (cases `0x1d`, `0x1e`; through the getter cases 6, `0x1c`, `0x1e`) | reset to 0 on weapon change; readers gate on `≤ 0.9999` = "not fully readied". `entity_weapon_choose` zeroes it while selecting, then writes an intermediate ramp float and finally `0x3f800000` (1.0f) once the choice settles. Above 0.98, cases `0x1d` and `0x1e` call `ai_talkman_say` |
| +0x11e4 | weapon_ready_ms | H | `entity_game_init`, `entity_reset` (0), `entity_weapon_choose`, `entity_equip_weapon_by_id` | `entity_weapon_choose`, `entity_equip_weapon_by_id` | low 28 bits: ms of the last readiness update (a 500 ms gate); top nibble `0xf`: set on a weapon switch, cleared once the held slot `+0x1e8` is 1..3. While the nibble is clear, a substate-3 lock with p1 5..7 is released |
| +0x11e8 | weapon_state | H | `entity_equip_weapon_by_id`, `entity_weapon_choose` (3, 4, 6), `entity_game_init` | `entity_weapon_choose` | weapon-state FSM, values 1..6; reaching state 4 raises `weapon_is_armed` |
| +0x11ec | weapon_is_armed | H | `entity_weapon_get_info`, `entity_equip_weapon_by_id`, `entity_weapon_choose`, `entity_game_init`, `entity_reset` | `entity_weapon_is_armed` | byte. Set iff weapon slot `+0x1e8` ∈ 1..3. It has no vision input |
| +0x11f0 | fire_burst_ms | H | `entity_game_init`, `entity_weapon_timing_setup` (by the held slot `+0x1e8`: 1 → 600, 2 → 2000, 3 with weapon 10 → 2000, 3 → 1000, else 2500) | `entity_do_shoot_timed` | `fire_burst_end_ms = now + fire_burst_ms + rand(fire_burst_rand_ms)` |
| +0x11f4 | fire_pause_rand_ms | H | `entity_game_init`, `entity_weapon_timing_setup` (1000 / 1000 / 750 / 500 / 1500, the same slot order) | `entity_do_shoot_timed` | the random part of the pause |
| +0x11f8 | fire_pause_ms | H | `entity_game_init`, `entity_weapon_timing_setup` (1000 / 2500 / 1500 / 2000 / 1000) | `entity_do_shoot_timed` | `fire_pause_end_ms = fire_burst_end_ms + fire_pause_ms + rand(fire_pause_rand_ms)` |
| +0x11fc | fire_burst_rand_ms | H | `entity_game_init`, `entity_weapon_timing_setup` (1500 / 3000 / 2000 / 3000 / 2000) | `entity_do_shoot_timed` | the random part of the burst |
| +0x1200 | fire_burst_end_ms | H | `entity_game_init`, `entity_do_shoot_timed` | `entity_do_shoot_timed` | the entity shoots while now ≤ it |
| +0x1204 | fire_pause_end_ms | H | `entity_game_init`, `entity_do_shoot_timed` | `entity_do_shoot_timed` | no shot until now passes it; then both ends are drawn again |
| +0x1208 | fire_area | H | `entity_beware_cross_fire` | `entity_beware_cross_fire` | an `ai_fire_area` (§1a), filled for each candidate shooter it walks; the strafe action it queues copies the whole area |
| +0x1230 | use_record | H | `entity_ctor`, `entity_game_init` (position (0, 1, 0) through the frame matrix, `radius_sq` 0.7, flags 0, owner this, `item_id` 1, `action_code` from event 60000), `entity_add_using_object` | `entity_add_using_object`, `human_change_state` (`using_object_del_object`, behind kind `0x1b`), `entity_game_done`, `entity_unregister_using_object` | an embedded `S_using_object`, `0x38` bytes (`structs/using_object.md` §1a): the entity's own record in the usable-object registry; `+0x123c`..`+0x125c` and `+0x1264` are its members |
| +0x1268 | **`ai_ghost` (collision exemptions)** | H | `ai_ghost_add_or_refresh`, `ai_ghost_add_human_colliders` | `entity_tick_ghosts`, `ai_ghost_invalidate_actor` | the object base; `+0x126c` / `+0x1270` are the vector's begin / end pointers. Records are `ai_ghost_actor`, stride **0x1c**. Written by `ai_path_tick` / `ai_path_revalidate_for_actor` for the blocker of the current detour waypoint, with kind-dependent expiry (500/800/2000 ms) and radius² (0.09/0.25/1.0); expired per frame by time OR drift. Their colliders form the collision ignore list at `C_human+0x698`. `structs/ai_ghost.md`. The head spans `+0x1268..+0x1278` |
| +0x127c | ai_target_fear | H | `entity_switch_ai_state`, `entity_release_target`, `entity_game_init` | `entity_ai`, `entity_force_ai`, `entity_release_target` | per-tick best-target pick by fear: `entity_switch_ai_state` stores what `ai_logs_get_max_fear` returns |
| +0x1280 | ai_target_angry | H | `entity_switch_ai_state`, `entity_release_target`, `entity_game_init` | `entity_ai`, `entity_force_ai`, `entity_switch_ai_state`, `entity_release_target` | per-tick best-target pick by anger: `entity_switch_ai_state` stores what `ai_logs_get_max_angry` returns |
| +0x1284 | fight_party_target | H | `entity_game_init`, `entity_ai` (= `ai_target_angry` after joining that target's fight party), `entity_update` (0 unless the AI state is 3) | `entity_ai`, `entity_update` | the target whose fight party the entity joined |
| +0x1288 | fight_party_ms | H | `entity_game_init` (now), `entity_ai` (now, in its fight branch) | `entity_update` | older than now means the fight branch did not run this frame; `entity_update` then leaves a party that is gone or whose `+0x30 != 1` |

### 2a. `ai_fire_area`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | shooter_pos | H | `entity_is_in_entity_fire_area`, `entity_is_in_player_fire_area` (on a hit) | `ai_action_new_strafe_beware` (copies the whole area) | three floats |
| +0x0c | aim_dir | H | `entity_is_in_entity_fire_area`, `entity_is_in_player_fire_area` | `ai_action_new_strafe_beware` (copies the whole area) | the unit aim direction, shooter → target |
| +0x18 | range | H | `entity_get_fire_metrics`, `entity_get_player_fire_metrics` | `ai_action_new_strafe_beware` (copies the whole area) | ← `entity_weapon_get_max_range()`. When the player holds no weapon, `entity_get_player_fire_metrics` defaults it to **1.5f** |
| +0x1c | dodge_cos | H | `entity_get_fire_metrics`, `entity_get_player_fire_metrics`, `entity_is_in_entity_fire_area`, `entity_is_in_player_fire_area` (the 0.94 cap within 10 units) | `entity_beware_cross_fire` | `cos(spread × 0.0005093 + 0.17)`; capped at 0.94 within 10 units; 0 when the range is **< 4.0** |
| +0x20 | line_of_fire_cos | H | `entity_get_fire_metrics`, `entity_get_player_fire_metrics`, `entity_is_in_entity_fire_area`, `entity_is_in_player_fire_area` (the 0.86 cap within 10 units) | `entity_beware_cross_fire` | `cos(spread × 0.0002546 + 0.17)`; capped at 0.86 within 10 units; 0 when the range is **< 4.0** |
| +0x24 | measured_cos | H | `entity_is_in_entity_fire_area`, `entity_is_in_player_fire_area` | `entity_beware_cross_fire` | compared against the two cones |

### 2b. The flag word `+0xf08`

`+0xf08` is one word with three spellings. The `C_entity` methods address it as `+0xf08`; the five
`C_program` overrides (§4) run with `this` = the program subobject at `+0xa9c` and address it as
**`program+0x46c`** — `entity_script_callback` and every command arm of `entity_process_command`.

The script keywords below are the names the shipped scripts use (§5). Every bit with its writers
and readers:

| bit | set by | cleared by | read by — what it gates |
|---|---|---|---|
| `0x2` | `PLAYANIM` | `STOPANIM`, `STOP` | `entity_ai`: a script-started animation is playing |
| `0x4` | - | `STOP` | `entity_script_tick` passes it to `human_move_to` as the run flag; no code sets it, so a scripted walk never runs |
| `0x8` | `LOOKTO`, `LOOK`, `ENTER_DOOR` | `STOP`, `USE_RAILWAY`, `entity_script_tick` once the head turn ends | `entity_script_tick`: a head turn is pending |
| `0x10` | `entity_game_init`, `STOP`, `ENTER_DOOR`, `USE_RAILWAY`, `entity_script_tick` | `LOOKTO`, `LOOK`, `FINDCORNER`, `ENTER_DOOR`, `USE_RAILWAY` | `WAIT` blocks until it is set: the current command has finished |
| `0x20` | `TALK` | `WAIT`, once the speech channel is silent | `WAIT`: a line is being spoken |
| `0x80` | `ENTER_DOOR` | `ENTER_DOOR` | the same arm: a door passage is in progress |
| `0x100` | `USE_RAILWAY` | `USE_RAILWAY` | the same arm: boarding is in progress |
| `0x200` | `entity_script_callback` (callback 0, raised by `program_set_event`) | `entity_script_callback` (callback 1, with `0x400` / `0x800`) | `entity_ai`: the script program drives the entity (`program_call_process` + `entity_script_tick`) |
| `0x400` | `entity_script_switch`, while `0x200` is set | the 16 action commands ¹ | an action command finishes once it is set: the AI phase has handed control back |
| `0x800` | the 16 action commands ¹ | the 16 action commands ¹ | `entity_ai`: the action queue also runs while the script drives |
| `0x1000` | `ACTION_FOLLOW` | `entity_force_script` | it becomes `0x2000` when the AI interrupts the follow |
| `0x2000` | `entity_force_script` | `entity_process_command`'s prologue | the prologue skips the interrupted command when `0x200` is clear |
| `0x4000` | `PODVADIM_JAK` | the same command | `ai_sensors_get_smelled_target_info`: the target's live position instead of the cached scent |
| `0x8000` | `NAPRDELIVAUTE` | the same command | `ACTION_FIRE` passes it to its action |
| `0x10000` | `FOLLOWPLAYER` | the same command | `entity_action_tick`: the in-vehicle follow and seat hand-off block |
| `0x20000` | `entity_suspend_ai_and_hide` | `entity_resume_ai_and_show` | the hide latch the resume half undoes |
| `0x40000` | `AZ_PO_PANICE_HOVNO` | - | `entity_force_ai`: no other state is forced while the entity is in AI state 4 |
| `0x80000` | `NEVYHAZUJ` | - | `entity_action_tick`: no `ai_action_new_kick_driver`, the entity never carjacks |
| `0x100000` | `LOCKSTATE`, `IGNORE_CORPSE` | - | `entity_switch_ai_state`: no reaction to a corpse |
| `0x200000` | `BLASTFIRE` | - | `entity_action_tick`: aim at a target car's fuel tank (`car_get_fuel_tank_world_pos`) |

¹ The commands that queue an AI action and wait for it: `MOVE`, `MOVE_TO_CAR`, `ACTION_MOVE`,
`ACTION_FOLLOW`, `ACTION_FIRE`, `ACTION_RUNAWAY`, `ACTION_SIDELOOK`, `ACTION_BACKLOOK`,
`ACTION_RUNCLOSERTO`, `MOVE_TO_FRAME`, `BOX_GOTO`, `ACTION_STEPAWAY`, `ACTION_STRAFE`,
`ACTION_JUMP`, `KICKDRIVER`, `ACTION_PHONECALL` — measured by walking `entity_process_command`'s
control flow from each jump-table entry to the instructions that set `0x800` and clear `0xc00`.

`entity_script_switch` keeps only `0x1c000` when it hands the entity to the AI, so the three
persistent script switches `0x4000` / `0x8000` / `0x10000` survive the hand-over.

### 2c. The type group `+0xf4c` and where `human_type` comes from

`entity_game_init` fixes the group once, from the single `human_type` bit at `C_human+0x5fc` (§1).
`human_type` itself has two sources:

* **A placed NPC carries it in the scene.** Each object definition of a `scene2.bin` hands its
  `0xAE24` record to the actor's `LoadData` (`structs/mission.md` §6a). For a `C_entity` the record
  opens with `human_load_data`'s part: a version byte, then, in versions 3 to 6, the `human_type`
  dword, then the stats block. Every shipped record is version 6. The cutscene characters a `.chg`
  difference file adds (`structs/chunk.md` §4b) carry the same record.
* **An NPC spawned at run time gets a literal:** `4` from `police_manager_apply_state` and
  `pedestrian_manager_spawn_cop_from_pedestrian`, `0x10` from `pedestrian_spawn_gang_on_hit` and
  `traffic_car_spawn_mafia_pursuit_car`, `0x80` from `pedestrian_wave_at_taxi`. `human_init` clears
  the field and `player_init` sets the player's.

The census reads the `0xAE24` record of every kind-`0x1b` and kind-`2` object in the 79 shipped
`scene2.bin`. All 46 player records carry `0x1`, the value the code gives the player, which checks
the reading. The 924 `C_entity` records:

| `human_type` | group | NPCs | scenes | example frame names (scene) | reading |
|---|---:|---:|---:|---|---|
| `0x2` | 0 | 51 | 26 | Paulie (MISE01), Sam (MISE01), Salieri (MISE06-MESTO), Frank (MISE06-MESTO), Ralph (MISE06-MESTO), SalMan01K (MISE09-PREJIMKA) | the NPCs that follow the player (gameplay reading) |
| `0x8` | 3 | 196 | 32 | Guard2 (MISE08-HOTEL), PortGuard4 (MISE15-PRISTAV), Luigi (MISE04-SALIERY), VincenzoHIGH (MISE04-SALIERY), Cop02 (MISE08-HOTEL), GatesMan2 (MISE12-GARAGE) | NPCs that do not follow the player (gameplay reading); nothing narrower is established |
| `0x10` | 4 | 324 | 32 | Enemy02 (MISE01), Hoolig (MISE07-SARA), Detektiv (MISE09-PREJIMKA), pol01 (MISE08-HOTEL), Delnik9 (MISE15-PRISTAV), odstrelovac01 (EXTREME) | the mission's enemies, by the frame names; also the run-time gang members |
| `0x20` | 5 | 4 | 2 | PoliceMan (MISE07-SARA), acivil (MISE10-LETISTE), civil10 (MISE10-LETISTE), Czena5 (MISE10-LETISTE) | a special case confined to these two scenes; nothing more is known |
| `0x80` | 1 | 348 | 41 | civil2 (MISE02-ULICKA), Czena3 (MISE02-ULICKA), waiter (MISE08-HOTEL), barman (MISE03-MORELLO), sailor (MISE14-MESTO), Vratny (MISE18-MESTO) | civilians; also the run-time taxi passenger |
| `0x100` | 1 | 1 | 1 | invisible (EXTREME) | a single helper NPC |
| `0x4` | 2 | 0 | 0 | none, run-time only | the police |
| `0x40` | 6 | 0 | 0 | none, and no code writes it | unused |

The 463 cutscene characters the `.chg` files add (`FMVTommyHIGH`, `FMVPaul`, …) carry `0x1` (457) or
`0` (6), so all of them fall in group 1.

What the census settles:

* **Group 2 is the police**, and only run-time police reach it. That is why
  `police_board_suspend_npcs` skips group 2: when `entity_action_tick` case `0x21` captures the
  player, it freezes every other `C_entity` and leaves the police running
  (`structs/police-board.md` §3).
* **Group 1 is the civilians**, the group `entity_force_script` and `entity_force_ai` single out in
  AI state 4 (§1, §2).
* **The type is a per-scene role, not a character's identity.** One frame name lands in different
  groups from scene to scene: `Paulie` is `0x2` in 19 records, `0x8` in 5 and `0x80` in 2; `pol` is
  `0x10` in 22, `0x8` in 10 and `0x80` in 2.

The relationship classes `ai_logs_event` assigns between the groups are in `structs/ai_logs.md`
§4d.

## 3. Method family

Coverage: of the **110** PS2 `8C_entity` methods, **102** are in the tables below — 93 CONFIRMED
twins (including the three savegame adjustor thunks), 2 STRUCTURAL, 1 `inlined` (`actionTick_2`)
and 6 base stubs — and **8** have no PC function and are listed in §9. The `@3056@__dt__`
thunk is not one of the 110 (³). A further **12** `entity_*` PC functions have no PS2 twin,
and §3a / §3d hold the translation unit's static-init code and free helpers (§8).

The PC cluster `[0x00410000, 0x00437000)` holds **216** true function starts against these 110
methods — a block of roughly 50 `vec2_*` / `vec3_*` / HUD / freeride helpers is emitted straight
through the middle of the C_entity code, and the `entity_dtor → entity_force_ai` gap alone is PC 85
against PS2 28. Emission order is nevertheless monotone between the builds.

Every C_entity lifecycle method calls its `C_human` **parent** — an unambiguous, build-independent
signal, and the lever that anchors the lifecycle block of this table.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0040f700 | `crt_static_init_guard_0040f700` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `0x006f9ae0` and tail-jumps to the registrar below |
| 0x0040f720 | `crt_atexit_register_0040f720` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `atexit` |
| 0x0040f740 | `vec_up_00655208_static_init` | `g_vec_up_00655208` | - | - | - | static initialiser of the constant up-vector `(0, 1.0, 0)` |
| 0x004114d0 | `entity_ctor` | instance | `__ct__8C_entityFv` | 0x00311300 | CONFIRMED | kind = 0x1b, vtable `0x0063b488` |
| 0x00411620 | `entity_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor, vtable slot 16 ¹ |
| 0x00411640 | `entity_dtor` | instance | `__dt__8C_entityFv` | 0x00311490 | CONFIRMED | base dtor |
| 0x004255a0 | `dir8_table_normalized_006551a8_static_init` | `g_dir8_table_normalized_006551a8` | - | - | - | fills this unit's copy of the normalised 8-direction table, the same 24 constants as the copy at `0x006550d8` |
| 0x0043abe0 | `dword_vec_copy_assign` | a `{byte_flag, begin, end, cap}` dword-vector record | - | - | - | deep-copy `operator=`: allocates a fresh buffer at exact capacity and copies the leading byte and each dword |
| 0x0043b030 | `program_base_thunk_entity_scalar_deleting_dtor` | instance | `@3056@__dt__8C_entity` | - | CONFIRMED | MI adjustor thunk: `C_program*` → `C_entity*`, then jump ³ |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004257a0 | `entity_clear_act_state` | - | - | - | stores 0 to `act_state` `+0x78`; 5 callsites |
| 0x004257b0 | `entity_set_act_state_shock` | - | - | - | stores 3 (shock) to `act_state` `+0x78`; 1 callsite |
| 0x004258b0 | `entity_get_program` | `GetProgram__8C_entityFv` | 0x0034b140 | CONFIRMED | a real virtual getter returning `this + 0xaa8` — §4 |
| 0x00425b60 | `entity_get_human_type_group` | - | - | - | loads `human_type_group` `+0xf4c` (§2c) |
| 0x00425b70 | `entity_get_ai_vis_logs` | - | - | - | returns `this + 0xb04`, the `ai_vis_logs` sub-object |
| 0x00425b80 | `entity_get_ai_sensors` | - | - | - | returns `this + 0xbb4`, the `ai_sensors` sub-object; 33 callsites |
| 0x00425b90 | `entity_get_ai_logs` | - | - | - | returns `this + 0xe88`, the `ai_logs` sub-object; 13 callsites |
| 0x00425ba0 | `entity_get_ai_fight_modes` | - | - | - | returns `this + 0xf64`, the `ai_fight_modes` sub-object; 35 references, 31 of them in `entity_action_tick` |
| 0x00432ea0 | `entity_weapon_get_max_range` | `WeaponGetMaxRange__8C_entityFv` | 0x0033f840 | CONFIRMED | getter for `+0x11d4`; consumer-proven a range |
| 0x00432eb0 | `entity_weapon_get_best_range` | `WeaponGetBestRange__8C_entityFv` | 0x0033f850 | CONFIRMED | getter for `+0x11d8` |
| 0x00432ec0 | `entity_weapon_get_spread` | `WeaponGetRozptyl__8C_entityFv` | 0x0033f860 | CONFIRMED | getter for `+0x11dc` (*rozptyl* = spread) |
| 0x00432ed0 | `entity_weapon_get_readiness` | `WeaponGetReady__8C_entityFv` | 0x0033f870 | CONFIRMED | getter for `+0x11e0`; readiness fraction 0→1 |
| 0x00432ee0 | `entity_weapon_is_armed` | `WeaponIsArmed__8C_entityFv` | 0x0033f880 | CONFIRMED | **byte** getter for `+0x11ec` (`undefined1` on both) |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004117a0 | `entity_init` | `Init__8C_entityFP9I3D_frame` | 0x00311650 | CONFIRMED | thin override forwarding to the `C_human` parent (`human_init` ×1 ↔ `Init__7C_human` ×1). **The parent call is the decisive class signal for this whole family** |
| 0x004117c0 | `entity_duplicate_from` | `DuplicateFrom__8C_entityFP7C_actor` | 0x00311670 | CONFIRMED | parent `human_duplicate_from`, then duplicates the script VM (`program_duplicate_from` ↔ `DuplicateProgramFrom__9C_program`) |
| 0x00411800 | `entity_game_init` | `GameInit__8C_entityFv` | 0x003116c0 | CONFIRMED | the real construction step: parent `human_game_init`, then inits the AI sub-objects (§7) — C_program, `ai_vis_logs_init`, `ai_sensors_init`, `ai_character_init`, `ai_logs_init`. Two exact counts: `program_is_event_id` ×2; `rand_range` ×2 ↔ `rnd__Fi` ×2 |
| 0x00411e80 | `entity_reset` | `Reset__8C_entityFv` | 0x00311bd0 | CONFIRMED | the FULL per-entity AI reset — re-inits every AI sub-object (vis_logs / sensors / character / logs inits, `ai_sensors_destroy`, action-manager clear, `human_do_crouched`, `human_pose_null_rotate`) |
| 0x00411fb0 | `entity_reset_state` | `ResetState__8C_entityFv` | 0x00311ca0 | CONFIRMED | the LIGHT reset, a strict subset of `entity_reset`. Exact 3:3 callee match (`ai_sensors_destroy`, `ai_vis_logs_init`, action-manager clear). `ai_sensors_destroy` leaves the sensors in their post-init state: it writes every field `ai_sensors_init` writes, frees each slot's buffer first and keeps the owner pointer, so no re-init is needed |
| 0x00412060 | `entity_game_done` | `GameDone__8C_entityFv` | 0x00311d10 | CONFIRMED | teardown: action-manager clear, `ai_sensors_destroy` (frees the scent blobs), path-node release, then parent `human_game_done` |
| 0x00412100 | `entity_save_data` | `SaveData__8C_entityFP7C_chunk` | 0x00311de0 | CONFIRMED | writes the entity to a mission **.dta chunk** (not the savegame path). Exact 3:3: parent `human_save_data` + `dtaWrite` + `program_save_source_code` |
| 0x00412150 | `entity_load_data` | `LoadData__8C_entityFP7C_chunk` | 0x00311e60 | CONFIRMED | reads it back: parent `human_load_data` + `dtaSeek` + `dtaRead` + `program_load_source_code`. True extent 149 B |
| 0x004121f0 | `entity_collision` | `Collision__8C_entityF…` | 0x00311f30 | CONFIRMED | a 16-byte override forwarding to `human_collision`; C_entity adds nothing to collision handling |
| 0x00412200 | `entity_hit` | `Hit__8C_entityF10E_hit_type…` | 0x00311f40 | CONFIRMED | reaction to being hit: reset `ai_ghost`, parent `human_hit`, then read the attacker's KIND and special-case a CAR whose owner is the PLAYER or another `C_entity`; logs the attacker and drives the police response (sighting + gun/handcuffs escalation). Same kind gate, same constants **4 / 2 / 0x1b** in the same nesting order, plus `car_get_owner` ↔ `GetOwner__5C_car` |
| 0x00412410 | `entity_cb_hit_info` | `cbHitInfo__8C_entityF10E_hit_typefP7C_actor` | 0x00312240 | CONFIRMED | signature matches exactly: `(E_hit_type, float, C_actor*)` — a float in the middle slot is a strong discriminator |
| 0x00412480 | `entity_ai` | `AI__8C_entityFUi` | 0x00312300 | CONFIRMED | the **AI BRAIN** — the per-frame perceive→decide→act tick: perception + logs + event ring, target and weapon choice, then it builds an `ai_action` from the 35-strong factory family (§6), installs it and drives the action queue; also runs the entity's script program (`program_call_process` ×12). 8560 B. **Reached ONLY by virtual dispatch — vtable slot 13**: the sole absolute reference to its VA is at `0x0063b4bc` inside `g_entity_vt`, which is why it has zero textual callsites. Callee multiset matches with two exact counts (`CallProcess` ×12, `WeaponChoose` ×7) |
| 0x004145f0 | `entity_action_tick` | `actionTick__8C_entityFP9ai_actionUiUiPUi` | 0x00315040 | CONFIRMED | **singular** — advances ONE `ai_action`, switching on its type across **40 outer cases** in `0x00..0x2a` (`0x0f` unused). Exact signature `this + (ai_action*, uint, uint, uint*)`; the outer case set equals `actionTick ∪ actionTick_2` exactly; sole caller ×4, mode multiset {0,1,2,2} on both |
| - | - | `actionTick_2__8C_entityFP9ai_actionUiUiPUi` | 0x003201f0 | inlined | a **PS2-only source split** of `entity_action_tick`'s switch, not a distinct method: its sole caller is `actionTick`, which passes all 5 args through verbatim, and it handles the high type tail `0x1d..0x2a`. MSVC emitted all 40 cases as ONE function |
| 0x00425740 | `entity_is_tickable` | - | - | - | returns 1 when the entity is alive (`+0x5d`), its `movement_mode` (`+0x40c`) is 1, 2 or 9, and `+0x1b0` / `+0x1c8` are not both set; 10 callsites |
| 0x004258c0 | `entity_actions_tick` | `actionsTick__8C_entityFUi` | 0x0032eaa0 | CONFIRMED | **plural** — the per-frame driver that ticks all of the entity's actions and does the idle/modifier-list teardown. Sole caller of the singular tick; exactly 4 callsites; same mode multiset {0,1,2,2} on both builds |
| 0x00425bb0 | `entity_update` | `Update__8C_entityFUi` | 0x0032ef40 | CONFIRMED | the per-frame Update override: parent `human_update`, then `entity_release_target` + `police_board_invalidate_actor`. The parent-Update call is the decisive class signal |
| 0x00425cb0 | `entity_death` | `Death__8C_entityFv` | 0x0032f430 | CONFIRMED | **shared magic constant `0xea61`** on both builds in the same guard, plus the identical chain IsEventID → SetEvent → vtable → CallProcess → `ai_action` Clear → parent `human_death`. This diff exposed the whole `C_program` script-VM class on the PC side |
| 0x00425d30 | `entity_script_decode_params` | `DecodeParams__8C_entityFPcRQ29C_program13S_instruction` | 0x0032f580 | CONFIRMED | `this = +0xa9c` script VM; a C_entity **override** of the C_program virtual. Signature `(char*, S_instruction&)`; uses `__strcmpi` (the script language is case-insensitive), `__ftol`, `web_path_find_node_id` |
| 0x00427690 | `entity_script_decode_instruction` | `DecodeInstruction__8C_entityFPcRi` | 0x00332ab0 | CONFIRMED | `this = +0xa9c` script VM. 9744 B, with **0 direct `E8` callsites** — only ever reached through the vtable, and identified by slot position: its single `.rdata` reference is `0x0063b474`, the slot between `DecodeParams` and `Process` in C_entity's `C_program` base-subobject vtable (`0x0063b45c`..`0x0063b480`), mirroring the PS2 `@3056@` thunk address order. Both builds materialise the same ~4.3 KB command-descriptor table in a local stack frame; PC unrolls the copy, which is the whole size difference |
| 0x00429ca0 | `entity_script_format_command` | `DecompileCommand__8C_entityFiPc` | 0x00332c10 | CONFIRMED | `this = +0xa9c` script VM override; renders an instruction back to source text via `crt_sprintf` (`Enemy_UseCar_act(%i,…)`, `EntityMoveToFrame(%d,%f)`) |
| 0x0042a660 | `entity_process_command` | `Process__8C_entityFUi` | 0x003336b0 | CONFIRMED | `this = +0xa9c` the script-VM instruction processor for the 64 entity commands (§5); `this` is the program subobject (§4). Opens with the MSVC SEH prologue `64 a1 00 00 00 00` after NOP padding |
| 0x0042de60 | `entity_script_tick` | `scriptTick__8C_entityFUi` | 0x00338df0 | CONFIRMED | the per-frame SCRIPT tick — the branch of the brain that runs under script control, against `entity_actions_tick` for the action queue. Gated on `+0x5d` and `movement_mode` (`+0x40c`) ∈ {1,2,9}; also feeds the blocked-con list. Exact ×10 ↔ ×10 callsites from the confirmed `entity_ai` twin, and the only ×10 callee on either side |
| 0x0042e530 | `entity_get_sight_range` | `GetVisibleRange__8C_entityFv` | 0x00339580 | CONFIRMED | **SIGHT RANGE in world units** = the embedded `ai_character`'s sight stat × **200.0** (`add ecx,0xebc; call ai_character_get_sight; fmul [0x63b5f4]`). Consumer-proven a range: `ai_vis_logs_tick` compares it to inter-entity distance (×4 on both builds) |
| 0x0042e550 | `entity_get_hearing_sensitivity` | `GetHearQuality__8C_entityFv` | 0x003395b0 | CONFIRMED | **HEARING SENSITIVITY** — forwards to the embedded `ai_character`. Consumer proves *sensitivity*, not "quality": `sound_arena_update_entity_sound_contacts` registers a contact iff loudness > `1.05 − this` ⇒ higher = hears quieter sounds |
| 0x0042e560 | `entity_get_head_dir` | `GetHeadDir__8C_entityFv` | 0x003395c0 | CONFIRMED | head/look DIRECTION vec3 @ `+0x11b4`. Identical sibling of the next row — separated by **order** (lower offset + earlier VA on both builds), not by body |
| 0x0042e590 | `entity_get_head_pos` | `GetHeadPos__8C_entityFv` | 0x003395d0 | CONFIRMED | head POSITION — the eye/ear point perception is measured from — vec3 @ `+0x11c0`. Identical sibling — separated by order (higher offset + later VA on both builds) |
| 0x0042e5c0 | `entity_script_switch` | `ScriptSwitch__8C_entityFv` | 0x003395e0 | CONFIRMED | ×2 ↔ ×2 from the confirmed `entity_ai` twin, identical nesting and **byte-identical flag logic** `if (f & 0x200) f \|= 0x400` on `+0xf08` (§2b) |
| 0x0042e650 | `entity_force_ai` | `ForceAI__8C_entityFUiUiUif` | 0x003396a0 | CONFIRMED | force an AI action |
| 0x0042e7f0 | `entity_script_is_fake` | `isFakeScript__8C_entityFv` | 0x003398e0 | CONFIRMED | reads the `C_program` instruction vector (`+0xaac`/`+0xab0` = `program+0x10`/`+0x14`) and returns 1 when the pc is past the end = the script is a placeholder. There is no lock involved |
| 0x0042e8a0 | `entity_force_script` | `ForceScript__8C_entityFbb` | 0x00339a00 | CONFIRMED | forces the entity back under script control, abandoning its current `ai_action`. Shared event constant `3`; calls `isFakeScript`; `Do_ShockUp ↔ human_do_shock_up` |
| 0x0042e990 | `entity_script_callback` | `CallBack__8C_entityFQ29C_program11E_cb_action` | 0x00339b50 | CONFIRMED | `this = +0xa9c` **byte-identical** — the same magic constants `0x200` / `0xfffff1ff` on the same enum values 0/1, on `program+0x46c` = `+0xf08` (§2b) |
| 0x0042e9c0 | `entity_dispatch_event` | `Event__8C_entityFP7C_actorUiPv` | 0x00339c40 | CONFIRMED | event dispatch |
| 0x00432550 | `entity_weapon_choose` | `WeaponChoose__8C_entityFv` | 0x0033eba0 | CONFIRMED | pick a weapon (AI); also ramps `+0x11e0` |
| 0x00432c50 | `entity_equip_weapon_by_id` | `WeaponChoose__8C_entityFUi` | 0x0033f5b0 | CONFIRMED | the **ID-taking overload**: same "weapon in hand ⇒ return 1" early-out through the inventory vector, same search, same return contract |
| 0x00432ef0 | `entity_weapon_get_info` | `WeaponGetInfo__8C_entityFv` | 0x0033f890 | CONFIRMED | writes the whole weapon band from the weapon-def record |
| 0x00432f60 | `entity_do_shoot_timed` | `Do_Shoot_Timed__8C_entityFbP8S_vectorP7C_actor` | 0x0033f920 | CONFIRMED | timed shot (calls C_human `Do_Shoot`) |
| 0x00433030 | `entity_weapon_timing_setup` | `WeaponTimingSetup__8C_entityFUi` | 0x0033fa20 | CONFIRMED | weapon timing setup |
| 0x00433170 | `entity_release_target` | `InvalidateActor__8C_entityFP7C_actor` | 0x0033fb60 | CONFIRMED | invalidate a referenced actor |
| 0x00433290 | `entity_rebind_neck_frame` | `InvalidateSelf__8C_entityFv` | 0x0033fcd0 | CONFIRMED | stores the model frame's child `"Neck"`, else `"neck"`, found by the frame vtable lookup, in `neck_frame` `+0x11cc`; `0xffff` is the lookup's last argument on both builds. Its only caller is `human_change_model` behind `if (kind == 0x1b)`, and the PS2 twin's caller guards the identical statement position the same way. Same two string literals in order, same retry-on-zero shape |
| 0x004332d0 | `entity_can_safe_shoot` | `CanSafeShoot__8C_entityFP7C_actorbb` | 0x0033fd50 | CONFIRMED | line-of-fire safety check |
| 0x00433910 | `entity_trace_human_scent` | `TraceHuman__8C_entityFP7C_humanP7ai_blob` | 0x00340570 | CONFIRMED | walks a human's **scent trail**: loops the target's `ai_blob_log` via `ai_blob_log_get_blob`, subtracting `blob.pos − frame.pos` per blob. Same frame fetch, same `& 0x20` staleness flag + `UpdateWMatrixProc`. The PS2 signature `(C_human*, ai_blob*)` is what identifies the loop subject as scent blobs. `structs/ai_blob_log.md` |
| 0x004339e0 | `entity_add_using_object` | `AddUsingObject__8C_entityFv` | 0x00340680 | CONFIRMED | registers the entity as a user of a `C_using_object`: script-event check, world-matrix refresh, dedup-ing push onto the user list. Reads its `+0x1230`..`+0x1258` band. Two exact callee matches (`program_is_event_id`, `UpdateWMatrixProc`) plus the field band and a forced position; its third callee pair is **refuted** (`using_object_add_object` is not `AddObject__14C_using_object`), so the row rests on two |
| 0x00433ab0 | `entity_switch_ai_state` | `StateSwitcher__8C_entityFv` | 0x00340740 | CONFIRMED | **the AI state machine.** Reads the strongest fear / anger / death / threat targets from `ai_logs`, checks the substate locks, then forces an AI state via `entity_force_ai` (×3) or returns control to the script; caches the target picks at `+0x127c`/`+0x1280`. Three exact call-count matches on independently confirmed twins |
| 0x004340c0 | `entity_spawn_police_response` | `SpawnPolice__8C_entityFP7C_humanUiUifUi` | 0x00341140 | CONFIRMED | builds the whole police AI response: forces AI states (×5), installs police / barricade-fight / car-jump actions, sets the fight-mode enemy, dispatches an entity event. **Seven exact call-counts.** It also **seeds a smell contact** (`ai_blob_log_get_blob` → `ai_sensors_build_smell_contact_from_blob` → `ai_sensors_new_smell_contact` → free) — the same chain `ai_sensors_cancel_visual_contact` runs on losing sight |
| 0x004342b0 | `entity_lock_substate` | `LockSubstate__8C_entityFUiUifUib` | 0x00341400 | CONFIRMED | upsert into the `std::vector<ai_substate_lock>` at `+0x10b8`; commits `key` to `+0x10c4` when its trailing bool is set. Same find→erase→insert shape; 6 params on both; PC's trailing `char` = PS2's `b`. PS2's `insert__…<ai_substate_lock>` template argument names the row type |
| 0x00434550 | `entity_unlock_substate` | `UnlockSubstate__8C_entityFUi` | 0x003414e0 | CONFIRMED | find the lock row for `key` and erase it. PC inlines the vector erase (shift-down + `end -= 0xc`); PS2 calls `erase__…<ai_substate_lock>` |
| 0x004345e0 | `entity_has_locked_state` | `hasLockedState__8C_entityFv` | 0x00341550 | CONFIRMED | **byte-identical**: `return *(this+0x10c4) != 0x29a` — the shared magic 0x29a (=666) "no lock" sentinel. Generic substate locking, not pursuit-specific |
| 0x004345f0 | `entity_has_locked_substate` | `hasLockedSubstate__8C_entityFUiPUiPf` | 0x00341560 | CONFIRMED | is substate `key` locked — and if so hand back its two payload values (both out-params NULL-checked). Identical: stride 0xc, key at row+0, outputs at row+4/+8, same 0/1 return. PS2's signature `(uint, uint*, float*)` types the payload |
| 0x00434680 | `entity_get_group_position` | `GetGroupPosition__8C_entityFP7C_actorP8S_vector` | 0x003415f0 | CONFIRMED | returns an actor's position **only if this entity can see it**. Exact 2:2: `ai_sensors_can_see` ↔ `isVisible__10ai_sensors` + `UpdateWMatrixProc` |
| 0x004346e0 | `entity_mem_write` | `MemWrite__8C_entityFPPvPvUiUi` | 0x00341670 | CONFIRMED | the byte-level write primitive the whole SaveGame chain is built on ² |
| 0x00434720 | `entity_mem_read` | `MemRead__8C_entityFPPvPvUiUi` | 0x003416c0 | CONFIRMED | the byte-level read primitive ² |
| 0x00434760 | `entity_get_vis_logs_size` | `GetVisLogsSize__8C_entityFv` | 0x00341710 | CONFIRMED | **constant 0x184** ² |
| 0x00434770 | `entity_save_vis_logs` | `SaveVisLogs__8C_entityFPPv` | 0x00341720 | CONFIRMED | VIS-LOGS block; field band `+0xb14`/`+0xba4` ² |
| 0x00434840 | `entity_load_vis_logs` | `LoadVisLogs__8C_entityFPPv` | 0x00341850 | CONFIRMED | the load half ² |
| 0x004348e0 | `entity_get_sensors_size` | `GetSensorsSize__8C_entityFv` | 0x003419c0 | CONFIRMED | **constant 0x3a0** ² |
| 0x004348f0 | `entity_save_sensors` | `SaveSensors__8C_entityFPPv` | 0x003419d0 | CONFIRMED | SENSORS block, base `+0xbc0`; the sound-contact ring at `+0xc74` lies inside it ² |
| 0x00434c10 | `entity_load_sensors` | `LoadSensors__8C_entityFPPv` | 0x00341ec0 | CONFIRMED | the load half ² |
| 0x00434ea0 | `entity_get_logs_size` | `GetLogsSize__8C_entityFv` | 0x00342340 | CONFIRMED | **variable** — it loops the list ² |
| 0x00434f00 | `entity_save_logs` | `SaveLogs__8C_entityFPPv` | 0x00342390 | CONFIRMED | AI-LOGS block, base `+0xeac` ² |
| 0x004351e0 | `entity_load_logs` | `LoadLogs__8C_entityFPPv` | 0x00342740 | CONFIRMED | the load half ² |
| 0x00435410 | `entity_get_character_size` | `GetCharacterSize__8C_entityFv` | 0x00342a00 | CONFIRMED | **constant 0x24** ² |
| 0x00435420 | `entity_save_character` | `SaveCharacter__8C_entityFPPv` | 0x00342a10 | CONFIRMED | CHARACTER block, band `+0xebc`..`+0xec8` ² |
| 0x004354e0 | `entity_load_character` | `LoadCharacter__8C_entityFPPv` | 0x00342b10 | CONFIRMED | the load half ² |
| 0x004355a0 | `entity_get_state_size` | `GetStateSize__8C_entityFv` | 0x00342c10 | CONFIRMED | **identical formula**: `(list_bytes/0x10)*0x10 + CONST + nlocks*0xc`; the constant differs per build (PC `0x168`). The savegame "State" block is the blocked-con list **plus the substate-lock table** — independent confirmation of what `+0x10b8` is |
| 0x00435600 | `entity_save_state` | `SaveState__8C_entityFPPv` | 0x00342c60 | CONFIRMED | exact count: `entity_mem_write` ×**27** ↔ `MemWrite` ×**27**. Serialises the substate-lock vector and the blocked-con list |
| 0x004358f0 | `entity_load_state` | `LoadState__8C_entityFPPv` | 0x00343060 | CONFIRMED | rebuilds the same two containers in the same order at the forced 1:1 position. Counts differ (`entity_mem_read` ×27 vs `MemRead` ×21) because PC inlines PS2's vector ops as `fill_n_stride_0xc` / `copy_range_stride_0xc` / `vec_count_stride_0xc`; the container evidence carries the row |
| 0x00435e00 | `entity_get_action_size` | `GetActionSize__8C_entityFUi` | 0x00343480 | CONFIRMED | the **serialized** size of one saved `ai_action`, switched on its type id. Its case set is neither a subset nor a superset of the factory table (§6) |
| 0x00435f60 | `entity_get_action_man_size` | `GetActionManSize__8C_entityFv` | 0x003435b0 | CONFIRMED | reads `+0x1168` = the `ai_action_manager` at `+0x1164` ² |
| 0x00436020 | `entity_save_action_man` | `SaveActionMan__8C_entityFPPv` | 0x003436d0 | CONFIRMED | every queued `ai_action` ² |
| 0x00436cf0 | `entity_load_action_man` | `LoadActionMan__8C_entityFPPv` | 0x00344da0 | CONFIRMED | the load half ² |
| 0x00438430 | `entity_save_game_get_size` | `SaveGameGetSize__8C_entityFv` | 0x00346b30 | CONFIRMED | **the same eight callees**: the `C_human` parent + the `C_program` + all six sub-block size getters (vis_logs, sensors, logs, character, state, action_man). It calls `human_save_game_get_size` as its parent, which is why the two were confusable |
| 0x00438490 | `entity_save_game_save` | `SaveGameSave__8C_entityFPPv` | 0x00346bc0 | CONFIRMED | parent → C_program → each sub-block ² |
| 0x00438550 | `entity_save_game_load` | `SaveGameLoad__8C_entityFPPv` | 0x00346cc0 | CONFIRMED | true extent 192 B ² |
| 0x00438610 | `entity_is_in_entity_fire_area` | `isInEntityFireArea__8C_entityF…` | 0x00346dd0 | CONFIRMED | friendly-fire test; exact arity `this` + 3 ² |
| 0x00438970 | `entity_is_in_player_fire_area` | `isInPlayerFireArea__8C_entityF…` | 0x003471d0 | CONFIRMED | exact arity `this` + 2 ² |
| 0x00438ea0 | `entity_get_fire_metrics` | `GetFireMetrics__8C_entityFP12ai_fire_area` | 0x003475d0 | CONFIRMED | fills an `ai_fire_area` (§1a) ² |
| 0x00438f00 | `entity_get_player_fire_metrics` | `GetPlayerFireMetrics__8C_entityF…` | 0x003476a0 | CONFIRMED | the same out-struct for the player ² |
| 0x00438fa0 | `entity_get_target` | `GetTarget__8C_entityFv` | 0x003477a0 | CONFIRMED | reads the PENDING `ai_action` and switches on its type id (case 6 = `NewFire`, `0x1c` = `NewCornerFight`), returning the target from its payload — a third, independent use of the type enum ² |
| 0x00439050 | `entity_beware_cross_fire` | `BewareCrossFire__8C_entityFUi` | 0x00347870 | CONFIRMED | crossfire avoidance; companion of the type-`0x2a` `ai_action_new_strafe_beware` ² |
| 0x004392e0 | `entity_has_heard_gunfire` | `isHearedShooting__8C_entityFUiUi` | 0x00347c00 | CONFIRMED | byte-level match; magic **type 2 = GUNFIRE**; reads `+0xc74`/`+0xc78` ² |
| 0x0043a150 | `entity_is_in_danger` | `IsInDanger__8C_entityFv` | 0x003495a0 | CONFIRMED | asks "is ANY `C_entity` running a fight/fire action targeting me?" Both builds scan the mission actor vector, gate on kind `0x1b`, then run a `GetJob(type)` chain comparing each action's payload to `this`. Shared type ids `0x1e` / `0x1d`; PC additionally tests `0x1c`, `0x21`, 6 |
| 0x0043a220 | `entity_tick_ghosts` | `TickGhost__8C_entityFUi` | 0x00349710 | CONFIRMED | the per-frame tick of the `ai_ghost` vector: walks it BACKWARDS, shift-down-erases records past their expiry or drifted beyond their radius, then rebuilds the collision ignore list at `+0x698`. Same reverse walk, same time and drift tests, same erase and rebuild; ×1 ↔ ×1 from the confirmed `entity_ai` twin |
| 0x0043a850 | `entity_suspend_ai_and_hide` | `SedniNaPrdelASledujKino__8C_entityFb` | 0x00349f20 | STRUCTURAL | adds 666 to the AI state, pushing it out of every `switch` range the AI dispatches on, so the entity stops being driven without losing state, then plays a breath animation. When the caller asks for it AND the frame is visible AND the entity is alive AND `+0x04 & 5` is clear it also latches `+0xf08 \|= 0x20000`, hides the I3D frame through the frame vtable `+0x24`, removes the shadow caster and clears `+0x47c`. Sole caller `game_entity_block` (§9). Dev-joke PS2 name |
| 0x0043a8e0 | `entity_resume_ai_and_show` | `VzbudSeUzSouTitulky__8C_entityFv` | 0x00349f30 | STRUCTURAL | the inverse of the row above: subtracts 666, undoes the hide when the `0x20000` latch is set. Sole caller `game_entity_block` (§9). Dev-joke PS2 name |
| 0x0043a980 | `entity_add_blocked_con` | `AddBlockedCon__8C_entityFP11blocked_con` | 0x00349fc0 | CONFIRMED | dedup-on-insert into the `vector<blocked_con>` at `+0x119c`. Shared magic `0x29a` tested on `param_2[2]` on both; stride 0x10; same dedup-with-early-return; the load callers pair (`LoadState` ×1 ↔ `entity_load_state` ×1). PC callers: `entity_action_tick` ×20, `entity_script_tick` ×1, `entity_load_state` ×1. **PS2's signature names the element type** |
| 0x0043b040 | `program_base_thunk_entity_save_game_get_size` | `@3056@SaveGameGetSize__8C_entityFv` | 0x0034b8c0 | CONFIRMED | MI adjustor thunk |
| 0x0043b050 | `program_base_thunk_entity_save_game_save` | `@3056@SaveGameSave__8C_entityFPPv` | 0x0034b8b0 | CONFIRMED | MI adjustor thunk |
| 0x0043b060 | `program_base_thunk_entity_save_game_load` | `@3056@SaveGameLoad__8C_entityFPPv` | 0x0034b8a0 | CONFIRMED | MI adjustor thunk |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__8C_entityFUi` | 0x0032ef30 | base-stub | slot 15; does nothing; the PS2 body is `jr ra; nop`, an empty function. A stub shared with other vtables (`structs/actor.md` §3c) |
| 0x005c8fa0 | `entity_get_ai_state_base_code` | - | - | - | reads the AI state `+0xf40` and strips both biases — minus 10000 when `>= 10000`, then modulo 666 — returning the base state code |
| 0x005c8fe0 | `entity_set_ai_state_resettable` | - | - | - | writes bit 0 of `ai_flags` `+0xf50` (script `HUMAN_RETURNFROMPANIC`, §2); a header inline with no symbol on either build |
| 0x005c9010 | `entity_set_state_switcher_suspended` | - | - | - | sets or clears bit 1 of `ai_flags` `+0xf50`, which makes `entity_switch_ai_state` return at once |
| 0x005c9040 | `entity_unregister_using_object` | - | - | - | removes the entity's `S_using_object` record at `+0x1230` from the mission-level usable-object registry |
| - | - | `@3056@CallBack__8C_entityF…` | 0x0034b850 | base-stub | MI adjustor thunk (8 B) |
| - | - | `@3056@DecompileCommand__8C_entityFiPc` | 0x0034b860 | base-stub | MI adjustor thunk |
| - | - | `@3056@Process__8C_entityFUi` | 0x0034b870 | base-stub | MI adjustor thunk |
| - | - | `@3056@DecodeInstruction__8C_entityFPcRi` | 0x0034b880 | base-stub | MI adjustor thunk |
| - | - | `@3056@DecodeParams__8C_entityF…` | 0x0034b890 | base-stub | MI adjustor thunk |

¹ An MSVC-only construct, so it can have no PS2 symbol. ² Recovered through the emission-order trailing
gap; the proof is structural — `entity_save_game_get_size` sums these members and so names them.
³ The PS2 `@3056@__dt__` thunk is not one of the 110 methods counted here. The five `@3056@` MI
thunks at the end of 3c are genuine stubs: the five overrides they would adjust fill the `C_program`
base-subobject vtable directly (§4).

### 3d. Free helpers

The translation unit also holds generic helpers: COMDAT emissions of inline vector math and STL
templates, which land in whichever object file used them, and a few inline accessors of other
classes. They are listed because they live in this unit; none is a `C_entity` method.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00424e80 | `vec2_set_xy` | - | - | - | sets the two components of a 2D vector |
| 0x00424ea0 | `vec2_length_sq` | - | - | - | squared length of a 2D vector |
| 0x00424ec0 | `vec2_normalize_in_place` | - | - | - | normalises a 2D vector in place |
| 0x00424f00 | `vec2_signed_distance_to_line` | - | - | - | signed distance of a 2D point from a line |
| 0x00424f60 | `vec3_scale_to_by_ptr` | - | - | - | scales a vector into an out-vector |
| 0x00424f90 | `vec3_scale_in_place` | - | - | - | scales a vector in place |
| 0x00424fb0 | `vec3_negate_to` | - | - | - | negates a vector into an out-vector |
| 0x00424fd0 | `vec3_add_to` | - | - | - | adds two vectors into an out-vector |
| 0x00425000 | `vec3_add_in_place` | - | - | - | adds a vector in place |
| 0x00425030 | `vec3_sub_in_place` | - | - | - | subtracts a vector in place |
| 0x00425060 | `clear3_dwords_at` | - | - | - | zeroes three dwords; not a vector primitive |
| 0x004250f0 | `vec3_length_sq` | - | - | - | squared length of a vector; the result is in ST0 only, so the decompiler shows an empty body |
| 0x00425110 | `vec3_cross_to` | - | - | - | cross product into an out-vector |
| 0x00425150 | `vec3_cross_global_lhs_to` | - | - | - | cross product with a global left operand |
| 0x004251a0 | `vec3_cross_global_rhs_to` | - | - | - | cross product with a global right operand |
| 0x00425390 | `act_mission_get_game` | - | - | - | returns the `C_game` singleton `*(act_mission+0x24)`; 330+ callsites |
| 0x004253a0 | `folded_get_dword_0x10` | - | - | - | the ICF-folded kind reader (`structs/actor.md` §3b) |
| 0x00425420 | `stub_ret0_args4_00425420` | - | - | - | returns 0; the inherited `GetMaterialID` slot 32 of `g_entity_vt` (`structs/actor.md` §3c) |
| 0x004254f0 | `ambient_ai_set_suspended` | - | - | - | `g_ambient_ai_suspended = value`, 1 while the ambient-AI population is suspended |
| 0x00427650 | `free_ptr_pair_and_clear` | - | - | - | deletes each of two non-null pointer slots and nulls them |
| 0x0042dc90 | `vec3_dot_product_0042dc90` | - | - | - | a second COMDAT emission of the 3D dot product; 3 callsites, all in `entity_process_command` |
| 0x0043ac80 | `blocked_con_vector_at` | - | - | - | `std::vector<blocked_con>::operator[]`, `*(this+4) + idx*0x10`; 4 callsites |
| 0x0043ac90 | `std_vector10_insert_n` | - | - | - | `std::vector::_Insert_n` for 16-byte elements |
| 0x0043afa0 | `uint_ptr_min` | - | - | - | minimum by dereference; on equality the second pointer wins |
| 0x0043afe0 | `stl_construct_stride_0x1c` | - | - | - | guarded copy of one `0x1c`-byte record; 10 callers in five unrelated regions |

## 4. The `C_program` sub-object

The script VM is a `C_program` sub-object embedded at **`+0xa9c`**, proven twice from the PC side:
`entity_death` calls its vtable through `*(this+0xa9c)`, and `entity_script_is_fake` reads
`+0xaac`/`+0xab0` = `program+0x10`/`+0x14`, the instruction vector.

PC `C_program` internals as this class uses them:

| item | PC offset (in the program) | note |
|---|---|---|
| instruction list | `begin +0x10`, `end +0x14` | the count is **computed** as `(end − begin) >> 3`, stride 8 |
| pc | `+0x1c` | - |
| cleared on the fake-script path | `+0x60` | = entity `+0xafc`, `ai_priority` |

`C_entity` **overrides three `C_program` virtuals** — `DecodeParams` / `DecodeInstruction` /
`DecompileCommand` — plus the savegame trio, which is why the entity carries adjustor thunks
(`program_base_thunk_entity_*`, `0x0043b030`..`0x0043b060`) that convert a `C_program *` receiver back
to a `C_entity *`.

**`entity_get_program` (`0x004258b0`) returns `this + 0xaa8`, i.e. `program + 0xc`.** A callsite that
does `GetProgram()->field` is indexing from `+0xc`; subtract `0xc` before comparing against the table
above. `+0xa9c` is the base — never "correct" it to `+0xaa8`. The getter has **0 direct `E8`
callsites** (pure vtable dispatch); it was found through its single `.rdata` data reference, C_entity
vtable **slot 35** (`0x0063b514`), whose neighbours identify it: the slot before holds
`human_get_bound_points_2d` (GetBoundPoints2D) and the slot after `human_arm_bloody_footprints`
(cbCollTester), exactly `GetProgram`'s position in the `C_actor` virtual order; three slots earlier
the shared `return 0` stub `stub_ret0_args4_00425420` fills the `GetMaterialID` position.

`program + 0xc` is the header of the instruction vector: MSVC's `{allocator, begin, end, cap}`, with
`begin` / `end` at `program+0x10` / `+0x14`. The vector indexer `vec_index_addr_stride_0x8` takes
exactly this address from `program_process_command` and from `entity_action_tick`
(`structs/program.md` §9), so `GetProgram` hands out the compiled script, not the `C_program`
object. The cross-build comparison of the skew is in §9.

**The five overrides run on the program subobject.** `entity_script_decode_params`,
`entity_script_decode_instruction`, `entity_process_command`, `entity_script_format_command` and
`entity_script_callback` fill slots 5..9 of the `C_program` base-subobject vtable (`0x0063b45c`)
directly, with no adjustor thunk, and the `C_program` code calls them with `ecx` = the program
(`program_set_event`). Inside them every entity field is program-relative: `+0x46c` is `+0xf08`,
`+0x4b4` is `+0xf50`, `+0x628` / `+0x62c` are `+0x10c4` / `+0x10c8`, and `entity_process_command`
recovers the entity itself as `lea ecx,[ebp-0xa9c]`.

## 5. The entity script commands

`entity_script_decode_instruction` builds its keyword table on the stack: 64 opcodes
`0x1000..0x103f`, each a 32-byte name and the opcode. A scan of that table gives every keyword;
nearly all exist twice, as `ENEMY_<name>` and `ENTITY_<name>`, and the tables in §2b and here give
the `<name>`. `entity_script_format_command`
prints its own debug name per opcode (`Entity_Blind(%d)` for `VIDIM`, and so on).

| opcode | name | opcode | name | opcode | name | opcode | name |
|---|---|---|---|---|---|---|---|
| `0x1000` | `MOVE` | `0x1010` | `TEXT` ¹ | `0x1020` | `BOX_PUT` | `0x1030` | `BLOCK_CHECKPOINT` |
| `0x1001` | `PLAYANIM` | `0x1011` | `ACTION_MOVE` ¹ | `0x1021` | `ACTION_STEPAWAY` | `0x1031` | `SHUT_UP` |
| `0x1002` | `STOPANIM` | `0x1012` | `ACTION_FOLLOW` | `0x1022` | `ACTION_STRAFE` | `0x1032` | `DEBUG_GRAPHS` ¹ |
| `0x1003` | `LOOKTO` | `0x1013` | `ENTER_DOOR` | `0x1023` | `ACTION_JUMP` | `0x1033` | `ACTION_CARJUMP` ¹ |
| `0x1004` | `WAIT` | `0x1014` | `ACTION_FIRE` | `0x1024` | `USE_DETECTOR` | `0x1034` | `PODVADIM_JAK` |
| `0x1005` | `LOOK` | `0x1015` | `USE_RAILWAY` | `0x1025` | `SETSTATE` | `0x1035` | `WATCH` |
| `0x1006` | `TALK` | `0x1016` | `ACTION_RUNAWAY` | `0x1026` | `LOCKSTATE` | `0x1036` | `GROUP_ADDCAR` |
| `0x1007` | `STOP` | `0x1017` | `ACTION_SIDELOOK` | `0x1027` | `TURNBACK` | `0x1037` | `GROUP_CHCIPNI_HAJZLE` |
| `0x1008` | `USECAR` | `0x1018` | `ACTION_BACKLOOK` | `0x1028` | `BRAINWASH` | `0x1038` | `NAROHYSEVYSER` |
| `0x1009` | `COMPAREOWNERWITH` ² | `0x1019` | `ACTION_RUNCLOSERTO` | `0x1029` | `KICKDRIVER` | `0x1039` | `ACTION_PHONECALL` |
| `0x100a` | `FINDCORNER` ¹ | `0x101a` | `VIDIM` | `0x102a` | `GROUP_NEW` | `0x103a` | `NAPRDELIVAUTE` |
| `0x100b` | `MOVE_TO_CAR` | `0x101b` | `EXIT_RAILWAY` | `0x102b` | `GROUP_ADD` | `0x103b` | `FOLLOWPLAYER` |
| `0x100c` | `CAR_HUNT` | `0x101c` | `MOVE_TO_FRAME` | `0x102c` | `GROUP_DEL` | `0x103c` | `AZ_PO_PANICE_HOVNO` |
| `0x100d` | `CAR_ESCAPE` | `0x101d` | `GETSTATE` | `0x102d` | `ARREST_PLAYER` | `0x103d` | `NEVYHAZUJ` ³ |
| `0x100e` | `CAR_MOVETO` | `0x101e` | `BOX_GOTO` | `0x102e` | `NASERSE` | `0x103e` | `IGNORE_CORPSE` |
| `0x100f` | `BLUESCREEN` ¹ | `0x101f` | `BOX_TAKE` | `0x102f` | `ACTIONSCLEAR` | `0x103f` | `BLASTFIRE` |

¹ `ENTITY_` form only. ² The bare keyword `COMPAREOWNERWITH`. ³ `ENEMY_` form only (listed twice).

The `+0xf50` and lock writes of the command arms: `VIDIM` with 1 sets `+0xf50` bit 1 and with 0
clears it; `SETSTATE` forces an AI state through `entity_force_ai` and then writes `+0xf50 = 2`;
`ARREST_PLAYER` sets bit 1; `BRAINWASH` re-inits the sensors, empties the substate-lock vector and
the `ai_logs` list, and resets `+0x10c4` to 666 and `+0x10c8` to 0.

## 6. The `ai_action` type table (35 factory types)

Every `ai_action` carries its TYPE at offset 0 of its `0x24`-byte header. `entity_action_tick`
switches on exactly this value across **40 outer cases**, and 35 of those types are built by one
factory each; the PC and PS2 factory sets join **1:1 on the type id**. This is the NPC AI action
vocabulary — what an AI-driven person in Mafia can be told to do. Only types **0x21** and **0x28**
are police-specific; `0x18..0x1c` are the **cover system**, the AI-action side of the C_human
`Do_Corner*` family.

**The table below is the FACTORY set, not the whole type space.** Five more type ids have
dispatcher arms and no factory — see below.

| type | PC factory (`ai_action_new_*`) | PS2 factory | inner (PC) | role |
|---|---|---|---|---|
| 0x00 | `_move` | `NewMove` | 0x20 | walk/run to a world position |
| 0x01 | `_follow` | `NewFollow` | 0x14 | follow an actor |
| 0x05 | `_turn_to` | `NewTurnTo` | 0x04 | rotate to face an actor |
| 0x06 | `_fire` | `NewFire` | 0x0c | shoot at an actor |
| 0x08 | `_follow_link` | `NewFollowLink` | 0x18 | follow along a path link |
| 0x09 | `_blocked_way` | `NewBlockedWay` | 0x08 | react to a blocked route |
| 0x0a | `_runaway_dist` | `NewRunawayDist` | 0x20 | flee until a distance is reached |
| 0x0b | `_side_look` | `NewSideLook` | 0x18 | glance sideways |
| 0x0c | `_back_look` | `NewBackLook` | 0x20 | glance behind |
| 0x0d | `_unlink` | `NewUnLink` | 0x04 | detach from a path link |
| 0x10 | `_run_closer_to` | `NewRunCloserTo` | 0x24 | approach an actor |
| 0x12 | `_freeze` | `NewFreeze` | 0x04 | stand still |
| 0x14 | `_move_to_car` | `NewMoveToCar` | 0x28 | go to a car (the boarding path) |
| 0x15 | `_step_away` | `NewStepAway` | 0x14 | step aside |
| 0x16 | `_strafe` | `NewStrafe` | 0x1c | strafe |
| 0x17 | `_jump` | `NewJump` | 0x04 | jump |
| **0x18** | `_corner_take_pos` | `NewCornerTakePos` | 0x40 | **cover** — take a cover position |
| **0x19** | `_corner_look` | `NewCornerLook` | 0x10 | **cover** — peek from cover |
| **0x1a** | `_corner_cover_fire` | `NewCornerCoverFire` | 0x10 | **cover** — suppressing fire from cover |
| **0x1b** | `_corner_fire` | `NewCornerFire` | 0x10 | **cover** — fire from cover |
| **0x1c** | `_corner_fight` | `NewCornerFight` | 0x48 | **cover** — full fight-from-cover behaviour |
| 0x1d | `_crouch_fight` | `NewCrouchFight` | 0x40 | fight while crouched |
| 0x1e | `_normal_fight` | `NewNormalFight` | 0x58 | default stand-up firefight |
| 0x1f | `_barricade_fight` | `NewBarricadeFight` | 0x1c | fight from behind a barricade |
| 0x20 | `_follow_in_rail` | `NewFollowInRail` | 0x14 | follow a human along a rail/track |
| **0x21** | `_police_action` | `NewPoliceAction` | 0x60 | **police** — the fine/arrest action (drives the busted cutscene and the HUD police symbol) |
| 0x22 | `_search` | `NewSearch` | 0x34 | search for a lost target |
| 0x23 | `_kick_driver` | `NewKickDriver` | 0x28 | the carjack eject (cf. `human_do_throw_occupant_from_car`). The ctor's FIRST parameter is the `C_human` to be thrown out: it goes to `inner+0x20`, and `inner+0x1c` caches THAT human's vehicle, not the running entity's |
| 0x24 | `_hostile_coverage` | `NewHostileCoverage` | 0x84 | biggest action; takes a `vector<C_actor*>` of hostiles |
| 0x25 | `_car_jump` | `NewCarJump` | 0x10 | jump out of the way of a car |
| 0x26 | `_car_stop` | `NewCarStop` | 0x20 | make a car stop |
| 0x27 | `_car_aim` | `NewCarAim` | 0x20 | aim at or from a car |
| **0x28** | `_police_call` | `NewPoliceCall` | 0x30 | **police** — radio for backup (the callable-device path) |
| 0x29 | `_strafe_him` | `NewStrafeHim` | 0x0c | strafe relative to a human |
| 0x2a | `_strafe_beware` | `NewStrafeBeware` | 0x30 | strafe while avoiding a friendly line of fire (takes an `ai_fire_area`) |

**`entity_get_action_size` is not a cross-check of this table.** Its case set adds `3`, `7`, `0x11`,
`0x13` and omits `0x14`, `0x21`, `0x24`, `0x25`, `0x26`, `0x27`, `0x28`, which fall through to its
`return 0` — i.e. **those seven action types are not persisted at all**, a separate finding about the
savegame. Its values are also SAVE widths, not inner-payload sizes: type `5` allocates an inner of
**4** and sizes at `0x20`; type `6` allocates **0xc** and sizes at `0x28`. Compare it with the
savegame writer, never with the `inner (PC)` column.

### 6a. The `ai_action` object layout

Proven by all 35 factories at once — every one builds the same header, and the type id it stamps at
offset 0 is what makes the 1:1 join possible.

```c
typedef struct ai_action {          // 0x24-byte header + a per-type inner payload
    /* +0x00 */ uint32_t type;      // [H] THE TYPE ID (0x00..0x2a); entity_action_tick switches on it (§6)
    /* +0x04 */ uint32_t uid;       // [H] from ai_action_generate_UID()
    /* +0x08 */ uint32_t owner_uid; // [H] the job this action belongs to, the factory's uid argument
    /* +0x0c */ uint32_t parent_uid;// [H] the same argument at creation, 0 for a root
    /* +0x10 */ uint32_t time_ms;   // [H] a phase stamp, an age base or a deadline, by type (entity_action_tick)
    /* +0x14 */ uint32_t inner_size;// [H] size of the inner payload (varies per type - see §6)
    /* +0x18 */ uint32_t signal_bits;// [H] 0 from every factory; ai_action_manager_signal_children ORs a mask in
    /* +0x1c */ uint32_t last_tick_result; // [H] the previous tick's result 0..3 (entity_actions_tick); 0 from every factory
    /* +0x20 */ void    *inner;     // [H] the per-type payload (a second operator_new)
} ai_action;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | type | H | the 35 `ai_action_new_*` factories | `entity_action_tick` (the outer switch), `entity_get_action_size`, `entity_get_target`, `entity_is_in_danger` | the type id of §6; each factory stamps it as `*outer = <TYPE>` |
| +0x04 | uid | H | the 35 factories (`ai_action_generate_UID`) | `entity_action_tick` (cases `0xa`, `0x10`, `0x11`, `0x1d`, `0x1e`, `0x21`, `0x22`, `0x26`, `0x28`) | the owner uid of every child action those cases build and of `ai_action_manager_signal_children` |
| +0x08 | owner_uid | H | the 35 factories (their uid argument) | `ai_action_manager_retire_top_job`, `ai_action_manager_signal_children` | `entity_action_tick` does not read it |
| +0x0c | parent_uid | H | the 35 factories, the adopt functions, `ai_action_manager_derive_job` | - | 0 for a root; its readers among the `ai_action_manager` methods are not read for this doc (`structs/ai_action_manager.md`) |
| +0x10 | time_ms | H | the 35 factories (the world clock), `entity_action_tick` (cases 0, 1, 8, `0xa`..`0xc`, `0x10`, `0x11`, `0x14`, `0x15`, `0x1f`, `0x23`) | `entity_action_tick` (cases 0, 1, 3, 8, `0xa`..`0xc`, `0x10`..`0x16`, `0x1f`, `0x23`, `0x26`..`0x28`) | by type a phase stamp (0, 1, 8, `0x14`, `0x15`, `0x1f`, `0x23`), an age base (3, `0x11`, `0x13`, `0x1f`, `0x23`) or a deadline (`0xa`, `0xb`, `0xc`, `0x10`, `0x12`); 0 means never planned in cases 0, 1, 8, `0x14`, `0x26`..`0x28` |
| +0x14 | inner_size | H | the 35 factories | - | the inner payload's size per type (§6); no reader read for this doc |
| +0x18 | signal_bits | H | the 35 factories (0), `ai_action_manager_signal_children` (ORs its mask) | `entity_action_tick` (bit 0: cases `0xb`, `0xc`, `0x22`, `0x24`; bit 7: cases `0x25`..`0x27`) | `entity_action_tick` sends mask 1 from cases `0xa`, `0x10` and case `0x21` substate 9, and mask `0x80` from case `0x21` substate `0xa` |
| +0x1c | last_tick_result | H | the 35 factories (0), `entity_load_action_man`, `entity_actions_tick` (the tick's result 0..3, top job only) | `entity_save_action_man`, `entity_action_tick` (case `0xa` twice, case `0x10`) | `== 2` with mode 0 makes cases `0xa` and `0x10` call `human_pose_null_rotate` and `ai_action_manager_signal_children(1, uid)` |
| +0x20 | inner | H | the 35 factories (a second `operator_new`) | `entity_action_tick` (every arm), `entity_get_target` | the per-type payload |

**The type id is the invariant across builds, not the inner size** (§9).

### 6b. The five producerless action types — `0x03`, `0x07`, `0x0e`, `0x11`, `0x13`

40 outer cases − 35 factories = **5 types the dispatcher handles but that nothing
constructs**:

| type | where it is handled | what its arm does |
|---|---|---|
| `0x03` | a `case` in `entity_get_action_size`; a full arm in `entity_action_tick` | **aims the body pose at an actor**: once 500 ms have passed since the action started, it takes the actor in payload `+0x00`, adds 1.5 to its frame's world height and calls `human_pose_set_pose_normal`, or `human_pose_set_pose_aimed` when the human's `+0x1e5` byte is set |
| **`0x07`** | a `case` in `entity_get_action_size`; a full arm in `entity_action_tick` | **opens a door** — `inner+0x00` is a `C_door *` (`structs/door.md` §7b) |
| `0x0e` | an outer `case` in `entity_action_tick` | **waits on payload `+0x00`**: the action completes once that dword is 0 and keeps running while it is not |
| `0x11` | a `case` in `entity_get_action_size` and in `ai_action_invalidate_path_for_actor`'s case set; a full arm in `entity_action_tick` | **an idle look-around, on foot only**: a random roll against the time since its last trigger queues a 2000 ms `ai_action_new_back_look` to a random side or a 2000 ms `ai_action_new_side_look` to the right or left as an idle action, then restarts its clock; in a vehicle it waits |
| **`0x13`** | a `case` in `entity_get_action_size`; a full arm in `entity_action_tick` | **closes a door once clear of it**, on a time + distance gate (`structs/door.md` §7b) |

No code constructs any of the five:

- every action takes its UID from `ai_action_generate_UID`, and a byte census of the `call` and
  `jmp` instructions that reach it returns exactly **35**, one in each named factory, none of
  which stamps one of these types;
- exactly **35** PC functions allocate a `0x24`-byte `ai_action`, and all 35 are the named factories
  (the other five `0x24`-allocators in the binary are unrelated — a float-const setter, a ring
  buffer, and so on);
- there is no in-place mutation of the type field anywhere in the dispatcher;
- a scan of the whole `.text` for `mov dword [reg], imm32` with one of the five values finds 11
  sites, all in script-parameter decoding, the police board, the rail classes and two GUI key
  handlers.

The five arms can therefore only receive an action that `entity_load_action_man` restores from a
savegame, and no running game creates one to save. For the two door arms this matches the game:
NPCs open doors through the path follower and the use-action (`structs/door.md` §7b), and in game
NPCs are seen opening doors and never closing one.

## 7. The `C_entity` AI sub-objects and the per-frame perception chain

A `C_entity` is an **aggregate of AI sub-objects**, each its own class with its own method family.
Every base below is byte-proven from the receiver setup at the callsites — `lea ecx,[this+N]` /
`add ecx,N` = **embedded**.

| PC base | class | methods | doc |
|---|---|---|---|
| `+0xa9c` | `C_program` (script VM) | - | §4 |
| `+0xb04` | `ai_vis_logs` | 4 | below |
| `+0xbb4` | `ai_sensors` | 23 | `structs/ai_sensors.md` |
| `+0xe88` | `ai_logs` | 14 | `structs/ai_logs.md` |
| `+0xebc` | `ai_character` | 6 | `structs/ai_character.md` |
| `+0xf64` | `ai_fight_modes` | 13 | `structs/ai_fight_modes.md` |
| `+0x10cc` | `ai_talkman` (speech scheduler) | 4 | `structs/ai_talkman.md` — four talk slots; `entity_ai` clears the per-frame request bits, the actions request talk groups, the tick plays them |
| `+0x1164` | `ai_action_manager` | 58 | `structs/ai_action_manager.md` — 58/58 accounted; the object is 0x34 bytes: three `std::vector<ai_action*>` (jobs `+0x04`, idles `+0x14`, modifiers `+0x24`) plus the action-UID counter at `+0x30` |
| `+0x1268` | `ai_ghost` (collision exemptions) | 5 | `structs/ai_ghost.md` — the blockers of the NPC's current detour, which its movement collision ignores for 0.5–2 s; not a position-history trail |

### The per-frame perception chain

`entity_ai` (vtable slot 13) runs this triple **19×** — once per branch of its state machine:

```
  lea ecx,[esi+0xe88] ; call ai_logs_tick
  lea ecx,[esi+0xb04] ; call ai_vis_logs_tick
  mov eax,[0x65115c] ; push esi ; mov ecx,[eax+0x24]
  add ecx,0x2b3c     ; call sound_arena_update_entity_sound_contacts
```

The PS2 build throttles the middle call instead of running it per entity — §9.

### `ai_vis_logs` (`+0xb04`) — 4 methods

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00410670 | `ai_vis_logs_init` | `Init__11ai_vis_logsFPv` | 0x0030fa80 | CONFIRMED | takes an owner backpointer; seeds the slots with 666.0f and `+0xa8` with 0x29a |
| 0x004106c0 | `ai_vis_logs_add_entity` | `AddEntity__11ai_vis_logsFPvfb` | 0x0030fb50 | CONFIRMED | shared magic constants |
| 0x00410770 | `ai_vis_logs_del_entity` | `DelEntity__11ai_vis_logsFPvb` | 0x0030fc50 | CONFIRMED | shared magic constants |
| 0x004107d0 | `ai_vis_logs_tick` | `Tick__11ai_vis_logsFv` **+ `TickCorpses__11ai_vis_logsFv` FUSED** | 0x0030fce0 / 0x00310c90 | CONFIRMED | 3328 B. **PC fuses the two PS2 functions into one**, proven by the callee multiset: it calls Tick-only callees (`NewVisualCon`, a direct `ai_character_get_sight`) *and* TickCorpses-only callees (`Event__8C_entity`, `hasLog__7ai_logs`), and the union counts match exactly (`GetVisibleRange` 2+2 = **4**; `GetLog` 1+2 = **3**; `TestLineS` 1+1 = **2**) |

## 8. The translation unit

The `C_entity` code lies between the CRT static-init guard `crt_static_init_guard_0040f700` (with
its registrar `crt_atexit_register_0040f720`) and the next guard, `crt_static_init_guard_0043b000`:
`[0x0040f700, 0x0043b000)`. A guard-to-guard range is a lower bound on what one object file holds
and can also span object files that have no static initializer of their own. Inside it:

* the AI sub-object classes of §7: `ai_sensors` and `ai_vis_logs` before `entity_ctor`, then
  `ai_action_manager`, `ai_character`, `ai_fight_modes`, `ai_talkman`, `ai_space_coverage` and
  `ai_ghost` between the `C_entity` methods;
* between `entity_action_tick` and `entity_actions_tick`, a block of inline accessors of other
  classes (`hud_*`, `i3d_frame_*`, `actor_*`, `vehicle_*`, `game_get_*`, `ai_path_*`,
  `ai_wayfind_*`, `door_get_state`, `human_*`) and the vector-math COMDATs of §3d;
* the STL and vector helpers of §3d at the tail.

The four `program_base_thunk_entity_*` adjustor thunks sit just past the closing guard at
`0x0043b030`..`0x0043b060`, and the four header-inline `entity_*` functions at
`0x005c8fa0`..`0x005c9040` were emitted inside the `C_program` translation unit's band.

## 9. PC vs PS2

**Element types named by PS2 signatures (§2).** PS2's `insert__…<ai_substate_lock>` template
argument names the `+0x10b8` row type. `AddBlockedCon__8C_entityFP11blocked_con` names the
`+0x119c` element type outright, and the PS2 `actionTick` / `actionTick_2` pair calls that adder ×20
(6 + 14), as the PC `entity_action_tick` does.

**C_entity band deltas.** The PC→PS2 delta **varies per band** — the layout was reorganised for the
port, so never rebase across a band boundary:

| PC | PS2 | delta | band |
|---|---|---|---|
| `+0xa9c` | `+0xbf0` | 0x154 | `C_program` |
| `+0xb04` / `+0xbb4` | `+0xc50` / `+0xd00` | 0x14c | `ai_vis_logs`, `ai_sensors` |
| `+0xc74`/`+0xc78` | `+0xdd0`/`+0xdd4` | - | the sound-contact ring: PS2 stride 0x30 against PC 0x20, and it sits at PS2 `ai_sensors+0xd0` against PC `+0xc0` — 3 S_vectors of drift |
| `+0xe88` | `+0x1060` | 0x1d8 | `ai_logs` |
| `+0xebc` | `+0x1090` | 0x1d4 | `ai_character` |
| `+0xf08` | `+0x10f0` | 0x1e8 | the flag word |
| `+0xf40` / `+0xf4c` / `+0xf50` | `+0x1128` / `+0x1134` / `+0x1138` | 0x1e8 | AI state band |
| `+0xf64` | `+0x114c` | 0x1e8 | `ai_fight_modes` (its internal layout has zero delta) |
| `+0x10b8` / `+0x10c4` / `+0x10c8` | `+0x129c` / `+0x12a8` / `+0x12ac` | 0x1e4 | substate locks (PS2 renders count @`+0x12a0`, data @`+0x12a4`) |
| `+0x10cc` | `+0x12b0` | 0x1e4 | `ai_talkman` |
| `+0x1164` | `+0x1348` | 0x1e4 | `ai_action_manager` |
| `+0x119c` / `+0x11a0` | `+0x1370` / `+0x1374` | 0x1d4 | blocked connections |
| `+0x11b4` | `+0x1390` | 0x1dc | head_dir |
| `+0x11c0` | `+0x13a0` | 0x1e0 | head_pos |
| `+0x11cc` | `+0x13b0` | 0x1e4 | neck-bone cache |
| `+0x11d0`..`+0x11ec` | `+0x13b4`..`+0x13d0` | 0x1e4 | weapon band (the pre-proven anchor) |
| `+0x1268` | `+0x1460` | 0x1f8 | `ai_ghost` (PS2 keeps `begin` at base+0x00; records stride 0x20 with the timestamp at `+0x10`) |

The head band walks by +4 per vector (`0x1dc` → `0x1e0` → `0x1e4`) because a PS2 `S_vector` is 16 B
where a PC vec3 is 12 B — the same drift that makes the `ai_ghost` record 0x20 on PS2 against 0x1c
on PC.

**The flag word is PS2 `C_entity+0x10f0`.** `CallBack` sets `0x200` / masks `0xfffff1ff` there,
`Sniff__10ai_sensors` tests `& 0x4000` and `ScriptSwitch` runs `if (f & 0x200) f |= 0x400` on it. PS2
reaches `CallBack` through an `@3056@` adjustor thunk, so its body is entity-relative; the PC calls
the override directly on the program subobject, which is why the PC body spells the word
`+0x46c` (§2b, §4). PS2 `GameInit` also sets bit `0x400000` when the type group is 0; the PC has no
writer of that bit.

**The `EntityBlock` pair and the police-board pair.** PS2 `EntityBlock__6C_gameFbb` calls
`SedniNaPrdelASledujKino(bool)` to block and `VzbudSeUzSouTitulky()` to unblock every `C_entity`;
both bodies are empty on PS2 (`jr ra`). Its PC twin `game_entity_block` makes the same two calls in
the same branches to `entity_suspend_ai_and_hide(char)` / `entity_resume_ai_and_show()`, their only
caller, so the pairs are twins with the PS2 bodies gutted; graded STRUCTURAL because an empty body
cannot be diffed. The other PS2 pair, `NeserADejchej` / `NeserANedejchej`, adds / subtracts 10000 and
updates `ai_sight_manager`; its only callers are the PS2 police board's `VseckyUbozakyStopni` and
`VseckyUbozakyRozejdi`, which walk the `C_entity` actors. Their PC twins are
`police_board_suspend_npcs` / `police_board_resume_npcs`, statement for statement, which open-code
the ±10000 with no call (`structs/police-board.md` §8). The two bumps are distinct on both builds: PS2 `SightFilter` strips 10000 once and then 666
repeatedly before it switches on the state.

**The eight PS2 methods with no PC function.** They are counted among the 110 of §3:

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00339ba0 | `ChangeState__8C_entityFv` | 160 B, inlined into its callers on PC: the `entity_force_ai → entity_dispatch_event` gap contains exactly 3 real PC functions and all 3 are body-confirmed as `isFakeScript` / `ForceScript` / `CallBack` |
| 0x0033fd40 | `InvalidatePaths__8C_entityFv` | 12 B; sole caller `InterCity_ChangeMission__9C_missionFv`, part of the PS2's city streaming |
| 0x00347c60 | `canAim__8C_entityFUi` | 320 B, a frame-budget-gated, 1000 ms-cached line-of-sight test (below) |
| 0x00349f40 | `NeserADejchej__8C_entityFv` | adds 10000 to the AI state; open-coded on PC in `police_board_suspend_npcs` |
| 0x00349f60 | `NeserANedejchej__8C_entityFv` | subtracts 10000; open-coded on PC in `police_board_resume_npcs` |
| 0x0034a290 | `UpdateAttention__8C_entityFv` | 3144 B; builds the per-human attention-delay table (below) |
| 0x0034aff0 | `GetAttentionDelay__8C_entityFP7C_human` | 92 B; reads that table |
| 0x0034b050 | `SightFilter__8C_entityFv` | 240 B; called only by the two ticks of `ai_sight_manager`, itself PS2-only (below) |

**The PS2-only members.** `SightFilter` belongs to `ai_sight_manager` (below).
`InvalidatePaths` is called only by `InterCity_ChangeMission`, part
of the PS2's city streaming. `canAim` opens with `ai_framerater::GetFPS` (the frame-budget throttle
the PC does not have) and caches its line test in PS2 `+0x1480`; the PS2 `actionTick` /
`actionTick_2` pair calls it 8 times, and `entity_action_tick` makes no collision call at all. `UpdateAttention` builds a
`{delay, C_human *}` table at PS2 `+0x1484` that `GetAttentionDelay` reads from `ai_vis_logs::Tick`
(default 668 ms, +200 in one state); the PC's fused `ai_vis_logs_tick` has no such lookup, and the
3144-byte builder cannot be inlined into the PC `entity_update` (256 B) or `entity_game_init`
(1664 B).

**`C_program` internals differ.** Only the vector base at `program+0x10` coincides:

| | PC (`entity_script_is_fake`) | PS2 (`isFakeScript`) |
|---|---|---|
| instruction list | `{begin @ +0x10, end @ +0x14}`, count **computed** as `(end−begin)>>3` | `{count @ +0xc, base @ +0x10}` — count **stored** |
| pc | `+0x1c` | `+0x14` |
| cleared on the fake path | `+0x60` (= entity `+0xafc`) | `+0x54` (= entity `+0xc44`) |

PS2's `C_program` sits at `C_entity+0xbf0`, proven by three methods that take the program as `this`
(`IsEventID`, `SetEvent`, `CallProcess`, all reading `+0xc`/`+0x10`/`+0x14`) plus `isFakeScript`,
which inlines the same access entity-relative at `+0xbfc`/`+0xc00`/`+0xc04`.

**`GetProgram`'s skew differs per build.** PS2 returns `program + 8` on every class that overrides it
(`C_detector` `+0x80` → `+0x88`, `C_entity` `+0xbf0` → `+0xbf8`; `C_actor`'s override returns 0):
on both builds the getter returns the instruction vector's header, which the PS2 build keeps at
`program+0x8` (count at `+0xc`, base at `+0x10`) and the PC at `program+0xc` (§4). The PC skew is
`0xc` (`+0xa9c` → `+0xaa8`). Use `0xc` for PC callsites and `8` for PS2 ones, and do not carry
one across. The `C_detector` case is what settles the PS2 side: `AI__10C_detector` reads `+0x94` (pc)
and `+0x8c` (count) and then calls `CallProcess__9C_program` with `a0` set to `this + 0x80` in the
delay slot — so the program is at `+0x80` while the getter returns `+0x88`.

**Perception is throttled on PS2 and not on PC.** PS2's `AI__8C_entity` calls `Tick__11ai_vis_logs`
**zero** times. Instead a central **`ai_sight_manager`** (7 methods: `Init` / `Add(C_entity*)` /
`Del` / `Next` / `NextCorpses` / `Tick` / `TickCorpses`), driven by `Tick__10ai_manager`,
round-robins: each frame it advances a cursor and ticks only **one** entity's vis_logs. PS2's
`GameInit__8C_entity` calls `Add__16ai_sight_manager` ×2 and `GameDone` calls
`Del__16ai_sight_manager` + `Del__16ai_panic_manager`, none of which has a PC equivalent.
**`ai_sight_manager` is PS2-only.** Outside its own class it is called by `ai_manager::Init` /
`Destroy` / `Tick`, by `C_entity::GameInit` / `GameDone` / `Death` / `ChangeState` /
`NeserADejchej` / `NeserANedejchej`, and by five `C_shvestky` methods. `ChangeState`, the two
`Neser*` and `Tick__10ai_managerFUi` have no PC function, and the PC twins of the others drop
exactly the sight-manager statement:

| PC function | PC statements | PS2 twin | PS2 statements |
|---|---|---|---|
| `police_manager_free_temp_enemy` | `human_deactivate`, `entity_reset_state` | `FreeTempEnemy` | `Deactivate`, `if (kind == 0x1b) Del`, `ResetState` |
| `entity_game_done` | ..., `using_object_del_object` | `GameDone` | ..., `DelObject`, sight `Del`, panic `Del` |
| `entity_death` | ..., `ai_fight_manager_leave_party` | `Death` | ..., `LeaveParty`, sight `Del`, panic `Del` |

The PC `ai_manager` keeps a code-less two-vector member where the PS2 places the sight manager
(`structs/ai_manager.md` §7). Gameplay consequence, worth an in-game check (§11): on PS2 an NPC's
vision updates only on its turn, so perception latency scales with the number of live NPCs, which
predicts PS2 NPCs are slower to notice the player in crowded scenes.

**The `ai_action` factory set (§6).** PS2's `ai_action_manager` exposes exactly **35** typed
factories, the same set as the PC; `NewJob` / `NewModifier` / `NewIdle` are **install wrappers**
taking an *existing* `ai_action*`, not factories. `GenUID__17ai_action_managerFv` has exactly 35
callers, all of them `New*` factories, so the PS2 has no producer for the five factory-less types
either. Inner payload sizes differ per build (e.g. type
`0x24`: PS2 `0x90` against PC `0x84`) — expected MIPS-vs-x86 packing.

**Other structural differences in the tables:** the PC fuses `Tick` + `TickCorpses`
into one `ai_vis_logs_tick` (§7); PS2 splits `actionTick` into two functions where MSVC emitted one
(§3); PS2's `ChangeState` is inlined on PC (below); the savegame "State" constant differs per build
(PC `0x168`, PS2 `0x14c`).

## 10. Related

- `structs/actor.md` (the `C_actor` base), `structs/human.md` (the `C_human` base).
- `structs/ai_action_manager.md` — the `+0x1164` action-queue sub-object.
- `structs/ai_sensors.md`, `structs/ai_character.md`, `structs/ai_logs.md`,
  `structs/ai_fight_modes.md`, `structs/ai_ghost.md`, `structs/ai_blob_log.md` — the AI sub-objects.
- `structs/program.md` — the `C_program` script VM.
- `structs/entity_kind.md` — the kind enum: `0x1b` is `C_entity`, and the script name `Enemy`.

## 11. Open items

| Item | State |
|---|---|
| PS2 perception latency | the round-robin predicts PS2 NPCs notice the player later in crowded scenes than PC NPCs; not tested in game (§9) |
