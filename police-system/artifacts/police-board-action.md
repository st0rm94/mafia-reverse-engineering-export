# police_board_tick / police_action_tick — the pursuit-squad tick

**Created** 2026-07-10 22:15 +0200 · **Modified** 2026-09-17 13:44 +0200

Companion to `police-system/artifacts/police-manager.md` (the player-side `police_manager` engine);
this doc covers the **squad-side** per-frame engine: how the tick propagates method to method, and
the fine → handcuffs escalation. Struct fields per `structs/police-board.md` (the board singleton)
and `structs/police-action.md` (one squad).

## 1. Tick propagation

Each frame the game runs two parallel police engines (see `police-system/artifacts/police-manager.md`).
This is the squad side.

**`police_board_tick(board, dt_ms)`** — VA `0x004477b0`; `this` = pursuit board singleton
`0x006552a8`.
1. `squad_count = (board+0x18 − board+0x14) / 0x6c` — the `police_action` squad array (stride `0x6c`).
2. Calls **`police_action_tick(dt_ms)`** once per squad (squad pointer passed in ECX).
3. Decrements the arrest-settle grace timer `board+0x24` by `dt_ms` (floored at 0).

**`police_action_tick(squad, dt_ms)`** — VA `0x00443d40`. Per-squad FSM on `squad+0x2c`
(`group_state`). Each tick, in order:
1. **`police_action_tick_sighting_timers(dt_ms)`** — refreshes the last-seen timestamp `squad+0x40`
   and adds `dt_ms` to the sighting accumulator `squad+0x44` while the target is perceivable.
2. **`group_state` switch (cases 0..7)** — distance-gated officer event dispatch, escalations
   (§2), chase (re)dispatch via the `pursuit_event_*` family, and self-disband
   (`police_board_end_pursuit`).

Base chain: `game tick → police_board_tick → police_action_tick → {sighting_timers, state FSM}`.

## 2. Escalation: fine → handcuffs

Inside `police_action_tick` (vehicle-chase states 0/2/4), when the target has been sighted long
enough and is still fleeing by car.

Gate (all must hold):
- `squad+0x44` (sighting_accum_ms) `> 15000` — target continuously tracked > 15 s.
- Target is in a vehicle (`target+0x98 != 0`).
- Vehicle is moving **fast**: car (kind 4, `C_car`) speed `car+0x688 > 5`, or rail vehicle
  (kind 8, `C_railway` — tram or elevated train) speed `railway+0xd4 > 4`. (Entity kind =
  `entity+0x10`; full kind enum in `structs/entity.md`.)

⇒ **`police_board_raise_handcuffs_escalation(target)`** — monotonically raises the committed
`police_state` (`pursuit_record+0x3c`) **fine (1) → handcuffs (2)** (see
`police-system/artifacts/police-manager.md` §1; never de-escalates).

Notes:
- The accumulator `+0x44` is fed only by step-1 `police_action_tick_sighting_timers`; if the
  target stays slow/stopped or is lost, the gate never trips.
- A slow/stopped vehicle (speed ≤ 5 / motion ≤ 4) does **not** escalate — a fleeing driver gets
  cuffed, a surrendering one does not.

## 3. PC vs PS2

| PC name | PC VA | PS2 twin |
|---|---|---|
| `police_board_tick` | `0x004477b0` | `Tick__18ai_police_nastenka` |

## 4. Open items

| item | state |
|---|---|
| per-state detail of the `group_state` switch (cases 0..7) | not documented; only the escalation arm (§2) is written up |
