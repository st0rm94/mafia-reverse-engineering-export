# `C_chunk` — the `.dta` archive chunk reader/writer

**Created** 2026-08-14 23:01 +0200 · **Modified** 2026-09-17 13:46 +0200

**PS2 class:** `C_chunk` (mangled tag `7C_chunk`). **PC prefix: `chunk_`.** Cross-build material
is in §7.

`C_chunk` is the small stack object the engine builds over an open `.dta` archive file to
walk its **chunk tree**: a header of `{id:u16, size:u32}` records that nest, so reading a
file means descending into a chunk, consuming its payload, and ascending back out. It is
always constructed as a **local**, never heap-allocated — all three `chunk_ctor` callsites
are `lea ecx,[esp+N]` immediately followed by `chunk_open_file`.

Its file I/O goes through `rw_data.dll` (`_dtaOpen@8`, `_dtaRead@12`, `_dtaSeek@12`,
`_dtaWrite@12`, …). That is why the class is thin: it is the chunk grammar on top of an archive
API that lives outside `Game.exe`.

---

## 1. Field layout (PC)

```c
typedef struct C_chunk {
    /* +0x00 */ void   *vtable;        // [H] = g_chunk_vt — a 2-slot ALLOCATOR
                                       //     interface, not a polymorphic class vtable
                                       //     (slot 0 allocate, slot 1 free)
    /* +0x04 */ int    *stack_begin;   // [H] open-chunk stack: std::vector<int>, one
    /* +0x08 */ int    *stack_end;     // [H] file offset per chunk currently descended
    /* +0x0c */ int    *stack_cap;     // [H] into (see §4). Empty = at the top level.
    /* +0x10 */ uint8_t is_open;       // [H] 1 between chunk_open_file and chunk_close
    /* +0x14 */ int     dta_handle;    // [H] the rw_data.dll handle, or -1
    /* +0x18 */ uint8_t write_mode;    // [H] 0 = reading (dtaOpen), 1 = writing
                                       //     (dtaOpenWrite); flips the whole close path
    /* +0x1c */ int     file_size;     // [H] dtaSeek(h,0,SEEK_END) when reading; -1 when
                                       //     writing or when the open failed
} C_chunk;                             // 0x20 bytes
```

The allocator byte at `+0x04`'s low byte is copied from the optional argument
`chunk_ctor` takes; `+0x04..+0x0c` are then used as an ordinary vector.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `chunk_ctor` | - | `g_chunk_vt`, **two slots only** (§6). An STL allocator interface, not a class vtable — no `C_chunk` method is virtual |
| +0x04..+0x0c | open-chunk stack | H | `chunk_enter_next`, `chunk_leave`, `chunk_close` | `chunk_read`, `chunk_bytes_remaining`, `chunk_close` | `end - begin >> 2` is the depth; `*(end-1)` is the current chunk's recorded offset (§4) |
| +0x10 | is_open | H | `chunk_open_file`, `chunk_close` | `chunk_open_file`, `chunk_read`, `chunk_close` | `chunk_open_file` refuses to re-open an already-open chunk; `chunk_read` returns false when closed |
| +0x14 | dta_handle | H | `chunk_open_file` | `chunk_read`, `chunk_close` | passed straight to the `rw_data.dll` entry points |
| +0x18 | write_mode | H | `chunk_open_file` | `chunk_read`, `chunk_close` | `chunk_read` refuses when set; on ascend or close a write-mode chunk **back-patches its own size** ¹ |
| +0x1c | file_size | H | `chunk_open_file` | - | `dtaSeek(h,0,SEEK_END)` on the read path; left `-1` on the write path |

¹ `dtaSeek(h,0,CUR)`, subtract the recorded start, `dtaSeek(h, start+2, SET)`, `dtaWrite(&size,4)`,
seek back.

## 3. PC function set

| PC VA | PC name | role |
|---|---|---|
| 0x004be680 | `chunk_ctor` | installs the allocator vtable + copies the allocator byte. 3 callsites, all `lea ecx,[esp+N]` — a stack local every time |
| 0x004be6b0 | `chunk_open_file` | `dtaOpen`/`dtaOpenWrite`, records the handle, seeks to the end for the size on the read path, rewinds, sets `is_open` |
| 0x00453a50 | `chunk_read` | bounded read: refuses when closed or in write mode, clamps the request against the enclosing chunk's remaining bytes |
| 0x004be720 | `chunk_close` | in write mode unwinds the open-chunk stack, back-patching each chunk's size; then `dtaClose`, handle `-1`, frees the stack vector |
| 0x00544590 | `chunk_close_handle` | second emission of the close body (§5) |
| 0x005a5c10 | `chunk_open_stack_insert_at` | vector insert on the open-chunk stack; 18 callsites |
| 0x005a5db0 | `chunk_open_stack_insert_n` | vector fill-insert on the open-chunk stack; 1 callsite |
| 0x005a5fe0 | `chunk_enter_next` | reads the 6-byte header, pushes the chunk's end offset, returns the id (§4) |
| 0x005a61b0 | `chunk_leave` | ascends: back-patch in write mode, seek to the recorded offset, pop. **83 callsites**, 53 in `mission_open_bin` — the most-used `C_chunk` entry point |
| 0x005a6250 | `chunk_bytes_remaining` | `current_chunk_end - tell()`, 0 on closed / empty stack / seek failure; 18 callsites, each the condition of a `while (remaining) { enter; …; leave; }` loop |
| 0x005a6280 | `chunk_read_string_and_leave` | reads `min(remaining, 999)` bytes as a C string into a `std::string`, then ascends; caps every name in `effects.tbl` and `cache.bin` |
| 0x00607710 | `chunk_get_dta_handle` | `return is_open ? dta_handle : -1` |
| 0x00607720 | `chunk_read_dword_and_leave` | reads one dword when ≥4 bytes remain, then performs `chunk_leave`'s exact pop including the write-mode back-patch, and returns the dword ² |
| 0x004be660 | `chunk_vector_allocate` | vtable slot 0 — `operator_new(n * 4)` (§6) |
| 0x00621680 | `std_allocator_deallocate` | vtable slot 1 — `operator_delete_wrapper(p)`; 1 dataref (that slot) + 1 direct callsite (§6) |

² Both callers are in `mission_open_bin` and feed the result straight to `mission_create_actor`,
so the dword is the entity **kind**.

A class's PC code is not one contiguous cluster:
* `chunk_read` is emitted with the `C_airplane` TU's tail — a shared helper, not a `C_airplane`
  member.
* `chunk_open_stack_insert_*`, `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining` and
  `chunk_read_string_and_leave` sit in the `C_PData` translation unit `[0x005a5fe0, 0x005a7070)`,
  emitted with the `effects.tbl` loader that uses them.
* `chunk_get_dta_handle` and `chunk_read_dword_and_leave` sit with `mission_open_bin`.
* `chunk_close_handle` sits alone in the `C_recorder` TU (§5).

## 4. The open-chunk stack

The stack at `+0x04..+0x0c` is a `std::vector<int>` grown through `chunk_open_stack_insert_at` /
`chunk_open_stack_insert_n`, which index `C_chunk+0x00..+0x0c` and allocate through the class's own
2-slot vtable `0x0063c828` rather than `operator_new`.

| mode | what an entry holds | measured in |
|---|---|---|
| read | the chunk's **end offset**: `tell() - 6 + size` after reading the `{u16 id, u32 size}` header | `chunk_enter_next` pushes it; `chunk_leave` seeks to it; `chunk_bytes_remaining` subtracts `tell()` from it |
| write | the chunk's **start offset** | `chunk_leave` / `chunk_close` write `tell() - entry` at `entry + 2` |

So the record is `{u16 id, u32 size}` with the payload following, and `size` counts the 6-byte
header.

## 5. 🔑 `C_chunk::Close` has TWO PC emissions

`chunk_close` (`0x004be720`) and `chunk_close_handle` (`0x00544590`) carry the same body: unwind the
open-chunk stack at `this+0x04`/`+0x08`, back-patching each chunk's 4-byte length at
`chunk_start + 2` through `dtaSeek`/`dtaWrite`, then `dtaClose` and invalidate the handle.

They differ in exactly one thing: `chunk_close` also releases the stack vector through its
allocator slot and returns `void`; `chunk_close_handle` stops after clearing the is-open byte and
returns `1`.

The second copy sits alone inside the `C_recorder` TU, 500 KB from the rest of the class — the
shape of a per-TU inline instantiation (§9).

## 6. The allocator vtable pair and the folded vector ctor

`C_chunk`'s 2-slot MSVC allocator vtable at `0x0063c828` holds a class-qualified slot 0 and a
generic slot 1, because their reference sets differ:

| slot | symbol | callsites | datarefs | verdict |
|---|---|---|---|---|
| 0 | `chunk_vector_allocate` (`0x004be660`) | **0** | **1** | the vtable is the only thing that can reach it — single ownership, class-qualified name |
| 1 | `std_allocator_deallocate` (`0x00621680`) | **1**, in `glass_shatter_init` | 1 | a FOREIGN direct caller releasing a `std::vector<int>` unrelated to `C_chunk`: one COMDAT, two users, generic name |

`glass_shatter_init` (`structs/glass_shatter.md`) is the only direct caller of slot 1. It also uses
`folded_vector_ctor_at_0x04_0044a120`, a folded vector constructor with **three** owners:

* Its 22-byte body occurs twice in `.text` (a byte-pattern search of `.text` for the whole body;
  the body contains no relative branch, so the count is meaningful); the other copy is `folded_vector_ctor_at_0x04` at
  `0x0061e010`.
* It is handed to `crt_ehvec_ctor` with stride **`0x2c`** by `check_seg_cluster_ctor` and with
  stride **`0x18`** by `glass_shatter_init` — two element strides, two element types.
* It is also called directly by `E8` from `program_process_command` on a single `0x18`-byte
  `operator_new` object — a third construction site with no stride, found only by a scan of the
  whole image for references to the address.

The fold does not affect `structs/check_seg.md`'s *"a PC `check_seg` is `0x2c` bytes"*: that rests
on `check_seg_cluster_ctor`'s own literal, not on the callee.

## 7. PC vs PS2

The PS2 build has **6 `C_chunk` symbols**.

| PS2 method | PS2 VA | size | PC twin | status |
|---|---|---|---|---|
| `OpenFile__7C_chunkFPCcb` | 0x0011f7b0 | 184 | `chunk_open_file` | **CONFIRMED** |
| `Read__7C_chunkFPvi` | 0x003861d0 | 216 | `chunk_read` | **CANDIDATE** — same role, same arity; not body-diffed |
| `Close__7C_chunkFv` | 0x0011f3a0 | 292 | `chunk_close` + `chunk_close_handle` | **CONFIRMED** — body-diffed ¹ |
| `Descend__7C_chunkFv` | 0x0011f4d0 | 228 | `chunk_leave` (`0x005a61b0`) | **CONFIRMED** — the verb is inverted: it ASCENDS ² |
| `RAscend__7C_chunkFRUs` | 0x00482e00 | 168 | `chunk_enter_next` (`0x005a5fe0`) | **CONFIRMED** — the verb is inverted: it DESCENDS ² |
| `Size__7C_chunkCFv` | 0x00482230 | 128 | `chunk_bytes_remaining` (`0x005a6250`) | **CONFIRMED** — returns what is LEFT, not the chunk size |
| - | - | - | `chunk_read_string_and_leave` (`0x005a6280`) | PC-only helper |
| - | - | - | `chunk_read_dword_and_leave` (`0x00607720`) | PC-side fusion of a read and `Descend`; matches neither |
| - | - | - | `chunk_ctor` | no PS2 symbol — inlined there |
| - | - | - | `chunk_vector_allocate` / `std_allocator_deallocate` | no PS2 symbol; the 2-slot allocator vtable is an MSVC STL artifact |

¹ The distinctive part is the write-path back-patch: read the current position, compute
`pos - chunk_start`, `dtaSeek` to `chunk_start + 2`, `dtaWrite` the 4-byte length, seek back. Two PC
emissions (§5).
² `RAscend` reads a SIX-byte header and **pushes** a level; `Descend` **pops** one and reads
nothing. The PC names follow the behaviour.

🔑 **The two builds use different stacks.** The PC's open-chunk stack is the `std::vector` at
`+0x04..+0x0c` (§4). The PS2's is a **fixed 256-entry array at `+0x10` with its depth counter at
`+0x410`**, read out of `Descend__7C_chunkFv` and `RAscend__7C_chunkFRUs`. Do not rebase a PS2
`C_chunk` offset onto the PC.

**`folded_vector_ctor_at_0x04_0044a120` carries no PS2 twin.** The PS2 `__ct__9check_segFv` builds
its vector at `this+0x10` through a Metrowerks `compressed_pair_imp`, and the PC symbol is shared by
two element types, so no single PS2 constructor is its twin (the fold note lives in
`structs/check_seg.md`).

## 8. Related

`structs/glass_shatter.md` (the only direct caller of `std_allocator_deallocate`) ·
`structs/check_seg.md` (the `0x2c`-stride owner of the folded ctor) ·
`structs/pedestrian.md` (the `taxi_pas.dat` consumer) · `structs/recorder.md` (the TU holding
`chunk_close_handle`) · `ghidra/Game-ghidra.exe.functions.md` (the function registry).

## 9. Open items

| item | state |
|---|---|
| `chunk_read` ↔ `Read__7C_chunkFPvi` | role and position agree; bodies not diffed |
| why MSVC emitted the close body twice | the lone copy in the `C_recorder` TU has the shape of a per-TU inline instantiation; not measured |
| the chunk id vocabulary | the header grammar is known (§4); no chunk-id table exists in `def-files-docs/` |
| `system\taxi_pas.dat` | read through a stack `C_chunk` by `pedestrian_wave_at_taxi`; format undocumented |
