# `C_smoke_trail` — entity kind `0xf`, the pieces an explosion throws

**Created** 2026-08-17 11:24 +0200 · **Modified** 2026-11-01 19:11 +0100

`C_smoke_trail` is a **`C_bottle` subclass**, the same family as `C_cartridge` (`structs/cartridge.md`),
`C_grenade` (`structs/grenade.md`) and the bottle itself (`structs/bottle.md`). It is a short-lived,
self-retiring actor: an explosion throws it on `C_bottle` flight physics, it shows one mesh through its
own model frame, and it removes itself from the world once its flight has stopped.

| | |
|---|---|
| PC prefix | `smoke_trail_` |
| PC kind (`+0x10`) | `0xf` |
| PC vtable | `g_smoke_trail_vt` @ `0x0063cd80` (39 slots) |
| PC base class | `C_bottle` (`structs/bottle.md`) |
| PC instance size | `0x138`, the size of `C_bottle` (no field past it is accessed) |
| PC constructor | `smoke_trail_ctor` @ `0x004f60e0` |
| PC translation unit | `[0x004f60b0, 0x004f6270)`, guard to guard: 10 starts, all named |
| PS2 class | `C_smoke_trail` (tag `13C_smoke_trail`), source file `C_smoke_trail.cpp`, 15 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class adds no field: every offset its functions touch is inside `C_bottle`.

```c
typedef struct C_smoke_trail {                // 0x138 bytes
    /* +0x00 */ C_bottle base;                // [H] the whole object; no own field
} C_smoke_trail;
```

### 1a. The base fields its functions touch

| Offset | Field | What this class does with it |
|---|---|---|
| `+0x10` | kind | the constructor writes `0xf` |
| `+0x65` | save-exclusion byte (`structs/actor.md` §2) | `smoke_trail_game_init` sets 1 |
| `+0x68` | the `C_actor` model frame | an `'OBJ_'` frame showing the `2stone.i3d` mesh; `smoke_trail_update` calls its `Tick`, `smoke_trail_game_done` releases it |
| `+0x6d` | `may_broadcast` | cleared by the constructor; the host-role guard of the chained `bottle_ai_tick` reads it, so a smoke trail never broadcasts |
| `+0x7c` | `flight_speed` | the retire test of §4 |
| `+0x80` | `fall_accum` | the retire test of §4 |
| `+0x84` | `end_sets_act_state` | cleared by `smoke_trail_game_init`; read at flight end by the chained `bottle_ai_tick` |
| `+0xa4` | `radius` | `smoke_trail_game_init` seeds 0.1 |
| `+0xa8` | `frame_axis` | seeded `(0, 1, 0)`; the chained `bottle_ai_tick` tumbles it and `bottle_update` passes it to `SetDir3` |
| `+0xfc` | `coll_registered` | cleared by the constructor; the chained `bottle_update` skips the proxy update while clear, so a smoke trail registers no dynamic-collision proxy |

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's six own functions; each
such offset is a row of §1a.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | base | H | `smoke_trail_ctor`, `smoke_trail_dtor`, `smoke_trail_game_init` | `smoke_trail_ai_tick`, `smoke_trail_game_done`, `smoke_trail_update` | the rows of §1a |

## 3. Method family

15 of 15 PS2 methods mapped: 7 base stubs and folds on shared functions, 8 with a function of their own;
3 PC-only starts (the unit's static-init pair and the scalar deleting destructor). Slot 21 keeps the
`C_bottle` default.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004f60b0 | `crt_static_init_guard_004f60b0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004f60d0 | `crt_atexit_register_004f60d0` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x004f60e0 | `smoke_trail_ctor` | instance | `__ct__13C_smoke_trailFv` | 0x00211e90 | CONFIRMED | `bottle_ctor`, vtable, kind `0xf`, clears `+0xfc` and `+0x6d` |
| 0x004f6110 | `smoke_trail_scalar_deleting_dtor` | instance | - | - | STRUCTURAL | slot 16, the family's scalar-deleting slot |
| 0x004f6130 | `smoke_trail_dtor` | instance | `__dt__13C_smoke_trailFv` | 0x00211ee0 | CONFIRMED | restores the vtable, `bottle_dtor` |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameSave__13C_smoke_trailFPPv` | 0x00212150 | base-stub | slot 23, the shared default |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameLoad__13C_smoke_trailFPPv` | 0x00212160 | base-stub | slot 24, the same stub |
| 0x00462420 | `stub_ret0_cdecl_00462420` | `SaveGameGetSize__13C_smoke_trailFv` | 0x00212140 | base-stub | slot 22, the shared default |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__13C_smoke_trailFP7C_chunk` | 0x00212190 | base-stub | slot 19, the shared default |
| 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__13C_smoke_trailFP7C_chunk` | 0x002121a0 | base-stub | slot 20, the same stub |
| 0x00482690 | `stub_retfalse_args24_00482690` | `Collision__13C_smoke_trailFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00212170 | base-stub | slot 30, the shared default |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__13C_smoke_trailF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00212180 | base-stub | slot 31, the shared default |
| 0x0048af90 | `bottle_subclass_init_require_frame` | `Init__13C_smoke_trailFP9I3D_frame` | 0x00211f40 | CONFIRMED | slot 18; the shared fold, also `C_cartridge` and `C_grenade` ¹ |
| 0x004f6140 | `smoke_trail_ai_tick` | `AI__13C_smoke_trailFUi` | 0x00211f70 | CONFIRMED | slot 13; `bottle_ai_tick`, then the retire test (§4) |
| 0x004f61b0 | `smoke_trail_update` | `Update__13C_smoke_trailFUi` | 0x00212010 | CONFIRMED | slot 14; `bottle_update`, then the frame's `Tick` |
| 0x004f61d0 | `thunk_bottle_net_apply_remote_state` | `NetDirect__13C_smoke_trailFUi` | 0x00212060 | STRUCTURAL | slot 15; a 5-byte `jmp` to `bottle_net_apply_remote_state`, also `C_cartridge`'s slot 15 |
| 0x004f61e0 | `smoke_trail_game_init` | `GameInit__13C_smoke_trailFv` | 0x00212070 | CONFIRMED | slot 25; `bottle_game_init`, then the seeds of §1 |
| 0x004f6240 | `smoke_trail_game_done` | `GameDone__13C_smoke_trailFv` | 0x002120d0 | CONFIRMED | slot 26; `bottle_game_done`, then releases the frame |

¹ A scan of the data sections for `0x0048af90` as a pointer shows 3 owning vtables: `C_cartridge`,
`C_grenade`, `C_smoke_trail`. Its docblock carries this twin and `C_grenade`'s. No other shared
realisation above carries a `PS2 twin:` line: each is referenced by many vtables, so a docblock field
would assert an ownership the scan refutes.

### 3d. Free helpers

None.

## 4. Behaviour

**Where they come from.** `smoke_trail_ctor` has exactly one caller, `game_init`, which builds 32
smoke trails into a ring at `C_game+0x324` (cursor `+0x3a4`); the entity factory has no `0xf` arm.
`game_new_smoke_trail` re-arms the next one in the ring with a random direction and length, and its
only caller is `game_new_explosion`, which fires 8 + a random number of them per explosion when the
detail level is above 1 (`structs/game.md` §3). Each one gets an `'OBJ_'` frame sharing the mesh
of `2stone.i3d` (a single `Box02` mesh with the texture `0LASFALT.BMP`) and is initialised with
it through slot 18.

**Construction.** `Init` (the shared fold) refuses a null frame and otherwise delegates to the
`C_bottle` initialiser, so a smoke trail is always parented to a model frame.

**Per frame.** `smoke_trail_update` advances the `C_bottle` half and then calls the model frame's
`Tick` (frame vtable `+0x14`). `smoke_trail_ai_tick` chains `bottle_ai_tick` and applies the **retire
test**, which has two independent arms:

```c
if ((+0x7c <= 1e-08 && +0x80 <= 1e-08) || 50.0 < +0x80) {
    game_invalidate_actor(this, 0);
    game_remove_temporary_actor(this);
}
```

That is, the flight has come to rest, **or** the fall accumulator has passed 50.0. No function of
the class or of its `C_bottle` base calls the particle manager. `game_remove_temporary_actor` is the
deferred retire queue (`structs/game.md`), so the actor disappears at the end of the frame rather than
mid-tick.

**Teardown.** `smoke_trail_game_done` chains `bottle_game_done` and then releases the model frame with
three virtual calls, in this order: `SetOn(false)` (`0x24`), `LinkTo(0, 0)` (`0x2c`) and `Release`
(`0x00`).

**A damaged car's smoke is not this class.** It comes from `vehicle_update_motor_particles`: once
the engine damage pool `C_Vehicle+0x234` drops below the threshold `+0x248` while the car is
slower than `+0x250`, the engine bay emits particle `+0x254`, and once the pool is also below
`+0x24c` it emits particle `+0x258` instead: the white, then black smoke a damaged car shows in
game (`structs/vehicle.md`). The emitters are the `tables\particles.def` records that
`car_load_particle_effects` builds (`structs/def_tables.md` §1a).

## 5. PC vs PS2

* **Field pairs**, each matched by role inside a body-diffed method; the delta is **not uniform**, so do
  not rebase band-wise:

| PC offset | PS2 offset | Field |
|---|---|---|
| `+0x10` | `+0x50` | kind |
| `+0x65` | `+0x69` | save-exclusion byte |
| `+0x68` | `+0x70` | model frame |
| `+0x6d` | `+0x75` | `may_broadcast` |
| `+0x7c` | `+0xe0` | `flight_speed` |
| `+0x80` | `+0xe4` | `fall_accum` |
| `+0x84` | `+0xe8` | `end_sets_act_state` |
| `+0xa4` | `+0xf0` | `radius` |
| `+0xa8` | `+0xb0` | `frame_axis` |
| `+0xfc` | `+0x160` | `coll_registered` |

* **Retire constants.** Both constants of the §4 retire test, `1e-08` and `50.0`, are byte-shared
  with the PS2 `AI` body, and the seeds `0.1` and `1.0` of `GameInit` are shared too.
* **Vtable slot numbering differs**: PC slot `0x14` ↔ PS2 slot `0x28` for the frame's per-frame call,
  and PC `0x24`/`0x2c`/`0x00` ↔ PS2 `0x38`/`0x40`/`0x08` for the teardown triple. The ORDER is identical
  on both builds. The PS2 `I3D_frame` vtable names all four: `Tick__9I3D_frameFi`, `SetOn__9I3D_frameFb`,
  `LinkTo__9I3D_frameFP9I3D_frameUi` and `Release__9I3D_frameFv`.
* **Seven of the fifteen PS2 methods are 8-byte symbols**, and each was read rather than counted:
  `NetDirect` is a tail jump (the PC mirrors it with a 5-byte `jmp` thunk), the `SaveGame*` triple and
  `SaveData`/`LoadData` return constants, and `Collision`/`Hit` return 0.
* **`Init` is a three-way ICF fold on the PC**, shared with `C_cartridge` and `C_grenade`; the PS2 emits
  it per class.
* MSVC splits the destructor into `__dt__` plus a scalar deleting destructor; the PS2 toolchain emits one
  function with a flag argument.

## 6. Related

`structs/bottle.md` (the base class and the slot map this class inherits) · `structs/cartridge.md` and
`structs/grenade.md` (the sibling subclasses; all three share `bottle_subclass_init_require_frame`) ·
`structs/entity_kind.md` §1 (row `0xf`) · `structs/game.md` (the ring and the deferred retire queue) ·
`structs/vehicle.md` (the engine damage pool behind a damaged car's smoke).

## 7. Open items

None.
