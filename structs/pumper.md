# `C_pumper` — the gas-station attendant

**Created** 2026-07-13 11:46 +0200 · **Modified** 2026-10-11 15:07 +0200

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

¹ MSVC-only. ² A shared stub: the slot's role, not a twin (§6).

The TU also contains `i3d_cache_items_erase_at` (`0x004d3330`), a resource-cache helper
(`structs/i3d_cache.md` §4b) that MSVC emitted into this object: all 80 of its callsites erase
an item from one of the three cache item vectors.

## 2. `pumper_notify_callback` — the free function in the TU

One PC start matches no `C_pumper` method: `pumper_notify_callback` is a **free** function (§6).
It sits between `pumper_can_pump` and `pumper_start_pump_fuel`.

It is the seam between the animation and the gameplay. `pumper_game_init` registers it on
the attendant's `"notify"` child frame; the animation machine fires message ids out of the
clip and the callback turns them into actions:

| message | effect |
|---|---|
| `0x2a` | `pumper_start_pump_fuel` — **the fuel transfer begins here** |
| `0x2b` | `pumper_end_pump_fuel` — and ends here |
| `1`, `2` | sound slot 0, the footstep |
| `0x2e` / `0x2f` / `0x30` / `0x31` | sound slots 1 / 5 / 2 / 4: hose up, hose down, open, close |
| `0x3c` / `0x3d` | show the cigarette pack (`+0x23c`) / the cigarette (`+0x238`) in the right hand |
| `0x3e` | smoke particles (types `0x18`, `0x27`) from the two cigarettes; restart the timer `+0xac` / `+0xb0` with 1000 ms |
| `0x3f` / `0x40` | hide the pack / hide the cigarette and destroy its particle `+0xa0` |
| `0x41` | exhale: particle type `0x19` at the `neck` frame; restart the timer with 6500 ms |
| `0x42` / `0x43` | move the pack from the right hand to the left hand (`+0x23c` to `+0x244`) / back |
| `0x44` / `0x45` | move the cigarette from the right hand to the mouth (`+0x238` to `+0x240`) / back |

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
* **Speech.** `pumper_choose_next_action` takes a free speech channel (`+0x230`) three
  times: in action 6 the attendant speaks his own line `+0x84` on his own model, and in
  actions 8 and 9 the constant lines 14940003 and 9990501 are spoken on the player's model
  (`sounds\14940003.wav`, `sounds\09990501.wav`).
* **Refuel.** The clip's notify events do the work (§2). `pumper_end_pump_fuel` calls
  `vehicle_set_fuel` and **`game_freeride_score_set(score − 500)`** — the tank is filled
  and you are charged **500** (§6).
* **Panic.** Hit him, or run into him with anything other than the car he is serving, and
  `pumper_panic` sets action `0x11`, kills the props and particles, and hands him to a
  driver object of type 9. `pumper_is_in_panic` is the query; `pumper_script_can_pump`
  reports it to scripts.

### 3a. He will not serve you if the cops are after you

`pumper_can_pump` refuses when the car's owner is under police attention. The predicate is
`police_manager_are_cops_after_player` (§6). It is true if any live officer is assigned to the player, **or** if a
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
| `+0x84` | the attendant's **speech line id** (designer) | ctor `= 0x497cc` (301004); `pumper_load_data`; spoken in action 6 (§3) |
| `+0x88..+0x9c` | six **sound ids** into `tables\sounds.dat` (§4a) | ctor writes `0x78..0x7d`, first overwritten with `0x1e`; `pumper_load_data`; read by `pumper_init_sound` |
| `+0xa0`/`+0xa4`/`+0xa8` | three particle handles, `-1` = none | ctor, `pumper_game_done`, `pumper_panic` |
| `+0xac` / `+0xb0` | a timer / period | the notify callback restarts it (1000 / 6500 ms); `pumper_update` steps it |
| `+0xb4..+0xcc` | six `I3D_sound*` slots | `pumper_init_sound` / `_play_sound` / `_release_sounds`. **Slot 3 = the pump loop** |
| `+0xd0` | the **action id** | `0x11` = panic; `pumper_is_in_panic`, `pumper_is_pumping`, `pumper_ai` |
| `+0xd4` / `+0xd8` | applied / pending animation id | `pumper_set_anim` |
| `+0xdc` / `+0xdd` | two per-clip flags | ctor, `pumper_set_anim` |
| `+0xe0` | the embedded `C_animations_machine_de_luxe` | `structs/anim_machine.md` §1 |
| `+0x20c` / `+0x210` | current / previous walk node | `pumper_advance_to_next_node` |
| `+0x218` | the station frame `_pumpa<id>` | `pumper_game_init`; `pumper_can_pump`, `pumper_start_refuel` |
| `+0x220` / `+0x224` | two dynamic collision objects | `pumper_release_colls`; excluded from his own ground ray |
| `+0x228` / `+0x22c` | ground-resnap timer / period | `pumper_update`, `rand_range() + 100` ms |
| `+0x230` | the speech channel | `pumper_choose_next_action` |
| `+0x234` | the **hose**, the scene model frame `_pumpa_hadice<id>` (frame type 9 required) | `pumper_game_init`; `pumper_update` switches it off when the pump frame goes off |
| `+0x238` | the cigarette in the right hand (`_pumpa_cigaro<id>`) | `pumper_game_init`, the notify callback, `pumper_panic` |
| `+0x23c` | the cigarette pack in the right hand (`_pumpa_cigaro_box<id>`) | same |
| `+0x240` | the cigarette in the mouth (`_pumpa_xcigaro<id>`, on `neck`) | same |
| `+0x244` | the cigarette pack in the left hand (`_pumpa_xcigaro_box<id>`) | same |
| `+0x248` | the **pump** frame `_pumpa_pumpa<id>` | `pumper_game_init`; `pumper_update` sends the attendant into panic when its frame flag bit 0 (on) is clear |
| `+0x24c` | the car being served | `pumper_end_pump_fuel`, `pumper_on_collision` |
| `+0x258` | the pumping flag | `pumper_start_pump_fuel` / `_end_pump_fuel` |
| `+0x25c` / `+0x260` | a dwell timer / period used by AI actions 1, 0xc, 0xd | `pumper_ai` |

**Savegame:** record type **5**, payload `actor_save_game_get_size() + 0x60`.

### 4a. The six sound slots

`pumper_init_sound` hands the id of each slot to `sound_table_construct_sound`. The ids
resolve in the shipped `tables\sounds.dat` (`structs/sound_table.md` §4). Every shipped
attendant record in the scene files stores exactly the constructor defaults below.

| slot | id | `sounds.dat` record (group) | file | played by |
|---|---|---|---|---|
| 0 | 30 | `Asfalt` (Steps) | `s011.wav` .. `s015.wav` | notify messages 1, 2 |
| 1 | 121 | `pumpa_hadice up` (Actors) | `pumpa1.wav` | notify message `0x2e` |
| 2 | 122 | `pumpa_open` (Actors) | `pumpa2.wav` | notify message `0x30` |
| 3 | 123 | `pumpa_tankovani` (Actors) | `pumpa3.wav` | the pump loop |
| 4 | 124 | `pumpa_close` (Actors) | `pumpa4.wav` | notify message `0x31` |
| 5 | 125 | `pumpa_hadice down` (Actors) | `pumpa5.wav` | notify message `0x2f` |

Slot 0 is a footstep on asphalt; slots 1 to 5 are the hose and pump sounds.

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

No shipped scene carries the four cigarette frames, so `pumper_game_init` always builds
them from the FMV models: `FMVcigareta2.I3D` for the two cigarettes (linked to `r_hand`
and `neck`) and `FMVcigara.i3d` for the two packs (linked to `r_hand` and `l_hand`). The
hose and the pump frames are scene frames.

## 6. PC vs PS2

**The field map.** The delta between the builds is not uniform, so the map is not a rebase:
it pairs the two constructors statement by statement (both initialise the members in
declaration order) and checks each pair against a body that uses the field on both builds.
The action id is the one field no constructor writes; `IsInPanic`, `IsPumping` and
`PumperPanic` read or write it on both builds.

| PC | PS2 | field | witness |
|---|---|---|---|
| `+0x10` | `+0x50` | entity kind `0x19` | ctor |
| `+0x68` | `+0x70` | the scene frame | `Update` |
| `+0x70` / `+0x74` | `+0x108` / `+0x10c` | walk speed 2.0f / 0.5f | ctor |
| `+0x78` / `+0x7c` | `+0xc4` / `+0xc8` | pump node / post node | ctor, `SaveData`, `DuplicateFrom` |
| `+0x80` | `+0xcc` | script id | ctor, `SaveData`, `DuplicateFrom` |
| `+0x84` | `+0xd0` | speech line id | ctor, `SaveData`, `ChooseNextAction` |
| `+0x88..+0x9c` | `+0xd4..+0xe8` | the six sound ids | ctor, `SaveData` |
| `+0xa0` / `+0xa4` / `+0xa8` | `+0xec` / `+0xf0` / `+0xf4` | particle handles | ctor |
| `+0xac` / `+0xb0` | `+0x130` / `+0x134` | timer / period | the notify callback (1000 / 6500 ms) |
| `+0xb4..+0xc8` | `+0x118..+0x12c` | the six sound slots | ctor |
| `+0xcc` | `+0x110` | - | ctor |
| `+0xd0` | `+0xc0` | action id | `IsInPanic`, `IsPumping`, `PumperPanic` |
| `+0xd4` / `+0xd8` | `+0xf8` / `+0xfc` | applied / pending animation id | ctor |
| `+0xdc` / `+0xdd` | `+0x290` / `+0x291` | per-clip flags | ctor |
| `+0xe0` | `+0x150` | the embedded animation machine | ctor |
| `+0x208` | `+0x29c` | - | ctor |
| `+0x20c` / `+0x210` / `+0x214` | `+0x2c0` / `+0x2c4` / `+0x2c8` | walk nodes | ctor |
| `+0x218` | `+0x294` | station frame | ctor |
| `+0x220` / `+0x224` | `+0x2cc` / `+0x2d0` | dynamic collision objects | `Update` |
| `+0x228` / `+0x22c` | `+0x138` / `+0x13c` | ground-resnap timer / period | `Update` |
| `+0x230` | `+0x104` | speech channel | `ChooseNextAction` |
| `+0x234` | `+0x2a0` | hose frame | `Update` |
| `+0x238` / `+0x23c` / `+0x240` / `+0x244` | `+0x2a4` / `+0x2a8` / `+0x2ac` / `+0x2b0` | the four cigarette props | the notify callback |
| `+0x248` | `+0x2b4` | pump frame | `Update` |
| `+0x24c` | `+0x2bc` | the car being served | `EndPumpFuel` |
| `+0x250` | `+0x100` | - | ctor |
| `+0x254` | `+0x114` | - | ctor |
| `+0x258` | `+0x2d4` | pumping flag | `StartPumpFuel`, `EndPumpFuel` |
| `+0x25c` / `+0x260` | `+0x140` / `+0x144` | dwell timer / period | ctor (both reset it twice) |
| `+0x264` | `+0x2d5` | - | ctor |
| `+0x2a8` | `+0x2b8` | - | ctor |

`-` = no role established. The three timers sit together before the animation machine on
the PS2 and around it on the PC; each is paired by its own witness.

Other cross-build notes:

* `__sinit_C_pumper.cpp` is in the PS2 source-file manifest.
* `IdePoliciePoPlayeroviBoPumper__10C_shvestky` (*"are the police after the player, for the
  pumper"*) is the PS2 name of `police_manager_are_cops_after_player`.
* `pumper_notify_callback`'s PS2 symbol is
  `pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` — `__F`, no class tag — and
  it sits at the same rank (between `CanPump` and `StartPumpFuel`) on both builds.
* The refuel charge of **500** in `pumper_end_pump_fuel` is identical on PS2.
* The speech line default `0x497cc` and the three speech calls of `ChooseNextAction` are
  identical on PS2; the PS2 notify callback restarts the timer for messages `0x3e` and
  `0x41` without spawning the smoke particles the PC spawns.
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

## 7. Cross-references

- `structs/anim_machine.md` §1 — the machine embedded at `+0xe0`.
- `structs/entity.md` §1 — the kind map; `0x19` is this class.
- `structs/pedestrian.md` — the `C_don_ocas` proxy this class forwards impacts to.
- `structs/police_manager.md` — the pursuit record `police_manager_are_cops_after_player` reads.
- `structs/sound_table.md` — the `sounds.dat` bank the six sound ids index.
- `structs/speech.md` — the speech channels.

## 8. Open items

| Item | State |
|---|---|
| in game: the charge | does the 500 charge apply exactly as the code says, and does refuelling still work while a *different* car of yours is under pursuit? `pumper_can_pump` tests the owner, not the car you sit in |
| in game: panic | shoot the attendant and watch for the type-9 driver object — does he actually run, and where to? |
