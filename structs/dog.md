# `C_dog` — the guard dog

**Created** 2026-08-13 12:57 +0200 · **Modified** 2026-11-09 20:58 +0100

A patrolling guard dog. It walks a route of `C_web_node`s, remembers which connections
it has found blocked, re-plans through `ai_wayfind` when it gets stuck, and switches to
an attack state when the player comes close enough. Hurt, it leaves blood stains; after
stepping into blood it leaves ten fading bloody paw prints (§6). It can be decapitated, and a
vehicle at speed kills it in one hit.

Its whole behaviour hangs off one field, the **state at `+0xb4`** (§2a). `dog_enter_state` writes it and picks the
animation that goes with the state; `dog_ai` makes the two moves 6 -> 8 and 8 -> 7 by itself. Every other method either reads
the state or feeds it.

| | |
|---|---|
| PC prefix | `dog_` |
| PC kind (`+0x10`) | `0x15` |
| PC vtable | `g_dog_vt` @ `0x0063c0d0` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`), not `C_entity` (§4) |
| PC instance size | `0x310` (entity factory `case 0x15`) |
| PC constructor | `dog_ctor` @ `0x0047ab60` |
| PC translation unit | `[0x0047aae0, 0x004800c0)`, guard to guard: 52 starts, all named. The unit's head holds five `C_detector` functions and its tail the helpers of §9 |
| PS2 class | `C_dog`, 40 methods, 38 mapped |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_dog {                         /* 0x310 bytes */
    /* +0x000 */ C_actor   base;               // [H] kind 0x15 at +0x10, frame at +0x68
    /* +0x070 */ float     walk_speed;         // [H] 1.0; the speed cap in state 1
    /* +0x074 */ float     run_speed;          // [H] 5.5; the speed cap in every other moving state
    /* +0x078 */ float     acceleration;       // [H] 4.0; units per second per second towards the cap
    /* +0x07c */ float     max_health;         // [H] 100.0; dog_game_init copies it into health
    /* +0x080 */ float     bite_damage;        // [H] 40.0; dog_ai passes it to the target's Hit (slot 31) in state 8
    /* +0x084 */ float     acquire_range;      // [H] 20.0; dog_acquire_player_target chases a player closer than this
    /* +0x088 */ bool      starts_hostile;     // [H] 1; dog_game_init copies it into +0x30f
    /* +0x089 */ bool      print_flag;         // [H] set by dog_arm_bloody_paw_prints, cleared at 0 prints
    /* +0x08c */ int       sound_id[5];        // [H] the sound id of each slot; saved and duplicated (§7)
    /* +0x0a0 */ I3D_sound *sound[5];          // [H] the live object of each slot, lazily created (§7)
    /* +0x0b4 */ int       state;              // [H] the behaviour state (§2a)
    /* +0x0b8 */ S_vector  forward;            // [H] unit heading (§2)
    /* +0x0c4 */ byte      unused_c4[0xc];     // [U] no access (.text scan)
    /* +0x0d0 */ float     health;             // [H] dog_hit subtracts damage; dog_on_collision subtracts 100
    /* +0x0d4 */ float     speed;              // [H] the current speed; accelerates to the cap
    /* +0x0d8 */ int       unread_d8;          // [U] the ctor's 0; no reader (.text scan)
    /* +0x0dc */ int       anim_cur;           // [H] last clip id pushed to the machine
    /* +0x0e0 */ int       anim_pending;       // [H] clip id dog_apply_pending_anim plays next (§6)
    /* +0x0e4 */ bool      anim_play_once;     // [H] anim_machine_set_anim's play-once flag
    /* +0x0e5 */ bool      anim_use_track;     // [H] anim_machine_set_anim's track flag: loads <clip>.tck, forces play-once
    /* +0x0e6 */ bool      anim_tick_enabled;  // [H] dog_ai ticks the machine only while set; no store of 0
    /* +0x0e7 */ bool      overlay_active;     // [H] state 0x10: the bark overlay is playing
    /* +0x0e8 */ C_animations_machine_de_luxe anim; // [H] embedded, 0x128 bytes (structs/anim_machine.md)
    /* +0x210 */ I3D_animation_set *overlay_anim;   // [H] Dbark3, opened on entry to state 0x10
    /* +0x214 */ uint      overlay_elapsed;    // [H] ms the overlay has played
    /* +0x218 */ uint      overlay_duration;   // [H] the clip's length, from the object's virtual +8
    /* +0x21c */ uint      unread_21c;         // [U] the ctor's 0; no reader (.text scan)
    /* +0x220 */ uint      unread_220;         // [U] the ctor's 200; no reader (.text scan)
    /* +0x224 */ uint      bleed_elapsed;      // [H] ms towards the next blood drop below a third of max_health
    /* +0x228 */ uint      bleed_period;       // [H] 2000
    /* +0x22c */ uint      wait_elapsed;       // [H] ms spent in state 9
    /* +0x230 */ uint      wait_period;        // [H] 1000 + a random amount, set on entry to state 9
    /* +0x234 */ tDynamicCollObject *dyn_coll_kind2; // [H] owned; word +0x08 = 0x100002; erased by Death / GameDone
    /* +0x238 */ tDynamicCollObject *dyn_coll_kind1; // [H] owned; word +0x08 = 0x100001; erased by Death / GameDone
    /* +0x23c */ S_vector  destination;        // [H] the route goal
    /* +0x248 */ S_vector  ground_normal;      // [H] out-param of dog_snap_to_ground's ray; saved
    /* +0x254 */ S_matrix  inverse_world;      // [H] inverse of the frame's world matrix; both collision objects' +0x40
    /* +0x294 */ S_matrix  world;              // [H] copy of the frame's world matrix
    /* +0x2d4 */ C_web_node *node_cur;         // [H] the current route node, record stride 0x1e
    /* +0x2d8 */ C_web_node *node_prev;        // [H] avoids doubling back
    /* +0x2dc */ uint      ground_elapsed;     // [H] ms towards the next ground ray
    /* +0x2e0 */ uint      ground_period;      // [H] 200
    /* +0x2e4 */ uint      target_elapsed;     // [H] ms towards the next target test
    /* +0x2e8 */ uint      target_period;      // [H] 2000, or 4000 after a lost chase
    /* +0x2ec */ uint      idle_elapsed;       // [H] 0 on entry to state 0xc; no reader (.text scan)
    /* +0x2f0 */ uint      idle_period;        // [H] 300 on entry to state 0xc; no reader (.text scan)
    /* +0x2f4 */ C_actor  *enemy;              // [H] the acquired player; cleared on return to patrol
    /* +0x2f8 */ ai_path  *path;               // [H] owned; destroyed on every re-plan
    /* +0x2fc */ char      blocked_alloc;      // [H] allocator byte of the blocked_con vector; the search's receiver
    /* +0x300 */ blocked_con *blocked_begin;   // [H] the remembered blocked connections: begin
    /* +0x304 */ blocked_con *blocked_end;     // [H] the remembered blocked connections: end
    /* +0x308 */ blocked_con *blocked_cap;     // [H] the remembered blocked connections: capacity
    /* +0x30c */ uint8_t   blood_drops;        // [H] drops spawned by dog_update and dog_hit; stops at 10
    /* +0x30d */ char      print_total;        // [H] the alpha denominator, synced from +0x30e
    /* +0x30e */ char      prints_left;        // [H] armed to 10, one per placed print
    /* +0x30f */ bool      hostile;            // [H] gates dog_update's target test; from +0x088 at GameInit, set by dog_hit
} C_dog;
```

**Why five sounds.** `sound_id` and `sound` are parallel, one entry per sound slot of §7: `dog_create_all_sounds`,
`dog_release_all_sounds` and the ctor's loop all run to 5, and each slot has its own job (footstep, a slot nothing plays, bark,
yelp, attack). `unread_21c` / `unread_220`, a timer pair the ctor sets to 0 / 200 and nothing arms, and
`unused_c4` are kept so that the typedef stays contiguous.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | base | H | `dog_ctor`, `dog_dtor` | `dog_ai`, `dog_save_game_load` | the vtable; kind at `+0x10` from the ctor; `+0x1c`, `+0x24..+0x38`, `+0x68` are `C_actor` fields (`structs/actor.md`) |
| +0x70 | walk_speed | H | `dog_ctor`, `dog_duplicate_from` | `dog_load_data`, `dog_move_toward_target`, `dog_save_data` | - |
| +0x74 | run_speed | H | `dog_ctor`, `dog_duplicate_from` | `dog_load_data`, `dog_move_toward_target`, `dog_save_data` | - |
| +0x78 | acceleration | H | `dog_ctor`, `dog_duplicate_from` | `dog_load_data`, `dog_move_toward_target`, `dog_save_data` | - |
| +0x7c | max_health | H | `dog_ctor`, `dog_duplicate_from` | `dog_game_init`, `dog_load_data`, `dog_save_data`, `dog_update` | - |
| +0x80 | bite_damage | H | `dog_ctor`, `dog_duplicate_from` | `dog_ai`, `dog_load_data`, `dog_save_data` | - |
| +0x84 | acquire_range | H | `dog_ctor`, `dog_duplicate_from` | `dog_acquire_player_target`, `dog_load_data`, `dog_save_data` | - |
| +0x88 | starts_hostile | H | `dog_ctor`, `dog_duplicate_from` | `dog_game_init` | - |
| +0x89 | print_flag | H | `dog_arm_bloody_paw_prints`, `dog_ctor`, `dog_game_init`, `dog_place_bloody_paw_print` | `dog_frame_notify_callback` | `dog_frame_notify_callback` reads it (§8) |
| +0x8c | sound_id | H | `dog_ctor` | `dog_create_sound_slot`, `dog_duplicate_from`, `dog_load_data`, `dog_save_data` | - |
| +0xa0 | sound | H | `dog_create_sound_slot` | `dog_create_all_sounds`, `dog_ctor`, `dog_get_sound`, `dog_hit`, `dog_on_collision`, `dog_release_all_sounds`, `dog_stop_sound` | - |
| +0xb4 | state | H | `dog_ai`, `dog_ctor`, `dog_enter_state`, `dog_game_init`, `dog_save_game_load` | `dog_ai`, `dog_move_toward_target`, `dog_save_game_load`, `dog_save_game_save`, `dog_snap_to_ground`, `dog_update` | §2a |
| +0xb8 | forward | H | `dog_turn_toward_target`, `dog_update` | `dog_ai`, `dog_follow_path`, `dog_game_init`, `dog_turn_toward_target`, `dog_update` | copied from the frame by `dog_update`, rotated at 4 rad/s by `dog_turn_toward_target` |
| +0xc4 | unused_c4 | U | - | - | no access (.text scan) |
| +0xd0 | health | H | `dog_game_init`, `dog_hit`, `dog_on_collision`, `dog_save_game_load` | `dog_hit`, `dog_on_collision`, `dog_save_game_save`, `dog_update` | - |
| +0xd4 | speed | H | `dog_ctor`, `dog_move_toward_target` | `dog_move_toward_target` | - |
| +0xd8 | unread_d8 | U | `dog_ctor` | - | the ctor's 0; no reader (.text scan) |
| +0xdc | anim_cur | H | `dog_apply_pending_anim`, `dog_ctor`, `dog_game_init`, `dog_save_game_load` | `dog_apply_pending_anim`, `dog_save_game_save` | - |
| +0xe0 | anim_pending | H | `dog_ai`, `dog_ctor`, `dog_enter_state`, `dog_game_init`, `dog_move_toward_target`, `dog_save_game_load` | `dog_apply_pending_anim` | - |
| +0xe4 | anim_play_once | H | `dog_ai`, `dog_ctor`, `dog_enter_state`, `dog_game_init`, `dog_move_toward_target`, `dog_save_game_load` | `dog_apply_pending_anim`, `dog_save_game_save` | `dog_apply_pending_anim` calls `anim_machine_set_anim(clip, 0, 0, +0xe4, 4.0, start, +0xe5, 0, 0, 0, -1)`, the argument shape of `pumper_set_anim` (`structs/pumper.md` §2); in the callee's parameter order this is the play-once flag (`structs/anim_machine.md` §2 `stage0_play_once`). 1 in states 2, 3, `0xd`, `0xf`, `0x11` and the attack (`dog_ai` case 8), the clips whose end `dog_ai` waits for |
| +0xe5 | anim_use_track | H | `dog_ai`, `dog_ctor`, `dog_enter_state`, `dog_game_init`, `dog_move_toward_target`, `dog_save_game_load` | `dog_apply_pending_anim`, `dog_save_game_save` | the track parameter of `anim_machine_set_anim`: when set the callee loads `<clip>.tck` and forces play-once (`structs/pumper.md` §2). 1 in states 9 and `0xf` only (the `Dbreath` wait, `Ddeath`) |
| +0xe6 | anim_tick_enabled | H | `dog_ai`, `dog_game_init` | `dog_ai` | both writers store 1 |
| +0xe7 | overlay_active | H | `dog_ai`, `dog_game_init`, `dog_save_game_load` | `dog_ai`, `dog_save_game_save` | - |
| +0xe8 | anim | H | `dog_ctor` | `dog_acquire_player_target`, `dog_ai`, `dog_apply_pending_anim`, `dog_dtor`, `dog_game_init`, `dog_hit`, `dog_return_to_patrol`, `dog_save_game_save` | the receiver of the machine's methods; its own fields are in `structs/anim_machine.md` |
| +0x210 | overlay_anim | H | `dog_ctor`, `dog_enter_state` | `dog_ai`, `dog_enter_state` | - |
| +0x214 | overlay_elapsed | H | `dog_ai`, `dog_ctor`, `dog_enter_state` | `dog_ai` | - |
| +0x218 | overlay_duration | H | `dog_ctor`, `dog_enter_state` | `dog_ai` | - |
| +0x21c | unread_21c | U | `dog_ctor` | - | the ctor's 0; no reader (.text scan) |
| +0x220 | unread_220 | U | `dog_ctor` | - | the ctor's 200; no reader (.text scan) |
| +0x224 | bleed_elapsed | H | `dog_ctor`, `dog_game_init`, `dog_save_game_load`, `dog_update` | `dog_save_game_save`, `dog_update` | - |
| +0x228 | bleed_period | H | `dog_ctor`, `dog_game_init`, `dog_save_game_load`, `dog_update` | `dog_save_game_load`, `dog_update` | - |
| +0x22c | wait_elapsed | H | `dog_ai`, `dog_ctor`, `dog_enter_state` | `dog_ai` | - |
| +0x230 | wait_period | H | `dog_ctor`, `dog_enter_state` | `dog_ai` | - |
| +0x234 | dyn_coll_kind2 | H | `dog_ctor`, `dog_death`, `dog_game_done`, `dog_game_init` | `dog_acquire_player_target`, `dog_ai`, `dog_death`, `dog_game_done`, `dog_game_init`, `dog_snap_to_ground`, `dog_update` | - |
| +0x238 | dyn_coll_kind1 | H | `dog_ctor`, `dog_death`, `dog_game_done`, `dog_game_init` | `dog_death`, `dog_game_done`, `dog_game_init`, `dog_update` | - |
| +0x23c | destination | H | `dog_ctor`, `dog_enter_state`, `dog_set_destination_and_plan` | `dog_ai`, `dog_game_init`, `dog_replan_path`, `dog_save_game_load`, `dog_save_game_save`, `dog_set_destination_and_plan` | - |
| +0x248 | ground_normal | H | `dog_snap_to_ground` | `dog_save_game_load`, `dog_save_game_save`, `dog_update` | written through the ray's out-parameter |
| +0x254 | inverse_world | H | `dog_game_init`, `dog_update` | - | - |
| +0x294 | world | H | `dog_update` | - | no reader of the copy in the class |
| +0x2d4 | node_cur | H | `dog_choose_next_web_node`, `dog_ctor`, `dog_enter_state`, `dog_game_init`, `dog_replan_path`, `dog_return_to_patrol`, `dog_save_game_load` | `dog_choose_next_web_node`, `dog_game_init`, `dog_is_segment_blocked`, `dog_replan_path`, `dog_save_game_save` | - |
| +0x2d8 | node_prev | H | `dog_choose_next_web_node`, `dog_enter_state`, `dog_game_init`, `dog_replan_path`, `dog_save_game_load` | `dog_choose_next_web_node`, `dog_save_game_save` | - |
| +0x2dc | ground_elapsed | H | `dog_ctor`, `dog_save_game_load`, `dog_snap_to_ground` | `dog_save_game_save`, `dog_snap_to_ground` | - |
| +0x2e0 | ground_period | H | `dog_ctor`, `dog_save_game_load`, `dog_snap_to_ground` | `dog_save_game_load`, `dog_snap_to_ground` | - |
| +0x2e4 | target_elapsed | H | `dog_acquire_player_target`, `dog_ai`, `dog_ctor`, `dog_save_game_load`, `dog_update` | `dog_save_game_save`, `dog_update` | - |
| +0x2e8 | target_period | H | `dog_acquire_player_target`, `dog_ai`, `dog_ctor`, `dog_save_game_load` | `dog_save_game_load`, `dog_save_game_save`, `dog_update` | - |
| +0x2ec | idle_elapsed | H | `dog_ctor`, `dog_enter_state`, `dog_game_init` | - | 0 from the ctor, `dog_game_init` and on entry to state `0xc`; no reader (.text scan) |
| +0x2f0 | idle_period | H | `dog_ctor`, `dog_enter_state`, `dog_game_init` | - | 0 from the ctor and `dog_game_init`, 300 on entry to state `0xc`; no reader (.text scan) |
| +0x2f4 | enemy | H | `dog_acquire_player_target`, `dog_ai`, `dog_ctor`, `dog_game_init`, `dog_hit`, `dog_return_to_patrol`, `dog_save_game_load` | `dog_acquire_player_target`, `dog_ai` | - |
| +0x2f8 | path | H | `dog_ctor`, `dog_replan_path`, `dog_set_destination_and_plan` | `dog_ai`, `dog_follow_path`, `dog_move_toward_target`, `dog_replan_path`, `dog_set_destination_and_plan`, `dog_update` | - |
| +0x2fc | blocked_alloc | H | `dog_ctor` | `dog_follow_path`, `dog_replan_path`, `dog_set_destination_and_plan` | the address is the receiver of the vector calls |
| +0x300 | blocked_begin | H | `dog_ctor`, `dog_dtor`, `dog_follow_path` | `dog_dtor`, `dog_game_done`, `dog_game_init`, `dog_is_segment_blocked` | - |
| +0x304 | blocked_end | H | `dog_ctor`, `dog_dtor`, `dog_follow_path`, `dog_game_done`, `dog_game_init` | `dog_follow_path`, `dog_game_done`, `dog_game_init`, `dog_is_segment_blocked` | - |
| +0x308 | blocked_cap | H | `dog_ctor`, `dog_dtor`, `dog_follow_path` | `dog_follow_path` | - |
| +0x30c | blood_drops | H | `dog_ctor`, `dog_game_init`, `dog_hit`, `dog_save_game_load`, `dog_update` | `dog_hit`, `dog_save_game_save`, `dog_update` | - |
| +0x30d | print_total | H | `dog_arm_bloody_paw_prints`, `dog_ctor`, `dog_game_init` | `dog_place_bloody_paw_print` | - |
| +0x30e | prints_left | H | `dog_arm_bloody_paw_prints`, `dog_ctor`, `dog_game_init`, `dog_place_bloody_paw_print` | `dog_arm_bloody_paw_prints`, `dog_place_bloody_paw_print` | - |
| +0x30f | hostile | H | `dog_game_init`, `dog_hit`, `dog_save_game_load` | `dog_save_game_save`, `dog_update` | - |

### 2a. `state`

`dog_enter_state(state, range)` stores `state`, or `rand_range(range)` when `range` is not 0, so `(4, 2)` means walk or run at
random and `(4, 4)` any of states 0 to 3. Clip ids are those of §6. State 4 has no arm in either function.

| Value | Meaning | Evidence |
|---|---|---|
| 0 | run the route, clip `Drun` | `dog_follow_path`; route done -> `(4, 4)`, blocked -> 5 |
| 1 | walk the route, clip `Dwalk`, speed cap `walk_speed` | the same |
| 2 | bark, clip `Dbark1` | clip end -> `(4, 2)` |
| 3 | sit down, clip `Dsit` | clip end -> `0x10` |
| 5 | re-anchor | `dog_enter_state` takes the nearest web node, then `(4, 2)` |
| 6 | chase `enemy`, clip `Dbreath` | re-plans when the enemy moved 0.3; within 1.3 with a clear line -> 8; a lost route -> `target_period` 4000 and back to patrol |
| 7 | attack, clip `Datack` | clip end -> 6, or 3 when the target's `+0x5e` is set |
| 8 | turn to the destination | turned -> 7: a random roll decides whether the target's `Hit` runs with `bite_damage`, and sound slot 4 plays |
| 9 | wait, clip `Dbreath` | `wait_period` elapsed -> 6 |
| 10 (`0xa`) | go back to the point the enemy held | `dog_follow_path`; done -> `dog_acquire_player_target`, blocked -> 5 |
| `0xc` | the state `dog_game_init` leaves, clip `Dbreath` | the next tick -> `(4, 2)` |
| `0xd` | stand up, clip `Dstand` | clip end -> `(4, 2)` |
| `0xe` | hurt, clip `Dhit` | clip end -> `dog_acquire_player_target` |
| `0xf` | dead, clip `Ddeath` | clip end -> virtual `+0x74`, `actor_set_act_state(2)` |
| `0x10` | sitting, clip `Dsitting`, with the `Dbark3` overlay | overlay end -> `0xd` |
| `0x11` | clip `Ddrb` | clip end -> `0xd` |

## 3. Method family

38 of the 40 PS2 methods mapped; the other 2 are in §10. An override of the `C_actor` vtable carries its slot.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0047aae0 | `crt_static_init_guard_0047aae0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0047ab00 | `crt_atexit_register_0047ab00` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x0047ab60 | `dog_ctor` | instance | `__ct__5C_dogFv` | 0x0014c790 | CONFIRMED | construct a C_dog |
| 0x0047ad60 | `dog_scalar_deleting_dtor` | instance | - | - | - | slot 16: MSVC-only construct |
| 0x0047ad80 | `dog_dtor` | instance | `__dt__5C_dogFv` | 0x0014ca50 | CONFIRMED | destroy a C_dog |

### 3b. Accessors

None.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0047ae00 | `dog_save_data` | `SaveData__5C_dogFP7C_chunk` | 0x0014cb20 | CONFIRMED | slot 19: write the dog's designer-set fields into a mission chunk |
| 0x0047af40 | `dog_load_data` | `LoadData__5C_dogFP7C_chunk` | 0x0014ccc0 | CONFIRMED | slot 20: read the designer band back out of a mission chunk |
| 0x0047b160 | `dog_duplicate_from` | `DuplicateFrom__5C_dogFP7C_actor` | 0x0014d100 | CONFIRMED | slot 21: copy another dog's designer band onto this one |
| 0x0047b1c0 | `dog_game_init` | `GameInit__5C_dogFv` | 0x0014d180 | CONFIRMED | slot 25: bring the dog to life when the scene starts |
| 0x0047b680 | `dog_load_tracks` | `LoadTCK__5C_dogFv` | 0x0014d880 | CONFIRMED | load the dog's three keyframe tracks |
| 0x0047b6a0 | `dog_game_done` | `GameDone__5C_dogFv` | 0x0014d8c0 | CONFIRMED | slot 26: tear the dog down at scene end |
| 0x0047b800 | `dog_spawn_severed_head` | `LoseHead__5C_dogFRC8S_vector` | 0x0014db00 | CONFIRMED | detach the dog's head and spawn it as its own falling actor |
| 0x0047bdd0 | `dog_hit` | `Hit__5C_dogF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0014e220 | CONFIRMED | slot 31: subtract the damage, set `hostile`, take a player attacker as `enemy`, spawn blood, and enter a hurt or dead state |
| 0x0047c190 | `dog_death` | `Death__5C_dogFv` | 0x0014e6c0 | CONFIRMED | slot 29: release the dog's world resources on death |
| 0x0047c210 | `dog_ai` | `AI__5C_dogFUi` | 0x0014e7a0 | CONFIRMED | slot 13: the dog's per-frame brain |
| 0x0047c990 | `dog_follow_path` | `SolvePath__5C_dogFUi` | 0x0014f170 | CONFIRMED | advance along the computed route, re-planning when blocked |
| 0x0047d0e0 | `dog_move_toward_target` | `AI_walk_run__5C_dogFUiRC8S_vector` | 0x0014fac0 | CONFIRMED | step along the route at `speed`, accelerating to the state's cap; refuses a step `dog_test_move_collision` blocks |
| 0x0047d450 | `dog_turn_toward_target` | `AI_turn__5C_dogFUiRC8S_vector` | 0x0014ff40 | CONFIRMED | rotate `forward` towards a point at 4 rad/s; returns 1 once within 0.349 rad (20 degrees) |
| 0x0047d7f0 | `dog_enter_state` | `ChooseAction__5C_dogFQ25C_dog7E_stateUc` | 0x00150320 | CONFIRMED | enter a behaviour state and select the animation that goes with it |
| 0x0047dba0 | `dog_update` | `Update__5C_dogFUi` | 0x00150900 | CONFIRMED | slot 14: per-frame transform and animation push |
| 0x0047e0e0 | `dog_apply_pending_anim` | `SetAnim__5C_dogFf` | 0x00150d90 | CONFIRMED | push the pending animation to the machine if it changed |
| 0x0047e1e0 | `dog_test_move_collision` | `TestCollision__5C_dogFR8S_vector` | 0x00150ed0 | CONFIRMED | test a proposed step against the world |
| 0x0047e480 | `dog_choose_next_web_node` | `ChooseNextCheckPoint__5C_dogFb` | 0x001511b0 | CONFIRMED | pick the next patrol node from the current node's connections |
| 0x0047e740 | `dog_is_segment_blocked` | `IsBlocked__5C_dogFUi` | 0x00151600 | CONFIRMED | has this dog found the connection to a given node blocked? |
| 0x0047e7f0 | `dog_snap_to_ground` | `ground__5C_dogFUi` | 0x001516e0 | CONFIRMED | drop the dog onto the surface beneath it, on a 200 ms timer |
| 0x0047e900 | `dog_frame_notify_callback` | `dog_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` | 0x00151820 | CONFIRMED | a free function on both builds (§10), installed by `dog_game_init` on the frame named `notify` |
| 0x0047eaf0 | `dog_place_bloody_paw_print` | `BloodStep__5C_dogFUi` | 0x00151a80 | CONFIRMED | place one paw print under the frame the event names and decrement `prints_left` (§8) |
| 0x0047eda0 | `dog_create_all_sounds` | `InitSounds__5C_dogFv` | 0x00151c80 | CONFIRMED | create all five of the dog's sound objects |
| 0x0047ee10 | `dog_create_sound_slot` | `InitSound__5C_dogFUi` | 0x00151d70 | CONFIRMED | create one of the dog's five sound objects by index |
| 0x0047eeb0 | `dog_release_all_sounds` | `ReleaseSounds__5C_dogFv` | 0x00151e60 | CONFIRMED | release all five sound objects |
| 0x0047eee0 | `dog_get_sound` | `GetSound__5C_dogFUi` | 0x00151ee0 | CONFIRMED | fetch one sound object, creating it on first use |
| 0x0047ef20 | `dog_stop_sound` | `StopSound__5C_dogFUi` | 0x00151f30 | CONFIRMED | stop one of the dog's sounds |
| 0x0047ef40 | `dog_return_to_patrol` | `ReturnFromAttackAI__5C_dogFv` | 0x00151f80 | CONFIRMED | drop the attack and go back to walking the route |
| 0x0047efb0 | `dog_acquire_player_target` | `TestEnemy__5C_dogFv` | 0x00152010 | CONFIRMED | look for the player and, if close enough, make him the target |
| 0x0047f290 | `dog_replan_path` | `UnblockPath__5C_dogFv` | 0x00152410 | CONFIRMED | throw away the current route and plan a fresh one to the same destination |
| 0x0047f3a0 | `dog_set_destination_and_plan` | `UpdatePath__5C_dogFRC8S_vector` | 0x001525d0 | CONFIRMED | set a new destination and plan a route to it |
| 0x0047f490 | `dog_save_game_get_size` | `SaveGameGetSize__5C_dogFv` | 0x00152750 | CONFIRMED | slot 22: how many savegame bytes this dog needs |
| 0x0047f4a0 | `dog_save_game_save` | `SaveGameSave__5C_dogFPPv` | 0x00152770 | CONFIRMED | slot 23: write the dog into a savegame |
| 0x0047f680 | `dog_save_game_load` | `SaveGameLoad__5C_dogFPPv` | 0x001529a0 | CONFIRMED | slot 24: read the dog back out of a savegame |
| 0x0047f950 | `dog_on_collision` | `Collision__5C_dogFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00152ff0 | CONFIRMED | slot 30: something ran into the dog |
| 0x0047f9e0 | `dog_arm_bloody_paw_prints` | `cbCollTester__5C_dogFP18tDynamicCollObject` | 0x001530d0 | CONFIRMED | slot 36: called by `collision_line_in_dynamics` (`case 0x48`) when the ground ray crosses a blood decal's sphere (§8) |
| 0x004ae890 | `actor_subclass_init_require_frame` | - | - | - | slot 18: an ICF fold shared with other classes (§9): `if (!frame) return false; return actor_init(frame) != 0;` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0047fe40 | `i3d_open_notify_forward_cb` | - | - | - | an `I3D_frame` enumeration callback taken by address at 3 sites; 0 direct callsites; no dog method calls it |

## 4. It derives from `C_actor`, so it has no smell sense

`C_dog` is a **`C_actor`**, not a `C_entity`. It therefore has **none** of the C_entity
AI sub-objects — no `ai_sensors`, no `ai_logs`, no `ai_blob_log`, no `ai_action_manager`
(`structs/entity.md` §7 lists them; none is present here).

A scent sense would make the dog the obvious consumer of the `ai_sensors` smell band, but across
all 39 functions **nothing in this class touches a blob, a scent contact, the smell extrema or any
part of `ai_sensors`.**

What the dog actually has instead is `dog_acquire_player_target` — a raw distance test
against the player actor, re-armed every 2000 ms. That is the entire perception system.

⇒ The smell sense's consumers are elsewhere. The producer half is documented in
`structs/ai_blob_log.md` and `structs/ai_group.md` §5; the consumer half is **not in this class**.

## 5. Navigation — the part worth reusing

The dog is the cleanest small consumer of the whole navigation stack, which makes it a
good worked example for `structs/web_path.md` / `ai_wayfind.md` / `ai_path.md`:

| step | function | what it does |
|---|---|---|
| pick a goal | `dog_choose_next_web_node` | walks the current `C_web_node`'s connection list (`+0x18`, count byte `+0x1d`, stride 8), skipping one-way and blocked links |
| plan | `dog_set_destination_and_plan` | writes the destination, publishes the blocked list, runs the shortest-route search |
| walk | `dog_follow_path` → `dog_move_toward_target` → `dog_turn_toward_target` | consumes the `ai_path`, turns then steps |
| test | `dog_test_move_collision` | probes 0.3 ahead before committing a step |
| re-plan | `dog_replan_path` | same search again after a blocked segment |
| remember | `dog_is_segment_blocked` | scans the `vector<blocked_con>` for this connection |

🔑 **The blocked-connection record is `blocked_con`** — the same type as
`C_entity+0x119c`.
The PC decomp shows only a stride-`0x10` array; the type name comes from the PS2 symbol
`reserve__Q23gif23vector<11blocked_con,2>Fi`, which is emitted **inside `C_dog.cpp`** (§10).
Each record is `{node_a, node_b, [U], expiry}`, and `dog_is_segment_blocked` compares the
node pair in **both orders**, so a blocked connection is undirected.

`dog_follow_path` writes the records, when the `ai_path` reports state 3 (`structs/ai_path.md`):
two of its appends copy the path's embedded `blocked_con` (`ai_path+0x30`), and the third
builds one per matching connection of the current node with the expiry set to the mission
clock + 3000 ms. Each append is followed by `dog_replan_path`.

The list is handed to the planner through the singleton field
`g_ai_wayfind_blocked_cons` (= `ai_wayfind + 0x5c`), set immediately before the
search and cleared immediately after — the same install/plan/release shape
`entity_script_tick` uses for `C_entity`'s own list.

## 6. The animation table

`dog_apply_pending_anim` maps `+0xe0` through a 13-case switch to a clip name. This table
identifies the class on its own:

| id | clip | id | clip | id | clip |
|---|---|---|---|---|---|
| 0 | `Dbreath` | 5 | `Dhit` | 10 | `DturnR` |
| 1 | `Dwalk` | 6 | `Ddeath` | 11 | `Datack` |
| 2 | `Drun` | 7 | `Dsitting` | 12 | `Ddrb` |
| 3 | `Dsit` | 8 | `Dbark1` | - | - |
| 4 | `Dstand` | 9 | `DturnL` | - | - |

The clips carry the events `dog_frame_notify_callback` reacts to, on the frame named `notify`.
In the shipped `.5DS` files a notes block is the flag dword `0x10`, a 16-bit count, the 16-bit
frame numbers and the 16-bit values:

| clip | note frames | values | what the callback does |
|---|---|---|---|
| `Dbark1.5DS` (id 8) | 3, 15, 24, 33 | `0x5a` ×4 | re-creates and plays sound slot 2 |
| `Dbark3.5DS` | 3, 15, 24, 33 | `0x5a` ×4 | the same; `dog_enter_state` loads this clip as a random overlay in state `0x10` |
| `Dwalk.5DS` (id 1) | 1, 5, 12, 16 | 1, 5, 2, 4 | footstep: sound slot 0, and a bloody paw print (§8) |
| `Drun.5DS` (id 2) | 4, 6, 9, 12 | 2, 1, 4, 5 | the same |

The severed head is a separate model, `hlavapsa.I3D` (Czech *hlava psa*), spawned by
`dog_spawn_severed_head`. `dog_load_tracks` loads three `.tck` keyframe tracks at start.

## 7. The sound bank

Five slots, two parallel arrays: ids at `+0x8c[5]`, live objects at `+0xa0[5]`.

| function | role |
|---|---|
| `dog_create_all_sounds` | build all five |
| `dog_create_sound_slot` | build one by index |
| `dog_get_sound` | fetch one, creating it on first use |
| `dog_stop_sound` | stop one |
| `dog_release_all_sounds` | release all five |

The constructor's default ids are `0x7f`, `0x7f`, `0x7e`, `0x81`, `0x80`. Slot 0 is the footstep,
slot 2 the bark re-triggered on the `0x5a` notes of the bark clips (§6), slot 3 the yelp played
by `dog_hit` and `dog_on_collision`, and slot 4 the attack sound `dog_ai` plays in state 8 (clip
`Datack`) and in state 7. Slot 1 is created by `dog_create_all_sounds` and stopped by
`dog_enter_state`, but nothing fetches or plays it; its default id is slot 0's.

## 8. Bloody paw prints

A small closed loop worth reading as a unit, because four functions and three fields make
the whole feature:

1. `dog_snap_to_ground` casts the dog's ground ray every 200 ms with
   `g_collision_next_query_actor` set to the dog. When that ray crosses the collision sphere a
   blood decal carries, `collision_line_in_dynamics` (`case 0x48`) calls the dog's vtable
   slot 36 (`structs/collision.md` §5c, `structs/blood.md` §7e).
2. `dog_arm_bloody_paw_prints` (slot 36): when the print counter `+0x30e` is 0 it arms ten
   prints (`+0x30e` = `+0x30d` = 10); otherwise it re-syncs `+0x30d` to `+0x30e` and raises
   the print flag `+0x89`.
3. `dog_frame_notify_callback` — installed by `dog_game_init` on the frame named
   `notify` — fires on the footstep events 1/2/4/5 and, at effects quality above 1
   (`g_act_mission+0x9c`) while `+0x89` is set, calls
4. `dog_place_bloody_paw_print`, which places one decal of texture 8 (`2dogfoot.bmp`) under
   the paw frame the event names (`v`, `p`, `n`, `j`), with alpha `(+0x30e + 2) / +0x30d`,
   and decrements `+0x30e`, clearing `+0x89` when it reaches zero.

The dog's own bleeding is a different path: `dog_hit`, `dog_enter_state` and `dog_update`
spawn blood drops (`structs/blood.md` §6).

## 9. Not part of this class, and the slots it does not override

| PC VA | what it is |
|---|---|
| `0x0047ab10`–`0x0047ab50` | **five `C_detector` functions**, COMDAT-placed after the unit's own CRT pair: four `sub ecx,0x70; jmp` adjustor thunks plus `detector_get_program`. See `structs/detector.md` §4a |
| `0x0047fa10` | `i3d_cache_anim_create` — a method of the model/animation cache singleton at `0x006f93c8`, 12 callsites across `human_*`, `npc_*`, `anim_machine_set_anim`, `program_process_command` (`structs/i3d_cache.md`) |
| `0x0047fe40` | `i3d_open_notify_forward_cb` — an `I3D_frame` enumeration callback taken by address at 3 sites; 0 direct callsites |
| `0x0047fe60` / `0x0047ffe0` / `0x00480040` | `i3d_cache_items_insert_one` / `i3d_cache_items_copy_range` / `i3d_cache_items_copy_backward_range` — container helpers for `C_I3D_cache<T>::S_itm` `{T *object, C_string name}`, the model / sound / animation cache record (`structs/i3d_cache.md`; 65 / 30 / 1 references) |
| `0x004800a0` | `i3d_cache_items_scalar_deleting_dtor` — the scalar deleting destructor of that element type; 2 callers, both outside this TU |

**Inherited, not overridden** (the vtable points outside the TU): slot 15 `NetDirect`,
slot 17 `Release__7C_actorFv`, slot 27 `ChangeState`, slot 28 `actor_get_actor_speed`,
slot 32 `GetMaterialID`, slot 34 `actor_get_bound_points_2d`, slots 35/37/38 shared
stubs.

**Slot 18 `Init` is an ICF fold, not a twin.** It points at `actor_subclass_init_require_frame`
(`0x004ae890`), outside the TU, whose body is `if (!frame) return false; return actor_init(frame)
!= 0;` — a body so trivial that MSVC folded every class's identical `Init` onto one function. That
is a role, not a twin.

## 10. PC vs PS2

* **Two PS2 methods are PS2-port-only:** `ChangeSceneInit__5C_dogFv` and
  `ChangeSceneDone__5C_dogFv`. A `ChangeScene` name is itself the city-streaming tell (the PS2
  loads the city in parts; the PC does not), and after the mapping in §3 no unclaimed PC
  start is left in the TU for either.
* **The class-band field delta is a flat `+0x11c`** on the designer band (PC `+0x70` ↔
  PS2 `+0x18c`) but it does **not** hold across the whole object — the sound arrays sit
  at PC `+0x8c`/`+0xa0` against PS2 `+0x14c`, and the blocked-connection vector at PC
  `+0x2fc` against PS2 `+0x33c`. Do not rebase band-wise. The designer fields agree: the PS2
  constructor writes 40.0 at `+0x19c` and 20.0 at `+0x1a0` (PC `+0x80` / `+0x84`), and the
  designer byte with default 1 sits at `+0x1b0` and is copied by `DuplicateFrom`.
* **The `C_web_node` record differs in size across builds** (PC `0x1e`, PS2 `0x20`) and
  every node-pointer computation in this class reflects it. Field offsets inside the
  record are identical — `structs/web_node.md`.
* **`dog_notify_callback` is a free function on both builds**, emitted between `ground`
  and `BloodStep` in the same source position, which is what pinned it.
* **PS2 allocates its `ai_blob`-style records through `ai_spageman`;** this class
  allocates nothing of the sort on either build, because it uses no scent (§4).

### 10a. PS2 methods without a PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00153100 | `ChangeSceneInit__5C_dogFv` | PS2-port-only: city streaming; no unclaimed PC start is left in the unit |
| 0x00153150 | `ChangeSceneDone__5C_dogFv` | PS2-port-only: city streaming; no unclaimed PC start is left in the unit |

## 11. Related

`structs/entity_kind.md` §1 (the kind enum) · `structs/actor.md` (the base) ·
`structs/anim_machine.md` §4 (the embedded machine) · `structs/web_path.md` / `web_node.md` /
`ai_wayfind.md` / `ai_path.md` (the navigation stack it consumes) · `structs/detector.md` (the
previous TU) · `structs/door.md` (the next TU).

## 12. Open items

| item | state |
|---|---|
| `dog_death` vs `dog_on_collision` | both can reach state `0xf`; whether the corpse-fade path runs for a road kill is not tested in game |
