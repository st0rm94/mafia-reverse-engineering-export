# The SafeDisc 2.70.030 protection stub — `stxt774` + `stxt371`

**Created** 2026-09-07 16:52 +0200 · **Modified** 2026-11-17 14:40 +0100

`Game.exe` carries two extra PE sections that are not part of the game: `stxt774`, 8 KB of encrypted payload, and `stxt371`, 13 KB of x86 code, which is
Macrovision **SafeDisc 2.70.030**'s loader. **In this binary it never runs**: the crack moved `AddressOfEntryPoint` back to `crt_startup`, so the whole
section is orphaned. 50 functions, all named.

> **This document is about what the protection IS.** What it did to the game (the encryption, the crack, the four patched bytes, the banner) is
> `basic/artifacts/crack.md`, the authority on every modification. Read it first.

**It is the only body of code in this project whose bytes the retail executable can authenticate.** `crack.md` §3a records that retail `.text` and `.data` are
encrypted. `stxt*` is plaintext in both files: measured, `stxt774` differs from `Game-uncracked.exe` in **0 bytes** and `stxt371` in exactly **36**, the six known
DLL-name case edits (`crack.md` §5e).

| | |
|---|---|
| PC prefix | `safedisc_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x006ff000, 0x0070225c)`, the section `stxt371`; the first function is at `0x006ff05e` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

| Section | VA | Vsize | Characteristics | Contents |
|---|---|---|---|---|
| `stxt774` | `0x006fc000` | `0x204f` | `0xE0000020` = code, read, write, execute | **encrypted payload data, not code**: high entropy, starting `e0 04 34 92 b4 a4 b7 ab` |
| `stxt371` | `0x006ff000` | `0x325c` | `0xE0000020` | the loader: 50 functions, a 53-entry obfuscated string table and the block of §1a |

**`stxt774` contains no functions.** Ghidra creates one at `0x006fc000` whose body decompiles to `swi(4)`: data decoded as code. It has no def in the decomp.
**VA to file is NOT the identity mapping for these two sections**, unlike `.text`: `stxt774` VA `0x006fc000` is at file `0x258000`, a delta of `0x4A4000`. Parse the
section table.

No record holds an array except the two names and the bootstrap table: `S_safedisc_directory_record.name` is the room left in the walker's `0x121`-byte read
buffer after the `0x1c`-byte head. The API table has one slot per resolved function, which is why its count is the 36 of §1c.

### 1a. `S_safedisc_stub_block` — at `0x006ff009`, `0x35` bytes

`safedisc_stub_entry` passes its address to `safedisc_run`. The file bytes are identical in `Game.exe` and `Game-uncracked.exe`; the values are in §2a.

```c
typedef struct S_safedisc_stub_block {
    /* +0x00 */ uint32_t import_rva;         // [H] the game's real `DataDirectory[1]` RVA
    /* +0x04 */ uint32_t import_size;        // [H] its size
    /* +0x08 */ uint32_t iat_rva;            // [H] the game's real `DataDirectory[12]` RVA
    /* +0x0c */ uint32_t iat_size;           // [H] its size
    /* +0x10 */ uint32_t bootstrap_rva;      // [H] RVA of the bootstrap table (§1b)
    /* +0x14 */ uint32_t stub_section_rva;   // [H] RVA inside the section header to drop
    /* +0x18 */ uint32_t overlay_offset;     // [H] file offset of the overlay directory (§6a)
    /* +0x1c */ uint32_t block_rva;          // [H] this block's own RVA, which makes the table reachable from any load base
    /* +0x20 */ void *abort_callback;        // [H] `ExitProcess`, copied from the bootstrap table; `g_safedisc_abort_callback`
    /* +0x24 */ void *extracted_dll;         // [H] the handle of the loaded `~df394b.tmp`
    /* +0x28 */ void *free_library;          // [H] `FreeLibrary`, for the DLL form's cleanup (§4a)
    /* +0x2c */ void *lock_file;             // [H] the handle of `~efe2.tmp`
    /* +0x30 */ void *close_handle;          // [H] `CloseHandle`, for the DLL form's cleanup (§4a)
    /* +0x34 */ uint8_t mode;                // [H] `g_safedisc_stub_mode`; 0 in the shipped file, non-zero selects the DLL form
} S_safedisc_stub_block;
```

### 1b. `S_safedisc_bootstrap_table` — at `0x007020fa`

The start of `stxt371`'s own import address table (`crack.md` §3), so the OS loader fills it in the retail file. In the file each slot holds the RVA of a hint and
name. `safedisc_run` reaches it as `block + (bootstrap_rva - block_rva)`.

```c
typedef struct S_safedisc_bootstrap_table {
    /* +0x00 */ void *get_module_handle_a;   // [H] `GetModuleHandleA`, opens `KERNEL32.dll` and `USER32.dll`
    /* +0x04 */ void *get_proc_address;      // [H] `GetProcAddress`
    /* +0x08 */ void *global_alloc;          // [H] `GlobalAlloc`
    /* +0x0c */ void *global_free;           // [H] `GlobalFree`
    /* +0x10 */ void *exit_process;          // [H] `ExitProcess`, stored as the abort callback
    /* +0x14 */ void *kernel32_end;          // [H] the import list's zero terminator
    /* +0x18 */ void *message_box_a;         // [H] `MessageBoxA`, the first slot of the USER32 list
} S_safedisc_bootstrap_table;
```

### 1c. `S_safedisc_api_table` — on `safedisc_run`'s stack, `0x90` bytes

The first of two tables the context on the stack points to. Three slots come from §1b; `safedisc_resolve_kernel32_imports` resolves the other 33 by name.

```c
typedef struct S_safedisc_api_table {
    /* +0x00 */ void *get_proc_address; // [H] `GetProcAddress`, copied from the bootstrap table by `safedisc_run`
    /* +0x04 */ void *get_module_file_name_a; // [H] `GetModuleFileNameA`, resolved by name
    /* +0x08 */ void *get_module_handle_a; // [H] `GetModuleHandleA`, resolved by name
    /* +0x0c */ void *virtual_protect; // [H] `VirtualProtect`, resolved by name
    /* +0x10 */ void *create_file_a; // [H] `CreateFileA`, resolved by name
    /* +0x14 */ void *read_file; // [H] `ReadFile`, resolved by name
    /* +0x18 */ void *write_file; // [H] `WriteFile`, resolved by name
    /* +0x1c */ void *close_handle; // [H] `CloseHandle`, resolved by name
    /* +0x20 */ void *delete_file_a; // [H] `DeleteFileA`, resolved by name
    /* +0x24 */ void *set_file_pointer; // [H] `SetFilePointer`, resolved by name
    /* +0x28 */ void *get_file_size; // [H] `GetFileSize`, resolved by name
    /* +0x2c */ void *global_alloc; // [H] `GlobalAlloc`, copied from the bootstrap table by `safedisc_run`
    /* +0x30 */ void *global_free; // [H] `GlobalFree`, copied from the bootstrap table by `safedisc_run`
    /* +0x34 */ void *local_free; // [H] `LocalFree`, resolved by name
    /* +0x38 */ void *get_windows_directory_a; // [H] `GetWindowsDirectoryA`, resolved by name
    /* +0x3c */ void *get_temp_path_a; // [H] `GetTempPathA`, resolved by name
    /* +0x40 */ void *get_tick_count; // [H] `GetTickCount`, resolved by name
    /* +0x44 */ void *create_directory_a; // [H] `CreateDirectoryA`, resolved by name
    /* +0x48 */ void *remove_directory_a; // [H] `RemoveDirectoryA`, resolved by name
    /* +0x4c */ void *get_drive_type_a; // [H] `GetDriveTypeA`, resolved by name
    /* +0x50 */ void *create_mutex_a; // [H] `CreateMutexA`, resolved by name
    /* +0x54 */ void *get_file_attributes_a; // [H] `GetFileAttributesA`, resolved by name
    /* +0x58 */ void *get_current_process_id; // [H] `GetCurrentProcessId`, resolved by name
    /* +0x5c */ void *create_process_a; // [H] `CreateProcessA`, resolved by name
    /* +0x60 */ void *release_mutex; // [H] `ReleaseMutex`, resolved by name
    /* +0x64 */ void *get_current_thread; // [H] `GetCurrentThread`, resolved by name
    /* +0x68 */ void *get_current_process; // [H] `GetCurrentProcess`, resolved by name
    /* +0x6c */ void *duplicate_handle; // [H] `DuplicateHandle`, resolved by name
    /* +0x70 */ void *create_thread; // [H] `CreateThread`, resolved by name
    /* +0x74 */ void *wait_for_single_object; // [H] `WaitForSingleObject`, resolved by name
    /* +0x78 */ void *get_last_error; // [H] `GetLastError`, resolved by name
    /* +0x7c */ void *format_message_a; // [H] `FormatMessageA`, resolved by name
    /* +0x80 */ void *load_library_a; // [H] `LoadLibraryA`, resolved by name
    /* +0x84 */ void *free_library; // [H] `FreeLibrary`, resolved by name
    /* +0x88 */ void *write_process_memory; // [H] `WriteProcessMemory`, resolved by name
    /* +0x8c */ void *is_d_b_c_s_lead_byte; // [H] `IsDBCSLeadByte`, resolved by name
} S_safedisc_api_table;
```

### 1d. `S_safedisc_user32_table` — on `safedisc_run`'s stack, 8 bytes

```c
typedef struct S_safedisc_user32_table {
    /* +0x00 */ void *message_box_a;         // [H] `MessageBoxA`, copied from the bootstrap table
    /* +0x04 */ void *wsprintf_a;            // [H] `wsprintfA`, resolved by name
} S_safedisc_user32_table;
```

### 1e. `S_safedisc_export_context` — the argument of the extracted DLL's export, `0x18` bytes

Built at `0x006ff3c4` and passed to the export `Ox12121212`.

```c
typedef struct S_safedisc_export_context {
    /* +0x00 */ void *module_base;           // [H] `safedisc_run`'s first argument, or `GetModuleHandleA(0)` when it is 0
    /* +0x04 */ void *headers_copy;          // [H] the `GlobalAlloc` copy of the PE headers, edited as in §4
    /* +0x08 */ uint32_t headers_size;       // [H] its size, from `safedisc_pe_headers_size_paged`
    /* +0x0c */ char *temp_dir;              // [M] the extraction directory buffer, `0x104` bytes
    /* +0x10 */ char *unique_subdir;         // [M] the sub-directory name buffer, `0x104` bytes
    /* +0x14 */ S_safedisc_stub_block *block; // [H] the block of §1a
} S_safedisc_export_context;
```

### 1f. `S_safedisc_directory_record` — one overlay directory record

The on-disk layout after the decryption of §6a. The file data follows the head, and the next record starts at offset + `header_size` + `data_size`.

```c
typedef struct S_safedisc_directory_record {
    /* +0x00 */ uint32_t marker_one;         // [H] 1 in every record; the walkers require it
    /* +0x04 */ uint32_t marker_zero;        // [H] 0 in every record; the walkers require it
    /* +0x08 */ uint32_t id;                 // [H] a record is extracted only when it equals the walker's id argument
    /* +0x0c */ uint32_t data_size;          // [H] the file data size
    /* +0x10 */ uint32_t header_size;        // [H] `0x1c` + name length + 1
    /* +0x14 */ uint32_t into_root;          // [H] 1: into the extraction directory itself; 0: into the sub-directory the caller names
    /* +0x18 */ uint32_t name_length;        // [H] the name's length without its NUL
    /* +0x1c */ char name[];                 // [H] the file name, NUL-terminated
} S_safedisc_directory_record;
```

## 2. Field evidence

### 2a. `S_safedisc_stub_block`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `import_rva` | H | - | `safedisc_restore_import_directory` | file value `0x00244fa8` |
| +0x04 | `import_size` | H | - | `safedisc_restore_import_directory` | `0xc8` |
| +0x08 | `iat_rva` | H | - | `safedisc_restore_import_directory` | `0x0023b000` |
| +0x0c | `iat_size` | H | - | `safedisc_restore_import_directory` | `0x254` |
| +0x10 | `bootstrap_rva` | H | - | `safedisc_run` | `0x003020fa` |
| +0x14 | `stub_section_rva` | H | - | `safedisc_pe_drop_last_section` | `0x002ff000`, the last section |
| +0x18 | `overlay_offset` | H | - | `safedisc_rollback_extraction` | `0x0025f000`, the first record of §6a; the maze of §7 hides the read in `safedisc_extract_archive` |
| +0x1c | `block_rva` | H | - | `safedisc_run` | `0x002ff009` |
| +0x20 | `abort_callback` | H | `safedisc_run` | `safedisc_stub_entry` | 0 in the file |
| +0x24 | `extracted_dll` | H | `safedisc_run` | `safedisc_run`, `safedisc_stub_entry` | - |
| +0x28 | `free_library` | H | `safedisc_run` | `safedisc_stub_entry` | - |
| +0x2c | `lock_file` | H | `safedisc_create_lock_file` | `safedisc_stub_entry` | - |
| +0x30 | `close_handle` | H | `safedisc_run` | `safedisc_stub_entry` | - |
| +0x34 | `mode` | H | - | `safedisc_stub_entry` | - |

### 2b. `S_safedisc_bootstrap_table`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `get_module_handle_a` | H | - | `safedisc_run` | filled by the OS loader in the retail file |
| +0x04 | `get_proc_address` | H | - | `safedisc_run` | as above |
| +0x08 | `global_alloc` | H | - | `safedisc_run` | as above |
| +0x0c | `global_free` | H | - | `safedisc_load_extracted_dll`, `safedisc_run` | as above |
| +0x10 | `exit_process` | H | - | `safedisc_run` | as above |
| +0x14 | `kernel32_end` | H | - | - | 0 in the file |
| +0x18 | `message_box_a` | H | - | `safedisc_run` | as above |

### 2c. `S_safedisc_api_table`

The Reader cells are a mention test: the functions whose body calls the slot in the shape the decompiler prints. `safedisc_extract_archive`'s maze (§7) prints only part
of its calls, so its cells are a lower bound, and the map follows from one function: `safedisc_resolve_kernel32_imports`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `get_proc_address` | H | `safedisc_run` | `safedisc_get_proc_address` | - |
| +0x04 | `get_module_file_name_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_payload` | - |
| +0x08 | `get_module_handle_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_run` | - |
| +0x0c | `virtual_protect` | H | `safedisc_resolve_kernel32_imports` | `safedisc_load_extracted_dll` | - |
| +0x10 | `create_file_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_create_lock_file`, `safedisc_extract_one_file`, `safedisc_extract_payload` | - |
| +0x14 | `read_file` | H | `safedisc_resolve_kernel32_imports` | `safedisc_decrypt_stream_to_file`, `safedisc_extract_archive`, `safedisc_rollback_extraction` | - |
| +0x18 | `write_file` | H | `safedisc_resolve_kernel32_imports` | `safedisc_decrypt_stream_to_file` | - |
| +0x1c | `close_handle` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_one_file`, `safedisc_extract_payload`, `safedisc_extract_under_mutex`, `safedisc_launch_loader_process`, `safedisc_run` | - |
| +0x20 | `delete_file_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_rollback_extraction` | - |
| +0x24 | `set_file_pointer` | H | `safedisc_resolve_kernel32_imports` | `safedisc_decrypt_stream_to_file`, `safedisc_extract_archive`, `safedisc_rollback_extraction` | - |
| +0x28 | `get_file_size` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_archive`, `safedisc_rollback_extraction` | - |
| +0x2c | `global_alloc` | H | `safedisc_run` | `safedisc_decrypt_stream_to_file`, `safedisc_string_decode_alloc` | - |
| +0x30 | `global_free` | H | `safedisc_run` | `safedisc_decrypt_stream_to_file`, `safedisc_get_proc_address`, `safedisc_run` | - |
| +0x34 | `local_free` | H | `safedisc_resolve_kernel32_imports` | `safedisc_show_error_box` | - |
| +0x38 | `get_windows_directory_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_temp_dir` | - |
| +0x3c | `get_temp_path_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_temp_dir` | - |
| +0x40 | `get_tick_count` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_unique_subdir` | - |
| +0x44 | `create_directory_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_dir_tree`, `safedisc_make_unique_subdir` | - |
| +0x48 | `remove_directory_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_dir_tree`, `safedisc_rollback_extraction` | - |
| +0x4c | `get_drive_type_a` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x50 | `create_mutex_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_under_mutex` | - |
| +0x54 | `get_file_attributes_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_make_dir_tree`, `safedisc_make_unique_subdir` | - |
| +0x58 | `get_current_process_id` | H | `safedisc_resolve_kernel32_imports` | `safedisc_launch_loader_process` | - |
| +0x5c | `create_process_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_launch_loader_process` | - |
| +0x60 | `release_mutex` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_under_mutex` | - |
| +0x64 | `get_current_thread` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x68 | `get_current_process` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x6c | `duplicate_handle` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x70 | `create_thread` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x74 | `wait_for_single_object` | H | `safedisc_resolve_kernel32_imports` | `safedisc_extract_under_mutex` | - |
| +0x78 | `get_last_error` | H | `safedisc_resolve_kernel32_imports` | `safedisc_create_lock_file`, `safedisc_decrypt_stream_to_file`, `safedisc_extract_archive`, `safedisc_extract_one_file`, `safedisc_extract_payload`, `safedisc_extract_under_mutex`, `safedisc_get_proc_address`, `safedisc_launch_loader_process`, `safedisc_load_extracted_dll`, `safedisc_make_dir_tree`, `safedisc_make_unique_subdir`, `safedisc_string_decode_alloc` | - |
| +0x7c | `format_message_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_show_error_box` | - |
| +0x80 | `load_library_a` | H | `safedisc_resolve_kernel32_imports` | `safedisc_load_library_from_dir` | - |
| +0x84 | `free_library` | H | `safedisc_resolve_kernel32_imports` | `safedisc_run` | - |
| +0x88 | `write_process_memory` | H | `safedisc_resolve_kernel32_imports` | - | no call in the stub |
| +0x8c | `is_d_b_c_s_lead_byte` | H | `safedisc_resolve_kernel32_imports` | `safedisc_create_lock_file`, `safedisc_extract_one_file`, `safedisc_launch_loader_process`, `safedisc_load_library_from_dir`, `safedisc_make_dir_tree`, `safedisc_path_append`, `safedisc_rollback_extraction` | passed as an argument to the `safedisc_mbs*` helpers |

### 2d. `S_safedisc_user32_table`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `message_box_a` | H | `safedisc_run` | `safedisc_show_error_box` | passed on to `safedisc_message_box` |
| +0x04 | `wsprintf_a` | H | `safedisc_resolve_user32_imports` | - | no call located in the stub |

### 2e. `S_safedisc_export_context`

Every reader is the export itself, inside the extracted DLL, so no reader is in this image.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `module_base` | H | `safedisc_run` | - | - |
| +0x04 | `headers_copy` | H | `safedisc_run` | - | - |
| +0x08 | `headers_size` | H | `safedisc_run` | - | - |
| +0x0c | `temp_dir` | M | `safedisc_run` | - | named from the arguments of `safedisc_load_library_from_dir` |
| +0x10 | `unique_subdir` | M | `safedisc_run` | - | as above |
| +0x14 | `block` | H | `safedisc_run` | - | - |

### 2f. `S_safedisc_directory_record`

The records are data of `Game-uncracked.exe`, written by whoever built the file. The reads inside `safedisc_extract_archive` are from a trace through the maze of §7.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `marker_one` | H | - | `safedisc_extract_archive`, `safedisc_rollback_extraction` | the walk ends at the first record that fails |
| +0x04 | `marker_zero` | H | - | `safedisc_extract_archive`, `safedisc_rollback_extraction` | as above |
| +0x08 | `id` | H | - | `safedisc_extract_archive` | - |
| +0x0c | `data_size` | H | - | `safedisc_extract_archive`, `safedisc_extract_one_file`, `safedisc_rollback_extraction` | - |
| +0x10 | `header_size` | H | - | `safedisc_extract_archive`, `safedisc_extract_one_file`, `safedisc_rollback_extraction` | - |
| +0x14 | `into_root` | H | - | `safedisc_extract_one_file` | - |
| +0x18 | `name_length` | H | - | `safedisc_extract_archive` | - |
| +0x1c | `name` | H | - | `safedisc_extract_one_file`, `safedisc_rollback_extraction` | - |

## 3. Method family

Every function in `stxt371` is named and each is a free function: no body has a `this`, and the context travels as an argument. The stub is PC-only (§10).

### 3a. Construction & destruction

The stub has no constructor.

### 3b. Accessors

The stub has no accessor.

### 3c. Behaviour

The stub has no function with a `this`.

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x006ff05e | `safedisc_stub_entry` | - | the retail entry point; rewrites its own first bytes, calls `safedisc_run` and goes on to the game (§4) |
| 0x006ff119 | `safedisc_jmp_oep` | - | five bytes, `jmp crt_startup`: the original entry point |
| 0x006ff120 | `safedisc_run` | - | the orchestrator, every step of §4 |
| 0x006ff430 | `safedisc_resolve_kernel32_imports` | - | fills 33 slots of the API table by decoded name |
| 0x006ff8c0 | `safedisc_resolve_user32_imports` | - | fills `wsprintfA` in the second table; returns whether the `GetLastError` slot is set |
| 0x006ff900 | `safedisc_extract_under_mutex` | - | takes the named mutex, then extracts and launches on success |
| 0x006ff9d0 | `safedisc_get_proc_address` | - | decodes an API name and resolves it; reports a failure |
| 0x006ffa20 | `safedisc_restore_import_directory` | - | copies the block's four directory values into a PE header's `DataDirectory[1]` and `[12]` |
| 0x006ffa70 | `safedisc_load_extracted_dll` | - | loads string `0x00`, `~df394b.tmp`, from the extraction directory |
| 0x006ffad0 | `safedisc_load_library_from_dir` | - | joins two directories and a file name, tries `LoadLibraryA`, then the bare name |
| 0x006ffc20 | `safedisc_pe_get_headers` | - | stores the DOS header and the NT header address |
| 0x006ffc40 | `safedisc_pe_drop_last_section` | - | finds the section of the block's `stub_section_rva`, decrements the count and zeroes its `0x28`-byte header |
| 0x006ffc90 | `safedisc_pe_section_for_rva` | - | computes the section-table range and looks the RVA up |
| 0x006ffce0 | `safedisc_pe_get_section_count` | - | reads `NumberOfSections` |
| 0x006ffd10 | `safedisc_pe_set_section_count` | - | writes `NumberOfSections` |
| 0x006ffd40 | `safedisc_pe_section_lookup` | - | the `0x28`-byte header walk by `VirtualAddress` and size |
| 0x006ffd80 | `safedisc_pe_headers_size_paged` | - | the header size rounded up to a page |
| 0x006ffdd0 | `safedisc_extract_payload` | - | opens the module file, makes the directories, extracts id 0, creates the lock file, rolls back on failure |
| 0x006fff20 | `safedisc_make_temp_dir` | - | `GetTempPathA`, falling back to `\Temp` under the Windows directory on error 0x70 or 5 |
| 0x006ffff0 | `safedisc_make_unique_subdir` | - | formats `GetTickCount` onto `\!E `, up to 10000 tries |
| 0x007001c0 | `safedisc_path_append` | - | appends a component, keeping one separator |
| 0x00700230 | `safedisc_make_dir_tree` | - | creates each missing component of a path, and proves the leaf writable with a probe directory `a` |
| 0x00700490 | `safedisc_extract_archive` | - | walks the overlay directory (§6a); carries the anti-disassembly of §7 |
| 0x007008c0 | `safedisc_extract_one_file` | - | creates one output file and decrypts into it (§8) |
| 0x00700a90 | `safedisc_decrypt_stream_to_file` | - | the payload cipher in `0x800`-byte blocks (§6) |
| 0x00700c00 | `safedisc_rollback_extraction` | - | walks the directory again, deletes every written file and the sub-directory |
| 0x00700e90 | `safedisc_create_lock_file` | - | creates string `0x16`, `~efe2.tmp`, and keeps the handle in the block's `+0x2c` |
| 0x00701030 | `safedisc_launch_loader_process` | - | starts string `0x17`, `~f1d055.tmp`, with the current process id on its command line |
| 0x00701370 | `safedisc_string_at` | - | the address of entry n of the ciphertext blob (§5) |
| 0x00701670 | `safedisc_string_decode` | - | the string cipher (§5) |
| 0x007016c0 | `safedisc_string_decode_alloc` | - | decodes into a new `GlobalAlloc` block |
| 0x00701710 | `safedisc_report_getprocaddress_error` | - | error box naming string `0x14` |
| 0x00701780 | `safedisc_report_createfile_error` | - | error box naming string `0x07` |
| 0x00701800 | `safedisc_report_readfile_error` | - | error box naming string `0x08` |
| 0x00701870 | `safedisc_report_writefile_error` | - | error box naming string `0x09` |
| 0x007018d0 | `safedisc_report_loadlibrary_error` | - | error box naming string `0x27` |
| 0x00701950 | `safedisc_report_globalalloc_error` | - | error box naming string `0x26` |
| 0x007019b0 | `safedisc_report_createdirectory_error` | - | error box naming string `0x0f` |
| 0x00701a30 | `safedisc_report_createprocess_error` | - | error box naming string `0x21` |
| 0x00701ab0 | `safedisc_report_getfilesize_error` | - | error box naming string `0x29` |
| 0x00701b20 | `safedisc_report_createmutex_error` | - | error box naming string `0x31` |
| 0x00701b90 | `safedisc_format_error_prefix` | - | formats `Error: <api> <detail>` |
| 0x00701c40 | `safedisc_show_error_box` | - | `FormatMessageA`, the box, `LocalFree` |
| 0x00701dc0 | `safedisc_stub_ret0` | - | `xor eax,eax; ret`; `safedisc_run` calls it twice |
| 0x00701dd0 | `safedisc_message_box` | - | `(*fn)(0, text, caption, 0x10)` |
| 0x00701df0 | `safedisc_itoa` | - | the runtime's `itoa`: negative flag for radix 10, then `safedisc_xtoa` |
| 0x00701e30 | `safedisc_xtoa` | - | the runtime's `xtoa` |
| 0x00701ea0 | `safedisc_mbsrchr` | - | last match, skipping trail bytes through the passed `IsDBCSLeadByte` |
| 0x00701ee0 | `safedisc_mbschr` | - | first match, the same walk |
| 0x00701f20 | `safedisc_mbslen` | - | byte length by the same walk |

`safedisc_itoa` and `safedisc_xtoa` are **statement-identical to `crt_itoa` and `crt_xtoa`** in `.text` and are not duplicates of them: different PE sections, so no
identical-code fold is possible. SafeDisc linked the same MSVC helpers into its own object.

## 4. What it does, in order

`safedisc_stub_entry` (`0x006ff05e`), the retail PE's `AddressOfEntryPoint`, is **self-modifying**. It reads the mode byte, computes a `jmp` displacement and overwrites its own
first five bytes with it; the section is writable, so a second entry short-circuits. In mode 0 the displacement comes from `safedisc_jmp_oep`, so re-entry goes straight to the
game. On a failed `safedisc_run` mode 0 calls `abort_callback` with the failure code.

`safedisc_run` (`0x006ff120`) then does the work:

1. decodes `KERNEL32.dll` and `USER32.dll`, opens them through the bootstrap table and resolves both API tables;
2. takes the module base and copies the PE headers into a `GlobalAlloc` block sized by `safedisc_pe_headers_size_paged`;
3. calls `safedisc_extract_under_mutex`, which holds the mutex `XeTuM bOY759` around extracting the payload (`safedisc_extract_payload`) and starting `~f1d055.tmp` as a child
   process (`safedisc_launch_loader_process`);
4. **writes the game's import data directories into the heap copy** (`safedisc_restore_import_directory`), and **drops the stub's section header from the heap copy**
   (`safedisc_pe_drop_last_section`);
5. calls `safedisc_stub_ret0`, loads `~df394b.tmp` (`safedisc_load_extracted_dll`) and stores its handle in the block, and calls `safedisc_stub_ret0` again;
6. resolves that DLL's export `Ox12121212` and calls it with the context of §1e; a non-zero result returns failure;
7. frees the heap copy and returns 0, and `safedisc_stub_entry` goes on to `crt_startup`.

`safedisc_run` never writes the header of the loaded image: steps 4 hand the edited copy to the export. Whether the export applies it is Open item 1.

The extraction: `safedisc_extract_payload` opens the module's own file, makes a unique temp directory, and `safedisc_extract_archive` walks the directory of variable-length
records appended to the executable (§6a, each read through a `0x121`-byte buffer), handing each selected record to `safedisc_extract_one_file`, which creates the output file and
runs the payload cipher into it. `safedisc_create_lock_file` then creates `~efe2.tmp`. On failure `safedisc_rollback_extraction` deletes everything it wrote.

### 4a. The non-zero mode: the DllMain wrapper

When `mode` is non-zero, `safedisc_stub_entry` behaves as the entry point of a DLL. It passes its first stack argument, DllMain's `hinstDLL`, to `safedisc_run` (mode 0 passes 0);
it rewrites its own first bytes into a `jmp` to the handler at `0x006ff0d3` instead of to the original entry point; and on a failed `safedisc_run` it rewrites itself into `ret 0xc` and
re-enters, returning 0 with DllMain's three arguments popped.

The handler passes every call straight to the original entry point except `DLL_PROCESS_DETACH` (`fdwReason == 0`): for that one it saves its return address at `0x006ff113` and
swaps it, so that after the real DllMain returns, two cleanup calls run: `close_handle(lock_file)` and `free_library(extracted_dll)`. The shipped byte is 0, so in `Game.exe` this path
never runs.

## 5. The obfuscated string table

`safedisc_string_at` indexes a NUL-separated ciphertext blob at `g_safedisc_string_table` (`0x007013d9`, 639 bytes, bounded by the literal `0x00701658`). The blob sits **inside the
function's own code**: the scan loop ends `xor ecx,ecx; test ecx,ecx; je 0x00701658` at `0x007013cf`, a jump that is always taken over the table, and the epilogue
`mov eax,[ebp-8]; pop edi; pop esi; pop ebx; mov esp,ebp; pop ebp; ret` follows at `0x00701658`. In the Ghidra database the `je` carries a BRANCH flow override and the function body is the
two code ranges `[0x00701370, 0x007013d8]` + `[0x00701658, 0x00701661]`; the table is undefined bytes between them. `safedisc_string_decode` is the cipher:

```
p[0] = c[0]
p[i] = (c[i] - 1) ^ p[i-1]        for i >= 1, until c[i] == 0
```

A running XOR over the previous **plaintext** byte. All 53 entries decode:

| idx | plaintext | idx | plaintext | idx | plaintext |
|---|---|---|---|---|---|
| `0x00` | `~df394b.tmp` | `0x13` | `WaitForSingleObject` | `0x27` | `LoadLibraryA` |
| `0x01` | `Ox12121212` | `0x14` | `GetProcAddress` | `0x28` | `FreeLibrary` |
| `0x02` | `GetModuleHandleA` | `0x15` | `\C-Dilla` | `0x29` | `GetFileSize` |
| `0x03` | `GetModuleFileNameA` | `0x16` | `~efe2.tmp` | `0x2a` | `wsprintfA` |
| `0x04` | `GetCurrentThread` | `0x17` | `~f1d055.tmp` | `0x2b` | `WriteProcessMemory` |
| `0x05` | `GetCurrentProcess` | `0x18` | `\!E ` | `0x2c` | `IsDBCSLeadByte` |
| `0x06` | `VirtualProtect` | `0x1e` | `GetDriveTypeA` | `0x2d` | `\Temp` |
| `0x07` | `CreateFileA` | `0x1f` | `GetFileAttributesA` | `0x2e` | `GetWindowsDirectoryA` |
| `0x08` | `ReadFile` | `0x20` | `GetCurrentProcessId` | `0x2f` | `KERNEL32.dll` |
| `0x09` | `WriteFile` | `0x21` | `CreateProcessA` | `0x30` | `USER32.dll` |
| `0x0a` | `CloseHandle` | `0x22` | `GetLastError` | `0x31` | `CreateMutexA` |
| `0x0b` | `DeleteFileA` | `0x23` | `FormatMessageA` | `0x32` | `ReleaseMutex` |
| `0x0c` | `SetFilePointer` | `0x24` | `LocalFree` | `0x33` | `XeTuM bOY759` |
| `0x0d` | `GetTempPathA` | `0x25` | `Error` | - | - |
| `0x0e` | `GetTickCount` | `0x26` | `GlobalAlloc` | - | - |
| `0x0f` | `CreateDirectoryA` | `0x10` | `RemoveDirectoryA` | - | - |
| `0x11` | `DuplicateHandle` | `0x12` | `CreateThread` | - | - |

Entries `0x19`–`0x1d` are empty. `XeTuM bOY759` is the **global mutex name**, SafeDisc's own: the whole blob is inside the 36-byte-identical `stxt371`. `\C-Dilla` is the Macrovision
C-Dilla driver directory.

## 6. The payload cipher

`safedisc_decrypt_stream_to_file` (`0x00700a90`) copies a byte range from the module file to an output file through a plaintext-feedback stream cipher. Disassembly-verified at
`0x00700b1b`:

```
ebp = 0x39F2B5D9                                  ; key register, seeded once per file
per byte:
    ecx   = ebp ^ 0xE3A19D27
    plain = (ecx>>24) ^ (ecx>>16) ^ (ecx>>8) ^ (ecx & 0xff) ^ cipher
    buf[i]= plain
    ebp   = (ebp << 8) + plain                    ; PLAINTEXT feedback
```

The key register shifts in each recovered plaintext byte, so the stream is self-synchronising over the whole file. Blocks are `0x800` bytes.

**The payload cannot be decrypted from `Game.exe`**: it is an OEP memory dump with no overlay. `Game-uncracked.exe` carries the 754,902-byte overlay, and its directory records give
each file's offset.

### 6a. The overlay directory

`safedisc_extract_archive` reads one record at the current file offset into a `0x121`-byte buffer and decrypts it in place with a keystream seeded by that offset (the value
`SetFilePointer` returns):

```
s = offset
per byte:
    s     = s * 0x13C6A5 + 0xD8430DED          ; mod 2^32
    plain = cipher ^ ((s ^ s>>8 ^ s>>16 ^ s>>24) & 0xff)
```

`Game-uncracked.exe`'s overlay starts at file offset `0x25f000`, `overlay_offset`, and holds nine records (§1f) followed by an 84-byte tail that is not a record:

| file offset | id | data size | name |
|---|---|---|---|
| `0x25f000` | 1 | `0x2c345` | `~def8c9.tmp` |
| `0x28b36d` | 1100 | `0x2e90` | `clcd32.dll` |
| `0x28e224` | 1100 | `0x1016` | `clcd16.dll` |
| `0x28f261` | 1101 | `0x944f` | `mcp.dll` |
| `0x2986d4` | 2 | `0x19d5` | `SECDRV.SYS` |
| `0x29a0d0` | 2 | `0x49a8` | `DrvMgt.dll` |
| `0x29ea9f` | 11 | `0x1542` | `SecDrv04.VxD` |
| `0x2a000a` | 0 | `0xb600` | `~f1d055.tmp` |
| `0x2ab632` | 0 | `0x6be28` | `~df394b.tmp` |

`safedisc_extract_payload` passes id 0, so the stub extracts only the last two; `~f1d055.tmp` is the one record with `into_root` = 1. Both decrypt to `MZ` with the §6 cipher. The other
seven are left for a later stage: none of them reads as the §6 stream at its offset, and all seven begin with the same raw bytes `5f 60`.

## 7. Anti-disassembly in `safedisc_extract_archive`

`safedisc_extract_archive` (`0x00700490`) carries dense **opaque-predicate** anti-disassembly from `0x007005b8` onward: complementary `jcc` pairs (`74 15`/`75 0f` = `je`/`jne`, `7a 0c`/`7b dc`
= `jp`/`jnp`, `7e 1d`/`7f 0f` = `jle`/`jg`) interleaved with `87 ed` / `8b ff` / `90 90` register-swap filler whose bytes double as the branch displacements. Both branches of each pair are
taken, so the byte between them is never executed, and a linear disassembler is permanently desynchronised. Ghidra renders `halt_baddata()`; Capstone loses sync at the same instruction.

A trace that carries the flag values each taken branch implies resolves the maze: on every path the conditional jumps that follow a known flag have one successor, and what remains is the
directory walk of §6a: the record read, the in-place keystream decryption, the three field tests and the call to `safedisc_extract_one_file`.

Ghidra's `size` for it is **329**, because the size stops at the first `ret` run; the body actually runs to `0x007008b9`, about 1065 bytes.

## 8. `safedisc_extract_one_file` and its two false starts

The function that occupies `[0x007008c0, 0x00700a8c)` is 460 bytes, one body in the Ghidra database. It has two alternate exit arms that an instruction finder takes for function starts; neither
is a function in the Ghidra database, and neither has a def in the decomp:

| VA | Proof |
|---|---|
| `0x00700a2a` | `jne 0x700a2a` at `0x00700a13` and `0x00700a19`; preceded by `5b 81 c4 04 01 00 00 c3` = `pop ebx; add esp,0x104; ret`; unaligned |
| `0x00700a4e` | `jne 0x700a4e` at `0x00700a0b`; identical preceding bytes; unaligned |

The two arm bodies are distinct, not duplicates. `0x00700a4e` is **not** inside `safedisc_get_proc_address` (`0x006ff9d0`), which is 74 bytes and ends at `0x006ffa1a`: Ghidra's default analysis is
call-graph driven and contains almost nothing from this orphaned stub, so a nearest-preceding-start lookup against it lands on the wrong host. Any phantom check over this band must accept every
**executable** PE section, parsed from the binary's own section table: a `.text`-only census of `E8` calls, datarefs and branch targets reports every VA here as unreferenced.

## 9. Data symbols

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_safedisc_saved_data_directories` | `0x006ff009` | the block of §1a, named for its first four dwords | `safedisc_create_lock_file`, `safedisc_run` | `safedisc_pe_drop_last_section`, `safedisc_restore_import_directory`, `safedisc_rollback_extraction`, `safedisc_run` |
| `g_safedisc_abort_callback` | `0x006ff029` | the block's `abort_callback`; zero in the dump | `safedisc_run` | `safedisc_stub_entry` |
| `g_safedisc_stub_mode` | `0x006ff03d` | the block's `mode` | - | `safedisc_stub_entry` |
| `g_safedisc_string_table` | `0x007013d9` | the 639-byte blob inside `safedisc_string_at`'s code (§5) | - | `safedisc_string_at` |

The block's first 16 bytes read `a8 4f 24 00 c8 00 00 00 00 b0 23 00 54 02 00 00`: byte for byte the four values `crack.md` §3b derives independently from a header diff against the retail file.

## 10. PC vs PS2

There is nothing to compare. `safedisc` and `dilla` return **0** hits in the PS2 symbol table: the PS2 has no disc protection of this kind, and that is a measurement over a closed set, not a failed
search.

## 11. Related

`basic/artifacts/crack.md` (what was modified: the SafeDisc encryption, both cracks, the four patched bytes, the banner; the authority on all of that) · `structs/crt_runtime_tail.md` (the MSVC 6 runtime
the stub's own `itoa` and `xtoa` were linked from) · `Game-ghidra.exe.functions.md` (the function registry).

## 12. Open items

| Item | State |
|---|---|
| the extracted DLL's export | `Ox12121212` in `~df394b.tmp` receives the edited header copy (§1e). Whether it writes the copy to the loaded image, and what it does with the two directories, was not read. The file decrypts to `MZ` with the §6 cipher, so it can be analysed offline |
| the tail of the bootstrap table | `S_safedisc_bootstrap_table` stops at the slots the stub reads: `safedisc_run` reads `+0x00`..`+0x10` and `+0x18`, `safedisc_load_extracted_dll` reads `+0x0c`, and no stub body reads past `+0x18`; the rest of the `0x58` bytes (`crack.md` §3) was not typed |
| the 84-byte overlay tail | it is not a directory record; its meaning was not read |
