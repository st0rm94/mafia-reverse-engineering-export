# The four global `.def` tables

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-11-16 12:35 +0100

A translation unit, not a class. Two functions load and free the four global lookup tables the rest of the game indexes
constantly: items, surface materials, particles and voices. Every one is a raw `.dta` read: the whole file into one heap
allocation, with a record count derived by dividing the file size by the record's stride. There is no parsing step and no per-record
allocation.

| | |
|---|---|
| PC prefix | `data_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x00592050, 0x005922c0)`, 4 starts, all named; `g_material.cpp` |
| PS2 class | free functions of `g_material.cpp`, 2 of them |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_game_item_table` | `0x006efb94` | `tables\predmety.def`, stride `0xbc`; `def-files-docs/predmety.md` | `data_load_def_tables`, `data_unload_def_tables` | 89 references |
| `g_num_game_items` | `0x006efb98` | its record count | `data_load_def_tables`, `data_unload_def_tables` | 45 references |
| `g_surface_material_table` | `0x006efba4` | `tables\materials.def`, stride `0x170`; `def-files-docs/materialy.md` | `data_load_def_tables`, `data_unload_def_tables` | 38 references |
| `g_num_surface_materials` | `0x006efba8` | its record count | `data_load_def_tables`, `data_unload_def_tables` | 13 references |
| `g_game_particle_table` | `0x006efb9c` | `tables\particles.def`, stride `0xa4`; §1a | `data_load_def_tables`, `data_unload_def_tables` | `car_load_particle_effects` |
| `g_num_game_particles` | `0x006efba0` | its record count | `data_load_def_tables`, `data_unload_def_tables` | `car_load_particle_effects` |
| `g_game_voice_table` | `0x006efb8c` | `tables\voices.def`, stride `0xa4`; column 0 is the speech priority, `structs/ai_talkman.md` §6 | `data_load_def_tables`, `data_unload_def_tables` | 21 references |
| `g_num_game_voices` | `0x006efb90` | its record count | `data_load_def_tables`, `data_unload_def_tables` | 11 references |

The reference counts are of the decomp's named uses. **The stride is the record layout's only witness in this unit**: nothing
here reads a field, it only divides, so a claim about what a record contains comes from a consumer or from the shipped file. The
item and material records are documented in the two `def-files-docs` files named above.

### 1a. `S_particle_def_record` — one `tables\particles.def` record, `0xa4` bytes

The shipped file is `3772` bytes = **23 records**. The field names state what is measured: the value's position in the
`I3D_emitor::SetParam(index, value)` calls of §4; what an index controls inside the emitter is the DLL's (Open item 1).

```c
typedef struct S_particle_def_record {
    /* +0x00 */ char model_name[0x20]; // [H] the model file name; record 0 is all zero
    /* +0x20 */ int32_t emitter_param_0x00; // [M] value of SetParam index 0x00; shipped 5..150
    /* +0x24 */ int32_t emitter_param_0x01; // [M] value of SetParam index 0x01; shipped 0, 50, 60
    /* +0x28 */ int32_t emitter_param_0x02; // [M] value of SetParam index 0x02; shipped 800..12000
    /* +0x2c */ int32_t emitter_param_0x03; // [M] value of SetParam index 0x03; shipped 0..4000
    /* +0x30 */ float emitter_param_0x04; // [M] value of SetParam index 0x04; shipped -2.0..1.5
    /* +0x34 */ float emitter_param_0x05; // [M] value of SetParam index 0x05; shipped -1.5..2.0
    /* +0x38 */ float emitter_param_0x06; // [M] value of SetParam index 0x06; shipped 0..0.8
    /* +0x3c */ float emitter_param_0x07; // [M] value of SetParam index 0x07; shipped 0..2.0
    /* +0x40 */ int32_t emitter_param_0x08; // [M] value of SetParam index 0x08; shipped 0, 400
    /* +0x44 */ int32_t emitter_param_0x09; // [M] value of SetParam index 0x09; shipped 0, 400
    /* +0x48 */ float emitter_param_0x0a; // [M] value of SetParam index 0x0a; shipped 0
    /* +0x4c */ float emitter_param_0x0b; // [M] value of SetParam index 0x0b; shipped 0..0.4
    /* +0x50 */ float emitter_param_0x0c; // [M] value of SetParam index 0x0c; shipped 0.1..1.0
    /* +0x54 */ float emitter_param_0x0d; // [M] value of SetParam index 0x0d; shipped 0, 0.1
    /* +0x58 */ float emitter_param_0x0e; // [M] value of SetParam index 0x0e; shipped 0..3.0
    /* +0x5c */ float emitter_param_0x0f; // [M] value of SetParam index 0x0f; shipped 0, 0.2
    /* +0x60 */ float emitter_param_0x10; // [M] value of SetParam index 0x10; shipped 0, 1.0
    /* +0x64 */ float emitter_param_0x11; // [M] value of SetParam index 0x11; shipped -3.0..2.9
    /* +0x68 */ float emitter_param_0x12; // [M] value of SetParam index 0x12; shipped 0
    /* +0x6c */ int32_t emitter_param_0x13; // [M] value of SetParam index 0x13; shipped 0x102
    /* +0x70 */ int32_t emitter_param_0x14; // [M] value of SetParam index 0x14; shipped 0
    /* +0x74 */ int32_t unread_74; // [U] file data, shipped 0x101; no reader (car_load_particle_effects, the table's only reader)
    /* +0x78 */ int32_t unread_78; // [U] file data, shipped 0, 1; no reader (car_load_particle_effects, the table's only reader)
    /* +0x7c */ float emitter_param_0x18; // [M] value of SetParam index 0x18; shipped -20.0..-0.25
    /* +0x80 */ float emitter_param_0x19; // [M] value of SetParam index 0x19; shipped 0.25..20.0
    /* +0x84 */ float emitter_param_0x1a; // [M] value of SetParam index 0x1a; shipped 0.1..6.0
    /* +0x88 */ int32_t unread_88; // [U] file data, shipped 0, 1; no reader (car_load_particle_effects, the table's only reader)
    /* +0x8c */ float emitter_param_0x1d[3]; // [M] passed by pointer as SetParam index 0x1d; shipped 0..1.0
    /* +0x98 */ float emitter_param_0x1e[3]; // [M] passed by pointer as SetParam index 0x1e; shipped 0..1.0
} S_particle_def_record;
```

`model_name` is `0x20` bytes because the next field starts at `+0x20`; the shipped names fit, the longest being 19 characters.
`emitter_param_0x1d` and `emitter_param_0x1e` hold three floats each because the game passes only their address and the next
field lies `0xc` bytes on (the second ends the record); every shipped component is in 0..1.0.

## 2. Field evidence

### 2a. `S_particle_def_record`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `model_name` | H | `data_load_def_tables` | `car_load_particle_effects` | `smoke_particle.I3D`, `smoke_particle2.I3D`, `smoke_particle4.I3D`, `prach_particle.I3D`, `prach_particle1.I3D`, `trava_particle.I3D`, `2stone.i3d`, `2listi.i3d` |
| +0x20 | `emitter_param_0x00` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x00`; shipped 5..150 |
| +0x24 | `emitter_param_0x01` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x01`; shipped 0, 50, 60 |
| +0x28 | `emitter_param_0x02` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x02`; shipped 800..12000 |
| +0x2c | `emitter_param_0x03` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x03`; shipped 0..4000 |
| +0x30 | `emitter_param_0x04` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x04`; shipped -2.0..1.5 |
| +0x34 | `emitter_param_0x05` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x05`; shipped -1.5..2.0 |
| +0x38 | `emitter_param_0x06` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x06`; shipped 0..0.8 |
| +0x3c | `emitter_param_0x07` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x07`; shipped 0..2.0; passed through `__ftol` first |
| +0x40 | `emitter_param_0x08` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x08`; shipped 0, 400 |
| +0x44 | `emitter_param_0x09` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x09`; shipped 0, 400 |
| +0x48 | `emitter_param_0x0a` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0a`; shipped 0 |
| +0x4c | `emitter_param_0x0b` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0b`; shipped 0..0.4 |
| +0x50 | `emitter_param_0x0c` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0c`; shipped 0.1..1.0 |
| +0x54 | `emitter_param_0x0d` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0d`; shipped 0, 0.1 |
| +0x58 | `emitter_param_0x0e` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0e`; shipped 0..3.0 |
| +0x5c | `emitter_param_0x0f` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x0f`; shipped 0, 0.2 |
| +0x60 | `emitter_param_0x10` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x10`; shipped 0, 1.0 |
| +0x64 | `emitter_param_0x11` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x11`; shipped -3.0..2.9 |
| +0x68 | `emitter_param_0x12` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x12`; shipped 0 |
| +0x6c | `emitter_param_0x13` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x13`; shipped `0x102`; `0x104` in record 21 |
| +0x70 | `emitter_param_0x14` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x14`; shipped 0 |
| +0x74 | `unread_74` | U | `data_load_def_tables` | - | shipped `0x101`; no reader (`car_load_particle_effects`, the table's only reader); the consumer passes the constant 1 for indices `0x15` and `0x16` |
| +0x78 | `unread_78` | U | `data_load_def_tables` | - | shipped 0, 1; no reader (`car_load_particle_effects`, the table's only reader) |
| +0x7c | `emitter_param_0x18` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x18`; shipped -20.0..-0.25 |
| +0x80 | `emitter_param_0x19` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x19`; shipped 0.25..20.0 |
| +0x84 | `emitter_param_0x1a` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x1a`; shipped 0.1..6.0 |
| +0x88 | `unread_88` | U | `data_load_def_tables` | - | shipped 0, 1; no reader (`car_load_particle_effects`, the table's only reader) |
| +0x8c | `emitter_param_0x1d[3]` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x1d`, by pointer; shipped 0..1.0 |
| +0x98 | `emitter_param_0x1e[3]` | M | `data_load_def_tables` | `car_load_particle_effects` | index `0x1e`, by pointer; shipped 0..1.0 |

`data_load_def_tables` writes the whole record by `dtaRead`, so it is every field's only writer.

## 3. Method family

2 of 2 PS2 functions mapped; 2 PC-only rows are the unit's CRT pair.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00592050 | `crt_static_init_def_tables` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard of the shared flag bit, then the pair's registrar (`structs/crt_runtime_tail.md`) |
| 0x00592070 | `crt_atexit_register_def_tables` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |

### 3b. Accessors

The unit has no accessor.

### 3c. Behaviour

The unit has no function with a receiver.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00592080 | `data_load_def_tables` | `LoadMaterialsFromBin__Fv` | 0x002e29a0 | CONFIRMED | Loads all four; returns 1 on success, 0 at the first failed open |
| 0x00592240 | `data_unload_def_tables` | `UnloadMaterials__Fv` | 0x002e2c40 | CONFIRMED | Frees all four and zeroes both pointer and count |

## 4. The load and the consumer of `particles.def`

`loadsave_init` calls `data_load_def_tables` once, as a bare statement that ignores the return value, and `loadsave_close` calls
`data_unload_def_tables` once; `engine_init` and `engine_shutdown` call those two, so the tables are loaded at start-up and freed
at shutdown.

The load is **not** transactional: `data_load_def_tables` returns 0 the moment a `dtaOpen` fails, leaving the tables loaded so far
in place. The four blocks are otherwise identical:

```
f = dtaOpen(name, 0);        if (f < 0) return 0;
if (table) operator delete(table);
size = dtaSeek(f, 0, 2);  dtaSeek(f, 0, 0);
table = operator new(size);  dtaRead(f, table, size);  dtaClose(f);
count = size / STRIDE;
```

`car_load_particle_effects` is the only consumer of `particles.def`. It walks every record once at start-up: it resolves the model
named in the record through the model cache, loading it when the cache has no match, creates one LS3D emitter frame (driver frame type
`0x11`, `I3D_emitor` in the shipped `LS3DF.dll`) and feeds the record's fields to it through vtable `+0x54`. The emitters are then
handed to `vehicle_set_particles`.

The value types are the shipped `LS3DF.dll`'s: its `I3D_emitor::SetParam` (`0x1003c460`, a 32-arm jump table; an index of `0x20` or
more prints `!! Invalid index:%u I3D_emitor::SetParam()`) stores indices `0x00`-`0x03`, `0x08`, `0x09`, `0x13` and `0x14` as dwords, the
other numbered indices of §1a as floats, and `0x15` / `0x16` as bytes. The game sets `0x15` and `0x16` to the constant 1 for every
record. Index `0x07` is the one mismatch: the game converts `+0x3c` to an integer with `__ftol` and passes that, while the DLL stores the
argument as a float, so the emitter receives the integer's bit pattern.

`prach`, `trava` and `listi` are Czech for dust, grass and leaves.

## 5. Naming and translation unit

* **Why `data_*_def_tables`, not `material_*`.** "Material" would be a claim about the whole set, and three of the four tables are not
  materials. The PC names state the set.
* **The unit** is delimited at both ends by MSVC static-init entries read out of the `.CRT$XCU` table at `.data:0x006461b4`:
  `crt_static_init_def_tables` at `0x00592050`, and the next unit's at `0x005922c0`. It is not part of the `structs/loadsave.md` unit
  `[0x00556ee0, 0x00561b40)` that holds its callers.

## 6. PC vs PS2

Both are **free functions** on PS2 — an empty class column — so a class-based census cannot
see them; they were found by enumerating the VA band with the class column ignored. The source
file name `g_material.cpp` comes from the PS2 `__sinit_` manifest. The developers' verb
`UnloadMaterials` is as loose as `material_*` would be; the PS2 symbol supplies the mapping, not
the meaning.

`data_load_def_tables` is statement-for-statement its twin: the same four blocks in the same
order, the same guarded delete before each, the same `dtaSeek(f,0,2)` / `dtaSeek(f,0,0)` size
probe, and `return 1` only after the fourth succeeds. `data_unload_def_tables` likewise — four
free-and-zero pairs in the load order.

**The PS2 body names all eight globals**:

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

## 7. Related

`Game-ghidra.exe.functions.md` (the function registry) · `structs/loadsave.md` (the callers `loadsave_init` / `loadsave_close`) ·
`structs/car.md` (`car_load_particle_effects`) · `structs/smoke_trail.md`, `def-files-docs/predmety.md`, `def-files-docs/materialy.md`.

## 8. Open items

| Item | State |
|---|---|
| 1. What each `SetParam` index controls | named by index in §1a; the 32 arms of `I3D_emitor::SetParam` and the emitter tick in `LS3DF.dll` are not decoded |
