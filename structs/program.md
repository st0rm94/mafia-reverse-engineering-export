# `C_program` — the per-entity mission-script VM

**Created** 2026-08-02 19:48 +0200 · **Modified** 2026-11-16 21:19 +0100

`C_program` is **the script virtual machine every scripted actor carries**. It compiles a script's
source text, which `program_load_source_code` reads from a `C_chunk` (for a placed actor, its record
in `scene2.bin`), decodes it into instructions, executes one command per call, holds the
actor / float / frame register files, and serialises the whole thing into the savegame. It is the
class the project's script-mapping work reads.

```
  C_entity  +0xa9c            (also C_detector +0x70)
      |  DERIVES FROM  (polymorphic base sub-object - §6a)
      v
  C_program                source text + instruction stream + three register files
      |  executes
      v
  one script command per program_process_command() call        §7
```

| | |
|---|---|
| PC prefix | `program_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_program_vt` @ `0x0063eab0` standalone; `0x0063b45c` as the base of `C_entity` (§6) |
| PC base class | -: itself a polymorphic base of `C_entity`, and embedded in `C_detector` |
| PC instance size | `0x68`, embedded at `C_entity+0xa9c` and `C_detector+0x70` |
| PC constructor | `program_ctor` @ `0x005af2f0` |
| PC translation unit | `[0x005af290, 0x005c9720)`, guard to guard: two emission blocks and the inline tail (§4) |
| PS2 class | `C_program`, 42 methods, 36 mapped |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

> ## 🔑 THE ADJUSTOR-THUNK QUARTET THAT IDENTIFIES THE CLASS
> Four six-byte functions at `0x0043b030/b040/b050/b060` all begin `81 e9 9c 0a 00 00`
> (**`sub ecx, 0xa9c`**) and tail-jump. Each carries exactly ONE dataref, into consecutive
> slots of the vtable at `.rdata 0x0063b45c`.
>
> That is the standard C++ ABI construct for a virtual call arriving through a **secondary
> base**: the caller holds a `C_program *` (which is `C_entity * + 0xa9c`), and the C_entity
> override it must reach was compiled expecting the `C_entity` `this`. The thunk fixes the
> pointer and jumps.
>
> ⇒ **`C_program` is a polymorphic BASE of `C_entity`, embedded at `+0xa9c`.**

## 1. Base field layout

Every offset below comes from a body read on both builds. The vectors store begin / end / capacity, so no array has a set
count: the three register files are sized by `dim_flt` / `dim_frm` / `dim_act` (§5).

```c
typedef struct C_program {                 // embedded at C_entity+0xa9c, C_detector+0x70
    /* +0x00 */ void   **vtable;           // [H] g_program_vt standalone / 0x0063b45c as a base
    /* +0x04 */ char    *program_name;     // [M] the program's name: the first source string, only saved and reloaded (§2)
    /* +0x08 */ char    *source_text;      // [H] the text SetSourceCode writes and Init compiles (§2)
    /* +0x0c */ char     instr_alloc;      // [H] allocator byte; its address is the vector's receiver
    /* +0x10 */ S_instruction *instr_begin;// [H] instruction vector, stride 8 = {opcode, payload*}
    /* +0x14 */ S_instruction *instr_end;  // [H] the same: end
    /* +0x18 */ S_instruction *instr_cap;  // [H] the same: capacity
    /* +0x1c */ int      pc;               // [H] program counter = an index into the vector
    /* +0x20 */ int      prev_pc;          // [H] write-only: NextCommand saves the pc here; Init writes -1
    /* +0x24 */ int      instr_elapsed_ms; // [H] ms accumulated on the current instruction (§5)
    /* +0x28 */ int      susp_elapsed_ms;  // [H] SetEvent snapshots +0x24 here at suspend
    /* +0x2c */ int      susp_game_time_ms;// [H] SetEvent snapshots the game clock here at suspend (§5)
    /* +0x30 */ char     retstack_alloc;   // [H] allocator byte; its address is the stack's receiver
    /* +0x34 */ uint16_t *retstack_begin;  // [H] the return stack: 2-byte packed frames (§5)
    /* +0x38 */ uint16_t *retstack_end;    // [H] the same: end
    /* +0x3c */ uint16_t *retstack_cap;    // [H] the same: capacity
    /* +0x40 */ void    *ctx_head;         // [H] saved-context list of 0x18-byte frames (§5a)
    /* +0x44 */ C_actor *act_state_owner;  // [H] the script's own actor (§2)
    /* +0x48 */ float   *float_regs;       // [H] the script `flt[]` file
    /* +0x4c */ int      float_reg_count;  // [H] entries in flt[]
    /* +0x50 */ I3D_frame **frame_regs;    // [H] the script `frm[]` file; typed by PS2 SetFrameReg
    /* +0x54 */ int      frame_reg_count;  // [H] entries in frm[]
    /* +0x58 */ C_actor **actor_regs;      // [H] the script `act[]` file; typed by PS2 SetActorReg
    /* +0x5c */ int      actor_reg_count;  // [H] entries in act[]
    /* +0x60 */ float    ai_priority;      // [H] `setaipriority xx` stores (100 - xx) * 0.01 (§2)
    /* +0x64 */ char     running;          // [H] keep executing instructions in the same frame (§5a)
    /* +0x65 */ char     stop_on_return;   // [H] when set, RETURN clears both it and `running` (§2)
    /* +0x66 */ char     in_compact_block; // [H] LeaveCompactBlock's gate
    /* +0x67 */ char     compact_dir;      // [H] 1 = force backwards, 2 = force forwards
} C_program;
```

## 2. Field evidence

Cells of the two allocator bytes and of the return stack read the address too: the vector helpers take it as their receiver.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vtable | H | `program_ctor`, `program_dtor` | `program_call_process`, `program_compile`, `program_process_command`, `program_set_event` | read at the slot calls |
| +0x4 | program_name | M | `program_ctor`, `program_load_source_code` | `program_dtor`, `program_load_source_code`, `program_save_source_code` | the first of the two source strings in the `C_chunk`; a scan of `.text` at `+0x04`, `C_entity+0xaa0` and `C_detector+0x74` finds no other access, so no PC code uses it as a name. The name comes from the PS2 build (§10e); a PC reader that compares it would make it [H] |
| +0x8 | source_text | H | `program_ctor`, `program_duplicate_from`, `program_load_source_code`, `program_set_source_code` | `program_dtor`, `program_duplicate_from`, `program_init`, `program_load_source_code`, `program_save_source_code`, `program_set_source_code` | `program_set_source_code` writes it, `program_init` compiles it, `program_duplicate_from` copies it |
| +0xc | instr_alloc | H | `program_ctor` | `program_clear_after_end`, `program_compile`, `program_instructions_clear`, `program_process_command` | `program_instructions_clear` and `vec_index_addr_stride_0x8` take this address as `this` |
| +0x10 | instr_begin | H | `program_clear_after_end` (by address, `std_vector8_insert_at`), `program_compile`, `program_ctor`, `program_dtor`, `program_process_command` (by address, `std_vector8_push_back`) | `program_call_process`, `program_clear_after_end`, `program_compile`, `program_done`, `program_dtor`, `program_init`, `program_is_event_id`, `program_leave_compact_block`, `program_next_command`, `program_process_command` (by address, `vec_count_stride_0x8`, `vec_index_addr_stride_0x8`, `std_vector8_push_back`), `program_set_event` | - |
| +0x14 | instr_end | H | `program_clear_after_end`, `program_compile`, `program_ctor`, `program_done`, `program_dtor`, `program_init`, `program_instructions_clear`, `program_process_command` (by address, `std_vector8_push_back`) | `program_call_process`, `program_compile`, `program_done`, `program_init`, `program_is_event_id`, `program_leave_compact_block`, `program_next_command`, `program_process_command` (by address, `vec_count_stride_0x8`, `std_vector8_push_back`), `program_set_event` | - |
| +0x18 | instr_cap | H | `program_clear_after_end` (by address, `std_vector8_insert_at`), `program_compile`, `program_ctor`, `program_dtor`, `program_process_command` (by address, `std_vector8_push_back`) | `program_clear_after_end` (by address, `std_vector8_insert_at`), `program_compile`, `program_process_command` (by address, `std_vector8_push_back`) | tested before each append and rewritten on growth: the `call` arm's copy into the live vector, `program_compile`'s open-coded append, `program_clear_after_end`'s re-insert |
| +0x1c | pc | H | `program_call_subroutine`, `program_clear_after_end`, `program_init`, `program_leave_compact_block`, `program_next_command`, `program_process_command`, `program_save_game_load`, `program_set_event` | `program_call_process`, `program_leave_compact_block`, `program_next_command`, `program_process_command`, `program_save_game_save`, `program_set_event` | - |
| +0x20 | prev_pc | H | `program_init`, `program_next_command` | - | no reader in `.text`: a scan at `+0x20`, `C_entity+0xabc` and `C_detector+0x90` attributes none to a `C_program` |
| +0x24 | instr_elapsed_ms | H | `program_init`, `program_next_command`, `program_process_command`, `program_save_game_load`, `program_set_event` | `program_process_command`, `program_save_game_save`, `program_set_event` | `program_process_command` adds the frame delta on every entry, `WAIT` compares against it, `program_next_command` clears it on a pc move (§5) |
| +0x28 | susp_elapsed_ms | H | `program_save_game_load`, `program_set_event` | `program_process_command`, `program_save_game_save` | - |
| +0x2c | susp_game_time_ms | H | `program_save_game_load`, `program_set_event` | `program_process_command`, `program_save_game_save` | `RETURN` uses the `+0x28` / `+0x2c` pair to credit the elapsed time (§5) |
| +0x30 | retstack_alloc | H | `program_ctor` | `program_done`, `program_process_command`, `program_return_stack_back`, `program_return_stack_pop`, `program_return_stack_push`, `program_save_game_load`, `program_set_event` | the `program_return_stack_*` helpers take this address as `this` |
| +0x34 | retstack_begin | H | `program_ctor`, `program_dtor`, `program_return_stack_push` | `program_call_subroutine`, `program_done`, `program_dtor`, `program_init`, `program_process_command` (by address, `std_vector2_size`), `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_event` | one frame per `gosub` / raised event (§5) |
| +0x38 | retstack_end | H | `program_call_subroutine`, `program_ctor`, `program_dtor`, `program_init`, `program_return_stack_pop`, `program_return_stack_push`, `program_save_game_load` | `program_call_subroutine`, `program_done`, `program_init`, `program_process_command` (by address, `std_vector2_size`), `program_return_stack_back`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save` | - |
| +0x3c | retstack_cap | H | `program_ctor`, `program_dtor`, `program_return_stack_push`, `program_save_game_load` and `program_set_event` (by address, `std_vector2_insert_at`) | `program_return_stack_push`, `program_save_game_load` and `program_set_event` (by address, `std_vector2_insert_at`) | - |
| +0x40 | ctx_head | H | `program_clear_after_end`, `program_done`, `program_init`, `program_process_command` | `program_clear_after_end`, `program_done`, `program_process_command` | the frames are linked through their own `+0x14`; `call` pushes, `end` pops (§5a) |
| +0x44 | act_state_owner | H | `detector_game_init`, `entity_game_init`, `program_call_subroutine` | `program_clear_after_end`, `program_process_command`, `program_set_event` | `entity_game_init` and `detector_game_init` store the owner itself; `program_clear_after_end` calls `actor_set_act_state(2)` on it |
| +0x48 | float_regs | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_done`, `program_get_float_reg`, `program_init`, `program_process_command`, `program_save_game_load`, `program_save_game_save`, `program_set_float_reg` | - |
| +0x4c | float_reg_count | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_get_float_reg`, `program_process_command`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_float_reg` | - |
| +0x50 | frame_regs | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_done`, `program_get_frame_reg`, `program_init`, `program_process_command`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_frame_reg` | - |
| +0x54 | frame_reg_count | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_get_frame_reg`, `program_process_command`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_frame_reg` | - |
| +0x58 | actor_regs | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_done`, `program_find_actor_later`, `program_get_actor_reg`, `program_init`, `program_invalidate_actor_reg`, `program_process_command`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_actor_reg` | - |
| +0x5c | actor_reg_count | H | `program_ctor`, `program_done`, `program_init`, `program_process_command`, `program_save_game_load` | `program_find_actor_later`, `program_get_actor_reg`, `program_invalidate_actor_reg`, `program_process_command`, `program_save_game_get_size`, `program_save_game_load`, `program_save_game_save`, `program_set_actor_reg` | - |
| +0x60 | ai_priority | H | `entity_script_is_fake`, `program_ctor`, `program_init`, `program_save_game_load`, `program_set_ai_priority` | `entity_force_ai`, `entity_script_switch`, `program_save_game_save` | `program_ctor` and `program_init` seed 0.5, and `program_init` zeroes it when the program compiled to no instructions; the setter's only caller is the `setaipriority` arm; `entity_force_ai` bails at 0.75 or more; `entity_script_switch` compares it with `entity+0xf58` |
| +0x64 | running | H | `program_call_process`, `program_call_subroutine`, `program_init`, `program_process_command`, `program_save_game_load` | `program_call_process`, `program_save_game_save` | `commandblock` is a script-side write of it (§5a) |
| +0x65 | stop_on_return | H | `program_init`, `program_process_command`, `program_save_game_load` | `program_process_command`, `program_save_game_save` | when set, `RETURN` clears both it and `running`, so the next return halts the program; `program_init` clears it |
| +0x66 | in_compact_block | H | `program_init`, `program_process_command`, `program_save_game_load` | `program_leave_compact_block`, `program_save_game_save` | - |
| +0x67 | compact_dir | H | `program_init`, `program_process_command`, `program_save_game_load` | `program_leave_compact_block`, `program_save_game_save` | - |

**Owned by another object:** `+0x110` -> `structs/i3d_frame.md` (`frame_type`), `+0x124` ->
`structs/i3d_frame.md` (the first child frame, not in its typedef) and `+0x1d4` ->
`structs/i3d_frame.md` (`class_tag`), read by `program_switch_light_map`, whose ECX is an
`I3D_frame`: it walks the children from `+0x124` through each child's next sibling at `+0x12c`.

## 3. Method family

36 of the 42 PS2 methods mapped, every one body-diffed on both builds; the other 6 are in §10e. The alignment is forced and
gapless: block A holds 37 definitions; the PS2 main cluster holds 38 methods; removing `TryChangeProgramData` and
`DecompileCommand` leaves 36, and 36 + the MSVC-only scalar deleting destructor = 37. The size ratio of
`program_decode_instruction` is explained in §10c, and `program_process_command` is graded per command arm (§10h).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0043b030 | `program_base_thunk_entity_scalar_deleting_dtor` | `C_entity` base | - | - | - | slot 0: `sub ecx, 0xa9c` + jump to `entity_scalar_deleting_dtor` |
| 0x005af290 | `crt_static_init_guard_005af290` | the translation unit | - | - | - | the unit's first `.CRT$XCU` initializer |
| 0x005af2b0 | `crt_atexit_register_005af2b0` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x005af2c0 | `crt_static_init_guard_005af2c0` | the translation unit | - | - | - | the unit's second `.CRT$XCU` initializer |
| 0x005af2e0 | `crt_atexit_register_005af2e0` | the translation unit | - | - | - | registers the unit's static destructor |
| 0x005af2f0 | `program_ctor` | instance | `__ct__9C_programFv` | 0x001b6450 | CONFIRMED | construct a C_program script VM |
| 0x005af340 | `program_scalar_deleting_dtor` | instance | - | - | - | MSVC-only construct |
| 0x005af360 | `program_dtor` | instance | `__dt__9C_programFv` | 0x001b6500 | CONFIRMED | destroy a C_program script VM |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8760 | `program_set_ai_priority` | - | - | - | one store of `+0x60`; a header inline: the PS2 build inlines it into its `Process` arm |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043b040 | `program_base_thunk_entity_save_game_get_size` | - | - | - | slot 2: `sub ecx, 0xa9c` + jump to `entity_save_game_get_size` |
| 0x0043b050 | `program_base_thunk_entity_save_game_save` | - | - | - | slot 3: `sub ecx, 0xa9c` + jump to `entity_save_game_save` |
| 0x0043b060 | `program_base_thunk_entity_save_game_load` | - | - | - | slot 4: `sub ecx, 0xa9c` + jump to `entity_save_game_load` |
| 0x005af3c0 | `program_save_source_code` | `SaveSourceCode__9C_programFP7C_chunk` | 0x001b65d0 | CONFIRMED | slot 1: write both source strings into a C_chunk |
| 0x005af4b0 | `program_load_source_code` | `LoadSourceCode__9C_programFP7C_chunk` | 0x001b6740 | CONFIRMED | read both source strings back from a C_chunk |
| 0x005af730 | `program_set_source_code` | `SetSourceCode__9C_programFPc` | 0x001b6d70 | CONFIRMED | replace the program's source text |
| 0x005af780 | `program_init` | `Init__9C_programFv` | 0x001b6df0 | CONFIRMED | reset the VM and compile the current source text |
| 0x005af8a0 | `program_done` | `Done__9C_programFv` | 0x001b6f30 | CONFIRMED | tear the VM's runtime state down, keeping the source |
| 0x005af9d0 | `program_duplicate_from` | `DuplicateProgramFrom__9C_programFP9C_program` | 0x001b70e0 | CONFIRMED | copy another program's source text into this one |
| 0x005afa30 | `program_split_params` | `ExtrudeParams__9C_programFPcRiPPc` | 0x001b7170 | CONFIRMED | split one command's argument text on commas |
| 0x005afb00 | `program_decode_operand_float` | `DecodeOperand__9C_programFPcRiRf` | 0x001b7320 | CONFIRMED | parse one operand as a float register or a literal |
| 0x005afbb0 | `program_decode_operand_int` | `DecodeOperandInt__9C_programFPcRiRi` | 0x001b7460 | CONFIRMED | parse one operand as a float register or an int literal |
| 0x005afc60 | `program_extract_quoted_string` | `ExtrudeString__9C_programFPc` | 0x001b7590 | CONFIRMED | unwrap a double-quoted script string literal |
| 0x005afca0 | `program_decode_let` | `DecodeLet__9C_programFRQ29C_program13S_instructionPc` | 0x001b7630 | CONFIRMED | compile a LET assignment into an S_instruction |
| 0x005afdd0 | `program_decode_if` | `DecodeIf__9C_programFRQ29C_program13S_instructionPc` | 0x001b7840 | CONFIRMED | compile an IF comparison into an S_instruction |
| 0x005afe70 | `program_decode_params` | `DecodeParams__9C_programFPcRQ29C_program13S_instruction` | 0x001b7970 | CONFIRMED | decode ONE script command's arguments into its S_instruction |
| 0x005b1b60 | `program_decode_instruction` | `DecodeInstruction__9C_programFPcRi` | 0x001b9e00 | CONFIRMED | map a script command NAME to its opcode |
| 0x005ba6a0 | `program_compile` | `Compile__9C_programFPcbPiPi` | 0x001b9f80 | CONFIRMED | compile the source text into the instruction stream |
| 0x005ba9a0 | `program_get_float_reg` | `GetFloatReg__9C_programFi` | 0x001ba250 | CONFIRMED | read a float register |
| 0x005ba9c0 | `program_get_frame_reg` | `GetFrameReg__9C_programFi` | 0x001ba290 | CONFIRMED | read a frame register |
| 0x005ba9e0 | `program_get_actor_reg` | `GetActorReg__9C_programFi` | 0x001ba2d0 | CONFIRMED | read an actor register |
| 0x005baa00 | `program_set_float_reg` | `SetFloatReg__9C_programFif` | 0x001ba310 | CONFIRMED | write a float register |
| 0x005baa20 | `program_set_frame_reg` | `SetFrameReg__9C_programFiP9I3D_frame` | 0x001ba340 | CONFIRMED | write a frame register |
| 0x005baa40 | `program_set_actor_reg` | `SetActorReg__9C_programFiP7C_actor` | 0x001ba370 | CONFIRMED | write an actor register |
| 0x005baa60 | `program_next_command` | `NextCommand__9C_programFPc` | 0x001ba3a0 | CONFIRMED | advance the program counter, or jump to a label |
| 0x005baae0 | `program_call_subroutine` | `CallSubroutine__9C_programFv` | 0x001ba490 | CONFIRMED | restart the program from line 0 as a subroutine |
| 0x005bab20 | `program_leave_compact_block` | `LeaveCompactBlock__9C_programFb` | 0x001ba4d0 | CONFIRMED | jump out of a compact block |
| 0x005babd0 | `program_call_process` | `CallProcess__9C_programFUi` | 0x001ba5c0 | CONFIRMED | run this script program's instruction loop |
| 0x005bac10 | `program_clear_after_end` | `ClearAfterEnd__9C_programFv` | 0x001ba650 | CONFIRMED | pop one saved-context frame and restore its instructions |
| 0x005bacb0 | `program_set_event` | `SetEvent__9C_programFi` | 0x001ba750 | CONFIRMED | raise a script event on this C_program |
| 0x005bad90 | `program_is_event_id` | `IsEventID__9C_programFiPi` | 0x001ba8d0 | CONFIRMED | does this program define this event? |
| 0x005bade0 | `program_invalidate_actor_reg` | `InvalidateActorReg__9C_programFP7C_actor` | 0x001ba950 | CONFIRMED | null every actor register holding a given actor |
| 0x005bae10 | `program_save_game_get_size` | `SaveGameGetSize__9C_programFv` | 0x001ba990 | CONFIRMED | savegame byte count for this C_program |
| 0x005bae90 | `program_save_game_save` | `SaveGameSave__9C_programFPPv` | 0x001baad0 | CONFIRMED | write this C_program to the savegame |
| 0x005bb020 | `program_save_game_load` | `SaveGameLoad__9C_programFPPv` | 0x001bad40 | CONFIRMED | read this C_program back from the savegame |
| 0x005bb2a0 | `program_find_actor_later` | `FindActorLater__9C_programFiUi` | 0x001bb060 | CONFIRMED | resolve an actor GUID into an actor register |
| 0x005bb2d0 | `program_process_command` | `Process__9C_programFUi` | 0x001bb0d0 | CONFIRMED | `C_program::Process`, the script VM's command executor |
| 0x005c9060 | `program_switch_light_map` | `SwitchLightMap__Q29C_program24S_help$4741C_program_cppFP9I3D_framei` | 0x001d29c0 | CONFIRMED | a recursive frame walk of a helper struct local to the unit, called by the dispatcher; its receiver is not a `C_program` |
| 0x005c9290 | `program_instructions_clear` | - | - | - | `this = +0x0c` empties the instruction vector, whose header it receives; one callsite, the dispatcher |
| 0x005c92c0 | `program_return_stack_back` | - | - | - | `this = +0x30` the top frame's address, from the return stack's header; three callsites, the dispatcher |
| 0x005c92d0 | `program_return_stack_push` | - | - | - | `this = +0x30` pushes a frame onto the return stack, growing the vector; one callsite, the dispatcher |
| 0x005c9480 | `program_return_stack_pop` | - | - | - | `this = +0x30` drops the return stack's top frame; one callsite, the dispatcher |

### 3d. Free helpers

The unit's helpers that fall under no other doc's prefix; every one is called by `program_process_command` (§8). The unit's other functions are inline members of other classes, each documented by its owner.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8020 | `matrix4_zero` | - | - | - | zero a 4x4 matrix |
| 0x005c8030 | `matrix4_invert_in_place` | - | - | - | invert a 4x4 matrix in place |
| 0x005c8060 | `quat_add` | - | - | - | add two quaternions componentwise |
| 0x005c80b0 | `quat_sub_w_add_xyz` | - | - | - | scalar difference, vector sum of two quaternions |
| 0x005c8100 | `quat_multiply_in` | - | - | - | multiply this quaternion by another, in place |
| 0x005c81f0 | `quat_scale_to` | - | - | - | scale a quaternion by a float into an output quaternion |
| 0x005c8240 | `quat_dot` | - | - | - | - |
| 0x005c8360 | `folded_get_ptr_0x90` | - | - | - | `lea eax,[ecx+0x90]; ret` 🔴 FOLDED SYMBOL - ONE address, TWO source-level accessors |
| 0x005c8ad0 | `cutscene_set_active` | - | - | - | set or clear the scripted-sequence suppression latch |
| 0x005c90d0 | `std_vector8_push_back` | - | - | - | append one 8-byte element to a std::vector, growing if needed |
| 0x005c94b0 | `std_vector8_insert_at` | - | - | - | generic `std::vector::insert(pos, count, value)` implementation for a stride-8 (8-byte element) vector triple at `receiver + 0x04 / +0x08 / +0x0c` |
| 0x005c96f0 | `stl_max_float_ptr` | `Max<f>__FRCfRCf` | 0x001571d0 | CONFIRMED | return a reference to the larger of two floats |
| 0x005c9710 | `stl_max_int_ptr` | - | - | - | generic 2-arg `std::max(int*, int*)` primitive |

## 4. The translation unit — one object file, two emission blocks

`.CRT$XCU` at `.data:0x00646224..0x0064622c` (the link-order table of dynamic initializers) puts
initializers at `0x005af290` and `0x005af2c0` — both immediately **before** the range, so they open
this unit — and the next one at `0x005c9720`, after it. **There is no initializer inside the range
at all.**

| span | what | count |
|---|---|---|
| `[0x005af2f0, 0x005c76fe)` | **block A** — the class's own methods, in PS2 source order | 37 defs |
| `[0x005c76fe, 0x005c7d94)` | **data** — `program_process_command`'s switch **jump table**, sitting inside `.text` | - |
| `[0x005c7da0, 0x005c8740)` | **block B** — out-of-line COMDAT inline members of *other* classes (§8) | 51 defs |

**0 phantom definitions.** A scan for function starts that the aggressive instruction finder invents
comes back clean here, and all 88 starts are 16-aligned; the three with no padding before them
(`0x005c8010`, `0x005c8530`, `0x005c85e0`) are preceded by a `ret` that lands exactly on the
boundary.

The jump table is why Ghidra's size for `program_process_command` (50 222 B) stops short of the
`int3` padding at `0x005c7d94` — nothing is missing from the function.

`C_program` sits at `C_entity+0xa9c` and at `C_detector+0x70` (§6a, §6d); the cross-build mapping
of the offset is in §10a.

## 5. The interpreter's runtime state — timing, the return stack, the register files

Every claim here is a body read of `program_process_command` plus the matching **load-time** arm —
`program_decode_params`, `program_decode_let`, `program_decode_if`,
`program_decode_operand_float` — which is the authority on the payload layout and on every operator
code.

**Timing.** `+0x24` is the milliseconds accumulated **on the current instruction**, not on the
program. `program_process_command` adds the frame delta to it on entry to *every* instruction;
`program_next_command` clears it whenever the pc moves. That single field is the whole timing
model: `WAIT` (case `0x4`) simply returns without advancing the pc while `+0x24` is below the
requested delay, so the same instruction re-evaluates next frame.

The `+0x28` / `+0x2c` pair exists so a suspended program does not lose that accumulation.
`program_set_event` snapshots `+0x24` into `+0x28` and the global game clock
(`C_game+0x2b0c`, `game_get_frame_time_ms`) into `+0x2c`; `RETURN` (case `0x3`) restores
`+0x24 = game_get_frame_time_ms() + (+0x28 - +0x2c)`, i.e. it **credits the wall time that
passed while the event handler ran**. A `wait` interrupted by an event therefore still expires
on schedule.

**The return stack** (`+0x34`..`+0x38`) holds **2-byte** frames, not pointers. One frame is a
packed `uint16`:

| bits | mask | meaning |
|---|---|---|
| 0..13 | `0x3fff` | the instruction index to resume at — written straight back into `pc` |
| 14 | `0x4000` | "inside a compact block". `RETURN` calls `program_leave_compact_block(0)`; when that returns 0 it also rebases `+0x24` as above. This is the bit `event_use_cb` (case `0x4a`) sets and clears on the top frame, and `program_set_event` sets it on every event frame it pushes. |
| 15 | `0x8000` | `RETURN` invokes virtual `+0x24` (slot 9, `CallBack`) with 1 before popping. `program_set_event` calls the same slot with 0 when it raises an event and pushes the pc with `0xc000` OR-ed in, so every event frame carries bits 14 and 15; it **refuses to raise an event while any frame carries this bit** — that is how re-entrancy is prevented. |

⚠️ **The 14-bit pc field caps a script at 16384 instructions.** Nothing checks it; a longer
program would wrap a `gosub` return silently. No shipped script comes near it: the longest are the
`GameInitEnd` scripts of `MISE15-MESTO` and `MISE19-MESTO` at about 600 lines, and a script
compiles to at most one instruction per line.

`GOSUB` (case `0x2`) pushes `pc + 1` with no flags and then falls **through** into the `GOTO`
body — the two commands share one implementation from the landing pad
`switchD_005bb32c_caseD_1` onward.

**The three register files** are plain `count`/`base` pairs — `flt[]` `+0x48`/`+0x4c`, `frm[]`
`+0x50`/`+0x54`, `act[]` `+0x58`/`+0x5c` — sized by `dim_flt` / `dim_frm` / `dim_act` (cases
`0x7`, `0xb`, `0xc`). All three share one implementation shape and one zeroing loop
(`LAB_005bb7e9`, which masks the count with `0x3fffffff`), and all three **only grow**: the
guard is `current_count < requested || requested == 0`, so a later `dim_*` with a smaller
non-zero argument is a complete no-op and the file keeps its size *and* its contents.

Every command that touches a register bounds-checks the index against the matching count
first, so a script that forgets its `dim_*` does not crash — the commands silently do nothing.

**Two payload conventions used across the whole dispatcher**, both settled at load time:

* **register-or-immediate.** An operand is a pair `(index, immediate)`. `index == -1` (or, in
  `IF`, any negative value) means "use the immediate". `program_decode_operand_float` writes
  the immediate with `crt_atof`, so **immediates are floats** — the `(float)argv[N]` cast that
  Ghidra renders in the case bodies is a typing artifact of `argv` being `int *`, not a
  conversion.
* **the empty string as a second form.** `GOTO`/`GOSUB` and `FINDFRAME` both take a name
  string and both give an empty name a different meaning. For `GOTO` it is a **computed
  jump**: `flt[argv[0]]` is converted with `__ftol`, formatted with `"%i"` (the literal at
  `0x00647794`) into a 0x400-byte stack buffer, and that decimal string becomes the label
  name — which is why shipped scripts number their jump targets (`label 101` … `goto 101`).
  For `FINDFRAME` it means "bind the owning actor's own `I3D_frame`" and requires
  `act_state_owner` (`+0x44`) to be set; shipped scripts do use it (`FINDFRAME  0`).

**Operator codes — byte-read, not inferred.**

| `IF` (`program_decode_if`, probe order) | code | `LET` (`program_decode_let`, probe order) | code |
|---|---|---|---|
| `<` | 1 | *(no operator)* → plain assignment, right index forced to `-1` | 0 |
| `>` | 2 | `+` | 1 |
| `=` | 3 | `-` | 2 |
| `!` | 4 | `*` | 3 |
| *(unrecognised)* → FALSE branch | - | `/` | 4 |

`IF`'s payload is `0x34` bytes: the two operand pairs, the code at `+0x10`, then **two 16-byte
label strings** copied by the caller to `+0x14` (the TRUE target, `yy`) and `+0x24` (the FALSE
target, `zz`) — each capped at 15 characters. The dispatcher's switch is written **inverted**:
each arm tests the *negation* and jumps to `argv+9` (FALSE), falling through to `argv+5` (TRUE)
otherwise. That is what makes the shipped `if flt[0]=1,SKIP,-1` idiom read correctly.

The `-1` in that idiom is **not a sentinel**: it is a label name no `label` instruction
defines, and `program_next_command` falls back to a plain pc increment when its backwards scan
misses. "Continue on the next line" is an emergent property of a failed lookup.

`LET`'s payload is `0x18` bytes (6 dwords). Its text must begin `flt[`, and the operator probe
order is `+` `/` `*` `-`, so the **first** of those characters found wins — there is no
precedence and no parenthesis handling. Division guards `|right| >= 1e-08` and writes 0 rather
than faulting.

**One undocumented arm.** `rnd xx, yy` (case `0xa`) splits on `yy < 2`: the documented form
(`yy >= 2`) is `rand_range(yy)` = `rand() % yy`, an integer in `0..yy-1` stored as a float; but
`yy < 2` calls `rand_float()` and yields a uniform **float in [0,1)**. The Mafia Script Data
Reference describes only the integer form.

### 5a. The three ways a script leaves its own instruction stream

`ctx_head` (`+0x40`) and `running` (`+0x64`) are two *different* mechanisms that both look like
"calling something".

**1. `gosub` / `return` — the RETURN STACK (`+0x34`).** Same instruction vector, different pc.
Documented above.

**2. `call <act_reg>` (case `0x12`) / `end` (case `0x6`) — the CONTEXT LIST (`+0x40`).** This
one swaps the *instruction vector itself* for another actor's script. The callee's vector comes from the actor's virtual slot 35
(`vtable+0x8c`), called twice: once as the guard (a null result skips the case and steps to the
next instruction) and once for the copy. Two classes fill the slot: `entity_get_program` returns
`C_entity+0xaa8` and `detector_get_program` `C_detector+0x7c`, each its `C_program`'s `+0x0c`
instruction-vector header; every other actor vtable the struct docs list keeps the base
`stub_ret0_cdecl_00477700`, so `call` on any other actor does nothing. `call` allocates a
**0x18-byte context frame**, saves `pc + 1` at its `+0x10` and the previous `ctx_head` at its
`+0x14` (so the list nests), copies every instruction of the current program into the frame's
own vector, publishes the frame, resets `pc` to 0, clears the live instruction vector and
pushes the callee's instructions in. `program_clear_after_end` — which `end` calls — is the
exact inverse: restore the frame's pc, empty the live vector, re-insert the saved
instructions, unlink and free.
⇒ `end` is therefore **not** unconditionally "stop": inside a `call` it is the return.
The Mafia Script Data Reference records `call` only as *"function call ???"*.

⚠️ `callsubroutine "name"` (case `0x22`) is a THIRD thing and shares nothing with either: it
hands a script name to `mission_global_program_run` and returns. No context frame, no pc
change, no bounds check.

**3. `commandblock nn` (case `0x21`) — `running` (`+0x64`).** The entire case body is
`running = argv[0] != 0`. That field is the interpreter loop's continue flag, so a command
block is literally "keep executing without giving the frame back", which is the Mafia Script Data
Reference's description (*code executed at the same time, as if it were a single command*) and the
reason that reference warns that a bad jump or an endless loop **inside** a block hangs the game
rather than just misbehaving. `return` also clears `+0x64` when `stop_on_return` (`+0x65`) is set,
so a subroutine can close a block it did not open.

### 5b. The owner fallback and fade rates

* **A fallback to the script's own actor.** `frm_seton`, `act_setstate`, `detector_issignal`,
  `detector_setsignal` and `playsound` all *start* from `act_state_owner` (`+0x44`) and only
  override it when the register index is valid. In those commands a bad index does **not**
  no-op — it targets the script's own actor. `door_lock`, `frm_ison`, `frm_linkto`,
  `getsoundtime` and the rest have no such fallback and do no-op. Which behaviour a command
  has is not guessable from its name; check the case.
* **Fades are stored as a RATE, not a duration.** `playsound`'s `+0x50` and
  `playsoundstop`'s `argv[1]` are floats that the *decoder* inverts (`x > 1.0 -> 1/x`), so a
  script writes seconds and the engine stores a per-unit rate. A zero is replaced at dispatch
  time by `0x3a83126f` = `1e-3f` rather than passed through.

### 5c. `compact_dir`, and two more payload conventions

**`+0x66` / `+0x67` are written by exactly two cases and nothing else.** `{` (case `0x45`) stores
`in_compact_block = 1` and `compact_dir` from the payload; `}` (case `0x46`) zeroes both. So
`compact_dir` never survives its block, and a stray `}` clears bytes that are zero.

🔑 **The `compact_dir` values are BYTE-READ from the load-time arm, not inferred.** The block may
carry one optional keyword, matched with `strnicmp`:

| written in the script | `compact_dir` |
|---|---:|
| `{` (no keyword) | 0 |
| `{ up` | 1 |
| `{ down` | 2 |
| `{ <anything else>` | 0 |

These are the `up` / `down` that the Mafia Script Data Reference describes only as "adjusting where
a subprogram returns to in the main program", and they are the same two directions §1's
`compact_dir` comment records as "force backwards / force forwards".

**Two further conventions, both of which change how a case must be read:**

* **An out-of-range register index can be a deliberate ARGUMENT, not a failure.** `garage_addcar`
  (`0x58`) and `garage_delcar` (`0x5a`) fall back to *the car the player is sitting in* when the
  `act[]` index is out of range, so `garage_addcar -1` is the documented way to say "this car".
  `human_looktoactor` (`0x51`) and `human_looktoframe` (`0x52`) use the same shape to mean *reset*:
  the load-time arm defaults the target to `-1`, and the dispatch arm routes an out-of-range target
  to `human_pose_null_rotate`. This is distinct from the §5b owner fallback — here the
  out-of-range value selects a **different behaviour**, not a different target.
* **Several commands take more arguments than the Mafia Script Data Reference documents.** The
  load-time arm is the authority on arity, and three cases accept arguments the reference does not
  list at all: `getactorsdist` (`0x38`) has a 4th argument that negates the distance when the second
  actor is dead; `human_unlinkfromhand` (`0x56`) takes up to five and, with a weapon id, *drops a
  pickable weapon* rather than merely unlinking; `person_playanim` (`0x5f`) takes a `SIMPLE` keyword
  and a trailing bool. Read the decoder arm before trusting an argument count.

### 5d. The payload as mutable state, and the event-target idiom

**🔑 A case may write back into its own payload, so an instruction can carry state between
executions.** Three instances:

* `human_talk` (case `0x83`) reserves `argv[3]`, initialised to `-1` by the load-time arm, and
  stores the live **speech channel** there. Its two arms are selected by that slot: `-1` means
  "start speaking and block", anything else means "check whether the channel is still busy".
  It resets the slot on the way out, so the instruction is reusable.
* `setaipriority` (case `0x47`) clamps its float argument **in place**, so a value outside
  0..100 is permanently clamped for that instruction after its first execution.
* `human_unlinkfromhand` and `createweaponfromframe` (cases `0x56`/`0x57`) read optional
  payload slots the load-time arm defaulted, which is the read-only half of the same idea.

⇒ Do not assume a case body only reads its payload. Check for writes before describing an
instruction as stateless — it changes what a second execution does.

**The event-target idiom, shared verbatim by three cases.** `settimeoutevent` (`0x6f`),
`setplayerfireevent` (`0x72`) and `setplayerhornevent` (`0x74`) all take an *actor* register and
convert it to a **`C_program` pointer** before storing it:

| the named actor's kind | what is stored |
|---|---|
| 5 (`C_detector`) | actor `+0x70` |
| 0x1b (`C_entity`) | actor `+0xa9c` |
| anything else | nothing — the case breaks |
| guard failed, or a null `act[]` slot | the pair `(0, 0)`, which **clears** the registration |

Those are the two `C_program` sub-object offsets §1 and `structs/entity.md` §4 record. The last
row is the part worth remembering: pointing one of these commands at an invalid actor does not leave
the previous registration alone, it **removes** it.

### 5e. The math register conventions and two payload rules

The `VECT_*` / `MATRIX_*` / `QUAT_*` / `FRM_*` families are where the script language's *typed* use
of the flat `flt[]` pool becomes visible.

🔑 **THE MATH TYPES ARE RUNS OF `flt[]` SLOTS, AND THE BOUNDS CHECK STATES THE WIDTH.** Every case
in those families validates its operand as

| type | slots | the guard the case emits |
|---|---:|---|
| scalar | 1 | `idx < flt_count` |
| vector | 3 | `idx + 2 < flt_count` |
| quaternion | 4 | `idx + 3 < flt_count` |
| matrix | 16 | `idx + 0xf < flt_count` |

That is worth stating as a convention rather than per case, because it is how an operand's TYPE is
recovered for the ~20 commands the Mafia Script Data Reference marks `???` with no argument list at
all — the guard is the type declaration. It also independently confirms that reference's
"3 / 4 variables in total" wording for the commands it does document, read off the binary instead of
the reference.

🔑 **THE `-1` IMMEDIATE SENTINEL.** Seven cases accept EITHER a literal or an `flt[]` index in one
operand and mark the literal case by storing `-1` in the index slot: `vect_mul_scl` (`0x103`),
`quat_mul_scl` (`0x11b`), `weather_setparam` (`0xcf`), `sound_setvolume` (`0x140`),
`camera_setfov` (`0x160`), `car_setactlevel` (`0x165`) and `frm_getchild` (`0x16f`). It is a
decoder-wide convention, not a per-family accident. Ghidra renders the float-typed compare
against `0xffffffff` as `== -NAN`; it is the integer `-1`. Note this is a *different* sentinel from
the project-wide `666` "none/unset" value.

🔑 **THE REGISTER OPERAND CAN BE THE LAST PAYLOAD SLOT.** When a command's leading arguments are
float immediates, the load-time decoder emits the immediates first and the register index after
them. `vect_set ww, xx, yy, zz` reads its OUTPUT index from `argv[3]` and the three components
from `argv[0..2]`; `car_setprojector ww, xx, yy, zz` reads its ACTOR index from `argv[3]` the same
way; `stream_fadevol`'s index is `argv[2]`. ⇒ **Never infer payload order from the written argument
order** — read the case. It is a decoder rule, not a per-command accident.

**Two unit conventions that disagree inside one family.** `math_cos` / `math_sin` (`0xf0` / `0xf1`)
take DEGREES — they multiply by `0.017453292` = pi/180 before calling the primitive — and
`getangleactortoactor` (`0xd3`) returns degrees, so the two compose. But `vect_angleto` (`0x10d`)
calls the *same* `S_vector::AngleTo` helper as `getangleactortoactor` and does NOT convert, so it
yields RADIANS. One primitive, two units, no marker in either command name.

**The remote-register family (`0xe2`..`0xe7`) reuses the event-target receiver idiom** of §5d:
the target actor is converted to a `C_program *` by kind — kind 5 (`C_detector`) at `+0x70`, kind
`0x1b` (`C_entity`) at `+0xa9c`. Both kind arms render identically in the decompiler because the
only difference between them is the hidden `this` adjustment. `setplayerwfallevent` (`0x128`) is the
family's fourth event registrar and clears the registration by storing `(0, 0)` on a failed guard,
exactly as the other three do.

### 5f. The two ways a case can block the script

🔑 **A CASE CAN REFUSE TO ADVANCE THE INSTRUCTION POINTER.** Almost every case ends in `break`,
which lets `program_process_command` advance the pc. Two cases instead restore the exception
list and **`return` from the dispatcher**, so the SAME instruction is re-executed next frame:

| case | command | blocks while |
|---|---|---|
| `0x170` | `human_waittoready` | `entity_is_tickable` is false for the named character |
| `0x17b` | `group_disband` | its phase counter is 0 or 1 (see below) |

This is a third control-flow mechanism alongside the return stack, the context list and
`callsubroutine` (§5a). It is also fail-safe in both cases: a guard failure `break`s normally,
so a bad argument does not hang the script — only a valid but still-busy target waits.

🥇 **`group_disband` is the most developed instance of the payload-as-mutable-state idiom**
(§5d). `argv[1]` holds a phase counter and `argv[2]` a saved vehicle handle, both written back
into the instruction's own payload, so one script instruction runs a three-phase wind-down one phase
per frame: freeze the members and record the fight vehicle, wait until every member is accounted
for, then per member clear the AI logs, re-init sensors and visibility logs, clear the action
manager, un-suspend the state switcher, leave the fight party and hand the actor back to `ai_groups`.
Disbanding is a supervised drain, not an event.

⚠️ **Two cases pass their argument in a register Ghidra does not render** — `taxidriver_enable`
(`0x16a`) and `npc_shutup` (`0x175`) both read the payload and then call a setter that appears to
take no argument. The payload IS the argument: it is passed in ECX as the `__thiscall` receiver,
which the decompiler drops from the call it renders. A third, `pumper_canwork` (`0x17a`), validates
its SECOND payload slot and never validates the first for the same reason.

All 375 case labels carry a body docblock in the decomp.

## 6. The `C_program` base vtable

### 6a. Slot map — `.rdata 0x0063b45c`

Ordered by slot. A **thunk** row means the slot is a `C_entity` override reached through the
adjustor stub; a **direct** row means the target is the class's own code.

| slot | PC target | kind | what it is |
|---|---|---|---|
| 0 | `program_base_thunk_entity_scalar_deleting_dtor` → `entity_scalar_deleting_dtor` | thunk | destructor |
| 1 | **`program_save_source_code`** | **direct** | the ONE virtual `C_entity` does not override — `C_program`'s own `SaveSourceCode` |
| 2 | `program_base_thunk_entity_save_game_get_size` → `entity_save_game_get_size` | thunk | savegame sizing |
| 3 | `program_base_thunk_entity_save_game_save` → `entity_save_game_save` | thunk | savegame write |
| 4 | `program_base_thunk_entity_save_game_load` → `entity_save_game_load` | thunk | savegame read |
| 5 | `entity_script_decode_params` | override | parses a command's argument TEXT into a typed `S_instruction` |
| 6 | `entity_script_decode_instruction` | override | maps a command NAME to its opcode |
| 7 | `entity_process_command` | override | **`C_entity::Process`** — executes one command |
| 8 | `entity_script_format_command` | override | renders a command back to text |
| 9 | `entity_script_callback` | override | `CallBack`: `program_set_event` calls it with 0 when it raises an event, `RETURN` with 1 when it pops a frame carrying bit `0x8000` |

The table has ten slots; the dword after slot 9 is 0.

`program_compile` confirms this slot map from the other end: it calls `vtable+0x14` for the
parameters and `vtable+0x18` for the opcode — slots 5 and 6. The slot-1 pairing is corroborated
across builds in §10g.

### 6b. Why slots 0/2/3/4 are thunked and 5–9 are not

Slots 0 and 2–4 are virtuals **declared on the C_entity side**, so their implementations expect
the `C_entity` `this` and need the `−0xa9c` adjustment. Slots 5–9 are virtuals **declared on
`C_program`**, so MSVC compiled C_entity's overrides to take the `C_program` sub-object pointer
directly and adjust internally.

### 6c. The four thunks are real definitions

Each was screened the way a suspected decompiler-invented stub is screened — alignment, the padding
before the start, datarefs and direct callers: all four are 16-byte aligned, preceded by `cc`
padding, and carry one dataref each (their vtable slot) with zero direct `E8` callers.

### 6d. The standalone vtable is a DIFFERENT table

`program_ctor` installs **`g_program_vt`** (`0x0063eab0`) and `program_dtor` restores it. That is
the vtable of a *standalone* `C_program` — the one a `C_detector` embeds at `+0x70`. The table in
§6a (`0x0063b45c`) is the *base sub-object* vtable inside a `C_entity`. Two tables, one class,
different embeddings.

| slot | PC target | what it is |
|---|---|---|
| 0 | `program_scalar_deleting_dtor` | destructor |
| 1 | `program_save_source_code` | `SaveSourceCode` |
| 2 | `program_save_game_get_size` | savegame sizing |
| 3 | `program_save_game_save` | savegame write |
| 4 | `program_save_game_load` | savegame read |
| 5 | `program_decode_params` | parses a command's argument text |
| 6 | `program_decode_instruction` | maps a command name to its opcode |
| 7 | `program_process_command` | executes one command |
| 8 | `stub_noop_args8_0060b1a0` | `DecompileCommand`: `ret 8`, a shared empty stub |
| 9 | `stub_noop_args4_004776f0` | `CallBack`: `ret 4`, a shared empty stub |

In a standalone program slots 8 and 9 return at once. The table has ten slots, like §6a's.

## 7. `Process` — the one place two classes are easy to confuse

| | PC | PS2 | size | reached how |
|---|---|---|---|---|
| base | **`program_process_command`** | `Process__9C_programFUi` | PC 50 222 B / PS2 95 704 B | direct call, 2 PC callsites |
| override | `entity_process_command` | `Process__8C_entityFUi` | 22 332 B (PS2) | vtable slot 7 / `@3056@Process` |

`program_process_command` is the giant opcode `switch` the whole `script-mapping/` corpus
documents, decompiled in full with 375 command arms; its switch jump table occupies `[0x005c76fe, 0x005c7d94)` in `.text`.

🔑 **51 of the functions in block B exist because of it.** Every out-of-line inline member in §8
has `program_process_command` among its callers — that is what makes block B one object file with
block A.

The twin is graded per command arm: the two dispatchers' arms are joined by command name and
compared call by call (§10h).

## 8. Block B — the out-of-line inline tail `[0x005c7da0, 0x005c9720)`

51 tiny COMDAT inline members that belong to **other** classes and were emitted into this object
file because `program_process_command` is where they were used. They are the first block of the
tail; the band runs to `0x005c9720` (§8c).

**How ownership is established here, and why callers do not establish it.** Header inlines are
always COMDAT, so this is exactly the population that identical-COMDAT folding merges: a body that
occurs exactly once image-wide is the *signature* of a folded body, not evidence that nothing
folded. One member of this band, `folded_get_ptr_0x90`, is a proven fold, and the address
`0x004253a0` (`folded_get_dword_0x10`) serves both `C_actor::GetKind` and `C_mission::GetScene`.
A caller set confined to one translation unit therefore does not prove ownership — and in fact
**10 of the 51 have callers elsewhere**: `mission_open_bin` calls seven of them (17 of
`i3d_frame_get_type_ptr`'s 41 callsites) and `game_init` calls three. The conclusion that this is
the `C_program` object file's inline tail rests on the `.CRT$XCU` boundary (§4), not on the caller
sets.

**Census of all 51**, by disassembling each callsite's ECX receiver, scanning `.rdata`/`.data` for
each address as a pointer, and byte-searching `.text` for each body and for `E9` tail-jumps to it:
no fold beyond `folded_get_ptr_0x90`, **0 datarefs** across all 51, 0 `E9` referrers, and 49 of 51
bodies byte-unique. **17 receivers are an immediate or a global load**; the other 34 are a register,
and following each register one step further back resolves every one of them. Inside
`program_process_command` the register is loaded from one of the VM's register files — `flt[]` at
`C_program+0x48`, `frm[]` at `+0x50`, `act[]` at `+0x58` (`ebp` is the `C_program` throughout the
dispatcher) — or is the return of `folded_get_dword_0x68` (an actor's frame),
`folded_get_dword_0x10` on `g_act_mission` (the scene), `i3d_frame_get_parent_ptr`,
`operator_new` or `mission_create_actor`; in `mission_open_bin` it is a frame from the frame map
or the deferred frame list.

| receiver | basis | functions |
|---|---|---|
| `hud_state` `0x00658330` (`G_IndicatorsClass`) | byte-proven | `hud_set_element_flags`, `hud_hide_element`, `hud_element_is_visible`, `hud_push_element_flags`, `hud_pop_element_flags`, `hud_set_police_wanted_bar` |
| an `S_matrix` / `S_quat` living in the script `flt[]` register file | `lea [base + idx*4]` with `base = C_program+0x48`; `matrix4_invert_in_place` takes its matrix as a stack argument, a `flt[]` span or a stack-local matrix | `matrix4_zero`, `matrix4_invert_in_place`, `quat_add`, `quat_sub_w_add_xyz`, `quat_multiply_in`, `quat_scale_to`, `quat_dot` |
| `g_racing` `0x006f958c` (`C_racing`) | byte-proven | `racing_get_racer_array_ptr`, `racing_get_racer_count` |
| `g_cache_base_block` `0x006efd80` | byte-proven | `cache_base_block_is_loaded` |
| an `I3D_frame` from the VM's frame register file | `C_program+0x50`, `folded_get_dword_0x68`, `i3d_frame_get_parent_ptr`, the frame map of `mission_open_bin` ¹ | `i3d_frame_get_type_ptr`, `i3d_frame_set_position_xyz_ptrs`, `i3d_frame_set_scale`, `folded_get_ptr_0x90` (its `I3D_frame::GetScale` role), `i3d_frame_get_rotation_ptr`, `i3d_frame_get_world_rotation_quat`, `i3d_frame_get_scaled_rot_matrix_ptr`, `i3d_frame_is_on`, `i3d_frame_get_name`, `i3d_frame_get_tag_ptr`, `i3d_frame_get_mesh_object_ptr`, `i3d_frame_get_lmap_entry_ptr`, `i3d_frame_get_model_name` |
| the `I3D_scene` at `*(g_act_mission + 0x10)` | the return of `folded_get_dword_0x10` on `g_act_mission` | `i3d_scene_get_active_camera_ptr`, `i3d_scene_set_frame_sector_pos` |
| a `C_actor` from the VM's actor register file | `C_program+0x58` | `actor_set_cached_world_pos` |
| `g_act_mission` `0x0065115c` (`C_mission`) | byte-proven | the seven `mission_*` accessors (§8a) |

¹ `folded_get_ptr_0x90` is the exception in that row: it is the band's one proven fold.
`i3d_frame_get_type_ptr` shares its 7 bytes (`lea eax,[ecx+0x110]; ret`) with
`net_session_settings_addr` (`0x00597710`), an unfolded surviving pair.

🔑 **The script VM keeps matrices and quaternions in the `flt[]` register file.** `matrix4_zero`'s
caller bound-checks `i + 0xf < float_reg_count` — sixteen consecutive registers — and the five
quaternion helpers take spans of four. That is what these math inlines are doing in a script VM.

### 8a. How a block-B member gets its name

Each is reached from exactly one arm of `program_process_command`'s switch. Decoding that arm names
the field, because the command's meaning is documented — `basic/artifacts/script-opcode-dispatch.md`
holds the command map and `script-mapping/artifacts/opcodes-verified.md` the per-command
documentation — and the PS2 build's inlined arm supplies the second body (§10g). A full command↔case
map is not required; only the arms that reach the member.

That is why these accessors carry FIELD names rather than offset names, among them these sixteen:
`i3d_frame_get_child_count_ptr` (`0x005c8530`),
`i3d_animation_set_set_default_options` (`0x005c8540`), `i3d_camera_get_fov_ptr` (`0x005c85a0`),
`i3d_scene_get_root_frame_ptr` (`0x005c85f0`), `i3d_sound_get_volume_ptr` (`0x005c8630`),
`mission_get_actor_list` (`0x005c8640`), `mission_get_particle_manager` (`0x005c8650`),
`mission_set_next_mission_number` / `mission_get_mission_number` /
`mission_commit_next_mission_number` (`0x005c8660`/`0x005c8670`/`0x005c8680`),
`mission_set_frame_skip_count` / `mission_get_frame_skip_count` (`0x005c8690`/`0x005c86a0`),
`using_object_record_ctor` (`0x005c86b0`, the constructor of an `operator_new(0x38)` object; it
zeroes `+0x10` and `+0x18..+0x34`, deliberately skipping `+0x14`), `actor_get_class_id`
(`0x005c86e0`), `actor_get_direction` (`0x005c8710`) and `actor_mark_created_at_runtime`
(`0x005c8730`).

### 8b. The four members reached through the LS3D visual chain

`i3d_face_group_get_material_ptr` (`0x005c8550`, `return this+4`),
`i3d_mesh_level_get_face_group_count_ptr` (`0x005c8560`, `+0x18`) and
`i3d_mesh_object_get_lod_count_ptr` (`0x005c8570`, `+8`) have one callsite each, in the chain
`frame → visual (+0x1e0) → +8 → vtbl[1] → +0x18 → vtbl[0x11] → +4` of the `STREAM_CONNECT` arm,
which crosses two LS3D classes. Two independent levers name every hop:

1. **A second PC consumer of the identical chain.** `glass_shatter_cut_mesh` walks
   `frame+0x1e0 -> vt[1](0) -> vt[5](0) -> [+4]` exactly as the `STREAM_CONNECT` arm does,
   and its twin is CONFIRMED (§10g).
2. **The loop bounds.** At `0x004fc8fd` each of the two counts is the bound of the indexed
   getter loop it guards, which is what a count is.

`i3d_animation_set_set_default_options` (`0x005c8540`) is named from its receiver instead: its ECX
is `[esp+0x4c]`, the OUT slot of `i3d_cache_anim_create`, whose twin's mangled name spells the type
(§10g). Its single callsite is cross-block, so the receiver census alone yields nothing. The classes
are documented in `structs/i3d_frame.md` §8 and `structs/i3d_cache.md` §1b.

### 8c. Where block B ends

The next `.CRT$XCU` initializer is at **`0x005c9720`**, so `[0x005c8740, 0x005c9720)` is the rest of
this same object file — 74 more helper-tail definitions, all named, from
`actor_set_save_excluded` to `stl_max_int_ptr` — and **`0x005c9720` is where the next
translation unit starts**. `basic/artifacts/script-opcode-dispatch.md` §9 maps every definition in
`[0x005c7da0, 0x005c9740)` to the script command(s) whose arm reaches it.

## 9. `vec_index_addr_stride_0x8` — the indexer the VM shares with the mission loader

`0x006065d0` is a 13-byte generic `std::vector<8-byte>::operator[]` — `return *(this+4) + i*8`,
no bounds check — paired at nearly every callsite with `vec_count_stride_0x8` (`0x00474200`),
which reads the same header's `+0x04`/`+0x08`. Its 377 `E8` callsites are the reason it matters to
this class: 353 of them are inside `program_process_command`.

1. **Byte-unique.** `8b41048b4c24048d04c8c2` occurs exactly once image-wide; 0 datarefs, 0 `E9`
   tail-jump referrers. For a COMDAT-eligible body that is the fold signature, not evidence
   against one.
2. **Field offset.** It indexes the vector's OWN `+0x04`, never an offset inside an owner.
   🥇 **The receiver is ALWAYS a vector header and NEVER a class instance** —
   `program_process_command`'s prologue is `mov ebp, ecx` then `lea esi, [ebp + 0xc]`, so the
   script VM passes `C_program+0x0c`, whose `begin`/`end` are `program+0x10`/`+0x14`: exactly the
   instruction vector `structs/entity.md` §4 proves independently. `entity_action_tick` passes
   the same header through `C_entity+0xaa8`.
3. **Receiver census, 377 `E8` callsites, hand-disassembled:**

| container | callsites | receiver | element type |
|---|---:|---|---|
| `program_process_command` | 353 | `[ebp+0xc]` = `C_program+0x0c` | `{int opcode, int *args}` |
| `mission_open_bin` | 13 | `[esp+0x60]` and `[esp+0x84]` — two STACK LOCALS | `{I3D_frame *, int}` |
| `entity_action_tick` | 6 | `[esi+0xaa8]` = the same `C_program+0x0c` | script node |
| `entity_process_command` | 5 | `esi` | script node |

The `mission_open_bin` arm is what settles the ownership question: its loop is
`vec_count_stride_0x8` → this helper → `i3d_frame_get_type_ptr(elem[0])` → `cmp [eax], 2/4/5/0xe` →
a type-dependent virtual. That is a **deferred frame-fixup list** built on the stack during `.bin`
load, holding `{I3D_frame *, index}` pairs, not a script node array.

⇒ One address serves the script VM and the mission loader. Whether that is identical-COMDAT
folding of two element types or dedup-by-name of one declared 8-byte pair type cannot be decided:
the element types are known only from their use, and neither binary records the declarations
(§10i). Either way the name describes the operation (`vec_index_addr_stride_0x8`) and not a
subsystem. `folded_*` is
reserved for folds whose owners are CLASSES sharing a field offset; here there is no class owner at
all. The stride-4 sibling is `vec_index_addr_stride_0x4` (`0x005a17b0`).

## 10. PC vs PS2

All cross-build material of this doc is in this section.

### 10a. Base offset and the thunks

- **Base sub-object: PC `C_entity+0xa9c` / PS2 `C_entity+0xbf0`** (delta `0x154`). Byte-proven on
  both sides — PC by the `sub ecx, 0xa9c` quartet, PS2 by GCC's `@3056@` prefix (3056 = `0xBF0`).
- **The base offset is not a rebase of any other C_entity offset.** The AI sub-objects on the same
  class sit at different deltas (`ai_ghost` `0x1f8`, `ai_action_manager` `0x1e4`). The deltas are
  **band-wise, not constant** — `structs/entity.md` warns against rebasing a single offset.
- The adjustor thunks exist on both builds in the same four slots; MSVC emits them for the
  C_entity-declared virtuals, GCC for the C_program-declared ones. Different halves, same
  construct — do not read the asymmetry as a behavioural difference.

### 10b. Field-offset delta

**PC = PS2 + 0x0c** across the scalar band: PS2 `+0x14`↔PC `+0x1c` (pc), `+0x18`↔`+0x20`,
`+0x1c`↔`+0x24`, `+0x3c`↔`+0x48`, `+0x40`↔`+0x4c`, `+0x44`↔`+0x50`, `+0x48`↔`+0x54`,
`+0x4c`↔`+0x58`, `+0x50`↔`+0x5c`, `+0x54`↔`+0x60`, `+0x58`↔`+0x64`, `+0x5b`↔`+0x67`.
The two source strings are the exception: PC `+0x04`/`+0x08` ↔ PS2 `+0x00`/`+0x04`.

### 10c. 🔑 Why `DecodeInstruction` is 94× bigger on the PC

PC 35 641 B against PS2 380 B. **Both bodies do the same thing**: build the `0x24`-stride keyword
table (`char name[0x20]; int opcode;`) as a **stack local**, then linear-scan it with
`strncasecmp`, returning the matched record's `+0x20`. GCC copies the table with a loop out of
`.rodata`; MSVC materialises all ~16 KB of it with inline immediate stores. The ratio says nothing
about the function.

🔑 **And the two builds do not ship the same command set:** the PC scans indices `0..0x186`
(**391 commands**), the PS2 `0..0x19a` (**411**).

### 10d. The script-opcode numbering differs by 2

Instruction-type constants are offset by two between the builds, consistently across the class:
block markers PC `0x45`/`0x46` ↔ PS2 `0x47`/`0x48`; the event marker PC `0x48` ↔ PS2 `0x4a`.

### 10e. Six PS2 methods with no `C_program` function of their own on the PC

| PS2 VA | PS2 method | What the PC has |
|---|---|---|
| 0x001d3ca0 | `DebilitizeScripts__9C_programFv` | **PS2-PORT-ONLY, terminal**: its *only* caller is `InterCity_ChangeMission__9C_mission`, itself PS2-only; the same block also calls `VystrcPero__7C_human` and `ChangeSceneVystrc__10C_shvestky`, both PS2-only |
| 0x001d2b50 | `GetInterCityName__9C_programFPc` | **PS2-PORT-ONLY, terminal**: `InterCity_*`: 127 mentions on PS2, 0 on PC |
| 0x001b6b60 | `TryChangeProgramData__9C_programFv` | **PS2-only, terminal**: the image holds exactly one call to it (one `jal`, no pointer), the tail of PS2 `LoadSourceCode`. PC `program_load_source_code` makes the same calls in the same order — the two frees, the seek/read pairs and the two allocations — and ends exactly before that position. It applies `DAT_004e2174`/`DAT_004e2178`, a `{mission path, program name, actor name, replacement source}` table with **zero writers anywhere in the PS2 image** — a compiled-in per-mission script override list. It compares PS2 `+0x00` (PC `program_name`, §10b) case-insensitively with the table's program-name field; both null also matches |
| 0x001d2bd0 | `PreloadAnims__9C_programFv` | **PS2-only, terminal**: the image holds exactly one call to it (one `jal`, no pointer), the tail of PS2 `Init`. PC `program_init` makes the same calls in the same order — three frees, the allocation, the compile, the free — and ends exactly before that position; 4304 B could not have been spliced into its 278-byte body either. It walks the compiled instruction stream for animation opcodes (`0x1001`…) and pre-loads every clip they name — a streaming adaptation. Sits in the PS2's SECOND `C_program` emission block (`0x001d2b50…`) between the two other terminal entries |
| 0x001ba240 | `DecompileCommand__9C_programFiPc` | 8-byte PS2 base stub. The PC base implementation is slot 8 of `g_program_vt`, the shared empty `stub_noop_args8_0060b1a0` (`ret 8`, §6d); the `C_entity` override is `entity_script_format_command` (`0x00429ca0`, §6a) |
| 0x0014c700 | `CallBack__9C_programFQ29C_program11E_cb_action` | 8-byte PS2 stub outside the cluster. The PC base implementation is slot 9 of `g_program_vt`, the shared empty `stub_noop_args4_004776f0` (`ret 4`, §6d); the `C_entity` override is `entity_script_callback` (`0x0042e990`, §6a) |

Both rows carry an inline banner in the decomp at the exact statement where the PS2
makes the call — see `program_load_source_code` (`LAB_005af709`) and `program_init` (after the
`operator_delete_wrapper(pcVar2)` that ends the compile step).

### 10f. Emission blocks

The PS2 splits this class over **two** source files: the main cluster `0x001b6450..0x001d2748`,
and a second block at `0x001d2b50..0x001d3e18` holding only `GetInterCityName`, `PreloadAnims` and
`DebilitizeScripts`. The PC has no second block — consistent with two of those three being
PS2-port-only.

### 10g. What the PS2 build supplies for §6 and §8

- **Vtable slot 1.** PS2 `SaveData__10C_detectorFP7C_chunk` tail-calls
  `SaveSourceCode__9C_program(this+0x80, chunk)`, exactly where PC `detector_save_data` tail-calls
  `program_save_source_code`. The pair is `SaveSourceCode` / `LoadSourceCode`; no
  `SaveData__9C_program` symbol exists in the PS2 symbol table.
- **Block B.** The PS2 build inlines every one of block B's accessors into its own
  `Process__9C_programFUi` arm, so the PS2 arm for the same command is a second body: block B's
  members are named from body diffs rather than from consumer guesses (§8).
- **The `STREAM_CONNECT` chain (§8b).** The PS2 twin of `glass_shatter_cut_mesh`,
  `CutMesh__19C_Break_plain_glass...`, is CONFIRMED. The PS2 `STREAM_CONNECT` arm (opcode 0xb8) is
  an empty stub — its entire body is `NextCommand__9C_programFPc(param_1, 0)` — so for the three
  chain members there is no second body on that arm; the levers of §8b go around it.
- **`i3d_animation_set_set_default_options` (§8b).** The PS2 twin of `i3d_cache_anim_create` spells
  the receiver type in its mangled name (`...FPCcPP17I3D_animation_set...`). The PS2
  `MODEL_PLAYANIM` counterpart is a cache-slot refcount INCREMENT where the PC assigns 2.

### 10h. The dispatcher, arm by arm

`program_process_command` and `Process__9C_programFUi` are compared one command arm at a time. The
arms are joined by command NAME through each build's own keyword table (391 records on the PC,
411 in the PS2 ELF; the opcode numbers differ, the names do not), which pairs 366 arms. For every
pair, each out-of-line call the PS2 arm makes to a method with a registered PC twin is looked up
in the PC arm. `NextCommand` is left out: every PS2 arm calls it, while the PC advances the pc
once after the switch.

| outcome | arms |
|---|---:|
| every such call has its PC twin in the PC arm | 113 |
| one to three calls differ | 12 |
| the PS2 arm could not be cut out of its if-chain cleanly | 4 |
| the PS2 arm calls no method with a registered PC twin | 237 |

The 12: `WAIT`, `HUMAN_LOOKTOFRAME`, `CAMERA_LOCK`, `AUTOSAVEGAME`, `AUTOSAVEGAMEFULL`,
`STREAM_CREATE`, `ACTOR_DELETE`, `ENEMY_BLOCK`, `CITYMUSIC_OFF`, `FREERIDE_SCORESET`,
`HUMAN_RETURNTOTRAFF`, `INVENTORY_CLEAR`. `GARAGE_GENERATECARS` and `GARAGE_RELEASECARS` call the
empty PS2 `GenerateCars` / `ReleaseCars`, which have no registered PC function: the PC arms call the
shared empty `stub_noop_cdecl_00574a40` (`structs/garage.md` §4a). The four:
`HUMAN_GETITEMINRHAND`, `LOADDIFFERENCES`, `MODEL_STOPANIM`, `MODEL_DESTROY`.

### 10i. The element types behind `vec_index_addr_stride_0x8`

Neither build records whether the script VM's and the mission loader's 8-byte vectors share one
declared element type (§9). The PC has no RTTI and no symbols; the PS2 `C_mission::OpenBin` calls
no helper of `vector<C_program::S_instruction>` (none of its vector helpers carries that
instantiation's length code), and the frame list it does build holds the 16-byte
`C_mission::S_frame_actor_buffer`, so it has no 8-byte frame pair to compare with.

## 11. Related

- `structs/i3d_frame.md` — the LS3D scene node §8 keeps referring to.
- `structs/entity.md` — `C_entity`'s layout; this class is a base of it.
- `structs/detector.md` — carries a standalone `C_program` at `+0x70`.
- `structs/i3d_cache.md` — the animation-set cache §8b reaches.
- `script-mapping/artifacts/opcodes-verified.md` · `script-mapping/artifacts/dispatcher-symbols.md`
  — the per-command documentation of `program_process_command`.
- `basic/artifacts/script-opcode-dispatch.md` — the command map.

## 12. Open items

Nothing is open.
