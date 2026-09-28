# The four global `.def` tables — PC `[0x00592050, 0x005922c0)`

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-09-17 13:43 +0200

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
| `tables\particles.def` | `g_game_particle_table` | `g_num_game_particles` | `0xa4` | particle definitions |
| `tables\voices.def` | `g_game_voice_table` | `g_num_game_voices` | `0xa4` | voice line groups |

⚠️ **The stride is the record layout's only witness here.** Nothing in this TU reads a field;
it just divides. Any claim about what a record contains has to come from a consumer or from
the shipped file, not from these two functions.

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

## 6. Open items

| item | state |
|---|---|
| `particles.def` record layout | `0xa4` bytes; 37 mentions of `g_game_particle_table` in the decomp, none read for this file |
