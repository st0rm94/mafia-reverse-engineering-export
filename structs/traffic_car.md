# `C_traffic_car` — entity kind `0xc`, the ambient-traffic MANAGER

**Created** 2026-08-17 15:33 +0200 · **Modified** 2026-09-23 15:55 +0200

| | |
|---|---|
| PC prefix | **`traffic_car_`** |
| PC kind (`entity+0x10`) | **`0xc`** |
| PC vtable | `0x0063ce20` |
| PC constructor | `traffic_car_ctor` @ `0x004f65a0` |
| PC translation unit | `[0x004f62a0, 0x00508120)` — 77 function starts, all named |
| PS2 class | `C_traffic_car` (`13C_traffic_car`), source file `C_trf_c.cpp`; 49 methods, all accounted (§8) |

## 1. What it is — and what it is not

🔴 **It is not "an ambient car".** The name reads like one and the class is one actor, but
that actor is the **manager of the whole cruising population**. A single instance owns:

* the `S_ELEMENT` records — the individual cruising cars (`this+0x74`, count `this+0x220`,
  mirrored by the file-scope `g_traffic_car_registry`);
* the `S_MAFIAN` crew records — spawned gangster crews, stride `0xc`
  `{car, driver, passenger}` (`g_traffic_mafia_crews_begin` / `_end` / `_cap`, vector at
  `0x0065616c`), pushed by `traffic_car_spawn_mafia_pursuit_car` and retired by `traffic_car_update`;
* the adopted element records — `S_ELEMENT *` pointers (`g_traffic_adopted_elements_begin` /
  `_end` / `_cap`, vector at `0x006561c0`), pushed by `traffic_car_adopt_car_with_new_driver`;
* the `S_BRIDGE` slots — the traffic-side view of the drawbridges
  (`g_traffic_bridge_slots`, stride 8, `{actor*, ushort node}`);
* the `S_LOCKCROSS` records — per-element crossing locks, PC stride **`0x22`**;
* the parked-car container `g_traffic_parking_cars` and the stolen-car list
  (`g_traffic_stolen_cars`), which holds the `C_car *` itself.

The individual ambient **pedestrians** are a different class entirely
(`C_traffic_element`, PC prefix `pedestrian_`, `structs/pedestrian.md`); this class is
their vehicular counterpart, and the two meet in
`traffic_car_seat_pedestrian_as_driver`.

🔑 **It is also where other subsystems' cars come from.** Police reinforcement cars, mafia
hit-cars and replacement cruisers are all spawned here, so methods that serve the police and
gang subsystems (`traffic_car_activate_or_spawn_pursuit_vehicles`,
`traffic_car_spawn_mafia_pursuit_car`, `traffic_car_adopt_car_with_new_driver`,
`traffic_car_destroy_crew_for_car`) carry this class's prefix, not the consumer's.

## 2. Instance and global state

| where | what | evidence |
|---|---|---|
| `+0x10` | entity kind `0xc` | ctor |
| `+0x74` / `+0x220` | the `S_ELEMENT*` vector and its count | walked by AI, Update, GameDone, `traffic_car_deactivate_adopted_elements` and both arms of `traffic_car_destroy_crew_for_car` |
| `+0x23c` / `+0x240` | the manager's query position (x, z) | `traffic_car_get_next_pos_info` |
| `+0x248` | the query direction | same |
| `+0x254` | the road-node table base (stride `0xec`) | `traffic_car_clear_lane_blocking` |
| `S_ELEMENT+0x0` | the active flag | `traffic_car_update`, `traffic_car_deactivate_adopted_elements` |
| `S_ELEMENT+0xc` | the element's `C_car` | the key every per-car lookup matches on |
| `S_ELEMENT+0x14` | the element's `C_driver` | destroyed by `traffic_car_destroy_crew_for_car` |
| `S_ELEMENT+0x1c` / `+0x20` | the two occupant models | destroyed by the same |
| `S_ELEMENT+0x24` | an embedded `C_animations_machine_de_luxe` | built by `traffic_car_element_ctor`; reached through the adjustor thunk `thunk_stub_noop_cdecl_00574a40` |
| `S_ELEMENT+0x14c/+0x14d/+0x14e` | state bytes set by the seat / steal / release paths | the three methods that write them |
| `S_ELEMENT+0x154` | the ADOPTED flag | §2a |
| `S_ELEMENT+0x1b0` | the element's `S_LOCKCROSS` array | `traffic_car_clear_lane_blocking` |

The `S_ELEMENT` record is `0x158` bytes. `traffic_car_element_ctor` (`0x00503e40`,
`lea ecx,[esi+0x24]; call anim_machine_ctor; mov eax,esi; ret`) is its default constructor,
handed to `crt_ehvec_ctor(base, 0x158, n, …)` inside `traffic_car_game_init` as a function
pointer, so it has no direct callsite.

**Six file-scope `std::vector` statics**, each with its own MSVC init/ctor/atexit/dtor
quartet and one bit of the shared run-once word `g_traffic_car_static_dtor_guard`:

| guard bit | address | what |
|---|---|---|
| `0x01` | `0x006561c0` | the adopted `S_ELEMENT *` records (`g_traffic_adopted_elements_*`) |
| `0x02` | `0x006561ac` | `g_traffic_bridge_slots` |
| `0x04` | `0x0065619c` | `g_traffic_car_registry` |
| `0x08` | `0x0065618c` | `g_traffic_parking_cars` |
| `0x10` | `0x0065617c` | the stolen cars, `C_car *` (`g_traffic_stolen_cars`) |
| `0x20` | `0x0065616c` | the crew records, `S_MAFIAN` (`g_traffic_mafia_crews_*`) |

Two counters, each with exactly one incrementer and one release:

| global | incremented by | released by |
|---|---|---|
| `g_traffic_cop_car_count` | `traffic_car_activate_or_spawn_pursuit_vehicles` | `traffic_car_release_cop_car_slot` |
| `g_traffic_mafia_car_count` | `traffic_car_spawn_mafia_pursuit_car` | `traffic_car_release_mafia_car_slot` |

### 2a. `S_ELEMENT+0x154` — the ADOPTED flag

Decoded from its setter and its consumer:

* **Set** by `traffic_car_adopt_car_with_new_driver` immediately after
  `operator_new(0x158)` — the path that takes over a car already in the world and gives it
  a fresh `C_driver` and anim machine.
* **Read** by `traffic_car_deactivate_element` as the last thing it does: clear ⇒ null the
  slot and return, leaving the pooled record alone; set ⇒ delete the driver and both
  driver-man models, `car_set_owner(0,0)` / `(0,1)`, `car_set_is_traffic_car(0)`, hand the
  car back to the police pool, run the `+0x24` anim-machine destructor and
  `operator_delete` the record.

⇒ **`+0x154` means "this record is heap-owned — the manager ADOPTED this car rather than
drawing it from a pool"**, and deactivation must free the record and give the car back.
`traffic_car_deactivate_adopted_elements` (`0x00506bb0`) is the scene-teardown pass that
retires exactly those: it deactivates each element with `+0x0` and `+0x154` both set, and takes
no parameter (it `ret`s popping 0). `traffic_car_update` guards on the same pair,
`*elem != 0 && elem[0x154] == 0`.

## 3. Vtable `0x0063ce20`

Slots 0–12 are the `C_actor_track` base (`structs/actor_track.md`). The class's own:

| slot | function | note |
|---|---|---|
| 13 | `traffic_car_ai` | - |
| 14 | `traffic_car_update` | - |
| 15 | `stub_noop_args4_00461180` | `NetDirect`, shared by 30 vtables — no twin stamp |
| 16 | `traffic_car_scalar_deleting_dtor` | MSVC-only |
| 18 | `traffic_car_init` | own override, not the shared default |
| 19–21 | `traffic_car_save_data` / `_load_data` / `_duplicate_from` | - |
| 22–24 | the `traffic_car_save_game_*` triple | - |
| 25 / 26 | `traffic_car_game_init` / `_game_done` | - |
| 27 | `traffic_car_change_state` | - |
| 29 | `stub_noop_cdecl_00574a40` | `Death`, shared by 36 vtables — no twin stamp |
| 31 | `traffic_car_hit` | own override |
| 33 | `traffic_car_is_possible_coll` | - |
| 39 | `traffic_car_collision` | 🔑 **exactly 1 dataref** — this class's own, not a shared stub |

## 4. The routing half

`traffic_car_ai` is the engine. Per frame it activates and retires elements by distance,
drives each along its path, and claims/releases crossings:

```
traffic_car_ai
  ├─ traffic_car_activate_elements   (0x004f76d0, §6)
  │    ├─ traffic_car_get_next_pos_info  ── chooses the next node + lane
  │    │    └─ traffic_car_find_last_crossing
  │    │       (FindNodesIndex is INLINED here — marked in the decomp)
  │    └─ traffic_car_prepare_next_direction   (also inlined a second time here)
  ├─ traffic_car_deactivate_element  ×3
  ├─ traffic_car_lock_crossing       ×2
  └─ traffic_car_clear_lane_blocking ×2
```

`traffic_car_clear_lane_blocking` is what byte-proves the `S_LOCKCROSS` stride: a nested
4×4 walk decrementing per-lane blocking counters on the node the element is leaving.

## 5. Inlining on the PC

* **`traffic_car_prepare_next_direction` is inlined once and called once.** MSVC spliced a copy
  into `traffic_car_activate_elements` (whose body makes two `vec2_signed_angle_diff` calls where
  the out-of-line copy makes one) and kept the out-of-line body for `traffic_car_ai`. That is why
  the out-of-line copy's caller is `traffic_car_ai`.
* **`FindNodesIndex` is inlined into `traffic_car_get_next_pos_info`**, marked in the decomp with a
  `// ===== INLINED ON PC:` banner (call-count proof in §8).

## 6. `traffic_car_activate_elements` — the activation pass

**14 429 bytes**, the second-largest function in the TU. It walks the `S_ELEMENT` vector
(`this+0x74`, count `this+0x220`, cap `this+0x218`) and brings cruising cars in and out of
existence around the query point: per element it picks the next road node and lane, seats a
driver, applies the car's CARINIT parameters, steers, starts the engine and fuels it, and retires
out-of-range elements through `traffic_car_deactivate_element` (9 callsites). It also claims
bridge slots and consults the railway list, so a spawn never lands on a drawbridge or a level
crossing. Signature `(this, char mode, int index)`; the only caller, `traffic_car_ai`, passes
`(0, -1)` — `-1` = every element.

**Why the stock Ghidra export has no body.** The prologue is `and esp,0xfffffff8` (stack
realignment) + an SEH frame + `mov eax,0x125c; call __chkstk`. `__chkstk` (`crt_chkstk`,
`0x00624fa0`) page-probes and then does `mov esp,ecx`, so ESP changes inside a callee by a
register-supplied amount on a base that is no longer an affine function of the entry ESP; Ghidra
models a call as a constant stack purge and fails with *"Low-level Error: Cannot properly adjust
input varnodes"*. 17 functions call `__chkstk` and all others decompile; this is the only one
that also realigns ESP (`and esp,0xfffffff8` occurs 5 times in the image, no other near a probe
call). The body in the annotated decomp was produced by naming `0x00624fa0` `__chkstk` and marking
it inline, which enables Ghidra's built-in `alloca_probe` p-code injection.

## 7. Globals near the class

| address | what |
|---|---|
| `0x00656154` / `0x00656158` / `0x0065615c` | `g_traffic_car_elems_coef` / `g_traffic_car_mafia_coef` / `g_traffic_car_police_coef`, seeded to `1.0f` by the ctor; `traffic_car_game_init` scales the element count by the first and weights the mafia / police spawn entries by the other two |
| `0x00656160` | `g_traffic_car_activation_suspended` — `traffic_car_ai` skips `traffic_car_activate_elements` while it is set; `traffic_car_game_init` clears it twice and nothing on the PC sets it |
| `0x00656161` | `g_traffic_cars_enabled` |

## 8. PC vs PS2

**Method accounting — 49 PS2 methods:**

| outcome | count | which |
|---|---|---|
| own PC body, body-diffed | 40 | everything named in §3, §4 and §6 |
| folds onto a shared stub | 3 | `Death` (36 vtables), `NetDirect` (30), `DejSiZatimSlofiiika` |
| inlined on the PC | 1 | `FindNodesIndex`, into `GetNewPosInfo` (§5) |
| PS2-port-only | 2 | `ChangeSceneInit`, `ChangeSceneDone` — city streaming, `InterCity` occurs 0× on PC |
| not located | 1 | `InvalidateActor` (§9) |
| second PS2 cluster | 1 | `DajDevatemuTyZkurvenyZmrdyAtFurtNevotravuje` — 12 B at PS2 `0x0032ea90`, sole caller `actionTick_2__8C_entity`; its PC twin is in the `C_entity` band, not this TU |
| PC-only | 1 | the MSVC scalar deleting destructor |

`traffic_car_element_ctor` is also PC-only: the PS2 holds the anim machine as a POINTER at
element `+0x0`, so no element constructor was emitted there.

**`ActivateElems__13C_traffic_carFbi` ↔ `traffic_car_activate_elements` (CONFIRMED).** An `E8`
scan of the PC byte range reproduces the PS2 callee multiset: `DeactivateElem` ×9↔×11,
`GetNewPosInfo` ×1, `GetNearNode` ×1, plus `car_deactivate`, `car_reset`, `car_set_owner`,
`car_set_enter_protect`, `driver_reset`, `vehicle_set_steer`, `vehicle_set_fuel`,
`vehicle_apply_carinit_params`, `car_set_has_driver` ×2, `UpdateWMatrixProc` ×1,
`S_vector::operator*` ×2, `GetWorldPos` ×1 — every one 1:1. The body diff agrees: same
`count <= cap` early-out, the element-pointer vector (PC `+0x74` / PS2 `+0xa0`), the counters
(PC `+0x220` / `+0x218`, PS2 `+0x24c` / `+0x244`), the `param < 0` = all-elements branch, the
`GetNewPosInfo` call at the same position, and the sole PC callsite's `(0, 0xffffffff)` against
the PS2 `(bool, int)` signature.

**`FindNodesIndex` inlining proof.** The PC `traffic_car_get_next_pos_info` makes **five**
`S_vector::AngleTo` calls — exactly PS2 `GetNewPosInfo`'s three plus `FindNodesIndex`'s two. On the
PS2, `PrepareNextDir` has a single caller, `ActivateElems`.

**The ADOPTED flag on the PS2.** `NoTakMiTedaDajTuPleckuJaJuOdvezu` (↔
`traffic_car_adopt_car_with_new_driver`) sets `+0x33` after `__nw__FUi(0x38)`; `DeactivateElem`'s
`+0x33` arm is the same free-and-return sequence, `__dl__FPv` included. `Update` guards on
`+0x34 != 0 && +0x33 == 0`, which pins **PC `S_ELEMENT+0x0` = PS2 `+0x34` = the active flag**.
`TakATedSiVasKupim_Vsecky__13C_traffic_carFb` ↔ `traffic_car_deactivate_adopted_elements` matches
statement for statement, but the PS2 takes a bool choosing between "elements with `+0x34`" and
"elements with `+0x34` and `+0x33`", and the PC implements only the `param == 0` arm.

**Record-layout differences, measured:**

* `S_ELEMENT` is `0x158` B on the PC against `0x38` B on the PS2, the difference being the
  **embedded** `C_animations_machine_de_luxe`. The flag band is not a uniform shift:
  `+0x30`↔`+0x150` is `+0x120`, `+0x33`↔`+0x154` is `+0x121`.
* `S_LOCKCROSS` is **`0x22` bytes on the PC and `0x32` on the PS2**, and `ClearBlocking`
  decrements **two** per-lane counters on the PC against **three** on the PS2. The road-node
  record is `0xec` on the PC and `0xe0` on the PS2.
* The path-node array is stride `0x70` on the PC and `0x90` on the PS2; the crossing table
  entry is `0x18` PC / `0x20` PS2.
* The ctor's kind write is byte-identical on both builds.

**The dev-humour names, decoded from behaviour rather than transliterated.** Two of them name a
global: *chlupatej* = a cop → `g_traffic_cop_car_count`; *mafián* = a gangster →
`g_traffic_mafia_car_count`. *Čórovačka* = a theft, which corroborates
`traffic_car_release_stolen_car`. The rest (`HulaaaaaMuf`, `Krles`,
`NoTakMiTedaDajTuPleckuJaJuOdvezu`, `SebraliMiKaaaruSvine`, `ChodDoRici`,
`DostalJsemRanuKachlou`, `TakATedSiVasKupim_Vsecky`, `ChcesMeSejmout`,
`VypraskeeeKaaaru`, `DejSiZatimSlofiiika`) carry no behaviour at all, and every PC name was
derived from the body and the caller/callee pairs.

**Caller pairs that match one-for-one across builds:**

| PC | PS2 |
|---|---|
| `game_set_traffic_visible` | `SetTrafficVisible__6C_game` |
| `human_do_throw_occupant_from_car` | `Do_ThrowCocotFromCar__7C_human` |
| `pedestrian_advance_to_next_node` | `SetNewNode__17C_traffic_element` |
| `pedestrian_tick` | `Tick__17C_traffic_element` |
| `car_hit` | `Hit__5C_car` |
| `car_game_done` | `GameDone__5C_car` |
| `game_done` | `Done__6C_game` |
| `ai_group_get_closest_usable_car` | `GetCarClosestAndUsable__8ai_group` |
| `pedestrian_manager_try_send_pedestrian_to_parked_car` | `TryGoToAssHole__19C_traffic_generator` |
| `police_manager_npc_list_push` / `_erase` / `_tick` | `AddDoRiciActor` / `DelDoRiciActor` / `TickDoRiciActors__10C_shvestky` |

## 9. Open items

| item | state |
|---|---|
| PS2 `InvalidateActor__13C_traffic_car` (548 B) | no PC start in its order position and not a vtable slot; `C_traffic_generator` has the same gap |
| PS2 `TakATedSiVasKupim_Vsecky` "deactivate ALL" arm | reached from the garage explosion on the PS2; no PC counterpart located |
| in game: does the cop-car counter cap reinforcements? | `g_traffic_cop_car_count` is watchable in Cheat Engine and the deploy path increments it once per car |
