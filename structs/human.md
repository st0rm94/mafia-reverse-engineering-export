# `C_human` — the concrete person body (animation / movement / weapons)

**Created** 2026-07-12 10:30 +0200 · **Modified** 2026-10-05 21:59 +0200

The concrete human/person layer, derived from `C_actor`: locomotion, the animation and pose system,
weapons / inventory / combat, hit and death reactions, car entry and exit, and the `C_human`
SaveGame block. `C_entity` (the scriptable-NPC AI layer) and `C_player` derive from it.

| | |
|---|---|
| PC kind (`+0x10`) | **`3`** — `human_ctor` stamps it over the `C_actor` base's `1` |
| PC translation unit | `[0x0048b5e0, 0x004ae800)` — **157 real starts, 0 unnamed** (§4c) |
| PC lifecycle | `human_ctor` `0x0048f430` · `human_dtor` `0x0048f670` · `human_scalar_deleting_dtor` `0x0048f650` |
| PS2 class | `C_human`, mangled tag `7C_human`, **146 methods**, 124 with a PC twin (§4) |

**Coverage of the 146 PS2 methods**, counted off the §4 rows: **123 CONFIRMED · 1 STRUCTURAL ·
10 INLINED · 8 PS2-ONLY · 4 base-stub.**
The field layout and every CONFIRMED twin are body-diffed against the non-stripped PS2 ELF.

**Confidence tags:** **[H]** body-read with a writer and a reader · **[M]** inferred · **[U]** not
observed.

The class's PC implementation is spread over several prefixes — see §3 before reading the method
table. The subject of this document is the PC binary; PS2 material is confined to the twin columns of
§4, the `PS2` column of §2a and the PC ↔ PS2 offset table of §2d.

---

## 1. Field layout (C_human band; relative to the object base)

Only C_human-specific fields verified from read bodies are listed (the `+0x00..` base fields are in
`structs/actor.md`). PC offsets; the PS2 equivalents a body-diff pinned are in §2d.

```c
typedef struct C_human {                    // extends C_actor; PC offsets, C_human band only
    /* +0x30 */ float    orient_vec[3];         // [M] forward/orientation vector; movement scales the velocity channels by it
    /* +0x70 */ uint32_t anim_slot;            // [H] the anim the human is CURRENTLY IN (read, not written, by the
                                             //     shock continuations: human_do_shock_up / _stumble accept only 3
                                             //     specific ids here before advancing).
    /* +0x74 */ uint32_t cur_anim_id;           // [H] the anim id being REQUESTED this frame. Written by the Go_* cohort,
                                             //     the whole Corner family, and the Shock family.
                                             //     KNOWN PC ANIM-ID BANDS (all per-build, §2d):
                                             //       0x2f/0x33 +n   shock start (2 groups 4 apart, n = severity 0..3)
                                             //       0x3d +3n       shock down (n = severity 0..2; 2nd clip at +1)
                                             //       0x3e/0x41/0x44 shock get-up   (spacing +3)
                                             //       0x46/0x48/0x4a shock stumble  (spacing +2)
                                             //       0x4e..0x4f     corner breath
                                             //       0x50..0x52     corner view   (0x52 requires crouched)
                                             //       0x53..0x56     corner fire        (+1 = weapon category 4)
                                             //       0x58..0x59     corner fire aimed  (+1 = weapon category 4)
                                             //       0x83..0x8b     death anims (human_get_anim_id_for_death)
    /* +0x98 */ void    *vehicle_link;          // [H] occupied vehicle (car when *(link+0x10)==4)
    /* +0x9c */ void    *vehicle_link_2;       // [M] SECOND vehicle slot; player_indicator_set_aimed kind-checks BOTH +0x98 and +0x9c
    /* +0xac */ int32_t  seat_index;            // [H] seat index in the occupied vehicle; -1 = none/exiting
    /* +0xbc */ C_animations_machine_de_luxe anim_machine;   // [H] EMBEDDED animation state machine, 0x128 bytes,
                                             //     spanning +0xbc..+0x1e3. Established from the
                                             //     DISASSEMBLY (`lea ecx,[this+0xbc]` before every anim_machine_* call);
                                             //     Ghidra hides that `this` in ECX. See structs/anim_machine.md.
    /* +0x1e4 */ uint8_t crouched;              // [H] CROUCHED stance flag; sole writer human_do_crouched.
                                             //     Downstream: forces walk (not run) in the Go_* cohort, suppresses the
                                             //     category-4-weapon anim variant, and shrinks dyn_coll_objects[0].
    /* +0x1e5 */ uint8_t aimed;                // [H] AIMED stance flag; sole writer human_do_aimed.
                                             //     Raising the weapon also ARMS the aimed-fire block (+0x1f0/+0x1f4).
                                             //     NB both stance flags are force-cleared while carrying an object (+0x6f8/+0x700).
    /* +0x1e8 */ uint32_t weapon_class;         // [H] the in-hand item's WEAPON CLASS, copied from its
                                             //     predmety.def record +0x5c by human_change_weapon_model
                                             //     (§2c): 0 hands, 1 handgun, 2 rifle, 3 Thompson/shotgun,
                                             //     4 throwable, 5 knuckle-duster, 6 knife/bottle, 7 blunt.
                                             //     The crosshair shows while it is in [1,4);
                                             //     **human_do_corner_fire and human_do_corner_fire_aimed REFUSE when
                                             //     it is 0 or 2** — no cover fire bare-handed or with a rifle
    /* +0x1ec */ uint8_t shooting;             // [H] the SHOOTING latch: while set and weapon_class is 1-3 (a
                                             //     firearm), human_update calls human_shooting every frame. Set by
                                             //     human_dispatch_weapon_action_at_target and
                                             //     human_notify_callback; cleared by every interrupting
                                             //     action
    /* +0x23c */ uint32_t shock_clip_armed;    // [H] flag set to 1 by human_do_shock_down alongside the clip below
    /* +0x240 */ uint32_t shock_clip_a;        // [H] the knock-down clip: `severity*3 + 0x3d`
    /* +0x254 */ uint32_t shock_clip_b;        // [H] the second clip staged with it: `severity*3 + 0x3e` (= clip_a + 1),
                                             //     so a knock-down always stages two clips.
    /* +0x1f0 */ void   *aimed_fire_action;    // [H] ptr to the aimed-fire action table (=&g_fire_action_aimed)
    /* +0x1f4 */ uint32_t aimed_fire_param;    // [M] zeroed when the aimed-fire block is armed
    /* +0x1fe */ uint8_t shotgun_reload_pending; // [H] the pump shotgun's shell-by-shell reload is running (§2)
    /* +0x218 */ uint32_t throw_charge_start_ms; // [H] mission clock when the throw wind-up began (§2)
    /* +0x21c */ uint8_t throw_charging;       // [H] a grenade / Molotov throw is being wound up (§2)
    /* +0x22c */ uint8_t extended_anim_active; // [H] the 4.0f "extended" aim animation is up; human_do_aimed clears it (and calls
                                             //     anim_machine_del_extended_anim(4.0f)) when the human lowers the weapon
    /* +0x22d */ uint8_t sniper_mode_flag;     // [H] set/cleared by player_set_sniper_mode
    /* +0x22e */ uint8_t ground_unsettled;     // [H] the ground under the human may change: an idle
                                             //     human re-runs the ground probe only while it is set (§2)
    /* +0x230 */ uint8_t weapon_model_dirty;   // [H] the visible weapon geometry needs rebuilding; human_do_aimed tests it and, if
                                             //     set, calls human_change_weapon_model
    /* +0x2dc */ C_ShotSkeleton shot_skeleton;  // [H] EMBEDDED, 0x130 B, spans +0x2dc..+0x40c — the per-limb
                                             //     hit locator. 16 I3D_frame joint handles
                                             //     then 10 capsules of 0x18 B. Byte-proven by a
                                             //     receiver census: 4 of 6 callsites of
                                             //     shot_skeleton_init/_test_collision are
                                             //     `lea ecx,[reg+0x2dc]`. See structs/shot_skeleton.md
    /* +0x40c */ uint32_t movement_mode;        // [H] locomotion mode; Go_* set = 2
    /* +0x410 */ uint32_t action_type;         // [M] the in-progress action; player_lock_controls switches on it to tear it down
    /* +0x418 */ float    fall_speed;          // [H] the downward speed in m/s: each movement tick adds
                                             //     dt * 9.81; above 2.0 the fall clip starts, the landing
                                             //     grade and the fall damage scale with it, above 50.0
                                             //     the human dies. A small seed starts a fall: script
                                             //     command human_forcefall (case 0xcb) adds 0.1 and
                                             //     human_intern_use_car sets 0.1f on leaving a vehicle (§2)
    /* +0x41c */ uint8_t  fall_started_in_jump; // [H] the fall began in the jump mode (0xb): the hard-
                                             //     landing limit rises from 8 to 18 and the damage
                                             //     offset from 2 to 12 (§2)
    /* +0x420 */ float   velocity_ch[3];        // [H] per-axis gas/velocity channels = speed·orient·speed_scale (written by Go_*)
    /* +0x43c */ uint32_t motion_intent;       // [H] cleared by human_do_play_anim and by player_lock_controls
    /* +0x44c */ int32_t  mode_timer_ms;       // [H] countdown of the timed movement modes 5, 7 and 10;
                                             //     -1 = none (§2)
    /* +0x478 */ void    *target_vehicle;       // [H] the vehicle the human is going FOR (approaching / breaking
                                             //     into), as distinct from the one it is IN (+0x98/+0x9c).
                                             //     Established by `human_is_this_car_stealing`, which asks "is anyone
                                             //     taking this car?" by comparing the car against exactly THREE actor
                                             //     fields — +0x478, +0x98 and +0x9c — and the latter two are the known
                                             //     vehicle_link pair, so +0x478 is a third vehicle reference.
                                             //     Cleared in player_lock_controls' state-10 teardown.
    /* +0x47c */ uint32_t shadow_frame_handle; // [M] handle returned by game_add_frame_for_shadow.
                                             //     Re-added on leaving a vehicle: a seated actor casts no world shadow
    /* +0x4a0 */ void    *weapon_slot;          // [H] equipped-weapon slot; human_weapon_drop unequips through
                                             //     inventory_remove_item_by_id(slot, 0)
    /* +0x564 */ I3D_frame *weapon_model_frame;  // [H] the VISIBLE WEAPON's model frame.
                                             //     human_repair_weapon_position re-parents THIS frame and rewrites
                                             //     its flag word (+0xac) with `& 0xfffffedf | 0x40000000`.
    /* +0x568 */ I3D_frame *weapon_model_frame_2; // [M] second weapon/held-item model frame.
                                             //     Released together with +0x564 (vtbl +0x24).
    /* +0x56c */ I3D_frame *weapon_attach_long_gun; // [H] attach frame for weapon_class 2 or 3 (the long guns)
    /* +0x570 */ I3D_frame *weapon_attach_default;  // [H] attach frame for every other weapon_class
    /* +0x580 */ uint8_t weapon_position_dirty;  // [H] "the weapon model needs re-seating"; human_repair_weapon_position
                                             //     gates on it and clears it
    /* +0x590 */ int32_t  idle_variation_timer_ms; // [H] armed to 5000 + rand while the idle clip plays;
                                             //     when it runs out an idle variation clip starts (§2)
    /* +0x594 */ int32_t  clip_hold_ms;        // [H] while >= 0 a slightly lower clip id cannot replace
                                             //     the current clip (§2)
    /* +0x5fc */ uint32_t human_type;           // [H] the human's TYPE / faction, as a **SINGLE-BIT MASK**
                                             //     (bit 0 = type 0, bit 1 = type 1, ...; max 11 types). The index is
                                             //     recovered by COUNTING TRAILING ZEROS. A mask of 0 = "no
                                             //     type" and hits are always allowed. Consumed by
                                             //     `human_is_hit_allowed_between_types` to index
                                             //     `g_hit_permission_matrix`.
    // ---- the 15-entry PROPERTY VECTOR, twice: maxima then current values ----
    // Named by the developers' own table (see §2b). prop_cur[i] / prop_max[i] * 100 is what
    // `human_getproperty` returns; `human_setproperty` writes prop_cur[i] = pct * 0.01 * prop_max[i].
    /* +0x600 */ float    prop_max[15];       // [H] 0 Strenght(sic) 1 Energy 2 EnergyHandL 3 EnergyHandR
                                             //     4 EnergyLegL 5 EnergyLegR 6 Reactions 7 Speed
                                             //     8 Aggresivity 9 Intelligence 10 Shooting 11 Sight
                                             //     12 Hearing 13 Driving 14 Mass.
                                             //     +0x604 = prop_max[1] Energy = max_health: C_human ctor
                                             //     -> 100.0f, C_player ctor OVERRIDES to 200.0f.
                                             //     +0x608..+0x614 = the four LIMB energies (4 x 40.0f).
    /* +0x63c */ float    f_63c;              // [M] always 0.5f; one past the property block, not part of it.
                                             //     Its current-value twin +0x67c is the morale that
                                             //     ai_character_get_morale reads (structs/ai_character.md)
    /* +0x640 */ float    prop_cur[15];       // [H] the live values, same 15 indices.
                                             //     +0x644 = prop_cur[1] Energy = cur_health. Damage subtracts
                                             //     from it; clamped to max_health on savegame load;
                                             //     game_cheat_check sets it to 100 (health cheat);
                                             //     cur/max*100 drives hud_set_player_health_value.
                                             //     Crossing 0.01 downwards through `human_setproperty` is a
                                             //     DEATH — it restores Energy to 1.0 and fires a hit.
    /* +0x65c */ float   speed_scale;           // [H] locomotion speed scale factor (multiplies the velocity channels)
    // ---- dynamic-collision block (the capsules that make the animated body hittable) ----
    // Built by human_create_dyn_colls (2 objects), freed by human_erase_dyn_colls, registered /
    // unregistered by human_activate_colls / human_deactivate_colls, refreshed every tick by
    // human_update_dyn_coll, and RESIZED by human_do_crouched (a crouching human is a smaller target).
    /* +0x688 */ void   **dyn_coll_objects;     // [H] array of tDynamicCollObject* (0x48 bytes each on PC)
    /* +0x68c */ void   **dyn_coll_frames;      // [H] parallel array of the I3D_frames they track; [0] = the actor's
                                             //     main frame (+0x68), [1] = a named sub-frame when the model has one
    /* +0x690 */ int32_t  dyn_coll_count;       // [H] element count of BOTH arrays; human_create_dyn_colls sets it to 2
    /* +0x694 */ uint8_t  dyn_colls_active;     // [H] 1 while the objects are registered with the collision world
    /* +0x6d8 */ int32_t  wingman_slot;         // [H] the HUD wingman slot (0..3) that shows this human's
                                                //     health; -1 = none (§2)
    /* +0x6dc */ uint32_t wingman_portrait;     // [H] the portrait id (0..7) shown in that slot
    /* +0x6ec */ int16_t  print_total;          // [H] bloody-footprint alpha denominator (§2)
    /* +0x6ee */ int16_t  prints_left;          // [H] armed to 10; one footprint per footstep event
    /* +0x6f0 */ uint8_t  print_texture;        // [H] decal texture index; set to 6 (2bfootprint.bmp)
    /* +0x6f8 */ void    *carried_object;       // [H] carried box/object slot (human_do_box_pick_up / human_do_box_drop)
    /* +0x6fc */ int32_t  carried_box_group;    // [H] the BOX GROUP id of +0x6f8. Script command
                                                //     human_havebox (case 0xdd) returns it, or
                                                //     -1.0 when nothing is carried
    /* +0x700 */ void    *carried_object_2;     // [H] second carried-object slot —
                                                //     specifically a carried BODY: script command
                                                //     human_havebody (case 0xde) tests exactly this
    /* +0x720 */ uint8_t  no_hit_anim;          // [H] suppress the flinch clip on a weapon hit; sole
                                                //     writer human_set_no_hit_anim, whose sole caller
                                                //     is script command setnoanimhit (case 0xbf)
    /* +0xa58 */ int32_t  freeze_countdown_ms;  // [H] while > 0, human_update subtracts dt and returns
                                                //     before anything else runs, and human_ai exits early.
                                                //     Set to -1 by human_game_init / human_activate; no
                                                //     store sets it positive
} C_human;                                      // partial — C_human band only
```

## 2. Field evidence — C_human band

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x74 | cur_anim_id | H | `human_set_movement_intent_forward` (+ the 7 `human_set_movement_intent_*` cohort), `human_do_corner_breath`, `human_do_corner_view`, `human_do_corner_fire`, `human_do_corner_fire_aimed` | `human_do_jump` | `6 - (run?1:0)` in the forward path. The `Do_Corner*` family writes a clip id chosen from a base (0x4e/0x50/0x53/0x58) plus a variant index, `+1` again when a category-4 weapon is held (see `+0x1e4`). |
| +0x98 / +0xac | vehicle_link / seat_index | H | `human_put_in_vehicle_at_seat` | `human_set_movement_intent_forward` (+ cohort) | writes `+0x98`=vehicle, `+0xac`=seat; kind@`+0x10`==4 ⇒ in a car |
| +0xbc..+0x1e3 | anim_machine (EMBEDDED) | H | `anim_machine_set_anim`, `anim_machine_recompile_matrix`, `anim_machine_del_extended_anim` | `anim_machine_get_active_anim_time`, `anim_machine_stop_anims` | The `C_animations_machine_de_luxe` sub-object. Confirmed by **disassembly**, not the decomp: every C_human caller does `lea ecx,[this+0xbc]` before the call, which Ghidra hides. `0x128` bytes, so it ends exactly at `+0x1e4`, the first field after it. Full layout: `structs/anim_machine.md` |
| +0x480..+0x557 | **G_Inventory** (EMBEDDED) | H | `inventory_insert_item`, `inventory_holster`, `inventory_remove_item_by_ptr` | `inventory_has_item_with_property`, `inventory_get_active_weapon_ammo`, `inventory_collect_items_to_list` | The weapon/item inventory, in four FIXED slot bands (§2d). Full layout: `structs/inventory.md` |
| +0x724..+0x757 | **ai_loc** (EMBEDDED, `0x34`) | H | `ai_loc_init`, `ai_loc_update`, `ai_loc_add_or_refresh_seg_stat`, `ai_loc_prune_stale_seg_stats` | `ai_wayfind_plan_directed_route_v3` (`node_a` / `node_b` / `flags` at `+0x730` / `+0x734` / `+0x73c`); nothing reads the `ai_seg_stat` list (`structs/ai_loc.md` §3) | The per-character **navigation-locality cache**: resolved `C_web_node` indices, the containing web connection, and a 4000 ms-expiring list of nearby `check_seg` specials. Proven by an ECX-receiver census over the callsites (`lea ecx,[esi+0x724]`). Full layout: `structs/ai_loc.md` |
| +0x758..+0x99b | **ai_blob_log** (EMBEDDED, `0x244`) | H | `ai_blob_log_add_blob`, `ai_blob_log_clear` | `ai_blob_log_get_blob`, `entity_trace_human_scent` | The **scent trail** — a 16-slot ring of `ai_blob` records (stride `0x24`), one blob per 500 ms at the frame origin `y+1.0`. The producer half of the SMELL sense. Proven by an ECX-receiver census (`lea ecx,[reg+0x758]` at 7 of 8 callsites across three methods). Full layout: `structs/ai_blob_log.md` |
| +0x70 / +0x74 | anim_slot / cur_anim_id | M | `player_lock_controls` | - | reset together to (0, 1) = idle when the controls are locked |
| +0x9c | vehicle_link_2 | M | - | `player_indicator_set_aimed`, `human_set_movement_intent_forward` (+ cohort) | a SECOND vehicle slot — the crosshair logic kind-checks BOTH `+0x98` and `+0x9c` |
| +0x1e5 | **aimed** (stance flag) | H | `human_do_aimed` (sole writer), `player_set_sniper_mode`, `player_lock_controls` | `player_indicator_set_aimed` | the AIMED stance flag (§2d). Force-cleared while carrying an object (`+0x6f8`/`+0x700` guard). Raising the weapon also arms the aimed-fire block below. |
| +0x1f0 / +0x1f4 | aimed_fire_action / _param | H | `player_set_sniper_mode`, `player_lock_controls` | - | the aimed-fire action block, armed alongside the `+0x1e5` stance: `player_set_sniper_mode` sets (`+0x1f0`=&g_fire_action_aimed, `+0x1f4`=0); `player_lock_controls` DISARMS it. |
| +0x22c | extended_anim_active | H | `human_do_aimed` | `human_do_aimed` | set while the 4.0f "extended" aim animation is up; on lowering the weapon `human_do_aimed` calls `anim_machine_del_extended_anim(4.0f)` and clears it. |
| +0x230 | weapon_model_dirty | H | `human_do_change_weapon` | `human_do_aimed` | when set, raising the weapon makes `human_do_aimed` call `human_change_weapon_model` to rebuild the visible geometry first. `human_do_change_weapon` sets it to 1 — a weapon switch marks the model dirty so the next aim rebuilds it. |
| +0x47c | shadow_frame_handle | M | `human_intern_use_car`, `human_intern_use_railway` | - | handle from `game_add_frame_for_shadow`. Re-added on LEAVING a vehicle — a seated actor casts no world shadow. |
| +0x688 / +0x68c / +0x690 / +0x694 | **dyn_coll_objects / _frames / _count / _active** | H | `human_create_dyn_colls`, `human_erase_dyn_colls`, `human_activate_colls`, `human_deactivate_colls`, `human_do_crouched` | `human_update_dyn_coll` | the DYNAMIC-COLLISION block: the capsules that make the animated body hittable. Two parallel arrays (objects + the I3D_frames they track) of `count` elements, built with `count = 2`. `human_update_dyn_coll` re-syncs them to the animated frames each tick; `human_do_crouched` SHRINKS object[0] when crouching. |
| +0x22d | sniper_mode_flag | H | `player_set_sniper_mode` | - | 1 while the sniper scope is up |
| +0x218 / +0x21c | throw_charge_start_ms / throw_charging | H | `human_dispatch_weapon_action_at_target` (1 on the press of a class-4 weapon, 0 on the release), and ten more `human_*` methods that interrupt the action | `human_dispatch_weapon_action_at_target`, `player_ai_walk`, `human_save_game_save` | the throw wind-up of the grenade and the Molotov cocktail. The press stores the mission clock at `+0x218` and plays the throw clip; the release cancels below 401 ms, caps the hold at 2000 ms and throws with power = hold ms * 0.015 * Strength (`+0x640`) |
| +0x22e | ground_unsettled | H | `human_movement_intern` (1 when the ground probe misses; on a hit 1 when the ground is a dynamic collision object, kind bit `0x40`, or carries flag `0x10000`, else 0; 0 while seated), `human_update` (1 when a played clip ends, except clips `0x68` / `0x69`), `human_do_play_anim` (0) | `human_movement`, `human_movement_intern` | while it is clear and `fall_speed` is 0, an idle human (movement_mode 1) skips the movement pass; while it is set the movement pass re-runs the ground probe every tick |
| +0x418 / +0x41c | fall_speed / fall_started_in_jump | H | `human_movement_intern` (`+= dt * 9.81`; at least 3.0 when the ground is out of reach; 0 after landing; `+0x41c` when the fall begins), `human_movement` (0.05 seed; 60.0 for the player below y -30), `human_update` (0.01 seed), `human_intern_use_car` (0.1 seed), script command `human_forcefall` (+0.1), `human_do_play_anim` (0) | `human_movement`, `human_movement_intern`, `human_save_game_save` | the vertical fall. Above 2.0 the human enters movement_mode 3 with clip `0x66`. On landing: up to 5.5 a soft landing; up to 8 (18 when `+0x41c` is set) the landing clip `0x67` and a recovery timer in `+0x44c`; above that a fall hit of (speed - 2, or - 12) * 10 through the hit virtual, with force feedback at speed * 0.1. Above 50.0 in mode 1 or 3 the human dies |
| +0x410 | action_type | M | `player_lock_controls` | `player_lock_controls` | the in-progress high-level action; the controls-lock teardown switches on it (cases 1-5) and then zeroes it |
| +0x43c | motion_intent | H | `human_do_play_anim`, `player_lock_controls` | - | cleared whenever locomotion is interrupted |
| +0x44c | mode_timer_ms | H | `player_try_use_car` (mode 10, the car break-in: `car_database_get_steal_time` clamped to 2000..30000, the total copied to `+0x474`), `human_movement_intern`, `human_dispatch_weapon_action_at_target`, `human_do_corner_view`, `human_do_corner_fire`, `human_do_dead_body_drop`, `human_hit` (mode 5: the clip time - 250 ms), `human_movement` (mode 7: 2000 + rand), `human_ai`, `player_lock_controls`, `player_ai` (-1) | `human_movement`, `player_try_use_car`, `player_ai`, `human_save_game_save` | `human_movement` subtracts dt in modes 5, 7 and 10 and leaves the mode when it runs out |
| +0x478 | target_vehicle | H | `player_try_use_car`, `player_lock_controls` (cleared in the state-10 teardown) | `human_is_this_car_stealing` | the vehicle the human is going FOR, per §1 |
| +0x564 / +0x568 | **weapon_model_frame / _2** | H | `human_repair_weapon_position` | `human_repair_weapon_position`, `human_intern_use_car`, `player_lock_controls` | the VISIBLE WEAPON's model frames, settled by `human_repair_weapon_position` (§2d): the function zeroes the frame's position (`+0x80..+0x88`) and rewrites its flag word (`+0xac`) with `& 0xfffffedf \| 0x40000000`. Both are released through vtable `+0x24` in `player_lock_controls`' teardown |
| +0x56c / +0x570 | weapon_attach_long_gun / weapon_attach_default | H | - | `human_repair_weapon_position` | the two ATTACH frames the weapon model is re-parented between: `+0x56c` while `weapon_class` is 2 or 3 (the rifles, the Thompson and the shotguns), `+0x570` for any other class |
| +0x580 | weapon_position_dirty | H | `human_do_reload`, `human_notify_callback`, `human_repair_weapon_position` (clears) | `human_repair_weapon_position` | "the weapon model needs re-seating" — the repair pass gates on it and clears it on entry. `human_do_reload` sets it to 1 (a reload displaces the weapon); `human_notify_callback` — the human I3D animation-event callback, a **free function** — sets it in its event-switch case `0x1f`. |
| +0x1fe | **shotgun_reload_pending** | H | `human_shooting` and `human_do_reload` (set); `human_ai`, `human_hit`, `human_change_weapon_model`, `human_do_change_weapon`, `human_dispatch_weapon_action_at_target`, `player_ai_walk` (clear); `human_game_init`, `human_activate` (init) | `human_ai`, `human_notify_callback`, `human_repair_weapon_position`, `human_dispatch_weapon_action_at_target`, `player_ai` | the pump shotgun's shell-by-shell reload. Set when a shot empties the pump-action shotgun (`predmety.def` id 11) with a round in reserve, and when `human_do_reload` starts on it with a round to load. While set, `human_ai` reloads one more shell each time the action step reaches 3 and clears the flag (dropping the extended animation) once no round is left; `human_repair_weapon_position` and two animation events wait for it to clear |
| +0x1f0 (globals) | the 3 fire-action tables | H | `player_set_sniper_mode` | `human_repair_weapon_position` | the three tables the pointer may hold: **`g_fire_action_ready`**, **`g_fire_action_aimed`**, **`g_fire_action_onoff`** (named from their PS2 twins, §2d) |
| +0x590 / +0x594 | idle_variation_timer_ms / clip_hold_ms | H | `human_animation_evaluate_and_dispatch` (`+0x590` = 5000 + rand when the dispatched clip is the idle clip 1, else -1; `+0x594` = the clip's remaining time - 280 ms when the caller asks to hold the clip, else -1), `human_update` (both count down), `human_ai` (the reload event sets 5000 + rand and -1), `human_shooting`, `human_do_change_weapon`, `human_drop_out_items`, `human_do_play_anim`, `human_do_jump`, `human_do_climb` | `human_update`, `human_animation_evaluate_and_dispatch`, `human_save_game_save` | `+0x590`: while the human is alive and `+0x2d8` is 0, `human_update` counts it down; when it passes zero with the byte `+0x598` set, an idle variation plays (`cur_anim_id` = 2 + rand). `+0x594`: while it is >= 0, a requested clip id up to 3 below the current one is held at the current one |
| +0x1e4 | **crouched** (stance flag) | H | `human_do_crouched` (sole writer) | `human_set_movement_intent_forward` (+ cohort), `human_do_corner_view`, `human_do_corner_fire`, `human_do_corner_fire_aimed` | the CROUCHED stance flag (§2d). What crouching does: (1) forces walk over run in the `Go_*` cohort; (2) when CLEAR **and** the equipped weapon's def record has `+0x50 == 4`, the anim setters pick the alternate clip (`base_id + 1`). It also SHRINKS `dyn_coll_objects[0]` — a crouching human is genuinely a smaller target. |
| +0x1e8 | **weapon_class** | H | `human_change_weapon_model` (from the in-hand item's `predmety.def` record `+0x5c`), `human_game_init`, `human_activate` | `player_indicator_set_aimed`, `human_do_corner_fire`, `human_do_corner_fire_aimed`, `human_update`, `human_repair_weapon_position`, `human_notify_callback` | the in-hand item's weapon class (§2c). The aim crosshair shows only for classes `[1,4)`, the firearms; the corner-fire setters refuse `0` (bare hands) and `2` (a rifle); `human_update` maps the four melee classes onto a block clip (§2c); `human_repair_weapon_position` picks the long-gun attach frame for 2 and 3 |
| +0x1ec | **shooting** latch | H | `human_dispatch_weapon_action_at_target` (set after it stores the shot target at `+0x200`), `human_notify_callback`; cleared by `human_hit`, `human_do_jump`, `human_change_weapon_model`, `human_do_change_weapon`, `human_put_in_vehicle_at_seat`, `human_drop_out_items`, `human_use_actor`, `human_do_box_pick_up`, `human_do_dead_body_pick_up`, `human_reset_actions`, `entity_action_tick`; saved and restored by `human_save_game_save` / `_load`, initialised by `human_game_init` / `human_activate` | `human_update`, `human_dispatch_weapon_action_at_target`, `player_ai`, `entity_action_tick`, `ai_blob_log_add_blob` | while set and `weapon_class` is 1-3 (a firearm), `human_update` calls `human_shooting` every frame; it also blocks the melee guard pose (§2c) and `player_ai`'s idle branch, and gives a scent blob its flag `0x04` (`structs/ai_blob_log.md`) |
| +0x70 / +0x74 | **anim_slot / cur_anim_id** | H | `human_do_shock_down` (+0x74), the `Go_*` cohort, the whole Corner family, the Shock family | `human_do_shock_up`, `human_do_shock_stumble` (read `+0x70`) | `+0x70` = the anim the human is CURRENTLY IN; `+0x74` = the anim being REQUESTED. The shock continuations read `+0x70`, accept one of 3 ids, and write `+0x74 = id+1`. Anim-id bands are listed in §1. **There is no out-of-line `C_human` reader of `+0x70`:** every read is inlined — dispatcher case `0x181` HUMAN_GETACTANIMID reads the field directly rather than calling anything. `C_door` uses the same offset for its own state field, which is a fact about the layout and not about any function. PS2 twin offsets: §2d. |
| +0x200..+0x208 | **shot target point** (world-space vec3) | H | `human_set_shot_target_pos` (VA 0x00425780); `human_dispatch_weapon_action_at_target` stores its own `S_vector *` parameter here; the corner-fire `ai_action` stores the target's position when the threat is high and its own position projected 1 m forward when it is not | `human_shooting` (shot direction = `*(this+0x200) − frame_world_pos`), `human_notify_callback` (→ `game_new_grenade`), `player_ai_drive` / `player_ai_car_fire` (→ `collision_test_line_horizontal`), the corner-fire action (`+0x204 + 0.8` = chest-height y-bias) | the CLASS is proven rather than inferred: a `C_human` method (`human_dispatch_weapon_action_at_target`) writes it, and a `C_human` method cannot address a `C_entity` field (§2d). |
| +0x23c / +0x240 / +0x254 | shock_clip_armed / _a / _b | H | `human_do_shock_down` | - | the knock-down staging block: flag=1, clip_a = `severity*3 + 0x3d`, clip_b = clip_a + 1, so a knock-down always stages two clips. |
| +0x40c / +0x420..+0x428 | movement_mode / velocity_ch | H | `human_set_movement_intent_forward` (+ cohort) | - | mode=2; vel = speed·`+0x30..`·`+0x65c` |
| +0x4a0 | weapon_slot | H | `human_weapon_drop`, `human_weapon_toggle_held` | - | equipped weapon (drop unequips; toggle re-selects) |
| +0x604 | max_health | H | `player_ctor` (200.0f), `human_load_stats_block_scaled`, `human_load_stats_block_plain` | `player_save_game_load` | **C_human field, NOT C_player.** The `C_human` ctor initialises it to **100.0f** (§1, §2d); `player_ctor` overrides it to **200.0f** — the player has double an NPC's max health. |
| +0x5fc | **human_type** | H | - | `human_is_hit_allowed_between_types` | the human's TYPE/faction as a **single-bit mask** (index = trailing-zero count; **max 11 types**). Mask 0 = no type ⇒ hits always allowed. It is the index into **`g_hit_permission_matrix`** (0x0064ba78) — an **11×11 byte matrix of who may hit whom**. |
| +0x608..+0x614 | **prop_max[2..5]** — the four LIMB energies | H | `player_ctor` (4 x 40.0f), `human_load_stats_block_scaled`, `human_load_stats_block_plain` | `human_getproperty` / `human_setproperty` (dispatcher cases `0x66` / `0x65`) | **`EnergyHandL`, `EnergyHandR`, `EnergyLegL`, `EnergyLegR`**, from the developers' own property-name table (§2b). The four 40.0f seeds are per-limb health against the player's 200.0f torso Energy. |
| +0x644 | cur_health | H | `game_cheat_check` (=100, the health cheat), `player_save_game_load`, `game_init` | `player_save_game_load` | **C_human field, NOT C_player.** Damage subtracts from it (§1, §2d). `player_save_game_load` clamps it to max_health, then pushes `cur/max*100` to `hud_set_player_health_value`. |
| +0x6d8 / +0x6dc | wingman_slot / wingman_portrait | H | `human_set_wingman_indicator` (script `wingman_setindicator` / `wingman_delindicator`, on the human `act[xx]`), `human_game_init` / `human_activate` (slot -1), `human_save_game_load` | `human_hit`, `human_hit_in_car`, `human_die_in_exploding_car`, `human_save_game_save` / `_load`, `game_character_stack_pop`, `program_process_command` | while the slot is 0..3 every hit pushes the human's health percentage (`+0x644 / +0x604 × 100`) to that HUD slot and a death in an exploding car pushes 0; a save load re-applies the portrait, the slot bit and the value (`basic/artifacts/hud_elements.md` §7) |
| +0x6ec / +0x6ee / +0x6f0 | print_total / prints_left / print_texture | H | `human_arm_bloody_footprints` (10 steps, texture 6), `human_notify_callback` (decrements `+0x6ee`), `human_game_init` / `human_activate` (clear `+0x6ee`), `human_save_game_load` | `human_notify_callback`, `human_save_game_save` | bloody footprints: when the foot ray crosses a blood decal's collision sphere, `collision_line_in_dynamics` calls `human_arm_bloody_footprints` (effects quality > 1 only); each footstep event then places one decal of texture `+0x6f0` with alpha `(+0x6ee + 2) / +0x6ec`, ten in all (`structs/blood.md` §5e) |
| +0x6f8 / +0x700 | carried_object[2] | H | `human_do_box_pick_up` | `human_do_box_drop`, `human_set_movement_intent_forward` (+ cohort) | carried box. `human_do_box_pick_up` writes `+0x6f8`; `human_do_box_drop` and `human_do_box_cancel` clear it. |
| +0xa58 | freeze_countdown_ms | H | `human_game_init`, `human_activate` (both -1), `human_update` (decrements) | `human_update`, `human_ai` | while positive, `human_update` only decrements it and returns, and `human_ai` exits early. A disassembly of every function finds no store that sets it positive (§2d) — so the freeze never engages, and `human_update`'s main path (which lays the scent trail, `structs/ai_blob_log.md`) always runs unless a recorded track drives the pose |

### 2a. Three more fields, from the collision-owner and animation-side paths

| Offset | Field | Conf | PS2 | Notes |
|---|---|---|---|---|
| `+0x69c` / `+0x6a0` | collision ignore list (begin/end; allocator byte `+0x698`) | H | `+0x6fc` / `+0x700` | the colliders of the entity's `ai_ghost` records, rebuilt by `entity_tick_ghosts` (`structs/ai_ghost.md` §3a); `human_set_next_coll_owners` and `human_movement_intern` merge it with the dyn-coll capsules before a query. PC stores begin/end pointers, PS2 a pointer + count |
| `+0x6ac` / `+0x6b0` | merged scratch vector (head `+0x6a8`) | H | `+0x708` / `+0x710` (`+0x70c` = count) | the temporary the two lists are concatenated into, then published to `g_collision`, whose queries skip every object in it |
| `+0x1e6` | animation SIDE flag (`'L'`/`'R'`) | H | `+0x392` | selects the mirrored clip variant; `human_shift_animation` patches the `'?'` in the clip file name from it, and `human_do_jump` sets it from which foot is down. Extends the proven `+0x1e4`↔`+0x390` / `+0x1e5`↔`+0x391` stance band by one |

### 2b. The 15 property names — the developers' own table

`human_setproperty` / `human_getproperty` (script dispatcher cases `0x65` / `0x66`) take the property
as a **NAME**, and the load-time arm resolves it against a table of 15 sixteen-byte strings that it
scans **backwards** from a sentinel. Byte-read from `Game.exe` at `0x0063e3a8`, stride `0x10`:

| idx | name | `prop_max` | `prop_cur` |
|---:|---|---|---|
| 0 | `Strenght` *(sic — the developers' typo)* | +0x600 | +0x640 |
| 1 | `Energy` | +0x604 | +0x644 |
| 2 | `EnergyHandL` | +0x608 | +0x648 |
| 3 | `EnergyHandR` | +0x60c | +0x64c |
| 4 | `EnergyLegL` | +0x610 | +0x650 |
| 5 | `EnergyLegR` | +0x614 | +0x654 |
| 6 | `Reactions` | +0x618 | +0x658 |
| 7 | `Speed` | +0x61c | +0x65c |
| 8 | `Aggresivity` *(sic)* | +0x620 | +0x660 |
| 9 | `Intelligence` | +0x624 | +0x664 |
| 10 | `Shooting` | +0x628 | +0x668 |
| 11 | `Sight` | +0x62c | +0x66c |
| 12 | `Hearing` | +0x630 | +0x670 |
| 13 | `Driving` | +0x634 | +0x674 |
| 14 | `Mass` | +0x638 | +0x678 |

A name that matches nothing leaves the index at -1 and the instruction is **rejected at compile time**,
so every property name a shipped script uses is in this list.

**Three blocks of the class band are this vector:**

* `+0x604` / `+0x644` (`max_health` / `cur_health`) are **index 1, `Energy`**.
* `+0x608..+0x614`, the four 40.0f floats, are the **four limb energies** (§2). This is the field the
  10 hit capsules of the embedded `C_ShotSkeleton` (§4d, `structs/shot_skeleton.md`) fold down onto.
* `+0x618..+0x638`, nine dwords, are `Reactions` … `Mass`.

**Corroboration from the loader.** `human_load_stats_block_plain` reads a **`0x3c`-byte** chunk record —
exactly 15 floats — and `human_load_stats_block_scaled` reads `0x40`, the same 15 plus a scale float it
folds in as `1/(1-x)`. The record IS the `prop_max` vector, which is why `+0x63c` (always 0.5f) sits
outside it.

**`prop_cur` is only partly independent.** `+0x65c` is separately documented in §1 as
`speed_scale`, i.e. **`prop_cur[7]` (`Speed`) is the live locomotion multiplier** — the property system
and the movement code are reading the same float. Treat any other `prop_cur` slot the same way: the
"current" value is the live game quantity, not a copy.

### 2c. `weapon_class` — the eight classes, and the four that name a melee BLOCK clip

`human_change_weapon_model` copies the class from the in-hand item's `predmety.def` record
(`+0x5c`, stride `0xbc`). The shipped table gives eight classes:

| `+0x1e8` | items (`predmety.def` id) |
|---|---|
| 0 | empty hands (0) and every non-weapon: the special action, the keys, the bucket, the flashlight, the documents, the papers, the explosives, the tickets, the package, the dog's head |
| 1 | the four handguns (6 .. 9) |
| 2 | the two rifles (13, 14) |
| 3 | the Thompson and the two shotguns (10 .. 12), and the unused silent copies (33, 34) |
| 4 | the throwables: the Molotov cocktail (5) and the grenade (15) |
| 5 | the knuckle-duster (2) |
| 6 | the knife (3) and the bottle (29) |
| 7 | the blunt melee items: the baseball bat (4), the steel bar (20), the crowbar (25), the plank (28), the sword (31) |

`human_update` runs **two consecutive switches on `+0x1e8`**. The first selects between three
bodies through a byte index table; its grouping is `{0, 5, 6, 7}` / `{1, 2, 3}` / `{4}` (§2d). Only
the `{0, 5, 6, 7}` arm continues, and only when every
one of six guards passes: the human is not seated in a type-4 linked object (`+0x98`, `+0xac`),
`+0x1e5` is set, `+0x1b0` is clear, `+0x40c` is not 5 and `+0x1ec` (the shooting latch) is clear.

The second switch is the payoff. Each arm names a shipped animation asset, loads it through
`i3d_cache_anim_create`, and — when the load returns non-null — sets `+0x22c`
(`extended_anim_active`) and calls `anim_machine_set_extended_anim(anim, 0.8f, 4.0f, 0)`. The
`4.0f` is the same "extended" blend constant `human_do_aimed` passes when it tears the clip down.

| `+0x1e8` | animation asset |
|---|---|
| 0 | `boj HPT box kryt.i3d` |
| 5 | `boj HPT box kryt.i3d` |
| 6 | `boj HPT nuz kryt.i3d` |
| 7 | `boj HPT basb kryt.i3d` |

*boj* = fight, *kryt* = guard, *box* / *nuz* / *basb* = fists / knife / baseball bat. These are the
four melee classes; 0 (bare hands) and 5 (the knuckle-duster) share the fists clip. Classes 1-4, the
firearms and the throwables, never reach this switch.

Reading the second switch requires a jump-table override in the disassembler: without one it
renders as a single opaque indirect call through the table address, and the three arms belong to
no function at all.

### 2d. PC ↔ PS2 field offsets

PC first. The band does not rebase by a constant — the kind field alone is PC `+0x10` / PS2
`+0x50` — so each pair is pinned by a body-diff. Anim ids are per-build: the PS2 uses different
bases.

| PC | PS2 | pinned by |
|---|---|---|
| `+0x70` / `+0x74` | `+0x200` / `+0x204` | the `Do_Corner*` / `Do_Shock*` body-diffs. PS2 passes `+0x200` to `GetAnimIndex__FUi` in `ShiftAnimation` / `_intern_ForceDeath` / `RecompileDeathPos` and compares it against clip ids `0x1b..0x23`, `0x1a..0x24`, `0xbf`, `0xc1` |
| `+0x98` / `+0x9c` / `+0xac` | `+0x224` / `+0x228` / `+0x238` | - |
| `+0xbc..+0x1e3` (anim_machine) | `+0x250` | - |
| `+0x1e4` / `+0x1e5` | `+0x390` / `+0x391` | `human_do_crouched` ↔ `Do_Crouched__7C_humanFb`; `human_do_aimed` ↔ `Do_Aimed__7C_humanFb` (line-for-line; shared float constant `0x40800000` = 4.0f) |
| `+0x1e8` | `+0x394` | the same `+4` delta as `+0x1e4` ↔ `+0x390`. `Do_Shoot__7C_human` switches on it (cases 0/5/6/7), the value set of `human_update`'s first switch (§2c); the corner-fire refusal on 0 or 2 is the same gate on both builds |
| `+0x1ec` | `+0x398` | `Update__7C_humanFUi` gates `Shooting__7C_humanFv` on the same byte |
| `+0x1f0` / `+0x1f4` | `+0x39c` / `+0x3a0` | the PS2 names the three tables the pointer may hold: `g_fire_action_ready` = `_fire_action_ready_`, `g_fire_action_aimed` = `_fire_action_aimed_`, `g_fire_action_onoff` = `_fire_action_onoff_` |
| `+0x1fe` | `+0x3aa` | - |
| `+0x200..+0x208` (shot target) | - | the PS2's own `+0x200` is the anim slot (first row); do not pair them |
| `+0x218` / `+0x21c` / `+0x22c` / `+0x22d` / `+0x22e` / `+0x230` | `+0x3b8` / `+0x3bc` / `+0x3cc` / `+0x3cd` / `+0x3ce` / `+0x3d0` | `Do_Shoot__7C_humanFbP8S_vector` keeps the throw start time and flag at `+0x3b8` / `+0x3bc`; `Movement__7C_humanFUi` tests `+0x3ce` in the same idle gate |
| `+0x23c` / `+0x240` | `+0x3dc` / `+0x3e0` | the PS2 knock-down clip base is `0x1b` |
| `+0x40c` | `+0x588` | - |
| `+0x418` | `+0x594` | `Movement__7C_humanFUi` / `MovementIntern__7C_humanFUi`: the same `dt * 9.81` integration, the same 0.05 seed and the same kill above 50.0 |
| `+0x43c` | `+0x5a0` | - |
| `+0x44c` | `+0x5a4` | `SetWaitAnim__7C_humanFv` stores the clip time - 250 there with mode 5; `Movement__7C_humanFUi` counts it down in mode 5 |
| `+0x478` | `+0x5c4` | `human_is_this_car_stealing` ↔ `IsThisCarStealing__7C_human` |
| `+0x47c` | `+0x5c8` | - |
| `+0x480..+0x557` (G_Inventory) | `+0x5cc` | delta `0x14c`; four FIXED slot bands on the PC where the PS2 uses one vector |
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

## 3. Prefix caveat (read before the coverage table)

The `human_` prefix marks a `C_human` method, but it is neither necessary nor sufficient. Two things
cut across it:

* **Subclass methods operate on C_human fields.** `player_ctor`, `player_lock_controls`,
  `player_set_sniper_mode`, `player_indicator_set_aimed` and `player_save_game_load` are the writers or
  readers of many of the fields in §2 — they are `C_player` methods reaching into the base band.
* **The translation unit holds foreign functions** (§4c): `inventory_*`, `ai_loc_*`, `ai_blob_log_*` and
  two unowned COMDAT folds sit inside the C_human code range without being C_human methods.

The §4 table therefore maps each PS2 method to whichever PC function implements it, whatever prefix
that function carries.

## 4. `C_human` method family — PC ↔ PS2 (all 146 methods)

PC-first: the PC function is the subject, the PS2 symbol supplies the developers' own name. A `-` in a
cell means "none". Sorted by PS2 VA. Status vocabulary:

| status | meaning |
|---|---|
| CONFIRMED | body-diffed on both builds |
| STRUCTURAL | symbol match only — the PS2 body is a stub, so a body match is impossible |
| INLINED | the PS2 method has no PC function: MSVC spliced it into its caller |
| PS2-ONLY | the method exists only in the PS2 port; no PC twin exists |
| base-stub | 8-byte overridden virtual with no distinct PC twin |

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0048f430 | `human_ctor` | `__ct__7C_humanFv` | 0x00165a40 | CONFIRMED | base ctor (kind=3) |
| 0x0048f670 | `human_dtor` | `__dt__7C_humanFv` | 0x00166180 | CONFIRMED | base dtor |
| 0x0048f7d0 | `human_init` | `Init__7C_humanFP9I3D_frame` | 0x00166300 | CONFIRMED | init from frame |
| 0x0048f880 | `human_duplicate_from` | `DuplicateFrom__7C_humanFP7C_actor` | 0x00166400 | CONFIRMED | - |
| 0x0048f8b0 | `human_save_data` | `SaveData__7C_humanFP7C_chunk` | 0x001664c0 | CONFIRMED | - |
| 0x0048f950 | `human_load_data` | `LoadData__7C_humanFP7C_chunk` | 0x00166840 | CONFIRMED | - |
| 0x0048fe50 | `human_save_game_get_size` | `SaveGameGetSize__7C_humanFv` | 0x00166d80 | CONFIRMED | C_human SaveGameGetSize; C_entity has its OWN in the C_entity vtable |
| 0x0048fe90 | `human_save_game_save` | `SaveGameSave__7C_humanFPPv` | 0x00166de0 | CONFIRMED | writes type-6 record + C_human fields |
| 0x004901e0 | `human_save_game_load` | `SaveGameLoad__7C_humanFPPv` | 0x00167180 | CONFIRMED | C_human SaveGameLoad |
| 0x00490a20 | `human_death` | `Death__7C_humanFv` | 0x00167c10 | CONFIRMED | - |
| 0x00490b10 | `human_extern_ai_update` | `Extern_AI_Update__7C_humanFUi` | 0x00167d30 | CONFIRMED | small externally-driven anim tick (NOT human_ai): anim-machine IsActive/Tick + frame virtual + human_update_dyn_coll tail |
| 0x00490b50 | `human_sounds_change` | `SoundsChange__7C_humanFi` | 0x00167d90 | CONFIRMED | - |
| 0x00490bf0 | `human_ai` | `AI__7C_humanFUi` | 0x00167e60 | CONFIRMED | huge (0x6044) AI tick |
| 0x00491b50 | `human_intern_use_car` | `Intern_UseCar__7C_humanFb` | 0x00169600 | CONFIRMED | internal enter/leave-vehicle transition; shared magic 0x3dcccccd (0.1f) |
| 0x00492000 | `human_intern_use_railway` | `Intern_UseRailway__7C_humanFb` | 0x00169bd0 | CONFIRMED | railway-carriage sibling; same vehicle-exit teardown + player gate |
| 0x00492620 | `human_intern_use_car_truck` | `_intern_UseCarTruck__7C_humanFP8C_car_exb` | 0x0016a290 | CONFIRMED | board/leave a truck CARGO BED; 5 identical guards incl. both carried_object slots |
| 0x004929a0 | `human_throw_from_car_truck` | `ThrowFromCarTruck__7C_humanFR8S_vectorf` | 0x0016a6d0 | CONFIRMED | eject from the cargo bed; guards kind==4 (car) + seat_index == -1 |
| 0x00492ff0 | `human_update` | `Update__7C_humanFUi` | 0x0016a920 | CONFIRMED | huge (0x9672) per-frame |
| 0x00494150 | `human_change_dead_anim` | `ChangeDeadAnim__7C_humanFv` | 0x0016cef0 | STRUCTURAL | picks the `NoseniMrtvolyMrt*` (carry-a-corpse) clip matching the current gait ³ |
| 0x004942c0 | `human_animation_evaluate_and_dispatch` | `PersonAnim__7C_humanFfbb` | 0x0016cf10 | CONFIRMED | **the animation evaluate+dispatch call** every anim-setting method ends with; the Movement / Use_Actor / Hit / Turn_Turn diffs all rest on it |
| 0x00494c40 | `human_shift_animation` | `ShiftAnimation__7C_humanFi` | 0x0016d950 | CONFIRMED | Patches the `'?'` placeholder in the current clip's file name to `'L'`/`'R'` from the `+0x1e6` side flag and re-acquires the animation, then walks its notify events. Shared magic `strchr(name, 0x3f)`; **names field `+0x1e6` ↔ PS2 `+0x392`**, extending the proven stance band by one. Ratio 3.2 — PS2 inlines the `C_I3D_cache` search the PC factors into `i3d_cache_anim_create` |
| - | - | `NetDirect__7C_humanFUi` | 0x0016dcf0 | base-stub | 8 B overridden virtual, no distinct PC twin |
| 0x00494d60 | `human_change_state` | `ChangeState__7C_humanFv` | 0x0016dd00 | CONFIRMED | - |
| 0x00495040 | `human_game_init` | `GameInit__7C_humanFv` | 0x0016e0b0 | CONFIRMED | - |
| 0x00495c30 | `human_game_done` | `GameDone__7C_humanFv` | 0x0016eaa0 | CONFIRMED | - |
| 0x00496110 | `human_create_dyn_colls` | `CreateDynColls__7C_humanFv` | 0x0016f0f0 | CONFIRMED | build the 2 dyn-coll capsules (+0x688/+0x68c/+0x690) |
| 0x00496340 | `human_erase_dyn_colls` | `EraseDynColls__7C_humanFv` | 0x0016f440 | CONFIRMED | free them |
| 0x004963d0 | `human_deactivate_colls` | `DeactivateColls__7C_humanFb` | 0x0016f500 | CONFIRMED | unregister from the collision world (mask-2 objects kept); does not free |
| 0x00496430 | `human_activate_colls` | `ActivateColls__7C_humanFi` | 0x0016f5c0 | CONFIRMED | (re)register; no-op while in a vehicle; mode 2 = mask-1 filtered |
| 0x004964b0 | `human_update_dyn_coll` | `UpdateDynColl__7C_humanFv` | 0x0016f700 | CONFIRMED | per-tick capsule refresh from the animated frames; pinned from both sides as the tail call of `Extern_AI_Update` |
| 0x00496710 | `human_hit` | `Hit__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0016fa50 | CONFIRMED | main hit handler (0x5384) |
| - | - | `HitLanding__7C_humanFv` | 0x00170f60 | INLINED ¹ | inlined into `human_hit`: the arm gated on its ninth argument (landing clip `0x67`, PS2 `0x45`) |
| 0x00498760 | `human_hit_in_car` | `HitInCar__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00171060 | CONFIRMED | damage path while INSIDE a vehicle. **The ONLY one of the 7 Hit* helpers that is a real function on PC** |
| - | - | `HitAgony__7C_humanFRC8S_vector` | 0x00171340 | INLINED ¹ | inlined into `human_hit`: the second arm; scream `+0x5e` is used by no other `Hit*` helper |
| - | - | `HitStroke__7C_humanFRC8S_vectorUi` | 0x00171470 | INLINED ¹ | inlined into `human_hit`: the fourth arm (π/4 and 3π/4 split, clips `0xcd` / `0xec..0xf0`, PS2 `0x9b` / `0xba..0xbe`) |
| - | - | `HitGun__7C_humanFRC8S_vectorUib` | 0x00171670 | INLINED ¹ | inlined into `human_hit`: the final `else` arm and both in-vehicle arms; the only helper that screams a constant (`0x54`) |
| - | - | `HitGunHPT__7C_humanFRC8S_vectorUib` | 0x00171af0 | INLINED ¹ | called only from inside HitGun, so it is inlined within HitGun's own inlined body |
| - | - | `HitFall__7C_humanFRC8S_vectorUi` | 0x00171ea0 | INLINED ¹ | inlined into `human_hit` (the third arm, gated on its fifth argument: π/4 and 3π/4 split, clips `0x6a..0x6c`, PS2 `0x48..0x4a`) and into `human_throw_from_car_truck`, whose PS2 twin ends in `HitFall(...)` |
| 0x00498a00 | `human_collision` | `Collision__7C_humanFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00172250 | CONFIRMED | - |
| 0x004991c0 | `human_is_possible_coll` | `IsPossibleColl__7C_humanFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x001728c0 | CONFIRMED | - |
| 0x00499440 | `human_pose_interpolate` | `PoseInterpolate__7C_humanFUi` | 0x00172b60 | CONFIRMED | per-frame driver of the PROCEDURAL POSE layer (additive aim/lean over the playing clip). Exact sig `(this,uint)` + exact callee counts |
| 0x00499b90 | `human_pose_set_pose_normal` | `PoseSetPoseNormal__7C_humanFRC8S_vector` | 0x00173330 | CONFIRMED | set the NORMAL/relaxed upper-body pose. Carries **1.5707964 (π/2)**, the constant its twin carries and Aimed does not |
| 0x0049a400 | `human_pose_set_pose_aimed` | `PoseSetPoseAimed__7C_humanFR8S_vector` | 0x00173a50 | CONFIRMED | set the AIMED upper-body pose (writes back through a non-const `S_vector&`). Carries **1.0471976 (π/3)** |
| 0x0049aac0 | `human_pose_null_rotate` | `PoseNullRotate__7C_humanFv` | 0x00173ff0 | CONFIRMED | reset the procedural POSE rotation to neutral; proven by the Use_Actor diff (×3 both sides) |
| 0x0049ab50 | `human_pose_validate` | `PoseValidate__7C_humanFb` | 0x00174050 | CONFIRMED | on/off switch for the pose layer: blend weight 1.0f/0.0f into the frame + matrix refresh |
| 0x0049ab90 | `human_get_anim_id_for_death` | `GetAnimIDForDeath__7C_humanFibRC8S_vectorRC8S_vector` | 0x001740c0 | CONFIRMED | pick the DEATH anim clip (front/back by AngleTo vs π/2, + random variant) |
| 0x0049ac70 | `human_movement` | `Movement__7C_humanFUi` | 0x00174230 | CONFIRMED | **top-level LOCOMOTION driver** (0x5152 B). Callee multiplicities match exactly; calls human_movement_intern once |
| 0x0049bbc0 | `human_movement_intern` | `MovementIntern__7C_humanFUi` | 0x00175650 | CONFIRMED | **inner locomotion integrator — LARGEST C_human method** (0x6188 B). Motion integration + fall damage (→ hud_set_player_health_value) + impact sfx/particles |
| 0x0049d540 | `human_set_next_coll_owners` | `DynSetNextCollOwnerMult__7C_humanFv` | 0x00176e80 | CONFIRMED | Publishes the human's dyn-coll capsules (plus the `ai_ghost` ignore list) as `g_collision`'s next-query owner set. Ratio 1.17. Names three fields — see §2a. It is the class's ONE rank inversion: PS2 emits it before `TestMoveColl`, PC after |
| 0x0049d0c0 | `human_test_move_coll` | `TestMoveColl__7C_humanFRC8S_vectorRC8S_vectorR8S_vector` | 0x00176fb0 | CONFIRMED | sweep-test a proposed move against the world (from, to, out-hit) |
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
| 0x0049fef0 | `human_inventory_truncate` | `InventoryTruncate__7C_humanFv` | 0x0017b440 | CONFIRMED | clamp carried ammo to the game-item table max. PS2 loops the inventory vector; PC does the equipped slot (factoring difference) |
| 0x0049fff0 | `human_drop_out_items` | `DropOutItems__7C_humanFRQ23gif22vector<10S_GameItem,8>` | 0x0017b4f0 | CONFIRMED | materialize inventory items on the ground |
| 0x004a0740 | `human_find_blocking_drawbridge` | `TozNaselJsemTenNejvyssiMost__7C_humanFv` | 0x0017c040 | CONFIRMED | find a nearby **C_bridge** (kind 0x14) within 100u in state 3–5 while in a car with flag bit 25 (`car+0x294`). Sole consumer `human_use_actor` uses a hit to **REFUSE car exit** — you can't get out on a moving drawbridge. Shared magic bit `0x2000000` |
| 0x004a0840 | `human_can_exit_car_free_space` | `Can_DropOutFromCar_Free__7C_humanFP5C_cari` | 0x0017c220 | CONFIRMED | is there FREE SPACE beside the **GIVEN** car (argument) at this seat? **2.5** clearance. Sole consumer = the carjack eject |
| 0x004a0b10 | `human_can_exit_car_at_seat` | `Can_DropOutFromCar__7C_humanFi` | 0x0017c5c0 | CONFIRMED | can I exit **MY** car (`+0x98`) at this seat? seat<0 = my seat (`+0xac`); **1.2** clearance. Sole consumer = `human_use_actor` |
| 0x004a0e10 | `human_can_enter_car` | `Can_EnterToCar__7C_humanFP5C_carib` | 0x0017c990 | CONFIRMED | can I get INTO the given car at this seat? (4 params — the discriminator vs its 2/3-param siblings) |
| 0x004a11f0 | `human_climb_out_of_car` | `ZasrateLezeniZAuta_ve2rano__7C_humanFv` | 0x0017ce50 | CONFIRMED | the climb-out sequence: engine off, unlink from the police tracker, raycast the exit spot, place via `human_eject_out_of_car` |
| 0x004a1770 | `human_eject_out_of_car` | `ZasrateVybliknutiZAuta_ve3rano__7C_humanFv` | 0x0017d570 | CONFIRMED | compute + apply the exit position (11× vec3_set_xyz, 4× collision_test_line_vertical_thunk). The placement half of `human_climb_out_of_car` |
| 0x004a2400 | `human_do_climb_in_car_lr` | `Do_ClimbInCarLR__7C_humanFv` | 0x0017e570 | CONFIRMED | climb across the car interior L↔R. SetOwner×3 / ChangeWeaponModel×2 exact |
| 0x004a26d0 | `human_use_actor` | `Use_Actor__7C_humanFP7C_actoriii` | 0x0017e920 | CONFIRMED | **the general 'interact with that actor' dispatcher** (door/car/lever/body). Exact 5-arg sig + PersonAnim×3 + PoseNullRotate×3 |
| - | - | `IsKickToCocotus__7C_humanFv` | 0x0017fe50 | PS2-ONLY ⁸ | 1168 B: kick a dog or a downed human within 1.5 units; §4c |
| 0x004a3ae0 | `human_dispatch_weapon_action_at_target` | `Do_Shoot__7C_humanFbP8S_vector` | 0x001802f0 | CONFIRMED | fire the equipped weapon at target (documented twin) |
| 0x004a4490 | `human_do_throw_grenade` | `Do_ThrowGrenade__7C_humanFR8S_vector` | 0x00181150 | CONFIRMED | throw the equipped throwable; gates on weapon_slot == 5 or 0xf (the throwable ids in predmety.def) |
| 0x004a4560 | `human_can_special_stroke` | `Can_SpecialStroke__7C_humanFb` | 0x00181290 | CONFIRMED | find a victim for a special melee takedown: kind ∈ {0x1b, 2}, alive, NOT crouched, movement_mode ∈ {1,2,9} |
| 0x004a49d0 | `human_do_castrate` | `Do_Castrate__7C_humanFP7C_human` | 0x00181760 | CONFIRMED | groin-strike melee; PAIRED anim (attacker 0xf1 / victim 0xf2). Identical prologue to hard_head → separated by SIBLING-DELTA + user override |
| 0x004a4aa0 | `human_do_hard_head` | `Do_HardHead__7C_humanFP7C_human` | 0x001818b0 | CONFIRMED | headbutt melee; PAIRED anim (attacker 0xf3 / victim 0xf4). Same trap, same resolution |
| - | - | `SetWaitAnim__7C_humanFv` | 0x00181a00 | INLINED ¹ | 108 B: the 250 ms wait block, inlined into `human_dispatch_weapon_action_at_target`; §4c |
| - | - | `SetFreeBoxingAnim__7C_humanFUi` | 0x00181a70 | INLINED ¹ | 772 B: the free-boxing clip block, inlined into `human_dispatch_weapon_action_at_target`; §4c |
| 0x004a4b70 | `human_shooting` | `Shooting__7C_humanFv` | 0x00181d80 | CONFIRMED | **the per-frame FIRING tick** (one of the largest methods). Calls `police_manager_register_gunfire_offence_for_target` — firing is what makes the cops notice you |
| - | - | `FireReload__7C_humanFP15S_GameItemTable` | 0x00183120 | INLINED ¹ | 932 B: inlined into the animation-event-1 arm of `human_ai`; §4c |
| 0x004a60b0 | `human_do_reload` | `Do_Reload__7C_humanFv` | 0x001834d0 | CONFIRMED | reload the weapon; sets `weapon_position_dirty` (+0x580) |
| 0x004a61d0 | `human_weapon_toggle_held` | `Do_Holster__7C_humanFv` | 0x001836b0 | CONFIRMED | re-select/holster held weapon (from `human_free_hand` diff) |
| 0x004a6300 | `human_weapon_drop` | `Do_WeaponDrop__7C_humanFv` | 0x00183840 | CONFIRMED | discard weapon to ground (throw-away anim) |
| - | - | `Do_WeaponDropAll__7C_humanFv` | 0x00183b90 | PS2-ONLY ⁸ | 680 B: drop every item of a dying non-player; §4c |
| 0x004a63f0 | `human_do_play_anim` | `Do_PlayAnim__7C_humanFPCcbb` | 0x00183e40 | CONFIRMED | play a named animation clip |
| 0x004a64b0 | `human_get_bound_points_2d` | `GetBoundPoints2D__7C_humanFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x00183f40 | CONFIRMED | - |
| 0x004a6700 | `human_do_jump` | `Do_Jump__7C_humanFbb` | 0x00184090 | CONFIRMED | Start a jump, or hand off to `human_do_climb`. Three identical guards; `movement_mode = 0xb` (same constant both builds); vertical channel 6.0f; identical teardown tail. Ratio 1.15. Sets the `+0x1e6` side flag from which foot is down |
| 0x004a6970 | `human_do_climb` | `Do_Climb__7C_humanFv` | 0x00184360 | CONFIRMED | Mount a ledge in front: probe at `y+0.3` along `forward*0.7`, retry at `y+0.7`, require a drop of 0.3–1.1 and `AngleTo ≤ 0.5`, three swept-sphere clearances, then COMMIT (zeroes `+0x420..+0x43c`, dispatches PersonAnim). Ratio 1.22 |
| 0x004a75b0 | `human_do_side_jump` | `Do_SideJump__7C_humanFRC8S_vectorRC8S_vector` | 0x00185260 | CONFIRMED | Side-jump/dodge: only with free hands (`+0x6f8`/`+0x700` == 0) and not already in the side-jump anim range (`+0x70` ∉ [0x142,0x149]); picks the clip from the facing-vs-target angle, writes it to `+0x74`. **Shared magic π/4 (`0.7853982`) & 3π/4 (`2.3561945`) both builds** + the free-hands guard + the 8-wide anim-range gate |
| 0x004a7700 | `human_change_model` | `_intern_ChangeModel__7C_humanFPcb` | 0x00185420 | CONFIRMED | **swaps the human's 3D model** and runs the subclass half ⁴ |
| 0x004a7e40 | `human_force_death` | `_intern_ForceDeath__7C_humanFv` | 0x00186050 | CONFIRMED | force a death/ragdoll pose immediately, bypassing the normal damage path; picks a random death variant ⁵ |
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
| 0x004aad00 | `human_can_aim` | `Can_Aimed__7C_humanFi` | 0x001891a0 | CONFIRMED | is there room in front to raise the weapon? ⁶ |
| 0x004aaed0 | `human_do_corner_breath` | `Do_CornerBreath__7C_humanFQ27C_human13E_corner_view` | 0x001892d0 | CONFIRMED | **COVER SYSTEM.** idle breathing pose against cover; clips 0x4e–0x4f |
| 0x004aaf10 | `human_do_corner_view` | `Do_CornerView__7C_humanFQ27C_human13E_corner_view` | 0x00189340 | CONFIRMED | **COVER.** lean out to look; variant 2 requires CROUCHED; clips 0x50–0x52 |
| 0x004aafd0 | `human_do_corner_fire` | `Do_CornerFire__7C_humanFQ27C_human13E_corner_view` | 0x00189430 | CONFIRMED | **COVER.** lean out and fire (blind); clips 0x53–0x56 |
| 0x004ab0f0 | `human_do_corner_fire_aimed` | `Do_CornerFireAimed__7C_humanFQ27C_human13E_corner_view` | 0x001895a0 | CONFIRMED | **COVER.** lean out and fire AIMED; clips 0x58–0x59 |
| 0x004ab1c0 | `human_force_ready_to_act` | `Force_CanWorking__7C_humanFv` | 0x001896d0 | CONFIRMED | force `movement_mode` (+0x40c) = **1** = the idle/READY mode. Consumer: **`entity_action_tick`** — makes the NPC available for a new task |
| 0x004ab1d0 | `human_turn_in_place` | `Turn_Turn__7C_humanFv` | 0x001896e0 | CONFIRMED | **rotate on the spot**: request the turn clip, zero motion_intent + the +0x430 motion vector. Consumers: `entity_action_tick` ×6, script label `c21_postrotate_facing` |
| - | - | `Turn_Left__7C_humanFv` | 0x00189710 | PS2-ONLY ⁸ | 48 B; §4c |
| - | - | `Turn_Right__7C_humanFv` | 0x00189740 | PS2-ONLY ⁸ | 48 B; §4c |
| 0x004ab200 | `human_activate` | `Activate__7C_humanFv` | 0x00189770 | CONFIRMED | lifecycle **activate**: frame on, `human_pose_validate`, register with the zone/AI-loc system |
| 0x004ab8a0 | `human_deactivate` | `Deactivate__7C_humanFv` | 0x00189c50 | CONFIRMED | lifecycle **deactivate**: frame off, release 2 object registrations, free +0x5f8 |
| 0x004ab9e0 | `human_new_scream` | `NewScream__7C_humanFUi` | 0x00189d90 | CONFIRMED | play a voice/scream line by id (guards + `%08lu.wav`). Its 4 calls inside human_hit ARE the inlined Hit* helpers' internals |
| 0x004abc00 | `human_do_box_drop` | `Do_FuckingBoxDrop__7C_humanFv` | 0x0018a440 | CONFIRMED | materialize carried object as scene entity |
| 0x004abd50 | `human_do_dead_body_drop` | `Do_DeadBodyDrop__7C_humanFv` | 0x0018a690 | CONFIRMED | put a carried body down; guards on carried_object_2 (+0x700); calls `human_do_dead_body_pick_up(0)` = release |
| 0x004abde0 | `human_reset_actions` | `ResetActions__7C_humanFv` | 0x0018a730 | CONFIRMED | cancel all actions → neutral idle: un-crouch, clear aimed, idle clip **1** (same id on both builds), PoseNullRotate |
| - | - | `PickObject__7C_humanFR8S_vector` | 0x0018a7c0 | base-stub | 8 B overridden virtual, no distinct PC twin |
| - | - | `UniversalUse__7C_humanFv` | 0x0018a7d0 | base-stub | 8 B overridden virtual, no distinct PC twin |
| 0x004abe40 | `human_recompile_death_pos` | `RecompileDeathPos__7C_humanFv` | 0x0018a7e0 | CONFIRMED | **DEATH.** recompute where a dead body settles. PS2 (7528 B) INLINED its frame helpers; PC keeps them separate (`human_death_pose_align_to_ground`, `human_death_pose_push_out_of_wall`, `human_death_pose_settle_to_ground`) — sum ×1.37. 2 callers on BOTH sides |
| 0x004ad3c0 | `human_process_death_pos` | `ProcessDeathPos__7C_humanFUi` | 0x0018c550 | CONFIRMED | **DEATH.** per-tick settling toward the computed death pose; identical guard on the +0xa50/+0xa54 pair |
| 0x004ad4b0 | `human_is_this_car_stealing` | `IsThisCarStealing__7C_humanFP5C_car` | 0x0018c670 | CONFIRMED | is anyone already taking this car? scans actors of kind ∈ {2, 0x1b} for a reference via +0x478 / +0x98 / +0x9c |
| 0x004ad520 | `human_free_hand` | `FreeHand__7C_humanFv` | 0x0018c720 | CONFIRMED | holster + parent physics prop to l_hand/r_hand |
| 0x004ad770 | `human_weapon_effect_no_magazine` | `BezZasobnikuWeaponEffect__7C_humanFiP9I3D_model` | 0x0018ca90 | CONFIRMED | **WEAPONEFFECT.** hide the MAGAZINE geometry — the PS2 name is Czech for "without magazine"; weapon ids 9 and 10 |
| 0x004ad7c0 | `human_start_weapon_effect` | `StartWeaponEffect__7C_humanFi` | 0x0018cb10 | CONFIRMED | **WEAPONEFFECT.** spawn + place a weapon visual effect on the weapon bone |
| 0x004adf70 | `human_tick_weapon_effect` | `TickWeaponEffect__7C_humanFUi` | 0x0018d400 | CONFIRMED | **WEAPONEFFECT.** per-frame advance of live effects |
| 0x004ae460 | `human_reset_weapon_effects` | `ResetWeaponEffects__7C_humanFv` | 0x0018dc10 | CONFIRMED | **WEAPONEFFECT.** tear down all live effects (called from the hit/death paths). Exact sig + exact x2/x2/x2 marker counts |
| 0x004ae710 | `human_fake_weapon_effect` | `FakeWeaponEffect__7C_humanFP9I3D_frameib` | 0x0018e060 | CONFIRMED | **WEAPONEFFECT.** non-simulated effect on a frame; id 9 only; attaches to the sub-frame `"zaver1"`, Czech for a gun's BOLT |
| - | - | `HideLegs__7C_humanFP9I3D_modelb` | 0x0018e100 | base-stub | 8 B overridden virtual, no distinct PC twin |
| 0x004ae770 | `human_is_hit_allowed_between_types` | `Can_Hit_FilterHumanType__7C_humanFP7C_humanP7C_human` | 0x0018e110 | CONFIRMED | **may type A hit type B?** 11×11 lookup in `g_hit_permission_matrix` indexed by each human's single-bit `human_type` (+0x5fc). Special rule: type 1 may hit type 0 only if the victim is the ANGRIEST actor |
| - | - | `ChangeSceneInit__7C_humanFv` | 0x0018e230 | PS2-ONLY ² | the `C_actor` virtual after `SoundsChange` (PS2 slot 27); the PC table has no slot for it |
| - | - | `ChangeSceneDone__7C_humanFv` | 0x0018e2b0 | PS2-ONLY ² | PS2 slot 28, the last `C_actor` virtual; the PC table has no slot for it |
| - | - | `PlaySoundFromSndTable__7C_humanFi` | 0x0018e350 | INLINED ¹ | 180 B: pulls a preloaded sound out of `C_sound_table` and positions it 1.5 units up ⁷ |
| - | - | `SchovejPero__7C_humanFv` | 0x0018e410 | PS2-ONLY ² | model **TEARDOWN** across a city-part load (`I3D_pose::Release` ×3, `StopAnims`, `EraseDynColls`, `LightnessDel`) |
| - | - | `VystrcPero__7C_humanFv` | 0x0018e920 | PS2-ONLY ² | model **REBUILD** after a city-part load (string-builds the model name, `_intern_ChangeModel`, `InvalidateSelf`). It is also why a whole-binary CRT string fingerprint for it returns 0 PC candidates: there is nothing to find |
| 0x00425710 | `human_get_actor_speed` | `GetActorSpeed__7C_humanFR8S_vector` | 0x0018f330 | CONFIRMED | 0x12 B; the `C_human` override of the `C_actor` base stub. Sits ~380 KB BELOW the C_human VA cluster (§4c) |

¹ **INLINED is terminal: there is no PC function to find.** MSVC spliced these bodies into their
callers, so a PC/PS2 count gap here is a FACTORING difference, not a missing function. Two independent
proofs for the `Hit*` family: (1) PS2 `Hit` calls `NewScream` **zero** times — all 11 of its calls live
inside the helper bodies — yet PC `human_hit` calls `human_new_scream` 4× and `human_movement` 4×
*directly*; (2) PS2 `ThrowFromCarTruck` calls `HitFall`, yet PC `human_throw_from_car_truck` calls no
unnamed function, just those same internals. Every unnamed callee of `human_hit` was fingerprint-tested
and none touches `cur_health` or the scream helper. The inline sites are marked in the decomp under a
banner inside `human_hit`. **Each arm is pinned** by position and by its clip ids: `human_hit` keeps the
PS2 dispatch order (landing, agony, fall, stroke, else gun), and every arm's clip ids are the PS2 ids
plus a constant band offset (`+0x22` for the landing and fall clips, `+0x32` for the stroke clips). The
five helpers hold 10 scream calls and `human_hit` only 4 because MSVC merged the identical scream tails:
the arms jump to three shared tails (`rand + 0x5c`, `rand + 0x55`, and a computed id). `SetWaitAnim`,
`SetFreeBoxingAnim` and `FireReload` are inlined the same way; §4c lists their sites.

² **PS2-ONLY: a PS2 method count is an UPPER BOUND on the PC class, never a target.** Every caller of
these four is the PS2 city-streaming subsystem (`InterCity_ChangeMission__9C_mission`,
`ChangeScene*__10C_shvestky`, `InternMission_Vykur/Spolkni`). The PS2 build splits the city into parts
with a load between them to fit 32 MB of RAM and the PC build does not: `InterCity` appears **127×** in
the PS2 decomp and **0×** on PC. `SchovejPero` / `VystrcPero` are crude developer slang, roughly "put it
away" / "stick it out"; the bodies are the model teardown/rebuild pair that streaming needs, and they
are large (980 / 2568 B), so their absence from the PC build is not inlining. For `ChangeSceneInit` /
`ChangeSceneDone` the PC vtable settles it: the PC `C_human` table maps slot for slot onto the PS2 one
through `SoundsChange` and then holds one more slot, the two-argument network-event virtual
(`door_apply_net_event_msg`, `car_apply_net_event_msg` and `physical_object_apply_net_event_msg` fill it
in their classes), with no slot for either method.

³ `human_change_dead_anim`: the gaits are Walk/Back/Left/Right/StrafL/StrafR, plus Nahozeni (heave
up) / Shozeni (put down) / Stat. Gated on `+0x700` (the carried body) AND `+0x704`. The **PS2 body
is a 28-byte `return;` stub** — the port gutted it, so the twin is a SYMBOL match and can never be a
body match.

⁴ `human_change_model` caches the old model's basename at `+0x5f8` (`strrchr(path,'\\')+1`),
invalidates the actor, drops shadow frames + fake shadow, stops all anim tracks, erases dyn-colls,
then `i3d_cache_model_open`s the replacement and runs the subclass half: `player_change_model` if this
human is the ACTIVE PLAYER (`*(g_act_mission+0x24)+0xe4` == this), `entity_rebind_neck_bone_handle`
if `kind`(+0x10) == 0x1b. Established from the caller side: identical `strrchr(name,0x5c)` head +
identical teardown order; `+0x68`↔`+0x70` and `+0x5f8`↔`+0x65c` both at the proven 0x64 band delta.
PS2 calls `strlen`/`strcpy` as newlib calls where MSVC compiled them as intrinsics, so a CRT callee
fingerprint finds nothing here.

⁵ `human_force_death` is separate from the `Death` virtual. Established from the caller side:
`game_process_death_actors` (= `DeathActorProcess__6C_game`) calls it in the exact
`_intern_ForceDeath` position on both builds. It ends by setting the anim-machine lock
(`structs/anim_machine.md` §3).

⁶ `human_can_aim`: frame world pos at `y + 1.5`, forward direction, one horizontal line test on mask
`0x80000`, `return hit == 0`. The call graph clinches it: PS2's callers `AI__8C_player` and
`EineMeineKleineAutoInLuft` both appear on the PC side. PS2 caches the answer behind
`Can_Aimed(1000)` and lowers the probe to 0.8 when crouched; PC does neither.

⁷ `PlaySoundFromSndTable`: its two PS2 callers are `StartWeaponEffect` (= `human_start_weapon_effect`)
and `FireReload`. Proof of the inlining: the distinctive frame-flag rewrite
`& 0xfffffedf | 0x40000000` that lives INSIDE the PS2 body appears directly in PC
`human_start_weapon_effect` with no intervening sound call — the body was spliced in. Same proof
pattern as the `Hit*` family.

⁸ **PS2-ONLY, measured over the callers:** each of these four has no PC function, and no PC body carries
an inlined copy at the PS2 call positions. §4c holds the measurement per method.

### 4a. Coverage, and how the mapping was built

**146 PS2 methods:** **123 CONFIRMED · 1 STRUCTURAL · 10 INLINED · 8 PS2-ONLY · 4 base-stub**,
counted off the rows above; every PS2 method is accounted for. §4c measures the seven that have no PC
function of their own outside the `Hit*` family and the city-streaming set.

Both builds emit a class's methods in **source order**, so inside the class's code cluster the PS2 VA
sequence and the PC VA sequence are the same sequence — `human_set_next_coll_owners` is the class's one
rank inversion. Confirmed twins therefore act as anchors, and a gap between two consecutive anchors
holding N unmapped PS2 methods and N unnamed PC functions has a forced rank-to-rank mapping. That
ordering only **generates** candidates; every CONFIRMED row above was body-diffed on both sides
afterwards, and the bodies have refuted rank proposals more than once. Three things limit the lever:

* A **foreign function inside a gap** shifts every rank below it. The `Do_Shock*` gap holds two, one of
  them `human_release_force_feedback_effects` — a 4-call cleanup stub that takes no arguments and is not
  a `C_human` method at all (§4c).
* A **size ratio outside ~1.0–1.6** is the tell that the ranks are shifted, rather than that the
  compiler did something exotic.
* The PC **tail after the last anchor** is not a gap and is invisible to a between-anchors walk; the
  WeaponEffect family sits there and had to be enumerated by hand.

A class's PC code is also **not one contiguous cluster** — MSVC emits per translation unit and the
linker concatenates the objects — which is why `human_get_actor_speed` sits ~380 KB below the rest.

### 4b. The `Do_Corner*` (cover) and `Do_Shock*` (stun) families

Two complete four-method families, both body-diff CONFIRMED.

#### `Do_Corner*` — the COVER SYSTEM (4/4)

| PC name | PS2 twin | anim ids |
|---|---|---|
| `human_do_corner_breath` | `Do_CornerBreath` | 0x4e–0x4f |
| `human_do_corner_view` | `Do_CornerView` | 0x50–0x52 |
| `human_do_corner_fire` | `Do_CornerFire` | 0x53–0x56 |
| `human_do_corner_fire_aimed` | `Do_CornerFireAimed` | 0x58–0x59 |

**`E_corner_view`** (the PS2 mangled signature names the enum) — the lean/peek variant, values **0 / 1 / 2**.
Shared rules across the family:
- refuses while the human is **carrying anything** (`carried_object` pair `+0x6f8`/`+0x700`);
- writes the chosen clip to `cur_anim_id` (`+0x74`) and returns **1** on success / **0** on rejection;
- **value 2 requires CROUCHED** (`+0x1e4`) — no low peek while standing;
- the two FIRE variants also refuse when `weapon_class` (`+0x1e8`) is **0 or 2** (bare hands or a rifle), and pick the **alternate
  clip (base + 1)** when the human is **not crouched AND the equipped weapon's def record has `+0x50 == 4`**
  (a weapon-category check; the def record comes from `inventory_get_in_hand_item_def` out of
  `g_game_item_table`).

None of the four touches the wanted substate, the wanted bar or anything else pursuit-related.

#### `Do_Shock*` — the SHOCK / STUN reactions (4/4)

| PC name | PS2 twin | mechanism |
|---|---|---|
| `human_do_shock_start` | `Do_ShockStart(int)` | clamp severity 0..3; rand between **two bases 4 apart** (0x2f/0x33) + severity |
| `human_do_shock_down` | `Do_ShockDown(int)` | clamp severity 0..2; clip = **`severity*3 + 0x3d`**, stages a 2nd clip at +1 |
| `human_do_shock_up` | `Do_ShockUp()` | read `+0x70`; accept 0x3e/0x41/0x44 (**spacing +3**); write `+0x74 = id+1` |
| `human_do_shock_stumble` | `Do_ShockStumble()` | read `+0x70`; accept 0x46/0x48/0x4a (**spacing +2**); write `+0x74 = id+1` |

**How the last two are separated.** `Do_ShockUp` and `Do_ShockStumble` are
**structurally identical** (read anim_slot, accept one of three ids, advance by 1, dispatch). A body-diff
*cannot* tell them apart, and the ids are rebased per build so their raw **values** are useless. What
separates them is the **ID SPACING**, which is build-independent because it reflects how many clips sit in
each group: PS2 `ShockUp` 0x1c/0x1f/0x22 → **+3**, matching PC 0x3e/0x41/0x44 → **+3**; PS2 `ShockStumble`
0x24/0x26/0x28 → **+2**, matching PC 0x46/0x48/0x4a → **+2**. VA order corroborates.
⇒ When two siblings are structurally identical, the discriminator has to be a build-INVARIANT derived
property (spacing, count, ratio) rather than a raw value.

### 4c. The `C_human.cpp` translation unit

**TU `[0x0048b5e0, 0x004ae800)`**, bounded by the CRT static-init/atexit pairs `0x0048b5b0` /
`0x0048b5d0` and `0x004ae800` / `0x004ae820`. Finding the **closing** pair is what fixes the extent.

**157 real starts, 0 unnamed** — measured by enumerating every PC function start in the range, unwind
entries included, not estimated. 159 apparent, minus the two split phantoms `0x004a663a` and
`0x004a66a0`, both inside `human_get_bound_points_2d`, whose true size is therefore 592 B and not 394 B.

#### What the TU contains, and why it is not all `C_human`

MSVC emits an inline member as a COMDAT into whichever TU instantiates it, and a class defined inside
another class's `.cpp` links with that file. So besides C_human's own methods this object holds:

| also in this TU | why | prefix |
|---|---|---|
| `ai_loc` (4/4), `ai_blob_log` (3/3) | declared inside `C_human.cpp` | their own |
| `inventory_dtor`, `inventory_reset`, `inventory_get_in_hand_item` | `G_Inventory` is EMBEDDED at `C_human+0x480`, so `C_human::~C_human` instantiates its inline members | `inventory_` |
| `vector_dispose_at_0x24` | an ICF fold across `ai_loc` and `ai_logs` | none — no owning class |
| the four force-feedback effects + their create/release pair | played only by `human_*` code | `human_` / `force_feedback_` |
| `clear1_dword_at_0x04` | a 54-callsite COMDAT fold that merely landed here | none — no owning class |

⇒ **The prefix follows the CLASS, never the TU.**

#### The seven methods with no PC function of their own

| PS2 method | size | resolution |
|---|---|---|
| `Turn_Left` / `Turn_Right` | 48 B each | PS2-ONLY: no caller and no data reference on the PS2, no slot in the 39-slot `C_human` vtable at `0x0063c468`, and their turn rate ±2.4 occurs nowhere in `Game.exe` as a float or a double |
| `Do_WeaponDropAll` | 680 B | PS2-ONLY: its one PS2 caller is `Hit`, for a dying non-player; at that position `human_hit` calls `human_weapon_drop` for every human |
| `SetWaitAnim` + `SetFreeBoxingAnim` | 108 + 772 B | INLINED into `human_dispatch_weapon_action_at_target`: the wait block (clip time - 250 ms into `+0x44c`, motion zeroed, mode 5) and the extended-clip block (end time into `+0x234`, `anim_machine_set_extended_anim(clip, 1.0, 4.0, 1)`) sit at every PS2 call position with no intervening call |
| `IsKickToCocotus` | 1168 B | PS2-ONLY: its three calls guard the melee arms of `Do_Shoot` (kick a dog or a downed human within 1.5 units); the PC melee arms are otherwise statement-aligned and carry no such test |
| `FireReload` | 932 B | INLINED into `human_ai`: its one PS2 caller is `AI` (animation event 1), and the PC event-1 arm carries the body statement for statement (`+0x1fc` set, in-car refusal, aim dropped, `+0x590` = rand + 5000, `+0x594` = -1, the standing or crouched reload clip, `anim_machine_set_extended_anim(clip, 1.0, 4.0, id != 8)`) |

### 4d. The embedded `C_ShotSkeleton` at `+0x2dc`

`C_human` embeds a whole `C_ShotSkeleton` (PS2 class name; PC prefix `shot_skeleton_`) at
`+0x2dc`. It is the per-limb hit locator: given a shot ray it returns which **body part**
(1 right arm, 2 left arm, 3 right leg, 4 left leg, 5 torso, 6 head/neck) was hit, or 0.

**How the offset was established** — the decomp cannot say, because both methods are
`__thiscall` and the receiver is invisible. A disassembly census of the ECX value at every
callsite of `shot_skeleton_init` / `shot_skeleton_test_collision` finds six callsites, and
four are `lea ecx,[reg+0x2dc]`: `human_game_init`, `human_change_model`, `human_ai` and
`game_tick_shoot`. The remaining two are `lea ecx,[esp+N]` inside `pedestrian_crowd_hit`,
which builds one on the stack rather than using a human's.

**Corroboration, from a different direction:** the object is `0x40` (16 joint handles) +
10 × `0x18` (the capsule table) = **`0x130`** bytes, so it spans `+0x2dc..+0x40c` — ending
exactly where the documented `movement_mode` at `+0x40c` begins. Same shape of argument as
the `anim_machine` at `+0xbc` (§1).

Full write-up, including the decoded joint names and capsule table:
`exe-re/structs/shot_skeleton.md`.

## 5. Related

* `structs/actor.md` — the `C_actor` base (kind 1) · `structs/entity.md` — the kind enum and the
  `C_entity` AI layer above · `structs/player.md` — the `C_player` layer above.
* The embedded sub-objects, one file each: `structs/anim_machine.md` (`+0xbc`) ·
  `structs/shot_skeleton.md` (`+0x2dc`) · `structs/inventory.md` (`+0x480`) · `structs/ai_loc.md`
  (`+0x724`) · `structs/ai_blob_log.md` (`+0x758`).
* PC names and VAs: `exe-re/ghidra/Game-ghidra.exe.functions.md`. PS2 mangled symbols and bodies:
  `exe-re/ghidra/SLUS_206.71.c`.

