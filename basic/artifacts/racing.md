# Racing mode — the PC feature that was cut at the last minute

**Created** 2026-08-16 07:38 +0200 · **Modified** 2026-09-30 23:08 +0200

Class layouts: `structs/racing.md`, `structs/racecam.md`, `structs/race_session.md`,
`structs/racing_menus.md`.

## 1. The headline

**The PC racing mode is not missing. It is in the retail binary, complete, and only the
menu entry was taken out.**

The PC release has no Racing item on its main menu, and the received story is that it was
"nearly finished and cut". What that actually means is unusually specific:

* **The whole engine ships.** One translation unit, `[0x0060b770, 0x00617a60)`, six classes
  (`C_racing`, `C_circuit`, `C_championship`, `C_champ_circuit`, `C_opponent`,
  `C_best_time`) plus the race-camera entity class `C_racecam` in its own TU. None of it is
  stubbed.
* **The data ships.** `tables/circuits.def` holds **11 circuits** and
  `tables/championship.def` holds **7 rounds**, both in the retail install.
* **The localized menu label ships.** `textdb` id `0x6a` is
  **`Racing` / `Course` / `Rennen` / `Závody`** in all four shipped languages.
* **What is missing is the ten track scenes and one menu item** — and the executable
  goes out of its way to delete that item at runtime.
* **The engine also runs in the retail story.** Mission 6 ("Fair Play") builds `g_racing`
  through the script command `racing_mission6_init` and races on circuit 0, the
  `mise06-autodrom` scene (§8).

## 2. What retail ships vs what it lacks

| shipped in retail PC | missing from retail PC |
|---|---|
| the whole racing engine (§6) | the 10 track scenes (`missions/trat01`…`trat10`) — retail has 81 mission folders, **zero** `trat*`; `MISE06-AUTODROM` **is** there |
| `tables/circuits.def` — 11 circuits (§5a) | the track preview maps (`circuit%d.bmp`) |
| `tables/championship.def` — 7 rounds (§5b) | the `Racing` menu item — absent from `menu.def` **and** deleted by the exe (§3) |
| every racing `textdb` string (`Racing`, `Racing:`, `DNF`, `ROUND n`, the class names, the lap/position/best-time labels) | - |
| the race HUD widgets (`hud_set_race_*`, 7 CONFIRMED twins in `structs/hud.md` §4) | - |
| `racestart.bmp` / `racewrong.bmp` (loaded by `hud_create`), `records\recordrace.rep`, `difrace.chg` | - |
| **`FormuleVolant.I3D`** — the open-wheel steering-wheel driver rig, selected by `game_add_driver_man`'s trailing bool against the ordinary `AutoRidicVolant.I3D`. Racing-only content, in the retail exe's string table | - |
| the script commands `RACING_CHANGE_MODEL`, `RACING_AUTOINVISIBLE`, `RACING_MISSION6_START`, `RACING_MISSION6_INIT` — the last two used by mission 6 (§8) | - |

## 3. How the cut was done — two places, belt and braces

### 3a. The menu item was removed from the data

Retail `tables/menu.def` describes the main-menu window as a run of `meti` records with
consecutive ids. Two ids are simply not there:

| id | text id | label |
|---|---|---|
| `0xf1b98` | 103 | Load Game |
| `0xf1b99` | 107 | Free Ride |
| `0xf1b9a` | 113 | Free Ride Extreme |
| **`0xf1b9b`** | - | **absent — this was `Racing` (textdb `0x6a`)** |
| **`0xf1b9c`** | - | **absent — this was `Multiplayer` (textdb 104)** ¹ |
| `0xf1b9d` | 110 | Carcyclopedia |
| `0xf1b9e` | 108 | Options |
| `0xf1b9f` | 114 | Credits |
| `0xf1ba0` | 109 | Change Profile |

¹ Three independent witnesses identify `0xf1b9c`: `gui_main_menu_tick`, the main-menu click
handler, maps it to menu choice **`0x14`**, and `engine_main` case `0x14` calls
`net_multiplayer_run`; the menu-camera table `g_menu_camera_position_table`
(`0x0064daf0`) pairs it with the frame **`multiplayer_pos`** in the `00menu` scene; and
**textdb 104 = "Multiplayer" / "Multijoueur" / "Mehrspieler" / "Multiplayer"** sits unused
between 103 "Load Game" and 105 "Tutorial". Retail blocks Multiplayer exactly like Racing —
no `meti` record plus the runtime disable-and-delete of §3b — although the whole front end
ships: `structs/multiplayer.md`.

### 3b. The executable deletes them anyway

`gui_main_menu_ctor` (`0x00576920`) — the main-menu screen's constructor — contains, in the
retail build:

```c
gui_menu_set_component_enabled(0xf1b9b, 0);   // clear the widget's enable bit
gui_menu_set_component_enabled(0xf1b9c, 0);
gui_menu_delete_component(0xf1b9b);           // find the widget by id, destroy it, erase it
gui_menu_delete_component(0xf1b9c);
```

Both ids are already absent from the shipped `menu.def`, so these four calls are dead code
in retail. They are a **defensive second cut**: whoever pulled the feature removed the
data *and* left instructions to remove the widget if it ever turned up again. That is what
makes the mod's job so small.

## 4. The mod, byte for byte

`exe-re/Game-racing-mod.exe` (the "RACING MENU for version 1.0" build shipped with the
community racing packs) and `exe-re/Game.exe` are **the same 2,486,272 bytes**, differing
in **511 bytes across 19 runs**. Most of that is cracker banners: every cracked build
carries one, and the two sit in *disjoint* file ranges — a `GiMP`-tagged banner at
`0x320..0x4d0` in `Game.exe`, FairLight's at `0x2f0..0x3af` in the mod — which is why a
two-way diff makes each file look clean exactly where the other is dirty.

The mapped-code differences are **13 bytes at 7 addresses**:

| VA | retail | mod | what it is |
|---|---|---|---|
| `0x0057699c` | `push 0xf1b9b` | `push 0xf1ccd` | `gui_main_menu_ctor` — the **disable** call is redirected |
| `0x005769b6` | `push 0xf1b9b` | `push 0xf1ccd` | `gui_main_menu_ctor` — the **delete** call is redirected |
| `0x0062407e` | `jne 0x6240a5` | `jmp 0x6240a5` | CD check — a crack, not racing |
| `0x006240e0` | `xor eax,eax; inc eax; ret` | `sub esp,0x18c` | CD check — a crack, not racing |
| `0x00629aaa` | `[0x63b100]` | `[0x63b06c]` | `LCMapStringW` vs `CompareStringW` — an import-rebuild error in the mod; `Game.exe` has the correct one ([`crack.md`](crack.md) §7a) |
| `0x0040bc9d` | `a1` | `0b` | 1 byte in dead inter-function padding, 0 references |
| `0x0040c5bd` | `a0` | `0a` | 1 byte in dead inter-function padding, 0 references |

So the mod:

1. **redirects the two `0xf1b9b` calls onto `0xf1ccd`**, so the racing entry it adds is
   neither disabled nor deleted. `0xf1ccd` is the RACING menu's own third item, labelled
   textdb 413 = "Multiplayer" (design `0x16`, which ships in retail `menu.def` as
   *Racing: / Single Race / Championship / Multiplayer / Exit*). The redirect is safe:
   `gui_menu_set_component_enabled` and `gui_menu_delete_component` only walk the CURRENT
   screen's own child vector, and `gui_main_menu_ctor` is building the main menu, so both
   redirected calls are no-ops there. It does mean the racing mod leaves the Racing menu's
   Multiplayer entry **untouched and clickable**, which is the shortest route to the
   multiplayer front end in a modded install (`structs/multiplayer.md` §1).
2. ships a `menu.def` that is 36 bytes larger — **exactly one extra `meti` record**, id
   `0xf1b9b`, text id `0x6a` = "Racing", inserted into the main-menu window.

It leaves `0xf1b9c` alone, so the Multiplayer main-menu item stays deleted in the modded build.

That is the entire modification. Everything else the racing packs ship is content: the
track scenes, models, preview maps and larger `circuits.def` / `championship.def` tables.

### 4a. Both PC binaries are cracked, in different ways

A trap for any byte-diff:

* `exe-re/Game.exe` (retail-derived) has the CD drive scan at `0x006240e0` **stubbed to
  `return 1`** — `xor eax,eax; inc eax; ret`, with the real function body still sitting
  untouched from `0x006240e6` onward.
* `exe-re/Game-racing-mod.exe` keeps the real drive-scan function (`sub esp,0x18c`, then
  `mov ebx,0x43` = `'C'` and a drive walk) and instead forces `jne` → `jmp` at
  `0x0062407e`.

Neither has anything to do with racing. A diff of the two exes will always show them.

This is the shallow end of a much larger story that is not this document's job. Both exes
sit on a **SafeDisc 2.70.030** wrapper; both are decrypted memory dumps with rebuilt import
tables; both carry cracker banners; and the genuine retail executable is in the repo as
`exe-re/Game-uncracked.exe`. **Read [`crack.md`](crack.md) before making any byte-level
claim about either binary.** The `0x006240e0` stub is repaired in the decomp as one merged
definition, `cd_scan_drives_for_disc`; that deliberate 4-byte divergence is listed in
`crack.md` §5c among the known divergences an image byte-diff is expected to show.

The two padding bytes `0x0040bc9d` and `0x0040c5bd` each sit in a 5-byte run of
inter-function padding after a `ret`, are not 16-aligned function starts, and have **zero
absolute references anywhere in the image**. They are not reachable code. Both change by
exactly `-0x96`, and what they are is open (§9); the byte-level context of both runs is in
[`crack.md`](crack.md) §5f.

## 5. The data files, decoded

### 5a. `tables/circuits.def` — 0x88 bytes per circuit

```c
struct S_circuit_def {          /* 0x88 */
    char     display_name[0x20];  /* +0x00  "Central Island"          */
    char     mission_name[0x20];  /* +0x20  "trat01"                  */
    uint32_t laps;                /* +0x40  7 (or 5)                  */
    uint32_t reserved;            /* +0x44  0 in every shipped record */
    uint32_t class_time_min[7];   /* +0x48  ms, per car class         */
    uint32_t class_time_max[7];   /* +0x68  ms, per car class         */
};
```

The retail file is 1496 B = **11 records exactly**. The two 7-entry tables are the
per-car-class reference times: `racing_generate_qualification` picks each AI opponent's
qualifying time at random between the min and the max for that opponent's class, so they
set the pace of the field. The seven slots line up one-for-one with the seven championship
rounds' car-class masks (§5b) — the values fall as the class gets faster.

Retail circuit roster:

| # | display name | scene | laps |
|---|---|---|---|
| 0 | Racing Circuit Long | `mise06-autodrom` | 7 |
| 1 | Central Island | `trat01` | 7 |
| 2 | Countryside 1 | `trat02` | 7 |
| 3 | Little Italy | `trat03` | 7 |
| 4 | Countryside 2 | `trat04` | 7 |
| 5 | Oakwood Rotary | `trat05` | 7 |
| 6 | Hoboken | `trat06` | 7 |
| 7 | Oakwood | `trat07` | 7 |
| 8 | Racing circuit Short | `trat08` | 7 |
| 9 | New Ark | `trat09` | 7 |
| 10 | Down Town | `trat10` | 7 |

Only record 0's scene exists in the retail install.

### 5b. `tables/championship.def` — 0xb8 bytes per round

The retail file is 1288 B = **7 records exactly**, which is what pins the PC stride
independently of the code. Decoded head:

| # | name id | label | circuits | car-class mask | field @+0x14 |
|---|---|---|---|---|---|
| 0 | 430 | Class A | 7 | 1 | 0 |
| 1 | 431 | Class B | 7 | 2 | 45 |
| 2 | 432 | Class C | 7 | 4 | 59 |
| 3 | 433 | Classic Designs | 7 | 8 | 43 |
| 4 | 434 | Sports Cars | 7 | 16 | 41 |
| 5 | 435 | Racing Cars | 7 | 32 | 46 |
| 6 | 446 | ROUND 7 | 7 | 64 | 60 |

Two 10-entry opponent id arrays follow at `+0x68` and `+0x90`.

### 5c. `tables/carcyclopedia.def` — 200 bytes per car

Two different `G_CarDatabase` tables must not be conflated. This 200-byte record is the
DETAIL table. The car LIST that `car_database_get_car_model` and
`car_database_get_car_params` index is a separate array whose stride is **0xa4**, and the
player/driver-model table `car_database_get_player_model` indexes has 0x24-byte entries.

The detail table is loaded ref-counted by `car_database_load_details` and released by
`car_database_unload_details`. `racing_generate_qualification` brackets its whole loop with
the pair, because it needs each car's class bitmask (`+0x88` of the entry).

## 6. Where the code lives

| what | where |
|---|---|
| `C_racecam` (entity kind `0x1c`) | `[0x004d33d0, 0x004d35c0)` — 9 starts, all named |
| the six racing classes + the free comparators | `[0x0060b770, 0x00617a60)` — 131 starts, all named (`structs/racing.md`) |
| the racing segment subsystem | `[0x005c9750, 0x005ca150)` — 8 starts, all named: `racing_optimal_segment_ctor`, `racing_record_segment_points`, `racing_create_segments`, `racing_save_segments`, `racing_load_segments`, `racing_clear_segments`, `racing_get_segment_bank_for_car`, `racing_finish_segment_and_advance` |
| the racing MENU screens | `[0x00566280, 0x0056adc0)` = the `g_m_race.cpp` translation unit — 14 `GM_Menu` subclasses, designs `0x16`..`0x23`, 40 starts, all named (`structs/racing_menus.md`). The root menu is `gui_race_root_ctor` (vtable `0x0063d434`, design `0x16`) with the tick `gui_race_root_tick`, driven by `race_menu_run`; its item `0xf1ccd` is where `net_multiplayer_run` is wired |
| the race replay / difficulty profile | `race_winners_replay_begin` (`0x005947e0`) — owns `difrace.chg` and `records\recordrace.rep` |
| the race HUD widgets | `structs/hud.md` §4 — 7 CONFIRMED twins |
| multiplayer racing | 12 `C_racing` functions plus a session object with a human-player list and an AI-opponent list, and seven kinds of event message (`structs/net_session.md`) |

The PC emits `C_racecam.cpp` right after `C_pumper.cpp` and then jumps to the rail family, and
it splits the segment subsystem into the separate cluster listed above (§7 compares the order).
The racing TU was located by `.text` cross-references to `tables\circuits.def` /
`tables\championship.def` / `racing_car%u`, not by order. All six classes are
non-polymorphic, so no vtable-sequence method can see them.

## 7. PC vs PS2

* **The PC has a multiplayer racing mode the PS2 build cannot have** — the PS2 has no
  networking layer equivalent to the PC's `inet.dll` (`basic/artifacts/dll.md`). Two racing
  methods are *larger* on PC than on PS2 purely because of it.
* **`championship.def` record stride** is 0xb8 on PC against 0xdc on PS2; the whole 0x24
  delta lands in one place, the two 10-entry opponent id arrays, which sit at PC `+0x68` /
  `+0x90` against PS2 `+0x8c` / `+0xb4`.
* **`carcyclopedia.def`** is 200 B per car on PC, 204 on PS2; the car list is 0xa4 on PC
  against 0xa8 on PS2, and the player/driver-model table is 0x24 on both.
* **Two PS2 racing-menu classes have no PC counterpart:** `GM_DamageDifficulty` (the PC
  toggles damage inline) and `GM_GameMenuRacing`.
* The PS2 main menu exposes a Racing mode — championships, single races, best times, a track
  roster; the PC build removes the item in both the data and the code (§3).
* **`circuits.def`** has the same 0x88-byte record on both builds.
* **The racing classes are NOT where PS2 link order predicts.** PS2 emits `C_racecam.cpp` →
  `C_racing_osegment` → `C_racing` → `C_circuit` consecutively; the PC order is in §6. All six
  classes are non-polymorphic on both builds.

## 8. How the engine is reached

### 8a. In retail: the mission 6 race

Mission 6's `GameInitStart` script (MISE06-AUTODROM) runs `racing_mission6_init 5, 12, 0, 44`.
`race_mission6_init` constructs `g_racing` when it is NULL, selects circuit 0 (Racing Circuit
Long, the `mise06-autodrom` scene), writes `g_racing+0x28 = 2` and `g_racing+0x2c = 3`, stores the
lap count and the player's car model, generates the opponents and enables damage; `racing_mission6_start` then runs the race from
`game_tick` (`structs/race_session.md` §4). The mission's `scene2.bin` carries 26 `C_racecam`
actors, and they register with the camera because `g_racing` is set (`structs/camera.md` §3b).
**Confirmed in game:** after the race finishes, the player watches the other racers from those
trackside cameras.

Every other mission leaves `g_racing` NULL, so there the ~310 `g_racing == 0` gates in ordinary
gameplay code answer "no race is running".

### 8b. In a modded install

The mod's own readme describes the player-facing behaviour, and it matches the code:

* the championship is the entry point; rounds unlock through
  `champ_circuit_set_lock_state`, which mirrors the unlocked state onto the `C_circuit`
  so single-race mode can offer the track;
* `change_mission` in a scene script can start a track directly, because a circuit is
  just a scene named by `circuits.def`'s second field.

The championship and single-race menus stay unreachable in retail: their only entry is the deleted
main-menu item (§3).

## 9. Open items

| item | state |
|---|---|
| the two padding bytes | `0x0040bc9d` / `0x0040c5bd`, both `-0x96`, both unreferenced. Measured, unexplained |
| the two `C_racecam` `70.0f` floats | which one is the field of view and which the activation distance. The cameras do cut in (§8a) |
| the race start lights | does `hud_set_race_start_flag` render as three lights plus GO, matching the four countdown steps `racing_tick` feeds it? Not observed |
