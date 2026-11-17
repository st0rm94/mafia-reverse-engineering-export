# `G_GarageClass` — the player's car garage and its three screens

**Created** 2026-08-25 22:28 +0200 · **Modified** 2026-10-31 11:17 +0100

The garage is where the player's owned cars are kept between missions. It has a hard capacity, and when a
mission ends with more cars than the cap the game forces the player to discard some: two of the three screens
exist for that, and the third is the post-mission report. On the PC the manager has no object of its own, so its
fields are a band of the `G_LoadSaveClass` singleton at `0x00671fb0`.

| | |
|---|---|
| PC prefix | `garage_` for the manager, `gui_garage_select_`, `gui_garage_full_`, `gui_garage_summary_` for the screens |
| PC kind (`+0x10`) | - |
| PC vtable | manager -; screens `g_gui_menu_vt_garage_select` @ `0x0063e02c`, `g_gui_menu_vt_garage_full` @ `0x0063e044`, `g_gui_menu_vt_garage_summary` @ `0x0063e05c`, 6 slots each |
| PC base class | manager -; screens `GM_Menu` (`structs/gui_screen.md`) |
| PC instance size | manager -, embedded in `G_LoadSaveClass`; screens `0x54`, `0x50`, `0x48` |
| PC constructor | manager -; screens `gui_garage_select_ctor`, `gui_garage_full_ctor`, `gui_garage_summary_ctor` |
| PC translation unit | manager `[0x0055d9e0, 0x0055ff70)`, inside the unit hosted by `structs/loadsave.md`; screens `[0x0057b930, 0x0057c8e0)`, inside the unit of `structs/game_menus.md` |
| PS2 class | `G_GarageClass` (tag `13G_GarageClass`), 40 methods; `GM_GarageSelect` 8, `GM_GarageFull` 7, `GM_GarageSummary` 5 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Offsets are `G_LoadSaveClass` offsets, the `this` every `garage_*` function receives. Only the garage's own fields
are listed: `+0x9c` between them is the Free Ride mask, whose setter and getter are `G_LoadSaveClass` methods (§4b),
and `+0x1d0..+0x1e7` belong to `structs/loadsave.md`.

```c
typedef struct G_GarageClass {              // a band of G_LoadSaveClass, no object of its own
    /* +0x098 */ uint32_t      steal_mask;              // [H] bit n = car group 'A' + n may be stolen (§4b)
    /* +0x0a0 */ S_garage_slot slots[25];               // [H] the parked cars
    /* +0x168 */ uint32_t      previous_packed_ids[25]; // [H] slots[n].packed_car_id at the last results screen
    /* +0x1cc */ uint8_t       show_results_request;    // [H] one-shot, set by the script command garage_show
    /* +0x1cd */ uint8_t       add_last_stolen_request; // [H] one-shot, set by garage_addlaststolen
    /* +0x1e8 */ C_car        *last_entered_car;        // [H] the last car the player got into
    /* +0x1ec */ C_car        *blocked_car;             // [H] the car the player cannot get out of
} G_GarageClass;
```

**Why 25 slots.** `0x19` is the loop bound of every walker of `slots` and of `previous_packed_ids`. Each entry of
`slots` is one parked car; an empty entry has both fields zero. Twenty-five is not the capacity the player sees: the
over-cap loop of `garage_game_done` cuts the count to 16 (§5a), and `garage_add_car_by_index` /
`garage_add_car_by_actor` drop a car silently only when all 25 are taken. No body states why the table is larger than
the cap.

**Why two parallel arrays of 25.** `previous_packed_ids[n]` is the id `slots[n]` held when `garage_show_results` last
ran. That function compares the two index by index to find the cars gained and lost, then copies the current ids
across.

### 1a. `S_garage_slot` — one parked car (`0x08`)

```c
typedef struct S_garage_slot {
    /* +0x00 */ uint32_t packed_car_id;  // [H] car index in the low word, colour in byte 3; byte 2 is cleared on parking
    /* +0x04 */ C_car   *live;           // [H] the car while it stands in the scene, else 0
} S_garage_slot;
```

### 1b. `S_garage_work_car` — one row of the working list (`0x0c`)

`garage_game_done` and `garage_run_car_select_menu` build a `std::vector` of these on the stack, from `slots` and from
the cars standing in the garage area. It has no set count.

```c
typedef struct S_garage_work_car {
    /* +0x00 */ uint32_t packed_car_id;  // [H] as S_garage_slot; 0 when only `live` is known
    /* +0x04 */ C_car   *live;           // [H] the car, or 0 for a car that is not in the scene
    /* +0x08 */ int      parked_slot;    // [H] index into slots (0..24), or -1 for a car not in the table
} S_garage_work_car;
```

### 1c. `GM_GarageSelect` — the car manager screen (`0x54`)

`operator_new(0x54)` in `garage_run_car_select_menu`. The base `GM_Menu` ends at `+0x48`.

```c
typedef struct GM_GarageSelect {        // : GM_Menu
    /* +0x48 */ void *cars;             // [H] the caller's vector of S_garage_work_car; the screen builds a row per entry
    /* +0x4c */ void *ticked;           // [H] the caller's vector of row indices ticked for discard
    /* +0x50 */ int  *focused_id;       // [H] the caller's int: the focused widget's id survives the reopen
} GM_GarageSelect;
```

### 1d. `GM_GarageFull` — the forced discard screen (`0x50`)

`operator_new(0x50)` in `garage_game_done`.

```c
typedef struct GM_GarageFull {          // : GM_Menu
    /* +0x48 */ void *cars;             // [H] the caller's vector of S_garage_work_car
    /* +0x4c */ void *discarded;        // [H] the caller's vector of row indices picked so far
} GM_GarageFull;
```

`GM_GarageSummary` allocates `0x48` bytes in `garage_show_results` and adds no field to `GM_Menu`: its constructor
reads its two vectors from its parameters and keeps neither.

## 2. Field evidence

Each table covers one typedef of §1. The cells were read from the bodies: `garage_game_done` and
`garage_run_car_select_menu` reach the singleton through a saved register, which a census of `this` alone misses.

### 2a. `G_GarageClass`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x98 | steal_mask | H | `garage_enable_steal_for_letter`, `loadsave_freeride_apply_script_params`, `loadsave_savegame_new_game`, `loadsave_savegame_load` | `garage_can_steal_car_model`, `loadsave_execute_credits`, `loadsave_savegame_save` | - |
| +0xa0 | slots | H | `garage_add_car_by_actor`, `garage_add_car_by_index`, `garage_clear`, `garage_destroy_car`, `garage_game_done`, `garage_game_init`, `garage_remove_car_by_model`, `loadsave_savegame_load` | `garage_add_car_by_actor`, `garage_add_car_by_index`, `garage_game_done`, `garage_game_init`, `garage_is_car_parked`, `garage_remove_car_by_model`, `garage_run_car_select_menu`, `garage_show_results`, `loadsave_savegame_save` | - |
| +0x168 | previous_packed_ids | H | `garage_clear`, `garage_show_results`, `loadsave_savegame_load` | `garage_show_results`, `loadsave_savegame_save` | - |
| +0x1cc | show_results_request | H | `garage_game_done`, `garage_game_init`, `garage_request_show_results` | `garage_game_done` | - |
| +0x1cd | add_last_stolen_request | H | `garage_game_done`, `garage_game_init`, `garage_request_add_last_stolen` | `garage_game_done` | - |
| +0x1e8 | last_entered_car | H | `garage_game_done`, `garage_set_last_entered_car`, `loadsave_init`, `loadsave_savegame_new_game` | `garage_game_done` | - |
| +0x1ec | blocked_car | H | `garage_set_blocked_car` | `garage_run_car_select_menu` | the select screen disables the row of this car |

### 2b. `S_garage_slot`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | packed_car_id | H | `garage_add_car_by_actor`, `garage_add_car_by_index`, `garage_clear`, `garage_destroy_car`, `garage_game_done`, `garage_remove_car_by_model` | `garage_game_init`, `garage_remove_car_by_model`, `garage_show_results` | built by `car_database_pack_id`; `structs/car_database.md` |
| +0x4 | live | H | `garage_add_car_by_actor`, `garage_add_car_by_index`, `garage_clear`, `garage_destroy_car`, `garage_game_done`, `garage_game_init`, `garage_remove_car_by_model`, `loadsave_savegame_load` | `garage_add_car_by_actor`, `garage_game_init`, `garage_is_car_parked` | - |

### 2c. `S_garage_work_car`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | packed_car_id | H | `garage_game_done`, `garage_run_car_select_menu` | `garage_game_done`, `gui_garage_full_ctor`, `gui_garage_select_ctor` | a zero id is resolved from `live` by `car_database_pack_id_from_car` |
| +0x4 | live | H | `garage_game_done`, `garage_run_car_select_menu` | `garage_destroy_car`, `garage_game_done`, `gui_garage_full_ctor`, `gui_garage_select_ctor` | - |
| +0x8 | parked_slot | H | `garage_game_done`, `garage_run_car_select_menu` | `garage_destroy_car`, `garage_game_done` | the discard screens never read it |

### 2d. `GM_GarageSelect`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | cars | H | `gui_garage_select_ctor` | `gui_garage_select_ctor` | - |
| +0x4c | ticked | H | `gui_garage_select_ctor` | `gui_garage_select_ctor`, `gui_garage_select_tick` | - |
| +0x50 | focused_id | H | `gui_garage_select_ctor` | `gui_garage_select_ctor`, `gui_garage_select_tick` | the tick stores `focused`'s id `+0x04` through it; the ctor reactivates that widget when it is enabled |

### 2e. `GM_GarageFull`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | cars | H | `gui_garage_full_ctor` | `gui_garage_full_ctor` | - |
| +0x4c | discarded | H | `gui_garage_full_ctor` | `gui_garage_full_ctor`, `gui_garage_full_tick` | - |

## 3. Method family

60 PS2 methods across the four classes: 19 have a PC function of their own, one is inlined, and 40 are in the PS2-only table of §6c. Two PC rows are shared `GM_Menu` code with no twin. `loadsave_freeride_enable_car_group`, inside the manager's address range, is a `G_LoadSaveClass` method; `structs/loadsave.md` lists it.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00578790 | `gui_menu_scalar_deleting_dtor_common` | instance | `__dt__7GM_MenuFv` | 0x002e4260 | STRUCTURAL | slot 0 of all three screens, shared with every field-less `GM_Menu` subclass; its twin is the delete half; §6c lists the PS2 destructors |
| 0x0057b930 | `gui_garage_select_ctor` | instance | `OnCreate__15GM_GarageSelectFv` | 0x0048f3e0 | CONFIRMED | builds design `0x25` and its rows; 3 -> 1 fold with `OnTick` and `OnClose`'s `UnloadDetails` (§5) |
| 0x0057bfa0 | `gui_garage_full_ctor` | instance | `OnCreate__13GM_GarageFullFv` | 0x0048fbb0 | CONFIRMED | builds design `0x26` and its rows; 3 -> 1 fold, the same two absorbed (§5) |
| 0x0057c540 | `gui_garage_summary_ctor` | instance | `OnCreate__16GM_GarageSummaryFv` | 0x0048ff80 | CONFIRMED | builds design `0x27`; its lockstep gained / lost row build is a shape no sibling screen has (§5b) |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0055fc90 | `garage_request_add_last_stolen` | `AddLastStolen__13G_GarageClassFv` | 0x0048bf30 | CONFIRMED | stores 1 to `add_last_stolen_request` (§4a) |
| 0x0055fca0 | `garage_request_show_results` | `ShowResults__13G_GarageClassFv` | 0x0048bef0 | CONFIRMED | stores 1 to `show_results_request` (§4a) |
| 0x0055fea0 | `garage_set_last_entered_car` | `CarIsStolen__13G_GarageClassFP5C_car` | 0x0048bf00 | CONFIRMED | stores `last_entered_car` |
| 0x0055feb0 | `garage_set_blocked_car` | `CarCantGetOut__13G_GarageClassFP5C_car` | 0x0048bf20 | CONFIRMED | stores `blocked_car` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0055d9e0 | `garage_clear` | `Clear__13G_GarageClassFv` | 0x0048a5c0 | CONFIRMED | zeroes `slots` and `previous_packed_ids` |
| 0x0055da10 | `garage_add_car_by_index` | `AddCarByIdx__13G_GarageClassFUiUi` | 0x0048a670 | CONFIRMED | parks a packed id in the first empty slot, then enables its race unlock even when the table is full |
| 0x0055da90 | `garage_add_car_by_actor` | `AddCarByActor__13G_GarageClassFP5C_car` | 0x0048a700 | CONFIRMED | parks a live car; returns at once if it holds a slot; a full table drops it |
| 0x0055db40 | `garage_remove_car_by_model` | `RemoveCar__13G_GarageClassFPc` | 0x0048a830 | CONFIRMED | clears the first slot whose car index matches the model name |
| 0x0055db90 | `garage_game_init` | `GameInit__13G_GarageClassFv` | 0x0048a960 | CONFIRMED | per-scene setup: clears the two request bytes and spawns each parked car in the garage area |
| 0x0055e6e0 | `garage_game_done` | `GameDone__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>` | 0x0048b3d0 | CONFIRMED | per-scene teardown: the four banking paths, the over-cap discard loop (§5a), the slot rewrite |
| - | - | `HlidejPocetVect__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>ib` | 0x0048d610 | inlined | the over-cap loop inside `garage_game_done` (§6) |
| 0x0055f510 | `garage_run_car_select_menu` | `CarManager__13G_GarageClassFv` | 0x0048bbf0 | CONFIRMED | the in-garage car manager: rebuilds and reopens design `0x25` while its tick returns `0x4f9`, then destroys the ticked cars |
| 0x0055fa90 | `garage_destroy_car` | `DestroyCar__13G_GarageClassFP5C_car` | 0x0048d0f0 | CONFIRMED | takes one `S_garage_work_car`, removes its car from the world and clears its slot |
| 0x0055fcb0 | `garage_show_results` | `GarageShowResultsProc__13G_GarageClassFv` | 0x0048b910 | CONFIRMED | diffs `slots` against `previous_packed_ids`, runs design `0x27` modally, then copies the ids across |
| 0x0055fec0 | `garage_enable_steal_for_letter` | `EnableSteal__13G_GarageClassFUc` | 0x0048bf80 | CONFIRMED | ORs a letter's bit into `steal_mask` (§4b) |
| 0x0055ff00 | `garage_can_steal_car_model` | `CanSteal__13G_GarageClassFPUc` | 0x0048bff0 | CONFIRMED | `steal_mask` against the car's `carindex.def` `+0x8c` |
| 0x0055ff40 | `garage_is_car_parked` | `IsCarIn__13G_GarageClassFP5C_car` | 0x0048bf40 | CONFIRMED | true when some slot's `live` is the car |
| 0x005771a0 | `thunk_gui_menu_on_close_child` | - | - | - | slot 3 of the select and full screens; shared `GM_Menu` code, see `structs/gui_screen.md` |
| 0x0057bd10 | `gui_garage_select_tick` | `OnClick__15GM_GarageSelectFUi` | 0x0048fb50 | CONFIRMED | slot 5; toggles a row in `ticked`, returns `0x4f9` to reopen, `0x4fa` on close; 2 -> 1 fold with `OnKey` |
| 0x0057c340 | `gui_garage_full_tick` | `OnClick__13GM_GarageFullFUi` | 0x0048ff20 | STRUCTURAL | slot 5; appends the picked row to `discarded` and returns `0x4f8`; 2 -> 1 fold with `OnKey` |
| 0x0057c880 | `gui_garage_summary_tick` | `OnKey__16GM_GarageSummaryFUcUi` | 0x004901a0 | CONFIRMED | slot 5; any key or the Close button (id 100) closes; 2 -> 1 fold with the empty `OnClick` |

### 3d. Free helpers

None: every function of the two ranges is a method of one of the four classes or of `G_LoadSaveClass`.

## 4. The manager

`garage_game_done` and `garage_run_car_select_menu` both locate the garage AREA in the scene, by frame name
`oblast_garaze_a` / `oblast_garaze_b` with a `watercity.`-prefixed fallback: the area is the X/Z rectangle spanned by the
two frames' world origins, height ignored (`basic/artifacts/garage/storing-cars-in-the-garage.md`).

`parked_slot` ties a working row to the table: `garage_destroy_car` clears the slot it names, and the tail of
`garage_game_done` frees the slots of discarded cars and gives each `-1` car the first free slot.

### 4a. The end-of-mission car-banking request pair

Measured from the script-VM dispatcher band `0x0bc`..`0x13f`. Each function is a single byte
store:

| function | writes | reached from |
|---|---|---|
| `garage_request_add_last_stolen` | `add_last_stolen_request = 1` | script command `garage_addlaststolen`, dispatcher case `0xc8` |
| `garage_request_show_results` | `show_results_request = 1` | script command `garage_show`, dispatcher case `0xc9` |

Both are cleared at the end of `garage_game_done` and by `garage_game_init`, and
**`garage_game_done` tests the two bytes in one condition**, so the two commands arm the same
behaviour: bank the last car the player got into (`last_entered_car`) wherever
it stands. That path requires the car NOT to be a police model (`car+0x1ff2` clear) and
`car_is_usable` to hold, refuses four car-database ids — **`0x2b` (`bugatti`) and `0x39`, `0x3a`,
`0x3d` (the three Bolt trucks)** — appends the packed id and calls `loadsave_race_cars_enable`. The
area scan that banks cars standing in the garage area refuses only the three trucks.
`garage_show` additionally opens the results screen. The four banking paths are
written up in `basic/artifacts/garage/storing-cars-in-the-garage.md`.

Do not confuse these with the script commands `garage_generatecars` (case `0x5b`) and
`garage_releasecars` (case `0x5c`), which are a dead pair — both are one call to the shared
do-nothing stub `stub_noop_cdecl_00574a40` and reach neither function above.

### 4b. The two car-group letter masks

`garage_enable_steal_for_letter` and `loadsave_freeride_enable_car_group` decode a letter the same
way — `A`..`Z` or `a`..`z` to bit `letter - 'A'`, any other byte to bit 0 — and OR the bit into one
mask of the singleton:

| mask | setter | script command | tested against |
|---|---|---|---|
| `steal_mask` `+0x98` | `garage_enable_steal_for_letter` | `garage_enablesteal` | `carindex.def +0x8c`, by `garage_can_steal_car_model` (`structs/car_database.md` §1) |
| Free Ride `+0x9c` (`G_LoadSaveClass`) | `loadsave_freeride_enable_car_group` | `freeride_enablecar` (case `0x176`) | `carindex.def +0xa0` (`car_database_is_enabled_for_freeride`) or the Carcyclopedia mask |

The Free Ride mask is read back by `loadsave_freeride_get_enabled_cars`, whose three callers — the
Carcyclopedia list builder, `gui_race_car_maker_ctor` and `gui_race_car_model_ctor` — OR in
`0x2000000`, bit `Z`, before testing. Every production car in the shipped `carindex.def` carries
only `Z`, so production cars are always available; each of `A`..`T` unlocks exactly one bonus car:

| letter | car (`carindex.def` entry) |
|---|---|
| `A` | `thunderbird` (60) |
| `B` | `fordhot` (62) |
| `C` .. `Q` | entries 66 .. 80, `blackdragon` .. `hotrodp6`, one letter each |
| `R` | `phantom` (41) |
| `S` | the four racers `bugatti`, `miller`, `duesenberg`, `alfa8c` (43 .. 46) |
| `T` | `phantomtaxi` (81) |

`lowgas` (65) carries an empty mask and is never available in Free Ride. All 40 shipped
`freeride_enablecar` calls are in the mission `EXTREME`: each letter is granted once by the script
that rewards it (`ulicka_unlockcar` grants `C`, `zeppelin_end` grants `N`, ...) and once by the
script `cheat_bonuscars`, which grants `A`..`T` in one run.

## 5. The three screens

All three are `GM_Menu` subclasses in the `game_menus` TU, each built from one shipped
`tables\menu.def` design: group N of that file (`stop`-terminated `0x24`-byte component records) is
design id N.

| class | design | shipped caption | ctor | slot-5 tick | driven by |
|---|---|---|---|---|---|
| **car select** | `0x25` | *"Would you like to discard a car?"* (textdb 1141) | `gui_garage_select_ctor` `0x0057b930` | `gui_garage_select_tick` `0x0057bd10` | `garage_run_car_select_menu` |
| **garage full** | `0x26` | *"The garage is full"* (1111) + *"Which car do you want to discard from the Garage?"* (1112) | `gui_garage_full_ctor` `0x0057bfa0` | `gui_garage_full_tick` `0x0057c340` | `garage_game_done` (§3a) |
| **summary** | `0x27` | *"New cars have been added to the garage:"* (1101) / *"Cars destroyed during the mission:"* (1106) | `gui_garage_summary_ctor` `0x0057c540` | `gui_garage_summary_tick` `0x0057c880` | `garage_show_results` |

**The two discard screens are near-identical.** The discriminator is the CALLER, not the
row-building code, which they share almost line for line:

* design `0x25` is a **toggle** list. Its tick flips membership of `(id - 2000)` in the ticked set
  and returns `0x4f9`, the code that makes the caller rebuild and reopen it; ticked rows are
  prefixed with textdb `0x45f` *"(removed) "*. It has a **Close** row (id 5000).
* design `0x26` is a **single pick**. Its tick appends `(id - 1000)` to the discarded set and
  returns `0x4f8`. There is no Close — the caller reopens it until the garage is under its cap.

Both compose each row label as *name + colour name* from `car_database_get_car_name` and
`car_database_get_color_name_id`, and append textdb `0x45d` / `0x45e` for the colour count.
Both bracket the build with `car_database_load_details` / `..._unload_details`.

### 5a. The over-cap loop

`garage_game_done` contains this loop:

```
while (surviving_cars > (discarded - flag) + 0x10) {
    operator_new(0x50);
    gui_garage_full_ctor(0, 0, cars, discarded, kept_cars);
    gui_menu_execute_modal(1);
}
```

`discarded` counts the cars picked so far and `flag` is `show_results_request != 0`, so the loop holds the surviving
count to 16 plus the discarded rows, one fewer when the results screen is due. `kept_cars` is the `std::vector` that
`game_done` hands to `garage_game_done`, the cars carried into the next mission: the constructor disables their rows. Only
one of the two discard screens is constructed inside the loop (§6).

### 5b. The summary screen's two bands

Design `0x27` takes a **gate**: `1` = cars gained only, `2` = cars lost only, `4` = both (the `4`
variant adds a separator `line`). The constructor builds one `'mtxt'` row per gained car from base
id **1000** and one per lost car from base id **2000**, in lockstep — a shape no sibling screen
has (§3a). Its tick dismisses on **any** key
or on the shipped *Close* button (id 100).

## 6. PC vs PS2

The PS2 garage is **`[0x0048a5c0, 0x004904b0]`** — `G_GarageClass` plus five `GM_Garage*` screen
classes. That band lies outside the PS2's GUI band (`0x002c…0x002e`); an enumeration of the symbol
table by VA range, ignoring the class column, finds all of them.

**The over-cap loop is an inline.** MSVC inlined the PS2 method `HlidejPocetVect__13G_GarageClassF…`
(*watch the count*) into `garage_game_done`; on the PS2 `GameDone` calls it
out of line. The site carries a `// ===== PS2-INLINED HERE: … =====` banner in the annotated
decomp. The loop also identifies the design-`0x26` screen as `GM_GarageFull`: the shared car-name
composition would fit either discard screen, but only one of them is constructed inside this loop.

**The request pair.** `garage_request_add_last_stolen` and `garage_request_show_results` are
`AddLastStolen` and `ShowResults`: the PS2 bodies store 1 to `+0x7d` / `+0x7c`, both builds call
them from the same two script arms, and both `GameDone` bodies test the pair in one condition and
open the results screen on the second byte alone. The PS2 `GenerateCars` / `ReleaseCars` are empty
8-byte bodies, and the PC arms that call them (§4a) reach only the shared empty stub.

`G_GarageClass` state sits in its own object on the PS2 (the request pair at `+0x7c` / `+0x7d`) and inside `G_LoadSaveClass` on the PC.

`loadsave_freeride_enable_car_group` is `FreerideEnableCar__15G_LoadSaveClassFUc` (CONFIRMED): the same letter decode into the same mask, PC
`+0x9c` and PS2 `+0x2b8` joined by the CONFIRMED accessor pair `loadsave_freeride_get_enabled_cars` / `FreerideGetEnabledCars`. The PS2 body also
maps the digits `0`..`6` to bits 26..32, remaps `U` to `0`, and ORs `G_CarDatabase::EnableCarsForFree` into `+0x2b4`.

### 6a. The presentation differs, the roles do not

The PS2 screens are **console carousels**: `GM_GarageSelect::OnCreate` preloads one `ITexture` per
car through a `sprintf`-built name, `OnTick` keeps a ±10 window of those textures live and releases
the rest, and `OnClose` frees them. The PC has no previews at all — it builds a text list once in
the constructor, which is why the PC constructors absorb the PS2's `OnTick`.

The PS2 also shows a *"n/N"* counter (component 299) that the PC design does not ship, and its
`OnTick` calls `G_IndicatorsClass::DrawMessage` while textures stream in.

### 6b. The two PS2-only screens

**`GM_GarageGarage` and `GM_GarageComplete` are PS2-only.** `GM_GarageGarage` is a root menu —
change car (opens `GM_GarageSelect`), park car (`ParkCar`), back — and `GM_GarageComplete` the
end-of-mission notice that leads into it. Only `G_GarageClass::DoGarageMenu` builds them, and
`GM_GarageComplete` opens `GM_GarageGarage`. `DoGarageMenu`'s one caller is the PS2 `C_game::Tick`, gated on a byte that only the
PS2-only script command `GARAGE_VRATA` sets; `DoFreerideMenu`, reached only from the PS2-only
`GARAGE_FREERIDE`, opens `GM_GarageSelect`. Both commands are absent from the PC's command table
(`basic/artifacts/script-opcode-dispatch.md` §7).

### 6c. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0048a8a0 | `Vysum__13G_GarageClassFP5C_car` | the PC callers call nothing in this unit and touch no slot (`structs/loadsave.md`) |
| 0x0048a900 | `AutoVybuchlo__13G_GarageClassFP5C_car` | `car_update` calls nothing in this unit and touches no slot (`structs/loadsave.md`) |
| 0x0048bed0 | `GenerateCars__13G_GarageClassFv` | an empty body on both builds; the PC arm calls the shared `stub_noop_cdecl_00574a40` (§4a) |
| 0x0048bee0 | `ReleaseCars__13G_GarageClassFv` | an empty body on both builds; the PC arm calls the shared `stub_noop_cdecl_00574a40` (§4a) |
| 0x0048c040 | `CanStart__13G_GarageClassFPUc` | `player_ai_drive` starts the engine with no car-mask test (`structs/loadsave.md`) |
| 0x0048c0d0 | `SaveGameLoad__13G_GarageClassFP19G_SaveGame_GameHead` | open-coded in `loadsave_savegame_load`, which walks `slots` and `previous_packed_ids` itself |
| 0x0048c180 | `SaveGameSave__13G_GarageClassFP19G_SaveGame_GameHead` | open-coded in `loadsave_savegame_save`, which walks `slots` and `previous_packed_ids` itself |
| 0x0048c280 | `EnumCarsForChange__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>P5C_carUi` | its one caller is `GM_GarageGarage::OnCreate` (§6b) |
| 0x0048c370 | `EnumCars__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>P5C_carUi` | called from `DestroyCar`; `garage_destroy_car` has no such call |
| 0x0048c450 | `EnumCarsVect__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>PQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>Ui` | called from `CarManager` and `DestroyCar`; on the PC `garage_run_car_select_menu` and `garage_game_done` build the working list in line |
| 0x0048c570 | `GetNumOfCars__13G_GarageClassFP5C_car` | called from `DestroyCar`; `garage_destroy_car` has no such call |
| 0x0048c5c0 | `GetParkableCar__13G_GarageClassFv` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048c6a0 | `GetChangeableCar__13G_GarageClassFv` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048c7a0 | `ParkCar__13G_GarageClassFP5C_car` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048c940 | `ChangeCar__13G_GarageClassFP5C_carUi` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048cdf0 | `TakeCar__13G_GarageClassFUi` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048d440 | `HlidejPocet__13G_GarageClassFP5C_carib` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048d7f0 | `HlidejPocetHard__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>i` | called from the PS2 `GameDone`; the PC `garage_game_done` holds only the one over-cap loop of §5a |
| 0x0048d970 | `UdelejBum__13G_GarageClassFv` | reached only through the PS2-only `DoGarageMenu` / `DoFreerideMenu` (§6b) |
| 0x0048e790 | `DoGarageMenu__13G_GarageClassFbUi` | §6b |
| 0x0048eb80 | `DoFreerideMenu__13G_GarageClassFv` | §6b |
| 0x0048efb0 | `__dt__13G_GarageClassFv` | the band has no destructor on the PC: it lives in `G_LoadSaveClass` |
| 0x0048f000 | `OnKey__17GM_GarageCompleteFUcUi` | §6b |
| 0x0048f040 | `OnClick__17GM_GarageCompleteFUi` | §6b |
| 0x0048f130 | `OnCreate__15GM_GarageGarageFv` | §6b |
| 0x0048f220 | `OnKey__15GM_GarageGarageFUcUi` | §6b |
| 0x0048f260 | `OnClick__15GM_GarageGarageFUi` | §6b |
| 0x0048f360 | `OnCloseChild__15GM_GarageGarageFP7GM_MenuUiUi` | §6b |
| 0x0048f5a0 | `OnClose__15GM_GarageSelectFv` | console texture release (§6a); the `UnloadDetails` half is in `gui_garage_select_ctor` |
| 0x0048f620 | `OnTick__15GM_GarageSelectFUi` | console texture streaming (§6a); the PC builds its list once in the constructor |
| 0x0048fad0 | `OnKey__15GM_GarageSelectFUcUi` | folded into `gui_garage_select_tick` |
| 0x0048fcf0 | `OnClose__13GM_GarageFullFv` | console texture release (§6a); the `UnloadDetails` half is in `gui_garage_full_ctor` |
| 0x0048fd70 | `OnTick__13GM_GarageFullFUi` | console texture streaming (§6a); the PC builds its list once in the constructor |
| 0x0048fe90 | `OnKey__13GM_GarageFullFUcUi` | folded into `gui_garage_full_tick` |
| 0x004901e0 | `OnClick__16GM_GarageSummaryFUi` | an 8-byte stub, folded into `gui_garage_summary_tick` |
| 0x004901f0 | `__dt__16GM_GarageSummaryFv` | `gui_menu_scalar_deleting_dtor_common`, slot 0 (§3a) |
| 0x00490280 | `__dt__13GM_GarageFullFv` | `gui_menu_scalar_deleting_dtor_common`, slot 0 (§3a) |
| 0x00490310 | `__dt__15GM_GarageSelectFv` | `gui_menu_scalar_deleting_dtor_common`, slot 0 (§3a) |
| 0x004903c0 | `__dt__15GM_GarageGarageFv` | §6b |
| 0x00490450 | `__dt__17GM_GarageCompleteFv` | §6b |
| 0x00490250 | `OnBeforeCreate__16GM_GarageSummaryFv` | absorbed by `gui_garage_summary_ctor`, which builds the design itself; PS2 body not diffed (Open item 1) |
| 0x00490300 | `OnBeforeCreate__13GM_GarageFullFv` | absorbed by `gui_garage_full_ctor`, which builds the design itself; PS2 body not diffed (Open item 1) |
| 0x00490390 | `OnBeforeCreate__15GM_GarageSelectFv` | absorbed by `gui_garage_select_ctor`, which builds the design itself; PS2 body not diffed (Open item 1) |
| 0x00490440 | `OnBeforeCreate__15GM_GarageGarageFv` | §6b |
| 0x004904b0 | `OnBeforeCreate__17GM_GarageCompleteFv` | §6b |

## 7. Related

`basic/artifacts/garage/storing-cars-in-the-garage.md` (how a car gets stored: the garage area, the four banking paths, the PS2 difference) · `structs/loadsave.md` (the object holding the band, and the savegame block) · `structs/game_menus.md` (the screen unit) · `structs/gui_screen.md` (`GM_Menu`) · `structs/car_database.md` (car name, colour name and packed ids).

## 8. Open items

1. The three `OnBeforeCreate` bodies of the screens with a PC class were read only in part; §6c records where the PC does the same work, and a statement-level diff was not made.
