# `C_string` / `C_str_base` — the refcounted string class

**Created** 2026-07-29 20:59 +0200 · **Modified** 2026-11-01 17:14 +0100

`C_string` is the engine's **refcounted string**. It is one pointer to a `C_str_base` control block, or `NULL` for the empty
string, so a `C_string` costs 4 bytes and copying one is a refcount bump. It is used everywhere a filename, frame name or mission
name is passed around: the i3d load path, the scene loader, the entity spawn pipeline.

| | |
|---|---|
| PC prefix | `string_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x4`, a bare handle; the block is `9 + length` bytes on the heap |
| PC constructor | `string_ctor_from_cstr`, `string_format` |
| PC translation unit | no unit of its own: the core `[0x00448db0, 0x00449000)` sits in the unit `police-board.md` hosts, and the other members are COMDAT copies placed where a caller first needed them (`structs/web_node.md` §3) |
| PS2 class | `C_string` (tag `8C_string`), `C_str_base`, `C_str` (tag `5C_str`), `C_fstring` (tag `9C_fstring`); 5 methods in all (§6) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_string {             // 4 bytes, a bare handle
    /* +0x00 */ C_str_base *block;        // [H] the control block; NULL is the empty string
} C_string;
```

### 1a. `C_str_base`

```c
typedef struct C_str_base {           // 9 + length bytes, heap
    /* +0x00 */ uint32_t refcount;        // [H] starts at 1; the block is freed when it reaches 0
    /* +0x04 */ uint32_t length;          // [H] the characters, without the terminating NUL
    /* +0x08 */ char     data[1];         // [H] length + 1 bytes, NUL-terminated
} C_str_base;
```

`data` is written with one element because its size is per block: the allocation is `strlen + 9`, 4 bytes of refcount, 4 of
length, the characters and the NUL.

## 2. Field evidence

### 2a. `C_string`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | block | H | `string_assign_cstr`, `string_ctor_from_cstr`, `string_format`, `string_append_cstr_in_place`, `string_get_mutable_data_at`, `string_ref_assign` | `string_dtor`, `string_get_data`, `string_concat_cstr`, `string_wildcard_match`, `string_ensure_buffer` | NULL is a first-class state (§4b) |

### 2b. `C_str_base`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | refcount | H | `string_base_init`, `string_base_init_concat`, `string_base_release`, `string_ref_assign`, `string_get_mutable_data_at` | `string_base_release`, `string_get_mutable_data_at`, `string_ref_assign` | - |
| +0x4 | length | H | `string_base_init`, `string_base_init_concat`, `string_append_cstr_in_place` | `string_concat_cstr`, `string_wildcard_match`, `string_get_mutable_data_at` | - |
| +0x8 | data | H | `string_base_init`, `string_base_init_concat`, `string_format`, `string_append_cstr_in_place` | `string_get_data`, `string_concat_cstr`, `string_wildcard_match` | - |

## 3. Method family

5 of the 5 PS2 methods with a PC body are mapped (§6); 22 PC functions have no PS2 counterpart. Rows `string_range_*` and
`string_vector_*` are the MSVC `std::vector<C_string>` instantiations, with no PS2 body to compare. Status comes from the function
registry `Game-ghidra.exe.functions.md`; the `string_format` and `string_get_data` rows were read on both builds.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00448e10 | `string_dtor` | instance | `__dt__8C_stringFv` | 0x0010f3a0 | CONFIRMED | The out-of-line half of the destructor: takes the handle and tests NULL itself (§5). The PS2 body adds a deleting-flag arm, which MSVC emits apart |
| 0x00448fe0 | `string_format` | instance | `__ct__9C_fstringFPCce` | 0x0010f410 | CONFIRMED | `vsnprintf` into a 1024-byte stack buffer, then builds the block; statement for statement with the PS2 including the `strlen + 9` allocation and the NUL write |
| 0x004626b0 | `string_assign_cstr` | instance | `__as__5C_strFPCc` | 0x00386110 | CONFIRMED | `strlen`, allocate, fill, then release the old block; the assign half. Statement for statement with the PS2 body, NULL argument included |
| 0x00464bd0 | `string_ctor_from_cstr` | instance | - | - | - | The same body without the release; the construct half |
| 0x005fbf20 | `string_ref_assign` | a slot of a `C_string` range | - | - | - | `operator=`: adds a reference to the source block, releases the destination's old one, stores the pointer |
| 0x005fe4b0 | `string_ref_construct_at` | instance | - | - | - | Copy-constructs one `C_string` reference into raw storage; the `__thiscall` instantiation |
| 0x005fe4c0 | `string_ref_scalar_deleting_dtor` | instance | - | - | - | The scalar deleting destructor of one `C_string` reference |
| 0x00608370 | `string_ref_copy_construct` | instance | - | - | - | Refcounts slot 0 of an 8-byte element, not a bare `C_string` |
| 0x00633a90 | `string_ref_construct_at_cdecl` | instance | - | - | - | Copy-constructs one `C_string` reference into raw storage; the `__cdecl` instantiation |

### 3b. Accessors

The class has no one-load or one-store method: `string_get_data` tests the handle.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00448db0 | `string_base_init` | - | - | - | Fills a freshly allocated block: `refcount = 1`, `length`, bytes, NUL. 50 direct callers. The PS2 spells these four statements inside its `C_fstring` ctor |
| 0x00448df0 | `string_base_release` | - | - | - | Decrements the block's refcount and frees at zero. Takes the block, with the NULL test in the caller: the inlined normal-path half of the destructor (§5) |
| 0x00448e30 | `string_get_data` | `GetData__Q28C_string10C_str_baseFv` | 0x003024f0 | STRUCTURAL | `block ? block + 8 : &handle`. The PS2 symbol sits on the base and returns `this + 8`; the PC one sits on `C_string` and inlines it, one level apart |
| 0x00448e40 | `string_concat_cstr` | - | - | - | `operator+` with a C string: reads the length from `block + 4`, allocates `len1 + len2 + 9` and builds a new block into the hidden result slot, its second parameter (`ret 8`); `*this` is not written |
| 0x004656b0 | `string_wildcard_match` | - | - | - | The class's only comparison: reads length and data on both operands and compares case-insensitively; in the second operand `?` matches one character and the first `*` the rest. Used against `.i3d` name patterns |
| 0x0053c1d0 | `string_ensure_buffer` | - | - | - | The string at `this + 0xc`: when it has no block, builds a temporary empty one and returns its data pointer, the block's refcount dropped again. Callers `vehicle_sound_slot_init`, `i3d_sound_cache_duplicate` |
| 0x005a5b90 | `string_get_mutable_data_at` | - | - | - | Copy on write: when the refcount is not 1, clones the block and drops one reference on the old; returns `data + index` |
| 0x005fbcb0 | `string_base_init_concat` | - | - | - | Builds one block from two byte ranges: refcount 1, the summed length, both ranges, the NUL. The caller owns the allocation |
| 0x005fde20 | `string_vector_destroy` | - | - | - | Destroys a `vector<C_string>`: releases every element, frees the storage, zeroes begin / end / capacity at `+0x04` / `+0x08` / `+0x0c` |
| 0x005fdef0 | `string_vector_insert_n` | - | - | - | Inserts `n` copies of one reference into a `vector<C_string>`, reallocating when the capacity is short |
| 0x005fe420 | `string_range_assign` | - | - | - | Assigns one value over a range of references |
| 0x005fe460 | `string_range_copy_backward` | - | - | - | `std::copy_backward` over references |
| 0x00604a10 | `string_append_cstr_in_place` | - | - | - | The in-place append: builds a new block from the old text and the argument and stores it back in the handle |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00448e00 | `crt_operator_delete_thunk_00448e00` | - | - | - | A jump to `operator_delete_wrapper` beside the string core; 27 callers, most of them unwind funclets. Hosted by `police-board.md` |
| 0x00448fc0 | `ascii_toupper` | - | - | - | Upper-cases one ASCII letter; `string_wildcard_match` calls it, and so do `i3d_cache_model_open` and others. Hosted by `police-board.md` |
| 0x005fdec0 | `string_range_release` | - | - | - | Releases every reference in `[first, last)` |
| 0x005fe190 | `string_range_uninit_copy` | - | - | - | `std::uninitialized_copy` over references |
| 0x005fe1d0 | `string_range_uninit_fill_n` | - | - | - | `std::uninitialized_fill_n` over references |
| 0x00633a40 | `string_range_copy_assign` | - | - | - | Assigns a range of references onto a constructed range |

**Not members of this class**, although they refcount something: `i3d_cache_items_copy_construct_one` and
`i3d_cache_items_copy_assign_one` refcount slot 1 of a different element, the `{T *object, C_string name}` item of
`C_I3D_cache<T>` (`structs/i3d_cache.md` §3c).

**No other comparison exists.** Of the 152 `strcmp`-family callsites in the decomp, none compares two `C_string` blocks: the one
call that compares two `+8` data fields is the MSVC runtime matching exception type names, and the functions that take string data
out of a `C_string` compare it with a plain `char *`.

## 4. The block

### 4a. `strlen + 9` is the family fingerprint

Every allocation of this family is `operator_new(strlen + 9)`. An unknown `operator_new(n + 9)` next to a `strlen` is this class;
§6 uses the same fingerprint to map `string_format`.

### 4b. The NULL handle is a first-class state

`string_get_data` returns a pointer to the **handle itself** when the block is NULL, so callers always receive a readable pointer.
`string_wildcard_match` treats two NULL handles as equal, and a NULL against a non-NULL as unequal, without touching either block.

## 5. The destructor has two halves

| | `string_base_release` (`0x00448df0`) | `string_dtor` (`0x00448e10`) |
|---|---|---|
| Takes | the **block** | the **handle** |
| NULL test | in the caller | internal |
| Total references | 242 | 190 |
| `Unwind_*` funclets | 0 | 181 |
| Ordinary calls | 242, from 56 distinct functions | 9, from 2 functions |

Neither is a folded generic, and the distribution is what shows it: a COMDAT fold shows many distinct game callers, a destructor
many unwind funclets. `string_base_release`'s 56 callers each hold a string temporary, with the shape
`string_ctor_from_cstr(<literal>); …use…; if (block != 0) string_base_release();`. `string_dtor`'s 181 funclets are the throw paths
of those functions. MSVC inlines the normal path and keeps one out-of-line copy for the unwinder, and the two bodies differ, so no
fold joined them.

## 6. PC vs PS2

* **The PS2 exports 6 symbols for this family**, and that is the notable asymmetry:
  `C_string`'s destructor, `C_str_base::GetData`, the two format-constructors (`C_fstring`,
  `fstring`), and two members of `C_str`, a second class over the same refcounted block:
  `__as__5C_strFPCc`, the assignment from a C string and the twin of `string_assign_cstr`, and
  `__ct__5C_strFPCce`, a format constructor with a 1040-byte buffer and a NULL-format test, which
  `string_format` (1024 bytes, no test) does not match. Metrowerks inlined everything else, so
  **most PC functions have no PS2 twin at all**, the reverse of the usual situation, where the PS2
  has more out-of-line methods than the PC.
* **The PS2 model cache compares names with `strcasecmp`.** At the position where the PC
  `i3d_cache_model_open` runs the wildcard matcher inline, `Open__23C_I3D_cache<9I3D_model>` calls
  `strcasecmp`, so `string_wildcard_match` has no PS2 counterpart.
* **Zero layout delta.** `refcount +0x00`, `length +0x04`, `data +0x08`, allocation `strlen + 9`
  — identical on both builds. This is one of the few classes in the project with no rebase table.
* **`C_fstring` is a subclass used as a temporary**: `C_fstring(fmt, ...)` formats and hands back
  a `C_string`. The PC does not emit it as a distinct type — `string_format` writes straight into
  a `C_string` handle.
* ⚠️ **The PS2 has two constructors with the identical mangled signature `FPCce`**
  (`const char *` + ellipsis): `__ct__9C_fstringFPCce` and `__ct__7fstringFPCce`. Size and
  position cannot separate them; storage can. The second builds a `std::basic_string` and calls
  `assign()` — no refcount block, no `+9` — so `string_format` maps to the first. **`fstring`
  (no `C_`) is an unrelated, `std::basic_string`-backed class. Do not map anything to it.**
* **`ascii_toupper` sits in the PC cluster but is not a member**: a hand-inlined `toupper` for `a`–`z`. The PS2 links the real
  libc `toupper` at `0x003d9190`.

## 7. Related

`structs/web_node.md` §3 (the COMDAT split) · `structs/i3d_cache.md` and `structs/app_bootstrap.md` (the `vector<C_string>` users) ·
`structs/crt_runtime_tail.md` (the Dinkumware `basic_string<char>` members, a different string class in the same image) ·
`ghidra/Game-ghidra.exe.functions.md` (the registry rows for this family).

## 8. Open items

Nothing is open.
