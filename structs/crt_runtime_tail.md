# The statically-linked runtime tail — `[0x006243b7, 0x00633d75)`

**Created** 2026-09-04 18:07 +0200 · **Modified** 2026-09-28 19:12 +0200

This is a **linker region, not a translation unit and not a class** — the tail of `.text` where
the linker parked the objects that came out of `.lib` files rather than out of Mafia's own
source. `structs/app_bootstrap.md` is the precedent for a doc shaped like this.

It contains the three Dinkumware `<stdexcept>` classes, four out-of-line `basic_string<char>`
members, the last import-thunk block, and the MSVC 6 C runtime — floating-point init, exit
handlers, C++ EH/SEH, the heap, stdio, `qsort`, the string and ctype routines, the x87 math
entry points, the Pentium FDIV-erratum object, the classic iostream, and MSVC's `exception` and
`type_info` classes. **No game translation unit lives here**, with one deliberate exclusion: the
fifth band's span also contains a PC-only game translation unit, the `.map`-file crash reporter,
which has its own file `structs/crash_report.md` — a finding is about the symbols it was
measured on, never about the address range they occupy.

| band | span | Ghidra starts | what is in it |
|---|---|---|---|
| 1 | `[0x006243b7, 0x00625021)` | 64 | the `<stdexcept>` classes, the last import thunks, FP init, the exit handlers, the C++ EH / SEH leaf helpers, the acos x87 entry |
| 2 | `[0x00625021, 0x00627686]` | 80 | stdio, `qsort`, the string and ctype routines, `memcpy` / `memmove`, the heap, `pow` and `asin`, the startup group, `exception` and `type_info` |
| 3 | `[0x006276dc, 0x0062ad14]` | 86 | the `cvt.c` tail, the whole C++ EH engine, the x87 FP error dispatch, `_output`, `_input`, the math-error machinery |
| 4 | `[0x0062ad26, 0x0062ec1e]` | 96 | both small-block heaps, the startup argv / environment group, the 12-byte long double, `ieee87`, `osfinfo`, `_matherr` |
| 5 | `[0x0062ec30, 0x00633d59]` | 101 ¹ | the Pentium FDIV-erratum object, `tzset` / `mbctype` / the decimal converters, `calloc`, `getenv` / `putenv`, and the classic iostream |

¹ Band 5's 101 starts (each with a def) break down as **73 library functions** (15 of them the
function-pass starts of §9) and **28 functions in the crash reporter's span**
`[0x00631620, 0x00633a90]` (documented in `structs/crash_report.md`): 25 `crashreport_` functions
plus `crt_operator_delete_thunk_00631fa0`, `string_range_copy_assign` and
`string_ref_construct_at_cdecl`. The band's seven phantoms (§11) are not functions.

Band 5 itself splits into three VA sub-bands:

| sub-band | span | what |
|---|---|---|
| A | `[0x0062ec30, 0x0062f9bb]` | the **Pentium FDIV-erratum FP object** — MSVC's `_adj_*` / `__safe_*` replacements for the x87 divide and remainder, plus `_fpclass` |
| B | `[0x0062fa4d, 0x006315e9]` | `tzset.c`, `mbctype.c`, the `_LDBL12` decimal converters, `winsig.c`, `calloc`, `getenv` / `putenv`, `__crtCompareStringA` |
| F | `[0x00633ab0, 0x00633d75)` | the MSVC 6 **classic (pre-standard) iostream** — `ios` / `istream` / `ifstream` / `filebuf`, continuing past the end of the region |

---

## 1. Why Ghidra had already named part of it

Roughly half the functions here arrive with real names while every game-side vector and
matrix helper in this project arrives as `FUN_`. Measured over **all** of Ghidra's default
analysis — 5322 definitions, 1371 of them non-`FUN_` — the names come from three mechanisms,
and only the third looks like "Ghidra understood the code":

| mechanism | count | what it produces |
|---|---|---|
| EH-funclet auto-naming | ~1330 | `Unwind_00635007`, … — generated from the exception tables |
| **Import-table naming** | ~45 | `dtaOpen`, `_CreateJoyBase_20`, `RtlUnwind`, `S_matrix::Inverse` |
| **Function ID (FID) hash match** | **~38** | `__ftol`, `__global_unwind2`, `operator_new`, `_malloc`, `_strlen`, `__cfltcvt`, `__amsg_exit`, `exception::~exception` |

FID only matches **library** code Ghidra ships a hash for; the import table only names
**thunks**. Neither can ever fire on `Game.exe`'s own code. So the answer to *"is it because
we are near the end of the address range?"* is **yes, but by way of the linker**: `.lib`
objects are placed after the program's objects, so the tail of `.text` is exactly where both
mechanisms have something to bite on. Position is the symptom; provenance is the cause.

⚠️ Only ~38 of 5322 definitions carry a FID name. FID coverage in this binary is tiny and
clustered almost entirely in this region.

---

## 2. The Dinkumware `<stdexcept>` classes — `[0x006243b7, 0x006248a8)`

### 2a. The binary names these classes itself

C++ exception handling needs type descriptors even when RTTI is switched off, so the mangled
class names sit in `.data` in the clear. **Three independent artifacts agree, and no row below
was assigned from a guess:**

| vtable | `_ThrowInfo` | catchables | type descriptor | class |
|---|---|---|---|---|
| `0x0063ef28` | `0x00644c38` | 3 | `.?AVlength_error@std@@` @ `0x00651b60` | `std::length_error` |
| `0x0063ef48` | `0x00644ca8` | 2 | `.?AVlogic_error@std@@` @ `0x00651b40` | `std::logic_error` |
| `0x0063ef5c` | `0x00644d60` | 3 | `.?AVout_of_range@std@@` @ `0x00651b80` | `std::out_of_range` |
| - | - | - | `.?AVexception@@` @ `0x00651b28` | `exception` (the base, §3) |

Each `_ThrowInfo` is `{attributes = 0, pmfnUnwind = <the class's plain dtor>, 0, catchables}`,
and the **catchable counts encode the hierarchy** — 2 for the base, 3 for each derived class.

### 2b. Object layout

```
+0x00  vptr
+0x0c  embedded basic_string<char>   { +0x00 allocator byte, +0x04 _Ptr, +0x08 _Len, +0x0c _Res }
```

so the string's `_Ptr` lands at `this+0x10`, which is exactly what `std_logic_error_what`
returns. `logic_error` is the base; `length_error` and `out_of_range` add nothing but a vtable.

### 2c. The vtable — 3 slots, identical shape on all three classes

| slot | `logic_error` | `length_error` | `out_of_range` |
|---|---|---|---|
| 0 | `std_logic_error_scalar_deleting_dtor` | `std_length_error_scalar_deleting_dtor` | `std_out_of_range_scalar_deleting_dtor` |
| 1 | `std_logic_error_what` | *(the same function)* | *(the same function)* |
| 2 | `std_logic_error_throw_copy` | `std_length_error_throw_copy` | `std_out_of_range_throw_copy` |

Slot 1 is **one function filling three slots**. That is ordinary vtable inheritance, not an
ICF fold: all 3 datarefs belong to one hierarchy and neither derived class overrides it.

### 2d. Two destructors per class, and where the twin goes

The PC emits **two** destructors per class — a plain one, reachable only through the
`_ThrowInfo`'s `pmfnUnwind` field, and a scalar-**deleting** one in vtable slot 0. GCC folds
both into a single flagged destructor, so the PS2 twin belongs on the **deleting** one:

```
PC   std_logic_error_scalar_deleting_dtor(this, byte flag)   if (flag & 1) operator delete
PS2  __dt__Q23std11logic_errorFv(long, short)                if (0 < flag) __dl__FPv
```

Settled by **signature, before either body was read** — a mangled symbol spells its whole
signature.

### 2e. `_Xlen` and `_Xran` inline their own derived constructors

`std_string_xlen` and `std_string_xran` each call the `logic_error` **base** constructor and
then store the derived vtable pointer themselves. That bare store *is* the derived
constructor — MSVC inlined it, which is why `length_error(const string&)` and
`out_of_range(const string&)` have no function anywhere in the image. Marked inline in the
decomp at both sites.

---

## 3. MSVC's `exception` and `type_info` — `[0x00627059, 0x00627150)`

`.?AVexception@@` is listed in §2a as "the base" of the three `<stdexcept>` classes. Its own
methods live in band 2.

```c
typedef struct exception {         // 12 bytes
    /* +0x00 */ void **vptr;       // g_exception_vt (0x0063efc8)
    /* +0x04 */ char  *_m_what;    // the message, or NULL
    /* +0x08 */ int    _m_doFree;  // 1 => _m_what is owned and must be released
} exception;
```

**The vtable has exactly two slots**, read straight from `.rdata`:

| slot | function | evidence |
|---|---|---|
| 0 | `exception_scalar_deleting_dtor` (`0x00627059`) | `datarefs == 1`, no callers |
| 1 | `exception_what` (`0x00627112`) | `datarefs == 1`, no callers |

`exception::~exception` (`0x006270fc`, a Ghidra FID name kept per §7) is the plain
destructor; it has 4 `E8` callers — the three `<stdexcept>` destructors and the deleting
form — plus 5 `E9` tail jumps. `exception_ctor_from_cstr` and `exception_copy_ctor` are the
base constructors the `<stdexcept>` classes chain to; the copy constructor's `_m_doFree`
branch (share the pointer when it is a literal, deep-copy it when it is owned) is the whole
reason that third field exists.

`type_info` is here for the same reason the type descriptors in §2a are: **C++ exception
handling needs them even with RTTI switched off.** Only its two destructors are out of line,
both reached exclusively through `type_info::vftable` (`.rdata:0x0063efe8`).

---

## 4. The import thunks — `[0x006248a8, 0x006248e4)`

Ten 6-byte `jmp dword ptr [IAT]`. The last import block in `.text`; the 29-thunk
`ls3df.dll` block sits immediately below at `[0x006242e0, 0x006243ac)`.

| thunk | IAT slot | DLL | callers |
|---|---|---|---|
| `_CreateJoyBase_20` | `0x0063b018` | `ijoy.dll` | 1 |
| `_CreateINetBase_4` | `0x0063b028` | `inet.dll` | 1 |
| `dtaRead` | `0x0063b248` | `rw_data.dll` | **261** |
| `dtaWrite` | `0x0063b24c` | `rw_data.dll` | 162 |
| `dtaSeek` | `0x0063b240` | `rw_data.dll` | 62 |
| `dtaClose` | `0x0063b244` | `rw_data.dll` | 62 |
| `dtaOpen` | `0x0063b23c` | `rw_data.dll` | 33 |
| `dtaCreate` | `0x0063b234` | `rw_data.dll` | 13 |
| `dtaOpenWrite` | `0x0063b238` | `rw_data.dll` | 5 |
| `dtaSetDtaFirstForce` | `0x0063b230` | `rw_data.dll` | 1 |

Full DLL inventory: `basic/artifacts/dll.md`.

---

## 5. The MSVC 6 C runtime — the inventory

### 5a. Library objects, band by band

**Padding is not an object boundary marker in this region.** In band 2 only 12 of the 79
inter-function gaps carry any `CC`/`NOP` at all, and the whole 16 KB of band 4 holds **four**
runs of four or more `CC`/`NOP` bytes (`0x0062b236`, `0x0062dbbb`, `0x0062e597`, `0x0062ec21`).
The `.obj` boundaries are therefore **not measurable** and the grouping below is an inference
from the CRT's source layout, not a measurement. The single 48-byte hole at `0x0062e1f9` is
not a hole: it is `crt_string_to_ld12`'s twelve-entry switch jump table.

Band 1, `[0x006243b7, 0x00625021)` — five sub-groups:

* **Floating-point init.** `crt_fp_init` is reached ONLY through the `.data` hook
  `g_crt_fp_init_hook` (`0x00651ba8`), which `_cinit` reads at `0x00626cfd` and calls before
  either initializer table runs. It fills `g_crt_cfltcvt_tab` through `crt_cfltcvt_init` and
  sets `g_crt_adjust_fdiv`.
* **Exit handlers.** `crt_onexit_init` (reached only through the `.CRT$XI` slot `0x00646314`)
  allocates the 0x80-byte table; `crt_onexit` appends and grows it; `crt_atexit` wraps that in
  C's 0/-1 contract. **167 callers**, essentially all of them per-translation-unit registrar
  stubs.
* **C++ EH and SEH.** The largest group. `crt_cxx_frame_handler` is the entry every C++ frame
  funnels through — **385 `E9` tail jumps target it, one per `EHHandler_*` stub**, which is why
  a byte scan for its address returns zero (the jumps are relative). Around it sit
  `crt_eh_call_catch_block` / `crt_eh_catch_guard_handler`, `crt_eh_call_se_translator` /
  `crt_eh_translator_guard_handler`, `crt_eh_get_try_range_to_check` (stride `0x14` =
  `sizeof(TryBlockMapEntry)`), `crt_eh_unwind_nested_frames`, `crt_eh_jump_to_continuation`,
  the two `crt_eh_call_member_function_*` thunks, and the `__local_unwind2` /
  `__abnormal_termination` / `crt_seh_unwind_collision_handler` / NLG group.
* **x87 math.** `math_acos_x87_entry` (`0x00624df0`) + `math_acos_continuation` (`0x00624e0d`);
  see §5f for what separates them from the asin pair in band 2.
* **Conversions and formatting.** `crt_atof`, `crt_atol`, `crt_atoi`, `crt_chkstk`, and
  `crt_sprintf` (**261 callers**).

Band 2, `[0x00625021, 0x00627686]`:

| sub-band | group | n |
|---|---|---|
| `[625021, 625071)` | `_vsnprintf` | 1 |
| `[625071, 6251d1)` | C++ EH array helpers — `__ehvec_dtor` / `__ehvec_ctor`, their two unwind funclets, `__ArrayUnwind` | 5 |
| `[6251d1, 625370)` | stdio — `fclose`, `fwrite`, `_fsopen`, `fopen` | 4 |
| `[625370, 625397)` | `strrchr` | 1 |
| `[625397, 6253bf)` | `srand` / `rand` | 2 |
| `[6253bf, 62558d)` | **`qsort`** + `shortsort` + `swap` | 3 |
| `[62558d, 625660)` | `_strupr`, `sscanf` | 2 |
| `[625660, 625700)` | `strncmp`, `isalpha`, `isspace` | 3 |
| `[625700, 625840)` | `strchr`, `strstr` | 2 |
| `[625840, 625b75)` | **`memmove`** (821 B) | 1 |
| `[625b75, 625cae)` | `fprintf`, `strncpy` | 2 |
| `[625cae, 625e4c)` | **`ceil` / `floor`** | 2 |
| `[625e4c, 625f60)` | `fread`, `remove` | 2 |
| `[625f60, 626180)` | **`pow`** — `_CIpow` entry, body, the is-integer helper | 3 |
| `[626180, 6262ac)` | `strncat`, **`_purecall`** | 2 |
| `[6262ac, 626443)` | `toupper` / `tolower` | 2 |
| `[626443, 626800)` | **heap** — `realloc`, `malloc`, `_nh_malloc`, `_heap_alloc`, `free` | 5 |
| `[626800, 626920)` | `__aulldiv`, `__allmul`, `vsprintf`, `__allshl` | 4 |
| `[626920, 6269eb)` | **asin** — `_CIasin` entry + body | 2 |
| `[6269eb, 626ac7)` | `time` | 1 |
| `[626ac7, 626cfd)` | `strtol` / `strtoxl` / `strtoul` | 3 |
| `[626cfd, 626dff)` | `crt0dat` — `_cinit`, `exit`, `_exit`, `doexit`, `_initterm` | 5 |
| `[626dff, 626f1a)` | `_getcwd` / `_getdcwd` / drive validation | 3 |
| `[626f1a, 627059)` | `WinMainCRTStartup`, `_amsg_exit`, `_fast_error_exit` | 3 |
| `[627059, 627150)` | **the `exception` class** (5) + **`type_info`** (2) — §3 | 7 |
| `[627150, 627230)` | `strlen`, `_SEH_prolog`, `_CxxThrowException` | 3 |
| `[627230, 627565)` | **`memcpy`** (821 B) | 1 |
| `[627565, 6275de)` | FP init — `_setdefaultprecision`, the FDIV-erratum query and self-test | 3 |
| `[6275de, 6276a0)` | `cvt` — `_forcdecpt`, `_cropzeros`, `_positive`, and `_fassign` (`0x0062769e`, §5e) | 4 |

Band 3, `[0x006276dc, 0x0062ad14]` — 86 Ghidra starts (85 functions + 1 thunk; `__except_handler3` is the one without a def) in ONE
contiguous VA cluster, 22 distinguishable library object groups:

| sub-band | group | starts |
|---|---|---|
| `[0x006276dc, 0x00627a19)` | `cvt.c` **tail** — `_cftoe` / `_cftof` / `_cftog` / `_cfltcvt` / `_shift` (its head is in band 2) | 7 |
| `0x00627a19` | `msize.c` | 1 |
| `[0x00627a7a, 0x00628280)` | 🥇 **`frame.cpp` — the whole MSVC C++ EH engine** (§5d) | 16 |
| `0x00628280` | `trnsctrl.cpp` — `_CallSettingFrame` | 1 |
| `[0x006282cc, 0x00628371)` | `hooks.cpp` — `terminate` / `unexpected` | 2 |
| `[0x00628371, 0x006286a8)` | the x87 FP error-entry and operand-class dispatch (hand-written asm), including `math_x87_dispatch_by_operand_class` / `_pair` (MSVC `_trandisp1` / `_trandisp2`) | 18 |
| `0x0062873c` | `_fltin` — string to double | 1 |
| `0x006287cd` | `isctype.c` | 1 |
| `0x00628842` | `_flsbuf.c` | 1 |
| `[0x00628957, 0x00629168)` | 🥇 **`output.c` — `_output` and its six statics** (§5e) | 7 |
| `[0x0062916c, 0x00629244)` | `exsup*.asm` — `__except_handler3`, `__seh_longjmp_unwind_4` (`0x00629229`; the `@4` decoration becomes `_4`, as in `__CallSettingFrame_12`) | 2 |
| `0x00629244` | `close.c` | 1 |
| `0x006292f7` | `_freebuf.c` | 1 |
| `[0x00629322, 0x0062942f)` | `fflush.c` and `flsall` | 4 |
| `0x0062942f` | `write.c` | 1 |
| `0x006295dc` | `_openfil.c` | 1 |
| `0x0062974c` | `_getstream.c` | 1 |
| `[0x006297d0, 0x006298c0)` | `strcat.asm` — `strcpy` and `strcat` share one copy loop | 2 |
| `[0x006298c0, 0x00629ae4)` | `a_map.c` — `__crtLCMapStringA` and its `__except` filter; the handler blocks and the post-`__try` code belong to the host (§5n) | 2 |
| `[0x00629ae4, 0x0062a595)` | 🥇 **`input.c` — the scanf engine and its four statics** (§5e) | 5 |
| `[0x0062a595, 0x0062a65f)` | `_sftbuf.c` — `_stbuf` / `_ftbuf` | 2 |
| `[0x0062a65f, 0x0062ad26)` | `handler.c` — the math error / `_matherr` / `_FPIEEE` machinery | 9 |

Band 4, `[0x0062ad26, 0x0062ec1e]` — 96 Ghidra starts (95 functions + 1 thunk) in ONE
contiguous VA cluster, ~41 library object groups:

| sub-band | group | starts |
|---|---|---|
| `[62ad26, 62ae6a)` | `frexp` and the double classifier | 3 |
| `[62ae6a, 62af00)` | the x87 status/control primitives — `_statfp`, `_clrfp`, `_ctrlfp`, `_set_statfp` | 4 |
| `0x0062af00` | `_filbuf` | 1 |
| `0x0062afd9` | `_read` | 1 |
| `0x0062b1cf` | `_dosmaperr` | 1 |
| `[62b240, 62b447)` | 🔴 the **dead `exp` object**'s x87 dispatch arms, `math_x87_integer_parity` and `math_pow_special_case_x87_wrapper` — §5i | 5 + 1 thunk |
| `[62b447, 62b5dc)` | the `pow` special-case helper and its parity test | 2 |
| `[62b5dc, 62c123)` | 🥇 the **V6** small-block heap (§5b) | 7 |
| `[62c123, 62c7f0)` | 🥇 the **V5** small-block heap (§5b) | 8 |
| `[62c7f0, 62c9dd)` | `_callnewh`, the linker-version reader, `__heap_select`, `_heap_init` | 4 |
| `0x0062c9dd` | `__loctotime_t` | 1 |
| `[62ca9f, 62cc23)` | `winxfltr.c` — `_XcptFilter` and its table search | 2 |
| `[62cc23, 62cf81)` | startup — `_wincmdln`, `_setenvp`, `_setargv`, `parse_cmdline` (§5g) | 4 |
| `0x0062cf81` | `__crtGetEnvironmentStringsA` | 1 |
| `0x0062d0b3` | `_ioinit` | 1 |
| `[62d25e, 62d3f0)` | `crt0msg.c` — the banner and the numbered message writer | 2 |
| `0x0062d3f0` | `strcmp.asm` | 1 |
| `[62d474, 62d4d8)` | `unhandld.cpp` — the C++ unhandled filter, its installer and its restorer | 3 |
| `[62d4d8, 62d63e)` | `ieee87.c` — `_control87`, `_controlfp` and the two translators | 4 |
| `[62d63e, 62dbc0)` | the 12-byte long double: `_LDBL12` helpers, `_atodbl`/`_atoflt`, `_fptostr`, `_fltout`, `_dtold` | 15 |
| `0x0062dbc0` | `memset.asm` | 1 |
| `0x0062dc18` | `_fptrap` | 1 |
| `[62dc21, 62dc71)` | the three `IsBad*Ptr` validators | 3 |
| `0x0062dc71` | `abort` | 1 |
| `0x0062dc88` | the type-driven math error reporter | 1 |
| `0x0062dd58` | `__strgtold12` + its jump table | 1 |
| `[62e229, 62e372)` | `a_str.c` — `__crtGetStringTypeA` | 1 |
| `[62e372, 62e476)` | `_lseek`, `_getbuf`, `_isatty` | 3 |
| `[62e476, 62e52f)` | `_file.c` — `__initstdio` and `__endstdio` (`crt_initstdio`, `crt_endstdio`) | 2 |
| `0x0062e52f` | `wctomb` | 1 |
| `0x0062e5a0` | `llrem.asm` | 1 |
| `[62e615, 62e82f)` | `osfinfo.c` + `commit.c` | 5 |
| `0x0062e82f` | `_sopen` | 1 |
| `0x0062eae8` | `mbtowc` | 1 |
| `0x0062ebb0` | `ungetc` | 1 |
| `0x0062ec1e` | `_matherr` — `crt_matherr_default` (§5h) | 1 |

### 5b. 🥇 BOTH small-block heaps ship, and the CONSUMERS assign every row

MSVC 6 SP5 carries two generations of small-block heap and picks one at startup.
`crt_heap_select` returns **1** on Windows 2000 or later (the Win32 heap alone), else
honours `__MSVCRT_HEAP_SELECT` / `__GLOBAL_HEAP_SELECTED` — both literals are in
`.rdata` — and otherwise falls back to `3 - (linker major < 6)`. `crt_heap_init` then
calls `crt_sbh_heap_init(0x3f8)` on mode 3 or `crt_old_sbh_new_region` on mode 2.

| | V6 (`g_crt_heap_mode == 3`) | V5 (`== 2`) |
|---|---|---|
| reserve per region | 1 MB | 4 MB |
| commit unit | a 32 KB group | a 4 KB page, 16 at a time |
| header | 0x14 bytes in an array + a 0x41c4 group map | one 0x2020-byte block, circular list |
| free-list shape | 64 size buckets + two 32-bit bitmaps | a per-page byte map, one byte per block |
| request unit | **bytes**, capped by `g_crt_sbh_threshold` | **paragraphs**, capped by `g_crt_old_sbh_threshold` |
| find / alloc / free / resize | `crt_sbh_find_block` · `_alloc_block` · `_free_block` · `_resize_block` | `crt_old_sbh_find_block` · `_alloc_block` · `_free_block` · `_resize_block` |

None of that is inferred from the bodies. **Four independent consumers make the same
three-way branch** and their arms are what assign the two families: `crt_heap_alloc`
(alloc, and it is where the byte-vs-paragraph difference is visible as `dwBytes >> 4`),
`crt_free` (find + free), `crt_realloc` (find + resize + alloc + free) and `crt_msize`
(find). The `sbh` noun is not invented either — `g_crt_heap_mode`'s registry cell calls the
field a "small-block-heap generation selector".

### 5c. 🥇 `memcpy` and `memmove` are byte-identical, and the STRING CLASS separates them

`0x00625840` and `0x00627230` are the same 821-byte function compiled into two library
objects. They differ in exactly **92 bytes, all jump-table operands**; nothing about size,
shape or position can tell them apart, and Ghidra's symbol table reports **664** for both —
it stops at the first `ret` run and misses the unrolled backward tail plus a second jump
table, which is why the two apparent 157-byte "gaps" after them are not gaps at all.

What decides it is a body-internal fact about the CALLERS:

| caller | callee | why it must be that one |
|---|---|---|
| `std_string_erase` | `0x00625840` | an erase shifts a buffer **onto itself** ⇒ `memmove` |
| `std_string_assign_substring` | `0x00627230` | assign copies into a freshly sized buffer ⇒ `memcpy` |
| `std_string_assign_buffer` | `0x00627230` | same |
| `inventory_compact_slots` | `0x00625840` | compaction slides elements down over themselves |
| `crt_fwrite` / `crt_fread` / `crt_realloc` ×4 / `crt_get_environment_strings_ansi` | `0x00627230` | all non-overlapping |

The Standard Library is *required* to use `memmove` for `erase` and may use `memcpy` for
`assign`, so the split is a fact about the C++ standard, not about this binary.

### 5d. 🥇 The whole C++ EH engine

`frame.cpp` — `__InternalCxxFrameHandler`, `FindHandler`, `FindHandlerForForeignException`,
`TypeMatch`, `__FrameUnwindToState`, `CatchIt`, `CallCatchBlock`, `BuildCatchObject`,
`AdjustPointer`, `_DestructExceptionObject`. Two constants read straight out of the bodies
pin the whole file: **`0x19930520`** (`EH_MAGIC_NUMBER1`) and **`0xe06d7363`**
(`EH_EXCEPTION_NUMBER`, `'msc' | 0xE0000000`).

The naming shape is DESCRIPTIVE rather than the MSVC internals, because that is what the
`crt_eh_*` leaf helpers below it already are (`crt_eh_get_try_range_to_check`,
`crt_eh_call_catch_block`, `crt_eh_unwind_nested_frames`, `crt_eh_jump_to_continuation`).
The `cvt.c` group goes the other way — `crt_cftoe` / `crt_cftof` / `crt_cftog` — because
three of that file's five helpers were already in the registry under their MSVC names
(`crt_cropzeros`, `crt_forcdecpt`, `crt_positive`). One family per shape, copied from the
registry rather than invented.

### 5e. 🥇 `_output` and `_input`, and the table they close between them

`crt_output` (`0x00628957`) is the printf formatting engine behind **all 261**
`crt_sprintf` callers plus `crt_vsnprintf`, `crt_fprintf` and `crt_vsprintf` — a closed
caller set the registry already named. `crt_input` (`0x00629ae4`) is the scanf engine.

Between them they close `g_crt_cfltcvt_tab`. `crt_output` calls slots **0, 1 and 3**
(`_cfltcvt`, `_cropzeros`, `_forcdecpt`) and `crt_input` calls slot **2** with the
signature `(flag, ptr, string)`, which is `_fassign` (`0x0062769e`). That slot is closed
from the other end too: `crt_atodbl` (`0x0062d9d0`) and `crt_atoflt` (`0x0062d9fd`) are
`__fassign`'s only callees, selected by its flag argument. `0x0062769e` sits 24 bytes past
the end of band 3 and belongs with the `cvt` group.

### 5f. 🥇 The math function-NAME table decodes, and it separates the x87 entry points

`g_crt_math_function_names` (`0x00651ef0`..`0x00651fc8`) is 27 `{int id; char *name}` pairs,
byte-decoded in full: exp(20) pow(29) log(26) log10(27) sinh(31) cosh(19) tanh(33) asin(14)
acos(13) atan(15) atan2(16) sqrt(5) sin(30) cos(18) tan(32) **ceil(12)** **floor(11)**
fabs(21) modf(28) ldexp(25) `_cabs`(17) `_hypot`(24) fmod(22) frexp(23) `_y0`(34) `_y1`(35)
`_yn`(36).

`crt_ceil` passes the literal `0xc` into `math_raise_fp_exception` and `crt_floor` passes
`0xb`. The table resolves those to "ceil" and "floor" — an independent artifact agreeing
with names that were assigned from bodies, and the thing that makes
`math_function_name_for_id` a measurement rather than a reading.

**The asin and acos pairs are separated by a constant each, read out of `Game.exe` rather
than out of a decompiled body:**

| VA | name | the deciding bytes |
|---|---|---|
| `0x00624df0` | `math_acos_x87_entry` | moves with its continuation |
| `0x00624e0d` | `math_acos_continuation` | `0x00624e72 fldpi` / `0x00624e76 fldz` ⇒ acos(−1) = π, acos(+1) = 0 |
| `0x00626920` | `math_asin_x87_entry` | moves with its continuation |
| `0x0062693d` | `math_asin_continuation` | `0x0062699e fld xword [0x00651c8a]` (= 1.5707963267948966 = π/2) then `fchs` under the sign bit ⇒ asin(±1) = ±π/2 |

The `|x| == 1` arm is a property of the program, where an argument order in a decompiler's
`fpatan(A, B)` rendering is a property of the decompiler. A π/0 pair can only be `acos` and a
±π/2 pair can only be `asin`.

### 5g. The startup argv / environment group

| function | what it is |
|---|---|
| `crt_get_cmdline_args` (`0x0062cc23`) | MSVC `_wincmdln`. It fetches nothing — `crt_startup` calls `GetCommandLineA` twelve instructions earlier. It SKIPS the program name and the blanks after it and returns the argument tail that `WinMain` receives |
| `crt_get_environment_strings_ansi` (`0x0062cf81`) | MSVC `__crtGetEnvironmentStringsA`. It initialises nothing; it RETURNS a fresh `malloc`ed copy of the environment block, which `crt_startup` assigns to `_aenvptr` |
| `crt_setup_argv` (`0x0062cd34`) | MSVC `_setargv`; it delegates the parsing to `crt_parse_cmdline` (`0x0062cdcd`) |
| `crt_heap_init`, `crt_exception_filter`, `crt_build_environ`, `crt_init_file_handles` | the rest of the startup group |

Two globals in this group are easy to confuse: **`g_crt_environment_block` (`0x006f97bc`)
holds `_aenvptr`, the raw BLOCK**, while MSVC's `_environ`, the argv-style array, is a
different object — `g_crt_environ` (`0x006f9798`). The command line itself is
`g_crt_command_line` (`0x006fae68`).

### 5h. `crt_matherr_default` (`0x0062ec1e`) is the user-replaceable `_matherr`

Its body is `33 c0 c3`, but the name comes from the CALLERS, not from the bytes: both pass a
`struct _exception {int type; char *name; double arg1, arg2, retval;}` and branch on the
result. Returning 0 is the C library's documented "not handled" answer, not an absent body.

### 5i. 🔴 The dead `exp` object

`[0x0062b240, 0x0062b447)` is hand-written x87 asm reached through a `jmp dword ptr [ebx]`
operand-class dispatch. The descriptor is at `.data:0x006521c0` — a flags word `0x00140100`
and one arm per operand class, in the order finite, zero, NaN, infinity:

| slot | arm | what it does |
|---|---|---|
| 0 | `math_exp_finite_argument_arm` (`0x0062b251`) | `fldl2e; fmulp`, i.e. `e**x = 2**(x·log2 e)`: `math_exp_fraction_core`, +1, the reciprocal for x < 0, `fscale`, exit through `math_x87_return_first_operand` |
| 1 | `math_x87_return_one` (`0x0062848d`) | `exp(0) = 1` |
| 2 | `math_x87_return_nan_operand` (`0x00628492`) | returns the NaN operand; quiet NaN: status byte `[ebp-0x90]` = 7; signalling NaN: = 1 and `+ g_crt_math_one_x87`, which quiets it and raises INVALID — the 7/1 pair `math_x87_classify_nan` returns |
| 3 | `math_exp_infinite_argument_arm` (`0x0062b391`) | `exp(+inf) = +inf`, `exp(-inf) = +0` |

The counted string `\x03"exp"` sits twelve bytes before the descriptor at `0x006521b4`, which
is what identifies the function. Each arm's only reference is its descriptor slot. The arms run
on their CALLER's frame — the dispatcher's status byte `[ebp-0x90]` and spill area — which is
why their rendered bodies read `unaff_EBP`. `math_exp_finite_argument_arm` is **not** interior
bytes of `crt_dosmaperr`, which ends at `0x0062b236` with ten bytes of `NOP`/`int3` padding
after it, and `math_x87_return_nan_operand` starts at the byte after `math_x87_return_one`'s
`ret`.

🔴 **The descriptor has ZERO references image-wide on all four channels**, and so does
`0x0062b415`. Nothing in the shipped binary calls `exp()`; the object survives because it
also holds `math_pow_special_case_result`, which `math_pow_continuation` does call. The
four arms are therefore capped at STRUCTURAL — there is no consumer to confirm them
against — and `thunk_math_x87_return_zero` is dead with them. So is
`math_x87_integer_parity` (`0x0062b3e1`): both of its `E8` callers, `0x0062b296` and
`0x0062b30d`, sit in this object's code outside every function (`0x0062b296` is the target of
the branch at `0x0062b249`, just before `math_exp_finite_argument_arm`).

`0x0062b350` and `0x0062b373` are code fragments of this object reached only by jumps; they
have no function of their own. In the same family, `0x0062e326` is the `__except`
continuation of `crt_getstringtype_ansi` rather than a duplicate of `0x0062e229`.

### 5j. 🥇 The FDIV-erratum object, closed from the flag the registry already named

`g_crt_adjust_fdiv` (`0x006f9754`) has exactly **two** references image-wide: the writer is
`crt_fp_init` ← `crt_detect_fdiv_erratum`, and the reader is the sequence

```
0x0062b272  cmp dword [0x6f9754], 1
0x0062b279  je  0x0062b27f
0x0062b27b  fdivrp st(1)              DE F1 - the plain instruction
0x0062b27d  jmp 0x0062b284
0x0062b27f  call 0x0062f421           math_x87_fdiv_erratum_fdivrp
```

so `math_x87_fdiv_erratum_fdivrp` is the stand-in for exactly the `fdivrp` the `je` skips. It
stores both x87 operands and calls `math_x87_fdiv_erratum_divide_core`. Writer, reader and
callee closed.

**The divide entries form a 2x2 matrix**, and the byte that separates the rows is the
`push eax` around the call into the core:

| | normal, ST(1) / ST(0) | reversed, ST(0) / ST(1) |
|---|---|---|
| internal, no EAX save — called by the memory-operand stand-ins, which save EAX themselves | `math_x87_fdiv_erratum_divide` (`0x0062f1e6`) | `math_x87_fdiv_erratum_reverse_divide` (`0x0062f1f9`) |
| in place of the instruction, EAX preserved | `math_x87_fdiv_erratum_fdivp` (`0x0062f40c`) | `math_x87_fdiv_erratum_fdivrp` (`0x0062f421`) |

The two columns differ only in the order of the two `fstp xword` stores. The memory forms
follow the same split: `math_x87_fdiv_erratum_divide_by_float` / `_by_double` / `_by_int16` /
`_by_int32` call the normal helper, and `math_x87_fdiv_erratum_reverse_divide_by_float` /
`_by_double` / `_by_int16` / `_by_int32` call the reversed one; the float and double forms
run the plain instruction when the memory operand's exponent is all ones.

**The register forms go through one dispatcher.**
`math_x87_fdiv_erratum_register_divide_dispatch` (`0x0062ed47`, MSVC `_adj_fdiv_r`) is
`sub esp,0x2c; and eax,0x3f; jmp [eax*4 + 0x0065464e]`. All 64 targets of
`g_crt_fdiv_register_form_dispatch_table` were disassembled: the selector is
`(st(i) << 3) | form`, where form 0 = `fdiv st,st(i)`, 2 = `fdivr st,st(i)`, 4 / 5 / 6 / 7 =
`fdiv` / `fdivp` / `fdivr` / `fdivrp st(i),st`, and forms 1 and 3 execute `int 6`. The arms
hold all 46 `E8` calls into `math_x87_fdiv_erratum_divide_core`. The core itself decides
whether the hardware is safe: it indexes `g_crt_fdiv_flawed_divisor_nibbles` (`0x006545f0`,
non-zero only at 1, 4, 7, 0xa and 0xd — the five flawed divisor patterns) with the divisor's
top fraction nibble and runs the plain `fdivp` when the byte is zero.

The object's tail corroborates the identification independently, because it is the
canonical `_adj_*` set and nothing else has that shape:

```
0x0062f9a9  call 0x62f63c ; ret      math_x87_fdiv_erratum_fprem     (_adj_fprem)
0x0062f9af  call 0x62f8f4 ; ret      math_x87_fdiv_erratum_fprem1    (_adj_fprem1)
0x0062f9b5  fpatan       ; ret       math_x87_fdiv_erratum_fpatan    a bare PASS-THROUGH
0x0062f9b8  fptan        ; ret       math_x87_fdiv_erratum_fptan     a bare PASS-THROUGH
```

A pass-through only makes sense as an entry in an erratum-adjustment table — there is no
other reason to wrap a single x87 instruction in a `call`.

**`math_x87_fdiv_erratum_fprem_core` and its `fprem1` sibling are separated by three bytes.**
The two 518-byte functions are byte-identical over their first 0x1e9 bytes with exactly
three differences, and the decisive one is `D9 F8` = `fprem` at `0x0062f5b8` against
`D9 F5` = `fprem1` at `0x0062f870`. Each core sits behind a body that routes by the
divisor's exponent — `math_x87_fdiv_erratum_fprem_scale_denormal` and
`math_x87_fdiv_erratum_fprem1_scale_denormal`: a normal divisor goes to the core, a zero one
runs the hardware instruction, and a denormal one scales both operands by 2^64
(`g_crt_fprem_scale_2p64`) first. The `fprem1` body's zero-divisor arm at `0x0062f9a2` executes `fprem`
(`D9 F8`), not `fprem1`; with a zero divisor both give an invalid operation.

🔴 **The whole object is DEAD.** Its only external caller is `0x0062b27f`, inside
`math_exp_finite_argument_arm`, arm 0 of the `exp` operand-class descriptor at
`.data:0x006521c0`, which §5i measures as having zero references on all four channels. The game was not compiled with `/QIfdiv`. Every entry of
the object that is not called from inside it has 0 references on all four channels.

### 5k. 🥇 `_purecall` — a 9-byte function with 36 vtable slots and no callers

`crt_purecall` (`0x006262a3`) is `push 0x19; call __amsg_exit`. **0x19 = 25 = `_RT_PUREVIRT`**
in MSVC's `rterr.h`, and the same table is confirmed 0x2000 bytes away by `crt_startup`
passing **0x1c = 28 = `_RT_HEAPINIT`** to `crt_fast_error_exit` — two constants from one
enum, each landing exactly where that enum says it should.

It has **zero `E8` callers and thirty-six `.rdata` references**. Those 36 are 36 abstract
classes' vtable slots, which makes this address a **census tool for abstract classes in
`Game.exe`** — a cheap, closed-set way to enumerate them that no name search can match.

### 5l. `srand(time(NULL))`, located

`game_init` runs `crt_time(0)` and feeds the result straight to `crt_srand`. `rand`'s seed
`g_crt_rand_seed` ships as **1** — the C standard's initial value, i.e. the value an
unseeded `rand` would have used — so that one pair is what makes every `rand_range` /
`rand_float` draw in the game differ between runs.

### 5m. Two byte-identical functions that were NOT folded

`crt_eh_call_member_function_00624a4b` and `crt_eh_call_member_function_00624a52` are the
same 7 bytes — `58 59 87 04 24 ff e0` — and both survive. The bytes contain no relative
branch (`ff e0` is `jmp eax`), so the duplication is real: a clean instance of the
non-COMDAT population that survives `/OPT:ICF`.

### 5n. `crt_lcmapstring_ansi_except_filter` (`0x00629a84`) is a FILTER, not a funclet

Its single dataref is `.rdata:0x0063f19c`, an `__except_handler3` SCOPE TABLE
`{prev, filter, handler}`, byte-decoded as `{-1, 0x00629a84, 0x00629a88}` — this address is
the FILTER. Its owner is `crt_lcmapstring_ansi`, which pushes the table's base in its own
prologue. The two HANDLER funclets of the same owner, `0x006299d4` and `0x00629a88`, are
paired with it in the scope table at `.rdata:0x0063f190` (§10).

---

## 6. PC vs PS2

The PS2 ships a **different Standard Library** (Metrowerks/SN) and a **different C runtime**
(newlib), so most of this region has no counterpart by construction rather than by omission.

| finding | evidence |
|---|---|
| 🔴 `std::out_of_range` **does not exist** in the PS2 build | VA-band enumeration of the PS2 symbol table with the class column **ignored**, not a class-name search |
| The PS2 has only 5 stdexcept symbols | `std::exception` {dtor, what}, `std::length_error` {dtor}, `std::logic_error` {dtor, what} |
| No `_Xlen`, no `_Xran`, no `_Freeze`, no `erase`, no `assign(const string&, off, n)` | the Metrowerks string routes all of it through `change_size` (916 B) |
| **No site in this region is "inlined on PC where the PS2 calls a real function"** | there is no PS2 function to have been inlined *from*; asserting one would be a terminal tag without evidence |
| The whole C runtime block is MSVC-specific | MIPS has no x87 and no stack probing; GCC's C++ EH ABI shares nothing with MSVC's |
| C-library counterparts are deliberately **not** recorded as twins | established practice — `crt_atof`, `crt_atol`, `crt_atoi`, `crt_fopen`, `crt_sscanf` all carry `-`. The newlib addresses are `atof 0x003d0850`, `atoi 0x003d0868`, `atol 0x003d0890`, `sprintf 0x003d6be8` |

**Every row in bands 2 to 5 carries `-`, and that is a measurement, not an omission.**
Enumerating the PS2 symbol table with the CLASS COLUMN IGNORED does return `printf`,
`sprintf`, `sscanf`, `vfprintf`, `vsprintf`, `strcpy`, `strcat`, `strlen`, `memcpy`,
`memmove`, `memset`, `strcmp`, `qsort`, `atof`, `atol`, `ceil`, `floor`, `fflush`, `abort`,
`ungetc`, `lseek`, `isatty`, `malloc`, `free`, `realloc`, `_read_r` and `_mbtowc_r` — the PS2
links **newlib**. Two independent implementations of one standard API cannot be body-diffed,
so a C-library counterpart is not a twin; that is an artifact-level cap, not a pending row.
It returns **zero** for every MSVC-internal member: `_filbuf`, `_getbuf`, `_dosmaperr`,
`_control87`, `_setargv`, `_setenvp`, `_wincmdln`, `_XcptFilter`, `__crtGetStringTypeA`,
`__sbh_*`, `_atodbl`, `_fltout`, `_fptostr`, `__strgtold12`, `wctomb`, `_matherr`, `frexp`,
`_output`, `_flsbuf`, `_input`, `__InternalCxxFrameHandler`.

Band 5 adds two reasons of its own:

* **Band A is x87-specific.** MIPS has no 80-bit FPU stack, so the whole `_adj_*` concept has
  no counterpart to have.
* **Band F is a different library, not a different spelling.** The PS2 links the **standard
  templated `<fstream>`** — 70 symbols of `std::basic_filebuf<c,Q23std14char_traits<c>>` and
  friends, with a `__sinit_iostream.cpp` of its own. The PC links MSVC's **classic
  pre-standard** iostream: non-templated, virtual-base `ios`, `ios::nocreate` (a flag the
  standard library does not have), `filebuf::openprot` = 0644. There is nothing to diff.

**Zero inlining banners anywhere in this region, and that too is a measurement.** There is no
PS2 function for MSVC to have inlined *from*, so marking one would be a terminal tag with no
evidence.

### 6a. The four STRUCTURAL twins

| PC | PS2 | why capped at STRUCTURAL |
|---|---|---|
| `std_string_assign_buffer` | `assign__Q23std59basic_string<>FPCcUi` | same `(const char *, unsigned)` signature and the same three statements in the same order; the PC sets `_Len` and the NUL itself where the PS2's `change_size` does it internally |
| `std_logic_error_what` | `what__Q23std11logic_errorCFv` | same method, different layout — the PC returns `[this+0x10]` with a nil fallback, the PS2 `[this+4]` with none |
| `std_logic_error_scalar_deleting_dtor` | `__dt__Q23std11logic_errorFv` | paired on the flagged-delete signature (§2d) |
| `std_length_error_scalar_deleting_dtor` | `__dt__Q23std12length_errorFv` | same |

The cap is the precedent `std_string_dtor` already set: two different Standard Library
implementations make a statement-level diff meaningless even when both bodies are read.

---

## 7. Naming policy for this region

**Names that Ghidra assigned in its default analysis are KEPT** and given docblocks and
registry rows instead — user decision. That covers `operator_new`, `__ftol`,
`__global_unwind2`, `__local_unwind2`, `__abnormal_termination`, `__NLG_Notify1`, `_strcmp`,
`_memset`, `exception::~exception` and all ten import thunks.

⚠️ The policy does not reach `crt_atexit` and `crt_onexit`: Ghidra's default analysis
has **zero** hits for either MSVC spelling (`atexit`, `_onexit`), so neither name comes
from Ghidra, and both follow the `crt_` convention of the rest of the region.

### 7a. Why the `ls3df.dll` thunks keep `S_vector::RotateByMatrix`

Those 29 names are verbatim demanglings of strings physically present in `Game.exe`'s import
directory — e.g. `?RotateByMatrix@S_vector@@QBG?AU1@ABUS_matrix@@@Z`. Four reasons not to
rename them:

1. **A rename is impossible to earn.** The bodies are in `ls3df.dll`, so a behaviour-derived
   name could never be body-diffed, and any replacement would be a transliteration of the
   export name.
2. It would **collide semantically with the 30-member in-exe `vec3_*` family**, which is
   `Game.exe`'s *own* inlined LS3D math. One prefix for two different things is the
   `C_car` / `C_Vehicle` failure again.
3. The `::` spelling is already a working convention: exactly 30 of 6759 definitions carry
   `::`, and 29 of them are this population. The punctuation *is* the marker for "6-byte
   import thunk, body in a DLL".
4. `basic/artifacts/dll.md` §4 is built on those names, and the mangling additionally records
   the calling convention (`QAG`/`QBG` = public `__stdcall`).

---

## 8. Data symbols

| address | name | what it is |
|---|---|---|
| `0x0063eaf8` | `g_std_string_nil` | Dinkumware `basic_string<char>::_Nil` — a lone NUL whose **preceding share-count byte is pinned at 0**, which is why `std_string_freeze` never unshares or frees it. ⚠️ Not `g_empty_string` (`0x00655214`), the game's own null-name fallback |
| `0x0063ef80` | `g_crt_ehvec_dtor_scope_table` | `{-1, 0, 0x006250d9}` |
| `0x0063ef90` | `g_crt_eh_array_unwind_scope_table` | `{-1, 0x0062512d, 0x00625137}` — both inside its own body |
| `0x0063efa0` | `g_crt_ehvec_ctor_scope_table` | `{-1, 0, 0x006251b9}` |
| `0x0063efb8` | `g_crt_startup_scope_table` | `{-1, 0x00626ff1, 0x00627005}` |
| `0x0063efc8` | `g_exception_vt` | the 2-slot vtable of §3 |
| `0x0063eff0` | `g_crt_cxx_exception_record` | the `EXCEPTION_RECORD` template: `0xe06d7363`, NONCONTINUABLE, 3 params, magic `0x19930520` |
| `0x006248a7` | `crt_atexit_noop_dtor_006248a7` | a one-byte `ret`; see §8b |
| `0x00646000` / `0x0064630c` | `g_crt_xc_a` / `_z` | MSVC `__xc_a` / `__xc_z`, the `.CRT$XC` bounds (C++ initializers); the second `_initterm` in `crt_call_init_fns` |
| `0x00646310` / `0x00646324` | `g_crt_xi_a` / `_z` | MSVC `__xi_a` / `__xi_z`, the `.CRT$XI` bounds (C initializers); the first `_initterm` in `crt_call_init_fns` |
| `0x00646328` / `0x00646330` | `g_crt_xp_a` / `_z` | MSVC `__xp_a` / `__xp_z`, the `.CRT$XP` bounds (pre-terminators); the first `_initterm` in `crt_doexit` |
| `0x00646334` / `0x0064633c` | `g_crt_xt_a` / `_z` | MSVC `__xt_a` / `__xt_z`, the `.CRT$XT` bounds (terminators); the second `_initterm` in `crt_doexit` |
| `0x00651ba8` | `g_crt_fp_init_hook` | holds `crt_fp_init`; read by `_cinit`. Row-only: no Ghidra symbol |
| `0x00651bb4` | `g_crt_nlg_destination` | MSVC `_NLG_Destination`; first dword is the signature **`0x19930520`**. Row-only |
| `0x00651be0` | `g_crt_rand_seed` | the LCG state; written by `crt_srand`, advanced by `rand`. Ships as **1** |
| `0x00651bf0` | `g_crt_x87_cw_round_up` | `0x1b3f`, bits 11:10 = 10b = toward +∞. Read only by `crt_ceil` |
| `0x00651bf4` | `g_crt_x87_cw_round_down` | `0x173f`, bits 11:10 = 01b = toward −∞. Read only by `crt_floor` |
| `0x00651c50` | `g_crt_cfltcvt_tab` | the 6-entry float→text conversion vector; Ghidra split it into six symbols, all renamed with index suffixes |
| `0x00651c80` | `g_crt_math_indefinite_x87` | the value pre-loaded on every math domain error |
| `0x00651c8a` | `g_crt_half_pi_x87` | 80-bit π/2 — the constant that separates the asin pair (§5f) |
| `0x00651cb4` | `g_crt_pctype` | MSVC `_pctype`, 2 bytes per character. 37 references |
| `0x00651ec0` | `g_crt_mb_cur_max` | the single-byte / MBCS branch selector at every ctype site. Value 1 |
| `0x00651ec4` | `g_crt_decimal_point` | `'.'`; written by `crt_forcdecpt`, compared by `crt_cropzeros` |
| `0x00651ef0` | `g_crt_math_function_names` | the 27-pair math name table decoded in §5f |
| `0x00654208` | `g_crt_xcpt_action_table` | MSVC `_XcptActTab`, typed `S_crt_xcpt_action[10]`: ten `{exception code, signal, action}` records, bounded by `g_crt_xcpt_action_count` (= 10); searched by `crt_xcpt_action_lookup` / `_by_signal` |
| `0x00654360` | `g_crt_iob_stdin` | `_iob[0]`, typed as one `FILE` (0x20); `crt_initstdio` resets the `_file` of it and the next two streams |
| `0x006545f0` | `g_crt_fdiv_flawed_divisor_nibbles` | 16 bytes, non-zero only at 1, 4, 7, 0xa, 0xd: the five Pentium-FDIV-flaw divisor patterns (§5j) |
| `0x0065464e` | `g_crt_fdiv_register_form_dispatch_table` | the 64 targets of `math_x87_fdiv_erratum_register_divide_dispatch`, indexed `(st(i) << 3) \| form` (§5j). Row-only: the address is unaligned and only ever a `jmp` operand, so it has no Ghidra symbol |
| `0x00654834` | `g_crt_days_before_month` | the base-minus-one-element pointer the code adds `month*4` to with a **1-based** month; see §8c |
| `0x006f9750` | `g_crt_math_skip_error_handler` | see §8a |
| `0x006f9754` | `g_crt_adjust_fdiv` | 🥇 MSVC `_adjust_fdiv`, the **Pentium FDIV-erratum flag** (§5j) |
| `0x006f9758` | `g_crt_time_dstflag_cache` | `crt_time`'s cached daylight-saving flag (-1 / 1 / 0), reused while the cached minute matches |
| `0x006f9760` | `g_crt_time_gmt_cache` | `crt_time`'s cached `SYSTEMTIME` (16 bytes), compared minute, hour, day, month, year against the current UTC time |
| `0x006f9770` | `g_crt_errno` | `ERANGE` from `crt_strtoxl`, `EACCES`/`ENOMEM`/`ERANGE` from `crt_getdcwd` |
| `0x006f9774` | `g_crt_doserrno` | `ERROR_INVALID_DRIVE`, written beside `EACCES` |
| `0x006f977c` | `g_crt_osver` | MSVC `_osver` = `GetVersion() >> 16`, set by `crt_startup` |
| `0x006f9780` | `g_crt_winver` | MSVC `_winver` = `g_crt_winmajor * 0x100 + g_crt_winminor` |
| `0x006f9784` | `g_crt_winmajor` | MSVC `_winmajor` = `GetVersion() & 0xff` |
| `0x006f9788` | `g_crt_winminor` | MSVC `_winminor` = `GetVersion() >> 8 & 0xff` |
| `0x006f9798` | `g_crt_environ` | MSVC `_environ` — the argv-style array (§5g) |
| `0x006f97bc` | `g_crt_environment_block` | `_aenvptr` — the raw block (§5g) |
| `0x006f97e0` | `g_crt_se_translator` | the `_set_se_translator` callback |
| `0x006f981c` | `g_crt_lc_codepage` | the default code page `crt_lcmapstring_ansi` substitutes when the caller passes 0 |
| `0x006f9834` | `g_crt_new_mode` | MSVC `_newmode`; gates the new-handler retry loop |
| `0x006f9984` | `g_crt_buffered_stream_count` | MSVC `_cflush`, incremented by `crt_openfile`, `crt_setup_temp_buffer` and `crt_getbuf`, read nowhere |
| `0x006fad40` | `g_crt_osfhandle_blocks` | MSVC `__pioinfo`, typed `pointer[64]`: block `fd >> 5`, record `(fd & 0x1f) * 8`; `crt_init_file_handles` clamps inherited handles at 0x800 = 64 blocks |
| `0x006fae44` | `g_crt_heap_handle` | argument 1 of every `Heap*` call in the runtime |
| `0x006fae48` | `g_crt_heap_mode` | the small-block-heap generation selector (2 / 3 / other) |
| `0x006fae64` | `g_crt_sbh_threshold` | upper size bound of the mode-3 small-block path |
| `0x006fae68` | `g_crt_command_line` | the full command line `GetCommandLineA` returns (§5g) |
| `0x006fae74` / `0x006fae78` | `g_crt_onexit_end` / `_begin` | the exit-handler table bounds |

Further named in this region: `g_crt_app_type`, `g_crt_error_mode` (MSVC `__error_mode`: 0
default, 1 stderr, 2 message box), `g_crt_old_sbh_threshold` (the mode-2 heap threshold at
`0x006541fc`), `g_crt_prev_unhandled_exception_filter`, `g_crt_terminate_handler`,
`g_crt_terminate_scope_table`, `g_crt_fdiv_exponent_class_table` (§13),
`g_crt_mbcs_case_ranges` and its two index-suffixed
siblings.

### 8a. `g_crt_math_skip_error_handler` is never written

A little-endian scan of the whole image finds **six references and every one is
`83 3d 50 97 6f 00` = `cmp dword [0x006f9750], imm8`** — reads only; through none of the four
reference channels is there a writer. As shipped the flag is permanently 0, so the asin /
acos / pow error path **always** runs and the raw-result shortcut is dead. The name is a
deliberate NARROWING of "fast mode", which would assert an intent nothing in the binary
backs; "skip error handler" is exactly what the six branches do.

### 8b. Not every `atexit` registrar passes the same no-op destructor

The 109 per-translation-unit `atexit` registrars pass `crt_atexit_noop_dtor_0060b700` — **all
but two**. `crt_atexit_register_0062489b` passes its own local one-byte `ret` at
`0x006248a7`, and `screen_fade_static_dtor` is the other exception.

### 8c. `g_crt_days_before_month` sits one element BEFORE the array it names

`0x00654834` holds **365** — the last entry of the LEAP table — and is the
base-minus-one-element pointer the code adds `month*4` to with a **1-based** month. The
arrays themselves begin at `0x00654804` (leap) and `0x00654838` (common), and both have
their own rows. The symbol Ghidra offers sits on element `[-1]` because that is where the
reference arithmetic starts; `0x00654834` genuinely IS the base pointer the code uses.

### 8d. 🥇 Four rows assigned by decoding data rather than by reading code

* **`math_fpclass_double`** matches all **ten** `_FPCLASS_*` values byte for byte — `0x200`
  PINF, `0x100` PN, `0x80` PD, `0x40` PZ, `0x20` NZ, `0x10` ND, `8` NN, `4` NINF, `2` QNAN,
  `1` SNAN. A complete enum match, not a shape argument.
* **`crt_dst_rule_to_yearday`'s two day tables** were byte-read and separated by their
  February step: `0x00654804` steps 31, **29**, 31, 30 … (leap) and `0x00654838` steps 31,
  **28**, 31, 30 … (common).
* **`crt_setmbcp`'s code-page table** at `0x00654878` decodes to four 0x30-byte records —
  **932, 936, 949, 950** — each with its case-conversion triple and four lead/trail byte
  range lists, bounded by the literal `0x654968` in the function's own loop.
* **`g_crt_mbcs_case_ranges`** decodes to the fullwidth case ranges of the active code page:
  CP932 `{0x8260, 0x8279}` delta `0x21` (Shift-JIS fullwidth `A`–`Z` → `a`–`z`), CP936/949
  `{0xa3c1, 0xa3da}` delta `0x20`, and CP950 `{0xa2cf, 0xa2e4}` delta `0x1a` **plus a second
  range** `{0xa2e5, 0xa2e8}` delta `0x5b` — which is what proves the field is an array of
  `{lo, hi, delta}` word triples, i.e. MSVC's `__mbulinfo[6]`.

---

## 9. The function starts Ghidra did not create

Every address below is real code that runs (or would run, on a buggy Pentium), and Ghidra's own
analysis makes no function start at any of them: most have no caller at all, and the rest are
reached only through a `.CRT` table or from other start-less code. Each start was made by a
dedicated function-start pass; the definitions carry the provenance line `recovered by a Ghidra
function-start pass` and all of them are named. Two of them,
`math_x87_fdiv_erratum_register_divide_dispatch` and `math_x87_fdiv_erratum_reverse_divide`,
are the only entry points of code with 46 and 8 dependents respectively.

| VA | name | what it is | how it is reached |
|---|---|---|---|
| `0x00627ffc` | `crt_eh_rethrow_exception_filter` | MSVC `ExFilterRethrow`: 1 for an MSVC C++ exception (`0xE06D7363`, 3 parameters, `0x19930520`) with a NULL ThrowInfo — a bare `throw;` — else 0 | **1 `E8` caller**, the `__except` filter funclet of `crt_eh_run_catch_block` |
| `0x00628380` | `math_x87_dispatch_by_operand_class` | MSVC `_trandisp1`: `fxam` ST(0), fold the class through `0x00651c9d`, `jmp [edx+0x10+off]` into the caller's descriptor | 0 references on all four channels |
| `0x006283e7` | `math_x87_dispatch_by_operand_class_pair` | MSVC `_trandisp2`: the same for ST(0) and ST(1) | 0 references on all four channels |
| `0x0062b3e1` | `math_x87_integer_parity` | 0 = not an integer, 1 = odd, 2 = even; the x87 sibling of `math_double_integer_parity` | **2 `E8` callers**, both inside arm 0 of the dead `exp` object (§5i) |
| `0x0062b415` | `math_pow_special_case_x87_wrapper` | the x87 calling wrapper around `math_pow_special_case_result` ¹ | 0 references on all four channels |
| `0x0062d4cb` | `crt_cxx_restore_unhandled_exception_filter` | 13 bytes: `SetUnhandledExceptionFilter(g_crt_prev_unhandled_exception_filter)` — the un-install half of `crt_cxx_install_unhandled_exception_filter` | the `.CRT$XT` terminator slot `0x00646338` |
| `0x0062e51b` | `crt_endstdio` | MSVC `__endstdio`: `crt_flushall()`, then, behind `0x006f97b0`, a tail jump to `crt_fcloseall` ² | the `.CRT$XP` pre-terminator slot `0x0064632c` |
| `0x0062ed47` | `math_x87_fdiv_erratum_register_divide_dispatch` | MSVC `_adj_fdiv_r`, the 64-arm register-form dispatcher (§5j) | 0 references on all four channels; the byte before it is a `ret` |
| `0x0062f1f9` | `math_x87_fdiv_erratum_reverse_divide` | the internal reversed divide, the mirror of `math_x87_fdiv_erratum_divide` | **8 `E8` callers**, two in each `fdivr` memory-operand stand-in |
| `0x0062f2a4` | `math_x87_fdiv_erratum_divide_by_int16` | safe `fdiv` by a 16-bit integer memory operand (`fild word`, `ret 4`) | 0 references on all four channels |
| `0x0062f2d8` | `math_x87_fdiv_erratum_divide_by_int32` | safe `fdiv` by a 32-bit integer memory operand (`fild dword`, `ret 4`) | 0 references on all four channels |
| `0x0062f30c` | `math_x87_fdiv_erratum_reverse_divide_by_float` | safe `fdivr` by a float memory operand (`ret 4`) | 0 references on all four channels |
| `0x0062f358` | `math_x87_fdiv_erratum_reverse_divide_by_double` | safe `fdivr` by a double memory operand (`ret 8`) | 0 references on all four channels |
| `0x0062f3a4` | `math_x87_fdiv_erratum_reverse_divide_by_int16` | safe `fdivr` by a 16-bit integer memory operand | 0 references on all four channels |
| `0x0062f3d8` | `math_x87_fdiv_erratum_reverse_divide_by_int32` | safe `fdivr` by a 32-bit integer memory operand | 0 references on all four channels |
| `0x0062f40c` | `math_x87_fdiv_erratum_fdivp` | MSVC `_adj_fdivp_sti_st`: the EAX-preserving stand-in for `fdivp st(1),st` | 0 references on all four channels |
| `0x0062f63c` | `math_x87_fdiv_erratum_fprem_scale_denormal` | the `fprem` body that routes by the divisor's exponent (§5j) | 1 `E8` caller, `math_x87_fdiv_erratum_fprem` |
| `0x0062f8f4` | `math_x87_fdiv_erratum_fprem1_scale_denormal` | the `fprem1` body | 1 `E8` caller, `math_x87_fdiv_erratum_fprem1` |
| `0x0062f9a9` | `math_x87_fdiv_erratum_fprem` | `_adj_fprem` — `call 0x62f63c; ret` | 0 references on all four channels |
| `0x0062f9af` | `math_x87_fdiv_erratum_fprem1` | `_adj_fprem1` — `call 0x62f8f4; ret` | 0 references on all four channels |
| `0x0062f9b5` | `math_x87_fdiv_erratum_fpatan` | `_adj_fpatan` — `fpatan; ret`, a pass-through | 0 references on all four channels |
| `0x0062f9b8` | `math_x87_fdiv_erratum_fptan` | `_adj_fptan` — `fptan; ret`, a pass-through | 0 references on all four channels |

¹ Its earlier start `0x0062b416` was an Aggressive-Instruction-Finder phantom one byte in,
which made every frame offset in the rendered body 4 too small; it is recorded in the phantom
register.

² `crt_flushall`'s caller is `crt_endstdio`, **not** `crt_initstdio`
(`0x0062e476`). The two are separated by a fact neither body shows: one address sits in the
`.CRT$XI` initializer table and the other in `.CRT$XP`. A VA-to-containing-function lookup
attributes the call to the previous start, which is not the caller here.

---

## 10. Identified, NOT named

A grade implies a name, so these are listed rather than graded.

| VA | what it is | why no name |
|---|---|---|
| `0x006299d4`, `0x00629a88` | the two `__except` HANDLER funclets of `crt_lcmapstring_ansi`, paired with the filter at `0x00629a84` in the scope table at `.rdata:0x0063f190` (§5n); `0x00629a88` lies inside the host's Ghidra function body, `0x006299d4` in no function | code of their owner, reached only through the scope table — not functions, so no def and no row |
| `0x00627e39`, `0x00627fa2`, `0x00627fb0` | `__except` continuations that run on their CALLER's frame | no def anywhere, and they are not entry points of their own |
| `0x006521b4` | the counted string `"exp"` | no Ghidra symbol and zero references on all four channels |
| `0x006521c0` | the `exp` operand-class dispatch descriptor (§5i) | same |
| `0x006f9988` | OR'd into every new stream's `_flag` (MSVC `_commode`) | proposed `g_crt_commit_mode`, CANDIDATE — only one consumer was read |
| `0x006f97b0` | the flag `crt_endstdio` tests before its tail jump; also written by `crt_doexit` as its `retcaller` argument | one of three `crt_doexit` state flags (`0x006f97b0` / `_b4` / `_b8`), read from ONE side only, so naming the trio would rest on a third of a measurement |
| `0x00651c18` | `PTR___exit_00651c18`, the pointer beside the initializer tables | left under its Ghidra name; it was not examined when the four `.CRT` bound pairs were named (§8) |
| `0x006f9a80` | the `ios` state word of `g_crashreport_map_file`, reached as `(&this)[*(int *)(obj + 4)]` through the virtual-base table | a FIELD of an object that already has a row, not a symbol of its own |
| `0x00654644` | the 80-bit constant whose last two bytes the operand-class jump table overlaps (`00 00 00 00 00 00 ff ff fe 7f`, the largest finite long double) | read only by the `fprem` cores through a `sub eax, 0x654644` index computation, so it has no Ghidra symbol and no consumer that names it |
| `0x00654908`, `0x00654a68` | the two 0x54-stride tables of pre-computed powers of ten that `crt_ld12_multiply_by_power_of_ten` walks (positive and negative exponents) | reached only as `base + n*0x54` index computations, so neither has a symbol; their extent and stride are measured, their contents are not decoded |

---

## 11. Phantoms — three shapes, and the blind spots

Nine Aggressive-Instruction-Finder phantoms were purged from this region: **two** of one
shape and **seven** of another. Both shapes are a def whose RENDERED BODY duplicates its
neighbour's, because Ghidra put the start somewhere only a fall-through reaches.

| phantom | real host start | what the rendered body dropped |
|---|---|---|
| `0x006297de` | two bytes of `mov edi, edi` alignment filler between `crt_strcpy` and `crt_strcat`; it falls through, so Ghidra rendered `crt_strcat`'s whole body a second time | a 75-line phantom record |
| `0x0062f203` | ten bytes INSIDE `math_x87_fdiv_erratum_reverse_divide` (`0x0062f1f9`) | 🥇 **the finding is its host**: 8 `E8` callers and no Ghidra start of its own (§9) |
| `0x0062ede1` | `0x0062edda` (a jump-table arm of `math_x87_fdiv_erratum_register_divide_dispatch`) | both `fstp xword` argument spills |
| `0x0062ee05` | `0x0062edfe` (another arm) | both spills |
| `0x0062f417` | `math_x87_fdiv_erratum_fdivp` (`0x0062f40c`) | the `push eax` frame and both spills |
| `0x0062f654` | `math_x87_fdiv_erratum_fprem_scale_denormal` (`0x0062f63c`) | the frame, the spills and the exponent gate |
| `0x0062f6dd` | `0x0062f63c` | ditto (the far arm) |
| `0x0062f90f` | `math_x87_fdiv_erratum_fprem1_scale_denormal` (`0x0062f8f4`) | ditto |
| `0x0062f998` | `0x0062f8f4` | ditto (the far arm) |

All nine: **0 `E8`, 0 `E9`, 0 datarefs, 0 immediate-operand references** — every channel
measured in `Game.exe`. Nothing live was lost: the entire FDIV-erratum object is unreachable
in the shipped binary (§5j).

**Three standing detection methods all report these ranges clean**, and each misses them for
its own reason:

| method | why it misses them |
|---|---|
| the `unaff_*` / `in_EAX` signature of a truncated body | every live value in the FDIV object is on the x87 stack, so no register artefact appears |
| the unreachable-*epilogue* prover | it requires something to branch to the address; these are plain fall-through |
| the duplicate-body hash | every one of these bodies contains a `call rel32`, which makes the hash meaningless for them |

What found them was reading the bytes AT the VA after a four-channel reference census came
back zero.

**The epilogue prover's other hits in this region are FALSE POSITIVES, all of one kind.**
`math_x87_restore_control_word` (`0x0062864e`), `__math_exit` (`0x0062865b`),
`math_x87_check_result_range` (`0x00628699`) and `crt_abort` (`0x0062dc71`) are reached by
**tail jump** (`E9`), never by `call` — from the acos, asin and pow continuations, and from
`crt_terminate` at `0x0062831d`. A BRANCHED_INTO criterion cannot tell an inter-function tail
call from an intra-function branch. The registry itself already said so for `crt_abort`:
`crt_terminate`'s cell reads *"calls `g_crt_terminate_handler` then abort"*. A second blind
spot surfaces with `__math_exit`: its decomp state is reported ABSENT when it is plainly
present, because the state probe keys on a `FUN_` name and this function carries a Ghidra
Function-ID name.

---

## 12. Related

`basic/artifacts/dll.md` · `structs/app_bootstrap.md` (the other region-shaped doc) ·
`structs/crash_report.md` (the game translation unit inside band 5) ·
`structs/string.md` (the game's own refcounted string class, a different one) ·
`structs/i3d_cache.md` (the three `crt_getcwd` callers) · `structs/check_seg.md` §7 and
`structs/chunk.md` §6 (the `folded_vector_ctor_at_0x04_0044a120` fold) ·
`ghidra/Game-ghidra.exe.functions.md` (the registry rows for everything here).

---

## 13. Open items

| item | state |
|---|---|
| `0x00651bac` and `0x00651bb0` | two CRT float hooks pre-set to `stub_noop_cdecl_006248fb`. An image-wide little-endian scan for **both slot addresses returns zero reads**, so what they would have done cannot be decided from this binary. NAMING-DEFERRED |
| the shared guard byte `0x006f9ae0` | 109 static-init guards all test bit 0 of one byte, and `crt_static_init_guard_006242b0` and `crt_static_init_guard_00624865` sit in adjacent `.CRT$XCU` slots guarding the same bit. Recorded, not chased |
| `crt_strtoul`'s only caller is `game_collect_scream_area` | a function named for an audio/AI concern has no obvious reason to parse an unsigned number; that caller is a misnomer suspect and its body was not read |
| `std_string_ctor_formatted` formats through the UNBOUNDED `crt_vsprintf` into a 1000-byte stack buffer | its sibling `string_format` uses the capped `crt_vsnprintf(buf, 0x400, …)`. Measured, not chased: whether any caller can overflow it was not traced |
| `crt_startup`'s docblock records only where the name came from | the same holds for `rand`'s neighbourhood in band 2. A documentation gap, not a rename |
| `g_crt_math_skip_error_handler` has no writer (§8a) | if it matters, the next lever is a displacement byte-scan |
| `rand` (`0x006253a1`) is the one `crt_`-less member of its band | its name and docblock are correct, and moving it would touch 101 callsites for cosmetics — recorded, not done |
| `0x006f97b0` is read from one side only | the other two flags of the `crt_doexit` trio were not read, so the group is still unnamed (§10) |
| `g_crt_fdiv_exponent_class_table` (`0x0065460c`) is byte-identical to `g_crt_fdiv_flawed_divisor_nibbles` | the two `fprem` cores index it with `(high word & 0x7800) >> 11`. Whether that field is the exponent, as the name says, or the divisor's top fraction nibble — which would make it the same flawed-divisor test and the name wrong — was not measured |
| `0x00654908` / `0x00654a68` are undecoded (§10) | decoding them would let `crt_ld12_multiply_by_power_of_ten` be verified against a shipped value rather than read |
| `g_crt_mbcs_case_ranges_dw1` / `_dw2` carry index suffixes, not field names | Ghidra split the six-WORD `__mbulinfo` array on dword boundaries that cut across its fields. A Ghidra retype of that address to `ushort[6]` would let all three take real field names |
