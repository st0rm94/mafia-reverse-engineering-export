# The crash reporter — `[0x00631620, 0x00633a90]`

**Created** 2026-09-07 09:40 +0200 · **Modified** 2026-10-01 10:31 +0200

A **PC-only game translation unit**, prefix **`crashreport_`**. Not a class: 28 free functions
plus two plain record types. It installs an unhandled-exception filter, and when the game faults
it walks the stack and resolves every return address against that module's **Microsoft linker
`.map` file**, producing a human-readable register dump and call path.

It sits at `0x00631620`, in the middle of the statically-linked library tail
(`structs/crt_runtime_tail.md`) — the linker pulled it in from a `.lib` between the C runtime's
`putenv` object and the classic iostream library, not next to the game's own objects. Its
`.text` boundary is fixed by its `.CRT$XCU` initializer (`crashreport_map_stream_static_init`,
whose one dataref is `.data:0x00646308`, a slot in the table `crt_call_init_fns` walks).

> **In the shipped game the report is built and thrown away.** `engine_main` calls
> `crashreport_install_handler(stub_noop_cdecl_00574a40, stub_noop_cdecl_00574a40)` — a
> bare `ret` for BOTH callbacks — so the 100 000-byte report is formatted into
> `g_crashreport_text_buffer` and freed without ever being shown, logged or written. The
> machinery is complete and reachable; only its two output hooks are empty (§5).

## 1. The two record types

Neither is polymorphic — no vtable, no `kind` byte — and both are plain aggregates the
translation unit instantiates `std::vector` over.

### 1a. The `.map` symbol record — 0xc bytes

```c
typedef struct {
/* +0x00 */ C_string *name;        // [H] the decorated symbol name, e.g. "?foo@@YAXXZ"
/* +0x04 */ unsigned  rva;         // [H] Rva+Base, rebased when the module moved
/* +0x08 */ C_string *lib_object;  // [H] the "lib:object" column of the map line
} S_crashreport_symbol;            // sizeof = 0xc
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `name` | H | `crashreport_module_load_map_symbols` | `crashreport_build_report` | the token between the section:offset column and the `Rva+Base` column |
| +0x04 | `rva` | H | `crashreport_module_load_map_symbols` | `crashreport_build_report`, `crashreport_symbol_compare_by_address` | `crt_sscanf(text, "%X", &rva)`; `+= 0x10000000` when the module's preferred load address is `0x10000000` |
| +0x08 | `lib_object` | H | `crashreport_module_load_map_symbols` | - | parsed, stored and copied; never read (§1b) |

The vector is kept **sorted by `rva`** — `crashreport_module_load_map_symbols` ends with
`crt_qsort(…, 0xc, crashreport_symbol_compare_by_address)` — and
`crashreport_build_report` then scans it backwards for the last record whose `rva` is not
above the address it is resolving: the classic "largest symbol ≤ address" lookup.

### 1b. The module record — 0x34 bytes

```c
typedef struct {
/* +0x00 */ C_string *name;                 // [H] MODULEENTRY32.szModule
/* +0x04 */ C_string *path;                 // [H] MODULEENTRY32.szExePath
/* +0x08 */ unsigned  base;                 // [H] modBaseAddr
/* +0x0c */ unsigned  size;                 // [H] modBaseSize
/* +0x10 */ vector<C_string> strings;       // [M] allocator byte + begin/end/cap; never populated
/* +0x20 */ unsigned  preferred_base;       // [H] "Preferred load address is ..." from the map file
/* +0x24 */ vector<S_crashreport_symbol> symbols;  // [H] allocator byte + begin/end/cap
/* +0x30 */ char      map_tried;            // [H] one-shot guard on the map-file load
} S_crashreport_module;                     // sizeof = 0x34
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `name` | H | `crashreport_module_list_build` | `crashreport_build_report` | printed as `Module: %s` |
| +0x04 | `path` | H | `crashreport_module_list_build` | `crashreport_module_load_map_symbols` | the full path, whose extension is replaced with `.map` |
| +0x08 | `base` | H | `crashreport_module_list_build` | `crashreport_find_module_for_address` | - |
| +0x0c | `size` | H | `crashreport_module_list_build` | `crashreport_find_module_for_address` | the containment test is `base <= addr < base + size` |
| +0x10 | `strings` | M | `crashreport_module_construct_at`, `crashreport_module_assign` | - | constructed, copied and destroyed; nothing pushes into it |
| +0x20 | `preferred_base` | H | `crashreport_module_load_map_symbols` | `crashreport_module_load_map_symbols` | compared against `0x10000000` to decide the rebase |
| +0x24 | `symbols` | H | `crashreport_module_load_map_symbols` | `crashreport_build_report` | stride 0xc, sorted by `rva` |
| +0x30 | `map_tried` | H | `crashreport_build_report` | `crashreport_build_report` | set to 1 before the load, so a module whose `.map` is missing is not searched for twice |

**Two fields are never used, and that is a closed-set measurement.** The module records live only
in `crashreport_build_report`'s stack-local vector, which `crashreport_module_list_build` fills
with zeroed records. Every function that receives a module or symbol record is in this translation
unit or one of its unwind funclets, and none of them pushes into the module's `+0x10` vector or
reads a symbol's `lib_object`. Both are only constructed, copied, assigned and destroyed.

## 2. The `.map` file format it parses

Microsoft linker map, the `/MAP` output. The parser is deliberately loose — it keys on four
line prefixes and ignores everything else.

| prefix | what the parser takes |
|---|---|
| `Timestamp is` | matched only to rewrite the following `(` into `MAPfile ` — a scratch edit inside the line buffer, not stored |
| `Preferred load` | the first digit run on the line, truncated to 8 characters and `crt_sscanf`ed with `%X` into `+0x20` |
| `Address` | starts the symbol section |
| `Static symbols` | starts the symbol section as well |

A symbol line is then split on blanks: the first token (the `section:offset` column) is
skipped, the second becomes `name`, the next digit run beginning with `0` is the `Rva+Base`
and is `%X`-scanned into `rva`, a following `f ` flag column is skipped if present, and the
remainder becomes `lib_object`. A blank line ends the section.

The buffer is 0x400 bytes, and each line is read by MSVC's inline `getline`: the caller
increments `g_crashreport_map_stream_getline_flag` (`0x006f9a7c`) and then calls
`istream_get_line(buf, 0x3ff, '\n')`.

### 2a. The file-scope stream `g_crashreport_map_file`

A classic (pre-standard) MSVC `ifstream` at `0x006f9a78`, built by
`crashreport_map_stream_static_ctor`. `ifstream_ctor` stores the vbtable `g_ifstream_vbtable`
(`{0, 0xc}`) at `+0x00` and constructs the virtual `ios` base at `+0xc`
(`g_crashreport_map_stream_ios_subobject`).

| Offset | Field | Evidence |
|---|---|---|
| `+0x00` | vbtable pointer | `ifstream_ctor`; every access resolves the `ios` base as `this + vbtable[1]` |
| `+0x04` | `istream::_fGline`, the getline-mode flag (`g_crashreport_map_stream_getline_flag`) | the caller increments it before each read; `istream_get_line` tests it, extracts the delimiter when set, and clears it on exit |
| `+0x08` | `istream::x_gcount` | `istream_get_line` counts the characters it extracts here |
| `+0x0c` | the `ios` base | `ifstream_ctor` calls `ios_ctor` on it |
| `+0x14` (`ios+0x08`) | the stream `state` | `crashreport_module_load_map_symbols` tests `& 6` (failbit, badbit) after each open and read, and stores 0 (`clear()`) before retrying another path; `istream_get_line` sets bits 1 and 2 at end of file |

The state test is written against the address `0x006f9a80` (`this + 8`) plus the runtime
vbtable displacement `0xc`, which is why no single symbol names the state word.

## 3. What the report looks like

Assembled by one `crt_sprintf` at the end of `crashreport_build_report`, from the format
literal at `0x00654e64`:

```
Machine: %s
Date: %i.%i.%i %i:%.2i
Address: %08X:%08X
Exception code: %08X (%s)
Registers:
eax = %.8x, ebx = %.8x, ecx = %.8x, edx = %.8x
esi = %.8x, edi = %.8x, ebp = %.8x, esp = %.8x
Execution path:
%s
```

The nineteen arguments map onto `CONTEXT` at `+0xbc` SegCs, `+0xb8` Eip, `+0xb0` Eax,
`+0xa4` Ebx, `+0xac` Ecx, `+0xa8` Edx, `+0xa0` Esi, `+0x9c` Edi, `+0xb4` Ebp and `+0xc4`
Esp — the format string's own register order, which is what identifies the record.
`Machine` is `GetComputerNameA`, the date is `GetLocalTime`.

The execution path is built one frame at a time as
`Module: <module>\n     <module>, <symbol>, offset 0x%X`, or
`<unknown>, <unknovn>, addr 0x%08X` when no `.map` symbol covers the address — the typo is
the shipped binary's. The walk stops at `_WinMainCRTStartup`, `main` or a frame inside
`KERNEL32.DLL`, and gives up after **0x1e** frames with
`<Hierachical list can't be build.>` — also the shipped spelling.

Symbol names pass through `UnDecorateSymbolName(name, buf, 0x100, 0x282)` into a 0x100-byte
buffer; the decomp shows those three pushes on the preceding `string_get_data` call, which takes
only ECX. `0x282` is `UNDNAME_NO_MS_KEYWORDS | UNDNAME_NO_ACCESS_SPECIFIERS |
UNDNAME_NO_MEMBER_TYPE`: the report keeps the return type, the qualified name and the argument
list, and drops calling-convention keywords, access specifiers and `static` / `virtual`.

### 3a. The exception-name table

`g_crashreport_exception_code_table` (`0x00654c30`) is `{DWORD code; char *name}` pairs
terminated by `code == -1`, with `g_crashreport_unknown_exception_name` (`0x00654c2c`,
`"unknown exception"`) as the fallback. Eighteen entries, byte-decoded, codes
`0xc0000005`..`0xc00000fd`:

`ACCESS_VIOLATION` `IN_PAGE_ERROR` `INVALID_HANDLE` `ILLEGAL_INSTRUCTION`
`NONCONTINUABLE_EXCEPTION` `INVALID_DISPOSITION` `ARRAY_BOUNDS_EXCEEDED`
`FLOAT_DENORMAL_OPERAND` `FLOAT_DIVIDE_BY_ZERO` `FLOAT_INEXACT_RESULT`
`FLOAT_INVALID_OPERATION` `FLOAT_OVERFLOW` `FLOAT_STACK_CHECK` `FLOAT_UNDERFLOW`
`INTEGER_DIVIDE_BY_ZERO` `INTEGER_OVERFLOW` `PRIVILEGED_INSTRUCTION` `STACK_OVERFLOW`

## 4. Function coverage — 28 of 28

Ordered by PC VA. Every PS2 column is `-`; §6 explains why that is a measurement.

| PC VA | PC name | PS2 twin | Status | Role |
|---|---|---|---|---|
| 0x00631620 | `crashreport_map_stream_static_init` | - | CONFIRMED | the `.CRT$XCU` initializer stub |
| 0x00631630 | `crashreport_map_stream_static_ctor` | - | CONFIRMED | constructs `g_crashreport_map_file` |
| 0x00631640 | `crashreport_map_stream_register_atexit_dtor` | - | CONFIRMED | `atexit(static_dtor)` |
| 0x00631650 | `crashreport_map_stream_static_dtor` | - | CONFIRMED | reachable only as an `atexit` argument (`LAB_`) |
| 0x00631670 | `crashreport_symbol_compare_by_address` | - | CONFIRMED | the `crt_qsort` predicate; only reference is an immediate operand |
| 0x00631690 | `crashreport_find_file_in_tree` | - | CONFIRMED | recursive `FindFirstFileA` search, self-recursive |
| 0x00631870 | `crashreport_find_module_for_address` | - | CONFIRMED | `base <= addr < base+size` over the module vector |
| 0x006318d0 | `crashreport_build_report` | - | CONFIRMED | the EBP walk and the `crt_sprintf` |
| 0x00631fa0 | `crt_operator_delete_thunk_00631fa0` | - | STRUCTURAL | the EH cleanup thunk; mechanical name, VA suffix |
| 0x00631fc0 | `crashreport_module_load_map_symbols` | - | CONFIRMED | the `.map` parser |
| 0x00632670 | `crashreport_symbol_dtor` | - | CONFIRMED | 0xc-byte record teardown |
| 0x006326a0 | `crashreport_module_list_build` | - | CONFIRMED | `CreateToolhelp32Snapshot` + `Module32First/Next` |
| 0x006328d0 | `crashreport_module_dtor` | - | CONFIRMED | 0x34-byte record teardown, both vectors |
| 0x00632980 | `crashreport_unhandled_exception_filter` | - | CONFIRMED | installed by an immediate operand |
| 0x00632a30 | `crashreport_install_handler` | - | CONFIRMED | stores the two callbacks and installs the filter (§5) |
| 0x00632a60 | `crashreport_module_vector_destroy` | - | CONFIRMED | stride 0x34 |
| 0x00632aa0 | `crashreport_symbol_range_destroy` | - | CONFIRMED | stride 0xc |
| 0x00632ad0 | `crashreport_symbol_vector_insert_n` | - | CONFIRMED | `vector<Symbol>::insert(pos, n, v)` |
| 0x00632e10 | `crashreport_module_vector_insert_n` | - | CONFIRMED | `vector<Module>::insert(pos, n, v)` |
| 0x00633180 | `crashreport_symbol_construct_at` | - | CONFIRMED | copy-construct into raw storage |
| 0x006331f0 | `crashreport_module_construct_at` | - | CONFIRMED | copy-construct incl. both nested vectors |
| 0x00633350 | `crashreport_symbol_assign` | - | CONFIRMED | `Symbol::operator=` |
| 0x006333b0 | `crashreport_module_assign` | - | CONFIRMED | `Module::operator=` |
| 0x00633560 | `crashreport_module_string_vector_assign` | - | CONFIRMED | `vector<C_string>::operator=` on `+0x10` ¹ |
| 0x00633770 | `crashreport_symbol_vector_assign` | - | CONFIRMED | `vector<Symbol>::operator=` on `+0x24` |
| 0x00633a10 | `crashreport_symbol_scalar_deleting_dtor` | - | CONFIRMED | MSVC sdd |
| 0x00633a40 | `string_range_copy_assign` | - | STRUCTURAL | mechanical: a free range algorithm with no receiver ² |
| 0x00633a90 | `string_ref_construct_at_cdecl` | - | CONFIRMED | a `__cdecl` sibling of `string_ref_construct_at`, not a duplicate ² |

¹ Its ECX census is 2 SAFE callsites, both inside `crashreport_module_vector_insert_n`, both on
the module record's own `+0x10` vector (`lea ecx,[edi+0x10]` / `lea ecx,[esi+8]`) — one owner,
hence the family name.

² `string_range_copy_assign` and `string_ref_construct_at_cdecl` operate on `C_string`
references through three raw pointers and have **no receiver**, so they have no owner; the
spelling follows the existing `string_range_uninit_copy`, `string_range_release`,
`string_ref_construct_at` family. `string_ref_construct_at_cdecl` is 25 bytes `__cdecl` with an
EBP frame against the 15-byte `__fastcall` `string_ref_construct_at` (`0x005fe4b0`), compared
disassembly side by side: a sibling whose difference is the calling convention; the suffix
exists because the bare name is taken.

## 5. The report is generated and discarded

`crashreport_install_handler`'s two parameters are stored into
`g_crashreport_begin_callback` and `g_crashreport_report_callback`, and
`crashreport_unhandled_exception_filter` calls them around the report build. Its **sole
caller** is `engine_main`, and the call is:

```c
crashreport_install_handler(stub_noop_cdecl_00574a40, stub_noop_cdecl_00574a40);
```

`stub_noop_cdecl_00574a40` is a single `ret` at a 16-aligned, NOP-padded address — a real
compiler-emitted empty function, not one of the crack's byte patches
(`basic/artifacts/crack.md`). So on a fault the game enumerates every loaded module, parses
every `.map` file it can find, walks the stack, formats the whole report into a 100 KB buffer —
and frees it.

No available binary shows what the two callbacks were meant to do: every `Game.exe` variant of
this release carries the same install call and the same unit byte for byte, and none of the DLLs
shipped with the game contains a crash reporter.

### 5a. The re-entrancy hack

A second fault takes a different path. The filter disarms itself with
`SetUnhandledExceptionFilter(NULL)` and then reads the byte at `Context->Eip`; if it is
`0x8b` — the `mov r32, r/m32` opcode — it points `Context->Ecx` at
`g_crashreport_null_this_scratch` and returns `EXCEPTION_CONTINUE_EXECUTION`. In other
words, when the reporter itself dereferences a null `this`, the handler supplies a dummy
object and lets the faulting instruction retry. Anything else returns 0 and the process
dies. It is a blunt instrument and it is deliberate: the reporter must not be the thing
that kills the process.

## 6. PC vs PS2

**The whole translation unit is PC-only, and that is a closed-set measurement rather than a
failed search:**

* The PS2 ELF carries its own **source-file manifest** — 173 `__sinit_<file>.cpp` symbols in
  the clear. It contains **no crash, exception, report, symbol or map file**. The absence of
  a file from that manifest proves the file was not compiled in.
* Every Win32 primitive the unit is built on has no PS2 counterpart by construction:
  `SetUnhandledExceptionFilter`, `EXCEPTION_POINTERS`/`CONTEXT`,
  `CreateToolhelp32Snapshot` + `Module32First`/`Module32Next`, `UnDecorateSymbolName`,
  `GetComputerNameA`, `FindFirstFileA`.
* The artifact it consumes — a **Microsoft linker `.map` file** — does not exist for a
  Metrowerks/GCC PS2 build.

There is therefore no PS2 function anywhere in the unit to have been inlined from.

## 7. Related

`structs/crt_runtime_tail.md` (the library bands either side of this unit, and the classic
iostream it reads `.map` files with) · `structs/app_bootstrap.md` (`engine_main`'s own
translation unit, the sole caller) · `basic/artifacts/crack.md` (why a bare `ret` needs checking
before it is called a stub).

## 8. Open items

| item | state |
|---|---|
| in game | not tested: force a fault (a debugger write to a code page, or a mod) and confirm nothing is written to disk and no dialog appears |
