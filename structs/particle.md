# The particle system — `[0x005a1b00, 0x005a5fb0)` (PC prefixes `particle_*`, `particle_effect_*`, `particle_manager_*`)

**Created** 2026-08-30 10:03 +0200 · **Modified** 2026-10-11 19:32 +0200

Three consecutive translation units implement every particle effect in the game — engine
smoke, exhaust, sparks, the damage plume, the level-placed ambient effects. They are
driven by two shipped data files and hang off one pointer in `C_game`.

---

## 1. The three translation units, and the object that is not what its PS2 name suggests

`.CRT$XCU` fixes all three boundaries, each confirmed by its own
`crt_static_init_guard`/`crt_atexit_register` pair:

| `.CRT$XCU` slot | range | unit | PC prefix |
|---|---|---|---|
| `.data:0x006461d4` | `[0x005a1b00, 0x005a3150)` | one particle | `particle_*` |
| `.data:0x006461d8` | `[0x005a3150, 0x005a38f0)` | one spawned **effect** | `particle_effect_*` |
| `.data:0x006461dc` | `[0x005a38f0, 0x005a5fb0)` | the manager + its four pools | `particle_manager_*` |

**The middle object is an EFFECT, not an emitter.** One PC object owns a *vector* of
`0xc0`-byte per-emitter runtime records — one per emitter definition in the effect type. So one
PC object is one spawned effect: one guid, one parent frame, N emitters, each with its own
live-particle list. The game's own vocabulary agrees: the two data files are `tables\effects.tbl`
and `missions\%s\effects.bin`.

## 2. Where the manager lives — `C_mission + 0x6c`

The manager is a **pointer** member of `C_mission`, not an embedded object:

* `mission_init` allocates it at `this+0x6c` (`operator_new(0x38)` → `particle_manager_ctor`) and
  `mission_uninit` frees it there.
* Every `C_game` caller reaches it through the mission pointer: the callsites disassemble to
  `mov eax,[0x65115c]` (`g_act_mission`) then `mov ecx,[eax+0x6c]`. Byte-measured on four call
  sites:

| caller | callee |
|---|---|
| `game_init` | `particle_manager_mission_start` |
| `game_init` | `particle_manager_load_static_particles` |
| `game_done` | `particle_manager_mission_done` |
| `game_tick` | `particle_manager_tick` |

Equal offsets on unrelated classes are common, so a `[reg+0x6c]` census proves nothing until the
base register is known — here it holds `g_act_mission`, not the `C_game` `this`.

## 3. Object layouts (PC)

### 3a. `particle_manager` — the singleton at `C_mission + 0x6c`

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` .. `+0x0c` | `vector<{effect*, guid}>`, **stride 8** — the LIVE effects, kept sorted by guid | `particle_manager_find_by_guid` binary-searches it; `particle_manager_tick` walks it |
| `+0x10` .. `+0x1c` | `vector<effect_type*>` — the `0x3c`-byte types loaded from `tables\effects.tbl` | filled by `particle_manager_load_data`, indexed by `particle_manager_create_particle` |
| `+0x20` .. `+0x2c` | `vector<effect*>` — the LEVEL-PLACED effects from `effects.bin` | filled by `particle_manager_create_static_particle`, ticked first each frame |
| `+0x30` | **the last effect guid issued**: `particle_manager_create_particle_from_type` increments it, stores the new value beside the effect in the live vector and returns it as the effect's guid; `-1` until the first effect | set to `-1` by the constructor, `mission_start` and `mission_done`; `particle_manager_tick` skips the live list while it is `-1`; `particle_manager_find_by_guid` binary-searches the live vector on these values |
| `+0x34` | started flag | gates `mission_start` and `mission_done` |

### 3b. `particle_effect` — 0x4c bytes, allocated from `g_particle_effect_pool`

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | parent `I3D_frame *` | written by `particle_effect_init`; cleared by `particle_effect_destroy` |
| `+0x04` .. `+0x10` | `vector<emitter_record*>` — one per emitter definition in the type | drained by `particle_effect_destroy`, freed by `particle_effect_dtor` |
| `+0x14` | **engine id** (short), taken from the engine driver's counter at `I3DGetDriver()+0x58`; not the manager's guid (§3a `+0x30`) | `particle_effect_init` post-increments the counter; `particle_effect_spawn_particle` hands it to `particle_create`, which stamps it into the particle frame's `+0x21c` |
| `+0x18` .. `+0x23` | local position offset | `particle_effect_init` |
| `+0x24` .. `+0x2f` | local direction | `particle_effect_init` |
| `+0x30` .. `+0x47` | two further vectors the creators pass through | `particle_effect_init` |
| `+0x48` | **spawn-enable** byte — 1 = emitting | `particle_effect_tick`'s spawn gate; the only field `particle_manager_set_particle_emitting` writes |
| `+0x49` | **pause** byte — non-zero makes the tick a no-op that still reports "alive" | `particle_effect_tick`'s first statement |
| `+0x4a` / `+0x4b` | rotate-offset-by-parent / rotate-direction-by-parent flags | read by `particle_effect_spawn_particle` |

### 3c. `emitter_record` — 0xc0 bytes, allocated from `g_particle_type_pool`

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x08` .. `+0x14` | `vector<phase_record*>` — the phase chain | `particle_begin_phase` indexes it; `particle_effect_tick` compares the index against its size to decide retirement |
| `+0x20` .. `+0x2c` | `vector<particle*>` — the live particles of THIS emitter | `particle_effect_spawn_particle` appends, `particle_effect_tick` swap-erases |
| `+0x88` | burst count — particles emitted at once on start | the `SpawnOnCreate` arm of `particle_effect_tick` |
| `+0x8c` / `+0x90` | current / initial emitter lifetime | aged by `particle_effect_tick`, restored on loop |
| `+0x94` | lifetime decay rate | `particle_effect_tick` |
| `+0x98` | derived spawn rate `1000 / lifetime` | recomputed whenever the lifetime changes |
| `+0xa0` / `+0xa4` | spawn-clock threshold / accumulator | `particle_effect_tick` |
| `+0xa8` / `+0xac` | initial / current start delay | counted down before the first spawn |
| `+0xb0` | remaining loop count; `>1` restarts the emitter | `particle_effect_tick` |
| `+0xb4` | fractional spawn accumulator | `particle_effect_tick`'s rate arm |
| `+0xb8` | loop-forever flag | `particle_effect_tick` |
| `+0xbc` | flag word copied from the definition | bit 1 = rotate to velocity, bit 2 = use parent frame, bit 4 = per-axis spin |

### 3d. `particle` — 0x64 bytes, allocated from `g_particle_pool`

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | its own `I3D_frame *` | created in `particle_create`, released by `particle_dtor` |
| `+0x04` | the owning `emitter_record *` | `particle_begin_phase` reaches the phase vector through it |
| `+0x08` | current phase index | `particle_begin_phase` writes it, `particle_effect_tick` reads it |
| `+0x0c` .. `+0x17` | velocity vector | integrated by `particle_tick` |
| `+0x18` .. `+0x37` | the slerp start/target `S_quat` pair | built by `particle_begin_phase` |
| `+0x38` | "is turning" flag — chooses slerp vs straight re-aim | `particle_begin_phase`'s 0.95 dot-product threshold |
| `+0x3c` | remaining phase lifetime (ms) | `particle_tick` decrements; `<1` advances the phase |
| `+0x44` | flag word copied from the emitter record | `particle_create` |
| `+0x4c` | total phase duration | used to normalise the slerp parameter |
| `+0x50` .. `+0x58` | three per-channel scale/colour rates | `(target - current) / duration`, applied by `particle_tick` |
| `+0x5c` | alpha rate | `particle_tick` |
| `+0x60` | spin rate | `particle_tick` |

### 3e. `phase_record` — 0x64 bytes, POD, allocated from `g_particle_phase_pool`

The only one of the four pooled types whose elements are **POD**:
`particle_manager_mission_done` frees them with no destructor call, and that is the
measurement that separates this pool from the same-sized `g_particle_pool`.

| Offset | Meaning |
|---|---|
| `+0x08` | flag byte — bit 3 world-space direction, bit 4 per-axis spin, bit 5 no move |
| `+0x0c` .. `+0x17` | primary direction |
| `+0x18` .. `+0x23` | secondary direction (the cone axis) |
| `+0x24` / `+0x28` / `+0x2c` | per-axis rotation speeds, read by `particle_tick` |
| `+0x38` .. `+0x43` | three target channel values |
| `+0x44` .. `+0x4f` | their three random spreads |
| `+0x50` / `+0x54` | target alpha and its spread |
| `+0x58` / `+0x5c` | target spin and its spread |
| `+0x60` | defaults to `0x411ccccd` = **9.8f** |

The size closes exactly: the initializer in `particle_manager_create_static_particle`
writes up to index `[0x18]` = byte `0x60`, and `0x60 + 4 = 0x64` = the `operator_new(100)`.

## 4. The four free lists

| global | VA | element | element size | destroyed by |
|---|---|---|---|---|
| `g_particle_effect_pool` | `0x006efbb0` | effect | `0x4c` | `particle_effect_dtor` |
| `g_particle_phase_pool` | `0x006efbc0` | phase record | `0x64` | *(nothing — POD)* |
| `g_particle_type_pool` | `0x006efbd0` | emitter record | `0xc0` | `particle_type_dtor` (`0x005a6c50`) |
| `g_particle_pool` | `0x006efbe0` | particle | `0x64` | `particle_dtor` |

Each is a `{allocator byte, begin, end, cap}` block with its own four-function CRT quartet
(`_static_init` / `_static_ctor` / `_register_atexit_dtor` / `_static_dtor`) and its own
`.CRT$XCU` slot at `0x006461e0`, `0x006461e4`, `0x006461e8`, `0x006461ec`. They sit in BSS
immediately after `g_net_session` (`0x006efbac`) — corroboration of the link order, since
BSS is emitted in link order and the particle TUs follow the net-session TU.

**`particle_manager_mission_done` is what identifies the pools.** Nothing else in the
binary says which destructor each pool's elements get, and without that the effect pool and
the particle pool are two indistinguishable vectors of same-shaped blocks.

## 5. The two shipped data files

### 5a. `tables\effects.tbl` — the effect-type table

Read by `particle_manager_load_data` through a stack `C_chunk`. Outer chunk id **100** (`0x64`)
per effect type (`operator_new(0x3c)`); the shipped file holds 93. The id tree, with the shipped
counts; in each of the three readers, every id not listed takes the level's `0x6a` arm:

| Id | Inside | Shipped | Payload and destination |
|---|---|---:|---|
| `0x65` | effect type | 169 | one emitter definition, `operator_new(0x9c)`, appended to the type's emitter list ¹ |
| `0x67` | effect type | 93 | a string into `type+0x08` |
| `0x68` | effect type | 93 | a string into `type+0x34`, the model name the scene cache is keyed on |
| `0x6a` | effect type | 93 | two 12-byte blocks from `type+0x1c` |
| `0x6e` | effect type | 93 | two flag bytes, `type+0x38` / `+0x39` |
| `0x66` | emitter | 261 | one phase definition, `operator_new(0x68)`, appended to the emitter's phase list |
| `0x67` | emitter | 155 | a name string |
| `0x69` | emitter | 169 | a string into `emitter+0x1c` |
| `0x6a` | emitter | 169 | two vec3 then fourteen dwords, `emitter+0x20`..`+0x98` |
| `0x6b` | emitter | 169 | two vec3, `emitter+0x38` / `+0x44` |
| `0x6c` | emitter | 169 | two vec3, `emitter+0x58` / `+0x64` |
| `0x6d` | emitter | 169 | a dword, `emitter+0x7c` |
| `0x67` | phase | 11 | a string into `phase+0x08` |
| `0x6a` | phase | 261 | three vec3 then nine dwords, `phase+0x0c`..`+0x60` |
| `0x6b` | phase | 261 | a dword, `phase+0x64` |
| `0x6c` | phase | 261 | two vec3, `phase+0x38` / `+0x44` |

¹ The `0x9c` definition is built by `particle_data_type_ctor` (`0x005a63d0`) and filled by
`particle_data_type_load_data` (`0x005a6490`); both read through the `C_chunk` helpers
`chunk_enter_next`, `chunk_bytes_remaining`, `chunk_read_string_and_leave` and `chunk_leave`.

**The definition vtables** (`.rdata 0x0063e33c..0x0063e348`) are assigned by the base-then-derived
install sequence in the inlined `0x3c` constructor inside `particle_manager_load_data`: it writes
`g_particle_data_vt` + kind 0, then `g_particle_data_emitter_vt` + kind 1.

| vtable | VA | object | kind |
|---|---|---|---|
| `g_particle_data_emitter_vt` | `0x0063e33c` | the `0x3c` **effect** definition | 1 |
| `g_particle_data_vt` | `0x0063e340` | their base | 0 |
| `g_particle_data_type_vt` | `0x0063e344` | the `0x9c` **emitter** definition | 2 |
| `g_particle_data_phase_vt` | `0x0063e348` | the `0x68` **phase** definition | 3 |

`particle_manager_ctor` itself calls `particle_manager_load_data`, so the table is loaded when
`mission_init` creates the manager (§2); `particle_manager_mission_start` loads it only while the
type list is empty.

### 5b. `missions\%s\effects.bin` — the level-placed effects

Read by `particle_manager_load_static_particles`. Outer chunk id must be **100** (the PC
accepts no other version, and all 79 shipped PC files are version 100). Each inner chunk id **1000** is one placement:

| field | size | meaning |
|---|---|---|
| matrix | `0x40` | the placement transform, passed straight to `particle_manager_create_static_particle` |
| type index | `4` | index into the manager's `+0x10` effect-type list |
| frame name | *(sub-chunk)* | resolved in the mission scene by name |

Its three failures are reported through `dbgPrintf`: *"Effects.bin - Error: could not
open"*, *"error in chunk"*, *"different version"*.

## 6. The level-placed effect registry in `C_game`

`C_game + 0x2c8` is a **stride-`0x18`** vector of `{guid, covered_flag, vec3 pos,
object*}`, walked every frame by `game_tick`. Per entry:

1. `particle_manager_is_particle_alive(guid)` — a dead effect is unlinked and swap-erased.
2. Beyond **150 units** of the player (`22500.0` = 150²) the effect is destroyed outright.
3. A vertical ray is cast **upward** from the effect's position. If it hits anything whose
   owner is not a `C_car`, `particle_manager_set_particle_emitting(guid, 0)` stops emission;
   when the sky is clear again it is switched back on with `1`.

⇒ **In game: a street plume stops making new particles under a bridge or an awning, and a
car parked over it does not count.** The particles already alive keep drawing.

## 7. PC vs PS2

### 7a. The class map

| PC | PS2 | note |
|---|---|---|
| `particle_*` | `C_Particle` | 5 PC functions ↔ 5 PS2 methods |
| `particle_effect_*` | `C_ParticleEmitter` | 5 PC functions ↔ 15 PS2 methods (see 7b) |
| `particle_manager_*` | `C_ParticleManager` | 22 PC functions ↔ 21 PS2 methods |
| the `0x3c` **effect** definition, kind 1 | `C_PData_Emitter` | vtable `g_particle_data_emitter_vt` (`0x0063e33c`) |
| the `0x9c` **emitter** definition, kind 2 | `C_PData_Type` | vtable `g_particle_data_type_vt` (`0x0063e344`) |
| the `0x68` **phase** definition, kind 3 | `C_PData_Phase` | vtable `g_particle_data_phase_vt` (`0x0063e348`) |
| their base, kind 0 | `C_PData` | vtable `g_particle_data_vt` (`0x0063e340`) |
| the `0xc0` runtime emitter record | `C_ParticleType` | pool `g_particle_type_pool`; PC prefix `particle_type_` |

**The kind byte is build-invariant** and settles the definition classes:
`particle_manager_load_data` writes kind **1** into the `0x3c` object and kind **2** into the
`0x9c`, and PS2 `__ct__12C_PData_TypeFv` writes kind **2** into a `0x9c` object. Corroborated three
ways: `ValidateParticleData__17C_ParticleManagerFP15C_PData_Emitter` validates the `0x3c` object;
`C_ParticleEmitter::Init` takes a `C_PData_Emitter*`, the effect definition; and
`C_Particle::Create` takes a `C_PData_Type*`, the per-emitter definition. On both builds the
containment is **`C_PData_Emitter` ⊃ `C_PData_Type` ⊃ `C_PData_Phase`**.

**The `0xc0` runtime record's PS2 type is `C_ParticleType`**, from the container instantiation
`insert__Q23std68__vector_imp<14C_ParticleType,…>` inside `Init__17C_ParticleEmitterF…`.
`C_ParticleEmitter::Init` appends one such record per `C_PData_Type` — exactly what
`particle_manager_create_particle_from_type` does with its `operator_new(0xc0)` loop.

**`C_ParticleType` has no PS2 methods**, so the PC `particle_type_*` functions have no twin of
their own: each is the per-record half of a `C_ParticleEmitter` method that the PS2 runs inline.
`particle_type_init` is that half of `Init`, called by the manager's two creators (§7b);
`particle_type_reset` of `Destroy`, called only by `particle_effect_destroy`; `particle_type_dtor`
of the destructor, called by `particle_effect_dtor` and `particle_manager_mission_done`. The
`particle_effect_*` functions carry the twins.

The only genuine structural difference is **where the particles live**: the PS2 keeps one shared
particle list on `C_ParticleEmitter+0xb4`; the PC gives each `0xc0` record its own list at
`record+0x20`. Both builds keep one runtime record per emitter definition.

The manager's guid counter is PC `+0x30` ↔ PS2 `+0xc34`: `CreateParticle__17C_ParticleManager`
increments it, stores it beside the emitter and returns it, as the PC creator does.

### 7b. What the PC folds that the PS2 keeps out of line

Each site is marked in the decomp by an inline banner.

| PS2 method | PC site |
|---|---|
| `ValidateParticleData` (224 B) | the five early-return gates of `particle_manager_create_particle_from_type` |
| `TickParticles` + `SpawnParticles` + `DeleteDeathParticles` + `Tick` (1364 B) | **4 → 1 fold** into `particle_effect_tick` (864 B) |
| `SpawnOnCreate` (240 B) | the burst arm of `particle_effect_tick` |
| `__ct__10C_ParticleFv`, `__ct__17C_ParticleEmitterFv`, `__ct__15C_PData_EmitterFv` | inlined at each `operator_new` site |
| `C_ParticleEmitter::Init`'s record construction | hoisted into the manager's two creators |

### 7c. The eight PS2 methods with no PC function

| PS2 method | why the PC has no function |
|---|---|
| `UpdateOptimized` | an empty body (`jr ra; nop`); `particle_effect_init` makes no call where the PS2 `Init` and `ReInit` call it |
| `SortParticlesAndCreateBBOX`, `PrepareVBuffer` | the PS2 `Tick` sorts the particles and fills one vertex buffer per emitter; each PC particle owns its own `I3D_frame` (§3d), a visual of type `'PRTB'` (`I3D_VISUAL_PART_BASE` in `LS3DF.dll`'s type table) that `particle_tick` positions, and `particle_effect_tick` calls nothing between the particle update and the spawn |
| `ReInit` | the PS2 `CreateParticle` takes an emitter from a per-type free list and re-initialises it; the PC takes the effect from the type-agnostic `g_particle_effect_pool` and always runs `particle_effect_init` and one `particle_type_init` per emitter |
| `PreloadTextures`, `ReleasePreloadedTextures` | a texture-handle list (manager `+0xc28..+0xc30`) filled by the game-init preloader `PreloadPicovinNaZasranyPSKram__Fv` and released by `Done__6C_gameFv`; the PC manager's `0x38` bytes are all mapped (§3a) and hold no such list |
| `LoadStaticParticles_100`, `LoadStaticParticles_201` | the PS2 `LoadStaticParticles` dispatches on the outer chunk id; the PC accepts one version and parses it inline (§7d) |

### 7d. Two shipped behavioural differences

* **Effect-file versions.** The PC reads outer chunk `0x64` only (all 79 shipped PC files) and
  prints *"different version"* otherwise. The PS2 dispatches chunk `0x65` to
  `LoadStaticParticles_100` (2 of the 104 shipped PS2 files) and `0xc9` to
  `LoadStaticParticles_201` (the other 102). Both read the PC's record (inner chunk 1000: matrix,
  type index, frame name); `_100` multiplies the matrix by the frame's world matrix before creating
  the effect, while `_201` passes it as read, as the PC does.
* **The pause bit.** PS2 `PauseParticle` toggles `flags & 4` — the PC's `+0x49` pause byte.
  The PC's same-slot function `particle_manager_set_particle_emitting` writes `+0x48`, the
  spawn-enable byte, instead. Same manager slot and the same `(guid, bool)` signature, different
  bit; the twin is STRUCTURAL for that reason.

### 7e. PS2 `C_mission` embeds the manager; the PC holds a pointer

The PS2 `C_mission` constructor calls `__ct__17C_ParticleManagerFv((int)this + 100)` and the
destructor calls `__dt__17C_ParticleManagerFv(this + 100, ...)`, so on that build the manager is a
**member of `C_mission` at `+0x64`**, spanning to about `+0xcb0`. The PC `C_mission` holds a pointer
at `+0x6c` (§2).

Consequence: every PS2 `C_mission` field above `+0x64` sits roughly **`0xc2c` higher** than its PC
counterpart, purely because of this member. Below `+0x64` the two maps agree. Do not carry a PS2
`C_mission` offset onto the PC by a constant (`structs/mission.md`).

The construction site also decides where `effects.tbl` is loaded. The PS2 constructor has no load
(the PS2 also builds a static `C_ParticleManager` in `__sinit_C_ParticleManager_cpp`), and
`Mission_Start` loads when the type list is empty. The PC constructor runs from `mission_init`
and loads there, through a real call; `particle_manager_mission_start` keeps the same empty-list
test. On both builds the table is therefore loaded once, before the first mission starts.

## 8. Cross-references

`structs/game.md` (the `+0x2c8` registry) · `structs/mission.md` (the `+0x6c` pointer) ·
`structs/chunk.md` §4 (the open-chunk stack both loaders push through) ·
`structs/def_tables.md` (`particles.def`, a *different* table — see its §5) ·
`basic/artifacts/dll.md` (`ls3df.dll`, which owns `I3D_emitor`).

## 9. Open items

* **In game:** does a street plume visibly stop emitting under an awning or a bridge, and
  resume when the camera moves it clear? (§6, step 3.)
* **In game:** the 150-unit destroy radius of §6 step 2 should make distant ambient effects
  pop out; worth confirming it is visible.
