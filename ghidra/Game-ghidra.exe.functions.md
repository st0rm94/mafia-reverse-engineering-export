# Game.exe (PC) — function definitions

Every function definition in `Game-ghidra.exe.annotated.c`, recovered from the definitions themselves (the PC binary has no symbol table). 5098 function definitions (5098 named, 0 still `FUN_`; 5098 with a recovered VA). The compiler-generated EH glue is not listed: ~1290 `Unwind_*` funclets and 385 `EHHandler_*` __ehhandler$ thunks, which stay in the decomp.

`Ghidra name` = the name Ghidra itself gave that address — normally the `FUN_xxxxxxxx` id this project renamed away from, and Ghidra's own name for the ~1600 functions it identified itself (CRT, imports, the I3D SDK). Blank when Ghidra has no function at that VA.

`PS2 twin` = the PS2 (`SLUS_206.71`) developer symbol identified as this function's counterpart, demangled to `class::method` (a free function shows just the method) and resolved against `SLUS_206.71.functions.md`. Most rows also carry it as a `PS2 twin:` line in the function's docblock. Blank when no twin is identified.

`VA` is resolved highest-priority-first from: (1) Ghidra's own function entry points (authoritative); (2) a `FUN_`/`Unwind_`/`EHHandler_` name that carries its own hex; (3) the address recorded when the function was named; (4) a provenance token in the immediate docblock. Blank only if all four miss.

| # | VA | Ghidra name | Name | PS2 twin |
|---:|---|---|---|---|
| 1 | 0x00401000 | FUN_00401000 | ai_character_log_new_contact | ai_character_log::NewContact |
| 2 | 0x00401060 | FUN_00401060 | ai_logs_init | ai_logs::Init |
| 3 | 0x004010b0 | FUN_004010b0 | ai_logs_tick | ai_logs::Tick |
| 4 | 0x00401440 | FUN_00401440 | ai_logs_event | ai_logs::Event |
| 5 | 0x00401810 | FUN_00401810 | ai_logs_get_aiming_weight | ai_logs::GetAimingWeight |
| 6 | 0x00401880 | FUN_00401880 | ai_logs_get_shooting_weight | ai_logs::GetShootingWeight |
| 7 | 0x004018f0 | FUN_004018f0 | ai_logs_compute_attitude | ai_logs::ComputeAttitude |
| 8 | 0x00401aa0 | FUN_00401aa0 | ai_character_log_event_hit | ai_character_log::EventHit |
| 9 | 0x00401b50 | FUN_00401b50 | ai_character_log_event_enemy_hit | ai_character_log::EventEnemyHit |
| 10 | 0x00401bc0 | FUN_00401bc0 | ai_character_log_event_friend_hit | ai_character_log::EventFriendHit |
| 11 | 0x00401c50 | FUN_00401c50 | ai_character_log_event_visual_contact | ai_character_log::EventVisualCon |
| 12 | 0x00401cd0 | FUN_00401cd0 | ai_character_log_event_sound_contact | ai_character_log::EventSoundCon |
| 13 | 0x00401d20 | FUN_00401d20 | ai_logs_has_log | ai_logs::hasLog |
| 14 | 0x00401d90 | FUN_00401d90 | ai_logs_get_log_for_actor | ai_logs::GetLog |
| 15 | 0x00401df0 | FUN_00401df0 | ai_logs_get_max_angry | ai_logs::GetMaxAngryCharacter |
| 16 | 0x00401e80 | FUN_00401e80 | ai_logs_get_max_fear | ai_logs::GetMaxFearCharacter |
| 17 | 0x00401f10 | FUN_00401f10 | ai_logs_get_max_death | ai_logs::GetMaxDeathCharacter |
| 18 | 0x00401fa0 | FUN_00401fa0 | ai_logs_get_max_threat_score | ai_logs::GetMaxOhrozeni |
| 19 | 0x00402030 | FUN_00402030 | ai_logs_destroy_log_for_actor | ai_logs::Destroy |
| 20 | 0x004020d0 | FUN_004020d0 | ai_logs_reset_stale_logs | ai_logs::LooseDistant |
| 21 | 0x00402290 | FUN_00402290 | crt_static_init_guard_00402290 |  |
| 22 | 0x004022b0 | FUN_004022b0 | crt_atexit_register_004022b0 |  |
| 23 | 0x004022c0 | FUN_004022c0 | ai_check_builder_test_space_8dir | ai_check_builder::TestSpace8RExt |
| 24 | 0x004025c0 | FUN_004025c0 | vec3_rotate_by_angle_in_place | ai_check_builder::RotateVect |
| 25 | 0x00402620 | FUN_00402620 | ai_check_builder_get_dir8_index | ai_check_builder::GetOrientation |
| 26 | 0x004026c0 | FUN_004026c0 | vec3_set_xyz |  |
| 27 | 0x004026e0 | FUN_004026e0 | vec3_sub |  |
| 28 | 0x00402710 | FUN_00402710 | vec3_make_scaled |  |
| 29 | 0x00402740 | FUN_00402740 | crt_static_init_guard_00402740 |  |
| 30 | 0x00402760 | FUN_00402760 | crt_atexit_register_00402760 |  |
| 31 | 0x00402770 | thunk_FUN_00402780 | thunk_crt_init_dir8_table_normalized_006550d8 |  |
| 32 | 0x00402780 | FUN_00402780 | crt_init_dir8_table_normalized_006550d8 |  |
| 33 | 0x00402880 | thunk_FUN_00402890 | thunk_crt_init_dir8_table_unnormalized_00655138 |  |
| 34 | 0x00402890 | FUN_00402890 | crt_init_dir8_table_unnormalized_00655138 |  |
| 35 | 0x00402990 | thunk_FUN_004029a0 | thunk_crt_init_vec_up_00655198 |  |
| 36 | 0x004029a0 | FUN_004029a0 | crt_init_vec_up_00655198 |  |
| 37 | 0x004029c0 | FUN_004029c0 | i3d_frame_get_bound_points_2d | GetBoundPoints2D |
| 38 | 0x00402cd4 | FUN_00402cd4 | thunk_jmp_via_retaddr_plus_0x953b |  |
| 39 | 0x00402ce0 | FUN_00402ce0 | ai_wayfind_cb_corner_edge_cost | fxForwardShortest |
| 40 | 0x00402dd0 | FUN_00402dd0 | ai_wayfind_cb_flee_goal_beyond_radius | fxRunawayEndFuncV1 |
| 41 | 0x00402e80 | FUN_00402e80 | ai_wayfind_cb_flee_edge_cost_directional | fxRunawayFuncV1 |
| 42 | 0x00403080 | FUN_00403080 | ai_wayfind_cb_forward_goal_by_mode | fxForward2EndFuncV1 |
| 43 | 0x004030a0 | FUN_004030a0 | ai_wayfind_cb_forward_edge_cost | fxForward2FuncV1 |
| 44 | 0x00403530 | FUN_00403530 | ai_check_desc_unpack_data | ai_check_desc::UNPACKDATA |
| 45 | 0x00403570 | FUN_00403570 | ai_check_desc_unpack_ranges | ai_check_desc::UNPACKRANGES |
| 46 | 0x004035e0 | FUN_004035e0 | ai_wayfind_build_path_node_from_nodes | ai_wayfind::GetPathNode |
| 47 | 0x00403c30 | FUN_00403c30 | ai_wayfind_build_path_node_from_positions | ai_wayfind::GetPathNode |
| 48 | 0x00403e90 | FUN_00403e90 | vec3_div_scalar_in_place |  |
| 49 | 0x00403ec0 | FUN_00403ec0 | vec3_dot_product |  |
| 50 | 0x00403ee0 | FUN_00403ee0 | ai_wayfind_build_optimal_path | ai_wayfind::GetOptimalPath |
| 51 | 0x00405510 | FUN_00405510 | ai_wayfind_resolve_start_node | ai_wayfind::FindStartCheckpoint |
| 52 | 0x004059d0 | FUN_004059d0 | ai_wayfind_resolve_end_node | ai_wayfind::FindEndCheckpoint |
| 53 | 0x00405dd0 | FUN_00405dd0 | ai_wayfind_get_node_for_position | ai_wayfind::GetPosition |
| 54 | 0x00405e10 | FUN_00405e10 | ai_wayfind_plan_shortest_route | ai_wayfind::FindWaySHORTEST |
| 55 | 0x00406040 | FUN_00406040 | ai_wayfind_plan_route_via_corner | ai_wayfind::FindWayCORNERNEAR |
| 56 | 0x00406550 | FUN_00406550 | ai_wayfind_plan_flee_route | ai_wayfind::FindWayRUNAWAY_V1 |
| 57 | 0x00406930 | FUN_00406930 | ai_wayfind_plan_directed_route_v2 | ai_wayfind::FindWayFORWARD_V2 |
| 58 | 0x00406de0 | FUN_00406de0 | ai_wayfind_plan_directed_route_v3 | ai_wayfind::FindWayFORWARD_V3 |
| 59 | 0x004072c0 | FUN_004072c0 | ai_wayfind_plan_search_route | ai_wayfind::FindWaySEARCH |
| 60 | 0x00407620 | FUN_00407620 | ai_wayfind_test_corner_hide_side | ai_wayfind::hasCornerVisHide |
| 61 | 0x00407920 | FUN_00407920 | ai_path_destroy | ai_path::Destroy |
| 62 | 0x004079b0 | FUN_004079b0 | ai_path_find_spec_node_from_coll_list | ai_path::GetSpecNodeA |
| 63 | 0x00408e70 | FUN_00408e70 | ai_path_find_spec_node_from_plane | ai_path::GetSpecNodeB |
| 64 | 0x0040a0c0 | FUN_0040a0c0 | ai_path_find_spec_node_from_positions | ai_path::GetSpecNodeC |
| 65 | 0x0040b390 | FUN_0040b390 | ai_path_update_spec_waypoints | ai_path::UpdateSpec |
| 66 | 0x0040bca0 | FUN_0040bca0 | ai_path_advance_to_next_normal_waypoint | ai_path::GetNextNormalWaypoint |
| 67 | 0x0040bf30 | FUN_0040bf30 | ai_path_load_normal_waypoint | ai_path::GetNormalWaypoint |
| 68 | 0x0040c1c0 | FUN_0040c1c0 | ai_path_advance_to_next_spec_waypoint | ai_path::GetNextSpecWaypoint |
| 69 | 0x0040c260 | FUN_0040c260 | ai_path_is_last_normal_waypoint | ai_path::isLastNormalWaypoint |
| 70 | 0x0040c2a0 | FUN_0040c2a0 | ai_path_is_pos_in_connection_corridor | ai_path::isInConnection |
| 71 | 0x0040c5c0 | FUN_0040c5c0 | ai_path_is_point2d_in_connection_corridor | ai_path::isInConnection |
| 72 | 0x0040c930 | FUN_0040c930 | ai_path_is_point2d_in_checkpoint_radius | ai_path::isInCheckpoint |
| 73 | 0x0040c970 | FUN_0040c970 | ai_path_tick | ai_path::Tick |
| 74 | 0x0040cca0 | FUN_0040cca0 | ai_path_revalidate_for_actor | ai_path::Update |
| 75 | 0x0040d2c0 | FUN_0040d2c0 | ai_path_get_destination | ai_path::GetDestination |
| 76 | 0x0040d310 | FUN_0040d310 | ai_path_arm_delay_timer | ai_path::DelayTest |
| 77 | 0x0040d330 | FUN_0040d330 | ai_path_must_evade | ai_path::mustEvade |
| 78 | 0x0040d340 | FUN_0040d340 | ai_path_find_segment_for_node | ai_path::GetSegments |
| 79 | 0x0040d4e0 | FUN_0040d4e0 | ai_wayfind_find_nearest_hide_corner | ai_wayfind::FindCORNERNEAR |
| 80 | 0x0040d700 | FUN_0040d700 | ai_wayfind_score_and_add_checkpoint | ai_wayfind::WeightCheckpoints |
| 81 | 0x0040db30 | FUN_0040db30 | ai_path_do_connections_overlap | ai_path::areOverlaped |
| 82 | 0x0040dc10 | FUN_0040dc10 | ai_wayfind_compute_2d_line_intersection | ai_wayfind::Compute2DIntersection |
| 83 | 0x0040dc90 | FUN_0040dc90 | ai_wayfind_compute_node_left_right_gate | ai_wayfind::GetLR |
| 84 | 0x0040e850 | FUN_0040e850 | ai_wayfind_compute_nearest_point_on_segment | ai_wayfind::ComputeNearestPoint |
| 85 | 0x0040ea20 | FUN_0040ea20 | ai_path_force_pass_side_right | ai_path::ForceRight |
| 86 | 0x0040ea50 | FUN_0040ea50 | ai_path_is_pos_in_next_connection | ai_path::isInNextConn |
| 87 | 0x0040ebb0 | FUN_0040ebb0 | ai_path_find_blocking_dyn_object | ai_path::TestDetailLines |
| 88 | 0x0040ee50 | FUN_0040ee50 | ai_path_is_turning_at_normal_waypoint | ai_path::isTurningNormalNode |
| 89 | 0x0040efe0 | FUN_0040efe0 | stl_destroy_n_via_fnptr |  |
| 90 | 0x0040f010 | FUN_0040f010 | web_node_get_connection |  |
| 91 | 0x0040f040 | FUN_0040f040 | vec3_copy_to |  |
| 92 | 0x0040f060 | FUN_0040f060 | std_vector3c_insert_at |  |
| 93 | 0x0040f280 | FUN_0040f280 | vec_count_stride_0x3c |  |
| 94 | 0x0040f2b0 | FUN_0040f2b0 | std_vector3c_erase_at |  |
| 95 | 0x0040f300 | FUN_0040f300 | std_vector18_erase_at |  |
| 96 | 0x0040f350 | FUN_0040f350 | std_vector3c_insert_n |  |
| 97 | 0x0040f650 | FUN_0040f650 | uninit_copy_range_stride_0x3c |  |
| 98 | 0x0040f690 | FUN_0040f690 | uninit_fill_n_stride_0x3c |  |
| 99 | 0x0040f6c0 | FUN_0040f6c0 | copy_elem_stride_0x3c |  |
| 100 | 0x0040f6e0 | FUN_0040f6e0 | copy_elem_stride_0x18 |  |
| 101 | 0x0040f700 | FUN_0040f700 | crt_static_init_guard_0040f700 |  |
| 102 | 0x0040f720 | FUN_0040f720 | crt_atexit_register_0040f720 |  |
| 103 | 0x0040f730 | thunk_FUN_0040f740 | thunk_crt_init_vec_up_00655208 |  |
| 104 | 0x0040f740 | FUN_0040f740 | crt_init_vec_up_00655208 |  |
| 105 | 0x0040f760 | FUN_0040f760 | ai_sensors_find_visual_contact | ai_sensors::hasVisualCon |
| 106 | 0x0040f790 | FUN_0040f790 | ai_sensors_find_sound_contact | ai_sensors::hasSoundCon |
| 107 | 0x0040f7f0 | FUN_0040f7f0 | ai_sensors_find_smell_contact | ai_sensors::hasSmellCon |
| 108 | 0x0040f820 | FUN_0040f820 | ai_sensors_new_visual_contact | ai_sensors::NewVisualCon |
| 109 | 0x0040f8b0 | FUN_0040f8b0 | ai_sensors_update_visual_contact | ai_sensors::UpdateVisualCon |
| 110 | 0x0040f900 | FUN_0040f900 | ai_sensors_cancel_visual_contact | ai_sensors::CancelVisualCon |
| 111 | 0x0040f9c0 | FUN_0040f9c0 | ai_sensors_recompute_visual_extrema | ai_sensors::UpdateVIS |
| 112 | 0x0040faf0 | FUN_0040faf0 | ai_sensors_new_sound_contact | ai_sensors::NewSoundCon |
| 113 | 0x0040fb80 | FUN_0040fb80 | ai_sensors_update_sound_contact | ai_sensors::UpdateSoundCon |
| 114 | 0x0040fbf0 | FUN_0040fbf0 | ai_sensors_cancel_sound_contact | ai_sensors::CancelSoundCon |
| 115 | 0x0040fc30 | FUN_0040fc30 | ai_sensors_recompute_sound_extrema | ai_sensors::UpdateSND |
| 116 | 0x0040fd70 | FUN_0040fd70 | ai_sensors_new_smell_contact | ai_sensors::NewSmellCon |
| 117 | 0x0040fe30 | FUN_0040fe30 | ai_sensors_update_smell_contact | ai_sensors::UpdateSmellCon |
| 118 | 0x0040feb0 | FUN_0040feb0 | ai_sensors_cancel_smell_contact | ai_sensors::CancelSmellCon |
| 119 | 0x0040ff10 | FUN_0040ff10 | ai_sensors_recompute_smell_extrema | ai_sensors::UpdateSMELL |
| 120 | 0x00410050 | FUN_00410050 | ai_sensors_init | ai_sensors::Init |
| 121 | 0x004100f0 | FUN_004100f0 | ai_sensors_destroy | ai_sensors::Destroy |
| 122 | 0x004101b0 | FUN_004101b0 | ai_sensors_can_see | ai_sensors::isVisible |
| 123 | 0x00410200 | FUN_00410200 | ai_sensors_can_hear | ai_sensors::isHearable |
| 124 | 0x00410290 | FUN_00410290 | ai_sensors_can_smell | ai_sensors::isSmellable |
| 125 | 0x00410300 | FUN_00410300 | ai_sensors_get_smelled_target_info | ai_sensors::Sniff |
| 126 | 0x004105a0 | FUN_004105a0 | ai_sensors_get_heard_sound_info | ai_sensors::Hear |
| 127 | 0x00410630 | FUN_00410630 | ai_sensors_build_smell_contact_from_blob | ai_sensors::AnalyzeBlob |
| 128 | 0x00410670 | FUN_00410670 | ai_vis_logs_init | ai_vis_logs::Init |
| 129 | 0x004106c0 | FUN_004106c0 | ai_vis_logs_add_entity | ai_vis_logs::AddEntity |
| 130 | 0x00410770 | FUN_00410770 | ai_vis_logs_del_entity | ai_vis_logs::DelEntity |
| 131 | 0x004107d0 | FUN_004107d0 | ai_vis_logs_tick | ai_vis_logs::Tick |
| 132 | 0x004114d0 | FUN_004114d0 | entity_ctor | C_entity::{ctor} |
| 133 | 0x00411620 | FUN_00411620 | entity_scalar_deleting_dtor |  |
| 134 | 0x00411640 | FUN_00411640 | entity_dtor | C_entity::{dtor} |
| 135 | 0x004117a0 | FUN_004117a0 | entity_init | C_entity::Init |
| 136 | 0x004117c0 | FUN_004117c0 | entity_duplicate_from | C_entity::DuplicateFrom |
| 137 | 0x00411800 | FUN_00411800 | entity_game_init | C_entity::GameInit |
| 138 | 0x00411e80 | FUN_00411e80 | entity_reset | C_entity::Reset |
| 139 | 0x00411fb0 | FUN_00411fb0 | entity_reset_state | C_entity::ResetState |
| 140 | 0x00412040 | FUN_00412040 | human_reset_pose_and_scent_trail |  |
| 141 | 0x00412060 | FUN_00412060 | entity_game_done | C_entity::GameDone |
| 142 | 0x00412100 | FUN_00412100 | entity_save_data | C_entity::SaveData |
| 143 | 0x00412150 | FUN_00412150 | entity_load_data | C_entity::LoadData |
| 144 | 0x004121f0 | FUN_004121f0 | entity_collision | C_entity::Collision |
| 145 | 0x00412200 | FUN_00412200 | entity_hit | C_entity::Hit |
| 146 | 0x00412410 | FUN_00412410 | entity_cb_hit_info | C_entity::cbHitInfo |
| 147 | 0x00412480 | FUN_00412480 | entity_ai | C_entity::AI |
| 148 | 0x004145f0 | FUN_004145f0 | entity_action_tick | C_entity::actionTick |
| 149 | 0x00424e20 | FUN_00424e20 | hud_show_element |  |
| 150 | 0x00424e40 | FUN_00424e40 | hud_set_police_symbol_and_alpha |  |
| 151 | 0x00424e80 | FUN_00424e80 | vec2_set_xy |  |
| 152 | 0x00424ea0 | FUN_00424ea0 | vec2_length_sq |  |
| 153 | 0x00424ec0 | FUN_00424ec0 | vec2_normalize_in_place |  |
| 154 | 0x00424f00 | FUN_00424f00 | vec2_signed_distance_to_line |  |
| 155 | 0x00424f60 | FUN_00424f60 | vec3_scale_to_by_ptr |  |
| 156 | 0x00424f90 | FUN_00424f90 | vec3_scale_in_place |  |
| 157 | 0x00424fb0 | FUN_00424fb0 | vec3_negate_to |  |
| 158 | 0x00424fd0 | FUN_00424fd0 | vec3_add_to |  |
| 159 | 0x00425000 | FUN_00425000 | vec3_add_in_place |  |
| 160 | 0x00425030 | FUN_00425030 | vec3_sub_in_place |  |
| 161 | 0x00425060 | FUN_00425060 | clear3_dwords_at |  |
| 162 | 0x00425070 | FUN_00425070 | vec3_dist |  |
| 163 | 0x004250f0 | FUN_004250f0 | vec3_dist_sq |  |
| 164 | 0x00425110 | FUN_00425110 | vec3_cross_to |  |
| 165 | 0x00425150 | FUN_00425150 | vec3_cross_global_lhs_to |  |
| 166 | 0x004251a0 | FUN_004251a0 | vec3_cross_global_rhs_to |  |
| 167 | 0x004251f0 | FUN_004251f0 | i3d_frame_get_local_position_ptr |  |
| 168 | 0x00425200 | FUN_00425200 | i3d_frame_get_world_position_ptr |  |
| 169 | 0x00425220 | FUN_00425220 | i3d_frame_get_forward_normalized |  |
| 170 | 0x00425390 | FUN_00425390 | act_mission_get_game |  |
| 171 | 0x004253a0 | FUN_004253a0 | folded_get_dword_0x10 |  |
| 172 | 0x004253b0 | FUN_004253b0 | actor_release | C_actor::Release |
| 173 | 0x004253d0 | FUN_004253d0 | actor_get_cached_world_pos |  |
| 174 | 0x004253f0 | FUN_004253f0 | actor_set_direction |  |
| 175 | 0x00425410 | FUN_00425410 | folded_get_dword_0x68 | C_actor::GetFrame |
| 176 | 0x00425420 | FUN_00425420 | stub_ret0_args4_00425420 |  |
| 177 | 0x00425430 | FUN_00425430 | actor_is_alive |  |
| 178 | 0x00425440 | FUN_00425440 | vehicle_get_speed |  |
| 179 | 0x00425450 | FUN_00425450 | vehicle_get_velocity_ptr |  |
| 180 | 0x00425460 | FUN_00425460 | vehicle_get_seat_count |  |
| 181 | 0x00425490 | FUN_00425490 | vehicle_get_forward_ptr |  |
| 182 | 0x004254a0 | FUN_004254a0 | car_get_fuel_tank_world_pos |  |
| 183 | 0x004254e0 | FUN_004254e0 | traffic_car_get_registry_header_addr |  |
| 184 | 0x004254f0 | FUN_004254f0 | ambient_ai_set_suspended |  |
| 185 | 0x00425500 | FUN_00425500 | police_manager_set_busted_cutscene_lock | C_shvestky::NeviDoKterePiceVlezl |
| 186 | 0x00425510 | FUN_00425510 | game_get_camera |  |
| 187 | 0x00425520 | FUN_00425520 | game_get_temp_actor_list_a |  |
| 188 | 0x00425530 | FUN_00425530 | game_get_player_actor |  |
| 189 | 0x00425540 | FUN_00425540 | game_get_frame_time_ms |  |
| 190 | 0x00425550 | FUN_00425550 | game_get_sound_arena |  |
| 191 | 0x00425560 | FUN_00425560 | game_get_police_block |  |
| 192 | 0x00425570 | FUN_00425570 | game_get_telephone_list |  |
| 193 | 0x00425580 | FUN_00425580 | game_freeride_score_get |  |
| 194 | 0x00425590 | thunk_FUN_004255a0 | thunk_crt_init_dir8_table_normalized_006551a8 |  |
| 195 | 0x004255a0 | FUN_004255a0 | crt_init_dir8_table_normalized_006551a8 |  |
| 196 | 0x004256a0 | FUN_004256a0 | ai_path_is_invalidated |  |
| 197 | 0x004256b0 | FUN_004256b0 | ai_path_is_abandoned |  |
| 198 | 0x004256c0 | FUN_004256c0 | ai_path_has_blocked_con |  |
| 199 | 0x004256d0 | FUN_004256d0 | ai_path_get_state |  |
| 200 | 0x004256e0 | FUN_004256e0 | ai_wayfind_set_checkpoint_container |  |
| 201 | 0x004256f0 | FUN_004256f0 | ai_wayfind_clear_checkpoint_container |  |
| 202 | 0x00425700 | FUN_00425700 | door_get_state |  |
| 203 | 0x00425710 | FUN_00425710 | human_get_actor_speed | C_human::GetActorSpeed |
| 204 | 0x00425730 | FUN_00425730 | actor_get_vehicle |  |
| 205 | 0x00425740 | FUN_00425740 | entity_is_tickable |  |
| 206 | 0x00425780 | FUN_00425780 | human_set_shot_target_pos |  |
| 207 | 0x004257a0 | FUN_004257a0 | entity_clear_act_state |  |
| 208 | 0x004257b0 | FUN_004257b0 | entity_set_act_state_shock |  |
| 209 | 0x004257c0 | FUN_004257c0 | ai_groups_extend_control_lock |  |
| 210 | 0x004257e0 | FUN_004257e0 | ai_path_scalar_deleting_dtor |  |
| 211 | 0x00425800 | FUN_00425800 | ai_path_free_buffers |  |
| 212 | 0x00425850 | FUN_00425850 | ai_modifier_tick | ai_modifier::Tick |
| 213 | 0x004258b0 | FUN_004258b0 | entity_get_program | C_entity::GetProgram |
| 214 | 0x004258c0 | FUN_004258c0 | entity_actions_tick | C_entity::actionsTick |
| 215 | 0x00425b60 | FUN_00425b60 | entity_get_ai_substate |  |
| 216 | 0x00425b70 | FUN_00425b70 | entity_get_ai_vis_logs |  |
| 217 | 0x00425b80 | FUN_00425b80 | entity_get_ai_sensors |  |
| 218 | 0x00425b90 | FUN_00425b90 | entity_get_ai_logs |  |
| 219 | 0x00425ba0 | FUN_00425ba0 | entity_get_ai_fight_modes |  |
| 220 | 0x00425bb0 | FUN_00425bb0 | entity_update | C_entity::Update |
| 221 | 0x00425cb0 | FUN_00425cb0 | entity_death | C_entity::Death |
| 222 | 0x00425d30 | FUN_00425d30 | entity_script_decode_params | C_entity::DecodeParams |
| 223 | 0x00427650 | FUN_00427650 | free_ptr_pair_and_clear |  |
| 224 | 0x00427690 | FUN_00427690 | entity_script_decode_instruction | C_entity::DecodeInstruction |
| 225 | 0x00429ca0 | FUN_00429ca0 | entity_script_format_command | C_entity::DecompileCommand |
| 226 | 0x0042a660 | FUN_0042a660 | entity_process_command | C_entity::Process |
| 227 | 0x0042dc90 | FUN_0042dc90 | vec3_dot_product_0042dc90 |  |
| 228 | 0x0042dcb0 | FUN_0042dcb0 | ai_group_add_car |  |
| 229 | 0x0042de60 | FUN_0042de60 | entity_script_tick | C_entity::scriptTick |
| 230 | 0x0042e530 | FUN_0042e530 | entity_get_sight_range | C_entity::GetVisibleRange |
| 231 | 0x0042e550 | FUN_0042e550 | entity_get_hearing_sensitivity | C_entity::GetHearQuality |
| 232 | 0x0042e560 | FUN_0042e560 | entity_get_head_dir | C_entity::GetHeadDir |
| 233 | 0x0042e590 | FUN_0042e590 | entity_get_head_pos | C_entity::GetHeadPos |
| 234 | 0x0042e5c0 | FUN_0042e5c0 | entity_script_switch | C_entity::ScriptSwitch |
| 235 | 0x0042e650 | FUN_0042e650 | entity_force_ai | C_entity::ForceAI |
| 236 | 0x0042e7f0 | FUN_0042e7f0 | entity_script_is_fake | C_entity::isFakeScript |
| 237 | 0x0042e8a0 | FUN_0042e8a0 | entity_force_script | C_entity::ForceScript |
| 238 | 0x0042e990 | FUN_0042e990 | entity_script_callback | C_entity::CallBack |
| 239 | 0x0042e9c0 | FUN_0042e9c0 | entity_dispatch_event | C_entity::Event |
| 240 | 0x0042ef70 | FUN_0042ef70 | ai_action_manager_clear_all | ai_action_manager::Clear |
| 241 | 0x0042f060 | FUN_0042f060 | ai_action_generate_UID | ai_action_manager::GenUID |
| 242 | 0x0042f070 | FUN_0042f070 | ai_action_destroy_action | ai_action_manager::DestroyAction |
| 243 | 0x0042f2f0 | FUN_0042f2f0 | ai_action_invalidate_path_for_actor | ai_action_manager::InvalidatePath |
| 244 | 0x0042f4f0 | FUN_0042f4f0 | ai_action_manager_invalidate_actor | ai_action_manager::Invalidate |
| 245 | 0x0042f570 | FUN_0042f570 | ai_action_new_move | ai_action_manager::NewMove |
| 246 | 0x0042f620 | FUN_0042f620 | ai_action_new_follow | ai_action_manager::NewFollow |
| 247 | 0x0042f6b0 | FUN_0042f6b0 | ai_action_new_turn_to | ai_action_manager::NewTurnTo |
| 248 | 0x0042f710 | FUN_0042f710 | ai_action_new_fire | ai_action_manager::NewFire |
| 249 | 0x0042f7b0 | FUN_0042f7b0 | ai_action_new_follow_link | ai_action_manager::NewFollowLink |
| 250 | 0x0042f850 | FUN_0042f850 | ai_action_new_blocked_way | ai_action_manager::NewBlockedWay |
| 251 | 0x0042f8c0 | FUN_0042f8c0 | ai_action_new_runaway_dist | ai_action_manager::NewRunawayDist |
| 252 | 0x0042f980 | FUN_0042f980 | ai_action_new_side_look | ai_action_manager::NewSideLook |
| 253 | 0x0042fa10 | FUN_0042fa10 | ai_action_new_back_look | ai_action_manager::NewBackLook |
| 254 | 0x0042fbb0 | FUN_0042fbb0 | ai_action_new_unlink | ai_action_manager::NewUnLink |
| 255 | 0x0042fc20 | FUN_0042fc20 | ai_action_new_run_closer_to | ai_action_manager::NewRunCloserTo |
| 256 | 0x0042fd00 | FUN_0042fd00 | ai_action_new_freeze | ai_action_manager::NewFreeze |
| 257 | 0x0042fd70 | FUN_0042fd70 | ai_action_new_move_to_car | ai_action_manager::NewMoveToCar |
| 258 | 0x0042fe10 | FUN_0042fe10 | ai_action_new_step_away | ai_action_manager::NewStepAway |
| 259 | 0x0042fe70 | FUN_0042fe70 | ai_action_new_strafe | ai_action_manager::NewStrafe |
| 260 | 0x0042ff30 | FUN_0042ff30 | ai_action_new_jump | ai_action_manager::NewJump |
| 261 | 0x0042ffa0 | FUN_0042ffa0 | ai_action_new_corner_look | ai_action_manager::NewCornerLook |
| 262 | 0x00430020 | FUN_00430020 | ai_action_new_corner_cover_fire | ai_action_manager::NewCornerCoverFire |
| 263 | 0x004300a0 | FUN_004300a0 | ai_action_new_corner_fire | ai_action_manager::NewCornerFire |
| 264 | 0x00430120 | FUN_00430120 | ai_action_new_corner_take_pos | ai_action_manager::NewCornerTakePos |
| 265 | 0x004301a0 | FUN_004301a0 | ai_action_new_corner_fight | ai_action_manager::NewCornerFight |
| 266 | 0x00430290 | FUN_00430290 | ai_action_new_crouch_fight | ai_action_manager::NewCrouchFight |
| 267 | 0x00430440 | FUN_00430440 | ai_action_new_normal_fight | ai_action_manager::NewNormalFight |
| 268 | 0x00430590 | FUN_00430590 | ai_action_new_barricade_fight | ai_action_manager::NewBarricadeFight |
| 269 | 0x00430650 | FUN_00430650 | ai_action_new_follow_in_rail | ai_action_manager::NewFollowInRail |
| 270 | 0x00430710 | FUN_00430710 | ai_action_new_police_action | ai_action_manager::NewPoliceAction |
| 271 | 0x00430810 | FUN_00430810 | ai_action_new_search | ai_action_manager::NewSearch |
| 272 | 0x004308e0 | FUN_004308e0 | ai_action_new_kick_driver | ai_action_manager::NewKickDriver |
| 273 | 0x004309d0 | FUN_004309d0 | ai_action_new_hostile_coverage | ai_action_manager::NewHostileCoverage |
| 274 | 0x00430cc0 | FUN_00430cc0 | ai_action_new_car_jump | ai_action_manager::NewCarJump |
| 275 | 0x00430d30 | FUN_00430d30 | ai_action_new_car_stop | ai_action_manager::NewCarStop |
| 276 | 0x00430dc0 | FUN_00430dc0 | ai_action_new_car_aim | ai_action_manager::NewCarAim |
| 277 | 0x00430e50 | FUN_00430e50 | ai_action_new_police_call | ai_action_manager::NewPoliceCall |
| 278 | 0x004312d0 | FUN_004312d0 | ai_action_new_strafe_him | ai_action_manager::NewStrafeHim |
| 279 | 0x00431350 | FUN_00431350 | ai_action_new_strafe_beware | ai_action_manager::NewStrafeBeware |
| 280 | 0x004313d0 | FUN_004313d0 | ai_action_manager_add_job | ai_action_manager::AddJob |
| 281 | 0x00431580 | FUN_00431580 | ai_action_manager_new_job | ai_action_manager::NewJob |
| 282 | 0x00431730 | FUN_00431730 | ai_action_manager_add_modifier | ai_action_manager::AddModifier |
| 283 | 0x004318e0 | FUN_004318e0 | ai_action_manager_new_modifier | ai_action_manager::NewModifier |
| 284 | 0x00431a90 | FUN_00431a90 | ai_action_manager_add_idle | ai_action_manager::AddIdle |
| 285 | 0x00431c40 | FUN_00431c40 | ai_action_manager_new_idle | ai_action_manager::NewIdle |
| 286 | 0x00431df0 | FUN_00431df0 | ai_action_manager_adopt_modifiers | ai_action_manager::AdoptModifiers |
| 287 | 0x00431e30 | FUN_00431e30 | ai_action_manager_adopt_idles | ai_action_manager::AdoptIdles |
| 288 | 0x00431e70 | FUN_00431e70 | ai_action_manager_derive_job | ai_action_manager::DeriveJob |
| 289 | 0x00431ec0 | FUN_00431ec0 | ai_action_manager_get_top_job | ai_action_manager::GetJob |
| 290 | 0x00431ee0 | FUN_00431ee0 | ai_action_manager_get_base_job | ai_action_manager::GetBaseJob |
| 291 | 0x00431f20 | FUN_00431f20 | ai_action_manager_retire_top_job | ai_action_manager::NextJob |
| 292 | 0x004321e0 | FUN_004321e0 | ai_action_manager_retire_top_modifier | ai_action_manager::NextModifier |
| 293 | 0x00432260 | FUN_00432260 | ai_action_manager_retire_top_idle | ai_action_manager::NextIdle |
| 294 | 0x004322e0 | FUN_004322e0 | ai_action_manager_has_idle_linked_to | ai_action_manager::hasIdle |
| 295 | 0x00432330 | FUN_00432330 | ai_action_manager_has_action | ai_action_manager::hasAction |
| 296 | 0x00432410 | FUN_00432410 | ai_action_manager_get_job_by_type | ai_action_manager::GetJob |
| 297 | 0x00432450 | FUN_00432450 | ai_action_manager_signal_children | ai_action_manager::SignalToChilds |
| 298 | 0x004324d0 | FUN_004324d0 | ai_character_init | ai_character::Init |
| 299 | 0x00432500 | FUN_00432500 | ai_character_get_morale | ai_character::GetMoralka |
| 300 | 0x00432510 | FUN_00432510 | ai_character_get_intelligence | ai_character::GetInteligence |
| 301 | 0x00432520 | FUN_00432520 | ai_character_get_aggression | ai_character::GetAgresivita |
| 302 | 0x00432530 | FUN_00432530 | ai_character_get_sight | ai_character::GetZrak |
| 303 | 0x00432540 | FUN_00432540 | ai_character_get_hearing | ai_character::GetSluch |
| 304 | 0x00432550 | FUN_00432550 | entity_weapon_choose | C_entity::WeaponChoose |
| 305 | 0x00432c50 | FUN_00432c50 | entity_equip_weapon_by_id | C_entity::WeaponChoose |
| 306 | 0x00432ea0 | FUN_00432ea0 | entity_weapon_get_max_range | C_entity::WeaponGetMaxRange |
| 307 | 0x00432eb0 | FUN_00432eb0 | entity_weapon_get_best_range | C_entity::WeaponGetBestRange |
| 308 | 0x00432ec0 | FUN_00432ec0 | entity_weapon_get_spread | C_entity::WeaponGetRozptyl |
| 309 | 0x00432ed0 | FUN_00432ed0 | entity_weapon_get_readiness | C_entity::WeaponGetReady |
| 310 | 0x00432ee0 | FUN_00432ee0 | entity_weapon_is_armed | C_entity::WeaponIsArmed |
| 311 | 0x00432ef0 | FUN_00432ef0 | entity_weapon_get_info | C_entity::WeaponGetInfo |
| 312 | 0x00432f60 | FUN_00432f60 | entity_do_shoot_timed | C_entity::Do_Shoot_Timed |
| 313 | 0x00433030 | FUN_00433030 | entity_weapon_timing_setup | C_entity::WeaponTimingSetup |
| 314 | 0x00433170 | FUN_00433170 | entity_release_target | C_entity::InvalidateActor |
| 315 | 0x00433290 | FUN_00433290 | entity_rebind_neck_bone_handle | C_entity::InvalidateSelf |
| 316 | 0x004332d0 | FUN_004332d0 | entity_can_safe_shoot | C_entity::CanSafeShoot |
| 317 | 0x00433910 | FUN_00433910 | entity_trace_human_scent | C_entity::TraceHuman |
| 318 | 0x004339e0 | FUN_004339e0 | entity_add_using_object | C_entity::AddUsingObject |
| 319 | 0x00433ab0 | FUN_00433ab0 | entity_switch_ai_state | C_entity::StateSwitcher |
| 320 | 0x004340c0 | FUN_004340c0 | entity_spawn_police_response | C_entity::SpawnPolice |
| 321 | 0x004342b0 | FUN_004342b0 | entity_lock_substate | C_entity::LockSubstate |
| 322 | 0x00434550 | FUN_00434550 | entity_unlock_substate | C_entity::UnlockSubstate |
| 323 | 0x004345e0 | FUN_004345e0 | entity_has_locked_state | C_entity::hasLockedState |
| 324 | 0x004345f0 | FUN_004345f0 | entity_has_locked_substate | C_entity::hasLockedSubstate |
| 325 | 0x00434680 | FUN_00434680 | entity_get_group_position | C_entity::GetGroupPosition |
| 326 | 0x004346e0 | FUN_004346e0 | entity_mem_write | C_entity::MemWrite |
| 327 | 0x00434720 | FUN_00434720 | entity_mem_read | C_entity::MemRead |
| 328 | 0x00434760 | FUN_00434760 | entity_get_vis_logs_size | C_entity::GetVisLogsSize |
| 329 | 0x00434770 | FUN_00434770 | entity_save_vis_logs | C_entity::SaveVisLogs |
| 330 | 0x00434840 | FUN_00434840 | entity_load_vis_logs | C_entity::LoadVisLogs |
| 331 | 0x004348e0 | FUN_004348e0 | entity_get_sensors_size | C_entity::GetSensorsSize |
| 332 | 0x004348f0 | FUN_004348f0 | entity_save_sensors | C_entity::SaveSensors |
| 333 | 0x00434c10 | FUN_00434c10 | entity_load_sensors | C_entity::LoadSensors |
| 334 | 0x00434ea0 | FUN_00434ea0 | entity_get_logs_size | C_entity::GetLogsSize |
| 335 | 0x00434f00 | FUN_00434f00 | entity_save_logs | C_entity::SaveLogs |
| 336 | 0x004351e0 | FUN_004351e0 | entity_load_logs | C_entity::LoadLogs |
| 337 | 0x00435410 | FUN_00435410 | entity_get_character_size | C_entity::GetCharacterSize |
| 338 | 0x00435420 | FUN_00435420 | entity_save_character | C_entity::SaveCharacter |
| 339 | 0x004354e0 | FUN_004354e0 | entity_load_character | C_entity::LoadCharacter |
| 340 | 0x004355a0 | FUN_004355a0 | entity_get_state_size | C_entity::GetStateSize |
| 341 | 0x00435600 | FUN_00435600 | entity_save_state | C_entity::SaveState |
| 342 | 0x004358f0 | FUN_004358f0 | entity_load_state | C_entity::LoadState |
| 343 | 0x00435e00 | FUN_00435e00 | entity_get_action_size | C_entity::GetActionSize |
| 344 | 0x00435f60 | FUN_00435f60 | entity_get_action_man_size | C_entity::GetActionManSize |
| 345 | 0x00436020 | FUN_00436020 | entity_save_action_man | C_entity::SaveActionMan |
| 346 | 0x00436cf0 | FUN_00436cf0 | entity_load_action_man | C_entity::LoadActionMan |
| 347 | 0x00438430 | FUN_00438430 | entity_save_game_get_size | C_entity::SaveGameGetSize |
| 348 | 0x00438490 | FUN_00438490 | entity_save_game_save | C_entity::SaveGameSave |
| 349 | 0x00438550 | FUN_00438550 | entity_save_game_load | C_entity::SaveGameLoad |
| 350 | 0x00438610 | FUN_00438610 | entity_is_in_entity_fire_area | C_entity::isInEntityFireArea |
| 351 | 0x00438970 | FUN_00438970 | entity_is_in_player_fire_area | C_entity::isInPlayerFireArea |
| 352 | 0x00438ea0 | FUN_00438ea0 | entity_get_fire_metrics | C_entity::GetFireMetrics |
| 353 | 0x00438f00 | FUN_00438f00 | entity_get_player_fire_metrics | C_entity::GetPlayerFireMetrics |
| 354 | 0x00438fa0 | FUN_00438fa0 | entity_get_target | C_entity::GetTarget |
| 355 | 0x00439050 | FUN_00439050 | entity_beware_cross_fire | C_entity::BewareCrossFire |
| 356 | 0x004392e0 | FUN_004392e0 | entity_has_heard_gunfire | C_entity::isHearedShooting |
| 357 | 0x00439320 | FUN_00439320 | ai_fight_modes_init | ai_fight_modes::Init |
| 358 | 0x00439460 | FUN_00439460 | ai_fight_modes_destroy | ai_fight_modes::Destroy |
| 359 | 0x00439490 | FUN_00439490 | ai_fight_modes_set_enemy | ai_fight_modes::SetEnemy |
| 360 | 0x004394d0 | FUN_004394d0 | ai_fight_modes_reset | ai_fight_modes::Invalidate |
| 361 | 0x00439510 | FUN_00439510 | ai_fight_modes_tick | ai_fight_modes::Tick |
| 362 | 0x00439980 | FUN_00439980 | ai_fight_modes_apply_hit | ai_fight_modes::Hit |
| 363 | 0x004399c0 | FUN_004399c0 | ai_fight_modes_enact_active_mode | ai_fight_modes::Select |
| 364 | 0x00439bf0 | FUN_00439bf0 | ai_fight_modes_get_hit_stat | ai_fight_modes::GetHitStat |
| 365 | 0x00439c10 | FUN_00439c10 | ai_fight_modes_set_mode_score | ai_fight_modes::PrefferMode |
| 366 | 0x00439c50 | FUN_00439c50 | ai_fight_modes_finish_mode | ai_fight_modes::FinishMode |
| 367 | 0x00439c70 | FUN_00439c70 | ai_fight_modes_request_finish_mode | ai_fight_modes::RequestFinishMode |
| 368 | 0x00439c90 | FUN_00439c90 | ai_fight_modes_consume_finish_request | ai_fight_modes::ModeMustFinish |
| 369 | 0x00439cb0 | FUN_00439cb0 | ai_fight_modes_force_mode | ai_fight_modes::ForceMode |
| 370 | 0x00439ce0 | FUN_00439ce0 | ai_talkman_tick | ai_talkman::Tick |
| 371 | 0x00439dd0 | FUN_00439dd0 | ai_talkman_say | ai_talkman::Say |
| 372 | 0x00439f10 | FUN_00439f10 | ai_talkman_init | ai_talkman::Init |
| 373 | 0x00439f50 | FUN_00439f50 | ai_talkman_begin_frame | ai_talkman::NewFrame |
| 374 | 0x00439f70 | FUN_00439f70 | ai_space_coverage_compute | ai_space_coverage::Compute |
| 375 | 0x00439fe0 | FUN_00439fe0 | ai_space_coverage_tick | ai_space_coverage::Tick |
| 376 | 0x0043a0d0 | FUN_0043a0d0 | ai_space_coverage_get_look_dir | ai_space_coverage::GetLook |
| 377 | 0x0043a120 | FUN_0043a120 | ai_space_coverage_prefer_dir | ai_space_coverage::Preffer |
| 378 | 0x0043a150 | FUN_0043a150 | entity_is_in_danger | C_entity::IsInDanger |
| 379 | 0x0043a220 | FUN_0043a220 | entity_expire_ghost_trail | C_entity::TickGhost |
| 380 | 0x0043a430 | FUN_0043a430 | ai_ghost_add_or_refresh | ai_ghost::Add |
| 381 | 0x0043a6e0 | FUN_0043a6e0 | ai_ghost_add_human_colliders | ai_ghost::AddHuman |
| 382 | 0x0043a730 | FUN_0043a730 | ai_ghost_clear_all | ai_ghost::Init |
| 383 | 0x0043a770 | FUN_0043a770 | ai_ghost_clear_all_and_mark_changed | ai_ghost::Reset |
| 384 | 0x0043a7b0 | FUN_0043a7b0 | ai_ghost_invalidate_actor | ai_ghost::Invalidate |
| 385 | 0x0043a850 | FUN_0043a850 | entity_suspend_ai_and_hide | C_entity::NeserADejchej |
| 386 | 0x0043a8e0 | FUN_0043a8e0 | entity_resume_ai_and_show | C_entity::NeserANedejchej |
| 387 | 0x0043a980 | FUN_0043a980 | entity_add_blocked_con | C_entity::AddBlockedCon |
| 388 | 0x0043abe0 | FUN_0043abe0 | dword_vec_copy_assign |  |
| 389 | 0x0043ac50 | FUN_0043ac50 | game_telephone_at |  |
| 390 | 0x0043ac60 | FUN_0043ac60 | ai_path_normal_waypoint_at |  |
| 391 | 0x0043ac80 | FUN_0043ac80 | ai_path_pass_side_at |  |
| 392 | 0x0043ac90 | FUN_0043ac90 | std_vector10_insert_n |  |
| 393 | 0x0043afa0 | FUN_0043afa0 | uint_ptr_min |  |
| 394 | 0x0043afb0 | FUN_0043afb0 | ai_character_log_copy_range |  |
| 395 | 0x0043afe0 | FUN_0043afe0 | stl_construct_stride_0x1c |  |
| 396 | 0x0043b000 | FUN_0043b000 | crt_static_init_guard_0043b000 |  |
| 397 | 0x0043b020 | FUN_0043b020 | crt_atexit_register_0043b020 |  |
| 398 | 0x0043b030 | FUN_0043b030 | program_base_thunk_entity_scalar_deleting_dtor | C_entity::{dtor} |
| 399 | 0x0043b040 | FUN_0043b040 | program_base_thunk_entity_save_game_get_size | C_entity::SaveGameGetSize |
| 400 | 0x0043b050 | FUN_0043b050 | program_base_thunk_entity_save_game_save | C_entity::SaveGameSave |
| 401 | 0x0043b060 | FUN_0043b060 | program_base_thunk_entity_save_game_load | C_entity::SaveGameLoad |
| 402 | 0x0043b070 | FUN_0043b070 | ai_group_add_member | ai_group::Add |
| 403 | 0x0043b100 | FUN_0043b100 | ai_group_del_member | ai_group::Del |
| 404 | 0x0043b160 | FUN_0043b160 | ai_group_send_sniff_event | ai_group::GetSniff |
| 405 | 0x0043b360 | FUN_0043b360 | ai_group_push_event | ai_group::UnderAttack |
| 406 | 0x0043b5c0 | FUN_0043b5c0 | ai_group_process_events | ai_group::ProcessEvents |
| 407 | 0x0043b710 | FUN_0043b710 | ai_group_get_car_factor | ai_group::GetCarFactor |
| 408 | 0x0043b770 | FUN_0043b770 | ai_group_get_closest_usable_car | ai_group::GetCarClosestAndUsable |
| 409 | 0x0043bb30 | FUN_0043bb30 | ai_groups_new_group | ai_groups::NewGroup |
| 410 | 0x0043bd00 | FUN_0043bd00 | ai_group_dtor | ai_group::{dtor} |
| 411 | 0x0043bd70 | FUN_0043bd70 | ai_groups_get_group | ai_groups::GetGroup |
| 412 | 0x0043bdd0 | FUN_0043bdd0 | ai_groups_clear_all_groups | ai_groups::Init |
| 413 | 0x0043be40 | FUN_0043be40 | ai_groups_destroy | ai_groups::Destroy |
| 414 | 0x0043bfe0 | FUN_0043bfe0 | ai_groups_tick | ai_groups::Tick |
| 415 | 0x0043c170 | FUN_0043c170 | ai_groups_get_save_size | ai_groups::GetSize |
| 416 | 0x0043c250 | FUN_0043c250 | ai_groups_save | ai_groups::Save |
| 417 | 0x0043c590 | FUN_0043c590 | ai_groups_load | ai_groups::Load |
| 418 | 0x0043ca60 | FUN_0043ca60 | ai_groups_invalidate_actor | ai_groups::InvalidateActor |
| 419 | 0x0043ccf0 | FUN_0043ccf0 | ai_group_can_receive | ai_group::canReceive |
| 420 | 0x0043cd30 | FUN_0043cd30 | ai_group_can_cast | ai_group::canCast |
| 421 | 0x0043cd70 | FUN_0043cd70 | ai_groups_group_has_car | ai_groups::hasCarGroup |
| 422 | 0x0043cdf0 | FUN_0043cdf0 | ai_follow_manager_del_follower | ai_follow_manager::DelFollower |
| 423 | 0x0043cf80 | FUN_0043cf80 | ai_follow_manager_add_follower | ai_follow_manager::AddFollower |
| 424 | 0x0043d090 | FUN_0043d090 | ai_follow_manager_add_follower_with_destination | ai_follow_manager::AddFollower |
| 425 | 0x0043d200 | FUN_0043d200 | ai_follow_manager_update | ai_follow_manager::Update |
| 426 | 0x0043da00 | FUN_0043da00 | ai_follow_manager_clear_all_groups | ai_follow_manager::Init |
| 427 | 0x0043da90 | FUN_0043da90 | ai_follow_manager_destroy | ai_follow_manager::Destroy |
| 428 | 0x0043db90 | FUN_0043db90 | ai_follow_manager_group_fits_leader_vehicle | ai_follow_manager::CanFolloshipFollow |
| 429 | 0x0043dcd0 | FUN_0043dcd0 | ai_follow_manager_get_car_seat_for_follower | ai_follow_manager::GetCarSeat |
| 430 | 0x0043de20 | FUN_0043de20 | ai_follow_manager_invalidate_actor | ai_follow_manager::Invalidate |
| 431 | 0x0043df40 | FUN_0043df40 | ai_follow_manager_event | ai_follow_manager::Event |
| 432 | 0x0043e0a0 | FUN_0043e0a0 | stub_retfalse_args4_0043e0a0 |  |
| 433 | 0x0043e0b0 | FUN_0043e0b0 | ai_follow_manager_should_follower_leave_group | ai_follow_manager::mustLeave |
| 434 | 0x0043e150 | FUN_0043e150 | ai_follow_manager_get_save_size | ai_follow_manager::GetSize |
| 435 | 0x0043e1c0 | FUN_0043e1c0 | ai_follow_manager_save | ai_follow_manager::Save |
| 436 | 0x0043e3d0 | FUN_0043e3d0 | ai_follow_manager_load | ai_follow_manager::Load |
| 437 | 0x0043e610 | FUN_0043e610 | ai_fight_manager_join_party | ai_fight_manager::JoinParty |
| 438 | 0x0043e870 | FUN_0043e870 | ai_fight_manager_leave_party | ai_fight_manager::LeaveParty |
| 439 | 0x0043ebc0 | FUN_0043ebc0 | ai_fight_followship_set_commander | ai_fight_followship::SetCommander |
| 440 | 0x0043ecc0 | FUN_0043ecc0 | ai_fight_followship_count_free_seats | ai_fight_followship::CountFreeSeats |
| 441 | 0x0043ee60 | FUN_0043ee60 | ai_fight_followship_take_car | ai_fight_followship::TakeCar |
| 442 | 0x0043f090 | FUN_0043f090 | ai_fight_manager_new_car_party | ai_fight_manager::NewCarParty |
| 443 | 0x0043f110 | FUN_0043f110 | ai_fight_followship_add_car_member | ai_fight_followship::AddCarMember |
| 444 | 0x0043f3f0 | FUN_0043f3f0 | ai_fight_followship_compute_position | ai_fight_followship::ComputePosition |
| 445 | 0x0043f590 | FUN_0043f590 | ai_fight_followship_must_car_hunt | ai_fight_followship::mustCarHunt |
| 446 | 0x0043f730 | FUN_0043f730 | ai_fight_manager_disband_followship | ai_fight_manager::DisbandCarParty |
| 447 | 0x0043f950 | FUN_0043f950 | ai_fight_manager_adjust_form_time | ai_fight_manager::AdjustFormTime |
| 448 | 0x0043f9b0 | FUN_0043f9b0 | ai_fight_manager_get_party | ai_fight_manager::GetParty |
| 449 | 0x0043fac0 | FUN_0043fac0 | ai_fight_manager_clear_all_parties | ai_fight_manager::Init |
| 450 | 0x0043fb90 | FUN_0043fb90 | ai_fight_manager_destroy | ai_fight_manager::Destroy |
| 451 | 0x0043fce0 | FUN_0043fce0 | ai_fight_manager_tick | ai_fight_manager::Tick |
| 452 | 0x00440e10 | FUN_00440e10 | ai_fight_manager_invalidate_actor | ai_fight_manager::Invalidate |
| 453 | 0x00441070 | FUN_00441070 | vec_count_stride_0x14 |  |
| 454 | 0x004410a0 | FUN_004410a0 | ai_group_destroy_range |  |
| 455 | 0x004410d0 | FUN_004410d0 | ai_fight_followship_vector_insert_at |  |
| 456 | 0x004412a0 | FUN_004412a0 | ai_fight_followship_vector_count |  |
| 457 | 0x004412d0 | FUN_004412d0 | ai_fight_followship_destroy_range |  |
| 458 | 0x00441300 | FUN_00441300 | ai_group_vector_insert_at |  |
| 459 | 0x00441620 | FUN_00441620 | ai_follow_group_vector_insert_at |  |
| 460 | 0x00441950 | FUN_00441950 | std_vector20_insert |  |
| 461 | 0x00441c50 | FUN_00441c50 | ai_fight_followship_vector_insert_n |  |
| 462 | 0x00441f90 | FUN_00441f90 | fill_n_stride_0x14 |  |
| 463 | 0x00441fc0 | FUN_00441fc0 | ai_fight_followship_uninit_copy_range |  |
| 464 | 0x00442000 | FUN_00442000 | ai_fight_followship_uninit_fill_n |  |
| 465 | 0x00442030 | FUN_00442030 | ai_fight_followship_assign_range |  |
| 466 | 0x00442280 | FUN_00442280 | ai_fight_followship_assign_range_backward |  |
| 467 | 0x004424e0 | FUN_004424e0 | ai_group_copy_ctor | ai_group::{ctor} |
| 468 | 0x004426c0 | FUN_004426c0 | stl_copy_dwords |  |
| 469 | 0x004426f0 | FUN_004426f0 | ai_follow_group_copy_ctor | ai_follow_group::{ctor} |
| 470 | 0x004427d0 | FUN_004427d0 | copy_element_stride_0x14 |  |
| 471 | 0x004427f0 | FUN_004427f0 | ai_fight_followship_copy_ctor | ai_fight_followship::{ctor} |
| 472 | 0x00442910 | FUN_00442910 | ai_group_copy_assign |  |
| 473 | 0x00442b30 | FUN_00442b30 | ai_follow_group_copy_assign |  |
| 474 | 0x00442cd0 | FUN_00442cd0 | ai_fight_followship_copy_assign | ai_fight_followship::operator= |
| 475 | 0x00442f50 | FUN_00442f50 | ai_group_scalar_deleting_dtor |  |
| 476 | 0x00442f70 | FUN_00442f70 | std_vector4_copy_assign |  |
| 477 | 0x004430f0 | FUN_004430f0 | std_vector8_copy_assign |  |
| 478 | 0x00443290 | FUN_00443290 | std_vector20_copy_assign |  |
| 479 | 0x004434f0 | FUN_004434f0 | ai_group_member_vector_capacity |  |
| 480 | 0x00443510 | FUN_00443510 | crt_static_init_guard_00443510 |  |
| 481 | 0x00443530 | FUN_00443530 | crt_atexit_register_00443530 |  |
| 482 | 0x00443540 | FUN_00443540 | ai_manager_static_init |  |
| 483 | 0x00443550 | FUN_00443550 | ai_manager_static_ctor |  |
| 484 | 0x00443560 | FUN_00443560 | ai_manager_register_atexit_dtor |  |
| 485 | 0x00443570 | FUN_00443570 | ai_manager_static_dtor |  |
| 486 | 0x00443580 | FUN_00443580 | ai_manager_dtor | ai_manager::{dtor} |
| 487 | 0x00443720 | FUN_00443720 | ai_manager_ctor |  |
| 488 | 0x00443840 | FUN_00443840 | ai_wayfind_dtor |  |
| 489 | 0x00443860 | FUN_00443860 | police_board_dtor |  |
| 490 | 0x00443910 | FUN_00443910 | ai_groups_dtor |  |
| 491 | 0x00443a20 | FUN_00443a20 | ai_follow_manager_dtor |  |
| 492 | 0x00443a90 | FUN_00443a90 | ai_fight_manager_dtor |  |
| 493 | 0x00443b00 | FUN_00443b00 | std_vector_construct_empty |  |
| 494 | 0x00443b20 | FUN_00443b20 | ai_group_vector_destroy |  |
| 495 | 0x00443b60 | FUN_00443b60 | police_action_destroy_range |  |
| 496 | 0x00443b90 | FUN_00443b90 | vec_owner_scalar_deleting_dtor |  |
| 497 | 0x00443bb0 | FUN_00443bb0 | crt_static_init_guard_00443bb0 |  |
| 498 | 0x00443bd0 | FUN_00443bd0 | crt_atexit_register_00443bd0 |  |
| 499 | 0x00443be0 | FUN_00443be0 | police_action_disband | ai_police_action::HosiKoncime |
| 500 | 0x00443d00 | FUN_00443d00 | police_action_broadcast_event | ai_police_action::EventVsem |
| 501 | 0x00443d40 | FUN_00443d40 | police_action_tick | ai_police_action::Tick |
| 502 | 0x004461d0 | FUN_004461d0 | police_action_find_officer_index | ai_police_action::KolikatejSemSefe |
| 503 | 0x00446200 | FUN_00446200 | police_action_tick_sighting_timers | ai_police_action::PanoveCekamVasNaWCSesynchronizujemSiCas |
| 504 | 0x004466e0 | FUN_004466e0 | police_action_dispatch_officer_commands | ai_police_action::TakZnovuDoToho |
| 505 | 0x00446930 | FUN_00446930 | police_action_on_vehicle_lost | ai_police_action::ZbavSeAutaSmrdi |
| 506 | 0x004469e0 | FUN_004469e0 | police_board_reset | ai_police_nastenka::ZacatekSmeny |
| 507 | 0x00446b00 | FUN_00446b00 | police_board_increment_counter | ai_police_nastenka::JaChcuCislo |
| 508 | 0x00446b10 | FUN_00446b10 | police_board_find_group_by_officer |  |
| 509 | 0x00446ba0 | FUN_00446ba0 | police_board_find_group_by_session_id |  |
| 510 | 0x00446c00 | FUN_00446c00 | police_board_create_and_engage | ai_police_nastenka::MamProVasAkciHosi |
| 511 | 0x004474a0 | FUN_004474a0 | police_board_end_pursuit | ai_police_nastenka::AkceDokoncenaPlukovniku |
| 512 | 0x00447610 | FUN_00447610 | police_board_invalidate_actor | ai_police_nastenka::ZinvalidujAktora |
| 513 | 0x004477b0 | FUN_004477b0 | police_board_tick | ai_police_nastenka::Tick |
| 514 | 0x00447820 | FUN_00447820 | police_board_arrest_settle_is_zero |  |
| 515 | 0x00447830 | FUN_00447830 | police_board_raise_gun_escalation | ai_police_nastenka::PlukovnikuRadsiHoSejmemNebudemSeSrat |
| 516 | 0x00447950 | FUN_00447950 | police_board_raise_handcuffs_escalation | ai_police_nastenka::PlukovnikuRadsiHoZatknemNebudemSeSrat |
| 517 | 0x00447a90 | FUN_00447a90 | police_board_disband_by_target |  |
| 518 | 0x00447b00 | FUN_00447b00 | police_board_clamp_arrest_settle_12s |  |
| 519 | 0x00447b10 | FUN_00447b10 | police_board_promote_handcuffs_escalation | ai_police_nastenka::PanoveZmenaZatkneteHoZmrda |
| 520 | 0x00447bb0 | FUN_00447bb0 | police_board_promote_gun_escalation | ai_police_nastenka::PanoveZmenaZabijteHoZmrda |
| 521 | 0x00447c40 | FUN_00447c40 | police_board_update_sighting | ai_police_nastenka::PanoveKokotPremistujePrdel |
| 522 | 0x00448150 | FUN_00448150 | police_board_target_matches | ai_police_nastenka::PanoveViditeZmrdaNeboSteTupci |
| 523 | 0x004481c0 | FUN_004481c0 | police_board_refresh_target_vehicle | ai_police_nastenka::PanoveVytahneteBlokyAZapisteSiZmrda |
| 524 | 0x00448230 | FUN_00448230 | police_board_has_recent_sighting |  |
| 525 | 0x00448300 | FUN_00448300 | police_board_has_pursuit_group_for_target | ai_police_nastenka::PanoveJdeteNekdoPoZmrdovi |
| 526 | 0x00448350 | FUN_00448350 | police_board_broadcast_for_target |  |
| 527 | 0x004483b0 | FUN_004483b0 | police_board_is_busted_cutscene_active |  |
| 528 | 0x004483c0 | FUN_004483c0 | police_board_set_busted_cutscene_active |  |
| 529 | 0x004483d0 | FUN_004483d0 | police_board_is_actor_in_pursuit | ai_police_nastenka::SemTrefenaLocklaKurva |
| 530 | 0x00448470 | FUN_00448470 | police_board_suspend_npcs |  |
| 531 | 0x00448550 | FUN_00448550 | police_board_resume_npcs |  |
| 532 | 0x00448620 | FUN_00448620 | police_board_has_active_group_for_target |  |
| 533 | 0x00448680 | FUN_00448680 | police_board_set_tracked_npc |  |
| 534 | 0x00448690 | FUN_00448690 | police_board_end_all_pursuits | ai_police_nastenka::KonecSmeny |
| 535 | 0x004486f0 | FUN_004486f0 | police_board_release_all_group_vehicles | ai_police_nastenka::ZastavMotorizovaneJednotky |
| 536 | 0x004487f0 | thunk_FUN_00448800 | thunk_ai_animitem_manager_clear |  |
| 537 | 0x00448800 | FUN_00448800 | ai_animitem_manager_clear | ai_animitem_manager::Clear |
| 538 | 0x00448900 | FUN_00448900 | ai_animitem_manager_init | ai_animitem_manager::Init |
| 539 | 0x00448940 | FUN_00448940 | i3d_cache_model_open | C_I3D_cache<9I3D_model>::Open |
| 540 | 0x00448db0 | FUN_00448db0 | string_base_init |  |
| 541 | 0x00448df0 | FUN_00448df0 | string_base_release |  |
| 542 | 0x00448e00 | FUN_00448e00 | crt_operator_delete_thunk_00448e00 |  |
| 543 | 0x00448e10 | FUN_00448e10 | string_dtor | C_string::{dtor} |
| 544 | 0x00448e30 | FUN_00448e30 | string_get_data | C_string::C_str_base::GetData |
| 545 | 0x00448e40 | FUN_00448e40 | string_append_cstr |  |
| 546 | 0x00448fc0 | FUN_00448fc0 | ascii_toupper |  |
| 547 | 0x00448fe0 | FUN_00448fe0 | string_format | C_fstring::{ctor} |
| 548 | 0x004490b0 | FUN_004490b0 | police_action_vector_count |  |
| 549 | 0x004490e0 | FUN_004490e0 | police_action_vector_insert_at | gif::vector<16ai_police_action,8>::reserve |
| 550 | 0x004493a0 | FUN_004493a0 | police_action_uninit_copy_range |  |
| 551 | 0x004493e0 | FUN_004493e0 | police_action_uninit_fill_n |  |
| 552 | 0x00449410 | FUN_00449410 | uninit_copy_range_stride_8 |  |
| 553 | 0x00449450 | FUN_00449450 | police_action_copy_assign_range |  |
| 554 | 0x00449620 | FUN_00449620 | police_action_copy_backward_range |  |
| 555 | 0x00449800 | FUN_00449800 | police_action_copy_ctor | ai_police_action::{ctor} |
| 556 | 0x00449930 | FUN_00449930 | copy_elem_stride_8_with_string |  |
| 557 | 0x00449970 | FUN_00449970 | police_action_assign | ai_police_action::operator= |
| 558 | 0x00449b60 | FUN_00449b60 | assign_elem_stride_8_with_string |  |
| 559 | 0x00449ba0 | FUN_00449ba0 | crt_static_init_guard_00449ba0 |  |
| 560 | 0x00449bc0 | FUN_00449bc0 | crt_atexit_register_00449bc0 |  |
| 561 | 0x00449bd0 | FUN_00449bd0 | check_seg_has_index | check_seg::existIndex |
| 562 | 0x00449c30 | FUN_00449c30 | check_seg_get_special_count | check_seg::NumSpecials |
| 563 | 0x00449c40 | FUN_00449c40 | check_seg_get_type_and_data | check_seg::GetType |
| 564 | 0x00449c60 | FUN_00449c60 | check_seg_get_special | check_seg::GetSpecial |
| 565 | 0x00449c90 | FUN_00449c90 | check_seg_is_pos_in_any_special | check_seg::isInSpecial |
| 566 | 0x00449d80 | FUN_00449d80 | check_seg_clusters_get_segment | check_seg_clusters::GetSegment |
| 567 | 0x00449dc0 | FUN_00449dc0 | check_seg_clusters_get_segment_count | check_seg_clusters::NumSegments |
| 568 | 0x00449e00 | FUN_00449e00 | check_seg_clusters_add_segment | check_seg_clusters::AddSegment |
| 569 | 0x0044a0f0 | FUN_0044a0f0 | check_seg_cluster_ctor | check_seg_cluster::{ctor} |
| 570 | 0x0044a110 | FUN_0044a110 | check_seg_cluster_dtor | check_seg_cluster::{dtor} |
| 571 | 0x0044a120 | FUN_0044a120 | folded_vector_ctor_at_0x04_0044a120 | check_seg::{ctor} |
| 572 | 0x0044a140 | FUN_0044a140 | check_seg_clusters_init | check_seg_clusters::Init |
| 573 | 0x0044a200 | FUN_0044a200 | check_seg_clusters_clear | check_seg_clusters::Clear |
| 574 | 0x0044a330 | FUN_0044a330 | check_seg_clusters_load | check_seg_clusters::Load |
| 575 | 0x0044a620 | FUN_0044a620 | crt_static_init_guard_0044a620 |  |
| 576 | 0x0044a640 | FUN_0044a640 | crt_atexit_register_0044a640 |  |
| 577 | 0x0044a650 | FUN_0044a650 | actor_track_create |  |
| 578 | 0x0044a670 | FUN_0044a670 | actor_shot_track_create |  |
| 579 | 0x0044a690 | FUN_0044a690 | actor_shot_track_apply_key_effects |  |
| 580 | 0x0044a780 | FUN_0044a780 | stub_retminus1_args4_0044a780 |  |
| 581 | 0x0044a790 | FUN_0044a790 | actor_track_apply_key_texture_anim |  |
| 582 | 0x0044a860 | FUN_0044a860 | actor_track_apply_pose_at_clock |  |
| 583 | 0x0044ad10 | FUN_0044ad10 | actor_track_apply_pose_at_time |  |
| 584 | 0x0044b250 | FUN_0044b250 | actor_track_get_pos_at_time |  |
| 585 | 0x0044b470 | FUN_0044b470 | actor_track_get_rot_at_time |  |
| 586 | 0x0044b5d0 | FUN_0044b5d0 | actor_global_list_static_init |  |
| 587 | 0x0044b5e0 | FUN_0044b5e0 | actor_global_list_ctor |  |
| 588 | 0x0044b600 | FUN_0044b600 | actor_global_list_register_atexit_dtor |  |
| 589 | 0x0044b610 | FUN_0044b610 | actor_global_list_dtor |  |
| 590 | 0x0044b650 | FUN_0044b650 | actor_posscoll_table_static_init |  |
| 591 | 0x0044b660 | FUN_0044b660 | actor_posscoll_table_ctor |  |
| 592 | 0x0044b680 | FUN_0044b680 | actor_posscoll_table_register_atexit_dtor |  |
| 593 | 0x0044b690 | FUN_0044b690 | actor_posscoll_table_dtor |  |
| 594 | 0x0044b6d0 | FUN_0044b6d0 | actor_ctor | C_actor::{ctor} |
| 595 | 0x0044b720 | FUN_0044b720 | actor_scalar_deleting_dtor |  |
| 596 | 0x0044b740 | FUN_0044b740 | actor_dtor | C_actor::{dtor} |
| 597 | 0x0044b780 | FUN_0044b780 | actor_init | C_actor::Init |
| 598 | 0x0044b810 | FUN_0044b810 | actor_internal_placement_reset | C_actor::InternalPlacementReset |
| 599 | 0x0044b890 | FUN_0044b890 | actor_reset_placement | C_actor::ResetPlacement |
| 600 | 0x0044b950 | FUN_0044b950 | actor_game_init | C_actor::GameInit |
| 601 | 0x0044bd90 | FUN_0044bd90 | actor_game_done | C_actor::GameDone |
| 602 | 0x0044bea0 | FUN_0044bea0 | actor_tick | C_actor::Tick |
| 603 | 0x0044bf40 | FUN_0044bf40 | actor_ref_save_get_size | C_actor::Actor_SaveGameGetSize |
| 604 | 0x0044bfa0 | FUN_0044bfa0 | actor_ref_save_write | C_actor::Actor_SaveGameSave |
| 605 | 0x0044c050 | FUN_0044c050 | actor_ref_save_resolve | C_actor::Actor_SaveGameLoad |
| 606 | 0x0044c120 | FUN_0044c120 | actor_save_game_get_size | C_actor::SaveGameGetSize |
| 607 | 0x0044c220 | FUN_0044c220 | actor_save_game_save | C_actor::SaveGameSave |
| 608 | 0x0044c4b0 | FUN_0044c4b0 | actor_save_game_load | C_actor::SaveGameLoad |
| 609 | 0x0044c730 | FUN_0044c730 | actor_save_game_load_guid | C_actor::SaveGameLoadGuid |
| 610 | 0x0044c750 | FUN_0044c750 | actor_set_act_state | C_actor::SetActState |
| 611 | 0x0044c820 | FUN_0044c820 | actor_rebirth | C_actor::Rebirth |
| 612 | 0x0044c840 | FUN_0044c840 | actor_is_possible_coll_all | C_actor::IsPossibleCollAll |
| 613 | 0x0044cf90 | FUN_0044cf90 | stub_retfalse_args40_0044cf90 |  |
| 614 | 0x0044cfa0 | FUN_0044cfa0 | actor_get_bound_points_2d | C_actor::GetBoundPoints2D |
| 615 | 0x0044d6d0 | FUN_0044d6d0 | actor_debustize_frame | C_actor::DebustizeFrame |
| 616 | 0x0044d710 | FUN_0044d710 | actor_no_shadow_frame | C_actor::NoShadowFrame |
| 617 | 0x0044d750 | FUN_0044d750 | actor_add_to_global_list | C_actor::AddActorToGlobal |
| 618 | 0x0044d950 | FUN_0044d950 | actor_del_from_global_list | C_actor::DelActorFromGlobal |
| 619 | 0x0044d9b0 | FUN_0044d9b0 | actor_is_in_global_list |  |
| 620 | 0x0044d9e0 | FUN_0044d9e0 | actor_get_world_position | C_actor::GetWorldPos |
| 621 | 0x0044da40 | FUN_0044da40 | actor_get_forward_normalized | C_actor::GetWorldDir |
| 622 | 0x0044dbd0 | FUN_0044dbd0 | actor_add_frame_for_possible_coll | C_actor::AddFrameForPossibleColl |
| 623 | 0x0044dde0 | FUN_0044dde0 | actor_del_frame_for_possible_coll | C_actor::DelFrameForPossibleColl |
| 624 | 0x0044de50 | FUN_0044de50 | rand_range | rnd |
| 625 | 0x0044de70 | FUN_0044de70 | rand_float | rnd |
| 626 | 0x0044de90 | FUN_0044de90 | vec2_signed_angle_diff | AngleTo2 |
| 627 | 0x0044ded0 | FUN_0044ded0 | vec3_rotate_around_y_xz_unit_in_place |  |
| 628 | 0x0044e030 | FUN_0044e030 | vec3_rotate_around_y_in_place |  |
| 629 | 0x0044e090 | FUN_0044e090 | compare_anim_index | compare_anim_index |
| 630 | 0x0044e0b0 | FUN_0044e0b0 | anim_index_init | InitAnimIndex |
| 631 | 0x0044e150 | FUN_0044e150 | anim_index_free | DoneAnimIndex |
| 632 | 0x0044e160 | FUN_0044e160 | anim_index_of_id | GetAnimIndex |
| 633 | 0x0044e1b0 | FUN_0044e1b0 | actor_post_hit_event |  |
| 634 | 0x0044e300 | FUN_0044e300 | actor_post_collision_event |  |
| 635 | 0x0044e440 | FUN_0044e440 | i3d_cache_sound_open | C_I3D_cache<9I3D_sound>::Open |
| 636 | 0x0044e880 | FUN_0044e880 | refcount_release_at_0x4 |  |
| 637 | 0x0044e8a0 | FUN_0044e8a0 | actor_get_actor_speed | C_actor::GetActorSpeed |
| 638 | 0x0044e8c0 | FUN_0044e8c0 | destroy_range_stride8 |  |
| 639 | 0x0044e8f0 | FUN_0044e8f0 | std_vector8_uninit_fill_n |  |
| 640 | 0x0044e920 | FUN_0044e920 | std_vector8_assign_range |  |
| 641 | 0x0044e970 | FUN_0044e970 | crt_static_init_guard_0044e970 |  |
| 642 | 0x0044e990 | FUN_0044e990 | crt_atexit_register_0044e990 |  |
| 643 | 0x0044e9a0 | FUN_0044e9a0 | actor_shot_ctor | C_actor_shot::{ctor} |
| 644 | 0x0044e9c0 | FUN_0044e9c0 | actor_shot_scalar_deleting_dtor |  |
| 645 | 0x0044e9e0 | FUN_0044e9e0 | actor_shot_dtor | C_actor_shot::{dtor} |
| 646 | 0x0044e9f0 | FUN_0044e9f0 | actor_shot_init | C_actor_shot::Init |
| 647 | 0x0044ea30 | FUN_0044ea30 | actor_shot_update | C_actor_shot::Update |
| 648 | 0x0044ed90 | FUN_0044ed90 | actor_shot_game_init | C_actor_shot::GameInit |
| 649 | 0x0044ee50 | FUN_0044ee50 | actor_shot_game_done | C_actor_shot::GameDone |
| 650 | 0x0044ee90 | FUN_0044ee90 | actor_shot_set_weapon | C_actor_shot::SetWeapon |
| 651 | 0x0044ef20 | FUN_0044ef20 | crt_static_init_guard_0044ef20 |  |
| 652 | 0x0044ef40 | FUN_0044ef40 | crt_atexit_register_0044ef40 |  |
| 653 | 0x0044ef50 | FUN_0044ef50 | airplane_ctor | C_airplane::{ctor} |
| 654 | 0x0044f0c0 | FUN_0044f0c0 | airplane_scalar_deleting_dtor |  |
| 655 | 0x0044f0e0 | FUN_0044f0e0 | airplane_dtor | C_airplane::{dtor} |
| 656 | 0x0044f140 | FUN_0044f140 | airplane_save_data | C_airplane::SaveData |
| 657 | 0x0044f380 | FUN_0044f380 | airplane_load_data | C_airplane::LoadData |
| 658 | 0x0044f600 | FUN_0044f600 | airplane_duplicate_from | C_airplane::DuplicateFrom |
| 659 | 0x0044f6c0 | FUN_0044f6c0 | airplane_game_init | C_airplane::GameInit |
| 660 | 0x004503c0 | FUN_004503c0 | airplane_game_done | C_airplane::GameDone |
| 661 | 0x004504a0 | FUN_004504a0 | airplane_pick_next_web_node | C_airplane::NextNode |
| 662 | 0x00450990 | FUN_00450990 | airplane_fly_along_route | C_airplane::AI_fly |
| 663 | 0x00451540 | FUN_00451540 | airplane_spin_propellers | C_airplane::Vrtule |
| 664 | 0x004516e0 | FUN_004516e0 | airplane_ai_tick | C_airplane::AI |
| 665 | 0x00451990 | FUN_00451990 | airplane_update | C_airplane::Update |
| 666 | 0x00452000 | FUN_00452000 | quat_multiply |  |
| 667 | 0x004520e0 | FUN_004520e0 | airplane_start_ground_run |  |
| 668 | 0x004522c0 | FUN_004522c0 | airplane_begin_takeoff |  |
| 669 | 0x004522e0 | FUN_004522e0 | airplane_apply_propeller_visibility | C_airplane::InitVrtule |
| 670 | 0x00452350 | FUN_00452350 | airplane_hit | C_airplane::Hit |
| 671 | 0x004525d0 | FUN_004525d0 | airplane_update_engine_particles | C_airplane::UpdateParticles |
| 672 | 0x00452810 | FUN_00452810 | airplane_get_engines_pos | C_airplane::GetEnginesPos |
| 673 | 0x00452e70 | FUN_00452e70 | airplane_get_engine_local_offsets |  |
| 674 | 0x00453090 | FUN_00453090 | airplane_create_sounds | C_airplane::InitSounds |
| 675 | 0x00453150 | FUN_00453150 | airplane_destroy_sounds | C_airplane::ReleaseSounds |
| 676 | 0x00453190 | FUN_00453190 | airplane_is_destroyed |  |
| 677 | 0x004531c0 | FUN_004531c0 | airplane_save_game_get_size | C_airplane::SaveGameGetSize |
| 678 | 0x004531d0 | FUN_004531d0 | airplane_save_game_save | C_airplane::SaveGameSave |
| 679 | 0x00453300 | FUN_00453300 | airplane_save_game_load | C_airplane::SaveGameLoad |
| 680 | 0x00453450 | FUN_00453450 | airplane_play_sound | C_airplane::PlaySound |
| 681 | 0x004534c0 | FUN_004534c0 | airplane_tick_sounds | C_airplane::TickSounds |
| 682 | 0x004537c0 | FUN_004537c0 | airplane_on_act_state_changed | C_airplane::ChangeState |
| 683 | 0x004538b0 | FUN_004538b0 | airplane_hud_damage_toggle | C_airplane::ShowDamageIndicator |
| 684 | 0x00453900 | FUN_00453900 | i3d_frame_build_scaled_rot_matrix |  |
| 685 | 0x00453a10 | FUN_00453a10 | airplane_get_material_id | C_airplane::GetMaterialID |
| 686 | 0x00453a20 | FUN_00453a20 | crt_static_init_guard_00453a20 |  |
| 687 | 0x00453a40 | FUN_00453a40 | crt_atexit_register_00453a40 |  |
| 688 | 0x00453a50 | FUN_00453a50 | chunk_read | C_chunk::Read |
| 689 | 0x00453ad0 | FUN_00453ad0 | bottle_ctor | C_bottle::{ctor} |
| 690 | 0x00453b10 | FUN_00453b10 | bottle_scalar_deleting_dtor |  |
| 691 | 0x00453b30 | FUN_00453b30 | bottle_dtor | C_bottle::{dtor} |
| 692 | 0x00453b40 | FUN_00453b40 | bottle_init | C_bottle::Init |
| 693 | 0x00453b70 | FUN_00453b70 | bottle_ai_tick | C_bottle::AI |
| 694 | 0x00454840 | FUN_00454840 | bottle_update | C_bottle::Update |
| 695 | 0x00454920 | FUN_00454920 | bottle_net_apply_remote_state | C_bottle::NetDirect |
| 696 | 0x00455160 | FUN_00455160 | bottle_game_init | C_bottle::GameInit |
| 697 | 0x00455480 | FUN_00455480 | vec3_div_scalar |  |
| 698 | 0x004554b0 | FUN_004554b0 | bottle_game_done | C_bottle::GameDone |
| 699 | 0x004554e0 | FUN_004554e0 | bottle_save_game_get_size | C_bottle::SaveGameGetSize |
| 700 | 0x004554f0 | FUN_004554f0 | bottle_save_game_save | C_bottle::SaveGameSave |
| 701 | 0x004555d0 | FUN_004555d0 | bottle_save_game_load | C_bottle::SaveGameLoad |
| 702 | 0x00455710 | FUN_00455710 | bottle_set_flight_impulse | C_bottle::AbsoluteImpulse |
| 703 | 0x004558d0 | FUN_004558d0 | bottle_on_collision | C_bottle::Collision |
| 704 | 0x00455bc0 | FUN_00455bc0 | bottle_hit | C_bottle::Hit |
| 705 | 0x00455eb0 | FUN_00455eb0 | bottle_apply_net_event_msg |  |
| 706 | 0x00456200 | FUN_00456200 | bottle_post_state_net_event_msg |  |
| 707 | 0x00456310 | FUN_00456310 | bottle_post_collision_net_event_msg |  |
| 708 | 0x00456460 | FUN_00456460 | crt_static_init_guard_00456460 |  |
| 709 | 0x00456480 | FUN_00456480 | crt_atexit_register_00456480 |  |
| 710 | 0x00456490 | thunk_FUN_004564a0 | thunk_box_static_init_gravity |  |
| 711 | 0x004564a0 | FUN_004564a0 | box_static_init_gravity |  |
| 712 | 0x004564c0 | FUN_004564c0 | box_wake_resting_bodies |  |
| 713 | 0x00456530 | FUN_00456530 | box_add_impulse_at_point |  |
| 714 | 0x00456630 | FUN_00456630 | box_get_contact_state_at_point |  |
| 715 | 0x00456720 | FUN_00456720 | box_solve_impulse_vs_box |  |
| 716 | 0x00456b00 | FUN_00456b00 | box_solve_impulse_vs_world |  |
| 717 | 0x00456e60 | FUN_00456e60 | box_apply_impulse_pair_at_point |  |
| 718 | 0x00456fb0 | FUN_00456fb0 | box_on_collision |  |
| 719 | 0x00457350 | FUN_00457350 | box_hit |  |
| 720 | 0x00457540 | FUN_00457540 | box_accumulate_forces |  |
| 721 | 0x00458000 | FUN_00458000 | vec3_mul_componentwise |  |
| 722 | 0x00458030 | FUN_00458030 | matrix3_rotate_by_angular_velocity |  |
| 723 | 0x004580a0 | FUN_004580a0 | matrix4_from_transposed_basis |  |
| 724 | 0x00458130 | FUN_00458130 | matrix3_orthonormalize |  |
| 725 | 0x004585b0 | FUN_004585b0 | box_integrate_state |  |
| 726 | 0x00458830 | FUN_00458830 | box_solve_impulse_vs_static |  |
| 727 | 0x00458a60 | FUN_00458a60 | box_contact_slot_acquire |  |
| 728 | 0x00458bc0 | FUN_00458bc0 | box_contact_slot_release |  |
| 729 | 0x00458c30 | thunk_FUN_00458c40 | thunk_stub_noop_cdecl_00458c40 |  |
| 730 | 0x00458c40 | FUN_00458c40 | stub_noop_cdecl_00458c40 |  |
| 731 | 0x00458c50 | thunk_FUN_00458c60 | thunk_stub_noop_cdecl_00458c60 |  |
| 732 | 0x00458c60 | FUN_00458c60 | stub_noop_cdecl_00458c60 |  |
| 733 | 0x00458c70 | FUN_00458c70 | box_build_inverse_inertia_tensor |  |
| 734 | 0x004592f0 | FUN_004592f0 | obb_vs_obb_separating_axis_test |  |
| 735 | 0x00459820 | FUN_00459820 | box_generate_contacts_vs_box |  |
| 736 | 0x00459cf0 | FUN_00459cf0 | vec3_equals |  |
| 737 | 0x00459d30 | FUN_00459d30 | box_collect_contacts |  |
| 738 | 0x0045a8c0 | FUN_0045a8c0 | box_resolve_contacts |  |
| 739 | 0x0045ae00 | FUN_0045ae00 | box_ctor |  |
| 740 | 0x0045ae80 | FUN_0045ae80 | box_scalar_deleting_dtor |  |
| 741 | 0x0045aea0 | FUN_0045aea0 | identity_thunk |  |
| 742 | 0x0045aeb0 | FUN_0045aeb0 | box_init |  |
| 743 | 0x0045afb0 | FUN_0045afb0 | box_recompute_dimensions_from_frame |  |
| 744 | 0x0045b300 | FUN_0045b300 | box_dtor |  |
| 745 | 0x0045b360 | FUN_0045b360 | box_build_dyn_collision_from_frame |  |
| 746 | 0x0045baf0 | FUN_0045baf0 | box_game_init |  |
| 747 | 0x0045c0d0 | FUN_0045c0d0 | box_game_done |  |
| 748 | 0x0045c0f0 | FUN_0045c0f0 | box_save_data |  |
| 749 | 0x0045c150 | FUN_0045c150 | box_load_data |  |
| 750 | 0x0045c240 | FUN_0045c240 | box_ai_tick |  |
| 751 | 0x0045dcf0 | FUN_0045dcf0 | box_write_state_to_frame |  |
| 752 | 0x0045ddf0 | FUN_0045ddf0 | box_track_create |  |
| 753 | 0x0045de10 | FUN_0045de10 | box_track_record_key |  |
| 754 | 0x0045e1d0 | FUN_0045e1d0 | box_clamp_state_matrix |  |
| 755 | 0x0045e220 | FUN_0045e220 | box_post_state_net_event_msg |  |
| 756 | 0x0045e280 | FUN_0045e280 | box_apply_net_event_msg |  |
| 757 | 0x0045e5a0 | FUN_0045e5a0 | box_net_apply_remote_state |  |
| 758 | 0x0045f000 | FUN_0045f000 | i3d_frame_set_position |  |
| 759 | 0x0045f040 | FUN_0045f040 | i3d_frame_set_orientation |  |
| 760 | 0x0045f090 | FUN_0045f090 | i3d_frame_get_scale |  |
| 761 | 0x0045f0f0 | FUN_0045f0f0 | stub_zero_vec3_out |  |
| 762 | 0x0045f110 | FUN_0045f110 | stub_zero_quat_out |  |
| 763 | 0x0045f130 | FUN_0045f130 | crt_static_init_guard_0045f130 |  |
| 764 | 0x0045f150 | FUN_0045f150 | crt_atexit_register_0045f150 |  |
| 765 | 0x0045f160 | FUN_0045f160 | box_new_ctor | C_box_new::{ctor} |
| 766 | 0x0045f180 | FUN_0045f180 | box_new_scalar_deleting_dtor |  |
| 767 | 0x0045f1a0 | FUN_0045f1a0 | box_new_dtor | C_box_new::{dtor} |
| 768 | 0x0045f1b0 | FUN_0045f1b0 | box_new_init | C_box_new::Init |
| 769 | 0x0045f2c0 | FUN_0045f2c0 | box_new_game_init | C_box_new::GameInit |
| 770 | 0x0045f390 | thunk_FUN_004bf0c0 | thunk_physical_object_game_done |  |
| 771 | 0x0045f3a0 | FUN_0045f3a0 | box_new_load_data | C_box_new::LoadData |
| 772 | 0x0045f720 | FUN_0045f720 | box_new_save_data | C_box_new::SaveData |
| 773 | 0x0045f890 | FUN_0045f890 | box_new_duplicate_from | C_box_new::DuplicateFrom |
| 774 | 0x0045f8f0 | FUN_0045f8f0 | box_new_on_collision | C_box_new::Collision |
| 775 | 0x0045f910 | FUN_0045f910 | box_new_hit | C_box_new::Hit |
| 776 | 0x0045f930 | FUN_0045f930 | crt_static_init_guard_0045f930 |  |
| 777 | 0x0045f950 | FUN_0045f950 | crt_atexit_register_0045f950 |  |
| 778 | 0x0045f960 | thunk_FUN_0045f970 | thunk_stub_noop_cdecl_0045f970 |  |
| 779 | 0x0045f970 | FUN_0045f970 | stub_noop_cdecl_0045f970 |  |
| 780 | 0x0045f980 | FUN_0045f980 | bridge_ctor | C_bridge::{ctor} |
| 781 | 0x0045fa70 | FUN_0045fa70 | bridge_scalar_deleting_dtor |  |
| 782 | 0x0045fa90 | FUN_0045fa90 | bridge_dtor | C_bridge::{dtor} |
| 783 | 0x0045fae0 | FUN_0045fae0 | bridge_init | C_bridge::Init |
| 784 | 0x0045fc20 | FUN_0045fc20 | bridge_ai | C_bridge::AI |
| 785 | 0x004601b0 | FUN_004601b0 | bridge_update | C_bridge::Update |
| 786 | 0x00461180 | FUN_00461180 | stub_noop_args4_00461180 |  |
| 787 | 0x00461190 | FUN_00461190 | bridge_game_init | C_bridge::GameInit |
| 788 | 0x00462280 | FUN_00462280 | bridge_game_done | C_bridge::GameDone |
| 789 | 0x00462330 | FUN_00462330 | stub_rettrue_args4_00462330 |  |
| 790 | 0x00462340 | FUN_00462340 | bridge_load_data | C_bridge::LoadData |
| 791 | 0x00462420 | FUN_00462420 | stub_ret0_cdecl_00462420 |  |
| 792 | 0x00462430 | FUN_00462430 | bridge_save_game_save | C_bridge::SaveGameSave |
| 793 | 0x00462450 | FUN_00462450 | bridge_shut_down | C_bridge::ShutDown |
| 794 | 0x00462480 | FUN_00462480 | bridge_collision | C_bridge::Collision |
| 795 | 0x004624e0 | FUN_004624e0 | bridge_set_steamboat_figures |  |
| 796 | 0x004626b0 | FUN_004626b0 | string_assign_cstr |  |
| 797 | 0x00462730 | FUN_00462730 | bridge_col_vector_insert_n |  |
| 798 | 0x00462a60 | FUN_00462a60 | bridge_col_vector_copy |  |
| 799 | 0x00462a90 | FUN_00462a90 | crt_static_init_guard_00462a90 |  |
| 800 | 0x00462ab0 | FUN_00462ab0 | crt_atexit_register_00462ab0 |  |
| 801 | 0x00462ac0 | thunk_FUN_00462ad0 | thunk_car_static_state_init |  |
| 802 | 0x00462ad0 | FUN_00462ad0 | car_static_state_init |  |
| 803 | 0x00462af0 | FUN_00462af0 | car_ctor | C_car::{ctor} |
| 804 | 0x00462c20 | FUN_00462c20 | car_scalar_deleting_dtor |  |
| 805 | 0x00462c40 | FUN_00462c40 | car_dtor | C_car::{dtor} |
| 806 | 0x00462d30 | FUN_00462d30 | car_init | C_car::Init |
| 807 | 0x00463500 | FUN_00463500 | car_seat_get_anim_placement | C_car::GetPlacementFromTCK |
| 808 | 0x00463630 | FUN_00463630 | car_seat_get_attach_positions | C_car::GetPlacementFromTCK1 |
| 809 | 0x00463740 | FUN_00463740 | car_seat_index_of_usable | C_car::GetUSindex |
| 810 | 0x004637a0 | FUN_004637a0 | car_load_materials | C_car::LoadMaterials |
| 811 | 0x00463a30 | FUN_00463a30 | car_load_particle_effects | C_car::LoadParticles |
| 812 | 0x00464430 | FUN_00464430 | car_load_tyre_surfaces | C_car::LoadTyres |
| 813 | 0x00464720 | FUN_00464720 | car_convert_tyre_surface | C_car::ConvertTyres |
| 814 | 0x004647c0 | FUN_004647c0 | car_set_owner | C_car::SetOwner |
| 815 | 0x00464bd0 | FUN_00464bd0 | string_ctor_from_cstr |  |
| 816 | 0x00464c50 | FUN_00464c50 | car_get_seat_property | C_car::GetSeatProperty |
| 817 | 0x00464ee0 | FUN_00464ee0 | car_get_owner | C_car::GetOwner |
| 818 | 0x00464f40 | FUN_00464f40 | car_set_has_driver | C_car::SetDriver |
| 819 | 0x00464f50 | FUN_00464f50 | car_copy_carinit_from_actor | C_car::DuplicateFrom |
| 820 | 0x00464fa0 | FUN_00464fa0 | car_save_data_chunk | C_car::SaveData |
| 821 | 0x00465090 | FUN_00465090 | car_load_data_chunk | C_car::LoadData |
| 822 | 0x00465290 | FUN_00465290 | car_load_model_level_tables | C_car::LoadLevelTables |
| 823 | 0x004656b0 | FUN_004656b0 | string_wildcard_match |  |
| 824 | 0x00465760 | FUN_00465760 | car_repair_position_on_ground | C_car::_RepairPosition |
| 825 | 0x00465a80 | FUN_00465a80 | car_set_is_traffic_car | C_car::SetTrafficCar |
| 826 | 0x00465a90 | FUN_00465a90 | car_get_model_level_count | C_car::GetLevelsCount |
| 827 | 0x00465ac0 | FUN_00465ac0 | car_get_model_level | C_car::GetActualLevel |
| 828 | 0x00465ad0 | FUN_00465ad0 | car_set_model_level | C_car::SetActualLevel |
| 829 | 0x00465bd0 | FUN_00465bd0 | car_create_force_feedback_effects |  |
| 830 | 0x00465d30 | FUN_00465d30 | car_release_force_feedback_effects |  |
| 831 | 0x00465d90 | FUN_00465d90 | car_game_init | C_car::GameInit |
| 832 | 0x00466680 | FUN_00466680 | car_game_done | C_car::GameDone |
| 833 | 0x00466840 | FUN_00466840 | car_ai_tick | C_car::AI |
| 834 | 0x00466b80 | FUN_00466b80 | car_update | C_car::Update |
| 835 | 0x00468970 | FUN_00468970 | car_recompute_velocity |  |
| 836 | 0x00468bc0 | FUN_00468bc0 | car_explosion | C_car::CarExplosion |
| 837 | 0x004699c0 | FUN_004699c0 | car_deactivate | C_car::DeactivateCar |
| 838 | 0x00469ad0 | FUN_00469ad0 | car_reset | C_car::Reset |
| 839 | 0x00469dd0 | FUN_00469dd0 | car_reset_seats_and_routes | C_car::ResetCar |
| 840 | 0x00469fd0 | FUN_00469fd0 | car_is_point_in_path_ahead | C_car::ChcesMeSejmout |
| 841 | 0x0046a230 | FUN_0046a230 | car_pack_light_state |  |
| 842 | 0x0046a270 | FUN_0046a270 | car_unpack_light_state |  |
| 843 | 0x0046a3b0 | FUN_0046a3b0 | car_set_shadow_receiver_frames | C_car::SetShadowsToMe |
| 844 | 0x0046a450 | FUN_0046a450 | car_set_transparency | C_car::SetTransparency |
| 845 | 0x0046a5b0 | FUN_0046a5b0 | car_get_surface_material_id | C_car::GetMaterialID |
| 846 | 0x0046a670 | FUN_0046a670 | car_hit | C_car::Hit |
| 847 | 0x0046b8a0 | FUN_0046b8a0 | car_detect_wheel | C_car::DetectWheel |
| 848 | 0x0046ba50 | FUN_0046ba50 | car_is_possible_collision | C_car::IsPossibleColl |
| 849 | 0x0046c1b0 | FUN_0046c1b0 | car_on_collision_with_actor | C_car::Collision |
| 850 | 0x0046c360 | FUN_0046c360 | car_get_collision_info | C_car::Actor_Col_Info |
| 851 | 0x0046c6e0 | FUN_0046c6e0 | car_react_to_and_report_collision | C_car::Call_Collision |
| 852 | 0x0046c910 | FUN_0046c910 | car_react_to_collision | C_car::Call_Collision1 |
| 853 | 0x0046cbd0 | FUN_0046cbd0 | car_collision_filter_actor | C_car::Actor_Filter |
| 854 | 0x0046cc40 | FUN_0046cc40 | car_on_collision_apply_damage | C_car::Was_Collision |
| 855 | 0x0046d520 | FUN_0046d520 | car_add_deform_scratch | C_car::Deform_Vehicle_Part |
| 856 | 0x0046d550 | FUN_0046d550 | car_clear_damage_decals | C_car::Vehicle_Repair |
| 857 | 0x0046d580 | FUN_0046d580 | car_break_glass | C_car::Break_Glass |
| 858 | 0x0046d620 | FUN_0046d620 | car_wake_up | C_car::Wake_Up |
| 859 | 0x0046d630 | FUN_0046d630 | car_on_body_surface_contact | C_car::Collision_Filter_Body |
| 860 | 0x0046e080 | FUN_0046e080 | car_collision_filter_wheel | C_car::Collision_Filter_Wheel |
| 861 | 0x0046e0c0 | FUN_0046e0c0 | car_prepare_wheel_dropout | C_car::Prepare_DropOut_Wheel |
| 862 | 0x0046e1b0 | FUN_0046e1b0 | car_prepare_body_part_dropout | C_car::Prepare_DropOut |
| 863 | 0x0046e300 | FUN_0046e300 | car_drop_out_part | C_car::Drop_Out |
| 864 | 0x0046e660 | FUN_0046e660 | car_pack_save_record | C_car::WriteSaveStruct |
| 865 | 0x0046f6f0 | FUN_0046f6f0 | car_unpack_save_record | C_car::ReadSaveStruct |
| 866 | 0x00470750 | FUN_00470750 | car_get_inverse_model_matrix | C_car::GetInvModelMatrix |
| 867 | 0x004707c0 | FUN_004707c0 | car_get_condition_fraction | C_car::ShowCarEnergy |
| 868 | 0x00470880 | FUN_00470880 | collision_earliest_contact_between_spheres | g_collision::TestSphereSphere |
| 869 | 0x004709f0 | FUN_004709f0 | car_get_ai_probe_point | C_car::GetAIPos |
| 870 | 0x00470a60 | FUN_00470a60 | car_set_enter_protect | C_car::SetEnterProtect |
| 871 | 0x00470c50 | FUN_00470c50 | car_set_enter_protect1 | C_car::SetEnterProtect1 |
| 872 | 0x00470c90 | FUN_00470c90 | car_get_wheel_coll | C_car::GetCarWheelColl |
| 873 | 0x00470ce0 | FUN_00470ce0 | car_set_wheel_coll | C_car::SetCarWheelColl |
| 874 | 0x00470d30 | FUN_00470d30 | car_is_usable | C_car::IsCarUsable |
| 875 | 0x00470ea0 | FUN_00470ea0 | car_get_navigation_corners | C_car::GetNavigationPoints |
| 876 | 0x00470f60 | FUN_00470f60 | car_on_act_state_changed | C_car::ChangeState |
| 877 | 0x00471090 | FUN_00471090 | car_set_shadow_enabled | C_car::ShadowCarOn |
| 878 | 0x00471140 | FUN_00471140 | car_get_actor_speed | C_car::GetActorSpeed |
| 879 | 0x00471160 | FUN_00471160 | car_enable_sounds_for_scene | C_car::SoundsChange |
| 880 | 0x004711a0 | FUN_004711a0 | car_mark_parking_start_time | C_car::SetTimeParking |
| 881 | 0x004711c0 | FUN_004711c0 | car_get_parked_duration | C_car::GetTimeParking |
| 882 | 0x004711e0 | FUN_004711e0 | car_prepare_showroom_model | C_car::PrepareCarToShow |
| 883 | 0x00472d80 | FUN_00472d80 | car_is_seat_left | C_car::IsSeatLeft |
| 884 | 0x00472de0 | FUN_00472de0 | car_set_roof_visible | C_car::OdjebajTuStrechu |
| 885 | 0x00472ea0 | FUN_00472ea0 | car_refresh_player_is_driver_flag | C_car::RefreshActivePlayer |
| 886 | 0x00472f00 | FUN_00472f00 | car_set_force_feedback_gains |  |
| 887 | 0x00472f40 | FUN_00472f40 | car_save_game_get_size | C_car::SaveGameGetSize |
| 888 | 0x00472f60 | FUN_00472f60 | car_save_game_save | C_car::SaveGameSave |
| 889 | 0x00473130 | FUN_00473130 | car_save_game_load | C_car::SaveGameLoad |
| 890 | 0x00473380 | FUN_00473380 | car_track_record_key |  |
| 891 | 0x00473590 | FUN_00473590 | car_track_spawn_dropout_part |  |
| 892 | 0x00473750 | FUN_00473750 | car_track_apply_pose_at_clock |  |
| 893 | 0x00473b20 | FUN_00473b20 | car_track_create |  |
| 894 | 0x00473b40 | FUN_00473b40 | car_track_apply_pose_at_time |  |
| 895 | 0x00473e40 | FUN_00473e40 | car_track_get_pos_at_time |  |
| 896 | 0x00473f90 | FUN_00473f90 | i3d_frame_match_deform_roof |  |
| 897 | 0x004741d0 | FUN_004741d0 | car_carinit_vector_count |  |
| 898 | 0x00474200 | FUN_00474200 | vec_count_stride_0x8 |  |
| 899 | 0x00474220 | FUN_00474220 | vec_count_stride_0x4 |  |
| 900 | 0x00474240 | FUN_00474240 | vec_count_stride_0x18 |  |
| 901 | 0x00474270 | FUN_00474270 | vector_erase_range_dwords |  |
| 902 | 0x004742a0 | FUN_004742a0 | car_material_vector_insert_n |  |
| 903 | 0x00474590 | FUN_00474590 | car_seat_vector_insert_n |  |
| 904 | 0x004748a0 | FUN_004748a0 | car_tyre_surface_vector_insert_n |  |
| 905 | 0x00474bd0 | FUN_00474bd0 | stl_uninit_copy_stride_0x1c |  |
| 906 | 0x00474c10 | FUN_00474c10 | stl_copy_back_stride_0x18 |  |
| 907 | 0x00474c40 | FUN_00474c40 | car_carinit_vector_copy |  |
| 908 | 0x00474c80 | FUN_00474c80 | car_carinit_vector_copy_back |  |
| 909 | 0x00474cc0 | FUN_00474cc0 | crt_static_init_guard_00474cc0 |  |
| 910 | 0x00474ce0 | FUN_00474ce0 | crt_atexit_register_00474ce0 |  |
| 911 | 0x00474cf0 | FUN_00474cf0 | car_extended_ctor | C_car_ex::{ctor} |
| 912 | 0x00474d90 | FUN_00474d90 | car_extended_scalar_deleting_dtor |  |
| 913 | 0x00474db0 | FUN_00474db0 | car_extended_dtor | C_car_ex::{dtor} |
| 914 | 0x00474e50 | FUN_00474e50 | car_extended_init | C_car_ex::Init |
| 915 | 0x00474e70 | FUN_00474e70 | car_extended_game_init | C_car_ex::GameInit |
| 916 | 0x00474f60 | FUN_00474f60 | car_extended_scan_frame_tags | C_car_ex::S_hlp$2074C_car_ex_cpp::cbEnumProp |
| 917 | 0x004758e0 | FUN_004758e0 | car_extended_game_done | C_car_ex::GameDone |
| 918 | 0x00475bc0 | FUN_00475bc0 | car_extended_load_data_chunk | C_car_ex::LoadData |
| 919 | 0x00475be0 | FUN_00475be0 | car_extended_save_data_chunk | C_car_ex::SaveData |
| 920 | 0x00475c00 | thunk_FUN_00464f50 | thunk_car_copy_carinit_from_actor |  |
| 921 | 0x00475c10 | FUN_00475c10 | car_extended_add_carried_person | C_car_ex::DCAddPerson |
| 922 | 0x00475dd0 | FUN_00475dd0 | car_extended_remove_carried_person | C_car_ex::DCDelPerson |
| 923 | 0x00475e40 | FUN_00475e40 | car_extended_update | C_car_ex::Update |
| 924 | 0x00476bb0 | FUN_00476bb0 | car_extended_shake_part_vector_erase |  |
| 925 | 0x00476c00 | FUN_00476c00 | car_extended_shake_part_vector_insert_n | gif::vector<Q28C_car_ex6S_clap,8>::push_back |
| 926 | 0x00476f20 | FUN_00476f20 | car_extended_exhaust_part_vector_copy |  |
| 927 | 0x00476f50 | FUN_00476f50 | crt_static_init_guard_00476f50 |  |
| 928 | 0x00476f70 | FUN_00476f70 | crt_atexit_register_00476f70 |  |
| 929 | 0x00476f80 | FUN_00476f80 | cartridge_ctor |  |
| 930 | 0x00476fb0 | FUN_00476fb0 | cartridge_scalar_deleting_dtor |  |
| 931 | 0x00476fd0 | FUN_00476fd0 | cartridge_dtor |  |
| 932 | 0x00476fe0 | FUN_00476fe0 | cartridge_ai_tick |  |
| 933 | 0x004771f0 | thunk_FUN_00454840 | thunk_bottle_update |  |
| 934 | 0x00477200 | FUN_00477200 | cartridge_game_init |  |
| 935 | 0x00477260 | FUN_00477260 | cartridge_game_done |  |
| 936 | 0x004772e0 | FUN_004772e0 | cartridge_set_type |  |
| 937 | 0x00477300 | FUN_00477300 | crt_static_init_guard_00477300 |  |
| 938 | 0x00477320 | FUN_00477320 | crt_atexit_register_00477320 |  |
| 939 | 0x00477330 | FUN_00477330 | clock_ctor | C_clock::{ctor} |
| 940 | 0x00477350 | FUN_00477350 | clock_scalar_deleting_dtor | C_clock::{dtor} |
| 941 | 0x00477370 | FUN_00477370 | clock_dtor | C_clock::{dtor} |
| 942 | 0x00477380 | FUN_00477380 | clock_init | C_clock::Init |
| 943 | 0x004773f0 | thunk_FUN_00461180 | thunk_stub_noop_args4_00461180 |  |
| 944 | 0x00477400 | FUN_00477400 | clock_game_init | C_clock::GameInit |
| 945 | 0x00477540 | thunk_FUN_0044bd90 | thunk_actor_game_done | C_clock::GameDone |
| 946 | 0x00477550 | FUN_00477550 | clock_ai_tick | C_clock::AI |
| 947 | 0x004776c0 | FUN_004776c0 | crt_static_init_guard_004776c0 |  |
| 948 | 0x004776e0 | FUN_004776e0 | crt_atexit_register_004776e0 |  |
| 949 | 0x004776f0 | FUN_004776f0 | stub_noop_args4_004776f0 |  |
| 950 | 0x00477700 | FUN_00477700 | stub_ret0_cdecl_00477700 |  |
| 951 | 0x00477710 | FUN_00477710 | stub_rettrue_args4_00477710 |  |
| 952 | 0x00477720 | thunk_FUN_00477730 | thunk_crash_obj_disable_frame_colls |  |
| 953 | 0x00477730 | FUN_00477730 | crash_obj_disable_frame_colls |  |
| 954 | 0x004777d0 | FUN_004777d0 | crash_obj_create_crash_actors | CreateCrashActors |
| 955 | 0x00477d50 | FUN_00477d50 | crash_obj_create_actor | S_hlp$2136C_crash_obj_cpp::CreateActor |
| 956 | 0x00478290 | FUN_00478290 | crash_obj_create_from_model | CreateCrashModel |
| 957 | 0x00478580 | FUN_00478580 | crash_obj_parse_model_swap_tag | DoChange |
| 958 | 0x004786c0 | FUN_004786c0 | crash_obj_ctor | C_crash_obj::{ctor} |
| 959 | 0x00478730 | FUN_00478730 | crash_obj_scalar_deleting_dtor |  |
| 960 | 0x00478750 | FUN_00478750 | crash_obj_dtor | C_crash_obj::{dtor} |
| 961 | 0x00478760 | FUN_00478760 | crash_obj_ai | C_crash_obj::AI |
| 962 | 0x00478c10 | thunk_FUN_0044c120 | thunk_actor_save_game_get_size |  |
| 963 | 0x00478c20 | FUN_00478c20 | crash_obj_save_game_save | C_crash_obj::SaveGameSave |
| 964 | 0x00478c40 | FUN_00478c40 | crash_obj_save_game_load | C_crash_obj::SaveGameLoad |
| 965 | 0x00478c60 | FUN_00478c60 | crash_obj_init | C_crash_obj::Init |
| 966 | 0x00478f90 | FUN_00478f90 | crash_obj_game_init | C_crash_obj::GameInit |
| 967 | 0x00478fa0 | FUN_00478fa0 | crash_obj_game_done | C_crash_obj::GameDone |
| 968 | 0x00478ff0 | FUN_00478ff0 | crash_obj_on_collision | C_crash_obj::Collision |
| 969 | 0x00479120 | FUN_00479120 | crash_obj_hit | C_crash_obj::Hit |
| 970 | 0x00479200 | FUN_00479200 | crash_obj_set_model_file_name | C_crash_obj::SetModelFileName |
| 971 | 0x00479270 | FUN_00479270 | crash_obj_destroy_outermost | C_crash_obj::DestroyOuterMostCrashObject |
| 972 | 0x00479370 | FUN_00479370 | crt_static_init_guard_00479370 |  |
| 973 | 0x00479390 | FUN_00479390 | crt_atexit_register_00479390 |  |
| 974 | 0x004793a0 | FUN_004793a0 | crash_obj_get_material_id | C_crash_obj::GetMaterialID |
| 975 | 0x004793b0 | FUN_004793b0 | detector_ctor | C_detector::{ctor} |
| 976 | 0x00479440 | FUN_00479440 | detector_scalar_deleting_dtor |  |
| 977 | 0x00479460 | FUN_00479460 | detector_dtor | C_detector::{dtor} |
| 978 | 0x004794c0 | FUN_004794c0 | detector_init | C_detector::Init |
| 979 | 0x004794f0 | FUN_004794f0 | detector_duplicate_from | C_detector::DuplicateFrom |
| 980 | 0x00479530 | FUN_00479530 | detector_game_init | C_detector::GameInit |
| 981 | 0x004795b0 | FUN_004795b0 | detector_game_done | C_detector::GameDone |
| 982 | 0x004795f0 | FUN_004795f0 | detector_save_data | C_detector::SaveData |
| 983 | 0x00479650 | FUN_00479650 | detector_load_data | C_detector::LoadData |
| 984 | 0x00479740 | FUN_00479740 | detector_save_game_get_size | C_detector::SaveGameGetSize |
| 985 | 0x00479760 | FUN_00479760 | detector_save_game_save | C_detector::SaveGameSave |
| 986 | 0x004797d0 | FUN_004797d0 | detector_save_game_load | C_detector::SaveGameLoad |
| 987 | 0x004798a0 | FUN_004798a0 | detector_ai | C_detector::AI |
| 988 | 0x004798e0 | FUN_004798e0 | detector_update | C_detector::Update |
| 989 | 0x00479b50 | FUN_00479b50 | detector_hit | C_detector::Hit |
| 990 | 0x00479be0 | FUN_00479be0 | detector_create_colls | C_detector::CreateColls |
| 991 | 0x0047a150 | FUN_0047a150 | detector_count_dynb_frames | C_detector::S_hlp$2205C_detector_cpp::cbDummyCount |
| 992 | 0x0047a1a0 | FUN_0047a1a0 | detector_create_dynb_coll | C_detector::S_hlp$2205C_detector_cpp::cbDummyCreate |
| 993 | 0x0047a610 | FUN_0047a610 | detector_erase_colls | C_detector::EraseColls |
| 994 | 0x0047a6b0 | FUN_0047a6b0 | detector_create_using_object | C_detector::CreateUO |
| 995 | 0x0047a790 | FUN_0047a790 | detector_use | C_detector::UseDetector |
| 996 | 0x0047a810 | thunk_FUN_005afe70 | thunk_program_decode_params | C_detector::DecodeParams |
| 997 | 0x0047a820 | thunk_FUN_005b1b60 | thunk_program_decode_instruction | C_detector::DecodeInstruction |
| 998 | 0x0047a830 | thunk_FUN_0060b1a0 | thunk_stub_noop_args8_0060b1a0 | C_detector::DecompileCommand |
| 999 | 0x0047a840 | FUN_0047a840 | detector_program_process | C_detector::Process |
| 1000 | 0x0047aae0 | FUN_0047aae0 | crt_static_init_guard_0047aae0 |  |
| 1001 | 0x0047ab00 | FUN_0047ab00 | crt_atexit_register_0047ab00 |  |
| 1002 | 0x0047ab10 | FUN_0047ab10 | thunk_detector_scalar_deleting_dtor | @128@__dt__10C_detectorFv |
| 1003 | 0x0047ab20 | FUN_0047ab20 | thunk_detector_save_game_get_size | C_detector::@128@SaveGameGetSize |
| 1004 | 0x0047ab30 | FUN_0047ab30 | thunk_detector_save_game_save | C_detector::@128@SaveGameSave |
| 1005 | 0x0047ab40 | FUN_0047ab40 | thunk_detector_save_game_load | C_detector::@128@SaveGameLoad |
| 1006 | 0x0047ab50 | FUN_0047ab50 | detector_get_program | C_detector::GetProgram |
| 1007 | 0x0047ab60 | FUN_0047ab60 | dog_ctor | C_dog::{ctor} |
| 1008 | 0x0047ad60 | FUN_0047ad60 | dog_scalar_deleting_dtor |  |
| 1009 | 0x0047ad80 | FUN_0047ad80 | dog_dtor | C_dog::{dtor} |
| 1010 | 0x0047ae00 | FUN_0047ae00 | dog_save_data | C_dog::SaveData |
| 1011 | 0x0047af40 | FUN_0047af40 | dog_load_data | C_dog::LoadData |
| 1012 | 0x0047b160 | FUN_0047b160 | dog_duplicate_from | C_dog::DuplicateFrom |
| 1013 | 0x0047b1c0 | FUN_0047b1c0 | dog_game_init | C_dog::GameInit |
| 1014 | 0x0047b680 | FUN_0047b680 | dog_load_tracks | C_dog::LoadTCK |
| 1015 | 0x0047b6a0 | FUN_0047b6a0 | dog_game_done | C_dog::GameDone |
| 1016 | 0x0047b800 | FUN_0047b800 | dog_spawn_severed_head | C_dog::LoseHead |
| 1017 | 0x0047bdd0 | FUN_0047bdd0 | dog_hit | C_dog::Hit |
| 1018 | 0x0047c190 | FUN_0047c190 | dog_death | C_dog::Death |
| 1019 | 0x0047c210 | FUN_0047c210 | dog_ai | C_dog::AI |
| 1020 | 0x0047c990 | FUN_0047c990 | dog_follow_path | C_dog::SolvePath |
| 1021 | 0x0047d0e0 | FUN_0047d0e0 | dog_move_toward_target | C_dog::AI_walk_run |
| 1022 | 0x0047d450 | FUN_0047d450 | dog_turn_toward_target | C_dog::AI_turn |
| 1023 | 0x0047d7f0 | FUN_0047d7f0 | dog_enter_state | C_dog::ChooseAction |
| 1024 | 0x0047dba0 | FUN_0047dba0 | dog_update | C_dog::Update |
| 1025 | 0x0047e0e0 | FUN_0047e0e0 | dog_apply_pending_anim | C_dog::SetAnim |
| 1026 | 0x0047e1e0 | FUN_0047e1e0 | dog_test_move_collision | C_dog::TestCollision |
| 1027 | 0x0047e480 | FUN_0047e480 | dog_choose_next_web_node | C_dog::ChooseNextCheckPoint |
| 1028 | 0x0047e740 | FUN_0047e740 | dog_is_segment_blocked | C_dog::IsBlocked |
| 1029 | 0x0047e7f0 | FUN_0047e7f0 | dog_snap_to_ground | C_dog::ground |
| 1030 | 0x0047e900 | FUN_0047e900 | dog_frame_notify_callback | dog_notify_callback |
| 1031 | 0x0047eaf0 | FUN_0047eaf0 | dog_spawn_blood_drop | C_dog::BloodStep |
| 1032 | 0x0047eda0 | FUN_0047eda0 | dog_create_all_sounds | C_dog::InitSounds |
| 1033 | 0x0047ee10 | FUN_0047ee10 | dog_create_sound_slot | C_dog::InitSound |
| 1034 | 0x0047eeb0 | FUN_0047eeb0 | dog_release_all_sounds | C_dog::ReleaseSounds |
| 1035 | 0x0047eee0 | FUN_0047eee0 | dog_get_sound | C_dog::GetSound |
| 1036 | 0x0047ef20 | FUN_0047ef20 | dog_stop_sound | C_dog::StopSound |
| 1037 | 0x0047ef40 | FUN_0047ef40 | dog_return_to_patrol | C_dog::ReturnFromAttackAI |
| 1038 | 0x0047efb0 | FUN_0047efb0 | dog_acquire_player_target | C_dog::TestEnemy |
| 1039 | 0x0047f290 | FUN_0047f290 | dog_replan_path | C_dog::UnblockPath |
| 1040 | 0x0047f3a0 | FUN_0047f3a0 | dog_set_destination_and_plan | C_dog::UpdatePath |
| 1041 | 0x0047f490 | FUN_0047f490 | dog_save_game_get_size | C_dog::SaveGameGetSize |
| 1042 | 0x0047f4a0 | FUN_0047f4a0 | dog_save_game_save | C_dog::SaveGameSave |
| 1043 | 0x0047f680 | FUN_0047f680 | dog_save_game_load | C_dog::SaveGameLoad |
| 1044 | 0x0047f950 | FUN_0047f950 | dog_on_collision | C_dog::Collision |
| 1045 | 0x0047f9e0 | FUN_0047f9e0 | dog_arm_bleeding | C_dog::cbCollTester |
| 1046 | 0x0047fa10 | FUN_0047fa10 | i3d_cache_anim_create | C_I3D_cache<17I3D_animation_set>::Create |
| 1047 | 0x0047fe40 | FUN_0047fe40 | i3d_open_notify_forward_cb |  |
| 1048 | 0x0047fe60 | FUN_0047fe60 | std_vector8_insert_one |  |
| 1049 | 0x0047ffe0 | FUN_0047ffe0 | array_copy_refcounted8 |  |
| 1050 | 0x00480040 | FUN_00480040 | array_copy_backward_refcounted8 |  |
| 1051 | 0x004800a0 | FUN_004800a0 | refcounted8_scalar_deleting_dtor |  |
| 1052 | 0x004800c0 | FUN_004800c0 | crt_static_init_guard_004800c0 |  |
| 1053 | 0x004800e0 | FUN_004800e0 | crt_atexit_register_004800e0 |  |
| 1054 | 0x004800f0 | FUN_004800f0 | door_ctor | C_door::{ctor} |
| 1055 | 0x004801d0 | FUN_004801d0 | door_scalar_deleting_dtor |  |
| 1056 | 0x004801f0 | FUN_004801f0 | door_dtor | C_door::{dtor} |
| 1057 | 0x00480200 | FUN_00480200 | door_duplicate_from | C_door::DuplicateFrom |
| 1058 | 0x004802c0 | FUN_004802c0 | door_hit | C_door::Hit |
| 1059 | 0x00480450 | FUN_00480450 | door_set_state | C_door::SetState |
| 1060 | 0x00480880 | FUN_00480880 | door_ai | C_door::AI |
| 1061 | 0x00480d50 | FUN_00480d50 | door_net_apply_remote_state | C_door::NetDirect |
| 1062 | 0x00480d70 | FUN_00480d70 | door_game_init | C_door::GameInit |
| 1063 | 0x00481c20 | FUN_00481c20 | door_game_done | C_door::GameDone |
| 1064 | 0x00481c90 | FUN_00481c90 | door_enable_using_object | C_door::EnableUsingObject |
| 1065 | 0x00481d10 | FUN_00481d10 | door_save_data | C_door::SaveData |
| 1066 | 0x00481ef0 | FUN_00481ef0 | door_load_data | C_door::LoadData |
| 1067 | 0x004823b0 | FUN_004823b0 | door_save_game_get_size | C_door::SaveGameGetSize |
| 1068 | 0x004823c0 | FUN_004823c0 | door_save_game_save | C_door::SaveGameSave |
| 1069 | 0x00482420 | FUN_00482420 | door_save_game_load | C_door::SaveGameLoad |
| 1070 | 0x00482690 | FUN_00482690 | stub_retfalse_args24_00482690 |  |
| 1071 | 0x004826a0 | FUN_004826a0 | door_set_open_angle | C_door::SetOpenAngle |
| 1072 | 0x00482940 | FUN_00482940 | door_get_bound_points_2d | C_door::GetBoundPoints2D |
| 1073 | 0x00482b50 | FUN_00482b50 | door_apply_net_event_msg |  |
| 1074 | 0x00482b80 | FUN_00482b80 | door_on_act_state_change | C_door::ChangeState |
| 1075 | 0x00482ba0 | FUN_00482ba0 | i3d_copy_bound_volume |  |
| 1076 | 0x00482bd0 | FUN_00482bd0 | crt_static_init_guard_00482bd0 |  |
| 1077 | 0x00482bf0 | FUN_00482bf0 | crt_atexit_register_00482bf0 |  |
| 1078 | 0x00482c00 | FUN_00482c00 | driver_ctor | C_driver::{ctor} |
| 1079 | 0x00482d50 | FUN_00482d50 | driver_dtor | C_driver::{dtor} |
| 1080 | 0x00482de0 | FUN_00482de0 | driver_compute_radius | C_driver::ComputeRadius |
| 1081 | 0x00482ff0 | FUN_00482ff0 | driver_reset | C_driver::Reset |
| 1082 | 0x004831c0 | FUN_004831c0 | driver_set_car | C_driver::SetCar |
| 1083 | 0x00483360 | FUN_00483360 | driver_tick | C_driver::Tick |
| 1084 | 0x00485ec0 | FUN_00485ec0 | driver_fill_start_values | C_driver::FillStartValues |
| 1085 | 0x004861a0 | FUN_004861a0 | driver_optimize_next_checkpoints | C_driver::OptimizeNextCheckpoints |
| 1086 | 0x004863f0 | FUN_004863f0 | driver_advance_checkpoint | C_driver::NevimJakToNazvat |
| 1087 | 0x00486760 | FUN_00486760 | driver_compute_brake_onset | C_driver::TakNaCemToUbrzdim |
| 1088 | 0x00486830 | FUN_00486830 | driver_last_route_in_brake_path_tol | C_driver::LastRouteInBrakePathPlusToleranciaLaskaASvoboda |
| 1089 | 0x004869e0 | FUN_004869e0 | driver_ramp_brake_distance | C_driver::MinSOld |
| 1090 | 0x00486a60 | FUN_00486a60 | driver_last_route_in_brake_path | C_driver::LastRouteInBrakePath |
| 1091 | 0x00486b80 | FUN_00486b80 | driver_compute_corner_speed_limit | C_driver::MinV |
| 1092 | 0x00486d70 | FUN_00486d70 | driver_path_correction | C_driver::PathCorrection |
| 1093 | 0x004873d0 | FUN_004873d0 | driver_lane_change_and_park | C_driver::PrejezdZPruhuDoPruhuAParkovani |
| 1094 | 0x00487a70 | FUN_00487a70 | driver_predictive_sphere_lane_cross | C_driver::PredvidaciKoulePripPrejezduMeziPruhy |
| 1095 | 0x00487f40 | FUN_00487f40 | driver_predict_collision | C_driver::PredvidaciKolize |
| 1096 | 0x00488e20 | FUN_00488e20 | driver_reduce_speed_in_corner | C_driver::SnizRychlostVzatacce |
| 1097 | 0x004891e0 | FUN_004891e0 | driver_compute_brake_pedal | C_driver::UtekloPraseZase |
| 1098 | 0x004893e0 | FUN_004893e0 | driver_turn | C_driver::Zatoc |
| 1099 | 0x00489950 | FUN_00489950 | driver_do_lights | C_driver::DoLights |
| 1100 | 0x00489c00 | FUN_00489c00 | driver_do_sounds | C_driver::DoSounds |
| 1101 | 0x00489cc0 | FUN_00489cc0 | std_vector70_size |  |
| 1102 | 0x00489cf0 | FUN_00489cf0 | std_vector70_insert_n |  |
| 1103 | 0x0048a010 | FUN_0048a010 | std_vector70_uninit_copy |  |
| 1104 | 0x0048a050 | FUN_0048a050 | std_vector70_uninit_fill_n |  |
| 1105 | 0x0048a080 | FUN_0048a080 | std_vector70_construct |  |
| 1106 | 0x0048a0a0 | FUN_0048a0a0 | crt_static_init_guard_0048a0a0 |  |
| 1107 | 0x0048a0c0 | FUN_0048a0c0 | crt_atexit_register_0048a0c0 |  |
| 1108 | 0x0048a0d0 | FUN_0048a0d0 | drop_in_weapon_ctor | C_drop_in_weapon::{ctor} |
| 1109 | 0x0048a140 | FUN_0048a140 | drop_in_weapon_scalar_deleting_dtor |  |
| 1110 | 0x0048a160 | FUN_0048a160 | drop_in_weapon_dtor | C_drop_in_weapon::{dtor} |
| 1111 | 0x0048a170 | FUN_0048a170 | drop_in_weapon_update | C_drop_in_weapon::Update |
| 1112 | 0x0048a1d0 | FUN_0048a1d0 | drop_in_weapon_on_act_state_changed | C_drop_in_weapon::ChangeState |
| 1113 | 0x0048a360 | FUN_0048a360 | drop_in_weapon_game_init | C_drop_in_weapon::GameInit |
| 1114 | 0x0048a440 | FUN_0048a440 | drop_in_weapon_game_done | C_drop_in_weapon::GameDone |
| 1115 | 0x0048a4f0 | FUN_0048a4f0 | stub_retfalse_args32_0048a4f0 |  |
| 1116 | 0x0048a500 | FUN_0048a500 | crt_static_init_guard_0048a500 |  |
| 1117 | 0x0048a520 | FUN_0048a520 | crt_atexit_register_0048a520 |  |
| 1118 | 0x0048a530 | FUN_0048a530 | dropout_ctor | C_DropOut::{ctor} |
| 1119 | 0x0048a5d0 | FUN_0048a5d0 | dropout_scalar_deleting_dtor | C_DropOut::{dtor} |
| 1120 | 0x0048a5f0 | FUN_0048a5f0 | dropout_dtor | C_DropOut::{dtor} |
| 1121 | 0x0048a630 | FUN_0048a630 | dropout_init | C_DropOut::Init |
| 1122 | 0x0048a810 | FUN_0048a810 | dropout_game_init | C_DropOut::GameInit |
| 1123 | 0x0048a830 | FUN_0048a830 | dropout_game_done | C_DropOut::GameDone |
| 1124 | 0x0048a880 | FUN_0048a880 | dropout_ai_tick | C_DropOut::AI |
| 1125 | 0x0048a8a0 | FUN_0048a8a0 | dropout_update | C_DropOut::Update |
| 1126 | 0x0048a8c0 | FUN_0048a8c0 | dropout_on_act_state_changed | C_DropOut::ChangeState |
| 1127 | 0x0048aa30 | thunk_FUN_004c4000 | thunk_physical_object_init_wheel | C_DropOut::InitWheel |
| 1128 | 0x0048aa40 | FUN_0048aa40 | dropout_init_box | C_DropOut::InitBox |
| 1129 | 0x0048aa60 | FUN_0048aa60 | dropout_on_collision | C_DropOut::Collision |
| 1130 | 0x0048aaa0 | FUN_0048aaa0 | dropout_hit | C_DropOut::Hit |
| 1131 | 0x0048aaf0 | FUN_0048aaf0 | track_apply_rigid_pose_at_clock |  |
| 1132 | 0x0048ada0 | FUN_0048ada0 | dropout_track_create |  |
| 1133 | 0x0048adc0 | FUN_0048adc0 | dropout_track_record_key |  |
| 1134 | 0x0048af00 | FUN_0048af00 | crt_static_init_guard_0048af00 |  |
| 1135 | 0x0048af20 | FUN_0048af20 | crt_atexit_register_0048af20 |  |
| 1136 | 0x0048af30 | FUN_0048af30 | grenade_ctor | C_grenade::{ctor} |
| 1137 | 0x0048af60 | FUN_0048af60 | grenade_scalar_deleting_dtor |  |
| 1138 | 0x0048af80 | FUN_0048af80 | grenade_dtor | C_grenade::{dtor} |
| 1139 | 0x0048af90 | FUN_0048af90 | bottle_subclass_init_require_frame |  |
| 1140 | 0x0048afb0 | FUN_0048afb0 | grenade_ai_tick | C_grenade::AI |
| 1141 | 0x0048b310 | FUN_0048b310 | grenade_game_init | C_grenade::GameInit |
| 1142 | 0x0048b340 | FUN_0048b340 | grenade_game_done | C_grenade::GameDone |
| 1143 | 0x0048b360 | FUN_0048b360 | grenade_save_game_get_size | C_grenade::SaveGameGetSize |
| 1144 | 0x0048b370 | FUN_0048b370 | grenade_save_game_save | C_grenade::SaveGameSave |
| 1145 | 0x0048b400 | FUN_0048b400 | grenade_save_game_load | C_grenade::SaveGameLoad |
| 1146 | 0x0048b490 | FUN_0048b490 | grenade_on_collision | C_grenade::Collision |
| 1147 | 0x0048b4c0 | FUN_0048b4c0 | grenade_hit | C_grenade::Hit |
| 1148 | 0x0048b510 | FUN_0048b510 | grenade_apply_net_event_msg |  |
| 1149 | 0x0048b590 | FUN_0048b590 | grenade_net_apply_remote_state | C_grenade::NetDirect |
| 1150 | 0x0048b5b0 | FUN_0048b5b0 | crt_static_init_guard_0048b5b0 |  |
| 1151 | 0x0048b5d0 | FUN_0048b5d0 | crt_atexit_register_0048b5d0 |  |
| 1152 | 0x0048b5e0 | FUN_0048b5e0 | human_track_create |  |
| 1153 | 0x0048b600 | FUN_0048b600 | human_track_record_key |  |
| 1154 | 0x0048bad0 | FUN_0048bad0 | human_track_apply_key_anim |  |
| 1155 | 0x0048bd30 | FUN_0048bd30 | human_track_apply_pose_at_clock |  |
| 1156 | 0x0048c320 | FUN_0048c320 | human_track_apply_pose_at_time |  |
| 1157 | 0x0048cb40 | FUN_0048cb40 | human_track_dump_keys |  |
| 1158 | 0x0048cc20 | FUN_0048cc20 | human_track_get_pos_at_time |  |
| 1159 | 0x0048d0d0 | FUN_0048d0d0 | human_track_get_rot_at_time |  |
| 1160 | 0x0048d590 | FUN_0048d590 | human_notify_callback | human_notify_callback |
| 1161 | 0x0048e7d0 | FUN_0048e7d0 | ai_loc_init | ai_loc::Init |
| 1162 | 0x0048e810 | FUN_0048e810 | ai_loc_update | ai_loc::Update |
| 1163 | 0x0048eb50 | FUN_0048eb50 | ai_loc_add_or_refresh_seg_stat | ai_loc::AddSegment |
| 1164 | 0x0048edb0 | FUN_0048edb0 | ai_loc_prune_stale_seg_stats | ai_loc::SegListUpdate |
| 1165 | 0x0048ee60 | FUN_0048ee60 | ai_blob_log_add_blob | ai_blob_log::Blob |
| 1166 | 0x0048f2c0 | FUN_0048f2c0 | ai_blob_log_get_blob | ai_blob_log::GetBlob |
| 1167 | 0x0048f310 | FUN_0048f310 | ai_blob_log_clear | ai_blob_log::Clear |
| 1168 | 0x0048f330 | FUN_0048f330 | human_ground_drop_below |  |
| 1169 | 0x0048f3b0 | FUN_0048f3b0 | human_ground_drop_below_static |  |
| 1170 | 0x0048f430 | FUN_0048f430 | human_ctor | C_human::{ctor} |
| 1171 | 0x0048f650 | FUN_0048f650 | human_scalar_deleting_dtor |  |
| 1172 | 0x0048f670 | FUN_0048f670 | human_dtor | C_human::{dtor} |
| 1173 | 0x0048f790 | FUN_0048f790 | inventory_dtor |  |
| 1174 | 0x0048f7b0 | FUN_0048f7b0 | vector_dispose_at_0x24 |  |
| 1175 | 0x0048f7d0 | FUN_0048f7d0 | human_init | C_human::Init |
| 1176 | 0x0048f880 | FUN_0048f880 | human_duplicate_from | C_human::DuplicateFrom |
| 1177 | 0x0048f8b0 | FUN_0048f8b0 | human_save_data | C_human::SaveData |
| 1178 | 0x0048f950 | FUN_0048f950 | human_load_data | C_human::LoadData |
| 1179 | 0x0048fbd0 | FUN_0048fbd0 | human_load_stats_block_scaled |  |
| 1180 | 0x0048fd30 | FUN_0048fd30 | human_load_stats_block_plain |  |
| 1181 | 0x0048fe50 | FUN_0048fe50 | human_save_game_get_size | C_human::SaveGameGetSize |
| 1182 | 0x0048fe90 | FUN_0048fe90 | human_save_game_save | C_human::SaveGameSave |
| 1183 | 0x004901e0 | FUN_004901e0 | human_save_game_load | C_human::SaveGameLoad |
| 1184 | 0x00490a20 | FUN_00490a20 | human_death | C_human::Death |
| 1185 | 0x00490b10 | FUN_00490b10 | human_extern_ai_update | C_human::Extern_AI_Update |
| 1186 | 0x00490b50 | FUN_00490b50 | human_sounds_change | C_human::SoundsChange |
| 1187 | 0x00490bf0 | FUN_00490bf0 | human_ai | C_human::AI |
| 1188 | 0x00491b50 | FUN_00491b50 | human_intern_use_car | C_human::Intern_UseCar |
| 1189 | 0x00492000 | FUN_00492000 | human_intern_use_railway | C_human::Intern_UseRailway |
| 1190 | 0x00492620 | FUN_00492620 | human_intern_use_car_truck | C_human::_intern_UseCarTruck |
| 1191 | 0x004929a0 | FUN_004929a0 | human_throw_from_car_truck | C_human::ThrowFromCarTruck |
| 1192 | 0x00492ff0 | FUN_00492ff0 | human_update | C_human::Update |
| 1193 | 0x00494150 | FUN_00494150 | human_change_dead_anim | C_human::ChangeDeadAnim |
| 1194 | 0x004942c0 | FUN_004942c0 | human_animation_evaluate_and_dispatch | C_human::PersonAnim |
| 1195 | 0x00494c40 | FUN_00494c40 | human_shift_animation | C_human::ShiftAnimation |
| 1196 | 0x00494d60 | FUN_00494d60 | human_change_state | C_human::ChangeState |
| 1197 | 0x00495040 | FUN_00495040 | human_game_init | C_human::GameInit |
| 1198 | 0x00495c30 | FUN_00495c30 | human_game_done | C_human::GameDone |
| 1199 | 0x00496110 | FUN_00496110 | human_create_dyn_colls | C_human::CreateDynColls |
| 1200 | 0x00496340 | FUN_00496340 | human_erase_dyn_colls | C_human::EraseDynColls |
| 1201 | 0x004963d0 | FUN_004963d0 | human_deactivate_colls | C_human::DeactivateColls |
| 1202 | 0x00496430 | FUN_00496430 | human_activate_colls | C_human::ActivateColls |
| 1203 | 0x004964b0 | FUN_004964b0 | human_update_dyn_coll | C_human::UpdateDynColl |
| 1204 | 0x00496710 | FUN_00496710 | human_hit | C_human::Hit |
| 1205 | 0x00498740 | FUN_00498740 | clear4_dwords_at |  |
| 1206 | 0x00498760 | FUN_00498760 | human_hit_in_car | C_human::HitInCar |
| 1207 | 0x00498970 | FUN_00498970 | inventory_reset |  |
| 1208 | 0x00498a00 | FUN_00498a00 | human_collision | C_human::Collision |
| 1209 | 0x004991c0 | FUN_004991c0 | human_is_possible_coll | C_human::IsPossibleColl |
| 1210 | 0x00499380 | FUN_00499380 | human_track_queue_anim_event |  |
| 1211 | 0x00499440 | FUN_00499440 | human_pose_interpolate | C_human::PoseInterpolate |
| 1212 | 0x00499b90 | FUN_00499b90 | human_pose_set_pose_normal | C_human::PoseSetPoseNormal |
| 1213 | 0x0049a400 | FUN_0049a400 | human_pose_set_pose_aimed | C_human::PoseSetPoseAimed |
| 1214 | 0x0049aac0 | FUN_0049aac0 | human_pose_null_rotate | C_human::PoseNullRotate |
| 1215 | 0x0049ab50 | FUN_0049ab50 | human_pose_validate | C_human::PoseValidate |
| 1216 | 0x0049ab90 | FUN_0049ab90 | human_get_anim_id_for_death | C_human::GetAnimIDForDeath |
| 1217 | 0x0049ac70 | FUN_0049ac70 | human_movement | C_human::Movement |
| 1218 | 0x0049bbc0 | FUN_0049bbc0 | human_movement_intern | C_human::MovementIntern |
| 1219 | 0x0049d0c0 | FUN_0049d0c0 | human_test_move_coll | C_human::TestMoveColl |
| 1220 | 0x0049d540 | FUN_0049d540 | human_set_next_coll_owners | C_human::DynSetNextCollOwnerMult |
| 1221 | 0x0049d640 | FUN_0049d640 | human_test_move_coll_in_owner | C_human::TestMoveCollInOwner |
| 1222 | 0x0049d840 | FUN_0049d840 | human_do_stop_anim | C_human::Do_StopAnim |
| 1223 | 0x0049d870 | FUN_0049d870 | human_do_breath | C_human::Do_Breath |
| 1224 | 0x0049d8c0 | FUN_0049d8c0 | human_set_movement_intent_forward | C_human::Go_Front |
| 1225 | 0x0049d990 | FUN_0049d990 | human_set_movement_intent_backward | C_human::Go_Back |
| 1226 | 0x0049da30 | FUN_0049da30 | human_set_movement_intent_left | C_human::Go_Left |
| 1227 | 0x0049db00 | FUN_0049db00 | human_set_movement_intent_right | C_human::Go_Right |
| 1228 | 0x0049dbd0 | FUN_0049dbd0 | human_set_movement_intent_forward_left | C_human::Go_FrontLeft |
| 1229 | 0x0049ddc0 | FUN_0049ddc0 | human_set_movement_intent_forward_right | C_human::Go_FrontRight |
| 1230 | 0x0049dfb0 | FUN_0049dfb0 | human_set_movement_intent_back_left | C_human::Go_BackLeft |
| 1231 | 0x0049e170 | FUN_0049e170 | human_set_movement_intent_back_right | C_human::Go_BackRight |
| 1232 | 0x0049e260 | FUN_0049e260 | human_change_anim | C_human::ChangeAnim |
| 1233 | 0x0049e520 | FUN_0049e520 | human_get_anim_id_for_use_car | C_human::GetAnimIDForUseCar |
| 1234 | 0x0049e580 | FUN_0049e580 | human_put_in_vehicle_at_seat | C_human::Intern_UseCar |
| 1235 | 0x0049ea60 | FUN_0049ea60 | human_intern_from_car | C_human::Intern_FromCar |
| 1236 | 0x0049eee0 | FUN_0049eee0 | human_repair_weapon_position | C_human::RepairWeaponPosition |
| 1237 | 0x0049f180 | FUN_0049f180 | human_change_weapon_model | C_human::ChangeWeaponModel |
| 1238 | 0x0049faa0 | FUN_0049faa0 | human_do_change_weapon | C_human::Do_Holster |
| 1239 | 0x0049fd80 | FUN_0049fd80 | human_do_aimed | C_human::Do_Aimed |
| 1240 | 0x0049fdf0 | FUN_0049fdf0 | human_do_crouched | C_human::Do_Crouched |
| 1241 | 0x0049fef0 | FUN_0049fef0 | human_inventory_truncate | C_human::InventoryTruncate |
| 1242 | 0x0049fff0 | FUN_0049fff0 | human_drop_out_items | C_human::DropOutItems |
| 1243 | 0x004a0740 | FUN_004a0740 | human_find_blocking_drawbridge | C_human::TozNaselJsemTenNejvyssiMost |
| 1244 | 0x004a0840 | FUN_004a0840 | human_can_exit_car_free_space | C_human::Can_DropOutFromCar_Free |
| 1245 | 0x004a0b10 | FUN_004a0b10 | human_can_exit_car_at_seat | C_human::Can_DropOutFromCar |
| 1246 | 0x004a0e10 | FUN_004a0e10 | human_can_enter_car | C_human::Can_EnterToCar |
| 1247 | 0x004a11f0 | FUN_004a11f0 | human_climb_out_of_car | C_human::ZasrateLezeniZAuta_ve2rano |
| 1248 | 0x004a1770 | FUN_004a1770 | human_eject_out_of_car | C_human::ZasrateVybliknutiZAuta_ve3rano |
| 1249 | 0x004a2400 | FUN_004a2400 | human_do_climb_in_car_lr | C_human::Do_ClimbInCarLR |
| 1250 | 0x004a26d0 | FUN_004a26d0 | human_use_actor | C_human::Use_Actor |
| 1251 | 0x004a3ae0 | FUN_004a3ae0 | human_dispatch_weapon_action_at_target | C_human::Do_Shoot |
| 1252 | 0x004a4490 | FUN_004a4490 | human_do_throw_grenade | C_human::Do_ThrowGrenade |
| 1253 | 0x004a4560 | FUN_004a4560 | human_can_special_stroke | C_human::Can_SpecialStroke |
| 1254 | 0x004a49d0 | FUN_004a49d0 | human_do_castrate | C_human::Do_Castrate |
| 1255 | 0x004a4aa0 | FUN_004a4aa0 | human_do_hard_head | C_human::Do_HardHead |
| 1256 | 0x004a4b70 | FUN_004a4b70 | human_shooting | C_human::Shooting |
| 1257 | 0x004a6070 | FUN_004a6070 | force_feedback_manager_is_enabled |  |
| 1258 | 0x004a6090 | FUN_004a6090 | inventory_get_in_hand_item |  |
| 1259 | 0x004a60b0 | FUN_004a60b0 | human_do_reload | C_human::Do_Reload |
| 1260 | 0x004a61d0 | FUN_004a61d0 | human_weapon_toggle_held | C_human::Do_Holster |
| 1261 | 0x004a6300 | FUN_004a6300 | human_weapon_drop | C_human::Do_Holster |
| 1262 | 0x004a63f0 | FUN_004a63f0 | human_do_play_anim | C_human::Do_PlayAnim |
| 1263 | 0x004a64b0 | FUN_004a64b0 | human_get_bound_points_2d | C_human::GetBoundPoints2D |
| 1264 | 0x004a6700 | FUN_004a6700 | human_do_jump | C_human::Do_Jump |
| 1265 | 0x004a6970 | FUN_004a6970 | human_do_climb | C_human::Do_Climb |
| 1266 | 0x004a75b0 | FUN_004a75b0 | human_do_side_jump | C_human::Do_SideJump |
| 1267 | 0x004a7700 | FUN_004a7700 | human_change_model | C_human::_intern_ChangeModel |
| 1268 | 0x004a7d10 | FUN_004a7d10 | human_create_force_feedback_effects |  |
| 1269 | 0x004a7df0 | FUN_004a7df0 | human_release_force_feedback_effects |  |
| 1270 | 0x004a7e40 | FUN_004a7e40 | human_force_death | C_human::_intern_ForceDeath |
| 1271 | 0x004a8130 | FUN_004a8130 | human_do_shock_start | C_human::Do_ShockStart |
| 1272 | 0x004a8190 | FUN_004a8190 | human_do_shock_down | C_human::Do_ShockDown |
| 1273 | 0x004a8220 | FUN_004a8220 | human_do_shock_up | C_human::Do_ShockUp |
| 1274 | 0x004a8260 | FUN_004a8260 | human_do_shock_stumble | C_human::Do_ShockStumble |
| 1275 | 0x004a82a0 | FUN_004a82a0 | human_cb_coll_tester | C_human::cbCollTester |
| 1276 | 0x004a82e0 | FUN_004a82e0 | human_do_throw_occupant_from_car | C_human::Do_ThrowCocotFromCar |
| 1277 | 0x004a90b0 | FUN_004a90b0 | human_intern_throw_self_from_car | C_human::_intern_ThrowMeFromCar |
| 1278 | 0x004a96b0 | FUN_004a96b0 | human_do_box_pick_up | C_human::Do_FuckingBox |
| 1279 | 0x004a9bf0 | FUN_004a9bf0 | human_do_box_cancel | C_human::Do_FuckingBoxCancel |
| 1280 | 0x004aa190 | FUN_004aa190 | human_do_dead_body_pick_up | C_human::Do_DeadBody |
| 1281 | 0x004aab10 | FUN_004aab10 | human_car_launched_into_air | C_human::EineMeineKleineAutoInLuft |
| 1282 | 0x004aad00 | FUN_004aad00 | human_can_aim | C_human::Can_Aimed |
| 1283 | 0x004aaed0 | FUN_004aaed0 | human_do_corner_breath | C_human::Do_CornerBreath |
| 1284 | 0x004aaf10 | FUN_004aaf10 | human_do_corner_view | C_human::Do_CornerView |
| 1285 | 0x004aafd0 | FUN_004aafd0 | human_do_corner_fire | C_human::Do_CornerFire |
| 1286 | 0x004ab0f0 | FUN_004ab0f0 | human_do_corner_fire_aimed | C_human::Do_CornerFireAimed |
| 1287 | 0x004ab1c0 | FUN_004ab1c0 | human_force_ready_to_act | C_human::Force_CanWorking |
| 1288 | 0x004ab1d0 | FUN_004ab1d0 | human_turn_in_place | C_human::Turn_Turn |
| 1289 | 0x004ab200 | FUN_004ab200 | human_activate | C_human::Activate |
| 1290 | 0x004ab8a0 | FUN_004ab8a0 | human_deactivate | C_human::Deactivate |
| 1291 | 0x004ab9e0 | FUN_004ab9e0 | human_new_scream | C_human::NewScream |
| 1292 | 0x004abc00 | FUN_004abc00 | human_do_box_drop | C_human::Do_FuckingBoxDrop |
| 1293 | 0x004abd50 | FUN_004abd50 | human_do_dead_body_drop | C_human::Do_DeadBodyDrop |
| 1294 | 0x004abde0 | FUN_004abde0 | human_reset_actions | C_human::ResetActions |
| 1295 | 0x004abe40 | FUN_004abe40 | human_recompile_death_pos | C_human::RecompileDeathPos |
| 1296 | 0x004ac430 | FUN_004ac430 | human_death_pose_align_to_ground |  |
| 1297 | 0x004ac9f0 | FUN_004ac9f0 | human_death_pose_push_out_of_wall |  |
| 1298 | 0x004ace80 | FUN_004ace80 | human_death_pose_settle_to_ground |  |
| 1299 | 0x004ad3c0 | FUN_004ad3c0 | human_process_death_pos | C_human::ProcessDeathPos |
| 1300 | 0x004ad4b0 | FUN_004ad4b0 | human_is_this_car_stealing | C_human::IsThisCarStealing |
| 1301 | 0x004ad520 | FUN_004ad520 | human_free_hand | C_human::FreeHand |
| 1302 | 0x004ad770 | FUN_004ad770 | human_weapon_effect_no_magazine | C_human::BezZasobnikuWeaponEffect |
| 1303 | 0x004ad7c0 | FUN_004ad7c0 | human_start_weapon_effect | C_human::StartWeaponEffect |
| 1304 | 0x004adf70 | FUN_004adf70 | human_tick_weapon_effect | C_human::TickWeaponEffect |
| 1305 | 0x004ae460 | FUN_004ae460 | human_reset_weapon_effects | C_human::ResetWeaponEffects |
| 1306 | 0x004ae710 | FUN_004ae710 | human_fake_weapon_effect | C_human::FakeWeaponEffect |
| 1307 | 0x004ae770 | FUN_004ae770 | human_is_hit_allowed_between_types | C_human::Can_Hit_FilterHumanType |
| 1308 | 0x004ae7f0 | FUN_004ae7f0 | clear1_dword_at_0x04 |  |
| 1309 | 0x004ae800 | FUN_004ae800 | crt_static_init_guard_004ae800 |  |
| 1310 | 0x004ae820 | FUN_004ae820 | crt_atexit_register_004ae820 |  |
| 1311 | 0x004ae830 | FUN_004ae830 | spike_strip_ctor | C_irenka::{ctor} |
| 1312 | 0x004ae860 | FUN_004ae860 | spike_strip_deleting_dtor | C_irenka::{dtor} |
| 1313 | 0x004ae880 | FUN_004ae880 | spike_strip_dtor | C_irenka::{dtor} |
| 1314 | 0x004ae890 | FUN_004ae890 | actor_subclass_init_require_frame |  |
| 1315 | 0x004ae8b0 | FUN_004ae8b0 | spike_strip_ai_tick | C_irenka::AI |
| 1316 | 0x004ae8c0 | FUN_004ae8c0 | spike_strip_game_init | C_irenka::GameInit |
| 1317 | 0x004aeb70 | FUN_004aeb70 | spike_strip_game_done | C_irenka::GameDone |
| 1318 | 0x004aebb0 | FUN_004aebb0 | spike_strip_deactivate | C_irenka::Deactivate |
| 1319 | 0x004aebe0 | FUN_004aebe0 | spike_strip_activate |  |
| 1320 | 0x004aeda0 | FUN_004aeda0 | spike_strip_test_segment_crossing | C_irenka::ZapichameSi |
| 1321 | 0x004af000 | FUN_004af000 | crt_static_init_guard_004af000 |  |
| 1322 | 0x004af020 | FUN_004af020 | crt_atexit_register_004af020 |  |
| 1323 | 0x004af030 | FUN_004af030 | bendable_plant_ctor |  |
| 1324 | 0x004af080 | FUN_004af080 | bendable_plant_scalar_deleting_dtor |  |
| 1325 | 0x004af0a0 | FUN_004af0a0 | bendable_plant_dtor |  |
| 1326 | 0x004af0d0 | FUN_004af0d0 | bendable_plant_init |  |
| 1327 | 0x004af100 | FUN_004af100 | bendable_plant_save_data |  |
| 1328 | 0x004af150 | FUN_004af150 | bendable_plant_load_data |  |
| 1329 | 0x004af230 | FUN_004af230 | bendable_plant_duplicate_from |  |
| 1330 | 0x004af270 | FUN_004af270 | bendable_plant_game_init |  |
| 1331 | 0x004af6b0 | FUN_004af6b0 | bendable_plant_game_done |  |
| 1332 | 0x004af710 | FUN_004af710 | bendable_plant_push_element |  |
| 1333 | 0x004af7b0 | FUN_004af7b0 | bendable_plant_ai |  |
| 1334 | 0x004afa80 | FUN_004afa80 | crt_static_init_004afa80 |  |
| 1335 | 0x004afa90 | FUN_004afa90 | bendable_plant_static_list_ctor |  |
| 1336 | 0x004afab0 | FUN_004afab0 | crt_atexit_register_004afab0 |  |
| 1337 | 0x004afac0 | FUN_004afac0 | bendable_plant_static_list_free |  |
| 1338 | 0x004afae0 | FUN_004afae0 | bendable_plant_element_vector_insert |  |
| 1339 | 0x004afe10 | FUN_004afe10 | crt_static_init_guard_004afe10 |  |
| 1340 | 0x004afe30 | FUN_004afe30 | crt_atexit_register_004afe30 |  |
| 1341 | 0x004afe40 | FUN_004afe40 | human_look_to | HumanLookTo |
| 1342 | 0x004b0280 | FUN_004b0280 | human_move_to | HumanMoveTo |
| 1343 | 0x004b0d00 | FUN_004b0d00 | crt_static_init_guard_004b0d00 |  |
| 1344 | 0x004b0d20 | FUN_004b0d20 | crt_atexit_register_004b0d20 |  |
| 1345 | 0x004b0d30 | FUN_004b0d30 | crt_static_init_004b0d30 |  |
| 1346 | 0x004b0d40 | FUN_004b0d40 | pedestrian_list_static_ctor | C_traffic_generator::array |
| 1347 | 0x004b0d60 | FUN_004b0d60 | crt_atexit_register_004b0d60 |  |
| 1348 | 0x004b0d70 | FUN_004b0d70 | pedestrian_list_static_dtor |  |
| 1349 | 0x004b0db0 | FUN_004b0db0 | crt_static_init_004b0db0 |  |
| 1350 | 0x004b0dc0 | FUN_004b0dc0 | pedestrian_anim_list_static_ctor | C_traffic_generator::anims |
| 1351 | 0x004b0de0 | FUN_004b0de0 | crt_atexit_register_004b0de0 |  |
| 1352 | 0x004b0df0 | FUN_004b0df0 | pedestrian_anim_list_static_dtor |  |
| 1353 | 0x004b0e30 | thunk_FUN_004b0e40 | thunk_stub_noop_cdecl_004b0e40 |  |
| 1354 | 0x004b0e40 | FUN_004b0e40 | stub_noop_cdecl_004b0e40 |  |
| 1355 | 0x004b0e50 | FUN_004b0e50 | pedestrian_crowd_ctor | C_don_ocas::{ctor} |
| 1356 | 0x004b0e70 | FUN_004b0e70 | pedestrian_crowd_scalar_deleting_dtor |  |
| 1357 | 0x004b0e90 | FUN_004b0e90 | pedestrian_crowd_dtor | C_don_ocas::{dtor} |
| 1358 | 0x004b0ea0 | FUN_004b0ea0 | pedestrian_crowd_game_init | C_don_ocas::GameInit |
| 1359 | 0x004b0eb0 | FUN_004b0eb0 | pedestrian_crowd_is_possible_coll | C_don_ocas::IsPossibleColl |
| 1360 | 0x004b1260 | FUN_004b1260 | pedestrian_crowd_collision | C_don_ocas::Collision |
| 1361 | 0x004b1500 | FUN_004b1500 | pedestrian_crowd_hit | C_don_ocas::Hit |
| 1362 | 0x004b19d0 | FUN_004b19d0 | pedestrian_crowd_get_bound_points_2d | C_don_ocas::GetBoundPoints2D |
| 1363 | 0x004b1c20 | FUN_004b1c20 | pedestrian_manager_ctor | C_traffic_generator::{ctor} |
| 1364 | 0x004b1c90 | FUN_004b1c90 | pedestrian_manager_scalar_deleting_dtor |  |
| 1365 | 0x004b1cb0 | FUN_004b1cb0 | pedestrian_manager_dtor | C_traffic_generator::{dtor} |
| 1366 | 0x004b1d20 | FUN_004b1d20 | pedestrian_manager_duplicate_from | C_traffic_generator::DuplicateFrom |
| 1367 | 0x004b1e80 | FUN_004b1e80 | pedestrian_manager_save_data | C_traffic_generator::SaveData |
| 1368 | 0x004b2030 | FUN_004b2030 | pedestrian_manager_load_data | C_traffic_generator::LoadData |
| 1369 | 0x004b2610 | FUN_004b2610 | pedestrian_manager_save_game_get_size | C_traffic_generator::SaveGameGetSize |
| 1370 | 0x004b2630 | FUN_004b2630 | pedestrian_manager_save_game_save | C_traffic_generator::SaveGameSave |
| 1371 | 0x004b2930 | FUN_004b2930 | pedestrian_manager_save_game_load | C_traffic_generator::SaveGameLoad |
| 1372 | 0x004b2d10 | FUN_004b2d10 | pedestrian_manager_game_init | C_traffic_generator::GameInit |
| 1373 | 0x004b3000 | FUN_004b3000 | pedestrian_manager_game_done | C_traffic_generator::GameDone |
| 1374 | 0x004b3080 | FUN_004b3080 | pedestrian_manager_player_talked_to_pedestrian | C_traffic_generator::PlayerPromluvilNaTehoChuja |
| 1375 | 0x004b30a0 | FUN_004b30a0 | pedestrian_manager_make_pedestrian_watch_player_on_weapon_drawn | C_traffic_generator::PlayerHoMaVytaseneho |
| 1376 | 0x004b3170 | FUN_004b3170 | pedestrian_manager_set_visible | C_traffic_generator::SetVisible |
| 1377 | 0x004b31d0 | FUN_004b31d0 | pedestrian_manager_is_possible_coll_pedestrian | C_traffic_generator::IsPossibleCollElem |
| 1378 | 0x004b33f0 | FUN_004b33f0 | pedestrian_manager_add_pedestrian | C_traffic_generator::AddElement |
| 1379 | 0x004b3510 | FUN_004b3510 | pedestrian_manager_del_pedestrian | C_traffic_generator::DelElement |
| 1380 | 0x004b35c0 | FUN_004b35c0 | pedestrian_manager_ai | C_traffic_generator::AI |
| 1381 | 0x004b3780 | FUN_004b3780 | pedestrian_manager_tick_pedestrians | C_traffic_generator::TickElements |
| 1382 | 0x004b38a0 | FUN_004b38a0 | pedestrian_manager_is_car_claimed_by_pedestrian | C_traffic_generator::IsCarHasOcas |
| 1383 | 0x004b38f0 | FUN_004b38f0 | pedestrian_manager_try_send_pedestrian_to_parked_car | C_traffic_generator::TryGoToAssHole |
| 1384 | 0x004b3f50 | FUN_004b3f50 | pedestrian_manager_player_in_taxi | C_traffic_generator::PlayerInTaxi |
| 1385 | 0x004b3fe0 | FUN_004b3fe0 | pedestrian_manager_generate_passenger_for_taxi | C_traffic_generator::GeneratePassengerForTaxi |
| 1386 | 0x004b4500 | FUN_004b4500 | pedestrian_manager_enable_taxi_driver | C_traffic_generator::EnableTaxiDriver |
| 1387 | 0x004b4520 | FUN_004b4520 | pedestrian_manager_end_all_taxi_waves | C_traffic_generator::ZrusVsechnyMavajici |
| 1388 | 0x004b4560 | FUN_004b4560 | pedestrian_manager_clear_pedestrians | C_traffic_generator::ClearElements |
| 1389 | 0x004b45f0 | FUN_004b45f0 | pedestrian_manager_spawn_cop_from_pedestrian | C_traffic_generator::FizlosNamakatDoAkce |
| 1390 | 0x004b4c60 | FUN_004b4c60 | pedestrian_manager_explosion | C_traffic_generator::Explosion |
| 1391 | 0x004b4f90 | FUN_004b4f90 | pedestrian_manager_fire | C_traffic_generator::Fire |
| 1392 | 0x004b5040 | FUN_004b5040 | pedestrian_manager_get_pedestrian_from_frame | C_traffic_generator::GetElemFromFrame |
| 1393 | 0x004b5070 | FUN_004b5070 | pedestrian_manager_deactivate_all_pedestrians | C_traffic_generator::FreeDynCollElements |
| 1394 | 0x004b50b0 | FUN_004b50b0 | pedestrian_manager_get_nearest_pedestrian_dist_sq | C_traffic_generator::GetNearElementDist2 |
| 1395 | 0x004b5130 | FUN_004b5130 | pedestrian_manager_find_pedestrian_ahead | C_traffic_generator::IsNearElementBefore |
| 1396 | 0x004b51e0 | FUN_004b51e0 | pedestrian_manager_new_panic_at_position | C_traffic_generator::NewPanic |
| 1397 | 0x004b5240 | FUN_004b5240 | pedestrian_manager_new_panic_at_frame | C_traffic_generator::NewPanic |
| 1398 | 0x004b5270 | FUN_004b5270 | pedestrian_manager_railway_in_station | C_traffic_generator::RailwayInStation |
| 1399 | 0x004b5710 | FUN_004b5710 | pedestrian_manager_add_free_person | C_traffic_generator::AddFreePerson |
| 1400 | 0x004b5790 | FUN_004b5790 | pedestrian_manager_add_free_person_from_car | C_traffic_generator::AddFreePersonFromCar |
| 1401 | 0x004b58c0 | FUN_004b58c0 | pedestrian_manager_play_pedestrian_scream | C_traffic_generator::ElementNewScream |
| 1402 | 0x004b5ab0 | FUN_004b5ab0 | pedestrian_manager_release_pedestrians_from_car | C_traffic_generator::VyfunOdTeMasiny |
| 1403 | 0x004b5bb0 | FUN_004b5bb0 | pedestrian_ctor | C_traffic_element::{ctor} |
| 1404 | 0x004b5c10 | FUN_004b5c10 | pedestrian_avoid_obstacle | C_traffic_element::ObejdiTuPicovinu |
| 1405 | 0x004b6070 | FUN_004b6070 | vec2_length |  |
| 1406 | 0x004b6090 | FUN_004b6090 | vec2_dot |  |
| 1407 | 0x004b60b0 | FUN_004b60b0 | pedestrian_advance_to_next_node | C_traffic_element::SetNewNode |
| 1408 | 0x004b6840 | FUN_004b6840 | pedestrian_wave_at_taxi | C_traffic_element::MavaNaTaxi |
| 1409 | 0x004b74f0 | FUN_004b74f0 | pedestrian_end_taxi_wave | C_traffic_element::EndMavaniNaTaxi |
| 1410 | 0x004b7520 | FUN_004b7520 | pedestrian_tick | C_traffic_element::Tick |
| 1411 | 0x004bacf0 | FUN_004bacf0 | pedestrian_update_model | C_traffic_element::UpdateModel |
| 1412 | 0x004bae30 | FUN_004bae30 | pedestrian_set_model | C_traffic_element::SetModel |
| 1413 | 0x004bb240 | FUN_004bb240 | pedestrian_create_model | C_traffic_element::CreateModel |
| 1414 | 0x004bb6a0 | FUN_004bb6a0 | pedestrian_set_alpha | C_traffic_element::SetAlpha |
| 1415 | 0x004bb6c0 | FUN_004bb6c0 | pedestrian_destroy_model | C_traffic_element::DestroyModel |
| 1416 | 0x004bb7a0 | FUN_004bb7a0 | pedestrian_activate_free | C_traffic_element::ActivateFree |
| 1417 | 0x004bbff0 | FUN_004bbff0 | pedestrian_activate | C_traffic_element::Activate |
| 1418 | 0x004bcfc0 | FUN_004bcfc0 | pedestrian_deactivate | C_traffic_element::Deactivate |
| 1419 | 0x004bd0d0 | FUN_004bd0d0 | pedestrian_hit | C_traffic_element::Hit |
| 1420 | 0x004bdc50 | FUN_004bdc50 | pedestrian_set_anim | C_traffic_element::SetAnim |
| 1421 | 0x004bde50 | FUN_004bde50 | pedestrian_set_panic | C_traffic_element::SetPanic |
| 1422 | 0x004bdfb0 | FUN_004bdfb0 | pedestrian_set_mega_panic | C_traffic_element::SetMegaPanic |
| 1423 | 0x004be050 | FUN_004be050 | pedestrian_on_player_talk | C_traffic_element::PlayerNaMneMluvi |
| 1424 | 0x004be0d0 | FUN_004be0d0 | pedestrian_greet_player | C_traffic_element::Karle_hoj |
| 1425 | 0x004be250 | FUN_004be250 | pedestrian_drop_umbrella | C_traffic_element::DropUmbrella |
| 1426 | 0x004be390 | FUN_004be390 | pedestrian_spawn_gang_on_hit | C_traffic_element::CreateMafianos |
| 1427 | 0x004be660 | FUN_004be660 | chunk_vector_allocate |  |
| 1428 | 0x004be680 | FUN_004be680 | chunk_ctor |  |
| 1429 | 0x004be6b0 | FUN_004be6b0 | chunk_open_file | C_chunk::OpenFile |
| 1430 | 0x004be720 | FUN_004be720 | chunk_close | C_chunk::Close |
| 1431 | 0x004be810 | FUN_004be810 | crt_static_init_guard_004be810 |  |
| 1432 | 0x004be830 | FUN_004be830 | crt_atexit_register_004be830 |  |
| 1433 | 0x004be840 | FUN_004be840 | physical_object_build_world_inertia_tensor |  |
| 1434 | 0x004be910 | FUN_004be910 | physical_object_ctor | C_PhThing::{ctor} |
| 1435 | 0x004bea00 | FUN_004bea00 | physical_object_scalar_deleting_dtor |  |
| 1436 | 0x004bea20 | FUN_004bea20 | physical_object_dtor | C_PhThing::{dtor} |
| 1437 | 0x004bea60 | FUN_004bea60 | physical_object_init | C_PhThing::Init |
| 1438 | 0x004bebd0 | FUN_004bebd0 | physical_object_game_init | C_PhThing::GameInit |
| 1439 | 0x004beff0 | FUN_004beff0 | physical_object_update_rest_state | C_PhThing::FindActors |
| 1440 | 0x004bf0c0 | FUN_004bf0c0 | physical_object_game_done | C_PhThing::GameDone |
| 1441 | 0x004bf120 | FUN_004bf120 | physical_object_ai_tick | C_PhThing::AI |
| 1442 | 0x004bf260 | FUN_004bf260 | physical_object_update | C_PhThing::Update |
| 1443 | 0x004bfa80 | FUN_004bfa80 | physical_object_on_act_state_changed | C_PhThing::ChangeState |
| 1444 | 0x004bfae0 | FUN_004bfae0 | physical_object_move_wheel | C_PhThing::MoveWheel |
| 1445 | 0x004c1dd0 | FUN_004c1dd0 | physical_object_move_box | C_PhThing::MoveBox |
| 1446 | 0x004c2a40 | FUN_004c2a40 | physical_object_integrate_transform | C_PhThing::MoveCount |
| 1447 | 0x004c31a0 | FUN_004c31a0 | physical_object_accumulate_contact_impulse_diag | C_PhThing::CollisionCount |
| 1448 | 0x004c3890 | FUN_004c3890 | physical_object_accumulate_contact_impulse_matrix | C_PhThing::CollisionCount |
| 1449 | 0x004c4000 | FUN_004c4000 | physical_object_init_wheel | C_PhThing::InitWheel |
| 1450 | 0x004c41a0 | FUN_004c41a0 | physical_object_init_box | C_PhThing::InitBox |
| 1451 | 0x004c4880 | FUN_004c4880 | physical_object_on_collision | C_PhThing::Collision |
| 1452 | 0x004c4b10 | FUN_004c4b10 | physical_object_hit | C_PhThing::Hit |
| 1453 | 0x004c5260 | FUN_004c5260 | physical_object_get_contact_terms | C_PhThing::GetRVelMassJ |
| 1454 | 0x004c53f0 | FUN_004c53f0 | physical_object_apply_impulse_at_point | C_PhThing::ExternalImpuls |
| 1455 | 0x004c5590 | FUN_004c5590 | physical_object_remove_surrounding_actor | C_PhThing::InvalidateActor |
| 1456 | 0x004c5640 | FUN_004c5640 | physical_object_collision_filter | C_PhThing::CollisionFilter |
| 1457 | 0x004c5670 | FUN_004c5670 | physical_object_refresh_world_transform | C_PhThing::UpdateImportantVariables |
| 1458 | 0x004c5750 | FUN_004c5750 | physical_object_save_game_get_size | C_PhThing::SaveGameGetSize |
| 1459 | 0x004c5760 | FUN_004c5760 | physical_object_save_game_save | C_PhThing::SaveGameSave |
| 1460 | 0x004c5940 | FUN_004c5940 | physical_object_save_game_load | C_PhThing::SaveGameLoad |
| 1461 | 0x004c5d90 | FUN_004c5d90 | physical_object_broadcast_state_net_event_msg |  |
| 1462 | 0x004c5e40 | FUN_004c5e40 | physical_object_post_state_net_event_msg |  |
| 1463 | 0x004c5fd0 | FUN_004c5fd0 | physical_object_net_restart_local_sim |  |
| 1464 | 0x004c6150 | FUN_004c6150 | physical_object_apply_net_event_msg |  |
| 1465 | 0x004c6380 | FUN_004c6380 | physical_object_net_snapshot_state |  |
| 1466 | 0x004c6480 | FUN_004c6480 | physical_object_net_apply_remote_state | C_PhThing::NetDirect |
| 1467 | 0x004c6b20 | FUN_004c6b20 | i3d_frame_set_direction |  |
| 1468 | 0x004c6b50 | FUN_004c6b50 | crt_static_init_guard_004c6b50 |  |
| 1469 | 0x004c6b70 | FUN_004c6b70 | crt_atexit_register_004c6b70 |  |
| 1470 | 0x004c6b80 | FUN_004c6b80 | player_set_navi_point | SetPlayerNaviPoint |
| 1471 | 0x004c6b90 | FUN_004c6b90 | player_ctor | C_player::{ctor} |
| 1472 | 0x004c6c20 | FUN_004c6c20 | player_scalar_deleting_dtor |  |
| 1473 | 0x004c6c40 | FUN_004c6c40 | player_dtor | C_player::{dtor} |
| 1474 | 0x004c6c80 | FUN_004c6c80 | player_init | C_player::Init |
| 1475 | 0x004c6cb0 | thunk_FUN_0048f880 | thunk_human_duplicate_from | C_player::DuplicateFrom |
| 1476 | 0x004c6cc0 | FUN_004c6cc0 | player_save_data | C_player::SaveData |
| 1477 | 0x004c6ce0 | FUN_004c6ce0 | player_load_data | C_player::LoadData |
| 1478 | 0x004c6d00 | FUN_004c6d00 | player_save_game_get_size | C_player::SaveGameGetSize |
| 1479 | 0x004c6d10 | FUN_004c6d10 | player_save_game_save | C_player::SaveGameSave |
| 1480 | 0x004c6da0 | FUN_004c6da0 | player_save_game_load | C_player::SaveGameLoad |
| 1481 | 0x004c6ec0 | thunk_FUN_00490a20 | thunk_human_death | C_player::Death |
| 1482 | 0x004c6ed0 | FUN_004c6ed0 | player_ai_walk | C_player::AI_walk |
| 1483 | 0x004c91f0 | FUN_004c91f0 | player_ai_drive | C_player::AI_drive |
| 1484 | 0x004ca570 | FUN_004ca570 | player_ai_car_fire | C_player::AI_car_fire |
| 1485 | 0x004cb1b0 | FUN_004cb1b0 | player_ai | C_player::AI |
| 1486 | 0x004cba50 | thunk_FUN_00492ff0 | thunk_human_update | C_player::Update |
| 1487 | 0x004cba60 | FUN_004cba60 | player_game_init | C_player::GameInit |
| 1488 | 0x004cbb80 | FUN_004cbb80 | player_game_done | C_player::GameDone |
| 1489 | 0x004cbbc0 | FUN_004cbbc0 | player_collision | C_player::Collision |
| 1490 | 0x004cbc10 | FUN_004cbc10 | player_hit | C_player::Hit |
| 1491 | 0x004cbcc0 | FUN_004cbcc0 | player_update_controls | C_player::UpdateControls |
| 1492 | 0x004cbd00 | FUN_004cbd00 | player_do_action_button_on_foot | C_player::Do_AB_OwnerNULL |
| 1493 | 0x004cc950 | FUN_004cc950 | player_do_action_button_in_car | C_player::Do_AB_OwnerCar |
| 1494 | 0x004ccc80 | FUN_004ccc80 | player_indicator_set_aimed | C_player::IndicatorSetAimed |
| 1495 | 0x004ccd20 | FUN_004ccd20 | player_add_shot_accuracy_penalty | C_player::ShootDeAimed |
| 1496 | 0x004ccdd0 | FUN_004ccdd0 | player_set_sniper_mode | C_player::SetSniperMode |
| 1497 | 0x004ccf00 | FUN_004ccf00 | player_change_model | C_player::_intern_ChangeModel |
| 1498 | 0x004ccf80 | FUN_004ccf80 | player_try_use_car | C_player::Can_UseCar |
| 1499 | 0x004cd5c0 | FUN_004cd5c0 | player_show_car_damage_bar | C_player::SetCarDamage |
| 1500 | 0x004cd5f0 | FUN_004cd5f0 | player_invalidate_car_damage_bar | C_player::InvalidateCarDamage |
| 1501 | 0x004cd640 | FUN_004cd640 | player_add_alpha_object | C_player::AddAlphaObject |
| 1502 | 0x004cd680 | FUN_004cd680 | player_alpha_list_add_frame_tree | C_player::AddAlphaObject |
| 1503 | 0x004cd700 | FUN_004cd700 | player_del_alpha_object | C_player::DelAlphaObject |
| 1504 | 0x004cd720 | FUN_004cd720 | player_alpha_list_remove_frame_tree |  |
| 1505 | 0x004cd7a0 | FUN_004cd7a0 | player_set_alpha | C_player::SetAlpha |
| 1506 | 0x004cd800 | FUN_004cd800 | player_show_weapon_name | C_player::ShowWeaponName |
| 1507 | 0x004cd890 | FUN_004cd890 | player_lock_controls | C_player::LockControls |
| 1508 | 0x004cda20 | FUN_004cda20 | player_set_hand_scale |  |
| 1509 | 0x004cdb40 | FUN_004cdb40 | player_fade_reset | C_player::FadeReset |
| 1510 | 0x004cdb60 | FUN_004cdb60 | std_vector4_insert_n |  |
| 1511 | 0x004cdd70 | FUN_004cdd70 | crt_static_init_guard_004cdd70 |  |
| 1512 | 0x004cdd90 | FUN_004cdd90 | crt_atexit_register_004cdd90 |  |
| 1513 | 0x004cdda0 | FUN_004cdda0 | crt_static_init_004cdda0 |  |
| 1514 | 0x004cddb0 | FUN_004cddb0 | public_physics_list_init |  |
| 1515 | 0x004cddd0 | FUN_004cddd0 | crt_atexit_register_004cddd0 |  |
| 1516 | 0x004cdde0 | FUN_004cdde0 | public_physics_list_free |  |
| 1517 | 0x004cde20 | FUN_004cde20 | public_physics_ctor | C_public_physics::{ctor} |
| 1518 | 0x004cde40 | FUN_004cde40 | public_physics_scalar_deleting_dtor |  |
| 1519 | 0x004cde60 | FUN_004cde60 | public_physics_dtor | C_public_physics::{dtor} |
| 1520 | 0x004cde70 | thunk_FUN_004bf120 | thunk_physical_object_ai_tick |  |
| 1521 | 0x004cde80 | thunk_FUN_004bf260 | thunk_physical_object_update |  |
| 1522 | 0x004cde90 | thunk_FUN_004c6480 | thunk_physical_object_net_apply_remote_state |  |
| 1523 | 0x004cdea0 | thunk_FUN_004bea60 | thunk_physical_object_init |  |
| 1524 | 0x004cdeb0 | FUN_004cdeb0 | public_physics_game_init | C_public_physics::GameInit |
| 1525 | 0x004cdf90 | FUN_004cdf90 | public_physics_game_done | C_public_physics::GameDone |
| 1526 | 0x004cdfb0 | FUN_004cdfb0 | public_physics_on_collision | C_public_physics::Collision |
| 1527 | 0x004cdfc0 | FUN_004cdfc0 | public_physics_hit | C_public_physics::Hit |
| 1528 | 0x004cdfd0 | FUN_004cdfd0 | crt_static_init_guard_004cdfd0 |  |
| 1529 | 0x004cdff0 | FUN_004cdff0 | crt_atexit_register_004cdff0 |  |
| 1530 | 0x004ce000 | FUN_004ce000 | pumper_ctor | C_pumper::{ctor} |
| 1531 | 0x004ce1a0 | FUN_004ce1a0 | pumper_scalar_deleting_dtor |  |
| 1532 | 0x004ce1c0 | FUN_004ce1c0 | pumper_dtor | C_pumper::{dtor} |
| 1533 | 0x004ce220 | FUN_004ce220 | pumper_save_data | C_pumper::SaveData |
| 1534 | 0x004ce300 | FUN_004ce300 | pumper_load_data | C_pumper::LoadData |
| 1535 | 0x004ce500 | FUN_004ce500 | pumper_duplicate_from | C_pumper::DuplicateFrom |
| 1536 | 0x004ce540 | FUN_004ce540 | pumper_game_init | C_pumper::GameInit |
| 1537 | 0x004d0180 | FUN_004d0180 | pumper_game_done | C_pumper::GameDone |
| 1538 | 0x004d03e0 | FUN_004d03e0 | pumper_on_collision | C_pumper::Collision |
| 1539 | 0x004d0460 | FUN_004d0460 | pumper_hit | C_pumper::Hit |
| 1540 | 0x004d04d0 | FUN_004d04d0 | pumper_ai | C_pumper::AI |
| 1541 | 0x004d0c10 | FUN_004d0c10 | pumper_ai_walk | C_pumper::AI_walk |
| 1542 | 0x004d0e80 | FUN_004d0e80 | pumper_ai_turn | C_pumper::AI_turn |
| 1543 | 0x004d1000 | FUN_004d1000 | pumper_update | C_pumper::Update |
| 1544 | 0x004d12d0 | FUN_004d12d0 | pumper_snap_to_ground | C_pumper::ground |
| 1545 | 0x004d1380 | FUN_004d1380 | pumper_set_anim | C_pumper::SetAnim |
| 1546 | 0x004d13f0 | FUN_004d13f0 | pumper_choose_next_action | C_pumper::ChooseNextAction |
| 1547 | 0x004d1bb0 | FUN_004d1bb0 | pumper_advance_to_next_node | C_pumper::NextNode |
| 1548 | 0x004d1c30 | FUN_004d1c30 | pumper_start_refuel | C_pumper::Start |
| 1549 | 0x004d2070 | FUN_004d2070 | pumper_can_pump | C_pumper::CanPump |
| 1550 | 0x004d2460 | FUN_004d2460 | pumper_notify_callback | pumper_notify_callback |
| 1551 | 0x004d2840 | FUN_004d2840 | pumper_start_pump_fuel | C_pumper::StartPumpFuel |
| 1552 | 0x004d2870 | FUN_004d2870 | pumper_end_pump_fuel | C_pumper::EndPumpFuel |
| 1553 | 0x004d28c0 | FUN_004d28c0 | pumper_init_sounds | C_pumper::InitSounds |
| 1554 | 0x004d28e0 | FUN_004d28e0 | pumper_init_sound | C_pumper::InitSound |
| 1555 | 0x004d29a0 | FUN_004d29a0 | pumper_release_sounds | C_pumper::ReleaseSounds |
| 1556 | 0x004d29e0 | FUN_004d29e0 | pumper_get_special_anim_name | C_pumper::GetSpecAnimName |
| 1557 | 0x004d2a70 | FUN_004d2a70 | pumper_panic | C_pumper::PumperPanic |
| 1558 | 0x004d2c30 | FUN_004d2c30 | pumper_save_game_get_size | C_pumper::SaveGameGetSize |
| 1559 | 0x004d2c40 | FUN_004d2c40 | pumper_save_game_save | C_pumper::SaveGameSave |
| 1560 | 0x004d2e80 | FUN_004d2e80 | pumper_save_game_load | C_pumper::SaveGameLoad |
| 1561 | 0x004d3210 | FUN_004d3210 | pumper_play_sound | C_pumper::PlaySound |
| 1562 | 0x004d3240 | FUN_004d3240 | pumper_release_colls | C_pumper::ReleaseColls |
| 1563 | 0x004d32b0 | FUN_004d32b0 | pumper_is_pumping | C_pumper::IsPumping |
| 1564 | 0x004d32d0 | FUN_004d32d0 | pumper_is_in_panic | C_pumper::IsInPanic |
| 1565 | 0x004d32e0 | FUN_004d32e0 | pumper_script_can_pump | C_pumper::ScriptCanPump |
| 1566 | 0x004d3330 | FUN_004d3330 | std_vector8_erase_at |  |
| 1567 | 0x004d33a0 | FUN_004d33a0 | crt_static_init_guard_004d33a0 |  |
| 1568 | 0x004d33c0 | FUN_004d33c0 | crt_atexit_register_004d33c0 |  |
| 1569 | 0x004d33d0 | FUN_004d33d0 | racecam_ctor | C_racecam::{ctor} |
| 1570 | 0x004d33f0 | FUN_004d33f0 | racecam_scalar_deleting_dtor |  |
| 1571 | 0x004d3410 | FUN_004d3410 | racecam_dtor | C_racecam::{dtor} |
| 1572 | 0x004d3420 | FUN_004d3420 | racecam_init | C_racecam::Init |
| 1573 | 0x004d3480 | FUN_004d3480 | racecam_save_data | C_racecam::SaveData |
| 1574 | 0x004d34b0 | FUN_004d34b0 | racecam_load_data | C_racecam::LoadData |
| 1575 | 0x004d3530 | FUN_004d3530 | racecam_duplicate_from | C_racecam::DuplicateFrom |
| 1576 | 0x004d3550 | FUN_004d3550 | racecam_game_init | C_racecam::GameInit |
| 1577 | 0x004d35a0 | FUN_004d35a0 | racecam_game_done | C_racecam::GameDone |
| 1578 | 0x004d35c0 | FUN_004d35c0 | crt_static_init_guard_004d35c0 |  |
| 1579 | 0x004d35e0 | FUN_004d35e0 | crt_atexit_register_004d35e0 |  |
| 1580 | 0x004d35f0 | FUN_004d35f0 | rail_seats_find_free_seat | S_RailSeats::FindFreeSeat |
| 1581 | 0x004d3740 | FUN_004d3740 | rail_seats_count_free | S_RailSeats::NumFreeSeats |
| 1582 | 0x004d3780 | FUN_004d3780 | rail_seats_get_seat_pos | S_RailSeats::GetSeatPos |
| 1583 | 0x004d39a0 | FUN_004d39a0 | rail_seats_get_seat_dir | S_RailSeats::GetSeatDir |
| 1584 | 0x004d3a10 | FUN_004d3a10 | rail_seats_empty_seat | S_RailSeats::EmptySeat |
| 1585 | 0x004d3aa0 | FUN_004d3aa0 | rail_seats_reserve_seat | S_RailSeats::ReserveSeat |
| 1586 | 0x004d3b20 | FUN_004d3b20 | rail_seats_occupy_seat | S_RailSeats::FullSeat |
| 1587 | 0x004d3c10 | FUN_004d3c10 | rail_seats_add_row | S_RailSeats::AddRow |
| 1588 | 0x004d3c90 | FUN_004d3c90 | rail_seats_duplicate_rows | S_RailSeats::Duplicate |
| 1589 | 0x004d3d50 | FUN_004d3d50 | rail_seats_init | S_RailSeats::Init |
| 1590 | 0x004d41c0 | FUN_004d41c0 | rail_seats_destroy | S_RailSeats::Destroy |
| 1591 | 0x004d4210 | FUN_004d4210 | rail_passenger_ctor | S_rail_human::{ctor} |
| 1592 | 0x004d4240 | FUN_004d4240 | rail_passenger_rotate_to | S_rail_human::RotateTo |
| 1593 | 0x004d44b0 | FUN_004d44b0 | rail_passenger_move_to | S_rail_human::MoveTo |
| 1594 | 0x004d50d0 | FUN_004d50d0 | rail_passenger_set_anim | S_rail_human::SetAnim |
| 1595 | 0x004d5160 | FUN_004d5160 | rail_passenger_kill | S_rail_human::Kill |
| 1596 | 0x004d5200 | FUN_004d5200 | rail_passenger_create_coll | S_rail_human::CreateColl |
| 1597 | 0x004d5360 | FUN_004d5360 | rail_passenger_update_coll | S_rail_human::UpdateColl |
| 1598 | 0x004d5430 | FUN_004d5430 | rail_passenger_done | S_rail_human::Done |
| 1599 | 0x004d5480 | FUN_004d5480 | rail_passenger_do_panic | S_rail_human::DoPanic |
| 1600 | 0x004d5520 | FUN_004d5520 | railway_traffic_ai | C_railway::trafficAI |
| 1601 | 0x004d71e0 | FUN_004d71e0 | rail_passenger_is_woman | S_rail_human::IsWoman |
| 1602 | 0x004d7250 | FUN_004d7250 | railway_any_passenger_wants_off | C_railway::WantLeave |
| 1603 | 0x004d72b0 | FUN_004d72b0 | railway_anyone_waiting_to_board | C_railway::WantEnter |
| 1604 | 0x004d7300 | FUN_004d7300 | crt_static_init_rail_clamp_candidates |  |
| 1605 | 0x004d7310 | FUN_004d7310 | crt_zero_init_rail_clamp_candidates |  |
| 1606 | 0x004d7330 | FUN_004d7330 | crt_atexit_register_004d7330 |  |
| 1607 | 0x004d7340 | FUN_004d7340 | crt_atexit_free_rail_clamp_candidates |  |
| 1608 | 0x004d7360 | FUN_004d7360 | crt_static_init_rail_schedule_log |  |
| 1609 | 0x004d7370 | FUN_004d7370 | crt_zero_init_rail_schedule_log |  |
| 1610 | 0x004d7390 | FUN_004d7390 | crt_atexit_register_004d7390 |  |
| 1611 | 0x004d73a0 | FUN_004d73a0 | crt_atexit_free_rail_schedule_log |  |
| 1612 | 0x004d73e0 | FUN_004d73e0 | crt_static_init_rail_semaphore_list |  |
| 1613 | 0x004d73f0 | FUN_004d73f0 | crt_zero_init_rail_semaphore_list |  |
| 1614 | 0x004d7410 | FUN_004d7410 | crt_atexit_register_004d7410 |  |
| 1615 | 0x004d7420 | FUN_004d7420 | crt_atexit_free_rail_semaphore_list |  |
| 1616 | 0x004d7460 | FUN_004d7460 | crt_static_init_railway_list |  |
| 1617 | 0x004d7470 | FUN_004d7470 | crt_zero_init_railway_list |  |
| 1618 | 0x004d7490 | FUN_004d7490 | crt_atexit_register_004d7490 |  |
| 1619 | 0x004d74a0 | FUN_004d74a0 | crt_atexit_free_railway_list |  |
| 1620 | 0x004d74e0 | FUN_004d74e0 | railway_ctor | C_railway::{ctor} |
| 1621 | 0x004d7670 | FUN_004d7670 | railway_scalar_deleting_dtor |  |
| 1622 | 0x004d7690 | FUN_004d7690 | railway_dtor | C_railway::{dtor} |
| 1623 | 0x004d77f0 | FUN_004d77f0 | railway_init | C_railway::Init |
| 1624 | 0x004d7820 | FUN_004d7820 | railway_duplicate_from | C_railway::DuplicateFrom |
| 1625 | 0x004d7850 | FUN_004d7850 | railway_save_data | C_railway::SaveData |
| 1626 | 0x004d7950 | FUN_004d7950 | railway_load_data | C_railway::LoadData |
| 1627 | 0x004d7b60 | FUN_004d7b60 | railway_build_back_node_vector | C_railway::CreateBackVector |
| 1628 | 0x004d7ce0 | FUN_004d7ce0 | railway_build_forward_node_vector | C_railway::CreateForwardVector |
| 1629 | 0x004d8050 | FUN_004d8050 | railway_is_semaphore_point | C_railway::IsSemaPoint |
| 1630 | 0x004d8440 | FUN_004d8440 | railway_must_stop | C_railway::MustStop |
| 1631 | 0x004d86d0 | FUN_004d86d0 | railway_find_back_pos | C_railway::FindBackPos |
| 1632 | 0x004d8c60 | FUN_004d8c60 | railway_spring_process | C_railway::SpringProcess |
| 1633 | 0x004d9070 | FUN_004d9070 | railway_ai | C_railway::AI |
| 1634 | 0x004da400 | FUN_004da400 | railway_update | C_railway::Update |
| 1635 | 0x004db2c0 | FUN_004db2c0 | railway_game_init | C_railway::GameInit |
| 1636 | 0x004dded0 | FUN_004dded0 | railway_game_done | C_railway::GameDone |
| 1637 | 0x004de460 | FUN_004de460 | railway_on_collision | C_railway::Collision |
| 1638 | 0x004de7c0 | FUN_004de7c0 | railway_is_possible_coll | C_railway::IsPossibleColl |
| 1639 | 0x004def00 | FUN_004def00 | railway_fill_track_curve | C_railway::FillCurve |
| 1640 | 0x004df6b0 | FUN_004df6b0 | railway_clamp_movement_against_traffic | C_railway::PreduceMovement |
| 1641 | 0x004e0790 | FUN_004e0790 | railway_place_on_track | C_railway::Place |
| 1642 | 0x004e1070 | FUN_004e1070 | railway_collide | C_railway::Collide |
| 1643 | 0x004e1d90 | FUN_004e1d90 | railway_hit | C_railway::Hit |
| 1644 | 0x004e2710 | FUN_004e2710 | railway_get_safe_speed | C_railway::GetSafeSpeed |
| 1645 | 0x004e3b30 | FUN_004e3b30 | railway_add_person | C_railway::AddPerson |
| 1646 | 0x004e3f10 | FUN_004e3f10 | railway_remove_person | C_railway::RemovePerson |
| 1647 | 0x004e3fb0 | FUN_004e3fb0 | railway_get_nearest_vagon | C_railway::GetNearestVagon |
| 1648 | 0x004e4050 | FUN_004e4050 | railway_enumerate_scene_frames | C_railway::Enumerate |
| 1649 | 0x004e4c50 | FUN_004e4c50 | railway_cb_enum_dummies | cbEnumDummies |
| 1650 | 0x004e57e0 | FUN_004e57e0 | railway_cb_enum_dummies_vagons | cbEnumDummiesVagons |
| 1651 | 0x004e5a20 | FUN_004e5a20 | railway_cb_enum_lights | cbEnumLights |
| 1652 | 0x004e5f10 | FUN_004e5f10 | railway_cb_enum_destroy | cbEnumDestroy |
| 1653 | 0x004e5f20 | FUN_004e5f20 | rail_entries_add_entry | rail_entries::AddEntry |
| 1654 | 0x004e5f90 | FUN_004e5f90 | rail_entries_destroy | rail_entries::Destroy |
| 1655 | 0x004e5fa0 | FUN_004e5fa0 | railway_destroy_entries | C_railway::DestroyEntries |
| 1656 | 0x004e5fd0 | FUN_004e5fd0 | railway_gen_entries | C_railway::GenEntries |
| 1657 | 0x004e6350 | FUN_004e6350 | railway_add_outer_entries | C_railway::AddOuterEntries |
| 1658 | 0x004e63b0 | FUN_004e63b0 | railway_add_inner_entries | C_railway::AddInnerEntries |
| 1659 | 0x004e6410 | FUN_004e6410 | railway_remove_outer_entries | C_railway::RemoveOuterEntries |
| 1660 | 0x004e6470 | FUN_004e6470 | railway_remove_inner_entries | C_railway::RemoveInnerEntries |
| 1661 | 0x004e64d0 | FUN_004e64d0 | railway_get_outer_entry_for_actor | C_railway::GetOuterEntry |
| 1662 | 0x004e69d0 | FUN_004e69d0 | railway_get_outer_entry_at | C_railway::GetOuterEntry |
| 1663 | 0x004e6d00 | FUN_004e6d00 | railway_get_inner_entry_for_actor | C_railway::GetInnerEntry |
| 1664 | 0x004e6f80 | FUN_004e6f80 | railway_get_entry_for_actor | C_railway::GetEntry |
| 1665 | 0x004e6fc0 | FUN_004e6fc0 | railway_get_outer_entry_near | C_railway::GetOuterEntry |
| 1666 | 0x004e7450 | FUN_004e7450 | railway_get_inner_entry_near | C_railway::GetInnerEntry |
| 1667 | 0x004e7650 | FUN_004e7650 | railway_export_entries | C_railway::ExportEntries |
| 1668 | 0x004e7a50 | FUN_004e7a50 | rail_doors_open | rail_doors::Open |
| 1669 | 0x004e7a60 | FUN_004e7a60 | rail_doors_close | rail_doors::Close |
| 1670 | 0x004e7a70 | FUN_004e7a70 | rail_doors_place | rail_doors::Place |
| 1671 | 0x004e7b50 | FUN_004e7b50 | rail_doors_tick | rail_doors::Tick |
| 1672 | 0x004e7bc0 | FUN_004e7bc0 | rail_doors_add_leaf_frame | rail_doors::AddFrame |
| 1673 | 0x004e7c90 | FUN_004e7c90 | rail_doors_init | rail_doors::Init |
| 1674 | 0x004e7ce0 | FUN_004e7ce0 | rail_doors_destroy | rail_doors::Destroy |
| 1675 | 0x004e7d10 | FUN_004e7d10 | rail_glass_game_init | rail_glass::GameInit |
| 1676 | 0x004e7d90 | FUN_004e7d90 | rail_glass_game_done | rail_glass::GameDone |
| 1677 | 0x004e7df0 | FUN_004e7df0 | rail_glass_add_window | rail_glass::AddWindow |
| 1678 | 0x004e7e90 | FUN_004e7e90 | rail_window_restore | rail_window::Restore |
| 1679 | 0x004e7f00 | FUN_004e7f00 | rail_window_hit | rail_window::Hit |
| 1680 | 0x004e7fe0 | FUN_004e7fe0 | railway_sweep_sphere_along_curve | C_railway::PreduceMovementCar |
| 1681 | 0x004e8250 | FUN_004e8250 | railway_is_in_station | C_railway::isInStation |
| 1682 | 0x004e8270 | FUN_004e8270 | railway_is_in_stop | C_railway::isInStop |
| 1683 | 0x004e82d0 | FUN_004e82d0 | railway_get_vagon | C_railway::GetVagon |
| 1684 | 0x004e8300 | FUN_004e8300 | railway_save_game_get_size | C_railway::SaveGameGetSize |
| 1685 | 0x004e8310 | FUN_004e8310 | railway_save_game_save | C_railway::SaveGameSave |
| 1686 | 0x004e8440 | FUN_004e8440 | railway_save_game_load | C_railway::SaveGameLoad |
| 1687 | 0x004e8bd0 | FUN_004e8bd0 | railway_lock | C_railway::Lock |
| 1688 | 0x004e8be0 | FUN_004e8be0 | railway_unlock | C_railway::Unlock |
| 1689 | 0x004e8c00 | FUN_004e8c00 | railway_is_on_schedule | C_railway::IsOnSchedule |
| 1690 | 0x004e8e50 | FUN_004e8e50 | railway_do_panic | C_railway::DoPanic |
| 1691 | 0x004e8ee0 | FUN_004e8ee0 | railway_set_state | C_railway::SetRLWState |
| 1692 | 0x004e8f10 | FUN_004e8f10 | railway_update_curve_squeal_sound | C_railway::DoBrousicka |
| 1693 | 0x004e92b0 | FUN_004e92b0 | railway_on_act_state_changed | C_railway::ChangeState |
| 1694 | 0x004e93d0 | FUN_004e93d0 | railway_get_bound_points_2d | C_railway::GetBoundPoints2D |
| 1695 | 0x004e99e0 | FUN_004e99e0 | railway_find_vehicle_ahead |  |
| 1696 | 0x004e9b60 | FUN_004e9b60 | railway_hold_behind_vehicle_ahead |  |
| 1697 | 0x004e9bf0 | FUN_004e9bf0 | railway_set_rain_dummy | C_railway::SetRainDummy |
| 1698 | 0x004e9cc0 | FUN_004e9cc0 | railway_get_material_id | C_railway::GetMaterialID |
| 1699 | 0x004e9cd0 | FUN_004e9cd0 | railway_get_actor_speed | C_railway::GetActorSpeed |
| 1700 | 0x004e9e90 | FUN_004e9e90 | vec_count_stride_0x24 |  |
| 1701 | 0x004e9ec0 | FUN_004e9ec0 | vec_insert_stride_0x10 |  |
| 1702 | 0x004ea0a0 | FUN_004ea0a0 | vec_erase_tail_stride_0x24 |  |
| 1703 | 0x004ea0f0 | FUN_004ea0f0 | stl_fill_n_stride_0x24 |  |
| 1704 | 0x004ea120 | FUN_004ea120 | stl_fill_n_dwords |  |
| 1705 | 0x004ea150 | FUN_004ea150 | stl_copy_stride_0x24 |  |
| 1706 | 0x004ea180 | FUN_004ea180 | stl_copy_construct_4dwords |  |
| 1707 | 0x004ea1a0 | FUN_004ea1a0 | crt_static_init_guard_004ea1a0 |  |
| 1708 | 0x004ea1c0 | FUN_004ea1c0 | crt_atexit_register_004ea1c0 |  |
| 1709 | 0x004ea1d0 | FUN_004ea1d0 | seagull_flock_ctor |  |
| 1710 | 0x004ea220 | FUN_004ea220 | seagull_flock_scalar_deleting_dtor |  |
| 1711 | 0x004ea240 | FUN_004ea240 | seagull_flock_dtor |  |
| 1712 | 0x004ea280 | FUN_004ea280 | seagull_flock_game_init |  |
| 1713 | 0x004eb090 | FUN_004eb090 | seagull_flock_game_done |  |
| 1714 | 0x004eb100 | FUN_004eb100 | seagull_flock_destroy_on_collision |  |
| 1715 | 0x004eb160 | FUN_004eb160 | seagull_flock_hit |  |
| 1716 | 0x004eb310 | FUN_004eb310 | seagull_flock_ai |  |
| 1717 | 0x004ec0a0 | FUN_004ec0a0 | vec3_cross_of_edges |  |
| 1718 | 0x004ec120 | FUN_004ec120 | vec_count_stride_0x5c |  |
| 1719 | 0x004ec150 | FUN_004ec150 | stl_fill_n_stride_0x5c |  |
| 1720 | 0x004ec180 | FUN_004ec180 | crt_static_init_guard_004ec180 |  |
| 1721 | 0x004ec1a0 | FUN_004ec1a0 | crt_atexit_register_004ec1a0 |  |
| 1722 | 0x004ec1b0 | FUN_004ec1b0 | pursuit_event_ctor | C_sledovacka::{ctor} |
| 1723 | 0x004ec1e0 | FUN_004ec1e0 | pursuit_event_set_change_tables | C_sledovacka::EnableChangeTables |
| 1724 | 0x004ec1f0 | FUN_004ec1f0 | pursuit_event_init | C_sledovacka::Init |
| 1725 | 0x004ec4f0 | FUN_004ec4f0 | pursuit_event_done | C_sledovacka::Done |
| 1726 | 0x004ec590 | FUN_004ec590 | pursuit_event_is_car_chased | C_sledovacka::HoniMiHoNekdoZKanclu |
| 1727 | 0x004ec5f0 | FUN_004ec5f0 | pursuit_event_dispatch_pursuit_vehicle | C_sledovacka::AddCarHunt |
| 1728 | 0x004ec980 | FUN_004ec980 | pursuit_event_dispatch_escape_vehicle | C_sledovacka::AddCarEscape |
| 1729 | 0x004ed0b0 | FUN_004ed0b0 | pursuit_event_dispatch_move_to_vehicle | C_sledovacka::AddCarMoveTo |
| 1730 | 0x004edac0 | FUN_004edac0 | pursuit_event_remove_vehicle | C_sledovacka::RemoveCar |
| 1731 | 0x004edc20 | FUN_004edc20 | pursuit_event_recompile_path | C_sledovacka::RecompilePath |
| 1732 | 0x004ee740 | FUN_004ee740 | pursuit_event_prepare_path | C_sledovacka::PreparePath |
| 1733 | 0x004ee900 | FUN_004ee900 | pursuit_event_tick_driver | C_sledovacka::TickDriver |
| 1734 | 0x004efcf0 | FUN_004efcf0 | pursuit_event_tick | C_sledovacka::Tick |
| 1735 | 0x004f4470 | FUN_004f4470 | pursuit_event_record_get_action_status_by_vehicle | C_sledovacka::ActionStatus |
| 1736 | 0x004f44d0 | FUN_004f44d0 | pursuit_event_record_set_action_status_by_vehicle | C_sledovacka::SetActionStatus |
| 1737 | 0x004f4530 | FUN_004f4530 | pursuit_event_record_get_mode_by_vehicle | C_sledovacka::GetVoleMode |
| 1738 | 0x004f4590 | FUN_004f4590 | pursuit_event_drop_actor_references | C_sledovacka::VypicujMiToyotu |
| 1739 | 0x004f46c0 | FUN_004f46c0 | i3d_frame_get_world_matrix_ptr |  |
| 1740 | 0x004f46e0 | FUN_004f46e0 | pursuit_event_record_is_chase_tracking_by_vehicle | C_sledovacka::DevaaateeeZasNeznaCas |
| 1741 | 0x004f4760 | FUN_004f4760 | pursuit_event_save_game_get_size | C_sledovacka::SaveGameGetSize |
| 1742 | 0x004f4870 | FUN_004f4870 | pursuit_event_save_game_save | C_sledovacka::SaveGameSave |
| 1743 | 0x004f4dc0 | FUN_004f4dc0 | pursuit_event_save_game_load | C_sledovacka::SaveGameLoad |
| 1744 | 0x004f5570 | FUN_004f5570 | std_vector2_insert_one |  |
| 1745 | 0x004f5710 | FUN_004f5710 | std_vector2_size |  |
| 1746 | 0x004f5730 | FUN_004f5730 | std_vector2_erase_one |  |
| 1747 | 0x004f5770 | FUN_004f5770 | std_vector44_size |  |
| 1748 | 0x004f57a0 | FUN_004f57a0 | std_vector44_at |  |
| 1749 | 0x004f57c0 | FUN_004f57c0 | actor_is_death_processed |  |
| 1750 | 0x004f57d0 | FUN_004f57d0 | actor_is_in_camera_sector |  |
| 1751 | 0x004f57e0 | FUN_004f57e0 | driver_path_node_at |  |
| 1752 | 0x004f5800 | FUN_004f5800 | std_vector70_insert_one |  |
| 1753 | 0x004f5a30 | FUN_004f5a30 | vehicle_get_engine_on |  |
| 1754 | 0x004f5a40 | FUN_004f5a40 | vehicle_set_clutch_gate |  |
| 1755 | 0x004f5a50 | FUN_004f5a50 | vehicle_set_horn |  |
| 1756 | 0x004f5a60 | FUN_004f5a60 | vehicle_get_siren |  |
| 1757 | 0x004f5a70 | FUN_004f5a70 | vehicle_is_throttle_applied |  |
| 1758 | 0x004f5aa0 | FUN_004f5aa0 | vehicle_get_up_axis |  |
| 1759 | 0x004f5ab0 | FUN_004f5ab0 | vehicle_get_wheel_count |  |
| 1760 | 0x004f5ac0 | FUN_004f5ac0 | vehicle_get_wheel_at |  |
| 1761 | 0x004f5ad0 | FUN_004f5ad0 | car_set_condition_channel |  |
| 1762 | 0x004f5af0 | FUN_004f5af0 | std_vector2_insert_at |  |
| 1763 | 0x004f5d00 | FUN_004f5d00 | std_vector44_insert_one |  |
| 1764 | 0x004f5f10 | FUN_004f5f10 | std_vector70_erase_range |  |
| 1765 | 0x004f5f60 | FUN_004f5f60 | stl_copy_range_stride2 |  |
| 1766 | 0x004f5f90 | FUN_004f5f90 | stl_fill_n_stride2 |  |
| 1767 | 0x004f5fc0 | FUN_004f5fc0 | stl_copy_range_stride8 |  |
| 1768 | 0x004f5ff0 | FUN_004f5ff0 | stl_fill_n_stride8 |  |
| 1769 | 0x004f6020 | FUN_004f6020 | stl_fill_n_stride44 |  |
| 1770 | 0x004f6050 | FUN_004f6050 | array_copy_u16 |  |
| 1771 | 0x004f6070 | FUN_004f6070 | stl_copy_range_stride70 |  |
| 1772 | 0x004f60a0 | FUN_004f60a0 | stl_copy_one_stride2 |  |
| 1773 | 0x004f60b0 | FUN_004f60b0 | crt_static_init_guard_004f60b0 |  |
| 1774 | 0x004f60d0 | FUN_004f60d0 | crt_atexit_register_004f60d0 |  |
| 1775 | 0x004f60e0 | FUN_004f60e0 | smoke_trail_ctor | C_smoke_trail::{ctor} |
| 1776 | 0x004f6110 | FUN_004f6110 | smoke_trail_scalar_deleting_dtor |  |
| 1777 | 0x004f6130 | FUN_004f6130 | smoke_trail_dtor | C_smoke_trail::{dtor} |
| 1778 | 0x004f6140 | FUN_004f6140 | smoke_trail_ai_tick | C_smoke_trail::AI |
| 1779 | 0x004f61b0 | FUN_004f61b0 | smoke_trail_update | C_smoke_trail::Update |
| 1780 | 0x004f61d0 | thunk_FUN_00454920 | thunk_bottle_net_apply_remote_state |  |
| 1781 | 0x004f61e0 | FUN_004f61e0 | smoke_trail_game_init | C_smoke_trail::GameInit |
| 1782 | 0x004f6240 | FUN_004f6240 | smoke_trail_game_done | C_smoke_trail::GameDone |
| 1783 | 0x004f6270 | FUN_004f6270 | crt_static_init_guard_004f6270 |  |
| 1784 | 0x004f6290 | FUN_004f6290 | crt_atexit_register_004f6290 |  |
| 1785 | 0x004f62a0 | FUN_004f62a0 | crt_static_init_004f62a0 |  |
| 1786 | 0x004f62b0 | FUN_004f62b0 | crt_static_ctor_004f62b0 |  |
| 1787 | 0x004f62d0 | FUN_004f62d0 | crt_atexit_register_004f62d0 |  |
| 1788 | 0x004f62e0 | FUN_004f62e0 | crt_static_dtor_004f62e0 |  |
| 1789 | 0x004f6320 | FUN_004f6320 | crt_static_init_004f6320 |  |
| 1790 | 0x004f6330 | FUN_004f6330 | crt_static_ctor_004f6330 |  |
| 1791 | 0x004f6350 | FUN_004f6350 | crt_atexit_register_004f6350 |  |
| 1792 | 0x004f6360 | FUN_004f6360 | crt_static_dtor_004f6360 |  |
| 1793 | 0x004f63a0 | FUN_004f63a0 | crt_static_init_004f63a0 |  |
| 1794 | 0x004f63b0 | FUN_004f63b0 | crt_static_ctor_004f63b0 |  |
| 1795 | 0x004f63d0 | FUN_004f63d0 | crt_atexit_register_004f63d0 |  |
| 1796 | 0x004f63e0 | FUN_004f63e0 | crt_static_dtor_004f63e0 |  |
| 1797 | 0x004f6420 | FUN_004f6420 | crt_static_init_004f6420 |  |
| 1798 | 0x004f6430 | FUN_004f6430 | crt_static_ctor_004f6430 |  |
| 1799 | 0x004f6450 | FUN_004f6450 | crt_atexit_register_004f6450 |  |
| 1800 | 0x004f6460 | FUN_004f6460 | crt_static_dtor_004f6460 |  |
| 1801 | 0x004f64a0 | FUN_004f64a0 | crt_static_init_004f64a0 |  |
| 1802 | 0x004f64b0 | FUN_004f64b0 | crt_static_ctor_004f64b0 |  |
| 1803 | 0x004f64d0 | FUN_004f64d0 | crt_atexit_register_004f64d0 |  |
| 1804 | 0x004f64e0 | FUN_004f64e0 | crt_static_dtor_004f64e0 |  |
| 1805 | 0x004f6520 | FUN_004f6520 | crt_static_init_004f6520 |  |
| 1806 | 0x004f6530 | FUN_004f6530 | crt_static_ctor_004f6530 |  |
| 1807 | 0x004f6550 | FUN_004f6550 | crt_atexit_register_004f6550 |  |
| 1808 | 0x004f6560 | FUN_004f6560 | crt_static_dtor_004f6560 |  |
| 1809 | 0x004f65a0 | FUN_004f65a0 | traffic_car_ctor | C_traffic_car::{ctor} |
| 1810 | 0x004f6610 | FUN_004f6610 | traffic_car_scalar_deleting_dtor |  |
| 1811 | 0x004f6630 | FUN_004f6630 | traffic_car_dtor | C_traffic_car::{dtor} |
| 1812 | 0x004f6660 | FUN_004f6660 | traffic_car_init | C_traffic_car::Init |
| 1813 | 0x004f66b0 | FUN_004f66b0 | traffic_car_change_state | C_traffic_car::ChangeState |
| 1814 | 0x004f67e0 | FUN_004f67e0 | traffic_car_registry_set_enabled | C_traffic_car::Scipni |
| 1815 | 0x004f6900 | FUN_004f6900 | traffic_car_duplicate_from | C_traffic_car::DuplicateFrom |
| 1816 | 0x004f69c0 | FUN_004f69c0 | traffic_car_save_data | C_traffic_car::SaveData |
| 1817 | 0x004f6af0 | FUN_004f6af0 | traffic_car_load_data | C_traffic_car::LoadData |
| 1818 | 0x004f6ef0 | FUN_004f6ef0 | traffic_car_collision |  |
| 1819 | 0x004f6f00 | FUN_004f6f00 | traffic_car_deactivate_element | C_traffic_car::DeactivateElem |
| 1820 | 0x004f76c0 | FUN_004f76c0 | thunk_stub_noop_cdecl_00574a40 |  |
| 1821 | 0x004f76d0 | FUN_004f76d0 | traffic_car_activate_elements | C_traffic_car::ActivateElems |
| 1822 | 0x004faf30 | FUN_004faf30 | vec2_scale_to |  |
| 1823 | 0x004faf50 | FUN_004faf50 | vec2_add_in_place |  |
| 1824 | 0x004faf70 | FUN_004faf70 | vec2_sub_to |  |
| 1825 | 0x004faf90 | FUN_004faf90 | vec2_line_intersect |  |
| 1826 | 0x004fb000 | FUN_004fb000 | traffic_car_get_next_pos_info | C_traffic_car::GetNewPosInfo |
| 1827 | 0x004fb4e0 | FUN_004fb4e0 | traffic_car_find_last_crossing | C_traffic_car::FindLastCrossing |
| 1828 | 0x004fba80 | FUN_004fba80 | traffic_car_ai | C_traffic_car::AI |
| 1829 | 0x004feef0 | FUN_004feef0 | traffic_car_prepare_next_direction | C_traffic_car::PrepareNextDir |
| 1830 | 0x004ffc50 | FUN_004ffc50 | traffic_car_clear_lane_blocking | C_traffic_car::ClearBlocking |
| 1831 | 0x004ffce0 | FUN_004ffce0 | traffic_car_lock_crossing | C_traffic_car::LockCrossing |
| 1832 | 0x005010b0 | FUN_005010b0 | traffic_car_update | C_traffic_car::Update |
| 1833 | 0x005012e0 | FUN_005012e0 | traffic_car_game_init | C_traffic_car::GameInit |
| 1834 | 0x00503e40 | FUN_00503e40 | traffic_car_element_ctor |  |
| 1835 | 0x00503e50 | FUN_00503e50 | traffic_car_game_done | C_traffic_car::GameDone |
| 1836 | 0x005042c0 | FUN_005042c0 | traffic_car_release_stolen_car | C_traffic_car::Corovacka |
| 1837 | 0x00504380 | FUN_00504380 | traffic_car_get_parking_cars | C_traffic_car::GetParkingCars |
| 1838 | 0x00504390 | FUN_00504390 | traffic_car_seat_pedestrian_as_driver | C_traffic_car::ChodDoRici |
| 1839 | 0x00504780 | FUN_00504780 | traffic_car_activate_or_spawn_pursuit_vehicles |  |
| 1840 | 0x005052a0 | FUN_005052a0 | traffic_car_on_collision_try_register_assault_on_police_car | C_traffic_car::CoDelasTyCuraku |
| 1841 | 0x00505310 | FUN_00505310 | traffic_car_on_collision_try_register_assault_on_gang_car | C_traffic_car::CoDelasTyCuraku1 |
| 1842 | 0x005053a0 | FUN_005053a0 | traffic_car_release_cop_car_slot | C_traffic_car::ChlupatejVystrik |
| 1843 | 0x005053c0 | FUN_005053c0 | traffic_car_release_mafia_car_slot | C_traffic_car::MafianVystrik |
| 1844 | 0x005053e0 | FUN_005053e0 | traffic_car_spawn_mafia_pursuit_car | C_traffic_car::Krles |
| 1845 | 0x00506130 | FUN_00506130 | traffic_car_take_over_cruising_car | C_traffic_car::SebraliMiKaaaruSvine |
| 1846 | 0x00506760 | FUN_00506760 | traffic_car_get_bridge_car_count | C_traffic_car::GetBridgeCarCount |
| 1847 | 0x005067f0 | FUN_005067f0 | traffic_car_is_possible_coll | C_traffic_car::IsPossibleColl |
| 1848 | 0x00506900 | FUN_00506900 | traffic_car_hit | C_traffic_car::Hit |
| 1849 | 0x00506b60 | FUN_00506b60 | traffic_car_find_car_heading_at_point | C_traffic_car::ChcesMeSejmout |
| 1850 | 0x00506bb0 | FUN_00506bb0 | traffic_car_deactivate_adopted_elements | C_traffic_car::TakATedSiVasKupim_Vsecky |
| 1851 | 0x00506c00 | FUN_00506c00 | traffic_car_kill_occupant_on_hit | C_traffic_car::DostalJsemRanuKachlou |
| 1852 | 0x00506cf0 | FUN_00506cf0 | traffic_car_adopt_car_with_new_driver |  |
| 1853 | 0x00507cc0 | FUN_00507cc0 | traffic_car_destroy_crew_for_car |  |
| 1854 | 0x00507f80 | FUN_00507f80 | traffic_car_save_game_get_size | C_traffic_car::SaveGameGetSize |
| 1855 | 0x00507f90 | FUN_00507f90 | traffic_car_save_game_save | C_traffic_car::SaveGameSave |
| 1856 | 0x00507fc0 | FUN_00507fc0 | traffic_car_save_game_load | C_traffic_car::SaveGameLoad |
| 1857 | 0x00507ff0 | FUN_00507ff0 | vec_count_stride_0xec |  |
| 1858 | 0x00508020 | FUN_00508020 | vec_count_stride_0x22 |  |
| 1859 | 0x00508050 | FUN_00508050 | vec_erase_at_stride_0x22 |  |
| 1860 | 0x005080b0 | FUN_005080b0 | vec_clear_keep_capacity |  |
| 1861 | 0x005080f0 | FUN_005080f0 | stl_copy_range_stride_0x22 |  |
| 1862 | 0x00508120 | FUN_00508120 | crt_static_init_guard_00508120 |  |
| 1863 | 0x00508140 | FUN_00508140 | crt_atexit_register_00508140 |  |
| 1864 | 0x00508150 | FUN_00508150 | railway_switch_ctor | C_turnout::{ctor} |
| 1865 | 0x00508260 | FUN_00508260 | railway_switch_scalar_deleting_dtor |  |
| 1866 | 0x00508280 | FUN_00508280 | railway_switch_dtor | C_turnout::{dtor} |
| 1867 | 0x00508290 | FUN_00508290 | railway_switch_save_data | C_turnout::SaveData |
| 1868 | 0x005083f0 | FUN_005083f0 | railway_switch_load_data | C_turnout::LoadData |
| 1869 | 0x00508620 | FUN_00508620 | railway_switch_duplicate_from | C_turnout::DuplicateFrom |
| 1870 | 0x005086b0 | FUN_005086b0 | railway_switch_game_init | C_turnout::GameInit |
| 1871 | 0x00508b30 | FUN_00508b30 | railway_switch_game_done | C_turnout::GameDone |
| 1872 | 0x00508c00 | FUN_00508c00 | railway_switch_ai | C_turnout::AI |
| 1873 | 0x00508dd0 | FUN_00508dd0 | railway_switch_update | C_turnout::Update |
| 1874 | 0x00509000 | FUN_00509000 | railway_switch_start_switching | C_turnout::Start |
| 1875 | 0x00509050 | FUN_00509050 | railway_switch_set_coll_pos | C_turnout::SetCollPos |
| 1876 | 0x00509170 | FUN_00509170 | railway_switch_get_web_nodes | C_turnout::NodesOK |
| 1877 | 0x00509200 | FUN_00509200 | railway_switch_save_game_get_size | C_turnout::SaveGameGetSize |
| 1878 | 0x00509210 | FUN_00509210 | railway_switch_save_game_save | C_turnout::SaveGameSave |
| 1879 | 0x00509250 | FUN_00509250 | railway_switch_save_game_load | C_turnout::SaveGameLoad |
| 1880 | 0x00509400 | FUN_00509400 | crt_static_init_guard_00509400 |  |
| 1881 | 0x00509420 | FUN_00509420 | crt_atexit_register_00509420 |  |
| 1882 | 0x00509430 | thunk_FUN_00509440 | thunk_stub_noop_cdecl_00509440 |  |
| 1883 | 0x00509440 | FUN_00509440 | stub_noop_cdecl_00509440 |  |
| 1884 | 0x00509450 | FUN_00509450 | crt_static_init_00509450 |  |
| 1885 | 0x00509460 | FUN_00509460 | vehicle_tyre_list_static_ctor |  |
| 1886 | 0x00509480 | FUN_00509480 | crt_atexit_register_00509480 |  |
| 1887 | 0x00509490 | FUN_00509490 | vehicle_tyre_list_static_dtor |  |
| 1888 | 0x005094d0 | FUN_005094d0 | crt_static_init_005094d0 |  |
| 1889 | 0x005094e0 | FUN_005094e0 | vehicle_material_list_static_ctor |  |
| 1890 | 0x00509500 | FUN_00509500 | crt_atexit_register_00509500 |  |
| 1891 | 0x00509510 | FUN_00509510 | vehicle_material_list_static_dtor |  |
| 1892 | 0x00509550 | FUN_00509550 | crt_static_init_00509550 |  |
| 1893 | 0x00509560 | FUN_00509560 | vehicle_particle_list_static_ctor |  |
| 1894 | 0x00509580 | FUN_00509580 | crt_atexit_register_00509580 |  |
| 1895 | 0x00509590 | FUN_00509590 | vehicle_particle_list_static_dtor |  |
| 1896 | 0x005095d0 | FUN_005095d0 | crt_static_init_005095d0 |  |
| 1897 | 0x005095e0 | FUN_005095e0 | vehicle_projector_sector_list_static_ctor |  |
| 1898 | 0x00509600 | FUN_00509600 | crt_atexit_register_00509600 |  |
| 1899 | 0x00509610 | FUN_00509610 | vehicle_projector_sector_list_static_dtor |  |
| 1900 | 0x00509650 | FUN_00509650 | crt_static_init_00509650 |  |
| 1901 | 0x00509660 | FUN_00509660 | vehicle_sound_sector_list_static_ctor |  |
| 1902 | 0x00509680 | FUN_00509680 | crt_atexit_register_00509680 |  |
| 1903 | 0x00509690 | FUN_00509690 | vehicle_sound_sector_list_static_dtor |  |
| 1904 | 0x005096d0 | FUN_005096d0 | vehicle_ctor | C_Vehicle::{ctor} |
| 1905 | 0x00509bb0 | FUN_00509bb0 | vehicle_init | C_Vehicle::Init |
| 1906 | 0x0050b6f0 | FUN_0050b6f0 | vehicle_init_strip_and_smoke_emitters |  |
| 1907 | 0x0050cc10 | FUN_0050cc10 | vehicle_init_axes | C_Vehicle::InitAxis |
| 1908 | 0x0050d0c0 | FUN_0050d0c0 | vehicle_init_lights | C_Vehicle::InitLights |
| 1909 | 0x0050db60 | FUN_0050db60 | vehicle_dtor | C_Vehicle::{dtor} |
| 1910 | 0x0050e0c0 | FUN_0050e0c0 | vehicle_create_light_projector | C_Vehicle::CreateLightProjector |
| 1911 | 0x0050e670 | FUN_0050e670 | vehicle_create_special_light_projector | C_Vehicle::CreateSpecialLightProjector |
| 1912 | 0x0050ee60 | FUN_0050ee60 | vehicle_deactivate | C_Vehicle::Deactivate |
| 1913 | 0x0050f140 | FUN_0050f140 | vehicle_reset_from_frame | C_Vehicle::Reset1 |
| 1914 | 0x0050f7f0 | FUN_0050f7f0 | vehicle_reset | C_Vehicle::Reset |
| 1915 | 0x00510150 | FUN_00510150 | vehicle_game_init | C_Vehicle::GameInit |
| 1916 | 0x00511890 | FUN_00511890 | vehicle_game_done | C_Vehicle::GameDone |
| 1917 | 0x00511b50 | FUN_00511b50 | vehicle_set_tyres | C_Vehicle::SetTyres |
| 1918 | 0x00511eb0 | FUN_00511eb0 | vehicle_close_tyres | C_Vehicle::CloseTyres |
| 1919 | 0x00511f40 | FUN_00511f40 | vehicle_set_materials | C_Vehicle::SetVehicleMaterials |
| 1920 | 0x00511fb0 | FUN_00511fb0 | vehicle_close_materials | C_Vehicle::CloseVehicleMaterials |
| 1921 | 0x00511ff0 | FUN_00511ff0 | vehicle_set_particles | C_Vehicle::AddParticle |
| 1922 | 0x005122b0 | FUN_005122b0 | vehicle_close_particles | C_Vehicle::CloseParticles |
| 1923 | 0x00512340 | FUN_00512340 | vehicle_get_wheel_cam_pos | C_Vehicle::GetWheelCamPos |
| 1924 | 0x005123d0 | FUN_005123d0 | vehicle_apply_carinit_params | C_Vehicle::InitCar |
| 1925 | 0x00517370 | FUN_00517370 | vehicle_get_carinit_params | C_Vehicle::GetInit |
| 1926 | 0x005173a0 | FUN_005173a0 | vehicle_set_gear | C_Vehicle::SetGear |
| 1927 | 0x00517460 | FUN_00517460 | vehicle_set_power | C_Vehicle::SetPower |
| 1928 | 0x00517600 | FUN_00517600 | vehicle_set_brake | C_Vehicle::SetBrake |
| 1929 | 0x00517790 | FUN_00517790 | vehicle_set_clutch | C_Vehicle::SetClutch |
| 1930 | 0x00517800 | FUN_00517800 | vehicle_set_steer | C_Vehicle::SetSteer |
| 1931 | 0x005178a0 | FUN_005178a0 | vehicle_set_steering_linearity | C_Vehicle::SetSteeringLinearity |
| 1932 | 0x005178e0 | FUN_005178e0 | vehicle_set_engine_on | C_Vehicle::SetEngineOn |
| 1933 | 0x005179d0 | FUN_005179d0 | vehicle_set_speed_limit | C_Vehicle::SetSpeedLimit |
| 1934 | 0x00517a10 | FUN_00517a10 | vehicle_set_fuel | C_Vehicle::SetFuel |
| 1935 | 0x00517a40 | FUN_00517a40 | vehicle_set_handbrake | C_Vehicle::SetHandbrake |
| 1936 | 0x00517ab0 | FUN_00517ab0 | vehicle_enable_sounds | C_Vehicle::EnableSounds |
| 1937 | 0x00517ae0 | FUN_00517ae0 | vehicle_set_cut_time | C_Vehicle::SetCutTime |
| 1938 | 0x00517b10 | FUN_00517b10 | vehicle_apply_external_impulse | C_Vehicle::ExternalImpuls |
| 1939 | 0x00517f00 | FUN_00517f00 | vehicle_apply_external_force | C_Vehicle::ExternalForce |
| 1940 | 0x005180a0 | FUN_005180a0 | vehicle_do_collision | C_Vehicle::DoCollision |
| 1941 | 0x00518500 | FUN_00518500 | vehicle_get_contact_impulse_terms | C_Vehicle::GetRVelMassJ |
| 1942 | 0x00518670 | FUN_00518670 | vehicle_get_head_pos | C_Vehicle::GetHeadPos |
| 1943 | 0x00518680 | FUN_00518680 | vehicle_get_max_engine_torque_at_rpm | C_Vehicle::_GetMaxMomentByRot |
| 1944 | 0x00518730 | FUN_00518730 | vehicle_set_seat_mass | C_Vehicle::SetSeatMass |
| 1945 | 0x00518a90 | FUN_00518a90 | vehicle_encode_position | C_Vehicle::CodePos |
| 1946 | 0x00518b40 | FUN_00518b40 | vehicle_decode_position | C_Vehicle::DecodePos |
| 1947 | 0x00518c10 | FUN_00518c10 | vehicle_get_part_name_tag | C_Vehicle::CodeIndexDan |
| 1948 | 0x00518d10 | FUN_00518d10 | vehicle_find_part_by_name_tag | C_Vehicle::DecodeIndexDan |
| 1949 | 0x00518d70 | FUN_00518d70 | vehicle_encode_part_position |  |
| 1950 | 0x00518f70 | FUN_00518f70 | vehicle_find_part_at_encoded_position |  |
| 1951 | 0x005195e0 | FUN_005195e0 | vehicle_get_engine_rpm | C_Vehicle::GetRotations |
| 1952 | 0x00519600 | FUN_00519600 | vehicle_is_unstable | C_Vehicle::IsVehicleUnstable |
| 1953 | 0x00519610 | FUN_00519610 | vehicle_get_size | C_Vehicle::GetVehicleSize |
| 1954 | 0x00519630 | FUN_00519630 | vehicle_get_pivot_pos | C_Vehicle::GetVehiclePivotPos |
| 1955 | 0x00519650 | FUN_00519650 | vehicle_lock | C_Vehicle::LockVehicle |
| 1956 | 0x00519690 | FUN_00519690 | vehicle_set_projector_size | C_Vehicle::SetProjectorSize |
| 1957 | 0x00519a70 | FUN_00519a70 | vehicle_open_door | C_Vehicle::OpenDoor |
| 1958 | 0x00519b10 | FUN_00519b10 | vehicle_set_max_door_angle | C_Vehicle::SetMaxDoorAngle |
| 1959 | 0x00519b30 | FUN_00519b30 | vehicle_set_door_open_speed | C_Vehicle::SetSpeedDoorOpen |
| 1960 | 0x00519b60 | FUN_00519b60 | vehicle_set_special_dynamic_colls | C_Vehicle::SetSpecialDynamicColls |
| 1961 | 0x00519d30 | FUN_00519d30 | vehicle_set_dynamic_colls_off | C_Vehicle::SetDynamicCollsOn |
| 1962 | 0x00519e50 | FUN_00519e50 | vehicle_update_cached_transform | C_Vehicle::UpdateImportantVariables |
| 1963 | 0x0051a150 | FUN_0051a150 | vehicle_clear_body_coll_add_track | C_Vehicle::ClearBodyCollAddTrack |
| 1964 | 0x0051a250 | FUN_0051a250 | vehicle_add_projector_lights_to_sectors | C_Vehicle::AddProjectorLightsToSectors |
| 1965 | 0x0051a3b0 | FUN_0051a3b0 | vehicle_remove_projector_lights_from_sectors | C_Vehicle::DeleteProjectorLightsFromSectors |
| 1966 | 0x0051a520 | FUN_0051a520 | vehicle_register_projector_light_sector | C_Vehicle::AddSectorForProjectorLights |
| 1967 | 0x0051a720 | FUN_0051a720 | vehicle_register_sound_sector | C_Vehicle::AddSectorForSounds |
| 1968 | 0x0051a920 | FUN_0051a920 | vehicle_settle | C_Vehicle::CalmVehicle |
| 1969 | 0x0051aac0 | FUN_0051aac0 | vehicle_get_save_state_size | C_Vehicle::GetVehicleStateSize |
| 1970 | 0x0051ad90 | FUN_0051ad90 | vehicle_save_state | C_Vehicle::SaveVehicleState |
| 1971 | 0x0051ba40 | FUN_0051ba40 | vehicle_load_state | C_Vehicle::LoadVehicleState |
| 1972 | 0x0051ccf0 | FUN_0051ccf0 | vehicle_cb_enum_objects | CbEnumObj |
| 1973 | 0x0051cdc0 | FUN_0051cdc0 | vehicle_cb_enum_first_object | CbEnumObj1 |
| 1974 | 0x0051cdd0 | FUN_0051cdd0 | vehicle_cb_enum_object_list | CbEnumObj6 |
| 1975 | 0x0051cf80 | FUN_0051cf80 | vehicle_cb_enum_body_colls | CbEnumBodyColl |
| 1976 | 0x0051d280 | FUN_0051d280 | vehicle_body_coll_attach_to_parent | IsParent |
| 1977 | 0x0051d330 | FUN_0051d330 | vehicle_colbody_head_dtor |  |
| 1978 | 0x0051d3b0 | FUN_0051d3b0 | vehicle_collect_coll_spheres | EnumColSphere |
| 1979 | 0x0051d440 | FUN_0051d440 | vehicle_cb_enum_coll_sphere | CbEnumCollsActiveSphere |
| 1980 | 0x0051d710 | FUN_0051d710 | vehicle_prepare_body_colls | C_Vehicle::PrepareBodyColls |
| 1981 | 0x0051d9a0 | FUN_0051d9a0 | vehicle_cb_enum_deform_zones | CbEnumDeform |
| 1982 | 0x0051ddf0 | FUN_0051ddf0 | vehicle_set_callback_collision_info | C_Vehicle::SetCallbackACI |
| 1983 | 0x0051de00 | FUN_0051de00 | vehicle_set_callback_collision | C_Vehicle::SetCallbackCC |
| 1984 | 0x0051de10 | FUN_0051de10 | vehicle_set_callback_collision_quiet | C_Vehicle::SetCallbackCC1 |
| 1985 | 0x0051de20 | FUN_0051de20 | vehicle_set_callback_actor_filter | C_Vehicle::SetCallbackAF |
| 1986 | 0x0051de30 | FUN_0051de30 | vehicle_set_callback_was_collision | C_Vehicle::SetCallbackWC |
| 1987 | 0x0051de40 | FUN_0051de40 | vehicle_set_callback_prepare_dropout | C_Vehicle::SetCallbackPD |
| 1988 | 0x0051de50 | FUN_0051de50 | vehicle_set_callback_prepare_wheel_dropout | C_Vehicle::SetCallbackPDW |
| 1989 | 0x0051de60 | FUN_0051de60 | vehicle_set_callback_break_glass | C_Vehicle::SetCallbackBG |
| 1990 | 0x0051de70 | FUN_0051de70 | vehicle_set_callback_wake_up | C_Vehicle::SetCallbackWU |
| 1991 | 0x0051de80 | FUN_0051de80 | vehicle_set_callback_body_collision_filter | C_Vehicle::SetCallbackCFB |
| 1992 | 0x0051de90 | FUN_0051de90 | vehicle_set_callback_wheel_collision_filter | C_Vehicle::SetCallbackCFW |
| 1993 | 0x0051dea0 | FUN_0051dea0 | vehicle_set_callback_deform_part | C_Vehicle::SetCallbackDVP |
| 1994 | 0x0051deb0 | FUN_0051deb0 | vehicle_set_callback_repair | C_Vehicle::SetCallbackVR |
| 1995 | 0x0051dec0 | FUN_0051dec0 | vehicle_colbody_head_vector_destroy |  |
| 1996 | 0x0051df00 | FUN_0051df00 | std_vector18_insert_at |  |
| 1997 | 0x0051e100 | FUN_0051e100 | vehicle_window_list_insert_at |  |
| 1998 | 0x0051e310 | FUN_0051e310 | vehicle_window_list_size |  |
| 1999 | 0x0051e340 | FUN_0051e340 | vehicle_colbody_head_vector_insert_n |  |
| 2000 | 0x0051e6d0 | FUN_0051e6d0 | std_vector20_insert_n |  |
| 2001 | 0x0051e970 | FUN_0051e970 | std_vector24_insert_n |  |
| 2002 | 0x0051ec70 | FUN_0051ec70 | std_vector1c_insert_n |  |
| 2003 | 0x0051ef90 | FUN_0051ef90 | vehicle_special_coll_list_insert_n |  |
| 2004 | 0x0051f290 | FUN_0051f290 | std_vector34_uninit_copy |  |
| 2005 | 0x0051f2d0 | FUN_0051f2d0 | std_vector34_uninit_fill_n |  |
| 2006 | 0x0051f300 | FUN_0051f300 | std_vector1c_copy_range |  |
| 2007 | 0x0051f330 | FUN_0051f330 | vehicle_special_coll_list_copy_range |  |
| 2008 | 0x0051f360 | FUN_0051f360 | vehicle_colbody_head_copy |  |
| 2009 | 0x0051f520 | FUN_0051f520 | std_vector20_construct_copy |  |
| 2010 | 0x0051f540 | FUN_0051f540 | vehicle_deform_zone_copy |  |
| 2011 | 0x0051f560 | FUN_0051f560 | vehicle_colbody_sphere_vector_copy_assign |  |
| 2012 | 0x0051f7a0 | FUN_0051f7a0 | vehicle_colbody_head_scalar_deleting_dtor |  |
| 2013 | 0x0051f7c0 | FUN_0051f7c0 | vehicle_coll_sphere_list_copy_assign |  |
| 2014 | 0x0051fa20 | FUN_0051fa20 | vehicle_colbody_head_vector_copy_assign |  |
| 2015 | 0x0051fca0 | FUN_0051fca0 | vehicle_coll_sphere_list_copy_range |  |
| 2016 | 0x0051fcd0 | FUN_0051fcd0 | crt_static_init_guard_0051fcd0 |  |
| 2017 | 0x0051fcf0 | FUN_0051fcf0 | crt_atexit_register_0051fcf0 |  |
| 2018 | 0x0051fd00 | FUN_0051fd00 | vehicle_emit_wheel_skid_segment | C_Vehicle::Slaif |
| 2019 | 0x005214e0 | FUN_005214e0 | vehicle_end_wheel_skid_segment | C_Vehicle::SlaifEnd |
| 2020 | 0x00521580 | FUN_00521580 | vehicle_prepare_deform_zone | C_Vehicle::PrepareDeformZone |
| 2021 | 0x005219a0 | FUN_005219a0 | vehicle_init_deform_glass_texture | C_Vehicle::InitDeformGlassTexture |
| 2022 | 0x00521a40 | FUN_00521a40 | vehicle_init_deform | C_Vehicle::InitDeform |
| 2023 | 0x00521d30 | FUN_00521d30 | vehicle_init_damage | C_Vehicle::InitDamage |
| 2024 | 0x00521e60 | FUN_00521e60 | vehicle_deform | C_Vehicle::Deform |
| 2025 | 0x00522dd0 | FUN_00522dd0 | vehicle_switch_special_projector_texture | C_Vehicle::SwitchSpecialProjectorTexture |
| 2026 | 0x00522ec0 | FUN_00522ec0 | vehicle_deform_glass | C_Vehicle::DeformGlass |
| 2027 | 0x00522f70 | FUN_00522f70 | vehicle_apply_damage | C_Vehicle::DamageVehicle |
| 2028 | 0x00523090 | FUN_00523090 | vehicle_update_axis | C_Vehicle::UpdateAxis |
| 2029 | 0x00523fa0 | FUN_00523fa0 | vehicle_animate_doors | C_Vehicle::DoDoors |
| 2030 | 0x005242d0 | FUN_005242d0 | vehicle_open_window | C_Vehicle::OpenWindow |
| 2031 | 0x00524320 | FUN_00524320 | vehicle_animate_windows | C_Vehicle::DoWindows |
| 2032 | 0x005244d0 | FUN_005244d0 | vehicle_is_window_open | C_Vehicle::IsWindowOpen |
| 2033 | 0x00524520 | FUN_00524520 | vehicle_apply_light_state | C_Vehicle::DoLights |
| 2034 | 0x005289c0 | FUN_005289c0 | i3d_frame_get_local_direction_normalized |  |
| 2035 | 0x00528b70 | FUN_00528b70 | i3d_frame_get_parent_ptr |  |
| 2036 | 0x00528b80 | FUN_00528b80 | vehicle_update_strips | C_Vehicle::UpdateStrips |
| 2037 | 0x0052a1b0 | FUN_0052a1b0 | vehicle_update_motor_particles | C_Vehicle::UpdateMotorParticles |
| 2038 | 0x0052a670 | FUN_0052a670 | vehicle_update_steering_wheels | C_Vehicle::UpdateSteeringWheels |
| 2039 | 0x0052aa30 | FUN_0052aa30 | vehicle_update_rotation_wheels | C_Vehicle::UpdateRotationWheels |
| 2040 | 0x0052af10 | FUN_0052af10 | vehicle_update_wheel_particles | C_Vehicle::UpdateWheelParticles |
| 2041 | 0x0052b8a0 | FUN_0052b8a0 | vehicle_update_exhaust_particles | C_Vehicle::UpdateExhaustParticles |
| 2042 | 0x0052bf20 | FUN_0052bf20 | vehicle_emit_body_collision_particles | C_Vehicle::UpdateBodyParticles |
| 2043 | 0x0052c350 | FUN_0052c350 | vehicle_light_at |  |
| 2044 | 0x0052c360 | FUN_0052c360 | vehicle_special_projector_at |  |
| 2045 | 0x0052c380 | FUN_0052c380 | crt_static_init_guard_0052c380 |  |
| 2046 | 0x0052c3a0 | FUN_0052c3a0 | crt_atexit_register_0052c3a0 |  |
| 2047 | 0x0052c3b0 | FUN_0052c3b0 | vehicle_tick_drivetrain | C_Vehicle::Tick |
| 2048 | 0x0052d480 | FUN_0052d480 | vehicle_update_visuals | C_Vehicle::Update |
| 2049 | 0x0052dfa0 | FUN_0052dfa0 | vehicle_compute_motor_force | C_Vehicle::MotorForce |
| 2050 | 0x0052e220 | FUN_0052e220 | vehicle_compute_moment | C_Vehicle::ComputeMoment |
| 2051 | 0x0052e3c0 | FUN_0052e3c0 | vehicle_update_motor_revs | C_Vehicle::MotorRot |
| 2052 | 0x0052e6d0 | FUN_0052e6d0 | vehicle_substep_move | C_Vehicle::Engine |
| 2053 | 0x0052ea80 | FUN_0052ea80 | vehicle_move | C_Vehicle::Move |
| 2054 | 0x005334e0 | FUN_005334e0 | vehicle_wheel_contact_solve | C_Vehicle::DoWheelsCollision |
| 2055 | 0x00535610 | FUN_00535610 | vehicle_body_coll_resolve | C_Vehicle::BodyCollision |
| 2056 | 0x00537300 | FUN_00537300 | vehicle_body_coll_sphere_sort_list_static_dtor |  |
| 2057 | 0x00537320 | FUN_00537320 | vehicle_body_coll_head_sort_list_static_dtor |  |
| 2058 | 0x00537340 | FUN_00537340 | std_vector74_size |  |
| 2059 | 0x00537370 | FUN_00537370 | std_vector8_erase_range |  |
| 2060 | 0x005373b0 | FUN_005373b0 | crt_static_init_guard_005373b0 |  |
| 2061 | 0x005373d0 | FUN_005373d0 | crt_atexit_register_005373d0 |  |
| 2062 | 0x005373e0 | FUN_005373e0 | vehicle_init_sounds | C_Vehicle::InitSounds |
| 2063 | 0x00537880 | FUN_00537880 | vehicle_add_sounds_to_sectors | C_Vehicle::AddSoundsToSectors |
| 2064 | 0x00537d50 | FUN_00537d50 | vehicle_remove_sounds_from_sectors | C_Vehicle::DeleteSoundsFromSectors |
| 2065 | 0x00538220 | FUN_00538220 | vehicle_update_engine_sound | C_Vehicle::EngineSnd |
| 2066 | 0x005385b0 | FUN_005385b0 | vehicle_update_engine_pitch | C_Vehicle::EngineFreq |
| 2067 | 0x00539550 | FUN_00539550 | vehicle_update_environment_sound | C_Vehicle::EnvironmentalSnd |
| 2068 | 0x005398d0 | FUN_005398d0 | vehicle_update_flat_tyre_sound | C_Vehicle::CrashTyreSnd |
| 2069 | 0x00539c60 | FUN_00539c60 | vehicle_update_skid_sound | C_Vehicle::SkidSnd |
| 2070 | 0x00539f80 | FUN_00539f80 | vehicle_play_gear_change_sound | C_Vehicle::GearSnd |
| 2071 | 0x0053a080 | FUN_0053a080 | vehicle_play_handbrake_sound | C_Vehicle::HandBrake |
| 2072 | 0x0053a120 | FUN_0053a120 | vehicle_play_engine_start_fail_sound | C_Vehicle::EngineSndNFS |
| 2073 | 0x0053a1b0 | FUN_0053a1b0 | vehicle_play_horn_sound | C_Vehicle::HornSnd |
| 2074 | 0x0053a280 | FUN_0053a280 | vehicle_play_siren_sound | C_Vehicle::SirenSnd |
| 2075 | 0x0053a350 | FUN_0053a350 | vehicle_play_tyre_burst_sound | C_Vehicle::TyreBreakSnd |
| 2076 | 0x0053a410 | FUN_0053a410 | vehicle_play_crash_sound | C_Vehicle::CrashSnd |
| 2077 | 0x0053b0c0 | FUN_0053b0c0 | vehicle_play_wheel_sound | C_Vehicle::WheelSnd |
| 2078 | 0x0053b120 | FUN_0053b120 | vehicle_play_door_sound | C_Vehicle::DoorSnd |
| 2079 | 0x0053b2c0 | FUN_0053b2c0 | vehicle_stop_motor_sounds | C_Vehicle::MotorSoundOff |
| 2080 | 0x0053b3f0 | FUN_0053b3f0 | vehicle_stop_all_sounds | C_Vehicle::SoundOff |
| 2081 | 0x0053b840 | FUN_0053b840 | vehicle_set_global_sound_volume |  |
| 2082 | 0x0053b850 | FUN_0053b850 | vehicle_open_sound | C_Vehicle::OpenSnd |
| 2083 | 0x0053b880 | FUN_0053b880 | vehicle_sound_slot_init | C_Vehicle::SoundInit |
| 2084 | 0x0053bbf0 | FUN_0053bbf0 | vehicle_release_sounds | C_Vehicle::ReleaseSounds |
| 2085 | 0x0053c1d0 | FUN_0053c1d0 | string_ensure_buffer |  |
| 2086 | 0x0053c280 | FUN_0053c280 | crt_static_init_guard_0053c280 |  |
| 2087 | 0x0053c2a0 | FUN_0053c2a0 | crt_atexit_register_0053c2a0 |  |
| 2088 | 0x0053c2b0 | FUN_0053c2b0 | wagon_ctor | C_wagon::{ctor} |
| 2089 | 0x0053c360 | FUN_0053c360 | wagon_scalar_deleting_dtor |  |
| 2090 | 0x0053c380 | FUN_0053c380 | wagon_dtor | C_wagon::{dtor} |
| 2091 | 0x0053c400 | FUN_0053c400 | wagon_init | C_wagon::Init |
| 2092 | 0x0053c420 | FUN_0053c420 | wagon_save_game_get_size | C_wagon::SaveGameGetSize |
| 2093 | 0x0053c470 | FUN_0053c470 | wagon_save_game_save | C_wagon::SaveGameSave |
| 2094 | 0x0053c630 | FUN_0053c630 | wagon_save_game_load | C_wagon::SaveGameLoad |
| 2095 | 0x0053ca50 | FUN_0053ca50 | wagon_extend_back_node_path | C_wagon::CreateBackVector |
| 2096 | 0x0053cc40 | FUN_0053cc40 | wagon_build_forward_node_path | C_wagon::CreateForwardVector |
| 2097 | 0x0053d040 | FUN_0053d040 | wagon_node_is_stop_point | C_wagon::IsStopPoint |
| 2098 | 0x0053d070 | FUN_0053d070 | wagon_distance_to_next_stop | C_wagon::MustStop |
| 2099 | 0x0053d2d0 | FUN_0053d2d0 | wagon_find_point_on_back_path | C_wagon::FindBackPos |
| 2100 | 0x0053d890 | FUN_0053d890 | wagon_apply_spring_sway | C_wagon::SpringProcess |
| 2101 | 0x0053dca0 | FUN_0053dca0 | wagon_ai | C_wagon::AI |
| 2102 | 0x0053f360 | FUN_0053f360 | wagon_update | C_wagon::Update |
| 2103 | 0x0053f7d0 | FUN_0053f7d0 | wagon_game_init | C_wagon::GameInit |
| 2104 | 0x00540590 | FUN_00540590 | wagon_cb_collect_wheel_frames | C_wagon::S_hlp$3866C_wagon_cpp::cbEnumFrame |
| 2105 | 0x00540620 | FUN_00540620 | wagon_game_done | C_wagon::GameDone |
| 2106 | 0x00540770 | FUN_00540770 | wagon_rebuild_all_forward_paths | C_wagon::RecompileNewPath |
| 2107 | 0x005407f0 | FUN_005407f0 | crt_static_init_guard_005407f0 |  |
| 2108 | 0x00540810 | FUN_00540810 | crt_atexit_register_00540810 |  |
| 2109 | 0x00540820 | thunk_FUN_00540830 | thunk_recorder_static_zero_globals |  |
| 2110 | 0x00540830 | FUN_00540830 | recorder_static_zero_globals |  |
| 2111 | 0x00540850 | FUN_00540850 | curve_ease_in_out |  |
| 2112 | 0x00540920 | FUN_00540920 | recorder_interp_key_pos_and_pair |  |
| 2113 | 0x00540ac0 | FUN_00540ac0 | recorder_interp_key_pos |  |
| 2114 | 0x00540c30 | FUN_00540c30 | recorder_new_track |  |
| 2115 | 0x00540db0 | FUN_00540db0 | recorder_track_reset_fields |  |
| 2116 | 0x00540de0 | FUN_00540de0 | recorder_set_mode |  |
| 2117 | 0x00540fb0 | FUN_00540fb0 | recorder_get_mode |  |
| 2118 | 0x00540fc0 | FUN_00540fc0 | recorder_sync_clock_from_system |  |
| 2119 | 0x005411d0 | FUN_005411d0 | recorder_get_source_clock_ms |  |
| 2120 | 0x005411e0 | FUN_005411e0 | recorder_get_playback_clock_ms |  |
| 2121 | 0x005411f0 | FUN_005411f0 | recorder_add_track |  |
| 2122 | 0x005413f0 | FUN_005413f0 | recorder_attach_existing_track |  |
| 2123 | 0x00541450 | FUN_00541450 | recorder_remove_track |  |
| 2124 | 0x00541520 | FUN_00541520 | recorder_release_all_key_streams |  |
| 2125 | 0x00541620 | FUN_00541620 | recorder_has_track |  |
| 2126 | 0x00541680 | FUN_00541680 | recorder_set_track_active |  |
| 2127 | 0x005417b0 | FUN_005417b0 | recorder_has_track_in_either_list |  |
| 2128 | 0x00541810 | FUN_00541810 | recorder_snapshot_track_list |  |
| 2129 | 0x00541a30 | FUN_00541a30 | recorder_track_snapshot_static_dtor |  |
| 2130 | 0x00541a50 | FUN_00541a50 | recorder_tick_tracks |  |
| 2131 | 0x00542670 | FUN_00542670 | recorder_track_rewind_cursors |  |
| 2132 | 0x005426d0 | FUN_005426d0 | recorder_track_step_keys_to_clock |  |
| 2133 | 0x00542790 | FUN_00542790 | recorder_track_stop_and_rewind |  |
| 2134 | 0x005427b0 | FUN_005427b0 | recorder_track_seek_to_clock |  |
| 2135 | 0x005427f0 | FUN_005427f0 | recorder_load_replay_file |  |
| 2136 | 0x00543040 | FUN_00543040 | recorder_save_replay_file |  |
| 2137 | 0x00543454 | FUN_00543454 | recorder_write_replay_blocks |  |
| 2138 | 0x00543540 | FUN_00543540 | recorder_advance_clock |  |
| 2139 | 0x00543620 | FUN_00543620 | recorder_binding_block_size |  |
| 2140 | 0x005436b0 | FUN_005436b0 | recorder_binding_block_write |  |
| 2141 | 0x00543840 | FUN_00543840 | recorder_binding_block_read |  |
| 2142 | 0x00543980 | FUN_00543980 | recorder_bind_object_by_name |  |
| 2143 | 0x005439c0 | FUN_005439c0 | recorder_unbind_object_by_name |  |
| 2144 | 0x00543a00 | FUN_00543a00 | recorder_names_ctor |  |
| 2145 | 0x00543a20 | FUN_00543a20 | recorder_names_alloc_id |  |
| 2146 | 0x00543a30 | FUN_00543a30 | recorder_names_append |  |
| 2147 | 0x00543b50 | FUN_00543b50 | recorder_names_find_id |  |
| 2148 | 0x00543bd0 | FUN_00543bd0 | recorder_names_find_name_by_id |  |
| 2149 | 0x00543c10 | FUN_00543c10 | recorder_names_get_value_by_id |  |
| 2150 | 0x00543c50 | FUN_00543c50 | recorder_names_set_value_by_id |  |
| 2151 | 0x00543c90 | FUN_00543c90 | recorder_names_get_count |  |
| 2152 | 0x00543ca0 | FUN_00543ca0 | recorder_names_clear |  |
| 2153 | 0x00543cd0 | FUN_00543cd0 | recorder_names_copy_from_block |  |
| 2154 | 0x00543d50 | FUN_00543d50 | recorder_names_copy_to_block |  |
| 2155 | 0x00543db0 | FUN_00543db0 | recorder_names_read_stdio |  |
| 2156 | 0x00543e50 | FUN_00543e50 | recorder_names_write_stdio |  |
| 2157 | 0x00543ec0 | FUN_00543ec0 | recorder_names_read_dta |  |
| 2158 | 0x00543f60 | FUN_00543f60 | recorder_names_reset |  |
| 2159 | 0x00543f80 | FUN_00543f80 | recorder_names_serialized_size |  |
| 2160 | 0x00543fa0 | FUN_00543fa0 | recorder_get_or_create_name_id |  |
| 2161 | 0x00544030 | FUN_00544030 | recorder_load_anim_by_name |  |
| 2162 | 0x00544190 | FUN_00544190 | recorder_get_object_by_id |  |
| 2163 | 0x005441c0 | FUN_005441c0 | recorder_get_object_by_name |  |
| 2164 | 0x00544200 | FUN_00544200 | recorder_get_name_for_id |  |
| 2165 | 0x00544210 | FUN_00544210 | recorder_load_tck |  |
| 2166 | 0x00544590 | FUN_00544590 | chunk_close_handle | C_chunk::Close |
| 2167 | 0x00544680 | FUN_00544680 | recorder_load_anim_for_object |  |
| 2168 | 0x005447d0 | FUN_005447d0 | recorder_names_free_row_array |  |
| 2169 | 0x00544800 | FUN_00544800 | recorder_rebuild_object_table |  |
| 2170 | 0x005449f0 | FUN_005449f0 | recorder_release_object_table |  |
| 2171 | 0x00544a60 | FUN_00544a60 | recorder_names_free_buffers |  |
| 2172 | 0x00544a90 | FUN_00544a90 | recorder_get_or_create_name_id_direct |  |
| 2173 | 0x00544ad0 | FUN_00544ad0 | recorder_get_name_value |  |
| 2174 | 0x00544ae0 | FUN_00544ae0 | recorder_set_name_value |  |
| 2175 | 0x00544af0 | FUN_00544af0 | recorder_rewind_timeline_cursor |  |
| 2176 | 0x00544b00 | FUN_00544b00 | recorder_set_timeline_enabled |  |
| 2177 | 0x00544b10 | FUN_00544b10 | recorder_get_timeline_enabled |  |
| 2178 | 0x00544b20 | FUN_00544b20 | recorder_clear_binding_values |  |
| 2179 | 0x00544b30 | FUN_00544b30 | recorder_add_binding_value |  |
| 2180 | 0x00544b40 | FUN_00544b40 | recorder_load_bindings_file |  |
| 2181 | 0x00544b50 | FUN_00544b50 | recorder_detach_object_binding |  |
| 2182 | 0x00544b60 | FUN_00544b60 | recorder_attach_object_binding |  |
| 2183 | 0x00544b90 | FUN_00544b90 | recorder_compact_bindings |  |
| 2184 | 0x00544ba0 | FUN_00544ba0 | recorder_end_sequence_and_restore_scene |  |
| 2185 | 0x00544eb0 | FUN_00544eb0 | recorder_replay_state_write |  |
| 2186 | 0x00544f80 | FUN_00544f80 | recorder_replay_state_read |  |
| 2187 | 0x005450e0 | FUN_005450e0 | recorder_replay_state_size |  |
| 2188 | 0x00545110 | FUN_00545110 | recorder_singleton_static_init |  |
| 2189 | 0x00545120 | FUN_00545120 | recorder_singleton_construct |  |
| 2190 | 0x00545130 | FUN_00545130 | recorder_ctor |  |
| 2191 | 0x00545320 | FUN_00545320 | recorder_bindings_free_vectors |  |
| 2192 | 0x00545350 | FUN_00545350 | recorder_singleton_register_atexit |  |
| 2193 | 0x00545360 | FUN_00545360 | recorder_singleton_destruct |  |
| 2194 | 0x00545370 | FUN_00545370 | recorder_dtor |  |
| 2195 | 0x00545460 | FUN_00545460 | recorder_bindings_init |  |
| 2196 | 0x005454e0 | FUN_005454e0 | recorder_bindings_has_key_arena |  |
| 2197 | 0x005454f0 | FUN_005454f0 | recorder_bindings_adopt_key_arena |  |
| 2198 | 0x00545520 | FUN_00545520 | recorder_bindings_find_by_name |  |
| 2199 | 0x005455d0 | FUN_005455d0 | recorder_bindings_find_by_name_pair |  |
| 2200 | 0x005456e0 | FUN_005456e0 | recorder_bindings_remove_for_object |  |
| 2201 | 0x005457b0 | FUN_005457b0 | recorder_bindings_clear_all |  |
| 2202 | 0x00545840 | FUN_00545840 | recorder_bindings_release_key_arena |  |
| 2203 | 0x005458b0 | FUN_005458b0 | recorder_bindings_compact_and_rebind |  |
| 2204 | 0x005459c0 | FUN_005459c0 | recorder_bindings_clear_values |  |
| 2205 | 0x005459f0 | FUN_005459f0 | recorder_bindings_add_value |  |
| 2206 | 0x00545c30 | FUN_00545c30 | recorder_bindings_bind_track_by_name |  |
| 2207 | 0x00545f50 | FUN_00545f50 | recorder_add_track_binding |  |
| 2208 | 0x005461c0 | FUN_005461c0 | recorder_bindings_load_file |  |
| 2209 | 0x00546430 | FUN_00546430 | recorder_bindings_detach_object |  |
| 2210 | 0x005464a0 | FUN_005464a0 | recorder_bindings_attach_object |  |
| 2211 | 0x00546510 | FUN_00546510 | recorder_open_speech_sound_for_text_id |  |
| 2212 | 0x005465f0 | FUN_005465f0 | recorder_get_constant_5 |  |
| 2213 | 0x00546600 | FUN_00546600 | recorder_set_overlay_texture |  |
| 2214 | 0x00546630 | FUN_00546630 | recorder_rebind_speaker_channel |  |
| 2215 | 0x00546660 | FUN_00546660 | recorder_tick_speech |  |
| 2216 | 0x00546670 | FUN_00546670 | recorder_open_synced_stream |  |
| 2217 | 0x005466d0 | FUN_005466d0 | recorder_set_stream_time_base |  |
| 2218 | 0x005466e0 | FUN_005466e0 | recorder_sync_stream_to_clock |  |
| 2219 | 0x00546790 | FUN_00546790 | recorder_stop |  |
| 2220 | 0x005467c0 | FUN_005467c0 | recorder_is_sequence_running |  |
| 2221 | 0x005467e0 | FUN_005467e0 | stl_vector_ctor_with_allocator |  |
| 2222 | 0x00546810 | FUN_00546810 | vector_clear |  |
| 2223 | 0x00546840 | FUN_00546840 | stl_copy_stride_0x58 |  |
| 2224 | 0x00546880 | FUN_00546880 | stl_fill_n_stride_0x58 |  |
| 2225 | 0x005468b0 | FUN_005468b0 | stl_copy_stride_0x28 |  |
| 2226 | 0x005468f0 | FUN_005468f0 | stl_fill_n_stride_0x28 |  |
| 2227 | 0x00546920 | FUN_00546920 | stl_copy_stride_0x24_00546920 |  |
| 2228 | 0x00546960 | FUN_00546960 | stl_construct_stride_0x58 |  |
| 2229 | 0x00546980 | FUN_00546980 | stl_construct_stride_0x28 |  |
| 2230 | 0x005469a0 | FUN_005469a0 | speech_morph_list_static_init_entry |  |
| 2231 | 0x005469b0 | FUN_005469b0 | speech_morph_list_static_init |  |
| 2232 | 0x005469d0 | FUN_005469d0 | speech_morph_list_register_atexit |  |
| 2233 | 0x005469e0 | FUN_005469e0 | speech_morph_list_static_dtor |  |
| 2234 | 0x00546a00 | FUN_00546a00 | speech_init_channels | DabingInit |
| 2235 | 0x00546a50 | FUN_00546a50 | speech_release_channels | DabingDone |
| 2236 | 0x00546ae0 | FUN_00546ae0 | speech_get_free_channel | DabingGetFreeChanel |
| 2237 | 0x00546b10 | FUN_00546b10 | speech_channel_set_human | DabingSetHuman |
| 2238 | 0x00546ea0 | FUN_00546ea0 | speech_release_channels_and_reset_morphs |  |
| 2239 | 0x00547050 | FUN_00547050 | speech_channel_set_model | DabingSetModel |
| 2240 | 0x00547210 | FUN_00547210 | speech_invalidate_channel_for_actor | DabingInvalidate |
| 2241 | 0x005473b0 | FUN_005473b0 | speech_channel_speak_line | DabingSpeak |
| 2242 | 0x00547960 | FUN_00547960 | speech_channel_attach_accessory_frame |  |
| 2243 | 0x00547a80 | FUN_00547a80 | speech_channel_is_speaking | DabingIsSpeaking |
| 2244 | 0x00547b00 | FUN_00547b00 | speech_channel_stop | DabingStop |
| 2245 | 0x00547b50 | FUN_00547b50 | speech_update_accessory_frames |  |
| 2246 | 0x00547c10 | FUN_00547c10 | speech_tick_channels | DabingTick |
| 2247 | 0x00548420 | FUN_00548420 | speech_pause_all_channels | DabingPauseSounds |
| 2248 | 0x005484b0 | FUN_005484b0 | speech_preload_for_record | DabingPreLoad |
| 2249 | 0x005485a0 | FUN_005485a0 | speech_morph_frame_list_add |  |
| 2250 | 0x00548830 | FUN_00548830 | crt_static_init_guard_00548830 |  |
| 2251 | 0x00548850 | FUN_00548850 | crt_atexit_register_00548850 |  |
| 2252 | 0x00548860 | thunk_FUN_00548870 | thunk_crt_static_init_zero_vector_00656930 |  |
| 2253 | 0x00548870 | FUN_00548870 | crt_static_init_zero_vector_00656930 |  |
| 2254 | 0x00548890 | FUN_00548890 | recorder_key_stream_clear |  |
| 2255 | 0x005488a0 | FUN_005488a0 | recorder_key_stream_size |  |
| 2256 | 0x00548900 | FUN_00548900 | actor_track_key_offset_of |  |
| 2257 | 0x00548950 | FUN_00548950 | actor_track_key_stream_size |  |
| 2258 | 0x005489b0 | FUN_005489b0 | recorder_key_stream_new_chunk |  |
| 2259 | 0x005489e0 | FUN_005489e0 | recorder_key_stream_alloc |  |
| 2260 | 0x00548a60 | FUN_00548a60 | actor_track_set_recording |  |
| 2261 | 0x00548a80 | FUN_00548a80 | actor_track_set_playback |  |
| 2262 | 0x00548aa0 | FUN_00548aa0 | actor_track_apply_mode_changes |  |
| 2263 | 0x00548ac0 | FUN_00548ac0 | hud_static_init |  |
| 2264 | 0x00548ad0 | FUN_00548ad0 | hud_static_ctor |  |
| 2265 | 0x00548ae0 | FUN_00548ae0 | hud_register_atexit_dtor |  |
| 2266 | 0x00548af0 | FUN_00548af0 | hud_static_dtor |  |
| 2267 | 0x00548b00 | FUN_00548b00 | hud_dtor | G_IndicatorsClass::{dtor} |
| 2268 | 0x00548bf0 | FUN_00548bf0 | hud_ctor | G_IndicatorsClass::{ctor} |
| 2269 | 0x00548d50 | FUN_00548d50 | hud_reset_values | G_IndicatorsClass::ResetValues |
| 2270 | 0x00548f90 | FUN_00548f90 | hud_load_font | G_IndicatorsClass::LoadFont |
| 2271 | 0x00549060 | FUN_00549060 | hud_language_select | G_IndicatorsClass::LanguageSelect |
| 2272 | 0x00549120 | FUN_00549120 | hud_create | G_IndicatorsClass::Create |
| 2273 | 0x005492f0 | FUN_005492f0 | hud_destroy | G_IndicatorsClass::Destroy |
| 2274 | 0x005494b0 | FUN_005494b0 | hud_load_speedometer_odometer_texture |  |
| 2275 | 0x00549590 | FUN_00549590 | hud_game_done | G_IndicatorsClass::GameDone |
| 2276 | 0x005495f0 | FUN_005495f0 | hud_timer_set_time | G_IndicatorsClass::TimerSetTime |
| 2277 | 0x00549630 | FUN_00549630 | hud_timer_set_interval | G_IndicatorsClass::TimerSetInterval |
| 2278 | 0x005496e0 | FUN_005496e0 | hud_timer_get_time | G_IndicatorsClass::TimerGetTime |
| 2279 | 0x00549710 | FUN_00549710 | hud_timer_get_interval | G_IndicatorsClass::TimerGetInterval |
| 2280 | 0x00549730 | FUN_00549730 | vec2_mul_to |  |
| 2281 | 0x00549750 | FUN_00549750 | vec2_add_to |  |
| 2282 | 0x00549770 | FUN_00549770 | hud_build_needle_quad | G_IndicatorsClass::_SpocitejRafiku2 |
| 2283 | 0x00549a80 | FUN_00549a80 | hud_build_indicator_icon_quads | G_IndicatorsClass::_PridejWingPodklad |
| 2284 | 0x00549bc0 | FUN_00549bc0 | hud_build_ammo_icon_quads | G_IndicatorsClass::_PridejAmmoPodklad |
| 2285 | 0x00549cb0 | FUN_00549cb0 | hud_build_indicator_value_digits | G_IndicatorsClass::_PridejWingLives |
| 2286 | 0x0054a080 | FUN_0054a080 | hud_build_ammo_mag_reserve_quads | G_IndicatorsClass::_PridejAmmo |
| 2287 | 0x0054a780 | FUN_0054a780 | hud_speedometer_set_gear | G_IndicatorsClass::TachometrSetGear |
| 2288 | 0x0054a7a0 | FUN_0054a7a0 | hud_speedometer_set_values | G_IndicatorsClass::TachometrSetValues |
| 2289 | 0x0054a9d0 | FUN_0054a9d0 | hud_set_player_health_value | G_IndicatorsClass::PlayerSetLives |
| 2290 | 0x0054aa00 | FUN_0054aa00 | hud_set_player_ammo_mag_reserve | G_IndicatorsClass::PlayerSetAmmo |
| 2291 | 0x0054aa30 | FUN_0054aa30 | hud_set_player_accuracy | G_IndicatorsClass::PlayerSetAccuracy |
| 2292 | 0x0054aa40 | FUN_0054aa40 | hud_wingman_set_portrait_for_slot | G_IndicatorsClass::PlayerSetWingmanName |
| 2293 | 0x0054aa60 | FUN_0054aa60 | hud_wingman_set_indicator_value_for_slot | G_IndicatorsClass::PlayerSetWingmanLives |
| 2294 | 0x0054aa80 | FUN_0054aa80 | hud_subtitles_clear | G_IndicatorsClass::SubtitlesClear |
| 2295 | 0x0054aad0 | FUN_0054aad0 | hud_subtitles_add | G_IndicatorsClass::SubtitlesAdd |
| 2296 | 0x0054ae30 | FUN_0054ae30 | hud_subtitles_add_from_record |  |
| 2297 | 0x0054b160 | FUN_0054b160 | hud_intro_subtitles_add_intern | G_IndicatorsClass::IntroSubtitlesAddIntern |
| 2298 | 0x0054b5a0 | FUN_0054b5a0 | hud_intro_subtitles_queue_id_range | G_IndicatorsClass::IntroSubtitlesAdd |
| 2299 | 0x0054b7b0 | FUN_0054b7b0 | hud_intro_subtitles_add_centered |  |
| 2300 | 0x0054ba60 | FUN_0054ba60 | hud_intro_subtitles_clear | G_IndicatorsClass::IntroSubtitlesClear |
| 2301 | 0x0054bad0 | FUN_0054bad0 | hud_mission_help_clear | G_IndicatorsClass::MissionHelpClear |
| 2302 | 0x0054bb20 | FUN_0054bb20 | hud_mission_help_add | G_IndicatorsClass::MissionHelpAdd |
| 2303 | 0x0054be10 | FUN_0054be10 | hud_mission_help_set_time | G_IndicatorsClass::MissionHelpSetTime |
| 2304 | 0x0054be40 | FUN_0054be40 | hud_map_enable | G_IndicatorsClass::MapEnable |
| 2305 | 0x0054be70 | FUN_0054be70 | hud_console_clear | G_IndicatorsClass::ConsoleClear |
| 2306 | 0x0054be80 | FUN_0054be80 | hud_console_push_message | G_IndicatorsClass::ConsoleAddText |
| 2307 | 0x0054bf10 | FUN_0054bf10 | hud_console_print | G_IndicatorsClass::ConsoleAddTextByID |
| 2308 | 0x0054bf40 | FUN_0054bf40 | hud_push_blood_splat |  |
| 2309 | 0x0054c150 | FUN_0054c150 | hud_compass_set_destination_frame | G_IndicatorsClass::CompassSetDestination |
| 2310 | 0x0054c160 | FUN_0054c160 | hud_set_theft_indicator_value | G_IndicatorsClass::ThiefIndicatorSetValue |
| 2311 | 0x0054c170 | FUN_0054c170 | hud_set_airplane_damage_fill | G_IndicatorsClass::RefullingIndicatorSetValue |
| 2312 | 0x0054c180 | FUN_0054c180 | hud_widescreen_build_bars | G_IndicatorsClass::WideScreenSetRatio |
| 2313 | 0x0054c420 | FUN_0054c420 | hud_widescreen_fade_bars | G_IndicatorsClass::WideScreenSetRatio |
| 2314 | 0x0054c490 | FUN_0054c490 | hud_fade_set | G_IndicatorsClass::FadeInOut |
| 2315 | 0x0054c500 | FUN_0054c500 | hud_fade_get_state | G_IndicatorsClass::FadeInOutGetState |
| 2316 | 0x0054c520 | FUN_0054c520 | hud_fade_is_active | G_IndicatorsClass::FadeInOutIsDark |
| 2317 | 0x0054c530 | FUN_0054c530 | hud_set_screen_tint |  |
| 2318 | 0x0054c550 | FUN_0054c550 | hud_set_overlay_texture |  |
| 2319 | 0x0054c5c0 | FUN_0054c5c0 | hud_radar_clear | G_IndicatorsClass::RadarClear |
| 2320 | 0x0054c620 | FUN_0054c620 | hud_radar_set_player_car | G_IndicatorsClass::RadarSetPlayerCar |
| 2321 | 0x0054c630 | FUN_0054c630 | hud_radar_add_car_with_color | G_IndicatorsClass::RadarAddCar |
| 2322 | 0x0054c840 | FUN_0054c840 | hud_radar_remove_car | G_IndicatorsClass::RadarRemoveCar |
| 2323 | 0x0054c8a0 | FUN_0054c8a0 | hud_radar_add_object_with_color | G_IndicatorsClass::RadarAddObject |
| 2324 | 0x0054cab0 | FUN_0054cab0 | hud_radar_remove_object | G_IndicatorsClass::RadarRemoveObject |
| 2325 | 0x0054cb10 | FUN_0054cb10 | hud_radar_set_left_edge | G_IndicatorsClass::RadarSetLeftEdge |
| 2326 | 0x0054cd10 | FUN_0054cd10 | hud_radar_set_right_edge | G_IndicatorsClass::RadarSetRightEdge |
| 2327 | 0x0054cf10 | FUN_0054cf10 | hud_set_race_start_flag | G_IndicatorsClass::RaceSetStartFlag |
| 2328 | 0x0054cfc0 | FUN_0054cfc0 | hud_set_race_time_text | G_IndicatorsClass::RaceSetTime |
| 2329 | 0x0054d000 | FUN_0054d000 | hud_set_race_laptime_text | G_IndicatorsClass::RaceSetLapTime |
| 2330 | 0x0054d040 | FUN_0054d040 | hud_set_race_besttime_text | G_IndicatorsClass::RaceSetBestTime |
| 2331 | 0x0054d080 | FUN_0054d080 | hud_set_race_position_text | G_IndicatorsClass::RaceSetPosition |
| 2332 | 0x0054d0b0 | FUN_0054d0b0 | hud_set_race_lap_text | G_IndicatorsClass::RaceSetLaps |
| 2333 | 0x0054d0e0 | FUN_0054d0e0 | hud_set_race_banner_text | G_IndicatorsClass::RaceFlashText |
| 2334 | 0x0054d130 | FUN_0054d130 | hud_freeride_update_score_text | G_IndicatorsClass::BonusSetCount |
| 2335 | 0x0054d180 | FUN_0054d180 | hud_render_all | G_IndicatorsClass::DrawAll |
| 2336 | 0x005532e0 | FUN_005532e0 | hud_rebuild_layout_for_resolution | G_IndicatorsClass::ResolutionChanged |
| 2337 | 0x00553ed0 | FUN_00553ed0 | vec2_div_scalar_to |  |
| 2338 | 0x00553ef0 | FUN_00553ef0 | hud_build_rect_quad |  |
| 2339 | 0x00553fd0 | FUN_00553fd0 | hud_build_diamond_quad |  |
| 2340 | 0x005540b0 | FUN_005540b0 | hud_tick | G_IndicatorsClass::Tick |
| 2341 | 0x00554fd0 | FUN_00554fd0 | hud_text_measure | G_IndicatorsClass::TextSize |
| 2342 | 0x00555090 | FUN_00555090 | hud_text_find_line_break | G_IndicatorsClass::HledejZlom |
| 2343 | 0x005551a0 | FUN_005551a0 | hud_text_draw_line | G_IndicatorsClass::OutText |
| 2344 | 0x00556040 | FUN_00556040 | hud_text_draw_multiline | G_IndicatorsClass::OutTextMultiLine |
| 2345 | 0x005560b0 | FUN_005560b0 | hud_text_measure_fixed_pitch |  |
| 2346 | 0x00556100 | FUN_00556100 | hud_text_draw_line_alt_font |  |
| 2347 | 0x00556d80 | FUN_00556d80 | hud_draw_mouse_cursor |  |
| 2348 | 0x00556ea0 | FUN_00556ea0 | std_vector_end |  |
| 2349 | 0x00556eb0 | FUN_00556eb0 | std_vector1c_size |  |
| 2350 | 0x00556ee0 | FUN_00556ee0 | crt_static_init_guard_00556ee0 |  |
| 2351 | 0x00556f00 | FUN_00556f00 | crt_atexit_register_00556f00 |  |
| 2352 | 0x00556f10 | FUN_00556f10 | loadsave_static_init |  |
| 2353 | 0x00556f20 | FUN_00556f20 | loadsave_static_ctor_call |  |
| 2354 | 0x00556f30 | FUN_00556f30 | loadsave_static_atexit_register |  |
| 2355 | 0x00556f40 | FUN_00556f40 | loadsave_static_dtor_call |  |
| 2356 | 0x00556f50 | FUN_00556f50 | car_database_static_init |  |
| 2357 | 0x00556f60 | FUN_00556f60 | car_database_static_ctor_call |  |
| 2358 | 0x00556f70 | FUN_00556f70 | car_database_static_atexit_register |  |
| 2359 | 0x00556f80 | FUN_00556f80 | car_database_static_dtor_call |  |
| 2360 | 0x00556f90 | FUN_00556f90 | loadsave_init | G_LoadSaveClass::Init |
| 2361 | 0x005570c0 | FUN_005570c0 | loadsave_close | G_LoadSaveClass::Close |
| 2362 | 0x00557100 | FUN_00557100 | loadsave_profile_change | G_LoadSaveClass::ProfileChange |
| 2363 | 0x005571d0 | FUN_005571d0 | loadsave_ctor | G_LoadSaveClass::{ctor} |
| 2364 | 0x00557270 | FUN_00557270 | loadsave_all_default | G_LoadSaveClass::AllDefault |
| 2365 | 0x005572f0 | FUN_005572f0 | loadsave_profile_enum_files | G_LoadSaveClass::ProfileEnumFiles |
| 2366 | 0x005575c0 | FUN_005575c0 | loadsave_profile_enum_release | G_LoadSaveClass::ProfileEnumRelease |
| 2367 | 0x005575f0 | FUN_005575f0 | folded_return_this |  |
| 2368 | 0x00557600 | FUN_00557600 | loadsave_profile_select | G_LoadSaveClass::ProfileSelect |
| 2369 | 0x00557650 | FUN_00557650 | loadsave_profile_create | G_LoadSaveClass::ProfileCreate |
| 2370 | 0x005577d0 | FUN_005577d0 | loadsave_profile_delete | G_LoadSaveClass::ProfileDelete |
| 2371 | 0x00557880 | FUN_00557880 | loadsave_profile_load | G_LoadSaveClass::ProfileLoad |
| 2372 | 0x00557a20 | FUN_00557a20 | loadsave_profile_save | G_LoadSaveClass::ProfileSave |
| 2373 | 0x00557b50 | FUN_00557b50 | loadsave_savegame_enum_files | G_LoadSaveClass::SaveGameEnumFiles |
| 2374 | 0x00557e40 | FUN_00557e40 | loadsave_savegame_enum_release | G_LoadSaveClass::SaveGameEnumRelease |
| 2375 | 0x00557e70 | FUN_00557e70 | loadsave_savegame_select | G_LoadSaveClass::SaveGameSelect |
| 2376 | 0x00557eb0 | FUN_00557eb0 | loadsave_savegame_get_selected_number | G_LoadSaveClass::SaveGameGetSelectedNumber |
| 2377 | 0x00557ec0 | FUN_00557ec0 | loadsave_savegame_create | G_LoadSaveClass::SaveGameCreate |
| 2378 | 0x00558020 | FUN_00558020 | loadsave_savegame_new_game | G_LoadSaveClass::SaveGameNewGame |
| 2379 | 0x00558050 | FUN_00558050 | loadsave_savegame_load_header | G_LoadSaveClass::SaveGameLoadHeader |
| 2380 | 0x00558180 | FUN_00558180 | loadsave_savegame_load | G_LoadSaveClass::SaveGameLoad |
| 2381 | 0x00558db0 | FUN_00558db0 | loadsave_save_actor_proc | G_LoadSaveClass::_SaveProc |
| 2382 | 0x00558ed0 | FUN_00558ed0 | loadsave_savegame_save | G_LoadSaveClass::SaveGameSave |
| 2383 | 0x00559310 | FUN_00559310 | loadsave_get_screenshot_idx | G_LoadSaveClass::GetScreenShotIdx |
| 2384 | 0x00559320 | FUN_00559320 | loadsave_savegame_from_script | G_LoadSaveClass::SaveGameFromScript |
| 2385 | 0x00559390 | FUN_00559390 | loadsave_language_select | G_LoadSaveClass::LanguageSelect |
| 2386 | 0x005593f0 | FUN_005593f0 | loadsave_binding_set |  |
| 2387 | 0x00559430 | FUN_00559430 | loadsave_binding_set_from_event |  |
| 2388 | 0x00559490 | FUN_00559490 | loadsave_binding_find_conflict |  |
| 2389 | 0x00559550 | FUN_00559550 | loadsave_binding_clear_conflicts |  |
| 2390 | 0x00559630 | FUN_00559630 | loadsave_default_key_bindings |  |
| 2391 | 0x00559a90 | FUN_00559a90 | loadsave_apply_key_bindings |  |
| 2392 | 0x00559ae0 | FUN_00559ae0 | loadsave_default_settings | G_LoadSaveClass::DefaultSettings |
| 2393 | 0x00559ba0 | FUN_00559ba0 | inventory_get_in_hand_item_def | G_Inventory::GetWeaponInRightHand |
| 2394 | 0x00559bd0 | FUN_00559bd0 | game_item_table_get | G_Inventory::GetWeaponInLeftHand |
| 2395 | 0x00559be0 | FUN_00559be0 | inventory_select_slot | G_Inventory::Select |
| 2396 | 0x0055a1f0 | FUN_0055a1f0 | inventory_select_by_id | G_Inventory::SelectByID |
| 2397 | 0x0055a2c0 | FUN_0055a2c0 | inventory_holster | G_Inventory::Holster |
| 2398 | 0x0055a540 | FUN_0055a540 | inventory_has_item_with_property | G_Inventory::HaveSomeWeapon |
| 2399 | 0x0055a610 | FUN_0055a610 | inventory_have_items | G_Inventory::HaveItems |
| 2400 | 0x0055a790 | FUN_0055a790 | inventory_build_cycle_list | G_Inventory::CyclePrepare |
| 2401 | 0x0055a910 | FUN_0055a910 | inventory_cycle_to_prev | G_Inventory::CycleUp |
| 2402 | 0x0055a9b0 | FUN_0055a9b0 | inventory_cycle_to_next | G_Inventory::CycleDown |
| 2403 | 0x0055aa40 | FUN_0055aa40 | inventory_contains_item | G_Inventory::IsIn |
| 2404 | 0x0055aac0 | FUN_0055aac0 | inventory_can_accept_item | G_Inventory::HaveSpaceFor |
| 2405 | 0x0055ac60 | FUN_0055ac60 | inventory_merge_ammo_into_slot | G_Inventory::DobijNaboje |
| 2406 | 0x0055ad00 | FUN_0055ad00 | inventory_place_in_free_slot |  |
| 2407 | 0x0055ae40 | FUN_0055ae40 | inventory_insert_new_item | G_Inventory::InsertNew |
| 2408 | 0x0055b2d0 | FUN_0055b2d0 | inventory_insert_item | G_Inventory::Insert |
| 2409 | 0x0055b580 | FUN_0055b580 | inventory_remove_slot | G_Inventory::RemoveByIdx |
| 2410 | 0x0055b600 | FUN_0055b600 | inventory_remove_item_by_id | G_Inventory::Remove |
| 2411 | 0x0055b940 | FUN_0055b940 | inventory_set_can_carry_big_weapon | G_Inventory::SetCanHaveBigWeapon |
| 2412 | 0x0055bb80 | FUN_0055bb80 | inventory_set_in_car | G_Inventory::SetInCar |
| 2413 | 0x0055bbd0 | FUN_0055bbd0 | inventory_collect_items_to_list | G_Inventory::EnemyToList |
| 2414 | 0x0055bcd0 | FUN_0055bcd0 | inventory_remove_item_by_ptr | G_Inventory::RemoveByPtr |
| 2415 | 0x0055bd40 | FUN_0055bd40 | inventory_get_active_weapon_ammo | G_Inventory::GetAmmo |
| 2416 | 0x0055bd80 | FUN_0055bd80 | inventory_get_ammo_by_id | G_Inventory::GetAmmoByID |
| 2417 | 0x0055be70 | FUN_0055be70 | inventory_consume_round | G_Inventory::Vystrel |
| 2418 | 0x0055be90 | FUN_0055be90 | inventory_reload_magazine | G_Inventory::Nabij |
| 2419 | 0x0055bf20 | FUN_0055bf20 | inventory_reload_one_round | G_Inventory::NabijJeden |
| 2420 | 0x0055bf60 | FUN_0055bf60 | inventory_can_reload_one_round | G_Inventory::MuzeNabitJeden |
| 2421 | 0x0055bfa0 | FUN_0055bfa0 | inventory_compact_slots | G_Inventory::_Sesyp |
| 2422 | 0x0055c020 | FUN_0055c020 | inventory_savegame_get_size | G_Inventory::SaveGameGetSize |
| 2423 | 0x0055c030 | FUN_0055c030 | inventory_savegame_save | G_Inventory::SaveGameSave |
| 2424 | 0x0055c150 | FUN_0055c150 | inventory_savegame_load | G_Inventory::SaveGameLoad |
| 2425 | 0x0055c270 | FUN_0055c270 | enabled_cars_have_car | G_EnabledCars::HaveCar |
| 2426 | 0x0055c310 | FUN_0055c310 | car_database_ctor | G_CarDatabase::{ctor} |
| 2427 | 0x0055c370 | FUN_0055c370 | car_database_init | G_CarDatabase::Init |
| 2428 | 0x0055c440 | FUN_0055c440 | car_database_close | G_CarDatabase::Close |
| 2429 | 0x0055c480 | FUN_0055c480 | car_database_get_car | G_CarDatabase::GetCar |
| 2430 | 0x0055c4b0 | FUN_0055c4b0 | car_database_get_car_details | G_CarDatabase::GetCarDetails |
| 2431 | 0x0055c4e0 | FUN_0055c4e0 | car_database_get_player_record |  |
| 2432 | 0x0055c510 | FUN_0055c510 | car_database_get_player_model | G_CarDatabase::GetPlayerModel |
| 2433 | 0x0055c540 | FUN_0055c540 | car_database_get_car_name | G_CarDatabase::GetCarName |
| 2434 | 0x0055c570 | FUN_0055c570 | car_database_get_car_variant_name_id | G_CarDatabase::GetCarSecondName |
| 2435 | 0x0055c620 | FUN_0055c620 | car_database_clamp_packed_id | G_CarDatabase::RepairIndex |
| 2436 | 0x0055c670 | FUN_0055c670 | car_database_get_max_color_index | G_CarDatabase::GetCarMaxColor |
| 2437 | 0x0055c6b0 | FUN_0055c6b0 | car_database_get_car_model | G_CarDatabase::GetCarModel |
| 2438 | 0x0055c710 | FUN_0055c710 | car_database_get_car_params | G_CarDatabase::GetCarParams |
| 2439 | 0x0055c7a0 | FUN_0055c7a0 | car_database_pack_id | G_CarDatabase::GetIDByParams |
| 2440 | 0x0055c7c0 | FUN_0055c7c0 | car_database_pack_id_by_model |  |
| 2441 | 0x0055c7e0 | FUN_0055c7e0 | car_database_get_index_by_model | G_CarDatabase::GetIdxByModel |
| 2442 | 0x0055c830 | FUN_0055c830 | car_database_get_index_by_model_filename | G_CarDatabase::GetIdxByModelLong |
| 2443 | 0x0055c8f0 | FUN_0055c8f0 | car_database_pack_id_from_car | G_CarDatabase::GetIDByActor |
| 2444 | 0x0055c950 | FUN_0055c950 | car_database_get_wreck_model | G_CarDatabase::GetWreckageByModel |
| 2445 | 0x0055ca10 | FUN_0055ca10 | car_database_get_shadow_model | G_CarDatabase::GetShadowByModel |
| 2446 | 0x0055cad0 | FUN_0055cad0 | car_database_get_civil_mask_by_model | G_CarDatabase::GetCivilMaskByModel |
| 2447 | 0x0055cb30 | FUN_0055cb30 | car_database_collect_cars_by_class_mask | G_CarDatabase::GetCarsByClass |
| 2448 | 0x0055cbe0 | FUN_0055cbe0 | car_database_collect_cars_by_manufacturer |  |
| 2449 | 0x0055cca0 | FUN_0055cca0 | car_database_collect_cars_by_carcyclopedia_mask | G_CarDatabase::GetCarsByCarcyclopediaMask |
| 2450 | 0x0055cd40 | FUN_0055cd40 | car_database_is_enabled_for_race | G_CarDatabase::IsEnabledForRace |
| 2451 | 0x0055cd70 | FUN_0055cd70 | car_database_is_enabled_for_championship | G_CarDatabase::IsEnabledForChampionship |
| 2452 | 0x0055cda0 | FUN_0055cda0 | car_database_is_enabled_for_freeride | G_CarDatabase::IsEnabledForFreeride |
| 2453 | 0x0055cdd0 | FUN_0055cdd0 | car_database_get_car_class_mask | G_CarDatabase::GetCarClass |
| 2454 | 0x0055ce00 | FUN_0055ce00 | car_database_get_steal_time | G_CarDatabase::GetStealTime |
| 2455 | 0x0055ce60 | FUN_0055ce60 | car_database_get_player_civil_mask_by_model | G_CarDatabase::GetPlayerCivilMaskByModel |
| 2456 | 0x0055ceb0 | FUN_0055ceb0 | car_database_collect_player_models_by_civil_mask |  |
| 2457 | 0x0055cf30 | FUN_0055cf30 | car_database_load_details | G_CarDatabase::LoadDetails |
| 2458 | 0x0055cfc0 | FUN_0055cfc0 | car_database_unload_details | G_CarDatabase::UnloadDetails |
| 2459 | 0x0055d010 | FUN_0055d010 | stream_ctor | G_Stream::{ctor} |
| 2460 | 0x0055d040 | thunk_FUN_0055d130 | thunk_stream_close |  |
| 2461 | 0x0055d050 | FUN_0055d050 | stream_set_params | G_Stream::SetParams |
| 2462 | 0x0055d070 | FUN_0055d070 | stream_open_read | G_Stream::OpenRead |
| 2463 | 0x0055d0c0 | FUN_0055d0c0 | stream_open_write | G_Stream::OpenWrite |
| 2464 | 0x0055d130 | FUN_0055d130 | stream_close | G_Stream::Close |
| 2465 | 0x0055d160 | FUN_0055d160 | stream_begin_block | G_Stream::WantWrite |
| 2466 | 0x0055d190 | FUN_0055d190 | stream_end_block | G_Stream::Write |
| 2467 | 0x0055d1d0 | FUN_0055d1d0 | stream_read_block | G_Stream::Read |
| 2468 | 0x0055d220 | FUN_0055d220 | stream_encrypt_dwords | G_Stream::Crypt |
| 2469 | 0x0055d270 | FUN_0055d270 | stream_decrypt_dwords | G_Stream::Decrypt |
| 2470 | 0x0055d2b0 | FUN_0055d2b0 | stream_write_at | G_Stream::WriteAdr |
| 2471 | 0x0055d340 | FUN_0055d340 | stream_read_at | G_Stream::ReadAdr |
| 2472 | 0x0055d3c0 | FUN_0055d3c0 | loadsave_replay_tick | G_LoadSaveClass::ReplayTick |
| 2473 | 0x0055d430 | FUN_0055d430 | loadsave_replay_set_mode | G_LoadSaveClass::ReplaySet |
| 2474 | 0x0055d490 | FUN_0055d490 | loadsave_replay_load | G_LoadSaveClass::ReplayLoad |
| 2475 | 0x0055d560 | FUN_0055d560 | loadsave_replay_save |  |
| 2476 | 0x0055d640 | FUN_0055d640 | loadsave_replay_read_next_frame | G_LoadSaveClass::_replayGetNext |
| 2477 | 0x0055d680 | FUN_0055d680 | loadsave_replay_write_next_frame | G_LoadSaveClass::_replayPutNext |
| 2478 | 0x0055d6b0 | FUN_0055d6b0 | loadsave_race_cars_enable | G_LoadSaveClass::RaceCarsEnable |
| 2479 | 0x0055d6d0 | FUN_0055d6d0 | loadsave_race_cars_disable | G_LoadSaveClass::RaceCarsDisable |
| 2480 | 0x0055d6f0 | FUN_0055d6f0 | loadsave_race_cars_get_enabled | G_LoadSaveClass::RaceCarsGetEnabled |
| 2481 | 0x0055d700 | FUN_0055d700 | loadsave_race_cars_unlock_from_car | G_LoadSaveClass::RaceCarsAddWin |
| 2482 | 0x0055d730 | FUN_0055d730 | loadsave_objectives_clear | G_LoadSaveClass::ObjectivesClear |
| 2483 | 0x0055d760 | FUN_0055d760 | loadsave_objectives_add | G_LoadSaveClass::ObjectivesAdd |
| 2484 | 0x0055d940 | FUN_0055d940 | loadsave_objectives_remove | G_LoadSaveClass::ObjectivesRemove |
| 2485 | 0x0055d990 | FUN_0055d990 | loadsave_objectives_display | G_LoadSaveClass::ObjectivesDisplay |
| 2486 | 0x0055d9e0 | FUN_0055d9e0 | garage_clear | G_GarageClass::Clear |
| 2487 | 0x0055da10 | FUN_0055da10 | garage_add_car_by_index | G_GarageClass::AddCarByIdx |
| 2488 | 0x0055da90 | FUN_0055da90 | garage_add_car_by_actor | G_GarageClass::AddCarByActor |
| 2489 | 0x0055db40 | FUN_0055db40 | garage_remove_car_by_model | G_GarageClass::RemoveCar |
| 2490 | 0x0055db90 | FUN_0055db90 | garage_game_init | G_GarageClass::GameInit |
| 2491 | 0x0055e6e0 | FUN_0055e6e0 | garage_game_done | G_GarageClass::GameDone |
| 2492 | 0x0055f510 | FUN_0055f510 | garage_run_car_select_menu | G_GarageClass::CarManager |
| 2493 | 0x0055fa90 | FUN_0055fa90 | garage_destroy_car | G_GarageClass::DestroyCar |
| 2494 | 0x0055fc90 | FUN_0055fc90 | garage_generate_cars | G_GarageClass::GenerateCars |
| 2495 | 0x0055fca0 | FUN_0055fca0 | garage_release_cars | G_GarageClass::ReleaseCars |
| 2496 | 0x0055fcb0 | FUN_0055fcb0 | garage_show_results | G_GarageClass::GarageShowResultsProc |
| 2497 | 0x0055fea0 | FUN_0055fea0 | garage_set_stolen_car | G_GarageClass::CarIsStolen |
| 2498 | 0x0055feb0 | FUN_0055feb0 | garage_set_blocked_car | G_GarageClass::CarCantGetOut |
| 2499 | 0x0055fec0 | FUN_0055fec0 | garage_enable_steal_for_letter | G_GarageClass::EnableSteal |
| 2500 | 0x0055ff00 | FUN_0055ff00 | garage_can_steal_car_model | G_GarageClass::CanSteal |
| 2501 | 0x0055ff40 | FUN_0055ff40 | garage_is_car_parked | G_GarageClass::IsCarIn |
| 2502 | 0x0055ff70 | FUN_0055ff70 | garage_enable_start_for_letter |  |
| 2503 | 0x0055ffb0 | FUN_0055ffb0 | loadsave_freeride_get_enabled_cars | G_LoadSaveClass::FreerideGetEnabledCars |
| 2504 | 0x0055ffc0 | FUN_0055ffc0 | loadsave_apply_settings | G_LoadSaveClass::Refresh |
| 2505 | 0x00560120 | FUN_00560120 | loadsave_get_key_binding_label_id |  |
| 2506 | 0x00560140 | FUN_00560140 | credits_render_one | RenderOneCredit |
| 2507 | 0x00560550 | FUN_00560550 | hud_text_draw_line_scaled |  |
| 2508 | 0x005605b0 | FUN_005605b0 | credits_parse | ParseCredits |
| 2509 | 0x00560690 | FUN_00560690 | loadsave_execute_credits | G_LoadSaveClass::ExecuteCredits |
| 2510 | 0x00560890 | FUN_00560890 | loadsave_dtor | G_LoadSaveClass::{dtor} |
| 2511 | 0x005608e0 | FUN_005608e0 | loadsave_freeride_set_mode | G_LoadSaveClass::FreerideEnable |
| 2512 | 0x005608f0 | FUN_005608f0 | loadsave_freeride_store_params | G_LoadSaveClass::FreerideStoreA |
| 2513 | 0x00560930 | FUN_00560930 | loadsave_freeride_store_state | G_LoadSaveClass::FreerideStoreB |
| 2514 | 0x00560940 | FUN_00560940 | loadsave_freeride_get_state | G_LoadSaveClass::FreerideGetState |
| 2515 | 0x00560950 | FUN_00560950 | loadsave_freeride_apply_script_params | G_LoadSaveClass::FreerideSetupFromScript |
| 2516 | 0x005609a0 | FUN_005609a0 | loadsave_freeride_setup_scene | G_LoadSaveClass::FreerideSetupCar |
| 2517 | 0x00560ec0 | FUN_00560ec0 | loadsave_freeride_setup_after | G_LoadSaveClass::FreerideSetupAfter |
| 2518 | 0x00560f20 | FUN_00560f20 | loadsave_profile_count |  |
| 2519 | 0x00560f50 | FUN_00560f50 | stl_insert_n_grow_stride_0xc |  |
| 2520 | 0x00561170 | FUN_00561170 | vector_insert_stride_0x54 | std::__vector_imp<16ai_character_log,Q23std29allocator<16ai_character_log>,0>::insert |
| 2521 | 0x00561480 | FUN_00561480 | stl_insert_n_stride_0x18 |  |
| 2522 | 0x00561780 | FUN_00561780 | stl_insert_n_shift_stride_0xc |  |
| 2523 | 0x00561ab0 | FUN_00561ab0 | copy_range_stride_0x54 |  |
| 2524 | 0x00561af0 | FUN_00561af0 | fill_n_stride_0x54 |  |
| 2525 | 0x00561b20 | FUN_00561b20 | vec3_copy |  |
| 2526 | 0x00561b40 | FUN_00561b40 | crt_static_init_guard_00561b40 |  |
| 2527 | 0x00561b60 | FUN_00561b60 | crt_atexit_register_00561b60 |  |
| 2528 | 0x00561b70 | FUN_00561b70 | carcyclopedia_ctor | GM_Carcyclopedia::OnCreate |
| 2529 | 0x00561fd0 | FUN_00561fd0 | carcyclopedia_scalar_deleting_dtor | GM_Carcyclopedia::{dtor} |
| 2530 | 0x00561ff0 | FUN_00561ff0 | carcyclopedia_dtor | GM_Carcyclopedia::{dtor} |
| 2531 | 0x00562040 | FUN_00562040 | carcyclopedia_on_close | GM_Carcyclopedia::OnClose |
| 2532 | 0x00562080 | FUN_00562080 | carcyclopedia_on_exchange | GM_Menu::OnExchange |
| 2533 | 0x005620b0 | FUN_005620b0 | carcyclopedia_refresh_display |  |
| 2534 | 0x00562120 | FUN_00562120 | carcyclopedia_tick | GM_Carcyclopedia::OnTick |
| 2535 | 0x00562bb0 | FUN_00562bb0 | carcyclopedia_build_car_list | GM_Carcyclopedia::OnCreate |
| 2536 | 0x00562da0 | FUN_00562da0 | carcyclopedia_refresh_car | GM_Carcyclopedia::RefreshCar |
| 2537 | 0x00562fa0 | FUN_00562fa0 | gui_mp_person_select_ctor |  |
| 2538 | 0x005634a0 | FUN_005634a0 | gui_menu_on_exchange_forward_to_parent | GM_Menu::OnExchange |
| 2539 | 0x005634b0 | FUN_005634b0 | gui_mp_person_select_on_close |  |
| 2540 | 0x005634d0 | FUN_005634d0 | gui_mp_person_select_tick |  |
| 2541 | 0x005639e0 | FUN_005639e0 | gui_mp_person_select_show_model |  |
| 2542 | 0x00563b00 | FUN_00563b00 | crt_static_init_guard_00563b00 |  |
| 2543 | 0x00563b20 | FUN_00563b20 | crt_atexit_register_00563b20 |  |
| 2544 | 0x00563b30 | FUN_00563b30 | gui_mp_menu_ctor |  |
| 2545 | 0x00563c20 | FUN_00563c20 | gui_mp_menu_refresh_mode_items |  |
| 2546 | 0x00563cc0 | FUN_00563cc0 | gui_mp_menu_tick |  |
| 2547 | 0x00563ee0 | FUN_00563ee0 | gui_mp_lobby_ctor |  |
| 2548 | 0x00564110 | FUN_00564110 | gui_mp_lobby_on_child_result |  |
| 2549 | 0x00564320 | FUN_00564320 | gui_mp_lobby_tick |  |
| 2550 | 0x00564f20 | FUN_00564f20 | gui_mp_settings_ctor |  |
| 2551 | 0x00565610 | FUN_00565610 | gui_mp_settings_scalar_deleting_dtor |  |
| 2552 | 0x00565630 | FUN_00565630 | gui_mp_settings_dtor |  |
| 2553 | 0x00565760 | FUN_00565760 | gui_mp_settings_on_exchange |  |
| 2554 | 0x00565a70 | FUN_00565a70 | gui_mp_settings_read_widgets |  |
| 2555 | 0x00565ac0 | FUN_00565ac0 | gui_mp_settings_write_widgets |  |
| 2556 | 0x00565bc0 | FUN_00565bc0 | gui_mp_settings_on_change |  |
| 2557 | 0x00565d00 | FUN_00565d00 | gui_mp_settings_tick |  |
| 2558 | 0x00565db0 | FUN_00565db0 | gui_mp_settings_set_advanced_visible |  |
| 2559 | 0x00565e90 | FUN_00565e90 | gui_mp_settings_resize_window |  |
| 2560 | 0x00565f00 | FUN_00565f00 | gui_race_champ_select_ctor |  |
| 2561 | 0x005660f0 | FUN_005660f0 | gui_race_champ_select_tick |  |
| 2562 | 0x00566190 | FUN_00566190 | gui_mp_message_screen_ctor |  |
| 2563 | 0x00566200 | FUN_00566200 | vector_dispose_at_0x04 |  |
| 2564 | 0x00566220 | FUN_00566220 | copy_range_stride_0x8 |  |
| 2565 | 0x00566250 | FUN_00566250 | crt_static_init_guard_00566250 |  |
| 2566 | 0x00566270 | FUN_00566270 | crt_atexit_register_00566270 |  |
| 2567 | 0x00566280 | FUN_00566280 | gui_race_root_ctor | GM_Menu::{ctor} |
| 2568 | 0x00566310 | FUN_00566310 | gui_race_root_tick | GM_Racing::OnClick |
| 2569 | 0x00566490 | FUN_00566490 | gui_race_root_open_single_race | GM_Racing::OpenMenuForSingleRace |
| 2570 | 0x00566560 | FUN_00566560 | gui_race_root_open_championship | GM_Racing::OpenMenuForChampionship |
| 2571 | 0x005666a0 | FUN_005666a0 | gui_race_championship_ctor | GM_Championship::OnCreate |
| 2572 | 0x005668c0 | FUN_005668c0 | gui_race_championship_tick | GM_Championship::OnClick |
| 2573 | 0x005669f0 | FUN_005669f0 | gui_race_round_select_ctor | GM_SelectInRound::OnCreate |
| 2574 | 0x00566b80 | FUN_00566b80 | gui_race_round_select_tick | GM_SelectInRound::OnClick |
| 2575 | 0x00566c80 | FUN_00566c80 | gui_race_champ_launch_ctor | GM_ChampionshipRace::OnCreate |
| 2576 | 0x00567170 | FUN_00567170 | gui_race_champ_launch_on_close_child | GM_ChampionshipRace::OnCloseChild |
| 2577 | 0x00567280 | FUN_00567280 | gui_race_champ_launch_tick | GM_ChampionshipRace::OnClick |
| 2578 | 0x00567450 | FUN_00567450 | gui_race_champ_results_ctor | GM_ChampionshipResults::OnCreate |
| 2579 | 0x00567b20 | FUN_00567b20 | gui_race_champ_results_tick | GM_ChampionshipResults::OnClick |
| 2580 | 0x00567fb0 | FUN_00567fb0 | gui_race_setup_ctor | GM_SingleRace::OnCreate |
| 2581 | 0x00568100 | FUN_00568100 | gui_race_setup_on_close_child | GM_SingleRace::OnCloseChild |
| 2582 | 0x005681e0 | FUN_005681e0 | gui_race_setup_tick | GM_SingleRace::OnClick |
| 2583 | 0x00568410 | FUN_00568410 | gui_race_circuit_select_ctor | GM_SelectCircuit::OnCreate |
| 2584 | 0x005685d0 | FUN_005685d0 | gui_race_circuit_select_tick | GM_SelectCircuit::OnClick |
| 2585 | 0x005686f0 | FUN_005686f0 | gui_race_opponents_ctor | GM_SelectOpponents::OnCreate |
| 2586 | 0x00568830 | FUN_00568830 | gui_race_opponents_on_close_child | GM_SelectOpponents::OnCloseChild |
| 2587 | 0x00568880 | FUN_00568880 | gui_race_opponents_tick | GM_SelectOpponents::OnClick |
| 2588 | 0x005689c0 | FUN_005689c0 | gui_race_car_maker_ctor | GM_SelectCarManufacturer::OnCreate |
| 2589 | 0x00568c70 | FUN_00568c70 | gui_race_car_maker_on_close_child | GM_SelectCarManufacturer::OnCloseChild |
| 2590 | 0x00568cc0 | FUN_00568cc0 | gui_race_car_maker_tick | GM_SelectCarManufacturer::OnClick |
| 2591 | 0x00568d90 | FUN_00568d90 | gui_race_car_model_ctor | GM_SelectCarModel::OnCreate |
| 2592 | 0x00569080 | FUN_00569080 | gui_race_car_model_scalar_deleting_dtor | GM_SelectCarModel::{dtor} |
| 2593 | 0x005690a0 | FUN_005690a0 | gui_race_car_model_dtor | GM_SelectCarModel::{dtor} |
| 2594 | 0x005690d0 | FUN_005690d0 | gui_race_car_model_close | GM_SelectCarModel::OnClose |
| 2595 | 0x00569100 | FUN_00569100 | gui_race_car_model_on_close_child | GM_SelectCarModel::OnCloseChild |
| 2596 | 0x00569150 | FUN_00569150 | gui_race_car_model_tick | GM_SelectCarModel::OnClick |
| 2597 | 0x005693d0 | FUN_005693d0 | gui_race_car_color_ctor | GM_SelectCarModification::OnCreate |
| 2598 | 0x00569500 | thunk_FUN_0056e930 | thunk_gui_menu_close |  |
| 2599 | 0x00569510 | FUN_00569510 | gui_race_car_color_tick | GM_SelectCarModification::OnClick |
| 2600 | 0x00569660 | FUN_00569660 | gui_race_launch_ctor | GM_SingleRaceSecond::OnCreate |
| 2601 | 0x00569ab0 | FUN_00569ab0 | gui_race_launch_on_close_child |  |
| 2602 | 0x00569bd0 | FUN_00569bd0 | gui_race_launch_tick | GM_SingleRaceSecond::OnClick |
| 2603 | 0x00569da0 | FUN_00569da0 | gui_race_results_ctor | GM_SingleRaceResults::OnCreate |
| 2604 | 0x0056a570 | FUN_0056a570 | gui_race_results_tick | GM_SingleRaceResults::OnClick |
| 2605 | 0x0056ab40 | FUN_0056ab40 | gui_race_bonus_car_ctor | GM_BonusCar::OnCreate |
| 2606 | 0x0056acd0 | FUN_0056acd0 | gui_race_bonus_car_tick | GM_BonusCar::OnClick |
| 2607 | 0x0056adc0 | FUN_0056adc0 | crt_static_init_guard_0056adc0 |  |
| 2608 | 0x0056ade0 | FUN_0056ade0 | crt_atexit_register_0056ade0 |  |
| 2609 | 0x0056adf0 | thunk_FUN_0056ae00 | thunk_gui_scale_static_init |  |
| 2610 | 0x0056ae00 | FUN_0056ae00 | gui_scale_static_init | __sinit_g_menu.cpp |
| 2611 | 0x0056ae20 | thunk_FUN_0056ae30 | thunk_gui_mouse_pos_static_init |  |
| 2612 | 0x0056ae30 | FUN_0056ae30 | gui_mouse_pos_static_init | __sinit_g_menu.cpp |
| 2613 | 0x0056ae50 | FUN_0056ae50 | gui_menu_ctor | GM_Menu::{ctor} |
| 2614 | 0x0056aef0 | FUN_0056aef0 | gui_menu_scalar_deleting_dtor | GM_Menu::{dtor} |
| 2615 | 0x0056af10 | FUN_0056af10 | gui_menu_dtor | GM_Menu::{dtor} |
| 2616 | 0x0056af70 | FUN_0056af70 | gui_screen_design_table_load |  |
| 2617 | 0x0056afb0 | FUN_0056afb0 | gui_widget_create_from_design | G_Factory<12GM_Component>::Make |
| 2618 | 0x0056c060 | FUN_0056c060 | stub_ret0_args8_0056c060 |  |
| 2619 | 0x0056c070 | FUN_0056c070 | stub_ret0_args20_0056c070 |  |
| 2620 | 0x0056c080 | FUN_0056c080 | gui_widget_scalar_deleting_dtor | GM_Component::{dtor} |
| 2621 | 0x0056c0a0 | FUN_0056c0a0 | gui_widget_frame_ctor |  |
| 2622 | 0x0056c0f0 | FUN_0056c0f0 | gui_widget_rect_tex_file_scalar_deleting_dtor |  |
| 2623 | 0x0056c110 | FUN_0056c110 | gui_widget_rect_tex_alpha_scalar_deleting_dtor |  |
| 2624 | 0x0056c130 | thunk_FUN_00570ec0 | thunk_gui_widget_rect_tex_file_dtor |  |
| 2625 | 0x0056c140 | FUN_0056c140 | gui_widget_rect_tex_dynamic_scalar_deleting_dtor | GM_RectangleTexture::{dtor} |
| 2626 | 0x0056c160 | FUN_0056c160 | gui_widget_rect_tex_dynamic_dtor | GM_RectangleTexture::{dtor} |
| 2627 | 0x0056c1b0 | FUN_0056c1b0 | gui_widget_scalar_deleting_dtor_common | GM_Component::{dtor} |
| 2628 | 0x0056c1d0 | FUN_0056c1d0 | gui_menu_clone_component_under | GM_Menu::PridejComponentPod |
| 2629 | 0x0056c600 | FUN_0056c600 | gui_menu_add_component_from_template | GM_Menu::PridejComponent |
| 2630 | 0x0056cb80 | FUN_0056cb80 | gui_menu_layout_add_vertical_space |  |
| 2631 | 0x0056cc20 | FUN_0056cc20 | gui_menu_delete_component | GM_Menu::DeleteComponent |
| 2632 | 0x0056cc90 | FUN_0056cc90 | gui_menu_i3d_draw_callback |  |
| 2633 | 0x0056ccc0 | FUN_0056ccc0 | gui_menu_execute_modal | GM_Menu::ExecuteMenu |
| 2634 | 0x0056e3f0 | FUN_0056e3f0 | gui_notice_panel_present_once | GM_QuestionBox::{ctor} |
| 2635 | 0x0056e500 | FUN_0056e500 | gui_menu_build_components_from_design | GM_Menu::CreateCompsByDef |
| 2636 | 0x0056e850 | FUN_0056e850 | gui_menu_free_components |  |
| 2637 | 0x0056e8a0 | FUN_0056e8a0 | gui_menu_destroy | GM_Menu::Destroy |
| 2638 | 0x0056e8f0 | FUN_0056e8f0 | gui_menu_on_close_child | GM_Menu::OnCloseChild |
| 2639 | 0x0056e930 | FUN_0056e930 | gui_menu_close | GM_Menu::MenuClose |
| 2640 | 0x0056e960 | FUN_0056e960 | gui_menu_draw | GM_Menu::Draw |
| 2641 | 0x0056ea70 | FUN_0056ea70 | gui_menu_tick | GM_Menu::Tick |
| 2642 | 0x0056eeb0 | FUN_0056eeb0 | gui_menu_update_hover_focus |  |
| 2643 | 0x0056ef20 | FUN_0056ef20 | gui_menu_activate_by_id | GM_Menu::ActivateByID |
| 2644 | 0x0056ef60 | FUN_0056ef60 | gui_menu_activate_by_id_or_prev | GM_Menu::ActivateByIDOrPrev |
| 2645 | 0x0056efa0 | FUN_0056efa0 | gui_menu_clear_focus |  |
| 2646 | 0x0056efc0 | FUN_0056efc0 | gui_menu_refocus_if_disabled |  |
| 2647 | 0x0056eff0 | FUN_0056eff0 | gui_menu_set_component_text | GM_Menu::SetText |
| 2648 | 0x0056f030 | FUN_0056f030 | gui_menu_set_component_string | GM_Menu::SetString |
| 2649 | 0x0056f070 | FUN_0056f070 | gui_menu_get_component_enabled | GM_Menu::GetDisabled |
| 2650 | 0x0056f0a0 | FUN_0056f0a0 | gui_menu_set_component_enabled | GM_Menu::SetDisabled |
| 2651 | 0x0056f100 | FUN_0056f100 | gui_menu_set_component_visible | GM_Menu::SetHidden |
| 2652 | 0x0056f160 | FUN_0056f160 | gui_menu_get_component_text_ptr | GM_Menu::GetText |
| 2653 | 0x0056f190 | FUN_0056f190 | gui_menu_set_component_text_ptr | GM_Menu::SetText |
| 2654 | 0x0056f1c0 | FUN_0056f1c0 | gui_menu_get_component_float | GM_Menu::GetFloat |
| 2655 | 0x0056f1f0 | FUN_0056f1f0 | gui_menu_set_component_float | GM_Menu::SetFloat |
| 2656 | 0x0056f220 | FUN_0056f220 | gui_menu_get_component_pointer |  |
| 2657 | 0x0056f250 | FUN_0056f250 | gui_menu_set_component_pointer | GM_Menu::SetPointer |
| 2658 | 0x0056f280 | FUN_0056f280 | gui_menu_get_component_dword | GM_Menu::GetDword |
| 2659 | 0x0056f2b0 | FUN_0056f2b0 | gui_menu_set_component_dword | GM_Menu::SetDword |
| 2660 | 0x0056f2e0 | FUN_0056f2e0 | gui_menu_get_component_text_id | GM_Menu::GetTextID |
| 2661 | 0x0056f310 | FUN_0056f310 | gui_menu_set_component_text_id | GM_Menu::SetTextID |
| 2662 | 0x0056f360 | FUN_0056f360 | gui_menu_force_update_component | GM_Component::ForceUpdate |
| 2663 | 0x0056f390 | FUN_0056f390 | gui_menu_force_update_all | GM_Component::ForceUpdate |
| 2664 | 0x0056f3c0 | FUN_0056f3c0 | gui_menu_tick_components | GM_Menu::Tick |
| 2665 | 0x0056f3f0 | FUN_0056f3f0 | gui_menu_set_component_text_from_option_list |  |
| 2666 | 0x0056f430 | FUN_0056f430 | gui_menu_set_component_option_list | GM_ComboBox::SetPointer |
| 2667 | 0x0056f450 | FUN_0056f450 | gui_menu_find_component_by_id | GM_Menu::FindComponentByID |
| 2668 | 0x0056f480 | FUN_0056f480 | gui_menu_find_component_by_id_prev | GM_Menu::FindComponentByIDPrev |
| 2669 | 0x0056f510 | FUN_0056f510 | gui_menu_find_component_by_id_next | GM_Menu::FindComponentByIDNext |
| 2670 | 0x0056f5a0 | FUN_0056f5a0 | gui_menu_find_prev_selectable_in_order |  |
| 2671 | 0x0056f610 | FUN_0056f610 | gui_menu_find_next_selectable_in_order |  |
| 2672 | 0x0056f680 | FUN_0056f680 | gui_draw_line | Line |
| 2673 | 0x0056f720 | FUN_0056f720 | gui_draw_rect_outline | Rectangle |
| 2674 | 0x0056f880 | FUN_0056f880 | gui_draw_rect_filled | RectangleFilled |
| 2675 | 0x0056f9a0 | FUN_0056f9a0 | gui_draw_rect_filled_partial | RectangleProgress |
| 2676 | 0x0056fad0 | FUN_0056fad0 | gui_draw_rect_gradient_h |  |
| 2677 | 0x0056fbf0 | FUN_0056fbf0 | gui_draw_rect_gradient_v |  |
| 2678 | 0x0056fd10 | FUN_0056fd10 | gui_draw_rect_double_bevel | RectangleBB |
| 2679 | 0x005701f0 | FUN_005701f0 | gui_draw_triangle_dir | Triangle |
| 2680 | 0x00570400 | FUN_00570400 | gui_set_clip_rect | SetViewport |
| 2681 | 0x005704b0 | FUN_005704b0 | gui_widget_ctor | GM_Component::{ctor} |
| 2682 | 0x00570580 | FUN_00570580 | gui_widget_copy_ctor_below | GM_Component::{ctor} |
| 2683 | 0x005705f0 | FUN_005705f0 | gui_widget_is_point_inside | GM_Component::IsOverActive |
| 2684 | 0x00570680 | FUN_00570680 | gui_widget_window_draw |  |
| 2685 | 0x00570740 | FUN_00570740 | gui_widget_window_titled_draw | GM_Window::Draw |
| 2686 | 0x005708f0 | FUN_005708f0 | gui_widget_window_drag_mouse |  |
| 2687 | 0x00570950 | FUN_00570950 | gui_widget_frame_draw |  |
| 2688 | 0x00570a90 | FUN_00570a90 | gui_widget_window_scroll_draw | GM_WindowScroll::Draw |
| 2689 | 0x00570b10 | FUN_00570b10 | gui_widget_window_scroll_mouse |  |
| 2690 | 0x00570b50 | FUN_00570b50 | gui_widget_window_scroll_tick | GM_WindowScroll::Tick |
| 2691 | 0x00570c10 | FUN_00570c10 | gui_widget_window_scroll_page_up |  |
| 2692 | 0x00570c20 | FUN_00570c20 | gui_widget_window_scroll_page_down |  |
| 2693 | 0x00570c30 | FUN_00570c30 | gui_widget_window_scroll_by |  |
| 2694 | 0x00570c40 | FUN_00570c40 | gui_widget_window_scroll_to_show | GM_WindowScroll::Tick |
| 2695 | 0x00570c90 | FUN_00570c90 | gui_widget_clip_draw | GM_Orezavac::Draw |
| 2696 | 0x00570d40 | FUN_00570d40 | gui_widget_viewport_draw |  |
| 2697 | 0x00570d80 | FUN_00570d80 | gui_widget_rect_outline_draw | GM_Rectangle::Draw |
| 2698 | 0x00570e30 | FUN_00570e30 | gui_widget_rect_tex_draw |  |
| 2699 | 0x00570e90 | FUN_00570e90 | gui_widget_rect_tex_file_set_text |  |
| 2700 | 0x00570ec0 | FUN_00570ec0 | gui_widget_rect_tex_file_dtor |  |
| 2701 | 0x00570f20 | FUN_00570f20 | gui_widget_rect_tex_alpha_set_text |  |
| 2702 | 0x00570f50 | FUN_00570f50 | gui_widget_rect_gradient_v_draw |  |
| 2703 | 0x00570fa0 | FUN_00570fa0 | gui_widget_rect_gradient_h_draw |  |
| 2704 | 0x00570ff0 | FUN_00570ff0 | gui_widget_rect_tex_dynamic_draw | GM_RectangleTexture::Draw |
| 2705 | 0x00571030 | FUN_00571030 | gui_widget_rect_tex_dynamic_force_update |  |
| 2706 | 0x00571070 | FUN_00571070 | gui_widget_rect_tex_dynamic_set_text | GM_RectangleTexture::SetString |
| 2707 | 0x005710b0 | FUN_005710b0 | gui_widget_line_draw | GM_Line::Draw |
| 2708 | 0x005710f0 | FUN_005710f0 | gui_widget_resize_grip_draw |  |
| 2709 | 0x00571170 | FUN_00571170 | gui_widget_resize_grip_mouse |  |
| 2710 | 0x00571250 | FUN_00571250 | gui_widget_item_draw | GM_MenuItem::Draw |
| 2711 | 0x005712c0 | FUN_005712c0 | gui_widget_item_buffered_draw | GM_MenuItemMem::Draw |
| 2712 | 0x00571330 | FUN_00571330 | gui_widget_item_num_draw | GM_MenuItemNum::Draw |
| 2713 | 0x00571510 | FUN_00571510 | gui_widget_item_num_key | GM_MenuItemNum::Key |
| 2714 | 0x00571560 | FUN_00571560 | gui_widget_item_num_mouse |  |
| 2715 | 0x005715a0 | FUN_005715a0 | gui_widget_select_item_ctor |  |
| 2716 | 0x005715d0 | FUN_005715d0 | gui_widget_select_item_draw |  |
| 2717 | 0x005716a0 | FUN_005716a0 | gui_widget_select_item_key |  |
| 2718 | 0x005716d0 | FUN_005716d0 | gui_widget_select_item_mouse |  |
| 2719 | 0x00571700 | FUN_00571700 | gui_widget_race_round_ctor | GM_RaceRoundItem::{ctor} |
| 2720 | 0x00571770 | FUN_00571770 | gui_widget_race_round_force_update | GM_RaceRoundItem::SetBool |
| 2721 | 0x005717c0 | FUN_005717c0 | gui_widget_race_round_draw | GM_RaceRoundItem::Draw |
| 2722 | 0x005718d0 | FUN_005718d0 | gui_widget_text_ctor | GM_StaticText::{ctor} |
| 2723 | 0x00571910 | FUN_00571910 | gui_widget_text_left_draw | GM_StaticText::Draw |
| 2724 | 0x00571960 | FUN_00571960 | gui_widget_text_center_draw | GM_StaticText::Draw |
| 2725 | 0x005719b0 | FUN_005719b0 | gui_widget_text_right_draw | GM_StaticText::Draw |
| 2726 | 0x00571a00 | FUN_00571a00 | gui_widget_text_buffered_draw | GM_StaticTextMem::Draw |
| 2727 | 0x00571ad0 | FUN_00571ad0 | gui_widget_text_buffered_set_text | GM_StaticTextMem::SetText |
| 2728 | 0x00571b00 | FUN_00571b00 | gui_widget_text_num_force_update | GM_StaticTextNum::SetDword |
| 2729 | 0x00571b30 | FUN_00571b30 | gui_widget_text_num_draw | GM_StaticTextNum::Draw |
| 2730 | 0x00571c00 | FUN_00571c00 | gui_widget_text_time_force_update | GM_StaticTextTime::SetDword |
| 2731 | 0x00571c70 | FUN_00571c70 | gui_widget_text_time_draw | GM_StaticTextTime::Draw |
| 2732 | 0x00571d00 | FUN_00571d00 | gui_widget_button_center_draw | GM_Button::Draw |
| 2733 | 0x00571d70 | FUN_00571d70 | gui_widget_button_left_draw | GM_Button::Draw |
| 2734 | 0x00571de0 | FUN_00571de0 | gui_widget_button_right_draw | GM_Button::Draw |
| 2735 | 0x00571e50 | FUN_00571e50 | gui_widget_button_highlight_draw |  |
| 2736 | 0x00571ec0 | FUN_00571ec0 | gui_widget_button_right_white_draw |  |
| 2737 | 0x00571f20 | FUN_00571f20 | gui_widget_triangle_draw | GM_Triangle::Draw |
| 2738 | 0x00571f60 | FUN_00571f60 | gui_widget_activate_on_hover_mouse |  |
| 2739 | 0x00571f80 | FUN_00571f80 | gui_widget_checkbox_draw | GM_CheckBox::Draw |
| 2740 | 0x005720f0 | FUN_005720f0 | gui_widget_checkbox_key | GM_CheckBox::Key |
| 2741 | 0x00572110 | FUN_00572110 | gui_widget_checkbox_mouse |  |
| 2742 | 0x00572130 | FUN_00572130 | gui_widget_combo_draw | GM_ComboBox::Draw |
| 2743 | 0x005722e0 | FUN_005722e0 | gui_widget_combo_key | GM_ComboBox::Key |
| 2744 | 0x00572390 | FUN_00572390 | gui_widget_combo_mouse |  |
| 2745 | 0x00572430 | FUN_00572430 | gui_widget_combo_range_ctor |  |
| 2746 | 0x00572690 | FUN_00572690 | gui_widget_combo_range_scalar_deleting_dtor |  |
| 2747 | 0x005726b0 | FUN_005726b0 | gui_widget_combo_range_dtor |  |
| 2748 | 0x00572700 | FUN_00572700 | gui_widget_combo_range_force_update |  |
| 2749 | 0x00572720 | FUN_00572720 | gui_widget_combo_range_key | GM_ComboBox::Key |
| 2750 | 0x005727d0 | FUN_005727d0 | gui_widget_combo_range_mouse |  |
| 2751 | 0x00572870 | FUN_00572870 | gui_widget_slider_float_draw | GM_Slider::Draw |
| 2752 | 0x00572b10 | FUN_00572b10 | gui_widget_slider_float_key | GM_Slider::Key |
| 2753 | 0x00572b80 | FUN_00572b80 | gui_widget_slider_float_mouse |  |
| 2754 | 0x00572c90 | FUN_00572c90 | gui_widget_slider_num_draw | GM_SliderNum::Draw |
| 2755 | 0x00572e70 | FUN_00572e70 | gui_widget_slider_num_key | GM_SliderNum::Key |
| 2756 | 0x00572f10 | FUN_00572f10 | gui_widget_slider_num_mouse |  |
| 2757 | 0x00572ff0 | FUN_00572ff0 | gui_widget_slider_int_draw | GM_SliderInteger::Draw |
| 2758 | 0x005731b0 | FUN_005731b0 | gui_widget_slider_int_key | GM_SliderInteger::Key |
| 2759 | 0x00573230 | FUN_00573230 | gui_widget_slider_int_mouse |  |
| 2760 | 0x005733d0 | FUN_005733d0 | gui_widget_scrollbar_draw |  |
| 2761 | 0x00573490 | FUN_00573490 | gui_widget_scrollbar_mouse |  |
| 2762 | 0x00573510 | FUN_00573510 | gui_widget_progress_bar_draw |  |
| 2763 | 0x00573580 | FUN_00573580 | gui_widget_progress_bar_track_draw | GM_ProgressBar::Draw |
| 2764 | 0x00573600 | FUN_00573600 | gui_widget_axis_range_draw |  |
| 2765 | 0x00573860 | FUN_00573860 | gui_widget_axis_range_mouse |  |
| 2766 | 0x005739f0 | FUN_005739f0 | gui_widget_key_binding_ctor |  |
| 2767 | 0x00573b50 | FUN_00573b50 | gui_widget_key_binding_draw |  |
| 2768 | 0x00573c70 | FUN_00573c70 | gui_widget_key_binding_force_update |  |
| 2769 | 0x00573da0 | FUN_00573da0 | gui_widget_key_binding_key |  |
| 2770 | 0x00573f00 | FUN_00573f00 | gui_widget_key_binding_mouse |  |
| 2771 | 0x00573f80 | FUN_00573f80 | gui_widget_save_slot_ctor |  |
| 2772 | 0x00574180 | FUN_00574180 | gui_widget_save_slot_scalar_deleting_dtor |  |
| 2773 | 0x005741a0 | FUN_005741a0 | gui_widget_save_slot_copy_ctor |  |
| 2774 | 0x00574350 | FUN_00574350 | gui_widget_save_slot_dtor |  |
| 2775 | 0x005743a0 | FUN_005743a0 | gui_widget_save_slot_draw |  |
| 2776 | 0x00574560 | FUN_00574560 | gui_widget_save_slot_set_text |  |
| 2777 | 0x005745b0 | FUN_005745b0 | gui_widget_net_player_ctor | GM_NetPlayer::{ctor} |
| 2778 | 0x00574680 | FUN_00574680 | gui_widget_net_player_scalar_deleting_dtor |  |
| 2779 | 0x005746a0 | FUN_005746a0 | gui_widget_net_player_dtor | GM_NetPlayer::{dtor} |
| 2780 | 0x005746b0 | FUN_005746b0 | gui_widget_net_player_draw | GM_NetPlayer::Draw |
| 2781 | 0x005749f0 | FUN_005749f0 | gui_widget_net_player_is_point_inside |  |
| 2782 | 0x00574a40 | FUN_00574a40 | stub_noop_cdecl_00574a40 |  |
| 2783 | 0x00574a50 | FUN_00574a50 | gui_widget_key_activate | GM_Button::Key |
| 2784 | 0x00574a70 | FUN_00574a70 | gui_widget_click_activate |  |
| 2785 | 0x00574a90 | FUN_00574a90 | gui_widget_net_player_set_row |  |
| 2786 | 0x00574b00 | FUN_00574b00 | gui_widget_net_player_get_pointer | GM_NetPlayer::GetPointer |
| 2787 | 0x00574b10 | FUN_00574b10 | gui_widget_edit_text_ctor | GM_TextEdit::{ctor} |
| 2788 | 0x00574b70 | FUN_00574b70 | gui_widget_edit_text_key_max10 | GM_TextEdit::Key |
| 2789 | 0x00574cf0 | FUN_00574cf0 | gui_widget_edit_text_set_text | GM_TextEdit::SetText |
| 2790 | 0x00574d30 | FUN_00574d30 | gui_widget_edit_text_get_text | GM_TextEdit::GetText |
| 2791 | 0x00574d60 | FUN_00574d60 | gui_widget_edit_text_draw | GM_TextEdit::Draw |
| 2792 | 0x00574e20 | FUN_00574e20 | gui_widget_edit_text_key_max31 | GM_TextEdit::Key |
| 2793 | 0x00574fa0 | FUN_00574fa0 | gui_widget_edit_number_ctor | GM_EditNum::{ctor} |
| 2794 | 0x00575000 | FUN_00575000 | gui_widget_edit_number_draw | GM_EditNum::Draw |
| 2795 | 0x005750a0 | FUN_005750a0 | gui_widget_edit_number_key | GM_EditNum::Key |
| 2796 | 0x00575270 | FUN_00575270 | gui_widget_edit_number_set_value | GM_EditNum::SetDword |
| 2797 | 0x005752b0 | FUN_005752b0 | gui_widget_edit_number_get_value | GM_EditNum::GetDword |
| 2798 | 0x005752d0 | FUN_005752d0 | gui_format_ip_address |  |
| 2799 | 0x00575350 | FUN_00575350 | gui_widget_edit_ip_ctor |  |
| 2800 | 0x005753b0 | FUN_005753b0 | gui_widget_edit_ip_draw |  |
| 2801 | 0x00575540 | FUN_00575540 | gui_widget_edit_ip_key |  |
| 2802 | 0x005756f0 | FUN_005756f0 | gui_widget_text_block_ctor | GM_StaticTextBig::{ctor} |
| 2803 | 0x00575780 | FUN_00575780 | gui_widget_text_block_draw | GM_StaticTextBig::Draw |
| 2804 | 0x005757f0 | FUN_005757f0 | stub_ret0_args20_005757f0 |  |
| 2805 | 0x00575800 | FUN_00575800 | gui_widget_text_block_clear | GM_StaticTextBig::Clear |
| 2806 | 0x00575810 | FUN_00575810 | gui_widget_text_block_add_text | GM_StaticTextBig::AddText |
| 2807 | 0x005758d0 | FUN_005758d0 | gui_widget_car_carousel_ctor | GM_ListCars::{ctor} |
| 2808 | 0x00575920 | FUN_00575920 | gui_widget_dtor | GM_Component::{dtor} |
| 2809 | 0x00575930 | FUN_00575930 | gui_widget_car_carousel_draw | GM_ListCars::Draw |
| 2810 | 0x00575a60 | FUN_00575a60 | stub_ret0_args8_00575a60 |  |
| 2811 | 0x00575a70 | FUN_00575a70 | gui_widget_car_carousel_mouse |  |
| 2812 | 0x00575b50 | FUN_00575b50 | gui_widget_car_carousel_tick | GM_ListCars::Tick |
| 2813 | 0x00575be0 | FUN_00575be0 | gui_widget_car_carousel_set_list | GM_ListCars::SetPointer |
| 2814 | 0x00575bf0 | FUN_00575bf0 | gui_widget_car_carousel_set_index | GM_ListCars::SetDword |
| 2815 | 0x00575c00 | FUN_00575c00 | stub_noop_args8_00575c00 |  |
| 2816 | 0x00575c10 | FUN_00575c10 | stl_construct_dword |  |
| 2817 | 0x00575c20 | FUN_00575c20 | crt_static_init_guard_00575c20 |  |
| 2818 | 0x00575c40 | FUN_00575c40 | crt_atexit_register_00575c40 |  |
| 2819 | 0x00575c50 | FUN_00575c50 | gui_combo_dropdown_screen_ctor | GM_ComboMenu::{ctor} |
| 2820 | 0x00575ea0 | FUN_00575ea0 | gui_combo_dropdown_tick | GM_ComboMenu::OnClick |
| 2821 | 0x00575f50 | FUN_00575f50 | gui_notice_panel_ctor |  |
| 2822 | 0x005761d0 | FUN_005761d0 | gui_notice_panel_ctor_from_strings |  |
| 2823 | 0x00576450 | FUN_00576450 | gui_notice_panel_tick |  |
| 2824 | 0x005764c0 | FUN_005764c0 | gui_question_box_ctor | GM_QuestionBox::{ctor} |
| 2825 | 0x00576560 | FUN_00576560 | gui_question_box_tick | GM_QuestionBox::OnClick |
| 2826 | 0x00576620 | FUN_00576620 | gui_message_box_ctor |  |
| 2827 | 0x005767a0 | FUN_005767a0 | gui_message_box_tick |  |
| 2828 | 0x00576810 | FUN_00576810 | gui_main_menu_exit_box_ctor | GM_MainMenuExit::OnCreate |
| 2829 | 0x00576890 | FUN_00576890 | gui_main_menu_exit_box_tick | GM_MainMenuExit::OnClick |
| 2830 | 0x00576920 | FUN_00576920 | gui_main_menu_ctor | GM_MainMenu::OnCreate |
| 2831 | 0x00576a20 | FUN_00576a20 | gui_main_menu_tick | GM_MainMenu::OnClick |
| 2832 | 0x00576cc0 | FUN_00576cc0 | gui_game_menu_exit_box_ctor | GM_GameMenuExit::OnCreate |
| 2833 | 0x00576d40 | FUN_00576d40 | gui_game_menu_exit_box_tick | GM_GameMenuExit::OnClick |
| 2834 | 0x00576dd0 | FUN_00576dd0 | gui_game_menu_ctor | GM_GameMenu::OnCreate |
| 2835 | 0x00576e30 | FUN_00576e30 | gui_game_menu_on_exchange |  |
| 2836 | 0x00576e60 | FUN_00576e60 | gui_game_menu_tick | GM_GameMenu::OnClick |
| 2837 | 0x00576fe0 | FUN_00576fe0 | gui_options_audio_video_ctor |  |
| 2838 | 0x005771a0 | thunk_FUN_0056e8f0 | thunk_gui_menu_on_close_child |  |
| 2839 | 0x005771b0 | FUN_005771b0 | gui_options_audio_video_tick |  |
| 2840 | 0x00577990 | FUN_00577990 | gui_options_ctor | GM_Options::OnCreate |
| 2841 | 0x00577a20 | FUN_00577a20 | gui_options_tick | GM_Options::OnClick |
| 2842 | 0x00577b10 | FUN_00577b10 | gui_options_controls_ctor | GM_OptionsControls::OnCreate |
| 2843 | 0x00577b90 | FUN_00577b90 | gui_options_controls_on_exchange | GM_OptionsControls::SetUpTexts |
| 2844 | 0x00577d20 | FUN_00577d20 | gui_options_controls_close |  |
| 2845 | 0x00577d50 | FUN_00577d50 | gui_options_controls_tick | GM_OptionsControls::OnClick |
| 2846 | 0x005782e0 | FUN_005782e0 | gui_key_capture_ctor |  |
| 2847 | 0x00578370 | FUN_00578370 | gui_key_capture_on_close_child |  |
| 2848 | 0x005783a0 | FUN_005783a0 | gui_key_capture_tick |  |
| 2849 | 0x005786e0 | FUN_005786e0 | gui_key_conflict_ctor |  |
| 2850 | 0x00578790 | FUN_00578790 | gui_menu_scalar_deleting_dtor_common | GM_Menu::{dtor} |
| 2851 | 0x005787b0 | thunk_FUN_0056af10 | thunk_gui_menu_dtor |  |
| 2852 | 0x005787c0 | FUN_005787c0 | gui_key_conflict_tick |  |
| 2853 | 0x00578940 | FUN_00578940 | gui_inventory_format_slot_row |  |
| 2854 | 0x00578a30 | FUN_00578a30 | gui_inventory_ctor |  |
| 2855 | 0x00578d10 | FUN_00578d10 | gui_inventory_on_close_child |  |
| 2856 | 0x00578d90 | FUN_00578d90 | gui_inventory_tick |  |
| 2857 | 0x00579020 | FUN_00579020 | gui_action_menu_ctor | GM_ItemPickUp::OnCreate |
| 2858 | 0x00579480 | FUN_00579480 | gui_action_menu_scalar_deleting_dtor |  |
| 2859 | 0x005794a0 | FUN_005794a0 | gui_action_menu_dtor | GM_ItemPickUp::{dtor} |
| 2860 | 0x005794d0 | FUN_005794d0 | gui_action_menu_pick_up | GM_ItemPickUp::PickUp |
| 2861 | 0x00579590 | FUN_00579590 | gui_action_menu_on_close_child |  |
| 2862 | 0x00579610 | FUN_00579610 | gui_action_menu_tick | GM_ItemPickUp::OnKey |
| 2863 | 0x00579900 | FUN_00579900 | gui_profile_select_ctor | GM_ProfileSelect::OnCreate |
| 2864 | 0x005799d0 | FUN_005799d0 | gui_profile_select_on_exchange |  |
| 2865 | 0x00579a00 | FUN_00579a00 | gui_profile_select_build_rows | GM_ProfileSelect::OnCreate |
| 2866 | 0x00579bf0 | FUN_00579bf0 | gui_profile_select_on_close_child | GM_ProfileSelect::OnCloseChild |
| 2867 | 0x00579cc0 | FUN_00579cc0 | gui_profile_select_tick | GM_ProfileSelect::OnClick |
| 2868 | 0x00579fb0 | FUN_00579fb0 | gui_profile_create_ctor | GM_ProfileCreate::OnCreate |
| 2869 | 0x0057a070 | FUN_0057a070 | gui_profile_create_scalar_deleting_dtor | GM_ProfileCreate::{dtor} |
| 2870 | 0x0057a090 | FUN_0057a090 | gui_profile_create_dtor | GM_ProfileCreate::{dtor} |
| 2871 | 0x0057a0f0 | FUN_0057a0f0 | gui_profile_create_tick | GM_ProfileCreate::OnClick |
| 2872 | 0x0057a1f0 | FUN_0057a1f0 | gui_profile_delete_ctor | GM_ProfileDelete::OnCreate |
| 2873 | 0x0057a370 | FUN_0057a370 | gui_profile_delete_on_close_child | GM_ProfileDelete::OnCloseChild |
| 2874 | 0x0057a3c0 | FUN_0057a3c0 | gui_profile_delete_tick | GM_ProfileDelete::OnClick |
| 2875 | 0x0057a4a0 | FUN_0057a4a0 | gui_load_game_ctor | GM_LoadGame::OnCreate |
| 2876 | 0x0057a670 | FUN_0057a670 | gui_load_game_tick | GM_LoadGame::OnClick |
| 2877 | 0x0057aaf0 | FUN_0057aaf0 | gui_free_ride_ctor | GM_FreeRide::OnCreate |
| 2878 | 0x0057ac40 | FUN_0057ac40 | gui_free_ride_on_close_child | GM_FreeRide::OnCloseChild |
| 2879 | 0x0057ad80 | FUN_0057ad80 | gui_free_ride_tick | GM_FreeRide::OnClick |
| 2880 | 0x0057aff0 | FUN_0057aff0 | gui_free_ride_location_ctor |  |
| 2881 | 0x0057b0b0 | FUN_0057b0b0 | gui_free_ride_location_tick |  |
| 2882 | 0x0057b150 | FUN_0057b150 | gui_session_list_ctor |  |
| 2883 | 0x0057b2e0 | FUN_0057b2e0 | gui_session_list_tick |  |
| 2884 | 0x0057b3a0 | FUN_0057b3a0 | gui_gun_shop_ctor | GM_GunShop::OnCreate |
| 2885 | 0x0057b530 | FUN_0057b530 | gui_gun_shop_format_slot_row |  |
| 2886 | 0x0057b600 | FUN_0057b600 | gui_gun_shop_refresh_slots | GM_GunShop::RefreshInventory |
| 2887 | 0x0057b700 | FUN_0057b700 | gui_gun_shop_tick | GM_GunShop::OnClick |
| 2888 | 0x0057b930 | FUN_0057b930 | gui_garage_select_ctor | GM_GarageSelect::OnCreate |
| 2889 | 0x0057bd10 | FUN_0057bd10 | gui_garage_select_tick | GM_GarageSelect::OnClick |
| 2890 | 0x0057bfa0 | FUN_0057bfa0 | gui_garage_full_ctor | GM_GarageFull::OnCreate |
| 2891 | 0x0057c340 | FUN_0057c340 | gui_garage_full_tick | GM_GarageFull::OnClick |
| 2892 | 0x0057c540 | FUN_0057c540 | gui_garage_summary_ctor | GM_GarageSummary::OnCreate |
| 2893 | 0x0057c880 | FUN_0057c880 | gui_garage_summary_tick | GM_GarageSummary::OnKey |
| 2894 | 0x0057c8e0 | FUN_0057c8e0 | gui_game_over_ctor | GM_GameFailed::OnCreate |
| 2895 | 0x0057ca40 | FUN_0057ca40 | gui_game_over_tick | GM_GameFailed::OnClick |
| 2896 | 0x0057cab0 | FUN_0057cab0 | gui_racing_game_menu_ctor | GM_GameMenuRacing::OnCreate |
| 2897 | 0x0057cb40 | FUN_0057cb40 | gui_racing_game_menu_tick | GM_GameMenuRacing::OnClick |
| 2898 | 0x0057cc90 | FUN_0057cc90 | gui_load_progress_ctor | GM_LoadProgress::OnCreate |
| 2899 | 0x0057cf80 | FUN_0057cf80 | gui_load_progress_on_exchange |  |
| 2900 | 0x0057d0c0 | FUN_0057d0c0 | gui_load_progress_tick | GM_LoadProgress::OnTick |
| 2901 | 0x0057d240 | FUN_0057d240 | gui_load_progress_set_progress_game_init | GM_LoadProgress::SetProgressGameInit |
| 2902 | 0x0057d320 | FUN_0057d320 | gui_load_progress_set_progress_mission_open | GM_LoadProgress::SetProgressMissionOpen |
| 2903 | 0x0057d400 | FUN_0057d400 | gui_load_progress_set_caption |  |
| 2904 | 0x0057d420 | FUN_0057d420 | gui_load_progress_present | GM_LoadProgress::Present |
| 2905 | 0x0057d4b0 | FUN_0057d4b0 | gui_free_ride_mode_ctor |  |
| 2906 | 0x0057d580 | FUN_0057d580 | gui_free_ride_mode_tick |  |
| 2907 | 0x0057d6e0 | FUN_0057d6e0 | crt_static_init_guard_0057d6e0 |  |
| 2908 | 0x0057d700 | FUN_0057d700 | crt_atexit_register_0057d700 |  |
| 2909 | 0x0057d710 | FUN_0057d710 | crt_static_init_textdb |  |
| 2910 | 0x0057d720 | FUN_0057d720 | crt_zero_init_textdb |  |
| 2911 | 0x0057d730 | FUN_0057d730 | crt_atexit_register_textdb |  |
| 2912 | 0x0057d740 | FUN_0057d740 | crt_atexit_free_textdb |  |
| 2913 | 0x0057d750 | FUN_0057d750 | textdb_ctor | G_TextDatabase::{ctor} |
| 2914 | 0x0057d760 | FUN_0057d760 | textdb_dtor | G_TextDatabase::{dtor} |
| 2915 | 0x0057d770 | FUN_0057d770 | textdb_load | G_TextDatabase::LoadDatabase |
| 2916 | 0x0057d820 | FUN_0057d820 | textdb_unload | G_TextDatabase::Unload |
| 2917 | 0x0057d850 | FUN_0057d850 | textdb_get_string_by_id | G_TextDatabase::GetText |
| 2918 | 0x0057d8b0 | FUN_0057d8b0 | textdb_get_string_by_id_or_null | G_TextDatabase::GetTextOrNULL |
| 2919 | 0x0057d910 | FUN_0057d910 | crt_static_init_guard_0057d910 |  |
| 2920 | 0x0057d930 | FUN_0057d930 | crt_atexit_register_0057d930 |  |
| 2921 | 0x0057d940 | FUN_0057d940 | crt_static_init_guard_0057d940 |  |
| 2922 | 0x0057d960 | FUN_0057d960 | crt_atexit_register_0057d960 |  |
| 2923 | 0x0057d970 | FUN_0057d970 | crt_static_init_collision |  |
| 2924 | 0x0057d980 | FUN_0057d980 | collision_construct_singleton |  |
| 2925 | 0x0057d990 | FUN_0057d990 | crt_atexit_register_collision |  |
| 2926 | 0x0057d9a0 | FUN_0057d9a0 | crt_atexit_free_collision |  |
| 2927 | 0x0057d9b0 | FUN_0057d9b0 | collision_dtor | g_collision::{dtor} |
| 2928 | 0x0057da00 | FUN_0057da00 | collision_free_level_data | g_collision::DeleteCollsLevel |
| 2929 | 0x0057dab0 | FUN_0057dab0 | collision_classify_static_primitives | g_collision::LoadCollision |
| 2930 | 0x0057dca0 | FUN_0057dca0 | collision_load_level | g_collision::LoadCollision |
| 2931 | 0x0057e740 | FUN_0057e740 | collision_dyn_optimize_tick | g_collision::DynOptimizeTick |
| 2932 | 0x0057e7f0 | FUN_0057e7f0 | collision_dyn_insert | g_collision::DynInsert |
| 2933 | 0x0057e9e0 | FUN_0057e9e0 | collision_dyn_erase | g_collision::DynErase |
| 2934 | 0x0057ebf0 | FUN_0057ebf0 | collision_dyn_update | g_collision::DynUpdate |
| 2935 | 0x0057ee40 | FUN_0057ee40 | collision_dyn_set_next_owner | g_collision::DynSetNextCollOwner |
| 2936 | 0x0057ee90 | FUN_0057ee90 | collision_dyn_set_next_multiple_owners | g_collision::DynSetNextCollOwnerMult |
| 2937 | 0x0057eed0 | FUN_0057eed0 | collision_dyn_clear_all | g_collision::DynWipeOut |
| 2938 | 0x0057f150 | FUN_0057f150 | collision_dyn_take_out_to_list | g_collision::DynGetIn |
| 2939 | 0x0057f300 | FUN_0057f300 | collision_dyn_put_back_from_list | g_collision::DynGetOut |
| 2940 | 0x0057f360 | FUN_0057f360 | collision_box_in_dynamics | g_collision::BoxInDynamic |
| 2941 | 0x0057f5e0 | FUN_0057f5e0 | collision_box_in_primitive_volume | g_collision::BoxInNonFace |
| 2942 | 0x0057f7f0 | FUN_0057f7f0 | collision_box_in_tree_level | g_collision::BoxInTreeLevel |
| 2943 | 0x0057f9e0 | FUN_0057f9e0 | collision_test_moving_box | g_collision::_TestDBox |
| 2944 | 0x005800d0 | FUN_005800d0 | collision_ctor | g_collision::{dtor} |
| 2945 | 0x00580140 | FUN_00580140 | crt_static_init_collision_line |  |
| 2946 | 0x00580160 | FUN_00580160 | crt_atexit_register_collision_line |  |
| 2947 | 0x00580170 | FUN_00580170 | collision_line_in_dynamics | g_collision::LineInDynamics |
| 2948 | 0x005806b0 | FUN_005806b0 | collision_line_in_dynamics_inner | g_collision::LineInDynamicsInnerDyn |
| 2949 | 0x00580c50 | FUN_00580c50 | collision_line_vs_aabb | g_collision::LineInAABox |
| 2950 | 0x00580ff0 | FUN_00580ff0 | collision_line_vs_aabb_from_inside | g_collision::LineInAABoxInverted |
| 2951 | 0x00581210 | FUN_00581210 | collision_line_in_primitive_volume | g_collision::LineInNonface |
| 2952 | 0x005817d0 | FUN_005817d0 | collision_line_in_tree_level | g_collision::LineInTreeLevel |
| 2953 | 0x00581c40 | FUN_00581c40 | collision_test_line_vertical | g_collision::_TestLineV |
| 2954 | 0x00582120 | FUN_00582120 | collision_test_line_vertical_inner_dyn | g_collision::_TestLineVInnerDyn |
| 2955 | 0x00582600 | FUN_00582600 | collision_test_line_horizontal_static | g_collision::TestLineHStatic |
| 2956 | 0x00583570 | FUN_00583570 | collision_test_line_horizontal_dynamic | g_collision::TestLineHDynamic |
| 2957 | 0x005838a0 | FUN_005838a0 | collision_test_line_horizontal | g_collision::_TestLineH |
| 2958 | 0x005848e0 | FUN_005848e0 | collision_line_in_vehicle | g_collision::LineInVehicle |
| 2959 | 0x00584cb0 | FUN_00584cb0 | collision_test_line_in_vehicle | g_collision::TestLineInVehicle |
| 2960 | 0x00584f70 | FUN_00584f70 | crt_static_init_collision_collect |  |
| 2961 | 0x00584f90 | FUN_00584f90 | crt_atexit_register_collision_collect |  |
| 2962 | 0x00584fa0 | FUN_00584fa0 | collision_collect_primitive_volume_in_sphere | g_collision::LocalityInNonface |
| 2963 | 0x005854a0 | FUN_005854a0 | collision_collect_tree_level_in_sphere | g_collision::LocalityInTreeLevel |
| 2964 | 0x00585510 | FUN_00585510 | collision_collect_static_hits_in_sphere | g_collision::FindStaticCollsInLocality |
| 2965 | 0x005856c0 | FUN_005856c0 | collision_find_hits_for_frame | g_collision::FindCollsByFrame |
| 2966 | 0x00585890 | FUN_00585890 | collision_find_dynamic_line_hits | g_collision::FindDynamicCollsInLine |
| 2967 | 0x00586080 | FUN_00586080 | crt_static_init_collision_sphere |  |
| 2968 | 0x005860a0 | FUN_005860a0 | crt_atexit_register_collision_sphere |  |
| 2969 | 0x005860b0 | FUN_005860b0 | collision_sphere_in_dynamics | g_collision::SphereInDynamics |
| 2970 | 0x00586fd0 | FUN_00586fd0 | collision_sphere_vs_edge | g_collision::SphereInEdge |
| 2971 | 0x00587570 | FUN_00587570 | collision_sphere_vs_triangle | g_collision::SphereInFace |
| 2972 | 0x00588140 | FUN_00588140 | collision_sphere_vs_aabb | g_collision::SphereInAABox |
| 2973 | 0x0058c6a0 | FUN_0058c6a0 | collision_sphere_in_primitive_volume | g_collision::SphereInNonface |
| 2974 | 0x0058e970 | FUN_0058e970 | collision_sphere_in_tree_level | g_collision::SphereInTreeLevel |
| 2975 | 0x0058eaa0 | FUN_0058eaa0 | collision_test_sphere_swept | g_collision::_TestSphereB |
| 2976 | 0x0058f670 | FUN_0058f670 | collision_slide_sphere |  |
| 2977 | 0x0058fb00 | FUN_0058fb00 | collision_earliest_contact_between_paths | g_collision::TestCurveCurve |
| 2978 | 0x0058fd30 | FUN_0058fd30 | collision_sphere_in_vehicle | g_collision::SphereInVehicle |
| 2979 | 0x00590590 | FUN_00590590 | collision_test_sphere_in_vehicle | g_collision::TestSphereInVehicle |
| 2980 | 0x005909d0 | FUN_005909d0 | crt_static_init_collision_api |  |
| 2981 | 0x005909f0 | FUN_005909f0 | crt_atexit_register_collision_api |  |
| 2982 | 0x00590a00 | FUN_00590a00 | collision_teardown | g_collision::GameDone |
| 2983 | 0x00590a10 | FUN_00590a10 | collision_test_line_vertical_thunk | g_collision::TestLineV |
| 2984 | 0x00590a20 | FUN_00590a20 | collision_test_line_vertical_inner_dyn_thunk | g_collision::TestLineVInnerDyn |
| 2985 | 0x00590a30 | thunk_FUN_005838a0 | thunk_collision_test_line_horizontal |  |
| 2986 | 0x00590a40 | FUN_00590a40 | collision_test_sphere_swept_thunk | g_collision::TestSphereB |
| 2987 | 0x00590a60 | FUN_00590a60 | collision_test_sphere_slide |  |
| 2988 | 0x00590ab0 | thunk_FUN_0057f9e0 | thunk_collision_test_moving_box |  |
| 2989 | 0x00590ac0 | FUN_00590ac0 | collision_test_character_proxy | g_collision::TestPanak2 |
| 2990 | 0x00591530 | FUN_00591530 | collision_test_character_proxy_in_vehicle | g_collision::TestPanak2InVehicle |
| 2991 | 0x00591f20 | FUN_00591f20 | crt_static_init_line_batch |  |
| 2992 | 0x00591f40 | FUN_00591f40 | crt_atexit_register_line_batch |  |
| 2993 | 0x00591f50 | FUN_00591f50 | line_batch_add |  |
| 2994 | 0x00591fe0 | FUN_00591fe0 | line_batch_draw |  |
| 2995 | 0x00592030 | FUN_00592030 | line_batch_reset |  |
| 2996 | 0x00592050 | FUN_00592050 | crt_static_init_def_tables |  |
| 2997 | 0x00592070 | FUN_00592070 | crt_atexit_register_def_tables |  |
| 2998 | 0x00592080 | FUN_00592080 | data_load_def_tables | LoadMaterialsFromBin |
| 2999 | 0x00592240 | FUN_00592240 | data_unload_def_tables | UnloadMaterials |
| 3000 | 0x005922c0 | FUN_005922c0 | crt_static_init_shot_skeleton |  |
| 3001 | 0x005922e0 | FUN_005922e0 | crt_atexit_register_shot_skeleton |  |
| 3002 | 0x005922f0 | FUN_005922f0 | shot_skeleton_init | C_ShotSkeleton::Init |
| 3003 | 0x00592350 | FUN_00592350 | shot_skeleton_test_collision | C_ShotSkeleton::TestCollision |
| 3004 | 0x005929c0 | FUN_005929c0 | crt_static_init_event_msg |  |
| 3005 | 0x005929e0 | FUN_005929e0 | crt_atexit_register_005929e0 |  |
| 3006 | 0x005929f0 | FUN_005929f0 | event_msg_serialize_bytes |  |
| 3007 | 0x00592a40 | FUN_00592a40 | event_msg_serialize_bytes_at |  |
| 3008 | 0x00592a90 | FUN_00592a90 | event_msg_deserialize_bytes |  |
| 3009 | 0x00592ad0 | FUN_00592ad0 | event_msg_deserialize_cstring |  |
| 3010 | 0x00592b10 | FUN_00592b10 | event_msg_serialize_cstring |  |
| 3011 | 0x00592b40 | FUN_00592b40 | event_msg_serialize_header |  |
| 3012 | 0x00592b70 | FUN_00592b70 | event_msg_deserialize_header |  |
| 3013 | 0x00592ba0 | FUN_00592ba0 | event_msg_set_body_from_wire |  |
| 3014 | 0x00592bf0 | FUN_00592bf0 | event_msg_chat_serialize |  |
| 3015 | 0x00592c20 | FUN_00592c20 | event_msg_chat_deserialize |  |
| 3016 | 0x00592c60 | FUN_00592c60 | event_msg_player_list_serialize |  |
| 3017 | 0x00592d10 | FUN_00592d10 | event_msg_player_list_deserialize |  |
| 3018 | 0x00592e20 | FUN_00592e20 | event_msg_player_add_serialize |  |
| 3019 | 0x00592e60 | FUN_00592e60 | event_msg_player_add_deserialize |  |
| 3020 | 0x00592eb0 | FUN_00592eb0 | event_msg_session_start_serialize |  |
| 3021 | 0x00592ed0 | FUN_00592ed0 | event_msg_session_start_deserialize |  |
| 3022 | 0x00592ef0 | FUN_00592ef0 | event_msg_u8_serialize |  |
| 3023 | 0x00592f20 | FUN_00592f20 | event_msg_u8_deserialize |  |
| 3024 | 0x00592f50 | FUN_00592f50 | event_msg_blob8_serialize |  |
| 3025 | 0x00592f80 | FUN_00592f80 | event_msg_blob8_deserialize |  |
| 3026 | 0x00592fb0 | FUN_00592fb0 | event_msg_blob16_serialize |  |
| 3027 | 0x00592fe0 | FUN_00592fe0 | event_msg_blob16_deserialize |  |
| 3028 | 0x00593010 | FUN_00593010 | event_msg_dword_pair_deserialize |  |
| 3029 | 0x00593050 | FUN_00593050 | event_msg_peer_latency_table_serialize |  |
| 3030 | 0x005930c0 | FUN_005930c0 | event_msg_peer_latency_table_deserialize |  |
| 3031 | 0x00593130 | FUN_00593130 | event_msg_actor_guid_table_serialize |  |
| 3032 | 0x005931c0 | FUN_005931c0 | event_msg_actor_state_serialize |  |
| 3033 | 0x00593280 | FUN_00593280 | event_msg_actor_state_deserialize |  |
| 3034 | 0x00593340 | FUN_00593340 | event_msg_bottle_state_serialize |  |
| 3035 | 0x00593370 | FUN_00593370 | event_msg_bottle_state_deserialize |  |
| 3036 | 0x005933b0 | FUN_005933b0 | event_msg_bottle_collision_serialize |  |
| 3037 | 0x005933e0 | FUN_005933e0 | event_msg_bottle_collision_deserialize |  |
| 3038 | 0x00593410 | FUN_00593410 | event_msg_grenade_spawn_serialize |  |
| 3039 | 0x00593440 | FUN_00593440 | event_msg_grenade_spawn_deserialize |  |
| 3040 | 0x00593470 | FUN_00593470 | event_msg_explosion_serialize |  |
| 3041 | 0x005934b0 | FUN_005934b0 | event_msg_explosion_deserialize |  |
| 3042 | 0x00593500 | FUN_00593500 | event_msg_fire_serialize |  |
| 3043 | 0x00593530 | FUN_00593530 | event_msg_fire_deserialize |  |
| 3044 | 0x00593560 | FUN_00593560 | event_msg_physics_state_serialize |  |
| 3045 | 0x005935f0 | FUN_005935f0 | event_msg_physics_state_deserialize |  |
| 3046 | 0x00593690 | FUN_00593690 | event_msg_car_state_serialize |  |
| 3047 | 0x005936d0 | FUN_005936d0 | event_msg_car_state_deserialize |  |
| 3048 | 0x00593720 | FUN_00593720 | event_msg_car_glass_break_serialize |  |
| 3049 | 0x00593770 | FUN_00593770 | event_msg_car_glass_break_deserialize |  |
| 3050 | 0x005937d0 | FUN_005937d0 | hit_event_payload_fill |  |
| 3051 | 0x00593840 | FUN_00593840 | hit_event_payload_serialize |  |
| 3052 | 0x005938c0 | FUN_005938c0 | hit_event_payload_deserialize |  |
| 3053 | 0x00593950 | FUN_00593950 | collision_event_payload_fill |  |
| 3054 | 0x005939c0 | FUN_005939c0 | collision_event_payload_serialize |  |
| 3055 | 0x00593a30 | FUN_00593a30 | collision_event_payload_deserialize |  |
| 3056 | 0x00593aa0 | FUN_00593aa0 | event_msg_dword_byte_serialize |  |
| 3057 | 0x00593ad0 | FUN_00593ad0 | event_msg_dword_byte_deserialize |  |
| 3058 | 0x00593b10 | FUN_00593b10 | event_msg_dword_pair_serialize |  |
| 3059 | 0x00593b40 | FUN_00593b40 | event_msg_car_damage_serialize |  |
| 3060 | 0x00593cf0 | FUN_00593cf0 | event_msg_car_damage_deserialize |  |
| 3061 | 0x00593ef0 | FUN_00593ef0 | event_msg_dword_pair_reversed_serialize |  |
| 3062 | 0x00593f20 | FUN_00593f20 | event_msg_car_collision_damage_serialize |  |
| 3063 | 0x00593fd0 | FUN_00593fd0 | event_msg_car_collision_damage_deserialize |  |
| 3064 | 0x00594230 | FUN_00594230 | dropout_event_payload_ctor |  |
| 3065 | 0x00594290 | FUN_00594290 | dropout_event_payload_init |  |
| 3066 | 0x005942a0 | FUN_005942a0 | dropout_event_payload_dtor |  |
| 3067 | 0x005942c0 | FUN_005942c0 | event_msg_car_part_dropout_serialize |  |
| 3068 | 0x00594330 | FUN_00594330 | event_msg_car_part_dropout_deserialize |  |
| 3069 | 0x005943b0 | FUN_005943b0 | event_msg_slot_flag_serialize |  |
| 3070 | 0x005943e0 | FUN_005943e0 | event_msg_slot_flag_deserialize |  |
| 3071 | 0x00594420 | FUN_00594420 | event_msg_session_setting_serialize |  |
| 3072 | 0x00594460 | FUN_00594460 | event_msg_session_setting_deserialize |  |
| 3073 | 0x005944b0 | FUN_005944b0 | event_msg_ai_racer_add_serialize |  |
| 3074 | 0x005944f0 | FUN_005944f0 | event_msg_ai_racer_add_deserialize |  |
| 3075 | 0x00594540 | FUN_00594540 | event_msg_ai_racer_remove_serialize |  |
| 3076 | 0x00594570 | FUN_00594570 | event_msg_ai_racer_remove_deserialize |  |
| 3077 | 0x005945a0 | FUN_005945a0 | event_msg_race_finish_serialize |  |
| 3078 | 0x005945e0 | FUN_005945e0 | event_msg_race_finish_deserialize |  |
| 3079 | 0x00594630 | FUN_00594630 | event_msg_dword_pair_payload_ctor |  |
| 3080 | 0x00594650 | FUN_00594650 | event_msg_dword_pair_payload_init |  |
| 3081 | 0x00594660 | FUN_00594660 | event_msg_dword_pair_reversed_deserialize |  |
| 3082 | 0x005946a0 | FUN_005946a0 | door_event_payload_ctor |  |
| 3083 | 0x005946c0 | FUN_005946c0 | door_event_payload_init |  |
| 3084 | 0x005946d0 | FUN_005946d0 | event_msg_door_state_serialize |  |
| 3085 | 0x00594700 | FUN_00594700 | event_msg_door_state_deserialize |  |
| 3086 | 0x00594740 | FUN_00594740 | crt_static_init_guard_00594740 |  |
| 3087 | 0x00594760 | FUN_00594760 | crt_atexit_register_00594760 |  |
| 3088 | 0x00594770 | FUN_00594770 | race_open_menu_scene | OpenMenuScene |
| 3089 | 0x005947e0 | FUN_005947e0 | race_winners_replay_begin | PlayMovie |
| 3090 | 0x005948f0 | FUN_005948f0 | race_winners_replay_end |  |
| 3091 | 0x005949c0 | FUN_005949c0 | race_game_loop | GameLoopSingle1 |
| 3092 | 0x00594e90 | FUN_00594e90 | race_game_loop_net |  |
| 3093 | 0x00595390 | FUN_00595390 | race_net_load_sync |  |
| 3094 | 0x00595a30 | FUN_00595a30 | race_run | RunRacing |
| 3095 | 0x00595d90 | FUN_00595d90 | race_menu_run | ExecuteRacingMenu |
| 3096 | 0x00595f20 | FUN_00595f20 | net_wait_for_session_mode |  |
| 3097 | 0x00595fb0 | FUN_00595fb0 | net_host_session_start |  |
| 3098 | 0x00596280 | FUN_00596280 | net_open_session_browser |  |
| 3099 | 0x00596370 | FUN_00596370 | net_multiplayer_run |  |
| 3100 | 0x00596ad0 | FUN_00596ad0 | race_mission6_init | RacingMisse06 |
| 3101 | 0x00596bd0 | FUN_00596bd0 | race_mission6_execute_loop | RacingMisse06ExecuteLoop |
| 3102 | 0x00596c20 | FUN_00596c20 | event_msg_ctor |  |
| 3103 | 0x00596c50 | FUN_00596c50 | crt_static_init_guard_00596c50 |  |
| 3104 | 0x00596c70 | FUN_00596c70 | crt_atexit_register_00596c70 |  |
| 3105 | 0x00596c80 | FUN_00596c80 | net_session_peer_ctor |  |
| 3106 | 0x00596d00 | FUN_00596d00 | net_session_peer_ctor_unset |  |
| 3107 | 0x00596d80 | FUN_00596d80 | net_session_peer_update_latency |  |
| 3108 | 0x00596e20 | FUN_00596e20 | net_session_peer_append_message_bytes |  |
| 3109 | 0x00596e90 | FUN_00596e90 | net_session_peer_dtor |  |
| 3110 | 0x00596ed0 | FUN_00596ed0 | net_session_ctor |  |
| 3111 | 0x00597130 | FUN_00597130 | net_session_create_connection |  |
| 3112 | 0x00597190 | FUN_00597190 | net_session_init_provider |  |
| 3113 | 0x005971f0 | FUN_005971f0 | net_session_dtor |  |
| 3114 | 0x005974c0 | FUN_005974c0 | net_session_enumerate_providers |  |
| 3115 | 0x00597710 | FUN_00597710 | net_session_settings_addr |  |
| 3116 | 0x00597720 | FUN_00597720 | net_session_game_settings_addr |  |
| 3117 | 0x00597730 | FUN_00597730 | net_session_settings_save |  |
| 3118 | 0x005977e0 | FUN_005977e0 | net_session_settings_apply_saved |  |
| 3119 | 0x005977f0 | FUN_005977f0 | net_session_set_state |  |
| 3120 | 0x00597800 | FUN_00597800 | net_session_settings_apply |  |
| 3121 | 0x005978b0 | FUN_005978b0 | net_session_selected_provider_is_valid |  |
| 3122 | 0x005978f0 | FUN_005978f0 | net_session_find_provider_index_by_type |  |
| 3123 | 0x00597930 | FUN_00597930 | net_session_connect_selected_provider |  |
| 3124 | 0x00597ab0 | FUN_00597ab0 | net_session_create_as_host |  |
| 3125 | 0x00597ba0 | FUN_00597ba0 | net_session_list_available |  |
| 3126 | 0x00597e00 | FUN_00597e00 | net_session_join |  |
| 3127 | 0x00597eb0 | FUN_00597eb0 | net_session_send_queued_messages |  |
| 3128 | 0x005980b0 | FUN_005980b0 | net_session_peer_flush_send_buffer |  |
| 3129 | 0x00598360 | FUN_00598360 | net_session_send_event_msg_direct |  |
| 3130 | 0x00598460 | FUN_00598460 | net_session_post_event_msg |  |
| 3131 | 0x005986f0 | FUN_005986f0 | net_session_send_chat_message |  |
| 3132 | 0x00598860 | FUN_00598860 | net_session_find_peer_by_id |  |
| 3133 | 0x00598890 | FUN_00598890 | net_session_find_peer_by_slot |  |
| 3134 | 0x005988e0 | FUN_005988e0 | net_session_set_name |  |
| 3135 | 0x005988f0 | FUN_005988f0 | net_session_leave |  |
| 3136 | 0x005989b0 | FUN_005989b0 | net_session_start |  |
| 3137 | 0x00598a30 | FUN_00598a30 | net_session_set_password |  |
| 3138 | 0x00598a40 | FUN_00598a40 | net_session_trim_message_log |  |
| 3139 | 0x00598aa0 | FUN_00598aa0 | net_session_ping_tick |  |
| 3140 | 0x00598be0 | FUN_00598be0 | net_session_get_peer_clock_ms |  |
| 3141 | 0x00598c50 | FUN_00598c50 | net_session_average_remote_latency |  |
| 3142 | 0x00598cd0 | FUN_00598cd0 | net_find_actor_by_guid |  |
| 3143 | 0x00598d50 | FUN_00598d50 | net_session_build_actor_guid_table |  |
| 3144 | 0x00598ea0 | FUN_00598ea0 | net_session_clear_message_log |  |
| 3145 | 0x00598f00 | FUN_00598f00 | net_session_peer_id_for_actor_guid |  |
| 3146 | 0x00598f20 | FUN_00598f20 | net_session_game_settings_ctor |  |
| 3147 | 0x00599130 | FUN_00599130 | net_session_game_settings_dtor |  |
| 3148 | 0x00599170 | FUN_00599170 | net_session_set_ready_flag |  |
| 3149 | 0x005991e0 | FUN_005991e0 | net_session_request_ready_flag |  |
| 3150 | 0x00599250 | FUN_00599250 | net_session_all_peers_ready |  |
| 3151 | 0x00599280 | FUN_00599280 | net_session_broadcast_setting |  |
| 3152 | 0x00599420 | FUN_00599420 | net_session_get_car_model_for_player |  |
| 3153 | 0x00599490 | FUN_00599490 | net_session_return_to_lobby |  |
| 3154 | 0x005994f0 | FUN_005994f0 | net_session_all_players_finished_race |  |
| 3155 | 0x00599540 | FUN_00599540 | net_session_reset_wait_timer |  |
| 3156 | 0x00599560 | FUN_00599560 | net_session_wait_timer_expired |  |
| 3157 | 0x00599580 | FUN_00599580 | net_session_is_active |  |
| 3158 | 0x005995a0 | FUN_005995a0 | net_session_can_start |  |
| 3159 | 0x005995d0 | FUN_005995d0 | net_session_local_player_is_configured |  |
| 3160 | 0x00599610 | FUN_00599610 | net_session_remove_peer |  |
| 3161 | 0x005997b0 | FUN_005997b0 | net_session_take_roster_dirty_flag |  |
| 3162 | 0x005997c0 | FUN_005997c0 | net_session_push_system_message |  |
| 3163 | 0x005999d0 | FUN_005999d0 | net_session_tick |  |
| 3164 | 0x00599a40 | FUN_00599a40 | net_session_add_ai_racer |  |
| 3165 | 0x00599b50 | FUN_00599b50 | net_session_remove_ai_racer |  |
| 3166 | 0x00599c30 | FUN_00599c30 | net_session_disable_timeout_watchdog |  |
| 3167 | 0x00599cc0 | FUN_00599cc0 | net_session_enable_timeout_watchdog |  |
| 3168 | 0x00599d20 | FUN_00599d20 | net_session_clear_ai_racers |  |
| 3169 | 0x00599d80 | FUN_00599d80 | net_session_validate_car_for_championship |  |
| 3170 | 0x00599e50 | FUN_00599e50 | net_session_post_scene_done |  |
| 3171 | 0x00599ed0 | FUN_00599ed0 | net_deactivate_actors_of_peer |  |
| 3172 | 0x00599fa0 | FUN_00599fa0 | net_session_settings_init_defaults |  |
| 3173 | 0x0059a070 | FUN_0059a070 | net_session_transfer_actor_ownership |  |
| 3174 | 0x0059a150 | FUN_0059a150 | net_session_partition_peers_by_relevance |  |
| 3175 | 0x0059a520 | FUN_0059a520 | net_session_build_position_grid |  |
| 3176 | 0x0059a810 | FUN_0059a810 | net_session_decode_position |  |
| 3177 | 0x0059a8c0 | FUN_0059a8c0 | net_session_encode_position |  |
| 3178 | 0x0059aaa0 | FUN_0059aaa0 | net_encode_fixed16 |  |
| 3179 | 0x0059ab10 | FUN_0059ab10 | net_decode_fixed16 |  |
| 3180 | 0x0059ab90 | FUN_0059ab90 | crt_static_init_guard_0059ab90 |  |
| 3181 | 0x0059abb0 | FUN_0059abb0 | crt_atexit_register_0059abb0 |  |
| 3182 | 0x0059abc0 | FUN_0059abc0 | car_net_snapshot_state |  |
| 3183 | 0x0059ad50 | thunk_FUN_0059d6d0 | thunk_car_clear_collision_damage_records |  |
| 3184 | 0x0059ad60 | FUN_0059ad60 | car_net_apply_remote_state | C_car::NetDirect |
| 3185 | 0x0059c0a0 | FUN_0059c0a0 | car_broadcast_state_net_event_msg |  |
| 3186 | 0x0059c200 | FUN_0059c200 | car_post_reduced_state_net_event_msg |  |
| 3187 | 0x0059c3a0 | FUN_0059c3a0 | car_post_state_net_event_msg |  |
| 3188 | 0x0059c690 | FUN_0059c690 | car_apply_net_event_msg |  |
| 3189 | 0x0059d2f0 | FUN_0059d2f0 | car_post_break_glass_net_event_msg |  |
| 3190 | 0x0059d3a0 | FUN_0059d3a0 | car_post_hit_net_event_msg |  |
| 3191 | 0x0059d410 | FUN_0059d410 | car_post_part_dropout_net_event_msg |  |
| 3192 | 0x0059d4d0 | FUN_0059d4d0 | car_post_dirty_net_event_msgs |  |
| 3193 | 0x0059d640 | FUN_0059d640 | car_post_collision_damage_net_event_msg |  |
| 3194 | 0x0059d6d0 | FUN_0059d6d0 | car_clear_collision_damage_records |  |
| 3195 | 0x0059d740 | FUN_0059d740 | car_get_collision_damage_records |  |
| 3196 | 0x0059d750 | FUN_0059d750 | car_queue_collision_damage_record |  |
| 3197 | 0x0059d950 | FUN_0059d950 | car_post_explosion_net_event_msg |  |
| 3198 | 0x0059d9d0 | FUN_0059d9d0 | crt_static_init_guard_0059d9d0 |  |
| 3199 | 0x0059d9f0 | FUN_0059d9f0 | crt_atexit_register_0059d9f0 |  |
| 3200 | 0x0059da00 | FUN_0059da00 | event_msg_dtor |  |
| 3201 | 0x0059da30 | FUN_0059da30 | crt_static_init_guard_0059da30 |  |
| 3202 | 0x0059da50 | FUN_0059da50 | crt_atexit_register_0059da50 |  |
| 3203 | 0x0059da60 | FUN_0059da60 | net_session_receive_messages |  |
| 3204 | 0x0059e120 | FUN_0059e120 | net_session_handle_event_msg |  |
| 3205 | 0x005a14b0 | FUN_005a14b0 | net_session_is_host |  |
| 3206 | 0x005a14c0 | FUN_005a14c0 | net_session_get_relevant_peer_list |  |
| 3207 | 0x005a14d0 | FUN_005a14d0 | racing_reset_phase_timer |  |
| 3208 | 0x005a14f0 | FUN_005a14f0 | matrix4_element_addr |  |
| 3209 | 0x005a1510 | FUN_005a1510 | net_session_send_racing_setup_to_peer |  |
| 3210 | 0x005a17b0 | FUN_005a17b0 | vec_index_addr_stride_0x4 |  |
| 3211 | 0x005a17c0 | FUN_005a17c0 | std_vector_begin |  |
| 3212 | 0x005a17d0 | FUN_005a17d0 | folded_get_dword_0x18 |  |
| 3213 | 0x005a17e0 | FUN_005a17e0 | actor_state_event_payload_ctor |  |
| 3214 | 0x005a1830 | FUN_005a1830 | event_msg_actor_state_ctor |  |
| 3215 | 0x005a1860 | FUN_005a1860 | event_msg_physics_state_ctor |  |
| 3216 | 0x005a1890 | FUN_005a1890 | event_msg_car_state_ctor |  |
| 3217 | 0x005a18c0 | FUN_005a18c0 | event_msg_car_glass_break_ctor |  |
| 3218 | 0x005a18f0 | FUN_005a18f0 | event_msg_car_damage_payload_init |  |
| 3219 | 0x005a1900 | FUN_005a1900 | event_msg_car_damage_ctor |  |
| 3220 | 0x005a1930 | FUN_005a1930 | event_msg_dword_pair_payload_init_005a1930 |  |
| 3221 | 0x005a1940 | FUN_005a1940 | event_msg_dword_pair_reversed_ctor |  |
| 3222 | 0x005a1970 | FUN_005a1970 | event_msg_car_collision_damage_payload_init |  |
| 3223 | 0x005a1980 | FUN_005a1980 | event_msg_car_collision_damage_ctor |  |
| 3224 | 0x005a19b0 | FUN_005a19b0 | event_msg_car_hit_ctor |  |
| 3225 | 0x005a19e0 | FUN_005a19e0 | event_msg_car_part_dropout_ctor |  |
| 3226 | 0x005a1a10 | FUN_005a1a10 | event_msg_car_explosion_ctor |  |
| 3227 | 0x005a1a40 | FUN_005a1a40 | event_msg_hit_ctor |  |
| 3228 | 0x005a1a70 | FUN_005a1a70 | event_msg_collision_ctor |  |
| 3229 | 0x005a1aa0 | FUN_005a1aa0 | event_msg_race_camera_ctor |  |
| 3230 | 0x005a1ad0 | FUN_005a1ad0 | event_msg_door_state_ctor |  |
| 3231 | 0x005a1b00 | FUN_005a1b00 | crt_static_init_guard_005a1b00 |  |
| 3232 | 0x005a1b20 | FUN_005a1b20 | crt_atexit_register_005a1b20 |  |
| 3233 | 0x005a1b30 | FUN_005a1b30 | vec3_perturb_with_random_cross_basis |  |
| 3234 | 0x005a1d60 | FUN_005a1d60 | particle_dtor | C_Particle::{dtor} |
| 3235 | 0x005a1d70 | FUN_005a1d70 | particle_release |  |
| 3236 | 0x005a1da0 | FUN_005a1da0 | particle_create | C_Particle::Create |
| 3237 | 0x005a22d0 | FUN_005a22d0 | particle_begin_phase | C_Particle::UpdateDataByPhase |
| 3238 | 0x005a2830 | FUN_005a2830 | particle_tick | C_Particle::Tick |
| 3239 | 0x005a3150 | FUN_005a3150 | crt_static_init_guard_005a3150 |  |
| 3240 | 0x005a3170 | FUN_005a3170 | crt_atexit_register_005a3170 |  |
| 3241 | 0x005a3180 | FUN_005a3180 | particle_effect_dtor | C_ParticleEmitter::{dtor} |
| 3242 | 0x005a3220 | FUN_005a3220 | particle_effect_destroy | C_ParticleEmitter::Destroy |
| 3243 | 0x005a3290 | FUN_005a3290 | particle_effect_init | C_ParticleEmitter::Init |
| 3244 | 0x005a3320 | FUN_005a3320 | particle_effect_spawn_particle | C_ParticleEmitter::CreateParticle |
| 3245 | 0x005a3570 | FUN_005a3570 | particle_effect_tick | C_ParticleEmitter::Tick |
| 3246 | 0x005a38d0 | FUN_005a38d0 | stl_copy_dwords_fastcall |  |
| 3247 | 0x005a38f0 | FUN_005a38f0 | crt_static_init_guard_005a38f0 |  |
| 3248 | 0x005a3910 | FUN_005a3910 | crt_atexit_register_005a3910 |  |
| 3249 | 0x005a3920 | FUN_005a3920 | particle_pool_static_init |  |
| 3250 | 0x005a3930 | FUN_005a3930 | particle_pool_static_ctor |  |
| 3251 | 0x005a3950 | FUN_005a3950 | particle_pool_register_atexit_dtor |  |
| 3252 | 0x005a3960 | FUN_005a3960 | particle_pool_static_dtor |  |
| 3253 | 0x005a3980 | FUN_005a3980 | particle_type_pool_static_init |  |
| 3254 | 0x005a3990 | FUN_005a3990 | particle_type_pool_static_ctor |  |
| 3255 | 0x005a39b0 | FUN_005a39b0 | particle_type_pool_register_atexit_dtor |  |
| 3256 | 0x005a39c0 | FUN_005a39c0 | particle_type_pool_static_dtor |  |
| 3257 | 0x005a39e0 | FUN_005a39e0 | particle_phase_pool_static_init |  |
| 3258 | 0x005a39f0 | FUN_005a39f0 | particle_phase_pool_static_ctor |  |
| 3259 | 0x005a3a10 | FUN_005a3a10 | particle_phase_pool_register_atexit_dtor |  |
| 3260 | 0x005a3a20 | FUN_005a3a20 | particle_phase_pool_static_dtor |  |
| 3261 | 0x005a3a40 | FUN_005a3a40 | particle_effect_pool_static_init |  |
| 3262 | 0x005a3a50 | FUN_005a3a50 | particle_effect_pool_static_ctor |  |
| 3263 | 0x005a3a70 | FUN_005a3a70 | particle_effect_pool_register_atexit_dtor |  |
| 3264 | 0x005a3a80 | FUN_005a3a80 | particle_effect_pool_static_dtor |  |
| 3265 | 0x005a3aa0 | FUN_005a3aa0 | particle_manager_ctor | C_ParticleManager::{ctor} |
| 3266 | 0x005a3b20 | FUN_005a3b20 | particle_manager_dtor | C_ParticleManager::{dtor} |
| 3267 | 0x005a3ba0 | FUN_005a3ba0 | particle_manager_mission_start | C_ParticleManager::Mission_Start |
| 3268 | 0x005a3bf0 | FUN_005a3bf0 | particle_manager_mission_done | C_ParticleManager::Mission_Done |
| 3269 | 0x005a3da0 | FUN_005a3da0 | particle_manager_load_data | C_ParticleManager::LoadData |
| 3270 | 0x005a40b0 | FUN_005a40b0 | particle_manager_create_particle | C_ParticleManager::CreateParticle |
| 3271 | 0x005a4100 | FUN_005a4100 | particle_manager_create_particle_from_type | C_ParticleManager::CreateParticle |
| 3272 | 0x005a4af0 | FUN_005a4af0 | particle_manager_tick | C_ParticleManager::ClientTick |
| 3273 | 0x005a4bb0 | FUN_005a4bb0 | particle_manager_find_by_guid | C_ParticleManager::FindGuid |
| 3274 | 0x005a4c60 | FUN_005a4c60 | particle_manager_destroy_particle | C_ParticleManager::DestroyParticle |
| 3275 | 0x005a4cd0 | FUN_005a4cd0 | particle_manager_is_particle_alive | C_ParticleManager::IsParticleAlive |
| 3276 | 0x005a4cf0 | FUN_005a4cf0 | particle_manager_set_particle_emitting | C_ParticleManager::PauseParticle |
| 3277 | 0x005a4d10 | FUN_005a4d10 | particle_manager_clear_emitters | C_ParticleManager::ClearEmitters |
| 3278 | 0x005a4d70 | FUN_005a4d70 | particle_manager_clear_data | C_ParticleManager::ClearData |
| 3279 | 0x005a4dc0 | FUN_005a4dc0 | particle_manager_create_static_particle | C_ParticleManager::CreateSParticle |
| 3280 | 0x005a5750 | FUN_005a5750 | particle_manager_load_static_particles | C_ParticleManager::LoadStaticParticles |
| 3281 | 0x005a5b90 | FUN_005a5b90 | string_get_mutable_data_at |  |
| 3282 | 0x005a5c10 | FUN_005a5c10 | chunk_open_stack_insert_at |  |
| 3283 | 0x005a5db0 | FUN_005a5db0 | chunk_open_stack_insert_n |  |
| 3284 | 0x005a5fb0 | FUN_005a5fb0 | crt_static_init_guard_005a5fb0 |  |
| 3285 | 0x005a5fd0 | FUN_005a5fd0 | crt_atexit_register_005a5fd0 |  |
| 3286 | 0x005a5fe0 | FUN_005a5fe0 | chunk_enter_next | C_chunk::RAscend |
| 3287 | 0x005a61b0 | FUN_005a61b0 | chunk_leave | C_chunk::Descend |
| 3288 | 0x005a6250 | FUN_005a6250 | chunk_bytes_remaining | C_chunk::Size |
| 3289 | 0x005a6280 | FUN_005a6280 | chunk_read_string_and_leave |  |
| 3290 | 0x005a63d0 | FUN_005a63d0 | particle_data_type_ctor | C_PData_Type::{ctor} |
| 3291 | 0x005a6490 | FUN_005a6490 | particle_data_type_load_data | C_PData_Type::LoadData |
| 3292 | 0x005a69f0 | FUN_005a69f0 | particle_data_ctor |  |
| 3293 | 0x005a6a10 | FUN_005a6a10 | particle_data_scalar_deleting_dtor | C_PData::{dtor} |
| 3294 | 0x005a6a30 | FUN_005a6a30 | particle_data_dtor | C_PData::{dtor} |
| 3295 | 0x005a6a50 | FUN_005a6a50 | particle_data_type_scalar_deleting_dtor |  |
| 3296 | 0x005a6a70 | FUN_005a6a70 | particle_data_type_dtor | C_PData_Type::{dtor} |
| 3297 | 0x005a6b50 | FUN_005a6b50 | particle_data_emitter_scalar_deleting_dtor |  |
| 3298 | 0x005a6b70 | FUN_005a6b70 | particle_data_emitter_dtor | C_PData_Emitter::{dtor} |
| 3299 | 0x005a6c50 | FUN_005a6c50 | particle_type_dtor | C_ParticleEmitter::{dtor} |
| 3300 | 0x005a6d80 | FUN_005a6d80 | particle_type_reset | C_ParticleEmitter::Destroy |
| 3301 | 0x005a6f60 | FUN_005a6f60 | particle_type_init | C_ParticleEmitter::Init |
| 3302 | 0x005a7070 | FUN_005a7070 | crt_static_init_guard_005a7070 |  |
| 3303 | 0x005a7090 | FUN_005a7090 | crt_atexit_register_005a7090 |  |
| 3304 | 0x005a70a0 | thunk_FUN_005a70b0 | thunk_g_vec3_zero_static_init |  |
| 3305 | 0x005a70b0 | FUN_005a70b0 | g_vec3_zero_static_init |  |
| 3306 | 0x005a70d0 | thunk_FUN_005a70e0 | thunk_dir8_unit_xz_static_init |  |
| 3307 | 0x005a70e0 | FUN_005a70e0 | dir8_unit_xz_static_init |  |
| 3308 | 0x005a71e0 | thunk_FUN_005a71f0 | thunk_dir8_square_xz_static_init |  |
| 3309 | 0x005a71f0 | FUN_005a71f0 | dir8_square_xz_static_init |  |
| 3310 | 0x005a72f0 | thunk_FUN_005a7300 | thunk_dir16_square_xz_static_init |  |
| 3311 | 0x005a7300 | FUN_005a7300 | dir16_square_xz_static_init |  |
| 3312 | 0x005a74f0 | FUN_005a74f0 | crt_static_init_guard_005a74f0 |  |
| 3313 | 0x005a7510 | FUN_005a7510 | crt_atexit_register_005a7510 |  |
| 3314 | 0x005a7520 | FUN_005a7520 | crt_static_init_guard_005a7520 |  |
| 3315 | 0x005a7540 | FUN_005a7540 | crt_atexit_register_005a7540 |  |
| 3316 | 0x005a7550 | FUN_005a7550 | cache_base_block_static_init |  |
| 3317 | 0x005a7560 | FUN_005a7560 | cache_base_block_static_ctor |  |
| 3318 | 0x005a7570 | FUN_005a7570 | cache_base_block_register_atexit_dtor |  |
| 3319 | 0x005a7580 | FUN_005a7580 | cache_base_block_static_dtor |  |
| 3320 | 0x005a7590 | FUN_005a7590 | cache_base_block_ctor | C_cache_base_block::{ctor} |
| 3321 | 0x005a75c0 | FUN_005a75c0 | cache_base_block_dtor | C_cache_base_block::{dtor} |
| 3322 | 0x005a7710 | FUN_005a7710 | cache_base_block_close | C_cache_base_block::Close |
| 3323 | 0x005a7800 | FUN_005a7800 | cache_base_block_prepare_scene | C_cache_base_block::PrepareScene |
| 3324 | 0x005a7830 | FUN_005a7830 | cache_base_block_prepare_scene_cb | C_cache_base_block::S_hlp$2095_m_cache_cpp::cbDupl |
| 3325 | 0x005a7910 | FUN_005a7910 | cache_base_block_open | C_cache_base_block::Open |
| 3326 | 0x005a8180 | FUN_005a8180 | cache_base_block_free_block |  |
| 3327 | 0x005a81b0 | FUN_005a81b0 | cache_base_block_open_model | C_cache_base_block::OpenModel |
| 3328 | 0x005a85a0 | FUN_005a85a0 | vector_dispose_at_0x08 |  |
| 3329 | 0x005a85c0 | FUN_005a85c0 | cache_base_block_tick | C_cache_base_block::Tick |
| 3330 | 0x005a8a20 | FUN_005a8a20 | cache_base_block_activate_block | C_cache_base_block::ActivateBlock |
| 3331 | 0x005a8e80 | FUN_005a8e80 | cache_base_block_deactivate_block | C_cache_base_block::DeactivateBlock |
| 3332 | 0x005a8f90 | FUN_005a8f90 | cache_base_block_walk_block_objects |  |
| 3333 | 0x005a90e0 | FUN_005a90e0 | cache_base_block_find_model | C_cache_base_block::FindModel |
| 3334 | 0x005a9120 | FUN_005a9120 | cache_base_block_destroy_frame | C_cache_base_block::DestroyFrame |
| 3335 | 0x005a91f0 | FUN_005a91f0 | cache_base_block_deactivate_all_blocks | C_cache_base_block::VisibleAllFrames |
| 3336 | 0x005a93c0 | FUN_005a93c0 | cache_base_block_is_block_in_range | C_cache_base_block::IsBlockInRange |
| 3337 | 0x005a9720 | FUN_005a9720 | cache_base_block_blocks_destroy |  |
| 3338 | 0x005a9760 | FUN_005a9760 | std_vector14_destroy |  |
| 3339 | 0x005a97a0 | FUN_005a97a0 | cache_base_block_model_pool_erase_at |  |
| 3340 | 0x005a97e0 | FUN_005a97e0 | cache_base_block_blocks_insert_at |  |
| 3341 | 0x005a99e0 | FUN_005a99e0 | cache_base_block_blocks_destroy_range |  |
| 3342 | 0x005a9a10 | FUN_005a9a10 | cache_base_block_model_sets_insert_at |  |
| 3343 | 0x005a9be0 | FUN_005a9be0 | cache_base_block_model_sets_destroy_range |  |
| 3344 | 0x005a9c10 | FUN_005a9c10 | std_vector44_uninit_copy |  |
| 3345 | 0x005a9c50 | FUN_005a9c50 | cache_base_block_blocks_uninit_copy |  |
| 3346 | 0x005a9c90 | FUN_005a9c90 | cache_base_block_blocks_uninit_fill_n |  |
| 3347 | 0x005a9cc0 | FUN_005a9cc0 | cache_base_block_model_sets_uninit_copy |  |
| 3348 | 0x005a9d00 | FUN_005a9d00 | cache_base_block_model_sets_uninit_fill_n |  |
| 3349 | 0x005a9d30 | FUN_005a9d30 | cache_base_block_blocks_copy_assign_forward |  |
| 3350 | 0x005a9ee0 | FUN_005a9ee0 | cache_base_block_blocks_copy_assign_range |  |
| 3351 | 0x005aa0a0 | FUN_005aa0a0 | cache_base_block_blocks_copy_backward |  |
| 3352 | 0x005aa250 | FUN_005aa250 | cache_base_block_model_sets_copy_assign_range |  |
| 3353 | 0x005aa400 | FUN_005aa400 | cache_base_block_model_sets_copy_backward |  |
| 3354 | 0x005aa5d0 | FUN_005aa5d0 | std_vector44_copy_one |  |
| 3355 | 0x005aa5f0 | FUN_005aa5f0 | cache_base_block_block_construct_copy |  |
| 3356 | 0x005aa740 | FUN_005aa740 | cache_base_block_model_set_construct_copy |  |
| 3357 | 0x005aa7f0 | FUN_005aa7f0 | cache_base_block_instances_copy_assign |  |
| 3358 | 0x005aaa50 | FUN_005aaa50 | crt_static_init_guard_005aaa50 |  |
| 3359 | 0x005aaa70 | FUN_005aaa70 | crt_atexit_register_005aaa70 |  |
| 3360 | 0x005aaa80 | FUN_005aaa80 | anim_track_table_static_init |  |
| 3361 | 0x005aaa90 | FUN_005aaa90 | anim_track_table_static_ctor |  |
| 3362 | 0x005aaab0 | FUN_005aaab0 | anim_track_table_register_atexit_dtor |  |
| 3363 | 0x005aaac0 | FUN_005aaac0 | anim_track_table_static_dtor |  |
| 3364 | 0x005aab20 | FUN_005aab20 | anim_machine_ctor | C_animations_machine_de_luxe::{ctor} |
| 3365 | 0x005aab30 | FUN_005aab30 | anim_machine_load_track | C_animations_machine_de_luxe::LoadTrack |
| 3366 | 0x005aaf80 | FUN_005aaf80 | anim_track_dtor | C_animations_machine_de_luxe::ClearTracks |
| 3367 | 0x005aafb0 | FUN_005aafb0 | anim_machine_get_track | C_animations_machine_de_luxe::GetTrack |
| 3368 | 0x005ab020 | FUN_005ab020 | anim_machine_clear_non_base_tracks | C_animations_machine_de_luxe::ClearNoBaseTracks |
| 3369 | 0x005ab170 | FUN_005ab170 | anim_machine_init_from_actor | C_animations_machine_de_luxe::Init |
| 3370 | 0x005ab1e0 | FUN_005ab1e0 | anim_machine_init_from_traffic_element | C_animations_machine_de_luxe::Init |
| 3371 | 0x005ab250 | FUN_005ab250 | anim_machine_init_from_model | C_animations_machine_de_luxe::Init |
| 3372 | 0x005ab2b0 | FUN_005ab2b0 | anim_machine_reset | C_animations_machine_de_luxe::Reset |
| 3373 | 0x005ab300 | FUN_005ab300 | anim_machine_sync_track_time | C_animations_machine_de_luxe::EqualTime |
| 3374 | 0x005ab3a0 | FUN_005ab3a0 | anim_machine_get_active_anim_time | C_animations_machine_de_luxe::GetActiveAnimTime |
| 3375 | 0x005ab3f0 | FUN_005ab3f0 | anim_machine_stop_anims | C_animations_machine_de_luxe::StopAnims |
| 3376 | 0x005ab410 | FUN_005ab410 | anim_machine_recompile_matrix | C_animations_machine_de_luxe::RecompileMatrix |
| 3377 | 0x005ab460 | FUN_005ab460 | anim_machine_set_anim | C_animations_machine_de_luxe::SetAnim |
| 3378 | 0x005abc80 | FUN_005abc80 | anim_machine_is_active | C_animations_machine_de_luxe::IsActive |
| 3379 | 0x005abcc0 | FUN_005abcc0 | anim_machine_tick | C_animations_machine_de_luxe::Tick |
| 3380 | 0x005acb70 | FUN_005acb70 | anim_machine_set_extended_anim | C_animations_machine_de_luxe::SetExtendedAnim |
| 3381 | 0x005acc60 | FUN_005acc60 | anim_machine_del_extended_anim | C_animations_machine_de_luxe::DelExtendedAnim |
| 3382 | 0x005acce0 | FUN_005acce0 | anim_machine_get_placement_from_track | C_animations_machine_de_luxe::GetPlacementFromTCK |
| 3383 | 0x005ace20 | FUN_005ace20 | anim_machine_get_track_end_point | C_animations_machine_de_luxe::GetEndPointOfTCK |
| 3384 | 0x005ace60 | FUN_005ace60 | anim_machine_get_track_mixing_point | C_animations_machine_de_luxe::GetMixingPointOfTCK |
| 3385 | 0x005acee0 | FUN_005acee0 | anim_machine_stage_blending_tick | C_animations_machine_de_luxe::StageBlendingTick |
| 3386 | 0x005ad000 | FUN_005ad000 | anim_track_keys_resize |  |
| 3387 | 0x005ad290 | FUN_005ad290 | anim_track_table_size |  |
| 3388 | 0x005ad2c0 | FUN_005ad2c0 | anim_track_table_insert_at |  |
| 3389 | 0x005ad4a0 | FUN_005ad4a0 | anim_track_table_destroy_range |  |
| 3390 | 0x005ad4d0 | FUN_005ad4d0 | anim_track_table_uninit_copy |  |
| 3391 | 0x005ad510 | FUN_005ad510 | anim_track_table_uninit_fill_n |  |
| 3392 | 0x005ad540 | FUN_005ad540 | anim_track_table_copy_assign_range |  |
| 3393 | 0x005ad760 | FUN_005ad760 | anim_track_table_copy_backward |  |
| 3394 | 0x005ad970 | FUN_005ad970 | anim_track_construct_copy |  |
| 3395 | 0x005adaf0 | FUN_005adaf0 | anim_track_assign |  |
| 3396 | 0x005add00 | FUN_005add00 | std_vectorc_copy_assign |  |
| 3397 | 0x005adf60 | FUN_005adf60 | vec_capacity_stride_0xc |  |
| 3398 | 0x005adf80 | FUN_005adf80 | crt_static_init_guard_005adf80 |  |
| 3399 | 0x005adfa0 | FUN_005adfa0 | crt_atexit_register_005adfa0 |  |
| 3400 | 0x005adfb0 | FUN_005adfb0 | blood_tick | C_blood::Tick |
| 3401 | 0x005ae130 | FUN_005ae130 | blood_init | C_blood::Init |
| 3402 | 0x005ae180 | FUN_005ae180 | blood_manager_game_init | C_blood_manager::GameInit |
| 3403 | 0x005ae1b0 | FUN_005ae1b0 | blood_manager_game_done | C_blood_manager::GameDone |
| 3404 | 0x005ae200 | FUN_005ae200 | blood_manager_tick | C_blood_manager::Tick |
| 3405 | 0x005ae2a0 | FUN_005ae2a0 | blood_manager_add_blood | C_blood_manager::AddBlood |
| 3406 | 0x005ae480 | FUN_005ae480 | crt_static_init_guard_005ae480 |  |
| 3407 | 0x005ae4a0 | FUN_005ae4a0 | crt_atexit_register_005ae4a0 |  |
| 3408 | 0x005ae4b0 | FUN_005ae4b0 | force_feedback_effect_zero_member_at_0x04 |  |
| 3409 | 0x005ae4c0 | FUN_005ae4c0 | force_feedback_effect_ctor |  |
| 3410 | 0x005ae510 | FUN_005ae510 | force_feedback_effect_upload_common_params |  |
| 3411 | 0x005ae550 | FUN_005ae550 | force_feedback_effect_is_playing |  |
| 3412 | 0x005ae580 | FUN_005ae580 | force_feedback_effect_create_on_device |  |
| 3413 | 0x005ae5b0 | FUN_005ae5b0 | force_feedback_effect_constant_upload_params |  |
| 3414 | 0x005ae5d0 | FUN_005ae5d0 | force_feedback_effect_ramp_upload_params |  |
| 3415 | 0x005ae5f0 | FUN_005ae5f0 | force_feedback_effect_periodic_ctor |  |
| 3416 | 0x005ae660 | FUN_005ae660 | force_feedback_effect_periodic_upload_params |  |
| 3417 | 0x005ae690 | FUN_005ae690 | force_feedback_effect_condition_upload_params |  |
| 3418 | 0x005ae6b0 | FUN_005ae6b0 | force_feedback_manager_ctor |  |
| 3419 | 0x005ae6f0 | FUN_005ae6f0 | force_feedback_manager_dtor |  |
| 3420 | 0x005ae720 | FUN_005ae720 | force_feedback_manager_reset_handle_counter |  |
| 3421 | 0x005ae730 | FUN_005ae730 | force_feedback_manager_game_init |  |
| 3422 | 0x005ae7c0 | FUN_005ae7c0 | force_feedback_manager_game_done |  |
| 3423 | 0x005ae890 | FUN_005ae890 | force_feedback_manager_create_effect |  |
| 3424 | 0x005aeb10 | FUN_005aeb10 | force_feedback_manager_find_effect_by_handle |  |
| 3425 | 0x005aeb80 | FUN_005aeb80 | force_feedback_manager_set_constant_magnitude |  |
| 3426 | 0x005aebc0 | FUN_005aebc0 | force_feedback_manager_set_ramp_range |  |
| 3427 | 0x005aec00 | FUN_005aec00 | force_feedback_manager_set_periodic_params |  |
| 3428 | 0x005aec60 | FUN_005aec60 | force_feedback_manager_set_condition_params |  |
| 3429 | 0x005aece0 | FUN_005aece0 | force_feedback_manager_set_gain |  |
| 3430 | 0x005aed10 | FUN_005aed10 | force_feedback_manager_set_direction |  |
| 3431 | 0x005aed40 | FUN_005aed40 | force_feedback_manager_set_duration |  |
| 3432 | 0x005aed80 | FUN_005aed80 | force_feedback_manager_play_effect |  |
| 3433 | 0x005af010 | FUN_005af010 | force_feedback_manager_release_effect |  |
| 3434 | 0x005af0e0 | FUN_005af0e0 | force_feedback_manager_stop_effect |  |
| 3435 | 0x005af140 | FUN_005af140 | force_feedback_manager_find_evictable_effect |  |
| 3436 | 0x005af200 | FUN_005af200 | force_feedback_manager_find_highest_priority_pending |  |
| 3437 | 0x005af240 | FUN_005af240 | force_feedback_manager_tick |  |
| 3438 | 0x005af260 | FUN_005af260 | force_feedback_manager_retry_pending_effects |  |
| 3439 | 0x005af290 | FUN_005af290 | crt_static_init_guard_005af290 |  |
| 3440 | 0x005af2b0 | FUN_005af2b0 | crt_atexit_register_005af2b0 |  |
| 3441 | 0x005af2c0 | FUN_005af2c0 | crt_static_init_guard_005af2c0 |  |
| 3442 | 0x005af2e0 | FUN_005af2e0 | crt_atexit_register_005af2e0 |  |
| 3443 | 0x005af2f0 | FUN_005af2f0 | program_ctor | C_program::{ctor} |
| 3444 | 0x005af340 | FUN_005af340 | program_scalar_deleting_dtor |  |
| 3445 | 0x005af360 | FUN_005af360 | program_dtor | C_program::{dtor} |
| 3446 | 0x005af3c0 | FUN_005af3c0 | program_save_source_code | C_program::SaveSourceCode |
| 3447 | 0x005af4b0 | FUN_005af4b0 | program_load_source_code | C_program::LoadSourceCode |
| 3448 | 0x005af730 | FUN_005af730 | program_set_source_code | C_program::SetSourceCode |
| 3449 | 0x005af780 | FUN_005af780 | program_init | C_program::Init |
| 3450 | 0x005af8a0 | FUN_005af8a0 | program_done | C_program::Done |
| 3451 | 0x005af9d0 | FUN_005af9d0 | program_duplicate_from | C_program::DuplicateProgramFrom |
| 3452 | 0x005afa30 | FUN_005afa30 | program_split_params | C_program::ExtrudeParams |
| 3453 | 0x005afb00 | FUN_005afb00 | program_decode_operand_float | C_program::DecodeOperand |
| 3454 | 0x005afbb0 | FUN_005afbb0 | program_decode_operand_int | C_program::DecodeOperandInt |
| 3455 | 0x005afc60 | FUN_005afc60 | program_extract_quoted_string | C_program::ExtrudeString |
| 3456 | 0x005afca0 | FUN_005afca0 | program_decode_let | C_program::DecodeLet |
| 3457 | 0x005afdd0 | FUN_005afdd0 | program_decode_if | C_program::DecodeIf |
| 3458 | 0x005afe70 | FUN_005afe70 | program_decode_params | C_program::DecodeParams |
| 3459 | 0x005b1b60 | FUN_005b1b60 | program_decode_instruction | C_program::DecodeInstruction |
| 3460 | 0x005ba6a0 | FUN_005ba6a0 | program_compile | C_program::Compile |
| 3461 | 0x005ba9a0 | FUN_005ba9a0 | program_get_float_reg | C_program::GetFloatReg |
| 3462 | 0x005ba9c0 | FUN_005ba9c0 | program_get_frame_reg | C_program::GetFrameReg |
| 3463 | 0x005ba9e0 | FUN_005ba9e0 | program_get_actor_reg | C_program::GetActorReg |
| 3464 | 0x005baa00 | FUN_005baa00 | program_set_float_reg | C_program::SetFloatReg |
| 3465 | 0x005baa20 | FUN_005baa20 | program_set_frame_reg | C_program::SetFrameReg |
| 3466 | 0x005baa40 | FUN_005baa40 | program_set_actor_reg | C_program::SetActorReg |
| 3467 | 0x005baa60 | FUN_005baa60 | program_next_command | C_program::NextCommand |
| 3468 | 0x005baae0 | FUN_005baae0 | program_call_subroutine | C_program::CallSubroutine |
| 3469 | 0x005bab20 | FUN_005bab20 | program_leave_compact_block | C_program::LeaveCompactBlock |
| 3470 | 0x005babd0 | FUN_005babd0 | program_call_process | C_program::CallProcess |
| 3471 | 0x005bac10 | FUN_005bac10 | program_clear_after_end | C_program::ClearAfterEnd |
| 3472 | 0x005bacb0 | FUN_005bacb0 | program_set_event | C_program::SetEvent |
| 3473 | 0x005bad90 | FUN_005bad90 | program_is_event_id | C_program::IsEventID |
| 3474 | 0x005bade0 | FUN_005bade0 | program_invalidate_actor_reg | C_program::InvalidateActorReg |
| 3475 | 0x005bae10 | FUN_005bae10 | program_save_game_get_size | C_program::SaveGameGetSize |
| 3476 | 0x005bae90 | FUN_005bae90 | program_save_game_save | C_program::SaveGameSave |
| 3477 | 0x005bb020 | FUN_005bb020 | program_save_game_load | C_program::SaveGameLoad |
| 3478 | 0x005bb2a0 | FUN_005bb2a0 | program_find_actor_later | C_program::FindActorLater |
| 3479 | 0x005bb2d0 | FUN_005bb2d0 | program_process_command | C_program::Process |
| 3480 | 0x005c7da0 | FUN_005c7da0 | hud_set_element_flags |  |
| 3481 | 0x005c7db0 | FUN_005c7db0 | hud_hide_element |  |
| 3482 | 0x005c7dd0 | FUN_005c7dd0 | hud_element_is_visible |  |
| 3483 | 0x005c7df0 | FUN_005c7df0 | hud_push_element_flags |  |
| 3484 | 0x005c7fb0 | FUN_005c7fb0 | hud_pop_element_flags |  |
| 3485 | 0x005c8010 | FUN_005c8010 | hud_set_police_wanted_bar |  |
| 3486 | 0x005c8020 | FUN_005c8020 | matrix4_zero |  |
| 3487 | 0x005c8030 | FUN_005c8030 | matrix4_invert_in_place | S_matrix::Inverse |
| 3488 | 0x005c8060 | FUN_005c8060 | quat_add |  |
| 3489 | 0x005c80b0 | FUN_005c80b0 | quat_sub_w_add_xyz |  |
| 3490 | 0x005c8100 | FUN_005c8100 | quat_multiply_in |  |
| 3491 | 0x005c81f0 | FUN_005c81f0 | quat_scale_to |  |
| 3492 | 0x005c8240 | FUN_005c8240 | quat_dot |  |
| 3493 | 0x005c8270 | FUN_005c8270 | racing_get_racer_array_ptr |  |
| 3494 | 0x005c8280 | FUN_005c8280 | racing_get_racer_count |  |
| 3495 | 0x005c8290 | FUN_005c8290 | cache_base_block_is_loaded |  |
| 3496 | 0x005c82a0 | FUN_005c82a0 | i3d_frame_get_type_ptr |  |
| 3497 | 0x005c82b0 | FUN_005c82b0 | i3d_frame_set_position_xyz_ptrs |  |
| 3498 | 0x005c8300 | FUN_005c8300 | i3d_frame_set_scale |  |
| 3499 | 0x005c8360 | FUN_005c8360 | folded_get_ptr_0x90 |  |
| 3500 | 0x005c8370 | FUN_005c8370 | i3d_frame_get_rotation_ptr |  |
| 3501 | 0x005c83a0 | FUN_005c83a0 | i3d_frame_get_world_rotation_quat |  |
| 3502 | 0x005c83f0 | FUN_005c83f0 | i3d_frame_get_scaled_rot_matrix_ptr |  |
| 3503 | 0x005c8510 | FUN_005c8510 | i3d_frame_is_on |  |
| 3504 | 0x005c8520 | FUN_005c8520 | i3d_frame_get_name |  |
| 3505 | 0x005c8530 | FUN_005c8530 | i3d_frame_get_child_count_ptr |  |
| 3506 | 0x005c8540 | FUN_005c8540 | i3d_animation_set_set_default_options |  |
| 3507 | 0x005c8550 | FUN_005c8550 | i3d_face_group_get_material_ptr |  |
| 3508 | 0x005c8560 | FUN_005c8560 | i3d_mesh_level_get_face_group_count_ptr |  |
| 3509 | 0x005c8570 | FUN_005c8570 | i3d_mesh_object_get_lod_count_ptr |  |
| 3510 | 0x005c8580 | FUN_005c8580 | i3d_frame_get_tag_ptr |  |
| 3511 | 0x005c8590 | FUN_005c8590 | i3d_frame_get_mesh_object_ptr |  |
| 3512 | 0x005c85a0 | FUN_005c85a0 | i3d_camera_get_fov_ptr |  |
| 3513 | 0x005c85b0 | FUN_005c85b0 | i3d_frame_get_lmap_entry_ptr |  |
| 3514 | 0x005c85d0 | FUN_005c85d0 | i3d_frame_get_model_name |  |
| 3515 | 0x005c85e0 | FUN_005c85e0 | i3d_scene_get_active_camera_ptr |  |
| 3516 | 0x005c85f0 | FUN_005c85f0 | i3d_scene_get_root_frame_ptr |  |
| 3517 | 0x005c8600 | FUN_005c8600 | i3d_scene_set_frame_sector_pos | I3D_scene::SetFrameSectorPos |
| 3518 | 0x005c8630 | FUN_005c8630 | i3d_sound_get_volume_ptr |  |
| 3519 | 0x005c8640 | FUN_005c8640 | mission_get_actor_list |  |
| 3520 | 0x005c8650 | FUN_005c8650 | mission_get_particle_manager |  |
| 3521 | 0x005c8660 | FUN_005c8660 | mission_set_next_mission_number |  |
| 3522 | 0x005c8670 | FUN_005c8670 | mission_get_mission_number |  |
| 3523 | 0x005c8680 | FUN_005c8680 | mission_commit_next_mission_number |  |
| 3524 | 0x005c8690 | FUN_005c8690 | mission_set_sound_restore_stage |  |
| 3525 | 0x005c86a0 | FUN_005c86a0 | mission_get_sound_restore_stage |  |
| 3526 | 0x005c86b0 | FUN_005c86b0 | using_object_record_ctor |  |
| 3527 | 0x005c86e0 | FUN_005c86e0 | actor_get_class_id |  |
| 3528 | 0x005c86f0 | FUN_005c86f0 | actor_set_cached_world_pos |  |
| 3529 | 0x005c8710 | FUN_005c8710 | actor_get_direction |  |
| 3530 | 0x005c8730 | FUN_005c8730 | actor_mark_created_at_runtime |  |
| 3531 | 0x005c8740 | FUN_005c8740 | vehicle_controller_set_field_0x65 |  |
| 3532 | 0x005c8750 | FUN_005c8750 | actor_get_global_list |  |
| 3533 | 0x005c8760 | FUN_005c8760 | program_set_ai_priority |  |
| 3534 | 0x005c8770 | FUN_005c8770 | loadsave_set_mission_name_id |  |
| 3535 | 0x005c8780 | FUN_005c8780 | vehicle_get_special_dynamic_colls |  |
| 3536 | 0x005c8790 | FUN_005c8790 | vehicle_set_velocity |  |
| 3537 | 0x005c8840 | FUN_005c8840 | vehicle_set_angular_velocity |  |
| 3538 | 0x005c8880 | FUN_005c8880 | vehicle_set_auto_indicators_from_steering |  |
| 3539 | 0x005c88b0 | FUN_005c88b0 | vehicle_set_light_bit_0x80 |  |
| 3540 | 0x005c88e0 | FUN_005c88e0 | vehicle_set_left_indicator |  |
| 3541 | 0x005c8920 | FUN_005c8920 | vehicle_set_right_indicator |  |
| 3542 | 0x005c8960 | FUN_005c8960 | vehicle_set_headlights |  |
| 3543 | 0x005c89a0 | FUN_005c89a0 | vehicle_set_light_script_override |  |
| 3544 | 0x005c89e0 | FUN_005c89e0 | vehicle_get_light_script_override |  |
| 3545 | 0x005c89f0 | FUN_005c89f0 | car_set_tank_hit_count |  |
| 3546 | 0x005c8a10 | FUN_005c8a10 | car_set_must_steal |  |
| 3547 | 0x005c8a20 | FUN_005c8a20 | car_set_destroy_motor |  |
| 3548 | 0x005c8a30 | FUN_005c8a30 | car_set_broken_motor |  |
| 3549 | 0x005c8a40 | FUN_005c8a40 | car_get_model_name_ptr |  |
| 3550 | 0x005c8a50 | FUN_005c8a50 | car_set_show_damage |  |
| 3551 | 0x005c8a60 | FUN_005c8a60 | car_adjust_brake_hold_count |  |
| 3552 | 0x005c8a90 | FUN_005c8a90 | vehicle_controller_set_field_0x214c |  |
| 3553 | 0x005c8aa0 | FUN_005c8aa0 | car_set_auto_headlights_suppressed |  |
| 3554 | 0x005c8ab0 | FUN_005c8ab0 | car_set_effect_mask |  |
| 3555 | 0x005c8ac0 | FUN_005c8ac0 | car_set_damage_inhibit_mask |  |
| 3556 | 0x005c8ad0 | FUN_005c8ad0 | cutscene_set_active |  |
| 3557 | 0x005c8ae0 | FUN_005c8ae0 | camera_get_car |  |
| 3558 | 0x005c8af0 | FUN_005c8af0 | police_manager_is_enabled |  |
| 3559 | 0x005c8b00 | FUN_005c8b00 | police_manager_set_speed_thresholds |  |
| 3560 | 0x005c8b20 | FUN_005c8b20 | game_get_using_object |  |
| 3561 | 0x005c8b30 | FUN_005c8b30 | game_request_clear_differences |  |
| 3562 | 0x005c8b40 | FUN_005c8b40 | game_get_pursuit_event_manager |  |
| 3563 | 0x005c8b50 | FUN_005c8b50 | game_get_frame_count |  |
| 3564 | 0x005c8b60 | FUN_005c8b60 | game_set_night_mission |  |
| 3565 | 0x005c8b70 | FUN_005c8b70 | game_set_timer_timeout_event |  |
| 3566 | 0x005c8b90 | FUN_005c8b90 | game_set_player_fire_event |  |
| 3567 | 0x005c8bb0 | FUN_005c8bb0 | game_set_player_horn_event |  |
| 3568 | 0x005c8bd0 | FUN_005c8bd0 | game_set_player_water_fall_event |  |
| 3569 | 0x005c8bf0 | FUN_005c8bf0 | game_set_mission_end_action |  |
| 3570 | 0x005c8c00 | FUN_005c8c00 | game_set_race_state |  |
| 3571 | 0x005c8c20 | FUN_005c8c20 | game_get_race_state |  |
| 3572 | 0x005c8c30 | FUN_005c8c30 | game_set_race_result_register |  |
| 3573 | 0x005c8c40 | FUN_005c8c40 | game_set_actor_lightness_flag |  |
| 3574 | 0x005c8c50 | FUN_005c8c50 | game_set_heavy_init_enabled |  |
| 3575 | 0x005c8c60 | FUN_005c8c60 | game_set_rain_buffer_size |  |
| 3576 | 0x005c8c70 | FUN_005c8c70 | game_set_script_mission_end |  |
| 3577 | 0x005c8c80 | FUN_005c8c80 | car_extended_can_board |  |
| 3578 | 0x005c8ca0 | FUN_005c8ca0 | wagon_set_event |  |
| 3579 | 0x005c8cc0 | FUN_005c8cc0 | wagon_get_last_stop_node_id |  |
| 3580 | 0x005c8d00 | FUN_005c8d00 | detector_get_signal |  |
| 3581 | 0x005c8d10 | FUN_005c8d10 | detector_set_signal |  |
| 3582 | 0x005c8d30 | FUN_005c8d30 | pedestrian_manager_set_scream_enabled | C_traffic_generator::global_valid_scream |
| 3583 | 0x005c8d40 | FUN_005c8d40 | human_pose_set_look_target_frame |  |
| 3584 | 0x005c8d50 | FUN_005c8d50 | player_set_wingman_indicator |  |
| 3585 | 0x005c8d70 | FUN_005c8d70 | human_set_fire_target_frame |  |
| 3586 | 0x005c8d80 | FUN_005c8d80 | human_set_no_hit_anim |  |
| 3587 | 0x005c8d90 | FUN_005c8d90 | human_set_idle_anims_enabled |  |
| 3588 | 0x005c8da0 | FUN_005c8da0 | human_set_silent |  |
| 3589 | 0x005c8db0 | FUN_005c8db0 | hud_set_car_damage_bar_enabled |  |
| 3590 | 0x005c8dc0 | FUN_005c8dc0 | ai_groups_add_tracked_actor |  |
| 3591 | 0x005c8f70 | FUN_005c8f70 | ai_logs_clear_all_logs |  |
| 3592 | 0x005c8fa0 | FUN_005c8fa0 | entity_get_ai_state_base_code |  |
| 3593 | 0x005c8fe0 | FUN_005c8fe0 | entity_set_ai_state_resettable |  |
| 3594 | 0x005c9010 | FUN_005c9010 | entity_set_state_switcher_suspended |  |
| 3595 | 0x005c9040 | FUN_005c9040 | entity_unregister_using_object |  |
| 3596 | 0x005c9060 | FUN_005c9060 | program_switch_light_map | C_program::S_help_4741C_program_cpp::SwitchLightMap |
| 3597 | 0x005c90d0 | FUN_005c90d0 | std_vector8_push_back |  |
| 3598 | 0x005c9290 | FUN_005c9290 | program_instructions_clear |  |
| 3599 | 0x005c92c0 | FUN_005c92c0 | program_return_stack_back |  |
| 3600 | 0x005c92d0 | FUN_005c92d0 | program_return_stack_push |  |
| 3601 | 0x005c9480 | FUN_005c9480 | program_return_stack_pop |  |
| 3602 | 0x005c94b0 | FUN_005c94b0 | std_vector8_insert_at |  |
| 3603 | 0x005c96f0 | FUN_005c96f0 | stl_max_float_ptr | Max<f> |
| 3604 | 0x005c9710 | FUN_005c9710 | stl_max_int_ptr |  |
| 3605 | 0x005c9720 | FUN_005c9720 | crt_static_init_guard_005c9720 |  |
| 3606 | 0x005c9740 | FUN_005c9740 | crt_atexit_register_005c9740 |  |
| 3607 | 0x005c9750 | FUN_005c9750 | racing_optimal_segment_ctor | C_racing_osegment::{ctor} |
| 3608 | 0x005c9770 | FUN_005c9770 | racing_record_segment_points |  |
| 3609 | 0x005c99a0 | FUN_005c99a0 | racing_create_segments | C_racing::CreateSegments |
| 3610 | 0x005c9c90 | FUN_005c9c90 | racing_save_segments |  |
| 3611 | 0x005c9e30 | FUN_005c9e30 | racing_load_segments | C_racing::LoadSegments |
| 3612 | 0x005ca040 | FUN_005ca040 | racing_clear_segments | C_racing::ClearSegments |
| 3613 | 0x005ca0a0 | FUN_005ca0a0 | racing_get_segment_bank_for_car |  |
| 3614 | 0x005ca0f0 | FUN_005ca0f0 | racing_finish_segment_and_advance |  |
| 3615 | 0x005ca290 | FUN_005ca290 | crt_static_init_guard_005ca290 |  |
| 3616 | 0x005ca2b0 | FUN_005ca2b0 | crt_atexit_register_005ca2b0 |  |
| 3617 | 0x005ca2c0 | FUN_005ca2c0 | random_morph_static_init | __sinit_C_rmorf.cpp |
| 3618 | 0x005ca2d0 | FUN_005ca2d0 | random_morph_static_ctor | __sinit_C_rmorf.cpp |
| 3619 | 0x005ca2e0 | FUN_005ca2e0 | random_morph_register_atexit_dtor | __sinit_C_rmorf.cpp |
| 3620 | 0x005ca2f0 | FUN_005ca2f0 | random_morph_static_dtor | __sinit_C_rmorf.cpp |
| 3621 | 0x005ca300 | FUN_005ca300 | random_morph_set_load_raw | RMRF_S_SETREF::LoadRAWSet |
| 3622 | 0x005ca4b0 | FUN_005ca4b0 | random_morph_ctor | C_rmorf::{ctor} |
| 3623 | 0x005ca4c0 | FUN_005ca4c0 | random_morph_dtor | C_rmorf::{dtor} |
| 3624 | 0x005ca510 | FUN_005ca510 | random_morph_set_destroy | RMRF_S_SETREF::{dtor} |
| 3625 | 0x005ca560 | FUN_005ca560 | random_morph_mission_start | C_rmorf::Mission_Start |
| 3626 | 0x005ca680 | FUN_005ca680 | std_string_ctor_formatted | fstring::{ctor} |
| 3627 | 0x005ca790 | FUN_005ca790 | random_morph_mission_done | C_rmorf::Mission_Done |
| 3628 | 0x005ca7e0 | FUN_005ca7e0 | random_morph_tick | C_rmorf::Tick |
| 3629 | 0x005caba0 | FUN_005caba0 | std_string_dtor |  |
| 3630 | 0x005cabe0 | FUN_005cabe0 | std_string_tidy |  |
| 3631 | 0x005cac30 | FUN_005cac30 | std_string_grow_buffer |  |
| 3632 | 0x005cac89 | Catch@005cac89 | Catch_005cac89 |  |
| 3633 | 0x005cad40 | FUN_005cad40 | crt_static_init_guard_005cad40 |  |
| 3634 | 0x005cad60 | FUN_005cad60 | crt_atexit_register_005cad60 |  |
| 3635 | 0x005cad70 | FUN_005cad70 | police_manager_ctor | C_shvestky::{ctor} |
| 3636 | 0x005cae30 | FUN_005cae30 | police_manager_dtor | C_shvestky::{dtor} |
| 3637 | 0x005caf00 | FUN_005caf00 | police_manager_game_init | C_shvestky::GameInit |
| 3638 | 0x005caff0 | FUN_005caff0 | police_manager_game_done | C_shvestky::GameDone |
| 3639 | 0x005cb330 | FUN_005cb330 | police_manager_record_free_officers |  |
| 3640 | 0x005cb350 | FUN_005cb350 | police_manager_add_tracked_police_car | C_shvestky::DezoSebralGumyIRadio |
| 3641 | 0x005cb510 | FUN_005cb510 | police_manager_tick | C_shvestky::Tick |
| 3642 | 0x005cc9a0 | FUN_005cc9a0 | vec3_is_near_zero |  |
| 3643 | 0x005cc9e0 | FUN_005cc9e0 | police_manager_apply_state | C_shvestky::SetActive |
| 3644 | 0x005cd960 | FUN_005cd960 | police_manager_register_actor | C_shvestky::AddPerson |
| 3645 | 0x005cdbd0 | FUN_005cdbd0 | police_manager_unregister_actor | C_shvestky::DelPerson |
| 3646 | 0x005cdca0 | FUN_005cdca0 | police_manager_register_gunfire_offence_for_target | C_shvestky::Firing |
| 3647 | 0x005cdd30 | FUN_005cdd30 | police_manager_link_target_to_vehicle | C_shvestky::DriveCar |
| 3648 | 0x005cde10 | FUN_005cde10 | police_manager_register_collision_offence_for_target | C_shvestky::CarCollision |
| 3649 | 0x005ce150 | FUN_005ce150 | police_manager_register_sidewalk_offence_for_target | C_shvestky::CarIllegalPath |
| 3650 | 0x005ce1b0 | FUN_005ce1b0 | police_manager_register_violence_offence_for_target | C_shvestky::HumanBoxing |
| 3651 | 0x005ce230 | FUN_005ce230 | police_manager_set_car_theft_offence_for_target | C_shvestky::CarSteal |
| 3652 | 0x005ce2b0 | FUN_005ce2b0 | police_manager_escalate_handcuffs_with_optional_wanted | C_shvestky::ForceArrest |
| 3653 | 0x005ce320 | FUN_005ce320 | police_manager_register_traffic_sign_offence_for_target | C_shvestky::DestroySignalMark |
| 3654 | 0x005ce3a0 | FUN_005ce3a0 | police_manager_increase_run_over_count_for_target | C_shvestky::NextNaSrackySoudruh |
| 3655 | 0x005ce3e0 | FUN_005ce3e0 | police_manager_report_police_car_stealing_sighting | C_shvestky::CorujuPoldumMasinu |
| 3656 | 0x005ce460 | FUN_005ce460 | police_manager_enable_roadblock_flag_for_target | C_shvestky::SerSeMuDoCesty |
| 3657 | 0x005ce4c0 | FUN_005ce4c0 | police_manager_get_offense_bits_for_target | C_shvestky::GetHumanStatus |
| 3658 | 0x005ce510 | FUN_005ce510 | police_manager_get_committed_police_state | C_shvestky::JakPicusNaTomJest |
| 3659 | 0x005ce560 | FUN_005ce560 | police_manager_get_crashtime_for_target | C_shvestky::GetCrashTime |
| 3660 | 0x005ce5a0 | FUN_005ce5a0 | police_manager_get_firetime_for_target | C_shvestky::GetFireTime |
| 3661 | 0x005ce5e0 | FUN_005ce5e0 | police_manager_count_officers_for_target | C_shvestky::PocetNasranych |
| 3662 | 0x005ce630 | FUN_005ce630 | police_manager_count_officers_alive_for_target | C_shvestky::PocetNasranychZivych |
| 3663 | 0x005ce6a0 | FUN_005ce6a0 | police_manager_get_num_of_offences | C_shvestky::PocetPrestupku |
| 3664 | 0x005ce6e0 | FUN_005ce6e0 | police_manager_target_identity_pending | C_shvestky::NeviDoKterePiceVlezl |
| 3665 | 0x005ce730 | FUN_005ce730 | police_manager_is_wanted_substate_active_for_target | C_shvestky::ChceHoOpichatKazdyPoldaNaOkresku |
| 3666 | 0x005ce770 | FUN_005ce770 | police_manager_deploy_vehicle_from_pool | C_shvestky::CreateCar |
| 3667 | 0x005ce8d0 | FUN_005ce8d0 | police_manager_deploy_temp_spike_strip |  |
| 3668 | 0x005ce9e0 | FUN_005ce9e0 | police_manager_deploy_npc_from_pool |  |
| 3669 | 0x005ceb00 | FUN_005ceb00 | police_manager_spawn_roadblock_in_neighbor_cells |  |
| 3670 | 0x005cf3c0 | FUN_005cf3c0 | police_manager_acquire_from_pool_by_name | C_shvestky::MasVolnouPlecku |
| 3671 | 0x005cf540 | FUN_005cf540 | police_manager_vehicle_pool_pop_random | C_shvestky::GetTempCar |
| 3672 | 0x005cf610 | FUN_005cf610 | police_manager_free_temp_car | C_shvestky::FreeTempCar |
| 3673 | 0x005cf6b0 | FUN_005cf6b0 | police_manager_get_free_officers_count | C_shvestky::GetFreeEnemy |
| 3674 | 0x005cf6e0 | FUN_005cf6e0 | police_manager_spike_strip_pool_pop |  |
| 3675 | 0x005cf720 | FUN_005cf720 | police_manager_free_temp_spike_strip | C_shvestky::FreeTempIris |
| 3676 | 0x005cf790 | FUN_005cf790 | police_manager_officer_pool_pop_random | C_shvestky::GetTempEnemy |
| 3677 | 0x005cf810 | FUN_005cf810 | police_manager_free_temp_enemy | C_shvestky::FreeTempEnemy |
| 3678 | 0x005cf960 | FUN_005cf960 | police_manager_free_temp_actor | C_shvestky::FreeMyTempActor |
| 3679 | 0x005cf9a0 | FUN_005cf9a0 | police_manager_npc_list_push | C_shvestky::AddDoRiciActor |
| 3680 | 0x005cfb60 | FUN_005cfb60 | police_manager_npc_list_erase | C_shvestky::DelDoRiciActor |
| 3681 | 0x005cfc00 | FUN_005cfc00 | police_manager_npc_list_tick | C_shvestky::TickDoRiciActors |
| 3682 | 0x005cff60 | FUN_005cff60 | police_manager_invalidate_discarded_npcs | C_shvestky::ZlikvidujVsechnyNasraneVyhozeneCuraky |
| 3683 | 0x005d0060 | FUN_005d0060 | police_manager_spawn_patrol_car_reinforcements_at_radius | C_shvestky::DalsiChalpiDoPartie |
| 3684 | 0x005d06c0 | FUN_005d06c0 | police_manager_remove_officer_from_all_records | C_shvestky::GlobalDelNasrany |
| 3685 | 0x005d0700 | FUN_005d0700 | police_manager_add_officer_to_target | C_shvestky::AddDoRiciActor |
| 3686 | 0x005d0750 | FUN_005d0750 | police_manager_record_add_officer | C_shvestky::S_shvestky::AddNasrany |
| 3687 | 0x005d0900 | FUN_005d0900 | police_manager_record_delete_officer | C_shvestky::S_shvestky::DelNasrany |
| 3688 | 0x005d0960 | FUN_005d0960 | police_manager_record_raise_police_state | C_shvestky::DamMuPokuritJesteVice |
| 3689 | 0x005d09d0 | FUN_005d09d0 | police_manager_record_clear_pursuit_state | C_shvestky::DobreMiPokuril_MuzeJitDom |
| 3690 | 0x005d0b00 | FUN_005d0b00 | police_manager_try_dispatch_pursuit_for_target | C_shvestky::VasekSelChlastatNaOslavu |
| 3691 | 0x005d1100 | FUN_005d1100 | police_manager_record_begin_wanted_substate | C_shvestky::NabonzovalJsemHo |
| 3692 | 0x005d11c0 | FUN_005d11c0 | police_manager_register_assault_on_police_officer_for_target | C_shvestky::DalJsemPochuzkariDoDrzky |
| 3693 | 0x005d1220 | FUN_005d1220 | police_manager_register_assault_on_police_car_for_target | C_shvestky::StrcilJsemHoPoldumDoVyfuku |
| 3694 | 0x005d12d0 | FUN_005d12d0 | police_manager_record_set_tracked_vehicle_for_target | C_shvestky::ZalozSpisATahniDoRiti_Haleluja |
| 3695 | 0x005d1340 | FUN_005d1340 | police_manager_are_cops_after_player | C_shvestky::IdePoliciePoPlayeroviBoPumper |
| 3696 | 0x005d13b0 | FUN_005d13b0 | police_manager_rename_police_car | C_shvestky::RenamePoliceCar |
| 3697 | 0x005d1420 | FUN_005d1420 | roads_get_node_or_crossing_pos |  |
| 3698 | 0x005d1470 | FUN_005d1470 | std_vector4_erase_at |  |
| 3699 | 0x005d14a0 | FUN_005d14a0 | crt_static_init_guard_005d14a0 |  |
| 3700 | 0x005d14c0 | FUN_005d14c0 | crt_atexit_register_005d14c0 |  |
| 3701 | 0x005d14d0 | FUN_005d14d0 | input_ctor |  |
| 3702 | 0x005d1560 | FUN_005d1560 | input_dtor |  |
| 3703 | 0x005d1610 | FUN_005d1610 | input_release_all |  |
| 3704 | 0x005d1740 | FUN_005d1740 | input_set_mouse_axis_range |  |
| 3705 | 0x005d1770 | FUN_005d1770 | input_init_devices |  |
| 3706 | 0x005d1a70 | FUN_005d1a70 | input_clear_bindings |  |
| 3707 | 0x005d1b20 | FUN_005d1b20 | input_get_action_values |  |
| 3708 | 0x005d1bc0 | FUN_005d1bc0 | input_get_action_value |  |
| 3709 | 0x005d1c30 | FUN_005d1c30 | input_set_binding |  |
| 3710 | 0x005d1ec0 | FUN_005d1ec0 | input_get_binding |  |
| 3711 | 0x005d1f30 | FUN_005d1f30 | input_update |  |
| 3712 | 0x005d2ad0 | FUN_005d2ad0 | input_poll_joysticks |  |
| 3713 | 0x005d2b40 | FUN_005d2b40 | input_capture_binding |  |
| 3714 | 0x005d3180 | FUN_005d3180 | input_binding_get_label |  |
| 3715 | 0x005d3990 | FUN_005d3990 | input_joy_states_copy_range |  |
| 3716 | 0x005d39d0 | FUN_005d39d0 | input_joy_states_fill_n |  |
| 3717 | 0x005d3a10 | FUN_005d3a10 | input_joy_states_copy_one |  |
| 3718 | 0x005d3a30 | FUN_005d3a30 | crt_static_init_guard_005d3a30 |  |
| 3719 | 0x005d3a50 | FUN_005d3a50 | crt_atexit_register_005d3a50 |  |
| 3720 | 0x005d3a60 | FUN_005d3a60 | console_registry_static_init |  |
| 3721 | 0x005d3a70 | FUN_005d3a70 | console_registry_static_ctor |  |
| 3722 | 0x005d3a90 | FUN_005d3a90 | console_registry_register_atexit_dtor |  |
| 3723 | 0x005d3aa0 | FUN_005d3aa0 | console_registry_static_dtor |  |
| 3724 | 0x005d3ac0 | FUN_005d3ac0 | console_ctor | C_console::{ctor} |
| 3725 | 0x005d3b20 | FUN_005d3b20 | console_dtor | C_console::{dtor} |
| 3726 | 0x005d3b70 | FUN_005d3b70 | console_clear | C_console::Clear |
| 3727 | 0x005d3bf0 | FUN_005d3bf0 | console_tick | C_console::Tick |
| 3728 | 0x005d3ce0 | FUN_005d3ce0 | console_render | C_console::Render |
| 3729 | 0x005d4230 | FUN_005d4230 | console_register | ConsoleReg |
| 3730 | 0x005d4440 | FUN_005d4440 | console_unregister | ConsoleUnReg |
| 3731 | 0x005d44f0 | FUN_005d44f0 | console_render_all | ConsoleRender |
| 3732 | 0x005d45c0 | FUN_005d45c0 | console_load_font | ConsolesInit |
| 3733 | 0x005d4620 | FUN_005d4620 | console_font_init | ConsolesInit |
| 3734 | 0x005d4630 | FUN_005d4630 | console_font_release | ConsolesDone |
| 3735 | 0x005d4640 | FUN_005d4640 | console_unused_vector_static_init |  |
| 3736 | 0x005d4650 | FUN_005d4650 | console_unused_vector_static_ctor |  |
| 3737 | 0x005d4670 | FUN_005d4670 | console_unused_vector_register_atexit_dtor |  |
| 3738 | 0x005d4680 | FUN_005d4680 | console_unused_vector_static_dtor |  |
| 3739 | 0x005d46a0 | FUN_005d46a0 | copy_range_stride_0xc |  |
| 3740 | 0x005d46e0 | FUN_005d46e0 | fill_n_stride_0xc |  |
| 3741 | 0x005d4720 | FUN_005d4720 | crt_static_init_guard_005d4720 |  |
| 3742 | 0x005d4740 | FUN_005d4740 | crt_atexit_register_005d4740 |  |
| 3743 | 0x005d4750 | FUN_005d4750 | crt_static_init_guard_005d4750 |  |
| 3744 | 0x005d4770 | FUN_005d4770 | crt_atexit_register_005d4770 |  |
| 3745 | 0x005d4780 | thunk_FUN_005d4790 | thunk_stub_noop_cdecl_005d4790 |  |
| 3746 | 0x005d4790 | FUN_005d4790 | stub_noop_cdecl_005d4790 |  |
| 3747 | 0x005d47a0 | thunk_FUN_005d47b0 | thunk_stub_noop_cdecl_005d47b0 |  |
| 3748 | 0x005d47b0 | FUN_005d47b0 | stub_noop_cdecl_005d47b0 |  |
| 3749 | 0x005d47c0 | thunk_FUN_005d47d0 | thunk_stub_noop_cdecl_005d47d0 |  |
| 3750 | 0x005d47d0 | FUN_005d47d0 | stub_noop_cdecl_005d47d0 |  |
| 3751 | 0x005d47e0 | thunk_FUN_005d47f0 | thunk_stub_noop_cdecl_005d47f0 |  |
| 3752 | 0x005d47f0 | FUN_005d47f0 | stub_noop_cdecl_005d47f0 |  |
| 3753 | 0x005d4800 | FUN_005d4800 | camera_dialog_actors_static_init |  |
| 3754 | 0x005d4810 | FUN_005d4810 | camera_dialog_actors_static_ctor |  |
| 3755 | 0x005d4830 | FUN_005d4830 | camera_dialog_actors_register_atexit_dtor |  |
| 3756 | 0x005d4840 | FUN_005d4840 | camera_dialog_actors_static_dtor |  |
| 3757 | 0x005d4860 | thunk_FUN_005d4870 | thunk_stub_noop_cdecl_005d4870 |  |
| 3758 | 0x005d4870 | FUN_005d4870 | stub_noop_cdecl_005d4870 |  |
| 3759 | 0x005d4880 | thunk_FUN_005d4890 | thunk_stub_noop_cdecl_005d4890 |  |
| 3760 | 0x005d4890 | FUN_005d4890 | stub_noop_cdecl_005d4890 |  |
| 3761 | 0x005d48a0 | FUN_005d48a0 | camera_ctor | G_Camera::{ctor} |
| 3762 | 0x005d4910 | FUN_005d4910 | camera_dtor | G_Camera::{dtor} |
| 3763 | 0x005d4950 | FUN_005d4950 | camera_reset | G_Camera::Reset |
| 3764 | 0x005d4a30 | FUN_005d4a30 | camera_game_done | G_Camera::GameDone |
| 3765 | 0x005d4a60 | FUN_005d4a60 | camera_set_i3d_camera | G_Camera::SetCamera |
| 3766 | 0x005d4a80 | FUN_005d4a80 | camera_set_player | G_Camera::SetPlayer |
| 3767 | 0x005d4aa0 | FUN_005d4aa0 | camera_set_car | G_Camera::SetCar |
| 3768 | 0x005d4af0 | FUN_005d4af0 | camera_link_to_frame | G_Camera::Link |
| 3769 | 0x005d4b20 | FUN_005d4b20 | camera_set_swing | G_Camera::SetSwing |
| 3770 | 0x005d4c40 | FUN_005d4c40 | camera_look_around | G_Camera::LookAround |
| 3771 | 0x005d4cf0 | FUN_005d4cf0 | camera_is_look_from_eyes | G_Camera::LookFromEyes |
| 3772 | 0x005d4d10 | FUN_005d4d10 | camera_set_mode | G_Camera::SetMode |
| 3773 | 0x005d4d70 | FUN_005d4d70 | camera_set_climbing_into_car | G_Camera::LezeDoToyoty |
| 3774 | 0x005d4dd0 | FUN_005d4dd0 | camera_tick | G_Camera::Tick |
| 3775 | 0x005d9710 | FUN_005d9710 | camera_apply_desired_pos_car | G_Camera::WantPosCar |
| 3776 | 0x005d9b90 | FUN_005d9b90 | camera_apply_desired_pos | G_Camera::WantPos |
| 3777 | 0x005da4b0 | FUN_005da4b0 | camera_begin_mode | G_Camera::Begin |
| 3778 | 0x005daa60 | FUN_005daa60 | camera_end_mode | G_Camera::End |
| 3779 | 0x005daad0 | FUN_005daad0 | camera_enable_free_look | G_Camera::EnableFreeLook |
| 3780 | 0x005dab30 | FUN_005dab30 | camera_free_look | G_Camera::FreeLook |
| 3781 | 0x005dae20 | FUN_005dae20 | camera_free_look_base | G_Camera::FreeLookBase |
| 3782 | 0x005db000 | FUN_005db000 | camera_lock_at | G_Camera::LockAt |
| 3783 | 0x005db080 | FUN_005db080 | camera_lock_at_human_pair | G_Camera::CumNaNe |
| 3784 | 0x005db5d0 | FUN_005db5d0 | camera_unlock | G_Camera::Unlock |
| 3785 | 0x005db5f0 | FUN_005db5f0 | camera_push_mode | G_Camera::PushMode |
| 3786 | 0x005db820 | FUN_005db820 | camera_pop_mode | G_Camera::PopMode |
| 3787 | 0x005db8e0 | FUN_005db8e0 | camera_dialog_look | G_Camera::DialogLook |
| 3788 | 0x005db970 | FUN_005db970 | camera_dialog_start | G_Camera::DialogStart |
| 3789 | 0x005dbc50 | FUN_005dbc50 | camera_dialog_stop | G_Camera::DialogStop |
| 3790 | 0x005dbc90 | FUN_005dbc90 | camera_is_in_dialog | G_Camera::IsInDialog |
| 3791 | 0x005dbca0 | FUN_005dbca0 | camera_add_race_camera | G_Camera::AddRaceCamera |
| 3792 | 0x005dbe70 | FUN_005dbe70 | camera_remove_race_camera | G_Camera::RemoveRaceCamera |
| 3793 | 0x005dbed0 | FUN_005dbed0 | camera_init_race_cameras | G_Camera::InitRaceCameras |
| 3794 | 0x005dc000 | FUN_005dc000 | camera_set_look_to | G_Camera::SetLookTo |
| 3795 | 0x005dc070 | FUN_005dc070 | camera_set_sniper_fov | G_Camera::SetSniperFov |
| 3796 | 0x005dc0b0 | FUN_005dc0b0 | camera_set_sniper_mode | G_Camera::SetSniperMode |
| 3797 | 0x005dc0f0 | FUN_005dc0f0 | camera_get_race_camera_at |  |
| 3798 | 0x005dc120 | FUN_005dc120 | copy_range_stride_0x14 |  |
| 3799 | 0x005dc160 | FUN_005dc160 | crt_static_init_guard_005dc160 |  |
| 3800 | 0x005dc180 | FUN_005dc180 | crt_atexit_register_005dc180 |  |
| 3801 | 0x005dc190 | FUN_005dc190 | console_log_static_init |  |
| 3802 | 0x005dc1a0 | FUN_005dc1a0 | console_log_static_ctor |  |
| 3803 | 0x005dc1b0 | FUN_005dc1b0 | console_log_register_atexit_dtor |  |
| 3804 | 0x005dc1c0 | FUN_005dc1c0 | console_log_static_dtor |  |
| 3805 | 0x005dc1d0 | FUN_005dc1d0 | console_overlay_static_init |  |
| 3806 | 0x005dc1e0 | FUN_005dc1e0 | console_overlay_static_ctor |  |
| 3807 | 0x005dc1f0 | FUN_005dc1f0 | console_overlay_register_atexit_dtor |  |
| 3808 | 0x005dc200 | FUN_005dc200 | console_overlay_static_dtor |  |
| 3809 | 0x005dc210 | FUN_005dc210 | game_set_human | C_game::SetHuman |
| 3810 | 0x005dc240 | FUN_005dc240 | game_init | C_game::Init |
| 3811 | 0x005df210 | FUN_005df210 | gui_show_load_progress | _show_progress_ |
| 3812 | 0x005df240 | FUN_005df240 | game_create_easter_egg_detectors |  |
| 3813 | 0x005df610 | FUN_005df610 | game_enum_sectors_for_projector_lights | C_game::S_hlp_3427game_cpp::cbEnumSectors |
| 3814 | 0x005df650 | FUN_005df650 | anim_track_load_by_name |  |
| 3815 | 0x005df6b0 | FUN_005df6b0 | game_done | C_game::Done |
| 3816 | 0x005e0780 | FUN_005e0780 | game_done_intern | C_game::DoneIntern |
| 3817 | 0x005e0ad0 | FUN_005e0ad0 | game_change_act_state | C_game::ChangeActState |
| 3818 | 0x005e0c20 | FUN_005e0c20 | game_tick | C_game::Tick |
| 3819 | 0x005e27e0 | FUN_005e27e0 | game_vehicle_track_tick | C_game::DoubleKiss |
| 3820 | 0x005e3220 | FUN_005e3220 | game_add_temporary_actor | C_game::AddTemporaryActor |
| 3821 | 0x005e3400 | FUN_005e3400 | game_remove_temporary_actor | C_game::RemoveTemporaryActor |
| 3822 | 0x005e35d0 | FUN_005e35d0 | game_add_temporary_actor_intern | C_game::InternTempActAdd |
| 3823 | 0x005e37a0 | FUN_005e37a0 | game_remove_temporary_actor_intern | C_game::InternTempActDel |
| 3824 | 0x005e3920 | FUN_005e3920 | game_remove_temporary_actor_now | C_game::_RemoveTemporaryActor |
| 3825 | 0x005e39c0 | FUN_005e39c0 | game_clear_temporary_actors | C_game::ClearTemporaryActors |
| 3826 | 0x005e3a50 | FUN_005e3a50 | game_update_controls | C_game::UpdateControls |
| 3827 | 0x005e3a90 | FUN_005e3a90 | game_cheat_check |  |
| 3828 | 0x005e3dc0 | FUN_005e3dc0 | game_set_traffic_visible | C_game::SetTrafficVisible |
| 3829 | 0x005e3df0 | FUN_005e3df0 | game_new_shoot | C_game::NewShoot |
| 3830 | 0x005e47c0 | FUN_005e47c0 | game_tick_shoot | C_game::TickShoot |
| 3831 | 0x005e6760 | FUN_005e6760 | game_new_explosion | C_game::NewExplosion |
| 3832 | 0x005e7070 | FUN_005e7070 | actor_apply_explosion_damage | _explosion |
| 3833 | 0x005e72e0 | FUN_005e72e0 | game_new_smoke_trail | C_game::NewSmokeTrail |
| 3834 | 0x005e7580 | FUN_005e7580 | game_new_small_fires | C_game::NewSmallFires |
| 3835 | 0x005e7800 | FUN_005e7800 | game_new_fire | C_game::NewFire |
| 3836 | 0x005e7a40 | FUN_005e7a40 | game_new_cartridge | C_game::NewCartidge |
| 3837 | 0x005e8010 | FUN_005e8010 | game_new_grenade | C_game::NewGrenade |
| 3838 | 0x005e8600 | FUN_005e8600 | game_play_3d_sound_by_name | C_game::Play3DSound |
| 3839 | 0x005e8790 | FUN_005e8790 | game_stop_3d_sound | C_game::Stop3DSound |
| 3840 | 0x005e8840 | FUN_005e8840 | game_play_3d_sound | C_game::Play3DSound |
| 3841 | 0x005e8a90 | FUN_005e8a90 | game_clear_effects | C_game::ClearEffects |
| 3842 | 0x005e8b40 | FUN_005e8b40 | game_fake_shadow_compare_by_distance |  |
| 3843 | 0x005e8b70 | FUN_005e8b70 | game_add_fake_shadow_buffer | C_game::Init |
| 3844 | 0x005e8da0 | FUN_005e8da0 | game_bind_fake_shadow_buffer |  |
| 3845 | 0x005e8e30 | FUN_005e8e30 | game_add_frame_for_shadow | C_game::AddFrameForShadow |
| 3846 | 0x005e91d0 | FUN_005e91d0 | game_set_frame_for_shadow | C_game::SetFrameForShadow |
| 3847 | 0x005e9350 | FUN_005e9350 | game_del_frame_for_shadow | C_game::DelFrameForShadow |
| 3848 | 0x005e9490 | FUN_005e9490 | actor_apply_net_guid_table |  |
| 3849 | 0x005e9540 | FUN_005e9540 | game_enum_traffic_signal_frames | cbSemaphores |
| 3850 | 0x005e9950 | FUN_005e9950 | game_load_traffic_signals | C_game::LoadSemaphores |
| 3851 | 0x005e99b0 | FUN_005e99b0 | game_free_traffic_signals | C_game::FreeSemaphores |
| 3852 | 0x005e9a60 | FUN_005e9a60 | game_tick_traffic_signals | C_game::TickSemaphores |
| 3853 | 0x005e9b20 | FUN_005e9b20 | game_load_city_music_areas | C_game::LoadCityMusicAreas |
| 3854 | 0x005e9b70 | FUN_005e9b70 | game_enum_city_music_areas | C_game::S_hlp_12237game_cpp::cbCityMusic |
| 3855 | 0x005e9d70 | FUN_005e9d70 | game_city_music_off | C_game::CityMusicOff |
| 3856 | 0x005e9df0 | FUN_005e9df0 | game_city_music_set_paused | C_game::CityMusicPaused |
| 3857 | 0x005e9e10 | FUN_005e9e10 | game_set_film_music | C_game::SetFilmMusic |
| 3858 | 0x005e9fd0 | FUN_005e9fd0 | game_get_film_music | C_game::GetFilmMusic |
| 3859 | 0x005ea010 | FUN_005ea010 | game_clear_film_music |  |
| 3860 | 0x005ea070 | FUN_005ea070 | game_clear_frame_from_fire | C_game::ClearFrameFromFireHole |
| 3861 | 0x005ea0c0 | FUN_005ea0c0 | game_load_differences |  |
| 3862 | 0x005eb480 | FUN_005eb480 | std_vector_deleter_ensure_alloc |  |
| 3863 | 0x005eb4a0 | FUN_005eb4a0 | std_vector_deleter_assign_alloc |  |
| 3864 | 0x005eb4e0 | FUN_005eb4e0 | game_clear_differences | C_game::ClearDiffData |
| 3865 | 0x005eb920 | FUN_005eb920 | game_free_frame_placement_recursive | C_game::S_hlp_13685game_cpp::cbFreeFramePlacement |
| 3866 | 0x005eb9d0 | FUN_005eb9d0 | game_global_program_run | C_game::GlobalProgramRun |
| 3867 | 0x005eba30 | FUN_005eba30 | game_invalidate_coll | C_game::InvalidateColl |
| 3868 | 0x005ebac0 | FUN_005ebac0 | game_invalidate_actor | C_game::InvalidateActor |
| 3869 | 0x005ebe90 | FUN_005ebe90 | game_invalidate_frame | C_game::InvalidateFrame |
| 3870 | 0x005ebeb0 | FUN_005ebeb0 | game_invalidate_frame_recursive | C_game::S_hlp_14436game_cpp::cbDelFrame |
| 3871 | 0x005ebf80 | FUN_005ebf80 | game_add_frame_placement | C_game::AddFramePlacement |
| 3872 | 0x005ec250 | FUN_005ec250 | game_del_frame_placement | C_game::DelFramePlacement |
| 3873 | 0x005ec2e0 | FUN_005ec2e0 | game_raise_timer_timeout_script_event | C_game::Timer_TimeOut |
| 3874 | 0x005ec300 | FUN_005ec300 | game_raise_player_fire_script_event | C_game::PFire_Fire |
| 3875 | 0x005ec320 | FUN_005ec320 | game_raise_horn_script_event | C_game::PHorn_Horn |
| 3876 | 0x005ec340 | FUN_005ec340 | game_raise_water_fall_script_event | C_game::WFall_Player |
| 3877 | 0x005ec360 | FUN_005ec360 | game_add_npc_kill_event | C_game::AddNPCKillEvent |
| 3878 | 0x005ec3d0 | FUN_005ec3d0 | game_do_npc_kill_event | C_game::Do_NPCKillEvent |
| 3879 | 0x005ec460 | FUN_005ec460 | game_add_driver_man | C_game::AddDriverMan |
| 3880 | 0x005ec700 | FUN_005ec700 | game_del_driver_man | C_game::DelDriverMan |
| 3881 | 0x005ec7e0 | FUN_005ec7e0 | game_create_track_effect | C_game::CreateTrackEffect |
| 3882 | 0x005ecfb0 | FUN_005ecfb0 | vec3_dot_difference | C_game::CreateTrackEffect |
| 3883 | 0x005ecff0 | FUN_005ecff0 | game_create_track_effect_between | C_game::CreateTrackEffect |
| 3884 | 0x005ed7e0 | FUN_005ed7e0 | vec3_make_basis_from_direction | C_game::CreateTrackEffect |
| 3885 | 0x005ed920 | FUN_005ed920 | game_create_grenade_track_effect | C_game::CreateTrackEffect_Grenade |
| 3886 | 0x005ed9d0 | FUN_005ed9d0 | game_projector_advance_ring_cursor |  |
| 3887 | 0x005ed9f0 | FUN_005ed9f0 | game_clear_projectors_for_frame | C_game::ClearProjectorsForFrame |
| 3888 | 0x005eda20 | FUN_005eda20 | game_inventory_stack_clear | C_game::InventoryStackClear |
| 3889 | 0x005eda90 | FUN_005eda90 | game_inventory_stack_push | C_game::InventoryStackPush |
| 3890 | 0x005edd30 | FUN_005edd30 | game_inventory_stack_pop | C_game::InventoryStackPop |
| 3891 | 0x005edfa0 | FUN_005edfa0 | game_float_stack_clear | C_game::FloatStackClear |
| 3892 | 0x005edfd0 | FUN_005edfd0 | game_float_stack_push | C_game::FloatStackPush |
| 3893 | 0x005ee1b0 | FUN_005ee1b0 | game_float_stack_pop | C_game::FloatStackPop |
| 3894 | 0x005ee210 | FUN_005ee210 | camera_snap_next_update |  |
| 3895 | 0x005ee220 | FUN_005ee220 | game_character_stack_clear | C_game::CharacterStackClear |
| 3896 | 0x005ee250 | FUN_005ee250 | game_character_stack_push | C_game::CharacterStackPush |
| 3897 | 0x005ee3d0 | FUN_005ee3d0 | game_character_stack_pop | C_game::CharacterStackPop |
| 3898 | 0x005ee690 | FUN_005ee690 | game_box_add | C_game::BoxPlacementAdd |
| 3899 | 0x005eeb70 | FUN_005eeb70 | game_box_del | C_game::BoxPlacementDel |
| 3900 | 0x005eec50 | FUN_005eec50 | game_box_recompile | C_game::BoxPlacementRecompile |
| 3901 | 0x005eeeb0 | FUN_005eeeb0 | game_box_dest_add | C_game::BoxPlacementDestAdd |
| 3902 | 0x005ef080 | FUN_005ef080 | game_box_dest_set_ou | C_game::BoxPlacementDestSetOU |
| 3903 | 0x005ef220 | FUN_005ef220 | game_box_repose_dest_player | C_game::BoxPlacementReposeDestPlayer |
| 3904 | 0x005ef340 | FUN_005ef340 | game_box_save_info | C_game::BoxPlacement_SaveInfo |
| 3905 | 0x005ef720 | FUN_005ef720 | game_box_del_info | C_game::BoxPlacement_DelInfo |
| 3906 | 0x005ef7a0 | FUN_005ef7a0 | game_box_restore_info | C_game::BoxPlacement_RestoreInfo |
| 3907 | 0x005ef890 | FUN_005ef890 | game_box_get_start_point | C_game::BoxPlacement_GetStartPoint |
| 3908 | 0x005ef910 | FUN_005ef910 | game_box_place_at_next_dest | C_game::BoxPlacementGetNextDest |
| 3909 | 0x005efca0 | FUN_005efca0 | game_box_get_enemy_dest | C_game::BoxPlacementGetEnemyDest |
| 3910 | 0x005efe20 | FUN_005efe20 | game_box_count_owned | C_game::BoxPlacement_GetBox |
| 3911 | 0x005efe60 | FUN_005efe60 | game_box_dest_count_owned | C_game::BoxPlacement_GetBoxDest |
| 3912 | 0x005efea0 | FUN_005efea0 | game_box_move_to_dest | C_game::BoxPlacement_MoveBoxToDest |
| 3913 | 0x005eff70 | FUN_005eff70 | game_box_recompile_for_car | C_game::BoxRecompileBoxForCar |
| 3914 | 0x005f0170 | FUN_005f0170 | game_box_count_owned_on_car | C_game::BoxPlacementPoPiciNaZkurvenemAute |
| 3915 | 0x005f01e0 | FUN_005f01e0 | game_death_actor_add | C_game::DeathActorAdd |
| 3916 | 0x005f03a0 | FUN_005f03a0 | game_process_death_actors | C_game::DeathActorProcess |
| 3917 | 0x005f0410 | FUN_005f0410 | game_get_actor_from_guid | C_game::GetActorFromGUID |
| 3918 | 0x005f0450 | FUN_005f0450 | game_save_game_get_size | C_game::SaveGameGetSize |
| 3919 | 0x005f05a0 | FUN_005f05a0 | game_save_game_save | C_game::SaveGameSave |
| 3920 | 0x005f0990 | FUN_005f0990 | game_save_game_load | C_game::SaveGameLoad |
| 3921 | 0x005f1090 | FUN_005f1090 | game_savegame_load_before | C_game::SaveGameLoadBefore |
| 3922 | 0x005f10d0 | FUN_005f10d0 | game_lightness_add | C_game::LightnessAdd |
| 3923 | 0x005f1370 | FUN_005f1370 | game_lightness_del | C_game::LightnessDel |
| 3924 | 0x005f1400 | FUN_005f1400 | game_lightness_clear | C_game::LightnessClear |
| 3925 | 0x005f1460 | FUN_005f1460 | game_lightness_tick | C_game::LightnessTick |
| 3926 | 0x005f1750 | FUN_005f1750 | game_lightness_apply_recursive |  |
| 3927 | 0x005f17c0 | FUN_005f17c0 | game_add_shoot_light | C_game::AddShootLight |
| 3928 | 0x005f1900 | FUN_005f1900 | game_use_light_cache_system | C_game::UseLightCacheSystem |
| 3929 | 0x005f1950 | FUN_005f1950 | game_light_cache_collect_recursive | C_game::S_hlp_18015game_cpp::cbLight |
| 3930 | 0x005f19c0 | FUN_005f19c0 | game_recompute_light_cache | C_game::RecomputeLightCache |
| 3931 | 0x005f1b10 | FUN_005f1b10 | game_media_create | C_game::CreateIShow |
| 3932 | 0x005f1d90 | FUN_005f1d90 | game_media_open | C_game::OpenIShow |
| 3933 | 0x005f1f40 | FUN_005f1f40 | game_media_destroy | C_game::DestroyIShow |
| 3934 | 0x005f2030 | FUN_005f2030 | game_media_destroy_all | C_game::DestroyIShowAll |
| 3935 | 0x005f2140 | FUN_005f2140 | game_media_get | C_game::GetIShow |
| 3936 | 0x005f21a0 | FUN_005f21a0 | game_media_store_volume | C_game::IShowStoreVolume |
| 3937 | 0x005f2200 | FUN_005f2200 | game_media_set_loop | C_game::IShowLoop |
| 3938 | 0x005f2280 | FUN_005f2280 | game_media_apply_volumes | C_game::IShowSetVolume |
| 3939 | 0x005f22e0 | FUN_005f22e0 | game_media_fade_all | C_game::IShowFadeDownAll |
| 3940 | 0x005f2390 | FUN_005f2390 | game_media_fade | C_game::IShowFade |
| 3941 | 0x005f2650 | FUN_005f2650 | game_load_road_blocks | C_game::LoadRoadBlocks |
| 3942 | 0x005f26d0 | FUN_005f26d0 | game_collect_road_block_node | C_game::S_hlp_18574game_cpp::cbRoadBlock |
| 3943 | 0x005f2790 | FUN_005f2790 | game_load_taxi_points | C_game::LoadTaxiPoints |
| 3944 | 0x005f27e0 | FUN_005f27e0 | game_collect_taxi_point | C_game::S_hlp_18659game_cpp::cbFindTaxiPoints |
| 3945 | 0x005f2990 | FUN_005f2990 | game_is_valid_taxi_passenger | C_game::IsValidTaxiPassenger |
| 3946 | 0x005f29d0 | FUN_005f29d0 | game_del_traffic_signal | C_game::DelSemaphore |
| 3947 | 0x005f2a80 | FUN_005f2a80 | game_load_telephones | C_game::LoadTelephones |
| 3948 | 0x005f2e30 | FUN_005f2e30 | game_collect_telephone | C_game::S_hlp_18990game_cpp::cbTelephones |
| 3949 | 0x005f30c0 | FUN_005f30c0 | game_telephone_list_erase_by_key | C_game::DelTelephones |
| 3950 | 0x005f3180 | FUN_005f3180 | game_telephone_list_contains_key | C_game::IsTelephoneExist |
| 3951 | 0x005f31e0 | FUN_005f31e0 | game_human_talk_add | C_game::AddHumanTalk |
| 3952 | 0x005f33b0 | FUN_005f33b0 | game_human_talk_del | C_game::DelHumanTalk |
| 3953 | 0x005f3440 | FUN_005f3440 | game_human_talk_stop_for_human | C_game::DeathHumanTalk |
| 3954 | 0x005f34e0 | FUN_005f34e0 | game_build_traffic_signal_model_idx | C_game::CreateSignalMarkIDX |
| 3955 | 0x005f3860 | FUN_005f3860 | game_destroy_control_signal | C_game::DestroyControlSignal |
| 3956 | 0x005f3b50 | FUN_005f3b50 | game_entity_block | C_game::EntityBlock |
| 3957 | 0x005f3bf0 | FUN_005f3bf0 | game_add_invisible_actor | C_game::AddInvisibleActor |
| 3958 | 0x005f3dd0 | FUN_005f3dd0 | game_flush_pending_actor_lookups | C_game::FindActorsVect |
| 3959 | 0x005f3e60 | FUN_005f3e60 | game_particle_add |  |
| 3960 | 0x005f4010 | FUN_005f4010 | game_particle_destroy_all |  |
| 3961 | 0x005f4070 | FUN_005f4070 | game_stop_all_sounds | C_game::StopAllSounds |
| 3962 | 0x005f40b0 | FUN_005f40b0 | game_pause_all_sounds | C_game::PauseAllSounds |
| 3963 | 0x005f41c0 | FUN_005f41c0 | game_pause_scene_sounds_recursive |  |
| 3964 | 0x005f4270 | FUN_005f4270 | game_play_all_sounds | C_game::PlayAllSounds |
| 3965 | 0x005f4400 | FUN_005f4400 | game_scratch_add | C_game::ScratchAdd |
| 3966 | 0x005f4830 | FUN_005f4830 | game_scratch_recompile | C_game::ScratchRecompile |
| 3967 | 0x005f48a0 | FUN_005f48a0 | game_scratch_delete_from_car | C_game::ScratchDeleteFromCar |
| 3968 | 0x005f48e0 | FUN_005f48e0 | game_scratch_get_free_idx | C_game::ScratchGetFreeIdx |
| 3969 | 0x005f4af0 | FUN_005f4af0 | game_add_created_frame | C_game::AddCreatedFrameToVector |
| 3970 | 0x005f4cc0 | FUN_005f4cc0 | game_del_created_frame | C_game::DelCreatedFrameToVector |
| 3971 | 0x005f4d30 | FUN_005f4d30 | game_load_scream_areas | C_game::LoadScreamAreas |
| 3972 | 0x005f4d90 | FUN_005f4d90 | game_collect_scream_area | C_game::S_hlp_20556game_cpp::cbScreamArea |
| 3973 | 0x005f5010 | FUN_005f5010 | game_tick_scream_areas | C_game::TickScreamAreas |
| 3974 | 0x005f51c0 | FUN_005f51c0 | game_apply_weather_detail_level | C_game::OsefujPocasi |
| 3975 | 0x005f5310 | FUN_005f5310 | game_dying_human_add | C_game::AddDeadHumanForTick |
| 3976 | 0x005f54d0 | FUN_005f54d0 | game_dying_human_del | C_game::DelDeadHumanForTick |
| 3977 | 0x005f5540 | FUN_005f5540 | game_dying_human_tick | C_game::TickDeadHumanForTick |
| 3978 | 0x005f57e0 | FUN_005f57e0 | game_add_hydrant | C_game::AddHydrant |
| 3979 | 0x005f5b40 | FUN_005f5b40 | game_del_all_hydrants | C_game::DelAllHydrants |
| 3980 | 0x005f5bf0 | FUN_005f5bf0 | game_freeride_score_set_on | C_game::ScoreSetOn |
| 3981 | 0x005f5c30 | FUN_005f5c30 | game_freeride_score_set | C_game::ScoreSet |
| 3982 | 0x005f5c60 | FUN_005f5c60 | game_set_camera_rot_repair | C_game::SetCameraRotRepair |
| 3983 | 0x005f5cd0 | FUN_005f5cd0 | game_clear_cartridges | C_game::ClearCartridges |
| 3984 | 0x005f5d30 | FUN_005f5d30 | compare_car_snd_by_distance | compare_car_snd |
| 3985 | 0x005f5d60 | FUN_005f5d60 | game_tick_car_sound_manager | C_game::TickCarSoundManager |
| 3986 | 0x005f5f60 | FUN_005f5f60 | game_item_blur_done | C_game::ItemBlurDone |
| 3987 | 0x005f5fe0 | FUN_005f5fe0 | game_item_blur_tick | C_game::ItemBlurTick |
| 3988 | 0x005f6690 | FUN_005f6690 | game_item_blur_del_item | C_game::ItemBlurDelItem |
| 3989 | 0x005f6770 | FUN_005f6770 | vec_clear_keep_capacity_005f6770 |  |
| 3990 | 0x005f67a0 | FUN_005f67a0 | vec_count_stride_0x34 |  |
| 3991 | 0x005f67d0 | FUN_005f67d0 | vec_clear_keep_capacity_005f67d0 |  |
| 3992 | 0x005f6800 | FUN_005f6800 | vec_erase_range_stride2 |  |
| 3993 | 0x005f6830 | FUN_005f6830 | std_vector4_insert_at |  |
| 3994 | 0x005f69d0 | FUN_005f69d0 | vec_erase_range_stride_0xc |  |
| 3995 | 0x005f6a20 | FUN_005f6a20 | std_vector5c_insert_at |  |
| 3996 | 0x005f6c40 | FUN_005f6c40 | vec_erase_at_stride0x18 |  |
| 3997 | 0x005f6c90 | FUN_005f6c90 | vec_erase_range_stride_0x14 |  |
| 3998 | 0x005f6ce0 | FUN_005f6ce0 | std_vector8_insert_at_005f6ce0 |  |
| 3999 | 0x005f6e90 | FUN_005f6e90 | std_vector14_insert_n |  |
| 4000 | 0x005f71a0 | FUN_005f71a0 | std_vector28_insert_n |  |
| 4001 | 0x005f74a0 | FUN_005f74a0 | copy_range_stride_0x10 |  |
| 4002 | 0x005f74f0 | FUN_005f74f0 | copy_range_stride_0x5c |  |
| 4003 | 0x005f7530 | FUN_005f7530 | copy_range_stride_0x18 |  |
| 4004 | 0x005f7570 | FUN_005f7570 | fill_n_stride_0x10 |  |
| 4005 | 0x005f75b0 | FUN_005f75b0 | stl_copy_range_stride_0x34 |  |
| 4006 | 0x005f75e0 | FUN_005f75e0 | stl_copy_range_stride_0x20 |  |
| 4007 | 0x005f7610 | FUN_005f7610 | stl_copy_range_stride_0x5c |  |
| 4008 | 0x005f7640 | FUN_005f7640 | stl_copy_range_stride_0x28 |  |
| 4009 | 0x005f7670 | FUN_005f7670 | stl_copy_range_stride_0x14 |  |
| 4010 | 0x005f76a0 | FUN_005f76a0 | vec_copy_range_stride0x10 |  |
| 4011 | 0x005f76e0 | FUN_005f76e0 | stl_copy_range_stride_0xc |  |
| 4012 | 0x005f7710 | FUN_005f7710 | stl_copy_construct_stride_0x14 |  |
| 4013 | 0x005f77b0 | FUN_005f77b0 | stl_copy_one_stride_0x5c |  |
| 4014 | 0x005f77d0 | FUN_005f77d0 | stl_copy_one_stride8 |  |
| 4015 | 0x005f77e0 | FUN_005f77e0 | std_vector4_copy_assign_at_0x04 |  |
| 4016 | 0x005f7980 | FUN_005f7980 | vec_owner_scalar_deleting_dtor_005f7980 |  |
| 4017 | 0x005f79a0 | FUN_005f79a0 | crt_static_init_guard_005f79a0 |  |
| 4018 | 0x005f79c0 | FUN_005f79c0 | crt_atexit_register_005f79c0 |  |
| 4019 | 0x005f79d0 | FUN_005f79d0 | crt_static_init_guard_005f79d0 |  |
| 4020 | 0x005f79f0 | FUN_005f79f0 | crt_atexit_register_005f79f0 |  |
| 4021 | 0x005f7a00 | FUN_005f7a00 | load_progress_callback | cbLoadScene |
| 4022 | 0x005f7bc0 | FUN_005f7bc0 | crt_static_init_guard_005f7bc0 |  |
| 4023 | 0x005f7be0 | FUN_005f7be0 | crt_atexit_register_005f7be0 |  |
| 4024 | 0x005f7bf0 | thunk_FUN_005f7c00 | thunk_ls3d_setup_static_init |  |
| 4025 | 0x005f7c00 | FUN_005f7c00 | ls3d_setup_static_init | __sinit_Main.cpp |
| 4026 | 0x005f7cd0 | FUN_005f7cd0 | mission_static_init |  |
| 4027 | 0x005f7ce0 | FUN_005f7ce0 | mission_static_ctor |  |
| 4028 | 0x005f7cf0 | FUN_005f7cf0 | mission_register_atexit_dtor |  |
| 4029 | 0x005f7d00 | FUN_005f7d00 | mission_static_dtor |  |
| 4030 | 0x005f7d10 | FUN_005f7d10 | i3d_model_cache_static_init |  |
| 4031 | 0x005f7d20 | FUN_005f7d20 | i3d_model_cache_static_ctor |  |
| 4032 | 0x005f7d30 | FUN_005f7d30 | i3d_model_cache_register_atexit_dtor |  |
| 4033 | 0x005f7d40 | FUN_005f7d40 | i3d_model_cache_static_dtor |  |
| 4034 | 0x005f7d50 | FUN_005f7d50 | i3d_model_cache_dtor | C_I3D_cache<9I3D_model>::{dtor} |
| 4035 | 0x005f7e70 | FUN_005f7e70 | i3d_model_cache_ctor | __sinit_Main.cpp |
| 4036 | 0x005f7eb0 | FUN_005f7eb0 | i3d_model_cache_scalar_deleting_dtor |  |
| 4037 | 0x005f7ed0 | FUN_005f7ed0 | i3d_sound_cache_static_init |  |
| 4038 | 0x005f7ee0 | FUN_005f7ee0 | i3d_sound_cache_static_ctor |  |
| 4039 | 0x005f7ef0 | FUN_005f7ef0 | i3d_sound_cache_register_atexit_dtor |  |
| 4040 | 0x005f7f00 | FUN_005f7f00 | i3d_sound_cache_static_dtor |  |
| 4041 | 0x005f7f10 | FUN_005f7f10 | i3d_sound_cache_dtor | C_I3D_cache<9I3D_sound>::{dtor} |
| 4042 | 0x005f8020 | FUN_005f8020 | i3d_sound_cache_ctor | __sinit_Main.cpp |
| 4043 | 0x005f8060 | FUN_005f8060 | i3d_sound_cache_scalar_deleting_dtor |  |
| 4044 | 0x005f8080 | FUN_005f8080 | i3d_anim_cache_static_init |  |
| 4045 | 0x005f8090 | FUN_005f8090 | i3d_anim_cache_static_ctor |  |
| 4046 | 0x005f80a0 | FUN_005f80a0 | i3d_anim_cache_register_atexit_dtor |  |
| 4047 | 0x005f80b0 | FUN_005f80b0 | i3d_anim_cache_static_dtor |  |
| 4048 | 0x005f80c0 | FUN_005f80c0 | i3d_anim_cache_dtor | C_I3D_cache<17I3D_animation_set>::{dtor} |
| 4049 | 0x005f81d0 | FUN_005f81d0 | i3d_anim_cache_ctor | __sinit_Main.cpp |
| 4050 | 0x005f8210 | FUN_005f8210 | i3d_anim_cache_scalar_deleting_dtor |  |
| 4051 | 0x005f8230 | FUN_005f8230 | input_static_init |  |
| 4052 | 0x005f8240 | FUN_005f8240 | input_static_ctor |  |
| 4053 | 0x005f8250 | FUN_005f8250 | input_register_atexit_dtor |  |
| 4054 | 0x005f8260 | FUN_005f8260 | input_static_dtor |  |
| 4055 | 0x005f8270 | FUN_005f8270 | joy_device_list_static_init |  |
| 4056 | 0x005f8280 | FUN_005f8280 | joy_device_list_static_ctor |  |
| 4057 | 0x005f82a0 | FUN_005f82a0 | joy_device_list_register_atexit_dtor |  |
| 4058 | 0x005f82b0 | FUN_005f82b0 | joy_device_list_static_dtor |  |
| 4059 | 0x005f82d0 | FUN_005f82d0 | screen_fade_static_init |  |
| 4060 | 0x005f82e0 | FUN_005f82e0 | screen_fade_static_ctor |  |
| 4061 | 0x005f8310 | FUN_005f8310 | screen_fade_register_atexit_dtor |  |
| 4062 | 0x005f8330 | thunk_FUN_005f8340 | thunk_stub_noop_cdecl_005f8340 |  |
| 4063 | 0x005f8340 | FUN_005f8340 | stub_noop_cdecl_005f8340 |  |
| 4064 | 0x005f8350 | thunk_FUN_005f8360 | thunk_stub_noop_cdecl_005f8360 |  |
| 4065 | 0x005f8360 | FUN_005f8360 | stub_noop_cdecl_005f8360 |  |
| 4066 | 0x005f8370 | FUN_005f8370 | force_feedback_manager_static_init |  |
| 4067 | 0x005f8380 | FUN_005f8380 | force_feedback_manager_static_ctor |  |
| 4068 | 0x005f8390 | FUN_005f8390 | force_feedback_manager_register_atexit_dtor |  |
| 4069 | 0x005f83a0 | FUN_005f83a0 | force_feedback_manager_static_dtor |  |
| 4070 | 0x005f83b0 | FUN_005f83b0 | engine_set_wbuffer | SetWBuffer |
| 4071 | 0x005f83e0 | FUN_005f83e0 | joy_devices_init |  |
| 4072 | 0x005f8660 | FUN_005f8660 | joy_devices_done |  |
| 4073 | 0x005f86c0 | FUN_005f86c0 | engine_draw_overlay_callback |  |
| 4074 | 0x005f8700 | FUN_005f8700 | engine_new_mission | InternMissionInit |
| 4075 | 0x005f9540 | FUN_005f9540 | engine_game_loop | GameLoopSingle |
| 4076 | 0x005f98e0 | FUN_005f98e0 | engine_igraph_error_callback |  |
| 4077 | 0x005f99b0 | FUN_005f99b0 | engine_main | main |
| 4078 | 0x005fa2a0 | FUN_005fa2a0 | ls3d_setup_set_defaults |  |
| 4079 | 0x005fa320 | FUN_005fa320 | engine_init |  |
| 4080 | 0x005fb330 | FUN_005fb330 | engine_show_init_error | InitErrorMessageBox |
| 4081 | 0x005fb4f0 | FUN_005fb4f0 | engine_shutdown |  |
| 4082 | 0x005fb6e0 | FUN_005fb6e0 | i3d_cache_model_create | C_I3D_cache<9I3D_model>::Create |
| 4083 | 0x005fbaf0 | FUN_005fbaf0 | i3d_cache_model_add_dir | C_I3D_cache<9I3D_model>::AddDir |
| 4084 | 0x005fbcb0 | FUN_005fbcb0 | string_base_init_concat |  |
| 4085 | 0x005fbd10 | FUN_005fbd10 | i3d_cache_model_del_dir | C_I3D_cache<9I3D_model>::DelDir |
| 4086 | 0x005fbf20 | FUN_005fbf20 | string_ref_assign |  |
| 4087 | 0x005fbf60 | FUN_005fbf60 | i3d_cache_model_resolve_name |  |
| 4088 | 0x005fc260 | FUN_005fc260 | i3d_cache_base_model_dtor | C_I3D_cache_base<9I3D_model>::{dtor} |
| 4089 | 0x005fc270 | FUN_005fc270 | i3d_cache_sound_create | C_I3D_cache<9I3D_sound>::Create |
| 4090 | 0x005fc680 | FUN_005fc680 | i3d_cache_sound_add_dir | C_I3D_cache<9I3D_sound>::AddDir |
| 4091 | 0x005fc830 | FUN_005fc830 | i3d_cache_sound_del_dir | C_I3D_cache<9I3D_sound>::DelDir |
| 4092 | 0x005fca40 | FUN_005fca40 | i3d_cache_sound_resolve_name |  |
| 4093 | 0x005fcd30 | FUN_005fcd30 | i3d_cache_base_sound_dtor | C_I3D_cache_base<9I3D_sound>::{dtor} |
| 4094 | 0x005fcd40 | FUN_005fcd40 | i3d_cache_set_size | C_I3D_cache<9I3D_model>::SetCacheSize |
| 4095 | 0x005fcd50 | FUN_005fcd50 | i3d_cache_anim_open | C_I3D_cache<17I3D_animation_set>::Open |
| 4096 | 0x005fd180 | FUN_005fd180 | i3d_cache_clear | C_I3D_cache<9I3D_model>::Clear |
| 4097 | 0x005fd210 | FUN_005fd210 | i3d_cache_anim_add_dir | C_I3D_cache<17I3D_animation_set>::AddDir |
| 4098 | 0x005fd3c0 | FUN_005fd3c0 | i3d_cache_anim_del_dir | C_I3D_cache<17I3D_animation_set>::DelDir |
| 4099 | 0x005fd5d0 | FUN_005fd5d0 | i3d_cache_clear_dirs | C_I3D_cache<9I3D_model>::ClearDirs |
| 4100 | 0x005fd640 | FUN_005fd640 | i3d_cache_num_dirs | C_I3D_cache<9I3D_model>::NumDirs |
| 4101 | 0x005fd660 | FUN_005fd660 | i3d_cache_get_dir | C_I3D_cache<9I3D_model>::GetDir |
| 4102 | 0x005fd680 | FUN_005fd680 | i3d_cache_anim_resolve_name |  |
| 4103 | 0x005fd970 | FUN_005fd970 | i3d_cache_base_anim_dtor | C_I3D_cache_base<17I3D_animation_set>::{dtor} |
| 4104 | 0x005fd980 | FUN_005fd980 | i3d_model_cache_duplicate | C_I3D_model_cache::Duplicate |
| 4105 | 0x005fd9b0 | FUN_005fd9b0 | i3d_model_cache_cb_null_data | C_I3D_model_cache::cbNullData |
| 4106 | 0x005fd9c0 | FUN_005fd9c0 | i3d_model_cache_create_new | C_I3D_model_cache::CreateNew |
| 4107 | 0x005fd9d0 | FUN_005fd9d0 | i3d_sound_cache_duplicate | C_I3D_sound_cache::Duplicate |
| 4108 | 0x005fda10 | FUN_005fda10 | i3d_sound_cache_create_new | C_I3D_sound_cache::CreateNew |
| 4109 | 0x005fda20 | FUN_005fda20 | i3d_anim_cache_create_new | C_I3D_anim_cache::CreateNew |
| 4110 | 0x005fda30 | FUN_005fda30 | i3d_cache_model_scalar_deleting_dtor |  |
| 4111 | 0x005fda50 | FUN_005fda50 | i3d_cache_model_dtor | C_I3D_cache<9I3D_model>::{dtor} |
| 4112 | 0x005fdb60 | FUN_005fdb60 | i3d_cache_base_model_scalar_deleting_dtor |  |
| 4113 | 0x005fdb80 | FUN_005fdb80 | i3d_cache_sound_scalar_deleting_dtor |  |
| 4114 | 0x005fdba0 | FUN_005fdba0 | i3d_cache_sound_dtor | C_I3D_cache<9I3D_sound>::{dtor} |
| 4115 | 0x005fdcb0 | FUN_005fdcb0 | i3d_cache_base_sound_scalar_deleting_dtor |  |
| 4116 | 0x005fdcd0 | FUN_005fdcd0 | i3d_cache_anim_scalar_deleting_dtor |  |
| 4117 | 0x005fdcf0 | FUN_005fdcf0 | i3d_cache_anim_dtor | C_I3D_cache<17I3D_animation_set>::{dtor} |
| 4118 | 0x005fde00 | FUN_005fde00 | i3d_cache_base_anim_scalar_deleting_dtor |  |
| 4119 | 0x005fde20 | FUN_005fde20 | string_vector_destroy |  |
| 4120 | 0x005fde80 | FUN_005fde80 | i3d_cache_items_vector_destroy |  |
| 4121 | 0x005fdec0 | FUN_005fdec0 | string_range_release |  |
| 4122 | 0x005fdef0 | FUN_005fdef0 | string_vector_insert_n |  |
| 4123 | 0x005fe190 | FUN_005fe190 | string_range_uninit_copy |  |
| 4124 | 0x005fe1d0 | FUN_005fe1d0 | string_range_uninit_fill_n |  |
| 4125 | 0x005fe200 | FUN_005fe200 | anim_track_table_copy_assign_range_005fe200 |  |
| 4126 | 0x005fe420 | FUN_005fe420 | string_range_assign |  |
| 4127 | 0x005fe460 | FUN_005fe460 | string_range_copy_backward |  |
| 4128 | 0x005fe4b0 | FUN_005fe4b0 | string_ref_construct_at |  |
| 4129 | 0x005fe4c0 | FUN_005fe4c0 | string_ref_scalar_deleting_dtor |  |
| 4130 | 0x005fe4f0 | FUN_005fe4f0 | crt_static_init_guard_005fe4f0 |  |
| 4131 | 0x005fe510 | FUN_005fe510 | crt_atexit_register_005fe510 |  |
| 4132 | 0x005fe520 | FUN_005fe520 | crt_static_init_guard_005fe520 |  |
| 4133 | 0x005fe540 | FUN_005fe540 | crt_atexit_register_005fe540 |  |
| 4134 | 0x005fe550 | FUN_005fe550 | frame_map_key_less |  |
| 4135 | 0x005fe5b0 | FUN_005fe5b0 | mission_ctor | C_mission::{ctor} |
| 4136 | 0x005fe6b0 | FUN_005fe6b0 | mission_dtor | C_mission::{dtor} |
| 4137 | 0x005fe7d0 | FUN_005fe7d0 | mission_init | C_mission::Init |
| 4138 | 0x005fe950 | FUN_005fe950 | mission_uninit | C_mission::UnInit |
| 4139 | 0x005feac0 | FUN_005feac0 | mission_add_anim_model | C_mission::AddAnimModel |
| 4140 | 0x005fecb0 | FUN_005fecb0 | mission_del_anim_model | C_mission::DelAnimModel |
| 4141 | 0x005fed10 | FUN_005fed10 | mission_clear_game_anim_models | C_mission::ClearGameAnimModels |
| 4142 | 0x005fed70 | FUN_005fed70 | mission_create_actor | C_mission::CreateActor |
| 4143 | 0x005ff400 | FUN_005ff400 | mission_add_actor | C_mission::AddActor |
| 4144 | 0x005ff5e0 | FUN_005ff5e0 | mission_unload_actor | C_mission::UnloadActor |
| 4145 | 0x005ff690 | FUN_005ff690 | mission_del_actor | C_mission::DelActor |
| 4146 | 0x005ff7a0 | FUN_005ff7a0 | mission_del_actors | C_mission::DelActors |
| 4147 | 0x005ff860 | FUN_005ff860 | mission_del_not_me_actors | C_mission::DelNotMeActors |
| 4148 | 0x005ff980 | FUN_005ff980 | mission_is_persistent_actor |  |
| 4149 | 0x005ff9f0 | FUN_005ff9f0 | mission_find_actor_by_name | C_mission::FindActorByName |
| 4150 | 0x005ffa80 | FUN_005ffa80 | mission_global_program_game_init | C_mission::GlobalProgramGameInit |
| 4151 | 0x005ffab0 | FUN_005ffab0 | mission_global_program_game_done | C_mission::GlobalProgramGameDone |
| 4152 | 0x005ffae0 | FUN_005ffae0 | mission_global_program_run | C_mission::GlobalProgramRun |
| 4153 | 0x005ffb40 | FUN_005ffb40 | mission_close | C_mission::Close |
| 4154 | 0x005ffd40 | FUN_005ffd40 | mission_tick | C_mission::Tick |
| 4155 | 0x005ffeb0 | FUN_005ffeb0 | scene_load_report_progress | LoadProgress |
| 4156 | 0x005fff30 | FUN_005fff30 | mission_open | C_mission::Open |
| 4157 | 0x00600540 | FUN_00600540 | scene_load_progress_callback | cbLoadScene |
| 4158 | 0x00600560 | FUN_00600560 | scene_fill_sector_bits | C_mission::S_hlp$4890Mission_cpp::cbSectorBitFill |
| 4159 | 0x006005a0 | FUN_006005a0 | mission_open_bin | C_mission::OpenBin |
| 4160 | 0x006047a0 | FUN_006047a0 | uv_scroll_channel_init |  |
| 4161 | 0x006047d0 | FUN_006047d0 | uv_scroll_add_channel |  |
| 4162 | 0x00604a10 | FUN_00604a10 | string_append_cstr_in_place |  |
| 4163 | 0x00604b20 | FUN_00604b20 | scene_register_frame_by_name_cb |  |
| 4164 | 0x00604c20 | FUN_00604c20 | frame_map_destroy |  |
| 4165 | 0x00604ce0 | FUN_00604ce0 | mission_fade_in_object | C_mission::AddTimeTransparentObject |
| 4166 | 0x00604ef0 | FUN_00604ef0 | mission_finish_object_fade_ins | C_mission::FinishTimeTransparentObjects |
| 4167 | 0x00604f70 | FUN_00604f70 | mission_update_settings | C_mission::UpdateSettings |
| 4168 | 0x00605010 | FUN_00605010 | mission_restore_settings | C_mission::RestoreSettings |
| 4169 | 0x00605060 | FUN_00605060 | mission_settings_set_details_scale |  |
| 4170 | 0x006050a0 | FUN_006050a0 | mission_settings_set_draw_distance_scale |  |
| 4171 | 0x006050e0 | FUN_006050e0 | mission_settings_set_effects_quality |  |
| 4172 | 0x00605110 | FUN_00605110 | mission_settings_set_sounds_quality |  |
| 4173 | 0x00605140 | FUN_00605140 | mission_settings_set_shadow_quality |  |
| 4174 | 0x006051a0 | FUN_006051a0 | mission_settings_get_shadow_quality_idx |  |
| 4175 | 0x006051f0 | FUN_006051f0 | mission_settings_set_particles_quality |  |
| 4176 | 0x00605220 | FUN_00605220 | mission_get_scene_bounds |  |
| 4177 | 0x00605250 | FUN_00605250 | scene_accumulate_frame_bounds |  |
| 4178 | 0x00605340 | FUN_00605340 | mission_actor_cache_store | C_mission::FrmActBuff_StoreActor |
| 4179 | 0x00605630 | FUN_00605630 | car_clear_all_seat_owners | ClearCarSeats |
| 4180 | 0x00605670 | FUN_00605670 | mission_model_cache_store | C_mission::FrmActBuff_StoreModel |
| 4181 | 0x006058e0 | FUN_006058e0 | mission_actor_cache_take | C_mission::FrmActBuff_GetActor |
| 4182 | 0x00605a30 | FUN_00605a30 | mission_model_cache_take | C_mission::FrmActBuff_GetModel |
| 4183 | 0x00605b30 | FUN_00605b30 | mission_active_actor_store | C_mission::FrmActBuff_StoreActiveActor |
| 4184 | 0x00605d40 | FUN_00605d40 | mission_active_actor_drop | C_mission::FrmActBuff_RemoveActiveActor |
| 4185 | 0x00605dd0 | FUN_00605dd0 | mission_active_actor_take | C_mission::FrmActBuff_IsActiveActor |
| 4186 | 0x00605e90 | FUN_00605e90 | mission_actor_sound_cache_store | C_mission::FrmActBuff_StoreActorSound |
| 4187 | 0x00606100 | FUN_00606100 | mission_actor_sound_cache_take | C_mission::FrmActBuff_GetActorSound |
| 4188 | 0x006061d0 | FUN_006061d0 | mission_carry_caches_clear | C_mission::FrmActBuff_ClearAll |
| 4189 | 0x00606370 | FUN_00606370 | i3d_frame_set_field_0x140_block |  |
| 4190 | 0x00606390 | FUN_00606390 | mission_get_uv_scroll |  |
| 4191 | 0x006063a0 | FUN_006063a0 | vec_dword_push_back_by_ref |  |
| 4192 | 0x00606550 | FUN_00606550 | vec_count_stride_0x10 |  |
| 4193 | 0x00606570 | FUN_00606570 | vec_count_stride_0xc |  |
| 4194 | 0x00606590 | FUN_00606590 | roads_nodes_clear |  |
| 4195 | 0x006065c0 | FUN_006065c0 | actor_mark_scene_placed |  |
| 4196 | 0x006065d0 | FUN_006065d0 | vec_index_addr_stride_0x8 |  |
| 4197 | 0x006065e0 | FUN_006065e0 | frame_map_at_or_insert |  |
| 4198 | 0x00606690 | FUN_00606690 | i3d_cache_items_clear |  |
| 4199 | 0x00606700 | FUN_00606700 | roads_crossings_erase_range |  |
| 4200 | 0x00606750 | FUN_00606750 | game_ctor | C_game::{ctor} |
| 4201 | 0x00606e20 | FUN_00606e20 | game_dtor | C_game::{dtor} |
| 4202 | 0x00607400 | FUN_00607400 | uv_scroll_clear_channels |  |
| 4203 | 0x00607450 | FUN_00607450 | game_subobject_0x208_dtor |  |
| 4204 | 0x00607500 | FUN_00607500 | game_subobject_0x240_dtor |  |
| 4205 | 0x00607530 | FUN_00607530 | frame_map_insert |  |
| 4206 | 0x00607610 | FUN_00607610 | frame_map_ctor |  |
| 4207 | 0x00607640 | FUN_00607640 | frame_map_begin |  |
| 4208 | 0x00607650 | FUN_00607650 | frame_map_pair_construct |  |
| 4209 | 0x00607670 | FUN_00607670 | i3d_cache_items_erase_range |  |
| 4210 | 0x00607710 | FUN_00607710 | chunk_get_dta_handle |  |
| 4211 | 0x00607720 | FUN_00607720 | chunk_read_dword_and_leave | C_chunk::RAscend |
| 4212 | 0x00607820 | FUN_00607820 | frame_map_insert_00607820 |  |
| 4213 | 0x00607a80 | FUN_00607a80 | frame_map_erase_node |  |
| 4214 | 0x00607e70 | FUN_00607e70 | frame_map_destroy_subtree |  |
| 4215 | 0x00607ec0 | FUN_00607ec0 | frame_map_init |  |
| 4216 | 0x00607f40 | FUN_00607f40 | stl_fill_n_stride_0x1c |  |
| 4217 | 0x00607f70 | FUN_00607f70 | frame_map_insert_node |  |
| 4218 | 0x006081e0 | FUN_006081e0 | frame_map_rotate_left |  |
| 4219 | 0x00608240 | FUN_00608240 | frame_map_rotate_right |  |
| 4220 | 0x006082a0 | FUN_006082a0 | frame_map_alloc_node |  |
| 4221 | 0x006082c0 | FUN_006082c0 | frame_map_iter_decrement |  |
| 4222 | 0x00608320 | FUN_00608320 | frame_map_iter_increment |  |
| 4223 | 0x00608370 | FUN_00608370 | string_ref_copy_construct |  |
| 4224 | 0x00608390 | FUN_00608390 | crt_static_init_guard |  |
| 4225 | 0x006083b0 | FUN_006083b0 | crt_register_atexit_stub |  |
| 4226 | 0x006083c0 | FUN_006083c0 | web_node_dtor | C_web_node::{dtor} |
| 4227 | 0x006083f0 | FUN_006083f0 | web_node_resize_connection_array | C_web_node::Resize |
| 4228 | 0x006084d0 | FUN_006084d0 | web_node_add_connection | C_web_node::AddConn |
| 4229 | 0x00608590 | FUN_00608590 | web_node_delete_connection | C_web_node::DeleteConn |
| 4230 | 0x006085d0 | FUN_006085d0 | web_node_find_connection_index | C_web_node::FindConnIdx |
| 4231 | 0x00608610 | FUN_00608610 | web_path_dtor | C_web_path::{dtor} |
| 4232 | 0x00608650 | FUN_00608650 | web_path_resize_node_array | C_web_path::Resize |
| 4233 | 0x006087e0 | FUN_006087e0 | web_path_add_node | C_web_path::AddNode |
| 4234 | 0x00608840 | FUN_00608840 | web_path_delete_node | C_web_path::DeleteNode |
| 4235 | 0x00608980 | FUN_00608980 | web_path_toggle_connection | C_web_path::CreateConnect |
| 4236 | 0x00608b00 | FUN_00608b00 | web_path_is_connection_legal | C_web_path::LegalConnect |
| 4237 | 0x00608bc0 | FUN_00608bc0 | web_path_validate_node_connections | C_web_path::ControlNodeConn |
| 4238 | 0x00608cf0 | FUN_00608cf0 | web_path_load_nodes | C_web_path::LoadNodes |
| 4239 | 0x00608f70 | FUN_00608f70 | web_path_merge_nodes | C_web_path::MergeNodes |
| 4240 | 0x00609210 | FUN_00609210 | web_path_recompute_connection_costs | C_web_path::RecompileDistance |
| 4241 | 0x00609340 | FUN_00609340 | web_path_clear_all_selections | C_web_path::ReleaseSelect |
| 4242 | 0x00609390 | FUN_00609390 | web_path_find_node_id | C_web_path::FindNodeID |
| 4243 | 0x006093d0 | FUN_006093d0 | web_path_find_path_by_roads | C_web_path::FindPath_FastByRoads |
| 4244 | 0x00609970 | FUN_00609970 | web_path_find_path | C_web_path::FindPath |
| 4245 | 0x00609f30 | FUN_00609f30 | web_path_find_path_with_cost_fn | C_web_path::FindPathCompFunc |
| 4246 | 0x0060a480 | FUN_0060a480 | web_path_find_path_to_goal_fn | C_web_path::FindPathCompFunc |
| 4247 | 0x0060a9d0 | FUN_0060a9d0 | web_path_find_selected_connection | C_web_path::FindSelectedLine |
| 4248 | 0x0060aa50 | FUN_0060aa50 | web_path_insert_node_in_connection | C_web_path::IncludeNode |
| 4249 | 0x0060ad00 | FUN_0060ad00 | web_path_set_connection_one_way | C_web_path::FlipOneWay |
| 4250 | 0x0060add0 | FUN_0060add0 | web_path_find_nearest_node | C_web_path::GetNearNode |
| 4251 | 0x0060afd0 | FUN_0060afd0 | web_path_sort_nodes_by_x | C_web_path::SortNodesByX |
| 4252 | 0x0060b120 | FUN_0060b120 | web_path_is_sorted_by_x | C_web_path::IsSorted |
| 4253 | 0x0060b150 | FUN_0060b150 | web_path_clear_marker_bits | C_web_path::DeRiciConns |
| 4254 | 0x0060b1a0 | FUN_0060b1a0 | stub_noop_args8_0060b1a0 |  |
| 4255 | 0x0060b1b0 | FUN_0060b1b0 | vector_uint_append_all | _STDVECTCOPY |
| 4256 | 0x0060b370 | FUN_0060b370 | web_graph_dfs_expand | nodeproc |
| 4257 | 0x0060b680 | FUN_0060b680 | web_path_find_path_jipo | C_web_path::FindPathCompFuncJIPO |
| 4258 | 0x0060b6f0 | FUN_0060b6f0 | web_conn_ctor | C_web_connection::{ctor} |
| 4259 | 0x0060b700 | FUN_0060b700 | crt_atexit_noop_dtor_0060b700 |  |
| 4260 | 0x0060b710 | FUN_0060b710 | web_node_ctor | C_web_node::{ctor} |
| 4261 | 0x0060b740 | FUN_0060b740 | crt_static_init_guard_0060b740 |  |
| 4262 | 0x0060b760 | FUN_0060b760 | crt_atexit_register_0060b760 |  |
| 4263 | 0x0060b770 | FUN_0060b770 | best_time_ctor | C_best_time::{ctor} |
| 4264 | 0x0060b7a0 | FUN_0060b7a0 | circuit_ctor | C_circuit::{ctor} |
| 4265 | 0x0060b880 | FUN_0060b880 | circuit_dtor | C_circuit::{dtor} |
| 4266 | 0x0060ba00 | FUN_0060ba00 | circuit_load_gates_from_scene | C_circuit::LoadGatesFromScene |
| 4267 | 0x0060bda0 | FUN_0060bda0 | circuit_get_first_gate | C_circuit::GetFirstGate |
| 4268 | 0x0060bdc0 | FUN_0060bdc0 | circuit_clear_gates | C_circuit::ClearGates |
| 4269 | 0x0060be10 | FUN_0060be10 | circuit_clear_paths | C_circuit::ClearPaths |
| 4270 | 0x0060be70 | FUN_0060be70 | circuit_clear_race_cams | C_circuit::ClearRaceCams |
| 4271 | 0x0060bed0 | FUN_0060bed0 | circuit_load_path | C_circuit::LoadPaths |
| 4272 | 0x0060bfb0 | FUN_0060bfb0 | circuit_determine_race_cams | C_circuit::DetermineRaceCams |
| 4273 | 0x0060c0a0 | FUN_0060c0a0 | circuit_save_best_times | C_circuit::SaveBestTimes |
| 4274 | 0x0060c130 | FUN_0060c130 | circuit_load_best_times | C_circuit::LoadBestTimesDta |
| 4275 | 0x0060c360 | FUN_0060c360 | circuit_get_class_qualify_time_min |  |
| 4276 | 0x0060c380 | FUN_0060c380 | circuit_get_class_qualify_time_max |  |
| 4277 | 0x0060c3a0 | FUN_0060c3a0 | champ_circuit_ctor | C_champ_circuit::{ctor} |
| 4278 | 0x0060c3e0 | FUN_0060c3e0 | champ_circuit_dtor | C_champ_circuit::{dtor} |
| 4279 | 0x0060c450 | FUN_0060c450 | champ_circuit_load_profile | C_champ_circuit::LoadProfile |
| 4280 | 0x0060c4c0 | FUN_0060c4c0 | champ_circuit_save_profile | C_champ_circuit::SaveProfile |
| 4281 | 0x0060c520 | FUN_0060c520 | champ_circuit_set_lock_state | C_champ_circuit::SetLockState |
| 4282 | 0x0060c540 | FUN_0060c540 | champ_circuit_final_positions_set | C_champ_circuit::FinalPositionsSet |
| 4283 | 0x0060c640 | FUN_0060c640 | championship_ctor | C_championship::{ctor} |
| 4284 | 0x0060c6a0 | FUN_0060c6a0 | championship_dtor | C_championship::{dtor} |
| 4285 | 0x0060c7d0 | FUN_0060c7d0 | championship_add_circuit | C_championship::AddCircuit |
| 4286 | 0x0060c980 | FUN_0060c980 | championship_load_profile | C_championship::LoadProfile |
| 4287 | 0x0060c9f0 | FUN_0060c9f0 | championship_save_profile | C_championship::SaveProfile |
| 4288 | 0x0060ca60 | FUN_0060ca60 | championship_set_opponents | C_championship::SetOpponents |
| 4289 | 0x0060cc60 | FUN_0060cc60 | racing_fill_champ_results | C_racing::FillChampResults |
| 4290 | 0x0060cda0 | FUN_0060cda0 | opponent_ctor | C_opponent::{ctor} |
| 4291 | 0x0060ce50 | FUN_0060ce50 | opponent_ctor_with_id | C_opponent::{ctor} |
| 4292 | 0x0060cf00 | FUN_0060cf00 | opponent_dtor | C_opponent::{dtor} |
| 4293 | 0x0060cf10 | FUN_0060cf10 | opponent_game_done | C_opponent::GameDone |
| 4294 | 0x0060cfa0 | FUN_0060cfa0 | opponent_game_init | C_opponent::GameInit |
| 4295 | 0x0060e000 | FUN_0060e000 | opponent_create_driver | C_opponent::CreateDriver |
| 4296 | 0x0060e0d0 | FUN_0060e0d0 | opponent_recover | C_opponent::Recover |
| 4297 | 0x0060eb40 | FUN_0060eb40 | opponent_do_recover | C_opponent::DoRecover |
| 4298 | 0x0060ed10 | FUN_0060ed10 | opponent_tick_recover | C_opponent::TickRecover |
| 4299 | 0x0060ee40 | FUN_0060ee40 | opponent_create_driver_path | C_opponent::CreateDriverPath |
| 4300 | 0x0060f6c0 | FUN_0060f6c0 | racing_set_race_camera_index | C_racing::SetRCamIndex |
| 4301 | 0x0060f750 | FUN_0060f750 | opponent_get_race_points | C_opponent::GetRacePoints |
| 4302 | 0x0060f7c0 | FUN_0060f7c0 | opponent_get_race_points_for_place | C_opponent::GetRacePoints |
| 4303 | 0x0060f830 | FUN_0060f830 | opponent_set_death | C_opponent::SetDeath |
| 4304 | 0x0060f910 | FUN_0060f910 | racing_ctor | C_racing::{ctor} |
| 4305 | 0x0060fb30 | FUN_0060fb30 | gif_vector_ctor |  |
| 4306 | 0x0060fb50 | FUN_0060fb50 | racing_dtor | C_racing::{dtor} |
| 4307 | 0x0060fc90 | FUN_0060fc90 | racing_load_circuits | C_racing::LoadCircuits |
| 4308 | 0x0060ff10 | FUN_0060ff10 | racing_load_champs | C_racing::LoadChamps |
| 4309 | 0x00610230 | FUN_00610230 | racing_get_circuit | C_racing::GetCircuit |
| 4310 | 0x00610260 | FUN_00610260 | racing_get_current_circuit | C_racing::GetCurrentCircuit |
| 4311 | 0x00610270 | FUN_00610270 | racing_set_current_champ | C_racing::SetCurrentChamp |
| 4312 | 0x006102f0 | FUN_006102f0 | racing_set_current_champ_circuit | C_racing::SetCurrentChampCircuit |
| 4313 | 0x00610370 | FUN_00610370 | racing_add_opponents | C_racing::AddOpponents |
| 4314 | 0x00610400 | FUN_00610400 | racing_add_racer_for_player |  |
| 4315 | 0x00610480 | FUN_00610480 | racing_add_racer_for_ai_slot |  |
| 4316 | 0x00610510 | FUN_00610510 | racing_bind_local_player_racer |  |
| 4317 | 0x00610540 | FUN_00610540 | racing_add_opponent | C_racing::AddOpponent |
| 4318 | 0x006105b0 | FUN_006105b0 | racing_generate_opponents | C_racing::GenerateOpponents |
| 4319 | 0x00610810 | FUN_00610810 | racing_clear_opponents | C_racing::ClearOpponents |
| 4320 | 0x00610870 | FUN_00610870 | racing_get_opponent_count | C_racing::GetOpponentCount |
| 4321 | 0x00610880 | FUN_00610880 | racing_fill_opponent_name_and_model | C_racing::FillOpponentNameModel |
| 4322 | 0x00610a10 | FUN_00610a10 | racing_place_cars | C_racing::PlaceCars |
| 4323 | 0x00610c00 | FUN_00610c00 | racing_init_racers | C_racing::InitRacers |
| 4324 | 0x00610d40 | FUN_00610d40 | race_compare_qualification_time | compareQtime |
| 4325 | 0x00610db0 | FUN_00610db0 | race_compare_result_points |  |
| 4326 | 0x00610dd0 | FUN_00610dd0 | race_compare_champ_points | compareChampPoint |
| 4327 | 0x00610e30 | FUN_00610e30 | race_compare_participant_slot |  |
| 4328 | 0x00610eb0 | FUN_00610eb0 | race_compare_position | comparePos |
| 4329 | 0x00610fc0 | FUN_00610fc0 | racing_load_champs_profile | C_racing::LoadChampsProfile |
| 4330 | 0x00611050 | FUN_00611050 | racing_save_champs_profile | C_racing::SaveChampsProfile |
| 4331 | 0x006110e0 | FUN_006110e0 | racing_load_best_times | C_racing::LoadBestTimes |
| 4332 | 0x00611140 | FUN_00611140 | racing_save_best_times | C_racing::SaveBestTimes |
| 4333 | 0x006111a0 | FUN_006111a0 | racing_can_run_race | C_racing::CanRunRace |
| 4334 | 0x006111e0 | FUN_006111e0 | racing_game_init | C_racing::GameInit |
| 4335 | 0x00611380 | FUN_00611380 | racing_game_done | C_racing::GameDone |
| 4336 | 0x00611430 | FUN_00611430 | racing_activate_racers | C_racing::ActivateRacers |
| 4337 | 0x00611630 | FUN_00611630 | opponent_play_death | C_opponent::PlayDeath |
| 4338 | 0x00611700 | FUN_00611700 | opponent_tick | C_opponent::Tick |
| 4339 | 0x006117e0 | FUN_006117e0 | opponent_tick_overtake | C_opponent::TickObjizdecka |
| 4340 | 0x00611d10 | FUN_00611d10 | opponent_end_overtake | C_opponent::EndOjed |
| 4341 | 0x00611f90 | FUN_00611f90 | opponent_test_node_vs_cars | C_opponent::ObjzTestNodeValidCars |
| 4342 | 0x00612100 | FUN_00612100 | opponent_test_node_vs_track_edges | C_opponent::ObjzTestNodeValid |
| 4343 | 0x00612290 | FUN_00612290 | opponent_overtake | C_opponent::Ojed |
| 4344 | 0x006137c0 | FUN_006137c0 | opponent_fill_path_node | C_opponent::FillObjzNode |
| 4345 | 0x00613880 | FUN_00613880 | opponent_get_next_driver_route | C_opponent::GetNextDriverRoute |
| 4346 | 0x006138d0 | FUN_006138d0 | opponent_get_prev_driver_route | C_opponent::GetPrevDriverRoute |
| 4347 | 0x00613920 | FUN_00613920 | opponent_get_next_driver_route_from | C_opponent::GetNextDriverRoute |
| 4348 | 0x00613970 | FUN_00613970 | opponent_get_prev_driver_route_from | C_opponent::GetPrevDriverRoute |
| 4349 | 0x006139c0 | FUN_006139c0 | racing_tick | C_racing::Tick |
| 4350 | 0x006141e0 | FUN_006141e0 | racing_next_finish_place |  |
| 4351 | 0x00614220 | FUN_00614220 | opponent_finish | C_opponent::Finish |
| 4352 | 0x00614280 | FUN_00614280 | racing_set_phase |  |
| 4353 | 0x00614290 | FUN_00614290 | racing_show_time_banner | C_racing::FlushText |
| 4354 | 0x00614340 | FUN_00614340 | racing_test_best_lap | C_racing::TestBestLap |
| 4355 | 0x00614660 | FUN_00614660 | racing_test_best_race | C_racing::TestBestRace |
| 4356 | 0x00614820 | FUN_00614820 | racing_end_race | C_racing::EndRace |
| 4357 | 0x00614db0 | FUN_00614db0 | racing_set_current_circuit | C_racing::SetCurrentCircuit |
| 4358 | 0x00614dd0 | FUN_00614dd0 | racing_generate_qualification | C_racing::GenerateQualification |
| 4359 | 0x00614ee0 | FUN_00614ee0 | racing_get_difficulty_scale | C_racing::GetDifficulty2 |
| 4360 | 0x00614f30 | FUN_00614f30 | racing_get_power_coeff_by_difficulty | C_racing::GetPowerCoeffByDifficulty |
| 4361 | 0x00614fd0 | FUN_00614fd0 | racing_pause_clock_begin |  |
| 4362 | 0x00614ff0 | FUN_00614ff0 | racing_pause_clock_end |  |
| 4363 | 0x00615020 | FUN_00615020 | racing_toggle_results_visibility | C_racing::GMenuTogleResultsVisibility |
| 4364 | 0x00615030 | FUN_00615030 | racing_get_results_visibility | C_racing::GMenuGetResultsVisibility |
| 4365 | 0x00615060 | FUN_00615060 | championship_get_results | C_championship::GetResults |
| 4366 | 0x00615080 | FUN_00615080 | championship_get_position | C_championship::GetChampionshipPos |
| 4367 | 0x006150a0 | FUN_006150a0 | racing_get_replay_setup_size |  |
| 4368 | 0x006150c0 | FUN_006150c0 | racing_save_replay_setup |  |
| 4369 | 0x00615120 | FUN_00615120 | racing_load_replay_setup | C_racing::LoadReplay |
| 4370 | 0x006151d0 | FUN_006151d0 | racing_get_current_circuit_id | C_racing::GetCurrentCircuitID |
| 4371 | 0x00615200 | FUN_00615200 | racing_get_current_champ_id | C_racing::GetCurrentChampID |
| 4372 | 0x00615230 | FUN_00615230 | racing_get_segment_id_for_car | GetSegmentIdFromCarIdG |
| 4373 | 0x006152d0 | FUN_006152d0 | racing_set_player_racer_model |  |
| 4374 | 0x00615330 | FUN_00615330 | racing_remove_racer_for_player |  |
| 4375 | 0x00615370 | FUN_00615370 | racing_remove_racer_for_ai_slot |  |
| 4376 | 0x006153b0 | FUN_006153b0 | racing_remove_opponent_at |  |
| 4377 | 0x006154c0 | FUN_006154c0 | racing_find_opponent_by_player |  |
| 4378 | 0x00615500 | FUN_00615500 | racing_find_opponent_by_ai_slot |  |
| 4379 | 0x00615540 | FUN_00615540 | line2d_intersect_params | PrusecikPrimek |
| 4380 | 0x00615610 | FUN_00615610 | seg2d_intersect_in_range | PrusecikUsecek |
| 4381 | 0x00615710 | FUN_00615710 | racing_set_damage_enabled | C_racing::SetDamage |
| 4382 | 0x00615810 | FUN_00615810 | racing_spawn_winners_scene |  |
| 4383 | 0x00616070 | FUN_00616070 | racing_release_winners_scene |  |
| 4384 | 0x006160e0 | FUN_006160e0 | racing_fake_final_positions | C_racing::FakeFinalPositions |
| 4385 | 0x00616150 | FUN_00616150 | racing_test_gate | C_racing::TestGate |
| 4386 | 0x00616200 | FUN_00616200 | racing_test_all_gates | C_racing::TestAllGates |
| 4387 | 0x00616280 | FUN_00616280 | racing_test_gate_crossing | C_racing::InternTestGate |
| 4388 | 0x00616510 | FUN_00616510 | racing_do_gates | C_racing::DoGates |
| 4389 | 0x00616ce0 | FUN_00616ce0 | racing_is_last_champ_circuit | C_racing::IsLastChampCircuit |
| 4390 | 0x00616d30 | FUN_00616d30 | racing_update_driver_power_coeff | C_racing::UpdateDriverPowerC |
| 4391 | 0x00616e10 | FUN_00616e10 | racing_get_distance_by_gates | C_racing::GetDistanceByGates |
| 4392 | 0x00616f20 | FUN_00616f20 | racing_predict_car_collision | C_racing::TestniPredvidaciKolize |
| 4393 | 0x00617430 | FUN_00617430 | racing_fill_driver_curve | C_racing::FillDriverCurve |
| 4394 | 0x00617a60 | FUN_00617a60 | crt_static_init_guard_00617a60 |  |
| 4395 | 0x00617a80 | FUN_00617a80 | crt_atexit_register_00617a80 |  |
| 4396 | 0x00617a90 | FUN_00617a90 | ram_compress_or_expand | Compress |
| 4397 | 0x00617db0 | FUN_00617db0 | crt_static_init_guard_00617db0 |  |
| 4398 | 0x00617dd0 | FUN_00617dd0 | crt_atexit_register_00617dd0 |  |
| 4399 | 0x00617de0 | FUN_00617de0 | roads_ctor | C_roads::{ctor} |
| 4400 | 0x00617e10 | FUN_00617e10 | roads_dtor | C_roads::{dtor} |
| 4401 | 0x00617e50 | FUN_00617e50 | roads_save_data | C_roads::SaveData |
| 4402 | 0x00617f70 | FUN_00617f70 | roads_load_data | C_roads::LoadData |
| 4403 | 0x00618320 | FUN_00618320 | roads_merge_data |  |
| 4404 | 0x00618730 | FUN_00618730 | roads_recompile_distances | C_roads::RecompileDistances |
| 4405 | 0x00619480 | FUN_00619480 | roads_get_near_node | C_roads::GetNearNode |
| 4406 | 0x00619680 | FUN_00619680 | roads_find_crossing | C_roads::FindCrossing |
| 4407 | 0x006196e0 | FUN_006196e0 | roads_find_road_slot_to_crossing | C_roads::NPC_FindCrossIdxToCross |
| 4408 | 0x00619820 | FUN_00619820 | roads_find_car_path_from_node_to_crossing | C_roads::NPC_FindPathFromNodeToCross |
| 4409 | 0x00619be0 | FUN_00619be0 | roads_find_car_path_to_next_crossing | C_roads::NPC_FindPathToNextCross1 |
| 4410 | 0x00619d20 | FUN_00619d20 | roads_fill_path_from_node_to_crossing | C_roads::FillPathFromNodeToCross |
| 4411 | 0x00619de0 | FUN_00619de0 | roads_find_crossing_from_node | C_roads::FindCrossingFromNode |
| 4412 | 0x00619f20 | FUN_00619f20 | roads_find_near_crossing_from_node | C_roads::FindNearCrossingFromNode |
| 4413 | 0x0061a070 | FUN_0061a070 | roads_find_path_between_crossings | C_roads::FindPathBeweenCross |
| 4414 | 0x0061a540 | FUN_0061a540 | roads_pathfind_entry_ctor | C_roads::S_pathfind$3865road_web_cpp::{ctor} |
| 4415 | 0x0061a550 | FUN_0061a550 | roads_fill_path_from_crossing_to_crossing | C_roads::FillPathFromCrossToCross |
| 4416 | 0x0061a5f0 | FUN_0061a5f0 | roads_find_path | C_roads::FindPath |
| 4417 | 0x0061a9a0 | FUN_0061a9a0 | roads_sort_nodes_by_x | C_roads::SortNodesByX |
| 4418 | 0x0061abb0 | FUN_0061abb0 | roads_is_sorted | C_roads::IsSorted |
| 4419 | 0x0061ac10 | FUN_0061ac10 | roads_disconnect_unused_road_links | C_roads::DestroyAllOffPaths |
| 4420 | 0x0061aca0 | FUN_0061aca0 | roads_crossings_insert_n | std::__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0>::insert |
| 4421 | 0x0061aff0 | FUN_0061aff0 | crt_static_init_guard_0061aff0 |  |
| 4422 | 0x0061b010 | FUN_0061b010 | crt_atexit_register_0061b010 |  |
| 4423 | 0x0061b020 | thunk_FUN_0061b030 | thunk_zero_vector_006f96b0_static_ctor |  |
| 4424 | 0x0061b030 | FUN_0061b030 | zero_vector_006f96b0_static_ctor |  |
| 4425 | 0x0061b050 | FUN_0061b050 | crt_static_init_guard_0061b050 |  |
| 4426 | 0x0061b070 | FUN_0061b070 | crt_atexit_register_0061b070 |  |
| 4427 | 0x0061b080 | thunk_FUN_0061b090 | thunk_dir8_unit_xz_006f96c0_static_ctor |  |
| 4428 | 0x0061b090 | FUN_0061b090 | dir8_unit_xz_006f96c0_static_ctor |  |
| 4429 | 0x0061b190 | thunk_FUN_0061b1a0 | thunk_dir8_square_xz_006f9590_static_ctor |  |
| 4430 | 0x0061b1a0 | FUN_0061b1a0 | dir8_square_xz_006f9590_static_ctor |  |
| 4431 | 0x0061b2a0 | thunk_FUN_0061b2b0 | thunk_dir16_square_xz_006f95f0_static_ctor |  |
| 4432 | 0x0061b2b0 | FUN_0061b2b0 | dir16_square_xz_006f95f0_static_ctor |  |
| 4433 | 0x0061b4a0 | FUN_0061b4a0 | sound_group_static_init | __sinit_tools.cpp |
| 4434 | 0x0061b4b0 | FUN_0061b4b0 | sound_group_static_ctor |  |
| 4435 | 0x0061b4d0 | FUN_0061b4d0 | sound_group_register_atexit_dtor |  |
| 4436 | 0x0061b4e0 | FUN_0061b4e0 | sound_group_static_dtor |  |
| 4437 | 0x0061b500 | FUN_0061b500 | sound_table_static_init |  |
| 4438 | 0x0061b510 | FUN_0061b510 | sound_table_static_ctor |  |
| 4439 | 0x0061b520 | FUN_0061b520 | sound_table_register_atexit_dtor |  |
| 4440 | 0x0061b530 | FUN_0061b530 | sound_table_static_dtor |  |
| 4441 | 0x0061b540 | FUN_0061b540 | compass_activate | C_compass::Activate |
| 4442 | 0x0061b5e0 | FUN_0061b5e0 | screen_fade_tick | C_silver_screen::Tick |
| 4443 | 0x0061b650 | FUN_0061b650 | fire_ctor | C_fire::{ctor} |
| 4444 | 0x0061b670 | FUN_0061b670 | fire_dtor | C_fire::{dtor} |
| 4445 | 0x0061b6a0 | FUN_0061b6a0 | fire_init | C_fire::Init |
| 4446 | 0x0061b850 | FUN_0061b850 | fire_tick | C_fire::Tick |
| 4447 | 0x0061bb30 | FUN_0061bb30 | using_object_add_object | C_using_object::AddObject |
| 4448 | 0x0061bce0 | FUN_0061bce0 | using_object_del_object | C_using_object::DelObject |
| 4449 | 0x0061bdb0 | FUN_0061bdb0 | using_object_clear_objects | C_using_object::ClearObjects |
| 4450 | 0x0061be50 | FUN_0061be50 | using_object_is_any_object_near | C_using_object::IsAnyObjectNear |
| 4451 | 0x0061bf50 | FUN_0061bf50 | using_object_find_near_objects |  |
| 4452 | 0x0061c1b0 | FUN_0061c1b0 | using_object_clear_invalid_items | C_using_object::ClearInvalidItems |
| 4453 | 0x0061c1e0 | FUN_0061c1e0 | using_object_add_invalid_item | C_using_object::AddInvalidItem |
| 4454 | 0x0061c390 | FUN_0061c390 | using_object_del_invalid_item | C_using_object::DelInvalidItem |
| 4455 | 0x0061c3f0 | FUN_0061c3f0 | using_object_is_invalid_item | C_using_object::IsInvalidItem |
| 4456 | 0x0061c430 | FUN_0061c430 | using_object_save_game_get_size | C_using_object::SaveGameGetSize |
| 4457 | 0x0061c4b0 | FUN_0061c4b0 | using_object_save_game_save | C_using_object::SaveGameSave |
| 4458 | 0x0061c750 | FUN_0061c750 | using_object_save_game_load | C_using_object::SaveGameLoad |
| 4459 | 0x0061cba0 | FUN_0061cba0 | sound_table_ctor | C_sound_table::{ctor} |
| 4460 | 0x0061cbd0 | FUN_0061cbd0 | sound_table_dtor | C_sound_table::{dtor} |
| 4461 | 0x0061cc30 | FUN_0061cc30 | sound_table_free_tables | C_sound_table::FreeTables |
| 4462 | 0x0061cd00 | FUN_0061cd00 | sound_table_add_group | C_sound_table::AddGroup |
| 4463 | 0x0061cd50 | FUN_0061cd50 | sound_table_del_group | C_sound_table::DelGroup |
| 4464 | 0x0061ce50 | FUN_0061ce50 | sound_table_add_sound | C_sound_table::AddSound |
| 4465 | 0x0061cef0 | FUN_0061cef0 | sound_table_del_sound | C_sound_table::DelSound |
| 4466 | 0x0061cfe0 | FUN_0061cfe0 | sound_table_get_record_by_id | C_sound_table::GetTableFromIDX |
| 4467 | 0x0061d040 | FUN_0061d040 | sound_table_set_file_name | C_sound_table::SetFileName |
| 4468 | 0x0061d0a0 | FUN_0061d0a0 | sound_table_sort_variants_by_weight | C_sound_table::OptimalizeTable |
| 4469 | 0x0061d160 | FUN_0061d160 | sound_table_load_tables | C_sound_table::LoadTables |
| 4470 | 0x0061d3b0 | FUN_0061d3b0 | sound_table_save_tables | C_sound_table::SaveTables |
| 4471 | 0x0061d5f0 | FUN_0061d5f0 | sound_table_construct_sound | C_sound_table::ConstructSound |
| 4472 | 0x0061d7d0 | thunk_FUN_0061d5f0 | thunk_sound_table_construct_sound | C_sound_table::TestConstructSound |
| 4473 | 0x0061d7e0 | FUN_0061d7e0 | sound_table_clear_invisible_flag | C_sound_table::ClearInvisibleFlag |
| 4474 | 0x0061d820 | FUN_0061d820 | sound_arena_emit_sound | soundArena::SoundEmit |
| 4475 | 0x0061d970 | FUN_0061d970 | sound_arena_update_entity_sound_contacts | soundArena::GetSounds |
| 4476 | 0x0061dac0 | FUN_0061dac0 | sound_arena_init | soundArena::Init |
| 4477 | 0x0061dae0 | FUN_0061dae0 | sound_arena_invalidate_actor | soundArena::InvalidateActor |
| 4478 | 0x0061db10 | FUN_0061db10 | sound_table_record_list_erase_range |  |
| 4479 | 0x0061db60 | FUN_0061db60 | std_vector10_erase_range |  |
| 4480 | 0x0061dbb0 | FUN_0061dbb0 | sound_table_get_num_sounds | C_sound_table::GetNumSounds |
| 4481 | 0x0061dbe0 | FUN_0061dbe0 | sound_table_record_list_insert_n | std::__vector_imp<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>,0>::insert |
| 4482 | 0x0061df00 | FUN_0061df00 | crt_static_init_guard_0061df00 |  |
| 4483 | 0x0061df20 | FUN_0061df20 | crt_atexit_register_0061df20 |  |
| 4484 | 0x0061df30 | FUN_0061df30 | uv_scroll_channel_tick | S_tshift_elem::Tick |
| 4485 | 0x0061e010 | FUN_0061e010 | folded_vector_ctor_at_0x04 |  |
| 4486 | 0x0061e030 | FUN_0061e030 | folded_vector_free_at_0x04 |  |
| 4487 | 0x0061e050 | FUN_0061e050 | uv_scroll_tick | C_tshift::Tick |
| 4488 | 0x0061e0d0 | FUN_0061e0d0 | crt_static_init_guard_0061e0d0 |  |
| 4489 | 0x0061e0f0 | FUN_0061e0f0 | crt_atexit_register_0061e0f0 |  |
| 4490 | 0x0061e100 | FUN_0061e100 | glass_shatter_merge_settled_shards | PrepareGlassSingleObject |
| 4491 | 0x0061e8f0 | FUN_0061e8f0 | glass_shatter_init | C_Break_plain_glass::Init |
| 4492 | 0x0061ea60 | FUN_0061ea60 | glass_shatter_done | C_Break_plain_glass::Done |
| 4493 | 0x0061ecd0 | FUN_0061ecd0 | glass_shatter_tick | C_Break_plain_glass::Tick |
| 4494 | 0x0061fb20 | FUN_0061fb20 | glass_shatter_pane_copy_assign |  |
| 4495 | 0x0061fc90 | FUN_0061fc90 | glass_shatter_break_frame | C_Break_plain_glass::BreakGlass |
| 4496 | 0x006202a0 | FUN_006202a0 | glass_shatter_cut_mesh | C_Break_plain_glass::CutMesh |
| 4497 | 0x00621130 | FUN_00621130 | glass_shatter_prepare_object | C_Break_plain_glass::PrepareObject |
| 4498 | 0x00621660 | FUN_00621660 | std_vector4_capacity |  |
| 4499 | 0x00621680 | FUN_00621680 | std_allocator_deallocate |  |
| 4500 | 0x00621690 | FUN_00621690 | std_vector20_insert_at |  |
| 4501 | 0x00621830 | FUN_00621830 | std_vector20_size |  |
| 4502 | 0x00621850 | FUN_00621850 | glass_shatter_triface_list_insert_at |  |
| 4503 | 0x00621a50 | FUN_00621a50 | glass_shatter_triface_list_size |  |
| 4504 | 0x00621a70 | FUN_00621a70 | glass_shatter_pane_list_insert_n |  |
| 4505 | 0x00621d70 | FUN_00621d70 | glass_shatter_triface_list_insert_n |  |
| 4506 | 0x00622070 | FUN_00622070 | stl_uninit_copy_stride_0x20 |  |
| 4507 | 0x006220b0 | FUN_006220b0 | stl_fill_n_stride_0x20 |  |
| 4508 | 0x006220e0 | FUN_006220e0 | glass_shatter_triface_list_uninit_copy |  |
| 4509 | 0x00622120 | FUN_00622120 | glass_shatter_triface_list_fill_n |  |
| 4510 | 0x00622160 | FUN_00622160 | glass_shatter_pane_construct_copy |  |
| 4511 | 0x00622210 | FUN_00622210 | glass_shatter_triface_list_construct_copy |  |
| 4512 | 0x00622230 | FUN_00622230 | crt_static_init_guard_00622230 |  |
| 4513 | 0x00622250 | FUN_00622250 | crt_atexit_register_00622250 |  |
| 4514 | 0x00622260 | FUN_00622260 | debris_init | C_Debris::Init |
| 4515 | 0x006222f0 | FUN_006222f0 | debris_done | C_Debris::Done |
| 4516 | 0x006223a0 | FUN_006223a0 | debris_tick | C_Debris::Tick |
| 4517 | 0x00622a30 | FUN_00622a30 | debris_generate | C_Debris::GenerateDebris |
| 4518 | 0x006230f0 | FUN_006230f0 | debris_generate_flash | C_Debris::GenerateFlash |
| 4519 | 0x00623700 | FUN_00623700 | debris_prepare_object | C_Debris::PrepareObject |
| 4520 | 0x00623ab0 | FUN_00623ab0 | debris_list_erase_at |  |
| 4521 | 0x00623b00 | FUN_00623b00 | std_vector44_insert_at |  |
| 4522 | 0x00623e10 | FUN_00623e10 | crt_static_init_guard_00623e10 |  |
| 4523 | 0x00623e30 | FUN_00623e30 | crt_atexit_register_00623e30 |  |
| 4524 | 0x00623e40 | FUN_00623e40 | winmain |  |
| 4525 | 0x00623ea0 | FUN_00623ea0 | crt_static_init_guard_00623ea0 |  |
| 4526 | 0x00623ec0 | FUN_00623ec0 | crt_atexit_register_00623ec0 |  |
| 4527 | 0x00623ed0 | FUN_00623ed0 | win_message_box |  |
| 4528 | 0x00623f80 | FUN_00623f80 | win_load_string_resource |  |
| 4529 | 0x00623fa0 | FUN_00623fa0 | win_set_app_icon |  |
| 4530 | 0x00623fc0 | FUN_00623fc0 | cd_find_game_file |  |
| 4531 | 0x00624070 | FUN_00624070 | cd_require_disc |  |
| 4532 | 0x006240b0 | FUN_006240b0 | cd_file_exists |  |
| 4533 | 0x006240e0 | FUN_006240e0 | cd_scan_drives_for_disc |  |
| 4534 | 0x00624190 | FUN_00624190 | std_string_reserve |  |
| 4535 | 0x006242b0 | FUN_006242b0 | crt_static_init_guard_006242b0 |  |
| 4536 | 0x006242d0 | FUN_006242d0 | crt_atexit_register_006242d0 |  |
| 4537 | 0x006242e0 | UpdateWMatrixProc | I3D_frame::UpdateWMatrixProc |  |
| 4538 | 0x006242e6 | RotateByMatrix | S_vector::RotateByMatrix |  |
| 4539 | 0x006242ec | Identity | S_matrix::Identity |  |
| 4540 | 0x006242f2 | operator* | S_vector::operator* |  |
| 4541 | 0x006242f8 | UpdateHRBoundVol | I3D_frame::UpdateHRBoundVol |  |
| 4542 | 0x006242fe | SetRot3 | S_matrix::SetRot3 |  |
| 4543 | 0x00624304 | Normalize | S_quat::Normalize |  |
| 4544 | 0x0062430a | AngleTo | S_vector::AngleTo |  |
| 4545 | 0x00624310 | operator*= | S_vector::operator*= |  |
| 4546 | 0x00624316 | RotationMatrix | S_quat::RotationMatrix |  |
| 4547 | 0x0062431c | Make | S_quat::Make |  |
| 4548 | 0x00624322 | Slerp | S_quat::Slerp |  |
| 4549 | 0x00624328 | GetScale | S_matrix::GetScale |  |
| 4550 | 0x0062432e | Inverse | S_matrix::Inverse |  |
| 4551 | 0x00624334 | Make | S_quat::Make |  |
| 4552 | 0x0062433a | SetDir | S_quat::SetDir |  |
| 4553 | 0x00624340 | SetDir3 | S_matrix::SetDir3 |  |
| 4554 | 0x00624346 | operator* | S_matrix::operator* |  |
| 4555 | 0x0062434c | SetRot | S_matrix::SetRot |  |
| 4556 | 0x00624352 | operator*= | S_matrix::operator*= |  |
| 4557 | 0x00624358 | IntersectionLine | S_plane::IntersectionLine |  |
| 4558 | 0x0062435e | RotateByMatrix | S_quat::RotateByMatrix |  |
| 4559 | 0x00624364 | UpdateHRBoundVolW | I3D_frame::UpdateHRBoundVolW |  |
| 4560 | 0x0062436a | dbgPrintf | dbgPrintf |  |
| 4561 | 0x00624370 | SetDir | S_matrix::SetDir |  |
| 4562 | 0x00624376 | GetIGraph | GetIGraph |  |
| 4563 | 0x0062437c | I3DGetDriver | I3DGetDriver |  |
| 4564 | 0x00624382 | RotateByNormMatrix | S_vector::RotateByNormMatrix |  |
| 4565 | 0x00624388 | GetUScale | S_matrix::GetUScale |  |
| 4566 | 0x0062438e | GetDir | S_quat::GetDir |  |
| 4567 | 0x00624394 | Inverse | S_quat::Inverse |  |
| 4568 | 0x0062439a | TEV | TEV |  |
| 4569 | 0x006243a0 | ISndGetDriver | ISndGetDriver |  |
| 4570 | 0x006243a6 | UpdateWBoundProc | I3D_visual::UpdateWBoundProc |  |
| 4571 | 0x006243ac | FUN_006243ac | operator_delete_wrapper | operator delete |
| 4572 | 0x006243b7 | FUN_006243b7 | std_string_xlen |  |
| 4573 | 0x00624411 | FUN_00624411 | std_logic_error_ctor_from_string |  |
| 4574 | 0x00624475 | FUN_00624475 | std_logic_error_what | std::logic_error::what |
| 4575 | 0x00624482 | FUN_00624482 | std_logic_error_throw_copy |  |
| 4576 | 0x0062449f | FUN_0062449f | std_logic_error_dtor |  |
| 4577 | 0x006244dc | FUN_006244dc | std_logic_error_scalar_deleting_dtor | std::logic_error::{dtor} |
| 4578 | 0x006244f8 | FUN_006244f8 | std_logic_error_copy_ctor |  |
| 4579 | 0x00624555 | FUN_00624555 | std_length_error_dtor |  |
| 4580 | 0x00624592 | FUN_00624592 | std_length_error_throw_copy |  |
| 4581 | 0x006245af | FUN_006245af | std_length_error_scalar_deleting_dtor | std::length_error::{dtor} |
| 4582 | 0x006245cb | FUN_006245cb | std_length_error_copy_ctor |  |
| 4583 | 0x006245e3 | FUN_006245e3 | std_string_xran |  |
| 4584 | 0x0062463d | FUN_0062463d | std_out_of_range_dtor |  |
| 4585 | 0x0062467a | FUN_0062467a | std_out_of_range_throw_copy |  |
| 4586 | 0x00624697 | FUN_00624697 | std_out_of_range_scalar_deleting_dtor |  |
| 4587 | 0x006246b3 | FUN_006246b3 | std_out_of_range_copy_ctor |  |
| 4588 | 0x006246cb | FUN_006246cb | std_string_assign_substring |  |
| 4589 | 0x00624799 | FUN_00624799 | std_string_assign_buffer | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>::assign |
| 4590 | 0x006247ce | FUN_006247ce | std_string_erase |  |
| 4591 | 0x00624835 | FUN_00624835 | std_string_freeze |  |
| 4592 | 0x00624865 | FUN_00624865 | crt_static_init_guard_00624865 |  |
| 4593 | 0x0062488f | FUN_0062488f | crt_atexit_register_0062488f |  |
| 4594 | 0x0062489b | FUN_0062489b | crt_atexit_register_0062489b |  |
| 4595 | 0x006248a8 | _CreateJoyBase@20 | _CreateJoyBase_20 |  |
| 4596 | 0x006248ae | _CreateINetBase@4 | _CreateINetBase_4 |  |
| 4597 | 0x006248b4 | dtaWrite | dtaWrite |  |
| 4598 | 0x006248ba | dtaRead | dtaRead |  |
| 4599 | 0x006248c0 | dtaSeek | dtaSeek |  |
| 4600 | 0x006248c6 | dtaClose | dtaClose |  |
| 4601 | 0x006248cc | dtaOpen | dtaOpen |  |
| 4602 | 0x006248d2 | dtaOpenWrite | dtaOpenWrite |  |
| 4603 | 0x006248d8 | dtaCreate | dtaCreate |  |
| 4604 | 0x006248de | dtaSetDtaFirstForce | dtaSetDtaFirstForce |  |
| 4605 | 0x006248e4 | FUN_006248e4 | crt_fp_init |  |
| 4606 | 0x006248fb | FUN_006248fb | stub_noop_cdecl_006248fb |  |
| 4607 | 0x006248fc | FUN_006248fc | crt_cfltcvt_init |  |
| 4608 | 0x00624934 | operator_new | operator_new |  |
| 4609 | 0x00624942 | FUN_00624942 | crt_onexit |  |
| 4610 | 0x006249af | FUN_006249af | crt_atexit |  |
| 4611 | 0x006249c1 | FUN_006249c1 | crt_onexit_init |  |
| 4612 | 0x006249f0 | __ftol | __ftol |  |
| 4613 | 0x00624a17 | FUN_00624a17 | crt_eh_jump_to_continuation |  |
| 4614 | 0x00624a4b | FUN_00624a4b | crt_eh_call_member_function_00624a4b |  |
| 4615 | 0x00624a52 | FUN_00624a52 | crt_eh_call_member_function_00624a52 |  |
| 4616 | 0x00624a59 | FUN_00624a59 | crt_eh_unwind_nested_frames |  |
| 4617 | 0x00624aa8 | FUN_00624aa8 | crt_cxx_frame_handler |  |
| 4618 | 0x00624ade | FUN_00624ade | crt_eh_call_catch_block |  |
| 4619 | 0x00624b32 | FUN_00624b32 | crt_eh_catch_guard_handler |  |
| 4620 | 0x00624b57 | FUN_00624b57 | crt_eh_call_se_translator |  |
| 4621 | 0x00624c0b | FUN_00624c0b | crt_eh_translator_guard_handler |  |
| 4622 | 0x00624c80 | FUN_00624c80 | crt_eh_get_try_range_to_check |  |
| 4623 | 0x00624cfc | __global_unwind2 | __global_unwind2 |  |
| 4624 | 0x00624d1c | FUN_00624d1c | crt_seh_unwind_collision_handler |  |
| 4625 | 0x00624d3e | __local_unwind2 | __local_unwind2 |  |
| 4626 | 0x00624da6 | __abnormal_termination | __abnormal_termination |  |
| 4627 | 0x00624dc9 | __NLG_Notify1 | __NLG_Notify1 |  |
| 4628 | 0x00624dd2 | FUN_00624dd2 | crt_nlg_notify |  |
| 4629 | 0x00624df0 | FUN_00624df0 | math_acos_x87_entry |  |
| 4630 | 0x00624e0d | FUN_00624e0d | math_acos_continuation |  |
| 4631 | 0x00624ebb | FUN_00624ebb | crt_atof |  |
| 4632 | 0x00624f08 | FUN_00624f08 | crt_atol |  |
| 4633 | 0x00624f93 | FUN_00624f93 | crt_atoi |  |
| 4634 | 0x00624fa0 | FUN_00624fa0 | crt_chkstk |  |
| 4635 | 0x00624fcf | FUN_00624fcf | crt_sprintf | sprintf |
| 4636 | 0x00625021 | FUN_00625021 | crt_vsnprintf |  |
| 4637 | 0x00625071 | FUN_00625071 | crt_ehvec_dtor |  |
| 4638 | 0x006250d9 | FUN_006250d9 | crt_ehvec_dtor_unwind_funclet |  |
| 4639 | 0x006250f1 | FUN_006250f1 | crt_eh_array_unwind |  |
| 4640 | 0x0062514f | FUN_0062514f | crt_ehvec_ctor |  |
| 4641 | 0x006251b9 | FUN_006251b9 | crt_ehvec_ctor_unwind_funclet |  |
| 4642 | 0x006251d1 | FUN_006251d1 | crt_fclose |  |
| 4643 | 0x00625227 | FUN_00625227 | crt_fwrite |  |
| 4644 | 0x00625331 | FUN_00625331 | crt_fsopen |  |
| 4645 | 0x00625351 | FUN_00625351 | crt_fopen |  |
| 4646 | 0x00625370 | _strrchr | _strrchr |  |
| 4647 | 0x00625397 | FUN_00625397 | crt_srand | srand |
| 4648 | 0x006253a1 | FUN_006253a1 | rand |  |
| 4649 | 0x006253bf | FUN_006253bf | crt_qsort |  |
| 4650 | 0x00625513 | FUN_00625513 | crt_qsort_shortsort |  |
| 4651 | 0x00625561 | FUN_00625561 | crt_qsort_swap |  |
| 4652 | 0x0062558d | FUN_0062558d | crt_strupr |  |
| 4653 | 0x0062562b | FUN_0062562b | crt_sscanf |  |
| 4654 | 0x00625660 | _strncmp | _strncmp |  |
| 4655 | 0x00625698 | FUN_00625698 | crt_isalpha |  |
| 4656 | 0x006256c6 | FUN_006256c6 | crt_isspace |  |
| 4657 | 0x00625700 | _strchr | _strchr |  |
| 4658 | 0x006257c0 | _strstr | _strstr |  |
| 4659 | 0x00625840 | FUN_00625840 | crt_memmove | memmove |
| 4660 | 0x00625b75 | FUN_00625b75 | crt_fprintf |  |
| 4661 | 0x00625bb0 | _strncpy | _strncpy |  |
| 4662 | 0x00625cae | FUN_00625cae | crt_ceil | ceil |
| 4663 | 0x00625d7d | FUN_00625d7d | crt_floor | floor |
| 4664 | 0x00625e4c | FUN_00625e4c | crt_fread |  |
| 4665 | 0x00625f34 | FUN_00625f34 | crt_remove |  |
| 4666 | 0x00625f60 | FUN_00625f60 | math_pow_x87_entry |  |
| 4667 | 0x00625f82 | FUN_00625f82 | math_pow_continuation |  |
| 4668 | 0x00626155 | FUN_00626155 | math_x87_integer_parity_pop |  |
| 4669 | 0x00626180 | _strncat | _strncat |  |
| 4670 | 0x006262a3 | FUN_006262a3 | crt_purecall |  |
| 4671 | 0x006262ac | FUN_006262ac | char_to_upper_locale | toupper |
| 4672 | 0x00626378 | FUN_00626378 | char_to_lower_locale |  |
| 4673 | 0x00626443 | FUN_00626443 | crt_realloc |  |
| 4674 | 0x006266e3 | _malloc | _malloc |  |
| 4675 | 0x006266f5 | __nh_malloc | __nh_malloc |  |
| 4676 | 0x00626721 | FUN_00626721 | crt_heap_alloc |  |
| 4677 | 0x00626795 | FUN_00626795 | crt_free |  |
| 4678 | 0x00626800 | __aulldiv | __aulldiv |  |
| 4679 | 0x00626870 | __allmul | __allmul |  |
| 4680 | 0x006268a4 | FUN_006268a4 | crt_vsprintf |  |
| 4681 | 0x00626900 | __allshl | __allshl |  |
| 4682 | 0x00626920 | FUN_00626920 | math_asin_x87_entry |  |
| 4683 | 0x0062693d | FUN_0062693d | math_asin_continuation |  |
| 4684 | 0x006269eb | FUN_006269eb | crt_time |  |
| 4685 | 0x00626ac7 | FUN_00626ac7 | crt_strtol | strtol |
| 4686 | 0x00626ade | FUN_00626ade | crt_strtoxl |  |
| 4687 | 0x00626ce6 | FUN_00626ce6 | crt_strtoul | strtoul |
| 4688 | 0x00626cfd | FUN_00626cfd | crt_call_init_fns |  |
| 4689 | 0x00626d2a | FUN_00626d2a | crt_exit |  |
| 4690 | 0x00626d3b | __exit | __exit |  |
| 4691 | 0x00626d4c | FUN_00626d4c | crt_doexit |  |
| 4692 | 0x00626de5 | FUN_00626de5 | crt_initterm |  |
| 4693 | 0x00626dff | FUN_00626dff | crt_getcwd |  |
| 4694 | 0x00626e12 | FUN_00626e12 | crt_getdcwd |  |
| 4695 | 0x00626ee3 | FUN_00626ee3 | crt_drive_is_valid |  |
| 4696 | 0x00626f1a | entry | crt_startup |  |
| 4697 | 0x00627010 | __amsg_exit | __amsg_exit |  |
| 4698 | 0x00627035 | FUN_00627035 | crt_fast_error_exit |  |
| 4699 | 0x00627059 | FUN_00627059 | exception_scalar_deleting_dtor | std::exception::{dtor} |
| 4700 | 0x00627075 | FUN_00627075 | exception_ctor_from_cstr |  |
| 4701 | 0x006270b2 | FUN_006270b2 | exception_copy_ctor |  |
| 4702 | 0x006270fc | ~exception | exception::~exception |  |
| 4703 | 0x00627112 | FUN_00627112 | exception_what | std::exception::what |
| 4704 | 0x0062711f | FUN_0062711f | type_info_dtor |  |
| 4705 | 0x00627134 | FUN_00627134 | type_info_scalar_deleting_dtor |  |
| 4706 | 0x00627150 | _strlen | _strlen |  |
| 4707 | 0x006271cc | FUN_006271cc | crt_seh_prolog |  |
| 4708 | 0x006271eb | __CxxThrowException@8 | __CxxThrowException_8 |  |
| 4709 | 0x00627230 | FUN_00627230 | crt_memcpy | memcpy |
| 4710 | 0x00627565 | FUN_00627565 | crt_set_default_fp_precision |  |
| 4711 | 0x00627577 | FUN_00627577 | crt_fdiv_erratum_selftest |  |
| 4712 | 0x006275b5 | FUN_006275b5 | crt_detect_fdiv_erratum |  |
| 4713 | 0x006275de | FUN_006275de | crt_forcdecpt |  |
| 4714 | 0x00627638 | FUN_00627638 | crt_cropzeros |  |
| 4715 | 0x00627686 | FUN_00627686 | crt_positive |  |
| 4716 | 0x0062769e | __fassign | __fassign |  |
| 4717 | 0x006276dc | FUN_006276dc | crt_cftoe |  |
| 4718 | 0x006277e0 | FUN_006277e0 | crt_cftof |  |
| 4719 | 0x006278be | FUN_006278be | crt_cftog |  |
| 4720 | 0x00627959 | FUN_00627959 | crt_cftoe_preconverted |  |
| 4721 | 0x00627980 | FUN_00627980 | crt_cftof_preconverted |  |
| 4722 | 0x006279a3 | __cfltcvt | __cfltcvt |  |
| 4723 | 0x006279f4 | FUN_006279f4 | crt_cvt_shift_string |  |
| 4724 | 0x00627a19 | FUN_00627a19 | crt_msize |  |
| 4725 | 0x00627a7a | FUN_00627a7a | crt_eh_handle_frame |  |
| 4726 | 0x00627b15 | FUN_00627b15 | crt_eh_find_handler |  |
| 4727 | 0x00627cbc | FUN_00627cbc | crt_eh_find_handler_for_foreign_exception |  |
| 4728 | 0x00627d64 | FUN_00627d64 | crt_eh_catch_type_matches |  |
| 4729 | 0x00627dc1 | FUN_00627dc1 | crt_eh_unwind_frame_to_state |  |
| 4730 | 0x00627e5f | FUN_00627e5f | crt_eh_unhandled_exception_filter |  |
| 4731 | 0x00627e70 | thunk_FUN_006282cc | thunk_crt_terminate |  |
| 4732 | 0x00627e75 | FUN_00627e75 | crt_eh_enter_catch_handler |  |
| 4733 | 0x00627ef0 | FUN_00627ef0 | crt_eh_run_catch_block |  |
| 4734 | 0x00627fb6 | FUN_00627fb6 | crt_eh_run_catch_block_finally |  |
| 4735 | 0x00627ffc | FUN_00627ffc | crt_eh_rethrow_exception_filter |  |
| 4736 | 0x00628026 | FUN_00628026 | crt_eh_build_catch_object |  |
| 4737 | 0x006281e2 | FUN_006281e2 | crt_eh_terminate_funclet_006281e2 |  |
| 4738 | 0x006281ea | FUN_006281ea | crt_eh_destroy_exception_object |  |
| 4739 | 0x00628249 | FUN_00628249 | crt_eh_terminate_funclet_00628249 |  |
| 4740 | 0x00628251 | FUN_00628251 | crt_eh_adjust_pointer |  |
| 4741 | 0x00628280 | __CallSettingFrame@12 | __CallSettingFrame_12 |  |
| 4742 | 0x006282cc | FUN_006282cc | crt_terminate |  |
| 4743 | 0x00628322 | FUN_00628322 | crt_unexpected |  |
| 4744 | 0x00628380 | FUN_00628380 | math_x87_dispatch_by_operand_class |  |
| 4745 | 0x006283e7 | FUN_006283e7 | math_x87_dispatch_by_operand_class_pair |  |
| 4746 | 0x00628478 | FUN_00628478 | math_x87_return_first_operand |  |
| 4747 | 0x00628486 | FUN_00628486 | math_x87_return_zero |  |
| 4748 | 0x0062848d | FUN_0062848d | math_x87_return_one |  |
| 4749 | 0x00628492 | FUN_00628492 | math_x87_return_nan_operand |  |
| 4750 | 0x00628539 | FUN_00628539 | math_x87_mark_operand_snan |  |
| 4751 | 0x00628550 | FUN_00628550 | __startTwoArgErrorHandling |  |
| 4752 | 0x00628567 | __startOneArgErrorHandling | __startOneArgErrorHandling |  |
| 4753 | 0x006285b0 | FUN_006285b0 | math_x87_exp2 |  |
| 4754 | 0x006285c5 | FUN_006285c5 | math_x87_mask_exceptions |  |
| 4755 | 0x006285dc | FUN_006285dc | math_x87_classify_nan |  |
| 4756 | 0x006285f5 | __fload_withFB | __fload_withFB |  |
| 4757 | 0x00628638 | FUN_00628638 | math_x87_exponent_or_nan_bits |  |
| 4758 | 0x0062864e | FUN_0062864e | math_x87_restore_control_word |  |
| 4759 | 0x0062865b | __math_exit | __math_exit |  |
| 4760 | 0x0062868b | FUN_0062868b | math_x87_check_result_range_prespilled |  |
| 4761 | 0x00628699 | FUN_00628699 | math_x87_check_result_range |  |
| 4762 | 0x0062873c | FUN_0062873c | crt_string_to_double |  |
| 4763 | 0x006287cd | FUN_006287cd | crt_isctype |  |
| 4764 | 0x00628842 | FUN_00628842 | crt_flsbuf |  |
| 4765 | 0x00628957 | FUN_00628957 | crt_output |  |
| 4766 | 0x00629098 | FUN_00629098 | crt_output_write_char |  |
| 4767 | 0x006290cd | FUN_006290cd | crt_output_write_multi_char |  |
| 4768 | 0x006290fe | FUN_006290fe | crt_output_write_string |  |
| 4769 | 0x00629136 | FUN_00629136 | crt_output_get_int_arg |  |
| 4770 | 0x00629143 | FUN_00629143 | crt_output_get_int64_arg |  |
| 4771 | 0x00629153 | FUN_00629153 | crt_output_get_short_arg |  |
| 4772 | 0x00629229 | __seh_longjmp_unwind@4 | __seh_longjmp_unwind_4 |  |
| 4773 | 0x00629244 | FUN_00629244 | crt_close |  |
| 4774 | 0x006292f7 | __freebuf | __freebuf |  |
| 4775 | 0x00629322 | FUN_00629322 | crt_fflush |  |
| 4776 | 0x0062935d | FUN_0062935d | crt_flush_stream |  |
| 4777 | 0x006293b9 | FUN_006293b9 | crt_flushall |  |
| 4778 | 0x006293c2 | flsall | flsall |  |
| 4779 | 0x0062942f | FUN_0062942f | crt_write |  |
| 4780 | 0x006295dc | FUN_006295dc | crt_openfile |  |
| 4781 | 0x0062974c | FUN_0062974c | crt_getstream |  |
| 4782 | 0x006297d0 | FUN_006297d0 | crt_strcpy |  |
| 4783 | 0x006297e0 | FUN_006297e0 | crt_strcat |  |
| 4784 | 0x006298c0 | FUN_006298c0 | crt_lcmapstring_ansi |  |
| 4785 | 0x00629a84 | FUN_00629a84 | crt_lcmapstring_ansi_except_filter |  |
| 4786 | 0x00629ae4 | FUN_00629ae4 | crt_input |  |
| 4787 | 0x0062a509 | FUN_0062a509 | crt_input_hex_digit_value |  |
| 4788 | 0x0062a540 | FUN_0062a540 | crt_input_getc |  |
| 4789 | 0x0062a55a | FUN_0062a55a | crt_input_ungetc |  |
| 4790 | 0x0062a571 | FUN_0062a571 | crt_input_skip_whitespace |  |
| 4791 | 0x0062a595 | FUN_0062a595 | crt_setup_temp_buffer |  |
| 4792 | 0x0062a622 | FUN_0062a622 | crt_flush_temp_buffer |  |
| 4793 | 0x0062a65f | FUN_0062a65f | math_raise_domain_error |  |
| 4794 | 0x0062a6b2 | FUN_0062a6b2 | math_raise_fp_exception |  |
| 4795 | 0x0062a74a | FUN_0062a74a | math_raise_ieee_exception |  |
| 4796 | 0x0062a9fd | FUN_0062a9fd | math_apply_masked_exception_result |  |
| 4797 | 0x0062ac14 | FUN_0062ac14 | math_invoke_matherr |  |
| 4798 | 0x0062ac9c | FUN_0062ac9c | math_set_errno_for_exception_type |  |
| 4799 | 0x0062acc2 | FUN_0062acc2 | math_function_name_for_id |  |
| 4800 | 0x0062ace7 | FUN_0062ace7 | math_exception_type_from_flags |  |
| 4801 | 0x0062ad14 | __frnd | __frnd |  |
| 4802 | 0x0062ad26 | FUN_0062ad26 | math_double_set_exponent |  |
| 4803 | 0x0062ad4f | FUN_0062ad4f | math_classify_double_special |  |
| 4804 | 0x0062ada9 | FUN_0062ada9 | math_frexp |  |
| 4805 | 0x0062ae6a | FUN_0062ae6a | math_x87_read_status_word |  |
| 4806 | 0x0062ae78 | FUN_0062ae78 | math_x87_read_and_clear_status_word |  |
| 4807 | 0x0062ae87 | FUN_0062ae87 | math_x87_merge_control_word |  |
| 4808 | 0x0062aeaa | FUN_0062aeaa | math_x87_raise_status_flags |  |
| 4809 | 0x0062af00 | FUN_0062af00 | crt_filbuf |  |
| 4810 | 0x0062afd9 | FUN_0062afd9 | crt_read |  |
| 4811 | 0x0062b1cf | FUN_0062b1cf | crt_dosmaperr |  |
| 4812 | 0x0062b251 | FUN_0062b251 | math_exp_finite_argument_arm |  |
| 4813 | 0x0062b344 | thunk_FUN_00628486 | thunk_math_x87_return_zero |  |
| 4814 | 0x0062b391 | FUN_0062b391 | math_exp_infinite_argument_arm |  |
| 4815 | 0x0062b39e | FUN_0062b39e | math_exp_fraction_core |  |
| 4816 | 0x0062b3e1 | FUN_0062b3e1 | math_x87_integer_parity |  |
| 4817 | 0x0062b415 | FUN_0062b415 | math_pow_special_case_x87_wrapper |  |
| 4818 | 0x0062b447 | FUN_0062b447 | math_pow_special_case_result |  |
| 4819 | 0x0062b577 | FUN_0062b577 | math_double_integer_parity |  |
| 4820 | 0x0062b5dc | FUN_0062b5dc | crt_sbh_heap_init |  |
| 4821 | 0x0062b624 | FUN_0062b624 | crt_sbh_find_block |  |
| 4822 | 0x0062b64f | FUN_0062b64f | crt_sbh_free_block |  |
| 4823 | 0x0062b978 | FUN_0062b978 | crt_sbh_alloc_block |  |
| 4824 | 0x0062bc81 | FUN_0062bc81 | crt_sbh_alloc_new_region |  |
| 4825 | 0x0062bd32 | FUN_0062bd32 | crt_sbh_alloc_new_group |  |
| 4826 | 0x0062be2d | FUN_0062be2d | crt_sbh_resize_block |  |
| 4827 | 0x0062c123 | FUN_0062c123 | crt_old_sbh_new_region |  |
| 4828 | 0x0062c267 | FUN_0062c267 | crt_old_sbh_release_region |  |
| 4829 | 0x0062c2bd | FUN_0062c2bd | crt_old_sbh_decommit_pages |  |
| 4830 | 0x0062c37f | FUN_0062c37f | crt_old_sbh_find_block |  |
| 4831 | 0x0062c3d6 | FUN_0062c3d6 | crt_old_sbh_free_block |  |
| 4832 | 0x0062c41b | FUN_0062c41b | crt_old_sbh_alloc_block |  |
| 4833 | 0x0062c623 | FUN_0062c623 | crt_old_sbh_alloc_from_page |  |
| 4834 | 0x0062c747 | FUN_0062c747 | crt_old_sbh_resize_block |  |
| 4835 | 0x0062c7f0 | FUN_0062c7f0 | crt_call_new_handler |  |
| 4836 | 0x0062c80b | FUN_0062c80b | crt_get_image_linker_version |  |
| 4837 | 0x0062c838 | FUN_0062c838 | crt_heap_select |  |
| 4838 | 0x0062c980 | FUN_0062c980 | crt_heap_init |  |
| 4839 | 0x0062c9dd | FUN_0062c9dd | crt_local_time_to_time_t |  |
| 4840 | 0x0062ca9f | FUN_0062ca9f | crt_exception_filter |  |
| 4841 | 0x0062cbe0 | FUN_0062cbe0 | crt_xcpt_action_lookup |  |
| 4842 | 0x0062cc23 | FUN_0062cc23 | crt_get_cmdline_args |  |
| 4843 | 0x0062cc7b | FUN_0062cc7b | crt_build_environ |  |
| 4844 | 0x0062cd34 | FUN_0062cd34 | crt_setup_argv |  |
| 4845 | 0x0062cdcd | FUN_0062cdcd | crt_parse_cmdline |  |
| 4846 | 0x0062cf81 | FUN_0062cf81 | crt_get_environment_strings_ansi |  |
| 4847 | 0x0062d0b3 | FUN_0062d0b3 | crt_init_file_handles |  |
| 4848 | 0x0062d25e | FUN_0062d25e | crt_write_runtime_error_banner |  |
| 4849 | 0x0062d297 | FUN_0062d297 | crt_write_runtime_error_message |  |
| 4850 | 0x0062d3f0 | _strcmp | _strcmp |  |
| 4851 | 0x0062d474 | FUN_0062d474 | crt_cxx_unhandled_exception_filter |  |
| 4852 | 0x0062d4ba | FUN_0062d4ba | crt_cxx_install_unhandled_exception_filter |  |
| 4853 | 0x0062d4cb | FUN_0062d4cb | crt_cxx_restore_unhandled_exception_filter |  |
| 4854 | 0x0062d4d8 | FUN_0062d4d8 | crt_control87 |  |
| 4855 | 0x0062d50d | FUN_0062d50d | crt_controlfp |  |
| 4856 | 0x0062d523 | FUN_0062d523 | crt_fp_control_word_decode |  |
| 4857 | 0x0062d5b5 | FUN_0062d5b5 | crt_fp_control_word_encode |  |
| 4858 | 0x0062d63e | FUN_0062d63e | crt_ld12_bits_below_are_zero |  |
| 4859 | 0x0062d687 | FUN_0062d687 | crt_ld12_add_one_at_bit |  |
| 4860 | 0x0062d6dd | FUN_0062d6dd | crt_ld12_round_at_bit |  |
| 4861 | 0x0062d769 | FUN_0062d769 | crt_ld12_mantissa_copy |  |
| 4862 | 0x0062d784 | FUN_0062d784 | crt_ld12_mantissa_clear |  |
| 4863 | 0x0062d790 | FUN_0062d790 | crt_ld12_mantissa_is_zero |  |
| 4864 | 0x0062d7ab | FUN_0062d7ab | crt_ld12_mantissa_shift_right |  |
| 4865 | 0x0062d838 | FUN_0062d838 | crt_ld12_to_ieee |  |
| 4866 | 0x0062d9a4 | FUN_0062d9a4 | crt_ld12_to_double |  |
| 4867 | 0x0062d9ba | FUN_0062d9ba | crt_ld12_to_float |  |
| 4868 | 0x0062d9d0 | FUN_0062d9d0 | crt_atodbl |  |
| 4869 | 0x0062d9fd | FUN_0062d9fd | crt_atoflt |  |
| 4870 | 0x0062da2a | FUN_0062da2a | crt_fptostr |  |
| 4871 | 0x0062daa1 | FUN_0062daa1 | crt_fltout |  |
| 4872 | 0x0062db05 | FUN_0062db05 | crt_double_to_ld10 |  |
| 4873 | 0x0062dbc0 | _memset | _memset |  |
| 4874 | 0x0062dc18 | FUN_0062dc18 | crt_fp_not_loaded_trap |  |
| 4875 | 0x0062dc21 | FUN_0062dc21 | crt_is_readable_pointer |  |
| 4876 | 0x0062dc3d | FUN_0062dc3d | crt_is_writable_pointer |  |
| 4877 | 0x0062dc59 | FUN_0062dc59 | crt_is_executable_pointer |  |
| 4878 | 0x0062dc71 | FUN_0062dc71 | crt_abort |  |
| 4879 | 0x0062dc88 | FUN_0062dc88 | math_raise_fp_exception_for_type |  |
| 4880 | 0x0062dd58 | FUN_0062dd58 | crt_string_to_ld12 |  |
| 4881 | 0x0062e229 | FUN_0062e229 | crt_getstringtype_ansi |  |
| 4882 | 0x0062e372 | FUN_0062e372 | crt_lseek |  |
| 4883 | 0x0062e40c | FUN_0062e40c | crt_getbuf |  |
| 4884 | 0x0062e450 | FUN_0062e450 | crt_isatty |  |
| 4885 | 0x0062e476 | FUN_0062e476 | crt_initstdio |  |
| 4886 | 0x0062e51b | FUN_0062e51b | crt_endstdio |  |
| 4887 | 0x0062e52f | FUN_0062e52f | crt_wctomb |  |
| 4888 | 0x0062e5a0 | __aullrem | __aullrem |  |
| 4889 | 0x0062e615 | FUN_0062e615 | crt_alloc_osfhandle_slot |  |
| 4890 | 0x0062e6aa | FUN_0062e6aa | crt_set_osfhandle |  |
| 4891 | 0x0062e721 | FUN_0062e721 | crt_free_osfhandle_slot |  |
| 4892 | 0x0062e79b | FUN_0062e79b | crt_get_osfhandle |  |
| 4893 | 0x0062e7d8 | FUN_0062e7d8 | crt_commit |  |
| 4894 | 0x0062e82f | FUN_0062e82f | crt_sopen |  |
| 4895 | 0x0062eae8 | FUN_0062eae8 | crt_mbtowc |  |
| 4896 | 0x0062ebb0 | FUN_0062ebb0 | crt_ungetc |  |
| 4897 | 0x0062ec1e | FUN_0062ec1e | crt_matherr_default |  |
| 4898 | 0x0062ec30 | FUN_0062ec30 | math_x87_fdiv_erratum_divide_core |  |
| 4899 | 0x0062ed47 | FUN_0062ed47 | math_x87_fdiv_erratum_register_divide_dispatch |  |
| 4900 | 0x0062f1e6 | FUN_0062f1e6 | math_x87_fdiv_erratum_divide |  |
| 4901 | 0x0062f1f9 | FUN_0062f1f9 | math_x87_fdiv_erratum_reverse_divide |  |
| 4902 | 0x0062f20c | FUN_0062f20c | math_x87_fdiv_erratum_divide_by_float |  |
| 4903 | 0x0062f258 | FUN_0062f258 | math_x87_fdiv_erratum_divide_by_double |  |
| 4904 | 0x0062f2a4 | FUN_0062f2a4 | math_x87_fdiv_erratum_divide_by_int16 |  |
| 4905 | 0x0062f2d8 | FUN_0062f2d8 | math_x87_fdiv_erratum_divide_by_int32 |  |
| 4906 | 0x0062f30c | FUN_0062f30c | math_x87_fdiv_erratum_reverse_divide_by_float |  |
| 4907 | 0x0062f358 | FUN_0062f358 | math_x87_fdiv_erratum_reverse_divide_by_double |  |
| 4908 | 0x0062f3a4 | FUN_0062f3a4 | math_x87_fdiv_erratum_reverse_divide_by_int16 |  |
| 4909 | 0x0062f3d8 | FUN_0062f3d8 | math_x87_fdiv_erratum_reverse_divide_by_int32 |  |
| 4910 | 0x0062f40c | FUN_0062f40c | math_x87_fdiv_erratum_fdivp |  |
| 4911 | 0x0062f421 | FUN_0062f421 | math_x87_fdiv_erratum_fdivrp |  |
| 4912 | 0x0062f436 | FUN_0062f436 | math_x87_fdiv_erratum_fprem_core |  |
| 4913 | 0x0062f63c | FUN_0062f63c | math_x87_fdiv_erratum_fprem_scale_denormal |  |
| 4914 | 0x0062f6ee | FUN_0062f6ee | math_x87_fdiv_erratum_fprem1_core |  |
| 4915 | 0x0062f8f4 | FUN_0062f8f4 | math_x87_fdiv_erratum_fprem1_scale_denormal |  |
| 4916 | 0x0062f9a9 | FUN_0062f9a9 | math_x87_fdiv_erratum_fprem |  |
| 4917 | 0x0062f9af | FUN_0062f9af | math_x87_fdiv_erratum_fprem1 |  |
| 4918 | 0x0062f9b5 | FUN_0062f9b5 | math_x87_fdiv_erratum_fpatan |  |
| 4919 | 0x0062f9b8 | FUN_0062f9b8 | math_x87_fdiv_erratum_fptan |  |
| 4920 | 0x0062f9bb | FUN_0062f9bb | math_fpclass_double |  |
| 4921 | 0x0062fa4d | FUN_0062fa4d | crt_tzset_once |  |
| 4922 | 0x0062fa62 | FUN_0062fa62 | crt_tzset |  |
| 4923 | 0x0062fcc0 | FUN_0062fcc0 | crt_is_in_dst |  |
| 4924 | 0x0062fe6c | FUN_0062fe6c | crt_dst_rule_to_yearday |  |
| 4925 | 0x0062ffac | FUN_0062ffac | crt_ismbblead |  |
| 4926 | 0x0062ffbd | FUN_0062ffbd | crt_ismbbtype |  |
| 4927 | 0x0062ffee | FUN_0062ffee | crt_setmbcp |  |
| 4928 | 0x00630187 | FUN_00630187 | crt_resolve_mbcs_codepage |  |
| 4929 | 0x006301d1 | FUN_006301d1 | crt_lcid_for_mbcs_codepage |  |
| 4930 | 0x00630204 | FUN_00630204 | crt_mbcs_set_single_byte |  |
| 4931 | 0x0063022d | FUN_0063022d | crt_mbcs_build_case_tables |  |
| 4932 | 0x006303b2 | FUN_006303b2 | crt_mbcs_init_once |  |
| 4933 | 0x006303ce | FUN_006303ce | crt_message_box |  |
| 4934 | 0x00630457 | FUN_00630457 | crt_ld12_add_dword_carry |  |
| 4935 | 0x00630478 | ___add_12 | ___add_12 |  |
| 4936 | 0x006304d6 | FUN_006304d6 | crt_ld12_shift_left_1 |  |
| 4937 | 0x00630504 | FUN_00630504 | crt_ld12_shift_right_1 |  |
| 4938 | 0x00630531 | FUN_00630531 | crt_ld12_from_decimal_digits |  |
| 4939 | 0x006305f8 | FUN_006305f8 | crt_ld10_to_decimal_digits |  |
| 4940 | 0x0063088b | FUN_0063088b | crt_raise |  |
| 4941 | 0x006309b8 | FUN_006309b8 | crt_xcpt_action_lookup_by_signal |  |
| 4942 | 0x006309fd | FUN_006309fd | crt_ld12_multiply |  |
| 4943 | 0x00630c1d | FUN_00630c1d | crt_ld12_multiply_by_power_of_ten |  |
| 4944 | 0x00630c99 | FUN_00630c99 | crt_calloc |  |
| 4945 | 0x00630d4a | FUN_00630d4a | crt_fcloseall |  |
| 4946 | 0x00630da2 | FUN_00630da2 | crt_chsize |  |
| 4947 | 0x00630ee8 | FUN_00630ee8 | crt_getenv |  |
| 4948 | 0x00630f65 | FUN_00630f65 | crt_setmode |  |
| 4949 | 0x00630fdb | __mbsnbicoll | __mbsnbicoll |  |
| 4950 | 0x0063101a | FUN_0063101a | crt_environ_from_wide |  |
| 4951 | 0x00631088 | FUN_00631088 | crt_comparestring_ansi |  |
| 4952 | 0x006312ae | FUN_006312ae | crt_comparestring_ansi_except_filter |  |
| 4953 | 0x00631305 | FUN_00631305 | crt_strnlen |  |
| 4954 | 0x00631330 | FUN_00631330 | crt_setenv |  |
| 4955 | 0x006314b7 | FUN_006314b7 | crt_environ_find_index |  |
| 4956 | 0x0063150f | FUN_0063150f | crt_environ_duplicate |  |
| 4957 | 0x00631576 | FUN_00631576 | crt_mbschr |  |
| 4958 | 0x006315e9 | FUN_006315e9 | crt_strdup |  |
| 4959 | 0x00631620 | FUN_00631620 | crashreport_map_stream_static_init |  |
| 4960 | 0x00631630 | FUN_00631630 | crashreport_map_stream_static_ctor |  |
| 4961 | 0x00631640 | FUN_00631640 | crashreport_map_stream_register_atexit_dtor |  |
| 4962 | 0x00631650 | FUN_00631650 | crashreport_map_stream_static_dtor |  |
| 4963 | 0x00631670 | FUN_00631670 | crashreport_symbol_compare_by_address |  |
| 4964 | 0x00631690 | FUN_00631690 | crashreport_find_file_in_tree |  |
| 4965 | 0x00631870 | FUN_00631870 | crashreport_find_module_for_address |  |
| 4966 | 0x006318d0 | FUN_006318d0 | crashreport_build_report |  |
| 4967 | 0x00631fa0 | FUN_00631fa0 | crt_operator_delete_thunk_00631fa0 |  |
| 4968 | 0x00631fc0 | FUN_00631fc0 | crashreport_module_load_map_symbols |  |
| 4969 | 0x00632670 | FUN_00632670 | crashreport_symbol_dtor |  |
| 4970 | 0x006326a0 | FUN_006326a0 | crashreport_module_list_build |  |
| 4971 | 0x006328d0 | FUN_006328d0 | crashreport_module_dtor |  |
| 4972 | 0x00632980 | FUN_00632980 | crashreport_unhandled_exception_filter |  |
| 4973 | 0x00632a30 | FUN_00632a30 | crashreport_install_handler |  |
| 4974 | 0x00632a60 | FUN_00632a60 | crashreport_module_vector_destroy |  |
| 4975 | 0x00632aa0 | FUN_00632aa0 | crashreport_symbol_range_destroy |  |
| 4976 | 0x00632ad0 | FUN_00632ad0 | crashreport_symbol_vector_insert_n |  |
| 4977 | 0x00632e10 | FUN_00632e10 | crashreport_module_vector_insert_n |  |
| 4978 | 0x00633180 | FUN_00633180 | crashreport_symbol_construct_at |  |
| 4979 | 0x006331f0 | FUN_006331f0 | crashreport_module_construct_at |  |
| 4980 | 0x00633350 | FUN_00633350 | crashreport_symbol_assign |  |
| 4981 | 0x006333b0 | FUN_006333b0 | crashreport_module_assign |  |
| 4982 | 0x00633560 | FUN_00633560 | crashreport_module_string_vector_assign |  |
| 4983 | 0x00633770 | FUN_00633770 | crashreport_symbol_vector_assign |  |
| 4984 | 0x00633a10 | FUN_00633a10 | crashreport_symbol_scalar_deleting_dtor |  |
| 4985 | 0x00633a40 | FUN_00633a40 | string_range_copy_assign |  |
| 4986 | 0x00633a90 | FUN_00633a90 | string_ref_construct_at_cdecl |  |
| 4987 | 0x00633ab0 | FUN_00633ab0 | ifstream_ctor |  |
| 4988 | 0x00633b36 | FUN_00633b36 | ifstream_scalar_deleting_dtor |  |
| 4989 | 0x00633b61 | FUN_00633b61 | ifstream_dtor |  |
| 4990 | 0x00633b74 | FUN_00633b74 | ifstream_open |  |
| 4991 | 0x00633bb0 | FUN_00633bb0 | ifstream_close |  |
| 4992 | 0x00633bdf | FUN_00633bdf | ios_ctor |  |
| 4993 | 0x00633c11 | FUN_00633c11 | ios_scalar_deleting_dtor |  |
| 4994 | 0x00633c2d | FUN_00633c2d | ios_dtor |  |
| 4995 | 0x00633c56 | FUN_00633c56 | ios_set_streambuf |  |
| 4996 | 0x00633c85 | FUN_00633c85 | istream_get_line |  |
| 4997 | 0x00633d3f | FUN_00633d3f | filebuf_ctor |  |
| 4998 | 0x00633d59 | FUN_00633d59 | filebuf_scalar_deleting_dtor |  |
| 4999 | 0x00633d75 | FUN_00633d75 | filebuf_dtor |  |
| 5000 | 0x00633dba | FUN_00633dba | filebuf_close |  |
| 5001 | 0x00633deb | FUN_00633deb | filebuf_overflow |  |
| 5002 | 0x00633e5c | FUN_00633e5c | streambuf_sputc |  |
| 5003 | 0x00633e7e | FUN_00633e7e | filebuf_underflow |  |
| 5004 | 0x00633f0e | FUN_00633f0e | streambuf_in_avail |  |
| 5005 | 0x00633f1e | FUN_00633f1e | filebuf_seekoff |  |
| 5006 | 0x00633f62 | FUN_00633f62 | filebuf_sync |  |
| 5007 | 0x0063403a | FUN_0063403a | filebuf_setbuf |  |
| 5008 | 0x00634079 | FUN_00634079 | istream_scalar_deleting_dtor |  |
| 5009 | 0x006340a4 | FUN_006340a4 | istream_ctor |  |
| 5010 | 0x0063411a | FUN_0063411a | istream_dtor |  |
| 5011 | 0x00634129 | FUN_00634129 | istream_ipfx |  |
| 5012 | 0x006341ab | FUN_006341ab | streambuf_sputbackc |  |
| 5013 | 0x006341d0 | FUN_006341d0 | istream_eatwhite |  |
| 5014 | 0x00634211 | FUN_00634211 | filebuf_open |  |
| 5015 | 0x0063435a | FUN_0063435a | streambuf_snextc |  |
| 5016 | 0x006343a6 | FUN_006343a6 | streambuf_stossc |  |
| 5017 | 0x006343df | FUN_006343df | streambuf_sgetc |  |
| 5018 | 0x00634404 | FUN_00634404 | streambuf_ctor |  |
| 5019 | 0x00634431 | FUN_00634431 | streambuf_scalar_deleting_dtor |  |
| 5020 | 0x0063444d | FUN_0063444d | streambuf_dtor |  |
| 5021 | 0x00634471 | FUN_00634471 | streambuf_setbuf |  |
| 5022 | 0x006344a5 | FUN_006344a5 | streambuf_xsputn |  |
| 5023 | 0x006344fb | FUN_006344fb | streambuf_xsgetn |  |
| 5024 | 0x00634592 | FUN_00634592 | streambuf_sync |  |
| 5025 | 0x006345a9 | FUN_006345a9 | streambuf_allocate |  |
| 5026 | 0x006345c7 | FUN_006345c7 | streambuf_doallocate |  |
| 5027 | 0x006345f4 | FUN_006345f4 | streambuf_setb |  |
| 5028 | 0x00634624 | FUN_00634624 | streambuf_seekoff |  |
| 5029 | 0x0063462a | FUN_0063462a | streambuf_seekpos |  |
| 5030 | 0x0063463c | FUN_0063463c | streambuf_pbackfail |  |
| 5031 | 0x00634698 | FUN_00634698 | ostream_flush |  |
| 5032 | 0x006346c0 | RtlUnwind | RtlUnwind |  |
| 5033 | 0x006346c6 | Module32Next | Module32Next |  |
| 5034 | 0x006346cc | Module32First | Module32First |  |
| 5035 | 0x006346d2 | CreateToolhelp32Snapshot | CreateToolhelp32Snapshot |  |
| 5036 | 0x006346e0 | __strcmpi | crt_stricmp | strcasecmp |
| 5037 | 0x00634770 | FUN_00634770 | crt_strnicmp | strncasecmp |
| 5038 | 0x0063481d | FUN_0063481d | crt_itoa |  |
| 5039 | 0x0063484a | FUN_0063484a | crt_xtoa |  |
| 5040 | 0x006348a6 | FUN_006348a6 | crt_spawnlp |  |
| 5041 | 0x006348be | FUN_006348be | crt_chdir |  |
| 5042 | 0x00634944 | FUN_00634944 | crt_spawnvpe |  |
| 5043 | 0x00634a8d | FUN_00634a8d | crt_spawnve |  |
| 5044 | 0x00634ade | FUN_00634ade | crt_mbctoupper |  |
| 5045 | 0x00634b59 | FUN_00634b59 | crt_access |  |
| 5046 | 0x00634b9d | FUN_00634b9d | crt_mbsrchr |  |
| 5047 | 0x00634bfd | FUN_00634bfd | crt_dospawn |  |
| 5048 | 0x00634ddc | FUN_00634ddc | crt_cenvarg |  |
| 5049 | 0x006ff05e | FUN_006ff05e | safedisc_stub_entry |  |
| 5050 | 0x006ff119 | thunk_FUN_00626f1a | safedisc_jmp_oep |  |
| 5051 | 0x006ff120 | FUN_006ff120 | safedisc_run |  |
| 5052 | 0x006ff430 | FUN_006ff430 | safedisc_resolve_kernel32_imports |  |
| 5053 | 0x006ff8c0 | FUN_006ff8c0 | safedisc_resolve_user32_imports |  |
| 5054 | 0x006ff900 | FUN_006ff900 | safedisc_extract_under_mutex |  |
| 5055 | 0x006ff9d0 | FUN_006ff9d0 | safedisc_get_proc_address |  |
| 5056 | 0x006ffa20 | FUN_006ffa20 | safedisc_restore_import_directory |  |
| 5057 | 0x006ffa70 | FUN_006ffa70 | safedisc_load_extracted_dll |  |
| 5058 | 0x006ffad0 | FUN_006ffad0 | safedisc_load_library_from_dir |  |
| 5059 | 0x006ffc20 | FUN_006ffc20 | safedisc_pe_get_headers |  |
| 5060 | 0x006ffc40 | FUN_006ffc40 | safedisc_pe_drop_last_section |  |
| 5061 | 0x006ffc90 | FUN_006ffc90 | safedisc_pe_section_for_rva |  |
| 5062 | 0x006ffce0 | FUN_006ffce0 | safedisc_pe_get_section_count |  |
| 5063 | 0x006ffd10 | FUN_006ffd10 | safedisc_pe_set_section_count |  |
| 5064 | 0x006ffd40 | FUN_006ffd40 | safedisc_pe_section_lookup |  |
| 5065 | 0x006ffd80 | FUN_006ffd80 | safedisc_pe_headers_size_paged |  |
| 5066 | 0x006ffdd0 | FUN_006ffdd0 | safedisc_extract_payload |  |
| 5067 | 0x006fff20 | FUN_006fff20 | safedisc_make_temp_dir |  |
| 5068 | 0x006ffff0 | FUN_006ffff0 | safedisc_make_unique_subdir |  |
| 5069 | 0x007001c0 | FUN_007001c0 | safedisc_path_append |  |
| 5070 | 0x00700230 | FUN_00700230 | safedisc_make_dir_tree |  |
| 5071 | 0x00700490 | FUN_00700490 | safedisc_extract_archive |  |
| 5072 | 0x007008c0 | FUN_007008c0 | safedisc_extract_one_file |  |
| 5073 | 0x00700a90 | FUN_00700a90 | safedisc_decrypt_stream_to_file |  |
| 5074 | 0x00700c00 | FUN_00700c00 | safedisc_rollback_extraction |  |
| 5075 | 0x00700e90 | FUN_00700e90 | safedisc_create_lock_file |  |
| 5076 | 0x00701030 | FUN_00701030 | safedisc_launch_loader_process |  |
| 5077 | 0x00701370 | FUN_00701370 | safedisc_string_at |  |
| 5078 | 0x00701670 | FUN_00701670 | safedisc_string_decode |  |
| 5079 | 0x007016c0 | FUN_007016c0 | safedisc_string_decode_alloc |  |
| 5080 | 0x00701710 | FUN_00701710 | safedisc_report_getprocaddress_error |  |
| 5081 | 0x00701780 | FUN_00701780 | safedisc_report_createfile_error |  |
| 5082 | 0x00701800 | FUN_00701800 | safedisc_report_readfile_error |  |
| 5083 | 0x00701870 | FUN_00701870 | safedisc_report_writefile_error |  |
| 5084 | 0x007018d0 | FUN_007018d0 | safedisc_report_loadlibrary_error |  |
| 5085 | 0x00701950 | FUN_00701950 | safedisc_report_globalalloc_error |  |
| 5086 | 0x007019b0 | FUN_007019b0 | safedisc_report_createdirectory_error |  |
| 5087 | 0x00701a30 | FUN_00701a30 | safedisc_report_createprocess_error |  |
| 5088 | 0x00701ab0 | FUN_00701ab0 | safedisc_report_getfilesize_error |  |
| 5089 | 0x00701b20 | FUN_00701b20 | safedisc_report_createmutex_error |  |
| 5090 | 0x00701b90 | FUN_00701b90 | safedisc_format_error_prefix |  |
| 5091 | 0x00701c40 | FUN_00701c40 | safedisc_show_error_box |  |
| 5092 | 0x00701dc0 | FUN_00701dc0 | safedisc_stub_ret0 |  |
| 5093 | 0x00701dd0 | FUN_00701dd0 | safedisc_message_box |  |
| 5094 | 0x00701df0 | FUN_00701df0 | safedisc_itoa |  |
| 5095 | 0x00701e30 | FUN_00701e30 | safedisc_xtoa |  |
| 5096 | 0x00701ea0 | FUN_00701ea0 | safedisc_mbsrchr |  |
| 5097 | 0x00701ee0 | FUN_00701ee0 | safedisc_mbschr |  |
| 5098 | 0x00701f20 | FUN_00701f20 | safedisc_mbslen |  |
