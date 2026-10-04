# `C_game` — the game-world manager singleton

**Created** 2026-07-18 18:46 +0200 · **Modified** 2026-10-04 16:35 +0200

**PS2 class:** `C_game` (mangled tag `6C_game`). **PC prefix: `game_`.** It is the
world-manager singleton that owns sound, the breakage and particle effects, the actor lists
(temporary / invisible / dying / death queue), fake shadows, save/load, the scene lifecycle
and the per-frame tick of all of the above.

**Singleton identity (the key anchor):**

* **PC:** the `C_game` object is `*(int *)(g_act_mission + 0x24)`, reached in code either
  through `act_mission_get_game` or as `*(g_display + 0x24)`. Any function taking that
  pointer as its `this` (ECX) is a `C_game` method.

**Size:** `0x3698` (`mission_init` calls `operator_new(0x3698)`). The object is heap-allocated
by `mission_init` and owned by `C_mission`; the back-pointer is `C_game + 0x44`.

**No vtable.** `game_ctor` opens by storing the allocator byte of the frame-placement vector at
`+0x00`, its only image-address constant is its exception handler, and every `C_game` method is
called directly; the PS2 has no `__vt__6C_game` either (§6f).

**PC cluster:** the `g_game.cpp` translation unit, `[0x005dc190, 0x005f79a0)`; the
constructor and destructor sit in `Mission.cpp` instead (§5).

**Coverage:** the PS2 symbol table holds **175 distinct `C_game` methods**. Matching that
list against the PS2-twin column of `Game-ghidra.exe.functions.md` leaves **146 located**
and **29 without a PC function of their own** — enumerated and classified in §6b. `C_game` also has
**39 PC functions with no PS2 twin at all**.

Everything outside §6 is a PC fact; §6 is the one section that carries PS2 material.

## 1. Method coverage

One row per method whose identification is written up here, PC-first. The full PC function
list, including the rows whose evidence lives only in their docblocks, is
`Game-ghidra.exe.functions.md`.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| `0x005dc240` | `game_init` | `Init__6C_gameFbb` | `0x002ea0a0` | CONFIRMED | Per-scene STATE reset: freeride score, render flags, stop sounds, state vectors, `police_board_reset`, `+0x2fd4 = 7000`, the decal-texture loads. Called by `loadsave_savegame_load` after `mission_open`. |
| `0x005df6b0` | `game_done` | `Done__6C_gameFv` | `0x002eda50` | CONFIRMED | Teardown behind the `+0x40` guard: stops sounds, tears down camera / police / garage / blood / particles, despawns pursuit assets, sniper off, clears the death queue and the temporary actors. Inlines most `Clear*` / `Del*` helpers. |
| `0x005e0780` | `game_done_intern` | `DoneIntern__6C_gameFv` | `0x002ef110` | CONFIRMED | The resource half of the teardown, once-gated on `+0x363d`; drains the cartridge pools. |
| `0x005e0c20` | `game_tick` | `Tick__6C_gameFUi` | `0x002ef870` | CONFIRMED | The per-frame world tick: pursuit / zone / waypath / police ticks, plus six helpers MSVC inlined here (§6a). |
| `0x005e27e0` | `game_vehicle_track_tick` | `DoubleKiss__6C_gameFUi` | `0x002f1350` | CONFIRMED | Per-frame update of the frame at `+0x2f50` that tracks the player's car (gate `+0x2f14 & 8`; player `+0xe4` kind == 4; distances 2500 / 1600). The PS2 name is a developer joke; the PC name is from behaviour. |
| `0x005e3220` | `game_add_temporary_actor` | `AddTemporaryActor__6C_gameFP7C_actor` | `0x002f1d20` | CONFIRMED | Register a `C_actor` as a scene-lifetime temporary: dedup the master vector, actor-virtual, branch on `act_state` (0/1) into vector A or B, push into the master vector. |
| `0x005e35d0` | `game_add_temporary_actor_intern` | `InternTempActAdd__6C_gameFP7C_actor` | `0x002f1ed0` | CONFIRMED | The same without the register-virtual: dedup master, branch on act-state into `+0xec` / `+0xfc`, push master. |
| `0x005e3400` | `game_remove_temporary_actor` | `RemoveTemporaryActor__6C_gameFP7C_actor` | `0x002f1e30` | CONFIRMED | The DEFERRED retirement queue: return if already queued, `SetActState(2)` (spent), push onto `+0x134`. That act-state 2 is what `grenade_on_collision` and `grenade_hit` test. |
| `0x005e37a0` | `game_remove_temporary_actor_intern` | `InternTempActDel__6C_gameFP7C_actor` | `0x002f1fd0` | CONFIRMED | The removal without the actor virtuals: returns if the actor is queued for retirement (`+0x134`), else swap-removes it from the master vector `+0x124` and the two act-state buckets `+0xec` / `+0xfc`. `police_manager_game_done` uses it for a police car carried into the next mission. |
| `0x005e3920` | `game_remove_temporary_actor_now` | `_RemoveTemporaryActor__6C_gameFP7C_actor` | `0x002f2100` | CONFIRMED | The IMMEDIATE half: find in the master vector, swap-remove, `game_invalidate_actor(actor, 0)`, two actor virtuals. |
| `0x005e39c0` | `game_clear_temporary_actors` | `ClearTemporaryActors__6C_gameFv` | `0x002f2280` | CONFIRMED | Empty the master temp-actor vector: loop, skip two sentinel globals, `game_invalidate_actor` plus two virtuals each, zero the count. |
| `0x005e3a50` | `game_update_controls` | `UpdateControls__6C_gameFv` | `0x002f2340` | CONFIRMED | Per-frame: if `+0x40` is set, walk the world actor list (`g_act_mission + 0x38`) and call `player_update_controls` on every kind-2 entry. |
| `0x005e3df0` | `game_new_shoot` | `NewShoot__6C_gameFP7C_actorRC8S_vectorRC8S_vectorfiP9I3D_framei` | `0x002f26a0` | CONFIRMED | Fire a weapon shot: normalise the direction, optional bullet-tracer actor (pool `+0x2ad8`, 12.0 / 100.0 gates), cast the ray (single below count 2, else a `count`-pellet spread on an orthonormal basis with 0.05 jitter), per-hit material record (`g_surface_material_table + 0x24`, stride `0x170`), impact effect, muzzle particle, gunshot sound (r = 50), NPC look-at broadcast. It is also the producer of the `+0x1d8` in-flight record vector (§4a). |
| `0x005e6760` | `game_new_explosion` | `NewExplosion__6C_gameFP7C_actorR8S_vectorffbbbi` | `0x002f5580` | CONFIRMED | Both builds sweep the actor lists TWICE, backwards, splitting on entity kinds `0xc` / `0xb` (hit virtual, hit type 2, a three-float radius block) versus a squared-distance test into a shared `_explosion` helper. Callers `crash_obj_ai` and `grenade_ai_tick`. |
| `0x005e72e0` | `game_new_smoke_trail` | `NewSmokeTrail__6C_gameFR8S_vectorff` | `0x002f6040` | CONFIRMED | Spawn a smoke-trail particle actor into the 32-slot ring at `+0x324` (cursor `+0x3a4`): dedup, random unit direction (`rand × 2 − 1` / `rand + 0.3`, `< 0.4 → +0.4`, normalise) scaled by a length lerped between the two float arguments, then `game_add_temporary_actor`. The ring's 32 `C_smoke_trail` actors are built by `game_init`; the only caller is `game_new_explosion`. |
| `0x005e7580` | `game_new_small_fires` | `NewSmallFires__6C_gameFR8S_vectori` | `0x002f6380` | CONFIRMED | Spawn N small fires at random offsets around a position (`rx² + ry² + 0.25`, `rand × 2` per fire). |
| `0x005e7800` | `game_new_fire` | `NewFire__6C_gameFR8S_vectorP9I3D_frameiffb` | `0x002f6660` | CONFIRMED | Spawn one `C_fire` (its `fire_ctor` / `fire_init` pair) and register it at `+0x318`; its three callers are in `structs/fire.md` §3. |
| `0x005e7a40` | `game_new_cartridge` | `NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` | `0x002f6860` | CONFIRMED | Eject a shell casing from one of the two pools (§4l). The PS2 twin is an 8-byte `{ return; }` stub, so no casing is ever ejected on that build. |
| `0x005e8010` | `game_new_grenade` | `NewGrenade__6C_gameFiR8S_vectorR8S_vectorbP9I3D_frame` | `0x002f6870` | CONFIRMED | Spawn a grenade/molotov projectile actor: type 0/1 → `2grenade.i3d`, 2/3 → `2mol.i3d`; allocate renderer frame type 9, model-cache lookup/evict, spawn a temporary actor. |
| `0x005e8790` | `game_stop_3d_sound` | `Stop3DSound__6C_gameFibf` | `0x002f7720` | CONFIRMED | Stop a 3D sound by handle: scan the `S_sound_play` vector at `+0x2f8`, match the id, stop and swap-remove. |
| `0x005e8840` | `game_play_3d_sound` | `Play3DSound__6C_gameFP9I3D_sound` | `0x002f77d0` | CONFIRMED | Start a positional sound: virtual play call, increment the handle counter `+0x30c`, return the pre-increment value as the handle, `push_back` an `S_sound_play` record. |
| `0x005e8e30` | `game_add_frame_for_shadow` | `AddFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_typeP9I3D_framePc` | `0x002f8ca0` | CONFIRMED | Add an `I3D_frame` as a shadow caster: bounds-check the `+0x284` slot table, select the shadow vector by `E_shadow_type`, dedup the frame, intern the name through the `+0x3678` free list. Called on the vehicle-exit path. |
| `0x005e9350` | `game_del_frame_for_shadow` | `DelFrameForShadow__6C_game…` | `0x002f8ed0` | STRUCTURAL | Remove a frame from `+0x290` (type 0) or `+0x2a0` (else), stride `0x1c`; the inverse of the adder. The PS2 twin is an 8-byte `return 0;` stub, so there is no body to diff. |
| `0x005e9950` | `game_load_traffic_signals` | `LoadSemaphores__6C_gameFv` | `0x002f90c0` | CONFIRMED | Clear the `+0x1a8` signal vector and re-enumerate the scene into it through `game_enum_traffic_signal_frames` (record stride `0x14`). |
| `0x005e99b0` | `game_free_traffic_signals` | `FreeSemaphores__6C_gameFv` | `0x002f9120` | CONFIRMED | Release the three sub-light objects per signal (`elem+0x4/+0x8/+0xc`, vtable `+0x24`), then clear. |
| `0x005e9a60` | `game_tick_traffic_signals` | `TickSemaphores__6C_gameFv` | `0x002f9200` | CONFIRMED | Drive the three sub-lights of each `+0x1a8` record from the phase table `g_game_semaphore_states` (PS2 `semaphore_states`). |
| `0x005e9d70` | `game_city_music_off` | `CityMusicOff__6C_gameFv` | `0x002f9c90` | STRUCTURAL | Silence the city music: clear the current track id `+0x2fb4`, destroy the outgoing track's media `+0x2fbc` at once, make the incoming track `+0x2fb8` the active one and fade it, unlooped, from its current volume to 0 over 5000 ms; `+0x2fb8 = -1`. Graded on call position (§6f). |
| `0x005e9df0` | `game_city_music_set_paused` | `CityMusicPaused__6C_gameFb` | `0x002f9cf0` | CONFIRMED | Set the gate `+0x2fc0`; when set, call `game_city_music_off`. Byte-match with the PS2 (`+0x2fc0` ↔ `+0x13d0`). |
| `0x005eba30` | `game_invalidate_coll` | `InvalidateColl__6C_gameFPUi` | `0x002fa5c0` | CONFIRMED | Invalidate collision for a set of ids; paired with `collision_dyn_erase` in the erase path. |
| `0x005ebac0` | `game_invalidate_actor` | `InvalidateActor__6C_gameFP7C_actorb` | `0x002fa670` | CONFIRMED | Walk the global actor table, dispatch per kind, clear every reference to a dying actor. |
| `0x005ec250` | `game_del_frame_placement` | `DelFramePlacement__6C_gameFP9I3D_frame` | `0x002fb1f0` | CONFIRMED | Drop a record from the frame-placement registry at `+0x04`. |
| `0x005ec2e0` | `game_raise_timer_timeout_script_event` | `Timer_TimeOut__6C_gameFv` | `0x002fb3b0` | CONFIRMED | Raise the registered program's event when a script timer times out (slot `+0x2d48` / `+0x2d4c`). |
| `0x005ec300` | `game_raise_player_fire_script_event` | `PFire_Fire__6C_gameFv` | `0x002fb3e0` | CONFIRMED | Player-fired-a-weapon hook: raise the event at `+0x2d54` on the program at `+0x2d50` through `program_set_event`. |
| `0x005ec320` | `game_raise_horn_script_event` | `PHorn_Horn__6C_gameFv` | `0x002fb410` | CONFIRMED | Player sounds the horn (slot `+0x2d58` / `+0x2d5c`). |
| `0x005ec340` | `game_raise_water_fall_script_event` | `WFall_Player__6C_gameFv` | `0x002fb440` | CONFIRMED | Player water-fall event (slot `+0x2d60` / `+0x2d64`); the exact in-game trigger is unverified. |
| `0x005ec3d0` | `game_do_npc_kill_event` | `Do_NPCKillEvent__6C_gameFPc` | `0x002fb560` | CONFIRMED | The player killed an entity: take the killed entity's model-name basename, scan the `+0x2d6c` registry `{program +0x0, event +0x4, name +0x8}` and fire the matching script event. Sole caller `pedestrian_hit`, which is also the PS2 caller. |
| `0x005ec460` | `game_add_driver_man` | `AddDriverMan__6C_gameFP9I3D_modelP5C_carib` | `0x002fb610` | CONFIRMED | Attach a driver rig to a car and register it at `+0x2d7c`. Picks between two shipped assets: `AutoRidicVolant.I3D` (*auto řidič volant* = car driver steering wheel) for an ordinary car and `FormuleVolant.I3D` for an open-wheel racer, selected by the trailing bool. |
| `0x005ec700` | `game_del_driver_man` | `DelDriverMan__6C_gameFP9I3D_model` | `0x002fb8d0` | CONFIRMED | Remove a driver rig from `+0x2d7c`. |
| `0x005ed920` | `game_create_grenade_track_effect` | `CreateTrackEffect_Grenade__6C_gameFR8S_vector` | `0x002fbee0` | CONFIRMED | Spawn a grenade-impact track/scorch effect at a position (constants −1.0 / 0.8 / 1.45 / 0.2, `pos.y + 1.0`, random). |
| `0x005ee690` | `game_box_add` | `BoxPlacementAdd__6C_gameFP9I3D_framei` | `0x002fd040` | CONFIRMED | Register an `I3D_frame` as a placeable box in `+0x2e84` (marker `0x1d97c`); calls `game_add_temporary_actor_intern`. |
| `0x005eeb70` | `game_box_del` | `BoxPlacementDel__6C_gameFP9I3D_frame` | `0x002fd440` | CONFIRMED | Unregister a box: `collision_dyn_erase`, path release, free, swap-remove from `+0x2e84`. |
| `0x005eec50` | `game_box_recompile` | `BoxPlacementRecompile__6C_gameFv` | `0x002fd530` | CONFIRMED | Rebuild the box stacking / path-node graph (neighbour search; node timer `0x7da`). |
| `0x005eeeb0` | `game_box_dest_add` | `BoxPlacementDestAdd__6C_gameFP9I3D_framei` | `0x002fd880` | CONFIRMED | Push a `{dest_frame, owner}` pair into the dest vector `+0x2e94`. |
| `0x005ef080` | `game_box_dest_set_ou` | `BoxPlacementDestSetOU__6C_gameFbP9I3D_frame` | `0x002fd940` | CONFIRMED | Build the carried-box-to-dest using-object path node (`+0x2ea0`, timer `0x7db`) plus `game_box_save_info`; returns the owner. |
| `0x005ef220` | `game_box_repose_dest_player` | `BoxPlacementReposeDestPlayer__6C_gameFv` | `0x002fdb00` | CONFIRMED | Rebuild the player's carried-box dest path node from `player+0x6f8` / `+0x6fc`. |
| `0x005ef340` | `game_box_save_info` | `BoxPlacement_SaveInfo__6C_gameFP9I3D_frame` | `0x002fdc30` | CONFIRMED | Store a box frame's transform into the `+0x34` saved-info vector (stride `0x20`). |
| `0x005ef720` | `game_box_del_info` | `BoxPlacement_DelInfo__6C_gameFP9I3D_frame` | `0x002fe010` | CONFIRMED | Swap-remove a box's saved-info record from `+0x34`. |
| `0x005ef7a0` | `game_box_restore_info` | `BoxPlacement_RestoreInfo__6C_gameFP9I3D_frame` | `0x002fe0c0` | CONFIRMED | Re-apply a box's saved transform to its frame, then remove the record. |
| `0x005ef890` | `game_box_get_start_point` | `BoxPlacement_GetStartPoint__6C_gameFP9I3D_frameR8S_vector` | `0x002fe230` | CONFIRMED | Copy a box's saved start position out of `+0x34` into an out-vector, world-transformed; returns 1/0. |
| `0x005ef910` | `game_box_place_at_next_dest` | `BoxPlacementGetNextDest__6C_gameFP9I3D_frameP9I3D_frame` | `0x002fe310` | CONFIRMED | PLACE a box at its dest: copy the transform, reparent, remove the dest, `game_box_add(box, 0x1d97c)`, car branch. The PS2 verb `GetNextDest` does not describe the body. |
| `0x005efca0` | `game_box_get_enemy_dest` | `BoxPlacementGetEnemyDest__6C_gameFP7C_humani` | `0x002fe600` | CONFIRMED | Pick an NPC's box dest: the nearest owned box (squared distance plus a 10.0 height penalty) when not carrying (`human+0x6f8`), else the assigned dest. |
| `0x005efe20` | `game_box_count_owned` | `BoxPlacement_GetBox__6C_gameFi` | `0x002fe7c0` | CONFIRMED | COUNT of boxes in `+0x2e84` whose owner (`box+0x90`) equals the argument. The PS2 verb `GetBox` does not describe the body. |
| `0x005efe60` | `game_box_dest_count_owned` | `BoxPlacement_GetBoxDest__6C_gameFi` | `0x002fe810` | CONFIRMED | COUNT of dests in `+0x2e94` whose owner (`+0x04`) equals the argument. |
| `0x005efea0` | `game_box_move_to_dest` | `BoxPlacement_MoveBoxToDest__6C_gameFi` | `0x002fe860` | CONFIRMED | Find an owner's box and dest and run the move sequence (set-ou / del / recompile / reparent / repose). |
| `0x005eff70` | `game_box_recompile_for_car` | `BoxRecompileBoxForCar__6C_gameFP5C_carb` | `0x002fe9b0` | CONFIRMED | Detach boxes parented to a car (`car+0x68`): clear `box+0x25` and `collision_dyn_erase`. |
| `0x005f0170` | `game_box_count_owned_on_car` | `BoxPlacementPoPiciNaZkurvenemAute__6C_gameFP5C_cari` | `0x002fec50` | CONFIRMED | COUNT of an owner's boxes attached to a car: iterate `+0x2e84` by `owner+0x90`, walk each box's child list, count nodes whose object is the car. The PS2 name is developer slang. |
| `0x005f01e0` | `game_death_actor_add` | `DeathActorAdd__6C_gameFP7C_actor` | `0x002fedc0` | CONFIRMED | Dedup-enqueue an actor onto the death queue `+0x2ee8`. |
| `0x005f03a0` | `game_process_death_actors` | `DeathActorProcess__6C_gameFv` | `0x002fee50` | CONFIRMED | Per-frame: drain `+0x2ee8` / `+0x2eec` and force-kill each kind-2 / kind-`0x1b` entry through `human_force_death`. |
| `0x005f0450` | `game_save_game_get_size` | `SaveGameGetSize__6C_gameFb` | `0x002fef80` | CONFIRMED | Serialized size of the save block: pursuit, the `0x43`-byte header, two stack tables, the inventory-size loop, using-object; the full flag adds the media block. |
| `0x005f05a0` | `game_save_game_save` | `SaveGameSave__6C_gameFPPvb` | `0x002ff020` | CONFIRMED | Serialize: header (counts, score `[0x3e]` / `[0x3f]` from `+0x363e` / `+0x3640`, timer), `cursor += 0x43`, pursuit save, sub-objects. |
| `0x005f0990` | `game_save_game_load` | `SaveGameLoad__6C_gameFPPv` | `0x002ff1f0` | CONFIRMED | Deserialize behind a tag-`0x0a` gate: header, score (through `game_freeride_score_set_on` / `_set`), timer, `cursor += 0x43`, pursuit load, stack clears, sub-objects. The score restore is byte-identical to the PS2. |
| `0x005f1090` | `game_savegame_load_before` | `SaveGameLoadBefore__6C_gameFv` | `0x002ff4c0` | CONFIRMED | Pre-load hook; reads the mission-number pair on `C_mission`. |
| `0x005f17c0` | `game_add_shoot_light` | `AddShootLight__6C_gameFR8S_vectorfiUs` | `0x002ff630` | CONFIRMED | Light a slot of the 10-slot shoot-light pool at `+0x2a48` for a given lifetime (§4a). The PS2 twin is an 8-byte stub, so the PC body is the only one that exists. |
| `0x005f1b10` | `game_media_create` | `CreateIShow__6C_gameFUi` | `0x002ffc20` | CONFIRMED | Allocate a media entry: fresh id from the `+0x2f38` counter, engine create, push into `+0x2f1c`. |
| `0x005f1d90` | `game_media_open` | `OpenIShow__6C_gameFPCci` | `0x002ffd20` | CONFIRMED | Open/load an in-game FMV clip by id into `+0x2f1c`, with a `.ext` → `.mp3` audio fallback. |
| `0x005f1f40` | `game_media_destroy` | `DestroyIShow__6C_gameFi` | `0x002fff30` | CONFIRMED | Remove one entry by id: release, free the name, swap-remove. |
| `0x005f2030` | `game_media_destroy_all` | `DestroyIShowAll__6C_gameFv` | `0x00300060` | CONFIRMED | Release every entry and reset the vector. |
| `0x005f2140` | `game_media_get` | `GetIShow__6C_gameFi` | `0x00300120` | CONFIRMED | Find a media entry by id and return its show object. |
| `0x005f21a0` | `game_media_store_volume` | `IShowStoreVolume__6C_gameFif` | `0x00300180` | CONFIRMED | Store the per-entry volume (`element+0x10`) by id. |
| `0x005f2200` | `game_media_set_loop` | `IShowLoop__6C_gameFib` | `0x003001d0` | CONFIRMED | Set or clear the loop flag (`element+0x08` bit 0) by id. |
| `0x005f2280` | `game_media_apply_volumes` | `IShowSetVolume__6C_gameFv` | `0x003002a0` | CONFIRMED | Re-apply `g_music_volume` × per-entry volume to every entry. |
| `0x005f22e0` | `game_media_fade_all` | `IShowFadeDownAll__6C_gameFi` | `0x00300320` | CONFIRMED | Fade every entry through `game_media_fade`, or stop it when the volume is already low. |
| `0x005f2390` | `game_media_fade` | `IShowFade__6C_gameFiffi` | `0x00300420` | CONFIRMED | Fade one entry by id, from one volume to another over a duration. |
| `0x005f2790` | `game_load_taxi_points` | `LoadTaxiPoints__6C_gameFv` | `0x00300680` | CONFIRMED | Clear and re-enumerate the scene for `taxi_point` objects into `+0x1f4`. The PC refactors the PS2's inline scan into the shared scene enumerator. |
| `0x005f2990` | `game_is_valid_taxi_passenger` | `IsValidTaxiPassenger__6C_gameFv` | `0x00300ad0` | CONFIRMED | The candidate at `+0x204` is valid iff its flag byte `+0x5e` is clear, its group `+0x98` is set, and the player's group (`+0xe4`) matches or is unset. |
| `0x005f29d0` | `game_del_traffic_signal` | `DelSemaphore__6C_gameFP9I3D_frame` | `0x00300b30` | CONFIRMED | Swap-remove a traffic-light record by frame key from `+0x1a8` (stride `0x14`). |
| `0x005f2a80` | `game_load_telephones` | `LoadTelephones__6C_gameFv` | `0x00300bf0` | CONFIRMED | Clear and re-enumerate the scene for `alarm01.i3d` / `PHbox01.i3d` callable devices into the `+0x1c4` list. This diff is what proved the singleton the PS2 calls `world` is `C_game`. |
| `0x005f30c0` | `game_telephone_list_erase_by_key` | `DelTelephones__6C_gameFP9I3D_frame` | `0x00300f60` | CONFIRMED | Swap-remove a callable device by frame key (stride `0xc`). |
| `0x005f3180` | `game_telephone_list_contains_key` | `IsTelephoneExist__6C_gameFP9I3D_frame` | `0x00301010` | CONFIRMED | Is a callable device registered for this frame? A backward scan of the list. |
| `0x005f34e0` | `game_build_traffic_signal_model_idx` | `CreateSignalMarkIDX__6C_gameFv` | `0x00301250` | CONFIRMED | Scan the scene model table for the phase models `dopravni1B..8B.I3D` and build the u16 model-index list at `+0x1e8`. |
| `0x005f3dd0` | `game_flush_pending_actor_lookups` | `FindActorsVect__6C_gameFv` | `0x00301ae0` | CONFIRMED | Drain the deferred actor-lookup queue at `+0x2f80` (stride `0xc`) and clear it. It touches no audio. |
| `0x005f40b0` | `game_pause_all_sounds` | `PauseAllSounds__6C_gameFv` | `0x00301c00` | CONFIRMED | Pause every sound, saving state into `+0x2d8` and the media-id sibling `+0x2e8`. The exact inverse of `game_play_all_sounds`, proven by the shared callee: `speech_pause_all_channels()` is `DabingPauseSounds(1,0)` here and `(0,0)` there, in the same statement position. |
| `0x005f4400` | `game_scratch_add` | `ScratchAdd__6C_gameFP5C_carP9I3D_frameUiRC8S_vectorRC8S_vector` | `0x00302500` | CONFIRMED | Stamp a car scratch/decal: `OBJ_`-frame and surface-type gate (excludes {2,3,4,5,7}), allocate a pool slot, orient (`SetDir3` or random spin), speed alpha (`× 1/55.0`, clamped 0.2 … 1.0), age alpha, then recompile. PS2 caller `Deform_Vehicle_Part`. |
| `0x005f4830` | `game_scratch_recompile` | `ScratchRecompile__6C_gameFP5C_carP9I3D_frame` | `0x00302ae0` | CONFIRMED | Rebuild the decal mesh for the pool record matching `{car, source_frame}`: save state, set flag, mesh-build virtual `(0x10, source)`, restore state. |
| `0x005f48a0` | `game_scratch_delete_from_car` | `ScratchDeleteFromCar__6C_gameFP5C_car` | `0x00302be0` | CONFIRMED | Remove every scratch belonging to a car: scan the 100-record pool for `car == param`, two frame virtuals (`+0x2c`(0,0) then `+0x24`(0)). |
| `0x005f48e0` | `game_scratch_get_free_idx` | `ScratchGetFreeIdx__6C_gameFP5C_carP9I3D_frameRC8S_vector` | `0x00302c90` | CONFIRMED | Allocate a pool slot in three passes: dedup (< 0.16, reuse the min-age of three or more), free slot (`car == 0`), LRU evict (farthest from the camera, 25.0 initial). Returns the index. |
| `0x005f5010` | `game_tick_scream_areas` | `TickScreamAreas__6C_gameFv` | `0x00303290` | CONFIRMED | Walk the scream-area vector `+0x361c` (stride 8, `{frame, u16 mask}`) against the camera and OR the hit masks into `+0x3628` (1 when none hit). |
| `0x005f5bf0` | `game_freeride_score_set_on` | `ScoreSetOn__6C_gameFb` | `0x00304330` | CONFIRMED | Freeride score-display toggle: `+0x363e = on`; refresh the HUD either way (the PS2 hides it with `BonusSetCount(0x7fffffff)`). |
| `0x005f5c30` | `game_freeride_score_set` | `ScoreSet__6C_gameFi` | `0x00304380` | CONFIRMED | `+0x3640 = value`; if `+0x363e` is set, refresh the HUD. |
| `0x00425580` | `game_freeride_score_get` | `-` | `-` | CONFIRMED | `return this+0x3640`. PC-only (an MSVC-emitted accessor) — the PS2 reads `+0x153c` inline. Consumed by the `FREERIDE_SCOREGET` / `SCOREADD` script commands. |
| `0x005f5d60` | `game_tick_car_sound_manager` | `TickCarSoundManager__6C_gameFv` | `0x00304550` | CONFIRMED | Budget car sounds to the nearest N cars by camera distance (gate `+0x3668` ↔ PS2 `+0x1550`; car virtuals `+0x94` / `+0x51e`). No radio is involved. |
| `0x005f5f60` | `game_item_blur_done` | `ItemBlurDone__6C_gameFv` | `0x00304830` | CONFIRMED | Clear every blur item, reset `end = begin` and the enable byte `+0x3684`. The PC clears and RETAINS capacity; the PS2 frees it (the PC storage free is in `game_dtor`). |
| `0x005f5fe0` | `game_item_blur_tick` | `ItemBlurTick__6C_gameFUi` | `0x00304910` | CONFIRMED | Per-frame motion-blur streak-trail update: ms to seconds, camera frame and `S_matrix::Inverse`, iterate `+0x368c`, 900.0 (30 u²) gate, × 0.5 fade, virtual sub-item destroy. Called from `game_tick` behind `+0x3684`. |
| `0x005f6690` | `game_item_blur_del_item` | `ItemBlurDelItem__6C_gameFib` | `0x00305300` | CONFIRMED | Remove one blur item from `+0x368c` (stride `0x28`): destroy the sub-item list; when `compact` is set, swap-remove. |

## 2. Field map

Offsets are relative to the `C_game` object. PC↔PS2 deltas are **not constant** across the
class — the packing differs — so pair per field and never extrapolate a delta across a band.
Confidence: **[H]** measured on both builds · **[M]** measured on PC, meaning partly open ·
**[U]** located, meaning unknown.

| PC offset | field | conf | PS2 | evidence |
|---|---|---|---|---|
| `+0x00` .. `+0x0c` | frame-placement registry, a vector (allocator `+0x00`, begin `+0x04`, end `+0x08`, cap `+0x0c`) of `0x34`-byte records, key (`I3D_frame *`) at record `+0x00` | H | `+0x00` / `+0x04`, `0x40`-byte records, key at `+0x30` | `game_ctor`, `game_del_frame_placement` |
| `+0x10` .. `+0x1c` | the broken-scene-model vector (allocator `+0x10`, begin `+0x14`, end `+0x18`, cap `+0x1c`): `crash_obj_create_from_model` pushes each scene model it breaks into crash pieces and switches it off; `game_done` switches every frame in it back on (frame vtable `+0x24` with 1); `game_init` empties it | H | `+0x10` / `+0x14` | `crash_obj_create_from_model`, `game_ctor`, `game_init`, `game_done` |
| `+0x20` .. `+0x2c` | created-frame vector (script-created frames): allocator `+0x20`, begin `+0x24`, end `+0x28`, cap `+0x2c` | H | `+0x18` / `+0x1c` / `+0x20` | `game_add_created_frame` / `_del_created_frame`; `game_done` releases each frame |
| `+0x34` / `+0x38` / `+0x3c` | box saved-info vector, stride `0x20` (`{frame, node, saved pos/dir}`) | H | - | `game_box_save_info` / `_restore_info` / `_get_start_point` |
| `+0x40` | the run guard — `game_done` returns unless it is set, `game_update_controls` gates on it | H | - | `game_done`, `game_update_controls` |
| `+0x44` | the owning `C_mission` back-pointer, written by `mission_init` right after the allocation | H | `+0x5c` | `mission_init`, `game_ctor` |
| `+0x48` | mission-end action (1/2/3) from the game-over screen | H | `+0x60` | `game_set_mission_end_action` |
| `+0x4c` | embedded **`G_Camera`** sub-object | H | - | §3 |
| `+0xe4` | **the player's `C_actor`** | H | `+0x150` | read through `game_get_player_actor` (`0x00425530`, 48 references). Written by `game_init`, which clears it and then takes the last kind-2 actor of the mission actor list whose `+0x04` track bit `0x10` is clear; by `engine_new_mission`; and by `loadsave_savegame_load`, which takes the first kind-2 actor while it is unset. `game_done` clears it, and `game_invalidate_actor` clears it when the dying actor is the player. Also the source of the `*(world+0xe4) == this` "am I the player?" downcast gate several `C_human` methods use before touching `C_player` fields |
| `+0xe8` | the human the camera follows | H | - | `game_set_human` (kinds 2 and `0x1b` only, then `camera_set_player`) and `racing_activate_racers`; `game_init` copies `+0xe4` into it, and `engine_new_mission` / `loadsave_savegame_load` set it together with `+0xe4` |
| `+0xec` | temp-actor vector A (`act_state == 0`) | H | `+0x158` | `game_add_temporary_actor`; address accessor `game_get_temp_actor_list_a` (`0x00425520`) |
| `+0xfc` | temp-actor vector B (`act_state == 1`) | H | `+0x164` | `game_add_temporary_actor` |
| `+0x114` .. `+0x11c` | the dying-human list | H | - | `game_dying_human_add` / `_del` / `_tick` |
| `+0x124` / `+0x128` / `+0x12c` | master temp-actor vector, `std::vector<C_actor *>` | H | `+0x180` / `+0x184` / `+0x188` | `game_add_temporary_actor` |
| `+0x134` / `+0x138` / `+0x13c` | the temp-actor RETIREMENT queue (deferred removal) | H | - | `game_remove_temporary_actor` |
| `+0x140` .. `+0x14c` | the carried-actor vector (allocator `+0x140`, begin `+0x144`, end `+0x148`, cap `+0x14c`): the player and the car carried over by the script command `change_mission` (`g_new_mission_actors`). `engine_new_mission` pushes them after `game_init`, sets `owns_frame` (`+0x6c`) on the one that is not a car and clears the frame parent `+0x58` of both; `game_done` first drops a car parked in the garage, then calls `mission_active_actor_drop` and `mission_del_actor` on every entry, clearing a car's `owns_frame`; `game_init` empties it; `loadsave_savegame_save` writes these actors into a mission save | H | - | `engine_new_mission`, `game_done`, `game_init`, `loadsave_savegame_save` |
| `+0x154` .. `+0x160` | the lightness records and their round-robin cursor | H | - | `structs/lightness.md` §2b |
| `+0x164` | the object-lightness enable byte: while it is set, `game_tick` runs `game_lightness_tick` (also gated on a non-empty lightness vector and a detail setting of 2 or more); while it is clear the pass is skipped. Set only by the script command `ACTORLIGHTNESSON` (opcode `0x143`), cleared by `game_init` | H | `+0x1b4` | writers `game_set_actor_lightness_flag`, `game_init`; reader `game_tick` ¹ |
| `+0x16c` .. `+0x178` | the `I3D_light` cache and its `0x7e7` re-test countdown | H | - | `structs/lightness.md` |
| `+0x180` .. `+0x188` | the invisible-actor list, 8-byte `{C_actor *, 0}` records | H | - | `game_add_invisible_actor` |
| `+0x18c` / `+0x190` | the traffic-generator pick timer and its period (6000 ms, set by `game_init`) | H | `+0x1d4` / `+0x1d8` | `game_tick`, `game_init` (§6a) |
| `+0x194` .. `+0x1a0` | the traffic-light clock: `+0x194` elapsed ms, `+0x198` phase period (3000 ms), `+0x19c` / `+0x1a0` the phases of the two crossing axes, 0..9 | H | `+0x1dc` .. `+0x1e8` | `game_init` (period, phases 0 and 5), `game_tick` (advances both mod 10, then `game_tick_traffic_signals`); §4d, `structs/driver.md` §3a |
| `+0x1a8` / `+0x1ac` | traffic-signal ("semaphore") vector `begin`/`end`, stride `0x14` (`{frame-key +0x0, three sub-lights RED +0x4 / ORANGE +0x8 / GREEN +0xc, phase-idx ptr +0x10}`); phase states from `g_game_semaphore_states` | H | `+0x1f0` / `+0x1f4` (PS2 count/begin) | `game_load_traffic_signals`, `game_free_traffic_signals`, `game_tick_traffic_signals`, `game_del_traffic_signal`, `game_enum_traffic_signal_frames` |
| `+0x1b4` .. `+0x1bc` | the roadblock vector, stride `0x14`, elements owning their own dword vector | H | - | the roadblock family |
| `+0x1c4` .. `+0x1cc` | callable-device ("telephone") list: header `+0x1c4`, `begin` / `end` `+0x1c8` / `+0x1cc`, stride `0xc` (`{frame +0x0, _ +0x4, type-byte +0x8}`) ² | H | `+0x208` / `+0x20c` (PS2 count/begin) | `game_load_telephones`, `game_telephone_list_contains_key`, `game_telephone_list_erase_by_key`, `game_get_telephone_list` ² |
| `+0x1d8` .. `+0x1e0` | the **`S_shoot`** in-flight bullet vector, stride `0x5c`: position `+0x00`, velocity `+0x0c`, range `+0x18`, previous position `+0x1c`, damage `+0x28`, owner `+0x2c` | H | `+0x210` / `+0x214`, stride `0x70` | producer `game_new_shoot`, consumer `game_tick_shoot` |
| `+0x1e8` / `+0x1ec` | the traffic-signal phase model-id list (u16 ids of the `dopravni1B..8B.I3D` models in the scene table) | M | `+0x21c` (count `+0x220`) | producer `game_build_traffic_signal_model_idx`, consumer `game_destroy_control_signal` |
| `+0x1f4` / `+0x1f8` / `+0x1fc` | taxi pick-up-point vector `base` / `begin` / `end` | M | `+0x228` | `game_load_taxi_points` (`taxi_point` scene objects) |
| `+0x204` | current taxi-passenger candidate actor | M | `+0x234` | `game_is_valid_taxi_passenger` |
| `+0x208` | embedded **`C_Break_plain_glass`** glass-shatter engine | H | - | §3 |
| `+0x240` | embedded **`C_Debris`** debris and spark effect | H | - | §3 |
| `+0x26c` | embedded **`C_compass`** | H | - | §3 |
| `+0x284` .. `+0x28c` | the `S_shadow_temp` mesh table, stride 8 | H | `+0x2c4` / `+0x2c8` | `game_add_frame_for_shadow`; `structs/fake_shadow.md` |
| `+0x290` / `+0x2a0` | the two `S_shadow_item` shadow-caster frame vectors, selected by `E_shadow_type`, stride `0x1c` | H | `+0x2d0` / `+0x2dc` | `game_add_frame_for_shadow` / `game_del_frame_for_shadow` |
| `+0x2b0` / `+0x2b4` / `+0x2b8` | the fake-shadow LOD timer, its period and the vector toggle | H | - | `game_tick` (its own LOD pass, §6a) |
| `+0x2c8` .. `+0x2d0` | the live level-effect registry, stride `0x18`: `{particle guid, covered flag, vec3 pos, I3D_sound *}` | H | - | §4d |
| `+0x2d8` / `+0x2dc` | the paused-sound save vector, stride `0x14` | H | - | `game_pause_all_sounds` / `game_play_all_sounds` |
| `+0x2e8` / `+0x2ec` | its media-id sibling, filled only for media that was playing | H | - | the same pair |
| `+0x2f8` / `+0x2fc` / `+0x300` | `std::vector<C_game::S_sound_play>` — the active 3D sounds, stride `0x18` | H | `+0x320` / `+0x324` / `+0x328` | `game_play_3d_sound`, `game_stop_3d_sound`, `game_tick` (inlined `TickEffects`) |
| `+0x304` / `+0x308` | the sound-list walk timer and its period | H | `+0x32c` / `+0x330` | `game_tick` (§6a) |
| `+0x30c` | 3D-sound handle counter, returned as the play handle | H | `+0x334` | `game_play_3d_sound` |
| `+0x310` | the number of `S_sound_play` records still fading; while it is non-zero the sound-list walk runs every frame | H | `+0x338` | `game_stop_3d_sound` (increment), `game_tick` (decrement) |
| `+0x324` (cursor `+0x3a4`) | the 32-slot smoke-trail actor ring | H | - | `game_new_smoke_trail` |
| `+0x3a8` / `+0x538` | cartridge pool A — **100** `C_cartridge *` (model `2nabojnice.i3d`) and its wrap cursor | H | `+0x36c`, **10** entries | §4l |
| `+0x53c` / `+0x5b4` | cartridge pool B — **30** `C_cartridge *` (model `2nabojniceb.i3d`) and its wrap cursor | H | `+0x398`, **5** entries | §4l |
| `+0x75c` .. `+0x29bc` | the decal projector pool — 100 records of `0x58` bytes: projector `+0x00`, a type-`0x48` collision object `+0x04`, its registered flag `+0x4c`, grow flag `+0x4d`, grow scale `+0x50`, owner frame `+0x54` | H | - | `game_create_track_effect` / `_between`, the grow loop in `game_tick`, `game_clear_projectors_for_frame`; `structs/blood.md` §5a |
| `+0x29bc` .. `+0x29e0` | ten decal textures, loaded by name in `game_init`: `2blood.bmp`, `2flek.bmp`, `2dirbet.bmp`, `2dirdre.bmp`, `2dirple.bmp`, `2dirsklo.bmp`, `2bfootprint.bmp`, `2blood3.bmp`, `2dogfoot.bmp`, `skrob.tga` | H | - | `game_init`; `structs/blood.md` §5 |
| `+0x29e4` | the decal ring cursor, reset to 0 in `game_init` | H | - | `game_init`, `game_projector_advance_ring_cursor` |
| `+0x29e8` | the number of decals still growing in; gates the grow loop | H | - | `game_create_track_effect` / `_between` (increment), `game_tick` (decrement), `game_init` (clear); `structs/blood.md` §5c |
| `+0x29ec` | embedded **`C_using_object`** — the usable-object registry behind the player action menu | H | `+0xde0` | §3 |
| `+0x2a48` .. `+0x2ac0` | the 10-slot shoot-light pool, stride `0xc` (`{I3D_light *, int lifetime_ms, u16 flags}`); flag bit 0 = flicker, then fade over the last 500 ms (§4a) | H | - | `game_add_shoot_light`; `game_tick` (inlined `ShootLightTick`); `structs/lightness.md` §4a |
| `+0x2ac0` .. `+0x2ad7` | the bullet-tracer pool, three 8-byte records | M | - | `game_new_shoot` walks them downwards from `+0x2ad0` |
| `+0x2ad8` | "the player touched a death-collision volume" (byte), material 40 (`def-files-docs/materialy.md`) | H | - | set to 1 by `human_movement_intern` (the player on foot) and `car_on_body_surface_contact` (a car the player drives), each also raising the water-fall script event; cleared by `game_init`; `game_tick` then opens the game-over screen, and `race_game_loop` / `race_game_loop_net` read it |
| `+0x2ad9` | one-shot "drop the difference data" request byte | H | - | `game_request_clear_differences`; sole reader `game_tick`. PC-only: the PS2 `CLEARDIFFERENCES` arm is an empty stub |
| `+0x2ae0` / `+0x2af0` | the removed-frame and removed-actor **difference** vectors | H | - | `game_load_differences` / `game_clear_differences` |
| `+0x2b00` | the global-program vector, scanned by name | H | - | `game_global_program_run` |
| `+0x2b0c` | current frame timestamp in ms — the global game clock | H | - | `game_get_frame_time_ms` (`0x00425540`, 148 callsites); the cooldown and duration math throughout the police-task scheduler |
| `+0x2b14` | **frame counter** — `game_init` zeroes it, `game_tick` increments and switches on it | H | `+0xe5c` | `game_get_frame_count` |
| `+0x2b3c` | the **`soundArena`** sub-object (mission sound ring) | H | `+0xe80` | address accessor `game_get_sound_arena` (`0x00425550`); `structs/soundArena.md` |
| `+0x2d44` | **night-mission flag** (byte; 0 = day, 1 = night) | H | - | writer `game_set_night_mission` (`0x005c8b60`); readers `car_update`, `car_deactivate` ³ |
| `+0x2d48` .. `+0x2d67` | **four `(C_program *, event id)` script-event slots**, stride 8 — timer timeout, player fire, player horn, player water-fall, in that order | H | `+0x1194` .. `+0x11b3` | the four `game_raise_*_script_event` functions; a 4-of-4 ordered lockstep with the PS2 |
| `+0x2d6c` / `+0x2d70` | **NPC-kill-event registry**, stride `0x28` (`{program +0x0, event +0x4, name[0x20] +0x8}`), matched by a killed entity's model name | H | `+0x11b8` / `+0x11bc` (PS2 count/begin) | `game_do_npc_kill_event`; the PS2 registrar is `AddNPCKillEvent` |
| `+0x2d7c` / `+0x2d80` / `+0x2d84` | the **driver-rig registry** — `0x10`-byte `{I3D_model *, I3D_animation_set *, …}` records | H | `+0x11c4` / `+0x11c8` | `game_add_driver_man` / `game_del_driver_man` |
| `+0x2d88` / `+0x2d8c` | the frame counts of the two steering-wheel animations, `AutoRidicVolant.I3D` and `FormuleVolant.I3D`, read by `game_init` | H | `+0x11cc` / `+0x11d0` | `game_init`, `game_tick` (§6a) |
| `+0x2d90` | embedded **`C_blood_manager`** | H | - | §3 |
| `+0x2e4c` | the script-stack first-use latch — clearing it clears all three stacks at once | H | - | §4f |
| `+0x2e54` / `+0x2e64` / `+0x2e74` | the inventory, character and float script stacks | H | - | §4f |
| `+0x2e84` / `+0x2e88` | the box vector — `0x98`-byte box records, owner at `box+0x90`, marker `0x1d97c` | H | `+0x12a0` / `+0x12a4` | `game_box_add`, `game_box_count_owned` |
| `+0x2e94` / `+0x2e98` | the box-destination vector — 8-byte `{dest_frame, owner +0x04}` | H | `+0x12ac` / `+0x12b0` | `game_box_dest_add`, `game_box_dest_count_owned` |
| `+0x2ea0` | the player's carried-box path node, rebuilt by repose (node timer `0x7db`) | H | - | `game_box_repose_dest_player` |
| `+0x2edc` / `+0x2ee0` | mission-6 race state and the script float register the race publishes its result into | H | `+0x1304` / `+0x1308` | the script dispatcher arms |
| `+0x2ee8` / `+0x2eec` | the queued dead-actor vector (death queue) | H | `+0x130c` .. `+0x1314` | `game_death_actor_add`, `game_process_death_actors` |
| `+0x2ef4` | the cheat-code buffer (`0x1f` chars) | H | - | `game_cheat_check` |
| `+0x2f14` | the gate bit `0x8` for the player-car tracking frame | M | - | `game_vehicle_track_tick` |
| `+0x2f1c` / `+0x2f20` | the media / in-game-FMV vector, stride `0x14`: `{obj +0x00, id +0x04, flags +0x08 (bit 0 = loop), name +0x0c, volume +0x10}` | H | `+0x1340` / `+0x1344` | the `game_media_*` family |
| `+0x2f38` | the media id counter | H | - | `game_media_create` |
| `+0x2f44` .. `+0x2f4c` | the human-talk queue, 8-byte `{C_human *, speech id}` pairs | H | - | §4g |
| `+0x2f50` | the frame that tracks the player's car | H | - | `game_vehicle_track_tick` |
| `+0x2f80` / `+0x2f84` | the deferred actor-lookup queue, stride `0xc` | H | - | `game_flush_pending_actor_lookups` |
| `+0x2f90` .. `+0x2f98` | the script particle-guid registry (PC-only) | H | - | the particle script commands |
| `+0x2f9c` | the heavy-init flag — gates the 43 shadow-mesh loads and the anim-track preload | H | - | `game_init` |
| `+0x2fa4` | the city-music-area vector | H | - | §4h |
| `+0x2fb0` .. `+0x2fc0` | the **city-music sub-block**: `+0x2fb0` ticker re-eval countdown (ms), `+0x2fb4` current track id, `+0x2fb8` pending track id, `+0x2fbc` active channel handle (−1 = none), `+0x2fc0` paused gate (byte) | H | …/`+0x13c4`/`+0x13c8`/`+0x13d0` — the PS2 uses TWO channels | `game_city_music_set_paused`, `game_city_music_off` |
| `+0x2fc8` .. `+0x2fd0` | the film-music handle vector | H | - | `game_set_film_music` / `_get_film_music` / `_clear_film_music` |
| `+0x2fd4` | the delay from the player's death to the game-over screen, ms | H | - | `game_init` sets `7000`; once the player actor's death-processed latch `+0x5e` is set, `game_tick` counts it down, then starts a 1000 ms fade and opens `gui_game_over` |
| `+0x2fd8` .. `+0x3618` | the **car scratch decal pool** — an inline FIXED array of 100 records, stride `0x10`, `{scratch_frame +0x0, car +0x4, source_frame +0x8, age_counter +0xc}`. `+0x3618` is where the array ends, not a separate structure | H | `+0x13d8`, **20** records | `game_scratch_add` / `_get_free_idx` / `_delete_from_car` |
| `+0x361c` .. `+0x3628` | the scream-area vector (stride 8, `{frame, u16 mask}`) and the OR-ed camera mask word at `+0x3628` | H | - | `game_tick_scream_areas`, producer `game_collect_scream_area` |
| `+0x3630` / `+0x3634` | the weather enable flag and the base particle count | H | - | `game_apply_weather_detail_level` |
| `+0x3638` | rain buffer size — the script command `weather_preparebuffer`; the Mafia Script Data Reference gives 3900 | H | `+0x1534` | the script dispatcher arm |
| `+0x363d` | the `DoneIntern` once-flag | H | - | `game_done_intern` |
| `+0x363e` | freeride score-display ON flag (byte) | H | `+0x153a` | `game_freeride_score_set_on` |
| `+0x3640` | freeride bonus-score value (int) | H | `+0x153c` | `game_freeride_score_set` / `game_freeride_score_get` |
| `+0x3644` .. `+0x3654` | the camera-rotation repair latch and the saved quaternion | H | - | `game_init` |
| `+0x3665` | "the mission is ending under script control" — set by BOTH `endofmission` and `change_mission` | H | `+0x154d` | sole reader `garage_game_done` |
| `+0x3668` / `+0x3670` | car-sound budget count and mode flag (the nearest N cars get sound) | H | `+0x1550` | `game_tick_car_sound_manager` |
| `+0x3678` / `+0x367c` | the recycled `0x20`-byte name free list — where `game_add_frame_for_shadow` interns and returns shadow names | H | - | `game_add_frame_for_shadow` |
| `+0x3684` | ItemBlur enable byte — gates `game_item_blur_tick` from `game_tick`, cleared by `game_item_blur_done` | H | `+0x1568` | the ItemBlur family |
| `+0x368c` / `+0x3690` / `+0x3694` | the **ItemBlur item vector** (motion-blur streak trails) `begin` / `end` / `cap`, stride `0x28`: `{source_frame +0x0.., prev-pos, active byte, sub-item ptr array +0x8, count +0xc}` | H | `+0x156c` / `+0x1570` / `+0x1574`, stride `0x30` | the ItemBlur family |

¹ The gate is `mov al,[ebp+0x164]; test al,al; je` around the `game_lightness_tick` call, followed
by the non-empty `+0x154 .. +0x158` test and `*(g_display + 0x9c) >= 2` (a detail-quality setting).
Ghidra folds the displacement into a register, so the read is only visible in the disassembly. No
shipped mission script uses `actorlightnesson`, so the byte stays clear in the retail game. The
Mafia Script Data Reference describes the command as making *no* light affect the player; the
binary shows only the gate above.

² Type 0 = wall alarm box, type 1 = phone booth; the list is consumed by the cop "go to a phone and
call HQ" task. `game_get_telephone_list` (`0x00425570`) is the address accessor; its result is
handed to `vec_count_stride_0xc`, which reads `obj+4` / `obj+8` — that is what pins the header to
this pair.

³ `game_set_night_mission` is the engine side of the script command `game_nightmission nn`, which
the Mafia Script Data Reference describes as *"sets the mission to day or night; **affects car
lights** and shooting effects"*. `car_update`'s automatic-headlight block (`C_Vehicle+0x19c` bit
`0x10000`, `structs/vehicle.md` §1b) runs only while the flag is non-zero; `car_deactivate` clears a
car's script light-override bits only while it is ZERO.

**`+0x2bc` is not a field.** The decomp renders one of `game_tick`'s real vtable calls as
`(*(code *)**(game + 0x2bc))(dt)`, but a write-scan of the image finds no store to `C_game + 0x2bc`
and a disassembly dump of `game_tick` contains no `[reg+0x2bc]` read; the tick's real effect work
is the inlined vtable loop over `+0x2f8`.

`C_game::S_sound_play` is `0x18` bytes: `+0x00` the `I3D_sound *`, `+0x04` the play handle,
`+0x08` the target volume of a fade-in, `+0x0c` the signed fade rate per ms (positive fades in
toward `+0x08`, otherwise the sound fades out and stops at 0), `+0x10` the fading flag (byte) and
`+0x14` the last playback position, which shows when a sound has finished. `game_stop_3d_sound`
either stops a sound at once or sets the flag, stores `-rate` and increments `+0x310`.

**`g_net_session` (`DAT_006efbac`) is a different object.** It is a separate top-level
session object (`structs/net_session.md`). The two are easy to conflate because **both have
a meaningful `+0x204`**: here it is the taxi-passenger candidate actor, there it is the
embedded local multiplayer player record.

## 3. Embedded sub-objects

Every one of these is EMBEDDED, not a pointer, and none has a global of its own — which is
why their translation units contain no data symbols. Each receiver was byte-proven by a
census: every lifecycle callsite is `lea ecx,[reg + <off>]` inside `game_ctor` / `game_init`
/ `game_tick` / `game_done`, and every external producer reaches it as `[obj + 0x24] + <off>`
through the singleton accessor.

| offset | type | evidence | doc |
|---|---|---|---|
| `+0x4c` | **`G_Camera`** | all **13** callsites of the address accessor `game_get_camera` (`0x00425510`) load its result into ECX and immediately call a `G_Camera` method; one resolves to `camera_set_car` = `SetCar__8G_CameraFP5C_car`. Method: locate every `E8` callsite in `.text` and decode *the next call at each* | `structs/camera.md` — 36 of 39 PS2 methods, all body-diffed |
| `+0x208` | **`C_Break_plain_glass`** | 3 lifecycle callsites `lea ecx,[reg + 0x208]`; 7 producers — `car_break_glass`, `crash_obj_ai`, `door_hit`, `human_ai`, `rail_window_hit`, `game_tick_shoot`, `game_new_explosion`. Its `GameDone`-style method is `glass_shatter_done` | `structs/glass_shatter.md` |
| `+0x240` | **`C_Debris`** | 3 lifecycle callsites `lea ecx,[reg + 0x240]`; 4 producers for pieces (`car_explosion`, `car_on_collision_apply_damage`, `crash_obj_ai`, `human_ai`) and 4 for flashes (`airplane_hit`, `car_on_collision_apply_damage` ×2, `crash_obj_ai`). Its `GameDone`-style method is `debris_done` | `structs/debris.md` |
| `+0x26c` | **`C_compass`** | `lea ecx,[esi+0x26c]` in `game_init` and `lea ecx,[ebp+0x26c]` in `game_done` — the only two callsites of `compass_activate` | - |
| `+0x29ec` | **`C_using_object`** | every safe callsite of all nine members loads `[reg+0x24] + 0x29ec` — 45 for `using_object_del_object`, 33 for `_add_object`, 24 and 16 for the invalid-item pair. PS2 `+0xde0` | `structs/using_object.md` |
| `+0x2b3c` | **`soundArena`** | address accessor `game_get_sound_arena` (`0x00425550`) | `structs/soundArena.md` |
| `+0x2d90` | **`C_blood_manager`** | 12 of 12 callsites: `lea ecx,[this+0x2d90]` in `game_init` / `game_tick` / `game_done`, and `mov ecx,[display+0x24]; add ecx,0x2d90` from every `human_*` / `dog_*` / `pedestrian_hit` wound path; `game_ctor` / `game_dtor` construct and free it through the shared folds `folded_vector_ctor_at_0x04` / `folded_vector_free_at_0x04` | `structs/blood.md` |

Three more `C_game` features — per-object ambient shading, the shoot-light pool at `+0x2a48`
and the `I3D_light` cache at `+0x16c` — have their own document, `structs/lightness.md` (§6c).

## 4. Subsystems

### 4a. Shooting

`game_new_shoot` is the producer and `game_tick_shoot` (`TickShoot__6C_gameFUi`) is the
consumer. `game_tick_shoot` is 8036 bytes that touch no sound at all: it steps the
**`S_shoot`** record vector at `+0x1d8` (stride `0x5c`) — position `+0x00`, velocity `+0x0c`,
range `+0x18`, previous position `+0x1c`, damage `+0x28`, owner `+0x2c` — advancing each
record by `dt * 0.3`, ray-testing the travelled segment and applying damage.

`game_add_shoot_light(pos, range, lifetime_ms, flags)` claims the first of the ten slots at
`+0x2a48` whose lifetime has run out, sets the light's range to `(range * 0.5, range)`, moves it to
`pos`, links it into the scene, switches it on with an intensity of `rand × 0.4 + 0.8` and stores
the lifetime and the flags. The per-frame part is inlined into `game_tick` (§6a): each slot's
lifetime counts down, and at zero the light is unlinked and switched off; while it lasts, a slot
with flag bit 0 flickers (`rand × 0.4 + 0.8`) and fades linearly over its last 500 ms. A slot
without the flag keeps a steady light until it goes out.

| caller | range | lifetime | flags |
|---|---|---|---|
| `human_shooting` (muzzle flash) | 7.0 | 50 ms | 0 |
| `actor_shot_update` | 7.0 | 50 ms | 0 |
| `game_new_explosion` | 12.0 | 100 ms | 0 |
| `game_new_fire` | 10.0 | the fire's own lifetime | 1 |

The muzzle effect of `game_new_shoot` itself is a particle: `particle_manager_create_particle` with
the weapon's particle type at the muzzle frame. Shell casings come out of the two cartridge pools
(§4l).

### 4b. Temporary actors

Six functions over four containers: the two act-state buckets `+0xec` / `+0xfc`, the master
vector `+0x124`, and the deferred retirement queue `+0x134`.

| function | what it does |
|---|---|
| `game_add_temporary_actor` | the public add, with the register-virtual |
| `game_add_temporary_actor_intern` | the same without it |
| `game_remove_temporary_actor` | DEFERRED: set act-state 2 (spent) and queue onto `+0x134` |
| `game_remove_temporary_actor_intern` | swap-remove from the master vector and both buckets, unless queued for retirement; no virtuals |
| `game_remove_temporary_actor_now` | IMMEDIATE: swap-remove from the master vector and invalidate |
| `game_clear_temporary_actors` | empty the master vector, skipping two sentinel globals |

### 4c. The level-effect registry and the hydrants

`C_game + 0x2c8` is one stride-`0x18` vector of `{particle guid, covered flag, vec3 pos,
I3D_sound *}`. It has two producers and one destroyer:

* **level-placed effects** are read from `missions\%s\effects.bin` at scene load
  (`structs/particle.md` §5b);
* **`game_add_hydrant`** pushes a broken fire hydrant's water spray: particle type `0x57`,
  a driver sound object of type 4 carrying sound `0x147`, looped and attached to the frame.
  It returns immediately unless the detail level at `C_mission + 0xa8` is 2 or more and the
  frame is within 150 units of the camera. Its callers are `crash_obj_init` and
  `crash_obj_save_game_load` — hydrants are `C_crash_obj` instances;
* **`game_del_all_hydrants`** (callers `game_init` and `game_done`) destroys every record's
  particle, releases its sound object through the object's own virtual destructor and empties
  the vector.

`game_tick` walks the vector every frame and does three things per entry:

1. `particle_manager_is_particle_alive(guid)` — a dead effect is unlinked and swap-erased;
2. beyond **150 units** of the player (`22500.0` = 150²) the effect is destroyed outright;
3. a vertical ray is cast **upward**; while it hits anything whose owner is not a `C_car`,
   `particle_manager_set_particle_emitting(guid, 0)` stops emission, and it is switched back
   on with `1` when the sky is clear again.

In game: a street plume stops making new particles under a bridge or an awning, and a car
parked over it does not count. The particles already alive keep drawing.

### 4d. Traffic signals and callable devices

Six functions over `+0x1a8`: `game_load_traffic_signals`, `game_enum_traffic_signal_frames`
(its enumerator callback), `game_free_traffic_signals`, `game_tick_traffic_signals`,
`game_build_traffic_signal_model_idx` and `game_del_traffic_signal`. There is **no
single-record adder** (§6f): the list is built wholesale at scene load, exactly like the
roadblock and telephone lists, and `Del` is the only single-record mutator.

`game_enum_traffic_signal_frames` takes every `semafor.i3d` frame and looks up its child frames
`RED`, `ORANGE` and `GREEN` into the record's three sub-light slots. Each tick of the light clock
(every 3000 ms) `game_tick_traffic_signals` switches the three on or off from
`g_game_semaphore_states[phase]` = `[1,1,1,1,1,4,4,4,4,2]`: bit `1` red, bit `2` orange, bit `4`
green. A cycle is therefore 15 s red (phases 0..4), 12 s green (5..8) and 3 s orange (9). The two
phases start at 0 and 5 and advance together, so the two crossing axes show opposite colours. The
traffic AI reads the same phases to stop at a red light (`structs/driver.md` §3a).

`game_collect_telephone` matches frame model basenames against exactly two literals —
`alarm01.i3d` → type 0 (the wall alarm box) and `PHbox01.i3d` → type 1 (the phone box) — so
the "telephone" list holds **two** device kinds.

`game_collect_scream_area` accepts only frames named `enm_scrm_*` and parses each area's u16
mask out of the hexadecimal number in parentheses inside the frame name.

### 4e. Sound

`game_stop_all_sounds` (`StopAllSounds`), `game_pause_all_sounds` (`PauseAllSounds`) and
`game_play_all_sounds` (`PlayAllSounds`) are the whole-scene family; `game_play_3d_sound` /
`game_stop_3d_sound` are the per-sound pair over `+0x2f8`; `game_flush_pending_actor_lookups`
drains an actor-lookup queue and has nothing to do with audio.
`game_pause_scene_sounds_recursive` exists only on the PC. The fades of the `+0x2f8` records
(§2) are advanced by the sound-list walk inlined into `game_tick` (§6a), which also retires every
sound that has stopped playing.

### 4f. The three script stacks are one API

`game_inventory_stack_push` / `_pop` / `_clear`, `game_character_stack_push` / `_pop` /
`_clear` and `game_float_stack_push` / `_pop` / `_clear` are nine functions over three
vectors that share one first-use latch at `+0x2e4c`: the first push of any of them clears all
three. The inventory stack snapshots `C_human+0x480 .. +0x560` (the `G_Inventory`
sub-object) into a `0xe4`-byte heap copy; the character stack snapshots 16 dwords from
`C_human+0x640 .. +0x67c`, and its pop re-drives `human_force_death`,
`hud_set_player_health_value` and `hud_wingman_set_indicator_value_for_slot`.

### 4g. The human-talk queue

`game_human_talk_add(who, id)` pushes the pair onto `+0x2f44`. `game_human_talk_del(who, id)`
erases by the **speech id** and stops nothing. `game_human_talk_stop_for_human(who)` erases by
the **human** and calls `speech_channel_stop` on each entry it removes — that call is the
whole discriminator between the two erasers, and it is why the third is the one invoked from
`human_hit`, `human_movement`, `human_car_launched_into_air`, `human_throw_from_car_truck`
and `entity_force_ai`.

### 4h. City music, media and film music

The city-music block is `+0x2fb0 .. +0x2fc0` with its area vector at `+0x2fa4`;
`game_city_music_set_paused` gates it and calls `game_city_music_off`, which destroys the outgoing
track at once and fades the incoming one to silence over 5000 ms. The script arm that starts an
in-game music clip calls `game_city_music_off` too.

The media / in-game-FMV player is eleven functions over `+0x2f1c` (§1), with per-entry volume
re-applied against `g_music_volume` by `game_media_apply_volumes`.

`game_load_differences` (the script command `loaddifferences`, dispatcher case `0x43`) loads a
`.chg` scene-difference file from `Mafia\diff` through a stack `C_chunk`, and
`game_clear_differences` (`cleardifferences`) drops everything it added; the other consumer of
the loader is `race_winners_replay_begin`, with `difrace.chg` and `difracech.chg`.
`game_set_film_music` / `game_get_film_music` are `setfilmmusic` (case `0x149`) and
`getfilmmusic` (case `0x14a`), and `game_clear_film_music` runs at the tail of the cutscene
teardown, case `0x2e`.

### 4i. The scratch decal pool

An inline fixed array of 100 records at `+0x2fd8`, stride `0x10`. `game_scratch_add` stamps a
mark, `game_scratch_get_free_idx` allocates the slot in three passes, `game_scratch_recompile`
rebuilds the mesh and `game_scratch_delete_from_car` clears a car's marks (pool sizes: §6d).

### 4j. Freeride score

`+0x363e` is the display flag and `+0x3640` the value. `game_freeride_score_set_on` /
`game_freeride_score_set` write them and refresh the HUD; `game_freeride_score_get` is a
PC-only accessor consumed by the `FREERIDE_SCOREGET` / `SCOREADD` script commands. All three
are re-driven by `game_save_game_load`.

### 4k. Box placement

Sixteen functions over two vectors (`+0x2e84` boxes, `+0x2e94` destinations), one saved-info
vector (`+0x34`) and the player's carried-box path node (`+0x2ea0`). They are listed in §1,
and the PC names state what the bodies do (§6f).

### 4l. The two shell-casing pools

Both are ring buffers of `C_cartridge *` built by `game_init`, drained by `game_done` and
`game_done_intern`, and served by `game_new_cartridge`; nothing allocates at runtime.
Pool A is 100 entries at `+0x3a8` with its cursor at `+0x538` (model `2nabojnice.i3d`); pool B
is 30 entries at `+0x53c` with its cursor at `+0x5b4` (model `2nabojniceb.i3d`).
`structs/cartridge.md` §1a.

### 4m. Script-written scalars

Eight scalar fields the script VM writes: `+0x48`, `+0x164`, `+0x2ad9`, `+0x2b14`,
`+0x2d48 .. +0x2d67`, `+0x2edc` / `+0x2ee0`, `+0x3638` and `+0x3665`. Their meanings are in §2;
the cross-build check is in §6f.

## 5. The constructor and destructor

`game_ctor` (`0x00606750`) and `game_dtor` (`0x00606e20`) are **not** in `g_game.cpp`. They
sit in the `Mission.cpp` translation unit (§6f), out of line, and `mission_init` calls
`game_ctor` straight after `operator_new(0x3698)`.

The sub-object construction order is `G_Camera`, the 100-slot decal pool at `+0x75c`,
`C_rmorf`, `C_sledovacka`, `C_blood_manager` at `+0x2d90`, `C_shvestky`; it settles the
identification (§6f). `game_dtor` unwinds it in reverse, ending at `camera_dtor`, and also
frees the ItemBlur storage that `game_item_blur_done` only empties.

The two breakage sub-objects are destroyed through `game_subobject_0x208_dtor` and
`game_subobject_0x240_dtor`, and `game_done` calls their `GameDone`-style methods
(`lea ecx,[ebp+0x208]; call glass_shatter_done` and
`lea ecx,[ebp+0x240]; call debris_done`), which is what identifies them (§3).

## 6. PC vs PS2

### 6a. Inlined into `game_tick`

MSVC folded seven PS2 `C_game` methods into `game_tick`, so they have no separate PC function.
Each site is bracketed with an `// ===== INLINED ... =====` banner in the decomp.

| PS2 method | where it went | proof |
|---|---|---|
| `ShootLightTick__6C_gameFUi` | the shoot-light loop over `+0x2a48` | byte-verified; the PS2 walks a pool of 2 records, the PC its 10 |
| `TickEffects__6C_gameFUi` | the per-frame `S_sound_play` walk over `+0x2f8` | statement for statement: the timer gate bypassed while fades are running, the is-playing test, the playback-position test, the three-virtual retire with swap-pop, both fade branches. The sound virtuals sit `0x2c` higher on the PS2 and the inherited frame virtuals `0x14` higher |
| `TickCityMusic__6C_gameFUi` | the `+0x2fb0` block | bannered |
| `TickShadows__6C_gameFv` | - | an empty PS2 stub; nothing to inline |
| `LightCacheTick__6C_gameFUi` | absorbed whole (388 bytes) into `game_recompute_light_cache` — a 2-to-1 fold | bannered |
| `SetActiveTrafficP__6C_gameFUi` | the traffic-generator pick over `+0x18c` / `+0x190`; `game_init` runs a second copy with the period plus one, which forces a pick on the first frame | statement for statement: the kind-`0x12` scan, the in-range-first then nearest choice, `SetActState(0)` and the selected byte |
| `TickDriverMan__6C_gameFv` | the steering-wheel loop over `+0x2d7c`: frame = (current steer `car+0x714` / max steer `car+0x6fc` + 1) × 0.5 × the wheel animation's frame count `+0x2d88` / `+0x2d8c` | statement for statement, gated on the detail setting |

`game_tick` also folds in per-frame blocks that are its OWN logic rather than separate
methods: actor visibility (`+0x180`), a `+0x16c` distance cull, the level-effect registry sweep (`+0x2c8`), the periodic scream-area pass
(`+0x361c`) and the fake-shadow LOD pass over `+0x290` / `+0x2a0`, which no PS2 method performs
(`structs/fake_shadow.md` §7).

### 6b. The 29 PS2 `C_game` methods with no located PC counterpart

| group | methods | reading |
|---|---|---|
| inlined into `game_tick` (§6a) | `ShootLightTick`, `TickEffects`, `TickCityMusic`, `TickShadows`, `LightCacheTick`, `SetActiveTrafficP`, `TickDriverMan` | a factoring difference, not an absence |
| inlined into `game_init` | `PreloadAnims` | the heavy-init block's clip preload and base-track seed, statement for statement (`structs/anim_machine.md` §4b); the PS2 calls it once from `InitSystem` |
| InterCity city streaming | `AddInterCityArea`, `ClearInterCityAreas`, `TickInterCity`, `InterCityNajeb`, `InterCityVyjeb`, `VyhjebejTemporaryho` | PS2-port-only and TERMINAL. `InterCity` appears 0 times in the PC decomp; the PS2 added city-part streaming for the console's 32 MB, and `VyhjebejTemporaryho` exists only to move a temporary actor onto the mission-global list through `InterCity_AddGlobalActor` so it survives a city-part load |
| FQL actor loading | `AddActorForFQL`, `CreateActorsFQL` | the same family as `C_mission`'s FQL path methods, which the PC also lacks |
| death-human fade | `AddDeathHumanFade`, `ClearDeathHumanFade`, `TickDeathHumanFade` | the PS2's alternative to the PC's part arena (`structs/dropout.md`) |
| fire flames | `AddFireFlame`, `TickFireFlames` | replaced: at the `AddFireFlame` call position in `NewShoot` the PC builds the same `(0,0,0)` / `(0,0,1)` locals and calls `particle_manager_create_particle` with the weapon's muzzle particle, so nothing on the PC keeps or ticks a flame ring |
| shadows for people | `AddFakeShadow`, `DelFakeShadow`, `TickFakeShadows` | PS2-only: a second shadow system, for people, whose calls the PC twins of every caller drop (`structs/fake_shadow.md` §7) |
| active-actor bookkeeping | `AddNewActorToActive`, `DelNewActorFromAll`, `IsActorInActList` | PS2-port-only: every PS2 caller is InterCity code, the PS2-only rail generator (entity kind `0x26`), or the freeride setup tail whose PC twin `loadsave_freeride_setup_scene` ends at `mission_add_actor`; `IsActorInActList` has no caller |
| singles | `TickHydrant`, `IsFramePlacement` | `IsFramePlacement` (92 bytes) is PS2-port-only: its one caller is the PS2-only `S_auto_hide_machine` (`structs/camera.md` §6d). `TickHydrant` is 864 real bytes on the PS2 doing distance culling and pause/resume of the hydrant particles; the PC has `game_add_hydrant` and `game_del_all_hydrants` and nothing between them, because `game_tick` does that work inline over `+0x2c8` (§4c) |

### 6c. Gutted on the PS2, shipped on the PC

Where the usual direction reverses, the PC body is the only one that exists and there is
nothing to diff against:

* `AddShootLight__6C_gameFR8S_vectorfiUs` and
  `NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` are 8-byte `return;` stubs, so no muzzle
  light is added and no casing is ever ejected on the PS2 — although it keeps both cartridge
  pools and `ClearCartridges__6C_gameFv`;
* `TickShadows`, `SetFrameForShadow` and `DelFrameForShadow` are `return` / `return 0` stubs;
* the whole lightness band (`structs/lightness.md`) is stubbed there — per-object ambient
  shading, the shoot-light pool at `+0x2a48` and the `I3D_light` cache at `+0x16c` — so that band
  is where the PS2 method count is a LOWER bound rather than an upper one.

### 6d. Container and layout differences

* **`ItemBlurDone`:** the PC clears and RETAINS the vector's capacity; the PS2 frees it. The
  PC storage free lives in `game_dtor` instead.
* **Pool sizes:** cartridges 100 + 30 on PC against 10 + 5 on PS2; scratch decals 100 against
  20; the level-effect record is `0x18` bytes on PC and `0x20` on PS2.
* **Inlined `std::vector<T>::insert`:** five functions (`game_human_talk_add`,
  `game_add_invisible_actor`, `game_dying_human_add`, `game_add_created_frame`,
  `game_collect_scream_area`) inline an insert the PS2 calls out of line, and that alone is
  the entire 3–8× size difference against their PS2 twins — a fact about the toolchain, not
  about the functions. Three erasers (`game_human_talk_del`,
  `game_human_talk_stop_for_human`, `game_del_traffic_signal`) are bannered for the opposite
  reason: the PS2 calls a `compressed_pair_imp<>::first` allocator hook there that MSVC's
  `std::vector` does not have.
* **Folds the other way:** `game_use_light_cache_system` delegates a walk the PS2 does inline,
  and `game_pause_scene_sounds_recursive` exists only on the PC.

### 6e. The COMDAT container tail

`[0x005f6770, 0x005f79a0)` holds 28 out-of-line copies of header inlines, emitted into this
object file because this is where they were used. A branch-aware receiver census over all 28
puts them on many unrelated owners — `std_vector4_insert_at` alone has **55** callsites across
about 30 — so none of them may carry a class-qualified name.

### 6f. Cross-build identification notes

* **Singleton and code span:** the PS2 singleton is `*(int *)(act_mission + 0x20)` (PC `+0x24`),
  and PS2 `C_game` code spans `0x002ea010`–`0x00306d70`.
* **Traffic signals (§4d):** there is no `AddSemaphore` on either build.
* **Broken models (§2):** PS2 `CreateCrashModel` pushes the broken model onto `C_game+0x10` and
  switches it off, exactly as `crash_obj_create_from_model` does on the PC.
* **City music (§4h):** `CityMusicOff` releases its two channel objects at once; the PC destroys
  the outgoing track and fades the incoming one. Both PC callers sit where the PS2 calls it — the
  music-clip script arm, statement for statement, and `CityMusicPaused` — so the twin is
  STRUCTURAL. The PS2 `Done` calls it as well; `game_done` has no call at that position.
* **Box placement (§4k):** three of the PS2 verbs (`GetBox`, `GetBoxDest`, `GetNextDest`)
  describe neither the return value nor the action of the twin bodies.
* **Script-written scalars (§4m):** each is body-diffed against the PS2 `C_program::Process` arm
  for the same command; the PS2 offsets are in §2.
* **Constructor and destructor (§5):** both builds place them in `Mission.cpp`. The PS2 has no
  `__ct__6C_gameFv`; it inlines the identical constructor into `Init__9C_missionFP10I3D_driver`
  and the identical destructor into `UnInit__9C_missionFv`, and its
  `std::__vector_deleter<C_game::S_*>` helpers are emitted in the same PS2 band. The PC
  sub-object construction order is the same sequence the PS2 emits inline, which settles the
  identification without a symbol to diff against.

## 7. Open items

Nothing is open.
