# `game_menus` — the in-game and front-end SCREEN translation unit

**Created** 2026-08-25 17:18 +0200 · **Modified** 2026-10-04 16:37 +0200

PC range **`[0x00575c50, 0x0057d6e0)`**, closed at both ends by a CRT static-init pair
(`crt_static_init_guard_00575c20` / `_0057d6e0`). It is the translation unit that immediately
FOLLOWS `g_menu.cpp`, and it holds the **screens** the GUI framework exists to run: the main menu,
the in-game menu and its racing variant, options and controls, the three player-profile screens,
load game, free ride, the three modal boxes, game over, and the loading-progress screen.

Sister documents: `structs/gui_screen.md` (the `GM_Menu` base class, its 6-slot vtable and the
framework API), `structs/gui_widget.md` (the `GM_Component` widget base and all 53 widget classes),
`structs/racing_menus.md` and `structs/multiplayer.md` (the two other screen TUs).
Prefix: one per screen class — `gui_main_menu_`, `gui_options_controls_`, `gui_profile_select_`
… — matching the `gui_race_` / `gui_mp_` practice rather than folding everything into
`gui_menu_`, which stays reserved for the framework.

> Docblocks in the decomp cite sections of this file as `SS<n>`; read that as `§<n>`.

---

## 1. What it is, and the lever that opened it

**88 real function starts, all 88 named — the TU is CLOSED.** The raw start count is 98; ten of
those are decompiler artifacts rather than functions (§9).

**Every screen constructor in the GUI is identifiable without reading its body.** Each one pushes
exactly one literal into

```c
gui_menu_build_components_from_design(&g_gui_screen_design_table, <design_id>, <gate>);
```

and that literal is the **group index in the shipped `tables\menu.def`** — byte-verified, the
group walker compares each record's tag against `'stop'` (`cmp ecx, 0x73746f70`). Decoding the
file therefore hands you a screen's component ids, its widget classes, its geometry and — via
`textdb.json` — its on-screen labels, before a single instruction is read.

Decoding `menu.def` by walking those group records yields **72 groups**, ids `0x00`..`0x47` (the
last 16 empty), **809 records of `0x24` bytes**:
`{u32 id, FourCC tag, f32 x, y, w, h, u32 text_id, u32 flags, u32 colour}`.

**Seven non-empty shipped designs are never built.** The builder has 58 `E8` callers in the whole
binary, no `E9` referrer, and every one of the 58 pushes a literal design id; the design table's
only other reference is its loader. None of the 58 pushes `0x11` (*Your inventory is full*),
`0x12` (a *Pickup/Use* list), `0x15` (three unlabelled items), `0x24` (*You have won a bonus
car*), `0x29` (a *Multiplayer:* name field), `0x2d` (a name field, a combo box and two buttons)
or `0x31` (*Exit to Main Menu* / *Retry*: the game-over design `0x30` with the two rows in the
other order).

### 1a. The design map of this TU

Thirty-five of the 88 starts push a design literal. `gate` selects a variant group within the
same design id.

| PC constructor | VA | design | gate | shipped caption |
|---|---|---|---|---|
| `gui_combo_dropdown_screen_ctor` | `0x00575c50` | `0x13` | 0 | *(the combo drop-down list)* |
| `gui_notice_panel_ctor` | `0x00575f50` | `0` | 0 | *(two message rows, no buttons)* |
| `gui_notice_panel_ctor_from_strings` | `0x005761d0` | `0` | 0 | *(same, `const char *` form)* |
| `gui_question_box_ctor` | `0x005764c0` | `1` | 0 | Yes / No |
| `gui_message_box_ctor` | `0x00576620` | `2` | 0 | OK |
| `gui_main_menu_exit_box_ctor` | `0x00576810` | `0x36` | 0 | "Quit Game?" (textdb 119) |
| `gui_main_menu_ctor` | `0x00576920` | `6` | **2** | Main Menu: |
| `gui_game_menu_exit_box_ctor` | `0x00576cc0` | `0x36` | 0 | "Exit Game?" (textdb 149) |
| `gui_game_menu_ctor` | `0x00576dd0` | `7` | 0 | Game menu: |
| `gui_game_menu_on_exchange` | `0x00576e30` | `7` | 0 | *(rebuild)* |
| `gui_options_audio_video_ctor` | `0x00576fe0` | `8` | `g_net_session != 0` | Audio & Video |
| `gui_options_ctor` | `0x00577990` | `9` | 0 | Options: |
| `gui_options_controls_ctor` | `0x00577b10` | `10` | 1 | Player Controls: |
| `gui_options_controls_on_exchange` | `0x00577b90` | `10` | 1/2/4 | *(per input device)* |
| `gui_key_capture_ctor` | `0x005782e0` | `0xb` | 0 | Setup Controls: |
| `gui_key_conflict_ctor` | `0x005786e0` | `0x35` | 0 | Control already used |
| `gui_inventory_ctor` | `0x00578a30` | `0xf` | var | Inventory: |
| `gui_action_menu_ctor` | `0x00579020` | `0x10` | 0 | Pickup/Use |
| `gui_profile_select_ctor` | `0x00579900` | `4` | 0 | Player Profile: |
| `gui_profile_select_on_exchange` | `0x005799d0` | `4` | 0 | *(rebuild)* |
| `gui_profile_create_ctor` | `0x00579fb0` | `3` | 0 | Enter Profile Name: |
| `gui_profile_delete_ctor` | `0x0057a1f0` | `5` | 0 | Delete Profile: |
| `gui_load_game_ctor` | `0x0057a4a0` | `0xc` | 0 | Load Game: |
| `gui_free_ride_ctor` | `0x0057aaf0` | `0xe` | 0 | Free Ride: |
| `gui_free_ride_location_ctor` | `0x0057aff0` | `0x34` | 0 | Location: *(start district)* |
| `gui_session_list_ctor` | `0x0057b150` | `0x2f` | 0 | Select session: |
| `gui_gun_shop_ctor` | `0x0057b3a0` | `0x14` | 0 | Inventory (Drop) *(the weapon shop)* |
| `gui_garage_select_ctor` | `0x0057b930` | `0x25` | 0 | Garage |
| `gui_garage_full_ctor` | `0x0057bfa0` | `0x26` | 0 | Garage |
| `gui_garage_summary_ctor` | `0x0057c540` | `0x27` | var | Close |
| `gui_game_over_ctor` | `0x0057c8e0` | `0x30` | 0 | Game Over |
| `gui_racing_game_menu_ctor` | `0x0057cab0` | `0x2e` | 0 | Game menu: *(with Skip Race)* |
| `gui_load_progress_ctor` | `0x0057cc90` | `0x32` | 0 | Loading progress |
| `gui_load_progress_on_exchange` | `0x0057cf80` | `0x32` | 0 | *(rebuild)* |
| `gui_free_ride_mode_ctor` | `0x0057d4b0` | `0x33` | 0 | Location: *(free-ride scenario)* |

### 1b. Slot 5 is the TICK, and it folds several PS2 virtuals

The PC `GM_Menu` vtable has **one** input slot. The PS2 base has **six** separate virtuals —
`Tick`, `OnTick`, `OnClick`, `OnKey`, `OnMouse`, `OnChange`. Every PC override therefore has the
same shape:

```c
id = gui_menu_tick(...);           // the per-frame focus/mouse/key walk, RETURNS the picked id
if (id == 0 && (key & 0xff) == 1)  // ESC                        == PS2 OnKey
switch (id) { ... }                // the dispatch                == PS2 OnClick
```

So a slot-5 function is named `_tick` after the **PC slot**, and its `PS2 twin:` line names
`OnClick__…` because that is the largest thing it absorbs. Each registry row records the fold
count. The extremes: `gui_main_menu_tick` is 1 → 1; `gui_load_game_tick` is **5 → 1**.

---

## 2. The main menu and the in-game menu

Both are plain item lists whose rows come straight from `menu.def`, and both delegate the
"really quit?" question to a tiny design-`0x36` box.

| function | VA | role |
|---|---|---|
| `gui_main_menu_ctor` | `0x00576920` | design 6 gate 2; camera target `0xf1b31`; deletes the two unshipped rows |
| `gui_main_menu_tick` | `0x00576a20` | 12 arms over ids `0xf1b95`..`0xf1ba1` |
| `gui_main_menu_exit_box_ctor` / `_tick` | `0x00576810` / `0x00576890` | "Quit Game?" → result `0x18` |
| `gui_game_menu_ctor` | `0x00576dd0` | design 7 |
| `gui_game_menu_on_exchange` | `0x00576e30` | slot 1 rebuild |
| `gui_game_menu_tick` | `0x00576e60` | ids `0x66`/`0x67`/`0x6c`/`0x6f` |
| `gui_game_menu_exit_box_ctor` / `_tick` | `0x00576cc0` / `0x00576d40` | "Exit Game?" → result `0x21` |

**The main-menu row ids are the cut-content evidence.** Shipped design 6 gate 2 lists
990101..990106 and 990109..990113. The constructor disables `0xf1b9a` (990106, *Free Ride
Extreme*) below a licence level and **deletes `0xf1b9b` and `0xf1b9c`** (990107, 990108) —
neither of which the retail `menu.def` contains. `0xf1b9c` is the Multiplayer entry
(`structs/multiplayer.md` §1); the tick still has live arms for both.

---

## 3. The three modal boxes

The PC splits its modal vocabulary three ways, one shipped design each. The discriminator is the
**component id set**, not the auto-fit layout pass — all three run that pass, so it separates
nothing.

| class | design | components | dismissal | PC functions |
|---|---|---|---|---|
| **notice panel** | `0` | `vers`, frame `1`, `ctxt` `0x82`, `ctxt` `0x83` — **no buttons** | any key or click | `gui_notice_panel_ctor` `0x00575f50`, `gui_notice_panel_ctor_from_strings` `0x005761d0`, `gui_notice_panel_tick` `0x00576450` |
| **question box** | `1` | frame `0x82`, message `0x83`, **item `0x8a` "Yes"**, **item `0x8b` "No"** | Yes → `0xb`, No/ESC → `0xc` | `gui_question_box_ctor` `0x005764c0`, `gui_question_box_tick` `0x00576560` |
| **message box** | `2` | frame `1`, message `0x82`, `butt` `0x83` "OK" | any click, ESC or ENTER | `gui_message_box_ctor` `0x00576620`, `gui_message_box_tick` `0x005767a0` |

The notice panel is used for plain game messages: script-dispatcher cases `0x100f`/`0x1010`,
and the inventory pair *"You do not have enough room."* / *"You must drop an item."*
`gui_notice_panel_present_once` (`0x0056e3f0`, in the `g_menu.cpp` TU) drives one for exactly
one frame — a splash, not an interactive modal. The notice panel is PC-only (§10c).

### 3a. The combo drop-down

`gui_combo_dropdown_screen_ctor` (`0x00575c50`, design `0x13`) is the pop-up list a combo-box
widget opens, and `gui_combo_dropdown_tick` (`0x00575ea0`) is its slot 5. Both guard on the
option vector at `+0x4c`; the picked row id is written through the out-pointer at `+0x48`.

**Component id 2 is the dismiss target.** Design `0x13` ships exactly three records — a `wind`
frame (1), a `rect` background (2) and an `orez` clipper (3) — and the tick hit-tests component
2 on every mouse button, closing with `-1` when the click landed outside it. That is the PC's
click-away dismissal (§10c).

---

## 4. Options, controls, items and pickers

| function | VA | role |
|---|---|---|
| `gui_options_ctor` / `_tick` | `0x00577990` / `0x00577a20` | design 9; rows 100 Controls, 101 Audio & Video, 400 Back |
| `gui_options_controls_ctor` | `0x00577b10` | design 10 gate 1 |
| `gui_options_controls_on_exchange` | `0x00577b90` | slot 1; gate 1/2/4 = keyboard / mouse / joystick, then re-seed every value |
| `gui_options_controls_close` | `0x00577d20` | slot 4; apply bindings + reload the speedo texture |
| `gui_options_controls_tick` | `0x00577d50` | every slider and toggle; `0x35` restores defaults |
| `gui_free_ride_ctor` | `0x0057aaf0` | design `0xe` |
| `gui_free_ride_on_close_child` | `0x0057ac40` | slot 3; car name, mode label and `freerideN.bmp` |
| `gui_free_ride_tick` | `0x0057ad80` | ids 100 / `0x65` / 200 / `0xc9` |

**Design 10 is the biggest group in the file — 149 records across three gates**, one per input
device. The controls page persists straight into the settings globals and calls
`loadsave_apply_settings` with a per-block dirty mask (8 = the joystick axis block, `0x10` =
force feedback). Toggle `0x4b` is the **mph/kph** switch, which is why closing the page reloads
the speedometer texture.

Free ride stores the chosen car at `+0x4c` and the mode at `+0x48`; modes `0x1c`..`0x20` each
have their own preview bitmap and their own label (textdb `0x41a`..`0x41e`). Mode `0x1c` starts
immediately; the others open a location picker first (§4f).

### 4a. Audio & Video — design 8 *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_options_audio_video_ctor` | `0x00576fe0` | design 8; seeds every control from the settings block |
| `gui_options_audio_video_tick` | `0x005771b0` | slot 5; applies whichever control moved |

**This screen is what decodes the display settings block.** Every control writes a field of
`g_act_mission+0x90`, and joining the component ids to their shipped labels by matching Y
coordinate names the five setters:

| component | shipped id | label (textdb) | setter |
|---|---|---|---|
| `0x6e` | 110 | 231 *Level of Details* | `mission_settings_set_details_scale`, block `+0x00`, clamp [0.4, 2.0] |
| `0x6f` | 111 | 232 *Draw Distance* | `mission_settings_set_draw_distance_scale`, block `+0x04`, clamp [0.3, 1.0] |
| `0x70` | 112 | 238 *Level of Blood* | `g_blood_level` |
| `0x71` | 113 | 260 *In Game Effects:* | `mission_settings_set_effects_quality`, block `+0x0c`, 0..2 |
| `0x72` | 114 | 270 *In Game Sounds:* | `mission_settings_set_sounds_quality`, block `+0x10`, 0..2 |
| `0x73` | 115 | 280 *Shadows:* | `mission_settings_set_shadow_quality`, block `+0x14`, 0..5 → `{0,1,3,6,9,15}` |
| `0x74` | 116 | 290 *Particles:* | `mission_settings_set_particles_quality`, block `+0x18`, 0..2 |
| `0x50` | 80 | 239 *Gamma* | `g_gamma`, settings block `4` |
| `0x51`..`0x54` | 81..84 | 251..254 *Sounds / Music / Cars / Speech* | `loadsave_apply_settings` masks 1 / 2 / `0x20` / `0x40` |

Three details pin that join:

* **Gamma is component `0x50`**, not one of the `0x6e`/`0x6f` sliders — it writes a different
  global entirely.
* **Every global the screen writes has a consumer outside it.** `g_gamma` is settings block
  `+0x4fc` of `g_loadsave_singleton`, the input of the 256-entry gamma ramp that
  `loadsave_apply_settings` builds for mask 4 and hands to engine slot `+0x110`; `g_cars_volume`
  is `+0x4ec`, divided by the sounds volume and passed to `vehicle_set_global_sound_volume` for
  mask `0x21`; `g_speech_volume` scales `speech_channel_speak_line` and the scream volumes of
  `human_new_scream`, `human_game_init`, `human_do_throw_occupant_from_car` and
  `pedestrian_manager_play_pedestrian_scream`; `g_blood_level` gates `hud_push_blood_splat` at
  0.9; `g_freeride_unlock_level` is read by `gui_main_menu_ctor`, `gui_free_ride_ctor` and
  `gui_free_ride_mode_ctor`.
* **The Shadows tier has six labels**, textdb 281..286 *Very Low … Highest*, matching its
  six-entry map exactly.
* `mission_update_settings` (`0x00604f70`) corroborates the first two rows independently: it
  pushes block `+0x00` to the I3D driver as render parameter `0x14`, and multiplies block `+0x04`
  by the base view range at `+0x98`.

The presets `0x65`/`0x66`/`0x67` (Low/Medium/High) set all four quality tiers at once; `0x5a`
restores defaults through `loadsave_default_settings(8)`. The tick returns close code `0x22`
only when the screen has no parent, i.e. when it was opened from the main menu.

### 4b. Key binding — designs `0xb` and `0x35` *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_key_capture_ctor` | `0x005782e0` | design `0xb` "Setup Controls:"; slot at `+0x48`, device flag at `+0x4c` |
| `gui_key_capture_on_close_child` | `0x00578370` | slot 3; closes itself once the conflict child returns |
| `gui_key_capture_tick` | `0x005783a0` | slot 5; captures the event, commits or escalates |
| `gui_key_conflict_ctor` | `0x005786e0` | design `0x35` "Control already used"; carries the captured triple |
| `gui_key_conflict_tick` | `0x005787c0` | slot 5; Yes commits, No/ESC abandons |

A committed binding is three dwords at **`g_key_binding_table + slot*0xc`**; the input side
reads it through `input_set_binding`, `input_update` and `input_binding_get_label`
(`structs/loadsave.md` §7). Both screens normalise the
paired axis codes the same way (5/7, `0x26`/`0x28` swap; `0x12`/`0x13` clear), which is what
pairs them. Both are PC-only (§10c).

### 4c. The inventory screen — design `0xf` *(PC-only as a screen)*

| function | VA | role |
|---|---|---|
| `gui_inventory_ctor` | `0x00578a30` | design `0xf`, gate 1/2 = on foot / in car |
| `gui_inventory_format_slot_row` | `0x00578940` | free function; one row's label |
| `gui_inventory_on_close_child` | `0x00578d10` | slot 3; takes an item a child handed back |
| `gui_inventory_tick` | `0x00578d90` | slot 5; select, drop, or leave |

Fixed rows: `0x100` Hands, `0x102`..`0x106` the five weapon slots, `0x107` Coat, `0x108` Car,
then one cloned `'seli'` row per carried item in the `0x200+` band. The `G_Inventory` is at
`+0x848` and the owning actor at `+0x84c`. Drop is key `0xe`/`0x2d`; selecting a row calls
`inventory_select_slot(index, kind, actor)` with kind 0 Hands / 1 weapon / 2 Coat / 3 item /
4 Car. The console counterpart is not a screen (§10c).

### 4d. The action menu — design `0x10`

| function | VA | role |
|---|---|---|
| `gui_action_menu_ctor` | `0x00579020` | design `0x10`; gathers world items + NPC inventories |
| `gui_action_menu_scalar_deleting_dtor` | `0x00579480` | slot 0 — the only class-specific one in the TU |
| `gui_action_menu_dtor` | `0x005794a0` | frees the option vector at `+0x1060` |
| `gui_action_menu_pick_up` | `0x005794d0` | records a take, or strips the item from every NPC |
| `gui_action_menu_on_close_child` | `0x00579590` | slot 3; takes the item a child confirmed |
| `gui_action_menu_tick` | `0x00579610` | slot 5; number keys 2..10 → rows `0x100`..`0x108` |

This is the interact-button menu, and it is **not only a pickup menu** — it also carries
open-door, enter-car and the other context actions, which is why the prefix is
`gui_action_menu_` rather than the PS2 class's `GM_ItemPickUp`.
`basic/artifacts/player-interactions.md` §4 is the behavioural write-up.

**The field map:** `+0x1048` the `G_Inventory`, `+0x104c` the NPC vector, `+0x1050` the
world-item vector, `+0x1054`/`+0x1058` the two taken-lists, `+0x1060` the gathered options.

**The tick auto-takes a lone option**, gated by an `& 2` flag on a mission field (§10a).

### 4e. The gun shop — design `0x14`

| function | VA | role |
|---|---|---|
| `gui_gun_shop_ctor` | `0x0057b3a0` | design `0x14`; builds the catalogue at `+0x850`, count `+0xc50` |
| `gui_gun_shop_format_slot_row` | `0x0057b530` | free function; one carried-slot label |
| `gui_gun_shop_refresh_slots` | `0x0057b600` | re-render the player's slots after every trade |
| `gui_gun_shop_tick` | `0x0057b700` | slot 5; drop a slot, or take from the catalogue |

**This is the weapon shop.** Two independent artifacts say so here, and the twin class name is a
third (§10a).

1. The shipped scripts call it: `gunshop_menu` appears twice in mission `MISE19-MESTO`, script
   `1ProdZ`, and once in mission `MISE20-MESTO`, script `1ProdZ` — `1ProdZ` = *prodavač zbraní*,
   the weapons dealer — each time directly after `dialog_end` with him and directly before
   `inventory_push` / `inventory_clear`.
2. The opcode: `GUNSHOP_MENU` is parser raw `0x060`, so dispatcher **case `0x61`**, whose body
   opens this design as a modal and then calls `human_change_weapon_model`. Documented in
   `script-mapping/artifacts/opcodes-verified.md`.

It is a weapon **PICKUP**, not a drop, despite the shipped window caption *"Inventory (Drop)"*
(textdb 1300): the right band lists every `g_game_item_table` entry whose flags lack `0x800000`
and inserts the pick, while the left band only frees capacity. Textdb 1310 labels the right band
*"Select (Pick Up)"*. There is no money or score — the main story has neither.

### 4f. The free-ride pickers — designs `0x33` and `0x34` *(PC-only)*

| function | VA | role |
|---|---|---|
| `gui_free_ride_mode_ctor` | `0x0057d4b0` | design `0x33`; the SCENARIO picker with previews |
| `gui_free_ride_mode_tick` | `0x0057d580` | slot 5; rows 401..405 → modes `0x1c`/`0x1e`/`0x20`/`0x1d`/`0x1f` |
| `gui_free_ride_location_ctor` | `0x0057aff0` | design `0x34`; the START-DISTRICT picker |
| `gui_free_ride_location_tick` | `0x0057b0b0` | slot 5; ids 101..107 → `loadsave_freeride_store_state(id-100)` |

Both ship the caption *"Location:"* (textdb 199), which is why they are easy to confuse. They do
different jobs and sit at different points in the flow:

`gui_free_ride_tick` row 100 opens **design `0x33`**, the five scenarios (textdb 1050..1054:
*City - Small*, *City - Daytime*, *City - Night*, *Countryside - Daytime*, *Countryside -
Night*), each with its own `freerideN.bmp` preview swapped on focus change. Picking one closes
with the free-ride mode. Then `gui_free_ride_tick`'s Start row (`0xc9`) starts immediately for
mode `0x1c`, and for anything else opens **design `0x34`**, the four districts (*Little Italy*,
*Central Island*, *Down Town*, *Hoboken*).

**Design `0x34`'s four rows serve both maps.** When the mode is `0x1f` or `0x20` — the two
COUNTRYSIDE scenarios — the constructor relabels rows 101..104 to textdb 4010..4013: *China Town
Exit*, *Dam*, *Lay-by*, *Motel*.

`g_freeride_unlock_level` gates how many scenarios design `0x33` enables: 0 leaves only *City - Small*, 1
adds the two City rows, 2 and above enables all five.

---

## 5. Game over and the racing in-game menu

| function | VA | role |
|---|---|---|
| `gui_game_over_ctor` | `0x0057c8e0` | design `0x30`; pushes the failure text into the `'list'` block, then re-lays-out |
| `gui_game_over_tick` | `0x0057ca40` | 1000 Retry → `0x4fd`, 1002 Exit → `0x4fb` |
| `gui_racing_game_menu_ctor` | `0x0057cab0` | design `0x2e`; relabels row `0x3e9` in a network session |
| `gui_racing_game_menu_tick` | `0x0057cb40` | ids `0x3e9`..`0x3ec` |

In **free ride** the game-over screen disables *Retry* and focuses *Exit to Main Menu* — there
is no mission to retry (§10a).

---

## 6. Load game and loading progress

| function | VA | role |
|---|---|---|
| `gui_load_game_ctor` | `0x0057a4a0` | design `0xc`; one `'seli'` row per save, newest first |
| `gui_load_game_tick` | `0x0057a670` | scrolling, the thumbnail preview, and the load |
| `gui_load_progress_ctor` | `0x0057cc90` | design `0x32`; `tables\load.def`; the MP roster |
| `gui_load_progress_on_exchange` | `0x0057cf80` | slot 1 rebuild + roster refresh |
| `gui_load_progress_tick` | `0x0057d0c0` | roster refresh + session-readiness codes `0x39`/`0x3a` |
| `gui_load_progress_set_progress_game_init` | `0x0057d240` | `base + (1 - base) * t` → component 300 |
| `gui_load_progress_set_progress_mission_open` | `0x0057d320` | `t * base` → component 300 |
| `gui_load_progress_set_caption` | `0x0057d400` | component 400 |
| `gui_load_progress_present` | `0x0057d420` | a hand-rolled frame, drawn while the loader blocks |

**The save list is reversed.** Rows are cloned walking the file vector backwards from `end`, so
row 100 is the newest save, and the click handler undoes it with `(count - id) + 99`. The
save-file record is **`0x18` bytes** on the PC.

**The scroll range is `'swnd'` + `0x28`.** `gui_load_game_ctor` computes it from the last row's
geometry and writes it directly (§10a).
`structs/gui_widget.md` §6f documents the scroll window this range feeds.

**The progress bar has two phases split at `+0x48`**: mission-open fills `[0, base]` and
game-init fills `[base, 1]`. In a network session each update is also broadcast as a **type
`0x36`** event message so the other players' bars advance; that is why those two functions touch
the message vtable `0x0063e0bc` — the vtable belongs to the message they construct, not to them.

**The save-slot widget's slot 8, `gui_widget_save_slot_set_text`, discards three lookups.** It
asks the text database for textdb `0xae`, `0xaf` and `0xb0` and drops all three results before
forwarding to its child at `+0x170`. The lookup is a pure binary search with no side effect;
`0xaf` is formatted by `gui_load_game_tick`, and `0xae` and `0xb0` are referenced nowhere else in
the binary.

**`tables\load.def`** is read into an 8000-byte buffer and walked as **`0x44`-byte** records:
the mission name at `+0x00`, the loading caption at `+0x20`, and an optional textdb id at
`+0x40`.

---

## 7. The multiplayer session browser

| function | VA | role |
|---|---|---|
| `gui_session_list_ctor` | `0x0057b150` | design `0x2f`; clones one `'seli'` row per session name |
| `gui_session_list_tick` | `0x0057b2e0` | pick / cancel / time out and refresh |

Design `0x2f` ships exactly **two** records — the frame (textdb 610 *"Select session:"*) and one
row template with id 1000 (textdb 611 *"<none found>"*). The constructor takes the caption's
textdb id, a `vector<char*>` of session names and a **refresh timeout in milliseconds**; both
callers in `net_multiplayer_run` pass **2000**. The tick counts that down and closes with `0x41`,
which is the case that re-opens the browser — a 2-second auto-refresh. The focused row is
remembered in `g_gui_session_list_focused_row_id` so the refresh does not lose the selection.

---

## 8. The three player-profile screens

| function | VA | role |
|---|---|---|
| `gui_profile_select_ctor` | `0x00579900` | design 4 |
| `gui_profile_select_on_exchange` | `0x005799d0` | slot 1 rebuild |
| `gui_profile_select_build_rows` | `0x00579a00` | one `'item'` row per profile + re-layout |
| `gui_profile_select_on_close_child` | `0x00579bf0` | slot 3; creates the profile |
| `gui_profile_select_tick` | `0x00579cc0` | pending action, then the picked row |
| `gui_profile_create_ctor` | `0x00579fb0` | design 3 |
| `gui_profile_create_scalar_deleting_dtor` | `0x0057a070` | slot 0 |
| `gui_profile_create_dtor` | `0x0057a090` | leaves text-input mode |
| `gui_profile_create_tick` | `0x0057a0f0` | OK/ENTER → `0xd`, Cancel/ESC → `0xe` |
| `gui_profile_delete_ctor` | `0x0057a1f0` | design 5 |
| `gui_profile_delete_on_close_child` | `0x0057a370` | slot 3; performs the delete |
| `gui_profile_delete_tick` | `0x0057a3c0` | opens the confirm box per row |

**The PC profile record is `0x54` bytes with the name at `+0x10`.** At most **12** profiles exist:
`gui_profile_select_build_rows` disables *Create New Profile* once the count exceeds 11, and
disables *Delete Profile* below two.

**`g_engine` vtable `+0x88` is the keyboard text-input mode selector.** The name-entry
constructor sets it to **2** and its destructor restores **1**; that 2 → 1 pairing is the only
thing in the binary that pins the field's meaning, and it is why this class is one of the few
screens that does not use the shared slot-0 destructor.

The select screen drives itself through a **pending-action code at `+0x48`**: 1 open the name
box, 2 open the delete screen, 3 raise *"This Profile Already Exists"*. The constructor seeds it
to 1 when no profile exists, so a fresh install lands straight in the name box.

---

## 9. The ten addresses in this range that are not function starts

Ten defs inside `[0x00575c50, 0x0057d6e0)` are Aggressive-Instruction-Finder artifacts, in two
distinct shapes. They are what separates the raw start count of 98 from the 88 real starts.

**Nine are EPILOGUE arms:** `0x00576612` · `0x00576ed5` · `0x00577dd2` · `0x00578e65` ·
`0x0057a866` · `0x0057a887` · `0x0057cbb3` · `0x0057d1e1` · `0x0057d206`. For each one, a byte
scan of `.text` finds no `E8 rel32` reaching it, a scan of the data sections finds it as no dword,
it is the target of an earlier `jcc` inside its host, and the eight bytes before it are a
`ret imm16` teardown (`c2 1c 00` — a `__thiscall` screen tick). Seven of the nine sit inside a
`*_tick`, whose bodies are one long switch full of early-outs.

**One is a MID-BODY SPLIT:** `0x0057bd70`, inside `gui_garage_select_tick`. Ghidra decompiles it
as a verbatim copy of its host's tail, reading `unaff_EBX` / `unaff_EDI` / `in_stack_*` — values
live on entry, which a real function start cannot have. The discriminator here is the **vtable
order in `.rdata`**: it appears in none of the eleven screen vtables of this TU while every one
of its neighbours does, and a screen-TU function in no vtable, with no `E8` caller and no
dataref, cannot be reached at all. The split sits 96 bytes into its host, a 608-byte function
(§10a).

---

## 10. PC vs PS2

The corresponding PS2 range is **`[0x002cf2f0, 0x002d41e0)`**, an 18-class TU:
`GM_MainMenu(Exit)`, `GM_GameMenu(Exit)`, `GM_Options`, `GM_OptionsControls`,
`GM_OptionsScreenSlide`, `GM_FreeRide`, `GM_QuestionBox`, `GM_LoadGame`, `GM_GameFailed`,
`GM_ItemPickUp`, `GM_GunShop`, `GM_LoadProgress`, `GM_PressStart`, `GM_GameMap`,
`GM_TutorialMessage`, `GM_HotRodMessage`. The PC TU additionally absorbs the three
`GM_Profile*` screens (a separate TU on the PS2, at `0x002df4c0`) and `GM_GameMenuRacing` (which
sits with the racing menus there), so the PC file is a wider cut of the same material.

### 10a. The strongest shared constants

These are what make the twins measurements rather than positional guesses.

| PC function | PS2 method | the shared fact |
|---|---|---|
| `gui_profile_delete_tick` | `OnClick__16GM_ProfileDeleteFUi` | both pass textdb **`0x82`** ("Delete") to the confirm box at the same site |
| `gui_profile_create_tick` | `OnClick__16GM_ProfileCreateFUi` | close arguments **1**/**0** and results **`0xd`**/**`0xe`** |
| `gui_profile_select_tick` | `OnClick__16GM_ProfileSelectFUi` | results **`0xf`**/**`0x10`**, same `100..199` row band |
| `gui_load_game_tick` | `OnClick__11GM_LoadGameFUi` | reversal `(count-1)-(id-100)`, close result **`0x13`**, textdb **`0xaf`** into component **`0x3f2`** |
| `gui_load_game_ctor` | `OnCreate__11GM_LoadGameFv` | textdb **`0xac`** into component **10**; the range write on component 4 |
| `gui_load_progress_ctor` | `OnCreate__15GM_LoadProgressFv` | 8000-byte buffer, **`0x44`** stride, caption `+0x20`, textdb id `+0x40`, constant **`0x2ce`** |
| `gui_load_progress_set_progress_*` | `SetProgressGameInit` / `SetProgressMissionOpen` | the two formulas verbatim |
| `gui_question_box_tick` | `OnClick__14GM_QuestionBoxFUi` | results **`0xb`**/**`0xc`** and the root-vs-child fork |
| `gui_free_ride_ctor` | `OnCreate__11GM_FreeRideFv` | car-name fallback textdb **`0x2bf`** under the same range test |
| `gui_main_menu_exit_box_tick` | `OnClick__15GM_MainMenuExitFUi` | result **`0x18`** |
| `gui_action_menu_pick_up` | `PickUp__13GM_ItemPickUpFv` | a statement-level diff of both bodies matches statement for statement |
| `gui_gun_shop_refresh_slots` | `RefreshInventory__10GM_GunShopFv` | STRUCTURAL: the PS2 body is a 16-byte stub, pinned by two identical call positions |

Further correspondences, keyed to the PC sections they concern:

* **Question box (§3).** `GM_QuestionBox` dispatches components `0x8a`/`0x8b` and returns
  `0xb`/`0xc`; those ids live in design **1**, so that class is the design-1 question box. Its
  constructor flag only picks the console layout `psx_questionbox`; the Yes/No pair is always
  there. The design-**0** notice panel has no PS2 counterpart (§10c).
* **Action menu (§4d).** The field map is 1:1 with `GM_ItemPickUp` at a constant `+0x101c` delta:
  PC `+0x1048`/`+0x104c`/`+0x1050`/`+0x1054`/`+0x1058`/`+0x1060` ↔ PS2
  `+0x2c`/`+0x30`/`+0x34`/`+0x38`/`+0x3c`/`+0x40`. One behaviour is RELOCATED, not folded: the PS2
  auto-takes a lone option inside `OnCreate`; the PC runs the same test — same `& 2` flag on the
  same mission field — in the tick. Both builds have both functions, so this is not an inlining.
* **Gun shop (§4e).** The twin class is `GM_GunShop`.
* **Game over (§5).** The free-ride tail — disable *Retry*, focus *Exit to Main Menu* — is
  identical on both builds.
* **Load game (§6).** `gui_load_game_ctor` writes the `'swnd'` + `0x28` scroll range directly at
  the place where `OnCreate__11GM_LoadGameFv` calls the widget's `SetRangeFloat`.
* **Garage select (§9).** A size ratio taken against the `GM_GarageSelect` methods uses the full
  608 bytes of `gui_garage_select_tick`, not the 96 bytes before the mid-body split.

### 10b. Folds — PC inlines what the PS2 calls

Each of these carries a `// PS2-INLINED HERE:` banner at the fold site in the annotated decomp.

| PC function | what it absorbs |
|---|---|
| `gui_options_controls_on_exchange` | `SetUpTexts__18GM_OptionsControlsFv` — a real method `OnCreate` CALLS; the PC inlines it **twice** (here and in the tick's `0x35` arm) |
| `gui_load_progress_set_progress_game_init` | `SetRawProgress__15GM_LoadProgressFf` |
| `gui_load_progress_set_progress_mission_open` | `SetRawProgress__15GM_LoadProgressFf` |
| `gui_profile_create_tick` | `OnKey__16GM_ProfileCreateFUcUi`, which on the PS2 calls `OnClick(0x66)` back through the vtable |
| `gui_question_box_ctor` | `OnCreate__14GM_QuestionBoxFv` |
| every `*_tick` | 1–5 of `OnClick` / `OnKey` / `OnTick` / `OnChange` / `OnMouse` — see §1b |

### 10c. PC-only, measured against the closed PS2 roster

* **Every `*_on_exchange`** (slot 1). No `GM_*` class in the PS2 TU overrides `OnExchange`; the
  console build cannot change language or resolution at runtime.
* **`gui_options_controls_close`** (slot 4) — the PS2 class has no `OnClose`/`MenuClose`
  override, and no mph/kph speedometer switch to commit.
* **`gui_message_box_ctor` / `_tick`** — the PS2 TU's only modal box is `GM_QuestionBox`.
* **The whole session browser** — the DirectPlay front end is PC-only.
* **The 15-row loading roster** — `GM_LoadProgress` has no network roster; its `OnTick` is an
  8-byte stub.
* **The five `freerideN.bmp` previews** and the free-ride location pickers.
* **The design-0 notice panel** (§3). At every call position of its constructors the PS2 builds
  nothing: its `C_entity::Process` arms for the two entity-script commands `0x100f` / `0x1010`
  only advance the script, its `GM_GunShop` is gutted (`OnKey` and `RefreshInventory` are
  stubs), and the remaining callers are the multiplayer front end.
* **`GM_GarageGarage` and `GM_GarageComplete`** — PS2-only: they are reached only through PS2
  script commands the PC lacks (`structs/garage.md` §4c).
* **The combo drop-down's click-away dismissal** (§3a) — `GM_ComboMenu` has no such arm; the
  console build has no mouse.
* **The key-capture and key-conflict screens** (§4b) — `GM_OptionsControls::OnKey` merely cycles
  a 0..3 control PRESET, and the closed `GM_*` roster has no capture or conflict class.
* **The inventory as a `GM_Menu` screen** (§4c) — the feature exists on the PS2, but that build
  draws the inventory in the HUD (`G_IndicatorsClass::_InventorySet`, `::InventoryScroll`,
  `::DrawInventory`, all taking a `G_Inventory *`). Measured against the closed roster plus a
  named alternative location.

---

## 11. Open items

| item | state |
|---|---|
| `GM_LoadGame::{IsTexturePreloaded, PreloadTextures, ReleasePreloadedTextures}` | no PC counterpart located, not searched. The PC binds `shot%03d.bmp` by name instead of pre-loading `ITexture`s |
| `GM_LoadProgress::Destroy` | no PC counterpart located, not searched |
| every `GM_PressStart` / `GM_GameMap` / `GM_TutorialMessage` / `GM_HotRodMessage` method | the four console-shell screens; no PC counterpart located, not searched |
