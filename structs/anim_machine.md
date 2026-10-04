# `C_animations_machine_de_luxe` — the animation machine (`anim_machine_`)

**Created** 2026-07-13 11:22 +0200 · **Modified** 2026-09-29 19:47 +0200

The engine's per-object **animation state machine**: it owns the animation tracks, the
currently-playing clip, the blend state, the "extended" (overlay) animation slot, and the
recompiled bone matrix. It is a **sub-object**, not a scene actor — it has no `kind` tag and no
vtable of its own in the PC build.

| | |
|---|---|
| PC prefix | **`anim_machine_`** (the static track table and its helpers: `anim_track_*`) |
| PC cluster | `[0x005aa800, 0x005ad600)` |
| Object size | **`0x128`** (§2) |
| Method coverage | all 24 PS2 methods accounted for: 21 own PC functions, 2 inlined, 1 folded (§4) |

Confidence: treat every field as **[M]** unless marked **[H]**.

It sits under both the human animation path and the traffic/pedestrian path, so a function that
looks like `C_human` animation code may belong to this class instead.

---

## 1. Where the object lives (six owners)

Established by a scan of `.text` for every `E8` call to the machine's methods, then a back-trace of
the ECX source (`lea ecx,[reg+disp]` = EMBEDDED, `mov ecx,[reg+disp]` = POINTER) with each callsite
attributed to its containing function. The decomp cannot answer this on its own: MSVC passes the
machine in ECX and Ghidra hides the `this` at every callsite.

| Owner | How it holds the machine | Offset | Conf | Evidence |
|---|---|---|---|---|
| **`C_human`** (and its subclasses `C_player` / `C_entity`) | EMBEDDED | **`+0xbc`** | H | 23 distinct owning functions. Worked example — `human_do_play_anim` (`0x004a63f0`): prologue `8b f1` (`mov esi,ecx`, ESI = `this`), then `8d be bc 00 00 00` (`lea edi,[esi+0xbc]`), `8b cf` (`mov ecx,edi`), `call anim_machine_set_anim` |
| **`C_dog`** | EMBEDDED | `+0xe8` | H | 6 owning functions in the `0x0047b800..0x0047f3a0` cluster; the cluster's strings are `hlavapsa_I3D` (*hlava psa* = dog's head), `Datack_I3D` / `Dbark1_i3d` / `Dbreath_i3d` / `Ddeath_I3D` |
| **`C_pumper`** (gas-station attendant) | EMBEDDED | `+0xe0` | H | The band's strings are `_pumpa_*` (*pumpa* = fuel pump), the idle set `Kurak*` (*kuřák* = smoker) and `Nuda*` (*nuda* = boredom) — `structs/pumper.md` |
| **`C_traffic_car`'s crew element** (`S_ELEMENT`) | EMBEDDED | `+0x24` | H | `traffic_car_element_ctor` and `traffic_car_adopt_car_with_new_driver` both do `lea ecx,[reg+0x24]; call anim_machine_ctor`; the teardown goes through the adjustor thunk `thunk_stub_noop_cdecl_00574a40` (`add ecx,0x24`) — `structs/traffic_car.md` |
| `C_traffic_element` | POINTER | `+0x08` | H | `pedestrian_end_taxi_wave` / `pedestrian_deactivate` deref `*(elem+8)` |
| `S_rail_human` (the tram passenger, `rail_passenger_`) | POINTER | `+0x08` | H | Built by `railway_game_init` and `railway_add_person`, driven by `rail_passenger_set_anim` from the passenger state at `+0x14` through the table `g_rail_passenger_anims`, destroyed by `rail_passenger_done` — `structs/railway.md` §2b |

**The owner set is closed at six by the constructor's callsites.** `anim_machine_ctor` has exactly
nine `E8` callers: `human_ctor`, `dog_ctor`, `pumper_ctor`, `traffic_car_element_ctor`,
`traffic_car_adopt_car_with_new_driver` (the four embedded owners), `pedestrian_manager_add_pedestrian`
(a heap machine for a `C_traffic_element`), and `railway_game_init` ×2 plus `railway_add_person`
(heap machines for `S_rail_human`). Every heap machine is `operator_new(0x128)` followed by the ctor.

**`pedestrian_set_anim` and `pedestrian_drop_umbrella` deref `this+0x8` as an anim-machine
pointer**, so their `this` is a `C_traffic_element`, not a `C_human`.

### 1a. Construction and teardown

The machine's destructor is empty, so MSVC emits no body for it: every teardown calls the shared
bare `ret` `stub_noop_cdecl_00574a40` with ECX = the machine, and the heap owners then free the
block with `operator_delete`. The receivers are proven at every site: `human_dtor`
(`lea ecx,[esi+0xbc]`), `dog_dtor` (`+0xe8`), `pumper_dtor` (`+0xe0`), three EH unwind funclets
(`this+0xbc` ×2, `this+0xe0`), the crew-element thunk (`+0x24`), and `rail_passenger_done` /
`pedestrian_manager_clear_pedestrians` (the pointer, followed by `operator_delete`). The stub has
30 callers in all, so it is not a symbol of this class.

### Consistency check on `C_human + 0xbc`

The machine is **`0x128` bytes** (§2), so embedded at `+0xbc` it spans exactly `+0xbc .. +0x1e4`.
`structs/human.md` documents `C_human` fields at `+0x4c`, `+0x74`, `+0x98`, `+0xac` (all below
`+0xbc`) and then `+0x1e4` / `+0x1e5` / `+0x1e8` — **the first documented field after the machine
is exactly where the machine ends.**

---

## 2. Field layout (PC offsets)

```c
typedef struct C_animations_machine_de_luxe {
    /* +0x08 */ void    *anim_object;        // [H] the I3D animation/model object. ALL track work is
                                             //     vtable-dispatched through it: +0x64 = get track by
                                             //     index, +0x68 = stop track N, +0x7c = get track length.
    /* +0x0c */ uint8_t  flags[7];           // [M] 7 byte-flags written by SetAnim (+0xc..+0x12)
    /* +0x14 */ uint32_t f_14;               // [U] written by SetAnim
    /* +0x18 */ uint32_t f_18;               // [U] written by SetAnim
    /* +0x1c */ uint32_t active_track;       // [H] index of the ACTIVE track/clip; passed as the track
                                             //     selector to anim_object vtable +0x64 / +0x7c
    /* +0x20 */ uint32_t blend_block[8];     // [M] +0x20..+0x3c — per-clip blend/weight params (SetAnim)
    /* +0x40 */ float    matrix[16];         // [H] 4x4 recompiled bone matrix; RecompileMatrix copies
                                             //     0x10 dwords from anim_object+0x50 into here...
    /* +0x70 */ float    translation[3];     // [H] ...then ZEROES +0x70/+0x74/+0x78 — i.e. the matrix's
                                             //     translation row is discarded (rotation-only recompile)
    /* +0x80 */ uint32_t pose_block[8];      // [M] +0x80..+0x9c — second transform/pose block
    /* +0xdc */ uint32_t misc_block[6];      // [M] +0xdc..+0xf0 — written by SetAnim
    /* +0xf4 */ uint8_t  ext_active;         // [H] extended (overlay) animation is active
    /* +0xf5 */ uint8_t  ext_blending_out;   // [H] extended anim is blending out
    /* +0xfc */ uint32_t ext_time;           // [H] extended-anim time/phase (reset to 0 on blend-out)
    /* +0x100*/ float    ext_blend;          // [H] extended-anim blend time; stored NEGATED while
                                             //     blending out (`= -blend`)
    /* +0x104*/ uint8_t  ext_flag;           // [M] cleared on both blend-out paths
    /* +0x10e*/ int16_t  ext_clip_id;        // [H] the extended anim's CLIP ID. 0xffff (-1) = none.
                                             //     -0x2357 is the "hail a taxi" wave clip — that exact
                                             //     magic is how pedestrian_end_taxi_wave detects it.
    /* +0x110*/ uint8_t  locked;             // [H] 1 = the machine is frozen: every animation entry
                                             //     point returns without acting (§3)
    /* +0x114*/ uint32_t tail_block[4];      // [M] +0x114..+0x120 — written by SetAnim
} C_animations_machine_de_luxe;              // size 0x128
```

**Size `0x128`**, measured at every heap site: `pedestrian_manager_add_pedestrian`,
`railway_game_init` ×2 and `railway_add_person` each do `operator_new(0x128)` followed by
`anim_machine_ctor`.

**Track count = 8.** `anim_machine_stop_anims` loops `i = 0..7` calling
`anim_object->vtable[+0x68](i)`. The **extended/overlay animation uses track slot 2**
(`anim_machine_del_extended_anim` stops track `2` specifically).

## 3. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x08 | anim_object | H | `anim_machine_set_anim` | `anim_machine_get_active_anim_time`, `anim_machine_stop_anims`, `anim_machine_del_extended_anim`, `anim_machine_recompile_matrix` | the I3D animation object; every track op is a vtable call on it (+0x64/+0x68/+0x7c) |
| +0x1c | active_track | H | `anim_machine_set_anim` | `anim_machine_get_active_anim_time` | the track selector handed to anim_object's getters |
| +0x40..+0x7f | matrix | H | `anim_machine_recompile_matrix` | `anim_machine_set_anim` | 16 dwords copied from `anim_object+0x50` |
| +0x70..+0x78 | translation | H | `anim_machine_recompile_matrix` | - | zeroed right after the matrix copy |
| +0xf4 | ext_active | H | `anim_machine_del_extended_anim` | - | guard for the whole extended-anim teardown |
| +0xf5 | ext_blending_out | H | `anim_machine_del_extended_anim` | - | set when a blend time > 0 was given |
| +0xfc | ext_time | H | `anim_machine_del_extended_anim` | - | zeroed when the blend-out starts |
| +0x100 | ext_blend | H | `anim_machine_del_extended_anim` | - | stored as `-blend` while blending out |
| +0x10e | ext_clip_id | H | `anim_machine_del_extended_anim` | `pedestrian_end_taxi_wave` | `0xffff` = none; `-0x2357` = the taxi-wave clip |
| +0x110 | locked | H | `anim_machine_init_from_actor`, `anim_machine_init_from_traffic_element`, `anim_machine_init_from_model`, `anim_machine_reset` (0); `human_force_death`, `human_car_launched_into_air` (1) | `anim_machine_set_anim`, `anim_machine_is_active`, `anim_machine_tick`, `anim_machine_set_extended_anim`, `anim_machine_del_extended_anim`, `anim_machine_stage_blending_tick` | ¹ |

¹ Set to 1 only when a `C_human` dies outside the normal damage path: `human_force_death` plays the
death clip directly on the I3D frame and then writes `C_human+0x1cc` (= `+0xbc + 0x110`);
`human_car_launched_into_air` makes the same three stores (`+0x644` = 0, `+0x5d` = 0, the lock = 1).
From then on the six readers return without acting, so the death pose is never overwritten. The
three `Init`s and `Reset` clear it.

## 4. Method coverage — all 24 PS2 methods

PC VA order, PC name first. The last three rows have no PC function of their own.

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x005aab20 | `anim_machine_ctor` | `__ct__28C_animations_machine_de_luxeFv` | CONFIRMED |
| 0x005aab30 | `anim_machine_load_track` | `LoadTrack__28C_animations_machine_de_luxeFPc` | CONFIRMED |
| 0x005aaf80 | `anim_track_dtor` | - | PC-only ¹ |
| 0x005aafb0 | `anim_machine_get_track` | `GetTrack__28C_animations_machine_de_luxeFPc` | CONFIRMED |
| 0x005ab020 | `anim_machine_clear_non_base_tracks` | `ClearNoBaseTracks__28C_animations_machine_de_luxeFv` | CONFIRMED |
| 0x005ab170 | `anim_machine_init_from_actor` | `Init__28C_animations_machine_de_luxeFP7C_actor` | CONFIRMED |
| 0x005ab1e0 | `anim_machine_init_from_traffic_element` | `Init__28C_animations_machine_de_luxeFP17C_traffic_element` | CONFIRMED |
| 0x005ab250 | `anim_machine_init_from_model` | `Init__28C_animations_machine_de_luxeFP9I3D_model` | CONFIRMED |
| 0x005ab2b0 | `anim_machine_reset` | `Reset__28C_animations_machine_de_luxeFv` | CONFIRMED |
| 0x005ab300 | `anim_machine_sync_track_time` | `EqualTime__28C_animations_machine_de_luxeFii` | CONFIRMED |
| 0x005ab3a0 | `anim_machine_get_active_anim_time` | `GetActiveAnimTime__28C_animations_machine_de_luxeFRUiRUi` | CONFIRMED |
| 0x005ab3f0 | `anim_machine_stop_anims` | `StopAnims__28C_animations_machine_de_luxeFv` | CONFIRMED |
| 0x005ab410 | `anim_machine_recompile_matrix` | `RecompileMatrix__28C_animations_machine_de_luxeFR8S_matrix` | CONFIRMED |
| 0x005ab460 | `anim_machine_set_anim` | `SetAnim__28C_animations_machine_de_luxeFPcbbbffbbP8S_vectorbUi` | STRUCTURAL ² |
| 0x005abc80 | `anim_machine_is_active` | `IsActive__28C_animations_machine_de_luxeFv` | CONFIRMED ³ |
| 0x005abcc0 | `anim_machine_tick` | `Tick__28C_animations_machine_de_luxeFUi` | CONFIRMED ³ |
| 0x005acb70 | `anim_machine_set_extended_anim` | `SetExtendedAnim__28C_animations_machine_de_luxeFP17I3D_animation_setffb` | CONFIRMED |
| 0x005acc60 | `anim_machine_del_extended_anim` | `DelExtendedAnim__28C_animations_machine_de_luxeFf` | CONFIRMED |
| 0x005acce0 | `anim_machine_get_placement_from_track` | `GetPlacementFromTCK__28C_animations_machine_de_luxeFPcR8S_vectorR8S_vector` | CONFIRMED |
| 0x005ace20 | `anim_machine_get_track_end_point` | `GetEndPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector` | CONFIRMED |
| 0x005ace60 | `anim_machine_get_track_mixing_point` | `GetMixingPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector` | CONFIRMED |
| 0x005acee0 | `anim_machine_stage_blending_tick` | `StageBlendingTick__28C_animations_machine_de_luxeFUi` | CONFIRMED |
| - | - | `ClearTracks__28C_animations_machine_de_luxeFv` | inlined into `engine_shutdown` ⁴ |
| - | - | `GetAnimBaseDelta__28C_animations_machine_de_luxeFP17I3D_animation_set` | inlined into `anim_machine_set_anim` ⁵ |
| - | - | `__dt__28C_animations_machine_de_luxeFv` | folded onto `stub_noop_cdecl_00574a40` ⁶ |

¹ The destructor of ONE 100-byte track record: it frees the two key vectors at `+0x48` and `+0x58`.
The PS2 record holds raw arrays freed inline by `ClearTracks` / `ClearNoBaseTracks`, so it has no
counterpart.
² 2073 B against 2812 B, body not diffed in full; the position is boxed 1:1 by eight anchors — a
structural argument, not a body one.
³ Both fell out of the `human_extern_ai_update` body diff: the PS2 calls `IsActive(this+0x250)` and,
if true, `Tick(this+0x250, t)`; the PC function makes exactly those two calls in that order with the
same guard polarity, its receiver hidden in ECX because `C_human` embeds the machine at `+0xbc`.
⁴ The table clear `engine_shutdown` does after the anim-cache clear and before the driver release
(§4a) — the position of the only PS2 call, in `CloseSystem`.
⁵ The `base clip == -1` branch of `anim_machine_set_anim`: look up the node `"blnd"` in the
animation set and take its position key 0 (zero vector when absent). The literal `"blnd"`
(`0x0064e43c`) has exactly one `.text` reference, inside `anim_machine_set_anim`.
⁶ The destructor is empty; §1a lists every call.

The PC cluster preserves the PS2 emission order: every located pair comes out in the same relative
order on both sides. Interpolation is only safe between two anchors with exactly one gap — by strict
index the run after `SetAnim` would put `Tick` at `0x005acb70`, which is 0xf0 bytes before the
`DelExtendedAnim` anchor while PS2 `Tick` is ~0x11d0 bytes, so the cluster has extra and/or missing
entries. Body-diff everything else.

### 4a. The static track table

| PC address | name | meaning |
|---|---|---|
| `0x006efdb8` | `g_anim_track_table_dtor_guard` | once-flag for the atexit destructor |
| `0x006efdbc` | `g_anim_track_table` | STL allocator byte |
| `0x006efdc0` | `g_anim_track_table_begin` | records of **100 bytes** each |
| `0x006efdc4` | `g_anim_track_table_end` | - |
| `0x006efdc8` | `g_anim_track_table_cap` | - |
| `0x006efdcc` | `g_anim_base_track_count` | the number of records loaded by the base preload (§4b); `anim_machine_clear_non_base_tracks` pops the table down to it |

One record = a track name at `+0x00`, two `0xc`-stride key vectors at `+0x48` and `+0x58`, and the
header fields `anim_machine_load_track` reads. Tracks are loaded from `anims\%s`, and **the file's
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

### 4b. The base preload (`game_init`)

Inside `game_init`'s heavy-init block (`C_game+0x2f9c`, `structs/game.md`), the engine walks
`g_anim_info_table` to its `0xffff` terminator and loads every clip's track with
`anim_track_load_by_name`. A clip name containing `?` is loaded twice, with the `?` replaced by
`L` and then by `R`. While a network session is active the loop ticks it (and the loading screen)
whenever more than 500 ms have passed. Then, only
if `g_anim_base_track_count` is still 0, it is set to the table's record count
(`(end − begin) / 100`, the `0x51eb851f` reciprocal). This is its only writer, so the base set is
fixed by the first heavy init; `dog_load_tracks` runs after the seed, so the dog's tracks sit above
the base mark.

## 5. PC vs PS2

- **`C_human` embeds the machine at PC `+0xbc` ↔ PS2 `+0x250`**; `C_traffic_element` holds the
  pointer at PC `elem+0x08` ↔ PS2 `elem+0x188`. The PS2 `C_traffic_car` crew element holds a
  POINTER and deletes it in `DeactivateElem`, `GameDone` and `VypraskeeeKaaaru`, where the PC embeds
  it at `+0x24`.
- **Size:** PS2 heap machines are `__nw__FUi(0x140)`; PC `0x128`.
- **Field delta:** `ext_clip_id` PC `+0x10e` ↔ PS2 `+0x12a` (pinned by the shared `-0x2357` magic)
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
  under a static once-flag; the PC inlines the same loop and the same seed into `game_init` (§4b).

## 6. Cross-references

- `structs/human.md` — `C_human` embeds the machine at `+0xbc`.
- `structs/pumper.md` — the gas-station attendant, embedded at `+0xe0`.
- `structs/traffic_car.md` — the crew element, embedded at `+0x24`.
- `structs/railway.md` §2b — `S_rail_human`, pointer at `+0x08`.
- `structs/game.md` — the heavy-init flag that gates the base preload.
