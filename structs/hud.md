# `G_IndicatorsClass` — the HUD singleton (`hud_state` @ PC `VA 0x00658330`)

**Created** 2026-07-22 11:22 +0200 · **Modified** 2026-10-04 19:22 +0200

PC prefix **`hud_*`**.

| | |
|---|---|
| PC singleton | `0x00658330` — one hardcoded instance, no dynamic allocation |
| PS2 class | `G_IndicatorsClass` (mangled tag `17G_IndicatorsClass`), instance `0x004eeff0` |
| Base class | none — a standalone manager |
| PC code | **four** clusters, not one (§1b) |
| Method coverage | 84 PS2 methods: **65 CONFIRMED · 1 STRUCTURAL · 1 DIVERGED · 11 PS2-ONLY · 6 INLINED = 84 accounted** · plus 26 PC-only functions (§4) |

It owns *every* 2D screen element: the element-visibility flag word, the stopwatch, the in-car
instrument cluster, player/wingman indicators, weapon-ammo, subtitles, mission help, intro
subtitles, the console ring, the compass, the fade overlay, the radar, the racing readouts, the map
overlay and the text/font engine.

Every CONFIRMED row in §4 was body-diffed on both sides. The companion prose doc — the `+0x90a4` bit
map, the script commands that drive HUD elements, in-game observations — is
`basic/artifacts/hud_elements.md`.

## 1. Two things to know before using this class

### 1a. The field rebase is BAND-WISE, not one constant

Do **not** port a PS2 offset by adding a fixed delta. Measured anchors:

| Band | PC | PS2 | delta |
|---|---|---|---|
| flag word | `+0x90a4` | `+0x40a4` | `+0x5000` |
| stopwatch / instrument cluster | `+0x9150` | `+0x4154` | `+0x4FFC` |
| player health / ammo / accuracy | `+0x9208` | `+0x4204` | `+0x5004` |
| wingman slots | `+0x9244` | `+0x4244` | `+0x5000` |
| compass | `+0x9270` | `+0x4270` | `+0x5000` |
| console ring | `+0x9274` | `+0x4278` | `+0x4FFC` |
| subtitles vector | `+0x93e8` | `+0x4b38` | `+0x48B0` |
| intro subtitles / fade / theft bar | `+0x9408` / `+0x9468` / `+0x9450` | `+0x4b50` / `+0x4bb0` / `+0x4b98` | `+0x48B8` |
| radar | `+0x9494` | `+0x4bd8` | `+0x48BC` |
| race / map | `+0x94bc` / `+0x96e4` | `+0x4bfc` / `+0x4e24` | `+0x48C0` |

The drift starts at the **console ring**, which the PC widened from 3 slots to 5, and grows again
across the subtitle/intro-subtitle containers.

### 1b. The PC code is FOUR clusters

A class's PC code is not one contiguous cluster, and this one is the clearest example.

| PC range | Contents |
|---|---|
| `0x00548420`–`0x0054d180` | the bulk: lifecycle, timers, instruments, indicators, text, console, radar, race, `hud_render_all` |
| `0x005540b0`–`0x00555200` | `hud_tick` + the text/font engine (`hud_text_measure`, `hud_text_find_line_break`, `hud_text_draw_line`) |
| `0x004538b0` | `airplane_hud_damage_toggle` (a `C_airplane` method that drives this HUD) |
| `0x00424e20` / `0x005c7db0` / `0x005c7dd0` | `hud_show_element` / `hud_hide_element` / `hud_element_is_visible` |

Emission order holds **within** a cluster only. Anything still missing should be hunted with that in
mind before it is called "inlined".

## 2. Field layout

Selected fields; the full per-offset map lives in `basic/artifacts/hud_elements.md` §5.

| Offset | Meaning | Confidence |
|---|---|---|
| `+0x0000`…`+0x5000` | **font table A** — FOUR blocks of `0x1400`, each 256 glyph records of `0x14` bytes with the advance at `rec+0x10`. Read ONLY by `hud_text_draw_line_alt_font`, whose only caller is the draw of the never-built `'edip'` widget (`structs/gui_widget.md` §4). Nothing fills it: the singleton lies in `.bss`, and no other function computes the `0x1400` font index | [H] |
| `+0x5000`…`+0x9070` | **font table B** — FOUR blocks of `0x101c`, each a `0x1c` header + 256 glyph records of `0x10` bytes, `dtaRead` straight from `tablesont%d_%s.gmf` by `hud_load_font`. Read by `hud_text_draw_line` | [H] |
| `+0x5000 + font_index*0x101c`, then `+0x500c` / `+0x5014` / `+0x5018` / `+0x501c` | per-font metric block of table B: scale factors + glyph records (stride `0x10`). The base is `+0x5000`, not 0 — byte-verified from `hud_load_font`'s `lea ecx,[eax + edx*4 + 0x5000]` with `edx = idx*1031`, and from `hud_text_draw_line`'s `[ecx + eax*4 + 0x5018]` | [H] |
| `+0x9070` + `idx*4` | the four font TEXTURE slots (`font%d_%s.tga`), released and reloaded per slot by `hud_load_font` | [H] |
| `+0x9080` | the `1int.tga` interface sheet, loaded by `hud_create`; `hud_draw_mouse_cursor` takes the pointer out of it, and the gear panel (`g_speedo_gear_vtx`) is drawn from it | [H] |
| `+0x9084` | instrument-dial texture (`2int.tga` / `2intmph.tga` / `2intnoc*.tga`), loaded by `hud_load_speedometer_odometer_texture` — which picks the sheet from `g_speed_unit_is_mph` and the mission's night flag | [H] |
| `+0x9090` | **full-map texture** (`0mapar.bmp`) — lazily loaded by `hud_tick`, bound by `hud_render_all` (texture slot `+0xc`, then a 2-triangle strip), released by `hud_destroy` / `hud_game_done` (§4) | [H] |
| `+0x90a4` | **element-visibility flag word** (= flat global `g_hud_flags`, abs `0x006613d4`). Four accessors: `hud_show_element` / `hud_hide_element` set and clear single bits, `hud_element_is_visible` tests one, and `hud_set_element_flags` (`0x005c7da0`) overwrites the whole word — 2 callers, `program_process_command` and `game_init` | [H] |
| `+0x90ac` / `+0x90b0` / `+0x90b4` | HUD-flag save/restore stack (`std::vector<uint>`). `hud_push_element_flags` (`0x005c7df0`) pushes the current `+0x90a4` word, `hud_pop_element_flags` (`0x005c7fb0`) pops it back. Both are out-of-line inline members hosted by the `C_program` translation unit (`structs/program.md` §6), and both have exactly one caller: `program_process_command`. So the stack exists **for the script language** — a mission script hides HUD elements, does something, and restores whatever the player had | [H] |
| `+0x90b8`…`+0x90c4` | screen scale / resolution factors; `+0x90c0` / `+0x90c4` are the instance twins of `g_hud_scale_x` / `g_hud_scale_y` (§4a) | [H] |
| `+0x9150` / `+0x9154` / `+0x9158` | stopwatch **time value** h / m / s (`hud_timer_set_time`, read back by `hud_timer_get_time`) | [H] |
| `+0x915c` / `+0x9160` | derived analog hand **angles** (fraction of a full circle) | [H] |
| `+0x9168` | stopwatch interval, milliseconds | [H] |
| `+0x916c` / `+0x9190` / `+0x91b4` | **the three instrument needles** — speed (km/h), **RPM**, fuel. The RPM needle is **PC-ONLY** (§3) | [H] |
| `+0x91dc` / `+0x91e0` | displayed gear / target gear, pre-scaled `gear * 11.0`; `hud_tick` eases `+0x91dc` toward `+0x91e0` at 0.02 per ms and slides the `g_speedo_gear_vtx` texture window with it | [H] |
| `+0x91e4` / `+0x91e8` | needle x / y scale factors | [H] |
| `+0x91fc` | **A/M transmission character** — UV select on the `2intauto*.tga` overlay. **PC-ONLY rendering** (§3) | [H] |
| `+0x91fd` / `+0x9200` | low-fuel warning flag + its 0.5 s blink phase (red dot draws while phase `< 0.25`) | [H] |
| `+0x9208` / `+0x9214` | player health number + its `"%d"` text | [H] |
| `+0x920c` / `+0x9210` / `+0x9224` | ammo mag / reserve + the `"%d/%d"` text | [H] |
| `+0x9234` | crosshair spread (bit `0x20`) | [H] |
| `+0x9244 + slot*4` (×4) | wingman icon-row id (`& 7`) | [H] |
| `+0x9254 + slot*4` (×4) | wingman **health** | [H] |
| `+0x9270` | compass destination frame | [H] |
| `+0x9274` (5×0x40) / `+0x93b4` / `+0x93c8` / `+0x93dc` / `+0x93e0` | console ring: strings / colours / TTL / head / count | [H] |
| `+0x93e8` / `+0x93ec` / `+0x93f0` | **subtitle** line vector — `vector<G_MISubtitleStruct>`, stride `0x10`, `+0xc` = TTL | [H] |
| `+0x93f8` / `+0x93fc` / `+0x9400` | **mission-help** line vector — same record type | [H] |
| `+0x9408` / `+0x940c` / `+0x9410` | intro-subtitle display lines, stride `0x1c` | [H] |
| `+0x9418` / `+0x941c` / `+0x9420` | intro-subtitle waiting queue — `vector<G_MIIntroSubtitlesWaiting{uint id; int waitMs}>` | [H] |
| `+0x9450` | **theft / timed-action progress-bar fill** (bit `0x1000`) | [H] |
| `+0x9454` | airplane-damage gauge fill (bit `0x40000`) | [H] |
| `+0x9458` / `+0x945c` / `+0x9460` / `+0x9464` | **cinematic letterbox** transition (bit `0x4000`): duration remaining / total in ms, the animated diffuse, and the direction. Written by `hud_widescreen_fade_bars`; the bars themselves live in the file-scope `g_hud_widescreen_bar_verts` (twelve TLVERTEXes = two 6-vertex quads), built by `hud_widescreen_build_bars` | [H] |
| `+0x9468`…`+0x9474` | fade-overlay state (bit `0x8000`) | [H] |
| `+0x947c` / `+0x9480` | **full-screen tint** — float alpha (0..255) and RGB. `hud_render_all` draws a viewport-covering quad coloured `__ftol(+0x947c) << 24 \| +0x9480` whenever the alpha is > 0. Set only by `hud_set_screen_tint`, from the PC-only `.tck` replay recorder | [H] |
| `+0x9484` / `+0x9488` / `+0x948c` | full-screen overlay (**PC-only**): `overlay_texture_name` — the file name, compared to skip a reload unless forced; `overlay_texture` — the loaded texture, released on change; `overlay_unmodulated` — set: `hud_render_all` draws the texture in white and the tint quad over it, clear: one quad, the texture modulated by the tint colour. Written only by `hud_set_overlay_texture`, which the `.tck` replay recorder drives | [H] |
| `+0x9494` / `+0x9498` / `+0x949c` | **radar CAR list** — `vector<{C_car*, rgba}>`, stride 8 | [H] |
| `+0x94a0` | the player's `C_car*` (what the radar centres on) | [H] |
| `+0x94a8` / `+0x94ac` / `+0x94b0` | **radar OBJECT list** — `vector<{tDynamicCollObject*, rgba}>`, stride 8 | [H] |
| `+0x94bc`…`+0x94c8` | race place indicator: cur/prev value + cur/prev alpha (bit `0x200000`) | [H] |
| `+0x94cc` / `+0x950c` / `+0x954c` / `+0x958c` / `+0x95cc` | race text buffers: Time / Lap Time / Best / Pos / Lap | [H] |
| `+0x960c` (0x80) / `+0x968b` / `+0x968c` | race banner text + flags (bit `0x400000`) | [H] |
| `+0x9690` / `+0x9694` | Free Ride money value + its `"$%d"` text | [H] |
| `+0x96d4` | per-frame millisecond accumulator (`hud_tick`) | [H] |
| `+0x96d8` / `+0x96dc` | police status icon selector + packed alpha | [H] |
| `+0x96e0` | police wanted bar fill (= `g_police_wanted_bar`) | [H] |
| `+0x96e4` / `+0x96e8` | **map-enabled bool** + cached "Out of City Limits" caption | [H] |

Related file-scope globals: `g_font_translation_tables` = the per-font character translation
tables; `g_radar_left_edge_vtx` / `g_radar_left_edge_points` /
`g_radar_left_edge_count` = radar **left** road-edge strip / points / count;
`g_radar_right_edge_vtx` / `g_radar_right_edge_points` = the **right** mirror;
`g_transmission_indicator_texture` = the auto-transmission dial overlay (`2intauto*.tga`), loaded
alongside `+0x9084`; `g_speed_unit_is_mph` = the mph/kph unit-system flag;
`g_speedo_dial_verts` = the speedometer + tachometer dial faces
(12 TLVERTEXes, the base of the dial-sheet draw); `g_speedo_gear_vtx` = the gear-panel quad;
`g_radar_clip_texture` = `radarclp.bmp` at abs `0x00658328`, bound by `hud_render_all` exactly like `+0x9090`.

## 3. The PC and PS2 HUDs are not the same product

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
  draws the destination cross in `0xffff0000` from `+0x9270` (§6).
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
- **File-scope globals (§2):** `g_font_translation_tables` = PS2 `fontTranstable`;
  `g_speedo_dial_verts` = PS2 `_VtxTachometrCiselnik`.

## 4. `G_IndicatorsClass` method family — PC ↔ PS2 (84 PS2 methods + 26 PC-only)

**PC-first. One table, both directions.** A PS2-only method leaves the PC columns `-`; a PC-only
function (class prefix `hud_*`, no identified twin) leaves the PS2 columns `-` and is tagged
**PC-ONLY**, with the reason in its Role cell or in a note under the table. The PS2-keyed half is one row per PS2 symbol — exactly
84 rows, zero duplicates, every PC VA resolved — and is ordered by **PC VA**, PC being primary. Rows
with no PC VA (PS2-only or not-located methods) trail the PC-ordered block, keeping PS2 emission
order among themselves.

Status vocabulary: **CONFIRMED** (body-diffed both sides) · **STRUCTURAL** (one-sided evidence only —
see the row's note) · **DIVERGED** (the slot matches but the bodies are different implementations;
twin deliberately not applied) · **PS2-ONLY** · **INLINED** · **PC-ONLY** · **not located**.

The HUD's OWN console — `hud_console_clear` / `_push_message` / `_print`, the
`ConsoleAddText__17G_IndicatorsClass` family — belongs here. It is unrelated to `C_console`, a
different class in a different translation unit; see `structs/console.md`.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00424e20 | `hud_show_element` | - | - | PC-ONLY | set one `+0x90a4` bit ¹ |
| 0x00424e40 | `hud_set_police_symbol_and_alpha` | - | - | PC-ONLY | set the police status icon and its alpha ² |
| 0x00548ac0 | `hud_static_init` | - | - | PC-ONLY | CRT static-init entry for the singleton. |
| 0x00548ad0 | `hud_static_ctor` | - | - | PC-ONLY | CRT static constructor thunk. |
| 0x00548ae0 | `hud_register_atexit_dtor` | - | - | PC-ONLY | CRT atexit registration for the singleton destructor. |
| 0x00548af0 | `hud_static_dtor` | - | - | PC-ONLY | CRT static destructor thunk. |
| 0x00548b00 | `hud_dtor` | `__dt__17G_IndicatorsClassFv` | 0x002b60c0 | CONFIRMED | frees the seven vectors in exact REVERSE order |
| 0x00548bf0 | `hud_ctor` | `__ct__17G_IndicatorsClassFv` | 0x0029d380 | CONFIRMED | seven vector members, same order — the fingerprint |
| 0x00548d50 | `hud_reset_values` | `ResetValues__17G_IndicatorsClassFv` | 0x0029d5c0 | CONFIRMED | all elements to defaults; pins +0x9450=0.5f, +0x94b8=0.9f |
| 0x00548f90 | `hud_load_font` | `LoadFont__17G_IndicatorsClassFbUi` | 0x0029d7f0 | CONFIRMED | exact arity (bool,uint); PC 4 font slots vs PS2 2; names the two language globals |
| 0x00549060 | `hud_language_select` | `LanguageSelect__17G_IndicatorsClassFUi` | 0x0029da10 | CONFIRMED | builds/loads the `tables/lang_<xx>.def` record |
| 0x00549120 | `hud_create` | `Create__17G_IndicatorsClassFv` | 0x0029daf0 | CONFIRMED | one-time texture load; fills `g_radar_clip_texture` (radarclp.bmp) |
| 0x005492f0 | `hud_destroy` | `Destroy__17G_IndicatorsClassFv` | 0x0029de60 | CONFIRMED | releases all font + element textures |
| 0x005494b0 | `hud_load_speedometer_odometer_texture` | - | - | PC-ONLY | per-scene dial-texture load; PS2 loads its dials inside GameInit/Create |
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
| 0x0054aa30 | `hud_set_player_accuracy` | `PlayerSetAccuracy__17G_IndicatorsClassFf` | 0x002a14e0 | CONFIRMED | crosshair spread (+0x9234) |
| 0x0054aa40 | `hud_wingman_set_portrait_for_slot` | `PlayerSetWingmanName__17G_IndicatorsClassFUiUi` | 0x002a14f0 | CONFIRMED | wingman icon row; `slot<4`, `& 7` |
| 0x0054aa60 | `hud_wingman_set_indicator_value_for_slot` | `PlayerSetWingmanLives__17G_IndicatorsClassFUiUi` | 0x002a1520 | CONFIRMED | wingman HEALTH |
| 0x0054aa80 | `hud_subtitles_clear` | `SubtitlesClear__17G_IndicatorsClassFv` | 0x002a3260 | CONFIRMED | clear the subtitle line list => bit 0x40 = SUBTITLES |
| 0x0054aad0 | `hud_subtitles_add` | `SubtitlesAdd__17G_IndicatorsClassFUiUib` | 0x002a3270 | STRUCTURAL | **PS2 body is an EMPTY STUB** — subtitles cut on PS2; mapping is one-sided |
| 0x0054ae30 | `hud_subtitles_add_from_record` | - | - | PC-ONLY | overload; subtitles were cut on PS2 (`SubtitlesAdd` is an empty stub), so no PS2 overload exists |
| 0x0054b160 | `hud_intro_subtitles_add_intern` | `IntroSubtitlesAddIntern__17G_IndicatorsClassFUif` | 0x002a3280 | CONFIRMED | parse the `~` / `,` layout header; 800x600 vs 640x480 |
| 0x0054b5a0 | `hud_intro_subtitles_queue_id_range` | `IntroSubtitlesAdd__17G_IndicatorsClassFUiUi` | 0x002a35f0 | CONFIRMED | queue `{id, atoi(text)}`; element `G_MIIntroSubtitlesWaiting` |
| 0x0054b7b0 | `hud_intro_subtitles_add_centered` | - | - | PC-ONLY | Slot matches `IntroSubtitlesAddSimple` but the BODIES DISAGREE — PS2 rewrote its version for the PS2-only hot-rod bonus-car unlock. Twin deliberately not applied. |
| 0x0054ba60 | `hud_intro_subtitles_clear` | `IntroSubtitlesClear__17G_IndicatorsClassFv` | 0x002a3740 | CONFIRMED | clears BOTH intro containers |
| 0x0054bad0 | `hud_mission_help_clear` | `MissionHelpClear__17G_IndicatorsClassFv` | 0x002a3770 | CONFIRMED | clear the mission-help line list |
| 0x0054bb20 | `hud_mission_help_add` | `MissionHelpAdd__17G_IndicatorsClassFUi` | 0x002a3780 | CONFIRMED | TTL magic `len*0x28+0x9c4` => bit 0x1000000 = MISSION HELP |
| 0x0054be10 | `hud_mission_help_set_time` | `MissionHelpSetTime__17G_IndicatorsClassFUi` | 0x002a3930 | CONFIRMED | retime every queued line; pins record +0xc = TTL |
| 0x0054be40 | `hud_map_enable` | `MapEnable__17G_IndicatorsClassFb` | 0x002a3c40 | CONFIRMED | byte-exact; +0x96e4 bool + textdb 0x5f cache |
| 0x0054be70 | `hud_console_clear` | `ConsoleClear__17G_IndicatorsClassFv` | 0x002a3c90 | CONFIRMED | zero the ring count |
| 0x0054be80 | `hud_console_push_message` | `ConsoleAddText__17G_IndicatorsClassFPUcUi` | 0x002a3ca0 | CONFIRMED | ring push; PC 5 slots / 5.0 s vs PS2 3 / 2.0 s |
| 0x0054bf10 | `hud_console_print` | `ConsoleAddTextByID__17G_IndicatorsClassFUiUi` | 0x002a3d70 | CONFIRMED | textdb lookup -> push; PS2 also de-dupes |
| 0x0054bf40 | `hud_push_blood_splat` | - | - | PC-ONLY | the screen blood-splat ring (bit `0x2000000`): `human_hit` pushes one randomly rotated splat when the player is hit; `hud_render_all` draws it with `2blood.bmp`. The PS2 has the ring but no push (§4b) |
| 0x0054c150 | `hud_compass_set_destination_frame` | `CompassSetDestination__17G_IndicatorsClassFP9I3D_frame` | 0x002a3e20 | CONFIRMED | +0x9270 target frame |
| 0x0054c160 | `hud_set_theft_indicator_value` | `ThiefIndicatorSetValue__17G_IndicatorsClassFf` | 0x002a3e40 | CONFIRMED | theft / timed-action bar => bit 0x1000 |
| 0x0054c170 | `hud_set_airplane_damage_fill` | `RefullingIndicatorSetValue__17G_IndicatorsClassFf` | 0x002a3e50 | CONFIRMED | airplane gauge fill; PS2 symbol is a dead dev name |
| 0x0054c180 | `hud_widescreen_build_bars` | `WideScreenSetRatio__17G_IndicatorsClassFf` | 0x002a3e60 | CONFIRMED | builds the letterbox bars ⇒ **flag bit 0x4000 is the cinematic letterbox** ³ |
| 0x0054c420 | `hud_widescreen_fade_bars` | - | - | PC-ONLY | fades the letterbox bars in/out over a duration; the PS2 has only `WideScreenSetRatio`. Shaped exactly like `hud_fade_set` |
| 0x0054c490 | `hud_fade_set` | `FadeInOut__17G_IndicatorsClassFbUiUi` | 0x002a4250 | CONFIRMED | statement-for-statement identical |
| 0x0054c500 | `hud_fade_get_state` | `FadeInOutGetState__17G_IndicatorsClassFv` | 0x002a42b0 | CONFIRMED | same predicate |
| 0x0054c520 | `hud_fade_is_active` | `FadeInOutIsDark__17G_IndicatorsClassFv` | 0x002a42e0 | CONFIRMED | bit 0x8000 test |
| 0x0054c530 | `hud_set_screen_tint` | - | - | PC-ONLY | the full-screen tint (+0x947c alpha / +0x9480 RGB) that the `.tck` replay recorder drives ⁴ |
| 0x0054c550 | `hud_set_overlay_texture` | - | - | PC-ONLY | the full-screen overlay texture ⁴ |
| 0x0054c5c0 | `hud_radar_clear` | `RadarClear__17G_IndicatorsClassFv` | 0x002a4300 | CONFIRMED | clears BOTH blip lists |
| 0x0054c620 | `hud_radar_set_player_car` | `RadarSetPlayerCar__17G_IndicatorsClassFP5C_car` | 0x002a4330 | CONFIRMED | +0x94a0; separated from its sibling by band layout |
| 0x0054c630 | `hud_radar_add_car_with_color` | `RadarAddCar__17G_IndicatorsClassFP5C_carUi` | 0x002a4340 | CONFIRMED | membership upsert; PC stores rgba, PS2 a type index + limit |
| 0x0054c840 | `hud_radar_remove_car` | `RadarRemoveCar__17G_IndicatorsClassFP5C_car` | 0x002a4410 | CONFIRMED | find + erase-by-shift |
| 0x0054c8a0 | `hud_radar_add_object_with_color` | `RadarAddObject__17G_IndicatorsClassFP18tDynamicCollObjectUi` | 0x002a4470 | CONFIRMED | the `tDynamicCollObject` list at +0x94a8 |
| 0x0054cab0 | `hud_radar_remove_object` | `RadarRemoveObject__17G_IndicatorsClassFP18tDynamicCollObject` | 0x002a4500 | CONFIRMED | find + erase-by-shift on the object list |
| 0x0054cb10 | `hud_radar_set_left_edge` | `RadarSetLeftEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>` | 0x002a4560 | CONFIRMED | `vector<C_web_node*>` -> TLVERTEX strip (nav web) |
| 0x0054cd10 | `hud_radar_set_right_edge` | `RadarSetRightEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>` | 0x002a4840 | CONFIRMED | mirror global set |
| 0x0054cf10 | `hud_set_race_start_flag` | `RaceSetStartFlag__17G_IndicatorsClassFUi` | 0x002a4b20 | CONFIRMED | bit 0x200000; the race start-light countdown — see §4b |
| 0x0054cfc0 | `hud_set_race_time_text` | `RaceSetTime__17G_IndicatorsClassFUi` | 0x002a4b90 | CONFIRMED | textdb 0x2c8 |
| 0x0054d000 | `hud_set_race_laptime_text` | `RaceSetLapTime__17G_IndicatorsClassFUi` | 0x002a4c20 | CONFIRMED | textdb 0x2d0 |
| 0x0054d040 | `hud_set_race_besttime_text` | `RaceSetBestTime__17G_IndicatorsClassFUi` | 0x002a4cb0 | CONFIRMED | textdb 0x2c9 |
| 0x0054d080 | `hud_set_race_position_text` | `RaceSetPosition__17G_IndicatorsClassFUiUi` | 0x002a4d40 | CONFIRMED | textdb 0x2ca |
| 0x0054d0b0 | `hud_set_race_lap_text` | `RaceSetLaps__17G_IndicatorsClassFUiUi` | 0x002a4da0 | CONFIRMED | textdb 0x2cb |
| 0x0054d0e0 | `hud_set_race_banner_text` | `RaceFlashText__17G_IndicatorsClassFPUcf` | 0x002a4e10 | CONFIRMED | bit 0x400000, strncpy 0x80 |
| 0x0054d130 | `hud_freeride_update_score_text` | `BonusSetCount__17G_IndicatorsClassFi` | 0x002a4e60 | CONFIRMED | textdb 0xb3; 0x7fffffff = hide |
| 0x0054d180 | `hud_render_all` | `DrawAll__17G_IndicatorsClassFv` | 0x002a4ed0 | CONFIRMED | shared `& 0x801` master early-out |
| 0x005532e0 | `hud_rebuild_layout_for_resolution` | `ResolutionChanged__17G_IndicatorsClassFv` | 0x002ad720 | CONFIRMED | the 8736-byte relayout pass ⁵ |
| 0x00553ef0 | `hud_build_rect_quad` | - | - | PC-ONLY | geometry helper for `hud_rebuild_layout_for_resolution` (8 call sites, all in it); free function, no receiver ⁵ |
| 0x00553fd0 | `hud_build_diamond_quad` | - | - | PC-ONLY | the 45-degree rhombus variant of `hud_build_rect_quad`; 1 call site, the speed-limiter icon ⁶ |
| 0x005540b0 | `hud_tick` | `Tick__17G_IndicatorsClassFUi` | 0x002af980 | CONFIRMED | dt accumulate + age every timed field |
| 0x00554fd0 | `hud_text_measure` | `TextSize__17G_IndicatorsClassFPUcfUiPUc` | 0x002b2540 | CONFIRMED | names the global `g_font_translation_tables` = `fontTranstable` |
| 0x00555090 | `hud_text_find_line_break` | `HledejZlom__17G_IndicatorsClassFPUcffUiPUc` | 0x002b2670 | CONFIRMED | the word-wrap primitive |
| 0x005551a0 | `hud_text_draw_line` | `OutText__17G_IndicatorsClassFPUcffffUiUiUiPUc` | 0x002b27d0 | CONFIRMED | the main text primitive, 70 call sites ⁷ |
| 0x00556040 | `hud_text_draw_multiline` | `OutTextMultiLine__17G_IndicatorsClassFPUcffffUiUiUi` | 0x002b3200 | CONFIRMED | one `hud_text_draw_line` per `~`-separated line ⁸ |
| 0x005560b0 | `hud_text_measure_fixed_pitch` | - | - | PC-ONLY | the measure half of the alternate-font pair: `TextSize`'s arity, but NO glyph table — returns `count * size * 0.5` |
| 0x00556100 | `hud_text_draw_line_alt_font` | - | - | PC-ONLY | SECOND line renderer, reading font table A (`this+0x0000`) instead of table B (§2, §6) ⁹ |
| 0x00556d80 | `hud_draw_mouse_cursor` | - | - | PC-ONLY | one 15x15 quad from `1int.tga` (`hud+0x9080`), drawn only for `g_gui_active_screen` after all its children; the PS2 has no mouse. See `structs/gui_screen.md` |
| 0x00560550 | `hud_text_draw_line_scaled` | - | - | PC-ONLY | scales x/width by `hud+0x90c0` and y/height by `hud+0x90c4`, then forwards to `hud_text_draw_line`; sole consumer `credits_render_one` (§4a) |
| 0x005c7da0 | `hud_set_element_flags` | - | - | PC-ONLY | overwrites the whole `+0x90a4` flag word; 2 callers |
| 0x005c7db0 | `hud_hide_element` | - | - | PC-ONLY | clear one `+0x90a4` bit ¹ |
| 0x005c7dd0 | `hud_element_is_visible` | - | - | PC-ONLY | test one `+0x90a4` bit ¹ |
| 0x005c7df0 | `hud_push_element_flags` | - | - | PC-ONLY | pushes `+0x90a4` onto the save/restore stack; sole caller `program_process_command` |
| 0x005c7fb0 | `hud_pop_element_flags` | - | - | PC-ONLY | pops the save/restore stack back into `+0x90a4`; sole caller `program_process_command` |
| 0x005c8010 | `hud_set_police_wanted_bar` | - | - | PC-ONLY | the police wanted bar fill ¹⁰ |
| 0x005c8db0 | `hud_set_car_damage_bar_enabled` | - | - | PC-ONLY | arms the HUD car-damage bar (bit `0x10000`); not a member of this class ¹¹ |
| - | - | `LoadFontFake__17G_IndicatorsClassFUiUi` | 0x0029d9a0 | PS2-ONLY | copies font slot 0 into slots 2 and 3 (§4b) |
| - | - | `NahrajMapu__17G_IndicatorsClassFv` | 0x0029dfe0 | INLINED | **inlined into `hud_tick`** — lazy load of `0mapar.bmp` into +0x9090 ¹² |
| - | - | `UvolniMapu__17G_IndicatorsClassFb` | 0x0029e080 | INLINED | **inlined into `hud_destroy` + `hud_game_done`** — the +0x9090 release. PS2's bool arg (also release the dial +0x4084) is what became the PC split |
| - | - | `GameInit__17G_IndicatorsClassFv` | 0x0029e0e0 | INLINED | **folded into `game_init`** — anchor: the `s_GameInitStart_00650f64` literal sits immediately before the surviving `hud_radar_clear()` + `hud_load_speedometer_odometer_texture()` block |
| - | - | `DrawInventory__17G_IndicatorsClassFv` | 0x002a1640 | PS2-ONLY | console inventory screen |
| - | - | `InventoryScroll__17G_IndicatorsClassFP11G_InventoryUiUii` | 0x002a16f0 | PS2-ONLY | console inventory screen |
| - | - | `_InventorySet__17G_IndicatorsClassFiiiii` | 0x002a1c40 | PS2-ONLY | console inventory screen (5664 B) |
| - | - | `IntroSubtitlesAddSimple__17G_IndicatorsClassFUi` | 0x002a3690 | DIVERGED | PC `hud_intro_subtitles_add_centered` (0x0054b7b0) holds the slot, but the PS2 method was rewritten for the PS2-only hot-rod unlock — **twin deliberately NOT applied** |
| - | - | `TutorialAddText__17G_IndicatorsClassFUi` | 0x002a3970 | PS2-ONLY | tutorial text (§4b) |
| - | - | `MapIsEnabled__17G_IndicatorsClassFv` | 0x002a3c80 | INLINED | 1-line getter; no distinct PC function |
| - | - | `CompassSetDestination2__17G_IndicatorsClassFP9I3D_frame` | 0x002a3e30 | PS2-ONLY | second compass slot; PC has one destination only |
| - | - | `MapGetScrollPos__17G_IndicatorsClassFRfRff` | 0x002ab490 | PS2-ONLY | the PC map does not scroll — confirmed in-game AND in code: the PC `DrawMap` branch carries no scroll state to query |
| - | - | `DrawMap__17G_IndicatorsClassFfff` | 0x002ab690 | INLINED | **inlined into `hud_render_all`**, the bit-0x2000 branch ¹³ |
| - | - | `DrawMessage__17G_IndicatorsClassFPUc` | 0x002ad540 | PS2-ONLY | a centred message box: `TextSizeMultiLine`, two `RectangleFilled` calls, then `OutTextMultiLine` (§4b) |
| - | - | `AByloTicho__17G_IndicatorsClassFi` | 0x002af940 | PS2-ONLY | dev-joke name; mutes `isound` for N HUD ticks through the countdown at PS2 `+0x4e30` (§4b) |
| - | - | `TextSizeMultiLine__17G_IndicatorsClassFRfRfPUcfUi` | 0x002b2470 | PS2-ONLY | the `~`-split measure loop over `TextSize` (§4b) |
| - | - | `ParheliaSetFov__17G_IndicatorsClassFP10I3D_cameraf` | 0x002b32e0 | PS2-ONLY | Matrox Parhelia multi-head field-of-view and aspect wrapper — see §4b |
| - | - | `PleaseWait__17G_IndicatorsClassFb` | 0x002b35e0 | INLINED | **inlined into `game_done`**; the same block also sits in `garage_game_done` (§4b) |

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

**The class head is complete.** `hud_ctor`, `hud_dtor`, `hud_reset_values`, `hud_load_font`,
`hud_language_select`, `hud_create` and `hud_destroy` are all confirmed in PC
`0x00548420`–`0x005494b0`, and `GameInit` is INLINED into `game_init`, which is also why no PC
function in the cluster comes close to PS2's 6316 bytes. The range `[0x00548ac0, 0x00556f10)` is
180 starts with **0 unnamed**. Six functions inside the cluster are foreign to this class and are
named as such: `recorder_key_stream_new_chunk`, `recorder_key_stream_alloc` (a chained bump
allocator), `recorder_key_stream_size`, `actor_track_key_offset_of`, `morph_frame_list_add` (gates
on the `'MRPH'` magic) and `speech_pause_all_channels` — they are what shift this class's
emission-order ranks. Every member of the class carries the `hud_*` prefix.

### 4a. The two resolution-scale globals

`g_hud_scale_x` (`0x006613f0`, 21 refs) and `g_hud_scale_y` (`0x006613f4`, 24 refs) are the
horizontal and vertical resolution scale factors every virtual-coordinate draw call multiplies by.
Their INSTANCE twins are the class fields `+0x90c0` and `+0x90c4`.

They sit in the `g_loadsave` translation unit, because that unit holds an off-family
`G_IndicatorsClass` helper: `hud_text_draw_line_scaled` (`0x00560550`), whose only consumer is
`credits_render_one` — which is why it was emitted next to the credits code rather than with the
rest of the class. Its definition lives inside `[0x00556f10, 0x00561b70)`, so it belongs to that
translation unit; only its receiver is the HUD singleton.

### 4b. Two rows with measured detail worth keeping

**`hud_set_race_start_flag` ↔ `RaceSetStartFlag` — what the widget depicts.** The PC hide-gate
(`!= 4 && != -1`) looks like a place domain of at most four, but the CALLER settles it: `racing_tick`
drives a **1000 ms-per-step start countdown** that calls this with the step counter `C_racing+0x2a8`
— 0, then 1, then 2, then 3 — and at 4 releases the drivers, stamps the race start time and switches
to the racing state. So `param_2 == 4` means *the race has started, hide the lights*, and the `<= 4`
domain is the countdown length, not a placing. PS2 passes `2 - counter` for the three lights and `7`
for GO — a different encoding of the same widget. Detail: `basic/artifacts/racing.md`,
`structs/racing.md`.

**`ParheliaSetFov` — PS2-only, measured over its three callers.** The PS2 calls it from
`CumNaNe`, `SetSniperFov` and `GM_Carcyclopedia::OnCreate`. Their PC twins —
`camera_lock_at_human_pair`, `camera_set_sniper_fov` and `carcyclopedia_ctor` — each call the
`I3D_camera`'s own FOV vtable slot `0x50` directly with the same argument (π/4 = `0x3f490fdb`, the
sniper field of view, π/3 = `0x3f860a92`), so no PC code carries the Parhelia branches.
`structs/camera.md` §6d.

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

## 5. Related

- `basic/artifacts/hud_elements.md` — the prose companion: `+0x90a4` bit map, script commands that
  drive HUD elements, in-game-confirmed observations, §8 PC-vs-PS2 differences.
- `structs/console.md` — `C_console`, the separate class whose render/register/unregister trio is
  NOT part of this one.
- `structs/gui_screen.md` — the mouse cursor and the two-font-table layout.
- `structs/camera.md` — `g_hud_flags` and the `ParheliaSetFov` callsites.
- `structs/racing.md` and `basic/artifacts/racing.md` — the race readouts and the start countdown.
- `structs/web_node.md` / `structs/web_path.md` — the `C_web_node` graph the radar road-edge strips
  are built from, owned by `C_web_path` at `*(g_act_mission+0x2c)`.
- `Game-ghidra.exe.functions.md` — the function registry, PS2-twin column filled.

## 6. Open items

| item | state |
|---|---|
| the PS2 minimap renderer (§3) | not compared against the PC radar renderer in `hud_render_all` |
