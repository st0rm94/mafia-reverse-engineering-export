# `C_dog` — the guard dog (kind `0x15`)

**Created** 2026-08-13 12:57 +0200 · **Modified** 2026-09-17 13:45 +0200

PC prefix **`dog_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x15`** |
| PC vtable | `.rdata 0x0063c0d0` (39 slots) |
| PC constructor | `dog_ctor` @ `0x0047ab60` |
| PC instance size | `0x310` (entity factory `case 0x15`) |
| PC translation unit | `[0x0047ab60, 0x004800c0)` — 45 starts, all named |
| Base class | **`C_actor`** — *not* `C_entity`; see §2 |
| Embedded sub-object | `C_animations_machine_de_luxe` at **`+0xe8`** |
| PS2 class | `C_dog`, 40 methods, **38/40 mapped**, 2 PS2-port-only |

## 1. What it is

A patrolling guard dog. It walks a route of `C_web_node`s, remembers which connections
it has found blocked, re-plans through `ai_wayfind` when it gets stuck, and switches to
an attack state when the player comes close enough. It bleeds a ten-drop trail after
being hurt, can be decapitated, and a vehicle at speed kills it in one hit.

Its whole behaviour hangs off one field — the **state at `+0xb4`**, written only by
`dog_enter_state`, which also picks the animation that goes with the state. Every other
method either reads that state or feeds it.

## 2. It derives from `C_actor`, so it has no smell sense

`C_dog` is a **`C_actor`**, not a `C_entity`. It therefore has **none** of the C_entity
AI sub-objects — no `ai_sensors`, no `ai_logs`, no `ai_blob_log`, no `ai_action_manager`
(`structs/entity.md` §6c lists them; none is present here).

A scent sense would make the dog the obvious consumer of the `ai_sensors` smell band, but across
all 39 functions **nothing in this class touches a blob, a scent contact, the smell extrema or any
part of `ai_sensors`.**

What the dog actually has instead is `dog_acquire_player_target` — a raw distance test
against the player actor, re-armed every 2000 ms. That is the entire perception system.

⇒ The smell sense's consumers are elsewhere. The producer half is documented in
`structs/ai_blob_log.md` and `structs/ai_group.md` §3; the consumer half is **not in this class**.

## 3. Navigation — the part worth reusing

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

The list is handed to the planner through the singleton field
`g_ai_wayfind_checkpoint_container` (= `ai_wayfind + 0x5c`), set immediately before the
search and cleared immediately after — the same install/plan/release shape
`entity_script_tick` uses for `C_entity`'s own list.

## 4. The animation table

`dog_apply_pending_anim` maps `+0xe0` through a 13-case switch to a clip name. This table
identifies the class on its own:

| id | clip | id | clip | id | clip |
|---|---|---|---|---|---|
| 0 | `Dbreath` | 5 | `Dhit` | 10 | `DturnR` |
| 1 | `Dwalk` | 6 | `Ddeath` | 11 | `Datack` |
| 2 | `Drun` | 7 | `Dsitting` | 12 | `Ddrb` |
| 3 | `Dsit` | 8 | `Dbark1` | | |
| 4 | `Dstand` | 9 | `DturnL` | | |

The severed head is a separate model, `hlavapsa.I3D` (Czech *hlava psa*), spawned by
`dog_spawn_severed_head`. `dog_load_tracks` loads three `.tck` keyframe tracks at start.

## 5. The sound bank

Five slots, two parallel arrays: ids at `+0x8c[5]`, live objects at `+0xa0[5]`.

| function | role |
|---|---|
| `dog_create_all_sounds` | build all five |
| `dog_create_sound_slot` | build one by index |
| `dog_get_sound` | fetch one, creating it on first use |
| `dog_stop_sound` | stop one |
| `dog_release_all_sounds` | release all five |

Slot 0 is the footstep, slot 2 the bark/growl re-triggered on event `0x5a`, slot 3 the
yelp played by `dog_on_collision`. Slots 1 and 4 have no traced consumer.

## 6. The blood trail

A small closed loop worth reading as a unit, because three functions and two fields make
the whole feature:

1. `dog_arm_bleeding` (vtable slot 36, the collision-tester callback) sets the drop
   counter `+0x30e` to **10** and raises the bleeding flag `+0x89`;
2. `dog_frame_notify_callback` — installed by `dog_game_init` on the frame named
   `notify` — fires on animation events 1/2/4/5 and, while `+0x89` is set, calls
3. `dog_spawn_blood_drop`, which places one decal under the named body frame and
   decrements `+0x30e`, clearing `+0x89` when it reaches zero.

## 7. Field layout

```c
typedef struct C_dog {                    /* 0x310 bytes */
    /* +0x000 */ C_actor  base;           // [H] kind 0x15 at +0x10, frame at +0x68
    /* +0x070 */ float    tunable[4];     // [H] 1.0 / 5.5 / 4.0 / — ; saved + duplicated
    /* +0x080 */ int      cfg_a, cfg_b;   // [M] saved + duplicated, meaning untraced
    /* +0x088 */ bool     cfg_flag;       // [M] saved + duplicated
    /* +0x089 */ bool     bleeding;       // [H] set by dog_arm_bleeding, cleared at 0 drops
    /* +0x08c */ int      sound_id[5];    // [H] designer-set; saved + duplicated
    /* +0x0a0 */ I3D_sound *sound[5];     // [H] live objects, lazily created
    /* +0x0b4 */ int      state;          // [H] 0xe = hurt, 0xf = dead; dog_enter_state
    /* +0x0b8 */ float    yaw_cur;        // [H] dog_turn_toward_target ramp
    /* +0x0c0 */ float    yaw_rate;       // [H]
    /* +0x0d0 */ float    health;         // [H] dog_hit subtracts damage; collision −100
    /* +0x0dc */ int      anim_cur;       // [H] last clip pushed to the machine
    /* +0x0e0 */ int      anim_pending;   // [H] dog_enter_state writes it
    /* +0x0e4 */ bool     anim_flag_a;    // [H] passed to anim_machine_set_anim
    /* +0x0e5 */ bool     anim_flag_b;    // [H]
    /* +0x0e8 */ C_animations_machine_de_luxe anim;  // [H] EMBEDDED
    /* +0x234 */ tDynamicCollObject *coll_a;   // [H] owned; erased by Death/GameDone
    /* +0x238 */ tDynamicCollObject *coll_b;   // [H] owned
    /* +0x23c */ S_vector  destination;   // [H] the route goal
    /* +0x2d4 */ C_web_node *node_cur;    // [H] record stride 0x1e
    /* +0x2d8 */ C_web_node *node_prev;   // [H] avoids doubling back
    /* +0x2dc */ uint     ground_timer;   // [H] period +0x2e0 = 200 ms
    /* +0x2e4 */ uint     target_timer;   // [H] period +0x2e8 = 2000 ms
    /* +0x2f4 */ C_actor *enemy;          // [H] the acquired player; cleared on return
    /* +0x2f8 */ ai_path *path;           // [H] owned; destroyed on every re-plan
    /* +0x2fc */ vector<blocked_con> blocked;  // [H] alloc/begin/end at +0x2fc/+0x300/+0x304
    /* +0x30d */ char     blood_working;  // [H] working copy of the drop counter
    /* +0x30e */ char     blood_drops;    // [H] armed to 10
    /* +0x30f */ bool     was_hit;        // [H] set by dog_hit
} C_dog;
```

Confidence: **[H]** measured on both builds · **[M]** measured on PC, meaning open ·
**[U]** untraced.

## 8. Method coverage — 38/40 PS2 methods

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x0047ab60` | `dog_ctor` | `__ct__5C_dogFv` | - | CONFIRMED |
| `0x0047ad60` | `dog_scalar_deleting_dtor` | - | 16 | STRUCTURAL ¹ |
| `0x0047ad80` | `dog_dtor` | `__dt__5C_dogFv` | - | CONFIRMED |
| `0x0047ae00` | `dog_save_data` | `SaveData__5C_dogFP7C_chunk` | 19 | CONFIRMED |
| `0x0047af40` | `dog_load_data` | `LoadData__5C_dogFP7C_chunk` | 20 | CONFIRMED |
| `0x0047b160` | `dog_duplicate_from` | `DuplicateFrom__5C_dogFP7C_actor` | 21 | CONFIRMED |
| `0x0047b1c0` | `dog_game_init` | `GameInit__5C_dogFv` | 25 | CONFIRMED |
| `0x0047b680` | `dog_load_tracks` | `LoadTCK__5C_dogFv` | - | CONFIRMED |
| `0x0047b6a0` | `dog_game_done` | `GameDone__5C_dogFv` | 26 | CONFIRMED |
| `0x0047b800` | `dog_spawn_severed_head` | `LoseHead__5C_dogFRC8S_vector` | - | CONFIRMED |
| `0x0047bdd0` | `dog_hit` | `Hit__5C_dogF10E_hit_type…` | 31 | CONFIRMED |
| `0x0047c190` | `dog_death` | `Death__5C_dogFv` | 29 | CONFIRMED |
| `0x0047c210` | `dog_ai` | `AI__5C_dogFUi` | 13 | CONFIRMED |
| `0x0047c990` | `dog_follow_path` | `SolvePath__5C_dogFUi` | - | CONFIRMED |
| `0x0047d0e0` | `dog_move_toward_target` | `AI_walk_run__5C_dogFUiRC8S_vector` | - | CONFIRMED |
| `0x0047d450` | `dog_turn_toward_target` | `AI_turn__5C_dogFUiRC8S_vector` | - | CONFIRMED |
| `0x0047d7f0` | `dog_enter_state` | `ChooseAction__5C_dogFQ25C_dog7E_stateUc` | - | CONFIRMED |
| `0x0047dba0` | `dog_update` | `Update__5C_dogFUi` | 14 | CONFIRMED |
| `0x0047e0e0` | `dog_apply_pending_anim` | `SetAnim__5C_dogFf` | - | CONFIRMED |
| `0x0047e1e0` | `dog_test_move_collision` | `TestCollision__5C_dogFR8S_vector` | - | CONFIRMED |
| `0x0047e480` | `dog_choose_next_web_node` | `ChooseNextCheckPoint__5C_dogFb` | - | CONFIRMED |
| `0x0047e740` | `dog_is_segment_blocked` | `IsBlocked__5C_dogFUi` | - | CONFIRMED |
| `0x0047e7f0` | `dog_snap_to_ground` | `ground__5C_dogFUi` | - | CONFIRMED |
| `0x0047e900` | `dog_frame_notify_callback` | `dog_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` | — | CONFIRMED ² |
| `0x0047eaf0` | `dog_spawn_blood_drop` | `BloodStep__5C_dogFUi` | - | CONFIRMED |
| `0x0047eda0` | `dog_create_all_sounds` | `InitSounds__5C_dogFv` | - | CONFIRMED |
| `0x0047ee10` | `dog_create_sound_slot` | `InitSound__5C_dogFUi` | - | CONFIRMED |
| `0x0047eeb0` | `dog_release_all_sounds` | `ReleaseSounds__5C_dogFv` | - | CONFIRMED |
| `0x0047eee0` | `dog_get_sound` | `GetSound__5C_dogFUi` | - | CONFIRMED |
| `0x0047ef20` | `dog_stop_sound` | `StopSound__5C_dogFUi` | - | CONFIRMED |
| `0x0047ef40` | `dog_return_to_patrol` | `ReturnFromAttackAI__5C_dogFv` | - | CONFIRMED |
| `0x0047efb0` | `dog_acquire_player_target` | `TestEnemy__5C_dogFv` | - | CONFIRMED |
| `0x0047f290` | `dog_replan_path` | `UnblockPath__5C_dogFv` | - | CONFIRMED |
| `0x0047f3a0` | `dog_set_destination_and_plan` | `UpdatePath__5C_dogFRC8S_vector` | - | CONFIRMED |
| `0x0047f490` | `dog_save_game_get_size` | `SaveGameGetSize__5C_dogFv` | 22 | CONFIRMED |
| `0x0047f4a0` | `dog_save_game_save` | `SaveGameSave__5C_dogFPPv` | 23 | CONFIRMED |
| `0x0047f680` | `dog_save_game_load` | `SaveGameLoad__5C_dogFPPv` | 24 | CONFIRMED |
| `0x0047f950` | `dog_on_collision` | `Collision__5C_dogF…` | 30 | CONFIRMED |
| `0x0047f9e0` | `dog_arm_bleeding` | `cbCollTester__5C_dogFP18tDynamicCollObject` | 36 | CONFIRMED |

¹ MSVC-only. ² A free function on both builds (§10).

**Inherited, not overridden** (the vtable points outside the TU): slot 15 `NetDirect`,
slot 17 `Release__7C_actorFv`, slot 27 `ChangeState`, slot 28 `actor_get_actor_speed`,
slot 32 `GetMaterialID`, slot 34 `actor_get_bound_points_2d`, slots 35/37/38 shared
stubs.

**Slot 18 `Init` is an ICF fold, not a twin.** It points at `actor_subclass_init_require_frame`
(`0x004ae890`), outside the TU, whose body is `if (!frame) return false; return actor_init(frame)
!= 0;` — a body so trivial that MSVC folded every class's identical `Init` onto one function. That
is a role, not a twin.

## 9. Not part of this class (they sit in its TU)

| PC VA | what it is |
|---|---|
| `0x0047ab10`–`0x0047ab50` | **five `C_detector` functions**, COMDAT-placed after the CRT pair that closes the detector TU — four `sub ecx,0x70; jmp` adjustor thunks plus `detector_get_program`. See `structs/detector.md` §2 |
| `0x0047fa10` | `i3d_cache_anim_create` — a method of the model/animation cache singleton at `0x006f93c8`, 12 callsites across `human_*`, `npc_*`, `anim_machine_set_anim`, `program_process_command` (`structs/i3d_cache.md`) |
| `0x0047fe40` | `i3d_open_notify_forward_cb` — an `I3D_frame` enumeration callback taken by address at 3 sites; 0 direct callsites |
| `0x0047fe60` / `0x0047ffe0` / `0x00480040` | `std_vector8_insert_one` / `array_copy_refcounted8` / `array_copy_backward_refcounted8` — shared container helpers for a refcounted 8-byte element type (65 / 30 / 1 references) |
| `0x004800a0` | `refcounted8_scalar_deleting_dtor` — the scalar deleting destructor of that element type; 2 callers, both outside this TU |

## 10. PC vs PS2

* **Two PS2 methods are PS2-port-only:** `ChangeSceneInit__5C_dogFv` and
  `ChangeSceneDone__5C_dogFv`. A `ChangeScene` name is itself the city-streaming tell (the PS2
  loads the city in parts; the PC does not), and after the mapping in §8 no unclaimed PC
  start is left in the TU for either.
* **The class-band field delta is a flat `+0x11c`** on the designer band (PC `+0x70` ↔
  PS2 `+0x18c`) but it does **not** hold across the whole object — the sound arrays sit
  at PC `+0x8c`/`+0xa0` against PS2 `+0x14c`, and the blocked-connection vector at PC
  `+0x2fc` against PS2 `+0x33c`. Do not rebase band-wise.
* **The `C_web_node` record differs in size across builds** (PC `0x1e`, PS2 `0x20`) and
  every node-pointer computation in this class reflects it. Field offsets inside the
  record are identical — `structs/web_node.md`.
* **`dog_notify_callback` is a free function on both builds**, emitted between `ground`
  and `BloodStep` in the same source position, which is what pinned it.
* **PS2 allocates its `ai_blob`-style records through `ai_spageman`;** this class
  allocates nothing of the sort on either build, because it uses no scent (§2).

## 11. Related

`structs/entity.md` §1 (the kind enum) · `structs/actor.md` (the base) ·
`structs/anim_machine.md` §1 (the embedded machine) · `structs/web_path.md` / `web_node.md` /
`ai_wayfind.md` / `ai_path.md` (the navigation stack it consumes) · `structs/detector.md` (the
previous TU) · `structs/door.md` (the next TU).

## 12. Open items

| item | state |
|---|---|
| the two unused sound slots | slots 1 and 4 are created and released but never played by any traced path |
| `+0x080` / `+0x084` / `+0x088` | saved, loaded and duplicated, so they are designer-set — but nothing read reveals what they configure |
| `blocked_con` field 3 | `dog_is_segment_blocked` compares `rec[3]` against the mission clock, so it is an expiry; **who writes a record** is not traced in this class |
| the element class of the refcounted 8-byte helpers | unidentified |
| the `0x5a` animation event | `dog_frame_notify_callback` re-creates and plays sound slot 2 on it; which clip emits the event is not identified |
| `dog_death` vs `dog_on_collision` | both can reach state `0xf`; whether the corpse-fade path runs for a road kill is not tested in game |

