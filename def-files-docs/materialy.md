# `tables\materials.def` — surface / material table

**Created** 2026-06-04 11:16 +0200 · **Modified** 2026-09-18 05:35 +0200

The collision **surface-material table**. Each numeric id names one record. Ids are used as a
**byte** throughout the engine: a collision contact stores the hit geometry's material id, which
then drives per-surface contact **sounds**, **particles**, tyre/footstep effects, car-handling
tweaks, and a couple of gameplay rules (the sidewalk police offence and lethal water).

The id↔name mapping comes from the Mafia Script Data Reference, a third-party Mafia script/data
reference. The original is in Czech; the table below is translated to English.

## 1. Runtime layout

* Loaded by `data_load_def_tables` (`0x00592080`) from the DTA archive into the global
  `g_surface_material_table`; record stride **`0x170` (368) bytes**; record count
  `g_num_surface_materials`. Freed by the sibling at `0x00592240`.
* A material record carries per-surface sound-event ids (record `+0x28` / `+0x2c`, read by the
  contact-sound routine) and float blocks from `+0x108`.
* A parallel surface table `DAT_00656320` (`0x28`-byte records) is indexed by the same material id.
* A collision contact holds its material id at `wheel_contact+0xb4`, a byte copied from the
  collision hit descriptor by the per-wheel ground-update routine (Ghidra label `LAB_005341b4`).
* `Game.exe` only ever compares these ids numerically — it never stores the names.

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
| 13 | `0x0d` | Wood 0% | wear tier ³ |
| 14 | `0x0e` | Wood 30% | wear tier ³ |
| 15 | `0x0f` | Wood 70% | wear tier ³ |
| 16 | `0x10` | Parquet | - |
| 17 | `0x11` | Marble / stone / floor tile | - |
| 18 | `0x12` | Carpet | - |
| 19 | `0x13` | Metal structures (bridges, fire escapes) | - |
| 20 | `0x14` | Sheet metal 0% (roofs, dock crates) | wear tier ³ |
| 21 | `0x15` | Sheet metal 30% | wear tier ³ |
| 22 | `0x16` | Sheet metal 70% | wear tier ³ |
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

³ "0% / 30% / 70%" are variant tiers of the same base material.

⁴ The player cannot swim, so entering a body of water (river, ocean) on foot or in a car results in
death. Id 44 is the wet-road handling material and is distinct from id 31. A special-case helper at
`0x0046e080` forces a wheel contact's `+0xb4` to `0x1f`.

⁵ Ids 40 and 41 are scripted collision volumes, not real-world surfaces.

## 3. Related

`police-system/artifacts/offences.md` — the sidewalk offence (material id 3) ·
`structs/crash_obj.md` — a consumer that resolves surface material ids through this table.

## 4. Open items

| item | state |
|---|---|
| what separates the 0% / 30% / 70% wear tiers at runtime | they are distinct ids in the table; no consumer has been traced that picks between them |
| the float block from record `+0x108` | present in every record; no consumer traced |
| the parallel `0x28`-byte table `DAT_00656320` | indexed by the same material id; its record layout is not decoded |
