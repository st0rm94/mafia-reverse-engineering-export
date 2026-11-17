# `C_chunk` — the `.dta` archive chunk reader/writer

**Created** 2026-08-14 23:01 +0200 · **Modified** 2026-11-16 16:16 +0100

`C_chunk` is the small stack object the engine builds over an open `.dta` archive file to walk its
**chunk tree**: a header of `{id:u16, size:u32}` records that nest, so reading a file means
descending into a chunk, consuming its payload, and ascending back out. It is always constructed
as a **local**, never heap-allocated: all three `chunk_ctor` callsites pass a stack address and are
followed by `chunk_open_file`. Its file I/O goes through `rw_data.dll` (`_dtaOpen@8`,
`_dtaRead@12`, `_dtaSeek@12`, `_dtaWrite@12`, …), which is why the class is thin: it is the chunk
grammar on top of an archive API that lives outside `Game.exe`.

| | |
|---|---|
| PC prefix | `chunk_` |
| PC kind (`+0x10`) | - |
| PC vtable | - (`g_chunk_vt` @ `0x0063c828` at `+0x00` is the open-chunk stack's 2-slot allocator interface, §6; no `C_chunk` method is virtual) |
| PC base class | - |
| PC instance size | `0x20` (the highest field `+0x1c`; the object only exists on the stack) |
| PC constructor | `chunk_ctor` @ `0x004be680` |
| PC translation unit | `[0x004be660, 0x004be810)`: `chunk_vector_allocate`, `chunk_ctor`, `chunk_open_file`, `chunk_dtor`, after the last `pedestrian_*` function and up to the guard `crt_static_init_guard_004be810` that opens `C_PhThing`. The unit has no CRT guard of its own; the other `C_chunk` functions are emitted with the units that use them (§3) |
| PS2 class | `C_chunk` (tag `7C_chunk`), 6 methods: 6 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_chunk {                    // 0x20 bytes; PC offsets
    /* +0x00 */ void    *allocator_vt;      // [H] g_chunk_vt: the stack's allocator interface (§6)
    /* +0x04 */ int     *stack_begin;       // [H] open-chunk stack, std::vector<int> (§4)
    /* +0x08 */ int     *stack_end;         // [H] one entry per chunk currently descended into
    /* +0x0c */ int     *stack_cap;         // [H] end of the stack vector's storage
    /* +0x10 */ uint8_t  is_open;           // [H] 1 between chunk_open_file and the close
    /* +0x14 */ int      dta_handle;        // [H] the rw_data.dll handle, or -1
    /* +0x18 */ uint8_t  write_mode;        // [H] 0 = reading (dtaOpen), 1 = writing (dtaOpenWrite)
    /* +0x1c */ int      file_size;         // [H] dtaSeek(h, 0, SEEK_END) when reading, else -1; no reader (class census)
} C_chunk;
```

`+0x00..+0x0c` is an MSVC `std::vector<int>` whose allocator carries a vtable: `chunk_ctor` writes
`g_chunk_vt` and copies one byte of the optional allocator argument into the low byte of `+0x04`;
every constructing scope then zeroes `+0x04..+0x0c` itself. The stack grows on demand: an empty
stack means the reader is at the top level. `write_mode` flips the whole ascend / close path into
back-patching chunk sizes.

## 2. Field evidence

Measured by a census of the decomp's `this+N` accesses inside the `chunk_*` functions and of every
operand with each displacement in their bytes. The three `chunk_open_stack_*` helpers receive the
`C_chunk` itself as their vector.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | allocator_vt | H | `chunk_ctor` | `chunk_dtor`, `chunk_open_stack_free` (slot 1), `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n` (slots 0 and 1) | `g_chunk_vt`, two slots (§6) |
| +0x04 | stack_begin | H | `chunk_ctor` (its low byte, the allocator byte), `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `chunk_enter_next` (on a reallocation), `chunk_open_stack_free` (= 0), `chunk_dtor` (= 0) | `chunk_read`, `chunk_close`, `chunk_dtor`, `chunk_enter_next`, `chunk_read_string_and_leave`, `chunk_read_dword_and_leave`, `chunk_open_stack_free`, `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `cache_base_block_open` (open-coded `chunk_read`) | the constructing scopes zero it after `chunk_ctor` |
| +0x08 | stack_end | H | `chunk_enter_next` (push), `chunk_leave`, `chunk_close`, `chunk_read_string_and_leave`, `chunk_read_dword_and_leave` (pop), `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `chunk_open_stack_free`, `chunk_dtor` | `chunk_read`, `chunk_close`, `chunk_dtor`, `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining`, `chunk_read_string_and_leave`, `chunk_read_dword_and_leave`, `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `cache_base_block_open` (open-coded `chunk_read` and `chunk_enter_next`) | `end - begin >> 2` is the depth; `*(end - 1)` is the current chunk's recorded offset (§4) |
| +0x0c | stack_cap | H | `chunk_enter_next` (on a reallocation), `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `chunk_open_stack_free` (= 0), `chunk_dtor` (= 0) | `chunk_enter_next`, `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n`, `chunk_open_stack_free`, `chunk_dtor` | the free passes `cap - begin >> 2` to allocator slot 1 |
| +0x10 | is_open | H | `chunk_open_file` (= 1), `chunk_close` (= 0), `chunk_dtor` (= 0) | `chunk_open_file`, `chunk_read`, `chunk_close`, `chunk_dtor`, `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining`, `chunk_read_string_and_leave`, `chunk_get_dta_handle`, `chunk_read_dword_and_leave`, `cache_base_block_open` (open-coded) | `chunk_open_file` refuses to re-open an open chunk; `chunk_read` returns false when closed |
| +0x14 | dta_handle | H | `chunk_open_file`, `chunk_close` (= -1), `chunk_dtor` (= -1) | `chunk_open_file`, `chunk_read`, `chunk_close`, `chunk_dtor`, `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining`, `chunk_read_string_and_leave`, `chunk_get_dta_handle`, `chunk_read_dword_and_leave`, `cache_base_block_open` (open-coded) | passed straight to the `rw_data.dll` entry points |
| +0x18 | write_mode | H | `chunk_open_file` | `chunk_read`, `chunk_close`, `chunk_dtor`, `chunk_leave`, `chunk_read_string_and_leave`, `chunk_read_dword_and_leave`, `cache_base_block_open` (open-coded `chunk_read`) | `chunk_read` refuses when set; on ascend or close a write-mode chunk **back-patches its own size** ¹ |
| +0x1c | file_size | H | `chunk_open_file` | - | `dtaSeek(h, 0, SEEK_END)` on the read path; `-1` on the write path and when the open fails. No reader in the `chunk_*` bodies nor in the open-coded reads of `cache_base_block_open` (class census; Open item 1) |

¹ `dtaSeek(h, 0, CUR)`, subtract the recorded start, `dtaSeek(h, start + 2, SET)`,
`dtaWrite(&size, 4)`, seek back.

## 3. Method family

Coverage: all **6** PS2 `7C_chunk` methods are in the tables below, **6 CONFIRMED twins**. The
constructor, the destructor, the read-and-leave helpers, the handle accessor and the vector
helpers have no PS2 symbol (§7).

The class's PC code is not one contiguous cluster. Outside its own range:
* `chunk_read` is emitted at the head of the `C_bottle` unit, between its guard
  `crt_static_init_guard_00453a20` and `bottle_ctor`: a shared helper, not a `C_bottle` member.
* `chunk_close` and `chunk_open_stack_free` sit in the `C_recorder` unit (opened by
  `crt_static_init_guard_005407f0`), the second among its STL helpers (§5).
* `chunk_open_stack_insert_at` and `chunk_open_stack_insert_n` close the unit before
  `crt_static_init_guard_005a5fb0`; `chunk_enter_next`, `chunk_leave`, `chunk_bytes_remaining` and
  `chunk_read_string_and_leave` open the `C_PData` unit `[0x005a5fb0, 0x005a7070)`, emitted with
  the `effects.tbl` loader that uses them.
* `chunk_get_dta_handle` and `chunk_read_dword_and_leave` sit with `mission_open_bin`.
* `std_allocator_deallocate` sits among the shared `std_vector*` helpers.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004be680 | `chunk_ctor` | instance | - | - | - | installs `g_chunk_vt` and copies the allocator byte; three callsites, each on a stack local. No PS2 symbol: inlined there |
| 0x004be720 | `chunk_dtor` | instance | - | - | - | the `chunk_close` sequence inline, then frees the stack vector (§5). No PS2 symbol: the PS2 stack is an array inside the object, so there is nothing to free |

### 3b. Accessors

The class has no accessor of its own: `chunk_get_dta_handle` tests `is_open` before it reads the
handle (§3c).

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00453a50 | `chunk_read` | `Read__7C_chunkFPvi` | 0x003861d0 | CONFIRMED | a bounded read: refuses when closed or in write mode, and refuses a request larger than what remains of the enclosing chunk (the `max(0, remaining) < size` idiom, `structs/bottle.md` §9c) |
| 0x004be6b0 | `chunk_open_file` | `OpenFile__7C_chunkFPCcb` | 0x0011f7b0 | CONFIRMED | `dtaOpen` / `dtaOpenWrite`, records the handle, seeks to the end for the size on the read path, rewinds, sets `is_open` |
| 0x00544590 | `chunk_close` | `Close__7C_chunkFv` | 0x0011f3a0 | CONFIRMED | in write mode unwinds the open-chunk stack, back-patching each chunk's size; then `dtaClose`, handle `-1`, `is_open` cleared; returns 1, or 0 when not open (§5) ¹ |
| 0x005a5fe0 | `chunk_enter_next` | `RAscend__7C_chunkFRUs` | 0x00482e00 | CONFIRMED | reads the 6-byte header, pushes the chunk's end offset, returns the id (§4) ² |
| 0x005a61b0 | `chunk_leave` | `Descend__7C_chunkFv` | 0x0011f4d0 | CONFIRMED | ascends: the write-mode back-patch, a seek to the recorded offset, a pop. **83 callsites**, 53 in `mission_open_bin`: the most-used `C_chunk` entry point ² |
| 0x005a6250 | `chunk_bytes_remaining` | `Size__7C_chunkCFv` | 0x00482230 | CONFIRMED | `current_chunk_end - tell()`, 0 on a closed chunk, an empty stack or a failed seek; 18 callsites, each the condition of a `while (remaining) { enter; …; leave; }` loop. It returns what is LEFT, not the chunk size |
| 0x005a6280 | `chunk_read_string_and_leave` | - | - | - | reads `min(remaining, 999)` bytes as a C string into a `std::string`, then ascends; caps every name in `effects.tbl` and `cache.bin` |
| 0x00607710 | `chunk_get_dta_handle` | - | - | - | `return is_open ? dta_handle : -1` |
| 0x00607720 | `chunk_read_dword_and_leave` | - | - | - | reads one dword when at least 4 bytes remain, then performs `chunk_leave`'s pop including the write-mode back-patch, and returns the dword ³ |

¹ The same open test with its 0 return, the same write-path back-patch, `dtaClose` and the 1
return on both builds: statement-identical.
² The PS2 verbs are inverted: `RAscend` reads a SIX-byte header and **pushes** a level, `Descend`
**pops** one and reads nothing. The PC names follow the behaviour.
³ Both callers are in `mission_open_bin` and feed the result straight to `mission_create_actor`, so
the dword is the entity **kind**. A PC fusion of a read and `Descend`; it matches no PS2 method.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004be660 | `chunk_vector_allocate` | - | - | - | `g_chunk_vt` slot 0: `operator_new(n * 4)`; no callsite, one dataref (the vtable) (§6) |
| 0x00546810 | `chunk_open_stack_free` | - | - | - | the stack vector's free: allocator slot 1, then zeroes `+0x04..+0x0c`; 6 calls, each right after `chunk_close` on the same stack `C_chunk`, and 9 unwind funclets (§5) |
| 0x005a5c10 | `chunk_open_stack_insert_at` | - | - | - | the stack vector's insert; 18 callsites |
| 0x005a5db0 | `chunk_open_stack_insert_n` | - | - | - | the stack vector's fill-insert; 1 callsite |
| 0x00621680 | `std_allocator_deallocate` | - | - | - | `g_chunk_vt` slot 1: `operator_delete_wrapper(p)`; one dataref (that slot) and one direct callsite in `glass_shatter_init`, which releases a vector unrelated to `C_chunk` (§6; also listed in `structs/glass_shatter.md`) |

The three `chunk_open_stack_*` helpers are methods of the stack's `std::vector<int>`; their receiver
is the `C_chunk`, whose first member is that vector.

## 4. The open-chunk stack

The stack at `+0x04..+0x0c` is a `std::vector<int>` grown through `chunk_open_stack_insert_at` /
`chunk_open_stack_insert_n`, which index `C_chunk+0x00..+0x0c` and allocate through the class's own
2-slot vtable `0x0063c828` rather than `operator_new`, and freed by `chunk_open_stack_free`.

| mode | what an entry holds | measured in |
|---|---|---|
| read | the chunk's **end offset**: `tell() - 6 + size` after reading the `{u16 id, u32 size}` header | `chunk_enter_next` pushes it; `chunk_leave` seeks to it; `chunk_bytes_remaining` subtracts `tell()` from it |
| write | the chunk's **start offset** | `chunk_leave` / `chunk_close` write `tell() - entry` at `entry + 2` |

So the record is `{u16 id, u32 size}` with the payload following, and `size` counts the 6-byte
header.

### 4a. The files read through a `C_chunk`, and where their ids are decoded

`C_chunk` itself never interprets an id. Every file opened through `chunk_open_file` (a closed set
of eleven `E8` calls) is listed here with the document that decodes its ids:

| File | Opened by | Ids walked by | Decoded in |
|---|---|---|---|
| `missions\<mission>\scene2.bin` | `mission_open_bin` | `mission_open_bin` | `structs/mission.md` §6a |
| `missions\<mission>\check.bin` | `mission_open` | `web_path_load_nodes` | `structs/web_path.md` §4 |
| `missions\<mission>\road.bin` | `mission_open` | `roads_load_data` | `structs/roads.md` |
| `missions\<mission>\cache.bin` | `mission_open` | `cache_base_block_open` | `structs/cache_base_block.md` §4 |
| `tables\effects.tbl` | `particle_manager_load_data` | `particle_manager_load_data`, `particle_data_type_load_data` | `structs/particle.md` §6a |
| `missions\<mission>\effects.bin` | `particle_manager_load_static_particles` | `particle_manager_load_static_particles` | `structs/particle.md` §6b |
| `*.tck` recorder tracks | `recorder_load_tck` | `recorder_load_tck` | `structs/recorder.md` |
| `anims\<name>` | `anim_machine_load_track` | `anim_machine_load_track` | - |
| `diff\*.chg` | `game_load_differences` | `game_load_differences` | §4b |
| `system\taxi_pas.dat` | `pedestrian_wave_at_taxi` | `program_load_source_code` | §4c |

### 4b. The `.chg` scene-difference files

`game_load_differences` (script command `loaddifferences`) reads one `.chg` file from `Mafia\diff`;
92 ship. The root chunk must be **1234** (`0x4d2`); its children are:

| Id | Shipped | What `game_load_differences` does with it |
|---|---:|---|
| 100 | 4 148 | adds a frame: creates it through the I3D driver, names it, places it under its parent, and loads a model (type 9) or sound (type 4) through the model or sound cache |
| 200 | 1 178 | adds an actor: the record opens with the kind dword (a 2 becomes `0x1b`), then the actor's own data; `mission_create_actor` of that kind, `Init` on the new frame, `LoadData`, `mission_add_actor`, and records it in the removed-actor difference vector. Shipped kinds: 1 (530), `0x1b` (463, the cutscene characters), 4 (98), `0x21` (77), 9 (7), 5 (3) |
| 300 | 1 | no arm: skipped |
| 400 | 190 | binds an actor to an existing frame: the record is a `u32` name length, the frame name and the kind dword; looks the frame up by name and, when the frame has no actor yet, creates one of that kind. Shipped kinds: 1 (143), `0x21` (45), 2 (1), 6 (1) |
| 500 | 89 | scripts: one `C_program` per entry (`program_ctor`) |

### 4c. `system\taxi_pas.dat` — a script template, not a chunk tree

The file has no chunk headers. `pedestrian_wave_at_taxi` opens it through a stack `C_chunk` and
hands that to `program_load_source_code`, which reads a serialised `C_program`: a `u8`, a `u32`
length and that many bytes of `source_a`, then a `u32` length and that many bytes of `source_b`.
The shipped file is `01`, length 0, length `0xc65`, then 3 173 bytes of script text: exactly its
3 182 bytes. The text is the **taxi passenger's script**, a template whose `%s` / `%i` placeholders
`pedestrian_wave_at_taxi` fills with `crt_sprintf` (the two frame names, three `flt` values and
the `timeron` arguments) before `program_set_source_code`. Scene chunk `0xAE51` carries the same
serialised form (`structs/mission.md` §6a).

## 5. `chunk_close` and `chunk_dtor` — the close, and the destructor that repeats it

Both bodies unwind the open-chunk stack at `this+0x04` / `+0x08`, back-patching each chunk's 4-byte
length at `chunk_start + 2` through `dtaSeek` / `dtaWrite`, then `dtaClose` and invalidate the
handle. They are two functions, not one body emitted twice:

| | `chunk_close` (`0x00544590`) | `chunk_dtor` (`0x004be720`) |
|---|---|---|
| role | `C_chunk::Close` | the destructor |
| after the close | returns 1 (0 when not open) | frees the stack vector through allocator slot 1, returns `void` |
| references | 13 direct `E8` calls across the binary | 7 direct `E8` calls plus 8 `E9` jumps from MSVC unwind funclets of the form `lea ecx,[ebp-N]; jmp chunk_dtor` |

The unwind funclets are what identify the destructor: they run when an exception leaves a scope
that holds a stack `C_chunk`, and a funclet only ever calls destructors. Callers that close the
file early call `chunk_close` and let `chunk_dtor` run at scope exit, where it finds the chunk
closed and only frees the vector. Six other scopes end with `chunk_close` followed by
`chunk_open_stack_free`, the vector-freeing tail of the destructor called out of line (`chunk_dtor`
ends with the same four statements inline); their nine unwind funclets jump to
`chunk_open_stack_free`. `chunk_close` sits in the `C_recorder` unit, 500 KB from the rest of the
class.

## 6. The allocator vtable pair and the folded vector ctor

`C_chunk`'s 2-slot MSVC allocator vtable at `0x0063c828` holds a class-qualified slot 0 and a
generic slot 1, because their reference sets differ:

| slot | symbol | callsites | datarefs | verdict |
|---|---|---|---|---|
| 0 | `chunk_vector_allocate` (`0x004be660`) | **0** | **1** | the vtable is the only thing that can reach it: single ownership, class-qualified name |
| 1 | `std_allocator_deallocate` (`0x00621680`) | **1**, in `glass_shatter_init` | 1 | a FOREIGN direct caller releasing a `std::vector<int>` unrelated to `C_chunk`: one COMDAT, two users, generic name |

`glass_shatter_init` (`structs/glass_shatter.md`) is the only direct caller of slot 1. It also uses
`folded_vector_ctor_at_0x04_0044a120`, a folded vector constructor with **three** owners:

* Its 22-byte body occurs twice in `.text` (a byte-pattern search of `.text` for the whole body;
  the body contains no relative branch, so the count is meaningful); the other copy is
  `folded_vector_ctor_at_0x04` at `0x0061e010`.
* It is handed to `crt_ehvec_ctor` with stride **`0x2c`** by `check_seg_cluster_ctor` and with
  stride **`0x18`** by `glass_shatter_init`: two element strides, two element types.
* It is also called directly by `E8` from `program_process_command` on a single `0x18`-byte
  `operator_new` object: a third construction site with no stride, found only by a scan of the
  whole image for references to the address.

The fold does not affect `structs/check_seg.md`'s *"a PC `check_seg` is `0x2c` bytes"*: that rests
on `check_seg_cluster_ctor`'s own literal, not on the callee.

## 7. PC vs PS2

The PS2 build has **6 `C_chunk` symbols**, all six twins of §3c. The PC-side functions with no PS2
symbol:

| PC function | Why there is no PS2 symbol |
|---|---|
| `chunk_ctor` | inlined on the PS2 |
| `chunk_dtor`, `chunk_open_stack_free` | the PS2 open-chunk stack is an array inside the object, so there is nothing to free |
| `chunk_open_stack_insert_at`, `chunk_open_stack_insert_n` | the same: the PS2 pushes into its array inline |
| `chunk_vector_allocate`, `std_allocator_deallocate` | the 2-slot allocator vtable is an MSVC STL artifact |
| `chunk_read_string_and_leave` | a PC-only helper |
| `chunk_read_dword_and_leave` | a PC fusion of a read and `Descend`; matches neither |
| `chunk_get_dta_handle` | no PS2 symbol |

**The two builds use different stacks.** The PC's open-chunk stack is the `std::vector` at
`+0x04..+0x0c` (§4). The PS2's is a **constant-size 256-entry array at `+0x10` with its depth counter at
`+0x410`**, read out of `Descend__7C_chunkFv` and `RAscend__7C_chunkFRUs`. Do not rebase a PS2
`C_chunk` offset onto the PC.

**`folded_vector_ctor_at_0x04_0044a120` carries no PS2 twin.** The PS2 `__ct__9check_segFv` builds
its vector at `this+0x10` through a Metrowerks `compressed_pair_imp`, and the PC symbol is shared by
two element types, so no single PS2 constructor is its twin (the fold note lives in
`structs/check_seg.md`).

## 8. Related

- `structs/glass_shatter.md` — the only direct caller of `std_allocator_deallocate`.
- `structs/check_seg.md` — the `0x2c`-stride owner of the folded ctor.
- `structs/pedestrian.md` — the `taxi_pas.dat` consumer.
- `structs/recorder.md` — the unit holding `chunk_close`.
- `structs/mission.md`, `structs/particle.md`, `structs/cache_base_block.md`,
  `structs/web_path.md`, `structs/roads.md` — the formats of §4a.
- `ghidra/Game-ghidra.exe.functions.md` — the function registry.

## 9. Open items

1. **`file_size` (`+0x1c`) has no reader** in the `chunk_*` bodies nor in the open-coded reads of
   `cache_base_block_open`. The five other loaders that push the open-chunk stack themselves (the
   callers of `chunk_open_stack_insert_at` / `chunk_open_stack_insert_n`: `mission_open_bin`,
   `game_load_differences`, `particle_manager_load_data`, `particle_data_type_load_data`,
   `particle_manager_load_static_particles`) were not read for it, nor for the cells of
   `+0x04..+0x18`.
