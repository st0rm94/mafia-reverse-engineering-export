# Changelog

One section per export version (date tag). Counts are measured on the annotated decompilation
`ghidra/Game-ghidra.exe.annotated.c` against Ghidra's default auto-analysis of `Game.exe` 1.0.

## 2026.10.13.1 — lap-glitch document trimmed

- `basic/artifacts/racing/lap-gates-and-lap-glitch.md`: the "Community claims assessed" section
  removed; the sections after it renumbered (Method §9, Cheat Engine test plan §10, Related
  documents §11, Open items §12) and the references to them updated.

## 2026.10.13 — open-item review with more name corrections, physics and collision findings, Ghidra names synced

- 296 more tracked open questions closed.
- Function renames (47; each docblock and registry row rewritten):
  - the `C_I3D_cache` item-vector helpers joined to their owner: `uninit_copy_range_stride_8`,
    `copy_elem_stride_8_with_string`, `assign_elem_stride_8_with_string`, `refcount_release_at_0x4`,
    `destroy_range_stride8`, `std_vector8_uninit_fill_n`, `std_vector8_assign_range`,
    `std_vector8_insert_one`, `array_copy_refcounted8`, `array_copy_backward_refcounted8`,
    `refcounted8_scalar_deleting_dtor`, `std_vector8_erase_at` → `i3d_cache_items_uninit_copy_range`,
    `_copy_construct_one`, `_copy_assign_one`, `_destroy_one`, `_destroy_range`, `_uninit_fill_n`,
    `_copy_assign_range`, `_insert_one`, `_copy_range`, `_copy_backward_range`,
    `_scalar_deleting_dtor`, `_erase_at`;
  - static destructors named after the object they free: `crt_static_dtor_004f62e0` …
    `crt_static_dtor_004f6560` → `traffic_adopted_elements_static_dtor`,
    `traffic_bridge_slots_static_dtor`, `traffic_car_registry_static_dtor`,
    `traffic_parking_cars_static_dtor`, `traffic_stolen_cars_static_dtor`,
    `traffic_mafia_crews_static_dtor`; `crt_atexit_free_rail_schedule_log` /
    `_rail_semaphore_list` / `_railway_list` / `_rail_clamp_candidates` →
    `rail_schedule_log_static_dtor`, `rail_semaphore_list_static_dtor`, `railway_list_static_dtor`,
    `rail_clamp_candidates_static_dtor`;
  - misnomers: `vec3_dist` / `vec3_dist_sq` → `vec3_length` / `vec3_length_sq`, `vector_clear` →
    `chunk_open_stack_free`, `copy_elem_stride_0x3c` → `ai_path_node_construct_copy`,
    `game_get_temp_actor_list_a` → `game_get_active_actor_list`, `string_append_cstr` →
    `string_concat_cstr`, `human_car_launched_into_air` → `human_die_in_exploding_car`,
    `recorder_sync_clock_from_system` → `recorder_set_clock`,
    `speech_channel_attach_accessory_frame` → `speech_channel_set_look_at_frame`,
    `speech_update_accessory_frames` → `speech_update_look_at_targets`, `speech_preload_for_record` →
    `speech_apply_recorder_key`, `game_item_table_get` → `inventory_get_left_hand_item_def`,
    `garage_generate_cars` / `garage_release_cars` → `garage_request_add_last_stolen` /
    `garage_request_show_results`, `garage_set_stolen_car` → `garage_set_last_entered_car`,
    `mission_set_sound_restore_stage` / `mission_get_sound_restore_stage` →
    `mission_set_frame_skip_count` / `mission_get_frame_skip_count`,
    `vehicle_controller_set_field_0x65` → `actor_set_save_excluded`, `vehicle_set_light_bit_0x80` →
    `vehicle_set_emergency_lights`, `vehicle_controller_set_field_0x214c` →
    `car_set_remove_at_game_done`, `player_set_wingman_indicator` → `human_set_wingman_indicator`,
    `i3d_frame_set_field_0x140_block` → `i3d_dummy_set_bound_box`, `game_subobject_0x208_dtor` →
    `glass_shatter_dtor`;
  - two shared vector-pair destructors that were filed under one owner: `force_feedback_manager_dtor`
    and `game_subobject_0x240_dtor` → `folded_vector_pair_free_at_0x04_0x14` /
    `folded_vector_pair_free_at_0x04_0x14_00607500`.
- Data names (22 renamed, 6 new): `g_hud_2intbenal_texture` → `g_hud_low_fuel_lamp_texture`; the
  `LS3D_setup` bytes named from the shipped `Setup.exe` dialog (`g_ls3d_setup_refresh_rate`,
  `_antialiasing`, `_multipass_rendering`, `_clip_always`, `_fullscreen`, `_hw_transform_lighting`,
  `_vsync`, `_software_mixing_only`, `_speaker_config`, `_language`), and its two flag words split
  into eight byte symbols (`_low_detail_textures`, `_compressed_textures`, `_triple_buffering`,
  `_truecolor_textures`, `_truecolor_lightmaps`, `_suspend_when_inactive`, `_sound_disabled`,
  `_eax_disabled`); nine `g_speech_channel_*` fields named (`_countdown_ms`, `_eye_target_frame`,
  `_neck_target_frame`, `_viseme_param_0x28` … `_0x3c`).
- PS2 twins: 2,590 → 2,602 functions mapped. New twins include `vec3_length` = `S_vector::Magnitude`,
  five `police_board_*` functions = `ai_police_nastenka` methods, `string_assign_cstr` =
  `C_str::operator=`, `engine_init` / `engine_shutdown` = `InitSystem` / `CloseSystem` and
  `using_object_find_near_objects` = `C_using_object::FindNearObjects`; the two garage requests
  re-paired to `AddLastStolen` / `ShowResults`; the `C_game::S_hlp$…` helper symbols spelled with
  their `$`.
- Annotated decompilation:
  - every docblock that carried only its provenance line now describes the function (among them
    `winmain`, `police_manager_set_speed_thresholds`, `vehicle_wheel_contact_solve` and
    `collision_load_level`); `police_board_has_recent_sighting` re-read from the disassembly (the
    receiver is the officer's sighting log at `officer+0xe88`, window 2000 ms);
  - 133 section references that read `SS<n>` restored to `§<n>`.
- Documentation: 96 docs updated, two new topic docs:
  `basic/artifacts/racing/lap-gates-and-lap-glitch.md` (the race lap gates and the mission 6 lap
  glitch, with 11 figures in `racing/img/`) and `basic/artifacts/garage/storing-cars-in-the-garage.md`
  (the four ways a car is banked in the garage). New findings include the collision-triangle flags
  read from a parse of all 81 shipped `tree.klz` files, the `ai_fight_manager` member order codes and
  what each makes the member do, the car physics sub-step (`+0xca0`, 0.028 s) and the wheel spring,
  bump and rebound terms, gear -1 = reverse and the brake / reverse light bits, the box cylinder flag
  and density, the player control lock, the odometer, which parser arm of the script dispatcher
  handles each command, and the `RenamePoliceCar` twin in `structs/police_manager.md` §6.
- Ghidra database `ghidra/Game.exe.gzf`: every rename above imported (47 functions, 28 data labels);
  the two `LS3D_setup` flag words retyped to bytes; a `CALL_RETURN` flow override at `0x006283e5`
  and `0x00628471` removes the two `jmp [ebx]` jump-table warnings in the x87 math dispatchers.
- README: the documentation list includes the two topic subdirectories.

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
