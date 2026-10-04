# `C_crash_obj` — the breakable scene prop (kind `0x1a`)

**Created** 2026-08-13 00:24 +0200 · **Modified** 2026-10-01 10:31 +0200

PC prefix **`crash_obj_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x1a`** |
| PC vtable | `.rdata 0x0063bf68` (39 slots) |
| PC constructor | `crash_obj_ctor` @ `0x004786c0` |
| PC instance size | `0x3e4` (`crash_obj_create_actor`'s `operator_new`) |
| PC translation unit | `[0x004776f0, 0x00479370)` — 28 starts, all named |
| Base class | **`C_PhThing`** (`structs/physical_object.md`) |
| PS2 class | `C_crash_obj`, 19 methods + 2 helper-class statics, **19/19 mapped** |

## 1. What it is

A static scene model that comes apart when something hits it — crates, fences, market
stalls, plaster, glass. It is **not** spawned by the entity factory: nothing constructs a
`C_crash_obj` except `crash_obj_create_actor`, which is reached from the class's public
entry point `crash_obj_create_from_model`.

That entry point has **four external callers**, and they are the whole feature:

| caller | what happened |
|---|---|
| `car_on_body_surface_contact` | a car drove into it |
| `human_ai` | a human punched or kicked it |
| `game_new_shoot` | it was shot |
| `game_destroy_control_signal` | destroys a control-signal (traffic-light) model |

The sequence is always the same: take the model frame that was hit, unhook it from the
traffic-light and telephone registries, disable every collision volume beneath it,
optionally swap the model for its `_c1.i3d` "cracked" variant, then create one
`C_crash_obj` actor per child frame of the replacement model and hand each one an
impulse. Each piece is a `C_PhThing`, so it falls and rolls on the physics solver until
it is culled.

### 1a. One of `C_PhThing`'s five subclasses

`crash_obj_ctor` chains `physical_object_ctor` and `crash_obj_dtor` chains
`physical_object_dtor`, so this class sits in the `C_PhThing` set alongside `C_box_new`,
`C_public_physics`, `C_drop_in_weapon` and `C_DropOut` (`structs/physical_object.md` §2, derived
from a caller census of the base constructor).

## 2. The frame tag is the configuration — and it lives in the shipped models

`crash_obj_init` reads the bound frame's user-property string, keeps everything after
the first `;`, upper-cases it, and looks up each letter with `strchr` — so **order in
the tag is irrelevant**. This is the class's entire configuration surface; there is no
`.def` file and no script command behind it.

| letter | effect |
|---|---|
| `G` | `flags \|= 0x02` — breaking-glass effect on death |
| `D` | `flags \|= 0x04` — debris burst |
| `F` | `flags \|= 0x08` — fire |
| `S` | `flags \|= 0x10` — debris flash |
| `E` | `flags \|= 0x20` — explosion |
| `+` | `flags \|= 0x40` — this piece can break into a FURTHER generation |
| `H` | calls `game_add_hydrant`, gated on detail level > 1 and a 150-unit player distance |
| `Z` | `+0x3e2 = 0` — skip the `C_PhThing` solver for this object |
| `X<f>` | health → `+0x3b4`, and `flags \|= 0x01` (the "has health" bit) |
| `M<f>` | mass scale → `+0x3b8`; divides the spawn impulse |
| `N<i>` | surface material id → `+0x3e1`, the byte `crash_obj_get_material_id` returns |
| `L<i>` | particle id → `+0x3e0`, and `flags \|= 0x80` (emit a dust particle on death) |
| `I<i>` | `+0x308` — the `C_PhThing` impact-sound column (`structs/physical_object.md` §4b) |
| `T<i>` | `+0x338`, stored as a bool — the impact sound may restart while still playing (`structs/physical_object.md` §4b) |

A second, separate tag grammar is read by `crash_obj_parse_model_swap_tag` from the same
string: a `.` anywhere in it means the model name already carries its extension (so the
`_c1` variant is **not** substituted), `Z` clears the caller's flag, and `P<digits>`
gives a piece count.

Surface material ids resolve through the materials table, `def-files-docs/materialy.md`.

## 3. Field layout

The `+0x3b0` band is `C_PhThing`'s; only the fields this class writes are listed.

```c
typedef struct C_crash_obj {          /* 0x3e4 bytes */
    /* +0x000 */ C_PhThing base;      // [H] kind 0x1a at +0x10, frame at +0x68
    /* +0x308 */ int    impact_sound_column;  // [H] C_PhThing field, set by the `I<i>` tag
    /* +0x338 */ bool   impact_sound_restart; // [H] C_PhThing field, set by the `T<i>` tag
    /* +0x3b0 */ uint   flags;        // [H] tag bits (above) + 0x8000 "destroy me"
    /* +0x3b4 */ float  health;       // [H] every impact subtracts the impulse; <0 = break
    /* +0x3b8 */ float  mass_scale;   // [H] divides the spawn impulse; tag default 10.0f
    /* +0x3bc */ char   generation;   // [H] ++ per break; formats the `%i.i3d` suffix
    /* +0x3c0 */ char  *model_name;   // [H] owned copy; freed by crash_obj_game_done
    /* +0x3c4 */ S_vector hit_point;  // [H] last contact point
    /* +0x3d0 */ S_vector hit_dir;    // [H] last contact direction (negated normal)
    /* +0x3dc */ float  hit_magnitude;// [H]
    /* +0x3e0 */ byte   particle_id;  // [H] `L<i>`; used by the flags-0x80 arm of AI
    /* +0x3e1 */ byte   material_id;  // [H] `N<i>`; returned by GetMaterialID
    /* +0x3e2 */ bool   solver_enabled; // [H] cleared by the `Z` tag
} C_crash_obj;
```

### 3a. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x308` | `impact_sound_column` | H | `crash_obj_init` | `physical_object_update` | a `C_PhThing` field; −1 = no impact sound (`structs/physical_object.md` §4b) |
| `+0x338` | `impact_sound_restart` | H | `crash_obj_init` | `physical_object_update` | a `C_PhThing` field (`structs/physical_object.md` §4b) |
| `+0x3b0` | `flags` | H | `crash_obj_init`, `crash_obj_destroy_outermost` (bit `0x8000`) | `crash_obj_ai`, `crash_obj_on_collision`, `crash_obj_hit`, `crash_obj_set_model_file_name` | bit `0x01` = has health; `0x8000` = destroy me |
| `+0x3b4` | `health` | H | `crash_obj_init`, `crash_obj_on_collision`, `crash_obj_hit` | `crash_obj_ai` | goes negative → the break-up ladder fires |
| `+0x3b8` | `mass_scale` | H | `crash_obj_init` | `crash_obj_create_actor` | `1/mass_scale` scales the spawn impulse |
| `+0x3bc` | `generation` | H | `crash_obj_ai`, `crash_obj_create_actor` | `crash_obj_ai` | the `%i.i3d` suffix counter |
| `+0x3c0` | `model_name` | H | `crash_obj_set_model_file_name`, `crash_obj_game_done` | `crash_obj_ai` | only stored when bit `0x40` is set |
| `+0x3e0` | `particle_id` | H | `crash_obj_init` | `crash_obj_ai` | passed to `particle_manager_create_particle` |
| `+0x3e1` | `material_id` | H | `crash_obj_init` | `crash_obj_get_material_id` | vtable slot 32 |
| `+0x3e2` | `solver_enabled` | H | `crash_obj_ctor`, `crash_obj_init` | `crash_obj_on_collision`, `crash_obj_hit` | `Z` tag clears it |

## 4. The break-up ladder (`crash_obj_ai`)

Per frame, in this exact order:

1. bit `0x8000` set → unregister and die (this is how `crash_obj_destroy_outermost` culls);
2. no health bit, or health still ≥ 0 → just run the `C_PhThing` solver and return;
3. otherwise, at the frame's world position: `0x08` fire (`game_new_fire`) · `0x20` explosion
   (`game_new_explosion`) · `0x02` breaking glass (`glass_shatter_break_frame`, with
   `lustr1.wav`; *lustr* = chandelier) · `0x10` a debris flash (`debris_generate_flash`) with a
   `rand()+20` particle count · `0x04` a debris burst (`debris_generate`);
4. bit `0x40` **and** a stored model name → build `<name>_<n>.i3d` and run the whole
   creation path again, so the piece breaks into a further generation;
5. bit `0x80` and detail level > 1 → a dust particle along the frame's up axis;
6. invalidate the actor and queue it for removal.

The glass and debris effects run on the two objects embedded in `C_game` at `+0x240`
(`C_Debris`, `structs/debris.md`) and `+0x280` (`C_Break_plain_glass`,
`structs/glass_shatter.md`).

## 5. Method coverage — 19/19 PS2 methods + both helper statics

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x00477720` | `thunk_crash_obj_disable_frame_colls` | the ICF alias ¹ | - | CONFIRMED |
| `0x00477730` | `crash_obj_disable_frame_colls` | `ModelColiisionsOFF__FP9I3D_frame` **+** `CollsOff__25S_hlp$1948C_crash_obj_cppFP9I3D_frame` | - | STRUCTURAL ¹ |
| `0x004777d0` | `crash_obj_create_crash_actors` | `CreateCrashActors__FP9I3D_frameR8S_vectorR8S_vectorfPcbUcibb` | - | CONFIRMED |
| `0x00477d50` | `crash_obj_create_actor` | `CreateActor__25S_hlp$2136C_crash_obj_cppF…` | - | CONFIRMED |
| `0x00478290` | `crash_obj_create_from_model` | `CreateCrashModel__FP9I3D_frameR8S_vectorR8S_vectorf` | - | CONFIRMED |
| `0x00478580` | `crash_obj_parse_model_swap_tag` | `DoChange__FP9I3D_frameRiRb` | - | CONFIRMED |
| `0x004786c0` | `crash_obj_ctor` | `__ct__11C_crash_objFv` | - | CONFIRMED |
| `0x00478730` | `crash_obj_scalar_deleting_dtor` | - | 16 | STRUCTURAL ² |
| `0x00478750` | `crash_obj_dtor` | `__dt__11C_crash_objFv` | - | CONFIRMED |
| `0x00478760` | `crash_obj_ai` | `AI__11C_crash_objFUi` | 13 | CONFIRMED |
| `0x00478c10` | `thunk_actor_save_game_get_size` | `SaveGameGetSize__11C_crash_objFv` | 22 | STRUCTURAL ³ |
| `0x00478c20` | `crash_obj_save_game_save` | `SaveGameSave__11C_crash_objFPPv` | 23 | CONFIRMED |
| `0x00478c40` | `crash_obj_save_game_load` | `SaveGameLoad__11C_crash_objFPPv` | 24 | CONFIRMED |
| `0x00478c60` | `crash_obj_init` | `Init__11C_crash_objFP9I3D_frame` | 18 | CONFIRMED |
| `0x00478f90` | `crash_obj_game_init` | `GameInit__11C_crash_objFv` | 25 | CONFIRMED |
| `0x00478fa0` | `crash_obj_game_done` | `GameDone__11C_crash_objFv` | 26 | CONFIRMED |
| `0x00478ff0` | `crash_obj_on_collision` | `Collision__11C_crash_objF…` | 30 | CONFIRMED |
| `0x00479120` | `crash_obj_hit` | `Hit__11C_crash_objF…` | 31 | CONFIRMED |
| `0x00479200` | `crash_obj_set_model_file_name` | `SetModelFileName__11C_crash_objFPc` | - | CONFIRMED |
| `0x00479270` | `crash_obj_destroy_outermost` | `DestroyOuterMostCrashObject__11C_crash_objFv` | - | CONFIRMED |
| `0x004793a0` | `crash_obj_get_material_id` | `GetMaterialID__11C_crash_objFP9I3D_frame` | 32 | CONFIRMED ⁴ |

¹ A 2→1 fold, §7. ² MSVC-only. ³ Empty PS2 body. ⁴ Physically emitted at the **head of the next
translation unit** (above `detector_ctor`) — a header-inline COMDAT the linker placed there. It is
`C_crash_obj`'s own override, not an ICF fold: exactly one `.rdata` dataref, in this class's slot 32.

**Role-only slots** (shared folded stubs that belong to no class — a role, not a twin; dataref
counts 225 / 43 / 43): `stub_rettrue_args4_00477710` at slots 19 and 20 (`SaveData` /
`LoadData`), `stub_ret0_cdecl_00477700` at slot 35, `stub_noop_args4_004776f0` at slots 36 and
37, and the shared forwarder `thunk_stub_noop_args4_00461180` at slot 21 (`DuplicateFrom`).

## 6. The global

`g_crash_obj_count` @ `0x00655ad0` — the live-piece counter. `crash_obj_game_init` increments it,
`crash_obj_game_done` decrements it, `crash_obj_destroy_outermost` early-outs on zero,
and `game_tick` calls that cull whenever the count exceeds **0x14 (20)**. So a
scene holds at most ~20 broken pieces, and the one culled is always the farthest from
the player.

## 7. PC vs PS2

* `g_crash_obj_count` mirrors the PS2 static class member `_count__11C_crash_obj`.
* **Field delta is a flat `+0x2b0`** on the class's own band: PC `+0x3b0`/`+0x3b4`/
  `+0x3bc`/`+0x3c0`/`+0x3e0`/`+0x3e1` against PS2 `+0x660`/`+0x664`/`+0x66c`/`+0x670`/
  `+0x678`/`+0x679`. Instance size PC `0x3e4` vs PS2 `0x680`.
* **Two PS2 symbols, one PC function.** `ModelColiisionsOFF` and `CollsOff` are the same
  code; GCC inlined one level of the recursion into the second, MSVC folded the pair and
  emitted the alias as a 5-byte jmp thunk.
* **Seven PS2 methods are 8-byte stubs** (`Update`, `NetDirect`, `SaveGameGetSize`,
  `DuplicateFrom`, `SaveData`, `LoadData`, `GetMaterialID`). On the PC six of them fold
  onto shared stubs; only `GetMaterialID` survives as its own function, because it
  returns a distinct value rather than a constant — the same reason
  `airplane_get_material_id` survives.
* **`C_I3D_cache<I3D_model>::Open` is a separate function on PS2 and inlined on the PC**,
  which accounts for the whole 1408 vs 2828-byte difference on `CreateCrashActors`.
* `DestroyOuterMostCrashObject` additionally calls `SetActState(0)` on PS2; the PC does
  not.
* The PS2 twins of the effect callees in §4: `game_new_fire` = `C_game::NewFire`,
  `game_new_explosion` = `C_game::NewExplosion`, `glass_shatter_break_frame` =
  `C_Break_plain_glass::BreakGlass`, `debris_generate_flash` / `debris_generate` =
  `C_Debris::GenerateFlash` / `GenerateDebris`, `game_add_hydrant` = `C_game::AddHydrant`.

## 8. Related

`structs/physical_object.md` (the base) · `structs/entity.md` §1 (the kind enum) ·
`structs/detector.md` (the next TU) · `structs/debris.md` · `structs/glass_shatter.md`.
