# `C_string` / `C_str_base` — the refcounted string class

**Created** 2026-07-29 20:59 +0200 · **Modified** 2026-09-18 05:38 +0200

PC prefix **`string_`**.

| | |
|---|---|
| PC instance size | `4` — a bare handle |
| PC control block | `C_str_base`, `9 + length` bytes on the heap |
| PC mapped functions | **8**, of which 3 have a PS2 twin |
| PC clusters | the core at `[0x00448db0, 0x00448fe0]`, plus `0x004626b0`, `0x00464bd0`, `0x004656b0` in a second translation unit (the COMDAT split — `structs/web_node.md` §2) |
| PS2 class | `C_string` / `C_str_base`, 4 exported symbols (§4) |

`C_string` is the engine's **refcounted string**. It is one pointer to a `C_str_base` control
block, or `NULL` for the empty string, so a `C_string` costs 4 bytes and copying one is a
refcount bump. It is used everywhere a filename, frame name or mission name is passed around:
the i3d load path, the scene loader, the entity spawn pipeline.

Confidence: **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. Layout

```c
typedef struct C_string {                 // 4 bytes - a bare handle
    /* +0x00 */ C_str_base *block;        // [H] NULL == the empty string
} C_string;

typedef struct C_str_base {               // 9 + length bytes, heap
    /* +0x00 */ uint32_t refcount;        // [H] starts at 1; freed when it reaches 0
    /* +0x04 */ uint32_t length;          // [H] excludes the NUL
    /* +0x08 */ char     data[length + 1];// [H] NUL-terminated
} C_str_base;
```

### 1a. 🔑 `strlen + 9` is the family fingerprint

Every allocation in this family is `operator_new(strlen + 9)` — 4 refcount + 4 length + the
bytes + the NUL. **It is identical on both builds**, and it is what identifies `string_format`
against a near-identical sibling that allocates nothing of the sort (§4). When you meet an
unknown `operator_new(n + 9)` next to a `strlen`, this is the class.

### 1b. The NULL handle is a first-class state

`string_get_data` returns a pointer to the **handle itself** when the block is NULL, so callers
always receive a readable pointer rather than crashing. `string_wildcard_match` treats two NULL
handles as equal, and a NULL against a non-NULL as unequal, without touching either block.

---

## 2. Method family — 8 PC functions, 3 twinned

Ordered by **PC VA**, PC name first. **Every row was body-diffed on both builds.**

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| `0x00448db0` | `string_base_init` | - | CONFIRMED | Fills a freshly allocated block: `refcount = 1`, `length`, bytes, NUL. The PS2 emits these four statements **inline** inside its `C_fstring` ctor, so no symbol exists to map to. 50 direct callers |
| `0x00448df0` | `string_base_release` | - | CONFIRMED | Decrements the block's refcount and frees at zero. Takes the **block**, with the NULL check hoisted into the caller — the inlined normal-path half of the destructor (§3) |
| `0x00448e10` | `string_dtor` | `__dt__8C_stringFv` | CONFIRMED | The **out-of-line** half: takes the handle, checks NULL internally. Identical to the PS2 destructor minus its deleting-flag arm, which MSVC emits separately |
| `0x00448e30` | `string_get_data` | `GetData__Q28C_string10C_str_baseFv` | STRUCTURAL | `block ? block + 8 : &handle`. The PS2 symbol is on the **base** (`return this + 8`); the PC one is on `C_string` and inlines it — one level apart, hence STRUCTURAL |
| `0x00448e40` | `string_append_cstr` | - | CONFIRMED | Reads the current length from `block+4`, `strlen`s the argument, allocates `len1 + len2 + 9` and concatenates. The fingerprint again |
| `0x00448fe0` | `string_format` | `__ct__9C_fstringFPCce` | CONFIRMED | 🔑 `vsnprintf` into a **1024**-byte stack buffer, then build the block. Statement-for-statement with the PS2 including the `strlen + 9` allocation and the NUL write at `+len+8` |
| `0x004626b0` | `string_assign_cstr` | - | CONFIRMED | `strlen`, allocate, fill, **then release the old block** — the assign half |
| `0x00464bd0` | `string_ctor_from_cstr` | - | CONFIRMED | The same body **without** the release — the construct half. The presence or absence of the release step is what separates the pair |
| `0x004656b0` | `string_wildcard_match` | - | CONFIRMED | Compares two `C_string`s with wildcard handling; reads length at `+4` and data at `+8` on both operands. Used against `.i3d` name patterns |

**Not members of this class**, although they refcount something: `string_ref_copy_construct`
(`0x00608370`) refcounts slot **0** of an 8-byte element, while this class's own pair helpers
`copy_elem_stride_8_with_string` and `assign_elem_stride_8_with_string` refcount slot **1** of a
*different* element. Different layouts, different owners.

---

## 3. 🔑 The destructor has two halves — and both look like COMDAT folds

| | `string_base_release` (`0x00448df0`) | `string_dtor` (`0x00448e10`) |
|---|---|---|
| Takes | the **block** | the **handle** |
| NULL check | hoisted into the caller | internal |
| Total references | 242 | 190 |
| `Unwind_*` funclets | **0** | **181** |
| Ordinary calls | 242, from **56 distinct** functions | 9, from **2** functions |

Neither is a folded generic, and the distribution is what shows it:

> **A COMDAT fold shows many DISTINCT game callers. A destructor shows many UNWIND funclets.**
> The distribution is the discriminator, not the count.

`string_base_release`'s 56 callers are 56 functions holding a string temporary — every callsite
has the shape `string_ctor_from_cstr(<literal>); …use…; if (block != 0) string_base_release();`.
`string_dtor`'s 181 unwind funclets are the same 181 functions' throw paths. MSVC inlines the
normal path and keeps one out-of-line copy for the unwinder; ICF demonstrably did **not** fold
the two, because their bodies genuinely differ.

---

## 4. PC vs PS2

* **The PS2 exports only 4 symbols for this family**, and that is the notable asymmetry:
  `C_string`'s destructor, `C_str_base::GetData`, and the two format-constructors (`C_fstring`,
  `fstring`). Metrowerks inlined everything else, so **five of the eight PC functions have no PS2
  twin at all** — the reverse of the usual situation, where the PS2 has more out-of-line methods
  than the PC.
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
* **`ascii_toupper` (`0x00448fc0`) sits in the PC cluster but is not a member** — a hand-inlined
  `toupper` for `a`–`z`. The PS2 links the real libc `toupper` at `0x003d9190`.

---

## 5. Related

`structs/web_node.md` §2 (the COMDAT split) · `structs/crt_runtime_tail.md` (the Dinkumware
`basic_string<char>` members, a different string class in the same image) ·
`ghidra/Game-ghidra.exe.functions.md` (the registry rows for this family).

---

## 6. Open items

| item | state |
|---|---|
| the 8-byte `{dword, C_string}` element | handled by `copy_elem_stride_8_with_string` / `assign_elem_stride_8_with_string` / `uninit_copy_range_stride_8`; the *owning* type is unidentified. Find the vector that holds it and the three names can be promoted off their mechanical form |
| `string_append_cstr`'s second parameter | read, but its role is not pinned; the function was diffed for its allocation shape, not statement by statement |
| is there a `C_string` comparison other than `string_wildcard_match`? | nothing in the cluster does a plain `strcmp`-style compare, which is mildly surprising for a string class |
