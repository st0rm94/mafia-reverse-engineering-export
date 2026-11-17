# `C_bottle` — the small thrown-physics-prop base class

**Created** 2026-08-05 12:38 +0200 · **Modified** 2026-11-09 10:33 +0100

A `C_actor` subclass: a small prop that flies a ballistic arc, tumbles, bounces off people,
reacts to being shot, and is replicated to other machines. Its own functions are the clearest
worked example of the engine's actor contract — lifecycle, savegame, collision, damage and
network replication.

**It cannot be picked up, and a `C_bottle` is never thrown** (§7b): the class has no
`S_using_object`, no use handler and no `GetKind == 0xa` test anywhere in the image, and all four
callers of `bottle_set_flight_impulse` spawn a **subclass** (`C_grenade` — the molotov —
`C_cartridge`, `C_smoke_trail`). The throw API belongs to the family, not to the leaf. The two
instances the shipped game places are **tin cans** (§7a).

| | |
|---|---|
| PC prefix | `bottle_` |
| PC kind (`+0x10`) | `0xa`, stamped by `bottle_ctor` ¹ |
| PC vtable | `0x0063b870`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§4) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x138` (`operator_new(0x138)` in `mission_create_actor` case `0xa`) |
| PC constructor | `bottle_ctor` @ `0x00453ad0` |
| PC translation unit | `[0x00453a20, 0x00456460)` (§8) |
| PS2 class | `C_bottle` (tag `8C_bottle`), 17 methods: 13 CONFIRMED twins, 1 STRUCTURAL and 3 base stubs, all mapped in §3 |

¹ Byte-proven from the ctor and corroborated twice: `structs/entity_kind.md` documents kind `0xa`,
and `recorder_new_track`'s kind→prefix switch falls through to `unk_` for it, so the recorder does
not track bottles.

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC. The savegame pair is the
backbone: `bottle_save_game_save` writes 22 fields (inherited and own) in one order and
`bottle_save_game_load` reads the same 22 back, so every field marked *saved* in §2 has two
independent witnesses.

```c
typedef struct C_bottle {                   // 0x138 bytes; PC offsets
    /* +0x000 */ C_actor    base;               // [H] the C_actor band (structs/actor.md §1)
    /* +0x070 */ S_vector   flight_dir;         // [H] normalised launch direction
    /* +0x07c */ float      flight_speed;       // [H] |impulse|; shrinks on every bounce
    /* +0x080 */ float      fall_accum;         // [H] downward speed; the flight ends past 50.0
    /* +0x084 */ uint8_t    end_sets_act_state; // [H] 1 = the flight end changes the act state
    /* +0x088 */ float      restitution;        // [H] speed kept per bounce; 0.4 by GameInit
    /* +0x08c */ S_vector   contact_normal;     // [H] ground normal of the last bounce
    /* +0x098 */ S_vector   tumble_rate;        // [H] 3 x rand_float; re-rolled on every impact
    /* +0x0a4 */ float      radius;             // [H] half the bound-volume diagonal
    /* +0x0a8 */ S_vector   frame_axis;         // [H] second SetDir3 axis, tumbled each tick
    /* +0x0b4 */ tDynamicCollObject coll;       // [H] 0x48 bytes, zeroed by the ctor
    /* +0x0fc */ uint8_t    coll_registered;    // [H] 1 = coll is inserted in g_collision
    /* +0x100 */ int32_t    bounce_count;       // [H] +1 per ground contact
    /* +0x104 */ uint8_t    on_ground;          // [H] the ground ray hit this tick
    /* +0x108 */ S_vector   net_pos;            // [H] last received authoritative position
    /* +0x114 */ S_vector   net_vel;            // [H] its velocity
    /* +0x120 */ float      net_gravity;        // [H] the fall term applied while extrapolating
    /* +0x124 */ uint32_t   ms_since_broadcast; // [H] time since the last state broadcast
    /* +0x128 */ uint8_t    broadcast_on;       // [H] 1 = the state broadcast runs
    /* +0x12c */ int32_t    ground_material;    // [H] surface material of the ground hit
    /* +0x130 */ void      *ground_dyn_obj;     // [H] owner of the proxy the ground ray hit, else 0
    /* +0x134 */ uint8_t    ray_all_layers;     // [H] ground-ray mask: 0 = mask 2, 1 = every layer
} C_bottle;
```

The class has no array field. The `S_vector` fields are three floats each; `coll` is one
`tDynamicCollObject` whose own members `bottle_game_init` fills (`0x42` at `+0x00`, frame `+0x04`, kind `+0x08`,
position `+0x10`, radius `+0x1c`, owner `+0x44`) and `bottle_update` keeps the position of.

## 2. Field evidence

Measured by a census of every operand with each displacement in the `bottle_*`, `cartridge_*`,
`grenade_*` and `smoke_trail_*` functions, then read in the bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `bottle_ctor` (through `actor_ctor`; kind `+0x10` = `0xa`, `may_broadcast` `+0x6d` = 1) | `folded_get_dword_0x10` (the kind), `bottle_on_collision`; the methods of §3 | of the base band this class uses the guid `+0x18` (its high byte selects the owner), the owner slot `+0x1b` (`net_session_get_peer_clock_ms`), `+0x20` (this machine is not authoritative: `bottle_on_collision` and `bottle_hit` report upstream instead of simulating), the position `+0x24` and frame direction `+0x30` (saved; `+0x30` feeds `S_matrix::SetDir3`), the frame `+0x68`, and `may_broadcast` `+0x6d` (`structs/actor.md`), half of the host-role broadcast guard (§5a) |
| +0x070 | flight_dir | H | `bottle_set_flight_impulse`, `bottle_apply_net_event_msg`, `bottle_ai_tick`, `bottle_save_game_load` | `bottle_ai_tick`, `bottle_on_collision`, `bottle_hit`, `bottle_game_init`, `bottle_post_state_net_event_msg`, `bottle_save_game_save` | saved |
| +0x07c | flight_speed | H | `bottle_set_flight_impulse`, `bottle_apply_net_event_msg`, `bottle_game_init`, `bottle_ai_tick` (× `restitution` on a bounce), `bottle_on_collision`, `bottle_hit`, `bottle_save_game_load` | `bottle_ai_tick`, `bottle_on_collision`, `bottle_hit`, `bottle_game_init`, `bottle_post_state_net_event_msg`, `cartridge_ai_tick`, `smoke_trail_ai_tick`, `bottle_save_game_save` | saved |
| +0x080 | fall_accum | H | `bottle_set_flight_impulse`, `bottle_game_init`, `bottle_ai_tick` (`+= dt × 0.01`), `cartridge_ai_tick`, `grenade_ai_tick`, `bottle_save_game_load` | `bottle_ai_tick`, `bottle_game_init`, `bottle_post_state_net_event_msg`, `cartridge_ai_tick`, `grenade_ai_tick`, `smoke_trail_ai_tick`, `bottle_save_game_save` | saved. **The flight ends when it passes 50.0** |
| +0x084 | end_sets_act_state | H | `bottle_game_init` (= 1), `cartridge_game_init`, `grenade_game_init`, `smoke_trail_game_init` (= 0), `bottle_save_game_load` | `bottle_ai_tick`, `bottle_save_game_save` | saved. At the flight end (`fall_accum` past 50.0, or speed and fall both at rest) a set flag switches the act state — 2 after the fall cut-off, 1 at rest; a clear flag only clears `broadcast_on`. So a bottle leaves the active list when it lands and the three subclasses stay in it |
| +0x088 | restitution | H | `bottle_game_init` (= 0.4), `cartridge_set_type` (0.2 for type B) | `bottle_ai_tick` | on a bounce the new speed is the old speed × `restitution` |
| +0x08c | contact_normal | H | `bottle_ai_tick` (each bounce), `bottle_game_init` (clear), `bottle_save_game_load` | `bottle_on_collision`, `bottle_hit`, `bottle_save_game_save` | saved. Sent in the type-`0x15` collision message and added × 0.01 as a push-off |
| +0x098 | tumble_rate | H | `bottle_game_init`, `bottle_on_collision`, `bottle_hit` (3 × `rand_float` each), `bottle_apply_net_event_msg` | `bottle_ai_tick`, `bottle_net_apply_remote_state` | re-rolled on every impact, which is why a bottle never tumbles the same way twice |
| +0x0a4 | radius | H | `bottle_game_init`, `cartridge_game_init`, `smoke_trail_game_init`, `bottle_save_game_load` | `bottle_ai_tick`, `bottle_game_init`, `bottle_save_game_save` | saved. Half the diagonal of the frame's hierarchical bound volume, after world scale |
| +0x0a8 | frame_axis | H | `bottle_ai_tick`, `bottle_net_apply_remote_state` (rotated by the tumble each tick), `bottle_game_init`, `cartridge_game_init` (= (0, 1, 0)), `smoke_trail_game_init`, `bottle_save_game_load` | `bottle_update` (`S_matrix::SetDir3`, pushed after `+0x30`; the decomp text shows only `+0x30`), `bottle_save_game_save` | saved |
| +0x0b4 | coll | H | `bottle_ctor` (zeroed), `bottle_game_init` (filled, `collision_dyn_insert`), `bottle_update` (position mirrored from `+0x24`) | `bottle_ai_tick`, `bottle_game_done` (`collision_dyn_erase`) | - |
| +0x0fc | coll_registered | H | `bottle_ctor` (= 1), `cartridge_ctor`, `smoke_trail_ctor` (= 0) | `bottle_game_init`, `bottle_update`, `bottle_game_done` | gates the insert, the update and the erase of `coll` |
| +0x100 | bounce_count | H | `bottle_game_init` (= 0), `bottle_ai_tick`, `grenade_on_collision`, `grenade_hit`, `bottle_save_game_load` | `bottle_ai_tick` (its own increment), `cartridge_ai_tick` (> 15 retires a casing), `grenade_ai_tick`, `bottle_save_game_save` | saved. In the other `bottle_*` bodies this displacement is `g_net_session+0x100` |
| +0x104 | on_ground | H | `bottle_game_init`, `bottle_ai_tick` (each ground ray), `bottle_save_game_load` | `cartridge_ai_tick`, `grenade_ai_tick`, `bottle_save_game_save` | saved. Clears when the flight ends at a speed below 3.0 on a surface within 0.5 rad of level |
| +0x108 | net_pos | H | `bottle_apply_net_event_msg` (type `0x16`), `bottle_game_init` (the position) | `bottle_net_apply_remote_state`, `bottle_apply_net_event_msg` (type `0x13`, state 1: snaps the position to it), `grenade_apply_net_event_msg` (type `0x16`, reason 5: the same) | §5a |
| +0x114 | net_vel | H | `bottle_apply_net_event_msg` (type `0x16`), `bottle_game_init` (`flight_dir` × `flight_speed`) | `bottle_net_apply_remote_state` | - |
| +0x120 | net_gravity | H | `bottle_apply_net_event_msg`, `bottle_game_init`, `bottle_net_apply_remote_state` | `bottle_net_apply_remote_state` | - |
| +0x124 | ms_since_broadcast | H | `bottle_game_init`, `bottle_ai_tick` (`+=` the session frame time `g_net_session+0x98`), `bottle_post_state_net_event_msg` (= 0) | `bottle_ai_tick` | once it exceeds the session setting at `+0x10` of `net_session_game_settings_addr`, `bottle_ai_tick` broadcasts the state |
| +0x128 | broadcast_on | H | `bottle_ctor` (= 1), `bottle_ai_tick` (= 0 at the flight end, §2 `+0x084`) | `bottle_post_state_net_event_msg` | the store runs only when `end_sets_act_state` is clear, so on the three subclasses; a bottle changes its act state at the flight end instead |
| +0x12c | ground_material | H | `bottle_game_init`, `bottle_ai_tick`, `bottle_save_game_load` | `cartridge_ai_tick`, `grenade_ai_tick`, `bottle_save_game_save` | saved. `cartridge_ai_tick` picks the landing sound from it and `on_ground` |
| +0x130 | ground_dyn_obj | H | `bottle_game_init`, `bottle_ai_tick` (each ground ray; 0 when the hit is static) | `grenade_ai_tick` | the hit proxy's `+0x44`. When a grenade's fire starts, `grenade_ai_tick` takes this object's frame (`+0x68`), converts the fire position into its local space and passes the frame to `game_new_fire`, so the fire moves with whatever the projectile landed on |
| +0x134 | ray_all_layers | H | `bottle_game_init` (= 0), `cartridge_game_init` (= 1) | `bottle_ai_tick` | selects the ground-ray mask: 2 when clear, every layer (`0xffffffff`) when set |

## 3. Method family

Coverage: all **17** PS2 `8C_bottle` methods are in the tables below — **13 CONFIRMED twins**,
**1 STRUCTURAL** (`NetDirect`, whose PS2 body is empty, §9a) and **3 base stubs**. The
scalar-deleting destructor and the three network functions are PC-only; the CRT pair belongs to
the translation unit.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00453a20 | `crt_static_init_guard_00453a20` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00453a40 | `crt_atexit_register_00453a40` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00453ad0 | `bottle_ctor` | instance | `__ct__8C_bottleFv` | 0x00122f40 | CONFIRMED | `actor_ctor`, zero the proxy, vtable, kind `0xa`, the three flags `may_broadcast` `+0x6d`, `coll_registered` `+0xfc`, `broadcast_on` `+0x128` |
| 0x00453b10 | `bottle_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor, vtable slot 16 ¹ |
| 0x00453b30 | `bottle_dtor` | instance | `__dt__8C_bottleFv` | 0x00122fb0 | CONFIRMED | restores the vtable, then `actor_dtor`; the class owns no heap memory |

¹ No standalone PS2 twin: GCC carries the free flag inside `__dt__8C_bottleFv`.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00453b40 | `bottle_init` | `Init__8C_bottleFP9I3D_frame` | 0x00123010 | CONFIRMED | slot 18; null guard, `actor_init`, `actor_debustize_frame` |
| 0x00453b70 | `bottle_ai_tick` | `AI__8C_bottleFUi` | 0x00123080 | CONFIRMED | slot 13; the ballistic arc, the ground ray, the bounce, the flight end; size ratio 1.01 |
| 0x00454840 | `bottle_update` | `Update__8C_bottleFUi` | 0x00123d40 | CONFIRMED | slot 14; the transform into the I3D frame with both shared frame masks; mirrors the proxy position |
| 0x00454920 | `bottle_net_apply_remote_state` | `NetDirect__8C_bottleFUi` | 0x00123e50 | STRUCTURAL | slot 15; dead reckoning and smoothing from the `net_*` band; the PS2 body is an empty stub (§9a) |
| 0x00455160 | `bottle_game_init` | `GameInit__8C_bottleFv` | 0x00123e60 | CONFIRMED | slot 25; clears the flight band, rolls the tumble rate, measures the radius, registers the proxy; size ratio 1.04 |
| 0x004554b0 | `bottle_game_done` | `GameDone__8C_bottleFv` | 0x00124160 | CONFIRMED | slot 26; a guarded `collision_dyn_erase`, then the base |
| 0x004554e0 | `bottle_save_game_get_size` | `SaveGameGetSize__8C_bottleFv` | 0x001241a0 | CONFIRMED | slot 22; the base size plus `0x53` |
| 0x004554f0 | `bottle_save_game_save` | `SaveGameSave__8C_bottleFPPv` | 0x001241c0 | CONFIRMED | slot 23; version byte 1, then 22 stores in the same order as the PS2 |
| 0x004555d0 | `bottle_save_game_load` | `SaveGameLoad__8C_bottleFPPv` | 0x001242d0 | CONFIRMED | slot 24; the version-gated mirror |
| 0x00455710 | `bottle_set_flight_impulse` | `AbsoluteImpulse__8C_bottleFR8S_vector` | 0x00124490 | CONFIRMED | launches the arc; non-virtual; size ratio 0.99 |
| 0x004558d0 | `bottle_on_collision` | `Collision__8C_bottleFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00124660 | CONFIRMED | slot 30; the identical kind gate `{2, 10, 0x1b}`; size ratio 1.06 |
| 0x00455bc0 | `bottle_hit` | `Hit__8C_bottleF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00124930 | CONFIRMED | slot 31; the identical hit-type gate `{0, 1, 2}` and `abs(y) × 15.0`; size ratio 0.99 |
| 0x00455eb0 | `bottle_apply_net_event_msg` | - | - | - | slot 38; PC-only; message types `0x13`, `0x15`, `0x16` (§5) |
| 0x00456200 | `bottle_post_state_net_event_msg` | - | - | - | PC-only; posts type `0x16`, descriptor `0x0063b92c` |
| 0x00456310 | `bottle_post_collision_net_event_msg` | - | - | - | PC-only; posts type `0x15`, descriptor `0x0063b93c` |
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__8C_bottleFP7C_actor` | 0x00123070 | base-stub | slot 21; a 5-byte `jmp` to the shared one-argument no-op ¹ |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__8C_bottleFP7C_chunk` · `LoadData__8C_bottleFP7C_chunk` | 0x00124c30 · 0x00124c40 | base-stub | slots 19 and 20 — a **2 → 1 fold**; both PS2 bodies are 8-byte stubs. A bottle carries no definition-record state |
| 0x0048af90 | `bottle_subclass_init_require_frame` | - | - | - | slot 18 of the three subclasses' vtables, not of this one: `frame ? bottle_init(frame) : false`, an ICF merge of identical `Init` bodies: `C_grenade`'s and `C_smoke_trail`'s PS2 `Init` are statement-identical to it, and `C_cartridge` has no PS2 methods (§6) |

¹ Ghidra names a thunk after its target, so the slot resolves through the Ghidra symbol table,
not from an id: there is no `FUN_004773f0`, but the thunk is a real function.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00455480 | `vec3_div_scalar` | - | - | - | a `S_vector` division emitted out of line here; 8 callsites in 6 functions, one of them `bottle_net_apply_remote_state`, the others in `C_PhThing` and `g_collision` |

## 4. The vtable

`0x0063b870`, 39 slots, PC slot order. Slots 0–12 are the `C_actor_track` interface every
`C_actor`-family class carries (`structs/actor_track.md`).

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 0 | 0x0044a780 | `stub_retminus1_args4_0044a780` | - |
| 1 | 0x0044a790 | `actor_track_apply_key_texture_anim` | - |
| 2 | 0x0044a860 | `actor_track_apply_pose_at_clock` | - |
| 3 | 0x0044ad10 | `actor_track_apply_pose_at_time` | - |
| 4 | 0x0044b250 | `actor_track_get_pos_at_time` | - |
| 5 | 0x0044b470 | `actor_track_get_rot_at_time` | - |
| 6 | 0x00548a60 | `actor_track_set_recording` | - |
| 7 | 0x00548a80 | `actor_track_set_playback` | - |
| 8 | 0x00548aa0 | `actor_track_apply_mode_changes` | - |
| 9 | 0x0044a650 | `actor_track_create` | - |
| 10 | 0x00548950 | `actor_track_key_stream_size` | - |
| 11 | 0x00548900 | `actor_track_key_offset_of` | - |
| 12 | 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | - |
| 13 | 0x00453b70 | `bottle_ai_tick` | `AI__8C_bottleFUi` |
| 14 | 0x00454840 | `bottle_update` | `Update__8C_bottleFUi` |
| 15 | 0x00454920 | `bottle_net_apply_remote_state` | `NetDirect__8C_bottleFUi` |
| 16 | 0x00453b10 | `bottle_scalar_deleting_dtor` | `__dt__8C_bottleFv` |
| 17 | 0x004253b0 | `actor_release` | `Release__7C_actorFv` |
| 18 | 0x00453b40 | `bottle_init` | `Init__8C_bottleFP9I3D_frame` |
| 19 | 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__8C_bottleFP7C_chunk` |
| 20 | 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__8C_bottleFP7C_chunk` |
| 21 | 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__8C_bottleFP7C_actor` |
| 22 | 0x004554e0 | `bottle_save_game_get_size` | `SaveGameGetSize__8C_bottleFv` |
| 23 | 0x004554f0 | `bottle_save_game_save` | `SaveGameSave__8C_bottleFPPv` |
| 24 | 0x004555d0 | `bottle_save_game_load` | `SaveGameLoad__8C_bottleFPPv` |
| 25 | 0x00455160 | `bottle_game_init` | `GameInit__8C_bottleFv` |
| 26 | 0x004554b0 | `bottle_game_done` | `GameDone__8C_bottleFv` |
| 27 | 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | `ChangeState__7C_actorFv` |
| 28 | 0x0044e8a0 | `actor_get_actor_speed` | `GetActorSpeed__7C_actorFR8S_vector` |
| 29 | 0x00574a40 | `stub_noop_cdecl_00574a40` | `Death__7C_actorFv` |
| 30 | 0x004558d0 | `bottle_on_collision` | `Collision__8C_bottleFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` |
| 31 | 0x00455bc0 | `bottle_hit` | `Hit__8C_bottleF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` |
| 32 | 0x00425420 | `stub_ret0_args4_00425420` | `GetMaterialID__7C_actorFP9I3D_frame` |
| 33 | 0x0044cf90 | `stub_retfalse_args40_0044cf90` | `IsPossibleColl__7C_actorFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` |
| 34 | 0x0044cfa0 | `actor_get_bound_points_2d` | `GetBoundPoints2D__7C_actorFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` |
| 35 | 0x00477700 | `stub_ret0_cdecl_00477700` | `GetProgram__7C_actorFv` |
| 36 | 0x004776f0 | `stub_noop_args4_004776f0` | `cbCollTester__7C_actorFP18tDynamicCollObject` |
| 37 | 0x004776f0 | `stub_noop_args4_004776f0` | `SoundsChange__7C_actorFi` |
| 38 | 0x00455eb0 | `bottle_apply_net_event_msg` | - |

The PS2 twin column is the PS2 vtable of `C_bottle` in order (PC slot = PS2 method index + 13,
`structs/actor.md` §8; the PS2 has no `C_actor_track` interface and keeps its destructor where the
PC keeps the scalar-deleting one). The PS2 `ChangeSceneInit` / `ChangeSceneDone` have no PC slot,
and slot 38, the PC-only network receiver, has no PS2 method (§9a).

## 5. Network replication

### 5a. How the network trio closes on itself

The three PC-only functions verify each other, with no PS2 side needed:

- `bottle_post_state_net_event_msg` emits type **0x16**; `bottle_apply_net_event_msg` handles
  `0x16` and writes the `net_pos` / `net_vel` / `net_gravity` band.
- `bottle_net_apply_remote_state` is the only reader of the band's velocity. The position is also
  read by the type-`0x13` arm of `bottle_apply_net_event_msg`, which snaps to it when the new act
  state is 1, and by `grenade_apply_net_event_msg`.
- `bottle_post_collision_net_event_msg` emits type **0x15**; the receiver handles `0x15` and ends
  with `actor_set_act_state(0)` — the same tail as `bottle_set_flight_impulse`, so a peer's collision
  relaunches the bottle locally. `bottle_on_collision` is the sender's only caller.
- `bottle_ai_tick` **and** `bottle_apply_net_event_msg` both re-broadcast through
  `bottle_post_state_net_event_msg` under the **byte-identical** guard
  `g_net_session+0x100 == 7 && may_broadcast != 0`. Two independent code paths agreeing on the
  host-role gate is what pins the whole loop.

The position quantise / dequantise pair the senders and the receiver use is
`net_session_encode_position` (`0x0059a8c0`) / `net_session_decode_position` (`0x0059a810`),
members of `g_net_session` (`structs/net_session.md`). The receiver's third type, **`0x13`**, is
the generic act-state change that `actor_set_act_state` posts for every actor in the host role
(`g_event_msg_vt_actor_act_state`, `structs/box.md` §6).

### 5b. The two bottle event messages

| Type | Descriptor | Serialize / deserialize | Payload |
|---|---|---|---|
| `0x16` | `g_event_msg_vt_bottle_state` (`0x0063b92c`) | `event_msg_bottle_state_serialize` / `event_msg_bottle_state_deserialize` | actor state + `u8 @+0x20` + `u32 @+0x1c` |
| `0x15` | `g_event_msg_vt_bottle_collision` (`0x0063b93c`) | `event_msg_bottle_collision_serialize` / `event_msg_bottle_collision_deserialize` | actor state + `u8 @+0x1c` |

Both extend the shared **actor-state base payload** — `{vec3 quantised to i16, 3x i16, u32
guid, u8 flags}` — which `event_msg_actor_state_serialize` writes and four derived classes
chain to. Full write-up: `structs/net_event_msg.md`.

## 6. The subclass set, and the slot-18 `Init` fold

`C_bottle` is a base; three PC subclasses are identified by kind byte:

| Kind | Class | PC ctor | Doc |
|---|---|---|---|
| `0xe` | `C_grenade` | `grenade_ctor` `0x0048af30` | `structs/grenade.md` |
| `0xf` | `C_smoke_trail` | `smoke_trail_ctor` `0x004f60e0` | `structs/smoke_trail.md` |
| `0x10` | `C_cartridge` | `cartridge_ctor` `0x00476f80` | `structs/cartridge.md` |

A census of `bottle_ctor`'s callers finds exactly these three plus the two casing pools. The
subclasses differ from the base in the fields their `GameInit` and constructor rewrite: all three
clear `end_sets_act_state`; `cartridge_ctor` and `smoke_trail_ctor` set `coll_registered`
and `may_broadcast` to 0, from one `xor al, al`; `cartridge_game_init` sets `ray_all_layers` and `frame_axis`.

**All three share ONE `Init` implementation**, `bottle_subclass_init_require_frame`
(`0x0048af90`), and that is a measurement: a scan of `.rdata` for the address as a pointer returns
exactly three datarefs — `0x0063be18` slot 18, `0x0063c3c0` slot 18, `0x0063cd80` slot 18. The
body is `frame ? bottle_init(frame) : false`, an `/OPT:ICF` merge (§9d), so the PC symbol is
named for the family: naming it `grenade_init` would assert one owner out of three
(`structs/actor_track.md` §4c). Its docblock carries two `PS2 twin:` lines, `C_grenade`'s and
`C_smoke_trail`'s `Init`, the only two PS2 bodies; `C_cartridge` has no PS2 methods.

## 7. Where the shipped instances are, and what they do

### 7a. The scene census — `C_bottle` is scene-placed, exactly twice

Method: walk all **79** shipped `scene2.bin` (`0x4c53` root → `0xAE20` special-object block →
`0xAE21` object → `0xAE22` type dword, which is the value `mission_create_actor` switches on).

```
special type 0x0a (C_bottle):  2 occurrences, 1 scene
   MISE18-SALIERY   xplechovka3    0xAE24 payload EMPTY (6-byte header only)
   MISE18-SALIERY   xplechovka4    0xAE24 payload EMPTY (6-byte header only)
```

* **`plechovka` means *tin can*.** The models are `xplechovka3.i3d` / `xplechovka4.i3d`. The
  user's independent reading of the mesh — *"it looks like e.g. a paint tin can, not much like a
  bottle"* — agrees with the asset name. The class name is the developers' own symbol
  (`__ct__8C_bottleFv`) and stays; the **content** is cans.
* The empty `0xAE24` chunk independently confirms §3c: a bottle carries no definition-record
  state (its `SaveData` / `LoadData` slots are stubs).
* `bottle_init` accepts any frame `actor_init` accepts, so both objects really do become live
  actors.
* **`MISE18-SALIERY` is textdb string 5490, "18-01 Just For Relaxation — Salieri Bar".**

**The sharpest control is inside the census itself.** Seventeen versions of the Salieri's bar
scene ship. `xplechovka3` and `xplechovka4` exist in all of them and are **plain static scene
meshes in sixteen** — only in `MISE18-SALIERY` are those two objects promoted to special type
`0x0a`. Somebody deliberately made two cans in the bar dynamic for one mission's opening.

A per-scene census is a **lower bound** on usage and can never prove absence — ten
certainly-live kinds appear in no scene either, because C++ code spawns them. Here it is used in
the direction it is valid in: to prove a **presence**. And `C_bottle` is alive a second way
regardless, as the base of three runtime classes (§6).

### 7b. Two things the class cannot do

| Claim | Measurement |
|---|---|
| picked up | **No.** No `S_using_object` in the layout (`C_door` has one at `+0x180`); no use or pick-up handler among the class's functions; zero `using_object` / inventory / action-menu calls in the whole unit; and **zero `GetKind == 0xa` tests in the entire image** — all 346 `E8` callsites of `folded_get_dword_0x10` were decoded and the kind gates that exist are `0x1b`×108, `4`×84, `2`×80, `5`×10, `6`×8, `0x16`×4, `8`×3, … Nothing anywhere singles out a bottle by kind |
| thrown | **Not as a `C_bottle`.** All four callers of `bottle_set_flight_impulse` spawn a subclass: `game_new_grenade` (`C_grenade`, kind `0xe` — the molotov), `game_new_cartridge` ×2 (`C_cartridge`, `0x10`), `game_new_smoke_trail` (`C_smoke_trail`, `0xf`). No caller launches a kind-`0xa` instance |

A shipped can can therefore only be moved by **being shot** (`bottle_hit`: hit-type gate
`{0, 1, 2}`, the incoming vector scaled × 15.0 with `abs(y)`, the tumble re-rolled from
3 × `rand_float`, then `actor_set_act_state(0)` to start the arc) or by **being walked into**
(`bottle_on_collision`: the *other* actor's kind gated on `{2 C_player, 10 C_bottle,
0x1b C_entity}` — a **car, kind 4, is not in that gate**, so you cannot drive one over).

### 7c. In-game observation — the behaviour is not distinguishable from a static mesh

The player-tester, testing `MISE18-SALIERY` directly:

> *"`xplechovka3`/`xplechovka4` and `xplechovka`/`xplechovka2` are behaving the same. But this
> behavior is very limited — the same sound is emitted when hit with a gun (metal cing) … From
> an in-game perspective there is no difference between `xplechovka` and `xplechovka3`."*

**One half of that is fully explained by the static reading.** `xplechovka` and `xplechovka2`
are **not special objects in that scene at all** — the census returns only `xplechovka3` and
`xplechovka4` for type `0x0a`, and the scene's whole type histogram is
`{0x02, 0x04, 0x05, 0x06, 0x0a×2, 0x0c, 0x12, 0x1b, 0x23}`. So two of the four cans the user
shot are plain meshes. And **`C_bottle` never plays a sound**: there is not one sound call in any
of its functions. The metal *cing* is the shared bullet-impact material sound, which a static mesh
produces too — identical audio is exactly what the code predicts, for all four.

**The other half is not:** `bottle_hit` writes the flight band and sets act state 0, so a shot can
should visibly move, and the tester did not observe it move (§11).

⇒ **`C_bottle` is used, but its shipped presence is two tin cans whose extra behaviour no player
is likely to notice.** That is a different statement from `seagull_flock` and `bendable_plant`,
which have **zero** scene instances and no located construction path at all.

## 8. The translation unit

`[0x00453a20, 0x00456460)`: it opens with its CRT static-init pair
(`crt_static_init_guard_00453a20` / `crt_atexit_register_00453a40`), then `chunk_read`
(`0x00453a50`), the `C_chunk` field reader that belongs to `structs/chunk.md` and leads this unit
only because MSVC emitted that out-of-line inline here (212 callsites in 20 functions), then the
nineteen `bottle_*` functions and `vec3_div_scalar` (§3d). The next unit's guard,
`crt_static_init_guard_00456460`, closes it; that unit is the PC-only `C_box`
(`structs/box.md`). The previous unit is `C_airplane` (`structs/airplane.md` §6).
`bottle_subclass_init_require_frame` lies outside it, in `C_grenade`'s unit.

## 9. PC vs PS2

**The class is a straight port apart from networking.** Every gameplay constant matches: the
flight cut-off 50.0, the hit scale 15.0, the kind gate `{2, 10, 0x1b}`, the hit-type gate
`{0, 1, 2}`, and both I3D frame-flag masks.

**The field rebase is not a constant delta** — PC `+0x24` ↔ PS2 `+0x00` but PC `+0x12c` ↔
PS2 `+0x178` — because the `C_actor` base is larger on the PS2 *and* its `S_vector` is a padded
vec4. **Map by rank, never by subtracting.** The 22-store savegame pair gives the ranks
directly:

| PC | PS2 | What |
|---|---|---|
| +0x00 (vptr) | +0x78 | MSVC puts the vptr first, GCC 2.x after the data |
| +0x10 | +0x50 | kind |
| +0x24 | +0x00 | position |
| +0x30 | +0x10 | frame direction |
| +0x68 | +0x70 | I3D frame |
| +0x70 | +0x80 | flight direction |
| +0x7c | +0xe0 | flight speed |
| +0x80 | +0xe4 | fall accumulator |
| +0x84 | +0xe8 | `end_sets_act_state` |
| +0x8c | +0x90 | contact normal |
| +0xa4 | +0xf0 | radius |
| +0xa8 | +0xb0 | frame axis |
| +0xb4 | +0x100 | dynamic collision proxy |
| +0xfc | +0x160 | proxy-registered flag |
| +0x100 / +0x104 / +0x12c | +0x164 / +0x168 / +0x178 | bounce count, on-ground flag, ground material |

The savegame record is **0x53 bytes on the PC and 0x58 on the PS2**: the PC packs the payload
byte-wise (store offsets 1, 5, 9, …) while the PS2 writes dword-aligned slots.

### 9a. The PS2 port emptied the per-class network replication bodies

The PS2 build has no networking. Its ELF names eight IOP modules (`SIO2MAN`, `PADMAN`, `LIBSD`,
`SOUND`, `SOUND2`, `SDRDRV`, `MCMAN`, `MCSERV`) and none is a network module; `GM_NetPlayer` is a
`GM_Component` GUI widget, not a network class (`structs/net_event_msg.md` §10). What the PS2 keeps
is the `NetDirect` slot of the shared class interface, with **empty bodies**:

| PS2 `NetDirect` size | Classes |
|---|---|
| **8 bytes (empty)** | **19** — `C_actor`, `C_actor_shot`, `C_airplane`, **`C_bottle`**, `C_bridge`, `C_crash_obj`, `C_detector`, `C_drop_in_weapon`, `C_entity`, `C_grenade`, `C_human`, `C_irenka`, `C_PhThing`, `C_public_physics`, `C_railway`, `C_smoke_trail`, `C_traffic_car`, `C_traffic_generator`, `C_wagon` |
| 16 bytes | `C_player` |
| 68 bytes | `C_door` |

So `C_bottle`'s PC vtable slot 15 holds **2112 bytes** of dead reckoning while
`NetDirect__8C_bottleFUi` is an 8-byte stub, and the PC's message receiver, its two senders and
the two event messages of §5b have **no PS2 symbol at all**. The same shape holds for `C_PhThing`,
whose slot 15 holds a **1696-byte** dead-reckoning body (`physical_object_net_apply_remote_state`)
against the same 8-byte PS2 stub, plus five further PC-only replication members
(`structs/physical_object.md` §6), and for `C_box`.

⇒ **For this subsystem a PS2 method count is a lower bound, not the usual upper bound.** Both
directions exist — elsewhere the PS2 build *added* subsystems — so check which one you are in
before writing a terminal tag.

⇒ It is also why `bottle_net_apply_remote_state` is graded **STRUCTURAL and not CONFIRMED**. The
twin *symbol* is certain — vtable slot 15 is `NetDirect(Ui)` family-wide, byte-proven on
`C_airplane`, whose slot 15 holds `stub_noop_args4_00461180` (`c2 04 00` = `ret 4`, exactly one
4-byte argument, present in 30 vtables) — but an empty body cannot be body-diffed, and "the symbol
exists" is not the same evidence as "the bodies agree".

### 9b. The three PS2 methods with no distinct PC function

All three fold onto shared PC stubs (§3c):

| PS2 method | Size | PC vtable slot | Folded onto |
|---|---|---|---|
| `DuplicateFrom__8C_bottleFP7C_actor` | 8 | 21 | `thunk_stub_noop_args4_00461180` (`0x004773f0`), a 5-byte `jmp` thunk |
| `SaveData__8C_bottleFP7C_chunk` | 8 | 19 | `stub_rettrue_args4_00477710` |
| `LoadData__8C_bottleFP7C_chunk` | 8 | 20 | **the same** `stub_rettrue_args4_00477710` |

Two adjacent vtable slots sharing one stub is the documented ICF fold (`structs/airplane.md` §7).

### 9c. Corollary twin this class produced

| PC | PS2 | How |
|---|---|---|
| `chunk_read` | `Read__7C_chunkFPvi` | its own diff — the `max(0, remaining) < size` idiom |

### 9d. The subclass set on the PS2

`__ct__9C_grenadeFv` and `__ct__13C_smoke_trailFv` both chain `__ct__8C_bottleFv`, which
corroborates the §6 subclass shape independently. The shared slot-18 `Init` of §6 is two
separate 44-byte bodies on the PS2 (`C_cartridge` has none), identical to one another — which is why `/OPT:ICF` merged them
into `bottle_subclass_init_require_frame` on the PC.

## 10. Related

- `structs/actor.md` — the base class; `structs/actor_track.md` — vtable slots 0–12.
- `structs/grenade.md`, `structs/smoke_trail.md`, `structs/cartridge.md` — the three subclasses.
- `structs/airplane.md` — the preceding class in the link order; `structs/box.md` — the next one.
- `structs/entity_kind.md` — the kind enum.
- `structs/net_event_msg.md`, `structs/net_session.md` — the replication layer.

## 11. Open items

1. **In game: do the two dynamic cans move when shot?** Shoot the four cans in `MISE18-SALIERY`
   one at a time and watch only for movement: `xplechovka3` / `xplechovka4` are the dynamic pair,
   `xplechovka`, `xplechovka2`, `xplechovka5` and `xplechovka6` are static. Candidates if they do
   not move: the launch is too small to read (the × 15 scale acts on an unmeasured hit vector); the
   dynamic pair was not the pair shot; the `Hit` never reaches them; or a degenerate bound volume
   leaves the collision proxy unhittable while `radius` looks correct.
