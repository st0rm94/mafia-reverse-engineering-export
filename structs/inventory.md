# `G_Inventory` — the per-person item/weapon inventory

**Created** 2026-07-26 21:35 +0200 · **Modified** 2026-10-11 21:38 +0200

| | |
|---|---|
| PC prefix | **`inventory_`** |
| PC location | **embedded at `C_human+0x480`** ¹ |
| Owner class | **`C_human`** — so cops, gangsters, pedestrians and the player all have one |
| PC instance size | **`0xe4`** ² |
| PC translation unit | inside `g_loadsave`, `[0x0055a1f0, 0x0055c270)` (next function: `enabled_cars_have_car`), plus three inline members in `C_human`'s object (§4) |
| PS2 class | `G_Inventory`, 33 methods, all accounted for (§3) |

¹ `lea ecx,[reg+0x480]` at six independent callsites across two methods (`inventory_holster` ×2,
`inventory_has_item_with_property` ×4); `lea` ⇒ embedded, not a pointer.
² `game_inventory_stack_push` does `operator_new(0xe4)` and then runs `inventory_reset`'s exact clear
sequence (§4a).

Confidence: **[H]** writer and reader both read · **[M]** inferred from one side · **[U]** not observed.

## 1. Field layout (relative to the `G_Inventory` base)

Derived from `inventory_have_items` and `inventory_has_item_with_property` (which walk every storage
band in the same order), `inventory_reset`, `inventory_holster` and the save pair.

```c
typedef struct G_Inventory {              // 0xe4 bytes, embedded at C_human+0x480
    /* +0x00 */ uint32_t can_carry_big_weapon; // [H] permits the big-weapon slot +0x80
    /* +0x04 */ uint32_t in_car;          // [H] 1 while the owner sits in a car
    /* +0x08 */ uint32_t extra_count;     // [H] live entries in extra[] at +0x90; both scanners
                                          //     gate the +0x90 loop on != 0
    /* +0x0c */ std::vector<uint32_t> cycle_order; // [H] {alloc, begin, end, cap} to +0x18: the
                                          //     weapon-cycle order, item ids
    /* +0x1c */ uint32_t cycle_current_id; // [H] the id the cycle stands on; 0xffffff = none
    // ---- the four storage bands, all holding 0x10-byte S_GameItem instances ----
    /* +0x20 */ S_GameItem in_hand;       // [H] the item currently HELD (band 0, "Hands")
    /* +0x30 */ S_GameItem belt[5];       // [H] band 1, five weapon slots, stride 0x10
    /* +0x80 */ S_GameItem big_weapon;    // [H] band 2, one slot for a big weapon; the UI labels
                                          //     it "Coat"
    /* +0x90 */ S_GameItem extra[4];      // [H] band 3, counted by +0x08, capped at 4
    /* +0xd0 */ S_GameItem car_slot;      // [H] the item put away while the owner sits in a car;
                                          //     the UI's kind 4, "Car"
    /* +0xe0 */ uint8_t  report_to_console; // [H] 1 on the player's inventory only
} G_Inventory;
```

`inventory_set_can_carry_big_weapon` writes `+0x00` and `inventory_set_in_car` writes `+0x04`;
`inventory_can_accept_item` reads `+0x00`.

### 1a. `S_GameItem` — the 0x10-byte inventory instance

```c
typedef struct S_GameItem {       // 0x10 bytes
    /* +0x00 */ uint16_t item_id;      // [H] index into g_game_item_table; 0 = the slot is EMPTY
    /* +0x02 */ uint16_t magazine_out; // [H] the spent magazine or clip has been dropped
    /* +0x04 */ uint32_t clip_ammo;    // [H] rounds in the weapon now — inventory_get_active_weapon_ammo's return
    /* +0x08 */ uint32_t reserve_ammo; // [H] spare rounds — the same function's out-param
    /* +0x0c */ uint32_t unk0c;        // [U] copied with the record; nothing addresses it alone
} S_GameItem;
```

`+0x02` is a latch: `human_start_weapon_effect` sets it when it drops the spent magazine or clip of the
in-hand gun as a physics prop and drops no second one while it is set, the reload animation event in
`human_ai` clears it, and `human_change_weapon_model` reads it.

Do not confuse the two item records. `S_GameItem` is what an inventory *holds*; `g_game_item_table` is
the *definition* table loaded from `tables/predmety.def`, stride **`0xbc`**, property flag word at
**`+0x20`** (`def-files-docs/predmety.md`). Every membership query in this class is:

```c
id != 0 && id < g_num_game_items && (g_game_item_table[id * 0xbc + 0x20] & mask) != 0
```

### 1b. Field evidence

| offset | field | conf | writer(s) | reader(s) | notes |
|---|---|---|---|---|---|
| `+0x00` | `can_carry_big_weapon` | H | `inventory_set_can_carry_big_weapon` | `inventory_can_accept_item`, `inventory_place_in_free_slot` | permits the big-weapon slot `+0x80` |
| `+0x04` | `in_car` | H | `inventory_set_in_car` | `inventory_set_in_car`, `inventory_build_cycle_list` | 1 while the owner sits in a car |
| `+0x08` | `extra_count` | H | - | `inventory_have_items`, `inventory_has_item_with_property` | gates the `+0x90` loop |
| `+0x0c` | `cycle_order` | H | `inventory_build_cycle_list` (erases the ids no longer carried, appends the new ones), `human_game_init`, `human_activate`, `inventory_reset` (`end = begin`); copy-assigned in `engine_new_mission` | `inventory_build_cycle_list`, `inventory_cycle_to_prev`, `inventory_cycle_to_next`, `inventory_dtor` (frees it) | the weapon-cycle order; a carried id keeps its place, so the order is stable ¹ |
| `+0x1c` | `cycle_current_id` | H | `inventory_reset` (`0xffffff`), `inventory_cycle_to_prev` / `_next` (the in-hand id, then the id stepped to) | `inventory_build_cycle_list`, `inventory_cycle_to_prev` / `_next` | the id the cycle stands on |
| `+0x20` | `in_hand` | H | `inventory_holster` (clears to 0) | both scanners | the in-hand-only query reads exactly this |
| `+0x30..+0x7f` | `belt[5]` | H | - | both scanners | stride `0x10`, exactly 5 iterations |
| `+0x80` | `big_weapon` | H | `inventory_place_in_free_slot`, `inventory_select_slot`, `inventory_set_can_carry_big_weapon` (spills it when the permission is withdrawn) | both scanners, `inventory_can_accept_item` | the one slot for an item with flag `0x200000`; it takes one only while `+0x00` is set or the item has flag `0x40000` (the sawed-off shotgun alone) ² |
| `+0x90..` | `extra[]` | H | - | both scanners | stride `0x10`, length from `+0x08` |
| `+0xd0` | `car_slot` | H | `inventory_holster` (stores the selection), `inventory_set_in_car` (empties it on entering a car) | `inventory_set_in_car` (re-inserts its item on leaving), `inventory_build_cycle_list` (counted only while `+0x04` is set), save pair | an `S_GameItem` |
| `+0xe0` | `report_to_console` | H | `inventory_reset` (0), `player_game_init` (1) | `inventory_holster`, `inventory_merge_ammo_into_slot` | gates the console messages textdb `0xd70`, `0xd6f` and `0xd6e`, so only the player is told |

¹ Elements are dwords: `vector_erase_range_dwords` indexes `+0x08` of the vector header as its end.
² `inventory_remove_slot`'s band selector 2 is `+0x80`, and the UI's kind 2 is "Coat" (§5).

## 2. Behaviour

- **Band selector.** `inventory_remove_slot` takes (0 in-hand / 1 belt / 2 `+0x80` / 3 overflow) and
  then calls `inventory_compact_slots`, which closes gaps in the belt and the overflow band.
- **Selection.** `inventory_select_slot(this, index, band, out_vector)` swaps the chosen band slot
  with the in-hand slot and spills what cannot be kept. The UI passes kind **0 Hands / 1 weapon /
  2 Coat / 3 item / 4 Car**.
- **Weapon cycling.** `inventory_build_cycle_list` collects the cyclable ids (in-hand gated on flag
  mask `0x24`, the five belt slots, `+0x80`, and the car slot `+0xd0` while in a car) and reconciles the
  cycle order at `+0x0c` with them; `inventory_cycle_to_prev` / `_next` step −1 / +1 with wrap and tail-call
  `inventory_select_by_id`.
- **Insertion.** `inventory_insert_new_item`: flag `0x2000` forces the in-hand slot; displaced items go
  to the caller's out-vector; `inventory_place_in_free_slot` (called twice) routes into a band.
  `inventory_can_accept_item` routes on flag bits `0x24` / `0x200000` / `0x2000` / `0x40000` and caps
  the overflow band at 4.
- **Ammo.** `inventory_merge_ammo_into_slot` folds a pickup's ammo into a carried weapon, capped at
  `predmety.def +0x64`, printing textdb `0xd6f` (already full) or `0xd6e` (already have it).
  `inventory_consume_round` decrements `+0x24`; `inventory_reload_magazine` moves `+0x28` into `+0x24`
  capped at `predmety.def +0x60` (item flag `0x2000000` selects a reserve-only variant);
  `inventory_reload_one_round` does `+0x24 += 1`, `+0x28 -= 1` below capacity;
  `inventory_can_reload_one_round` is the same predicate without the write.
- **Removal.** `inventory_remove_item_by_id` locates by id; `inventory_remove_item_by_ptr` by address
  (zeroes the slot's id). `inventory_collect_items_to_list` clears the caller's vector and appends a
  pointer to every occupied slot in all four bands, printing textdb `0xd48` ("Nothing found") when
  empty.
- **Save.** `inventory_savegame_get_size` returns `0xc4`. `inventory_savegame_save` writes the flags
  packed into one byte (value `& 0x7f` plus bit `0x80`), then `+0x20`, the belt `+0x30` and the
  `+0xd0` stash; `inventory_savegame_load` is the field-for-field inverse.
- **The armed query.** `ai_logs_tick` calls `inventory_has_item_with_property(true, 0x20)` twice to
  decide whether a target is aiming (feeds `ai_logs_get_aiming_weight`); a script predicate passes
  `0x20` twice as well — every located caller uses `0x20`. Bit `0x20` of the item flag word marks a ranged
  weapon (the firearms and the two throwables), bit `0x04` a melee weapon, so `0x24` is any weapon
  (`def-files-docs/predmety.md`).

## 3. Method coverage — PC ↔ PS2

Ordered by PC VA.

| PC VA | PC name | PS2 twin | status |
|---|---|---|---|
| `0x0048f790` | `inventory_dtor` | - | CONFIRMED ¹ |
| `0x00498970` | `inventory_reset` | - | CONFIRMED ¹ |
| `0x004a6090` | `inventory_get_in_hand_item` | - | CONFIRMED ¹ |
| `0x00559ba0` | `inventory_get_in_hand_item_def` | `GetWeaponInRightHand__11G_InventoryFv` | CONFIRMED ² |
| `0x00559bd0` | `inventory_get_left_hand_item_def` | `GetWeaponInLeftHand__11G_InventoryFv` | CONFIRMED ³ |
| `0x00559be0` | `inventory_select_slot` | `Select__11G_InventoryFUiUiPQ23gif22vector<10S_GameItem,8>` | CONFIRMED |
| `0x0055a1f0` | `inventory_select_by_id` | `SelectByID__11G_InventoryFUiPQ23gif22vector<10S_GameItem,8>` | CONFIRMED ⁴ |
| `0x0055a2c0` | `inventory_holster` | `Holster__11G_InventoryFPQ23gif22vector<>bb` | CONFIRMED |
| `0x0055a540` | `inventory_has_item_with_property` | `HaveSomeWeapon__11G_InventoryFbUi` | CONFIRMED |
| `0x0055a610` | `inventory_have_items` | `HaveItems__11G_InventoryFPQ23std52vector<10S_GameItem,…>Ui` | CONFIRMED |
| `0x0055a790` | `inventory_build_cycle_list` | `CyclePrepare__11G_InventoryFv` | CONFIRMED |
| `0x0055a910` | `inventory_cycle_to_prev` | `CycleUp__11G_InventoryFPQ23gif22vector<10S_GameItem,8>` | CONFIRMED |
| `0x0055a9b0` | `inventory_cycle_to_next` | `CycleDown__11G_InventoryFPQ23gif22vector<10S_GameItem,8>` | CONFIRMED |
| `0x0055aa40` | `inventory_contains_item` | `IsIn__11G_InventoryFUib` | CONFIRMED |
| `0x0055aac0` | `inventory_can_accept_item` | `HaveSpaceFor__11G_InventoryFUib` | STRUCTURAL ⁵ |
| `0x0055ac60` | `inventory_merge_ammo_into_slot` | `DobijNaboje__11G_InventoryFP10S_GameItemP10S_GameItem` | CONFIRMED |
| `0x0055ad00` | `inventory_place_in_free_slot` | - | CONFIRMED ⁶ |
| `0x0055ae40` | `inventory_insert_new_item` | `InsertNew__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8>` | CONFIRMED |
| `0x0055b2d0` | `inventory_insert_item` | `Insert__11G_InventoryFP10S_GameItemb` | CONFIRMED |
| `0x0055b580` | `inventory_remove_slot` | `RemoveByIdx__11G_InventoryFUiUi` | CONFIRMED |
| `0x0055b600` | `inventory_remove_item_by_id` | `Remove__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8>` | CONFIRMED |
| `0x0055b940` | `inventory_set_can_carry_big_weapon` | `SetCanHaveBigWeapon__11G_InventoryFbPQ23gif22vector<10S_GameItem,8>` | STRUCTURAL ⁵ |
| `0x0055bb80` | `inventory_set_in_car` | `SetInCar__11G_InventoryFbPQ23gif22vector<10S_GameItem,8>` | CONFIRMED ⁹ |
| `0x0055bbd0` | `inventory_collect_items_to_list` | `EnemyToList__11G_InventoryFPQ23gif23vector<P10S_GameItem,8>` | CONFIRMED |
| `0x0055bcd0` | `inventory_remove_item_by_ptr` | `RemoveByPtr__11G_InventoryFP10S_GameItem` | CONFIRMED |
| `0x0055bd40` | `inventory_get_active_weapon_ammo` | `GetAmmo__11G_InventoryFPi` | CONFIRMED |
| `0x0055bd80` | `inventory_get_ammo_by_id` | `GetAmmoByID__11G_InventoryFUiPi` | CONFIRMED |
| `0x0055be70` | `inventory_consume_round` | `Vystrel__11G_InventoryFv` | CONFIRMED |
| `0x0055be90` | `inventory_reload_magazine` | `Nabij__11G_InventoryFv` | CONFIRMED |
| `0x0055bf20` | `inventory_reload_one_round` | `NabijJeden__11G_InventoryFv` | CONFIRMED |
| `0x0055bf60` | `inventory_can_reload_one_round` | `MuzeNabitJeden__11G_InventoryFv` | CONFIRMED |
| `0x0055bfa0` | `inventory_compact_slots` | `_Sesyp__11G_InventoryFv` | STRUCTURAL ⁵ |
| `0x0055c020` | `inventory_savegame_get_size` | `SaveGameGetSize__11G_InventoryFv` | CONFIRMED |
| `0x0055c030` | `inventory_savegame_save` | `SaveGameSave__11G_InventoryFPPv` | CONFIRMED |
| `0x0055c150` | `inventory_savegame_load` | `SaveGameLoad__11G_InventoryFPPv` | CONFIRMED |
| - | - | `RemoveInHand__11G_InventoryFv` | INLINED ⁷ |
| - | - | `ItemPickUp__11G_InventoryF…` | INLINED ⁸ |

¹ Inline member emitted into `C_human`'s object (§4); no PS2 symbol.
² Reads the in-hand id at `+0x20` and returns `g_game_item_table + id*0xbc`, clamped to
`g_num_game_items` with the table base as the miss value. 9 of 27 callsites are `lea ecx,[esi+0x480]`.
³ The body is `return g_game_item_table;`: it never reads the inventory, because no left-hand slot
exists, so the answer is always the table's first record. Its one caller, `human_change_weapon_model`,
passes the human's inventory (`lea ecx,[esi+0x480]`).
⁴ Name approximate.
⁵ The PS2 body is an 8-byte stub, empty or a constant, so no code diff is possible (§6).
⁶ PC-only band router.
⁷ At its one call position, the throw event of the human animation callback, `human_notify_callback`
calls `inventory_remove_slot(0, 0)`: in the band layout the in-hand slot is always band 0, so the
method reduces to that one call.
⁸ Both PC callers, `player_do_action_button_on_foot` and `player_do_action_button_in_car`, build the
action-menu screen (`gui_action_menu_ctor`) at the call position, run it with `gui_menu_execute_modal`
and post-process the result exactly as the other build does after `ItemPickUp` returns (close code
`0xa8` on the PC). The PC always shows the menu (§6).
⁹ The PS2 body is an empty stub (§6), so the grade rests on the callers: 9 callsites on each
build, with the same two argument shapes, `(1, 0)` and `(0, &dropped)`, and the same non-empty
guard on the out-vector in the statement-aligned `human_intern_use_car` pair.

**Coverage:** all 33 PS2 methods accounted for — 28 CONFIRMED, 3 STRUCTURAL and 2 INLINED — plus one
PC-only method and three PC inline members. The 33 are every symbol whose class column is
`G_Inventory` in the PS2 symbol table.

## 4. Inline members in `C_human`'s object, and the heap inventory list

Because the inventory is embedded in `C_human`, MSVC emitted its inline members as COMDATs into
`C_human.cpp`'s object, far from the `inventory_*` cluster. The prefix follows the class, not the
translation unit.

| PC VA | PC name | what it does | receiver proof |
|---|---|---|---|
| `0x0048f790` | `inventory_dtor` | frees the only heap member, the vector at `+0x0c`, and nulls the triple | unwind funclet `add ecx, 0x480`; a second funclet on a stack `G_Inventory`; the `p->~G_Inventory(); operator delete(p);` pair in `game_inventory_stack_clear` |
| `0x00498970` | `inventory_reset` | clears every band in §1 order | `add ecx, 0x480`; 4 callsites: `human_ctor` (ctor position), `game_save_game_load`, `engine_new_mission`, `program_process_command` |
| `0x004a6090` | `inventory_get_in_hand_item` | returns the in-hand id and, optionally, the slot address | `+0x480` |

`vector_dispose_at_0x24` (`0x0048f7b0`), adjacent to `inventory_dtor`, is **not** a member: its unwind
funclets are `add ecx, 0x724` (`ai_loc`) and `add ecx, 0xe88` (`ai_logs`) — an ICF fold owned by
neither.

### 4a. The scene's list of heap inventories

`game_inventory_stack_clear` (`0x005eda20`) destroys and empties a **`std::vector<G_Inventory*>` at
scene `+0x2e54`**. The element type is byte-proven: `game_inventory_stack_push` does
`operator_new(0xe4)`, runs `inventory_reset`'s clear sequence, then copies a `C_human`'s `+0x480`
inventory member-wise — copying `+0x20..+0x2c` as four plain dwords immediately before the 5×4-dword
belt loop, which agrees with `in_hand` at `+0x20`.

## 5. The three GUI screens that drive `G_Inventory`

All three take a `G_Inventory *` and are the only UI that mutates one directly. Full write-ups:
`structs/game_menus.md` §4c, §4d, §4e.

| screen | design | prefix | how it reaches the inventory | opened by |
|---|---|---|---|---|
| inventory list | `0xf` | `gui_inventory_` | `G_Inventory` at screen `+0x848`, owner at `+0x84c` | the inventory key, from `player_ai_walk` / `_drive` / `_car_fire` |
| action menu | `0x10` | `gui_action_menu_` | `G_Inventory` at screen `+0x1048`, NPC vector at `+0x104c` | `player_do_action_button_on_foot` / `_in_car` |
| gun shop | `0x14` | `gui_gun_shop_` | `G_Inventory` at screen `+0x848` | script command `gunshop_menu`, dispatcher case `0x61` |

The fixed bands are visible in the UI component ids: `0x100` Hands, `0x102`..`0x106` the five weapon
slots, `0x107` Coat, `0x108` Car, and a cloned row per item from `0x200`. Selecting a row calls
`inventory_select_slot(index, kind, actor)` with the §2 kind enum. Item labels are textdb
`item_id + 0xdac`; the console pickup line is textdb `item_id + 0xd48`.

Item flags read from `g_game_item_table+0x20`:

| flag | effect |
|---|---|
| `0x800000` | excluded from the gun shop's catalogue |
| `0x2000` | the row shows a bare name instead of `"%s - %d/%d"` (no ammo counts) |
| `0x100` | the inventory list disables the row (present but not selectable) |

- `gui_action_menu_pick_up` treats the two sources differently: an option from the WORLD moves into
  the screen's taken-lists; an option from an NPC's inventory is removed from every NPC in the actor
  vector via `inventory_remove_item_by_ptr`.
- The gun shop's left band calls `inventory_remove_slot(index, kind)`, its right band
  `inventory_insert_item`. On failure it shows textdb `0x528` / `0x529` ("You do not have enough
  room." / "You must drop an item.") or `0x52b` when the item is already carried — these strings
  belong to the gun shop, not the inventory screen.

## 6. PC vs PS2

- **Owner offset:** PS2 `C_human+0x5cc` (93 receiver-adjust sites) ↔ PC `C_human+0x480`, delta
  `0x14c` — the same delta the `ai_sensors` sub-object shows in that band (`structs/entity.md`). PS2
  `SaveGameGetSize__7C_human` serializes `G_Inventory::SaveGameGetSize(this+0x5cc)`, which is what
  places the inventory in the person body.
- **The storage model differs.** PS2 keeps items in one `gif::vector<S_GameItem,8>` (begin, end, and
  an in-hand *index* at `+0xc`); PC uses four fixed bands. Loops therefore do not resemble each other,
  and matching on loop shape or callee set grades true twins as non-matches. What survives: the
  membership predicate, the flag offset, the signature and emission order — `inventory_has_item_with_property`
  was identified on exactly those. The same divergence explains:
  - `inventory_select_slot` ratio 23.9 and `inventory_insert_new_item` ratio 0.56;
  - `inventory_remove_item_by_ptr` ratio 2.29 (PS2's in-hand-index fixup and tail shift are absent);
  - `inventory_remove_slot`: the PS2's second `Ui` is inert;
  - `GetWeaponInRightHand` indexing the vector by the in-hand index, where PC reads a constant `+0x20`;
  - **no PS2 constructor symbol**: there is nothing for one to clear.
- **`S_GameItem` fields** were named from `GetAmmo__11G_InventoryFPi`, which runs the same four-way test
  but reads `slot+4` / `slot+8` of the indexed vector element; the PC reads `this+0x24` / `this+0x28`.
- **PS2 stubs:** `SetCanHaveBigWeapon`, `_Sesyp` and `SetInCar` are empty (`jr ra; nop`), and
  `HaveSpaceFor` is `jr ra; li v0, 1`, a constant yes that the decompiler shows as `{ return; }`.
  The three STRUCTURAL rows are graded on field + rank + arity:
  `inventory_can_accept_item` on arity 3↔3 and its rank between `IsIn` and `DobijNaboje`;
  `inventory_set_can_carry_big_weapon` because it writes field `[0]`, `inventory_set_in_car` writes
  `[1]`, and the PS2 emits `SetCanHaveBigWeapon` immediately before `SetInCar`;
  `inventory_compact_slots` because it sits immediately before `SaveGameGetSize` on both builds.
  `SetInCar` is the fourth stub, and `inventory_set_in_car` stays CONFIRMED on its callers (§3 ⁹).
- **No `G_Inventory` method is port-only.** All 33 have PC code: 31 twins and 2 inlined. The one
  inventory-related PS2 function inside a port-only subsystem is the free function
  `InternMissionInit_InterCity__FR11G_Inventory`, part of the city-part streaming code
  (`structs/mission.md` §5f).
- **Emission order is monotone:** `SelectByID < Holster < HaveSomeWeapon < HaveItems < Insert <
  SetInCar < GetAmmoByID` ↔ `0x0055a1f0 < 0x0055a2c0 < 0x0055a540 < 0x0055a610 < 0x0055b2d0 <
  0x0055bb80 < 0x0055bd80`.
- **Definition-table stride:** PC `0xbc`, PS2 `0xf0`; the flag word is at `+0x20` on both.
- **Translation unit:** PC emits the class inside `g_loadsave` (`structs/loadsave.md`); the PS2 keeps
  `G_Inventory` in a different file.
- **`ItemPickUp__11G_InventoryFP11G_Inventory…PQ23std56vector<P11G_Inventory,…>`** takes a
  `std::vector<G_Inventory*>`, which predicted the scene list of §4a. Its body builds a `GM_Menu`, runs `ExecuteMenu` and returns the close
  code (`0xb3`); for a single item it inserts directly without the menu, a path the PC does not have.
- `gui_action_menu_pick_up` is `GM_ItemPickUp::PickUp`; the world-vs-NPC asymmetry of §5 is identical
  statement for statement.
- **Czech method names:** `Vystrel` = a shot, `Nabij` = reload, `NabijJeden` = reload one,
  `MuzeNabitJeden` = can reload one, `DobijNaboje` = top up ammo, `_Sesyp` = (slang) collapse.
  `EnemyToList` has nothing to do with enemies, and `CycleUp` steps to the previous index. The PC
  names follow behaviour.

## 7. Related

`def-files-docs/predmety.md` (the item table) · `structs/human.md` (the owner) · `structs/ai_logs.md`
(`ai_logs_tick`'s armed query) · `structs/game_menus.md` (the three screens) · `structs/loadsave.md`
(the host TU) · `basic/artifacts/player-interactions.md` (the action menu).
