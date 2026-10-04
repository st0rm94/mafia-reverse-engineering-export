# `C_using_object` — the usable-object registry behind the action menu

**Created** 2026-09-04 00:21 +0200 · **Modified** 2026-10-01 18:52 +0200

PC prefix **`using_object_`**. The object is **embedded at `C_game + 0x29ec`**.

Every prop the player can interact with on foot — a dropped weapon, a car door, a
telephone, a body — registers itself here. When the action button is pressed,
`player_do_action_button_on_foot` asks this registry for everything in reach, and the list
it returns is what the on-screen action menu displays.

## 1. The registry object

The receiver is byte-proven, not inferred from a VA range. A census of the value loaded into ECX
before every `E8` call to the nine non-savegame members returns **118 callsites, all resolved, none
ambiguous**, and every one of them loads the same address:

```
mov ecx, [<reg> + 0x24]     ; the C_game singleton
add ecx, 0x29ec             ; -> C_using_object
```

or the equivalent `lea ecx,[eax + 0x29ec]` when the caller already holds the `C_game`.
Per member: 45 (`using_object_del_object`), 33 (`_add_object`), 24
(`_del_invalid_item`), 16 (`_add_invalid_item`), and 2 or 1 for the rest.

```c
typedef struct C_using_object {
/* +0x00 */ char                alloc0;        // MSVC allocator byte
/* +0x04 */ S_using_object    **obj_begin;     // [H] the registered objects
/* +0x08 */ S_using_object    **obj_end;
/* +0x0c */ S_using_object    **obj_cap;
/* +0x10 */ char                alloc1;
/* +0x14 */ uint               *invalid_begin; // [H] suppressed action codes
/* +0x18 */ uint               *invalid_end;
/* +0x1c */ uint               *invalid_cap;
} C_using_object;                               // 0x20 bytes
```

**The second vector belongs to this class.** `using_object_add_invalid_item`,
`_del_invalid_item` and `_is_invalid_item` have textbook container boilerplate bodies, but all
three index `+0x14`/`+0x18`/`+0x1c` — the SECOND vector of this class. A bare
`std::vector<int>` method would index its own `+0x00`.

### 1a. `S_using_object` — the element

Decoded from `using_object_del_object`, `_clear_objects`, `_is_any_object_near` and the record
constructor. The type name is the PS2 template argument on `AddObject` / `DelObject`.

| offset | field | conf | evidence |
|---|---|---|---|
| `+0x00..+0x0b` | world position (3 floats) | H | read by `using_object_is_any_object_near` and `_find_near_objects`, then optionally transformed |
| `+0x0c` | radius **squared** | H | the horizontal distance test compares against it directly |
| `+0x10` | `S_matrix *` / parent transform | H | when non-null the position is multiplied by it before the distance test |
| `+0x14` | flags | H | bit `0x1` = the record is owned and must be `delete`d; bit `0x2` = it owns the `+0x18` frame. A dropped item's heap record carries 3 |
| `+0x18` | `I3D_frame *` | H | invalidated, detached from its parent and released when bit `0x2` is set |
| `+0x1c` | owning actor | H | read by `player_do_action_button_on_foot` after a pickup: non-zero → `game_remove_temporary_actor(owner)`, zero → `using_object_del_object`. A dropped item's in-place record stores the item actor, its heap record 0 (`structs/drop_in_weapon.md` §3a) |
| `+0x20` | frame filter | H | `0` means "any"; otherwise the object is only offered when the caller's frame argument matches |
| `+0x28..+0x37` | **`S_GameItem`**, 0x10 bytes | H | copied wholesale into the action-menu list by `using_object_find_near_objects` |
| `+0x28` | **item id (`uint16`)** = `S_GameItem + 0x00` | H | ¹ |
| `+0x2c` | **action code** = `S_GameItem + 0x04` | H | the value `using_object_is_invalid_item` is called with ² |
| `+0x30` | `S_GameItem + 0x08` | M | pocket ammo on the `createweaponfromframe` path ¹ |
| `+0x34` | self-pointer = `S_GameItem + 0x0c` | H | ¹; the action-menu copy carries it back to the record |

¹ **The record constructor path.** `using_object_record_ctor` zero-initialises a new record: it
clears `+0x10`, `+0x18`, `+0x1c`, `+0x20`, `+0x24` and the whole `S_GameItem` block, deliberately
skipping `+0x14` (the flags) and `+0x00..+0x0f`, which the caller fills. Its only caller, the
`createweaponfromframe` arm of `program_process_command`, calls `operator_new(0x38)`, then the
ctor, then writes `int16 item_id @+0x28` = `argv[1]` (documented by the Mafia Script Data
Reference as *"xx = weapon ID"*), `int32 @+0x2c` = `argv[2]` (the magazine round count),
`int32 @+0x30` = `argv[3]` (the pocket count), and `self @+0x34`, before calling
`using_object_add_object`.

² **`+0x2c` is a per-item-kind CODE, not one fixed quantity.**

* `human_do_box_pick_up` fills the invalid-item list with `0x7d3, 0x7d4, 0x7d7, 0x7d9, 0x7da,
  0x7dc, 0x7dd, 0x7df` = **2003…2015**, inside the textdb action-label range 2000-2017
  (`basic/artifacts/player-interactions.md`), and does so exactly while the player is carrying a
  box — "these actions are unavailable right now".
* `detector_create_using_object` writes its caller's action argument into the same slot
  (`detector + 0x10c`, the record's `+0x2c`).
* The `createweaponfromframe` path puts the magazine round count there instead (¹), and a dropped
  item its clip ammo (`structs/drop_in_weapon.md` §2).

### 1b. The reach test

Both query methods use the identical gate:

```
|dy|² < 2.25                      // within 1.5 units vertically
dx² + dz² < object->radius²       // within the object's own radius horizontally
```

The reach volume is a flat cylinder, so standing on a balcony directly above a prop should NOT
offer it.

## 2. PC coverage — 12 PC twins of 14 PS2 methods

| PC VA | PC name | PS2 twin | PS2 VA | status | role |
|---|---|---|---|---|---|
| `0x0061bb30` | `using_object_add_object` | `AddObject__14C_using_objectFPQ214C_using_object14S_using_object` | `0x003ade00` | CONFIRMED | register a prop, ignoring duplicates |
| `0x0061bce0` | `using_object_del_object` | `DelObject__14C_using_objectFPQ214C_using_object14S_using_object` | `0x003ae5a0` | CONFIRMED | unregister one, tearing down its frame; swap-and-pop |
| `0x0061bdb0` | `using_object_clear_objects` | `ClearObjects__14C_using_objectFv` | `0x003ae700` | CONFIRMED | drop them all; called from `game_init` and `game_done` |
| - | - | `Tick__14C_using_objectFUi` | `0x003ae840` | PS2-ONLY CANDIDATE | §4 |
| `0x0061be50` | `using_object_is_any_object_near` | `IsAnyObjectNear__14C_using_objectFRC8S_vectorRC8S_vectorP9I3D_frame` | `0x003aeae0` | CONFIRMED | is anything in reach? |
| `0x0061bf50` | `using_object_find_near_objects` | `FindNearObjects__14C_using_objectFRC8S_vectorRC8S_vectorRQ23gif22vector<10S_GameItem,8>P9I3D_frame` | `0x003aec50` | CONFIRMED | build the action-menu list, sorted nearest first |
| - | - | `FindAutoPickObjects__14C_using_objectF…` | `0x003af100` | PS2-ONLY CANDIDATE | §4 |
| `0x0061c1b0` | `using_object_clear_invalid_items` | `ClearInvalidItems__14C_using_objectFv` | `0x003af450` | CONFIRMED | empty the suppressed list |
| `0x0061c1e0` | `using_object_add_invalid_item` | `AddInvalidItem__14C_using_objectFUi` | `0x003af460` | CONFIRMED | suppress one action code |
| `0x0061c390` | `using_object_del_invalid_item` | `DelInvalidItem__14C_using_objectFUi` | `0x003af4f0` | CONFIRMED | un-suppress one action code |
| `0x0061c3f0` | `using_object_is_invalid_item` | `IsInvalidItem__14C_using_objectFUi` | `0x003af550` | CONFIRMED | is this code suppressed? |
| `0x0061c430` | `using_object_save_game_get_size` | `SaveGameGetSize__14C_using_objectFv` | `0x003af5a0` | CONFIRMED | `4 + 0x30` per restorable object |
| `0x0061c4b0` | `using_object_save_game_save` | `SaveGameSave__14C_using_objectFPPv` | `0x003af6a0` | CONFIRMED | write one 0x30-byte record each |
| `0x0061c750` | `using_object_save_game_load` | `SaveGameLoad__14C_using_objectFPPv` | `0x003afa80` | CONFIRMED | rebuild each object and re-register it |

`using_object_record_ctor` (`0x005c86b0`) has no PS2 symbol: the PS2 script arm inlines it (§4).

### 2a. Savegame

Two populations are saved: registered `S_using_object` records whose flags `+0x14` are 3 (the
registry owns the record and its frame) and whose `+0x24` flag is clear and whose `+0x28` word is greater than 1, and every live actor of
**kind `0x25`** (`C_drop_in_weapon`) with a frame — the dropped weapons lying in the world. The
loader rebuilds each from the item table and hands it back to `using_object_add_object`.

For a dropped weapon the saver probes straight **down** with a vertical line test from one unit
above the sampled position and lifts the stored Y by **0.2** on a hit, so a weapon resting on
geometry is not reloaded inside it.

The PC savegame record writes ids at `[0..3]` and the quaternion at `[7..10]`.

## 3. Consumers

* **`player_do_action_button_on_foot`** — the sole caller of `using_object_find_near_objects`; after
  a pickup it follows each taken item back to its record and routes the teardown on `+0x1c`.
* **`player_ai_walk`** — the sole caller of `using_object_is_any_object_near`, presumably to
  decide whether to show the "you can interact" prompt.
* **`human_do_box_pick_up` / `human_do_dead_body_pick_up` / `human_do_box_cancel`** — the
  suppressed list. An item being carried hides its actions from the menu; cancel restores them.
* **`game_init` / `game_done`** — call `using_object_clear_objects` immediately followed by
  `using_object_clear_invalid_items`.
* **`using_object_add_object`** only registers a pointer someone else owns; the `flags & 1`
  "delete me" bit says at least some of them are heap records.

## 4. PC vs PS2

* **Location:** PS2 `C_game + 0xde0` (PC `+0x29ec`).
* **Record offsets:** the whole PS2 record is a uniform shift of the PC's — PC `+0x14` / `+0x18` /
  `+0x20` / `+0x28` / `+0x2c` = PS2 `+0x18` / `+0x1c` / `+0x24` / `+0x30` / `+0x34`. On both builds
  the action code (PC `+0x2c`, PS2 `+0x34`) is what `IsInvalidItem` is called with. **Do not carry
  a PS2 offset into a PC expression.**
* **Item id proof:** PS2 `C_human::DropOutItems` indexes `gameItemTable + *(u16 *)&item[i] * 0xf0`
  — the FIRST halfword of `S_GameItem`, stride `0xf0`.
* **The record constructor is inlined on PS2:** the PS2 arm for `createweaponfromframe` (opcode
  `0x59`) inlines the identical zero-init, writes the same four `S_GameItem` fields in the same
  order at `+0x30/+0x34/+0x38/+0x3c`, and calls `AddObject`.
* **Consumers are twins:** PS2 `Do_AB_OwnerNULL__8C_player` is the only caller of
  `FindNearObjects`, `AI_walk__8C_player` of `IsAnyObjectNear`, and the PS2 game init / done make
  the identical adjacent `ClearObjects` + `ClearInvalidItems` pair.
* **`Tick__14C_using_objectFUi` (664 B) — PS2-only candidate.** (a) Closed set: its **only** PS2
  caller is `Tick__6C_gameFUi`, and the PC `game_tick`'s subsystem dispatch list, which mirrors the
  PS2 call sequence, contains no such call and no unnamed call at that rank. (b) Structural: the
  tick needs two float phase accumulators at `+0x18`/`+0x1c`, and on the PC those two dwords are
  the invalid-item vector's `end` and `cap`. On PS2 it animates a pulsing glow
  (`sin(phase) * 0.2 + 0.8` written into an `I3D_frame`'s scale triple, 2π wrap) and a slow spin
  on every registered object — a console pickup highlight the PC does not have.
* **`FindAutoPickObjects__14C_using_objectF…` (844 B) — PS2-only candidate.** Its only PS2 caller
  is `AI_walk__8C_player`; the PC twin `player_ai_walk` calls only
  `using_object_is_any_object_near`, and the PC emission rank between
  `using_object_find_near_objects` and `using_object_clear_invalid_items` is **zero bytes wide**.
  Auto-pickup while walking is a plausible no-mouse console affordance.
* **`AddObject`'s PS2 body is ~1500 bytes longer** than the PC's, and the extra is an
  `I3D_model` cache load-and-evict: it tests `obj[6] & 4`, allocates driver object type 9,
  strips the `.i3D` suffix off the model name and pulls the model in on demand. That is the
  32 MB console port pattern — the console streams a usable object's model in when it is
  registered, the PC keeps it resident. The twin holds despite the length difference; this is
  bannered on `using_object_add_object` in the decomp.
* **`DelObject`'s PS2 body releases a `+0x2c` handle the PC record does not have** — the
  same model handle its `AddObject` acquired.
* **`std::__vector_pod<>::data()` is a real call on the PS2 and inlined on the PC**, so the
  callee multiset for this whole family is one entry short on the PC by construction.
  Bannered on `using_object_add_object`.
* **Savegame:** the PS2 record layout has the ids and the quaternion the other way round, and the type constant is PS2 `+0x18 == 0xf` against PC `+0x14 == 3`. Expected: a savegame blob
  is per-build and never exchanged. The downward line test on a dropped weapon is present on both.
* **The reach constant** (`|dy|² < 2.25`) is identical on both builds.

## 5. Related

`structs/game.md` (the owner, `C_game + 0x29ec`) · `basic/artifacts/player-interactions.md` (the
action menu this feeds) · `structs/drop_in_weapon.md` · `structs/door.md` · `structs/detector.md`.

## 6. Open items

| item | state |
|---|---|
| `S_GameItem + 0x08` | carries the pocket count on the `createweaponfromframe` path and a dropped item's reserve ammo; meaning for other item kinds not decoded |
| `S_using_object + 0x24` | tested only by the savegame filter; no other reader located |
| `Tick` PS2-only status | would close with a byte scan of the PC image for the `0.004` / `0.001` per-ms rates paired with `6.2831855` |
| `FindAutoPickObjects` PS2-only status | would close with a whole-binary hunt for a second `S_GameItem` gatherer |
| in game: pickup highlight | a usable object should pulse and rotate on the PS2 and sit still on the PC — the visible consequence of the missing `Tick` |
| in game: auto-pickup | the PS2 should auto-pick items while walking over them; the PC should require the action button |
| in game: reach cylinder | standing on a balcony directly above a prop should NOT offer it (§1b) |
