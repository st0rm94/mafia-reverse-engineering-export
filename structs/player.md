# `C_player` — the player actor

**Created** 2026-07-12 14:05 +0200 · **Modified** 2026-09-24 21:39 +0200

**PC prefix: `player_`.** Type `C_player`, derived from `C_human` (its ctor tail-calls
`human_ctor`). It is the player-controlled human: on-foot/driving/in-car control, sniper mode,
control lock/fade, the car-damage bar, and the player SaveGame block.

- **kind value:** `+0x10 = 2` (PC) — stamped by `player_ctor` over `C_human`'s 3, byte-confirmed in
  `structs/entity.md` §1.
- **PC vtable:** `g_player_vt`.
- **PC translation unit:** `C_player.cpp`, `[0x004c6b80, 0x004cdd70)` — 41 starts, all named.
- **Lifecycle:** `player_ctor` (`0x004c6b90`), `player_dtor` (`0x004c6c40`),
  `player_scalar_deleting_dtor` (`0x004c6c20`).
- **Coverage:** 44 PS2 methods, all accounted (§3): 33 CONFIRMED, 3 STRUCTURAL, 5 PS2-only,
  1 inlined, 1 inlined-candidate, 1 not located.

All PS2 material lives in §7 and in §3's twin table.

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

**`+0x604`, `+0x608..+0x614` and `+0x644` are `C_human` fields, not `C_player` fields** — max
health, the four limb energies and current health. `player_ctor` only OVERRIDES max health to
200.0f (double an NPC's 100.0f) and the four limb energies to 40.0f each. They are documented in
`structs/human.md` (§2b for the limb energies, which are `EnergyHandL` / `EnergyHandR` /
`EnergyLegL` / `EnergyLegR`, entries 2..5 of C_human's 15-slot property vector).

---

## 1. `C_player` field layout (player band; relative to the object base)

C_player-specific fields (the object also carries the full C_actor + C_human bands documented in
`actor.md` / `human.md`). PC offsets; verified from the `player_ctor` body.

```c
typedef struct C_player {                   // extends C_human; PC offsets, C_player band only
    /* +0x00  */ void   *vtable;                // [H] C_player vtable = g_player_vt (player_ctor sets)
    /* +0x10  */ int32_t kind;                  // [H] = 2 (player_ctor writes over the C_human 3)
    /* +0x628 */ float   field_628;             // [M] = 1.0f (player_ctor)
    /* +0x680 */ int32_t field_680;             // [M] = 3 (player_ctor)
    /* +0xa9c */ uint16_t drive_primary_bits;   // [H] one bit per driving axis (1 steer, 2 accel,
                                                //     4 brake, 8 clutch): set while the PRIMARY key
                                                //     binding owns the axis. First field of the save
                                                //     record; player_game_init writes 0xffff
    /* +0xab0 */ int32_t field_ab0;             // [M] = 200 (player_ctor)
    /* +0xab4 */ I3D_frame *model_base_frame;   // [H] cached "base" sub-frame of the player model;
                                                //     re-resolved by player_change_model, accepted only
                                                //     if type==1 and FourCC(+0x1d4) is 'SNGM'/'SMMR'
    /* +0xab8 */ float   alpha;                 // [H] CURRENT ALPHA (fade level), 1.0 = opaque.
                                                //     player_fade_reset sets 1.0f; player_add_alpha_object
                                                //     re-applies it when < 1.0; player_change_model resets it.
    /* +0xabc */ float   alpha_partner;         // [M] second float, also set to 1.0f by player_fade_reset
                                                //     and by player_save_game_load. Role [U]
    /* +0xac0 */ struct {                       // [H] THE ALPHA-OBJECT LIST — MSVC std::vector,
        uint8_t alloc;                          //     4 words. `lea edx,[this+0xac0]` is the argument
        void  **begin;   /* +0xac4 */           //     player_add_alpha_object / _del_alpha_object hand
        void  **end;     /* +0xac8 */           //     to their helper. Elements are `I3D_object*`.
        void  **cap;     /* +0xacc */
    } alpha_list;
    /* +0xad0 */ float   shot_accuracy_spread;  // [H] accumulated per-shot accuracy penalty (crosshair
                                                //     bloom). player_add_shot_accuracy_penalty ADDs
                                                //     weapon_record+0x84; player_ai decays it. Pushed to
                                                //     the HUD as spread*3+1 via hud_set_player_accuracy.
                                                //     Clamped: >1.0 re-seeds to ~0.8..1.0.
    /* +0xad4 */ int32_t shot_spread_window_ms; // [H] = 3000 ms on each shot
    /* +0xad8 */ int32_t action_button_scratch; // [M] used x2 by player_do_action_button_in_car
    /* +0xadc */ int32_t car_damage_window_ms;  // [H] = 3000 ms on show, -1 on invalidate
    /* +0xae0 */ C_car  *car_damage_target;     // [H] the car whose damage bar is displayed; latched by
                                                //     player_show_car_damage_bar, cleared by
                                                //     player_invalidate_car_damage_bar
    /* +0xada */ uint8_t controls_locked;       // [H] player_lock_controls; last field of the save record
    /* +0xaf0 */ uint32_t last_shown_weapon;    // [H] last weapon slot announced by player_show_weapon_name;
                                                //     compared against the equipped slot at +0x4a0
} C_player;                                     // partial — C_player band only
```

### 1a. The C_player band has a 4-byte packing shift

The PC↔PS2 offset delta for the C_player band is **not constant**. Eight field pairs proven by
body-diff split cleanly in two:

| PC offset | PS2 offset | delta | proven by |
|---|---|---|---|
| +0xab4 | +0xc38 | **0x184** | `player_change_model` |
| +0xab8 | +0xc3c | **0x184** | `player_change_model` |
| +0xac8 | +0xc48 | **0x180** ¹ | `player_change_model` |
| +0xad0 | +0xc50 | **0x180** | `player_add_shot_accuracy_penalty` |
| +0xad4 | +0xc54 | **0x180** | `player_add_shot_accuracy_penalty` |
| +0xad8 | +0xc58 | **0x180** | `player_do_action_button_in_car` |
| +0xadc | +0xc5c | **0x180** | `player_show_car_damage_bar` / `_invalidate_car_damage_bar` |
| +0xae0 | +0xc60 | **0x180** | `player_show_car_damage_bar` / `_invalidate_car_damage_bar` |

¹ Same list, not literally the same field: PC `+0xac8` is the vector's END POINTER and PS2 `+0xc48`
is its SIZE. `player_change_model`'s `+0xac8 = +0xac4` is `end = begin`, i.e. clear the list, which
is what PS2's `+0xc48 = 0` does.

⇒ **delta 0x184 at/below PC `+0xabc`; delta 0x180 at/above PC `+0xad0`.**

**The shift is the alpha-object list.** PC `+0xac0` is an MSVC `std::vector` in its 4-word form —
`{alloc, begin, end, cap}`, **16 bytes** — against PS2 `+0xc44`, a 3-word `__vector_pod`, **12
bytes**. `0xad0 − 0xac0 = 0x10` on the PC; `0xc50 − 0xc44 = 0xc` on the PS2. A PC object that is 4
bytes larger makes every later PC offset 4 bytes higher, so the delta (PS2 − PC) SHRINKS by 4
across it: `0x184 → 0x180`. Nothing is missing on either side; the two builds spell one member
differently.

This explains the save-record map (PS2 `+0xc20..+0xc5a` ↔ PC `+0xa9c..+0xada`, 11 fields, same
order and types, same type-tag `1`), whose *start* implies delta 0x184 and whose *end* implies
0x180: the record straddles the shift. **Use 0x180 for anything above `+0xac8` and 0x184 below
`+0xab8`; do not extrapolate one delta across the whole band.** Any function touching that band
has a C_player receiver. Corollary field map: PS2 `+0x224`↔PC `+0x98` (vehicle link), PS2
`+0x228`↔PC `+0x9c`, PS2 `+0x238`↔PC `+0xac` (seat).

## 2. Field evidence — C_player band

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `player_ctor` | - | = `g_player_vt` |
| +0x10 | kind | H | `player_ctor` | - | = 2 (over the C_human base's 3) |
| +0x680 | field_680 | M | `player_ctor` | - | = 3 |
| +0xa9c | **drive_primary_bits** | H | `player_ai_drive`, `player_game_init` (`0xffff`), `player_save_game_load` | `player_ai_drive` | §5a |
| +0xab0 | field_ab0 | M | `player_ctor` | - | = 200 |
| +0xab4 | model_base_frame | H | `player_change_model` | `player_set_alpha` | Cached `"base"` sub-frame of the player model. Accepted only if the frame's type (+0x110) == 1 **and** its FourCC (+0x1d4) is `'SNGM'` (0x4d474e53) or `'SMMR'` (0x524d4d53) — both constants byte-identical on PS2 |
| +0xab8 | **alpha** | H | `player_fade_reset`, `player_change_model`, `player_save_game_load` | `player_add_alpha_object`, `player_fade_reset` | The **current fade level**, 1.0 = opaque. `player_fade_reset` writes 1.0f then calls `player_set_alpha(1.0f)`; `player_add_alpha_object` re-applies it to a newly registered object when `< 1.0` |
| +0xabc | alpha_partner | M | `player_fade_reset`, `player_save_game_load` | - | Set to 1.0f beside `+0xab8` by both writers. A target alpha or a fade rate would both fit; **no reader is located**, so the role stays [U] |
| +0xac0 .. +0xacc | **alpha_list** | H | `player_add_alpha_object`, `player_del_alpha_object`, `player_change_model` | `player_set_alpha` | The **alpha-object list**: MSVC `std::vector` at `+0xac0` = `{alloc, begin +0xac4, end +0xac8, cap +0xacc}`. `lea edx,[this+0xac0]` is the argument both add/del methods hand their helper. **Elements are `I3D_object*`** — the PC decomp can only show a stride-4 pointer vector; the element type comes from `cbAlpha`'s template argument |
| +0xad0 | **shot_accuracy_spread** | H | `player_add_shot_accuracy_penalty`, `player_ai` | `player_add_shot_accuracy_penalty`, `player_ai` | Accumulated per-shot **crosshair accuracy penalty**. Each shot ADDs `weapon_record+0x84`; `player_ai` decays it on foot. Pushed to the HUD as `spread*3.0+1.0` via `hud_set_player_accuracy` (hud+0x9234 = the scale of the bit-0x20 AIM CROSSHAIR). Clamp: >1.0 re-seeds to `rand*0.2+0.8`. Also mirrored to world `+0xd8` |
| +0xad4 | shot_spread_window_ms | H | `player_add_shot_accuracy_penalty` | - | = 3000 ms, armed on every shot |
| +0xad8 | action_button_scratch | M | `player_do_action_button_in_car` | `player_do_action_button_in_car` | Touched ×2 by the in-car action-button handler (×2 on PS2 too). Role not decoded |
| +0xadc | car_damage_window_ms | H | `player_show_car_damage_bar`, `player_invalidate_car_damage_bar` | - | = 3000 ms on show; = -1 on invalidate |
| +0xae0 | **car_damage_target** | H | `player_show_car_damage_bar`, `player_invalidate_car_damage_bar` | `player_invalidate_car_damage_bar` | The `C_car` whose damage bar is on screen (HUD flag bit **0x10000** = `cardamagevisible`). Show requires `car+0x2031`; invalidate only fires if the passed car IS this one, and keeps the bar up when the player is in a car (`+0x98` kind==4) |
| +0xada | controls_locked | H | `player_lock_controls` | `player_collision`, `player_hit` | Last field of the save record; the guard both overrides test |
| +0xaf0 | last_shown_weapon | H | `player_show_weapon_name` | `player_show_weapon_name` | Cache of the last weapon slot announced on the HUD console; compared against the equipped slot at `+0x4a0` so the message fires only on a change. **PC-only** |

The C_human fields `player_ctor` overrides: `+0x604` max health = 200.0f, `+0x608..+0x614` the four
limb energies = 40.0f each. `+0x644` is current health — `player_save_game_load` clamps it to max
health, then pushes `cur/max*100` to `hud_set_player_health_value`.

---

## 3. `C_player` method family — PC ↔ PS2 (all 44 methods)

PC-first; PS2-only rows leave the PC columns `-`. Status: **CONFIRMED** (body-diffed) ·
**STRUCTURAL** (same slot and role, bodies not diffable) · **PS2-ONLY** · **INLINED** ·
**not located**. Sorted by PS2 VA.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004c6b90 | `player_ctor` | `__ct__8C_playerFv` | 0x001aaed0 | CONFIRMED | base ctor (kind=2, vtable 0063c908, tail-calls `human_ctor`) |
| 0x004c6c40 | `player_dtor` | `__dt__8C_playerFv` | 0x001aaf70 | CONFIRMED | base dtor |
| 0x004c6c80 | `player_init` | `Init__8C_playerFP9I3D_frame` | 0x001ab030 | CONFIRMED | Chains `human_init`, then stamps `human_type` (+0x5fc) = **1** — the player's identity in the 11x11 `g_hit_permission_matrix` (`human.md` +0x5fc; the mask is single-bit, so mask 1 = type index 0) |
| 0x004c6cb0 | `thunk_human_duplicate_from` | `DuplicateFrom__8C_playerFP7C_actor` | 0x001ab070 | CONFIRMED | Both builds override the slot with a bare jump to the C_human base: PS2 `j DuplicateFrom__7C_human; nop`, PC a 5-byte `jmp 0x0048f880`. C_player vtable slot **21**, 1 dataref |
| 0x004c6cc0 | `player_save_data` | `SaveData__8C_playerFP7C_chunk` | 0x001ab080 | CONFIRMED | Bare forward to `human_save_data` → bool; C_player adds nothing of its own to the chunk stream |
| 0x004c6ce0 | `player_load_data` | `LoadData__8C_playerFP7C_chunk` | 0x001ab0a0 | CONFIRMED | Bare forward to `human_load_data` → bool. Inverse of `player_save_data` |
| 0x004c6d00 | `player_save_game_get_size` | `SaveGameGetSize__8C_playerFv` | 0x001ab0c0 | CONFIRMED | C_human base size + the 0x22 player record |
| 0x004c6d10 | `player_save_game_save` | `SaveGameSave__8C_playerFPPv` | 0x001ab0e0 | CONFIRMED | chains `human_save_game_save`, tag `1` + 11 fields, advance 0x22 |
| 0x004c6da0 | `player_save_game_load` | `SaveGameLoad__8C_playerFPPv` | 0x001ab190 | CONFIRMED | inverse; resets `+0xab8`=0/`+0xabc`=1.0f; active-player gate → HUD re-sync |
| 0x004c6ec0 | `thunk_human_death` | `Death__8C_playerFv` | 0x001ab3a0 | CONFIRMED | Same shape as `DuplicateFrom`: PS2 `j Death__7C_human; nop`, PC `jmp 0x00490a20` (`human_death`). C_player vtable slot **29**, 1 dataref |
| - | - | `FireMagnetism__8C_playerFP8S_vectorP8S_vector` | 0x001ab3b0 | PS2-ONLY | Autoaim (§7). Uses `AngleTo__8S_vector` angle-to-target geometry; called ONLY from `AI_walk`/`AI_drive` |
| - | - | `FireDriveFindTarget__8C_playerFP8S_vector` | 0x001aba00 | PS2-ONLY | Drive-by auto-target acquisition. Called only from `AI_drive` |
| - | - | `FindNearestEnemyToTurn__8C_playerFb` | 0x001abdb0 | PS2-ONLY | Auto-turn-to-nearest-enemy. Called only from `AI_walk` |
| - | - | `ZkurvenaPrebijeciFunkceAdvanced__8C_playerFv` | 0x001ac3c0 | INLINED? | Reload (dev-joke name). Called only from `AI`; on PC `player_ai` calls only the 3 control ticks, so it is inlined into one of them. Not separately located (§8) |
| 0x004c6ed0 | `player_ai_walk` | `AI_walk__8C_playerFUiP17G_ConsoleControls` | 0x001ac740 | CONFIRMED | ON-FOOT control tick. **Dispatch-locked**: `player_ai` selects it under `+0x98`==0/kind≠4 ↔ PS2 `+0x224`/`+0x50`. Body diverges on input reading (§7) |
| 0x004c91f0 | `player_ai_drive` | `AI_drive__8C_playerFUiP17G_ConsoleControls` | 0x001af120 | CONFIRMED | IN-CAR DRIVER tick (seat `+0xac`==0). PS2 `AI_drive` calls `FireMagnetism`/`FireDriveFindTarget`; PC uses manual `human_do_aimed` + `collision_test_line_horizontal` |
| 0x004ca570 | `player_ai_car_fire` | `AI_car_fire__8C_playerFUiP17G_ConsoleControls` | 0x001afb80 | CONFIRMED | IN-CAR PASSENGER-FIRE tick (the `else` of the driver-seat test). Third sub-tick; dispatch-locked |
| 0x004cb1b0 | `player_ai` | `AI__8C_playerFUi` | 0x001b0590 | CONFIRMED | player control tick; chains `human_ai` |
| - | - | `GoToCar_Init__8C_playerFv` | 0x001b1b10 | PS2-ONLY | The gamepad auto-walk-to-car assist (§7). Not in the PC cluster; `player_ai` never calls it |
| - | - | `GoToCar_Cancel__8C_playerFv` | 0x001b3270 | INLINED | 1-line clear `if(+0xc14) +0xc14=0`; inlined into each PC caller |
| - | - | `GoToCar_Process__8C_playerFUiP17G_ConsoleControls` | 0x001b3290 | PS2-ONLY | The per-frame body of the auto-walk-to-car assist (takes `G_ConsoleControls`) |
| 0x004cba50 | `thunk_human_update` | `Update__8C_playerFUi` | 0x001b38e0 | CONFIRMED | A 5-byte `jmp 0x00492ff0` (`human_update`) in C_player vtable slot **14**, 1 dataref; PS2's 44 bytes are a bare forward to `Update__7C_human`. The player's per-frame body is `player_ai` |
| 0x00461180 | `stub_noop_args4_00461180` *(shared)* | `NetDirect__8C_playerFUi` | 0x001b3910 | STRUCTURAL | C_player vtable slot **15** holds the shared `ret 4` no-op that **30 vtables** point at, as `structs/bottle.md` §3a describes for the gutted replication layer. The mapping is a slot role, not a twin: it belongs in this table and in no docblock |
| 0x004cba60 | `player_game_init` | `GameInit__8C_playerFv` | 0x001b3920 | CONFIRMED | (Re)init at game/level start: resets the save-record band (`+0xa9c`=0xffff ↔ PS2 `+0xc20`, the proven record start) + re-caches the "base" model frame |
| 0x004cbb80 | `player_game_done` | `GameDone__8C_playerFv` | 0x001b3a60 | CONFIRMED | Teardown at game/level end: destroys the `+0xaf8` sub-object (↔ PS2 `+0xc78`), chains `human_game_done` |
| 0x004cbbc0 | `player_collision` | `Collision__8C_playerFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001b3ac0 | CONFIRMED | Collision override: if controls locked (`+0xada` ↔ PS2 `+0xc5a`), suppress unless `police_board_is_actor_in_pursuit` (↔ `SemTrefenaLocklaKurva`); else chain `human_collision` |
| 0x004cbc10 | `player_hit` | `Hit__8C_playerF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001b3b70 | CONFIRMED | Hit override: same lock guard as `player_collision`, chains `human_hit`, then refreshes the active player's HUD health (+ cancels sniper mode in world special-mode 5) |
| 0x004cbcc0 | `player_update_controls` | `UpdateControls__8C_playerFv` | 0x001b3d00 | CONFIRMED | Per-frame driver steering feel: when driving (kind==4, seat `+0xac`==0) + a world gate, calls `vehicle_set_steering_linearity`. PC passes tunable `g_steering_linearity`, PS2 the literal `0.5f` |
| 0x004cbd00 | `player_do_action_button_on_foot` | `Do_AB_OwnerNULL__8C_playerFv` | 0x001b3d60 | CONFIRMED | **AB = Action Button**; "OwnerNULL" = vehicle link `+0x98` is null ⇒ on foot. Dispatches the action menu (textdb ids `0x7d1..0x7e1` = strings 2001..2017). Decisive: `player_try_use_car`/`Can_UseCar` is called **×2 from this caller and nowhere else, on both builds**; shared ids `0x7d3`×3 / `0x7d9`×2 |
| 0x004cc950 | `player_do_action_button_in_car` | `Do_AB_OwnerCar__8C_playerFv` | 0x001b4a40 | CONFIRMED | Sibling of the above, for a player whose `+0x98` link IS a kind-4 car. Shared menu ids `0x7d5`×2/`0x7d6`×2; `vehicle_set_engine_on` ↔ `SetEngineOn__9C_Vehicle`; +0xad8↔+0xc58. Size ratio 0.55 = factoring differs (PS2 inlines the menu/vector ops). Calls `Start__8C_pumper`, a live PC callee into `structs/pumper.md`'s class |
| - | - | `NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji__8C_playerFv` | 0x001b5010 | not located | Crosshair-type-by-weapon selector (writes PS2 crosshair global `DAT_004f3224`). Not inlined into `player_indicator_set_aimed` (checked). See §8 |
| 0x004ccc80 | `player_indicator_set_aimed` | `IndicatorSetAimed__8C_playerFv` | 0x001b50a0 | CONFIRMED | aim crosshair — sets/clears HUD bits `0x20`/`0x8000000` |
| 0x004ccd20 | `player_add_shot_accuracy_penalty` | `ShootDeAimed__8C_playerFP15S_GameItemTable` | 0x001b51d0 | CONFIRMED | Statement-for-statement identical: 3000 ms window · weapon `+0x84` · world-mode ==5 ⇒ ×0.1 · 1.0 clamp · `rand*0.2+0.8` reseed · `x*3.0+1.0` to the indicator (PS2 `PlayerSetAccuracy__17G_IndicatorsClass`). Caller sets match exactly on both builds |
| 0x004ccdd0 | `player_set_sniper_mode` | `SetSniperMode__8C_playerFb` | 0x001b52b0 | CONFIRMED | sniper zoom; identical FOV constant `0x3df5c28f` + weapon flag `0x4000`. Also called from C_human `Go_Front` |
| 0x004ccf00 | `player_change_model` | `_intern_ChangeModel__8C_playerFv` | 0x001b5420 | CONFIRMED | The C_player half of a model change, called by `human_change_model` under an **is-the-active-player** guard (`*(g_act_mission+0x24)+0xe4` == this). Re-resolves the `"base"` sub-frame and re-caches it at +0xab4. **Not an override** of the C_human method of the same name — signatures differ (`Fv` vs `FPcb`); it is a downcall, like the `kind==0x1b` C_entity branch beside it. Shared FourCCs `0x4d474e53`/`0x524d4d53` |
| 0x004ccf80 | `player_try_use_car` | `Can_UseCar__8C_playerFP5C_cari` | 0x001b54e0 | CONFIRMED | **Not a predicate despite `Can_`** — returns void, commits side effects: reports the attempt via `police_manager_report_police_car_stealing_sighting` if `car+0x2044`; refuses with a scream (`human_new_scream(rand+0x84)`) if `garage_can_steal_car_model` says no AND world bit 0x20 is clear; else picks the enter anim. Shared `0x84` + bit `0x20`, identical guard order. Distinct from `MuzuSeTamNasomrovat__FP7C_humanP5C_cari`. Names `mission+0x2da0` = the police block (PS2 +0x11e0) |
| 0x004cd5c0 | `player_show_car_damage_bar` | `SetCarDamage__8C_playerFP5C_car` | 0x001b5cb0 | CONFIRMED | Latches the car at +0xae0, arms 3000 ms at +0xadc, raises `g_hud_flags` bit **0x10000** = `cardamagevisible`. Byte-exact both builds. Corollary global twin: `g_hud_flags` ↔ PS2 `DAT_004f3094` |
| 0x004cd5f0 | `player_invalidate_car_damage_bar` | `InvalidateCarDamage__8C_playerFP5C_car` | 0x001b5d00 | CONFIRMED | Drops the bar only if the passed car IS the latched one. Byte-exact; both tested fields are proven pairs (+0x98↔+0x224, kind +0x10↔+0x50 ==4). `game_invalidate_actor` calls it type-4-only on teardown |
| 0x004cd640 | `player_add_alpha_object` | `AddAlphaObject__8C_playerFP9I3D_frame` | 0x001b5d60 | CONFIRMED | Calls `player_alpha_list_add_frame_tree(this->frame, &this->alpha_list)` to collect every type-1 `'OBJ_'` (`0x5f4a424f`) frame of a subtree, then re-applies the fade when `+0xab8 < 1.0` — a tail statement-identical to PS2's. Callers: `human_change_weapon_model` ×2 under its active-player gate, `engine_new_mission` |
| 0x004cd700 | `player_del_alpha_object` | `DelAlphaObject__8C_playerFP9I3D_frame` | 0x001b60f0 | STRUCTURAL | The PS2's 8 bytes are `*(this+0xc48) = 0`, two MIPS instructions that clear the list; the PC removes one frame subtree via `player_alpha_list_remove_frame_tree`. Same role, same slot, different bodies. Sole caller: `human_change_weapon_model` on the outgoing weapon model |
| 0x004cd7a0 | `player_set_alpha` | `SetAlpha__8C_playerFf` | 0x001b6100 | CONFIRMED | Applies transparency to the cached "base" frame (`+0xab4` ↔ PS2 `+0xc38`) + every frame in the alpha vector. Pairs with `player_add_alpha_object` |
| 0x004cd800 | `player_show_weapon_name` | `ShowWeaponName__8C_playerFv` | 0x001b6190 | STRUCTURAL | The PS2 body is `return;`, the PC has 144 bytes of real code: on a change of the equipped-weapon slot (`+0x4a0` vs the cache at `+0xaf0`) it calls `hud_console_print(slot + 0xce4, 0xffffff)`. `0xce4` = 3300, and textdb 3300 = "Empty hands", 3302 = "Knuckle-duster", 3308 = "S&W Model 10 M&P" — the shipped string table names the function. 13 callsites, all in the three control ticks. STRUCTURAL because an empty PS2 body cannot be diffed |
| 0x004cd890 | `player_lock_controls` | `LockControls__8C_playerFb` | 0x001b61a0 | CONFIRMED | take player control away (scripted/cutscene handoff); flag `+0xada`; locking also tears down the in-progress action |
| 0x004cdb40 | `player_fade_reset` | `FadeReset__8C_playerFv` | 0x001b6430 | CONFIRMED | Statement-for-statement: `+0xab8 = 1.0f; +0xabc = 1.0f; player_set_alpha(1.0f)` ↔ `+0xc3c = 1.0f; +0xc40 = 1.0f; SetAlpha(+0xc3c)`, both pairs at delta 0x184. This row is what proves `+0xab8` is the ALPHA and not a model scale. Called twice from `camera_begin_mode` |

### 3a. TU accounting, both directions

The `C_player.cpp` translation unit `[0x004c6b80, 0x004cdd70)` is **closed**: 41/41 PC starts
named, 44/44 PS2 methods accounted. Bounds from the CRT static-init/atexit pairs; all 41 starts
were tested for being an early-exit block misread as a function start, and none is one, so no size
in this TU is distorted.

| PC side (41 starts) | | PS2 side (44 methods) | |
|---|---|---|---|
| C_player methods | 33 | mapped to a PC body | 32 |
| vtable thunks | 3 | realised as a PC thunk | 3 |
| file-local alpha helpers | 2 | shared no-op stub (`NetDirect`) | 1 |
| free function (`player_set_navi_point`) | 1 | PS2-only (autoaim ×3, `GoToCar_Init`/`_Process`) | 5 |
| PC-only cheat helper (`player_set_hand_scale`) | 1 | inlined (`GoToCar_Cancel`) | 1 |
| shared STL helper (`std_vector4_insert_n`, 192 callers) | 1 | inlined-candidate (the reload) | 1 |
| | | not located (`Nastav…`) | 1 |

---

## 4. `player_*` PC functions that are NOT C_player methods

| PC VA | PC name | what it is |
|---|---|---|
| 0x005c8d50 | `player_set_wingman_indicator` | a HUD helper; no matching PS2 method name |
| 0x00425390 | `act_mission_get_game` | a singleton accessor on a different object |

## 5. `C_player.cpp` — free function and file-scope statics

**Nothing in this section is a `C_player` method**; none of it belongs in §3's 44-row table.

| PC VA | PC name | what it is |
|---|---|---|
| 0x004c6b80 | `player_set_navi_point` | A **free function** — mangled `__FP9I3D_frame`, no class tag, no `this`. One statement: `g_player_navi_frame = frame`. It is the FIRST symbol of `C_player.cpp` on both builds. Callsite census 3↔3, roles matching one for one: the SETCOMPASS script case in range (the frame) and out of range (NULL), plus game/mission init (NULL) |
| 0x004cd680 | `player_alpha_list_add_frame_tree` | The file-local recursive collector that fills the alpha list. Its PS2 template argument is what types the list: **`I3D_object*`** |
| 0x004cd720 | `player_alpha_list_remove_frame_tree` | Its mirror; PC-only |
| 0x004cda20 | `player_set_hand_scale` | PC-only. The BOXER cheat's hand-scale setter (`game_cheat_check` case 4; its sole caller passes `[world+0xe4]` = the active player). Writes `(1,1,1)` or `(2,2,2)` to `frame+0x90..+0x98` (local SCALE) on `r_hand`/`l_hand` |
| 0x004cdb60 | `std_vector4_insert_n` | **Not C_player at all** — 192 callsites across ~40 modules, COMDAT-placed in this TU; the mechanical name is correct |

**File-scope data.** The BSS run belonging to this TU is `0x006560d0..0x006560e8`,
bracketed below by the pedestrian TU (`…0x006560cc`) and above by the next TU
(`0x006560ec…`) — established by an absolute-reference census of `.text` attributed to the
containing functions:

| address | read by | state |
|---|---|---|
| `0x006560d0`..`0x006560e0` | **`player_ai_drive` only** — 5 floats, 2 refs each | the secondary-binding latches of §5a: `g_player_drive_steer_right_secondary_latch`, `_steer_left_`, `_clutch_`, `_brake_`, `_accel_secondary_latch` |
| `0x006560e4` | `game_init`, `human_intern_use_car`, `human_put_in_vehicle_at_seat`, `human_update`, `player_ai`, `player_ai_drive`, + its setter `hud_set_car_damage_bar_enabled` | `g_hud_car_damage_bar_enabled`, an `extern` defined in this run: set by script `cardamagevisible`, cleared by `game_init` |
| `0x006560e8` | **nothing** | `g_player_navi_frame` (PS2 `navi_frame`). Write-only on both builds |

### 5a. The driving-input latches

Every driving axis has a PRIMARY and a SECONDARY key binding (action slots `0x21`..`0x28` and
`0x37`/`0x38`, `structs/input.md` §2). `player_ai_drive` arbitrates between them per axis:

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
`C_car+0x594..+0x597` (`structs/vehicle.md` §1c), which switch the vehicle setters between a
ramped digital input and a direct analog one. `player_game_init` writes `0xffff`, so at a level
start every secondary axis is ignored until it moves.

## 6. Related

- `structs/human.md` (the C_human base) · `structs/actor.md` (the C_actor root) ·
  `structs/entity.md` (the sibling C_entity).
- `structs/hud.md` — `g_hud_flags` and the accuracy/health/console sinks.
- `Game-ghidra.exe.functions.md` — the function registry.

---

## 7. PC vs PS2

| # | difference | evidence |
|---|---|---|
| 1 | **PS2 has AUTOAIM; the PC does not.** `FireMagnetism` / `FireDriveFindTarget` / `FindNearestEnemyToTurn` are PS2-only. | PC's `player_ai_drive` does drive-by aim with manual `human_do_aimed` + `collision_test_line_horizontal` and calls no magnetism. ✅ Confirmed by the user (Tier-1) |
| 2 | **No auto-goto-car on the PC.** The `GoToCar_*` family is the PS2 gamepad auto-walk-to-car assist; `GoToCar_Init`/`Process` have no PC twin and `GoToCar_Cancel` (a 1-line clear) is inlined. | ✅ Confirmed by the user (Tier-1). The third PS2 `Can_UseCar` caller has no PC counterpart because the feature is absent |
| 3 | **Both alpha methods ignore their frame parameter on PC.** `player_add_alpha_object` and `player_del_alpha_object` accept an `I3D_frame*` (`ret 4`) and never read it; they walk `this->+0x68`, the actor's own scene frame (`structs/actor.md`), so the whole model is rescanned. PS2 walks the frame it is given. | Disassembly of both: `mov ecx,[esi+0x68]` / `lea edx,[esi+0xac0]` and no access to `[esp+8]`. The divergence is **symmetric across both methods**, which makes it a build difference rather than a decompiler artifact |
| 4 | **`DelAlphaObject` clears the whole list on PS2 and removes one subtree on PC.** | PS2's 8-byte body is `*(this+0xc48) = 0`; the PC recurses a frame tree doing swap-with-last removals |
| 5 | **`ShowWeaponName` exists only on PC.** The PS2 body is empty; the PC prints the weapon's localized name (textdb 3300 + slot) to the HUD console on every weapon change. | Here the PS2 method count is a **lower** bound |
| 6 | **The C_player band's 4-byte packing shift is the alpha vector.** PC `std::vector` 16 B vs PS2 `__vector_pod` 12 B. | §1a |
| 7 | **Control input.** The three control ticks take a `G_ConsoleControls*` on PS2 (gamepad); the PC reads input directly. | `AI_walk` / `AI_drive` / `AI_car_fire` signatures, §3 |
| 8 | **Steering linearity.** `player_update_controls` passes the tunable `g_steering_linearity`; PS2 passes the literal `0.5f`. | §3 |
| 9 | **Accuracy mirror.** PC's `player_add_shot_accuracy_penalty` mirrors the accumulated spread to world `+0xd8`; PS2's `ShootDeAimed` has no such statement. | §8 |

A PS2 method count of 44 was never the PC target: several of those methods are console-input
features the PC never had. Both feature differences follow from the controls differing between
builds (gamepad vs mouse/keyboard).

## 8. Open items

| item | state |
|---|---|
| `NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji` (140 B) | the crosshair-type-by-weapon selector. The PC gap between `player_do_action_button_in_car` and `player_indicator_set_aimed` is **empty**, so it is recorded *not located*, never "inlined"; it is also not inlined into `player_indicator_set_aimed` (checked). Lever: `player_indicator_set_aimed` (flag bit 0x20) + `hud_set_player_accuracy` (+0x9234) are the crosshair path — search their neighbourhood |
| `ZkurvenaPrebijeciFunkceAdvanced` (reload, 888 B) | CANDIDATE-inlined into one of the three control ticks. Confirm which by scanning `player_ai_walk` / `_car_fire` for the reload state machine |
| what reads `g_player_navi_frame` | nothing does, on either build (§5). The value is published and consumed by nobody |
| `+0xabc` (`alpha_partner`) | set to 1.0f beside the alpha by two writers, read by nobody located |
| the `+0xd8` world mirror | PC's `player_add_shot_accuracy_penalty` writes it, PS2's `ShootDeAimed` does not. Whether `+0xd8` drives a separate screen/camera effect on the PC, or is read back by the same crosshair path, is undecided |
| `car+0x2044` / `car+0x2031` | the gates on `player_try_use_car`'s police-report branch and on `player_show_car_damage_bar`; both unidentified |
| `player_set_wingman_indicator` and `act_mission_get_game` (§4) | not body-diffed against any PS2 method |
