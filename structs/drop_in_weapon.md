# `C_drop_in_weapon` — the dropped item you can walk up to and pick up (kind `0x25`)

**Created** 2026-08-13 17:18 +0200 · **Modified** 2026-10-01 18:52 +0200

PC prefix **`drop_in_weapon_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x25`** |
| PC vtable | `.rdata 0x0063c280` (39 slots) |
| PC constructor | `drop_in_weapon_ctor` @ `0x0048a0d0` |
| PC instance size | **`0x3f8`** — `operator_new(0x3f8)` at all four creation sites |
| PC translation unit | `[0x0048a0d0, 0x0048a500)` — 8 function starts, all named |
| Base class | **`C_PhThing`** (`structs/physical_object.md`, which lists it as one of its five subclasses in §2), derived from `C_actor` |
| PS2 class | `C_drop_in_weapon`, 14 methods, all accounted (§4) |

## 1. What it is

An item lying in the world that the player can walk up to and interact with. The class
does two things and only two: it lets the shared physics base tumble the object until it
comes to rest, and it publishes an entry in the **`C_using_object` registry** — the list
that backs the player's action menu (`basic/artifacts/player-interactions.md`).

**The name is the class's, not a description of its contents.** `C_drop_in_weapon` is the
developers' name and the PC prefix records it, but the creation census shows that "weapon" is
only the common case. The four creation sites are a closed set: `drop_in_weapon_ctor` has exactly
four `E8` callers in the image and no data reference.

| creator | what is actually being dropped | item (`+0x3b0..+0x3bc`, §2) |
|---|---|---|
| `human_drop_out_items` | the weapons a dying human was carrying | the inventory `S_GameItem`, clip and reserve clamped to `predmety.def` `+0x60` / `+0x64` |
| `human_do_box_drop` | a carried crate the player puts down | id 0; `+0x3b4` = the carrier's box owner value `human+0x6fc` |
| `dog_spawn_severed_head` | a decapitated dog's head | id `0x20` ("Dogs head"), ammo 0 |
| `program_process_command` | whatever a mission script asks for | `human_unlinkfromhand`: the weapon id and the two ammo arguments |

So the class is *"an item dropped into the world"*. Do not write "weapon" into a field
description without checking which creator filled it.

## 2. `+0x3b0..+0x3bc` is an embedded `S_GameItem`

The class's own band opens with one **`S_GameItem`** (`structs/inventory.md` §1a), the 16-byte
inventory instance: item id (`predmety.def`, a `uint16` at `+0x3b0`), clip ammo `+0x3b4`, reserve
ammo `+0x3b8`, and the item's last dword `+0x3bc`. `human_drop_out_items` copies the dying human's
inventory slot into it whole.

The id is the switch the whole class turns on:

* **non-zero** ⇒ the object registers a using-object record and gets an action prompt;
* **zero** ⇒ `drop_in_weapon_on_act_state_changed` takes the other arm entirely and adds
  the frame to the **static collision-box placement** instead
  (`game_box_add(frame, +0x3b4)` + `game_box_recompile`) — i.e. it becomes scenery, not a pickup.
  On this arm `+0x3b4` is the box owner value `human_do_box_drop` stores, not ammo.

What identifies the id's type rather than merely its role is a range guard that hits the shipped
item table exactly: in `predmety.def`, ids **5..15 are the guns and throwables** (§5).

`+0x3bc` never reaches a consumer: both record builders copy it to the record's `+0x34` and then
overwrite that dword with the record's own address (§3a).

## 3. Field layout

Everything below `+0x3b0` belongs to `C_actor` / `C_PhThing` and is documented in
`structs/actor.md` and `structs/physical_object.md`.

| offset | size | field | evidence |
|---|---|---|---|
| `+0x10` | 4 | `kind` = `0x25` | written by the ctor |
| `+0x68` | 4 | `I3D_frame*` — the item's scene frame | `C_actor` base field |
| `+0xc0` | 4 | physics flag word, forced to `0x80000` at scene start | `drop_in_weapon_game_init` |
| `+0x3b0` | 2 | **item id** (`predmety.def`) | `S_GameItem+0x00`, §2 |
| `+0x3b4` | 4 | clip ammo; the box owner value on a box | `S_GameItem+0x04`, §2 |
| `+0x3b8` | 4 | reserve ammo | `S_GameItem+0x08` |
| `+0x3bc` | 4 | `S_GameItem+0x0c` | overwritten in the published copy, §2 |
| `+0x3c0` | `0x38` | **embedded `S_using_object` record** | §3a; filled in place by `drop_in_weapon_game_init` |

`0x3c0 + 0x38 = 0x3f8`, which is exactly the `operator_new` size at all four creation
sites. The layout and the allocation size confirm each other independently.

### 3a. The published `S_using_object` record

The same record is built two ways: **in place** at `+0x3c0` by `game_init`, and on the
**heap** (`operator_new(0x38)`) by `on_act_state_changed` when the item becomes active
after the scene has already started. Both then hand it to the registry.

Field names follow `structs/using_object.md` §1a.

| record offset | field | in place (`game_init`) | heap (`on_act_state_changed`) |
|---|---|---|---|
| `+0x00..+0x08` | world position | the frame's position, Y raised by 0.4 ¹ | the same |
| `+0x0c` | prompt radius | `0.5f` | `0.5f` |
| `+0x10` | parent transform | 0 (world space) | 0 |
| `+0x14` | flags | 0 | 3: the registry owns the record and the frame |
| `+0x18` | `I3D_frame*` | the item's frame | the item's frame |
| `+0x1c` | owning actor | this `C_drop_in_weapon` | 0: the actor is already removed |
| `+0x28..+0x34` | `S_GameItem` | a 16-byte copy of `+0x3b0..+0x3bc` | the same |
| `+0x34` | `S_GameItem+0x0c` | the record's own address | the same |

¹ So the prompt floats above the item rather than inside the floor.

The heap path runs when the item's act state becomes 1, the at-rest state (`structs/box_new.md`
§1a): it removes the actor (`game_invalidate_actor` + `game_remove_temporary_actor`) and leaves the
registry as the owner of the record and the frame.

### 3b. The pickup hand-off

1. `using_object_find_near_objects` copies each record's `S_GameItem` into the action-menu list, so
   every list entry carries the record's address in its `+0x0c`.
2. `gui_action_menu_tick` passes the chosen `S_GameItem` to `inventory_insert_new_item` and prints
   the textdb line `0xd48 + item id`; `gui_action_menu_pick_up` records the take.
3. `player_do_action_button_on_foot` follows each taken entry's `+0x0c` back to its record: a
   non-zero `+0x1c` hands the item actor to `game_remove_temporary_actor`; a zero `+0x1c` goes to
   `using_object_del_object`, which deletes the heap record and its frame (flags 3). The in-place
   record leaves the registry in `drop_in_weapon_on_act_state_changed`'s act-state-2 arm, which
   unregisters it and releases the frame.

## 4. Method coverage — 14/14 PS2 methods accounted

| PC VA | PC name | slot | PS2 twin | status |
|---|---|---|---|---|
| `0x0048a0d0` | `drop_in_weapon_ctor` | - | `__ct__16C_drop_in_weaponFv` | CONFIRMED |
| `0x0048a140` | `drop_in_weapon_scalar_deleting_dtor` | 16 | - | STRUCTURAL ¹ |
| `0x0048a160` | `drop_in_weapon_dtor` | - | `__dt__16C_drop_in_weaponFv` | CONFIRMED |
| `0x0048a170` | `drop_in_weapon_update` | 14 | `Update__16C_drop_in_weaponFUi` | CONFIRMED |
| `0x0048a1d0` | `drop_in_weapon_on_act_state_changed` | 27 | `ChangeState__16C_drop_in_weaponFv` | CONFIRMED |
| `0x0048a360` | `drop_in_weapon_game_init` | 25 | `GameInit__16C_drop_in_weaponFv` | CONFIRMED |
| `0x0048a440` | `drop_in_weapon_game_done` | 26 | `GameDone__16C_drop_in_weaponFv` | CONFIRMED |
| `0x0048a4f0` | `stub_retfalse_args32_0048a4f0` | 31 | `Hit__16C_drop_in_weaponF…` | STRUCTURAL ² |

¹ MSVC-only construct. ² Shared stub, §4a.

### 4a. The eight PS2 stubs, and why none of them gets a twin stamped on it

Eight of the fourteen PS2 methods are 8-byte bodies; they are not all empty (§5):

| PS2 method | PS2 8-byte body actually is | PC realisation | shared across |
|---|---|---|---|
| `AI` | tail jump → `C_PhThing::AI` | `thunk_physical_object_ai_tick` (`0x004cde70`), slot 13 | 2 vtables |
| `Init` | tail jump → `C_PhThing::Init` | `thunk_physical_object_init` (`0x004cdea0`), slot 18 | 2 vtables |
| `NetDirect` | empty `return;` | `thunk_physical_object_net_apply_remote_state` (`0x004cde90`), slot 15 | 3 vtables |
| `DuplicateFrom` | empty `return;` | `thunk_stub_noop_args4_00461180` (`0x004773f0`), slot 21 | many |
| `Collision` | `return 0;` | `stub_retfalse_args24_00482690`, slot 30 | **16** vtables |
| `Hit` | `return 0;` | `stub_retfalse_args32_0048a4f0`, slot 31 | **13** vtables |
| `SaveData` | `return 1;` | `stub_rettrue_args4_00477710`, slot 19 | 37 vtables |
| `LoadData` | `return 1;` | `stub_rettrue_args4_00477710`, slot 20 | (same symbol) |

**The mapping is recorded here and nowhere else, deliberately.** Every PC symbol in that table is
shared by several classes (a scan of `.rdata` for each address as a vtable pointer), so a
`PS2 twin:` line on any of them would assert an ownership the scan refutes. A slot table can hold a
many-to-one relation honestly; a docblock field cannot.

## 5. PC vs PS2

* **The eight 8-byte PS2 stubs of §4a are not all empty** — on MIPS a tail jump is also 8 bytes,
  and Ghidra renders it as its *target's* body, so each one has to be read.
* `NetDirect` is empty on PS2 while the PC slot forwards to 1696 bytes of dead reckoning: the PS2
  port gutted the network direction, so "PS2 has nothing here" does not mean the PC has nothing.

* **Field bridge:** PC `+0x3b0` ↔ PS2 `+0x640`, fixed by the constructor diff and re-used
  by every other row. The class band starts where `C_PhThing` ends on each build.
* **The item-id proof:** PS2's `ChangeState` guards the registration on `5 < id && id < 0xf`,
  which is the guns-and-throwables range of `predmety.def` (§2).
* **The published record is 0x38 bytes on PC and 0x40 on PS2** — PS2's position is a
  16-byte `S_vector` where the PC uses three dwords. `operator_new(0x38)` vs
  `__nw__FUi(0x40)` in the same statement of the same method.
* **`ChangeState` has a third arm on PS2 that the PC does not have:** for
  `5 < id < 0xf` with both leading payload dwords zero, PS2 destroys the frame instead of
  registering the item. The PC always registers.
* **`GameInit` writes two extra bytes on PS2** (`+0x616`, `+0x617`) with no PC equivalent.
* The heap record's flags value is `0xf` on PS2 against the PC's 3 (`structs/using_object.md` §4).
* The physics flag magic `0x80000` (PC `+0xc0`) is shared with PS2 `+0x538`.
* Size ratios, PC/PS2: ctor 0.70 · Update 0.92 · ChangeState 0.68 · **GameInit 0.98** ·
  GameDone 0.79. Every one inside the ~1.0–1.6 band once MSVC's inlined `memset` and the
  extra PS2 arm are accounted for.

## 6. Related

`structs/physical_object.md` (the base, §2 lists this class) ·
`structs/dropout.md` (the sibling TU) ·
`structs/entity.md` §1 (the kind enum) ·
`structs/using_object.md` (the registry) ·
`basic/artifacts/player-interactions.md` (the action menu this class feeds).
