# `C_drop_in_weapon` — the dropped item you can walk up to and pick up (kind `0x25`)

**Created** 2026-08-13 17:18 +0200 · **Modified** 2026-09-17 13:44 +0200

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
only the common case. All four creation sites:

| creator | what is actually being dropped |
|---|---|
| `human_drop_out_items` | the weapons a dying human was carrying |
| `human_do_box_drop` | a carried crate the player puts down |
| `dog_spawn_severed_head` | a decapitated dog's head |
| `program_process_command` | whatever a mission script asks for |

So the class is *"an item dropped into the world"*. Do not write "weapon" into a field
description without checking which creator filled it.

## 2. `+0x3b0` is a `predmety.def` item id

The class's first own field is a **short** at `+0x3b0`, and it is the switch the whole
class turns on:

* **non-zero** ⇒ the object registers a using-object record and gets an action prompt;
* **zero** ⇒ `drop_in_weapon_on_act_state_changed` takes the other arm entirely and adds
  the frame to the **static collision-box placement** instead
  (`game_box_add` + `game_box_recompile`) — i.e. it becomes scenery, not a pickup.

What identifies the field's type rather than merely its role is a range guard that hits the
shipped item table exactly: in `predmety.def`, ids **5..15 are the guns and throwables** (§5).

`+0x3b4`, `+0x3b8` and `+0x3bc` are three more designer dwords copied verbatim into the
published record; nothing in either build reads them back, so their meanings are **[U]**.

## 3. Field layout

Everything below `+0x3b0` belongs to `C_actor` / `C_PhThing` and is documented in
`structs/actor.md` and `structs/physical_object.md`.

| offset | size | field | evidence |
|---|---|---|---|
| `+0x10` | 4 | `kind` = `0x25` | written by the ctor |
| `+0x68` | 4 | `I3D_frame*` — the item's scene frame | `C_actor` base field |
| `+0xc0` | 4 | physics flag word, forced to `0x80000` at scene start | `drop_in_weapon_game_init` |
| `+0x3b0` | 2 | **item id** (`predmety.def`) | §2 |
| `+0x3b4` | 4 | designer dword 1 **[U]** | copied to record `+0x2c` |
| `+0x3b8` | 4 | designer dword 2 **[U]** | copied to record `+0x30` |
| `+0x3bc` | 4 | designer dword 3 **[U]** | copied to record `+0x34` |
| `+0x3c0` | `0x38` | **embedded `S_using_object` record** | §3a; filled in place by `drop_in_weapon_game_init` |

`0x3c0 + 0x38 = 0x3f8`, which is exactly the `operator_new` size at all four creation
sites. The layout and the allocation size confirm each other independently.

### 3a. The published `S_using_object` record

The same record is built two ways: **in place** at `+0x3c0` by `game_init`, and on the
**heap** (`operator_new(0x38)`) by `on_act_state_changed` when the item becomes active
after the scene has already started. Both then hand it to the registry.

| record offset | field | value |
|---|---|---|
| `+0x00..+0x08` | world position | the frame's world position, Y raised by 0.4 ¹ |
| `+0x0c` | prompt radius | `0.5f` (`0x3f000000`), literal on both builds |
| `+0x10` | **[U]** | 0 |
| `+0x14` | **[U]** | 0 in place / 3 on the heap path |
| `+0x18` | `I3D_frame*` | the item's own frame |
| `+0x1c` | **[U]** | 0 |
| `+0x28..+0x34` | the four designer values | `+0x3b0`, `+0x3b4`, `+0x3b8`, `+0x3bc` |
| `+0x34` | self-pointer | overwritten with the record's own address |

¹ So the prompt floats above the item rather than inside the floor.

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

Eight of the fourteen PS2 methods are 8-byte bodies. **They are not all empty** — on MIPS a tail
jump is also 8 bytes, and Ghidra renders it as its *target's* body, so each one has to be read:

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

`NetDirect` is empty on PS2 while the PC slot forwards to 1696 bytes of dead reckoning: the PS2
port gutted the network direction, so "PS2 has nothing here" does not mean the PC has nothing.

## 5. PC vs PS2

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

## 7. Open items

| item | state |
|---|---|
| designer dwords `+0x3b4` / `+0x3b8` / `+0x3bc` | copied into the published record and read by nobody located on either build; the consumer would be inside `C_using_object` |
| record fields `+0x10`, `+0x14`, `+0x1c` | **[U]**; `+0x14` is 0 in place, 3 on the heap path, and `0xf` on PS2 — likely an action/menu-kind selector |
| `car_apply_net_event_msg` (`0x0059c690`) | the third `C_DropOut` creator; whether it bears on this class's script creation path is untraced |
| item id → player inventory | nothing traced links `+0x3b0` to the inventory the player receives on pickup; the hand-off is on the `C_using_object` side |
