# The SafeDisc 2.70.030 protection stub — `stxt774` + `stxt371`

**Created** 2026-09-07 16:52 +0200 · **Modified** 2026-09-28 19:40 +0200

50 functions, all named, prefix `safedisc_`. PC range `[0x00633d75, 0x00701f20]`.

> **This document is about what the protection IS.** What it *did to the game* — the
> encryption, the crack, the four patched bytes, the banner — is `basic/artifacts/crack.md`, and
> that file remains the authority on every modification. Read it first; this file only adds the
> code behind its §3 and §4 claims.

## 0. The one-paragraph version

`Game.exe` carries two extra PE sections that are not part of the game: `stxt774`, 8 KB of
encrypted payload, and `stxt371`, 13 KB of x86 code. That code is Macrovision **SafeDisc
2.70.030**'s loader. It resolves 34 Win32 imports by name from an obfuscated string table,
restores the game's real PE import directories, unpacks three temp files out of the
executable through a stream cipher, launches one of them as a child process, loads another
as a DLL, and jumps to the game's original entry point. **In our binary it never runs** —
the crack moved `AddressOfEntryPoint` back to `crt_startup`, so the whole section is
orphaned.

**It is the only body of code in this project whose bytes the retail executable can
authenticate.** `crack.md` §3a records that retail `.text` and `.data` are encrypted and
cannot supply a single byte of game code. `stxt*` is plaintext in both files: measured,
`stxt774` differs from `Game-uncracked.exe` in **0 bytes** and `stxt371` in exactly **36**
— the six known §5e DLL-name case edits. Every identification below is retail-verified.

## 1. The two sections

| section | VA | vsize | characteristics | contents |
|---|---|---|---|---|
| `stxt774` | `0x006fc000` | `0x204f` | `0xE0000020` = code, read, write, execute | **encrypted payload data, not code** ¹ |
| `stxt371` | `0x006ff000` | `0x325c` | `0xE0000020` | the loader: 50 functions, a 53-entry obfuscated string table, and a saved-header block |

¹ High entropy; the bytes at the section start are `e0 04 34 92 b4 a4 b7 ab …`.

* **`stxt774` contains no functions.** Ghidra creates one at `0x006fc000` whose body decompiles
  to `swi(4)` — data past the end of `.text` decoded as code. It has no def in the decomp.
* **VA→file is NOT the identity mapping for these two sections**, unlike `.text`. `stxt774` VA
  `0x006fc000` is at file `0x258000`, a delta of `0x4A4000`. Parse the section table.

## 2. What it does, in order

`safedisc_stub_entry` (`0x006ff05e`) — the retail PE's `AddressOfEntryPoint` — is
**self-modifying**. It reads the mode byte `g_safedisc_stub_mode`, computes a `jmp`
displacement, and overwrites its own first five bytes with it; the section is writable, so
a second entry short-circuits. In mode 0 the displacement comes from `safedisc_jmp_oep`, so
re-entry goes straight to the game.

Then `safedisc_run` (`0x006ff120`) does the work:

1. decode `KERNEL32.dll` / `USER32.dll` and resolve both API tables
   (`safedisc_resolve_kernel32_imports`, `safedisc_resolve_user32_imports`);
2. copy the PE headers into a `GlobalAlloc` page sized by
   `safedisc_pe_headers_size_paged`;
3. **restore the game's import data directories** (`safedisc_restore_import_directory`);
4. **drop the stub's own section header** (`safedisc_pe_drop_last_section`), so the loaded
   image does not mention it;
5. extract the payload under a named mutex (`safedisc_extract_under_mutex`);
6. load the extracted DLL (`safedisc_load_extracted_dll`) and call into it;
7. `jmp` to `safedisc_jmp_oep` → `crt_startup`.

The extraction itself: `safedisc_extract_payload` opens the module's own file, makes a
unique temp directory, and `safedisc_extract_archive` walks a directory of **0x121-byte
entry records** appended to the executable, handing each to `safedisc_extract_one_file`,
which creates the output file and runs the payload cipher into it. On failure
`safedisc_rollback_extraction` deletes everything it wrote.

## 3. The obfuscated string table — decoded

`safedisc_string_at` indexes a NUL-separated ciphertext blob at
`g_safedisc_string_table` (`0x007013d9`, 639 bytes, bounded by the literal `0x00701658`).
The blob sits **inside the function's own code**: the scan loop ends
`xor ecx,ecx; test ecx,ecx; je 0x00701658` at `0x007013cf`, a jump that is always taken over
the table, and the epilogue `mov eax,[ebp-8]; pop edi; pop esi; pop ebx; mov esp,ebp; pop ebp;
ret` follows at `0x00701658` (the early exit at `0x007013b6` jumps there too). In the Ghidra
database the `je` carries a BRANCH flow override and the function body is the two code ranges
`[0x00701370, 0x007013d8]` + `[0x00701658, 0x00701661]`; the table is undefined bytes between them.
`safedisc_string_decode` is the cipher:

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

Entries `0x19`–`0x1d` are empty. `XeTuM bOY759` is the **global mutex name** — SafeDisc's
own, not a cracker artifact: the whole blob is inside the 36-byte-identical `stxt371`.
`\C-Dilla` is the Macrovision C-Dilla driver directory.

## 4. The API dispatch table — 33 slots

`safedisc_resolve_kernel32_imports` fills `+0x04..+0x8c` of the stub's context. The map follows
from that one function and is confirmed by nineteen independent consumers elsewhere in the band,
every offset agreeing:

| off | API | off | API | off | API |
|---|---|---|---|---|---|
| `+0x04` | GetModuleFileNameA | `+0x38` | GetWindowsDirectoryA | `+0x68` | GetCurrentProcess |
| `+0x08` | GetModuleHandleA | `+0x3c` | GetTempPathA | `+0x6c` | DuplicateHandle |
| `+0x0c` | VirtualProtect | `+0x40` | GetTickCount | `+0x70` | CreateThread |
| `+0x10` | CreateFileA | `+0x44` | CreateDirectoryA | `+0x74` | WaitForSingleObject |
| `+0x14` | ReadFile | `+0x48` | RemoveDirectoryA | `+0x78` | GetLastError |
| `+0x18` | WriteFile | `+0x4c` | GetDriveTypeA | `+0x7c` | FormatMessageA |
| `+0x1c` | CloseHandle | `+0x50` | CreateMutexA | `+0x80` | LoadLibraryA |
| `+0x20` | DeleteFileA | `+0x54` | GetFileAttributesA | `+0x84` | FreeLibrary |
| `+0x24` | SetFilePointer | `+0x58` | GetCurrentProcessId | `+0x88` | WriteProcessMemory |
| `+0x28` | GetFileSize | `+0x5c` | CreateProcessA | `+0x8c` | IsDBCSLeadByte |
| `+0x34` | LocalFree | `+0x60` | ReleaseMutex | - | - |
| - | - | `+0x64` | GetCurrentThread | - | - |

`+0x2c` (GlobalAlloc) and `+0x30` (GlobalFree) are **not** filled here — they come from the
bootstrap descriptor `safedisc_run` reaches through a self-relocating displacement. The
second table, `ctx[1]`, holds `wsprintfA` at `+0x04` (filled by
`safedisc_resolve_user32_imports`) and the `MessageBoxA` the error reporter uses at `+0x00`.

## 5. The payload cipher

`safedisc_decrypt_stream_to_file` (`0x00700a90`) copies a byte range from the module file
to an output file through a plaintext-feedback stream cipher. Disassembly-verified at
`0x00700b1b`:

```
ebp = 0x39F2B5D9                                  ; key register, seeded once per file
per byte:
    ecx   = ebp ^ 0xE3A19D27
    plain = (ecx>>24) ^ (ecx>>16) ^ (ecx>>8) ^ (ecx & 0xff) ^ cipher
    buf[i]= plain
    ebp   = (ebp << 8) + plain                    ; PLAINTEXT feedback
```

The key register shifts in each recovered plaintext byte, so the stream is
self-synchronising over the whole file. Blocks are 0x800 bytes.

**The payload cannot be decrypted from `Game.exe`.** It is an OEP memory dump with no
overlay; the retail file has the 754,902-byte overlay, but the starting offset comes from
the 0x121-byte entry records that `safedisc_extract_archive` reads — and that function is
the one the anti-disassembly protects (§6).

## 6. Anti-disassembly in `safedisc_extract_archive`

`safedisc_extract_archive` (`0x00700490`) carries dense **opaque-predicate**
anti-disassembly from `0x007005b8` onward: complementary `jcc` pairs (`74 15`/`75 0f` =
`je`/`jne`, `7a 0c`/`7b dc` = `jp`/`jnp`, `7e 1d`/`7f 0f` = `jle`/`jg`) interleaved with
`87 ed` / `8b ff` / `90 90` register-swap filler whose bytes double as the branch
displacements. Both branches of each pair are taken, so the byte between them is never
executed — and a linear disassembler is permanently desynchronised. Ghidra renders
`halt_baddata()`; Capstone loses sync at the same instruction.

The row is **STRUCTURAL and capped**: its name rests on the disassembled prologue
(`GetFileSize` → `SetFilePointer` → a loop of 0x121-byte `ReadFile`s), its API set and its
callee set. The middle needs a recursive-descent trace (§11).

Ghidra's `size` for it is **329**, because the size stops at the first `ret` run; the body
actually runs to `0x007008b9`, about 1065 bytes.

## 7. `safedisc_extract_one_file` and its two false starts

The function that occupies `[0x007008c0, 0x00700a8c)` is 460 bytes, one body in the Ghidra
database. It has two alternate exit arms that an instruction finder takes for function starts;
neither is a function in the Ghidra database, and neither has a def in the decomp:

| VA | proof |
|---|---|
| `0x00700a2a` | `jne 0x700a2a` at `0x00700a13` and `0x00700a19`; preceded by `5b 81 c4 04 01 00 00 c3` = `pop ebx; add esp,0x104; ret`; unaligned |
| `0x00700a4e` | `jne 0x700a4e` at `0x00700a0b`; identical preceding bytes; unaligned |

* The two arm bodies are distinct, not duplicates.
* `0x00700a4e` is **not** inside `safedisc_get_proc_address` (`0x006ff9d0`): that function is
  74 bytes and ends at `0x006ffa1a`. Ghidra's default analysis is call-graph driven and contains almost
  nothing from this orphaned stub, so a nearest-preceding-start lookup against it lands on the
  wrong host.
* Any phantom check over this band must accept every **executable** PE section, parsed from the
  binary's own section table: a `.text`-only census of `E8` calls, datarefs and branch targets
  reports every VA here as unreferenced.

## 8. Function coverage — 50 of 50

Every function in `stxt371` is named. Full rows, with evidence, in
`ghidra/Game-ghidra.exe.functions.md`.

| band | functions |
|---|---|
| entry + orchestration | `safedisc_stub_entry`, `safedisc_jmp_oep`, `safedisc_run`, `safedisc_stub_ret0` |
| import resolution | `safedisc_resolve_kernel32_imports`, `safedisc_resolve_user32_imports`, `safedisc_get_proc_address` |
| PE header surgery | `safedisc_pe_get_headers`, `safedisc_pe_drop_last_section`, `safedisc_pe_section_for_rva`, `safedisc_pe_get_section_count`, `safedisc_pe_set_section_count`, `safedisc_pe_section_lookup`, `safedisc_pe_headers_size_paged`, `safedisc_restore_import_directory` |
| extraction | `safedisc_extract_under_mutex`, `safedisc_extract_payload`, `safedisc_extract_archive`, `safedisc_extract_one_file`, `safedisc_decrypt_stream_to_file`, `safedisc_rollback_extraction`, `safedisc_create_lock_file` |
| temp directories | `safedisc_make_temp_dir`, `safedisc_make_unique_subdir`, `safedisc_make_dir_tree`, `safedisc_path_append` |
| loading + launching | `safedisc_load_extracted_dll`, `safedisc_load_library_from_dir`, `safedisc_launch_loader_process` |
| strings | `safedisc_string_at`, `safedisc_string_decode`, `safedisc_string_decode_alloc` |
| error reporting | `safedisc_format_error_prefix`, `safedisc_show_error_box`, `safedisc_message_box`, and ten `safedisc_report_<api>_error` |
| helpers | `safedisc_itoa`, `safedisc_xtoa`, `safedisc_mbsrchr`, `safedisc_mbschr`, `safedisc_mbslen` |

`safedisc_itoa` / `safedisc_xtoa` are **statement-identical to `crt_itoa` / `crt_xtoa`** in
`.text` and are *not* duplicates of them: different PE sections, so no identical-code fold is
possible. SafeDisc statically linked the same MSVC helpers into its own object.

## 9. Data symbols

| VA | name | what it is |
|---|---|---|
| `0x006ff009` | `g_safedisc_saved_data_directories` | the saved `DataDirectory[1]` and `[12]` — see below |
| `0x006ff029` | `g_safedisc_abort_callback` | failure-path function pointer; zero in the dump |
| `0x006ff03d` | `g_safedisc_stub_mode` | the mode byte the entry stub branches on; zero in the shipped file |
| `0x007013d9` | `g_safedisc_string_table` | the 639-byte obfuscated blob inside `safedisc_string_at`'s code (§3) |

**`g_safedisc_saved_data_directories` closes `crack.md` §4 step 4 from the other end.**
Its bytes read `a8 4f 24 00 c8 00 00 00 00 b0 23 00 54 02 00 00` = Import RVA `0x00244fa8`
size `0xc8`, IAT RVA `0x0023b000` size `0x254` — byte for byte the four values `crack.md`
§3b derives independently from a header diff against the retail file.
`safedisc_restore_import_directory` writes them back at run time; the cracker simply
persisted the post-restore state into the file. Two artifacts, two methods, one answer.

## 10. PC vs PS2

There is nothing to compare. `safedisc` and `dilla` return **0** hits in the PS2 symbol
table: the PS2 has no disc protection of this kind, so every twin cell is `-`, and that is a
measurement over a closed set, not a failed search.

## 11. Open items

| item | state |
|---|---|
| `safedisc_extract_archive` past `0x007005b8` | unread (§6); a recursive-descent trace through the opaque predicates would decode the 0x121-byte entry record layout — the last gap in a full description of the payload format |
| `g_safedisc_stub_mode` non-zero branch | undecoded; the shipped byte is 0, so only the mode-0 path is exercised |
| `Ox12121212` (string `0x01`) | no located consumer in the functions read |

## 12. Related

* `basic/artifacts/crack.md` — what was *modified*: the SafeDisc encryption, both cracks, the four
  patched bytes, the banner. **The authority on all of that.**
* `structs/crt_runtime_tail.md` — the MSVC 6 runtime the stub's own `itoa`/`xtoa` were linked from.
