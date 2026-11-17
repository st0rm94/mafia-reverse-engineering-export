# `C_using_object` — the usable-object registry behind the action menu

**Created** 2026-09-04 00:21 +0200 · **Modified** 2026-11-09 10:35 +0100

Every prop the player can interact with on foot (a dropped weapon, a car door, a telephone, a body)
registers a record here. When the action button is pressed, `player_do_action_button_on_foot` asks
the registry for everything in reach, and the list it returns is what the on-screen action menu
displays. The object is embedded in `C_game` at `+0x29ec`.

| | |
|---|---|
| PC prefix | `using_object_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x20` |
| PC constructor | - |
| PC translation unit | `[0x0061b050, 0x0061df00)`, the `tools.cpp` unit hosted by `structs/sound_table.md`; the twelve methods are `[0x0061bb30, 0x0061cba0)` inside it, and `using_object_record_ctor` sits in the out-of-line tail of the `C_program` unit (`structs/program.md`) |
| PS2 class | `C_using_object` (tag `14C_using_object`), 14 methods: 12 CONFIRMED twins, 2 PS2-only (§5a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_using_object {                 // 0x20 bytes; at C_game + 0x29ec
    /* +0x00 */ uint8_t          obj_alloc;     // [H] MSVC allocator byte of the object vector
    /* +0x04 */ S_using_object **obj_begin;     // [H] the registered records
    /* +0x08 */ S_using_object **obj_end;       // [H] one past the last registered record
    /* +0x0c */ S_using_object **obj_cap;       // [H] end of the object vector's storage
    /* +0x10 */ uint8_t          invalid_alloc; // [H] MSVC allocator byte of the code vector
    /* +0x14 */ uint32_t        *invalid_begin; // [H] the suppressed action codes
    /* +0x18 */ uint32_t        *invalid_end;   // [H] one past the last suppressed code
    /* +0x1c */ uint32_t        *invalid_cap;   // [H] end of the code vector's storage
} C_using_object;
```

The class has no constructor of its own: `game_ctor` writes the object vector's allocator byte and
zeroes its triple inline at `C_game + 0x29ec`, then calls `std_vector_construct_empty` with ECX on
`+0x10`, which does the same for the code vector.

**The receiver is byte-proven.** A census of the value loaded into ECX before every `E8` call to the
nine non-savegame members returns **118 callsites, all resolved, none ambiguous**, and every one
loads the same address:

```
mov ecx, [<reg> + 0x24]     ; the C_game singleton
add ecx, 0x29ec             ; -> C_using_object
```

or the equivalent `lea ecx,[eax + 0x29ec]` when the caller holds the `C_game`. Per member:
45 (`using_object_del_object`), 33 (`_add_object`), 24 (`_del_invalid_item`), 16
(`_add_invalid_item`), and 2 or 1 for the rest.

**The second vector belongs to this class.** `using_object_add_invalid_item`, `_del_invalid_item`
and `_is_invalid_item` have container boilerplate bodies, but all three index `+0x14` / `+0x18` /
`+0x1c`, the second vector of this class. A bare `std::vector<int>` method would index its own
`+0x00`.

### 1a. `S_using_object` — one registered record, `0x38` bytes

The type name is the PS2 template argument of `AddObject` / `DelObject`. The block `+0x28..+0x37` is
an `S_GameItem` (`structs/inventory.md` §1a), copied whole into the action-menu list.

```c
typedef struct S_using_object {                 // 0x38 bytes
    /* +0x00 */ float      pos[3];              // [H] world position (x, y, z)
    /* +0x0c */ float      radius_sq;           // [H] the horizontal reach, squared
    /* +0x10 */ S_matrix  *transform;           // [H] when set, pos is transformed by it first
    /* +0x14 */ uint32_t   flags;               // [H] 0x1 = the registry deletes the record, 0x2 = it owns the frame at +0x18
    /* +0x18 */ I3D_frame *frame;               // [H] released on removal when flag 0x2 is set
    /* +0x1c */ C_actor   *owner;               // [H] the item actor to remove after a pickup, or 0
    /* +0x20 */ I3D_frame *frame_filter;        // [H] 0 = any; else the asker's parent frame must match
    /* +0x24 */ uint8_t    save_excluded;       // [H] 1 = the savegame filter skips the record
    /* +0x28 */ uint16_t   item_id;             // [H] S_GameItem +0x00
    /* +0x2c */ uint32_t   action_code;         // [H] S_GameItem +0x04 (§1b)
    /* +0x30 */ uint32_t   reserve_ammo;        // [H] S_GameItem +0x08
    /* +0x34 */ S_using_object *self;           // [H] S_GameItem +0x0c, back to the record
} S_using_object;
```

### 1b. `action_code` is a code per item kind

* `human_do_box_pick_up` fills the invalid-code list with `0x7d3, 0x7d4, 0x7d7, 0x7d9, 0x7da, 0x7dc,
  0x7dd, 0x7df` = **2003…2015**, inside the textdb action-label range 2000–2017
  (`basic/artifacts/player-interactions.md`), and does so exactly while the player is carrying a
  box: "these actions are unavailable right now".
* `detector_create_using_object` writes its caller's action argument into the slot (`detector +
  0x10c`, the record's `+0x2c`).
* The `createweaponfromframe` path puts the magazine round count there instead, and a dropped item
  its clip ammo (`structs/drop_in_weapon.md` §1).

### 1c. Where records come from

The 28 functions that call `using_object_add_object` register two kinds of record. Most are members
of their owner, filled in place before registration: `C_door` `+0x180`, `C_entity`'s own record at
`+0x1230` and its body's at `+0x9c4` (offered once it is dead), `C_detector` `+0xe0`,
`C_drop_in_weapon` `+0x3c0`, the railway switch `+0x10c`, `C_game` `+0x2ea0` (the carried-box
node); the record each box record points to at its `+0x04`; and records inside blocks their owner
allocates: the car's seat block at `C_car + 0x210c` (stride `0xa0` from `+0x14`, plus one at
`+0xec`) and the railway's two entry arrays at `+0x270` / `+0x274`. Three paths allocate a
`0x38`-byte record on the heap: the `createweaponfromframe` arm of `program_process_command`,
`drop_in_weapon_on_act_state_changed` and `using_object_save_game_load`.

**The record constructor path.** `using_object_record_ctor` zero-initialises a new record: it clears
`+0x10`, `+0x18`, `+0x1c`, `+0x20`, `+0x24` and the whole `S_GameItem` block, and skips `+0x14` and
`+0x00..+0x0f`, which the caller fills. Its only caller, the `createweaponfromframe` arm, calls
`operator_new(0x38)`, then the ctor, then writes `item_id` = `argv[1]` (the Mafia Script Data
Reference: *"xx = weapon ID"*), `action_code` = `argv[2]` (the magazine round count),
`reserve_ammo` = `argv[3]` (the pocket count) and `self`, before calling `using_object_add_object`.
The arm also sets `radius_sq` = 0.5, `flags` = 3, `frame` = the script's frame, `save_excluded` = 1
and `frame_filter` = that frame's first ancestor of type 5.

### 1d. The reach test

Both query methods use the identical gate:

```
|dy|² < 2.25                      // within 1.5 units vertically
dx² + dz² < object->radius_sq     // within the object's own radius horizontally
```

The reach volume is a flat cylinder, so standing on a balcony directly above a prop does not offer
it (Open item 3).

## 2. Field evidence

Measured by a census of every access in the `using_object_*` functions and a body read of each.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | obj_alloc | H | `game_ctor` | - | allocator byte |
| +0x04 | obj_begin | H | `game_ctor` (0), `using_object_add_object` | every member that walks the objects, `folded_vector_pair_free_at_0x04_0x14_00607500` | - |
| +0x08 | obj_end | H | `game_ctor` (0), `using_object_add_object`, `using_object_del_object` (swap-and-pop), `using_object_clear_objects` | the same walkers | - |
| +0x0c | obj_cap | H | `game_ctor` (0), `using_object_add_object` | `using_object_add_object` | - |
| +0x10 | invalid_alloc | H | `std_vector_construct_empty` (from `game_ctor`) | - | allocator byte |
| +0x14 | invalid_begin | H | `std_vector_construct_empty` (0), `using_object_add_invalid_item` | `using_object_clear_invalid_items`, `using_object_add_invalid_item`, `using_object_del_invalid_item`, `using_object_is_invalid_item`, `folded_vector_pair_free_at_0x04_0x14_00607500` | - |
| +0x18 | invalid_end | H | `std_vector_construct_empty` (0), `using_object_clear_invalid_items`, `using_object_add_invalid_item`, `using_object_del_invalid_item` | `using_object_add_invalid_item`, `using_object_del_invalid_item`, `using_object_is_invalid_item` | - |
| +0x1c | invalid_cap | H | `std_vector_construct_empty` (0), `using_object_add_invalid_item` | `using_object_add_invalid_item` | - |

### 2a. `S_using_object`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | the registering owner (§1c), `using_object_save_game_load` | `using_object_is_any_object_near`, `using_object_find_near_objects` | transformed by `+0x10` when set |
| +0x0c | radius_sq | H | the registering owner, `using_object_save_game_load` | `using_object_is_any_object_near`, `using_object_find_near_objects`, `using_object_save_game_save` | compared against `dx² + dz²` directly; 0.5 on the `createweaponfromframe` records |
| +0x10 | transform | H | `using_object_record_ctor` (0), `using_object_save_game_load` (0), the registering owner | `using_object_is_any_object_near`, `using_object_find_near_objects` | - |
| +0x14 | flags | H | the registering owner (3 on the heap paths) | `using_object_del_object`, `using_object_clear_objects`, `using_object_save_game_get_size`, `using_object_save_game_save` | the savegame keeps only flags == 3 (§4) |
| +0x18 | frame | H | `using_object_record_ctor` (0), `using_object_save_game_load`, the registering owner | `using_object_del_object`, `using_object_clear_objects`, `using_object_save_game_save` (world position and rotation), `player_do_action_button_on_foot` (the box of action codes `0x7da`, `0x7db`) | invalidated, detached and released when flag 0x2 is set |
| +0x1c | owner | H | `using_object_record_ctor` (0), `using_object_save_game_load` (0), the registering owner | `player_do_action_button_on_foot` | after a pickup: non-zero → `game_remove_temporary_actor(owner)`, zero → `using_object_del_object` (`structs/drop_in_weapon.md` §5) |
| +0x20 | frame_filter | H | `using_object_record_ctor` (0), `using_object_save_game_load` (0), the `createweaponfromframe` arm, the registering owner | `using_object_is_any_object_near`, `using_object_find_near_objects` | both queries compare it with the frame their caller passes, the actor's parent (`I3D_frame +0x120`) |
| +0x24 | save_excluded | H | `using_object_record_ctor`, `using_object_save_game_load`, `door_ctor`, `pedestrian_ctor`, `entity_ctor`, `detector_ctor`, `drop_in_weapon_on_act_state_changed`, `game_box_add` (all 0); the `createweaponfromframe` arm (1) | `using_object_save_game_get_size`, `using_object_save_game_save` | so the `createweaponfromframe` records are not saved. Writers by a byte-store scan of `.text` at this displacement; a record embedded in an owner has another displacement |
| +0x28 | item_id | H | `using_object_record_ctor` (0), the `createweaponfromframe` arm, the registering owner | `using_object_save_game_get_size`, `using_object_save_game_save` (saved only when above 1), `using_object_find_near_objects` (the block copy), `player_do_action_button_on_foot` (the pickup sound) | - |
| +0x2c | action_code | H | `using_object_record_ctor` (0), the `createweaponfromframe` arm, `detector_create_using_object`, the registering owner | `using_object_is_any_object_near`, `using_object_find_near_objects` (passed to `using_object_is_invalid_item`), `using_object_save_game_save` | §1b |
| +0x30 | reserve_ammo | H | `using_object_record_ctor` (0), the `createweaponfromframe` arm, `human_drop_out_items`, `using_object_save_game_load` | `using_object_find_near_objects` (the block copy, read on by `inventory_insert_new_item`), `using_object_save_game_save` | only weapon records set it ¹ |
| +0x34 | self | H | `using_object_record_ctor` (0), the `createweaponfromframe` arm, the registering owner | `player_do_action_button_on_foot` (through the menu copy), `using_object_save_game_save` | the loader replaces the saved value with the new record's address |

¹ A census of the writes at record `+0x30` of every registrar finds writers only for weapon records:
`human_drop_out_items` and the `createweaponfromframe` arm, through the dropped weapon's embedded
`S_GameItem`, and the savegame loader. Door, car, detector and body records leave it at the
constructor's zero.

## 3. Method family

Coverage: **12** of the 14 PS2 `14C_using_object` methods are CONFIRMED twins; `Tick` and
`FindAutoPickObjects` have no PC function (§5a). `using_object_record_ctor` has no PS2 symbol: the
PS2 script arm inlines it.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005c86b0 | `using_object_record_ctor` | a record | - | - | - | zero-initialises a heap record (§1c); called once, by `program_process_command` |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061bb30 | `using_object_add_object` | `AddObject__14C_using_objectFPQ214C_using_object14S_using_object` | 0x003ade00 | CONFIRMED | registers a record unless the pointer is present (backward duplicate scan); 33 callsites |
| 0x0061bce0 | `using_object_del_object` | `DelObject__14C_using_objectFPQ214C_using_object14S_using_object` | 0x003ae5a0 | CONFIRMED | unregisters one record, tears down its frame on flag 0x2, deletes it on flag 0x1, swap-and-pop; 45 callsites |
| 0x0061bdb0 | `using_object_clear_objects` | `ClearObjects__14C_using_objectFv` | 0x003ae700 | CONFIRMED | the same teardown for every record, then empties the vector; called by `game_init` and `game_done` |
| 0x0061be50 | `using_object_is_any_object_near` | `IsAnyObjectNear__14C_using_objectFRC8S_vectorRC8S_vectorP9I3D_frame` | 0x003aeae0 | CONFIRMED | is any record not suppressed inside the reach cylinder (§1d)? Its only caller is `player_ai_walk` |
| 0x0061bf50 | `using_object_find_near_objects` | `FindNearObjects__14C_using_objectFRC8S_vectorRC8S_vectorRQ23gif22vector<10S_GameItem,8>P9I3D_frame` | 0x003aec50 | CONFIRMED | builds the action-menu list of `S_GameItem` blocks, nearest first; its only caller is `player_do_action_button_on_foot` |
| 0x0061c1b0 | `using_object_clear_invalid_items` | `ClearInvalidItems__14C_using_objectFv` | 0x003af450 | CONFIRMED | empties the suppressed list; called by `game_init` and `game_done` |
| 0x0061c1e0 | `using_object_add_invalid_item` | `AddInvalidItem__14C_using_objectFUi` | 0x003af460 | CONFIRMED | suppresses one action code; 16 callsites in `human_do_box_pick_up` and `human_do_dead_body_pick_up` |
| 0x0061c390 | `using_object_del_invalid_item` | `DelInvalidItem__14C_using_objectFUi` | 0x003af4f0 | CONFIRMED | lifts one suppression; 24 callsites, those two plus `human_do_box_cancel` |
| 0x0061c3f0 | `using_object_is_invalid_item` | `IsInvalidItem__14C_using_objectFUi` | 0x003af550 | CONFIRMED | is this code suppressed? Called by the two queries |
| 0x0061c430 | `using_object_save_game_get_size` | `SaveGameGetSize__14C_using_objectFv` | 0x003af5a0 | CONFIRMED | `4 + 0x30` per restorable record and dropped weapon (§4) |
| 0x0061c4b0 | `using_object_save_game_save` | `SaveGameSave__14C_using_objectFPPv` | 0x003af6a0 | CONFIRMED | writes one `0x30`-byte record each |
| 0x0061c750 | `using_object_save_game_load` | `SaveGameLoad__14C_using_objectFPPv` | 0x003afa80 | CONFIRMED | rebuilds each object and re-registers it |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061db60 | `std_vector10_erase_range` | - | - | - | erases a range of 16-byte elements; `using_object_find_near_objects` trims the menu list with it. Four more callers in other units |

## 4. Savegame

Two populations are saved: registered records whose `flags` are 3 (the registry owns the record and
its frame), whose `save_excluded` byte is clear and whose `item_id` is greater than 1; and every live
actor of **kind `0x25`** (`C_drop_in_weapon`) with a frame, the dropped weapons lying in the world.
The loader rebuilds each from the item table and hands it back to `using_object_add_object`.

For a dropped weapon the saver probes straight **down** with a vertical line test from one unit
above the sampled position and lifts the stored Y by **0.2** on a hit, so a weapon resting on
geometry is not reloaded inside it. The PC savegame record writes ids at `[0..3]` and the quaternion
at `[7..10]`.

### 4a. Consumers

* **`player_do_action_button_on_foot`**, the sole caller of `using_object_find_near_objects`; after
  a pickup it follows each taken item back to its record and routes the teardown on `+0x1c`.
* **`player_ai_walk`**, the sole caller of `using_object_is_any_object_near`.
* **`human_do_box_pick_up` / `human_do_dead_body_pick_up` / `human_do_box_cancel`**, the suppressed
  list: an item being carried hides its actions from the menu, and cancel restores them.
* **`game_init` / `game_done`** call `using_object_clear_objects` immediately followed by
  `using_object_clear_invalid_items`.

## 5. PC vs PS2

* **Location:** PS2 `C_game + 0xde0` (PC `+0x29ec`).
* **Record offsets:** the whole PS2 record is a uniform shift of the PC's: PC `+0x14` / `+0x18` /
  `+0x20` / `+0x24` / `+0x28` / `+0x2c` = PS2 `+0x18` / `+0x1c` / `+0x24` / `+0x28` / `+0x30` /
  `+0x34`. On both builds the action code is what `IsInvalidItem` is called with. Do not carry a PS2
  offset into a PC expression.
* **Item id proof:** PS2 `C_human::DropOutItems` indexes `gameItemTable + *(u16 *)&item[i] * 0xf0`,
  the first halfword of `S_GameItem`, stride `0xf0`.
* **The record constructor is inlined on PS2:** the PS2 arm for `createweaponfromframe` (opcode
  `0x59`) inlines the identical zero-init, writes the same four `S_GameItem` fields in the same order
  at `+0x30` / `+0x34` / `+0x38` / `+0x3c`, and calls `AddObject`.
* **Consumers are twins:** PS2 `Do_AB_OwnerNULL__8C_player` is the only caller of
  `FindNearObjects`, `AI_walk__8C_player` of `IsAnyObjectNear`, and the PS2 game init / done make the
  identical adjacent `ClearObjects` + `ClearInvalidItems` pair.
* **`AddObject`'s PS2 body is ~1500 bytes longer** than the PC's, and the extra is an `I3D_model`
  cache load-and-evict: it tests `obj[6] & 4`, allocates driver object type 9, strips the `.i3D`
  suffix off the model name and pulls the model in on demand. The console streams a usable object's
  model in when it is registered; the PC keeps it resident. The twin holds despite the length
  difference, on the identical backward duplicate scan that opens both bodies.
* **`DelObject`'s PS2 body releases a `+0x2c` handle the PC record does not have**, the same model
  handle its `AddObject` acquired.
* **`std::__vector_pod<>::data()` is a real call on the PS2 and inlined on the PC**, so the callee
  multiset for this family is one entry short on the PC by construction.
* **Savegame:** the PS2 record has the ids and the quaternion the other way round, and the type
  constant is PS2 `+0x18 == 0xf` against PC `+0x14 == 3`; a savegame blob is per-build. The downward
  line test on a dropped weapon is present on both.
* **The reach constant** (`|dy|² < 2.25`) is identical on both builds.
* **Record `+0x24` is PS2 `+0x28`.** Both savegame filters test it. On the PS2 the `createweaponfromframe`
  arm (opcode `0x59`) stores 1 there after its inlined zero-init stores 0, as the PC arm does; the
  decomp renders the store as a byte write through a `float *` at index 10
  (`*(undefined1 *)(record + 10) = 1`), so a text search for the offset misses it.
  `BoxPlacementAdd__6C_game` stores 0.

### 5a. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x003ae840 | `Tick__14C_using_objectFUi` | its only PS2 caller is `Tick__6C_gameFUi`, and the PC `game_tick`'s subsystem list, which mirrors the PS2 call sequence, has no such call and no unnamed call at that rank; the tick needs two float phase accumulators at `+0x18` / `+0x1c`, which on the PC are the code vector's `end` and `cap`; no PC function loads a `0.004` or `0.001` per-ms rate together with `6.2831855` ¹ |
| 0x003af100 | `FindAutoPickObjects__14C_using_objectF…` | its only PS2 caller is `AI_walk__8C_player`; the PC twin `player_ai_walk` calls only `using_object_is_any_object_near`, the PC emission rank between `using_object_find_near_objects` and `using_object_clear_invalid_items` is zero bytes wide, and the reach constant `2.25` has five references in the PC image, none a second gatherer ² |

¹ On the PS2 the tick animates a pulsing glow (`sin(phase) * 0.2 + 0.8` written into an
`I3D_frame`'s scale triple, 2π wrap) and a slow spin on every registered object, a console pickup
highlight. The five PC functions that load `0.004` (`door_ctor`, `railway_switch_ctor`,
`human_track_apply_pose_at_clock`, `human_update`, `game_lightness_tick`) are all accounted for
elsewhere.

² The five references are the two query members of §3c (one caller each) and two `ai_path`
functions.

## 6. Related

- `structs/game.md` — the owner, `C_game + 0x29ec`.
- `basic/artifacts/player-interactions.md` — the action menu this feeds.
- `structs/drop_in_weapon.md`, `structs/door.md`, `structs/detector.md` — registrars.
- `structs/inventory.md` §1a — `S_GameItem`.
- `structs/sound_table.md` — the host of the `tools.cpp` unit.

## 7. Open items

1. **In game, pickup highlight:** a usable object should pulse and rotate on the PS2 and sit still on
   the PC, the visible consequence of the missing `Tick`.
2. **In game, auto-pickup:** the PS2 should pick items up while the player walks over them; the PC
   should require the action button.
3. **In game, reach cylinder:** standing on a balcony directly above a prop should not offer it
   (§1d).
