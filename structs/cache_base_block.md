# `C_cache_base_block` — the `cache.bin` object-streaming system

**Created** 2026-08-30 15:50 +0200 · **Modified** 2026-09-17 17:54 +0200

**PC prefix: `cache_base_block_`.** **PC singleton: `g_cache_base_block` @ `0x006efd80`.**
Class and source-file names come from the PS2 build (§7).

This is the system that streams the city's **static object instances** in and out around
the player. Every lamp post, bin, crate, bench and billboard that is placed in a mission
but not part of the baked scene geometry comes from `missions\%s\cache.bin` through this
class. Blocks are activated and deactivated by proximity as you move, which is the pop-in
you see while driving.

---

## 1. The translation unit

`.CRT$XCU` at `.data:0x00646210` opens it and `.data:0x00646218` opens the next, so the
unit is **`[0x005a7520, 0x005aaa50)`** — 44 function starts, 0 phantoms.

The singleton's address is byte-proven, not inferred: **15 references to `0x006efd80` in
`.text`, every one a `mov ecx, 0x6efd80`**, and the CRT thunk
`cache_base_block_static_ctor` is literally `mov ecx, 0x6efd80 / jmp cache_base_block_ctor`.

⚠️ Like the `G_TextDatabase` singleton, the object itself has **no textual existence in the
decomp** — every access is `__thiscall`, so the receiver travels in ECX and Ghidra emits no
argument. `g_cache_base_block` is a function-registry row, not a symbol in the decomp.

## 2. Object layout (PC)

```c
typedef struct C_cache_base_block {
    /* +0x00 */ uint8_t  blocks_alloc;      // [H] STL allocator byte
    /* +0x04 */ void    *blocks_begin;      // [H] vector<S_cache_base_block>, stride 0x74
    /* +0x08 */ void    *blocks_end;        // [H]   — the loaded blocks
    /* +0x0c */ void    *blocks_cap;        // [H]
    /* +0x10 */ uint8_t  model_sets_alloc;  // [H] STL allocator byte
    /* +0x14 */ void    *model_sets_begin;  // [H] vector<S_cache_model_set>, stride 0x14
    /* +0x18 */ void    *model_sets_end;    // [H]   — one per distinct model name
    /* +0x1c */ void    *model_sets_cap;    // [H]
    /* +0x20 */ uint32_t retire_cursor;     // [H] round-robin: one instance retired per frame
    /* +0x24 */ uint8_t  loaded;            // [H] set by Open, cleared by Close/ctor
    /* +0x28 */ uint32_t retest_accum_ms;   // [H] ms since the last full re-evaluation
    /* +0x2c */ float    last_ref_pos[3];   // [H] reference position at the last re-test
} C_cache_base_block;                       // 0x38 bytes
```

The data symbols on the singleton are `g_cache_base_block_blocks_begin` / `_end` (`+0x04` /
`+0x08`) and `g_cache_base_block_loaded` (`+0x24`).

### 2a. `S_cache_base_block` — one streamed block, 0x74 bytes

| Offset | Meaning | Evidence |
|---|---|---|
| `+0x00` | the block's own `I3D_frame` | shown/hidden by `cache_base_block_activate_block` / `_deactivate_block` |
| `+0x04` | active flag | the gate both of those write |
| `+0x0c` .. `+0x14` | `vector<I3D_frame*>` — the block's child objects, collected at load | `cache_base_block_free_block` frees it |
| `+0x18` / `+0x1c` | footprint centre X / Z | first reject in `cache_base_block_is_block_in_range` |
| `+0x20` | footprint radius | the same reject, compared squared against `(r + margin)²` |
| `+0x24` .. `+0x3c` | four XZ corner points | the four corner distance tests |
| `+0x44` .. `+0x60` | four edge normals | the four half-space tests |
| `+0x68` .. `+0x70` | `vector<S_cache_model_inbb>`, stride 0x44 — the block's instances | `cache_base_block_free_block` frees it |

### 2b. `S_cache_model_set` — one distinct model, 0x14 bytes

| Offset | Meaning |
|---|---|
| `+0x00` | the model name (`char *`, owned) |
| `+0x08` .. `+0x10` | `vector<S_cache_model_one>`, stride 8 — the pool of `I3D` instances of this model |

### 2c. `S_cache_model_one` — one pooled instance, 8 bytes

| Offset | Meaning |
|---|---|
| `+0x00` | the `I3D` object |
| `+0x04` | in-use byte |

### 2d. `S_cache_model_inbb` — one placed instance, 0x44 bytes

*Model in base block* (the type name is the PS2 parameter type of `DestroyFrame`).

| Offset | Meaning |
|---|---|
| `+0x00` | block index |
| `+0x02` | model-set index (`0xffff` = none) |
| `+0x04` | slot index within that set (`0xffff` = none) |
| `+0x08` .. `+0x13` | position |
| `+0x14` .. `+0x23` | rotation quaternion |
| `+0x24` .. `+0x2f` | scale |
| `+0x34` | enabled byte |
| `+0x38` .. `+0x43` | a second vec3 applied to the instance's `'OBJ_'` child |

### 2e. The `I3D` model cache

`g_i3d_model_cache_begin` / `_end` / `_limit` is a name-keyed `I3D` model cache (not an entity
pool). It is decoded from two independent sites carrying the identical inlined block:
`cache_base_block_open_model` and `particle_manager_create_particle_from_type`.

## 3. The `cache.bin` file format

Read by `cache_base_block_open` off a `C_chunk` that `mission_open` has already opened on
`missions\%s\cache.bin`.

| level | chunk id | contents |
|---|---|---|
| file | **500** (`0x1f4`) | a version dword, which must be **1** |
| block | **1000** (`0x3e8`) | a length-prefixed frame name resolved in the mission scene, then `8 + 4` bytes and four `8 + 8` byte pairs — the footprint |
| instance | **2000** (`0x7d0`) | a length-prefixed model name, then vec3 + quat + vec3 + float + vec3 |

Failure returns:

| return | cause |
|---|---|
| 1 | wrong outer chunk id |
| 2 | wrong version |
| 3 | block frame not in the scene |
| 4 | model not found ¹ |

¹ Also raises `MessageBoxA` with the Czech title **`Chybi model`** ("Missing model") and the text
`model "%s" not found.`

🔑 **A shipped data fix-up lives in the loader:** an instance whose model name is
`0rek23.i3d` is silently loaded as **`0rek17.i3d`** instead.

## 4. How streaming actually behaves

`cache_base_block_tick` runs from `engine_game_loop`:

1. One instance is retired per frame through the round-robin cursor at `+0x20`.
2. The full re-evaluation runs only **every 100 ms**, or immediately when the reference
   frame has moved more than **100 units** (`10000.0` compared squared). The reference
   frame is the mission scene's `+0x17c`.
3. A block further away than its own radius **+ 230.0**, or failing the four edge and four
   corner tests, is **deactivated**.
4. A block within **200.0** of the reference point that is not already active is
   **activated**.

`game_init` and `program_process_command` also call it with `0x0fffffff` as the frame time,
which forces a complete re-evaluation in a single call.

⇒ **In game: small props pop in at a fixed distance while driving.** That distance is the
200/230-unit band above.

**The pop-in is softened by a fade.** `cache_base_block_walk_block_objects` prunes subtrees
whose scaled bound extents are all below **2.0** and hands every `'OBJ_'`-tagged child
(magic `0x5f4a424f`) of a newly activated block to `mission_fade_in_object` with the literal
**700** — a 700 ms alpha fade-in that `mission_tick` ramps from 0 to 1.

## 5. PC function set

| PC VA | PC name | role |
|---|---|---|
| 0x005a7550 | `cache_base_block_static_init` | `.CRT$XCU` entry |
| 0x005a7560 | `cache_base_block_static_ctor` | `mov ecx, 0x6efd80 / jmp ctor` — the receiver proof |
| 0x005a7570 | `cache_base_block_register_atexit_dtor` | `atexit(&cache_base_block_static_dtor)` |
| 0x005a7580 | `cache_base_block_static_dtor` | `mov ecx, 0x6efd80 / jmp dtor` |
| 0x005a7590 | `cache_base_block_ctor` | zero both vectors, clear the loaded flag |
| 0x005a75c0 | `cache_base_block_dtor` | free both vectors and everything they own |
| 0x005a7710 | `cache_base_block_close` | unload; called from `mission_close` and from `Open` |
| 0x005a7800 | `cache_base_block_prepare_scene` | fold the scene's `wcol` collision frames |
| 0x005a7830 | `cache_base_block_prepare_scene_cb` | the frame-walk callback that does it |
| 0x005a7910 | `cache_base_block_open` | **the `cache.bin` loader** |
| 0x005a8180 | `cache_base_block_free_block` | one `S_cache_base_block` destructor |
| 0x005a81b0 | `cache_base_block_open_model` | model name → model-set index |
| 0x005a85c0 | `cache_base_block_tick` | **the per-frame streamer** |
| 0x005a8a20 | `cache_base_block_activate_block` | stream one block in |
| 0x005a8e80 | `cache_base_block_deactivate_block` | stream one block out |
| 0x005a8f90 | `cache_base_block_walk_block_objects` | recursive per-object setup, starts the fade-in (§4) |
| 0x005a90e0 | `cache_base_block_find_model` | `(set, slot)` → the pooled `I3D` object |
| 0x005a9120 | `cache_base_block_destroy_frame` | return one instance's model to its pool slot |
| 0x005a91f0 | `cache_base_block_deactivate_all_blocks` | drop everything, force a re-test |
| 0x005a93c0 | `cache_base_block_is_block_in_range` | the footprint test |
| 0x005c8290 | `cache_base_block_is_loaded` | the `+0x24` accessor — **out of the TU**, in the `0x005c8xxx` helper tail |

Plus 17 `std::vector` instantiations named for the container they serve
(`cache_base_block_blocks_*`, `_model_sets_*`, `_model_pool_*`, `_instances_*`) — see §6.

## 6. Why the container helpers carry the class prefix

Identical-code folding is enabled in this link — one `mov eax,[ecx+0x10]; ret` serves two
unrelated classes, and `folded_get_ptr_0x90` is a two-owner fold whose every callsite sits in a
single function — so "every caller is in this TU" alone does not prove a helper is unshared.

All 18 helpers in this TU (and the 10 anim-machine ones) pass three measurements:

1. **byte-unique** — the exact body occurs once in `.text`;
2. **0 datarefs** — a scan of `.rdata`/`.data` for the address as a pointer finds nothing, so it
   is in no vtable and no class can reach it virtually;
3. **no foreign referrer** — a byte scan of `.text` for `E8` calls and `E9` jumps to the address
   returns only this TU.

And the decisive one, which is body-internal and immune to folding: **each helper calls its
element type's own named ctor/dtor at the element stride** —
`cache_base_block_block_construct_copy` and `cache_base_block_free_block` for stride `0x74`,
`cache_base_block_model_set_construct_copy` for stride `0x14`. A fold of two classes' vector
helpers would require the two element types to have the same constructor, which is the same
as saying they are one type.

Two measured details:

* `cache_base_block_blocks_destroy` (`0x005a9720`) has **zero `E8` callers** — its sole referrer
  is an `E9` tail-jump from the unwind funclet at `0x00638853`, the normal-path destructor being
  inlined.
* `anim_track_table_destroy_range` has a second caller in `engine_shutdown`, which passes
  `ecx = 0x6efdbc`, the track table's **own** vector header — the static-table teardown inlined
  into shutdown, not a foreign owner.

Four helpers in the same VA range carry **mechanical** names, which assert no owner:
`vector_dispose_at_0x08`, `std_vector14_destroy`, `std_vector44_uninit_copy` and
`std_vector44_copy_one`.

## 7. PC vs PS2

**PS2 class:** `C_cache_base_block` (mangled tag `18C_cache_base_block`), source file
**`m_cache.cpp`** — named by the file-local static `S_hlp$2095_m_cache_cpp`.

### 7a. Which class the PC ships

The PS2 has **two** near-identical cache classes: `C_cache_base_block` (`m_cache.cpp`) and
`C_cache_base_block_ex` (`m_cache_ex.cpp`), the latter with `S_cache_model_set` /
`S_cache_model_one` as *nested* types. Six methods exist only on the non-`_ex` class —
`FindModel`, `DestroyFrame`, `VisibleAllFrames`, `IsBlockInRange`, `PrepareScene`,
`cbDupl` — and the PC has a counterpart for **all six**. That closed-set agreement settles the
identification.

### 7b. Twin table

| PC VA | PC name | PS2 twin | PS2 size | PC size | status |
|---|---|---|---|---|---|
| 0x005a7590 | `cache_base_block_ctor` | `__ct__18C_cache_base_blockFv` | 116 | 44 | STRUCTURAL |
| 0x005a75c0 | `cache_base_block_dtor` | `__dt__18C_cache_base_blockFv` | 140 | 331 | STRUCTURAL |
| 0x005a7710 | `cache_base_block_close` | `Close__18C_cache_base_blockFv` | 424 | 239 | CONFIRMED |
| 0x005a7800 | `cache_base_block_prepare_scene` | `PrepareScene__18C_cache_base_blockFv` | 64 | 39 | CONFIRMED |
| 0x005a7830 | `cache_base_block_prepare_scene_cb` | `cbDupl__Q218C_cache_base_block22S_hlp$2095_m_cache_cppFP9I3D_frameUi` | 372 | 217 | CONFIRMED |
| 0x005a7910 | `cache_base_block_open` | `Open__18C_cache_base_blockFP7C_chunkP9I3D_scene` | 5276 | 2155 | CONFIRMED |
| 0x005a81b0 | `cache_base_block_open_model` | `OpenModel__18C_cache_base_blockFPCc` | 1844 | 1002 | CONFIRMED |
| 0x005a85c0 | `cache_base_block_tick` | `Tick__18C_cache_base_blockFUi` | 664 | 1108 | CONFIRMED |
| 0x005a8a20 | `cache_base_block_activate_block` | `ActivateBlock__18C_cache_base_blockFUi` | 1256 | 1107 | CONFIRMED |
| 0x005a8e80 | `cache_base_block_deactivate_block` | `DeactivateBlock__18C_cache_base_blockFUi` | 364 | 263 | CONFIRMED |
| 0x005a8f90 | `cache_base_block_walk_block_objects` | - | - | 333 | PC-only ¹ |
| 0x005a90e0 | `cache_base_block_find_model` | `FindModel__18C_cache_base_blockFUsUs` | 80 | 53 | CONFIRMED |
| 0x005a9120 | `cache_base_block_destroy_frame` | `DestroyFrame__18C_cache_base_blockFP18S_cache_model_inbb` | 316 | 202 | CONFIRMED |
| 0x005a91f0 | `cache_base_block_deactivate_all_blocks` | `VisibleAllFrames__18C_cache_base_blockFv` | 848 | 455 | CONFIRMED |
| 0x005a93c0 | `cache_base_block_is_block_in_range` | `IsBlockInRange__18C_cache_base_blockFUi9S_vector2f` | 1320 | 859 | CONFIRMED |
| 0x005c8290 | `cache_base_block_is_loaded` | - | - | 5 | PC-only |
| - | - | `AssociateBaseBlocks__18C_cache_base_blockFv` | 420 | - | not located |
| - | - | `GetNumModelVisual__18C_cache_base_blockFP9I3D_modelPP10I3D_object` | 168 | - | not located |
| - | - | `cbVisCount__Q218C_cache_base_block22S_hlp$2923_m_cache_cppFP9I3D_framePiPP10I3D_object` | 276 | - | not located |
| - | - | `IsModelOnlyObject__18C_cache_base_blockFUs` | 36 | - | not located |
| - | - | `ShowObject__18C_cache_base_blockFUs` | 840 | - | not located |
| - | - | `HideObject__18C_cache_base_blockFP10I3D_object` | 48 | - | not located |
| - | - | `ShowModel__18C_cache_base_blockFUsPUs` | 1228 | - | not located |
| - | - | `HideModel__18C_cache_base_blockFUsUs` | 144 | - | not located |
| - | - | `ClearUnusedModels__18C_cache_base_blockFv` | 276 | - | not located |

¹ See §7d.

### 7c. Two names that diverge from the PS2 verb

* `VisibleAllFrames` → **`cache_base_block_deactivate_all_blocks`**. The body hides every
  active block, clears every active flag, returns every model to its pool and forces the
  re-test accumulator to 10000. It makes nothing visible; the next tick re-shows whatever
  is still in range.
* `cbDupl` → **`cache_base_block_prepare_scene_cb`**. "Dupl" describes the intent (fold
  duplicates onto one frame); the PC name says where it is used from.

### 7d. `cache_base_block_walk_block_objects` is not `GetNumModelVisual` / `cbVisCount`

Those two PS2 methods sit in the matching position but are a pure recursive *count* of visible
children, capturing the `'OBJ_'` one into an out-parameter. The PC function prunes small
subtrees and starts the 700 ms fade-in (§4). Only the `0x5f4a424f` magic is shared.

### 7e. Suspected PC inlinings

Nine PS2 methods have no PC function in the unit, and equivalent code is visible inside
`cache_base_block_activate_block` / `_deactivate_block` / `_open`. That is a **shape, not a
measurement** — none of the nine PS2 bodies has been read — so no inlining is marked in the
decomp (§9).

## 8. Cross-references

`structs/chunk.md` (the `C_chunk` the loader runs on) · `structs/particle.md` §7 (the same `I3D`
model cache, decoded from the other side) · `structs/mission.md` (`mission_fade_in_object`) ·
`basic/artifacts/dll.md` (`rw_data.dll`, the archive layer under `C_chunk`).

## 9. Open items

* **Nine PS2 methods not located** (§7b). Reading `ShowModel` (1228 B) and `ShowObject`
  (840 B) against `cache_base_block_activate_block`'s body would turn §7e into a measurement.
* **Does the PC ship a `C_cache_base_block_ex`?** Not searched.
* **`wcol` / `LLwcol`** — `cache_base_block_prepare_scene_cb` folds every such frame onto
  the first one and hides it. What those frames are *for* (walk collision?) is not decoded.
* **In game:** the `0rek23.i3d` → `0rek17.i3d` substitution should be observable — the
  model that `0rek23` names never appears anywhere in the city.
