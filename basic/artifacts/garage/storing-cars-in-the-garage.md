# Storing cars in the garage

**Created** 2026-10-05 22:01 +0200 · **Modified** 2026-10-05 22:01 +0200

How the PC build of Mafia decides which cars go into the player's garage when a mission ends,
where the "garage area" comes from, how big it is and which cars it refuses. The other build is
compared in §7 only.

---

## 1. Summary

* Every time a mission is torn down, `game_done` calls `garage_game_done`, which **banks** cars into
  the garage's 25-slot table (`structs/garage.md` §1). It has four paths (§5); the one that stores a
  car "parked near Salieri's bar" is the **area scan**.
* The area scan is **not tied to a hard-coded position or to a "city" mission type.** It looks up two
  marker frames by name in the loaded scene, `oblast_garaze_a` and `oblast_garaze_b` ("garage area"),
  and banks every eligible car standing in the rectangle between them (§2, §3). A scene without the
  two frames has no garage area, whatever its coordinates.
* The rectangle is **31.0 m by 71.5 m on the ground plane**, height ignored, and covers the street in
  front of Salieri's bar, the bar building and the garage yard behind it (§3).
* Any car actor counts, scene-placed or spawned, whether or not the player ever drove it — unless a
  script switched it off, it is wrecked, it is a police model, its frame name contains `nesahat`, or
  it is one of the three Bolt trucks (§4).
* The "new cars have been added to the garage" screen is separate from the banking: it appears only
  when the mission ends through a script (`change_mission` or `endofmission`) **and** that mission ran
  the script command `garage_show` (§6).

## 2. Where the two marker frames come from

A mission directory holds two scene files, and `mission_open` loads them in this order:

| file | what it is | how the game reads it |
|---|---|---|
| `scene.4ds` | the mission's 3D scene: geometry and the frame hierarchy | `mission_open` opens `missions\<mission>\scene.i3d` through the scene object, which loads the `.4ds` on disk |
| `scene2.bin` | the placement layer: frames with positions, the models they load, lights, sounds, object definitions, scripts (`structs/mission.md` §4a) | `mission_open_bin`; each frame record looks its frame up by name, edits it when it exists and creates it when it does not |

A model that a `scene2.bin` record loads onto a frame brings its own frames with it, named with that
frame's name as a prefix. Every city mission places the frame `watercity` with the model
`watercity.i3d` (`models/watercity.4ds`), so the city's frames become `watercity.<name>`;
`scene2.bin` itself addresses them that way (`watercity.kolize01` is the frame `kolize01` of
`watercity.4ds`).

`garage_game_done` asks the scene for `oblast_garaze_a`, falling back to
`watercity.oblast_garaze_a`, and the same for `_b`. The frames ship in two places:

| missions | where the frames are | name that resolves |
|---|---|---|
| `MISE03-SALIERY`, `MISE03-SALIERYKONEC`, `MISE04`/`05`/`06`/`07`/`07B`/`09`/`10`/`11`/`14`/`15`/`16`/`17`/`18-SALIERY` | the mission's own `scene.4ds` | `oblast_garaze_a` / `_b` |
| every mission whose `scene2.bin` places the `watercity` frame — every `*-MESTO`, `MISE01`, `MISE02A-TAXI`, `MISE03-MORELLO`, `MISE07B-CHULIGANI`, `FREERIDE`, `FREERIDENOC`, `FREEITALY`, `EXTREME` | `models/watercity.4ds` | `watercity.oblast_garaze_a` / `_b` |
| every other mission, `MISE02-ULICKA`, `MISE02-SALIERY` and `MISE12-SALIERY` among them | nowhere | none |

When either frame is missing the rectangle stays at zero size at the world origin, so in practice no
car is banked by the area scan in such a mission.

## 3. The garage area

### 3a. Coordinates

The two frames are root-level dummies (0.67 m marker cubes). `watercity` sits at the origin with no
rotation or scale in every city mission, so their `.4ds` positions are world positions; the
Salieri scenes carry the same two positions with the names swapped:

| frame | X | Y (height) | Z |
|---|---:|---:|---:|
| `oblast_garaze_b` in `watercity.4ds`, `_a` in the Salieri `scene.4ds` | −1784.51 | −5.78 | 67.34 |
| `oblast_garaze_a` in `watercity.4ds`, `_b` in the Salieri `scene.4ds` | −1753.50 | −5.78 | −4.18 |

`garage_game_done` takes the minimum and the maximum of each axis, so the naming does not matter.
The test uses **X and Z only**; Y, the height, is never compared. The area is therefore a rectangle
on the map: **X −1784.51 … −1753.50 (31.0 m), Z −4.18 … 67.34 (71.5 m)**. A car directly above or
below it counts as well.

The point that must lie inside is the car's position `actor+0x24` / `+0x2c`, which `car_update`
refreshes from the body position on every tick of a locally simulated car — the car's centre, not a
corner of it.

### 3b. Map

To scale: one character is 1 m across, one row is 2 m. `1` … `16` are the frames `park01` …
`park16` of the Salieri `scene.4ds`; `garage_game_init` spawns each stored car on the next
`park%02d` frame it finds. All sixteen lie inside the area.

```
  Z(m)  X=-1784.5                        X=-1753.5
  67.3   +--------------------------------+   corner (-1784.51, 67.34)
   66    |   12        13                 |
   62    |  11                            |
   58    | 10                             |
   54    |  9                             |   the garage yard
   50    | 8                              |
   46    | 7                              |
   42    |              14                |
   40    |              15  4   5         |
   36    |1                               |
   34    |2                               |
   30    |3                               |
   22    |                     6          |
   18    |                16              |
   16    |            T                   |   T = the bar's back threshold (bar_prah_out)
   12    |   the bar building             |   the bar's interior lights, Z 6 .. 19
    6    |          D                     |   D = the bar's front door (dvere_front, Z 5.3)
    0    |   the street in front of the   |
   -4    |   bar, about 9.5 m deep        |
  -4.2   +--------------------------------+   corner (-1753.50, -4.18)
          ^-1784    ^-1774    ^-1764    ^-1754
```

"Parked in front of Salieri's bar" is the strip between the area's front edge (Z −4.18) and the
front door (Z 5.3); the bar building and the whole yard behind it are inside the area too.

## 4. Which cars the area scan considers

The scan walks `C_game`'s two actor lists (`structs/game.md` §4b): the **active list** (act state 0)
and the **inactive list** (act state 1). `game_init` puts every actor of the mission into the active
list, and actors spawned at run time join by their act state, so scene-placed and spawned cars are
both considered; an actor a script switched `off` (act state 2) is in neither list. Ambient traffic
cars are low-detail stand-ins kept by the traffic system, not actors of either list, until something
takes one over: `traffic_car_take_over_cruising_car` (from `human_use_actor` or an AI group) then
builds a real car named `Corle auto N` and registers it as a temporary actor, which the scan does see.

A car is banked when **all** of these hold:

| gate | field or call |
|---|---|
| it is a car | kind `+0x10 == 4` |
| it stands inside the area | `+0x24` (X) and `+0x2c` (Z) inside the rectangle |
| its frame is switched on | frame `+0xac` bit 0 |
| it is not a police model | `+0x1ff2` clear — set by `car_init` for the models `POLI`, `POLCAD`, `POLIMFOR`, `POLIMTUD` (`structs/car.md` §1) |
| its frame name does not contain `nesahat` ("do not touch") | `strstr`, which is case-sensitive: the shipped name `NesahatANepouzivat` does not match |
| it is usable | `car_is_usable(1)`: the body and engine damage pools and the fuel above 0, not death-processed, the `+0x2071` byte clear, no wheel lost |
| it is not in the working list yet | the list is searched first |
| it is not a Bolt truck | car-database ids `0x39` `trucka`, `0x3a` `truckb`, `0x3d` `TruckBx` are refused |

No test asks whether the player ever drove the car. Frames named with `nesahat` occur in 25 shipped
`scene2.bin` files; they mark the cars the designers keep out of the garage.

## 5. The four banking paths

`garage_game_done` builds a working list of `{packed id, live car, slot}` records, then writes the
table back. Its paths, in order:

| path | which cars | refused |
|---|---|---|
| 1. the table itself | each of the 25 slots: a slot holding only a packed id is kept; a slot holding a live car is kept when that car is carried into the next mission or still passes `car_is_usable(1)` | a parked car that is no longer usable loses its slot |
| 2. the area scan, active list | §3, §4 | police models, `nesahat`, unusable, duplicates, the three Bolt trucks |
| 3. the area scan, inactive list | the same test over the inactive list | the same |
| 4. the last car the player entered | `garage+0x1e8`, wherever it stands, only when the mission ran `garage_show` or `garage_addlaststolen` | police models, unusable, duplicates, the three Bolt trucks **and `0x2b` `bugatti`** |

`garage+0x1e8` is written by `garage_set_last_entered_car`: `human_use_actor` calls it on every car
the player gets into, `engine_new_mission` with the car carried into the new mission, and
`human_save_game_load` on a restore.

`garage_add_car_by_actor` and `garage_add_car_by_index` are not part of the end-of-mission banking:
their only callers are the script commands `garage_addcar` (the car in an actor register, or the
car the player sits in) and `garage_addcaridx`, and neither refuses any car.

After the paths, every new record (slot −1) takes the first free slot: its packed id goes into the
slot, and the live pointer only when the car is carried into the next mission. A parked car that is
not carried on leaves the world — its occupants are taken out, it is dropped from both actor lists,
the radar and the scene — and its slot keeps only the packed id.

## 6. What the player sees

* **When the banking happens.** On every mission teardown, because `game_done` always calls
  `garage_game_done`. A mission change by `change_mission` therefore stores a car standing in the
  area even when no screen says so.
* **The discard screen.** Only when the mission is ending under script control — `C_game+0x3665`, set
  by both `change_mission` and `endofmission` — and more than 16 cars survive, the "The garage is
  full" screen opens until the count fits (`structs/garage.md` §3a).
* **The results screen.** "New cars have been added to the garage" / "Cars destroyed during the
  mission" (`garage_show_results`) needs the same script-controlled end **and** `garage+0x1cc`, which
  only the script command `garage_show` sets. Fifteen missions run it, among them `MISE05-MESTO`
  (script `detector_konecmise`, right before `change_mission "mise06-saliery"`); `MISE02A-TAXI`
  does not, so a car stored at its end appears in the garage without a screen.

Worked cases:

| mission | what happens |
|---|---|
| `MISE02A-TAXI` | a car pushed into the area is banked by the area scan when the mission changes, silently |
| `MISE05-MESTO` | the race car, `bugatti00.i3d` (car-database `bugatti`, Carrozella C-Otto 4WD), is banked by the area scan when it stands in the area; path 4 would refuse it, but the player is in the pushing car, so that car is the last one entered |
| `MISE01` | the script `detector_konec` moves the taxi to `placement_hidetaxi` (X −1622.4, Z 35.5, about 131 m outside the area) before the ending, and the chasing `enemy_car` is switched off by the script `supervisor` once the player gets 200 m ahead, and the mission ends only after the player has lost it (user observation); neither car is in the area scan at the end |

## 7. PC vs PS2

The PS2 `G_GarageClass::GameDone` runs the same area scan: the same four frame names, the same two
positions (in the city's LEFT half of the PS2's split city scenes and in the Salieri `scene.4ds`),
the same X/Z rectangle, the global actor list instead of the two act-state lists, and the same
frame-on, police-model, `nesahat` and `IsCarUsable` gates. Three things differ:

| | PC | PS2 |
|---|---|---|
| how a car is added | inline in `garage_game_done`, with a separate refusal list per path | every path calls `AddCarByActor`, which refuses the police model and four car-database ids itself |
| refused by the area scan | the three Bolt trucks | the three Bolt trucks **and `bugatti`** (PS2 ids `0x24`, `0x32`, `0x33`, `0x36`; the same four entries of the PS2 `carindex.def`) |
| garage size | 25 slots, discard screen above 16 | 10 slots; under script control with `garage_show` the discard screen above 7, without it `HlidejPocetHard` silently destroys the lowest-id cars until 8 remain |

**Why the pushed `MISE05-MESTO` race car is not stored on the PS2.** The PS2 places the same
`bugatti00.i3d` in that mission, and `AddCarByActor` refuses `bugatti` on every path, the area scan
included. On the PC the area scan carries its own list with only the three trucks, so the race car
is banked. `AddCarByActor`'s PS2 callers are `GameDone` (area scan and last-entered car), the
`GARAGE_ADDCAR` script arm and the PS2-only garage menu (`ParkCar`, `ChangeCar`).

## 8. Related

* `structs/garage.md` — `G_GarageClass`, the 25-slot table, the screens, the request bytes
  `+0x1cc` / `+0x1cd`.
* `structs/game.md` — `game_done`, the active and inactive actor lists (§4b), `C_game+0x3665`.
* `structs/mission.md` — the `scene2.bin` chunk vocabulary (§4a) and `mission_open`.
* `structs/car.md` — `+0x1ff2`, `car_is_usable`; `structs/car_database.md` — packed ids and
  `carindex.def`.
* `structs/loadsave.md` — `+0x1e8` and the garage's savegame block.
* `Game-ghidra.exe.annotated.c` — the inline comments in `garage_game_done` and
  `garage_add_car_by_actor`.

## 9. Open items

Nothing is open.
