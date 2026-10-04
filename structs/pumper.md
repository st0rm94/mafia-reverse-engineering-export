# `C_pumper` — the gas-station attendant

**Created** 2026-07-13 11:46 +0200 · **Modified** 2026-09-30 23:08 +0200

The NPC who stands at the fuel pump, walks out to a stopped car, refuels it, charges you
for it, and smokes and fidgets in between. Czech *pumpa* = fuel pump.

- **PC prefix:** `pumper_`.
- **Base class:** `C_actor` (`structs/actor.md`). `pumper_ctor` → `actor_ctor`.
- **kind:** `0x19` (`structs/entity.md` §1), re-confirmed from inside the class by
  `pumper_script_can_pump`'s own `kind == 0x19` literal.
- **PC vtable:** `.rdata 0x0063ca58`, 39 slots.
- **Embeds** `C_animations_machine_de_luxe` at **`+0xe0`** (`structs/anim_machine.md` §1).
- **PC translation unit:** **`[0x004ce000, 0x004d33a0)`** — **36 function starts**,
  bounded at both ends by a CRT static-init/atexit pair.

---

## 1. Coverage — 36/36 PC starts named, 37/37 PS2 symbols accounted

37 = 36 `C_pumper` methods + 1 free function. Minus `Init` and `Death` (both folded onto
shared PC stubs), plus the MSVC scalar deleting destructor = **36 PC starts**.

The addresses `0x004d2c5d`, `0x004d2e9f` and `0x004d2eb1` are mid-body addresses, not function
starts. `pumper_save_game_save` and `pumper_save_game_load` are 576 B and 912 B (PS2 716 B /
948 B), and the SaveGame triple lands on PC starts 29/30/31 against PS2 methods 29/30/31 — the
alignment's cleanest corroboration.

| PC function | VA | slot | PS2 method | ratio | grade |
|---|---|---|---|---|---|
| `pumper_ctor` | 0x004ce000 | - | `__ct__8C_pumperFv` | 0.71 | CONFIRMED |
| `pumper_scalar_deleting_dtor` | 0x004ce1a0 | 16 | - | - | STRUCTURAL ¹ |
| `pumper_dtor` | 0x004ce1c0 | - | `__dt__8C_pumperFv` | 0.86 | CONFIRMED |
| `pumper_save_data` | 0x004ce220 | 19 | `SaveData__8C_pumperFP7C_chunk` | 0.78 | CONFIRMED |
| `pumper_load_data` | 0x004ce300 | 20 | `LoadData__8C_pumperFP7C_chunk` | 0.69 | CONFIRMED |
| `pumper_duplicate_from` | 0x004ce500 | 21 | `DuplicateFrom__8C_pumperFP7C_actor` | 1.60 | CONFIRMED |
| `pumper_game_init` | 0x004ce540 | 25 | `GameInit__8C_pumperFv` | 0.68 | CONFIRMED |
| `pumper_game_done` | 0x004d0180 | 26 | `GameDone__8C_pumperFv` | 0.90 | CONFIRMED |
| `pumper_on_collision` | 0x004d03e0 | 30 | `Collision__8C_pumperF…` | 0.60 | CONFIRMED |
| `pumper_hit` | 0x004d0460 | 31 | `Hit__8C_pumperF…` | 0.48 | CONFIRMED |
| `pumper_ai` | 0x004d04d0 | 13 | `AI__8C_pumperFUi` | 0.70 | CONFIRMED |
| `pumper_ai_walk` | 0x004d0c10 | - | `AI_walk__8C_pumperFUi` | 0.80 | CONFIRMED |
| `pumper_ai_turn` | 0x004d0e80 | - | `AI_turn__8C_pumperFUiRC8S_vector` | 0.75 | CONFIRMED |
| `pumper_update` | 0x004d1000 | 14 | `Update__8C_pumperFUi` | 0.86 | CONFIRMED |
| `pumper_snap_to_ground` | 0x004d12d0 | - | `ground__8C_pumperFv` | 0.85 | CONFIRMED |
| `pumper_set_anim` | 0x004d1380 | - | `SetAnim__8C_pumperFf` | 0.64 | CONFIRMED |
| `pumper_choose_next_action` | 0x004d13f0 | - | `ChooseNextAction__8C_pumperFv` | **1.01** | CONFIRMED |
| `pumper_advance_to_next_node` | 0x004d1bb0 | - | `NextNode__8C_pumperFv` | 0.91 | CONFIRMED |
| `pumper_start_refuel` | 0x004d1c30 | - | `Start__8C_pumperFP5C_car` | 1.36 | CONFIRMED |
| `pumper_can_pump` | 0x004d2070 | - | `CanPump__8C_pumperFP5C_car` | 1.04 | CONFIRMED |
| `pumper_notify_callback` | 0x004d2460 | - | `pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` | 1.15 | CONFIRMED |
| `pumper_start_pump_fuel` | 0x004d2840 | - | `StartPumpFuel__8C_pumperFv` | 0.55 | CONFIRMED |
| `pumper_end_pump_fuel` | 0x004d2870 | - | `EndPumpFuel__8C_pumperFv` | 0.87 | CONFIRMED |
| `pumper_init_sounds` | 0x004d28c0 | - | `InitSounds__8C_pumperFv` | 0.42 | CONFIRMED |
| `pumper_init_sound` | 0x004d28e0 | - | `InitSound__8C_pumperFUi` | 0.70 | CONFIRMED |
| `pumper_release_sounds` | 0x004d29a0 | - | `ReleaseSounds__8C_pumperFv` | 0.55 | CONFIRMED |
| `pumper_get_special_anim_name` | 0x004d29e0 | - | `GetSpecAnimName__8C_pumperF18E_anim_pumper_spec` | 0.88 | CONFIRMED |
| `pumper_panic` | 0x004d2a70 | - | `PumperPanic__8C_pumperFv` | 0.93 | CONFIRMED |
| `pumper_save_game_get_size` | 0x004d2c30 | 22 | `SaveGameGetSize__8C_pumperFv` | 0.50 | CONFIRMED |
| `pumper_save_game_save` | 0x004d2c40 | 23 | `SaveGameSave__8C_pumperFPPv` | 0.80 | CONFIRMED |
| `pumper_save_game_load` | 0x004d2e80 | 24 | `SaveGameLoad__8C_pumperFPPv` | **0.96** | CONFIRMED |
| `pumper_play_sound` | 0x004d3210 | - | `PlaySound__8C_pumperFUib` | 0.71 | CONFIRMED |
| `pumper_release_colls` | 0x004d3240 | - | `ReleaseColls__8C_pumperFv` | 1.17 | CONFIRMED |
| `pumper_is_pumping` | 0x004d32b0 | - | `IsPumping__8C_pumperFv` | 0.53 | CONFIRMED |
| `pumper_is_in_panic` | 0x004d32d0 | - | `IsInPanic__8C_pumperFv` | 1.00 | CONFIRMED |
| `pumper_script_can_pump` | 0x004d32e0 | - | `ScriptCanPump__8C_pumperFUi` | 0.67 | CONFIRMED |
| `actor_subclass_init_require_frame` *(shared)* | 0x004ae890 | 18 | `Init__8C_pumperFP9I3D_frame` | - | STRUCTURAL ² |
| `stub_noop_cdecl_00574a40` *(shared)* | 0x00574a40 | 29 | `Death__8C_pumperFv` | - | STRUCTURAL ² |
| `stub_noop_args4_004776f0` *(shared)* | 0x004776f0 | 15 | `NetDirect` | - | STRUCTURAL ² |

¹ MSVC-only. ² A shared stub: the slot's role, not a twin (§7).

The TU also contains `std_vector8_erase_at` (`0x004d3330`), a shared STL helper with 82
callsites — instantiated here, owned by nobody.

## 2. `pumper_notify_callback` — the free function in the TU

One PC start matches no `C_pumper` method: `pumper_notify_callback` is a **free** function (§7).
It sits between `pumper_can_pump` and `pumper_start_pump_fuel`.

It is the seam between the animation and the gameplay. `pumper_game_init` registers it on
the attendant's `"notify"` child frame; the animation machine fires message ids out of the
clip and the callback turns them into actions:

| message | effect |
|---|---|
| `0x2a` | `pumper_start_pump_fuel` — **the fuel transfer begins here** |
| `0x2b` | `pumper_end_pump_fuel` — and ends here |
| `1`, `2` | sound slot 0 |
| `0x2e` / `0x2f` / `0x30` / `0x31` | sound slots 1 / 5 / 2 / 4 |
| `0x3c` / `0x3d` | show the hose (`+0x23c`) / the cigarette (`+0x238`) |

⇒ The refuel is **animation-driven**, not timer-driven: the money changes hands when the
clip reaches its notify frame.

## 3. What the attendant actually does

* **Idle.** `pumper_choose_next_action` parks him at his post node (`+0x7c`), where
  `pumper_get_special_anim_name` supplies the fidget and smoking clips —
  `Nuda01/02/03/NudaStativ` (*nuda* = boredom) and
  `KurakZapaleni/KurakPotahnuti/KurakLoop/KurakZahozeni` (*kuřák* = smoker: light up,
  drag, smoke, throw away).
* **Approach.** `pumper_start_refuel(car)` is reached from
  `player_do_action_button_in_car` — the action button while you sit in a car at a
  station. It asks `pumper_can_pump` first, then sets action 7 and walks:
  `pumper_ai_walk` steps toward the current node, `pumper_ai_turn` faces it (±π/4 starts
  a turn clip), `pumper_advance_to_next_node` picks the next.
* **Refuel.** The clip's notify events do the work (§2). `pumper_end_pump_fuel` calls
  `vehicle_set_fuel` and **`game_freeride_score_set(score − 500)`** — the tank is filled
  and you are charged **500** (§7).
* **Panic.** Hit him, or run into him with anything other than the car he is serving, and
  `pumper_panic` sets action `0x11`, kills the props and particles, and hands him to a
  driver object of type 9. `pumper_is_in_panic` is the query; `pumper_script_can_pump`
  reports it to scripts.

### 3a. He will not serve you if the cops are after you

`pumper_can_pump` refuses when the car's owner is under police attention. The predicate is
`police_manager_are_cops_after_player` (§7). It is true if any live officer is assigned to the player, **or** if a
pursuit record for him carries a committed police_state (`+0x3c`) or the wanted substate
(`+0x50`) with identity settled (`+0x51 == 0`). There is also a second gate,
`pursuit_event_is_car_chased`, on the car itself.

Distance gates: **6.0** units from his post for the car in general, **4.0** on the
police-check path.

## 4. Fields established

Only offsets a body actually reads are listed.

| PC offset | meaning | evidence |
|---|---|---|
| `+0x10` | entity kind = `0x19` | ctor; re-read by `pumper_script_can_pump` |
| `+0x24..+0x2c` | position | `pumper_snap_to_ground`, `pumper_update`, the savegame |
| `+0x30..+0x38` | facing | `pumper_ai_turn`, the savegame |
| `+0x68` | the scene frame | ctor, `pumper_update`, `pumper_init_sound` |
| `+0x70` | walk speed | `pumper_ai_walk`'s `dt * 0.001 * speed` |
| `+0x78` | the **pump** node id (designer) | ctor `= 0x29a`; `pumper_choose_next_action` |
| `+0x7c` | the **post** node id (designer) | ctor `= 0x29b`; `pumper_choose_next_action` |
| `+0x80` | the attendant's **script id** | ctor `= 0`; `pumper_script_can_pump`, `pumper_game_init`'s name formatting |
| `+0x84` | a designer dword | ctor `= 0x497cc`; saved and duplicated (§9) |
| `+0x88..+0x9c` | six **sound-name ids** | ctor writes `0x78..0x7d`, first overwritten with `0x1e`; read by `pumper_init_sound` |
| `+0xa0`/`+0xa4`/`+0xa8` | three particle handles, `-1` = none | ctor, `pumper_game_done`, `pumper_panic` |
| `+0xb4..+0xcc` | six `I3D_sound*` slots | `pumper_init_sound` / `_play_sound` / `_release_sounds`. **Slot 3 = the pump loop** |
| `+0xd0` | the **action id** | `0x11` = panic; `pumper_is_in_panic`, `pumper_is_pumping`, `pumper_ai` |
| `+0xd4` / `+0xd8` | applied / pending animation id | `pumper_set_anim` |
| `+0xdc` / `+0xdd` | two per-clip flags | ctor, `pumper_set_anim` |
| `+0xe0` | the embedded `C_animations_machine_de_luxe` | `structs/anim_machine.md` §1 |
| `+0x20c` / `+0x210` | current / previous walk node | `pumper_advance_to_next_node` |
| `+0x220` / `+0x224` | two dynamic collision objects | `pumper_release_colls`; excluded from his own ground ray |
| `+0x228` / `+0x22c` | ground-resnap timer / period | `pumper_update`, `rand_range() + 100` ms |
| `+0x234` | a watched object driving the panic watchdog | `pumper_update`, `pumper_game_done` |
| `+0x238`..`+0x244` | four idle props (cigarette, hose, …) | `pumper_panic`, `pumper_start_refuel`, the notify callback |
| `+0x24c` | the car being served | `pumper_end_pump_fuel`, `pumper_on_collision` |
| `+0x258` | the pumping flag | `pumper_start_pump_fuel` / `_end_pump_fuel` |
| `+0x25c` / `+0x260` | a dwell timer / period used by AI actions 1, 0xc, 0xd | `pumper_ai` |

**Savegame:** record type **5**, payload `actor_save_game_get_size() + 0x60`.

**Action ids:** `0` idle/walk · `1` at the pump · `7` approaching a car · `10` at the post
· `0x11` **panic**. `pumper_is_pumping` is true for `{0, 2..10}` — note it INCLUDES the
idle action, so it means "working", not "transferring fuel"; the transfer flag is `+0x258`.

## 5. The scene props it needs

`pumper_game_init` formats every frame name from the attendant's script id (`+0x80`) and
looks it up in the scene. The vocabulary, from the string literals in that function:

`_pumpa_pumpa_i` · `_pumpa_hadice_i` (*hadice* = hose) · `_pumpa_cigaro_i` /
`_pumpa_cigaro_box_i` · `_pumpa_xcigaro_i` / `_pumpa_xcigaro_box_i` · `_pumpa_camera_i` ·
`_pumpa_i`, plus `base` and `notify` child frames and the two FMV cigarette models
`FMVcigara.i3d` / `FMVcigareta2.I3D`.

Corroboration from a third artifact: the **last two entries of the engine-wide
animation registry** `g_anim_info_table` are `pumpar.i3d` (id 339) and
`pumpar hadice.i3d` (id 340) — the attendant and the attendant-with-hose clips.

## 7. PC vs PS2

**The field delta is not uniform, and no full map is claimed** (the same holds for `C_detector`,
`structs/detector.md` §7). Two witness pairs are certain:

* the **action id** — PC `+0xd0` ↔ PS2 `+0xc0`, from three independent readers
  (`IsInPanic` compares it to `0x11` on both builds, `IsPumping` switches the same value
  set, `PumperPanic` writes `0x11` to it);
* the **SaveData / DuplicateFrom band** — PC `+0x78`/`+0x7c`/`+0x84`/`+0x80`/`+0x88`(0x18)
  ↔ PS2 `+0xc4`/`+0xc8`/`+0xd0`/`+0xcc`/`+0xd4`(0x18), matched in the same order by BOTH
  methods on BOTH builds, including the deliberately skipped dword.

Those two imply different deltas, so a full map cannot be derived by rebasing either band (§9).

Other cross-build notes:

* `__sinit_C_pumper.cpp` is in the PS2 source-file manifest.
* `IdePoliciePoPlayeroviBoPumper__10C_shvestky` (*"are the police after the player, for the
  pumper"*) is the PS2 name of `police_manager_are_cops_after_player`.
* `pumper_notify_callback`'s PS2 symbol is
  `pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` — `__F`, no class tag — and
  it sits at the same rank (between `CanPump` and `StartPumpFuel`) on both builds.
* The refuel charge of **500** in `pumper_end_pump_fuel` is identical on PS2.
* PS2 `Do_AB_OwnerCar__8C_player` calls `Start__8C_pumperFP5C_car`; its PC twin
  `pumper_start_refuel` is correspondingly a callee of `player_do_action_button_in_car`.
* `Init` and `Death` are real 44 B / 8 B PS2 methods whose PC realisations are **shared**
  stubs (`actor_subclass_init_require_frame`, `stub_noop_cdecl_00574a40`), so they are roles,
  not twins.
* The MSVC scalar deleting destructor has no PS2 symbol by construction.
* `pumper_start_refuel` is the one method whose bodies diverge materially: PS2 spends part
  of it opening a dabing (voice) channel and speaking a line; the PC arm does not, which
  is why its ratio is 1.36 rather than the class's usual 0.7–0.9.
* Node stride in the scene node array: PC `0x1e`, PS2 `0x20`.
* PC `C_game+0x2a1c` ↔ PS2 `C_game+0xe00` — the `C_don_ocas` pedestrian-crowd proxy that
  `pumper_on_collision` / `pumper_hit` forward to — an independent witness for
  `structs/pedestrian.md`'s offset.

## 8. Cross-references

- `structs/anim_machine.md` §1 — the machine embedded at `+0xe0`.
- `structs/entity.md` §1 — the kind map; `0x19` is this class.
- `structs/pedestrian.md` — the `C_don_ocas` proxy this class forwards impacts to.
- `structs/police_manager.md` — the pursuit record `police_manager_are_cops_after_player` reads.

## 9. Open items

| Item | State |
|---|---|
| the full PC↔PS2 field map | not derivable by a uniform shift: the two certain witness pairs (§7) imply different deltas |
| `+0x84` (ctor `= 0x497cc`) | saved, loaded and duplicated by three methods; no reader located on either build |
| the four idle props at `+0x238`..`+0x244` | the notify callback shows `0x3c` = `+0x23c` and `0x3d` = `+0x238`; the other two are only ever released |
| the watched object at `+0x234` | drives the panic watchdog in `pumper_update`; its class is not identified |
| sound slots 0..5 | only slot **3** (the pump loop) is pinned; the other five are addressed by index from the notify callback, and their names (ids at `+0x88`) are not resolved |
| in game: the charge | does the 500 charge apply exactly as the code says, and does refuelling still work while a *different* car of yours is under pursuit? `pumper_can_pump` tests the owner, not the car you sit in |
| in game: panic | shoot the attendant and watch for the type-9 driver object — does he actually run, and where to? |
