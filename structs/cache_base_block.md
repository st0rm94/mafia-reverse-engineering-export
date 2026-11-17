# `C_cache_base_block` — the `cache.bin` object-streaming system

**Created** 2026-08-30 15:50 +0200 · **Modified** 2026-11-16 16:16 +0100

This is the system that streams the city's **static object instances** in and out around the
player. Every lamp post, bin, crate, bench and billboard that is placed in a mission but not part
of the baked scene geometry comes from `missions\%s\cache.bin` through this class. Blocks are
activated and deactivated by proximity as you move, which is the pop-in you see while driving.
The class and source-file names come from the PS2 build (§8).

| | |
|---|---|
| PC prefix | `cache_base_block_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x38` (the last field, `last_ref_pos`, ends at `+0x38`) |
| PC constructor | `cache_base_block_ctor` @ `0x005a7590`, run on the singleton `g_cache_base_block` @ `0x006efd80` by `cache_base_block_static_ctor` |
| PC translation unit | `[0x005a7520, 0x005aaa50)` (§7) |
| PS2 class | `C_cache_base_block` (tag `18C_cache_base_block`), 23 methods: 12 CONFIRMED twins, 2 STRUCTURAL and 3 inlined in §3; 6 PS2-only (§8e) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

PC offsets of the singleton. Each `vector` is the MSVC layout: an allocator byte, then begin /
end / capacity pointers.

```c
typedef struct C_cache_base_block {         // 0x38 bytes; the singleton g_cache_base_block
    /* +0x00 */ vector    blocks;               // [H] S_cache_base_block, stride 0x74 (§1a)
    /* +0x10 */ vector    model_sets;           // [H] S_cache_model_set, stride 0x14 (§1b)
    /* +0x20 */ uint32_t  retire_cursor;        // [H] the model set the next retire step looks at
    /* +0x24 */ uint8_t   loaded;               // [H] 1 between Open and Close
    /* +0x28 */ uint32_t  retest_accum_ms;      // [H] ms since the last full re-evaluation
    /* +0x2c */ S_vector  last_ref_pos;         // [H] the reference position at that re-evaluation
} C_cache_base_block;
```

The data symbols on the singleton are `g_cache_base_block_blocks_begin` / `_end` (`+0x04` /
`+0x08`) and `g_cache_base_block_loaded` (`+0x24`). The singleton's address is byte-proven, not
inferred: **15 references to `0x006efd80` in `.text`, every one a `mov ecx, 0x6efd80`**, and
`cache_base_block_static_ctor` is literally `mov ecx, 0x6efd80 / jmp cache_base_block_ctor`. Like
the `G_TextDatabase` singleton, the object itself has **no textual existence in the decomp** —
every access is `__thiscall`, so the receiver travels in ECX and Ghidra emits no argument;
`g_cache_base_block` is a registry row, not a symbol in the decomp.

### 1a. `S_cache_base_block` — one streamed block

```c
typedef struct S_cache_base_block {         // 0x74 bytes; blocks element
    /* +0x00 */ I3D_frame *frame;               // [H] the block's own frame in the scene
    /* +0x04 */ uint8_t    active;              // [H] the block is streamed in
    /* +0x08 */ vector     children;            // [H] the block's I3D_frame children
    /* +0x18 */ float      center_x;            // [H] footprint centre X
    /* +0x1c */ float      center_z;            // [H] footprint centre Z
    /* +0x20 */ float      radius;              // [H] footprint radius
    /* +0x24 */ S_vector2  corner[4];           // [H] the footprint's four XZ corners
    /* +0x44 */ S_vector2  edge_normal[4];      // [H] the outward normals of its four edges
    /* +0x64 */ vector     instances;           // [H] S_cache_model_inbb, stride 0x44 (§1d)
} S_cache_base_block;
```

**Why four corners and four edge normals.** A block's footprint is a quadrilateral in the XZ
plane. `cache_base_block_is_block_in_range` rejects first on the centre and the radius, then runs
the four half-space tests (one per edge normal) and the four corner distance tests; the loader
reads the footprint as the centre, the radius and four `8 + 8`-byte pairs, one corner and one
edge normal per side (§4). The count is that of a quadrilateral.

### 1b. `S_cache_model_set` — one distinct model

```c
typedef struct S_cache_model_set {          // 0x14 bytes; model_sets element
    /* +0x00 */ char   *name;                   // [H] the model name, owned
    /* +0x04 */ vector  pool;                   // [H] S_cache_model_one, stride 8 (§1c)
} S_cache_model_set;
```

### 1c. `S_cache_model_one` — one pooled instance

```c
typedef struct S_cache_model_one {          // 8 bytes; pool element
    /* +0x00 */ I3D_object *object;             // [H] the pooled I3D object
    /* +0x04 */ uint8_t     in_use;             // [H] the instance is placed in an active block
} S_cache_model_one;
```

### 1d. `S_cache_model_inbb` — one placed instance

*Model in base block*; the type name is the PS2 parameter type of `DestroyFrame`.

```c
typedef struct S_cache_model_inbb {         // 0x44 bytes; instances element
    /* +0x00 */ uint16_t block_index;           // [H] the owning block
    /* +0x02 */ uint16_t model_set_index;       // [H] 0xffff = none
    /* +0x04 */ uint16_t slot_index;            // [H] slot within that set's pool; 0xffff = none
    /* +0x08 */ S_vector pos;                   // [H] the instance position
    /* +0x14 */ S_quat   rot;                   // [H] the instance rotation
    /* +0x24 */ S_vector scale;                 // [H] the instance scale
    /* +0x30 */ float    unread_30;             // [U] the cache.bin float after the scale; no reader (class census)
    /* +0x34 */ uint8_t  enabled;               // [H] the instance is placed
    /* +0x38 */ S_vector visual_scale;          // [H] the scale of each type-1 child of the pooled model
} S_cache_model_inbb;
```

## 2. Field evidence

The singleton by a census of every `this+N` access in the `cache_base_block_*` functions; the
element records from the bodies of the functions named in their rows.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | blocks | H | `cache_base_block_ctor` (empty), `cache_base_block_open` (`cache_base_block_blocks_insert_at`), `cache_base_block_close`, `cache_base_block_dtor` | `cache_base_block_tick`, `cache_base_block_activate_block`, `cache_base_block_deactivate_block`, `cache_base_block_deactivate_all_blocks`, `cache_base_block_is_block_in_range`, `cache_base_block_destroy_frame`; outside the class `game_load_telephones`, `game_destroy_control_signal` | begin / end are `g_cache_base_block_blocks_begin` / `_end` |
| +0x10 | model_sets | H | `cache_base_block_ctor` (empty), `cache_base_block_open_model` (`cache_base_block_model_sets_insert_at`), `cache_base_block_close`, `cache_base_block_dtor` | `cache_base_block_tick`, `cache_base_block_activate_block`, `cache_base_block_deactivate_block`, `cache_base_block_find_model`, `cache_base_block_destroy_frame`, `cache_base_block_deactivate_all_blocks`, `cache_base_block_open_model` | one set per distinct model name |
| +0x20 | retire_cursor | H | `cache_base_block_open`, `cache_base_block_tick` (advance, wrap) | `cache_base_block_tick` | one pooled instance retired per frame (§5) |
| +0x24 | loaded | H | `cache_base_block_ctor`, `cache_base_block_close` (= 0), `cache_base_block_open` (= 1) | `cache_base_block_is_loaded`; through the data symbol `engine_game_loop`, `game_done`, `game_load_telephones`, `game_destroy_control_signal`, `game_build_traffic_signal_model_idx`, `human_movement`, `mission_update_settings`, `race_game_loop`, `race_game_loop_net` | - |
| +0x28 | retest_accum_ms | H | `cache_base_block_ctor`, `cache_base_block_open`, `cache_base_block_tick`, `cache_base_block_deactivate_all_blocks` (= 10000) | `cache_base_block_tick` | the full re-evaluation runs every 100 ms (§5) |
| +0x2c | last_ref_pos | H | `cache_base_block_open`, `cache_base_block_tick` | `cache_base_block_tick` | a move of more than 100 units (`10000.0` squared) forces a re-evaluation |

### 2a. `S_cache_base_block`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | frame | H | `cache_base_block_open` (the frame name resolved in the scene) | `cache_base_block_activate_block`, `cache_base_block_deactivate_block` | shown and hidden by them |
| +0x04 | active | H | `cache_base_block_activate_block`, `cache_base_block_deactivate_block`, `cache_base_block_deactivate_all_blocks` | `cache_base_block_tick` | the gate both block functions write |
| +0x08 | children | H | `cache_base_block_open` (the frame's type-1 children, each switched off) | `cache_base_block_free_block` (frees it) | §8e: the PS2 runs this as `AssociateBaseBlocks` |
| +0x18 | center_x | H | `cache_base_block_open` | `cache_base_block_is_block_in_range` | the first reject |
| +0x1c | center_z | H | `cache_base_block_open` | `cache_base_block_is_block_in_range` | the first reject |
| +0x20 | radius | H | `cache_base_block_open` | `cache_base_block_is_block_in_range` | compared squared against `(r + margin)²` |
| +0x24 | corner | H | `cache_base_block_open` | `cache_base_block_is_block_in_range` | the four corner distance tests |
| +0x44 | edge_normal | H | `cache_base_block_open` | `cache_base_block_is_block_in_range` | the four half-space tests |
| +0x64 | instances | H | `cache_base_block_open` | `cache_base_block_activate_block`, `cache_base_block_deactivate_block`, `cache_base_block_free_block` (frees it) | - |

### 2b. `S_cache_model_set`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | name | H | `cache_base_block_open_model` | `cache_base_block_open_model` | owned by the record; the lookup compares it with each requested name through `crt_stricmp` |
| +0x04 | pool | H | `cache_base_block_activate_block` (appends when every slot is taken), `cache_base_block_tick` (`cache_base_block_model_pool_erase_at`) | `cache_base_block_activate_block`, `cache_base_block_find_model`, `cache_base_block_tick` | - |

### 2c. `S_cache_model_one`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | object | H | `cache_base_block_activate_block` (create and name) | `cache_base_block_find_model`, `cache_base_block_tick` (destroys a retired one) | - |
| +0x04 | in_use | H | `cache_base_block_activate_block` (= 1), `cache_base_block_deactivate_block` (= 0) | `cache_base_block_activate_block` (the free-slot search), `cache_base_block_deactivate_block`, `cache_base_block_tick` | the retire step frees only an unused last slot |

### 2d. `S_cache_model_inbb`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | block_index | H | `cache_base_block_open` (the block count at the time of the read) | `cache_base_block_destroy_frame` | bounds-checked against the block count, then the block's `active` is read |
| +0x02 | model_set_index | H | `cache_base_block_open` (from `cache_base_block_open_model`) | `cache_base_block_deactivate_block`, `cache_base_block_destroy_frame` | `0xffff` tested by `cache_base_block_deactivate_block` |
| +0x04 | slot_index | H | `cache_base_block_activate_block` | `cache_base_block_deactivate_block`, `cache_base_block_destroy_frame` | `0xffff` tested by `cache_base_block_deactivate_block` |
| +0x08 | pos | H | `cache_base_block_open` | `cache_base_block_activate_block` | copied into the pooled frame's position `+0x80` under the write guard |
| +0x14 | rot | H | `cache_base_block_open` | `cache_base_block_activate_block` | copied into the frame's rotation `+0x9c`, then normalised |
| +0x24 | scale | H | `cache_base_block_open` | `cache_base_block_activate_block` | copied into the frame's scale `+0x90` |
| +0x30 | unread_30 | U | `cache_base_block_open` (the `cache.bin` float after the scale) | - | only copied with the whole record (`std_vector44_*`, `cache_base_block_instances_copy_assign`, the block copy); no reader (class census) |
| +0x34 | enabled | H | `cache_base_block_open` (= 1), `cache_base_block_destroy_frame` (= 0), `cache_base_block_deactivate_all_blocks` (= 1) | `cache_base_block_activate_block`, `cache_base_block_deactivate_block`, `cache_base_block_destroy_frame` | only an enabled instance is placed or returned to its pool |
| +0x38 | visual_scale | H | `cache_base_block_open` | `cache_base_block_activate_block` | copied into the scale `+0x90` of every type-1 child of the pooled model, with the rotation / scale flag rewrite |

## 3. Method family

Coverage: **17 of 23** PS2 `18C_cache_base_block` methods are in the tables below — **12
CONFIRMED**, **2 STRUCTURAL** (the constructor and destructor) and **3 inlined** into a CONFIRMED
twin. The other six have no PC function (§8e). `cache_base_block_walk_block_objects` and
`cache_base_block_is_loaded` are PC-only; the container helpers are compiler-emitted (§6).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005a7520 | `crt_static_init_guard_005a7520` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x005a7540 | `crt_atexit_register_005a7540` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x005a7550 | `cache_base_block_static_init` | `g_cache_base_block` | - | - | - | the singleton's `.CRT$XCU` entry |
| 0x005a7560 | `cache_base_block_static_ctor` | `g_cache_base_block` | - | - | - | `mov ecx, 0x6efd80 / jmp cache_base_block_ctor` — the receiver proof |
| 0x005a7570 | `cache_base_block_register_atexit_dtor` | `g_cache_base_block` | - | - | - | `atexit(&cache_base_block_static_dtor)` |
| 0x005a7580 | `cache_base_block_static_dtor` | `g_cache_base_block` | - | - | - | `mov ecx, 0x6efd80 / jmp cache_base_block_dtor` |
| 0x005a7590 | `cache_base_block_ctor` | instance | `__ct__18C_cache_base_blockFv` | 0x0010cd40 | STRUCTURAL | empties both vectors and clears `loaded`; 44 bytes against 116 ¹ |
| 0x005a75c0 | `cache_base_block_dtor` | instance | `__dt__18C_cache_base_blockFv` | 0x0010cde0 | STRUCTURAL | frees both vectors and everything they own; 331 bytes against 140 ¹ |
| 0x005a8180 | `cache_base_block_free_block` | `S_cache_base_block` | - | - | - | the element destructor: frees `instances` and `children` |
| 0x005a9720 | `cache_base_block_blocks_destroy` | `blocks` | - | - | - | the `blocks` vector destructor; no `E8` caller, its one referrer an `E9` tail-jump from the unwind funclet at `0x00638853` (§6) |
| 0x005aa5f0 | `cache_base_block_block_construct_copy` | `S_cache_base_block` | - | - | - | the element copy constructor, stride `0x74` |
| 0x005aa740 | `cache_base_block_model_set_construct_copy` | `S_cache_model_set` | - | - | - | the element copy constructor, stride `0x14` |
| 0x005aa7f0 | `cache_base_block_instances_copy_assign` | `instances` | - | - | - | the `instances` vector copy-assignment |

¹ Both bodies match the PS2 statement for statement apart from the `vector` constructors and
destructors that MSVC inlined and GCC calls out of line, hence STRUCTURAL.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8290 | `cache_base_block_is_loaded` | - | - | - | PC-only; returns `loaded`; out of the unit, in the `0x005c8xxx` helper tail |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005a7710 | `cache_base_block_close` | `Close__18C_cache_base_blockFv` | 0x0010ce70 | CONFIRMED | unloads everything; called from `mission_close` and from `cache_base_block_open` |
| 0x005a7800 | `cache_base_block_prepare_scene` | `PrepareScene__18C_cache_base_blockFv` | 0x0010d020 | CONFIRMED | groups the scene's `wcol` collision frames (§5a) |
| 0x005a7830 | `cache_base_block_prepare_scene_cb` | `cbDupl__Q218C_cache_base_block22S_hlp$2095_m_cache_cppFP9I3D_frameUi` | 0x0010d060 | CONFIRMED | the frame-walk callback that does it (§8c) |
| 0x005a7910 | `cache_base_block_open` | `Open__18C_cache_base_blockFP7C_chunkP9I3D_scene` | 0x0010d1e0 | CONFIRMED | **the `cache.bin` loader** (§4) |
| - | - | `AssociateBaseBlocks__18C_cache_base_blockFv` | 0x0010e8d0 | inlined | open-coded in `cache_base_block_open` (§8e) |
| 0x005a81b0 | `cache_base_block_open_model` | `OpenModel__18C_cache_base_blockFPCc` | 0x0010ec60 | CONFIRMED | model name → model-set index; loads the model through `g_i3d_model_cache` |
| 0x005a85c0 | `cache_base_block_tick` | `Tick__18C_cache_base_blockFUi` | 0x0010f5b0 | CONFIRMED | **the per-frame streamer** (§5) |
| 0x005a8a20 | `cache_base_block_activate_block` | `ActivateBlock__18C_cache_base_blockFUi` | 0x0010f850 | CONFIRMED | streams one block in |
| - | - | `ShowModel__18C_cache_base_blockFUsPUs` | 0x00110530 | inlined | open-coded in `cache_base_block_activate_block` (§8e) |
| 0x005a8e80 | `cache_base_block_deactivate_block` | `DeactivateBlock__18C_cache_base_blockFUi` | 0x0010fd40 | CONFIRMED | streams one block out |
| - | - | `HideModel__18C_cache_base_blockFUsUs` | 0x00110a00 | inlined | open-coded in `cache_base_block_deactivate_block` (§8e) |
| 0x005a8f90 | `cache_base_block_walk_block_objects` | - | - | - | PC-only; the recursive per-object setup that starts the fade-in (§5, §8d) |
| 0x005a90e0 | `cache_base_block_find_model` | `FindModel__18C_cache_base_blockFUsUs` | 0x00110a90 | CONFIRMED | `(set, slot)` → the pooled `I3D` object |
| 0x005a9120 | `cache_base_block_destroy_frame` | `DestroyFrame__18C_cache_base_blockFP18S_cache_model_inbb` | 0x00110ae0 | CONFIRMED | returns one instance's model to its pool slot |
| 0x005a91f0 | `cache_base_block_deactivate_all_blocks` | `VisibleAllFrames__18C_cache_base_blockFv` | 0x00110c20 | CONFIRMED | drops everything and forces a re-test (§8c) |
| 0x005a93c0 | `cache_base_block_is_block_in_range` | `IsBlockInRange__18C_cache_base_blockFUi9S_vector2f` | 0x00110fb0 | CONFIRMED | the footprint test (§1a) |
| 0x005a97a0 | `cache_base_block_model_pool_erase_at` | - | - | - | erases one `S_cache_model_one` from a pool |
| 0x005a97e0 | `cache_base_block_blocks_insert_at` | - | - | - | the `blocks` insert |
| 0x005a99e0 | `cache_base_block_blocks_destroy_range` | - | - | - | runs `cache_base_block_free_block` over a range |
| 0x005a9a10 | `cache_base_block_model_sets_insert_at` | - | - | - | the `model_sets` insert |
| 0x005a9be0 | `cache_base_block_model_sets_destroy_range` | - | - | - | destroys a range of model sets |
| 0x005a9c50 | `cache_base_block_blocks_uninit_copy` | - | - | - | copy-constructs a range of blocks |
| 0x005a9c90 | `cache_base_block_blocks_uninit_fill_n` | - | - | - | fill-constructs N blocks |
| 0x005a9cc0 | `cache_base_block_model_sets_uninit_copy` | - | - | - | copy-constructs a range of model sets |
| 0x005a9d00 | `cache_base_block_model_sets_uninit_fill_n` | - | - | - | fill-constructs N model sets |
| 0x005a9d30 | `cache_base_block_blocks_copy_assign_forward` | - | - | - | copy-assigns a range of blocks, forward |
| 0x005a9ee0 | `cache_base_block_blocks_copy_assign_range` | - | - | - | copy-assigns a range of blocks |
| 0x005aa0a0 | `cache_base_block_blocks_copy_backward` | - | - | - | copy-assigns a range of blocks, backward |
| 0x005aa250 | `cache_base_block_model_sets_copy_assign_range` | - | - | - | copy-assigns a range of model sets |
| 0x005aa400 | `cache_base_block_model_sets_copy_backward` | - | - | - | copy-assigns a range of model sets, backward |

### 3d. Free helpers

These four carry mechanical names, which assert no owner (§6).

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005a85a0 | `vector_dispose_at_0x08` | - | - | - | frees the buffer of a vector whose begin sits at `+0x08`; 9 callers, among them `cache_base_block_close`, `cache_base_block_open_model` and `cache_base_block_model_sets_destroy_range` |
| 0x005a9760 | `std_vector14_destroy` | - | - | - | a stride-`0x14` vector destructor; callers `game_dtor` and three unwind funclets, none in this class |
| 0x005a9c10 | `std_vector44_uninit_copy` | - | - | - | a stride-`0x44` range copy; its only caller is `std_vector44_insert_one` |
| 0x005aa5d0 | `std_vector44_copy_one` | - | - | - | a stride-`0x44` element copy; its only caller is `std_vector44_insert_one` |

## 4. The `cache.bin` file format

Read by `cache_base_block_open` off a `C_chunk` that `mission_open` has opened on
`missions\%s\cache.bin`.

| Level | Chunk id | Contents |
|---|---|---|
| file | **500** (`0x1f4`) | a version dword, which must be **1** |
| block | **1000** (`0x3e8`) | a length-prefixed frame name resolved in the mission scene, then `8 + 4` bytes and four `8 + 8` byte pairs — the footprint (§1a) |
| instance | **2000** (`0x7d0`) | a length-prefixed model name, then `pos` (vec3), `rot` (quat), `scale` (vec3), `unread_30` (float) and `visual_scale` (vec3) of `S_cache_model_inbb` |

The loader fills each `S_cache_model_inbb` on the stack: `block_index` = the block count at the
time, `model_set_index` from `cache_base_block_open_model`, `slot_index` = `0xffff`, the five file
fields, `enabled` = 1.

Failure returns:

| Return | Cause |
|---|---|
| 1 | wrong outer chunk id |
| 2 | wrong version |
| 3 | block frame not in the scene |
| 4 | model not found ¹ |

¹ Also raises `MessageBoxA` with the title **`Chybi model`** ("Missing model") and the text
`model "%s" not found.`

🔑 **A shipped data fix-up lives in the loader:** an instance whose model name is `0rek23.i3d` is
silently loaded as **`0rek17.i3d`** instead.

## 5. How streaming actually behaves

`cache_base_block_tick` runs from `engine_game_loop`:

1. One instance is retired per frame through the round-robin `retire_cursor`.
2. The full re-evaluation runs only **every 100 ms**, or immediately when the reference frame has
   moved more than **100 units** (`10000.0` compared squared). The reference frame is the
   mission scene's `+0x17c`.
3. A block further away than its own radius **+ 230.0**, or failing the four edge and four corner
   tests, is **deactivated**.
4. A block within **200.0** of the reference point that is not active is **activated**.

`game_init` and `program_process_command` also call it with `0x0fffffff` as the frame time, which
forces a complete re-evaluation in a single call.

⇒ **In game: small props pop in at a set distance while driving.** That distance is the
200 / 230-unit band above.

**The pop-in is softened by a fade.** `cache_base_block_walk_block_objects` prunes subtrees whose
scaled bound extents are all below **2.0** and hands every `'OBJ_'`-tagged child (magic
`0x5f4a424f`) of a newly activated block to `mission_fade_in_object` with the literal **700** — a
700 ms alpha fade-in that `mission_tick` ramps from 0 to 1.

**Retiring pooled instances.** Step 1 is the only place pooled `I3D` instances are freed: each
tick looks at the model set under `retire_cursor`, and when that set's last pool slot is not in
use it destroys the object and erases the slot. The cursor then advances and wraps.

### 5a. The `wcol` / `LLwcol` frames

`cache_base_block_prepare_scene` runs at the start of `cache_base_block_open` and walks every frame
of the mission scene through `cache_base_block_prepare_scene_cb`. A frame whose name starts with
`LLwcol` or `wcol` (case-insensitive) is linked under the first such frame; the first one is linked
under the scene frame at `scene+0x210` and switched off. The frames are **invisible collision
geometry**, measured from the shipped mission data:

| Measurement | Result |
|---|---|
| frame kind in `scene.4ds` | ordinary mesh frames (visual type 0), 943 distinct names across 37 missions |
| mission `tree.klz` collision name table | 886 of the 943 names appear in the same mission's `tree.klz` |
| `scene2.bin` frame records | 530 of 532 records of these frames carry the empty chunk `0x4033`, which `mission_open_bin` applies as "switch the frame off" (frame vtable `+0x24` with 0); 165 of 113,496 other frame records carry it |

So the scene data hides them; the cache pass only groups them under one frame. It runs only
in the 25 missions that ship a `cache.bin`, because `mission_open` calls `cache_base_block_open`
only when that file opens.

### 5b. The `I3D` model cache

`cache_base_block_open_model` loads each model through `g_i3d_model_cache`, the name-keyed `I3D`
model cache of `structs/i3d_cache.md` (`g_i3d_model_cache_items_begin` / `_items_end`,
`g_i3d_model_cache_size_limit`). The same inlined lookup block sits in
`particle_manager_create_particle_from_type` (`structs/particle.md` §8).

## 6. Why the container helpers carry the class prefix

Identical-code folding is enabled in this link — one `mov eax,[ecx+0x10]; ret` serves two
unrelated classes, and `folded_get_ptr_0x90` is a two-owner fold whose every callsite sits in a
single function — so "every caller is in this unit" alone does not prove a helper is unshared.

All 18 class-prefixed helpers in this unit (and the 10 anim-machine ones) pass three measurements:

1. **byte-unique** — the exact body occurs once in `.text`;
2. **0 datarefs** — a scan of `.rdata` / `.data` for the address as a pointer finds nothing, so it
   is in no vtable and no class can reach it virtually;
3. **no foreign referrer** — a byte scan of `.text` for `E8` calls and `E9` jumps to the address
   returns only this unit.

And the decisive one, which is body-internal and immune to folding: **each helper calls its
element type's own named constructor or destructor at the element stride** —
`cache_base_block_block_construct_copy` and `cache_base_block_free_block` for stride `0x74`,
`cache_base_block_model_set_construct_copy` for stride `0x14`. A fold of two classes' vector
helpers would require the two element types to have the same constructor, which is the same as
saying they are one type.

Two measured details:

* `cache_base_block_blocks_destroy` (`0x005a9720`) has **zero `E8` callers** — its sole referrer
  is an `E9` tail-jump from the unwind funclet at `0x00638853`, the normal-path destructor being
  inlined.
* `anim_track_table_destroy_range` has a second caller in `engine_shutdown`, which passes
  `ecx = 0x6efdbc`, the track table's **own** vector header — the static-table teardown inlined
  into shutdown, not a foreign owner.

The four helpers of §3d carry **mechanical** names, which assert no owner: their callers lie
outside this class (`std_vector14_destroy`, the two `std_vector44_*`) or span several
(`vector_dispose_at_0x08`).

## 7. The translation unit

`.CRT$XCU` at `.data:0x00646210` opens it and `.data:0x00646218` opens the next, so the unit is
**`[0x005a7520, 0x005aaa50)`** — 44 function starts, 0 phantoms: the CRT pair, the singleton's
static-init quartet, the class's own functions, the 18 class-prefixed container helpers and the
four mechanical ones (§3). `cache_base_block_is_loaded` lies outside it.

## 8. PC vs PS2

**PS2 class:** `C_cache_base_block` (mangled tag `18C_cache_base_block`), source file
**`m_cache.cpp`** — named by the file-local static `S_hlp$2095_m_cache_cpp`.

### 8a. Which class the PC ships

The PS2 has **two** near-identical cache classes: `C_cache_base_block` (`m_cache.cpp`) and
`C_cache_base_block_ex` (`m_cache_ex.cpp`), the latter with `S_cache_model_set` /
`S_cache_model_one` as *nested* types. Six methods exist only on the non-`_ex` class —
`FindModel`, `DestroyFrame`, `VisibleAllFrames`, `IsBlockInRange`, `PrepareScene`, `cbDupl` —
and the PC has a counterpart for **all six**. That closed-set agreement settles the
identification.

The PC has **no `C_cache_base_block_ex`**. The PS2 mission loader opens a second file,
`missions\%s\cache_ex.bin`, into a second singleton of the `_ex` class right after `cache.bin`.
The PC loader opens `cache.bin` only: the string `cache_ex` does not occur in `Game.exe`, and no
shipped mission contains a `cache_ex.bin`.

### 8b. Method sizes

| PC name | PS2 size | PC size |
|---|---|---|
| `cache_base_block_ctor` | 116 | 44 |
| `cache_base_block_dtor` | 140 | 331 |
| `cache_base_block_close` | 424 | 239 |
| `cache_base_block_prepare_scene` | 64 | 39 |
| `cache_base_block_prepare_scene_cb` | 372 | 217 |
| `cache_base_block_open` | 5276 | 2155 |
| `cache_base_block_open_model` | 1844 | 1002 |
| `cache_base_block_tick` | 664 | 1108 |
| `cache_base_block_activate_block` | 1256 | 1107 |
| `cache_base_block_deactivate_block` | 364 | 263 |
| `cache_base_block_find_model` | 80 | 53 |
| `cache_base_block_destroy_frame` | 316 | 202 |
| `cache_base_block_deactivate_all_blocks` | 848 | 455 |
| `cache_base_block_is_block_in_range` | 1320 | 859 |

### 8c. Two names that diverge from the PS2 verb

* `VisibleAllFrames` → **`cache_base_block_deactivate_all_blocks`**. The body hides every active
  block, clears every active flag, returns every model to its pool and forces the re-test
  accumulator to 10000. It makes nothing visible; the next tick re-shows whatever is still in
  range.
* `cbDupl` → **`cache_base_block_prepare_scene_cb`**. "Dupl" describes the intent (fold
  duplicates onto one frame); the PC name says where it is used from.

### 8d. `cache_base_block_walk_block_objects` is not `GetNumModelVisual` / `cbVisCount`

Those two PS2 methods sit in the matching position but are a pure recursive *count* of visible
children, capturing the `'OBJ_'` one into an out-parameter. The PC function prunes small subtrees
and starts the 700 ms fade-in (§5). Only the `0x5f4a424f` magic is shared.

### 8e. The nine PS2 methods with no PC function of their own

Three are inlined into a CONFIRMED PC twin (§3c), read statement for statement against the PS2
body:

| PS2 method | Inlined into | What matches |
|---|---|---|
| `ShowModel` | `cache_base_block_activate_block` | the free-slot search over the pool, the create-and-name path when every slot is taken, the in-use byte set to 1. The PS2 tail (bound-volume test and fade-in) is `cache_base_block_walk_block_objects` on the PC (§8d) |
| `HideModel` | `cache_base_block_deactivate_block` | both indices tested against `0xffff`, the in-use byte tested and cleared, the same frame call with `(0, 0)` |
| `AssociateBaseBlocks` | `cache_base_block_open` | per block: find the frame by name in the scene, switch it off, collect its type-1 children into `children`, switching each off. The PS2 runs it as a separate pass after `Open` succeeds |

Six are PS2-only, each a measurement over a closed set of callers:

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0010feb0 | `IsModelOnlyObject__18C_cache_base_blockFUs` | the PS2 has a second placement path for a model whose only visual is one `'OBJ_'` object, keyed on a model-set field the PC record does not have (PS2 model set `0x18` bytes, PC `0x14`). Its only PS2 callers are `Open`, `ActivateBlock` and `DeactivateBlock`; the PC twins of all three have the model path only |
| 0x0010fee0 | `ShowObject__18C_cache_base_blockFUs` | the same second placement path |
| 0x00110500 | `HideObject__18C_cache_base_blockFP10I3D_object` | the same second placement path |
| 0x0010ea90 | `GetNumModelVisual__18C_cache_base_blockFP9I3D_modelPP10I3D_object` | it feeds that same decision; its only PS2 caller is `OpenModel`, and `cache_base_block_open_model` always pools the loaded model |
| 0x0010eb40 | `cbVisCount__Q218C_cache_base_block22S_hlp$2923_m_cache_cppFP9I3D_framePiPP10I3D_object` | the callback of `GetNumModelVisual` |
| 0x001114e0 | `ClearUnusedModels__18C_cache_base_blockFv` | the PS2 frees every unused pooled instance after a block is deactivated; the PC frees one per frame from the tick's round-robin retire step (§5) |

## 9. Related

- `structs/chunk.md` — the `C_chunk` the loader runs on.
- `structs/i3d_cache.md` — `g_i3d_model_cache`, the model cache the pool loads through.
- `structs/particle.md` §8 — the same model cache, decoded from the other side.
- `structs/mission.md` — `mission_fade_in_object`.
- `basic/artifacts/dll.md` — `rw_data.dll`, the archive layer under `C_chunk`.

## 10. Open items

1. **In game:** the `0rek23.i3d` → `0rek17.i3d` substitution should be observable — the model that
   `0rek23` names never appears anywhere in the city.
