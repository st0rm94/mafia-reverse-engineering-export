# `C_airplane` — the scripted aeroplane actor

**Created** 2026-08-04 13:27 +0200 · **Modified** 2026-09-17 17:52 +0200

A `C_actor` subclass: the aeroplane the player shoots
down from a moving car in **MISE16-LETISTE** (Czech *letiště* = airport). It flies a route
over the nav web, takes damage into a HUD gauge, carries six dynamic collision proxies and
two particle effects, and is fully data-driven from the scene chunk.

- **PC prefix:** `airplane_`.
- **Base class:** `C_actor` (`structs/actor.md`). `airplane_ctor` → `actor_ctor`,
  `airplane_dtor` → `actor_dtor`, `airplane_game_init`/`_game_done` chain to their base first.
- **kind (`+0x10`) = `0x16`.** Byte-proven from the ctor.
- **PC vtable:** `.rdata 0x0063b7c0`.
- **PC translation unit:** `[0x0044ef50, 0x00453a20)` — **33 function starts**. Bounded at
  *both* ends by a CRT static-init/atexit pair (`0x0044ef20` opens it, `0x00453a20` closes
  it); see §2.

---

## 1. Field layout — the tunable band

Four independent sources agree on this band: the ctor seeds it, `airplane_duplicate_from`
copies exactly these 16 values, `airplane_save_data` writes them in this order and
`airplane_load_data` reads them back in the same order.

```c
typedef struct C_airplane {              /* extends C_actor */
    /* ...    */ C_actor    base;
    /* +0x10  */ int        kind;             // [H] = 0x16
    /* +0x70  */ float      speed_cruise;     // [H] 5.0f
    /* +0x74  */ float      speed_max;        // [M] 25.0f
    /* +0x78  */ float      unk_78;           // [U] 0.0f
    /* +0x7c  */ float      unk_7c;           // [U] 5.0f  (ctor only, not duplicated)
    /* +0x80  */ float      unk_80;           // [U] 9.0f  — copied LAST by DuplicateFrom
    /* +0x84  */ float      scale;            // [M] 1.0f
    /* +0x88  */ float      ratio_a;          // [U] 0.8f
    /* +0x8c  */ float      pitch_max;        // [H] 1.5f — engine-sound pitch, upper end
    /* +0x90  */ int        sound_id_engine;  // [H] 0x82 \_ the 12-byte block SaveData
    /* +0x94  */ int        sound_id_b;       // [H] 0x83  | writes in one dtaWrite
    /* +0x98  */ int        sound_id_c;       // [H] 0x84 _/  SOUND ids, not anim ids
    /* +0x9c  */ float      rate_a;           // [U] 0.01f
    /* +0xa0  */ float      prop_speed_max;   // [H] 0.1f  — propeller angular-speed cap
    /* +0xa4  */ float      prop_speed_rate;  // [H] 0.015f — its ramp rate per ms
    /* +0xa8  */ float      damage_max;       // [H] 400.0f — the HUD gauge's denominator
    /* +0xac  */ int        particle_kind_lo; // [H] 9 — effect spawned below half damage
    /* +0xb0  */ int        particle_kind_hi; // [H] 6 — effect spawned at zero damage
    /* +0xb4  */ int        flight_state;     // [H] 0 idle · 1 route-follow · 2 takeoff run
    /* +0xb8  */ float      route_progress;   // [H] 0..1 along the segment; also the speed
                                              //     scalar TickSounds feeds to the emitters
    /* +0xbc  */ float      segment_t;        // [H] zeroed by airplane_pick_next_web_node
    /* +0x110 */ C_web_node *node_cur;        // [H] saved/loaded as an INDEX (/0x1e)
    /* +0x114 */ C_web_node *node_next;       // [H] same
    /* +0x118 */ vector     prop_meshes_a;    // [H] slow-blade meshes; visibility = +0x169
    /* +0x128 */ vector     prop_meshes_b;    // [H] blur-disc meshes;  visibility = +0x168
    /* +0x138 */ float      prop_angle;       // [H] shared blade angle, wraps at 2*pi
    /* +0x13c */ float      prop_speed;       // [H] ramps toward +0xa0 at rate +0xa4
    /* +0x140 */ S_vector   prop_axis;        // [H] rotation axis passed to each mesh
    /* +0x14c */ void      *sound[3];         // [H] I3D sound emitters (create_sounds)
    /* +0x158 */ int        sound_prev;       // [H] outgoing slot, -1 = none
    /* +0x15c */ int        sound_cur;        // [H] incoming slot, -1 = none
    /* +0x160 */ float      vol_prev;         // [H] fades out over 6000 ms
    /* +0x164 */ float      vol_cur;          // [H] fades in over 6000 ms, clamped 1.0
    /* +0x168 */ char       prop_blur_visible;// [H] drives prop_meshes_b
    /* +0x169 */ char       prop_blades_visible;//[H] drives prop_meshes_a
    /* +0x16a */ char       damage_hud_shown; // [H] airplane_hud_damage_toggle's flag
    /* +0x16c */ void      *dyn_coll_a;       // [H] tDynamicCollObject*
    /* +0x170 */ void      *dyn_coll_b;       // [H] tDynamicCollObject*
    /* +0x174 */ void      *dyn_coll[4];      // [H] the 4-slot array
    /* +0x1c4 */ int        node_id;          // [H] = 0x29a (the project-wide "none")
    /* +0x1c8 */ float      damage_a;         // [H] numerator of the HUD gauge fill
    /* +0x1cc */ float      damage_b;         // [H] the other numerator
    /* +0x1d0 */ int        particle_a;       // [H] -1 = none
    /* +0x1d4 */ int        particle_b;       // [H] -1 = none
    /* +0x1d8 */ int        particle_kind_a;  // [H] which kind is live on engine A
    /* +0x1dc */ int        particle_kind_b;  // [H] ditto engine B — the swap guard
} C_airplane;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x10 | kind | H | `airplane_ctor` (= 0x16) | `folded_get_dword_0x10` (its `C_actor::GetKind` role) | the engine's stand-in for RTTI |
| +0x70..+0xb0 | the tunable band | H/M/U | `airplane_ctor`, `airplane_duplicate_from`, `airplane_load_data` | `airplane_save_data`, the flight code | 16 values, identical set on both builds |
| +0x90/+0x94/+0x98 | **sound ids**, not anim ids | H | `airplane_ctor` (0x82/0x83/0x84) | `airplane_create_sounds` | passed straight to `sound_table_construct_sound` |
| +0xb4 | flight_state | H | `airplane_start_ground_run` (=1), `airplane_begin_takeoff` (=2), `airplane_ai_tick` (→0 at route end) | `airplane_ai_tick`, `airplane_save_game_save` | 0 idle · 1 route-follow · 2 takeoff run |
| +0x14c[3] / +0x158 / +0x15c / +0x160 / +0x164 | the sound band | H | `airplane_create_sounds`, `airplane_play_sound` | `airplane_tick_sounds`, `airplane_destroy_sounds` | prev/current slot + their crossfade volumes; the fade is 6000 ms on **both** builds |
| +0x168 / +0x169 | propeller visibility | H | `airplane_spin_propellers` | `airplane_apply_propeller_visibility`, `airplane_save_game_save` | exactly one of the two mesh sets is visible — the slow-blade ↔ blur-disc swap |
| +0xa8 | damage_max | H | `airplane_ctor` (= 400.0f) | `airplane_hud_damage_toggle` | the gauge denominator |
| +0x118 / +0x128 | two vectors | M | `airplane_ctor` | - | freed in REVERSE order by `airplane_dtor` |
| +0x16a | damage_hud_shown | H | `airplane_hud_damage_toggle` | - | - |
| +0x16c / +0x170 / +0x174[4] | 6 dynamic collision proxies | H | `airplane_game_init` (`collision_dyn_insert` ×3 + the array) | `airplane_game_done` (`collision_dyn_erase`) | fuselage + wings/engines |
| +0x1c4 | node_id | H | `airplane_ctor` (= 0x29a) | `airplane_start_ground_run` (`web_path_find_node_id`) | `0x29a` = the project-wide "none/unset" sentinel |
| +0x1c8 / +0x1cc | damage_a / damage_b | H | the flight/hit code | `airplane_hud_damage_toggle` | fill = `(a + b) / (max + max)` |
| +0x1d0 / +0x1d4 | two particle handles | H | `airplane_game_init` | `airplane_game_done` | −1 = none |

---

## 2. Method coverage — 29/32 PS2 methods mapped, 32/32 accounted

PC-first, by PC VA. The TU is bounded at **both** ends by a CRT static-init/atexit pair —
`0x0044ef20` before `airplane_ctor` and `0x00453a20` after the last method — so it runs
`[0x0044ef50, 0x00453a20)` and contains **33** function starts, counting unnamed and named
definitions alike. The region is **forced 1:1**:

```
33 PC = 32 PS2 methods − 3 absent (Init · Death · NetDirect) + 4 PC-only
        (the MSVC scalar deleting dtor · 2 foreign helpers · 1 PC-side sibling)
```

Two rows are pinned by the vtable `0x0063b7c0` alone: slot 13 =
`airplane_ai_tick` and slot 14 = `airplane_update` (the `C_actor`-family AI/Update slots),
with slot 16 = `airplane_scalar_deleting_dtor`, the slot-16 sdd fingerprint.

| PC VA | PC name | PS2 twin (mangled) | PS2 size | Status | Role |
|---|---|---|---|---|---|
| 0x0044ef50 | `airplane_ctor` | `__ct__10C_airplaneFv` | 368 | CONFIRMED | **PC 368 B = PS2 368 B**; every constant matches in order |
| 0x0044f0c0 | `airplane_scalar_deleting_dtor` | - | - | STRUCTURAL | MSVC thunk; GCC folds the free flag into `__dt__` |
| 0x0044f0e0 | `airplane_dtor` | `__dt__10C_airplaneFv` | 152 | CONFIRMED | frees the 2nd vector then the 1st, then `actor_dtor` |
| 0x0044f140 | `airplane_save_data` | `SaveData__10C_airplaneFP7C_chunk` | 688 | CONFIRMED | guarded `dtaWrite` chain over the tunable band |
| 0x0044f380 | `airplane_load_data` | `LoadData__10C_airplaneFP7C_chunk` | 1904 | CONFIRMED | the mirror; delegates most fields to `chunk_read` |
| 0x0044f600 | `airplane_duplicate_from` | `DuplicateFrom__10C_airplaneFP7C_actor` | 164 | CONFIRMED | 16 fields, same 16 values both builds |
| 0x0044f6c0 | `airplane_game_init` | `GameInit__10C_airplaneFv` | 2756 | CONFIRMED | 6 collision proxies, sounds, particles, propeller setup |
| 0x004503c0 | `airplane_game_done` | `GameDone__10C_airplaneFv` | 268 | CONFIRMED | statement-aligned teardown |
| 0x004504a0 | `airplane_pick_next_web_node` | `NextNode__10C_airplaneFv` | 1316 | CONFIRMED | walks the `C_web_node` conn array; called at PS2 `AI`'s `NextNode` statement |
| 0x00450990 | `airplane_fly_along_route` | `AI_fly__10C_airplaneFUi` | 3396 | CONFIRMED | lerps pos between `+0x110`/`+0x114` by `+0xbc`, normalises the heading |
| 0x00451540 | `airplane_spin_propellers` | `Vrtule__10C_airplaneFUi` | 548 | CONFIRMED | 2π wrap, ramp toward `+0xa0` at `+0xa4`; *vrtule* = propeller |
| 0x004516e0 | `airplane_ai_tick` | `AI__10C_airplaneFUi` | 1064 | CONFIRMED | **vtable slot 13**; 0.5 arrival radius, Slerp, the state-2 takeoff run |
| 0x00451990 | `airplane_update` | `Update__10C_airplaneFUi` | 2296 | CONFIRMED | **vtable slot 14**; shared masks `&0xfffffedf\|0x40000000` then `&0xfffffecb\|0x40000008` |
| 0x00452000 | `quat_multiply` | - | - | STRUCTURAL | 🔸 **foreign** — an S_quat inline; 3 callsites in 3 unrelated TUs, all stack receivers |
| 0x004520e0 | `airplane_start_ground_run` | `Start__10C_airplaneFUif` | 544 | CONFIRMED | `SetActState(0)` → `FindNodeID` → conn walk, fully aligned |
| 0x004522c0 | `airplane_begin_takeoff` | `Start2__10C_airplaneFv` | 48 | CONFIRMED | byte-identical `if (state != 2) { state = 2; PlaySound(1); }` |
| 0x004522e0 | `airplane_apply_propeller_visibility` | `InitVrtule__10C_airplaneFv` | 164 | CONFIRMED | applies `+0x169`/`+0x168` to the two mesh vectors — applies state, initialises nothing |
| 0x00452350 | `airplane_hit` | `Hit__10C_airplaneF10E_hit_type…` | 880 | CONFIRMED | picks the nearer engine by squared distance, drains its pool |
| 0x004525d0 | `airplane_update_engine_particles` | `UpdateParticles__10C_airplaneFv` | 616 | CONFIRMED | same `damage < max*0.5` threshold, same kind-cache guard |
| 0x00452810 | `airplane_get_engines_pos` | `GetEnginesPos__10C_airplaneFR8S_vector…` | 1484 | CONFIRMED | matrix transform + the triple ±1.0 normalisation cascade |
| 0x00452e70 | `airplane_get_engine_local_offsets` | - | - | STRUCTURAL | 🔸 **PC-only**; three fixed local-space offsets, **no `this`** ¹ |
| 0x00453090 | `airplane_create_sounds` | `InitSounds__10C_airplaneFv` | 292 | CONFIRMED | same 4 seeds, same create→construct→mute→loop→attach→register loop |
| 0x00453150 | `airplane_destroy_sounds` | `ReleaseSounds__10C_airplaneFv` | 148 | CONFIRMED | statement-identical 3-slot unregister+destroy |
| 0x00453190 | `airplane_is_destroyed` | `IsDestroyed__10C_airplaneFv` | 68 | CONFIRMED | 1 iff both engine damages are 0 |
| 0x004531c0 | `airplane_save_game_get_size` | `SaveGameGetSize__10C_airplaneFv` | 32 | CONFIRMED | **identical**: base `+ 0x7c` |
| 0x004531d0 | `airplane_save_game_save` | `SaveGameSave__10C_airplaneFPPv` | 332 | CONFIRMED | version byte 2; web nodes stored as indices |
| 0x00453300 | `airplane_save_game_load` | `SaveGameLoad__10C_airplaneFPPv` | 416 | CONFIRMED | the 4-call tail matches exactly — it re-confirms three other rows |
| 0x00453450 | `airplane_play_sound` | `PlaySound__10C_airplaneF…E_AIRPLANE_SOUND` | 156 | CONFIRMED | plays one of the three sound ids at `+0x90..+0x98` |
| 0x004534c0 | `airplane_tick_sounds` | `TickSounds__10C_airplaneFUi` | 664 | CONFIRMED | same 6000 ms crossfade constant on both builds |
| 0x004537c0 | `airplane_on_act_state_changed` | `ChangeState__10C_airplaneFv` | 276 | CONFIRMED | gated on act_state `+0x1c == 2`; same teardown as GameDone minus the base call |
| 0x004538b0 | `airplane_hud_damage_toggle` | `ShowDamageIndicator__10C_airplaneFb` | 148 | CONFIRMED | a class method that shows/hides the HUD damage gauge |
| 0x00453900 | `i3d_frame_build_scaled_rot_matrix` | - | - | STRUCTURAL | 🔸 **foreign** — a shared I3D helper; 36 callsites over 15+ unrelated functions |
| 0x00453a10 | `airplane_get_material_id` | `GetMaterialID__10C_airplaneFP9I3D_frame` | 8 | CONFIRMED | **byte-identical**: `return 0x13;` (material 19) |
| - | *(absent)* | `Init` 44 · `Death` 8 · `NetDirect` 8 | - | STRUCTURAL | see §3a — all three accounted for, none of them missing |

¹ Built from the constants `-1.0` / `2.9` / `4.0`, no transform, no field or global read. The
prologue is `sub esp,0x48`; every `lea ecx,[esp+N]` is a stack `S_vector` for `vec3_set_xyz`, and
ECX is written before it is ever read, so the function is not `__thiscall` — the `mov ecx, esi`
at its one callsite in `airplane_update_engine_particles` is the caller's own state.
`UpdateParticles` passes the offsets to `CreateParticle` *with* the frame, so they are
frame-local; PS2 calls the world-space `GetEnginesPos` there instead.

---

## 3. PC vs PS2

**The class is a straight port.** The tunable band holds the same 16 values on both builds,
and `SaveData`/`LoadData` walk it in the same order. The field rebase is **not a constant
delta** (PS2 `+0x33c` ↔ PC `+0x70`, but PS2 `+0x3f0` ↔ PC `+0x1c4`) because the C_actor base
is larger on PS2 *and* the two vector members have different sizes — map by RANK, never by
subtracting.

**`airplane_load_data`'s size ratio is 0.33 and that is not a rank error.** It is factoring in
the *PS2-inlines* direction: GCC open-codes every guarded `dtaRead`, MSVC
factors them into `chunk_read(field, size)`. A ratio far outside 1.0–1.6 means the
factoring differs — here it demonstrably does, and the field *sequence* still matches
one-for-one.

### 3a. The three PS2 methods with no PC function — all accounted, none missing

Each absence is proven, not assumed:

| PS2 method | Size | Why there is no PC function |
|---|---|---|
| `Init__10C_airplaneFP9I3D_frame` | 44 | **Body read.** It is a bare forwarder — `if (!frame) return false; return C_actor::Init(frame);` — with no distinctive constant for MSVC to keep out of line. Inlined on the PC |
| `NetDirect__10C_airplaneFUi` | 8 | **Byte-proven folded.** Airplane vtable slot 15 holds `stub_noop_args4_00461180`, whose whole body is `c2 04 00` (`ret 4` = exactly one 4-byte argument), and which sits in **30** vtables |
| `Death__10C_airplaneFv` | 8 | Same ICF fold — an 8-byte void stub. Its slot was not isolated, so this row is the weakest of the three |

💡 The general shape: **PS2's three 8-byte stubs collapse to one shared PC function each**,
and `GetMaterialID` survives as its own PC function *only* because it returns a distinct
constant (`0x13`) rather than nothing.

### 3b. Corollary twins this class produced

| PC | PS2 | how |
|---|---|---|
| `airplane_hud_damage_toggle` | `ShowDamageIndicator__10C_airplaneFb` | 2nd statement of the GameDone pair on both builds |
| `collision_dyn_insert` | `DynInsert__11g_collisionFP18tDynamicCollObject` | ×3 ↔ ×3 in the GameInit pair |
| `particle_manager_destroy_particle` | `DestroyParticle__17C_ParticleManagerFi` ¹ | the particle teardown |
| `particle_manager_find_by_guid` | `FindGuid__17C_ParticleManagerFiRi` | - |
| `chunk_read` | `Read__7C_chunkFPvi` ² | ×11 in `airplane_load_data` |
| `airplane_play_sound` | `PlaySound__10C_airplaneF…` | witnessed twice: the `Start2` pair and the `SaveGameLoad` tail |
| `airplane_apply_propeller_visibility` | `InitVrtule__10C_airplaneFv` | its own diff, then re-confirmed by the `SaveGameLoad` tail |
| `airplane_update_engine_particles` | `UpdateParticles__10C_airplaneFv` | same |

¹ The record stride differs (PC 8 vs PS2 0x10), but that is record content: the key field
`+0x04` and the sorted-by-guid access are identical (`structs/recorder.md`).
² A bounds-checked `dtaRead` of one chunk field; a shared helper with **212 callsites across 20+
unrelated functions** (`mission_open_bin` ×46, `airplane_load_data` ×11, `human_load_data`). It
leads `C_bottle`'s TU (`structs/bottle.md` §2).

### 3c. Two cross-build constants worth reusing elsewhere

- **The `C_web_node` record size shows up in the savegame math.** `airplane_save_game_save`
  stores node pointers as `(node − web_base) / 0x1e` on the PC and `>> 5` (0x20) on PS2 — an
  independent re-proof of the record-size difference documented in `structs/web_node.md`,
  arrived at from a completely different direction.
- **The 6000 ms sound crossfade is identical on both builds** — PS2 divides the frame delta
  by `6000.0`, MSVC multiplies by `0.00016666666`. Same tuning, different codegen.

---

## 4. Open items

| Item | Why it is open |
|---|---|
| `+0x78` / `+0x7c` / `+0x80` / `+0x9c` meaning | [U]. `airplane_ai_tick` reads `+0x80` as the route-progress scale (and `+0x70` as its ramp rate), and `airplane_spin_propellers` compares `+0x7c` against `route_progress`; `+0x78` and `+0x9c` have no traced consumer |
| Where instances come from | placed in the scene file; no spawn path traced |

**Related:** `structs/actor.md` (base class) · `structs/hud.md` (the `0x40000` damage-gauge
bit) · `basic/artifacts/hud_elements.md`.
