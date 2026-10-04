# `G_CarDatabase` — the car catalogue

**Created** 2026-08-23 16:22 +0200 · **Modified** 2026-10-01 08:56 +0200

PC prefix **`car_database_`**. A file-scope singleton constructed by `car_database_static_init`
and living in the same translation unit as `G_LoadSaveClass` — see `structs/loadsave.md` for the
unit itself.

## 1. What it is

The read-only catalogue behind every car in the game: which models exist, what each is
called, which colours it has, what it is worth to steal, and which of race, championship
and Free Ride it is unlocked for. It owns three `.def` tables loaded whole out of the DTA
archive and never written back.

| table | file | stride | count field | buffer |
|---|---|---|---|---|
| car index | `tables\carindex.def` | **0xa4** | `+0x00` | `+0x04` |
| player index | `tables\playerindex.def` | **0x24** | `+0x08` | `+0x0c` |
| carcyclopedia details | `tables\carcyclopedia.def` | **200 (0xc8)** | `+0xb8` | `+0xbc` |

The first two are loaded by `car_database_init` at construction and released by
`car_database_close`. The third is **reference-counted** (`+0xb4`) and loaded on demand by
`car_database_load_details` / `car_database_unload_details` — a menu screen that shows car
statistics pulls it in and drops it again.

Each record count is derived as `file_size / stride`, so the strides above are byte-proven
by the loader rather than inferred from a reader.

## 2. The packed car id

Almost every entry point takes or returns a **packed id**, not a plain index:

```
bits  0..15   car index into the carindex table
bits 16..23   always 0 on the PC (below)
bits 24..31   colour index
```

| helper | direction |
|---|---|
| `car_database_pack_id(idx, _, colour)` | `(idx & 0xffff) \| (colour << 24)` |
| `car_database_pack_id_by_model(name, _, colour)` | name → index → the same pack |
| `car_database_pack_id_from_car(car)` | a live `C_car` → id + colour; it calls `car_get_model_level`, but `xor al,al; shl eax,0x10` clears the level before the shift, so bits 16..23 stay 0 |
| `car_database_get_index_by_model(name)` | name → index, `__strcmpi` scan from entry 1 |
| `car_database_get_index_by_model_filename(name, &colour)` | a full **filename** → index + colour |
| `car_database_clamp_packed_id(&id)` | repairs an id in place: clears an out-of-range car, clamps the colour |

**Bits 16..23 carry nothing.** Every producer writes 0 there, and the one reader,
`enabled_cars_have_car`, uses the byte to pick a bank only when its third argument is set;
both of its callers (`car_database_collect_cars_by_class_mask` and
`car_database_collect_cars_by_manufacturer`, a closed set of two `E8` calls) push 0. The
variant-looking string choice is made by the colour byte (§3).

`car_database_get_index_by_model_filename` is where the model filename convention is
readable: it strips any path, lifts the colour digit from `name[len-6]`, then truncates
the trailing `NNxx` suffix before looking the base name up. Colour digits above 4 are
forced to 0. The model file itself is built back with `"%s%02d.i3d"` by
`car_database_get_car_model`, which adds **+10** to the colour index when the caller asks
for the second (wreck) set.

**Scans start at entry 1, never 0** — in every by-name lookup in the class. Entry 0 is not
a car.

## 3. Record fields

### Car index record (`carindex.def`, stride 0xa4)

The record is four 0x20-byte strings and nine dwords; every byte is accounted for. The shipped
file holds 83 records: entry 0 is the placeholder `-`, entries 1..81 are cars and entry 82 is
all zero.

| offset | type | meaning | reader |
|---|---|---|---|
| `+0x00` | char[] | model base name | every `__strcmpi` scan |
| `+0x20` | char[] | **wreck** model name | `car_database_get_wreck_model` |
| `+0x40` | char[] | **shadow** model name — the join key of the fake-shadow subsystem ¹ | `car_database_get_shadow_model` |
| `+0x60` | char[] | display name | `car_database_get_car_name` |
| `+0x80` | dword | manufacturer bit | `car_database_collect_cars_by_manufacturer` (`1 << n`) |
| `+0x84` | dword | **race** unlock mask | `car_database_is_enabled_for_race`, `loadsave_race_cars_unlock_from_car` |
| `+0x88` | dword | **class / championship** mask ² | `car_database_is_enabled_for_championship`, `car_database_get_car_class_mask`, `car_database_collect_cars_by_class_mask` |
| `+0x8c` | dword | **steal-group mask** — bit n is letter `A` + n ³ | `garage_can_steal_car_model` (`garage+0x98 & +0x8c`) |
| `+0x90` | dword | **steal time, milliseconds** | `car_database_get_steal_time` (default **15000**) |
| `+0x94` | dword | 0 in all 83 records | none |
| `+0x98` | dword | civilian-usage mask | `car_database_get_civil_mask_by_model` |
| `+0x9c` | dword | **seat count** — 2 on 55 cars, 4 on 21, 1 on the five single-seaters (the four racers and `HotRod`) ⁴ | none |
| `+0xa0` | dword | **Free Ride** unlock mask | `car_database_is_enabled_for_freeride` |

¹ `game_add_frame_for_shadow` stores it on the car's `S_shadow_item`, and
`game_bind_fake_shadow_buffer` matches it case-insensitively against the 43 meshes `game_init`
preloaded, so a name that is not one of the 43 silently gets no shadow (`structs/fake_shadow.md`).
² A double duty: `car_database_get_car_class_mask` and `car_database_is_enabled_for_championship`
read the same dword.
³ Script command `garage_enablesteal` (`garage_enable_steal_for_letter`) ORs letter `A`..`Z`
into `garage+0x98`; the shipped scripts grant every letter at least once. In the shipped table
60 cars carry the same mask here as in the race mask `+0x84`, the five racing cars carry only
bit 31, which no letter can grant, and the 16 bonus cars (entries 66..81) carry 0, so neither
group can ever pass the steal test. No car carries the `Z` bit.
⁴ In game one person enters a car per door, so a two-door car seats two (user). The values
follow the body style: 2 on the two-door Tudors, coupes, roadsters, pickups and trucks, 4 on
the four-door cars.

### Detail record (`carcyclopedia.def`, stride 200)

`+0xa4` = colour count (hence `car_database_get_max_color_index` returning it minus one),
`+0xac` = the carcyclopedia mask, `+0xb0`..`+0xc0` = five **colour-name** string ids, one per
colour index, returned by `car_database_get_color_name_id`. Every non-zero id in the shipped file
is a colour name, textdb 3800..3813 ("Blue", "Dark blue", "Brown", "Green", "Red", ...).

### Player index record (`playerindex.def`, stride 0x24)

`+0x00` = model name (what `car_database_get_player_model` returns), `+0x20` = civilian mask.

**This table is the roster of playable character models.** The retail file is 2376 B =
**66 records**, and every name is a `civilNN.i3d` human model. Two consumers use it:
`car_database_collect_player_models_by_civil_mask`, and the multiplayer **driver-appearance
picker** (`structs/multiplayer.md` §3), whose result the session stores at `g_net_session+0x228`.
`car_database_get_player_model` returns a PLAYER-CHARACTER model name, not a car model.

Its miss fallback at `0x0064d82c` (`s_minus_0064d82c`) is `"-"`, a placeholder dash, not a default
model name. That literal has three readers: `car_database_get_player_model`,
`gui_widget_item_num_draw` (which draws it as the decrement label beside `s_plus_0064dc9c`), and
case `0x0c` of the key-name table in `input_binding_get_label` (`0x005d3180`), i.e. the `-` key.

## 4. The by-name family, and the `"FMV"` prefix

`car_database_get_wreck_model` and `car_database_get_shadow_model` are byte-identical
except for the record offset they return (`+0x20` vs `+0x40`) — a near-sibling pair. Both first
strip a leading **`"FMV"`** from the model name; that string is `s_fmv_model_prefix_0064d850`
(neither a pointer nor a label).

## 5. Permission queries

Three identical predicates differing only in the mask field they test — 46 bytes each on
the PC:

| function | field |
|---|---|
| `car_database_is_enabled_for_race` | `+0x84` |
| `car_database_is_enabled_for_championship` | `+0x88` |
| `car_database_is_enabled_for_freeride` | `+0xa0` |

and three collectors that fill a caller's vector:

| function | walks | tests | filter |
|---|---|---|---|
| `car_database_collect_cars_by_class_mask` | car index | `+0x88 & mask` | optional `G_EnabledCars` |
| `car_database_collect_cars_by_manufacturer` | car index | `+0x80 & (1 << n)` | optional `G_EnabledCars` |
| `car_database_collect_cars_by_carcyclopedia_mask` | **details** | `+0xac & mask` | - |

The optional filter is `enabled_cars_have_car` — **its own class**, `G_EnabledCars`, not a
`car_database_` method. It is a four-bank bitmap at `+0x00/+0x10/+0x20/+0x30`; byte 2 of
the packed id selects the bank and the low 16 bits select the bit.

## 6. Coverage

**37 function starts, all named.** 30 carry a confirmed PS2 twin.

| PC name | VA | PS2 twin |
|---|---|---|
| `car_database_static_init` | `0x00556f50` | - ¹ |
| `car_database_ctor` | `0x0055c310` | `__ct__13G_CarDatabaseFv` |
| `car_database_init` | `0x0055c370` | `Init__13G_CarDatabaseFv` |
| `car_database_close` | `0x0055c440` | `Close__13G_CarDatabaseFv` |
| `car_database_get_car` | `0x0055c480` | `GetCar__13G_CarDatabaseFUi` |
| `car_database_get_car_details` | `0x0055c4b0` | `GetCarDetails__13G_CarDatabaseFUi` |
| `car_database_get_player_record` | `0x0055c4e0` | - ² |
| `car_database_get_player_model` | `0x0055c510` | `GetPlayerModel__13G_CarDatabaseFUi` |
| `car_database_get_car_name` | `0x0055c540` | `GetCarName__13G_CarDatabaseFUi` |
| `car_database_get_color_name_id` | `0x0055c570` | `GetCarSecondName__13G_CarDatabaseFUiUi` |
| `car_database_clamp_packed_id` | `0x0055c620` | `RepairIndex__13G_CarDatabaseFRUi` |
| `car_database_get_max_color_index` | `0x0055c670` | `GetCarMaxColor__13G_CarDatabaseFUi` |
| `car_database_get_car_model` | `0x0055c6b0` | `GetCarModel__13G_CarDatabaseFUiPUcb` |
| `car_database_get_car_params` | `0x0055c710` | `GetCarParams__13G_CarDatabaseFUiPUiPUiPUi` |
| `car_database_pack_id` | `0x0055c7a0` | `GetIDByParams__13G_CarDatabaseFUiUiUi` |
| `car_database_pack_id_by_model` | `0x0055c7c0` | - ² |
| `car_database_get_index_by_model` | `0x0055c7e0` | `GetIdxByModel__13G_CarDatabaseFPUc` |
| `car_database_get_index_by_model_filename` | `0x0055c830` | `GetIdxByModelLong__13G_CarDatabaseFPUcRUi` |
| `car_database_pack_id_from_car` | `0x0055c8f0` | `GetIDByActor__13G_CarDatabaseFP5C_car` |
| `car_database_get_wreck_model` | `0x0055c950` | `GetWreckageByModel__13G_CarDatabaseFPUc` |
| `car_database_get_shadow_model` | `0x0055ca10` | `GetShadowByModel__13G_CarDatabaseFPUc` |
| `car_database_get_civil_mask_by_model` | `0x0055cad0` | `GetCivilMaskByModel__13G_CarDatabaseFPUc` |
| `car_database_collect_cars_by_class_mask` | `0x0055cb30` | `GetCarsByClass__13G_CarDatabaseF…` |
| `car_database_collect_cars_by_manufacturer` | `0x0055cbe0` | `GetCarsByManufacturer__13G_CarDatabaseF…` |
| `car_database_collect_cars_by_carcyclopedia_mask` | `0x0055cca0` | `GetCarsByCarcyclopediaMask__13G_CarDatabaseF…` |
| `car_database_is_enabled_for_race` | `0x0055cd40` | `IsEnabledForRace__13G_CarDatabaseFUiUi` |
| `car_database_is_enabled_for_championship` | `0x0055cd70` | `IsEnabledForChampionship__13G_CarDatabaseFUiUi` |
| `car_database_is_enabled_for_freeride` | `0x0055cda0` | `IsEnabledForFreeride__13G_CarDatabaseFUiUi` |
| `car_database_get_car_class_mask` | `0x0055cdd0` | `GetCarClass__13G_CarDatabaseFUi` |
| `car_database_get_steal_time` | `0x0055ce00` | `GetStealTime__13G_CarDatabaseFPUc` |
| `car_database_get_player_civil_mask_by_model` | `0x0055ce60` | `GetPlayerCivilMaskByModel__13G_CarDatabaseFPUc` |
| `car_database_collect_player_models_by_civil_mask` | `0x0055ceb0` | - ² |
| `car_database_load_details` | `0x0055cf30` | `LoadDetails__13G_CarDatabaseFv` |
| `car_database_unload_details` | `0x0055cfc0` | `UnloadDetails__13G_CarDatabaseFv` |

¹ CRT. The three other CRT thunks — `car_database_static_ctor_call` (`0x00556f60`),
`car_database_static_atexit_register` (`0x00556f70`), `car_database_static_dtor_call`
(`0x00556f80`) — complete the 37. ² PC-only (§7).

## 7. PC vs PS2

* **Strides differ:** car record **0xa4 PC / 0xa8 PS2**, detail record **200 PC / 204
  PS2**. The player record is 0x24 on both. The car-record offsets of §3 are the same on both
  builds — the PS2 accessors read `+0x84` (race), `+0x88` (class), `+0x90` (steal time), `+0x98`
  (civil mask) and `+0xa0` (Free Ride) — and the extra PS2 dword is `+0xa4`, which the
  PS2-only `CreateBestTimes__8C_racingFv` reads as a racing-segment selector.
* **Three PS2 methods have no PC function start:** `GetCarsByRaceMask` (which would test `+0x84`
  in a loop), `EnableCarsForFree` (it ORs together the race masks of every record whose Free Ride
  mask matches) and `GetWreckageByID`.
* **The level byte is dropped on both builds.** PS2 `GetIDByActor` ORs `level << 16` into the id
  and then masks it with `& 0xff00ffff`, the same outcome as the PC's cleared shift (§2).
* **Three PC functions are surplus:** `car_database_get_player_record` (the NULL-returning
  variant of `GetPlayerModel`, which returns the `"-"` default string instead),
  `car_database_pack_id_by_model` and `car_database_collect_player_models_by_civil_mask`.
* **Emission order differs slightly:** the PC emits *Class* before *Manufacturer*, the PS2
  emits *Class, RaceMask, Manufacturer*. Both collectors are therefore mapped on the
  **field offset**, not the rank.
* **The mapping anchor** is the identical triple `IsEnabledForRace/Championship/Freeride` —
  46/46/46 bytes on the PC against 80/80/80 on the PS2, three consecutive functions differing only
  by one offset. Working outward from it forces the whole tail.
* **`GetStealTime` matches on a magic constant:** the fallback **15000** is identical on both
  builds. It is *not* `garage_can_steal_car_model`, which is `CanSteal__13G_GarageClass` and tests
  a permission mask on the loadsave singleton.
* On the PS2 the player-model accessor is also called from
  `GameInit__10C_opponentFP17I3D_animation_setUi`, which builds a race opponent's driver model
  from `C_opponent+0x17c`.

## 8. Open items

| item | state |
|---|---|
| steal time in game | hot-wire two different cars and time them — `+0x90` says the durations are per-model and the fallback is 15 s |
