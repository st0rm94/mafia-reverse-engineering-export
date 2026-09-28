# `bendable_plant` — entity kind `0x1d`

**Created** 2026-08-14 14:23 +0200 · **Modified** 2026-09-17 13:46 +0200

PC prefix **`bendable_plant_`**. 16/16 PC starts named.

| | |
|---|---|
| entity kind (`+0x10`) | **`0x1d`** |
| vtable | `.rdata 0x0063c608` |
| instance size | **`0xa0`** — `mission_create_actor` `case 0x1d` does `operator_new(0xa0)`, and the ctor's last field write ends at `+0x9c`; the layout closes exactly |
| base | `C_actor` (the ctor chains `actor_ctor`, the dtor chains `actor_dtor`) |
| translation unit | `[0x004af030, 0x004afe10)`, closed by the CRT pair `0x004afe10`/`0x004afe30` |
| PS2 class | none — PC-only (§6) |

> **The noun is the user's decision, not a measurement.** It rests on the measured mechanism in
> §1/§3: a patch of cloned `OBJ_` prototypes scattered by *uniform disc sampling*, ground-snapped by
> a vertical raycast, each running a damped Verlet spring whose tilt Y is **pinned to 1.0** so a
> clone leans and springs back but can never fall over, published as dyn-coll type `0x47` whose
> collision arm pushes a clone along the query direction and **never reports a hit**. Upright,
> ground-planted, non-blocking, bends when you walk through it and recovers — grass, reeds or a
> bush.
>
> Nothing in the binary names the cloned prop: the class clones whatever `OBJ_` child frames the
> designer parented to its own frame, and no string, table, material, asset or consumer says
> "plant". There is no PS2 symbol, no shipped scene uses kind `0x1d`, and no construction path is
> located (§5), so no in-game observation backs the noun either. The class name is the project's
> own — the PS2 build has no such class (compare `C_actor_track` and `C_recorder`, also
> project-invented names).
>
> **Not observed during gameplay; most likely unused in the shipped game** (§5).

## 1. What it is

A `C_actor` subclass whose single instance is a **clump**, not one object. It grows a patch of
cloned props around its own frame, publishes each clone as a dynamic-collision record, and then
runs a damped spring on every clone so that anything moving through the patch **bends** the clones
aside and they **spring back upright**.

There is **no wind and no idle sway anywhere in the class.** The only external writer of the
acceleration that `bendable_plant_ai` integrates is `bendable_plant_push_element`, so a clone moves
if and only if something physically passes through it.

## 2. Instance layout (`0xa0` bytes)

| Offset | Field | Conf | Notes |
|---|---|---|---|
| `+0x00` | vtable | H | `0x0063c608` |
| `+0x10` | entity kind | H | `0x1d` |
| `+0x65` | byte, set to 1 by `GameInit` | M | a `C_actor` field, not the class's own |
| `+0x68` | `I3D_frame*` | H | `C_actor`'s frame; its CHILD frames are the clone prototypes |
| `+0x70` | allocator byte of the element vector | H | - |
| `+0x74` / `+0x78` / `+0x7c` | **element vector** begin / end / cap | H | stride `0x74`; freed by the dtor, emptied by `GameDone` |
| `+0x80` | **scatter radius** (float) | H | ctor default `1.0f`; persisted; copied by `DuplicateFrom` |
| `+0x84` .. `+0x9c` | **seven clone counts** (uint) | H | one per child-frame index 0..6; persisted as one `0x1c`-byte block |

**Everything the designer sets is those eight values** — `SaveData` writes exactly `+0x80` (4
bytes) and `+0x84` (`0x1c` bytes) and nothing else, and `DuplicateFrom` copies exactly the same
pair. The clones are never persisted; `GameInit` rebuilds them from scratch.

## 3. The element record (stride `0x74`)

A dynamic-collision object with the class's own simulation tail. Byte-anchored from
`bendable_plant_game_init` (which writes it) and from `g_collision`'s `case 0x47` arm (which reads
it).

| Offset | Field | Notes |
|---|---|---|
| `+0x00` | dyn-coll type | **`0x47`** — this class's own type value |
| `+0x10` .. `+0x18` | world position | the ground-snapped point, `+0.2` on Y |
| `+0x1c` | radius | `0.4f` |
| `+0x44` | **owner** | back-pointer to the `bendable_plant` instance; read as `record[0x11]` by the collision arm |
| `+0x48` .. `+0x50` | **tilt vector** | the current lean; its Y component is pinned to `1.0` every tick |
| `+0x54` .. `+0x5c` | previous tilt | the Verlet history |
| `+0x60` .. `+0x68` | acceleration | written by the spring, and by the push callback |
| `+0x6c` | **spring stiffness** | per-clone random, `rand * 2/32768 + 0.25` ⇒ range 0.25 .. 2.25 |
| `+0x70` | `I3D_frame*` | the clone's own frame |

### 3a. How a patch is grown (`bendable_plant_game_init`)

For each child-frame index `i` in 0..6 whose frame type (`frame+0x110`) is **1** (an `OBJ_` model),
the prototype is cloned `*(this+0x84+4*i)` times. Each clone:

1. gets a fresh `OBJ_` frame from the I3D driver (FourCC `0x5f4a424f`);
2. is placed at `sqrt(rand * 1/32768) * radius` from the centre, at angle `rand * 0.00019175345`
   (= 2π/32767) — **uniform sampling of a disc**, not of a square;
3. is dropped onto the ground by `collision_test_line_vertical_thunk`;
4. gets the frame-move flag rewrite `& 0xfffffedf | 0x40000000`;
5. is published with `collision_dyn_insert`.

### 3b. How a clone bends and recovers

`bendable_plant_ai(dt)` per clone:

* **Verlet step** — `tilt += (tilt − prev) * 0.93 + accel * dt²`, then `prev = tilt`.
* **Restoring force** — `accel = normalize(−tilt) * 20.0 * stiffness`, a spring pulling the lean
  back to zero at the clone's own random stiffness.
* **Upright pin** — the tilt's Y component is forced to `1.0`, so the clone leans about its base and
  can never fall over.
* **Orientation** — an orthonormal basis is built from `(tilt.x, 1.0, tilt.z)` via `vec3_cross_to`
  and written with `S_matrix::SetDir3(frame+0x50)`, followed by the rotation-dirty rewrite
  `& 0xfffffed3 | 0x40000010`.

`bendable_plant_push_element(record, query)` is the only thing that ever disturbs it:

```
accel.x += query.dir.x * 100.0        // query +0x0c
accel.z += query.dir.z * 100.0        // query +0x14
// then both are scaled together so neither exceeds +-5.0 — the push DIRECTION survives the clamp
```

Its two callers are `collision_line_in_dynamics_inner` and `collision_sphere_in_dynamics`, from a
`case 0x47:` arm gated on `*(int *)(record[0x11] + 0x10) == 0x1d`. **Neither arm reports a hit** —
the objects never block movement or bullets. They are decoration that reacts.

That gate is also a second, independent byte-proof of the kind value: the collision code reaches
the owner through the record and tests `0x1d` there, with no reference to this class's code.

## 4. Function map (16/16)

| VA | Function | Slot | Role |
|---|---|---|---|
| `0x004af030` | `bendable_plant_ctor` | - | kind, vtable, radius `1.0f`, zeroes vector + counts |
| `0x004af080` | `bendable_plant_scalar_deleting_dtor` | 16 | - |
| `0x004af0a0` | `bendable_plant_dtor` | - | frees the element vector, chains `actor_dtor` |
| `0x004af0d0` | `bendable_plant_init` | 18 | refuses a null frame or a frame whose type is not 9 |
| `0x004af100` | `bendable_plant_save_data` | 19 | `+0x80` (4 B) + `+0x84` (`0x1c` B) |
| `0x004af150` | `bendable_plant_load_data` | 20 | the same pair, with remaining-bytes guards |
| `0x004af230` | `bendable_plant_duplicate_from` | 21 | copies `+0x80` and the seven counts |
| `0x004af270` | `bendable_plant_game_init` | 25 | grows the patch (§3a) |
| `0x004af6b0` | `bendable_plant_game_done` | 26 | erases every record, releases every clone frame |
| `0x004af710` | `bendable_plant_push_element` | - | the `g_collision` `case 0x47` callback (§3b) |
| `0x004af7b0` | `bendable_plant_ai` | 13 | the spring integrator (§3b) |
| `0x004afa80` | `crt_static_init_004afa80` | - | static-init entry, **no guard bit** |
| `0x004afa90` | `bendable_plant_static_list_ctor` | - | constructs the file-scope vector |
| `0x004afab0` | `crt_atexit_register_004afab0` | - | `atexit(&…_static_list_free)` |
| `0x004afac0` | `bendable_plant_static_list_free` | - | its atexit destructor |
| `0x004afae0` | `bendable_plant_element_vector_insert` | - | `vector<element>::insert(pos,n,val)`, stride `0x74` |

**No phantom starts.** Every one of the 16 starts (and the neighbouring TU) was checked for being an
alternate-exit block of an enclosing function — no `E8` callsite, no data reference, a local branch
into it, no padding boundary, no prologue — and none qualifies; `0x004af0dd` and `0x004af0eb` are
not function starts.

**`bendable_plant_element_vector_insert` belongs to this class.** The field offsets it indexes
(`+0x04`/`+0x08`/`+0x0c`) are those of a bare `std::vector`, but the census is **2 callsites, 0
datarefs, both inside `bendable_plant_game_init`** — not shared, so the instantiation is this
class's.

### 4a. The file-scope vector

`g_bendable_plant_list_allocator` / `_list` / `_list_end` / `_list_cap` (`0x00656060`..`0x0065606c`)
is a static member vector with the same head shape as the per-instance one. It is constructed by
`bendable_plant_static_list_ctor` and freed by `bendable_plant_static_list_free`, and **nothing else
in the binary reads or writes it** (§7).

## 5. No construction path is located

Two routes build an actor in this engine. Both dead-end.

**(a) Direct C++ construction** — `operator_new(size)` + `<class>_ctor`, which is how every
runtime-spawned class works (`pedestrian_crowd_ctor`, kind `0x13`, has exactly one real callsite and
never goes through the factory). **`bendable_plant_ctor` has exactly ONE callsite in the whole
decomp, and it is `case 0x1d:` of the entity factory.** No direct creator exists.

**(b) The factory `mission_create_actor(kind)`** — 7 callsites, all enumerated:

| caller | where the kind comes from | can it be `0x1d`? |
|---|---|---|
| `loadsave_savegame_load` | the scene's `0xAE22` special-object type | not in any of the 79 shipped scenes |
| savegame chunk 200 (×2) | read from the save chunk (with a `2 → 0x1b` remap) | only replays what a scene created |
| `program_process_command` | **copied from an existing actor** via `folded_get_dword_0x10` (its `C_actor::GetKind` role) | needs one to exist already |
| `loadsave_freeride_setup_scene` (×2) | disassembled: `push 2`, `push 4` | no — hard-coded player + car |
| `garage_game_init` | literal `4` | no |

**The scene census is corroboration, not the argument.** Special type `0x1d` occurs 0 times across
all 79 `scene2.bin` (80 mission directories; only `CARCYCLOPEDIA` has none) — but *ten kinds that
are certainly live never appear there either* (`3` C_human, `0xe` C_grenade, `0xf` C_smoke_trail,
`0x10` C_cartridge, `0x11` C_DropOut, `0x13` C_don_ocas, `0x1a` C_crash_obj, `0x1f` C_irenka, `0x25`
C_drop_in_weapon, `0x26` C_rail_generator), because they are all spawned at runtime by C++ code.
Absence from `scene2.bin` means only *"not scene-placed"* — a lower bound. Route (a) carries the
weight.

**In-game corroboration (user):** nothing in the game bends except the trees in the MISE09
thunderstorm — and the four MISE09 scenes contain special types `{1, 2, 4, 5, 6, 0x1b, 0x23}`, no
`0x1d`. That storm is a different subsystem (the `0xAEF0`/`0xAEF1` weather chunks plus the
`weather_setparam` script commands); the bending trees are most likely morph animation (§6).

**What would overturn this:** any sighting in game of a small object bending aside as you walk
through it and springing back. Then one of (a), (b) or the scene set is wrong, and (a) is the
cheapest to recheck.

## 6. PC vs PS2

**There is no PS2 side**, with evidence in both directions:

* **No PS2 constructor writes `0x1d`.** A sweep of every `__ct__*` symbol for its `+0x50` write
  returns 33 rows; none is this kind. (`0x1c` is `C_racecam`, `0x1e` is `C_wagon` — the neighbours
  on both sides are accounted for.)
* **No PS2 vtable sits where this one does.** Aligning the PC and PS2 vtable sequences in link order
  places PC vtable `0x0063c608` between `C_irenka` (`0x0063c568`) and `C_don_ocas` (`0x0063c6d8`);
  the PS2 sequence steps straight from `C_irenka` to `C_don_ocas`.
* **No PS2 source file fits.** The PS2 `__sinit_*` symbols (one per `.cpp`) list no file sorting
  between `C_irenka.cpp` and `C_movement.cpp`.

Every function in this class is graded on the PC body alone — there is no second body to diff — and
none carries a `PS2 twin:` line.

**The MISE09 storm** is not this class. The shipped models include `strom morph.4ds` /
`strom morph1.4ds` / `truckmorph.4ds` / `TE-hadicemorph.4ds`, and the PS2 carries
`__sinit_C_rmorf.cpp` plus a five-symbol class — `__ct__7C_rmorfFv`, `__dt__7C_rmorfFv`,
`Mission_Start__7C_rmorfFv`, `Mission_Done__7C_rmorfFv`, `Tick__7C_rmorfFi` — and
`LoadRAWSet__13RMRF_S_SETREFFPP13RMRF_S_SETREFPc`, which loads **RAW morph sets**. A mission-scoped
per-tick vertex-morph player is the shape that fits "two kinds of tree bending in a storm".

## 7. Open items

| Item | State |
|---|---|
| what the objects ARE | the mechanism (ground-rooted, springs upright, bends on contact, r=0.4, up to 7 prototypes) is certain; the noun "plant" is the user's decision, and nothing that creates an instance is located (§5) |
| the file-scope vector | §4a — no consumer, element type unknown |
| the source file name | the TU sorts alphabetically **between `C_irenka.cpp` and `C_movement.cpp`**, so the file begins `C_i…`..`C_m…`; the PS2 has nothing in that window, consistent with a PC-only file |
| `+0x65` | set to 1 by `GameInit`; a `C_actor` field with no documentation here |
| the `0x47` dyn-coll type | used by this class alone; `structs/collision.md` carries no type table, and `0x47` belongs in one with "no hit reported" against it |
| the PC side of the MISE09 morph player (§6) | no PC function is read for it; the PS2 `C_rmorf` symbols are a lead only |
