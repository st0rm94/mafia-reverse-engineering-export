# `G_GarageClass` — the player's car garage and its three screens

**Created** 2026-08-25 22:28 +0200 · **Modified** 2026-09-17 13:44 +0200

| | |
|---|---|
| PC prefix | `garage_` for the manager, one `gui_garage_*_` per screen class |
| Manager (garage logic) | **`[0x0055d9e0, 0x0055ff70]`** |
| Screens | in the `game_menus` TU, **`[0x0057b930, 0x0057c880]`** |
| Related | `structs/game_menus.md` (the screen TU) · `structs/car_database.md` (car name/variant lookups every screen uses) · `structs/loadsave.md` (the garage's savegame block) |

This document is the join between the manager and its screens. Docblocks in the decomp cite
sections of this file as `SS<n>`; read that as `§<n>`.

## 1. What it is

The garage is where the player's owned cars are kept between missions. It has a hard capacity,
and when a mission ends with more cars than the cap the game forces the player to discard some —
that is what two of the three screens are for. The third is the post-mission report.

The manager is a singleton with a **stride-`0xc` car record** (`{u32 packed_car_id, C_car *live,
u32 flags}`) held in a `std::vector`, plus side vectors of car ids used as "ticked for discard" and
"already discarded" sets.

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
| `garage_generate_cars` | `0x0055fc90` | sets the one-shot request byte `+0x1cd` ¹ |
| `garage_release_cars` | `0x0055fca0` | sets the one-shot request byte `+0x1cc` ¹ |
| `garage_show_results` | `0x0055fcb0` | drives the design-`0x27` post-mission report |
| `garage_set_stolen_car` | `0x0055fea0` | mark a car as stolen |
| `garage_set_blocked_car` | `0x0055feb0` | mark a car as blocked |
| `garage_enable_steal_for_letter` | `0x0055fec0` | per-letter steal permission |
| `garage_can_steal_car_model` | `0x0055ff00` | steal permission query |
| `garage_is_car_parked` | `0x0055ff40` | membership query |
| `garage_enable_start_for_letter` | `0x0055ff70` | per-letter start permission |

¹ Neither spawns nor despawns anything; see §2a.

`garage_run_car_select_menu` also locates the garage AREA in the scene, by frame name
`oblast_garaze_a` / `oblast_garaze_b` with a `watercity.`-prefixed fallback — the two frames whose
bounding boxes give the parking volume.

### 2a. `+0x1cc` / `+0x1cd` — the end-of-mission car-banking request pair

Measured from the script-VM dispatcher band `0x0bc`..`0x13f`. Each function is a single byte
store:

| function | writes | reached from |
|---|---|---|
| `garage_generate_cars` (`GenerateCars__13G_GarageClass`) | `G_GarageClass+0x1cd = 1` | script command `garage_addlaststolen`, dispatcher case `0xc8` |
| `garage_release_cars` (`ReleaseCars__13G_GarageClass`) | `G_GarageClass+0x1cc = 1` | script command `garage_show`, dispatcher case `0xc9` |

Both are cleared by `garage_game_init`, and **`garage_game_done` tests `+0x1cc` OR `+0x1cd` in one
condition**, so the two commands arm the same behaviour: bank the car the garage is tracking at
`+0x1e8` into the garage list. That branch additionally requires the car NOT to be a police car
(`car+0x1ff2` clear) and `car_is_usable` to hold, appends the packed model id, and calls
`loadsave_race_cars_enable`. **Four model pack ids are excluded outright — `0x2b`, `0x39`, `0x3a`
and `0x3d`** — so those cars can never be banked however the mission ends.

The PC names follow the developers' own PS2 names (`GenerateCars`, `ReleaseCars`); those names
record an intent the bodies do not carry out.

Do not confuse these with the script commands `garage_generatecars` (case `0x5b`) and
`garage_releasecars` (case `0x5c`), which are a dead pair — both are one call to a shared
do-nothing stub and reach neither function above.

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

Both compose each row label as *name + variant* from `car_database_get_car_name` and
`car_database_get_car_variant_name_id`, and append textdb `0x45d` / `0x45e` for the colour count.
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

This loop is the watch-the-count method inlined (§4). It identifies the design-`0x26` screen as
`GM_GarageFull`: the shared car-name composition would fit either discard screen, but only one of
them is constructed inside this loop.

### 3b. The summary screen's two bands

Design `0x27` takes a **gate**: `1` = cars gained only, `2` = cars lost only, `4` = both (the `4`
variant adds a separator `line`). The constructor builds one `'mtxt'` row per gained car from base
id **1000** and one per lost car from base id **2000**, in lockstep — a shape no sibling screen
has, which is why its twin is CONFIRMED rather than positional. Its tick dismisses on **any** key
or on the shipped *Close* button (id 100).

## 4. PC vs PS2

The PS2 garage is **`[0x0048a5c0, 0x004904b0]`** — `G_GarageClass` plus five `GM_Garage*` screen
classes. That band lies outside the PS2's GUI band (`0x002c…0x002e`); an enumeration of the symbol
table by VA range, ignoring the class column, finds all of them.

**The over-cap loop is an inline.** MSVC inlined the PS2 method `HlidejPocetVect__13G_GarageClassF…`
(Czech *"hlídej počet"* = *watch the count*) into `garage_game_done`; on the PS2 `GameDone` calls it
out of line. The site carries a `// ===== PS2-INLINED HERE: … =====` banner in the annotated
decomp.

### 4a. Twins

| PC function | PC VA | PS2 method | grade | fold |
|---|---|---|---|---|
| `garage_game_done` | `0x0055e6e0` | `GameDone__13G_GarageClassF…` | CONFIRMED | absorbs `HlidejPocetVect` |
| `garage_run_car_select_menu` | `0x0055f510` | `CarManager__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `garage_show_results` | `0x0055fcb0` | `GarageShowResultsProc__13G_GarageClassFv` | CONFIRMED | 1 → 1 |
| `gui_garage_select_ctor` | `0x0057b930` | `OnCreate__15GM_GarageSelectFv` | CONFIRMED | 3 → 1 ¹ |
| `gui_garage_select_tick` | `0x0057bd10` | `OnClick__15GM_GarageSelectFUi` | CONFIRMED | 2 → 1 (+`OnKey`) |
| `gui_garage_full_ctor` | `0x0057bfa0` | `OnCreate__13GM_GarageFullFv` | CONFIRMED | 3 → 1 ¹ |
| `gui_garage_full_tick` | `0x0057c340` | `OnClick__13GM_GarageFullFUi` | STRUCTURAL | 2 → 1 (+`OnKey`) |
| `gui_garage_summary_ctor` | `0x0057c540` | `OnCreate__16GM_GarageSummaryFv` | CONFIRMED | 1 → 1 |
| `gui_garage_summary_tick` | `0x0057c880` | `OnKey__16GM_GarageSummaryFUcUi` | CONFIRMED | 2 → 1 ² |
| - | - | `GM_GarageGarage` (4 methods) | not located | - |
| - | - | `GM_GarageComplete` (2 methods) | not located | - |

¹ Absorbs `OnTick` and `OnClose`'s `UnloadDetails`. ² Absorbs `OnClick`, an 8-byte stub.

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

## 5. Open items

| item | state |
|---|---|
| PC counterparts of `GM_GarageGarage` / `GM_GarageComplete` | not searched; on the PS2 they are the garage root menu and its completion notice, opened by `G_GarageClass::DoGarageMenu` / `::DoFreerideMenu`, whose PC equivalents are also unread |
| car record `+0x08` | read as a flag by both discard screens; no writer traced |
| `garage_enable_steal_for_letter` / `garage_enable_start_for_letter` | the letter index's meaning (a mission-letter gate?) is undocumented |
