# The racing front-end screens — `g_m_race.cpp`

**Created** 2026-08-24 11:05 +0200 · **Modified** 2026-10-01 08:56 +0200

The single-player racing menus: **14 `GM_Menu` subclasses** in one translation unit,
`[0x00566280, 0x0056adc0)`, **40 starts, 0 unnamed**.

| | |
|---|---|
| Prefix | `gui_race_<screen>_<method>` — one second token per class, extending the `gui_race_champ_select_*` pair of the multiplayer menus |
| Base class | `GM_Menu` — read `structs/gui_screen.md` first; the 6-slot vtable map, the widget field map and the accessor vocabulary live there |
| Related | `basic/artifacts/racing.md` (the feature, and why these menus are unreachable in retail while mission 6 still runs a race), `structs/racing.md` (`C_racing` and its five owned classes) |

## 1. The class table

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

Plus one shared thunk, `thunk_gui_menu_close` at `0x00569500`, which is
`gui_race_car_color_`'s slot 4. 39 + 1 = 40.

## 2. How the class map is established

The constructors say almost nothing on their own: chain the base, install a vtable, push an
integer, call the builder. **That integer is the answer.** Decoding the 14 `menu.def` groups
and resolving every `text_id` through `textdb.json` names all 14 screens without reading a
body; each is independently CONFIRMED against the other build (§7).

⚠️ **Read the fr/cz columns of `textdb.json`, not en.** Retail English overwrote ids
410-412 with debug strings ("Controls unlocked", "Thrower spawned") when racing was cut;
fr/cz still say *Course simple* / *Championnat*.

**Entry points.** `gui_race_root_open_single_race` and `gui_race_root_open_championship`
build a whole stack in one call — root → setup → launch, or root → list → rounds → launch —
and set `g_menu_camera_item_id` to `0x18453` / `0x18454` so the `00menu` camera flies to the
right frame.

## 3. The garage chain

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

The colour screen lists one colour name per colour index
(`car_database_get_color_name_id`).

`gui_race_opponents_tick` reaches the same manufacturer screen for each AI slot.

## 4. The championship / single-race pair

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

Both results screens run `OnTick` + `OnKey` + `OnClick` in slot 5: a 500 ms standings
refresh, a **15 000 ms auto-continue** countdown printed into widget `0x78`, and `F2` to
toggle the overlay through `racing_toggle_results_visibility`.

`gui_race_champ_launch_tick`'s `0x1c8` arm toggles `racing_set_damage_enabled` inline and swaps
the label between text `0x1ca` and `0x1cb`; `gui_race_setup_tick` has no `0x1dc` arm.

## 5. `GM_BonusCar`

Design `0x23`, the award screen. Its constructor loads the car-database details, formats
the awarded car's name and variant into one string (textdb `0x45d`/`0x45e` supply the
variant wording) and writes it into widget `10` between a `car_database_load_details` /
`car_database_unload_details` pair. Its slot 5 closes the whole stack with `0x34`, or opens
the Carcyclopedia on `0x1f6`.

## 6. `C_racing` / `C_championship` fields this TU reads

| field | meaning | PS2 |
|---|---|---|
| `g_racing+0x08` | the circuit vector handed to the circuit picker | - |
| `g_racing+0x1c` / `+0x20` | the championship vector, begin / end | `CRacing+0x604` / `+0x600` (ptr + count) |
| `g_racing+0x28` | **the racing mode: 1 = championship, 2 = single race** | `CRacing+0xaa8` |
| `g_racing+0x34` | the best-times owner used by the launch screens | - |
| `g_racing+0x38` | the current championship | `CRacing+0xa5c` |
| `g_racing+0x3c` | the opponent difficulty, 0..4; label text `0x1ef + level` | - |
| `g_racing+0x64` | the player's chosen car (packed id) | `CRacing+0x580` |
| `g_racing+0x27c` | lap count, clamped 1..99 | `CRacing+0xab8` |
| `g_racing+0x28c` | the opponent count used by the results tables | - |
| `g_racing+0x2b4` | track-mode flag (the "Track Mode: Normal" toggle) | `CRacing+0xb04` |
| `g_racing+0x2b7` | damage-enabled flag; text `0x1ca` on, `0x1cb` off | - |
| `C_championship+0xac` | the round vector | `+0x44` |
| `C_championship+0xcc` | the championship's state: 0 open · 1 shown-not-set · 2+ locked | `+0x30` |
| `C_championship+0xd4` / `+0xd8` | opponent count / championship car mask | - |
| `C_circuit+0x50` | the circuit's state; `!= 2` means selectable | `+0x00` |

Two data symbols this TU touches: `g_gui_race_champ_results_refresh_countdown_ms` (the 500 ms results-refresh
countdown, 3 refs) and `g_gui_race_champ_launch_circuit_bmp_name` (the `circuit%d.bmp` scratch buffer, 3 refs; a shared
buffer, §8).

## 7. PC vs PS2

**94 PS2 methods across 16 classes map onto 39 PC functions**, and the whole difference is
accounted for:

* the PC **fuses** `{ctor}` + `OnBeforeCreate` + `OnCreate` into one constructor — that is
  14 × 2 = 28 PS2 methods absorbed;
* the PC **folds** `OnTick`, `OnKey`, `OnChange` and `OnCloseChild` into slot 5 wherever the
  behaviour is small: 2→1 on the circuit picker, the car-model picker, the colour picker,
  the opponent screen and the bonus-car screen; 3→1 on both results screens;
* 🔴 **two PS2 classes have no PC counterpart at all**, and both absences are measured:
  * **`GM_DamageDifficulty`** — the PS2 `GM_SingleRace::OnClick` case `0x1dc` and
    `GM_ChampionshipRace::OnClick` case `0x1c8` each `operator_new(0x30)` this screen. The
    PC has neither (§4). The difficulty half of that PS2 screen (`CRacing+0xab0`) has no PC
    equivalent either.
  * **`GM_GameMenuRacing`** — the in-race pause menu. No vtable in the PC run, no design id
    in `menu.def`, and no caller. The PC reaches the generic in-game menu instead.

**PS2 methods with no PC twin, individually:** the 28 fused `{ctor}`/`OnBeforeCreate` pairs,
the 12 folded `OnTick`/`OnKey`/`OnChange` bodies, `GM_DamageDifficulty` × 5 and
`GM_GameMenuRacing` × 7. That accounts for all 94.

**Emission order differs between the builds.** The PC follows the `menu.def` group order
(`GM_Racing`, `GM_Championship`, `GM_SelectInRound`, `GM_ChampionshipRace`,
`GM_ChampionshipResults`, `GM_SingleRace`, …); the PS2 emits `GM_Racing`, `GM_SingleRace`,
`GM_SelectCircuit`, `GM_SelectCarManufacturer`, `GM_SelectCarModel`, … An order-align over
this TU gives wrong ranks for eleven of the fourteen classes; the design-id join (§2) does not
depend on order.

**Per-screen comparison:**

* **Class map** — each of the 14 design-id screens (§2) is independently CONFIRMED by an
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
* **Opponent grid** — `gui_race_opponents_ctor` clones a **fixed** 19 rows and walks 20, and
  `gui_race_opponents_tick` passes a **hardcoded `0x14`** to `racing_add_opponents`. The PS2
  derives both from the circuit's own `+0x08`. So a PC single-race grid should always be 20
  cars regardless of circuit.
* **Colour-picker gate** — `gui_race_car_model_tick` opens the colour picker when
  `car_database_get_max_color_index >= 2`; the PS2 opens it when `GetCarMaxColor != 0`.

## 8. Open items

| item | state |
|---|---|
| root menu and garage chain | in game (needs the racing mod — retail deletes the menu item): Main Menu → Racing should show the four-item root menu; Single Race → GARAGE should walk the three-screen car chain |
| single-race grid size | in game: should hold 20 opponents regardless of circuit (§7) |
| damage toggle | in game: the toggle on the championship launch screen should change its own label rather than open a sub-screen — the visible form of the missing `GM_DamageDifficulty` |
| `g_gui_race_champ_launch_circuit_bmp_name` | a shared buffer; its writers outside this TU are not censused |
| `g_racing+0x34` / `+0x28c` | `structs/racing.md` lists `+0x34` as the current `C_champ_circuit`, while this TU uses it as the launch screens' best-times owner; `+0x28c` is not in `structs/racing.md` |
