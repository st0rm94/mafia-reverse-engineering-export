# The PC multiplayer front end — `gui_mp_*`

**Created** 2026-08-24 11:05 +0200 · **Modified** 2026-09-17 17:54 +0200

PC translation units `[0x00563b30, 0x00566250)` (the menus) and the second class inside
`[0x00561b70, 0x00563b00)` (the driver-appearance picker), plus six `inet.dll` helpers around
`0x00597190`. Prefix **`gui_`** with the class as the second token, per `structs/gui_screen.md`.
The PS2 build has no networking at all; every function in this file is PC-only. Cross-build
material is in §9.

---

## 1. What it is, and how to reach it

Retail PC Mafia ships a **complete DirectPlay multiplayer front end** — a mode menu, a
session lobby with chat and an eight-slot participant list, a connection-settings screen
with TCP/IP and serial support, and a driver-appearance picker. Nothing is stubbed. What
retail removes is the **door**, in the same belt-and-braces way it removed Racing
(`basic/artifacts/racing.md` §3):

* `tables/menu.def`'s main-menu design (`0x06`) has **no `meti` record `0xf1b9c`**, and
* `gui_main_menu_ctor` calls `gui_menu_set_component_enabled(0xf1b9c, 0)` and
  `gui_menu_delete_component(0xf1b9c)` at construction time, so the item is
  deleted at runtime if it ever reappears.

**`0xf1b9c` was the "Multiplayer" item**, on three independent witnesses:

| witness | what it says |
|---|---|
| `gui_main_menu_tick` (`0x00576a20`) | `case 0xf1b9c:` returns menu_choice **`0x14`**, and `engine_main` case `0x14` calls `net_multiplayer_run` |
| `g_menu_camera_position_table` (`0x0064daf0`) | pairs `0xf1b9c` with the `00menu` camera frame **`multiplayer_pos`** |
| `textdb` | **104 = "Multiplayer" / "Multijoueur" / "Mehrspieler" / "Multiplayer"**, unused, sitting between 103 "Load Game" and 105 "Tutorial" |

### 1a. The reachable route — through the Racing menu

`net_multiplayer_run` has a **second** caller: `race_menu_run` case `0x2e`, reached from the
Racing menu's own item **`0xf1ccd`**, which is labelled `textdb 413 = "Multiplayer"` and
**does ship in retail `menu.def`** (design `0x16`: *Racing: / Single Race / Championship /
Multiplayer / Exit*).

So in a build where the Racing menu is reachable — i.e. with the community racing mod,
which restores main-menu item `0xf1b9b` — the path is:

```
Main Menu → Racing → Multiplayer → Host SingleRace → the lobby → "Person:"
```

🔴 **The racing mod's own patch touches this id.** It redirects `gui_main_menu_ctor`'s two
`push 0xf1b9b` operands to `push 0xf1ccd` (`basic/artifacts/racing.md` §4) — the Racing menu's
Multiplayer item. The redirect is nevertheless safe, because
`gui_menu_set_component_enabled` and `gui_menu_delete_component` walk only the
CURRENT screen's child vector (`screen+0x20`/`+0x24`) and `gui_main_menu_ctor` is building the
main menu — so both redirected calls are no-ops and the Racing menu's Multiplayer entry is
left untouched.

### 1b. The runtime gate — DirectPlay

`net_multiplayer_run` calls `net_session_init_provider` first. That function creates the INet driver
through the `inet.dll` import `_CreateINetBase@4`; on failure it zeroes both the driver
pointer (`g_net_session+0x10c`) and the session role (`g_net_session+0x100`) and stores
textdb **900** in `+0x1fc`. `net_multiplayer_run` then refuses with a message box —
textdb **907 "Network error!"** over **900 "Error in Direct Play initialization!"** — and
returns to the menu.

⇒ On a modern Windows install, where DirectPlay is an optional feature that is off by
default, the menu item is reachable and the mode is not. That is a *runtime* block, not a
data one.

---

## 2. The Multiplayer menu — design `0x28`, vtable `0x0063d3b4`

`operator_new(0x48)` — the bare `GM_Menu` size; the class adds no members.

| Item id | textdb | label | what `gui_mp_menu_tick` does |
|---|---|---|---|
| `0x65` | 601 | **Host Deathmatch** | `session+0x38 = 1`, return `0x37` |
| `0x66` | 602 | **Host Team Deathmatch** | `session+0x38 = 2`, return `0x37` |
| `0x67` | 603 | Host SingleRace | `session+0x38 = 3`, return `0x37` |
| `0x68` | 604 | Host Championship | `session+0x38 = 4`, return `0x37` |
| `0x6e` | 605 | Join existing game | return `0x36` |
| `0x6f` | — | *(no record anywhere)* | opens `gui_mp_message_screen_ctor` (§6) |
| `0x78` | 606 | Settings | opens `gui_mp_settings_ctor` (§5) |
| `0x82` | 607 | Back to Main Menu | return `0x38` |

🔴 **`0x65` and `0x66` — the two DEATHMATCH modes — are fully implemented in code and have
no `menu.def` record in retail.** Both `gui_mp_menu_ctor` and
`gui_mp_menu_refresh_mode_items` compute their enable state from bits 1..4 of the session's
supported-mode mask `session+0x4c`, and `gui_mp_menu_tick` writes the mode. The labels ship
in all four languages. Only the two records are gone.

The `0x36` (Join) return is handled by `net_multiplayer_run`, which opens the "Select session:"
browser `gui_session_list_ctor` (`0x0057b150`, design `0x2f`, textdb 610; `structs/game_menus.md`).

The whole mask is forced to zero unless `net_session_selected_provider_is_valid` confirms the
configured connection provider still exists, so a machine with no usable provider gets every
host and join item greyed out.

---

## 3. The driver-appearance picker — design `0x2c`, vtable `0x0063d398`

This is the class that sat unnamed inside the `GM_Carcyclopedia` translation unit. It is
**not** a car viewer: it selects the **character model your driver wears**, out of
`tables\playerindex.def`.

### 3a. Field map of the `0x68`-byte object

Conf: **H** measured on ≥2 independent sites · **M** one site, unambiguous · **U** unproven.

| Offset | Field | Conf | Evidence |
|---|---|---|---|
| `+0x00` | vptr → `0x0063d398` | H | installed by `gui_mp_person_select_ctor`; 6 slots |
| `+0x04` | the `missions\carcyclopedia\scene.i3d` scene object | H | created by the ctor through the I3D driver factory (type `0xd`); released by `gui_mp_person_select_on_close` |
| `+0x18` | the **parent** screen | H | `GM_Menu` base field — see §7 |
| `+0x40` | framework flag word | H | ctor clears bit `0x00010000` |
| `+0x48` | **selected `playerindex.def` index** | H | seeded from `*param_4`, stepped by the tick, passed to `gui_mp_person_select_show_model` |
| `+0x4c` | preview object A (I3D factory type 9) | H | `S_matrix::SetDir3` to one facing; gets the model + `walk1.i3d` |
| `+0x50` | preview object B (I3D factory type 9) | H | same model, the opposite facing |
| `+0x54` | the orbit camera (factory type 3) | H | placed on a sphere of radius 8 by the tick |
| `+0x58` | the fill light (factory type 2) | H | colour `(1,1,1)`, cone `0x3fc00000`, parented to the scene root |
| `+0x5c` | **turntable yaw, degrees** | H | tick wraps it into `[0, 360)` and feeds `sin`/`cos` of `yaw * 0.017453292` |
| `+0x60` | **camera pitch, degrees** | H | tick clamps it to 10..80 |
| `+0x64` | `int*` — where to write the accepted index | H | stored from `param_4` by the ctor; written on accept; read back by `gui_mp_lobby_on_child_result` |

### 3b. What is on screen

The ctor loads the showroom set `missions\carcyclopedia\scene.i3d`, then clones the
design's `seli` list-row template (id 1000, FourCC `'seli'`) once per `playerindex.def`
record, labelling each row with that record's model name, and grows the enclosing scroll
window (`swnd` id 4, 240×528) by the accumulated row height.

`gui_mp_person_select_show_model(this, index)` — `__thiscall`, `ret 4`, the second argument
invisible in the decomp — resolves the index through `car_database_get_player_model`, loads
the model into **both** preview objects, binds the shared `walk1.i3d` clip at weight 1.0
through frame vtable `+0x60`, and moves the list selection to row `index + 1000`.

### 3c. 🔑 Why `walk1.i3d` is not an anomaly

`car_database_get_player_model` reads `G_CarDatabase+0x0c`, the buffer of
**`tables\playerindex.def`** — 2376 bytes / 0x24 = **66 records** of
`{char model[0x20]; u32 civil_mask}`, every one a `civilNN.i3d` **character** model
(`structs/car_database.md` §1). The previewed model is a person, so a human walk clip is
exactly right.

`car_database_get_player_model` is named for what it reads: a character model for a driver
(PS2 name and consumer in §9).

The chosen index lands in **`g_net_session+0x228`**, the session's driver-model field.

---

## 4. The lobby — design `0x2a`, vtable `0x0063d3cc`

`operator_new(0x58)`. This is where a session is configured and started.

**One design serves four layouts.** `gui_mp_lobby_ctor` builds design `0x2a` with the gate
`1 << session_mode`, shifted left by 8 when this machine is a CLIENT
(`g_net_session+0x200 == 0`), so the four `gate` records in the shipped file — `8`, `0x10`,
`0x800`, `0x1000` — select host/client × single-race/championship. A fifth gate `0x1010`
carries the row labels and `0xffff` the chat panel.

| Component | id | role |
|---|---|---|
| participant rows | `100`..`0x6b` | nine `netp` rows; `gui_mp_lobby_tick` fills eight from `g_net_session+0x268` (humans) then `+0x294` (AI slots) |
| chat log | `200`..`0x6b`+ | seven `stxt` lines, textdb 702 `<no messages>` when empty |
| chat input | `10` | an `edtl`; ENTER sends it and clears it |
| Circuit / Championship | `11000` (`0x2af8`) | textdb 660 / 665 |
| **Garage:** | `0x2af9` | textdb 661 — opens the car picker `gui_race_car_maker_ctor` (`structs/racing_menus.md`), result into `g_racing+0x64` |
| **Person:** | `0x2afa` | textdb 662 (cz *Postava:*) — opens §3, result into `g_net_session+0x228` |
| laps − / + | `0x2afb` / `0x2b05` | steps `g_racing+0x27c`, clamped 1..99; shown on `ntxt` `0x3f5` |
| Direction: | `0x2afc` | flips `g_racing+0x2b4`; textdb 688 "Normal" / 689 "Reverse" |
| circuit / championship picker | `0x2afd` | opens §6 |
| damage | `0x2afe` | `racing_set_damage_enabled`, `g_racing+0x2b7` |
| Cancel / Start / Race / Ready | `0x1e` / `0x14` / `0x28` | textdb 690 / 692 / 693 / 691 |
| Add AI Opponent | `0x32` | opens the car picker, then `net_session_add_ai_racer` |
| Generate All Opponents | `0x3c` | enabled while `g_racing+0x64 != -1` |

The Garage and Person buttons are greyed once the local player has readied up
(`g_net_session+0x252`).

---

## 5. Multiplayer Settings — design `0x2b`, vtable `0x0063d3e4`

`operator_new(0x168)`. The only class in these two TUs with its own scalar deleting
destructor and its own destructor, because it owns seven combo-item vectors.

| Offset | Field |
|---|---|
| `+0x5c`/`+0x60`/`+0x64` | the session list |
| `+0x6c`… | the connection-provider list |
| `+0x7c` `+0x8c` `+0x9c` `+0xac` `+0xbc` | COM port, baud rate, stop bits, parity, flow control lists |
| `+0xc8` | the screen's working copy of the 0x9c-byte settings block |
| `+0xe8` | selected provider TYPE (1 / 2 / 4) — decides the design gate |
| `+0xec` | selected session |
| `+0x150`…`+0x160` | the five serial selections |
| `+0x164` | the "advanced block is visible" flag |

The combo contents are the module's own string literals at `0x0064d9a0..0x0064da68`:
`Com1`..`Com4`, `9600`/`14400`/`19200`/`38400`/`56000`/`57600`/`115200` (each row carrying
its numeric value), `No parity` / `Odd` / `Even` / `Mark`, and `No flow` / `RtSf flow` /
`Dtr flow` / `RtSdTr flow` / `XonXoff flow`.

`gui_mp_settings_tick`'s OK arm (`0x26d`) reads every widget back, applies the block to the
session through `net_session_settings_apply`, and calls the settings-committed hook.

---

## 6. The championship picker and the dead-item screen

`gui_race_champ_select_ctor` (design `0x17`, vtable `0x0063d404`) clones design `0x17`'s
`rari` round template `0x1b8` ("ROUND 1", textdb 440) once per entry of the `g_racing+0x1c`
championship vector; `gui_race_champ_select_tick` maps `0x1b8`..`0x1cb` to
`racing_set_current_champ` and `0x1a7` to EXIT.

`gui_mp_message_screen_ctor` (design `0x23`, vtable `0x0063d41c`) is the odd one out. It
adds no members and overrides nothing — vtable `0x0063d41c` is **byte-identical** to
`0x0063d404`, so it inherits the championship tick, which handles none of design `0x23`'s
component ids. Design `0x23` in the shipped file is the bonus-car award layout (textdb 780
"You have won a bonus car"), but the *working* bonus-car screen is a different class,
`gui_race_bonus_car_ctor` (vtable `0x0063d56c`, four parameters, called from `racing_end_race` case
`0x33`). Its only entry point is the Multiplayer menu's item `0x6f`, which has no record in
any design and is disabled unconditionally. As shipped, the screen closes on ESC and does
nothing else.

---

## 7. The `GM_Menu` framework

Measured across these five vtables; the detail lives in `structs/gui_screen.md`.

| Slot | Default | Role |
|---|---|---|
| 0 | `gui_menu_scalar_deleting_dtor_common` | scalar deleting destructor (calls `gui_menu_dtor`); `gui_menu_scalar_deleting_dtor` on the base |
| 1 | `gui_menu_on_exchange_forward_to_parent` | rebuild in place; the default forwards to the **parent** at `this+0x18` |
| 2 | `gui_menu_destroy` | close: destroy every child component, then delete self |
| 3 | `gui_menu_on_close_child` | a child closed / a control changed; the default re-samples the mouse |
| 4 | `gui_menu_close` | pop the stack: `g_gui_active_screen = this->+0x18`, notify it through slot 3, then close |
| 5 | *(per class)* | per-frame tick + input |

🔴 **`screen+0x18` is the PARENT, not a child.** `gui_menu_close` assigns it to
`g_gui_active_screen` on close and then notifies it, and every screen constructor here is
called as `ctor(g_gui_active_screen, …)`.

**The GUI design table** `g_gui_screen_design_table` (`0x00672630`) is the parsed
`tables\menu.def`. `gui_menu_build_components_from_design(this, table, design_id, gate)` walks `stop`-delimited
groups `design_id` times — byte-verified, `cmp ecx, 0x73746f70` at `0x0056e5c3` — so the
group index **is** the design id. 72 designs, ids `0x00..0x47` (the last 16 empty), records
of 0x24 bytes `{u32 id, FourCC tag, float x, y, w, h, u32 text_id, u32 flags, u32 colour}`.
Eight ids are verified one-for-one against the shipped file: `0x06` main menu, `0x0d`
carcyclopedia, `0x16` racing menu, `0x17` championship, `0x28` multiplayer menu, `0x2a`
lobby, `0x2b` multiplayer settings, `0x2c` driver picker. The loader that fills the table at
start-up is `gui_screen_design_table_load` (`0x0056af70`, `structs/gui_screen.md` §6).

---

## 8. Coverage — PC, ordered by VA

**33 of 33 starts named** across the two ranges plus the six network helpers.
Every row is PC-only; the PS2 column exists for the one framework default that has a twin.

| PC VA | PC name | PS2 twin | Grade |
|---|---|---|---|
| 0x00562fa0 | `gui_mp_person_select_ctor` | – | **CONFIRMED** |
| 0x005634a0 | `gui_menu_on_exchange_forward_to_parent` | `OnExchange__7GM_MenuFv` | **STRUCTURAL** |
| 0x005634b0 | `gui_mp_person_select_on_close` | – | **CONFIRMED** |
| 0x005634d0 | `gui_mp_person_select_tick` | – | **CONFIRMED** |
| 0x005639e0 | `gui_mp_person_select_show_model` | – | **CONFIRMED** |
| 0x00563b30 | `gui_mp_menu_ctor` | – | **CONFIRMED** |
| 0x00563c20 | `gui_mp_menu_refresh_mode_items` | – | **CONFIRMED** |
| 0x00563cc0 | `gui_mp_menu_tick` | – | **CONFIRMED** |
| 0x00563ee0 | `gui_mp_lobby_ctor` | – | **CONFIRMED** |
| 0x00564110 | `gui_mp_lobby_on_child_result` | – | **CONFIRMED** |
| 0x00564320 | `gui_mp_lobby_tick` | – | **CONFIRMED** |
| 0x00564f20 | `gui_mp_settings_ctor` | – | **CONFIRMED** |
| 0x00565610 | `gui_mp_settings_scalar_deleting_dtor` | – | **CONFIRMED** |
| 0x00565630 | `gui_mp_settings_dtor` | – | **CONFIRMED** |
| 0x00565760 | `gui_mp_settings_on_exchange` | – | **CONFIRMED** |
| 0x00565a70 | `gui_mp_settings_read_widgets` | – | **CONFIRMED** |
| 0x00565ac0 | `gui_mp_settings_write_widgets` | – | **CONFIRMED** |
| 0x00565bc0 | `gui_mp_settings_on_change` | – | **CONFIRMED** |
| 0x00565d00 | `gui_mp_settings_tick` | – | **CONFIRMED** |
| 0x00565db0 | `gui_mp_settings_set_advanced_visible` | – | **CONFIRMED** |
| 0x00565e90 | `gui_mp_settings_resize_window` | – | **CONFIRMED** |
| 0x00565f00 | `gui_race_champ_select_ctor` | – | **CONFIRMED** |
| 0x005660f0 | `gui_race_champ_select_tick` | – | **CONFIRMED** |
| 0x00566190 | `gui_mp_message_screen_ctor` | – | **CONFIRMED** |
| 0x00595d90 | `race_menu_run` | – | **CONFIRMED** |
| 0x00596370 | `net_multiplayer_run` | – | **CONFIRMED** |
| 0x00597190 | `net_session_init_provider` | – | **CONFIRMED** |
| 0x005974c0 | `net_session_enumerate_providers` | – | **CONFIRMED** |
| 0x00597710 | `net_session_settings_addr` | – | **CONFIRMED** |
| 0x005977e0 | `net_session_settings_apply_saved` | – | **CONFIRMED** |
| 0x00597800 | `net_session_settings_apply` | – | **CONFIRMED** |
| 0x005978b0 | `net_session_selected_provider_is_valid` | – | **CONFIRMED** |
| 0x005978f0 | `net_session_find_provider_index_by_type` | – | **CONFIRMED** |

Two further starts in `[0x00563b30, 0x00566250)` are shared STL helpers that belong to no
class and keep their mechanical names: `vector_dispose_at_0x04` (`0x00566200`, **531**
references, 0 datarefs) and `copy_range_stride_0x8` (`0x00566220`, 21 references).

---

## 9. PC vs PS2

**Everything here is PC-only.** The PS2 build ships no networking: `Game.exe` imports
`_CreateINetBase@4` from `inet.dll`, and the PS2, having no DLLs, would have had to compile
that in — it did not (`structs/net_session.md`). The PS2's `GM_*` roster
(109 classes) contains **no** multiplayer menu, no lobby, no connection-settings screen and
no character picker; `GM_NetPlayer` exists as a class with a `G_Factory<>` but nothing
drives it.

The one exception is the framework default `gui_menu_on_exchange_forward_to_parent`, whose
twin `OnExchange__7GM_MenuFv` is COMDAT-emitted into `g_m_carc.cpp` on both builds — 16
bytes each, the same forward through a member pointer, different member offset (PS2
`+0x28`, PC `+0x18`) and different slot index (PS2 5, PC 1), so the mapping is STRUCTURAL.

`car_database_get_player_model` ↔ `GetPlayerModel__13G_CarDatabaseFUi`. Its PS2 consumer is
`GameInit__10C_opponentFP17I3D_animation_setUi` — a race opponent building its **driver's**
character model from `+0x17c`, chosen by code. Only the PC lets a human choose it.

---

## 10. Open items

| item | state |
|---|---|
| `g_net_session+0x100`'s full enum | `0` = no provider was ever created, `7` = host, `5` and `1` appear in `gui_mp_lobby_tick`'s exit codes; `2`, `3`, `4` and `6` are unexplained |
| deathmatch | `session+0x38` modes 1 and 2 have labels, enable logic and a mode write, but nothing downstream of `net_multiplayer_run` is traced for them; whether a deathmatch session can run is unknown |
| in game (modded install, DirectPlay enabled) | confirm that Racing → Multiplayer opens the mode menu, that Host SingleRace reaches the lobby, and that the **Person:** button lists 66 `civilNN.i3d` rows with a walking preview |
