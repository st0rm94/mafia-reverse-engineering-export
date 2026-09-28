# `police_manager` & `pursuit_record` — struct layout

**Created** 2026-07-20 19:59 +0200 · **Modified** 2026-09-28 19:34 +0200

Two C++ structs (unnamed in the binary) reconstructed from static analysis of
`Game-ghidra.exe.annotated.c`. **`police_manager`** is the police tracker embedded as a **SINGLE
instance** in the C_game game-world singleton at `C_game + 0x2DA0` (reached as
`*(g_act_mission+0x24)+0x2DA0`, `g_act_mission` = `0x0065115c`) — **not per-actor** (byte-proven,
§1). **`pursuit_record`** is one heap record per tracked suspect, owned by the manager's records
vector. Field names match the rest of the police docs. This file is the spec; the `Conf` columns flag
how solid each field is, and runtime validation with Cheat Engine / ReClass is still pending.

---

## 1. Overview

```
C_game object  (the game-world singleton, *(g_act_mission+0x24))
+----------------------------------------------------------+
| +0x2DA0  police_manager          embedded, single, 0xB0    |
|   +0x00  enabled               master on/off             |
|   +0x40  records[]             vector<pursuit_record*>   |
|   +0x50  police_vehicles[]     reserve pool (cars)       |
|   +0x60  spike_strips[]        reserve pool (barriers)   |
|   +0x70  foot_cops[]           reserve pool (npc*)       |
|   +0x80  npc_list[]            vector<npc*> (tracked)    |
|   +0x90  tracked_police_cars[] cars the manager owns      |
|   +0xA0  rndr_arr_b[]          vector<uint16_t>          |
+----------------------------------------------------------+

records[] entries are pointers to heap pursuit_record structs:

pursuit_record   (heap, 0x80 bytes, one per tracked suspect)
+----------------------------------------------------------+
|   +0x00  target                the tracked actor         |
|   +0x04  linked_vehicle        car being driven          |
|   +0x08  offense_bits_a        offence bitfield          |
|   +0x0A  offense_bits_b        long-decay mirror         |
|   +0x3C  police_state          1 fine / 2 cuffs / 3 gun  |
|   +0x3E  police_state_pending  pending level             |
|   +0x44  wanted_bar_remaining  wanted-bar countdown      |
|   +0x50  wanted_substate_flag  'all cops alerted'        |
|   +0x51  identity_pending      1 searching / 0 known     |
|   +0x74  officers[]            vector<officer*>          |
+----------------------------------------------------------+
```

- **`police_manager`** is **embedded** (not separately allocated) at `C_game + 0x2DA0`, size
  **0xB0 (176)** — the next C_game member begins at `C_game + 0x2E50`. It is plain (no vtable;
  `+0x00` is the `enabled` byte). It is a **SINGLE instance** on the C_game game-world singleton
  (`*(g_act_mission+0x24)`, `g_act_mission` = `0x0065115c`), **not per-actor**.
- **`pursuit_record`** is heap-allocated by `operator_new(0x80)` in `police_manager_register_actor`.
  The manager owns a `std::vector<pursuit_record*>` at `+0x40/+0x44/+0x48`. One record per tracked
  suspect (the player auto-registers first).
- **Naming policy & what "helper" means:** the struct *type* is `police_manager`; a **helper** is a
  `__thiscall` member function of that class — its `this` (ECX) is a `police_manager*`
  (= `C_game+0x2DA0`). The project names this family `police_manager_*` (matching the
  `s_POLICEMANAGER_*` `.rdata` strings). The **`police_manager_*_for_target`** subfamily additionally
  take a target actor and look up that actor's `pursuit_record` in the records vector (`+0x40`) — so
  by construction every one of them reads `records_begin`/`records_end` and the record's
  `+0x00 target`. `police_manager_*` / `*_for_target` in the tables below therefore denote that whole
  defined family (not one function); the **authoritative enumerated list is the function registry**,
  and the specific members that touch each field are the named readers/writers cited in §2.1 / §3.1.
  The separate `pursuit_group_*` / `pursuit_group_manager_*` officer-squad system operates on its own
  objects (§4).

---

## 2. `police_manager` — embedded at `C_game + 0x2DA0` (single, not per-actor), size 0xB0

```c
typedef struct pursuit_record pursuit_record;  // 0x80 bytes, see §3

typedef struct police_manager {
    /* +0x00 */ uint8_t   enabled;                 // master on/off; all helpers early-out when 0
    /* +0x01 */ uint8_t   _unk_01[0x03];           // padding
    /* +0x04 */ uint32_t  tick_acc_inner;          // accumulator (ms) for the inner ~3 Hz tick
    /* +0x08 */ uint32_t  tick_period_inner;       // inner-tick period (330 ms); also the per-record timer decrement step
    /* +0x0C */ float     speed_threshold_fine;    // speed (m/s) above which speeding = fine (~17)
    /* +0x10 */ float     speed_threshold_arrest;  // speed (m/s) above which speeding = handcuffs (~28)
    /* +0x14 */ uint32_t  tick_acc_outer;          // accumulator (ms) for the outer ~1.1 Hz tick
    /* +0x18 */ uint32_t  tick_period_outer;       // outer-tick period (913 ms)
    /* +0x1C */ uint32_t  tick_acc_visibility;     // accumulator (ms) for the middle ~4.6 Hz visibility tick
    /* +0x20 */ uint32_t  tick_period_visibility;  // middle-tick period (217 ms)
    /* +0x24 */ uint32_t  tick_acc_vestigial;      // accumulator of an unused timer pair
    /* +0x28 */ uint32_t  tick_period_vestigial;   // period (4013 ms) of the unused pair — no reader
    /* +0x2C */ uint32_t  _unk_2c;                  // unused (zeroed at reset)
    /* +0x30 */ uint32_t  _unk_30;                  // unused
    /* +0x34 */ uint32_t  _unused_xx;               // cache of the `policemanager_on xx` arg; never read
    /* +0x38 */ uint8_t   busted_cutscene_lock;      // player busted-cutscene lock (arrest OR fine); 1 while the player is in a capture cutscene, else 0; see §2.4
    /* +0x39 */ uint8_t   vehicle_tier;             // backup vehicle-pool selector (`policemanager_on yy`)
    /* +0x3A */ uint8_t   _unk_3a[0x06];            // padding
    /* +0x40 */ pursuit_record** records_begin;     // std::vector<pursuit_record*> — one per tracked suspect
    /* +0x44 */ pursuit_record** records_end;
    /* +0x48 */ pursuit_record** records_cap;
    /* +0x4C */ uint8_t   _unk_4c[0x04];
    /* +0x50 */ void*     police_vehicles_begin;    // reserve pool: patrol cars (vehicle_controller*)
    /* +0x54 */ void*     police_vehicles_end;
    /* +0x58 */ void*     police_vehicles_cap;
    /* +0x5C */ uint8_t   _unk_5c[0x04];
    /* +0x60 */ void*     spike_strips_begin;       // reserve pool: spike strips (barrier.4ds)
    /* +0x64 */ void*     spike_strips_end;
    /* +0x68 */ void*     spike_strips_cap;
    /* +0x6C */ uint8_t   _unk_6c[0x04];            // padding (helper anchor for the foot_cops triple)
    /* +0x70 */ void*     foot_cops_begin;          // reserve pool: pedestrian cops (npc*)
    /* +0x74 */ void*     foot_cops_end;
    /* +0x78 */ void*     foot_cops_cap;
    /* +0x7C */ uint8_t   _unk_7c[0x04];
    /* +0x80 */ npc**     npc_list_begin;           // std::vector<npc*> — NPCs this manager tracks
    /* +0x84 */ npc**     npc_list_end;
    /* +0x88 */ npc**     npc_list_cap;
    /* +0x8C */ uint8_t   _unk_8c[0x04];
    /* +0x90 */ C_car**   tracked_cars_begin;       // cars the manager must clean up
    /* +0x94 */ C_car**   tracked_cars_end;
    /* +0x98 */ C_car**   tracked_cars_cap;
    /* +0x9C */ uint8_t   _unk_9c[0x04];
    /* +0xA0 */ uint16_t* rndr_arr_b_begin;         // std::vector<uint16_t> — visibility-cell indices
    /* +0xA4 */ uint16_t* rndr_arr_b_end;
    /* +0xA8 */ uint16_t* rndr_arr_b_cap;
    /* +0xAC */ uint8_t   _pad_ac[0x04];            // struct-size padding (total 0xB0)
} police_manager;
```

### 2.1 Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | enabled | H | `police_manager_apply_state`; `police_manager_game_init` (→0) | `police_manager_is_enabled` (dedicated getter); plus the inline `if (*this == 0) return;` early-out idiom shared by the `police_manager_*` family (§1) | 0 = disabled, 1 = enabled. |
| +0x04 | tick_acc_inner | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` (acc += dt) | `police_manager_tick` | Inner tick accumulator. |
| +0x08 | tick_period_inner | H | `police_manager_ctor` (200); `police_manager_game_init` (330) | `police_manager_tick` (overflow threshold + per-record timer step) | 330 ms operational; doubles as the decrement step for 9 per-record timers. |
| +0x0C | speed_threshold_fine | H | `police_manager_set_speed_thresholds`; `police_manager_game_init` (17.0f) | `police_manager_tick` (speeding check: `vehicle+0x688 > +0x0C` → speeding bit / fine) | ~17 m/s. Set via `policemanager_setspeed`. |
| +0x10 | speed_threshold_arrest | H | `police_manager_set_speed_thresholds`; `police_manager_game_init` (28.0f) | `police_manager_tick` (heavy-speeding check: `speed > +0x10` → handcuffs tier) | ~28 m/s. |
| +0x14 | tick_acc_outer | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` | `police_manager_tick` (first statement, unconditional) | Outer tick accumulator. |
| +0x18 | tick_period_outer | H | `police_manager_ctor` (200); `police_manager_game_init` (913) | `police_manager_tick` | 913 ms; handler = `police_manager_npc_list_tick`. |
| +0x1C | tick_acc_visibility | H | `police_manager_ctor` (→0); `police_manager_game_init` (→0); `police_manager_tick` | `police_manager_tick` | Middle tick accumulator (gated; §2.2). |
| +0x20 | tick_period_visibility | H | `police_manager_ctor` (200); `police_manager_game_init` (217) | `police_manager_tick` | 217 ms; handler = visibility/LOS refresh. |
| +0x24 | tick_acc_vestigial | M-vest | `police_manager_ctor` (→0); `police_manager_game_init` (→0) | none | Unused pair (§2.2). |
| +0x28 | tick_period_vestigial | M-vest | `police_manager_ctor` (200); `police_manager_game_init` (4013) | none | ~0.25 Hz if it were read. |
| +0x2C / +0x30 | _unk_2c / _unk_30 | U-vest | `police_manager_ctor` (→0); `police_manager_game_init` (→0) | none | Zeroed pair, no period, no readers. |
| +0x34 | _unused_xx | H | `police_manager_apply_state` | none | `policemanager_on xx` cache; never read. |
| +0x38 | busted_cutscene_lock | H | `police_manager_set_busted_cutscene_lock` (→1 set / →0 clear, in `entity_action_tick` case 0x21 player capture — arrest OR fine); `police_manager_apply_state` (→0 on activate) | `police_manager_target_identity_pending`; `police_manager_tick`; `police_manager_try_dispatch_pursuit_for_target`; `police_manager_record_clear_pursuit_state` | **LIVE** (not dead): 1 while the player is in a busted cutscene → suppresses the searching HUD icon, identity-pending and pursuit re-dispatch; else 0. §2.4. |
| +0x39 | vehicle_tier | H | `police_manager_apply_state` | `police_manager_apply_state` (model-pool select) | Backup-vehicle pool selector. |
| +0x40/44/48 | records_begin/end/cap | H | `police_manager_register_actor` | the `police_manager_*_for_target` record-lookup family (§1; enumerated in the function registry) | `std::vector<pursuit_record*>`. |
| +0x50/54/58 | police_vehicles | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` (free) | `police_manager_apply_state`; `police_manager_deploy_vehicle_from_pool`; `police_manager_vehicle_pool_pop_random` | Stride-8 `(vehicle_controller*, meta)`. |
| +0x60/64/68 | spike_strips | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_temp_spike_strip`; `police_manager_spike_strip_pool_pop` | Stride-8 `(spike_strip*, meta)`. spike_strip = PS2 `C_irenka` (kind 0x1f). |
| +0x70/74/78 | foot_cops | H | `std_vector8_insert_at` (via `police_manager_apply_state`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_deploy_npc_from_pool`; `police_manager_officer_pool_pop_random` | Stride-8 `(npc*, meta)`. **Not** cleared by `police_manager_game_init`. |
| +0x80/84/88 | npc_list | H | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_game_init` (clear) | `police_manager_npc_list_push`; `police_manager_npc_list_erase`; `police_manager_npc_list_tick` | `std::vector<npc*>`; elements non-owning. |
| +0x90/94/98 | tracked_police_cars | H | `police_manager_add_tracked_police_car`; `police_manager_dtor` | `police_manager_apply_state`; `police_manager_game_done`; `police_manager_dtor` | The manager's **tracked police cars**, not a renderer cache: `police_manager_game_done` walks it calling `hud_radar_remove_car` + `game_remove_temporary_actor` on each entry, and the PS2 twin of its writer is `DezoSebralGumyIRadio__10C_shvestkyFP5C_car`, whose parameter is typed `P5C_car`. §7. |
| +0xA0/A4/A8 | rndr_arr_b | H | `police_manager_apply_state` + `police_manager_tick` visibility refresh (via `std_vector2_insert_at`); `police_manager_dtor` | `police_manager_apply_state`; `police_manager_tick` visibility refresh; `police_manager_dtor` | `std::vector<uint16_t>` (stride-2) — visibility-cell indices. |

### 2.2 Per-frame tick scheduler

The 10 scalar fields at `+0x04..+0x30` are **five `(accumulator, period_ms)` pairs** read only by
`police_manager_tick` — three active, two vestigial. Each active pair runs the frame-rate-independent
pattern (so the handler fires at `1000/period` Hz regardless of frame time):

```
acc += dt_ms;  if (period_ms < acc) { acc -= period_ms;  <handler>; }
```

| Pair | acc / period | period | rate | Overflow handler |
|---|---|---|---|---|
| Outer | +0x14 / +0x18 | 913 ms | ~1.1 Hz | `police_manager_npc_list_tick` — distance-cull `npc_list` vs camera. Runs unconditionally. |
| Middle | +0x1C / +0x20 | 217 ms | ~4.6 Hz | Visibility/LOS refresh; sets sighting markers; populates `rndr_arr_b`. Gated on not-busted-cutscene + render quality ≥ 10. |
| Inner | +0x04 / +0x08 | 330 ms | ~3.0 Hz | Per-record state machine: re-derive live offence bits, decay offence timers, escalate `pending → committed`, drive HUD/bar. `+0x08` also = the decrement step for 9 per-record timers. |

The two vestigial pairs (`+0x24/+0x28 = 4013 ms` and `+0x2c/+0x30`) are written at reset but read
nowhere. The four operational periods (330/913/217/4013) appear in the binary only inside
`police_manager_game_init` — no reload-on-zero re-arm sites, confirming accumulator
(not countdown) timers.

### 2.3 Reserve pools, npc_list, renderer arrays, lifecycle

`police_manager` owns seven `std::vector` triples. Three are **pre-loaded reserve pools** for
pursuit assets — populated up-front by `police_manager_apply_state` (script `POLICEMANAGER_ON`)
and consumed on demand mid-pursuit by a deploy trio. This is a ready-reserve cache: mid-chase spawns
incur no i3d-load / `operator_new` (the heavy work is done at `POLICEMANAGER_ON` time).

| Pool | offset | element | populate model pool (`police_manager_apply_state`) | deploy / pop helpers | `police_manager_game_init` clears? |
|---|---|---|---|---|---|
| police_vehicles | +0x50 | `vehicle_controller*` | 4-way pool gated by `vehicle_tier` | `police_manager_deploy_vehicle_from_pool` / `police_manager_vehicle_pool_pop_random` | yes |
| spike_strips | +0x60 | `spike_strip*` (`barrier.4ds`; PS2 `C_irenka`, kind 0x1f) | single type | `police_manager_deploy_temp_spike_strip` / `police_manager_spike_strip_pool_pop` | yes |
| foot_cops | +0x70 | `npc*` | 3-way pool (`pol11/12/13`) | `police_manager_deploy_npc_from_pool` / `police_manager_officer_pool_pop_random` | **no — survives soft reset** |

All three are stride-8 `(object*, metadata)` pairs inserted by `std_vector8_insert_at`. A gameplay
"roadblock" is a *composite* drawn from all three (cars + spike strips + armed foot cops) — see
`police-system/artifacts/police-system.md` "Wanted with Gun". The asymmetry (foot cops survive
`police_manager_game_init`) means after a soft reset, vehicle/spike-strip deploys
return 0 until the script re-issues `POLICEMANAGER_ON`; foot-cop deploys stay available.

- **npc_list** (`+0x80`, `std::vector<npc*>`) — NPCs the manager tracks: officers aggregated from each
  record's officer vector (`record+0x74`), plus script-added slots (`program_process_command` cases
  0x16b/0x171 = erase/push). Maintained by `police_manager_npc_list_push` / `police_manager_npc_list_erase`; distance-culled each
  outer tick by `police_manager_npc_list_tick`. Elements are non-owning (`police_manager_dtor` frees
  the array, not the NPCs).
- **tracked_police_cars** (`+0x90`) — the police cars the manager must clean up (§2.1, §7).
  **rndr_arr_b** (`+0xA0`, `std::vector<uint16_t>`) — the visibility cache; it receives
  visibility-cell indices from `police_manager_apply_state` and the `police_manager_tick` middle-tick
  refresh.

**Lifecycle trio.** The manager is the single instance embedded in the C_game object, so it is built
when its owning object is constructed and freed when that object is destroyed; the runtime receiver
is byte-proven `C_game+0x2DA0` (§1).

- `police_manager_ctor` — cold init (timer budgets 200; all triples zeroed).
- `police_manager_game_init` — soft reset: clears records / police_vehicles /
  spike_strips / npc_list (`end = begin`, buffers kept), reseeds timer budgets to operational values;
  does **not** touch foot_cops or the renderer arrays. Sole caller: `game_init`.
- `police_manager_dtor` — frees all seven triples, each by releasing the backing array and zeroing
  the three pointers. **No triple destroys its elements**: the records vector's only extra step is a
  call to `stub_noop_args8_00575c00`, whose whole body is `return;`, so the heap `pursuit_record`s
  are not freed here.

### 2.4 `+0x38 busted_cutscene_lock` — the player-only suppression lock

The field is **live**. The game sets it to **1** whenever the **player** is captured in a busted
cutscene — **both arrest and fine** — in `entity_action_tick` case 0x21 (via
`police_manager_set_busted_cutscene_lock`, gated on `folded_get_dword_0x10() == 2`, the
`C_actor::GetKind` role of that folded symbol, = player; callsite VA `0x420081` sets 1,
VA `0x421656` clears it), and clears it to 0 on cutscene teardown; `police_manager_apply_state` also
resets it to 0.

While it is 1 it suppresses the searching HUD icon, identity-pending reporting and player-pursuit
re-dispatch. An EXE-patch test that forces the byte to 1 reproduces exactly that: ambient cops ignore
the player and the searching HUD breaks.

The writer in `entity_action_tick` reaches `+0x38` through
`act_mission_get_game → game_get_police_block → set`, i.e. from outside the police files.

**Cross-platform:** the field exists at `C_shvestky+0x38` on PS2 and is read identically by
`NeviDoKterePiceVlezl__10C_shvestky` (the same `record+0x51 && +0x38 == 0` gate).

**Distinct from `police_board+0x2c`** (`busted_cutscene_active`, read by
`police_board_is_busted_cutscene_active`): that is the **any-target** board-side "capture cutscene
animating" flag, set for a player or an NPC perpetrator upstream of the player-kind gate; `+0x38` is
the **player-only** manager-side suppression lock, set inside that gate.

---

## 3. `pursuit_record` — heap, 0x80 bytes

One per registered suspect (`operator_new(0x80)` in `police_manager_register_actor`). The player's is
auto-created on `POLICEMANAGER_ON`. Offence storage / escalation / strings:
**`police-system/artifacts/offences.md`**.

```c
typedef struct pursuit_record {
    /* +0x00 */ actor*    target;                  // the tracked actor (match key)
    /* +0x04 */ void*     linked_vehicle;          // the car the suspect is driving (mirrors actor+0x98)
    /* +0x08 */ uint16_t  offense_bits_a;          // offence bitfield (bit = 1<<offence type id), short-decay lane
    /* +0x0A */ uint16_t  offense_bits_b;          // long-decay mirror of the collision bits (0x08/0x10/0x20)
    /* +0x0C */ uint32_t  car_theft_timer;         // car-theft offence window (ms)
    /* +0x10 */ uint32_t  last_fire_time_ms;       // shooting-offence decay timer
    /* +0x14 */ uint16_t  on_sidewalk_timer;       // sidewalk-offence on-surface accumulator (ms)
    /* +0x18 */ uint32_t  collision_short_timer;   // short (~4 s) collision-bit decay
    /* +0x1C */ uint32_t  last_crash_time_ms;      // long (~20 s) collision-bit decay
    /* +0x20 */ uint32_t  violence_event_timer;    // violence-offence decay
    /* +0x24 */ uint32_t  running_red_or_sign_damage_timer; // shared decay for red-light + traffic-sign bits
    /* +0x28 */ uint32_t  num_of_offences;         // vestigial on PC (never written; reads dead); PS2 PocetPrestupku
    /* +0x2C */ uint8_t   roadblock_eligible_flag; // gun-tier roadblock-eligibility gate
    /* +0x2D */ uint8_t   _unk_2d[0x03];
    /* +0x30 */ uint8_t   _unk_30[0x04];
    /* +0x34 */ float     linked_vehicle_pos_x;    // linked_vehicle world-X at link time
    /* +0x38 */ float     linked_vehicle_pos_y;    // linked_vehicle world-Y at link time
    /* +0x3C */ uint16_t  police_state;            // committed level: 1 fine / 2 handcuffs / 3 gun
    /* +0x3E */ uint16_t  police_state_pending;    // pending level (re-derived each tick from offence bits)
    /* +0x40 */ int32_t   collision_episode_timer; // ~5 s window grouping rapid collisions (-1 = idle)
    /* +0x44 */ uint32_t  wanted_bar_remaining;    // wanted-bar countdown (ms; -1 = not running)
    /* +0x48 */ uint32_t  wanted_bar_max;          // wanted-bar max, state-keyed (20000/40000/60000)
    /* +0x4C */ uint32_t  collision_episode_count; // collisions in the current episode (written, never read)
    /* +0x50 */ uint8_t   wanted_substate_flag;    // "all cops alerted" substate
    /* +0x51 */ uint8_t   identity_pending;        // 1 = searching/UNKNOWN, 0 = KNOWN
    /* +0x52 */ uint8_t   _unk_52[0x02];
    /* +0x54 */ void*     identity_tracked_vehicle;// the vehicle the suspect is identified with
    /* +0x58 */ uint8_t   identity_on_foot_byte;   // on-foot identity flag
    /* +0x59 */ uint8_t   _unk_59[0x03];
    /* +0x5C */ uint32_t  reinforcement_counter;   // backup-wave latch (0 none / 1 handcuffs / 2 gun)
    /* +0x60 */ uint8_t   _unk_60[0x04];
    /* +0x64 */ uint32_t  action_code;             // current offence display/event code (string = 2100+code)
    /* +0x68 */ uint32_t  run_over_count;          // pedestrians run over this episode (>=6 -> gun)
    /* +0x6C */ uint8_t   _unk_6c[0x04];
    /* +0x70 */ uint8_t   identity_byte;           // low byte of target ptr (likely cheap identity fingerprint)
    /* +0x71 */ uint8_t   _unk_71[0x03];
    /* +0x74 */ officer** officers_begin;          // std::vector<officer*> — cops chasing this target
    /* +0x78 */ officer** officers_end;
    /* +0x7C */ officer** officers_cap;
} pursuit_record;                                  // 0x80 bytes
```

### 3.1 Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | target | H | `police_manager_register_actor` | the `police_manager_*_for_target` family's find-loop (`*(int*)record == target`; §1, enumerated in the function registry) | Match key. |
| +0x04 | linked_vehicle | M | `police_manager_register_actor` (if `actor+0x98` is type 4); `police_manager_link_target_to_vehicle` | `police_manager_register_collision_offence_for_target`; `police_manager_register_sidewalk_offence_for_target`; `police_manager_tick` (speed read `*(record+4)+0x688`) | The car the suspect drives; pos captured at +0x34/+0x38. |
| +0x08 | offense_bits_a | H | `police_manager_register_collision_offence_for_target` (0x08/0x10/0x20); `police_manager_register_gunfire_offence_for_target` (0x02); `police_manager_register_violence_offence_for_target` (0x200); `police_manager_register_assault_on_police_officer_for_target` (0x2000); `police_manager_register_assault_on_police_car_for_target` (0x8000); `police_manager_escalate_handcuffs_with_optional_wanted` (0x4000); `police_manager_register_traffic_sign_offence_for_target` (0x1000); `police_manager_set_car_theft_offence_for_target` (0x800); `police_manager_register_sidewalk_offence_for_target` (0x40); `police_manager_tick` (live 0x01 weapon / 0x04 speeding) | `police_manager_get_offense_bits_for_target` | Short-decay lane. `bit = 1<<offence type id`. Full per-bit setter map: `police-system/artifacts/offences.md` §2.1. |
| +0x0A | offense_bits_b | H | `police_manager_register_collision_offence_for_target` (lockstep with bits_a) | `police_manager_get_offense_bits_for_target` (OR with bits_a) | Long-decay (~20 s) collision mirror. `police-system/artifacts/offences.md` §3. |
| +0x0C | car_theft_timer | M | `police_manager_set_car_theft_offence_for_target` (5000/0) | (no reader located) | Car-theft window. |
| +0x10 | last_fire_time_ms | H | `police_manager_register_actor` (-1); `police_manager_register_gunfire_offence_for_target` (+5000, clamp 60000) | `police_manager_get_firetime_for_target` | -1 = never fired. |
| +0x14 | on_sidewalk_timer | H | `police_manager_register_actor` (0); `police_manager_register_sidewalk_offence_for_target`; `police_manager_tick` (grows on surface, clamp 5000) | `police_manager_tick` (sidewalk fine gate) | Tied to bit 0x40. `police-system/artifacts/offences.md` §2.1/§8. |
| +0x18 | collision_short_timer | M | `police_manager_register_collision_offence_for_target` (clamp 4000) | `police_manager_tick` (collision-bit decay) | ~4 s collision window. |
| +0x1C | last_crash_time_ms | H | `police_manager_register_collision_offence_for_target` (clamp 20000) | `police_manager_get_crashtime_for_target`; `police_manager_tick` | ~20 s long collision window. |
| +0x20 | violence_event_timer | M | `police_manager_register_violence_offence_for_target` (+5000, no clamp) | `police_manager_tick` (clears bit 9 on expiry) | Accumulates. |
| +0x24 | running_red_or_sign_damage_timer | M | `police_manager_register_traffic_sign_offence_for_target` (+4000, clamp 20000); `police_manager_tick` red-light block (clamp 10000) | `police_manager_tick` (clears bits 10 + 12 together) | Shared decay for red-light + sign bits. |
| +0x28 | num_of_offences | U-vest | only the gated bump in `police_manager_record_clear_pursuit_state` (rarely hit) | `police_manager_get_num_of_offences`; `police_manager_tick` (`>2` gate) | Effectively always 0 → readers dead. PS2 `PocetPrestupku` returns it as a 1/2/3 severity tier. |
| +0x2C | roadblock_eligible_flag | H | `police_manager_enable_roadblock_flag_for_target` (1 when substate + state ≥ gun); `police_manager_record_clear_pursuit_state` (→0) | `police_manager_tick` (per-cell roadblock scan, sole reader) | Functional roadblock-eligibility gate. |
| +0x34 | linked_vehicle_pos_x | H | `police_manager_link_target_to_vehicle` | (no reader located) | Linked vehicle world-X at link time. |
| +0x38 | linked_vehicle_pos_y | H | `police_manager_link_target_to_vehicle` | (no reader located) | World-Y. Z capture exists but its slot isn't visible in the decomp. |
| +0x3C | police_state | H | `police_manager_record_raise_police_state` (loud raisers); `police_manager_tick` (commit from +0x3e) | `police_manager_get_committed_police_state`; `police_manager_enable_roadblock_flag_for_target`; `police_manager_try_dispatch_pursuit_for_target` (radius select) | Committed icon level. Distinct from the wanted substate. |
| +0x3E | police_state_pending | H | `police_manager_register_collision_offence_for_target`, `police_manager_register_gunfire_offence_for_target`, `police_manager_register_violence_offence_for_target`, `police_manager_register_traffic_sign_offence_for_target` (1/2/3, escalate-only); `police_manager_tick` escalation walk (then zeroed each pass) | `police_manager_tick` (commit + dispatch level); `police_manager_register_collision_offence_for_target` (escalate guard) | Re-derived from current offence bits each inner tick. |
| +0x40 | collision_episode_timer | H | `police_manager_register_actor` (-1); `police_manager_register_collision_offence_for_target` (5000 on first hit) | `police_manager_register_collision_offence_for_target` (`<0` first-vs-repeat); `police_manager_tick` (countdown) | ~5 s collision grouping window. |
| +0x44 | wanted_bar_remaining | H | `police_manager_register_actor` (-1); `police_manager_record_begin_wanted_substate`; `police_manager_tick` (= max / refill / drain) | `police_manager_tick` (bar numerator) | Drains while unseen; `<0` ⇒ clear. |
| +0x48 | wanted_bar_max | H | `police_manager_record_begin_wanted_substate`; `police_manager_tick` | `police_manager_tick` (bar denominator) | State-keyed 20000/40000/60000. |
| +0x4C | collision_episode_count | M | `police_manager_register_collision_offence_for_target`; `police_manager_tick` (→0 on close) | (no reader located) | Written, never read ⇒ dead. |
| +0x50 | wanted_substate_flag | H | `police_manager_register_actor` (0); `police_manager_record_begin_wanted_substate`; `police_manager_escalate_handcuffs_with_optional_wanted` | `police_manager_is_wanted_substate_active_for_target`; `police_manager_tick` (bar + HUD wrap); `police_manager_enable_roadblock_flag_for_target` | "All cops alerted." Not the icon level (+0x3c). |
| +0x51 | identity_pending | H | `police_manager_register_actor` (0); `police_pursuit_group_per_frame_tick`; `police_manager_tick` (HUD-icon recompute, §3.2) | `police_manager_target_identity_pending`; `police_manager_tick` (searching override); `police_manager_try_dispatch_pursuit_for_target` | 1 = UNKNOWN/searching, 0 = KNOWN. |
| +0x54 | identity_tracked_vehicle | H | `police_manager_record_set_tracked_vehicle_for_target` (= `police_action+0x64`) | `police_manager_tick` (in-car identity recompute) | Switching cars unseen → searching. §3.2. |
| +0x58 | identity_on_foot_byte | H | `police_manager_record_set_tracked_vehicle_for_target` (= `police_action+0x68`, set-only) | `police_manager_tick` (on-foot identity recompute) | On-foot identity flag. §3.2. |
| +0x5C | reinforcement_counter | H | `police_manager_register_actor` (0); `police_manager_tick` (1 at handcuffs+wanted, 2 at gun+wanted) | `police_manager_tick` (`<1`/`<2` spawn gates) | Backup-wave latch; gates `police_manager_spawn_patrol_car_reinforcements_at_radius`. |
| +0x64 | action_code | H | `police_manager_register_gunfire_offence_for_target` (1); `police_manager_register_violence_offence_for_target` (5); `police_manager_register_traffic_sign_offence_for_target` (8); `police_manager_register_collision_offence_for_target` (3); `police_manager_tick` escalation (0/2/4/6/7/9/10) | `police_manager_try_dispatch_pursuit_for_target` (string = 2100+code) | `police-system/artifacts/offences.md` §5.2. |
| +0x68 | run_over_count | M | `police_manager_increase_run_over_count_for_target` (+1 per NPC struck); `police_manager_record_clear_pursuit_state` (reset) | `police_manager_register_collision_offence_for_target` (`<6` gate) | `>=6` ⇒ straight to gun. |
| +0x70 | identity_byte | L | `police_manager_register_actor` (= low byte of target ptr) | (no reader located) | Likely a cheap identity fingerprint. |
| +0x74/78/7C | officers_begin/end/cap | H | `police_manager_register_actor` (0); `police_manager_record_add_officer` (dispatch) | `police_manager_count_predators_for_target`; `police_manager_count_officers_visible_for_target`; `police_manager_tick` (officer-count gate, via `vec_count_stride_0x4`) | `std::vector<officer*>` of cops chasing this target; cap inferred. |

### 3.2 Identity recompute — foot vs car (the "searching" swap)

`identity_pending` (+0x51) is what the HUD reads to draw the **searching** icon (searching iff
`identity_pending != 0 && police_manager+0x38 (busted_cutscene_lock) == 0`). `police_manager_tick` recomputes it every inner
tick for the player record, branching on whether any officer currently sees the player
(`police_manager_count_officers_visible_for_target`):

- **An officer sees you** → identity from the pursuit group: `identity_pending = !police_board_target_matches(...)`
  (0 when no group).
- **No officer sees you** (the transparent-icon case) → from disguise, by stance:
  - **On foot** (`actor+0x98 == 0`): `identity_pending = (identity_on_foot_byte == 0)`.
  - **In a car**: `identity_pending = (identity_tracked_vehicle != actor+0x98 current car)`.

Both `+0x54` and `+0x58` are stamped by `police_manager_record_set_tracked_vehicle_for_target` from
the group's `+0x64` / `+0x68`; the on-foot flag is set-only. So: seen-in-a-car then leaving the car
unseen reads `+0x58` (= 0, never set) → searching; switching cars unseen → `+0x54 != current` →
searching. The assault-on-officer exception (leaving the car keeps you KNOWN) is **not** explained by
this recompute — §9.

### 3.3 Raise ↔ clear inverse pair

`police_manager_register_collision_offence_for_target` (escalate) and
`police_manager_record_clear_pursuit_state` (wipe) are the canonical round-trip on a record, both
keyed by target lookup against `police_manager.records`. Every field the registrar *writes* (offence
bits, collision timers, `police_state_pending`, `action_code`, `collision_episode_count`) is reset by
the clear; the clear additionally zeroes fields the registrar only *reads* (`run_over_count`,
`wanted_substate_flag`, `car_theft_timer`, `roadblock_eligible_flag`) and writes the `-1` sentinel to
`collision_episode_timer` / `wanted_bar_remaining`. The registrar fires routinely (every offence);
the clear has two callers (`police_manager_tick`, `entity_action_tick`) — offences accumulate, then
are wiped in bulk on a reset event.

---

## 4. `police_action` — moved

The per-pursuit officer-squad object (`police_action`, 0x6C bytes) is **not** part of `police_manager` —
it belongs to the global `police_board` singleton (`0x006552A8`). Its full struct layout, the
police_manager-style field-evidence table, the squad FSM (`+0x2c` states 0/2/4 vehicle-chase, 1/3/5
on-foot search, 7 disband), the per-squad `police_action_*` function family, and the Cheat Engine table
all live in `structs/police-action.md`.

---

## 5. Cheat Engine address tables (player instance)

`Game.exe` is fixed-base (no ASLR), so `0x0065115C` and `0x006552A8` are literal. **Type** = the CE
value type to pick; pointer/array/padding fields are omitted.

**Pointer-chain roots:** `[[0065115C]+24]` → the **C_game** game-world singleton (NOT the player
actor — `*(g_act_mission+0x24)` is C_game). `police_manager`
is inline at `C_game+0x2DA0`. `pursuit_record[0]` base = `[[[[0065115C]+24]+2DE0]]` (`0x2DE0 = 0x2DA0+0x40`). For
index N: `record[N]` = `[[[[0065115C]+24]+2DE0]+N*4]`. (The `police_action[0]` CE table lives in
`structs/police-action.md` §5.)

### 5.1 `police_manager` — base `C_game + 0x2DA0`

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x00 | enabled | Byte | `[[0065115C]+24]+2DA0` |
| +0x04 | tick_acc_inner | 4 Bytes | `[[0065115C]+24]+2DA4` |
| +0x08 | tick_period_inner | 4 Bytes | `[[0065115C]+24]+2DA8` |
| +0x0C | speed_threshold_fine | Float | `[[0065115C]+24]+2DAC` |
| +0x10 | speed_threshold_arrest | Float | `[[0065115C]+24]+2DB0` |
| +0x14 | tick_acc_outer | 4 Bytes | `[[0065115C]+24]+2DB4` |
| +0x18 | tick_period_outer | 4 Bytes | `[[0065115C]+24]+2DB8` |
| +0x1C | tick_acc_visibility | 4 Bytes | `[[0065115C]+24]+2DBC` |
| +0x20 | tick_period_visibility | 4 Bytes | `[[0065115C]+24]+2DC0` |
| +0x24 | tick_acc_vestigial | 4 Bytes | `[[0065115C]+24]+2DC4` |
| +0x28 | tick_period_vestigial | 4 Bytes | `[[0065115C]+24]+2DC8` |
| +0x34 | _unused_xx | 4 Bytes | `[[0065115C]+24]+2DD4` |
| +0x38 | busted_cutscene_lock | Byte | `[[0065115C]+24]+2DD8` |
| +0x39 | vehicle_tier | Byte | `[[0065115C]+24]+2DD9` |

### 5.2 `pursuit_record[0]` — base `[[[[0065115C]+24]+2DE0]]`

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x08 | offense_bits_a | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+8` |
| +0x0A | offense_bits_b | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+A` |
| +0x0C | car_theft_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+C` |
| +0x10 | last_fire_time_ms | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+10` |
| +0x14 | on_sidewalk_timer | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+14` |
| +0x18 | collision_short_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+18` |
| +0x1C | last_crash_time_ms | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+1C` |
| +0x20 | violence_event_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+20` |
| +0x24 | running_red_or_sign_damage_timer | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+24` |
| +0x28 | num_of_offences (vestigial, always 0) | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+28` |
| +0x2C | roadblock_eligible_flag | Byte | `[[[[0065115C]+24]+2DE0]]+2C` |
| +0x34 | linked_vehicle_pos_x | Float | `[[[[0065115C]+24]+2DE0]]+34` |
| +0x38 | linked_vehicle_pos_y | Float | `[[[[0065115C]+24]+2DE0]]+38` |
| +0x3C | police_state (1 fine / 2 cuffs / 3 gun) | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+3C` |
| +0x3E | police_state_pending | 2 Bytes | `[[[[0065115C]+24]+2DE0]]+3E` |
| +0x40 | collision_episode_timer (-1 sentinel) | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+40` |
| +0x44 | wanted_bar_remaining | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+44` |
| +0x48 | wanted_bar_max | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+48` |
| +0x4C | collision_episode_count | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+4C` |
| +0x50 | wanted_substate_flag | Byte | `[[[[0065115C]+24]+2DE0]]+50` |
| +0x51 | identity_pending | Byte | `[[[[0065115C]+24]+2DE0]]+51` |
| +0x5C | reinforcement_counter | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+5C` |
| +0x64 | action_code | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+64` |
| +0x68 | run_over_count | 4 Bytes | `[[[[0065115C]+24]+2DE0]]+68` |
| +0x70 | identity_byte | Byte | `[[[[0065115C]+24]+2DE0]]+70` |

---

## 6. `police_manager_*` method family — PC ↔ PS2 twins

**PC-first.** This is the PC `police_manager_*` police-manager engine; the PS2 build supplies the
original developer names through its `C_shvestky` class (*švestky* = slang for cops), whose
per-target record sub-struct `S_shvestky` = PC `pursuit_record`. Rows are **sorted by PC VA**;
PS2-only / not-yet-located methods (no PC twin) trail at the end. Cells with no counterpart are left
`-`. Status: **CONFIRMED** (body-diffed, shared fingerprint) · **STRUCTURAL** (role / call-tree /
one-sided — PC and PS2 differ structurally, so many police methods match only at this level) ·
**CANDIDATE** (plausible, unverified) · **PC-only** (a PC helper with no identified PS2 twin) ·
**not located** (no PC twin found, or inlined / PS2-only). Pure lifecycle/STL boilerplate
(compressed_pair / __vector_deleter / insert / swap / clear / data) is omitted.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004b45f0 | `pedestrian_manager_spawn_cop_from_pedestrian` | `FizlosNamakatDoAkce__19C_traffic_generatorFP7C_humanUiRQ23std46vector<P7C_actor,…>bfb` | 0x001972e0 | CONFIRMED | promote an ambient pedestrian into an active cop; a `C_traffic_generator` member the manager drives, not a `police_manager_*` method |
| 0x00504780 | `traffic_car_activate_or_spawn_pursuit_vehicles` | - | - | PC-only | reactivate pooled / spawn up to N=cops/2 pursuit cars from `g_traffic_car_registry` |
| 0x005c8af0 | `police_manager_is_enabled` | - | - | PC-only | getter — returns police_manager+0x00 `enabled` byte |
| 0x005c8b00 | `police_manager_set_speed_thresholds` | - | - | PC-only | set speeding thresholds +0x0c fine / +0x10 arrest (m/s); backs POLICEMANAGER_SETSPEED |
| 0x005cad70 | `police_manager_ctor` | `__ct__10C_shvestkyFv` | 0x001f99b0 | CONFIRMED | ctor — scalar inits + 7 std::vector zero-inits |
| 0x005cae30 | `police_manager_dtor` | `__dt__10C_shvestkyFv` | 0x001f9be0 | CONFIRMED | dtor — reverse teardown of the 7 vectors |
| 0x005caf00 | `police_manager_game_init` | `ChangeSceneInit__10C_shvestkyFv` | 0x001fa3f0 | CANDIDATE | soft reset keeping buffers |
| 0x005caff0 | `police_manager_game_done` | - | - | PC-only | heavy teardown — despawn records/officers, npc_list, renderer arrays, vehicle/spike-strip/foot-cop pools (counterpart to the soft reset) |
| 0x005cb330 | `police_manager_record_free_officers` | - | - | PC-only | dtor sub-step — free a pursuit_record's officer vector (rec+0x74) + zero the triple |
| 0x005cb350 | `police_manager_add_tracked_police_car` | `DezoSebralGumyIRadio__10C_shvestkyFP5C_car` | 0x001fa940 | CONFIRMED | dedup-append of a police car to the manager's tracked-car vector (+0x90) |
| 0x005cb510 | `police_manager_tick` | `Tick__10C_shvestkyFUi` | 0x001faa30 | STRUCTURAL | per-frame manager tick (PS2 SPLIT across Tick + ai_police_nastenka::Tick + ai_police_action::Tick; PC fuses) |
| 0x005cc9e0 | `police_manager_apply_state` | `SetActive__10C_shvestkyFbii` | 0x001fcb20 | STRUCTURAL | activate / tear down police presence (4388 B PS2) |
| 0x005cd960 | `police_manager_register_actor` | `AddPerson__10C_shvestkyFP7C_human` | 0x001fdc50 | CONFIRMED | alloc + insert a pursuit_record (PS2 0x7c / PC 0x80) |
| 0x005cdbd0 | `police_manager_unregister_actor` | `DelPerson__10C_shvestkyFP7C_human` | 0x001fddf0 | CONFIRMED | return officers to pool, free vector, swap-remove |
| 0x005cdca0 | `police_manager_register_gunfire_offence_for_target` | `Firing__10C_shvestkyFP7C_human` | 0x001fdf20 | CONFIRMED | gunfire offence: bit 0x2, tier3, fire timer +=5000 |
| 0x005cdd30 | `police_manager_link_target_to_vehicle` | `DriveCar__10C_shvestkyFP7C_humanP5C_car` | 0x001fe010 | CONFIRMED | link/unlink the ridden vehicle (rec+0x04) |
| 0x005cde10 | `police_manager_register_collision_offence_for_target` | `CarCollision__10C_shvestkyFP5C_carPUif` | 0x001fe120 | CONFIRMED | collision offence: key on rec+0x04, bit 0x8 |
| 0x005ce150 | `police_manager_register_sidewalk_offence_for_target` | `CarIllegalPath__10C_shvestkyFP5C_car` | 0x001fe600 | CONFIRMED | sidewalk/illegal-path offence: bit 0x40 |
| 0x005ce1b0 | `police_manager_register_violence_offence_for_target` | `HumanBoxing__10C_shvestkyFP7C_human` | 0x001fe690 | CONFIRMED | melee offence: bit 0x200, tier2 |
| 0x005ce230 | `police_manager_set_car_theft_offence_for_target` | `CarSteal__10C_shvestkyFP7C_humanbb` | 0x001fe750 | CONFIRMED | car-theft offence: bit 0x800 |
| 0x005ce2b0 | `police_manager_escalate_handcuffs_with_optional_wanted` | `ForceArrest__10C_shvestkyFP7C_humanbb` | 0x001fe800 | CONFIRMED | arrest bit 0x4000; optional wanted substate rec+0x50 |
| 0x005ce320 | `police_manager_register_traffic_sign_offence_for_target` | `DestroySignalMark__10C_shvestkyFP7C_human` | 0x001fe8a0 | STRUCTURAL | traffic-sign offence: bit 0x1000 + tier1 + action8 + timer |
| 0x005ce3a0 | `police_manager_increase_run_over_count_for_target` | - | - | PC-only | +1 run_over_count (rec+0x68) per NPC struck |
| 0x005ce3e0 | `police_manager_report_police_car_stealing_sighting` | `CorujuPoldumMasinu__10C_shvestkyFP7C_humanP5C_car` | 0x001fe9f0 | CONFIRMED | report sighting of stealing of seated police car (§7a) |
| 0x005ce460 | `police_manager_enable_roadblock_flag_for_target` | `SerSeMuDoCesty__10C_shvestkyFP7C_human` | 0x001fead0 | CONFIRMED | set roadblock flag rec+0x2c (gated wanted substate + gun) |
| 0x005ce4c0 | `police_manager_get_offense_bits_for_target` | `GetHumanStatus__10C_shvestkyFP7C_human` | 0x001feb50 | CONFIRMED | return offense_bits_a OR offense_bits_b (rec+0x08 / rec+0x0a) |
| 0x005ce510 | `police_manager_get_committed_police_state` | `JakPicusNaTomJest__10C_shvestkyFP7C_human` | 0x001febb0 | CONFIRMED | return committed tier rec+0x3c |
| 0x005ce560 | `police_manager_get_crashtime_for_target` | `GetCrashTime__10C_shvestkyFP7C_human` | 0x001fec00 | CONFIRMED | return rec+0x1c |
| 0x005ce5a0 | `police_manager_get_firetime_for_target` | `GetFireTime__10C_shvestkyFP7C_human` | 0x001fec50 | CONFIRMED | return rec+0x10 |
| 0x005ce5e0 | `police_manager_count_officers_for_target` | `PocetNasranych__10C_shvestkyFP7C_human` | 0x001feca0 | CONFIRMED | assigned-officer count (rec+0x74) |
| 0x005ce630 | `police_manager_count_officers_alive_for_target` | `PocetNasranychZivych__10C_shvestkyFP7C_human` | 0x001fecf0 | CONFIRMED | count 'alive' officers in the vector |
| 0x005ce6a0 | `police_manager_get_num_of_offences` | `PocetPrestupku__10C_shvestkyFP7C_human` | 0x001fedd0 | CONFIRMED | offence-count/severity tier rec+0x28 (num_of_offences) |
| 0x005ce6e0 | `police_manager_target_identity_pending` | `NeviDoKterePiceVlezl__10C_shvestkyFP7C_human` | 0x001fee20 | CONFIRMED | identity-pending rec+0x51 (gated mgr+0x38==0) |
| 0x005ce730 | `police_manager_is_wanted_substate_active_for_target` | `ChceHoOpichatKazdyPoldaNaOkresku__10C_shvestkyFP7C_human` | 0x001fee90 | CONFIRMED | wanted substate rec+0x50 |
| 0x005ce8d0 | `police_manager_deploy_temp_spike_strip` | - | - | PC-only | deploy a spike strip from the +0x60 pool |
| 0x005ceb00 | `police_manager_spawn_roadblock_in_neighbor_cells` | - | - | PC-only | spawn a roadblock composite across neighbor cells |
| 0x005cf3c0 | `police_manager_acquire_from_pool_by_name` | `MasVolnouPlecku__10C_shvestkyFPc` | 0x001ff0b0 | CONFIRMED | name-keyed pool acquire + reactivate |
| 0x005cf540 | `police_manager_vehicle_pool_pop_random` | `GetTempCar__10C_shvestkyFb` | 0x001ff2c0 | STRUCTURAL | random-pop from the vehicle pool (+0x50) |
| 0x005cf610 | `police_manager_free_temp_car` | `FreeTempCar__10C_shvestkyFP5C_car` | 0x001ff450 | CONFIRMED | release a temp car (dispatch type 4) |
| 0x005cf6b0 | `police_manager_get_free_officers_count` | `GetFreeEnemy__10C_shvestkyFv` | 0x001ff550 | STRUCTURAL | count +4-flagged officer-pool entries |
| 0x005cf6e0 | `police_manager_spike_strip_pool_pop` | - | - | PC-only | random-pop a spike strip from the +0x60 pool |
| 0x005cf720 | `police_manager_free_temp_spike_strip` | `FreeTempIris__10C_shvestkyFP8C_irenka` | 0x001ff5a0 | CONFIRMED | release a temp spike strip (dispatch type 0x1f) |
| 0x005cf790 | `police_manager_officer_pool_pop_random` | `GetTempEnemy__10C_shvestkyFv` | 0x001ff650 | STRUCTURAL | random-pop from the cop/enemy pool |
| 0x005cf810 | `police_manager_free_temp_enemy` | `FreeTempEnemy__10C_shvestkyFP8C_entity` | 0x001ff730 | CONFIRMED | release temp enemy NPC to ambient traffic (dispatch type 0x1b) |
| 0x005cf960 | `police_manager_free_temp_actor` | `FreeMyTempActor__10C_shvestkyFP7C_actor` | 0x001ff970 | CONFIRMED | kind-dispatch 4/0x1b/0x1f -> the three FreeTemp* |
| 0x005cf9a0 | `police_manager_npc_list_push` | `AddDoRiciActor__10C_shvestkyFP7C_actor` | 0x001ff9e0 | STRUCTURAL | unique-append writer for npc_list (+0x80); PS2 adds an actor to the DoRici squad (polymorphic on receiver) |
| 0x005cfb60 | `police_manager_npc_list_erase` | `DelDoRiciActor__10C_shvestkyFP7C_actor` | 0x001ffa70 | STRUCTURAL | erase-swap helper on npc_list (+0x80); PS2 removes an actor from the DoRici squad (polymorphic on receiver) |
| 0x005cfc00 | `police_manager_npc_list_tick` | `TickDoRiciActors__10C_shvestkyFb` | 0x001ffbf0 | STRUCTURAL | per-target assigned-squad maintenance (top-of-tick, mode 0) |
| 0x005cff60 | `police_manager_invalidate_discarded_npcs` | `ZlikvidujVsechnyNasraneVyhozeneCuraky__10C_shvestkyFv` | 0x00200110 | CONFIRMED | purge flagged NPCs from npc_list |
| 0x005d0060 | `police_manager_spawn_patrol_car_reinforcements_at_radius` | - | - | PC-only | spawn a backup patrol-car wave at radius; gated by reinforcement_counter (rec+0x5c) |
| 0x005d06c0 | `police_manager_remove_officer_from_all_records` | `GlobalDelNasrany__10C_shvestkyFP7C_actor` | 0x002009f0 | CONFIRMED | remove the given officer from EVERY pursuit_record's officer list (rec+0x74) |
| 0x005d0700 | `police_manager_add_officer_to_target` | - | - | STRUCTURAL | add a cop to a target's keyed record squad (rec+0x74); the receiver is the manager, and no distinct PS2 twin exists |
| 0x005d0750 | `police_manager_record_add_officer` | `AddNasrany__Q210C_shvestky10S_shvestkyFP7C_actor` | 0x00200a60 | CONFIRMED | unique-append officer to rec+0x74 |
| 0x005d0900 | `police_manager_record_delete_officer` | `DelNasrany__Q210C_shvestky10S_shvestkyFP7C_actor` | 0x00200af0 | CONFIRMED | swap-remove officer from rec+0x74 |
| 0x005d0960 | `police_manager_record_raise_police_state` | `DamMuPokuritJesteVice__10C_shvestkyFP7C_humanb` | 0x00200b50 | CONFIRMED | monotonic raise committed tier rec+0x3c |
| 0x005d09d0 | `police_manager_record_clear_pursuit_state` | `DobreMiPokuril_MuzeJitDom__10C_shvestkyFP7C_humanb` | 0x00200c10 | CONFIRMED | de-escalate/release: zero record fields |
| 0x005d0b00 | `police_manager_try_dispatch_pursuit_for_target` | - | - | PC-only | dispatch cops/cars that pass range+facing+LOS; rolls back the provisional police-state otherwise |
| 0x005d1100 | `police_manager_record_begin_wanted_substate` | `NabonzovalJsemHo__10C_shvestkyFP7C_human` | 0x00201600 | CONFIRMED | begin the wanted substate on a record; shared textdb id `0x898`, shared colour `0xffff0000` and the same 20000/40000/60000 ladder |
| 0x005d11c0 | `police_manager_register_assault_on_police_officer_for_target` | `DalJsemPochuzkariDoDrzky__10C_shvestkyFP7C_human` | 0x00201710 | CONFIRMED | assault-on-cop offence: bit 13 (`+9 \|= 0x20`) → pending GUN (3), action 9 |
| 0x005d1220 | `police_manager_register_assault_on_police_car_for_target` | `StrcilJsemHoPoldumDoVyfuku__10C_shvestkyFP7C_actorP5C_carf` | 0x002017a0 | CONFIRMED | assault-on-cop-car: bit 0x8000, rec ring-rotate |
| 0x005d12d0 | `police_manager_record_set_tracked_vehicle_for_target` | `ZalozSpisATahniDoRiti_Haleluja__10C_shvestkyFP7C_humanP7C_actorb` | 0x002018a0 | CONFIRMED | set rec+0x54 tracked vehicle + rec+0x58 on-foot byte |
| - | - | `GameInit__10C_shvestkyFv` | 0x001f9cf0 | not located | per-scene init (≈ ChangeSceneInit region) |
| - | - | `GameDone__10C_shvestkyFv` | 0x001f9d80 | not located | per-scene teardown |
| - | - | `ChangeSceneDone__10C_shvestkyFv` | 0x001fa8d0 | not located | - |
| - | - | `ChangeSceneVystrc__10C_shvestkyFv` | 0x001fa8e0 | not located | - |
| - | - | `DezoSebralGumyIRadio__10C_shvestkyFP5C_car` | 0x001fa940 | not located | tracked-car ADD (Dezo pair) |
| - | - | `DezoVratilGumyAleRadioSiNechal__10C_shvestkyFP5C_car` | 0x001fa9d0 | not located | tracked-car REMOVE (Dezo pair) |
| - | INLINED | `CreateShvestkyTest__10C_shvestkyFv` | 0x001fb6e0 | not located | escalation-decision block — inlined into police_manager_tick |
| - | INLINED | `TestOverRed__10C_shvestkyFv` | 0x001fc1c0 | not located | red-light DETECTOR — inlined into police_manager_tick; it detects, it does not register the offence |
| - | - | `NextNaSrackySoudruh__10C_shvestkyFP7C_human` | 0x001fe980 | not located | (dev-joke helper) |
| - | - | `CreateCar__10C_shvestkyFR8S_vectorR8S_vectorb` | 0x001feee0 | not located | spawn a pool car (PS2 pool region) |
| - | - | `PocetPolicejnichPlechovekVRici__10C_shvestkyFv` | 0x001ffb40 | not located | count cops in the squad |
| - | - | `DalsiChalpiDoPartie__10C_shvestkyFP7C_humanUif` | 0x00200240 | not located | add another guy to the squad |
| - | - | `VasekSelChlastatNaOslavu__10C_shvestkyFP7C_humanRC8S_vectorfbb` | 0x00200df0 | not located | (dev-joke voice/event setter) |
| - | - | `NabonzovalJsemHo__10C_shvestkyFP7C_human` | 0x00201600 | not located | 'I ratted him out' (voice/event setter) |
| - | - | `IdePoliciePoPlayeroviBoPumper__10C_shvestkyFv` | 0x00201920 | not located | - |
| - | - | `RenamePoliceCar__10C_shvestkyFP5C_car` | 0x002019f0 | not located | - |
| - | - | `GetPolicePosForKokot__10C_shvestkyFP7C_humanR18S_police_interface` | 0x00201aa0 | not located | police-interface position query |
| - | - | `BugZamrdanychTesteruZTepleDve_410__10C_shvestkyFP7C_humaniR18S_police_interface` | 0x00201d60 | not located | (named bug workaround) |
| - | - | `JeToDutolebecniAuto__10C_shvestkyFP5C_carb` | 0x002023a0 | not located | - |

---

## 7. PC vs PS2 — the whole-TU alignment against `C_shvestky`

The PC translation unit is `[0x005cad40, 0x005d14a0)`, bounded by the `.CRT$XCU` initializers at
`0x005cad40` and `0x005d14a0`. It holds 66 PC defs against 65 PS2 `C_shvestky` methods, and the
alignment is 1:1 all the way down once the three `ChangeScene*` methods (InterCity, PS2-port-only)
are dropped.

### 7a. `police_manager_report_police_car_stealing_sighting` — the semantics are the caller's

The body itself tests neither "police car" nor "stealing"; its sole caller does. `player_try_use_car`
(`C_player::Can_UseCar`) guards the call with

```c
if ((*(char *)(car + 0x2044) != ' ') &&                          // the car IS a police car
   (*(char *)(*(int *)(g_act_mission + 0x24) + 0x2da0) != ' '))   // police system enabled
```

`car+0x2044` is the cop-car flag — the same byte `police_manager_npc_list_erase` tests, and the same
one PS2 spells `car+0x2473`. For a single-callsite function the semantics are the caller's guard plus
the callee's effect, so every word of the name is backed.

### 7b. Cross-build tuning differences

* `police_manager_game_init` seeds the two speed thresholds at `+0x0c`/`+0x10` with
  **17.0 / 28.0** on the PC and **27.0 / 1000.0** on the PS2. Every other constant in that
  function (`0x14a`, `0xd9`, `0xfad`, `0x391`) is identical.
* `police_manager_spawn_patrol_car_reinforcements_at_radius` turns the spawn heading by
  **2·pi** on the PC and by **pi** on the PS2, in the same statement slot.
* The PC's radar takes a literal ARGB (`0xffff0000`); the PS2 takes a 3-bit palette index
  into `g_radarLimit`.

### 7c. Methods present on one build only

**PS2 methods with no PC function at their rank:** `DezoVratilGumyAleRadioSiNechal` (the remove half
of the tracked-car list) · `PocetPolicejnichPlechovekVRici` · `CreateShvestkyTest` · `TestOverRed` ·
`GetPolicePosForKokot` · `BugZamrdanychTesteruZTepleDve_410` · `JeToDutolebecniAuto`. None of them is
tagged terminal.

**Terminal PS2-port-only:** `ChangeSceneInit` / `ChangeSceneDone` / `ChangeSceneVystrc` (InterCity).

**PC has, PS2 does not:** `police_manager_deploy_temp_spike_strip`,
`police_manager_deploy_npc_from_pool`, `police_manager_spawn_roadblock_in_neighbor_cells`,
`police_manager_spike_strip_pool_pop`, `police_manager_add_officer_to_target`.

---

## 8. Cross-references

- `police-system/artifacts/offences.md` — offence bit map, escalation, noticing, strings.
- `police-system/artifacts/police-manager.md` — `police_manager_tick` phases + the noticing pipeline.
- `police-system/artifacts/police-system.md` — runtime behaviour, state machine, call chain.
- `structs/police-board.md` + `structs/police-action.md` — the officer-squad system (`police_board`
  board + `police_action` squad).
- `ghidra/Game-ghidra.exe.functions.md` — the function registry.

---

## 9. Open items

| item | state |
|---|---|
| `+0x38 busted_cutscene_lock` (§2.4) | the exact PS2 writer is not pinned, and no reader is known to distinguish arrest from fine |
| assault-on-officer identity exception | leaving the car after assaulting an officer keeps the player KNOWN, unlike other in-car offences; the §3.2 recompute does not explain why `+0x58` stays set on that path. Needs a runtime field-watch |
| `+0x70 identity_byte` | initialised from the low byte of the target pointer; real fingerprint field or constructor quirk is undecided |
| reserve-pool metadata half | the second 4 bytes of each stride-8 pool element (slot id / pool index?) need a runtime breakpoint to confirm |
| the owning-class constructor | the `police_manager_ctor` / `police_manager_dtor` callsites are tagged inside a ctor/dtor pair whose class identity is not yet pinned; the runtime receiver is byte-proven `C_game+0x2DA0` |
| `police_action` (`structs/police-action.md`) | slots `+0x00`, `+0x3C`, `+0x58` unmapped; `anchor_dir (+0x1c)` has a writer but no located reader |
