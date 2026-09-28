# `C_mission` — the mission singleton

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-09-24 13:55 +0200

The object every `g_act_mission` dereference in the decomp lands on. Its translation unit
`Mission.cpp` = `[0x005fe4f0, 0x00608390)`; `[0x005fe550, 0x00607f70]` holds **84 function
starts, all named**.

**Coverage: 35 of the PS2's 68 `C_mission` methods are located**, plus all four free
functions of the PS2 band. The 33 that are not are enumerated in §5d, and 26 of them belong
to `InterCity_*`, a PS2-port-only subsystem, so 68 is an upper bound rather than a target.

Everything outside §5 is a PC fact; §5 is the one section that carries PS2 material.

## 1. The object, and the pointer that names it

| symbol | address | what it is |
|---|---|---|
| `g_mission` | `0x006f9440` | **the static object** — a `Main.cpp` file-scope variable |
| `g_act_mission` | `0x0065115c` | **a pointer to it**, statically initialised |

`.data:0x0065115c` holds the literal `0x006f9440` — byte-read from `Game.exe`, not inferred.
That is what ties the initializer stub in `Main.cpp` to the 1500-odd `g_act_mission`
dereferences elsewhere in the decomp.

`g_mission` has **zero textual existence** in the decomp: every access goes through the
pointer or through ECX, so a search for `0x006f9440` in the decomp returns nothing. Its
name can only ever be a registry row, exactly as for `g_input`.

## 2. `C_mission + 0x90` — the graphics/quality settings sub-object

A `0x1c`-byte block of six tunables. It has its own accessor family, and the family's
receiver is byte-proven: an ECX-tracking census of every callsite of the seven accessors
finds **38 callsites, 0 unsafe**, and every single receiver is `C_mission + 0x90`
(`mov ecx,[0x65115c]; add ecx,0x90`, or `lea ecx,[reg+0x90]` where `reg` already holds the
mission pointer).

```c
typedef struct {                        /* at C_mission + 0x90, sizeof = 0x1c */
/* +0x00 */ float details_scale;        // [H] clamped 0.4 .. 2.0
/* +0x04 */ float draw_distance_scale;  // [H] clamped 0.3 .. 1.0
/* +0x08 */ float view_distance;        // [H] world units, default 250.0
/* +0x0c */ int   effects_quality;      // [H] 0..2  — the weather detail level
/* +0x10 */ int   sounds_quality;       // [H] 0..2
/* +0x14 */ int   shadow_quality;       // [H] index 0..5 -> {0,1,3,6,9,15}
/* +0x18 */ int   particles_quality;    // [H] 0..2  — the hydrant detail level
} S_mission_settings;
```

| mission offset | sub-object offset | setter | getter | range |
|---|---|---|---|---|
| `+0x90` | `+0x00` | `mission_settings_set_details_scale` | - | 0.4 … 2.0 |
| `+0x94` | `+0x04` | `mission_settings_set_draw_distance_scale` | - | 0.3 … 1.0 |
| `+0x98` | `+0x08` | - | - | default 250.0 |
| `+0x9c` | `+0x0c` | `mission_settings_set_effects_quality` | - | 0 … 2 |
| `+0xa0` | `+0x10` | `mission_settings_set_sounds_quality` | - | 0 … 2 |
| `+0xa4` | `+0x14` | `mission_settings_set_shadow_quality` | `mission_settings_get_shadow_quality_idx` | idx 0 … 5 |
| `+0xa8` | `+0x18` | `mission_settings_set_particles_quality` | - | 0 … 2 |

`+0x98` has no accessor: `mission_open_bin` writes `0x437a0000` (250.0f) into it twice and
`mission_update_settings` hands it to the scene's `+0x17c` object together with `0.05f`.

Two of the six were decoded independently of the accessor family and agree exactly: `+0x9c`
is the **weather detail** level (it scales the weather particle count by `/0x14`, `>>3`, or
unscaled) and `+0xa8` is the **hydrant detail** level (broken hydrants spray only at level 2
or above).

The block is mirrored into `g_ls3d_setup + 0x27 … +0x3b` on the way out to the registry and
restored from it on the way in; that mirror is documented in `structs/app_bootstrap.md`.

## 3. Other `C_mission` fields already known elsewhere

Recorded here only so the offsets are in one place; each is documented where it was found.

| offset | what | where |
|---|---|---|
| `+0x08` | cleared by `mission_open` before the teardown | §7 |
| `+0x0c` | **the "a mission is open" flag** — `mission_open` sets it, `mission_close` gates on it and clears it | §4 |
| `+0x10` | the `I3D_scene` | `structs/i3d_frame.md`, `structs/program.md` |
| `+0x18 … +0x20` | `vector<S_anim_model>`, stride 8 — `{I3D_model *, bool per_game}` | §3a |
| `+0x20` | the actor vector's owner | `structs/entity.md` |
| `+0x24` | the `C_game` singleton | `structs/game.md` |
| `+0x28` | **`C_tshift`** — the UV-scroll animator | `structs/uv_scroll.md` |
| `+0x2c` | the `C_web_path` navigation graph | `structs/web_path.md` |
| `+0x30` | `C_roads` | `structs/roads.md` |
| `+0x38 … +0x40` | **the master actor vector** (dwords) | §3a |
| `+0x48 … +0x50` | **the TRANSIENT actor vector** — the set `game_done` destroys | §3a |
| `+0x58 … +0x60` | the global-script vector (`C_program *`) | §3a |
| `+0x68` | the mission-name buffer, `operator new[]`-owned | §4 |
| `+0x6c` | **the particle manager** — NOT `C_game + 0x6c` | `structs/particle.md` §2 |
| `+0x70` / `+0x74` | **the PENDING / CURRENT mission number** — the superglobal mission number | §3d |
| `+0x78` / `+0x7c` | the sound-restore stage and its latch, consumed by `mission_tick` | §3d |
| `+0x84 … +0x8c` | **the pending fade-in list**, stride 8 — `{I3D_objects *, s16 elapsed, s16 duration}` | §3b |
| `+0x90 … +0xa8` | the settings sub-object | §2 |
| `+0xb0 … +0xb8` | **the frame/actor carry cache**, stride `0x10` | §3c |
| `+0xc0 … +0xc8` | **the active-actor carry vector** (dwords) | §3c |
| `+0xd0 … +0xd8` | **the actor-sound carry cache**, stride `0xc` | §3c |

### 3a. The three lifecycle vectors

`mission_add_anim_model` / `_del_anim_model` / `_clear_game_anim_models` own `+0x18`; the
flag byte in each record marks a model registered for the lifetime of the GAME, and
`mission_clear_game_anim_models` purges exactly those with a swap-with-last pass.

`mission_add_actor(actor, transient)` appends to `+0x38` always and to `+0x48` when the
flag is set. `mission_del_not_me_actors` — whose sole caller is `game_done` — destroys
everything in `+0x48`. So **`+0x48` is the set that does NOT survive a scene change**, and
`mission_is_persistent_actor` is the predicate for the complement. Callers that pass 1:
`loadsave_savegame_load`, `racing_spawn_winners_scene`. Callers that pass 0:
`engine_new_mission` (the carried-over player and car) and
`loadsave_freeride_setup_scene`.

### 3b. The pending fade-in list at `+0x84`

`mission_fade_in_object(obj, duration_ms)` takes a reference, sets the object's alpha to
**0** and appends `{obj, elapsed = 0, duration}`. `mission_tick` ramps the alpha to
`elapsed / duration` each frame and, on reaching the duration, forces 1.0, releases and
swap-pops. `mission_finish_object_fade_ins` forces all of them at once.

**Its sole producer is the city streamer.** `cache_base_block_walk_block_objects` calls
it with the literal **700** for every frame of a newly activated block whose visual type
is 1 and whose tag is `'OBJ_'` — so this is the pop-in hider: streamed-in geometry fades
up over 700 ms instead of appearing instantly. The `0x52` floor in the setter rejects a
duration under 82 ms.

### 3c. The three scene-change carry caches

Between `mission_close` and the next `mission_open`, actors and models that must survive
are parked here. `mission_carry_caches_clear` empties all three, and `mission_close` calls
it.

| vector | stride | record | store | take |
|---|---|---|---|---|
| `+0xb0` | `0x10` | `{C_actor *, I3D_frame *, char *name, bool own_model}` | `mission_actor_cache_store`, `mission_model_cache_store` | `mission_actor_cache_take`, `mission_model_cache_take` |
| `+0xc0` | 4 | `C_actor *` | `mission_active_actor_store` | `mission_active_actor_take`, `mission_active_actor_drop` |
| `+0xd0` | `0xc` | `{C_actor *, int id, I3D_sound *}` | `mission_actor_sound_cache_store` | `mission_actor_sound_cache_take` |

The `+0xb0` record holds EITHER an actor (slot 0) or a bare model (slot 1), which is what
the two `take` functions discriminate on; the name in slot 2 is the model file's basename,
split out of `I3D_frame + 0x154` with `strrchr('\\')`.

### 3d. The mission number and the sound-restore stage

`+0x70` / `+0x74` are the **pending** and **current** mission number, and the pair is
superglobal: `mission_set_next_mission_number` fills `+0x70`,
`mission_commit_next_mission_number` moves it into `+0x74` at `change_mission`, and
`mission_get_mission_number` reads `+0x74`. The savegame chunk hook
`game_savegame_load_before` reads the same pair, which is the same field under a second
reading, not a second field.

`+0x78` is a monotonic STAGE, not a countdown: `recloadfull` (a cutscene started) raises it
to 1 and `recunload` (it ended) raises it to 2, through `mission_set_sound_restore_stage` /
`mission_get_sound_restore_stage`; `+0x7c` latches it and `mission_tick` consumes the pair.
Both commands belong to `C_recorder`, which makes this a PC-only path.

## 4. PC functions

**Scaffolding and accessors**

| PC VA | PC name | role |
|---|---|---|
| `0x005f7cd0` | `mission_static_init` | `.CRT$XCU` entry (in `Main.cpp`) |
| `0x005f7ce0` | `mission_static_ctor` | `mov ecx, 0x6f9440; jmp mission_ctor` |
| `0x005f7cf0` | `mission_register_atexit_dtor` | `atexit(&mission_static_dtor)` |
| `0x005f7d00` | `mission_static_dtor` | `mov ecx, 0x6f9440; jmp mission_dtor` |
| `0x00425390` | `act_mission_get_game` | returns `+0x24` |
| `0x005c8660` | `mission_set_next_mission_number` | writes `+0x70` |
| `0x005c8670` | `mission_get_mission_number` | reads `+0x74` |
| `0x005c8680` | `mission_commit_next_mission_number` | `+0x70` → `+0x74` |
| `0x005c8690` | `mission_set_sound_restore_stage` | writes `+0x78` |
| `0x005c86a0` | `mission_get_sound_restore_stage` | reads `+0x78` |
| `0x00606390` | `mission_get_uv_scroll` | returns `+0x28` |
| `0x005f8700` | `engine_new_mission` | the mission-change driver (free function, `Main.cpp`) |

**Lifecycle**

| PC VA | PC name | role |
|---|---|---|
| `0x005fe5b0` | `mission_ctor` | the constructor |
| `0x005fe6b0` | `mission_dtor` | the destructor |
| `0x005fe7d0` | `mission_init` | build the engine sub-objects and the `C_game` |
| `0x005fe950` | `mission_uninit` | tear them down |
| `0x005fff30` | `mission_open` | open a mission by name |
| `0x005ffb40` | `mission_close` | close whatever is open |
| `0x006005a0` | `mission_open_bin` | the scene `.bin` reader (16 KB) |
| `0x005ffd40` | `mission_tick` | one frame |

**Actors**

| PC VA | PC name | role |
|---|---|---|
| `0x005fed70` | `mission_create_actor` | the kind → `operator_new` → ctor factory |
| `0x005ff400` | `mission_add_actor` | register, optionally as transient |
| `0x005ff5e0` | `mission_unload_actor` | unregister without destroying |
| `0x005ff690` | `mission_del_actor` | unregister and destroy |
| `0x005ff7a0` | `mission_del_actors` | destroy all |
| `0x005ff860` | `mission_del_not_me_actors` | destroy the transient set |
| `0x005ff980` | `mission_is_persistent_actor` | survives `game_done`? (PC-only) |
| `0x005ff9f0` | `mission_find_actor_by_name` | by `I3D_frame` name |

**Animated models, fades and settings**

| PC VA | PC name | role |
|---|---|---|
| `0x005feac0` | `mission_add_anim_model` | register an animatable model |
| `0x005fecb0` | `mission_del_anim_model` | drop one |
| `0x005fed10` | `mission_clear_game_anim_models` | purge the per-game ones |
| `0x00604ce0` | `mission_fade_in_object` | start a timed fade-in |
| `0x00604ef0` | `mission_finish_object_fade_ins` | force them all to completion |
| `0x00604f70` | `mission_update_settings` | push the quality settings into the engine |
| `0x00605010` | `mission_restore_settings` | put the engine back on unscaled settings |
| `0x00605060` … `0x006051f0` | the seven `mission_settings_*` accessors | §2 |

**Global scripts and the carry caches**

| PC VA | PC name | role |
|---|---|---|
| `0x005ffa80` | `mission_global_program_game_init` | `Init` every global script |
| `0x005ffab0` | `mission_global_program_game_done` | `Done` every global script |
| `0x005ffae0` | `mission_global_program_run` | invoke a named subroutine |
| `0x00605340` | `mission_actor_cache_store` | park an actor across a scene change |
| `0x006058e0` | `mission_actor_cache_take` | reclaim one by name and kind |
| `0x00605670` | `mission_model_cache_store` | park a bare model |
| `0x00605a30` | `mission_model_cache_take` | reclaim one by name |
| `0x00605b30` | `mission_active_actor_store` | park a live actor |
| `0x00605dd0` | `mission_active_actor_take` | revive and unpark |
| `0x00605d40` | `mission_active_actor_drop` | discard without reviving |
| `0x00605e90` | `mission_actor_sound_cache_store` | park an actor's sound |
| `0x00606100` | `mission_actor_sound_cache_take` | reclaim it |
| `0x006061d0` | `mission_carry_caches_clear` | empty all three |
| `0x00605220` | `mission_get_scene_bounds` | world bbox (PC-only; multiplayer) |

**Free helpers hosted by the same TU** — `scene_fill_sector_bits`,
`scene_accumulate_frame_bounds`, `scene_register_frame_by_name_cb`,
`scene_load_report_progress`, `scene_load_progress_callback`,
`car_clear_all_seat_owners`, `game_ctor`, `game_dtor`, plus the `frame_map_*`,
`uv_scroll_*`, `roads_*`, `chunk_*`, `i3d_cache_items_*` and `string_*` helpers. Full
list: `Game-ghidra.exe.functions.md`, the `Mission.cpp` section and its three siblings.

`mission_open_bin` also drives **13 callsites** of `vec_index_addr_stride_0x8`
(`0x006065d0`) against two stack-local vectors of `{I3D_frame *, int}` deferred frame-fixup
pairs. That address is a generic `std::vector<8-byte>::operator[]` with 377 callsites
image-wide; the script VM indexes `{opcode, args}` through the same address, so the sharing
is an ICF fold of two different element types (`structs/program.md` §6).

## 5. PC vs PS2

PS2 class `C_mission`; the static object is at `0x00509b10`, constructed by
`__sinit_Main.cpp` in the same lockstep slot as the PC's.

**Twin table** (PC VA order, PC name first):

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| `0x005fe5b0` | `mission_ctor` | `__ct__9C_missionFv` | CONFIRMED |
| `0x005fe6b0` | `mission_dtor` | `__dt__9C_missionFv` | CONFIRMED |
| `0x005fe7d0` | `mission_init` | `Init__9C_missionFP10I3D_driver` | CONFIRMED |
| `0x005fe950` | `mission_uninit` | `UnInit__9C_missionFv` | CONFIRMED |
| `0x005feac0` | `mission_add_anim_model` | `AddAnimModel__9C_missionFP9I3D_modelb` | CONFIRMED |
| `0x005fecb0` | `mission_del_anim_model` | `DelAnimModel__9C_missionFP9I3D_model` | CONFIRMED |
| `0x005fed10` | `mission_clear_game_anim_models` | `ClearGameAnimModels__9C_missionFv` | CONFIRMED |
| `0x005fed70` | `mission_create_actor` | `CreateActor__9C_missionFi` | CONFIRMED |
| `0x005ff400` | `mission_add_actor` | `AddActor__9C_missionFP7C_actorb` | CONFIRMED |
| `0x005ff5e0` | `mission_unload_actor` | `UnloadActor__9C_missionFP7C_actor` | CONFIRMED |
| `0x005ff690` | `mission_del_actor` | `DelActor__9C_missionFP7C_actor` | CONFIRMED |
| `0x005ff7a0` | `mission_del_actors` | `DelActors__9C_missionFv` | CONFIRMED |
| `0x005ff860` | `mission_del_not_me_actors` | `DelNotMeActors__9C_missionFv` | CONFIRMED |
| `0x005ff980` | `mission_is_persistent_actor` | `-` | PC-only |
| `0x005ff9f0` | `mission_find_actor_by_name` | `FindActorByName__9C_missionFPCc` | CONFIRMED |
| `0x005ffa80` | `mission_global_program_game_init` | `GlobalProgramGameInit__9C_missionFv` | CONFIRMED |
| `0x005ffab0` | `mission_global_program_game_done` | `GlobalProgramGameDone__9C_missionFv` | CONFIRMED |
| `0x005ffae0` | `mission_global_program_run` | `GlobalProgramRun__9C_missionFPc` | CONFIRMED |
| `0x005ffb40` | `mission_close` | `Close__9C_missionFv` | CONFIRMED |
| `0x005ffd40` | `mission_tick` | `Tick__9C_missionFi` | CONFIRMED |
| `0x005ffeb0` | `scene_load_report_progress` | `ShowProgress__FP7C_chunk` | CONFIRMED (free fn) |
| `0x005fff30` | `mission_open` | `Open__9C_missionFPCcbUib` | CONFIRMED |
| `0x00600540` | `scene_load_progress_callback` | `__SceneErrFnc__F21LS3D_CALLBACK_MESSAGEUiUiPv` | CONFIRMED (free fn) |
| `0x00600560` | `scene_fill_sector_bits` | `cbSectorBitFill__Q29C_mission21S_hlp$4890Mission_cpp…` | CONFIRMED (file static) |
| `0x006005a0` | `mission_open_bin` | `OpenBin__9C_missionFPCcUi` | CONFIRMED |
| `0x00604ce0` | `mission_fade_in_object` | `AddTimeTransparentObject__9C_missionFP10I3D_objects` | CONFIRMED |
| `0x00604ef0` | `mission_finish_object_fade_ins` | `FinishTimeTransparentObjects__9C_missionFv` | CONFIRMED |
| `0x00604f70` | `mission_update_settings` | `UpdateSettings__9C_missionFv` | CONFIRMED |
| `0x00605010` | `mission_restore_settings` | `RestoreSettings__9C_missionFv` | CONFIRMED |
| `0x00605060` … `0x006051f0` | the seven `mission_settings_*` | `-` | PC-only |
| `0x00605220` | `mission_get_scene_bounds` | `-` | PC-only |
| `0x00605340` | `mission_actor_cache_store` | `FrmActBuff_StoreActor__9C_missionFP7C_actorb` | CONFIRMED |
| `0x00605630` | `car_clear_all_seat_owners` | `ClearCarSeats__FP5C_car` | CONFIRMED (free fn) |
| `0x00605670` | `mission_model_cache_store` | `FrmActBuff_StoreModel__9C_missionFP9I3D_model` | CONFIRMED |
| `0x006058e0` | `mission_actor_cache_take` | `FrmActBuff_GetActor__9C_missionFPci` | CONFIRMED |
| `0x00605a30` | `mission_model_cache_take` | `FrmActBuff_GetModel__9C_missionFPc` | CONFIRMED |
| `0x00605b30` | `mission_active_actor_store` | `FrmActBuff_StoreActiveActor__9C_missionFP7C_actor` | CONFIRMED |
| `0x00605d40` | `mission_active_actor_drop` | `FrmActBuff_RemoveActiveActor__9C_missionFP7C_actor` | CONFIRMED |
| `0x00605dd0` | `mission_active_actor_take` | `FrmActBuff_IsActiveActor__9C_missionFP7C_actor` ¹ | CONFIRMED |
| `0x00605e90` | `mission_actor_sound_cache_store` | `FrmActBuff_StoreActorSound__9C_missionFP7C_actoriP9I3D_sound` | CONFIRMED |
| `0x00606100` | `mission_actor_sound_cache_take` | `FrmActBuff_GetActorSound__9C_missionFP7C_actori` | CONFIRMED |
| `0x006061d0` | `mission_carry_caches_clear` | `FrmActBuff_ClearAll__9C_missionFv` | CONFIRMED |

¹ The PS2 verb is inaccurate on both builds: neither body is a predicate — both reset a car,
call `Rebirth` and REMOVE the entry. The PC name states what the body does.

### 5a. What the ctor/dtor diff established

**Constructor.** Both zero the leading dword, construct ~20 member vectors, and write the
six settings defaults. The PC writes `1.0f, 1.0f, 2, 2, 3, 2` at
`+0x90/+0x94/+0x9c/+0xa0/+0xa4/+0xa8`; the PS2 writes the same six values at
`+0xcbc/+0xcc0/+0xcc8/+0xccc/+0xcd0/+0xcd4`. That reproduces the `+0x9c ↔ +0xcc8` and
`+0xa8 ↔ +0xcd4` pairs the weather/hydrant detail evidence produces from a completely
different argument — an independent confirmation of both.

**Destructor.** Both call an un-init helper first and then free every member vector in
**descending offset order** — PC `0xd0, 0xc0, 0xb0, 0x84, 0x58, 0x48, 0x38, …`;
PS2 `0xd50, 0xd38, 0xd08, 0xcfc, 0xcf0, 0xce4, 0xcd8, 0xcb0, … 0x4c`.

### 5b. The constant offset delta is the PS2's embedded particle manager

The two field maps differ by a constant `0xc2c` in the tail, and that number has a cause:
**PS2 `C_mission` embeds a `C_ParticleManager` at `+0x64`**, spanning to about `+0xcb0`
(`__ct__17C_ParticleManagerFv((int)param_1 + 100)` in the constructor,
`__dt__17C_ParticleManagerFv(iVar2 + 100, …)` in the destructor). The PC has no such member
— its particle manager lives at `C_game + 0x6c` (`structs/particle.md`). The delta is
`0xcb0 − 0x64 = 0xc4c`, which is why every PS2 offset above the manager sits roughly
`0xc2c` higher than its PC counterpart.

Do **not** rebase a PS2 `C_mission` offset onto the PC by a constant. Below `+0x64` the two
maps agree; above it they do not.

### 5d. The 33 PS2 methods with no PC counterpart

An inventory of what is left, not a closed verdict on any single method.

| group | count | why it is very likely PS2-only |
|---|---|---|
| `InterCity_*` (Prebuffering, Init, Done, GameInit, GameDone, GameDoneInGameDone, ActorsToMission, ActorsFromMission, ChangeMission, SetMissionName, RepairDirName, AddGlobalActor, DelGlobalActor, PicaDelActors, SaveLoadInit, SaveLoadDone, RepairActorActState, AddTempAct, DelTempAct, TickTempAct) + `IsInterCityGlobalActor` | 21 | the confirmed city-part streaming subsystem; `InterCity` appears 127× on PS2 and **0×** on PC |
| `LoadFQLPath` / `UnloadFQLPath` / `GetFQLPath` / `OpenFQLFile` / `CloseFQLFile` | 5 | the FQL path/index family; `mission_open` calls nothing at the rank where PS2 `Open` calls `OpenFQLFile` |
| `LoadAnimationPath` / `UnloadAnimationPath` / `GetAnimationPath` | 3 | the same family, animation half |
| `Garage_AddCar` / `Garage_DelCar` / `Freeride_ChangeCar` | 3 | the PC has a `garage_*` family in its own TU; not searched from here |
| `ClearFramesRefBuff` | 1 | a CONTAINER difference, not an absence — §5e |

Two PS2 **free** functions in the band are also PS2-only on the same argument:
`ReleaseUniqueTextures__Fv` (its state is `inter_city_texture`) and `DelPicaModelDel__Fv`.
`MC_DeleteData` / `MC_LoadData` / `MC_Format`, immediately before the constructor, are the
memory-card layer.

Three PS2 free functions in the band are plausible PC inlinings but are **not** proven:
`ReadVector3__FR7C_chunk`, `ReadQuatOld__FR7C_chunk` and
`ReadBBoxOld__FR7C_chunkR8I3D_bbox`. The PC reads the same records with inline
`chunk_read(buf, N)` calls, and no distinctive constant survives to prove the match.

### 5e. The frames-ref buffer is a CONTAINER difference

The PS2 keeps a `C_mission` member `vector<I3D_frame *>` and clears it with
`ClearFramesRefBuff__9C_missionFv`. The PC keeps **no such member**: `mission_open_bin`
builds a **stack-local `std::map<C_str, I3D_frame *>`**, indexes every scene frame by name
into it while reading the file, and destroys it before returning. That map is the entire
`frame_map_*` family — 18 functions, whose single-owner census is what earns them a family
name rather than a generic red-black-tree one
(`Game-ghidra.exe.functions.md`, the frame-name-map section). A banner marks the absent
rank inside `mission_close`.

### 5f. The new-mission transition needs two PS2 functions the PC does not

At the rank between the teardown and the reopen (§6b), PS2 `InternMissionInit__Fv` calls
`InternMission_Vykur__Fv`; `C_game::Init` later calls `InternMission_Spolkni__Fv`. Both
are **PS2-port-only**, measured over a closed set:

* their private API `C_car::DamageDeformGetSize` / `DamageDeformSave` / `DamageDeformLoad`
  has **exactly three callsites in the whole PS2 image, and all three are this pair**;
* the PC has no `C_car`-level deform trio at all, and the complete caller set of the
  `C_Vehicle` ones is `car_save_game_get_size` / `_save` / `_load` plus `vehicle_settle` —
  none in the mission-transition path;
* `Vykur` calls `SchovejPero__7C_human` and `Spolkni` calls `VystrcPero__7C_human`, the
  PS2-port-only model teardown/rebuild pair.

**The cause is the model cache.** `Spolkni` asks the driver for a fresh `I3D_model`,
evicts the stale cache entry and re-opens by the saved name before restoring the
deformation — because the PS2 flushes its model cache between city parts. The PC keeps
the object alive and re-parents its frame, so there is nothing to serialise. A banner
marking the absent rank sits in `engine_new_mission` in the decomp.

`InternMissionInit_InterCity__FR11G_Inventory` is PS2-only for the ordinary reason: one
caller, gated on the InterCity flag, every distinctive callee `InterCity_*`.

## 6. The new-mission transition

A mission change requested by the script command `newmission`
(`program_process_command` case `0x33`) is not performed on the spot. The case fills a
block of file-scope globals, raises `g_reload_pending`, and the next iteration of
`engine_game_loop` clears the flag and calls **`engine_new_mission`** (`0x005f8700`).

### 6a. The request block

The PS2 symbol table names every one of these, with sizes, and the PC layout matches
field for field.

| PC address | PC name | PS2 symbol | what it holds |
|---|---|---|---|
| `0x006f9210` | `g_new_mission_rot_dir` (`S_vector`) | `NewMission_ROT` | the arrival FORWARD vector |
| `0x006f921c` | `g_new_mission_rot_up` (`S_vector`) | *(same symbol, +0x10)* | the arrival UP vector |
| `0x006f9228` | `g_new_mission_pos` (`S_vector`) | `NewMission_POS` | the arrival position |
| `0x006f9234` | `g_new_mission_entry` | `NewMission_ENTRY` | the ENTRY FRAME name in the new scene |
| `0x006f9254` | `g_reload_pending` | `ActiveNewMission` | the one-shot request flag |
| `0x006f9258` | `g_new_mission_name` | `NewMission` | the MISSION name |
| `0x006f9570` | `g_new_mission_actors` (`pointer[2]`) | `NewMissionActors` | the 2-slot carried-actor array |
| `0x006f9578` | `g_new_mission_speed_kmh` | `NewMissionSpeed` | arrival speed, **km/h** |

The PS2 gives `NewMission_ROT` a size of 32 and `NewMissionActors` a size of 8; the PC's
`S_matrix::SetDir3(dir, up)` call and the script arm's own loop bound `< 0x6f9578` prove
the same splits independently. The speed's unit is proven by
`car_reset(value * 0.2777778, 1)` — 1/3.6.

### 6b. What `engine_new_mission` does

Two paths, chosen by whether `g_new_mission_entry` is empty:

* **empty** — a plain reload: `game_done`, `mission_close`,
  `mission_open(g_new_mission_name)`, `collision_load_level` on `missions\%s\tree.klz`,
  `game_init`.
* **set** — the same, but the player and (if he was driving) his car are **carried over**.
  Both are deactivated, the inventory block `C_human + 0x480 .. +0x560` is saved to the
  stack and cleared, the old scene is torn down, every kind-2 actor in the NEW scene is
  deleted, the two carried actors are pushed back into the new `C_game`'s actor list at
  `+0x148`, the inventory is restored, and the entry frame is resolved so the actor can be
  placed and the car reset to the requested speed.

## 7. Open items

| item | state |
|---|---|
| `[0x00608390, …)` — the tail of the TU | not enumerated. The swept range stops at `0x00607f70`; the six functions between it and the next `.CRT$XCU` initializer are the `frame_map_*` primitives, all named, but nothing beyond `0x00608390` was looked at |
| `C_mission + 0x08` | [M] — one writer and one reader inside `mission_open` / `mission_tick`, no decoded meaning |
| `I3D_frame + 0x140` | [U] — at least `0x18` bytes, loaded from the scene file for visual-type-6 frames, and no more (`structs/i3d_frame.md`) |
| the two UV channels of `C_tshift` | [M] — `structs/uv_scroll.md` §6 |
| `Tick__8C_tshiftFUi` | body not read, so `uv_scroll_tick`'s twin rests on callsite position alone and is not stamped |
| in game | streamed-in city geometry should fade up over ~0.7 s rather than popping — `mission_fade_in_object`, §3b |
| in game | a car placed by the scene `.bin` should survive a mission change while one spawned by script should not — the `C_actor + 0x66` `scene_placed` flag, §3c and `structs/actor.md` |
