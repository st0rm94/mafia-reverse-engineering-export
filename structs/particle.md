# The particle system — manager, effect, emitter record, particle, phase and the `C_PData` definitions

**Created** 2026-08-30 10:03 +0200 · **Modified** 2026-11-16 12:35 +0100

Four consecutive translation units implement every particle effect in the game: engine smoke, exhaust, sparks, the damage
plume and the level-placed ambient effects. Two shipped data files drive them, and one pointer in `C_mission` holds the manager.

| | |
|---|---|
| PC prefix | `particle_manager_`, `particle_effect_`, `particle_type_`, `particle_data_type_`, `particle_data_emitter_`, `particle_data_`, `particle_` |
| PC kind (`+0x10`) | - |
| PC vtable | only the definitions: `g_particle_data_vt` `0x0063e340`, `g_particle_data_emitter_vt` `0x0063e33c`, `g_particle_data_type_vt` `0x0063e344`, `g_particle_data_phase_vt` `0x0063e348`, 1 slot each |
| PC base class | `C_PData` for the three definition classes, otherwise - |
| PC instance size | manager `0x38`; effect `0x4c`; emitter record `0xc0`; particle `0x64`; phase record `0x64`; `C_PData` `0x0c`; `C_PData_Emitter` `0x3c`; `C_PData_Type` `0x9c`; `C_PData_Phase` `0x68` |
| PC constructor | `particle_manager_ctor`, `particle_data_ctor`, `particle_data_type_ctor`; the others are inlined at their `operator_new` sites |
| PC translation unit | `[0x005a1b00, 0x005a7070)`, four units guard to guard (§4) |
| PS2 class | `C_ParticleManager` (tag `17C_ParticleManager`), `C_ParticleEmitter` (`17C_ParticleEmitter`), `C_Particle` (`10C_Particle`), `C_PData` (`7C_PData`), `C_PData_Emitter` (`15C_PData_Emitter`), `C_PData_Type` (`12C_PData_Type`), `C_PData_Phase` (`13C_PData_Phase`); `C_ParticleType` has no methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The manager owns **effects**; an effect owns one **emitter record** per emitter definition of its type; a record owns its live
**particles** and one **phase record** per phase definition. The records and particles are copied from the loaded definitions at
spawn time and go back to the four free lists of §5 when they retire. Every vector below is MSVC's `std::vector`: an empty
allocator byte, then begin, end and capacity.

### 1a. `particle_manager`

```c
typedef struct particle_manager {     // 0x38 bytes, heap, pointed at by C_mission + 0x6c
    /* +0x00 */ uint8_t  live_alloc;      // [H] allocator byte of the live-effect vector
    /* +0x04 */ S_live_effect *live_begin; // [H] the live effects, stride 8, sorted by guid
    /* +0x08 */ S_live_effect *live_end;  // [H] one past the last live effect
    /* +0x0c */ S_live_effect *live_cap;  // [H] end of the live vector's storage
    /* +0x10 */ uint8_t  types_alloc;     // [H] allocator byte of the effect-type vector
    /* +0x14 */ C_PData_Emitter **types_begin; // [H] the types loaded from tables\effects.tbl
    /* +0x18 */ C_PData_Emitter **types_end; // [H] one past the last type
    /* +0x1c */ C_PData_Emitter **types_cap; // [H] end of the type vector's storage
    /* +0x20 */ uint8_t  statics_alloc;   // [H] allocator byte of the level-placed vector
    /* +0x24 */ particle_effect **statics_begin; // [H] the effects placed by effects.bin
    /* +0x28 */ particle_effect **statics_end; // [H] one past the last level-placed effect
    /* +0x2c */ particle_effect **statics_cap; // [H] end of the level-placed vector's storage
    /* +0x30 */ int32_t  last_guid;       // [H] the last effect guid issued; -1 before the first
    /* +0x34 */ uint8_t  started;         // [H] between mission_start and mission_done
} particle_manager;
```

`S_live_effect` is the pair `{particle_effect *effect, int guid}`. The vector stays sorted because the guid only grows and each
new pair goes to the end, which is what lets `particle_manager_find_by_guid` halve it.

### 1b. `particle_effect`

```c
typedef struct particle_effect {      // 0x4c bytes, from g_particle_effect_pool or operator_new
    /* +0x00 */ I3D_frame *parent;        // [H] the frame the effect hangs off, or NULL
    /* +0x04 */ uint8_t  records_alloc;   // [H] allocator byte of the record vector
    /* +0x08 */ S_emitter_record **records_begin; // [H] one record per emitter definition
    /* +0x0c */ S_emitter_record **records_end; // [H] one past the last record
    /* +0x10 */ S_emitter_record **records_cap; // [H] end of the record vector's storage
    /* +0x14 */ int16_t  engine_id;       // [H] taken from the driver's counter at +0x58
    /* +0x18 */ float    position[3];     // [H] the caller's position, x y z
    /* +0x24 */ float    direction[3];    // [H] the caller's direction, x y z
    /* +0x30 */ float    type_offset[3];  // [H] the type's first vec3, added to position
    /* +0x3c */ float    unread_3c[3];    // [U] copied from C_PData_Emitter +0x28, shipped (0, 0, 0); no reader (class census)
    /* +0x48 */ uint8_t  spawn_enabled;   // [H] 1 = emitting
    /* +0x49 */ uint8_t  paused;          // [H] non-zero makes the tick a no-op reporting alive; no non-zero store (.text scan)
    /* +0x4a */ uint8_t  rotate_offset;   // [H] turn the offset by the parent's world matrix
    /* +0x4b */ uint8_t  rotate_direction; // [H] turn the direction by the parent's world matrix
} particle_effect;
```

`position`, `direction`, `type_offset` and `unread_3c` are one vec3 each, x y z.

### 1c. `S_emitter_record`

The PS2 calls this runtime record `C_ParticleType`.

```c
typedef struct S_emitter_record {     // 0xc0 bytes, from g_particle_type_pool or operator_new
    /* +0x00 */ uint32_t unused_00;       // [U] no access; no reader (class census)
    /* +0x04 */ uint8_t  phases_alloc;    // [H] allocator byte of the phase vector
    /* +0x08 */ S_phase_record **phases_begin; // [H] the phase chain
    /* +0x0c */ S_phase_record **phases_end; // [H] one past the last phase
    /* +0x10 */ S_phase_record **phases_cap; // [H] end of the phase vector's storage
    /* +0x14 */ I3D_visual *visual;       // [H] the cached visual; refcount at its +0x04
    /* +0x18 */ I3D_frame *parent;        // [H] the frame the rotate flags of +0xbc use
    /* +0x1c */ uint8_t  particles_alloc; // [H] allocator byte of the particle vector
    /* +0x20 */ S_particle **particles_begin; // [H] the live particles of this emitter
    /* +0x24 */ S_particle **particles_end; // [H] one past the last particle
    /* +0x28 */ S_particle **particles_cap; // [H] end of the particle vector's storage
    /* +0x2c */ uint32_t frame_mask_bit;  // [H] the particle frame's +0x220 = 1 << this | 0x100
    /* +0x30 */ float    offset[3];       // [H] start offset from the effect
    /* +0x3c */ float    offset_spread[3]; // [H] random spread of the offset, per axis
    /* +0x48 */ float    direction[3];    // [H] start direction; the zero vector takes the effect's
    /* +0x54 */ float    direction_spread[3]; // [H] random spread of the direction, per axis
    /* +0x60 */ float    speed;           // [H] start speed
    /* +0x64 */ float    speed_spread;    // [H] random spread of the speed
    /* +0x68 */ float    scale[3];        // [H] start scale, per axis
    /* +0x74 */ float    scale_spread[3]; // [H] random spread of the scale, per axis
    /* +0x80 */ float    alpha;           // [H] start alpha
    /* +0x84 */ float    alpha_spread;    // [H] random spread of the alpha
    /* +0x88 */ int32_t  burst_count;     // [H] particles emitted at once on start
    /* +0x8c */ float    lifetime;        // [H] current emitter lifetime
    /* +0x90 */ float    lifetime_initial; // [H] restored on a loop
    /* +0x94 */ float    lifetime_decay;  // [H] lifetime lost per second after the threshold
    /* +0x98 */ float    spawn_rate;      // [H] 1000 / lifetime, recomputed as lifetime changes
    /* +0x9c */ float    decay_reciprocal; // [H] 1000 / lifetime_decay; no reader (class census)
    /* +0xa0 */ int32_t  clock_threshold; // [H] clock value after which the lifetime decays
    /* +0xa4 */ int32_t  clock;           // [H] ms since the last start; 0 fires the burst
    /* +0xa8 */ int32_t  delay_initial;   // [H] start delay, restored on a loop
    /* +0xac */ int32_t  delay;           // [H] current start delay, -1 from type_init
    /* +0xb0 */ int32_t  loops;           // [H] remaining loops; above 1 restarts the emitter
    /* +0xb4 */ float    spawn_fraction;  // [H] fractional spawn accumulator
    /* +0xb8 */ uint8_t  loop_forever;    // [H] set when the definition's loop count is -1
    /* +0xbc */ uint32_t flags;           // [H] copied from the definition, decoded in §2c
} S_emitter_record;
```

`offset`, `direction` and their spreads are x y z; `scale` and its spread are the three axes of the particle frame's scale at `+0x90`.

### 1d. `S_particle`

```c
typedef struct S_particle {           // 0x64 bytes, from g_particle_pool or operator_new
    /* +0x00 */ I3D_visual *frame;        // [H] its own engine frame, type 'PRTB'
    /* +0x04 */ S_emitter_record *record; // [H] the record that owns this particle
    /* +0x08 */ int32_t  phase_index;     // [H] index into the record's phase chain
    /* +0x0c */ float    velocity[3];     // [H] world velocity, x y z
    /* +0x18 */ S_quat   turn_from;       // [H] slerp start of a phase's direction change
    /* +0x28 */ S_quat   turn_to;         // [H] slerp target of a phase's direction change
    /* +0x38 */ uint8_t  turning;         // [H] slerp the velocity through this phase
    /* +0x3c */ int32_t  phase_ms;        // [H] remaining phase time; below 1 ends the phase
    /* +0x40 */ uint32_t unused_40;       // [U] no access; no reader (class census)
    /* +0x44 */ uint32_t flags;           // [H] copied from the record's +0xbc
    /* +0x48 */ uint32_t unused_48;       // [U] no access; no reader (class census)
    /* +0x4c */ int32_t  phase_duration;  // [H] the phase's whole time, normalises the slerp
    /* +0x50 */ float    scale_rate[3];   // [H] scale change per second, per axis
    /* +0x5c */ float    alpha_rate;      // [H] alpha change per second
    /* +0x60 */ float    spin_rate;       // [H] spin per second
} S_particle;
```

### 1e. `S_phase_record`

```c
typedef struct S_phase_record {       // 0x64 bytes, POD, from g_particle_phase_pool or operator_new
    /* +0x00 */ int32_t  duration;        // [H] phase time in ms, default 1000
    /* +0x04 */ int32_t  duration_spread; // [H] random spread of the phase time
    /* +0x08 */ uint32_t flags;           // [H] default 1, decoded in §2e
    /* +0x0c */ float    direction[3];    // [H] the direction the particle turns to
    /* +0x18 */ float    cone_axis[3];    // [H] non-zero: the direction is perturbed about it
    /* +0x24 */ float    axis_spin[3];    // [H] per-axis rotation speed, used with flag 0x10
    /* +0x30 */ float    unused_30[2];    // [U] no access; no reader (class census)
    /* +0x38 */ float    scale[3];        // [H] target scale, per axis
    /* +0x44 */ float    scale_spread[3]; // [H] random spread of the target scale
    /* +0x50 */ float    alpha;           // [H] target alpha
    /* +0x54 */ float    alpha_spread;    // [H] random spread of the target alpha
    /* +0x58 */ float    spin;            // [H] spin rate
    /* +0x5c */ float    spin_spread;     // [H] random spread of the spin rate
    /* +0x60 */ float    gravity;         // [H] default 9.8; velocity y falls by it, flag 0x02
} S_phase_record;
```

`unused_30` holds two floats because the phase record copies the definition's fields by name and skips the definition's `+0x30` / `+0x34`, which went to `duration`. `operator_new(100)` and the initialiser's last index `[0x18]` bound the size: `0x60 + 4 = 0x64`.

### 1f. `C_PData`

```c
typedef struct C_PData {              // 0x0c bytes
    /* +0x00 */ void    *vtable;          // [H] one slot, the scalar deleting destructor
    /* +0x04 */ int32_t  kind;            // [H] 0 base, 1 effect, 2 emitter, 3 phase
    /* +0x08 */ char    *name;            // [H] refcounted string, chunk 0x67
} C_PData;
```

### 1g. `C_PData_Emitter`

The **effect** definition, in spite of the PS2 class name (§8a).

```c
typedef struct C_PData_Emitter {      // 0x3c bytes, C_PData at +0x00
    /* +0x0c */ uint8_t  emitters_alloc;  // [H] allocator byte of the emitter vector
    /* +0x10 */ C_PData_Type **emitters_begin; // [H] the emitter definitions, chunk 0x65
    /* +0x14 */ C_PData_Type **emitters_end; // [H] one past the last definition
    /* +0x18 */ C_PData_Type **emitters_cap; // [H] end of the vector's storage
    /* +0x1c */ float    offset[3];       // [H] copied to the effect's type_offset, chunk 0x6a
    /* +0x28 */ float    unread_28[3];    // [U] chunk 0x6a, only copied into the effect's unread_3c; no reader (class census)
    /* +0x34 */ char    *model_name;      // [H] the name the scene cache is keyed on, chunk 0x68
    /* +0x38 */ uint8_t  rotate_direction; // [H] copied to the effect's +0x4b, chunk 0x6e
    /* +0x39 */ uint8_t  rotate_offset;   // [H] copied to the effect's +0x4a, chunk 0x6e
} C_PData_Emitter;
```

### 1h. `C_PData_Type`

The **emitter** definition, one per emitter of an effect type.

```c
typedef struct C_PData_Type {         // 0x9c bytes, C_PData at +0x00
    /* +0x0c */ uint8_t  phases_alloc;    // [H] allocator byte of the phase vector
    /* +0x10 */ C_PData_Phase **phases_begin; // [H] the phase definitions, chunk 0x66
    /* +0x14 */ C_PData_Phase **phases_end; // [H] one past the last definition
    /* +0x18 */ C_PData_Phase **phases_cap; // [H] end of the vector's storage
    /* +0x1c */ char    *frame_name;      // [H] the model's sub-frame to clone, chunk 0x69
    /* +0x20 */ float    offset[3];       // [H] copied to the record's offset, chunk 0x6a
    /* +0x2c */ float    offset_spread[3]; // [H] copied to the record's spread, chunk 0x6a
    /* +0x38 */ float    direction[3];    // [H] copied to the record, chunk 0x6b
    /* +0x44 */ float    direction_spread[3]; // [H] copied to the record, chunk 0x6b
    /* +0x50 */ float    speed;           // [H] copied to the record, chunk 0x6a
    /* +0x54 */ float    speed_spread;    // [H] copied to the record, chunk 0x6a
    /* +0x58 */ float    scale[3];        // [H] copied to the record, chunks 0x6a and 0x6c
    /* +0x64 */ float    scale_spread[3]; // [H] copied to the record, chunks 0x6a and 0x6c
    /* +0x70 */ float    alpha;           // [H] copied to the record, chunk 0x6a
    /* +0x74 */ float    alpha_spread;    // [H] copied to the record, chunk 0x6a
    /* +0x78 */ float    lifetime;        // [H] the record's lifetime and lifetime_initial
    /* +0x7c */ int32_t  burst_count;     // [H] the record's burst_count, chunk 0x6d
    /* +0x80 */ float    lifetime_decay;  // [H] the record's lifetime_decay
    /* +0x84 */ float    clock_threshold; // [H] truncated into the record's +0xa0
    /* +0x88 */ float    loops;           // [H] truncated into the record's +0xb0; -1 = forever
    /* +0x8c */ float    delay;           // [H] truncated into the record's +0xa8
    /* +0x90 */ uint32_t frame_mask_bit;  // [H] copied to the record's +0x2c
    /* +0x94 */ float    variant;         // [H] the emitter runs when the creator's variant equals it
    /* +0x98 */ uint32_t flags;           // [H] copied to the record's +0xbc
} C_PData_Type;
```

### 1i. `C_PData_Phase`

```c
typedef struct C_PData_Phase {        // 0x68 bytes, C_PData at +0x00
    /* +0x0c */ float    direction[3];    // [H] default (0, 1, 0), copied to the phase record
    /* +0x18 */ float    cone_axis[3];    // [H] copied to the phase record
    /* +0x24 */ float    axis_spin[3];    // [H] copied to the phase record
    /* +0x30 */ int32_t  duration;        // [H] default 1000, copied to the phase record +0x00
    /* +0x34 */ int32_t  duration_spread; // [H] copied to the phase record +0x04
    /* +0x38 */ float    scale[3];        // [H] default (1, 1, 1), chunks 0x6a and 0x6c
    /* +0x44 */ float    scale_spread[3]; // [H] chunks 0x6a and 0x6c
    /* +0x50 */ float    alpha;           // [H] default 1.0, copied to the phase record
    /* +0x54 */ float    alpha_spread;    // [H] copied to the phase record
    /* +0x58 */ float    spin;            // [H] copied to the phase record
    /* +0x5c */ float    spin_spread;     // [H] copied to the phase record
    /* +0x60 */ uint32_t flags;           // [H] default 1, copied to the phase record +0x08
    /* +0x64 */ float    gravity;         // [H] default 9.8, chunk 0x6b, copied to the record +0x60
} C_PData_Phase;
```

## 2. Field evidence

The cells come from a census of each class's functions and from the bodies. A record reached through a walking pointer (`record` inside `particle_effect_tick`, `phase` inside `particle_begin_phase`) has its cells read from those bodies.

### 2a. `particle_manager`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | live_alloc | H | `particle_manager_ctor` | - | the ctor copies an empty allocator object; it carries no value |
| +0x4 | live_begin | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_clear_emitters`, `particle_manager_destroy_particle`, `particle_manager_dtor`, `particle_manager_find_by_guid`, `particle_manager_tick` | - |
| +0x8 | live_end | H | `particle_manager_clear_emitters`, `particle_manager_create_particle_from_type`, `particle_manager_ctor`, `particle_manager_destroy_particle`, `particle_manager_dtor`, `particle_manager_tick` | `particle_manager_clear_emitters`, `particle_manager_create_particle_from_type`, `particle_manager_destroy_particle`, `particle_manager_find_by_guid`, `particle_manager_tick` | - |
| +0xc | live_cap | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_create_particle_from_type` | read inside `std_vector8_insert_at_005f6ce0` |
| +0x10 | types_alloc | H | `particle_manager_ctor` | - | as `live_alloc` |
| +0x14 | types_begin | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_clear_data`, `particle_manager_create_particle`, `particle_manager_create_static_particle`, `particle_manager_dtor`, `particle_manager_mission_start` | - |
| +0x18 | types_end | H | `particle_manager_clear_data`, `particle_manager_ctor`, `particle_manager_dtor`, `particle_manager_load_data` | `particle_manager_clear_data`, `particle_manager_create_particle`, `particle_manager_create_static_particle`, `particle_manager_load_data`, `particle_manager_mission_start` | - |
| +0x1c | types_cap | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_load_data` | read inside `std_vector4_insert_at` |
| +0x20 | statics_alloc | H | `particle_manager_ctor` | - | as `live_alloc` |
| +0x24 | statics_begin | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_dtor`, `particle_manager_mission_done`, `particle_manager_tick` | - |
| +0x28 | statics_end | H | `particle_manager_create_static_particle`, `particle_manager_ctor`, `particle_manager_dtor`, `particle_manager_mission_done` | `particle_manager_create_static_particle`, `particle_manager_mission_done`, `particle_manager_tick` | - |
| +0x2c | statics_cap | H | `particle_manager_ctor`, `particle_manager_dtor` | `particle_manager_create_static_particle` | read inside the vector helper |
| +0x30 | last_guid | H | `particle_manager_create_particle_from_type`, `particle_manager_ctor`, `particle_manager_mission_done`, `particle_manager_mission_start` | `particle_manager_create_particle_from_type`, `particle_manager_tick` | the tick skips the live list while it is -1 |
| +0x34 | started | H | `particle_manager_mission_done`, `particle_manager_mission_start` | `particle_manager_mission_done` | `mission_done` does nothing unless it is set |

### 2b. `particle_effect`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | parent | H | `particle_effect_destroy`, `particle_effect_init` | `particle_effect_spawn_particle` | - |
| +0x4 | records_alloc | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - | the inlined ctor, through `std_vector_construct_empty` |
| +0x8 | records_begin | H | `particle_effect_dtor` | `particle_effect_destroy`, `particle_effect_dtor`, `particle_effect_tick` | - |
| +0xc | records_end | H | `particle_effect_destroy`, `particle_effect_dtor` | `particle_effect_destroy`, `particle_effect_dtor`, `particle_effect_tick` | - |
| +0x10 | records_cap | H | `particle_effect_dtor` | `particle_manager_create_particle_from_type` | read inside `std_vector4_insert_at` |
| +0x14 | engine_id | H | `particle_effect_init` | `particle_effect_spawn_particle` | `particle_create` stamps it into the particle frame's `+0x21c`; it is not the manager's guid |
| +0x18 | position | H | `particle_effect_init` | `particle_effect_spawn_particle` | - |
| +0x24 | direction | H | `particle_effect_init` | `particle_effect_spawn_particle` | read only with record flag `0x4` |
| +0x30 | type_offset | H | `particle_effect_init` | `particle_effect_spawn_particle` | `C_PData_Emitter.offset`; added to `position`, rotated with `rotate_offset` |
| +0x3c | unread_3c | U | `particle_effect_init` | - | `C_PData_Emitter.unread_28`; the second vec3 of chunk `0x6a` is (0, 0, 0) in all 93 shipped types; no reader (class census) |
| +0x48 | spawn_enabled | H | `particle_effect_destroy`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_manager_set_particle_emitting` | `particle_effect_tick` | 1 from the ctor and `destroy` |
| +0x49 | paused | M | `particle_effect_destroy`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_effect_tick` | the tick's early return (alive, nothing done); every byte store at displacement `0x49` in `.text` was traced (16): the three on this field store 0, the rest have other bases or are misdecoded data; no non-zero store (§8d) |
| +0x4a | rotate_offset | H | `particle_effect_destroy`, `particle_effect_init` | `particle_effect_spawn_particle` | - |
| +0x4b | rotate_direction | H | `particle_effect_destroy`, `particle_effect_init` | `particle_effect_spawn_particle` | - |

### 2c. `S_emitter_record`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | unused_00 | U | - | - | no access (class census) |
| +0x4 | phases_alloc | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - | as `live_alloc` |
| +0x8 | phases_begin | H | `particle_type_dtor` | `particle_begin_phase`, `particle_create`, `particle_effect_tick`, `particle_tick`, `particle_type_dtor`, `particle_type_reset` | - |
| +0xc | phases_end | H | `particle_type_dtor`, `particle_type_reset` | `particle_create`, `particle_effect_tick`, `particle_type_dtor`, `particle_type_reset` | - |
| +0x10 | phases_cap | H | `particle_type_dtor` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | read inside `std_vector4_insert_at` |
| +0x14 | visual | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_init`, `particle_type_reset` | `particle_create`, `particle_type_dtor`, `particle_type_reset` | `particle_type_init` bumps the refcount at `visual+0x04`; `dtor` and `reset` call its slot 0 |
| +0x18 | parent | H | `particle_type_init` | `particle_begin_phase`, `particle_create` | - |
| +0x1c | particles_alloc | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - | as `live_alloc` |
| +0x20 | particles_begin | H | `particle_type_dtor` | `particle_effect_tick`, `particle_type_dtor`, `particle_type_reset` | - |
| +0x24 | particles_end | H | `particle_effect_tick`, `particle_type_dtor`, `particle_type_reset` | `particle_effect_spawn_particle`, `particle_effect_tick`, `particle_type_dtor`, `particle_type_reset` | - |
| +0x28 | particles_cap | H | `particle_type_dtor` | `particle_effect_spawn_particle` | read inside `std_vector4_insert_n` |
| +0x2c | frame_mask_bit | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_create` | - |
| +0x30 | offset | H | `particle_type_init` | `particle_create` | - |
| +0x3c | offset_spread | H | `particle_type_init` | `particle_create` | - |
| +0x48 | direction | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_create` | - |
| +0x54 | direction_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_create` | - |
| +0x60 | speed | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_create` | - |
| +0x64 | speed_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_create` | - |
| +0x68 | scale | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_create` | - |
| +0x74 | scale_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_create` | - |
| +0x80 | alpha | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_create` | - |
| +0x84 | alpha_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_create` | - |
| +0x88 | burst_count | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0x8c | lifetime | H | `particle_effect_tick`, `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0x90 | lifetime_initial | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0x94 | lifetime_decay | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0x98 | spawn_rate | H | `particle_effect_tick`, `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | `(1 / lifetime) * 1000`, 0 for a zero lifetime |
| +0x9c | decay_reciprocal | H | `particle_type_init`, `particle_type_reset` | - | `(1 / lifetime_decay) * 1000` from `particle_type_init`, 0 from `particle_type_reset`; no reader (class census) |
| +0xa0 | clock_threshold | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0xa4 | clock | H | `particle_effect_tick`, `particle_type_init` | `particle_effect_tick` | `reset` leaves it; `init` zeroes it on every reuse |
| +0xa8 | delay_initial | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0xac | delay | H | `particle_effect_tick`, `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | `init` stores -1, so the first spawn waits for no delay |
| +0xb0 | loops | H | `particle_effect_tick`, `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0xb4 | spawn_fraction | H | `particle_effect_tick`, `particle_type_reset` | `particle_effect_tick` | - |
| +0xb8 | loop_forever | H | `particle_type_init`, `particle_type_reset` | `particle_effect_tick` | - |
| +0xbc | flags | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase`, `particle_create`, `particle_effect_spawn_particle` | §2c-i |

#### 2c-i. `S_emitter_record.flags`

| Value | Meaning | Evidence |
|---|---|---|
| `0x01` | copied into the particle frame's byte `+0x270` | `particle_create` |
| `0x02` | the frame turns to the velocity, at creation and every tick | `particle_create` and `particle_tick`, through the particle's `+0x44`, call frame slot `+0x0c` |
| `0x04` | the start direction is the effect's `direction`, otherwise zero | `particle_effect_spawn_particle` |
| `0x08` | `direction` is turned by the parent's world matrix | `particle_create`, `particle_begin_phase` |
| `0x10` | the position offset is turned by the parent's world matrix | `particle_create` |

### 2d. `S_particle`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | frame | H | `particle_create` | `particle_begin_phase`, `particle_create`, `particle_dtor`, `particle_release`, `particle_tick` | `particle_release` calls frame slot `+0x2c` and keeps the frame; `particle_dtor` releases it |
| +0x4 | record | H | `particle_create`, `particle_release` | `particle_begin_phase`, `particle_create`, `particle_tick` | - |
| +0x8 | phase_index | H | `particle_begin_phase`, `particle_release` | `particle_effect_tick`, `particle_tick` | - |
| +0xc | velocity | H | `particle_begin_phase`, `particle_create`, `particle_tick` | `particle_begin_phase`, `particle_create`, `particle_tick` | - |
| +0x18 | turn_from | H | `particle_begin_phase` | `particle_tick` | `S_quat::SetDir`, `S_quat::Slerp` |
| +0x28 | turn_to | H | `particle_begin_phase` | `particle_tick` | - |
| +0x38 | turning | H | `particle_begin_phase` | `particle_tick` | set when the old and new directions have a dot product under 0.95 |
| +0x3c | phase_ms | H | `particle_begin_phase`, `particle_create`, `particle_release`, `particle_tick` | `particle_begin_phase`, `particle_effect_tick`, `particle_tick` | `particle_create` stores `0x7fffffff` on failure |
| +0x40 | unused_40 | U | - | - | no access (class census); the inlined ctor skips it |
| +0x44 | flags | H | `particle_create`, `particle_release` | `particle_tick` | - |
| +0x48 | unused_48 | U | - | - | no access (class census); the inlined ctor skips it |
| +0x4c | phase_duration | H | `particle_begin_phase` | `particle_tick` | - |
| +0x50 | scale_rate | H | `particle_begin_phase` | `particle_tick` | `(target - current) / duration`, against the frame's `+0x90` |
| +0x5c | alpha_rate | H | `particle_begin_phase`, `particle_release` | `particle_tick` | against the frame's `+0x214` |
| +0x60 | spin_rate | H | `particle_begin_phase`, `particle_release` | `particle_tick` | - |

### 2e. `S_phase_record`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | duration | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | `particle_begin_phase`: `duration + (2r - 1) * spread` with integer operands; `reset` zeroes it, the ctor too |
| +0x4 | duration_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x8 | flags | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase`, `particle_create`, `particle_tick` | §2e-i |
| +0xc | direction | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_begin_phase` | - |
| +0x18 | cone_axis | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_begin_phase` | non-zero calls `vec3_perturb_with_random_cross_basis` |
| +0x24 | axis_spin | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | `particle_tick` | - |
| +0x30 | unused_30 | U | - | - | the creators and `particle_type_reset` skip it; no access (class census) |
| +0x38 | scale | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x44 | scale_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x50 | alpha | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x54 | alpha_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x58 | spin | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x5c | spin_spread | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_begin_phase` | - |
| +0x60 | gravity | H | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`, `particle_type_reset` | `particle_tick` | `0x411ccccd` = 9.8; `velocity.y -= dt * gravity * 0.001` |

#### 2e-i. `S_phase_record.flags`

| Value | Meaning | Evidence |
|---|---|---|
| `0x02` | gravity applies | `particle_tick` |
| `0x08` | the particle's frame is off in this phase: slot `+0x24`, `SetOn`, gets 0, otherwise 1 | `particle_begin_phase`, `particle_create` for phase 0 |
| `0x10` | per-axis spin from `axis_spin` | `particle_tick` |
| `0x20` | no move: the velocity is not integrated | `particle_tick` |

### 2f. `C_PData`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vtable | H | `particle_data_ctor`, `particle_data_dtor`, `particle_data_emitter_dtor`, `particle_data_type_ctor`, `particle_data_type_dtor`, `particle_data_type_load_data`, `particle_manager_load_data` | `particle_data_emitter_dtor`, `particle_data_type_dtor`, `particle_manager_clear_data` | slot 0 is called with 1, the delete flag |
| +0x4 | kind | H | `particle_data_ctor`, `particle_data_type_ctor`, `particle_data_type_load_data`, `particle_manager_load_data` | - | no reader located; the kinds are what maps the classes (§8a) |
| +0x8 | name | H | `particle_data_ctor`, `particle_data_type_ctor`, `particle_data_type_load_data`, `particle_manager_load_data` | `particle_data_dtor`, `particle_data_emitter_dtor`, `particle_data_type_dtor` | a refcounted string; the destructors drop it |

### 2g. `C_PData_Emitter`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xc | emitters_alloc | H | `particle_manager_load_data` | - | as `live_alloc` |
| +0x10 | emitters_begin | H | `particle_data_emitter_dtor`, `particle_manager_load_data` | `particle_data_emitter_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x14 | emitters_end | H | `particle_data_emitter_dtor`, `particle_manager_load_data` | `particle_data_emitter_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x18 | emitters_cap | H | `particle_data_emitter_dtor`, `particle_manager_load_data` | `particle_manager_load_data` | read inside `std_vector4_insert_at` |
| +0x1c | offset | H | `particle_manager_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x28 | unread_28 | U | `particle_manager_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | the loads only copy it into the effect's `unread_3c`; (0, 0, 0) in all 93 shipped types |
| +0x34 | model_name | H | `particle_manager_load_data` | `particle_data_emitter_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x38 | rotate_direction | H | `particle_manager_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x39 | rotate_offset | H | `particle_manager_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |

### 2h. `C_PData_Type`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xc | phases_alloc | H | `particle_data_type_ctor` | - | as `live_alloc` |
| +0x10 | phases_begin | H | `particle_data_type_ctor`, `particle_data_type_dtor` | `particle_data_type_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x14 | phases_end | H | `particle_data_type_ctor`, `particle_data_type_dtor`, `particle_data_type_load_data` | `particle_data_type_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x18 | phases_cap | H | `particle_data_type_ctor`, `particle_data_type_dtor` | `particle_data_type_load_data` | read inside `std_vector4_insert_at` |
| +0x1c | frame_name | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_data_type_dtor`, `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x20 | offset | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x2c | offset_spread | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x38 | direction | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x44 | direction_spread | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x50 | speed | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x54 | speed_spread | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x58 | scale | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x64 | scale_spread | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x70 | alpha | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x74 | alpha_spread | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x78 | lifetime | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x7c | burst_count | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x80 | lifetime_decay | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x84 | clock_threshold | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x88 | loops | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x8c | delay | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x90 | frame_mask_bit | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x94 | variant | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | an emitter is skipped when the creator's variant is positive, this is positive, and they differ |
| +0x98 | flags | H | `particle_data_type_ctor`, `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |

### 2i. `C_PData_Phase`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xc | direction | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x18 | cone_axis | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x24 | axis_spin | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x30 | duration | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x34 | duration_spread | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x38 | scale | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x44 | scale_spread | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x50 | alpha | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x54 | alpha_spread | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x58 | spin | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x5c | spin_spread | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x60 | flags | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |
| +0x64 | gravity | H | `particle_data_type_load_data` | `particle_manager_create_particle_from_type`, `particle_manager_create_static_particle` | - |

## 3. Method family

All 47 PS2 methods of the seven classes are placed: in the tables below, as `inlined` rows, or in §8c. The PC `particle_type_*` functions carry no twin of their own (§8a). `C_PData_Phase` has no PC constructor or destructor of its own: its constructor is inlined and its vtable's slot 0 is the base's.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005a1b00 | `crt_static_init_guard_005a1b00` | - | - | - | - | opening guard of the first unit |
| 0x005a1b20 | `crt_atexit_register_005a1b20` | - | - | - | - | the guard's `atexit` registrar |
| 0x005a1d60 | `particle_dtor` | instance | `__dt__10C_ParticleFv` | 0x0047e630 | STRUCTURAL | `this = S_particle` releases the frame; the PS2 slot is the same, the bodies differ because each PC particle owns a frame |
| 0x005a3150 | `crt_static_init_guard_005a3150` | - | - | - | - | opening guard of the second unit |
| 0x005a3170 | `crt_atexit_register_005a3170` | - | - | - | - | the guard's `atexit` registrar |
| 0x005a3180 | `particle_effect_dtor` | instance | `__dt__17C_ParticleEmitterFv` | 0x0047f2a0 | STRUCTURAL | drains the record vector through `particle_type_dtor` and frees it |
| 0x005a38f0 | `crt_static_init_guard_005a38f0` | - | - | - | - | opening guard of the third unit |
| 0x005a3910 | `crt_atexit_register_005a3910` | - | - | - | - | the guard's `atexit` registrar |
| 0x005a3920 | `particle_pool_static_init` | `g_particle_pool` | - | - | - | static-init entry of the free list of particles |
| 0x005a3930 | `particle_pool_static_ctor` | `g_particle_pool` | - | - | - | builds the empty vector |
| 0x005a3950 | `particle_pool_register_atexit_dtor` | `g_particle_pool` | - | - | - | registers the static destructor |
| 0x005a3960 | `particle_pool_static_dtor` | `g_particle_pool` | - | - | - | frees the storage; `particle_manager_mission_done` drained the elements |
| 0x005a3980 | `particle_type_pool_static_init` | `g_particle_type_pool` | - | - | - | static-init entry of the free list of emitter records |
| 0x005a3990 | `particle_type_pool_static_ctor` | `g_particle_type_pool` | - | - | - | builds the empty vector |
| 0x005a39b0 | `particle_type_pool_register_atexit_dtor` | `g_particle_type_pool` | - | - | - | registers the static destructor |
| 0x005a39c0 | `particle_type_pool_static_dtor` | `g_particle_type_pool` | - | - | - | frees the storage; `particle_manager_mission_done` drained the elements |
| 0x005a39e0 | `particle_phase_pool_static_init` | `g_particle_phase_pool` | - | - | - | static-init entry of the free list of phase records |
| 0x005a39f0 | `particle_phase_pool_static_ctor` | `g_particle_phase_pool` | - | - | - | builds the empty vector |
| 0x005a3a10 | `particle_phase_pool_register_atexit_dtor` | `g_particle_phase_pool` | - | - | - | registers the static destructor |
| 0x005a3a20 | `particle_phase_pool_static_dtor` | `g_particle_phase_pool` | - | - | - | frees the storage; `particle_manager_mission_done` drained the elements |
| 0x005a3a40 | `particle_effect_pool_static_init` | `g_particle_effect_pool` | - | - | - | static-init entry of the free list of effects |
| 0x005a3a50 | `particle_effect_pool_static_ctor` | `g_particle_effect_pool` | - | - | - | builds the empty vector |
| 0x005a3a70 | `particle_effect_pool_register_atexit_dtor` | `g_particle_effect_pool` | - | - | - | registers the static destructor |
| 0x005a3a80 | `particle_effect_pool_static_dtor` | `g_particle_effect_pool` | - | - | - | frees the storage; `particle_manager_mission_done` drained the elements |
| 0x005a3aa0 | `particle_manager_ctor` | instance | `__ct__17C_ParticleManagerFv` | 0x00481720 | CONFIRMED | builds the three vectors and loads `tables\effects.tbl` |
| 0x005a3b20 | `particle_manager_dtor` | instance | `__dt__17C_ParticleManagerFv` | 0x004818c0 | CONFIRMED | clears the types and frees the three vectors |
| 0x005a5fb0 | `crt_static_init_guard_005a5fb0` | - | - | - | - | opening guard of the fourth unit |
| 0x005a5fd0 | `crt_atexit_register_005a5fd0` | - | - | - | - | the guard's `atexit` registrar |
| 0x005a63d0 | `particle_data_type_ctor` | instance | `__ct__12C_PData_TypeFv` | 0x00482c80 | CONFIRMED | `C_PData_Type`: kind 2 and the defaults |
| 0x005a69f0 | `particle_data_ctor` | instance | - | - | - | the base: vtable, kind 0, NULL name |
| 0x005a6a10 | `particle_data_scalar_deleting_dtor` | instance | `__dt__7C_PDataFv` | 0x00482eb0 | STRUCTURAL | slot 0 of `C_PData` and `C_PData_Phase` |
| 0x005a6a30 | `particle_data_dtor` | instance | `__dt__7C_PDataFv` | 0x00482eb0 | CONFIRMED | restores the base vtable and drops the name |
| 0x005a6a50 | `particle_data_type_scalar_deleting_dtor` | instance | - | - | - | slot 0 of `C_PData_Type` |
| 0x005a6a70 | `particle_data_type_dtor` | instance | `__dt__12C_PData_TypeFv` | 0x00483720 | CONFIRMED | `this = C_PData_Type` destroys the phase definitions and drops both strings |
| 0x005a6b50 | `particle_data_emitter_scalar_deleting_dtor` | instance | - | - | - | slot 0 of `C_PData_Emitter` |
| 0x005a6b70 | `particle_data_emitter_dtor` | instance | `__dt__15C_PData_EmitterFv` | 0x004835f0 | CONFIRMED | `this = C_PData_Emitter` destroys the emitter definitions and drops both strings |
| 0x005a6c50 | `particle_type_dtor` | instance | - | - | - | `this = S_emitter_record` frees the phase records, returns the particles to `g_particle_pool` and frees both vectors |

### 3b. Accessors

No class here has a one-load or one-store method; T2 lists none.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005a1d70 | `particle_release` | - | - | - | `this = S_particle` recycles a particle: frame slot `+0x2c`, then zeroes `record`, `phase_index`, `phase_ms`, `flags`, `alpha_rate`, `spin_rate` |
| 0x005a1da0 | `particle_create` | `Create__10C_ParticleFRC8S_matrixP12C_PData_Typeii` | 0x0047e680 | STRUCTURAL | `this = S_particle` builds the frame and rolls the start values from the record |
| 0x005a22d0 | `particle_begin_phase` | `UpdateDataByPhase__10C_ParticleFRC8S_matrix` | 0x0047e980 | CONFIRMED | `this = S_particle` enters a phase: duration, turn, target rates |
| 0x005a2830 | `particle_tick` | `Tick__10C_ParticleFi` | 0x0047eed0 | CONFIRMED | `this = S_particle` integrates position, spin, scale and alpha on the frame |
| 0x005a3220 | `particle_effect_destroy` | `Destroy__17C_ParticleEmitterFv` | 0x0047fb50 | CONFIRMED | returns the records to the free list and resets the bytes |
| 0x005a3290 | `particle_effect_init` | `Init__17C_ParticleEmitterFP9I3D_sceneP9I3D_frameRC8S_matrixP15C_PData_Emitterb` | 0x0047f410 | STRUCTURAL | stores the frame, vectors and flags and takes an engine id |
| 0x005a3320 | `particle_effect_spawn_particle` | `CreateParticle__17C_ParticleEmitterFP12C_PData_TypeRC8S_matrixii` | 0x0047fbc0 | CONFIRMED | `this = particle_effect` takes a particle from the free list and creates it; two callsites, the burst and the rate arm of the tick |
| - | - | `__ct__10C_ParticleFv` | 0x0047e620 | inlined | host `particle_effect_spawn_particle`: the seven field writes after its `operator_new` |
| 0x005a3570 | `particle_effect_tick` | `Tick__17C_ParticleEmitterFi` | 0x00480e90 | CONFIRMED | the whole emitter tick, a 4 -> 1 fold with `TickParticles`, `SpawnParticles`, `DeleteDeathParticles` (§8b) |
| - | - | `SpawnOnCreate__17C_ParticleEmitterFv` | 0x0047fe00 | inlined | host `particle_effect_tick`: the burst arm |
| - | - | `SpawnParticles__17C_ParticleEmitterFi` | 0x0047fef0 | inlined | host `particle_effect_tick`: the rate arm |
| - | - | `DeleteDeathParticles__17C_ParticleEmitterFv` | 0x004801a0 | inlined | host `particle_effect_tick`: the swap-erase of finished particles |
| - | - | `TickParticles__17C_ParticleEmitterFi` | 0x004802b0 | inlined | host `particle_effect_tick`: the `particle_tick` loop |
| 0x005a3ba0 | `particle_manager_mission_start` | `Mission_Start__17C_ParticleManagerFv` | 0x004819b0 | CONFIRMED | resets the guid, loads the table while the type list is empty, sets `started` |
| 0x005a3bf0 | `particle_manager_mission_done` | `Mission_Done__17C_ParticleManagerFv` | 0x00481a70 | CONFIRMED | destroys the effects and drains the four free lists |
| 0x005a3da0 | `particle_manager_load_data` | `LoadData__17C_ParticleManagerFv` | 0x00481b80 | CONFIRMED | parses `tables\effects.tbl` |
| 0x005a40b0 | `particle_manager_create_particle` | - | - | - | index check, then the creator |
| 0x005a4100 | `particle_manager_create_particle_from_type` | `CreateParticle__17C_ParticleManagerFiRC8S_matrixP9I3D_framebb` | 0x00482fb0 | STRUCTURAL | builds an effect from a type and issues its guid |
| - | - | `ValidateParticleData__17C_ParticleManagerFP15C_PData_Emitter` | 0x00483160 | inlined | host `particle_manager_create_particle_from_type`: the five early-return gates |
| - | - | `__ct__17C_ParticleEmitterFv` | 0x0047f1e0 | inlined | host `particle_manager_create_particle_from_type`: after its `operator_new(0x4c)` |
| 0x005a4af0 | `particle_manager_tick` | `ClientTick__17C_ParticleManagerFi` | 0x00483240 | CONFIRMED | ticks the level-placed effects, then the live ones, and retires the dead |
| 0x005a4bb0 | `particle_manager_find_by_guid` | `FindGuid__17C_ParticleManagerFiRi` | 0x00483370 | CONFIRMED | halving search of the live vector |
| 0x005a4c60 | `particle_manager_destroy_particle` | `DestroyParticle__17C_ParticleManagerFi` | 0x00483460 | CONFIRMED | retires one effect by guid |
| 0x005a4cd0 | `particle_manager_is_particle_alive` | `IsParticleAlive__17C_ParticleManagerFi` | 0x004838f0 | CONFIRMED | `find_by_guid != 0` |
| 0x005a4cf0 | `particle_manager_set_particle_emitting` | `PauseParticle__17C_ParticleManagerFib` | 0x004834c0 | STRUCTURAL | writes `spawn_enabled`; same slot as the PS2 `PauseParticle`, another bit (§8d) |
| 0x005a4d10 | `particle_manager_clear_emitters` | `ClearEmitters__17C_ParticleManagerFv` | 0x00483500 | CONFIRMED | destroys the live effects |
| 0x005a4d70 | `particle_manager_clear_data` | `ClearData__17C_ParticleManagerFv` | 0x00483570 | CONFIRMED | destroys the loaded types |
| 0x005a4dc0 | `particle_manager_create_static_particle` | `CreateSParticle__17C_ParticleManagerFiRC8S_matrixP9I3D_frame` | 0x00483910 | CONFIRMED | the same build from a type for a level-placed effect |
| 0x005a5750 | `particle_manager_load_static_particles` | `LoadStaticParticles__17C_ParticleManagerFv` | 0x00483a20 | CONFIRMED | parses `missions\%s\effects.bin` |
| 0x005a6490 | `particle_data_type_load_data` | `LoadData__12C_PData_TypeFP7C_chunk` | 0x004822b0 | CANDIDATE | reads an emitter definition with its phases |
| 0x005a6d80 | `particle_type_reset` | - | - | - | `this = S_emitter_record` the per-record half of `C_ParticleEmitter::Destroy` |
| 0x005a6f60 | `particle_type_init` | - | - | - | the per-record half of `C_ParticleEmitter::Init`, called by both creators |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005a1b30 | `vec3_perturb_with_random_cross_basis` | - | - | - | perturbs a direction about a random basis; its one caller is `particle_begin_phase` |
| 0x005a38d0 | `stl_copy_dwords_fastcall` | - | - | - | `std::copy` of dwords, stride 4; the free-list pops use it |

The fourth unit also holds `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining` and `chunk_read_string_and_leave`, which belong to `structs/chunk.md`.

## 4. Where it lives

`.CRT$XCU` fixes the boundaries, each confirmed by its own
`crt_static_init_guard`/`crt_atexit_register` pair:

| `.CRT$XCU` slot | range | unit | PC prefix |
|---|---|---|---|
| `.data:0x006461d4` | `[0x005a1b00, 0x005a3150)` | one particle | `particle_*` |
| `.data:0x006461d8` | `[0x005a3150, 0x005a38f0)` | one spawned **effect** | `particle_effect_*` |
| `.data:0x006461dc` | `[0x005a38f0, 0x005a5fb0)` | the manager + its four pools | `particle_manager_*` |
| `.data:0x006461f0` | `[0x005a5fb0, 0x005a7070)` | the `C_PData` definitions, the record's `particle_type_*` functions and four `C_chunk` helpers | `particle_data_*`, `particle_type_*` |

**The middle object is an EFFECT, not an emitter.** One PC object owns a *vector* of
`0xc0`-byte per-emitter runtime records — one per emitter definition in the effect type. So one
PC object is one spawned effect: one guid, one parent frame, N emitters, each with its own
live-particle list. The game's own vocabulary agrees: the two data files are `tables\effects.tbl`
and `missions\%s\effects.bin`.

### 4a. `C_mission + 0x6c`

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

## 5. The four free lists

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

## 6. The two shipped data files

### 6a. `tables\effects.tbl` — the effect-type table

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
| `0x6a` | emitter | 169 | two vec3 then fourteen dwords, `emitter+0x20`..`+0x98`, in the order of §1h |
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
`mission_init` creates the manager (§4a); `particle_manager_mission_start` loads it only while the
type list is empty.

### 6b. `missions\%s\effects.bin` — the level-placed effects

Read by `particle_manager_load_static_particles`. Outer chunk id must be **100** (the PC
accepts no other version, and all 79 shipped PC files are version 100). Each inner chunk id **1000** is one placement:

| field | size | meaning |
|---|---|---|
| matrix | `0x40` | the placement transform, passed straight to `particle_manager_create_static_particle` |
| type index | `4` | index into the manager's `+0x10` effect-type list |
| frame name | *(sub-chunk)* | resolved in the mission scene by name |

Its three failures are reported through `dbgPrintf`: *"Effects.bin - Error: could not
open"*, *"error in chunk"*, *"different version"*.

## 7. The level-placed effect registry in `C_game`

`C_game + 0x2c8` is a **stride-`0x18`** vector of `{guid, covered_flag, vec3 pos,
object*}`, walked every frame by `game_tick`. Per entry:

1. `particle_manager_is_particle_alive(guid)` — a dead effect is unlinked and swap-erased.
2. Beyond **150 units** of the player (`22500.0` = 150²) the effect is destroyed outright.
3. A vertical ray is cast **upward** from the effect's position. If it hits anything whose
   owner is not a `C_car`, `particle_manager_set_particle_emitting(guid, 0)` stops emission;
   when the sky is clear again it is switched back on with `1`.

⇒ **In game: a street plume stops making new particles under a bridge or an awning, and a
car parked over it does not count.** The particles alive by then keep drawing.

## 8. PC vs PS2

### 8a. The class map

| PC | PS2 | note |
|---|---|---|
| `particle_*` | `C_Particle` | 5 PC functions ↔ 5 PS2 methods |
| `particle_effect_*` | `C_ParticleEmitter` | 5 PC functions ↔ 15 PS2 methods (see 8b) |
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
`particle_type_init` is that half of `Init`, called by the manager's two creators (§8b);
`particle_type_reset` of `Destroy`, called only by `particle_effect_destroy`; `particle_type_dtor`
of the destructor, called by `particle_effect_dtor` and `particle_manager_mission_done`. The
`particle_effect_*` functions carry the twins.

The only genuine structural difference is **where the particles live**: the PS2 keeps one shared
particle list on `C_ParticleEmitter+0xb4`; the PC gives each `0xc0` record its own list at
`record+0x20`. Both builds keep one runtime record per emitter definition.

The manager's guid counter is PC `+0x30` ↔ PS2 `+0xc34`: `CreateParticle__17C_ParticleManager`
increments it, stores it beside the emitter and returns it, as the PC creator does.

### 8b. What the PC folds that the PS2 keeps out of line

Each site is marked in the decomp by an inline banner.

| PS2 method | PC site |
|---|---|
| `ValidateParticleData` (224 B) | the five early-return gates of `particle_manager_create_particle_from_type` |
| `TickParticles` + `SpawnParticles` + `DeleteDeathParticles` + `Tick` (1364 B) | **4 → 1 fold** into `particle_effect_tick` (864 B) |
| `SpawnOnCreate` (240 B) | the burst arm of `particle_effect_tick` |
| `__ct__10C_ParticleFv`, `__ct__17C_ParticleEmitterFv` | inlined at each `operator_new` site; the definition classes `C_PData_Emitter` and `C_PData_Phase` have no constructor symbol on the PS2 either |
| `C_ParticleEmitter::Init`'s record construction | hoisted into the manager's two creators |

### 8c. The PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00481010 | `UpdateOptimized__17C_ParticleEmitterFi` | an empty body (`jr ra; nop`); `particle_effect_init` makes no call where the PS2 `Init` and `ReInit` call it |
| 0x00480380 | `SortParticlesAndCreateBBOX__17C_ParticleEmitterFb` | the PS2 `Tick` sorts the particles and fills one vertex buffer per emitter; each PC particle owns its own frame (§1d), a visual of type `'PRTB'` (`I3D_VISUAL_PART_BASE` in `LS3DF.dll`'s type table) that `particle_tick` positions |
| 0x00480970 | `PrepareVBuffer__17C_ParticleEmitterFv` | as `SortParticlesAndCreateBBOX`: `particle_effect_tick` calls nothing between the particle update and the spawn |
| 0x0047f840 | `ReInit__17C_ParticleEmitterFP9I3D_sceneP9I3D_frameRC8S_matrixb` | the PS2 `CreateParticle` takes an emitter from a per-type free list and re-initialises it; the PC takes the effect from the type-agnostic `g_particle_effect_pool` and always runs `particle_effect_init` and one `particle_type_init` per emitter |
| 0x00480f80 | `GetWorldMatrix__17C_ParticleEmitterFv` | `particle_effect_spawn_particle` refreshes the parent's world matrix in place |
| 0x004849a0 | `PreloadTextures__17C_ParticleManagerFv` | a texture-handle list (manager `+0xc28..+0xc30`) filled by the game-init preloader `PreloadPicovinNaZasranyPSKram__Fv` and released by `Done__6C_gameFv`; the PC manager's `0x38` bytes are all mapped (§1a) and hold no such list |
| 0x00484a80 | `ReleasePreloadedTextures__17C_ParticleManagerFv` | as `PreloadTextures` |
| 0x00483e70 | `LoadStaticParticles_100__17C_ParticleManagerFR7C_chunk` | the PS2 `LoadStaticParticles` dispatches on the outer chunk id; the PC accepts one version and parses it inline (§8d) |
| 0x00484430 | `LoadStaticParticles_201__17C_ParticleManagerFR7C_chunk` | as `LoadStaticParticles_100` |
| 0x00482f00 | `CreateParticle__17C_ParticleManagerFiRC8S_vectorRC8S_vectorP9I3D_framebb` | builds a matrix from the position and direction and forwards to the matrix overload; a PC effect keeps the two vectors, so no such step exists. `particle_manager_create_particle` sits in the same place in front of the creator and takes an index check instead; no twin is graded |
| 0x00483850 | `__dt__13C_PData_PhaseFv` | the phase vtable's slot 0 holds `particle_data_scalar_deleting_dtor`, the base's |

### 8d. Two shipped behavioural differences

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

### 8e. PS2 `C_mission` embeds the manager; the PC holds a pointer

The PS2 `C_mission` constructor calls `__ct__17C_ParticleManagerFv((int)this + 100)` and the
destructor calls `__dt__17C_ParticleManagerFv(this + 100, ...)`, so on that build the manager is a
**member of `C_mission` at `+0x64`**, spanning to about `+0xcb0`. The PC `C_mission` holds a pointer
at `+0x6c` (§4a).

Consequence: every PS2 `C_mission` field above `+0x64` sits roughly **`0xc2c` higher** than its PC
counterpart, purely because of this member. Below `+0x64` the two maps agree. Do not carry a PS2
`C_mission` offset onto the PC by a constant (`structs/mission.md`).

The construction site also decides where `effects.tbl` is loaded. The PS2 constructor has no load
(the PS2 also builds a static `C_ParticleManager` in `__sinit_C_ParticleManager_cpp`), and
`Mission_Start` loads when the type list is empty. The PC constructor runs from `mission_init`
and loads there, through a real call; `particle_manager_mission_start` keeps the same empty-list
test. On both builds the table is therefore loaded once, before the first mission starts.

## 9. Related

`structs/game.md` (the `+0x2c8` registry) · `structs/mission.md` (the `+0x6c` pointer) ·
`structs/chunk.md` §4 (the open-chunk stack both loaders push through) ·
`structs/def_tables.md` (`particles.def`, a *different* table; see its §5) ·
`basic/artifacts/dll.md` (`ls3df.dll`, which owns `I3D_emitor`).

## 10. Open items

1. **In game:** does a street plume visibly stop emitting under an awning or a bridge, and resume when the camera moves it clear? (§7, step 3.)
2. **In game:** the 150-unit destroy radius of §7 step 2 should make distant ambient effects pop out; worth confirming it is visible.
