# `C_animations_machine_de_luxe` — the animation machine (`anim_machine_`)

**Created** 2026-07-13 11:22 +0200 · **Modified** 2026-11-16 12:35 +0100

The engine's per-object **animation state machine**: it drives two blended full-body clips on one
model, one overlay clip, and an optional position track that moves the owner, and it owns the static table of
loaded tracks. It is a **sub-object**, not a scene actor: it has no `kind` tag and no vtable of its own. It sits
under both the human animation path and the traffic / pedestrian path, so a function that looks like `C_human`
animation code may belong to this class instead.

| | |
|---|---|
| PC prefix | `anim_machine_` for the machine, `anim_track_` for the track record and its table |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x128` |
| PC constructor | `anim_machine_ctor` |
| PC translation unit | `[0x005aaa50, 0x005adf80)`, guard to guard |
| PS2 class | `C_animations_machine_de_luxe` (tag `28C_animations_machine_de_luxe`), 24 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. `C_animations_machine_de_luxe`

```c
typedef struct C_animations_machine_de_luxe {  // 0x128 bytes; embedded or heap (section 4)
    /* +0x000 */ C_actor  *owner_actor;        // [H] set by the actor Init; a track writes its placement to this actor
    /* +0x004 */ C_traffic_element *owner_element; // [H] set by the element Init; used when owner_actor is null
    /* +0x008 */ I3D_model *model;             // [H] every clip call is a vtable call on it; null makes the machine inert
    /* +0x00c */ uint8_t   stage0_active;      // [H] clip slot 0 holds a clip
    /* +0x00d */ uint8_t   stage0_stop_at_full_blend; // [H] stop slot 0 too when the blend reaches 1.0
    /* +0x00e */ uint8_t   stage0_play_once;   // [H] the tick stops slot 0 when its clip has ended
    /* +0x00f */ uint8_t   stage1_active;      // [H] clip slot 1 holds a clip
    /* +0x010 */ uint8_t   stage1_stop_at_zero_blend; // [H] stop slot 1 too when the blend reaches 0.0
    /* +0x011 */ uint8_t   stage1_play_once;   // [H] the tick stops slot 1 when its clip has ended
    /* +0x012 */ uint8_t   blending;           // [H] a crossfade between slots 0 and 1 is running
    /* +0x014 */ float     blend_weight;       // [H] weight of slot 0, 0.0..1.0; slot 1 takes the rest
    /* +0x018 */ float     blend_rate;         // [H] weight change per second; negative while slot 1 comes up
    /* +0x01c */ uint32_t  active_stage;       // [H] the slot the newest clip went to, 0 or 1
    /* +0x020 */ float     start_time_override; // [H] above zero it replaces SetAnim's start time and is reset to -1.0; only -1.0 is stored
    /* +0x024 */ uint32_t  stage0_caller_value; // [H] SetAnim's eleventh argument for a clip in slot 0; no reader (class census)
    /* +0x028 */ uint32_t  stage1_caller_value; // [H] SetAnim's eleventh argument for a clip in slot 1; no reader (class census)
    /* +0x02c */ int32_t   track;              // [H] index of the running position track in the table, -1 = none
    /* +0x030 */ float     track_origin[3];    // [H] world position the track's keys are relative to
    /* +0x03c */ uint32_t  track_time_ms;      // [H] time into the running track, clamped to its duration
    /* +0x040 */ float     track_basis[16];    // [H] the rotation the track's keys are turned by; row 3 is zeroed
    /* +0x080 */ int32_t   old_track;          // [H] the track being faded out, -1 = none
    /* +0x084 */ float     old_track_weight;   // [H] 0.0..1.0, the new track's share of the crossfade
    /* +0x088 */ float     old_track_rate;     // [H] crossfade progress per ms, 1 / the old track's remaining time
    /* +0x08c */ uint32_t  old_track_time_ms;  // [H] time into the old track
    /* +0x090 */ float     old_track_origin[3]; // [H] track_origin of the old track
    /* +0x09c */ float     old_track_basis[16]; // [H] track_basis of the old track
    /* +0x0dc */ uint8_t   drift_active;       // [H] the owner is being moved by drift_offset
    /* +0x0e0 */ float     drift_rate;         // [H] drift progress per ms
    /* +0x0e4 */ float     drift_progress;     // [H] 0.0..1.0
    /* +0x0e8 */ float     drift_offset[3];    // [H] the gap between the old clip's exit point and the new one's entry point
    /* +0x0f4 */ uint8_t   ext_active;         // [H] the overlay clip in slot 2 is live
    /* +0x0f5 */ uint8_t   ext_blending;       // [H] its weight is moving
    /* +0x0f8 */ float     ext_weight;         // [H] its current weight
    /* +0x0fc */ float     ext_target_weight;  // [H] the weight it moves to; 0.0 while blending out
    /* +0x100 */ float     ext_rate;           // [H] weight change per second; negative while blending out
    /* +0x104 */ uint8_t   ext_play_once;      // [H] remove the overlay when ext_remaining_ms runs out
    /* +0x108 */ int32_t   ext_remaining_ms;   // [H] countdown of a play-once overlay
    /* +0x10c */ uint8_t   ext_blocks_actions; // [H] the overlay is a stroke or a hit reaction: with ext_active it refuses new actions
    /* +0x10e */ int16_t   ext_tag;            // [H] the caller's label for the overlay; -1 = none (section 2a)
    /* +0x110 */ uint8_t   locked;             // [H] 1 = frozen: every animation entry point returns at once
    /* +0x114 */ float     track_offset[3];    // [H] added to the placement a track computes; a human's movement accumulates here
    /* +0x120 */ float     height_ramp;        // [H] height a track adds in proportion to clip progress
    /* +0x124 */ uint8_t   height_ramp_invert; // [H] 1 = the ramp runs from full to zero
    /* +0x125 */ uint8_t   height_ramp_stage;  // [H] the clip slot whose progress drives the ramp
} C_animations_machine_de_luxe;
```

**Size `0x128`**, measured at every heap site: `pedestrian_manager_add_pedestrian`, `railway_game_init` ×2 and
`railway_add_person` each do `operator_new(0x128)` followed by `anim_machine_ctor`.

**The arrays.** `track_origin`, `old_track_origin` and `drift_offset` are one world vector each, x y z.
`track_basis` and `old_track_basis` are the 4x4 matrix `anim_machine_recompile_matrix` and `anim_machine_set_anim`
copy as 16 dwords from `model+0x50`; they then zero `+0x70`, `+0x74`, `+0x78`, the translation row, so only the
rotation and scale apply to the keys. `track_offset` is one vector, x y z.

**Clip slots.** The model has eight clip slots: `anim_machine_stop_anims` stops `i = 0..7`. The machine itself uses
three: slots 0 and 1 crossfade, slot 2 is the overlay (`anim_machine_set_extended_anim`,
`anim_machine_del_extended_anim`).

**The vtable calls on `model`:** `+0x14` the animation-set node lookup, `+0x18` the placement update, `+0x60`
attach a clip to a slot, `+0x64` a slot's clip, `+0x68` stop a slot, `+0x70` set a slot's weight, `+0x78` set a slot's
time, `+0x7c` a slot's time, `+0x88` is a slot still playing.

### 1b. `S_anim_track` — one element of the track table

```c
typedef struct S_anim_track {                  // 100 bytes
    /* +0x00 */ char      name[0x20];          // [H] the `.tck` file name, the lookup key
    /* +0x20 */ uint32_t  duration_ms;         // [H] file data; clamps track_time_ms
    /* +0x24 */ uint32_t  key_interval_ms;     // [H] file data; time / this = the key index
    /* +0x28 */ float     entry_point[3];      // [M] file data; subtracted from the previous clip's exit_point
    /* +0x34 */ float     exit_point[3];       // [M] file data; the minuend of that subtraction
    /* +0x40 */ uint8_t   has_direction_keys;  // [H] the second key list is not empty
    /* +0x44 */ uint8_t   position_keys_alloc; // [H] MSVC's empty allocator of the first vector
    /* +0x48 */ float   (*position_keys_begin)[3]; // [H] the position keys, stride 0xc
    /* +0x4c */ float   (*position_keys_end)[3];   // [H] count = (end - begin) / 0xc
    /* +0x50 */ float   (*position_keys_cap)[3];   // [H] one past the allocation
    /* +0x54 */ uint8_t   direction_keys_alloc; // [H] MSVC's empty allocator of the second vector
    /* +0x58 */ float   (*direction_keys_begin)[3]; // [H] the direction keys, stride 0xc
    /* +0x5c */ float   (*direction_keys_end)[3];   // [H] count = (end - begin) / 0xc
    /* +0x60 */ float   (*direction_keys_cap)[3];   // [H] one past the allocation
} S_anim_track;
```

**The arrays.** `name` holds a file name, `0x20` because `anim_machine_load_track` copies up to the NUL into that
space. `entry_point` and `exit_point` are one vector each, read in that order from the file by a `0xc`-byte read each.

## 2. Field evidence

Cells come from a census of `anim_machine_*` over the machine's own receiver, read against the bodies. The owners'
accesses through `human+0xbc + N` that were read in this document are added by hand; the rest of them are Open item 1.

### 2a. `C_animations_machine_de_luxe`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | owner_actor | H | `anim_machine_ctor`, `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element` | `anim_machine_tick` | `load_track` is not a user: its ECX is the name |
| +0x4 | owner_element | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element` | `anim_machine_tick` | - |
| +0x8 | model | H | `anim_machine_ctor`, `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element` | `anim_machine_del_extended_anim`, `anim_machine_get_active_anim_time`, `anim_machine_recompile_matrix`, `anim_machine_set_anim`, `anim_machine_set_extended_anim`, `anim_machine_stage_blending_tick`, `anim_machine_stop_anims`, `anim_machine_sync_track_time`, `anim_machine_tick` | `actor+0x68`, `element+0x40`, or the argument, by Init overload |
| +0xc | stage0_active | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | - |
| +0xd | stage0_stop_at_full_blend | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_stage_blending_tick`, `anim_machine_tick` | SetAnim's third argument when the clip goes to slot 0 |
| +0xe | stage0_play_once | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_tick` | SetAnim's play-once argument; forced to 1 when the call starts a track |
| +0xf | stage1_active | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | - |
| +0x10 | stage1_stop_at_zero_blend | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_stage_blending_tick`, `anim_machine_tick` | SetAnim's third argument when the clip goes to slot 1 |
| +0x11 | stage1_play_once | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_tick` | as `stage0_play_once` |
| +0x12 | blending | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_is_active`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_is_active` is true while it is set |
| +0x14 | blend_weight | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | starts at 0.05 going up or 0.95 going down; the first 0.5 test picks which clip SetAnim drops |
| +0x18 | blend_rate | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_stage_blending_tick`, `anim_machine_tick` | SetAnim's float argument, negated for slot 1 |
| +0x1c | active_stage | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_get_active_anim_time`, `anim_machine_set_anim`, `human_animation_evaluate_and_dispatch`, `human_intern_throw_self_from_car`, `human_use_actor` | the selector `anim_machine_get_active_anim_time` passes to the model; `human_use_actor` and two more copy it to `height_ramp_stage` |
| +0x20 | start_time_override | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_set_anim` | a positive value replaces SetAnim's start-time argument and is reset to -1.0; every writer stores -1.0, so that arm has no located trigger; no `this` hit at `human+0xdc` |
| +0x24 | stage0_caller_value | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_set_anim` | - | SetAnim's eleventh stack argument, which the decomp's signature drops (`mov eax, [esp+0x94]`, store at `0x005ab8b5`); 10 of the 11 callers pass -1, `entity_process_command` passes the return value of `entity_force_ai`; no reader (class census; no `this` hit in the `C_human` band) |
| +0x28 | stage1_caller_value | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_set_anim` | - | as `stage0_caller_value`, for slot 1 (store at `0x005ab844`) |
| +0x2c | track | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim` | `anim_machine_is_active`, `anim_machine_set_anim`, `anim_machine_tick`, `pedestrian_set_anim` | `anim_machine_get_track` or `anim_machine_load_track`'s result; also read through the pointer by `pedestrian_set_anim` |
| +0x30 | track_origin | H | `anim_machine_tick` | `anim_machine_recompile_matrix`, `anim_machine_set_anim`, `anim_machine_tick` | the model's position when no old track runs, else the old track's end key added to `old_track_origin` |
| +0x3c | track_time_ms | H | `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_tick` | - |
| +0x40 | track_basis | H | `anim_machine_recompile_matrix`, `anim_machine_set_anim` | `anim_machine_recompile_matrix`, `anim_machine_set_anim`, `anim_machine_tick` | built from `model+0x50`; SetAnim's vector argument scales rows 0..2 |
| +0x80 | old_track | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_tick` | set by SetAnim's blend-out argument when at least 100 ms of the running track remain |
| +0x84 | old_track_weight | H | `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_tick` | - |
| +0x88 | old_track_rate | H | `anim_machine_set_anim` | `anim_machine_tick` | - |
| +0x8c | old_track_time_ms | H | `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_set_anim`, `anim_machine_tick` | - |
| +0x90 | old_track_origin | H | `anim_machine_set_anim` | `anim_machine_recompile_matrix`, `anim_machine_set_anim`, `anim_machine_tick` | SetAnim adds `height_ramp` to its y |
| +0x9c | old_track_basis | H | `anim_machine_set_anim` | `anim_machine_set_anim`, `anim_machine_tick` | copied from `track_basis` |
| +0xdc | drift_active | H | `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_tick` | - |
| +0xe0 | drift_rate | H | `anim_machine_set_anim` | `anim_machine_tick` | - |
| +0xe4 | drift_progress | H | `anim_machine_set_anim`, `anim_machine_tick` | `anim_machine_tick` | - |
| +0xe8 | drift_offset | H | `anim_machine_set_anim`, `anim_machine_set_anim` (by address, `S_vector::operator*=`) | `anim_machine_set_anim`, `anim_machine_tick` | §7: the no-track branch takes it from the `blnd` node |
| +0xf4 | ext_active | H | `anim_machine_del_extended_anim`, `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_extended_anim`, `anim_machine_tick` | `anim_machine_del_extended_anim`, `anim_machine_is_active`, `anim_machine_tick`, `dog_acquire_player_target`, `dog_hit`, `dog_return_to_patrol`, `entity_process_command` | `dog_acquire_player_target`, `dog_hit` and `dog_return_to_patrol` read `dog+0x1dc`, the same byte; `entity_process_command` reads it as `human+0x1b0` |
| +0xf5 | ext_blending | H | `anim_machine_del_extended_anim`, `anim_machine_set_extended_anim`, `anim_machine_tick` | `anim_machine_is_active`, `anim_machine_tick` | - |
| +0xf8 | ext_weight | H | `anim_machine_set_extended_anim`, `anim_machine_tick` | `anim_machine_tick` | starts at 0; passed to the model's `+0x70` on slot 2 |
| +0xfc | ext_target_weight | H | `anim_machine_del_extended_anim`, `anim_machine_set_extended_anim` | `anim_machine_tick` | SetExtendedAnim's weight; `anim_machine_del_extended_anim` stores 0 |
| +0x100 | ext_rate | H | `anim_machine_del_extended_anim`, `anim_machine_set_extended_anim` | `anim_machine_tick` | `anim_machine_del_extended_anim` stores the negated blend time |
| +0x104 | ext_play_once | H | `anim_machine_del_extended_anim`, `anim_machine_set_extended_anim` | `anim_machine_is_active`, `anim_machine_tick` | - |
| +0x108 | ext_remaining_ms | H | `anim_machine_set_extended_anim`, `anim_machine_tick` | `anim_machine_tick` | clip length - 250, or half the length when shorter; at zero the tick removes the overlay with blend 4.0 |
| +0x10c | ext_blocks_actions | H | `anim_machine_set_extended_anim` (0), `human_dispatch_weapon_action_at_target` (1), `human_hit` (1) | `entity_ai`, `entity_is_tickable`, `human_can_special_stroke`, `human_do_throw_grenade`, `human_movement`, `human_update`, `player_ai`, `player_ai_drive` | reached as `human+0x1c8`; the 1 is stored right after `anim_machine_set_extended_anim`. Every reader tests it with `ext_active` as the "can act" condition, also `entity_actions_tick`, `entity_beware_cross_fire`, `entity_dispatch_event`, `entity_process_command`, `entity_script_tick`, `human_do_castrate`, `human_do_hard_head`, `player_ai_car_fire` |
| +0x10e | ext_tag | H | `anim_machine_del_extended_anim`, `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `anim_machine_set_extended_anim`, `human_ai`, `pedestrian_manager_generate_passenger_for_taxi` | `human_ai`, `pedestrian_end_taxi_wave` | ¹ |
| +0x110 | locked | H | `anim_machine_init_from_actor`, `anim_machine_init_from_model`, `anim_machine_init_from_traffic_element`, `anim_machine_reset`, `human_die_in_exploding_car`, `human_force_death` | `anim_machine_del_extended_anim`, `anim_machine_is_active`, `anim_machine_set_anim`, `anim_machine_set_extended_anim`, `anim_machine_stage_blending_tick`, `anim_machine_tick` | ² |
| +0x114 | track_offset | H | `anim_machine_set_anim`, `human_update` | `anim_machine_tick`, `human_update` | `human_update` adds its movement correction to x and z |
| +0x120 | height_ramp | H | `anim_machine_set_anim`, `human_animation_evaluate_and_dispatch`, `human_intern_throw_self_from_car`, `human_use_actor` | `anim_machine_set_anim`, `anim_machine_tick` | SetAnim adds it to `old_track_origin.y` when a new track fades in |
| +0x124 | height_ramp_invert | H | `human_animation_evaluate_and_dispatch`, `human_intern_throw_self_from_car`, `human_use_actor` | `anim_machine_tick` | the tick's progress becomes 1 - progress |
| +0x125 | height_ramp_stage | H | `human_animation_evaluate_and_dispatch`, `human_intern_throw_self_from_car`, `human_use_actor` | `anim_machine_tick` | progress = the slot's time / (its clip length - 240) |

¹ A label callers use to recognise their own overlay; the machine only clears it. `human_ai` writes `0xc` (`human+0x1ca`) and
tests for it; `pedestrian_manager_generate_passenger_for_taxi` writes `-0x2357` and `pedestrian_end_taxi_wave` removes the
overlay when it finds that value.

² Set to 1 only when a `C_human` dies outside the normal damage path: `human_force_death` plays the death clip directly on the
I3D frame and then writes `human+0x1cc` (= `+0xbc + 0x110`); `human_die_in_exploding_car` makes the same three stores
(`+0x644` = 0, `+0x5d` = 0, the lock = 1). From then on the six readers return without acting, so the death pose is never
overwritten. The three `Init`s and `Reset` clear it.

### 2b. `S_anim_track`

Every element is read as `g_anim_track_table_begin + index * 100`, so these cells come from the bodies, not the census.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `anim_machine_load_track`, `anim_track_assign`, `anim_track_construct_copy` | `anim_machine_get_track`, `anim_machine_load_track` | compared with `crt_stricmp` |
| +0x20 | duration_ms | H | `anim_machine_load_track`, `anim_track_assign`, `anim_track_construct_copy` | `anim_machine_get_track_mixing_point`, `anim_machine_set_anim`, `anim_machine_tick` | at most 250 ms and `anim_machine_get_track_mixing_point` returns false |
| +0x24 | key_interval_ms | H | `anim_machine_load_track`, `anim_track_assign`, `anim_track_construct_copy` | `anim_machine_get_track_mixing_point`, `anim_machine_tick` | - |
| +0x28 | entry_point | M | `anim_machine_load_track`, `anim_track_assign`, `anim_track_construct_copy` | `anim_machine_set_anim` | `drift_offset` = old `exit_point` - new `entry_point`; the names are from that subtraction, Open item 2 |
| +0x34 | exit_point | M | `anim_machine_load_track`, `anim_track_assign`, `anim_track_construct_copy` | `anim_machine_set_anim` | also taken alone when the new clip has no track |
| +0x40 | has_direction_keys | H | `anim_machine_load_track` | `anim_machine_get_placement_from_track`, `anim_machine_tick` | `count > 0` of the second list |
| +0x44 | position_keys_alloc | H | - | - | an empty class; no access |
| +0x48 | position_keys_begin | H | `anim_machine_load_track`, `anim_track_construct_copy`, `anim_track_dtor` | `anim_machine_get_placement_from_track`, `anim_machine_get_track_mixing_point`, `anim_machine_set_anim`, `anim_machine_tick` | - |
| +0x4c | position_keys_end | H | `anim_machine_load_track`, `anim_track_construct_copy`, `anim_track_dtor` | `anim_machine_get_placement_from_track`, `anim_machine_get_track_end_point`, `anim_machine_get_track_mixing_point`, `anim_machine_set_anim`, `anim_machine_tick` | the last key is the track's end point |
| +0x50 | position_keys_cap | H | `anim_track_construct_copy`, `anim_track_dtor` | - | - |
| +0x54 | direction_keys_alloc | H | - | - | an empty class; no access |
| +0x58 | direction_keys_begin | H | `anim_machine_load_track`, `anim_track_construct_copy`, `anim_track_dtor` | `anim_machine_set_anim`, `anim_machine_tick` | the tick writes the interpolated key to `actor+0x30`, `element+0x1c`, or the model's `+0xc` |
| +0x5c | direction_keys_end | H | `anim_machine_load_track`, `anim_track_construct_copy`, `anim_track_dtor` | `anim_machine_set_anim` | - |
| +0x60 | direction_keys_cap | H | `anim_track_construct_copy`, `anim_track_dtor` | - | - |

## 3. Method family

All 24 PS2 methods are accounted for: 21 own PC functions, 2 inlined, 1 folded. 21 PC-only rows are the CRT quartet, the
guard pair, the track record's helpers and the shared helpers the unit holds.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005aaa50 | `crt_static_init_guard_005aaa50` | - | - | - | - | The unit's opening guard |
| 0x005aaa70 | `crt_atexit_register_005aaa70` | - | - | - | - | The guard's `atexit` helper |
| 0x005aaa80 | `anim_track_table_static_init` | `g_anim_track_table` | - | - | - | First of the quartet |
| 0x005aaa90 | `anim_track_table_static_ctor` | `g_anim_track_table` | - | - | - | Second of the quartet |
| 0x005aaab0 | `anim_track_table_register_atexit_dtor` | `g_anim_track_table` | - | - | - | Third of the quartet |
| 0x005aaac0 | `anim_track_table_static_dtor` | `g_anim_track_table` | - | - | - | Fourth: destroys every record and frees the buffer |
| 0x005aab20 | `anim_machine_ctor` | instance | `__ct__28C_animations_machine_de_luxeFv` | 0x0011eaa0 | CONFIRMED | Clears `owner_actor` and `model` |
| 0x005aaf80 | `anim_track_dtor` | instance | - | - | - | Frees the two key vectors of one record |
| 0x005ad970 | `anim_track_construct_copy` | instance | - | - | - | Copy constructor of one record |
| - | - | instance | `__dt__28C_animations_machine_de_luxeFv` | 0x0011eab0 | inlined | The destructor is empty: every teardown calls `stub_noop_cdecl_00574a40`, §4a |
| 0x005adaf0 | `anim_track_assign` | instance | - | - | - | `operator=` of one record |

### 3b. Accessors

No function here is a single load or store of one field.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005aab30 | `anim_machine_load_track` | `LoadTrack__28C_animations_machine_de_luxeFPc` | 0x0011eb00 | CONFIRMED | Finds a track by name or loads `anims\%s`, version dword 4, and appends it |
| 0x005aafb0 | `anim_machine_get_track` | `GetTrack__28C_animations_machine_de_luxeFPc` | 0x0011f870 | CONFIRMED | Backwards `crt_stricmp` search of the table; -1 when absent |
| 0x005ab020 | `anim_machine_clear_non_base_tracks` | `ClearNoBaseTracks__28C_animations_machine_de_luxeFv` | 0x0011f930 | CONFIRMED | Pops the table down to `g_anim_base_track_count` (§5); sole caller `game_done` |
| 0x005ab170 | `anim_machine_init_from_actor` | `Init__28C_animations_machine_de_luxeFP7C_actor` | 0x0011fb20 | CONFIRMED | Owner = the actor, model = `actor+0x68`; resets the state |
| 0x005ab1e0 | `anim_machine_init_from_traffic_element` | `Init__28C_animations_machine_de_luxeFP17C_traffic_element` | 0x0011fba0 | CONFIRMED | Owner = the element, model = `element+0x40`; resets the state |
| 0x005ab250 | `anim_machine_init_from_model` | `Init__28C_animations_machine_de_luxeFP9I3D_model` | 0x0011fc20 | CONFIRMED | No owner; resets the state |
| 0x005ab2b0 | `anim_machine_reset` | `Reset__28C_animations_machine_de_luxeFv` | 0x0011fc90 | CONFIRMED | The state reset alone, owners and model kept |
| 0x005ab300 | `anim_machine_sync_track_time` | `EqualTime__28C_animations_machine_de_luxeFii` | 0x0011fce0 | CONFIRMED | Sets one clip slot's time from another's progress |
| 0x005ab3a0 | `anim_machine_get_active_anim_time` | - | - | - | The active slot's current time and clip length |
| 0x005ab3f0 | `anim_machine_stop_anims` | - | - | - | Stops the model's eight clip slots |
| 0x005ab410 | `anim_machine_recompile_matrix` | - | - | - | Transforms both origins by the argument and rebuilds `track_basis` |
| 0x005ab460 | `anim_machine_set_anim` | - | - | - | Starts a clip, its track and the crossfades |
| - | - | `GetAnimBaseDelta__28C_animations_machine_de_luxeFP17I3D_animation_set` | 0x00120b20 | inlined | The no-track branch of `anim_machine_set_anim`: the `blnd` node's position key 0 (`0x0064e43c`, one `.text` reference) |
| 0x005abc80 | `anim_machine_is_active` | `IsActive__28C_animations_machine_de_luxeFv` | 0x00120bb0 | CONFIRMED | False when locked; true while blending, on a track or with a live overlay |
| 0x005abcc0 | `anim_machine_tick` | `Tick__28C_animations_machine_de_luxeFUi` | 0x00120c20 | CONFIRMED | Ends finished clips, advances the blend, applies the tracks, the drift and the overlay |
| 0x005acb70 | `anim_machine_set_extended_anim` | `SetExtendedAnim__28C_animations_machine_de_luxeFP17I3D_animation_setffb` | 0x00121df0 | CONFIRMED | Starts the overlay in slot 2 |
| 0x005acc60 | `anim_machine_del_extended_anim` | - | - | - | Stops the overlay at once, or blends it out |
| 0x005acce0 | `anim_machine_get_placement_from_track` | `GetPlacementFromTCK__28C_animations_machine_de_luxeFPcR8S_vectorR8S_vector` | 0x00121fa0 | CONFIRMED | Offset and direction a track moves its owner by |
| 0x005ace20 | `anim_machine_get_track_end_point` | `GetEndPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector` | 0x00122110 | CONFIRMED | The track's last position key |
| 0x005ace60 | `anim_machine_get_track_mixing_point` | `GetMixingPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector` | 0x00122180 | CONFIRMED | The position key 250 ms before the end |
| 0x005acee0 | `anim_machine_stage_blending_tick` | `StageBlendingTick__28C_animations_machine_de_luxeFUi` | 0x00122220 | CONFIRMED | The blend step of `anim_machine_tick`, alone |
| 0x005ad000 | `anim_track_keys_resize` | - | - | - | Resizes one key vector, stride `0xc` |
| 0x005ad290 | `anim_track_table_size` | - | - | - | `(end - begin) / 100` |
| 0x005ad2c0 | `anim_track_table_insert_at` | - | - | - | `vector::insert` on the table |
| 0x005ad4a0 | `anim_track_table_destroy_range` | - | - | - | Destroys a range of records |
| 0x005ad4d0 | `anim_track_table_uninit_copy` | - | - | - | Copy-constructs a range into raw memory |
| 0x005ad510 | `anim_track_table_uninit_fill_n` | - | - | - | Copy-constructs n records into raw memory |
| 0x005ad540 | `anim_track_table_copy_assign_range` | - | - | - | Assigns a range forwards |
| 0x005ad760 | `anim_track_table_copy_backward` | - | - | - | Assigns a range backwards |
| 0x005df650 | `anim_track_load_by_name` | - | - | - | Strips the extension, appends `.tck`, loads; caller `game_init` |
| 0x005fe200 | `anim_track_table_copy_assign_range_005fe200` | - | - | - | A second copy of the range assign, outside the unit |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005add00 | `std_vectorc_copy_assign` | - | - | - | `vector::operator=`, stride `0xc`, for a record's key lists |
| 0x005adf60 | `vec_capacity_stride_0xc` | - | - | - | `(cap - begin) / 0xc` |

## 4. Where the object lives (six owners)

Established by a scan of `.text` for every `E8` call to the machine's methods, then a back-trace of
the ECX source (`lea ecx,[reg+disp]` = EMBEDDED, `mov ecx,[reg+disp]` = POINTER) with each callsite
attributed to its containing function. The decomp cannot answer this on its own: MSVC passes the
machine in ECX and Ghidra hides the `this` at every callsite.

| Owner | How it holds the machine | Offset | Conf | Evidence |
|---|---|---|---|---|
| **`C_human`** (and its subclasses `C_player` / `C_entity`) | EMBEDDED | **`+0xbc`** | H | 23 distinct owning functions. Worked example — `human_do_play_anim` (`0x004a63f0`): prologue `8b f1` (`mov esi,ecx`, ESI = `this`), then `8d be bc 00 00 00` (`lea edi,[esi+0xbc]`), `8b cf` (`mov ecx,edi`), `call anim_machine_set_anim` |
| **`C_dog`** | EMBEDDED | `+0xe8` | H | 6 owning functions in the `0x0047b800..0x0047f3a0` cluster; the cluster's strings are `hlavapsa_I3D` (*hlava psa* = dog's head), `Datack_I3D` / `Dbark1_i3d` / `Dbreath_i3d` / `Ddeath_I3D` |
| **`C_pumper`** (gas-station attendant) | EMBEDDED | `+0xe0` | H | The band's strings are `_pumpa_*` (*pumpa* = fuel pump), the idle set `Kurak*` (smoker) and `Nuda*` (boredom) — `structs/pumper.md` |
| **`C_traffic_car`'s crew element** (`S_ELEMENT`) | EMBEDDED | `+0x24` | H | `traffic_car_element_ctor` and `traffic_car_adopt_car_with_new_driver` both do `lea ecx,[reg+0x24]; call anim_machine_ctor`; the teardown goes through the adjustor thunk `thunk_stub_noop_cdecl_00574a40` (`add ecx,0x24`) — `structs/traffic_car.md` |
| `C_traffic_element` | POINTER | `+0x08` | H | `pedestrian_end_taxi_wave` / `pedestrian_deactivate` deref `*(elem+8)` |
| `S_rail_human` (the tram passenger, `rail_passenger_`) | POINTER | `+0x08` | H | Built by `railway_game_init` and `railway_add_person`, driven by `rail_passenger_set_anim` from the passenger state at `+0x14` through the table `g_rail_passenger_anims`, destroyed by `rail_passenger_done` — `structs/railway.md` §6b |

**The owner set is closed at six by the constructor's callsites.** `anim_machine_ctor` has exactly
nine `E8` callers: `human_ctor`, `dog_ctor`, `pumper_ctor`, `traffic_car_element_ctor`,
`traffic_car_adopt_car_with_new_driver` (the four embedded owners), `pedestrian_manager_add_pedestrian`
(a heap machine for a `C_traffic_element`), and `railway_game_init` ×2 plus `railway_add_person`
(heap machines for `S_rail_human`). Every heap machine is `operator_new(0x128)` followed by the ctor.

**`pedestrian_set_anim` and `pedestrian_drop_umbrella` deref `this+0x8` as an anim-machine
pointer**, so their `this` is a `C_traffic_element`, not a `C_human`.

### 4a. Construction and teardown

The machine's destructor is empty, so MSVC emits no body for it: every teardown calls the shared
bare `ret` `stub_noop_cdecl_00574a40` with ECX = the machine, and the heap owners then free the
block with `operator_delete`. The receivers are proven at every site: `human_dtor`
(`lea ecx,[esi+0xbc]`), `dog_dtor` (`+0xe8`), `pumper_dtor` (`+0xe0`), three EH unwind funclets
(`this+0xbc` ×2, `this+0xe0`), the crew-element thunk (`+0x24`), and `rail_passenger_done` /
`pedestrian_manager_clear_pedestrians` (the pointer, followed by `operator_delete`). The stub has
30 callers in all, so it is not a symbol of this class.

### 4b. Consistency check on `C_human + 0xbc`

The machine is **`0x128` bytes** (§1a), so embedded at `+0xbc` it spans exactly `+0xbc .. +0x1e4`.
`structs/human.md` documents `C_human` fields at `+0x4c`, `+0x74`, `+0x98`, `+0xac` (all below
`+0xbc`) and then `+0x1e4` / `+0x1e5` / `+0x1e8` — **the first documented field after the machine
is exactly where the machine ends.**

## 5. The static track table

| PC address | name | meaning |
|---|---|---|
| `0x006efdb8` | `g_anim_track_table_dtor_guard` | once-flag for the atexit destructor |
| `0x006efdbc` | `g_anim_track_table` | STL allocator byte |
| `0x006efdc0` | `g_anim_track_table_begin` | records of **100 bytes** each |
| `0x006efdc4` | `g_anim_track_table_end` | - |
| `0x006efdc8` | `g_anim_track_table_cap` | - |
| `0x006efdcc` | `g_anim_base_track_count` | the number of records loaded by the base preload (§6); `anim_machine_clear_non_base_tracks` pops the table down to it |

One record is `S_anim_track` (§1b). Tracks are loaded from `anims\%s`, and **the file's
version dword must be 4**.

**The 10 `anim_track_table_*` / `anim_track_*` container helpers belong to this table**, not to a
shared STL instantiation: each body is byte-unique in `.text`, each has 0 datarefs, an image-wide
`E8`+`E9` census finds no foreign referrer, and — the decisive, body-internal evidence — each calls
this table's own element ctor/dtor (`anim_track_construct_copy`, `anim_track_dtor`) at the 100-byte
record stride. A fold of two classes' vector helpers would require both element types to share a
constructor, i.e. to be one type.

**Three routines empty the table:**

| routine | what it does |
|---|---|
| `engine_shutdown` | `erase(begin, end)`: `anim_track_table_destroy_range` (with `ecx = 0x6efdbc`, the table's own allocator byte) destroys every record; the buffer is kept |
| `anim_machine_clear_non_base_tracks` (sole caller `game_done`) | when the base count is at least 1, pops records down to it; when it is 0 or less, destroys every record and keeps the buffer |
| `anim_track_table_static_dtor` (atexit) | the static vector's destructor: destroys every record and frees the buffer |

## 6. The base preload (`game_init`)

Inside `game_init`'s heavy-init block (`C_game+0x2f9c`, `structs/game.md`), the engine walks
`g_anim_info_table` to its `0xffff` terminator and loads every clip's track with
`anim_track_load_by_name`. A clip name containing `?` is loaded twice, with the `?` replaced by
`L` and then by `R`. While a network session is active the loop ticks it (and the loading screen)
whenever more than 500 ms have passed. Then, only
if `g_anim_base_track_count` is still 0, it is set to the table's record count
(`(end − begin) / 100`, the `0x51eb851f` reciprocal). This is its only writer, so the base set is
set once, by the first heavy init; `dog_load_tracks` runs after the seed, so the dog's tracks sit above
the base mark.

## 7. PC vs PS2

- **`C_human` embeds the machine at PC `+0xbc` ↔ PS2 `+0x250`**; `C_traffic_element` holds the
  pointer at PC `elem+0x08` ↔ PS2 `elem+0x188`. The PS2 `C_traffic_car` crew element holds a
  POINTER and deletes it in `DeactivateElem`, `GameDone` and `VypraskeeeKaaaru`, where the PC embeds
  it at `+0x24`.
- **Size:** PS2 heap machines are `__nw__FUi(0x140)`; PC `0x128`.
- **Field delta:** `ext_tag` PC `+0x10e` ↔ PS2 `+0x12a` (pinned by the shared `-0x2357` magic)
  and `locked` PC `+0x110` ↔ PS2 `+0x12c` — both delta `0x1c`. Do **not** extrapolate that delta to
  other fields; cross-build layouts diverge. The PS2 `SetExtendedAnim` / `DelExtendedAnim` test the
  same lock first, and the PS2 twins `_intern_ForceDeath__7C_humanFv` and
  `EineMeineKleineAutoInLuft__7C_humanFP5C_car` make the same three stores as the PC writers.
- **Three `Init` overloads** exist on both builds (`P7C_actor`, `P17C_traffic_element`,
  `P9I3D_model`). The PS2 also puts `SetAnim` on `C_dog`, `C_pumper` and `S_rail_human`.
- **`tracks__28C_animations_machine_de_luxe` and `base_track__28C_animations_machine_de_luxe` are
  static class members** on the PS2, which is what makes the PC globals at `0x006efdbc` and
  `0x006efdcc` this class's members rather than free data; the PC decomp cannot supply that.
- **`__dt__28C_animations_machine_de_luxeFv`** is only the delete-flag test; its 11 PS2 callers are
  the three embedded owners (flag `-1`) and eight delete sites (flag `1`).
- **`GetAnimBaseDelta`** looks up `"blnd"` (PS2 `0x4aaf58`) with `GetAnimIndex` and reads
  `GetPosKey(index, 0)`; its only caller is `SetAnim`, in the same branch the PC inlines it into.
- **`ClearTracks`** has one caller, `CloseSystem__Fv`, and also frees the buffer. The PS2
  `ClearNoBaseTracks` has only the pop-to-base loop; the PC's clear-everything arm for a base count
  of 0 or less has no PS2 counterpart, and the PS2 has no exit-time destructor for the table.
- **The base preload is `PreloadAnims__6C_gameFv`** on the PS2, called once from `InitSystem__FPvRb`
  under a static once-flag; the PC inlines the same loop and the same seed into `game_init` (§6).
- **Two rows of §3 rest on more than a body match.** `anim_machine_set_anim` is 2073 B against 2812 B and its body is not diffed
  in full; its position is boxed 1:1 by eight anchors, a structural argument, which is why it is STRUCTURAL.
  `anim_machine_is_active` and `anim_machine_tick` fell out of the `human_extern_ai_update` diff: the PS2 calls
  `IsActive(this+0x250)` and, if true, `Tick(this+0x250, t)`; the PC function makes the same two calls with the same guard.
- **`start_time_override` and the stage caller values.** The PS2 `SetAnim` symbol ends `...P8S_vectorbUi`, and its
  body stores `param_12` in the same two places as the PC's eleventh argument. It reads and resets the start-time
  float as the PC does, and no PS2 method of the class stores a positive value there.
- **`anim_track_dtor` has no PS2 counterpart.** The PS2 record holds raw arrays, freed inline by `ClearTracks` and
  `ClearNoBaseTracks`.
- **Emission order.** The PC cluster preserves the PS2 emission order: every located pair comes out in the same relative order.
  Interpolation is only safe between two anchors with exactly one gap: by strict index the run after `SetAnim` would put `Tick` at
  `0x005acb70`, `0xf0` bytes before the `DelExtendedAnim` anchor, while PS2 `Tick` is about `0x11d0` bytes.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0011f9f0 | `ClearTracks__28C_animations_machine_de_luxeFv` | Open-coded in `engine_shutdown`, after the anim-cache clear and before the driver release, the position of the only PS2 call, in `CloseSystem` |

## 8. Related

`structs/human.md` (embeds the machine at `+0xbc`) · `structs/dog.md` (`+0xe8`) · `structs/pumper.md` (`+0xe0`) ·
`structs/traffic_car.md` (the crew element, `+0x24`) · `structs/railway.md` §6b (`S_rail_human`, a pointer at `+0x08`) ·
`structs/game.md` (the heavy-init flag that gates the base preload).

## 9. Open items

| # | Item | State |
|---|---|---|
| 1 | The owners' other accesses | in the `C_human` band only `human+0xdc..+0xe7` was traced to its base (no `this` hit); the rest of `human+0xbc..+0x1e3`, and the `dog+0xe8..`, `pumper+0xe0..` and crew-element `+0x24..` bands, were counted by displacement, and only the accesses named in §2 were read. Each still needs its base proven |
| 2 | `entry_point`, `exit_point` | the names come from the one subtraction that reads them. In the 283 shipped version-4 `.tck` files neither vector equals the first or the last position key, nor (0, 0, 0); y is 0.10..0.11 in 237 files and the two are equal in 18. The keys start near 0, so the vectors are not in the keys' frame; the no-track branch of `anim_machine_set_anim` subtracts the clip's `blnd` node key 0 from `exit_point`, which puts both in the clip's node space. A comparison with the `blnd` node keys of the `.i3d` animations would decide the names |
