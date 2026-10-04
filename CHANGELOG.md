# Changelog

One section per export version (date tag). Counts are measured on the annotated decompilation
`ghidra/Game-ghidra.exe.annotated.c` against Ghidra's default auto-analysis of `Game.exe` 1.0.

## 2026.10.04 — open-item review, name corrections, Ghidra names re-imported

- 334 tracked open questions closed (18 review batches plus the Ghidra name import); the answers
  and their evidence are in the docs and docblocks listed below.
- Function renames (28 functions + 9 thunks; each docblock and registry row rewritten):
  - static initialisers named after the object they build: `crt_init_dir8_table_normalized_006550d8`
    → `dir8_table_normalized_static_init`, `crt_init_dir8_table_unnormalized_00655138` →
    `dir8_table_static_init`, `crt_init_dir8_table_normalized_006551a8` →
    `dir8_table_normalized_006551a8_static_init`, `dir8_unit_xz_006f96c0_static_ctor` /
    `dir8_square_xz_006f9590_static_ctor` / `dir16_square_xz_006f95f0_static_ctor` → `…_static_init`,
    `crt_init_vec_up_00655198` / `crt_init_vec_up_00655208` → `vec_up_00655198_static_init` /
    `vec_up_00655208_static_init`, `g_vec3_zero_static_init` → `vec3_zero_static_init`; each thunk
    follows its target;
  - network payloads joined to their family: `door_event_payload_{ctor,init}` →
    `event_msg_door_state_payload_{ctor,init}`, `dropout_event_payload_{ctor,init,dtor}` →
    `event_msg_car_part_dropout_payload_{ctor,init,dtor}`;
  - misnomers: `game_get_police_block` → `game_get_police_manager`,
    `police_board_has_pursuit_group_for_target` → `police_board_has_group_for_target`,
    `entity_get_ai_substate` → `entity_get_human_type_group`, `entity_expire_ghost_trail` →
    `entity_tick_ghosts`, `ai_path_pass_side_at` → `blocked_con_vector_at`, `dog_spawn_blood_drop` →
    `dog_place_bloody_paw_print`, `dog_arm_bleeding` → `dog_arm_bloody_paw_prints`,
    `human_cb_coll_tester` → `human_arm_bloody_footprints`, `car_database_get_car_variant_name_id` →
    `car_database_get_color_name_id`, `garage_enable_start_for_letter` →
    `loadsave_freeride_enable_car_group`;
  - `C_chunk`: `chunk_close` (`0x004be720`) → `chunk_dtor`, and `chunk_close_handle` (`0x00544590`)
    → `chunk_close`;
  - runtime: `stub_noop_cdecl_006248fb` → `crt_fpclear`, `rand` → `crt_rand`.
- Data names: `g_crt_fdiv_exponent_class_table` → `g_crt_fprem_flawed_divisor_nibbles`,
  `PTR___exit_00651c18` → `g_crt_aexit_rtn`, `s_Velka_0064b8b8` / `s_Sekundova_0064b8ac` →
  `s_clock_hand_minute` / `s_clock_hand_second`; new `g_crt_fp_mt_init_hook`,
  `g_crt_fp_mt_term_hook`, `g_crt_pow10_positive`, `g_crt_pow10_negative`; `g_crt_mbcs_case_ranges`
  typed as `ushort[6]` (its two split-dword names retired).
- PS2 twins: 2,651 → 2,590 functions mapped. The registry counts a twin only when it leads its
  cell, which removes 60 false twins. New CONFIRMED twins for `police_board_suspend_npcs` /
  `police_board_resume_npcs`, `entity_tick_ghosts` and `loadsave_freeride_enable_car_group`;
  `entity_suspend_ai_and_hide` / `entity_resume_ai_and_show` re-paired; PS2 symbols claimed by two
  PC functions resolved: `i3d_{model,sound,anim}_cache_dtor` twin the derived PS2 destructors
  (`__dt__17C_I3D_model_cacheFv` …) and the `C_I3D_cache<T>` symbols stay on `i3d_cache_*_dtor`;
  `particle_type_*` are per-record halves of `C_ParticleEmitter` methods (PS2 `C_ParticleType`
  has no methods); `game_city_music_off` graded STRUCTURAL; the four notice-panel functions are
  PC-only.
- Annotated decompilation:
  - the 14 bodies that call `crt_chkstk` re-spliced: they still had the stack model from before
    Ghidra's `__chkstk` fixup (`mission_open_bin` was 171 of 2,508 lines, its chunk loop folded
    away; also `program_decode_instruction` and `entity_script_decode_instruction`);
  - comments corrected where an object was misread: `police_manager` lives at `C_game + 0x2da0`
    (not on the actor); the prose "pursuit group" now names the object meant (the `police_action`
    squad, the `pursuit_record` vector or an `ai_group`); `entity+0xf4c` is the human type group and
    `entity+0x11ac` the `ai_group` id; the `ai_talkman_say` arguments are talk groups and timings;
  - inline `/* */` comments converted to `//`; comments state current understanding only (no
    dates, no name-to-name history, no references to working notes).
- Documentation: 126 docs updated; `structs/pursuit_group_manager.md` (a redirect page) removed, so
  114 struct docs. The docs were brought to one convention: open items in a single section at the
  end, uniform tables, a Created / Modified header, current understanding only (no retracted
  claims or rename history, no references to working notes). New findings include the
  police offence `1 << zz` clamp, the despawn list and the reinforcement waves (`police-*`,
  `offences.md`); the `C_actor` PC↔PS2 field reconciliation; the `ai_logs` relationship matrix, the
  `ai_talkman` talk groups, ghosts as collision exemptions and a PC-only defect in
  `ai_follow_manager_load`; airplane instances, the drawbridges (West and East Marshall Bridge),
  blood decals, camera modes, car engine fields and the `carindex.def` field map, the chunk-id
  vocabulary, the runtime tail, the `particles.def` layout, the detector / dog / door / driver /
  dropout / entity fields, fake shadows, fire, force feedback, `C_game`, lightness, the garage
  screens, glass shatter, grenade, the GUI translation unit, the HUD radar and map textures (bound by
  `hud_render_all`), and the `C_human` weapon class and shotgun reload.
- `ghidra/Game-ghidra.exe.functions.md`: three columns after the row number (VA · name · PS2 twin);
  the Ghidra-name column is gone. `ghidra/SLUS_206.71.functions.md`: header reworded.
- Ghidra database `ghidra/Game.exe.gzf`: the `.i3d` literal at `0x006517a0` defined; `0x006f9af0`
  typed as `ushort[6]`; the 37 function names and 8 data labels above imported, so a fresh Ghidra
  export now pairs with every annotated function.
- README: the cleanup row now carries the measured count, Ghidra `WARNING` lines 2,688 → 0.

## 2026.09.28.1 — docblock and layout corrections

- Annotated decompilation, comments only (no code or name changes):
  - the one remaining stacked docblock (`ai_sensors_get_heard_sound_info`) merged into a single
    block; its `out_extra` source (the contact slot's leading dword) and caller count (6) corrected;
  - 101 duplicated `Renamed from FUN_xxxxxxxx` provenance lines removed (each function keeps one);
  - 9 comments that cited internal working notes rewritten as plain statements.
- `ai_space_coverage` layout conflict resolved from the machine code:
  `ai_check_builder_test_space_8dir` writes ONE float through its out-parameter (the mean of the
  8 clearances minus 0.25) and keeps the clearances local, so `ai_space_coverage` is `0x5c` bytes
  and `+0x54` is that threshold. The returned permit mask sets bit *i* when clearance *i* is at
  least the threshold. Updated: both functions' docblocks, `structs/ai_space_coverage.md` §1a,
  `structs/ai_check.md` §1a.

## 2026.09.28 — initial export

- Annotated PC decompilation: 5,098 functions named, 0 `FUN_` left; 385 `EHHandler_*` and
  1,290 `Unwind_*` funclets named by owner.
- 2,651 functions mapped to their original PS2 developer method.
- ~5,400 docblocks, ~9,800 inline `//` comment lines, ~400 per-`case` docblocks.
- ~1,100 `g_*` globals and ~1,700 `s_*` strings named; raw `DAT_` tokens 1,253 → 14.
- Ghidra `WARNING` lines 2,688 → 0; records sorted by VA.
- Documentation: 116 struct/subsystem docs, 11 gameplay/engine write-ups, 5 police-system docs,
  5 data-format docs, 3 script-VM docs.
- Ghidra database `ghidra/Game.exe.gzf` (Ghidra 11.4.2) with all project names and data types.
