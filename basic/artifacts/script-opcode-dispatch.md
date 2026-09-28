# How a script command becomes a `program_process_command` case

**Created** 2026-09-07 22:05 +0200 · **Modified** 2026-09-24 21:39 +0200

This file answers the question *given a script command name, which `case 0xNN:` in
`program_process_command` implements it?* The answer is byte-exact for all 391 commands and
needs no heuristic. **This document is the explanation; the data lives in
`script-mapping/artifacts/opcode-table.md` (generated).**

> 🥇 **Headline: the opcode stored in the game's own keyword table IS the `case 0xNN`
> label.** `case == opcode`, for every one of the 391 commands.

---

## 1. The two halves of the VM

| | function | PC VA | PS2 twin |
|---|---|---|---|
| **compile** | `program_decode_instruction` | `0x005b1b60` | `DecodeInstruction__9C_programFPcRi` |
| **execute** | `program_process_command` | `0x005bb2d0` | `Process__9C_programFUi` |

`program_decode_instruction` turns a command NAME into an integer opcode, which the
compiled script node carries. `program_process_command` switches on that integer.

### 1a. The keyword table

`program_decode_instruction` is **not a lookup into a table — it materialises the table**,
on its own stack, every time it is called:

```c
struct KEYWORD { char name[0x20]; int opcode; };   /* stride 0x24 */
```

```
0x005b1b60  mov  eax, 0x3704          ; 0x3704 = 391 * 0x24 + 8   -> 391 records
0x005b1b65  call __chkstk
            ... ~5500 instructions of UNROLLED initialiser ...
            strncasecmp(input, entry->name, min(strlen(entry->name), strlen(input)))
            return entry->opcode                  ; 0xffffffff on a miss
```

* Names are stored **with a trailing space** and NUL-padded to `0x20`. That trailing space
  is why the scan computes `min(len(entry), len(input))` and adjusts by
  `input[len-1] != ' '` — the stored keyword may be longer than the token being matched.
* The frame size is the census: **`0x3704 = 391 × 0x24 + 8`**, and the search loop bound is
  `index <= 0x186`. Both say 391.
* The matcher is `strncasecmp`, which is why the script language is case-insensitive.
* **The record is `{name, opcode}`, in that order.** The PS2 twin's decompiled body settles it
  in one glance: it computes `strlen(base + i*0x24)`, so the NAME is at offset 0. An opcode
  listing built on the opposite layout reads a window four bytes early and pairs every name
  with the PREVIOUS record's opcode; in such a listing the "raw" number for a command at table
  index *i* is the opcode at index *i−1*.

**Why the function is 35 641 bytes.** MSVC materialises the whole `const` array inline —
`rep movsd` / bare `movsw` / bare `movsb` out of `.data` string slots for each name, and a
`mov [esp+N], imm32` for each opcode. That unrolling *is* the function. The PS2 build
compiles the identical source with GCC, which copies the same array out of `.rodata` with a
loop, and comes to **380 bytes** — a size ratio of 94 on a statement-identical twin.

### 1b. The dispatch

```
0x005bb31e  mov eax, [eax]                     ; the node's opcode
0x005bb320  dec eax
0x005bb321  cmp eax, 0x185
0x005bb326  ja  0x005c76da                     ; the default arm
0x005bb32c  jmp dword ptr [eax*4 + 0x005c7700] ; the jump table
```

**The jump table is at `0x005c7700`**, `0x186` entries of 4 bytes
(`0x005c7700 .. 0x005c7d18`), indexed by `opcode - 1`. It sits immediately after
`program_process_command`'s body, which ends at `0x005c76fe`.

The `-1` bias exists for exactly one reason: **opcode 0 is `LABEL`**, which is a no-op the
compiler consumes, so it has no arm. Ghidra recovers the switch expression as the opcode
itself and therefore prints `case 0xNN` where `NN` is the opcode. Hence:

> **`case == opcode`.** Not `opcode + 1`, not `opcode - 1`.

---

## 2. How the map is extracted

The extraction **emulates the unrolled initialiser** — esp tracking, `.data` loads and the
string primitives — and reads the 391 records back out of the emulated stack frame. It writes
**nothing** unless all six consistency gates pass, and it re-derives the table from the binary
rather than editing the generated document. Two emulation traps decide whether the result is
usable at all: a **bare** `stos` and a **bare** `movs` must not be treated as their `rep`
forms.

---

## 3. Why the extraction is trustworthy

Nine independent checks, none of which was consulted while building the extraction:

| check | result |
|---|---|
| structural | 391 records; opcodes **`0x000`–`0x186` contiguous, 0 gaps, 0 collisions, 0 duplicate names, 391/391 named** |
| `script-mapping/artifacts/opcodes-verified.md` anchors | `VLVP`=`0x122`, `POLICEITCHFORPLAYER`=`0x12c`, `POLICEMANAGER_ON`=`0x86`, `GET_PM_HUMANSTATE`=`0xb2` — 4/4 |
| in-game confirmed anchors | `DAN_INTERNAL_1`=`0x185`, `DAN_INTERNAL_2`=`0x186` — 2/2 |
| the PS2 twin's own name | `M7_ZASTAVZKURVENEPOLISE`=`0x183`, the case `ZastavMotorizovaneJednotky` had already fixed |
| the dispatcher's verified-label docblock | **26 of 27 agree**; the 27th is `case 0xd` (§6) |
| the parser's 13 anonymous strings | they resolve exactly: `DIM_FLT`(7) `LET`(8) `IF`(9) `RND`(0xa) `DIM_FRM`(0xb) `WAIT`(4) `BEEP`(5) `END`(6) `GOTO`(1) `ZATMYSE`(0xe9) `[BREAK]`(0xaa) `{`(0x45) `}`(0x46) |
| the PS2 `.rodata` table | 411 records at file `0x395be0` (VA `0x00495b60`); **385 of the 391 PC names match verbatim** |
| the 6 that do not | exactly `RECCLEAR RECADDACTOR RECLOAD RECLOADFULL RECWAITFOREND RECUNLOAD` — and `structs/recorder.md` independently records `C_recorder` as **PC-only** |
| the function registry's notes | `case 0x171` = `ACTOR_ADDDORICI` against the PS2 callee `AddDoRiciActor`; `case 0x80/0x81` = `AUTOSAVEGAME`/`AUTOSAVEGAMEFULL`; `case 0x148` = `PM_SETPROGRESS` calling `hud_set_police_wanted_bar` |

The bijection is the strongest of these. 391 records producing 391 distinct opcodes that
exactly fill `0x000..0x186` cannot survive an off-by-one, a dropped record or a
mis-modelled instruction — every one of those defects shows up as a gap or a duplicate.

---

## 4. The keyword table is not in opcode order

Only **21 of 391** records have `opcode == table_index`: the developers appended new commands
at the END of the array while giving them opcodes that slot into the middle, and every
out-of-order insertion shifts the positional count for everything after it. Five records carry
an opcode far from their position — `FRM_ISON`(0xf), `HUMAN_GETITEMINRHAND`(0x41),
`GET_PM_HUMANSTATE`(0xb2), `M7_ZASTAVZKURVENEPOLISE`(0x183), `ISVALIDTAXIPASSENGER`(0x184).

⇒ **A command's position in the table carries no information about its opcode**, and the
distance between the two is unbounded by construction. Read the `Opcode` cell; never count
rows.

---

## 5. Which artifact answers which question

| artifact | what it is the authority for |
|---|---|
| `script-mapping/artifacts/opcode-table.md` (and its `.csv`) | 🥇 name ↔ opcode ↔ case, generated by the byte read |
| `script-mapping/artifacts/opcodes-verified.md` | what each command DOES — the curated per-command evidence; its opcode column agrees with the generated table |
| this document | how the mapping is derived, and the mechanisms the table cannot show (§6, §8, §9) |

### 5a. The evidence rule

A byte read of the game's own keyword table does not outrank an anchor confirmed in game, but
it is the **default answer**: it has been checked against every in-game anchor the project
holds (§3). If it ever disagrees with one, that disagreement is a finding — re-run the
extraction and read its gate output before touching the anchor.

---

## 6. Two case facts the table settles

1. **`case 0xd` is `FINDFRAME`, not `FINDACTOR`.** Both bodies read: `case 0xd` writes the
   **frame** register file (`param_1[0x14]`, bounded by `[0x15]`) from an `I3D_scene` vtable
   `+0x58` name lookup, while `case 0x11` writes the **actor** file (bounded by
   `param_1[0x17]`) by walking the actor list comparing `i3d_frame_get_name`.
   `FINDFRAME` = `0xd`, `FINDACTOR` = `0x11`.
2. **`POLICE_SUPPORT` (`0x15e`) is implemented.** Its jump-table slot points at `0x005c64a6`,
   a real arm that reads an argument and bounds-checks the actor register file. The 15 commands
   that genuinely point at the default arm (`0x005c76da`) are listed in the generated table's
   header.

---

## 7. PC vs PS2 — all cross-build material for this subsystem

The PS2 build carries the same table in `.rodata` at **file offset `0x395be0`, VA
`0x00495b60`**, 411 records of the identical `{char name[0x20]; int opcode;}` layout —
which is what makes it a free check on the PC extraction.

* **PC 391 commands, PS2 411.**
* **6 PC-only:** the `REC*` recorder family (`RECCLEAR`, `RECADDACTOR`, `RECLOAD`,
  `RECLOADFULL`, `RECWAITFOREND`, `RECUNLOAD`) — opcodes `0x29`–`0x2e`. The PS2 table leaves
  those six opcode values unused (its gaps are `0x2b`–`0x30`, shifted by the two extra
  commands it has earlier). Consistent with `structs/recorder.md`: `C_recorder` is PC-only.
* **26 PS2-only:** `SETCOMPASS2`, `MISSION_OBJECTIVESCOMPLETE`, `ISPOINTINAREA1`,
  `SOUND_SETVOLUME1`, `FRM_SETON1`, `MATH_ACOS`, `MATH_ASIN`, `MATH_ATAN`,
  `GRABER_SETFOCUS`, `VECT_ANGLETO_2D`, `VECT_ROTATE_2D`, `PLAY_VIDEO`, **`ADDINTERCITYAREA`,
  `INITINTERCITY`**, `HUMAN_DONTFADE`, `GARAGE_VRATA`, `TUTORIAL_OUTTEXT`, `CAR_GETFUEL`,
  `CAR_SETFUEL`, `CAR_CHANGEMODEL`, `GARAGE_FREERIDE`, `HUMAN_ISCROUCH`, `CAMERA_HARDLOCK`,
  `CAMERA_HARDUNLOCK`, `CAR_SETNEUTRAL`, `HUMAN_CROUCH`.
  The two `INTERCITY` commands are the script-side face of the PS2-only city-streaming
  subsystem — a third independent corroboration that it is PS2-port-only.
* ⚠️ **The two builds' opcode NUMBERS diverge.** They agree up to `SETCOMPASS`/`SETCOMPASS2`
  and drift thereafter (PS2 max `0x19c` vs PC `0x186`). A PS2 case number is never a PC case
  number; use the PC table for PC work, and locate a PS2 arm by byte-reading the PS2 keyword
  table rather than by translating a PC case.
* The PS2 build **inlines** the §8 accessors into its own `Process__9C_programFUi` arm, so the
  PS2 arm for a command is a second body for every helper that arm calls.

---

## 8. Appendix — the `[0x005c7da0, 0x005c9740)` helper tail, by command

126 function starts. **112 are reached ONLY from `program_process_command`**, which is what
makes this range hard to name from behaviour alone (`structs/program.md` §6): a one-line
accessor tells nothing by itself, and its single consumer is a 50 KB switch. With the case map
the consumer is readable, and the PS2 build's inlined arm (§7) supplies the second body.

Each callsite is attributed to an arm by taking the greatest jump-table arm start ≤ the
callsite VA. ⚠️ **This is a strong lead, not a proof:** arms share tails and fall through, so
a callsite in a shared tail is attributed to whichever arm starts last before it. A row whose
several commands form a coherent family (the `CARLIGHT_*` block, the `POLICEMANAGER_*` block)
is self-corroborating; a lone incoherent pair is a tell that a shared tail was crossed. Where
the attribution is taken from the decompiler's own `case` labels instead, note that **Ghidra
prints 50 of the 375 case labels in DECIMAL** (`case 199:`), so a `case 0x…` pattern silently
attributes those callsites to the previous label.

The names are the validation: every row lands on the command its name predicts —
`hud_set_police_wanted_bar` ← `PM_SETPROGRESS`, `police_manager_is_enabled` ← the three
`PM_*` commands that the dispatcher docblock says gate on it, `matrix4_zero` ← `MATRIX_ZERO`,
`i3d_frame_is_on` ← `FRM_ISON`, `i3d_scene_get_active_camera_ptr` ← `GETACTIVECAMERA`,
`cutscene_set_active` ← `RECLOADFULL`/`RECUNLOAD`.

| PC VA | Size | Name | Callsites in the dispatcher | Command(s) whose arm calls it |
|---|---:|---|---:|---|
| `0x005c7da0` | 13 | `hud_set_element_flags` | 1 (+1 outside) | `0x2c` RECLOADFULL |
| `0x005c7db0` | 23 | `hud_hide_element` | 9 | `0x1d` SETCOMPASS, `0x32` TIMEROFF, `0x4d` DIALOG_BEGIN, `0x7a` WINGMAN_DELINDICATOR, `0xea` CARDAMAGEVISIBLE, `0x146` PM_SHOWSYMBOL |
| `0x005c7dd0` | 18 | `hud_element_is_visible` | 2 | `0x16c` TIMER_GETINTERVAL, `0x16d` TIMER_SETINTERVAL |
| `0x005c7df0` | 438 | `hud_push_element_flags` | 1 | `0x2c` RECLOADFULL |
| `0x005c7fb0` | 96 | `hud_pop_element_flags` | 1 | `0x2e` RECUNLOAD |
| `0x005c8010` | 13 | `hud_set_police_wanted_bar` | 2 | `0x148` PM_SETPROGRESS |
| `0x005c8020` | 14 | `matrix4_zero` | 1 | `0x10e` MATRIX_ZERO |
| `0x005c8030` | 41 | `matrix4_invert_in_place` | 3 | `0x33` CHANGE_MISSION, `0x111` MATRIX_INVERSE |
| `0x005c8060` | 77 | `quat_add` | 1 | `0x119` QUAT_ADD |
| `0x005c80b0` | 76 | `quat_sub_w_add_xyz` | 1 | `0x11a` QUAT_SUB |
| `0x005c8100` | 230 | `quat_multiply_in` | 1 | `0x11c` QUAT_MUL_QUAT |
| `0x005c81f0` | 78 | `quat_scale_to` | 3 | `0x117` QUAT_INVERSE, `0x11b` QUAT_MUL_SCL |
| `0x005c8240` | 36 | `quat_dot` | 1 | `0x11d` QUAT_DOT |
| `0x005c8270` | 7 | `racing_get_racer_array_ptr` | 2 | `0x141` RACING_CHANGE_MODEL |
| `0x005c8280` | 7 | `racing_get_racer_count` | 1 | `0x141` RACING_CHANGE_MODEL |
| `0x005c8290` | 4 | `cache_base_block_is_loaded` | 1 (+1 outside) | `0x142` CITYCACHING_TICK |
| `0x005c82a0` | 7 | `i3d_frame_get_type_ptr` | 25 (+16 outside) | `0xe` FRM_SETON, `0x16` PLAYSOUNDEX, `0x17` STOPSOUND, `0x18` GETSOUNDTIME, `0x1e` SETLMLEVEL, `0x23` SOUNDFADE, `0x55` HUMAN_LINKTOHAND, `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME, `0x7b` MODEL_PLAYANIM, `0x7c` MODEL_STOPANIM, `0xac` SETMODELTOCAR, `0xb5` STREAM_CONNECT, `0xda` ACTOR_DUPLICATE, `0xf6` FRM_SETPOS, `0x13c` ISPOINTINSECTOR, `0x13d` ISPOINTINAREA, `0x13f` SOUND_GETVOLUME, `0x140` SOUND_SETVOLUME, `0x152` FRM_SETALPHA |
| `0x005c82b0` | 65 | `i3d_frame_set_position_xyz_ptrs` | 2 | `0x55` HUMAN_LINKTOHAND, `0x185` DAN_INTERNAL_1 |
| `0x005c8300` | 83 | `i3d_frame_set_scale` | 3 (+1 outside) | `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME, `0xf8` FRM_SETSCALE |
| `0x005c8360` | 7 | `folded_get_ptr_0x90` | 4 | `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME, `0x70` EMITPARTICLE, `0xf7` FRM_GETSCALE |
| `0x005c8370` | 45 | `i3d_frame_get_rotation_ptr` | 5 | `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME, `0xda` ACTOR_DUPLICATE, `0xf9` FRM_GETROT, `0x16b` HUMAN_RETURNTOTRAFF |
| `0x005c83a0` | 74 | `i3d_frame_get_world_rotation_quat` | 1 | `0xfe` FRM_GETWORLDROT |
| `0x005c83f0` | 275 | `i3d_frame_get_scaled_rot_matrix_ptr` | 1 | `0xff` FRM_GETLOCALMATRIX |
| `0x005c8510` | 10 | `i3d_frame_is_on` | 2 | `0xf` FRM_ISON, `0x23` SOUNDFADE |
| `0x005c8520` | 16 | `i3d_frame_get_name` | 7 (+1 outside) | `0x11` FINDACTOR, `0x87` POLICEMANAGER_ADD, `0x88` POLICEMANAGER_DEL, `0xda` ACTOR_DUPLICATE |
| `0x005c8530` | 7 | `i3d_frame_get_child_count_ptr` | 2 (+2 outside) | `0x16e` FRM_GETNUMCHILDREN, `0x16f` FRM_GETCHILD |
| `0x005c8540` | 10 | `i3d_animation_set_set_default_options` | 1 | `0x7b` MODEL_PLAYANIM |
| `0x005c8550` | 4 | `i3d_face_group_get_material_ptr` | 1 | `0xb5` STREAM_CONNECT |
| `0x005c8560` | 4 | `i3d_mesh_level_get_face_group_count_ptr` | 1 | `0xb5` STREAM_CONNECT |
| `0x005c8570` | 4 | `i3d_mesh_object_get_lod_count_ptr` | 1 | `0xb5` STREAM_CONNECT |
| `0x005c8580` | 7 | `i3d_frame_get_tag_ptr` | 3 (+3 outside) | `0x1e` SETLMLEVEL, `0xb5` STREAM_CONNECT, `0x152` FRM_SETALPHA |
| `0x005c8590` | 7 | `i3d_frame_get_mesh_object_ptr` | 1 (+1 outside) | `0xb5` STREAM_CONNECT |
| `0x005c85a0` | 7 | `i3d_camera_get_fov_ptr` | 1 | `0x161` CAMERA_GETFOV |
| `0x005c85b0` | 25 | `i3d_frame_get_lmap_entry_ptr` | 1 | `0x1e` SETLMLEVEL |
| `0x005c85d0` | 16 | `i3d_frame_get_model_name` | 1 | `0xda` ACTOR_DUPLICATE |
| `0x005c85e0` | 7 | `i3d_scene_get_active_camera_ptr` | 6 | `0x24` ENDOFMISSION, `0xd9` CAMERA_SETRANGE, `0x13e` GETACTIVECAMERA, `0x160` CAMERA_SETFOV, `0x161` CAMERA_GETFOV |
| `0x005c85f0` | 7 | `i3d_scene_get_root_frame_ptr` | 3 | `0x10` FRM_LINKTO, `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME |
| `0x005c8600` | 43 | `i3d_scene_set_frame_sector_pos` | 7 (+1 outside) | `0x10` FRM_LINKTO, `0x56` HUMAN_UNLINKFROMHAND, `0x57` CREATEWEAPONFROMFRAME, `0x6c` ACTOR_SETPOS, `0x6d` ACTOR_SETPLACEMENT, `0xf6` FRM_SETPOS, `0x185` DAN_INTERNAL_1 |
| `0x005c8630` | 7 | `i3d_sound_get_volume_ptr` | 1 | `0x13f` SOUND_GETVOLUME |
| `0x005c8640` | 4 | `mission_get_actor_list` | 3 | `0x11` FINDACTOR, `0x87` POLICEMANAGER_ADD, `0x88` POLICEMANAGER_DEL |
| `0x005c8650` | 4 | `mission_get_particle_manager` | 2 | `0x70` EMITPARTICLE, `0x71` STOPPARTICLE |
| `0x005c8660` | 10 | `mission_set_next_mission_number` | 1 | `0x54` SETMISSIONNUMBER |
| `0x005c8670` | 4 | `mission_get_mission_number` | 1 | `0x53` GETMISSIONNUMBER |
| `0x005c8680` | 14 | `mission_commit_next_mission_number` | 1 | `0x33` CHANGE_MISSION |
| `0x005c8690` | 10 | `mission_set_sound_restore_stage` | 2 (+1 outside) | `0x2c` RECLOADFULL, `0x2e` RECUNLOAD |
| `0x005c86a0` | 4 | `mission_get_sound_restore_stage` | 2 | `0x2c` RECLOADFULL, `0x2e` RECUNLOAD |
| `0x005c86b0` | 34 | `using_object_record_ctor` | 1 | `0x57` CREATEWEAPONFROMFRAME |
| `0x005c86e0` | 4 | `actor_get_class_id` | 1 | `0x14c` HUMAN_ENTERTOTRUCK |
| `0x005c86f0` | 26 | `actor_set_cached_world_pos` | 3 | `0x6c` ACTOR_SETPOS, `0x6d` ACTOR_SETPLACEMENT, `0x126` ACTORUPDATEPLACEMENT |
| `0x005c8710` | 30 | `actor_get_direction` | 2 | `0x6b` ACTOR_SETDIR, `0x6d` ACTOR_SETPLACEMENT |
| `0x005c8730` | 10 | `actor_mark_created_at_runtime` | 1 | `0xda` ACTOR_DUPLICATE |
| `0x005c8740` | 10 | `vehicle_controller_set_field_0x65` | 1 | `0x185` DAN_INTERNAL_1 |
| `0x005c8750` | 6 | `actor_get_global_list` | 3 | `0x73` FINDNEARACTOR, `0xcd` DESTROY_PHYSICALOBJECT, `0x186` DAN_INTERNAL_2 |
| `0x005c8760` | 10 | `program_set_ai_priority` | 1 | `0x47` SETAIPRIORITY |
| `0x005c8770` | 13 | `loadsave_set_mission_name_id` | 1 | `0x78` SETMISSIONNAMEID |
| `0x005c8780` | 7 | `vehicle_get_special_dynamic_colls` | 3 | `0xa1` CAR_SETSPECCOLL, `0xa2` CAR_GETSPECCOLL |
| `0x005c8790` | 163 | `vehicle_set_velocity` | 1 | `0xa1` CAR_SETSPECCOLL |
| `0x005c8840` | 49 | `vehicle_set_angular_velocity` | 1 | `0xa1` CAR_SETSPECCOLL |
| `0x005c8880` | 36 | `vehicle_set_auto_indicators_from_steering` | 8 | `0xc2` CARLIGHT_INDIC_L, `0xc3` CARLIGHT_INDIC_R, `0xc4` CARLIGHT_INDIC_OFF |
| `0x005c88b0` | 47 | `vehicle_set_light_bit_0x80` | 3 | `0xc1` CARLIGHT_LIGHT |
| `0x005c88e0` | 58 | `vehicle_set_left_indicator` | 3 | `0xc2` CARLIGHT_INDIC_L, `0xc4` CARLIGHT_INDIC_OFF |
| `0x005c8920` | 58 | `vehicle_set_right_indicator` | 3 | `0xc3` CARLIGHT_INDIC_R, `0xc4` CARLIGHT_INDIC_OFF |
| `0x005c8960` | 53 | `vehicle_set_headlights` | 3 | `0xc0` CARLIGHT_MAIN |
| `0x005c89a0` | 52 | `vehicle_set_light_script_override` | 7 | `0xc0` CARLIGHT_MAIN, `0xc1` CARLIGHT_LIGHT, `0xc2` CARLIGHT_INDIC_L, `0xc3` CARLIGHT_INDIC_R |
| `0x005c89e0` | 12 | `vehicle_get_light_script_override` | 4 | `0xc0` CARLIGHT_MAIN, `0xc1` CARLIGHT_LIGHT, `0xc2` CARLIGHT_INDIC_L, `0xc3` CARLIGHT_INDIC_R |
| `0x005c89f0` | 19 | `car_set_tank_hit_count` | 1 | `0xbe` SETTANKHITCOUNT |
| `0x005c8a10` | 13 | `car_set_must_steal` | 1 | `0xc5` CAR_MUSTSTEAL |
| `0x005c8a20` | 13 | `car_set_destroy_motor` | 1 | `0x6a` CAR_SETDESTROYMOTOR |
| `0x005c8a30` | 13 | `car_set_broken_motor` | 1 | `0x95` CAR_BREAKMOTOR |
| `0x005c8a40` | 7 | `car_get_model_name_ptr` | 3 | `0x5a` GARAGE_DELCAR, `0xc7` GARAGE_CANSTEAL, `0x177` GETCARLINENUMFROMTABLE |
| `0x005c8a50` | 13 | `car_set_show_damage` | 1 | `0xd5` SHOWCARDAMAGE |
| `0x005c8a60` | 34 | `car_adjust_brake_hold_count` | 1 | `0x144` CAR_LOCK |
| `0x005c8a90` | 13 | `vehicle_controller_set_field_0x214c` | 1 | `0x185` DAN_INTERNAL_1 |
| `0x005c8aa0` | 13 | `car_set_auto_headlights_suppressed` | 2 | `0xc0` CARLIGHT_MAIN |
| `0x005c8ab0` | 13 | `car_set_effect_mask` | 1 | `0x182` CAR_SETAXIS |
| `0x005c8ac0` | 13 | `car_set_damage_inhibit_mask` | 1 | `0x156` CAR_UNBREAKABLE |
| `0x005c8ad0` | 7 | `cutscene_set_active` | 2 | `0x2c` RECLOADFULL, `0x2e` RECUNLOAD |
| `0x005c8ae0` | 4 | `camera_get_car` | 1 | `0x93` CAR_SETSPEED |
| `0x005c8af0` | 3 | `police_manager_is_enabled` | 3 | `0x146` PM_SHOWSYMBOL, `0x147` PM_SHOWPROGRESS, `0x148` PM_SETPROGRESS |
| `0x005c8b00` | 17 | `police_manager_set_speed_thresholds` | 1 | `0x89` POLICEMANAGER_SETSPEED |
| `0x005c8b20` | 7 | `game_get_using_object` | 1 | `0x57` CREATEWEAPONFROMFRAME |
| `0x005c8b30` | 8 | `game_request_clear_differences` | 1 | `0x44` CLEARDIFFERENCES |
| `0x005c8b40` | 7 | `game_get_pursuit_event_manager` | 2 | `0xab` CAR_REMOVE_DRIVER, `0x17b` GROUP_DISBAND |
| `0x005c8b50` | 7 | `game_get_frame_count` | 1 | `0x76` CAMERA_LOCK |
| `0x005c8b60` | 13 | `game_set_night_mission` | 1 | `0x50` GAME_NIGHTMISSION |
| `0x005c8b70` | 23 | `game_set_timer_timeout_event` | 1 | `0x6f` SETTIMEOUTEVENT |
| `0x005c8b90` | 23 | `game_set_player_fire_event` | 1 | `0x72` SETPLAYERFIREEVENT |
| `0x005c8bb0` | 23 | `game_set_player_horn_event` | 1 | `0x74` SETPLAYERHORNEVENT |
| `0x005c8bd0` | 23 | `game_set_player_water_fall_event` | 1 | `0x128` SETPLAYERWFALLEVENT |
| `0x005c8bf0` | 10 | `game_set_mission_end_action` | 2 | `0x24` ENDOFMISSION |
| `0x005c8c00` | 20 | `game_set_race_state` | 1 | `0xa7` RACING_MISSION6_START |
| `0x005c8c20` | 7 | `game_get_race_state` | 2 | `0xa7` RACING_MISSION6_START |
| `0x005c8c30` | 13 | `game_set_race_result_register` | 1 | `0xa7` RACING_MISSION6_START |
| `0x005c8c40` | 13 | `game_set_actor_lightness_flag` | 1 | `0x143` ACTORLIGHTNESSON |
| `0x005c8c50` | 13 | `game_set_heavy_init_enabled` | 1 | `0x12d` NOANIMPRELOAD |
| `0x005c8c60` | 13 | `game_set_rain_buffer_size` | 1 | `0x158` WEATHER_PREPAREBUFFER |
| `0x005c8c70` | 13 | `game_set_script_mission_end` | 2 | `0x24` ENDOFMISSION, `0x33` CHANGE_MISSION |
| `0x005c8c80` | 29 | `car_extended_can_board` | 1 | `0x14c` HUMAN_ENTERTOTRUCK |
| `0x005c8ca0` | 23 | `wagon_set_event` | 1 | `0x9f` WAGON_SETEVENT |
| `0x005c8cc0` | 61 | `wagon_get_last_stop_node_id` | 1 | `0xa0` WAGON_GETLASTNODE |
| `0x005c8d00` | 7 | `detector_get_signal` | 1 | `0x1a` DETECTOR_ISSIGNAL |
| `0x005c8d10` | 17 | `detector_set_signal` | 1 | `0x1b` DETECTOR_SETSIGNAL |
| `0x005c8d30` | 7 | `pedestrian_manager_set_scream_enabled` | 1 | `0x175` NPC_SHUTUP |
| `0x005c8d40` | 13 | `human_pose_set_look_target_frame` | 2 | `0x51` HUMAN_LOOKTOACTOR, `0x52` HUMAN_LOOKTOFRAME |
| `0x005c8d50` | 23 | `player_set_wingman_indicator` | 2 | `0x79` WINGMAN_SETINDICATOR, `0x7a` WINGMAN_DELINDICATOR |
| `0x005c8d70` | 13 | `human_set_fire_target_frame` | 2 | `0x8d` HUMAN_SETFIRETARGET |
| `0x005c8d80` | 13 | `human_set_no_hit_anim` | 1 | `0xbf` SETNOANIMHIT |
| `0x005c8d90` | 13 | `human_set_idle_anims_enabled` | 4 | `0x2c` RECLOADFULL, `0x2e` RECUNLOAD, `0x4d` DIALOG_BEGIN, `0x4e` DIALOG_END |
| `0x005c8da0` | 13 | `human_set_silent` | 1 | `0x179` HUMAN_SHUTUP |
| `0x005c8db0` | 7 | `hud_set_car_damage_bar_enabled` | 1 | `0xea` CARDAMAGEVISIBLE |
| `0x005c8dc0` | 424 | `ai_groups_add_tracked_actor` | 3 | `0x17b` GROUP_DISBAND |
| `0x005c8f70` | 47 | `ai_logs_clear_all_logs` | 1 | `0x17b` GROUP_DISBAND |
| `0x005c8fa0` | 58 | `entity_get_ai_state_base_code` | 1 | `0xad` GETENEMYAISTATE |
| `0x005c8fe0` | 36 | `entity_set_ai_state_resettable` | 1 | `0x91` HUMAN_RETURNFROMPANIC |
| `0x005c9010` | 36 | `entity_set_state_switcher_suspended` | 2 | `0x17b` GROUP_DISBAND |
| `0x005c9040` | 27 | `entity_unregister_using_object` | 2 | `0x84` HUMAN_ERASEAB, `0x85` HUMAN_CREATEAB |
| `0x005c9060` | 98 | `program_switch_light_map` | 1 (+1 outside) | `0x1e` SETLMLEVEL |
| `0x005c90d0` | 446 | `std_vector8_push_back` | 2 (+1 outside) | `0x12` CALL |
| `0x005c9290` | 40 | `program_instructions_clear` | 1 | `0x12` CALL |
| `0x005c92c0` | 7 | `program_return_stack_back` | 3 | `0x3` RETURN, `0x4a` EVENT_USE_CB |
| `0x005c92d0` | 422 | `program_return_stack_push` | 1 | `0x2` GOSUB |
| `0x005c9480` | 43 | `program_return_stack_pop` | 1 | `0x3` RETURN |
| `0x005c94b0` | 575 | `std_vector8_insert_at` | 0 (+48 outside) | - |
| `0x005c96f0` | 18 | `stl_max_float_ptr` | 1 | `0xcc` CREATE_PHYSICALOBJECT |
| `0x005c9710` | 15 | `stl_max_int_ptr` | 3 | `0x65` HUMAN_SETPROPERTY, `0x79` WINGMAN_SETINDICATOR |
| `0x005c9720` | 25 | `crt_static_init_guard_005c9720` | 0 | - |

### 8a. What the command map settles

| VA | command | what it settles |
|---|---|---|
| `0x005c8550` / `_8560` / `_8570` | `STREAM_CONNECT` | `i3d_face_group_get_material_ptr`, `i3d_mesh_level_get_face_group_count_ptr`, `i3d_mesh_object_get_lod_count_ptr`. The PS2 arm for this command is an empty stub, so the second body comes from `glass_shatter_cut_mesh`, which walks the identical chain and whose twin is CONFIRMED (`structs/i3d_frame.md` §9) |
| `0x005c86b0` | `CREATEWEAPONFROMFRAME` | the `operator_new(0x38)` constructor's owner class: `using_object_record_ctor` |
| `0x005c9060` | `SETLMLEVEL` | the `'LMAP'` frame-tree walk: `program_switch_light_map` |
| `0x005c8fe0` | `HUMAN_RETURNFROMPANIC` | `entity_set_ai_state_resettable`, whose noun had no consumer behind it |
| `i3d_frame_get_lmap_entry_ptr` | `SETLMLEVEL` | its noun, now backed by a consumer |
| `0x005c8740` / `0x005c8a90` | `DAN_INTERNAL_1` | the two writes that seat the player in the spawned Thunderbird |
| `0x005c92d0` / `_9480` / `_90d0` / `_9290` | `GOSUB` / `RETURN` / `CALL` | the script call-stack machinery |
| `0x005c8880`..`0x005c89e0` (7 fns) | `CARLIGHT_*` | a complete, coherent car-lighting family |

Two rows are **not** dispatcher helpers: `std_vector8_insert_at` (`0x005c94b0`, 48 callers,
all outside — a genuinely shared container helper) and `crt_static_init_guard_005c9720`, which
has no dispatcher callsite and belongs to the next translation unit.

---

## 9. The base dispatcher is not the only dispatcher

`Process__9C_programFUi` is **virtual**. `C_detector` overrides it, and
`detector_program_process` handles six opcodes itself before delegating everything else
to the base — which is why `DETECTOR_WAITFORUSE` and `DETECTOR_WAITFORHIT` work in the 30+
shipped scripts that use them while the base jump table sends their opcodes to the default arm:

| opcode | command | what the override does |
|---|---|---|
| `0x1c` | `DETECTOR_INRANGE` | counts matching actors in range, writes the count to a float register, advances |
| `0x1f` | `DETECTOR_WAITFORUSE` | releases the previous using-object, calls `detector_create_using_object`, **does not advance** |
| `0x20` | `DETECTOR_WAITFORHIT` | sets `+0x6e`, act-state 1, erase + create colls, **does not advance** |
| `0x25` | `PROGRAM_STORAGE` | parks the actor in act-state 2 while `+0x40` is 0 |
| `0x154` | `DETECTOR_CREATEDYNCOLL` | sets `+0x6f`, erase + create colls in mode 1 |
| `0x155` | `DETECTOR_ERASEDYNCOLL` | clears `+0x6f`, erase colls |

🔑 **How the wait works, and why no jump table can show it.** The base default arm at
`0x005c76da` is `push 0; mov ecx,ebp; call program_next_command` — a genuine no-op that
advances the program counter. The two `WAITFOR*` cases above **omit that call**, so the
program counter stays put and the script re-runs the same instruction every tick until
`detector_use` / `detector_hit` releases it. **Blocking is the ABSENCE of an advance.**

⚠️ Their base-dispatcher slot points at the default arm *precisely because* the derived
class owns them — the empty slot is the consequence of the override, not evidence of a
missing feature. `structs/detector.md` §1 carries five of these six mappings from the
`C_detector` side. The extraction carries the six-opcode map, so the generated table reports
**9** genuinely unimplemented commands, not 15.

**How to find the other overrides of any dispatcher:** grep the decomp for the functions
that TAIL INTO it. There are exactly two here — this one and `entity_process_command`,
which intercepts only the `0x1000+` AI-action instruction space and therefore adds no
script command.

### 9a. A default-arm slot means three different things

Whether a command works depends on a second dimension the jump table does not carry: whether
**`program_decode_params`**, the load-time argument decoder, has an arm for the opcode.
Measured over its outer switch, **384 of the 391 opcodes have one**, including a shared
`return 1` arm for zero-argument commands such as `END` and `RETURN` — so a missing arm is not
an arity artefact. The switch carries **no `default:` label**, so an unmatched opcode falls
through to `return 0`, a decode failure at script load.

| group | commands | what actually happens |
|---|---|---|
| **implemented — a MARKER, not a command** (1) | `EVENT` (`0x48`) | Its parser arm resolves the event NAME to an id (`use_AB` → 60000, `death` → 60001, `enemy_to_AI` → 60002, `enemy_to_Script` → …, else `atoi`) and emits an instruction of **type `0x48`**. `program_set_event` (PS2 `SetEvent__9C_programFi`) scans the instruction vector for `type == 0x48` carrying that id and moves the program counter there. So `event` is a **jump-target label**, exactly like `label` — which is why shipped scripts write it after `end` — and the default arm, a no-op that advances the pc, is the CORRECT fall-through. Matches the in-game report that `event` works |
| **compiles, then silently ignored** (1) | `HUMAN_CANDIE` (`0x15a`) | Has a parser arm (one `atoi` integer, in the large shared group) and no dispatcher arm. No error, no effect — a genuinely dead command that 16 shipped scripts still use |
| **fails to COMPILE** (7) | `BEEP` (`0x005`), `DEBUG_TEXT` (`0x037`), `[BREAK]` (`0x0aa`), `[DEBUG_ME]` (`0x0eb`), `[DEBUG_STOP]` (`0x0ec`), `DEBUG_GETFRAMEINFO` (`0x12b`), `[DAN_DEBUG]` (`0x13a`) | No parser arm at all ⇒ `program_decode_params` returns 0 at load time. `DEBUG_TEXT` reproduces this in game as a compilation error, which is what validates the split. The other six are the developers' in-house debugger vocabulary, kept in the keyword table and stripped from both the parser and the VM |

🔑 **`EVENT`'s opcode IS the instruction type.** The compiler stores a command's opcode AS the
instruction type, so the marker `program_set_event` scans for *is* the `EVENT` command — which
joins this document's `EVENT = opcode 0x48` to `structs/program.md` §5d's event marker at
instruction type `0x48`.

The extraction carries all three groups, plus a gate that fails if a hand-classified opcode
stops being a default-arm case, so a re-extraction cannot leave a stale note behind. That gate
was exercised on a deliberately broken input before being relied on.

---

## 10. Related

`script-mapping/artifacts/opcode-table.md` (the data) ·
`script-mapping/artifacts/opcodes-verified.md` (what each command does) ·
`structs/program.md` (the `C_program` class) · `structs/detector.md` (the second dispatcher) ·
the 21 shipped mission scripts in source form · the Mafia Script Data Reference (the shipped
designer reference).

---

## 11. Open items

| item | state |
|---|---|
| the nearest-arm attribution in §8 | a lead, not a proof, for any row whose commands do not form a coherent family; arms share tails |
| `BEEP` | predicted to fail compilation exactly as `DEBUG_TEXT` does (§9a); not tried in game |
| the 26 PS2-only commands (§7) | whether any corresponds to a visible PS2 feature the PC lacks is untested; `HUMAN_CROUCH` / `HUMAN_ISCROUCH` and `CAR_GETFUEL` / `CAR_SETFUEL` are the interesting pairs |
