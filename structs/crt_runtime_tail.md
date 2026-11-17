# The statically-linked runtime tail

**Created** 2026-09-04 18:07 +0200 · **Modified** 2026-11-17 14:40 +0100

A **linker region, not a translation unit and not a class**: the tail of `.text` where the linker parked the objects that came out of
`.lib` files rather than out of Mafia's own source, up to the first unwind funclet at `0x00634fe0`. `structs/app_bootstrap.md` is the precedent
for a doc shaped like this.

It contains the three Dinkumware `<stdexcept>` classes, four out-of-line `basic_string<char>` members, the last import-thunk block, and the MSVC 6
C runtime: floating-point init, exit handlers, C++ EH / SEH, the heap, stdio, `qsort`, the string and ctype routines, the x87 math entry points, the
Pentium FDIV-erratum object, the classic iostream, and MSVC's `exception` and `type_info` classes. **No game translation unit lives here**, with one
deliberate exclusion: the fifth band's span also contains a PC-only game translation unit, the `.map`-file crash reporter, which has its own file
`structs/crash_report.md`. A finding is about the symbols it was measured on, never about the address range they occupy.

| | |
|---|---|
| PC prefix | - (selected by range; the names carry the library prefixes crt, math, std, ios and their neighbours) |
| PC kind (`+0x10`) | - |
| PC vtable | - (one per library class, §6c, §7, §4) |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x006243b7, 0x00631620)` and `[0x00633ab0, 0x00634fe0)`, a linker region; `[0x00631620, 0x00633ab0)` between them is `structs/crash_report.md` |
| PS2 class | 6 STRUCTURAL twins onto the Metrowerks `std` classes (§15a); every other row is `-` by measurement (§15) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

The region's data symbols are in §11. The records and library classes below are typed from their bodies. The stream's `FILE` (`0x20` bytes), the
two small-block-heap headers and `_ThrowInfo` (§6a) are the standard MSVC 6 shapes and are not typed here (Open item 1).

### 1a. `exception` — the base of §1c, `0xc` bytes

```c
typedef struct exception {
    /* +0x00 */ void ** vptr; // [H] g_exception_vt, two slots
    /* +0x04 */ char * message; // [H] MSVC _m_what: the text, or NULL
    /* +0x08 */ int32_t owns_message; // [H] MSVC _m_doFree: 1 when the text is owned and must be released
} exception;
```

### 1b. `std_basic_string` — Dinkumware `basic_string<char>`, `0x10` bytes

```c
typedef struct std_basic_string {
    /* +0x00 */ uint8_t allocator; // [H] the empty allocator's byte, copied by the constructors
    /* +0x04 */ char * text; // [H] _Ptr; the shared byte before it is the reference count
    /* +0x08 */ uint32_t length; // [H] _Len
    /* +0x0c */ uint32_t reserved; // [H] _Res: the capacity
} std_basic_string;
```

### 1c. `std_logic_error` — also `std::length_error` and `std::out_of_range`, which add only a vtable

```c
typedef struct std_logic_error {
    /* +0x00 */ exception base; // [H] §1a, with its vptr replaced by the class's
    /* +0x0c */ std_basic_string what_text; // [H] the message; its text pointer is this + 0x10, what std_logic_error_what returns
} std_logic_error;
```

### 1d. `type_info` — only its destructors are out of line

```c
typedef struct type_info {
    /* +0x00 */ void ** vptr; // [H] type_info::vftable at 0x0063efe8
    /* +0x04 */ void * undecorated_name; // [H] freed by the destructor when non-NULL
} type_info;
```

### 1e. `S_crt_xcpt_action` — one of the ten `g_crt_xcpt_action_table` records, `0xc` bytes

```c
typedef struct S_crt_xcpt_action {
    /* +0x00 */ uint32_t exception_code; // [H] the Win32 exception code the lookup compares
    /* +0x04 */ int32_t signal; // [H] the C signal number
    /* +0x08 */ void * action; // [H] the handler; NULL, or the SIG_DFL and SIG_IGN sentinels
} S_crt_xcpt_action;
```

### 1f. `S_crt_math_function_name` — one of the 27 `g_crt_math_function_names` pairs, 8 bytes

```c
typedef struct S_crt_math_function_name {
    /* +0x00 */ int32_t id; // [H] the id the math error paths pass, §9f
    /* +0x04 */ char * name; // [H] the C name of the function
} S_crt_math_function_name;
```

### 1g. `ios` — the classic stream base, the virtual base of §1i, `0x34` bytes

```c
typedef struct ios {
    /* +0x00 */ void ** vptr; // [H] g_ios_vt
    /* +0x04 */ streambuf * buffer; // [H] the attached buffer
    /* +0x08 */ uint32_t state; // [H] stream state; 4 while there is no buffer, and & 6 tests failbit and badbit
    /* +0x0c */ int32_t unread_0c; // [U] ios_ctor stores 0; no reader (class census + vbtable reads)
    /* +0x10 */ int32_t unread_10; // [U] ios_ctor stores 0; no reader (class census + vbtable reads)
    /* +0x14 */ uint32_t unused_14; // [U] no access; no reader (class census + vbtable reads)
    /* +0x18 */ uint32_t unused_18; // [U] no access; no reader (class census + vbtable reads)
    /* +0x1c */ int32_t delete_buffer; // [H] non-zero: the destructor and the setter delete the attached buffer
    /* +0x20 */ void * tie; // [H] the tied ostream, flushed by istream_ipfx before input (§2g)
    /* +0x24 */ uint32_t format_flags; // [H] format flags; bit 0 = skip whitespace (§2g)
    /* +0x28 */ int32_t precision; // [M] 6 from ios_ctor, the C++ default precision; no reader
    /* +0x2c */ char fill; // [M] ' ' from ios_ctor, the default fill; no reader
    /* +0x30 */ int32_t width; // [M] 0 from ios_ctor, the default width; no reader
} ios;
```

### 1h. `streambuf` — and `filebuf`, which adds two fields

```c
typedef struct streambuf {
    /* +0x00 */ void ** vptr; // [H] g_streambuf_vt or g_filebuf_vt
    /* +0x04 */ int32_t owns_buffer; // [H] the destructor and streambuf_setb delete buffer_base when set
    /* +0x08 */ int32_t unbuffered; // [H] streambuf_allocate does nothing when set; filebuf_setbuf sets it
    /* +0x0c */ int32_t last_char; // [H] the last character the get side returned
    /* +0x10 */ char * buffer_base; // [H] the reserve area's start
    /* +0x14 */ char * buffer_end; // [H] the reserve area's end
    /* +0x18 */ char * put_base; // [H] the put area's start
    /* +0x1c */ char * put_pointer; // [H] the next put position
    /* +0x20 */ char * put_end; // [H] the put area's end
    /* +0x24 */ char * get_back; // [H] the start of the putback area
    /* +0x28 */ char * get_pointer; // [H] the next get position
    /* +0x2c */ char * get_end; // [H] the get area's end
    /* +0x30 */ int32_t file_handle; // [H] filebuf only: the crt_sopen handle, -1 while closed
    /* +0x34 */ int32_t opened_here; // [H] filebuf only: 0 from the ctor, 1 after filebuf_open
} streambuf;
```

### 1i. `istream` — and `ifstream`, which adds nothing the reader touches

```c
typedef struct istream {
    /* +0x00 */ const int * vbtable; // [H] {0, 0xc}; the ios base is this + vbtable[1]
    /* +0x04 */ int32_t getline_mode; // [H] MSVC _fGline: a get_line extracts the delimiter when set
    /* +0x08 */ int32_t gcount; // [H] MSVC x_gcount: the characters the last read extracted
} istream;
```

No record holds an array. Only `structs/crash_report.md`'s reader uses the stream classes: `g_crashreport_map_file` is the one `ifstream` in the image.

## 2. Field evidence

The stream and string cells are the census of register-tagged `this` accesses of the named prefixes, plus the `ios` accesses the stream methods make through the vbtable, read from the bodies; `§1a`, `§1d`, `§1e` and `§1f` are read from the bodies.

### 2a. `exception`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `vptr` | H | `exception::~exception`, `exception_copy_ctor`, `exception_ctor_from_cstr` | - | virtual dispatch only; `ostream_flush`'s read of `[this+0]` is the vbtable of the tied `ostream`, not this class |
| +0x4 | `message` | H | `exception_copy_ctor`, `exception_ctor_from_cstr` | `exception::~exception`, `exception_copy_ctor`, `exception_what` | `exception_what` falls back to the literal `Unknown exception` |
| +0x8 | `owns_message` | H | `exception_copy_ctor`, `exception_ctor_from_cstr` | `exception::~exception`, `exception_copy_ctor` | the copy shares a literal and deep-copies an owned text |

### 2b. `std_basic_string`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `allocator` | H | `std_logic_error_copy_ctor`, `std_logic_error_ctor_from_string` | - | copied by the owner, never tested |
| +0x4 | `text` | H | `std_string_assign_substring`, `std_string_dtor`, `std_string_reserve`, `std_string_tidy` | `std_string_assign_buffer`, `std_string_assign_substring`, `std_string_dtor`, `std_string_erase`, `std_string_freeze`, `std_string_reserve`, `std_string_tidy` | - |
| +0x8 | `length` | H | `std_string_assign_buffer`, `std_string_assign_substring`, `std_string_dtor`, `std_string_erase`, `std_string_reserve`, `std_string_tidy` | `std_string_erase` | - |
| +0xc | `reserved` | H | `std_string_assign_substring`, `std_string_dtor`, `std_string_reserve`, `std_string_tidy` | `std_string_reserve` | - |

### 2c. `std_logic_error`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `base` | H | `std_length_error_copy_ctor`, `std_length_error_dtor`, `std_logic_error_copy_ctor`, `std_logic_error_ctor_from_string`, `std_logic_error_dtor`, `std_out_of_range_copy_ctor`, `std_out_of_range_dtor`, `std_string_xlen`, `std_string_xran` | - | the three destructors store the `logic_error` vtable; the two derived copy constructors, `std_string_xlen` and `std_string_xran` store the derived one (§6e) |
| +0xc | `what_text` | H | `std_logic_error_copy_ctor`, `std_logic_error_ctor_from_string` | `std_logic_error_copy_ctor`, `std_logic_error_what` | - |

### 2d. `type_info`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `vptr` | H | `type_info_dtor` | - | the destructor resets it; the image has no constructor |
| +0x4 | `undecorated_name` | M | - | `type_info_dtor` | named from the MSVC class; no writer in the image |

### 2e. `S_crt_xcpt_action`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `exception_code` | H | - | `crt_xcpt_action_lookup` | shipped data |
| +0x4 | `signal` | H | - | `crt_exception_filter`, `crt_xcpt_action_lookup_by_signal` | shipped data |
| +0x8 | `action` | H | `crt_exception_filter` | `crt_exception_filter` | cleared before the handler is called |

### 2f. `S_crt_math_function_name`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `id` | H | - | `math_function_name_for_id` | shipped data, §9f |
| +0x4 | `name` | H | - | `math_function_name_for_id` | shipped data |

### 2g. `ios`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `vptr` | H | `ifstream_ctor`, `ifstream_dtor`, `ios_ctor`, `ios_dtor`, `istream_ctor`, `istream_dtor` | - | the stream constructors and destructors store their own vtable through the vbtable |
| +0x4 | `buffer` | H | `ios_ctor`, `ios_dtor`, `ios_set_streambuf` | `ifstream_close`, `ifstream_open`, `ios_dtor`, `ios_set_streambuf`, `istream_eatwhite`, `istream_get_line`, `istream_ipfx`, `ostream_flush` | the stream methods read it through the vbtable |
| +0x8 | `state` | H | `crashreport_module_load_map_symbols`, `ifstream_close`, `ifstream_open`, `ios_ctor`, `ios_dtor`, `ios_set_streambuf`, `istream_eatwhite`, `istream_get_line`, `istream_ipfx`, `ostream_flush` | `crashreport_module_load_map_symbols`, `ifstream_close`, `ifstream_open`, `ios_set_streambuf`, `istream_eatwhite`, `istream_get_line`, `istream_ipfx`, `ostream_flush` | the stream methods and `crashreport_module_load_map_symbols` reach it through the vbtable |
| +0xc | `unread_0c` | U | `ios_ctor` | - | stores 0; no reader (class census + vbtable reads) |
| +0x10 | `unread_10` | U | `ios_ctor` | - | stores 0; no reader (class census + vbtable reads) |
| +0x14 | `unused_14` | U | - | - | no access (class census + vbtable reads) |
| +0x18 | `unused_18` | U | - | - | no access (class census + vbtable reads) |
| +0x1c | `delete_buffer` | H | `ifstream_ctor`, `ios_ctor` | `ios_dtor`, `ios_set_streambuf` | `ios_ctor` stores 0; `ifstream_ctor` stores 1 through the vbtable, so the stream deletes the `filebuf` it allocated |
| +0x20 | `tie` | H | `ios_ctor` | `istream_ipfx` | `ios_ctor` stores 0, the only store; `istream_ipfx` reads it through the vbtable and, when it is non-zero, calls `ostream_flush` on it unless the requested count is available (`streambuf_in_avail`) |
| +0x24 | `format_flags` | H | `ios_ctor`, `istream_ctor` | `istream_ipfx` | `ios_ctor` stores 0; `istream_ctor` sets bit 0 through the vbtable; `istream_ipfx` tests bit 0 before `istream_eatwhite` (skip whitespace) |
| +0x28 | `precision` | M | `ios_ctor` | - | 6, the C++ stream default precision; no reader |
| +0x2c | `fill` | M | `ios_ctor` | - | the byte `0x20` (`' '`), the default fill; no reader |
| +0x30 | `width` | M | `ios_ctor` | - | 0, the default width; no reader |

### 2h. `streambuf`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `vptr` | H | `filebuf_ctor`, `streambuf_ctor`, `streambuf_dtor` | `filebuf_close`, `filebuf_open`, `streambuf_allocate`, `streambuf_pbackfail`, `streambuf_seekpos`, `streambuf_sgetc`, `streambuf_snextc`, `streambuf_sputbackc`, `streambuf_sputc`, `streambuf_stossc`, `streambuf_xsgetn`, `streambuf_xsputn` | - |
| +0x4 | `owns_buffer` | H | `streambuf_ctor`, `streambuf_setb` | `streambuf_dtor`, `streambuf_setb` | - |
| +0x8 | `unbuffered` | H | `filebuf_open`, `filebuf_setbuf`, `streambuf_ctor`, `streambuf_setbuf` | `filebuf_open`, `filebuf_overflow`, `filebuf_sync`, `filebuf_underflow`, `streambuf_allocate`, `streambuf_pbackfail`, `streambuf_sgetc`, `streambuf_snextc`, `streambuf_stossc`, `streambuf_xsgetn`, `streambuf_xsputn` | - |
| +0xc | `last_char` | H | `filebuf_sync`, `filebuf_underflow`, `streambuf_ctor`, `streambuf_sgetc`, `streambuf_snextc`, `streambuf_stossc`, `streambuf_xsgetn` | `streambuf_sgetc`, `streambuf_snextc`, `streambuf_stossc`, `streambuf_xsgetn` | - |
| +0x10 | `buffer_base` | H | `streambuf_ctor`, `streambuf_setb`, `streambuf_setbuf` | `filebuf_overflow`, `filebuf_underflow`, `streambuf_allocate`, `streambuf_dtor`, `streambuf_setb`, `streambuf_setbuf` | - |
| +0x14 | `buffer_end` | H | `streambuf_ctor`, `streambuf_setb`, `streambuf_setbuf` | `filebuf_open`, `filebuf_overflow`, `filebuf_setbuf`, `filebuf_underflow` | - |
| +0x18 | `put_base` | H | `filebuf_overflow`, `filebuf_sync`, `streambuf_ctor` | `filebuf_sync`, `streambuf_sync` | - |
| +0x1c | `put_pointer` | H | `filebuf_overflow`, `filebuf_sync`, `streambuf_ctor`, `streambuf_sputc`, `streambuf_xsputn` | `filebuf_overflow`, `filebuf_sync`, `streambuf_sputc`, `streambuf_sync`, `streambuf_xsputn` | - |
| +0x20 | `put_end` | H | `filebuf_overflow`, `filebuf_sync`, `streambuf_ctor` | `filebuf_overflow`, `filebuf_sync`, `streambuf_sputc`, `streambuf_xsputn` | - |
| +0x24 | `get_back` | H | `filebuf_sync`, `filebuf_underflow`, `streambuf_ctor` | `streambuf_pbackfail`, `streambuf_sputbackc` | - |
| +0x28 | `get_pointer` | H | `filebuf_sync`, `filebuf_underflow`, `streambuf_ctor`, `streambuf_snextc`, `streambuf_sputbackc`, `streambuf_stossc`, `streambuf_xsgetn` | `filebuf_sync`, `filebuf_underflow`, `streambuf_in_avail`, `streambuf_pbackfail`, `streambuf_snextc`, `streambuf_sputbackc`, `streambuf_stossc`, `streambuf_sync`, `streambuf_xsgetn` | - |
| +0x2c | `get_end` | H | `filebuf_sync`, `filebuf_underflow`, `streambuf_ctor` | `filebuf_sync`, `streambuf_in_avail`, `streambuf_pbackfail`, `streambuf_snextc`, `streambuf_stossc`, `streambuf_sync`, `streambuf_xsgetn` | - |
| +0x30 | `file_handle` | H | `filebuf_close`, `filebuf_ctor`, `filebuf_open` | `filebuf_close`, `filebuf_open`, `filebuf_overflow`, `filebuf_seekoff`, `filebuf_setbuf`, `filebuf_sync`, `filebuf_underflow`, `ifstream_open` | `ifstream_open` reads it through the `ios` buffer pointer |
| +0x34 | `opened_here` | H | `filebuf_ctor`, `filebuf_open` | `filebuf_dtor` | - |

### 2i. `istream`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `vbtable` | H | `ifstream_ctor`, `istream_ctor` | `ifstream_close`, `ifstream_open`, `istream_eatwhite`, `istream_get_line`, `istream_ipfx`, `ostream_flush` | - |
| +0x4 | `getline_mode` | H | `crashreport_module_load_map_symbols`, `istream_ctor`, `istream_get_line` | `istream_get_line` | the caller's increment is at each read of the map file |
| +0x8 | `gcount` | H | `istream_ctor`, `istream_get_line`, `istream_ipfx` | `istream_get_line` | - |

## 3. Method family

The twins are the six of §15a; a C-library counterpart is deliberately not a twin, and no `-` here is an omission (§15). Rows are in PC VA order inside each table.
`Unwind_` and `EHHandler_` funclets are not listed: they belong to the functions whose exception tables name them.

### 3a. Construction & destruction

#### `std::logic_error`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00624411 | `std_logic_error_ctor_from_string` | instance | - | - | - | `this = std_logic_error` Constructs a `std::logic_error` from a `basic_string` |
| 0x0062449f | `std_logic_error_dtor` | instance | - | - | - | `this = std_logic_error` Stores the `logic_error` vtable `0x0063ef48`, releases the embedded string through `std_string_tidy(1)`, runs `exception::~exception` (which restores the base vtable) and unlinks the EH frame |
| 0x006244dc | `std_logic_error_scalar_deleting_dtor` | instance | `__dt__Q23std11logic_errorFv` | 0x0010b1d0 | STRUCTURAL | `this = std_logic_error` Runs `std_logic_error_dtor`, then frees the object itself when bit 0 of the flag argument is set |
| 0x006244f8 | `std_logic_error_copy_ctor` | instance | - | - | - | `this = std_logic_error` Copy-constructs a `logic_error` |

#### `std::length_error`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00624555 | `std_length_error_dtor` | instance | - | - | - | `this = std_logic_error` The `std::length_error` destructor |
| 0x006245af | `std_length_error_scalar_deleting_dtor` | instance | `__dt__Q23std12length_errorFv` | 0x0010b110 | STRUCTURAL | `this = std_logic_error` Runs `std_length_error_dtor`, then frees the object when bit 0 of the flag argument is set |
| 0x006245cb | `std_length_error_copy_ctor` | instance | - | - | - | `this = std_logic_error` Runs the `logic_error` copy constructor over the source, then overwrites the vptr with the `length_error` vtable |

#### `std::out_of_range`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0062463d | `std_out_of_range_dtor` | instance | - | - | - | `this = std_logic_error` The `std::out_of_range` destructor; all three `<stdexcept>` destructors store the `logic_error` vtable `0x0063ef48`, and the only bytes that differ are their EH function-info addresses (`mov eax, 0x63a35c` / `0x63a384` / `0x63a3ac`) |
| 0x00624697 | `std_out_of_range_scalar_deleting_dtor` | instance | - | - | - | `this = std_logic_error` Runs `std_out_of_range_dtor`, then frees the object when bit 0 of the flag argument is set |
| 0x006246b3 | `std_out_of_range_copy_ctor` | instance | - | - | - | `this = std_logic_error` Runs the `logic_error` copy constructor over the source, then overwrites the vptr with the `out_of_range` vtable |

#### the unit's CRT pairs

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00624865 | `crt_static_init_guard_00624865` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard of the shared flag bit and of `g_crt_static_init_flags_1`, §11e |
| 0x0062488f | `crt_atexit_register_0062488f` | `g_crt_static_init_flags` | - | - | - | Registers the binary's shared do-nothing static destructor |
| 0x0062489b | `crt_atexit_register_0062489b` | `g_crt_static_init_flags` | - | - | - | `crt_atexit` of the local no-op at `0x006248a7`, §11b |

#### `exception`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00627059 | `exception_scalar_deleting_dtor` | instance | `__dt__Q23std9exceptionFv` | 0x00101b00 | STRUCTURAL | runs the plain destructor `exception::~exception` and then, when bit 0 of the flag argument is set, releases the object itself through `operator delete` |
| 0x00627075 | `exception_ctor_from_cstr` | instance | - | - | - | installs the `exception` vtable, duplicates the caller's message with `operator new(strlen + 1)` + `strcpy`, and sets `_m_doFree` to 1 so the destructor will release the copy |
| 0x006270b2 | `exception_copy_ctor` | instance | - | - | - | installs the vtable and copies `_m_doFree` from the source |
| 0x006270fc | `exception::~exception` | instance | - | - | - | the plain destructor, §7 |

#### `type_info`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0062711f | `type_info_dtor` | instance | - | - | - | reinstalls `type_info::vftable` and releases the undecorated-name cache at `this+4` through `crt_free` when one was ever built |
| 0x00627134 | `type_info_scalar_deleting_dtor` | instance | - | - | - | calls `type_info_dtor` and then, when bit 0 of the flag is set, releases the object through `operator delete` |

#### `ifstream`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00633ab0 | `ifstream_ctor` | instance | - | - | - | `this = istream` installs the vbtable at `0x0063f658` and runs `ios_ctor` when the most-derived flag is set, allocates a 0x38-byte `filebuf` with `operator_new` and constructs it, binds it with `ios_set_streambuf`, |
| 0x00633b36 | `ifstream_scalar_deleting_dtor` | instance | - | - | - | `this = istream` adjusts `this` by -0xc to the most-derived object, runs `ifstream_dtor` and `ios_dtor`, then frees the storage when bit 0 of the flag is set |
| 0x00633b61 | `ifstream_dtor` | instance | - | - | - | `this = istream` restores the `ifstream` vtable pointer through the vbtable and destroys the owned `filebuf`; the virtual `ios` base is torn down separately by `ios_dtor` |

#### `ios`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00633bdf | `ios_ctor` | instance | - | - | - | installs the `ios` vtable `g_ios_vt`, clears the streambuf pointer, sets the state to `badbit` (4) until a buffer is bound, zeroes `unread_0c`, `unread_10`, `tie`, `format_flags`, `width` and `delete_buffer`, and installs the two formatting defaults the standard fixes: precision 6 and fill `' '` |
| 0x00633c11 | `ios_scalar_deleting_dtor` | instance | - | - | - | `ios_dtor(this); if (flag & 1) operator delete(this);` |
| 0x00633c2d | `ios_dtor` | instance | - | - | - | restores the `ios` vtable, deletes the bound streambuf through its own virtual destructor when the owns-buffer flag at `+0x1c` is set, clears the pointer and resets the state to `badbit` |

#### `filebuf`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00633d3f | `filebuf_ctor` | instance | - | - | - | `this = streambuf` runs the `streambuf` base constructor, sets the descriptor field at `+0x30` to -1 and the owns-file flag at `+0x34` to 0, then installs the `filebuf` vtable `g_filebuf_vt` |
| 0x00633d59 | `filebuf_scalar_deleting_dtor` | instance | - | - | - | `this = streambuf` runs the `filebuf` destructor at 0x00633d75 and frees the storage when bit 0 of the flag is set |
| 0x00633d75 | `filebuf_dtor` | instance | - | - | - | `this = streambuf` installs the `filebuf` vtable, then closes the file if `+0x34` (opened) is set or flushes with `filebuf_sync` if it is not, and finally runs `streambuf_dtor` |

#### `istream`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00634079 | `istream_scalar_deleting_dtor` | instance | - | - | - | runs `istream_dtor` and `ios_dtor`, then frees the object when bit 0 of the flag is set |
| 0x006340a4 | `istream_ctor` | instance | - | - | - | constructs the virtual `ios` base with `ios_ctor` when the constructor flag says it is the most-derived call, installs the vbtable and `g_istream_vt`, binds the buffer with `ios_set_streambuf`, sets |
| 0x0063411a | `istream_dtor` | instance | - | - | - | fifteen bytes: reinstalls `g_istream_vt` through the vbtable displacement so that the `ios` base destructor runs against the right type |

#### `streambuf`

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00634404 | `streambuf_ctor` | instance | - | - | - | installs `g_streambuf_vt`, sets `+0x0c` (the pushback slot) to -1 and zeroes every other field from `+0x04` to `+0x2c` |
| 0x00634431 | `streambuf_scalar_deleting_dtor` | instance | - | - | - | runs `streambuf_dtor` and frees the object when bit 0 of the flag is set |
| 0x0063444d | `streambuf_dtor` | instance | - | - | - | reinstalls the vtable, calls `streambuf_sync`, and deletes the buffer only when `+0x04` (the alloc flag) says the object owns it |

### 3b. Accessors

The accessor test of the method tables finds no accessor here: each one-field body also tests or falls back.

### 3c. Behaviour

#### `basic_string<char>`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x006243b7 | `std_string_xlen` | - | - | - | `this = none` The Dinkumware `_Xlen()` helper |
| 0x006245e3 | `std_string_xran` | - | - | - | `this = none` The Dinkumware `_Xran()` helper - the range guard every indexed `basic_string` operation calls |
| 0x006246cb | `std_string_assign_substring` | - | - | - | `this = std_basic_string` Assigns a substring of another string |
| 0x00624799 | `std_string_assign_buffer` | `assign__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FPCcUi` | 0x0010c640 | STRUCTURAL | `this = std_basic_string` Replaces the string's contents with `count` bytes from a raw buffer |
| 0x006247ce | `std_string_erase` | - | - | - | `this = std_basic_string` Removes `count` characters at `off` |
| 0x00624835 | `std_string_freeze` | - | - | - | `this = std_basic_string` Makes the buffer private |

#### `std::logic_error`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00624475 | `std_logic_error_what` | `what__Q23std11logic_errorCFv` | 0x0010c040 | STRUCTURAL | `this = std_logic_error` Returns the message pointer |
| 0x00624482 | `std_logic_error_throw_copy` | - | - | - | `this = std_logic_error` Copy-constructs the exception onto the caller's stack and throws that copy |

#### `std::length_error`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00624592 | `std_length_error_throw_copy` | - | - | - | `this = std_logic_error` Copy-constructs the `length_error` onto the caller's stack and throws that copy |

#### `std::out_of_range`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0062467a | `std_out_of_range_throw_copy` | - | - | - | `this = std_logic_error` Copy-constructs the `out_of_range` onto the caller's stack and throws that copy |

#### `exception`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00627112 | `exception_what` | `what__Q23std9exceptionCFv` | 0x00101b50 | STRUCTURAL | returns `_m_what`, or the literal `"Unknown exception"` when the object was default-constructed and carries no message |

#### `ifstream`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00633b74 | `ifstream_open` | - | - | - | `this = istream` when the `filebuf`'s descriptor is still -1 it calls `filebuf::open` with `ios::in` forced into the mode and returns on success; on any failure it sets `failbit` in the `ios` state |
| 0x00633bb0 | `ifstream_close` | - | - | - | `this = istream` calls `filebuf::close`; on success the state is cleared, on failure `failbit` is set |

#### `ios`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00633c56 | `ios_set_streambuf` | - | - | - | bind a streambuf to the stream, replacing and deleting any owned |

#### `istream`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00633c85 | `istream_get_line` | - | - | - | guards on the sentry, then reads characters until EOF, the delimiter or `count-1` bytes; EOF sets `eofbit` (ios `+0x08` bit 1) and, with nothing read, `failbit` (bit 2) |
| 0x00634129 | `istream_ipfx` | - | - | - | `istream::ipfx(int)` - the input prefix run before every extraction |
| 0x006341d0 | `istream_eatwhite` | - | - | - | `streambuf_sgetc` / `crt_isspace` / `streambuf_snextc` loop; on exhaustion it sets `eofbit` in `ios::state` at vbase `+0x08` and returns the address of that field |

#### `filebuf`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00633dba | `filebuf_close` | - | - | - | `this = streambuf` when `+0x30` (the fd) is not -1 it calls vtable slot 1 (`filebuf_sync`), then `crt_close`, and sets the fd back to -1 only if BOTH succeeded |
| 0x00633deb | `filebuf_overflow` | - | - | - | `this = streambuf` `streambuf_allocate` then `filebuf_sync`; on an unbuffered stream it writes the byte with `crt_write(fd, &c, 1)`, otherwise it hands it to `streambuf_sputc` |
| 0x00633e7e | `filebuf_underflow` | - | - | - | `this = streambuf` returns the pending byte when `streambuf_in_avail` is non-zero; otherwise `streambuf_allocate`, `filebuf_sync`, then `crt_read` into `+0x10..+0x14` and sets `+0x24`/`+0x28`/`+0x2c` (eback/gptr/egptr) |
| 0x00633f1e | `filebuf_seekoff` | - | - | - | `this = streambuf` maps `ios::beg`/`cur`/`end` (0/1/2) onto `crt_lseek`'s whence and rejects anything else, after flushing with `filebuf_sync` |
| 0x00633f62 | `filebuf_sync` | - | - | - | `this = streambuf` writes out the pending put area with `crt_write` (and `crt_memmove`s the remainder back on a short write), then seeks the unread read-ahead back with `crt_lseek` |
| 0x0063403a | `filebuf_setbuf` | - | - | - | `this = streambuf` overrides `streambuf_setbuf` by adding the `fd == -1` guard: a buffer may only be attached while the file is closed |
| 0x00634211 | `filebuf_open` | - | - | - | `this = streambuf` translates `ios::openmode` into `_open` oflags - bit 0x80 selects `_O_BINARY` over `_O_TEXT`, 0x20 (`nocreate`) suppresses `_O_CREAT`, 0x40 (`noreplace`) adds `_O_EXCL`, 0x08 (`app`) and 0x10 |

#### `streambuf`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00633e5c | `streambuf_sputc` | - | - | - | `if (pptr < epptr) *pptr++ = c;` else it dispatches to vtable slot 7 (`overflow`) |
| 0x00633f0e | `streambuf_in_avail` | - | - | - | `egptr > gptr ? egptr - gptr : 0` |
| 0x006341ab | `streambuf_sputbackc` | - | - | - | `if (eback < gptr) *--gptr = c;` else it dispatches to vtable slot 9 (`pbackfail`) |
| 0x0063435a | `streambuf_snextc` | - | - | - | handles the unbuffered case through the `+0x0c` pushback slot; otherwise steps `gptr` and calls vtable slot 8 (`underflow`) whenever it reaches `egptr` |
| 0x006343a6 | `streambuf_stossc` | - | - | - | the same walk as `streambuf_snextc` with the value thrown away; in the unbuffered case it just clears the `+0x0c` pushback slot |
| 0x006343df | `streambuf_sgetc` | - | - | - | returns the `+0x0c` pushback slot when unbuffered, filling it from vtable slot 8 (`underflow`) first if it is empty; otherwise calls `underflow` directly |
| 0x00634471 | `streambuf_setbuf` | - | - | - | attaches a caller-supplied buffer only while `+0x10` (base) is still NULL; a NULL or zero-length buffer marks the stream unbuffered instead |
| 0x006344a5 | `streambuf_xsputn` | - | - | - | per byte: straight into the put area when there is room, otherwise through vtable slot 7 (`overflow`); stops at the first failure and reports how many were written |
| 0x006344fb | `streambuf_xsgetn` | - | - | - | `crt_memcpy`s straight out of the get area in runs, calling vtable slot 8 (`underflow`) between them; the unbuffered path walks the `+0x0c` pushback slot one byte at a time |
| 0x00634592 | `streambuf_sync` | - | - | - | `streambuf::sync()` - vtable slot 1, the base implementation |
| 0x006345a9 | `streambuf_allocate` | - | - | - | `if (!unbuffered && !base) return doallocate() != -1 ? 1 : -1; return 0;` - vtable slot 10 does the actual allocation |
| 0x006345c7 | `streambuf_doallocate` | - | - | - | `operator_new(0x200)` then `streambuf_setb(p, p + 0x200, 1)` - the 1 marks the buffer as owned, so `streambuf_dtor` will free it |
| 0x006345f4 | `streambuf_setb` | - | - | - | frees the previous buffer when the object owned it, then writes `+0x10` (base), `+0x14` (ebuf) and `+0x04` (the ownership flag) |
| 0x00634624 | `streambuf_seekoff` | - | - | - | `or eax,-1; ret 0xc` - six bytes that return EOF and pop three dword arguments |
| 0x0063462a | `streambuf_seekpos` | - | - | - | `streambuf::seekpos(streampos, int)` - vtable slot 4 |
| 0x0063463c | `streambuf_pbackfail` | - | - | - | uses `streambuf_sputbackc` when there is still room below `gptr`; otherwise it seeks the file back one byte through vtable slot 3 and `crt_memmove`s the get area up to make space |

#### `ostream`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00634698 | `ostream_flush` | - | - | - | calls vtable slot 1 (`sync`) on the streambuf held at `ios::bp` and sets `failbit` in `ios::state` when it returns EOF |

### 3d. Free helpers

#### Band 1, `[0x006243b7, 0x00625021)`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x006248a8 | `_CreateJoyBase_20` | - | - | - | Creates the joystick / force-feedback device object |
| 0x006248ae | `_CreateINetBase_4` | - | - | - | Creates the networking transport object |
| 0x006248b4 | `dtaWrite` | - | - | - | Writes a block into an open `.dta` archive stream |
| 0x006248ba | `dtaRead` | - | - | - | Reads a block from an open `.dta` archive stream |
| 0x006248c0 | `dtaSeek` | - | - | - | Repositions the read/write cursor of an open `.dta` stream |
| 0x006248c6 | `dtaClose` | - | - | - | Closes an open `.dta` archive stream |
| 0x006248cc | `dtaOpen` | - | - | - | Opens a file inside a `.dta` archive for reading |
| 0x006248d2 | `dtaOpenWrite` | - | - | - | Opens a file inside a `.dta` archive for writing |
| 0x006248d8 | `dtaCreate` | - | - | - | Creates a new file inside a `.dta` archive |
| 0x006248de | `dtaSetDtaFirstForce` | - | - | - | Sets the archive-before-loose-file search priority for subsequent opens |
| 0x006248e4 | `crt_fp_init` | - | - | - | Initialises the CRT's floating-point support |
| 0x006248fb | `crt_fpclear` | - | - | - | MSVC's empty `_fpclear`, a one-byte `ret`; both per-thread hooks hold it, §9a |
| 0x006248fc | `crt_cfltcvt_init` | - | - | - | fill the six-entry float-to-text conversion vector |
| 0x00624934 | `operator_new` | - | - | - | Forwards to the CRT's new-handler-aware allocator, `__nh_malloc(size, 1)`, so a failed allocation runs the installed new-handler before giving up |
| 0x00624942 | `crt_onexit` | - | - | - | Appends one callback to the CRT's exit table |
| 0x006249af | `crt_atexit` | - | - | - | Appends the callback to the CRT's exit table through `crt_onexit`, and maps that function's pointer-or-null result onto C's 0-on-success / non-zero-on- failure contract |
| 0x006249c1 | `crt_onexit_init` | - | - | - | Allocates the 0x80-byte table that `crt_onexit` appends to, aborts the process through `__amsg_exit(0x18)` if the allocation fails, zeroes the first entry and points both table pointers at it |
| 0x006249f0 | `__ftol` | - | - | - | The compiler's float-to-integer helper |
| 0x00624a17 | `crt_eh_jump_to_continuation` | - | - | - | Leaves an exception handler for good |
| 0x00624a4b | `crt_eh_call_member_function_00624a4b` | - | - | - | Seven bytes of stack surgery |
| 0x00624a52 | `crt_eh_call_member_function_00624a52` | - | - | - | Identical to `crt_eh_call_member_function_00624a4b`, byte for byte |
| 0x00624a59 | `crt_eh_unwind_nested_frames` | - | - | - | Runs `RtlUnwind` up to the given registration node, then clears the `EXCEPTION_UNWINDING` bit in the exception record and relinks the SEH chain so the caller can carry on using the same record |
| 0x00624aa8 | `crt_cxx_frame_handler` | - | - | - | The entry point every C++ frame's exception handler funnels through |
| 0x00624ade | `crt_eh_call_catch_block` | - | - | - | calls a catch funclet under `crt_eh_catch_guard_handler` |
| 0x00624b32 | `crt_eh_catch_guard_handler` | - | - | - | Catches an exception thrown from inside a catch block |
| 0x00624b57 | `crt_eh_call_se_translator` | - | - | - | calls the `_set_se_translator` callback under `crt_eh_translator_guard_handler` |
| 0x00624c0b | `crt_eh_translator_guard_handler` | - | - | - | Handles an exception raised by the translator itself |
| 0x00624c80 | `crt_eh_get_try_range_to_check` | - | - | - | Given the current EH state and a catch nesting depth, walks the function's try-block map BACKWARDS to find which contiguous run of try blocks is eligible, and returns that run's `[start, end)` |
| 0x00624cfc | `__global_unwind2` | - | - | - | Calls `RtlUnwind` with a continuation label inside itself, which makes the OS run every `__finally` block between the current frame and the target one |
| 0x00624d1c | `crt_seh_unwind_collision_handler` | - | - | - | The exception handler that `__local_unwind2` installs while it runs `__finally` blocks |
| 0x00624d3e | `__local_unwind2` | - | - | - | Walks the frame's scope table backwards from the current level to the stop level, invoking each entry's `__finally` handler |
| 0x00624da6 | `__abnormal_termination` | - | - | - | The intrinsic behind `AbnormalTermination()` |
| 0x00624dc9 | `__NLG_Notify1` | - | - | - | Records where a non-local goto is about to jump so a debugger can follow it |
| 0x00624dd2 | `crt_nlg_notify` | - | - | - | Records where a non-local goto is about to jump, so a debugger can follow it |
| 0x00624df0 | `math_acos_x87_entry` | - | - | - | hands the operand in ST0 to the FPU-status helper at 0x00628638 and falls into `math_acos_continuation`, which computes the result and leaves it in ST0 |
| 0x00624e0d | `math_acos_continuation` | - | - | - | normalises the x87 control word if the caller left it non-default, then for /x/ < 1 computes `fpatan` of `sqrt((1 - x) * (1 + x))` against x — i.e |
| 0x00624ebb | `crt_atof` | - | - | - | Skips leading whitespace using the CRT ctype table (`g_crt_pctype`, class bit 8, with the multi-byte fallback when the locale flag `g_crt_mb_cur_max` is >= 2), takes `strlen`, runs the CRT |
| 0x00624f08 | `crt_atol` | - | - | - | Skips leading whitespace using the ctype table (the `isspace` bit 8, via the locale-aware helper when the codepage is not the default), takes an optional sign, then accumulates decimal digits |
| 0x00624f93 | `crt_atoi` | - | - | - | One-line forwarder to `crt_atol` — the shape MSVC emits for `atoi`, which is defined as `(int)atol(s)` |
| 0x00624fa0 | `crt_chkstk` | - | - | - | Given a frame size in EAX, page-probes the stack downward in 0x1000 steps (`test dword ptr [ecx], eax` per page so the guard page faults in order), then `mov esp,ecx` and returns through the saved |
| 0x00624fcf | `crt_sprintf` | - | - | - | Formats into a caller-supplied buffer with no bound |

#### Band 2, `[0x00625021, 0x006276a0)`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00625021 | `crt_vsnprintf` | - | - | - | builds a fake `FILE` on the stack — `_ptr`/`_base` = the caller's buffer, `_cnt` = the caller's size, `_flag` = 0x42 (`_IOWRT/_IOSTRG`) — hands it to the shared `_output` engine, then terminates: |
| 0x00625071 | `crt_ehvec_dtor` | - | - | - | Runs the element destructor over `count` elements in REVERSE order inside an SEH frame |
| 0x006250d9 | `crt_ehvec_dtor_unwind_funclet` | - | - | - | runs when an element destructor throws while `crt_ehvec_dtor` is tearing an array down |
| 0x006250f1 | `crt_eh_array_unwind` | - | - | - | walks `param_3` elements backwards from `param_1`, calling the element destructor `param_4` on each, inside its own SEH frame |
| 0x0062514f | `crt_ehvec_ctor` | - | - | - | MSVC `__ehvec_ctor` — construct an array of N objects with exception unwinding |
| 0x006251b9 | `crt_ehvec_ctor_unwind_funclet` | - | - | - | runs when an element CONSTRUCTOR throws part-way through building an array |
| 0x006251d1 | `crt_fclose` | - | - | - | Flushes, frees the buffer, closes the OS handle and removes any temp-file name |
| 0x00625227 | `crt_fwrite` | - | - | - | Buffered write, the mirror of `crt_fread` |
| 0x00625331 | `crt_fsopen` | - | - | - | takes a free `FILE` slot from the stream table (`_getstream`) and, if one is available, hands it plus the caller's name / mode / share flag to the real open routine |
| 0x00625351 | `crt_fopen` | - | - | - | Thin wrapper over the internal open with share mode 0x40 |
| 0x00625370 | `_strrchr` | - | - | - | `strrchr` |
| 0x00625397 | `crt_srand` | - | - | - | stores its argument into `g_crt_rand_seed` and returns |
| 0x006253a1 | `crt_rand` | - | - | - | advances the global LCG seed `g_crt_rand_seed` with the MSVC constants (`seed = seed*0x343fd + 0x269ec3`) and returns `(seed >> 16) & 0x7fff`, a value in [0, 0x7fff] |
| 0x006253bf | `crt_qsort` | - | - | - | sorts `num` elements of `width` bytes at `base` using the caller's three-way comparator |
| 0x00625513 | `crt_qsort_shortsort` | - | - | - | sorts the inclusive byte range [lo, hi] in place by repeatedly selecting the maximum with the caller's comparator and swapping it to the end, shrinking the range by one element each pass |
| 0x00625561 | `crt_qsort_swap` | - | - | - | exchanges `param_3` bytes between the two addresses one byte at a time |
| 0x0062558d | `crt_strupr` | - | - | - | while `g_crt_mbcs_locale_active` is clear it walks the buffer and subtracts 0x20 from every byte in `'a'..'z'`, touching nothing else |
| 0x0062562b | `crt_sscanf` | - | - | - | Builds a fake `FILE` on the stack — `{buf, strlen(buf), buf, flag 0x49}`, the `_IOREAD/_IOSTRG/_IOMYBUF` combination — and hands it to the scanf engine with the caller's format and varargs |
| 0x00625660 | `_strncmp` | - | - | - | `strncmp` |
| 0x00625698 | `crt_isalpha` | - | - | - | returns a non-zero value when the character is a letter |
| 0x006256c6 | `crt_isspace` | - | - | - | identical shape to `crt_isalpha` with the mask 8 = `_SPACE` |
| 0x00625700 | `_strchr` | - | - | - | `strchr` |
| 0x006257c0 | `_strstr` | - | - | - | `strstr` |
| 0x00625840 | `crt_memmove` | - | - | - | when `src < dst < src + n` it copies BACKWARDS from the end; otherwise forwards |
| 0x00625b75 | `crt_fprintf` | - | - | - | locks the stream, runs the shared `_output` engine over the caller's format and varargs, unlocks, and returns `_output`'s character count |
| 0x00625bb0 | `_strncpy` | - | - | - | `strncpy` |
| 0x00625cae | `crt_ceil` | - | - | - | installs the x87 control word `g_crt_x87_cw_round_up`, rounds with `frndint`, restores the caller's control word and returns |
| 0x00625d7d | `crt_floor` | - | - | - | identical body to `crt_ceil` but installs `g_crt_x87_cw_round_down` and reports FP exceptions with opcode 0x0b |
| 0x00625e4c | `crt_fread` | - | - | - | Buffered read honouring the `FILE` flag bits 0x10c |
| 0x00625f34 | `crt_remove` | - | - | - | `DeleteFileA` plus the Win32-to-errno mapping |
| 0x00625f60 | `math_pow_x87_entry` | - | - | - | takes both operands on the x87 register stack rather than on the integer stack — `sub esp,0x10; fxch st(1); fstp [esp]; fst [esp+8]` — spills them to a 16-byte scratch frame, calls |
| 0x00625f82 | `math_pow_continuation` | - | - | - | normalises the x87 control word if the caller left it non-default, classifies both operands (zero / finite / infinity / NaN, and the quiet-vs-signalling bit), applies the IEEE special cases, and |
| 0x00626155 | `math_x87_integer_parity_pop` | - | - | - | compares `frndint(ST0)` with ST0 |
| 0x00626180 | `_strncat` | - | - | - | `strncat` |
| 0x006262a3 | `crt_purecall` | - | - | - | nine bytes — `push 0x19; call __amsg_exit; pop ecx` — which prints the runtime-error banner and terminates |
| 0x006262ac | `char_to_upper_locale` | - | - | - | ASCII fast path (`'a'..'z'` minus 0x20) while the MBCS locale flag is clear; otherwise checks the `_LOWER` bit (2) in the locale class table at `g_crt_pctype`, assembles the possibly two-byte |
| 0x00626378 | `char_to_lower_locale` | - | - | - | ASCII fast path when the MBCS locale flag is clear; otherwise consults the locale class table at `g_crt_pctype` and handles MBCS lead bytes |
| 0x00626443 | `crt_realloc` | - | - | - | the three-way `realloc` contract - a null pointer forwards to `_malloc`, a zero size forwards to the free routine at 0x00626795, and otherwise it takes the heap grow/move path (including the |
| 0x006266e3 | `_malloc` | - | - | - | `malloc` |
| 0x006266f5 | `__nh_malloc` | - | - | - | `_nh_malloc`: the allocation loop around the new handler |
| 0x00626721 | `crt_heap_alloc` | - | - | - | dispatches on `g_crt_heap_mode` |
| 0x00626795 | `crt_free` | - | - | - | returns immediately on NULL; then, depending on `g_crt_heap_mode`, returns the block to the small-block allocator it came from, and otherwise calls `HeapFree(g_crt_heap_handle, 0, p)` |
| 0x00626800 | `__aulldiv` | - | - | - | 64-bit unsigned divide helper |
| 0x00626870 | `__allmul` | - | - | - | 64-bit multiply helper |
| 0x006268a4 | `crt_vsprintf` | - | - | - | byte-for-byte `crt_vsnprintf` except that `_cnt` is pinned at 0x7fffffff, so no size limit is ever enforced; the stack `FILE` carries `_flag` 0x42 and the caller's buffer, `_output` runs, then the |
| 0x00626900 | `__allshl` | - | - | - | 64-bit shift-left helper |
| 0x00626920 | `math_asin_x87_entry` | - | - | - | hands the operand in ST0 to the FPU-status helper at 0x00628638 and falls into `math_asin_continuation`, which computes the result and leaves it in ST0 |
| 0x0062693d | `math_asin_continuation` | - | - | - | normalises the x87 control word if the caller left it non-default, then for /x/ < 1 computes `fpatan` of x against `sqrt((1 - x) * (1 + x))` — i.e |
| 0x006269eb | `crt_time` | - | - | - | reads `GetLocalTime` and `GetSystemTime`; when the UTC year / month / day / hour / minute differ from the cached copy it refreshes that cache and re-evaluates daylight saving through |
| 0x00626ac7 | `crt_strtol` | - | - | - | forwards to the shared core `crt_strtoxl` with a flags word of 0, which selects the signed range and the `LONG_MIN`/`LONG_MAX` saturation limits |
| 0x00626ade | `crt_strtoxl` | - | - | - | skips leading whitespace with the `_SPACE` class bit, consumes an optional sign, resolves a radix of 0 by sniffing `0x`/`0X` (16), a leading `0` (8) or neither (10), skips a `0x` prefix when the |
| 0x00626ce6 | `crt_strtoul` | - | - | - | forwards to `crt_strtoxl` with a flags word of 1, selecting the unsigned range and `ULONG_MAX` saturation |
| 0x00626cfd | `crt_call_init_fns` | - | - | - | calls `g_crt_fp_init_hook` when set, then `crt_initterm` over `.CRT$XI` (`g_crt_xi_a`..`g_crt_xi_z`) and `.CRT$XC` (`g_crt_xc_a`..`g_crt_xc_z`), which runs every static constructor and `atexit` |
| 0x00626d2a | `crt_exit` | - | - | - | `crt_doexit(code, 0, 0)`: the `atexit` table, the pre-terminators, the terminators, then `ExitProcess(code)` |
| 0x00626d3b | `__exit` | - | - | - | `_exit`; held by `g_crt_aexit_rtn` |
| 0x00626d4c | `crt_doexit` | - | - | - | MSVC `doexit`: the shared body of `exit` and `_exit` |
| 0x00626de5 | `crt_initterm` | - | - | - | walks the half-open dword range [first, last) forward, calling every entry that is not NULL. The NULL skip is what lets the linker leave gaps between `.CRT$XC`/`.CRT$XI` contributions |
| 0x00626dff | `crt_getcwd` | - | - | - | forwards to `crt_getdcwd` with drive 0, which is that function's "current drive" selector |
| 0x00626e12 | `crt_getdcwd` | - | - | - | for drive 0 it calls `GetCurrentDirectoryA` into a MAX_PATH scratch buffer; otherwise it validates the drive with `crt_drive_is_valid`, builds the two-character path `"<L>:."` and resolves it with |
| 0x00626ee3 | `crt_drive_is_valid` | - | - | - | returns 1 immediately for drive 0 (the current drive, always valid); otherwise builds `"<L>:\"` on the stack and rejects `GetDriveTypeA` results 0 (unknown) and 1 (no root directory) |
| 0x00626f1a | `crt_startup` | - | - | - | under an `__except_handler3` frame (`g_crt_startup_scope_table`) it splits `GetVersion` into `g_crt_winminor`, `g_crt_winmajor`, `g_crt_winver` and `g_crt_osver`; runs `crt_heap_init(0)` and leaves |
| 0x00627010 | `__amsg_exit` | - | - | - | writes the numbered runtime message and exits through `g_crt_aexit_rtn` |
| 0x00627035 | `crt_fast_error_exit` | - | - | - | prints the runtime-error banner for the given `_RT_*` code (via the 0x0062d25e / 0x0062d297 message pair, gated on the app-type flag `g_crt_error_mode`) and then calls `ExitProcess(0xff)` DIRECTLY — |
| 0x00627150 | `_strlen` | - | - | - | `strlen` |
| 0x006271cc | `crt_seh_prolog` | - | - | - | `push -1; push eax; mov eax, fs:[0]; push eax; mov eax,[esp+0xc]; mov fs:[0], esp; mov [esp+0xc], ebp` — links a fresh `EXCEPTION_REGISTRATION` record onto the FS:[0] chain, saving the caller's |
| 0x006271eb | `__CxxThrowException_8` | - | - | - | `_CxxThrowException` |
| 0x00627230 | `crt_memcpy` | - | - | - | aligns the destination, moves whole dwords in an unrolled 8-way loop and finishes the 0..3 trailing bytes through a jump table |
| 0x00627565 | `crt_set_default_fp_precision` | - | - | - | calls `_controlfp(0x10000, 0x30000)` — `_PC_53` under the `_MCW_PC` mask — putting the x87 into 53-bit (double) precision rather than its 64-bit default |
| 0x00627577 | `crt_fdiv_erratum_selftest` | - | - | - | loads two constants from `.rdata`, computes `y - (y / x) * x` and compares the residual against a tolerance at 0x0063efb0 |
| 0x006275b5 | `crt_detect_fdiv_erratum` | - | - | - | `GetModuleHandleA("KERNEL32")` then `GetProcAddress("IsProcessorFeaturePresent")`, and calls it with feature 0 = `PF_FLOATING_POINT_PRECISION_ERRATA` |
| 0x006275de | `crt_forcdecpt` | - | - | - | given a `%g`-style buffer with no fractional part, finds the insertion point (before the exponent letter if there is one, else after the digits) and shifts the tail one byte right to make room for |
| 0x00627638 | `crt_cropzeros` | - | - | - | walks to `g_crt_decimal_point`, then to the exponent letter or the end, backs up over every trailing `'0'`, drops the decimal point too if nothing survives after it, and shifts the exponent tail down |
| 0x00627686 | `crt_positive` | - | - | - | `return *(double *)arg >= 0.0` |
| 0x0062769e | `__fassign` | - | - | - | `_fassign`: slot 2 of `g_crt_cfltcvt_tab`, §9e |

#### Band 3, `[0x006276a0, 0x0062ad26)`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x006276dc | `crt_cftoe` | - | - | - | MSVC `_cftoe`: format a double in `%e` (scientific) form |
| 0x006277e0 | `crt_cftof` | - | - | - | same conversion gate as `crt_cftoe`, then emits the integer part (a single '0' when `decpt < 1`), the decimal point, and `_memset('0', n)` zero padding for a decimal point left of the buffer |
| 0x006278be | `crt_cftog` | - | - | - | converts ONCE into the shared `g_crt_cvt_strflt`, records `decpt - 1` in `g_crt_cvt_exponent` and whether rounding bumped it in `g_crt_cvt_rounded_up`, then picks scientific notation when the |
| 0x00627959 | `crt_cftoe_preconverted` | - | - | - | sets `g_crt_cvt_preconverted`, calls `crt_cftoe`, clears it again |
| 0x00627980 | `crt_cftof_preconverted` | - | - | - | sets `g_crt_cvt_preconverted`, calls `crt_cftof`, clears it |
| 0x006279a3 | `__cfltcvt` | - | - | - | `_cfltcvt`: slot 0 of `g_crt_cfltcvt_tab` |
| 0x006279f4 | `crt_cvt_shift_string` | - | - | - | `if (n) memmove(s + n, s, strlen(s) + 1)` — moves the string (and its terminator) right by `n`, leaving `n` uninitialised bytes at the front for the caller to fill with a sign, a digit or the decimal |
| 0x00627a19 | `crt_msize` | - | - | - | three-way on `g_crt_heap_mode`, exactly as `crt_heap_alloc`, `crt_free` and `crt_realloc` do |
| 0x00627a7a | `crt_eh_handle_frame` | - | - | - | `__InternalCxxFrameHandler`, §9d |
| 0x00627b15 | `crt_eh_find_handler` | - | - | - | validates the frame's current try-state, resolves a RETHROW (`throw;`, spotted as a ThrowInfo-less record with magic 0x19930520) against `g_crt_eh_current_exception`, then walks the try-block map |
| 0x00627cbc | `crt_eh_find_handler_for_foreign_exception` | - | - | - | first offers the record to `g_crt_se_translator` through `crt_eh_call_se_translator` and returns if the translator threw a C++ exception |
| 0x00627d64 | `crt_eh_catch_type_matches` | - | - | - | accepts immediately for a catch-all (null or empty type name) |
| 0x00627dc1 | `crt_eh_unwind_frame_to_state` | - | - | - | walks `FuncInfo->pUnwindMap` from `frame->state` toward `target`, validating each index against the map's length (`crt_unexpected` on a corrupt one), and calls each non-null unwind action through |
| 0x00627e5f | `crt_eh_unhandled_exception_filter` | - | - | - | `if (pPtrs->ExceptionRecord->ExceptionCode != 0xe06d7363) return 0;` (EXCEPTION_CONTINUE_SEARCH) — otherwise calls `crt_terminate`, which does not return |
| 0x00627e70 | `thunk_crt_terminate` | - | - | - | a jump to `crt_terminate` |
| 0x00627e75 | `crt_eh_enter_catch_handler` | - | - | - | four steps in order — build the catch parameter with `crt_eh_build_catch_object` (skipped for a catch-all, which has no parameter); `crt_eh_unwind_nested_frames` to pop any frames between the throw |
| 0x00627ef0 | `crt_eh_run_catch_block` | - | - | - | `CatchIt` / `CallCatchBlock`, §9d |
| 0x00627fb6 | `crt_eh_run_catch_block_finally` | - | - | - | restores `g_crt_eh_current_exception` and `g_crt_eh_current_exception_context` from its parent's frame, then — only on ABNORMAL termination (`__abnormal_termination()`), and only when the record |
| 0x00627ffc | `crt_eh_rethrow_exception_filter` | - | - | - | takes the EXCEPTION_POINTERS, and returns 1 (EXCEPTION_EXECUTE_HANDLER) when the record is an MSVC C++ exception - code 0xE06D7363, NumberParameters 3, magic 0x19930520 - whose ThrowInfo pointer |
| 0x00628026 | `crt_eh_build_catch_object` | - | - | - | computes the parameter's address in the catch's frame (`FuncInfo->dispCatchObj + frame`) and then takes one of four routes, chosen by the HandlerType and CatchableType flags: catch-BY-REFERENCE (`HT` |
| 0x006281e2 | `crt_eh_terminate_funclet_006281e2` | - | - | - | eight bytes — restores ESP from the parent frame and tail-jumps to `crt_terminate` |
| 0x006281ea | `crt_eh_destroy_exception_object` | - | - | - | if the record carries a ThrowInfo with a non-null destructor (`[[record+0x1c]+4]`), calls it on the object at `record+0x18` through the shared member-function trampoline, under |
| 0x00628249 | `crt_eh_terminate_funclet_00628249` | - | - | - | eight bytes — restores ESP from the parent frame and tail-jumps to `crt_terminate` |
| 0x00628251 | `crt_eh_adjust_pointer` | - | - | - | `p = base + pmd.mdisp`; then, when `pmd.pdisp >= 0`, follows the virtual base table — `p += *(int *)(*(int *)(base + pmd.pdisp) + pmd.vdisp) + pmd.pdisp` |
| 0x00628280 | `__CallSettingFrame_12` | - | - | - | brackets the indirect call in `__NLG_Notify1` - once with the caller's code (0x101 for an unwind funclet, 0x100 for a catch block) and once on the way out, with 0x100 mapped to 2 |
| 0x006282cc | `crt_terminate` | - | - | - | calls `g_crt_terminate_handler` if one is installed, then `abort` (0x0062dc71) unconditionally |
| 0x00628322 | `crt_unexpected` | - | - | - | calls `g_crt_unexpected_handler` if one is installed, then `crt_terminate` |
| 0x00628380 | `math_x87_dispatch_by_operand_class` | - | - | - | sets the x87 control word (masked, 64-bit precision; or the caller's saved word with PC forced when the descriptor byte +0xe is 5), runs `fxam` on ST(0), folds C3/C2/C0 through the class table at |
| 0x006283e7 | `math_x87_dispatch_by_operand_class_pair` | - | - | - | the two-operand form of `math_x87_dispatch_by_operand_class`: same control-word setup, `fxam` on ST(0) and (via `fxch`) on ST(1), each folded through the table at 0x00651c9d; the index is class(ST0) |
| 0x00628478 | `math_x87_return_first_operand` | - | - | - | `fxch st(1); fstp st(0); ret` — five bytes |
| 0x00628486 | `math_x87_return_zero` | - | - | - | `fstp st(0); fldz; ret` |
| 0x0062848d | `math_x87_return_one` | - | - | - | `fstp st(0); fld1; ret` |
| 0x00628492 | `math_x87_return_nan_operand` | - | - | - | stores ST(0) as an 80-bit value in the caller's frame ([ebp-0x9e]) and tests the quiet-NaN bit (bit 0x40 of byte [ebp-0x97], mantissa bit 62) |
| 0x00628539 | `math_x87_mark_operand_snan` | - | - | - | ten bytes — stores 1 into `[ebp-0x90]`, the operand-class byte in its CALLER's frame, then `or cl, cl` to set the flags from the packed `fxam` class the dispatcher keeps in CL. It is a frame-SHARING |
| 0x00628550 | `__startTwoArgErrorHandling` | - | - | - | the x87 error entry of a two-argument math function |
| 0x00628567 | `__startOneArgErrorHandling` | - | - | - | the x87 error entry of a one-argument math function |
| 0x006285b0 | `math_x87_exp2` | - | - | - | `f2xm1` only covers a fractional exponent, so this splits x into `round(x)` and the remainder: `fscale(1 + f2xm1(-(round(x) - x)), round(x))` |
| 0x006285c5 | `math_x87_mask_exceptions` | - | - | - | `cw = (saved_cw & 0x300) / 0x7f; fldcw cw` — keeps the caller's PRECISION control (bits 9:8) and replaces everything else: all six exception masks set and rounding control forced to 00 (nearest) |
| 0x006285dc | `math_x87_classify_nan` | - | - | - | tests bit 0x80000 of the value's HIGH DWORD — bit 51 of the mantissa, the quiet-NaN flag |
| 0x006285f5 | `__fload_withFB` | - | - | - | for a finite value it is just `fld qword [edx]` |
| 0x00628638 | `math_x87_exponent_or_nan_bits` | - | - | - | returns `hi & 0x7ff00000` — the biased exponent field, which is 0 for zero and denormals — EXCEPT when that field is all ones, where it returns the WHOLE high dword instead so the caller can separate |
| 0x0062864e | `math_x87_restore_control_word` | - | - | - | put the caller's x87 control word back on the way out |
| 0x0062865b | `__math_exit` | - | - | - | if the caller's saved control word is the hardware default 0x27f it does nothing |
| 0x0062868b | `math_x87_check_result_range_prespilled` | - | - | - | the entry into `math_x87_check_result_range` for a result on the stack |
| 0x00628699 | `math_x87_check_result_range` | - | - | - | spills ST(0) and reads its exponent field |
| 0x0062873c | `crt_string_to_double` | - | - | - | parses the string through the shared scanner at 0x0062dd58, classifies the result with 0x0062d9a4, and fills the static record `g_crt_fltin_result`: `+0x00` status flags (0x80 = overflow, 0x100 = |
| 0x006287cd | `crt_isctype` | - | - | - | for a single byte (`c + 1 <= 0x100`) it is just `g_crt_pctype[c] & mask` |
| 0x00628842 | `crt_flsbuf` | - | - | - | rejects a stream not open for writing (`_flag & 0x82`, or `_IOSTRG` 0x40) by setting `_IOERR` |
| 0x00628957 | `crt_output` | - | - | - | walks the format string as a state machine — `g_crt_output_char_class_table` maps a character to a class and `g_crt_output_state_table` maps (class, state) to the next state — accumulating flags, |
| 0x00629098 | `crt_output_write_char` | - | - | - | `if (--stream->_cnt >= 0) *stream->_ptr++ = c; else c = _flsbuf(c, s);` then bumps the caller's running count, or sets it to -1 when the store failed |
| 0x006290cd | `crt_output_write_multi_char` | - | - | - | loops `crt_output_write_char`, stopping early when the running count goes to -1 |
| 0x006290fe | `crt_output_write_string` | - | - | - | loops `crt_output_write_char` over the buffer, stopping early on the -1 sentinel |
| 0x00629136 | `crt_output_get_int_arg` | - | - | - | `*ap += 4; return ((int *)*ap)[-1];` |
| 0x00629143 | `crt_output_get_int64_arg` | - | - | - | `*ap += 8; return ((__int64 *)*ap)[-1];` |
| 0x00629153 | `crt_output_get_short_arg` | - | - | - | `*ap += 4;` then returns the low half-word of the slot, leaving the high half of the register as it was — the wide-character case, where the argument occupies a full slot but only two bytes are |
| 0x0062916c | `__except_handler3` | - | - | - | MSVC's SEH handler; it has a registry row and no def, since nothing calls it |
| 0x00629229 | `__seh_longjmp_unwind_4` | - | - | - | takes an MSVC `jmp_buf` (`_JUMP_BUFFER`), establishes the target frame by loading its saved `Ebp` from `jb+0x00`, then calls `__local_unwind2(jb->Registration, jb->TryLevel)` — `jb+0x18` and |
| 0x00629244 | `crt_close` | - | - | - | validates the descriptor against `g_crt_handle_count` and the `FOPEN` bit in `g_crt_osfhandle_blocks[fd >> 5][fd & 0x1f]`, setting EBADF if either fails |
| 0x006292f7 | `__freebuf` | - | - | - | only acts on an open stream (`_flag & 0x83`) whose buffer is the library's own (`_IOMYBUF`, 0x8) |
| 0x00629322 | `crt_fflush` | - | - | - | a NULL stream means "flush every open stream", delegated to `flsall(0)` |
| 0x0062935d | `crt_flush_stream` | - | - | - | for a write-mode stream with a buffer (`_flag & 3 == 2` and `_flag & 0x108`), writes `_ptr - _base` bytes through `crt_write`; a short write sets `_IOERR` and returns -1 |
| 0x006293b9 | `crt_flushall` | - | - | - | nine bytes — `flsall(1)` |
| 0x006293c2 | `flsall` | - | - | - | walks `g_crt_stream_table` up to `g_crt_stream_count` and flushes every open stream through `crt_fflush` |
| 0x0062942f | `crt_write` | - | - | - | validates the descriptor against `g_crt_handle_count` and the `FOPEN` bit, seeks to the end first for an append-mode handle, then either calls `WriteFile` directly (binary mode) or, for a TEXT-mode |
| 0x006295dc | `crt_openfile` | - | - | - | the first character must be `r`, `w` or `a` and fixes the base oflags and stream flags |
| 0x0062974c | `crt_getstream` | - | - | - | scans `g_crt_stream_table` up to `g_crt_stream_count` |
| 0x006297d0 | `crt_strcpy` | - | - | - | seven bytes of its own — `push edi; mov edi, [esp+8]; jmp` — into the dword-at-a-time copy loop it SHARES with `crt_strcat` |
| 0x006297e0 | `crt_strcat` | - | - | - | finds the destination's terminator with the same dword-at-a-time zero scan, then falls into the copy loop `crt_strcpy` jumps into — one object file, one loop, two entry points |
| 0x006298c0 | `crt_lcmapstring_ansi` | - | - | - | `__crtLCMapStringA`, §9n |
| 0x00629a84 | `crt_lcmapstring_ansi_except_filter` | - | - | - | four bytes, `push 1; pop eax; ret` — returns EXCEPTION_EXECUTE_HANDLER unconditionally, so any fault raised while the wide-character conversion buffer is being allocated or filled transfers to the |
| 0x00629ae4 | `crt_input` | - | - | - | walks the format string, matching literal text and whitespace runs against the stream and executing one conversion per `%` |
| 0x0062a509 | `crt_input_hex_digit_value` | - | - | - | a decimal digit is returned unchanged (the caller subtracts '0' afterwards); a letter is upper-cased and reduced by 7, so 'a'/'A' land 10 above '9' |
| 0x0062a540 | `crt_input_getc` | - | - | - | `--stream->_cnt >= 0 ? *stream->_ptr++ : _filbuf(stream)` — the inline `getc` fast path with the buffer-refill call as its slow path |
| 0x0062a55a | `crt_input_ungetc` | - | - | - | `if (c != -1) ungetc(c, stream)` — the guard is the whole function; a scanf conversion that stops on EOF must not push it back |
| 0x0062a571 | `crt_input_skip_whitespace` | - | - | - | `do { ++*count; c = crt_input_getc(s); } while (crt_isspace(c));` and returns the first non-space character, which the caller then re-examines |
| 0x0062a595 | `crt_setup_temp_buffer` | - | - | - | MSVC `_stbuf`: give stdout or stderr a temporary buffer for |
| 0x0062a622 | `crt_flush_temp_buffer` | - | - | - | flushes the stream through `crt_flush_stream` when it is buffered |
| 0x0062a65f | `math_raise_domain_error` | - | - | - | when `g_crt_math_skip_matherr` is clear (which it always is as shipped), builds the `_exception` record with type 1 = `_DOMAIN` and hands it to `math_invoke_matherr` |
| 0x0062a6b2 | `math_raise_fp_exception` | - | - | - | first `math_apply_masked_exception_result`, which supplies the default result for every MASKED exception and reports whether any are left |
| 0x0062a74a | `math_raise_ieee_exception` | - | - | - | maps the internal flag set to one `STATUS_FLOAT_*` code — 0xC000008E divide-by-zero, 0xC000008F inexact, 0xC0000090 invalid operation, 0xC0000091 overflow, 0xC0000093 underflow — and fills the |
| 0x0062a9fd | `math_apply_masked_exception_result` | - | - | - | applies the masked math exception's result, one arm per exception |
| 0x0062ac14 | `math_invoke_matherr` | - | - | - | looks the function's name up with `math_function_name_for_id` and bails out to `math_set_errno_for_exception_type` if the id is unknown |
| 0x0062ac9c | `math_set_errno_for_exception_type` | - | - | - | type 1 (`_DOMAIN`) sets EDOM (0x21); types 2 and 3 (`_SING` and `_OVERFLOW`) set ERANGE (0x22); anything else leaves `errno` alone |
| 0x0062acc2 | `math_function_name_for_id` | - | - | - | a linear scan of the `{ int id; char *name; }` pairs in `g_crt_math_function_names`, bounded by the literal end address 0x00651fc8 that the loop compares against |
| 0x0062ace7 | `math_exception_type_from_flags` | - | - | - | highest priority first — 0x20 gives 5, invalid (8) gives 1 `_DOMAIN`, divide-by-zero (4) gives 2 `_SING`, overflow (1) gives 3 `_OVERFLOW`, and underflow (2) gives 4 `_UNDERFLOW` |
| 0x0062ad14 | `__frnd` | - | - | - | a `frndint` wrapper |

#### Band 4, `[0x0062ad26, 0x0062ec30)`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0062ad26 | `math_double_set_exponent` | - | - | - | rebuilds the high 16 bits as `(exponent + 0x3fe) << 4 / (hi16 & 0x800f)`, keeping the sign bit and the top four mantissa bits and replacing the 11-bit exponent |
| 0x0062ad4f | `math_classify_double_special` | - | - | - | tests the raw halves: `hi == 0x7ff00000 && lo == 0` is +INF, `hi == 0xfff00000 && lo == 0` is -INF, `(hi >> 16) & 0x7ff8 == 0x7ff8` is a quiet NaN, an all-ones exponent with a non-zero mantissa and |
| 0x0062ada9 | `math_frexp` | - | - | - | returns 0 with exponent 0 for a zero input; for a denormal it shifts the 96-bit pattern left until the implicit bit appears, counting down from -0x3fd, restores the sign and calls |
| 0x0062ae6a | `math_x87_read_status_word` | - | - | - | `wait; fnstsw [ebp-2]; movsx eax, word [ebp-2]` |
| 0x0062ae78 | `math_x87_read_and_clear_status_word` | - | - | - | `fnstsw [ebp-2]; fnclex; movsx eax` |
| 0x0062ae87 | `math_x87_merge_control_word` | - | - | - | `fnstcw` into a local, replaces the bits selected by the mask with those from the new value, `fldcw`s the result and returns the PREVIOUS raw word |
| 0x0062aeaa | `math_x87_raise_status_flags` | - | - | - | for each bit of the argument it executes the operation that sets the corresponding sticky flag: 0x01 INVALID (`fistp` of an out-of-range value), 0x04 ZERODIVIDE (`fldz; fld1; fdivrp`), 0x08 OVERFLOW |
| 0x0062af00 | `crt_filbuf` | - | - | - | MSVC `_filbuf` - refill a `FILE`'s read buffer and return its first byte |
| 0x0062afd9 | `crt_read` | - | - | - | validates the fd against `g_crt_handle_count` and the FOPEN bit, replays a pushed-back character from the `ioinfo`, calls `ReadFile`, maps a failure through `crt_dosmaperr` (treating |
| 0x0062b1cf | `crt_dosmaperr` | - | - | - | stores the raw code in `g_crt_doserrno`, then scans the 44 `{code, errno}` pairs of `g_crt_dos_errno_map` (`0x00652008`, loop bound `0x652170` in the code); on a miss it falls back to EACCES for |
| 0x0062b251 | `math_exp_finite_argument_arm` | - | - | - | arm 0 of the dead `exp` object, `e**x = 2**(x * log2 e)`, §9i |
| 0x0062b344 | `thunk_math_x87_return_zero` | - | - | - | nothing of its own - it forwards to `math_x87_return_zero` (0x00628486), which pops the x87 stack and pushes +0.0 |
| 0x0062b391 | `math_exp_infinite_argument_arm` | - | - | - | returns +INF (the 80-bit constant at `0x00652170`) when the caller's sign byte in CL is zero, and otherwise jumps to `thunk_math_x87_return_zero` - i.e |
| 0x0062b39e | `math_exp_fraction_core` | - | - | - | compares `/x/` with the 80-bit threshold at `0x0065218e` (~2**15) and jumps to the overflow arm when it is exceeded; otherwise it rounds to an integer, subtracts to get the fraction, records both |
| 0x0062b3e1 | `math_x87_integer_parity` | - | - | - | compares `frndint(x)` with x; if they differ returns 0 |
| 0x0062b415 | `math_pow_special_case_x87_wrapper` | - | - | - | reserves 0x74 bytes, spills ST(1) and ST(0) to the stack as two doubles, `fnsave`s the whole FPU state into the scratch area, calls `math_pow_special_case_result`, `frstor`s, reloads the result and - |
| 0x0062b447 | `math_pow_special_case_result` | - | - | - | handles y = +INF and y = -INF against `/x/` less than, equal to and greater than 1; x = +INF and x = -INF with the sign taken from `math_double_integer_parity(y)`; and x = 0 |
| 0x0062b577 | `math_double_integer_parity` | - | - | - | classifies the operand first, then tests `__frnd(x) == x`; if that holds it tests `__frnd(x / 2) == x / 2` to separate even from odd |
| 0x0062b5dc | `crt_sbh_heap_init` | - | - | - | `HeapAlloc`s the 0x140-byte region-header array (16 entries of 0x14 bytes), zeroes the count and the deferred-group state, points the allocation cursor at the start and records the caller's size |
| 0x0062b624 | `crt_sbh_find_block` | - | - | - | walks the 0x14-byte headers testing `(p - header->base) < 0x100000`, the size of one reserved region |
| 0x0062b64f | `crt_sbh_free_block` | - | - | - | coalesces with the preceding and following free blocks, unlinks them from their size buckets and maintains both 32-bit bucket bitmaps plus the per-group counters; when a group falls empty it defers |
| 0x0062b978 | `crt_sbh_alloc_block` | - | - | - | rounds the request to a 16-byte multiple plus the 8-byte header, builds the two bucket masks for that size class, searches from the allocation cursor forward then from the start, commits a new group |
| 0x0062bc81 | `crt_sbh_alloc_new_region` | - | - | - | grows the header array by 16 entries with `HeapReAlloc` when it is full, `HeapAlloc`s the 0x41c4-byte group map, `VirtualAlloc`s 0x100000 bytes as MEM_RESERVE, marks every group uncommitted and bumps |
| 0x0062bd32 | `crt_sbh_alloc_new_group` | - | - | - | picks the first uncommitted group from the region's bitmap, `VirtualAlloc`s 0x8000 bytes at `base + index * 0x8000`, initialises the 64 bucket list heads, lays out 0x1c00 free 0xff0-byte blocks with |
| 0x0062be2d | `crt_sbh_resize_block` | - | - | - | when growing it requires the following block to be free and large enough, unlinks it, splits off the remainder and re-files it; when shrinking it splits the tail off, coalescing it with a following |
| 0x0062c123 | `crt_old_sbh_new_region` | - | - | - | reuses the statically allocated header `g_crt_old_sbh_static_region` when its base field is -1, else `HeapAlloc`s a 0x2020-byte header and links it into the circular region list; `VirtualAlloc`s |
| 0x0062c267 | `crt_old_sbh_release_region` | - | - | - | `VirtualFree`s the reserve, moves the allocation cursor off it, unlinks it from the circular list and `HeapFree`s the header - or, for the static header, simply writes -1 into its base field to mark |
| 0x0062c2bd | `crt_old_sbh_decommit_pages` | - | - | - | walks the region list from the allocation cursor, scanning each page map from the top; a page whose free counter is the full 0xf0 is `VirtualFree`d with MEM_DECOMMIT, marked -1 and subtracted from |
| 0x0062c37f | `crt_old_sbh_find_block` | - | - | - | walks the circular region list comparing the pointer against each `[base, end)` window, then rejects anything not 16-byte aligned or sitting inside the page header, and converts the offset into the |
| 0x0062c3d6 | `crt_old_sbh_free_block` | - | - | - | adds the block's paragraph count back to the owning page's free counter, zeroes its size byte, marks the page dirty, and when the page becomes wholly free bumps `g_crt_old_sbh_free_page_count` - |
| 0x0062c41b | `crt_old_sbh_alloc_block` | - | - | - | walks the region list from the allocation cursor, trying every page whose free counter can hold the request through `crt_old_sbh_alloc_from_page`; when none can, it commits the next run of up to 16 |
| 0x0062c623 | `crt_old_sbh_alloc_from_page` | - | - | - | each byte of the map is a block's paragraph count and 0 means free, so it advances by the stored count over allocated runs, measures each free run, and takes the first run large enough - updating the |
| 0x0062c747 | `crt_old_sbh_resize_block` | - | - | - | shrinking just rewrites the size byte and gives the difference back to the page; growing requires every byte-map slot up to the new end to be free and inside the page, then claims them and moves the |
| 0x0062c7f0 | `crt_call_new_handler` | - | - | - | calls `g_crt_new_handler(size)` when one is installed and reports whether it claims the caller should retry |
| 0x0062c80b | `crt_get_image_linker_version` | - | - | - | `GetModuleHandleA(NULL)`, checks the `MZ` signature, follows `e_lfanew` and copies `MajorLinkerVersion` and `MinorLinkerVersion` from `IMAGE_OPTIONAL_HEADER + 2` into the caller's two-byte buffer |
| 0x0062c838 | `crt_heap_select` | - | - | - | returns 1 (the Win32 heap alone) on Windows 2000 or later; otherwise it honours the `__MSVCRT_HEAP_SELECT` environment variable - whose value is either `__GLOBAL_HEAP_SELECTED,n` or a per-executable |
| 0x0062c980 | `crt_heap_init` | - | - | - | `HeapCreate` (serialised unless the caller asks for the single-threaded heap), then `crt_heap_select` to choose between the Win32 heap alone (1), the V5 small-block heap (2) and the V6 one (3), |
| 0x0062c9dd | `crt_local_time_to_time_t` | - | - | - | range-gates the year to 1970..2038, adds the cumulative days of `g_crt_days_before_month` plus the leap-day correction, folds hours, minutes and seconds in, then adds `g_crt_timezone` and - when |
| 0x0062ca9f | `crt_exception_filter` | - | - | - | looks the exception code up in `g_crt_xcpt_action_table` through `crt_xcpt_action_lookup` |
| 0x0062cbe0 | `crt_xcpt_action_lookup` | - | - | - | linear scan of the 12-byte `{code, signal, handler}` records at `g_crt_xcpt_action_table`, bounded by `g_crt_xcpt_action_count` |
| 0x0062cc23 | `crt_get_cmdline_args` | - | - | - | MSVC `_wincmdln`: the ARGUMENT tail of the command line |
| 0x0062cc7b | `crt_build_environ` | - | - | - | makes sure the multibyte tables are up (`g_crt_mbctype_initialized` gate), counts the `name=value` strings in `g_crt_environment_block` while SKIPPING the `=X:` per-drive current-directory entries, |
| 0x0062cd34 | `crt_setup_argv` | - | - | - | fetches the module file name into `g_crt_module_filename` and points `g_crt_pgmptr` at it, picks the command line (falling back to the module name when `g_crt_command_line` is empty), calls |
| 0x0062cdcd | `crt_parse_cmdline` | - | - | - | handles the program name (quoted or blank-terminated) and then each argument under the backslash/quote rules, honouring MBCS lead bytes through `g_crt_mbctype` |
| 0x0062cf81 | `crt_get_environment_strings_ansi` | - | - | - | on the first call it probes `GetEnvironmentStringsW` and records the verdict in `g_crt_environment_api_mode` (1 wide, 2 ANSI-only) |
| 0x0062d0b3 | `crt_init_file_handles` | - | - | - | `malloc`s the first 0x100-byte `ioinfo` block (32 entries), sets `g_crt_handle_count` to 32 and initialises every entry to an invalid handle with a '\n' pushback slot |
| 0x0062d25e | `crt_write_runtime_error_banner` | - | - | - | runs only when `g_crt_error_mode == 1` or it is 0 and `g_crt_app_type == 1` (a console application); writes runtime-error message 0xfc (the CR/LF), calls `g_crt_error_banner_hook` if one is |
| 0x0062d297 | `crt_write_runtime_error_message` | - | - | - | MSVC `_NMSG_WRITE` - print one numbered C runtime error |
| 0x0062d3f0 | `_strcmp` | - | - | - | aligns the first pointer to a 4-byte boundary one or two bytes at a time, then compares four bytes per iteration, checking each byte for inequality and for the terminating NUL before moving on |
| 0x0062d474 | `crt_cxx_unhandled_exception_filter` | - | - | - | MSVC `__CxxUnhandledExceptionFilter` - turn an escaped C++ |
| 0x0062d4ba | `crt_cxx_install_unhandled_exception_filter` | - | - | - | `g_crt_prev_unhandled_exception_filter = SetUnhandledExceptionFilter(crt_cxx_unhandled_exception_filter)` |
| 0x0062d4cb | `crt_cxx_restore_unhandled_exception_filter` | - | - | - | `SetUnhandledExceptionFilter(g_crt_prev_unhandled_exception_filter)` - puts back the filter that `crt_cxx_install_unhandled_exception_filter` replaced at startup |
| 0x0062d4d8 | `crt_control87` | - | - | - | `fnstcw` into a local, decodes it to the `_MCW_*` flag set with `crt_fp_control_word_decode`, merges `(new & mask)` over the bits the mask selects, re-encodes with `crt_fp_control_word_encode` and |
| 0x0062d50d | `crt_controlfp` | - | - | - | forwards to `crt_control87` with `mask & ~0x80000`, i.e |
| 0x0062d523 | `crt_fp_control_word_decode` | - | - | - | IM 0x01 to `_EM_INVALID` 0x10, DM 0x02 to `_EM_DENORMAL` 0x80000, ZM 0x04 to `_EM_ZERODIVIDE` 0x08, OM 0x08 to `_EM_OVERFLOW` 0x04, UM 0x10 to `_EM_UNDERFLOW` 0x02, PM 0x20 to `_EM_INEXACT` 0x01; |
| 0x0062d5b5 | `crt_fp_control_word_encode` | - | - | - | the exact inverse of `crt_fp_control_word_decode`, bit for bit |
| 0x0062d63e | `crt_ld12_bits_below_are_zero` | - | - | - | masks off the bits at and above N in the containing dword and then requires the remaining dwords of the 96-bit mantissa to be zero - the exact-tie test round-half-to-even needs |
| 0x0062d687 | `crt_ld12_add_one_at_bit` | - | - | - | adds the single bit into its dword through the shared add-with-carry helper, then keeps adding 1 to each higher dword while the carry survives |
| 0x0062d6dd | `crt_ld12_round_at_bit` | - | - | - | when the guard bit is set and the bits below it are not all zero it increments at bit N-1 (round half to even via `crt_ld12_bits_below_are_zero`), then masks the mantissa off at N and zeroes the |
| 0x0062d769 | `crt_ld12_mantissa_copy` | - | - | - | three dword stores; Ghidra renders the destination as a biased pointer |
| 0x0062d784 | `crt_ld12_mantissa_clear` | - | - | - | three zero stores |
| 0x0062d790 | `crt_ld12_mantissa_is_zero` | - | - | - | requires all three dwords to be zero |
| 0x0062d7ab | `crt_ld12_mantissa_shift_right` | - | - | - | does the sub-dword shift with carry-in across the three dwords first, then moves whole dwords down by `N / 32`, zero-filling from the top |
| 0x0062d838 | `crt_ld12_to_ieee` | - | - | - | reads the 80-bit exponent and sign, treats an all-zero exponent as zero or a denormal, rounds the mantissa to the target's bit count, and then classifies the result against the descriptor's exponent |
| 0x0062d9a4 | `crt_ld12_to_double` | - | - | - | `crt_ld12_to_ieee` with the width-0x40 descriptor at `g_crt_ld12_to_double_descriptor` |
| 0x0062d9ba | `crt_ld12_to_float` | - | - | - | `crt_ld12_to_ieee` with the width-0x20 descriptor at `g_crt_ld12_to_float_descriptor` |
| 0x0062d9d0 | `crt_atodbl` | - | - | - | `crt_string_to_ld12` into a stack `_LDBL12`, then `crt_ld12_to_double` |
| 0x0062d9fd | `crt_atoflt` | - | - | - | `crt_string_to_ld12` into a stack `_LDBL12`, then `crt_ld12_to_float` |
| 0x0062da2a | `crt_fptostr` | - | - | - | writes a leading '0' and then `ndec` digits from the record's mantissa, padding with '0' once it runs out; rounds up when the next digit is '5' or more, carrying through trailing nines; if the carry |
| 0x0062daa1 | `crt_fltout` | - | - | - | widens the value with `crt_double_to_ld10`, calls the 17-digit decimal converter into `g_crt_fltout_decimal_exponent` / `_sign_char` / `_digits`, and fills the four fields of `g_crt_fltout_strflt` |
| 0x0062db05 | `crt_double_to_ld10` | - | - | - | maps an all-zero exponent to a true zero or, for a denormal, to exponent 0x3c01 with the implicit bit cleared; maps 0x7ff to 0x7fff; otherwise adds the bias difference 0x3c00, then shifts the 53-bit |
| 0x0062dbc0 | `_memset` | - | - | - | for counts under four it stores bytes; otherwise it aligns the destination to a 4-byte boundary byte by byte, broadcasts the fill value with `value * 0x01010101`, stores whole dwords, and finishes |
| 0x0062dc18 | `crt_fp_not_loaded_trap` | - | - | - | `push 2; call __amsg_exit` - runtime-error number 2 is MSVC's `_RT_FLOAT`, "floating point support not loaded" |
| 0x0062dc21 | `crt_is_readable_pointer` | - | - | - | one Win32 call, result inverted |
| 0x0062dc3d | `crt_is_writable_pointer` | - | - | - | one Win32 call, result inverted |
| 0x0062dc59 | `crt_is_executable_pointer` | - | - | - | one Win32 call, result inverted |
| 0x0062dc71 | `crt_abort` | - | - | - | `crt_write_runtime_error_message(10)` - runtime-error 10 is MSVC's `_RT_ABORT` - then `raise(0x16)` (SIGABRT) and `_exit(3)` |
| 0x0062dc88 | `math_raise_fp_exception_for_type` | - | - | - | maps the record's `type` to the internal flag set (1 and 5 to 8, 2 to 4, 3 to 0x11, 4 to 0x12, 8 to 0x10, 7 rewritten to 1 and skipped), applies the masked IEEE defaults through |
| 0x0062dd58 | `crt_string_to_ld12` | - | - | - | `__strgtold12` with its twelve-entry jump table, §9a |
| 0x0062e229 | `crt_getstringtype_ansi` | - | - | - | `__crtGetStringTypeA` |
| 0x0062e372 | `crt_lseek` | - | - | - | validates the fd, fetches the OS handle with `crt_get_osfhandle`, calls `SetFilePointer`, maps a failure through `crt_dosmaperr` and clears the stream's end-of-file flag on success |
| 0x0062e40c | `crt_getbuf` | - | - | - | counts the acquisition in `g_crt_buffered_stream_count`, `malloc`s 0x1000 bytes and sets `_IOMYBUF`; on failure it falls back to the two-byte buffer inside the `FILE` itself and marks the stream |
| 0x0062e450 | `crt_isatty` | - | - | - | returns the FDEV bit (0x40) of the descriptor's `ioinfo`, or 0 when the fd is out of range |
| 0x0062e476 | `crt_initstdio` | - | - | - | clamps `g_crt_stream_count` to 20 (or 512 when it is zero), `calloc`s `g_crt_stream_table`, points the first twenty slots at the static `_iob` array starting at `g_crt_iob_stdin` (stride 0x20), and |
| 0x0062e51b | `crt_endstdio` | - | - | - | `crt_flushall()`; then, when the exit-mode flag at 0x006f97b0 is set, tail-jumps to `crt_fcloseall` |
| 0x0062e52f | `crt_wctomb` | - | - | - | in the C locale it stores the low byte when the value fits in 8 bits; otherwise it calls `WideCharToMultiByte` with `g_crt_lc_codepage` and rejects the conversion when a default character had to be |
| 0x0062e5a0 | `__aullrem` | - | - | - | when the divisor fits in 32 bits it does the classic two-step 64/32 division and keeps the remainder; otherwise it shifts both operands right until the divisor fits, divides, multiplies the quotient |
| 0x0062e615 | `crt_alloc_osfhandle_slot` | - | - | - | scans each committed `ioinfo` block for an entry without the FOPEN bit; when every block is full it `malloc`s another 0x100-byte block of 32 entries, initialises them and raises `g_crt_handle_count` |
| 0x0062e6aa | `crt_set_osfhandle` | - | - | - | stores the handle in the slot, and for descriptors 0, 1 and 2 in a console application (`g_crt_app_type == 1`) mirrors it with `SetStdHandle` |
| 0x0062e721 | `crt_free_osfhandle_slot` | - | - | - | clears the corresponding standard handle first when the descriptor is 0, 1 or 2 in a console application, then writes -1 into the slot |
| 0x0062e79b | `crt_get_osfhandle` | - | - | - | checks the descriptor against `g_crt_handle_count` and the FOPEN bit, then returns the stored handle; otherwise sets EBADF |
| 0x0062e7d8 | `crt_commit` | - | - | - | `FlushFileBuffers` on the descriptor's handle, reporting EBADF with the captured `GetLastError` on failure |
| 0x0062e82f | `crt_sopen` | - | - | - | translates the `_O_*` flags into `CreateFileA` access, share mode, creation disposition and attributes; applies `g_crt_umask` to the permission argument for `_O_CREAT`; takes a descriptor slot, |
| 0x0062eae8 | `crt_mbtowc` | - | - | - | in the C locale it widens the byte directly; otherwise it consults `g_crt_pctype` for a lead byte and calls `MultiByteToWideChar` with either one byte or `g_crt_mb_cur_max`, failing with EILSEQ on a |
| 0x0062ebb0 | `crt_ungetc` | - | - | - | rejects EOF and a stream that is neither reading nor readable-after-write, acquires a buffer if the stream has none, backs `_ptr` up (refusing when the pushback would leave the buffer) and, for a |
| 0x0062ec1e | `crt_matherr_default` | - | - | - | `xor eax, eax; ret` — returns 0, which in the C library's contract means "I did not handle this error", telling the caller to apply the default behaviour and set `errno` |

#### Band 5, `[0x0062ec30, 0x00634fe0)` without the crash reporter

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0062ec30 | `math_x87_fdiv_erratum_divide_core` | - | - | - | loads two 80-bit operands from the caller's spill slots `[esp+4]` and `[esp+0x10]`, normalises a denormal by multiplying by `g_crt_fdiv_normalize_scale` (2**63) in a loop, classifies each operand |
| 0x0062ed47 | `math_x87_fdiv_erratum_register_divide_dispatch` | - | - | - | `sub esp,0x2c; and eax,0x3f; jmp [eax*4 + 0x0065464e]` (`g_crt_fdiv_register_form_dispatch_table`) |
| 0x0062f1e6 | `math_x87_fdiv_erratum_divide` | - | - | - | `sub esp,0x2c; fstp xword[esp]; fstp xword[esp+0xc]; call the core; add esp,0x2c` |
| 0x0062f1f9 | `math_x87_fdiv_erratum_reverse_divide` | - | - | - | reserves 0x2c bytes, stores ST(0) at [esp+0xc] as the dividend and ST(1) at [esp] as the divisor - the opposite order to `math_x87_fdiv_erratum_divide` - and calls `math_x87_fdiv_erratum_divide_core` |
| 0x0062f20c | `math_x87_fdiv_erratum_divide_by_float` | - | - | - | when the operand's exponent field is all ones (`0x7f800000`) the value is Inf or NaN and the hardware `fdiv dword [esp+8]` is used directly; otherwise it loads the operand and routes through the |
| 0x0062f258 | `math_x87_fdiv_erratum_divide_by_double` | - | - | - | identical in shape to the float variant with the DOUBLE exponent mask (`0x7ff00000`), an 8-byte operand and `ret 8`; fast path `fdiv qword [esp+8]` |
| 0x0062f2a4 | `math_x87_fdiv_erratum_divide_by_int16` | - | - | - | `push eax`;  reads the x87 status word and, when the stack has no free slot, parks ST(0) in a 12-byte frame first; then `fild word` loads the operand and `math_x87_fdiv_erratum_divide` divides |
| 0x0062f2d8 | `math_x87_fdiv_erratum_divide_by_int32` | - | - | - | `push eax`;  reads the x87 status word and, when the stack has no free slot, parks ST(0) in a 12-byte frame first; then `fild dword` loads the operand and `math_x87_fdiv_erratum_divide` divides |
| 0x0062f30c | `math_x87_fdiv_erratum_reverse_divide_by_float` | - | - | - | `push eax`; if the float's exponent is all ones it runs the plain `fdivr dword` and returns |
| 0x0062f358 | `math_x87_fdiv_erratum_reverse_divide_by_double` | - | - | - | `push eax`; if the double's exponent is all ones it runs the plain `fdivr qword` and returns |
| 0x0062f3a4 | `math_x87_fdiv_erratum_reverse_divide_by_int16` | - | - | - | `push eax`;  reads the x87 status word and, when the stack has no free slot, parks ST(0) in a 12-byte frame first; then `fild word` loads the operand and `math_x87_fdiv_erratum_reverse_divide` divides |
| 0x0062f3d8 | `math_x87_fdiv_erratum_reverse_divide_by_int32` | - | - | - | `push eax`;  reads the x87 status word and, when the stack has no free slot, parks ST(0) in a 12-byte frame first; then `fild dword` loads the operand and `math_x87_fdiv_erratum_reverse_divide` |
| 0x0062f40c | `math_x87_fdiv_erratum_fdivp` | - | - | - | `push eax`, reserves 0x2c bytes, stores ST(0) at [esp] as the divisor and ST(1) at [esp+0xc] as the dividend, calls `math_x87_fdiv_erratum_divide_core`, releases the frame and `pop eax` |
| 0x0062f421 | `math_x87_fdiv_erratum_fdivrp` | - | - | - | `push eax`, reserves 0x2c bytes, stores ST(0) at [esp+0xc] as the dividend and ST(1) at [esp] as the divisor, calls `math_x87_fdiv_erratum_divide_core`, releases the frame and `pop eax` |
| 0x0062f436 | `math_x87_fdiv_erratum_fprem_core` | - | - | - | classifies both operands, rejects Inf/NaN combinations, and reduces the dividend by repeated subtraction of the scaled divisor - halving through `g_crt_fprem_half` - until the exponent difference is |
| 0x0062f63c | `math_x87_fdiv_erratum_fprem_scale_denormal` | - | - | - | saves EDX, stores both operands (divisor ST(0) at [esp], dividend at [esp+0x18]) |
| 0x0062f6ee | `math_x87_fdiv_erratum_fprem1_core` | - | - | - | byte-for-byte the `fprem` core above apart from the instruction it wraps, which is `fprem1` (round-to-nearest quotient) instead of `fprem` (truncated quotient) |
| 0x0062f8f4 | `math_x87_fdiv_erratum_fprem1_scale_denormal` | - | - | - | the `fprem1` copy of `math_x87_fdiv_erratum_fprem_scale_denormal`, calling `math_x87_fdiv_erratum_fprem1_core` on the normal and denormal paths |
| 0x0062f9a9 | `math_x87_fdiv_erratum_fprem` | - | - | - | `call math_x87_fdiv_erratum_fprem_scale_denormal; ret` - the compiler-callable entry of the remainder path |
| 0x0062f9af | `math_x87_fdiv_erratum_fprem1` | - | - | - | `call math_x87_fdiv_erratum_fprem1_scale_denormal; ret` |
| 0x0062f9b5 | `math_x87_fdiv_erratum_fpatan` | - | - | - | `fpatan; ret` |
| 0x0062f9b8 | `math_x87_fdiv_erratum_fptan` | - | - | - | `fptan; ret` |
| 0x0062f9bb | `math_fpclass_double` | - | - | - | an all-ones exponent goes to `math_classify_double_special` and maps 1/2/3 to +INF / -INF / quiet NaN and anything else to signalling NaN; a zero exponent with a non-zero mantissa is a denormal; a |
| 0x0062fa4d | `crt_tzset_once` | - | - | - | `if (g_crt_tzset_done == 0) { crt_tzset(); g_crt_tzset_done++; }` |
| 0x0062fa62 | `crt_tzset` | - | - | - | reads `getenv("TZ")`; when set it caches the string in `g_crt_tz_env_cached`, copies the 3-letter standard abbreviation, parses `[+-]hh[:mm[:ss]]` into `g_crt_timezone` and stores the daylight |
| 0x0062fcc0 | `crt_is_in_dst` | - | - | - | returns false immediately when `g_crt_daylight` is 0 |
| 0x0062fe6c | `crt_dst_rule_to_yearday` | - | - | - | for the day-in-month form it finds the year's weekday for the first of the month and steps to the requested week and weekday, clamping week 5 to the last occurrence; for the absolute form it adds the |
| 0x0062ffac | `crt_ismbblead` | - | - | - | `crt_ismbbtype(c, 0, 4)` |
| 0x0062ffbd | `crt_ismbbtype` | - | - | - | `return (g_crt_mbctype[c+1] & mbmask) // (ctype[c] & mask)` - the MBCS table first, the C ctype table at 0x00651cc0 second |
| 0x0062ffee | `crt_setmbcp` | - | - | - | C `_setmbcp` - select the multibyte codepage and rebuild the MBCS tables |
| 0x00630187 | `crt_resolve_mbcs_codepage` | - | - | - | -2 returns `GetOEMCP()`, -3 returns `GetACP()`, -4 returns `g_crt_lc_codepage`, anything else is passed through |
| 0x006301d1 | `crt_lcid_for_mbcs_codepage` | - | - | - | 932 -> 0x411 (ja-JP), 936 -> 0x804 (zh-CN), 949 -> 0x412 (ko-KR), 950 -> 0x404 (zh-TW); anything else 0 |
| 0x00630204 | `crt_mbcs_set_single_byte` | - | - | - | zeroes all 257 bytes of `g_crt_mbctype` and the five codepage globals (`g_crt_mbcs_codepage`, `g_crt_mbcs_is_multibyte`, `g_crt_mbcs_lcid` and the three-dword `g_crt_mbcs_case_ranges`), so every byte |
| 0x0063022d | `crt_mbcs_build_case_tables` | - | - | - | builds a 256-entry identity string with the lead-byte ranges blanked to spaces, then asks Windows once for the character types (`crt_getstringtype_ansi(CT_CTYPE1)`) and twice for the mapped forms |
| 0x006303b2 | `crt_mbcs_init_once` | - | - | - | `if (!g_crt_mbctype_initialized) { crt_setmbcp(_MB_CP_ANSI); g_crt_mbctype_initialized = 1; }` |
| 0x006303ce | `crt_message_box` | - | - | - | on first use `LoadLibraryA("user32.dll")` and caches `MessageBoxA`, `GetActiveWindow` and `GetLastActivePopup`; then owns the box to the active popup of the active window so it cannot appear behind |
| 0x00630457 | `crt_ld12_add_dword_carry` | - | - | - | `*sum = x + y; return (sum < x // sum < y)` |
| 0x00630478 | `___add_12` | - | - | - | adds the three dwords low to high through `crt_ld12_add_dword_carry`, propagating each carry into the next word - and, when the lowest add carries, adding 1 into the middle word first so a double |
| 0x006304d6 | `crt_ld12_shift_left_1` | - | - | - | carries bit 31 of each dword into the next: `[0] <<= 1`, `[1] = [1]<<1 / [0]>>31`, `[2] = [2]<<1 / [1]>>31` |
| 0x00630504 | `crt_ld12_shift_right_1` | - | - | - | the exact inverse of `crt_ld12_shift_left_1`, carrying bit 0 of each dword down into the next |
| 0x00630531 | `crt_ld12_from_decimal_digits` | - | - | - | per digit it computes `x = x*10 + digit` as shl, shl, add-the-original, shl - which is `((x*2)*2 + x)*2` - through `crt_ld12_shift_left_1` and `___add_12`; then seeds the exponent field to 0x404e and |
| 0x006305f8 | `crt_ld10_to_decimal_digits` | - | - | - | the 10-byte long double to decimal digits converter |
| 0x0063088b | `crt_raise` | - | - | - | resolves the handler slot: SIGINT (2), SIGBREAK (0x15), SIGABRT (0x16) and SIGTERM (0xf) each have their own global, while SIGILL (4), SIGFPE (8) and SIGSEGV (0xb) come from the `_XcptActTab` record |
| 0x006309b8 | `crt_xcpt_action_lookup_by_signal` | - | - | - | linear scan of the same 12-byte `{exception code, signal, handler}` records at `g_crt_xcpt_action_table`, bounded by `g_crt_xcpt_action_count`, but keyed on field [1] instead of field [0] |
| 0x006309fd | `crt_ld12_multiply` | - | - | - | rejects the Inf/NaN exponents and the underflow window, then accumulates the 5x5 word cross-product through `crt_ld12_add_dword_carry`, adds the exponents with the 0xc002 bias, normalises with |
| 0x00630c1d | `crt_ld12_multiply_by_power_of_ten` | - | - | - | decomposes /n/ three bits at a time and multiplies by the pre-computed power of ten through `crt_ld12_multiply` |
| 0x00630c99 | `crt_calloc` | - | - | - | rounds the request up to a 16-byte multiple, then takes the same three-way `g_crt_heap_mode` branch as the rest of the allocator: the V6 small-block heap under `g_crt_sbh_threshold`, the V5 one under |
| 0x00630d4a | `crt_fcloseall` | - | - | - | walks `g_crt_stream_table` from index 3 to `g_crt_stream_count`, calls `crt_fclose` on each stream whose flags carry `_IOREAD/_IOWRT/_IOMYBUF` (0x83), counts the successes, and frees the dynamically |
| 0x00630da2 | `crt_chsize` | - | - | - | seeks to the end to learn the current size; to GROW it forces binary mode with `crt_setmode`, writes 4 KB blocks of zeros through `crt_write` and restores the mode; to SHRINK it seeks to the target |
| 0x00630ee8 | `crt_getenv` | - | - | - | materialises `g_crt_environ` from `g_crt_wide_environ` on first use, then scans it comparing the first `strlen(name)` bytes with `__mbsnbicoll` and requiring the next byte to be `'='` |
| 0x00630f65 | `crt_setmode` | - | - | - | 0x8000 (`_O_BINARY`) clears the FTEXT bit 0x80 in the descriptor's osfhandle flags and 0x4000 (`_O_TEXT`) sets it; anything else is EINVAL. The return value is reconstructed from the PREVIOUS state |
| 0x00630fdb | `__mbsnbicoll` | - | - | - | returns 0 for a zero count; otherwise calls `crt_comparestring_ansi(g_crt_mbcs_lcid, NORM_IGNORECASE, s1, n, s2, n, g_crt_mbcs_codepage)` and normalises the Windows 1/2/3 answer to the C -1/0/+1 by |
| 0x0063101a | `crt_environ_from_wide` | - | - | - | walks `g_crt_wide_environ`, converts each `name=value` with `WideCharToMultiByte(CP_OEMCP)` into a fresh `malloc` and hands it to `crt_setenv(str, 0)` so the ANSI array is built through the normal |
| 0x00631088 | `crt_comparestring_ansi` | - | - | - | `__crtCompareStringA` |
| 0x006312ae | `crt_comparestring_ansi_except_filter` | - | - | - | `return 1` - EXCEPTION_EXECUTE_HANDLER, so a fault inside the Windows call unwinds into the handler that returns 0 rather than propagating |
| 0x00631305 | `crt_strnlen` | - | - | - | walks at most n bytes looking for the NUL; returns the distance when it finds one and n when it does not |
| 0x00631330 | `crt_setenv` | - | - | - | rejects a string with no `'='` or an empty name |
| 0x006314b7 | `crt_environ_find_index` | - | - | - | compares the first `len` bytes of each entry with `__mbsnbicoll` and requires the next byte to be `'='` or NUL. The two outcomes share one return value: a non-negative index on a hit, and the NEGATED |
| 0x0063150f | `crt_environ_duplicate` | - | - | - | counts the entries, mallocs `(n+1)*4` and `crt_strdup`s each string; a failed array allocation is fatal (`__amsg_exit(9)` = R6009, out of environment space) |
| 0x00631576 | `crt_mbschr` | - | - | - | delegates to `strchr` when `g_crt_mbcs_is_multibyte` is 0 |
| 0x006315e9 | `crt_strdup` | - | - | - | `strlen` + `malloc(len+1)` + `crt_strcpy`; returns 0 for a NULL argument and for a failed allocation alike |
| 0x006346c0 | `RtlUnwind` | - | - | - | import thunk, `kernel32.dll` |
| 0x006346c6 | `Module32Next` | - | - | - | import thunk, `kernel32.dll` |
| 0x006346cc | `Module32First` | - | - | - | import thunk, `kernel32.dll` |
| 0x006346d2 | `CreateToolhelp32Snapshot` | - | - | - | import thunk, `kernel32.dll` |
| 0x006346e0 | `crt_stricmp` | - | - | - | the unbounded sibling of `crt_strnicmp` |
| 0x00634770 | `crt_strnicmp` | - | - | - | an ASCII fast path while `g_crt_mbcs_locale_active` is clear - folding only 'A'..'Z' by adding 0x20 - and a per-character path through `char_to_lower_locale` when it is set |
| 0x0063481d | `crt_itoa` | - | - | - | sets the negative flag only when the radix is 10, then tail-calls `crt_xtoa` which does the digits |
| 0x0063484a | `crt_xtoa` | - | - | - | emits digits least-significant first with a `'0'` bias below 10 and a `'W'` bias above it (so 10 becomes `'a'`), NUL-terminates, then reverses the string in place |
| 0x006348a6 | `crt_spawnlp` | - | - | - | `_spawnlp`: forwards its variable argument list to `crt_spawnvpe` |
| 0x006348be | `crt_chdir` | - | - | - | `SetCurrentDirectoryA`, then reads the result back with `GetCurrentDirectoryA` and - unless the path is a bare root - records it in the per-drive environment variable `=X:` via |
| 0x00634944 | `crt_spawnvpe` | - | - | - | prepends `g_crt_spawn_curdir_prefix` (`.\\`) when the name carries neither a path separator nor a drive colon; when it carries no extension it walks `g_crt_spawn_extensions` downward - `.com`, |
| 0x00634a8d | `crt_spawnve` | - | - | - | `crt_cenvarg` to flatten argv and envp into two heap blocks, `crt_dospawn` to run the child, then frees both blocks whatever the outcome |
| 0x00634ade | `crt_mbctoupper` | - | - | - | single-byte characters go through `g_crt_mbctype`'s bit 0x20 and the `g_crt_mbcs_case_map` table; lead-byte pairs go through `crt_lcmapstring_ansi(..., 0x200, ...)` |
| 0x00634b59 | `crt_access` | - | - | - | `GetFileAttributesA`; a failure goes through `crt_dosmaperr`, and `FILE_ATTRIBUTE_READONLY` meeting a write request yields EACCES (0xd) with `g_crt_doserrno` = 5 |
| 0x00634b9d | `crt_mbsrchr` | - | - | - | delegates straight to `_strrchr` when `g_crt_mbcs_is_multibyte` is clear; otherwise it walks the string a character at a time, skipping trail bytes, so a trail byte that happens to equal the needle |
| 0x00634bfd | `crt_dospawn` | - | - | - | packs the inheritable file descriptors from `g_crt_osfhandle_blocks` into the `STARTUPINFO.lpReserved2` block, joins the argument block into one command line, calls `CreateProcessA`, then acts on the |
| 0x00634ddc | `crt_cenvarg` | - | - | - | sums the argument lengths, allocates one block and joins them with spaces; for the environment it first copies the `=X:=...` drive-current-directory entries out of `g_crt_environment_block` (fetching |

## 4. The bands

| Band | Span | Starts | What is in it |
|---|---|---|---|
| 1 | `[0x006243b7, 0x00625021)` | 64 | the `<stdexcept>` classes, the last import thunks, FP init, the exit handlers, the C++ EH / SEH leaf helpers, the acos x87 entry |
| 2 | `[0x00625021, 0x00627686]` | 80 | stdio, `qsort`, the string and ctype routines, `memcpy` / `memmove`, the heap, `pow` and `asin`, the startup group, `exception` and `type_info` |
| 3 | `[0x006276dc, 0x0062ad14]` | 86 | the `cvt.c` tail, the whole C++ EH engine, the x87 FP error dispatch, `_output`, `_input`, the math-error machinery |
| 4 | `[0x0062ad26, 0x0062ec1e]` | 96 | both small-block heaps, the startup argv / environment group, the 12-byte long double, `ieee87`, `osfinfo`, `_matherr` |
| 5 | `[0x0062ec30, 0x00634fe0)` | 151 ¹ | the Pentium FDIV-erratum object, `tzset` / `mbctype` / the decimal converters, `calloc`, `getenv` / `putenv`, and the classic iostream |

¹ Band 5's starts break down as **123 library functions** (15 of them the function-pass starts of §12) and **28 functions in the crash reporter's span** `[0x00631620, 0x00633ab0)` (documented in `structs/crash_report.md`): 25 `crashreport_` functions plus `crt_operator_delete_thunk_00631fa0`, `string_range_copy_assign` and `string_ref_construct_at_cdecl`. The band's seven phantoms (§14) are not functions.

Band 5 itself splits into three VA sub-bands:

| Sub-band | Span | What |
|---|---|---|
| A | `[0x0062ec30, 0x0062f9bb]` | the **Pentium FDIV-erratum FP object**: MSVC's `_adj_*` / `__safe_*` replacements for the x87 divide and remainder, plus `_fpclass` |
| B | `[0x0062fa4d, 0x006315e9]` | `tzset.c`, `mbctype.c`, the `_LDBL12` decimal converters, `winsig.c`, `calloc`, `getenv` / `putenv`, `__crtCompareStringA` |
| F | `[0x00633ab0, 0x00634fe0)` | the MSVC 6 **classic (pre-standard) iostream** (`ios`, `istream`, `ifstream`, `filebuf`, `streambuf`), the `kernel32.dll` thunks of the crash reporter, then `stricmp`, `strnicmp`, `itoa`, `chdir`, `access` and the spawn group |

## 5. Why Ghidra names part of it

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

Only ~38 of 5322 definitions carry a FID name. FID coverage in this binary is tiny and
clustered almost entirely in this region.

## 6. The Dinkumware `<stdexcept>` classes — `[0x006243b7, 0x006248a8)`

### 6a. The binary names these classes itself

C++ exception handling needs type descriptors even when RTTI is switched off, so the mangled
class names sit in `.data` in the clear. **Three independent artifacts agree, and no row below
was assigned from a guess:**

| vtable | `_ThrowInfo` | catchables | type descriptor | class |
|---|---|---|---|---|
| `0x0063ef28` | `0x00644c38` | 3 | `.?AVlength_error@std@@` @ `0x00651b60` | `std::length_error` |
| `0x0063ef48` | `0x00644ca8` | 2 | `.?AVlogic_error@std@@` @ `0x00651b40` | `std::logic_error` |
| `0x0063ef5c` | `0x00644d60` | 3 | `.?AVout_of_range@std@@` @ `0x00651b80` | `std::out_of_range` |
| - | - | - | `.?AVexception@@` @ `0x00651b28` | `exception` (the base, §7) |

Each `_ThrowInfo` is `{attributes = 0, pmfnUnwind = <the class's plain dtor>, 0, catchables}`,
and the **catchable counts encode the hierarchy** — 2 for the base, 3 for each derived class.

### 6b. Object layout

```
+0x00  vptr
+0x0c  embedded basic_string<char>   { +0x00 allocator byte, +0x04 _Ptr, +0x08 _Len, +0x0c _Res }
```

so the string's `_Ptr` lands at `this+0x10`, which is exactly what `std_logic_error_what`
returns. `logic_error` is the base; `length_error` and `out_of_range` add nothing but a vtable.

### 6c. The vtable — 3 slots, identical shape on all three classes

| slot | `logic_error` | `length_error` | `out_of_range` |
|---|---|---|---|
| 0 | `std_logic_error_scalar_deleting_dtor` | `std_length_error_scalar_deleting_dtor` | `std_out_of_range_scalar_deleting_dtor` |
| 1 | `std_logic_error_what` | *(the same function)* | *(the same function)* |
| 2 | `std_logic_error_throw_copy` | `std_length_error_throw_copy` | `std_out_of_range_throw_copy` |

Slot 1 is **one function filling three slots**. That is ordinary vtable inheritance, not an
ICF fold: all 3 datarefs belong to one hierarchy and neither derived class overrides it.

### 6d. Two destructors per class

The PC emits **two** destructors per class — a plain one, reachable only through the
`_ThrowInfo`'s `pmfnUnwind` field, and a scalar-**deleting** one in vtable slot 0,
`std_logic_error_scalar_deleting_dtor(this, byte flag)`, which runs `if (flag & 1) operator
delete`. The cross-build pairing is in §15a.

### 6e. `_Xlen` and `_Xran` inline their own derived constructors

`std_string_xlen` and `std_string_xran` each call the `logic_error` **base** constructor and
then store the derived vtable pointer themselves. That bare store *is* the derived
constructor — MSVC inlined it, which is why `length_error(const string&)` and
`out_of_range(const string&)` have no function anywhere in the image. Marked inline in the
decomp at both sites.

## 7. MSVC's `exception` and `type_info` — `[0x00627059, 0x00627150)`

`.?AVexception@@` is listed in §6a as "the base" of the three `<stdexcept>` classes. Its own
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

`exception::~exception` (`0x006270fc`, a Ghidra FID name kept per §10) is the plain
destructor; it has 4 `E8` callers — the three `<stdexcept>` destructors and the deleting
form — plus 5 `E9` tail jumps. `exception_ctor_from_cstr` and `exception_copy_ctor` are the
base constructors the `<stdexcept>` classes chain to; the copy constructor's `_m_doFree`
branch (share the pointer when it is a literal, deep-copy it when it is owned) is the whole
reason that third field exists.

`type_info` is here for the same reason the type descriptors in §6a are: **C++ exception
handling needs them even with RTTI switched off.** Only its two destructors are out of line,
both reached exclusively through `type_info::vftable` (`.rdata:0x0063efe8`).

## 8. The import thunks — `[0x006248a8, 0x006248e4)`

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

## 9. The MSVC 6 C runtime — the inventory

### 9a. Library objects, band by band

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
  sets `g_crt_adjust_fdiv`. The `.data` run around the hook is MSVC's FP-init object: `_fltused`
  (`0x9875`) at `0x00651ba0`, `_ldused` (`0x9873`), `_FPinit` (`g_crt_fp_init_hook`), then the
  per-thread hooks `_FPmtinit` / `_FPmtterm` (`g_crt_fp_mt_init_hook` / `g_crt_fp_mt_term_hook`).
  Both per-thread hooks hold `crt_fpclear`, MSVC's empty `_fpclear` — the one-byte `ret` right
  after `crt_fp_init`'s own `ret` — and an image-wide scan finds no reader for either: only the
  multithreaded CRT calls them, and it is not linked. The shipped `LS3DF.dll` and `IJoy.dll`
  carry the same unread triple.
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
  see §9f for what separates them from the asin pair in band 2.
* **Conversions and formatting.** `crt_atof`, `crt_atol`, `crt_atoi`, `crt_chkstk`, and
  `crt_sprintf` (**261 callers**).

Band 2, `[0x00625021, 0x00627686]`:

| sub-band | group | starts |
|---|---|---|
| `[0x00625021, 0x00625071)` | `_vsnprintf` | 1 |
| `[0x00625071, 0x006251d1)` | C++ EH array helpers — `__ehvec_dtor` / `__ehvec_ctor`, their two unwind funclets, `__ArrayUnwind` | 5 |
| `[0x006251d1, 0x00625370)` | stdio — `fclose`, `fwrite`, `_fsopen`, `fopen` | 4 |
| `[0x00625370, 0x00625397)` | `strrchr` | 1 |
| `[0x00625397, 0x006253bf)` | `srand` / `rand` | 2 |
| `[0x006253bf, 0x0062558d)` | **`qsort`** + `shortsort` + `swap` | 3 |
| `[0x0062558d, 0x00625660)` | `_strupr`, `sscanf` | 2 |
| `[0x00625660, 0x00625700)` | `strncmp`, `isalpha`, `isspace` | 3 |
| `[0x00625700, 0x00625840)` | `strchr`, `strstr` | 2 |
| `[0x00625840, 0x00625b75)` | **`memmove`** (821 B) | 1 |
| `[0x00625b75, 0x00625cae)` | `fprintf`, `strncpy` | 2 |
| `[0x00625cae, 0x00625e4c)` | **`ceil` / `floor`** | 2 |
| `[0x00625e4c, 0x00625f60)` | `fread`, `remove` | 2 |
| `[0x00625f60, 0x00626180)` | **`pow`** — `_CIpow` entry, body, the is-integer helper | 3 |
| `[0x00626180, 0x006262ac)` | `strncat`, **`_purecall`** | 2 |
| `[0x006262ac, 0x00626443)` | `toupper` / `tolower` | 2 |
| `[0x00626443, 0x00626800)` | **heap** — `realloc`, `malloc`, `_nh_malloc`, `_heap_alloc`, `free` | 5 |
| `[0x00626800, 0x00626920)` | `__aulldiv`, `__allmul`, `vsprintf`, `__allshl` | 4 |
| `[0x00626920, 0x006269eb)` | **asin** — `_CIasin` entry + body | 2 |
| `[0x006269eb, 0x00626ac7)` | `time` | 1 |
| `[0x00626ac7, 0x00626cfd)` | `strtol` / `strtoxl` / `strtoul` | 3 |
| `[0x00626cfd, 0x00626dff)` | `crt0dat` — `_cinit`, `exit`, `_exit`, `doexit`, `_initterm` | 5 |
| `[0x00626dff, 0x00626f1a)` | `_getcwd` / `_getdcwd` / drive validation | 3 |
| `[0x00626f1a, 0x00627059)` | `WinMainCRTStartup`, `_amsg_exit`, `_fast_error_exit` | 3 |
| `[0x00627059, 0x00627150)` | **the `exception` class** (5) + **`type_info`** (2) — §7 | 7 |
| `[0x00627150, 0x00627230)` | `strlen`, `_SEH_prolog`, `_CxxThrowException` | 3 |
| `[0x00627230, 0x00627565)` | **`memcpy`** (821 B) | 1 |
| `[0x00627565, 0x006275de)` | FP init — `_setdefaultprecision`, the FDIV-erratum query and self-test | 3 |
| `[0x006275de, 0x006276a0)` | `cvt` — `_forcdecpt`, `_cropzeros`, `_positive`, and `_fassign` (`0x0062769e`, §9e) | 4 |

`crt_strtoul`'s single caller is `game_collect_scream_area`, which parses the hexadecimal mask in
an `enm_scrm_...(hex)` frame name with radix 16.

Band 3, `[0x006276dc, 0x0062ad14]` — 86 Ghidra starts (85 functions + 1 thunk; `__except_handler3` is the one without a def) in ONE
contiguous VA cluster, 22 distinguishable library object groups:

| sub-band | group | starts |
|---|---|---|
| `[0x006276dc, 0x00627a19)` | `cvt.c` **tail** — `_cftoe` / `_cftof` / `_cftog` / `_cfltcvt` / `_shift` (its head is in band 2) | 7 |
| `0x00627a19` | `msize.c` | 1 |
| `[0x00627a7a, 0x00628280)` | **`frame.cpp` — the whole MSVC C++ EH engine** (§9d) | 16 |
| `0x00628280` | `trnsctrl.cpp` — `_CallSettingFrame` | 1 |
| `[0x006282cc, 0x00628371)` | `hooks.cpp` — `terminate` / `unexpected` | 2 |
| `[0x00628371, 0x006286a8)` | the x87 FP error-entry and operand-class dispatch (hand-written asm), including `math_x87_dispatch_by_operand_class` / `_pair` (MSVC `_trandisp1` / `_trandisp2`) | 18 |
| `0x0062873c` | `_fltin` — string to double | 1 |
| `0x006287cd` | `isctype.c` | 1 |
| `0x00628842` | `_flsbuf.c` | 1 |
| `[0x00628957, 0x00629168)` | **`output.c` — `_output` and its six statics** (§9e) | 7 |
| `[0x0062916c, 0x00629244)` | `exsup*.asm` — `__except_handler3`, `__seh_longjmp_unwind_4` (`0x00629229`; the `@4` decoration becomes `_4`, as in `__CallSettingFrame_12`) | 2 |
| `0x00629244` | `close.c` | 1 |
| `0x006292f7` | `_freebuf.c` | 1 |
| `[0x00629322, 0x0062942f)` | `fflush.c` and `flsall` | 4 |
| `0x0062942f` | `write.c` | 1 |
| `0x006295dc` | `_openfil.c` | 1 |
| `0x0062974c` | `_getstream.c` | 1 |
| `[0x006297d0, 0x006298c0)` | `strcat.asm` — `strcpy` and `strcat` share one copy loop | 2 |
| `[0x006298c0, 0x00629ae4)` | `a_map.c` — `__crtLCMapStringA` and its `__except` filter; the handler blocks and the post-`__try` code belong to the host (§9n) | 2 |
| `[0x00629ae4, 0x0062a595)` | **`input.c` — the scanf engine and its four statics** (§9e) | 5 |
| `[0x0062a595, 0x0062a65f)` | `_sftbuf.c` — `_stbuf` / `_ftbuf` | 2 |
| `[0x0062a65f, 0x0062ad26)` | `handler.c` — the math error / `_matherr` / `_FPIEEE` machinery | 9 |

Band 4, `[0x0062ad26, 0x0062ec1e]` — 96 Ghidra starts (95 functions + 1 thunk) in ONE
contiguous VA cluster, ~41 library object groups:

| sub-band | group | starts |
|---|---|---|
| `[0x0062ad26, 0x0062ae6a)` | `frexp` and the double classifier | 3 |
| `[0x0062ae6a, 0x0062af00)` | the x87 status/control primitives — `_statfp`, `_clrfp`, `_ctrlfp`, `_set_statfp` | 4 |
| `0x0062af00` | `_filbuf` | 1 |
| `0x0062afd9` | `_read` | 1 |
| `0x0062b1cf` | `_dosmaperr` | 1 |
| `[0x0062b240, 0x0062b447)` | the **dead `exp` object**'s x87 dispatch arms, `math_x87_integer_parity` and `math_pow_special_case_x87_wrapper` — §9i | 5 + 1 thunk |
| `[0x0062b447, 0x0062b5dc)` | the `pow` special-case helper and its parity test | 2 |
| `[0x0062b5dc, 0x0062c123)` | the **V6** small-block heap (§9b) | 7 |
| `[0x0062c123, 0x0062c7f0)` | the **V5** small-block heap (§9b) | 8 |
| `[0x0062c7f0, 0x0062c9dd)` | `_callnewh`, the linker-version reader, `__heap_select`, `_heap_init` | 4 |
| `0x0062c9dd` | `__loctotime_t` | 1 |
| `[0x0062ca9f, 0x0062cc23)` | `winxfltr.c` — `_XcptFilter` and its table search | 2 |
| `[0x0062cc23, 0x0062cf81)` | startup — `_wincmdln`, `_setenvp`, `_setargv`, `parse_cmdline` (§9g) | 4 |
| `0x0062cf81` | `__crtGetEnvironmentStringsA` | 1 |
| `0x0062d0b3` | `_ioinit` | 1 |
| `[0x0062d25e, 0x0062d3f0)` | `crt0msg.c` — the banner and the numbered message writer | 2 |
| `0x0062d3f0` | `strcmp.asm` | 1 |
| `[0x0062d474, 0x0062d4d8)` | `unhandld.cpp` — the C++ unhandled filter, its installer and its restorer | 3 |
| `[0x0062d4d8, 0x0062d63e)` | `ieee87.c` — `_control87`, `_controlfp` and the two translators | 4 |
| `[0x0062d63e, 0x0062dbc0)` | the 12-byte long double: `_LDBL12` helpers, `_atodbl`/`_atoflt`, `_fptostr`, `_fltout`, `_dtold` | 15 |
| `0x0062dbc0` | `memset.asm` | 1 |
| `0x0062dc18` | `_fptrap` | 1 |
| `[0x0062dc21, 0x0062dc71)` | the three `IsBad*Ptr` validators | 3 |
| `0x0062dc71` | `abort` | 1 |
| `0x0062dc88` | the type-driven math error reporter | 1 |
| `0x0062dd58` | `__strgtold12` + its jump table | 1 |
| `[0x0062e229, 0x0062e372)` | `a_str.c` — `__crtGetStringTypeA` | 1 |
| `[0x0062e372, 0x0062e476)` | `_lseek`, `_getbuf`, `_isatty` | 3 |
| `[0x0062e476, 0x0062e52f)` | `_file.c` — `__initstdio` and `__endstdio` (`crt_initstdio`, `crt_endstdio`) | 2 |
| `0x0062e52f` | `wctomb` | 1 |
| `0x0062e5a0` | `llrem.asm` | 1 |
| `[0x0062e615, 0x0062e82f)` | `osfinfo.c` + `commit.c` | 5 |
| `0x0062e82f` | `_sopen` | 1 |
| `0x0062eae8` | `mbtowc` | 1 |
| `0x0062ebb0` | `ungetc` | 1 |
| `0x0062ec1e` | `_matherr` — `crt_matherr_default` (§9h) | 1 |

### 9b. BOTH small-block heaps ship, and the CONSUMERS assign every row

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

### 9c. `memcpy` and `memmove` are byte-identical, and the STRING CLASS separates them

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

### 9d. The whole C++ EH engine

`frame.cpp` — `__InternalCxxFrameHandler`, `FindHandler`, `FindHandlerForForeignException`,
`TypeMatch`, `__FrameUnwindToState`, `CatchIt`, `CallCatchBlock`, `BuildCatchObject`,
`AdjustPointer`, `_DestructExceptionObject`. Two constants read straight out of the bodies
pin the whole file: **`0x19930520`** (`EH_MAGIC_NUMBER1`) and **`0xe06d7363`**
(`EH_EXCEPTION_NUMBER`, `'msc' | 0xE0000000`).

The naming shape is DESCRIPTIVE rather than the MSVC internals, because that is what the
`crt_eh_*` leaf helpers below it are (`crt_eh_get_try_range_to_check`,
`crt_eh_call_catch_block`, `crt_eh_unwind_nested_frames`, `crt_eh_jump_to_continuation`).
The `cvt.c` group goes the other way — `crt_cftoe` / `crt_cftof` / `crt_cftog` — because
three of that file's five helpers are in the registry under their MSVC names
(`crt_cropzeros`, `crt_forcdecpt`, `crt_positive`). One family per shape, copied from the
registry rather than invented.

### 9e. `_output` and `_input`, and the table they close between them

`crt_output` (`0x00628957`) is the printf formatting engine behind **all 261**
`crt_sprintf` callers plus `crt_vsnprintf`, `crt_fprintf` and `crt_vsprintf` — a closed
caller set the registry names. `crt_input` (`0x00629ae4`) is the scanf engine.

Between them they close `g_crt_cfltcvt_tab`. `crt_output` calls slots **0, 1 and 3**
(`_cfltcvt`, `_cropzeros`, `_forcdecpt`) and `crt_input` calls slot **2** with the
signature `(flag, ptr, string)`, which is `_fassign` (`0x0062769e`). That slot is closed
from the other end too: `crt_atodbl` (`0x0062d9d0`) and `crt_atoflt` (`0x0062d9fd`) are
`__fassign`'s only callees, selected by its flag argument. `0x0062769e` sits 24 bytes past
the end of band 3 and belongs with the `cvt` group.

One game-side caller formats through the UNBOUNDED `crt_vsprintf`: `std_string_ctor_formatted`
writes into a 1000-byte stack buffer, where its sibling `string_format` uses
`crt_vsnprintf(buf, 0x400, …)`. Its only caller, `random_morph_mission_start`, formats
`missions\%s\rmorf.bin` — 19 characters plus the mission name, whose longest shipped value
(`MISE03-SALIERYKONEC`) is 19 characters. `check_seg_clusters_load` formats the same name into a
64-byte buffer.

### 9f. The math function-NAME table decodes, and it separates the x87 entry points

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

### 9g. The startup argv / environment group

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

### 9h. `crt_matherr_default` (`0x0062ec1e`) is the user-replaceable `_matherr`

Its body is `33 c0 c3`, but the name comes from the CALLERS, not from the bytes: both pass a
`struct _exception {int type; char *name; double arg1, arg2, retval;}` and branch on the
result. Returning 0 is the C library's documented "not handled" answer, not an absent body.

### 9i. The dead `exp` object

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

**The descriptor has ZERO references image-wide on all four channels**, and so does
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

### 9j. The FDIV-erratum object, closed from the flag the registry names

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

Both cores test the divisor the way the divide core does. They read its high mantissa word at
`[esp+0x16]`, run the plain instruction unless bits 56..58 are all set (`xor eax,0x700;
test eax,0x700`), and otherwise index `g_crt_fprem_flawed_divisor_nibbles` (`0x0065460c`)
with bits 59..62, the top fraction nibble (`shr eax,0xb; and eax,0xf`). That table is a
byte-identical second copy of `g_crt_fdiv_flawed_divisor_nibbles`: the same source array
emitted twice.

**The whole object is DEAD.** Its only external caller is `0x0062b27f`, inside
`math_exp_finite_argument_arm`, arm 0 of the `exp` operand-class descriptor at
`.data:0x006521c0`, which §9i measures as having zero references on all four channels. The game was not compiled with `/QIfdiv`. Every entry of
the object that is not called from inside it has 0 references on all four channels.

### 9k. `_purecall` — a 9-byte function with 36 vtable slots and no callers

`crt_purecall` (`0x006262a3`) is `push 0x19; call __amsg_exit`. **0x19 = 25 = `_RT_PUREVIRT`**
in MSVC's `rterr.h`, and the same table is confirmed 0x2000 bytes away by `crt_startup`
passing **0x1c = 28 = `_RT_HEAPINIT`** to `crt_fast_error_exit` — two constants from one
enum, each landing exactly where that enum says it should.

It has **zero `E8` callers and thirty-six `.rdata` references**. Those 36 are 36 abstract
classes' vtable slots, which makes this address a **census tool for abstract classes in
`Game.exe`** — a cheap, closed-set way to enumerate them that no name search can match.

### 9l. `srand(time(NULL))`, located

`game_init` runs `crt_time(0)` and feeds the result straight to `crt_srand`. `crt_rand`'s seed
`g_crt_rand_seed` ships as **1** — the C standard's initial value, i.e. the value an
unseeded `rand` would have used — so that one pair is what makes every `rand_range` /
`rand_float` draw in the game differ between runs.

### 9m. Two byte-identical functions that were NOT folded

`crt_eh_call_member_function_00624a4b` and `crt_eh_call_member_function_00624a52` are the
same 7 bytes — `58 59 87 04 24 ff e0` — and both survive. The bytes contain no relative
branch (`ff e0` is `jmp eax`), so the duplication is real: a clean instance of the
non-COMDAT population that survives `/OPT:ICF`.

### 9n. `crt_lcmapstring_ansi_except_filter` (`0x00629a84`) is a FILTER, not a funclet

Its single dataref is `.rdata:0x0063f19c`, an `__except_handler3` SCOPE TABLE
`{prev, filter, handler}`, byte-decoded as `{-1, 0x00629a84, 0x00629a88}` — this address is
the FILTER. Its owner is `crt_lcmapstring_ansi`, which pushes the table's base in its own
prologue. The two HANDLER funclets of the same owner, `0x006299d4` and `0x00629a88`, are
paired with it in the scope table at `.rdata:0x0063f190` (§13).

## 10. Naming policy for this region

**Names that Ghidra assigned in its default analysis are kept** and given docblocks and
registry rows. That covers `operator_new`, `__ftol`,
`__global_unwind2`, `__local_unwind2`, `__abnormal_termination`, `__NLG_Notify1`, `_strcmp`,
`_memset`, `exception::~exception` and all ten import thunks.

The policy does not reach `crt_atexit`, `crt_onexit` and `crt_rand`: Ghidra's default
analysis has **zero** hits for the MSVC spellings `atexit` and `_onexit`, and it shows `rand`
only as `FUN_006253a1`, so none of the three names comes from Ghidra, and all three follow the
`crt_` convention of the rest of the region.

### 10a. Why the `ls3df.dll` thunks keep `S_vector::RotateByMatrix`

Those 29 names are verbatim demanglings of strings physically present in `Game.exe`'s import
directory — e.g. `?RotateByMatrix@S_vector@@QBG?AU1@ABUS_matrix@@@Z`. Four reasons not to
rename them:

1. **A rename is impossible to earn.** The bodies are in `ls3df.dll`, so a behaviour-derived
   name could never be body-diffed, and any replacement would be a transliteration of the
   export name.
2. It would **collide semantically with the 30-member in-exe `vec3_*` family**, which is
   `Game.exe`'s *own* inlined LS3D math.
3. The `::` spelling is a working convention: exactly 30 of 6759 definitions carry
   `::`, and 29 of them are this population. The punctuation *is* the marker for "6-byte
   import thunk, body in a DLL".
4. `basic/artifacts/dll.md` §4 is built on those names, and the mangling additionally records
   the calling convention (`QAG`/`QBG` = public `__stdcall`).

## 11. Data symbols

The writer and reader cells are the named uses in the bodies; a row-only symbol, or a global reached through a register, shows `-` there, and its Role cell says what is measured.

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_std_string_nil` | `0x0063eaf8` | Dinkumware `basic_string<char>::_Nil` — a lone NUL whose **preceding share-count byte is pinned at 0**, which is why `std_string_freeze` never unshares or frees it. Not `g_empty_string` (`0x00655214`), the game's own null-name fallback | - | `random_morph_mission_start`, `std_logic_error_what`, `std_string_assign_substring` |
| `g_crt_ehvec_dtor_scope_table` | `0x0063ef80` | `{-1, 0, 0x006250d9}` | - | `crt_ehvec_dtor` |
| `g_crt_eh_array_unwind_scope_table` | `0x0063ef90` | `{-1, 0x0062512d, 0x00625137}` — both inside its own body | - | `crt_eh_array_unwind` |
| `g_crt_ehvec_ctor_scope_table` | `0x0063efa0` | `{-1, 0, 0x006251b9}` | - | `crt_ehvec_ctor` |
| `g_crt_startup_scope_table` | `0x0063efb8` | `{-1, 0x00626ff1, 0x00627005}` | - | `crt_startup` |
| `g_exception_vt` | `0x0063efc8` | the 2-slot vtable of §7 | - | - |
| `g_crt_cxx_exception_record` | `0x0063eff0` | the `EXCEPTION_RECORD` template: `0xe06d7363`, NONCONTINUABLE, 3 params, magic `0x19930520` | - | `__CxxThrowException_8` |
| `crt_atexit_noop_dtor_006248a7` | `0x006248a7` | a one-byte `ret`; see §11b | - | - |
| `g_crt_xc_a` / `_z` | `0x00646000` / `0x0064630c` | MSVC `__xc_a` / `__xc_z`, the `.CRT$XC` bounds (C++ initializers); the second `_initterm` in `crt_call_init_fns` | - | `crt_call_init_fns` |
| `g_crt_xi_a` / `_z` | `0x00646310` / `0x00646324` | MSVC `__xi_a` / `__xi_z`, the `.CRT$XI` bounds (C initializers); the first `_initterm` in `crt_call_init_fns` | - | `crt_call_init_fns` |
| `g_crt_xp_a` / `_z` | `0x00646328` / `0x00646330` | MSVC `__xp_a` / `__xp_z`, the `.CRT$XP` bounds (pre-terminators); the first `_initterm` in `crt_doexit` | - | `crt_doexit` |
| `g_crt_xt_a` / `_z` | `0x00646334` / `0x0064633c` | MSVC `__xt_a` / `__xt_z`, the `.CRT$XT` bounds (terminators); the second `_initterm` in `crt_doexit` | - | `crt_doexit` |
| `g_crt_fp_init_hook` | `0x00651ba8` | holds `crt_fp_init`; read by `_cinit`. Row-only: no Ghidra symbol | - | `crt_call_init_fns` |
| `g_crt_fp_mt_init_hook` | `0x00651bac` | MSVC `_FPmtinit`, holding `crt_fpclear`; no reader (§9a). Row-only: no Ghidra symbol | - | - |
| `g_crt_fp_mt_term_hook` | `0x00651bb0` | MSVC `_FPmtterm`, holding `crt_fpclear`; no reader (§9a). Row-only: no Ghidra symbol | - | - |
| `g_crt_nlg_destination` | `0x00651bb4` | MSVC `_NLG_Destination`; first dword is the signature **`0x19930520`**. Row-only | - | - |
| `g_crt_rand_seed` | `0x00651be0` | the LCG state; written by `crt_srand`, advanced by `crt_rand`. Ships as **1** | `crt_rand`, `crt_srand` | `crt_rand` |
| `g_crt_x87_cw_round_up` | `0x00651bf0` | `0x1b3f`, bits 11:10 = 10b = toward +∞. Read only by `crt_ceil` | - | `crt_ceil` |
| `g_crt_x87_cw_round_down` | `0x00651bf4` | `0x173f`, bits 11:10 = 01b = toward −∞. Read only by `crt_floor` | - | `crt_floor` |
| `g_crt_aexit_rtn` | `0x00651c18` | MSVC `_aexit_rtn`; holds `__exit` (`0x00626d3b`). Its only reference is `__amsg_exit`'s `call [0x651c18]` with 0xff | - | `__amsg_exit` |
| `g_crt_cfltcvt_tab` | `0x00651c50` | the 6-entry float→text conversion vector; Ghidra split it into six symbols, all named with index suffixes | - | `crt_cfltcvt_init`, `crt_input`, `crt_output` |
| `g_crt_math_indefinite_x87` | `0x00651c80` | the value pre-loaded on every math domain error | - | `math_acos_continuation`, `math_asin_continuation` |
| `g_crt_half_pi_x87` | `0x00651c8a` | 80-bit π/2 — the constant that separates the asin pair (§9f) | - | `math_asin_continuation` |
| `g_crt_pctype` | `0x00651cb4` | MSVC `_pctype`, 2 bytes per character. 37 references | - | 14 functions |
| `g_crt_mb_cur_max` | `0x00651ec0` | the single-byte / MBCS branch selector at every ctype site. Value 1 | - | 13 functions |
| `g_crt_decimal_point` | `0x00651ec4` | `'.'`; written by `crt_forcdecpt`, compared by `crt_cropzeros` | - | `crt_cftoe`, `crt_cropzeros`, `crt_input`, `crt_string_to_ld12` |
| `g_crt_math_function_names` | `0x00651ef0` | the 27-pair math name table decoded in §9f | - | `math_function_name_for_id` |
| `g_crt_xcpt_action_table` | `0x00654208` | MSVC `_XcptActTab`, typed `S_crt_xcpt_action[10]`: ten `{exception code, signal, action}` records, bounded by `g_crt_xcpt_action_count` (= 10); searched by `crt_xcpt_action_lookup` / `_by_signal` | - | `crt_exception_filter`, `crt_xcpt_action_lookup`, `crt_xcpt_action_lookup_by_signal` |
| `g_crt_iob_stdin` | `0x00654360` | `_iob[0]`, typed as one `FILE` (0x20); `crt_initstdio` resets the `_file` of it and the next two streams | - | `crt_initstdio` |
| `g_crt_fdiv_flawed_divisor_nibbles` | `0x006545f0` | 16 bytes, non-zero only at 1, 4, 7, 0xa, 0xd: the five Pentium-FDIV-flaw divisor patterns (§9j) | - | `math_x87_fdiv_erratum_divide_core` |
| `g_crt_fprem_flawed_divisor_nibbles` | `0x0065460c` | the byte-identical copy the two `fprem` cores index with the divisor's top fraction nibble (§9j) | - | `math_x87_fdiv_erratum_fprem1_core`, `math_x87_fdiv_erratum_fprem_core` |
| `g_crt_fdiv_register_form_dispatch_table` | `0x0065464e` | the 64 targets of `math_x87_fdiv_erratum_register_divide_dispatch`, indexed `(st(i) << 3) \| form` (§9j). Row-only: the address is unaligned and only ever a `jmp` operand, so it has no Ghidra symbol | - | - |
| `g_crt_days_before_month` | `0x00654834` | the base-minus-one-element pointer the code adds `month*4` to with a **1-based** month; see §11c | - | `crt_dst_rule_to_yearday`, `crt_local_time_to_time_t` |
| `g_crt_pow10_positive` | `0x00654968` | MSVC `_pow10pos`, 29 twelve-byte powers of ten (§11d). Row-only: loaded only as an `imm32` | - | - |
| `g_crt_pow10_negative` | `0x00654ac8` | MSVC `_pow10neg`, the 29 matching negative powers (§11d). Row-only: loaded only as an `imm32` | - | - |
| `g_crt_math_skip_error_handler` | `0x006f9750` | see §11a | - | `math_acos_continuation`, `math_asin_continuation`, `math_pow_continuation` |
| `g_crt_adjust_fdiv` | `0x006f9754` | MSVC `_adjust_fdiv`, the **Pentium FDIV-erratum flag** (§9j) | `crt_fp_init` | `math_exp_finite_argument_arm` |
| `g_crt_time_dstflag_cache` | `0x006f9758` | `crt_time`'s cached daylight-saving flag (-1 / 1 / 0), reused while the cached minute matches | `crt_time` | `crt_time` |
| `g_crt_time_gmt_cache` | `0x006f9760` | `crt_time`'s cached `SYSTEMTIME` (16 bytes), compared minute, hour, day, month, year against the current UTC time | - | `crt_time` |
| `g_crt_errno` | `0x006f9770` | `ERANGE` from `crt_strtoxl`, `EACCES`/`ENOMEM`/`ERANGE` from `crt_getdcwd` | 21 functions | - |
| `g_crt_doserrno` | `0x006f9774` | `ERROR_INVALID_DRIVE`, written beside `EACCES` | 15 functions | `crt_chsize`, `crt_commit`, `crt_sopen` |
| `g_crt_osver` | `0x006f977c` | MSVC `_osver` = `GetVersion() >> 16`, set by `crt_startup` | `crt_startup` | - |
| `g_crt_winver` | `0x006f9780` | MSVC `_winver` = `g_crt_winmajor * 0x100 + g_crt_winminor` | `crt_startup` | - |
| `g_crt_winmajor` | `0x006f9784` | MSVC `_winmajor` = `GetVersion() & 0xff` | `crt_startup` | `crt_startup` |
| `g_crt_winminor` | `0x006f9788` | MSVC `_winminor` = `GetVersion() >> 8 & 0xff` | `crt_startup` | `crt_startup` |
| `g_crt_environ` | `0x006f9798` | MSVC `_environ` — the argv-style array (§9g) | `crt_build_environ`, `crt_setenv` | `crt_environ_find_index`, `crt_getenv`, `crt_setenv` |
| `g_crt_exitflag` | `0x006f97b0` | MSVC `_exitflag`: written by `crt_doexit` from its `retcaller` argument; `crt_endstdio` closes all streams only when it is set | `crt_doexit` | `crt_endstdio` |
| `g_crt_c_termination_done` | `0x006f97b4` | MSVC `_C_Termination_Done`: set by `crt_doexit` before the exit-handler passes; no reader | `crt_doexit` | - |
| `g_crt_c_exit_done` | `0x006f97b8` | MSVC `_C_Exit_Done`: `crt_doexit` calls `TerminateProcess` when it is set and sets it before `ExitProcess` | `crt_doexit` | `crt_doexit` |
| `g_crt_environment_block` | `0x006f97bc` | `_aenvptr` — the raw block (§9g) | `crt_build_environ`, `crt_cenvarg`, `crt_startup` | `crt_build_environ`, `crt_cenvarg` |
| `g_crt_se_translator` | `0x006f97e0` | the `_set_se_translator` callback | - | `crt_eh_call_se_translator`, `crt_eh_find_handler_for_foreign_exception` |
| `g_crt_lc_codepage` | `0x006f981c` | the default code page `crt_lcmapstring_ansi` substitutes when the caller passes 0 | - | 7 functions |
| `g_crt_new_mode` | `0x006f9834` | MSVC `_newmode`; gates the new-handler retry loop | - | `_malloc`, `crt_calloc`, `crt_realloc` |
| `g_crt_buffered_stream_count` | `0x006f9984` | MSVC `_cflush`, incremented by `crt_openfile`, `crt_setup_temp_buffer` and `crt_getbuf`, read nowhere | `crt_getbuf`, `crt_openfile`, `crt_setup_temp_buffer` | `crt_getbuf`, `crt_openfile`, `crt_setup_temp_buffer` |
| `g_crt_commode` | `0x006f9988` | MSVC `_commode`, OR'd into every new stream's flags by `crt_openfile`, its only reference. In `.bss` with no writer, so always 0 | - | `crt_openfile` |
| `g_crt_osfhandle_blocks` | `0x006fad40` | MSVC `__pioinfo`, typed `pointer[64]`: block `fd >> 5`, record `(fd & 0x1f) * 8`; `crt_init_file_handles` clamps inherited handles at 0x800 = 64 blocks | - | 19 functions |
| `g_crt_heap_handle` | `0x006fae44` | argument 1 of every `Heap*` call in the runtime | `crt_heap_init` | 11 functions |
| `g_crt_heap_mode` | `0x006fae48` | the small-block-heap generation selector (2 / 3 / other) | `crt_heap_init` | `crt_calloc`, `crt_free`, `crt_heap_alloc`, `crt_heap_init`, `crt_msize`, `crt_realloc` |
| `g_crt_sbh_threshold` | `0x006fae64` | upper size bound of the mode-3 small-block path | `crt_sbh_heap_init` | `crt_calloc`, `crt_heap_alloc`, `crt_realloc` |
| `g_crt_command_line` | `0x006fae68` | the full command line `GetCommandLineA` returns (§9g) | `crt_startup` | `crt_get_cmdline_args`, `crt_setup_argv` |
| `g_crt_onexit_end` / `_begin` | `0x006fae74` / `0x006fae78` | the exit-handler table bounds | `crt_onexit`, `crt_onexit_init` | `crt_doexit`, `crt_onexit` |

Further named in this region: `g_crt_app_type`, `g_crt_error_mode` (MSVC `__error_mode`: 0
default, 1 stderr, 2 message box), `g_crt_old_sbh_threshold` (the mode-2 heap threshold at
`0x006541fc`), `g_crt_prev_unhandled_exception_filter`, `g_crt_terminate_handler`,
`g_crt_terminate_scope_table` and `g_crt_mbcs_case_ranges` (§11d).

### 11a. `g_crt_math_skip_error_handler` is never written

A little-endian scan of the whole image finds **six references and every one is
`83 3d 50 97 6f 00` = `cmp dword [0x006f9750], imm8`** — reads only; through none of the four
reference channels is there a writer. The scan covers displacements, immediates and data pointers
alike, and `Game.exe` has no export directory, so nothing outside the image can reach the flag
either. As shipped the flag is permanently 0, so the asin /
acos / pow error path **always** runs and the raw-result shortcut is dead. The name is a
deliberate NARROWING of "fast mode", which would assert an intent nothing in the binary
backs; "skip error handler" is exactly what the six branches do.

### 11b. Not every `atexit` registrar passes the same no-op destructor

The 109 per-translation-unit `atexit` registrars pass `crt_atexit_noop_dtor_0060b700` — **all
but two**. `crt_atexit_register_0062489b` passes its own local one-byte `ret` at
`0x006248a7`, and `screen_fade_static_dtor` is the other exception.

### 11c. `g_crt_days_before_month` sits one element BEFORE the array it names

`0x00654834` holds **365** — the last entry of the LEAP table — and is the
base-minus-one-element pointer the code adds `month*4` to with a **1-based** month. The
arrays themselves begin at `0x00654804` (leap) and `0x00654838` (common), and both have
their own rows. The symbol Ghidra offers sits on element `[-1]` because that is where the
reference arithmetic starts; `0x00654834` genuinely IS the base pointer the code uses.

### 11d. Five rows assigned by decoding data rather than by reading code

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
  `{lo, hi, delta}` word triples, i.e. MSVC's `__mbulinfo[6]`. It is typed `ushort[6]`. The
  image addresses it only as a whole: three `mov edi, 0x6f9af0` sites, a 3-dword copy in
  `crt_setmbcp` and 3-dword zeroes in `crt_setmbcp` and `crt_mbcs_set_single_byte`. No
  instruction reads a single word, so the decomp renders the copy in dword pieces.
* **`g_crt_pow10_positive` / `g_crt_pow10_negative`** (`0x00654968` / `0x00654ac8`), MSVC
  `_pow10pos` / `_pow10neg`, decode to 29 twelve-byte extended-precision powers of ten each:
  10^±1..10^±7, then 10^±8..10^±56 in steps of 8, 10^±64..10^±448 in steps of 64,
  10^±512..10^±3584 in steps of 512, then 10^±4096. Every entry equals the exact power to
  within 1.1e-19. `crt_ld12_multiply_by_power_of_ten` loads the table base (`mov ebx,
  0x654968` / `0x654ac8`), subtracts 0x60 and adds 0x54 (seven entries) per three-bit group
  of the exponent, so the decomp shows only the biased constants `0x654908` / `0x654a68`. An
  entry whose first, extra-precision word is `>= 0x8000` was rounded up; the function copies
  it and decrements the copy's mantissa before multiplying.

### 11e. One shared guard byte for 109 static initializers

`g_crt_static_init_flags` (`0x006f9ae0`) has 218 references: 109 guards, each a read and a write
of bit 0. Every guard is the same code — test bit 0, set it, then, outside the test, go to its
translation unit's `atexit` registrar — with no constructor code in between, and the registrars
pass the no-op destructor of §11b. That is one header-defined (COMDAT) static whose constructor
and destructor bodies are empty: every translation unit that includes the header emits its own
guarded initializer, and the shared bit lets only the first one run. Two adjacent `.CRT$XCU`
slots guarding the same bit, as `crt_static_init_guard_006242b0` and
`crt_static_init_guard_00624865` do, are two consecutive translation units. The object itself
is never accessed, so it cannot be named. `crt_static_init_guard_00624865` also guards bit 0 of
`g_crt_static_init_flags_1` (`0x006f9ae1`), a second object of the same kind.

## 12. The function starts Ghidra did not create

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
| `0x0062b3e1` | `math_x87_integer_parity` | 0 = not an integer, 1 = odd, 2 = even; the x87 sibling of `math_double_integer_parity` | **2 `E8` callers**, both inside arm 0 of the dead `exp` object (§9i) |
| `0x0062b415` | `math_pow_special_case_x87_wrapper` | the x87 calling wrapper around `math_pow_special_case_result` | 0 references on all four channels |
| `0x0062d4cb` | `crt_cxx_restore_unhandled_exception_filter` | 13 bytes: `SetUnhandledExceptionFilter(g_crt_prev_unhandled_exception_filter)` — the un-install half of `crt_cxx_install_unhandled_exception_filter` | the `.CRT$XT` terminator slot `0x00646338` |
| `0x0062e51b` | `crt_endstdio` | MSVC `__endstdio`: `crt_flushall()`, then, behind `g_crt_exitflag`, a tail jump to `crt_fcloseall` ¹ | the `.CRT$XP` pre-terminator slot `0x0064632c` |
| `0x0062ed47` | `math_x87_fdiv_erratum_register_divide_dispatch` | MSVC `_adj_fdiv_r`, the 64-arm register-form dispatcher (§9j) | 0 references on all four channels; the byte before it is a `ret` |
| `0x0062f1f9` | `math_x87_fdiv_erratum_reverse_divide` | the internal reversed divide, the mirror of `math_x87_fdiv_erratum_divide` | **8 `E8` callers**, two in each `fdivr` memory-operand stand-in |
| `0x0062f2a4` | `math_x87_fdiv_erratum_divide_by_int16` | safe `fdiv` by a 16-bit integer memory operand (`fild word`, `ret 4`) | 0 references on all four channels |
| `0x0062f2d8` | `math_x87_fdiv_erratum_divide_by_int32` | safe `fdiv` by a 32-bit integer memory operand (`fild dword`, `ret 4`) | 0 references on all four channels |
| `0x0062f30c` | `math_x87_fdiv_erratum_reverse_divide_by_float` | safe `fdivr` by a float memory operand (`ret 4`) | 0 references on all four channels |
| `0x0062f358` | `math_x87_fdiv_erratum_reverse_divide_by_double` | safe `fdivr` by a double memory operand (`ret 8`) | 0 references on all four channels |
| `0x0062f3a4` | `math_x87_fdiv_erratum_reverse_divide_by_int16` | safe `fdivr` by a 16-bit integer memory operand | 0 references on all four channels |
| `0x0062f3d8` | `math_x87_fdiv_erratum_reverse_divide_by_int32` | safe `fdivr` by a 32-bit integer memory operand | 0 references on all four channels |
| `0x0062f40c` | `math_x87_fdiv_erratum_fdivp` | MSVC `_adj_fdivp_sti_st`: the EAX-preserving stand-in for `fdivp st(1),st` | 0 references on all four channels |
| `0x0062f63c` | `math_x87_fdiv_erratum_fprem_scale_denormal` | the `fprem` body that routes by the divisor's exponent (§9j) | 1 `E8` caller, `math_x87_fdiv_erratum_fprem` |
| `0x0062f8f4` | `math_x87_fdiv_erratum_fprem1_scale_denormal` | the `fprem1` body | 1 `E8` caller, `math_x87_fdiv_erratum_fprem1` |
| `0x0062f9a9` | `math_x87_fdiv_erratum_fprem` | `_adj_fprem` — `call 0x62f63c; ret` | 0 references on all four channels |
| `0x0062f9af` | `math_x87_fdiv_erratum_fprem1` | `_adj_fprem1` — `call 0x62f8f4; ret` | 0 references on all four channels |
| `0x0062f9b5` | `math_x87_fdiv_erratum_fpatan` | `_adj_fpatan` — `fpatan; ret`, a pass-through | 0 references on all four channels |
| `0x0062f9b8` | `math_x87_fdiv_erratum_fptan` | `_adj_fptan` — `fptan; ret`, a pass-through | 0 references on all four channels |

¹ `crt_flushall`'s caller is `crt_endstdio`, **not** `crt_initstdio`
(`0x0062e476`). The two are separated by a fact neither body shows: one address sits in the
`.CRT$XI` initializer table and the other in `.CRT$XP`. A VA-to-containing-function lookup
attributes the call to the previous start, which is not the caller here.

## 13. Identified, not named

A grade implies a name, so these are listed rather than graded.

| VA | what it is | why no name |
|---|---|---|
| `0x006299d4`, `0x00629a88` | the two `__except` HANDLER funclets of `crt_lcmapstring_ansi`, paired with the filter at `0x00629a84` in the scope table at `.rdata:0x0063f190` (§9n); `0x00629a88` lies inside the host's Ghidra function body, `0x006299d4` in no function | code of their owner, reached only through the scope table — not functions, so no def and no row |
| `0x00627e39`, `0x00627fa2`, `0x00627fb0` | `__except` continuations that run on their CALLER's frame | no def anywhere, and they are not entry points of their own |
| `0x006521b4` | the counted string `"exp"` | no Ghidra symbol and zero references on all four channels |
| `0x006521c0` | the `exp` operand-class dispatch descriptor (§9i) | same |
| `0x006f9a80` | the `ios` state word of `g_crashreport_map_file`, reached as `(&this)[*(int *)(obj + 4)]` through the virtual-base table | a FIELD of an object that has a row, not a symbol of its own |
| `0x00654644` | the 80-bit constant whose last two bytes the operand-class jump table overlaps (`00 00 00 00 00 00 ff ff fe 7f`, the largest finite long double) | read only by the `fprem` cores through a `sub eax, 0x654644` index computation, so it has no Ghidra symbol and no consumer that names it |

## 14. Phantoms — fall-through points that are not function starts

Nine addresses in this region are points inside other code that only a fall-through reaches.
A def placed at any of them (a phantom start, as Ghidra's Aggressive Instruction Finder
proposes) renders a body that duplicates its neighbour's, so none of them is a function start.

| phantom | real host start | what the rendered body dropped |
|---|---|---|
| `0x006297de` | two bytes of `mov edi, edi` alignment filler between `crt_strcpy` and `crt_strcat`; it falls through, so Ghidra rendered `crt_strcat`'s whole body a second time | a 75-line phantom record |
| `0x0062f203` | ten bytes INSIDE `math_x87_fdiv_erratum_reverse_divide` (`0x0062f1f9`) | **the finding is its host**: 8 `E8` callers and no Ghidra start of its own (§12) |
| `0x0062ede1` | `0x0062edda` (a jump-table arm of `math_x87_fdiv_erratum_register_divide_dispatch`) | both `fstp xword` argument spills |
| `0x0062ee05` | `0x0062edfe` (another arm) | both spills |
| `0x0062f417` | `math_x87_fdiv_erratum_fdivp` (`0x0062f40c`) | the `push eax` frame and both spills |
| `0x0062f654` | `math_x87_fdiv_erratum_fprem_scale_denormal` (`0x0062f63c`) | the frame, the spills and the exponent gate |
| `0x0062f6dd` | `0x0062f63c` | ditto (the far arm) |
| `0x0062f90f` | `math_x87_fdiv_erratum_fprem1_scale_denormal` (`0x0062f8f4`) | ditto |
| `0x0062f998` | `0x0062f8f4` | ditto (the far arm) |

All nine: **0 `E8`, 0 `E9`, 0 datarefs, 0 immediate-operand references** — every channel
measured in `Game.exe`. None of them is live: the entire FDIV-erratum object is unreachable
in the shipped binary (§9j).

**Four genuine functions here are entered only by tail jump.**
`math_x87_restore_control_word` (`0x0062864e`), `__math_exit` (`0x0062865b`),
`math_x87_check_result_range` (`0x00628699`) and `crt_abort` (`0x0062dc71`) are reached by
**tail jump** (`E9`), never by `call` — from the acos, asin and pow continuations, and from
`crt_terminate` at `0x0062831d`. They are real function starts, not fall-through points.

## 15. PC vs PS2

The PS2 ships a **different Standard Library** (Metrowerks/SN) and a **different C runtime**
(newlib), so most of this region has no counterpart by construction rather than by omission.

| finding | evidence |
|---|---|
| `std::out_of_range` **does not exist** in the PS2 build | VA-band enumeration of the PS2 symbol table with the class column **ignored**, not a class-name search |
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

### 15a. The six STRUCTURAL twins

| PC | PS2 | why capped at STRUCTURAL |
|---|---|---|
| `std_string_assign_buffer` | `assign__Q23std59basic_string<>FPCcUi` | same `(const char *, unsigned)` signature and the same three statements in the same order; the PC sets `_Len` and the NUL itself where the PS2's `change_size` does it internally |
| `std_logic_error_what` | `what__Q23std11logic_errorCFv` | same method, different layout — the PC returns `[this+0x10]` with a nil fallback, the PS2 `[this+4]` with none |
| `std_logic_error_scalar_deleting_dtor` | `__dt__Q23std11logic_errorFv` | paired on the flagged-delete signature (below) |
| `std_length_error_scalar_deleting_dtor` | `__dt__Q23std12length_errorFv` | same |
| `exception_what` | `what__Q23std9exceptionCFv` | same method; the PC returns `[this+4]` with a literal fallback, the PS2 a constant |
| `exception_scalar_deleting_dtor` | `__dt__Q23std9exceptionFv` | the flagged-delete signature again |

GCC folds the plain and the deleting destructor of §6d into a single flagged destructor, so the
PS2 twin belongs on the **deleting** one:

```
PC   std_logic_error_scalar_deleting_dtor(this, byte flag)   if (flag & 1) operator delete
PS2  __dt__Q23std11logic_errorFv(long, short)                if (0 < flag) __dl__FPv
```

The pairing rests on the **signature** — a mangled symbol spells its whole signature.

The cap is the precedent `std_string_dtor` set: two different Standard Library
implementations make a statement-level diff meaningless even when both bodies are read.

## 16. Related

`basic/artifacts/dll.md` · `structs/app_bootstrap.md` (the other region-shaped doc) · `structs/crash_report.md` (the game unit inside band 5) · `structs/safedisc_stub.md` (the code after this region) ·
`structs/string.md` (the game's own refcounted string class, a different one) · `structs/i3d_cache.md` (the three `crt_getcwd` callers) · `structs/check_seg.md` §3 and `structs/chunk.md` §6 (the
`folded_vector_ctor_at_0x04_0044a120` fold) · `Game-ghidra.exe.functions.md` (the registry rows for everything here).

## 17. Open items

| Item | State |
|---|---|
| 1. `FILE`, the two small-block-heap headers, `_ThrowInfo`, `S_crt` code-page records | their bytes are used by the bodies of §9 and are not typed here |
| 2. `undecorated_name` of `type_info` | named from the MSVC class; the image has no writer, so the field's content is not measured |
