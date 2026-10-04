# `C_door` — the hinged door (kind `6`)

**Created** 2026-08-13 12:57 +0200 · **Modified** 2026-10-01 14:23 +0200

PC prefix **`door_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`6`** |
| PC vtable | `.rdata 0x0063c180` (39 slots) |
| PC constructor | `door_ctor` @ `0x004800f0` |
| PC instance size | `0x1b8` (entity factory `case 6`) |
| PC translation unit | `[0x004800f0, 0x00482bd0)` — 22 function starts, all named |
| Base class | **`C_actor`** |
| PS2 class | `C_door`, 23 methods, 21 mapped, 2 PS2-port-only (§7) |

## 1. What it is

A door leaf on a hinge, driven by a single angle. `door_set_open_angle` places the leaf at a
fraction of its maximum swing and rebuilds the collision proxy from it; `door_ai` ramps that
fraction toward whatever the current state wants; and `door_set_state` is the public verb
everything else calls — the AI walking a path, a script command, the player's use-action, the
door's own init, and the network receiver.

Two things make it more than a rotating prop: it registers itself in `C_game`'s using-object list
so the player action menu can offer it, and it links to a **twin leaf** so double doors swing
together from one call.

## 2. The sounds

`door_ctor` copies three sound file names into every instance before anything else happens:

| offset | default | when it plays |
|---|---|---|
| `+0x130` | `d_woodop.wav` | state 2 (opening) on an unlocked door |
| `+0x140` | `d_woodcl.wav` | state 3 (closing) |
| `+0x150` | `d_woodlo.wav` | state 2 refused because the door is locked — the rattle |

The order is fixed by four artifacts: the ctor's copy order; `door_set_state`'s state-3 arm, which
plays `+0x140`; the locked refusal (§4c), the only reader of `+0x150`; and the in-game rattle on a
locked door. The file names agree: *lo* = locked, *cl* = closed.

The names are overridable per instance — `door_save_data` / `door_load_data` persist all three and
`door_duplicate_from` copies them — so a level designer can hang a metal or glass door on the same
class.

## 3. `door_set_state` — the public verb

Five parameters: `(this, state, actor, force, walk_up)`.

1. **Linked-leaf walk-up.** With `walk_up` set, follow `+0x118` to the outermost leaf and act on
   that one instead; step 5 pushes the same state down `+0x114`. One call, both leaves.
2. **State 2 — open. The lock gate comes first.** If `+0x125` (the `door_lock` flag) is set, it
   plays the `+0x150` rattle and **returns** before the `+0x70 = state` write at the bottom of the
   function. The door does not move, and every caller is refused alike (§4c). Otherwise it plays the
   open sound behind four gates (mission sound count > 0, flag bit `0x8` clear, non-empty name
   buffer, sound master on), then a **1000 ms** re-trigger window on `+0x160`, then the queue call
   with the literal quartet **4.0 / 15.0 / 1.0 / 0.001** (volume, range, pitch, rolloff).
3. **Hinge geometry.** When `+0x11d` enables it and an actor was supplied, takes the angle between
   the hinge offset at `+0x168` and the direction to that actor and writes
   `+0x11e = (angle > π/2)`. That is the **mirror flag** `door_set_open_angle` reads — so *a door
   opens away from whoever opened it*.
4. **State 3 — closing.** Plays `+0x140` (`d_woodcl.wav`) and resets the hinge point. Not gated by
   the lock: a locked door can still be closed.
5. **Linked-leaf push-down** into `+0x114`.
6. **Replication** (§5). Builds the door-state event message, type **`0x34`**, and posts it.

## 4. Field layout

```c
typedef struct C_door {                   /* 0x1b8 bytes */
    /* +0x000 */ C_actor  base;           // [H] kind 6 at +0x10, frame at +0x68
    /* +0x070 */ int      state;          // [H] 0 resting OPEN, 1 resting CLOSED,
                                          //     2 opening, 3 closing (see 4a).
                                          //     door_set_state writes 2/3; door_ai settles 0/1.
    /* +0x074 */ float    angle;          // [H] current = fraction x max_angle
    /* +0x078 */ tDynamicCollObject coll; // [H] embedded; erased on state 2 / GameDone
    /* +0x110 */ uint     flags;          // [H] see the flag table below
    /* +0x114 */ C_door  *leaf_next;      // [H] the other leaf, pushed to
    /* +0x118 */ C_door  *leaf_prev;      // [H] the other leaf, walked up from
    /* +0x11d */ bool     hinge_enabled;  // [H] gates the step-3 geometry
    /* +0x11e */ bool     mirror;         // [H] negates the hinge axis; from flags bit 0x10
    /* +0x120 */ float    max_angle;      // [H] ctor seeds pi/2 (0x3fc90fdb)
    /* +0x124 */ bool     start_open;     // [M] door_game_init: set => door_set_state(2) and
                                          //     rest open at max_angle; clear => rest closed
    /* +0x125 */ bool     locked;         // [H] set by door_lock (case 0x26); read only by
                                          //     door_set_state, which refuses state 2 (4c)
    /* +0x128 */ float    open_speed;     // [H] door_ai: angle += dt * this while state 2
    /* +0x12c */ float    close_speed;    // [H] door_ai: angle -= dt * this while state 3
    /* +0x130 */ char     snd_open[0x10];   // [H] default `d_woodop.wav`
    /* +0x140 */ char     snd_close[0x10];  // [H] default `d_woodcl.wav`
    /* +0x150 */ char     snd_locked[0x10]; // [H] default `d_woodlo.wav`
    /* +0x160 */ uint     last_sound_ms;  // [H] the 1000 ms re-trigger gate
    /* +0x164 */ bool     hinge_locked;   // [H] skips the hinge push in step 3
    /* +0x168 */ S_vector hinge_offset;   // [H] local-space, drives the mirror test
    /* +0x174 */ S_matrix hinge_matrix;   // [H] cached by GameInit, used by GetBoundPoints2D
    /* +0x180 */ S_using_object uo;       // [H] the player action-menu record
} C_door;
```

Confidence: **[H]** measured on both builds · **[M]** measured on PC only.

### The `+0x110` flag word

| bit | meaning | evidence |
|---|---|---|
| `0x2` | **no dynamic collision** — skip the DynErase and the proxy copy | `door_game_done`, `door_on_act_state_change`, `door_set_open_angle` |
| `0x4` | **never usable** — do not register a using-object | `door_game_done`, `door_enable_using_object` |
| `0x8` | **muted** — suppress all three sounds | `door_set_state` |
| `0x10` | persisted into `+0x11e` (mirror) at GameDone | `door_game_done` |
| `0x20` | **locked** — persisted into `+0x125` at GameDone | `door_game_done` |
| `0x40` | **currently in C_game's using-object list** | `door_enable_using_object` |

### 4a. `door_ai` — the state machine

`door_ai` is two arms on `+0x70` and **returns immediately for any value other than 2 or 3**, so a
door at rest costs nothing per frame:

| `+0x70` | meaning | what `door_ai` does |
|---:|---|---|
| **0** | resting fully **OPEN** | nothing (early return) |
| **1** | resting fully **CLOSED** | nothing (early return) |
| **2** | **opening** | `angle += dt * open_speed`; at `max_angle` clamps, sets `+0x70 = 0`, `actor_set_act_state(1)`, writes **2001** to `+0x1ac` ¹ |
| **3** | **closing** | `angle -= dt * close_speed`; at 0 clamps, sets `+0x70 = 1`, `actor_set_act_state(1)`, writes **2000** to `+0x1ac` ¹ |

¹ Skipped when flag `0x4` (never usable) is set.

Both arms then rebuild the leaf's orientation from `+0x11e` (mirror), refresh the world matrix,
cache the inverse into `+0xc0` and the transformed bound centre into `+0x88`, and finish with
`collision_dyn_update(+0x78)` — unless flag `0x2` (no dynamic collision) cut them short.

What this establishes:

1. **`state` has four values.** 2 and 3 are the requests `door_set_state` writes; 0 and 1 are the
   resting values only `door_ai` writes. The script command `door_getstate xx, yy` (dispatcher case
   `0x162`, gated `GetKind == 6`) returns this field; the Mafia Script Data Reference documents only
   **0 open / 1 closed**, so a script polling a door mid-swing observes 2 or 3, which the reference
   does not cover.
2. **`+0x128` / `+0x12c` are the angular speeds** — open and close, in radians per unit time against
   `max_angle`'s `pi/2`. They are duplicated and saved because they are designer-set per door.
3. **`+0x1ac` is inside the embedded `S_using_object`** (`uo` at `+0x180`, so `+0x2c` into it — the
   action code, `structs/using_object.md` §1a). The door **swaps its own action-menu prompt as it
   settles**: 2001 once fully open, 2000 once fully closed. That is how the Action Button offers
   "close" on an open door without any other code running.

### 4b. `door_get_state` — the `+0x70` reader, and the two AI actions that use it

`door_get_state` (`0x00425700`) is the whole of `mov eax,[ecx+0x70]; ret`.

**Why the class is settled.** Every referrer channel was measured, so the referrer set is closed:

| channel | method | result |
|---|---|---|
| body uniqueness | byte-pattern search of `.text` for the whole body | `8b4170c3`, **1** site image-wide |
| `E8` callsites | byte scan of `.text` for relative calls to the address | **4** |
| `.rdata`/`.data` datarefs | scan of the data sections for the address as a pointer | **0** — it is in no vtable |
| `E9` tail-jump referrers | byte scan of `.text` for relative jumps to the address | **0** |

All four receivers are a `C_door`:

| callsite | container | how the receiver is proven |
|---|---|---|
| `0x005bc1b8` | `program_process_command` case `0x27` DOOR_OPEN | explicit `GetKind == 6` gate |
| `0x005c6711` | `program_process_command` case `0x162` DOOR_GETSTATE | explicit `GetKind == 6` gate |
| `0x0041620a` | `entity_action_tick` action type `0x07` | same receiver → `door_set_state` |
| `0x004181f5` | `entity_action_tick` action type `0x13` | same receiver → `door_set_state` |

In the two `entity_action_tick` arms the same receiver expression is loaded twice in one
straight-line run — once for `door_get_state`, once as `this` for `door_set_state` with the ticking
actor pushed as the `C_actor *` opener. One of them has a `je` between the ECX write and the call,
so an ECX census that refuses to cross a branch grades it unsafe; the disassembly settles it anyway.

It is **not an ICF fold**: a fold means one address serves two receiver classes, and the census
returns one. What is shared is only the **offset**: `C_human+0x70` is that class's current animation
id, read **inline** by dispatcher case `0x181` HUMAN_GETACTANIMID, never through this address.

**The two AI actions.** Types `0x07` and `0x13` carry a `C_door *` at `inner+0x00` and drive a door
through this getter/setter pair:

| type | what the arm does |
|---|---|
| `0x07` | **Open it.** State 1 or 3 ⇒ `door_set_state(2)`; state 0 ⇒ the task pops; state 2 ⇒ wait. |
| `0x13` | **Close it once clear.** 2 s / 5 s deadline window plus a distance gate ¹. |

¹ Farther than 3 m ⇒ the task completes without acting; closer than 1.4 m ⇒ still in the doorway,
wait; in between with the door resting open ⇒ `door_set_state(3)`.

**Neither arm runs in the shipped game, because nothing creates either action.** Every
`ai_action` gets its UID from `ai_action_generate_UID`, and a byte census of its callers returns
exactly the 35 `ai_action_new_*` factories (`structs/entity.md` §6b), none of which stamps `0x07`
or `0x13`; the PS2 `GenUID` likewise has 35 callers, all `New*` factories. No instruction in the
image stores the immediate `7` or `0x13` into a register-addressed object as an action type.
NPCs open doors through the path follower and the use-action instead (the caller table below),
and nothing closes a door behind an NPC. In game, NPCs are seen opening doors and never
closing one.

**Every caller of `door_set_state`** (an `E8` census of `.text`, 11 callsites):

| caller | ×  | what it asks for |
|---|---|---|
| `ai_path_update_spec_waypoints` | 3 | state 2 for the closed door a spec-node search left at `ai_path+0x68`, when the walker is a `C_entity` — **the path follower opens doors on its route** (`structs/ai_path.md` §1b) |
| `entity_action_tick` | 2 | the type-`0x07` open and type-`0x13` close actions above, which no code creates |
| `program_process_command` | 2 | dispatcher case `0x27` `door_open xx, yy`: state 3 for `yy == 0` on a door resting open, state 2 otherwise on a door resting closed |
| `human_use_actor` | 1 | the use-action: the player's Action Button, and `ai_path_revalidate_for_actor` for an NPC stuck behind a door (`structs/ai_path.md` §3) |
| `door_game_init` | 1 | state 2, with the active player as the opener, when `+0x124` (start open) is set |
| `door_apply_net_event_msg` | 1 | the network receiver (§5) |
| `door_set_state` | 1 | the linked-leaf push-down (§3 step 5) |

### 4c. The lock is `+0x125`, and `door_set_state` refuses to open

```
door_set_state(this, state, actor, force, walk_up):
    ... linked-leaf walk-up ...
    if (state == 2) {                      // 2 = open
        if (this->locked /* +0x125 */) {   // the lock
            ... play this->+0x150 (d_woodlo.wav), behind the 1000 ms gate on +0x160 ...
            return;                        // every path in this block returns
        }
        ... normal open: sound, hinge geometry ...
    }
    else if (state == 3) { ... closing sound + hinge reset ...  }   // not gated
    ... linked-leaf push-down, replication ...
    this->state /* +0x70 */ = state;       // never reached when locked
    actor_set_act_state(0);
```

| # | artifact | what it says |
|---|---|---|
| 1 | the script vocabulary | the command that writes `+0x125` is **`door_lock xx, yy`** (dispatcher case `0x26`), gated on kind 6 |
| 2 | `door_set_state` | the state-2 arm tests `+0x125` first and every path under it returns, skipping the `+0x70` write |
| 3 | the shipped asset names | the refusal plays `+0x150` = `d_woodlo.wav` (§2) |
| 4 | in-game | the prompt still appears, the locked sound plays, the door stays closed |

Consequences:

* **Only opening is gated.** State 3 is not tested, so `door_open xx, 0` and the network receiver
  can still close a locked door.
* The Mafia Script Data Reference's note on `door_open` — *"does not work if the door is locked"* —
  is accurate: a locked door is one reason the command does nothing, alongside a door in mid-swing.
* The AI pathfinder reads `+0x125` at four sites in the three `ai_path_find_spec_node_*`
  functions and, on a locked door, stamps a `blocked_con` (magic `0x29a`, 30 000 ms expiry) so the
  NPC routes around it; an unlocked closed door is opened instead (§4b).

#### 4c-i. Two independent "locked" mechanisms

| | `door_lock xx, 1` (case `0x26`) | `door_enableus xx, 0` (case `0x28`) |
|---|---|---|
| what it writes | `door+0x125` | removes the door from `C_game`'s using-object list (flag `0x40`) |
| Action Button prompt | still shown | gone |
| player uses it | rattle (`+0x150`), door stays shut | cannot ask |
| `door_set_state(2)` from any caller | refused | succeeds |
| an NPC's path follower (`ai_path_update_spec_waypoints`) | refused; the edge is blocked for 30 s | opens the door |

`door_enableus 0` removes the prompt via `using_object_del_object` and never touches `+0x70`,
`+0x125` or `door_set_state`. A door can be locked, unusable, both or neither, and a player calls
both results "locked" — before explaining a locked-door behaviour, establish which one the door has
from the mission scripts.

**Worked case — mission `MISE19-BANKA`, script `1get key from doors`.** In game, the door
`Cylinder00` (prison-style bars) is opened by the guards `BnkO5` / `BnkO4` when they chase the
player, while the player cannot open it. The script contains the whole explanation:

```
findactor 3, "Cylinder000"
findactor 4, "Cylinder00"      // the door
findactor 5, "BnkO5"           // the two guards
findactor 6, "BnkO4"
...
door_enableus 4, 0             // at load: Cylinder00 gets no Action Button prompt
...                            // (the player picks up the key here)
human_addweapon 0, 16, 0, 0
...
door_lock 1, 0                 // after the key: unlock all five bar doors
door_lock 2, 0
door_lock 3, 0
door_lock 4, 0                 // Cylinder00 is now unlocked
door_lock 7, 0
act_setstate 5, active         // and the two guards wake up
act_setstate 6, active
```

What is measured:

* `door_enableus 4, 0` is issued at scene load and never undone; no `door_enableus 4, 1` exists, and
  this is the only script in the mission that resolves `Cylinder00`.
* `door_lock` clears the lock on five doors in one batch, immediately after the key pickup and
  immediately before `act_setstate` wakes the two guards.
* With the lock clear and the prompt absent, `door_set_state(2)` from the guards' path follower
  (`ai_path_update_spec_waypoints`) succeeds and the player has no way to ask. Once a guard opens it, the player can walk through.

In-game assessment by the player-tester:

> *"I don't think it's a bug. Developers wanted you to take the longer route. If this door was
> open, there would be no reason to access part of the built map / doorway."*
> *"No one thought about it that guards can still use it."*
> *"If guards open it then also the player can walk through, which is not expected."*

Designer intent is not recoverable from the binary; this file records only the measurements and the
observed outcome.

## 5. Replication

`door_set_state` step 6 is the send half; `door_apply_net_event_msg` (vtable slot 38) is the receive
half.

| step | function | what it does |
|---|---|---|
| 1 | `event_msg_door_state_payload_ctor` (`0x005946a0`) | called once, from `door_set_state`, as `(door+0x18, param_2 == 2)`: field 0 = door GUID, field 1 = the open/close verb |
| 2 | `event_msg_door_state_ctor` (`0x005a1ad0`) | builds the `0x18`-byte message on `g_event_msg_vt_door_state` (`.rdata 0x0063c21c`), type **`0x34`** |
| 3 | `event_msg_door_state_serialize` / `_deserialize` (`0x005946d0` / `0x00594700`) | vtable slots 2 / 3 |
| 4 | `door_apply_net_event_msg` (`0x00482b50`) | accepts exactly type `0x34` and turns it back into a `door_set_state` call |

The message class sits in `.rdata` between `C_door`'s vtable and `C_drop_in_weapon`'s. Full
write-up: `structs/net_event_msg.md`.

## 6. Method coverage — 21/23 PS2 methods

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x00425700` | `door_get_state` | - | - | STRUCTURAL ¹ |
| `0x004800f0` | `door_ctor` | `__ct__6C_doorFv` | - | CONFIRMED |
| `0x004801d0` | `door_scalar_deleting_dtor` | - | 16 | STRUCTURAL ² |
| `0x004801f0` | `door_dtor` | `__dt__6C_doorFv` | - | CONFIRMED |
| `0x00480200` | `door_duplicate_from` | `DuplicateFrom__6C_doorFP7C_actor` | 21 | CONFIRMED |
| `0x004802c0` | `door_hit` | `Hit__6C_doorF10E_hit_type…` | 31 | CONFIRMED |
| `0x00480450` | `door_set_state` | `SetState__6C_doorFQ26C_door7E_stateP7C_actorbb` | - | CONFIRMED |
| `0x00480880` | `door_ai` | `AI__6C_doorFUi` | 13 | CONFIRMED |
| `0x00480d50` | `door_net_apply_remote_state` | `NetDirect__6C_doorFUi` | 15 | CONFIRMED |
| `0x00480d70` | `door_game_init` | `GameInit__6C_doorFv` | 25 | CONFIRMED |
| `0x00481c20` | `door_game_done` | `GameDone__6C_doorFv` | 26 | CONFIRMED |
| `0x00481c90` | `door_enable_using_object` | `EnableUsingObject__6C_doorFb` | - | CONFIRMED |
| `0x00481d10` | `door_save_data` | `SaveData__6C_doorFP7C_chunk` | 19 | CONFIRMED |
| `0x00481ef0` | `door_load_data` | `LoadData__6C_doorFP7C_chunk` | 20 | CONFIRMED |
| `0x004823b0` | `door_save_game_get_size` | `SaveGameGetSize__6C_doorFv` | 22 | CONFIRMED |
| `0x004823c0` | `door_save_game_save` | `SaveGameSave__6C_doorFPPv` | 23 | CONFIRMED |
| `0x00482420` | `door_save_game_load` | `SaveGameLoad__6C_doorFPPv` | 24 | CONFIRMED |
| `0x004826a0` | `door_set_open_angle` | `SetOpenAngle__6C_doorFf` | - | CONFIRMED |
| `0x00482940` | `door_get_bound_points_2d` | `GetBoundPoints2D__6C_doorFRQ23std48vector<9S_vector2,…>` | 34 | CONFIRMED |
| `0x00482b50` | `door_apply_net_event_msg` | - | 38 | STRUCTURAL ³ |
| `0x00482b80` | `door_on_act_state_change` | `ChangeState__6C_doorFv` | 27 | CONFIRMED |

¹ Receiver census, §4b; no PS2 symbol (§7). ² MSVC-only. ³ PC-only (§7).

**Role-only slots** (shared stubs — a role, not a twin): slot 14 `Update` =
`stub_noop_args4_00461180`, slot 30 `Collision` = `stub_retfalse_args24_00482690`, slot 18 `Init` =
`wagon_init` (`return actor_init(frame) != 0;`, a body several classes share).

**Not part of this class** (sits in its TU): `i3d_copy_bound_volume` (`0x00482ba0`) — receiver
byte-proven an `I3D_frame` (`+0xac` flags, `+0xb0` bound volume), 5 references; a shared helper.

`door_hit` shatters a glass pane through `glass_shatter_break_frame`.

## 7. PC vs PS2

* **Two PS2 methods are PS2-port-only:** `ChangeSceneInit__6C_doorFv` and
  `ChangeSceneDone__6C_doorFv` — the city-streaming pair the PC build does not have. No unclaimed PC
  function start remains for either.
* **`door_apply_net_event_msg` is PC-only.** Vtable slot 38 is the same network-event slot as
  `bottle_apply_net_event_msg`, and no PS2 symbol exists for it: the port emptied the per-class
  replication bodies, so the PS2 method count is a lower bound for this class.
* **`door_get_state` has no PS2 symbol.** None of `C_door`'s 23 PS2 symbols is a getter, so the
  accessor is a header inline there and the PC out-of-line copy has nothing to diff against; its
  class was settled PC-side (§4b). The enum name `C_door::E_state` is spelled out in the setter's
  mangled signature, `SetState__6C_doorFQ26C_door7E_stateP7C_actorbb`.
* **`door_load_data` is 1216 B against a 5044 B PS2 `LoadData` (ratio 0.24) and the mapping is
  right.** GCC memcpys keyword rodata onto the stack where MSVC emits immediates, so a low ratio here
  is data materialisation, not a mismatch.
* **Field delta:** the flag word is PC `+0x110` ↔ PS2 `+0x160`, the sound names PC `+0x130` ↔ PS2
  `+0x180`, the act state PC `+0x1c` ↔ PS2 `+0x5c`. The class band is a flat `+0x50`, but the
  `C_actor` base band is not — do not rebase across the boundary.

## 8. Related

`structs/entity.md` §1 (the kind enum) · `structs/actor.md` (the base) · `structs/dog.md` (the
previous TU) · `structs/program.md` (`C_using_object`) · `structs/using_object.md` ·
`structs/net_event_msg.md` · `basic/artifacts/player-interactions.md` (the using-object action menu).

## 9. Open items

| item | state |
|---|---|
| can an NPC open a door whose `+0x125` is set? | on the code as read it cannot (§4c); not tested in game |
| does a glass door still block movement after shattering? | `door_hit` shatters the pane, but nothing observed clears the collision proxy |
