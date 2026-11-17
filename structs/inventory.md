# `G_Inventory` — the per-person item and weapon inventory

**Created** 2026-07-26 21:35 +0200 · **Modified** 2026-11-04 09:26 +0100

The item and weapon store every person carries: cops, gangsters, pedestrians and the player alike.
It is not a heap object of its own but a `0xe4`-byte member **embedded at `C_human+0x480`**: six
independent callsites in two methods (`inventory_holster` ×2, `inventory_has_item_with_property`
×4) pass `lea ecx,[reg+0x480]`, and a `lea` is an embedded member, never a pointer. The class has
no vtable and no constructor; `human_ctor` runs `inventory_reset` in the constructor's place.

| | |
|---|---|
| PC prefix | `inventory_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0xe4` (`game_inventory_stack_push` allocates `operator_new(0xe4)` for a heap copy, §5) |
| PC constructor | - |
| PC translation unit | `[0x00559ba0, 0x0055c270)`, a band inside the `g_loadsave` unit (`structs/loadsave.md`); the next function is `enabled_cars_have_car`. Three inline members sit in `C_human`'s unit (§3a, §3c) |
| PS2 class | `G_Inventory` (tag `11G_Inventory`), 33 methods, all placed (§3, §7) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct G_Inventory {                        // 0xe4 bytes, embedded at C_human+0x480
    /* +0x00 */ uint32_t   can_carry_big_weapon;    // [H] permits the big-weapon slot +0x80
    /* +0x04 */ uint32_t   in_car;                  // [H] 1 while the owner sits in a car
    /* +0x08 */ uint32_t   extra_count;             // [H] live entries of extra[], 0..4
    /* +0x0c */ std::vector<uint32_t> cycle_order;  // [H] {alloc, begin, end, cap}: the weapon-cycle order, item ids
    /* +0x1c */ uint32_t   cycle_current_id;        // [H] the id the cycle stands on; 0xffffff = none
    /* +0x20 */ S_GameItem in_hand;                 // [H] band 0: the item held ("Hands")
    /* +0x30 */ S_GameItem belt[5];                 // [H] band 1: the five weapon slots
    /* +0x80 */ S_GameItem big_weapon;              // [H] band 2: one big-weapon slot ("Coat")
    /* +0x90 */ S_GameItem extra[4];                // [H] band 3: up to four further items
    /* +0xd0 */ S_GameItem car_slot;                // [H] the item put away while the owner is in a car ("Car")
    /* +0xe0 */ uint8_t    report_to_console;       // [H] 1 on the player's inventory only
} G_Inventory;
```

**The four storage bands.** Every slot holds one `S_GameItem` (§1a), and every method that looks
for an item walks the bands in the same order: `in_hand`, `belt`, `big_weapon`, `extra`, and
`car_slot` where the method cares about the car.

* **`belt[5]` — why five.** The bands mirror the inventory screen's permanent rows: component ids
  `0x102`..`0x106` are the five weapon slots (§6), and every belt loop runs exactly five iterations
  with a `0x10` stride (`inventory_place_in_free_slot`, `inventory_select_by_id`, both scanners).
  Entry *i* is weapon slot *i*; `inventory_compact_slots` closes gaps, so occupied entries come
  first.
* **`extra[4]` — why four.** `inventory_can_accept_item` refuses a fourth-band item once
  `extra_count` reaches 4 and `inventory_place_in_free_slot` refuses at `3 < count`; entry *i* is
  the *i*-th non-weapon item in pickup order, and only the first `extra_count` entries are live.
  On screen they are the cloned rows from component `0x200` (§6).

`big_weapon` takes an item with flag `0x200000` only while `can_carry_big_weapon` is set or the
item has flag `0x40000` (the sawed-off shotgun alone).

### 1a. `S_GameItem` — one carried item, 0x10 bytes

```c
typedef struct S_GameItem {                   // 0x10 bytes
    /* +0x00 */ uint16_t item_id;             // [H] index into g_game_item_table; 0 = empty slot
    /* +0x02 */ uint16_t magazine_out;        // [H] the spent magazine or clip has been dropped
    /* +0x04 */ uint32_t clip_ammo;           // [H] rounds in the weapon
    /* +0x08 */ uint32_t reserve_ammo;        // [H] spare rounds
    /* +0x0c */ S_using_object *source_record; // [H] the world record the item came from
} S_GameItem;
```

`S_GameItem` is what an inventory *holds*. Its definition is the `g_game_item_table` record loaded
from `tables/predmety.def`, stride **`0xbc`**, property flag word at **`+0x20`**
(`def-files-docs/predmety.md`). A membership query of the class is

```
id != 0 && id < g_num_game_items && (g_game_item_table[id * 0xbc + 0x20] & mask) != 0
```

`inventory_has_item_with_property` leaves out `id != 0`, and `inventory_have_items` leaves out the
bound.

`source_record` is the pointer the world copy of the item carries back to its own
`S_using_object` record (`structs/using_object.md` §1a, the record's `+0x34`): the action-menu list
copies the whole 16 bytes, and the inventory keeps them as they arrive. No `inventory_*` method
reads it. `player_do_action_button_on_foot` follows the menu and taken copies back to the record, and
`human_drop_out_items` copies a dropped slot's 16 bytes into the new `C_drop_in_weapon`.

## 2. Field evidence

Measured by a census of every `this+N` access in the `inventory_*` functions, by body reads of
every method that reaches a band through an alias, and by a census of `C_human+0x480..+0x563`
accesses outside the class (its users go through `lea ecx,[reg+0x480]`, which the census counts
as a call, not a field access).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | can_carry_big_weapon | H | `inventory_reset` (1), `inventory_set_can_carry_big_weapon`, `inventory_savegame_load`, `human_game_init` (1), `human_activate` (1), `engine_new_mission` | `inventory_can_accept_item`, `inventory_place_in_free_slot`, `inventory_insert_item`, `inventory_select_slot`, `inventory_savegame_save`, `engine_new_mission` | withdrawing it spills `big_weapon` (§4) |
| +0x04 | in_car | H | `inventory_reset` (0), `inventory_set_in_car`, `inventory_savegame_load` | `inventory_set_in_car`, `inventory_build_cycle_list`, `inventory_select_slot`, `inventory_select_by_id`, `inventory_holster`, `inventory_savegame_save` | saved as bit `0x80` of the record's flag byte |
| +0x08 | extra_count | H | `inventory_reset` (0), `inventory_place_in_free_slot` (+1), `inventory_insert_item` (+1), `inventory_remove_item_by_id` (−1), `inventory_compact_slots`, `inventory_savegame_load` | `inventory_select_by_id`, `inventory_has_item_with_property`, `inventory_have_items`, `inventory_contains_item`, `inventory_can_accept_item`, `inventory_remove_slot`, `inventory_collect_items_to_list`, `inventory_remove_item_by_ptr`, `inventory_get_ammo_by_id`, `inventory_insert_item`, `inventory_place_in_free_slot`, `inventory_remove_item_by_id`, `inventory_compact_slots`, `inventory_savegame_save` | every `extra[]` loop is bounded by it |
| +0x0c | cycle_order | H | `inventory_build_cycle_list` (erases the ids no longer carried, appends new ones); `end = begin` in `inventory_reset`, `human_game_init`, `human_activate`; freed by `inventory_dtor` and inline in `human_dtor`; copy-assigned in `engine_new_mission` | `inventory_build_cycle_list`, `inventory_cycle_to_prev`, `inventory_cycle_to_next` | ¹ |
| +0x1c | cycle_current_id | H | `inventory_reset` (`0xffffff`), `inventory_cycle_to_prev`, `inventory_cycle_to_next` | `inventory_build_cycle_list`, `inventory_cycle_to_prev`, `inventory_cycle_to_next` | the in-hand id, then the id stepped to |
| +0x20 | in_hand | H | `inventory_reset`, `inventory_select_slot`, `inventory_insert_new_item`, `inventory_insert_item`, `inventory_remove_slot`, `inventory_remove_item_by_id`, `inventory_remove_item_by_ptr`, `inventory_holster`, `inventory_consume_round`, `inventory_reload_magazine`, `inventory_reload_one_round`, `inventory_savegame_load`, `human_inventory_truncate` (the ammo pair), `human_start_weapon_effect` / `human_ai` (`magazine_out`) | `inventory_get_in_hand_item`, `inventory_get_in_hand_item_def`, every band walker, `inventory_get_active_weapon_ammo`, `inventory_can_reload_one_round`, `inventory_savegame_save`, `entity_process_command` (`clip_ammo`), `human_change_weapon_model` (`magazine_out`) | an `S_GameItem` (§1a) |
| +0x30 | belt | H | `inventory_reset`, `inventory_place_in_free_slot`, `inventory_select_slot`, `inventory_insert_item`, `inventory_remove_slot`, `inventory_remove_item_by_id`, `inventory_remove_item_by_ptr`, `inventory_compact_slots`, `inventory_savegame_load` | every band walker, `inventory_savegame_save` | five `S_GameItem`, stride `0x10` |
| +0x80 | big_weapon | H | `inventory_reset`, `inventory_place_in_free_slot`, `inventory_select_slot`, `inventory_insert_item`, `inventory_insert_new_item`, `inventory_remove_slot`, `inventory_remove_item_by_id`, `inventory_remove_item_by_ptr`, `inventory_set_can_carry_big_weapon` (spills it), `inventory_savegame_load` | every band walker, `inventory_can_accept_item`, `inventory_set_can_carry_big_weapon`, `inventory_savegame_save` | the one slot for flag `0x200000` |
| +0x90 | extra | H | `inventory_reset`, `inventory_place_in_free_slot`, `inventory_select_slot`, `inventory_insert_item`, `inventory_remove_slot`, `inventory_remove_item_by_id`, `inventory_remove_item_by_ptr`, `inventory_compact_slots`, `inventory_savegame_load` | every band walker, `inventory_savegame_save` | stride `0x10`, live length `+0x08` |
| +0xd0 | car_slot | H | `inventory_reset`, `inventory_holster` (stores the held item), `inventory_set_in_car` (empties it on entering a car), `inventory_select_slot`, `inventory_savegame_load` | `inventory_set_in_car` (re-inserts its item on leaving), `inventory_build_cycle_list` (counted only while `+0x04` is set), `inventory_select_slot`, `inventory_select_by_id`, `inventory_holster`, `inventory_savegame_save` | an `S_GameItem` |
| +0xe0 | report_to_console | H | `inventory_reset` (0), `human_game_init` (0), `human_activate` (0), `player_game_init` (1), `engine_new_mission` (carries it across) | `inventory_holster`, `inventory_merge_ammo_into_slot` | gates the console lines textdb `0xd70`, `0xd6f` and `0xd6e`, so only the player is told |

¹ The elements are dwords: `vector_erase_range_dwords` indexes `+0x08` of the vector header as its
end. A carried id keeps its place, so the cycle order is stable.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | item_id | H | the band writers of §2 (each copies or clears the whole record) | every band walker; `inventory_get_in_hand_item_def` (`g_game_item_table + id * 0xbc`) | 0 = the slot is empty |
| +0x02 | magazine_out | H | `human_start_weapon_effect` (1), `human_ai` (0, the reload animation event) | `human_start_weapon_effect`, `human_change_weapon_model`, `human_drop_out_items` | ² |
| +0x04 | clip_ammo | H | `inventory_consume_round` (−1), `inventory_reload_magazine`, `inventory_reload_one_round` (+1), `inventory_merge_ammo_into_slot` (the pickup's to 0), `human_inventory_truncate` | `inventory_get_active_weapon_ammo` (return value), `inventory_get_ammo_by_id`, `inventory_can_reload_one_round`, `inventory_merge_ammo_into_slot`, `entity_process_command`, `human_drop_out_items` | capped at `predmety.def +0x60` |
| +0x08 | reserve_ammo | H | `inventory_reload_magazine`, `inventory_reload_one_round` (−1), `inventory_merge_ammo_into_slot`, `human_inventory_truncate` | `inventory_get_active_weapon_ammo` (out-parameter), `inventory_reload_magazine`, `inventory_can_reload_one_round`, `inventory_merge_ammo_into_slot` | capped at `predmety.def +0x64` |
| +0x0c | source_record | H | the band writers of §2, as part of the record; the value is the world record's own address (`structs/using_object.md` §1a) | `player_do_action_button_on_foot` (the menu and taken copies), `human_drop_out_items` (a block copy) | §1a; no `inventory_*` method reads it (Open item 2) |

² `human_start_weapon_effect` sets the latch when it drops the spent magazine or clip of the
in-hand gun as a physics prop and drops no second one while it is set; the reload animation event
in `human_ai` clears it.

## 3. Method family

Coverage: all **33** PS2 `G_Inventory` methods are placed — **31 twins** in the tables below
(28 CONFIRMED, 3 STRUCTURAL) and **2** realised inline in callers outside this doc
(`RemoveInHand`, `ItemPickUp`; PS2-only table in §7). Four PC functions have no PS2 symbol: the
three inline members `inventory_dtor`, `inventory_reset`, `inventory_get_in_hand_item`, and the
band router `inventory_place_in_free_slot`.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0048f790 | `inventory_dtor` | instance | - | - | - | an inline member emitted into `C_human`'s unit: frees `cycle_order`, the only heap member, and nulls its triple. Receiver proof: an unwind funclet `add ecx, 0x480`, a second funclet on a stack `G_Inventory`, and the `p->~G_Inventory(); operator delete(p);` pair in `game_inventory_stack_clear` |

### 3b. Accessors

The class has no accessor of its own: `inventory_get_in_hand_item` also stores the slot address,
and the two `*_item_def` readers index the item table.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00498970 | `inventory_reset` | - | - | - | an inline member in `C_human`'s unit (`add ecx, 0x480`): clears every band in §1 order, sets `+0x00` = 1 and `+0x1c` = `0xffffff`. Its four callers: `human_ctor` (the constructor position), `game_save_game_load`, `engine_new_mission`, `program_process_command` |
| 0x004a6090 | `inventory_get_in_hand_item` | - | - | - | an inline member in `C_human`'s unit (`+0x480`): returns the in-hand id and, when asked, the slot's address |
| 0x00559ba0 | `inventory_get_in_hand_item_def` | `GetWeaponInRightHand__11G_InventoryFv` | 0x0047b250 | CONFIRMED | the in-hand id's `g_game_item_table` record, clamped to `g_num_game_items` with the table base as the miss value; 9 of 27 callsites pass `lea ecx,[esi+0x480]` |
| 0x00559bd0 | `inventory_get_left_hand_item_def` | `GetWeaponInLeftHand__11G_InventoryFv` | 0x0047b2a0 | CONFIRMED | `return g_game_item_table;` on both builds (8 bytes on the PS2): no left-hand slot exists, so the answer is always the table's first record. Its one caller, `human_change_weapon_model`, passes the human's inventory |
| 0x00559be0 | `inventory_select_slot` | `Select__11G_InventoryFUiUiPQ23gif22vector<10S_GameItem,8>` | 0x0047b2b0 | CONFIRMED | `(index, band, out_vector)`: swaps the chosen band slot with `in_hand` and spills what cannot be kept |
| 0x0055a1f0 | `inventory_select_by_id` | `SelectByID__11G_InventoryFUiPQ23gif22vector<10S_GameItem,8>` | 0x0047b2f0 | CONFIRMED | finds the id band by band and selects it through `inventory_select_slot`; true at once when it is in hand |
| 0x0055a2c0 | `inventory_holster` | `Holster__11G_InventoryFPQ23gif22vector<10S_GameItem,8>bb` | 0x0047b370 | CONFIRMED | puts the held item away, into `car_slot` while in a car |
| 0x0055a540 | `inventory_has_item_with_property` | `HaveSomeWeapon__11G_InventoryFbUi` | 0x0047b5e0 | CONFIRMED | `(in_hand_only, mask)`: any carried item with a flag of the mask (§4) |
| 0x0055a610 | `inventory_have_items` | `HaveItems__11G_InventoryFPQ23std52vector<10S_GameItem,Q23std23allocator<10S_GameItem>>Ui` | 0x0047b6d0 | CONFIRMED | collects the carried items matching a mask |
| 0x0055a790 | `inventory_build_cycle_list` | `CyclePrepare__11G_InventoryFv` | 0x0047b7c0 | CONFIRMED | reconciles `cycle_order` with the cyclable ids (§4) |
| 0x0055a910 | `inventory_cycle_to_prev` | `CycleUp__11G_InventoryFPQ23gif22vector<10S_GameItem,8>` | 0x0047ba00 | CONFIRMED | one step back with wrap, then `inventory_select_by_id` |
| 0x0055a9b0 | `inventory_cycle_to_next` | `CycleDown__11G_InventoryFPQ23gif22vector<10S_GameItem,8>` | 0x0047bdb0 | CONFIRMED | one step forward with wrap, then `inventory_select_by_id` |
| 0x0055aa40 | `inventory_contains_item` | `IsIn__11G_InventoryFUib` | 0x0047c140 | CONFIRMED | `(id, include_hand)`: membership by id |
| 0x0055aac0 | `inventory_can_accept_item` | `HaveSpaceFor__11G_InventoryFUib` | 0x0047c1e0 | STRUCTURAL | routes on flags `0x24` / `0x200000` / `0x2000` / `0x40000` and caps band 3 at 4; the PS2 body is a constant yes (§7) |
| 0x0055ac60 | `inventory_merge_ammo_into_slot` | `DobijNaboje__11G_InventoryFP10S_GameItemP10S_GameItem` | 0x0047c1f0 | CONFIRMED | folds a pickup's ammo into a carried weapon's reserve, capped at `predmety.def +0x64` |
| 0x0055ad00 | `inventory_place_in_free_slot` | - | - | - | the PC-only band router: weapon → first free belt slot, flag `0x200000` → `big_weapon`, anything else → `extra[count++]` |
| 0x0055ae40 | `inventory_insert_new_item` | `InsertNew__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8>` | 0x0047c2b0 | CONFIRMED | flag `0x2000` forces the hand; displaced items go to the caller's out-vector |
| 0x0055b2d0 | `inventory_insert_item` | `Insert__11G_InventoryFP10S_GameItemb` | 0x0047cad0 | CONFIRMED | merges ammo into a carried copy or places the item through `inventory_place_in_free_slot` |
| 0x0055b580 | `inventory_remove_slot` | `RemoveByIdx__11G_InventoryFUiUi` | 0x0047cf90 | CONFIRMED | `(index, band)`: empties one slot (band 0 hand, 1 belt, 2 `big_weapon`, 3 `extra`), then `inventory_compact_slots` |
| 0x0055b600 | `inventory_remove_item_by_id` | `Remove__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8>` | 0x0047d070 | CONFIRMED | locates by id and moves the item to the out-vector |
| 0x0055b940 | `inventory_set_can_carry_big_weapon` | `SetCanHaveBigWeapon__11G_InventoryFbPQ23gif22vector<10S_GameItem,8>` | 0x0047d2c0 | STRUCTURAL | writes `+0x00`; on withdrawal re-places or spills `big_weapon`; the PS2 body is empty (§7) |
| 0x0055bb80 | `inventory_set_in_car` | `SetInCar__11G_InventoryFbPQ23gif22vector<10S_GameItem,8>` | 0x0047d2d0 | CONFIRMED | writes `+0x04`; empties `car_slot` on entering, re-inserts its item on leaving. The PS2 body is empty; the grade rests on the callers (§7) |
| 0x0055bbd0 | `inventory_collect_items_to_list` | `EnemyToList__11G_InventoryFPQ23gif23vector<P10S_GameItem,8>` | 0x0047d2e0 | CONFIRMED | clears the caller's vector and appends a pointer to every occupied slot; textdb `0xd48` ("Nothing found") when empty |
| 0x0055bcd0 | `inventory_remove_item_by_ptr` | `RemoveByPtr__11G_InventoryFP10S_GameItem` | 0x0047d4a0 | CONFIRMED | zeroes the id of the slot at that address |
| 0x0055bd40 | `inventory_get_active_weapon_ammo` | `GetAmmo__11G_InventoryFPi` | 0x0047d5a0 | CONFIRMED | returns the in-hand clip, writes the reserve to the out-parameter |
| 0x0055bd80 | `inventory_get_ammo_by_id` | `GetAmmoByID__11G_InventoryFUiPi` | 0x0047d620 | CONFIRMED | the same for a carried id |
| 0x0055be70 | `inventory_consume_round` | `Vystrel__11G_InventoryFv` | 0x0047d6d0 | CONFIRMED | in-hand clip −1 |
| 0x0055be90 | `inventory_reload_magazine` | `Nabij__11G_InventoryFv` | 0x0047d730 | CONFIRMED | moves reserve into clip, capped at `predmety.def +0x60`; item flag `0x2000000` selects a reserve-only variant |
| 0x0055bf20 | `inventory_reload_one_round` | `NabijJeden__11G_InventoryFv` | 0x0047d840 | CONFIRMED | clip +1, reserve −1, below capacity |
| 0x0055bf60 | `inventory_can_reload_one_round` | `MuzeNabitJeden__11G_InventoryFv` | 0x0047d8e0 | CONFIRMED | the same test without the write |
| 0x0055bfa0 | `inventory_compact_slots` | `_Sesyp__11G_InventoryFv` | 0x0047d950 | STRUCTURAL | closes the gaps in `belt` and `extra`; the PS2 body is empty (§7) |
| 0x0055c020 | `inventory_savegame_get_size` | `SaveGameGetSize__11G_InventoryFv` | 0x0047d960 | CONFIRMED | returns `0xc4` |
| 0x0055c030 | `inventory_savegame_save` | `SaveGameSave__11G_InventoryFPPv` | 0x0047d970 | CONFIRMED | the `0xc4`-byte record of §4 |
| 0x0055c150 | `inventory_savegame_load` | `SaveGameLoad__11G_InventoryFPPv` | 0x0047da50 | CONFIRMED | its field-for-field inverse; `extra_count` is recounted from the loaded slots |

### 3d. Free helpers

The class's range holds no free helper. `vector_dispose_at_0x24` (`0x0048f7b0`), next to
`inventory_dtor`, is not a member: its unwind funclets are `add ecx, 0x724` (`ai_loc`) and
`add ecx, 0xe88` (`ai_logs`), an ICF fold that neither owns.

## 4. Behaviour

- **Band selector.** `inventory_remove_slot` takes 0 in-hand / 1 belt / 2 `big_weapon` / 3 `extra`
  and then calls `inventory_compact_slots`, which closes gaps in the belt and in `extra`.
- **Selection.** `inventory_select_slot(this, index, band, out_vector)` swaps the chosen slot with
  the in-hand slot and spills what cannot be kept. The UI passes kind **0 Hands / 1 weapon / 2 Coat
  / 3 item / 4 Car**; kind 2 is `big_weapon`, kind 4 `car_slot`.
- **Weapon cycling.** `inventory_build_cycle_list` collects the cyclable ids (in-hand gated on flag
  mask `0x24`, the five belt slots, `big_weapon`, and `car_slot` while in a car) and reconciles
  `cycle_order` with them; `inventory_cycle_to_prev` / `_next` step −1 / +1 with wrap and
  tail-call `inventory_select_by_id`.
- **Insertion.** `inventory_insert_new_item`: flag `0x2000` forces the in-hand slot; displaced
  items go to the caller's out-vector; `inventory_place_in_free_slot` (called twice) routes into a
  band. `inventory_can_accept_item` routes on flag bits `0x24` / `0x200000` / `0x2000` /
  `0x40000` and caps `extra` at 4.
- **Ammo.** `inventory_merge_ammo_into_slot` adds the pickup's clip and reserve to the carried
  weapon's reserve, capped at `predmety.def +0x64`, and on the player's inventory prints textdb
  `0xd6f` ("The weapon is fully loaded") while the carried reserve is below the cap, else `0xd6e`
  (the weapon is carried). `inventory_consume_round` decrements `clip_ammo`;
  `inventory_reload_magazine` moves `reserve_ammo` into `clip_ammo` capped at
  `predmety.def +0x60`; `inventory_reload_one_round` moves one round below capacity.
- **Removal.** `inventory_remove_item_by_id` locates by id; `inventory_remove_item_by_ptr` by
  address. `inventory_collect_items_to_list` appends a pointer to every occupied slot in all four
  bands, printing textdb `0xd48` ("Nothing found") when there is none.
- **Save record, `0xc4` bytes.** A flag dword (`can_carry_big_weapon`, with bit `0x80` added while
  `in_car`; the loader splits it as `& 0x7f` and `& 0x80`), then `0xc0` bytes cleared and filled
  with `in_hand`, `car_slot`, the five `belt` slots, `big_weapon` and the `extra_count` live
  `extra` entries.
- **The armed query.** `ai_logs_tick` calls `inventory_has_item_with_property(true, 0x20)` twice
  to decide whether a target is aiming (`ai_logs_get_aiming_weight`); a script predicate passes
  `0x20` as well, and every located caller uses `0x20`. Bit `0x20` of the item flag word marks a
  ranged weapon (the firearms and the two throwables), bit `0x04` a melee weapon, so `0x24` is any
  weapon (`def-files-docs/predmety.md`).

## 5. The scene's list of heap inventories

`game_inventory_stack_clear` (`0x005eda20`) destroys and empties a **`std::vector<G_Inventory*>`
at scene `+0x2e54`**. The element type is byte-proven: `game_inventory_stack_push` calls
`operator_new(0xe4)`, runs `inventory_reset`'s clear sequence, then copies a `C_human`'s `+0x480`
inventory member-wise, copying `+0x20..+0x2c` as four plain dwords immediately before the
5×4-dword belt loop, which agrees with `in_hand` at `+0x20`.

`engine_new_mission` carries the player's inventory across a mission change by hand: it saves
`+0x00`, the cycle vector and `+0xe0` to the stack, resets the member, and writes them back after
the new scene is open (`structs/mission.md` §7b).

## 6. The three GUI screens that drive `G_Inventory`

All three take a `G_Inventory *` and are the only UI that changes one directly. Full write-ups:
`structs/game_menus.md` §7c, §7d, §7e.

| Screen | Design | Prefix | How it reaches the inventory | Opened by |
|---|---|---|---|---|
| inventory list | `0xf` | `gui_inventory_` | `G_Inventory` at screen `+0x848`, the caller's removed-items vector at `+0x84c` | the inventory key, from `player_ai_walk` / `_drive` / `_car_fire` |
| action menu | `0x10` | `gui_action_menu_` | `G_Inventory` at screen `+0x1048`, NPC vector at `+0x104c` | `player_do_action_button_on_foot` / `_in_car` |
| gun shop | `0x14` | `gui_gun_shop_` | `G_Inventory` at screen `+0x848` | script command `gunshop_menu`, dispatcher case `0x61` |

The bands are visible in the UI component ids: `0x100` Hands, `0x102`..`0x106` the five weapon
slots, `0x107` Coat, `0x108` Car, and a cloned row per item from `0x200`. Selecting a row calls
`inventory_select_slot(index, kind, removed_items)` with the §4 kind enum. Item labels are textdb
`item_id + 0xdac`; the console pickup line is textdb `item_id + 0xd48`.

Item flags read from `g_game_item_table+0x20`:

| Flag | Effect |
|---|---|
| `0x800000` | excluded from the gun shop's catalogue |
| `0x2000` | the row shows a bare name instead of `"%s - %d/%d"` (no ammo counts) |
| `0x100` | the inventory list disables the row (present but not selectable) |

- `gui_action_menu_pick_up` treats the two sources differently: an option from the WORLD moves
  into the screen's taken-lists; an option from an NPC's inventory is removed from every NPC in the
  actor vector via `inventory_remove_item_by_ptr`.
- The gun shop's left band calls `inventory_remove_slot(index, kind)`, its right band
  `inventory_insert_item`. On failure it shows textdb `0x528` / `0x529` ("You do not have enough
  room." / "You must drop an item.") or `0x52b` when the item is carried; these strings belong to
  the gun shop, not to the inventory screen.

## 7. PC vs PS2

- **Owner offset:** PS2 `C_human+0x5cc` (93 receiver-adjust sites) ↔ PC `C_human+0x480`, delta
  `0x14c`, the same delta the `ai_sensors` sub-object shows in that band (`structs/entity.md`).
  PS2 `SaveGameGetSize__7C_human` serializes `G_Inventory::SaveGameGetSize(this+0x5cc)`, which is
  what places the inventory in the person body.
- **The storage model differs.** The PS2 keeps items in one `gif::vector<S_GameItem,8>` (begin,
  end, and an in-hand *index* at `+0xc`); the PC uses the four bands of §1. Loops therefore do not
  resemble each other, and matching on loop shape or callee set grades true twins as non-matches.
  What survives: the membership predicate, the flag offset, the signature and the emission order;
  `inventory_has_item_with_property` was identified on exactly those. The same divergence explains
  `inventory_select_slot` (size ratio 23.9), `inventory_insert_new_item` (0.56),
  `inventory_remove_item_by_ptr` (2.29, the PS2's in-hand-index fixup and tail shift are absent),
  `inventory_select_by_id` (the PS2 walks one vector, the PC four bands), the PS2's second `Ui` of
  `RemoveByIdx` being inert, `GetWeaponInRightHand` indexing the vector by the in-hand index where
  the PC reads `+0x20`, and the absence of a PS2 constructor: there is nothing for one to clear.
- **`S_GameItem` fields** were named from `GetAmmo__11G_InventoryFPi`, which runs the same
  four-way test but reads `slot+4` / `slot+8` of the indexed vector element. The PS2
  `DobijNaboje` also writes the leftover ammo back through `slot+0xc` into the world object's
  `+0x34` / `+0x38`, which names `source_record`; the PC merge leaves the world record alone.
- **PS2 stubs:** `SetCanHaveBigWeapon`, `_Sesyp` and `SetInCar` are empty (`jr ra; nop`), and
  `HaveSpaceFor` is `jr ra; li v0, 1`, a constant yes that the decompiler shows as `{ return; }`.
  The three STRUCTURAL rows are graded on field + rank + arity: `inventory_can_accept_item` on
  arity 3↔3 and its rank between `IsIn` and `DobijNaboje`; `inventory_set_can_carry_big_weapon`
  because it writes field `[0]`, `inventory_set_in_car` writes `[1]`, and the PS2 emits
  `SetCanHaveBigWeapon` immediately before `SetInCar`; `inventory_compact_slots` because it sits
  immediately before `SaveGameGetSize` on both builds. `inventory_set_in_car` stays CONFIRMED on
  its callers: 9 callsites on each build, with the same two argument shapes, `(1, 0)` and
  `(0, &dropped)`, and the same non-empty guard on the out-vector in the statement-aligned
  `human_intern_use_car` pair.
- **Emission order is monotone:** `SelectByID < Holster < HaveSomeWeapon < HaveItems < Insert <
  SetInCar < GetAmmoByID` ↔ `0x0055a1f0 < 0x0055a2c0 < 0x0055a540 < 0x0055a610 < 0x0055b2d0 <
  0x0055bb80 < 0x0055bd80`.
- **Definition-table stride:** PC `0xbc`, PS2 `0xf0`; the flag word is at `+0x20` on both.
- **Translation unit:** the PC emits the class inside `g_loadsave` (`structs/loadsave.md`); the
  PS2 keeps `G_Inventory` in a different file.
- **No `G_Inventory` method is port-only.** All 33 have PC code. The one inventory-related PS2
  function inside a port-only subsystem is the free function
  `InternMissionInit_InterCity__FR11G_Inventory`, part of the city-part streaming code
  (`structs/mission.md` §8f).
- `gui_action_menu_pick_up` is `GM_ItemPickUp::PickUp`; the world-versus-NPC asymmetry of §6 is
  identical statement for statement.
- **The developers' method names** are partly Czech: `Vystrel` = a shot, `Nabij` = reload,
  `NabijJeden` = reload one, `MuzeNabitJeden` = can reload one, `DobijNaboje` = top up ammo,
  `_Sesyp` = (slang) collapse. `EnemyToList` has nothing to do with enemies, and `CycleUp` steps to
  the previous index. The PC names follow behaviour.

**PS2 methods realised inline in callers outside this doc:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0047cf60 | `RemoveInHand__11G_InventoryFv` | at its one call position, the throw event of the human animation callback, `human_notify_callback` calls `inventory_remove_slot(0, 0)`: in the band layout the in-hand slot is always band 0, so the method reduces to that one call |
| 0x0047dc30 | `ItemPickUp__11G_InventoryFP11G_InventoryPQ23gif22vector<10S_GameItem,8>PQ23gif22vector<10S_GameItem,8>PQ23gif22vector<10S_GameItem,8>PQ23std56vector<P11G_Inventory,Q23std25allocator<P11G_Inventory>>` | both PC callers, `player_do_action_button_on_foot` and `player_do_action_button_in_car`, build the action-menu screen (`gui_action_menu_ctor`) at the call position, run it with `gui_menu_execute_modal` and post-process the result as the PS2 does after `ItemPickUp` returns (close code `0xa8` on the PC, `0xb3` on the PS2). The PS2 inserts a single item without the menu; the PC always shows it. The method's `std::vector<G_Inventory*>` parameter predicted the scene list of §5 |

## 8. Related

- `def-files-docs/predmety.md` — the item table.
- `structs/human.md` — the owner.
- `structs/using_object.md` — the world record behind `source_record`.
- `structs/ai_logs.md` — `ai_logs_tick`'s armed query.
- `structs/game_menus.md` — the three screens.
- `structs/loadsave.md` — the host unit.
- `basic/artifacts/player-interactions.md` — the action menu.

## 9. Open items

1. **In game: can a fifth `extra` item be picked up?** `inventory_can_accept_item` caps band 3 at
   four entries (§1); picking up a fifth non-weapon item should be refused.
2. **Whether the copy inside a slot is ever followed.** The menu and taken copies are (§1a). Searched
   for a slot's copy: every `inventory_*` body, `human_drop_out_items` and `gui_action_menu_pick_up`
   show block copies only. Other users reach the member through `lea reg,[this+0x480]` and a small
   displacement, which a scan of the outer offsets `+0x4ac` and up cannot see.
