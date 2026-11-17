# `C_pumper` — the gas-station attendant (kind `0x19`)

**Created** 2026-07-13 11:46 +0200 · **Modified** 2026-11-09 18:21 +0100

The NPC who stands at the fuel pump, walks out to a stopped car, refuels it, charges you for it, and
smokes and fidgets in between.

| | |
|---|---|
| PC prefix | `pumper_` |
| PC kind (`+0x10`) | `0x19` (`structs/entity_kind.md` §1), re-tested by `pumper_script_can_pump` and `pumper_notify_callback` |
| PC vtable | `g_pumper_vt` @ `0x0063ca58` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x2ac` (`operator_new(0x2ac)` in `mission_create_actor`) |
| PC constructor | `pumper_ctor` @ `0x004ce000` |
| PC translation unit | `[0x004cdfd0, 0x004d33a0)`, guard to guard: 39 starts, all named |
| PS2 class | `C_pumper` (tag `8C_pumper`), 36 methods + the free `pumper_notify_callback` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base (`structs/actor.md` §1). The class
uses the kind `+0x10`, the act state `+0x1c`, the position `+0x24`, the forward row `+0x30` and the
frame `+0x68`.

```c
typedef struct C_pumper {                     // 0x2ac bytes; C_actor base +0x00..+0x70
    /* +0x70 */ float walk_speed;                  // [H] units per second (ctor 2.0)
    /* +0x74 */ float second_speed;                // [U] ctor 0.5; no reader (.text scan)
    /* +0x78 */ uint32_t pump_node_id;             // [H] web node where he waits (designer; ctor 0x29a)
    /* +0x7c */ uint32_t post_node_id;             // [H] web node at the car (designer; ctor 0x29b)
    /* +0x80 */ uint32_t script_id;                // [H] the number in every `_pumpa*_%i` frame name
    /* +0x84 */ uint32_t speech_line_id;           // [H] his own line (designer; ctor 0x497cc = 301004)
    /* +0x88 */ uint32_t sound_ids[6];             // [H] sounds.dat ids, one per sound slot (§6)
    /* +0xa0 */ int32_t cigarette_smoke;           // [H] particle on the right-hand cigarette, -1 = none
    /* +0xa4 */ int32_t mouth_smoke;               // [H] particle on the mouth cigarette or the exhale, -1 = none
    /* +0xa8 */ int32_t second_smoke;              // [H] second particle started with the cigarette smoke, -1 = none
    /* +0xac */ uint32_t smoke_timer;              // [H] ms, stepped by pumper_update while a smoke runs
    /* +0xb0 */ uint32_t smoke_period;             // [H] 1000 after message 0x3e, 6500 after 0x41; ctor 200
    /* +0xb4 */ I3D_sound *sounds[6];              // [H] one per sound slot (§6)
    /* +0xcc */ uint32_t unread_cc;                // [U] ctor 0, savegame round-trip; no reader (.text scan)
    /* +0xd0 */ int32_t action;                    // [H] the state of §2a
    /* +0xd4 */ uint32_t anim_applied;             // [H] the clip pumper_set_anim last started
    /* +0xd8 */ uint32_t anim_pending;             // [H] the clip to start; >= 0xffff = a special clip
    /* +0xdc */ uint8_t anim_play_once;            // [H] anim_machine_set_anim's play-once flag
    /* +0xdd */ uint8_t anim_use_track;            // [H] anim_machine_set_anim's track flag: loads <clip>.tck, forces play-once
    /* +0xe0 */ C_animations_machine_de_luxe anim; // [H] embedded (structs/anim_machine.md §4)
    /* +0x208 */ I3D_animation_set *hose_anim_set;  // [H] from the cache, set on the hose by pumper_game_init
    /* +0x20c */ C_web_node *next_node;             // [H] the node he walks to
    /* +0x210 */ C_web_node *prev_node;             // [H] the node he left
    /* +0x214 */ C_web_node *third_node;            // [U] ctor 0, saved as a node index; no reader (.text scan)
    /* +0x218 */ I3D_frame *station_frame;          // [H] `_pumpa_%i`, the car's target
    /* +0x21c */ I3D_frame *camera_frame;           // [H] `_pumpa_camera_%i`, the refuel camera position
    /* +0x220 */ tDynamicCollObject *coll_box;      // [H] type 0x42, allocated 0x48 bytes
    /* +0x224 */ tDynamicCollObject *coll_sphere;   // [H] type 0x40, allocated 0x48 bytes
    /* +0x228 */ uint32_t resnap_timer;             // [H] ms to the next ground snap
    /* +0x22c */ uint32_t resnap_period;            // [H] `rand_range() + 100` ms; ctor 200
    /* +0x230 */ uint32_t speech_channel;           // [H] the channel of the running line
    /* +0x234 */ I3D_frame *hose;                   // [H] `_pumpa_hadice_%i`, a model frame (type 9)
    /* +0x238 */ I3D_frame *cigarette_hand;         // [H] `_pumpa_cigaro_%i`, right hand
    /* +0x23c */ I3D_frame *pack_hand;              // [H] `_pumpa_cigaro_box_%i`, right hand
    /* +0x240 */ I3D_frame *cigarette_mouth;        // [H] `_pumpa_xcigaro_%i`, on `neck`
    /* +0x244 */ I3D_frame *pack_left_hand;         // [H] `_pumpa_xcigaro_box_%i`, left hand
    /* +0x248 */ I3D_frame *pump_frame;             // [H] `_pumpa_pumpa_%i`; off = he panics
    /* +0x24c */ C_car *served_car;                 // [H] the car being refuelled
    /* +0x250 */ uint32_t refuel_duration;          // [H] ms between the 0x2a and 0x2b clip events, at least 1000
    /* +0x254 */ float fuel_per_ms;                 // [H] (capacity - fuel) / refuel_duration
    /* +0x258 */ uint8_t fuel_flowing;              // [H] set between the clip events 0x2a and 0x2b
    /* +0x25c */ uint32_t dwell_timer;              // [H] ms in actions 1, 0xc, 0xd
    /* +0x260 */ uint32_t dwell_period;             // [H] `rand_range() + 1000` or `+ 2000`, or 7500 in action 0xd
    /* +0x264 */ uint8_t repeat_count;              // [H] fidgets or drags left
    /* +0x268 */ S_matrix world_inverse;            // [H] the frame's inverse world matrix, for both coll objects
    /* +0x2a8 */ I3D_frame *hose_parent;            // [H] the hose's parent at scene start; pumper_game_done links it back
} C_pumper;
```

**`sound_ids[6]` and `sounds[6]`** are parallel: slot 0 is the footstep, slots 1 to 5 the hose and
pump sounds of §6. Six is the bound of `pumper_init_sound` (`param < 6`) and of the loops in
`pumper_init_sounds` and `pumper_release_sounds`.

**The three particle handles** are three separate fields, not an array: each is created at its own
clip event (§4) and destroyed by name in `pumper_update`, `pumper_panic`, `pumper_game_done` and
`pumper_start_refuel`.

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions (ECX on entry,
followed through moves and stack slots), plus what that census cannot see: the receiver of
`pumper_notify_callback`, which it reads from the frame, and the indexed sound slots. Every
hand-added cell was read in its body.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | walk_speed | H | `pumper_ctor` | `pumper_ai_walk` | - |
| +0x74 | second_speed | U | `pumper_ctor` | - | ctor 0.5; no reader (.text scan); Open item 3 |
| +0x78 | pump_node_id | H | `pumper_ctor`, `pumper_duplicate_from`, `pumper_load_data` | `pumper_ai`, `pumper_choose_next_action`, `pumper_game_init`, `pumper_save_data` | - |
| +0x7c | post_node_id | H | `pumper_ctor`, `pumper_duplicate_from`, `pumper_load_data` | `pumper_choose_next_action`, `pumper_save_data` | - |
| +0x80 | script_id | H | `pumper_ctor`, `pumper_load_data` | `pumper_game_done`, `pumper_game_init`, `pumper_save_data`, `pumper_script_can_pump` | - |
| +0x84 | speech_line_id | H | `pumper_ctor`, `pumper_duplicate_from`, `pumper_load_data` | `pumper_choose_next_action`, `pumper_save_data` | §5 |
| +0x88 | sound_ids | H | `pumper_ctor`, `pumper_duplicate_from`, `pumper_load_data` | `pumper_duplicate_from`, `pumper_init_sound`, `pumper_save_data` | - |
| +0xa0 | cigarette_smoke | H | `pumper_ctor`, `pumper_game_init`, `pumper_notify_callback`, `pumper_save_game_load` | `pumper_game_done`, `pumper_notify_callback`, `pumper_panic`, `pumper_start_refuel` | §4 |
| +0xa4 | mouth_smoke | H | `pumper_ctor`, `pumper_game_init`, `pumper_notify_callback`, `pumper_update` | `pumper_game_done`, `pumper_panic`, `pumper_start_refuel`, `pumper_update` | §4 |
| +0xa8 | second_smoke | H | `pumper_ctor`, `pumper_game_init`, `pumper_notify_callback`, `pumper_update` | `pumper_game_done`, `pumper_panic`, `pumper_start_refuel`, `pumper_update` | §4 |
| +0xac | smoke_timer | H | `pumper_ctor`, `pumper_notify_callback`, `pumper_update` | `pumper_update` | - |
| +0xb0 | smoke_period | H | `pumper_ctor`, `pumper_notify_callback` | `pumper_update` | ctor 200 |
| +0xb4 | sounds | H | `pumper_init_sound`, `pumper_release_sounds` | `pumper_ctor`, `pumper_init_sound`, `pumper_play_sound`, `pumper_release_sounds`, `pumper_start_pump_fuel` | - |
| +0xcc | unread_cc | U | `pumper_ctor`, `pumper_save_game_load` | `pumper_save_game_save` | ctor 0, savegame round-trip (record `+4`); no reader (.text scan) |
| +0xd0 | action | H | `pumper_ai`, `pumper_choose_next_action`, `pumper_game_init`, `pumper_hit`, `pumper_on_collision`, `pumper_panic`, `pumper_save_game_load`, `pumper_start_refuel` | `pumper_ai`, `pumper_can_pump`, `pumper_choose_next_action`, `pumper_hit`, `pumper_is_in_panic`, `pumper_is_pumping`, `pumper_on_collision`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_script_can_pump`, `pumper_update` | - |
| +0xd4 | anim_applied | H | `pumper_ctor`, `pumper_game_init`, `pumper_set_anim` | `pumper_set_anim` | - |
| +0xd8 | anim_pending | H | `pumper_ai`, `pumper_ai_turn`, `pumper_ai_walk`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load`, `pumper_start_refuel` | `pumper_save_game_save`, `pumper_set_anim` | - |
| +0xdc | anim_play_once | H | `pumper_ai_turn`, `pumper_ai_walk`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load`, `pumper_start_refuel` | `pumper_save_game_save`, `pumper_set_anim` | `pumper_set_anim` calls `anim_machine_set_anim(clip, 0, 0, +0xdc, 4.0, start, +0xdd, 0, 0, 0, -1)`; in the callee's parameter order this is the play-once flag (stored into `stage0_play_once` / `stage1_play_once`). 1 for the hose clip and the fidget and drag clips whose end the AI waits for, 0 for `KurakLoop` and the walk |
| +0xdd | anim_use_track | H | `pumper_ai_turn`, `pumper_ai_walk`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load`, `pumper_start_refuel` | `pumper_save_game_save`, `pumper_set_anim` | the track parameter of `anim_machine_set_anim`: when set the callee loads `<clip>.tck` with `anim_machine_get_track`, stores the track index at machine `+0x2c` and forces play-once. 1 only for `Nuda01`, `Nuda02` and `turn1` |
| +0xe0 | anim | H | `anim_machine_ctor` | `anim_machine_set_anim`, `anim_machine_tick`, `pumper_ai`, `pumper_ctor`, `pumper_dtor`, `pumper_game_init`, `pumper_save_game_save`, `pumper_set_anim` | - |
| +0x208 | hose_anim_set | H | `pumper_ctor`, `pumper_game_init` | `pumper_game_init` | - |
| +0x20c | next_node | H | `pumper_advance_to_next_node`, `pumper_ai`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load` | `pumper_advance_to_next_node`, `pumper_ai`, `pumper_ai_walk`, `pumper_choose_next_action`, `pumper_game_init`, `pumper_save_game_save` | - |
| +0x210 | prev_node | H | `pumper_advance_to_next_node`, `pumper_ai`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load` | `pumper_advance_to_next_node`, `pumper_ai`, `pumper_choose_next_action`, `pumper_game_init`, `pumper_save_game_save` | - |
| +0x214 | third_node | U | `pumper_ctor`, `pumper_save_game_load` | `pumper_save_game_save` | ctor 0, saved as a node index; no reader (.text scan); Open item 3 |
| +0x218 | station_frame | H | `pumper_ctor`, `pumper_game_init` | `pumper_can_pump`, `pumper_start_refuel` | - |
| +0x21c | camera_frame | H | `pumper_game_init` | `pumper_save_game_load`, `pumper_start_refuel` | - |
| +0x220 | coll_box | H | `pumper_ai`, `pumper_ctor`, `pumper_game_init`, `pumper_release_colls` | `pumper_ai`, `pumper_game_init`, `pumper_release_colls`, `pumper_snap_to_ground`, `pumper_update` | - |
| +0x224 | coll_sphere | H | `pumper_ai`, `pumper_ctor`, `pumper_game_init`, `pumper_release_colls` | `pumper_ai`, `pumper_game_init`, `pumper_release_colls`, `pumper_update` | - |
| +0x228 | resnap_timer | H | `pumper_ai`, `pumper_ctor`, `pumper_game_init`, `pumper_update` | `pumper_update` | - |
| +0x22c | resnap_period | H | `pumper_ai`, `pumper_ctor`, `pumper_game_init`, `pumper_update` | `pumper_update` | ctor 200 |
| +0x230 | speech_channel | H | `pumper_choose_next_action`, `pumper_ctor`, `pumper_start_refuel` | `pumper_ai`, `pumper_choose_next_action`, `pumper_start_refuel` | - |
| +0x234 | hose | H | `pumper_ctor`, `pumper_game_init` | `pumper_ai`, `pumper_choose_next_action`, `pumper_game_done`, `pumper_game_init`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_update` | - |
| +0x238 | cigarette_hand | H | `pumper_ctor`, `pumper_game_done`, `pumper_game_init` | `pumper_game_done`, `pumper_game_init`, `pumper_notify_callback`, `pumper_panic`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_start_refuel` | - |
| +0x23c | pack_hand | H | `pumper_ctor`, `pumper_game_done`, `pumper_game_init` | `pumper_game_done`, `pumper_game_init`, `pumper_notify_callback`, `pumper_panic`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_start_refuel` | - |
| +0x240 | cigarette_mouth | H | `pumper_ctor`, `pumper_game_done`, `pumper_game_init` | `pumper_game_done`, `pumper_game_init`, `pumper_notify_callback`, `pumper_panic`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_start_refuel` | - |
| +0x244 | pack_left_hand | H | `pumper_ctor`, `pumper_game_done`, `pumper_game_init` | `pumper_game_done`, `pumper_game_init`, `pumper_notify_callback`, `pumper_panic`, `pumper_save_game_load`, `pumper_save_game_save`, `pumper_start_refuel` | - |
| +0x248 | pump_frame | H | `pumper_ctor`, `pumper_game_init` | `pumper_ai`, `pumper_save_game_load`, `pumper_update` | - |
| +0x24c | served_car | H | `pumper_choose_next_action`, `pumper_ctor`, `pumper_save_game_load`, `pumper_start_refuel` | `pumper_ai`, `pumper_choose_next_action`, `pumper_end_pump_fuel`, `pumper_on_collision`, `pumper_save_game_load`, `pumper_save_game_save` | - |
| +0x250 | refuel_duration | H | `pumper_ctor`, `pumper_game_init` | `pumper_choose_next_action`, `pumper_save_game_save` | §5; saved, and not restored by `pumper_save_game_load` (§6) |
| +0x254 | fuel_per_ms | H | `pumper_choose_next_action`, `pumper_ctor`, `pumper_save_game_load` | `pumper_ai`, `pumper_save_game_save` | §5 |
| +0x258 | fuel_flowing | H | `pumper_ctor`, `pumper_end_pump_fuel`, `pumper_game_init`, `pumper_save_game_load`, `pumper_start_pump_fuel` | `pumper_ai`, `pumper_save_game_save` | - |
| +0x25c | dwell_timer | H | `pumper_ai`, `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load` | `pumper_ai`, `pumper_save_game_save` | - |
| +0x260 | dwell_period | H | `pumper_choose_next_action`, `pumper_ctor`, `pumper_game_init`, `pumper_save_game_load` | `pumper_ai`, `pumper_save_game_load`, `pumper_save_game_save` | `pumper_game_init` seeds `rand_range() + 2000`; the ctor stores 200 and then 0 |
| +0x264 | repeat_count | H | `pumper_choose_next_action`, `pumper_ctor`, `pumper_save_game_load` | `pumper_choose_next_action`, `pumper_save_game_save` | - |
| +0x268 | world_inverse | H | `pumper_ai`, `pumper_game_init`, `pumper_update` | `pumper_ai`, `pumper_game_init`, `pumper_update` | - |
| +0x2a8 | hose_parent | H | `pumper_ctor`, `pumper_game_init` | `pumper_game_done` | - |

### 2a. `action` — the state machine

Read from `pumper_ai` (what each action does per frame) and `pumper_choose_next_action` (where it goes
next).

| Value | Per frame | Next | Evidence |
|---|---|---|---|
| 0 | `pumper_ai_walk` | post node → 10; pump node → 1, handing the car back; else the next node | both bodies |
| 1 | the dwell timer, idle at the pump | 0xf (light up, `repeat_count` = 3 + random) or 0xc (`repeat_count` = 1 + random) | `pumper_game_init` enters it |
| 2 | wait for the clip | 4 | entered with clip `0x10008` (`turn1`) after action 5 |
| 3 | wait for the clip | - | `pumper_ai` has the case; no store of 3 in the class |
| 4 | at once | 6, speaking `speech_line_id` on his own model | - |
| 5 | steps the hose animation and, while `fuel_flowing`, adds `dt * fuel_per_ms` to the fuel; on clip end the next action | 2 | entered with clip id `0x153` (339) |
| 6 | wait for the line | 9, speaking line 14940003 on the player's model | - |
| 7 | wait for the line | 8, speaking line 9990501 on the player's model | set by `pumper_start_refuel` |
| 8 | wait for the line | 0 | - |
| 9 | wait for the line | 0, swapping `next_node` and `prev_node` | - |
| 10 | at once | 5, computing `fuel_per_ms` and starting the hose animation | at the post node |
| 0xb | wait for the clip | 0xc | a fidget clip |
| 0xc | the dwell timer, clip `NudaStativ` | `repeat_count` = 0 → 1; else 0xb with one of `Nuda01` / `Nuda02` / `Nuda03` | - |
| 0xd | the dwell timer (7500 ms), clip `KurakLoop` | `repeat_count` = 0 → 0x10 (`KurakZahozeni`); else 0xe (`KurakPotahnuti`) | - |
| 0xe | wait for the clip | 0xd | a drag |
| 0xf | wait for the clip | 0xd | `KurakZapaleni`, lighting up |
| 0x10 | wait for the clip | 1 | throwing it away |
| 0x11 | panic: the respawn watch of §5 | 1 on the respawn | `pumper_hit`, `pumper_on_collision`, `pumper_panic` |

## 3. Method family

37 of 37 PS2 symbols mapped: 34 with an own PC body (CONFIRMED), `Init` STRUCTURAL on the shared
default, `Death` a base stub, and `NetDirect` has no PS2 method here. 3 PC-only starts: the unit's
static-init pair and the scalar deleting destructor. The unit also holds
`i3d_cache_items_erase_at` (`0x004d3330`), a resource-cache helper (`structs/i3d_cache.md` §3c) that
MSVC emitted into this object: all its callsites erase from a cache item vector, so it is listed there.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004cdfd0 | `crt_static_init_guard_004cdfd0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004cdff0 | `crt_atexit_register_004cdff0` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x004ce000 | `pumper_ctor` | instance | `__ct__8C_pumperFv` | 0x001d4ca0 | CONFIRMED | `actor_ctor`, `anim_machine_ctor`, vtable, kind `0x19`, the defaults of §1; one caller, `mission_create_actor` case `0x19` |
| 0x004ce1a0 | `pumper_scalar_deleting_dtor` | instance | - | - | - | slot 16; MSVC-only |
| 0x004ce1c0 | `pumper_dtor` | instance | `__dt__8C_pumperFv` | 0x001d4ef0 | CONFIRMED | restores the vtable, `actor_dtor` |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004d32d0 | `pumper_is_in_panic` | `IsInPanic__8C_pumperFv` | 0x001db600 | CONFIRMED | `action == 0x11`; sole caller `pumper_script_can_pump` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004ae890 | `actor_subclass_init_require_frame` | `Init__8C_pumperFP9I3D_frame` | 0x001d4f60 | STRUCTURAL | slot 18; the shared `Init` of 6 vtables |
| 0x004ce220 | `pumper_save_data` | `SaveData__8C_pumperFP7C_chunk` | 0x001d4f90 | CONFIRMED | slot 19; a version byte, then pump node, post node, line, script id, six sound ids |
| 0x004ce300 | `pumper_load_data` | `LoadData__8C_pumperFP7C_chunk` | 0x001d50b0 | CONFIRMED | slot 20; the same fields back |
| 0x004ce500 | `pumper_duplicate_from` | `DuplicateFrom__8C_pumperFP7C_actor` | 0x001d53a0 | CONFIRMED | slot 21; copies pump node, post node, line and the sound ids (not the script id) |
| 0x004ce540 | `pumper_game_init` | `GameInit__8C_pumperFv` | 0x001d53d0 | CONFIRMED | slot 25; builds the props of §7, the hose animation, the `notify` callback, both coll objects, enters action 1 |
| 0x004d0180 | `pumper_game_done` | `GameDone__8C_pumperFv` | 0x001d7db0 | CONFIRMED | slot 26; releases what `GameInit` built, and resolves and shows the four cigarette frames again |
| 0x004d03e0 | `pumper_on_collision` | `Collision__8C_pumperFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001d8060 | CONFIRMED | slot 30; §5 |
| 0x004d0460 | `pumper_hit` | `Hit__8C_pumperF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001d8140 | CONFIRMED | slot 31; §5 |
| 0x004d04d0 | `pumper_ai` | `AI__8C_pumperFUi` | 0x001d8240 | CONFIRMED | slot 13; the per-action dispatch (§2a), the respawn of §5, the clip tick |
| 0x004d0c10 | `pumper_ai_walk` | `AI_walk__8C_pumperFUi` | 0x001d8ca0 | CONFIRMED | steps toward `next_node` at `walk_speed`; on arrival `pumper_choose_next_action` |
| 0x004d0e80 | `pumper_ai_turn` | `AI_turn__8C_pumperFUiRC8S_vector` | 0x001d8fb0 | CONFIRMED | faces the target; beyond ±π/4 it starts a turn clip and returns 0 |
| 0x004d1000 | `pumper_update` | `Update__8C_pumperFUi` | 0x001d91b0 | CONFIRMED | slot 14; the pump-frame watch, the frame pose, the ground resnap, both coll objects, the smoke timers |
| 0x004d12d0 | `pumper_snap_to_ground` | `ground__8C_pumperFv` | 0x001d9500 | CONFIRMED | a vertical line test from 1.0 above him, ignoring his two coll objects |
| 0x004d1380 | `pumper_set_anim` | `SetAnim__8C_pumperFf` | 0x001d95d0 | CONFIRMED | starts `anim_pending` when it differs from `anim_applied` |
| 0x004d13f0 | `pumper_choose_next_action` | `ChooseNextAction__8C_pumperFv` | 0x001d9680 | CONFIRMED | the transition table of §2a |
| 0x004d1bb0 | `pumper_advance_to_next_node` | `NextNode__8C_pumperFv` | 0x001d9e30 | CONFIRMED | walks the node's connections down to the first flag-`2` node that is not `prev_node` |
| 0x004d1c30 | `pumper_start_refuel` | `Start__8C_pumperFP5C_car` | 0x001d9ec0 | CONFIRMED | §5; sole caller `player_do_action_button_in_car` |
| 0x004d2070 | `pumper_can_pump` | `CanPump__8C_pumperFP5C_car` | 0x001da1e0 | CONFIRMED | §5a; 0 = refused silently, 1 = can, 2 = refused or message shown |
| 0x004d2840 | `pumper_start_pump_fuel` | `StartPumpFuel__8C_pumperFv` | 0x001da910 | CONFIRMED | clip event `0x2a`: `fuel_flowing` = 1, starts the pump loop (slot 3) |
| 0x004d2870 | `pumper_end_pump_fuel` | `EndPumpFuel__8C_pumperFv` | 0x001da970 | CONFIRMED | clip event `0x2b`: `fuel_flowing` = 0, tank to capacity, score − 500, stops slot 3 |
| 0x004d28c0 | `pumper_init_sounds` | `InitSounds__8C_pumperFv` | 0x001da9d0 | CONFIRMED | `pumper_init_sound` for slots 0..5 |
| 0x004d28e0 | `pumper_init_sound` | `InitSound__8C_pumperFUi` | 0x001daa20 | CONFIRMED | takes one slot from the mission sound cache, else builds it from `sound_ids` |
| 0x004d29a0 | `pumper_release_sounds` | `ReleaseSounds__8C_pumperFv` | 0x001dab40 | CONFIRMED | gives every slot back to the mission sound cache |
| 0x004d2a70 | `pumper_panic` | `PumperPanic__8C_pumperFv` | 0x001dac70 | CONFIRMED | §5; returns the new model frame, or 0 while he is working |
| 0x004d2c30 | `pumper_save_game_get_size` | `SaveGameGetSize__8C_pumperFv` | 0x001dae60 | CONFIRMED | slot 22; `actor_save_game_get_size() + 0x60` |
| 0x004d2c40 | `pumper_save_game_save` | `SaveGameSave__8C_pumperFPPv` | 0x001dae80 | CONFIRMED | slot 23; record type 5 |
| 0x004d2e80 | `pumper_save_game_load` | `SaveGameLoad__8C_pumperFPPv` | 0x001db150 | CONFIRMED | slot 24; the same record back |
| 0x004d3210 | `pumper_play_sound` | `PlaySound__8C_pumperFUib` | 0x001db510 | CONFIRMED | slot's play / stop by the bool argument |
| 0x004d3240 | `pumper_release_colls` | `ReleaseColls__8C_pumperFv` | 0x001db560 | CONFIRMED | `collision_dyn_erase` + free, for both coll objects |
| 0x004d32b0 | `pumper_is_pumping` | `IsPumping__8C_pumperFv` | 0x001db5c0 | CONFIRMED | true for action `0` and `2..10`: *working*, not *fuel is flowing* |
| 0x00574a40 | `stub_noop_cdecl_00574a40` | `Death__8C_pumperFv` | 0x001d8230 | base-stub | slot 29, the `C_actor` default; `pumper_panic` and `pumper_dtor` call it too |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004d2460 | `pumper_notify_callback` | `pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` | 0x001da5b0 | CONFIRMED | §4; registered on the `notify` child frame |
| 0x004d29e0 | `pumper_get_special_anim_name` | `GetSpecAnimName__8C_pumperF18E_anim_pumper_spec` | 0x001dabc0 | CONFIRMED | maps ids `0x10000..0x10008` to the clip names of §5 |
| 0x004d32e0 | `pumper_script_can_pump` | `ScriptCanPump__8C_pumperFUi` | 0x001db610 | CONFIRMED | finds the kind-`0x19` actor with `script_id` == argument in the mission list; -1 none, 0 in panic, 1 fine; caller `program_process_command` |

## 4. `pumper_notify_callback` — the seam between clip and gameplay

`pumper_game_init` registers it on the attendant's `notify` child frame. The animation machine fires
message ids out of the clip, the callback finds the actor through the frame (kind must be `0x19`) and
turns them into actions:

| Message | Effect |
|---|---|
| `0x2a` | `pumper_start_pump_fuel`: **the fuel transfer begins here** |
| `0x2b` | `pumper_end_pump_fuel`: and ends here |
| `1`, `2` | sound slot 0, the footstep |
| `0x2e` / `0x2f` / `0x30` / `0x31` | sound slots 1 / 5 / 2 / 4: hose up, hose down, open, close |
| `0x3c` / `0x3d` | show the pack (`pack_hand`) / the cigarette (`cigarette_hand`) |
| `0x3e` | smoke particles (types `0x18`, `0x27`) from the two cigarettes; `smoke_period` = 1000 |
| `0x3f` / `0x40` | hide the pack / hide the cigarette and destroy `cigarette_smoke` |
| `0x41` | exhale: particle type `0x19` at the `neck` frame into `mouth_smoke`; `smoke_period` = 6500 |
| `0x42` / `0x43` | move the pack from the right hand to the left hand / back |
| `0x44` / `0x45` | move the cigarette from the right hand to the mouth / back |

The refuel is **animation-driven**, not timer-driven: the money changes hands when the clip reaches
its notify frame.

## 5. What the attendant does

* **Idle.** At the pump node, in actions 1 and 0xb..0x10: `pumper_get_special_anim_name` supplies
  `Nuda01`, `Nuda02`, `Nuda03`, `NudaStativ` (boredom) and `KurakZapaleni`, `KurakPotahnuti`,
  `KurakLoop`, `KurakZahozeni` (the smoker: light up, drag, smoke, throw away), and `turn1`.
* **The request.** `player_do_action_button_in_car`, the action button while sitting in a car at a
  station, finds the kind-`0x19` actor and calls `pumper_can_pump`; on result 1 it calls
  `pumper_start_refuel`. That function asks `pumper_can_pump` again and enters action 7: it speaks
  line 28940023 on his own model, hides the props and destroys the three particles, swings the car to
  the station's forward row, moves it so its FUEL dummy (`car+0x2078`) sits on `station_frame`,
  deactivates it (`car+0x211c` + 1, engine off, the player's controls locked), and pushes a camera
  looking from `camera_frame` at his `base` frame.
* **The walk.** Action 8 falls into action 0: `pumper_ai_walk` steps toward `next_node`,
  `pumper_ai_turn` faces it and `pumper_advance_to_next_node` picks the next one.
* **Speech.** Four lines: 28940023 and his own `speech_line_id` on his model, 14940003 and 9990501 on
  the player's model (§2a).
* **Refuel.** At the post node action 10 computes `fuel_per_ms` = (`car+0x1f8c` capacity − `car+0xd1c`
  fuel) / `refuel_duration` and starts the hose. The clip events do the rest (§4): while `fuel_flowing`
  action 5 adds fuel every frame; `pumper_end_pump_fuel` sets the tank to capacity through
  `vehicle_set_fuel` and calls **`game_freeride_score_set(score − 500)`**.
  Back at the pump node, action 0 hands the car back: it moves it by the forward row, `car_reset`,
  `camera_set_car`, engine on, `car+0x211c` − 1, controls unlocked, `served_car` = 0.
* **Panic.** `pumper_hit` and `pumper_on_collision` (the latter only for an actor other than
  `served_car`) act **only when `pumper_is_pumping` is false**, that is in action 1 or above 10; in
  action 0 and 2..10 he ignores both. `pumper_update` also calls `pumper_panic` when the pump frame is
  off, and hides the hose. `pumper_panic` sets action `0x11`, hides the props and destroys the
  particles, builds a model frame (type 9, `g_i3d_driver` slot `0x4c`) as a copy of his own, gives it
  to the crowd (`pedestrian_manager_add_free_person`, `pedestrian_manager_new_panic_at_frame`), hides
  his own frame, and releases the coll objects and sounds. Its result goes to the crowd proxy
  `C_game+0x2a1c` (`structs/pedestrian.md`) from `pumper_hit` and `pumper_on_collision`.
* **The respawn.** In action `0x11`, once the pump frame is on and the camera (`*(g_act_mission+0x10)`
  `+0x17c`) is over 200 units from his frame, `pumper_ai` puts him back: action 1 at the pump node,
  facing the first connection, frame shown, new coll objects, sounds rebuilt, snapped to the ground.

### 5a. `pumper_can_pump` — the checks in order

| Check | On failure |
|---|---|
| `station_frame` set and `action != 0x11` | returns 0 |
| the car's FUEL dummy within **6.0** of the station | returns 0 |
| no owner under `police_manager_are_cops_after_player` | banner textdb 805 *"First I gotta lose the cops!"*, returns 2 |
| `pursuit_event_is_car_chased` false | the same banner, returns 2 |
| the car's position within **4.0** of the station, and its forward within π/4 of the station's | banner textdb 804 *"Move to the pump to refuel."*, returns 2 |
| speed (`car+0x688`) below 0.1001 | returns 2, no banner |
| all passed | returns 1 |

`police_manager_are_cops_after_player` has this one caller. It is true if any live officer is assigned
to the player, **or** if a pursuit record for him carries a committed police_state (`+0x3c`) or the
wanted substate (`+0x50`) with identity settled (`+0x51 == 0`).

## 6. The six sound slots

`pumper_init_sound` hands the id of each slot to `sound_table_construct_sound`. The ids resolve in the
shipped `tables\sounds.dat` (`structs/sound_table.md` §5). Every shipped attendant record in the scene
files stores exactly the constructor defaults below.

| Slot | Id | `sounds.dat` record (group) | File | Played by |
|---|---|---|---|---|
| 0 | 30 | `Asfalt` (Steps) | `s011.wav` .. `s015.wav` | clip events 1, 2 |
| 1 | 121 | `pumpa_hadice up` (Actors) | `pumpa1.wav` | clip event `0x2e` |
| 2 | 122 | `pumpa_open` (Actors) | `pumpa2.wav` | clip event `0x30` |
| 3 | 123 | `pumpa_tankovani` (Actors) | `pumpa3.wav` | `pumper_start_pump_fuel`, a loop |
| 4 | 124 | `pumpa_close` (Actors) | `pumpa4.wav` | clip event `0x31` |
| 5 | 125 | `pumpa_hadice down` (Actors) | `pumpa5.wav` | clip event `0x2f` |

**Savegame:** record type **5**, payload `actor_save_game_get_size() + 0x60`: `unread_cc` `+4`,
`action` `+8`, position `+0xc`, forward `+0x18`, `anim_pending` `+0x24`, both clip flags `+0x28`, the
clip's progress as a fraction `+0x2c`, the hose's time `+0x30`, the three nodes as indices `+0x34`, the
served car's `+0x18` id `+0x40`, `refuel_duration`, `fuel_per_ms`, `fuel_flowing`, `dwell_period`,
`dwell_timer`, `repeat_count`, and the shown bit of the four cigarette frames `+0x59..+0x5c`.
`pumper_save_game_load` restores every saved field except `refuel_duration` (record `+0x44`).

## 7. The scene props it needs

`pumper_game_init` formats every frame name from `script_id` and looks it up in the scene. The
vocabulary, from the string literals in that function:

`_pumpa_pumpa_%i` · `_pumpa_hadice_%i` (the hose) · `_pumpa_cigaro_%i` / `_pumpa_cigaro_box_%i` ·
`_pumpa_xcigaro_%i` / `_pumpa_xcigaro_box_%i` · `_pumpa_camera_%i` · `_pumpa_%i`, plus the `base` and
`notify` child frames and the two FMV cigarette models `FMVcigara.i3d` / `FMVcigareta2.I3D`.

No shipped scene carries the four cigarette frames, so `pumper_game_init` always builds them from the
FMV models: `FMVcigareta2.I3D` for the two cigarettes (linked to `r_hand` and `neck`) and
`FMVcigara.i3d` for the two packs (linked to `r_hand` and `l_hand`). The hose and the pump frames are
scene frames.

## 8. PC vs PS2

**The field map.** The delta between the builds is not uniform, so the map is not a rebase: it pairs
the two constructors statement by statement (both initialise the members in declaration order) and
checks each pair against a body that uses the field on both builds. The action id is the one field no
constructor writes; `IsInPanic`, `IsPumping` and `PumperPanic` read or write it on both builds.

| PC | PS2 | Field | Witness |
|---|---|---|---|
| `+0x10` | `+0x50` | kind `0x19` | ctor |
| `+0x68` | `+0x70` | frame | `Update` |
| `+0x70` / `+0x74` | `+0x108` / `+0x10c` | `walk_speed` / `second_speed` | ctor |
| `+0x78` / `+0x7c` | `+0xc4` / `+0xc8` | `pump_node_id` / `post_node_id` | ctor, `SaveData`, `DuplicateFrom` |
| `+0x80` | `+0xcc` | `script_id` | ctor, `SaveData`, `DuplicateFrom` |
| `+0x84` | `+0xd0` | `speech_line_id` | ctor, `SaveData`, `ChooseNextAction` |
| `+0x88..+0x9c` | `+0xd4..+0xe8` | `sound_ids` | ctor, `SaveData` |
| `+0xa0` / `+0xa4` / `+0xa8` | `+0xec` / `+0xf0` / `+0xf4` | the particle handles | ctor |
| `+0xac` / `+0xb0` | `+0x130` / `+0x134` | `smoke_timer` / `smoke_period` | the notify callback (1000 / 6500 ms) |
| `+0xb4..+0xc8` | `+0x118..+0x12c` | `sounds` | ctor |
| `+0xcc` | `+0x110` | `unread_cc` | ctor |
| `+0xd0` | `+0xc0` | `action` | `IsInPanic`, `IsPumping`, `PumperPanic` |
| `+0xd4` / `+0xd8` | `+0xf8` / `+0xfc` | `anim_applied` / `anim_pending` | ctor |
| `+0xdc` / `+0xdd` | `+0x290` / `+0x291` | the clip flags | ctor |
| `+0xe0` | `+0x150` | `anim` | ctor |
| `+0x208` | `+0x29c` | `hose_anim_set` | ctor, `GameInit` |
| `+0x20c` / `+0x210` / `+0x214` | `+0x2c0` / `+0x2c4` / `+0x2c8` | the three nodes | ctor |
| `+0x218` | `+0x294` | `station_frame` | ctor |
| `+0x220` / `+0x224` | `+0x2cc` / `+0x2d0` | `coll_box` / `coll_sphere` | `Update` |
| `+0x228` / `+0x22c` | `+0x138` / `+0x13c` | `resnap_timer` / `resnap_period` | `Update` |
| `+0x230` | `+0x104` | `speech_channel` | `ChooseNextAction` |
| `+0x234` | `+0x2a0` | `hose` | `Update` |
| `+0x238` / `+0x23c` / `+0x240` / `+0x244` | `+0x2a4` / `+0x2a8` / `+0x2ac` / `+0x2b0` | the four cigarette props | the notify callback |
| `+0x248` | `+0x2b4` | `pump_frame` | `Update` |
| `+0x24c` | `+0x2bc` | `served_car` | `EndPumpFuel` |
| `+0x250` | `+0x100` | `refuel_duration` | ctor, `ChooseNextAction` |
| `+0x254` | `+0x114` | `fuel_per_ms` | ctor, `AI`, `ChooseNextAction` |
| `+0x258` | `+0x2d4` | `fuel_flowing` | `StartPumpFuel`, `EndPumpFuel` |
| `+0x25c` / `+0x260` | `+0x140` / `+0x144` | `dwell_timer` / `dwell_period` | ctor (both reset it twice) |
| `+0x264` | `+0x2d5` | `repeat_count` | ctor, `ChooseNextAction`, the savegame pair |
| `+0x2a8` | `+0x2b8` | `hose_parent` | ctor, `GameDone` |

The three timers sit together before the animation machine on the PS2 and around it on the PC; each is
paired by its own witness.

Other cross-build notes:

* `IdePoliciePoPlayeroviBoPumper__10C_shvestky` (*"are the police after the player, for the pumper"*)
  is the PS2 name of `police_manager_are_cops_after_player`.
* `pumper_notify_callback`'s PS2 symbol has `__F` and no class tag, and it sits at the same rank
  (between `CanPump` and `StartPumpFuel`) on both builds.
* The refuel charge of **500** in `pumper_end_pump_fuel` is identical on PS2.
* The speech line default `0x497cc` and the speech calls of `ChooseNextAction` are identical on PS2; the
  PS2 notify callback restarts the timer for messages `0x3e` and `0x41` without spawning the smoke
  particles the PC spawns.
* PS2 `Do_AB_OwnerCar__8C_player` calls `Start__8C_pumperFP5C_car`; its PC twin `pumper_start_refuel` is
  correspondingly a callee of `player_do_action_button_in_car`.
* `Init` and `Death` are real 44 B / 8 B PS2 methods whose PC realisations are **shared** stubs, so they
  are roles, not twins.
* The MSVC scalar deleting destructor has no PS2 symbol by construction.
* Node stride in the scene node array: PC `0x1e`, PS2 `0x20`.
* PC `C_game+0x2a1c` ↔ PS2 `C_game+0xe00`, the `C_don_ocas` pedestrian-crowd proxy that
  `pumper_on_collision` / `pumper_hit` forward to, an independent witness for `structs/pedestrian.md`'s
  offset.

## 9. Related

`structs/anim_machine.md` §4 (the machine embedded at `+0xe0`) · `structs/entity_kind.md` §1 (the kind
map) · `structs/pedestrian.md` (the `C_don_ocas` proxy) · `structs/police_manager.md` (the pursuit
record `police_manager_are_cops_after_player` reads) · `structs/sound_table.md` (the `sounds.dat` bank)
· `structs/speech.md` (the speech channels) · `structs/car.md` §1 (the FUEL dummy).

## 10. Open items

1. **In game: the charge.** Does the 500 charge apply exactly as the code says, and does refuelling
   still work while a *different* car of yours is under pursuit? `pumper_can_pump` tests the owner,
   not the car you sit in.
2. **In game: panic.** Shoot the attendant in action 1 and watch the crowd take over his model; walk
   200 units away and watch him come back. Shots in action 0 or 2..10 should do nothing.
3. **Fields with no reader.** `second_speed` has only the ctor's store (0.5) and `third_node` only the
   ctor's 0 and the savegame round-trip as a node index; neither has a reader (`.text` scan). Their
   meaning is not measured.
