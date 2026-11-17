# The racing front-end screens — `g_m_race.cpp`

**Created** 2026-08-24 11:05 +0200 · **Modified** 2026-11-09 15:07 +0100

The single-player racing menus: 14 `GM_Menu` subclasses in one translation unit, each with a shipped
`menu.def` design of its own (§4). Retail removes the door to them, and mission 6 still runs a race
(`basic/artifacts/racing.md`).

| | |
|---|---|
| PC prefix | `gui_race_root_`, `gui_race_championship_`, `gui_race_round_select_`, `gui_race_champ_launch_`, `gui_race_champ_results_`, `gui_race_setup_`, `gui_race_circuit_select_`, `gui_race_opponents_`, `gui_race_car_maker_`, `gui_race_car_model_`, `gui_race_car_color_`, `gui_race_launch_`, `gui_race_results_`, `gui_race_bonus_car_` |
| PC kind (`+0x10`) | - |
| PC vtable | a contiguous run from `g_gui_menu_vt_race_root` @ `0x0063d434`, stride `0x18`, 6 slots each (§4) |
| PC base class | `GM_Menu` (`structs/gui_screen.md`) |
| PC instance size | per screen, §1 |
| PC constructor | per screen, §3a |
| PC translation unit | `[0x00566250, 0x0056adc0)` guard to guard |
| PS2 class | 14 `GM_*` classes (§4) and `GM_DamageDifficulty`; 85 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Every screen inherits the `GM_Menu` layout, `0x48` bytes (`structs/gui_screen.md` §1). `GM_Racing`, `GM_Championship` and `GM_BonusCar` add no field: `operator_new(0x48)` at every caller. The eleven others follow, each size read at the `operator_new` before its constructor.

### 1a. `GM_SelectInRound` — `gui_race_round_select_`

```c
typedef struct GM_SelectInRound {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ void * rounds;                     // [H] the round vector the ctor receives: `C_championship+0xac`; begin at its `+0x04`, end at `+0x08`
} GM_SelectInRound;
```

### 1b. `GM_ChampionshipRace` — `gui_race_champ_launch_`

```c
typedef struct GM_ChampionshipRace {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t chosen_car;               // [H] the packed car id; seeded from `g_racing+0x64`, written by the garage chain through its out pointer
} GM_ChampionshipRace;
```

### 1c. `GM_ChampionshipResults` — `gui_race_champ_results_`

```c
typedef struct GM_ChampionshipResults {                  // 0xd4 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t elapsed_ms;               // [H] time counted towards `period_ms`; the 15 s countdown prints from it
    /* +0x4c */ uint32_t period_ms;                // [H] the auto-continue period: 200 from the ctor's first store, 0 after the build, 15000 in the tick
    /* +0x50 */ char countdown_text[0x84];         // [H] the formatted countdown string, shown in widget `0x78`
} GM_ChampionshipResults;
```

`countdown_text` has `0x84` bytes because the object ends at `0xd4`: the allocation is the only bound, and the `%s %u` format needs far less. `GM_SingleRaceResults` (§1k) has the same three fields.

### 1d. `GM_SingleRace` — `gui_race_setup_`

```c
typedef struct GM_SingleRace {                  // 0x8c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t chosen_car;               // [H] the packed car id; seeded from `g_racing+0x64`, written by the garage chain through its out pointer
    /* +0x4c */ char circuit_bitmap_name[0x40];    // [H] the `circuit%d.bmp` name widget `100` points at
} GM_SingleRace;
```

`circuit_bitmap_name` is the `0x40` bytes between `chosen_car` and the object's end; the longest string written is `circuit%d.bmp`.

### 1e. `GM_SelectCircuit` — `gui_race_circuit_select_`

```c
typedef struct GM_SelectCircuit {                  // 0x90 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ void * circuits;                   // [H] the circuit vector the ctor receives; begin at its `+0x04`, end at `+0x08`
    /* +0x4c */ char circuit_bitmap_name[0x40];    // [H] the `circuit%d.bmp` name widget `1000` points at
    /* +0x8c */ uint32_t focused_row;              // [H] the row last previewed; a change of the focused id redraws the bitmap
} GM_SelectCircuit;
```

`circuit_bitmap_name` is the `0x40` bytes before `focused_row`; the longest string written is `circuit%d.bmp`.

### 1f. `GM_SelectOpponents` — `gui_race_opponents_`

```c
typedef struct GM_SelectOpponents {                  // 0x98 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t opponent_cars[20];        // [H] one packed car id per opponent row; widget id = 100 + index
} GM_SelectOpponents;
```

`opponent_cars` holds 20 entries because the grid has 20 rows: the ctor clones 19 from the template (ids 101..119), the walk runs `i < 0x14`, and the tick passes `0x14` to `racing_add_opponents`. The ctor zeroes only the first ten entries (`for (i = 10; ...)`); see Open item 5.

### 1g. `GM_SelectCarManufacturer` — `gui_race_car_maker_`

```c
typedef struct GM_SelectCarManufacturer {                  // 0x54 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t * out_car;                // [H] where the accepted car id goes: the opening screen's field
    /* +0x4c */ uint32_t chosen_car;               // [H] the model screen's result, through the out pointer the tick hands down
    /* +0x50 */ uint32_t race_mask;                // [H] the car mask the opener passes; -1 selects `loadsave_race_cars_get_enabled`
} GM_SelectCarManufacturer;
```

### 1h. `GM_SelectCarModel` — `gui_race_car_model_`

```c
typedef struct GM_SelectCarModel {                  // 0x70 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t maker_index;              // [M] the manufacturer's index, row id minus 100; stored and never read, the ctor uses its parameter
    /* +0x4c */ uint32_t * out_car;                // [H] where the accepted car id goes: the maker screen's `chosen_car`
    /* +0x50 */ uint32_t chosen_car;               // [H] the colour screen's result, through the out pointer the tick hands down
    /* +0x54 */ char cars_allocator;               // [M] the vector's empty allocator byte; its address reaches `car_database_collect_cars_by_manufacturer`
    /* +0x58 */ uint32_t * cars_begin;             // [H] the listed cars, packed ids; a row indexes it by id minus 100
    /* +0x5c */ uint32_t * cars_end;               // [H] one past the last listed car
    /* +0x60 */ uint32_t * cars_capacity_end;      // [M] one past the allocation; only zeroed here
    /* +0x64 */ uint32_t focused_row;              // [H] the row last described; -1 from the ctor, a change redraws name and details
    /* +0x68 */ uint32_t unused_68;                // [U] no access; no reader (class census)
    /* +0x6c */ uint32_t race_mask;                // [M] the mask the maker screen passes; stored and never read
} GM_SelectCarModel;
```

`+0x54` to `+0x60` is one `std::vector` with its allocator byte.

### 1i. `GM_SelectCarModification` — `gui_race_car_color_`

```c
typedef struct GM_SelectCarModification {                  // 0x58 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t car;                      // [H] the packed car id being modified; a picked row repacks the colour index into it
    /* +0x4c */ uint32_t * out_car;                // [H] where the accepted car id goes: the model screen's `chosen_car`
    /* +0x50 */ uint32_t unused_50[2];             // [U] no access; no reader (class census)
} GM_SelectCarModification;
```

The caller allocates `0x58`, so `unused_50` runs to the object's end.

### 1j. `GM_SingleRaceSecond` — `gui_race_launch_`

```c
typedef struct GM_SingleRaceSecond {                  // 0x4c bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t chosen_car;               // [H] the packed car id; seeded from `g_racing+0x64`, written by the garage chain through its out pointer
} GM_SingleRaceSecond;
```

### 1k. `GM_SingleRaceResults` — `gui_race_results_`

```c
typedef struct GM_SingleRaceResults {                  // 0xd4 bytes; the base GM_Menu is 0x48 bytes
    /* +0x48 */ uint32_t elapsed_ms;               // [H] time counted towards `period_ms`; the 15 s countdown prints from it
    /* +0x4c */ uint32_t period_ms;                // [H] the auto-continue period: 200 from the ctor's first store, 0 after the build, 15000 in the tick
    /* +0x50 */ char countdown_text[0x84];         // [H] the formatted countdown string, shown in widget `0x78`
} GM_SingleRaceResults;
```

## 2. Field evidence

Cells come from a census of each screen's `this` accesses in `.text` plus a read of every function named; a write through an out pointer is listed at the field it lands in.

### 2a. `GM_SelectInRound`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | rounds | H | `gui_race_round_select_ctor` | `gui_race_round_select_ctor`, `gui_race_round_select_tick` | - |

### 2b. `GM_ChampionshipRace`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | chosen_car | H | `gui_race_champ_launch_ctor`, `gui_race_car_maker_on_close_child` | `gui_race_champ_launch_ctor`, `gui_race_champ_launch_on_close_child` | - |

### 2c. `GM_ChampionshipResults`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | elapsed_ms | H | `gui_race_champ_results_ctor`, `gui_race_champ_results_tick` | `gui_race_champ_results_tick` | - |
| +0x4c | period_ms | H | `gui_race_champ_results_ctor`, `gui_race_champ_results_tick` | `gui_race_champ_results_tick` | - |
| +0x50 | countdown_text | H | `gui_race_champ_results_tick` | `gui_menu_set_component_text_ptr` | - |

### 2d. `GM_SingleRace`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | chosen_car | H | `gui_race_setup_ctor`, `gui_race_car_maker_on_close_child` | `gui_race_setup_ctor`, `gui_race_setup_on_close_child` | - |
| +0x4c | circuit_bitmap_name | H | `gui_race_setup_ctor`, `gui_race_setup_on_close_child` | `gui_menu_set_component_text_ptr` | - |

### 2e. `GM_SelectCircuit`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | circuits | H | `gui_race_circuit_select_ctor` | `gui_race_circuit_select_ctor`, `gui_race_circuit_select_tick` | - |
| +0x4c | circuit_bitmap_name | H | `gui_race_circuit_select_ctor`, `gui_race_circuit_select_tick` | `gui_menu_set_component_text_ptr` | - |
| +0x8c | focused_row | H | `gui_race_circuit_select_tick` | `gui_race_circuit_select_tick` | - |

### 2f. `GM_SelectOpponents`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | opponent_cars | H | `gui_race_opponents_ctor`, `gui_race_car_maker_on_close_child` | `gui_race_opponents_ctor`, `gui_race_opponents_on_close_child`, `racing_add_opponents` | - |

### 2g. `GM_SelectCarManufacturer`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | out_car | H | `gui_race_car_maker_ctor` | `gui_race_car_maker_on_close_child` | - |
| +0x4c | chosen_car | H | `gui_race_car_model_on_close_child`, `gui_race_car_model_tick` | `gui_race_car_maker_on_close_child` | - |
| +0x50 | race_mask | H | `gui_race_car_maker_ctor` | `gui_race_car_maker_ctor`, `gui_race_car_maker_tick` | - |

### 2h. `GM_SelectCarModel`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | maker_index | M | `gui_race_car_model_ctor` | - | - |
| +0x4c | out_car | H | `gui_race_car_model_ctor` | `gui_race_car_model_on_close_child`, `gui_race_car_model_tick` | - |
| +0x50 | chosen_car | H | `gui_race_car_color_tick` | `gui_race_car_model_on_close_child` | - |
| +0x54 | cars_allocator | M | `gui_race_car_model_ctor` | - | - |
| +0x58 | cars_begin | H | `car_database_collect_cars_by_manufacturer`, `gui_race_car_model_dtor` | `gui_race_car_model_ctor`, `gui_race_car_model_dtor`, `gui_race_car_model_tick` | - |
| +0x5c | cars_end | H | `car_database_collect_cars_by_manufacturer`, `gui_race_car_model_dtor` | `gui_race_car_model_ctor`, `gui_race_car_model_tick` | - |
| +0x60 | cars_capacity_end | M | `car_database_collect_cars_by_manufacturer`, `gui_race_car_model_dtor` | - | - |
| +0x64 | focused_row | H | `gui_race_car_model_ctor`, `gui_race_car_model_tick` | `gui_race_car_model_tick` | - |
| +0x68 | unused_68 | U | - | - | no access; no reader (class census) |
| +0x6c | race_mask | M | `gui_race_car_model_ctor` | - | - |

### 2i. `GM_SelectCarModification`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | car | H | `gui_race_car_color_ctor`, `gui_race_car_color_tick` | `gui_race_car_color_ctor`, `gui_race_car_color_tick` | - |
| +0x4c | out_car | H | `gui_race_car_color_ctor` | `gui_race_car_color_tick` | - |
| +0x50 | unused_50 | U | - | - | no access; no reader (class census) |

### 2j. `GM_SingleRaceSecond`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | chosen_car | H | `gui_race_launch_ctor`, `gui_race_car_maker_on_close_child` | `gui_race_launch_ctor`, `gui_race_launch_on_close_child` | - |

### 2k. `GM_SingleRaceResults`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | elapsed_ms | H | `gui_race_results_ctor`, `gui_race_results_tick` | `gui_race_results_tick` | - |
| +0x4c | period_ms | H | `gui_race_results_ctor`, `gui_race_results_tick` | `gui_race_results_tick` | - |
| +0x50 | countdown_text | H | `gui_race_results_tick` | `gui_menu_set_component_text_ptr` | - |

## 3. Method family

37 of 85 PS2 methods of these classes have a row in §3, the other 48 are in §10a; `gui_race_car_model_scalar_deleting_dtor` and `gui_race_launch_on_close_child` have no twin.

### 3a. Construction & destruction

#### The unit

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00566250 | `crt_static_init_guard_00566250` | the translation unit | - | - | - | `.CRT$XCU` guard of this unit's static destructor |
| 0x00566270 | `crt_atexit_register_00566270` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x00578790 | `gui_menu_scalar_deleting_dtor_common` | shared | - | - | - | slot 0 of every screen here: calls `gui_menu_dtor`, frees on request; the address lies in the unit of `structs/game_menus.md` |

#### `GM_Racing` (`gui_race_root_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00566280 | `gui_race_root_ctor` | instance | `OnBeforeCreate__9GM_RacingFv` | 0x002df4a0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d434`, builds design `0x16` from `g_gui_screen_design_table`, clears flag bit `0x10000` and sets `0x80000` on `this+0x40`, then disables widgets |

#### `GM_Championship` (`gui_race_championship_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005666a0 | `gui_race_championship_ctor` | instance | `OnCreate__15GM_ChampionshipFv` | 0x002dcaf0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d44c`, builds design `0x17` ("Championship" / Class A / a `rari` round row / EXIT), then clones one `txts` + `rari` pair per championship in |

#### `GM_SelectInRound` (`gui_race_round_select_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005669f0 | `gui_race_round_select_ctor` | instance | `OnCreate__16GM_SelectInRoundFv` | 0x002dcfb0 | CONFIRMED | design `0x18`: one row per round of the vector |

#### `GM_ChampionshipRace` (`gui_race_champ_launch_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00566c80 | `gui_race_champ_launch_ctor` | instance | `OnCreate__19GM_ChampionshipRaceFv` | 0x002dd330 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d47c`, builds design `0x19` (PRACTICE / QUALIFY / RACE / REPLAY / GARAGE / damage toggle / BACK plus a best-times and a competitors table), then fills |

#### `GM_ChampionshipResults` (`gui_race_champ_results_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00567450 | `gui_race_champ_results_ctor` | instance | `OnCreate__22GM_ChampionshipResultsFv` | 0x002dded0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d494`, builds design `0x1a` ("Racing:" + CONTINUE / SHOW REPLAY / SAVE REPLAY / RETRY RACE, plus a second gate with CONTINUE / ABORT), then fills the |

#### `GM_SingleRace` (`gui_race_setup_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00567fb0 | `gui_race_setup_ctor` | instance | `OnCreate__13GM_SingleRaceFv` | 0x002d9d00 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d4ac`, builds design `0x1b` (Select Circuit / GARAGE / Select Opponents / Track Mode / Laps / RACE / REPLAY / EXIT), then seeds every widget from |

#### `GM_SelectCircuit` (`gui_race_circuit_select_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00568410 | `gui_race_circuit_select_ctor` | instance | `OnCreate__16GM_SelectCircuitFv` | 0x002da200 | CONFIRMED | design `0x1c`: one row per circuit; a circuit of state 2 is disabled |

#### `GM_SelectOpponents` (`gui_race_opponents_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005686f0 | `gui_race_opponents_ctor` | instance | `OnCreate__18GM_SelectOpponentsFv` | 0x002db110 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d4dc`, builds design `0x1d` ("Opponents:" / "Difficulty:" / "Opponent Cars:" / OK / Cancel), clones 19 further opponent rows, points the difficulty |

#### `GM_SelectCarManufacturer` (`gui_race_car_maker_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005689c0 | `gui_race_car_maker_ctor` | instance | `OnCreate__24GM_SelectCarManufacturerFv` | 0x002da4a0 | CONFIRMED | design `0x1e`: three erase filters per maker; a maker with no car left is disabled |

#### `GM_SelectCarModel` (`gui_race_car_model_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00568d90 | `gui_race_car_model_ctor` | instance | `OnCreate__17GM_SelectCarModelFv` | 0x002da850 | CONFIRMED | design `0x1f`: collects the maker's cars, one `item` row per car, the same three filters |
| 0x00569080 | `gui_race_car_model_scalar_deleting_dtor` | instance | `__dt__17GM_SelectCarModelFv` | 0x002df230 | STRUCTURAL | calls `gui_race_car_model_dtor`, then `operator delete`s the object when bit 0 of the flag argument is set |
| 0x005690a0 | `gui_race_car_model_dtor` | instance | `__dt__17GM_SelectCarModelFv` | 0x002df230 | CONFIRMED | frees the car-id vector this screen owns (`+0x58` begin, `+0x5c` end, `+0x60` cap, all three zeroed), then tail-calls the base destructor `gui_menu_dtor` |

#### `GM_SelectCarModification` (`gui_race_car_color_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005693d0 | `gui_race_car_color_ctor` | instance | `OnCreate__24GM_SelectCarModificationFv` | 0x002daf70 | STRUCTURAL | design `0x20`: one row per colour index |

#### `GM_SingleRaceSecond` (`gui_race_launch_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00569660 | `gui_race_launch_ctor` | instance | `OnCreate__19GM_SingleRaceSecondFv` | 0x002db3d0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d53c`, builds design `0x21` ("Single race" + PRACTICE / QUALIFY / RACE / REPLAY / damage toggle / BACK), then fills the best-times block and the |

#### `GM_SingleRaceResults` (`gui_race_results_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00569da0 | `gui_race_results_ctor` | instance | `OnCreate__20GM_SingleRaceResultsFv` | 0x002dbad0 | CONFIRMED | chains `gui_menu_ctor`, installs vtable `0x0063d554`, builds design `0x22` ("Racing:" / Best Lap Times / Best Race times / "You finished " / New Record), then fills the three parallel result columns |

#### `GM_BonusCar` (`gui_race_bonus_car_`)

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0056ab40 | `gui_race_bonus_car_ctor` | instance | `OnCreate__11GM_BonusCarFv` | 0x002deae0 | CONFIRMED | design `0x23`: writes the awarded car's name and variant into widget `10` |

### 3b. Accessors

No accessor: every field is reached from a constructor, a tick or a child handler.

### 3c. Behaviour

#### `GM_Racing` (`gui_race_root_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00566310 | `gui_race_root_tick` | `OnClick__9GM_RacingFUi` | 0x002d98e0 | CONFIRMED | slot 5: dispatches the root menu's rows |
| 0x00566490 | `gui_race_root_open_single_race` | `OpenMenuForSingleRace__9GM_RacingFb` | 0x002d9a30 | CONFIRMED | constructs the racing root menu (`operator_new(0x48)`), then the single-race setup screen (`0x8c`), then — when the flag is set — the launch screen (`0x4c`) |
| 0x00566560 | `gui_race_root_open_championship` | `OpenMenuForChampionship__9GM_RacingFi` | 0x002d9b10 | CONFIRMED | constructs the racing root menu (`operator_new(0x48)`), then the championship list (`0x48`), then — when asked — the round picker (`0x4c`) seeded with the index of the current championship inside |

#### `GM_Championship` (`gui_race_championship_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005668c0 | `gui_race_championship_tick` | `OnClick__15GM_ChampionshipFUi` | 0x002dce80 | CONFIRMED | slot 5: `0x1a7` EXIT; `0x1b8`..`0x1cb` set the current championship and open the round picker |

#### `GM_SelectInRound` (`gui_race_round_select_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00566b80 | `gui_race_round_select_tick` | `OnClick__16GM_SelectInRoundFUi` | 0x002dd230 | CONFIRMED | slot 5: `500` back; `100`..`0x95` set the current round and open the launch screen |

#### `GM_ChampionshipRace` (`gui_race_champ_launch_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00567170 | `gui_race_champ_launch_on_close_child` | `OnCloseChild__19GM_ChampionshipRaceFP7GM_MenuUiUi` | 0x002ddbe0 | CONFIRMED | when the garage child (`0x1c7`) closes with a result, adopts the chosen car into `g_racing+0x64`, refreshes widget `200` with its name and rewrites every opponent row `0x640+i`; then chains the |
| 0x00567280 | `gui_race_champ_launch_tick` | `OnClick__19GM_ChampionshipRaceFUi` | 0x002ddaa0 | CONFIRMED | slot 5: `0x1c3`..`0x1c6`, `0x1c7` garage, `0x1c8` damage toggle, `0x1c9` back |

#### `GM_ChampionshipResults` (`gui_race_champ_results_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00567b20 | `gui_race_champ_results_tick` | `OnClick__22GM_ChampionshipResultsFUi` | 0x002de930 | CONFIRMED | slot 5: standings refresh, the 15 s countdown, F2 overlay; closes with `0x33` |

#### `GM_SingleRace` (`gui_race_setup_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00568100 | `gui_race_setup_on_close_child` | `OnCloseChild__13GM_SingleRaceFP7GM_MenuUiUi` | 0x002da0b0 | CONFIRMED | child `0x1d7` (circuit picker) closed ⇒ re-format `circuit%d.bmp` into widget `100` and refresh the circuit name in widget `0xc` |
| 0x005681e0 | `gui_race_setup_tick` | `OnClick__13GM_SingleRaceFUi` | 0x002d9e60 | CONFIRMED | slot 5: opens the circuit picker, the garage and the launch screen; flips the track mode; steps the laps |

#### `GM_SelectCircuit` (`gui_race_circuit_select_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005685d0 | `gui_race_circuit_select_tick` | `OnClick__16GM_SelectCircuitFUi` | 0x002da390 | CONFIRMED | slot 5: previews the focused circuit; a row sets the current circuit and closes |

#### `GM_SelectOpponents` (`gui_race_opponents_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00568830 | `gui_race_opponents_on_close_child` | `OnCloseChild__18GM_SelectOpponentsFP7GM_MenuUiUi` | 0x002db220 | CONFIRMED | when a manufacturer-picker child closes on a widget in `100..0x95`, refreshes that opponent row's label with the chosen car's name, then chains the framework default |
| 0x00568880 | `gui_race_opponents_tick` | `OnClick__18GM_SelectOpponentsFUi` | 0x002db2d0 | CONFIRMED | slot 5: steps the difficulty; `500` adds the 20 opponents |

#### `GM_SelectCarManufacturer` (`gui_race_car_maker_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00568c70 | `gui_race_car_maker_on_close_child` | `OnCloseChild__24GM_SelectCarManufacturerFP7GM_MenuUiUi` | 0x002da800 | CONFIRMED | when a model-picker child closes on a widget in `100..0x95`, writes the car id it produced through this screen's own out-pointer and closes this screen with result 1, then chains the framework default |
| 0x00568cc0 | `gui_race_car_maker_tick` | `OnClick__24GM_SelectCarManufacturerFUi` | 0x002da730 | CONFIRMED | slot 5: `500` back; `100`..`0x95` open the model picker |

#### `GM_SelectCarModel` (`gui_race_car_model_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005690d0 | `gui_race_car_model_close` | `OnClose__17GM_SelectCarModelFv` | 0x002dab40 | CONFIRMED | calls `car_database_unload_details()` to drop the detail records the constructor loaded, then chains the framework's `gui_menu_close` |
| 0x00569100 | `gui_race_car_model_on_close_child` | `OnCloseChild__17GM_SelectCarModelFP7GM_MenuUiUi` | 0x002dab70 | CONFIRMED | when a colour-picker child closes on a widget in `100..0x95`, forwards the chosen car id through this screen's own out-pointer and closes this screen too with result 1, then chains the framework |
| 0x00569150 | `gui_race_car_model_tick` | `OnClick__17GM_SelectCarModelFUi` | 0x002dabc0 | CONFIRMED | slot 5: describes the focused car; a row accepts it or opens the colour picker |

#### `GM_SelectCarModification` (`gui_race_car_color_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00569500 | `thunk_gui_menu_close` | - | - | - | slot 4 of the colour screen: jumps to `gui_menu_close` |
| 0x00569510 | `gui_race_car_color_tick` | `OnClick__24GM_SelectCarModificationFUi` | 0x002db100 | CONFIRMED | slot 5: a row repacks the colour into the car; `500` stores it |
| 0x005771a0 | `thunk_gui_menu_on_close_child` | - | - | - | slot 3 of the colour screen: jumps to `gui_menu_on_close_child`; the address is shared with `structs/game_menus.md` |

#### `GM_SingleRaceSecond` (`gui_race_launch_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00569ab0 | `gui_race_launch_on_close_child` | - | - | - | when the garage child (`0x1c7`) closes with a result, adopts the chosen car into `g_racing+0x64`, refreshes widget `200` with its name, rewrites every opponent row `0x640+i` from |
| 0x00569bd0 | `gui_race_launch_tick` | `OnClick__19GM_SingleRaceSecondFUi` | 0x002dba20 | CONFIRMED | slot 5: `0x1c3`..`0x1c6` to `0x2a`..`0x2d`, `0x1c9` back |

#### `GM_SingleRaceResults` (`gui_race_results_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056a570 | `gui_race_results_tick` | `OnClick__20GM_SingleRaceResultsFUi` | 0x002dc750 | CONFIRMED | slot 5: standings refresh, the 15 s countdown, F2 overlay; closes with `0x2f` |

#### `GM_BonusCar` (`gui_race_bonus_car_`)

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0056acd0 | `gui_race_bonus_car_tick` | `OnClick__11GM_BonusCarFUi` | 0x002dec60 | CONFIRMED | slot 5: closes the stack with `0x34`, or opens the Carcyclopedia on `0x1f6` |

### 3d. Free helpers

No accessor: every field is reached from a constructor, a tick or a child handler.

## 4. The class table

Every constructor installs a vtable from the contiguous run at `0x0063d434 + 0x18*i` and
builds a design id from `0x16` to `0x23`, **monotone with VA**. That is the whole index:

| design | vtable | PC class prefix | PS2 class | `menu.def` title | PC functions |
|---|---|---|---|---|---|
| `0x16` | `0x0063d434` | `gui_race_root_` | `GM_Racing` | "Racing:" | 4 |
| `0x17` | `0x0063d44c` | `gui_race_championship_` | `GM_Championship` | "Championship" | 2 |
| `0x18` | `0x0063d464` | `gui_race_round_select_` | `GM_SelectInRound` | "Select Circuit:" (rounds) | 2 |
| `0x19` | `0x0063d47c` | `gui_race_champ_launch_` | `GM_ChampionshipRace` | "Race %d" | 3 |
| `0x1a` | `0x0063d494` | `gui_race_champ_results_` | `GM_ChampionshipResults` | "Racing:" (results) | 2 |
| `0x1b` | `0x0063d4ac` | `gui_race_setup_` | `GM_SingleRace` | "Single Race:" | 3 |
| `0x1c` | `0x0063d4c4` | `gui_race_circuit_select_` | `GM_SelectCircuit` | "Select Circuit:" | 2 |
| `0x1d` | `0x0063d4dc` | `gui_race_opponents_` | `GM_SelectOpponents` | "Opponents:" | 3 |
| `0x1e` | `0x0063d4f4` | `gui_race_car_maker_` | `GM_SelectCarManufacturer` | "Garage:" | 3 |
| `0x1f` | `0x0063d50c` | `gui_race_car_model_` | `GM_SelectCarModel` | "Models:" | 6 |
| `0x20` | `0x0063d524` | `gui_race_car_color_` | `GM_SelectCarModification` | "Car Colors:" | 2 |
| `0x21` | `0x0063d53c` | `gui_race_launch_` | `GM_SingleRaceSecond` | "Single race" | 3 |
| `0x22` | `0x0063d554` | `gui_race_results_` | `GM_SingleRaceResults` | "Racing:" (results) | 2 |
| `0x23` | `0x0063d56c` | `gui_race_bonus_car_` | `GM_BonusCar` | "You have won a bonus car" | 2 |

Plus `thunk_gui_menu_close` at `0x00569500`, `gui_race_car_color_`'s slot 4: 40 starts, all named, between the guard pair and the next unit's. The colour screen's slot 3 is `thunk_gui_menu_on_close_child`, at `0x005771a0` in the unit of `structs/game_menus.md`.

## 5. How the class map is established

The constructors say almost nothing on their own: chain the base, install a vtable, push an
integer, call the builder. **That integer is the answer.** Decoding the 14 `menu.def` groups
and resolving every `text_id` through `textdb.json` names all 14 screens without reading a
body; §10 holds the cross-build check.

**Read the fr/cz columns of `textdb.json`, not en.** Retail English overwrote ids
410-412 with debug strings ("Controls unlocked", "Thrower spawned") when racing was cut;
fr/cz still say *Course simple* / *Championnat*.

**Entry points.** `gui_race_root_open_single_race` and `gui_race_root_open_championship`
build a whole stack in one call — root → setup → launch, or root → list → rounds → launch —
and set `g_menu_camera_item_id` to `0x18453` / `0x18454` so the `00menu` camera flies to the
right frame.

## 6. The garage chain

Five screens form one pipeline, each handing an out-pointer down and a car id back up:

```
  gui_race_setup_tick  (0x1d8 GARAGE)
        └─ gui_race_car_maker_ctor(out_ptr, race_mask)          design 0x1e, new(0x54)
              └─ gui_race_car_model_ctor(out_ptr, maker, mask)  design 0x1f, new(0x70)
                    └─ gui_race_car_color_ctor(out_ptr, car)    design 0x20, new(0x58)
```

The manufacturer screen runs **three erase filters** over each maker's car list —
`car_database_is_enabled_for_race`, `_for_championship`, `_for_freeride` with the shared
literal `| 0x2000000` — and disables the maker's row when nothing survives, in an `i < 0x14`
maker loop. The model screen runs the same three per car.

Each screen hands its own field down as the out pointer (§1): the launch and setup screens' `chosen_car`, the maker's, the model's. The colour screen lists one colour name per colour index
(`car_database_get_color_name_id`).

`gui_race_opponents_tick` reaches the same manufacturer screen for each AI slot.

## 7. The championship / single-race pair

The two halves of the mode are near-mirrors:

| role | championship | single race |
|---|---|---|
| list | `gui_race_championship_*` (0x17) | - |
| pick a track | `gui_race_round_select_*` (0x18) | `gui_race_circuit_select_*` (0x1c) |
| launch | `gui_race_champ_launch_*` (0x19) | `gui_race_launch_*` (0x21) |
| results | `gui_race_champ_results_*` (0x1a) | `gui_race_results_*` (0x22) |

**The discriminators, all measured:**

* the two launch screens share `0x1c3`..`0x1c6` and `0x1c9`, but only the championship one
  has `0x1c7` (GARAGE);
* the two results screens share six widget ids and differ in one return code (`0x33`
  championship, `0x2f` single race).

Both results screens hold one slot-5 function: a 500 ms standings
refresh, a **15 000 ms auto-continue** countdown printed into widget `0x78`, and `F2` to
toggle the overlay through `racing_toggle_results_visibility`.

`gui_race_champ_launch_tick`'s `0x1c8` arm toggles `racing_set_damage_enabled` inline and swaps
the label between text `0x1ca` and `0x1cb`; `gui_race_setup_tick` has no `0x1dc` arm.

## 8. `GM_BonusCar`

Design `0x23`, the award screen. Its constructor loads the car-database details, formats
the awarded car's name and variant into one string (textdb `0x45d`/`0x45e` supply the
variant wording) and writes it into widget `10` between a `car_database_load_details` /
`car_database_unload_details` pair. Its slot 5 closes the whole stack with `0x34`, or opens
the Carcyclopedia on `0x1f6`.

## 9. `C_racing` / `C_championship` fields this TU reads

| field | meaning |
|---|---|
| `g_racing+0x08` | the circuit vector handed to the circuit picker |
| `g_racing+0x1c` / `+0x20` | the championship vector, begin / end |
| `g_racing+0x28` | **the racing mode: 1 = championship, 2 = single race** |
| `g_racing+0x34` | the current `C_champ_circuit`: the championship launch and results screens list its `+0x68` best-race table; the single-race launch lists the current `C_circuit`'s `+0x58` best-lap table instead |
| `g_racing+0x38` | the current championship |
| `g_racing+0x3c` | the opponent difficulty, 0..4; label text `0x1ef + level` |
| `g_racing+0x64` | the player's chosen car (packed id) |
| `g_racing+0x27c` | lap count, clamped 1..99 |
| `g_racing+0x28c` | the standings row count (`structs/racing.md` §1a): both results screens walk that many rows of the racer array `+0x228` |
| `g_racing+0x2b4` | `mirror_track`, the "Track Mode: Normal" toggle |
| `g_racing+0x2b7` | damage-enabled flag; text `0x1ca` on, `0x1cb` off |
| `C_championship+0xac` | the round vector |
| `C_championship+0xcc` | the championship's state: 0 open · 1 shown-not-set · 2+ locked |
| `C_championship+0xd4` / `+0xd8` | opponent count / championship car mask |
| `C_circuit+0x50` | the circuit's state; `!= 2` means selectable |

Two data symbols this TU touches: `g_gui_race_champ_results_refresh_countdown_ms` (the 500 ms results-refresh
countdown, 3 refs) and `g_gui_race_champ_launch_circuit_bmp_name` (the `circuit%d.bmp` scratch buffer, 3 refs; a shared
buffer, §12).

## 10. PC vs PS2

**85 PS2 methods across 15 classes map onto 39 PC functions**, and the difference is accounted for by §10a:

* the PC **fuses** `OnBeforeCreate` + `OnCreate` into one constructor;
* the PC **folds** `OnTick`, `OnKey`, `OnChange` and `OnCloseChild` into slot 5 wherever the
  behaviour is small: 2→1 on the circuit picker, the car-model picker, the colour picker,
  the opponent screen and the bonus-car screen; 3→1 on both results screens;
* **one PS2 class has no PC counterpart at all**, and the absence is measured:
  * **`GM_DamageDifficulty`** — the PS2 `GM_SingleRace::OnClick` case `0x1dc` and
    `GM_ChampionshipRace::OnClick` case `0x1c8` each `operator_new(0x30)` this screen. The
    PC has neither (§7). The difficulty half of that PS2 screen (`CRacing+0xab0`) has no PC
    equivalent either.

**`GM_GameMenuRacing` is real on the PC, in another unit.** The in-race pause menu is `gui_racing_game_menu_ctor` (`0x0057cab0`), design `0x2e`, vtable `0x0063e08c`, opened by `race_game_loop` and `race_game_loop_net` (`structs/game_menus.md` §8).

**Emission order differs between the builds.** The PC follows the `menu.def` group order
(`GM_Racing`, `GM_Championship`, `GM_SelectInRound`, `GM_ChampionshipRace`,
`GM_ChampionshipResults`, `GM_SingleRace`, …); the PS2 emits `GM_Racing`, `GM_SingleRace`,
`GM_SelectCircuit`, `GM_SelectCarManufacturer`, `GM_SelectCarModel`, … An order-align over
this TU gives wrong ranks for eleven of the fourteen classes; the design-id join (§5) does not
depend on order.

**Per-screen comparison:**

* **Class map** — each of the 14 design-id screens (§5) is independently CONFIRMED by an
  `OnClick` case-set diff against the PS2 method of that name.
* **Garage chain** — the manufacturer and model filters are a lockstep with the PS2: same
  filters, same order, same literal, same `i < 0x14` maker loop.
* **Colour screen** — the one place the bodies genuinely diverge: the PS2 builds a
  colour-swatch vector and `SetPointer`s it into the list widget instead of listing colour
  names, so its row is STRUCTURAL.
* **Launch screens** — only the championship one has `0x1c7` (GARAGE) on both builds.
* **Constructor discriminators** — `0x224`/`0x225` appear only in the PS2 championship-results
  body, `0x212`/`0x213`/`0x216`/`0x21b`/`0x231`/`0x708` only in the single-race results,
  `0x515`/`0x5dd`/`0x641` only in the single-race launch.
* **`GM_BonusCar`** — identical callee sequence to the PS2; the Carcyclopedia arm (`0x1f6`) is
  PC-only.
* **Opponent grid** — `gui_race_opponents_ctor` clones a **constant** 19 rows and walks 20, and
  `gui_race_opponents_tick` passes a **hardcoded `0x14`** to `racing_add_opponents`. The PS2
  derives both from the circuit's own `+0x08`. So a PC single-race grid should always be 20
  cars regardless of circuit.
* **Colour-picker gate** — `gui_race_car_model_tick` opens the colour picker when
  `car_database_get_max_color_index >= 2`; the PS2 opens it when `GetCarMaxColor != 0`.

**The fields of §9 on the other build:**

| PC field | PS2 field |
|---|---|
| `g_racing+0x1c` / `+0x20` | `CRacing+0x604` / `+0x600` (ptr + count) |
| `g_racing+0x28` | `CRacing+0xaa8` |
| `g_racing+0x34` | `CRacing+0xa58` |
| `g_racing+0x38` | `CRacing+0xa5c` |
| `g_racing+0x64` | `CRacing+0x580` |
| `g_racing+0x27c` | `CRacing+0xab8` |
| `g_racing+0x2b4` | `CRacing+0xb04` |
| `C_championship+0xac` | `+0x44` |
| `C_championship+0xcc` | `+0x30` |
| `C_circuit+0x50` | `+0x00` |

### 10a. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002d9a00 | `OnChange__9GM_RacingFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_race_root_tick` |
| 0x002d9cf0 | `OnTick__9GM_RacingFUi` | folded into slot 5, `gui_race_root_tick` |
| 0x002da030 | `OnKey__13GM_SingleRaceFUcUi` | folded into slot 5, `gui_race_setup_tick` |
| 0x002da070 | `OnTick__13GM_SingleRaceFUi` | folded into slot 5, `gui_race_setup_tick` |
| 0x002da420 | `OnChange__16GM_SelectCircuitFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_race_circuit_select_tick` |
| 0x002dad20 | `OnChange__17GM_SelectCarModelFP12GM_ComponentP12GM_Component` | folded into slot 5, `gui_race_car_model_tick` |
| 0x002dae70 | `GetCarColor__24GM_SelectCarModificationFUiUi` | the PC lists colour names through `car_database_get_color_name_id`; no swatch accessor exists |
| 0x002db020 | `OnTick__24GM_SelectCarModificationFUi` | folded into slot 5, `gui_race_car_color_tick` |
| 0x002db070 | `OnKey__24GM_SelectCarModificationFUcUi` | folded into slot 5, `gui_race_car_color_tick` |
| 0x002db290 | `OnKey__18GM_SelectOpponentsFUcUi` | folded into slot 5, `gui_race_opponents_tick` |
| 0x002dc850 | `OnKey__20GM_SingleRaceResultsFUcUi` | folded into slot 5, `gui_race_results_tick` |
| 0x002dc920 | `OnTick__20GM_SingleRaceResultsFUi` | folded into slot 5, `gui_race_results_tick` |
| 0x002dc980 | `OnCreate__19GM_DamageDifficultyFv` | the class has no PC counterpart: the PC has neither `case` that opens it (§7) |
| 0x002dca10 | `OnClick__19GM_DamageDifficultyFUi` | the class has no PC counterpart: the PC has neither `case` that opens it (§7) |
| 0x002dcab0 | `OnKey__19GM_DamageDifficultyFUcUi` | the class has no PC counterpart: the PC has neither `case` that opens it (§7) |
| 0x002dcf70 | `OnTick__15GM_ChampionshipFUi` | folded into slot 5, `gui_race_championship_tick` |
| 0x002dd2f0 | `OnTick__16GM_SelectInRoundFUi` | folded into slot 5, `gui_race_round_select_tick` |
| 0x002de8d0 | `OnTick__22GM_ChampionshipResultsFUi` | folded into slot 5, `gui_race_champ_results_tick` |
| 0x002de9f0 | `OnKey__22GM_ChampionshipResultsFUcUi` | folded into slot 5, `gui_race_champ_results_tick` |
| 0x002dec90 | `OnKey__11GM_BonusCarFUcUi` | folded into slot 5, `gui_race_bonus_car_tick` |
| 0x002decf0 | `__dt__11GM_BonusCarFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002ded50 | `OnBeforeCreate__11GM_BonusCarFv` | fused into `gui_race_bonus_car_ctor` |
| 0x002ded60 | `__dt__22GM_ChampionshipResultsFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002dedc0 | `OnBeforeCreate__22GM_ChampionshipResultsFv` | fused into `gui_race_champ_results_ctor` |
| 0x002dee70 | `__dt__19GM_ChampionshipRaceFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002deed0 | `OnBeforeCreate__19GM_ChampionshipRaceFv` | fused into `gui_race_champ_launch_ctor` |
| 0x002deef0 | `__dt__16GM_SelectInRoundFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002def50 | `OnBeforeCreate__16GM_SelectInRoundFv` | fused into `gui_race_round_select_ctor` |
| 0x002def60 | `__dt__15GM_ChampionshipFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002defc0 | `OnBeforeCreate__15GM_ChampionshipFv` | fused into `gui_race_championship_ctor` |
| 0x002defd0 | `__dt__19GM_DamageDifficultyFv` | the class has no PC counterpart: the PC has neither `case` that opens it (§7) |
| 0x002df030 | `OnBeforeCreate__19GM_DamageDifficultyFv` | the class has no PC counterpart: the PC has neither `case` that opens it (§7) |
| 0x002df040 | `__dt__20GM_SingleRaceResultsFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df0a0 | `OnBeforeCreate__20GM_SingleRaceResultsFv` | fused into `gui_race_results_ctor` |
| 0x002df0b0 | `__dt__19GM_SingleRaceSecondFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df110 | `OnBeforeCreate__19GM_SingleRaceSecondFv` | fused into `gui_race_launch_ctor` |
| 0x002df130 | `__dt__18GM_SelectOpponentsFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df190 | `OnBeforeCreate__18GM_SelectOpponentsFv` | fused into `gui_race_opponents_ctor` |
| 0x002df1a0 | `__dt__24GM_SelectCarModificationFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df220 | `OnBeforeCreate__24GM_SelectCarModificationFv` | fused into `gui_race_car_color_ctor` |
| 0x002df2b0 | `OnBeforeCreate__17GM_SelectCarModelFv` | fused into `gui_race_car_model_ctor` |
| 0x002df2c0 | `__dt__24GM_SelectCarManufacturerFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df320 | `OnBeforeCreate__24GM_SelectCarManufacturerFv` | fused into `gui_race_car_maker_ctor` |
| 0x002df340 | `__dt__16GM_SelectCircuitFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df3a0 | `OnBeforeCreate__16GM_SelectCircuitFv` | fused into `gui_race_circuit_select_ctor` |
| 0x002df3c0 | `__dt__13GM_SingleRaceFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |
| 0x002df420 | `OnBeforeCreate__13GM_SingleRaceFv` | fused into `gui_race_setup_ctor` |
| 0x002df440 | `__dt__9GM_RacingFv` | slot 0 is the shared `gui_menu_scalar_deleting_dtor_common`, which calls `gui_menu_dtor` |

## 11. Related

`structs/gui_screen.md` (the `GM_Menu` base, the vtable map, the design table) · `structs/gui_widget.md` (the widgets) · `structs/game_menus.md`, `structs/multiplayer.md` (the two other screen units) · `structs/racing.md` (`C_racing` and its records) · `structs/car_database.md` · `structs/race_session.md` (the drivers that open these menus) · `basic/artifacts/racing.md` (the feature).

## 12. Open items

| item | state |
|---|---|
| root menu and garage chain | in game (needs the racing mod — retail deletes the menu item): Main Menu → Racing should show the four-item root menu; Single Race → GARAGE should walk the three-screen car chain |
| single-race grid size | in game: should hold 20 opponents regardless of circuit (§10) |
| damage toggle | in game: the toggle on the championship launch screen should change its own label rather than open a sub-screen — the visible form of the missing `GM_DamageDifficulty` |
| `g_gui_race_champ_launch_circuit_bmp_name` | a shared buffer; its writers outside this TU are not censused |
| `GM_SelectOpponents.opponent_cars` | the ctor zeroes ten of the 20 entries, then reads `entrants[i]` `+0x24` for all 20 slots; a slot past the entrant count is 0 (`racing_ctor`, `racing_clear_opponents` and `racing_remove_opponent_at` null it), so entries 10..19 are tested against 0 while unwritten whenever their slot is empty. What they hold in game is not observed |
