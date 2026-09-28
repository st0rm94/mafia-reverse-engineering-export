# Freeride Score System

**Created** 2026-05-31 19:59 +0200 · **Modified** 2026-09-17 13:47 +0200

The Freeride score (money) system is a 4-command cluster in `program_process_command` (cases
0x166–0x169). It manages a per-game integer score displayed on the HUD.

## 1. Key fields

The score lives on the `C_game` object (`act_mission_get_game()` return value):

| Offset | Type | Purpose |
|--------|------|---------|
| `+0x363e` | char | Score display enabled flag (0/1) |
| `+0x3640` | int | Current score value |

HUD rendering state lives on the HUD singleton `hud_state` (BSS VA `0x00658330`):

| Offset | Type | Purpose |
|--------|------|---------|
| `+0x9690` | int | Cached score value for display |
| `+0x9694` | char[] | Formatted score string buffer |

## 2. Script commands

| Case | Command | Arg | Behaviour |
|------|---------|-----|-----------|
| 0x166 | `FREERIDE_SCOREON` | int (0/1) | Enable/disable score HUD; writes `+0x363e` |
| 0x167 | `FREERIDE_SCORESET` | float→int | Set score to absolute value; writes `+0x3640` |
| 0x168 | `FREERIDE_SCOREGET` | out float var | Read `+0x3640`, cast to float, store in script var |
| 0x169 | `FREERIDE_SCOREADD` | float→int | Add value to current `+0x3640` |

Cases 0x167 and 0x169 share a common tail at `LAB_005c69b9` which calls the setter and
conditionally refreshes the HUD text.

## 3. Functions

| VA | Name | Role |
|----|------|------|
| 0x00425580 | `game_freeride_score_get` | __fastcall getter for `+0x3640` |
| 0x005f5bf0 | `game_freeride_score_set_on` | __thiscall; writes char to `+0x363e` |
| 0x005f5c30 | `game_freeride_score_set` | __thiscall; writes int to `+0x3640`, refreshes HUD if display enabled |
| 0x0054d130 | `hud_freeride_update_score_text` | __thiscall; stores value at `+0x9690`, formats into `+0x9694` via `crt_sprintf`; `0x7fffffff` clears the string |

## 4. HUD update flow

1. `game_freeride_score_set` writes `+0x3640`
2. If `+0x363e` != 0, calls `hud_freeride_update_score_text` with the new value
3. `hud_freeride_update_score_text` fetches the format string via `textdb_get_string_by_id(0xb3)`
   (`"$%d"`), formats the score into `+0x9694`
4. When called with `0x7fffffff`, the display string is cleared (score hidden)

## 5. Relationship to police_manager

The score fields (`+0x363e`, `+0x3640`) are **not** part of `police_manager` (which lives at
`C_game+0x2da0` and spans ~0xA8 bytes). They reside at much higher offsets on the same `C_game`
object — a separate subsystem with no structural overlap with the police/pursuit structs documented
in `police-system/artifacts/police-manager.md`. `game_freeride_score_set` writes Freeride money, not
police state.

## 6. The $1000 "caught" deduction

Substate 0xc of `entity_action_tick` case 0x21 (arrest / fine cutscene sequencer) calls
`game_freeride_score_set(game_freeride_score_get() - 1000)` when the cutscene targets the player.

The deduction is gated by **ENTITY-mode** (`folded_get_dword_0x10() == 2`, the `C_actor::GetKind`
role of that folded symbol = the player is the target), **not** by the arrest-vs-fine choice — so it
fires for **both** the **arrest** and the **fine** of the player (NPC-on-NPC arrests carry no
freeride score and never reach it). User-verified:

- **Fine** (player): $1000 deducted **and visible** — normal HUD score update.
- **Arrest** (player): $1000 **still deducted** (the code runs), but the player is dropped into the
  **game-over layout** (triggered by a *separate* path, not case 0x21), so the reduced score is never
  shown. Shown by the known "delete save files ⇒ resume after game-over" bug: on resume the score is
  already $1000 lower despite the arrest.

Net effect in normal gameplay: the $1000 cost is **only observed after a fine**.

## 7. Evidence level

**Tier-2 (Hypothesis)** for the script commands — case body logic matches the Mafia Script Data
Reference descriptions, argument patterns match shipped script usage, and commands form a
sequential 4-opcode cluster. No string literals inside the case bodies provide a Tier-1 anchor.
