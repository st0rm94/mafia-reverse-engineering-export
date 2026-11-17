# `C_drop_in_weapon` — the dropped item you can walk up to and pick up (kind `0x25`)

**Created** 2026-08-13 17:18 +0200 · **Modified** 2026-10-29 21:46 +0100

An item lying in the world that the player can walk up to and take. The class lets the shared
physics base tumble the object until it comes to rest, and publishes one record in the
**`C_using_object` registry**, the list behind the player's action menu
(`basic/artifacts/player-interactions.md`). The developers' name says "weapon"; the creators of §4
drop boxes and a dog's head through it too.

| | |
|---|---|
| PC prefix | `drop_in_weapon_` |
| PC kind (`+0x10`) | `0x25` |
| PC vtable | `g_drop_in_weapon_vt` @ `0x0063c280` (39 slots) |
| PC base class | `C_PhThing` (`structs/physical_object.md`), derived from `C_actor` |
| PC instance size | `0x3f8` (`operator_new(0x3f8)` at all four creation sites; the record at `+0x3c0` ends at `+0x3f8`) |
| PC constructor | `drop_in_weapon_ctor` @ `0x0048a0d0` |
| PC translation unit | `[0x0048a0a0, 0x0048a500)`, guard to guard: 10 starts, all named and all this class's |
| PS2 class | `C_drop_in_weapon` (tag `16C_drop_in_weapon`), 14 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x3b0` is the `C_PhThing` base (`structs/physical_object.md`
§1, `structs/actor.md` §1). The class uses five base fields: the vtable `+0x00` and kind `+0x10`
(the ctor), the act state `+0x1c` (read by `drop_in_weapon_on_act_state_changed` and
`drop_in_weapon_game_done`), the frame `+0x68`, and the physics flag word `+0xc0`, which
`drop_in_weapon_game_init` sets to `0x80000`.

```c
typedef struct C_drop_in_weapon {             // 0x3f8 bytes; C_PhThing base +0x00..+0x3b0
    /* +0x3b0 */ uint16_t        item_id;          // [H] predmety.def id; 0 = a dropped box (§4)
    /* +0x3b2 */ uint16_t        magazine_out;     // [H] S_GameItem +0x02, carried along
    /* +0x3b4 */ uint32_t        clip_ammo;        // [H] rounds; on a box the box owner value
    /* +0x3b8 */ uint32_t        reserve_ammo;     // [H] spare rounds
    /* +0x3bc */ S_using_object *source_record;    // [H] S_GameItem +0x0c; replaced in the copy
    /* +0x3c0 */ S_using_object  use_record;       // [H] the action-menu record, built in place (§5)
} C_drop_in_weapon;
```

**`+0x3b0..+0x3bc` is one `S_GameItem`** (`structs/inventory.md` §1a), the 16-byte inventory
instance: `human_drop_out_items` copies the dying human's inventory slot into it whole, and both
record builders copy it whole into the record's `+0x28..+0x34`. The four fields keep the
`S_GameItem` names because that is what they are; `clip_ammo` carries a second meaning only when
`item_id` is 0 (§4).

**`use_record` is one `S_using_object`** (`structs/using_object.md` §1a, `0x38` bytes). The class
fills it in place at scene start and refreshes its position every frame; §5 lists what each record
field receives.

## 2. Field evidence

Measured by a census of every `this + N` access in the `drop_in_weapon_*` functions, a `.text`
displacement census for `+0x3b0..+0x3bc` (which finds the four creators), and body reads.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x3b0 | item_id | H | `drop_in_weapon_ctor` (0), `human_drop_out_items` (the whole item), `human_do_box_drop` (0), `dog_spawn_severed_head` (`0x20`), `program_process_command` | `drop_in_weapon_game_init`, `drop_in_weapon_on_act_state_changed`, `drop_in_weapon_game_done` | the switch of §4 |
| +0x3b2 | magazine_out | H | `drop_in_weapon_ctor` (0), `human_drop_out_items` (the whole item) | `drop_in_weapon_game_init`, `drop_in_weapon_on_act_state_changed` | copied into the record as one dword with `item_id` |
| +0x3b4 | clip_ammo | H | `drop_in_weapon_ctor` (0), `human_drop_out_items` (clamped), `human_do_box_drop` (`human+0x6fc`), `dog_spawn_severed_head` (0), `program_process_command` | `drop_in_weapon_game_init`, `drop_in_weapon_on_act_state_changed`, `human_drop_out_items` | ¹ |
| +0x3b8 | reserve_ammo | H | `drop_in_weapon_ctor` (0), `human_drop_out_items` (clamped), `dog_spawn_severed_head` (0), `program_process_command` | `drop_in_weapon_game_init`, `drop_in_weapon_on_act_state_changed`, `human_drop_out_items` | ¹ |
| +0x3bc | source_record | H | `drop_in_weapon_ctor` (0), `human_drop_out_items` (the whole item), `dog_spawn_severed_head` (0), `program_process_command` (0) | `drop_in_weapon_game_init`, `drop_in_weapon_on_act_state_changed` | both copy it to the record's `+0x34` and then overwrite that dword with the record's address |
| +0x3c0 | use_record | H | `drop_in_weapon_ctor` (record `+0x10` and `+0x18..+0x34`), `drop_in_weapon_game_init`, `drop_in_weapon_update` (the position) | `using_object_add_object` (from `drop_in_weapon_game_init`), `using_object_del_object` (from `drop_in_weapon_on_act_state_changed`) | §5 |

¹ `human_drop_out_items` clamps the two counts for an item id above 1 and below the item-table
count: `clip_ammo` to the item record's `+0x60` and `reserve_ammo` to its `+0x64`
(`g_game_item_table`, `0xbc`-byte records loaded from `predmety.def`).

## 3. Method family

14 of 14 PS2 methods mapped: 6 twins and 8 base stubs on shared PC functions; 3 PC-only starts
(the scalar-deleting destructor and the unit's static-init pair).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0048a0a0 | `crt_static_init_guard_0048a0a0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x0048a0c0 | `crt_atexit_register_0048a0c0` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x0048a0d0 | `drop_in_weapon_ctor` | instance | `__ct__16C_drop_in_weaponFv` | 0x00161330 | CONFIRMED | `physical_object_ctor`, zeroes the item and the record fields of §2, vtable, kind `0x25`; four callsites (§4) |
| 0x0048a140 | `drop_in_weapon_scalar_deleting_dtor` | instance | - | - | - | slot 16; MSVC-only construct |
| 0x0048a160 | `drop_in_weapon_dtor` | instance | `__dt__16C_drop_in_weaponFv` | 0x001613d0 | CONFIRMED | the non-deleting arm |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__16C_drop_in_weaponFP7C_actor` | 0x00161720 | base-stub | slot 21; the PS2 body is an empty `return`; the PC thunk fills slot 21 of 9 vtables (§6) |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__16C_drop_in_weaponFP7C_chunk` | 0x00161920 | base-stub | slot 19; `return 1` on both builds; 37 vtable slots share the PC symbol |
| 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__16C_drop_in_weaponFP7C_chunk` | 0x00161930 | base-stub | slot 20; the same PC symbol |
| 0x00482690 | `stub_retfalse_args24_00482690` | `Collision__16C_drop_in_weaponFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00161900 | base-stub | slot 30; `return 0` on both builds; 16 vtables |
| 0x0048a170 | `drop_in_weapon_update` | `Update__16C_drop_in_weaponFUi` | 0x00161440 | CONFIRMED | slot 14; `physical_object_update`, then copies the frame position, Y + 0.4, into the record (§5) |
| 0x0048a1d0 | `drop_in_weapon_on_act_state_changed` | `ChangeState__16C_drop_in_weaponFv` | 0x001614c0 | CONFIRMED | slot 27; act state 1 hands the item to a heap record or the box placement, 2 unregisters (§5) |
| 0x0048a360 | `drop_in_weapon_game_init` | `GameInit__16C_drop_in_weaponFv` | 0x00161730 | CONFIRMED | slot 25; `physical_object_game_init`, flag word `0x80000`, and for a non-zero `item_id` builds and registers the in-place record |
| 0x0048a440 | `drop_in_weapon_game_done` | `GameDone__16C_drop_in_weaponFv` | 0x00161820 | CONFIRMED | slot 26; snapshots the frame pose into the `C_actor` base, `physical_object_game_done`, and releases the frame of an item that is still active |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__16C_drop_in_weaponF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00161910 | base-stub | slot 31; `return 0` on both builds; 13 vtables share the PC symbol, which sits in this unit |
| 0x004cde70 | `thunk_physical_object_ai_tick` | `AI__16C_drop_in_weaponFUi` | 0x00161430 | base-stub | slot 13; both builds forward to the `C_PhThing` AI; 2 vtables |
| 0x004cde90 | `thunk_physical_object_net_apply_remote_state` | `NetDirect__16C_drop_in_weaponFUi` | 0x001614b0 | base-stub | slot 15; the PS2 body is empty, the PC slot forwards to the `C_PhThing` dead reckoning; 3 vtables |
| 0x004cdea0 | `thunk_physical_object_init` | `Init__16C_drop_in_weaponFP9I3D_frame` | 0x00161710 | base-stub | slot 18; both builds forward to the `C_PhThing` Init; 2 vtables |

Every PC symbol on a `base-stub` row is shared by several vtables (a scan of `.rdata` for the
address as a pointer gives the counts above), so no `PS2 twin:` line sits on any of them: the
slot table holds a many-to-one relation that a docblock field cannot.

### 3d. Free helpers

The unit has no free helper.

## 4. What is dropped, and the `item_id` switch

The four creation sites are a closed set: `drop_in_weapon_ctor` has exactly four `E8` callers and
no data reference.

| Creator | What is dropped | The item band (`+0x3b0..+0x3bc`) |
|---|---|---|
| `human_drop_out_items` | the weapons a dying human carried | the inventory `S_GameItem`, the counts clamped (§2 ¹) |
| `human_do_box_drop` | a carried crate the player puts down | `item_id` 0; `clip_ammo` = the carrier's box owner value `human+0x6fc` |
| `dog_spawn_severed_head` | a decapitated dog's head | `item_id` `0x20` ("Dogs head"), the rest 0 |
| `program_process_command` | whatever a mission script asks for | `item_id` and the two counts from the command, `source_record` 0 |

So the class is *an item dropped into the world*; "weapon" is only the common case.

**`item_id` decides what the object is.** Non-zero: the object registers an action-menu record
and the player can take it. Zero: `drop_in_weapon_on_act_state_changed` adds the frame to the
static collision-box placement instead (`game_box_add(frame, clip_ammo)` + `game_box_recompile`),
so a put-down crate becomes scenery; on that arm `clip_ammo` is the box owner value
`human_do_box_drop` stored.

## 5. The published `S_using_object` record

The record is built two ways: **in place** at `+0x3c0` by `drop_in_weapon_game_init`, and on the
**heap** (`operator_new(0x38)`) by `drop_in_weapon_on_act_state_changed` when the body goes to act
state 1 (asleep: `C_PhThing` puts a body at rest there, `structs/physical_object.md`). Both are
handed to `using_object_add_object`, and only for a non-zero `item_id`. Field names follow
`structs/using_object.md` §1a.

| Record offset | Field | In place (`drop_in_weapon_game_init`) | Heap (`drop_in_weapon_on_act_state_changed`) |
|---|---|---|---|
| +0x00 | pos | the frame position, Y + 0.4; `drop_in_weapon_update` repeats the copy every frame | the frame position, Y + 0.4, once |
| +0x0c | radius_sq | `0.5f` | `0.5f` |
| +0x10 | transform | 0 (world space) | 0 |
| +0x14 | flags | 0 | 3: the registry deletes the record and releases the frame |
| +0x18 | frame | the item's frame | the item's frame |
| +0x1c | owner | this `C_drop_in_weapon` | 0: the actor is removed in the same call |
| +0x20 | frame_filter | 0 (the ctor) | 0 |
| +0x24 | save_excluded | 0 (the ctor) | 0 |
| +0x28 | the `S_GameItem` | a copy of `+0x3b0..+0x3bc` | the same |
| +0x34 | self | the record's own address | the same |

The Y offset makes the prompt float above the item rather than inside the floor.

**Act state 1** removes the actor (`game_invalidate_actor` + `game_remove_temporary_actor`),
unregisters the in-place record, and either registers the heap record (the registry then owns the
record and the frame) or, for `item_id` 0, places the box. **Act state 2** unregisters the
in-place record and, for a non-zero `item_id`, releases the frame and clears `+0x68`.
`drop_in_weapon_game_done` releases the frame of an item whose act state is still 0.

### 5a. The pickup hand-off

1. `using_object_find_near_objects` copies each record's `S_GameItem` into the action-menu list, so
   every list entry carries the record's address in its `+0x0c`.
2. `gui_action_menu_tick` passes the chosen `S_GameItem` to `inventory_insert_new_item` and prints
   the textdb line `0xd48 + item id`; `gui_action_menu_pick_up` records the take.
3. `player_do_action_button_on_foot` follows each taken entry's `+0x0c` back to its record: a
   non-zero `owner` hands the item actor to `game_remove_temporary_actor`; a zero `owner` goes to
   `using_object_del_object`, which deletes the heap record and its frame (flags 3).

## 6. PC vs PS2

* **Field bridge:** PC `+0x3b0` ↔ PS2 `+0x640`, set by the constructor diff and used by every other
  row. The class band starts where `C_PhThing` ends on each build.
* **The item-id proof:** PS2's `ChangeState` guards the registration on `5 < id && id < 0xf`, which
  is the guns-and-throwables range of `predmety.def`.
* **`ChangeState` has a third arm on PS2 that the PC does not have:** for `5 < id < 0xf` with both
  leading payload dwords zero, the PS2 destroys the frame instead of registering the item. The PC
  always registers.
* **The published record is `0x38` bytes on PC and `0x40` on PS2** — the PS2 position is a 16-byte
  `S_vector` where the PC uses three dwords: `operator_new(0x38)` vs `__nw__FUi(0x40)` in the same
  statement of the same method. The heap record's flags value is `0xf` on PS2 against the PC's 3
  (`structs/using_object.md` §5).
* **`GameInit` writes two extra bytes on PS2** (`+0x616`, `+0x617`) with no PC equivalent. The
  physics flag magic `0x80000` (PC `+0xc0`) is PS2 `+0x538`.
* **The eight 8-byte PS2 stubs of §3c are not all empty**: on MIPS a tail jump is also 8 bytes and
  Ghidra renders it as its target's body. `AI` and `Init` are tail jumps to `C_PhThing`;
  `NetDirect` and `DuplicateFrom` are empty `return`; `Collision` and `Hit` return 0; `SaveData`
  and `LoadData` return 1. `NetDirect` is empty on PS2 while the PC slot forwards to 1696 bytes of
  dead reckoning, so "PS2 has nothing here" does not mean the PC has nothing.
* Size ratios, PC/PS2 (from the two symbol tables): ctor 0.70 · Update 0.92 · ChangeState 0.68 ·
  GameInit 0.98 · GameDone 0.79, inside the usual band once MSVC's inlined `memset` and the extra
  PS2 arm are accounted for.

## 7. Related

`structs/physical_object.md` (the base) · `structs/dropout.md` (the next unit) ·
`structs/entity_kind.md` §1 (the kind enum) · `structs/using_object.md` (the registry) ·
`structs/inventory.md` §1a (`S_GameItem`) · `basic/artifacts/player-interactions.md` (the action
menu this class feeds).

## 8. Open items

None.
