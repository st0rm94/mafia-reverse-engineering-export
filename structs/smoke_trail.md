# `C_smoke_trail` — entity kind `0xf`, the pieces an explosion throws

**Created** 2026-08-17 11:24 +0200 · **Modified** 2026-10-11 10:45 +0200

PC prefix **`smoke_trail_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0xf`** |
| PC vtable | `.rdata 0x0063cd80` |
| PC constructor | `smoke_trail_ctor` @ `0x004f60e0` |
| PC translation unit | `[0x004f60e0, 0x004f6270)` — 8 function starts, all named |
| Base class | **`C_bottle`** |
| PS2 class | `C_smoke_trail` (`13C_smoke_trail`), source file `C_smoke_trail.cpp`, 15 methods, all accounted (§3) |

`C_smoke_trail` is a **`C_bottle` subclass** — the same family as `C_cartridge`
(`structs/cartridge.md`), `C_grenade` (`structs/grenade.md`) and the bottle itself
(`structs/bottle.md`). It is a short-lived, self-retiring actor: an explosion throws it on
`C_bottle` flight physics, it shows one mesh through its own model frame, and it removes itself
from the world once its flight has stopped.

Every twin that owns a PC function of its own was body-diffed; the seven that do not are shared
stubs or folds and are listed in §3 with the reason.

## 1. Identity

| Fact | Value | How we know |
|---|---|---|
| Entity kind | **`0xf`** | `smoke_trail_ctor` writes `0xf` to `+0x10`, the kind field (`structs/entity.md` §1); the other build agrees (§5) |
| Vtable | `0x0063cd80` | installed by the constructor and re-installed by the destructor |
| Base class | `C_bottle` | every method chains its `bottle_*` counterpart first |
| PC translation unit | `[0x004f60e0, 0x004f6270)` | bounded by the CRT static-init/atexit pairs `0x004f60b0`/`0x004f60d0` and `0x004f6270`/`0x004f6290`, which is where the compiler emits one TU's boundary |
| PC starts | **8** | an enumeration of the function definitions in the annotated decomp over that range; none is a phantom of the aggressive instruction finder — each of the eight has direct callsites or a data reference, sits behind function-start padding, and begins with a prologue rather than a stack teardown |

## 2. Field map

Only the fields this class's own methods touch are listed; everything below `+0x84` is
`C_bottle`/`C_actor` state documented in those docs.

| PC offset | PS2 offset | Field | Conf | Evidence |
|---|---|---|---|---|
| `+0x10` | `+0x50` | entity kind = `0xf` | H | both constructors |
| `+0x65` | `+0x69` | `C_actor` save_excluded, set to 1 by `GameInit` | H | `smoke_trail_game_init` and its twin |
| `+0x68` | `+0x70` | the inherited `C_actor` model frame: an `'OBJ_'` frame showing the `2stone.i3d` mesh | H | `smoke_trail_update` calls its `Tick`; `smoke_trail_game_done` its `SetOn(false)`, `LinkTo(0, 0)` and `Release` (§4) |
| `+0x6d` | `+0x75` | inherited `may_broadcast`, cleared by the constructor | H | read by the host-role guard of the chained `bottle_ai_tick`, so a smoke trail never broadcasts |
| `+0x7c` | `+0xe0` | inherited `flight_speed` | H | `smoke_trail_ai_tick` retires when it and `+0x80` are both `<= 1e-08` |
| `+0x80` | `+0xe4` | inherited `fall_accum` | H | same test, plus the independent `> 50.0` retire gate |
| `+0x84` | `+0xe8` | inherited `landed_sets_state`, cleared by `GameInit` | H | read at flight end by the chained `bottle_ai_tick` |
| `+0xa4` | `+0xf0` | `0.1f` seed | M | `GameInit`; shared constant `0x3dcccccd` on both builds |
| `+0xa8` | `+0xb0` | inherited `frame_axis` x, cleared by `GameInit` | H | the chained `bottle_ai_tick` tumbles the axis and `bottle_update` passes it to `SetDir3` |
| `+0xac` | `+0xb4` | inherited `frame_axis` y, `1.0f` seed | H | `GameInit`; shared constant `0x3f800000`; same readers |
| `+0xb0` | `+0xb8` | inherited `frame_axis` z, cleared by `GameInit` | H | same readers |
| `+0xfc` | `+0x160` | inherited `coll_registered`, cleared by the constructor | H | the chained `bottle_update` skips the proxy update while it is clear, so a smoke trail registers no dynamic-collision proxy |

Confidence: **[H]** measured on both builds · **[M]** measured, meaning open. The inherited
fields are read by `bottle_ai_tick` and `bottle_update`, which `smoke_trail_ai_tick` and
`smoke_trail_update` chain first (`structs/bottle.md` §1).

The PC↔PS2 delta is **not uniform** across the class (`+0x10`→`+0x50` is `+0x40`,
`+0x7c`→`+0xe0` is `+0x64`, `+0xa4`→`+0xf0` is `+0x4c`). Do not rebase band-wise. Each row above
was matched by ROLE inside a body-diffed method.

## 3. Method coverage — 15/15 PS2 methods accounted

| PC VA | PC name | PS2 symbol | PS2 VA | Status | Notes |
|---|---|---|---|---|---|
| `0x004f60e0` | `smoke_trail_ctor` | `__ct__13C_smoke_trailFv` | `0x00211e90` | CONFIRMED | vtable + kind `0xf`; 48/68 = 0.71 |
| `0x004f6110` | `smoke_trail_scalar_deleting_dtor` | - | - | STRUCTURAL | MSVC-only; vtable slot 16, the family's sdd slot |
| `0x004f6130` | `smoke_trail_dtor` | `__dt__13C_smoke_trailFv` | `0x00211ee0` | CONFIRMED | the PS2 body also carries the `delete` arm MSVC splits into the sdd |
| `0x004f6140` | `smoke_trail_ai_tick` | `AI__13C_smoke_trailFUi` | `0x00211f70` | CONFIRMED | vtable slot 13; shared magics `1e-08` and `50.0`; 112/160 = 0.70 |
| `0x004f61b0` | `smoke_trail_update` | `Update__13C_smoke_trailFUi` | `0x00212010` | CONFIRMED | vtable slot 14; one virtual on the effect object |
| `0x004f61d0` | `thunk_bottle_net_apply_remote_state` | `NetDirect__13C_smoke_trailFUi` | `0x00212060` | STRUCTURAL | a 5-byte `jmp` to the `C_bottle` implementation; the PS2 symbol is an 8-byte tail jump |
| `0x004f61e0` | `smoke_trail_game_init` | `GameInit__13C_smoke_trailFv` | `0x00212070` | CONFIRMED | vtable slot 25; 96/84 = 1.14 |
| `0x004f6240` | `smoke_trail_game_done` | `GameDone__13C_smoke_trailFv` | `0x002120d0` | CONFIRMED | vtable slot 26; the same 3-call teardown |
| - | *(folded)* | `Init__13C_smoke_trailFP9I3D_frame` | `0x00211f40` | STRUCTURAL | realised by the shared slot-18 fold **`bottle_subclass_init_require_frame`** (`0x0048af90`) ¹ |
| - | *(shared stub)* | `SaveGameGetSize` / `SaveGameSave` / `SaveGameLoad` | `0x00212140`/`50`/`60` | STRUCTURAL | 8-byte PS2 stubs; the PC realisations are shared no-op stubs used by dozens of vtables |
| - | *(shared stub)* | `Collision` / `Hit` | `0x00212170`/`80` | STRUCTURAL | 8-byte `return 0;` on PS2 |
| - | *(shared stub)* | `SaveData` / `LoadData` | `0x00212190`/`a0` | STRUCTURAL | 8-byte `return 1;` on PS2 |

¹ A scan of the data sections for `0x0048af90` as a pointer shows 3 owning vtables:
`C_cartridge`, `C_grenade`, `C_smoke_trail`.

**None of the shared realisations carries a `PS2 twin:` line in its docblock.** Each is referenced
by many vtables, so a docblock field would assert an ownership the data-section scan refutes; the
many-to-one mapping belongs in this table.

## 4. Behaviour

**Where they come from.** `smoke_trail_ctor` has exactly one caller, `game_init`, which builds 32
smoke trails into a ring at `C_game+0x324` (cursor `+0x3a4`); the entity factory has no `0xf` arm.
`game_new_smoke_trail` re-arms the next one in the ring with a random direction and length, and its
only caller is `game_new_explosion`, which fires 8 + a random number of them per explosion when the
detail level is above 1 (`structs/game.md` §1). Each one gets an `'OBJ_'` frame sharing the mesh
of `2stone.i3d` (a single `Box02` mesh with the texture `0LASFALT.BMP`) and is initialised with
it through slot 18.

**Construction.** `smoke_trail_ctor` chains `bottle_ctor`, installs vtable `0x0063cd80`,
stamps entity kind `0xf` and clears two fields. `Init` (the shared fold) refuses a null
frame and otherwise delegates to the `C_bottle` initialiser — so a smoke trail is always
parented to a model frame.

**Arming.** `smoke_trail_game_init` chains `bottle_game_init`, then seeds the fade band:
`+0xa4 = 0.1f`, `+0xac = 1.0f`, three cleared words, and the save-exclusion byte `+0x65 = 1` (`structs/actor.md` §1b).

**Per frame.** `smoke_trail_update` advances the `C_bottle` half and then calls the model
frame's `Tick` (frame vtable `+0x14`). `smoke_trail_ai_tick` chains
`bottle_ai_tick` and applies the **retire test**, which has two independent arms:

```c
if ((+0x7c <= 1e-08 && +0x80 <= 1e-08) || 50.0 < +0x80) {
    game_invalidate_actor(this, 0);
    game_remove_temporary_actor(this);
}
```

— i.e. the flight has come to rest, **or** the fall accumulator has passed 50.0. No function of
the class or of its `C_bottle` base calls the particle manager.
`game_remove_temporary_actor` is the deferred retire queue (`structs/game.md`), so the actor
disappears at the end of the frame rather than mid-tick.

**Teardown.** `smoke_trail_game_done` chains `bottle_game_done` and then releases the
model frame with three virtual calls, in this order: `SetOn(false)` (`0x24`),
`LinkTo(0, 0)` (`0x2c`) and `Release` (`0x00`).

**A damaged car's smoke is not this class.** It comes from `vehicle_update_motor_particles`: once
the engine damage pool `C_Vehicle+0x234` drops below the threshold `+0x248` while the car is
slower than `+0x250`, the engine bay emits particle `+0x254`, and once the pool is also below
`+0x24c` it emits particle `+0x258` instead — the white, then black smoke a damaged car shows in
game (`structs/vehicle.md`). The emitters are the `tables\particles.def` records that
`car_load_particle_effects` builds (`structs/def_tables.md` §2a).

## 5. PC vs PS2

* **Field deltas are not uniform** — see the note under §2.
* **Kind stamp.** The PS2 constructor writes `0xf` to `+0x50`, the proven PS2 counterpart of the
  PC kind field `+0x10` (`structs/entity.md` §1).
* **Retire constants.** Both constants of the §4 retire test, `1e-08` and `50.0`, are byte-shared
  with the PS2 `AI` body.
* **Vtable slot numbering differs**: PC slot `0x14` ↔ PS2 slot `0x28` for the frame's
  per-frame call, and PC `0x24`/`0x2c`/`0x00` ↔ PS2 `0x38`/`0x40`/`0x08` for the teardown
  triple. The ORDER is identical on both builds. The PS2 `I3D_frame` vtable names all four:
  `Tick__9I3D_frameFi`, `SetOn__9I3D_frameFb`, `LinkTo__9I3D_frameFP9I3D_frameUi` and
  `Release__9I3D_frameFv`, on the PS2 actor frame field `+0x70`.
* **Seven of the fifteen PS2 methods are 8-byte symbols**, and each was read rather than
  counted: `NetDirect` is a tail jump (the PC mirrors it with a 5-byte `jmp` thunk), the
  `SaveGame*` triple and `SaveData`/`LoadData` return constants, and `Collision`/`Hit` return 0.
* **`Init` is a three-way ICF fold on the PC**, shared with `C_cartridge` and `C_grenade`;
  the PS2 emits it per class.
* MSVC splits the destructor into `__dt__` plus a scalar deleting destructor; the PS2
  toolchain emits one function with a flag argument. That is why the PC has 8 starts
  against 15 PS2 symbols and both counts are complete.

## 6. Related

`structs/bottle.md` — the base class, and the slot map this class inherits ·
`structs/cartridge.md` and `structs/grenade.md` — the sibling subclasses; all three share
`bottle_subclass_init_require_frame` · `structs/entity.md` §1 — the kind map, row `0xf` ·
`structs/game.md` — the deferred retire queue · `structs/vehicle.md` — the engine damage pool
that drives a damaged car's smoke.
