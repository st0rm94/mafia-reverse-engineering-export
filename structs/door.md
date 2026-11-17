# `C_door` — the hinged door

**Created** 2026-08-13 12:57 +0200 · **Modified** 2026-10-28 18:41 +0100

A door leaf on a hinge, driven by a single angle. `door_set_open_angle` places the leaf at a
fraction of its maximum swing and rebuilds the collision proxy from it; `door_ai` ramps that
fraction toward whatever the current state wants; and `door_set_state` is the public verb
everything else calls (§4).

| | |
|---|---|
| PC prefix | `door_` |
| PC kind (`+0x10`) | `6`, stamped by `door_ctor` |
| PC vtable | `g_door_vt` @ `0x0063c180`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x1b8` (`operator_new(0x1b8)` in `mission_create_actor` `case 6`) |
| PC constructor | `door_ctor` @ `0x004800f0` |
| PC translation unit | `[0x004800c0, 0x00482bd0)`: the unit's CRT pair, the `door_*` functions, one shared slot stub and one frame helper; the next unit's guard `crt_static_init_guard_00482bd0` closes it. `door_get_state` (`0x00425700`) is emitted far outside the range |
| PS2 class | `C_door` (tag `6C_door`), 23 methods: 19 CONFIRMED twins, 2 base stubs, all mapped in §3; 2 PS2-port-only (§9) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC.

```c
typedef struct C_door {                          // 0x1b8 bytes; PC offsets
    /* +0x000 */ C_actor            base;         // [H] the C_actor band (structs/actor.md)
    /* +0x070 */ int32_t            state;        // [H] 0 rest open, 1 rest closed, 2 opening, 3 closing
    /* +0x074 */ float              angle;        // [H] the current opening angle, 0..max_angle
    /* +0x078 */ tDynamicCollObject coll;         // [H] the embedded collision proxy, 0x98 bytes
    /* +0x110 */ uint32_t           flags;        // [H] the flag word (§2a)
    /* +0x114 */ C_door            *leaf_next;    // [H] the linked leaf the state is pushed down to
    /* +0x118 */ C_door            *leaf_prev;    // [H] the linked leaf the walk-up follows
    /* +0x11c */ uint8_t            inert;        // [H] designer data: forces flags 0x2 | 0x4
    /* +0x11d */ uint8_t            hinge_enabled;// [H] gates the opener-side test of §6 step 3
    /* +0x11e */ uint8_t            mirror;       // [H] negates the swing; set by the opener-side test
    /* +0x120 */ float              max_angle;    // [H] the full swing; the ctor seeds pi/2
    /* +0x124 */ uint8_t            start_open;   // [H] door_game_init opens the door when set
    /* +0x125 */ uint8_t            locked;       // [H] refuses state 2 (§7c)
    /* +0x128 */ float              open_speed;   // [H] door_ai: angle += dt * this in state 2
    /* +0x12c */ float              close_speed;  // [H] door_ai: angle -= dt * this in state 3
    /* +0x130 */ char               snd_open[16]; // [H] default `d_woodop.wav` (§5)
    /* +0x140 */ char               snd_close[16];// [H] default `d_woodcl.wav`
    /* +0x150 */ char               snd_locked[16];// [H] default `d_woodlo.wav`
    /* +0x160 */ uint32_t           last_sound_ms;// [H] the 1000 ms re-trigger gate
    /* +0x164 */ uint8_t            hinge_locked; // [H] skips the opener-side test of §6 step 3
    /* +0x168 */ S_vector           leaf_dir;     // [H] the leaf's unit width axis, world space
    /* +0x174 */ S_vector           edge_point;   // [H] the leaf's free edge, frame-local
    /* +0x180 */ S_using_object     uo;           // [H] the player action-menu record, 0x38 bytes
} C_door;
```

The three sound names are **16-byte buffers** holding a file name: each shipped default is twelve
characters plus the terminating NUL, and the buffers sit 0x10 apart. `leaf_dir` is set by
`door_game_init` to the unit axis (X or Z) along the larger horizontal extent of the frame's bound
box, flipped when the box lies on the negative side, then rotated into world space; `door_set_state`
measures the angle between it and the opener. `edge_point` is the bound box's far extent along that
axis pushed out by 0.4 units; `door_get_bound_points_2d` transforms it. `uo` is the record the
using-object menu lists (`structs/using_object.md` §1a): position `+0x180`, radius `+0x18c`, and the
action code `+0x1ac` that `door_ai` swaps (§7a).

## 2. Field evidence

Measured by a census of the decomp's `this+N` accesses inside the `door_*` functions and of every
operand with each displacement in their bytes; the readers outside the class come from the same
displacement census, kept where the receiver is a door.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `door_ctor` (through `actor_ctor`; vtable, kind `+0x10` = 6) | the `C_actor` methods; `door_set_state` (the GUID `+0x18`), `door_on_act_state_change` (the act state `+0x1c`) | the inherited band |
| +0x070 | state | H | `door_set_state` (2, 3), `door_ai` (0, 1), `door_game_init`, `door_save_game_load`, `door_set_open_angle` | `door_get_state`, `door_ai`, `door_net_apply_remote_state`, `door_game_init`, `door_save_game_save` | §7a; `door_get_state`'s four callers in §7b |
| +0x074 | angle | H | `door_ai`, `door_game_init`, `door_save_game_load`, `door_set_open_angle` | `door_ai`, `door_save_game_save` | - |
| +0x078 | coll | H | `door_game_init` (builds it), `door_ai`, `door_set_open_angle`, `door_save_game_load` (its centre `+0x88..+0x90`) | `collision_dyn_update` (`door_ai`), `collision_dyn_erase` (`door_game_done`, `door_on_act_state_change`), `door_game_init`, `door_set_open_angle` | erased while opening and at GameDone unless flag `0x2` |
| +0x110 | flags | H | `door_game_init`, `door_enable_using_object` (bit `0x40`), `door_save_game_load` | `door_set_state`, `door_ai`, `door_game_init`, `door_game_done`, `door_enable_using_object`, `door_save_game_save`, `door_save_game_load`, `door_set_open_angle`, `door_on_act_state_change` | §2a |
| +0x114 | leaf_next | H | `door_game_init` | `door_set_state`, `door_game_init` | §6 step 5 |
| +0x118 | leaf_prev | H | `door_game_init` | `door_set_state` | §6 step 1 |
| +0x11c | inert | H | `door_ctor`, `door_load_data` | `door_game_init`, `door_save_data` | when set, `door_game_init` sets flags `0x2 \| 0x4`: no dynamic collision, never usable |
| +0x11d | hinge_enabled | H | `door_ctor` (= 1), `door_duplicate_from`, `door_load_data` | `door_set_state`, `door_duplicate_from`, `door_save_data` | - |
| +0x11e | mirror | H | `door_ctor`, `door_duplicate_from`, `door_set_state`, `door_game_done` (from flag `0x10`), `door_save_game_load`, `door_load_data` | `door_ai`, `door_game_init`, `door_set_open_angle`, `door_save_game_save`, `door_save_data`, `door_duplicate_from` | a door opens away from whoever opened it (§6 step 3) |
| +0x120 | max_angle | H | `door_ctor` (= π/2), `door_duplicate_from`, `door_load_data` | `door_ai`, `door_game_init`, `door_set_open_angle`, `door_save_data`, `door_duplicate_from` | - |
| +0x124 | start_open | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_game_init`, `door_save_data`, `door_duplicate_from` | set ⇒ `door_set_state(2)` and rest open at `max_angle`; clear ⇒ rest closed |
| +0x125 | locked | H | `door_ctor`, `door_duplicate_from`, `door_game_done` (from flag `0x20`), `door_save_game_load`, `door_load_data`, `program_process_command` (case `0x26` `door_lock`) | `door_set_state`, `door_game_init`, `door_save_game_save`, `door_save_data`, `door_duplicate_from`, `ai_path_find_spec_node_from_coll_list`, `ai_path_find_spec_node_from_plane`, `ai_path_find_spec_node_from_positions` | §7c |
| +0x128 | open_speed | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_ai`, `door_save_data`, `door_duplicate_from` | radians per unit time against `max_angle` |
| +0x12c | close_speed | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_ai`, `door_save_data`, `door_duplicate_from` | - |
| +0x130 | snd_open | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_set_state`, `door_save_data` | §5 |
| +0x140 | snd_close | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_set_state`, `door_save_data` | §5 |
| +0x150 | snd_locked | H | `door_ctor`, `door_duplicate_from`, `door_load_data` | `door_set_state`, `door_save_data` | the only reader is the locked refusal (§7c) |
| +0x160 | last_sound_ms | H | `door_game_init` (= 0), `door_set_state` | `door_set_state` | - |
| +0x164 | hinge_locked | H | `door_ctor`, `door_duplicate_from` | `door_set_state`, `door_game_init`, `door_duplicate_from` | - |
| +0x168 | leaf_dir | H | `door_game_init` | `door_set_state` | - |
| +0x174 | edge_point | H | `door_game_init` | `door_get_bound_points_2d` | - |
| +0x180 | uo | H | `door_ctor` (its defaults), `door_game_init`, `door_ai` (the action code `+0x1ac`) | `using_object_add_object`, `using_object_del_object` (through its address, from `door_enable_using_object`, `door_game_init`, `door_game_done`) | registered in `C_game`'s using-object list |

### 2a. The `+0x110` flag word

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | set by `door_game_init` when the frame has a child of object type 6; a later `door_game_init` returns at once while it is set | `door_game_init` |
| `0x2` | **no dynamic collision**: skip the erase and the proxy copy | `door_game_done`, `door_on_act_state_change`, `door_set_open_angle`, `door_ai` |
| `0x4` | **never usable**: do not register a using-object, do not swap the action code | `door_game_done`, `door_enable_using_object`, `door_ai` |
| `0x8` | **muted**: suppress all three sounds | `door_set_state` |
| `0x10` | persisted into `mirror` (`+0x11e`) at GameDone | `door_game_done` |
| `0x20` | **locked**: persisted into `locked` (`+0x125`) at GameDone | `door_game_done` |
| `0x40` | **currently in `C_game`'s using-object list** | `door_enable_using_object` |

## 3. Method family

Coverage: all **23** PS2 `6C_door` methods are placed: **19 CONFIRMED** twins
(`Init` on the folded `folded_actor_init_bool`), **2 base stubs** (`Update`, `Collision`) and **2 PS2-port-only**
methods (`ChangeSceneInit` / `ChangeSceneDone`, §9). The scalar-deleting destructor,
`door_get_state` and `door_apply_net_event_msg` have no PS2 symbol.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004800c0 | `crt_static_init_guard_004800c0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x004800e0 | `crt_atexit_register_004800e0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x004800f0 | `door_ctor` | instance | `__ct__6C_doorFv` | 0x001531a0 | CONFIRMED | `actor_ctor`, the three sound names, the field defaults, vtable `0x0063c180`, kind 6 |
| 0x004801d0 | `door_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `door_dtor`, vtable slot 16 ¹ |
| 0x004801f0 | `door_dtor` | instance | `__dt__6C_doorFv` | 0x001532e0 | CONFIRMED | restores the vtable and chains the `C_actor` destructor |

¹ No standalone PS2 twin: GCC carries the free flag inside the destructor.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425700 | `door_get_state` | - | - | - | `mov eax,[ecx+0x70]; ret`; four callsites, every receiver a door (§7b). A header inline on the PS2, so no symbol exists |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `Update__6C_doorFUi` | 0x001540a0 | base-stub | slot 14; the shared `ret 4` no-op; the PS2 body is 8 bytes |
| 0x00480200 | `door_duplicate_from` | `DuplicateFrom__6C_doorFP7C_actor` | 0x00153360 | CONFIRMED | slot 21; copies the designer fields and the sound names |
| 0x004802c0 | `door_hit` | `Hit__6C_doorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00153400 | CONFIRMED | slot 31; shatters a glass pane through `glass_shatter_break_frame` |
| 0x00480450 | `door_set_state` | `SetState__6C_doorFQ26C_door7E_stateP7C_actorbb` | 0x001535a0 | CONFIRMED | the public verb (§6) |
| 0x00480880 | `door_ai` | `AI__6C_doorFUi` | 0x00153a30 | CONFIRMED | slot 13; the state machine (§7a) |
| 0x00480d50 | `door_net_apply_remote_state` | `NetDirect__6C_doorFUi` | 0x001540b0 | CONFIRMED | slot 15 |
| 0x00480d70 | `door_game_init` | `GameInit__6C_doorFv` | 0x00154100 | CONFIRMED | slot 25; builds the collision proxy, `leaf_dir`, `edge_point` and `uo`, links the leaves, applies `start_open` |
| 0x00481c20 | `door_game_done` | `GameDone__6C_doorFv` | 0x00155160 | CONFIRMED | slot 26; persists flags `0x10` / `0x20`, unregisters `uo`, erases the proxy |
| 0x00481c90 | `door_enable_using_object` | `EnableUsingObject__6C_doorFb` | 0x00155200 | CONFIRMED | adds or removes `uo` from `C_game`'s using-object list (flag `0x40`); called by dispatcher case `0x28` `door_enableus` |
| 0x00481d10 | `door_save_data` | `SaveData__6C_doorFP7C_chunk` | 0x001552a0 | CONFIRMED | slot 19; the designer record, including the three sound names |
| 0x00481ef0 | `door_load_data` | `LoadData__6C_doorFP7C_chunk` | 0x001554e0 | CONFIRMED | slot 20 (§9) |
| 0x004823b0 | `door_save_game_get_size` | `SaveGameGetSize__6C_doorFv` | 0x001568a0 | CONFIRMED | slot 22 |
| 0x004823c0 | `door_save_game_save` | `SaveGameSave__6C_doorFPPv` | 0x001568c0 | CONFIRMED | slot 23 |
| 0x00482420 | `door_save_game_load` | `SaveGameLoad__6C_doorFPPv` | 0x00156940 | CONFIRMED | slot 24 |
| 0x00482690 | `stub_retfalse_args24_00482690` | `Collision__6C_doorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00156d30 | base-stub | slot 30; a shared `return false` emitted in this unit, which other `C_actor` vtables also hold in slot 30; the PS2 body is 8 bytes |
| 0x004826a0 | `door_set_open_angle` | `SetOpenAngle__6C_doorFf` | 0x00156d40 | CONFIRMED | places the leaf at a fraction of `max_angle` and rebuilds the proxy; called by dispatcher case `0x8f` `door_setopenpose` |
| 0x00482940 | `door_get_bound_points_2d` | `GetBoundPoints2D__6C_doorFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x00157010 | CONFIRMED | slot 34; transforms `edge_point` |
| 0x00482b50 | `door_apply_net_event_msg` | - | - | - | slot 38; the network receiver (§8); PC-only (§9) |
| 0x00482b80 | `door_on_act_state_change` | `ChangeState__6C_doorFv` | 0x001570c0 | CONFIRMED | slot 27; erases the proxy unless flag `0x2` |
| 0x0053c400 | `folded_actor_init_bool` | `Init__6C_doorFP9I3D_frame` | 0x00153340 | CONFIRMED | slot 18; `return actor_init(frame) != 0;`, one body that `C_door`, `C_traffic_generator` and `C_wagon` share (ICF fold) |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00482ba0 | `i3d_copy_bound_volume` | - | - | - | copies a frame's bound volume (`I3D_frame+0xb0`, refreshed when `+0xac` asks for it); its receiver is an `I3D_frame`, not a door. Three decomp callsites, all in this class (`door_ai` ×2, `door_save_game_load`) |

## 4. What it is

`door_set_state` is the public verb everything else calls: the AI walking a path, a script
command, the player's use-action, the door's own init, and the network receiver (§7b). Two things
make the class more than a rotating prop: it registers itself in `C_game`'s using-object list so
the player action menu can offer it, and it links to a **twin leaf** so double doors swing
together from one call.

## 5. The sounds

`door_ctor` copies three sound file names into every instance before anything else happens:

| offset | default | when it plays |
|---|---|---|
| `+0x130` | `d_woodop.wav` | state 2 (opening) on an unlocked door |
| `+0x140` | `d_woodcl.wav` | state 3 (closing) |
| `+0x150` | `d_woodlo.wav` | state 2 refused because the door is locked: the rattle |

Four artifacts settle the order: the ctor's copy order; `door_set_state`'s state-3 arm, which
plays `+0x140`; the locked refusal (§7c), the only reader of `+0x150`; and the in-game rattle on a
locked door. The file names agree: *lo* = locked, *cl* = closed.

The names are overridable per instance (`door_save_data` / `door_load_data` persist all three and
`door_duplicate_from` copies them), so a level designer can hang a metal or glass door on the same
class.

## 6. `door_set_state` — the public verb

Five parameters: `(this, state, actor, force, walk_up)`.

1. **Linked-leaf walk-up.** With `walk_up` set, follow `+0x118` to the outermost leaf and act on
   that one instead; step 5 pushes the same state down `+0x114`. One call, both leaves.
2. **State 2: open. The lock gate comes first.** If `+0x125` (the `door_lock` flag) is set, it
   plays the `+0x150` rattle and **returns** before the `+0x70 = state` write at the bottom of the
   function. The door does not move, and every caller is refused alike (§7c). Otherwise it plays the
   open sound behind four gates (mission sound count > 0, flag bit `0x8` clear, non-empty name
   buffer, sound master on), then a **1000 ms** re-trigger window on `+0x160`, then the queue call
   with the literal quartet **4.0 / 15.0 / 1.0 / 0.001** (volume, range, pitch, rolloff).
3. **Opener side.** When `+0x11d` enables it and an actor was supplied, takes the angle between
   `leaf_dir` (`+0x168`) and the direction to that actor and writes `+0x11e = (angle > π/2)`. That
   is the **mirror flag** `door_set_open_angle` reads, so *a door opens away from whoever opened
   it*.
4. **State 3: closing.** Plays `+0x140` (`d_woodcl.wav`) and resets the hinge point. Not gated by
   the lock: a locked door can still be closed.
5. **Linked-leaf push-down** into `+0x114`.
6. **Replication** (§8). Builds the door-state event message, type **`0x34`**, and posts it.

## 7. The state machine, its callers and the lock

### 7a. `door_ai` — the state machine

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
`collision_dyn_update(+0x78)`, unless flag `0x2` (no dynamic collision) cut them short.

What this establishes:

1. **`state` has four values.** 2 and 3 are the requests `door_set_state` writes; 0 and 1 are the
   resting values only `door_ai` writes. The script command `door_getstate xx, yy` (dispatcher case
   `0x162`, gated `GetKind == 6`) returns this field; the Mafia Script Data Reference documents only
   **0 open / 1 closed**, so a script polling a door mid-swing observes 2 or 3, which the reference
   does not cover.
2. **`+0x128` / `+0x12c` are the angular speeds**, open and close, in radians per unit time against
   `max_angle`'s `π/2`. They are duplicated and saved because they are designer-set per door.
3. **`+0x1ac` is inside the embedded `S_using_object`** (`uo` at `+0x180`, so `+0x2c` into it: the
   action code, `structs/using_object.md` §1a). The door **swaps its own action-menu prompt as it
   settles**: 2001 once fully open, 2000 once fully closed. That is how the Action Button offers
   "close" on an open door without any other code running.

### 7b. `door_get_state` — the `+0x70` reader, and the two AI actions that use it

`door_get_state` (`0x00425700`) is the whole of `mov eax,[ecx+0x70]; ret`.

**Why the class is settled.** Every referrer channel was measured, so the referrer set is closed:

| channel | method | result |
|---|---|---|
| body uniqueness | byte-pattern search of `.text` for the whole body | `8b4170c3`, **1** site image-wide |
| `E8` callsites | byte scan of `.text` for relative calls to the address | **4** |
| `.rdata`/`.data` datarefs | scan of the data sections for the address as a pointer | **0**: it is in no vtable |
| `E9` tail-jump referrers | byte scan of `.text` for relative jumps to the address | **0** |

All four receivers are a `C_door`:

| callsite | container | how the receiver is proven |
|---|---|---|
| `0x005bc1b8` | `program_process_command` case `0x27` DOOR_OPEN | explicit `GetKind == 6` gate |
| `0x005c6711` | `program_process_command` case `0x162` DOOR_GETSTATE | explicit `GetKind == 6` gate |
| `0x0041620a` | `entity_action_tick` action type `0x07` | same receiver → `door_set_state` |
| `0x004181f5` | `entity_action_tick` action type `0x13` | same receiver → `door_set_state` |

In the two `entity_action_tick` arms the same receiver expression is loaded twice in one
straight-line run: once for `door_get_state`, once as `this` for `door_set_state` with the ticking
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

**Neither arm runs in the shipped game, because nothing creates either action.** Every `ai_action`
gets its UID from `ai_action_generate_UID`, and a byte census of its callers returns exactly the 35
`ai_action_new_*` factories (`structs/entity.md` §6), none of which stamps `0x07` or `0x13`; the
PS2 `GenUID` likewise has 35 callers, all `New*` factories. No instruction in the image stores the
immediate `7` or `0x13` into a register-addressed object as an action type. NPCs open doors through
the path follower and the use-action instead (the caller table below), and nothing closes a door
behind an NPC. In game, NPCs are seen opening doors and never closing one.

**Every caller of `door_set_state`** (an `E8` census of `.text`, 11 callsites):

| caller | ×  | what it asks for |
|---|---|---|
| `ai_path_update_spec_waypoints` | 3 | state 2 for the closed door a spec-node search left at `ai_path+0x68`, when the walker is a `C_entity`: **the path follower opens doors on its route** (`structs/ai_path.md` §4) |
| `entity_action_tick` | 2 | the type-`0x07` open and type-`0x13` close actions above, which no code creates |
| `program_process_command` | 2 | dispatcher case `0x27` `door_open xx, yy`: state 3 for `yy == 0` on a door resting open, state 2 otherwise on a door resting closed |
| `human_use_actor` | 1 | the use-action: the player's Action Button, and `ai_path_revalidate_for_actor` for an NPC stuck behind a door (`structs/ai_path.md` §7) |
| `door_game_init` | 1 | state 2, with the active player as the opener, when `+0x124` (start open) is set |
| `door_apply_net_event_msg` | 1 | the network receiver (§8) |
| `door_set_state` | 1 | the linked-leaf push-down (§6 step 5) |

### 7c. The lock is `+0x125`, and `door_set_state` refuses to open

```
door_set_state(this, state, actor, force, walk_up):
    ... linked-leaf walk-up ...
    if (state == 2) {                      // 2 = open
        if (this->locked /* +0x125 */) {   // the lock
            ... play this->+0x150 (d_woodlo.wav), behind the 1000 ms gate on +0x160 ...
            return;                        // every path in this block returns
        }
        ... normal open: sound, opener side ...
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
| 3 | the shipped asset names | the refusal plays `+0x150` = `d_woodlo.wav` (§5) |
| 4 | in-game | the prompt still appears, the locked sound plays, the door stays closed |

Consequences:

* **Only opening is gated.** State 3 is not tested, so `door_open xx, 0` and the network receiver
  can still close a locked door.
* The Mafia Script Data Reference's note on `door_open` (*"does not work if the door is locked"*)
  is accurate: a locked door is one reason the command does nothing, alongside a door in mid-swing.
* The AI pathfinder reads `+0x125` at four sites in the three `ai_path_find_spec_node_*`
  functions and, on a locked door, stamps a `blocked_con` (magic `0x29a`, 30 000 ms expiry) so the
  NPC routes around it; an unlocked closed door is opened instead (§7b).

#### 7c-i. Two independent "locked" mechanisms

| | `door_lock xx, 1` (case `0x26`) | `door_enableus xx, 0` (case `0x28`) |
|---|---|---|
| what it writes | `door+0x125` | removes the door from `C_game`'s using-object list (flag `0x40`) |
| Action Button prompt | still shown | gone |
| player uses it | rattle (`+0x150`), door stays shut | cannot ask |
| `door_set_state(2)` from any caller | refused | succeeds |
| an NPC's path follower (`ai_path_update_spec_waypoints`) | refused; the edge is blocked for 30 s | opens the door |

`door_enableus 0` removes the prompt via `using_object_del_object` and never touches `+0x70`,
`+0x125` or `door_set_state`. A door can be locked, unusable, both or neither, and a player calls
both results "locked": before explaining a locked-door behaviour, establish which one the door has
from the mission scripts.

**Worked case: mission `MISE19-BANKA`, script `1get key from doors`.** In game, the door
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
  (`ai_path_update_spec_waypoints`) succeeds and the player has no way to ask. Once a guard opens
  it, the player can walk through.

In-game assessment by the player-tester:

> *"I don't think it's a bug. Developers wanted you to take the longer route. If this door was
> open, there would be no reason to access part of the built map / doorway."*
> *"No one thought about it that guards can still use it."*
> *"If guards open it then also the player can walk through, which is not expected."*

Designer intent is not recoverable from the binary; this file records only the measurements and the
observed outcome.

## 8. Replication

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

## 9. PC vs PS2

* **Two PS2 methods are PS2-port-only:**

  | PS2 VA | PS2 method | Why there is no PC function |
  |---|---|---|
  | 0x00157110 | `ChangeSceneInit__6C_doorFv` | the city-streaming pair the PC build does not have; no unclaimed PC function start remains for either |
  | 0x00157160 | `ChangeSceneDone__6C_doorFv` | the same |

* **`door_apply_net_event_msg` is PC-only.** Vtable slot 38 is the same network-event slot as
  `bottle_apply_net_event_msg`, and no PS2 symbol exists for it: the port emptied the per-class
  replication bodies, so the PS2 method count is a lower bound for this class.
* **`door_get_state` has no PS2 symbol.** None of `C_door`'s 23 PS2 symbols is a getter, so the
  accessor is a header inline there and the PC out-of-line copy has nothing to diff against; its
  class was settled PC-side (§7b). The enum name `C_door::E_state` is spelled out in the setter's
  mangled signature, `SetState__6C_doorFQ26C_door7E_stateP7C_actorbb`.
* **`door_load_data` is 1216 B against a 5044 B PS2 `LoadData` (ratio 0.24) and the mapping is
  right.** GCC memcpys keyword rodata onto the stack where MSVC emits immediates, so a low ratio here
  is data materialisation, not a mismatch.
* **`Update` and `Collision` are 8-byte PS2 bodies** whose PC slots hold shared stubs; `Init` is a
  32-byte body identical to `C_wagon`'s and `C_traffic_generator`'s, folded onto
  `folded_actor_init_bool` on the PC.
* **Field delta:** the flag word is PC `+0x110` ↔ PS2 `+0x160`, the sound names PC `+0x130` ↔ PS2
  `+0x180`, the act state PC `+0x1c` ↔ PS2 `+0x5c`. The class band is a flat `+0x50`, but the
  `C_actor` base band is not: do not rebase across the boundary.

## 10. Related

- `structs/actor.md` — the base.
- `structs/entity_kind.md` — kind 6.
- `structs/dog.md` — the previous unit.
- `structs/using_object.md` — the `S_using_object` record and the using-object list.
- `structs/program.md` — the script commands that drive a door.
- `structs/net_event_msg.md` — the door-state message.
- `structs/ai_path.md` — the path follower that opens doors.
- `structs/wagon.md`, `structs/pedestrian.md` — the other two owners of the folded `Init`.
- `basic/artifacts/player-interactions.md` — the using-object action menu.

## 11. Open items

1. **Can an NPC open a door whose `+0x125` is set?** On the code as read it cannot (§7c); not
   tested in game.
2. **Does a glass door still block movement after shattering?** `door_hit` shatters the pane, but
   nothing observed clears the collision proxy.
3. **Flag `0x1`** (§2a): `door_game_init` sets it when the frame has a child of object type 6 and
   then skips its whole body on the next call; what such a door is in the shipped scenes is not
   measured.
