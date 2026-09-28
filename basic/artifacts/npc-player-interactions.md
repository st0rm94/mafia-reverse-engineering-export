# NPC ↔ player voice interactions (greeting / brush-off)

**Created** 2026-06-20 14:42 +0200 · **Modified** 2026-09-17 13:43 +0200

How ambient NPCs vocally react to the player: the **interaction-button response** (live) and the
**proactive greeting** (built but dead). Static analysis of the decomp, cross-checked against
in-game observation by the user.

## 1. The two NPC voice fields

Each ambient NPC (the `C_traffic_element` record that `pedestrian_tick`, VA `0x004b7520`, drives —
there are no zones; see `structs/pedestrian.md`) has two relevant flag bytes:

| Field | Name | Meaning |
|---|---|---|
| `npc+0x1fc` | voice-enabled | NPC is allowed to speak (civilian/gang; set per the `back1` model archetype) |
| `npc+0x1fd` | **player-acknowledged** | NPC has greeted the player; switches the interaction response from brush-off to greeting. **One writer:** `pedestrian_greet_player` (§4) |

The **voice bank** that decides which speaker is heard is `npc+0x1f6` (the column into
`tables\voices.def`); it is gender/role-derived: the archetype carried by the `back1`
model selects the gender/role, and that selects the voice-bank column.

## 2. Interaction-button response — `pedestrian_on_player_talk` (VA `0x004be050`) — LIVE

Fires when the player **uses/interacts** with an NPC. Call graph:

```
player presses USE/interact near an NPC
 └ player_do_action_button_on_foot (VA 0x004cbd00)   case 0x7de
    └ pedestrian_manager_player_talked_to_pedestrian (VA 0x004b3080)
       ├ pedestrian_manager_get_pedestrian_from_frame -> the aimed-at NPC; if found:
       └ pedestrian_on_player_talk(npc)
```

It branches on `npc+0x1fd`:

| `npc+0x1fd` | voices.def rows | pick | meaning | in-game |
|---|---|---|---|---|
| `!= 0` | **168–173** | `rand()%6` | **GREETING** — friendly, acknowledges the player | never heard |
| `== 0` (default) | **184–190** | `rand()%7` | **BRUSH-OFF** — "leave me alone / I don't have time" | this is what you hear |

The brush-off path also clears `npc+0x1fc`. The sound is emitted at the NPC origin `npc+0x10` via
`pedestrian_manager_play_pedestrian_scream` (VA `0x004b58c0`).

Because `npc+0x1fd` is never set in shipped play (§4), **only the brush-off ever plays**; the
greeting branch is reachable code on a flag that never gets set.

## 3. voices.def line groups used

`pedestrian_manager_play_pedestrian_scream` selects `voices.def[row][col = npc+0x1f6]`. The civilian
voice-line bank is contiguous — `row = line# + 158`, ids `…940010`–`…940031`, each available in 6
speaker voices (cols 26–28 male / 29–31 female):

| voices.def rows | line ids (col 26 / male #1) | used by | content (user-identified) | in-game |
|---|---|---|---|---|
| 168–173 | `27940010`…`27940015` | greeting branch of §2 + `pedestrian_greet_player` (§4) | greeting, knows the player's name | never heard |
| 174–176 | `27940016`…`27940018` | the dead police-call bark (`police-system/artifacts/offences.md` §5.4) | "call the police, a crime happened" | never heard |
| 184–190 | `27940025`…`27940031` | brush-off (§2) | "leave me alone / I don't have time" | heard on the interact button |

Three adjacent line groups exist in the data; only the brush-off group is reachable in shipped play.

## 4. Proactive greeting — `pedestrian_greet_player` (VA `0x004be0d0`) — DEAD

The **only** writer of `npc+0x1fd`, i.e. the only thing that would unlock the greeting branch of
§2. Called periodically from the ambient tick:

```
pedestrian_tick (per frame)
 └ every ~1.5 s, behind a rand() gate (npc+0x1f8 countdown):
    └ pedestrian_greet_player(npc)
```

When its gate passes it sets `npc+0x1fd = 1`, plays a greeting line (rows 168–173), and head-tracks
the player (frame vtable `+0x38` with the `"neck"` bone string `s_neck`, VA `0x00646540`).

**The gate (all must hold):**
- `npc+0x1fc` voice-enabled; `npc+0x19e == 0` (wander mode);
- player exists and is in the **same scene sector** (`frame+0x120` match);
- **police are not chasing the player** — `police_manager_count_officers_for_target(player) == 0`
  (early return if non-zero);
- **player is "engaged"** — `police_board_is_actor_in_pursuit(player) != 0`;
- player **on foot** (`player+0x98 == 0`);
- within ~5 units (`dist² < 25`) and facing it (`S_vector::AngleTo < 2.0`).

**Why it is dead:** the gate requires the player to be **"engaged"** (in a pursuit group — mixed up
in a chase) **and** to have **no officers** chasing him at the same time. Those two states do not
co-occur in normal play, so the gate never passes, `npc+0x1fd` is never set, and the greeting is
never heard — matching the user's in-game test. Same shape as the dead police-call path
(`police-system/artifacts/offences.md` §5.4): a fully built NPC reaction whose trigger conditions
are mutually exclusive. Cut / unfinished content.

## 5. Reviving it

Two byte patches make both halves observable, each reversible by restoring the original bytes:

| patch | bytes | effect |
|---|---|---|
| swap | the `je` at `0x004be06f` in `pedestrian_on_player_talk` → `jne` | the default interaction plays the **greeting** instead of the brush-off |
| greeting | neutralise the officer-count early return at `0x004be13c` and the "engaged" requirement at `0x004be14f` in `pedestrian_greet_player` | the greeting fires for any nearby on-foot player not in an active chase, setting `npc+0x1fd` |

## 6. Cross-references

- `police-system/artifacts/offences.md` §5.4 — the sibling dead NPC police-call path (and its own
  revive patch).

## 7. Open items

| item | state |
|---|---|
| in-game effect of the §5 patches | untested |
