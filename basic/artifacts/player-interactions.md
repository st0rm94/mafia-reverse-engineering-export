# Player context interactions & the action menu

**Created** 2026-06-20 16:26 +0200 · **Modified** 2026-10-05 21:40 +0200

How the "use/interact" button works: how the game decides *which* actions are available where the
player is standing, how it shows the choose-action menu when there is more than one, and how the
picked action is executed. Static analysis of `Game-ghidra.exe.annotated.c` + `textdb.json`.

---

## 1. The pipeline at a glance

```
player_ai (VA 0x004cb1b0)        per-actor per-frame virtual (vtable 0x0063c908 slot +0x34)
 └ on foot -> player_do_action_button_on_foot (VA 0x004cbd00)
      ├ using_object_find_near_objects (VA 0x0061bf50)   gather nearby options, SORT by distance
      ├ if >1 option:
      │    gui_action_menu_ctor (VA 0x00579020)   build the choose-action menu (one line per option)
      │    gui_menu_execute_modal(1)            show it, return the picked index
      └ execute the picked action  (the big switch keyed on the action code = textdb id)
```

There are **two distinct lists** involved — do not confuse them:

| Vector | Where | Role | Primitives |
|---|---|---|---|
| **action-suppression set** | `actor + 0x29ec` (`std::vector<int>`) | action ids that are currently **disallowed** (e.g. while carrying a crate); a candidate whose id is in here is dropped | `using_object_add_invalid_item` (0x0061c1e0), `using_object_del_invalid_item` (0x0061c390), `using_object_is_invalid_item` (0x0061c3f0) |
| **gathered option list** | a local array of 0x10-byte entries | the options actually offered this press, **sorted by distance** | built + sorted by `using_object_find_near_objects` |

The suppression set only filters; the menu order is the **distance sort** (§3), not the order of
the `+0x29ec` vector.

---

## 2. Action codes ARE textdb string ids (2000–2017)

The action code is one number that does three jobs at once: it is the `switch` case in the executor,
the menu line's label, and the textdb string id. From `def-files-docs/textdb.json`:

| id (dec / hex) | String | Executor behaviour |
|---|---|---|
| 2000 `0x7d0` | Open door | operate static interactable, mode 0 |
| 2001 `0x7d1` | Close door | operate static interactable, mode 0 |
| 2002 `0x7d2` | Use switch | operate static interactable, mode 0 |
| 2003 `0x7d3` | Get in the front | enter vehicle as **passenger** (front), on foot |
| 2004 `0x7d4` | Get in the back | enter vehicle as **passenger** (back), on foot |
| 2005 `0x7d5` | Get out of the car | handled by the vehicle/exit path, not the foot switch |
| 2006 `0x7d6` | Refuel | no-op in the foot switch |
| 2007 `0x7d7` | Board train | board rail vehicle (mode 1), then stop the actor |
| 2008 `0x7d8` | Exit train | leave rail vehicle (mode 2), only while riding a type-8 object |
| 2009 `0x7d9` | Hijack car | take a type-4 road vehicle (eject driver if present), on foot |
| 2010 `0x7da` | Pick up box | `human_do_box_pick_up(crate)`, then detach crate from world |
| 2011 `0x7db` | Drop box | place carried crate (actor+0x6f8), then `human_do_box_pick_up(0)` |
| 2012 `0x7dc` | Search dead body | on a type-0x1b corpse: build a **sub** action-menu of the body's options |
| 2013 `0x7dd` | Move dead body | start dragging the corpse |
| 2014 `0x7de` | Talk to the pedestrian | `pedestrian_manager_player_talked_to_pedestrian` (the NPC says a line) |
| 2015 `0x7df` | Get in the car as the driver | enter vehicle as **driver**, on foot |
| 2016 `0x7e0` | Board Trolley Car | board rail vehicle (mode 1) |
| 2017 `0x7e1` | Exit Trolley Car | leave rail vehicle (mode 2) |

The executor switch lives in `player_do_action_button_on_foot`. The target object of the chosen
entry is `entry[+0xc]`; the interaction node is `entry`/`iVar3`, and the node's owning entity is
`node+0x1c`. "Get in front" vs "as driver" differ only by the final arg to `human_use_actor`
(0 = passenger seat, 1 = driver seat).

---

## 3. How options are gathered and ordered — `using_object_find_near_objects`

Each press, this walks the candidate interaction-node list near the player and keeps a node iff:
- its action id is **not** in the suppression set (`using_object_is_invalid_item` over `actor+0x29ec`), AND
- it is roughly in front (lateral offset² < 2.25), AND
- the player is within the node's own range (`node+0xc`).

For every kept option it records the **squared distance** to the player, then runs a **selection sort
ascending by distance** — closest option first. So the menu is **nearest-interaction-first**, not
arbitrary.

### 3.1 What happens on an EXACT distance tie (the speedrun question)

The selection sort compares with a strict `<` and starts each pass with the current slot as the
provisional minimum, so it only moves an element ahead of another when it is **strictly** closer.
Therefore:

- **Distances differ at all** → the nearer option wins; order is fully determined by geometry.
- **Distances are EXACTLY equal** → the tie is broken by the options' **pre-sort array order = the
  gather order**, a back-to-front walk of that entity's interaction-node list. That node order is
  fixed at build time, so the tie-break is **deterministic for that car model** — but it follows
  internal node registration order, not anything the player can see or influence.

In game, entering a car from the passenger side ("Get in the front" 2003 vs "Get in the car as the
driver" 2015) **is unpredictable in practice** (user-confirmed: the order changes for the same car
model). The two seats' interaction points sit at nearly equal distance, so a sub-unit difference in
exactly where the player stands flips which is marginally closer, and nobody can re-stand in exactly
the same spot to float precision. Exact ties essentially never occur in play.

- There is **no priority field** and **no stable "driver first" rule** — the order is a pure function
  of the exact distances: deterministic given identical float position, unpredictable in play.
- A deterministic fix would sort by a fixed key (e.g. ascending action id) before display.

---

## 4. The choose-action menu — `gui_action_menu_ctor`

When more than one option survives, the dispatcher allocates a menu object (`operator_new(0x106c)`),
calls `gui_action_menu_ctor`, then `gui_menu_execute_modal(1)` to display it and get the pick.

**The action menu is NOT a `hud_state` HUD element** (it is not one of the `+0x90a4` flag-word
elements in `basic/artifacts/hud_elements.md`). It is a separate **modal menu widget** — a
`GM_Menu` screen subclass whose vtable `0x0063df54` is installed by the ctor, driven by the same
`gui_menu_execute_modal` dispatcher as the ESC/pause menu (see `basic/artifacts/game-loop.md`). So it
lives in the menu/UI layer, not the per-frame HUD-state renderer.

`gui_action_menu_ctor` iterates the sorted options in order and emits one numbered line each,
formatted `"%d - %s"` (`s__d__d`/`s__d___s` format), where the label is
`textdb_get_string_by_id(id)`, `id` = the option's action code (the 2000..2017 strings) for standard
entries, or `(code + 0xdac)` for a secondary class.

### 4.1 The empty-menu-line bug

`gui_action_menu_ctor` emits a line **per option with no empty-string filtering**. If an option's id
has no textdb string, `textdb_get_string_by_id` returns an empty string and the line renders as
`"N - "` — a number with a blank label. So an empty row means an option reached the menu whose label
id has no textdb entry (an action code outside the labelled 2000..2017 set, or an empty textdb slot).
The fix would be to skip entries whose label resolves empty.

---

## 5. The on-foot vs in-car gates

Most vehicle/board actions are gated on `actor+0x98 == 0` (no linked vehicle = on foot). Exit-train
(2008/2017) requires being in a type-8 (rail) linked object. The corpse actions (2012/2013) require
the target entity type to be `0x1b`. The `actor+0xaf4` flag is set to "this entry came from a
multi-option menu" (entry count > 1) on the car-entry paths.

---

## 6. Related: the shot accuracy penalty (actor+0xad0 / hud_state+0x9234)

Separate from interactions but also driven from `player_ai`: `actor+0xad0` is a brief, decaying
**crosshair accuracy penalty** that firing a firearm builds up.

| step | what happens |
|---|---|
| raise | `player_add_shot_accuracy_penalty` (`0x004ccd20`); both call sites are inside the weapon-fire tick `human_shooting` (`0x004a4b70`), right after `police_manager_register_gunfire_offence_for_target` + `inventory_get_in_hand_item_def`, gated `actor == player` ¹ |
| decay | `player_ai` decays it toward 0 each frame on foot (the same block inline) |
| display | both push `intensity*3 + 1` to `hud_set_player_accuracy`, which sizes the HUD element gated by `hud_state+0x90a4` bit `0x20` (renderer reads `+0x9234 * 4 * +0x90c0`) |
| mirror | the raw intensity is also written to `C_game+0xd8` (`*(g_act_mission+0x24) + 0xd8`) by both writers; nothing reads it: of the 353 instructions in `.text` with displacement `0xd8`, the only two whose base is the `C_game` are these two stores |

¹ The kick is per weapon (`weapon_record+0x84`), clamped to 1.0, ×0.1 in world mode 5, and arms a
3000 ms window at `actor+0xad4`.

**In-game (user-confirmed):** the effect shows **only on foot with a firearm** — not melee, not when
hit by a vehicle, not in a car. That matches the firearm + on-foot gating above: the trigger is
**firing** a firearm, not being hit.

---

## 7. Cross-references
- `def-files-docs/textdb.json` — the 2000..2017 action-label strings.
- `basic/artifacts/hud_elements.md` — the `hud_state` HUD elements (the action menu is NOT one of them).
- `basic/artifacts/game-loop.md` — `gui_menu_execute_modal` / the per-frame actor update.
- `basic/artifacts/npc-player-interactions.md` — the "Talk to the pedestrian" (2014) voice path.

## 8. Open items

Nothing is open.
