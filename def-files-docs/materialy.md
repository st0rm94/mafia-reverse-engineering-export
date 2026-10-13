# `tables\materials.def` — surface / material table

**Created** 2026-06-04 11:16 +0200 · **Modified** 2026-10-11 18:25 +0200

The collision **surface-material table**. Each numeric id names one record. Ids are used as a
**byte** throughout the engine: a collision contact stores the hit geometry's material id, which
then drives per-surface contact **sounds**, **particles**, tyre/footstep effects, car-handling
tweaks, and a couple of gameplay rules (the sidewalk police offence and lethal water).

The id↔name mapping comes from the Mafia Script Data Reference, a third-party Mafia script/data
reference. The original is in Czech; the table below is translated to English.

## 1. Runtime layout

* Loaded by `data_load_def_tables` (`0x00592080`) from the DTA archive into the global
  `g_surface_material_table`; record stride **`0x170` (368) bytes**; record count
  `g_num_surface_materials`. Freed by `data_unload_def_tables` (`0x00592240`).
* The record fields with a located consumer:

  | offset | contents | consumer |
  |---|---|---|
  | `+0x000` | name, `0x20` bytes | none |
  | `+0x020` | footstep sound id | `human_notify_callback`, keyed on the human's ground material `+0x6d4` |
  | `+0x024` | bullet-impact sound id | `game_new_shoot` |
  | `+0x028` / `+0x02c` | landing sound of a spent cartridge / of a shotgun shell | `cartridge_ai_tick`, selected by the cartridge type `+0x138` |
  | `+0x030` / `+0x034` / `+0x038` | melee-strike sound ids, chosen by the striking weapon (below) | `human_ai` |
  | `+0x03c` | bullet-mark decal index | `game_tick_shoot` |
  | `+0x040` | hit-particle id | `game_tick_shoot` |
  | `+0x108` .. `+0x12c` | the vehicle surface block (§1a) | `car_load_materials` |
  | `+0x130` | contact-sound id array, indexed by the contact kind `C_PhThing+0x308` | `physical_object_move_wheel` |

  The melee column follows the weapon id (`def-files-docs/predmety.md`): `+0x038` for ids 0 and 2
  (empty hands, knuckle-duster), `+0x034` for ids 3, 25, 29 and 31 (knife, crowbar, bottle, sword),
  `+0x030` for every other weapon.
* Record `+0x3c` is the **bullet-mark decal index** and `+0x40` the **hit-particle id**:
  `game_tick_shoot` places decal `+0x3c` at a bullet impact (when it lies in 0..9) and passes
  `+0x40` to `particle_manager_create_particle` (when it is not negative). `-1` means none. The
  shipped file's decal indices (`structs/blood.md` §5d for the textures):

  | decal | texture | material ids |
  |---|---|---|
  | 2 | `2dirbet.bmp` | 1-4, 6-10, 17, 33, 36-38, 46 |
  | 3 | `2dirdre.bmp` | 13-16, 18, 42 |
  | 4 | `2dirple.bmp` | 19-22 |
  | 5 | `2dirsklo.bmp` | 24, 25 |
  | -1 | none | every other id (grass, foliage, fences, water, body, the unused and scripted ids) |

* A collision contact holds its material id at `wheel_contact+0xb4`, a byte copied from the
  collision hit descriptor by the per-wheel ground-update routine (Ghidra label `LAB_005341b4`).
* `Game.exe` only ever compares these ids numerically — it never stores the names.

### 1a. The vehicle surface block and `S_VEH_MATERIAL`

`car_load_materials` copies the block at `+0x108` .. `+0x12c` of every record, clamped, into one
`S_VEH_MATERIAL` (`0x30` bytes) per material id and hands the list to `vehicle_set_materials`, which
keeps it at `g_vehicle_materials_begin` and fills the 256-byte reverse index
`g_vehicle_material_index_by_id` (material id to list position; `structs/vehicle.md` §6a). Every
vehicle consumer reads a field through that index.

| material record | `S_VEH_MATERIAL` | clamp | default ¹ | meaning | consumer |
|---|---|---|---|---|---|
| `+0x108` | `+0x18` | 0 .. 1 | 0.35 | friction of a car body sliding on the surface ² | `vehicle_body_coll_resolve` |
| `+0x10c` | `+0x1c` | 1 .. 2 | 1.0 | impulse factor of a car body striking the surface, 1 = no bounce ² | `vehicle_body_coll_resolve` |
| `+0x110` / `+0x114` | `+0x08` / `+0x04` | `+0x04` ≤ `+0x08` | 100 / 100 | longest / shortest flat stretch between two wheel bumps | `vehicle_update_visuals` |
| `+0x118` / `+0x11c` | `+0x10` / `+0x0c` | `+0x0c` ≤ `+0x10` | 100 / 100 | longest / shortest wheel bump | `vehicle_update_visuals` |
| `+0x120` | `+0x14` | 0 .. 0.2 | 0 | largest bump height, applied with a random sign | `vehicle_update_visuals` |
| `+0x124` | `+0x20` | - | 0 | wheel particle while the car moves faster than 3 (`+0x618`) | `vehicle_update_wheel_particles` |
| `+0x128` | `+0x24` | - | 0 | wheel particle while wheel flag bits `0x10000a0` are set | `vehicle_update_wheel_particles` |
| `+0x12c` | `+0x28` | - | 0 | body-impact particle; a negative id also scatters debris | `vehicle_emit_body_collision_particles`, `car_on_collision_apply_damage` |
| - | `+0x2c` | - | 0 | tyre screech allowed ³ | `vehicle_update_skid_sound` |
| - | `+0x2d` | - | 0 | skid marks allowed ³ | `vehicle_emit_wheel_skid_segment` |

¹ The single record `car_load_materials` builds when no material table is loaded.
² A car whose CARINIT sets its own-friction byte uses its own pair instead
(`def-files-docs/vehicles-bin.md` §4e).
³ Not in the file: `car_load_materials` sets both bytes for ids 1-4, 9, 10 and 38 (asphalt,
cobblestone, sidewalk, concrete, rock, rails, brick wall).

In the shipped file only ids 1-11, 19, 37, 45 and 46 carry a non-zero block. Cobblestone (2) has the
densest bumps (0.1 to 0.7 apart, up to 0.03 high), asphalt (1) the sparsest (5 to 8 apart, up to
0.012); the dust-trail particle is set on grass, sand, dirt, gravel and the dusty road.

## 2. Materials

| ID (dec) | ID (hex) | Material (English) | Notes |
|---|---|---|---|
| 0 | `0x00` | Invalid | placeholder / unset |
| 1 | `0x01` | Asphalt | road surface |
| 2 | `0x02` | Cobblestone paving | road surface |
| 3 | `0x03` | **Sidewalk** | drives the **sidewalk police offence** ¹ |
| 4 | `0x04` | Concrete | - |
| 5 | `0x05` | Grass | - |
| 6 | `0x06` | Sand / dusty road | - |
| 7 | `0x07` | Dirt / field path | - |
| 8 | `0x08` | Gravel | - |
| 9 | `0x09` | Rock (for car handling) | - |
| 10 | `0x0a` | Rails (tram track) | - |
| 11 | `0x0b` | Foliage (bushes, trees, roadside leaves) | - |
| 12 | `0x0c` | (unused) | reserved slot ² |
| 13 | `0x0d` | Wood 0% | sound variant ³ |
| 14 | `0x0e` | Wood 30% | sound variant ³ |
| 15 | `0x0f` | Wood 70% | sound variant ³ |
| 16 | `0x10` | Parquet | - |
| 17 | `0x11` | Marble / stone / floor tile | - |
| 18 | `0x12` | Carpet | - |
| 19 | `0x13` | Metal structures (bridges, fire escapes) | - |
| 20 | `0x14` | Sheet metal 0% (roofs, dock crates) | sound variant ³ |
| 21 | `0x15` | Sheet metal 30% | sound variant ³ |
| 22 | `0x16` | Sheet metal 70% | sound variant ³ |
| 23 | `0x17` | (unused) | reserved slot ² |
| 24 | `0x18` | Glass | - |
| 25 | `0x19` | Bulletproof glass | - |
| 26 | `0x1a` | (unused) | reserved slot ² |
| 27 | `0x1b` | Wire fence | - |
| 28 | `0x1c` | Wooden fence (slats) | - |
| 29 | `0x1d` | Hedge (living fence) | - |
| 30 | `0x1e` | (unused) | reserved slot ² |
| 31 | `0x1f` | **Water (lethal — "voda death")** | deep / open water, fatal ⁴ |
| 32 | `0x20` | (unused) | reserved slot ² |
| 33 | `0x21` | Universal sound (generic, deliberately unidentifiable) | fallback sound material |
| 34 | `0x22` | (unused) | reserved slot ² |
| 35 | `0x23` | Body (flesh) | - |
| 36 | `0x24` | Plaster | - |
| 37 | `0x25` | Junk / debris (dirty alley) | - |
| 38 | `0x26` | Brick wall | - |
| 39 | `0x27` | (unused) | reserved slot ² |
| 40 | `0x28` | Death collision | a collision volume that kills on contact ⁵ |
| 41 | `0x29` | Crash collision | a collision volume that triggers a crash ⁵ |
| 42 | `0x2a` | Straw | - |
| 43 | `0x2b` | Very slippery ("buttery") | extra-low-friction surface for cars |
| 44 | `0x2c` | Water (extra slippery for car) | **shallow / drivable**, not lethal ⁴ |
| 45 | `0x2d` | Puddle (extra grippy for car) | grip tweak only, not lethal |
| 46 | `0x2e` | Dusty road | - |

¹ A wheel contact with id 3 sets pursuit-record offence bit `0x40`
(`police-system/artifacts/offences.md`). `Game.exe` only does the numeric `== 3` compare.

² Marked do-not-use in the source reference.

³ "0% / 30% / 70%" are three ids of one base material. Apart from the name the records differ only
in their sound ids (`+0x020` .. `+0x038` and contact-sound slot 8 at `+0x150`); every physical column,
the decal and the particle are equal. No code chooses between them: the scene's collision geometry
carries the id.

⁴ The player cannot swim, so entering a body of water (river, ocean) on foot or in a car results in
death. Id 44 is the wet-road handling material and is distinct from id 31. `car_collision_filter_wheel`
(`0x0046e080`) forces a wheel contact's `+0xb4` to `0x1f`.

⁵ Ids 40 and 41 are scripted collision volumes, not real-world surfaces.

## 3. Related

`police-system/artifacts/offences.md` — the sidewalk offence (material id 3) ·
`structs/crash_obj.md` — a consumer that resolves surface material ids through this table ·
`structs/blood.md` §5 — the decal pool the bullet marks go into ·
`structs/vehicle.md` §6a — the `S_VEH_MATERIAL` list and its index ·
`def-files-docs/vehicles-bin.md` — the CARINIT own-friction pair that overrides `+0x108` / `+0x10c`.

## 4. Open items

Nothing is open.
