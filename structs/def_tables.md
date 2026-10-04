# The four global `.def` tables — PC `[0x00592050, 0x005922c0)`

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-10-01 14:20 +0200

A translation unit, not a class. Two functions load and free the four global lookup tables the
rest of the game indexes constantly: items, surface materials, particles and voices. Every one
is a raw `.dta` read — the whole file into one heap allocation, with a record count derived by
dividing the file size by a fixed stride. There is no parsing step and no per-record
allocation.

---

## 1. The two functions

| PC VA | PC name | role | sole caller |
|---|---|---|---|
| `0x00592080` | `data_load_def_tables` | load all four; returns 1 on success, 0 at the first failed open | `loadsave_init` |
| `0x00592240` | `data_unload_def_tables` | free all four and zero both pointer and count | `loadsave_close` |

`loadsave_init` runs once from `engine_init` and `loadsave_close` once from `engine_shutdown`, so
the tables are loaded once at start-up and freed once at shutdown. `loadsave_init` calls
`data_load_def_tables` as a bare statement and ignores the return value.

Both are all-or-nothing in shape but **not** transactional: `data_load_def_tables` returns 0
the moment any `dtaOpen` fails, leaving the tables loaded so far in place. The four blocks are
otherwise identical:

```
f = dtaOpen(name, 0);        if (f < 0) return 0;
if (table) operator delete(table);
size = dtaSeek(f, 0, 2);  dtaSeek(f, 0, 0);
table = operator new(size);  dtaRead(f, table, size);  dtaClose(f);
count = size / STRIDE;
```

`data_unload_def_tables` is the mirror: four free-and-zero pairs in the load order.

## 2. The tables

| file | pointer | count | PC stride | notes |
|---|---|---|---|---|
| `tables\predmety.def` | `g_game_item_table` | `g_num_game_items` | `0xbc` | *předměty* = items; weapon/item ids |
| `tables\materials.def` | `g_surface_material_table` | `g_num_surface_materials` | `0x170` | surface ids index this; id 3 = sidewalk, id 31 = lethal water |
| `tables\particles.def` | `g_game_particle_table` | `g_num_game_particles` | `0xa4` | vehicle particle emitters; the record layout is §2a |
| `tables\voices.def` | `g_game_voice_table` | `g_num_game_voices` | `0xa4` | voice line groups |

⚠️ **The stride is the record layout's only witness here.** Nothing in this TU reads a field;
it just divides. Any claim about what a record contains has to come from a consumer or from
the shipped file, not from these two functions.

### 2a. `particles.def` — the vehicle particle emitters

The shipped file is `3772` bytes = **23 records** of `0xa4`. Its only consumer is
`car_load_particle_effects`, which walks every record once at start-up: it resolves the model
named in the record through the model cache (loading it when the cache has no match), creates
one LS3D emitter frame (driver frame type `0x11`, `I3D_emitor` in the shipped `LS3DF.dll`) and
feeds record fields to it through `I3D_emitor::SetParam(index, value)` (vtable `+0x54`). The
emitters are then handed to `vehicle_set_particles`.

| offset | type | `SetParam` index | shipped values |
|---|---|---|---|
| `+0x00` | `char[0x20]` | - (the model file name) | `smoke_particle.I3D`, `smoke_particle2.I3D`, `smoke_particle4.I3D`, `prach_particle.I3D`, `prach_particle1.I3D`, `trava_particle.I3D`, `2stone.i3d`, `2listi.i3d`; record 0 is all zero |
| `+0x20` | int | `0x00` | 5..150 |
| `+0x24` | int | `0x01` | 0, 50, 60 |
| `+0x28` | int | `0x02` | 800..12000 |
| `+0x2c` | int | `0x03` | 0..4000 |
| `+0x30` | float | `0x04` | -2.0..1.5 |
| `+0x34` | float | `0x05` | -1.5..2.0 |
| `+0x38` | float | `0x06` | 0..0.8 |
| `+0x3c` | float | `0x07`, passed through `__ftol` first | 0..2.0 |
| `+0x40` | int | `0x08` | 0, 400 |
| `+0x44` | int | `0x09` | 0, 400 |
| `+0x48` | float | `0x0a` | 0 |
| `+0x4c` | float | `0x0b` | 0..0.4 |
| `+0x50` | float | `0x0c` | 0.1..1.0 |
| `+0x54` | float | `0x0d` | 0, 0.1 |
| `+0x58` | float | `0x0e` | 0..3.0 |
| `+0x5c` | float | `0x0f` | 0, 0.2 |
| `+0x60` | float | `0x10` | 0, 1.0 |
| `+0x64` | float | `0x11` | -3.0..2.9 |
| `+0x68` | float | `0x12` | 0 |
| `+0x6c` | int | `0x13` | `0x102`; `0x104` in record 21 |
| `+0x70` | int | `0x14` | 0 |
| `+0x74` | int | - (not read) | `0x101` |
| `+0x78` | int | - (not read) | 0, 1 |
| `+0x7c` | float | `0x18` | -20.0..-0.25 |
| `+0x80` | float | `0x19` | 0.25..20.0 |
| `+0x84` | float | `0x1a` | 0.1..6.0 |
| `+0x88` | int | - (not read) | 0, 1 |
| `+0x8c` | 3 floats | `0x1d`, by pointer | 0..1.0 |
| `+0x98` | 3 floats | `0x1e`, by pointer | 0..1.0 |

The value types are the shipped `LS3DF.dll`'s: its `I3D_emitor::SetParam` (`0x1003c460`, a
32-arm jump table; an index of `0x20` or more prints `!! Invalid index:%u
I3D_emitor::SetParam()`) stores indices `0x00`-`0x03`, `0x08`, `0x09`, `0x13` and `0x14` as dwords,
the other numbered indices above as floats, and `0x15` / `0x16` as bytes. The game sets `0x15`
and `0x16` to the constant 1 for every record. Index `0x07` is the one mismatch: the game
converts `+0x3c` to an integer with `__ftol` and passes that, while the DLL stores the argument
as a float, so the emitter receives the integer's bit pattern. What each index controls inside
the emitter is the DLL's business and is not decoded here.

`prach`, `trava` and `listi` are Czech for dust, grass and leaves.

## 3. Naming and translation unit

* **Why `data_*_def_tables`, not `material_*`.** "Material" would be a claim about the whole set,
  and three of the four tables are not materials. The PC names state the set.
* **The TU** is delimited at both ends by MSVC static-init entries read out of the `.CRT$XCU`
  table at `.data:0x006461b4`: `crt_static_init_def_tables` at `0x00592050`, and the next TU's
  at `0x005922c0`. It is not part of the `g_loadsave` file `[0x00556f10, 0x00561b70)` that holds
  its callers.

## 4. PC vs PS2

| PC name | PS2 twin | PS2 VA | Status |
|---|---|---|---|
| `data_load_def_tables` | `LoadMaterialsFromBin__Fv` | `0x002e29a0` | CONFIRMED |
| `data_unload_def_tables` | `UnloadMaterials__Fv` | `0x002e2c40` | CONFIRMED |

Both are **free functions** on PS2 — an empty class column — so a class-based census cannot
see them; they were found by enumerating the VA band with the class column ignored. The source
file name `g_material.cpp` comes from the PS2 `__sinit_` manifest. The developers' verb
`UnloadMaterials` is as loose as `material_*` would be; the PS2 symbol supplies the mapping, not
the meaning.

`data_load_def_tables` is statement-for-statement its twin: the same four blocks in the same
order, the same guarded delete before each, the same `dtaSeek(f,0,2)` / `dtaSeek(f,0,0)` size
probe, and `return 1` only after the fourth succeeds. `data_unload_def_tables` likewise — four
free-and-zero pairs in the load order.

🔑 **The PS2 body names all eight globals**:

| PC | PS2 |
|---|---|
| `g_game_item_table` / `g_num_game_items` | `gameItemTable` / `numOfGameItems` |
| `g_surface_material_table` / `g_num_surface_materials` | `materialList` / `numOfMaterials` |
| `g_game_particle_table` / `g_num_game_particles` | `gameParticleTable` / `numOfGameParticles` |
| `g_game_voice_table` / `g_num_game_voices` | `gameVoices` / `numOfGameVoices` |

**Two of the four strides differ between builds** — items `0xbc` PC vs `0xf0` PS2, particles
`0xa4` vs `0xf0`; materials `0x170` and voices `0xa4` are identical. The shipped `.def` files
differ per platform, so a record offset measured on one build must never be carried to the
other. The slot (which table, in which order) is the same on both.

## 5. Cross-references

* `Game-ghidra.exe.functions.md` — the function registry.
* `structs/loadsave.md` — the callers `loadsave_init` / `loadsave_close`.
* `structs/car.md` — `car_load_particle_effects`, the `particles.def` consumer.
