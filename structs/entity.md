# Entity / actor `type` field and per-type properties

**Created** 2026-07-10 22:46 +0200 · **Modified** 2026-09-24 12:45 +0200

Documents the shared **entity kind field** every `C_actor`-derived object carries, the value→class
enum, the per-type properties recovered from the decomp, and — from §4 on — the **`C_entity` class**
(kind `0x1b`), the scriptable-NPC AI brain.

**Kind field location:** `entity+0x10`. Each subclass constructor writes its own value; the base
`C_actor` default is `1`. All cross-build material is in §7.

---

## 1. Type enum (`entity+0x10`)

Sixteen of these values are also named in the shipped script-compiler table read in §1a, and every
one agrees with the value here. Where §1a supplies a name, that is the developers' script-facing
word for the kind.

| value | class | PC ctor / vtable / size | what it is |
|---|---|---|---|
| 1 | `C_actor` | - | base default, overridden by every subclass |
| 2 | `C_player` | - | the player actor (a `C_human` subclass) |
| 3 | `C_human` | - | human / pedestrian body |
| 4 | `C_car` | - | drivable car |
| 5 | `C_detector` | `detector_ctor` `0x004793b0` · `0x0063c008` + `0x0063c0a4` · `0x128` | the **script trigger volume** behind the `detector_*` script vocabulary (`detector_inrange` ×1123, `detector_issignal` ×567, `detector_setsignal` ×517, `detector_waitforuse` ×505, `detector_waitforhit` ×96 across the shipped scripts). Derives from `C_actor` **and** from `C_program` at `+0x70` — the ctor installs two vtables and `detector_program_process` reaches the owning frame as `this - 8`. Collision shape built from child frames named `DYNB…`. `structs/detector.md` |
| 6 | `C_door` | `door_ctor` `0x004800f0` · `0x0063c180` · `0x1b8` | one leaf on one angle; `door_set_state` is the public verb, `+0x114`/`+0x118` link a twin leaf, and the leaf opens away from whoever opened it. `structs/door.md` |
| 8 | `C_railway` | `railway_ctor` `0x004d74e0` · `0x0063cbd0` · `0x36c` | the ambient rail-traffic system — trams **and** elevated trains (§3). `structs/railway.md` |
| 9 | `C_box` | `box_ctor` `0x0045ae00` · `0x0063b980` · - | **PC-only** rigid-body box with its own complete solver in 55 functions, deriving straight from `C_actor`. Distinct from `C_box_new` (`0x23`): `__ct__9C_box_newFv` writes `0x23`, and the PS2 source-file manifest has no `C_box.cpp`. `structs/box.md` |
| 0xa | `C_bottle` | `bottle_ctor` · `0x0063b870` · `0x138` | throwable bottle; base of the cartridge / grenade / smoke-trail family. `structs/bottle.md` |
| 0xc | `C_traffic_car` | `traffic_car_ctor` `0x004f65a0` · `0x0063ce20` · - | the ambient-traffic **manager**, not one car: owns the `S_ELEMENT` cars, `S_MAFIAN` crews, `S_BRIDGE` slots, `S_LOCKCROSS` crossing locks, the parked-car and stolen-car lists — and spawns police reinforcement and mafia hit cars. Ambient *pedestrians* are `C_traffic_element` (`structs/pedestrian.md`); the two meet in `traffic_car_seat_pedestrian_as_driver`. `structs/traffic_car.md` |
| 0xe | `C_grenade` | `grenade_ctor` `0x0048af30` · `0x0063c3c0` · `0x160` | the thrown grenade **and** the Molotov — two independent payload flags on one class. A `C_bottle` subclass. `structs/grenade.md` |
| 0xf | `C_smoke_trail` | `smoke_trail_ctor` `0x004f60e0` · `0x0063cd80` · - | the short-lived damage smoke plume, a `C_bottle` subclass. Owns one particle-effect object at `+0x68` and retires itself once the plume has faded (`+0x7c`/`+0x80` both `<= 1e-08`) or aged out (`+0x80 > 50.0`). Its `Init` is the three-way slot-18 fold shared with `C_cartridge` and `C_grenade`. `structs/smoke_trail.md` |
| 0x10 | `C_cartridge` | `cartridge_ctor` `0x00476f80` · `0x0063be18` · `0x13c` | the ejected shell casing, a `C_bottle` subclass adding exactly one dword. Pooled 100 + 30 on `C_game` from `2nabojnice.i3d` / `2nabojniceb.i3d` (*nábojnice* = cartridge case) and ejected by `game_new_cartridge`. `structs/cartridge.md` |
| 0x11 | `C_DropOut` | `0x0048a530` · `0x0063c320` · `0x3b8` | a car part that has come off — a wheel or a body panel — spawned as its own temporary actor so it can fall and roll. `car_track_spawn_dropout_part` creates one on a "part came off" keyframe, resolving a wheel through `car+0xd24` and a panel through the `0x34`-stride table at `car+0x308`; `recorder_new_track` labels its tracks `drp_%d`. In game the PC sheds wheels, bumpers and lights. Live parts are capped at **15** in a class-wide arena, oldest killed to make room. `structs/dropout.md` |
| 0x12 | `C_traffic_generator` | `pedestrian_manager_ctor` `0x004b1c20` · `0x0063c778` · - | the **ambient-pedestrian spawner** (people only). Owns an arena of `C_traffic_element` records at stride `0x200`, a designer table of `0x11`-byte model names with per-name spawn counts, and a population budget scaled by `g_traffic_ratio_all`. `pedestrian_manager_ai` walks the arena round-robin every `0x70` ms and activates one free slot. `pedestrian_manager_spawn_cop_from_pedestrian` deactivates a foot-cop-category pedestrian and allocates a full `C_entity` (`operator_new(0x128c)`) in its place. `structs/pedestrian.md` |
| 0x13 | `C_don_ocas` | `pedestrian_crowd_ctor` `0x004b0e50` · `0x0063c6d8` · `0x70` | the crowd's **proxy actor**, one instance in the game: `game_init` allocates it and stores it at `C_game+0x2a1c`, and `pedestrian_set_model` stamps it as the owner of every ambient pedestrian's `I3D_model`, so all engine-level collision / hit / bounds dispatch lands on it. Its four real methods either sweep `g_pedestrian_list` or resolve the pedestrian via `pedestrian_manager_get_pedestrian_from_frame`. Derives directly from `C_actor`. `structs/pedestrian.md` |
| 0x14 | `C_bridge` | `bridge_ctor` `0x0045f980` · `0x0063bb40` · - | (draw)bridge. **State field `+0x144`**: `bridge_ai`'s six-state cycle puts raising / up / lowering at **3 / 4 / 5**, and the dynamic collision object is registered only across that band. `human_find_blocking_drawbridge` treats 3/4/5 as mid-cycle and, while the player is in a car within **100 units** of such a bridge, refuses to let him get out. `structs/bridge.md` |
| 0x15 | `C_dog` | `dog_ctor` `0x0047ab60` · `0x0063c0d0` · `0x310` | the guard dog. Patrols a route of `C_web_node`s, remembers blocked connections in a `vector<blocked_con>` at `+0x2fc`, re-plans through `ai_wayfind`, attacks when the player comes close. Derives from **`C_actor`, not `C_entity`** — no `ai_sensors`, no smell; its whole perception is one distance test on a 2000 ms timer. Embeds `C_animations_machine_de_luxe` at `+0xe8`; 13 `D*` clips; `hlavapsa.I3D` is the severed head. A vehicle kills it in one hit (flat 100.0). `structs/dog.md` |
| 0x16 | `C_airplane` | - | the scripted aeroplane. `structs/airplane.md` |
| 0x17 | seagull flock (`seagull_flock_`, name ours) | `seagull_flock_ctor` `0x004ea1d0` · `0x0063cc98` · `0x244` | **PC-only.** One scene actor owning **50** seagulls: `seagull_flock_game_init` seeds an axis-aligned box from the actor's frame, then loads `racek.i3d` (*racek* = seagull) fifty times, each bird a `0x5c`-byte record in the vector at `+0x114`/`+0x118`. `seagull_flock_ai` steers a bird to a random point in the box, rejecting points on the wrong side of the `+0x240` clipping planes; `seagull_flock_hit` downs the bird nearest an incoming ray (state 4, 2000 ms timer). **Never constructed in the shipped game** — no PS2 ctor writes `0x17`, `racek` occurs 0 times in the PS2 decomp, the ctor's only callsite is the entity factory's `case 0x17`, and special type `0x17` occurs 0 times across all 79 shipped `scene2.bin` including `INTRO`. The `racek*` objects that do ship are I3D **sound** emitters (`14m_gull*.wav` / `18_gull*.wav` / `17_bird*.wav`) hosted by a `C_detector` whose script only calls `frm_seton` — seagulls are heard and never seen (§9). `structs/seagull_flock.md` |
| 0x18 | `C_turnout` | `railway_switch_ctor` `0x00508150` · `0x0063cee0` · - | the railway **turnout** (points; Czech *výhybka*), PC prefix `railway_switch_`. A blade swinging between two rest positions at 0.004 rad/ms over 0.5 rad, showing one of two alternative rail-piece frames per position and, at each end-stop, **rewiring the navigation graph**: it disconnects both `C_web_path` edges from a common node and reconnects exactly one, after which the tram recomputes its route. It publishes an `S_using_object`, so the player can throw it from the action menu. `structs/railway_switch.md` |
| 0x19 | `C_pumper` | `pumper_ctor` `0x004ce000` · `0x0063ca58` · - | the gas-station attendant. Walks out to a stopped car, refuels it and **charges 500** (`pumper_end_pump_fuel` → `game_freeride_score_set(score − 500)`); smokes and fidgets at his post in between (`pumper_get_special_anim_name`'s `Kurak*` / `Nuda*` clips). Reached from `player_do_action_button_in_car`; refuses a driver the police are after (`pumper_can_pump` → `police_manager_are_cops_after_player`). Shooting or ramming him sets action `0x11` = panic. Derives directly from `C_actor`; embeds `C_animations_machine_de_luxe` at `+0xe0`. `structs/pumper.md` |
| 0x1a | `C_crash_obj` | `crash_obj_ctor` `0x004786c0` · `0x0063bf68` · `0x3e4` | the breakable scene prop, one of `C_PhThing`'s five subclasses. Never built by the entity factory: the only construction path is `crash_obj_create_actor` via `crash_obj_create_from_model`, whose four callers are a car crash (`car_on_body_surface_contact`), a punch (`human_ai`) and a bullet (`game_new_shoot`). Everything it does is configured by single letters in the frame's user-property tag (`G` glass, `D` debris, `F` fire, `S` flash, `E` explosion, `+` break again, `X<f>` health, `M<f>` mass, `N<i>` material id…). Live pieces are counted in `g_crash_obj_count` and culled past 20 by `game_tick`. `structs/crash_obj.md` |
| 0x1b | `C_entity` | `entity_ctor` `0x004114d0` · `g_entity_vt` · `0x128c` | the scriptable NPC actor and police pursuit target — §4 onwards |
| 0x1c | `C_racecam` | - · `0x0063cb00` · - | the racing camera actor: it holds one `0x18`-byte `G_RaceCamera` record at `+0x70` (frame, 70.0f, 70.0f, 1.0f, 0, 0) and registers it with the `G_Camera` singleton at `C_game+0x4c` for the duration of the scene. No `AI`, no `Update`, no `Collision`, no `Hit`. **In retail it never registers**: `racecam_game_init` gates on `g_racing != 0` and nothing constructs a `C_racing`, because the racing mode's only entry point is the main-menu item the shipped executable deletes. TU `[0x004d33d0, 0x004d35c0)`, 9 starts. `structs/racecam.md`; the mode itself in `basic/artifacts/racing.md` |
| 0x1d | `bendable_plant` (name ours) | `bendable_plant_ctor` `0x004af030` · `0x0063c608` · `0xa0` | a clump of props that bend when you walk through them. One instance clones up to **7** child `OBJ_` prototypes of its own frame, `count` times each (counts at `+0x84..+0x9c`), scatters them by uniform disc sampling within the radius at `+0x80`, ground-snaps each with a vertical raycast and publishes it as a dynamic-collision record of type **`0x47`**, radius `0.4f`. `bendable_plant_ai` runs a Verlet damped spring per clone (damping `0.93`, restore `normalize(-tilt) * 20.0 * stiffness`, tilt Y pinned to `1.0`), and the `g_collision` `case 0x47` arm — gated on this kind byte through the record's owner back-pointer — pushes a clone along the query direction without reporting a hit, so they never block anything. **PC-only** and **no construction path located**: the ctor's only caller is the entity factory's `case 0x1d`, and `0x1d` appears in none of the 79 shipped `scene2.bin` (§9). `structs/bendable_plant.md` |
| 0x1e | `C_wagon` | - | the scriptable mission wagon prop (MISE15-PRISTAV), **not** a tram (§3) |
| 0x1f | `C_irenka` | - · `0x0063c568` · - | the police **spike strip** — an invisible deployable line-segment collision barrier (asset `barrier.4ds`; the police-side name is "Iris", and the pool record type is `C_shvestky::S_road_block_iris`). PC prefix `spike_strip_*`; extends `C_actor`. `GameInit` builds the dynamic-collision object at `+0x74` and the bound volume, `spike_strip_activate` commits the placement and inserts the collision object, `ZapichameSi` tests segment crossing. Spawned and freed by the police roadblock temp-actor pool. TU `[0x004ae830, 0x004af000)`. `structs/spike_strip.md` |
| 0x20 | `C_public_physics` | `public_physics_ctor` `0x004cde20` · `0x0063c9b8` · `0x3b0` | the `create_physicalobject` prop: the general-purpose `C_PhThing` subclass that turns an existing scene frame into a free-falling physics prop, where its four siblings are each specialised to one kind of debris. Five creators: `human_free_hand`, `pedestrian_drop_umbrella`, `human_start_weapon_effect`, dispatcher case `0xcc`, and the entity factory. `structs/public_physics.md` |
| 0x22 | `C_clock` | `clock_ctor` `0x00477330` · `0x0063beb8` · `0x88` | a working clock actor: resolves the child frames `Mala` / `Velka` / `Sekundova` (*malá / velká / sekundová ručička* = hour / minute / second hand) and drives them from the machine's wall clock via `GetLocalTime`. Only three shipped models carry the hand frames — `9hodinky.4ds`, `xhodiny 4.4ds`, `rucky.4ds`. `structs/clock.md` |
| 0x23 | `C_box_new` | `0x0045f160` · `0x0063baa0` · - | a `C_PhThing` subclass (its ctor chains `physical_object_ctor`, exactly as the PS2 ctor chains `__ct__9C_PhThingFv`). `box_new_init` alone shares five magic constants in order with its twin. `structs/box_new.md` |
| 0x25 | `C_drop_in_weapon` | `drop_in_weapon_ctor` `0x0048a0d0` · `0x0063c280` · `0x3f8` | **an item dropped into the world**, not specifically a weapon: the four creation sites are `human_drop_out_items` (a dying human's guns), `human_do_box_drop` (a carried crate), `dog_spawn_severed_head` and `program_process_command`. Its job is to publish an `S_using_object` record — embedded at `+0x3c0`, or heap-built when the item appears mid-scene — into the registry behind the player's action menu. `+0x3b0` is a `predmety.def` item id; id 0 takes the other arm and becomes a static collision box instead. `structs/drop_in_weapon.md` |
| 0x26 | `C_rail_generator` | - | rail-vehicle spawner |

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
| `Traffic_person` | 0x12 | the ambient pedestrian ¹ |
| `Traffic_car` | 0xc | `C_traffic_car` |
| `Bridge` | 0x14 | `C_bridge` |
| `Dog` | 0x15 | `C_dog` |
| `Airplane` | 0x16 | `C_airplane` |
| `Turnout` | 0x18 | `C_turnout` |
| `Pumper` | 0x19 | `C_pumper` |
| `Race camera` | 0x1c | `C_racecam` |

¹ Kind `0x12` is the `C_traffic_generator` spawner in §1; no other source names `0x12` for the
pedestrian side (§9).

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
- Supporting: `C_rail_generator` (type 0x26) spawns rail traffic; `C_turnout` (type 0x18) is the
  track switch.
- **Radar representation.** `railway_update` (`0x004da400`) puts rail vehicles on the HUD radar
  through the **object** list, not the vehicle list: on the in-range transition (state field
  `+0x2a8 == 2`) it calls `hud_radar_add_object_with_color(part, 0xffffff00)` — **yellow** — once for
  the engine and once per wagon (`+0xe8 + i*0x48`, count at `+0x1a8`), and removes them all again on
  the out-of-range transition (state == 1). The whole registration is gated on the byte at
  `railway+0x1d8`. That is *why* trams have their own radar colour: a rail vehicle is a
  `tDynamicCollObject` entry in `hud_state+0x94a8` and can never appear in the `C_car` list at
  `+0x9494`. User-observed in game ("trams are yellow"), then byte-confirmed.
  See `structs/hud.md` §3 and `basic/artifacts/hud_elements.md` §6.

Script evidence: `wagon_*` commands appear only around `MISE15-PRISTAV`'s `aktivace vagonu*` scripts;
trams are ambient (e.g. mission `MISE08-HOTEL`, scripts `tram1` / `tram2`, which just `frm_seton` a
frame on a random timer — the `C_railway` traffic system, no `wagon_*`).

### 3a. The FLAVOUR field `+0x14` — a second, `C_actor`-level discriminator

`+0x10` is the kind. **`+0x14` is a separate enum written by the same constructors**, and it exists
because a subclass can share its base's kind:

| `+0x14` | class | written by |
|---|---|---|
| 1 | `C_actor` | `__ct__7C_actorFv` (which also writes kind 1) |
| 2 | `C_human` | `__ct__7C_humanFv` (which also writes kind 3) |
| 3 | **`C_car_ex`** | `__ct__8C_car_exFv` — and it writes **no kind at all**, inheriting 4 from `C_car` |

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

---

# The `C_entity` class (kind 0x1b) — scriptable-NPC AI brain

`C_entity` derives from `C_human` (kind `0x1b` over `C_human`'s 3). It is the scriptable-NPC
**AI/perception brain**: it owns a `C_program` (the script VM), perception logs / sensors /
attention, target selection, weapon-choice AI, the action-tick state machine, and the per-subsystem
SaveGame chain. **PC prefix: `entity_`.**

- **kind value:** `+0x10 = 0x1b`, stamped by `entity_ctor` over `C_human`'s 3.
- **PC vtable:** `g_entity_vt`.
- **Lifecycle:** `entity_ctor` (`0x004114d0`), `entity_dtor` (`0x00411640`),
  `entity_scalar_deleting_dtor` (`0x00411620`).

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 4. `C_entity` field layout (AI band; relative to the object base)

C_entity-specific fields; the object also carries the full `C_actor` + `C_human` bands
(`structs/actor.md`, `structs/human.md`). PC offsets. The PS2 offsets and the band deltas are in §7.

```c
typedef struct C_entity {                   // extends C_human; PC offsets, C_entity band only
    /* +0x78  */ uint32_t act_state;        // [H] act/anim state. entity_force_script does
                                            //     `if (+0x78 == 3) human_do_shock_up()`.
    /* +0x46c */ uint32_t script_cb_flags;  // [H] script-callback flag word. entity_script_callback:
                                            //     cb_action 0 => |= 0x200; 1 => &= 0xfffff1ff
                                            //     (`mov eax,[ecx+0x46c]; and ah,0xf1`).  See 4b.
    /* +0xa9c */ struct C_program program;  // [H] THE SCRIPT VM sub-object. See 4c.
    /* +0xafc */ float    faded;            // [H] visibility/"faded" fraction. entity_force_ai BAILS
                                            //     when this is >= 0.75 (the player cannot see the
                                            //     NPC); entity_script_is_fake clears it to 0.
                                            //     = program+0x60.
    // ---- THE SAVEGAME SUB-BLOCKS ---------------------------------------------------------
    // C_entity's savegame is: C_human parent + C_program + SIX sub-blocks. Each sub-block has a
    // Get<X>Size / Save<X> / Load<X> triple, and `entity_save_game_get_size` SUMS them - which is
    // how the whole chain is proven: a size function names its own members.
    /* +0xb04 */ struct ai_vis_logs vis_logs;  // [H] the ai_vis_logs SUB-OBJECT BASE.
                                            //     Byte-proven: `lea ecx,[esi+0xb04]` at all 19
                                            //     entity_ai callsites. Ticked by entity_ai.
    /* +0xb14 */ /* vis_logs list */        // [H] = vis_logs + 0x10. entity_save_vis_logs also
                                            //     touches +0xba4. Serialized size: CONSTANT 0x184.
    /* +0xbb4 */ struct ai_sensors sensors; // [H] the ai_sensors SUB-OBJECT BASE. Byte-proven:
                                            //     `lea ecx,[ebp+0xbb4]`. THE THREE-SENSE CONTACT
                                            //     SYSTEM (visual/sound/smell) - structs/ai_sensors.md
    /* +0xbc0 */ /* sensors serialized */   // [H] = sensors + 0xc, where serialization BEGINS (not
                                            //     the object base). Serialized size: CONSTANT 0x3a0.
    /* +0xc74 */ struct { uint32_t t; uint32_t type; } heard_events[]; // [H] SOUND-CONTACT RING,
                                            //     stride 0x20, = ai_sensors + 0xc0. entry[0] =
                                            //     timestamp, entry[1] = type. **TYPE 2 = GUNFIRE.**
                                            //     entity_has_heard_gunfire scans it for a type-2
                                            //     entry still inside its deadline (t + window > now).
                                            //     Hearing IS a sensor - it lies inside that band.
    /* +0xe88 */ struct ai_logs   ai_logs;  // [H] the ai_logs SUB-OBJECT BASE. Byte-proven:
                                            //     `lea ecx,[esi+0xe88]` at all 19 entity_ai
                                            //     callsites. Ticked by entity_ai.
    /* +0xeac */ /* ai_logs list */         // [H] = ai_logs + 0x24. The AI LOGS the AI scores
                                            //     targets from. Serialized size VARIABLE (a list).
    /* +0xebc */ struct ai_character character; // [H] the ai_character SUB-OBJECT BASE, size 0x28.
                                            //     Byte-proven: `add ecx,0xebc` in both C_entity
                                            //     wrappers. THE NPC PERSONALITY/STAT BLOCK (morale,
                                            //     intelligence, aggression, sight, hearing) -
                                            //     structs/ai_character.md. Serialized size 0x24 =
                                            //     its NINE stat deltas; the +0x24 cfg backpointer is
                                            //     a live pointer and is NOT saved.
    // The remaining two sub-blocks are fields documented below:
    //     "State"     = the blocked_con vector at +0x119c/+0x11a0 plus the substate-lock table
    //     "ActionMan" = the ai_action_manager at +0x1164 (entity_get_action_man_size reads +0x1168)
    /* +0xf08 */ uint32_t entity_flags;     // [H] the entity's main flag word.
                                            //     bit 0x200/0x400 = entity_script_switch;
                                            //     bit 0x4000 = ai_sensors_get_smelled_target_info
                                            //     picks LIVE vs CACHED smell;
                                            //     bit 0x20000 = latched by entity_suspend_ai_and_hide
                                            //     so the resume half knows to undo the hide.
    /* +0xf1c */ struct ai_modifier anger;  // [H] {value, rate_per_ms, remaining_ms}, stride 0xc
    /* +0xf28 */ struct ai_modifier fear;   // [H] same triple
    /* +0xf34 */ struct ai_modifier threat; // [H] same triple
    /* +0xf40 */ uint32_t ai_state;         // [H] the entity's AI state enum. Gate in
                                            //     entity_force_script: `+0xf40 != 0 ||
                                            //     !force_if_idle`; the value 4 is special-cased.
    /* +0xf48 */ uint32_t flags;            // [H] flag word. entity_death sets bit 3 (`|= 8`).
    /* +0xf4c */ uint32_t ai_substate;      // [M] sub-state/role. Tested `== 1` by
                                            //     entity_force_script and entity_force_ai
                                            //     (arrest-sound gate).
    /* +0xf50 */ uint32_t ai_flags;         // [M] bit 0 tested by entity_force_script;
                                            //     bit 1 (0x2) = STATE-SWITCHER SUSPENDED [H] -
                                            //     while set, entity_switch_ai_state returns
                                            //     immediately, so the AI state, mode (+0xf4c) and
                                            //     target slots (+0x127c/+0x1280) stay frozen.
                                            //     Sole writer: entity_set_state_switcher_suspended.
    /* +0xf64 */ struct ai_fight_modes fight_modes; // [H] the ai_fight_modes SUB-OBJECT BASE.
                                            //     Byte-proven two ways: `lea ecx,[esi+0xf64]` at
                                            //     four set_enemy/reset callsites, and
                                            //     `mov ecx,[edi+0x30]` + `add ecx,0xf64` at the
                                            //     apply_hit callsite inside ai_logs_event (edi+0x30
                                            //     = the ai_logs owner backpointer, i.e. this).
                                            //     THE COMBAT-MODE SCORER - 6 slots, stride 0x34;
                                            //     structs/ai_fight_modes.md
    // ---- THE SUBSTATE-LOCK TABLE (+0x10b8..+0x10c8) -----------------------------------------
    /* +0x10b8 */ struct ai_substate_lock *lock_vec_begin; // [H] vector<ai_substate_lock>, stride
                                            //     0xc, rows {key, p1 (uint), p2 (float)}.
                                            //     Writers: entity_lock_substate /
                                            //     entity_unlock_substate. Readers:
                                            //     entity_has_locked_substate, entity_get_state_size,
                                            //     entity_save_state / entity_load_state.
    /* +0x10bc */ struct ai_substate_lock *lock_vec_end;   // [H]
    /* +0x10c4 */ uint32_t locked_state;    // [H] the LOCKED STATE. **0x29a (= 666) means NO LOCK.**
                                            //     entity_has_locked_state is `!= 0x29a`;
                                            //     entity_lock_substate commits `key` here when its
                                            //     trailing bool is set. 666 is the project-wide
                                            //     "none" sentinel and appears in both int and float
                                            //     form - ai_vis_logs_init seeds slots with
                                            //     0x44268000 (= 666.0f) and sets its own +0xa8 to
                                            //     0x29a.
    /* +0x10c8 */ uint32_t lock_aux;        // [M] written by entity_lock_substate from its `aux`
                                            //     param. Role not pinned.
    /* +0x1164 */ struct ai_action_manager actions; // [H] the ACTION QUEUE sub-object - byte-proven
                                            //     (`lea ecx,[reg+0x1164]` at 12+ callsites). Three
                                            //     ai_action* lists + a session counter;
                                            //     structs/ai_action_manager.md
    /* +0x119c */ struct blocked_con *bc_begin; // [H] std::vector<blocked_con> - nav-graph edges
                                            //     this entity may not traverse. Stride 0x10; row[2]
                                            //     carries the 0x29a (= 666) "none" sentinel.
                                            //     Writer: entity_add_blocked_con (dedup-on-insert).
    /* +0x11a0 */ struct blocked_con *bc_end;   // [H]
    // ---- HEAD / PERCEPTION-ORIGIN BAND (+0x11b4..+0x11cb) -----------------------------------
    // The two vectors perception is computed FROM, read by ai_vis_logs_tick together with
    // entity_get_sight_range.
    /* +0x11b4 */ float    head_dir[3];     // [H] head/look DIRECTION (entity_get_head_dir)
    /* +0x11c0 */ float    head_pos[3];     // [H] head POSITION - the eye/ear point perception is
                                            //     measured from (entity_get_head_pos)
    /* +0x11cc */ uint32_t neck_bone;       // [H] the cached neck-bone handle
                                            //     entity_rebind_neck_bone_handle refreshes after a
                                            //     model swap; 0xffff = none.
    // ---- WEAPON-AI BAND (+0x11d0..+0x11ec) --------------------------------------------------
    // Cached copy of the HELD weapon's stats, refreshed by entity_weapon_get_info from the
    // weapon-definition record, so the AI can read weapon params every tick without re-walking
    // the item table.
    /* +0x11d0 */ float    weapon_def_param;  // [M] from weapon_def+0x6c; role not pinned
    /* +0x11d4 */ float    weapon_max_range;  // [H] from weapon_def+0x74; entity_weapon_get_max_range
    /* +0x11d8 */ float    weapon_best_range; // [H] from weapon_def+0x70; entity_weapon_get_best_range
    /* +0x11dc */ float    weapon_spread;     // [H] from weapon_def+0x88; entity_weapon_get_spread
    /* +0x11e0 */ float    weapon_readiness;  // [H] 0->1 fraction; reset to 0 on weapon change;
                                              //     ramped by entity_weapon_choose
    /* +0x11e8 */ uint32_t weapon_state;      // [H] weapon-state FSM, values 1..6
                                              //     (entity_equip_weapon_by_id); state 4 raises
                                              //     weapon_is_armed
    /* +0x11ec */ uint8_t  weapon_is_armed;   // [H] 1 = a weapon is drawn; entity_weapon_is_armed
    // ---- ai_ghost OBSTACLE MEMORY (+0x1268) -------------------------------------------------
    // Head = { +0x00 allocator, +0x04 begin, +0x08 end, +0x0c cap, +0x10 changed-flag byte }.
    // Records are ai_ghost_actor, stride 0x1c:
    //   { vec3 pos; uint32 expiry @+0x0c; float drift_radius2 @+0x10; C_actor* @+0x14;
    //     tDynamicCollObject* @+0x18 }.
    // ai_path_tick / ai_path_revalidate_for_actor record one per collidable object met on a path
    // segment, with type-dependent expiry (500/800/2000 ms) and radius2 (0.09/0.25/1.0);
    // entity_expire_ghost_trail walks the vector BACKWARDS once per frame and shift-down-erases
    // every record expired by time OR drift.
    /* +0x1268 */ struct ai_ghost ghost;    // [H] the object base (structs/ai_ghost.md)
    /* +0x126c */ void  *ghost_begin;       // [H] = ghost + 0x04
    /* +0x1270 */ void  *ghost_end;         // [H]
    /* +0x127c */ void  *ai_target_fear;    // [H] best-by-fear target slot (ai_logs_get_max_fear)
    /* +0x1280 */ void  *ai_target_angry;   // [H] best-by-anger target slot (ai_logs_get_max_angry)
} C_entity;                                 // partial - C_entity AI band only; instance 0x128c
```

The three emotion ramps at `+0xf1c` / `+0xf28` / `+0xf34` are three embedded **`ai_modifier`**
instances, stride `0xc`; there is no modifier *list*. `ai_modifier_tick` (`0x00425850`) does
`value += min(dt, remaining) * rate; remaining -= min(dt, remaining)`. `entity_ai` inlines the ANGER
ramp and calls `ai_modifier_tick` for the other two. The `ai_logs` target scorer multiplies its
derived anger (`log+0x4c`) by `+0xf1c`, its derived fear (`log+0x50`) by `+0xf28`, and scales its
threat term by `+0xf34` — so a script or event can push a temporary emotional multiplier with a decay
rate and a time budget, and it ramps back on its own.

## 4b. The two script flag words, `+0x46c` and `+0xf08`

PC keeps **two** flag words where the entity's script side is concerned:

| word | set by | read by | bits |
|---|---|---|---|
| `+0x46c` | `entity_script_callback` | - | `0x200` set on cb_action 0; mask `0xfffff1ff` on cb_action 1 |
| `+0xf08` | `entity_script_switch` (`\|= 0x400`), `entity_suspend_ai_and_hide` (`\|= 0x20000`) | `entity_script_switch` (`& 0x200`), `ai_sensors_get_smelled_target_info` (`& 0x4000`), `entity_resume_ai_and_show` | see §4 |

`entity_script_switch` tests bit `0x200` of `+0xf08`, while the only located `0x200` **setter**
writes `+0x46c`. Which PC code sets bit `0x200` on `+0xf08` — or keeps the two words in sync — is
open (§9). The cross-build reading of the same pair is in §7.

## 4c. The `C_program` sub-object

The script VM is a `C_program` sub-object embedded at **`+0xa9c`**, proven twice from the PC side:
`entity_death` calls its vtable through `*(this+0xa9c)`, and `entity_script_is_fake` reads
`+0xaac`/`+0xab0` = `program+0x10`/`+0x14`, the instruction vector.

PC `C_program` internals as this class uses them:

| item | PC offset (in the program) | note |
|---|---|---|
| instruction list | `begin +0x10`, `end +0x14` | the count is **computed** as `(end − begin) >> 3`, stride 8 |
| pc | `+0x1c` | |
| cleared on the fake-script path | `+0x60` | = entity `+0xafc`, the `faded` fraction |

`C_entity` **overrides three `C_program` virtuals** — `DecodeParams` / `DecodeInstruction` /
`DecompileCommand` — plus the savegame trio, which is why the entity carries adjustor thunks
(`program_base_thunk_entity_*`, `0x0043b030`..`0x0043b060`) that convert a `C_program *` receiver back
to a `C_entity *`.

**`entity_get_program` (`0x004258b0`) returns `this + 0xaa8`, i.e. `program + 0xc`.** A callsite that
does `GetProgram()->field` is indexing from `+0xc`; subtract `0xc` before comparing against the table
above. `+0xa9c` is the base — never "correct" it to `+0xaa8`. The getter has **0 direct `E8`
callsites** (pure vtable dispatch); it was found through its single `.rdata` data reference, C_entity
vtable **slot 35** (`0x0063b514`), whose neighbours identify it: the slot before holds
`human_get_bound_points_2d` (GetBoundPoints2D) and the slot after `human_cb_coll_tester`
(cbCollTester), exactly `GetProgram`'s position in the `C_actor` virtual order; three slots earlier
the shared `return 0` stub `stub_ret0_args4_00425420` fills the `GetMaterialID` position.

What lives at `program + 0xc` — which base or interface subobject the getter's return type is — is
open (§9). The cross-build comparison of the skew is in §7.

## 5. Field evidence — C_entity AI band

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x78 | act_state | H | `entity_clear_act_state` (=0, `0x004257a0`), `entity_set_act_state_shock` (=3, `0x004257b0`) | `entity_force_script` | `+0x78 == 3` ⇒ `human_do_shock_up()` |
| +0x46c | script_cb_flags | H | `entity_script_callback` | - | §4b |
| +0xafc | faded | H | `entity_script_is_fake` (clears) | `entity_force_ai`, `entity_script_is_fake` | `entity_force_ai` bails when ≥ 0.75 (the player cannot see the NPC) |
| **+0xb04** | **`ai_vis_logs` sub-object** | H | `ai_vis_logs_tick` | `entity_ai` | the OBJECT BASE (`lea ecx,[esi+0xb04]` at all 19 `entity_ai` callsites); `+0xb14` is its internal list. Address accessor `entity_get_ai_vis_logs` (`0x00425b70`) |
| +0xb14 (+0xba4) | vis_logs (list) | H | `entity_save_vis_logs` | `entity_load_vis_logs` | = `ai_vis_logs + 0x10`. Serialized size = constant 0x184 |
| **+0xbb4** | **`ai_sensors` sub-object** | H | `ai_sensors_new_sound_contact` | `sound_arena_update_entity_sound_contacts` | the OBJECT BASE (`lea ecx,[ebp+0xbb4]`). The three-sense contact system: `structs/ai_sensors.md`. Address accessor `entity_get_ai_sensors` (`0x00425b80`) |
| +0xbc0 | sensors (serialized start) | H | `entity_save_sensors` | `entity_load_sensors` | = `ai_sensors + 0xc`, where serialization begins — **not** the object base. Serialized size = constant 0x3a0 |
| **+0xc74** | **heard_events[]** (= `ai_sensors + 0xc0`) | H | `ai_sensors_new_sound_contact` (`0x0040faf0`), driven by `sound_arena_update_entity_sound_contacts`; killed by `ai_sensors_cancel_sound_contact` | `entity_has_heard_gunfire` | ring of sound CONTACTS, stride 0x20: `entry[0]` = timestamp, `entry[1]` = type, **type 2 = GUNFIRE**. Audibility is gated by the NPC's hearing stat (`intensity > 1.05 − entity_get_hearing_sensitivity()`), not a fixed radius. Cancelling ZEROES the type field at `+0xc78`, which is what makes the type-2 scan miss it. Full chain: `structs/ai_sensors.md` §3 |
| **+0xe88** | **`ai_logs` sub-object** | H | `ai_logs_tick` | `entity_ai` | the OBJECT BASE (`lea ecx,[esi+0xe88]` at all 19 `entity_ai` callsites); `+0xeac` is its internal list (`ai_logs + 0x24`). Address accessor `entity_get_ai_logs` (`0x00425b90`) |
| +0xeac | ai_logs (list) | H | `entity_save_logs` | `entity_load_logs`, `ai_logs_get_max_*`, `ai_logs_get_log_for_actor` | the AI logs the AI scores targets from. Serialized size is VARIABLE (a list) |
| **+0xebc** | **`ai_character` sub-object** | H | `ai_character_init` | all `ai_character_get_*`, `entity_get_sight_range`, `entity_get_hearing_sensitivity` | the OBJECT BASE (`add ecx,0xebc`), size 0x28. The NPC personality/stat block: `structs/ai_character.md` |
| +0xebc..+0xec8 | character (serialized) | H | `entity_save_character` | `entity_load_character` | the nine stat deltas. Serialized size = constant 0x24: `ai_character_init` zeroes exactly nine dwords (`+0x00..+0x20`) and stores the cfg backpointer at `+0x24`, which is a live pointer and is deliberately not saved |
| +0xf08 | entity_flags | H | `entity_script_switch`, `entity_suspend_ai_and_hide` | `entity_script_switch`, `ai_sensors_get_smelled_target_info`, `entity_resume_ai_and_show` | §4b |
| +0xf1c / +0xf20 / +0xf24 | **ANGER ramp** `{value, rate_per_ms, remaining_ms}` | H | `entity_ai` (seeds `+0xf1c = 1.0f`, `+0xf20 = 0`, and open-codes the advance on this slot) | the `ai_logs` target scorer, which multiplies its derived anger (`log+0x4c`) by `+0xf1c` | an embedded `ai_modifier` |
| +0xf28 / +0xf2c / +0xf30 | **FEAR ramp** (same triple) | H | `entity_ai` via `ai_modifier_tick` | the scorer multiplies its derived fear (`log+0x50`) by `+0xf28` | an embedded `ai_modifier` |
| +0xf34 / +0xf38 / +0xf3c | **THREAT ramp** (same triple) | H | `entity_ai` via `ai_modifier_tick` | the scorer scales its threat term by `+0xf34` | an embedded `ai_modifier`; `ai_modifier_tick` (`0x00425850`) is class-less, and these three slots are its only known users |
| +0xf40 | ai_state | H | `entity_force_ai` | `entity_force_script` | the AI state enum; value **4** is special-cased |
| +0xf48 | flags | H | `entity_death` (sets bit 3) | - | flag word |
| +0xf4c | ai_substate | M | - | `entity_force_script`, `entity_force_ai`; getter `entity_get_ai_substate` (`0x00425b60`) | tested `== 1` (arrest-sound gate) |
| +0xf50 | ai_flags | M | `entity_set_state_switcher_suspended` (bit 0x2) | `entity_force_script` (bit 0x1), `entity_switch_ai_state` (bit 0x2) | bit 1 (0x2) = state-switcher suspended — `entity_switch_ai_state` early-returns while it is set, freezing the AI state machine. Remaining bits not pinned |
| **+0xf64** | **`ai_fight_modes` sub-object** | H | `ai_fight_modes_set_enemy`, `ai_fight_modes_reset`, `ai_fight_modes_apply_hit` | `ai_fight_modes_get_hit_stat`, `ai_fight_modes_consume_finish_request`, `entity_get_ai_fight_modes` (address accessor, `0x00425ba0`) | the OBJECT BASE, byte-proven two independent ways (§4). The 6-slot combat-mode scorer (stride 0x34): `structs/ai_fight_modes.md` |
| +0x10b8..+0x10bc | **substate-lock vector** | H | `entity_lock_substate`, `entity_unlock_substate` | `entity_has_locked_substate`, `entity_get_state_size`, `entity_save_state`, `entity_load_state` | `std::vector<ai_substate_lock>`, stride 0xc, rows `{key, p1 (uint), p2 (float)}` — PS2's `insert__…<ai_substate_lock>` template argument names the row type. `entity_get_state_size` sums `nlocks * 0xc` into the savegame "State" block, which confirms the member independently |
| **+0x10c4** | **locked_state** | H | `entity_lock_substate` (when its trailing bool is set) | `entity_has_locked_state` | **0x29a (= 666) = NO LOCK**, the project-wide "none" sentinel, in both int and float form (`ai_vis_logs_init` uses 0x44268000 = 666.0f). Generic substate locking, not pursuit-specific |
| +0x10c8 | lock_aux | M | `entity_lock_substate` | - | written from the `aux` param; role not pinned |
| **+0x1164** | **`ai_action_manager` sub-object** | H | `ai_action_manager_clear_all`, `ai_action_manager_new_job` | `ai_action_manager_get_base_job`, `ai_action_manager_has_action`, `entity_get_action_man_size` | the OBJECT BASE (`lea ecx,[reg+0x1164]` at 12+ callsites). Three `ai_action*` lists — jobs `+0x04`, idles `+0x14`, modifiers `+0x24` — plus the action-UID counter at `+0x30`: `structs/ai_action_manager.md` |
| +0x119c..+0x11a0 | **`vector<blocked_con>`** | H | `entity_add_blocked_con` | `entity_switch_ai_state`, `entity_save_state` | blocked connections — nav-graph edges the entity may not use. PS2's signature names the element type outright (`AddBlockedCon__8C_entityFP11blocked_con`), and `Tick__11ai_modifier` calls the adder ×20, which fits path edges. Stride 0x10; `row[2]` carries the 0x29a (= 666) sentinel |
| **+0x1268..+0x1278** | **`ai_ghost` (obstacle memory)** | H | `ai_ghost_add_or_refresh`, `ai_ghost_add_human_colliders` | `entity_expire_ghost_trail`, `ai_ghost_invalidate_actor` | the object base; `+0x126c` is the vector's begin pointer. Records are `ai_ghost_actor`, stride **0x1c**. Written by `ai_path_tick` / `ai_path_revalidate_for_actor`, one per collidable object met on a path segment, with type-dependent expiry (500/800/2000 ms) and radius² (0.09/0.25/1.0); expired per frame by time OR drift. `structs/ai_ghost.md` |
| +0x11cc | neck_bone | H | `entity_rebind_neck_bone_handle` | the pose code | cached handle, `0xffff` = none; refreshed after a model swap |
| +0x11d0 | weapon_def_param | M | `entity_weapon_get_info` | - | copied from weapon_def+0x6c; role not pinned |
| +0x11d4 | weapon_max_range | H | `entity_weapon_get_info` | `entity_weapon_get_max_range` | from weapon_def+0x74. Consumers compute `radius² − dist² > 0` ⇒ target in range, so it stores a LINEAR range, squared at the callsite |
| +0x11d8 | weapon_best_range | H | `entity_weapon_get_info` | `entity_weapon_get_best_range` | from weapon_def+0x70 |
| +0x11dc | weapon_spread | H | `entity_weapon_get_info` | `entity_weapon_get_spread`, `entity_can_safe_shoot` | from weapon_def+0x88; feeds the safe-shot cone `cos(spread*0.0002546479 + 0.17)` |
| +0x11e0 | weapon_readiness | H | `entity_weapon_choose`, `entity_weapon_get_info` | `entity_weapon_get_readiness` | reset to 0 on weapon change; readers gate on `≤ 0.9999` = "not fully readied". `entity_weapon_choose` zeroes it while selecting, then writes an intermediate ramp float and finally `0x3f800000` (1.0f) once the choice settles |
| +0x11e8 | weapon_state | H | `entity_equip_weapon_by_id` | `entity_equip_weapon_by_id` | weapon-state FSM, values 1..6; reaching state 4 raises `weapon_is_armed` |
| +0x11ec | weapon_is_armed | H | `entity_weapon_get_info`, `entity_equip_weapon_by_id` | `entity_weapon_is_armed` | byte. Set iff weapon slot `+0x1e8` ∈ 1..3. It has no vision input |
| +0x127c / +0x1280 | ai_target_fear / _angry | H | `ai_logs_get_max_fear`, `ai_logs_get_max_angry` | `entity_switch_ai_state` | per-tick best-target picks |

## 6. `C_entity` method family — PC ↔ PS2 (110 PS2 methods)

Ordered by PC VA, PS2-only rows last (by PS2 VA). Status: **CONFIRMED** (body-diffed or otherwise
established twin) · **STRUCTURAL** (structure matches, identity not body-proven) · **base-stub**
(an 8-byte overridden virtual or `@3056@` MI adjustor thunk with no distinct PC twin) · **not
located**.

The PC cluster `[0x00410000, 0x00437000)` holds **216** true function starts against these 110
methods — a block of roughly 50 `vec2_*` / `vec3_*` / HUD / freeride helpers is emitted straight
through the middle of the C_entity code, and the `entity_dtor → entity_force_ai` gap alone is PC 85
against PS2 28. Emission order is nevertheless monotone between the builds.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004107d0 | `ai_vis_logs_tick` | `Tick__11ai_vis_logsFv` + `TickCorpses__11ai_vis_logsFv` FUSED | 0x0030fce0 / 0x00310c90 | CONFIRMED | §6c — a sub-object method, listed there |
| 0x004114d0 | `entity_ctor` | `__ct__8C_entityFv` | 0x00311300 | CONFIRMED | kind = 0x1b, vtable `0x0063b488` |
| 0x00411620 | `entity_scalar_deleting_dtor` | - | - | PC-only ¹ | MSVC construct |
| 0x00411640 | `entity_dtor` | `__dt__8C_entityFv` | 0x00311490 | CONFIRMED | base dtor |
| 0x004117a0 | `entity_init` | `Init__8C_entityFP9I3D_frame` | 0x00311650 | CONFIRMED | thin override forwarding to the `C_human` parent (`human_init` ×1 ↔ `Init__7C_human` ×1). **The parent call is the decisive class signal for this whole family** |
| 0x004117c0 | `entity_duplicate_from` | `DuplicateFrom__8C_entityFP7C_actor` | 0x00311670 | CONFIRMED | parent `human_duplicate_from`, then duplicates the script VM (`program_duplicate_from` ↔ `DuplicateProgramFrom__9C_program`) |
| 0x00411800 | `entity_game_init` | `GameInit__8C_entityFv` | 0x003116c0 | CONFIRMED | the real construction step: parent `human_game_init`, then inits the AI sub-objects (§6c) — C_program, `ai_vis_logs_init`, `ai_sensors_init`, `ai_character_init`, `ai_logs_init`. Two exact counts: `program_is_event_id` ×2; `rand_range` ×2 ↔ `rnd__Fi` ×2 |
| 0x00411e80 | `entity_reset` | `Reset__8C_entityFv` | 0x00311bd0 | CONFIRMED | the FULL per-entity AI reset — re-inits every AI sub-object (vis_logs / sensors / character / logs inits, `ai_sensors_destroy`, action-manager clear, `human_do_crouched`, `human_pose_null_rotate`) |
| 0x00411fb0 | `entity_reset_state` | `ResetState__8C_entityFv` | 0x00311ca0 | CONFIRMED | the LIGHT reset, a strict subset of `entity_reset`. Exact 3:3 callee match (`ai_sensors_destroy`, `ai_vis_logs_init`, action-manager clear). It destroys the sensors **without** re-initialising them (§9) |
| 0x00412060 | `entity_game_done` | `GameDone__8C_entityFv` | 0x00311d10 | CONFIRMED | teardown: action-manager clear, `ai_sensors_destroy` (frees the scent blobs), path-node release, then parent `human_game_done` |
| 0x00412100 | `entity_save_data` | `SaveData__8C_entityFP7C_chunk` | 0x00311de0 | CONFIRMED | writes the entity to a mission **.dta chunk** (not the savegame path). Exact 3:3: parent `human_save_data` + `dtaWrite` + `program_save_source_code` |
| 0x00412150 | `entity_load_data` | `LoadData__8C_entityFP7C_chunk` | 0x00311e60 | CONFIRMED | reads it back: parent `human_load_data` + `dtaSeek` + `dtaRead` + `program_load_source_code`. True extent 149 B |
| 0x004121f0 | `entity_collision` | `Collision__8C_entityF…` | 0x00311f30 | CONFIRMED | a 16-byte override forwarding to `human_collision`; C_entity adds nothing to collision handling |
| 0x00412200 | `entity_hit` | `Hit__8C_entityF10E_hit_type…` | 0x00311f40 | CONFIRMED | reaction to being hit: reset `ai_ghost`, parent `human_hit`, then read the attacker's KIND and special-case a CAR whose owner is the PLAYER or another `C_entity`; logs the attacker and drives the police response (sighting + gun/handcuffs escalation). Same kind gate, same constants **4 / 2 / 0x1b** in the same nesting order, plus `car_get_owner` ↔ `GetOwner__5C_car` |
| 0x00412410 | `entity_cb_hit_info` | `cbHitInfo__8C_entityF10E_hit_typefP7C_actor` | 0x00312240 | CONFIRMED | signature matches exactly: `(E_hit_type, float, C_actor*)` — a float in the middle slot is a strong discriminator |
| 0x00412480 | `entity_ai` | `AI__8C_entityFUi` | 0x00312300 | CONFIRMED | the **AI BRAIN** — the per-frame perceive→decide→act tick: perception + logs + event ring, target and weapon choice, then it builds an `ai_action` from the 35-strong factory family (§6b), installs it and drives the action queue; also runs the entity's script program (`program_call_process` ×12). 8560 B. **Reached ONLY by virtual dispatch — vtable slot 13**: the sole absolute reference to its VA is at `0x0063b4bc` inside `g_entity_vt`, which is why it has zero textual callsites. Callee multiset matches with two exact counts (`CallProcess` ×12, `WeaponChoose` ×7) |
| 0x004145f0 | `entity_action_tick` | `actionTick__8C_entityFP9ai_actionUiUiPUi` | 0x00315040 | CONFIRMED | **singular** — advances ONE `ai_action`, switching on its type across **40 outer cases** in `0x00..0x2a` (`0x0f` unused). Exact signature `this + (ai_action*, uint, uint, uint*)`; the outer case set equals `actionTick ∪ actionTick_2` exactly; sole caller ×4, mode multiset {0,1,2,2} on both |
| 0x004258b0 | `entity_get_program` | `GetProgram__8C_entityFv` | 0x0034b140 | CONFIRMED | a real virtual getter returning `this + 0xaa8` — §4c |
| 0x004258c0 | `entity_actions_tick` | `actionsTick__8C_entityFUi` | 0x0032eaa0 | CONFIRMED | **plural** — the per-frame driver that ticks all of the entity's actions and does the idle/modifier-list teardown. Sole caller of the singular tick; exactly 4 callsites; same mode multiset {0,1,2,2} on both builds |
| 0x00425bb0 | `entity_update` | `Update__8C_entityFUi` | 0x0032ef40 | CONFIRMED | the per-frame Update override: parent `human_update`, then `entity_release_target` + `police_board_invalidate_actor`. The parent-Update call is the decisive class signal |
| 0x00425cb0 | `entity_death` | `Death__8C_entityFv` | 0x0032f430 | CONFIRMED | **shared magic constant `0xea61`** on both builds in the same guard, plus the identical chain IsEventID → SetEvent → vtable → CallProcess → `ai_action` Clear → parent `human_death`. This diff exposed the whole `C_program` script-VM class on the PC side |
| 0x00425d30 | `entity_script_decode_params` | `DecodeParams__8C_entityFPcRQ29C_program13S_instruction` | 0x0032f580 | CONFIRMED | script VM; a C_entity **override** of the C_program virtual. Signature `(char*, S_instruction&)`; uses `__strcmpi` (the script language is case-insensitive), `__ftol`, `web_path_find_node_id` |
| 0x00427690 | `entity_script_decode_instruction` | `DecodeInstruction__8C_entityFPcRi` | 0x00332ab0 | CONFIRMED | script VM. 9744 B, with **0 direct `E8` callsites** — only ever reached through the vtable, and identified by slot position: its single `.rdata` reference is `0x0063b474`, the slot between `DecodeParams` and `Process` in C_entity's `C_program` base-subobject vtable (`0x0063b45c`..`0x0063b480`), mirroring the PS2 `@3056@` thunk address order. Both builds materialise the same ~4.3 KB command-descriptor table in a local stack frame; PC unrolls the copy, which is the whole size difference |
| 0x00429ca0 | `entity_script_format_command` | `DecompileCommand__8C_entityFiPc` | 0x00332c10 | CONFIRMED | script VM override; renders an instruction back to source text via `crt_sprintf` (`Enemy_UseCar_act(%i,…)`, `EntityMoveToFrame(%d,%f)`) |
| 0x0042a660 | `entity_process_command` | `Process__8C_entityFUi` | 0x003336b0 | CONFIRMED | the script-VM instruction processor. Opens with the MSVC SEH prologue `64 a1 00 00 00 00` after NOP padding |
| 0x0042de60 | `entity_script_tick` | `scriptTick__8C_entityFUi` | 0x00338df0 | CONFIRMED | the per-frame SCRIPT tick — the branch of the brain that runs under script control, against `entity_actions_tick` for the action queue. Gated on `+0x5d` and `movement_mode` (`+0x40c`) ∈ {1,2,9}; also feeds the blocked-con list. Exact ×10 ↔ ×10 callsites from the confirmed `entity_ai` twin, and the only ×10 callee on either side |
| 0x0042e530 | `entity_get_sight_range` | `GetVisibleRange__8C_entityFv` | 0x00339580 | CONFIRMED | **SIGHT RANGE in world units** = the embedded `ai_character`'s sight stat × **200.0** (`add ecx,0xebc; call ai_character_get_sight; fmul [0x63b5f4]`). Consumer-proven a range: `ai_vis_logs_tick` compares it to inter-entity distance (×4 on both builds) |
| 0x0042e550 | `entity_get_hearing_sensitivity` | `GetHearQuality__8C_entityFv` | 0x003395b0 | CONFIRMED | **HEARING SENSITIVITY** — forwards to the embedded `ai_character`. Consumer proves *sensitivity*, not "quality": `sound_arena_update_entity_sound_contacts` registers a contact iff loudness > `1.05 − this` ⇒ higher = hears quieter sounds |
| 0x0042e560 | `entity_get_head_dir` | `GetHeadDir__8C_entityFv` | 0x003395c0 | CONFIRMED | head/look DIRECTION vec3 @ `+0x11b4`. Identical sibling of the next row — separated by **order** (lower offset + earlier VA on both builds), not by body |
| 0x0042e590 | `entity_get_head_pos` | `GetHeadPos__8C_entityFv` | 0x003395d0 | CONFIRMED | head POSITION — the eye/ear point perception is measured from — vec3 @ `+0x11c0`. Identical sibling — separated by order (higher offset + later VA on both builds) |
| 0x0042e5c0 | `entity_script_switch` | `ScriptSwitch__8C_entityFv` | 0x003395e0 | CONFIRMED | ×2 ↔ ×2 from the confirmed `entity_ai` twin, identical nesting and **byte-identical flag logic** `if (f & 0x200) f \|= 0x400` on `+0xf08` (§4b) |
| 0x0042e650 | `entity_force_ai` | `ForceAI__8C_entityFUiUiUif` | 0x003396a0 | CONFIRMED | force an AI action |
| 0x0042e7f0 | `entity_script_is_fake` | `isFakeScript__8C_entityFv` | 0x003398e0 | CONFIRMED | reads the `C_program` instruction vector (`+0xaac`/`+0xab0` = `program+0x10`/`+0x14`) and returns 1 when the pc is past the end = the script is a placeholder. There is no lock involved |
| 0x0042e8a0 | `entity_force_script` | `ForceScript__8C_entityFbb` | 0x00339a00 | CONFIRMED | forces the entity back under script control, abandoning its current `ai_action`. Shared event constant `3`; calls `isFakeScript`; `Do_ShockUp ↔ human_do_shock_up` |
| 0x0042e990 | `entity_script_callback` | `CallBack__8C_entityFQ29C_program11E_cb_action` | 0x00339b50 | CONFIRMED | **byte-identical** — the same magic constants `0x200` / `0xfffff1ff` on the same enum values 0/1, on `+0x46c` (§4b) |
| 0x0042e9c0 | `entity_dispatch_event` | `Event__8C_entityFP7C_actorUiPv` | 0x00339c40 | CONFIRMED | event dispatch |
| 0x00432550 | `entity_weapon_choose` | `WeaponChoose__8C_entityFv` | 0x0033eba0 | CONFIRMED | pick a weapon (AI); also ramps `+0x11e0` |
| 0x00432c50 | `entity_equip_weapon_by_id` | `WeaponChoose__8C_entityFUi` | 0x0033f5b0 | CONFIRMED | the **ID-taking overload**: same "already-held ⇒ return 1" early-out through the inventory vector, same search, same return contract |
| 0x00432ea0 | `entity_weapon_get_max_range` | `WeaponGetMaxRange__8C_entityFv` | 0x0033f840 | CONFIRMED | getter for `+0x11d4`; consumer-proven a range |
| 0x00432eb0 | `entity_weapon_get_best_range` | `WeaponGetBestRange__8C_entityFv` | 0x0033f850 | CONFIRMED | getter for `+0x11d8` |
| 0x00432ec0 | `entity_weapon_get_spread` | `WeaponGetRozptyl__8C_entityFv` | 0x0033f860 | CONFIRMED | getter for `+0x11dc` (*rozptyl* = spread) |
| 0x00432ed0 | `entity_weapon_get_readiness` | `WeaponGetReady__8C_entityFv` | 0x0033f870 | CONFIRMED | getter for `+0x11e0`; readiness fraction 0→1 |
| 0x00432ee0 | `entity_weapon_is_armed` | `WeaponIsArmed__8C_entityFv` | 0x0033f880 | CONFIRMED | **byte** getter for `+0x11ec` (`undefined1` on both) |
| 0x00432ef0 | `entity_weapon_get_info` | `WeaponGetInfo__8C_entityFv` | 0x0033f890 | CONFIRMED | writes the whole weapon band from the weapon-def record |
| 0x00432f60 | `entity_do_shoot_timed` | `Do_Shoot_Timed__8C_entityFbP8S_vectorP7C_actor` | 0x0033f920 | CONFIRMED | timed shot (calls C_human `Do_Shoot`) |
| 0x00433030 | `entity_weapon_timing_setup` | `WeaponTimingSetup__8C_entityFUi` | 0x0033fa20 | CONFIRMED | weapon timing setup |
| 0x00433170 | `entity_release_target` | `InvalidateActor__8C_entityFP7C_actor` | 0x0033fb60 | CONFIRMED | invalidate a referenced actor |
| 0x00433290 | `entity_rebind_neck_bone_handle` | `InvalidateSelf__8C_entityFv` | 0x0033fcd0 | CONFIRMED | drops the cached neck-bone handle at `+0x11cc` and re-resolves it after a model swap. Its only caller is `human_change_model` behind `if (kind == 0x1b)`, and the PS2 twin's caller guards the identical statement position the same way. Same two string literals in order (`"Neck"`, `"neck"`), same `0xffff` sentinel, same retry-on-zero shape |
| 0x004332d0 | `entity_can_safe_shoot` | `CanSafeShoot__8C_entityFP7C_actorbb` | 0x0033fd50 | CONFIRMED | line-of-fire safety check |
| 0x00433910 | `entity_trace_human_scent` | `TraceHuman__8C_entityFP7C_humanP7ai_blob` | 0x00340570 | CONFIRMED | walks a human's **scent trail**: loops the target's `ai_blob_log` via `ai_blob_log_get_blob`, subtracting `blob.pos − frame.pos` per blob. Same frame fetch, same `& 0x20` staleness flag + `UpdateWMatrixProc`. The PS2 signature `(C_human*, ai_blob*)` is what identifies the loop subject as scent blobs. `structs/ai_blob_log.md` |
| 0x004339e0 | `entity_add_using_object` | `AddUsingObject__8C_entityFv` | 0x00340680 | CONFIRMED | registers the entity as a user of a `C_using_object`: script-event check, world-matrix refresh, dedup-ing push onto the user list. Reads its `+0x1230`..`+0x1258` band. Two exact callee matches (`program_is_event_id`, `UpdateWMatrixProc`) plus the field band and a forced position; its third callee pair is **refuted** (`using_object_add_object` is not `AddObject__14C_using_object`), so the row rests on two |
| 0x00433ab0 | `entity_switch_ai_state` | `StateSwitcher__8C_entityFv` | 0x00340740 | CONFIRMED | **the AI state machine.** Reads the strongest fear / anger / death / threat targets from `ai_logs`, checks the substate locks, then forces an AI state via `entity_force_ai` (×3) or returns control to the script; caches the target picks at `+0x127c`/`+0x1280`. Three exact call-count matches on independently confirmed twins |
| 0x004340c0 | `entity_spawn_police_response` | `SpawnPolice__8C_entityFP7C_humanUiUifUi` | 0x00341140 | CONFIRMED | builds the whole police AI response: forces AI states (×5), installs police / barricade-fight / car-jump actions, sets the fight-mode enemy, dispatches an entity event. **Seven exact call-counts.** It also **seeds a smell contact** (`ai_blob_log_get_blob` → `ai_sensors_build_smell_contact_from_blob` → `ai_sensors_new_smell_contact` → free) — the same chain `ai_sensors_cancel_visual_contact` runs on losing sight |
| 0x004342b0 | `entity_lock_substate` | `LockSubstate__8C_entityFUiUifUib` | 0x00341400 | CONFIRMED | upsert into the `std::vector<ai_substate_lock>` at `+0x10b8`; commits `key` to `+0x10c4` when its trailing bool is set. Same find→erase→insert shape; 6 params on both; PC's trailing `char` = PS2's `b`. PS2's `insert__…<ai_substate_lock>` template argument names the row type |
| 0x00434550 | `entity_unlock_substate` | `UnlockSubstate__8C_entityFUi` | 0x003414e0 | CONFIRMED | find the lock row for `key` and erase it. PC inlines the vector erase (shift-down + `end -= 0xc`); PS2 calls `erase__…<ai_substate_lock>` |
| 0x004345e0 | `entity_has_locked_state` | `hasLockedState__8C_entityFv` | 0x00341550 | CONFIRMED | **byte-identical**: `return *(this+0x10c4) != 0x29a` — the shared magic 0x29a (=666) "no lock" sentinel. Generic substate locking, not pursuit-specific |
| 0x004345f0 | `entity_has_locked_substate` | `hasLockedSubstate__8C_entityFUiPUiPf` | 0x00341560 | CONFIRMED | is substate `key` locked — and if so hand back its two payload values (both out-params NULL-checked). Identical: stride 0xc, key at row+0, outputs at row+4/+8, same 0/1 return. PS2's signature `(uint, uint*, float*)` types the payload |
| 0x00434680 | `entity_get_group_position` | `GetGroupPosition__8C_entityFP7C_actorP8S_vector` | 0x003415f0 | CONFIRMED | returns an actor's position **only if this entity can see it**. Exact 2:2: `ai_sensors_can_see` ↔ `isVisible__10ai_sensors` + `UpdateWMatrixProc` |
| 0x004346e0 | `entity_mem_write` | `MemWrite__8C_entityFPPvPvUiUi` | 0x00341670 | CONFIRMED ² | the byte-level write primitive the whole SaveGame chain is built on |
| 0x00434720 | `entity_mem_read` | `MemRead__8C_entityFPPvPvUiUi` | 0x003416c0 | CONFIRMED ² | the byte-level read primitive |
| 0x00434760 | `entity_get_vis_logs_size` | `GetVisLogsSize__8C_entityFv` | 0x00341710 | CONFIRMED ² | **constant 0x184** |
| 0x00434770 | `entity_save_vis_logs` | `SaveVisLogs__8C_entityFPPv` | 0x00341720 | CONFIRMED ² | VIS-LOGS block; field band `+0xb14`/`+0xba4` |
| 0x00434840 | `entity_load_vis_logs` | `LoadVisLogs__8C_entityFPPv` | 0x00341850 | CONFIRMED ² | the load half |
| 0x004348e0 | `entity_get_sensors_size` | `GetSensorsSize__8C_entityFv` | 0x003419c0 | CONFIRMED ² | **constant 0x3a0** |
| 0x004348f0 | `entity_save_sensors` | `SaveSensors__8C_entityFPPv` | 0x003419d0 | CONFIRMED ² | SENSORS block, base `+0xbc0`; the sound-contact ring at `+0xc74` lies inside it |
| 0x00434c10 | `entity_load_sensors` | `LoadSensors__8C_entityFPPv` | 0x00341ec0 | CONFIRMED ² | the load half |
| 0x00434ea0 | `entity_get_logs_size` | `GetLogsSize__8C_entityFv` | 0x00342340 | CONFIRMED ² | **variable** — it loops the list |
| 0x00434f00 | `entity_save_logs` | `SaveLogs__8C_entityFPPv` | 0x00342390 | CONFIRMED ² | AI-LOGS block, base `+0xeac` |
| 0x004351e0 | `entity_load_logs` | `LoadLogs__8C_entityFPPv` | 0x00342740 | CONFIRMED ² | the load half |
| 0x00435410 | `entity_get_character_size` | `GetCharacterSize__8C_entityFv` | 0x00342a00 | CONFIRMED ² | **constant 0x24** |
| 0x00435420 | `entity_save_character` | `SaveCharacter__8C_entityFPPv` | 0x00342a10 | CONFIRMED ² | CHARACTER block, band `+0xebc`..`+0xec8` |
| 0x004354e0 | `entity_load_character` | `LoadCharacter__8C_entityFPPv` | 0x00342b10 | CONFIRMED ² | the load half |
| 0x004355a0 | `entity_get_state_size` | `GetStateSize__8C_entityFv` | 0x00342c10 | CONFIRMED | **identical formula**: `(list_bytes/0x10)*0x10 + CONST + nlocks*0xc`; the constant differs per build (PC `0x168`). The savegame "State" block is the blocked-con list **plus the substate-lock table** — independent confirmation of what `+0x10b8` is |
| 0x00435600 | `entity_save_state` | `SaveState__8C_entityFPPv` | 0x00342c60 | CONFIRMED | exact count: `entity_mem_write` ×**27** ↔ `MemWrite` ×**27**. Serialises the substate-lock vector and the blocked-con list |
| 0x004358f0 | `entity_load_state` | `LoadState__8C_entityFPPv` | 0x00343060 | CONFIRMED | rebuilds the same two containers in the same order at the forced 1:1 position. Counts differ (`entity_mem_read` ×27 vs `MemRead` ×21) because PC inlines PS2's vector ops as `fill_n_stride_0xc` / `copy_range_stride_0xc` / `vec_count_stride_0xc`; the container evidence carries the row |
| 0x00435e00 | `entity_get_action_size` | `GetActionSize__8C_entityFUi` | 0x00343480 | CONFIRMED | the **serialized** size of one saved `ai_action`, switched on its type id. Its case set is neither a subset nor a superset of the factory table (§6b) |
| 0x00435f60 | `entity_get_action_man_size` | `GetActionManSize__8C_entityFv` | 0x003435b0 | CONFIRMED ² | reads `+0x1168` = the `ai_action_manager` at `+0x1164` |
| 0x00436020 | `entity_save_action_man` | `SaveActionMan__8C_entityFPPv` | 0x003436d0 | CONFIRMED ² | every queued `ai_action` |
| 0x00436cf0 | `entity_load_action_man` | `LoadActionMan__8C_entityFPPv` | 0x00344da0 | CONFIRMED ² | the load half |
| 0x00438430 | `entity_save_game_get_size` | `SaveGameGetSize__8C_entityFv` | 0x00346b30 | CONFIRMED | **the same eight callees**: the `C_human` parent + the `C_program` + all six sub-block size getters (vis_logs, sensors, logs, character, state, action_man). It calls `human_save_game_get_size` as its parent, which is why the two were confusable |
| 0x00438490 | `entity_save_game_save` | `SaveGameSave__8C_entityFPPv` | 0x00346bc0 | CONFIRMED ² | parent → C_program → each sub-block |
| 0x00438550 | `entity_save_game_load` | `SaveGameLoad__8C_entityFPPv` | 0x00346cc0 | CONFIRMED ² | true extent 192 B |
| 0x00438610 | `entity_is_in_entity_fire_area` | `isInEntityFireArea__8C_entityF…` | 0x00346dd0 | CONFIRMED ² | friendly-fire test; exact arity `this` + 3 |
| 0x00438970 | `entity_is_in_player_fire_area` | `isInPlayerFireArea__8C_entityF…` | 0x003471d0 | CONFIRMED ² | exact arity `this` + 2 |
| 0x00438ea0 | `entity_get_fire_metrics` | `GetFireMetrics__8C_entityFP12ai_fire_area` | 0x003475d0 | CONFIRMED ² | fills an `ai_fire_area` (below) |
| 0x00438f00 | `entity_get_player_fire_metrics` | `GetPlayerFireMetrics__8C_entityF…` | 0x003476a0 | CONFIRMED ² | the same out-struct for the player |
| 0x00438fa0 | `entity_get_target` | `GetTarget__8C_entityFv` | 0x003477a0 | CONFIRMED ² | reads the PENDING `ai_action` and switches on its type id (case 6 = `NewFire`, `0x1c` = `NewCornerFight`), returning the target from its payload — a third, independent use of the type enum |
| 0x00439050 | `entity_beware_cross_fire` | `BewareCrossFire__8C_entityFUi` | 0x00347870 | CONFIRMED ² | crossfire avoidance; companion of the type-`0x2a` `ai_action_new_strafe_beware` |
| 0x004392e0 | `entity_has_heard_gunfire` | `isHearedShooting__8C_entityFUiUi` | 0x00347c00 | CONFIRMED ² | byte-level match; magic **type 2 = GUNFIRE**; reads `+0xc74`/`+0xc78` |
| 0x0043a150 | `entity_is_in_danger` | `IsInDanger__8C_entityFv` | 0x003495a0 | CONFIRMED | asks "is ANY `C_entity` running a fight/fire action targeting me?" Both builds scan the mission actor vector, gate on kind `0x1b`, then run a `GetJob(type)` chain comparing each action's payload to `this`. Shared type ids `0x1e` / `0x1d`; PC additionally tests `0x1c`, `0x21`, 6 |
| 0x0043a220 | `entity_expire_ghost_trail` | `TickGhost__8C_entityFUi` | 0x00349710 | CONFIRMED | GCs the `ai_ghost` trail: walks it BACKWARDS and shift-down-erases records older than the deadline. Same reverse walk, same expiry test, same erase; ×1 ↔ ×1 from the confirmed `entity_ai` twin |
| 0x0043a850 | `entity_suspend_ai_and_hide` | `NeserADejchej__8C_entityFv` | 0x00349f40 | STRUCTURAL | bumps the AI state out of every `switch` range the AI dispatches on, so the entity stops being driven without losing state, then plays a breath animation. When the caller asks for it AND the frame is visible AND the entity is alive AND `+0x04 & 5` is clear it also latches `+0xf08 \|= 0x20000`, hides the I3D frame through the frame vtable `+0x24`, removes the shadow caster and clears `+0x47c`. Dev-joke PS2 name |
| 0x0043a8e0 | `entity_resume_ai_and_show` | `NeserANedejchej__8C_entityFv` | 0x00349f60 | STRUCTURAL | the inverse of the row above, driven by the `0x20000` latch. Dev-joke PS2 name |
| 0x0043a980 | `entity_add_blocked_con` | `AddBlockedCon__8C_entityFP11blocked_con` | 0x00349fc0 | CONFIRMED | dedup-on-insert into the `vector<blocked_con>` at `+0x119c`. Shared magic `0x29a` tested on `param_2[2]` on both; stride 0x10; same dedup-with-early-return; caller `LoadState` ×1. **PS2's signature names the element type** |
| 0x0043b030 | `program_base_thunk_entity_scalar_deleting_dtor` | `@3056@__dt__8C_entity` | - | CONFIRMED ³ | MI adjustor thunk: `C_program*` → `C_entity*`, then jump |
| 0x0043b040 | `program_base_thunk_entity_save_game_get_size` | `@3056@SaveGameGetSize__8C_entityFv` | 0x0034b8c0 | CONFIRMED | MI adjustor thunk |
| 0x0043b050 | `program_base_thunk_entity_save_game_save` | `@3056@SaveGameSave__8C_entityFPPv` | 0x0034b8b0 | CONFIRMED | MI adjustor thunk |
| 0x0043b060 | `program_base_thunk_entity_save_game_load` | `@3056@SaveGameLoad__8C_entityFPPv` | 0x0034b8a0 | CONFIRMED | MI adjustor thunk |
| - | - | `actionTick_2__8C_entityFP9ai_actionUiUiPUi` | 0x003201f0 | CONFIRMED terminal | a **PS2-only source split** of the switch above, not a distinct method: its sole caller is `actionTick`, which passes all 5 args through verbatim, and it handles the high type tail `0x1d..0x2a`. MSVC emitted all 40 cases as ONE function. Do not hunt a PC twin |
| - | - | `NetDirect__8C_entityFUi` | 0x0032ef30 | base-stub ⁴ | 8 B |
| - | - | `ChangeState__8C_entityFv` | 0x00339ba0 | CONFIRMED terminal | 160 B, inlined into its callers on PC: the `entity_force_ai → entity_dispatch_event` gap contains exactly 3 real PC functions and all 3 are body-confirmed as `isFakeScript` / `ForceScript` / `CallBack`. Do not hunt a PC twin |
| - | - | `InvalidatePaths__8C_entityFv` | 0x0033fd40 | not located | 12 B |
| - | - | `canAim__8C_entityFUi` | 0x00347c60 | not located | 320 B, a time-throttled aim predicate |
| - | - | `SedniNaPrdelASledujKino__8C_entityFb` | 0x00349f20 | base-stub ⁴ | 8 B (dev-joke name) |
| - | - | `VzbudSeUzSouTitulky__8C_entityFv` | 0x00349f30 | base-stub ⁴ | 8 B (dev-joke name) |
| - | - | `UpdateAttention__8C_entityFv` | 0x0034a290 | not located | 3144 B, perception. Its PC twin is not in this cluster: the largest unnamed PC function in the trailing gap is 1136 B |
| - | - | `GetAttentionDelay__8C_entityFP7C_human` | 0x0034aff0 | not located | 92 B |
| - | - | `SightFilter__8C_entityFv` | 0x0034b050 | not located | 240 B |
| - | - | `@3056@CallBack__8C_entityF…` | 0x0034b850 | base-stub | MI adjustor thunk (8 B) |
| - | - | `@3056@DecompileCommand__8C_entityFiPc` | 0x0034b860 | base-stub | MI adjustor thunk |
| - | - | `@3056@Process__8C_entityFUi` | 0x0034b870 | base-stub | MI adjustor thunk |
| - | - | `@3056@DecodeInstruction__8C_entityFPcRi` | 0x0034b880 | base-stub | MI adjustor thunk |
| - | - | `@3056@DecodeParams__8C_entityF…` | 0x0034b890 | base-stub | MI adjustor thunk |

¹ MSVC-only construct, so it can have no PS2 symbol. ² Recovered through the emission-order trailing
gap; the proof is structural — `entity_save_game_get_size` sums these members and so names them.
³ The PS2 `@3056@__dt__` thunk is not one of the 110 methods counted here. ⁴ An 8-byte MIPS function
is a perfectly good getter or forwarder, so this tag is a size observation, not a proof: these three
rows are unread (§9). The 5 `@3056@` MI thunks are genuine stubs.

**Coverage**, counted off the rows: **92 CONFIRMED** (including the two terminals `actionTick_2`
FUSED and `ChangeState` INLINED, and the three savegame adjustor thunks) · **2 STRUCTURAL** ·
**7 base-stub** · **6 not located**.

Every C_entity lifecycle method calls its `C_human` **parent** — an unambiguous, build-independent
signal, and the lever that anchors the lifecycle block of this table.

### `ai_fire_area` — the out-struct of the fire-metrics pair

`entity_get_fire_metrics` / `entity_get_player_fire_metrics` both FILL an `ai_fire_area`:

| offset | field | evidence |
|---|---|---|
| +0x18 | engagement range | ← `entity_weapon_get_max_range()`. When the player holds no weapon, `entity_get_player_fire_metrics` defaults it to **1.5f** |
| +0x1c, +0x20 | role not pinned | both ZEROED when the range is **< 4.0** |

## 6b. The `ai_action` type table (35 factory types)

Every `ai_action` carries its TYPE at offset 0 of its `0x24`-byte header. `entity_action_tick`
switches on exactly this value across **40 outer cases**, and 35 of those types are built by one
factory each; the PC and PS2 factory sets join **1:1 on the type id**. This is the NPC AI action
vocabulary — what an AI-driven person in Mafia can be told to do. Only types **0x21** and **0x28**
are police-specific; `0x18..0x1c` are the **cover system**, the AI-action side of the C_human
`Do_Corner*` family.

**The table below is the FACTORY set, not the whole type space.** Five more type ids are live and
have no factory — see below.

| type | PC factory (`ai_action_new_*`) | PS2 factory | inner (PC) | role |
|---|---|---|---|---|
| 0x00 | `_move` | `NewMove` | 0x20 | walk/run to a world position |
| 0x01 | `_follow` | `NewFollow` | 0x14 | follow an actor |
| 0x05 | `_turn_to` | `NewTurnTo` | 0x04 | rotate to face an actor |
| 0x06 | `_fire` | `NewFire` | 0x0c | shoot at an actor |
| 0x08 | `_follow_link` | `NewFollowLink` | 0x18 | follow along a path link |
| 0x09 | `_blocked_way` | `NewBlockedWay` | 0x08 | react to a blocked route |
| 0x0a | `_runaway_dist` | `NewRunawayDist` | 0x20 | flee until a distance is reached |
| 0x0b | `_side_look` | `NewSideLook` | 0x18 | glance sideways |
| 0x0c | `_back_look` | `NewBackLook` | 0x20 | glance behind |
| 0x0d | `_unlink` | `NewUnLink` | 0x04 | detach from a path link |
| 0x10 | `_run_closer_to` | `NewRunCloserTo` | 0x24 | approach an actor |
| 0x12 | `_freeze` | `NewFreeze` | 0x04 | stand still |
| 0x14 | `_move_to_car` | `NewMoveToCar` | 0x28 | go to a car (the boarding path) |
| 0x15 | `_step_away` | `NewStepAway` | 0x14 | step aside |
| 0x16 | `_strafe` | `NewStrafe` | 0x1c | strafe |
| 0x17 | `_jump` | `NewJump` | 0x04 | jump |
| **0x18** | `_corner_take_pos` | `NewCornerTakePos` | 0x40 | **cover** — take a cover position |
| **0x19** | `_corner_look` | `NewCornerLook` | 0x10 | **cover** — peek from cover |
| **0x1a** | `_corner_cover_fire` | `NewCornerCoverFire` | 0x10 | **cover** — suppressing fire from cover |
| **0x1b** | `_corner_fire` | `NewCornerFire` | 0x10 | **cover** — fire from cover |
| **0x1c** | `_corner_fight` | `NewCornerFight` | 0x48 | **cover** — full fight-from-cover behaviour |
| 0x1d | `_crouch_fight` | `NewCrouchFight` | 0x40 | fight while crouched |
| 0x1e | `_normal_fight` | `NewNormalFight` | 0x58 | default stand-up firefight |
| 0x1f | `_barricade_fight` | `NewBarricadeFight` | 0x1c | fight from behind a barricade |
| 0x20 | `_follow_in_rail` | `NewFollowInRail` | 0x14 | follow a human along a rail/track |
| **0x21** | `_police_action` | `NewPoliceAction` | 0x60 | **police** — the fine/arrest action (drives the busted cutscene and the HUD police symbol) |
| 0x22 | `_search` | `NewSearch` | 0x34 | search for a lost target |
| 0x23 | `_kick_driver` | `NewKickDriver` | 0x28 | the carjack eject (cf. `human_do_throw_occupant_from_car`). The ctor's FIRST parameter is the `C_human` to be thrown out: it goes to `inner+0x20`, and `inner+0x1c` caches THAT human's vehicle, not the running entity's |
| 0x24 | `_hostile_coverage` | `NewHostileCoverage` | 0x84 | biggest action; takes a `vector<C_actor*>` of hostiles |
| 0x25 | `_car_jump` | `NewCarJump` | 0x10 | jump out of the way of a car |
| 0x26 | `_car_stop` | `NewCarStop` | 0x20 | make a car stop |
| 0x27 | `_car_aim` | `NewCarAim` | 0x20 | aim at or from a car |
| **0x28** | `_police_call` | `NewPoliceCall` | 0x30 | **police** — radio for backup (the callable-device path) |
| 0x29 | `_strafe_him` | `NewStrafeHim` | 0x0c | strafe relative to a human |
| 0x2a | `_strafe_beware` | `NewStrafeBeware` | 0x30 | strafe while avoiding a friendly line of fire (takes an `ai_fire_area`) |

**`entity_get_action_size` is not a cross-check of this table.** Its case set adds `3`, `7`, `0x11`,
`0x13` and omits `0x14`, `0x21`, `0x24`, `0x25`, `0x26`, `0x27`, `0x28`, which fall through to its
`return 0` — i.e. **those seven action types are not persisted at all**, a separate finding about the
savegame. Its values are also SAVE widths, not inner-payload sizes: type `5` allocates an inner of
**4** and sizes at `0x20`; type `6` allocates **0xc** and sizes at `0x28`. Compare it with the
savegame writer, never with the `inner (PC)` column.

### 6b-i. The `ai_action` object layout

Proven by all 35 factories at once — every one builds the same header, and the type id it stamps at
offset 0 is what makes the 1:1 join possible.

```c
typedef struct ai_action {          // 0x24-byte header + a per-type inner payload
    /* +0x00 */ uint32_t type;      // [H] THE TYPE ID (0x00..0x2a). The diagnostic: `*outer = <TYPE>`.
                                    //     entity_action_tick switches on exactly this. See 6b.
    /* +0x04 */ uint32_t uid;       // [H] from ai_action_generate_UID()
    /* +0x08 */ uint32_t expiry;    // [H] the caller's expiry seed
    /* +0x0c */ uint32_t expiry2;   // [H] the SAME value written twice by every factory
    /* +0x10 */ uint32_t start_ms;  // [M] world clock stamp (`*(*(g_act_mission+0x24)+0x2b0c)`)
    /* +0x14 */ uint32_t inner_size;// [H] size of the inner payload (varies per type - see 6b)
    /* +0x18 */ uint32_t _zero_18;  // [H] zeroed by every factory
    /* +0x1c */ uint32_t _zero_1c;  // [H] zeroed by every factory
    /* +0x20 */ void    *inner;     // [H] the per-type payload (a second operator_new)
} ai_action;
```

Inner payload sizes differ per build (e.g. type `0x24`: PS2 `0x90` against PC `0x84`) — expected
MIPS-vs-x86 packing. **The type id is the invariant, not the size.**

### The five producerless action types — `0x03`, `0x07`, `0x0e`, `0x11`, `0x13`

40 outer cases − 35 factories = **5 types the dispatcher handles but that nothing located
constructs**:

| type | evidence it is live | what it does |
|---|---|---|
| `0x03` | a `case` in `entity_get_action_size` | not investigated |
| **`0x07`** | a `case` in `entity_get_action_size`; a full arm in `entity_action_tick` | **opens a door** — `inner+0x00` is a `C_door *` (`structs/door.md` §4b) |
| `0x0e` | an outer `case` in `entity_action_tick` | not investigated |
| `0x11` | a `case` in `entity_get_action_size`, and in `ai_action_invalidate_path_for_actor`'s case set | not investigated |
| **`0x13`** | a `case` in `entity_get_action_size`; a full arm in `entity_action_tick` | **closes a door once clear of it**, on a time + distance gate (`structs/door.md` §4b) |

The search for a producer is exhaustive over the factory pattern and came back empty:

- exactly **35** PC functions allocate a `0x24`-byte `ai_action`, and all 35 are the named factories
  (the other five `0x24`-allocators in the binary are unrelated — a float-const setter, a ring
  buffer, and so on);
- PS2's `ai_action_manager` exposes exactly **35** typed factories; `NewJob` / `NewModifier` /
  `NewIdle` are **install wrappers** taking an *existing* `ai_action*`, not factories;
- there is no in-place mutation of the type field anywhere in the dispatcher;
- a sweep of the whole `.text` finds **zero** immediate dword stores of `7`, so the type is written
  from a register by some parameterised creator, or the action only ever arrives back through
  `entity_load_action_man`. `ai_path_update_spec_waypoints` is not the creator — it calls
  `door_set_state` directly, never through an action.

So the producer of these five is open (§9), and `0x07` / `0x13` are demonstrably live behaviour, not
dead handler code.

## 6c. The `C_entity` AI sub-objects and the per-frame perception chain

A `C_entity` is an **aggregate of AI sub-objects**, each its own class with its own method family.
Every base below is byte-proven from the receiver setup at the callsites — `lea ecx,[this+N]` /
`add ecx,N` = **embedded**.

| PC base | class | methods | doc |
|---|---|---|---|
| `+0xa9c` | `C_program` (script VM) | - | §4c |
| `+0xb04` | `ai_vis_logs` | 5 | below |
| `+0xbb4` | `ai_sensors` | 23 | `structs/ai_sensors.md` |
| `+0xe88` | `ai_logs` | 14 | `structs/ai_logs.md` |
| `+0xebc` | `ai_character` | 6 | `structs/ai_character.md` |
| `+0xf64` | `ai_fight_modes` | 13 | `structs/ai_fight_modes.md` |
| `+0x1164` | `ai_action_manager` | 58 | `structs/ai_action_manager.md` — 58/58 accounted; the object is 0x34 bytes: three `std::vector<ai_action*>` (jobs `+0x04`, idles `+0x14`, modifiers `+0x24`) plus the action-UID counter at `+0x30` |
| `+0x1268` | `ai_ghost` (obstacle memory) | 5 | `structs/ai_ghost.md` — a short-term memory of collidable objects met on a path, not a position-history trail |

### The per-frame perception chain

`entity_ai` (vtable slot 13) runs this triple **19×** — once per branch of its state machine:

```
  lea ecx,[esi+0xe88] ; call ai_logs_tick
  lea ecx,[esi+0xb04] ; call ai_vis_logs_tick
  mov eax,[0x65115c] ; push esi ; mov ecx,[eax+0x24]
  add ecx,0x2b3c     ; call sound_arena_update_entity_sound_contacts
```

The PS2 build throttles the middle call instead of running it per entity — §7.

### `ai_vis_logs` (`+0xb04`) — 5 methods

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00410670 | `ai_vis_logs_init` | `Init__11ai_vis_logsFPv` | 0x0030fa80 | CONFIRMED | takes an owner backpointer; seeds the slots with 666.0f and `+0xa8` with 0x29a |
| 0x004106c0 | `ai_vis_logs_add_entity` | `AddEntity__11ai_vis_logsFPvfb` | 0x0030fb50 | CONFIRMED | shared magic constants |
| 0x00410770 | `ai_vis_logs_del_entity` | `DelEntity__11ai_vis_logsFPvb` | 0x0030fc50 | CONFIRMED | shared magic constants |
| 0x004107d0 | `ai_vis_logs_tick` | `Tick__11ai_vis_logsFv` **+ `TickCorpses__11ai_vis_logsFv` FUSED** | 0x0030fce0 / 0x00310c90 | CONFIRMED | 3328 B. **PC fuses the two PS2 functions into one**, proven by the callee multiset: it calls Tick-only callees (`NewVisualCon`, a direct `ai_character_get_sight`) *and* TickCorpses-only callees (`Event__8C_entity`, `hasLog__7ai_logs`), and the union counts match exactly (`GetVisibleRange` 2+2 = **4**; `GetLog` 1+2 = **3**; `TestLineS` 1+1 = **2**) |

## 7. PC vs PS2

**Kind field.** The enum values in §1 are identical on both builds; the field sits at `entity+0x50`
on PS2 (`__ct__<N>C_<class>Fv` → `*(this+0x50) = <value>`), and the §1 values are byte-confirmed from
those constructors. The `+0x14` flavour field of §3a is PS2 `+0x54`, one step off the same bridge.

**Rail content.** In the PS2 build **no trams or trains actually drive in the city** (in-game
observation). The rail classes are still present in the PS2 code, so this is a content/spawn
difference, not a code removal.

**C_entity band deltas.** The PC→PS2 delta **varies per band** — the layout was reorganised for the
port, so never rebase across a band boundary:

| PC | PS2 | delta | band |
|---|---|---|---|
| `+0x78` | `+0x208` | - | act_state |
| `+0xa9c` | `+0xbf0` | 0x154 | `C_program` |
| `+0xb04` / `+0xbb4` | `+0xc50` / `+0xd00` | 0x14c | `ai_vis_logs`, `ai_sensors` |
| `+0xc74`/`+0xc78` | `+0xdd0`/`+0xdd4` | - | the sound-contact ring: PS2 stride 0x30 against PC 0x20, and it sits at PS2 `ai_sensors+0xd0` against PC `+0xc0` — 3 S_vectors of drift |
| `+0xe88` | `+0x1060` | 0x1d8 | `ai_logs` |
| `+0xebc` | `+0x1090` | 0x1d4 | `ai_character` |
| `+0x46c` and `+0xf08` | `+0x10f0` | - | **merged** — see below |
| `+0xf40` / `+0xf4c` / `+0xf50` | `+0x1128` / `+0x1134` / `+0x1138` | 0x1e8 | AI state band |
| `+0xf64` | `+0x114c` | 0x1e8 | `ai_fight_modes` (its internal layout has zero delta) |
| `+0x10b8` / `+0x10c4` / `+0x10c8` | `+0x129c` / `+0x12a8` / `+0x12ac` | 0x1e4 | substate locks (PS2 renders count @`+0x12a0`, data @`+0x12a4`) |
| `+0x1164` | `+0x1348` | 0x1e4 | `ai_action_manager` |
| `+0x119c` / `+0x11a0` | `+0x1370` / `+0x1374` | 0x1d4 | blocked connections |
| `+0x11b4` | `+0x1390` | 0x1dc | head_dir |
| `+0x11c0` | `+0x13a0` | 0x1e0 | head_pos |
| `+0x11cc` | `+0x13b0` | 0x1e4 | neck-bone cache |
| `+0x11d0`..`+0x11ec` | `+0x13b4`..`+0x13d0` | 0x1e4 | weapon band (the pre-proven anchor) |
| `+0x1268` | `+0x1460` | 0x1f8 | `ai_ghost` (PS2 keeps `begin` at base+0x00; records stride 0x20 with the timestamp at `+0x10`) |

The head band walks by +4 per vector (`0x1dc` → `0x1e0` → `0x1e4`) because a PS2 `S_vector` is 16 B
where a PC vec3 is 12 B — the same drift that makes the `ai_ghost` record 0x20 on PS2 against 0x1c
on PC.

**The two flag words are one word on PS2.** PS2 `C_entity+0x10f0` carries all of it: `CallBack` sets
`0x200` / masks `0xfffff1ff` there (PC `+0x46c`), while `Sniff__10ai_sensors` tests `& 0x4000` and
`ScriptSwitch` runs `if (f & 0x200) f |= 0x400` on the same word (PC `+0xf08`). The mapping is
**many-to-one**: the port merged the two PC words. That is why a PS2 pair that talks to itself
through one word legitimately splits across `+0x46c` / `+0xf08` on PC (§4b).

**`C_program` internals differ.** Only the vector base at `program+0x10` coincides:

| | PC (`entity_script_is_fake`) | PS2 (`isFakeScript`) |
|---|---|---|
| instruction list | `{begin @ +0x10, end @ +0x14}`, count **computed** as `(end−begin)>>3` | `{count @ +0xc, base @ +0x10}` — count **stored** |
| pc | `+0x1c` | `+0x14` |
| cleared on the fake path | `+0x60` (= entity `+0xafc`) | `+0x54` (= entity `+0xc44`) |

PS2's `C_program` sits at `C_entity+0xbf0`, proven by three methods that take the program as `this`
(`IsEventID`, `SetEvent`, `CallProcess`, all reading `+0xc`/`+0x10`/`+0x14`) plus `isFakeScript`,
which inlines the same access entity-relative at `+0xbfc`/`+0xc00`/`+0xc04`.

**`GetProgram`'s skew differs per build.** PS2 returns `program + 8` on every class that overrides it
(`C_detector` `+0x80` → `+0x88`, `C_entity` `+0xbf0` → `+0xbf8`; `C_actor`'s override returns 0),
which looks like a secondary base at `program+0x8`. The PC build does **not** reproduce it: the PC
skew is `0xc` (`+0xa9c` → `+0xaa8`). Use `0xc` for PC callsites and `8` for PS2 ones, and do not carry
one across. The `C_detector` case is what settles the PS2 side: `AI__10C_detector` reads `+0x94` (pc)
and `+0x8c` (count) and then calls `CallProcess__9C_program` with `a0` set to `this + 0x80` in the
delay slot — so the program is at `+0x80` while the getter returns `+0x88`.

**Perception is throttled on PS2 and not on PC.** PS2's `AI__8C_entity` calls `Tick__11ai_vis_logs`
**zero** times. Instead a central **`ai_sight_manager`** (7 methods: `Init` / `Add(C_entity*)` /
`Del` / `Next` / `NextCorpses` / `Tick` / `TickCorpses`), driven by `Tick__10ai_manager`,
round-robins: each frame it advances a cursor and ticks only **one** entity's vis_logs. PS2's
`GameInit__8C_entity` calls `Add__16ai_sight_manager` ×2 and `GameDone` calls
`Del__16ai_sight_manager` + `Del__16ai_panic_manager`, none of which has a PC equivalent.
`ai_sight_manager` is therefore very likely **PS2-port-only** — graded CANDIDATE: the positive
evidence is on the PC side (PC ticks per-entity inline from `entity_ai`, so a central batching
manager would be redundant), while its caller `ai_manager::Tick` is not itself a known port-only
subsystem. Gameplay consequence, worth an in-game check (§9): on PS2 an NPC's vision updates only on
its turn, so perception latency scales with the number of live NPCs, which predicts PS2 NPCs are
slower to notice the player in crowded scenes.

**Other structural differences already noted in the tables:** the PC fuses `Tick` + `TickCorpses`
into one `ai_vis_logs_tick` (§6c); PS2 splits `actionTick` into two functions where MSVC emitted one
(§6); PS2's `ChangeState` is inlined on PC (§6); the savegame "State" constant differs per build
(PC `0x168`, PS2 `0x14c`).

## 8. Related

- `structs/actor.md` (the `C_actor` base), `structs/human.md` (the `C_human` base).
- `structs/ai_action_manager.md` — the `+0x1164` action-queue sub-object.
- `structs/ai_sensors.md`, `structs/ai_character.md`, `structs/ai_logs.md`,
  `structs/ai_fight_modes.md`, `structs/ai_ghost.md`, `structs/ai_blob_log.md` — the AI sub-objects.
- `structs/program.md` — the `C_program` script VM.
- `structs/car_extended.md` — the `+0x14` flavour field's only user.

## 9. Open items

| Item | State |
|---|---|
| which PC code sets bit `0x200` on `+0xf08` | the located setter writes `+0x46c`, while `entity_script_switch` reads `+0xf08` (§4b). Either PC has a second setter, or the two words are kept in sync somewhere |
| what lives at `program + 0xc` (PC) / `+0x8` (PS2) | which base or interface subobject `GetProgram`'s return type is. Decoding `C_program`'s first 0x10 bytes would answer it (§4c) |
| `+0x10c8` lock_aux | written by `entity_lock_substate` from its `aux` param; no reader located |
| `+0xf50` ai_flags | bits 0 and 1 are known; the rest are not pinned |
| `+0x11d0` weapon_def_param | copied from `weapon_def+0x6c`; no consumer decoded |
| `ai_fire_area` `+0x1c` / `+0x20` | zeroed when the engagement range is < 4.0; role unknown |
| six unmapped PS2 methods | `canAim`, `InvalidatePaths`, `UpdateAttention`, `GetAttentionDelay`, `SightFilter` are not located (the last four are outside this PC cluster on size grounds); `actionTick_2` and `ChangeState` are terminal and must not be hunted |
| three unread `base-stub` rows | `NetDirect`, `SedniNaPrdelASledujKino`, `VzbudSeUzSouTitulky` — each 8 bytes on PS2, which is also the size of a real getter or forwarder. Read them before trusting the tag |
| who CREATES a type `0x03` / `0x07` / `0x0e` / `0x11` / `0x13` `ai_action` | no factory stamps any of them and no immediate store of the constant exists (§6b); the type is written from a register, or the action only arrives through `entity_load_action_man` |
| `entity_reset_state` destroys the sensors without re-initialising them | observed, not explained; `entity_reset` re-inits them |
| `ai_sight_manager` PS2-port-only | CANDIDATE (§7). Closing it means checking that every PS2 caller is port-added code |
| PS2 perception latency | the round-robin predicts PS2 NPCs notice the player later in crowded scenes than PC NPCs; not tested in game (§7) |
| kind `0x12` | §1 has it as the `C_traffic_generator` spawner, while the script name table (§1a) maps `Traffic_person` to the same value. Whether the script kind means the spawner actor or the crowd record is unresolved |
| kind `0x17` (seagull flock) and kind `0x1d` (bendable plant) | both are PC-only with no construction path located and no in-game observation; recorded as open hunts, not as cut content |
| what creates a `C_smoke_trail` (kind `0xf`) | not located |
