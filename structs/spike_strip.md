# `C_irenka` — the police spike strip (entity kind `0x1f`)

**Created** 2026-08-14 09:10 +0200 · **Modified** 2026-11-09 20:59 +0100

A static, deployable obstacle the police lay across a road. In gameplay a roadblock is a **composite**
of patrol cars (`police_manager` vec2), spike strips (vec3) and foot cops with Thompsons and shotguns
(vec4), and this class is only the strip. Its whole behaviour is: be placed, register a 2-D crossing
segment, and answer "did something drive over me". Its AI sets act state 1 and returns.

| | |
|---|---|
| PC prefix | `spike_strip_` |
| PC kind (`+0x10`) | `0x1f` |
| PC vtable | `g_spike_strip_vt` @ `0x0063c568` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x10c` (`operator_new(0x10c)` in `mission_create_actor`) |
| PC constructor | `spike_strip_ctor` @ `0x004ae830` |
| PC translation unit | `[0x004ae800, 0x004af000)`, guard to guard: 12 starts, all named |
| PS2 class | `C_irenka` (tag `8C_irenka`), 13 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base. The class also uses the kind `+0x10`,
the savegame exclusion byte `+0x65` (set by `spike_strip_game_init`) and the frame `+0x68`, the model
`barrier.4ds`.

```c
typedef struct C_irenka {                     // C_actor base +0x00..+0x70
    /* +0x70 */ uint8_t  owns_frame;          // [H] 1 on the police-built strips; GameDone releases the frame when set
    /* +0x74 */ tDynamicCollObject coll;      // [H] the collision record, 0x48 bytes
    /* +0xbc */ S_matrix inverse_world;       // [H] inverse of the frame's world matrix
    /* +0xfc */ float    end_a_x;             // [H] the strip's first end point, world x
    /* +0x100 */ float   end_a_z;             // [H] the first end point, world z
    /* +0x104 */ float   end_b_x;             // [H] the second end point, world x
    /* +0x108 */ float   end_b_z;             // [H] the second end point, world z
} C_irenka;
```

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions and by reading
all seven own bodies. The end-point writes of `spike_strip_activate` and the crossing test show that the
four floats are x and z: the test takes the `[0]` and `[2]` components of its two arguments.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | owns_frame | H | `police_manager_apply_state`, `spike_strip_game_init` | `spike_strip_game_done` | `spike_strip_game_init` stores 0, so a scene-placed strip keeps 0. `police_manager_apply_state` builds each pool strip (`operator_new(0x10c)`, `spike_strip_ctor`, Init and GameInit through the vtable), stores `+0x65` = 1 and this byte = 1, then deactivates it. `spike_strip_game_done` unlinks the frame and destroys it through its vtable slot 0 only when the byte is set, so the police-built strips own their frame |
| +0x74 | coll | H | `spike_strip_activate`, `spike_strip_game_init` | `collision_dyn_erase`, `collision_dyn_insert`, `spike_strip_activate`, `spike_strip_ctor`, `spike_strip_game_init` | type `0x40` at `+0x74`, frame at `+0x78`, flags `0x1000` at `+0x7c`, centre `+0x84`, radius `+0x90`, owner `+0xb8`; passed to `collision_dyn_insert` / `collision_dyn_erase` |
| +0xbc | inverse_world | H | `spike_strip_activate`, `spike_strip_game_init` (by address, `S_matrix::Inverse` in place; `spike_strip_game_init` first copies the frame's world matrix in with `rep movsd`) | `spike_strip_activate`, `spike_strip_game_init` | the frame's world matrix inverted; `coll` points at it from `+0xb4` |
| +0xfc | end_a_x | H | `spike_strip_activate`, `spike_strip_game_init` | `spike_strip_test_segment_crossing` | - |
| +0x100 | end_a_z | H | `spike_strip_activate` | `spike_strip_test_segment_crossing` | `spike_strip_game_init` leaves it untouched |
| +0x104 | end_b_x | H | `spike_strip_activate` | `spike_strip_test_segment_crossing` | `spike_strip_game_init` leaves it untouched |
| +0x108 | end_b_z | H | `spike_strip_activate`, `spike_strip_game_init` | `spike_strip_test_segment_crossing` | - |

## 3. Method family

13 of 13 PS2 methods mapped: 8 with a PC function (`Init` on the shared fold), 5 base stubs. PC-only: the
unit's static-init pair and `spike_strip_activate` (§8). A body shared by several classes has no single
owner, so none of the shared stubs carries a twin line. No start is an epilogue phantom: all were tested
for being an early-out block of an enclosing function (no `E8` callsite, no data reference, a branch into
it, no alignment padding before it, restore-fragment bytes) and none qualifies.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004ae800 | `crt_static_init_guard_004ae800` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004ae820 | `crt_atexit_register_004ae820` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x004ae830 | `spike_strip_ctor` | instance | `__ct__8C_irenkaFv` | 0x0018fe50 | CONFIRMED | `actor_ctor`, zeroes `coll`, vtable, kind `0x1f`; callers `mission_create_actor`, `police_manager_apply_state` |
| 0x004ae860 | `spike_strip_deleting_dtor` | instance | `__dt__8C_irenkaFv` | 0x0018feb0 | CONFIRMED | slot 16 |
| 0x004ae880 | `spike_strip_dtor` | instance | `__dt__8C_irenkaFv` | 0x0018feb0 | CONFIRMED | restores the vtable, `actor_dtor` |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `Update__8C_irenkaFUi` | 0x0018ff50 | base-stub | slot 14, the shared `ret 4` stub |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__8C_irenkaFUi` | 0x0018ff60 | base-stub | slot 15, the same stub |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameSave__8C_irenkaFPPv` | 0x00190380 | base-stub | slot 23, the shared default |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameLoad__8C_irenkaFPPv` | 0x00190390 | base-stub | slot 24, the same stub |
| 0x00462420 | `stub_ret0_cdecl_00462420` | `SaveGameGetSize__8C_irenkaFv` | 0x00190370 | base-stub | slot 22, the shared default |
| 0x004ae890 | `actor_subclass_init_require_frame` | `Init__8C_irenkaFP9I3D_frame` | 0x0018ff10 | STRUCTURAL | slot 18; a fold owned by no class (§6) |
| 0x004ae8b0 | `spike_strip_ai_tick` | `AI__8C_irenkaFUi` | 0x0018ff40 | CONFIRMED | slot 13; sets act state 1 |
| 0x004ae8c0 | `spike_strip_game_init` | `GameInit__8C_irenkaFv` | 0x0018ff70 | CONFIRMED | slot 25; fills `coll`, inserts it, computes the end points (§7) |
| 0x004aeb70 | `spike_strip_game_done` | `GameDone__8C_irenkaFv` | 0x00190300 | CONFIRMED | slot 26; `actor_game_done`, frame release while `owns_frame`, `collision_dyn_erase` |
| 0x004aebb0 | `spike_strip_deactivate` | `Deactivate__8C_irenkaFv` | 0x001903a0 | CONFIRMED | unlinks and hides the frame, `collision_dyn_erase`; callers `police_manager_apply_state`, `police_manager_free_temp_spike_strip` |
| 0x004aebe0 | `spike_strip_activate` | - | - | - | links and shows the frame, recomputes bounds and end points, `collision_dyn_insert`; caller `police_manager_deploy_temp_spike_strip` |
| 0x004aeda0 | `spike_strip_test_segment_crossing` | `ZapichameSi__8C_irenkaFRC8S_vectorRC8S_vector` | 0x00190410 | CONFIRMED | does a 2-D segment cross the strip; caller `car_on_body_surface_contact` |

### 3d. Free helpers

None.

## 4. The placement pipeline

`police_manager_deploy_temp_spike_strip` (`0x005ce8d0`) is the only creator path of a deployed strip:

1. `police_manager_spike_strip_pool_pop` takes a strip from `police_manager+0x60` (vec3);
2. the requested position is **ground-snapped** with `collision_test_line_vertical`, a `-5.0f` downward
   probe;
3. the position is written into the strip's `I3D_frame` at `+0x80/+0x84/+0x88` and the frame-move flag
   rewrite `& 0xfffffedf | 0x40000000` is applied;
4. **`spike_strip_activate`** commits it;
5. `actor_rebirth` + `game_add_temporary_actor_intern` publish it.

## 5. The vtable slot map (`0x0063c568`)

Identical in shape to the `C_dog` table: **13** AI · **14** Update · **15** NetDirect · **16** scalar
deleting destructor · **17** Release · **18** Init · **19/20** SaveData/LoadData · **21** DuplicateFrom ·
**22/23/24** the SaveGame triple · **25 GameInit** · **26 GameDone** · **29** Death · **30** Collision.
Slots 0–12 are the `C_actor_track` keyframe-recorder band (`structs/actor_track.md`).

`spike_strip_deactivate`, `spike_strip_activate` and `spike_strip_test_segment_crossing` hold **no** slot:
they are non-virtual and called directly.

## 6. `Init` is a fold owned by no class

`actor_subclass_init_require_frame` (`0x004ae890`) realises this class's `Init` at slot 18 (the body match
is in §8), but a scan of `.rdata` for the address as a pointer puts it at **slot 18 of six vtables**:
`0x0063b7c0` (`C_airplane`), `0x0063c0d0` (`C_dog`), `0x0063c568` (this class), `0x0063ca58` (`C_pumper`),
`0x0063cc98` (the seagull flock) and `0x0063cee0` (`C_turnout`).

So it is named for the **family** and carries **no `PS2 twin:` line**: a docblock cannot say "one of six".
It is a second, disjoint identical-code fold of the same body as `bottle_subclass_init_require_frame`
(3 vtables, the `C_bottle` subclasses).

## 7. The activate / deactivate pair

`spike_strip_activate` and `spike_strip_deactivate` are inverses, and `spike_strip_game_init` does the
collision half of the first:

| | activate | deactivate |
|---|---|---|
| scene | re-links the frame into the mission scene | unlinks (`+0x2c` with `0,0`) |
| visibility | frame vtable `+0x24` with **1** | frame vtable `+0x24` with **0** |
| bounds | `UpdateHRBoundVol`, the four end-point floats, `inverse_world`, centre y `+0x88` += 0.5 | - |
| collision | **`collision_dyn_insert(this+0x74)`** | **`collision_dyn_erase(this+0x74)`** |

## 8. PC vs PS2

* **Class name.** `C_irenka`: Czech *Irenka*, a developer's pet name; the police code calls the object an
  **Iris** (`FreeTempIris`, `S_road_block_iris`).
* **`Init` (§6).** `actor_subclass_init_require_frame` is statement-identical to PS2's
  `Init__8C_irenkaFP9I3D_frame` (`if (!frame) return false; return C_actor::Init(frame) != 0;`).
* **Field skew.** PC `+0x68/+0x70/+0x74/+0xb8` ↔ PS2 `+0x70/+0xc0/+0xd0/+0x118`; PS2's dyn-coll block is
  `0x60` bytes where the PC's is `0x48`, so **do not rebase band-wise**. PS2 writes the kind at `+0x50`, the
  PC at `+0x10`.
* **`owns_frame`.** PS2 `GameDone__8C_irenkaFv` tests `+0xc0` the same way before it releases the frame.
* **`ZapichameSi`** is Czech, roughly *"we're sticking it in"*, a developer joke rather than a description.
  The PC name states the behaviour (`spike_strip_test_segment_crossing`).
* **The container lever names the pool record.** `C_shvestky::S_road_block_iris` is the element type of the
  vec3 at `police_manager+0x60`: independent corroboration of the whole spike-strip reading, and a type name
  `structs/police_manager.md` carries untyped.
* **The PS2 never deploys a strip from the police side.** `FreeTempIris__10C_shvestkyFP8C_irenka` exists;
  `GetTempIris` does not, although `GetTempCar` and `GetTempEnemy` do. The PS2 instantiates no `insert` for
  `std::vector<C_shvestky::S_road_block_iris>`, only its allocator constructor, `clear` and destructor, so
  the pool is never filled (a vector that is filled, such as the `C_I3D_cache<I3D_model>` item vector,
  carries an `insert` instantiation). The PS2 roadblock builder
  `BugZamrdanychTesteruZTepleDve_410__10C_shvestkyFP7C_humaniR18S_police_interface` spawns cars and officers
  and places no strip. So `police_manager_spike_strip_pool_pop` and `police_manager_deploy_temp_spike_strip`
  have no PS2 counterpart, which is consistent with `spike_strip_activate` having no PS2 rank.

## 9. Related

`structs/police_manager.md` (the pool and the deploy path) · `structs/actor.md` (the base) ·
`structs/actor_track.md` (slots 0–12) · `structs/collision.md` (`tDynamicCollObject`) ·
`structs/seagull_flock.md` and the five other owners of the slot-18 fold.

## 10. Open items

None.
