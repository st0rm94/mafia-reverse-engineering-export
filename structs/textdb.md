# `G_TextDatabase` — the localized string table

**Created** 2026-08-25 22:28 +0200 · **Modified** 2026-09-17 13:43 +0200

Singleton **`g_textdb`** @ `0x0067a57c`. PC prefix **`textdb_`**.

This is the object behind every `textdb_get_string_by_id(...)` call in the binary — the single
most-called accessor in the UI code. Consumers: `structs/game_menus.md`, `structs/gui_widget.md`.

> Docblocks in the decomp cite sections of this file as `SS<n>`; read that as `§<n>`.

## 1. The object

Three fields, all set by `textdb_load`:

| offset | type | meaning |
|---|---|---|
| `+0x00` | `void *` | the WHOLE `.dta` file, read into one `operator new` allocation |
| `+0x04` | `u32` | record count — the file's first dword |
| `+0x08` | `record *` | the record array, which is simply `buffer + 8` |

A record is **8 bytes**: `{ u32 string_id, char *text }`. On disk the second dword is a file
OFFSET; `textdb_load` adds the buffer base to every one of them at load time, so after loading
they are live pointers into the same allocation. There is no separate string heap.

## 2. Loading — `textdb_load` (`0x0057d770`)

```
dtaOpen(name, 0)                     -> fail returns 0
release any previous buffer
dtaSeek(f, 0, 2) / dtaSeek(f, 0, 0)  -> size the file
buffer = operator new(size + 1); dtaRead(...); dtaClose(...)
count  = *(u32 *)buffer              -> +0x04
records = buffer + 8                 -> +0x08
for each record: record.text += (u32)buffer
```

Returns 1 on success, 0 on a null name or a failed open.

## 3. Lookup — `textdb_get_string_by_id` (`0x0057d850`)

A **binary search** over the sorted record array, seeded at `0x1000` and halving. On a miss it
returns `&s_space_0064dcf0` — a single space — never a null, which is why a widget bound to an
unknown id renders as blank rather than crashing. That behaviour is what the *EMPTY-LINE BUG*
noted in `gui_action_menu_ctor`'s docblock ultimately rests on.

| function | VA | what it does |
|---|---|---|
| `textdb_get_string_by_id` | `0x0057d850` | binary search; a miss returns the single-space literal |
| `textdb_get_string_by_id_or_null` | `0x0057d8b0` | same table, same search; a miss returns `0` |
| `textdb_unload` | `0x0057d820` | frees the buffer and zeroes `+0x00` / `+0x04` |

**The receiver is invisible in the decomp.** Every call is `__thiscall`, so `g_textdb` travels in
ECX and Ghidra renders the call expression with no argument at all — a search of the annotated
decomp for `0067a57c` returns **zero** hits. The singleton is read out of the disassembly
(`mov ecx, 0x67a57c` before each call).

## 4. The static init/atexit pair — `[0x0057d6e0, 0x0057d770)`

Standard MSVC file-scope static scaffolding. A file-scope static's own init/atexit pair marks the
start of its translation unit, so this is where the TU BEGINS.

| function | VA | what it is |
|---|---|---|
| `crt_static_init_textdb` | `0x0057d710` | `call crt_zero_init_textdb; jmp crt_atexit_register_textdb` |
| `crt_zero_init_textdb` | `0x0057d720` | `mov ecx, 0x67a57c; jmp textdb_ctor` — calls the constructor |
| `crt_atexit_register_textdb` | `0x0057d730` | `atexit(&crt_atexit_free_textdb)` |
| `crt_atexit_free_textdb` | `0x0057d740` | `mov ecx, 0x67a57c; jmp textdb_dtor` — the atexit adaptor |
| `textdb_ctor` | `0x0057d750` | `*this = 0` — the constructor itself |
| `textdb_dtor` | `0x0057d760` | `if (buffer) operator delete(buffer)` |

* **`crt_atexit_free_textdb` is only ever taken by ADDRESS** (the `atexit` argument); it has no
  `E8` callers.
* **`textdb_ctor` has exactly one inbound edge** in the whole image — the `E9` tail jump at
  `0x0057d725` — with zero `E8` callers and zero data references.
* **The chain is live and runs at start-up:** `crt_static_init_textdb` is reached from the
  `.CRT$XCU` table at `.data:0x00646190` and `E8`-calls `crt_zero_init_textdb`.

## 5. Callers and invariants

* **`textdb_load`'s sole caller is `loadsave_language_select`** (`0x00559390`). It copies the
  template `tables\textdb_xx.def` and patches the two `xx` bytes from a packed 16-bit language
  code — `0x637a` = `"cz"` when the mission flag at `*(g_act_mission+0x24)+0x40` is clear — then
  passes the same code to `hud_language_select`.
* The lookup binary-searches the record array, so the array must be sorted; the sort is done by the
  shipped file, not by any code here.
* `+0x04` is read by the lookup before `+0x00` is tested, so a lookup against a never-loaded
  database reads an uninitialised count. The constructor zeroes only `+0x00`.

## 6. PC vs PS2

The PS2 singleton is **`0x508100`** and every call is spelled explicitly, e.g.
`GetText__14G_TextDatabaseFUi(0x508100, 0x9e)` — the argument the PC hides in ECX. That is where
the class name `G_TextDatabase` comes from; the PC decomp could not have supplied it.

The class is closed: **7 PS2 methods, 6 PC twins and 1 PS2-console-only.**

| PC VA | PC name | PS2 twin | status |
|---|---|---|---|
| `0x0057d750` | `textdb_ctor` | `__ct__14G_TextDatabaseFv` | CONFIRMED |
| `0x0057d760` | `textdb_dtor` | `__dt__14G_TextDatabaseFv` | CONFIRMED |
| `0x0057d770` | `textdb_load` | `LoadDatabase__14G_TextDatabaseFPcb` | CONFIRMED |
| `0x0057d820` | `textdb_unload` | `Unload__14G_TextDatabaseFv` | CONFIRMED |
| `0x0057d850` | `textdb_get_string_by_id` | `GetText__14G_TextDatabaseFUi` | CONFIRMED |
| `0x0057d8b0` | `textdb_get_string_by_id_or_null` | `GetTextOrNULL__14G_TextDatabaseFUi` | CONFIRMED |
| - | - | `Transpose__14G_TextDatabaseFUiUiUi` | PS2-ONLY |

`Transpose(600, 0x29e, 1000)` remaps every record whose id falls in `[600, 670)` to
`id + 1000`. Its **only** caller is `PSXLoadProductIdFromDTA__15G_LoadSaveClass`, gated on
`PSXIEUVER` — the PlayStation memory-card / product-id path. The PS2 has **36 `PSX*` methods
and the PC has zero**, so the missing PC twin is a console-only subsystem over a closed set, not a
failed search.

The PS2 verb `GetTextOrNULL` names exactly what distinguishes `textdb_get_string_by_id_or_null`
from its sibling — a miss returns `0` instead of the shared single-space literal.
