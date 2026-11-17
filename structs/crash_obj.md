# `C_crash_obj` — the breakable scene prop

**Created** 2026-08-13 00:24 +0200 · **Modified** 2026-10-28 23:29 +0100

A static scene model that comes apart when something hits it: crates, fences, market stalls,
plaster, glass. Each broken piece is a `C_crash_obj`, a `C_PhThing` that falls and rolls on the
physics solver until it is culled. The class is **not** built by the entity factory: nothing
constructs one except `crash_obj_create_actor`, reached from the class's entry points (§4).

| | |
|---|---|
| PC prefix | `crash_obj_` |
| PC kind (`+0x10`) | `0x1a`, stamped by `crash_obj_ctor` |
| PC vtable | `g_crash_obj_vt` @ `0x0063bf68`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_PhThing` (`structs/physical_object.md`), itself a `C_actor` |
| PC instance size | `0x3e4` (`operator_new(0x3e4)` in `crash_obj_create_actor`) |
| PC constructor | `crash_obj_ctor` @ `0x004786c0` |
| PC translation unit | `[0x004776c0, 0x00479370)`: the unit's CRT pair, three shared slot stubs of the `C_actor` family, then the `crash_obj_*` functions and three thunks; the next unit's guard `crt_static_init_guard_00479370` opens `C_detector`, whose head holds this class's `crash_obj_get_material_id` (`0x004793a0`) |
| PS2 class | `C_crash_obj` (tag `11C_crash_obj`), 19 methods: 13 CONFIRMED and 1 STRUCTURAL twins, 5 base stubs, all mapped in §3; plus 4 free functions and 2 statics of file-local helper structs |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_PhThing` base; offsets are PC.

```c
typedef struct C_crash_obj {                // 0x3e4 bytes; PC offsets
    /* +0x000 */ C_PhThing base;            // [H] the C_PhThing band (structs/physical_object.md)
    /* +0x3b0 */ uint32_t  flags;           // [H] the tag bits (§5) + 0x8000 "destroy me"
    /* +0x3b4 */ float     health;          // [H] every impact subtracts from it; < 0 = break
    /* +0x3b8 */ float     mass_scale;      // [H] divides the spawn impulse; 10.0 by default
    /* +0x3bc */ uint8_t   generation;      // [H] +1 per break; formats the `_<n>.i3d` suffix
    /* +0x3c0 */ char     *model_name;      // [H] owned copy; freed by crash_obj_game_done
    /* +0x3c4 */ S_vector  hit_point;       // [H] last contact point
    /* +0x3d0 */ S_vector  hit_dir;         // [H] last contact direction
    /* +0x3dc */ float     hit_magnitude;   // [H] last impact strength
    /* +0x3e0 */ uint8_t   particle_id;     // [H] the `L<i>` tag; the 0x80 arm of the break
    /* +0x3e1 */ uint8_t   material_id;     // [H] the `N<i>` tag; returned by GetMaterialID
    /* +0x3e2 */ uint8_t   solver_enabled;  // [H] cleared by the `Z` tag
} C_crash_obj;
```

The frame tag (§5) also configures two `C_PhThing` fields, which belong to the base band: `I<i>`
writes the impact-sound column `+0x308` and `T<i>` the restart flag `+0x338`
(`structs/physical_object.md` §1).

## 2. Field evidence

Measured by a census of every operand with each displacement in `.text` and of the decomp's
`this+N` accesses, attributed to the containing function. The `crash_obj_*` free functions write
the fields of the piece they create or cull, not of a `this`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `crash_obj_ctor` (through `physical_object_ctor`; kind `+0x10` = `0x1a`), `crash_obj_init` (C_PhThing `+0x308`, `+0x338` from the tag) | the `C_PhThing` methods of `structs/physical_object.md`; `physical_object_update` (`+0x308`, `+0x338`) | the inherited band; `+0x308` = −1 means no impact sound |
| +0x3b0 | flags | H | `crash_obj_init`, `crash_obj_destroy_outermost` (bit `0x8000`) | `crash_obj_init`, `crash_obj_ai`, `crash_obj_on_collision`, `crash_obj_hit`, `crash_obj_set_model_file_name`, `crash_obj_destroy_outermost` | bit `0x01` = has health; `0x8000` = destroy me (§6) |
| +0x3b4 | health | H | `crash_obj_init`, `crash_obj_on_collision`, `crash_obj_hit`, `crash_obj_create_actor` (−2.0 on a new piece) | `crash_obj_ai`, `crash_obj_on_collision`, `crash_obj_hit`, `crash_obj_create_actor` | goes negative → the break-up ladder fires (§6) |
| +0x3b8 | mass_scale | H | `crash_obj_ctor` (= 10.0), `crash_obj_init` | `crash_obj_create_actor` | `1/mass_scale` scales the spawn impulse |
| +0x3bc | generation | H | `crash_obj_ctor` (= 0), `crash_obj_create_actor`, `crash_obj_ai` | `crash_obj_ai` | the `_<n>.i3d` suffix counter |
| +0x3c0 | model_name | H | `crash_obj_ctor` (= 0), `crash_obj_set_model_file_name`, `crash_obj_game_done` | `crash_obj_ai`, `crash_obj_set_model_file_name`, `crash_obj_game_done` | only stored when bit `0x40` is set |
| +0x3c4 | hit_point | H | `crash_obj_ctor` (= 0), `crash_obj_on_collision`, `crash_obj_hit` | `crash_obj_ai` | `crash_obj_ai` passes its address on to the next generation |
| +0x3d0 | hit_dir | H | `crash_obj_ctor` (= 0), `crash_obj_on_collision`, `crash_obj_hit` | `crash_obj_ai` | the negated contact normal; `crash_obj_ai` passes its address on |
| +0x3dc | hit_magnitude | H | `crash_obj_ctor` (= 0), `crash_obj_on_collision`, `crash_obj_hit` | `crash_obj_ai` | - |
| +0x3e0 | particle_id | H | `crash_obj_ctor` (= 0), `crash_obj_init` | `crash_obj_ai` | passed to `particle_manager_create_particle` |
| +0x3e1 | material_id | H | `crash_obj_ctor` (= 0), `crash_obj_init` | `crash_obj_get_material_id` | vtable slot 32 |
| +0x3e2 | solver_enabled | H | `crash_obj_init`, `crash_obj_create_actor` (AND with the caller's flag) | `crash_obj_on_collision`, `crash_obj_hit`, `crash_obj_create_actor` | the `Z` tag clears it |

## 3. Method family

Coverage: all **19** PS2 `11C_crash_obj` methods are in the tables below: **13 CONFIRMED**,
**1 STRUCTURAL** (`SaveGameGetSize` on its thunk), and **5 base stubs** whose slots hold shared PC
functions (`Update`, `NetDirect`, `DuplicateFrom`, `SaveData`, `LoadData`). The four free functions
and the two helper-struct statics are in §3d, the two collision-off symbols on one PC function. The
scalar-deleting destructor has no PS2 twin.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004776c0 | `crt_static_init_guard_004776c0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x004776e0 | `crt_atexit_register_004776e0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x004786c0 | `crash_obj_ctor` | instance | `__ct__11C_crash_objFv` | 0x00149a50 | CONFIRMED | `physical_object_ctor`, the field defaults of §2, vtable `0x0063bf68`, kind `0x1a` |
| 0x00478730 | `crash_obj_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `crash_obj_dtor`, vtable slot 16 ¹ |
| 0x00478750 | `crash_obj_dtor` | instance | `__dt__11C_crash_objFv` | 0x00149ac0 | CONFIRMED | restores the vtable and chains `physical_object_dtor` |

¹ No standalone PS2 twin: GCC carries the free flag inside the destructor.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004793a0 | `crash_obj_get_material_id` | `GetMaterialID__11C_crash_objFP9I3D_frame` | 0x0014a870 | CONFIRMED | slot 32; `return material_id`. Emitted at the head of the next unit, a header-inline body the linker placed there; exactly one `.rdata` dataref, in this class's slot 32 |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__11C_crash_objFP7C_actor` | 0x0014a350 | base-stub | slot 21; a 5-byte `jmp` to the shared one-argument no-op, which nine `C_actor` vtables share; the PS2 body is an empty return |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__11C_crash_objFP7C_chunk` | 0x0014a850 | base-stub | slot 19; returns true, as the PS2 body does |
| 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__11C_crash_objFP7C_chunk` | 0x0014a860 | base-stub | slot 20; the same function as the row above |
| 0x00478760 | `crash_obj_ai` | `AI__11C_crash_objFUi` | 0x00149b20 | CONFIRMED | slot 13; the break-up ladder (§6) |
| 0x00478c10 | `thunk_actor_save_game_get_size` | `SaveGameGetSize__11C_crash_objFv` | 0x00149ef0 | STRUCTURAL | slot 22; a 5-byte `jmp` to `actor_save_game_get_size`; the 8-byte PS2 body is a tail-jump to the `C_actor` method, which Ghidra renders as that body ¹ |
| 0x00478c20 | `crash_obj_save_game_save` | `SaveGameSave__11C_crash_objFPPv` | 0x00149f00 | CONFIRMED | slot 23 |
| 0x00478c40 | `crash_obj_save_game_load` | `SaveGameLoad__11C_crash_objFPPv` | 0x00149f20 | CONFIRMED | slot 24 |
| 0x00478c60 | `crash_obj_init` | `Init__11C_crash_objFP9I3D_frame` | 0x00149f40 | CONFIRMED | slot 18; parses the frame tag (§5) |
| 0x00478f90 | `crash_obj_game_init` | `GameInit__11C_crash_objFv` | 0x0014a360 | CONFIRMED | slot 25; increments `g_crash_obj_count` (§7) |
| 0x00478fa0 | `crash_obj_game_done` | `GameDone__11C_crash_objFv` | 0x0014a390 | CONFIRMED | slot 26; frees `model_name`, decrements `g_crash_obj_count` |
| 0x00478ff0 | `crash_obj_on_collision` | `Collision__11C_crash_objFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x0014a410 | CONFIRMED | slot 30; records the contact, subtracts the impact from `health` |
| 0x00479120 | `crash_obj_hit` | `Hit__11C_crash_objF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0014a570 | CONFIRMED | slot 31; the same for a hit |
| 0x00479200 | `crash_obj_set_model_file_name` | `SetModelFileName__11C_crash_objFPc` | 0x0014a640 | CONFIRMED | stores an owned copy when bit `0x40` is set |
| 0x00479270 | `crash_obj_destroy_outermost` | `DestroyOuterMostCrashObject__11C_crash_objFv` | 0x0014a6c0 | CONFIRMED | a static: finds the piece farthest from the scene's active camera (`I3D_scene+0x17c`, `structs/i3d_frame.md`) and sets its bit `0x8000`; `game_tick` calls it while `g_crash_obj_count` exceeds 20 (§7) |
| 0x004cde80 | `thunk_physical_object_update` | `Update__11C_crash_objFUi` | 0x00149ed0 | base-stub | slot 14; a 5-byte `jmp` to `physical_object_update`, shared with one other vtable (`0x0063c9b8`); the 8-byte PS2 body is a tail-jump to the `C_PhThing` `Update` |
| 0x004cde90 | `thunk_physical_object_net_apply_remote_state` | `NetDirect__11C_crash_objFUi` | 0x00149ee0 | base-stub | slot 15; a 5-byte `jmp` to `physical_object_net_apply_remote_state`, shared with two other vtables; the PS2 body is an empty return (§8) |

¹ Ghidra names a thunk after its target: the Ghidra ids of the thunks in this section are
`thunk_FUN_00461180`, `thunk_FUN_0044c120`, `thunk_FUN_004bf260` and `thunk_FUN_004c6480`.

Slots 35–37 hold `stub_ret0_cdecl_00477700` and `stub_noop_args4_004776f0`, shared stubs of the
`C_actor` family that answer no `C_crash_obj` method (§3d).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004776f0 | `stub_noop_args4_004776f0` | - | - | - | the shared one-argument no-op of the `C_actor` family (225 datarefs), emitted at the head of this unit; slots 36 and 37 here (`structs/actor.md`) |
| 0x00477700 | `stub_ret0_cdecl_00477700` | - | - | - | the shared `return 0` of the `C_actor` family, emitted here; slot 35 here |
| 0x00477720 | `thunk_crash_obj_disable_frame_colls` | - | - | - | a 5-byte `jmp` to the function below, the alias MSVC kept when it folded the two PS2 symbols into one body (§8); its one caller is `crash_obj_create_from_model` |
| 0x00477730 | `crash_obj_disable_frame_colls` | `ModelColiisionsOFF__FP9I3D_frame` | 0x00147e10 | STRUCTURAL | disables every collision volume under a frame, recursively; also `CollsOff__25S_hlp$1948C_crash_obj_cppFP9I3D_frame` (`0x00148000`), a 2 → 1 fold (§8) |
| 0x004777d0 | `crash_obj_create_crash_actors` | `CreateCrashActors__FP9I3D_frameR8S_vectorR8S_vectorfPcbUcibb` | 0x001485c0 | CONFIRMED | one `C_crash_obj` per child frame of the (replacement) model, each handed an impulse; callers `crash_obj_create_from_model`, `crash_obj_ai` (the next generation), `game_destroy_control_signal` |
| 0x00477d50 | `crash_obj_create_actor` | `CreateActor__25S_hlp$2136C_crash_obj_cppFP9I3D_frameP9I3D_modelR8S_vectorP9I3D_framebPcUcfbRQ23std56vector<P11C_crash_obj,Q23std25allocator<P11C_crash_obj>>` | 0x001490f0 | CONFIRMED | `operator_new(0x3e4)`, `crash_obj_ctor`, then the piece's generation, health, solver flag and model name; a static of a file-local helper struct on the PS2 |
| 0x00478290 | `crash_obj_create_from_model` | `CreateCrashModel__FP9I3D_frameR8S_vectorR8S_vectorf` | 0x00149860 | CONFIRMED | the public entry point (§4) |
| 0x00478580 | `crash_obj_parse_model_swap_tag` | `DoChange__FP9I3D_frameRiRb` | 0x001483d0 | CONFIRMED | the second tag grammar (§5) |

## 4. What triggers a break

`crash_obj_create_from_model` has **four callers**, and with `game_destroy_control_signal` they are
the whole feature:

| caller | calls | what happened |
|---|---|---|
| `car_on_body_surface_contact` | `crash_obj_create_from_model` | a car drove into it |
| `human_ai` | `crash_obj_create_from_model` | a human punched or kicked it |
| `game_tick_shoot` | `crash_obj_create_from_model` | it was shot |
| `game_new_explosion` | `crash_obj_create_from_model` | an explosion reached it |
| `game_destroy_control_signal` | `crash_obj_create_crash_actors` | a control-signal (traffic-light) model is destroyed |

The sequence is always the same: take the model frame that was hit, unhook it from the
traffic-light and telephone registries, disable every collision volume beneath it, optionally
swap the model for its `_c1.i3d` "cracked" variant, then create one `C_crash_obj` actor per child
frame of the replacement model and hand each one an impulse.

### 4a. One of `C_PhThing`'s five subclasses

`crash_obj_ctor` chains `physical_object_ctor` and `crash_obj_dtor` chains `physical_object_dtor`,
so this class sits in the `C_PhThing` set alongside `C_box_new`, `C_public_physics`,
`C_drop_in_weapon` and `C_DropOut` (`structs/physical_object.md` §5, derived from a caller census
of the base constructor).

## 5. The frame tag is the configuration — and it lives in the shipped models

`crash_obj_init` reads the bound frame's user-property string, keeps everything after the first
`;`, upper-cases it, and looks up each letter with `strchr`, so **order in the tag is irrelevant**.
This is the class's entire configuration surface; there is no `.def` file and no script command
behind it.

| letter | effect |
|---|---|
| `G` | `flags \|= 0x02`: breaking-glass effect on death |
| `D` | `flags \|= 0x04`: debris burst |
| `F` | `flags \|= 0x08`: fire |
| `S` | `flags \|= 0x10`: debris flash |
| `E` | `flags \|= 0x20`: explosion |
| `+` | `flags \|= 0x40`: this piece can break into a FURTHER generation |
| `H` | calls `game_add_hydrant`, gated on detail level > 1 and a 150-unit player distance |
| `Z` | `solver_enabled = 0`: skip the `C_PhThing` solver for this object |
| `X<f>` | `health`, and `flags \|= 0x01` (the "has health" bit) |
| `M<f>` | `mass_scale`; divides the spawn impulse |
| `N<i>` | `material_id`, the byte `crash_obj_get_material_id` returns |
| `L<i>` | `particle_id`, and `flags \|= 0x80` (emit a dust particle on death) |
| `I<i>` | C_PhThing `+0x308`, the impact-sound column (`structs/physical_object.md` §1) |
| `T<i>` | C_PhThing `+0x338`, stored as a bool: the impact sound may restart while still playing |

A second, separate tag grammar is read by `crash_obj_parse_model_swap_tag` from the same string: a
`.` anywhere in it means the model name carries its extension (so the `_c1` variant is **not**
substituted), `Z` clears the caller's flag, and `P<digits>` gives a piece count.

Surface material ids resolve through the materials table, `def-files-docs/materialy.md`.

## 6. The break-up ladder (`crash_obj_ai`)

Per frame, in this exact order:

1. bit `0x8000` set → unregister and die (this is how `crash_obj_destroy_outermost` culls);
2. no health bit, or health still ≥ 0 → just run the `C_PhThing` solver and return;
3. otherwise, at the frame's world position: `0x08` fire (`game_new_fire`) · `0x20` explosion
   (`game_new_explosion`) · `0x02` breaking glass (`glass_shatter_break_frame`, with
   `lustr1.wav`; *lustr* = chandelier) · `0x10` a debris flash (`debris_generate_flash`) with a
   `rand()+20` particle count · `0x04` a debris burst (`debris_generate`);
4. bit `0x40` **and** a stored model name → build `<name>_<n>.i3d` and run the whole creation path
   again (`crash_obj_create_crash_actors`), so the piece breaks into a further generation;
5. bit `0x80` and detail level > 1 → a dust particle along the frame's up axis;
6. invalidate the actor and queue it for removal.

The glass and debris effects run on the two objects embedded in `C_game` at `+0x208`
(`C_Break_plain_glass`, `structs/glass_shatter.md`) and `+0x240` (`C_Debris`, `structs/debris.md`).

## 7. The global

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_crash_obj_count` | `0x00655ad0` | the live-piece counter | `crash_obj_game_init` (+1), `crash_obj_game_done` (−1), `game_init` (reset) | `crash_obj_game_init`, `crash_obj_game_done`, `crash_obj_destroy_outermost` (early-out on zero), `game_tick` |

`game_tick` calls `crash_obj_destroy_outermost` whenever the count exceeds **0x14 (20)**, so a
scene holds at most about 20 broken pieces, and the one culled is always the farthest from the
active camera.

## 8. PC vs PS2

* `g_crash_obj_count` mirrors the PS2 static class member `_count__11C_crash_obj`.
* **The field delta is a flat `+0x2b0`** on the class's own band: PC `+0x3b0` / `+0x3b4` /
  `+0x3bc` / `+0x3c0` / `+0x3e0` / `+0x3e1` against PS2 `+0x660` / `+0x664` / `+0x66c` / `+0x670`
  / `+0x678` / `+0x679`. Instance size PC `0x3e4` against PS2 `0x680`.
* **Two PS2 symbols, one PC function.** `ModelColiisionsOFF` and `CollsOff` are the same code; GCC
  inlined one level of the recursion into the second, MSVC folded the pair and emitted the alias as
  a 5-byte `jmp` thunk.
* **Seven PS2 methods are 8-byte bodies**: `Update` (a tail-jump to `C_PhThing::Update`),
  `NetDirect` (empty), `SaveGameGetSize` (a tail-jump to `C_actor`'s), `DuplicateFrom` (empty),
  `SaveData`, `LoadData` (return 1) and `GetMaterialID` (returns the byte at PS2 `+0x679`). Only
  `GetMaterialID` survives as its own PC function, because it returns a field rather than a
  constant (the same reason `airplane_get_material_id` survives); the others fill their slots with
  shared PC functions (§3c).
* **`NetDirect` differs in kind**: the PS2 body is empty, while the PC slot 15 reaches the
  `C_PhThing` network extrapolation. The PS2 port emptied the per-class network replication bodies
  (`structs/bottle.md` §9a).
* **`C_I3D_cache<I3D_model>::Open` is a separate function on the PS2 and inlined on the PC**, which
  accounts for the whole 1408 against 2828-byte difference on `CreateCrashActors`.
* `DestroyOuterMostCrashObject` additionally calls `SetActState(0)` on the PS2; the PC does not.
* The PS2 twins of the effect callees in §6: `game_new_fire` = `C_game::NewFire`,
  `game_new_explosion` = `C_game::NewExplosion`, `glass_shatter_break_frame` =
  `C_Break_plain_glass::BreakGlass`, `debris_generate_flash` / `debris_generate` =
  `C_Debris::GenerateFlash` / `GenerateDebris`, `game_add_hydrant` = `C_game::AddHydrant`.

## 9. Related

- `structs/physical_object.md` — `C_PhThing`, the base.
- `structs/entity_kind.md` — the kind enum (`0x1a`).
- `structs/detector.md` — the next unit, which holds `crash_obj_get_material_id`.
- `structs/debris.md`, `structs/glass_shatter.md` — the break effects.
- `structs/clock.md` — the previous unit, and the other user of the shared slot stubs.

## 10. Open items

The class has no open item.
