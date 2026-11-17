# `G_TextDatabase` — the localized string table

**Created** 2026-08-25 22:28 +0200 · **Modified** 2026-10-30 15:47 +0100

This is the object behind every `textdb_get_string_by_id(...)` call in the binary, the single most-called
accessor in the UI code. Consumers: `structs/game_menus.md`, `structs/gui_widget.md`.

| | |
|---|---|
| PC prefix | `textdb_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0xc`; the singleton `g_textdb` @ `0x0067a57c` |
| PC constructor | `textdb_ctor` @ `0x0057d750` |
| PC translation unit | `[0x0057d6e0, 0x0057d910)`, guard to guard: 12 starts, all named |
| PS2 class | `G_TextDatabase` (tag `14G_TextDatabase`), 7 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Three fields, all set by `textdb_load`.

```c
typedef struct G_TextDatabase {               // 0xc bytes
    /* +0x00 */ void     *buffer;             // [H] the WHOLE .dta file, in one operator new allocation
    /* +0x04 */ uint32_t count;               // [H] record count, the file's first dword
    /* +0x08 */ S_text_record *records;       // [H] the record array, which is buffer + 8
} G_TextDatabase;
```

### 1a. `S_text_record` — one string, 8 bytes

```c
typedef struct S_text_record {                // 8 bytes
    /* +0x00 */ uint32_t string_id;           // [H] the numeric id every caller passes
    /* +0x04 */ char     *text;               // [H] the string, inside the same allocation
} S_text_record;
```

There is no separate string heap: on disk `text` is a file offset, and `textdb_load` adds the buffer base
to every one of them.

## 2. Field evidence

Measured by a register-tag census of the class's functions and by reading all six bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | buffer | H | `textdb_ctor`, `textdb_load`, `textdb_unload` | `textdb_dtor`, `textdb_load`, `textdb_unload` | - |
| +0x4 | count | H | `textdb_load`, `textdb_unload` | `textdb_get_string_by_id`, `textdb_get_string_by_id_or_null`, `textdb_load` | - |
| +0x8 | records | H | `textdb_load` | `textdb_get_string_by_id`, `textdb_get_string_by_id_or_null`, `textdb_load` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | string_id | H | the shipped file | `textdb_get_string_by_id`, `textdb_get_string_by_id_or_null` | the search key; the shipped file holds the records sorted |
| +0x4 | text | H | the shipped file, `textdb_load` | `textdb_get_string_by_id`, `textdb_get_string_by_id_or_null` | a file offset on disk, a live pointer after the load |

## 3. Method family

6 of 7 PS2 methods mapped, all CONFIRMED; `Transpose` is in the PS2-only table of §8. PC-only: the six
functions of the two static-init pairs. No vtable, so no base stub.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0057d6e0 | `crt_static_init_guard_0057d6e0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0057d700 | `crt_atexit_register_0057d700` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x0057d710 | `crt_static_init_textdb` | `g_textdb` | - | - | - | `.CRT$XCU` entry at `.data:0x00646190`: calls the next two |
| 0x0057d720 | `crt_zero_init_textdb` | `g_textdb` | - | - | - | `mov ecx, g_textdb; jmp textdb_ctor` |
| 0x0057d730 | `crt_atexit_register_textdb` | `g_textdb` | - | - | - | `atexit(&crt_atexit_free_textdb)` |
| 0x0057d740 | `crt_atexit_free_textdb` | `g_textdb` | - | - | - | `mov ecx, g_textdb; jmp textdb_dtor`; reached only by address |
| 0x0057d750 | `textdb_ctor` | instance | `__ct__14G_TextDatabaseFv` | 0x002e7b20 | CONFIRMED | `buffer = 0`; one inbound edge, the `E9` tail jump above |
| 0x0057d760 | `textdb_dtor` | instance | `__dt__14G_TextDatabaseFv` | 0x002e7b30 | CONFIRMED | frees `buffer` |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0057d770 | `textdb_load` | `LoadDatabase__14G_TextDatabaseFPcb` | 0x002e7b90 | CONFIRMED | §4; sole caller `loadsave_language_select` |
| 0x0057d820 | `textdb_unload` | `Unload__14G_TextDatabaseFv` | 0x002e7cb0 | CONFIRMED | frees `buffer`, zeroes `buffer` and `count`; sole caller `loadsave_close` |
| 0x0057d850 | `textdb_get_string_by_id` | `GetText__14G_TextDatabaseFUi` | 0x002e7da0 | CONFIRMED | binary search; a miss returns the single-space literal (§5) |
| 0x0057d8b0 | `textdb_get_string_by_id_or_null` | `GetTextOrNULL__14G_TextDatabaseFUi` | 0x002e7e80 | CONFIRMED | the same search; a miss returns 0 |

### 3d. Free helpers

None.

## 4. Loading: `textdb_load` (`0x0057d770`)

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

## 5. Lookup: `textdb_get_string_by_id` (`0x0057d850`)

A **binary search** over the sorted record array, seeded at `0x1000` and halving. On a miss it returns
`&s_space_0064dcf0`, a single space, never a null, which is why a widget bound to an unknown id renders as
blank rather than crashing. That behaviour is what the *EMPTY-LINE BUG* noted in `gui_action_menu_ctor`'s
docblock ultimately rests on.

**The receiver is invisible in the decomp.** Every call is `__thiscall`, so `g_textdb` travels in ECX and
Ghidra renders the call expression with no argument at all: a search of the annotated decomp for
`0067a57c` returns **zero** hits. The singleton is read out of the disassembly (`mov ecx, 0x67a57c` before
each call).

## 6. The static init/atexit pair

Standard MSVC file-scope static scaffolding. A file-scope static's own init/atexit pair marks the start of
its translation unit: the unit's guard pair at `0x0057d6e0` and then this one at `[0x0057d710, 0x0057d750)`
(§3a).

* **`crt_atexit_free_textdb` is only ever taken by ADDRESS** (the `atexit` argument); it has no `E8` callers.
* **`textdb_ctor` has exactly one inbound edge** in the whole image, the `E9` tail jump at `0x0057d725`, with
  zero `E8` callers and zero data references.
* **The chain is live and runs at start-up:** `crt_static_init_textdb` is reached from the `.CRT$XCU` table
  at `.data:0x00646190` and `E8`-calls `crt_zero_init_textdb`.

## 7. Callers and invariants

* **`textdb_load`'s sole caller is `loadsave_language_select`** (`0x00559390`). It copies the template
  `tables\textdb_xx.def` and patches the two `xx` bytes from a packed 16-bit language code (`0x637a` = `"cz"`
  when the mission flag at `*(g_act_mission+0x24)+0x40` is clear), then passes the same code to
  `hud_language_select`.
* The lookup binary-searches the record array, so the array must be sorted; the sort is done by the shipped
  file, not by any code here.
* Neither lookup tests `buffer`, and they read `count` first, so a lookup against a never-loaded database
  reads a `count` nothing has written: the constructor zeroes only `buffer`.

## 8. PC vs PS2

The PS2 singleton is **`0x508100`** and every call is spelled explicitly, e.g.
`GetText__14G_TextDatabaseFUi(0x508100, 0x9e)`, the argument the PC hides in ECX. That is where the class
name `G_TextDatabase` comes from; the PC decomp could not have supplied it.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| `0x002e7cf0` | `Transpose__14G_TextDatabaseFUiUiUi` | console-only: its **only** caller is `PSXLoadProductIdFromDTA__15G_LoadSaveClass`, gated on `PSXIEUVER`, the PlayStation memory-card / product-id path. `Transpose(600, 0x29e, 1000)` remaps every record whose id falls in `[600, 670)` to `id + 1000`. The PS2 has **36 `PSX*` methods and the PC has zero**, so this is a console-only subsystem over a closed set, not a failed search |

The PS2 verb `GetTextOrNULL` names exactly what distinguishes `textdb_get_string_by_id_or_null` from its
sibling: a miss returns `0` instead of the shared single-space literal.

## 9. Related

`structs/game_menus.md`, `structs/gui_widget.md` (the consumers) · `structs/loadsave.md` (the language
select and the close that load and unload the table).

## 10. Open items

None.
