# `G_CarDatabase` — the car catalogue

**Created** 2026-08-23 16:22 +0200 · **Modified** 2026-11-04 16:46 +0100

The read-only catalogue behind every car in the game: which models exist, what each is called,
which colours it has, what it is worth to steal, and which of race, championship and Free Ride it
is unlocked for. It owns three `.def` tables loaded whole out of the DTA archive and never written
back. A file-scope singleton at `0x00671ef0` (registered as `g_car_database_car_count`, its `+0x00`),
constructed by `car_database_static_init`.

| | |
|---|---|
| PC prefix | `car_database_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0xc0`: the singleton at `0x00671ef0` is followed by the `G_LoadSaveClass` singleton at `0x00671fb0`, and the last field ends at `+0xc0` |
| PC constructor | `car_database_ctor` @ `0x0055c310` |
| PC translation unit | `[0x00556ee0, 0x00561b40)`, the `G_LoadSaveClass` unit, hosted by `structs/loadsave.md`; this class is the static quartet at `0x00556f50`..`0x00556f80` and `[0x0055c310, 0x0055d000)` |
| PS2 class | `G_CarDatabase` (tag `13G_CarDatabase`), 34 methods: 30 CONFIRMED twins, the destructor inlined, 3 with no PC function (§6) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct G_CarDatabase {               // 0xc0 bytes; singleton at 0x00671ef0
    /* +0x00 */ uint32_t        car_count;      // [H] carindex.def size / 0xa4
    /* +0x04 */ S_car_index    *cars;           // [H] the carindex.def buffer (§1a)
    /* +0x08 */ uint32_t        player_count;   // [H] playerindex.def size / 0x24
    /* +0x0c */ S_player_index *players;        // [H] the playerindex.def buffer (§1c)
    /* +0x10 */ S_car_index     fallback_car;   // [H] returned while the car table is not loaded
    /* +0xb4 */ int32_t         details_refs;   // [H] the detail table's load counter (§5c)
    /* +0xb8 */ uint32_t        details_count;  // [H] carcyclopedia.def size / 200
    /* +0xbc */ S_car_details  *details;        // [H] the carcyclopedia.def buffer (§1b)
} G_CarDatabase;
```

| Table | File | Stride | Count | Buffer |
|---|---|---|---|---|
| car index | `tables\carindex.def` | **`0xa4`** | `+0x00` | `+0x04` |
| player index | `tables\playerindex.def` | **`0x24`** | `+0x08` | `+0x0c` |
| carcyclopedia details | `tables\carcyclopedia.def` | **200 (`0xc8`)** | `+0xb8` | `+0xbc` |

Each count is `file_size / stride`, so the strides are proven by the loader rather than inferred
from a reader. The first two tables are loaded by `car_database_init` and released by
`car_database_close`; the third is loaded on demand by `car_database_load_details` and dropped by
`car_database_unload_details`, so a menu screen that shows car statistics pulls it in and drops it
again (§5c).

**`fallback_car` is one whole car record** (`0x10 + 0xa4 = 0xb4`). `car_database_ctor` copies
`"none"` into its model name (`+0x10`), the Czech error literal `s_car_index_error_name_0064d7e0`
(roughly "bad car/player index.def") into its display name (`+0x70`) and zeroes its class mask
(`+0x98`). `car_database_get_car` returns it only while the car buffer is NULL; an index past the
table returns entry 0 instead.

### 1a. `S_car_index` — one car, `0xa4` bytes (`carindex.def`)

```c
typedef struct S_car_index {                 // 0xa4 bytes
    /* +0x00 */ char     model[0x20];          // [H] model base name; the key of every by-name scan
    /* +0x20 */ char     wreck_model[0x20];    // [H] the wreck model
    /* +0x40 */ char     shadow_model[0x20];   // [H] the fake-shadow join key
    /* +0x60 */ char     display_name[0x20];   // [H] the name shown in menus
    /* +0x80 */ uint32_t manufacturer;         // [H] the manufacturer number n, tested as 1 << n
    /* +0x84 */ uint32_t race_mask;            // [H] race unlock mask
    /* +0x88 */ uint32_t class_mask;           // [H] class / championship mask
    /* +0x8c */ uint32_t steal_mask;           // [H] steal groups; bit n = letter 'A' + n
    /* +0x90 */ uint32_t steal_time_ms;        // [H] hot-wire time; the fallback is 15000
    /* +0x94 */ uint32_t unread_94;            // [U] 0 in all 83 shipped records; no reader (.text scan)
    /* +0x98 */ uint32_t civil_mask;           // [H] civilian-usage mask
    /* +0x9c */ uint32_t seat_count;           // [H] 1, 2 or 4 in the shipped table; no reader
    /* +0xa0 */ uint32_t freeride_mask;        // [H] Free Ride unlock mask
} S_car_index;
```

**Why four 32-byte strings.** Each is a 32-byte model or display name; every byte of the record
is accounted for by the four strings and nine dwords. The shipped file holds 83 records: entry 0 is
the placeholder `-`, entries 1..81 are cars and entry 82 is all zero. **Every by-name scan starts at
entry 1**, so entry 0 is never found by name.

**Seat count.** 2 on 55 cars, 4 on 21, 1 on the five single-seaters (the four racers and `HotRod`).
In game one person enters a car per door, so a two-door car seats two (user). The values follow the
body style: 2 on the two-door Tudors, coupes, roadsters, pickups and trucks, 4 on the four-door cars.

### 1b. `S_car_details` — one carcyclopedia record, 200 bytes (`carcyclopedia.def`)

```c
typedef struct S_car_details {               // 0xc8 bytes; index-parallel to the car table
    /* +0x00 */ S_car_index car_record;       // [H] a copy of the car record of the same index (§1b)
    /* +0xa4 */ uint32_t color_count;          // [H] number of paint colours
    /* +0xa8 */ uint32_t unread_a8;            // [U] 3 in every car record, 0 in entries 0 and 82; no reader (.text scan)
    /* +0xac */ uint32_t carcyclopedia_mask;   // [H] the Carcyclopedia collector's filter
    /* +0xb0 */ uint32_t color_name_id[5];     // [H] textdb id of each colour's name
    /* +0xc4 */ uint32_t description_id;       // [H] textdb id of the description text
} S_car_details;
```

**Why five colour names, and what each entry is.** Entry `k` is the textdb id of colour index `k`'s
name, returned by `car_database_get_color_name_id`. Five is the colour range of the packed id: a
colour digit above 4 in a model file name is forced to 0 (§4), and `car_database_get_max_color_index`
returns `color_count - 1`. Every non-zero id in the shipped file is a colour name, textdb
3800..3813 ("Blue", "Dark blue", "Brown", "Green", "Red", ...). The record is indexed by the car
index: `car_database_get_max_color_index` bounds the index against `car_count` before reading
`details`.

**The first `0xa4` bytes are the car record again.** In the shipped files the first `0xa4` bytes of
every `carcyclopedia.def` record equal the `carindex.def` record of the same index, 83 of 83, so
`car_record` is an `S_car_index` (§1a). No `car_database_` function reads it; its one reader
outside the class is `racing_generate_qualification`, which takes the lowest set bit of its
`class_mask` (`+0x88`), clamped to 7, as the class index for `circuit_get_class_qualify_time_min`
and `circuit_get_class_qualify_time_max`. `unread_a8` is 3 in every car record and 0 in entries 0
and 82, and nothing reads it.

### 1c. `S_player_index` — one character model, `0x24` bytes (`playerindex.def`)

```c
typedef struct S_player_index {              // 0x24 bytes
    /* +0x00 */ char     model[0x20];          // [H] a civilNN.i3d human model name
    /* +0x20 */ uint32_t civil_mask;           // [H] civilian-usage mask
} S_player_index;
```

**This table is the roster of playable character models.** The retail file is 2376 B = **66
records**, and every name is a `civilNN.i3d` human model. Two consumers use it:
`car_database_collect_player_models_by_civil_mask`, and the multiplayer **driver-appearance picker**
(`structs/multiplayer.md` §6), whose result the session stores at `g_net_session+0x228`.
`car_database_get_player_model` returns a PLAYER-CHARACTER model name, not a car model. Its miss
fallback at `0x0064d82c` (`s_minus_0064d82c`) is `"-"`, a placeholder dash, not a default model
name. That literal has three readers: `car_database_get_player_model`, `gui_widget_item_num_draw`
(which draws it as the decrement label beside `s_plus_0064dc9c`), and case `0x0c` of the key-name
table in `input_binding_get_label` (`0x005d3180`), the `-` key.

## 2. Field evidence

Measured by a census of every access in the `car_database_*` functions (the `this + N` text and the
disassembly, which shows the `+0x00` count and the record reads), and a displacement census of the
record offsets in the named consumers.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | car_count | H | `car_database_ctor`, `car_database_init`, `car_database_close` | every index-taking `car_database_*` function, the by-name scans and the two car collectors; absolute loads of `0x00671ef0` in `gui_mp_lobby_ctor`, `gui_mp_lobby_on_child_result`, `gui_mp_lobby_tick`, `gui_free_ride_ctor`, `gui_free_ride_on_close_child` | - |
| +0x04 | cars | H | `car_database_ctor`, `car_database_init`, `car_database_close` | `car_database_init`, `car_database_close` (free it), every car-record reader | - |
| +0x08 | player_count | H | `car_database_ctor`, `car_database_init`, `car_database_close` | `car_database_get_player_record`, `car_database_get_player_model`, `car_database_get_player_civil_mask_by_model`, `car_database_collect_player_models_by_civil_mask`; absolute loads of `0x00671ef8` in `gui_mp_person_select_ctor`, `gui_mp_person_select_tick`, `gui_mp_lobby_ctor`, `gui_mp_lobby_tick` | - |
| +0x0c | players | H | `car_database_ctor`, `car_database_init`, `car_database_close` | the same four, `car_database_init`, `car_database_close` | - |
| +0x10 | fallback_car | H | `car_database_ctor` | `car_database_get_car` | §1 |
| +0xb4 | details_refs | H | `car_database_ctor`, `car_database_load_details`, `car_database_unload_details` | `car_database_load_details`, `car_database_unload_details` | §5c |
| +0xb8 | details_count | H | `car_database_ctor`, `car_database_load_details`, `car_database_unload_details` | `car_database_get_car_details`, `car_database_collect_cars_by_carcyclopedia_mask` | - |
| +0xbc | details | H | `car_database_ctor`, `car_database_load_details`, `car_database_unload_details` | `car_database_get_car_details`, `car_database_get_color_name_id`, `car_database_clamp_packed_id`, `car_database_get_max_color_index`, `car_database_collect_cars_by_carcyclopedia_mask`, `car_database_unload_details` | - |

### 2a. `S_car_index`

Every field is written by `car_database_init`, which reads the file whole.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | model | H | `car_database_init` | every by-name scan (`__strcmpi`) | - |
| +0x20 | wreck_model | H | `car_database_init` | `car_database_get_wreck_model` | - |
| +0x40 | shadow_model | H | `car_database_init` | `car_database_get_shadow_model` | ¹ |
| +0x60 | display_name | H | `car_database_init` | `car_database_get_car_name` | - |
| +0x80 | manufacturer | H | `car_database_init` | `car_database_collect_cars_by_manufacturer` | tested as `1 << n` |
| +0x84 | race_mask | H | `car_database_init` | `car_database_is_enabled_for_race`, `loadsave_race_cars_unlock_from_car`, `garage_add_car_by_index`, `garage_add_car_by_actor`, `garage_game_done` | the three `garage_` functions read it through `car_database_get_car` and pass it to `loadsave_race_cars_enable` |
| +0x88 | class_mask | H | `car_database_init` | `car_database_is_enabled_for_championship`, `car_database_get_car_class_mask`, `car_database_collect_cars_by_class_mask` | ² |
| +0x8c | steal_mask | H | `car_database_init` | `garage_can_steal_car_model` (`garage+0x98 & +0x8c`) | ³ |
| +0x90 | steal_time_ms | H | `car_database_init` | `car_database_get_steal_time` | the fallback is 15000 |
| +0x94 | unread_94 | U | `car_database_init` | - | 0 in all 83 records; no reader in the class, at the 13 callsites of the record getters or in the copy at `S_car_details +0x00` (.text scan) |
| +0x98 | civil_mask | H | `car_database_init` | `car_database_get_civil_mask_by_model` | - |
| +0x9c | seat_count | H | `car_database_init` | - | §1a |
| +0xa0 | freeride_mask | H | `car_database_init` | `car_database_is_enabled_for_freeride` | - |

¹ `game_add_frame_for_shadow` stores it on the car's `S_shadow_item`, and
`game_bind_fake_shadow_buffer` matches it case-insensitively against the 43 meshes `game_init`
preloaded, so a name that is not one of the 43 silently gets no shadow (`structs/fake_shadow.md`).
² A double duty: `car_database_get_car_class_mask` and `car_database_is_enabled_for_championship`
read the same dword.
³ Script command `garage_enablesteal` (`garage_enable_steal_for_letter`) ORs letter `A`..`Z` into
`garage+0x98`; the shipped scripts grant every letter at least once. In the shipped table 60 cars
carry the same mask here as in the race mask `+0x84`, the five racing cars carry only bit 31, which
no letter can grant, and the 16 bonus cars (entries 66..81) carry 0, so neither group can ever pass
the steal test. No car carries the `Z` bit.

### 2b. `S_car_details`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | car_record | H | `car_database_load_details` | `racing_generate_qualification` (`class_mask`, `+0x88`) | equal to the `carindex.def` record of the same index in 83 of 83 shipped records (§1b) |
| +0xa4 | color_count | H | `car_database_load_details` | `car_database_clamp_packed_id`, `car_database_get_max_color_index` | - |
| +0xa8 | unread_a8 | U | `car_database_load_details` | - | 3 in every car record, 0 in entries 0 and 82; no reader (.text scan) |
| +0xac | carcyclopedia_mask | H | `car_database_load_details` | `car_database_collect_cars_by_carcyclopedia_mask` | - |
| +0xb0 | color_name_id | H | `car_database_load_details` | `car_database_get_color_name_id` | §1b |
| +0xc4 | description_id | H | `car_database_load_details` | `carcyclopedia_refresh_car`, `gui_race_car_model_tick` | `structs/carcyclopedia.md` §6 |

### 2c. `S_player_index`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | model | H | `car_database_init` | `car_database_get_player_model`, `car_database_get_player_civil_mask_by_model` (the by-name scan) | - |
| +0x20 | civil_mask | H | `car_database_init` | `car_database_get_player_civil_mask_by_model`, `car_database_collect_player_models_by_civil_mask` | - |

## 3. Method family

34 PS2 methods: 30 CONFIRMED twins, the destructor inlined into the static destructor, 3 with no
PC function (§6). Three PC functions have no PS2 twin.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00556f50 | `car_database_static_init` | the singleton `0x00671ef0` | - | - | - | the singleton's `.CRT$XCU` entry: calls the two below |
| 0x00556f60 | `car_database_static_ctor` | the singleton `0x00671ef0` | - | - | - | `mov ecx, 0x00671ef0; jmp car_database_ctor` |
| 0x00556f70 | `car_database_register_atexit_dtor` | the singleton `0x00671ef0` | - | - | - | `atexit(car_database_static_dtor)` |
| 0x00556f80 | `car_database_static_dtor` | the singleton `0x00671ef0` | - | - | - | `mov ecx, 0x00671ef0; jmp car_database_close` |
| - | - | the singleton `0x00671ef0` | `__dt__13G_CarDatabaseFv` | 0x002bfba0 | inlined | inlined into `car_database_static_dtor`: the PS2 destructor is `Close()` plus the delete, the PC destructor a jump to `car_database_close` |
| 0x0055c310 | `car_database_ctor` | instance | `__ct__13G_CarDatabaseFv` | 0x002ba2e0 | CONFIRMED | zeroes the counts and buffers, fills `fallback_car` (§1) |

### 3b. Accessors

The class has no accessor: every getter bounds its index first.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0055c370 | `car_database_init` | `Init__13G_CarDatabaseFv` | 0x002ba350 | CONFIRMED | loads `carindex.def` and `playerindex.def` whole |
| 0x0055c440 | `car_database_close` | `Close__13G_CarDatabaseFv` | 0x002ba4c0 | CONFIRMED | frees both buffers, zeroes the counts |
| 0x0055c480 | `car_database_get_car` | `GetCar__13G_CarDatabaseFUi` | 0x002ba520 | CONFIRMED | the record of a packed id; `fallback_car` while unloaded, entry 0 when out of range |
| 0x0055c4b0 | `car_database_get_car_details` | `GetCarDetails__13G_CarDatabaseFUi` | 0x002ba570 | CONFIRMED | the detail record, or 0 |
| 0x0055c4e0 | `car_database_get_player_record` | - | - | - | the player record, NULL when unloaded (§6) |
| 0x0055c510 | `car_database_get_player_model` | `GetPlayerModel__13G_CarDatabaseFUi` | 0x002ba5c0 | CONFIRMED | a character model name, `"-"` on a miss (§1c) |
| 0x0055c540 | `car_database_get_car_name` | `GetCarName__13G_CarDatabaseFUi` | 0x002ba610 | CONFIRMED | `display_name` |
| 0x0055c570 | `car_database_get_color_name_id` | `GetCarSecondName__13G_CarDatabaseFUiUi` | 0x002ba660 | CONFIRMED | the colour's textdb id |
| 0x0055c620 | `car_database_clamp_packed_id` | `RepairIndex__13G_CarDatabaseFRUi` | 0x002ba790 | CONFIRMED | repairs an id in place: clears an out-of-range car, clamps the colour |
| 0x0055c670 | `car_database_get_max_color_index` | `GetCarMaxColor__13G_CarDatabaseFUi` | 0x002ba810 | CONFIRMED | `color_count - 1` |
| 0x0055c6b0 | `car_database_get_car_model` | `GetCarModel__13G_CarDatabaseFUiPUcb` | 0x002ba880 | CONFIRMED | builds `"%s%02d.i3d"`, +10 on the colour for the wreck set (§4) |
| 0x0055c710 | `car_database_get_car_params` | `GetCarParams__13G_CarDatabaseFUiPUiPUiPUi` | 0x002ba940 | CONFIRMED | unpacks a packed id |
| 0x0055c7a0 | `car_database_pack_id` | `GetIDByParams__13G_CarDatabaseFUiUiUi` | 0x002ba9f0 | CONFIRMED | `(idx & 0xffff) \| (colour << 24)` (§4) |
| 0x0055c7c0 | `car_database_pack_id_by_model` | - | - | - | name → index → the same pack (§6) |
| 0x0055c7e0 | `car_database_get_index_by_model` | `GetIdxByModel__13G_CarDatabaseFPUc` | 0x002baa20 | CONFIRMED | `__strcmpi` scan from entry 1 |
| 0x0055c830 | `car_database_get_index_by_model_filename` | `GetIdxByModelLong__13G_CarDatabaseFPUcRUi` | 0x002baab0 | CONFIRMED | a full model filename → index + colour (§4) |
| 0x0055c8f0 | `car_database_pack_id_from_car` | `GetIDByActor__13G_CarDatabaseFP5C_car` | 0x002babd0 | CONFIRMED | a live `C_car` → id + colour (§4) |
| 0x0055c950 | `car_database_get_wreck_model` | `GetWreckageByModel__13G_CarDatabaseFPUc` | 0x002bac70 | CONFIRMED | §5a |
| 0x0055ca10 | `car_database_get_shadow_model` | `GetShadowByModel__13G_CarDatabaseFPUc` | 0x002bade0 | CONFIRMED | §5a |
| 0x0055cad0 | `car_database_get_civil_mask_by_model` | `GetCivilMaskByModel__13G_CarDatabaseFPUc` | 0x002baf00 | CONFIRMED | `civil_mask` by name |
| 0x0055cb30 | `car_database_collect_cars_by_class_mask` | `GetCarsByClass__13G_CarDatabaseF…` | 0x002bafb0 | CONFIRMED | §5b |
| 0x0055cbe0 | `car_database_collect_cars_by_manufacturer` | `GetCarsByManufacturer__13G_CarDatabaseF…` | 0x002bb160 | CONFIRMED | §5b |
| 0x0055cca0 | `car_database_collect_cars_by_carcyclopedia_mask` | `GetCarsByCarcyclopediaMask__13G_CarDatabaseF…` | 0x002bb250 | CONFIRMED | §5b |
| 0x0055cd40 | `car_database_is_enabled_for_race` | `IsEnabledForRace__13G_CarDatabaseFUiUi` | 0x002bb4a0 | CONFIRMED | §5b |
| 0x0055cd70 | `car_database_is_enabled_for_championship` | `IsEnabledForChampionship__13G_CarDatabaseFUiUi` | 0x002bb4f0 | CONFIRMED | §5b |
| 0x0055cda0 | `car_database_is_enabled_for_freeride` | `IsEnabledForFreeride__13G_CarDatabaseFUiUi` | 0x002bb540 | CONFIRMED | §5b |
| 0x0055cdd0 | `car_database_get_car_class_mask` | `GetCarClass__13G_CarDatabaseFUi` | 0x002bb590 | CONFIRMED | `class_mask` |
| 0x0055ce00 | `car_database_get_steal_time` | `GetStealTime__13G_CarDatabaseFPUc` | 0x002bb5e0 | CONFIRMED | `steal_time_ms` by name, 15000 on a miss |
| 0x0055ce60 | `car_database_get_player_civil_mask_by_model` | `GetPlayerCivilMaskByModel__13G_CarDatabaseFPUc` | 0x002bb690 | CONFIRMED | a character model's `civil_mask` |
| 0x0055ceb0 | `car_database_collect_player_models_by_civil_mask` | - | - | - | fills a vector with the character models of a civil mask (§6) |
| 0x0055cf30 | `car_database_load_details` | `LoadDetails__13G_CarDatabaseFv` | 0x002bb730 | CONFIRMED | §5c |
| 0x0055cfc0 | `car_database_unload_details` | `UnloadDetails__13G_CarDatabaseFv` | 0x002bb810 | CONFIRMED | §5c |

### 3d. Free helpers

None of this class's functions calls a free helper of the shared unit; those helpers belong to
`structs/loadsave.md`.

## 4. The packed car id

Almost every entry point takes or returns a **packed id**, not a plain index:

```
bits  0..15   car index into the carindex table
bits 16..23   always 0 on the PC (below)
bits 24..31   colour index
```

| Helper | Direction |
|---|---|
| `car_database_pack_id(idx, _, colour)` | `(idx & 0xffff) \| (colour << 24)` |
| `car_database_pack_id_by_model(name, _, colour)` | name → index → the same pack |
| `car_database_pack_id_from_car(car)` | a live `C_car` → id + colour; it calls `car_get_model_level`, but `xor al,al; shl eax,0x10` clears the level before the shift, so bits 16..23 stay 0 |
| `car_database_get_index_by_model(name)` | name → index, `__strcmpi` scan from entry 1 |
| `car_database_get_index_by_model_filename(name, &colour)` | a full **filename** → index + colour |
| `car_database_clamp_packed_id(&id)` | repairs an id in place: clears an out-of-range car, clamps the colour |

**Bits 16..23 carry nothing.** Every producer writes 0 there, and the one reader,
`enabled_cars_have_car`, uses the byte to pick a bank only when its third argument is set; both of
its callers (`car_database_collect_cars_by_class_mask` and
`car_database_collect_cars_by_manufacturer`, a closed set of two `E8` calls) push 0. The
variant-looking string choice is made by the colour byte (§1b).

`car_database_get_index_by_model_filename` is where the model filename convention is readable: it
strips any path, lifts the colour digit from `name[len-6]`, then truncates the trailing `NNxx`
suffix before looking the base name up. Colour digits above 4 are forced to 0. The model file itself
is built back with `"%s%02d.i3d"` by `car_database_get_car_model`, which adds **+10** to the colour
index when the caller asks for the second (wreck) set.

## 5. Lookups

### 5a. The by-name family and the `"FMV"` prefix

`car_database_get_wreck_model` and `car_database_get_shadow_model` are byte-identical except for
the record offset they return (`+0x20` vs `+0x40`), a near-sibling pair. Both first strip a leading
**`"FMV"`** from the model name; that string is `s_fmv_model_prefix_0064d850` (neither a pointer nor
a label).

### 5b. Permission queries and collectors

Three identical predicates differing only in the mask field they test, 46 bytes each:

| Function | Field |
|---|---|
| `car_database_is_enabled_for_race` | `+0x84` |
| `car_database_is_enabled_for_championship` | `+0x88` |
| `car_database_is_enabled_for_freeride` | `+0xa0` |

and three collectors that fill a caller's vector:

| Function | Walks | Tests | Filter |
|---|---|---|---|
| `car_database_collect_cars_by_class_mask` | car index | `+0x88 & mask` | optional `G_EnabledCars` |
| `car_database_collect_cars_by_manufacturer` | car index | `+0x80 & (1 << n)` | optional `G_EnabledCars` |
| `car_database_collect_cars_by_carcyclopedia_mask` | **details** | `+0xac & mask` | - |

The optional filter is `enabled_cars_have_car`, a method of its own class `G_EnabledCars`, not of
this one. It is a four-bank bitmap at `+0x00` / `+0x10` / `+0x20` / `+0x30`; byte 2 of the packed id
selects the bank and the low 16 bits select the bit.

### 5c. The detail table's load counter

`car_database_load_details` increments `details_refs` and loads `carcyclopedia.def` only when the
new value is below 2, setting it to 1 after the read. `car_database_unload_details` decrements it,
frees the table when the result is below 1, and then **sets `details_refs` to 0 in every case**. So
the counter is not a reference count: with two loads outstanding, the first unload leaves 0 and
keeps the table, and the second unload frees it.

## 6. PC vs PS2

* **Strides differ:** car record **`0xa4` PC / `0xa8` PS2**, detail record **200 PC / 204 PS2**. The
  player record is `0x24` on both. The car-record offsets of §1a are the same on both builds (the
  PS2 accessors read `+0x84` race, `+0x88` class, `+0x90` steal time, `+0x98` civil mask and `+0xa0`
  Free Ride), and the extra PS2 dword is `+0xa4`, which the PS2-only `CreateBestTimes__8C_racingFv`
  reads as a racing-segment selector.
* **The level byte is dropped on both builds.** PS2 `GetIDByActor` ORs `level << 16` into the id and
  then masks it with `& 0xff00ffff`, the same outcome as the PC's cleared shift (§4).
* **Three PC functions have no PS2 twin:** `car_database_get_player_record` (the NULL-returning
  variant of `GetPlayerModel`, which returns the `"-"` default string instead),
  `car_database_pack_id_by_model` and `car_database_collect_player_models_by_civil_mask`.
* **Emission order differs slightly:** the PC emits *Class* before *Manufacturer*, the PS2 emits
  *Class, RaceMask, Manufacturer*. Both collectors are therefore mapped on the **field offset**, not
  the rank.
* **The mapping anchor** is the identical triple `IsEnabledForRace` / `Championship` / `Freeride`:
  46/46/46 bytes on the PC against 80/80/80 on the PS2, three consecutive functions differing only
  by one offset. Working outward from it forces the whole tail.
* **`GetStealTime` matches on a magic constant:** the fallback **15000** is identical on both
  builds. It is *not* `garage_can_steal_car_model`, which is `CanSteal__13G_GarageClass` and tests a
  permission mask on the loadsave singleton.
* **The destructor.** PS2 `__dt__13G_CarDatabaseFv` (`0x002bfba0`, outside the class's band) calls
  `Close` and deletes; the PC static destructor jumps straight to `car_database_close` (§3a).
* On the PS2 the player-model accessor is also called from
  `GameInit__10C_opponentFP17I3D_animation_setUi`, which builds a race opponent's driver model from
  `C_opponent+0x17c`.

### 6a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002bad90 | `GetWreckageByID__13G_CarDatabaseFi` | no PC function start at its rank; the PC tail aligns without it |
| 0x002bb0a0 | `GetCarsByRaceMask__13G_CarDatabaseF…` | no PC function start at its rank (it would test `+0x84` in a loop) |
| 0x002bb440 | `EnableCarsForFree__13G_CarDatabaseFUi` | no PC function start at its rank (it ORs together the race masks of every record whose Free Ride mask matches) |

## 7. Related

- `structs/loadsave.md` — the `G_LoadSaveClass` translation unit this class lives in, and its host.
- `structs/carcyclopedia.md` — the Carcyclopedia screen, the detail table's consumer.
- `structs/multiplayer.md` §6 — the driver-appearance picker over the character table.
- `structs/fake_shadow.md` — the shadow-model join key.
- `structs/garage.md` — the steal test.

## 8. Open items

1. **Steal time in game:** hot-wire two different cars and time them; `+0x90` says the durations are
   per-model and the fallback is 15 s.
