# `C_I3D_cache<T>` — the three engine resource caches

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-10-04 17:00 +0200

Three file-scope objects of one class template, constructed by the `Main.cpp` initializer block
`[0x005f79a0, 0x005f8370]` — the **model**, **sound** and **animation-set** caches. Each is a
name-keyed store of already-loaded engine resources with a capacity cap, so a `.i3d` asked for
twice is loaded once.

Everything below is a **PC** fact unless it sits in §6, the one dedicated PS2 section.

---

## 1. The three instances

| PC symbol | address | what it caches | its `Open`/`Create` entry point |
|---|---|---|---|
| `g_i3d_anim_cache` | `0x006f93c8` | `I3D_animation_set` — clips out of `Anims\` | `i3d_cache_anim_create` (`0x0047fa10`), 23 callsites |
| `g_i3d_sound_cache` | `0x006f93f0` | `I3D_sound` | `i3d_cache_sound_open` (`0x0044e440`), 5 callsites |
| `g_i3d_model_cache` | `0x006f9418` | `I3D_model` — `.i3d` meshes | `i3d_cache_model_open` (`0x00448940`), 27 callsites |

The three sit **consecutively at a `0x28` stride** and are constructed in that order (anim,
sound, model — reverse of address order) by `Main.cpp`'s `.CRT$XCU` block.

**Every receiver is byte-proven.** A census of the instructions that load ECX before each `E8`
call to the three entry points finds **27/27**, **5/5** and **23/23** callsites loading the object
address as a literal immediate (`mov ecx, 0x6f9418` and friends), none through an unprovable path.

`g_i3d_model_cache` is **not a spawn manager** and there is no entity pool: `vt[+0xc]` and
`vt[+0x30]` are cache virtuals (slot 3 `CreateNew` and slot 12 `Duplicate` of `0x0063ec3c`).
`police_manager_apply_state` walks the cache because spawning a police car needs its **model**.

---

## 2. Field layout — 0x28 bytes, identical for all three

```c
typedef struct {                     /* sizeof = 0x28 */
/* +0x00 */ void      *vptr;         // [H] the DERIVED class vtable (see §3)
/* +0x04 */ char       item_alloc;   // [H] MSVC std::allocator empty sub-object
/* +0x08 */ S_itm     *items_begin;  // [H] g_i3d_model_cache_items_begin
/* +0x0c */ S_itm     *items_end;    // [H] g_i3d_model_cache_items_end
/* +0x10 */ S_itm     *items_cap;    // [H] capacity end
/* +0x14 */ char       dir_alloc;    // [H] second std::allocator sub-object
/* +0x18 */ char     **dirs_begin;   // [H] the search-directory string vector
/* +0x1c */ char     **dirs_end;     // [H]
/* +0x20 */ char     **dirs_cap;     // [H]
/* +0x24 */ int        size_limit;   // [H] g_i3d_model_cache_size_limit, = 0x100
} C_I3D_cache;
```

`S_itm` is an **8-byte** record — `{T *object, C_string name}` — which is what the destructor's
`(end - begin) >> 3` and its stride-8 teardown loop measure. The name is a `C_string`, a
one-pointer handle to a refcounted block (`structs/string.md`): the shared stride-8 helpers
`std_vector8_insert_one`, `array_copy_refcounted8`, `array_copy_backward_refcounted8`,
`copy_elem_stride_8_with_string` and `refcounted8_scalar_deleting_dtor` copy the object pointer
plainly and bump or release the block's refcount at `+0x00`. The directory vector is a plain
`vector<char*>`; the anim cache's is seeded with the literal `"Anims"`.

### 2a. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x00` | vptr | H | `i3d_model_cache_ctor`, `i3d_model_cache_dtor` | virtual dispatch | ctor installs the derived table, dtor installs the mid then the base one |
| `+0x08` | `items_begin` | H | `i3d_model_cache_ctor`, `i3d_model_cache_dtor` | `bridge_game_init`, `police_manager_apply_state` | stride-8 `{T*, name}` |
| `+0x0c` | `items_end` | H | `i3d_model_cache_ctor`, `i3d_model_cache_dtor` | same | entry count is `(end - begin) >> 3` |
| `+0x18` | `dirs_begin` | H | `i3d_model_cache_ctor`, `i3d_model_cache_dtor` | - | freed BEFORE the item vector, i.e. reverse declaration order |
| `+0x24` | `size_limit` | H | `i3d_model_cache_ctor` | `police_manager_apply_state` | `0x100` = 256 entries |

---

## 3. Three vtables per object, and what that means

Each object owns a contiguous **`0x98`-byte `.rdata` group of three tables** — 12 + 13 + 13
slots:

| object | base table (12) | mid table (13) | derived table (13) |
|---|---|---|---|
| model | `0x0063ebd8` | `0x0063ec08` | `0x0063ec3c` |
| sound | `0x0063ec70` | `0x0063eca0` | `0x0063ecd4` |
| anim | `0x0063ed08` | `0x0063ed38` | `0x0063ed6c` |

The constructor installs the **derived** table; the destructors install the **mid** table on
entry and the **base** table on exit. That is a three-level hierarchy whose derived level adds no
members, so the derived destructor reached from derived slot 5 is a statement-identical copy of
the mid one (§4a).

### 3a. The complete slot map — all 13 slots, all three levels

Read from the nine tables' pointer arrays in `Game.exe`'s `.rdata`.

| slot | role | model | sound | anim |
|---|---|---|---|---|
| 0 | `SetCacheSize` | `i3d_cache_set_size` | *(same symbol)* | *(same symbol)* |
| 1 | `Open` | `i3d_cache_model_open` | `i3d_cache_sound_open` | `i3d_cache_anim_open` |
| 2 | `Create` | `i3d_cache_model_create` | `i3d_cache_sound_create` | `i3d_cache_anim_create` |
| 3 | `CreateNew` | `i3d_model_cache_create_new` | `i3d_sound_cache_create_new` | `i3d_anim_cache_create_new` |
| 4 | `Clear` | `i3d_cache_clear` | *(same symbol)* | *(same symbol)* |
| 5 | scalar deleting dtor | per level, see §4 | per level | per level |
| 6 | `AddDir` | `i3d_cache_model_add_dir` | `i3d_cache_sound_add_dir` | `i3d_cache_anim_add_dir` |
| 7 | `DelDir` | `i3d_cache_model_del_dir` | `i3d_cache_sound_del_dir` | `i3d_cache_anim_del_dir` |
| 8 | `ClearDirs` | `i3d_cache_clear_dirs` | *(same symbol)* | *(same symbol)* |
| 9 | `NumDirs` | `i3d_cache_num_dirs` | *(same symbol)* | *(same symbol)* |
| 10 | `GetDir` | `i3d_cache_get_dir` | *(same symbol)* | *(same symbol)* |
| 11 | resolve-name **(PC-only)** ¹ | `i3d_cache_model_resolve_name` | `i3d_cache_sound_resolve_name` | `i3d_cache_anim_resolve_name` |
| 12 | `Duplicate` | `i3d_model_cache_duplicate` | `i3d_sound_cache_duplicate` | `stub_noop_args8_00575c00` |

¹ Resolves a path against the registered search directories, stripping the current-directory
prefix. Zero `E8` callers.

**Five slots hold ONE symbol for all three instantiations** — 0, 4, 8, 9 and 10. Their
bodies never mention `T`, so ICF folded the three instances onto one address; each of
those five appears in **6** vtable slots (mid + derived × three caches). That is why they
take the template-level prefix `i3d_cache_*` and not a per-resource one.

The base table's slot 3 and every slot except 5 hold `crt_purecall` (`0x006262a3`), MSVC's
`_purecall`.

---

## 4. PC functions

| PC VA | PC name | role |
|---|---|---|
| `0x005f7d10` | `i3d_model_cache_static_init` | `.CRT$XCU` entry |
| `0x005f7d20` | `i3d_model_cache_static_ctor` | `mov ecx, 0x6f9418; jmp ctor` |
| `0x005f7d30` | `i3d_model_cache_register_atexit_dtor` | `atexit(&…_static_dtor)` |
| `0x005f7d40` | `i3d_model_cache_static_dtor` | `mov ecx, 0x6f9418; jmp dtor` |
| `0x005f7d50` | `i3d_model_cache_dtor` | the class destructor |
| `0x005f7e70` | `i3d_model_cache_ctor` | the class constructor |
| `0x005f7eb0` | `i3d_model_cache_scalar_deleting_dtor` | derived slot 5 |
| `0x005f7ed0`–`0x005f8060` | `i3d_sound_cache_*` | the same seven, one per stub |
| `0x005f8080`–`0x005f8210` | `i3d_anim_cache_*` | the same seven |

### 4a. The template members — `[0x005fb6e0, 0x005fe4f0)`

The 13 slots of each class, emitted as COMDATs into `Main.cpp`'s code half.

| PC VA | PC name | slot | level |
|---|---|---|---|
| `0x005fb6e0` | `i3d_cache_model_create` | 2 | mid |
| `0x005fbaf0` | `i3d_cache_model_add_dir` | 6 | mid |
| `0x005fbd10` | `i3d_cache_model_del_dir` | 7 | mid |
| `0x005fbf60` | `i3d_cache_model_resolve_name` | 11 | mid |
| `0x005fc260` | `i3d_cache_base_model_dtor` | - | base |
| `0x005fc270` | `i3d_cache_sound_create` | 2 | mid |
| `0x005fc680` | `i3d_cache_sound_add_dir` | 6 | mid |
| `0x005fc830` | `i3d_cache_sound_del_dir` | 7 | mid |
| `0x005fca40` | `i3d_cache_sound_resolve_name` | 11 | mid |
| `0x005fcd30` | `i3d_cache_base_sound_dtor` | - | base |
| `0x005fcd40` | `i3d_cache_set_size` | 0 | mid, folded ×3 |
| `0x005fcd50` | `i3d_cache_anim_open` | 1 | mid |
| `0x005fd180` | `i3d_cache_clear` | 4 | mid, folded ×3 |
| `0x005fd210` | `i3d_cache_anim_add_dir` | 6 | mid |
| `0x005fd3c0` | `i3d_cache_anim_del_dir` | 7 | mid |
| `0x005fd5d0` | `i3d_cache_clear_dirs` | 8 | mid, folded ×3 |
| `0x005fd640` | `i3d_cache_num_dirs` | 9 | mid, folded ×3 |
| `0x005fd660` | `i3d_cache_get_dir` | 10 | mid, folded ×3 |
| `0x005fd680` | `i3d_cache_anim_resolve_name` | 11 | mid |
| `0x005fd970` | `i3d_cache_base_anim_dtor` | - | base |
| `0x005fd980` | `i3d_model_cache_duplicate` | 12 | derived |
| `0x005fd9b0` | `i3d_model_cache_cb_null_data` | - | derived, the callback slot 12 passes |
| `0x005fd9c0` | `i3d_model_cache_create_new` | 3 | derived |
| `0x005fd9d0` | `i3d_sound_cache_duplicate` | 12 | derived |
| `0x005fda10` | `i3d_sound_cache_create_new` | 3 | derived |
| `0x005fda20` | `i3d_anim_cache_create_new` | 3 | derived |
| `0x005fda30` | `i3d_cache_model_scalar_deleting_dtor` | 5 | mid |
| `0x005fda50` | `i3d_cache_model_dtor` | - | mid |
| `0x005fdb60` | `i3d_cache_base_model_scalar_deleting_dtor` | 5 | base |
| `0x005fdb80` | `i3d_cache_sound_scalar_deleting_dtor` | 5 | mid |
| `0x005fdba0` | `i3d_cache_sound_dtor` | - | mid |
| `0x005fdcb0` | `i3d_cache_base_sound_scalar_deleting_dtor` | 5 | base |
| `0x005fdcd0` | `i3d_cache_anim_scalar_deleting_dtor` | 5 | mid |
| `0x005fdcf0` | `i3d_cache_anim_dtor` | - | mid |
| `0x005fde00` | `i3d_cache_base_anim_scalar_deleting_dtor` | 5 | base |
| `0x005fde80` | `i3d_cache_items_vector_destroy` | - | the item vector's deleter |

⚠️ **Three destructors per cache, not one.** `i3d_model_cache_dtor` (`0x005f7d50`) is the
DERIVED one, reached from derived slot 5; `i3d_cache_model_dtor` (`0x005fda50`) is the MID
one, reached from mid slot 5; `i3d_cache_base_model_dtor` (`0x005fc260`) is the base one.
All three install the next vtable down on the way out.

All three derived destructors run: install own vtable → `Clear` → destroy the directory vector →
destroy the item vector → install the base vtable. `Clear` is inlined at each of the three sites
(the backwards walk over the item vector calling each object's virtual destructor, plus the
`destroy_range_stride8` that follows); each site carries an inline banner in the decomp.

### 4b. The item-vector helpers in `Mission.cpp`

Both live in the `Mission.cpp` helper tail, and all their callsites' receivers are byte-proven
to be the three cache objects' item vectors at `0x006f93cc` / `0x006f93f4` / `0x006f941c`.

| PC VA | PC name | role |
|---|---|---|
| `0x00606690` | `i3d_cache_items_clear` | release each item's refcounted name, then reset the end pointer. 3 callsites, all in `mission_close` |
| `0x00607670` | `i3d_cache_items_erase_range` | shift the stride-8 tail down over an erased range, addref-ing what moves and releasing what it overwrites. 3 callsites, all in `mission_uninit` |

---

## 5. Name matching

`S_itm.name` is a **refcounted string**, compared with a `*` / `?` wildcard matcher
(`ascii_toupper`, case-insensitive), so the PC can resolve a cached resource by PATTERN.

The cap is **256** entries (`size_limit`); a 257th distinct resource evicts.

---

## 6. PC vs PS2

The class is PS2 `C_I3D_cache<T>`, deriving from `C_I3D_cache_base<T>`, with a
per-resource derived class on top:

| PC object | PS2 mid class | PS2 derived class | PS2 address |
|---|---|---|---|
| `g_i3d_model_cache` | `C_I3D_cache<9I3D_model>` | `C_I3D_model_cache` | `0x0050a880` |
| `g_i3d_sound_cache` | `C_I3D_cache<9I3D_sound>` | `C_I3D_sound_cache` | `0x0050a8c0` |
| `g_i3d_anim_cache` | `C_I3D_cache<17I3D_animation_set>` | `C_I3D_anim_cache` | `0x0050a900` |

The three-level hierarchy is the PS2's, and it explains the PC's three-table `.rdata` group (§3).
The PS2's nine tables were read out of the `SLUS_206.71` ELF `.symtab`.

**Twin table** (PC VA order):

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| `0x005f7d50` | `i3d_model_cache_dtor` | `__dt__17C_I3D_model_cacheFv` ¹ | CONFIRMED |
| `0x005f7e70` | `i3d_model_cache_ctor` | - ² | STRUCTURAL |
| `0x005f7eb0` | `i3d_model_cache_scalar_deleting_dtor` | - ³ | - |
| `0x005f7f10` | `i3d_sound_cache_dtor` | `__dt__17C_I3D_sound_cacheFv` ¹ | CONFIRMED |
| `0x005f8020` | `i3d_sound_cache_ctor` | - ² | STRUCTURAL |
| `0x005f80c0` | `i3d_anim_cache_dtor` | `__dt__16C_I3D_anim_cacheFv` ¹ | CONFIRMED |
| `0x005f81d0` | `i3d_anim_cache_ctor` | - ² | STRUCTURAL |

¹ The DERIVED destructors, reached from derived slot 5 and from each global's exit destructor;
the PS2's derived slot 8 holds `__dt__17C_I3D_model_cacheFv` (`0x00378430`). Each PS2 derived
destructor stores its own vtable, a dead store, and then runs the template destructor inline; the
PC copy is that sequence without the dead store. The MID symbols `__dt__23C_I3D_cache<9I3D_model>Fv`,
`__dt__23C_I3D_cache<9I3D_sound>Fv` and `__dt__32C_I3D_cache<17I3D_animation_set>Fv` belong to
`i3d_cache_model_dtor`, `i3d_cache_sound_dtor` and `i3d_cache_anim_dtor`. ² Inlined into `__sinit_Main.cpp` (§6c).
³ MSVC-only shape.

### 6a. What the destructor diff established

All three PC destructors and all three PS2 destructors run the identical sequence (§4a).
Member order is reverse-declaration on both sides. The PS2 dispatches `Clear` through
`*vt + 0x18` (slot 6); MSVC inlined it — one PC inlining, three times.

### 6b. Build differences

* **Cache size.** PS2 writes `0x400` (1024 entries) into `+0x1c` of its own layout; the PC
  writes `0x100` (256) into `+0x24`. The PC cap is four times smaller.
* **A trailing field.** The PS2 object carries a `-1` word after the cap that the PC
  constructor never writes.
* **Name matching.** The PS2 uses a plain `strcasecmp`; the PC's wildcard matcher (§5) can
  resolve by pattern and the PS2's cannot — a real build difference, not a rendering one.

### 6c. Why the constructors are capped at STRUCTURAL

There is no PS2 out-of-line constructor for any of the three templates — the entire
construction is spelled inline inside `__sinit_Main.cpp`. That fragment was compared field
for field, and it agrees; but a row whose other side is not a function cannot reach CONFIRMED.

### 6d. The vtables, and the one declaration difference

Both classes declare **13** virtuals and the two tables agree slot for slot, with exactly
two differences:

| | PS2 | PC |
|---|---|---|
| between `Clear` and the destructor | `ClearUpper` | - |
| between `GetDir` and `Duplicate` | - | the resolve-name virtual, §3a slot 11 |

So `Clear` is PS2 slot 6 and PC slot 4. The same one-slot shift appears in all three DERIVED
tables **and** all three BASE tables — the PS2 base table has exactly two live slots (`CreateNew`
at idx 3 and one merged destructor at idx 6), and the PC's has `_purecall` in exactly the slots
the PS2's base table leaves NULL. Three independent tables agreeing makes this a measurement.

* **`ClearUpper` has no PC function and none is expected.** Its work — evict every entry
  above `size_limit` — is inlined into `Create`/`Open` on **both** builds; the PS2's
  out-of-line copy has zero textual callers and is reached only through the vtable.
* **Slot 11 is PC-only.** The PS2 has no current working directory and loads through
  `RW_data` archives at fixed paths, so the concept does not exist there.

### 6e. The destructor levels

The PC emits **three** destructors and three scalar-deleting destructors per cache; the
PS2 merges each `__dt__` with its delete flag, so every PC pair is a **2 → 1 fold** in the
PS2 direction (which is why the base-dtor size ratio is 0.09).

### 6f. `I3D_animation_set` (§7)

Every PC offset of §7 is the PS2 offset + 4: the PC class is polymorphic, while the PS2 class has
no vtable symbol and calls `Release` directly. The PS2 class body supplies the field evidence:

* **refcount** (PS2 `+0x00`) — the PC cache-hit `inc dword [set+4]` at `0x0047fb9a` sits where the
  PS2 twin does `++*(int *)set`; the PS2 ctor seeds it to 1 and `Release__17I3D_animation_setFv`
  decrements and deletes at 0.
* **default `I3D_ANIMOPTIONS`** (PS2 `+0x04`) — `SetAnimation__11I3D_IPolSet...` reads it as
  `if (options == 0) options = set->[+4];`. The PS2 ctor seeds it to **1**.
* **`delete[]`-owned array** (PS2 `+0x08`) — freed by `Close__17I3D_animation_setFv`.
* **animation blob** (PS2 `+0x0c`) — `NumAnimations` reads its first `u16`, `GetEndTime` its second
  `* 0x28`, `GetAnims` skips 4 bytes; `LoadAnimation__FPCcUi...` fills it.
* **What `I3D_ANIMOPTIONS == 2` means** — `Tick__11I3D_IPolSetFi` decodes it. When a track's clock
  passes the clip's end time the options word stored for that track is tested: `== 2` wraps the
  clock by ±`GetEndTime`, i.e. loops; anything else clears the track's playing flag, i.e. stops.

---

## 7. `I3D_animation_set` — the cached clip set

The element type of `C_I3D_cache<17I3D_animation_set>`. Reached from script code only
through the cache, so its layout is decoded from the cache's own twin pair (§6f).

| PC offset | PS2 offset | field | evidence |
|---|---|---|---|
| `+0x00` | - | vtable | the PC class is polymorphic: `Release` is virtual index 0, called as `(**(code **)*set)(set)` at `0x005bfd2a` |
| `+0x04` | `+0x00` | **refcount** | the PC cache-hit path does `inc dword [set+4]` at `0x0047fb9a` (§6f) |
| `+0x08` | `+0x04` | **default `I3D_ANIMOPTIONS`** | `i3d_animation_set_set_default_options` writes **2** (§6f) |
| `+0x0c` | `+0x08` | a `delete[]`-owned array | §6f |
| `+0x10` | `+0x0c` | the loaded animation blob | §6f |

### 7a. What `I3D_ANIMOPTIONS == 2` means

**`== 2` loops the clip; anything else plays it once** (§6f). The default of 1 is therefore
play-once, and `MODEL_PLAYANIM` overriding it to 2 is what makes a
scripted model animation loop until `MODEL_STOPANIM` — which matches every shipped use
(mission `EXTREME`, script `detector_ufo01`: the hovering UFO; mission `EXTREME`, script
`godzila`: the walk cycle).

⚠️ The field is on the **shared, cached** clip set, not on the track, so writing it changes
the default for every later user of the same clip. Nothing in the PC build ever writes it
back.

---

## 8. Open items

* **What slot 11 is for.** Its behaviour is measured (§3a) but no consumer is traced — it has zero
  `E8` callers on the PC and no PS2 counterpart, so nothing says who dispatches it or when.
* **In game:** asking for 257 distinct `.i3d` models should evict, because the cap is 256 on the PC
  (1024 on the PS2) — a PC-only pressure point. Not tested.
