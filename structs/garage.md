# `G_GarageClass` — the player's car garage and its three screens

**Created** 2026-08-25 22:28 +0200 · **Modified** 2026-10-11 21:38 +0200

| | |
|---|---|
| PC prefix | `garage_` for the manager, one `gui_garage_*_` per screen class |
| Manager (garage logic) | **`[0x0055d9e0, 0x0055ff70]`** |
| Screens | in the `game_menus` TU, **`[0x0057b930, 0x0057c880]`** |
| Related | `basic/artifacts/garage/storing-cars-in-the-garage.md` (how a car gets stored: the garage area, the four banking paths, the PS2 difference) · `structs/game_menus.md` (the screen TU) · `structs/car_database.md` (car name and colour-name lookups every screen uses) · `structs/loadsave.md` (the garage's savegame block) |

This document is the join between the manager and its screens. Docblocks in the decomp cite
sections of this file as `SS<n>`; read that as `§<n>`.

## 1. What it is

The garage is where the player's owned cars are kept between missions. It has a hard capacity,
and when a mission ends with more cars than the cap the game forces the player to discard some —
that is what two of the three screens are for. The third is the post-mission report.

The manager keeps a **25-slot parked-car table** at `+0xa0` (`{u32 packed_car_id, C_car *live}`,
stride 8). `garage_game_done` and `garage_run_car_select_menu` build a working `std::vector` of
**stride-`0xc` records** (`{u32 packed_car_id, C_car *live, int parked_slot}`) from that table and
the cars standing in the garage area, plus two side vectors of car ids: the cars ticked for discard
and the cars discarded. `parked_slot` is the car's index in the table (0..24), or `-1` for a car
found in the garage area or banked this mission. `garage_destroy_car` clears the slot it names, and
the tail of `garage_game_done` frees the slots of discarded cars and gives each `-1` car the first
free slot. The discard screens never read it.

**The capacity is 16 on the PC**, carried as the literal `0x10` in `garage_game_done`'s over-cap
loop (§3a). It is not read from a table.

## 2. The manager — `garage_*`, `[0x0055d9e0, 0x0055ff70]`

| function | VA | role |
|---|---|---|
| `garage_clear` | `0x0055d9e0` | empty the garage |
| `garage_add_car_by_index` | `0x0055da10` | park a car by packed id |
| `garage_add_car_by_actor` | `0x0055da90` | park a live `C_car` |
| `garage_remove_car_by_model` | `0x0055db40` | remove by model name |
| `garage_game_init` | `0x0055db90` | per-scene setup |
| `garage_game_done` | `0x0055e6e0` | per-scene teardown; **hosts the over-cap discard loop** (§3a) |
| `garage_run_car_select_menu` | `0x0055f510` | the in-garage car manager; drives the design-`0x25` screen |
| `garage_destroy_car` | `0x0055fa90` | destroy one parked car |
| `garage_request_add_last_stolen` | `0x0055fc90` | sets the one-shot request byte `+0x1cd` ¹ |
| `garage_request_show_results` | `0x0055fca0` | sets the one-shot request byte `+0x1cc` ¹ |
| `garage_show_results` | `0x0055fcb0` | drives the design-`0x27` post-mission report |
| `garage_set_last_entered_car` | `0x0055fea0` | remember the last car the player got into (`+0x1e8`) |
| `garage_set_blocked_car` | `0x0055feb0` | mark a car as blocked |
| `garage_enable_steal_for_letter` | `0x0055fec0` | per-letter steal permission |
| `garage_can_steal_car_model` | `0x0055ff00` | steal permission query |
| `garage_is_car_parked` | `0x0055ff40` | membership query |
| `loadsave_freeride_enable_car_group` | `0x0055ff70` | unlocks one Free Ride car group (§2b); a `G_LoadSaveClass` method |

¹ Neither spawns nor despawns anything; see §2a.

`garage_game_done` and `garage_run_car_select_menu` both locate the garage AREA in the scene, by
frame name `oblast_garaze_a` / `oblast_garaze_b` with a `watercity.`-prefixed fallback: the area is
the X/Z rectangle spanned by the two frames' world origins, height ignored
(`basic/artifacts/garage/storing-cars-in-the-garage.md`).

### 2a. `+0x1cc` / `+0x1cd` — the end-of-mission car-banking request pair

Measured from the script-VM dispatcher band `0x0bc`..`0x13f`. Each function is a single byte
store:

| function | writes | reached from |
|---|---|---|
| `garage_request_add_last_stolen` | `G_GarageClass+0x1cd = 1` | script command `garage_addlaststolen`, dispatcher case `0xc8` |
| `garage_request_show_results` | `G_GarageClass+0x1cc = 1` | script command `garage_show`, dispatcher case `0xc9` |

Both are cleared at the end of `garage_game_done` and by `garage_game_init`, and
**`garage_game_done` tests `+0x1cc` OR `+0x1cd` in one condition**, so the two commands arm the same
behaviour: bank the last car the player got into (`+0x1e8`, `garage_set_last_entered_car`) wherever
it stands. That path requires the car NOT to be a police model (`car+0x1ff2` clear) and
`car_is_usable` to hold, refuses four car-database ids — **`0x2b` (`bugatti`) and `0x39`, `0x3a`,
`0x3d` (the three Bolt trucks)** — appends the packed id and calls `loadsave_race_cars_enable`. The
area scan that banks cars standing in the garage area refuses only the three trucks.
`garage_show` (`+0x1cc`) additionally opens the results screen. The four banking paths are
written up in `basic/artifacts/garage/storing-cars-in-the-garage.md`.

Do not confuse these with the script commands `garage_generatecars` (case `0x5b`) and
`garage_releasecars` (case `0x5c`), which are a dead pair — both are one call to the shared
do-nothing stub `stub_noop_cdecl_00574a40` and reach neither function above.

### 2b. The two car-group letter masks

`garage_enable_steal_for_letter` and `loadsave_freeride_enable_car_group` decode a letter the same
way — `A`..`Z` or `a`..`z` to bit `letter - 'A'`, any other byte to bit 0 — and OR the bit into one
mask of the singleton:

| mask | setter | script command | tested against |
|---|---|---|---|
| `+0x98` steal | `garage_enable_steal_for_letter` | `garage_enablesteal` | `carindex.def +0x8c`, by `garage_can_steal_car_model` (`structs/car_database.md` §3) |
| `+0x9c` Free Ride | `loadsave_freeride_enable_car_group` | `freeride_enablecar` (case `0x176`) | `carindex.def +0xa0` (`car_database_is_enabled_for_freeride`) or the Carcyclopedia mask |

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

## 3. The three screens

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
  and returns `0x4f9`, the code that makes the caller rebuild and reopen it; already-ticked rows are
  prefixed with textdb `0x45f` *"(removed) "*. It has a **Close** row (id 5000).
* design `0x26` is a **single pick**. Its tick appends `(id - 1000)` to the discarded set and
  returns `0x4f8`. There is no Close — the caller reopens it until the garage is under its cap.

Both compose each row label as *name + colour name* from `car_database_get_car_name` and
`car_database_get_color_name_id`, and append textdb `0x45d` / `0x45e` for the colour count.
Both bracket the build with `car_database_load_details` / `..._unload_details`.

### 3a. The over-cap loop

`garage_game_done` contains this loop:

```
while (surviving_cars > (discarded - flag) + 0x10) {
    operator_new(0x50);
    gui_garage_full_ctor(0, 0, cars, discarded, third);
    gui_menu_execute_modal(1);
}
```

This loop is the watch-the-count method inlined (§4). Only one of the two discard screens is
constructed inside it.

### 3b. The summary screen's two bands

Design `0x27` takes a **gate**: `1` = cars gained only, `2` = cars lost only, `4` = both (the `4`
variant adds a separator `line`). The constructor builds one `'mtxt'` row per gained car from base
id **1000** and one per lost car from base id **2000**, in lockstep — a shape no sibling screen
has (§4a). Its tick dismisses on **any** key
or on the shipped *Close* button (id 100).

## 4. PC vs PS2

The PS2 garage is **`[0x0048a5c0, 0x004904b0]`** — `G_GarageClass` plus five `GM_Garage*` screen
classes. That band lies outside the PS2's GUI band (`0x002c…0x002e`); an enumeration of the symbol
table by VA range, ignoring the class column, finds all of them.

**The over-cap loop is an inline.** MSVC inlined the PS2 method `HlidejPocetVect__13G_GarageClassF…`
(Czech *"hlídej počet"* = *watch the count*) into `garage_game_done`; on the PS2 `GameDone` calls it
out of line. The site carries a `// ===== PS2-INLINED HERE: … =====` banner in the annotated
decomp. The loop also identifies the design-`0x26` screen as `GM_GarageFull`: the shared car-name
composition would fit either discard screen, but only one of them is constructed inside this loop.

**The request pair.** `garage_request_add_last_stolen` and `garage_request_show_results` are
`AddLastStolen` and `ShowResults`: the PS2 bodies store 1 to `+0x7d` / `+0x7c`, both builds call
them from the same two script arms, and both `GameDone` bodies test the pair in one condition and
open the results screen on the second byte alone. The PS2 `GenerateCars` / `ReleaseCars` are empty
8-byte bodies, and the PC arms that call them (§2a) reach only the shared empty stub.

### 4a. Twins

| PC function | PC VA | PS2 method | grade | fold |
|---|---|---|---|---|
| `garage_game_done` | `0x0055e6e0` | `GameDone__13G_GarageClassF…` | CONFIRMED | absorbs `HlidejPocetVect` |
| `garage_run_car_select_menu` | `0x0055f510` | `CarManager__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `garage_request_add_last_stolen` | `0x0055fc90` | `AddLastStolen__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `garage_request_show_results` | `0x0055fca0` | `ShowResults__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `garage_show_results` | `0x0055fcb0` | `GarageShowResultsProc__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `loadsave_freeride_enable_car_group` | `0x0055ff70` | `FreerideEnableCar__15G_LoadSaveClassFUc` | CONFIRMED | 1 → 1 ³ |
| `gui_garage_select_ctor` | `0x0057b930` | `OnCreate__15GM_GarageSelectFv` | CONFIRMED | 3 → 1 ¹ |
| `gui_garage_select_tick` | `0x0057bd10` | `OnClick__15GM_GarageSelectFUi` | CONFIRMED | 2 → 1 (+`OnKey`) |
| `gui_garage_full_ctor` | `0x0057bfa0` | `OnCreate__13GM_GarageFullFv` | CONFIRMED | 3 → 1 ¹ |
| `gui_garage_full_tick` | `0x0057c340` | `OnClick__13GM_GarageFullFUi` | STRUCTURAL | 2 → 1 (+`OnKey`) |
| `gui_garage_summary_ctor` | `0x0057c540` | `OnCreate__16GM_GarageSummaryFv` | CONFIRMED | 1 → 1 |
| `gui_garage_summary_tick` | `0x0057c880` | `OnKey__16GM_GarageSummaryFUcUi` | CONFIRMED | 2 → 1 ² |

¹ Absorbs `OnTick` and `OnClose`'s `UnloadDetails`. ² Absorbs `OnClick`, an 8-byte stub.
³ The same letter decode into the same mask: PC `+0x9c` and PS2 `+0x2b8` are joined by the
CONFIRMED accessor pair `loadsave_freeride_get_enabled_cars` / `FreerideGetEnabledCars`. The PS2
body also maps the digits `0`..`6` to bits 26..32, remaps `U` to `0`, and ORs
`G_CarDatabase::EnableCarsForFree` into `+0x2b4`.

`gui_garage_summary_ctor` is CONFIRMED rather than positional because its lockstep gained/lost row
build (§3b) is a shape no sibling screen has. `GM_GarageGarage` and `GM_GarageComplete`: §4c.

### 4b. The presentation differs, the roles do not

The PS2 screens are **console carousels**: `GM_GarageSelect::OnCreate` preloads one `ITexture` per
car through a `sprintf`-built name, `OnTick` keeps a ±10 window of those textures live and releases
the rest, and `OnClose` frees them. The PC has no previews at all — it builds a text list once in
the constructor, which is why the PC constructors absorb the PS2's `OnTick`.

The PS2 also shows a *"n/N"* counter (component 299) that the PC design does not ship, and its
`OnTick` calls `G_IndicatorsClass::DrawMessage` while textures stream in.

### 4c. PS2-only

`GM_GarageSelect::OnTick`'s texture streaming, the `+0x40` counter string, and both `OnClose`
overrides are console-side memory management with no PC equivalent — the PC keeps the whole car
database detail block loaded for the life of the screen instead.

**`GM_GarageGarage` and `GM_GarageComplete` are PS2-only.** `GM_GarageGarage` is a root menu —
change car (opens `GM_GarageSelect`), park car (`ParkCar`), back — and `GM_GarageComplete` the
end-of-mission notice that leads into it. Only `G_GarageClass::DoGarageMenu` builds them, and
`GM_GarageComplete` opens `GM_GarageGarage`. `DoGarageMenu`'s one caller is the PS2 `C_game::Tick`, gated on a byte that only the
PS2-only script command `GARAGE_VRATA` sets; `DoFreerideMenu`, reached only from the PS2-only
`GARAGE_FREERIDE`, opens `GM_GarageSelect`. Both commands are absent from the PC's command table
(`basic/artifacts/script-opcode-dispatch.md` §7).

## 5. Open items

Nothing is open.
