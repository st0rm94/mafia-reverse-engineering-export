# HUD elements & the `hud_state` singleton (`VA 0x00658330`)

**Created** 2026-05-30 22:55 +0200 · **Modified** 2026-10-04 19:30 +0200

The engine HUD-state singleton, the functions that operate on it, the script commands that drive HUD
elements, and the per-bit / per-field map of what is known and what is not. Everything here is anchored
by VA, function name, struct offset or bit mask. The per-method PC↔PS2 table for the class lives in
`structs/hud.md`.

---

## 1. The singleton

A single hardcoded BSS object at **`VA 0x00658330`** holds the entire HUD state. ~311 byte-references in `Game.exe`, all to this one address. Canonical name here: **`hud_state`**. It is read every frame by the per-frame HUD draw/update function (reads the `+0x90a4` flag word + the per-element fields below).

**Class identity.** The singleton is an instance of **`G_IndicatorsClass`** (evidence in §8). Per-method table and the PC↔PS2 field-offset correspondence: **`structs/hud.md`** — this file documents the PC side.

**The PC code for this class is not contiguous** — it occupies four separate ranges: `0x00548420`–`0x0054d180` (the bulk), `0x005540b0`–`0x00555200` (`hud_tick` + the text/font engine), plus the satellites `0x004538b0` (`airplane_hud_damage_toggle`) and `0x00424e20` (`hud_show_element`).

The central field is the **`+0x90a4` flag word** — a 32-bit set of per-element visibility bits (§4). Note `*(0x00658330 + 0x90a4) == DAT_006613d4` (abs `0x006613d4`); much of the code toggles the flag word via that **flat global**, which is why some bit-sets are invisible to `hud_show_element`/`hud_hide_element` searches.

---

## 2. Functions that operate on `hud_state`

| Function | VA | Touches | Role |
|---|---|---|---|
| `hud_show_element(mask)` | `0x00424e20` | `+0x90a4` (OR mask) | set element-visibility bit(s) |
| `hud_ctor()` / `hud_dtor()` | `0x00548bf0` / `0x00548b00` | the seven vector members | construct / destroy the singleton. The dtor frees the seven vectors in exact reverse construction order. |
| `hud_reset_values()` | `0x00548d50` | almost every element field | return every element to defaults (health 100, ammo 16/30, accuracy 1.0f, 4 wingman slots 0/100, stopwatch from `GetLocalTime`, theft bar 0.5f, police symbol 0/alpha 0xffffffff, both radar lists, race buffers). |
| `hud_load_font(use_credits_lang, index)` | `0x00548f90` | `+0x9070 + index*4`, metric block `hud + index*0x101c` | load `font<i><lang>.tga` + `tablesont<i><lang>.gmf`. Four font slots. The language records are `g_font_language_primary` (primary) and `g_font_language_alternate` (credits). |
| `hud_language_select(lang)` | `0x00549060` | the language record | build/load `tables\lang_<xx>.def`. |
| `hud_create()` | `0x00549120` | `+0x9080`..`+0x90a0` + texture globals | one-time load of every HUD texture — the GUI window art `g_gui_papir5a_texture` / `g_gui_papir3_texture` / `g_gui_sponka_texture` / `g_gui_kulicky_texture`, which only the menu widgets draw (`structs/gui_widget.md` §5), `logoplay`, `2sight` (scope), `racestart`/`racewrong`, `2blood`, `1int`, `1numbers`, `0textura`, **`radarclp` → `g_radar_clip_texture`**, `2intbenal`; ends with `hud_reset_values`. |
| `hud_destroy()` | `0x005492f0` | the same texture slots | release everything `hud_create`/`hud_load_font` acquired. Distinct from `hud_game_done` (per-scene). |
| `hud_set_police_symbol_and_alpha(sel, intensity)` | `0x00424e40` | `+0x96d8` (symbol 1..4), `+0x96dc` (`alpha<<24 \| 0xffffff`) | police status icon (§4) |
| `racing_activate_racers(racing)` | `0x00611430` | race text buffers + both radar lists + `g_hud_flags` | a `C_racing` method, run **once** per race from `racing_game_init`: seeds the whole race HUD, rebuilds the radar and installs the two track-edge strips from the current circuit (§6). The only caller that ever passes a real node run to the edge builders. |
| `railway_update(railway)` | `0x004da400` | radar OBJECT list | a `C_railway` method, not a HUD one: the tram tick registers the engine and every wagon as **yellow** radar objects when in range and removes them when out. Why trams have their own radar colour (§6). |
| `hud_tick(dt_ms)` | `0x005540b0` | `+0x96d4` accumulator + every timed field | **the per-frame ageing half of the HUD**: decrements the stopwatch, every subtitle/mission-help TTL, the fade timer, the value bar and the race-place alpha. Paired with `hud_render_all`. |
| `hud_render_all()` | `0x0054d180` | reads `+0x90a4` + most per-element fields | the per-frame HUD draw function; branches per element bit and emits each widget's quads. Master early-out mask `& 0x801`. Draws the indicator cluster (gate bit `0x4`): weapon-ammo via `hud_build_ammo_icon_quads`/`hud_build_ammo_mag_reserve_quads`, player-health + wingman slots via `hud_build_indicator_icon_quads`/`hud_build_indicator_value_digits`. |
| `hud_game_done()` | `0x00549590` | releases `+0x9090`, then resets 6 subsystems | HUD scene teardown: radar edges, Free Ride score, intro subtitles, race place, stopwatch interval. |
| `hud_load_speedometer_odometer_texture()` | `0x005494b0` | `+0x9084`, `g_transmission_indicator_texture` | loads the instrument-panel artwork `2int.tga` / `2intmph.tga` / `2intnoc*.tga` (unit system `g_speed_unit_is_mph`, night flag `C_game+0x2d44`) + the auto-transmission overlay. **PC-only.** |
| `hud_speedometer_set_values(speed, b, c)` | `0x0054a7a0` | `+0x916c` (km/h), `+0x9190`, `+0x91b4` | drives the in-car instrument cluster: `speed * 3.6`, three change-gated needles, and the odometer digit strip. |
| `hud_speedometer_set_gear(gear, show)` | `0x0054a780` | `+0x91e0` (`gear * 11.0`), `+0x91fc` | gear readout of the instrument cluster. |
| `hud_build_needle_quad(def, val, origin, vtx)` | `0x00549770` | needle scale `+0x91e4`/`+0x91e8` | rotates one needle by `angle = def.start + clamp((v-min)/range) * sweep` and emits its 4-corner quad. Builds every HUD needle: the speedometer's speed / rpm / fuel needles and the mission timer's hour / minute / second hands (callers `hud_speedometer_set_values`, `hud_rebuild_layout_for_resolution`, `hud_tick`). The needle definition/value records are `G_RafickaDef` / `G_RafickaValue`. |
| `hud_timer_get_time(out_h,out_m,out_s)` / `hud_timer_get_interval(out_s)` | `0x005496e0` / `0x00549710` | `+0x9150/54/58`, `+0x9168` | read the stopwatch back out — the reason those fields are the TIME VALUE, not hand geometry. |
| `hud_map_enable(enabled)` | `0x0054be40` | `+0x96e4`, `+0x96e8` | enable/disable the bit-`0x2000` map overlay and cache textdb `0x5f` ("Out of City Limits"). |
| `hud_console_clear()` | `0x0054be70` | `+0x93e0 = 0` | empty the console ring. |
| `hud_intro_subtitles_add_intern(id, min_dur)` | `0x0054b160` | `+0x9408` vector (stride 0x1c) | intro/cutscene subtitle formatter: parses the string's `~`-separated lines + `,`-separated header (duration, x%, y%, align `c`/`r`) against the 800×600 virtual screen. |
| `hud_intro_subtitles_queue_id_range(first, last)` | `0x0054b5a0` | `+0x9418` vector (stride 8) | queue a run of textdb ids as `{id, atoi(text)}` — the numeric prefix of each string is its cue time in ms. Element type `G_MIIntroSubtitlesWaiting`. |
| `hud_intro_subtitles_add_centered(id)` | `0x0054b7b0` | `+0x9408` vector | the "simple" intro-subtitle adder: centres the string at `(400, 300-12.5)`, height 25.0, duration 4.0. |
| `hud_intro_subtitles_clear()` | `0x0054ba60` | `+0x9408` and `+0x9418` | clears both intro containers. |
| `hud_radar_clear()` | `0x0054c5c0` | `+0x9494` and `+0x94a8` | empties **both** radar blip lists. |
| `hud_radar_set_player_car(car)` | `0x0054c620` | `+0x94a0` | the `C_car` the radar centres on. |
| `hud_radar_add_object_with_color(obj, rgb)` / `hud_radar_remove_object(obj)` | `0x0054c8a0` / `0x0054cab0` | `+0x94a8` list, stride 8 | **second radar list** — blips for `tDynamicCollObject`s (props/pickups), same upsert/erase shape as the car pair. |
| `hud_radar_set_left_edge(nodes)` / `hud_radar_set_right_edge(nodes)` | `0x0054cb10` / `0x0054cd10` | `g_radar_left_edge_vtx`/`g_radar_left_edge_points`/`g_radar_left_edge_count` and `g_radar_right_edge_vtx`/`g_radar_right_edge_points` | rebuild the radar's road-outline strips from a `std::vector<C_web_node*>` — i.e. from **nav-graph nodes**. Passing 0 tears them down. |
| `hud_text_measure(text, height, font, end)` | `0x00554fd0` | per-font metric block `hud + font*0x101c` | rendered pixel width of a string. Names the global `g_font_translation_tables`. |
| `hud_text_find_line_break(text, max_w, h, font, end)` | `0x00555090` | same font metrics | the **word-wrap primitive** behind subtitles / mission help / intro subtitles: last space before overflow, or immediately at a `~` hard break. |
| `hud_text_draw_line(...)` | `0x005551a0` | the font metric blocks | draws one line of HUD text |
| `hud_set_overlay_texture(key, flags, force)` | `0x0054c550` | `+0x9484`/`+0x9488`/`+0x948c` | change-gated swap of the full-screen overlay texture. **PC-only.** |
| `hud_set_screen_tint(a, b)` | `0x0054c530` | `+0x947c`/`+0x9480` | two-field setter of the full-screen tint, driven by the `.tck` replay recorder; `hud_render_all` draws the tint quad (§5) |
| `hud_hide_element(mask)` | `0x005c7db0` | `+0x90a4` (AND-NOT mask) | clear element-visibility bit(s) |
| `hud_element_is_visible` (test) | `0x005c7dd0` | `+0x90a4` | `return (mask & flag) != 0` (bit test) |
| `hud_set_police_wanted_bar(float)` | `0x005c8010` | `+0x96e0` (= `g_police_wanted_bar`, abs `0x00661a10`) | wanted-bar fill (script writes clobbered each frame by `police_manager_tick`) |
| `hud_console_print(string_id, rgba)` | `0x0054bf10` | → `hud_console_push_message` | resolve textdb id + enqueue a console line |
| `hud_console_push_message(this, text, color)` | `0x0054be80` | `+0x9274` strings (5×0x40), `+0x93b4` colors, `+0x93c8` TTL (5.0f), `+0x93dc` head, `+0x93e0` count | 5-slot HUD console message ring |
| `hud_radar_add_car_with_color(vehicle, rgb)` | `0x0054c630` | `+0x9494/+0x9498/+0x949c` — `vector<{C_car*, rgba}>`, stride 8 | **add a car to the radar** with a blip colour (alpha forced to 0xff), or recolour it if already listed. |
| `hud_radar_remove_car(vehicle)` | `0x0054c840` | same `+0x9494` list | **remove the car's blip from the radar** (find + erase-by-shift); the blip disappears (§6). |
| `hud_compass_set_destination_frame(frame)` | `0x0054c150` | `+0x9270` (target frame ptr) | compass target |
| `hud_timer_set_time(h,m,s)` | `0x005495f0` | `+0x9150/+0x9154/+0x9158` (+ resets `+0x9108/+0x90e4/+0x912c`) | stopwatch hands |
| `hud_timer_set_interval(...)` | `0x00549630` | timer fields | stopwatch value (analog `/60`, `*1/60` arithmetic) |
| `hud_fade_set(...)` | `0x0054c490` | `+0x9468..+0x9474` (state) + bit `0x8000` | general full-screen fade overlay (busted screen, `zatmyse` case 0xe9, cheat flash) |
| `hud_fade_get_state` / `hud_fade_is_active` | `0x0054c500` / `0x0054c520` | `+0x9468..` + bit `0x8000` | poll fade armed+active / test bit only |
| `hud_set_car_damage_bar_enabled(x)` | `0x005c8db0` | satellite global `g_hud_car_damage_bar_enabled` (NOT a `hud_state` field) | sole caller is `cardamagevisible` (case 0xea) |
| `hud_set_player_accuracy(this, val)` | `0x0054aa30` | `+0x9234` | setter for the bit-`0x20` element's scale/param (§6) |
| `airplane_hud_damage_toggle(actor, enable)` | `0x004538b0` | `+0x90a4` bit `0x40000`; actor `+0x16a`; `hud_state+0x9454` | toggle the airplane-damage gauge bar (bit `0x40000`); on enable writes the actor damage ratio to `+0x9454` + sets the bit; driver = `airplaneshowdamage` (script `case 0x14f`) |
| `hud_set_airplane_damage_fill(this, value)` | `0x0054c170` | `+0x9454` | value/fill of the bit-`0x40000` airplane-damage gauge; **single caller** = `airplane_hud_damage_toggle` |
| `hud_set_theft_indicator_value(this, value)` | `0x0054c160` | `+0x9450`; element bit `0x1000` | fill `[0,1]` of the **timed-action / theft progress bar**. The four writers are the car-theft path and two held-action timers (car theft `1.0 - remaining/total` with the steal time clamped 2000..30000 ms; a held action `(t-400)/1600`; a second `(t-200)/300`), each preceded by `g_hud_flags \|= 0x1000`. |
| `hud_wingman_set_portrait_for_slot(this, slot, portrait)` | `0x0054aa40` | `+0x9244 + slot*4` (4 entries) | slot-keyed wingman portrait setter; `slot < 4`; portrait masked `& 7` (domain 0..7) (§7) |
| `hud_wingman_set_indicator_value_for_slot(this, slot, value)` | `0x0054aa60` | `+0x9254 + slot*4` (4 entries) | slot-keyed wingman indicator-value setter; `slot < 4` (§7) |
| `player_set_wingman_indicator(player, slot, portrait)` | `0x005c8d50` | `player+0x6d8` (slot), `player+0x6dc` (portrait) — **on the PLAYER struct, not `hud_state`** | records the pinned (slot, portrait) pair; persisted to `.sav`; per-frame player-tick reapply pushes it into the HUD singleton (§7) |
| `hud_push_blood_splat(...)` | `0x0054bf40` | `+0x9424` ring (10 slots), `+0x944c` count; gated on bit `0x2000000` | the **screen blood splatter**: `hud_render_all` binds `g_hud_blood_overlay_texture` (`2blood.bmp`) and draws the ring with `count*2`. `[user-verified]` — blood stains appear on the player's screen when hit |
| `player_indicator_set_aimed(player)` | `0x004ccc80` | `+0x90a4` bits `0x20` / `0x8000000` | **C_player method** — recomputes the **aim-indicator (crosshair)** bit `0x20` and the weapon-ammo bit `0x8000000` from player state. Sets BOTH while the human's weapon class `+0x1e8` is in `[1,4)` — a firearm (`structs/human.md` §2c) — and clears both otherwise. Only **`0x20`** reads the vehicle slots (`+0x98`/`+0x9c`) + seat `+0xac`; **`0x8000000`** is NOT vehicle-gated (§4/§6). |
| `hud_set_player_health_value(value)` | `0x0054a9d0` | `+0x9208` (number) + `+0x9214` (`"%d"` text) | player-health number store (always-on indicator value) |
| `hud_set_player_ammo_mag_reserve(mag, reserve)` | `0x0054aa00` | `+0x920c`/`+0x9210` + `+0x9224` (`"%d/%d"` text) | weapon-ammo value store (counterpart of the ammo renderer) |
| `hud_subtitles_add(id, ttl_ms, clear_first)` | `0x0054aad0` | bit `0x40`; line vector `+0x93e8/+0x93ec/+0x93f0` | **SUBTITLE line adder** — word-wraps a textdb string via `hud_text_find_line_break` and pushes one `G_MISubtitleStruct` (stride 0x10, `+0xc` = TTL) per line. TTL ms, min 2000, else length-derived. |
| `hud_subtitles_clear()` | `0x0054aa80` | `+0x93ec = +0x93e8` | logical clear of the subtitle line list (bit `0x40`). |
| `hud_subtitles_add_from_record(record)` | `0x0054ae30` | bit `0x40`; same `+0x93e8` vector | second subtitle-adder overload taking a record whose `+8` is the textdb id. **PC-only.** |
| `hud_mission_help_add(id)` | `0x0054bb20` | bit `0x1000000`; line vector `+0x93f8/+0x93fc/+0x9400` | **MISSION-HELP text adder** — same word-wrap + `G_MISubtitleStruct` records as the subtitle list. TTL = `strlen(text) * 0x28 + 0x9c4`. |
| `hud_mission_help_clear()` | `0x0054bad0` | `+0x93fc = +0x93f8` | logical clear of the mission-help line list. |
| `hud_mission_help_set_time(ttl_ms)` | `0x0054be10` | walks `+0x93f8`, writes each element `+0xc` | overwrite the TTL of every queued mission-help line. Pins `G_MISubtitleStruct+0xc` as the per-line TTL. |
| `hud_widescreen_build_bars(ratio)` | `0x0054c180` | `g_hud_widescreen_bar_verts` (`0x006570c0`, 12 TLVERTEXes) | cinematic letterbox-bar geometry (width = 801·scale·ratio); paired with `hud_widescreen_fade_bars` |
| `hud_widescreen_fade_bars(show, duration_ms)` | `0x0054c420` | bit `0x4000`; `+0x9458/+0x945c/+0x9460/+0x9464` | fades the **cinematic letterbox bars** in or out; shaped exactly like `hud_fade_set`. Every located call site passes `(1, 250)` / `(0, 250)`. |
| `hud_set_race_start_flag(place)` | `0x0054cf10` | bit `0x200000`; `+0x94bc..+0x94c8` | **racing-mission** place/position indicator (prev→cur transition + alpha); `place` 4/-1 = hide |
| `hud_set_race_time_text(ms)` | `0x0054cfc0` | `+0x94cc` | **racing-mission** elapsed time (`"Time: %2d:%02d"`, textdb 0x2c8) |
| `hud_set_race_laptime_text(ms)` | `0x0054d000` | `+0x950c` | **racing-mission** current lap time (`"Lap Time: %2d:%02d"`, 0x2d0) |
| `hud_set_race_besttime_text(ms)` | `0x0054d040` | `+0x954c` | **racing-mission** best time (`"Best: %2d:%02d"`, 0x2c9) |
| `hud_set_race_position_text(place, total)` | `0x0054d080` | `+0x958c` | **racing-mission** position (`"Pos: %d/%d"`, 0x2ca) |
| `hud_set_race_lap_text(cur, total)` | `0x0054d0b0` | `+0x95cc` | **racing-mission** lap counter (`"Lap: %d/%d"`, 0x2cb) |
| `hud_set_race_banner_text(text, arg)` | `0x0054d0e0` | bit `0x400000`; `+0x960c` (strncpy 0x80), `+0x968c` | **racing-mission** banner/message text |
| `hud_freeride_update_score_text(score)` | `0x0054d130` | `+0x9690`/`+0x9694` (`"$%d"`, textdb 0xb3) | **Free Ride** money/score readout (the ONLY Free-Ride HUD writer here — unrelated to the racing-mission functions above) |

**Racing-mission HUD ≠ Free Ride.** The `Time / Lap / Best / Pos / Lap / banner / place` readouts belong
to the racetrack mission, which shares nothing with Free Ride mode; only
`hud_freeride_update_score_text` is Free Ride.

---

## 3. Script commands that drive HUD elements

| Command | Dispatch case | HUD effect |
|---|---|---|
| `timeron` / `timeroff` | `0x31` / `0x32` | show/hide stopwatch (bit `0x8`); `timeron` also sets the hand positions |
| `setcompass <frame>` / `setcompass -1` | `0x1d` | show (valid frame) / hide (-1) compass (bit `0x80`); sets `+0x9270` |
| `pm_showsymbol <1..8>` | `0x146` | police status icon (bit `0x4000000`) + `hud_set_police_symbol_and_alpha` (1=fine, 2=handcuffs, 3=gun, 4=searching) |
| `pm_showprogress <0/1>` | `0x147` | show/hide police progress bar (bit `0x10`) |
| `pm_setprogress <f>` | `0x148` | write `+0x96e0` bar fill (clobbered each frame by the tick — no lasting effect) |
| `cardamagevisible <0/1>` | `0xea` | show/hide car-damage bar (bit `0x10000`) |
| `console_addtext <id>` | `0xd8` | print textdb string `id` to the console ring (in white) |
| `airplaneshowdamage <actor>, <0/1>` | `0x14f` | show/hide the airplane-damage gauge (bit `0x40000`) |
| `wingman_setindicator <npc_id>, <portrait_id>, <slot>` | `0x79` | pin wingman portrait+value to slot 0..3 (one of bits `0x10000000`..`0x80000000`); see §7 |
| `wingman_delindicator <npc_id>, <slot>` | `0x7a` | clear pinned wingman state + hide one of bits `0x10000000`..`0x80000000`; see §7 |

The fade overlay, the radar and the blood-splatter ring are engine-driven, not script commands.

---

## 4. `+0x90a4` flag-word bit map

Confidence tags, used the same way in every row: **`[E]`** = identified in game by toggling the bit
through `hud_show_element`/`hud_hide_element` and observing the result · **`[user-verified]`** = the
element itself was observed in game · **`[PS2-CONFIRMED]`** = pinned by a body-diff against
`G_IndicatorsClass`. An untagged row is established from the PC code alone.

| Bit | Element | Notes |
|---|---|---|
| `0x00000001` | **Global HUD suppress** (master on/off) `[E]` | when SET the renderer early-outs and draws **no** HUD; when CLEAR the HUD draws normally. Inverted sense vs every other bit: `hud_show_element(0x1)` *hides* the whole HUD and `hud_hide_element(0x1)` restores it. Engine-internal — not intended to be driven through the script show/hide path. The renderer's early-out mask is `& 0x801`, i.e. bits `0x1` **and** `0x800` — not the `0x00800000` wrong-way bit. |
| `0x00000002` | **In-car instrument cluster (speedometer panel)** `[user-verified]` | the whole dashboard readout, drawn by `hud_render_all` under `& 2`, from the `+0x916c`..`+0x9200` band: speed needle (`+0x916c`), RPM needle (`+0x9190`), fuel needle (`+0x91b4`), gear panel (`+0x91e0`), odometer digit strip, and the A/M transmission character (`+0x91fc`). The low-fuel red dot draws at `scale×668, scale×559` when `+0x91fd` is set and the blink phase `+0x9200 < 0.25`. The speed-limiter banner bit `0x100` is tested **inside** this branch, which is why that bit only shows while in a car. Full breakdown: §6b. |
| `0x00000004` | **Indicator-cluster gate** | enclosing gate in `hud_render_all` for the bottom-screen widget cluster: the **weapon-ammo** element (`0x8000000`) **and** the player-health + 4 **wingman** slots (`0x10000000`..`0x80000000`). Those elements draw only when `0x4` is also set. Set as part of the combined mask `DAT_006613d4 & … \| 0x180006` / `\| 0x190006` (the standard gameplay HUD enable, which also sets bits `0x2`, `0x10000`, `0x80000`, `0x100000`) — which is why a single-bit `\| 4` search misses it. |
| `0x00000008` | **Timer / stopwatch** | `timeron`/`timeroff` (case 0x31/0x32) |
| `0x00000010` | **Police progress bar** | `pm_showprogress` (case 0x147) |
| `0x00000020` | **Aim indicator (crosshair)** `[PS2-CONFIRMED]` | set **directly** (`DAT_006613d4 \| 0x20`) by `player_indicator_set_aimed` (VA `0x004ccc80`); cleared by the same function + `hud_hide_element(0x20)` (one script case); renderer `& 0x20` branch draws via `+0x9234` (crosshair **spread**, widened per shot by `player_add_shot_accuracy_penalty`) / `+0x9264`. Toggling it via show/hide shows no change because the function recomputes it from player state every call. |
| `0x00000040` | **Subtitles** `[PS2-CONFIRMED]` | the in-game subtitle line list at `+0x93e8` (`G_MISubtitleStruct`, stride 0x10, `+0xc` = per-line TTL). Setters `hud_subtitles_add` (VA `0x0054aad0`) and `hud_subtitles_add_from_record` (VA `0x0054ae30`), clear `hud_subtitles_clear`. |
| `0x00000080` | **Compass** | `setcompass` (case 0x1d); target frame at `+0x9270` |
| `0x00000100` | **Speed-limiter banner** `[user-verified]` | the speed-limiter (legal-speed cap) banner — NOT the speedometer itself. Set/cleared by actor/vehicle code from the linked vehicle's speed field (`+0x590` vs `800.0`). Set in **6 sites across 5 functions**: `human_save_game_load` (VA `0x004901e0`, in both car-entry branches), `human_intern_use_car`, `human_update`, `player_ai_drive`, and `engine_new_mission` (several set it via the combined mask `\| 0x80102`). Renderer reads `& 0x100` inside the `& 2` branch. **Caveat:** driving it via `hud_show_element(0x100)` only shows the banner **while the player is in a car** — it is an in-car-context element, and the vehicle-state writers re-clear it on foot. |
| `0x00000200` | **Action prompt** ("!" context-action marker) `[E]` | exclamation mark above the player's health bar signalling an available context action (enter car / open door / talk to NPC). Engine auto-shows it when the player is near an actionable object. |
| `0x00001000` | **Timed-action / theft progress bar** `[PS2-CONFIRMED]` | fill at `+0x9450` via `hud_set_theft_indicator_value`. Shown while the player performs a held "thief" action: stealing a car (steal time from the car database, clamped 2000..30000 ms, bar = `1.0 - remaining/total`), and two other held actions gated at 400 ms (`/1600`) and 200 ms (`/300`). Every writer sets this bit immediately before pushing the value. |
| `0x00002000` | **Full map** `[E]` | the hold-key full-screen map overlay. Gated with `+0x96e4`. |
| `0x00004000` | **Cinematic letterbox bars** `[PS2-CONFIRMED]` | `hud_widescreen_build_bars` (VA `0x0054c180`) writes **twelve** TLVERTEXes into `g_hud_widescreen_bar_verts` (`0x006570c0`) — **two 6-vertex quads, a top bar and a bottom bar** — leaving a visible band `601*scale_y*ratio` tall, centred. `hud_widescreen_fade_bars` (VA `0x0054c420`) fades them in/out over a duration (`+0x9458`/`+0x945c` ms, `+0x9464` direction, `+0x9460` animated diffuse) and paints them opaque black. 🔑 The ratio argument settles it: `program_process_command` and the race-replay screen `race_winners_replay_begin` both pass **`0x3f0a60dd` = 0.5405405f = 1/1.85**, the CinemaScope aspect ratio; `hud_reset_values` seeds 0.5. A replay screen raising letterbox bars is what the geometry predicts. |
| `0x00008000` | **Full-screen fade overlay** (busted/fail screen, `zatmyse`, cheat flash) | `hud_fade_set`; state at `+0x9468..+0x9474`. General fade-to-colour overlay, not end-screen-specific. The bit alone produces no visible change via show/hide `[E]` — the overlay also needs its state block populated by `hud_fade_set`. |
| `0x00010000` | **Car-damage bar** | `cardamagevisible` (case 0xea) |
| `0x00020000` | **Sniper scope** (SNIPERMODE) `[E]` | full-screen scope overlay; engine shows it while the player holds the aim key with a scoped weapon equipped. |
| `0x00040000` | **Airplane-damage gauge** (small rectangle, top-centre) `[user-verified]` | shown by script command `airplaneshowdamage` (case 0x14f). `hud_render_all`'s `& 0x40000` branch draws a ~100×8px quad at ≈`(scale·350, scale·20)` + a value-driven fill. Used **only** in MISE16-LETISTE, where the player shoots down a plane from a car and this bar visualises the plane's accumulating damage — which is why it is never a normally-visible HUD element otherwise. Toggled by `airplane_hud_damage_toggle` (VA `0x004538b0`) for the airplane actor; the actor's damage ratio `(actor+0x1c8 + actor+0x1cc)/(2*actor+0xa8)` → `hud_state+0x9454` via `hud_set_airplane_damage_fill`. The two summed fields `actor+0x1c8` / `actor+0x1cc` are the plane's **two engines the player must shoot**; the gauge shows their combined condition, and `airplane_isdestroyed` (case 0x90, `airplane_is_destroyed` VA `0x00453190`) reports destroyed once **both** reach 0. Tier-1 anchor: string literal `s_AIRPLANESHOWDAMAGE_0064e950`. |
| `0x00080000` | **In-car instrument / RADAR panel** `[PS2-CONFIRMED]` | **the radar's visibility bit.** `hud_render_all` draws the whole panel under this bit: the backdrop sub-rect of `+0x9084` (`2int*.tga`), then the car blips (`+0x9494`), the object blips (`+0x94a8`) and the destination cross. It is never set on its own, which is why single-bit searches find no writer: the writers use the combined masks `\| 0x80002` (in-car, 5 sites) and `\| 0x180006` / `\| 0x190006` (the standard gameplay HUD enable, e.g. `racing_activate_racers`) — the same situation as bit `0x4`. |
| `0x00200000` | **Racing-mission place indicator** | current place/position with a prev→cur transition animation; set/cleared by `hud_set_race_start_flag` (VA `0x0054cf10`; fields `+0x94bc..+0x94c8`); `place` 4/-1 = hide. Racing-mission only ⇒ no visible change when toggled on a normal map. |
| `0x00400000` | **Racing-mission banner text** | banner/message string; set by `hud_set_race_banner_text` (VA `0x0054d0e0`; strncpy 0x80 → `+0x960c`). Racing-mission only. |
| `0x00800000` | **"Wrong way" warning** `[E]` | the WRONG-WAY message shown during the racing mission when the car drives the wrong direction. Part of the racing-mission HUD group with `0x200000`/`0x400000`. |
| `0x01000000` | **Mission-help text** `[PS2-CONFIRMED]` | the mission-help line list at `+0x93f8` (same `G_MISubtitleStruct` record type as the subtitle list). Setter `hud_mission_help_add` (VA `0x0054bb20`), clear `hud_mission_help_clear`, retime `hud_mission_help_set_time`. TTL = `strlen(text) * 0x28 + 0x9c4`. |
| `0x02000000` | **Screen blood splatter** `[user-verified]` | the `+0x9424` ring (10 slots) with count `+0x944c`, pushed by `hud_push_blood_splat` and drawn by `hud_render_all` with `g_hud_blood_overlay_texture` (`2blood.bmp`). Toggling the bit alone via show/hide produces no visible change `[E]` — the ring also needs entries pushed. |
| `0x04000000` | **Police status icon** | `pm_showsymbol` (case 0x146); symbol `+0x96d8`, alpha `+0x96dc` |
| `0x08000000` | **Weapon ammo display** `[E]` | weapon/ammo readout (icon + numeric counter at screen bottom; shows automatically while a weapon is equipped). Drawn by `hud_render_all` under cluster-gate bit `0x4`, via `hud_build_ammo_icon_quads` (VA `0x00549bc0`) + `hud_build_ammo_mag_reserve_quads` (VA `0x0054a080`). **Sole writer = `player_indicator_set_aimed`** (VA `0x004ccc80`), which *sets* this bit whenever the actor's weapon class `+0x1e8` is in `[1,4)` — a firearm in hand — and clears it otherwise — it is **not vehicle-gated** (only bit `0x20` depends on the vehicle slots), so the weapon HUD **stays up in a car**. |
| `0x10000000`–`0x80000000` | **Wingman-indicator 4-slot array** | one bit per slot (`0x10000000`=slot 0, `0x20000000`=slot 1, `0x40000000`=slot 2, `0x80000000`=slot 3); set by `wingman_setindicator` (case 0x79), cleared by `wingman_delindicator` (case 0x7a); per-slot HUD portrait stored at `+0x9244 + slot*4`, per-slot value at `+0x9254 + slot*4`; persistent pin lives on the **player struct** at `+0x6d8`/`+0x6dc` (see §7) |

**HUD-flag save/restore stack.** Menus, loading screens and cutscenes temporarily reconfigure the HUD by *pushing* the current `g_hud_flags` onto the `std::vector<uint>` at `+0x90b0` (fields in §5), setting a temporary flag word, then *popping* to restore. Examples: `gui_notice_panel_present_once` and `gui_menu_execute_modal` push then set `(g_hud_flags & 0x2c000) | 1` (suppress the HUD via bit `0x1` but keep sniper / fade / letterbox); the race-replay screen `race_winners_replay_begin` pushes then sets `g_hud_flags = 0x4000` (letterbox bars only), and `race_winners_replay_end` pops to restore. Push = `std_vector4_insert_n(end, 1, &g_hud_flags)`, pop = read `*(end-4)` then `std_vector4_erase_at`. `0x006613d4` is the HUD flag word on every one of these paths; the `float` typings Ghidra puts on some of these writes are type-inference noise.

---

## 5. Known `hud_state` fields (by offset)

| Offset | Meaning | Confidence |
|---|---|---|
| `+0x9070`..`+0x907c` | the four HUD font slots; each carries a metric block at `hud + index*0x101c`; setter `hud_load_font` | high |
| `+0x9084` | **instrument-dial texture** (`2int*.tga`); loaded by `hud_load_speedometer_odometer_texture`. Also supplies the backdrop sub-rect of the bit-`0x80000` radar panel | high |
| `+0x9090` | **full-map texture** (`0mapar.bmp`) — lazily loaded by `hud_tick` when the map-enabled bool `+0x96e4` is set and the slot is still null; bound by `hud_render_all` (texture slot `+0xc`, then a 2-triangle strip); released in `hud_destroy` / `hud_game_done` | high |
| `+0x90a4` | flag word (§4) — abs `0x006613d4` = `g_hud_flags` | high |
| `+0x90ac` / `+0x90b0` | **HUD-flag save/restore stack** — `std::vector<uint>` (begin = abs `0x006613dc`, end = abs `0x006613e0`); push = `std_vector4_insert_n`, pop = `std_vector4_erase_at` (§4) | high |
| `+0x90b8/+0x90bc/+0x90c0/+0x90c4` | screen-scale / resolution factors used by the renderer to place elements | high |
| `+0x90e4`, `+0x9108`, `+0x912c` | stopwatch companion fields (reset to -1.0f by `hud_timer_set_time`) | med |
| `+0x9150/+0x9154/+0x9158` | stopwatch hand positions (h/m/s) | high |
| `+0x916c` | **speed needle** value, km/h (`m/s * 3.6`); needle def `g_speedo_needle_def_speed`, quads → `g_speedo_needle_vtx_speed` | high |
| `+0x9190` | **RPM needle** value; def `g_speedo_needle_def_rpm`, quads → `g_speedo_needle_vtx_rpm` (§6b) | high |
| `+0x91b4` | **fuel needle** value; def `g_speedo_needle_def_fuel`, quads → `g_speedo_needle_vtx_fuel`. Drives the low-fuel warning — `hud_speedometer_set_values` raises `+0x91fd` when this is `< 0.02` | high |
| `+0x91e0` | **gear panel** value, pre-scaled `gear * 11.0` (the digit-atlas cell pitch); setter `hud_speedometer_set_gear` | high |
| `+0x91e4` / `+0x91e8` | needle x/y scale factors used by `hud_build_needle_quad` | high |
| `+0x91fc` | **transmission indicator** — selects the `A` vs `M` UV rect on the `2intauto*.tga` overlay (`g_transmission_indicator_texture`), drawn 32×32 at `scale×758, scale×558`. Written by `hud_speedometer_set_gear`'s 3rd arg (§6b) | high |
| `+0x91fd` | **low-fuel warning flag** — raised when the fuel value `+0x91b4 < 0.02` | high |
| `+0x9200` | **low-fuel blink phase** — advanced by `hud_tick` under bit `0x2`, wrapping at 0.5 s; the red dot draws at `scale×668, scale×559` only while this is `< 0.25`, i.e. a 50%-duty blink. Reset when fuel `< 0.02` **and** RPM `< 0.01` (engine off) | high |
| `+0x9204` | **car-damage bar value** — abs `0x00661534` = `g_hud_car_damage_bar_value`; the bar's FILL, written as `1.0 - car_get_condition_fraction()` by the player-HUD update (3 sites) and by `racing_tick`, and zeroed by the racing HUD setup in the same statement pair that raises bit `0x10000` (`cardamagevisible`). | high |
| `+0x9208` / `+0x9214` | **player-health** number + its `"%d"` text (setter `hud_set_player_health_value`); the number is the always-on indicator value drawn by `hud_build_indicator_value_digits` | high |
| `+0x920c` / `+0x9210` / `+0x9224` | **weapon-ammo** mag / reserve + the `"%d/%d"` text (setter `hud_set_player_ammo_mag_reserve`); rendered by `hud_build_ammo_mag_reserve_quads` | high |
| `+0x9234` | **crosshair spread/bloom** — scale/param for the bit-`0x20` (aim-indicator) element; setter `hud_set_player_accuracy`, driven by `player_add_shot_accuracy_penalty` with `intensity*3 + 1` on every shot | high |
| `+0x9244 + slot*4` (4 entries) | **wingman portrait table** (slot 0..3) — portrait_id 0..7 (setter masks `& 7`); setter `hud_wingman_set_portrait_for_slot` (§7) | high |
| `+0x9254 + slot*4` (4 entries) | **wingman indicator-value table** (slot 0..3) — the wingman's health, rendered next to the portrait; init loop sets all 4 to 100 (default); setter `hud_wingman_set_indicator_value_for_slot` (§7) | high |
| `+0x9264`, `+0x9268` | position/geometry for the bit-`0x20` (aim-indicator/crosshair) element | med |
| `+0x9270` | compass target frame pointer | high |
| `+0x9274` (5×0x40) / `+0x93b4` (5×4) / `+0x93c8` (5×4) / `+0x93dc` / `+0x93e0` | console message ring: strings / colors / TTL / head / count | high |
| `+0x93e8` / `+0x93ec` / `+0x93f0` | **SUBTITLE** line vector (begin/end/cap) — element type `G_MISubtitleStruct`, stride 0x10, `+0xc` = per-line TTL; bit `0x40`; setters `hud_subtitles_add` / `hud_subtitles_add_from_record`, clear `hud_subtitles_clear` | high |
| `+0x93f8` / `+0x93fc` / `+0x9400` | **MISSION-HELP** line vector (begin/end/cap) — same `G_MISubtitleStruct` record type as the subtitle list; bit `0x1000000`; setter `hud_mission_help_add`, clear `hud_mission_help_clear`, retime `hud_mission_help_set_time` | high |
| `+0x9408` / `+0x940c` / `+0x9410` | **intro-subtitle display line vector** (stride 0x1c); writers `hud_intro_subtitles_add_intern` / `_add_centered` | high |
| `+0x9418` / `+0x941c` / `+0x9420` | **intro-subtitle waiting queue** — `vector<G_MIIntroSubtitlesWaiting{uint textId; int waitMs}>`, stride 8; writer `hud_intro_subtitles_queue_id_range` | high |
| `+0x9424` (10×) / `+0x944c` | **screen blood splatter** ring (slots / count), gated on bit `0x2000000`; written by `hud_push_blood_splat`, bound and drawn by `hud_render_all` with `g_hud_blood_overlay_texture` = `2blood.bmp` | high |
| `+0x9450` | **timed-action / theft progress-bar fill** `[0,1]`; bit `0x1000`; setter `hud_set_theft_indicator_value` | high |
| `+0x9454` | bit-`0x40000` **airplane-damage gauge** fill/value; setter `hud_set_airplane_damage_fill` (sole caller `airplane_hud_damage_toggle`); driver `airplaneshowdamage` (case 0x14f) | high |
| `+0x9458` / `+0x945c` / `+0x9460` / `+0x9464` | **cinematic letterbox bar** state (duration ×2, animated diffuse, direction); bit `0x4000`; setter `hud_widescreen_fade_bars` (geometry → vtx buffer `0x006570c0`) | high |
| `+0x9468..+0x9474` | full-screen fade overlay state (4 dwords); bit `0x8000`; setter `hud_fade_set` | high |
| `+0x947c` / `+0x9480` | **full-screen tint** — float alpha (0..255) and RGB; setter `hud_set_screen_tint`; `hud_render_all` draws a viewport-covering quad coloured `alpha << 24 \| rgb` while the alpha is above 0, combined with the overlay texture below (`structs/hud.md` §2) | high |
| `+0x9484` / `+0x9488` / `+0x948c` | full-screen overlay: `overlay_texture_name` (the file name, compared to skip a reload) / `overlay_texture` (the loaded texture) / `overlay_unmodulated` (set: the texture drawn in white with the tint quad over it; clear: one quad, the texture modulated by the tint); setter `hud_set_overlay_texture` (PC-only), detail in `structs/hud.md` §2 | high |
| `+0x9494/+0x9498/+0x949c` | **radar CAR list** — `vector<{C_car*, rgba}>`, stride 8 (abs begin `0x006617c4`). A MEMBERSHIP list, not a colour override: every spawned car is pushed here with its category colour and removing the entry deletes the blip (§6) | high |
| `+0x94a0` | the player's `C_car*` — the car the radar centres on; setter `hud_radar_set_player_car` | high |
| `+0x94a8` / `+0x94ac` / `+0x94b0` | **second radar blip list** — `vector<{tDynamicCollObject*, rgba}>`, stride 8; setters `hud_radar_add_object_with_color` / `hud_radar_remove_object` | high |
| `+0x94b8` | radar blip scale about the player-car centre (`0.9f`) | high |
| `+0x94bc`..`+0x94c8` | **racing-mission place indicator** (cur/prev value + cur/prev alpha); bit `0x200000`; setter `hud_set_race_start_flag` | high |
| `+0x94cc` / `+0x950c` / `+0x954c` / `+0x958c` / `+0x95cc` | **racing-mission** text buffers: Time / Lap Time / Best / Pos / Lap (setters `hud_set_race_*`) | high |
| `+0x960c` (0x80) / `+0x968b` / `+0x968c` | **racing-mission banner** text + flags; bit `0x400000`; setter `hud_set_race_banner_text` | high |
| `+0x9690` / `+0x9694` | **Free Ride** money/score value + `"$%d"` text (setter `hud_freeride_update_score_text`) — Free Ride, NOT the racing mission | high |
| `+0x96d4` | per-frame millisecond accumulator, `+= dt` at the top of `hud_tick` | high |
| `+0x96d8` | police symbol selector (1..4) | high |
| `+0x96dc` | police symbol packed RGBA `(alpha<<24)\|0xffffff` | high |
| `+0x96e0` | police wanted-bar fill `[0,1]` (= `g_police_wanted_bar`, abs `0x00661a10`) | high |
| `+0x96e4` | **map-enabled bool** — the gate on the bit-`0x2000` map overlay; setter `hud_map_enable` | high |
| `+0x96e8` | cached textdb `0x5f` ("Out of City Limits") caption for the map overlay; written by `hud_map_enable` | high |

---

## 6. The in-car radar — what maps, what doesn't

The radar is the top-left minimap shown **while the player is in a car**. Its visibility bit is
`0x00080000` (§4), never set alone — only via `| 0x80002` / `| 0x180006` / `| 0x190006`.

**The blip layer.**

- `+0x9494/+0x9498/+0x949c` is a stride-8 `vector<{C_car*, rgba}>` and it is the radar's car
  **MEMBERSHIP list, not a colour-override list**. `hud_radar_add_car_with_color` adds a car (or
  recolours it if already listed); `hud_radar_remove_car` **deletes the blip** — there is no default
  colour to revert to. The evidence is the caller side: the traffic/actor spawn path classifies **every**
  spawned car and pushes it here unconditionally, parked cars are added explicitly, and
  despawn/teardown paths call the remove sibling.
- **A SECOND blip list exists at `+0x94a8`** — same stride-8 shape, holding `tDynamicCollObject*`
  entries (props / pickups / mission objects) via `hud_radar_add_object_with_color` /
  `hud_radar_remove_object`. `hud_radar_clear` empties
  both.
- **Trams are on the OBJECT list, which is why they have their own colour.** `railway_update` (VA
  `0x004da400`) registers the tram engine **and each wagon separately** via
  `hud_radar_add_object_with_color(part, 0xffffff00)`. A tram is a `tDynamicCollObject`, not a `C_car`,
  so it never enters the vehicle list.

Blip-colour palette — these are the categories that get added to the list, which is why every ordinary
car has one `[user-verified]`:

| Colour | RGBA | Category |
|---|---|---|
| white | `0xffffffff` | the player's own car |
| pale green | `0xffb0e0b0` | ordinary traffic |
| orange | `0xffff9f00` | gangster (Free Ride) |
| light blue | `0xff009fff` | police, not chasing |
| red | `0xffff0000` | chasing the player |
| light gray | `0xffc0c0c0` | parked car |
| yellow | `0xffffff00` | tram (object list) |

**The renderer and the geometry.**

- **The blip renderer is `hud_render_all`.** The lists are walked through a register cursor, so only the
  END pointers (`+0x9498`, `+0x94ac`) ever appear as literal offsets — which is why an offset search
  finds no reader of the begin pointers.
- **Blip geometry:** each car blip is a rectangle built from `vehicle_get_pivot_pos()` +
  **`vehicle_get_size()`**, so blip size scales with the vehicle `[user-verified]`. Corners are mapped
  into radar space by the `+0x94b8` scale (0.9f) about the player-car centre and emitted as engine
  primitive type **5** (outline) in the entry's stored colour.
- **The red destination cross** is drawn from `+0x9270` (the compass destination) as engine primitive
  type **1** (line list, two crossing segments) in `0xffff0000`, arm length `+0x90c0 * 5.0`. It is
  double-gated: inside the `0x80000` block *and* on compass bit `0x80` with a non-null destination —
  which is exactly why it always marks where the compass points.

**What is NOT on it.**

- **There is no road outline in normal play** `[user-verified]`. The PC radar draws blips over a
  **partially transparent background sprite** — `radarclp.bmp`, loaded by `hud_create` into
  `g_radar_clip_texture` (abs `0x00658328`) and bound by `hud_render_all` (texture slot `+0xc`,
  then a 2-triangle strip). No minimap, no city road edges. What is on it: vehicles as
  coloured rectangles, trams in yellow, and the red destination cross.
- **The road-edge strips are RACE-TRACK edges.** `hud_radar_set_left_edge` /
  `hud_radar_set_right_edge` take a `std::vector<C_web_node*>` and build two TLVERTEX strips into
  `g_radar_left_edge_vtx`/`g_radar_right_edge_vtx` (parallel point arrays, shared count
  `g_radar_left_edge_count`) — but **`racing_activate_racers` (VA `0x00611430`) is the only callsite
  that ever passes a real node run**, from the current circuit's left and right paths; every other call
  passes `0` (the teardown in `hud_game_done`). That is why the outline is visible only during the
  racing mission.
- **Neither `0x20` nor `0x8000000` is a radar bit** — they are the aim crosshair and the weapon-ammo
  readout (§4). `hud_set_player_accuracy` drives `+0x9234` from `player_add_shot_accuracy_penalty` with
  `intensity*3 + 1` on every shot, i.e. `+0x9234` is the crosshair spread, which widens as you fire.

---

## 6b. The in-car instrument cluster (speedometer panel)

The cluster is the dashboard readout shown while driving. It is one HUD element as far as visibility
goes — **flag-word bit `0x00000002`** — but it draws eight distinct panels, all fed from one producer.

### 6b.1 The panels

| Panel | Value field | Geometry | Notes |
|---|---|---|---|
| **Speed needle** | `+0x916c` (km/h) | def `g_speedo_needle_def_speed` → `g_speedo_needle_vtx_speed` | The large needle. Value is `m/s × 3.6`. |
| **RPM needle** (tachometer) | `+0x9190` | def `g_speedo_needle_def_rpm` → `g_speedo_needle_vtx_rpm` | Smaller needle built into the dial. Normalised `[0,1]` — the producer divides live RPM by the car's maximum. |
| **Fuel needle** | `+0x91b4` | def `g_speedo_needle_def_fuel` → `g_speedo_needle_vtx_fuel` | Smallest needle. Normalised `[0,1]` = tank level ÷ capacity. |
| **Low-fuel warning** | `+0x91fd` flag, `+0x9200` phase | drawn at `scale×668, scale×559`, 16 px | Red dot next to the fuel gauge. Raised when the fuel value drops **below 0.02**; blinks at 50 % duty (6b.3). |
| **Dial faces** | - | `g_speedo_dial_verts`, two quads on the `+0x9084` sheet: speedometer 180×180 at `+0x91ec`/`+0x91f0`, tachometer 76×76 at `+0x91f4`/`+0x91f8` | The speed and fuel needles pivot on the speedometer face's origin, the RPM needle on the tachometer's. Built by `hud_rebuild_layout_for_resolution`. |
| **Gear panel** | `+0x91e0` | `g_speedo_gear_vtx`, one quad (alpha `0x70`) on the `1int.tga` sheet (`+0x9080`) | Stored pre-scaled as `gear × 11.0` — 11 is the atlas cell pitch, the same one the indicator digits use. The quad is a window onto the digit strip that slides toward the new gear (6b.3). |
| **Odometer** | `g_speedo_odometer_digit_verts` | 5 digits, 0xc0 bytes (6 vertices) each | Total distance. Digits are emitted least-significant-first; each glyph's atlas V is `130.0 − (d0×1.3 + d1×13.0)`. |
| **Transmission indicator** | `+0x91fc` | `g_transmission_indicator_texture`, 32×32 at `scale×758, scale×558` | The **A** / **M** character. The flag picks one of two UV rects on the same sprite sheet. |

### 6b.2 Where the values come from

One producer drives the whole cluster, per frame, from the player's current vehicle (`player+0x98`):

| HUD field | Source | Meaning |
|---|---|---|
| `+0x916c` speed | `car+0x688` | the car's speed readout |
| `+0x9190` RPM | `car_get_engine_rpm(car) ÷ (car+0x1f88 × 60)` | live RPM over maximum. The helper returns `car+0x5c4 × 60` when the engine-running flag `car+0xca8` is set, else 0 — so the needle drops to zero with the engine off. |
| `+0x91b4` fuel | `car+0xd1c ÷ car+0x1f8c` | tank level ÷ tank capacity |
| `+0x91e0` gear | `car+0x648` | the gearbox gear index (`-1` = reverse) |
| `+0x91fc` transmission | `car+0x5b8` | the per-car transmission-mode flag that selects A vs M |

`hud_reset_values` clears the cluster by calling `hud_speedometer_set_gear(0, 0)` and
`hud_speedometer_set_values(0, 0, 0, 0)`.

### 6b.3 Update and draw

- **Value push** — `hud_speedometer_set_values` stores the three needle values, but each needle's
  geometry is **change-gated**: `hud_build_needle_quad` only re-runs for a needle whose
  value actually differs from the cached one, so a steady gauge costs nothing per frame.
- **Needle maths** — a needle's angle is `def.start + clamp((value − def.min) ÷ def.range, 0, 1) ×
  def.sweep`; its four local corner offsets are rotated by that angle and scaled by `+0x91e4` /
  `+0x91e8` about a pivot.
- **Blink** — `hud_tick` advances `+0x9200` by the frame delta whenever bit `0x2` is set **and** the
  low-fuel flag `+0x91fd` is raised, wrapping at 0.5 s. `hud_render_all` draws the red dot only while
  the phase is `< 0.25`, giving an even on/off blink. The phase is reset to 0 when fuel is `< 0.02`
  **and** RPM is `< 0.01`, i.e. when the car is stopped with the engine off.
- **Gear slide** — `hud_tick` moves the displayed gear `+0x91dc` toward `+0x91e0` by 0.02 per ms
  and re-derives the gear quad's texture coordinates from it, so a gear change scrolls the digit
  strip instead of jumping.
- **Relayout** — `hud_rebuild_layout_for_resolution` (VA `0x005532e0`) re-emits all three needles (and
  the surrounding panel quads) after a screen-size change, because their vertex buffers hold absolute
  screen coordinates rather than being scaled at draw time.
- **Draw** — `hud_render_all` binds the dial sheet `+0x9084` and draws one triangle list starting at
  `g_speedo_dial_verts`: 14 triangles = the 12 dial-face vertices plus the 30 odometer vertices
  (`g_speedo_odometer_digit_verts`), or 16 with the 6 speed-limiter-icon vertices when bit `0x100`
  is set, which the three buffers' contiguous addresses make possible. The gear panel follows as a
  separate 2-triangle draw from `+0x9080`.
- **Nesting** — the speed-limiter banner (bit `0x100`) is tested **inside** the `& 2` branch, which is
  why that banner only ever appears while the player is in a car.

### 6b.4 Textures

`hud_load_speedometer_odometer_texture` loads the dial sheet into `+0x9084`, choosing among four
variants: `2int.tga` / `2intmph.tga` by unit system (`g_speed_unit_is_mph`) and `2intnoc.tga` /
`2intnocmph.tga` at night (`C_game+0x2d44`). The A/M overlay `2intauto.tga` / `2intauton.tga` goes to
`g_transmission_indicator_texture` in the same call.

---

## 7. The wingman-indicator system

The 4 high HUD flag-word bits `0x10000000`..`0x80000000`, the field tables at `+0x9244`/`+0x9254` and the player-struct fields at `+0x6d8`/`+0x6dc` form a single **4-slot wingman-indicator subsystem** driven by two script commands: `wingman_setindicator` (case 0x79) and `wingman_delindicator` (case 0x7a).

**Per-slot resources (slot ∈ 0..3):**

| Slot | Flag-word bit | Portrait field | Value field |
|:---:|:---:|:---:|:---:|
| 0 | `0x10000000` | `hud_state+0x9244` | `hud_state+0x9254` |
| 1 | `0x20000000` | `hud_state+0x9248` | `hud_state+0x9258` |
| 2 | `0x40000000` | `hud_state+0x924c` | `hud_state+0x925c` |
| 3 | `0x80000000` | `hud_state+0x9250` | `hud_state+0x9260` |

Each slot displays a **portrait** (face icon, `portrait_id` 0..7 masked `& 7` by `hud_wingman_set_portrait_for_slot`; scripts use 1..5) paired with the wingman's **health** (§8). The engine-side init loop resets all 4 slots to `(portrait=0, value=100)` on startup / scene-reset.

**The persistent pin (on the PLAYER struct, not `hud_state`):**

| Player offset | Meaning |
|---|---|
| `player+0x6d8` | pinned wingman slot index (0..3; `0xffffffff` = -1 = "no pin, indicator system idle") |
| `player+0x6dc` | pinned portrait_id |
| `player+0x5e` | `C_actor+0x5e`, the once-only death latch (`actor_is_death_processed`, `structs/actor.md` §1); the reapply pipeline branches on it |
| `player+0x5d` | a second byte read on the same path; its meaning is open (§10) |

The `(slot, portrait)` pair is the **source of truth** — it is recorded by `player_set_wingman_indicator` (called from cases 0x79 and 0x7a), serialized into the `.sav` record at record-offsets `+0x169`/`+0x16d` (so wingman-indicator state survives save/load), and re-applied to the HUD singleton's slot tables **every frame** by the per-frame player tick (three reapply sites covering on-foot / in-vehicle / cutscene contexts). The HUD singleton's portrait+value tables are therefore **frame-volatile caches**, not authoritative state.

**Per-frame reapply pipeline** (when `player+0x6d8` is non-sentinel):
1. `hud_wingman_set_portrait_for_slot(player+0x6d8, player+0x6dc)` — push portrait
2. `switch(player+0x6d8) { 0..3 → DAT_006613d4 |= per_slot_bit; }` — light the indicator
3. compute indicator value: if `player+0x5e` set → use script-supplied FPU-derived scalar (`__ftol` of TOS); else → an engine-live value through `stl_max_int_ptr` (its source is open, §10)
4. `hud_wingman_set_indicator_value_for_slot(player+0x6d8, value)` — push value

**Script command semantics** (full case bodies: `script-mapping/artifacts/opcodes-verified.md`):

- `wingman_setindicator <npc_id>, <portrait_id>, <slot>` — pins the indicator. `npc_id` is validated against the NPC table (`param_1[0x17]`) but is NOT carried into the recorded pin — only the bounds check proves the wingman actor exists. **npc_id 0 = the player** (user-asserted; the Mafia Script Data Reference maps id 0 to Paulie for this opcode, which does not hold). Lights one of the per-slot bits via `hud_show_element(bit)`.
- `wingman_delindicator <npc_id>, <slot>` — clears the pin and hides one per-slot bit. **Quirk:** the player-struct pin clear is GLOBAL (`(slot=-1, portrait=0)`), not per-slot — calling `wingman_delindicator` for one slot wipes the pinned state for all slots; other slots' HUD bits stay set until the next frame stops reapplying them, then go stale.

---

## 8. PC vs PS2 — the HUD is not the same product

The two builds share the class, the flag word and most of the field layout, but several elements were re-implemented rather than ported. Recording the differences matters because a PS2 body-diff is the main tool for this class, and these are exactly the places where a diff is *supposed* to fail.

| Element | PC | PS2 | Where the difference lives |
|---|---|---|---|
| **Map** | **not scrollable** `[user-verified]` | scrollable — PS2 has `MapGetScrollPos__17G_IndicatorsClassFRfRff` and a `DrawMap(f,f,f)` taking scroll/zoom arguments | PS2-only methods. There is no PC twin for `MapGetScrollPos`, which is consistent with the PC map having no scroll state to query. |
| **Radar** | blips over a **partially transparent background sprite** (`radarclp.bmp` → `g_radar_clip_texture`): coloured vehicle rectangles sized by vehicle, yellow trams, a red cross at the compass destination. **No minimap and no road outline** — the road-edge strips are fed only by `racing_activate_racers`, i.e. only on the racetrack `[user-verified]` | works as a **moving minimap** — the background is the map image itself, scrolling under the player | The draw side. Both builds share the edge builders, the blip lists and the background-texture load; only the renderer differs. |
| **Subtitles** | fully implemented (`hud_subtitles_add`, `hud_subtitles_add_from_record`) | **cut** — `SubtitlesAdd__17G_IndicatorsClass` is an empty stub | PS2 stub. This is why that one twin is graded `[STRUCTURAL]`, not `[CONFIRMED]`. |
| **Intro subtitles, "simple" variant** | builds a centred line and pushes it (`hud_intro_subtitles_add_centered`) | rewritten as a **hot-rod bonus-car unlock check** (`_hotRodTable`, `RaceCarsGetEnabled`, `IsEnabledForRace`) | PS2-port-only content reusing the method slot. The twin is deliberately **not** applied. |
| **Console ring** | 5 slots, TTL 5.0 s | 3 slots, TTL 2.0 s, **plus** a same-id repeat suppressor (`+0x4358`) that refreshes an existing line instead of pushing a duplicate | Field that does not exist on PC. `hud_console_print` correspondingly lacks the branch. |
| **Compass** | one destination slot (`+0x9270`) | two (`CompassSetDestination` / `CompassSetDestination2`) | PS2-only second slot; on PC the adjacent `+0x9274` is the console ring. |
| **Inventory screen** | absent from this class | `DrawInventory`, `InventoryScroll`, `_InventorySet` (5664 B) | PS2-only, consistent with the console UI. |
| **Layout resolution** | 800×600 virtual screen | 640×480 | Percent-based layout in `hud_intro_subtitles_add_intern` resolves against different constants; glyph atlases likewise (PC ÷256/÷128, PS2 ÷255/÷127). |
| **Car-damage bar fill** | `1.0 - car_get_condition_fraction()` into `+0x9204` | the same fill (`DAT_004f31f0`), plus a forced `1.0` when `IsCarUsable` fails | The fill's writer. |
| **Vertex format** | TLVERTEX stride `0x20` (8 floats) | stride `0x1c` (7 floats) | Every quad builder differs by this constant and nothing else. |

**Cross-build evidence that settled PC-side questions** (kept here rather than in §4/§5/§6):

| PC question | What the PS2 side showed |
|---|---|
| the singleton's class is `G_IndicatorsClass` | 50+ of the PC methods operating on `0x00658330` body-diff 1:1 against `G_IndicatorsClass` methods, and the PS2 instance at `0x004eeff0` carries the same flag word at `+0x40a4` (PC `+0x90a4`) with the same bit assignments (`0x801` master early-out, `0x8000` fade, `0x40000` airplane gauge, `0x200000`/`0x400000` race, `0x1000000` mission help). |
| bit `0x20` = crosshair, not radar | `player_indicator_set_aimed` is a 1:1 body-diff of `IndicatorSetAimed__8C_playerFv`, which sets bit `0x20` then immediately calls `NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji` — *"set the crosshair type according to the weapon"*. |
| `+0x9450` = the theft/timed-action bar | All four PC writers match all four callers of `ThiefIndicatorSetValue` constant-for-constant. |
| `+0x9254` wingman value = health | The setter is `PlayerSetWingmanLives`. |
| bit `0x40` vs `0x1000000` | `SubtitlesClear`/`SubtitlesAdd` operate on the `+0x93e8` vector and `MissionHelpAdd` on `+0x93f8`, which separates subtitles from mission help. `MissionHelpAdd` also shares the TTL constant `strlen(text) * 0x28 + 0x9c4` byte-for-byte. |
| bit `0x4000` = the letterbox bars | `hud_widescreen_build_bars` is `WideScreenSetRatio__17G_IndicatorsClassFf` statement-for-statement, sharing six magic constants (`801.0`, `601.0`, the ratio multiply, `0xc0000000` = -2.0f, the diffuse `0xaf0000ff`, `+3.0`, `(h-band)*0.5`). |
| `+0x94a8` holds `tDynamicCollObject*` entries | The PS2 add/remove signatures name the element type. |
| `+0x96e4` = map-enabled bool | Byte-exact against `MapEnable`. |
| `+0x9454` = airplane gauge, not "refuelling" | The PS2 symbol `RefullingIndicatorSetValue` is a dead dev name; its only caller is `ShowDamageIndicator__10C_airplaneFb`, the twin of `airplane_hud_damage_toggle`, same bit `0x40000`. |
| the map texture `+0x9090` | Its lazy load is the inlined `NahrajMapu`; the release in `hud_destroy`/`hud_game_done` is the inlined `UvolniMapu`; the bit-`0x2000` draw branch is the inlined `DrawMap`. |
| the race place indicator (bit `0x200000`) | PS2 calls it `RaceSetStartFlag` — which conflicts with the PC hide-gate's ≤4 place domain. Mapping certain, meaning open (§10). |
| `hud_text_draw_line` (`0x005551a0`) | `OutText__17G_IndicatorsClass` — position, size and body all agree. |
| `hud_set_screen_tint` (`0x0054c530`) | No PS2 counterpart in the class at all. |
| `g_speedo_dial_verts` = the dial faces | PS2 names the buffer `_VtxTachometrCiselnik` ("tachometer dial face"); `ResolutionChanged` stores `hud+0x41f0` into its first vertex, the band-rebased twin of the PC's first store from `+0x91ec`. The gear-panel quad has no PS2 symbol. |

**The class head is complete:** `hud_ctor`, `hud_dtor`, `hud_reset_values`, `hud_load_font`,
`hud_language_select`, `hud_create`, `hud_destroy`. `GameInit` is **INLINED into `game_init`** (anchor:
the `s_GameInitStart_00650f64` literal), and `NahrajMapu` / `UvolniMapu` / `DrawMap` are inlined into
`hud_tick`, `hud_destroy`+`hud_game_done` and `hud_render_all`'s bit-`0x2000` branch respectively.
`PleaseWait` is inlined into `game_done`; `LoadFontFake`, `DrawMessage`, `AByloTicho`,
`TutorialAddText`, `TextSizeMultiLine` and `ParheliaSetFov` are PS2-only — the evidence per method
is in `structs/hud.md` §4b.

⚠️ **Consequence for future work:** when a `G_IndicatorsClass` body-diff fails, check this table
*before* concluding "inlined" — a PS2-port-only element and an inlined one look identical from the PC
side.

---

## 9. Cross-references

- `structs/hud.md` — the per-method PC↔PS2 table for `G_IndicatorsClass` and the field-offset correspondence.
- `police-system/artifacts/police-system.md` — radar colours + police HUD states.
- `Game-ghidra.exe.functions.md` — the HUD function cluster + the STL primitives behind the console/radar lists.
- `script-mapping/artifacts/opcodes-verified.md` — full case bodies for `wingman_setindicator` (0x79) / `wingman_delindicator` (0x7a) and the other HUD-driving script commands.
- `basic/artifacts/player-interactions.md` — the action menu, which is NOT one of these HUD elements.

---

## 10. Open items

| item | state |
|---|---|
| bit `0x4000` (cinematic letterbox bars) | mechanism and geometry known; not confirmed in game |
| bit `0x200000` (`hud_set_race_start_flag`) | a semantic conflict, not a mapping one: the PC hide-gate (`!= 4 && != -1`) implies a place/position domain of at most four, while the PS2 name is `RaceSetStartFlag`. One in-game observation settles what the widget depicts |
| bits `0x400`, `0x800`, `0x100000` of the flag word | no element identified; `0x800` is half of the renderer's `0x801` early-out mask, and `0x100000` is set by the standard gameplay masks `0x180006` / `0x190006` |
| the wingman-indicator value source | which actor field the engine-live path reads (§7 step 3), and the two arguments to `stl_max_int_ptr` (Ghidra dropped both at the case-0x79 callsite) |
| `player+0x5d` | read on the wingman reapply path; untraced |
| the radar panel backdrop sub-rects | `+0x9084` supplies both the instrument dial sheet and the bit-`0x80000` radar backdrop; the exact sub-rects are not mapped |
| the odometer's units | the dial art follows `g_speed_unit_is_mph`, but nothing has been traced that converts the *distance value* itself |
