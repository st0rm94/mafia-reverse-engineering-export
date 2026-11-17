# `bendable_plant` — a patch of cloned props that bends and springs back (entity kind `0x1d`)

**Created** 2026-08-14 14:23 +0200 · **Modified** 2026-10-30 15:49 +0100

A `C_actor` subclass whose single instance is a **clump**, not one object: it grows a patch of cloned
props around its own frame, publishes each clone as a dynamic-collision record, and runs a damped
spring on every clone, so anything moving through the patch **bends** the clones aside and they
**spring back upright**. Nothing in the shipped game constructs one (§6).

| | |
|---|---|
| PC prefix | `bendable_plant_` |
| PC kind (`+0x10`) | `0x1d` |
| PC vtable | `g_bendable_plant_vt` @ `0x0063c608` |
| PC base class | `C_actor` (`structs/actor.md`): the ctor chains `actor_ctor`, the dtor `actor_dtor` |
| PC instance size | `0xa0` (`operator_new(0xa0)` in `mission_create_actor` `case 0x1d`; the last field ends at `+0xa0`) |
| PC constructor | `bendable_plant_ctor` @ `0x004af030` |
| PC translation unit | `[0x004af000, 0x004afe10)`, guard to guard: 18 starts, all named and all this class's |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

> **The noun is the user's decision, not a measurement.** It rests on the measured mechanism of §4:
> a patch of cloned `OBJ_` prototypes scattered by uniform disc sampling, ground-snapped by a
> vertical raycast, each running a damped Verlet spring whose tilt Y is pinned to 1.0 so a clone
> leans and springs back but can never fall over, published as dyn-coll type `0x47` whose collision
> arm pushes a clone along the query direction and never reports a hit. Upright, ground-planted,
> non-blocking, bends when you walk through it and recovers: grass, reeds or a bush. Nothing in the
> binary names the cloned prop (the class clones whatever `OBJ_` child frames the designer parented
> to its frame, and no string, table, material, asset or consumer says "plant"), no shipped scene
> constructs one, so no in-game observation backs the noun either, and there is no PS2 counterpart
> (§7). `C_actor_track` and `C_recorder` are project-invented names of the same kind.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base (`structs/actor.md` §1). The class
uses three base fields: the vtable `+0x00` and kind `+0x10` (written by the ctor), the savegame
exclusion byte `+0x65` (set to 1 by `bendable_plant_game_init`, so the savegame skips the patch)
and the frame `+0x68`, whose CHILD frames are the clone prototypes.

```c
typedef struct C_bendable_plant {             // 0xa0 bytes; C_actor base +0x00..+0x70
    /* +0x70 */ uint8_t                elem_alloc;   // [H] allocator byte of the element vector
    /* +0x74 */ S_bendable_element    *elem_begin;   // [H] one record per clone, stride 0x74 (§1a)
    /* +0x78 */ S_bendable_element    *elem_end;     // [H] one past the last clone's record
    /* +0x7c */ S_bendable_element    *elem_cap;     // [H] end of the element vector's storage
    /* +0x80 */ float                  radius;       // [H] scatter radius; ctor 1.0; persisted
    /* +0x84 */ uint32_t               clone_count[7]; // [H] clones per child-frame index 0..6; persisted
} C_bendable_plant;
```

**Why seven counts, and what each entry is.** Entry `i` is the number of clones grown from child
frame `i` of the actor's frame; `bendable_plant_game_init` walks `i` = 0..6 and clones the child
only when its frame type (`frame+0x110`) is 1, an `OBJ_` model. Seven is a designer limit:
the ctor zeroes seven dwords, `bendable_plant_duplicate_from` copies seven, and `SaveData` writes the
array as one `0x1c`-byte block. Nothing in the binary derives the number.

**Everything the designer sets is `radius` and `clone_count`.** `bendable_plant_save_data` writes
exactly `+0x80` (4 bytes) and `+0x84` (`0x1c` bytes), `bendable_plant_load_data` reads the same pair
and `bendable_plant_duplicate_from` copies the same pair. The clones are never persisted;
`bendable_plant_game_init` rebuilds them.

### 1a. `S_bendable_element` — one clone, `0x74` bytes

A dynamic-collision record with the class's own simulation tail. The type name is the project's;
there is no PS2 class to supply one.

```c
typedef struct S_bendable_element {           // 0x74 bytes
    /* +0x00 */ uint32_t          coll_type;     // [H] 0x47, this class's dyn-coll type
    /* +0x10 */ float             pos[3];        // [H] the ground-snapped point, +0.2 on Y
    /* +0x1c */ float             radius;        // [H] the record's collision radius, 0.4
    /* +0x44 */ C_bendable_plant *owner;         // [H] back-pointer the collision arm tests
    /* +0x48 */ float             tilt[3];       // [H] the current lean; Y pinned to 1.0
    /* +0x54 */ float             prev_tilt[3];  // [H] the Verlet history
    /* +0x60 */ float             accel[3];      // [H] spring force plus the push of §4b
    /* +0x6c */ float             stiffness;     // [H] per clone, rand * 2/32767 + 0.25 = 0.25 .. 2.25
    /* +0x70 */ I3D_frame        *frame;         // [H] the clone's own frame
} S_bendable_element;
```

The bytes between `+0x04` and `+0x10` and between `+0x20` and `+0x44` belong to the generic
dynamic-collision record (`structs/collision.md` §5c); this class writes none of them.

## 2. Field evidence

Measured by a census of every `this + N` access in the `bendable_plant_*` functions plus their
disassembly (the count array and the allocator byte are reached through aliases), and body reads
of the two collision callers.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | elem_alloc | H | `bendable_plant_ctor` | - | the vector header `bendable_plant_element_vector_insert` receives |
| +0x74 | elem_begin | H | `bendable_plant_ctor`, `bendable_plant_dtor`, `bendable_plant_element_vector_insert` | `bendable_plant_dtor` (frees it), `bendable_plant_game_init`, `bendable_plant_game_done`, `bendable_plant_ai` | - |
| +0x78 | elem_end | H | `bendable_plant_ctor`, `bendable_plant_dtor`, `bendable_plant_game_done` (= begin), `bendable_plant_element_vector_insert` | `bendable_plant_game_init`, `bendable_plant_game_done`, `bendable_plant_ai` | - |
| +0x7c | elem_cap | H | `bendable_plant_ctor`, `bendable_plant_dtor`, `bendable_plant_element_vector_insert` | `bendable_plant_element_vector_insert` | - |
| +0x80 | radius | H | `bendable_plant_ctor` (1.0), `bendable_plant_load_data`, `bendable_plant_duplicate_from` | `bendable_plant_save_data`, `bendable_plant_game_init`, `bendable_plant_duplicate_from` (its source) | - |
| +0x84 | clone_count | H | `bendable_plant_ctor` (zeroes seven), `bendable_plant_load_data`, `bendable_plant_duplicate_from` | `bendable_plant_save_data`, `bendable_plant_game_init`, `bendable_plant_duplicate_from` (its source) | §1 |

### 2a. `S_bendable_element`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | coll_type | H | `bendable_plant_game_init` | `collision_line_in_dynamics_inner`, `collision_sphere_in_dynamics` (their `case 0x47:` arms) | - |
| +0x10 | pos | H | `bendable_plant_game_init` | - | the generic dyn-coll readers are not censused (Open item 1) |
| +0x1c | radius | H | `bendable_plant_game_init` | - | Open item 1 |
| +0x44 | owner | H | `bendable_plant_game_init` | `collision_line_in_dynamics_inner`, `collision_sphere_in_dynamics` | both arms call the push only when `owner + 0x10` is `0x1d` |
| +0x48 | tilt | H | `bendable_plant_ai` | `bendable_plant_ai` | - |
| +0x54 | prev_tilt | H | `bendable_plant_ai` | `bendable_plant_ai` | - |
| +0x60 | accel | H | `bendable_plant_ai`, `bendable_plant_push_element` | `bendable_plant_ai`, `bendable_plant_push_element` | - |
| +0x6c | stiffness | H | `bendable_plant_game_init` | `bendable_plant_ai` | - |
| +0x70 | frame | H | `bendable_plant_game_init` | `bendable_plant_ai`, `bendable_plant_game_done` (releases it) | - |

## 3. Method family

The class is PC-only: 18 starts, no PS2 counterpart (§7).

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x004af000 | `crt_static_init_guard_004af000` | the translation unit | - | the unit's `.CRT$XCU` initializer |
| 0x004af020 | `crt_atexit_register_004af020` | the translation unit | - | its `atexit` registrar |
| 0x004af030 | `bendable_plant_ctor` | instance | - | `actor_ctor`, the vtable, kind `0x1d`, radius 1.0; empties the vector and zeroes the seven counts; one callsite, `mission_create_actor` `case 0x1d` |
| 0x004af080 | `bendable_plant_scalar_deleting_dtor` | instance | - | vtable slot 16 |
| 0x004af0a0 | `bendable_plant_dtor` | instance | - | frees the element vector, chains `actor_dtor` |
| 0x004afa80 | `crt_static_init_004afa80` | `g_bendable_plant_list` | - | the file-scope vector's initializer: calls the ctor and the registrar below; it has no guard bit |
| 0x004afa90 | `bendable_plant_static_list_ctor` | `g_bendable_plant_list` | - | constructs the file-scope vector (§5) |
| 0x004afab0 | `crt_atexit_register_004afab0` | `g_bendable_plant_list` | - | `atexit(&bendable_plant_static_list_free)` |
| 0x004afac0 | `bendable_plant_static_list_free` | `g_bendable_plant_list` | - | its `atexit` destructor |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00477700 | `stub_ret0_cdecl_00477700` | - | slot 22, `SaveGameGetSize`: returns 0, so the patch adds nothing to the savegame; a generic stub shared by 39 vtables |
| 0x00477710 | `stub_rettrue_args4_00477710` | - | slots 23 and 24, `SaveGameSave` / `SaveGameLoad`: a 2 → 1 fold onto one stub that returns true and touches nothing; shared by 17 vtables |
| 0x004af0d0 | `bendable_plant_init` | - | slot 18; refuses a null frame or a frame whose type is not 9, then `actor_init` |
| 0x004af100 | `bendable_plant_save_data` | - | slot 19; `+0x80` (4 bytes) and `+0x84` (`0x1c` bytes) |
| 0x004af150 | `bendable_plant_load_data` | - | slot 20; the same pair, each behind a remaining-bytes guard |
| 0x004af230 | `bendable_plant_duplicate_from` | - | slot 21; copies `+0x80` and the seven counts |
| 0x004af270 | `bendable_plant_game_init` | - | slot 25; grows the patch (§4a) |
| 0x004af6b0 | `bendable_plant_game_done` | - | slot 26; erases every record from the collision world and releases every clone frame |
| 0x004af7b0 | `bendable_plant_ai` | - | slot 13; the spring integrator (§4b) |

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x004af710 | `bendable_plant_push_element` | - | the `case 0x47` callback of the collision tests; its first argument is the element record (§4b) |
| 0x004afae0 | `bendable_plant_element_vector_insert` | - | `vector<S_bendable_element>::insert(pos, n, value)`, stride `0x74`; its `this` is the vector. 2 callsites, both in `bendable_plant_game_init`, no data reference, so the instantiation is this class's |

## 4. How the patch works

There is **no wind and no idle sway anywhere in the class.** The only external writer of the
acceleration `bendable_plant_ai` integrates is `bendable_plant_push_element`, so a clone moves if
and only if something physically passes through it.

### 4a. How a patch is grown (`bendable_plant_game_init`)

For each child-frame index `i` in 0..6 whose frame type (`frame+0x110`) is **1** (an `OBJ_` model),
the prototype is cloned `clone_count[i]` times. Each clone:

1. gets a fresh `OBJ_` frame from the I3D driver (FourCC `0x5f4a424f`);
2. is placed at `sqrt(rand * 1/32768) * radius` from the centre, at angle `rand * 0.00019175345`
   (= 2π/32767): **uniform sampling of a disc**, not of a square;
3. is dropped onto the ground by `collision_test_line_vertical_thunk`;
4. gets the frame-move flag rewrite `& 0xfffffedf | 0x40000000`;
5. is published with `collision_dyn_insert`.

### 4b. How a clone bends and recovers

`bendable_plant_ai(dt)` per clone:

* **Verlet step**: `tilt += (tilt − prev) * 0.93 + accel * dt²`, then `prev = tilt`.
* **Restoring force**: `accel = normalize(−tilt) * 20.0 * stiffness`, a spring pulling the lean back
  to zero at the clone's own random stiffness.
* **Upright pin**: the tilt's Y component is forced to `1.0`, so the clone leans about its base and
  can never fall over.
* **Orientation**: an orthonormal basis is built from `(tilt.x, 1.0, tilt.z)` via `vec3_cross_to`
  and written with `S_matrix::SetDir3(frame+0x50)`, followed by the rotation-dirty rewrite
  `& 0xfffffed3 | 0x40000010`.

`bendable_plant_push_element(record, query)` is the only thing that ever disturbs it:

```
accel.x += query.dir.x * 100.0        // query +0x0c
accel.z += query.dir.z * 100.0        // query +0x14
// then both are scaled together so neither exceeds +-5.0: the push DIRECTION survives the clamp
```

Its two callers are `collision_line_in_dynamics_inner` and `collision_sphere_in_dynamics`, from a
`case 0x47:` arm gated on `*(int *)(record[0x11] + 0x10) == 0x1d`; the sphere test also requires the
byte `+0x2c0` of its first argument. **Neither arm reports a hit**: the clones never block movement
or bullets. That gate is also a second, independent byte-proof of the kind value: the collision code
reaches the owner through the record and tests `0x1d` there, with no reference to this class's code.

## 5. The file-scope vector

`g_bendable_plant_list_allocator` / `_list` / `_list_end` / `_list_cap` (`0x00656060`..`0x0065606c`)
is a static member vector with the same head shape as the per-instance one. It is constructed by
`bendable_plant_static_list_ctor` and freed by `bendable_plant_static_list_free`, and **nothing else
in the binary reads or writes it**: a census of the four names finds only those two functions. The
free routine runs no element destructor, so the element type is trivially destructible; nothing else
about it is recoverable.

## 6. Nothing constructs it

Two routes build an actor in this engine. Both dead-end.

**(a) Direct C++ construction**: `operator_new(size)` + `<class>_ctor`, which is how every
runtime-spawned class works (`pedestrian_crowd_ctor`, kind `0x13`, has exactly one real callsite and
never goes through the factory). **`bendable_plant_ctor` has exactly ONE callsite in the whole
decomp, and it is `case 0x1d:` of the entity factory.** No direct creator exists.

**(b) The factory `mission_create_actor(kind)`**: 8 callsites, all enumerated by a census of the
`E8` calls to it:

| Caller | Where the kind comes from | Can it be `0x1d`? |
|---|---|---|
| `mission_open_bin` | the scene's `0xAE22` object kind | not in any of the 79 shipped scenes |
| `game_load_differences` (×2) | the `.chg` difference file: chunk 200 (with a `2 → 0x1b` remap) and chunk 400 (`structs/chunk.md` §4b) | not in any of the 92 shipped files, which carry kinds 1, 2, 4, 5, 6, 9, `0x1b` and `0x21` |
| `loadsave_savegame_load` | the savegame | only replays actors a running game created |
| `program_process_command` | copied from an existing actor via `folded_get_dword_0x10` (its `C_actor::GetKind` role) | needs one to exist first |
| `loadsave_freeride_setup_scene` (×2) | disassembled: `push 2`, `push 4` | no: hard-coded player + car |
| `garage_game_init` | literal `4` | no |

**The scene census is corroboration, not the argument.** Special type `0x1d` occurs 0 times across
all 79 `scene2.bin` (80 mission directories; only `CARCYCLOPEDIA` has none), but ten kinds that are
certainly live never appear there either (`3` C_human, `0xe` C_grenade, `0xf` C_smoke_trail, `0x10`
C_cartridge, `0x11` C_DropOut, `0x13` C_don_ocas, `0x1a` C_crash_obj, `0x1f` C_irenka, `0x25`
C_drop_in_weapon, `0x26` C_rail_generator), because they are all spawned at runtime by C++ code.
Absence from `scene2.bin` means only "not scene-placed", a lower bound. Route (a) carries the
weight.

**In-game corroboration (user):** nothing in the game bends except the trees in the MISE09
thunderstorm, and the four MISE09 scenes contain special types `{1, 2, 4, 5, 6, 0x1b, 0x23}`, no
`0x1d`. That storm is a different subsystem (the `0xAEF0` / `0xAEF1` weather chunks plus the
`weather_setparam` script commands), and the shipped data ties tree bending in MISE09 to the
random-morph module (`structs/random_morph.md`): `MISE09-PREJIMKA`'s `rmorf.bin` drives the morph
channels of 14 tree frames (`s_oranz*` / `s_barevny*`, placed by the same mission's `scene2.bin`), two
juniper frames and two laundry frames, and `MISE09-KRAJINA`'s drives `Truck.Mesh00`.

**What would overturn this:** any sighting in game of a small object bending aside as you walk
through it and springing back. Then one of (a), (b) or the scene set is wrong, and (a) is the
cheapest to recheck.

## 7. PC vs PS2

**There is no PS2 side**, with evidence in both directions:

* **No PS2 constructor writes `0x1d`.** A sweep of every `__ct__*` symbol for its `+0x50` write
  returns 33 rows; none is this kind. (`0x1c` is `C_racecam`, `0x1e` is `C_wagon`: the neighbours on
  both sides are accounted for.)
* **No PS2 vtable sits where this one does.** Aligning the PC and PS2 vtable sequences in link order
  places PC vtable `0x0063c608` between `C_irenka` (`0x0063c568`) and `C_don_ocas` (`0x0063c6d8`); the
  PS2 sequence steps straight from `C_irenka` to `C_don_ocas`.
* **No PS2 source file fits.** The PS2 `__sinit_*` symbols (one per `.cpp`) list no file sorting
  between `C_irenka.cpp` and `C_movement.cpp`, which is where the PC link order puts this translation
  unit. `Game.exe` carries no source-file strings at all, so the file's name is not recoverable from
  either build.

Every function in this class is graded on the PC body alone (there is no second body to diff), and
none carries a `PS2 twin:` line.

**The MISE09 storm** is not this class. The shipped models include `strom morph.4ds` /
`strom morph1.4ds` / `truckmorph.4ds` / `TE-hadicemorph.4ds`, and the PS2 carries
`__sinit_C_rmorf.cpp` plus a five-symbol class (`__ct__7C_rmorfFv`, `__dt__7C_rmorfFv`,
`Mission_Start__7C_rmorfFv`, `Mission_Done__7C_rmorfFv`, `Tick__7C_rmorfFi`) and
`LoadRAWSet__13RMRF_S_SETREFFPP13RMRF_S_SETREFPc`, which loads **RAW morph sets**: a mission-scoped
per-tick vertex-morph player. Its PC side is `random_morph_*` (`structs/random_morph.md`), and the
MISE09 `rmorf.bin` files of §6 are what it plays.

## 8. Related

- `structs/actor.md` — the `C_actor` base, its frame `+0x68` and the savegame exclusion `+0x65`.
- `structs/collision.md` §5c — the dynamic-collision record and type `0x47`.
- `structs/entity_kind.md` §1 — kind `0x1d`.
- `structs/random_morph.md` — the MISE09 tree bending, a different subsystem.

## 9. Open items

1. **The element's generic collision fields:** which collision functions read `pos` (`+0x10`) and
   `radius` (`+0x1c`) of a type-`0x47` record is not censused; the cells are `-`.
