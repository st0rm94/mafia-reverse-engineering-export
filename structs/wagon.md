# `C_wagon` — entity kind `0x1e`, the scriptable rail wagon

**Created** 2026-08-18 21:09 +0200 · **Modified** 2026-11-09 18:19 +0100

A `C_actor` that drives itself along the rail web graph, brakes for stop nodes, waits and departs, and can
raise a script event. `structs/entity_kind.md` §3 separates it from the tram system `C_railway`.

| | |
|---|---|
| PC prefix | `wagon_` |
| PC kind (`+0x10`) | `0x1e` |
| PC vtable | `g_wagon_vt` @ `0x0063cff0` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x1a8` (`operator_new(0x1a8)` in `mission_create_actor`) |
| PC constructor | `wagon_ctor` @ `0x0053c2b0` |
| PC translation unit | `[0x0053c280, 0x005407f0)`, guard to guard: 21 starts, 17 776 bytes, all named |
| PS2 class | `C_wagon` (tag `7C_wagon`), source file `C_wagon.cpp`, 23 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base. The class also uses the kind `+0x10`, the
savegame GUID `+0x18`, the cached position `+0x24` and forward row `+0x30`, and the body frame `+0x68`.

```c
typedef struct C_wagon {                      // 0x1a8 bytes; C_actor base +0x00..+0x70
    /* +0x70 */ S_vector direction;           // [H] front bogie to the target node; the node-passed test, persisted
    /* +0x7c */ uint8_t  back_alloc;          // [H] allocator byte of the back path
    /* +0x80 */ uint32_t *back_begin;         // [H] back path: node indices behind the wagon
    /* +0x84 */ uint32_t *back_end;           // [H] one past the last back node
    /* +0x88 */ uint32_t *back_cap;           // [H] end of the back path's storage
    /* +0x8c */ float    back_path_radius;    // [H] how far the back path reaches
    /* +0x90 */ S_vector front_position;      // [H] the leading bogie's point
    /* +0x9c */ S_vector rear_position;       // [H] the trailing bogie's point
    /* +0xa8 */ uint8_t  forward_alloc;       // [H] allocator byte of the forward path
    /* +0xac */ uint32_t *forward_begin;      // [H] forward path: look-ahead node indices
    /* +0xb0 */ uint32_t *forward_end;        // [H] one past the last forward node
    /* +0xb4 */ uint32_t *forward_cap;        // [H] end of the forward path's storage
    /* +0xb8 */ float    *spring_phases;      // [H] sway phase of the body and of each vagon
    /* +0xbc */ S_vector *vagon_positions;      // [H] position of each trailing vagon
    /* +0xc0 */ S_vector *vagon_directions;     // [H] direction of each trailing vagon
    /* +0xc4 */ I3D_frame **vagon_frames;       // [H] model frame of each trailing vagon
    /* +0xc8 */ S_vector *position_deltas;    // [H] each body's position change in the last wagon_ai; no consumer (.text scan)
    /* +0xcc */ S_vector *direction_deltas;   // [H] each body's direction change in the last wagon_ai; no consumer (.text scan)
    /* +0xd0 */ float    target_speed;        // [H] the speed being approached
    /* +0xd4 */ float    speed;               // [H] the current speed
    /* +0xd8 */ uint8_t  halted;              // [H] standing at a stop
    /* +0xdc */ uint32_t halt_ms;             // [H] time standing, ms
    /* +0xe0 */ tDynamicCollObject coll;      // [H] the collision record, 0x48 bytes
    /* +0x128 */ S_matrix inverse_world;      // [H] inverse of the frame's world matrix
    /* +0x168 */ uint8_t forward_path_open; // [H] the forward path can still grow; wagon_ai extends it only while set
    /* +0x16c */ uint8_t wheels_alloc;        // [H] allocator byte of the wheel vector
    /* +0x170 */ I3D_frame **wheels_begin;    // [H] first wheel frame
    /* +0x174 */ I3D_frame **wheels_end;      // [H] one past the last wheel frame
    /* +0x178 */ I3D_frame **wheels_cap;      // [H] end of the wheel vector's storage
    /* +0x17c */ float   wheel_inverse_radius; // [H] converts distance to wheel rotation
    /* +0x180 */ uint32_t vagon_count;          // [H] trailing vagons; always 0 in the shipped code
    /* +0x184 */ float   bogie_spacing;       // [H] distance between a body's two bogies
    /* +0x188 */ float   vagon_length;          // [H] distance from one body to the next
    /* +0x18c */ float   cruise_speed;        // [H] the target speed a departure restores
    /* +0x190 */ float   deceleration;        // [H] the a of the braking distance
    /* +0x194 */ I3D_sound *running_sound;    // [H] nadzemka.wav
    /* +0x198 */ C_program *event_program;    // [H] script raised when the linked actor finishes
    /* +0x19c */ uint32_t event_id;           // [H] the event raised
    /* +0x1a0 */ int32_t path_clear_countdown; // [H] frames until the forward path is emptied
    /* +0x1a4 */ C_actor *linked_actor;       // [H] the actor the wagon waits for
} C_wagon;
```

**The per-vagon arrays.** Six heap arrays have no constant count: `wagon_game_init` allocates `spring_phases`,
`position_deltas` and `direction_deltas` with `vagon_count + 1` entries (entry 0 is this body) and `vagon_positions`,
`vagon_directions` and `vagon_frames` with `vagon_count` entries (one per trailing vagon). `vagon_length` and
`bogie_spacing` place each vagon on the back path: `wagon_ai` steps back `vagon_length - bogie_spacing`, then
`bogie_spacing`, for the vagon's two ends. Each vagon's frame is a clone of the body's own model
(`I3D_frame+0x154`, the model file name). The word *vagon* is `structs/railway.md`'s, whose same-shaped array `+0xc4` holds them; here `vagon_count` has no
non-zero writer, so all of this runs with zero vagons.

**The two delta arrays.** `wagon_ai` stores the body's position `+0x24` and forward row `+0x30` into entry 0 of `position_deltas` / `direction_deltas` and each vagon's position and direction into entry i + 1, runs `wagon_apply_spring_sway`, then stores new minus old; while halted it zeroes both arrays. They hold the change in the last `wagon_ai`, and nothing reads them (a `.text` scan of `+0xc8` / `+0xcc` finds `C_wagon` loads only in `wagon_ai`, `wagon_game_init` and `wagon_game_done`).

**The wheel vector has no constant count either:** one entry per frame whose name ends in `kolo` after its last
dot. The 4-character literal `s_wagon_wheel_frame_tag` (`0x0064c9b8`) is Czech for *wheel*, and that is what
makes `+0x170` a wheel vector; `wagon_update` spins each entry by a quaternion scaled with
`wheel_inverse_radius`.

**Both paths are capped at 100 entries** (§6).

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions; each hand-named
writer or reader beyond it was read in the body. An allocator byte's "readers" are `lea` of the vector, so
they are dropped from those rows.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | direction | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_save_game_load`, `wagon_save_game_save` | the vector from the front bogie to the target node: `wagon_ai` seeds it when its length is under 1e-16, rewrites it when a node is passed, and tests `S_vector::AngleTo` against the fresh to-node vector, 90 degrees or more or closer than 0.1 meaning passed; persisted (§7) |
| +0x7c | back_alloc | H | `wagon_ctor` | - | allocator byte |
| +0x80 | back_begin | H | `wagon_ctor`, `wagon_dtor` | `wagon_ai`, `wagon_distance_to_next_stop`, `wagon_dtor`, `wagon_extend_back_node_path`, `wagon_find_point_on_back_path`, `wagon_game_done`, `wagon_get_last_stop_node_id`, `wagon_save_game_get_size`, `wagon_save_game_load`, `wagon_save_game_save` | §6 |
| +0x84 | back_end | H | `wagon_ctor`, `wagon_dtor`, `wagon_game_done` | `wagon_ai`, `wagon_distance_to_next_stop`, `wagon_extend_back_node_path`, `wagon_find_point_on_back_path`, `wagon_game_done`, `wagon_get_last_stop_node_id`, `wagon_save_game_get_size`, `wagon_save_game_save` | - |
| +0x88 | back_cap | H | `wagon_ctor`, `wagon_dtor` | - | - |
| +0x8c | back_path_radius | H | `wagon_game_init` | `wagon_ai`, `wagon_extend_back_node_path` | `(vagon_count + 1) × vagon_length + 30`; the stop condition of `wagon_extend_back_node_path` |
| +0x90 | front_position | H | `wagon_ai` | `wagon_ai`, `wagon_distance_to_next_stop`, `wagon_game_init`, `wagon_save_game_load`, `wagon_save_game_save` | `+0x24 + forward × bogie_spacing / 2`; `wagon_distance_to_next_stop` measures from it |
| +0x9c | rear_position | H | `wagon_ai` | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load`, `wagon_save_game_save` | `+0x24 - forward × bogie_spacing / 2`; each frame the point `bogie_spacing` back along the path |
| +0xa8 | forward_alloc | H | `wagon_ctor` | - | allocator byte |
| +0xac | forward_begin | H | `wagon_ctor`, `wagon_dtor` | `wagon_ai`, `wagon_build_forward_node_path`, `wagon_distance_to_next_stop`, `wagon_dtor`, `wagon_game_done`, `wagon_game_init`, `wagon_save_game_get_size`, `wagon_save_game_load`, `wagon_save_game_save`, `wagon_update` | §6 |
| +0xb0 | forward_end | H | `wagon_ctor`, `wagon_dtor`, `wagon_game_done` | `wagon_ai`, `wagon_distance_to_next_stop`, `wagon_game_done`, `wagon_game_init`, `wagon_save_game_get_size`, `wagon_save_game_load`, `wagon_save_game_save`, `wagon_update` | - |
| +0xb4 | forward_cap | H | `wagon_ctor`, `wagon_dtor` | - | - |
| +0xb8 | spring_phases | H | `wagon_game_init` | `wagon_apply_spring_sway`, `wagon_game_done`, `wagon_game_init` | `vagon_count + 1` floats; entry 0 is this body, random start × π |
| +0xbc | vagon_positions | H | `wagon_game_init` | `wagon_ai`, `wagon_game_done`, `wagon_game_init`, `wagon_update` | `vagon_count` `S_vector`s |
| +0xc0 | vagon_directions | H | `wagon_game_init` | `wagon_ai`, `wagon_game_done`, `wagon_game_init`, `wagon_update` | `vagon_count` `S_vector`s |
| +0xc4 | vagon_frames | H | `wagon_game_done`, `wagon_game_init` | `wagon_apply_spring_sway`, `wagon_game_done`, `wagon_game_init`, `wagon_update` | `vagon_count` model clones of the frame's own model |
| +0xc8 | position_deltas | H | `wagon_game_init` | `wagon_ai`, `wagon_game_done`, `wagon_game_init` | `vagon_count + 1` `S_vector`s: each body's position change in the last `wagon_ai`, zeroed while halted; no consumer (.text scan) |
| +0xcc | direction_deltas | H | `wagon_game_init` | `wagon_ai`, `wagon_game_done`, `wagon_game_init` | `vagon_count + 1` `S_vector`s, the same for the direction; no consumer (.text scan) |
| +0xd0 | target_speed | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_save_game_save` | `cruise_speed` on departure, 0 when braking |
| +0xd4 | speed | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_apply_spring_sway`, `wagon_rebuild_all_forward_paths`, `wagon_save_game_save`, `wagon_update` | numerator of the braking distance; restarts from 0 on departure |
| +0xd8 | halted | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_save_game_save`, `wagon_update` | while set, `wagon_ai` counts `halt_ms` and departs after 10 000 ms |
| +0xdc | halt_ms | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_save_game_save` | - |
| +0xe0 | coll | H | `wagon_game_init`, `wagon_update` | `wagon_ai`, `wagon_cb_collect_wheel_frames`, `wagon_ctor`, `wagon_game_init`, `wagon_save_game_load`, `wagon_update` | type `0x40`; `collision_dyn_insert` / `collision_dyn_erase` |
| +0x128 | inverse_world | H | `wagon_game_init`, `wagon_save_game_load`, `wagon_update` (by address, `S_matrix::Inverse`) | `wagon_game_init`, `wagon_save_game_load`, `wagon_update` | the frame's world matrix inverted |
| +0x168 | forward_path_open | H | `wagon_build_forward_node_path`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_build_forward_node_path`, `wagon_save_game_save` | `wagon_build_forward_node_path` clears it before each step, sets it when the node has an onward link that is not the way back, and returns when it stayed 0 (a dead end); `wagon_ai` extends the forward path only while it is set; `wagon_game_init` seeds 1 |
| +0x16c | wheels_alloc | H | `wagon_ctor` | - | allocator byte |
| +0x170 | wheels_begin | H | `wagon_cb_collect_wheel_frames`, `wagon_ctor`, `wagon_dtor` | `wagon_dtor`, `wagon_game_init`, `wagon_update` | the `kolo` frames, filled by `wagon_cb_collect_wheel_frames` |
| +0x174 | wheels_end | H | `wagon_cb_collect_wheel_frames`, `wagon_ctor`, `wagon_dtor` | `wagon_update` | - |
| +0x178 | wheels_cap | H | `wagon_cb_collect_wheel_frames`, `wagon_ctor`, `wagon_dtor` | - | - |
| +0x17c | wheel_inverse_radius | H | `wagon_game_init` | `wagon_update` | `1 / (bound height of the first wheel / 2)` |
| +0x180 | vagon_count | H | `wagon_ctor` | `wagon_ai`, `wagon_game_done`, `wagon_game_init`, `wagon_update` | the ctor's 0 is its **only** writer, by a scan of every `[reg + 0x180]` store in `.text`; Open item 1 |
| +0x184 | bogie_spacing | H | `wagon_ctor` | `wagon_ai`, `wagon_game_init` | ctor 5.2; the only writer |
| +0x188 | vagon_length | H | `wagon_ctor` | `wagon_ai`, `wagon_game_init` | ctor 9.4; the only writer |
| +0x18c | cruise_speed | H | `wagon_ctor` | `wagon_ai`, `wagon_apply_spring_sway`, `wagon_game_init` | ctor 5.0; the sway phase rate is `speed / cruise_speed` |
| +0x190 | deceleration | H | `wagon_ctor` | `wagon_ai`, `wagon_rebuild_all_forward_paths` | ctor 0.5; denominator of the braking distance `v² / 2a` |
| +0x194 | running_sound | H | `wagon_game_done`, `wagon_game_init` | `wagon_ai`, `wagon_game_done`, `wagon_game_init` | `wagon_game_init` creates it as driver object type 4 and loads `nadzemka.wav` through the sound cache; `wagon_ai` restarts it on departure; `wagon_game_done` stops and releases it through slots `0x2c` / `0x24` / `0x00` |
| +0x198 | event_program | H | `wagon_game_init`, `wagon_save_game_load`, `wagon_set_event` | `wagon_ai`, `wagon_save_game_save` | `wagon_set_event`; the owner's GUID is what the save stores |
| +0x19c | event_id | H | `wagon_save_game_load`, `wagon_set_event` | `wagon_ai`, `wagon_save_game_save` | - |
| +0x1a0 | path_clear_countdown | H | `wagon_game_init`, `wagon_save_game_load`, `wagon_update` | `wagon_save_game_save`, `wagon_update` | `wagon_game_init` seeds 2; `wagon_update` decrements it; at zero it empties the forward path and sets act state 1 |
| +0x1a4 | linked_actor | H | `wagon_ai`, `wagon_game_init`, `wagon_save_game_load` | `wagon_ai`, `wagon_save_game_save`, `wagon_update` | while set, `wagon_ai` does nothing else: the actor's `+0x1c` at 2 raises the event and parks the wagon in act state 2, at 0 drops the link |

## 3. Method family

23 of 23 PS2 methods mapped: 18 twins (17 own functions and the `Init` fold) and 5 base stubs. PC-only: the
unit's CRT pair, the scalar deleting destructor, and the two script members `wagon_set_event` and
`wagon_get_last_stop_node_id`, which live in the script dispatcher's address range (§8a).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0053c280 | `crt_static_init_guard_0053c280` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0053c2a0 | `crt_atexit_register_0053c2a0` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x0053c2b0 | `wagon_ctor` | instance | `__ct__7C_wagonFv` | 0x0026f2d0 | CONFIRMED | `actor_ctor`, the three vectors, zeroes `coll`, vtable, kind `0x1e`, the five seeds of §1; caller `mission_create_actor` |
| 0x0053c360 | `wagon_scalar_deleting_dtor` | instance | - | - | STRUCTURAL | slot 16; MSVC-only |
| 0x0053c380 | `wagon_dtor` | instance | `__dt__7C_wagonFv` | 0x0026f3c0 | CONFIRMED | frees the three vectors, `actor_dtor` |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8ca0 | `wagon_set_event` | - | - | - | stores `(C_program *, event id)` at `+0x198` / `+0x19c`: the `wagon_setevent` command, from `program_process_command` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__7C_wagonFUi` | 0x00272b50 | base-stub | slot 15, the shared `ret 4` stub |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveData__7C_wagonFP7C_chunk` | 0x0026f4a0 | base-stub | slot 19, the shared default |
| 0x00462330 | `stub_rettrue_args4_00462330` | `LoadData__7C_wagonFP7C_chunk` | 0x0026f4b0 | base-stub | slot 20, the same stub |
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__7C_wagonFP7C_actor` | 0x0026fa90 | base-stub | slot 21, the shared default |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__7C_wagonF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0026faa0 | base-stub | slot 31, the shared default |
| 0x0053c400 | `folded_actor_init_bool` | `Init__7C_wagonFP9I3D_frame` | 0x0026f480 | CONFIRMED | slot 18; a fold with other owners (`structs/actor.md`) |
| 0x0053c420 | `wagon_save_game_get_size` | `SaveGameGetSize__7C_wagonFv` | 0x0026f4c0 | CONFIRMED | slot 22; base + `0x5f` + 4 per node of both paths |
| 0x0053c470 | `wagon_save_game_save` | `SaveGameSave__7C_wagonFPPv` | 0x0026f500 | CONFIRMED | slot 23; §7 |
| 0x0053c630 | `wagon_save_game_load` | `SaveGameLoad__7C_wagonFPPv` | 0x0026f6e0 | CONFIRMED | slot 24; §7 |
| 0x0053ca50 | `wagon_extend_back_node_path` | `CreateBackVector__7C_wagonFR8S_vector` | 0x0026fab0 | CONFIRMED | extends the back path up to `back_path_radius` |
| 0x0053cc40 | `wagon_build_forward_node_path` | `CreateForwardVector__7C_wagonFUif` | 0x0026fd70 | CONFIRMED | fills the forward path over a given distance |
| 0x0053d070 | `wagon_distance_to_next_stop` | `MustStop__7C_wagonFv` | 0x002702c0 | CONFIRMED | distance from `front_position` to the next stop node |
| 0x0053d2d0 | `wagon_find_point_on_back_path` | `FindBackPos__7C_wagonFR8S_vectorfPi` | 0x002705e0 | CONFIRMED | the point a given distance back along the back path |
| 0x0053d890 | `wagon_apply_spring_sway` | `SpringProcess__7C_wagonFUiiR8S_vectorR8S_vectorR8S_vectorR8S_vector` | 0x00270d30 | CONFIRMED | the sinusoidal suspension sway of one body (§8) |
| 0x0053dca0 | `wagon_ai` | `AI__7C_wagonFUi` | 0x00271140 | CONFIRMED | slot 13; §8 |
| 0x0053f360 | `wagon_update` | `Update__7C_wagonFUi` | 0x00272620 | CONFIRMED | slot 14; places the frames, spins the wheels, runs the countdown |
| 0x0053f7d0 | `wagon_game_init` | `GameInit__7C_wagonFv` | 0x00272b60 | CONFIRMED | slot 25; allocates the vagon arrays, builds both paths, the sound and the collision record |
| 0x00540590 | `wagon_cb_collect_wheel_frames` | `cbEnumFrame__Q27C_wagon21S_hlp$3866C_wagon_cppFP9I3D_frameRQ23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>>` | 0x00274010 | CONFIRMED | walks the frame tree and appends every `kolo` frame to the wheel vector |
| 0x00540620 | `wagon_game_done` | `GameDone__7C_wagonFv` | 0x00274200 | CONFIRMED | slot 26; empties the paths, releases the sound and the vagons' frames, `collision_dyn_erase` |
| 0x005c8cc0 | `wagon_get_last_stop_node_id` | - | - | - | the first back-path entry through the mission road-node table: the `wagon_getlastnode` command, from `program_process_command` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0053d040 | `wagon_node_is_stop_point` | `IsStopPoint__7C_wagonFUi` | 0x00270290 | CONFIRMED | bit 3 of the node's flags; no receiver in use |
| 0x00540770 | `wagon_rebuild_all_forward_paths` | `RecompileNewPath__7C_wagonFv` | 0x00274380 | CONFIRMED | the global re-plan of §8; its `this` is unused; caller `railway_switch_ai` |

`wagon_node_is_stop_point` and `wagon_rebuild_all_forward_paths` are member functions on the PS2 whose bodies do
not use the receiver. `wagon_rebuild_all_forward_paths` rebuilds **only** the forward vector; the back path is
read for its first node and never touched.

The unit is a **forced 1:1**: after its CRT pair, 19 PC starts = 1 MSVC scalar deleting destructor + 18 PS2
methods, in emission order.

## 4. How the class was identified

Two independent PC witnesses, neither of which needed a body diff (the source-file evidence is in §10):

1. **`wagon_ctor` chains `actor_ctor`** and installs vtable `0x0063cff0`, so it is a `C_actor` subclass rather
   than anything in the `C_Vehicle` family.
2. **`wagon_rebuild_all_forward_paths` filters on the kind byte:** it scans `g_actor_global_list` for actors whose
   `+0x10` is **`0x1e`**.

## 5. Why it sits exactly here: the link-order corroboration

The PC link order is alphabetical by source file, and five consecutive files lead up to this one (the other
build's link order agrees, §10):

```
C_turnout.cpp  ->  C_Vehicle.cpp  ->  _effects.cpp  ->  _kernel.cpp  ->  _sounds.cpp  ->  C_wagon.cpp
0x00508150         0x00509400        0x0051fcd0       0x0052c380      0x005373b0       0x0053c280
```

`C_recorder` follows at `0x005407f0` (see `structs/recorder.md`). So the continuous vehicle cluster is
`[0x00509400, 0x0053c280)` and **`C_wagon` is the first class after it** (`structs/vehicle.md` §8).

## 6. The two node paths, and what mapped them

`wagon_save_game_get_size` is statement-identical across the builds and its two `count * 4` terms are what
pinned both vectors at once:

| Role | PC |
|---|---|
| **back path** (behind the wagon; `wagon_find_point_on_back_path` walks it) | `+0x80` / `+0x84` |
| **forward path** (look-ahead; rebuilt over the braking distance) | `+0xac` / `+0xb0` |

Both are capped at **100 entries**. The FIRST entry of the back path is the wagon's last stop, which is why
`wagon_get_last_stop_node_id` and `wagon_rebuild_all_forward_paths` read only its first node.

The node records live in the rail web-node table hanging off `g_act_mission+0x2c`, stride **`0x1e`**:

| Offset | Field |
|---|---|
| `+0x00`..`+0x08` | position (3 floats) |
| `+0x0c` | flags; **bit 3 = this node is a stop** |
| `+0x0e` | node id (read by `wagon_get_last_stop_node_id`) |
| `+0x18` | edge array (8-byte entries; the flag byte at `+2` marks a reverse edge) |
| `+0x1d` | edge count |

## 7. The persisted record

`wagon_save_game_save` writes, after the `C_actor` block, a `0x5f`-byte header and then the node ids of both
paths:

| Record offset | Size | Content |
|---|---|---|
| `+0x00` | 1 | version, 1 |
| `+0x01` | 0x18 | `+0x24`..`+0x38`, the cached position and forward vector (`structs/actor.md`) |
| `+0x19` | 0x0c | `front_position` |
| `+0x25` | 0x0c | `rear_position` |
| `+0x31` | 4 | `target_speed` |
| `+0x35` | 4 | `speed` |
| `+0x39` | 1 | `halted` |
| `+0x3a` | 4 | `halt_ms` |
| `+0x3e` | 1 | `forward_path_open` |
| `+0x3f` | 4 | `path_clear_countdown` |
| `+0x43` | 4 | the savegame GUID (`actor+0x18`) of `linked_actor`, -1 when none |
| `+0x47` | 4 | the GUID of the actor owning `event_program`, -1 when none |
| `+0x4b` | 4 | `event_id` |
| `+0x4f` | 2 | forward-path node count |
| `+0x51` | 2 | back-path node count |
| `+0x53` | 0x0c | `direction` |
| `+0x5f` | 4 per node | the forward-path node ids, then the back-path node ids |

## 8. Behaviour

The wagon drives itself along the rail web graph. Each frame `wagon_ai`:

* keeps the forward path filled over the **braking distance** `v² / 2a`;
* asks `wagon_distance_to_next_stop` how far the next stop-point node is and brakes when it is inside that
  distance;
* places the trailing bogie with `wagon_find_point_on_back_path`;
* applies a sinusoidal suspension sway to both ends with `wagon_apply_spring_sway` (phase advanced by
  `speed / cruise_speed`, wrapped at 2π, offset `cos(phase) × 0.01`);
* resolves collisions; in the **kind-4 arm** (it hit a `C_car`) the PC has `C_Vehicle::DoCollisionStatic`
  **inlined**, which is marked in the decomp;
* halts at a stop: after 10 000 ms halted it departs, `target_speed` becomes `cruise_speed`, `speed` restarts from
  0 and `running_sound` is restarted.

`wagon_rebuild_all_forward_paths` is the global re-plan: when a `C_turnout` is thrown, every wagon in the world
rebuilds its look-ahead path (`structs/railway_switch.md` §5).

### 8a. Script-facing members

| PC | What |
|---|---|
| `wagon_set_event` (`0x005c8ca0`) | the `wagon_setevent` command |
| `wagon_get_last_stop_node_id` (`0x005c8cc0`) | the `wagon_getlastnode` command: the first element of `back_begin`, indexed into the mission road-node table (stride `0x1e`, id at `+0xe`) |

### 8b. Code shared with `C_railway`

Two members are compiled from the same source as their `C_railway` namesakes (`structs/railway.md`):

| `C_wagon` | `C_railway` | How they compare |
|---|---|---|
| `wagon_apply_spring_sway` (1040 B) | `railway_spring_process` (1040 B) | the same 289 instructions; the only differences are the speed field (`+0x18c` here, `+0x1b4` there) and the sway amplitude constant (0.01 here, 0.03 there) |
| `wagon_find_point_on_back_path` (1472 B) | `railway_find_back_pos` (1424 B) | the instruction sequences agree to 92 % |

## 9. Off-family finds from this unit

| PC name | What it is |
|---|---|
| `game_get_actor_from_guid` (`0x005f0410`) | `C_game::GetActorFromGUID`; names **`actor+0x18` = the savegame GUID** |
| `crt_strnicmp` (`0x00634770`) | MSVC CRT `_strnicmp` (`structs/string.md`) |
| `char_to_lower_locale` (`0x00626378`) | the locale/MBCS sibling of `ascii_toupper` |

## 10. PC vs PS2

* **Unit size.** PS2 `[0x0026f2d0, 0x00274450)` = 20 864 bytes against the PC's 17 776 (ratio **0.85**). GCC never
  emits the scalar deleting destructor. Size ratios of the rows run 0.62–1.33.
* **Source file.** `__sinit_C_wagon.cpp` is in the PS2 source-file manifest, so the file was compiled into the PS2
  build; the PS2 link order matches the PC chain in §5.
* **Five 8-byte PS2 stubs.** `SaveData`, `LoadData`, `DuplicateFrom`, `Hit` and `NetDirect` are genuine `return;` /
  `return 0;` bodies (not tail jumps); the PC folds them onto its shared vtable stubs (§3c).
* **`RecompileNewPath__7C_wagonFv`** is declared `(void)`, walks the global actor list filtering kind `0x1e`, clears
  the forward vector and calls `CreateForwardVector` with the same `v² / 2a` term: statement for statement
  `wagon_rebuild_all_forward_paths`.
* **`cbEnumFrame`** is a file-local helper struct method (`C_wagon::S_hlp`) on PS2.
* **Field delta.**

| Role | PC | PS2 |
|---|---|---|
| back path | `+0x80` / `+0x84` | `+0x104` / `+0x108` (the last-stop reader uses `+0x100` / `+0x104`) |
| forward path | `+0xac` / `+0xb0` | `+0x114` / `+0x118` |
| script event | `+0x198` / `+0x19c` | `+0x1cc` / `+0x1d0` |
| rail web-node stride | `0x1e` | `0x20` |

## 11. Related

`structs/railway.md` (`C_railway`, the tram system) · `structs/railway_switch.md` (`C_turnout`, which triggers the
global re-plan) · `structs/vehicle.md` §8 (the cluster map that ends at this class) and §12 (the
`DoCollisionStatic` inlining) · `structs/recorder.md` (the unit immediately after) · `structs/entity_kind.md` §1-3
(the kind map).

## 12. Open items

| Item | State |
|---|---|
| 1. `vagon_count` (`+0x180`) | the constructor's 0 is the only store a `[reg + 0x180]` scan of `.text` finds for this class, and `SaveData` / `LoadData` are stubs, so the six per-vagon arrays hold no vagon; a store through another register form or from scene data would change this |
| 2. in game: end stop | ride a wagon to an end stop and watch whether it reverses in place (`wagon_node_is_stop_point` / `wagon_distance_to_next_stop` / `wagon_extend_back_node_path`) |
| 3. in game: switch | throw a switch: every wagon should re-plan at once |
