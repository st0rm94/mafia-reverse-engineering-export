# `C_car` — the car entity

**Created** 2026-07-23 10:48 +0200 · **Modified** 2026-10-13 17:51 +0200

**Subject:** the **PC** `Game.exe` class the reconstruction calls `car_*`. The car ENTITY: it owns
the physics/drivetrain sub-object `C_Vehicle` (`structs/vehicle.md`) embedded at **`car+0x70`**,
plus the seat/owner system, model-level (CARINIT) switching, collision/damage/deform, save/load and
per-frame AI/Update.

**Entity kind:** `car+0x10 == 4` (`structs/entity.md` §1). Hierarchy: `C_car` is a distinct entity
class, **not** part of the `C_actor`←`C_human` line. Instance size `0x21f4`.

**Coverage:** 77 of 89 PS2 `C_car` methods are twinned; the PC translation unit is closed with 0
unnamed starts (§3).

> **`C_car` vs `C_Vehicle` — TWO classes, TWO prefixes.** `C_car` (mangled `5C_car`, the entity) →
> **`car_*`**. `C_Vehicle` (mangled `9C_Vehicle`, the physics sub-object at `car+0x70`) →
> **`vehicle_*`** (`structs/vehicle.md`). The PC `C_car` code cluster is at `0x0046xxxx`; the
> `C_Vehicle` cluster is at `0x00517xxx`. A function in the `0x0046xxxx` cluster whose `this`
> derefs a `C_car` field — the `+0x210c` seat array, the `+0x308` part table — is a `C_car` method
> whatever its offsets look like.

**Confidence tags:** **[H]** body-read + writer + reader; **[M]** inferred; **[U]** not observed.

---

## 1. `C_car` field layout (PC offsets)

PC offsets, off the `C_car` base. PS2-equivalent offsets are in §6, not here.

| Offset | Field | Conf | Evidence |
|---|---|---|---|
| +0x10 | entity kind (= **4** for a car) | H | standard entity kind (`structs/entity.md` §1) |
| +0x70 | embedded `C_Vehicle` (physics/drivetrain) | H | `lea ecx,[car+0x70]` at the `vehicle_set_engine_on` / `vehicle_get_engine_rpm` callsites |
| +0x294 | the `C_Vehicle` master flag word (`C_Vehicle+0x224`, `structs/vehicle.md` §1). Bit 25 (`0x2000000`) = **the last physics step made a contact whose collision record carries flag `0x10000`** | H | `vehicle_update_visuals` sets `0x2000001` when `C_Vehicle+0x28` is set and clears bit 25 otherwise; `+0x28` is raised by `vehicle_wheel_contact_solve` and `vehicle_body_coll_resolve` on such a contact. Readers: `human_find_blocking_drawbridge` (`car+0x294 >> 0x19 & 1`, the drawbridge exit-block) and two arms of `vehicle_wheel_contact_solve`. `0x10000` is bit 16 of a static collision primitive's flag dword, loaded unchanged from `missions\<mission>\tree.klz` by `collision_load_level`. A parse of all 81 shipped `tree.klz` files finds it only on box primitives (kind `0x83`): five per leaf on the drawbridge leaves `LLsklap01` / `LLsklap02` and `sklapx01` / `sklapx02` in every city scene (the east leaves are named `sklap01` / `sklap02` in three scenes), plus four single boxes in three other scenes. So bit 25 marks a contact with a drawbridge leaf |
| +0x634 | **engine revs per second** = `C_Vehicle+0x5c4` | H | written by the drivetrain (`vehicle_update_motor_revs`) and, on a remote car, by `car_recompute_velocity` from the received 16-bit value × `+0x1f88`; `vehicle_get_engine_rpm` returns it × 60 while the engine runs; `car_extended_update` builds its rev fraction from it (`structs/vehicle.md` §1) |
| +0x6d8 | **idle revs per second** = `C_Vehicle+0x668` | H | `vehicle_apply_carinit_params` sets it from the CARINIT `idling rpm` ÷ 60 (`def-files-docs/vehicles-bin.md`); the drivetrain and `vehicle_update_strips` measure the revs above it |
| +0xd18 | **engine running** (byte) = `C_Vehicle+0xca8` | H | `vehicle_set_engine_on` writes it; `car_recompute_velocity` sets it on a remote car while the received revs are at least 1 per second. Read by `car_update` (four times, the steady-vibration force-feedback effect among them), `car_extended_update`, `player_ai_drive`, `player_do_action_button_in_car` and `traffic_car_take_over_cruising_car` |
| +0xd90 / +0xd94 | **door/window record array** begin / end (stride `0x24`) = `C_Vehicle+0xd20` — parallel to `+0x210c`, same index space | H | `vehicle_get_seat_count` (`0x00425460`) returns `(end − begin)/0x24` ¹ |
| +0x1ac | under-pursuit flag | H | pursuit dispatch 1 / remove 0 (§1a) |
| +0x1f7c..+0x1f84 | velocity vec3 (vehicle-space) | H | `car_recompute_velocity` writes it, `car_get_actor_speed` copies it out; the wheel-contact proximity helpers use it |
| +0x1f88 | max engine revs/sec = `C_Vehicle+0x1f18` | H | `vehicle_apply_carinit_params` sets it from the CARINIT `maximum rpm` ÷ 60; the HUD normalises RPM as `vehicle_get_engine_rpm / (car+0x1f88 * 60)` (`basic/artifacts/hud_elements.md` §6b) |
| +0x1f8c | fuel-tank capacity | H | denominator of the HUD fuel-needle ratio |
| +0x1f90 | odometer reading (float) | H | `car_game_init` stores `251000 + 150 × crt_rand()` and nothing else writes it; `player_ai` and `player_ai_drive` pass it to `hud_speedometer_set_values`, which shows the five low digits of ⌊value × 0.1⌋ (`basic/artifacts/hud_elements.md` §6b) |
| +0x1ff2 | **police-model flag** (byte) | H | `car_ctor` clears it; `car_init` sets it when the uppercased model name `+0x20cc` matches one of the four police model strings `POLI`, `POLCAD`, `POLIMFOR`, `POLIMTUD` (`basic/artifacts/vehicle-movement.md` §5.1). Readers: `police_manager_tick` and `garage_game_done`, which never banks such a car (`basic/artifacts/garage/storing-cars-in-the-garage.md`). A different field from `+0x2044` |
| +0x1ff4.. | 6-field CARINIT scratch block | H | `car_set_model_level` copies 6 dwords here from the level record `+0x1124`; `+0x1ffc` is the maximum of damage pool 2 (§1b) |
| +0x2004 | **damage pool 2, current value** | H | `car_hit` and `car_on_collision_apply_damage` subtract damage from it and clamp at 0; `car_get_condition_fraction` channel 2 reads it against `+0x1ffc`; `car_set_model_level` preserves it around the block copy, so a boost swap keeps the damage; `car_init`, `car_game_init`, `car_reset` and `car_explosion` write it. Nothing else reads it, and nothing selects channel 2 (§1b) |
| +0x2020 | derived power/mass ratio | H | `car_set_model_level` writes `(+0x660·+0x564)/(+0x6e0·+0x69c)` |
| +0x2031 | **show-damage flag** (byte) | H | written only by `car_set_show_damage`, the script command `showcardamage` (dispatcher case `0xd5`, six shipped calls); cleared by `car_game_init`, saved by `car_save_game_save`. Its one reader is `player_show_car_damage_bar`, so the car-damage bar appears only for cars a script marked |
| +0x2036 | **must-steal flag** (byte) — the lock must be forced before the car can be entered | H | setter `car_set_must_steal` (script `car_muststeal`, dispatcher case `0xc5`). Also written `1` by `traffic_car_activate_elements` and by `police_action_tick` / `police_action_dispatch_officer_commands` when a stopped chase car's crew is ordered out (event `0xf`); cleared by `human_movement` and `human_do_throw_occupant_from_car`; saved by `car_save_game_save`. Read three times by `player_do_action_button_on_foot`, which takes its ordinary enter-car path only while the flag is 0 |
| +0x2038 | chase-flag word (`0xC00`) | H | pursuit dispatch / remove (§1a) |
| +0x2044 | **is-police-car flag** | H | task seed; read by the police-car sighting/assault path. `police_action_tick` **clears** it just before handing the car to `traffic_car_adopt_car_with_new_driver`, so a released chase car stops being a police car |
| +0x209c | player-driving flag | H | `car_set_owner` sets it when the player takes the driver seat; `car_refresh_player_is_driver_flag` refreshes it |
| +0x20b8 | **condition channel selector** | H | §1b |
| +0x20bc | **automatic-headlight suppress** (byte) | H | `car_set_auto_headlights_suppressed` writes it; `car_update`'s automatic headlight block runs only while it is 0. Script command `carlight_main` sets it when it forces the lights off, so the per-frame night-time re-enable cannot undo it |
| +0x20bd | **is-traffic-car flag** (byte) | H | written only by `car_ctor`, `car_set_is_traffic_car` and `car_save_game_load`. The ambient-traffic gate `car_ai_tick` reads; `car_deactivate` clears a traffic car's script light overrides (`0x0ff00000`) when `C_game+0x2d44` says day. Not the same field as `+0x20bc` |
| +0x20c / +0x3f8 | **light-state flag word** / the `C_Vehicle` wake-up callback slot the light setters invoke | H | ² |
| +0x20ec | **current model level** (CARINIT level index) | H | `car_get_model_level` returns it, `car_set_model_level` writes it ³ |
| +0x20f4 / +0x20f8 | model-level table begin / end (stride `0x113c`) | H | `car_get_model_level_count` = `(+0x20f8 − +0x20f4)/0x113c`; these are the `vehicles.bin` boost levels (`police-system/artifacts/boosting.md`) |
| +0x210c / +0x2110 | **seat RECORD array** begin / end (stride `0xa0`) — occupancy byte `+0xc`, occupant `+0x10`, two placement vec3s `+0x84`/`+0x90` | H | `car_set_owner`, `car_get_owner`, `car_get_seat_property`, `car_seat_get_attach_positions`, `car_set_enter_protect` |
| +0x2214 | truck cargo-bed enabled | H | `human_intern_use_car_truck` guard |
| +0x2230 | truck cargo-bed frame | H | the same guard |
| +0x21d4 | the collision-damage record queue | H | §8 |

¹ Three consumers feed that index straight into `car_get_owner`:
`ai_fight_followship_count_free_seats`, `ai_follow_manager_get_car_seat_for_follower` and
`ai_follow_manager_group_fits_leader_vehicle`; the car initialiser builds one seat record per
element. The array is reached as `C_Vehicle+0xd20` by its own accessor — the same two-bases
situation as `+0x688` speed — and `structs/vehicle.md` §1 decodes it: one record per door, the
door/window object at record `+0x20`. `car_init` passes whether element 0 has that object to
`human_get_anim_id_for_use_car`, which picks the enter-car animation variant.

² The `C_Vehicle` spelling of the pair is `+0x19c` / `+0x388`; the slot belongs to the callback
bank and the light setters are its consumers (`structs/vehicle.md` §8a). The authoritative bit map
is `structs/vehicle.md` §1b. `car_update` and `car_deactivate` reach them through the `C_car` base
while the script setters reach the same memory as `C_Vehicle` (`add ecx, 0x70`).

³ The script-side trio is `car_getactlevel` / `car_getmaxlevels` / `car_setactlevel` (dispatcher
cases `0x164` / `0x163` / `0x165`). **Both GETTERS gate on `GetKind == 6` (`C_door`) instead of 4
and are therefore dead**, with 0 shipped call sites each. `car_setactlevel` gates correctly and is
the script entry to the boosting swap. The Mafia Script Data Reference reads all three as a FUEL
gauge; there is no fuel model here.

### 1a. Chase / HUD high-level readouts (`C_car`-relative)

The chase AI (`C_driver`, reached as `driver+0x10 + N`) and the HUD instrument producer read these
directly off the `C_car`. PS2 equivalents are in §6.

| Offset | Field | Conf | Where |
|---|---|---|---|
| +0x30..+0x38 | car world position | H | `driver_fill_start_values` reads |
| +0xA0..+0xA8 | AI look-at target | H | `driver_turn` writes (`+0xac` valid-flag = 1) |
| +0x20C | light/indicator + control flags | H | `driver_do_lights`, `driver_reset`. Authoritative bit map: `structs/vehicle.md` §1b ¹ |
| +0x51C | engine/siren sound trigger | H | `driver_do_sounds` writes |
| +0x51D | **chase-order-active** (byte) | H | set to `1` by every `pursuit_event_dispatch_pursuit_vehicle` dispatch in `police_action_tick`, cleared on every cancel and teardown there and in `police_action_disband` / `police_action_dispatch_officer_commands`. Marks "this car currently carries a pursuit order" |
| +0x594..+0x597 | the `C_Vehicle` **per-axis direct-input flags** `+0x524..+0x527` (steer / accelerate / brake / clutch) | H | written by `player_ai_drive` (1 while that axis is driven by its secondary binding), `driver_tick` (all 0, or 1/1/1/0), `traffic_car_activate_elements` (all 1) and `car_net_snapshot_state`; `car_update` plays force feedback only while `+0x594` is set. Semantics in `structs/vehicle.md` §1 |
| +0x5B8 | brake-light / coast flag **and** transmission mode (A/M) | H | `driver_tick`; also read by the HUD producer → `hud_speedometer_set_gear` (`basic/artifacts/hud_elements.md` §6b) |
| +0x648 | gearbox gear index | H | `vehicle_set_gear` called with `+0x648 ± 1`; `-1` = reverse |
| +0x688 | scalar speed | H | the police speed gate and most driver helpers; getter `vehicle_get_speed` (`0x00425440`), which is declared on the **C_Vehicle** (`lea ecx,[car+0x70]`, so `C_Vehicle+0x618` is this word) — one field, two reachable spellings (`structs/vehicle.md` §1) |
| +0x2040 / +0x215A | chase-order markers | H | `pursuit_event_remove_vehicle` |
| +0x2078..+0x2080 | **`"FUEL"` dummy position** (model-local vec3) | H | filled at car init from the `"FUEL"` frame's `frame+0x80..+0x88` through the model's find-child-by-name virtual (`+0x38`); zeroed when the model has no such dummy |
| +0x2084..+0x208c | **`"TANK"` dummy position** (model-local vec3) | H | the same block, from the `"TANK"` frame ² |
| +0x2090 | **fuel-tank hit radius** | H | `ABS(bbox extent) * 0.5` of the `"TANK"` frame, taken from `frame+0x140` (6 dwords = the bbox pair) at car init. Gates the tank hit test together with `+0x2094` |
| +0x2094 / +0x2098 | **fuel-tank hit counter / max** | H | `+0x2094` is decremented when something passes within `+0x2090` of the tank point (`if (0 < +0x2094 && 0.0 < +0x2090)`); the branch fires when it drops below 1. `car_set_tank_hit_count` writes **both** fields from its one argument, so script command `settankhitcount` (dispatcher case `0xbe`) resets the running count as well as the threshold — re-issuing it heals a tank that has already been shot |
| +0x2159 / +0x215a | **in-water flag** / **collisions-off flag** | H | ³ |
| +0x20C0 | AI avoid target | M | `driver_predictive_sphere_lane_cross` |
| +0x6E0 | gravity / accel constant | H | the braking math |
| +0x6FC | max steer angle | H | `driver_turn` clamp |

¹ Bit `0x10000` is the **main headlights**. `driver_do_lights` sets `car+0x20bc = 1`
(`car_set_auto_headlights_suppressed`) and then **flashes** `0x10000` on a timed pattern from the
duration table at `driver+0x130` — the chase AI flashing its headlights. Script command
`carlight_main` (dispatcher case `0xc0`) drives the same bit, verified in game. Two gates are
confirmed with it: `0x800` gates `0x100` (brake) and `0x8` gates bits `0x1`/`0x2` (indicators).

² It is a POINT, not a matrix. `car_get_fuel_tank_world_pos` (`0x004254a0`) transforms it by the
body matrix at **`car+0xdd4`**; Ghidra renders the call as
`S_vector::operator*((S_matrix *)(this + 0x2084))`, which mis-assigns the operands — the
disassembly is `f(point = car+0x2084, out, matrix = car+0xdd4)`. The accessor's receiver is the
**C_car itself**, not the `C_Vehicle` at `+0x70` (`mov ecx,eax` straight off `actor_get_vehicle`,
with the kind checked one instruction earlier). Consumers use the world-space result as the AI's
aim point when `entity+0xf08 & 0x200000` and the target's kind is 4 — **shooters aim at the fuel
tank**.

³ Two adjacent bytes, each with one script-side reader or writer. `+0x2159` is set by
`car_on_body_surface_contact` on deep water and read by script command `car_inwater` (dispatcher
case `0xca`). `+0x215a` is written by script command `car_invisible` (case `0x129`), alongside
`vehicle_set_dynamic_colls_off` and `car_set_enter_protect` — the three writes that together make
a car non-solid, non-enterable and still visible.

### 1b. The condition CHANNEL (`+0x20b8`) and the four damage pools

`car_get_condition_fraction` returns a 0..1 fraction, and `+0x20b8` selects WHICH meter it
reports. Ghidra renders its FPU returns as integer returns, so the arms are read from the
disassembly: a four-entry jump table at `0x00470868`. Arms 0..2 are **current ÷ maximum** of a
separate damage pool; arm 3 is one deform zone's integrity.

| Channel | Formula (PC offsets) | Notes |
|---|---|---|
| 0 | `+0x2dc / +0x2e0` (`C_Vehicle+0x26c / +0x270`) | **the gearbox pool.** `car_hit` subtracts from it and clamps at 0; `car_is_usable` requires it above 0. With damage enabled (`C_Vehicle+0xce4` bit `0x4000`) an empty pool makes `vehicle_set_gear` refuse every gear change and stops the automatic shift in `vehicle_tick_drivetrain`; `vehicle_update_visuals` tests the same gate |
| **1** | `+0x2a4 / +0x2a8` (`C_Vehicle+0x234 / +0x238`) | **the engine pool.** `car_hit` subtracts from it and then derives `+0x2a0 = +0x29c + (1 − +0x29c)·(pool/max)` (`C_Vehicle+0x230` from `+0x22c`), the factor `vehicle_compute_motor_force` multiplies the motor force by; `vehicle_apply_damage` drains it the same way and latches `C_Vehicle+0x66c` once it is empty (`structs/vehicle.md` §2a); `car_is_usable` requires the pool above 0. **This is the channel the chase AI uses** |
| 2 | `+0x2004 / +0x1ffc` | `car_hit` and `car_on_collision_apply_damage` subtract from it (the collision path touches this pool only); the one pool `car_set_model_level` preserves across a boost swap. **No consumer:** a census of every instruction in `.text` with displacement `0x2004` finds only its updaters and this arm, and no code selects channel 2 — the channel is 3 after `car_game_init`, and every override sets 1 |
| 3 | `zone+0x20 / zone+0x18` of the **last** `DEFORM BODY` zone (type 1) in the deform-zone table `+0x308` (§4c) | `1.0f` when the car has no body zone |
| default | `1.0f` | - |

Before any of that, the getter returns **0.0f** when `+0x5e` is set — the `C_actor`
death-processed latch (`actor_is_death_processed`, `structs/actor.md` §1).

**Reading and restoring the channel** is `car_set_condition_channel` (`0x004f5ad0`), an
out-of-line `C_car` inline member that writes `+0x20b8` and returns the previous value.
All nine of its callsites are a scoped override — `old = set(1); read(); set(old)` — and
the pushed immediate is literally `1` at every one, disassembled rather than inferred. The chase
AI reads `== 0.0` as *wrecked* and `< 0.15` as *give up*.

---

## 2. PC↔PS2 method coverage

Every PC function carrying the `car_` prefix gets a row, ordered by PC VA, **including those with
no identified PS2 twin** — those show `-` in the PS2 columns and status `PC-only`. PS2 methods with
no located PC twin follow at the bottom with `-` in the PC columns and status `PS2-only`, each
carrying the reason there is no PC function.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role / notes |
|---|---|---|---|---|---|
| 0x004254a0 | `car_get_fuel_tank_world_pos` | - | - | PC-only | Transforms the `"TANK"` point at `+0x2084` by the body matrix `+0xdd4` (§1a) |
| 0x00462ad0 | `car_static_state_init` | - | - | PC-only | The TU's CRT static initialiser, reached through the `crt_static_init_guard_00462a90` pair. Zeroes `g_car_ff_prev_velocity` (`0x00655ab0`), the previous car-space velocity `car_update`'s force-feedback impact block differences, which is why it must start at zero |
| 0x00462af0 | `car_ctor` | `__ct__5C_carFv` | 0x00129cd0 | CONFIRMED | `actor_ctor` then the `C_Vehicle` ctor; installs BOTH MI vtables `0x0063bc08`/`0x0063bcac`, stamps kind `+0x10 = 4`, empties the four owned vectors, seeds `+0x2098` from `rand()` |
| 0x00462c20 | `car_scalar_deleting_dtor` | - | - | PC-only | Vtable slot 16 of `0x0063bc08`; an MSVC-only shape |
| 0x00462c40 | `car_dtor` | `__dt__5C_carFv` | 0x00129e50 | CONFIRMED | Re-installs both MI vtables, `hud_radar_remove_car`, frees the four owned vectors, chains the `C_Vehicle` dtor and `actor_dtor` |
| 0x00462d30 | `car_init` | `Init__5C_carFP9I3D_frame` | 0x00129f40 | CONFIRMED | `actor_init`, the six tuning floats (27.7/3.0/10.0/100.0/100.0/0) in the same order on both builds, `vehicle_init`, then the THIRTEEN-entry `C_Vehicle` callback bank `+0x364..+0x398` in an order identical on both builds, the AI probe points `+0x1fc0`/`+0x1fcc`, and the `SM_`/`LG_` seat-dummy scan |
| 0x00463500 | `car_seat_get_anim_placement` | `GetPlacementFromTCK__5C_carFiiR8S_vectorR8S_vector` | 0x0012ac50 | CONFIRMED | Seat enter/exit placement from the `.tck` track. Byte-exact opening: seat index × `0xa0` (PS2 × `0xd0`), the `+0xc` present-flag early-out (PS2 `+0xc4`), then a 128-byte clip-name buffer. 304 vs 364 B |
| 0x00463630 | `car_seat_get_attach_positions` | `GetPlacementFromTCK1__5C_carFiR8S_vectorR8S_vectorb` | 0x0012adc0 | CONFIRMED | 5-arg `(this, seat, vec*, vec*, bool)`; guard seat-flag byte + `int == 0`; two matrix-transformed seat placements; returns 1/0. Seat array `+0x210c` stride `0xa0` (PS2 `+0x2444` stride `0xd0`) |
| 0x00463740 | `car_seat_index_of_usable` | `GetUSindex__5C_carFPCQ214C_using_object14S_using_object` | 0x0012af00 | CONFIRMED | Reverse lookup of a `C_using_object` to its seat index: backwards scan of `+0x210c` stride `0xa0` over the TWO usables each seat owns, `-1` sentinel. 96 vs 128 B. US = using-object |
| 0x004637a0 | `car_load_materials` | `LoadMaterials__5C_carFv` | 0x0012af80 | CONFIRMED | Gated on `g_num_surface_materials == 0`; builds the default `S_TSRFC` record (grip 0.35, scale 1.0, four 100.0 limits) and grows the raw-material vector (stride `0x30`). Rank forced after `car_seat_index_of_usable`. 656 vs 788 B (0.83) |
| 0x00463a30 | `car_load_particle_effects` | `LoadParticles__5C_carFv` | 0x0012b310 | STRUCTURAL | N→1: MSVC inlined `Load_Particle` into `LoadParticles`, so no single body agreement can be asserted. 336 + 2288 = 2624 against the PC's 2560 B is a ratio of 0.98, and the PC body carries the `s_Car_particle_0064b7cc` string the PS2 loop uses |
| 0x00464430 | `car_load_tyre_surfaces` | `LoadTyres__5C_carFv` | 0x0012c680 | CONFIRMED | STRING ANCHOR: `dtaOpen(tables\tyres.bin)` plus the same `version == 1` gate on both builds; per record it reads the raw floats, converts through `car_convert_tyre_surface` and appends with stride `0x5c`. 752 vs 824 B (0.91) |
| 0x00464720 | `car_convert_tyre_surface` | `ConvertTyres__5C_carFP7S_TSRFCP6S_SRFC` | 0x0012ca40 | CONFIRMED | 2-parameter signature exact. 18 source floats copied out, two scaled by 1000.0, two by 0.01745328 (deg→rad), and three max−min span fields precomputed at `+0x1c` / `+0x28` / `+0x48`, in the same order on both builds. 160 vs 140 B (1.14) |
| 0x004647c0 | `car_set_owner` | `SetOwner__5C_carFP7C_actorif` | 0x0012cad0 | CONFIRMED | Assign or vacate a seat occupant; seat array `+0x210c` stride `0xa0` |
| 0x00464c50 | `car_get_seat_property` | `GetSeatProperty__5C_carFiRbRbRbRb` | 0x0012d060 | CONFIRMED | 4-flag seat query; returns 1/0 |
| 0x00464ee0 | `car_get_owner` | `GetOwner__5C_carFi` | 0x0012d230 | CONFIRMED | The occupant of a seat |
| 0x00464f40 | `car_set_has_driver` | `SetDriver__5C_carFb` | 0x0012d2b0 | CONFIRMED | One-line byte setter on `car+0x3b0`. The field bridge is proven from a CONFIRMED twin, not from position: `car_set_owner` writes `+0x209c = 0`, `+0x3b0 = 0` and `seat+0x40 = 0x7d3` when vacating; PS2 `SetOwner` writes `+0x2462 = 0`, `+0x2299 = 0` and `seat+0x74 = 0x7d3` at the same statement, and `+0x2299` is the byte PS2 `SetDriver` writes. All 7 callsites pass 1 |
| 0x00464f50 | `car_copy_carinit_from_actor` | `DuplicateFrom__5C_carFP7C_actor` | 0x0012d2d0 | CONFIRMED | `kind == 4` gate, actor `DuplicateFrom` (folded onto `stub_noop_args4_00461180` on the PC), `GetInit__9C_Vehicle`, `InitCar__9C_Vehicle`. The PS2 body's 4672-byte local is why the PC body opens with `__chkstk` |
| 0x00464fa0 | `car_save_data_chunk` | `SaveData__5C_carFP7C_chunk` | 0x0012d330 | CONFIRMED | `operator_new(0x111d)` (PS2 `0x124b`), `car_pack_save_record`, Compress, three guarded `dtaWrite` calls, free. Identical control flow |
| 0x00465090 | `car_load_data_chunk` | `LoadData__5C_carFP7C_chunk` | 0x0012d450 | CONFIRMED | Mirror of `SaveData`; the version dword must be 1; calls `car_unpack_save_record` at the exact statement PS2 `LoadData` calls `ReadSaveStruct` |
| 0x00465290 | `car_load_model_level_tables` | `LoadLevelTables__5C_carFv` | 0x0012d610 | CONFIRMED | Empties the level vector (`+0x20f8 = +0x20f4`), `dtaOpen`, uppercases each record name, matches it against the car model name at `+0x20cc` and appends `S_CARINIT` levels of stride `0x113c`. Ratio 0.10 — the PS2 inlined the field parser; the role is anchored on the level-vector pair `car_get_model_level_count` divides |
| 0x00465760 | `car_repair_position_on_ground` | `_RepairPosition__5C_carFb` | 0x0012ff50 | CONFIRMED | Downward collision probe at −200 then a −2000 retry, then cross-product re-orthonormalisation of the body axes `+0x30`/`+0x34`/`+0x38` with a 1e-08 tolerance. 800 vs 780 B (1.03) |
| 0x00465a80 | `car_set_is_traffic_car` | `SetTrafficCar__5C_carFb` | 0x00130260 | CONFIRMED | One-line byte setter on `car+0x20bd` — the exact ambient-traffic gate `car_ai_tick` reads, and a field `car_save_game_save` round-trips. PS2 `+0x2460` |
| 0x00465a90 | `car_get_model_level_count` | `GetLevelsCount__5C_carFv` | 0x00130270 | CONFIRMED | Returns the CARINIT model-level count `(+0x20f8 − +0x20f4)/0x113c` (PS2 caches it at `car+0x2434`). These are the `vehicles.bin` boosting levels |
| 0x00465ac0 | `car_get_model_level` | `GetActualLevel__5C_carFv` | 0x00130280 | CONFIRMED | Returns the current model level `car+0x20ec` (PS2 `car+0x242c`) |
| 0x00465ad0 | `car_set_model_level` | `SetActualLevel__5C_carFi` | 0x00130290 | CONFIRMED | Switches CARINIT model level: bound check, `InitCar` on the `C_Vehicle`, copy 6 fields, set `car+0x20ec`, recompute the ratio. Byte-structural match |
| 0x00465bd0 | `car_create_force_feedback_effects` | - | - | PC-only | No PS2 rank between `SetActualLevel` and `GameInit`. Builds the FIVE shared car force-feedback effects (types 10/7 Condition, 0 Constant 220 ms, 2 Periodic ×2), handles in `g_ff_car_effect_*`. It creates no sound channel. See `structs/force_feedback.md` §5b |
| 0x00465d30 | `car_release_force_feedback_effects` | - | - | PC-only | Releases all five car force-feedback effects, `0x0064b73c` included |
| 0x00465d90 | `car_game_init` | `GameInit__5C_carFv` | 0x001303d0 | CONFIRMED | Per-scene car init: `vehicle_enable_sounds(0)` → base-class actor `GameInit` → model level `-1` → `car_set_model_level(0)` → randomise |
| 0x00466680 | `car_game_done` | `GameDone__5C_carFv` | 0x00131370 | CONFIRMED | `EnableSounds(0)`, `vehicle_game_done`, `actor_game_done`, the seat `DelObject` loop, claim mask 0, `ShadowCarOn(0)`, `hud_radar_remove_car`. Forced 1:1 gap; 448 vs 568 B. This diff identified `vehicle_game_done` |
| 0x00466840 | `car_ai_tick` | `AI__5C_carFUi` | 0x001315b0 | CONFIRMED | The per-frame tick of every car: the traffic block gated on `+0x20bd` (brake ×0.7/×0.4, steer ±0.2 = `0xbe4ccccd`/`0x3e4ccccd`), the `rand()%200` / `%300+100` stall timer, then `vehicle_tick_drivetrain` unconditionally (`basic/artifacts/vehicle-movement.md` §3). Byte-exact |
| 0x00466b80 | `car_update` | `Update__5C_carFUi` | 0x00131a90 | CONFIRMED | The primary per-frame vtable tick: Reset-flag velocity guard, `0x53` dt-clamp, distance² LOD/cull flags `+0xd54`, inlined path-node management, tail `SetPower`/`SetBrake`/`IsCarUsable`. Head statement-for-statement against the PS2 |
| 0x00468970 | `car_recompute_velocity` | - | - | PC-only | Derives the velocity vec3 at `+0x1f7c` from the position delta ÷ dt; a `car_update` callee. Inlined into `Update` on the PS2 |
| 0x00468bc0 | `car_explosion` | `CarExplosion__5C_carFUi` | 0x00134270 | CONFIRMED | Turns the car's lights and indicators off by progressively clearing the light-flag word `car+0x20c` (= PS2 `car+0x5f8`) plus the `code*` callback `+0x3f8`; `car_update` calls it as `(0)` |
| 0x004699c0 | `car_deactivate` | `DeactivateCar__5C_carFUi` | 0x001354c0 | CONFIRMED | `0xf00fffff` mask clear |
| 0x00469ad0 | `car_reset` | `Reset__5C_carFfb` | 0x00135620 | CONFIRMED | Flag `+0x2158` from `param == 0`, Reset the `C_Vehicle`, clear route masks `+0x2100`/`+0x2104`, re-seat door states `0x7d3`/`0x7d4`/`0x7d9`, speed-gated (< 0.3), disable sounds. Statement-for-statement against the PS2 |
| 0x00469dd0 | `car_reset_seats_and_routes` | `ResetCar__5C_carFv` | 0x00135d80 | CONFIRMED | Byte-exact on every landmark: the PS2 writes `+0x2440 = 0xffffffff` and `+0x2470 = 0`, gates on speed `+0x8f0 < 0.3`, and re-stamps the seat door codes `0x7d9` / `0x7d3` / `0x7d4` in the same branch order with the index-1 special case at `+0x150`; the PC writes `+0x2104` / `+0x2034`, gates `+0x688 < 0.3`, same codes, special case at `+0xec`. The no-arg sibling of `car_reset`. 512 vs 1560 B — the PS2 inlined its vector maths |
| 0x00469fd0 | `car_is_point_in_path_ahead` | `ChcesMeSejmout__5C_carFRC8S_vectorf` | 0x00136400 | CONFIRMED | A run-over corridor test. The opening is byte-exact (bit 0 of `+0x206c` / PS2 `+0x23e4`, cached byte `+0x2070` / PS2 `+0x2465`, the wheel-contact scan over `+0xd24` / PS2 `+0xf8c` with flags `&8` and `&0x40000000` and surface `!= 1 && != 10`); the tail accepts the point only within ±4.0 laterally, inside `max(speed*1.8, range)`, a 1.6 rad cone and a `sin(theta)*d < 2.0` miss distance. It returns the car itself, so the caller gets a predicate and a handle at once. The PS2 name is slang and is deliberately not transliterated. 608 vs 912 B (0.67) |
| 0x0046a230 | `car_pack_light_state` | - | - | PC-only | Folds `car+0x20c` bits `0x1` / `0x2` / `0x100` / `0x1000` / `0x10000` / `0x80` into a 6-bit byte. Bit map in `structs/vehicle.md` §1b; the exact inverse of `car_unpack_light_state`, which is what makes the mapping checkable |
| 0x0046a270 | `car_unpack_light_state` | - | - | PC-only | The inverse, applied one bit at a time: each lamp is only touched when its override bit is clear (`0x8` guards the indicators, `0x800` the brake/reverse pair), and every applied change fires `car+0x3f8` = `C_Vehicle+0x388` — the evidence behind `structs/vehicle.md` §8a, that the slot is the WAKE-UP hook used here as a change notification rather than owned by the light field |
| 0x0046a3b0 | `car_set_shadow_receiver_frames` | `SetShadowsToMe__5C_carFb` | 0x00136790 | CONFIRMED | Frame enumeration by name match off the model at `+0x438` (PS2 `+0x710`), one vtable call with 0 per hit, temporary vector freed. 160 vs 196 B |
| 0x0046a450 | `car_set_transparency` | `SetTransparency__5C_carFf` | 0x00136860 | CONFIRMED | Enumerates the model frames, SKIPS glass panes (`+0x308`) and wheels whose contact carries flag `0x40` at `+0x123`, then filters on the frame magic `'OBJ_'` (`0x5f4a424f`) / `'SNGM'` (`0x4d474e53`) and calls the material vtable slot `+0x84` with the alpha. The same enumerate-then-per-frame idiom as `car_set_shadow_receiver_frames`. 352 vs 600 B (0.59) |
| 0x0046a5b0 | `car_get_surface_material_id` | `GetMaterialID__5C_carFP9I3D_frame` | 0x00136ac0 | CONFIRMED | BYTE-EXACT: the `-1` guard at `+0x1ff0`, the glass array `+0x308` stride `0x34` (type 3 → `0x18`, type 9 → `0x12`), the wheel array `+0xd24` → `0x13`, else `0x14`. True size 192 B (§3c) |
| 0x0046a670 | `car_hit` | `Hit__5C_carF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00136ba0 | CONFIRMED | The class's damage entry point. The 8-argument signature matches; the switch is on the `E_hit_type` parameter exactly as the PS2 symbol declares, and `actor_post_hit_event` is handed `param_2` plus `param_5..param_8` — the same forwarding the PS2 body performs. Damage arms walk `+0x308`, raise `+0x1e8 \| 2`, clamp negative part health and compute the impact point off `+0x484` with the body axes at `+0x510`. True size 4656 B against 5424 (0.86) |
| 0x0046b8a0 | `car_detect_wheel` | `DetectWheel__5C_carFiRC8S_vectorb` | 0x001380d0 | CONFIRMED | Wheel-contact distance classifier: mode 0 = 3D distance ref→contact, else matrix-transform 2D distance; identical 1e-08 / SQRT / 1.0 normalise; contact array `car+0xd24` = PS2 `+0xf8c` |
| 0x0046ba50 | `car_is_possible_collision` | `IsPossibleColl__5C_carFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x00138380 | CONFIRMED | The 11-parameter signature matches the PS2 symbol exactly once Ghidra's expansion of the two `S_vector&` arguments into six floats is accounted for. Sweeps the bounding radius `+0x23c` along the velocity vec3 `+0x1f7c` that `car_recompute_velocity` writes, returning the earliest contact time and the struck actor. Forced 2:2 with `car_on_collision_with_actor`. 1888 vs 3200 B (0.59) |
| 0x0046c1b0 | `car_on_collision_with_actor` | `Collision__5C_carFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00139000 | CONFIRMED | `+0x2198 \|= 1`, `actor_set_act_state(0)`, `+0x215b = 1`, then a switch on the STRIKER's kind `{2, 0x1b}`. True size 432 B vs 492 B (0.88) |
| 0x0046c360 | `car_get_collision_info` | `Actor_Col_Info__5C_carFPCvRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vectorPfRb` | 0x001391f0 | CONFIRMED | QUERIES the per-struck-kind collision-response parameters (contact radius 80.0, restitution −1.0/−1.2, normal, matrix); it does not apply the collision |
| 0x0046c6e0 | `car_react_to_and_report_collision` | `Call_Collision__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vectorf` | 0x00139480 | CONFIRMED | The FULL collision handler: the struck entity's `react_to_impact` vtable call **+** the `CoDelasTyCuraku` traffic-car assault **+** `CarCollision__C_shvestky` police-offence registration, impact-strength keyed. 6-arg |
| 0x0046c910 | `car_react_to_collision` | `Call_Collision1__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vector` | 0x00139790 | CONFIRMED | The consequence-free sibling: the same `react_to_impact` dispatch with a scaled and normalised direction, no offence and no assault. 5-arg |
| 0x0046cbd0 | `car_collision_filter_actor` | `Actor_Filter__5C_carFPCvPCvRb` | 0x00139a40 | CONFIRMED | BYTE-EXACT: the `&0x40` gate, kinds `{6,8,0x1e,0x16}` → 1, kind 4 plus flag bit `0x10000000`, kinds `{2,0x1b,0x13}` plus speed `+0x688 < 4.0` (PS2 `+0x8f0`). Registered by `car_init` in the `SetCallbackAF` slot, `C_Vehicle+0x370` |
| 0x0046cc40 | `car_on_collision_apply_damage` | `Was_Collision__5C_carFPCvRC8S_vectorRC8S_vectorRC8S_vectorfRC8S_vectorPCvPCvb` | 0x00139b20 | CONFIRMED | Body-diffed AND slot-proven (`SetCallbackWC`, `C_Vehicle+0x374`). Normalises the impact by mass `+0x6e0`, raises the four damage bytes `+0x2154..+0x2157` above the 6.0/10 thresholds, walks the seat list injuring occupants through `car_get_owner`, and reports to `police_manager_register_collision_offence_for_target` when `+0x2035` is set. 9 arguments on both sides. 2272 vs 2920 B (0.78). Distinct from `car_react_to_and_report_collision`, which dispatches to the STRUCK entity |
| 0x0046d520 | `car_add_deform_scratch` | `Deform_Vehicle_Part__5C_carFPCvP9I3D_frameUiRC8S_vectorRC8S_vectorb` | 0x0013a690 | CONFIRMED | A one-line forward of all five arguments to `C_game::ScratchAdd`. `SetCallbackDVP` slot, `C_Vehicle+0x394`. 48 vs 44 B |
| 0x0046d550 | `car_clear_damage_decals` | `Vehicle_Repair__5C_carFPCv` | 0x0013a6c0 | CONFIRMED | `ScratchDeleteFromCar` plus `ClearFrameFromFireHole` on the root frame. `SetCallbackVR` slot, `C_Vehicle+0x398`. 48 vs 64 B |
| 0x0046d580 | `car_break_glass` | `Break_Glass__5C_carFPCviRC8S_vectorPCc` | 0x0013a700 | CONFIRMED | Glass array `+0x308` stride `0x34`, and the same NULL-defaults-the-sound branch on both builds (PC `s_lustr1_wav_0064b83c`). `SetCallbackBG` slot, `C_Vehicle+0x380`. 160 vs 128 B |
| 0x0046d620 | `car_wake_up` | `Wake_Up__5C_carFPCv` | 0x0013a780 | CONFIRMED | Forces the act state back to 0. `car_init` leaves the WU slot NULL on BOTH builds; `car_on_act_state_changed` installs THIS address exactly where PS2 `ChangeState` installs `Wake_Up` |
| 0x0046d630 | `car_on_body_surface_contact` | `Collision_Filter_Body__5C_carFPCvPCvRC8S_vectorRC8S_vector` | 0x0013a790 | CONFIRMED | Body-diffed AND slot-proven (`SetCallbackCFB`, `C_Vehicle+0x38c`). Above 0.1 speed with contact flags `&0xc0` it reflects the velocity `+0x1f7c` and marks `+0x2159`; the distinctive arm is deep water — when seat 0's occupant is the PLAYER (kind 2) and the car is not suppressed it raises `game_raise_water_fall_script_event`, spawns particle `0x25` and plays sound `0xec`. True size 2640 B against 2952 (0.89) |
| 0x0046e080 | `car_collision_filter_wheel` | `Collision_Filter_Wheel__5C_carFPCvPCvi` | 0x0013b320 | CONFIRMED | Hit-byte special case: `0x1f` (lethal water) → `contact+0xb4 = 0x1f`, return 0; `0x2b` (slippery) → return 0. Byte-exact |
| 0x0046e0c0 | `car_prepare_wheel_dropout` | `Prepare_DropOut_Wheel__5C_carFPCviRC8S_vectorPC8S_vector` | 0x0013b370 | CONFIRMED | BYTE-EXACT on both documented field bridges: wheel array `+0xd24` (PS2 `+0xf8c`), contact flag `+0x120 \|= 0x40000000` (PS2 `+0x1fc`), mass 10.0, impulse scale 0.1 (`0x3dcccccd`). `SetCallbackPDW` slot, `C_Vehicle+0x37c` |
| 0x0046e1b0 | `car_prepare_body_part_dropout` | `Prepare_DropOut__5C_carFPCviRC8S_vectorPC8S_vector` | 0x0013b490 | CONFIRMED | Body-part array `+0x308`, switch on the part type into a mass/kind pair from `{10.0/2, 2.0/5, 4.0/3, 7.0/6, 2.0/4, 20.0/7}`, then the shared spawner. `SetCallbackPD` slot, `C_Vehicle+0x378` |
| 0x0046e300 | `car_drop_out_part` | `Drop_Out__5C_carFP5C_carP9I3D_framePvii` | 0x0013bb40 | CONFIRMED | The shared back half of both drop-out paths and the **live** part-detach path: it allocates a `C_DropOut`, branches `if (selector == 1) InitWheel else InitBox`, and registers it as a temporary actor. Called from `car_prepare_wheel_dropout` and `car_prepare_body_part_dropout` at exactly the positions where the PS2 calls `Drop_Out` |
| 0x0046e660 | `car_pack_save_record` | `WriteSaveStruct__5C_carFP6STCarsP9S_CARINITb` | 0x0013bc40 | CONFIRMED | Called by `car_save_data_chunk` between the `0x111d` allocation and the compression, at PS2 `SaveData`'s exact `WriteSaveStruct` position. 4240 vs 3956 B (1.07) |
| 0x0046f6f0 | `car_unpack_save_record` | `ReadSaveStruct__5C_carFP6STCarsP9S_CARINITb` | 0x0013cbc0 | CONFIRMED | Called by `car_load_data_chunk` after the decompression and before `vehicle_apply_carinit_params`, at PS2 `LoadData`'s exact `ReadSaveStruct` position. 4192 vs 6024 B |
| 0x00470750 | `car_get_inverse_model_matrix` | `GetInvModelMatrix__5C_carFb` | 0x0013e350 | CONFIRMED | BYTE-EXACT: the `&0x20` dirty test before `UpdateWMatrixProc`, two 16-dword copies into `+0xdd4` and `+0xe14` (PS2 `+0x2e0` / `+0x320`), `S_matrix::Inverse`, return `+0xe14` |
| 0x004707c0 | `car_get_condition_fraction` | `ShowCarEnergy__5C_carFv` | 0x0013e420 | CONFIRMED | The death-processed latch `+0x5e` (PS2 `+0x62`), then a four-way switch on the channel field `+0x20b8` (PS2 `+0x2404`) over three damage pools and one deform zone (§1b; the build difference in arm 3 is in §6). It DISPLAYS nothing — it is the getter the HUD widget reads |
| 0x004709f0 | `car_get_ai_probe_point` | `GetAIPos__5C_carFb` | 0x0013e570 | CONFIRMED | BYTE-EXACT and self-validating: it transforms the two model-space probe points `car_init` seeds (`+0x1fc0` / `+0x1fcc`) into `+0x1fd8` / `+0x1fe4`; the PS2 quartet is `+0x22e0` / `+0x22f0` / `+0x2300` / `+0x2310`. 112 vs 112 B |
| 0x00470a60 | `car_set_enter_protect` | `SetEnterProtect__5C_carFb` | 0x0013e5e0 | CONFIRMED | BYTE-EXACT: seat array `+0x210c` stride `0xa0` (PS2 `+0x2444` stride `0xd0`), the claim/enable mask pair `+0x2100` / `+0x2104` (PS2 `+0x243c` / `+0x2440`), the seat list `+0xd90` stride `0x24` (PS2 `+0xfd0` stride `0x30`), the SAME door action codes `0x7d3` / `0x7d4` / `0x7d9` in the same branch order, and the same `index == 1` special case. Sibling of `car_set_enter_protect1` |
| 0x00470c50 | `car_set_enter_protect1` | `SetEnterProtect1__5C_carFib` | 0x0013e880 | CONFIRMED | Seat-claimed guard — one door at a time |
| 0x00470c90 | `car_get_wheel_coll` | `GetCarWheelColl__5C_carFPbi` | 0x0013e8c0 | CONFIRMED | Reads `(contact+0x120 >> 3) & 1` into an out bool array for `min(count, car+0x5b0)` contacts (array `car+0xd24` = PS2 `+0xf8c`, flag `+0x120` = PS2 `+0x1fc` bit 8) |
| 0x00470ce0 | `car_set_wheel_coll` | `SetCarWheelColl__5C_carFPbi` | 0x0013e950 | CONFIRMED | Sets or clears bit 8 of `contact+0x120` per an in bool array, the same loop as `GetCarWheelColl` |
| 0x00470d30 | `car_is_usable` | `IsCarUsable__5C_carFUi` | 0x0013ea10 | CONFIRMED | `(mask & 1)` → three `float > 0` tests (fuel `+0xd1c` included) + two byte flags + a `0x40000000` contact-array loop. Body-exact. Harvested from `Update`'s callee set |
| 0x00470ea0 | `car_get_navigation_corners` | `GetNavigationPoints__5C_carFfR8S_vectorR8S_vectorR8S_vectorR8S_vector` | 0x0013ec60 | CONFIRMED | The four footprint corners from the bbox fields `+0x254` / `+0x25c` / `+0x26c` / `+0x274` inflated by the margin argument, at the AI ground height `+0x1fc4`. 5-parameter signature exact. 192 vs 180 B |
| 0x00470f60 | `car_on_act_state_changed` | `ChangeState__5C_carFv` | 0x0013ed20 | CONFIRMED | Three-way dispatch on the act state `+0x1c` (PS2 `+0x5c`): state 2 teardown, state 1 installs `car_wake_up` in the WU callback slot, state 0 re-enables sounds and clears it. 304 vs 448 B |
| 0x00471090 | `car_set_shadow_enabled` | `ShadowCarOn__5C_carFb` | 0x0013eee0 | CONFIRMED | Idempotent through the cached flag `+0x1fbc`; `game_add` / `del_frame_for_shadow` around the shadow object at `+0x2118`. 176 vs 224 B |
| 0x00471140 | `car_get_actor_speed` | `GetActorSpeed__5C_carFR8S_vector` | 0x0013efc0 | CONFIRMED | Copies the car velocity vec3 `car+0x1f7c` — the field `car_recompute_velocity` writes — to the out parameter; the PS2 copies `car+0x460` |
| 0x00471160 | `car_enable_sounds_for_scene` | `SoundsChange__5C_carFi` | 0x0013efd0 | CONFIRMED | BYTE-EXACT including the magic scene id `0x7b` and the sounds-enabled byte `+0x1ff1` (PS2 `+0x2468`). 64 vs 80 B |
| 0x004711a0 | `car_mark_parking_start_time` | `SetTimeParking__5C_carFv` | 0x0013f020 | CONFIRMED | Copies the `C_game` mission clock into `car+0x2010` (PS2 `car+0x23a8` from `game+0xe54`) |
| 0x004711c0 | `car_get_parked_duration` | `GetTimeParking__5C_carFv` | 0x0013f040 | CONFIRMED | The exact inverse of the setter, reading the same two fields. 32 vs 24 B |
| 0x004711e0 | `car_prepare_showroom_model` | `PrepareCarToShow__5C_carFP9I3D_model` | 0x0013f060 | CONFIRMED | The menu / showroom model builder — it runs no simulation at all. Duplicates the model name buffer, enumerates frames twice (filters `0x10021` then `0x10001` against the `AXIS_` tag), rebuilds the frame vector, scales the bbox by 0.13, re-centres it and re-runs the model update vtable. Single `I3D_model*` parameter matches. 7072 vs 6180 B (1.14) |
| 0x00472d80 | `car_is_seat_left` | `IsSeatLeft__5C_carFi` | 0x001408c0 | CONFIRMED | BYTE-EXACT: the bounds test, then `0.0 < seat[i].f0` gives 0, else 1. 96 vs 124 B |
| 0x00472de0 | `car_set_roof_visible` | `OdjebajTuStrechu__5C_carFP9I3D_modelb` | 0x00140940 | CONFIRMED | BYTE-EXACT: the same frame enumeration, the same per-frame vtable call with `param == 0`, the same 1/0 return and the same vector teardown on both arms. Reached from `game_cheat_check` case `0xc` (JIZDA), confirmed in game |
| 0x00472ea0 | `car_refresh_player_is_driver_flag` | `RefreshActivePlayer__5C_carFv` | 0x00140a40 | CONFIRMED | BYTE-EXACT: the empty-seat guard, seat 0's occupant against the `C_game` active player (PC `game+0xe4`, PS2 `game+0x150`), the cached byte `+0x209c` (PS2 `+0x2462`) |
| 0x00472f00 | `car_set_force_feedback_gains` | - | - | PC-only | Writes the five force-feedback gain weights `g_ff_car_gain_*` through `force_feedback_manager_set_gain`. It sets no volume. See `structs/force_feedback.md` §5b |
| 0x00472f40 | `car_save_game_get_size` | `SaveGameGetSize__5C_carFv` | 0x00140ae0 | CONFIRMED | `vehicle_get_save_state_size + 0x7b + actor_save_game_get_size` (the PS2 uses `0x80` for the fixed term). This diff identified `vehicle_get_save_state_size` |
| 0x00472f60 | `car_save_game_save` | `SaveGameSave__5C_carFPPv` | 0x00140b20 | CONFIRMED | Record version 3; the base writer, then `vehicle_save_state`, then about 45 field copies out of the `+0x1ff3..+0x2155` state band. 464 vs 540 B |
| 0x00473130 | `car_save_game_load` | `SaveGameLoad__5C_carFPPv` | 0x00140d40 | CONFIRMED | The exact inverse; it vacates every seat first, gates on `version == 3`, and re-drives `car_set_enter_protect` and `car_set_model_level` from the restored bytes rather than poking them. 592 vs 760 B |
| 0x00473380 | `car_track_record_key` | - | - | PC-only | `C_actor_track` slot 0 — §4a |
| 0x00473590 | `car_track_spawn_dropout_part` | - | - | PC-only | `C_actor_track` slot 1 — §4a, §4b |
| 0x00473750 | `car_track_apply_pose_at_clock` | - | - | PC-only | `C_actor_track` slot 2 — §4a |
| 0x00473b20 | `car_track_create` | - | - | PC-only | `C_actor_track` slot 9 — §4a |
| 0x00473b40 | `car_track_apply_pose_at_time` | - | - | PC-only | `C_actor_track` slot 3 — §4a |
| 0x00473e40 | `car_track_get_pos_at_time` | - | - | PC-only | `C_actor_track` slot 4 — §4a. True size 336 B (§3c) |
| 0x004741d0 | `car_carinit_vector_count` | - | - | PC-only | CARINIT level-vector helper, one owner, class-specific stride |
| 0x004742a0 | `car_material_vector_insert_n` | - | - | PC-only | `std::vector<S_TSRFC>::insert`, stride `0x30`. Family-specific rather than generic, measured: 3 callsites, 2 in `car_load_materials` plus one `C_Vehicle` function — both owners in this family and sharing the record type. The type name comes from the PS2 signature `ConvertTyres__5C_carFP7S_TSRFCP6S_SRFC` |
| 0x00474590 | `car_seat_vector_insert_n` | - | - | PC-only | `std::vector<seat>::insert`, stride `0xa0`. ONE callsite, `car_init`, receiver `lea ecx,[ebp+0x2108]` — the header of the seat vector whose records live at `car+0x210c` |
| 0x004748a0 | `car_tyre_surface_vector_insert_n` | - | - | PC-only | `std::vector<S_SRFC>::insert`, stride `0x5c`. ONE callsite, `car_load_tyre_surfaces`; the stride is corroborated independently by `car_convert_tyre_surface` writing that record out to `+0x58` |
| 0x00474c40 | `car_carinit_vector_copy` | - | - | PC-only | CARINIT level-vector helper, one owner, class-specific stride |
| 0x00474c80 | `car_carinit_vector_copy_back` | - | - | PC-only | CARINIT level-vector helper, one owner, class-specific stride |
| 0x004f5ad0 | `car_set_condition_channel` | - | - | PC-only | Out-of-line inline member: writes `+0x20b8` and returns the previous value (§1b) |
| 0x005c8aa0 | `car_set_auto_headlights_suppressed` | - | - | PC-only | Writes `car+0x20bc`, the per-car flag that stops `car_update` from driving the headlight bit automatically at night. All 89 PS2 `C_car` methods were enumerated; none matches |
| - | - | `CalmVehicle__5C_carFUifi` | 0x001363a0 | PS2-only | A three-line forwarder to `CalmVehicle__9C_Vehicle` |
| - | - | `ChangeSceneDone__5C_carFv` | 0x001417f0 | PS2-only | 64 B; a `ChangeScene` pair is the city-streaming shape the PC build does not have |
| - | - | `ChangeSceneInit__5C_carFv` | 0x001417e0 | PS2-only | 12 B; the sibling of the above |
| - | - | `Change_Particle__5C_carFii` | 0x0012bde0 | PS2-only | No PC rank at all, and `car_init` never registers `SetCallbackCP` — the two agree |
| - | - | `CloseCar__5C_carFv` | 0x0012abc0 | PS2-only | No PC start between `car_init` and `car_seat_get_anim_placement` |
| - | - | `DamageDeformGetSize__5C_carFv` | 0x00141040 | PS2-only | No PC rank in the tail; the three tail functions there are container helpers |
| - | - | `DamageDeformLoad__5C_carFPPv` | 0x00141150 | PS2-only | The same |
| - | - | `DamageDeformSave__5C_carFPPv` | 0x00141060 | PS2-only | The same |
| - | - | `DuplicateDamageAndDeform__5C_carFP5C_car` | 0x00140ab0 | PS2-only | A three-line forwarder to the `C_Vehicle` method |
| - | - | `GetDriver__5C_carFv` | 0x0012d2c0 | PS2-only | An 8-byte accessor; MSVC inlined it. Its setter `car_set_has_driver` survives |
| - | - | `Load_Particle__5C_carFR14S_VEH_PARTICLEi` | 0x0012b460 | PS2-only | Folded into `car_load_particle_effects` — `336 + 2288 = 2624` vs the PC's 2560 B, ratio 0.98 |
| - | - | `UpdateShadow__5C_carFv` | 0x00133360 | PS2-only | Inlined into `car_update` |

---

## 3. The translation unit

**`[0x00462ac0, 0x00474cc0)`**, bounded by the CRT static-init / atexit pairs at
`0x00462a90`/`0x00462ab0` and `0x00474cc0`/`0x00474ce0`. **0 unnamed starts.**

### 3a. The accounting closes with no remainder

```
108 real starts (119 Ghidra starts - 11 decompiler artifact starts)
  =  77 twin-mapped bodies
  +   2 CRT static-init / atexit stubs
  +  12 PC-only members
  +   3 S_TSRFC / seat / S_SRFC vector helpers   (family-specific, census-graded)
  +   8 foreign or genuinely-generic helpers
  +   4 thunks
  +   2 already-named C_car members outside the twin set
```

The 12 PS2 methods with no PC counterpart each carry their reason in the §2 table; none is a
failed search.

### 3b. Not `C_car` — foreign or genuinely generic, leave them

`string_ctor_from_cstr` (`0x00464bd0`) and `string_wildcard_match` (`0x004656b0`) are a
refcounted-string helper class linked into this cluster; they shift the ranks an order-alignment
comparison computes. Also `collision_earliest_contact_between_spheres` (`0x00470880`),
`i3d_frame_match_deform_roof` (`0x00473f90`), and the shared STL helpers
`stl_uninit_copy_stride_0x1c` (27 callsites across ≥10 owners),
`stl_copy_back_stride_0x18` (10 across 7), `vec_count_stride_0x4/0x8/0x18` and
`vector_erase_range_dwords`.

The same five-function container-helper family **splits both ways** under a per-member receiver
census: the two string helpers stay mechanical, while `car_carinit_vector_count` / `_copy` /
`_copy_back` and the three `*_vector_insert_n` functions each have one owner and a class-specific
record stride. Census each member separately rather than grading the family as a whole.

### 3c. Eleven decompiler artifact starts sat inside this TU

They are Aggressive-Instruction-Finder splits, not functions: none is reached by any `E8 rel32`,
none appears as a dword in any data section, and each is the target of an earlier `jcc` inside its
host. They matter because such a split truncates the real function's apparent size, and that size
feeds every order-alignment smell test:

| real function | apparent → true | ratio vs PS2 |
|---|---|---|
| `car_get_surface_material_id` | 151 → **192 B** | 0.69 → **0.87** |
| `car_hit` | 488 → **4656 B** | 0.09 → **0.86** |
| `car_on_collision_with_actor` | 81 → **432 B** | 0.16 → **0.88** |
| `car_collision_filter_actor` | 66 → **112 B** | 0.30 → **0.51** |
| `car_on_body_surface_contact` | 2103 → **2640 B** | 0.71 → **0.89** |
| `car_collision_filter_wheel` | 38 → **64 B** | 0.53 → **0.89** |
| `car_track_get_pos_at_time` | 39 → **336 B** | - |

Three of the eleven are unanswerable by shape alone: their raw bytes decode as
`pop edi ; mov eax, 0x18 ; pop esi ; ret 4` and the same with `0x12` and `0x13` — **three of the
four return constants of `car_get_surface_material_id` itself**. `car_hit` is why this is not
cosmetic: at its apparent 488 B it looked unmappable against a 5424-byte PS2 method.

### 3d. The TU's globals

Every global the TU's functions reference is named: `0x0064b73c..0x0064b74c` are the five
force-feedback effect handles (`g_ff_car_effect_wheel_friction` / `_lateral_spring` / `_impact` /
`_speed_vibration` / `_steady_vibration`), `0x0064b750..0x0064b768` their gain weights and
change-detect caches (`g_ff_car_gain_*`, `g_ff_car_last_gain_*`), `0x00655ab0` is
`g_car_ff_prev_velocity`, and the frame-name suffix strings `".DWHL"` / `".DWHR"` are
`s_dwhl_0064b85c` / `s_dwhr_0064b864`. The remaining data references are string literals Ghidra
labels itself.

---

## 4. The recorder track interface and detachable parts

### 4a. The `car_track_*` family — `C_car`'s override of the `C_actor_track` interface

`C_car` overrides **six** slots of the 13-virtual `C_actor_track` base
(`structs/actor_track.md`), which is the PC-only `C_recorder` keyframe system.

| Slot | PC function | VA | What it does |
|---|---|---|---|
| 0 | `car_track_record_key` | 0x00473380 | Appends a pos/quat/wheel-contact keyframe, 50 ms-gated. Returns **-1** when nothing was recorded — the contract set by the base slot-0 stub `stub_retminus1_args4_0044a780` |
| 1 | `car_track_spawn_dropout_part` | 0x00473590 | Replays a "part came off" key — §4b |
| 2 | `car_track_apply_pose_at_clock` | 0x00473750 | Playback at the **global** `g_recorder_playback_clock_ms`; the only one that derives velocity, into `car+0x1f7c` |
| 3 | `car_track_apply_pose_at_time` | 0x00473b40 | Playback at an **explicit time**; walks the variable-stride key list (stride from `track+0x44 + tag*4`) |
| 4 | `car_track_get_pos_at_time` | 0x00473e40 | Read-only **position** query at an arbitrary time |
| 9 | `car_track_create` | 0x00473b20 | Declares this class's three key sizes: `0x2c`, `0x38`, `0x38` |

**The receiver is byte-proven `C_car`, not `C_Vehicle`:** every one of them reads the track from
`this+0x08` and the I3D frame from `this+0x68`, and both are `C_actor` **base** fields that
`C_Vehicle` does not have. Corroboration: their vtable `0x0063bc08` also carries `AI__5C_carFUi`,
`Update__5C_carFUi`, `GameInit__5C_carFv` and `GetActorSpeed__5C_carFR8S_vector`.

**None of the six can have a PS2 twin** — the whole `C_recorder` subsystem is absent from
SLUS_206.71. The `-` in their twin column is a measurement, not a gap.

### 4b. `C_DropOut` — the parts a car sheds

A **`C_DropOut`** is an actor of `0x3b8` bytes, kind `0x11`, vtable `0x0063c320`, constructor
`dropout_ctor` (`0x0048a530`, PS2 twin `__ct__9C_DropOutFv` CONFIRMED). Two paths create one:

* the **live** damage path — `car_prepare_wheel_dropout` / `car_prepare_body_part_dropout` call
  `car_drop_out_part`, which allocates the actor, branches
  `if (selector == 1) InitWheel else InitBox` and registers it as a temporary actor;
* the **playback** path — `car_track_spawn_dropout_part` (slot 1), on a key of type 2, does the
  same for a recorded track: it binds the actor to the car part the key names, hides the part on
  the car body, registers it with `game_add_temporary_actor`, gives it its model from the key's
  name string and starts it.

The key's selector word `key+0x2c` splits into a low 3-bit part CLASS and a high index:

| Class | Resolved through | Which parts |
|---|---|---|
| 1 | the wheel list at `car+0xd24` | wheels |
| 2..7 | remapped to deform-zone types 2/5/7/4/6/8, then the `0x34`-stride deform-zone table at `car+0x308` (§4c) | bumper / light / licence plate / mirror / wing / door |

So a DropOut is either a **wheel** or a **body panel** — two initialisation paths, nothing else.
Which panels can come off depends on the model's zones: the shipped models tag bumpers, lights
and wings, and none tags a mirror, a licence plate or a door (§4c).
The kind byte is labelled by the game itself: `recorder_new_track` formats these tracks `drp_%d`
for kind `0x11`, and a debug name builder is labelled data.

**In game (user-observed, PC build):** a car can lose its **wheels, bumpers and lights**. That is
the PC's DropOut vocabulary. The PS2 build sheds **doors as well**; that difference is recorded in
§6.

**`C_DropOut`'s base is `C_PhThing`, not `C_actor` directly** — `dropout_ctor` chains to
`physical_object_ctor` (`0x004be910`), the engine's physics-body base. Both builds have exactly
**five** `C_PhThing` subclasses.

### 4c. The deform-zone table `car+0x308`

`car+0x308` / `car+0x30c` is `C_Vehicle+0x298` / `+0x29c`: a vector of **`0x34`-byte deform
zones**, one per breakable or deformable body part. It is built from the car's MODEL, not from
`tables/`: `vehicle_init` enumerates every mesh frame of the model (type mask `0x10001`, name `*`)
through `vehicle_cb_enum_deform_zones`, which reads the text between `[` and `]` in the frame's
user-parameter string and appends one zone per recognised tag:

| tag | zone type | shipped models carrying it |
|---|---|---|
| `DEFORM BODY` | 1 | 437 |
| `DEFORM BUMPER` | 2 | 178 |
| `DEFORM GLASS` | 3 | 381 |
| `DEFORM MIRROR` | 4 | 0 |
| `DEFORM LIGHT` | 5 | 193 |
| `DEFORM WING` | 6 | 2 |
| `DEFORM SPZ` (licence plate) | 7 | 0 |
| `DEFORM DOOR` | 8 | 0 |
| `DEFORM ROOF` | 9 | 48 |

The counts come from the 446 shipped `.4ds` models that carry any `[DEFORM ...]` tag. Two frames
spell the tag `DEFORMGLASS` without the space and are not recognised, so they get no zone.

| Zone offset | Type | Meaning |
|---|---|---|
| `+0x00` | `I3D_frame *` | **the model frame this zone is** — Ghidra types the cast directly (`this = (I3D_frame *)*puVar11`) |
| `+0x0c` | `short` | zone type, 1..9 (table above); type 3 (glass) takes its own handling path |
| `+0x0e` | flag byte | bit `1` = already detached (the gate `car_track_spawn_dropout_part` sets), bit `2` = damaged |
| `+0x18` | `float` | maximum integrity — the denominator of condition channel 3 (§1b) |
| `+0x20` | `float` | integrity / remaining health, clamped at 0 |

**Consumers**, receiver confirmed by vtable membership rather than proximity: `car_hit` and
`car_get_surface_material_id` are `C_car` vtable slots 31 and 32; `car_set_transparency` also
walks it; `car_prepare_body_part_dropout` switches on the zone type; `car_get_condition_fraction`
reads the last body zone; and `car_track_spawn_dropout_part` indexes it through
`vehicle_find_part_at_encoded_position`.

---

## 5. Related

`structs/vehicle.md` — the `C_Vehicle` physics/drivetrain sub-object at `car+0x70` ·
`police-system/artifacts/boosting.md` — the CARINIT model-level swap (`car_get_model_level` /
`car_set_model_level`) IS the boost mechanism · `def-files-docs/vehicles-bin.md` — the level
records · `structs/driver.md` — the chase `C_driver` AI that reads the high-level readouts ·
`structs/entity.md` — the entity kind field · `structs/car_extended.md` — the one subclass (§7) ·
`structs/actor_track.md` — the recorder interface (§4a) · `structs/dropout.md` — `C_DropOut` ·
`structs/glass_shatter.md` — `car_break_glass` and the window list · `structs/net_event_msg.md` —
the replication layer (§8) · `structs/force_feedback.md` — the five car effects ·
`def-files-docs/materialy.md` — the surface material ids `car_get_surface_material_id` returns.

---

## 6. PC vs PS2

This is the one section carrying PS2 offsets and cross-build reasoning. PC conclusions live in
§1–§4.

* **Embedded `C_Vehicle` base:** PC `car+0x70`, PS2 `car+0x80`. Field offsets differ throughout —
  the PS2 `C_car` is a larger struct — so the coverage table maps by ROLE, not raw offset.
* **Detachable parts: the PS2 sheds DOORS, the PC does not.** *(User in-game observation.)* On the
  **PC** a car can lose its wheels, bumpers and lights (§4b); on the **PS2** it can lose doors as
  well. The difference is MODEL DATA, not code: the PC handles a door zone (deform type 8,
  DropOut class 7) exactly like the other panels, both builds run the identical two-branch
  `if (selector == 1) InitWheel else InitBox` logic, and no shipped PC model carries a
  `[DEFORM DOOR]` tag (§4c). The PS2 side of that reading is open (§9).
* **Condition channels (`ShowCarEnergy`).** The PS2 arms are: 0 `+0x684 / +0x688` (resistance
  `+0x68c`); 1 `+0x658 / +0x65c` (resistance `+0x660`; exhaustion latches `+0x22a8` and de-rates
  `+0x654 = +0x650 + (1 − +0x650)·(pool/max)`); 2 `(+0x238c <= 0) ? 0 : +0x239c / +0x2394`
  (resistance `+0x2398`, `+0x238c` an enabled gate); 3 the mean of `part+0x24 / part+0x1c` over the
  body-part array (`+0x6a0` count, `+0x6a4` base, stride `0x3c`) counting parts with
  `part+0x3a == 1`. Channel 1 carries the same de-rate on both builds (§1b). The PC arm 2 has no
  enabled gate, and the PC arm 3 reads one body zone where the PS2 averages every flagged part.
* **`C_DropOut` cross-build material:**
  * The two-path split is corroborated by the PS2 class itself: `C_DropOut` exposes exactly two
    initialisers, `InitWheel__9C_DropOutFP15S_DROPOUT_WHEEL` and
    `InitBox__9C_DropOutFP13S_DROPOUT_BOX` — matching the PC's `selector == 1` wheel branch versus
    the body-panel branch. Those two parameter types name PC structures the PC decomp cannot
    supply.
  * `C_PhThing`'s method set (the shared base): `MoveWheel` (6276 B), `FindActors`, `Init`,
    `GameInit`, `GameDone`, `AI`, `Update`, `ChangeState`. `__ct__9C_PhThingFv` is what identifies
    PC `physical_object_ctor`.
  * PS2 `C_DropOut` is **0x650 bytes** against the PC's **0x3b8**.
* **The `C_recorder` track interface is PC-ONLY.** All six `car_track_*` overrides (§4a) have no
  PS2 counterpart — no class symbol, and none of the subsystem's debug strings exist in
  SLUS_206.71. A measurement, not a failed search.
* **Model-level fields:** current level PC `car+0x20ec` = PS2 `car+0x242c`; the level table is PC
  `car+0x20f4`/`+0x20f8` (begin/end, stride `0x113c`) against a PS2 **cached count** at
  `car+0x2434` and a table pointer at `car+0x2438` (stride `0x1258`); the ratio inputs PC
  `+0x660`/`+0x564`/`+0x6e0`/`+0x69c` = PS2 `+0x8b4`/`+0x794`/`+0x940`/`+0x904`.
* **Seat arrays.** The record array is PC `car+0x210c` stride `0xa0`, PS2 `car+0x2444` stride
  `0xd0` — a larger per-seat record on the PS2. Seat guard fields: PC seat `+0xc` byte / `+0x10`
  int → PS2 seat `+0xc4` byte / `+0x30` int; placement sources PC seat `+0x84`/`+0x90` → PS2 seat
  `+0x10`/`+0x20`. The second, parallel array is PC `car+0xd90`/`+0xd94` stride `0x24` → PS2
  `car+0xfd0`/`+0xfd4` stride `0x30`; `ai_fight_followship_count_free_seats` body-diffs against
  that pair. **There are TWO parallel per-seat arrays sharing one index space** — §1's `+0xd90`
  and `+0x210c` rows.
* **Cheap accessor offsets (§2):** `SetDriver`/`GetDriver` byte PC `car+0x3b0` = PS2 `car+0x2299`;
  `SetTrafficCar` byte PC `car+0x20bd` = PS2 `car+0x2460`; `GetActorSpeed` vec3 PC `car+0x1f7c` =
  PS2 `car+0x460`; `SetTimeParking` writes PC `car+0x2010` = PS2 `car+0x23a8` (the PS2 from
  `*(act_mission+0x20)+0xe54`), and `GetTimeParking` returns the delta.
* **Drawbridge flag `+0x294`** — what CONFIRMED `human_find_blocking_drawbridge`: the PC renders
  the test as `(*(uint *)(car + 0x294) >> 0x19 & 1) != 0`, the PS2 as
  `(*(uint *)(car + 0x648) & 0x2000000) != 0` — the same bit 25, differently decompiled. So PC
  `car+0x294` = PS2 `car+0x648`.
* **Truck cargo-bed fields:** PC `car+0x2214` = PS2 `car+0x24b0` (enabled); PC `car+0x2230` = PS2
  `car+0x24b4` (frame).
* **Chase/HUD readout equivalents (§1a):** light/control flags PC `car+0x20c` = PS2 `car+0x5f8`;
  engine/siren sound PC `car+0x51c` = PS2 `car+0x229c`; scalar speed PC `car+0x688` = PS2
  `car+0x8f0`; accel constant PC `car+0x6e0` = PS2 `car+0x940`; max steer PC `car+0x6fc` = PS2
  `car+0x95c`.
* **The method count is an UPPER BOUND.** There are 89 PS2 `C_car` methods; some are inlined on
  the PC, live in a second PC cluster, are virtual-only, or are PS2-port-only. Do not treat 89 as
  a target.

---

## 7. `C_car_ex` — the EXTENDED car, this class's one subclass

`C_car_ex` (PC prefix `car_extended_`, doc `structs/car_extended.md`) extends `C_car`. It is a
plain single-inheritance subclass, and the vtables say so precisely:

* its table `0x0063bd68` is **41 slots — the same width as `C_car`'s `0x0063bc08`** — and the
  slots are **aligned one-for-one with no shift**;
* **exactly 8 slots differ** (14, 16, 18, 19, 20, 21, 25, 26), which are exactly the 8 methods
  `C_car_ex` overrides on the PS2 side: `Update`, the scalar deleting destructor, `Init`,
  `SaveData`, `LoadData`, `DuplicateFrom`, `GameInit`, `GameDone`;
* it keeps `kind +0x10 = 4`, so every "is this a car" test in the engine still passes. The
  discriminator is the `C_actor`-level flavour field **`+0x14`**: `C_actor` = 1, `C_human` = 2,
  **`C_car_ex` = 3** (`structs/entity.md` §3a). `human_game_done` gates on
  `kind == 4 && +0x14 == 3`.

`sizeof(C_car_ex)` is `0x2254` against `C_car`'s `0x21f4`; the extra `0x60` is four vector triples
plus the `~DCS` spot. The entity factory builds it at `case 0x24`.

---

## 8. The `C_car` replication band

`C_car`'s multiplayer code does **not** live in the `C_car` cluster. It is split across the
event-message translation unit `[0x005929c0, 0x00594740)` and the sender cluster at
`[0x0059ab90, 0x0059d9d0)` — the clearest instance in this project of *a class's PC code is not
one contiguous cluster*. `car_apply_net_event_msg` sits 1.1 MB away from the rest of the class.

| PC VA | PC name | slot | role |
|---|---|---|---|
| 0x0059abc0 | `car_net_snapshot_state` | 39 | Latches the transform as the dead-reckoning baseline |
| 0x0059ad60 | `car_net_apply_remote_state` | 15 | `NetDirect(Ui)` — extrapolates a REMOTE car between state messages. 4928 B |
| 0x0059c0a0 | `car_broadcast_state_net_event_msg` | - | On the host, the full snapshot to every near peer (`g_net_session+0xb4`) and, on one call in three, the reduced one to every far peer (`+0xc4`); a client sends to the host only. `car_update` calls it when bit 1 of `+0x2198` is set or the session's update interval runs out |
| 0x0059c200 | `car_post_reduced_state_net_event_msg` | - | The cheap payload for far peers, types `0x21` / `0x23` (`structs/net_event_msg.md` §4) |
| 0x0059c3a0 | `car_post_state_net_event_msg` | - | Builds one type-`0x1e`/`0x1f` snapshot |
| 0x0059c690 | `car_apply_net_event_msg` | 38 | Receive: applies types `0x29` / `0x2a` / `0x2b` and the glass arms ¹ |
| 0x0059d2f0 | `car_post_break_glass_net_event_msg` | - | Type `0x27`, from `car_break_glass` |
| 0x0059d3a0 | `car_post_hit_net_event_msg` | - | Type `0x2c`, from `car_hit` |
| 0x0059d410 | `car_post_part_dropout_net_event_msg` | - | Type `0x2d`, from `car_drop_out_part` |
| 0x0059d4d0 | `car_post_dirty_net_event_msgs` | - | Flushes `car+0x1e8` dirty bits 2/4/8 |
| 0x0059d640 | `car_post_collision_damage_net_event_msg` | - | Type `0x2b`, then clears the queue |
| 0x0059d740 | `car_get_collision_damage_records` | - | One line, `return car + 0x21d4` |
| 0x0059d750 | `car_queue_collision_damage_record` | - | Pushes one record onto the `+0x21d4` queue |
| 0x0059d950 | `car_post_explosion_net_event_msg` | - | Type `0x24`, from `car_explosion` |

¹ Ghidra renders its switch selector as a float, so the case labels print as `5.74532e-44`
(= `0x29`) and friends — read the raw integers.

**The dead-reckoning band**, latched by slot 39 and refreshed by slot 38: `+0x2164` position,
`+0x2170` linear velocity, `+0x217c` quaternion, `+0x218c` correction accumulator, `+0x219c`
snap-back threshold, `+0x21a8` angular velocity, `+0x21b4` damping coefficients, `+0x21c0`
received velocity, `+0x21cc` message timestamp, `+0x21d0` extrapolation window, `+0x21e4`
correction velocity, `+0x21f0` the broadcast counter that raises bit `0x100` of `+0x2198` on one call in three.

**Slot 15 is `NetDirect(Ui)` family-wide** — `structs/bottle.md` §3a byte-proves the role on
`C_airplane`, whose slot 15 is a bare `ret 4`. It extrapolates a remotely-owned car; the velocity
it integrates arrives over the wire.

**Slot 39 exists only in the car family.** `C_PhThing`'s and `C_bottle`'s vtables end at slot 39
— the next dword is float data — while `C_car`'s `0x0063bc08` and `C_car_ex`'s `0x0063bd64` carry
one more. `car_net_snapshot_state` has **0 textual callsites and 2 datarefs** — pure virtual
dispatch — and `car_game_init` calls slot `0x9c`.

**`C_car+0x21d4` is the collision-damage queue.** `car_ctor` empties four owned vectors and
`car_dtor` frees them; `+0x21d4` is a **queue of heap `{S_vector position, S_vector direction,
u32}` records**, pushed by `car_queue_collision_damage_record` whose **sole caller is
`car_on_collision_apply_damage`**, drained once per frame into the type-`0x2b` message and then
freed by `car_clear_collision_damage_records`. That producer is invisible to every decomp text
search; it was found by a disassembly scan of the whole `.text` for the displacement `0x21d4`
across 719 589 instructions.

**What a car snapshot carries.** `car_post_state_net_event_msg`'s payload maps one-for-one onto
`event_msg_car_state_serialize`: the body quaternion from `frame+0x9c`, a door/window bitmask
built from the part flags `0x10`/`0x20` over `car+0xd24`, the gear index `car+0x648`,
`car_pack_light_state`, the GUID `car+0x18`, and two velocity triples scaled by `car+0x20b4`. The
type is chosen bit-wise from `car+0x2198`: `0x1f` on the one call in three that also feeds the far
peers, `0x1e` otherwise.

Full write-up of the message layer: `structs/net_event_msg.md`.

---

## 9. Open items

| item | state |
|---|---|
| PS2 door drop-outs (§6) | attributed to model data because no shipped PC model carries a `[DEFORM DOOR]` tag; the PS2 model archives were not examined |
