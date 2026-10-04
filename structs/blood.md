# `C_blood` / `C_blood_manager` — the blood-drop and decal system

**Created** 2026-08-30 15:50 +0200 · **Modified** 2026-09-30 23:08 +0200

PC prefixes **`blood_`** and **`blood_manager_`**. This doc also owns the **decal pool** the drops
write into (§5), which serves bullet marks, grenade scorches and bloody prints as well.

| | |
|---|---|
| PS2 classes | `C_blood` (tag `7C_blood`), `C_blood_manager` (tag `15C_blood_manager`) |
| Manager location | **embedded in `C_game` at `+0x2d90`** — byte-proven on 12 of 12 callsites |
| PC functions | 6, all with CONFIRMED PS2 twins (§6); the manager's ctor/dtor are shared folds (§2) |
| PC translation unit | `[0x005adf80, 0x005ae480)` — 8 starts including the CRT guard pair |
| Polymorphic | no — neither class has a vtable on either build (§6) |

## 1. What it is — and what it is not

A `C_blood` is an **invisible ballistic marker**. The record is 0x24 bytes — a position, a
velocity, a size scale, a decal texture index and a flag — and it holds **no `I3D_frame`
and no model**. Nothing draws it. Its only visible output is the **decal it places on
whatever it lands on**.

That is why there is no visible animation of blood falling: the drop is launched from the
wound, falls under gravity, hits static geometry usually within a frame or two, stamps a
decal and dies.

Each spawn call emits **one** drop from the body's current position; nothing in the code lays
a path of drops.

In-game observation by the user:
> * blood appears for a moment around the point where the enemy was hit — that is a
>   separate particle effect, **not** this system;
> * a blood stain then appears **on the ground under the enemy's current position**, using
>   `2blood3.bmp`;
> * **there is no trail** of stains behind a wounded NPC that walks;
> * there is no visible falling animation.

## 2. Object layouts (PC)

```c
typedef struct C_blood {          // 0x24 bytes, heap, owned by the manager's vector
    /* +0x00 */ float    pos[3];      // [H] current position, integrated by blood_tick
    /* +0x0c */ float    vel[3];      // [H] velocity; vel.y takes gravity every tick
    /* +0x18 */ float    size;        // [H] decal size scale
    /* +0x1c */ int      texture;     // [H] index into C_game+0x29bc, the 10 decal textures
    /* +0x20 */ uint8_t  grow_in;     // [H] placer argument 10: the decal grows in (§5c)
} C_blood;

typedef struct C_blood_manager {  // embedded at C_game + 0x2d90
    /* +0x00 */ uint8_t  alloc;       // [H] STL allocator byte
    /* +0x04 */ C_blood **begin;      // [H] vector<C_blood*> — the live drops
    /* +0x08 */ C_blood **end;        // [H]
    /* +0x0c */ C_blood **cap;        // [H]
} C_blood_manager;                    // 0x10 bytes
```

The `C_blood` constructor has no address of its own: `blood_manager_add_blood` calls
`folded_return_this` (`0x005575f0`, body `mov eax,ecx; ret`) on the `operator_new(0x24)` result.
That body is a three-owner fold — 9 callsites take the `G_LoadSaveClass` singleton, 1 is this
`C_blood` constructor, and 1 takes a stack parameter in `recorder_sync_clock_from_system`.

The **manager's constructor and destructor** are the two-owner folds `folded_vector_ctor_at_0x04`
and `folded_vector_free_at_0x04` (the other owner is the `C_tshift` at `C_mission + 0x28`).
`game_ctor` calls the first with `lea ecx,[esi+0x2d90]` (`0x00606beb`), between the `+0x2d78`
vector and `police_manager_ctor`; `game_dtor` calls the second with the same receiver
(`0x006071a4`), at the mirror position.

## 3. The physics

`blood_tick(drop, dt_seconds)`:

1. `vel.y -= dt * 10.0` — gravity, 10 units/s².
2. **If `vel.y > 20.0`, return false** and the drop is destroyed.
3. Line-test `pos → pos + vel*dt` against **static** collision.
4. **Hit:** place a decal at the impact point, offset by half the surface normal, with a
   random size `0.5 + rand * drop.size * 0.3` and a random rotation `rand * 2π`, using the
   drop's texture index; the call registers the decal's collision sphere (argument 9 = 1) and
   passes the drop's `grow_in` byte as argument 10 (§5b). Return false — the drop is done.
5. **Miss:** `pos += vel*dt`, return true.

`blood_manager_tick` runs the swap-and-pop loop over every live drop each frame from
`game_tick`.

## 4. Who spawns drops, and with which texture

`blood_manager_add_blood(pos, dir, size, texture, flag)` — 10 callsites, every one a wound,
death or bleeding path, all reaching the manager as
`mov ecx,[reg+0x24]; add ecx,0x2d90` (the `C_game` singleton plus the embed offset):

| caller | texture | grow_in |
|---|---|---|
| `human_hit` (×2) | 0 — `2blood.bmp` | 0 |
| `dog_hit` (×2) | 0 | 0 |
| `pedestrian_hit` | 0 | 0 |
| `human_force_death` | 0 | 0 |
| `human_update` | 7 — `2blood3.bmp` | 1 |
| `human_movement` | 7 | 1 |
| `dog_enter_state` | 7 | 1 |
| `dog_update` | 7 | 1 |

Texture **0** is the hit/death spray and appears at full size; texture **7** is the stain that
appears under a wounded body — which is what the in-game observation reports — and it grows in
over 3000 ms (§5c).

## 5. The decal pool — `C_game + 0x75c`

One ring of **100 records of 0x58 bytes** at `C_game + 0x75c .. +0x29bc` serves every decal the
game places: blood, bullet marks, grenade scorches and bloody prints.

| `C_game` offset | what |
|---|---|
| `+0x75c` | the 100 records (§5a) |
| `+0x29bc` | the ten decal textures (§5d), loaded in `game_init`, released in `game_done` |
| `+0x29e4` | the ring cursor; `game_projector_advance_ring_cursor` steps it and wraps it at 100 |
| `+0x29e8` | the number of records still growing (§5c) |

### 5a. The record (`0x58` bytes)

| offset | field | writer / reader |
|---|---|---|
| `+0x00` | the `I3D_projector` object | created in `game_init`; configured by the placer |
| `+0x04` | a `0x48`-byte `tDynamicCollObject`: word 0 = type `0x48`, word 2 = `0x200001`, sphere centre `+0x10` = the impact point 0.5 lower, radius `+0x1c` = 0.3 | the placer, only with argument 9 set (blood decals) |
| `+0x4c` | collision object registered | the placer; erased when the slot is reused |
| `+0x4d` | growing | set by the placer (argument 10), cleared by the grow loop |
| `+0x50` | grow scale, 0.1 → 1.0 | the placer, the grow loop |
| `+0x54` | owner frame (argument 11) | read by `game_clear_projectors_for_frame`, which hides every decal bound to a frame |

### 5b. The placers

`game_create_track_effect` (scalar size) and `game_create_track_effect_between` (four vectors)
take the same eight stack arguments after their vectors; each slot was traced through the
disassembly. For the scalar form:

| # | type | meaning |
|---|---|---|
| 1 | `S_vector *` | position |
| 2 | `S_vector *` | surface normal |
| 3 | float | rotation angle |
| 4 | `S_vector *` | size |
| 5 | float | 0.3 blood, 0.002 shots, 0.2 grenade (§8) |
| 6 | int | texture index 0..9; refused when out of range or not loaded |
| 7 | bool | fit the orientation to the surface (`vec3_make_basis_from_direction` + four `vec3_dot_difference` probes) |
| 8 | float | alpha |
| 9 | bool | register the collision sphere of §5a |
| 10 | bool | grow in (§5c) instead of appearing at full size |
| 11 | `I3D_frame *` | owner frame, stored at `+0x54` |

The projector calls resolve against the shipped `LS3DF.dll` (`I3D_projector` vtable
`0x100987b8`): `+0x70` SetTexture, `+0x78` SetOrthogonal, `+0x7c` SetAlpha, `+0x84` SetColor,
`+0x88` SetMode, `+0x8c` SetProjectorLock. The colour is a grey level from a scene query at the
point. Without argument 10 the placer locks the projector at once with `SetProjectorLock(3)`.

### 5c. Grow-in, and a PC defect in it

With argument 10 set the placer starts the decal at 10 % size (`+0x4d` = 1, `+0x50` = 0.1,
`++C_game+0x29e8`). While `+0x29e8` is non-zero, `game_tick` walks all 100 records; each record
with `+0x4c` and `+0x4d` set grows by `0.0003 × dt_ms`, so it reaches full size in **3000 ms**.
At 1.0 the loop clears `+0x4d`, decrements `+0x29e8` and calls `SetProjectorLock(3)`.

That call indexes the record at the **ring cursor** `+0x29e4`, not the record that finished
growing (`0x005e26b6`): the grown decal is never locked, and the slot the next decal will reuse
is locked instead. Only grow-in decals reach this path, i.e. the texture-7 stains of §4 (on-screen
effect: §8).

### 5d. The ten textures and who places them

| index | asset | what it is | placed by |
|---|---|---|---|
| 0 | `2blood.bmp` | blood | `blood_tick` (§4) |
| 1 | `2flek.bmp` | a stain (Czech *flek*) | `game_create_grenade_track_effect` |
| 2 | `2dirbet.bmp` | bullet mark on concrete (*beton*) | `game_tick_shoot` |
| 3 | `2dirdre.bmp` | bullet mark on wood (*dřevo*) | `game_tick_shoot` |
| 4 | `2dirple.bmp` | bullet mark on sheet metal (*plech*) | `game_tick_shoot` |
| 5 | `2dirsklo.bmp` | bullet mark on glass (*sklo*) | `game_tick_shoot` |
| 6 | `2bfootprint.bmp` | a bloody footprint | `human_notify_callback` (§5e) |
| 7 | `2blood3.bmp` | blood, variant 3 | `blood_tick` (§4) |
| 8 | `2dogfoot.bmp` | a bloody dog paw print | `dog_place_bloody_paw_print` (§5e) |
| 9 | `skrob.tga` | a scrape | nothing — loaded, never placed |

`game_tick_shoot` takes the index from the hit surface's `materials.def` record `+0x3c`
(`def-files-docs/materialy.md`), so the surface decides the mark; materials with `-1` (grass,
foliage, water, body) get none. The producer list is a closed set: the two placers have five
`E8` callsites between them and no `E9` or data reference. Index 9 therefore has no producer.

### 5e. Bloody prints

1. Every blood decal (argument 9 = 1) carries a type-`0x48` collision sphere.
2. `dog_snap_to_ground` (every 200 ms) and `human_movement_intern` (the foot ray) set
   `g_collision_next_query_actor` to themselves around their downward ray.
3. `collision_line_in_dynamics` never reports a hit on type `0x48`; it calls the querying
   actor's vtable `+0x90` with the object (`structs/collision.md` §1d).
4. `human_arm_bloody_footprints` (effects quality `g_act_mission+0x9c` > 1 only) arms ten
   prints with texture 6; `dog_arm_bloody_paw_prints` arms ten prints and the print flag `+0x89`.
5. On each footstep animation event, `human_notify_callback` and `dog_place_bloody_paw_print`
   place one print through `game_create_track_effect_between`, with alpha
   `(steps left + 2) / total`, so the prints fade out over ten steps.

## 6. PC vs PS2

| PC VA | PC name | PS2 twin | PS2 size | PC size | status |
|---|---|---|---|---|---|
| 0x005adfb0 | `blood_tick` | `Tick__7C_bloodFf` | 472 | 369 | CONFIRMED |
| 0x005ae130 | `blood_init` | `Init__7C_bloodFR8S_vectorR8S_vectorfib` | 32 | 71 | CONFIRMED |
| 0x005ae180 | `blood_manager_game_init` | `GameInit__15C_blood_managerFv` | 8 | 34 | CONFIRMED |
| 0x005ae1b0 | `blood_manager_game_done` | `GameDone__15C_blood_managerFv` | 96 | 77 | CONFIRMED |
| 0x005ae200 | `blood_manager_tick` | `Tick__15C_blood_managerFUi` | 244 | 150 | CONFIRMED |
| 0x005ae2a0 | `blood_manager_add_blood` | `AddBlood__15C_blood_managerFR8S_vectorR8S_vectorfib` | 196 | 473 | CONFIRMED |
| - | - | `__ct__7C_bloodFv` | 8 | - | folded on PC into `folded_return_this` (§2) |
| - | - | `__dt__7C_bloodFv` | 68 | - | inlined on PC as a bare `operator_delete_wrapper` |
| - | - | `__ct__15C_blood_managerFv` | 52 | - | folded on PC into `folded_vector_ctor_at_0x04` (§2) |
| - | - | `__dt__15C_blood_managerFv` | 100 | - | folded on PC into `folded_vector_free_at_0x04` (§2) |

* **Record size differs:** PC 0x24 against PS2 0x30, because the PS2 pads `S_vector` to 16
  bytes. The manager differs too: the PS2 keeps a count + array, the PC a `std::vector`.
* **The `vel.y > 20.0` test** (§3 step 2): the PS2 decompiles it cleanly as `if (20.0 < fVar4)`,
  which settles Ghidra's x87 comparison output on the PC side.
* **Not polymorphic on either build.** The PS2 symbol table declares `__vt__8C_bottle` but no
  `__vt__7C_blood` or `__vt__15C_blood_manager`; the PS2 `__ct__15C_blood_managerFv` is the same
  allocator-plus-two-zeros vector constructor and `__dt__15C_blood_managerFv` the same free.
* **The PS2 draws no decals at all.** Both `CreateTrackEffect__6C_game…` overloads are 8-byte
  stubs, so blood drops, bullet marks and bloody prints are simulated but never placed there. The
  placer's argument order and types come from the mangled signature
  `(S_vector&, S_vector&, float, S_vector&, float, int, bool, float, bool, bool, I3D_frame*)`,
  and the projector slots of §5b were matched field for field to the PS2 `I3D_projector` setters
  (`SetOrthogonal` `+0x1e0` and `SetAlpha` `+0x1e8` on both builds; `SetProjectorLock` PS2
  `+0x200` ↔ PC `+0x1f8`, `SetMode` `+0x20c` ↔ `+0x204`, texture `+0x204` ↔ `+0x1fc`).
* **`BloodStep__5C_dogFUi`** is the twin of `dog_place_bloody_paw_print`: it places texture-8
  paw prints through `CreateTrackEffect`. `cbCollTester__5C_dogFP18tDynamicCollObject` / `cbCollTester__7C_humanFP18tDynamicCollObject`
  are the twins of the two arming callbacks of §5e.

## 7. Related

`structs/game.md` (the `+0x2d90` embed and the `+0x75c` pool) · `structs/collision.md` §1d
(dyn-coll type `0x48`) · `def-files-docs/materialy.md` (the per-surface decal index) ·
`structs/dog.md` (paw prints) · `structs/human.md` (footprints) ·
`basic/artifacts/hud_elements.md` §4 (the screen blood overlay, a different consumer of
`2blood.bmp`).

## 8. Open items

| item | state |
|---|---|
| bloody footprints (index 6) and dog paw prints (index 8) in game | the producers are traced (§5e); whether they appear in game at effects quality 2 after walking through a blood stain is not checked |
| placer argument 5 (§5b) | 0.3 for blood, 0.002 for shots, 0.2 for a grenade; its consumer is not traced |
| the misdirected `SetProjectorLock(3)` of §5c | what the lock changes on screen is not measured |
