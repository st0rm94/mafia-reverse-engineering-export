# `C_player` — the player actor

**Created** 2026-07-12 14:05 +0200 · **Modified** 2026-11-16 21:19 +0100

The player-controlled person: on-foot, driving and in-car fire control, sniper mode, control lock
and fade, the car-damage bar and the player save record. It derives from `C_human` and adds a
`0x64`-byte band of its own; there is exactly one live instance, the active player
(`*(g_act_mission+0x24)+0xe4`).

| | |
|---|---|
| PC prefix | `player_` |
| PC kind (`+0x10`) | `2`, stamped by `player_ctor` over `C_human`'s 3 (`structs/entity_kind.md` §1) |
| PC vtable | `g_player_vt` @ `0x0063c908` |
| PC base class | `C_human` (`structs/human.md`); `player_ctor` calls `human_ctor` |
| PC instance size | `0xb00` (`operator_new(0xb00)` in the actor factory's kind-2 arm; the last field ends at `0xb00`) |
| PC constructor | `player_ctor` @ `0x004c6b90` |
| PC translation unit | `[0x004c6b50, 0x004cdd70)`: the unit's CRT pair, 37 `player_*` functions, three vtable thunks and `std_vector4_insert_n`; the next unit's guard `crt_static_init_guard_004cdd70` follows |
| PS2 class | `C_player` (tag `8C_player`), 44 methods, all accounted (§3, §6) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_human` base; offsets are PC. `player_ctor` also overrides
five `C_human` fields: max health `+0x604` = 200.0 (twice an NPC's), the four limb energies
`+0x608..+0x614` = 40.0 each (`EnergyHandL` / `EnergyHandR` / `EnergyLegL` / `EnergyLegR`, property
entries 2..5), the `Shooting` property `+0x628` = 1.0 and `+0x680` = 3, the `voice_id` that `human_new_scream`
reads (`structs/human.md`). `+0x644` is current health, which
`player_save_game_load` clamps to the max and pushes to `hud_set_player_health_value`.

```c
typedef struct C_player {                        // 0xb00 bytes; extends C_human
    /* +0xa9c */ uint16_t drive_primary_bits;    // [H] one bit per driving axis owned by its primary key
    /* +0xaa0 */ float    turn_last_input;       // [H] the last non-zero turn input (its sign)
    /* +0xaa4 */ float    turn_idle_s;           // [H] seconds since the turn input was non-zero
    /* +0xaa8 */ float    turn_pending;          // [H] turn amount still to apply
    /* +0xaac */ uint32_t use_probe_elapsed_ms;  // [H] time since the last usable-object probe
    /* +0xab0 */ uint32_t use_probe_period_ms;   // [H] the probe interval: 200 by the ctor, 536 by GameInit
    /* +0xab4 */ I3D_frame *model_base_frame;    // [H] the model's "base" sub-frame
    /* +0xab8 */ float    alpha;                 // [H] current fade level, 1.0 = opaque
    /* +0xabc */ float    alpha_target;          // [H] 0.0 in the eye view, else 1.0
    /* +0xac0 */ std::vector<I3D_object*> alpha_list; // [H] {alloc, begin, end, cap}
    /* +0xad0 */ float    shot_accuracy_spread;  // [H] accumulated crosshair penalty
    /* +0xad4 */ int32_t  aim_hold_ms;           // [H] time the aimed pose is held
    /* +0xad8 */ uint8_t  horn_held;             // [H] the horn key is down (driving)
    /* +0xad9 */ uint8_t  walk_run_toggle;       // [H] 1 = run, 0 = walk; flipped by the toggle key
    /* +0xada */ uint8_t  controls_locked;       // [H] player_lock_controls
    /* +0xadb */ uint8_t  manual_gearbox;        // [M] flipped by the gearbox key; the car's flag is its inverse
    /* +0xadc */ int32_t  car_damage_window_ms;  // [H] 3000 on show, -1 on invalidate
    /* +0xae0 */ C_car   *car_damage_target;     // [H] the car whose damage bar is shown
    /* +0xae4 */ int32_t  side_roll_state;       // [H] double-tap detector state, 0..3
    /* +0xae8 */ uint32_t side_roll_tap_ms;      // [H] game time of the last tap edge
    /* +0xaec */ uint8_t  side_roll_first_dir;   // [H] the side of the first tap
    /* +0xaf0 */ uint32_t last_shown_weapon;     // [H] last weapon slot announced on the HUD
    /* +0xaf4 */ uint8_t  action_menu_had_choices; // [H] the last action menu offered > 1 option
    /* +0xaf5 */ uint8_t  car_aim_active;        // [H] aiming from a car with free look
    /* +0xaf8 */ I3D_sound *horn_on_foot_sound;  // [H] the 7b_zip.wav sound, created on demand
    /* +0xafc */ uint32_t horn_on_foot_ms;       // [H] game time it was last triggered
} C_player;
```

* **`drive_primary_bits`.** Bits 1 steer, 2 accelerate, 4 brake, 8 clutch: set while the axis's
  PRIMARY key binding owns it (§5a). `player_game_init` writes `0xffff`.
* **The turn smoothing (`+0xaa0`..`+0xaa8`).** `player_ai_walk` adds each frame's turn input to
  `turn_pending` (restarting it when the input changes sign), applies at most
  `g_max_aiming_speed_x * frame_s * 40` of it per frame, and drops the rest once the input has
  been zero for 0.12 s.
* **The use probe.** Every `use_probe_period_ms` `player_ai_walk` asks
  `using_object_is_any_object_near` whether a usable object stands 0.8 in front of the player at a
  height of 0.9.
* **`aim_hold_ms`.** A shot arms 3000 ms (`player_add_shot_accuracy_penalty`), aiming from a car
  300000 ms; `player_ai` counts it down and drops the aimed pose (`human_do_aimed(0)`) at 0. The
  countdown stops while `shotgun_reload_pending`, `shooting`, `crouched` or `car_aim_active` is set
  and in world special mode 5 (sniper). On foot, `human_can_aim` failing drops the pose at once.
* **The side roll (`+0xae4`..`+0xaec`).** With the options toggle `g_side_roll_enabled` on,
  `player_ai_walk` runs a 4-state detector over the strafe keys: press, release and press again on
  the same side with each edge within 200 ms (`side_roll_tap_ms`) reaches state 3, which sets
  `anim_slot` `+0x70` to `0xb` (left) or `0xc` (right) and forces the jump input, so `human_do_jump`
  runs in the same frame.
* **The horn key.** Driving, the horn input sets `horn_held`, sounds the car's horn (`car+0x51c`)
  and raises the horn script event; on foot the same input plays `horn_on_foot_sound`, at most once
  per 2000 ms.
* **`manual_gearbox`.** The gearbox key flips it. `player_ai_drive`, `human_update`, `human_use_actor`
  and `human_put_in_vehicle_at_seat` each store its inverse into the car's `+0x5b8`
  (`C_Vehicle+0x548`, `structs/vehicle.md` §1), where 1 is the automatic gearbox:
  `vehicle_tick_drivetrain` runs the automatic shift only while it is set. The HUD shows the same
  byte as the A / M transmission character (`structs/hud.md`); the letter it draws for 1 is Open
  item 1.

## 2. Field evidence

Measured by a census of every `this+N` access in the `player_*` functions and by body reads of
`player_ai_walk`, `player_ai_drive`, `player_ai_car_fire` and `player_ai`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0xa9c | drive_primary_bits | H | `player_ai_drive`, `player_game_init` (`0xffff`), `player_save_game_load` | `player_ai_drive`, `player_save_game_save` | first field of the save record (§5a) |
| +0xaa0 | turn_last_input | H | `player_ai_walk`, `player_game_init`, `player_save_game_load` | `player_ai_walk`, `player_save_game_save` | §1 |
| +0xaa4 | turn_idle_s | H | `player_ai_walk`, `player_game_init`, `player_save_game_load` | `player_ai_walk`, `player_save_game_save` | §1 |
| +0xaa8 | turn_pending | H | `player_ai_walk`, `player_game_init`, `player_save_game_load` | `player_ai_walk`, `player_save_game_save` | §1 |
| +0xaac | use_probe_elapsed_ms | H | `player_ctor`, `player_ai_walk`, `player_game_init` | `player_ai_walk` | §1 |
| +0xab0 | use_probe_period_ms | H | `player_ctor` (200), `player_game_init` (`0x218` = 536) | `player_ai_walk` | §1 |
| +0xab4 | model_base_frame | H | `player_game_init`, `player_change_model` | `player_set_alpha` | accepted only if the frame's type (`+0x110`) is 1 and its FourCC (`+0x1d4`) is `'SNGM'` (`0x4d474e53`) or `'SMMR'` (`0x524d4d53`), the visual types `LS3DF.dll` names `I3D_VISUAL_SINGLEMESH` and `I3D_VISUAL_SINGLEMORPH` |
| +0xab8 | alpha | H | `player_ai`, `player_game_init`, `player_change_model`, `player_fade_reset`, `player_save_game_load` (0; the saved value is not read) | `player_ai`, `player_add_alpha_object`, `player_save_game_save` | `player_ai` moves it toward `alpha_target` by `0.003 × frame ms` (0 to 1 in about 333 ms), clamps it and applies it with `player_set_alpha` |
| +0xabc | alpha_target | H | `player_ai` (0.0 while `camera_is_look_from_eyes`, else 1.0), `player_fade_reset`, `player_save_game_load` (1.0; the saved value is not read) | `player_save_game_save` | `player_ai` steers `alpha` with the value it has just computed |
| +0xac0 | alpha_list | H | `player_ctor`, `player_dtor`, `player_game_init`, `player_change_model` (`end = begin`), `player_alpha_list_add_frame_tree`, `player_alpha_list_remove_frame_tree` | `player_set_alpha`, `player_dtor` | `lea edx,[this+0xac0]` is the argument `player_add_alpha_object` / `player_del_alpha_object` hand their helper; elements are `I3D_object*` |
| +0xad0 | shot_accuracy_spread | H | `player_add_shot_accuracy_penalty` (adds `weapon_record+0x84`), `player_ai` (decay), `player_game_init`, `player_save_game_load` | `player_add_shot_accuracy_penalty`, `player_ai`, `player_save_game_save` | pushed to the HUD as `spread*3.0+1.0` (`hud_set_player_accuracy`, the aim-crosshair scale); above 1.0 it re-seeds to `rand*0.2+0.8`; also mirrored to `C_game+0xd8`, the camera's `aim_spread` (`structs/camera.md` §2), which scales the sniper-scope sway |
| +0xad4 | aim_hold_ms | H | `player_add_shot_accuracy_penalty` (3000), `player_ai_drive` and `player_ai_car_fire` (300000), `player_ai` (countdown, 0), `player_game_init`, `player_save_game_load` | `player_ai`, `player_save_game_save` | §1 |
| +0xad8 | horn_held | H | `player_ai_drive`, `player_do_action_button_in_car` (0 on menu option `0x7d5`), `player_game_init`, `player_save_game_load` | `player_ai_drive`, `player_do_action_button_in_car`, `player_save_game_save` | §1 |
| +0xad9 | walk_run_toggle | H | `player_ai_walk` (the toggle key at controls `+0x6c`), `player_game_init` (1), `player_save_game_load` | `player_ai_walk` (inverted while input slot 8 or 9 is held), `player_save_game_save` | set runs: `human_set_movement_intent_forward` takes 5.0 for a set argument and 1.7 for a clear one; `+0x71c` below 0.5 forces walking |
| +0xada | controls_locked | H | `player_lock_controls`, `player_game_init`, `player_save_game_load` | `player_ai`, `player_collision`, `player_hit`, `player_save_game_save` | last field of the save record |
| +0xadb | manual_gearbox | M | `player_ai_drive` (the key at controls `+0xe4`), `player_game_init` | `player_ai_drive`, `human_update`, `human_use_actor`, `human_put_in_vehicle_at_seat` | each stores `car+0x5b8 = (this == 0)`. That 1 on the car is the automatic gearbox rests on `structs/vehicle.md`; a body read of `vehicle_tick_drivetrain` promotes the row. §1, Open item 1 |
| +0xadc | car_damage_window_ms | H | `player_show_car_damage_bar` (3000), `player_invalidate_car_damage_bar` (−1), `player_ai`, `player_game_init` | `player_ai` | - |
| +0xae0 | car_damage_target | H | `player_show_car_damage_bar`, `player_invalidate_car_damage_bar`, `player_ai`, `player_game_init` | `player_ai`, `player_invalidate_car_damage_bar` | HUD flag bit `0x10000` (`cardamagevisible`); show requires the car's show-damage flag `car+0x2031`, which only the script command `showcardamage` sets (`structs/car.md` §1); invalidate fires only for this car and keeps the bar while the player sits in a car |
| +0xae4 | side_roll_state | H | `player_ai_walk`, `player_game_init` | `player_ai_walk` | §1 |
| +0xae8 | side_roll_tap_ms | H | `player_ai_walk` (the game clock `C_game+0x2b0c`), `player_game_init` | `player_ai_walk` | §1 |
| +0xaec | side_roll_first_dir | H | `player_ai_walk` | `player_ai_walk` | §1 |
| +0xaf0 | last_shown_weapon | H | `player_game_init`, `player_show_weapon_name` | `player_show_weapon_name` | compared with the equipped slot `+0x4a0` so the HUD line fires only on a change |
| +0xaf4 | action_menu_had_choices | H | `player_do_action_button_on_foot` (option count > 1, in the car-entry arms) | `player_ai` | gates cancelling the car break-in, `+0x40c` = 10: while set, the cancel also needs bit 0 of the `g_engine` vtable `+0xe8` query clear |
| +0xaf5 | car_aim_active | H | `player_ai_drive`, `player_ai_car_fire` (1, with `camera_enable_free_look`), `player_do_action_button_in_car` (0), `player_game_init` | `player_ai` | holds `aim_hold_ms` |
| +0xaf8 | horn_on_foot_sound | H | `player_ai_walk` (through `g_i3d_driver` slot `0x4c`, type 4), `player_game_init`, `player_game_done` (released) | `player_ai_walk`, `player_game_done` | §1 |
| +0xafc | horn_on_foot_ms | H | `player_ai_walk`, `player_game_init` | `player_ai_walk` | the 2000 ms gate |

## 3. Method family

Coverage: all **44** PS2 `C_player` methods are accounted — **36** in the tables below (33
CONFIRMED, 2 STRUCTURAL, 1 base stub) and **8** in the table of §6 (7 PS2-only, and
`GoToCar_Cancel`, inlined into several callers). Two further twins are not `C_player` methods: the
file-local helper `cbAlpha` (`player_alpha_list_add_frame_tree`) and the free function
`SetPlayerNaviPoint` (§3d). Five PC functions have no PS2 symbol: the scalar deleting destructor,
`player_alpha_list_remove_frame_tree`, `player_set_hand_scale`, and the CRT pair.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004c6b50 | `crt_static_init_guard_004c6b50` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x004c6b70 | `crt_atexit_register_004c6b70` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x004c6b90 | `player_ctor` | instance | `__ct__8C_playerFv` | 0x001aaed0 | CONFIRMED | `human_ctor`, kind 2, `g_player_vt`, the `C_human` overrides of §1 |
| 0x004c6c20 | `player_scalar_deleting_dtor` | instance | - | - | - | the MSVC scalar deleting destructor |
| 0x004c6c40 | `player_dtor` | instance | `__dt__8C_playerFv` | 0x001aaf70 | CONFIRMED | frees `alpha_list`, chains the `C_human` destructor |

### 3b. Accessors

The class has no accessor of its own: every member does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__8C_playerFUi` | 0x001b3910 | base-stub | slot 15; the shared `ret 4` no-op of 30 vtables (`structs/bottle.md` §9a); the PS2 body is empty |
| 0x004c6c80 | `player_init` | `Init__8C_playerFP9I3D_frame` | 0x001ab030 | CONFIRMED | chains `human_init`, then sets `human_type` (`+0x5fc`) = 1, the player's row of `g_hit_permission_matrix` |
| 0x004c6cb0 | `thunk_human_duplicate_from` | `DuplicateFrom__8C_playerFP7C_actor` | 0x001ab070 | CONFIRMED | slot 21, one dataref: a 5-byte `jmp` to `human_duplicate_from` |
| 0x004c6cc0 | `player_save_data` | `SaveData__8C_playerFP7C_chunk` | 0x001ab080 | CONFIRMED | forwards to `human_save_data`; the class adds nothing to the chunk stream |
| 0x004c6ce0 | `player_load_data` | `LoadData__8C_playerFP7C_chunk` | 0x001ab0a0 | CONFIRMED | forwards to `human_load_data` |
| 0x004c6d00 | `player_save_game_get_size` | `SaveGameGetSize__8C_playerFv` | 0x001ab0c0 | CONFIRMED | the `C_human` size + `0x22` |
| 0x004c6d10 | `player_save_game_save` | `SaveGameSave__8C_playerFPPv` | 0x001ab0e0 | CONFIRMED | chains `human_save_game_save`; tag byte 1, then `drive_primary_bits`, the three turn fields, `alpha`, `alpha_target`, `shot_accuracy_spread`, `aim_hold_ms`, `horn_held`, `walk_run_toggle`, `controls_locked` (`0x22` bytes) |
| 0x004c6da0 | `player_save_game_load` | `SaveGameLoad__8C_playerFPPv` | 0x001ab190 | CONFIRMED | the inverse, except `alpha` and `alpha_target`, which it sets to 0 and 1.0; for the active player re-syncs the HUD |
| 0x004c6ec0 | `thunk_human_death` | `Death__8C_playerFv` | 0x001ab3a0 | CONFIRMED | slot 29, one dataref: a 5-byte `jmp` to `human_death` |
| 0x004c6ed0 | `player_ai_walk` | `AI_walk__8C_playerFUiP17G_ConsoleControls` | 0x001ac740 | CONFIRMED | the on-foot control tick (§1) |
| 0x004c91f0 | `player_ai_drive` | `AI_drive__8C_playerFUiP17G_ConsoleControls` | 0x001af120 | CONFIRMED | the driver-seat tick (seat `+0xac` = 0): axes, horn, gearbox, drive-by aim through `human_do_aimed` and `collision_test_line_horizontal` |
| 0x004ca570 | `player_ai_car_fire` | `AI_car_fire__8C_playerFUiP17G_ConsoleControls` | 0x001afb80 | CONFIRMED | the passenger-seat fire tick |
| 0x004cb1b0 | `player_ai` | `AI__8C_playerFUi` | 0x001b0590 | CONFIRMED | the control tick: picks one of the three sub-ticks by the vehicle link `+0x98` and the seat, runs the fade, the aim hold and the car-damage window; chains `human_ai` |
| 0x004cba50 | `thunk_human_update` | `Update__8C_playerFUi` | 0x001b38e0 | CONFIRMED | slot 14, one dataref: a 5-byte `jmp` to `human_update` |
| 0x004cba60 | `player_game_init` | `GameInit__8C_playerFv` | 0x001b3920 | CONFIRMED | resets the whole band (`drive_primary_bits` = `0xffff`) and re-caches `model_base_frame` |
| 0x004cbb80 | `player_game_done` | `GameDone__8C_playerFv` | 0x001b3a60 | CONFIRMED | releases `horn_on_foot_sound`, chains `human_game_done` |
| 0x004cbbc0 | `player_collision` | `Collision__8C_playerFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001b3ac0 | CONFIRMED | while `controls_locked`, ignored unless `police_board_is_actor_in_pursuit`; else `human_collision` |
| 0x004cbc10 | `player_hit` | `Hit__8C_playerF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001b3b70 | CONFIRMED | the same lock guard, `human_hit`, then the HUD health; leaves sniper mode in world mode 5 |
| 0x004cbcc0 | `player_update_controls` | `UpdateControls__8C_playerFv` | 0x001b3d00 | CONFIRMED | while driving, `vehicle_set_steering_linearity(g_steering_linearity)` |
| 0x004cbd00 | `player_do_action_button_on_foot` | `Do_AB_OwnerNULL__8C_playerFv` | 0x001b3d60 | CONFIRMED | the action button on foot: builds the action menu (textdb `0x7d1..0x7e1`); the only caller of `player_try_use_car` (twice) |
| 0x004cc950 | `player_do_action_button_in_car` | `Do_AB_OwnerCar__8C_playerFv` | 0x001b4a40 | CONFIRMED | the action button in a car (menu ids `0x7d5`, `0x7d6`; `vehicle_set_engine_on`; calls `Start__8C_pumper`, `structs/pumper.md`) |
| 0x004ccc80 | `player_indicator_set_aimed` | `IndicatorSetAimed__8C_playerFv` | 0x001b50a0 | CONFIRMED | sets or clears HUD bits `0x20` / `0x8000000` (the aim crosshair) |
| 0x004ccd20 | `player_add_shot_accuracy_penalty` | `ShootDeAimed__8C_playerFP15S_GameItemTable` | 0x001b51d0 | CONFIRMED | adds the weapon's penalty to `shot_accuracy_spread` (×0.1 in world mode 5), arms `aim_hold_ms` |
| 0x004ccdd0 | `player_set_sniper_mode` | `SetSniperMode__8C_playerFb` | 0x001b52b0 | CONFIRMED | the sniper zoom: FOV constant `0x3df5c28f`, weapon flag `0x4000`; also called from `human_movement` |
| 0x004ccf00 | `player_change_model` | `_intern_ChangeModel__8C_playerFv` | 0x001b5420 | CONFIRMED | the `C_player` half of `human_change_model` (called for the active player only): clears `alpha_list`, re-resolves `model_base_frame` |
| 0x004ccf80 | `player_try_use_car` | `Can_UseCar__8C_playerFP5C_cari` | 0x001b54e0 | CONFIRMED | returns nothing: reports a police-car theft (`police_manager_report_police_car_stealing_sighting` when `car+0x2044`), refuses with a scream when `garage_can_steal_car_model` says no and world bit `0x20` is clear, else picks the enter animation |
| 0x004cd5c0 | `player_show_car_damage_bar` | `SetCarDamage__8C_playerFP5C_car` | 0x001b5cb0 | CONFIRMED | latches `car_damage_target`, arms 3000 ms, raises `g_hud_flags` bit `0x10000` |
| 0x004cd5f0 | `player_invalidate_car_damage_bar` | `InvalidateCarDamage__8C_playerFP5C_car` | 0x001b5d00 | CONFIRMED | drops the bar when the given car is the latched one; `game_invalidate_actor` calls it for kind 4 |
| 0x004cd640 | `player_add_alpha_object` | `AddAlphaObject__8C_playerFP9I3D_frame` | 0x001b5d60 | CONFIRMED | collects the model's `'OBJ_'` objects into `alpha_list` and re-applies `alpha` below 1.0; callers `human_change_weapon_model` (twice), `engine_new_mission` |
| 0x004cd680 | `player_alpha_list_add_frame_tree` | `cbAlpha__Q28C_player22S_hlp$8169C_player_cppFP9I3D_frameRQ23std54vector<P10I3D_object,Q23std24allocator<P10I3D_object>>` | 0x001b5eb0 | CONFIRMED | the recursive collector: every type-1 `'OBJ_'` (`0x5f4a424f`) frame of a subtree into the list |
| 0x004cd700 | `player_del_alpha_object` | `DelAlphaObject__8C_playerFP9I3D_frame` | 0x001b60f0 | STRUCTURAL | removes one frame subtree through `player_alpha_list_remove_frame_tree`; sole caller `human_change_weapon_model` (§6) |
| 0x004cd720 | `player_alpha_list_remove_frame_tree` | - | - | - | its recursive mirror, swap-with-last removal |
| 0x004cd7a0 | `player_set_alpha` | `SetAlpha__8C_playerFf` | 0x001b6100 | CONFIRMED | applies the transparency to `model_base_frame` and every object of `alpha_list` |
| 0x004cd800 | `player_show_weapon_name` | `ShowWeaponName__8C_playerFv` | 0x001b6190 | STRUCTURAL | on a change of the equipped slot prints textdb `3300 + slot` (3300 "Empty hands", 3302 "Knuckle-duster", …) with `hud_console_print`; 13 callsites, all in the three control ticks (§6) |
| 0x004cd890 | `player_lock_controls` | `LockControls__8C_playerFb` | 0x001b61a0 | CONFIRMED | writes `controls_locked`; locking also ends the action in progress |
| 0x004cda20 | `player_set_hand_scale` | - | - | - | the BOXER cheat (`game_cheat_check` case 4): writes the local scale `(1,1,1)` or `(2,2,2)` of `r_hand` / `l_hand` |
| 0x004cdb40 | `player_fade_reset` | `FadeReset__8C_playerFv` | 0x001b6430 | CONFIRMED | `alpha` = `alpha_target` = 1.0, `player_set_alpha(1.0)`; called twice from `camera_begin_mode` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004c6b80 | `player_set_navi_point` | `SetPlayerNaviPoint__FP9I3D_frame` | 0x001aaec0 | CONFIRMED | no `this`: `g_player_navi_frame = frame`. Three callsites: the `SETCOMPASS` script case (the frame, or NULL out of range) and the game / mission init (NULL) |
| 0x004cdb60 | `std_vector4_insert_n` | - | - | - | a shared STL helper with 192 callsites across about 40 modules, placed in this unit; not a `C_player` member |

## 4. Two functions the player code leans on that are not `C_player` methods

| PC VA | PC name | What it is |
|---|---|---|
| 0x00425390 | `act_mission_get_game` | the `C_mission` accessor `return this+0x24`, the `C_game` singleton (`structs/mission.md`) |
| 0x005c8d50 | `human_set_wingman_indicator` | a `C_human` setter: the script commands `wingman_setindicator` / `wingman_delindicator` pin a wingman slot and portrait on any human of kind 2 or `0x1b` (`structs/human.md`) |

## 5. File-scope data

The unit's BSS run is `0x006560d0..0x006560e8`, between the pedestrian unit (`…0x006560cc`) and
the next unit (`0x006560ec…`), established by a census of absolute references in `.text`
attributed to their containing functions:

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_player_drive_steer_right_secondary_latch`, `_steer_left_`, `_clutch_`, `_brake_`, `_accel_secondary_latch` | `0x006560d0`..`0x006560e0` | the secondary-binding latches of §5a, 5 floats | `player_ai_drive` | `player_ai_drive` |
| `g_hud_car_damage_bar_enabled` | `0x006560e4` | set by the script command `cardamagevisible` | `hud_set_car_damage_bar_enabled`, `game_init` (cleared) | `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update`, `player_ai`, `player_ai_drive` |
| `g_player_navi_frame` | `0x006560e8` | the compass target frame | `player_set_navi_point` | - |

`g_player_navi_frame` is write-only: the store in `player_set_navi_point` is the only reference to
its address in the image.

### 5a. The driving-input latches

Every driving axis has a PRIMARY and a SECONDARY key binding (action slots `0x21`..`0x28` and
`0x37` / `0x38`, `structs/input.md` §2a). `player_ai_drive` arbitrates between them per axis:

| Axis | Primary / secondary slot | Bit in `+0xa9c` | Latch |
|---|---|---|---|
| accelerate | `0x21` / `0x22` | `0x2` | `g_player_drive_accel_secondary_latch` |
| brake / reverse | `0x23` / `0x24` | `0x4` | `g_player_drive_brake_secondary_latch` |
| steer left | `0x25` / `0x26` | `0x1` | `g_player_drive_steer_left_secondary_latch` |
| steer right | `0x27` / `0x28` | `0x1` | `g_player_drive_steer_right_secondary_latch` |
| clutch | `0x37` / `0x38` | `0x8` | `g_player_drive_clutch_secondary_latch` |

While the primary value is non-zero the bit is set and the secondary's current value is stored in
the latch. Once the primary is released the secondary stays ignored until it moves more than `0.5`
away from the latched value, which clears the bit. The inverted bits go to the car every frame as
`C_car+0x594..+0x597` (`structs/vehicle.md` §2c), which switch the vehicle setters between a ramped
digital input and a direct analog one. `player_game_init` writes `0xffff`, so at a level start
every secondary axis is ignored until it moves.

## 6. PC vs PS2

| # | Difference | Evidence |
|---|---|---|
| 1 | **The PS2 has autoaim; the PC does not.** `FireMagnetism`, `FireDriveFindTarget` and `FindNearestEnemyToTurn` are PS2-only | `player_ai_drive` aims a drive-by through `human_do_aimed` and `collision_test_line_horizontal` and calls no magnetism; observed in game |
| 2 | **No walk-to-car assist on the PC.** `GoToCar_Init` / `GoToCar_Process` have no PC twin and `GoToCar_Cancel` (a 1-line clear of `+0xc14`) is inlined | observed in game; the third PS2 `Can_UseCar` caller has no PC counterpart |
| 3 | **Both alpha methods ignore their frame parameter on the PC.** `player_add_alpha_object` and `player_del_alpha_object` take an `I3D_frame*` (`ret 4`) and walk `this->+0x68`, the whole model; the PS2 walks the frame it is given | the disassembly of both: `mov ecx,[esi+0x68]` / `lea edx,[esi+0xac0]`, no access to `[esp+8]`; symmetric across both methods |
| 4 | **`DelAlphaObject` clears the whole list on the PS2 and removes one subtree on the PC** | the PS2's 8-byte body is `*(this+0xc48) = 0`; hence STRUCTURAL |
| 5 | **`ShowWeaponName` has a body only on the PC**; the PS2 body is `return;`, hence STRUCTURAL | here the PS2 method count is a lower bound |
| 6 | **The band's 4-byte packing shift is the alpha vector**: PC `std::vector` 16 bytes, PS2 `__vector_pod` 12 bytes | §6a |
| 7 | **Control input.** The three control ticks take a `G_ConsoleControls*` on the PS2 (gamepad); the PC reads the input directly | the signatures |
| 8 | **Steering linearity.** `player_update_controls` passes the tunable `g_steering_linearity`; the PS2 passes `0.5f` | the bodies |
| 9 | **Accuracy mirror.** `player_add_shot_accuracy_penalty` mirrors the spread to world `+0xd8`; `ShootDeAimed` has no such statement | the bodies |
| 10 | **No crosshair-type selector on the PC.** At all three PS2 call positions (`Movement__7C_humanFUi`, twice in `IndicatorSetAimed`) the PC twins raise HUD bit `0x20` and call nothing | the PS2 selector stores a global, `DAT_004f3224`, that the ELF stores 4 times and loads 0 times |
| 11 | **No automatic out-of-ammo switch on the PC.** PS2 `AI` calls `ZkurvenaPrebijeciFunkceAdvanced` when magazine and reserve are empty; it drops the gun and selects the best remaining weapon | on the PC `human_do_reload` is reached only from the reload input, and none of the 11 callers of `inventory_select_by_id` is a player function |
| 12 | **The wingman-indicator setter is inlined on the PS2**: the `Process__9C_programFUi` arm for `WINGMAN_SETINDICATOR` (PS2 opcode `0x7b`) stores the same pair at `C_human +0x734` / `+0x738` behind the same kind 2 / `0x1b` guard | both bodies |
| 13 | **`act_mission_get_game` has no PS2 method**: none of the 68 PS2 `C_mission` symbols reads `+0x20`; PS2 code reads `act_mission+0x20` inline | the PS2 symbol table and bodies |

Several PS2 methods are console-input features the PC never had; both feature differences follow
from the controls differing between builds (gamepad against mouse and keyboard).

**PS2 methods with no PC function:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001ab3b0 | `FireMagnetism__8C_playerFP8S_vectorP8S_vector` | autoaim, called only from `AI_walk` / `AI_drive` (row 1) |
| 0x001aba00 | `FireDriveFindTarget__8C_playerFP8S_vector` | drive-by target acquisition, called only from `AI_drive` (row 1) |
| 0x001abdb0 | `FindNearestEnemyToTurn__8C_playerFb` | auto-turn to the nearest enemy, called only from `AI_walk` (row 1) |
| 0x001ac3c0 | `ZkurvenaPrebijeciFunkceAdvanced__8C_playerFv` | the out-of-ammo weapon switch (row 11) |
| 0x001b1b10 | `GoToCar_Init__8C_playerFv` | the walk-to-car assist; `player_ai` never calls it (row 2) |
| 0x001b3270 | `GoToCar_Cancel__8C_playerFv` | inlined into each PC caller as the clear of `+0xc14` (row 2) |
| 0x001b3290 | `GoToCar_Process__8C_playerFUiP17G_ConsoleControls` | the per-frame body of the assist (row 2) |
| 0x001b5010 | `NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji__8C_playerFv` | the crosshair-type selector (row 10) |

`GoToCar_Cancel` is realised inline in several PC callers with no single host, so it sits in this
table and not in §3.

### 6a. The `C_player` band has a 4-byte packing shift

The PC↔PS2 offset delta of the band is not constant. Nine field pairs proven by body diff split in
two:

| PC offset | PS2 offset | Delta | Proven by |
|---|---|---|---|
| +0xab4 | +0xc38 | **0x184** | `player_change_model` |
| +0xab8 | +0xc3c | **0x184** | `player_change_model` |
| +0xabc | +0xc40 | **0x184** | `player_ai` / `player_fade_reset` |
| +0xac8 | +0xc48 | **0x180** ¹ | `player_change_model` |
| +0xad0 | +0xc50 | **0x180** | `player_add_shot_accuracy_penalty` |
| +0xad4 | +0xc54 | **0x180** | `player_add_shot_accuracy_penalty` |
| +0xad8 | +0xc58 | **0x180** | `player_do_action_button_in_car` |
| +0xadc | +0xc5c | **0x180** | `player_show_car_damage_bar` / `_invalidate_car_damage_bar` |
| +0xae0 | +0xc60 | **0x180** | `player_show_car_damage_bar` / `_invalidate_car_damage_bar` |

¹ The same list, not the same field: PC `+0xac8` is the vector's end pointer and PS2 `+0xc48` its
size. `player_change_model`'s `+0xac8 = +0xac4` is `end = begin`, the PS2's `+0xc48 = 0`.

The shift is the alpha-object list: PC `+0xac0` is an MSVC `std::vector` of 16 bytes
(`{alloc, begin, end, cap}`), PS2 `+0xc44` a 12-byte `__vector_pod`, so every later PC offset is 4
bytes higher and the delta falls from `0x184` to `0x180`. The save record (PS2 `+0xc20..+0xc5a` ↔
PC `+0xa9c..+0xada`, 11 fields, same order and types, same tag 1) straddles the shift: **use
`0x180` above `+0xac8` and `0x184` below `+0xab8`.** Corollary pairs: PS2 `+0x224` ↔ PC `+0x98`
(vehicle link), `+0x228` ↔ `+0x9c`, `+0x238` ↔ `+0xac` (seat); `player_game_done`'s sub-object
`+0xaf8` ↔ PS2 `+0xc78`; the controls lock `+0xada` ↔ `+0xc5a`.

### 6b. Build-invariant details

- The `'SNGM'` / `'SMMR'` FourCC constants are byte-identical on the PS2.
- `+0xad8` is touched twice by the in-car action-button handler on the PS2 too.
- The alpha list's element type `I3D_object*` is the template argument of the PS2 collector
  `cbAlpha`, the twin of `player_alpha_list_add_frame_tree`.
- `player_set_navi_point` is mangled `__FP9I3D_frame` (no class tag) and is the first symbol of
  `C_player.cpp` on both builds; callsite census 3 ↔ 3. `g_player_navi_frame` is PS2 `navi_frame`,
  write-only on both builds.
- The fade target is PS2 `+0xc40`: `AI__8C_playerFUi` sets it from `LookFromEyes__8G_CameraFv`
  and steps `+0xc3c` toward it, comparing against the field itself.
- `player_try_use_car` reads the police block at `mission+0x2da0`, PS2 `+0x11e0`; it is distinct
  from the free function `MuzuSeTamNasomrovat__FP7C_humanP5C_cari`.
- Slots 14 / 21 / 29: both builds override them with a bare jump to the `C_human` base (PS2
  `j <base>; nop`).

## 7. Related

- `structs/human.md` — the base class; `structs/actor.md` — the root.
- `structs/hud.md` — `g_hud_flags` and the accuracy, health and console sinks.
- `structs/input.md` — the action slots of §5a.
- `basic/artifacts/player-interactions.md` — the action button and its menu.
- `Game-ghidra.exe.functions.md` — the function registry.

## 8. Open items

1. **The gearbox letter on screen.** Which letter the HUD draws for `car+0x5b8` = 1 (the automatic
   gearbox, set from the inverse of `manual_gearbox`) is not measured; an in-game test of the key
   settles it.
2. **The on-foot horn sound.** `horn_on_foot_sound` is bound to the sound cache entry matching
   `7b_zip.wav`; what the player hears when the horn key is pressed on foot is not checked in game.
