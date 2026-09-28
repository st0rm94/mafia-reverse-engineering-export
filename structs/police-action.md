# `police_action` — Pursuit-Squad Struct (0x6C bytes)

**Created** 2026-07-08 22:08 +0200 · **Modified** 2026-09-17 17:54 +0200

Type `ai_police_action`. One `police_action` is one officer squad chasing one suspect.
Reconstructed from `Game.exe` disassembly and static analysis of the annotated decomp; no runtime
validation (§10).

**Owner:** the `police_board` singleton (`structs/police-board.md`, VA `0x006552a8`) owns a
contiguous stride-`0x6c` array of these in a `std::vector` whose **object begins at `board+0x10`**
— `+0x10` allocator, `+0x14` begin, `+0x18` end, `+0x1c` capacity (`police_action_vector_count`
reads its argument at `+0x04`/`+0x08`, so the pointer it is handed is `board+0x10`).
`police_board_create_and_engage` allocates one per new chase and stamps it with a session ID from
the board's `+0x20` counter. The squad runs its own state machine in `police_action_tick`, called
once per squad per frame from `police_board_tick`.

**Not a `police_manager` field.** `police_manager` is a **single instance embedded in the `C_game`
singleton at `C_game + 0x2DA0`**, reached as `*(g_act_mission+0x24) + 0x2DA0`
(`structs/police_manager.md` §1). Every `police_manager_*` call inside `police_action_tick`
disassembles to `mov eax,[g_act_mission]; mov eax,[eax+0x24]; lea ecx,[eax+0x2da0]` — the `C_game`
instance, never a per-actor one. The two structs are co-located in the police call chain but
unrelated in memory.

**Confidence tags:** **[H]** disasm/body-read + writer + reader; **[M]** body-read, semantics
partly inferred; **[L]** weak/preliminary; **[U]** never observed.

---

## 1. Field layout (0x6C bytes)

```c
typedef struct police_action {              // 0x6C bytes; element of the police_board's stride-0x6c vector at board+0x10
    /* +0x00 */ uint32_t  _unk_00;              // [U] not observed (session-id slot +0x30 is the stamped ID; this is separate)
    /* +0x04 */ officer** officers_begin;       // [H] std::vector<officer*> — the squad roster pursuing target_npc
    /* +0x08 */ officer** officers_end;         // [H]
    /* +0x0C */ officer** officers_cap;         // [M]
    /* +0x10 */ float     anchor_pos[3];        // [H] +0x10..+0x18 last-sighting world position (the "go to point" anchor)
    /* +0x1C */ float     anchor_dir[3];        // [M] +0x1C..+0x24 last-sighting direction (normalised)
    /* +0x28 */ uint32_t  last_sighting_ts_ms;  // [M] last-sighting timestamp; 0 = never sighted (disband guard pre-on-foot)
    /* +0x2C */ uint32_t  tier;                 // [H] the RESPONSE TIER, not a phase: 0/1 gun, 2/3 fine, 4/5 handcuffs, 6 roadblock, 7 disbanded (§3)
    /* +0x30 */ uint32_t  session_id;           // [M] pursuit-session ID stamped from police_board+0x20; propagates to officers' +0xf54
    /* +0x34 */ uint32_t  on_foot_phase_flag;   // [H] 0 = vehicle phase, 1 = on-foot (exit-vehicle / arrest) phase entered. ALSO the disband tier stash — §3a
    /* +0x38 */ uint32_t  arrest_grace_deadline;// [H] armed ONCE at creation as game_time + 60000 + rand(); once passed, the +0x38 tail runs
    /* +0x3C */ uint32_t  created_ts_ms;        // [H] game time at squad creation; written once, never re-armed — the "squad older than 30 s" gate in tier 1
    /* +0x40 */ uint32_t  los_timestamp_ms;     // [H] last time an officer perceived the target; los_age = game_time - this
    /* +0x44 */ uint32_t  sighting_accum_ms;    // [H] tracking/sighting accumulator (drives the 15s/40s wanted-substate escalations)
    /* +0x48 */ uint32_t  search_rearm_deadline;// [H] game-time deadline gating the next event-0x12 dispatch
    /* +0x4C */ uint32_t  cancel_order_timer_ms;// [H] accumulates frame dt while the chase vehicle is stuck; >10000 ⇒ cancel order / advance FSM
    /* +0x50 */ uint32_t  flags;                // [H] bit0 = active (recomputed each tick); bit1 = redispatch-orders request; bits 0x20|0x40 backup re-request; bit2 = vehicle lost
    /* +0x54 */ uint32_t  backup_rerequest_deadline;// [H] game-time deadline; on pass, re-arms backup request (sets +0x50 0x20|0x40) and pushes +3000ms
    /* +0x58 */ uint32_t  _unk_58;              // [U]
    /* +0x5C */ vehicle*  group_vehicle;        // [H] the squad's chase vehicle (0 when on foot / lost)
    /* +0x60 */ npc*      target_npc;           // [H] the suspect this squad pursues (match key for all _by_target ops)
    /* +0x64 */ vehicle*  tracked_target_vehicle;// [H] the vehicle the target was last seen in (compared to target+0x98)
    /* +0x68 */ uint8_t   on_foot_sighting_flag;// [H] set when the target is being tracked on foot
} police_action;                                // 0x6C bytes (size byte-verified via the board's stride-0x6c iteration)
```

## 2. Field evidence

The squad's own per-frame driver is `police_action_tick`; the sighting helper is
`police_action_tick_sighting_timers`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x04/08/0C | officers_begin/end/cap | H | `police_board_create_and_engage` (bind roster); `police_action_tick` (pop dead officer ¹); `police_board_invalidate_actor` (remove dead npc); `police_action_disband` (`end = begin`) | `police_action_tick`; `police_action_find_officer_index`; `police_action_dispatch_officer_commands`; `police_action_tick_sighting_timers`; `police_board_is_actor_in_pursuit` | `std::vector<officer*>`, stride 4. Officers non-owning. |
| +0x10..0x18 | anchor_pos[3] | H | `police_board_update_sighting` (writes target frame pos) | `police_action_tick` (offset arg of the "go to point" pursuit event in states 1/3) | Last-sighting world position. |
| +0x1C..0x24 | anchor_dir[3] | M | `police_board_update_sighting` (normalised direction) | - | Last-sighting facing; no in-tick reader located. |
| +0x28 | last_sighting_ts_ms | M | `police_board_update_sighting` (bumps timestamp) | `police_action_tick` (`== 0` ⇒ never sighted ⇒ disband when target leaves the tracked vehicle before on-foot) | Timestamp, ms game time. |
| +0x2C | tier | H | see ² | `police_action_tick` (both switches); `police_action_tick_sighting_timers`; `police_action_dispatch_officer_commands` | **Response TIER, not a chase phase** — §3 table. |
| +0x30 | session_id | M | `police_board_create_and_engage` (from `police_board_increment_counter`) | `police_board_find_group_by_session_id`; `entity_spawn_police_response` (stamps officers' +0xf54) | Unique per chase. |
| +0x34 | on_foot_phase_flag | H | see ³ | `police_action_tick` (phase branch in every tier case); `police_action_tick_sighting_timers` (`==0` fast path); `police_action_dispatch_officer_commands` (tier 7 reads the stash back) | 0 = vehicle phase, 1 = on-foot phase — **except on the disband path, §3a**. |
| +0x38 | arrest_grace_deadline | H | `police_board_create_and_engage`, once: `= game_time + 60000 + rand_range(...)` | `police_action_tick` (2nd switch: `+0x38 < game_time` ⇒ `police_manager_enable_roadblock_flag_for_target` + the per-officer `ai_logs_get_log_for_actor` loop) | ~60 s grace with jitter, from squad creation. |
| +0x3C | created_ts_ms | H | `police_board_create_and_engage`, once: `= game_time`; no other writer outside the copy-ctor / assignment helpers | `police_action_tick` tier 1 only (`game_time - +0x3c > 30000` ⇒ escalate the wanted substate) | Never re-armed: the tier-1 gate means "this squad is older than 30 s", **not** "the search has run 30 s". |
| +0x40 | los_timestamp_ms | H | `police_action_tick_sighting_timers` (= game time while target perceivable) | `police_action_tick` (`los_age = game_time - +0x40` gates the 5s/10s/15s/30s/40s officer-recall tiers) | Line-of-sight anchor. |
| +0x44 | sighting_accum_ms | H | `police_action_tick_sighting_timers` (`+= dt` while perceivable) | `police_action_tick` (`>40000` / `>15000` ⇒ `police_manager_record_begin_wanted_substate`) | Tracking accumulator. |
| +0x48 | search_rearm_deadline | H | `police_action_tick` (`= game_time + 3000`; `+= 10000` after a dispatch) | `police_action_tick` (gate before sending officers to a callable device via event 0x12) | Game-time deadline. |
| +0x4C | cancel_order_timer_ms | H | `police_action_tick` (`+= dt` when `car_is_usable(2)` false; `= 0` when true; `= 20000` when channel-1 false) | `police_action_tick` (`>10000` ⇒ cancel vehicle order + advance FSM) | Stuck-vehicle watchdog. |
| +0x50 | flags | H | `police_action_tick` (bit0 recompute via `ai_action_manager_has_action(0x23)`; clear bit1 after dispatch; set `0x20\|0x40` on backup re-request); `police_action_on_vehicle_lost` (set bit2) | `police_action_tick`; `police_board_has_active_group_for_target` (bit0); `police_board_target_matches` | bit0 = active, bit1 = redispatch request, bit2 = vehicle lost. |
| +0x54 | backup_rerequest_deadline | H | `police_action_tick` (`= game_time + 3000` on pass) | `police_action_tick` (`+0x54 < game_time` ⇒ re-arm backup, set flags `0x20\|0x40`) | Game-time deadline. |
| +0x5C | group_vehicle | H | `police_action_tick` (`= 0` on drop); `police_action_on_vehicle_lost` (clear); `police_board_invalidate_actor` (clear if vehicle == npc) | `police_action_tick` (chase-order logic); `police_action_disband` (`!= 0` ⇒ transfer ownership); `police_board_release_all_group_vehicles` | `vehicle*`; 0 = on foot / lost. |
| +0x60 | target_npc | H | `police_board_create_and_engage` | `police_action_tick`; `police_board_target_matches`; `police_board_is_actor_in_pursuit`; `police_board_invalidate_actor`; `police_board_has_active_group_for_target`; `police_board_broadcast_for_target`; `police_board_disband_by_target` | The pursued suspect. |
| +0x64 | tracked_target_vehicle | H | `police_board_refresh_target_vehicle` (`= target+0x98`); `police_board_update_sighting` (force branch) | `police_action_tick` (`== target+0x98` keeps the chase live); `police_action_tick_sighting_timers`; `police_board_target_matches` | Vehicle the target was seen in. |
| +0x68 | on_foot_sighting_flag | H | `police_board_refresh_target_vehicle` (`= 1` when target has no vehicle); `police_board_update_sighting` (force branch) | `police_action_tick`; `police_action_tick_sighting_timers`; `police_board_target_matches` | Set while the target is tracked on foot. |

¹ `stl_copy_dwords_fastcall` shifts the tail down, then `stub_noop_args8_00575c00` (an empty
`ret 8` that does nothing), then `end -= 4`.

² `police_board_create_and_engage` (initial tier from its mode arg); `police_board_raise_gun_escalation`
(**0/1**); `police_board_raise_handcuffs_escalation` (**4/5**); `police_action_tick` +
`police_action_on_vehicle_lost` (even→odd step on vehicle loss); `police_action_disband` (→7);
`police_action_dispatch_officer_commands` (tier 7 restores the stash from +0x34);
`police_board_invalidate_actor`.

³ `police_action_tick` (1 on entering the exit-vehicle/arrest phase; 0 when re-issuing the vehicle
order); `police_action_on_vehicle_lost` (1); `police_action_dispatch_officer_commands` (1);
`police_action_disband` (with-car path: stashes the pre-disband TIER here, §3a).

## 3. Squad tiers (`+0x2c`)

**`+0x2c` encodes the RESPONSE TIER, not a chase phase.** The tier pairs the same way the HUD
police-state tiers do (`record+0x3c`: 1 fine / 2 handcuffs / 3 gun). **Even = the squad is in a
car, odd = on foot**; the even→odd step happens when the car is lost
(`police_action_on_vehicle_lost`, `police_action_tick`'s watchdog).

| tier | in car | on foot | set by |
|---|---|---|---|
| **GUN** | 0 | 1 | `police_board_raise_gun_escalation`; `police_board_create_and_engage` mode 0 |
| **FINE** | 2 | 3 | `police_board_create_and_engage` mode 2 — the caller's default, so a fresh dispatch opens here |
| **HANDCUFFS** | 4 | 5 | `police_board_raise_handcuffs_escalation`; `police_board_create_and_engage` mode 4 |
| **roadblock squad** | - | 6 | `police_board_create_and_engage` mode 6. Never reaches either `police_action_tick` switch |
| **disbanded** | - | 7 | `police_action_disband`, `police_board_invalidate_actor` |

**Three independent witnesses fix the mapping:** (1) `police_board_raise_gun_escalation` writes 0/1
and gates only on `!= 6`; (2) `police_board_raise_handcuffs_escalation` writes 4/5 and refuses when
the tier is 0 or 1, i.e. *handcuffs cannot demote the gun tier*; (3) `police_board_create_and_engage`'s
mode switch maps 0→(1,0), 2→(3,2), 4→(5,4), 6→6, and `police_manager_try_dispatch_pursuit_for_target`
passes mode 2 by default. This is also why tier 2 is the only tier that calls
`police_board_raise_handcuffs_escalation` from inside the tick: fine → handcuffs is the one
escalation the squad can raise on itself.

Every tier case shares the same **officer distance-cull idiom**: resolve a reference frame (the
suspect's body frame at `npc+0x68`, refreshed when the dirty bit `frame[0xac] & 0x20` is clear),
read its world position, then walk the officer roster and `entity_dispatch_event(0, <event>, <arg>)`
on any officer past a squared-metre gate (`400`=20 m, `1600`=40 m, `2500`=50 m, `6400`=80 m,
`22500`=150 m, `40000`=200 m).

| tier | behaviour in `police_action_tick` | officer events |
|---|---|---|
| 0 | gun tier, in car. Escalates the wanted substate after 40 s of accumulated sighting; disbands when the car drifts >150 m from an unseen suspect | 0x11 advance, 0x12 call HQ |
| 1 | gun tier, on foot. Escalates the wanted substate once the squad is older than 30 s (`+0x3c`) | 0xc search, 0x11, 0x12 call HQ |
| 2 | fine tier, in car. **Raises the handcuffs tier** when the suspect's car keeps moving; disbands on >150 m drift | 0x11 advance |
| 3 | fine tier, on foot | 0xc search |
| 4 | handcuffs tier, in car; mirrors tier 0 | 0x11 advance, 0x12 call HQ |
| 5 | handcuffs tier, on foot | 0xc search, 0x11, 0x12 call HQ |
| 7 | disbanded; recalls officers >200 m from the **active camera**, not from the player entity ¹ | 0xc recall |

¹ The reference frame is `(*(g_act_mission+0x10))->+0x17c`, the `I3D_scene`'s camera frame
(`structs/i3d_frame.md`).

**Event 0x12 is "go to a callable device and call HQ", not "go to a search point".** The list walked
by those branches is `C_game+0x1c8/+0x1cc` — the stride-`0xc` callable-device list of emergency
alarms and phone booths built by `game_load_telephones` (`structs/game.md`). The officer picks the
first device within 40 m; §8's spawn-gating chain is the same list.

The chase car is driven by `pursuit_event_dispatch_pursuit_vehicle` with a tier-keyed event mask
(`0x400` gun / `0x800` fine / `0x200` handcuffs) and a timeout. **Every such dispatch first ARMS the
model-level boost** (`pursuit_event_set_change_tables` with `ECX = 1`;
`police-system/artifacts/boosting.md` §6) — the sole disarm in the family is the no-sighting teardown
branch. The on-foot arrest is staged in the **second** `+0x2c` switch (tiers 0/2/4): once the chase
car stops, the squad broadcasts event 0xf (exit vehicle / converge), enters the on-foot phase, and
issues arrest orders (event 0x10) to the officers whose KIND (`+0x10`) is `0x1b` = **`C_entity`, the
scriptable-NPC class** (`structs/entity.md`) — ordinary scripted NPC officers, not a dedicated
"arrest unit" class.

### 3a. Tier 7 is not always terminal — the `+0x34` stash

`police_action_disband`'s **with-car** path writes `+0x34 = +0x2c` *before* setting `+0x2c = 7`, so
the pre-disband tier is stashed in the on-foot-phase slot. `police_action_dispatch_officer_commands`
case 7 reads it straight back (`+0x2c = +0x34`, then `+0x34 = 1`) and broadcasts a final event 0xf.
A squad that disbanded while still holding a car is therefore **revived at its old tier, on foot**,
by the next redispatch request (`flags` bit1). The no-car disband path has no such stash — it calls
`police_board_end_pursuit` and the squad is spliced out for good.

## 4. `police_action_*` function family (per-squad operations)

These take a single `police_action*` as `this` (distinct from the `police_board_*` functions in
`police-board.md` §3, which take the **board** singleton and walk all squads). The `police_board_*`
functions listed here are board-receiver ops that *operate on* a squad.

| VA | Function | Role |
|---|---|---|
| 0x00443d40 | `police_action_tick` | The squad's per-frame driver: prune dead officers, manage the chase car, run the tier FSM (§3), stage arrests. Called from `police_board_tick` ¹ |
| 0x00443d00 | `police_action_broadcast_event` | Fan an officer event (e.g. 0xf converge) to every roster officer except an excluded one |
| 0x00443be0 | `police_action_disband` | Terminate the squad: event 0x11 to all officers (except `exclude_self`), clear the roster, hand off/teardown the vehicle, `police_board_end_pursuit` |
| 0x004466e0 | `police_action_dispatch_officer_commands` | Push the current tier down to each officer as dispatched orders; clears officer flag `+0xf48` bit3 on everyone it touches. **Its tier-7 case un-disbands the squad** (§3a) |
| 0x00446930 | `police_action_on_vehicle_lost` | Clear `group_vehicle (+0x5c)`, set flags bit2 (+0x50), advance the tier (+0x2c) one even→odd step |
| 0x004461d0 | `police_action_find_officer_index` | Index of an officer in the roster, or sentinel `0x29a` (=666) if absent |
| 0x00446200 | `police_action_tick_sighting_timers` | Per-frame refresh of `los_timestamp (+0x40)` + `sighting_accum (+0x44)` from officers' LOS. Distinct from `police_board_update_sighting` (position/direction) |
| 0x00443b60 | `police_action_destroy_range` | Destroy a contiguous run of `police_action` structs (stride 0x6c) |
| 0x00443b90 | `vec_owner_scalar_deleting_dtor` | MSVC scalar deleting destructor for one `police_action` |
| 0x004474a0 | `police_board_end_pursuit` | (board receiver) Splice a squad out of the board's stride-0x6c array, run the per-element teardown, `end -= 0x6c` |
| 0x00447c40 | `police_board_update_sighting` | (board receiver) Write the latest sighting position/direction (`+0x10..+0x24`) + timestamp (+0x28); force branch also writes `+0x64` / `+0x68`. Updates the matching squad |
| 0x00446c00 | `police_board_create_and_engage` | (board receiver) Allocate a 0x6c squad, bump the session counter, bind the roster, dispatch officers into pursuit, fire the siren, register with the board |
| 0x004481c0 | `police_board_refresh_target_vehicle` | (board receiver) Re-sync each matching squad's `+0x64` / `+0x68` to the target's current vehicle status |
| 0x00448150 | `police_board_target_matches` | (board receiver) Predicate: a squad already covers `target` in its current vehicle status (suppresses duplicate creation) |
| 0x00448230 | `police_board_has_recent_sighting` | (board receiver) Predicate: any squad sighted the target within 2000 ms |
| 0x004483d0 | `police_board_is_actor_in_pursuit` | (board receiver) Predicate: npc is a squad's target or roster member (vehicle-related entity types only) |

¹ Its `+0x38` tail calls `ai_logs_get_log_for_actor` once per officer and **discards the result**;
that function is a pure lookup, so the loop has no observable effect.

## 5. Cheat Engine — `police_action[0]` (only while a pursuit is active)

Base `[006552A8+14]` (the board's `+0x14` array begin). For index N: `[006552A8+14]+<N*0x6c>`. If
it reads garbage, the board pointer indirects once more: try `[[006552A8]+14]`.

| Offset | Field | Type | Address |
|---|---|---|---|
| +0x10 | anchor_pos_x | Float | `[006552A8+14]+10` |
| +0x14 | anchor_pos_y | Float | `[006552A8+14]+14` |
| +0x18 | anchor_pos_z | Float | `[006552A8+14]+18` |
| +0x1C | anchor_dir_x | Float | `[006552A8+14]+1C` |
| +0x20 | anchor_dir_y | Float | `[006552A8+14]+20` |
| +0x24 | anchor_dir_z | Float | `[006552A8+14]+24` |
| +0x28 | last_sighting_ts_ms | 4 Bytes | `[006552A8+14]+28` |
| +0x2C | tier (0/1 gun, 2/3 fine, 4/5 cuffs, 6 roadblock, 7 disbanded) | 4 Bytes | `[006552A8+14]+2C` |
| +0x30 | session_id | 4 Bytes | `[006552A8+14]+30` |
| +0x34 | on_foot_phase_flag | 4 Bytes | `[006552A8+14]+34` |
| +0x38 | arrest_grace_deadline | 4 Bytes | `[006552A8+14]+38` |
| +0x3C | created_ts_ms | 4 Bytes | `[006552A8+14]+3C` |
| +0x40 | los_timestamp_ms | 4 Bytes | `[006552A8+14]+40` |
| +0x44 | sighting_accum_ms | 4 Bytes | `[006552A8+14]+44` |
| +0x48 | search_rearm_deadline | 4 Bytes | `[006552A8+14]+48` |
| +0x4C | cancel_order_timer_ms | 4 Bytes | `[006552A8+14]+4C` |
| +0x50 | flags (bit0 = active) | 4 Bytes | `[006552A8+14]+50` |
| +0x54 | backup_rerequest_deadline | 4 Bytes | `[006552A8+14]+54` |
| +0x68 | on_foot_sighting_flag | Byte | `[006552A8+14]+68` |

---

## 6. The element operations — `std::vector<police_action>` machinery

Eight functions implement `std::vector<police_action>`. The instantiation is **exclusive**: a
byte scan of `.text` for `E8` calls and `E9` jumps to all eight finds every reference in
`police_board_create_and_engage` or inside the cluster — zero foreign callers, so none of the
COMDAT-fold exposure that normally forces a mechanical name. The element type is named by the
PS2 container symbols (§7).

| PC VA | PC name | Role |
|---|---|---|
| 0x004490b0 | `police_action_vector_count` | `(end - begin) / 0x6c`, NULL-begin guarded |
| 0x004490e0 | `police_action_vector_insert_at` | `insert(pos, n, value)` — the grow path allocates, copy-constructs the prefix, the n new elements and the suffix, then destroys and frees the old buffer |
| 0x004493a0 | `police_action_uninit_copy_range` | copy-construct a range into raw storage (copies, does not move) |
| 0x004493e0 | `police_action_uninit_fill_n` | n copy-constructions from one value |
| 0x00449450 | `police_action_copy_assign_range` | `std::copy` over already-constructed elements (assignment semantics) |
| 0x00449620 | `police_action_copy_backward_range` | overlap-safe shift-up used by the insert |
| 0x00449800 | `police_action_copy_ctor` | deep-copies the officers vector, then the `+0x10`..`+0x68` dwords |
| 0x00449970 | `police_action_assign` | `operator=`: self-assign guard + three-way vector reuse/grow/shrink |

**Six of the eight are free function templates, not class members.** Read
`police_action_uninit_copy_range` as *"the `police_action` instantiation of `uninit_copy`"*, never
as a method. Only `police_action_copy_ctor` and `police_action_assign` are real members.

---

## 7. PC vs PS2

- **PS2 class** `ai_police_action`; its owner `police_board` is PS2 `ai_police_nastenka`.
- **Object size: PC `0x6c`, PS2 `0x80`.** The officers vector rebases **PC `+0x04` ↔ PS2 `+0x20`**
  (PS2 uses a Metrowerks `std::__vector_imp`, PC an MSVC `std::vector`).
- **The container symbols.** Two PS2 symbols name the element type where the PC decomp cannot:
  - `__ct__16ai_police_actionFRC16ai_police_action` — the copy constructor
  - `reserve__Q23gif28vector<16ai_police_action,8>Fi` — the container, element type in the
    template argument

  PS2's `reserve` calls the copy-ctor and then `__dt__std::__vector_imp(old + 0x20)` in its grow
  loop, exactly where PC calls `police_action_copy_ctor` and `vector_dispose_at_0x04(old + 0x04)`
  — the same two callees at the documented rebase. That closes the identification.
- **Twins:**

  | PC name | PS2 twin | Grade |
  |---|---|---|
  | `police_action_disband` | `HosiKoncime__16ai_police_action` | CONFIRMED |
  | `police_action_broadcast_event` | `EventVsem__16ai_police_action` | CONFIRMED |
  | `police_action_tick` | `Tick__16ai_police_action` | CONFIRMED |
  | `police_action_find_officer_index` | `KolikatejSemSefe__16ai_police_action` | CONFIRMED |
  | `police_action_tick_sighting_timers` | `PanoveCekamVasNaWCSesynchronizujemSiCas__16ai_police_action` | CONFIRMED |
  | `police_action_dispatch_officer_commands` | `TakZnovuDoToho__16ai_police_action` | CONFIRMED |
  | `police_action_on_vehicle_lost` | `ZbavSeAutaSmrdi__16ai_police_action` | CONFIRMED |
  | `police_board_create_and_engage` | `MamProVasAkciHosi__18ai_police_nastenka` | CONFIRMED |
  | `police_board_end_pursuit` | `AkceDokoncenaPlukovniku__18ai_police_nastenka` | CONFIRMED |
  | `police_action_copy_ctor` | `__ct__16ai_police_actionFRC16ai_police_action` | CONFIRMED (ratio 1.33) |
  | `police_action_assign` | `__as__16ai_police_actionFRC16ai_police_action` | CONFIRMED |
  | `police_action_vector_insert_at` | `reserve__Q23gif28vector<16ai_police_action,8>Fi` | STRUCTURAL ¹ |
  | `ai_action_new_police_call` | `NewPoliceCall__17ai_action_manager` | see `structs/entity.md` §6b |

  ¹ PS2's `reserve(n)` is only the grow-and-move core, while PC's function is
  `insert(pos, n, val)` — it *contains* that core plus the position split and the fill. Same
  algorithm, different API boundary, which is also why the size ratio reads 2.35.
- **`police_action_assign`'s ratio 2.18** has the same cause in reverse: PS2 delegates the grow
  to `gif::vector::reserve`; MSVC inlines it.
- **The other five vector helpers have no PS2 twin.** Metrowerks inlined the equivalent loops
  inside `reserve`; MSVC decomposed the same work into separate helpers. A factoring difference,
  not a gap.

---

## 8. Spawn-gating chain for the type-0x28 action (squad-driven)

- `police_action_tick` case 0 fires event 0x12 to a specific officer when:
  1. The squad reaches the 10 s reference-ts mark, AND
  2. A roster officer has `sub_state == 7`, AND
  3. `dist²(officer, target) > 400.0` (20 m officer-to-target gate), AND
  4. A callable device exists within `dist²(officer, device) < 1600.0` (40 m officer-to-device gate).
- Event 0x12 ⇒ `entity_dispatch_event` case 0x12 ⇒ `ai_action_new_police_call(device_idx, 20000, 3, 0, 0)`
  (the type-0x28 `ai_action`, `structs/entity.md` §6b) ⇒ `ai_action_manager_new_job` on the
  officer's `NPC+0x1164` ⇒ the officer latches `sub_state=9`.
- The officer's line-of-sight flag does not gate this chain.

---

## 9. Related

- `structs/police-board.md` — the owning `police_board` singleton (VA 0x006552a8) that holds the
  stride-0x6c array of these squads.
- `structs/police_manager.md` — the `police_manager` embedded at `C_game+0x2DA0`; distinct struct,
  do not conflate.
- `police-system/artifacts/police-system.md` — diagrams / flow descriptions involving squad lifecycle.
- `Game-ghidra.exe.functions.md` — the function registry.

---

## 10. Open items

- **Runtime validation.** The layout is static-analysis only; no field has been watched live.
- `+0x00` and `+0x58` are never observed.
- `+0x1c..+0x24` (`anchor_dir`) has no located reader in the tick.
