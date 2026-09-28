# Named locals, tail labels and accessors of `program_process_command`

**Created** 2026-06-16 21:58 +0200 · **Modified** 2026-09-17 13:47 +0200

`program_process_command` (the script-VM dispatcher) reuses a handful of compiler scratch slots and
jump targets across hundreds of case bodies. This document explains the role of each named symbol
in three categories:

1. **Function-scoped locals** — compiler scratch slots reused across many case bodies.
2. **Shared dispatcher tail labels** — jump targets reused by several cases to write a float into
   the script's `flt[]` register file.
3. **Field accessors** — three one-line getters on `C_game` that the boolean query cases call.

Two case bodies are the reference for the patterns below:

- **VLVP (case `0x122`)** — writes 1.0 to `flt[arg0]` if any officer has a recent sighting of the
  player.
- **POLICEITCHFORPLAYER (case `300`)** — writes 1.0 to `flt[arg0]` depending on the officers that see
  the target and on whether the target's identity is still pending.

Common register-file layout: `param_1[0x12]` = `flt[]` base pointer (32-bit floats), `param_1[0x13]`
= number of valid `flt[]` slots.

---

## 1. Function-scoped locals

All six locals are scoped to `program_process_command`. Every one of them is **reused across many
cases**: the role below is the dominant pattern, not a per-callsite guarantee — always read the
surrounding case body.

| Local | Type | Dominant role | Confidence |
|---|---|---|---|
| `script_scratch_i0` | `int` | primary scratch: the opcode's first argument (typically the destination `flt[]` index), range-checked against `param_1[0x13]`; also receives `vec_index_addr_stride_0x8(param_1[7])` at the top of most cases | high for the role, low per callsite ¹ |
| `script_scratch_i1` | `int` | secondary scratch: the second argument of a multi-arg opcode, or an intermediate callee result | high for the role, low per callsite ¹ |
| `script_scratch_u0` | `uint` | most often a re-fetched actor handle passed to a follow-up call ² | medium-high ³ |
| `script_pred_c` | `char` | the boolean result of the case's main predicate call, tested against `'\0'` to pick the 0.0 / 1.0 write tail | high ⁴ |
| `script_flt_payload` | `float` | the value ferried into the shared `script_flt_write` tail (1.0 / 0.0 for boolean queries, the computed value for numeric ones) | high |
| `script_argv` | `int *` | the opcode's argument list: `*script_argv` = arg0, `script_argv[1]` = arg1, … | high — universal |

¹ A case body may temporarily reuse the slot for an unrelated value mid-block.
² In VLVP and POLICEITCHFORPLAYER the body re-queries `game_get_player_actor()` after each
`act_mission_get_game()` because the `__fastcall` ECX setup does not survive across calls in the
decompiled output; the handle goes into `script_scratch_u0` and is handed to
`police_board_has_recent_sighting` / `police_manager_target_identity_pending`.
³ Some cases use this slot for unrelated unsigned scratch.
⁴ No non-boolean use observed.

Examples in the reference cases:

| Local | VLVP (`0x122`) | POLICEITCHFORPLAYER (`300`) |
|---|---|---|
| `script_scratch_i1` | `game_get_player_actor()`, tested for non-zero | `game_get_player_actor()`, then `police_manager_count_officers_alive_for_target(...)` |
| `script_pred_c` | `police_board_has_recent_sighting(...)` | `police_board_is_busted_cutscene_active()` (no officer) or `police_manager_target_identity_pending(...) == 0` |
| `script_flt_payload` | `1.0` before `goto script_flt_write` | `1.0` before `goto script_flt_write` |

Every argument-taking case opens with the same shape:

```c
script_scratch_i0 = vec_index_addr_stride_0x8(param_1[7]);
script_argv = *(int **)(script_scratch_i0 + 4);   // arg list
// then: *script_argv = arg0, script_argv[1] = arg1, ...
```

VLVP and POLICEITCHFORPLAYER take a single argument and read only `*script_argv`.

---

## 2. Shared dispatcher tail labels

| Label | Address | Defined in | Jumped to from | Body |
|---|---|---|---|---|
| `script_flt_write` | `0x005c76d4` | case `0x53` GETMISSIONNUMBER | cases `0x122`, `300`, `0x168`, `0x17e`, `0x184` | `*(float *)(param_1[0x12] + script_scratch_i0 * 4) = script_flt_payload;` then break |
| `script_flt_write_zero` | `0x005c76ce` | case `0x184` ISVALIDTAXIPASSENGER | cases `0x122`, `300` | `script_flt_payload = 0.0`, falls into `script_flt_write` |
| `script_vlvp_no_player_zero` | `0x005c4d7e` | case `0x122` VLVP | case `300` | `*(param_1[0x12] + script_scratch_i0 * 4) = 0;` then break |

* **`script_flt_write`** is the shared store of the payload float into the destination `flt[]`
  slot: a single MOV-store followed by the case-end dispatch.
* **`script_flt_write_zero`** is the boolean-false tail of the "is this true?" queries. It sits 6
  bytes before `script_flt_write` and drops into it after loading the float zero.
* **`script_vlvp_no_player_zero`** is the "no active player" tail
  (`game_get_player_actor() == NULL`). It writes a literal 0 directly and skips
  `script_flt_payload` entirely. It lives inside the VLVP body and POLICEITCHFORPLAYER is its only
  other consumer, hence the `vlvp` in the name.

---

## 3. Field accessors on `C_game`

All three are `__fastcall` one-line bodies whose receiver is the **`C_game`** singleton. The receiver
chain at the callsites is byte-level:

```
mov ecx,[0x65115c]            ; g_act_mission
call act_mission_get_game     ; eax = C_game
mov ecx,eax
call game_get_player_actor / game_get_police_block
```

| VA | Name | Body | Callsites | What it returns |
|---|---|---|---|---|
| `0x00425390` | `act_mission_get_game` | `return *(this + 0x24);` | 330+ | the `C_game` singleton; `this` is the act_mission framework root (`*g_act_mission`, `0x0065115c`) |
| `0x00425530` | `game_get_player_actor` | `return *(this + 0xe4);` | 38 ¹ | the player's `C_actor *`, or 0 before the player exists |
| `0x00425560` | `game_get_police_block` | `return this + 0x2da0;` | 26 | the address of the `police_manager` embedded at `C_game+0x2da0` (`structs/police_manager.md` §1) |

¹ 34 of the 38 share the exact receiver chain above, byte-read at `0x005bc438`.

`game_get_player_actor` is the canonical "who is the player" read throughout the script VM and the
AI dispatch, and the source of the `*(world+0xe4) == this` "am I the player?" gate several `C_human`
methods use. In the reference cases a zero return takes the no-player tail.

`game_get_police_block` is address-of, not a dereference: POLICEITCHFORPLAYER calls it to set up
ECX for `police_manager_count_officers_alive_for_target` and
`police_manager_target_identity_pending`.
