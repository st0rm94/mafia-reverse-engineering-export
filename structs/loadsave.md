# `G_LoadSaveClass` and the `g_loadsave` translation unit

**Created** 2026-08-23 16:22 +0200 · **Modified** 2026-10-11 21:37 +0200

The PC translation unit `[0x00556f10, 0x00561b70)` — 176 function starts, all named. This
document is the PC's; PS2 material is in §9 and in the twin cells of the coverage tables.

`G_CarDatabase` is the second singleton in this same TU and has its own document:
[car_database.md](car_database.md).

---

## 1. What the translation unit is

One file holding **five classes** and two file-scope singletons. Its boundaries come from the
MSVC dynamic-initializer table at `0x0064615c`, read as an ordered list:

```
0x00646164: 0x00556ee0   crt_static_init_guard_00556ee0    the G_IndicatorsClass TU closes
0x00646168: 0x00556f10   loadsave_static_init              <- this TU opens
0x0064616c: 0x00556f50   car_database_static_init          the SECOND singleton
0x00646170: 0x00561b40   crt_static_init_guard_00561b40    this TU closes
0x00646174: 0x00563b00   crt_static_init_guard_00563b00    the autosalon TU closes
```

The two init groups resolve to ctor `loadsave_ctor` / dtor `loadsave_dtor` and ctor
`car_database_ctor` / dtor `car_database_close`. **Two adjacent groups at the head of a
block is the fingerprint of a TU with two file-scope objects** — that single observation
establishes the unit's identity without reading a body.

| class | PC prefix | starts | what it owns |
|---|---|---|---|
| `G_LoadSaveClass` | `loadsave_` | 57 | profiles, savegame slots, replay, objectives, freeride, settings, credits |
| `G_CarDatabase` | `car_database_` | 37 | the three `.def` tables and packed car ids — see [car_database.md](car_database.md) |
| `G_Stream` | `stream_` | 13 | the encrypted savegame stream |
| `G_EnabledCars` | `enabled_cars_` | 1 | the four-bank car permission bitmap |
| `G_GarageClass` | `garage_` | 16 | the Free Ride garage |

The remaining starts are CRT scaffolding, five MSVC file-I/O routines, two STL helpers,
two credits free functions and `textdb_unload`.

> ⚠️ **`data_unload_def_tables` is NOT in this translation unit.** It is reached from
> `loadsave_close`'s call lockstep, but its VA `0x00592240` lies in `[0x00592050, 0x005922c0)`,
> PS2 `g_material.cpp`; see `structs/def_tables.md`. A caller tells you who calls a function,
> never where it lives.

⚠️ **`G_GarageClass` state lives inside the `G_LoadSaveClass` object on the PC.** Every
`garage_*` function here takes the same `this` as the `loadsave_*` functions: the steal
mask is `+0x98`, the Free-Ride enabled-car mask `+0x9c`, the 25-slot car table `+0xa0`.

---

## 2. Field map of the `G_LoadSaveClass` singleton

Offsets are PC. Every row has at least one named reader or writer in the decomp.

| offset | size | field | established by |
|---|---|---|---|
| `+0x00` | 0x0c | profile vector (begin/end/cap), **stride 0x54** | `loadsave_profile_enum_files`, `copy_range_stride_0x54` |
| `+0x10` | 0x0c | savegame-slot vector, **stride 0x18** | `loadsave_savegame_enum_files`, `stl_insert_n_stride_0x18` |
| `+0x20` | 0x54 | the ACTIVE profile record; its `+0x10` is the profile name (`g_loadsave_active_profile_name`, `0x00671fe0`) | `loadsave_profile_select` copies into it; `loadsave_profile_create` compares names at record `+0x10` |
| `+0x74` | 0x18 | the ACTIVE savegame-slot record | `loadsave_savegame_select` copies into it |
| `+0x7c` | 4 | active slot number | `loadsave_savegame_get_selected_number` |
| `+0x8c` | 4 | replay mode: `-2` record, `1` play, `0` off | `loadsave_replay_set_mode` / `_tick` |
| `+0x91` | 1 | profile-load-failed flag | set by `loadsave_ctor`, cleared on a good load |
| `+0x94` | 4 | **race-unlock mask** | `loadsave_race_cars_enable` / `_disable` / `_get_enabled` |
| `+0x98` | 4 | **steal-permission mask** | `garage_enable_steal_for_letter`, `garage_can_steal_car_model` |
| `+0x9c` | 4 | **Free-Ride enabled-car mask** — bit `n` = car group `'A' + n` | `loadsave_freeride_enable_car_group`, `loadsave_freeride_get_enabled_cars` (§9e; `structs/garage.md` §2b) |
| `+0xa0` | 25 × 8 | garage table: `{packed_car_id, C_car*}` | `garage_clear`, `garage_add_car_by_*`, `garage_is_car_parked` |
| `+0x168` | 25 × 4 | previous-scene SNAPSHOT of the table above | `garage_show_results` diffs the two, then re-snapshots |
| `+0x1d4` | 1 | the one-shot save-suppress flag (§9h) | `loadsave_savegame_load` sets it, `loadsave_savegame_from_script` reads and clears it |
| `+0x1d8` | 0x0c | objectives vector (dword string ids) | the four `loadsave_objectives_*` |
| `+0x1e8` | 4 | the last car the player got into | written by `garage_set_last_entered_car` (from `human_use_actor` on every car entry, `engine_new_mission`, `human_save_game_load`); read by `garage_game_done` (`structs/garage.md` §2a) |
| `+0x1ec` | 4 | the car the player cannot get out of | `garage_set_blocked_car` |
| `+0x1f0` | 4 | **Free Ride mode word** | `loadsave_freeride_set_mode`; `>2` gates the script params |
| `+0x1f4` | 0x2d0 | **key-binding table, 60 entries × 0xC** | the five `loadsave_binding_*` / `loadsave_default_key_bindings` |
| `+0x4c4` | 0x5c | settings block (volumes, gains, flags) — §2a | `loadsave_default_settings`, `loadsave_apply_settings` |
| `+0x520` | 0x9c | **the saved multiplayer settings** `g_net_settings_saved` — player name, connection provider, address, port and serial settings (`structs/net_session.md` §4a) | `net_session_settings_save` writes it, `net_session_settings_apply_saved` hands it back to the session; zeroed by `loadsave_all_default` |
| `+0x5bc` | 0x18 | six dwords, cleared and never used (below) | `loadsave_ctor` |
| `+0x5d4` | 4 | the load-progress screen shown while a savegame reopens its mission | `loadsave_savegame_load` creates it (`operator_new(0x50)`, `gui_load_progress_ctor`) and destroys it after `game_init`; `loadsave_ctor` zeroes it |

The profile file streams `+0x20` (0x54), `+0x1f4` (0x2d0), `+0x4c4` (0x5c) and `+0x520`
(0x9c) in that order — `loadsave_profile_load` and `loadsave_profile_save` are exact
inverses, which is how the block sizes above are byte-proven.

**`+0x5bc` is touched only by the constructor's clear.** No instruction references its absolute
range `0x0067256c..0x00672583`; inside this translation unit the clear in `loadsave_ctor` is the
only access at displacements `0x5bc..0x5d3`; and none of the 125 instructions that load the
singleton's address `0x00671fb0` is followed by an access in that range. It is not part of the
profile stream.

### 2a. The settings block `+0x4c4..+0x51f`

The singleton sits at `0x00671fb0`, so every settings field is also reachable as a fixed global,
and the rest of the game reads them under those names. The Options screens edit them through
their `menu.def` components.

| offset | global | what |
|---|---|---|
| `+0x4c4` | `g_steering_linearity` | steering linearity slider |
| `+0x4c8` | `g_blood_level` | "Level of Blood" slider |
| `+0x4cc`..`+0x4d7` | - | §10 |
| `+0x4d8` / `+0x4dc` | `g_aiming_sensitivity_x` / `_y` | aiming sensitivity sliders |
| `+0x4e0` | `g_max_aiming_speed_x` | "Max Aiming Speed X" slider |
| `+0x4e4` | `g_saved_sounds_volume` | sounds volume |
| `+0x4e8` | `g_music_volume` | music volume |
| `+0x4ec` | `g_cars_volume` | car sounds volume |
| `+0x4f0` | `g_speech_volume` | speech volume |
| `+0x4f4` / `+0x4f8` | `g_camera_last_onfoot_mode` / `_last_vehicle_mode` | the remembered camera modes; `loadsave_apply_settings` resets them to 1 / 7 when out of range |
| `+0x4fc` | `g_gamma` | gamma slider |
| `+0x500`..`+0x510` | `g_force_feedback_global_volume`, `_suspension`, `_collisions`, `_revs`, `_tire_grip` | the five car force-feedback sliders; `loadsave_apply_settings` (bit `8`) passes them to `car_set_force_feedback_gains` (`structs/force_feedback.md` §5c) |
| `+0x514` | `g_force_feedback_strength` | "Player Movement" force-feedback slider |
| `+0x518` | `g_cursor_style` | cursor style button |
| `+0x51c` | `g_speed_unit_is_mph` | speedometer unit |
| `+0x51d` | `g_side_roll_enabled` | Side Roll toggle |
| `+0x51e` | `g_mouse_control_inverted` | mouse Y inversion |
| `+0x51f` | `g_subtitles_enabled` | Subtitles toggle |

---

## 3. The on-disk layout

Two file families, two four-character sigils, both written through `G_Stream`.

| path | sigil | version | content |
|---|---|---|---|
| `savegame\mafia%03d.sav` | `'Prof'` (`0x50726f66`) | 1 | one PLAYER PROFILE: name, settings, key bindings, unlock masks |
| `savegame\mafia%03d.%03d` | `'SavG'` (`0x53617647`) | 0x10 | one SAVE SLOT of that profile |
| `savegame\reply%03d.r%02d` | - | - | a race replay: setup blob + 0x4000 dwords of input |

`loadsave_profile_delete` removes the `.sav` **and all 600** slot files, which is what
fixes the per-profile slot cap at 600. Slot *names* come from the text database: slot *i*
uses string id **5000 + i**, and `loadsave_savegame_enum_files` skips any slot whose id
does not resolve — so the number of visible save slots is a localisation-table property,
not a code constant.

The replay path literal is `savegame\reply%03d.r%02d`. **"reply" is the developers' own
typo** and is present on both builds; it is not a transcription error here.

---

## 4. 🔑 The savegame cipher — a complete spec, and PC-only

`stream_ctor` seeds two accumulators with values that are **dates**:

```
key1 = 0x23101976     // 23.10.1976
key2 = 0x10072002     // 10.07.2002
```

Encryption walks the buffer one dword at a time (a trailing partial dword is left in
clear):

```
encrypt:  key2 += plaintext;  ciphertext = plaintext ^ key1;  key1 += key2;
decrypt:  plaintext = ciphertext ^ key1;  key2 += plaintext;  key1 += key2;
```

The two are exact inverses, which is how `stream_decrypt_dwords` is confirmed — against
its own sibling rather than against the PS2, because **the PS2 bodies of `Crypt` and
`Decrypt` are empty 8-byte stubs** (§9d). A `.sav` reader written from this section alone
will round-trip.

`stream_set_params(magic, encrypt, flag2)` decides whether a given stream is encrypted at
all; `stream_write_at` copies to scratch before encrypting so a caller's buffer is never
mutated in place.

---

## 5. Free Ride

The mode word at `+0x1f0` drives everything. The script parks its world parameters in a
file-scope block first and they are published only once the scene is live:

```
loadsave_freeride_store_params(mode, flags, ratio_a, ratio_all, ratio_c)
        -> g_freeride_stored_mode / _flags / _traffic_car_ratio / _traffic_all_ratio /
           _traffic_ratio_c

loadsave_freeride_apply_script_params()          // only when mode > 2
        g_freeride_stored_traffic_car_ratio     -> g_traffic_car_elems_coef   (0x00656154)
        g_freeride_stored_traffic_all_ratio     -> g_traffic_ratio_all        (0x0064c058)
        g_freeride_stored_traffic_police_ratio  -> g_traffic_car_police_coef  (0x0065615c)
                                                   and g_traffic_ratio_police (0x0064c05c)
        this+0x98 = this+0x94                // steal mask := race-unlock mask
```

`g_traffic_car_elems_coef` is the traffic element-count coefficient, `g_traffic_car_police_coef`
weights the police spawn entries in `traffic_car_game_init`, and `g_traffic_ratio_police` has no
reader on the PC (§9j).

The last line is a real gameplay fact: **in Free Ride you may steal every car you have
unlocked for racing.**

Scene construction is `loadsave_freeride_setup_scene` (builds the `Tommy` and
`TomyhoAuto` frames and loads both models) followed, once the scene is live, by
`loadsave_freeride_setup_after`, which seats the player in the car.

---

## 6. Replay

A fixed **0x4000-dword** ring of packed control words in `g_replay_frame_buffer`, with
`g_replay_frame_cursor` as the index and `g_replay_input_word` as the live word. Recording
and playback are the same function, `loadsave_replay_tick`, switched by the sign of the
mode field; the control word is masked to `0x7f`, so **at most seven control bits are
recorded**.

`loadsave_replay_load` also carries a racing setup blob, handed to
`racing_load_replay_setup` when `g_racing` is non-null — which is the only reason the
replay system touches the racing module at all. `loadsave_replay_save` is PC-only.

---

## 7. The key-binding table (PC-only)

60 entries × 0xC bytes at `+0x1f4`, in two bands: **0..0x20** and **0x21..0x3b**.
`loadsave_binding_find_conflict` and `loadsave_binding_clear_conflicts` both walk the band
an entry belongs to, so the split is real and not cosmetic.

The record is `{int mode; short device_kind; ushort code; int device_index;}`, `mode`
selecting whole axis (0), positive half (1) or negative half (2). It is read by
`input_set_binding` / `input_update` / `input_binding_get_label`, and the shipped label
strings decode it end to end — see [input.md](input.md) §1a:

| `device_kind` | meaning |
|---|---|
| 1 | keyboard scancode |
| 2 | **mouse**, buttons and axes: `1`/`2`/`4` = the three buttons, `0x100`/`0x200`/`0x300` = axes X/Y/Z |
| 3 | **joystick**, `device_index` selecting it: `code < 0x100` an axis, `0x100`..`0x1ff` a button, `>= 0x200` a POV hat ¹ |

¹ Written only via `loadsave_binding_set_from_event`. A stale `device_index` renders as the
shipped string "Not attached joy".

The whole family is PC-only — the PS2 has no rebindable input — and the table is streamed
into the profile file as one 0x2d0 block.

### 7a. The action-label table `g_key_binding_label_string_ids` — 60 textdb string ids

A second 60-entry table, **60 dwords at `0x0064d668`**, gives each binding slot its
on-screen name. `loadsave_get_key_binding_label_id(slot)` (`0x00560120`) returns
`table[slot]` for `0 <= slot < 0x3c` and the fallback id **100** otherwise. Decoded against
`textdb.json` the entries are exactly the Controls-screen action names, and **the two bands of
§7 fall out of the data**:

| slot range | contents |
|---|---|
| 0..0x17 | on-foot actions, in PAIRS (each action has a primary and a secondary binding): 301 `Forwards`, 302 `Backwards`, 303 `Left`, 304 `Right`, 305 `Walk / Run`, 307 `Action Button`, 308 `Fire`, 309 `Crouch`, 311 `Jump`, 312 `Aiming`, 317 `Cycle Up` (x4) |
| 0x18..0x1f | the unpaired on-foot singles: 319 `Inventory`, 320 `Hide Weapon`, 321 `Drop Weapon`, 322 `Toggle Run / Walk`, 323 `Reload`, 324 `Sniper Mode`, 325 `Objectives`, 343 `City Map` |
| 0x20 | **0 — no label.** The last slot of band A, unused |
| 0x21..0x38 | in-car actions, again in pairs: 330 `Accelerate`, 331 `Brake / Reverse`, 332 `Steer Left`, 333 `Steer Right`, 334 `Handbrake`, 335 `Speed Limiter`, 337 `Change Gears Up`, 338 `Change Gears Down`, 339 `Horn`, 340 `Look Left`, 341 `Look Right`, 342 `Use Clutch` |
| 0x39..0x3b | the in-car singles: 336 `Manual/Auto Gearing`, 310 `Change camera`, 344 `Replace car (racing)` |

The split at slot `0x20`/`0x21` is the same one `loadsave_binding_find_conflict` and
`loadsave_binding_clear_conflicts` walk, so **band A is on-foot and band B is in-car** —
a key may be bound to one action on foot and a different one in a car without conflicting.

`loadsave_get_key_binding_label_id`'s two callers are GUI row constructors in the
`GM_Menu` framework cluster `[0x00566280, 0x0056adf0)`; each stores the slot index in
its own object and pushes the resolved label into element `0xc`. The function belongs to
`G_LoadSaveClass` — the class that owns the whole binding family — not to the framework
it is called from.

---

## 8. Coverage

**176 of 176 starts named**, 32 of them PC-only; the rest carry a PS2 twin cell in the
function registry. Six functions that would read as `C_game`'s wear the class prefix instead,
all six body-diffed CONFIRMED (§9h): `loadsave_savegame_create`, `loadsave_savegame_load`,
`loadsave_save_actor_proc`, `loadsave_savegame_save`, `loadsave_savegame_from_script` and
`loadsave_execute_credits`. `game_` is `C_game`'s alone (`game_init` = `Init__6C_gameFbb`).

One row is special: `ProfileGetList`'s PC code is shared with two other owners (§9i).

---

## 9. PC vs PS2

Everything cross-build lives here.

### 9a. The unit's identity

The PS2 file is **`g_loadsave.cpp`** (`__sinit_g_loadsave.cpp` at `0x004c8bc0`), emitted
immediately after `g_indicators.cpp`. With this TU, **PC and PS2 link order agree across
seven consecutive translation units**. PS2 code range: `0x002b61d0`–`0x002bfcd0`,
156 symbols against the PC's 176 starts.

`G_GarageClass` is the exception: on the PS2 it is a separate class at
`0x0048a5c0`–`0x0048efb0` in its own file, and the PC folded its state into the
`G_LoadSaveClass` object.

### 9b. The three locksteps that carried the identification

| PC function | PS2 method | what matched |
|---|---|---|
| `loadsave_close` | `Close__15G_LoadSaveClassFv` | five statements, same order, including the `profileIgnoreSave` guard. Named five functions and one global at once |
| `loadsave_all_default` | `AllDefault__15G_LoadSaveClassFv` | call for call, including the argument pair `RaceCarsDisable(-1)` then `RaceCarsEnable(1)` and the `SaveGameNewGame` tail call |
| `garage_game_init` / `garage_game_done` | `GameInit`/`GameDone__13G_GarageClassFv` | pinned by CALLER NAME — `game_init` and `game_done`, exactly as PS2 `C_game` calls them |

### 9c. Where symbol RANK disagrees with the call graph, the call graph wins

* `loadsave_default_key_bindings` (`0x00559630`) occupies `DefaultSettings`' rank, but its body
  is ~90 keyboard-scancode writes and it is PC-only. The real twin of `DefaultSettings` is
  `loadsave_default_settings` (`0x00559ae0`), two ranks later, sharing the `0.5f`/`1.0f`
  constants **and** the `AllDefault(0xffff)` callsite.
* Rank put `GarageShowResultsProc`/`CarManager` on `0x0055f510`/`0x0055fa90`. PS2 `CarManager`
  builds a `GM_GarageSelect` menu **and calls `DestroyCar`**: that is
  `garage_run_car_select_menu` (`0x0055f510`), which builds a menu and calls
  `garage_destroy_car` (`0x0055fa90`). `GarageShowResultsProc` is called *from* `GameDone` on
  both builds, which is `garage_show_results` (`0x0055fcb0`). Two independent edges, both
  agreeing.

### 9d. 🔴 Eight PS2 methods in this unit have no real body

Read as raw instruction words, each is 8 bytes. Seven are `jr ra; nop`, i.e. empty: `_Sesyp`,
`SetCanHaveBigWeapon` and `SetInCar` (`G_Inventory`), `Crypt` / `Decrypt` (`G_Stream`) and
`GenerateCars` / `ReleaseCars` (`G_GarageClass`). The eighth, `HaveSpaceFor`, is
`jr ra; li v0, 1`: it always answers yes, and the decompiler drops the delay-slot load and shows
`{ return; }`. On the PC the inventory and stream ones are real code: the port gutted the PC's
four-band inventory and its savegame encryption. `GenerateCars` / `ReleaseCars` are empty on both
builds: their script arms (`GARAGE_GENERATECARS` / `GARAGE_RELEASECARS`) call the shared empty
`stub_noop_cdecl_00574a40` on the PC.

⇒ **A PS2 method count is an upper bound whose entries can be hollow.** Five rows in this unit
are graded STRUCTURAL for exactly this reason (`HaveSpaceFor`, `SetCanHaveBigWeapon`, `_Sesyp`,
`Crypt`, `Decrypt`): no code diff is possible, so they rest on field, rank and arity instead.
`SetInCar` is CONFIRMED on its callers (`structs/inventory.md` §3).

### 9e. PS2 methods with no PC start

`RemoveInHand` is inlined into `human_notify_callback` as `inventory_remove_slot(0, 0)`
(`structs/inventory.md` §3).

**`AddLastStolen` and `ShowResults` ARE located.** Their PS2 bodies store 1 to `+0x7d` / `+0x7c`;
the PC twins `garage_request_add_last_stolen` / `garage_request_show_results` store 1 to `+0x1cd` /
`+0x1cc`. Both builds call them from the `GARAGE_ADDLASTSTOLEN` / `GARAGE_SHOW` arms, and both
`GameDone` bodies test the pair in one condition and open the results screen on the first byte
alone. `ShowResults` is a flag set, not a forwarder, which is why `garage_show_results` (484 B,
called from `GameDone`) is `GarageShowResultsProc`.

**Nine have no PC realisation**, measured at every PS2 caller:

| PS2 method | PS2 callers | at that position on the PC |
|---|---|---|
| `Vysum__13G_GarageClassFP5C_car` | InterCity code, `C_game::InvalidateActor`, the `C_car` destructor | `game_invalidate_actor` and `car_dtor` call nothing in this unit and touch no garage slot |
| `AutoVybuchlo__13G_GarageClassFP5C_car` | `C_car::Update` (twice) | `car_update` calls nothing in this unit and touches no garage slot |
| `CanStart__13G_GarageClassFPUc` | `C_player::AI_drive` (twice) | `player_ai_drive` starts the engine on any pedal above 0.25 with no car-mask test |
| `GetCarsByRaceMask__13G_CarDatabaseF…` | `C_racing::EndRace`, `G_GarageClass::DoFreerideMenu` | `racing_end_race` makes no car-database call; `DoFreerideMenu` is PS2-only (`structs/garage.md` §4) |
| `EnableCarsForFree__13G_CarDatabaseFUi` | `FreerideEnableCar` | `loadsave_freeride_enable_car_group` has no such call (below) |
| `GetWreckageByID__13G_CarDatabaseFi` | `C_racing::PreloadWrecks` | `racing_game_init` has no wreck preload (`structs/racing.md`) |
| `WriteAdrAtPos__8G_StreamFPvUiUi` | `SaveGameSave` (twice: the actor count and the header, written back) | `loadsave_savegame_save` closes the stream right after the actor walk, and `loadsave_savegame_load` reads `0x8c`-byte actor records until the stream ends |
| `ObjectivesComplete__15G_LoadSaveClassFUi` | the `C_program` arm of an objectives-complete command | the PC command table has only `MISSION_OBJECTIVES`, `MISSION_OBJECTIVESREMOVE` and `MISSION_OBJECTIVESCLEAR` |
| `ObjectivesGetList__15G_LoadSaveClassFv` | `GM_GameMenu::OnCreate` | `gui_game_menu_ctor` only builds its components; the PC shows objectives through `loadsave_objectives_display` |

**`FreerideEnableCar` IS located.** Its PC twin is **`loadsave_freeride_enable_car_group`** (VA
`0x0055ff70`), CONFIRMED: both bodies decode the letter `A`..`Z` / `a`..`z` to a bit index and OR
that bit into the mask the accessor below returns (PS2 `+0x2b8`, PC `+0x9c`). The PS2 body also
maps the digits `0`..`6` to bits 26..32, remaps `U` to `0`, and ORs
`G_CarDatabase::EnableCarsForFree` into `+0x2b4`; the PC has none of the three.

**`FreerideGetEnabledCars` IS located.** It is the 8-byte accessor `{ return *(this + 0x2b8); }`.
Its PC twin is **`loadsave_freeride_get_enabled_cars`** (VA `0x0055ffb0`,
`{ return *(this + 0x9c); }`), whose three call sites all load `ecx = 0x00671fb0` — this
singleton. Two one-line accessors match on shape alone, so the grade rests on the call POSITION,
which is decisive: in the Carcyclopedia's car-list builder both builds take this accessor's
result, OR in `0x2000000` and hand it to `GetCarsByCarcyclopediaMask` /
`car_database_collect_cars_by_carcyclopedia_mask` — the same edge on both builds. ⇒ `this+0x9c`
is the **Free-Ride enabled-car mask** (§2). See `structs/carcyclopedia.md` §6e.

The whole `PSX*` family (memory-card handling, ~35 methods) is PS2-port-only.

### 9f. Divergent record strides

| record | PC | PS2 |
|---|---|---|
| car index (`carindex.def`) | **0xa4** | 0xa8 |
| carcyclopedia detail | **200 (0xc8)** | 204 (0xcc) |
| player index (`playerindex.def`) | 0x24 | 0x24 |
| profile header (`G_SaveGame_FileHead`) | 0x54 | 0x54 |

### 9g. Two PS2 symbols read against their bodies

* `GetWeaponInLeftHand__11G_Inventory` is `return gameItemTable;` on both builds: it returns the
  `predmety.def` table base without reading the inventory. It is still a `G_Inventory` method —
  both callers pass the human's inventory — so the PC name is `inventory_get_left_hand_item_def`.
* `RaceCarsAddWin__15G_LoadSaveClassFUi` unlocks whatever a car unlocks; nothing in the
  body mentions a win. PC name: `loadsave_race_cars_unlock_from_car`.

### 9h. The six class-prefix rows, body-diffed

Ordered by PC VA, PC name first.

| PC VA | PC name | PC size | PS2 method | PS2 size | grade | what the diff turned on |
|---|---|---|---|---|---|---|
| 0x00557ec0 | `loadsave_savegame_create` | 352 | `SaveGameCreate__15G_LoadSaveClassFUib` | 416 | **CONFIRMED** | the `id < 100000` split; the find-by-`entry+0x08` loop tail-calling SaveGameSelect(index); the else-arm that starts at id 1 and resets its cursor to 0xffffffff on every collision; `insert(.., 1, &zeroed)` then SaveGameSelect(count-1) |
| 0x00558180 | `loadsave_savegame_load` | 3120 | `SaveGameLoad__15G_LoadSaveClassFv` | 3672 | **CONFIRMED** | THREE OpenRead passes on both builds; `FreerideStoreB(0)`; SetParams(0x53617647,1,0); `C_game::Done` before, `C_game::Init` between; the inlined mission-name strcmp; `GM_LoadProgress` ↔ `gui_load_progress_ctor`; `Open(..,0,0xffffffff,1)` ↔ `mission_open(..,0,0xffffffff,1)`; SaveGameLoadGuid; ai_groups/ai_follow_manager Load; the tail sets the suppress flag and runs `GameAfterLoad` |
| 0x00558db0 | `loadsave_save_actor_proc` | 288 | `_SaveProc__15G_LoadSaveClassFP8G_StreamP7C_actorP20G_SaveGame_ActorHeadUiRUi` | 356 | **CONFIRMED** | statement for statement: strncpy 0x40 + NUL at [0x3f]; [0x84] = size; [0x80] = kind; the `== 9` sub-name branch with strrchr('\\') then strrchr('/') and +1; [0x88..0x8b] = -1; WriteAdr(0x8c); WantWrite; the virtual SaveGameSave slot; Write |
| 0x00558ed0 | `loadsave_savegame_save` | 1088 | `SaveGameSave__15G_LoadSaveClassFb` | 1456 | **CONFIRMED** | both 'SavG' spellings (0x53617647 and 0x47766153); the 100-default score with the `0 -> 1` fixup; the packed GetLocalTime words; ai_groups + ai_follow_manager GetSize then Save; the four freeride globals; the two-pass actor walk gated on kind 4/8 then the rest — which re-confirms `loadsave_save_actor_proc` |
| 0x00559320 | `loadsave_savegame_from_script` | 112 | `SaveGameFromScript__15G_LoadSaveClassFUibb` | 3056 | **CONFIRMED** ¹ | the one-shot suppress flag read-and-cleared in the else arm; the script id store; `SaveGameCreate(id - 5000)`; SaveGameSave; SaveGameEnumRelease; ProfileSave; and the tail `if (verbose) ConsoleAddTextByID(0x89b, 0xffffff)` — same id, same colour |
| 0x00560690 | `loadsave_execute_credits` | 512 | `ExecuteCredits__15G_LoadSaveClassFv` | 1104 | **CONFIRMED** ² | dtaOpen/dtaSeek(0,2)/`< 1` bail/LoadFont(1,0)/`size / 0x8c`/new/dtaSeek(0,0)/dtaRead/dtaClose/ParseCredits/the `+= dt * 0.03` scroll/the `array + count*0x8c - 0x88` terminator/delete/LoadFont(0,0) |

¹ **The 27× size ratio is not a mismatch** — every byte of the difference is the PS2 memory-card
user interface (the `GM_PSX_*` menu tree, `MC_GetMemoryCardsInfo`, `MC_SaveData`, `MC_Format`,
the format/overwrite question boxes), which the PC build has no counterpart for. Strip that and
the two functions are the same six calls in the same order; the shared literals (5000, 0x89b,
0xffffff) and the shared callee sequence carry the row.

² `loadsave_execute_credits` keeps its helpers on the `credits_` prefix: `credits_parse` and
`credits_render_one` twin `ParseCredits__Fv` and `RenderOneCredit__FP15G_CreditsStructf`, which
are FREE functions on the PS2, not `G_LoadSaveClass` members. Only `ExecuteCredits` is a method.

🔑 **The suppress flag is a cross-function proof.** PS2 `+0x2c0` / PC `+0x1d4` is SET at the
end of `loadsave_savegame_load` and READ-AND-CLEARED at the head of
`loadsave_savegame_from_script`, on both builds. One field, two functions, two twins — each
row corroborates the other.

### 9i. `ProfileGetList` has no PC symbol of its own — it is one owner of a three-way fold

`ProfileGetList__15G_LoadSaveClassFv` compiles to `return this`, because the profile vector
sits at **offset 0** of the singleton. On the PC that is the three-byte body
`mov eax,ecx; ret` at **`0x005575f0`**, named `folded_return_this` — and identical-COMDAT
folding put **three unrelated source functions on that one address**:

| owner | how its callsites reach it | callsites |
|---|---|---|
| **`G_LoadSaveClass::ProfileGetList`** — this class | `mov ecx, 0x671fb0`, then `[eax+4]` | 9 |
| the trivial `C_blood` constructor | `operator_new(0x24)`, `mov ecx,eax` — `structs/blood.md` | 1 |
| no class: a one-argument identity inline on a time value | `mov ecx,[esp+0x14]` in `recorder_set_clock` — its first stack argument | 1 |

Read the row as *"this class's `ProfileGetList` is realised by the code at `0x005575f0`"*, never
as *"`0x005575f0` is `ProfileGetList`"*, and expect no `PS2 twin:` stamp on the PC symbol.

Method: a branch-aware census of the receiver at each callsite. The body is *not* byte-unique —
`identity_thunk` (`0x0045aea0`) is an unfolded second copy — so uniqueness is not what proves
this; the three disagreeing receivers are.

⚠️ Consequence for §8: every PS2 method still has located PC code, but `ProfileGetList` is the
one row whose PC code is **not exclusively its own**. There is no dedicated function to find.

### 9j. The Free Ride traffic knobs

The four globals `loadsave_freeride_apply_script_params` publishes to (§5) are identified by their
PS2 names: `g_traffic_car_elems_coef` = `elems_coefff__13C_traffic_car`, `g_traffic_ratio_all` =
`ratio_all__19C_traffic_generator`, `g_traffic_car_police_coef` = `police_coefff__13C_traffic_car`
and `g_traffic_ratio_police` = `ratio_police__19C_traffic_generator`.

---

## 10. Open items

| item | state |
|---|---|
| settings block `+0x4cc`..`+0x4d7` (§2a) | not decoded |
| in game: profile delete | delete a profile and confirm all its save slots disappear with it (§3) |
| in game: Free Ride stealing | start a Free Ride and check that every car unlocked for racing is stealable (§5) |
