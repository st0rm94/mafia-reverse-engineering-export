# The ambient pedestrian crowd — `C_traffic_element` + `C_traffic_generator` + `C_don_ocas`

**Created** 2026-08-14 17:51 +0200 · **Modified** 2026-09-24 12:22 +0200

| | |
|---|---|
| PC translation unit | `[0x004b0d30, 0x004be810)` — 86 real starts, all named |
| PS2 file | `C_don_ocas.cpp` |
| PC prefix `pedestrian_` | `C_traffic_element` — the pedestrian record (§2) |
| PC prefix `pedestrian_manager_` | `C_traffic_generator` — the spawner, entity kind `0x12` (§3) |
| PC prefix `pedestrian_crowd_` | `C_don_ocas` — the crowd's proxy actor, entity kind `0x13` (§4) |

The PC prefixes state what each class does; the developers' own class names live in the
`PS2 method` column of §6. `C_traffic_generator` generates **only pedestrians** — the
ambient **car** traffic is `C_traffic_car` (entity kind `0xc`, family `traffic_car_*`,
`structs/traffic_car.md`). The class globals keep the `g_traffic_*` spelling (§5).

⚠️ `pedestrian_` is a proper prefix of **both** `pedestrian_crowd_` **and**
`pedestrian_manager_`, so any name filter on `pedestrian_` picks up all three classes.

## 1. Why there are three classes

**`C_traffic_element` is the ambient pedestrian.** It is *not* an entity: it has **no
vtable** (`pedestrian_ctor` installs none), **no `kind` byte**, and it never appears in
the global actor list. It is a plain **`0x200`-byte record** living in an arena the
generator allocates.

**`C_don_ocas` is the crowd's stand-in ACTOR, and there is exactly one instance.**
`game_init` does `operator_new(0x70)` → `pedestrian_crowd_ctor` and stores the result at
**`C_game + 0x2a1c`**. `pedestrian_set_model` then stamps that singleton as the owner of
*every* pedestrian's `I3D_model`:

```
operator_new(4); *cell = *(C_game + 0x2a1c); model->+0x08 = cell;
```

So every engine-level collision, hit or bounds query aimed at a pedestrian model is
dispatched to the one `C_don_ocas`, whose four real methods either sweep the whole list
(`pedestrian_crowd_is_possible_coll`) or call
`pedestrian_manager_get_pedestrian_from_frame` to find *which* pedestrian was involved and
forward to that record. `pedestrian_destroy_model` clears the cell again.

**Why the split exists:** a real `C_actor` is heavyweight — the cop entity this very TU
spawns is `operator_new(0x128c)`, ten times a pedestrian's `0x200` — and every actor is
ticked, saved and replicated. A crowd of dozens could not afford that, so the crowd is an
array of light records with **one** actor in front of it.
`pedestrian_manager_spawn_cop_from_pedestrian` is the seam: when an ambient pedestrian
must become a real participant, it deactivates the record and allocates a full
`C_entity`.

**`C_traffic_generator`** (entity kind `0x12`, vtable `0x0063c778`) is the scene-placed
spawner: it owns the arena, the designer model-name table, the population budget, the
save/load path and the taxi feature. Two of its methods look like car work and are not:
`pedestrian_manager_railway_in_station` iterates `g_pedestrian_list` to fill a station
platform, and the four taxi methods are pedestrians hailing and riding taxis.

### 1a. Every element is a humanoid — measured

* `pedestrian_create_model` attaches an `I3D_pose` to the frame **`neck`** and reads a
  two-digit height scale off the user-property tag of the frame **`back1`** (×0.01) —
  **unconditionally, for every element**.
* Every element owns a `C_animations_machine_de_luxe`, taken from the parallel pool
  `g_pedestrian_anim_list`.
* The only creation paths are `pedestrian_activate` (from the generator's model-name
  table) and `pedestrian_activate_free` (from `pedestrian_manager_add_free_person`,
  `_add_free_person_from_car`, `_generate_passenger_for_taxi`).
* The behaviour set is human-only: `pedestrian_drop_umbrella`, `pedestrian_wave_at_taxi`,
  `pedestrian_greet_player`, `pedestrian_on_player_talk`,
  `pedestrian_spawn_gang_on_hit`.
* Ambient **cars** are a different class entirely — `C_traffic_car`, kind `0xc`, its own
  `traffic_car_*` family.

The one "special" element type is a **foot cop** (`+0x15a` ∈ {1, 3}) — still a
pedestrian, and the population `pedestrian_manager_spawn_cop_from_pedestrian` draws from.

## 2. `C_traffic_element` — the pedestrian record (PC, stride `0x200`)

| offset | field | conf | writer | reader |
|---|---|---|---|---|
| `+0x00` | `is_player_controlled` (byte) | H | `pedestrian_ctor` | every list walk skips it |
| `+0x04` | owning generator / free flag | H | `pedestrian_ctor`, `pedestrian_create_model` | `pedestrian_destroy_model` decides destroy-vs-return |
| `+0x08` | `anim_machine*` | H | `pedestrian_create_model` | `pedestrian_end_taxi_wave`, `pedestrian_wave_at_taxi` |
| `+0x0c` | `active` (byte) | H | `pedestrian_activate` / `_deactivate` | every list walk gates on it |
| `+0x10..+0x18` | world position | H | `pedestrian_tick` | the whole TU |
| `+0x1c..+0x24` | motion direction | H | `pedestrian_tick` | `pedestrian_crowd_is_possible_coll` |
| `+0x40` | `I3D_model*` | H | `pedestrian_set_model` | everywhere |
| `+0x48` | cull distance (`90.0` seeded) | M | `pedestrian_set_model` | - |
| `+0x54` | squared distance to player | H | `pedestrian_manager_tick_pedestrians` | `pedestrian_tick` |
| `+0x60`/`+0x64`/`+0x68`/`+0x6c`/`+0x70`/`+0x7c` | previous / current path node id, node position, node speed | M | `pedestrian_set_panic`, `pedestrian_advance_to_next_node` | idem |
| `+0x80..+0x88` | current move target | H | `pedestrian_advance_to_next_node`, `pedestrian_manager_release_pedestrians_from_car` | `pedestrian_tick` |
| `+0x98..+0xa0` | saved move target (restored on car release) | H | `pedestrian_activate*` | `pedestrian_manager_release_pedestrians_from_car` |
| `+0xb0` | path state / node countdown (`< 0` = no path) | H | `pedestrian_advance_to_next_node` | `pedestrian_avoid_obstacle`, the car-release map |
| `+0xb8` | child frame selected by the `'SNGM'` / `'SMMR'` magic | M | `pedestrian_create_model` | - |
| `+0xc4..+0x10c` | 0x12 dwords zeroed by the ctor | U | `pedestrian_ctor` | - |
| `+0xd4..+0xdc` | cached head point (frame origin, Y `+0.9`) | H | `pedestrian_update_model` | - |
| `+0x15a` | category — `0`=civilian, `1`=patrol cop, `3`=mission cop, `4`=U | H | `pedestrian_activate` | `pedestrian_manager_ai`, `pedestrian_set_panic`, `pedestrian_manager_spawn_cop_from_pedestrian` |
| `+0x15c` | model scale (`1.0` default, `back1` tag overrides) | H | `pedestrian_create_model` | `pedestrian_set_mega_panic` gate `>= 0.1` |
| `+0x164`/`+0x168` | panic animation id pair | H | `pedestrian_set_mega_panic` | the anim machine |
| `+0x184` | `I3D_pose*` on the `neck` frame | H | `pedestrian_create_model` | `pedestrian_destroy_model` |
| `+0x188` | look-at target frame (the player's `neck`) | H | `pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn` | `pedestrian_tick` |
| `+0x190` | report delay, `3000` ms | M | `pedestrian_advance_to_next_node` | - |
| `+0x19c` | `panicking` (byte) | H | `pedestrian_set_panic` | `pedestrian_manager_try_send_pedestrian_to_parked_car` |
| `+0x19e` | panic-once latch | H | `pedestrian_set_panic` | `pedestrian_manager_is_car_claimed_by_pedestrian` |
| `+0x1a0` | panic expiry, `rand + 20000` ms | H | `pedestrian_set_panic` | - |
| `+0x1a4` | mega-panic level 1..3 | H | `pedestrian_set_mega_panic` | - |
| `+0x1a5` | already-activated latch | M | `pedestrian_manager_spawn_cop_from_pedestrian` | idem |
| `+0x1a8` | assigned car | H | the taxi/boarding paths | `pedestrian_manager_is_car_claimed_by_pedestrian`, `_release_pedestrians_from_car` |
| `+0x1b8` | witness actor (drives the police report) | H | `pedestrian_tick` | `pedestrian_advance_to_next_node` |
| `+0x1bc` | path-node handle | H | `pedestrian_activate*` | `pedestrian_destroy_model` releases it |
| `+0x1fc` | greeted latch | H | `pedestrian_greet_player` | `pedestrian_on_player_talk` |
| `+0x1fd` | greeting-in-progress | H | `pedestrian_greet_player` | `pedestrian_on_player_talk` |
| `+0x1fe` | hailing a taxi | H | `pedestrian_wave_at_taxi` | `pedestrian_manager_end_all_taxi_waves` |

Conf: **H** measured on ≥2 independent sites · **M** one site, unambiguous · **U**
unproven · `-` no reader located.

## 3. `C_traffic_generator` — the spawner (PC, entity kind `0x12`)

| offset | field | conf | note |
|---|---|---|---|
| `+0x10` | `kind` = `0x12` | H | `pedestrian_manager_ctor` |
| `+0x70` | enabled (byte, `1`) | H | designer, saved |
| `+0x74` | spawn radius `100.0` | H | designer, saved |
| `+0x78` | visible (byte) | H | `pedestrian_manager_game_init` clears it |
| `+0x7c` | pedestrian arena base (stride `0x200`) | H | `pedestrian_manager_game_init` allocates |
| `+0x80` | live pedestrian count | H | budget comparison in `_ai` |
| `+0x84` | arena slot count | H | sum of the per-name spawn counts |
| `+0x88`/`+0x8c`/`+0x90`/`+0x94` | designer floats `100.0` / `50.0` / `90.0` / `50.0` | H | saved; individual meanings unproven |
| `+0x98` | model-name array (`char*`, each `0x11` bytes) | H | designer table |
| `+0x9c` | model-name count | H | idem |
| `+0xa0` | per-name spawn-count array (dwords) | H | idem |
| `+0xa4` | spawn accumulator (ms) | H | `_ai` compares against `0x70` |
| `+0xa8` | population budget (`10` seeded, scaled by `g_traffic_ratio_all`) | H | - |
| `+0xac` | designer budget, saved across the scene | H | restored by `_game_done` |
| `+0xb0` | round-robin spawn cursor | H | `_ai` |
| `+0xb4` | cop-slot boundary index | M | `_ai` restricts categories 1/3 to slots ≤ this |

**Savegame record:** `0xbc` bytes per live pedestrian, plus a 10-byte header
(`pedestrian_manager_save_game_get_size`).

### 3a. The taxi model tables — the same list on both builds, PS2 has two more

`pedestrian_manager_player_in_taxi` sets `g_traffic_player_in_taxi` when the player's
current car's **model name** (`car+0x20cc`; PS2 `car+0x240c` — an already-proven field
pair) case-insensitively matches one of a hard-coded table of model names. The flag's
only reader is `pedestrian_manager_tick_pedestrians`, `&&`-ed with
`g_traffic_taxi_driver_enabled`, and it is what allows a passenger to be generated.

Both tables were resolved from the binaries (PC `.data` `0x0064c060..0x0064c0c0`;
PS2 pointer table at `0x00495b30`) and they are the same list in the same order:

| # | PC (9 entries) | PS2 (11 entries) |
|---|---|---|
| 1 | `taxi` | `taxi` |
| 2 | `hotrodp4` | `hotrodp4` |
| 3 | `hotrodp5` | `hotrodp5` |
| 4 | `tbirdold` | `tbirdold` |
| 5 | `fordhot` | `fordhot` |
| 6 | `phantomtaxi` | `phantomtaxi` |
| 7 | `hotrodp2` | `hotrodp2` |
| 8 | `fordapicktaxi` | `fordapicktaxi` |
| 9 | `blackdragon` | `blackdragon` |
| 10 | - | **`hotrodp3`** |
| 11 | - | **`thunderbird`** |

Both loops walk the table **downwards** from the last entry. Identical models in
identical order with two extra PS2 rows is as strong as a shared-magic-constant match
gets, and it is what grades the `PlayerInTaxi` twin **CONFIRMED**.

⚠️ The list is **not** "the taxis": it is the set of models the passenger generator will
accept. Six of the nine are not taxis at all. What the hot-rods and `blackdragon` are
doing in it is **not established** — do not infer a reason from the name.

🎮 **A checkable gameplay difference:** on PS2 a `hotrodp3` or a `thunderbird` counts for
the Free-Ride passenger generator and on PC neither does. One in-game test settles it.

## 4. `C_don_ocas` — the crowd's stand-in actor (PC, entity kind `0x13`)

Instance size `0x70`; vtable `0x0063c6d8` (39 slots); one instance, at `C_game+0x2a1c`.
Only four of its methods do anything — the rest of the vtable is the inherited `C_actor`
default set. It owns no pedestrian state at all; `+0x65` is the only field it writes.

| PC method | what the engine uses it for |
|---|---|
| `pedestrian_crowd_is_possible_coll` | swept-sphere query against the **whole** crowd |
| `pedestrian_crowd_collision` | resolve the struck pedestrian, forward, register the offences |
| `pedestrian_crowd_hit` | resolve the struck pedestrian, forward to `pedestrian_hit` |
| `pedestrian_crowd_get_bound_points_2d` | a `0.7`-half-width diamond around one pedestrian, for navigation |

## 5. File-scope data

| address | name | what it is |
|---|---|---|
| `0x0064c058` | `g_traffic_ratio_all` | population multiplier applied to the designer budget |
| `0x00656084` | `g_pedestrian_scream_enabled` | the crowd's shout permission — a STATIC MEMBER of `C_traffic_generator` ¹ |
| `0x00656085` | `g_traffic_player_in_taxi` | set by `pedestrian_manager_player_in_taxi` |
| `0x00656086` | `g_traffic_taxi_driver_enabled` | sole writer `pedestrian_manager_enable_taxi_driver` |
| `0x00656088` | `g_traffic_taxi_timer_ms` | countdown to the next taxi passenger |
| `0x0065608c` | `g_traffic_all_visible` | master crowd visibility; gates `_ai`, `_tick_pedestrians`, `_explosion`, `_fire` |
| `0x00656090` | `g_traffic_parked_car_scan_timer_ms` | resets to `0x84b` ms |
| `0x00656094` | `g_traffic_horn_timer_ms` | decremented at the head of `_tick_pedestrians` |
| `0x00656098` | `g_traffic_scream_budget` | `> 100` refuses a scream; `+= 500` per scream |
| `0x0065609c`/`a0`/`a4` | `g_traffic_last_panic` | written by `_new_panic_at_position`, read by `pedestrian_set_panic` |
| `0x006560a8` | `g_traffic_first_frame` | the extra gate inside `_ai` |
| `0x006560ac..b8` | `g_pedestrian_anim_list*` | pool of `C_animations_machine_de_luxe`, one per pedestrian |
| `0x006560bc` | `g_traffic_static_dtor_guard` | bits 1 and 2 guard the two static destructors |
| `0x006560c0..cc` | `g_pedestrian_list*` | the global `std::vector<C_traffic_element*>` of live pedestrians |

¹ It gates `pedestrian_manager_play_pedestrian_scream`, and the script command
`npc_shutup` writes it through `pedestrian_manager_set_scream_enabled` with **INVERTED**
polarity (`flag = argument == 0`), byte-verified on both builds and agreeing with the
Mafia Script Data Reference (*0 — the pedestrian will talk*). It is not a sound master
switch.

## 6. PC ↔ PS2 coverage (PC VA order, PC name first)

### 6a. `C_don_ocas` — 7/7

| PC VA | PC name | PS2 method | grade |
|---|---|---|---|
| `0x004b0e50` | `pedestrian_crowd_ctor` | `__ct__10C_don_ocasFv` | CONFIRMED |
| `0x004b0e70` | `pedestrian_crowd_scalar_deleting_dtor` | - | PC-only (MSVC sdd) |
| `0x004b0e90` | `pedestrian_crowd_dtor` | `__dt__10C_don_ocasFv` | CONFIRMED |
| `0x004b0ea0` | `pedestrian_crowd_game_init` | `GameInit__10C_don_ocasFv` | CONFIRMED |
| `0x004b0eb0` | `pedestrian_crowd_is_possible_coll` | `IsPossibleColl__10C_don_ocasF…` | CONFIRMED |
| `0x004b1260` | `pedestrian_crowd_collision` | `Collision__10C_don_ocasF…` | CONFIRMED |
| `0x004b1500` | `pedestrian_crowd_hit` | `Hit__10C_don_ocasF…` | CONFIRMED |
| `0x004b19d0` | `pedestrian_crowd_get_bound_points_2d` | `GetBoundPoints2D__10C_don_ocasF…` | CONFIRMED |

### 6b. `C_traffic_generator` — 39 mapped, 49/49 accounted

| PC VA | PC name | PS2 method | grade |
|---|---|---|---|
| `0x004b1c20` | `pedestrian_manager_ctor` | `__ct__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b1c90` | `pedestrian_manager_scalar_deleting_dtor` | - | PC-only |
| `0x004b1cb0` | `pedestrian_manager_dtor` | `__dt__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b1d20` | `pedestrian_manager_duplicate_from` | `DuplicateFrom__19C_traffic_generatorFP7C_actor` | CONFIRMED |
| `0x004b1e80` | `pedestrian_manager_save_data` | `SaveData__19C_traffic_generatorFP7C_chunk` | CONFIRMED |
| `0x004b2030` | `pedestrian_manager_load_data` | `LoadData__19C_traffic_generatorFP7C_chunk` | CONFIRMED |
| `0x004b2610` | `pedestrian_manager_save_game_get_size` | `SaveGameGetSize__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b2630` | `pedestrian_manager_save_game_save` | `SaveGameSave__19C_traffic_generatorFPPv` | CONFIRMED |
| `0x004b2930` | `pedestrian_manager_save_game_load` | `SaveGameLoad__19C_traffic_generatorFPPv` | CONFIRMED |
| `0x004b2d10` | `pedestrian_manager_game_init` | `GameInit__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b3000` | `pedestrian_manager_game_done` | `GameDone__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b3080` | `pedestrian_manager_player_talked_to_pedestrian` | `PlayerPromluvilNaTehoChuja__19C_traffic_generatorFP9I3D_frame` | CONFIRMED |
| `0x004b30a0` | `pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn` | `PlayerHoMaVytaseneho__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b3170` | `pedestrian_manager_set_visible` | `SetVisible__19C_traffic_generatorFb` | CONFIRMED |
| `0x004b31d0` | `pedestrian_manager_is_possible_coll_pedestrian` | `IsPossibleCollElem__19C_traffic_generatorFP17C_traffic_element` | CONFIRMED |
| `0x004b33f0` | `pedestrian_manager_add_pedestrian` | `AddElement__19C_traffic_generatorFP17C_traffic_element` | CONFIRMED |
| `0x004b3510` | `pedestrian_manager_del_pedestrian` | `DelElement__19C_traffic_generatorFP17C_traffic_element` | CONFIRMED |
| `0x004b35c0` | `pedestrian_manager_ai` | `AI__19C_traffic_generatorFUi` | CONFIRMED |
| `0x004b3780` | `pedestrian_manager_tick_pedestrians` | `TickElements__19C_traffic_generatorFUi` | CONFIRMED |
| `0x004b38a0` | `pedestrian_manager_is_car_claimed_by_pedestrian` | `IsCarHasOcas__19C_traffic_generatorFP5C_car` | CONFIRMED |
| `0x004b38f0` | `pedestrian_manager_try_send_pedestrian_to_parked_car` | `TryGoToAssHole__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b3f50` | `pedestrian_manager_player_in_taxi` | `PlayerInTaxi__19C_traffic_generatorFP5C_car` | CONFIRMED |
| `0x004b3fe0` | `pedestrian_manager_generate_passenger_for_taxi` | `GeneratePassengerForTaxi__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b4500` | `pedestrian_manager_enable_taxi_driver` | `EnableTaxiDriver__19C_traffic_generatorFb` | CONFIRMED |
| `0x004b4520` | `pedestrian_manager_end_all_taxi_waves` | `ZrusVsechnyMavajici__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b4560` | `pedestrian_manager_clear_pedestrians` | `ClearElements__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b45f0` | `pedestrian_manager_spawn_cop_from_pedestrian` | `FizlosNamakatDoAkce__19C_traffic_generatorF…` | CONFIRMED |
| `0x004b4c60` | `pedestrian_manager_explosion` | `Explosion__19C_traffic_generatorFR8S_vectorff` | CONFIRMED |
| `0x004b4f90` | `pedestrian_manager_fire` | `Fire__19C_traffic_generatorFR8S_vectorff` | CONFIRMED |
| `0x004b5040` | `pedestrian_manager_get_pedestrian_from_frame` | `GetElemFromFrame__19C_traffic_generatorFP9I3D_frame` | CONFIRMED |
| `0x004b5070` | `pedestrian_manager_deactivate_all_pedestrians` | `FreeDynCollElements__19C_traffic_generatorFv` | CONFIRMED |
| `0x004b50b0` | `pedestrian_manager_get_nearest_pedestrian_dist_sq` | `GetNearElementDist2__19C_traffic_generatorFR8S_vector` | CONFIRMED |
| `0x004b5130` | `pedestrian_manager_find_pedestrian_ahead` | `IsNearElementBefore__19C_traffic_generatorFP17C_traffic_element` | CONFIRMED |
| `0x004b51e0` | `pedestrian_manager_new_panic_at_position` | `NewPanic__19C_traffic_generatorFR8S_vector` | CONFIRMED |
| `0x004b5240` | `pedestrian_manager_new_panic_at_frame` | `NewPanic__19C_traffic_generatorFP9I3D_frame` | CONFIRMED |
| `0x004b5270` | `pedestrian_manager_railway_in_station` | `RailwayInStation__19C_traffic_generatorF…` | CONFIRMED |
| `0x004b5710` | `pedestrian_manager_add_free_person` | `AddFreePerson__19C_traffic_generatorF…` | CONFIRMED |
| `0x004b5790` | `pedestrian_manager_add_free_person_from_car` | `AddFreePersonFromCar__19C_traffic_generatorF…` | CONFIRMED |
| `0x004b58c0` | `pedestrian_manager_play_pedestrian_scream` | `ElementNewScream__19C_traffic_generatorFiiR8S_vectorffff` | CONFIRMED |
| `0x004b5ab0` | `pedestrian_manager_release_pedestrians_from_car` | `VyfunOdTeMasiny__19C_traffic_generatorFP5C_car` | CONFIRMED |
| - | - | `Init` (32 B, tail-calls the base) | the PC vtable slot points straight at `actor_init` |
| - | - | `IsPossibleColl` · `Collision` · `Hit` · `Update` · `NetDirect` · `DetachMesh` (8 B each) | folded onto shared PC stubs |
| - | - | `ChangeSceneInit` · `ChangeSceneDone` | PS2-port-only (city streaming) |
| - | - | `InvalidateActor` | not located on the PC (§8) |

### 6c. `C_traffic_element` — 22/26

| PC VA | PC name | PS2 method | grade |
|---|---|---|---|
| `0x004b5bb0` | `pedestrian_ctor` | `__ct__17C_traffic_elementFv` | CONFIRMED |
| `0x004b5c10` | `pedestrian_avoid_obstacle` | `ObejdiTuPicovinu__17C_traffic_elementF…` | CONFIRMED |
| `0x004b60b0` | `pedestrian_advance_to_next_node` | `SetNewNode__17C_traffic_elementFv` | CONFIRMED |
| `0x004b6840` | `pedestrian_wave_at_taxi` | `MavaNaTaxi__17C_traffic_elementFUi` | CONFIRMED |
| `0x004b74f0` | `pedestrian_end_taxi_wave` | `EndMavaniNaTaxi__17C_traffic_elementFv` | CONFIRMED |
| `0x004b7520` | `pedestrian_tick` | `Tick__17C_traffic_elementFUi` | CONFIRMED |
| `0x004bacf0` | `pedestrian_update_model` | `UpdateModel__17C_traffic_elementFv` | CONFIRMED |
| `0x004bae30` | `pedestrian_set_model` | `SetModel__17C_traffic_elementFP9I3D_model` | CONFIRMED |
| `0x004bb240` | `pedestrian_create_model` | `CreateModel__17C_traffic_elementFPcP19C_traffic_generator` | CONFIRMED |
| `0x004bb6a0` | `pedestrian_set_alpha` | `SetAlpha__17C_traffic_elementFf` | CONFIRMED |
| `0x004bb6c0` | `pedestrian_destroy_model` | `DestroyModel__17C_traffic_elementFv` | CONFIRMED |
| `0x004bb7a0` | `pedestrian_activate_free` | `ActivateFree__17C_traffic_elementFbb` | CONFIRMED |
| `0x004bbff0` | `pedestrian_activate` | `Activate__17C_traffic_elementFbff` | CONFIRMED |
| `0x004bcfc0` | `pedestrian_deactivate` | `Deactivate__17C_traffic_elementFv` | CONFIRMED |
| `0x004bd0d0` | `pedestrian_hit` | `Hit__17C_traffic_elementF10E_hit_type…` | CONFIRMED |
| `0x004bdc50` | `pedestrian_set_anim` | `SetAnim__17C_traffic_elementFv` | CONFIRMED |
| `0x004bde50` | `pedestrian_set_panic` | `SetPanic__17C_traffic_elementFv` | CONFIRMED |
| `0x004bdfb0` | `pedestrian_set_mega_panic` | `SetMegaPanic__17C_traffic_elementFi` | CONFIRMED |
| `0x004be050` | `pedestrian_on_player_talk` | `PlayerNaMneMluvi__17C_traffic_elementFv` | CONFIRMED |
| `0x004be0d0` | `pedestrian_greet_player` | `Karle_hoj__17C_traffic_elementFv` | CONFIRMED |
| `0x004be250` | `pedestrian_drop_umbrella` | `DropUmbrella__17C_traffic_elementFv` | CONFIRMED |
| `0x004be390` | `pedestrian_spawn_gang_on_hit` | `CreateMafianos__17C_traffic_elementF10E_hit_type…` | CONFIRMED |
| - | - | `__dt__17C_traffic_elementFv` (68 B) | trivial; no PC start in the gap |
| - | - | `HitFall` · `HitGun` · `HitStroke` (file-local) | most likely inlined into `pedestrian_hit` (§8) |

### 6d. Foreign helpers emitted inside the TU

`vec2_length`, `vec2_dot` (shared math) and the `C_chunk` tail — `chunk_vector_allocate`
(vtable `0x0063c828` slot 0), `chunk_ctor`, `chunk_open_file`
(`OpenFile__7C_chunkFPCcb`), `chunk_close` (`Close__7C_chunkFv`). They land here because
`pedestrian_wave_at_taxi` builds a stack `C_chunk` to read `system\taxi_pas.dat`.

## 7. PC vs PS2

Everything cross-build lives here.

| topic | PC | PS2 |
|---|---|---|
| `C_traffic_element` size | `0x200` | `0x250` |
| element field base | `+0x00` | `+0x180` (a 0x180-byte prefix the PC lacks) — ⚠️ the delta is **not** uniform: the path/AI band differs again (`+0xb0` ↔ `+0x1c4`, `+0x190` ↔ `+0x210`, `+0x1b8` ↔ `+0x238`). Do not rebase band-wise |
| `C_traffic_generator` fields | see §3 | a clean **+0x10** skew throughout, agreed by the ctor, `DuplicateFrom` and `SaveData` |
| savegame header | 10 bytes | `0x10` bytes (the per-record `0xbc` is shared) |
| `NPC_speed_ratio_player` | absent — no such global and no such write | `GameInit__10C_don_ocas` seeds it to `1.0f` and `AI__19C_traffic_generator` recomputes it every frame from the player's speed, clamped 1.0–2.5. A PS2-only crowd-speed adaptation |
| `SetVisible` | toggles the flag and the model visibility | additionally adds/removes each pedestrian's fake shadow in the same loop |
| `ChangeSceneInit` / `ChangeSceneDone` | none | present — the city-streaming pair the PC build does not have |
| `C_traffic_element` polymorphism | no vtable | `__vt__17C_traffic_element` occurs **0 times** — non-polymorphic on both builds, so the vtable link-order channel can never show it |
| mega-panic animation ids | `0x14a`/`0x14b`, `0x14d`/`0x14e`, `0x150`/`0x151` | `0x118`/`0x119`, `0x11b`/`0x11c`, `0x11e`/`0x11f` — rebased per build; the **pairing** is the build-invariant |
| CRT string routines | intrinsics (`rep movsb`) | newlib calls (`strcpy`, `strcat`, `strcasecmp`), visible in `GeneratePassengerForTaxi` — a size ratio there measures library style, not a different implementation |

### 7a. Cross-build evidence that settled PC-side questions

| PC question | what the PS2 answered |
|---|---|
| what does the global vector at `0x006560c0` hold? | `AddElement__19C_traffic_generatorFP17C_traffic_element` — a `std::vector<C_traffic_element*>`. The container lever; the PC decomp can only show a stride-4 pointer vector |
| what is the parallel vector at `0x006560ac`? | `ClearElements` destroys its elements as `__dt__28C_animations_machine_de_luxeFv` |
| is `pedestrian_manager_explosion` really an explosion? | `Explosion__19C_traffic_generatorFR8S_vectorff` |
| which of the two `NewPanic` overloads is which? | the PS2 signatures (`R8S_vector` vs `P9I3D_frame`) split them; the PC pair is otherwise identical in shape |
| is the 14 KB function at `0x004b7520` the element tick? | `Tick__17C_traffic_elementFUi`, matched on exact callee counts (`SetAnim` 5↔5, `SetNewNode` 4↔4, `AngleTo2` 3↔3) |

### 7b. Independent witness for `C_game+0x2a1c`

`pumper_on_collision` and `pumper_hit` both forward the impact to `*(C_game+0x2a1c)`'s
vtable slots 30 and 31 — the gas-station attendant hands collisions and hits to the
**`C_don_ocas` proxy**, so the engine processes them the way it processes a pedestrian's.
The PS2 twins forward to `C_game+0xe00`.

⇒ **PC `C_game+0x2a1c` ↔ PS2 `C_game+0xe00`**, corroborated from outside this subsystem:
the offset is otherwise derived from `game_init`'s own `operator_new(0x70)` store — one
producer; this is a consumer, in a different translation unit, reached from a different
direction (`structs/pumper.md`).

⚠️ It also says something about the class: `C_pumper` is a `C_actor` subclass with its
own vtable and its own kind byte, and it STILL routes impacts through the crowd proxy.
So the proxy is not only the pedestrians' stand-in — it is the shared impact handler
for human-shaped actors that do not implement one. Whether any third class does the
same is **not** established.

## 8. Open items

| item | state |
|---|---|
| 🥇 `InvalidateActor__19C_traffic_generator` | no PC function in the TU: the gap between `_clear_pedestrians` and `_get_pedestrian_from_frame` holds exactly 6 starts and all 6 are body-matched to other methods, so it is genuinely absent from this cluster — **not located**, not "inlined". Look for a second cluster before tagging it |
| `HitFall` / `HitGun` / `HitStroke` | PS2 file-local; the PC gap between `pedestrian_set_anim` and `pedestrian_set_panic` is empty and `pedestrian_hit` is 1.43× the PS2 `Hit`. Consistent with inlining; **not proven** |
| the four designer floats `+0x88..+0x94` | saved, loaded and duplicated, but no reader is traced for any of them individually |
| `+0xb4`, the cop-slot boundary | `pedestrian_manager_ai` restricts categories 1/3 to slots at or below it; who writes it is untraced |
| `+0xc4..+0x10c` | 0x12 dwords the ctor zeroes with no located reader |
| `+0x190` = `3000` | written on the witness path; no located reader |
| the `'SNGM'` / `'SMMR'` child-frame magic | `pedestrian_create_model` stores the matching child frame at `+0xb8`; what it selects is unproven |
| `system\taxi_pas.dat` | the passenger-profile table `pedestrian_wave_at_taxi` reads; format undocumented |
| in game | does `g_traffic_scream_budget` audibly cap crowd screams during a shoot-out? A grenade in a busy street should hit the `> 100` refusal |
