# The actor kind field `+0x10` and per-kind properties

**Created** 2026-10-14 20:49 +0200 · **Modified** 2026-11-01 15:38 +0100

Documents the shared **kind field** every `C_actor`-derived object carries, the value→class enum,
the developers' script-facing names for the kinds, the per-kind properties recovered from the
decomp, and the second discriminator at `+0x14`. Both fields are `C_actor` members
(`structs/actor.md` §1); the `C_entity` class (kind `0x1b`) is `structs/entity.md`.

**Kind field location:** `actor+0x10`. Each subclass constructor writes its own value; the base
`C_actor` default is `1`. All cross-build material is in §4.

---

## 1. Type enum (`actor+0x10`)

Sixteen of these values are also named in the shipped script-compiler table read in §1a, and every
one agrees with the value here. Where §1a supplies a name, that is the developers' script-facing
word for the kind.

| value | class | PC ctor / vtable / size | what it is |
|---|---|---|---|
| 1 | `C_actor` | - | base default, overridden by every subclass |
| 2 | `C_player` | - | the player actor (a `C_human` subclass) |
| 3 | `C_human` | - | human / pedestrian body |
| 4 | `C_car` | - | drivable car |
| 5 | `C_detector` | `detector_ctor` `0x004793b0` · `0x0063c008` + `0x0063c0a4` · `0x128` | the **script trigger volume** behind the `detector_*` script vocabulary; derives from `C_actor` **and** from `C_program` at `+0x70`. `structs/detector.md` |
| 6 | `C_door` | `door_ctor` `0x004800f0` · `0x0063c180` · `0x1b8` | one leaf on one angle; `door_set_state` is the public verb, `+0x114`/`+0x118` link a twin leaf, and the leaf opens away from whoever opened it. `structs/door.md` |
| 8 | `C_railway` | `railway_ctor` `0x004d74e0` · `0x0063cbd0` · `0x36c` | the ambient rail-traffic system — trams **and** elevated trains (§3). `structs/railway.md` |
| 9 | `C_box` | `box_ctor` `0x0045ae00` · `0x0063b980` · - | **PC-only** rigid-body box with its own complete solver in 55 functions, deriving straight from `C_actor`. Distinct from `C_box_new` (`0x23`, §4). `structs/box.md` |
| 0xa | `C_bottle` | `bottle_ctor` · `0x0063b870` · `0x138` | throwable bottle; base of the cartridge / grenade / smoke-trail family. `structs/bottle.md` |
| 0xc | `C_traffic_car` | `traffic_car_ctor` `0x004f65a0` · `0x0063ce20` · - | the ambient-traffic **manager**, not one car: owns the `S_ELEMENT` cars, `S_MAFIAN` crews, `S_BRIDGE` slots, `S_LOCKCROSS` crossing locks, the parked-car and stolen-car lists — and spawns police reinforcement and mafia hit cars. Ambient *pedestrians* are `C_traffic_element` (`structs/pedestrian.md`); the two meet in `traffic_car_seat_pedestrian_as_driver`. `structs/traffic_car.md` |
| 0xe | `C_grenade` | `grenade_ctor` `0x0048af30` · `0x0063c3c0` · `0x160` | the thrown grenade **and** the Molotov — two independent payload flags on one class. A `C_bottle` subclass. `structs/grenade.md` |
| 0xf | `C_smoke_trail` | `smoke_trail_ctor` `0x004f60e0` · `0x0063cd80` · - | a short-lived piece an explosion throws, a `C_bottle` subclass. `game_init` builds 32 of them into a ring at `C_game+0x324`, and `game_new_smoke_trail` re-arms them from it for `game_new_explosion` only; the entity factory has no `0xf` arm. Its model frame at `+0x68` shows the `2stone.i3d` mesh, and it retires itself once its flight has stopped (`+0x7c`/`+0x80` both `<= 1e-08`) or its fall accumulator passes 50.0 (`+0x80`). Its `Init` is the three-way slot-18 fold shared with `C_cartridge` and `C_grenade`. `structs/smoke_trail.md` |
| 0x10 | `C_cartridge` | `cartridge_ctor` `0x00476f80` · `0x0063be18` · `0x13c` | the ejected shell casing, a `C_bottle` subclass adding exactly one dword. Pooled 100 + 30 on `C_game` from `2nabojnice.i3d` / `2nabojniceb.i3d` (the file name means cartridge case) and ejected by `game_new_cartridge`. `structs/cartridge.md` |
| 0x11 | `C_DropOut` | `0x0048a530` · `0x0063c320` · `0x3b8` | a car part that has come off — a wheel or a body panel — spawned by `car_track_spawn_dropout_part` as its own temporary actor so it can fall and roll. In game the PC sheds wheels, bumpers and lights. Live parts are capped at **15**. `structs/dropout.md` |
| 0x12 | `C_traffic_generator` | `pedestrian_manager_ctor` `0x004b1c20` · `0x0063c778` · - | the **ambient-pedestrian spawner** (people only): an arena of `C_traffic_element` records under a population budget, activated round-robin by `pedestrian_manager_ai`. The only writer of kind `0x12`, so it is the one actor the script name `Traffic_person` can find (§1a). `structs/pedestrian.md` |
| 0x13 | `C_don_ocas` | `pedestrian_crowd_ctor` `0x004b0e50` · `0x0063c6d8` · `0x70` | the crowd's **proxy actor**, one instance in the game (`C_game+0x2a1c`): it owns every ambient pedestrian's `I3D_model`, so all engine-level collision / hit / bounds dispatch lands on it. Derives directly from `C_actor`. `structs/pedestrian.md` |
| 0x14 | `C_bridge` | `bridge_ctor` `0x0045f980` · `0x0063bb40` · - | (draw)bridge. **State field `+0x144`**: `bridge_ai`'s six-state cycle puts raising / up / lowering at **3 / 4 / 5**, and the dynamic collision object is registered only across that band. `human_find_blocking_drawbridge` treats 3/4/5 as mid-cycle and, while the player is in a car within **100 units** of such a bridge, refuses to let him get out. `structs/bridge.md` |
| 0x15 | `C_dog` | `dog_ctor` `0x0047ab60` · `0x0063c0d0` · `0x310` | the guard dog: patrols a route of `C_web_node`s and attacks when the player comes close. Derives from **`C_actor`, not `C_entity`** — no `ai_sensors`, no smell. `structs/dog.md` |
| 0x16 | `C_airplane` | - | the scripted aeroplane. `structs/airplane.md` |
| 0x17 | `seagull_flock` (name ours) | `seagull_flock_ctor` `0x004ea1d0` · `0x0063cc98` · `0x244` | **PC-only** and **never constructed in the shipped game**. One scene actor owning **50** seagulls (`racek.i3d`, *racek* = seagull) flying inside a box built from a chain of navigation-web nodes near the actor's frame. The ctor's only callsite is the entity factory's `case 0x17`, and kind `0x17` occurs in none of the 79 shipped `scene2.bin` and none of the 92 `.chg` difference files; the shipped `racek*` objects are sound emitters, so seagulls are heard and never seen. `structs/seagull_flock.md` |
| 0x18 | `C_turnout` | `railway_switch_ctor` `0x00508150` · `0x0063cee0` · - | the railway **turnout** (points), PC prefix `railway_switch_`: a blade swinging between two rest positions that **rewires the navigation graph** at each end-stop; the player can throw it from the action menu. `structs/railway_switch.md` |
| 0x19 | `C_pumper` | `pumper_ctor` `0x004ce000` · `0x0063ca58` · - | the gas-station attendant: walks out to a stopped car, refuels it and **charges 500**, and refuses a driver the police are after. Derives directly from `C_actor`. `structs/pumper.md` |
| 0x1a | `C_crash_obj` | `crash_obj_ctor` `0x004786c0` · `0x0063bf68` · `0x3e4` | the breakable scene prop, one of `C_PhThing`'s five subclasses. Never built by the entity factory: the only construction path is `crash_obj_create_actor` via `crash_obj_create_from_model`. Everything it does is configured by single letters in the frame's user-property tag. `structs/crash_obj.md` |
| 0x1b | `C_entity` | `entity_ctor` `0x004114d0` · `g_entity_vt` · `0x128c` | the scriptable NPC actor and police pursuit target. `structs/entity.md` |
| 0x1c | `C_racecam` | - · `0x0063cb00` · - | the racing camera actor: it holds one `G_RaceCamera` record at `+0x70` and registers it with the `G_Camera` singleton for the scene, only while `g_racing != 0` (in retail, mission 6 alone). `structs/racecam.md`; the mode itself in `basic/artifacts/racing.md` |
| 0x1d | `bendable_plant` (name ours) | `bendable_plant_ctor` `0x004af030` · `0x0063c608` · `0xa0` | **PC-only** and **never constructed in the shipped game**. A clump of props that bend when you walk through them: up to **7** cloned child `OBJ_` prototypes, each a dynamic-collision record of type **`0x47`** that never blocks anything. The ctor's only caller is the entity factory's `case 0x1d`, and `0x1d` appears in none of the 79 shipped `scene2.bin` and none of the 92 `.chg` difference files. `structs/bendable_plant.md` |
| 0x1e | `C_wagon` | - | the scriptable mission wagon prop (MISE15-PRISTAV), **not** a tram (§3) |
| 0x1f | `C_irenka` | - · `0x0063c568` · - | the police **spike strip** — an invisible deployable line-segment collision barrier (asset `barrier.4ds`). PC prefix `spike_strip_*`; extends `C_actor`. Spawned and freed by the police roadblock temp-actor pool. `structs/spike_strip.md` |
| 0x20 | `C_public_physics` | `public_physics_ctor` `0x004cde20` · `0x0063c9b8` · `0x3b0` | the `create_physicalobject` prop: the general-purpose `C_PhThing` subclass that turns an existing scene frame into a free-falling physics prop, where its four siblings are each specialised to one kind of debris. Five creators: `human_free_hand`, `pedestrian_drop_umbrella`, `human_start_weapon_effect`, dispatcher case `0xcc`, and the entity factory. `structs/public_physics.md` |
| 0x21 | `C_actor_shot` | `actor_shot_ctor` `0x0044e9a0` · `0x0063b720` · `0x8c` | a placed object that fires a weapon on a timer at the weapon's own cadence. No shipped `scene2.bin` places one; 38 of the 92 `.chg` difference files add 122 of them through `game_load_differences`. `structs/actor_shot.md` |
| 0x22 | `C_clock` | `clock_ctor` `0x00477330` · `0x0063beb8` · `0x88` | a working clock actor: resolves the child frames `Mala` / `Velka` / `Sekundova` (the Czech words for the hour, minute and second hand) and drives them from the machine's wall clock via `GetLocalTime`. Only three shipped models carry the hand frames — `9hodinky.4ds`, `xhodiny 4.4ds`, `rucky.4ds`. `structs/clock.md` |
| 0x23 | `C_box_new` | `0x0045f160` · `0x0063baa0` · - | a `C_PhThing` subclass (its ctor chains `physical_object_ctor`; §4). `structs/box_new.md` |
| 0x25 | `C_drop_in_weapon` | `drop_in_weapon_ctor` `0x0048a0d0` · `0x0063c280` · `0x3f8` | **an item dropped into the world**, not specifically a weapon: it publishes an `S_using_object` record into the registry behind the player's action menu. `+0x3b0` is a `predmety.def` item id. `structs/drop_in_weapon.md` |
| 0x26 | `C_rail_generator` | - | **PS2-only** (§4): the entity factory `mission_create_actor` has no `0x26` arm and the PC has no function of the class |

Police pursuit treats **type 2 (`C_player`)** and **type 0x1b (`C_entity`)** as pursuable actors; it
reads the vehicle they ride (`actor+0x98`) and special-cases that vehicle's type (§2).

## 1a. The developers' name→kind table

The script command `findnearactor xx, yy, "type"` (dispatcher case `0x73`) takes its type as a
**NAME**. The load-time arm in `program_decode_params` resolves it against a table of sixteen
`0x14`-byte records — the name at record`+0`, the kind dword at record`+0x10` — walked forwards
from `0x0063e588` until `0x0063e6c8`. Read straight out of `Game.exe`:

| script name | kind | class (§1) |
|---|---:|---|
| `Player` | 2 | `C_player` |
| `Enemy` | 0x1b | `C_entity` |
| `Car` | 4 | `C_car` |
| `Detector` | 5 | `C_detector` |
| `Door` | 6 | `C_door` |
| `Railway` | 8 | `C_railway` |
| `Box` | 9 | `C_box` |
| `Bottle` | 0xa | `C_bottle` |
| `Traffic_person` | 0x12 | `C_traffic_generator` ¹ |
| `Traffic_car` | 0xc | `C_traffic_car` |
| `Bridge` | 0x14 | `C_bridge` |
| `Dog` | 0x15 | `C_dog` |
| `Airplane` | 0x16 | `C_airplane` |
| `Turnout` | 0x18 | `C_turnout` |
| `Pumper` | 0x19 | `C_pumper` |
| `Race camera` | 0x1c | `C_racecam` |

¹ `Traffic_person` can only ever find the `C_traffic_generator` spawner, not a pedestrian:
`findnearactor` walks the global actor list comparing the kind, an ambient pedestrian is a plain
record that has no kind and never enters that list (`structs/pedestrian.md` §4), and
`pedestrian_manager_ctor` is the only writer of kind `0x12`. No shipped script searches for
`Traffic_person`; the shipped `findnearactor` calls ask for `CAR` and `RAILWAY`.

**Why this matters more than a name list.** The kind map in §1 is reconstructed class by class from
ctors, vtables and `GetKind` gates. This table is a **single shipped artifact that states sixteen of
those values outright**, and it agrees with every one of them — including `0x23` vs `9` staying
distinct (`C_box` is `Box`; `C_box_new` has no script name).

**`Enemy` is `C_entity` (`0x1b`), not a hostile-specific class.** The whole scripted-character
vocabulary — `enemy_*` commands, `enemy_to_AI` / `enemy_to_Script` events — is named after this word,
so "enemy" in the script language means *a scripted `C_entity`*, friendly or not.

The sixteen are the kinds a script may **search for**; classes absent from the table are still
reachable by other means. A name that matches nothing leaves the stored kind at `-1`, which matches
no actor, so a misspelled type silently finds nothing rather than erroring.

## 2. Known per-type properties

**Speed** (how the police pursuit reads a ridden vehicle's speed, `police_action_tick`):

| vehicle | type | speed field (PC) | note |
|---|---|---|---|
| `C_car` | 4 | `car+0x688` | scalar speed; also read by `police_manager_tick` (`*(record+4)+0x688`) |
| `C_railway` | 8 | `railway+0xd4` | scalar speed |
| any other | - | virtual `vtable+0x70` | writes a velocity vector; magnitude taken |

---

## 3. Rail vehicles — `C_railway` vs `C_wagon`

`C_railway` and `C_wagon` are **not** "tram vs train":

- **`C_railway` (type 8)** = the ambient rail-**traffic system**, handling **both trams and elevated
  trains**. A large class (~100 methods): owns vagons (`GetVagon`, `GetNearestVagon`), stations and
  stops (`isInStation`, `IsStopPoint`, `MustStop`), schedules (`IsOnSchedule`), rider traffic
  (`AddPerson`/`RemovePerson` of `S_rail_human`, `GeneratePersonTrafic`), and the per-frame
  `trafficAI` / `GetSafeSpeed` movement. The tram vs elevated-train difference is **data-driven**
  (rail line + model + schedule via `C_web_node`), not a class split — a pursuit target on either
  reads speed at `railway+0xd4`. In game the elevated train runs faster than the tram, and that
  ordering lives in the line/model data.
  It is not one class but six types in one translation unit `[0x004d35f0, 0x004ea1a0)` (126 starts):
  `C_railway` itself plus `S_RailSeats` (seat banks — a row is a bench, seat positions lerped along
  it), `S_rail_human` (a passenger, PC prefix `rail_passenger_`), `rail_doors` (two leaf banks,
  opening to π/2), `rail_glass`/`rail_window` (breakable windows, two hits each) and `rail_entries`
  (the boarding points pedestrians walk to). Departures are rate-limited to one per **60 000 ms** per
  stop by a shared schedule log, signals come from a file-scope `S_rail_semaphore` list, and a tram
  holds back when the vehicle ahead is within **40 units**.
- **`C_wagon` (type 0x1e)** = a **scriptable mission wagon prop**, a thin class (~20 methods:
  ctor/dtor, Init, AI, Update, path helpers, `RecompileNewPath`). It is the rollable wagon in mission
  `MISE15-PRISTAV` (harbour): scripts `aktivace vagonu 1` / `aktivace vagonu 2` drive it via the
  dedicated `wagon_setevent` / `wagon_getlastnode` script commands (frames `vagon1`,
  `vagcisterna4` = a tank wagon); reaching path nodes (`wagon_getlastnode` → 7 / 35) opens gates to
  progress the mission. Not part of city traffic and not a tram.
- Supporting: `C_turnout` (type 0x18) is the track switch. Kind `0x26`, `C_rail_generator`, exists
  only on the PS2 (§4).
- **Radar representation.** `railway_update` (`0x004da400`) puts rail vehicles on the HUD radar
  through the **object** list, not the vehicle list: on the in-range transition (state field
  `+0x2a8 == 2`) it calls `hud_radar_add_object_with_color(part, 0xffffff00)` — **yellow** — once for
  the engine and once per wagon (`+0xe8 + i*0x48`, count at `+0x1a8`), and removes them all again on
  the out-of-range transition (state == 1). The whole registration is gated on the byte at
  `railway+0x1d8`. That is *why* trams have their own radar colour: a rail vehicle is a
  `tDynamicCollObject` entry in `hud_state+0x94a8` and can never appear in the `C_car` list at
  `+0x9494`. User-observed in game ("trams are yellow"), then byte-confirmed.
  See `structs/hud.md` §7b and `basic/artifacts/hud_elements.md` §6.

Script evidence: `wagon_*` commands appear only around `MISE15-PRISTAV`'s `aktivace vagonu*` scripts;
trams are ambient (e.g. mission `MISE08-HOTEL`, scripts `tram1` / `tram2`, which just `frm_seton` a
frame on a random timer — the `C_railway` traffic system, no `wagon_*`).

### 3a. The FLAVOUR field `+0x14` — a second, `C_actor`-level discriminator

`+0x10` is the kind. **`+0x14` is a separate enum written by the same constructors**, and it exists
because a subclass can share its base's kind:

| `+0x14` | class | written by |
|---|---|---|
| 1 | `C_actor` | `actor_ctor` (which also writes kind 1) |
| 2 | `C_human` | `human_ctor` (which also writes kind 3) |
| 3 | **`C_car_ex`** | `car_extended_ctor` — and it writes **no kind at all**, inheriting 4 from `C_car` |

So `C_car_ex` is invisible to the kind enum by design: `human_game_done` identifies it as
`kind == 4 && +0x14 == 3`. Doc: `structs/car_extended.md`.

Consequence for the kind table in §1: **a class missing from it is not necessarily undiscovered** —
it may be a subclass that never writes `+0x10`. Check `+0x14` before concluding a kind value is
unused.

### 3b. The entity factory's `case 0x24`

`mission_create_actor(type)` (`0x005fed70`) — the scene-loader entity factory — maps the `scene2.bin`
actor type to a constructor. Its `case 0x24` is `operator_new(0x2254)` + `car_extended_ctor`, i.e.
**`C_car_ex`**. For every other case the switch value equals the kind the constructor writes; this
one is the exception, for the reason in §3a.

## 4. PC vs PS2

**Kind field.** The enum values in §1 are identical on both builds; the field sits at `entity+0x50`
on PS2 (`__ct__<N>C_<class>Fv` → `*(this+0x50) = <value>`), and the §1 values are byte-confirmed from
those constructors. The `+0x14` flavour field of §3a is PS2 `+0x54`, one step off the same bridge;
its writers there are `__ct__7C_actorFv` (1), `__ct__7C_humanFv` (2) and `__ct__8C_car_exFv` (3).

**Kind-table cross-build evidence (§1).** `C_box` (kind 9) is PC-only: `__ct__9C_box_newFv` writes
`0x23`, and the PS2 source-file manifest has no `C_box.cpp`. The seagull flock (kind `0x17`) is
PC-only: no PS2 ctor writes `0x17`, and `racek` occurs 0 times in the PS2 decomp. `C_rail_generator`
(kind `0x26`) is PS2-only: the PS2 factory `CreateActor__9C_missionFi` has a `0x26` arm that the PC
`mission_create_actor` lacks, and none of the class's PS2 methods has a PC twin. `C_box_new`'s PS2
ctor chains `__ct__9C_PhThingFv` exactly as the PC ctor chains `physical_object_ctor`, and
`box_new_init` alone shares five magic constants in order with its twin.

**Rail content.** In the PS2 build **no trams or trains actually drive in the city** (in-game
observation). The rail classes are still present in the PS2 code, so this is a content/spawn
difference, not a code removal.

## 5. Related

- `structs/actor.md` — the `C_actor` base that owns `+0x10` and `+0x14`, and the kind reader
  `folded_get_dword_0x10`.
- `structs/entity.md` — the `C_entity` class, kind `0x1b`.
- `structs/car_extended.md` — the `+0x14` value 3, the class the kind enum cannot see.
- `structs/railway.md`, `structs/wagon.md`, `structs/railway_switch.md` — the rail classes of §3.
- `structs/mission.md` — the scene loader whose entity factory `mission_create_actor` maps
  `scene2.bin` actor types to constructors (§3b).

## 6. Open items

Nothing is open.
