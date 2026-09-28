# `pursuit_event_manager` — the chase driving system (`C_sledovacka`)

**Created** 2026-07-09 20:19 +0200 · **Modified** 2026-09-17 17:53 +0200

PC prefix **`pursuit_event_`**. Class `C_sledovacka` (*sledovačka* = Czech "chase / tail").

The engine-wide system that drives a car physically chasing (or fleeing) a target — used by
police pursuits, AI combat chases, and scripted mission chases alike. Distinct from the police
squad bookkeeping in `structs/police-board.md` / `structs/police-action.md`: those decide *who*
chases; `C_sledovacka` executes the *driving*.

Reconstructed from static analysis of the disassembly and the annotated decomp.

**Confidence tags:** **[H]** disasm/body-read + writer + reader; **[M]** body-read,
semantics partly inferred; **[L]** weak/preliminary; **[U]** not observed.

## 1. Instance location and shape

There is one live `C_sledovacka` instance, a **heap sub-object** embedded inside the game/
world object:

```
manager = [[0x0065115C] + 0x24] + 0x2B18
```

Every `AddCarHunt` / `RemoveCar` callsite loads `this` (ECX) with exactly this expression
(`mov eax,[0x0065115C] ; mov ecx,[eax+0x24] ; add ecx,0x2B18`). The manager owns a
`std::vector` of **S_ELEMENT** chase records at `manager+0x14`(begin)/`+0x18`(end), stride
**0x44** (68 bytes).

Separately, `C_sledovacka` keeps a **file-static** data block at **0x0064C6C0** (not part
of the heap instance) whose byte at `+0x18` (**0x0064C6D8**) is the `EnableChangeTables`
boost flag `g_pursuit_change_tables` — see §5.

```c
// heap instance (partial)
struct pursuit_event_manager {              // @ [[0x0065115C]+0x24]+0x2B18
    /* +0x14 */ S_ELEMENT* records_begin;   // [H] std::vector<S_ELEMENT>, stride 0x44
    /* +0x18 */ S_ELEMENT* records_end;     // [H]
    /* +0x1C */ S_ELEMENT* records_cap;     // [M] vector capacity end (std::vector idiom)
    /* +0x20 */ uint32_t   first_dispatch_ts; // [M] from g_act_mission+0x30 on the first
                                            //     dispatch (0 until then)
    /* ...    */ // path / S_BRIDGE / tick / savegame state — see §11
};
```

## 2. `S_ELEMENT` — one chase record (0x44 bytes)

One record per car currently being driven by the chase system. Built by the three `AddCar*`
entry points (§9a); the dispatch de-dups on `+0x00` (one record per vehicle) and requires any
target to be entity type 2 (player-actor) or 0x1B (NPC).

```c
struct S_ELEMENT {                          // 0x44 bytes; element of manager+0x14 vector
    /* +0x00 */ C_car*   vehicle;           // [H] the chase car (dedup key; record[0])
    /* +0x04 */ C_driver* driver;           // [H] 0x29C-byte C_driver; driver_ctor / driver_dtor
    /* +0x08 */ C_car*   target_vehicle;    // [H] target's current vehicle (= target+0x98) or 0
    /* +0x0C */ C_actor* target;            // [H] pursued actor (0 => chase a fixed position)
    /* +0x10 */ float    dispatch_pos[3];   // [M] +0x10..+0x18 fixed destination (target == 0)
    /* +0x1C */ uint32_t event_flags;       // [M] ((sub_event >> 8) & 0x201F) | 1
    /* +0x20 */ uint32_t action_status;     // [H] E_STATUS (init 0; getter default 1)
    /* +0x24 */ uint32_t mode;              // [H] chase TYPE, "VoleMode", set at build only (§8)
    /* +0x28 */ uint32_t chase_tracking_confirmed; // [M] zeroed at build, never written; read
                                            //     by is_chase_tracking behind a mode==1 gate
    /* +0x2C */ // remaining fields (duration, pose) not enumerated ([U])
};
```

**`driver` sub-record — a `C_driver` (0x29C bytes, at `+0x04`)** — dispatch-time writes:
`+0x84 = speed_factor` (`g_police_speed_factor`), `+0x170 = 0`, `+0x1FC = target`,
`+0x200 = target_vehicle`, `+0x208 = 1`, `+0x218 = 1000.0f`. The type is `C_driver`: the dispatch
allocates `operator_new(0x29C)` and calls `driver_ctor`.

**Vehicle fields the chase system stamps** (on the `C_car`, not the record):
`+0x1AC` under-pursuit flag (1 on dispatch / 0 on remove), `+0x2038` chase-flag word
(`0xC00` on dispatch / 0 on remove), `+0x2040 = 1.0f` on remove, `+0x51C/+0x51D` chase-order
markers, `+0x215A = 0` on remove.

## 3. Field evidence

| offset | field | conf | writer(s) | reader(s) | notes |
|---|---|---|---|---|---|
| mgr +0x14/+0x18 | records_begin/end | H | `pursuit_event_dispatch_pursuit_vehicle` (insert), `pursuit_event_remove_vehicle` (erase-shift) | both, plus every `pursuit_event_record_*_by_vehicle` getter/setter | `std::vector<S_ELEMENT>`, stride 0x44 |
| mgr +0x20 | first_dispatch_ts | M | `pursuit_event_dispatch_pursuit_vehicle` (`= g_act_mission+0x30` on first dispatch) | - | one-shot stamp |
| rec +0x00 | vehicle | H | dispatch (`= param_2`) | dispatch (de-dup scan), remove (match key), all getters | chase car |
| rec +0x04 | driver (C_driver) | H | dispatch (`operator_new(0x29C)` + `driver_ctor`) | remove (`driver_dtor` + free) | 0x29C C_driver sub-record |
| rec +0x08 | target_vehicle | H | dispatch (`= target+0x98`) | - | 0 when target on foot |
| rec +0x0C | target | H | dispatch (`= param_3`) | dispatch (visible test) | type 2 or 0x1B |
| rec +0x10..0x18 | dispatch_pos | M | dispatch (`= *param_6` when target==0) | - | fixed destination |
| rec +0x1C | event_flags | M | dispatch | - | event-flag mask |
| rec +0x20 | action_status (E_STATUS) | H | dispatch (0), `pursuit_event_record_set_action_status_by_vehicle`, tick | `pursuit_event_record_get_action_status_by_vehicle` (default 1); remove (`!= 3` gates siren reset) | - |
| rec +0x24 | mode (VoleMode) | H | build only: Hunt→1/2, Escape→3, MoveTo→4 | `pursuit_event_record_get_mode_by_vehicle` (default 0); remove (`in {3,4}` ⇒ un-boost) | chase TYPE, values §8 |
| rec +0x28 | chase_tracking_confirmed | M | never written (zeroed at build) | `pursuit_event_record_is_chase_tracking_by_vehicle` | gated active-hunt predicate |

## 4. Method family (`pursuit_event_*` / `C_sledovacka`)

| VA | PC name | PS2 twin | role |
|---|---|---|---|
| `0x004EC1B0` | `pursuit_event_ctor` | `__ct__12C_sledovackaFv` | constructor |
| `0x004EC1E0` | `pursuit_event_set_change_tables` | `EnableChangeTables__12C_sledovacka` | write the boost flag `g_pursuit_change_tables` (§5) |
| `0x004EC1F0` | `pursuit_event_init` | `Init__12C_sledovacka` | init |
| `0x004EC4F0` | `pursuit_event_done` | `Done__12C_sledovacka` | teardown |
| `0x004EC590` | `pursuit_event_is_car_chased` | `HoniMiHoNekdoZKanclu__12C_sledovacka` | is this car chased? |
| `0x004EC5F0` | `pursuit_event_dispatch_pursuit_vehicle` | `AddCarHunt__12C_sledovacka` | enroll a car + target actor; applies the model-level boost when `g_pursuit_change_tables` is set (§5) |
| `0x004EC980` | `pursuit_event_dispatch_escape_vehicle` | `AddCarEscape__12C_sledovacka` | enroll a car fleeing a target actor ¹ |
| `0x004ED0B0` | `pursuit_event_dispatch_move_to_vehicle` | `AddCarMoveTo__12C_sledovacka…` | enroll a car driving to a destination `S_vector` |
| `0x004EDAC0` | `pursuit_event_remove_vehicle` | `RemoveCar__12C_sledovacka` | cancel a car's chase record; resets the car to model level 0 |
| `0x004EDC20` | `pursuit_event_recompile_path` | `RecompilePath__12C_sledovacka` | path |
| `0x004EE740` | `pursuit_event_prepare_path` | `PreparePath__12C_sledovacka` | path |
| `0x004EE900` | `pursuit_event_tick_driver` | `TickDriver__12C_sledovacka` | per-driver tick |
| `0x004EFCF0` | `pursuit_event_tick` | `Tick__12C_sledovacka` | per-frame tick |
| `0x004F4470` | `pursuit_event_record_get_action_status_by_vehicle` | `ActionStatus__12C_sledovacka` | E_STATUS getter (record `+0x20`), default 1 |
| `0x004F44D0` | `pursuit_event_record_set_action_status_by_vehicle` | `SetActionStatus__12C_sledovacka` | E_STATUS setter (record `+0x20`) |
| `0x004F4530` | `pursuit_event_record_get_mode_by_vehicle` | `GetVoleMode__12C_sledovackaFP5C_car` | mode getter (record `+0x24`), default 0; values §8 |
| `0x004F4590` | `pursuit_event_drop_actor_references` | `VypicujMiToyotu__12C_sledovackaFP7C_actor` | drop references to an actor |
| `0x004F46E0` | `pursuit_event_record_is_chase_tracking_by_vehicle` | `DevaaateeeZasNeznaCas__12C_sledovacka` | gated active-hunt predicate ² |
| `0x004F4760` | `pursuit_event_save_game_get_size` | `SaveGameGetSize__12C_sledovackaFv` | savegame (§9b) |
| `0x004F4870` | `pursuit_event_save_game_save` | `SaveGameSave__12C_sledovackaFPPv` | savegame (§9b) |
| `0x004F4DC0` | `pursuit_event_save_game_load` | `SaveGameLoad__12C_sledovackaFPPv` | savegame (§9b) |

Each row's twin grade is in the function registry, `ghidra/Game-ghidra.exe.functions.md`.

¹ Same dedup + `{2, 0x1b}` gate + `vehicle_set_engine_on` + `C_driver` sub-record as `AddCarHunt`;
identical km/h→m/s `×0.2777778` (= /3.6), squared distance, bool→1000.0f.
² Returns record `+0x28` only while mode == 1 with a live target vehicle and flag bit 2 clear. The
`+0x28` field is never written, so it functions purely as "is this vehicle in an active actor-hunt
with a live target vehicle". It is not the mode getter.

`ai_fight_manager_join_party` (`0x0043E610`) and `ai_fight_manager_adjust_form_time`
(`0x0043F950`) are **not** members: their receiver is the `ai_fight_manager` singleton
`0x00655318`, and the stride-0x48 record list they walk is a fight party
(`structs/ai_fight_manager.md` §2a).

## 5. The boost flag (`g_pursuit_change_tables`)

`pursuit_event_set_change_tables(bool)` writes the file-static byte at **0x0064C6D8**
(= `C_sledovacka` static block `0x0064C6C0 + 0x18`). When set, `AddCarHunt` swaps the chase
car to its **top model level** (`car_set_model_level(count-1)`), which re-applies that
variant's CARINIT physics table (faster + better handling = "boosting"); `RemoveCar` reverts
to level 0. Only cars with `> 1` model level can be boosted — the level count comes from the
car's record in `tables/vehicles.bin` (14 cars carry a 2nd, boosted CARINIT block; not tied
to model files — `police-system/artifacts/boosting.md` §3). This byte is what the LiveSplit
autosplitter reads as `boostEnabled` (`0xE7D64, 0x18`). Full mechanism, toggle map, and
boostable-car list in `police-system/artifacts/boosting.md`.

## 6. Cheat Engine

- **Boost flag (`boostEnabled`):** 4-byte at `Game.exe+0xE7D64` → `+0x18`, i.e. absolute
  `0x0064C6D8`. 1 = a chase is boosting, 0 = not.
- **Chase record N (only while a chase is active):**
  base `[[[0x0065115C]+0x24]+0x2B18 +0x14]`; record N at `base + N*0x44`.
  Useful fields: `+0x00` chase car ptr, `+0x0C` target ptr, `+0x24` mode (VoleMode, §8),
  `+0x20` action_status (E_STATUS).

## 7. Cross-references

- `police-system/artifacts/boosting.md` — the boost mechanism this system implements.
- `structs/police-board.md` / `structs/police-action.md` — the police squads that call
  `AddCarHunt`/`RemoveCar` to drive their chase vehicles.
- `structs/ai_fight_manager.md` — the fight-party manager (not part of this class).
- `ghidra/Game-ghidra.exe.functions.md` — `pursuit_event_*` rows + PS2-twin columns.

## 8. `mode` / "VoleMode" values (record `+0x24`)

The chase record's **mode** is the developers' "VoleMode" field — a jokey name (*vole* = Czech
slang, ≈ "dude/idiot"; siblings include `VypicujMiToyotu`, `HoniMiHoNekdoZKanclu`). It records
*what kind of driving the AI chase driver is doing*, and is the field the police squad polls each
tick to decide whether to re-issue the chase order. It is **distinct** from `action_status`
(E_STATUS, `+0x20`).

**PC getter:** `pursuit_event_record_get_mode_by_vehicle` (`0x004F4530`, default 0 on miss).
There is no setter.

| value | meaning | evidence / conf |
|---|---|---|
| **0** | no active chase record for this vehicle (getter miss default) | [H] both getters return 0 on miss |
| **1** | hunt a target actor; stamped by `AddCarHunt` when a target actor is passed. The "desired" engaged mode — the police tick leaves the chase alone while mode == 1 | [H] `AddCarHunt` build (target present); the `+0x28` accessor's gate is `mode==1` |
| **2** | drive to a fixed position (no target); stamped by `AddCarHunt` when `target == 0` | [H] `AddCarHunt` build (`target==0` ⇒ 2) |
| **3** | flee / escape a target; stamped by `pursuit_event_dispatch_escape_vehicle` | [H] build sets mode = 3 |
| **4** | scripted move-to a position; stamped by `pursuit_event_dispatch_move_to_vehicle` | [H] build sets mode = 4 |

**Mode is the chase TYPE stamped at BUILD time — it is NOT a runtime tick transition.** No
`C_sledovacka` method writes the mode field after the record is built (`pursuit_event_tick` and
`pursuit_event_tick_driver` never touch it). Each `AddCar*` entry point hard-codes its own value.
`pursuit_event_remove_vehicle` un-boosts (model level → 0) when mode ∈ {3,4} **even if the global
boost flag `g_pursuit_change_tables` is clear** — escape and move-to always restore level 0 on
teardown, while hunt (1/2) defers to the global flag.

**Consumer (re-dispatch):** the police squad tick reads `get_mode(chase_car)` at `group+0x5c`;
when the mode is not the desired value it corrects the chase — sets a car flag, toggles
`pursuit_event_set_change_tables` (boost), then `pursuit_event_remove_vehicle` +
`pursuit_event_dispatch_pursuit_vehicle` with a fresh event id chosen from the squad's own
state.

## 9. Method coverage — 21 of 22 PS2 methods located

The translation unit is `[0x004ec1b0, 0x004f60b0)` — **51 PC function starts**, bounded by the CRT
static-init/atexit pairs `0x004ec180`/`0x004ec1a0` and `0x004f60b0`/`0x004f60d0` (a file-scope
static's own init/atexit pair marks a TU boundary). None of them is a phantom start. The 21
class members are listed in §4; the only PS2 method without a PC function is the destructor
`__dt__12C_sledovackaFv` — the TU head holds only the ctor and `pursuit_event_set_change_tables`.

### 9a. The three PUBLIC entry points

The class has exactly three ways in, and they are the three `AddCar*` methods:

| PC | developer name | extra argument that distinguishes it |
|---|---|---|
| `pursuit_event_dispatch_pursuit_vehicle` | `AddCarHunt` | a `C_actor*` to hunt |
| `pursuit_event_dispatch_escape_vehicle` | `AddCarEscape` | a `C_actor*` to flee |
| `pursuit_event_dispatch_move_to_vehicle` | `AddCarMoveTo` | no actor at all — a destination `S_vector` |

All three share one callee spine: `vehicle_set_engine_on` → `operator_new` → `driver_ctor`
→ `driver_set_car` (per-vehicle dispatch setup) → record insert → `roads_get_near_node` ×2 → …
→ `pursuit_event_prepare_path` → `pursuit_event_recompile_path`. The script side reaches
them at `entity_process_command` cases `0x100c`/`0x100d`/`0x100e`.

### 9b. The savegame chunk

`pursuit_event_save_game_get_size` / `_save` / `_load` are a closed triple. The chunk tag
is **6**; each surviving record costs **0xb8 bytes** plus `0x70` per driver path node, and
the header is 4 bytes. All three apply the identical five-clause survival guard — the car,
the driver and both tracked actors must not be mid-teardown (`actor+0x65` clear) and the
car's `+0x1c` must not be 2. `_load` wipes every live record through
`pursuit_event_remove_vehicle` before rebuilding.

### 9c. The TU also hosts other classes' inline members

Thirteen PC starts in this range are out-of-line inline members of classes whose headers are
included in this TU. They keep their OWN class prefix, not `pursuit_event_`:
`vehicle_get_engine_on`, `vehicle_set_clutch_gate`, `vehicle_set_horn`,
`vehicle_get_siren`, `vehicle_is_throttle_applied`, `vehicle_get_up_axis`,
`vehicle_get_wheel_count`, `vehicle_get_wheel_at` (all `C_Vehicle`),
`car_set_condition_channel` (`C_car`), `actor_is_death_processed` and
`actor_is_in_camera_sector` (`C_actor`), `driver_path_node_at` (`C_driver`) and
`i3d_frame_get_world_matrix_ptr` (shared).

**How the `vehicle_` / `car_` receiver split is proved:**

1. *The same caller distinguishes the two receivers.* Inside `pursuit_event_tick`,
   `vehicle_set_power` (`C_Vehicle::SetPower`) is reached as `mov ecx,[esi] ; add ecx,0x70`,
   while `car_get_condition_fraction` (`C_car::ShowCarEnergy`) and `car_get_owner` are reached as
   `mov ecx,[ebp]` with **no** `+0x70`. The `+0x70` step is the compiler's own discriminator, not
   an inference.
2. Eight of the nine vehicle/car accessors carry that step; `car_set_condition_channel` does not.
3. A field-offset census over the whole binary agrees — `+0xca8`, `+0x548`, `+0x540`,
   `+0xcb4`, `+0xcd0`, `+0x5bc/+0x5c0` and `+0x4ae` are read only by named `vehicle_*` functions,
   while `+0x20b8` is read only by `car_get_condition_fraction`.

`box_game_init` / `box_write_state_to_frame` / `box_apply_net_event_msg` also touch `+0xcb4` and
`+0xcd0`. Those are **`C_box`** at coincidentally equal offsets, not evidence for either receiver.

## 10. PC vs PS2

**Record layout.** PC `S_ELEMENT` is stride 0x44, PS2 is 0x50: the PS2 record carries an extra
`+0x1c` position word, so every field from `event_flags` onward sits +4 higher on PS2 —
`event_flags` PS2 `+0x20`, `action_status` PS2 `+0x24`, `mode` (VoleMode) PS2 `+0x28`,
`chase_tracking_confirmed` PS2 `+0x2c`. The savegame blob follows: 0xb8 on PC, 0xbc on PS2. The
driver path node is 0x70 on PC and 0x90 on PS2.

**`GetVoleMode` ↔ `+0x24`, `DevaaateeeZasNeznaCas` ↔ `+0x28`.** PS2 `+0x28` (VoleMode) == PC
`+0x24` (mode). The PC `+0x28` accessor reads PS2 `+0x2c`, and its twin is
`DevaaateeeZasNeznaCas__12C_sledovacka` (byte-exact, PS2 VA `0x002100b0`).

**Mode writers on PS2:** `AddCarEscape` sets the mode field = 3 and `AddCarMoveTo` = 4 in the
`S_ELEMENT` each pushes. The PS2 `ai_police_action` tick reads `GetVoleMode` and branches on
`!=1 / ==0 / {2,3,4}`.

**`C_driver` twin:** PS2 `__ct__8C_driver` after `__nw__(0x320)` == PC `driver_ctor` after
`operator_new(0x29C)`, fixed by the `AddCarEscape` body-diff.

**Size-ratio band.** This TU runs 0.48–0.71 (PC ÷ PS2). `AddCarEscape` 0.481 and
`AddCarMoveTo` 0.475 are the two extremes and they agree with each other.

**Proven PC inlinings**, marked in the decomp at their sites with `// ===== INLINED ON PC:`
banners (each is the only PS2 call missing at its mirrored position while everything around it
matches):

| PS2 symbol | folded into (PC) |
|---|---|
| `__as__Q212C_sledovacka9S_ELEMENT` (operator=) | the erase-shift loop in `pursuit_event_remove_vehicle` |
| `__ct__Q212C_sledovacka9S_ELEMENT` (copy ctor) | `std_vector44_insert_one` |
| `reserve__Q23gif36vector<S_ELEMENT,8>` | `std_vector44_insert_one`'s grow path |
| `__ct__10S_car_nodeFRC10S_car_node` | `stl_copy_range_stride70` |

**Not marked:** PS2 `RemoveCar` also calls `SetDynamicCollsOn__9C_Vehicle` and
`SetClutch__9C_Vehicle`, and neither appears in `pursuit_event_remove_vehicle` (§11).

**`AddCarEscape` lockstep:** PS2 `GetAIPos__5C_carFb` sits at the position where the PC calls
`roads_find_path` (`C_roads::FindPath`) — a positional coincidence, not a twin.

## 11. Open items

| item | state |
|---|---|
| `__dt__12C_sledovackaFv` | no PC function located; the TU head holds only the ctor and `pursuit_event_set_change_tables` |
| `RemoveCar`'s `SetDynamicCollsOn` / `SetClutch` | absent from `pursuit_event_remove_vehicle`; as consistent with a behaviour difference as with a fold |
| manager fields beyond `+0x20` | path buffers, `S_BRIDGE` vector, tick cursors, save/load state not enumerated |
| `S_ELEMENT +0x2C..+0x43` | duration, pose — not enumerated |
| `C_driver` full layout | only the dispatch-time writes (§2) are recorded |
| runtime validation | none of this file is validated at run time |
