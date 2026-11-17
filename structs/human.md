# `C_human` — the concrete person body (animation / movement / weapons)

**Created** 2026-07-12 10:30 +0200 · **Modified** 2026-11-09 10:34 +0100

The concrete human layer, derived from `C_actor`: locomotion, the animation and pose system,
weapons, inventory and combat, hit and death reactions, car entry and exit, and the `C_human`
savegame block. `C_entity` (the scriptable-NPC AI layer) and `C_player` derive from it, so most
`C_human` objects in a running game are one of those two.

| | |
|---|---|
| PC prefix | `human_` |
| PC kind (`+0x10`) | `3`, stamped by `human_ctor` over the `C_actor` base's `1` |
| PC vtable | `g_human_vt` @ `0x0063c468`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0xa9c` (`operator_new(0xa9c)` before `human_ctor` in `racing_spawn_winners_scene`, the one site that builds a plain `C_human`) |
| PC constructor | `human_ctor` @ `0x0048f430` |
| PC translation unit | `[0x0048b5b0, 0x004ae800)`: 159 function starts, all named (§4); the next unit's guard `crt_static_init_guard_004ae800` follows |
| PS2 class | `C_human` (tag `7C_human`), 146 methods, all accounted for in §3 and §7 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base (`0x00..0x6f`); offsets are PC. Five
sub-objects are embedded whole and documented in their own files.

```c
typedef struct C_human {                    // 0xa9c bytes; PC offsets
    /* +0x000 */ C_actor                      base;          // [H] the C_actor band, 0x00..0x6f (structs/actor.md)
    /* +0x070 */ uint32_t                     anim_slot;     // [H] the clip the human is IN
    /* +0x074 */ uint32_t                     cur_anim_id;   // [H] the clip requested this frame; id bands §2c
    /* +0x078 */ uint32_t                     act_state;     // [H] the clip set: 0 none, 1 and 2 the locomotion bands, 3 shock
    /* +0x07c */ std_vector                   anim_overrides; // [H] {alloc +0x7c, begin +0x80, end +0x84, cap +0x88} of {char *file; int id}
    /* +0x08c */ float                        clip_track_scale; // [H] scale of a clip's position track
    /* +0x090 */ float                        climb_reach_scale; // [H] the climb clip's other track scale
    /* +0x094 */ uint32_t                     state_flags;   // [H] bit 0 the locomotion phase, bit 1 the script hit event
    /* +0x098 */ C_actor *                    vehicle_link;  // [H] the vehicle the human is in
    /* +0x09c */ C_actor *                    transition_vehicle; // [H] the vehicle of an entry or exit in progress
    /* +0x0a0 */ C_car *                      entered_car;   // [H] the car entered, for door clip events
    /* +0x0a4 */ C_car *                      brake_held_car; // [H] the car this entity holds braked
    /* +0x0a8 */ C_car *                      locked_car;    // [H] the car this entity holds locked
    /* +0x0ac */ int32_t                      seat_index;    // [H] seat in the vehicle; -1 none, leaving, or the cargo bed
    /* +0x0b0 */ C_car *                      engine_scream_car; // [H] the car that has had its engine-start scream
    /* +0x0b4 */ int32_t                      queued_seat_action; // [H] a seat action waiting until seated
    /* +0x0b8 */ int32_t                      queued_seat_timer_ms; // [H] the delay of that action; -1 none
    /* +0x0bc */ C_animations_machine_de_luxe anim_machine;  // [H] EMBEDDED, 0x128 bytes, +0xbc..+0x1e3
    /* +0x1e4 */ uint8_t                      crouched;      // [H] the crouched stance
    /* +0x1e5 */ uint8_t                      aimed;         // [H] the aimed stance
    /* +0x1e6 */ uint8_t                      anim_side;     // [H] non-zero gives the 'L' clip variant, zero 'R'
    /* +0x1e8 */ uint32_t                     weapon_class;  // [H] the in-hand item's class, §2b
    /* +0x1ec */ uint8_t                      shooting;      // [H] the attack is held: fire for a firearm, the press for melee
    /* +0x1ed */ uint8_t                      shot_any_angle; // [H] the shot's 90 degree limit is waived
    /* +0x1f0 */ void *                       fire_action;   // [H] the fire-action table in force
    /* +0x1f4 */ uint32_t                     fire_action_step; // [H] index into `fire_action`, stride 8
    /* +0x1f8 */ uint32_t                     fire_action_step_ms; // [H] ms into the current fire-action step
    /* +0x1fc */ uint8_t                      reload_active; // [H] a reload is running
    /* +0x1fd */ uint8_t                      reload_requested; // [H] the reload was asked for, not forced by an empty magazine
    /* +0x1fe */ uint8_t                      shotgun_reload_pending; // [H] the pump shotgun's shell-by-shell reload
    /* +0x200 */ S_vector                     shot_target;   // [H] world-space point the shot or throw aims at
    /* +0x20c */ I3D_frame *                  bolt_frame;    // [H] the weapon model's frame `zaver`, 0 with no model
    /* +0x210 */ I3D_frame *                  muzzle_frame;  // [H] the weapon model's frame `muzzle`, 0 with no model
    /* +0x214 */ int32_t                      muzzle_particle; // [H] the particle id of the shot, -1 none
    /* +0x218 */ uint32_t                     throw_charge_start_ms; // [H] mission clock when the throw wind-up began
    /* +0x21c */ uint8_t                      throw_charging; // [H] a grenade or Molotov throw is being wound up
    /* +0x220 */ uint32_t                     melee_button_ms; // [H] mission clock of the last melee press or release
    /* +0x224 */ uint8_t                      melee_windup_latched; // [H] the wind-up clip has started in this hold
    /* +0x228 */ int32_t                      melee_combo_step; // [H] the combo step 0..3; -1 a charged stroke
    /* +0x22c */ uint8_t                      guard_clip_active; // [H] the melee guard clip is up
    /* +0x22d */ uint8_t                      sniper_mode;   // [H] the sniper scope is up
    /* +0x22e */ uint8_t                      ground_unsettled; // [H] the ground under the human may change
    /* +0x22f */ uint8_t                      lethal_water_latched; // [H] the first contact with material `0x1f` has run
    /* +0x230 */ uint8_t                      weapon_model_dirty; // [H] the visible weapon must be rebuilt
    /* +0x234 */ int32_t                      melee_stroke_ms; // [H] countdown of the running melee stroke; -1 none
    /* +0x238 */ float                        aim_pitch;     // [H] the on-foot aim pitch, 0.1..0.85; default 0.5
    /* +0x23c */ S_human_clip_step            clip_steps[5]; // [H] the staged clip steps, §1a
    /* +0x2a0 */ uint32_t                     clip_step_index; // [H] the current step of `clip_steps`
    /* +0x2a4 */ void *                       pose_back1;    // [H] pose object bound to the bone `frame_back1`
    /* +0x2a8 */ void *                       pose_back2;    // [H] pose object bound to the bone `back2`
    /* +0x2ac */ void *                       pose_neck;     // [H] pose object bound to the bone `neck`
    /* +0x2b0 */ uint8_t                      pose_moving;   // [H] a pose object is still turning
    /* +0x2b4 */ S_vector                     pose_target_back1; // [H] target of `pose_back1`, in the actor's frame
    /* +0x2c0 */ S_vector                     pose_target_back2; // [H] target of `pose_back2`
    /* +0x2cc */ S_vector                     pose_target_neck; // [H] target of `pose_neck`
    /* +0x2d8 */ I3D_frame *                  look_target_frame; // [H] the frame the head pose follows
    /* +0x2dc */ C_ShotSkeleton               shot_skeleton; // [H] EMBEDDED, 0x130 bytes, +0x2dc..+0x40b
    /* +0x40c */ uint32_t                     movement_mode; // [H] the locomotion mode: 1 idle, 2 walking, 3 falling, 4 position-track clip, 5 / 7 / 10 timed, 6 clip steps, 8 dead clip ended, 9 seated, 0xb jump
    /* +0x410 */ uint32_t                     vehicle_transition; // [H] the vehicle entry or exit running: `human_update` acts when the clip ends
    /* +0x414 */ void *                       last_contact;  // [H] the contact record of the last move test
    /* +0x418 */ float                        fall_speed;    // [H] downward speed in m/s
    /* +0x41c */ uint8_t                      fall_started_in_jump; // [H] the fall began in the jump mode
    /* +0x420 */ float                        velocity[3];   // [H] the per-axis velocity channels
    /* +0x42c */ float                        turn_rate;     // [H] `human_update` rotates `+0x30` about y by `dt * x * 0.001`
    /* +0x430 */ float                        coast_velocity[3]; // [H] the last non-zero velocity
    /* +0x43c */ float                        motion_ramp;   // [H] the motion ramp 0..1, multiplied into `velocity`
    /* +0x440 */ float                        ground_normal[3]; // [H] the ground normal under the feet
    /* +0x44c */ int32_t                      mode_timer_ms; // [H] countdown of the timed modes 5, 7 and 10; -1 none
    /* +0x450 */ int32_t                      corpse_finish_ms; // [H] ms until `actor_set_act_state(2)` once dead
    /* +0x454 */ uint8_t                      corpse_blood_pending; // [H] a blood decal is due after that countdown
    /* +0x458 */ int32_t                      agony_clip_band; // [H] the agony clip band: 0, 4, 8 or 0xc; -1 none
    /* +0x45c */ int32_t                      agony_repeats_left; // [H] agony repeats left; at 0 the last clip plays
    /* +0x460 */ uint32_t                     last_shot_ms;  // [H] mission clock of the last shot or throw
    /* +0x464 */ uint8_t                      track_sweep_active; // [H] a position-track clip is being swept
    /* +0x468 */ float                        track_sweep_pos[3]; // [H] the last swept position
    /* +0x474 */ int32_t                      break_in_total_ms; // [H] the break-in's total time
    /* +0x478 */ C_car *                      break_in_car;  // [H] the car of break-in mode 10
    /* +0x47c */ uint32_t                     shadow_frame_handle; // [H] handle from game_add_frame_for_shadow
    /* +0x480 */ G_Inventory                  inventory;     // [H] EMBEDDED, 0xe4 bytes, +0x480..+0x563
    /* +0x564 */ I3D_frame *                  weapon_model_frame; // [H] the visible weapon's model frame
    /* +0x568 */ I3D_frame *                  left_hand_item_frame; // [H] the left-hand item's model frame
    /* +0x56c */ I3D_frame *                  right_hand_bone; // [H] frame `gun1`, or `r_hand`
    /* +0x570 */ I3D_frame *                  left_hand_bone; // [H] frame `gun2`, or `l_hand`
    /* +0x574 */ I3D_frame *                  left_hand_mount; // [H] driver type 7 frame under `+0x570`
    /* +0x578 */ I3D_frame *                  right_hand_mount; // [H] driver type 7 frame under `+0x56c`
    /* +0x57c */ I3D_frame *                  base_bone_frame; // [H] the bone `frame_base`
    /* +0x580 */ uint8_t                      weapon_position_dirty; // [H] the weapon model must be re-seated
    /* +0x581 */ uint8_t                      reload_effects_armed; // [H] the reload's weapon effects may run
    /* +0x584 */ int32_t                      railway_car_index; // [H] 0 the vehicle's frame, n the car n-1
    /* +0x588 */ I3D_frame *                  eye_target_frame; // [H] the frame `targetE`, the eye aim target
    /* +0x58c */ I3D_frame *                  neck_target_frame; // [H] the frame `targetN`, the neck aim target
    /* +0x590 */ int32_t                      idle_variation_timer_ms; // [H] armed to 5000 + rand while the idle clip plays
    /* +0x594 */ int32_t                      clip_hold_ms;  // [H] while >= 0 a lower clip id cannot replace the current one
    /* +0x598 */ uint8_t                      idle_anims_enabled; // [H] idle variations may play
    /* +0x59c */ int32_t                      melee_sweep_count; // [H] live entries of the two sweep arrays
    /* +0x5a0 */ I3D_frame *                  melee_sweep_frames[4]; // [H] the frames the melee sweep traces
    /* +0x5b0 */ S_vector                     melee_sweep_prev_pos[4]; // [H] their previous world positions
    /* +0x5e0 */ uint8_t                      melee_sweep_active; // [H] `human_ai` runs the melee sweep
    /* +0x5e4 */ int32_t                      seated_state;  // [H] 0 none, 1 steering clip at seat 0, 2 the gun at the window
    /* +0x5e8 */ int32_t                      seated_step_ms; // [H] ms until the next window step: clip time - 250
    /* +0x5ec */ uint32_t                     seated_anim_stage; // [H] the anim stage id for frame slot `+0x78`
    /* +0x5f0 */ float                        seated_clip_time; // [H] the clip time for that stage
    /* +0x5f4 */ float                        aim_ratio;     // [H] the window aim ratio 0..1; default 0.5
    /* +0x5f8 */ char *                       original_model_name; // [H] base name of the model before the first swap; 0 until then
    /* +0x5fc */ uint32_t                     human_type;    // [H] the type / faction as a single-bit mask
    /* +0x600 */ float                        prop_max[15];  // [H] the property maxima, §2a
    /* +0x63c */ float                        morale_base;   // [H] 0.5; copied to `morale` by the block copy
    /* +0x640 */ float                        prop_cur[15];  // [H] the live property values, §2a
    /* +0x67c */ float                        morale;        // [H] the live morale, block-copied from `+0x63c`
    /* +0x680 */ int32_t                      voice_id;      // [H] the column of `g_game_voice_table`
    /* +0x684 */ uint8_t                      silent;        // [H] the human makes no scream
    /* +0x688 */ void **                      dyn_coll_objects; // [H] tDynamicCollObject* array, 0x48 bytes each
    /* +0x68c */ I3D_frame **                 dyn_coll_frames; // [H] the frames they track
    /* +0x690 */ int32_t                      dyn_coll_count; // [H] element count of both arrays; 2
    /* +0x694 */ uint8_t                      dyn_colls_active; // [H] registered with the collision world
    /* +0x698 */ std_vector                   coll_ignore;   // [H] {alloc +0x698, begin +0x69c, end +0x6a0}
    /* +0x6a8 */ std_vector                   coll_owner_scratch; // [H] {alloc +0x6a8, begin +0x6ac, end +0x6b0}
    /* +0x6b8 */ I3D_sound *                  footstep_sound; // [H] the footstep sound object
    /* +0x6bc */ uint32_t                     last_footstep_ms; // [H] `frame_time_ms` of the last footstep
    /* +0x6c0 */ I3D_sound *                  shot_sound;    // [H] the weapon's shot sound
    /* +0x6c4 */ I3D_sound *                  reload_sound;  // [H] the weapon's reload sound
    /* +0x6c8 */ I3D_sound *                  gun_action_sound; // [H] the pump or bolt sound
    /* +0x6cc */ I3D_sound *                  speech_sound;  // [H] the scream sound object
    /* +0x6d0 */ int32_t                      scream_priority; // [H] priority of the running scream; -1 none
    /* +0x6d4 */ uint32_t                     ground_material; // [H] the material byte of the ground under the human
    /* +0x6d8 */ int32_t                      wingman_slot;  // [H] the HUD wingman slot 0..3; -1 none
    /* +0x6dc */ uint32_t                     wingman_portrait; // [H] the portrait id 0..7 in that slot
    /* +0x6e0 */ I3D_frame *                  fire_target_frame; // [H] the frame the human fires at
    /* +0x6e4 */ int32_t                      blood_drips_left; // [H] blood drips left to place
    /* +0x6e8 */ int32_t                      blood_drip_timer_ms; // [H] time to the next drip
    /* +0x6ec */ int16_t                      print_total;   // [H] bloody-footprint alpha denominator
    /* +0x6ee */ int16_t                      prints_left;   // [H] footprints still to place; armed to 10
    /* +0x6f0 */ uint8_t                      print_texture; // [H] decal texture index; 6 (2bfootprint.bmp)
    /* +0x6f4 */ void *                       spawn_record;  // [H] {uint8_t release_frame; pedestrian element *}: `new(8)` by a spawner
    /* +0x6f8 */ C_actor *                    carried_box;   // [H] the carried box
    /* +0x6fc */ int32_t                      carried_box_group; // [H] the carried box's group id
    /* +0x700 */ C_actor *                    carried_body;  // [H] the carried dead body
    /* +0x704 */ C_actor *                    carried_body_actor; // [H] the actor of the carried body, kind 2 or `0x1b`
    /* +0x708 */ int32_t                      body_drop_clip_ms; // [H] countdown of the put-down clip
    /* +0x70c */ uint16_t                     progress_sample_ms; // [H] ms since the progress sample
    /* +0x710 */ float                        progress_sample_pos[3]; // [H] the position at that sample
    /* +0x71c */ float                        progress_ratio; // [H] moved over expected distance, smoothed; 1.0 when no motion was expected
    /* +0x720 */ uint8_t                      no_hit_anim;   // [H] suppress the flinch clip
    /* +0x724 */ ai_loc                       nav_locality;  // [H] EMBEDDED, 0x34 bytes, +0x724..+0x757
    /* +0x758 */ ai_blob_log                  scent_trail;   // [H] EMBEDDED, 0x244 bytes, +0x758..+0x99b
    /* +0x99c */ int32_t                      break_in_sound; // [H] the sound handle of the car break-in
    /* +0x9a0 */ uint16_t                     scream_area_mask; // [H] the scream areas this human is in
    /* +0x9a4 */ uint32_t                     last_reaction_tick; // [H] `GetTickCount` of the last hit reaction
    /* +0x9a8 */ void *                       last_reaction_sector; // [H] `mission` `camera_sector` at that reaction
    /* +0x9ac */ float                        shots_fired;   // [H] shots in the window
    /* +0x9b0 */ float                        hits_landed;   // [H] hits landed in the window
    /* +0x9b4 */ float                        shots_fired_mark; // [H] the snapshot `human_ai` rebases `+0x9ac` by
    /* +0x9b8 */ float                        hits_landed_mark; // [H] the same for `+0x9b0`
    /* +0x9bc */ float                        hit_ratio;     // [H] `+0x9b0 / +0x9ac`
    /* +0x9c0 */ int32_t                      hit_ratio_window_ms; // [H] countdown of that window; 5000
    /* +0x9c4 */ S_using_object               corpse_use_search; // [H] the corpse's record: action 2012 "Search dead body"
    /* +0x9fc */ S_using_object               corpse_use_move; // [H] the corpse's record: action 2013 "Move dead body"
    /* +0xa34 */ uint8_t                      entity_use_enabled; // [H] the entity's using-object `+0x1230` may be registered
    /* +0xa35 */ uint8_t                      exit_colls_pending; // [H] the capsules wait for the next clip change
    /* +0xa36 */ uint8_t                      death_pos_done; // [H] the corpse's settle pose is computed
    /* +0xa38 */ float                        death_pos_start[3]; // [H] the settle's start position
    /* +0xa44 */ float                        death_pos_delta[3]; // [H] the settle's offset
    /* +0xa50 */ int32_t                      death_pos_elapsed_ms; // [H] ms into the settle
    /* +0xa54 */ int32_t                      death_pos_duration_ms; // [H] the settle's length; -1 none
    /* +0xa58 */ int32_t                      freeze_countdown_ms; // [H] while > 0 the update returns early
    /* +0xa5c */ I3D_frame *                  weapon_effect_frame; // [H] the weapon part the effect moves
    /* +0xa60 */ int32_t                      weapon_effect_id; // [H] the running effect, a clip event id; 0 none
    /* +0xa64 */ float                        weapon_effect_progress; // [H] the effect's progress
    /* +0xa68 */ float                        weapon_effect_rate; // [H] progress per ms
    /* +0xa6c */ int32_t                      pump_clip_delay_ms; // [H] countdown to the pump overlay; -1 none
    /* +0xa70 */ I3D_frame *                  hand_magazine_frame; // [H] a copy of the magazine in the left hand
    /* +0xa74 */ C_actor *                    last_attacker; // [H] the last attacker
    /* +0xa78 */ uint8_t                      speed_limiter_on; // [H] the player's speed limiter
    /* +0xa79 */ uint8_t                      scores_in_freeride; // [H] killing it in freeride scores
    /* +0xa7a */ uint8_t                      temp_ejected_driver; // [H] the driver created to be thrown out of a traffic car
    /* +0xa7c */ float                        pose_look_point[3]; // [H] the point looked at
    /* +0xa88 */ uint32_t                     pose_request_flags; // [H] recorder pose request: bit 0 changed, bit 1 aimed, bit 2 null
    /* +0xa8c */ float                        track_blend_weight; // [H] playback crossfade weight, moved by `dt * 0.004`
    /* +0xa90 */ float                        track_clip_time; // [H] playback clip time
    /* +0xa94 */ float                        track_clip_dt; // [H] clock minus key time, consumed once as the frame dt
    /* +0xa98 */ uint8_t                      track_blend_running; // [H] the crossfade is running
    /* +0xa99 */ uint8_t                      track_clip_slot; // [H] which of the two clip slots plays
    /* +0xa9a */ uint8_t                      track_pos_applies; // [H] the clip's position track applies
} C_human;
```

**The arrays.**

* **`prop_max[15]` / `prop_cur[15]`** are the developers' property vector: fifteen named
  properties, a maximum and a live value each (§2a). The count is the length of the shipped name
  table, and the stats record the loaders read is exactly 15 floats (`0x3c` bytes, `0x40` with a
  scale). Index 1 (`Energy`) is the health, indices 2..5 the four limb energies, index 7 (`Speed`)
  the live locomotion multiplier.
* **`velocity[3]`** is one value per world axis, written by the movement-intent setters as
  speed × the movement direction (`C_actor+0x30..+0x38`) × `prop_cur[7]`. The strafe setters use the
  direction's perpendicular, the diagonal setters the normalised sum.
* **`dyn_coll_objects` / `dyn_coll_frames`** point to two parallel heap arrays of
  erases both and the count is 0. `human_do_crouched` shrinks entry 0.

### 1a. `S_human_clip_step` — one staged clip step, 0x14 bytes

```c
typedef struct S_human_clip_step {            // 0x14 bytes
    /* +0x00 */ uint32_t code;                // [H] 0 wait, 1 play the clip, 2 agony, 3 end
    /* +0x04 */ uint32_t clip;                // [H] the clip id
    /* +0x08 */ int32_t  wait_ms;             // [H] the wait; with flags bit 1 the clip time is added
    /* +0x0c */ uint8_t  unk_0c;              // [U] stored 0 by every writer; loaded as an argument of the evaluate call, meaning not decoded (class census)
    /* +0x10 */ uint32_t flags;               // [H] bit 0 an argument of the evaluate call, bit 1 turns the step into a wait
} S_human_clip_step;
```

`human_movement` mode 6 runs the steps by `+0x2a0`. Code 0 counts `wait_ms` down by dt, then goes to
the next step. Code 1 plays the clip; with flags bit 1 it adds the clip time to `wait_ms` and turns
the step into code 0. Code 2 enters mode 7 with clip `+0x458 + 0x95`. Code 3 ends: alive to mode 1
with clip 1, dead to mode 8. `human_hit` stores a wait of 0 or -250, so such a step ends 250 ms before
its clip does. The clips cover the knock-down band `severity*3 + 0x3d` and also `0x6a..0x72`,
`0xc4..0xe1`, `0xec..0xf0`, the death ids and the agony band.

## 2. Field evidence

Measured by a census of every `this+N` access in the `human_*` and `player_*` functions, joined with
the functions the earlier body reads named for each field (the decomp text cannot see an access
through an alias). Cells with more than eight functions are listed in the notes below the table.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `human_ctor` (`actor_ctor`, vtable `g_human_vt`, kind `+0x10` = 3) | the `C_actor` methods of `structs/actor.md` | the inherited band; `+0x30..+0x38` is the movement direction the velocity channels scale (§1) |
| +0x070 | anim_slot | H | 24 functions ¹ | 16 functions ² | the shock continuations accept only three ids here before advancing (§5); `entity_process_command` `STOPANIM` stores 0, and `entity_action_tick` cases `0x12` and `0x28` test it against 1 |
| +0x074 | cur_anim_id | H | 55 functions ³ | 9 functions ⁴ | every anim setter writes it; `human_animation_evaluate_and_dispatch` consumes it; `entity_action_tick` case `0x25` stores the side-jump ids `0x142`..`0x149` to dodge a car |
| +0x078 | act_state | H | `entity_clear_act_state`, `entity_force_ai`, `entity_force_script`, `entity_set_act_state_shock`, `human_activate`, `human_game_init`, `human_save_game_load` | `entity_force_ai`, `entity_force_script`, `human_animation_evaluate_and_dispatch`, `human_save_game_save` | `human_animation_evaluate_and_dispatch` maps the requested clip: 1 to `0x15..0x21`, 2 to `0x22..0x2e`, 3 to the shock get-up and stumble ids. `entity_force_script` calls `human_do_shock_up` at 3 and refuses at 4 |
| +0x07c | anim_overrides | H | `human_activate`, `human_change_anim`, `human_ctor`, `human_deactivate`, `human_dtor`, `human_game_done`, `human_game_init` | 11 functions ³⁸ | searched backward for a clip id before `g_anim_info_table`, so an entry overrides the table's file name; `human_change_anim` removes and appends entries, and every exit from a carry clears them |
| +0x08c | clip_track_scale | H | `human_do_climb`, `human_do_climb_in_car_lr` | `human_animation_evaluate_and_dispatch` | the ledge height for clips `0x68`, `0x69`; the seat-to-seat distance over the track's for clips `0x130`, `0x131`, else 1.0 |
| +0x090 | climb_reach_scale | H | `human_do_climb` | `human_animation_evaluate_and_dispatch` | `human_do_climb`: the distance to the wall * 2.5, or 1.0 under 0.5; passed for clips `0x68`, `0x69` |
| +0x094 | state_flags | H | `entity_game_init`, `human_activate`, `human_animation_evaluate_and_dispatch`, `human_game_init` | `entity_game_init`, `human_animation_evaluate_and_dispatch`, `human_hit` | bit 0 flips on each locomotion clip `5..0x14`, and when set the clip starts at 0.5; bit 1 on kind `0x1b` with the player as attacker raises event `0xea64` in `human_hit`; `entity_game_init` sets bit 1 when the program defines that event (`hit_from_player`) |
| +0x098 | vehicle_link | H | 15 functions ⁵ | 62 functions ⁶ | a car when its kind `+0x10` is 4 |
| +0x09c | transition_vehicle | H | 16 functions ⁷ | 16 functions ⁸ | `human_intern_use_car(1)` and `human_intern_use_railway(1)` move it to `+0x98` and zero it; `human_update` zeroes it when `+0x410` finishes; the crosshair logic kind-checks both `+0x98` and `+0x9c` |
| +0x0a0 | entered_car | H | `human_activate`, `human_game_init`, `human_put_in_vehicle_at_seat`, `human_use_actor` | `human_notify_callback` | third fallback after `+0x9c` and `+0x98` for the door that clip events `0x16` and `0x18` move; no exit arm clears it |
| +0x0a4 | brake_held_car | H | `human_activate`, `human_death`, `human_game_init`, `program_process_command` | `human_death` | stored by script `car_lock` on a kind `0x1b` caller; `human_death` lowers the car's `brake_hold_count` and zeroes it |
| +0x0a8 | locked_car | H | `human_activate`, `human_death`, `human_game_init`, `program_process_command` | `human_death` | stored by script `car_lock_all` on a kind `0x1b` caller; `human_death` calls `vehicle_lock(0)` on it and zeroes it |
| +0x0ac | seat_index | H | `human_death_pose_align_to_ground`, `human_death_pose_push_out_of_wall`, `human_death_pose_settle_to_ground`, `human_do_climb_in_car_lr`, `human_intern_use_car_truck`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_use_actor` | 45 functions ⁹ | `human_intern_use_car_truck` writes -1 for the cargo bed, so -1 with `+0x98` set is on the bed |
| +0x0b0 | engine_scream_car | H | `human_activate`, `human_game_init`, `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update`, `player_ai_drive` | `player_ai_drive` | `player_ai_drive`: one scream per car when the engine starts at speed 0; zeroed on taking a seat |
| +0x0b4 | queued_seat_action | H | `human_activate`, `human_ai`, `human_game_init`, `human_intern_use_car`, `human_update`, `human_use_actor` | `human_ai`, `human_intern_use_car`, `human_update` | `human_use_actor`: 1 on entry with its fifth argument, 2 when the seat cannot exit and the other can. On arrival + 100; `human_ai` then runs `0x65` `human_do_climb_in_car_lr` or `0x66` `human_use_actor(car, 2, 0, 0)` |
| +0x0b8 | queued_seat_timer_ms | H | `human_activate`, `human_ai`, `human_game_init`, `human_intern_use_car`, `human_update` | `entity_process_command`, `human_ai` | armed to 250 on seating; `human_ai` subtracts dt and acts unless the result is exactly 0 |
| +0x0bc | anim_machine | H | `anim_machine_del_extended_anim`, `anim_machine_recompile_matrix`, `anim_machine_set_anim` | `anim_machine_get_active_anim_time`, `anim_machine_stop_anims` | `lea ecx,[this+0xbc]` before every `anim_machine_*` call from a `C_human` method; ends at `+0x1e4`, the next field; `entity_process_command` calls `anim_machine_set_anim` on it. Layout: `structs/anim_machine.md` |
| +0x1e4 | crouched | H | `human_activate`, `human_do_crouched`, `human_game_init`, `human_save_game_load` | 19 functions ¹⁰ | forces walk over run, suppresses the category-4 clip variant, shrinks `dyn_coll_objects[0]`; `entity_action_tick` reads it in cases 1, 6, `0x18`, `0x1c`, `0x1d`, and the target's in case `0x1e` |
| +0x1e5 | aimed | H | 17 functions ¹¹ | 16 functions ¹² | cleared while carrying (`+0x6f8` / `+0x700`); raising the weapon arms `+0x1f0` |
| +0x1e6 | anim_side | H | `human_do_jump`, `human_save_game_load` | `human_animation_evaluate_and_dispatch`, `human_recompile_death_pos`, `human_save_game_save`, `human_shift_animation` | `human_shift_animation` patches the `?` in the clip file name from it; `human_do_jump` sets it while the active clip is in its first half |
| +0x1e8 | weapon_class | H | `human_activate`, `human_change_weapon_model`, `human_game_init` | 21 functions ¹³ | copied from the item's `predmety.def` record `+0x5c`; the crosshair shows for `[1,4)`; corner fire refuses 0 and 2 |
| +0x1ec | shooting | H | 16 functions ¹⁴ | `ai_blob_log_add_blob`, `entity_action_tick`, `human_dispatch_weapon_action_at_target`, `human_save_game_save`, `human_update`, `player_ai` | while set with a firearm class (1-3), `human_update` calls `human_shooting` every frame; a melee class sets it on the press; also gives a scent blob its flag `0x04` (`structs/ai_blob_log.md`) |
| +0x1ed | shot_any_angle | H | `human_activate`, `human_game_init`, `human_notify_callback` | `human_shooting` | `human_notify_callback`: clip event `0x1a` sets it with `+0x1ec`, `0x1b` clears both; `human_shooting` refuses a target over pi/2 from the facing only while it is clear |
| +0x1f0 | fire_action | H | 10 functions ¹⁵ | `human_ai`, `human_do_reload`, `human_repair_weapon_position`, `human_shooting`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk` | one of `g_fire_action_ready`, `g_fire_action_aimed`, `g_fire_action_onoff` |
| +0x1f4 | fire_action_step | H | 11 functions ¹⁶ | `human_ai`, `human_do_reload`, `human_repair_weapon_position`, `human_shooting`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk` | the table's stride is 8, `{int code; uint16 ms}`; `human_ai` advances it, and code 3 lets `human_repair_weapon_position` and `human_shooting` act |
| +0x1f8 | fire_action_step_ms | H | `human_activate`, `human_ai`, `human_change_weapon_model`, `human_do_reload`, `human_game_init`, `human_reset_weapon_effects`, `human_shooting` | `human_ai` | `human_do_reload` seeds it from the table; step code 4 resets it to dt |
| +0x1fc | reload_active | H | `human_activate`, `human_ai`, `human_change_weapon_model`, `human_do_change_weapon`, `human_game_init` | `human_ai` | `human_ai`: set on step code 1 with the sound `+0x6c4`; on code 3 it loads through `inventory_reload_one_round` or `_magazine`, then clears it |
| +0x1fd | reload_requested | H | `human_activate`, `human_ai`, `human_change_weapon_model`, `human_do_change_weapon`, `human_do_reload`, `human_game_init`, `human_shooting` | `human_ai` | set by `human_do_reload`, cleared by a shot. Set: `human_ai` never skips step code 1 and step code 2 ejects no cartridge |
| +0x1fe | shotgun_reload_pending | H | 10 functions ¹⁷ | `human_ai`, `human_dispatch_weapon_action_at_target`, `human_notify_callback`, `human_repair_weapon_position`, `player_ai` | set when a shot empties the pump shotgun (`predmety.def` id 11) with a round in reserve, or when a reload starts on it; `human_ai` loads one shell per action step 3 |
| +0x200 | shot_target | H | `entity_action_tick`, `human_dispatch_weapon_action_at_target`, `human_do_throw_grenade`, `human_save_game_load`, `human_set_shot_target_pos`, `human_shooting`, `player_ai_car_fire`, `player_ai_drive` | `human_dispatch_weapon_action_at_target`, `human_movement`, `human_notify_callback`, `human_save_game_save`, `human_shooting`, `player_ai_car_fire`, `player_ai_drive` | a `C_human` method writes it, so the field is the class's; `human_notify_callback` passes it to `game_new_grenade`; `entity_action_tick` cases `0x1a` and `0x1b` aim the corner fire with it |
| +0x20c | bolt_frame | H | `human_change_weapon_model` | `human_ai` | `human_ai` ejects the cartridge from it on step code 2 |
| +0x210 | muzzle_frame | H | `human_change_weapon_model` | `human_movement`, `human_shooting`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk` | the shot's origin; `human_shooting` returns while it is 0; `human_movement` mode 9 takes the aim ratio from it |
| +0x214 | muzzle_particle | H | `human_change_weapon_model` | `human_shooting` | `atoi` of the `muzzle` frame's string, plus 1 when `C_game +0x2d44` is set; `human_shooting` passes it to `game_new_shoot`, which creates that particle when it is above 0 |
| +0x218 | throw_charge_start_ms | H | `human_dispatch_weapon_action_at_target`, `human_save_game_load` | `human_dispatch_weapon_action_at_target`, `human_save_game_save`, `player_ai_walk` | the release cancels below 401 ms, caps the hold at 2000 ms and throws with power = hold ms * 0.015 * Strength (`+0x640`) |
| +0x21c | throw_charging | H | 14 functions ¹⁸ | `human_dispatch_weapon_action_at_target`, `human_save_game_save`, `player_ai_walk` | - |
| +0x220 | melee_button_ms | H | `human_activate`, `human_dispatch_weapon_action_at_target`, `human_game_init`, `human_save_game_load` | `entity_beware_cross_fire`, `human_dispatch_weapon_action_at_target`, `human_save_game_save`, `player_ai_walk` | `player_ai_walk` shows the HUD hold bar 200 ms after it; a release within 500 ms is a quick stroke |
| +0x224 | melee_windup_latched | H | `human_dispatch_weapon_action_at_target` | `human_dispatch_weapon_action_at_target` | class 7 held over 200 ms: `boj basb naprah HPT.i3d` once |
| +0x228 | melee_combo_step | H | `human_activate`, `human_dispatch_weapon_action_at_target`, `human_game_init`, `human_save_game_load` | `human_ai`, `human_dispatch_weapon_action_at_target`, `human_save_game_save` | +1 on a press within 1200 ms of the last, else 0, and wraps after 3; -1 on a stroke released after 500 ms. `human_ai`: -1 doubles melee damage, and with item id 3 or 4 two angle tests between the humans can deal the victim's health + 1 |
| +0x22c | guard_clip_active | H | `human_activate`, `human_dispatch_weapon_action_at_target`, `human_do_aimed`, `human_game_init`, `human_save_game_load`, `human_update` | `human_do_aimed`, `human_save_game_save` | only `human_update`'s guard arm stores 1 (§2b); lowering the weapon calls `anim_machine_del_extended_anim(4.0)` and clears it |
| +0x22d | sniper_mode | H | `human_activate`, `human_game_init`, `human_save_game_load`, `player_set_sniper_mode` | `human_save_game_save`, `human_shooting` | - |
| +0x22e | ground_unsettled | H | `human_activate`, `human_do_play_anim`, `human_game_init`, `human_movement_intern`, `human_update` | `human_movement`, `human_movement_intern` | while clear and `fall_speed` is 0 an idle human skips the movement pass |
| +0x22f | lethal_water_latched | H | `human_activate`, `human_game_init`, `human_movement_intern` | `human_movement_intern` | player camera mode 6, particle `0x25`, sound `0xec` within 30 units, health 0 |
| +0x230 | weapon_model_dirty | H | `human_activate`, `human_change_weapon_model`, `human_do_change_weapon`, `human_game_init` | `human_dispatch_weapon_action_at_target`, `human_do_aimed`, `human_do_change_weapon`, `human_hit`, `human_shooting` | `human_do_aimed` calls `human_change_weapon_model` when it is set |
| +0x234 | melee_stroke_ms | H | `human_activate`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_change_weapon`, `human_game_init`, `human_update` | `human_dispatch_weapon_action_at_target`, `human_update`, `player_ai_walk` | the clip time at the stroke; `human_update` counts it down; a press is refused while it is >= 0 |
| +0x238 | aim_pitch | H | `human_activate`, `human_game_init`, `human_save_game_load`, `player_ai_walk` | `game_tick`, `human_save_game_save` | `player_ai_walk` adds mouse y * `g_aiming_sensitivity_y` and clamps it; `game_tick` stores it, minus `+0xad0` * a constant, in `C_game +0x74`, the camera's `pitch` |
| +0x23c | clip_steps | H | `human_do_shock_down`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement`, `human_update` | indexed by `+0x2a0`; the array fills the space to `+0x2a0`, and direct stores reach step 2 (Open item 2) |
| +0x2a0 | clip_step_index | H | `human_activate`, `human_do_shock_down`, `human_game_init`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement`, `human_update` | 0 on arming, + 1 on each code 0 end or code 1 |
| +0x2a4 | pose_back1 | H | `human_change_model`, `human_game_done`, `human_game_init` | `human_change_model`, `human_game_done`, `human_pose_interpolate` | from driver slot `+0x6c`, bound through model slot `+0x94`; `human_pose_interpolate` turns it toward `+0x2b4` |
| +0x2a8 | pose_back2 | H | `human_change_model`, `human_game_done`, `human_game_init` | `human_change_model`, `human_game_done`, `human_pose_interpolate` | the same, toward `+0x2c0` |
| +0x2ac | pose_neck | H | `human_change_model`, `human_game_done`, `human_game_init` | `human_change_model`, `human_game_done`, `human_pose_interpolate` | the same, toward `+0x2cc` |
| +0x2b0 | pose_moving | H | `human_activate`, `human_change_model`, `human_game_init`, `human_pose_interpolate`, `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_save_game_load` | `human_pose_interpolate`, `human_save_game_save`, `human_update` | `human_pose_interpolate` clears it and sets it back per unfinished bone; `human_update` runs that function while it is set |
| +0x2b4 | pose_target_back1 | H | `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal` | `human_pose_interpolate`, `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_save_game_load`, `human_save_game_save` | normal: 4 x forward + look, normalised; null: (0,0,1) |
| +0x2c0 | pose_target_back2 | H | `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal` | `human_pose_interpolate`, `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_save_game_load`, `human_save_game_save` | the same as `+0x2b4` |
| +0x2cc | pose_target_neck | H | `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal` | `human_pose_interpolate`, `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_save_game_load`, `human_save_game_save` | normal: the blended look; aimed and null: (0,0,1) |
| +0x2d8 | look_target_frame | H | `human_change_model`, `human_game_init`, `human_pose_null_rotate`, `human_pose_set_look_target_frame` | `human_update`, `player_ai` | `human_update` plays an idle variation only while it is 0 |
| +0x2dc | shot_skeleton | H | `shot_skeleton_init` | `game_tick_shoot`, `shot_skeleton_test_collision` | 4 of the 6 callsites of `shot_skeleton_init` / `_test_collision` are `lea ecx,[reg+0x2dc]` (§6) |
| +0x40c | movement_mode | H | 40 functions ¹⁹ | 17 functions ²⁰ | - |
| +0x410 | vehicle_transition | H | 12 functions ²¹ | `entity_action_tick`, `entity_do_shoot_timed`, `human_hit`, `human_movement`, `human_save_game_load`, `human_save_game_save`, `human_update`, `player_lock_controls` | `human_update` when the clip ends: 1 `human_intern_use_car(1)`, 2 and 5 `human_intern_use_car(0)`, 3 `human_intern_use_railway(1)`, 4 `human_intern_use_railway(0)`, 6 takes the seat in place; `player_lock_controls` switches on it (cases 1-5) and zeroes it; `entity_action_tick` (cases `0x21`, `0x23`) and `entity_do_shoot_timed` read it on another human, the target |
| +0x414 | last_contact | H | `entity_release_target`, `game_invalidate_actor`, `game_invalidate_coll`, `human_activate`, `human_game_init`, `human_movement_intern`, `human_save_game_load` | `human_movement_intern` | the return of `human_test_move_coll` / `_in_owner`: byte 3 the material, byte 0 `& 0x40` dynamic, `+0x44` the actor. `game_invalidate_actor` and `game_invalidate_coll` zero it across the actors |
| +0x418 | fall_speed | H | 9 functions ²² | `human_movement`, `human_movement_intern`, `human_save_game_save` | §2d |
| +0x41c | fall_started_in_jump | H | `human_movement_intern` | `human_movement_intern` | raises the hard-landing limit from 8 to 18 and the damage offset from 2 to 12 |
| +0x420 | velocity | H | 24 functions ²³ | 11 functions ²⁴ | speed * the movement direction * `prop_cur[7]` (Speed); `human_get_actor_speed` returns it |
| +0x42c | turn_rate | H | 10 functions ³⁹ | `human_save_game_save`, `human_update` | stopped by `human_do_breath` and the strafe and diagonal setters |
| +0x430 | coast_velocity | H | 15 functions ⁴⁰ | `human_activate`, `human_game_init`, `human_movement`, `human_movement_intern`, `human_save_game_load`, `human_save_game_save`, `human_update`, `human_use_actor` | `human_update` copies `velocity` here; `human_movement` mode 5 moves the position by dt * this; `human_movement_intern` uses this * `+0x43c` when the velocity is zero. `human_use_actor`'s railway arm seeds it with forward * 1.5 |
| +0x43c | motion_ramp | H | 23 functions ²⁵ | `human_movement_intern`, `human_save_game_save`, `human_shooting`, `human_update` | `human_update` moves it by `dt * 0.004`; `human_shooting` multiplies it by 0.3 per shot; cleared whenever locomotion is interrupted |
| +0x440 | ground_normal | H | `human_movement_intern` | `human_movement_intern`, `human_save_game_load`, `human_save_game_save` | the out argument of the foot ray in `human_movement_intern`; over 1 rad from up counts as no ground, and the step is projected on it |
| +0x44c | mode_timer_ms | H | 14 functions ²⁶ | `human_movement`, `human_save_game_save`, `player_ai`, `player_try_use_car` | `human_movement` counts it down and leaves the mode at 0; `player_try_use_car` arms mode 10, the car break-in, with `car_database_get_steal_time` clamped to 2000..30000 and copies the total to `+0x474`; `human_hit` arms mode 5 with the clip time - 250 ms; `human_movement` arms mode 7 with rand + 2000 when rand gives 0, otherwise 0 |
| +0x450 | corpse_finish_ms | H | `human_activate`, `human_game_init`, `human_hit_in_car`, `human_movement`, `human_save_game_load` | `human_movement`, `human_save_game_save` | 500 from `human_game_init`, 1500 from `human_activate`, the clip time from `human_hit_in_car`; `human_movement` mode 8 counts it down |
| +0x454 | corpse_blood_pending | H | `human_activate`, `human_game_init`, `human_hit` | `human_movement` | set by `human_hit` for hit type 0; `human_movement` places it at `back2` + 0.5 y |
| +0x458 | agony_clip_band | H | `human_activate`, `human_game_init`, `human_hit`, `human_save_game_load` | `human_hit`, `human_movement`, `human_save_game_save` | by angle and crouch in `human_hit`; clips `+0x93` first, `+0x94` repeat, `+0x95` wait, `+0x96` last |
| +0x45c | agony_repeats_left | H | `human_hit`, `human_save_game_load` | `human_movement`, `human_save_game_save` | rand + 5 from `human_hit`; `human_movement` mode 7 counts it |
| +0x460 | last_shot_ms | H | `human_activate`, `human_dispatch_weapon_action_at_target`, `human_do_throw_grenade`, `human_game_init`, `human_save_game_load`, `human_shooting` | `ai_logs_tick`, `human_save_game_save` | `ai_logs_tick`: a human heard within 250 ms of it updates the log |
| +0x464 | track_sweep_active | H | `human_activate`, `human_animation_evaluate_and_dispatch`, `human_game_init`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_update` | `human_save_game_save`, `human_update` | 1 outside clips `0xf6..0x141`, `0x68`, `0x69`, `0x130`, `0x131`; both tracks at -1 clear it and seed a fall speed of 0.01 |
| +0x468 | track_sweep_pos | H | - | `human_animation_evaluate_and_dispatch`, `human_save_game_load`, `human_save_game_save`, `human_update` | `human_update` sweeps from it to the new position |
| +0x474 | break_in_total_ms | H | `player_try_use_car` | `human_movement`, `player_ai` | `human_movement` mode 10: the HUD bar is 1 - `+0x44c` / this; `player_ai` reads it for the same break-in |
| +0x478 | break_in_car | H | `human_activate`, `human_ai`, `human_game_init`, `human_hit`, `human_movement`, `player_ai`, `player_lock_controls`, `player_try_use_car` | `human_ai`, `human_is_this_car_stealing`, `human_movement` | `human_ai` cancels the break-in when mode is not 10; `human_is_this_car_stealing` compares a car with `+0x478`, `+0x98` and `+0x9c` |
| +0x47c | shadow_frame_handle | H | 13 functions ²⁷ | `human_change_state` | re-added on leaving a vehicle: a seated actor casts no world shadow |
| +0x480 | inventory | H | `human_activate`, `human_game_init`, `human_weapon_drop`, `human_weapon_toggle_held`, `inventory_holster`, `inventory_insert_item`, `inventory_remove_item_by_ptr` | `inventory_collect_items_to_list`, `inventory_get_active_weapon_ammo`, `inventory_has_item_with_property` | four slot bands of constant size; `human_weapon_drop` / `human_weapon_toggle_held` reach the in-hand slot at `+0x4a0`; ends at `+0x563`, the next field. Layout: `structs/inventory.md` |
| +0x564 | weapon_model_frame | H | `human_game_done`, `human_game_init` | 25 functions ²⁸ | `human_repair_weapon_position` zeroes its position and rewrites its flag word with `& 0xfffffedf \| 0x40000000` |
| +0x568 | left_hand_item_frame | H | `human_change_weapon_model`, `human_game_done`, `human_game_init` | 17 functions ²⁹ | the model of `inventory_get_left_hand_item_def`, parented to `+0x570`; released with `+0x564` through vtable `+0x24` |
| +0x56c | right_hand_bone | H | `human_change_model`, `human_game_init` | 10 functions ⁴⁴ | `human_change_weapon_model` parents every class's model to it first; `human_do_box_pick_up` hangs the box from it, and both pick-ups refuse while it is 0 |
| +0x570 | left_hand_bone | H | `human_change_model`, `human_game_init` | `entity_action_tick`, `human_change_model`, `human_change_weapon_model`, `human_drop_out_items`, `human_game_init`, `human_notify_callback`, `human_repair_weapon_position`, `human_start_weapon_effect` | `human_change_weapon_model` parents class 1 at seat 0 and classes 2, 3 off the left seats to it, and the left-hand item always; `human_notify_callback` event `0x21` returns any class but 0 to `+0x56c`; `entity_action_tick` case 6 aims from it at seat 0 in a car |
| +0x574 | left_hand_mount | H | `human_change_model`, `human_game_done`, `human_game_init` | `human_change_model`, `human_change_weapon_model`, `human_do_change_weapon`, `human_do_reload`, `human_game_done`, `human_game_init`, `human_notify_callback`, `human_repair_weapon_position` | slots `+0x50`, `+0x54` add and remove the weapon model; `human_notify_callback` events `0x1e`..`0x21` switch it during a clip |
| +0x578 | right_hand_mount | H | `human_change_model`, `human_game_done`, `human_game_init` | `human_change_model`, `human_change_weapon_model`, `human_game_done`, `human_game_init`, `human_repair_weapon_position` | slots `+0x50`, `+0x54` add and remove the weapon model |
| +0x57c | base_bone_frame | H | `human_change_model`, `human_game_init`, `human_init` | 18 functions ⁴¹ | its world position goes to scene slot `+0xbc` and `game_lightness_add`; `speech_channel_set_human` validates it as a morph frame |
| +0x580 | weapon_position_dirty | H | `human_activate`, `human_change_weapon_model`, `human_do_reload`, `human_game_init`, `human_notify_callback`, `human_repair_weapon_position` | `human_animation_evaluate_and_dispatch`, `human_repair_weapon_position`, `human_shooting` | `human_notify_callback` sets it in event cases `0x1f` and `0x20` |
| +0x581 | reload_effects_armed | H | 13 functions ⁴² | `human_notify_callback` | set by `human_do_reload` and by `human_shooting` on the last round with a reserve; `human_notify_callback` events `0x49..0x4d` run `human_start_weapon_effect` only while it is set, event `0x4b` always |
| +0x584 | railway_car_index | H | `human_save_game_load`, `human_use_actor` | `human_intern_use_railway`, `human_movement_intern`, `human_save_game_save`, `human_shooting`, `human_test_move_coll_in_owner`, `human_use_actor` | the out argument of `railway_get_entry_for_actor`; `n * 0x10` indexes the vehicle's collision lists |
| +0x588 | eye_target_frame | H | `human_change_model`, `human_init` | `human_change_model`, `human_game_done`, `human_init`, `speech_channel_set_human` | `speech_channel_set_human` hands it to the speech channel (`structs/speech.md`) |
| +0x58c | neck_target_frame | H | `human_change_model`, `human_init` | `human_change_model`, `human_game_done`, `human_init`, `speech_channel_set_human` | the same |
| +0x590 | idle_variation_timer_ms | H | 13 functions ³⁰ | `human_save_game_save`, `human_update` | when it passes 0 with `+0x598` set and `+0x2d8` clear, an idle variation plays |
| +0x594 | clip_hold_ms | H | 13 functions ³¹ | `human_animation_evaluate_and_dispatch`, `human_save_game_save`, `human_update` | a requested id up to 3 below the current one is held |
| +0x598 | idle_anims_enabled | H | `human_activate`, `human_game_init`, `human_set_idle_anims_enabled`, `player_game_init` | `human_update` | - |
| +0x59c | melee_sweep_count | H | `human_change_weapon_model` | `human_ai`, `human_change_weapon_model`, `human_notify_callback` | classes 0 and 5 the two hand bones, classes 6 and 7 the frames `hit1`..`hit4` |
| +0x5a0 | melee_sweep_frames | H | `human_change_weapon_model` | `human_ai`, `human_notify_callback` | the same |
| +0x5b0 | melee_sweep_prev_pos | H | `human_ai`, `human_notify_callback` | `human_ai` | `human_notify_callback` event `0x28` seeds them; `human_ai` line-tests from them to the current position, then stores it |
| +0x5e0 | melee_sweep_active | H | `human_activate`, `human_ai`, `human_change_weapon_model`, `human_game_init`, `human_movement`, `human_notify_callback` | `human_ai`, `human_movement` | clip event `0x28` sets it; a hit clears it; `human_movement` keeps it only while `ext_active` and `ext_blocks_actions` are both set |
| +0x5e4 | seated_state | H | `human_activate`, `human_game_init`, `human_intern_use_car`, `human_movement`, `human_update` | `human_die_in_exploding_car`, `human_hit_in_car`, `human_movement`, `human_save_game_save`, `human_shooting`, `human_update` | `human_movement` enters 2 from the aimed stance for class 1, or classes 1 and 3 off the driver's seat; `human_shooting` needs 2 in a car |
| +0x5e8 | seated_step_ms | H | `human_activate`, `human_game_init`, `human_hit_in_car`, `human_intern_use_car`, `human_movement`, `human_update` | `human_movement`, `human_save_game_save`, `human_shooting`, `human_update` | while >= 0 mode 9 stops and `human_shooting` returns |
| +0x5ec | seated_anim_stage | H | `human_activate`, `human_game_init`, `human_update` | `human_movement`, `human_save_game_save` | copied from `anim_machine` `active_stage` by `human_update` |
| +0x5f0 | seated_clip_time | H | `human_activate`, `human_game_init`, `human_update` | `human_movement`, `human_save_game_save` | `human_movement` passes `+0x5f0 * +0x5f4` behind `__ftol` at `0x0049bb29`, so the decomp text does not show it |
| +0x5f4 | aim_ratio | H | `human_activate`, `human_game_init`, `human_movement`, `human_update` | `human_movement`, `human_save_game_save` | `human_movement` moves it by dt * 0.003 toward the angle between the muzzle's forward and the shot target |
| +0x5f8 | original_model_name | H | `human_change_model`, `human_deactivate`, `human_game_done`, `human_game_init` | `human_change_model`, `human_deactivate`, `human_game_done` | `human_change_model` stores `strrchr(path, '\\') + 1` only while it is 0; `human_game_done` and `human_deactivate` restore that model and free it |
| +0x5fc | human_type | H | `human_init`, `human_load_data`, `player_init` | `human_hit`, `human_hit_in_car`, `human_is_hit_allowed_between_types`, `human_save_data` | index = trailing-zero count, at most 11 types; 0 = none (hits always allowed); indexes the 11x11 `g_hit_permission_matrix` |
| +0x600 | prop_max | H | `human_ctor`, `human_duplicate_from`, `human_load_data`, `human_load_stats_block_plain`, `human_load_stats_block_scaled`, `human_save_game_load`, `player_ctor` | `human_duplicate_from`, `human_game_init`, `human_getproperty`, `human_hit`, `human_save_data`, `human_save_game_save`, `player_save_game_load` | the 15-entry property vector, §2a |
| +0x63c | morale_base | H | `human_ctor`, `human_load_data`, `human_load_stats_block_plain`, `human_load_stats_block_scaled`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_spawn_gang_on_hit`, `police_manager_apply_state` | `human_activate`, `human_duplicate_from`, `human_game_init`, `human_save_data`, `human_save_game_save` | set to 0.5 by the ctor and both stats loaders right after the 15 maxima; the block copy of `+0x600` carries it to `+0x67c` |
| +0x640 | prop_cur | H | 14 functions ³² | 28 functions ³³ | §2a; `prop_cur[1]` (`+0x644`) is the health |
| +0x67c | morale | H | `human_activate`, `human_game_init`, `human_save_game_load` | `ai_character_get_morale`, `game_character_stack_push`, `human_save_game_save` | the 16th dword of the `+0x640` block; `ai_character_get_morale` adds a character offset to it |
| +0x680 | voice_id | H | 9 functions ⁴⁵ | `human_do_throw_occupant_from_car`, `human_new_scream`, `human_save_data` | 12 from `human_ctor`, 3 from `player_ctor`, rand + `0x1a` or `0x1d` on a carjacked driver; `human_new_scream` indexes the table by it |
| +0x684 | silent | H | `human_activate`, `human_game_init`, `human_set_silent` | `human_new_scream` | `human_new_scream` returns early while it is set |
| +0x688 | dyn_coll_objects | H | `human_create_dyn_colls`, `human_erase_dyn_colls` | 21 functions ³⁴ | the capsules that make the animated body hittable (§1) |
| +0x68c | dyn_coll_frames | H | `human_create_dyn_colls`, `human_erase_dyn_colls` | `human_create_dyn_colls`, `human_erase_dyn_colls`, `human_update_dyn_coll` | [0] the actor's frame, [1] the sub-frame `frame_base` |
| +0x690 | dyn_coll_count | H | `human_create_dyn_colls`, `human_erase_dyn_colls` | 21 functions ³⁵ | - |
| +0x694 | dyn_colls_active | H | `human_activate_colls`, `human_create_dyn_colls`, `human_deactivate_colls`, `human_erase_dyn_colls` | `human_save_game_load`, `human_update_dyn_coll` | - |
| +0x698 | coll_ignore | H | `entity_game_init`, `entity_reset`, `entity_tick_ghosts`, `human_ctor`, `human_dtor` | `human_dtor`, `human_movement_intern`, `human_set_next_coll_owners` | the colliders of the entity's `ai_ghost` records, rebuilt by `entity_tick_ghosts` (`structs/ai_ghost.md` §4a); merged with the capsules before a query; `entity_game_init` and `entity_reset` empty it (`end = begin`) |
| +0x6a8 | coll_owner_scratch | H | `human_can_exit_car_at_seat`, `human_ctor`, `human_dtor`, `human_set_next_coll_owners` | `human_can_enter_car`, `human_can_exit_car_at_seat`, `human_can_exit_car_free_space`, `human_dtor`, `human_movement_intern`, `human_set_next_coll_owners` | the merged list published to `g_collision`, whose queries skip every object in it |
| +0x6b8 | footstep_sound | H | `human_game_done`, `human_game_init` | `human_game_done`, `human_game_init`, `human_movement_intern`, `human_notify_callback`, `human_sounds_change` | the surface's sound from `g_surface_material_table` by `+0x6d4` on events 1, 2; sound `0xc0` on a hard landing |
| +0x6bc | last_footstep_ms | H | `human_activate`, `human_game_init`, `human_notify_callback` | `human_notify_callback` | a new one needs over 300 ms, so `human_sounds_change` skips this slot |
| +0x6c0 | shot_sound | H | `human_game_done`, `human_game_init` | `human_change_weapon_model`, `human_game_done`, `human_game_init`, `human_notify_callback`, `human_shooting`, `human_sounds_change` | loaded from the item's `predmety.def` `+0x54` |
| +0x6c4 | reload_sound | H | `human_game_done`, `human_game_init` | `human_ai`, `human_change_weapon_model`, `human_do_climb`, `human_game_done`, `human_game_init`, `human_sounds_change` | loaded from the item's `predmety.def` `+0x58`; switched on at reload start |
| +0x6c8 | gun_action_sound | H | `human_game_done`, `human_game_init` | `human_change_weapon_model`, `human_do_climb`, `human_game_done`, `human_game_init`, `human_sounds_change`, `human_start_weapon_effect` | sound `0xd7` for item `0xb`, `0xbe` for items `0xd`, `0xe`; played by weapon effect `0x4b` |
| +0x6cc | speech_sound | H | `human_game_done`, `human_game_init` | `entity_suspend_ai_and_hide`, `human_game_done`, `human_game_init`, `human_new_scream`, `human_sounds_change` | range 7..15 and the speech volume ratio |
| +0x6d0 | scream_priority | H | `human_activate`, `human_do_throw_occupant_from_car`, `human_game_init`, `human_new_scream` | `human_do_throw_occupant_from_car`, `human_new_scream` | a new scream needs a higher priority or no sound playing, and 501 or more refuses all: `human_do_throw_occupant_from_car` sets 1000 around its hit call |
| +0x6d4 | ground_material | H | `human_activate`, `human_game_init`, `human_movement_intern`, `human_save_game_load` | `human_notify_callback`, `human_save_game_save` | selects the footstep sound |
| +0x6d8 | wingman_slot | H | `human_activate`, `human_game_init`, `human_save_game_load`, `human_set_wingman_indicator` | `game_character_stack_pop`, `human_die_in_exploding_car`, `human_hit`, `human_hit_in_car`, `human_save_game_load`, `human_save_game_save`, `program_process_command` | while 0..3 every hit pushes the health percentage to that slot (`basic/artifacts/hud_elements.md` §7) |
| +0x6dc | wingman_portrait | H | `human_save_game_load`, `human_set_wingman_indicator` | `human_save_game_load`, `human_save_game_save` | - |
| +0x6e0 | fire_target_frame | H | `human_activate`, `human_game_init`, `human_set_fire_target_frame` | `human_shooting` | - |
| +0x6e4 | blood_drips_left | H | `human_activate`, `human_game_init`, `human_hit`, `human_save_game_load`, `human_update` | `human_hit`, `human_save_game_save`, `human_update` | armed to 10 by `human_hit` for hit type 0 with no attacker, a rand of 0, health over 1 and under 30% of the maximum; `blood_manager_add_blood` under the human |
| +0x6e8 | blood_drip_timer_ms | H | `human_hit`, `human_save_game_load`, `human_update` | `human_save_game_save`, `human_update` | rand + 500 |
| +0x6ec | print_total | H | `human_arm_bloody_footprints`, `human_save_game_load` | `human_notify_callback`, `human_save_game_save` | each footstep places one decal of texture `+0x6f0` with alpha `(+0x6ee + 2) / +0x6ec` (`structs/blood.md` §7e) |
| +0x6ee | prints_left | H | `human_activate`, `human_arm_bloody_footprints`, `human_game_init`, `human_notify_callback`, `human_save_game_load` | `human_arm_bloody_footprints`, `human_notify_callback`, `human_save_game_save` | - |
| +0x6f0 | print_texture | H | `human_arm_bloody_footprints`, `human_save_game_load` | `human_notify_callback`, `human_save_game_save` | - |
| +0x6f4 | spawn_record | H | 9 functions ⁴³ | `human_do_throw_occupant_from_car`, `human_game_done`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_spawn_gang_on_hit`, `pedestrian_wave_at_taxi`, `police_manager_apply_state`, `traffic_car_spawn_mafia_pursuit_car` | `human_game_done` restores the element's position, calls `pedestrian_activate_free(1, 0)`, releases the actor's frame when the byte is set, and frees it |
| +0x6f8 | carried_box | H | `human_activate`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_game_init` | 38 functions ³⁶ | every stance and anim setter refuses while it or `+0x700` is set |
| +0x6fc | carried_box_group | H | `entity_process_command`, `human_do_box_pick_up` | `entity_process_command`, `human_do_box_cancel`, `human_do_box_drop` | script `human_havebox` (case `0xdd`) returns it, or -1.0; the entity commands `BOX_TAKE` and `BOX_PUT` read it, and one of them stores it |
| +0x700 | carried_body | H | `human_activate`, `human_do_dead_body_pick_up`, `human_game_init` | 38 functions ³⁷ | script `human_havebody` (case `0xde`) tests it |
| +0x704 | carried_body_actor | H | `human_activate`, `human_do_dead_body_pick_up`, `human_game_init`, `human_update` | `human_animation_evaluate_and_dispatch`, `human_change_dead_anim`, `human_do_dead_body_pick_up`, `human_update` | non-zero runs `human_extern_ai_update` and `human_change_dead_anim` |
| +0x708 | body_drop_clip_ms | H | `human_activate`, `human_do_dead_body_pick_up`, `human_game_init`, `human_update` | `human_update` | the clip time of `NoseniMrtvolyMrtShozeni`; `human_update` counts it and clears `+0x704` |
| +0x70c | progress_sample_ms | H | `human_activate`, `human_game_init`, `human_movement` | `human_movement` | every 100 ms for the player, 500 for others |
| +0x710 | progress_sample_pos | H | `human_movement` | `human_activate`, `human_game_init`, `human_movement` | the distance moved since it feeds `+0x71c` |
| +0x71c | progress_ratio | H | `human_activate`, `human_game_init`, `human_movement` | `human_movement`, `human_movement_intern`, `player_ai_walk` | `human_movement_intern` below 0.3 uses a straight foot ray; `player_ai_walk` below 0.5 forces walking |
| +0x720 | no_hit_anim | H | `human_activate`, `human_game_init`, `human_set_no_hit_anim` | `human_hit` | the early return in `human_hit` follows the hit type switch, so it covers every type that reaches it; its writer's only caller is script `setnoanimhit` (case `0xbf`) |
| +0x724 | nav_locality | H | `ai_loc_add_or_refresh_seg_stat`, `ai_loc_init`, `ai_loc_prune_stale_seg_stats`, `ai_loc_update` | `ai_wayfind_plan_directed_route_v3` | the navigation-locality cache; `lea ecx,[esi+0x724]` at the callsites. Layout: `structs/ai_loc.md` |
| +0x758 | scent_trail | H | `ai_blob_log_add_blob`, `ai_blob_log_clear` | `ai_blob_log_get_blob`, `entity_trace_human_scent` | a 16-slot ring of scent blobs, one per 500 ms; `lea ecx,[reg+0x758]` at 7 of 8 callsites. Layout: `structs/ai_blob_log.md` |
| +0x99c | break_in_sound | H | `player_game_init`, `player_try_use_car` | `human_ai`, `human_hit`, `human_movement`, `player_ai`, `player_lock_controls` | `player_try_use_car` starts it; `game_stop_3d_sound` stops it when the break-in ends |
| +0x9a0 | scream_area_mask | H | `game_tick_scream_areas`, `human_activate`, `human_game_init` | `human_new_scream` | `human_new_scream` needs it to share a bit with `C_game` `scream_mask` |
| +0x9a4 | last_reaction_tick | H | `human_activate`, `human_game_init`, `human_hit` | `human_hit` | within 200 ms an alive human gets no new clip |
| +0x9a8 | last_reaction_sector | H | `human_activate`, `human_game_init`, `human_hit` | `human_hit` | the same value also refuses a new clip |
| +0x9ac | shots_fired | H | `human_activate`, `human_ai`, `human_game_init`, `human_shooting` | `human_ai`, `human_shooting` | `human_shooting`: + 1, or the pellet count (flag `2` of the item's `+0x20` gives 10) |
| +0x9b0 | hits_landed | H | `human_activate`, `human_ai`, `human_game_init`, `human_hit` | `human_ai` | `human_hit` adds 1.0 on the attacker, for kinds `0x1b` and 2 |
| +0x9b4 | shots_fired_mark | H | `human_activate`, `human_ai`, `human_game_init` | `human_ai` | every 5000 ms each pair becomes (x - mark) + 1 |
| +0x9b8 | hits_landed_mark | H | `human_activate`, `human_ai`, `human_game_init` | `human_ai` | the same |
| +0x9bc | hit_ratio | H | `human_ai` | `entity_action_tick` | `entity_action_tick` cases `0x1d` and `0x1e` scale the squared weapon range (`entity_weapon_get_max_range`) by it: a target beyond it is out of range, and in case `0x1e` × 0.8 is in range and × 1.2 the band between |
| +0x9c0 | hit_ratio_window_ms | H | `human_ai`, `human_game_init` | `human_ai` | - |
| +0x9c4 | corpse_use_search | H | `human_change_state`, `human_ctor`, `human_do_dead_body_pick_up` | `human_change_state`, `human_ctor`, `human_deactivate`, `human_do_dead_body_pick_up`, `human_game_done` | position of `frame_back1` + 0.5 y, `radius_sq` 0.7, `item_id` 1; registered when not in a vehicle |
| +0x9fc | corpse_use_move | H | `human_change_state`, `human_ctor`, `human_do_dead_body_pick_up` | `human_change_state`, `human_deactivate`, `human_do_dead_body_pick_up`, `human_game_done` | filled by `human_change_state`; `human_do_dead_body_pick_up` moves it, and `human_deactivate` deletes it |
| +0xa34 | entity_use_enabled | H | `human_activate`, `human_game_init`, `program_process_command` | `human_change_state` | the script arms of `program_process_command` around `entity_unregister_using_object` and `entity_add_using_object`; `human_change_state` re-registers it with event 60000 and state 0 |
| +0xa35 | exit_colls_pending | H | `human_activate`, `human_animation_evaluate_and_dispatch`, `human_game_init`, `human_use_actor` | `human_animation_evaluate_and_dispatch`, `human_update_dyn_coll` | set by `human_use_actor` on an exit, with `human_activate_colls(1)`; the next clip change clears it and calls `human_activate_colls(2)`; `human_update_dyn_coll` skips mask 1 capsules while it is set |
| +0xa36 | death_pos_done | H | `human_activate`, `human_game_init`, `human_recompile_death_pos` | `human_update` | clear: `human_update` applies the facing and `human_recompile_death_pos`; set: `human_process_death_pos` |
| +0xa38 | death_pos_start | H | `human_recompile_death_pos` | `human_process_death_pos`, `human_recompile_death_pos` | - |
| +0xa44 | death_pos_delta | H | `human_recompile_death_pos` | `human_process_death_pos` | `human_process_death_pos` lerps by elapsed / duration |
| +0xa50 | death_pos_elapsed_ms | H | `human_process_death_pos`, `human_recompile_death_pos` | `human_process_death_pos` | - |
| +0xa54 | death_pos_duration_ms | H | `human_recompile_death_pos` | `human_process_death_pos` | (clip time - 1) / 3, or 1 |
| +0xa58 | freeze_countdown_ms | H | `human_activate`, `human_game_init`, `human_update` | `human_ai`, `human_update` | -1 at init; no store sets it positive, so the freeze never engages |
| +0xa5c | weapon_effect_frame | H | `human_reset_weapon_effects`, `human_start_weapon_effect` | `human_reset_weapon_effects`, `human_tick_weapon_effect` | `cylinder`, `clip`, `zaver1`, `foregrip` or `hlavne`, by item |
| +0xa60 | weapon_effect_id | H | `human_activate`, `human_game_init`, `human_reset_weapon_effects`, `human_start_weapon_effect`, `human_tick_weapon_effect` | `human_tick_weapon_effect`, `human_update` | `human_update` runs `human_tick_weapon_effect` while it is non-zero |
| +0xa64 | weapon_effect_progress | H | `human_start_weapon_effect`, `human_tick_weapon_effect` | `human_tick_weapon_effect` | - |
| +0xa68 | weapon_effect_rate | H | `human_start_weapon_effect` | `human_tick_weapon_effect` | - |
| +0xa6c | pump_clip_delay_ms | H | `human_activate`, `human_game_init`, `human_reset_weapon_effects`, `human_shooting`, `human_update` | `human_update` | 500 from a pump shot with rounds left; at 0 `human_update` plays `gun08_stoj_Pump.i3d` |
| +0xa70 | hand_magazine_frame | H | `human_activate`, `human_game_done`, `human_game_init`, `human_reset_weapon_effects`, `human_start_weapon_effect` | `human_game_done`, `human_reset_weapon_effects`, `human_start_weapon_effect` | weapon effect `0x4d`, items 9 and 10; released by effect `0x4a` |
| +0xa74 | last_attacker | H | `game_invalidate_actor`, `human_activate`, `human_game_init`, `human_hit` | `ai_vis_logs_tick`, `camera_tick`, `police_action_tick` | `human_hit` stores it past its guards; `camera_tick` looks at it on the player's death; `police_action_tick` tests it against the squad's target |
| +0xa78 | speed_limiter_on | H | `human_activate`, `human_game_init`, `player_ai_drive` | `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update` | the limit is 16 while set, else 1000, with HUD flag `0x100` |
| +0xa79 | scores_in_freeride | H | `human_activate`, `human_game_init`, `pedestrian_spawn_gang_on_hit`, `traffic_car_spawn_mafia_pursuit_car` | `human_hit`, `human_hit_in_car` | `C_game +0x3640` + 500; human type 4 scores it plain; set on spawned attackers |
| +0xa7a | temp_ejected_driver | H | `human_activate`, `human_do_throw_occupant_from_car`, `human_game_init` | `police_manager_invalidate_discarded_npcs` | `police_manager_invalidate_discarded_npcs` removes and frees such an actor |
| +0xa7c | pose_look_point | H | `human_pose_set_pose_aimed`, `human_pose_set_pose_normal` | `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_track_record_key` | copied into recorder keys of type 3 at index `0xb..0xd` |
| +0xa88 | pose_request_flags | H | `human_activate`, `human_game_init`, `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal` | `human_pose_null_rotate`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_track_record_key` | bit 0 shortens the key throttle from 100 to 40 ms, bit 3 forces a key; values set the key's top nibble |
| +0xa8c | track_blend_weight | H | `human_change_model`, `human_game_init`, `human_track_apply_key_anim`, `human_track_apply_pose_at_clock` | `human_track_apply_pose_at_clock` | - |
| +0xa90 | track_clip_time | H | `human_change_model`, `human_game_init`, `human_track_apply_key_anim`, `human_track_apply_pose_at_clock`, `human_track_apply_pose_at_time`, `human_track_get_pos_at_time`, `human_track_get_rot_at_time` | `human_track_apply_pose_at_clock` | - |
| +0xa94 | track_clip_dt | H | `human_track_apply_key_anim`, `human_track_apply_pose_at_clock` | `human_track_apply_pose_at_clock` | `human_game_init` does not set it |
| +0xa98 | track_blend_running | H | `human_change_model`, `human_game_init`, `human_track_apply_key_anim`, `human_track_apply_pose_at_clock` | `human_track_apply_pose_at_clock` | - |
| +0xa99 | track_clip_slot | H | `human_change_model`, `human_game_init`, `human_track_apply_key_anim` | `human_track_apply_key_anim`, `human_track_apply_pose_at_clock` | - |
| +0xa9a | track_pos_applies | H | `human_change_model`, `human_game_init`, `human_track_apply_key_anim`, `human_track_apply_pose_at_time`, `human_track_get_pos_at_time`, `human_track_get_rot_at_time` | `human_track_apply_pose_at_clock`, `human_track_apply_pose_at_time`, `human_track_get_pos_at_time`, `human_track_get_rot_at_time` | key flag `0x1000` and the clip has one |

¹ `entity_process_command`, `human_activate`, `human_animation_evaluate_and_dispatch`, `human_change_anim`, `human_change_model`, `human_dispatch_weapon_action_at_target`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_do_dead_body_drop`, `human_do_dead_body_pick_up`, `human_do_stop_anim`, `human_do_throw_occupant_from_car`, `human_force_death`, `human_game_init`, `human_hit`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_reset_actions`, `human_save_game_load`, `human_throw_from_car_truck`, `human_track_apply_key_anim`, `human_update`, `player_ai_walk`, `player_lock_controls`.

² `entity_action_tick`, `human_animation_evaluate_and_dispatch`, `human_death_pose_align_to_ground`, `human_do_climb`, `human_do_jump`, `human_do_shock_stumble`, `human_do_shock_up`, `human_do_side_jump`, `human_hit`, `human_recompile_death_pos`, `human_save_game_save`, `human_shooting`, `human_track_apply_key_anim`, `human_track_record_key`, `human_update`, `player_ai_walk`.

³ `entity_action_tick`, `human_activate`, `human_animation_evaluate_and_dispatch`, `human_change_model`, `human_climb_out_of_car`, `human_die_in_exploding_car`, `human_dispatch_weapon_action_at_target`, `human_do_breath`, `human_do_castrate`, `human_do_climb`, `human_do_climb_in_car_lr`, `human_do_corner_breath`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_corner_view`, `human_do_dead_body_drop`, `human_do_dead_body_pick_up`, `human_do_hard_head`, `human_do_jump`, `human_do_shock_start`, `human_do_shock_stumble`, `human_do_shock_up`, `human_do_side_jump`, `human_do_stop_anim`, `human_do_throw_grenade`, `human_do_throw_occupant_from_car`, `human_force_death`, `human_game_init`, `human_hit`, `human_hit_in_car`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_look_to`, `human_movement`, `human_movement_intern`, `human_put_in_vehicle_at_seat`, `human_reset_actions`, `human_save_game_load`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_track_apply_key_anim`, `human_track_apply_pose_at_clock`, `human_turn_in_place`, `human_update`, `human_use_actor`, `player_ai_walk`, `player_lock_controls`, `player_try_use_car`.

⁴ `human_animation_evaluate_and_dispatch`, `human_change_dead_anim`, `human_death_pose_align_to_ground`, `human_do_climb`, `human_do_jump`, `human_do_side_jump`, `human_movement`, `human_track_apply_key_anim`, `human_update`.

⁵ `human_activate`, `human_climb_out_of_car`, `human_deactivate`, `human_die_in_exploding_car`, `human_eject_out_of_car`, `human_game_init`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_throw_from_car_truck`, `human_use_actor`.

⁶ `entity_process_command`, `human_activate_colls`, `human_ai`, `human_can_enter_car`, `human_can_exit_car_at_seat`, `human_change_state`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_collision`, `human_die_in_exploding_car`, `human_dispatch_weapon_action_at_target`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_climb_in_car_lr`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_eject_out_of_car`, `human_find_blocking_drawbridge`, `human_game_done`, `human_hit`, `human_hit_in_car`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_is_possible_coll`, `human_look_to`, `human_move_to`, `human_movement`, `human_movement_intern`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_put_in_vehicle_at_seat`, `human_repair_weapon_position`, `human_save_game_load`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_shooting`, `human_test_move_coll_in_owner`, `human_throw_from_car_truck`, `human_track_apply_pose_at_clock`, `human_update`, `human_update_dyn_coll`, `human_use_actor`, `player_ai`, `player_ai_drive`, `player_ai_walk`, `player_do_action_button_in_car`, `player_do_action_button_on_foot`, `player_indicator_set_aimed`, `player_invalidate_car_damage_bar`, `player_set_sniper_mode`, `player_show_weapon_name`, `player_update_controls`.

⁷ `human_activate`, `human_climb_out_of_car`, `human_deactivate`, `human_eject_out_of_car`, `human_game_init`, `human_hit`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_save_game_load`, `human_throw_from_car_truck`, `human_update`, `human_use_actor`, `player_lock_controls`.

⁸ `entity_ai`, `entity_process_command`, `human_change_state`, `human_change_weapon_model`, `human_intern_use_car`, `human_intern_use_railway`, `human_is_this_car_stealing`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_repair_weapon_position`, `human_save_game_get_size`, `human_save_game_load`, `human_save_game_save`, `human_update`, `player_ai`, `player_indicator_set_aimed`.

⁹ `human_ai`, `human_can_exit_car_at_seat`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_death_pose_align_to_ground`, `human_death_pose_push_out_of_wall`, `human_death_pose_settle_to_ground`, `human_die_in_exploding_car`, `human_dispatch_weapon_action_at_target`, `human_do_climb_in_car_lr`, `human_eject_out_of_car`, `human_game_done`, `human_hit`, `human_hit_in_car`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_movement`, `human_movement_intern`, `human_notify_callback`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_put_in_vehicle_at_seat`, `human_repair_weapon_position`, `human_save_game_load`, `human_save_game_save`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_shooting`, `human_test_move_coll_in_owner`, `human_throw_from_car_truck`, `human_update`, `human_update_dyn_coll`, `human_use_actor`, `player_ai`, `player_ai_walk`, `player_do_action_button_in_car`, `player_indicator_set_aimed`, `player_update_controls`.

¹⁰ `entity_action_tick`, `human_ai`, `human_animation_evaluate_and_dispatch`, `human_do_change_weapon`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_corner_view`, `human_do_crouched`, `human_do_jump`, `human_hit`, `human_save_game_save`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_shooting`, `human_test_move_coll`, `player_ai`.

¹¹ `human_activate`, `human_ai`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_aimed`, `human_do_change_weapon`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_throw_occupant_from_car`, `human_drop_out_items`, `human_game_init`, `human_reset_actions`, `human_save_game_load`, `human_use_actor`, `player_lock_controls`, `player_set_sniper_mode`, `player_try_use_car`.

¹² `entity_action_tick`, `human_ai`, `human_animation_evaluate_and_dispatch`, `human_dispatch_weapon_action_at_target`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_save_game_load`, `human_save_game_save`, `human_update`, `human_use_actor`, `player_ai`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk`, `player_do_action_button_in_car`, `player_set_sniper_mode`.

¹³ `human_animation_evaluate_and_dispatch`, `human_change_weapon_model`, `human_die_in_exploding_car`, `human_dispatch_weapon_action_at_target`, `human_do_climb_in_car_lr`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_jump`, `human_do_reload`, `human_hit`, `human_hit_in_car`, `human_movement`, `human_put_in_vehicle_at_seat`, `human_repair_weapon_position`, `human_update`, `human_use_actor`, `player_ai`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk`, `player_indicator_set_aimed`.

¹⁴ `entity_action_tick`, `human_activate`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_drop_out_items`, `human_game_init`, `human_hit`, `human_notify_callback`, `human_put_in_vehicle_at_seat`, `human_reset_actions`, `human_save_game_load`, `human_use_actor`.

¹⁵ `human_activate`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_change_weapon`, `human_do_reload`, `human_game_init`, `human_reset_weapon_effects`, `human_shooting`, `player_lock_controls`, `player_set_sniper_mode`.

¹⁶ `human_activate`, `human_ai`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_change_weapon`, `human_do_reload`, `human_game_init`, `human_reset_weapon_effects`, `human_shooting`, `player_lock_controls`, `player_set_sniper_mode`.

¹⁷ `human_activate`, `human_ai`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_change_weapon`, `human_do_reload`, `human_game_init`, `human_hit`, `human_shooting`, `player_ai_walk`.

¹⁸ `human_activate`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_climb`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_play_anim`, `human_do_throw_grenade`, `human_drop_out_items`, `human_game_init`, `human_save_game_load`, `human_use_actor`.

¹⁹ `human_activate`, `human_animation_evaluate_and_dispatch`, `human_climb_out_of_car`, `human_die_in_exploding_car`, `human_dispatch_weapon_action_at_target`, `human_do_breath`, `human_do_corner_fire`, `human_do_corner_view`, `human_do_dead_body_drop`, `human_do_jump`, `human_do_play_anim`, `human_do_shock_down`, `human_do_stop_anim`, `human_eject_out_of_car`, `human_force_ready_to_act`, `human_game_init`, `human_hit`, `human_hit_in_car`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_movement`, `human_movement_intern`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_throw_from_car_truck`, `human_update`, `human_use_actor`, `player_ai`, `player_lock_controls`, `player_try_use_car`.

²⁰ `entity_process_command`, `human_ai`, `human_animation_evaluate_and_dispatch`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_breath`, `human_do_throw_grenade`, `human_hit`, `human_movement`, `human_movement_intern`, `human_save_game_save`, `human_update`, `player_ai`, `player_ai_car_fire`, `player_ai_drive`, `player_ai_walk`, `player_lock_controls`.

²¹ `human_activate`, `human_do_climb_in_car_lr`, `human_game_init`, `human_hit`, `human_intern_throw_self_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_save_game_load`, `human_update`, `human_use_actor`, `player_lock_controls`.

²² `human_activate`, `human_do_play_anim`, `human_game_init`, `human_intern_use_car`, `human_movement`, `human_movement_intern`, `human_save_game_load`, `human_update`, script command human_forcefall.

²³ `human_activate`, `human_animation_evaluate_and_dispatch`, `human_death`, `human_dispatch_weapon_action_at_target`, `human_do_breath`, `human_do_climb`, `human_do_jump`, `human_do_play_anim`, `human_game_init`, `human_look_to`, `human_move_to`, `human_movement_intern`, `human_save_game_load`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_update`, `player_lock_controls`, `player_try_use_car`.

²⁴ `human_get_actor_speed`, `human_movement`, `human_movement_intern`, `human_save_game_save`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_update`.

²⁵ `entity_action_tick`, `human_activate`, `human_animation_evaluate_and_dispatch`, `human_climb_out_of_car`, `human_death`, `human_dispatch_weapon_action_at_target`, `human_do_climb`, `human_do_corner_fire`, `human_do_corner_view`, `human_do_dead_body_drop`, `human_do_play_anim`, `human_game_init`, `human_hit`, `human_intern_use_car_truck`, `human_look_to`, `human_move_to`, `human_movement_intern`, `human_save_game_load`, `human_shooting`, `human_turn_in_place`, `human_update`, `player_lock_controls`, `player_try_use_car`.

²⁶ `human_ai`, `human_climb_out_of_car`, `human_dispatch_weapon_action_at_target`, `human_do_corner_fire`, `human_do_corner_view`, `human_do_dead_body_drop`, `human_hit`, `human_movement`, `human_movement_intern`, `human_save_game_load`, `human_use_actor`, `player_ai`, `player_lock_controls`, `player_try_use_car`.

²⁷ `human_activate`, `human_change_model`, `human_change_state`, `human_climb_out_of_car`, `human_eject_out_of_car`, `human_game_init`, `human_intern_from_car`, `human_intern_use_car`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_put_in_vehicle_at_seat`, `human_throw_from_car_truck`, `human_use_actor`.

²⁸ `entity_action_tick`, `entity_is_in_player_fire_area`, `human_ai`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_dead_body_pick_up`, `human_do_reload`, `human_eject_out_of_car`, `human_game_done`, `human_hit`, `human_intern_from_car`, `human_intern_use_car`, `human_movement`, `human_put_in_vehicle_at_seat`, `human_repair_weapon_position`, `human_reset_weapon_effects`, `human_start_weapon_effect`, `human_update`, `human_use_actor`, `player_ai`, `player_lock_controls`, `player_try_use_car`.

²⁹ `human_ai`, `human_change_weapon_model`, `human_climb_out_of_car`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_do_dead_body_pick_up`, `human_eject_out_of_car`, `human_game_done`, `human_hit`, `human_intern_from_car`, `human_intern_use_car`, `human_movement`, `human_put_in_vehicle_at_seat`, `human_use_actor`, `player_ai`, `player_lock_controls`, `player_try_use_car`.

³⁰ `entity_process_command`, `human_activate`, `human_ai`, `human_animation_evaluate_and_dispatch`, `human_do_change_weapon`, `human_do_climb`, `human_do_jump`, `human_do_play_anim`, `human_drop_out_items`, `human_game_init`, `human_save_game_load`, `human_shooting`, `human_update`.

³¹ `entity_process_command`, `human_activate`, `human_ai`, `human_animation_evaluate_and_dispatch`, `human_do_change_weapon`, `human_do_climb`, `human_do_jump`, `human_do_play_anim`, `human_drop_out_items`, `human_game_init`, `human_save_game_load`, `human_shooting`, `human_update`.

³² `game_cheat_check`, `game_init`, `human_activate`, `human_die_in_exploding_car`, `human_force_death`, `human_game_init`, `human_hit`, `human_hit_in_car`, `human_movement`, `human_movement_intern`, `human_save_game_load`, `human_setproperty`, `human_throw_from_car_truck`, `player_save_game_load`.

³³ `human_ai`, `human_can_special_stroke`, `human_collision`, `human_dispatch_weapon_action_at_target`, `human_do_castrate`, `human_do_climb_in_car_lr`, `human_do_hard_head`, `human_getproperty`, `human_hit`, `human_hit_in_car`, `human_look_to`, `human_move_to`, `human_movement_intern`, `human_put_in_vehicle_at_seat`, `human_save_game_load`, `human_save_game_save`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_backward`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_shooting`, `human_throw_from_car_truck`, `human_use_actor`, `player_save_game_load`.

³⁴ `entity_action_tick`, `human_activate`, `human_activate_colls`, `human_ai`, `human_can_enter_car`, `human_can_exit_car_at_seat`, `human_create_dyn_colls`, `human_deactivate_colls`, `human_do_climb`, `human_do_crouched`, `human_erase_dyn_colls`, `human_game_done`, `human_game_init`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_movement_intern`, `human_set_next_coll_owners`, `human_test_move_coll_in_owner`, `human_throw_from_car_truck`, `human_update_dyn_coll`, `player_ai_walk`.

³⁵ `entity_action_tick`, `human_activate`, `human_activate_colls`, `human_ai`, `human_can_enter_car`, `human_can_exit_car_at_seat`, `human_create_dyn_colls`, `human_deactivate_colls`, `human_do_climb`, `human_do_crouched`, `human_erase_dyn_colls`, `human_game_done`, `human_game_init`, `human_intern_use_car_truck`, `human_intern_use_railway`, `human_movement_intern`, `human_set_next_coll_owners`, `human_test_move_coll_in_owner`, `human_throw_from_car_truck`, `human_update_dyn_coll`, `player_ai_walk`.

³⁶ `entity_process_command`, `human_animation_evaluate_and_dispatch`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_aimed`, `human_do_box_cancel`, `human_do_box_drop`, `human_do_box_pick_up`, `human_do_climb`, `human_do_corner_breath`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_corner_view`, `human_do_crouched`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_reload`, `human_do_shock_down`, `human_do_shock_start`, `human_do_shock_stumble`, `human_do_shock_up`, `human_do_side_jump`, `human_do_throw_occupant_from_car`, `human_hit`, `human_intern_use_car_truck`, `human_movement_intern`, `human_put_in_vehicle_at_seat`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_use_actor`, `human_weapon_drop`, `human_weapon_toggle_held`, `player_ai_walk`, `player_do_action_button_on_foot`, `player_indicator_set_aimed`.

³⁷ `human_animation_evaluate_and_dispatch`, `human_change_dead_anim`, `human_change_weapon_model`, `human_dispatch_weapon_action_at_target`, `human_do_aimed`, `human_do_box_pick_up`, `human_do_climb`, `human_do_corner_breath`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_do_corner_view`, `human_do_crouched`, `human_do_dead_body_drop`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_reload`, `human_do_shock_down`, `human_do_shock_start`, `human_do_shock_stumble`, `human_do_shock_up`, `human_do_side_jump`, `human_do_throw_occupant_from_car`, `human_hit`, `human_intern_use_car_truck`, `human_movement_intern`, `human_put_in_vehicle_at_seat`, `human_set_movement_intent_forward`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`, `human_update`, `human_use_actor`, `human_weapon_drop`, `human_weapon_toggle_held`, `player_ai_walk`, `player_do_action_button_on_foot`, `player_indicator_set_aimed`.

³⁸ `human_activate`, `human_animation_evaluate_and_dispatch`, `human_change_anim`, `human_deactivate`, `human_death_pose_align_to_ground`, `human_death_pose_push_out_of_wall`, `human_death_pose_settle_to_ground`, `human_dtor`, `human_game_done`, `human_game_init`, `human_recompile_death_pos`.

³⁹ `human_activate`, `human_do_breath`, `human_game_init`, `human_save_game_load`, `human_set_movement_intent_back_left`, `human_set_movement_intent_back_right`, `human_set_movement_intent_forward_left`, `human_set_movement_intent_forward_right`, `human_set_movement_intent_left`, `human_set_movement_intent_right`.

⁴⁰ `entity_action_tick`, `human_climb_out_of_car`, `human_death`, `human_dispatch_weapon_action_at_target`, `human_do_climb`, `human_do_corner_fire`, `human_do_corner_view`, `human_do_dead_body_drop`, `human_hit`, `human_intern_use_car_truck`, `human_look_to`, `human_move_to`, `human_movement_intern`, `human_turn_in_place`, `human_use_actor`.

⁴¹ `human_activate`, `human_change_model`, `human_climb_out_of_car`, `human_deactivate`, `human_death`, `human_eject_out_of_car`, `human_game_done`, `human_game_init`, `human_intern_from_car`, `human_intern_throw_self_from_car`, `human_intern_use_railway`, `human_pose_set_pose_aimed`, `human_pose_set_pose_normal`, `human_track_apply_pose_at_clock`, `human_track_apply_pose_at_time`, `human_update`, `human_use_actor`, `speech_channel_set_human`.

⁴² `human_activate`, `human_ai`, `human_change_weapon_model`, `human_deactivate`, `human_do_box_pick_up`, `human_do_change_weapon`, `human_do_climb`, `human_do_dead_body_pick_up`, `human_do_jump`, `human_do_reload`, `human_game_init`, `human_hit`, `human_shooting`.

⁴³ `human_activate`, `human_do_throw_occupant_from_car`, `human_game_done`, `human_game_init`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_spawn_gang_on_hit`, `pedestrian_wave_at_taxi`, `police_manager_apply_state`, `traffic_car_spawn_mafia_pursuit_car`.

⁴⁴ `entity_action_tick`, `human_change_model`, `human_change_weapon_model`, `human_do_box_cancel`, `human_do_box_pick_up`, `human_do_dead_body_pick_up`, `human_drop_out_items`, `human_game_init`, `human_notify_callback`, `human_repair_weapon_position`.

⁴⁵ `human_ctor`, `human_do_throw_occupant_from_car`, `human_load_data`, `pedestrian_manager_spawn_cop_from_pedestrian`, `pedestrian_spawn_gang_on_hit`, `pedestrian_wave_at_taxi`, `player_ctor`, `police_manager_apply_state`, `traffic_car_spawn_mafia_pursuit_car`.

### 2a. The 15 properties — the developers' own table

`human_setproperty` / `human_getproperty` (script dispatcher cases `0x65` / `0x66`) take the
property as a **name**; the load-time arm resolves it against a table of 15 sixteen-byte strings
that it scans backwards from a sentinel. Byte-read from `Game.exe` at `0x0063e3a8`, stride `0x10`:

| Index | Name | `prop_max` | `prop_cur` | Notes |
|---|---|---|---|---|
| 0 | `Strenght` | +0x600 | +0x640 | the throw power multiplies by it |
| 1 | `Energy` | +0x604 | +0x644 | the health: the ctor sets the maximum to 100.0, `player_ctor` to 200.0; damage subtracts from the live value; `game_cheat_check` sets it to 100; `player_save_game_load` clamps it and pushes `cur / max * 100` to `hud_set_player_health_value`; crossing 0.01 downward through `human_setproperty` is a death |
| 2 | `EnergyHandL` | +0x608 | +0x648 | the four limb energies: `player_ctor` seeds 4 × 40.0; the 10 hit capsules of the embedded `C_ShotSkeleton` fold onto them (§6) |
| 3 | `EnergyHandR` | +0x60c | +0x64c | the same |
| 4 | `EnergyLegL` | +0x610 | +0x650 | the same |
| 5 | `EnergyLegR` | +0x614 | +0x654 | the same |
| 6 | `Reactions` | +0x618 | +0x658 | - |
| 7 | `Speed` | +0x61c | +0x65c | the live value multiplies the velocity channels |
| 8 | `Aggresivity` | +0x620 | +0x660 | - |
| 9 | `Intelligence` | +0x624 | +0x664 | - |
| 10 | `Shooting` | +0x628 | +0x668 | - |
| 11 | `Sight` | +0x62c | +0x66c | - |
| 12 | `Hearing` | +0x630 | +0x670 | - |
| 13 | `Driving` | +0x634 | +0x674 | - |
| 14 | `Mass` | +0x638 | +0x678 | - |

`human_getproperty` returns `prop_cur[i] / prop_max[i] * 100`; `human_setproperty` writes
`prop_cur[i] = pct * 0.01 * prop_max[i]`. A name that matches nothing leaves the index at -1 and
the instruction is rejected at compile time, so every property name a shipped script uses is in
this list. The names are the table's exact strings, including the two misspellings `Strenght`
and `Aggresivity`: a script must spell them that way. The live values are the game quantities themselves, not copies: the movement code reads
`prop_cur[7]` directly.

### 2b. `weapon_class` — the eight classes, and the four melee guard clips

`human_change_weapon_model` copies the class from the in-hand item's `predmety.def` record (`+0x5c`,
stride `0xbc`):

| `+0x1e8` | Items (`predmety.def` id) |
|---|---|
| 0 | empty hands (0) and every non-weapon: the special action, the keys, the bucket, the flashlight, the documents, the papers, the explosives, the tickets, the package, the dog's head |
| 1 | the four handguns (6 .. 9) |
| 2 | the two rifles (13, 14) |
| 3 | the Thompson and the two shotguns (10 .. 12), and the unused silent copies (33, 34) |
| 4 | the throwables: the Molotov cocktail (5) and the grenade (15) |
| 5 | the knuckle-duster (2) |
| 6 | the knife (3) and the bottle (29) |
| 7 | the blunt melee items: the baseball bat (4), the steel bar (20), the crowbar (25), the plank (28), the sword (31) |

`human_update` runs two consecutive switches on `+0x1e8`. The first selects between three bodies
through a byte index table, grouping `{0, 5, 6, 7}` / `{1, 2, 3}` / `{4}`. Only the `{0, 5, 6, 7}`
arm continues, and only when the human is not seated in a type-4 linked object (`+0x98`, `+0xac`),
`aimed` is set, `+0x1b0` (`anim_machine` `ext_active`) is clear, `movement_mode` is not 5 and `shooting` is clear. The second
switch names a shipped guard clip, loads it through `i3d_cache_anim_create` and, on success, sets
`guard_clip_active` and calls `anim_machine_set_extended_anim(anim, 0.8, 4.0, 0)`:

| `+0x1e8` | Clip |
|---|---|
| 0 | `boj HPT box kryt.i3d` |
| 5 | `boj HPT box kryt.i3d` |
| 6 | `boj HPT nuz kryt.i3d` |
| 7 | `boj HPT basb kryt.i3d` |

*boj* = fight, *kryt* = guard, *box* / *nuz* / *basb* = fists / knife / baseball bat. Reading the
second switch needs a jump-table override in the disassembler; without one it renders as a single
indirect call through the table address.

### 2c. The clip-id bands of `cur_anim_id`

| Ids | Clips | Writer |
|---|---|---|
| `1` | idle (the same id on both builds) | `human_do_stop_anim`, `human_reset_actions` |
| `0x2f` / `0x33` + n | shock start, two groups 4 apart, n = severity 0..3 | `human_do_shock_start` |
| `0x3d` + 3n | shock down, n = severity 0..2; a second clip at + 1 | `human_do_shock_down` |
| `0x3e` / `0x41` / `0x44` | shock get-up (spacing + 3) | `human_do_shock_up` |
| `0x46` / `0x48` / `0x4a` | shock stumble (spacing + 2) | `human_do_shock_stumble` |
| `0x4e`..`0x4f` | corner breath | `human_do_corner_breath` |
| `0x50`..`0x52` | corner view (`0x52` requires crouched) | `human_do_corner_view` |
| `0x53`..`0x56` | corner fire (+ 1 for a weapon category 4) | `human_do_corner_fire` |
| `0x58`..`0x59` | corner fire aimed (+ 1 for a weapon category 4) | `human_do_corner_fire_aimed` |
| `0x66` / `0x67` | fall / landing (§2d) | `human_movement`, `human_movement_intern` |
| `0x83`..`0x92` | death; the second branch adds `0x8c`..`0x92` | `human_get_anim_id_for_death` |
| `0x142`..`0x149` | side jump | `human_do_side_jump` |

### 2d. The fall (`+0x418` / `+0x41c`)

Each movement tick adds `dt * 9.81` to `fall_speed`; at least 3.0 when the ground is out of reach.
Seeds start a fall: 0.05 in `human_movement` (60.0 for the player below y -30), 0.01 in
`human_update`, 0.1 when `human_intern_use_car` leaves a vehicle, +0.1 from script
`human_forcefall` (case `0xcb`); `human_do_play_anim` clears it. Above 2.0 the human enters
movement_mode 3 with clip `0x66`, or clip 1 while carrying a box or a body. On landing: up to 5.5 a soft landing; up to 8 (18 when
`fall_started_in_jump` is set) the landing clip `0x67` and a recovery timer in `mode_timer_ms`;
above that a fall hit of `(speed - 2) * 10` (`- 12` after a jump) through the hit virtual, with force
feedback at `speed * 0.1`. Above 50.0 in mode 1 or 3 the human dies.

### 2e. `S_human_clip_step` evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | code | H | `human_activate`, `human_do_shock_down`, `human_game_init`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement`, `human_update` | `human_game_init` and `human_activate` store 3 in step 0, so no step runs |
| +0x04 | clip | H | `human_do_shock_down`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement` | §1a |
| +0x08 | wait_ms | H | `human_hit`, `human_movement` | `human_movement` | - |
| +0x0c | unk_0c | U | `human_do_shock_down`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement` | Open item 1 |
| +0x10 | flags | H | `human_do_shock_down`, `human_hit`, `human_movement`, `human_throw_from_car_truck` | `human_movement` | 3 or 0 |

## 3. Method family

Coverage: all **146** PS2 `7C_human` methods are accounted for: **123 CONFIRMED** and
**1 STRUCTURAL** twins on PC functions, **10 inlined** into a PC host (each listed directly after
it), **1 base stub** on its shared PC slot function, and **11 PS2-only** methods with no PC
realisation (§7). PC-only: the `C_actor_track` overrides, the stats loaders, the death-pose splits,
the force-feedback pair, the script setters and the CRT pair.

The `human_` prefix is neither necessary nor sufficient. `C_player` methods (`player_ctor`,
`player_lock_controls`, `player_set_sniper_mode`, `player_indicator_set_aimed`,
`player_save_game_load`) write and read many fields of §2 as subclass methods; three twins of
`C_human` methods sit outside the unit (`human_get_actor_speed` and `human_set_shot_target_pos` in
the `C_actor` unit, `human_reset_pose_and_scent_trail` in the `C_entity` unit); and the unit holds
foreign classes (§4).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0048b5b0 | `crt_static_init_guard_0048b5b0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x0048b5d0 | `crt_atexit_register_0048b5d0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0048f430 | `human_ctor` | instance | `__ct__7C_humanFv` | 0x00165a40 | CONFIRMED | base ctor (kind=3) |
| 0x0048f650 | `human_scalar_deleting_dtor` | instance | - | - | - | vtable slot 16; the MSVC scalar-deleting destructor around `human_dtor` |
| 0x0048f670 | `human_dtor` | instance | `__dt__7C_humanFv` | 0x00166180 | CONFIRMED | base dtor |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8d40 | `human_pose_set_look_target_frame` | - | - | - | `+0x2d8` = the frame; script-command helper |
| 0x005c8d70 | `human_set_fire_target_frame` | - | - | - | `+0x6e0` = the frame |
| 0x005c8d80 | `human_set_no_hit_anim` | - | - | - | `+0x720`; sole caller script `setnoanimhit` (case `0xbf`) |
| 0x005c8d90 | `human_set_idle_anims_enabled` | - | - | - | `+0x598` |
| 0x005c8da0 | `human_set_silent` | - | - | - | `+0x684` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00412040 | `human_reset_pose_and_scent_trail` | - | - | - | un-crouches, resets the pose rotation and clears the scent trail (`human_do_crouched(0)`, `human_pose_null_rotate`, `ai_blob_log_clear`); sole caller `entity_resume_ai_and_show`. Sits in the `C_entity` unit |
| 0x00425710 | `human_get_actor_speed` | `GetActorSpeed__7C_humanFR8S_vector` | 0x0018f330 | CONFIRMED | 0x12 B; the `C_human` override of the `C_actor` base stub. Sits ~380 KB BELOW the C_human VA cluster (§4) |
| 0x00425780 | `human_set_shot_target_pos` | - | - | - | copies a point into `shot_target` (`+0x200`); sits in the `C_actor` unit |
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__7C_humanFUi` | 0x0016dcf0 | base-stub | slot 15; the shared one-argument no-op of the `C_actor` family; the PS2 body is empty |
| 0x0048b5e0 | `human_track_create` | - | - | - | slot 9 of the `C_actor_track` interface: creates the human recorder track (key types `0x28`, `0x2c`, `0x38`) |
| 0x0048b600 | `human_track_record_key` | - | - | - | slot 0: writes one recorder key (time, type, position, rotation, clip id), throttled to 100 / 40 ms |
| 0x0048bad0 | `human_track_apply_key_anim` | - | - | - | slot 1: applies a recorded clip |
| 0x0048bd30 | `human_track_apply_pose_at_clock` | - | - | - | slot 2 |
| 0x0048c320 | `human_track_apply_pose_at_time` | - | - | - | slot 3; sole caller of `human_track_queue_anim_event` |
| 0x0048cb40 | `human_track_dump_keys` | - | - | - | slot 12: writes `<frameName>.dump` |
| 0x0048cc20 | `human_track_get_pos_at_time` | - | - | - | slot 4 |
| 0x0048d0d0 | `human_track_get_rot_at_time` | - | - | - | slot 5 |
| 0x0048f7d0 | `human_init` | `Init__7C_humanFP9I3D_frame` | 0x00166300 | CONFIRMED | init from frame |
| 0x0048f880 | `human_duplicate_from` | `DuplicateFrom__7C_humanFP7C_actor` | 0x00166400 | CONFIRMED | - |
| 0x0048f8b0 | `human_save_data` | `SaveData__7C_humanFP7C_chunk` | 0x001664c0 | CONFIRMED | - |
| 0x0048f950 | `human_load_data` | `LoadData__7C_humanFP7C_chunk` | 0x00166840 | CONFIRMED | - |
| 0x0048fbd0 | `human_load_stats_block_scaled` | - | - | - | reads the `0x40`-byte stats record: the 15 property maxima plus a scale folded into `+0x604..+0x614` as `1/(1-x)`; then `morale_base` = 0.5 |
| 0x0048fd30 | `human_load_stats_block_plain` | - | - | - | reads the `0x3c`-byte stats record, the 15 maxima; then `morale_base` = 0.5 |
| 0x0048fe50 | `human_save_game_get_size` | `SaveGameGetSize__7C_humanFv` | 0x00166d80 | CONFIRMED | C_human SaveGameGetSize; C_entity has its OWN in the C_entity vtable |
| 0x0048fe90 | `human_save_game_save` | `SaveGameSave__7C_humanFPPv` | 0x00166de0 | CONFIRMED | writes type-6 record + C_human fields |
| 0x004901e0 | `human_save_game_load` | `SaveGameLoad__7C_humanFPPv` | 0x00167180 | CONFIRMED | C_human SaveGameLoad |
| 0x00490a20 | `human_death` | `Death__7C_humanFv` | 0x00167c10 | CONFIRMED | - |
| 0x00490b10 | `human_extern_ai_update` | `Extern_AI_Update__7C_humanFUi` | 0x00167d30 | CONFIRMED | small externally-driven anim tick (NOT human_ai): anim-machine IsActive/Tick + frame virtual + human_update_dyn_coll tail |
| 0x00490b50 | `human_sounds_change` | `SoundsChange__7C_humanFi` | 0x00167d90 | CONFIRMED | - |
| 0x00490bf0 | `human_ai` | `AI__7C_humanFUi` | 0x00167e60 | CONFIRMED | huge (0x6044) AI tick |
| - | - | `FireReload__7C_humanFP15S_GameItemTable` | 0x00183120 | inlined | 932 B: inlined into the animation-event-1 arm of `human_ai`; §4 |
| 0x00491b50 | `human_intern_use_car` | `Intern_UseCar__7C_humanFb` | 0x00169600 | CONFIRMED | internal enter/leave-vehicle transition; shared magic 0x3dcccccd (0.1f) |
| 0x00492000 | `human_intern_use_railway` | `Intern_UseRailway__7C_humanFb` | 0x00169bd0 | CONFIRMED | railway-carriage sibling; same vehicle-exit teardown + player gate |
| 0x00492620 | `human_intern_use_car_truck` | `_intern_UseCarTruck__7C_humanFP8C_car_exb` | 0x0016a290 | CONFIRMED | board/leave a truck CARGO BED; 5 identical guards incl. both carried_object slots |
| 0x004929a0 | `human_throw_from_car_truck` | `ThrowFromCarTruck__7C_humanFR8S_vectorf` | 0x0016a6d0 | CONFIRMED | eject from the cargo bed; guards kind==4 (car) + seat_index == -1 |
| 0x00492ff0 | `human_update` | `Update__7C_humanFUi` | 0x0016a920 | CONFIRMED | huge (0x9672) per-frame |
| 0x00494150 | `human_change_dead_anim` | `ChangeDeadAnim__7C_humanFv` | 0x0016cef0 | STRUCTURAL | picks the `NoseniMrtvolyMrt*` (carry-a-corpse) clip matching the current gait (§7) |
| 0x004942c0 | `human_animation_evaluate_and_dispatch` | `PersonAnim__7C_humanFfbb` | 0x0016cf10 | CONFIRMED | **the animation evaluate+dispatch call** every anim-setting method ends with; the Movement / Use_Actor / Hit / Turn_Turn diffs all rest on it |
| 0x00494c40 | `human_shift_animation` | `ShiftAnimation__7C_humanFi` | 0x0016d950 | CONFIRMED | Patches the `'?'` placeholder in the current clip's file name to `'L'`/`'R'` from the `+0x1e6` side flag and re-acquires the animation, then walks its notify events. Shared magic `strchr(name, 0x3f)`; **names field `+0x1e6` ↔ PS2 `+0x392`**, extending the proven stance band by one. Ratio 3.2 — PS2 inlines the `C_I3D_cache` search the PC factors into `i3d_cache_anim_create` |
| 0x00494d60 | `human_change_state` | `ChangeState__7C_humanFv` | 0x0016dd00 | CONFIRMED | - |
| 0x00495040 | `human_game_init` | `GameInit__7C_humanFv` | 0x0016e0b0 | CONFIRMED | - |
| 0x00495c30 | `human_game_done` | `GameDone__7C_humanFv` | 0x0016eaa0 | CONFIRMED | - |
| 0x00496110 | `human_create_dyn_colls` | `CreateDynColls__7C_humanFv` | 0x0016f0f0 | CONFIRMED | build the 2 dyn-coll capsules (+0x688/+0x68c/+0x690) |
| 0x00496340 | `human_erase_dyn_colls` | `EraseDynColls__7C_humanFv` | 0x0016f440 | CONFIRMED | free them |
| 0x004963d0 | `human_deactivate_colls` | `DeactivateColls__7C_humanFb` | 0x0016f500 | CONFIRMED | unregister from the collision world (mask-2 objects kept); does not free |
| 0x00496430 | `human_activate_colls` | `ActivateColls__7C_humanFi` | 0x0016f5c0 | CONFIRMED | (re)register; no-op while in a vehicle; mode 2 = mask-1 filtered |
| 0x004964b0 | `human_update_dyn_coll` | `UpdateDynColl__7C_humanFv` | 0x0016f700 | CONFIRMED | per-tick capsule refresh from the animated frames; pinned from both sides as the tail call of `Extern_AI_Update` |
| 0x00496710 | `human_hit` | `Hit__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0016fa50 | CONFIRMED | main hit handler (0x5384) |
| - | - | `HitLanding__7C_humanFv` | 0x00170f60 | inlined | inlined into `human_hit`: the arm gated on its ninth argument (landing clip `0x67`, PS2 `0x45`) |
| - | - | `HitAgony__7C_humanFRC8S_vector` | 0x00171340 | inlined | inlined into `human_hit`: the second arm; scream `+0x5e` is used by no other `Hit*` helper |
| - | - | `HitStroke__7C_humanFRC8S_vectorUi` | 0x00171470 | inlined | inlined into `human_hit`: the fourth arm (π/4 and 3π/4 split, clips `0xcd` / `0xec..0xf0`, PS2 `0x9b` / `0xba..0xbe`) |
| - | - | `HitGun__7C_humanFRC8S_vectorUib` | 0x00171670 | inlined | inlined into `human_hit`: the final `else` arm and both in-vehicle arms; the only helper that screams a constant (`0x54`) |
| - | - | `HitGunHPT__7C_humanFRC8S_vectorUib` | 0x00171af0 | inlined | called only from inside HitGun, so it is inlined within HitGun's own inlined body |
| - | - | `HitFall__7C_humanFRC8S_vectorUi` | 0x00171ea0 | inlined | inlined into `human_hit` (the third arm, gated on its fifth argument: π/4 and 3π/4 split, clips `0x6a..0x6c`, PS2 `0x48..0x4a`) and into `human_throw_from_car_truck`, whose PS2 twin ends in `HitFall(...)` |
| 0x00498760 | `human_hit_in_car` | `HitInCar__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00171060 | CONFIRMED | damage path while INSIDE a vehicle. **The ONLY one of the 7 Hit* helpers that is a real function on PC** |
| 0x00498a00 | `human_collision` | `Collision__7C_humanFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00172250 | CONFIRMED | - |
| 0x004991c0 | `human_is_possible_coll` | `IsPossibleColl__7C_humanFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x001728c0 | CONFIRMED | - |
| 0x00499380 | `human_track_queue_anim_event` | - | - | - | queues a `0x48`-byte {tag, value, frame name, position, rotation} record at `C_actor+0x60` for the next recorded key, while recording |
| 0x00499440 | `human_pose_interpolate` | `PoseInterpolate__7C_humanFUi` | 0x00172b60 | CONFIRMED | per-frame driver of the PROCEDURAL POSE layer (additive aim/lean over the playing clip). Exact sig `(this,uint)` + exact callee counts |
| 0x00499b90 | `human_pose_set_pose_normal` | `PoseSetPoseNormal__7C_humanFRC8S_vector` | 0x00173330 | CONFIRMED | set the NORMAL/relaxed upper-body pose. Carries **1.5707964 (π/2)**, the constant its twin carries and Aimed does not |
| 0x0049a400 | `human_pose_set_pose_aimed` | `PoseSetPoseAimed__7C_humanFR8S_vector` | 0x00173a50 | CONFIRMED | set the AIMED upper-body pose (writes back through a non-const `S_vector&`). Carries **1.0471976 (π/3)** |
| 0x0049aac0 | `human_pose_null_rotate` | `PoseNullRotate__7C_humanFv` | 0x00173ff0 | CONFIRMED | reset the procedural POSE rotation to neutral; proven by the Use_Actor diff (×3 both sides) |
| 0x0049ab50 | `human_pose_validate` | `PoseValidate__7C_humanFb` | 0x00174050 | CONFIRMED | on/off switch for the pose layer: blend weight 1.0f/0.0f into the frame + matrix refresh |
| 0x0049ab90 | `human_get_anim_id_for_death` | `GetAnimIDForDeath__7C_humanFibRC8S_vectorRC8S_vector` | 0x001740c0 | CONFIRMED | pick the DEATH anim clip (front/back by AngleTo vs π/2, + random variant) |
| 0x0049ac70 | `human_movement` | `Movement__7C_humanFUi` | 0x00174230 | CONFIRMED | **top-level LOCOMOTION driver** (0x5152 B). Callee multiplicities match exactly; calls human_movement_intern once |
| 0x0049bbc0 | `human_movement_intern` | `MovementIntern__7C_humanFUi` | 0x00175650 | CONFIRMED | **inner locomotion integrator — LARGEST C_human method** (0x6188 B). Motion integration + fall damage (→ hud_set_player_health_value) + impact sfx/particles |
| 0x0049d0c0 | `human_test_move_coll` | `TestMoveColl__7C_humanFRC8S_vectorRC8S_vectorR8S_vector` | 0x00176fb0 | CONFIRMED | sweep-test a proposed move against the world (from, to, out-hit) |
| 0x0049d540 | `human_set_next_coll_owners` | `DynSetNextCollOwnerMult__7C_humanFv` | 0x00176e80 | CONFIRMED | Publishes the human's dyn-coll capsules (plus the `ai_ghost` ignore list) as `g_collision`'s next-query owner set. Ratio 1.17. Names three fields — see §2a. It is the class's ONE rank inversion: PS2 emits it before `TestMoveColl`, PC after |
| 0x0049d640 | `human_test_move_coll_in_owner` | `TestMoveCollInOwner__7C_humanFRC8S_vectorRC8S_vectorR8S_vector` | 0x001774c0 | CONFIRMED | the same sweep restricted to the OWNER's collision set. **Identical signature to the above** — separated by size ordering + VA order |
| 0x0049d840 | `human_do_stop_anim` | `Do_StopAnim__7C_humanFv` | 0x00177720 | CONFIRMED | stop the anim → idle: movement_mode=1, cur_anim_id=**1** (idle clip id 1 is the SAME on both builds) |
| 0x0049d870 | `human_do_breath` | `Do_Breath__7C_humanFv` | 0x00177760 | CONFIRMED | settle to a breathing idle: movement_mode ∈ {1,2,3} → 1, zero the velocity channels |
| 0x0049d8c0 | `human_set_movement_intent_forward` | `Go_Front__7C_humanFb` | 0x001777c0 | CONFIRMED | walk/run forward: mode=2, vel channels, anim id |
| 0x0049d990 | `human_set_movement_intent_backward` | `Go_Back__7C_humanFv` | 0x001778c0 | CONFIRMED | cohort (address-order 1:1 with Go_*) |
| 0x0049da30 | `human_set_movement_intent_left` | `Go_Left__7C_humanFb` | 0x001779f0 | CONFIRMED | cohort |
| 0x0049db00 | `human_set_movement_intent_right` | `Go_Right__7C_humanFb` | 0x00177b00 | CONFIRMED | cohort |
| 0x0049dbd0 | `human_set_movement_intent_forward_left` | `Go_FrontLeft__7C_humanFb` | 0x00177c10 | CONFIRMED | cohort |
| 0x0049ddc0 | `human_set_movement_intent_forward_right` | `Go_FrontRight__7C_humanFb` | 0x00177e60 | CONFIRMED | cohort |
| 0x0049dfb0 | `human_set_movement_intent_back_left` | `Go_BackLeft__7C_humanFv` | 0x001780b0 | CONFIRMED | cohort |
| 0x0049e170 | `human_set_movement_intent_back_right` | `Go_BackRight__7C_humanFv` | 0x001782e0 | CONFIRMED | cohort |
| 0x0049e260 | `human_change_anim` | `ChangeAnim__7C_humanFPcPc` | 0x00178480 | CONFIRMED | swap the current animation BY NAME (two char*); scans the global anim-info table |
| 0x0049e520 | `human_get_anim_id_for_use_car` | `GetAnimIDForUseCar__7C_humanFbbbbbb` | 0x00178650 | CONFIRMED | pick the car enter/exit anim id from 6 bools (bitmask + base). Base consts differ per build — anim tables not comparable |
| 0x0049e580 | `human_put_in_vehicle_at_seat` | `Intern_UseCar__7C_humanFP5C_cari` | 0x001786b0 | CONFIRMED | the `(C_car*, int)` OVERLOAD of Intern_UseCar (distinct from the `(bool)` one = human_intern_use_car) |
| 0x0049ea60 | `human_intern_from_car` | `Intern_FromCar__7C_humanFP9I3D_frame` | 0x00178db0 | CONFIRMED | internal get-out-of-vehicle placement (release seat, drop free-look, engine-off if driver, re-parent frame). 1:1 on both builds: same 4-guard prologue, same player gate, same driver-seat engine-off |
| 0x0049eee0 | `human_repair_weapon_position` | `RepairWeaponPosition__7C_humanFv` | 0x00179270 | CONFIRMED | re-parent the weapon model frame to the correct attach frame. Shared mask `& 0xfffffedf \| 0x40000000` |
| 0x0049f180 | `human_change_weapon_model` | `ChangeWeaponModel__7C_humanFv` | 0x00179650 | CONFIRMED | rebuild the visible weapon geometry |
| 0x0049faa0 | `human_do_change_weapon` | `Do_ChangeWeapon__7C_humanFib` | 0x0017abd0 | CONFIRMED | switch equipped weapon + play the handling anim |
| 0x0049fd80 | `human_do_aimed` | `Do_Aimed__7C_humanFb` | 0x0017b210 | CONFIRMED | enter/leave the AIMED stance (+0x1e5) |
| 0x0049fdf0 | `human_do_crouched` | `Do_Crouched__7C_humanFb` | 0x0017b2a0 | CONFIRMED | enter/leave the CROUCHED stance (+0x1e4); resizes the dyn-coll capsule |
| 0x0049fef0 | `human_inventory_truncate` | `InventoryTruncate__7C_humanFv` | 0x0017b440 | CONFIRMED | clamps carried ammo to the game-item table max in seven slots: `+0x4a0`, `+0x500` and the five at `+0x4b0`..`+0x4f0` |
| 0x0049fff0 | `human_drop_out_items` | `DropOutItems__7C_humanFRQ23gif22vector<10S_GameItem,8>` | 0x0017b4f0 | CONFIRMED | materialize inventory items on the ground |
| 0x004a0740 | `human_find_blocking_drawbridge` | `TozNaselJsemTenNejvyssiMost__7C_humanFv` | 0x0017c040 | CONFIRMED | find a nearby **C_bridge** (kind 0x14) within 100u in state 3–5 while in a car with flag bit 25 (`car+0x294`). Sole consumer `human_use_actor` uses a hit to **REFUSE car exit** — you can't get out on a moving drawbridge. Shared magic bit `0x2000000` |
| 0x004a0840 | `human_can_exit_car_free_space` | `Can_DropOutFromCar_Free__7C_humanFP5C_cari` | 0x0017c220 | CONFIRMED | is there FREE SPACE beside the **GIVEN** car (argument) at this seat? **2.5** clearance. Sole consumer = the carjack eject |
| 0x004a0b10 | `human_can_exit_car_at_seat` | `Can_DropOutFromCar__7C_humanFi` | 0x0017c5c0 | CONFIRMED | can I exit **MY** car (`+0x98`) at this seat? seat<0 = my seat (`+0xac`); **1.2** clearance. Sole consumer = `human_use_actor` |
| 0x004a0e10 | `human_can_enter_car` | `Can_EnterToCar__7C_humanFP5C_carib` | 0x0017c990 | CONFIRMED | can I get INTO the given car at this seat? (4 params — the discriminator vs its 2/3-param siblings) |
| 0x004a11f0 | `human_climb_out_of_car` | `ZasrateLezeniZAuta_ve2rano__7C_humanFv` | 0x0017ce50 | CONFIRMED | the climb-out sequence: engine off, detach it from the police pursuit record, raycast the exit spot, place via `human_eject_out_of_car` |
| 0x004a1770 | `human_eject_out_of_car` | `ZasrateVybliknutiZAuta_ve3rano__7C_humanFv` | 0x0017d570 | CONFIRMED | compute + apply the exit position (11× vec3_set_xyz, 4× collision_test_line_vertical_thunk). The placement half of `human_climb_out_of_car` |
| 0x004a2400 | `human_do_climb_in_car_lr` | `Do_ClimbInCarLR__7C_humanFv` | 0x0017e570 | CONFIRMED | climb across the car interior L↔R. SetOwner×3 / ChangeWeaponModel×2 exact |
| 0x004a26d0 | `human_use_actor` | `Use_Actor__7C_humanFP7C_actoriii` | 0x0017e920 | CONFIRMED | **the general 'interact with that actor' dispatcher** (door/car/lever/body). Exact 5-arg sig + PersonAnim×3 + PoseNullRotate×3 |
| 0x004a3ae0 | `human_dispatch_weapon_action_at_target` | `Do_Shoot__7C_humanFbP8S_vector` | 0x001802f0 | CONFIRMED | fire the equipped weapon at target (documented twin) |
| - | - | `SetWaitAnim__7C_humanFv` | 0x00181a00 | inlined | 108 B: the 250 ms wait block, inlined into `human_dispatch_weapon_action_at_target`; §4 |
| - | - | `SetFreeBoxingAnim__7C_humanFUi` | 0x00181a70 | inlined | 772 B: the free-boxing clip block, inlined into `human_dispatch_weapon_action_at_target`; §4 |
| 0x004a4490 | `human_do_throw_grenade` | `Do_ThrowGrenade__7C_humanFR8S_vector` | 0x00181150 | CONFIRMED | throw the equipped throwable; gates on weapon_slot == 5 or 0xf (the throwable ids in predmety.def) |
| 0x004a4560 | `human_can_special_stroke` | `Can_SpecialStroke__7C_humanFb` | 0x00181290 | CONFIRMED | find a victim for a special melee takedown: kind ∈ {0x1b, 2}, alive, NOT crouched, movement_mode ∈ {1,2,9} |
| 0x004a49d0 | `human_do_castrate` | `Do_Castrate__7C_humanFP7C_human` | 0x00181760 | CONFIRMED | groin-strike melee; PAIRED anim (attacker 0xf1 / victim 0xf2). Identical prologue to hard_head → separated by SIBLING-DELTA + user override |
| 0x004a4aa0 | `human_do_hard_head` | `Do_HardHead__7C_humanFP7C_human` | 0x001818b0 | CONFIRMED | headbutt melee; PAIRED anim (attacker 0xf3 / victim 0xf4). Same trap, same resolution |
| 0x004a4b70 | `human_shooting` | `Shooting__7C_humanFv` | 0x00181d80 | CONFIRMED | **the per-frame FIRING tick** (one of the largest methods). Calls `police_manager_register_gunfire_offence_for_target` — firing is what makes the cops notice you |
| 0x004a60b0 | `human_do_reload` | `Do_Reload__7C_humanFv` | 0x001834d0 | CONFIRMED | reload the weapon; sets `weapon_position_dirty` (+0x580) |
| 0x004a61d0 | `human_weapon_toggle_held` | `Do_Holster__7C_humanFv` | 0x001836b0 | CONFIRMED | re-select/holster held weapon (from `human_free_hand` diff) |
| 0x004a6300 | `human_weapon_drop` | `Do_WeaponDrop__7C_humanFv` | 0x00183840 | CONFIRMED | discard weapon to ground (throw-away anim) |
| 0x004a63f0 | `human_do_play_anim` | `Do_PlayAnim__7C_humanFPCcbb` | 0x00183e40 | CONFIRMED | play a named animation clip |
| 0x004a64b0 | `human_get_bound_points_2d` | `GetBoundPoints2D__7C_humanFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x00183f40 | CONFIRMED | - |
| 0x004a6700 | `human_do_jump` | `Do_Jump__7C_humanFbb` | 0x00184090 | CONFIRMED | Start a jump, or hand off to `human_do_climb`. Three identical guards; `movement_mode = 0xb` (same constant both builds); vertical channel 6.0f; identical teardown tail. Ratio 1.15. Sets the `+0x1e6` side flag from which half of the active clip is playing |
| 0x004a6970 | `human_do_climb` | `Do_Climb__7C_humanFv` | 0x00184360 | CONFIRMED | Mount a ledge in front: probe at `y+0.3` along `forward*0.7`, retry at `y+0.7`, require a drop of 0.3–1.1 and `AngleTo ≤ 0.5`, three swept-sphere clearances, then COMMIT (zeroes `+0x420..+0x43c`, dispatches PersonAnim). Ratio 1.22 |
| 0x004a75b0 | `human_do_side_jump` | `Do_SideJump__7C_humanFRC8S_vectorRC8S_vector` | 0x00185260 | CONFIRMED | Side-jump/dodge: only with free hands (`+0x6f8`/`+0x700` == 0) and not in the side-jump anim range (`+0x70` ∉ [0x142,0x149]); picks the clip from the facing-vs-target angle, writes it to `+0x74`. **Shared magic π/4 (`0.7853982`) & 3π/4 (`2.3561945`) both builds** + the free-hands guard + the 8-wide anim-range gate |
| 0x004a7700 | `human_change_model` | `_intern_ChangeModel__7C_humanFPcb` | 0x00185420 | CONFIRMED | **swaps the human's 3D model** and runs the subclass half (§7) |
| 0x004a7d10 | `human_create_force_feedback_effects` | - | - | - | creates the four force-feedback effects the human code plays (`g_ff_effect_weapon` / `_jump` / `_hit` / `g_ff_effect_landing`); called from `game_init` |
| 0x004a7df0 | `human_release_force_feedback_effects` | - | - | - | releases them; called from `game_done` |
| 0x004a7e40 | `human_force_death` | `_intern_ForceDeath__7C_humanFv` | 0x00186050 | CONFIRMED | force a death/ragdoll pose immediately, bypassing the normal damage path; picks a random death variant (§7) |
| 0x004a8130 | `human_do_shock_start` | `Do_ShockStart__7C_humanFi` | 0x001866f0 | CONFIRMED | **SHOCK/STUN.** clamp severity 0..3 + rand between 2 bases 4 apart |
| 0x004a8190 | `human_do_shock_down` | `Do_ShockDown__7C_humanFi` | 0x00186790 | CONFIRMED | **SHOCK.** knock-down; clip = `severity*3 + base`, 2 clips staged |
| 0x004a8220 | `human_do_shock_up` | `Do_ShockUp__7C_humanFv` | 0x00186820 | CONFIRMED | **SHOCK.** get up; accepts ids spaced +3, advances by 1 |
| 0x004a8260 | `human_do_shock_stumble` | `Do_ShockStumble__7C_humanFv` | 0x001868a0 | CONFIRMED | **SHOCK.** stumble; accepts ids spaced +2, advances by 1 |
| 0x004a82a0 | `human_arm_bloody_footprints` | `cbCollTester__7C_humanFP18tDynamicCollObject` | 0x00186920 | CONFIRMED | the collision-tester callback: arms ten bloody footprints (`+0x6ec`/`+0x6ee`/`+0x6f0`) when the foot ray crosses a blood decal's sphere |
| 0x004a82e0 | `human_do_throw_occupant_from_car` | `Do_ThrowCocotFromCar__7C_humanFP5C_cari` | 0x00186960 | CONFIRMED | **the CARJACK EJECT** — haul the occupant out so you can take the car. Sole caller of `human_intern_throw_self_from_car` AND `human_can_exit_car_free_space` |
| 0x004a90b0 | `human_intern_throw_self_from_car` | `_intern_ThrowMeFromCar__7C_humanFv` | 0x00187590 | CONFIRMED | internal "eject ME from the car". Exactly 2 callsites, both inside the carjack — same shape on both builds |
| 0x004a96b0 | `human_do_box_pick_up` | `Do_FuckingBox__7C_humanFP9I3D_frame` | 0x00187b70 | CONFIRMED | PICK UP the carry-box; it never drops (the drop is `human_do_box_drop`). Exact callee counts: Add/DelInvalidItem ×8/×8 |
| 0x004a9bf0 | `human_do_box_cancel` | `Do_FuckingBoxCancel__7C_humanFv` | 0x00188130 | CONFIRMED | abort an in-progress box pick-up (mirror fingerprint of the pick-up) |
| 0x004aa190 | `human_do_dead_body_pick_up` | `Do_DeadBody__7C_humanFP9I3D_frame` | 0x00188820 | CONFIRMED | PICK UP a dead body and carry it; called with 0 by `human_do_dead_body_drop` to release |
| 0x004aab10 | `human_die_in_exploding_car` | `EineMeineKleineAutoInLuft__7C_humanFP5C_car` | 0x00188c70 | CONFIRMED | a human seated in a car that `car_explosion` blows up dies in its seat: the charred model `ohorelec01.i3d` (a PC-only statement), the kill stores `cur_health` 0, `+0x5d` 0 and the anim-machine lock 1 (PS2 `+0x6a8`, `+0x61`, `+0x37c`), and a seated death clip by seat. Statement for statement; the size ratio 2.7 is the PS2's inlined animation-cache search |
| 0x004aad00 | `human_can_aim` | `Can_Aimed__7C_humanFi` | 0x001891a0 | CONFIRMED | is there room in front to raise the weapon? (§7) |
| 0x004aaed0 | `human_do_corner_breath` | `Do_CornerBreath__7C_humanFQ27C_human13E_corner_view` | 0x001892d0 | CONFIRMED | **COVER SYSTEM.** idle breathing pose against cover; clips 0x4e–0x4f |
| 0x004aaf10 | `human_do_corner_view` | `Do_CornerView__7C_humanFQ27C_human13E_corner_view` | 0x00189340 | CONFIRMED | **COVER.** lean out to look; variant 2 requires CROUCHED; clips 0x50–0x52 |
| 0x004aafd0 | `human_do_corner_fire` | `Do_CornerFire__7C_humanFQ27C_human13E_corner_view` | 0x00189430 | CONFIRMED | **COVER.** lean out and fire (blind); clips 0x53–0x56 |
| 0x004ab0f0 | `human_do_corner_fire_aimed` | `Do_CornerFireAimed__7C_humanFQ27C_human13E_corner_view` | 0x001895a0 | CONFIRMED | **COVER.** lean out and fire AIMED; clips 0x58–0x59 |
| 0x004ab1c0 | `human_force_ready_to_act` | `Force_CanWorking__7C_humanFv` | 0x001896d0 | CONFIRMED | force `movement_mode` (+0x40c) = **1** = the idle/READY mode. Consumer: **`entity_action_tick`** — makes the NPC available for a new task |
| 0x004ab1d0 | `human_turn_in_place` | `Turn_Turn__7C_humanFv` | 0x001896e0 | CONFIRMED | **rotate on the spot**: request the turn clip, zero `motion_ramp` and `coast_velocity`. Consumers: `entity_action_tick` ×6, script label `c21_postrotate_facing` |
| 0x004ab200 | `human_activate` | `Activate__7C_humanFv` | 0x00189770 | CONFIRMED | lifecycle **activate**: frame on, `human_pose_validate`, register with the zone/AI-loc system |
| 0x004ab8a0 | `human_deactivate` | `Deactivate__7C_humanFv` | 0x00189c50 | CONFIRMED | lifecycle **deactivate**: frame off, release 2 object registrations, free +0x5f8 |
| 0x004ab9e0 | `human_new_scream` | `NewScream__7C_humanFUi` | 0x00189d90 | CONFIRMED | play a voice/scream line by id (guards + `%08lu.wav`). Its 4 calls inside human_hit ARE the inlined Hit* helpers' internals |
| 0x004abc00 | `human_do_box_drop` | `Do_FuckingBoxDrop__7C_humanFv` | 0x0018a440 | CONFIRMED | materialize carried object as scene entity |
| 0x004abd50 | `human_do_dead_body_drop` | `Do_DeadBodyDrop__7C_humanFv` | 0x0018a690 | CONFIRMED | put a carried body down; guards on carried_object_2 (+0x700); calls `human_do_dead_body_pick_up(0)` = release |
| 0x004abde0 | `human_reset_actions` | `ResetActions__7C_humanFv` | 0x0018a730 | CONFIRMED | cancel all actions → neutral idle: un-crouch, clear aimed, idle clip **1** (same id on both builds), PoseNullRotate |
| 0x004abe40 | `human_recompile_death_pos` | `RecompileDeathPos__7C_humanFv` | 0x0018a7e0 | CONFIRMED | **DEATH.** recompute where a dead body settles. PS2 (7528 B) INLINED its frame helpers; PC keeps them separate (`human_death_pose_align_to_ground`, `human_death_pose_push_out_of_wall`, `human_death_pose_settle_to_ground`) — sum ×1.37. 2 callers on BOTH sides |
| 0x004ac430 | `human_death_pose_align_to_ground` | - | - | - | first split of `human_recompile_death_pos`: averages the ground normal under the sample points (surfaces within 63° of up) and rebuilds the corpse's rotation |
| 0x004ac9f0 | `human_death_pose_push_out_of_wall` | - | - | - | second split: slides the corpse out of the deepest wall intersection by `dist + 0.3` |
| 0x004ace80 | `human_death_pose_settle_to_ground` | - | - | - | third split: a three-point support test; drops the corpse to the nearest ground or drapes it over an edge |
| 0x004ad3c0 | `human_process_death_pos` | `ProcessDeathPos__7C_humanFUi` | 0x0018c550 | CONFIRMED | **DEATH.** per-tick settling toward the computed death pose; identical guard on the +0xa50/+0xa54 pair |
| 0x004ad4b0 | `human_is_this_car_stealing` | `IsThisCarStealing__7C_humanFP5C_car` | 0x0018c670 | CONFIRMED | is anyone else taking this car? scans actors of kind ∈ {2, 0x1b} for a reference via +0x478 / +0x98 / +0x9c |
| 0x004ad520 | `human_free_hand` | `FreeHand__7C_humanFv` | 0x0018c720 | CONFIRMED | holster + parent physics prop to l_hand/r_hand |
| 0x004ad770 | `human_weapon_effect_no_magazine` | `BezZasobnikuWeaponEffect__7C_humanFiP9I3D_model` | 0x0018ca90 | CONFIRMED | **WEAPONEFFECT.** hide the MAGAZINE geometry — the PS2 name is Czech for "without magazine"; weapon ids 9 and 10 |
| 0x004ad7c0 | `human_start_weapon_effect` | `StartWeaponEffect__7C_humanFi` | 0x0018cb10 | CONFIRMED | **WEAPONEFFECT.** spawn + place a weapon visual effect on the weapon bone |
| - | - | `PlaySoundFromSndTable__7C_humanFi` | 0x0018e350 | inlined | 180 B: pulls a preloaded sound out of `C_sound_table` and positions it 1.5 units up (§7) |
| 0x004adf70 | `human_tick_weapon_effect` | `TickWeaponEffect__7C_humanFUi` | 0x0018d400 | CONFIRMED | **WEAPONEFFECT.** per-frame advance of live effects |
| 0x004ae460 | `human_reset_weapon_effects` | `ResetWeaponEffects__7C_humanFv` | 0x0018dc10 | CONFIRMED | **WEAPONEFFECT.** tear down all live effects (called from the hit/death paths). Exact sig + exact x2/x2/x2 marker counts |
| 0x004ae710 | `human_fake_weapon_effect` | `FakeWeaponEffect__7C_humanFP9I3D_frameib` | 0x0018e060 | CONFIRMED | **WEAPONEFFECT.** non-simulated effect on a frame; id 9 only; attaches to the sub-frame `"zaver1"`, Czech for a gun's BOLT |
| 0x004ae770 | `human_is_hit_allowed_between_types` | `Can_Hit_FilterHumanType__7C_humanFP7C_humanP7C_human` | 0x0018e110 | CONFIRMED | **may type A hit type B?** 11×11 lookup in `g_hit_permission_matrix` indexed by each human's single-bit `human_type` (+0x5fc). Special rule: type 1 may hit type 0 only if the victim is the ANGRIEST actor |
| 0x005c8d50 | `human_set_wingman_indicator` | - | - | - | `+0x6d8` = HUD slot, `+0x6dc` = portrait; (-1, 0) clears it. Script cases `0x79` / `0x7a` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0048d590 | `human_notify_callback` | `human_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi` | 0x00163870 | CONFIRMED | the human I3D animation-event callback, a free function: footsteps and bloody prints, the throw (`game_new_grenade`), `weapon_position_dirty` (event `0x1f`) |
| 0x0048f330 | `human_ground_drop_below` | - | - | - | ground probe below a point, dynamic objects included; the point in ECX. 5 callsites, all `human_*` |
| 0x0048f3b0 | `human_ground_drop_below_static` | - | - | - | the same probe with dynamic objects excluded. 8 callsites in `human_*` and `player_try_use_car` |
| 0x0048f7b0 | `vector_dispose_at_0x24` | - | - | - | `std::vector` destructor at a constant displacement, ICF-folded across `ai_loc` and `ai_logs`; reached only from three unwind funclets |
| 0x00498740 | `clear4_dwords_at` | - | - | - | zeroes four dwords; 3 callsites, all in `program_process_command` |
| 0x004ae7f0 | `clear1_dword_at_0x04` | - | - | - | a 54-callsite COMDAT fold in 28 unrelated functions, owned by no class |
| 0x004afe40 | `human_look_to` | `HumanLookTo__FP7C_humanRC8S_vectorUibb` | 0x00190730 | CONFIRMED | steers a human's facing toward a point in XZ; a free function of the movement unit |
| 0x004b0280 | `human_move_to` | `HumanMoveTo__FP7C_humanR8S_vectorUibUibb` | 0x00190b40 | CONFIRMED | commits a human to move toward a point in XZ, steering through `human_look_to` |

## 4. The `C_human.cpp` translation unit

**`[0x0048b5b0, 0x004ae800)`**, bounded by the unit's CRT static-init pair `0x0048b5b0` /
`0x0048b5d0` and the next unit's pair `0x004ae800` / `0x004ae820`. Enumerating every PC function
start in the range, unwind entries included, gives **159 real starts, all named**: 161 apparent,
minus the two split phantoms `0x004a663a` and `0x004a66a0`, both inside
`human_get_bound_points_2d`, whose true size is therefore 592 B.

MSVC emits an inline member as a COMDAT into whichever unit instantiates it, and a class defined
inside another class's `.cpp` links with that file. So besides `C_human`'s own functions the unit
holds:

| Also in this unit | Why | Prefix |
|---|---|---|
| `ai_loc` (4 of 4), `ai_blob_log` (3 of 3) | declared inside `C_human.cpp` | their own |
| `inventory_dtor`, `inventory_reset`, `inventory_get_in_hand_item` | `G_Inventory` is embedded at `+0x480`, so `human_dtor` instantiates its inline members | `inventory_` |
| `vector_dispose_at_0x24` | an ICF fold across `ai_loc` and `ai_logs` | - |
| the four force-feedback effects and their create / release pair | played only by `human_*` code | `human_` / `force_feedback_` |
| `clear1_dword_at_0x04`, `clear4_dwords_at` | COMDAT folds that landed here | - |

⇒ The prefix follows the class, never the unit. Both builds emit a class's methods in source
order, so inside the cluster the PS2 VA sequence and the PC VA sequence are the same sequence
(`human_set_next_coll_owners` is the class's one rank inversion), which is how the mapping of §3
was generated; every CONFIRMED row was then body-diffed on both sides.

## 5. The `Do_Corner*` and `Do_Shock*` families

Two complete four-method families.

**The cover system** — `human_do_corner_breath` / `_view` / `_fire` / `_fire_aimed`, clips
`0x4e`..`0x59` (§2c). The variant argument (`E_corner_view`, values 0 / 1 / 2) selects the lean.
Shared rules: each refuses while the human carries anything (`+0x6f8` / `+0x700`); each writes the
chosen clip to `cur_anim_id` and returns 1 on success and 0 on refusal; variant 2 requires
`crouched`; the two fire variants also refuse when `weapon_class` is 0 or 2 (bare hands or a
rifle), and pick the alternate clip (base + 1) when the human is not crouched and the equipped
item's def record has `+0x50 == 4` (`inventory_get_in_hand_item_def`). None of the four touches
the police system.

**The shock / stun reactions:**

| PC name | Mechanism |
|---|---|
| `human_do_shock_start` | clamp severity 0..3; a random pick between two bases 4 apart (`0x2f` / `0x33`) + severity |
| `human_do_shock_down` | clamp severity 0..2; clip `severity * 3 + 0x3d`, a second clip staged at + 1 (`clip_steps`, §1a) |
| `human_do_shock_up` | reads `anim_slot`; accepts `0x3e` / `0x41` / `0x44` (spacing + 3); writes `cur_anim_id = id + 1` |
| `human_do_shock_stumble` | reads `anim_slot`; accepts `0x46` / `0x48` / `0x4a` (spacing + 2); writes `cur_anim_id = id + 1` |

`human_do_shock_up` and `human_do_shock_stumble` are structurally identical; the spacing of the
ids they accept is what tells them apart (§7).

## 6. The embedded `C_ShotSkeleton` at `+0x2dc`

`C_human` embeds a whole `C_ShotSkeleton` (prefix `shot_skeleton_`) at `+0x2dc`: the per-limb hit
locator. Given a shot ray it returns which body part was hit (1 right arm, 2 left arm, 3 right leg,
4 left leg, 5 torso, 6 head and neck), or 0.

Both methods are `__thiscall`, so the decomp cannot show the offset. A disassembly census of the
ECX value at every callsite of `shot_skeleton_init` / `shot_skeleton_test_collision` finds six
callsites, four of them `lea ecx,[reg+0x2dc]`: `human_game_init`, `human_change_model`,
`human_ai` and `game_tick_shoot`. The other two are `lea ecx,[esp+N]` inside
`pedestrian_crowd_hit`, which builds one on the stack. The object is `0x40` (16 joint handles) +
10 × `0x18` (the capsule table) = `0x130` bytes, so it spans `+0x2dc..+0x40b` and ends exactly
where `movement_mode` begins. Full write-up: `structs/shot_skeleton.md`.

## 7. PC vs PS2

### 7a. Field offsets

The band does not rebase by a constant (the kind field alone is PC `+0x10` / PS2 `+0x50`), so
each pair is pinned by a body diff. Clip ids are per-build.

| PC | PS2 | Pinned by |
|---|---|---|
| `+0x70` / `+0x74` | `+0x200` / `+0x204` | the `Do_Corner*` / `Do_Shock*` body-diffs. PS2 passes `+0x200` to `GetAnimIndex__FUi` in `ShiftAnimation` / `_intern_ForceDeath` / `RecompileDeathPos` and compares it against clip ids `0x1b..0x23`, `0x1a..0x24`, `0xbf`, `0xc1` |
| `+0x78` | `+0x208` | act_state: `ForceScript__8C_entityFbb` and `ForceAI__8C_entityFUiUiUif` test 3 (shock) there and store 0, as `entity_force_script` and `entity_force_ai` do at `+0x78` |
| `+0x98` / `+0x9c` / `+0xac` | `+0x224` / `+0x228` / `+0x238` | - |
| `+0xbc..+0x1e3` (anim_machine) | `+0x250` | - |
| `+0x1e4` / `+0x1e5` | `+0x390` / `+0x391` | `human_do_crouched` ↔ `Do_Crouched__7C_humanFb`; `human_do_aimed` ↔ `Do_Aimed__7C_humanFb` (line-for-line; shared float constant `0x40800000` = 4.0f) |
| `+0x1e8` | `+0x394` | the same `+4` delta as `+0x1e4` ↔ `+0x390`. `Do_Shoot__7C_human` switches on it (cases 0/5/6/7), the value set of `human_update`'s first switch (§2c); the corner-fire refusal on 0 or 2 is the same gate on both builds |
| `+0x1ec` | `+0x398` | `Update__7C_humanFUi` gates `Shooting__7C_humanFv` on the same byte |
| `+0x1f0` / `+0x1f4` | `+0x39c` / `+0x3a0` | the PS2 names the three tables the pointer may hold: `g_fire_action_ready` = `_fire_action_ready_`, `g_fire_action_aimed` = `_fire_action_aimed_`, `g_fire_action_onoff` = `_fire_action_onoff_` |
| `+0x1fe` | `+0x3aa` | - |
| `+0x200..+0x208` (shot target) | - | the PS2's own `+0x200` is the anim slot (the first row of this table); do not pair them |
| `+0x218` / `+0x21c` / `+0x22c` / `+0x22d` / `+0x22e` / `+0x230` | `+0x3b8` / `+0x3bc` / `+0x3cc` / `+0x3cd` / `+0x3ce` / `+0x3d0` | `Do_Shoot__7C_humanFbP8S_vector` keeps the throw start time and flag at `+0x3b8` / `+0x3bc`; `Movement__7C_humanFUi` tests `+0x3ce` in the same idle gate |
| `+0x23c` / `+0x240` | `+0x3dc` / `+0x3e0` | the PS2 knock-down clip base is `0x1b` |
| `+0x40c` | `+0x588` | - |
| `+0x418` | `+0x594` | `Movement__7C_humanFUi` / `MovementIntern__7C_humanFUi`: the same `dt * 9.81` integration, the same 0.05 seed and the same kill above 50.0 |
| `+0x43c` | `+0x5a0` | - |
| `+0x44c` | `+0x5a4` | `SetWaitAnim__7C_humanFv` stores the clip time - 250 there with mode 5; `Movement__7C_humanFUi` counts it down in mode 5 |
| `+0x478` | `+0x5c4` | `human_is_this_car_stealing` ↔ `IsThisCarStealing__7C_human` |
| `+0x47c` | `+0x5c8` | - |
| `+0x480..+0x563` (G_Inventory) | `+0x5cc` | delta `0x14c`; four FIXED slot bands on the PC where the PS2 uses one vector |
| `+0x564` / `+0x568` / `+0x56c` / `+0x570` | `+0x5f0` / `+0x5f4` / `+0x5f8` / `+0x5fc` | `human_repair_weapon_position` ↔ `RepairWeaponPosition__7C_human` |
| `+0x580` | `+0x60c` | - |
| `+0x590` / `+0x594` | `+0x61c` / `+0x620` | `FireReload__7C_humanFP15S_GameItemTable` stores rand + 5000 and -1 there, as its inlined PC copy does |
| `+0x5fc` | `+0x660` | both builds recover the type index by counting trailing zeros |
| `+0x604` (max_health) | `+0x668` | `__ct__7C_humanFv` initialises it to 100.0f |
| `+0x644` (cur_health) | `+0x6a8` | `HitLanding`, `HitFall`, `Do_Castrate`, `Do_HardHead` and `_intern_ForceDeath` all subtract from it and test `< 0.01` |
| `+0x688` / `+0x68c` / `+0x690` / `+0x694` | `+0x6ec` / `+0x6f0` / `+0x6f4` / `+0x6f8` | - |
| `+0x6d8` / `+0x6dc` | `+0x734` / `+0x738` | both builds' `WINGMAN_SETINDICATOR` arm stores the pair there; `UpdateDynColl__7C_humanFv` reads `+0x734` to refresh the slot |
| `+0x6f8` / `+0x700` | `+0x754` / `+0x75c` | - |
| `+0x724..+0x757` (ai_loc) | `+0x780` | delta `0x5c` |
| `+0x758..+0x99b` (ai_blob_log) | `+0x7c0` | delta `0x68` — it differs from `ai_loc`'s `0x5c`, so this band does not rebase by a constant |
| `+0xa58` | `+0xb8c` | the same init and countdown, and no positive writer on either build |

### 7b. PS2 methods with no PC function

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0017fe50 | `IsKickToCocotus__7C_humanFv` | 1168 B: kick a dog or a downed human within 1.5 units. Its three calls guard the melee arms of `Do_Shoot`; the PC melee arms are otherwise statement-aligned and carry no such test |
| 0x00183b90 | `Do_WeaponDropAll__7C_humanFv` | 680 B: drop every item of a dying non-player. Its one caller is `Hit`; at that position `human_hit` calls `human_weapon_drop` for every human |
| 0x00189710 | `Turn_Left__7C_humanFv` | 48 B: no caller and no data reference on the PS2, no slot in the 39-slot PC vtable, and its turn rate ±2.4 occurs nowhere in `Game.exe` as a float or a double |
| 0x00189740 | `Turn_Right__7C_humanFv` | the same |
| 0x0018a7c0 | `PickObject__7C_humanFR8S_vector` | an empty 8-byte non-virtual body, called directly; it has no slot in either vtable |
| 0x0018a7d0 | `UniversalUse__7C_humanFv` | the same |
| 0x0018e100 | `HideLegs__7C_humanFP9I3D_modelb` | the same |
| 0x0018e230 | `ChangeSceneInit__7C_humanFv` | city streaming (below); PS2 slot 27, and the PC vtable has no slot for it |
| 0x0018e2b0 | `ChangeSceneDone__7C_humanFv` | city streaming; PS2 slot 28 |
| 0x0018e410 | `SchovejPero__7C_humanFv` | city streaming: the model teardown across a city-part load (`I3D_pose::Release` ×3, `StopAnims`, `EraseDynColls`, `LightnessDel`), 980 B |
| 0x0018e920 | `VystrcPero__7C_humanFv` | city streaming: the model rebuild after the load (`_intern_ChangeModel`, `InvalidateSelf`), 2568 B |

**City streaming.** Every caller of the last four is the PS2 city-streaming subsystem
(`InterCity_ChangeMission__9C_mission`, `ChangeScene*__10C_shvestky`,
`InternMission_Vykur/Spolkni`). The PS2 build splits the city into parts with a load between them
to fit 32 MB of RAM and the PC build does not: `InterCity` appears 127 times in the PS2 decomp and 0
times on the PC. The bodies are large, so their absence is not inlining; for the two `ChangeScene*`
methods the PC vtable settles it: the PC `C_human` table maps slot for slot onto the PS2 one through
`SoundsChange` and then holds one more slot, the two-argument network-event virtual, with no slot
for either. *SchovejPero* / *VystrcPero* are crude developer slang, roughly "put it away" / "stick it
out". A PS2 method count is an upper bound on the PC class, never a target.

### 7c. The inlined methods

The ten `inlined` rows of §3 have no PC function: MSVC spliced their bodies into the host.

* **The `Hit*` family** (`HitLanding`, `HitAgony`, `HitStroke`, `HitGun`, `HitGunHPT`, `HitFall`).
  PS2 `Hit` calls `NewScream` zero times — all its scream calls live inside the helpers — yet
  `human_hit` calls `human_new_scream` four times and `human_movement` four times directly; and PS2
  `ThrowFromCarTruck` calls `HitFall` while `human_throw_from_car_truck` calls no unnamed function,
  only the same internals. `human_hit` keeps the PS2 dispatch order (landing, agony, fall, stroke,
  else gun) and every arm's clip ids are the PS2 ids plus a constant band offset (`+0x22` for the
  landing and fall clips, `+0x32` for the stroke clips). The five helpers hold ten scream calls and
  `human_hit` four because MSVC merged the identical scream tails into three shared tails.
* **`SetWaitAnim` + `SetFreeBoxingAnim`** sit in `human_dispatch_weapon_action_at_target` at every
  PS2 call position with no intervening call: the wait block (clip time - 250 ms into `+0x44c`,
  motion zeroed, mode 5) and the extended-clip block (end time into `+0x234`,
  `anim_machine_set_extended_anim(clip, 1.0, 4.0, 1)`).
* **`FireReload`**: its one PS2 caller is `AI` (animation event 1), and the PC event-1 arm of
  `human_ai` carries the body statement for statement (`+0x1fc` set, the in-car refusal, the aim
  dropped, `+0x590` = rand + 5000, `+0x594` = -1, the standing or crouched reload clip,
  `anim_machine_set_extended_anim(clip, 1.0, 4.0, id != 8)`).
* **`PlaySoundFromSndTable`**: its two PS2 callers are `StartWeaponEffect` and `FireReload`; the
  frame-flag rewrite `& 0xfffffedf | 0x40000000` that lives inside the PS2 body appears directly in
  `human_start_weapon_effect` with no intervening sound call.

### 7d. Notes on single twins

* `human_change_dead_anim` (STRUCTURAL): picks the carry-a-corpse clip matching the gait (Walk /
  Back / Left / Right / StrafL / StrafR, plus Nahozeni = heave up, Shozeni = put down, Stat); gated
  on `+0x700` and `+0x704`. The PS2 body is a 28-byte `return;` stub, so the twin is a symbol match
  and can never be a body match.
* `human_change_model` caches the original model's basename at `+0x5f8` while it is 0, invalidates the actor, drops
  the shadow frames and the fake shadow, stops the clips, erases the dyn-colls, opens the
  replacement and runs the subclass half: `player_change_model` for the active player,
  `entity_rebind_neck_frame` for kind `0x1b`. Matched from the caller side: the identical
  `strrchr(name, 0x5c)` head and teardown order, with `+0x68` ↔ `+0x70` and `+0x5f8` ↔ `+0x65c` at
  the proven `0x64` band delta.
* `human_force_death` is separate from the `Death` virtual: `game_process_death_actors`
  (`DeathActorProcess__6C_game`) calls it in the exact `_intern_ForceDeath` position on both builds.
  It ends by setting the anim-machine lock (`structs/anim_machine.md` §2a).
* `human_can_aim`: the frame world position at `y + 1.5`, the forward direction, one horizontal
  line test on mask `0x80000`, `return hit == 0`. The PS2 callers `AI__8C_player` and
  `EineMeineKleineAutoInLuft` both appear on the PC side. The PS2 caches the answer behind
  `Can_Aimed(1000)` and lowers the probe to 0.8 when crouched; the PC does neither.
* `human_do_shock_up` / `human_do_shock_stumble`: a body diff cannot tell them apart, and the ids are
  rebased per build. The id spacing is build-invariant: PS2 `ShockUp` accepts `0x1c` / `0x1f` /
  `0x22` (+ 3) like PC `0x3e` / `0x41` / `0x44`, PS2 `ShockStumble` `0x24` / `0x26` / `0x28` (+ 2)
  like PC `0x46` / `0x48` / `0x4a`.
* `human_get_actor_speed` sits about 380 KB below the unit: MSVC emits per translation unit, and the
  override was instantiated in the `C_actor` unit.
* The script setters of §3b have no PS2 symbol: the PS2 build inlines each into its own `Process`
  arm. Their PS2 offsets are in §7a.

## 8. Related

- `structs/actor.md` — the `C_actor` base (kind 1).
- `structs/entity.md` — the `C_entity` AI layer above; `structs/entity_kind.md` — the kind enum.
- `structs/player.md` — the `C_player` layer above.
- The embedded sub-objects: `structs/anim_machine.md` (`+0xbc`), `structs/shot_skeleton.md`
  (`+0x2dc`), `structs/inventory.md` (`+0x480`), `structs/ai_loc.md` (`+0x724`),
  `structs/ai_blob_log.md` (`+0x758`).
- `structs/grenade.md` — what `human_notify_callback` throws.
- `structs/speech.md` — the speech channel's eye and neck targets, `+0x588` and `+0x58c`.
- `structs/using_object.md` — the corpse's two records, `+0x9c4` and `+0x9fc`.
- `basic/artifacts/hud_elements.md` §7 — the wingman health slots.

## 9. Open items

1. **`S_human_clip_step.unk_0c`.** Every writer read stores 0, and `human_movement` passes it as an
   argument of `human_animation_evaluate_and_dispatch`. What a non-zero value would do is not decoded.
2. **The length of `clip_steps`.** Five steps is the space up to `+0x2a0`. Direct stores reach step 2's
   `code` at `+0x264`, and `+0x268..+0x29f` has no access in `human_*`, `player_*` or `entity_*`. Hits
   at those displacements in other functions were counted and not read one by one.
3. **`human_get_anim_id_for_death`'s EDX argument**, which selects the second branch of §2c, was not
   traced.
