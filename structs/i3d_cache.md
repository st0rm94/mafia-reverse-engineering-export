# `C_I3D_cache<T>` — the three engine resource caches

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-11-16 16:16 +0100

Three file-scope objects of one class template — the **model**, **sound** and **animation-set**
caches. Each is a name-keyed store of loaded engine resources with a capacity cap, so a `.i3d`
asked for twice is loaded once. The template is three levels deep (a base, the template proper and
a per-resource derived class, §5), and its element type `I3D_animation_set` is documented here too
(§1b).

| | |
|---|---|
| PC prefix | `i3d_cache_` (the template members, and the base level as `i3d_cache_base_`), `i3d_model_cache_` / `i3d_sound_cache_` / `i3d_anim_cache_` (the three derived classes), `i3d_cache_items_` (the item vector), `i3d_animation_set_` (the cached clip set) |
| PC kind (`+0x10`) | - |
| PC vtable | three per object, base / mid / derived (§5); the derived ones are `g_i3d_model_cache_vt`, `g_i3d_sound_cache_vt`, `g_i3d_anim_cache_vt` |
| PC base class | - |
| PC instance size | `0x28` (§1) |
| PC constructor | `i3d_model_cache_ctor` @ `0x005f7e70`, `i3d_sound_cache_ctor` @ `0x005f8020`, `i3d_anim_cache_ctor` @ `0x005f81d0` |
| PC translation unit | `[0x005fb6e0, 0x005fe4f0)`: the template members, emitted as COMDATs into the code half of `Main.cpp` (`structs/app_bootstrap.md`), whose initializer block also holds the three static quartets with their ctors and dtors; the item-vector helpers and two `Open` / `Create` copies sit in their callers' units (§3c) |
| PS2 class | `C_I3D_cache<T>` (tags `23C_I3D_cache<9I3D_model>`, `23C_I3D_cache<9I3D_sound>`, `32C_I3D_cache<17I3D_animation_set>`), its base `C_I3D_cache_base<T>` and the derived `C_I3D_model_cache`, `C_I3D_sound_cache`, `C_I3D_anim_cache`: 49 methods, all placed (§3, §7) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

All three objects share one layout.

```c
typedef struct C_I3D_cache {                 // 0x28 bytes
    /* +0x00 */ void  *vtable;               // [H] the derived table, §5
    /* +0x04 */ std::vector<S_itm> items;    // [H] {alloc, begin, end, cap}: the cached resources
    /* +0x14 */ std::vector<C_string> dirs;  // [H] {alloc, begin, end, cap}: the search directories
    /* +0x24 */ int32_t size_limit;          // [H] the capacity cap, 0x100 entries
} C_I3D_cache;
```

`items` holds one 8-byte `S_itm` per cached resource (§1a), which is what the destructor's
`(end − begin) >> 3` and its stride-8 teardown loop measure. `dirs` holds one `C_string` handle per
directory: the destructors and `i3d_cache_clear_dirs` release each entry's refcounted block,
`i3d_cache_get_dir` returns `block + 8`, the `*_add_dir` members insert through
`string_vector_insert_n` and the `*_del_dir` members compare by length and bytes. `engine_init`
seeds the anim cache's with the literal `"Anims"`. The cap is **256** (`size_limit = 0x100`,
written by each ctor and by `i3d_cache_set_size`): at the 257th distinct resource the cache evicts
an entry only it holds (§6a).

Ghidra names the fields of the three global objects as standalone globals
(`g_i3d_model_cache_items_begin` = `0x006f9420` = model cache `+0x08`, `…_items_end`,
`…_dirs_begin`, `…_dirs_end`, `…_size_limit`, and the same for `sound` and `anim`).

### 1a. `S_itm` — one cached resource, 8 bytes

```c
typedef struct S_itm {                       // 8 bytes; the PS2's name for the element
    /* +0x00 */ T       *object;             // [H] the cached resource
    /* +0x04 */ C_string name;               // [H] the key, a refcounted string handle
} S_itm;
```

`name` is a `C_string`, a one-pointer handle to a refcounted block (`structs/string.md`): the
item-vector helpers copy `object` plainly and bump or release the block's refcount at `+0x00`.

### 1b. `I3D_animation_set` — the cached clip set

The element type of the anim cache. Its body is inside `ls3df.dll`; its layout is decoded from
the PS2 class body (§7f), shifted by the PC's vtable pointer.

```c
typedef struct I3D_animation_set {
    /* +0x00 */ void    *vtable;             // [H] virtual 0 = Release, virtual 1 = the load
    /* +0x04 */ int32_t  refcount;           // [H] bumped on every cache hit
    /* +0x08 */ int32_t  default_options;    // [H] the I3D_ANIMOPTIONS a track starts with
    /* +0x0c */ void    *owned_array;        // [U] a delete[]-owned array; no Game.exe access, meaning in ls3df.dll
    /* +0x10 */ void    *anim_blob;          // [M] the loaded animation data; no Game.exe access
} I3D_animation_set;
```

**What `default_options == 2` means.** `== 2` loops the clip; anything else plays it once (§6b).

## 2. Field evidence

Measured by a census of every `this+N` access in the `i3d_cache_*`, `i3d_*_cache_*` and
`i3d_cache_items_*` functions, and by a writer / reader census of the fifteen Ghidra globals of
§1 across the decomp. Because the cache `Open` is inlined at most of its callsites, most external
functions reach the fields through those globals (footnotes).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | the three ctors (derived table); every dtor level (its own table on entry, the next one down on exit) | virtual dispatch | §5 |
| +0x04 | items | H | the ctors, the dtors, the `Open` / `Create` members, `i3d_cache_clear`, the `i3d_cache_items_*` helpers, and 11 external functions ¹ | the same members and 31 external functions ² | stride-8 `S_itm`; entry count `(end − begin) >> 3` |
| +0x14 | dirs | H | the ctors, the dtors, `i3d_cache_*_add_dir`, `i3d_cache_*_del_dir`, `i3d_cache_clear_dirs`, `engine_init` (the `"Anims"` seed) | the `Open` / `Create` members, `i3d_cache_*_del_dir`, `i3d_cache_clear_dirs`, `i3d_cache_num_dirs`, `i3d_cache_get_dir`, the three `*_resolve_name`, and 7 external functions ³ | freed before `items`, in reverse declaration order |
| +0x24 | size_limit | H | the three ctors (`0x100`), `i3d_cache_set_size` | the `Open` / `Create` members, and 28 external functions ⁴ | the eviction bound (§6a) |

¹ `bridge_game_init`, `engine_shutdown`, `game_init`, `human_do_throw_occupant_from_car`,
`loadsave_savegame_load`, `player_ai_walk`, `police_manager_apply_state`, `pumper_game_init`,
`racing_spawn_winners_scene`, `railway_game_init`, `traffic_car_game_init` (the `end` pointer: the
inlined erase step).
² The eleven of ¹ plus `cache_base_block_open_model`, `car_explosion`,
`car_load_particle_effects`, `crash_obj_create_crash_actors`, `dog_spawn_severed_head`,
`game_load_differences`, `game_new_grenade`, `human_drop_out_items`, `mission_close`,
`mission_open_bin`, `mission_uninit`, `opponent_game_init`,
`particle_manager_create_particle_from_type`, `particle_manager_create_static_particle`,
`traffic_car_activate_or_spawn_pursuit_vehicles`, `traffic_car_adopt_car_with_new_driver`,
`traffic_car_spawn_mafia_pursuit_car`, `traffic_car_take_over_cruising_car`,
`using_object_save_game_load`, `wagon_game_init`. `police_manager_apply_state` walks the model
cache because spawning a police car needs its **model**; the cache is not a spawn manager.
³ `bridge_game_init`, `engine_init`, `game_init`, `human_do_throw_occupant_from_car`,
`recorder_open_speech_sound_for_text_id`, `traffic_car_adopt_car_with_new_driver`,
`traffic_car_game_init`.
⁴ The functions of ² except `engine_shutdown`, `mission_close` and `mission_uninit`, which empty
the vector without the cap.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | object | H | `i3d_cache_items_copy_construct_one`, `i3d_cache_items_copy_assign_one`, `i3d_cache_items_insert_one`, the inlined `Open` add step | the `Open` / `Create` members (hit test), `i3d_cache_clear` (virtual release) | copied plainly |
| +0x04 | name | H | `i3d_cache_items_copy_construct_one` (addref), `i3d_cache_items_copy_assign_one`, `i3d_cache_items_copy_assign_range`, `i3d_cache_items_insert_one` | the `Open` / `Create` members (the wildcard match of §6a), `i3d_cache_items_destroy_one` / `_destroy_range` (release) | a refcounted `C_string` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `ls3df.dll` | the anim-cache release path, `(**(code **)*set)(set)` at `0x005bfd2a`; `i3d_cache_anim_create` (virtual 0 on a failed load and on eviction, virtual 1 the load), `i3d_cache_anim_open` (virtual 1 on the caller's set), `i3d_cache_clear` and the three anim destructors (virtual 0) | the PC class is polymorphic |
| +0x04 | refcount | H | `i3d_cache_anim_create` (`inc dword [set+4]` at `0x0047fb9a`, a cache hit; and + 1 on a newly loaded set) | `i3d_cache_anim_create` (eviction takes the first set whose count is 1, one the cache alone holds), `ls3df.dll` (`Release`) | §7f |
| +0x08 | default_options | H | `i3d_animation_set_set_default_options` | `ls3df.dll` (the track's options when none are given) | §6b |
| +0x0c | owned_array | U | - | - | no access in the 77 bodies of this doc; the `delete[]` ownership is §7f's; Open item 2 |
| +0x10 | anim_blob | M | `ls3df.dll` (the clip loader) | `ls3df.dll` | no access in the 77 bodies of this doc; §7f; Open item 2 |

## 3. Method family

Coverage: all **49** PS2 methods of the six cache classes are placed — **46** in the tables below
(42 twins, 4 base stubs; the five ICF-folded members carry three PS2 twins each) and the three
`ClearUpper` in the PS2-only table of §7. PC-only: the three `*_resolve_name` virtuals, the nine
scalar-deleting destructors, the three static quartets, the three constructors (inlined on the
PS2) and the item-vector helpers.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004800a0 | `i3d_cache_items_scalar_deleting_dtor` | an `S_itm` | - | - | - | destroys one item, then frees it when flag bit 0 is set; 2 callsites, `i3d_cache_anim_create` and `i3d_cache_anim_open` |
| 0x005f7d10 | `i3d_model_cache_static_init` | `g_i3d_model_cache` | - | - | - | `Main.cpp`'s `.CRT$XCU` entry |
| 0x005f7d20 | `i3d_model_cache_static_ctor` | `g_i3d_model_cache` | - | - | - | `mov ecx, 0x006f9418; jmp i3d_model_cache_ctor` |
| 0x005f7d30 | `i3d_model_cache_register_atexit_dtor` | `g_i3d_model_cache` | - | - | - | `atexit(&i3d_model_cache_static_dtor)` |
| 0x005f7d40 | `i3d_model_cache_static_dtor` | `g_i3d_model_cache` | - | - | - | `mov ecx, 0x006f9418; jmp i3d_model_cache_dtor` |
| 0x005f7d50 | `i3d_model_cache_dtor` | instance | `__dt__17C_I3D_model_cacheFv` | 0x00378430 | CONFIRMED | the DERIVED destructor (§5a) |
| 0x005f7e70 | `i3d_model_cache_ctor` | instance | - | - | - | zeroes both vectors, `size_limit` = `0x100`, the derived vtable; inlined on the PS2 (§7c) |
| 0x005f7eb0 | `i3d_model_cache_scalar_deleting_dtor` | instance | - | - | - | derived slot 5 |
| 0x005f7ed0 | `i3d_sound_cache_static_init` | `g_i3d_sound_cache` | - | - | - | as the model quartet |
| 0x005f7ee0 | `i3d_sound_cache_static_ctor` | `g_i3d_sound_cache` | - | - | - | `mov ecx, 0x006f93f0; jmp i3d_sound_cache_ctor` |
| 0x005f7ef0 | `i3d_sound_cache_register_atexit_dtor` | `g_i3d_sound_cache` | - | - | - | - |
| 0x005f7f00 | `i3d_sound_cache_static_dtor` | `g_i3d_sound_cache` | - | - | - | - |
| 0x005f7f10 | `i3d_sound_cache_dtor` | instance | `__dt__17C_I3D_sound_cacheFv` | 0x003781a0 | CONFIRMED | the derived destructor |
| 0x005f8020 | `i3d_sound_cache_ctor` | instance | - | - | - | as the model ctor |
| 0x005f8060 | `i3d_sound_cache_scalar_deleting_dtor` | instance | - | - | - | derived slot 5 |
| 0x005f8080 | `i3d_anim_cache_static_init` | `g_i3d_anim_cache` | - | - | - | as the model quartet |
| 0x005f8090 | `i3d_anim_cache_static_ctor` | `g_i3d_anim_cache` | - | - | - | `mov ecx, 0x006f93c8; jmp i3d_anim_cache_ctor` |
| 0x005f80a0 | `i3d_anim_cache_register_atexit_dtor` | `g_i3d_anim_cache` | - | - | - | - |
| 0x005f80b0 | `i3d_anim_cache_static_dtor` | `g_i3d_anim_cache` | - | - | - | - |
| 0x005f80c0 | `i3d_anim_cache_dtor` | instance | `__dt__16C_I3D_anim_cacheFv` | 0x00377fa0 | CONFIRMED | the derived destructor |
| 0x005f81d0 | `i3d_anim_cache_ctor` | instance | - | - | - | as the model ctor |
| 0x005f8210 | `i3d_anim_cache_scalar_deleting_dtor` | instance | - | - | - | derived slot 5 |
| 0x005fc260 | `i3d_cache_base_model_dtor` | instance | `__dt__28C_I3D_cache_base<9I3D_model>Fv` | 0x00378670 | CONFIRMED | the base destructor |
| 0x005fcd30 | `i3d_cache_base_sound_dtor` | instance | `__dt__28C_I3D_cache_base<9I3D_sound>Fv` | 0x003783e0 | CONFIRMED | the base destructor |
| 0x005fd970 | `i3d_cache_base_anim_dtor` | instance | `__dt__37C_I3D_cache_base<17I3D_animation_set>Fv` | 0x00378150 | CONFIRMED | the base destructor |
| 0x005fda30 | `i3d_cache_model_scalar_deleting_dtor` | instance | - | - | - | mid slot 5 |
| 0x005fda50 | `i3d_cache_model_dtor` | instance | `__dt__23C_I3D_cache<9I3D_model>Fv` | 0x00378590 | CONFIRMED | the MID destructor |
| 0x005fdb60 | `i3d_cache_base_model_scalar_deleting_dtor` | instance | - | - | - | base slot 5 |
| 0x005fdb80 | `i3d_cache_sound_scalar_deleting_dtor` | instance | - | - | - | mid slot 5 |
| 0x005fdba0 | `i3d_cache_sound_dtor` | instance | `__dt__23C_I3D_cache<9I3D_sound>Fv` | 0x00378300 | CONFIRMED | the mid destructor |
| 0x005fdcb0 | `i3d_cache_base_sound_scalar_deleting_dtor` | instance | - | - | - | base slot 5 |
| 0x005fdcd0 | `i3d_cache_anim_scalar_deleting_dtor` | instance | - | - | - | mid slot 5 |
| 0x005fdcf0 | `i3d_cache_anim_dtor` | instance | `__dt__32C_I3D_cache<17I3D_animation_set>Fv` | 0x00378070 | CONFIRMED | the mid destructor |
| 0x005fde00 | `i3d_cache_base_anim_scalar_deleting_dtor` | instance | - | - | - | base slot 5 |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8540 | `i3d_animation_set_set_default_options` | - | - | - | stores `+0x08` of an `I3D_animation_set`; a header inline out-lined into the `C_program` unit's tail, used by the `MODEL_PLAYANIM` arm with 2 (§6b) |
| 0x005fcd40 | `i3d_cache_set_size` | `SetCacheSize__23C_I3D_cache<9I3D_model>Fi` | 0x00378a60 | CONFIRMED | slot 0; stores `size_limit`; ICF-folded across the three caches |
| 0x005fcd40 | `i3d_cache_set_size` | `SetCacheSize__23C_I3D_cache<9I3D_sound>Fi` | 0x003789e0 | CONFIRMED | the same function, sound cache |
| 0x005fcd40 | `i3d_cache_set_size` | `SetCacheSize__32C_I3D_cache<17I3D_animation_set>Fi` | 0x00378960 | CONFIRMED | the same function, anim cache |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00448940 | `i3d_cache_model_open` | `Open__23C_I3D_cache<9I3D_model>FP9I3D_modelPCcUiPFPCcPv_vPvPv` | 0x002e8650 | CONFIRMED | slot 1; the model entry point, 27 callsites ⁵ |
| 0x00449410 | `i3d_cache_items_uninit_copy_range` | - | - | - | the item vector: copy-constructs a range into raw storage; 3 callsites, all in `i3d_cache_items_insert_one` |
| 0x00449930 | `i3d_cache_items_copy_construct_one` | - | - | - | the item vector: zeroes the name slot, copies `object`, addrefs the name; 4 callsites in these helpers |
| 0x00449b60 | `i3d_cache_items_copy_assign_one` | - | - | - | the item's `operator=`; 10 callsites, each the inlined erase in a cache `Open`, `Create` or destructor |
| 0x0044e440 | `i3d_cache_sound_open` | `Open__23C_I3D_cache<9I3D_sound>FP9I3D_soundPCcUiPFPCcPv_vPvPv` | 0x003792b0 | CONFIRMED | slot 1; the sound entry point, 5 callsites ⁵ |
| 0x0044e880 | `i3d_cache_items_destroy_one` | - | - | - | the item destructor, releases the name; 7 callsites in these helpers and `i3d_cache_clear`, and 60 unwind funclets of the functions that inline `Open` |
| 0x0044e8c0 | `i3d_cache_items_destroy_range` | - | - | - | releases the name of each item in a range; 37 callsites |
| 0x0044e8f0 | `i3d_cache_items_uninit_fill_n` | - | - | - | copy-constructs N copies of one item; 1 callsite, in `i3d_cache_items_insert_one` |
| 0x0044e920 | `i3d_cache_items_copy_assign_range` | - | - | - | copy-assigns a range, taking the source reference before releasing the destination's; 1 callsite, in `i3d_cache_items_insert_one` |
| 0x0047fa10 | `i3d_cache_anim_create` | `Create__32C_I3D_cache<17I3D_animation_set>FPCcPP17I3D_animation_setPFPCcPv_vPv` | 0x00306770 | CONFIRMED | slot 2; the anim entry point, 23 callsites ⁵ |
| 0x0047fe60 | `i3d_cache_items_insert_one` | - | - | - | inserts one item, growing by half the count when full; 60 callsites, each the add-item step of an inlined `Open` |
| 0x0047ffe0 | `i3d_cache_items_copy_range` | - | - | - | copy-assigns a range forward; 28 callsites, all cache code |
| 0x00480040 | `i3d_cache_items_copy_backward_range` | - | - | - | copy-assigns a range backward, safe for an overlapping shift; 1 callsite |
| 0x004d3330 | `i3d_cache_items_erase_at` | - | - | - | erases one item: shifts the tail down, addref-ing what moves and releasing what it overwrites, then destroys the last slot; 80 callsites, each the erase step of an inlined `Open`, every receiver one of the three item vectors |
| 0x00575c00 | `stub_noop_args8_00575c00` | `Duplicate__16C_I3D_anim_cacheFP17I3D_animation_setP17I3D_animation_set` | 0x00377d50 | base-stub | anim derived slot 12; the 8-byte PS2 body does nothing, and the slot holds the shared do-nothing stub |
| 0x005fb6e0 | `i3d_cache_model_create` | `Create__23C_I3D_cache<9I3D_model>FPCcPP9I3D_modelPFPCcPv_vPv` | 0x00378a70 | CONFIRMED | slot 2 |
| 0x005fbaf0 | `i3d_cache_model_add_dir` | `AddDir__23C_I3D_cache<9I3D_model>FPCc` | 0x003788a0 | CONFIRMED | slot 6 |
| 0x005fbd10 | `i3d_cache_model_del_dir` | `DelDir__23C_I3D_cache<9I3D_model>FPCc` | 0x00376d40 | CONFIRMED | slot 7 |
| 0x005fbf60 | `i3d_cache_model_resolve_name` | - | - | - | slot 11, PC-only: resolves a path against the search directories, stripping the current-directory prefix; never dispatched (§5) |
| 0x005fc270 | `i3d_cache_sound_create` | `Create__23C_I3D_cache<9I3D_sound>FPCcPP9I3D_soundPFPCcPv_vPv` | 0x00378fa0 | CONFIRMED | slot 2 |
| 0x005fc680 | `i3d_cache_sound_add_dir` | `AddDir__23C_I3D_cache<9I3D_sound>FPCc` | 0x00378750 | CONFIRMED | slot 6 |
| 0x005fc830 | `i3d_cache_sound_del_dir` | `DelDir__23C_I3D_cache<9I3D_sound>FPCc` | 0x00376c90 | CONFIRMED | slot 7 |
| 0x005fca40 | `i3d_cache_sound_resolve_name` | - | - | - | slot 11, PC-only, as the model one |
| 0x005fcd50 | `i3d_cache_anim_open` | `Open__32C_I3D_cache<17I3D_animation_set>FP17I3D_animation_setPCcUiPFPCcPv_vPvPv` | 0x00379820 | CONFIRMED | slot 1 |
| 0x005fd180 | `i3d_cache_clear` | `Clear__23C_I3D_cache<9I3D_model>Fv` | 0x00378500 | CONFIRMED | slot 4; releases every cached object backwards and empties `items`; ICF-folded across the three caches |
| 0x005fd180 | `i3d_cache_clear` | `Clear__23C_I3D_cache<9I3D_sound>Fv` | 0x00378270 | CONFIRMED | the same function, sound cache |
| 0x005fd180 | `i3d_cache_clear` | `Clear__32C_I3D_cache<17I3D_animation_set>Fv` | 0x00377e40 | CONFIRMED | the same function, anim cache; the PS2 copy calls `Release` directly (§7f) |
| 0x005fd210 | `i3d_cache_anim_add_dir` | `AddDir__32C_I3D_cache<17I3D_animation_set>FPCc` | 0x00377d80 | CONFIRMED | slot 6 |
| 0x005fd3c0 | `i3d_cache_anim_del_dir` | `DelDir__32C_I3D_cache<17I3D_animation_set>FPCc` | 0x00376be0 | CONFIRMED | slot 7 |
| 0x005fd5d0 | `i3d_cache_clear_dirs` | `ClearDirs__23C_I3D_cache<9I3D_model>Fv` | 0x00378d90 | CONFIRMED | slot 8; ICF-folded across the three caches |
| 0x005fd5d0 | `i3d_cache_clear_dirs` | `ClearDirs__23C_I3D_cache<9I3D_sound>Fv` | 0x003789f0 | CONFIRMED | the same function, sound cache |
| 0x005fd5d0 | `i3d_cache_clear_dirs` | `ClearDirs__32C_I3D_cache<17I3D_animation_set>Fv` | 0x00378970 | CONFIRMED | the same function, anim cache |
| 0x005fd640 | `i3d_cache_num_dirs` | `NumDirs__23C_I3D_cache<9I3D_model>CFv` | 0x00378df0 | CONFIRMED | slot 9; `(end − begin) >> 2` of `dirs`; ICF-folded across the three caches |
| 0x005fd640 | `i3d_cache_num_dirs` | `NumDirs__23C_I3D_cache<9I3D_sound>CFv` | 0x00378a50 | CONFIRMED | the same function, sound cache |
| 0x005fd640 | `i3d_cache_num_dirs` | `NumDirs__32C_I3D_cache<17I3D_animation_set>CFv` | 0x003789d0 | CONFIRMED | the same function, anim cache |
| 0x005fd660 | `i3d_cache_get_dir` | `GetDir__23C_I3D_cache<9I3D_model>CFi` | 0x00378e00 | CONFIRMED | slot 10; directory *i*; ICF-folded across the three caches |
| 0x005fd660 | `i3d_cache_get_dir` | `GetDir__23C_I3D_cache<9I3D_sound>CFi` | 0x00378ed0 | CONFIRMED | the same function, sound cache |
| 0x005fd660 | `i3d_cache_get_dir` | `GetDir__32C_I3D_cache<17I3D_animation_set>CFi` | 0x00379750 | CONFIRMED | the same function, anim cache |
| 0x005fd680 | `i3d_cache_anim_resolve_name` | - | - | - | slot 11, PC-only, as the model one |
| 0x005fd980 | `i3d_model_cache_duplicate` | `Duplicate__17C_I3D_model_cacheFP9I3D_modelP9I3D_model` | 0x00378810 | CONFIRMED | derived slot 12 |
| 0x005fd9b0 | `i3d_model_cache_cb_null_data` | `cbNullData__17C_I3D_model_cacheFP9I3D_frameUi` | 0x00378870 | CONFIRMED | the callback slot 12 passes |
| 0x005fd9c0 | `i3d_model_cache_create_new` | `CreateNew__17C_I3D_model_cacheCFv` | 0x00378880 | CONFIRMED | derived slot 3 |
| 0x005fd9d0 | `i3d_sound_cache_duplicate` | `Duplicate__17C_I3D_sound_cacheFP9I3D_soundP9I3D_sound` | 0x003786c0 | CONFIRMED | derived slot 12 |
| 0x005fda10 | `i3d_sound_cache_create_new` | `CreateNew__17C_I3D_sound_cacheCFv` | 0x00378730 | CONFIRMED | derived slot 3 |
| 0x005fda20 | `i3d_anim_cache_create_new` | `CreateNew__16C_I3D_anim_cacheCFv` | 0x00377d60 | CONFIRMED | derived slot 3 |
| 0x005fde80 | `i3d_cache_items_vector_destroy` | - | - | - | the item vector's deleter |
| 0x00606690 | `i3d_cache_items_clear` | - | - | - | releases each item's name, then resets `end`; 3 callsites, all in `mission_close` |
| 0x00607670 | `i3d_cache_items_erase_range` | - | - | - | shifts the stride-8 tail down over an erased range; 3 callsites, all in `mission_uninit` |
| 0x006262a3 | `crt_purecall` | `CreateNew__28C_I3D_cache_base<9I3D_model>CFv` | 0x0010f510 | base-stub | base slot 3; MSVC's `_purecall`, where the PS2 base has an 8-byte stub |
| 0x006262a3 | `crt_purecall` | `CreateNew__28C_I3D_cache_base<9I3D_sound>CFv` | 0x00116980 | base-stub | the same, sound base |
| 0x006262a3 | `crt_purecall` | `CreateNew__37C_I3D_cache_base<17I3D_animation_set>CFv` | 0x00120b90 | base-stub | the same, anim base |

⁵ **Every receiver is byte-proven.** A census of the instructions that load ECX before each `E8`
call to the three entry points finds 27/27, 5/5 and 23/23 callsites loading the object address
as a literal immediate (`mov ecx, 0x6f9418` and the other two), none through an unprovable path.
The receivers of `i3d_cache_items_clear`, `i3d_cache_items_erase_range` and
`i3d_cache_items_erase_at` are byte-proven to be the three item vectors at `0x006f93cc` /
`0x006f93f4` / `0x006f941c`.

### 3d. Free helpers

The class has no free helper; the item-vector helpers of §3c are member functions of
`std::vector<S_itm>`, each called only from cache code (the members, the `Open` inlined at its
callsites and their unwind funclets, `engine_shutdown`, `mission_close`, `mission_uninit`).

## 4. The three instances

| PC symbol | Address | What it caches | Its entry point |
|---|---|---|---|
| `g_i3d_anim_cache` | `0x006f93c8` | `I3D_animation_set`, clips out of `Anims\` | `i3d_cache_anim_create`, 23 callsites |
| `g_i3d_sound_cache` | `0x006f93f0` | `I3D_sound` | `i3d_cache_sound_open`, 5 callsites |
| `g_i3d_model_cache` | `0x006f9418` | `I3D_model`, `.i3d` meshes | `i3d_cache_model_open`, 27 callsites |

The three sit **consecutively at a `0x28` stride** and are constructed in the order anim, sound,
model (the reverse of address order) by `Main.cpp`'s `.CRT$XCU` block.

## 5. Three vtables per object

Each object owns a contiguous **`0x98`-byte `.rdata` group of three tables**, 12 + 13 + 13 slots:

| Object | Base table (12) | Mid table (13) | Derived table (13) |
|---|---|---|---|
| model | `0x0063ebd8` | `0x0063ec08` | `0x0063ec3c` |
| sound | `0x0063ec70` | `0x0063eca0` | `0x0063ecd4` |
| anim | `0x0063ed08` | `0x0063ed38` | `0x0063ed6c` |

The constructor installs the **derived** table; the destructors install the **mid** table on
entry and the **base** table on exit. That is a three-level hierarchy whose derived level adds no
members.

| Slot | Role | Model | Sound | Anim |
|---|---|---|---|---|
| 0 | `SetCacheSize` | `i3d_cache_set_size` | `i3d_cache_set_size` | `i3d_cache_set_size` |
| 1 | `Open` | `i3d_cache_model_open` | `i3d_cache_sound_open` | `i3d_cache_anim_open` |
| 2 | `Create` | `i3d_cache_model_create` | `i3d_cache_sound_create` | `i3d_cache_anim_create` |
| 3 | `CreateNew` | `i3d_model_cache_create_new` | `i3d_sound_cache_create_new` | `i3d_anim_cache_create_new` |
| 4 | `Clear` | `i3d_cache_clear` | `i3d_cache_clear` | `i3d_cache_clear` |
| 5 | scalar-deleting dtor | per level (§3a) | per level | per level |
| 6 | `AddDir` | `i3d_cache_model_add_dir` | `i3d_cache_sound_add_dir` | `i3d_cache_anim_add_dir` |
| 7 | `DelDir` | `i3d_cache_model_del_dir` | `i3d_cache_sound_del_dir` | `i3d_cache_anim_del_dir` |
| 8 | `ClearDirs` | `i3d_cache_clear_dirs` | `i3d_cache_clear_dirs` | `i3d_cache_clear_dirs` |
| 9 | `NumDirs` | `i3d_cache_num_dirs` | `i3d_cache_num_dirs` | `i3d_cache_num_dirs` |
| 10 | `GetDir` | `i3d_cache_get_dir` | `i3d_cache_get_dir` | `i3d_cache_get_dir` |
| 11 | resolve-name (PC-only) | `i3d_cache_model_resolve_name` | `i3d_cache_sound_resolve_name` | `i3d_cache_anim_resolve_name` |
| 12 | `Duplicate` | `i3d_model_cache_duplicate` | `i3d_sound_cache_duplicate` | `stub_noop_args8_00575c00` |

Read from the nine tables' pointer arrays in `Game.exe`'s `.rdata`. **Five slots hold ONE symbol
for all three instantiations** — 0, 4, 8, 9 and 10. Their bodies never mention `T`, so ICF folded
the three instances onto one address; each of the five appears in **6** vtable slots (mid +
derived × three caches), which is why they carry the template-level prefix `i3d_cache_`. The
base table's slot 3 and every slot except 5 hold `crt_purecall` (`0x006262a3`), MSVC's
`_purecall`.

**Slot 11 is never dispatched.** No `E8` or `E9` referrer reaches any of the three bodies, their
only references are their mid and derived vtable slots, and every one of the 210 instructions that
reference a cache object loads ECX for a call through another slot, so the object address never
reaches memory where a later dispatch could pick it up.

### 5a. Three destructors per cache

`i3d_model_cache_dtor` (`0x005f7d50`) is the DERIVED one, reached from derived slot 5 and from the
global's exit destructor; `i3d_cache_model_dtor` (`0x005fda50`) is the MID one, reached from mid
slot 5; `i3d_cache_base_model_dtor` (`0x005fc260`) is the base one. Each installs the next table
down on the way out. The three derived destructors run: install the own table → `Clear` → destroy
`dirs` → destroy `items` → install the base table. `Clear` is inlined at each of the three sites
(the backwards walk over `items` calling each object's virtual destructor, plus the
`i3d_cache_items_destroy_range` that follows); each site carries an inline banner in the decomp.
Because the derived level adds no members, the derived destructor reached from derived slot 5 is
a statement-identical copy of the mid one.

## 6. Name matching and the cached clip set

### 6a. Name matching

`S_itm.name` is a **refcounted string**, compared with a `*` / `?` wildcard matcher
(`ascii_toupper`, case-insensitive), so the PC can resolve a cached resource by PATTERN.

**Eviction.** When the entry count reaches `size_limit`, `Open` and `Create` release and erase the
**first** entry whose object's refcount (`+0x04`) is 1, one only the cache holds. When no entry
qualifies, the new entry is inserted anyway, so the count can pass the cap.

### 6b. `default_options == 2` loops the clip

`== 2` loops the clip; anything else plays it once (§7f). The default of 1 is therefore play-once,
and `MODEL_PLAYANIM` overriding it to 2 through `i3d_animation_set_set_default_options` is what
makes a scripted model animation loop until `MODEL_STOPANIM`, which matches every shipped use
(mission `EXTREME`, script `detector_ufo01`: the hovering UFO; mission `EXTREME`, script
`godzila`: the walk cycle). The field is on the **shared, cached** clip set, not on the track, so
writing it changes the default for every later user of the same clip; nothing in the PC build
writes it back.

## 7. PC vs PS2

The class is PS2 `C_I3D_cache<T>`, deriving from `C_I3D_cache_base<T>`, with a per-resource
derived class on top:

| PC object | PS2 mid class | PS2 derived class | PS2 object |
|---|---|---|---|
| `g_i3d_model_cache` | `C_I3D_cache<9I3D_model>` | `C_I3D_model_cache` | `0x0050a880` |
| `g_i3d_sound_cache` | `C_I3D_cache<9I3D_sound>` | `C_I3D_sound_cache` | `0x0050a8c0` |
| `g_i3d_anim_cache` | `C_I3D_cache<17I3D_animation_set>` | `C_I3D_anim_cache` | `0x0050a900` |

The three-level hierarchy is the PS2's, and it explains the PC's three-table `.rdata` group (§5).
The PS2's nine tables were read out of the `SLUS_206.71` ELF `.symtab`.

### 7a. The destructor diff

All three PC destructors and all three PS2 destructors run the identical sequence (§5a). Member
order is reverse-declaration on both sides. The PS2 dispatches `Clear` through `*vt + 0x18`
(slot 6); MSVC inlined it, one PC inlining three times. Each PS2 derived destructor stores its own
vtable, a dead store, and then runs the template destructor inline; the PC copy is that sequence
without the dead store. The PC emits **three** destructors and three scalar-deleting destructors
per cache; the PS2 merges each `__dt__` with its delete flag, so every PC pair is a **2 → 1 fold**
in the PS2 direction (which is why the base-destructor size ratio is 0.09).

### 7b. Build differences

* **Cache size.** The PS2 writes `0x400` (1024 entries) into `+0x1c` of its own layout; the PC
  writes `0x100` (256) into `+0x24`. The PC cap is four times smaller.
* **A trailing field.** The PS2 object carries a `-1` word after the cap that the PC constructor
  never writes.
* **Name matching.** The PS2 uses a plain `strcasecmp`; the PC's wildcard matcher (§6a) can
  resolve by pattern and the PS2's cannot.

### 7c. Why the constructors have no twin

There is no PS2 out-of-line constructor for any of the three templates: the entire construction is
spelled inline inside `__sinit_Main.cpp`. That fragment was compared field for field and agrees,
but a row whose other side is not a function carries no twin. The static quartets are MSVC
`??__E` / `??__F` scaffolding with no Metrowerks counterpart, and the scalar-deleting destructors
an MSVC shape.

### 7d. The vtables, and the one declaration difference

Both builds declare **13** virtuals and the tables agree slot for slot, with exactly two
differences:

| | PS2 | PC |
|---|---|---|
| between `Clear` and the destructor | `ClearUpper` | - |
| between `GetDir` and `Duplicate` | - | the resolve-name virtual, slot 11 |

So `Clear` is PS2 slot 6 and PC slot 4. The same one-slot shift appears in all three DERIVED
tables **and** all three BASE tables — the PS2 base table has exactly two live slots (`CreateNew`
at index 3 and one merged destructor at index 6), and the PC's has `_purecall` in exactly the
slots the PS2's base table leaves NULL. Three independent tables agreeing makes this a
measurement. Slot 11 is PC-only: the PS2 has no current working directory and loads through
`RW_data` archives at set paths.

**PS2 methods with no PC function:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00378e20 | `ClearUpper__23C_I3D_cache<9I3D_model>Fv` | its work, eviction at `size_limit`, is inlined into `Create` / `Open` on both builds (the PC rule is §6a); the PS2 copy is unreachable: the three cache objects are the only instances, every use of their addresses is a direct method call, a dispatch through slot `+0x14`, `+0x28` or `+0x38`, or the exit-destructor registration, and none dispatches slot `+0x1c` |
| 0x00378ef0 | `ClearUpper__23C_I3D_cache<9I3D_sound>Fv` | the same |
| 0x00379770 | `ClearUpper__32C_I3D_cache<17I3D_animation_set>Fv` | the same |

### 7e. The folded members

The PS2 emits a separate body per instantiation for `SetCacheSize`, `Clear`, `ClearDirs`,
`NumDirs` and `GetDir`; the three bodies of each are statement-identical, except that the
animation `Clear` calls `Release__17I3D_animation_setFv` directly where the model and sound ones
dispatch through the element's vtable (`I3D_animation_set` is non-polymorphic on the PS2, §7f). On
the PC the element is polymorphic, the three bodies are byte-identical, and `/OPT:ICF` merged them.

### 7f. `I3D_animation_set`

Every PC offset of §1b is the PS2 offset + 4: the PC class is polymorphic, while the PS2 class has
no vtable symbol and calls `Release` directly. The PS2 class body supplies the field evidence:

* **refcount** (PS2 `+0x00`): the PC cache-hit `inc dword [set+4]` at `0x0047fb9a` sits where the
  PS2 twin does `++*(int *)set`; the PS2 ctor seeds it to 1 and `Release__17I3D_animation_setFv`
  decrements and deletes at 0.
* **default options** (PS2 `+0x04`): `SetAnimation__11I3D_IPolSet...` reads it as
  `if (options == 0) options = set->[+4];`. The PS2 ctor seeds it to **1**.
* **owned array** (PS2 `+0x08`): freed with `delete[]` by `Close__17I3D_animation_setFv` when it
  is set; `LoadAnimation__FPCcUiP17I3D_animation_set` does not write it.
* **animation blob** (PS2 `+0x0c`): `LoadAnimation__FPCcUiP17I3D_animation_set` fills it through
  `LoadAnimation5ds`; `NumAnimations` reads its first `u16`, `GetEndTime` its second `* 0x28`,
  `GetAnims` skips 4 bytes, `GetAnimIndex` walks its name table; `Close` frees it.
* **What option 2 means**: `Tick__11I3D_IPolSetFi` decodes it. When a track's clock passes the
  clip's end time the options word stored for that track is tested: `== 2` wraps the clock by
  ±`GetEndTime`, i.e. loops; anything else clears the track's playing flag, i.e. stops.

## 8. Related

- `structs/app_bootstrap.md` — `Main.cpp`, the host unit, and its initializer block.
- `structs/string.md` — the refcounted `C_string` of `S_itm.name`.
- `structs/mission.md` — `mission_close` / `mission_uninit`, the item-vector users at scene change.
- `structs/i3d_frame.md` — the `I3D_model` frames the model cache returns.
- `structs/program.md` — the unit that hosts `i3d_animation_set_set_default_options`.

## 9. Open items

1. **In game: the cap.** Asking for 257 distinct `.i3d` models should evict, because the cap is
   256 on the PC (1024 on the PS2), a PC-only pressure point.
2. **`I3D_animation_set+0x0c` and `+0x10`.** No `Game.exe` code reads or writes either field; both
   are written and read inside `ls3df.dll` (§7f), so their content is outside this binary.
