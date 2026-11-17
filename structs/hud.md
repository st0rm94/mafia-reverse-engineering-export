# `G_IndicatorsClass` — the HUD singleton (`hud_state` @ `0x00658330`)

**Created** 2026-07-22 11:22 +0200 · **Modified** 2026-11-16 09:13 +0100

It owns *every* 2D screen element: the element-visibility flag word, the stopwatch, the in-car
instrument cluster, player and wingman indicators, weapon ammo, subtitles, mission help, intro
subtitles, the console ring, the compass, the fade overlay, the radar, the racing readouts, the map
overlay and the text / font engine. The companion prose doc (the `+0x90a4` bit map, the script
commands that drive HUD elements, in-game observations) is `basic/artifacts/hud_elements.md`.

| | |
|---|---|
| PC prefix | `hud_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x96ec`, one instance at `0x00658330`; `hud + 0x96ec` (`0x00661a1c`) is the separate global `g_font_translation_tables` |
| PC constructor | `hud_ctor` @ `0x00548bf0` |
| PC translation unit | `[0x00548830, 0x00556ee0)`, guard to guard; three members outside it (§4) |
| PS2 class | `G_IndicatorsClass` (tag `17G_IndicatorsClass`), instance `0x004eeff0`, 84 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class has no vtable: `+0x00` is the first font table.

```c
typedef struct G_IndicatorsClass {               // 0x96ec bytes, the singleton
    /* +0x0000 */ uint8_t font_table_a[4];             // [H] four 0x1400 blocks, 256 glyph records of 0x14 bytes
    /* +0x5000 */ S_gmf_font font_table_b[4];          // [H] four .gmf bodies (§1a)
    /* +0x9070 */ void *font_textures[4];              // [H] font%d_%s.tga, one per font
    /* +0x9080 */ void *interface_sheet;               // [H] 1int.tga: the pointer, the gear panel
    /* +0x9084 */ void *dial_texture;                  // [H] 2int*.tga by unit and night flag
    /* +0x9088 */ void *numbers_texture;               // [H] 1numbers.tga
    /* +0x908c */ void *race_texture;                  // [H] 0textura.tga
    /* +0x9090 */ void *map_texture;                   // [H] 0mapar.bmp, lazily loaded
    /* +0x9094 */ void *logoplay_texture;              // [H] logoplay.bmp
    /* +0x9098 */ void *sight_texture;                 // [H] 2sight.bmp
    /* +0x909c */ void *racestart_texture;             // [H] racestart.bmp
    /* +0x90a0 */ void *racewrong_texture;             // [H] racewrong.bmp
    /* +0x90a4 */ uint32_t element_flags;              // [H] the visibility word, = g_hud_flags
    /* +0x90a8 */ uint8_t flags_stack_alloc;           // [H] allocator byte
    /* +0x90ac */ uint32_t *flags_stack_begin;         // [H] save / restore stack of element_flags
    /* +0x90b0 */ uint32_t *flags_stack_end;           // [H] one past the last saved word
    /* +0x90b4 */ uint32_t *flags_stack_cap;           // [H] end of the allocation
    /* +0x90b8 */ float screen_width;                  // [H] the engine's screen width, vtable +0x6c
    /* +0x90bc */ float screen_height;                 // [H] the engine's screen height, vtable +0x70
    /* +0x90c0 */ float scale_x;                       // [H] instance twin of g_hud_scale_x
    /* +0x90c4 */ float scale_y;                       // [H] instance twin of g_hud_scale_y
    /* +0x90c8 */ uint8_t unused_90c8[0xc];            // [U] no access; no reader (.text scan)
    /* +0x90d4 */ S_vector2 stopwatch_origin;          // [H] top-left of the stopwatch dial, (694, 20) x scale
    /* +0x90dc */ uint8_t unused_90dc[8];              // [U] no access; no reader (.text scan)
    /* +0x90e4 */ G_RafickaValue stopwatch_hands[3];   // [H] hours, minutes, seconds; 0x24 bytes each
    /* +0x9150 */ uint32_t timer_hours;                // [H] stopwatch value
    /* +0x9154 */ uint32_t timer_minutes;              // [H] stopwatch value
    /* +0x9158 */ uint32_t timer_seconds;              // [H] stopwatch value
    /* +0x915c */ float interval_end_minute_frac;      // [H] end of the interval as a fraction of a turn, minute hand
    /* +0x9160 */ float interval_end_second_frac;      // [H] end of the interval as a fraction of a turn, second hand
    /* +0x9164 */ uint32_t timer_accumulator;          // [H] milliseconds into the current second
    /* +0x9168 */ uint32_t timer_interval;             // [H] milliseconds
    /* +0x916c */ G_RafickaValue needles[3];           // [H] speed (km/h), RPM, fuel; 0x24 bytes each
    /* +0x91d8 */ uint32_t unused_91d8;                // [U] no access; no reader (.text scan)
    /* +0x91dc */ float gear_displayed;                // [H] eased toward gear_target at 0.02 per ms
    /* +0x91e0 */ float gear_target;                   // [H] `gear * 11.0`
    /* +0x91e4 */ float needle_scale_x;                // [H] horizontal factor of hud_build_needle_quad
    /* +0x91e8 */ float needle_scale_y;                // [H] vertical factor of hud_build_needle_quad
    /* +0x91ec */ S_vector2 speedo_dial_pos;           // [H] origin of the 180 x 180 speedometer face: speed and fuel needles
    /* +0x91f4 */ S_vector2 rpm_dial_pos;              // [H] origin of the 76 x 76 tachometer face: RPM needle
    /* +0x91fc */ uint8_t automatic_gearbox;           // [H] A / M: UV select on 2intauto*.tga
    /* +0x91fd */ uint8_t low_fuel;                    // [H] the warning flag
    /* +0x9200 */ float low_fuel_phase;                // [H] 0.5 s blink; the dot draws below 0.25
    /* +0x9204 */ float car_damage_fill;               // [H] = g_hud_car_damage_bar_value
    /* +0x9208 */ uint32_t player_health;              // [H] the number
    /* +0x920c */ uint32_t ammo_mag;                   // [H] rounds in the magazine
    /* +0x9210 */ uint32_t ammo_reserve;               // [H] rounds in reserve
    /* +0x9214 */ char player_health_text[16];         // [H] "%d"
    /* +0x9224 */ char ammo_text[16];                  // [H] "%d/%d"
    /* +0x9234 */ float crosshair_spread;              // [H] bit 0x20
    /* +0x9238 */ uint32_t unused_9238;                // [U] no access; no reader (.text scan)
    /* +0x923c */ uint32_t action_prompt_alpha;        // [H] 0..0xff alpha of the action-prompt icon, flag bit 0x200
    /* +0x9240 */ uint32_t unused_9240;                // [U] no access; no reader (.text scan)
    /* +0x9244 */ uint32_t wingman_icon[4];            // [H] icon row, `& 7`
    /* +0x9254 */ uint32_t wingman_health[4];          // [H] default 100
    /* +0x9264 */ float crosshair_pos[2];              // [H] the screen centre, screen size / 2; bit 0x20
    /* +0x926c */ uint32_t unused_926c;                // [U] no access; no reader (.text scan)
    /* +0x9270 */ I3D_frame *compass_frame;            // [H] the compass destination
    /* +0x9274 */ char console_text[5][0x40];          // [H] the console ring
    /* +0x93b4 */ uint32_t console_color[5];           // [H] text colour per ring slot
    /* +0x93c8 */ uint32_t console_ttl[5];             // [H] 5000 ms
    /* +0x93dc */ uint32_t console_head;               // [H] the ring's head slot
    /* +0x93e0 */ uint32_t console_count;              // [H] live ring slots
    /* +0x93e4 */ uint8_t subtitles_alloc;             // [H] allocator byte
    /* +0x93e8 */ G_MISubtitleStruct *subtitles_begin; // [H] 0x10 bytes each, +0xc = TTL
    /* +0x93ec */ G_MISubtitleStruct *subtitles_end;   // [H] one past the last line
    /* +0x93f0 */ G_MISubtitleStruct *subtitles_cap;   // [H] end of the allocation
    /* +0x93f4 */ uint8_t mission_help_alloc;          // [H] allocator byte
    /* +0x93f8 */ G_MISubtitleStruct *mission_help_begin; // [H] the same record
    /* +0x93fc */ G_MISubtitleStruct *mission_help_end; // [H] one past the last line
    /* +0x9400 */ G_MISubtitleStruct *mission_help_cap; // [H] end of the allocation
    /* +0x9404 */ uint8_t intro_lines_alloc;           // [H] allocator byte
    /* +0x9408 */ S_intro_line *intro_lines_begin;     // [H] 0x1c bytes each (§1c)
    /* +0x940c */ S_intro_line *intro_lines_end;       // [H] one past the last line
    /* +0x9410 */ S_intro_line *intro_lines_cap;       // [H] end of the allocation
    /* +0x9414 */ uint8_t intro_waiting_alloc;         // [H] allocator byte
    /* +0x9418 */ G_MIIntroSubtitlesWaiting *intro_waiting_begin; // [H] {uint id; int wait_ms}
    /* +0x941c */ G_MIIntroSubtitlesWaiting *intro_waiting_end; // [H] one past the last queued id
    /* +0x9420 */ G_MIIntroSubtitlesWaiting *intro_waiting_cap; // [H] end of the allocation
    /* +0x9424 */ float blood_splat_ages[10];          // [H] ring of ten, bit 0x2000000
    /* +0x944c */ uint32_t blood_splat_count;          // [H] live splats in the ring
    /* +0x9450 */ float timed_action_fill;             // [H] timed-action bar, bit 0x1000
    /* +0x9454 */ float airplane_damage_fill;          // [H] bit 0x40000
    /* +0x9458 */ uint32_t letterbox_remaining;        // [H] ms, bit 0x4000
    /* +0x945c */ uint32_t letterbox_total;            // [H] ms
    /* +0x9460 */ uint32_t letterbox_diffuse;          // [H] animated diffuse
    /* +0x9464 */ uint32_t letterbox_direction;        // [H] whether the bars fade in or out
    /* +0x9468 */ uint32_t fade_remaining;             // [H] ms, bit 0x8000
    /* +0x946c */ uint32_t fade_total;                 // [H] ms
    /* +0x9470 */ uint32_t fade_diffuse;               // [H] the animated overlay colour
    /* +0x9474 */ uint32_t fade_direction;             // [H] whether the overlay fades in or out
    /* +0x9478 */ void *fade_texture;                  // [H] texture of the fade quad, bit 0x8000; only ever 0
    /* +0x947c */ float tint_alpha;                    // [H] 0..255; a viewport quad while > 0
    /* +0x9480 */ uint32_t tint_rgb;                   // [H] RGB of the full-screen tint
    /* +0x9484 */ char *overlay_texture_name;          // [H] compared to skip a reload
    /* +0x9488 */ void *overlay_texture;               // [H] released on change
    /* +0x948c */ uint8_t overlay_unmodulated;         // [H] set: texture in white + tint quad
    /* +0x9490 */ uint8_t radar_cars_alloc;            // [H] allocator byte
    /* +0x9494 */ S_radar_car *radar_cars_begin;       // [H] {C_car *, rgba}
    /* +0x9498 */ S_radar_car *radar_cars_end;         // [H] one past the last car
    /* +0x949c */ S_radar_car *radar_cars_cap;         // [H] end of the allocation
    /* +0x94a0 */ C_car *radar_player_car;             // [H] what the radar centres on
    /* +0x94a4 */ uint8_t radar_objects_alloc;         // [H] allocator byte
    /* +0x94a8 */ S_radar_object *radar_objects_begin; // [H] {tDynamicCollObject *, rgba}
    /* +0x94ac */ S_radar_object *radar_objects_end;   // [H] one past the last object
    /* +0x94b0 */ S_radar_object *radar_objects_cap;   // [H] end of the allocation
    /* +0x94b4 */ float unread_94b4;                   // [U] zeroed by hud_reset_values; no reader (.text scan)
    /* +0x94b8 */ float radar_blip_scale;              // [H] 0.9 about the player car
    /* +0x94bc */ uint32_t race_start_value;           // [H] start-light step, bit 0x200000
    /* +0x94c0 */ uint32_t race_start_prev;            // [H] the previous start-light step
    /* +0x94c4 */ float race_start_alpha;              // [H] fade of the current step
    /* +0x94c8 */ float race_start_prev_alpha;         // [H] fade of the previous step
    /* +0x94cc */ char race_time_text[0x40];           // [H] the formatted race time
    /* +0x950c */ char race_laptime_text[0x40];        // [H] the formatted lap time
    /* +0x954c */ char race_besttime_text[0x40];       // [H] the formatted best time
    /* +0x958c */ char race_position_text[0x40];       // [H] the formatted position
    /* +0x95cc */ char race_lap_text[0x40];            // [H] the formatted lap count
    /* +0x960c */ char race_banner_text[0x80];         // [H] strncpy 0x80, bit 0x400000
    /* +0x968c */ float race_banner_time;              // [H] banner display time left; the setter stores it, hud_tick steps it
    /* +0x9690 */ uint32_t freeride_score;             // [H] 0x7fffffff = hide
    /* +0x9694 */ char freeride_score_text[0x40];      // [H] "$%d"
    /* +0x96d4 */ uint32_t ms_accumulator;             // [H] += dt at the top of hud_tick
    /* +0x96d8 */ uint32_t police_symbol;              // [H] 1..4
    /* +0x96dc */ uint32_t police_symbol_rgba;         // [H] (alpha << 24) | 0xffffff
    /* +0x96e0 */ float police_wanted_fill;            // [H] the wanted bar, = g_police_wanted_bar
    /* +0x96e4 */ uint8_t map_enabled;                 // [H] the gate of bit 0x2000
    /* +0x96e8 */ char *map_caption;                   // [H] textdb 0x5f, "Out of City Limits"
} G_IndicatorsClass;
```

**Why four fonts:** `hud_load_font` takes an index 0..3 and four `font%d.gmf` files ship; a text draw
selects one with its last argument. **Why three hands and three needles:** each is one
`G_RafickaValue` (`0x24` bytes) that `hud_build_needle_quad` turns into a rotated quad against a
file-scope `G_RafickaDef`: `g_speedo_needle_def_speed`, `_rpm`, `_fuel` for the needles, and hours,
minutes, seconds for the stopwatch. **Why four wingman slots:** the setters test `slot < 4`, and
`hud_render_all` reads the four entries by constant offset. **Why five console and ten blood-splat
slots:** the ring modulus of `hud_console_push_message` and the cap of `hud_push_blood_splat`.

**The element records.** `G_MISubtitleStruct` (§1b) is the line record of the subtitle and the
mission-help vectors, `S_intro_line` (§1c) the line record of the intro-subtitle vector;
`G_MIIntroSubtitlesWaiting` is `{uint id; int wait_ms}`, and both radar lists have stride 8: a pointer
and an rgba.

### 1a. `S_gmf_font` — one `.gmf` body

`hud_load_font` reads `0x101c` bytes straight from the file into `font_table_b[index]`.

```c
typedef struct S_gmf_font {                   // 0x101c bytes
    /* +0x00 */ uint8_t unread_00[8];         // [U] file data; no reader (class census)
    /* +0x08 */ float line_height;            // [H] scales the draw height
    /* +0x0c */ float glyph_width_scale;      // [H] scales a glyph's width into its advance
    /* +0x10 */ float glyph_v_extent;         // [H] added to a glyph's v0 to give v1
    /* +0x14 */ float advance_overlap;        // [H] times the size, taken off every advance
    /* +0x18 */ float baseline;               // [H] subtracted from y, times the size
    /* +0x1c */ uint8_t glyphs[256][0x10];    // [H] one record per character code
} S_gmf_font;
```

`glyphs[256]`: one record per byte value; the text is translated through
`g_font_translation_tables[font]` before the index.

### 1b. `G_MISubtitleStruct` — one subtitle or mission-help line, `0x10` bytes

```c
typedef struct G_MISubtitleStruct {           // 0x10 bytes, the stride of the subtitle and mission-help vectors
    /* +0x00 */ char    *text;                // [H] start of the line's text
    /* +0x04 */ char    *text_end;            // [H] the break that ends the line, 0 on the last line
    /* +0x08 */ uint32_t delay_ms;            // [H] counts down in hud_tick; the line draws only at 0
    /* +0x0c */ uint32_t ttl_ms;              // [H] display time left; the record is erased when it runs out
} G_MISubtitleStruct;
```

Each adder splits its text with `hud_text_find_line_break` and pushes one record per line.

### 1c. `S_intro_line` — one intro-subtitle display line, `0x1c` bytes

```c
typedef struct S_intro_line {                 // 0x1c bytes, the stride of the intro-lines vector
    /* +0x00 */ float    x;                   // [H] screen x of the line, aligned by the header's C / R code
    /* +0x04 */ float    y;                   // [H] screen y of the line
    /* +0x08 */ float    size;                // [H] text size: 40 for a line opened by ^, else 27
    /* +0x0c */ char    *text;                // [H] start of the line's text
    /* +0x10 */ char    *text_end;            // [H] the ~ that ends the line, 0 on the last line
    /* +0x14 */ float    time_left_s;         // [H] seconds left; hud_tick erases the line below 0
    /* +0x18 */ float    age_s;               // [M] seconds since the line was added; its use in hud_render_all is not decoded
} S_intro_line;
```

`hud_intro_subtitles_add_intern` parses the text header `<wait_ms>,<duration_ms>,<x %>,<y %>,<C|R|other>~line~line`
into one record per `~`-separated line, `x %` and `y %` of the 800 x 600 virtual screen.

## 2. Field evidence

Measured by a register-tag census of `.text` over the whole singleton (ECX on entry to a `hud_`
method, `mov reg, 0x658330`, absolute operands inside the object, moves and stack slots followed), plus
the indexed accesses that census cannot see (the arrays), each read in its body. `render_all` draws
through a register cursor, so an offset search alone misses it.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0000 | font_table_a | H | - | `hud_text_draw_line_alt_font` | Read only by `hud_text_draw_line_alt_font`, whose only caller is the draw of the never-built `'edip'` widget (`structs/gui_widget.md` §6). Nothing fills it: the singleton lies in `.bss`, and no other function computes the `0x1400` font index |
| +0x5000 | font_table_b | H | `hud_load_font` | `hud_text_draw_line`, `hud_text_find_line_break`, `hud_text_measure` | `dtaRead` from `tables\font%d_%s.gmf` |
| +0x9070 | font_textures | H | `hud_destroy`, `hud_load_font` | `hud_destroy` | released and reloaded per slot |
| +0x9080 | interface_sheet | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_draw_mouse_cursor`, `hud_render_all` | - |
| +0x9084 | dial_texture | H | `hud_create`, `hud_destroy`, `hud_load_speedometer_odometer_texture` | `hud_destroy`, `hud_load_speedometer_odometer_texture`, `hud_render_all` | loaded by `hud_load_speedometer_odometer_texture` |
| +0x9088 | numbers_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x908c | race_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x9090 | map_texture | H | `hud_create`, `hud_destroy`, `hud_game_done`, `hud_tick` | `hud_destroy`, `hud_game_done`, `hud_render_all`, `hud_tick` | bound by `hud_render_all` (texture slot `+0xc`, a 2-triangle strip) |
| +0x9094 | logoplay_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x9098 | sight_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x909c | racestart_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x90a0 | racewrong_texture | H | `hud_create`, `hud_destroy` | `hud_create`, `hud_destroy`, `hud_render_all` | - |
| +0x90a4 | element_flags | H | `airplane_hud_damage_toggle`, `camera_begin_mode`, `camera_end_mode`, `engine_new_mission`, `entity_process_command`, `game_done`, `game_save_game_load`, `gui_menu_execute_modal`, `gui_notice_panel_present_once`, `hud_fade_set`, `hud_hide_element`, `hud_mission_help_add`, `hud_pop_element_flags`, `hud_push_blood_splat`, `hud_render_all`, `hud_reset_values`, `hud_set_element_flags`, `hud_set_race_banner_text`, `hud_set_race_start_flag`, `hud_show_element`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_tick`, `hud_widescreen_fade_bars`, `human_ai`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_dispatch_weapon_action_at_target`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_climb`, `human_do_climb_in_car_lr`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_play_anim`, `human_do_throw_grenade`, `human_drop_out_items`, `human_eject_out_of_car`, `human_hit`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_movement`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_update`, `human_use_actor`, `loadsave_init`, `player_ai`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk`, `player_indicator_set_aimed`, `player_invalidate_car_damage_bar`, `player_lock_controls`, `player_show_car_damage_bar`, `player_try_use_car`, `police_manager_tick`, `race_winners_replay_begin`, `race_winners_replay_end`, `racing_activate_racers`, `racing_tick`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | `camera_begin_mode`, `camera_end_mode`, `camera_free_look_base`, `engine_new_mission`, `entity_process_command`, `game_save_game_load`, `game_save_game_save`, `gui_menu_execute_modal`, `gui_notice_panel_present_once`, `hud_element_is_visible`, `hud_fade_get_state`, `hud_fade_is_active`, `hud_fade_set`, `hud_hide_element`, `hud_mission_help_add`, `hud_pop_element_flags`, `hud_push_blood_splat`, `hud_push_element_flags`, `hud_render_all`, `hud_set_race_banner_text`, `hud_set_race_start_flag`, `hud_show_element`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_tick`, `hud_widescreen_fade_bars`, `human_ai`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_dispatch_weapon_action_at_target`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_climb`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_play_anim`, `human_do_throw_grenade`, `human_drop_out_items`, `human_hit`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_movement`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_update`, `human_use_actor`, `pedestrian_manager_generate_passenger_for_taxi`, `player_ai`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk`, `player_indicator_set_aimed`, `player_lock_controls`, `player_try_use_car`, `police_manager_tick`, `racing_activate_racers`, `racing_tick`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | `hud_show_element` / `hud_hide_element` set and clear single bits, `hud_element_is_visible` tests one, `hud_set_element_flags` overwrites it (`basic/artifacts/hud_elements.md` §4) |
| +0x90a8 | flags_stack_alloc | H | `hud_ctor` | `hud_push_element_flags` | - |
| +0x90ac | flags_stack_begin | H | `hud_ctor` | `gui_notice_panel_present_once`, `hud_dtor`, `hud_pop_element_flags`, `hud_reset_values`, `race_winners_replay_end` | exists for the script language: both accessors have one caller, `program_process_command` |
| +0x90b0 | flags_stack_end | H | `gui_notice_panel_present_once`, `hud_ctor`, `hud_pop_element_flags`, `hud_reset_values` | `gui_notice_panel_present_once`, `hud_dtor`, `hud_pop_element_flags`, `hud_push_element_flags`, `hud_reset_values`, `race_winners_replay_begin`, `race_winners_replay_end` | - |
| +0x90b4 | flags_stack_cap | H | `hud_ctor` | - | - |
| +0x90b8 | screen_width | H | `hud_rebuild_layout_for_resolution` | `hud_render_all` | the engine's width (vtable `+0x6c`) as a float; the same call times 1/800 gives `scale_x` |
| +0x90bc | screen_height | H | `hud_rebuild_layout_for_resolution` | `hud_render_all` | the engine's height (vtable `+0x70`) as a float; the same call times 1/600 gives `scale_y` |
| +0x90c0 | scale_x | H | `hud_rebuild_layout_for_resolution` | `credits_render_one`, `game_done`, `garage_game_done`, `hud_build_ammo_icon_quads`, `hud_build_ammo_mag_reserve_quads`, `hud_build_indicator_icon_quads`, `hud_build_indicator_value_digits`, `hud_push_blood_splat`, `hud_render_all`, `hud_text_draw_line_scaled`, `hud_widescreen_build_bars` | §5 |
| +0x90c4 | scale_y | H | `hud_rebuild_layout_for_resolution` | `credits_render_one`, `game_done`, `garage_game_done`, `hud_build_ammo_icon_quads`, `hud_build_ammo_mag_reserve_quads`, `hud_build_indicator_icon_quads`, `hud_build_indicator_value_digits`, `hud_push_blood_splat`, `hud_render_all`, `hud_text_draw_line_scaled`, `hud_widescreen_build_bars` | §5 |
| +0x90c8 | unused_90c8 | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x90d4 | stopwatch_origin | H | `hud_rebuild_layout_for_resolution` | `hud_build_needle_quad` (through the address `hud_rebuild_layout_for_resolution` and `hud_tick` pass), `hud_tick` | the stopwatch dial's top-left, `(694, 20) x scale`; x at `+0x90d4`, y at `+0x90d8`; the dial quad is 86 x 105 from it, and each hand adds its def's pivot |
| +0x90dc | unused_90dc | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x90e4 | stopwatch_hands | H | `hud_tick`, `hud_timer_set_interval`, `hud_timer_set_time` | `hud_build_needle_quad`, `hud_rebuild_layout_for_resolution`, `hud_tick` | `hud_timer_set_time` seeds each first float with -1.0 |
| +0x9150 | timer_hours | H | `hud_tick`, `hud_timer_set_time` | `hud_tick`, `hud_timer_get_time` | - |
| +0x9154 | timer_minutes | H | `hud_tick`, `hud_timer_set_time` | `hud_tick`, `hud_timer_get_time`, `hud_timer_set_interval` | - |
| +0x9158 | timer_seconds | H | `hud_tick`, `hud_timer_set_time` | `hud_tick`, `hud_timer_get_time`, `hud_timer_set_interval` | - |
| +0x915c | interval_end_minute_frac | H | `hud_timer_set_interval` | `hud_tick` | `(minutes + interval / 60) / 60`; the red sector is drawn from the minute hand to it while more than 60 s are left (§6) |
| +0x9160 | interval_end_second_frac | H | `hud_timer_set_interval` | `hud_tick` | `(seconds + interval % 60) / 60`; the red sector is drawn from the second hand to it otherwise (§6) |
| +0x9164 | timer_accumulator | H | `hud_tick` | `hud_tick` | - |
| +0x9168 | timer_interval | H | `hud_tick`, `hud_timer_set_interval` | `hud_render_all`, `hud_tick`, `hud_timer_get_interval` | - |
| +0x916c | needles | H | `hud_speedometer_set_values` | `hud_build_needle_quad`, `hud_rebuild_layout_for_resolution`, `hud_speedometer_set_values` | `hud_speedometer_set_values` raises `low_fuel` when the fuel value is below 0.02 |
| +0x91d8 | unused_91d8 | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x91dc | gear_displayed | H | `hud_tick` | `hud_rebuild_layout_for_resolution`, `hud_tick` | `hud_tick` slides the `g_speedo_gear_vtx` window with it |
| +0x91e0 | gear_target | H | `hud_speedometer_set_gear` | `hud_tick` | - |
| +0x91e4 | needle_scale_x | H | `hud_rebuild_layout_for_resolution` | `hud_build_needle_quad`, `hud_rebuild_layout_for_resolution`, `hud_tick` | stored through the pointer the relayout forms to it |
| +0x91e8 | needle_scale_y | H | `hud_rebuild_layout_for_resolution` | `hud_build_needle_quad`, `hud_tick` | - |
| +0x91ec | speedo_dial_pos | H | `hud_rebuild_layout_for_resolution` | `hud_build_needle_quad` (through the address `hud_speedometer_set_values` passes), `hud_rebuild_layout_for_resolution` | the 180 x 180 speedometer face's origin; the speed and fuel needles are built from it |
| +0x91f4 | rpm_dial_pos | H | `hud_rebuild_layout_for_resolution` | `hud_build_needle_quad` (through the address `hud_speedometer_set_values` passes), `hud_rebuild_layout_for_resolution` | the 76 x 76 tachometer face's origin; the RPM needle is built from it |
| +0x91fc | automatic_gearbox | H | `hud_speedometer_set_gear` | `hud_render_all` | drawn 32 x 32 at `scale x 758, scale x 558` |
| +0x91fd | low_fuel | H | `hud_speedometer_set_values` | `hud_render_all`, `hud_tick` | the red dot is `2intbenal.tga` (`g_hud_low_fuel_lamp_texture`) |
| +0x9200 | low_fuel_phase | H | `hud_reset_values`, `hud_speedometer_set_values`, `hud_tick` | `hud_render_all`, `hud_tick` | - |
| +0x9204 | car_damage_fill | H | `player_ai`, `player_ai_drive`, `racing_activate_racers`, `racing_tick` | `hud_render_all` | `1.0 - car_get_condition_fraction()` |
| +0x9208 | player_health | H | `hud_set_player_health_value` | `hud_render_all` | - |
| +0x920c | ammo_mag | H | `hud_set_player_ammo_mag_reserve` | `hud_render_all` | - |
| +0x9210 | ammo_reserve | H | `hud_set_player_ammo_mag_reserve` | `hud_render_all` | - |
| +0x9214 | player_health_text | H | `hud_set_player_health_value` | `hud_render_all` | - |
| +0x9224 | ammo_text | H | `hud_set_player_ammo_mag_reserve` | `hud_render_all` | - |
| +0x9234 | crosshair_spread | H | `hud_set_player_accuracy` | `hud_render_all` | - |
| +0x9238 | unused_9238 | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x923c | action_prompt_alpha | H | `hud_tick` | `hud_render_all`, `hud_tick` | `hud_tick` steps it up by 2 x dt while flag bit `0x200` is set and down otherwise; `hud_render_all` draws the 33 x 33 action-prompt icon at x = 9 with it |
| +0x9240 | unused_9240 | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x9244 | wingman_icon | H | `hud_wingman_set_portrait_for_slot` | `hud_render_all` | four slots: the setter's `slot < 4` |
| +0x9254 | wingman_health | H | `hud_reset_values`, `hud_wingman_set_indicator_value_for_slot` | `hud_render_all` | - |
| +0x9264 | crosshair_pos | H | `hud_rebuild_layout_for_resolution` | `hud_render_all` | the screen centre, screen size / 2 |
| +0x926c | unused_926c | U | - | - | no load and no store: a register trace and a scan for `0x00658330 + off` as an absolute operand (`.text scan`) |
| +0x9270 | compass_frame | H | `hud_compass_set_destination_frame`, `hud_reset_values` | `hud_render_all` | - |
| +0x9274 | console_text | H | `hud_console_push_message` | `hud_render_all` | five slots of 0x40 characters: the modulus in `hud_console_push_message` |
| +0x93b4 | console_color | H | `hud_console_push_message` | `hud_render_all` | - |
| +0x93c8 | console_ttl | H | `hud_console_push_message`, `hud_tick` | `hud_tick` | - |
| +0x93dc | console_head | H | `hud_console_push_message`, `hud_reset_values`, `hud_tick` | `hud_console_push_message`, `hud_render_all`, `hud_tick` | - |
| +0x93e0 | console_count | H | `hud_console_clear`, `hud_console_push_message`, `hud_reset_values`, `hud_tick` | `hud_console_push_message`, `hud_render_all`, `hud_tick` | - |
| +0x93e4 | subtitles_alloc | H | `hud_ctor` | `hud_render_all`, `hud_subtitles_add`, `hud_subtitles_add_from_record` | - |
| +0x93e8 | subtitles_begin | H | `hud_ctor`, `hud_dtor` | `hud_dtor`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_subtitles_clear`, `hud_tick` | bit `0x40` |
| +0x93ec | subtitles_end | H | `hud_ctor`, `hud_dtor`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_subtitles_clear`, `hud_tick` | `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_subtitles_clear`, `hud_tick` | - |
| +0x93f0 | subtitles_cap | H | `hud_ctor`, `hud_dtor` | `hud_subtitles_add`, `hud_subtitles_add_from_record` | - |
| +0x93f4 | mission_help_alloc | H | `hud_ctor` | `hud_mission_help_add` | - |
| +0x93f8 | mission_help_begin | H | `hud_ctor`, `hud_dtor` | `hud_dtor`, `hud_mission_help_clear`, `hud_mission_help_set_time`, `hud_render_all`, `hud_tick` | bit `0x1000000` |
| +0x93fc | mission_help_end | H | `hud_ctor`, `hud_dtor`, `hud_mission_help_clear`, `hud_tick` | `hud_mission_help_add`, `hud_mission_help_clear`, `hud_mission_help_set_time`, `hud_render_all`, `hud_tick` | - |
| +0x9400 | mission_help_cap | H | `hud_ctor`, `hud_dtor` | `hud_mission_help_add` | - |
| +0x9404 | intro_lines_alloc | H | `hud_ctor` | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_render_all` | the three take the vector's address; the two adders push through it |
| +0x9408 | intro_lines_begin | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_dtor`, `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_intro_subtitles_clear`, `hud_tick` | - |
| +0x940c | intro_lines_end | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_intro_subtitles_clear`, `hud_tick` | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_intro_subtitles_clear`, `hud_tick` | - |
| +0x9410 | intro_lines_cap | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | - |
| +0x9414 | intro_waiting_alloc | H | `hud_ctor` | `hud_intro_subtitles_queue_id_range` | it takes the vector's address and pushes through it |
| +0x9418 | intro_waiting_begin | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_queue_id_range` | `hud_dtor`, `hud_intro_subtitles_clear`, `hud_intro_subtitles_queue_id_range`, `hud_tick` | - |
| +0x941c | intro_waiting_end | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_clear`, `hud_intro_subtitles_queue_id_range`, `hud_tick` | `hud_intro_subtitles_clear`, `hud_intro_subtitles_queue_id_range`, `hud_tick` | - |
| +0x9420 | intro_waiting_cap | H | `hud_ctor`, `hud_dtor`, `hud_intro_subtitles_queue_id_range` | `hud_intro_subtitles_queue_id_range` | - |
| +0x9424 | blood_splat_ages | H | `hud_push_blood_splat`, `hud_tick` | `hud_render_all`, `hud_tick` | ten slots: the cap in `hud_push_blood_splat` |
| +0x944c | blood_splat_count | H | `hud_push_blood_splat`, `hud_tick` | `hud_push_blood_splat`, `hud_render_all`, `hud_tick` | - |
| +0x9450 | timed_action_fill | H | `hud_reset_values`, `hud_set_timed_action_bar_value` | `hud_render_all` | the car break-in, the throw wind-up, the melee hold |
| +0x9454 | airplane_damage_fill | H | `hud_set_airplane_damage_fill` | `hud_render_all` | - |
| +0x9458 | letterbox_remaining | H | `hud_tick`, `hud_widescreen_fade_bars` | `hud_tick` | §4 |
| +0x945c | letterbox_total | H | `hud_widescreen_fade_bars` | `hud_tick` | - |
| +0x9460 | letterbox_diffuse | H | `hud_tick`, `hud_widescreen_fade_bars` | `hud_tick`, `hud_widescreen_fade_bars` | `hud_tick` clears its alpha byte (`+0x9463`) at the end |
| +0x9464 | letterbox_direction | H | `hud_widescreen_fade_bars` | `hud_tick` | - |
| +0x9468 | fade_remaining | H | `hud_fade_set`, `hud_tick` | `hud_fade_get_state`, `hud_tick` | - |
| +0x946c | fade_total | H | `hud_fade_set` | `hud_tick` | - |
| +0x9470 | fade_diffuse | H | `hud_fade_set`, `hud_tick` | `hud_render_all`, `hud_tick` | `hud_tick` clears its alpha byte (`+0x9473`) at the end |
| +0x9474 | fade_direction | H | `hud_fade_set` | `hud_tick` | - |
| +0x9478 | fade_texture | H | `hud_reset_values` | `hud_render_all` | the texture argument of the fade quad's engine `SetTexture` call under bit `0x8000` (`0x00552feb`); only `hud_reset_values` stores it, 0, so the fade quad is untextured |
| +0x947c | tint_alpha | H | `hud_reset_values`, `hud_set_screen_tint` | `hud_render_all` | set only by `hud_set_screen_tint`, from the PC-only `.tck` replay recorder |
| +0x9480 | tint_rgb | H | `hud_set_screen_tint` | `hud_render_all` | - |
| +0x9484 | overlay_texture_name | H | `hud_reset_values`, `hud_set_overlay_texture` | `hud_set_overlay_texture` | - |
| +0x9488 | overlay_texture | H | `hud_reset_values` | `hud_render_all`, `hud_set_overlay_texture` | - |
| +0x948c | overlay_unmodulated | H | `hud_set_overlay_texture` | `hud_render_all` | clear: one quad, modulated |
| +0x9490 | radar_cars_alloc | H | `hud_ctor` | `hud_radar_add_car_with_color` | - |
| +0x9494 | radar_cars_begin | H | `hud_ctor`, `hud_dtor` | `hud_dtor`, `hud_radar_add_car_with_color`, `hud_radar_clear`, `hud_radar_remove_car`, `hud_render_all`, `hud_reset_values` | - |
| +0x9498 | radar_cars_end | H | `hud_ctor`, `hud_dtor`, `hud_radar_clear`, `hud_radar_remove_car`, `hud_reset_values` | `hud_radar_add_car_with_color`, `hud_radar_clear`, `hud_radar_remove_car`, `hud_render_all`, `hud_reset_values` | - |
| +0x949c | radar_cars_cap | H | `hud_ctor`, `hud_dtor` | - | - |
| +0x94a0 | radar_player_car | H | `hud_radar_set_player_car`, `hud_reset_values` | `hud_render_all` | - |
| +0x94a4 | radar_objects_alloc | H | `hud_ctor` | `hud_radar_add_object_with_color` | - |
| +0x94a8 | radar_objects_begin | H | `hud_ctor`, `hud_dtor` | `hud_dtor`, `hud_radar_add_object_with_color`, `hud_radar_clear`, `hud_radar_remove_object`, `hud_render_all`, `hud_reset_values` | - |
| +0x94ac | radar_objects_end | H | `hud_ctor`, `hud_dtor`, `hud_radar_clear`, `hud_radar_remove_object`, `hud_reset_values` | `hud_radar_add_object_with_color`, `hud_radar_clear`, `hud_radar_remove_object`, `hud_render_all`, `hud_reset_values` | - |
| +0x94b0 | radar_objects_cap | H | `hud_ctor`, `hud_dtor` | - | - |
| +0x94b4 | unread_94b4 | U | `hud_reset_values` | - | one store, 0; no load by register or absolute address (`.text scan`) |
| +0x94b8 | radar_blip_scale | H | `hud_reset_values` | `hud_render_all` | - |
| +0x94bc | race_start_value | H | `hud_set_race_start_flag`, `hud_tick` | `hud_render_all` | §6 |
| +0x94c0 | race_start_prev | H | `hud_set_race_start_flag`, `hud_tick` | `hud_render_all`, `hud_set_race_start_flag` | - |
| +0x94c4 | race_start_alpha | H | `hud_reset_values`, `hud_set_race_start_flag`, `hud_tick` | `hud_render_all`, `hud_tick` | - |
| +0x94c8 | race_start_prev_alpha | H | `hud_reset_values`, `hud_set_race_start_flag`, `hud_tick` | `hud_render_all`, `hud_set_race_start_flag`, `hud_tick` | - |
| +0x94cc | race_time_text | H | `hud_reset_values`, `hud_set_race_time_text` | `hud_render_all` | - |
| +0x950c | race_laptime_text | H | `hud_reset_values`, `hud_set_race_laptime_text` | `hud_render_all` | - |
| +0x954c | race_besttime_text | H | `hud_reset_values`, `hud_set_race_besttime_text` | `hud_render_all` | - |
| +0x958c | race_position_text | H | `hud_reset_values`, `hud_set_race_position_text` | `hud_render_all` | - |
| +0x95cc | race_lap_text | H | `hud_reset_values`, `hud_set_race_lap_text` | `hud_render_all` | - |
| +0x960c | race_banner_text | H | `hud_set_race_banner_text` | `hud_render_all`, `hud_set_race_banner_text` | - |
| +0x968c | race_banner_time | H | `hud_set_race_banner_text`, `hud_tick` | `hud_render_all`, `hud_tick` | - |
| +0x9690 | freeride_score | H | `hud_freeride_update_score_text` | `hud_render_all` | Free Ride, not the racing mission |
| +0x9694 | freeride_score_text | H | `hud_freeride_update_score_text` | `hud_render_all` | - |
| +0x96d4 | ms_accumulator | H | `hud_reset_values`, `hud_tick` | `hud_tick`, `loadsave_savegame_save` | - |
| +0x96d8 | police_symbol | H | `entity_process_command`, `hud_reset_values`, `hud_set_police_symbol_and_alpha` | `hud_render_all` | - |
| +0x96dc | police_symbol_rgba | H | `entity_process_command`, `hud_reset_values`, `hud_set_police_symbol_and_alpha` | `hud_render_all` | - |
| +0x96e0 | police_wanted_fill | H | `hud_set_police_wanted_bar`, `police_manager_tick` | `hud_render_all` | - |
| +0x96e4 | map_enabled | H | `hud_map_enable`, `hud_reset_values` | `hud_render_all`, `hud_tick` | - |
| +0x96e8 | map_caption | H | `hud_map_enable`, `hud_reset_values` | `hud_render_all` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | unread_00 | U | `hud_load_font` (file) | - | no read body of the class loads it |
| +0x08 | line_height | H | `hud_load_font` (file) | `hud_text_draw_line` | - |
| +0x0c | glyph_width_scale | H | `hud_load_font` (file) | `hud_text_draw_line`, `hud_text_measure` | a glyph's width times it is the glyph's advance |
| +0x10 | glyph_v_extent | H | `hud_load_font` (file) | `hud_text_draw_line` | `v1 = v0 + field`: the v extent of a glyph cell |
| +0x14 | advance_overlap | H | `hud_load_font` (file) | `hud_text_draw_line` | times the size, taken off every advance |
| +0x18 | baseline | H | `hud_load_font` (file) | `hud_text_draw_line` | - |
| +0x1c | glyphs | H | `hud_load_font` (file) | `hud_text_draw_line`, `hud_text_measure`, `hud_text_find_line_break` | - |

### 2b. `G_MISubtitleStruct`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | text | H | `hud_mission_help_add`, `hud_subtitles_add`, `hud_subtitles_add_from_record` | `hud_render_all` | - |
| +0x04 | text_end | H | `hud_mission_help_add`, `hud_subtitles_add`, `hud_subtitles_add_from_record` | `hud_render_all` | the result of `hud_text_find_line_break` |
| +0x08 | delay_ms | H | `hud_mission_help_add`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_tick` | `hud_render_all`, `hud_tick` | every adder stores 0; a line draws only while it is 0 |
| +0x0c | ttl_ms | H | `hud_mission_help_add`, `hud_mission_help_set_time`, `hud_subtitles_add`, `hud_subtitles_add_from_record`, `hud_tick` | `hud_tick` | `hud_mission_help_set_time` retimes every queued line |

### 2c. `S_intro_line`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | x | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_render_all` | `x %` of 800, less the measured width for `R` or half of it for `C`; the centred adder uses 400 less half the width |
| +0x04 | y | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_render_all` | `y %` of 600; the centred adder uses 300 less half the size |
| +0x08 | size | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_render_all` | 40 or 27; the centred adder stores 25 |
| +0x0c | text | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_render_all` | - |
| +0x10 | text_end | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern` | `hud_render_all` | the next `~`, found by `strchr` |
| +0x14 | time_left_s | H | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_tick` | `hud_tick` | `hud_tick` subtracts dt and erases the line below 0 |
| +0x18 | age_s | M | `hud_intro_subtitles_add_centered`, `hud_intro_subtitles_add_intern`, `hud_tick` | `hud_render_all`, `hud_tick` | `hud_tick` adds dt; `hud_render_all` converts it with `__ftol` before the draw, and that use is not decoded |

## 3. Method family

84 PS2 methods: 65 CONFIRMED, 1 STRUCTURAL, 3 `inlined` rows, the rest in the PS2-only table of §7. The
PC adds functions with no twin; every CONFIRMED row was body-diffed on both sides. Three kinds of
row have status `-`: PC-only members, the unit's static-init rows, and its helpers.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00548830 | `crt_static_init_guard_00548830` | the translation unit | - | - | - | C++ static-initialization entry for one translation unit |
| 0x00548850 | `crt_atexit_register_00548850` | the translation unit | - | - | - | registers this translation unit's static destructor |
| 0x00548870 | `crt_static_init_zero_vector_00656930` | a file-scope zero vector | - | - | - | C++ dynamic initializer for the file-scope container at `0x00656930` |
| 0x00548ac0 | `hud_static_init` | the singleton | - | - | - | CRT static-init entry for the singleton. |
| 0x00548ad0 | `hud_static_ctor` | the singleton | - | - | - | CRT static constructor thunk. |
| 0x00548ae0 | `hud_register_atexit_dtor` | the singleton | - | - | - | CRT atexit registration for the singleton destructor. |
| 0x00548af0 | `hud_static_dtor` | the singleton | - | - | - | CRT static destructor thunk. |
| 0x00548b00 | `hud_dtor` | instance | `__dt__17G_IndicatorsClassFv` | 0x002b60c0 | CONFIRMED | frees the seven vectors in exact REVERSE order |
| 0x00548bf0 | `hud_ctor` | instance | `__ct__17G_IndicatorsClassFv` | 0x0029d380 | CONFIRMED | seven vector members, same order — the fingerprint |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0054aa30 | `hud_set_player_accuracy` | `PlayerSetAccuracy__17G_IndicatorsClassFf` | 0x002a14e0 | CONFIRMED | crosshair spread (+0x9234) |
| 0x0054be70 | `hud_console_clear` | `ConsoleClear__17G_IndicatorsClassFv` | 0x002a3c90 | CONFIRMED | zero the ring count |
| 0x0054c150 | `hud_compass_set_destination_frame` | `CompassSetDestination__17G_IndicatorsClassFP9I3D_frame` | 0x002a3e20 | CONFIRMED | +0x9270 target frame |
| 0x0054c160 | `hud_set_timed_action_bar_value` | `ThiefIndicatorSetValue__17G_IndicatorsClassFf` | 0x002a3e40 | CONFIRMED | timed-action bar => bit 0x1000 |
| 0x0054c170 | `hud_set_airplane_damage_fill` | `RefullingIndicatorSetValue__17G_IndicatorsClassFf` | 0x002a3e50 | CONFIRMED | airplane gauge fill; PS2 symbol is a dead dev name |
| 0x0054c620 | `hud_radar_set_player_car` | `RadarSetPlayerCar__17G_IndicatorsClassFP5C_car` | 0x002a4330 | CONFIRMED | +0x94a0; separated from its sibling by band layout |
| 0x005c7da0 | `hud_set_element_flags` | - | - | - | overwrites the whole `+0x90a4` flag word; 2 callers |
| 0x005c8010 | `hud_set_police_wanted_bar` | - | - | - | the police wanted bar fill ¹⁰ |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00424e20 | `hud_show_element` | - | - | - | set one `+0x90a4` bit ¹ |
| 0x00424e40 | `hud_set_police_symbol_and_alpha` | - | - | - | set the police status icon and its alpha ² |
| 0x00548d50 | `hud_reset_values` | `ResetValues__17G_IndicatorsClassFv` | 0x0029d5c0 | CONFIRMED | all elements to defaults; pins +0x9450=0.5f, +0x94b8=0.9f |
| 0x00548f90 | `hud_load_font` | `LoadFont__17G_IndicatorsClassFbUi` | 0x0029d7f0 | CONFIRMED | exact arity (bool,uint); PC 4 font slots vs PS2 2; names the two language globals |
| 0x00549060 | `hud_language_select` | `LanguageSelect__17G_IndicatorsClassFUi` | 0x0029da10 | CONFIRMED | builds/loads the `tables/lang_<xx>.def` record |
| 0x00549120 | `hud_create` | `Create__17G_IndicatorsClassFv` | 0x0029daf0 | CONFIRMED | one-time texture load; fills `g_radar_clip_texture` (radarclp.bmp) |
| 0x005492f0 | `hud_destroy` | `Destroy__17G_IndicatorsClassFv` | 0x0029de60 | CONFIRMED | releases all font + element textures |
| - | - | `UvolniMapu__17G_IndicatorsClassFb` | 0x0029e080 | inlined | inlined into `hud_destroy`; **inlined into `hud_destroy` + `hud_game_done`** — the +0x9090 release. PS2's bool arg (also release the dial +0x4084) is what became the PC split |
| 0x005494b0 | `hud_load_speedometer_odometer_texture` | - | - | - | per-scene dial-texture load; PS2 loads its dials inside GameInit/Create |
| 0x00549590 | `hud_game_done` | `GameDone__17G_IndicatorsClassFb` | 0x0029f990 | CONFIRMED | per-scene teardown; **exact 6-call sequence** — confirms 6 siblings too |
| 0x005495f0 | `hud_timer_set_time` | `TimerSetTime__17G_IndicatorsClassFUiUiUi` | 0x0029fb30 | CONFIRMED | stopwatch h/m/s value |
| 0x00549630 | `hud_timer_set_interval` | `TimerSetInterval__17G_IndicatorsClassFUi` | 0x0029fb50 | CONFIRMED | stopwatch interval (s) + derived hand angles |
| 0x005496e0 | `hud_timer_get_time` | `TimerGetTime__17G_IndicatorsClassFPUiPUiPUi` | 0x0029fc90 | CONFIRMED | read h/m/s back out |
| 0x00549710 | `hud_timer_get_interval` | `TimerGetInterval__17G_IndicatorsClassFPUi` | 0x0029fcb0 | CONFIRMED | read the interval back out |
| 0x00549770 | `hud_build_needle_quad` | `_SpocitejRafiku2__17G_IndicatorsClassFP12G_RafickaDefP14G_RafickaValueR9S_vector2P8TLVERTEX` | 0x0029fce0 | CONFIRMED | rotate one needle -> 4-corner quad; types `G_RafickaDef`/`G_RafickaValue` |
| 0x00549a80 | `hud_build_indicator_icon_quads` | `_PridejWingPodklad__17G_IndicatorsClassFP8TLVERTEXPUiUiUi` | 0x002a01a0 | CONFIRMED | wingman/health icon backing quad |
| 0x00549bc0 | `hud_build_ammo_icon_quads` | `_PridejAmmoPodklad__17G_IndicatorsClassFP8TLVERTEXPUi` | 0x002a0340 | CONFIRMED | ammo icon quad |
| 0x00549cb0 | `hud_build_indicator_value_digits` | `_PridejWingLives__17G_IndicatorsClassFP8TLVERTEXPUiUiUi` | 0x002a0450 | CONFIRMED | indicator value digits |
| 0x0054a080 | `hud_build_ammo_mag_reserve_quads` | `_PridejAmmo__17G_IndicatorsClassFP8TLVERTEXPUiUiUi` | 0x002a0890 | CONFIRMED | ammo mag/reserve digits |
| 0x0054a780 | `hud_speedometer_set_gear` | `TachometrSetGear__17G_IndicatorsClassFib` | 0x002a1100 | CONFIRMED | gear readout (`gear * 11.0`) |
| 0x0054a7a0 | `hud_speedometer_set_values` | `TachometrSetValues__17G_IndicatorsClassFffff` | 0x002a1130 | CONFIRMED | speed `*3.6` + 3 needles + odometer strip |
| 0x0054a9d0 | `hud_set_player_health_value` | `PlayerSetLives__17G_IndicatorsClassFUi` | 0x002a14a0 | CONFIRMED | player health number |
| 0x0054aa00 | `hud_set_player_ammo_mag_reserve` | `PlayerSetAmmo__17G_IndicatorsClassFUiUi` | 0x002a14c0 | CONFIRMED | ammo mag / reserve |
| 0x0054aa40 | `hud_wingman_set_portrait_for_slot` | `PlayerSetWingmanName__17G_IndicatorsClassFUiUi` | 0x002a14f0 | CONFIRMED | wingman icon row; `slot<4`, `& 7` |
| 0x0054aa60 | `hud_wingman_set_indicator_value_for_slot` | `PlayerSetWingmanLives__17G_IndicatorsClassFUiUi` | 0x002a1520 | CONFIRMED | wingman HEALTH |
| 0x0054aa80 | `hud_subtitles_clear` | `SubtitlesClear__17G_IndicatorsClassFv` | 0x002a3260 | CONFIRMED | clear the subtitle line list => bit 0x40 = SUBTITLES |
| 0x0054aad0 | `hud_subtitles_add` | `SubtitlesAdd__17G_IndicatorsClassFUiUib` | 0x002a3270 | STRUCTURAL | **PS2 body is an EMPTY STUB** — subtitles cut on PS2; mapping is one-sided |
| 0x0054ae30 | `hud_subtitles_add_from_record` | - | - | - | overload; subtitles were cut on PS2 (`SubtitlesAdd` is an empty stub), so no PS2 overload exists |
| 0x0054b160 | `hud_intro_subtitles_add_intern` | `IntroSubtitlesAddIntern__17G_IndicatorsClassFUif` | 0x002a3280 | CONFIRMED | parse the `~` / `,` layout header; 800x600 vs 640x480 |
| 0x0054b5a0 | `hud_intro_subtitles_queue_id_range` | `IntroSubtitlesAdd__17G_IndicatorsClassFUiUi` | 0x002a35f0 | CONFIRMED | queue `{id, atoi(text)}`; element `G_MIIntroSubtitlesWaiting` |
| 0x0054b7b0 | `hud_intro_subtitles_add_centered` | - | - | - | Slot matches `IntroSubtitlesAddSimple` but the BODIES DISAGREE — PS2 rewrote its version for the PS2-only hot-rod bonus-car unlock. Twin deliberately not applied. |
| 0x0054ba60 | `hud_intro_subtitles_clear` | `IntroSubtitlesClear__17G_IndicatorsClassFv` | 0x002a3740 | CONFIRMED | clears BOTH intro containers |
| 0x0054bad0 | `hud_mission_help_clear` | `MissionHelpClear__17G_IndicatorsClassFv` | 0x002a3770 | CONFIRMED | clear the mission-help line list |
| 0x0054bb20 | `hud_mission_help_add` | `MissionHelpAdd__17G_IndicatorsClassFUi` | 0x002a3780 | CONFIRMED | TTL magic `len*0x28+0x9c4` => bit 0x1000000 = MISSION HELP |
| 0x0054be10 | `hud_mission_help_set_time` | `MissionHelpSetTime__17G_IndicatorsClassFUi` | 0x002a3930 | CONFIRMED | retime every queued line; pins record +0xc = TTL |
| 0x0054be40 | `hud_map_enable` | `MapEnable__17G_IndicatorsClassFb` | 0x002a3c40 | CONFIRMED | byte-exact; +0x96e4 bool + textdb 0x5f cache |
| 0x0054be80 | `hud_console_push_message` | `ConsoleAddText__17G_IndicatorsClassFPUcUi` | 0x002a3ca0 | CONFIRMED | ring push; PC 5 slots / 5.0 s vs PS2 3 / 2.0 s |
| 0x0054bf10 | `hud_console_print` | `ConsoleAddTextByID__17G_IndicatorsClassFUiUi` | 0x002a3d70 | CONFIRMED | textdb lookup -> push; PS2 also de-dupes |
| 0x0054bf40 | `hud_push_blood_splat` | - | - | - | the screen blood-splat ring (bit `0x2000000`): `human_hit` pushes one randomly rotated splat when the player is hit; `hud_render_all` draws it with `2blood.bmp`. The PS2 has the ring but no push (§7c) |
| 0x0054c180 | `hud_widescreen_build_bars` | `WideScreenSetRatio__17G_IndicatorsClassFf` | 0x002a3e60 | CONFIRMED | builds the letterbox bars ⇒ **flag bit 0x4000 is the cinematic letterbox** ³ |
| 0x0054c420 | `hud_widescreen_fade_bars` | - | - | - | fades the letterbox bars in/out over a duration; the PS2 has only `WideScreenSetRatio`. Shaped exactly like `hud_fade_set` |
| 0x0054c490 | `hud_fade_set` | `FadeInOut__17G_IndicatorsClassFbUiUi` | 0x002a4250 | CONFIRMED | statement-for-statement identical |
| 0x0054c500 | `hud_fade_get_state` | `FadeInOutGetState__17G_IndicatorsClassFv` | 0x002a42b0 | CONFIRMED | same predicate |
| 0x0054c520 | `hud_fade_is_active` | `FadeInOutIsDark__17G_IndicatorsClassFv` | 0x002a42e0 | CONFIRMED | bit 0x8000 test |
| 0x0054c530 | `hud_set_screen_tint` | - | - | - | the full-screen tint (+0x947c alpha / +0x9480 RGB) that the `.tck` replay recorder drives ⁴ |
| 0x0054c550 | `hud_set_overlay_texture` | - | - | - | the full-screen overlay texture ⁴ |
| 0x0054c5c0 | `hud_radar_clear` | `RadarClear__17G_IndicatorsClassFv` | 0x002a4300 | CONFIRMED | clears BOTH blip lists |
| 0x0054c630 | `hud_radar_add_car_with_color` | `RadarAddCar__17G_IndicatorsClassFP5C_carUi` | 0x002a4340 | CONFIRMED | membership upsert; PC stores rgba, PS2 a type index + limit |
| 0x0054c840 | `hud_radar_remove_car` | `RadarRemoveCar__17G_IndicatorsClassFP5C_car` | 0x002a4410 | CONFIRMED | find + erase-by-shift |
| 0x0054c8a0 | `hud_radar_add_object_with_color` | `RadarAddObject__17G_IndicatorsClassFP18tDynamicCollObjectUi` | 0x002a4470 | CONFIRMED | the `tDynamicCollObject` list at +0x94a8 |
| 0x0054cab0 | `hud_radar_remove_object` | `RadarRemoveObject__17G_IndicatorsClassFP18tDynamicCollObject` | 0x002a4500 | CONFIRMED | find + erase-by-shift on the object list |
| 0x0054cb10 | `hud_radar_set_left_edge` | `RadarSetLeftEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>` | 0x002a4560 | CONFIRMED | `vector<C_web_node*>` -> TLVERTEX strip (nav web) |
| 0x0054cd10 | `hud_radar_set_right_edge` | `RadarSetRightEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>` | 0x002a4840 | CONFIRMED | mirror global set |
| 0x0054cf10 | `hud_set_race_start_flag` | `RaceSetStartFlag__17G_IndicatorsClassFUi` | 0x002a4b20 | CONFIRMED | bit 0x200000; the race start-light countdown — see §7c |
| 0x0054cfc0 | `hud_set_race_time_text` | `RaceSetTime__17G_IndicatorsClassFUi` | 0x002a4b90 | CONFIRMED | textdb 0x2c8 |
| 0x0054d000 | `hud_set_race_laptime_text` | `RaceSetLapTime__17G_IndicatorsClassFUi` | 0x002a4c20 | CONFIRMED | textdb 0x2d0 |
| 0x0054d040 | `hud_set_race_besttime_text` | `RaceSetBestTime__17G_IndicatorsClassFUi` | 0x002a4cb0 | CONFIRMED | textdb 0x2c9 |
| 0x0054d080 | `hud_set_race_position_text` | `RaceSetPosition__17G_IndicatorsClassFUiUi` | 0x002a4d40 | CONFIRMED | textdb 0x2ca |
| 0x0054d0b0 | `hud_set_race_lap_text` | `RaceSetLaps__17G_IndicatorsClassFUiUi` | 0x002a4da0 | CONFIRMED | textdb 0x2cb |
| 0x0054d0e0 | `hud_set_race_banner_text` | `RaceFlashText__17G_IndicatorsClassFPUcf` | 0x002a4e10 | CONFIRMED | bit 0x400000, strncpy 0x80 |
| 0x0054d130 | `hud_freeride_update_score_text` | `BonusSetCount__17G_IndicatorsClassFi` | 0x002a4e60 | CONFIRMED | textdb 0xb3; 0x7fffffff = hide |
| 0x0054d180 | `hud_render_all` | `DrawAll__17G_IndicatorsClassFv` | 0x002a4ed0 | CONFIRMED | shared `& 0x801` master early-out |
| - | - | `DrawMap__17G_IndicatorsClassFfff` | 0x002ab690 | inlined | inlined into `hud_render_all`; **inlined into `hud_render_all`**, the bit-0x2000 branch ¹³ |
| 0x005532e0 | `hud_rebuild_layout_for_resolution` | `ResolutionChanged__17G_IndicatorsClassFv` | 0x002ad720 | CONFIRMED | the 8736-byte relayout pass ⁵ |
| 0x005540b0 | `hud_tick` | `Tick__17G_IndicatorsClassFUi` | 0x002af980 | CONFIRMED | dt accumulate + age every timed field |
| - | - | `NahrajMapu__17G_IndicatorsClassFv` | 0x0029dfe0 | inlined | inlined into `hud_tick`; **inlined into `hud_tick`** — lazy load of `0mapar.bmp` into +0x9090 ¹² |
| 0x00554fd0 | `hud_text_measure` | `TextSize__17G_IndicatorsClassFPUcfUiPUc` | 0x002b2540 | CONFIRMED | names the global `g_font_translation_tables` = `fontTranstable` |
| 0x00555090 | `hud_text_find_line_break` | `HledejZlom__17G_IndicatorsClassFPUcffUiPUc` | 0x002b2670 | CONFIRMED | the word-wrap primitive |
| 0x005551a0 | `hud_text_draw_line` | `OutText__17G_IndicatorsClassFPUcffffUiUiUiPUc` | 0x002b27d0 | CONFIRMED | the main text primitive, 70 call sites ⁷ |
| 0x00556040 | `hud_text_draw_multiline` | `OutTextMultiLine__17G_IndicatorsClassFPUcffffUiUiUi` | 0x002b3200 | CONFIRMED | one `hud_text_draw_line` per `~`-separated line ⁸ |
| 0x005560b0 | `hud_text_measure_fixed_pitch` | - | - | - | the measure half of the alternate-font pair: `TextSize`'s arity, but NO glyph table — returns `count * size * 0.5` |
| 0x00556100 | `hud_text_draw_line_alt_font` | - | - | - | SECOND line renderer, reading font table A (`this+0x0000`) instead of table B (§1, §7b) ⁹ |
| 0x00556d80 | `hud_draw_mouse_cursor` | - | - | - | one 15x15 quad from `1int.tga` (`hud+0x9080`), drawn only for `g_gui_active_screen` after all its children; the PS2 has no mouse. See `structs/gui_screen.md` |
| 0x00560550 | `hud_text_draw_line_scaled` | - | - | - | scales x/width by `hud+0x90c0` and y/height by `hud+0x90c4`, then forwards to `hud_text_draw_line`; sole consumer `credits_render_one` (§5) |
| 0x005c7db0 | `hud_hide_element` | - | - | - | clear one `+0x90a4` bit ¹ |
| 0x005c7dd0 | `hud_element_is_visible` | - | - | - | test one `+0x90a4` bit ¹ |
| 0x005c7df0 | `hud_push_element_flags` | - | - | - | pushes `+0x90a4` onto the save/restore stack; sole caller `program_process_command` |
| 0x005c7fb0 | `hud_pop_element_flags` | - | - | - | pops the save/restore stack back into `+0x90a4`; sole caller `program_process_command` |
| 0x005c8db0 | `hud_set_car_damage_bar_enabled` | - | - | - | arms the HUD car-damage bar (bit `0x10000`); not a member of this class ¹¹ |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00549730 | `vec2_mul_to` | - | - | - | > void NOT a HUD method — an inlined `S_vector2` operator emitted into the HUD translation unit |
| 0x00549750 | `vec2_add_to` | - | - | - | > void method — an inlined `S_vector2` operator emitted into the HUD translation unit |
| 0x00553ed0 | `vec2_div_scalar_to` | - | - | - | component-wise `out = a * (1 / s)` on 2-component float vectors |
| 0x00553ef0 | `hud_build_rect_quad` | - | - | - | geometry helper for `hud_rebuild_layout_for_resolution` (8 call sites, all in it); free function, no receiver ⁵ |
| 0x00553fd0 | `hud_build_diamond_quad` | - | - | - | the 45-degree rhombus variant of `hud_build_rect_quad`; 1 call site, the speed-limiter icon ⁶ |
| 0x00556ea0 | `std_vector_end` | - | - | - | returns a `std::vector`'s end pointer |
| 0x00556eb0 | `std_vector1c_size` | - | - | - | element count of a `std::vector` whose elements are 0x1c bytes |

¹ `hud_show_element` / `hud_hide_element` / `hud_element_is_visible`: the PS2 has no
Show/Hide/IsVisible method in the 84-symbol enumeration — it pokes the flat flag global inline at
every site.
² The PS2 fields exist (`+0x4e18`/`+0x4e1c`) and `DrawAll` reads them, but the entire PS2 decomp
contains only the `ResetValues` init and that read — nothing ever sets the police symbol on PS2.
³ Statement-for-statement, six shared constants: `801.0`, `601.0`, the ratio multiply,
`0xc0000000`, the diffuse `0xaf0000ff`, `+3.0`, `(h-band)*0.5`. Both write a file-scope vertex
array.
⁴ `hud_set_screen_tint` and `hud_set_overlay_texture` are the two surplus PC functions in the
`FadeInOutIsDark` -> `RadarClear` gap, between the confirmed fade trio and `hud_radar_clear`;
neither has a counterpart in the PS2 method list.
⁵ The twin rests on the shared four-field screen-scale band, the same opening screen-size query and
the emission position before `Tick`. `hud_rebuild_layout_for_resolution` out-lines three quad
helpers (`hud_build_rect_quad`, `hud_build_diamond_quad`, `vec2_div_scalar_to`) that PS2 inlines
into `ResolutionChanged`.
⁶ Builds `g_speedo_speed_limiter_icon_vtx` (20x20 at 760,380), drawn as the last 2 triangles of the
dial-sheet batch while flag bit `0x100` is set.
⁷ Arity 9 (`ret 0x24`) matches `PUc f f f f Ui Ui Ui PUc` exactly; byte-verified font stride
`idx*0x101c + 0x5018`; LOCKED from the caller side by `hud_text_draw_multiline`.
⁸ Statement-for-statement: `while (p) { p = strchr(text, 0x7e); OutText(..., p); y += lineHeight;
text = p+1; }`. Arity 8; ratio 112/212 = 0.53, in this class band.
⁹ Same `ret 0x24` arity, same translation table and same 400-vertex cap as `hud_text_draw_line`;
the only difference is the 0x1400-stride glyph table at `this+0x0000` instead of the 0x101c one at
`+0x5000`.
¹⁰ The PS2 field `+0x4e20` (bracketed by the confirmed `+0x4e1c` and `+0x4e24` pairs) has ZERO
references anywhere in the PS2 decomp.
¹¹ A plain `__fastcall` on a separate BSS global, not on the HUD singleton. It carries the `hud_`
prefix because the byte it stores arms the HUD bar.
¹² Gated on the map-enabled bool `+0x96e4`, so it only loads once the map is available. The PS2
also resolves 43 `onMapLabels` entries; the PC inline has no label pass.
¹³ World->map UV: `u = x*0.00022986 + 0.7346424`, `v = z*-0.00050728 + 0.470246`. The PC branch
has NO scroll/zoom state — the code-side reason the PC map cannot scroll.

## 4. The PC code is four clusters

A class's PC code is not one contiguous cluster, and this one is the clearest example.

| PC range | Contents |
|---|---|
| `0x00548ac0`–`0x0054d180` | the bulk: lifecycle, timers, instruments, indicators, text, console, radar, race, `hud_render_all` |
| `0x005532e0`–`0x00556d80` | the relayout, `hud_tick` and the text / font engine |
| `0x00424e20` | `hud_show_element` and `hud_set_police_symbol_and_alpha` |
| `0x005c7da0`–`0x005c8db0` | the flag-word accessors, `hud_set_police_wanted_bar`, `hud_set_car_damage_bar_enabled` |

Emission order holds **within** a cluster only. `airplane_hud_damage_toggle` (`0x004538b0`) is a
`C_airplane` method that drives this HUD. The unit also holds six functions foreign to this class, which
shift its emission-order ranks: `recorder_key_stream_new_chunk`, `recorder_key_stream_alloc`,
`recorder_key_stream_size`, `actor_track_key_offset_of`, `morph_frame_list_add` (gates on the `'MRPH'`
magic) and `speech_pause_all_channels`; the first four are listed in `structs/recorder.md` and
`structs/actor_track.md`.

**The letterbox and the bars.** `hud_widescreen_fade_bars` writes the `letterbox_*` fields; the bars
themselves live in the file-scope `g_hud_widescreen_bar_verts` (twelve TLVERTEXes = two 6-vertex
quads), built by `hud_widescreen_build_bars`.

**Related file-scope globals:** `g_font_translation_tables` = the per-font character translation
tables; `g_radar_left_edge_vtx` / `g_radar_left_edge_points` / `g_radar_left_edge_count` = the radar's
**left** road-edge strip, points and count; `g_radar_right_edge_vtx` / `g_radar_right_edge_points` =
the **right** mirror; `g_transmission_indicator_texture` = the auto-transmission dial overlay
(`2intauto*.tga`), loaded alongside `dial_texture`; `g_speed_unit_is_mph` = the mph / kph flag;
`g_speedo_dial_verts` = the speedometer and tachometer dial faces (12 TLVERTEXes, the base of the
dial-sheet draw); `g_speedo_gear_vtx` = the gear-panel quad; `g_radar_clip_texture` = `radarclp.bmp` at
`0x00658328`, bound by `hud_render_all` like `map_texture`.

## 5. The two resolution-scale globals

`g_hud_scale_x` (`0x006613f0`, 21 refs) and `g_hud_scale_y` (`0x006613f4`, 24 refs) are the horizontal
and vertical resolution scale factors every virtual-coordinate draw call multiplies by. They are the
class fields `scale_x` and `scale_y` by address (`0x00658330 + 0x90c0`).

They are cited from the `g_loadsave` translation unit because that unit holds an off-family helper:
`hud_text_draw_line_scaled` (`0x00560550`), whose only consumer is `credits_render_one`, which is why
it was emitted next to the credits code. Its definition lives inside `[0x00556f10, 0x00561b70)`; only
its receiver is the HUD singleton.

## 6. The stopwatch and the start lights

**The stopwatch.** `hud_timer_set_time` stores h / m / s and seeds the hands; `hud_timer_set_interval`
stores the interval and derives `interval_end_minute_frac` / `interval_end_second_frac`, where the
interval ends on the minute and the second hand; `hud_tick` advances `timer_accumulator`, carries into
the seconds, minutes and hours, and draws the red sector from the minute hand to its end while more than
60 s are left, else from the second hand.

**`hud_set_race_start_flag` — what the widget depicts.** The PC hide-gate (`!= 4 && != -1`) looks like
a place domain of at most four, but the caller settles it: `racing_tick` drives a **1000 ms-per-step
start countdown** that calls this with the step counter `C_racing+0x2a8`: 0, then 1, then 2, then 3,
and at 4 releases the drivers, stamps the race start time and switches to the racing state. So
`param_2 == 4` means *the race has started, hide the lights*, and the `<= 4` domain is the countdown
length, not a placing. Detail: `basic/artifacts/racing.md`, `structs/racing.md`.

## 7. PC vs PS2

### 7a. The field rebase is band-wise, not one constant

Do **not** port a PS2 offset by adding one delta. Measured anchors:

| Band | PC | PS2 | Delta |
|---|---|---|---|
| flag word | `+0x90a4` | `+0x40a4` | `+0x5000` |
| stopwatch / instrument cluster | `+0x9150` | `+0x4154` | `+0x4ffc` |
| player health / ammo / accuracy | `+0x9208` | `+0x4204` | `+0x5004` |
| wingman slots | `+0x9244` | `+0x4244` | `+0x5000` |
| compass | `+0x9270` | `+0x4270` | `+0x5000` |
| console ring | `+0x9274` | `+0x4278` | `+0x4ffc` |
| subtitles vector | `+0x93e8` | `+0x4b38` | `+0x48b0` |
| intro subtitles / fade / timed-action bar | `+0x9408` / `+0x9468` / `+0x9450` | `+0x4b50` / `+0x4bb0` / `+0x4b98` | `+0x48b8` |
| radar | `+0x9494` | `+0x4bd8` | `+0x48bc` |
| race / map | `+0x94bc` / `+0x96e4` | `+0x4bfc` / `+0x4e24` | `+0x48c0` |

The drift starts at the **console ring**, which the PC widened from 3 slots to 5, and grows again across
the subtitle and intro-subtitle containers.

### 7b. The PC and PS2 HUDs are not the same product

Recorded because a PS2 body-diff is the main tool for this class, and these are exactly the places
where a diff is *supposed* to fail. Full table with evidence: `basic/artifacts/hud_elements.md` §8.

- **Map:** the PC map is **not scrollable** *(user-confirmed in-game)*; PS2 has `MapGetScrollPos` and
  a `DrawMap(f,f,f)` taking scroll/zoom arguments. No PC twin for `MapGetScrollPos` exists, which is
  consistent, and the PC `DrawMap` branch carries no scroll state to query.
- **Radar:** on PS2 it is a **moving minimap** (the map image scrolls under the player). On PC it is
  **blips over a partially transparent background sprite** — `radarclp.bmp`, loaded by `hud_create`
  into `g_radar_clip_texture`. **No minimap and no road outline in normal play**
  (user-confirmed in game): coloured vehicle rectangles sized by vehicle, yellow trams, and a red
  cross at the compass destination. The road-edge strips are **race-track** edges and appear only
  during the racing mission — `hud_radar_set_left_edge` / `_right_edge` are only ever fed a real
  node run by `racing_activate_racers` (VA `0x00611430`); everywhere else they are called with `0`.
  The divergence is therefore in the **renderer**, not in the data feeding it: both builds share the
  edge builders and both blip lists. The PC renderer is `hud_render_all`, under flag bit
  `0x00080000`. It walks the two lists through a register cursor — which is why offset searches for
  `+0x9494` do not find it — builds each car blip as a rectangle sized by `vehicle_get_size()`, and
  draws the destination cross in `0xffff0000` from `compass_frame`.
- **Trams ride the OBJECT list, not the car list.** `railway_update` (VA `0x004da400`, = PS2
  `Update__9C_railwayFUi`) registers the tram engine **and each wagon separately** through
  `hud_radar_add_object_with_color(part, 0xffffff00)` — yellow, byte-identical on both builds. A tram
  is a `tDynamicCollObject`, so it can never enter the `C_car` list; that is why it has its own colour.
- **Subtitles:** cut on PS2 (`SubtitlesAdd` is an empty stub) — hence the single STRUCTURAL row.
- **`IntroSubtitlesAddSimple`:** rewritten on PS2 for the PS2-only hot-rod bonus-car unlock; the twin
  is deliberately **not** applied.
- **Console ring:** PC 5 slots / 5.0 s; PS2 3 slots / 2.0 s **plus** a same-id repeat suppressor the
  PC build does not have.
- **Compass:** PS2 has two destination slots, PC one.
- **Tachometer (RPM):** PC has a third instrument needle at `+0x9190`; **PS2 does not** — its
  corresponding field `+0x4194` has zero references and `TachometrSetValues` builds only two needles.
- **Transmission indicator (A/M):** PC's `hud_render_all` picks an `A` vs `M` sprite from `+0x91fc`;
  PS2 stores the same field but never draws it — the PS2 game is automatic-only.
- **Inventory screen:** PS2-only (3 methods).
- **Layout:** PC 800×600 virtual screen, PS2 640×480; PC glyph atlases ÷256/÷128, PS2 ÷255/÷127.
- **Vertex format:** PC TLVERTEX stride `0x20`, PS2 `0x1c` — every quad builder differs by this and
  nothing else.
- **File-scope globals (§4):** `g_font_translation_tables` = PS2 `fontTranstable`;
  `g_speedo_dial_verts` = PS2 `_VtxTachometrCiselnik`.

### 7c. The methods with no PC function

Each verdict rests on the method's callers, not on its absence from the PC cluster.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0029d9a0 | `LoadFontFake__17G_IndicatorsClassFUiUi` | copies font slot 0 into slots 2 and 3 (§7c) |
| 0x0029e0e0 | `GameInit__17G_IndicatorsClassFv` | folded into `game_init` — anchor: the `s_GameInitStart_00650f64` literal sits immediately before the surviving `hud_radar_clear()` + `hud_load_speedometer_odometer_texture()` block |
| 0x002a1640 | `DrawInventory__17G_IndicatorsClassFv` | console inventory screen |
| 0x002a16f0 | `InventoryScroll__17G_IndicatorsClassFP11G_InventoryUiUii` | console inventory screen |
| 0x002a1c40 | `_InventorySet__17G_IndicatorsClassFiiiii` | console inventory screen (5664 B) |
| 0x002a3690 | `IntroSubtitlesAddSimple__17G_IndicatorsClassFUi` | PC `hud_intro_subtitles_add_centered` (0x0054b7b0) holds the slot, but the PS2 method was rewritten for the PS2-only hot-rod unlock — twin deliberately NOT applied |
| 0x002a3970 | `TutorialAddText__17G_IndicatorsClassFUi` | tutorial text (§7c) |
| 0x002a3c80 | `MapIsEnabled__17G_IndicatorsClassFv` | 1-line getter; no distinct PC function |
| 0x002a3e30 | `CompassSetDestination2__17G_IndicatorsClassFP9I3D_frame` | second compass slot; PC has one destination only |
| 0x002ab490 | `MapGetScrollPos__17G_IndicatorsClassFRfRff` | the PC map does not scroll — confirmed in-game AND in code: the PC `DrawMap` branch carries no scroll state to query |
| 0x002ad540 | `DrawMessage__17G_IndicatorsClassFPUc` | a centred message box: `TextSizeMultiLine`, two `RectangleFilled` calls, then `OutTextMultiLine` (§7c) |
| 0x002af940 | `AByloTicho__17G_IndicatorsClassFi` | dev-joke name; mutes `isound` for N HUD ticks through the countdown at PS2 `+0x4e30` (§7c) |
| 0x002b2470 | `TextSizeMultiLine__17G_IndicatorsClassFRfRfPUcfUi` | the `~`-split measure loop over `TextSize` (§7c) |
| 0x002b32e0 | `ParheliaSetFov__17G_IndicatorsClassFP10I3D_cameraf` | Matrox Parhelia multi-head field-of-view and aspect wrapper — see §7c |
| 0x002b35e0 | `PleaseWait__17G_IndicatorsClassFb` | inlined into `game_done`; the same block also sits in `garage_game_done` (§7c) |

PS2 `RaceSetStartFlag` passes `2 - counter` for the three lights and `7`

**`ParheliaSetFov` — PS2-only, measured over its three callers.** The PS2 calls it from
`CumNaNe`, `SetSniperFov` and `GM_Carcyclopedia::OnCreate`. Their PC twins —
`camera_lock_at_human_pair`, `camera_set_sniper_fov` and `carcyclopedia_ctor` — each call the
`I3D_camera`'s own FOV vtable slot `0x50` directly with the same argument (π/4 = `0x3f490fdb`, the
sniper field of view, π/3 = `0x3f860a92`), so no PC code carries the Parhelia branches.
`structs/camera.md` §7d.

**The last six PS2 methods — one inlined, five PS2-only.** Each verdict rests on the method's
callers, not on its absence from the PC cluster:

* `PleaseWait` — the PC `game_done` holds its body at the same position, after `StopAllSounds`
  and `ItemBlurDone`: the same engine bracket and the same draw, textdb `0x581` at (780, 545),
  height 40, align 4. The PS2 repeats the pass three times. `garage_game_done` holds the same block.
* `LoadFontFake` — the PS2 `LoadFont` calls it after loading slot 0 to copy that slot into slots 2
  and 3. The PC loads all four fonts from file: `hud_load_font` for indices 0..3, and four
  `font%d.gmf` files ship.
* `DrawMessage` — its callers are PS2-only code (`G_CarcCache`, `GM_GarageSelect::OnTick`) or
  `ExecuteMenu`, which shows textdb 656. The PC textdb has neither 656 nor 555, the `G_CarcCache`
  message.
* `AByloTicho(n)` — mutes `isound` and arms the countdown `+0x4e30`, which the PS2 `Tick`
  decrements, restoring the volume at 0. Its callers are the PS2-only script command `PLAY_VIDEO`
  and a mission-name gate in `C_game::Init` that the PC `game_init` does not have, and the
  countdown is missing from `hud_tick`'s lockstep (accumulator, map load, fuel blink, stopwatch).
  The PC object has no room for it either: the map caption `+0x96e8` is its last field —
  `hud + 0x96ec` (`0x00661a1c`) is the separate global `g_font_translation_tables` — and no
  instruction addresses `+0x96ec` or `+0x96f0` off a register.
* `TutorialAddText` — its only caller is the PS2-only script command `TUTORIAL_OUTTEXT`.
* `TextSizeMultiLine` — its callers are the PS2-only memory-card and tutorial screens, the free
  function `_GIconDraw`, `DrawMessage`, and `GM_QuestionBox::OnCreate`, whose PC twin
  `gui_question_box_ctor` measures nothing. Of the six PC callers of `hud_text_measure`, only
  `hud_intro_subtitles_add_intern` splits on `~`, and its PS2 twin does not call
  `TextSizeMultiLine`.

**`hud_push_blood_splat` — PC-only, measured.** The PS2 keeps the splat ring itself: its `Tick`
ages the lifetimes at `+0x4b6c` and decrements the count `+0x4b94`, and `DrawAll` draws the ring
under bit `0x2000000` with `2blood.bmp`, held in the PS2 global `textureCrew`. Nothing on the PS2
increments `+0x4b94`, and `Hit__7C_human` has no call at the position where `human_hit` pushes a
splat.

## 8. Related

- `basic/artifacts/hud_elements.md`: the prose companion: `+0x90a4` bit map, script commands that drive
  HUD elements, in-game-confirmed observations, §8 PC-vs-PS2 differences.
- `structs/console.md`: `C_console`, the separate class whose render / register / unregister trio is
  NOT part of this one; the HUD's OWN console is `hud_console_*` (`ConsoleAddText__17G_IndicatorsClass`).
- `structs/gui_screen.md`: the mouse cursor and the two-font-table layout.
- `structs/camera.md`: `g_hud_flags` and the `ParheliaSetFov` callsites.
- `structs/racing.md` and `basic/artifacts/racing.md`: the race readouts and the start countdown.
- `structs/web_node.md` / `structs/web_path.md`: the `C_web_node` graph the radar road-edge strips are
  built from, owned by `C_web_path` at `*(g_act_mission+0x2c)`.
- `Game-ghidra.exe.functions.md`: the function registry, PS2-twin column filled.

## 9. Open items

1. **The PS2 minimap renderer (§7b)** is not compared against the PC radar renderer in `hud_render_all`.
2. **`S_intro_line.age_s` (§1c).** `hud_tick` adds dt to it, and `hud_render_all` converts it with
   `__ftol` before the draw; what the draw does with the value is not decoded.
