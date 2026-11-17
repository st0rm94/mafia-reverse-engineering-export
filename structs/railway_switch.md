# `C_turnout` — the railway turnout (points / track switch, kind `0x18`)

**Created** 2026-08-17 15:33 +0200 · **Modified** 2026-10-30 12:24 +0100

A **turnout** is the railway engineering term for the movable track junction, the points. This class
is a swinging blade, two alternative rail pieces and a navigation-graph edit: at each end stop it
moves one edge of the `C_web_path` graph, so every wagon that rebuilds its path takes the other branch.

| | |
|---|---|
| PC prefix | `railway_switch_` |
| PC kind (`+0x10`) | `0x18` |
| PC vtable | `g_railway_switch_vt` @ `0x0063cee0` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x1a0` (`operator_new(0x1a0)` in `mission_create_actor`) |
| PC constructor | `railway_switch_ctor` @ `0x00508150` |
| PC translation unit | `[0x00508120, 0x00509400)`, guard to guard: 18 starts, all named |
| PS2 class | `C_turnout` (tag `9C_turnout`), 17 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base (`structs/actor.md` §1). The class
uses the kind `+0x10`, the cached position `+0x24` and forward row `+0x30`, and the frame `+0x68`.

```c
typedef struct C_turnout {                    // 0x1a0 bytes; C_actor base +0x00..+0x70
    /* +0x70 */ float    blade_travel;        // [H] radians either side of centre (ctor 0.5)
    /* +0x74 */ float    blade_speed;         // [H] radians per millisecond (ctor 0.004)
    /* +0x78 */ uint32_t sound_id;            // [H] sounds.dat id of the switching noise (ctor 0x85)
    /* +0x7c */ uint32_t web_node_common;     // [H] node id on both connections (ctor 0x7b)
    /* +0x80 */ uint32_t web_node_branch_a;   // [H] connected at rest position 0 (ctor 0x7c)
    /* +0x84 */ uint32_t web_node_branch_b;   // [H] connected at rest position 1 (ctor 0x7d)
    /* +0x88 */ char     rail_a_name[0x20];   // [H] frame shown at rest position 0 (ctor "rail1")
    /* +0xa8 */ char     rail_b_name[0x20];   // [H] frame shown at rest position 1 (ctor "rail2")
    /* +0xc8 */ uint8_t  start_at_b;          // [H] 0: GameInit enters rest 0; else rest 1
    /* +0xcc */ S_matrix rotation;            // [H] the frame's rotation matrix, for the centre
    /* +0x10c */ S_using_object use_record;   // [H] the action-menu record, 0x38 bytes (§1a)
    /* +0x144 */ float   blade_angle;         // [H] the current angle, radians
    /* +0x148 */ tDynamicCollObject coll;     // [H] the collision record, 0x48 bytes (§1b)
    /* +0x190 */ uint32_t state;              // [H] §2a
    /* +0x194 */ I3D_sound *switch_sound;     // [H] started by railway_switch_start_switching
    /* +0x198 */ I3D_frame *rail_a;           // [H] resolved from rail_a_name; may be 0
    /* +0x19c */ I3D_frame *rail_b;           // [H] resolved from rail_b_name; may be 0
} C_turnout;
```

**The name arrays are `0x20` bytes** because `railway_switch_save_data` and `_load_data` move exactly
`0x20` bytes each: each holds a frame name, in a record of that width in the scene data.

### 1a. `use_record` — what it holds

`railway_switch_game_init` fills the `S_using_object` (`structs/using_object.md` §1a) and registers
it: position = the frame's world position, `radius_sq` (`+0x118`) = 1.5, `owner` (`+0x128`) = this,
`item_id` (`+0x134`) = 1, `action_code` (`+0x138`) = `0x7d2`, textdb 2002 *"Use switch"*, `self`
(`+0x140`) = the record. The action menu's `human_use_actor` is the only caller of
`railway_switch_start_switching`, so the player throws the switch from the menu.

### 1b. `coll` — what it holds

`railway_switch_game_init` zeroes the `tDynamicCollObject` (`structs/collision.md`) and sets type
`0x42` (`+0x148`), frame (`+0x14c`), `+0x150` = 0, `+0x164` (record `+0x1c`) = 0.2 and owner (`+0x18c`) =
this; `railway_switch_set_coll_pos` writes the centre `+0x158..+0x160`. `collision_dyn_insert` /
`_update` / `_erase` carry the record through `game_init`, `update` and `game_done`.

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions (ECX on entry,
followed through moves and stack slots) and by reading all fifteen bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | blade_travel | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_ai`, `railway_switch_game_init`, `railway_switch_save_data` | - |
| +0x74 | blade_speed | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_ai`, `railway_switch_save_data` | - |
| +0x78 | sound_id | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_game_init`, `railway_switch_save_data` | passed to `sound_table_construct_sound` |
| +0x7c | web_node_common | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_get_web_nodes`, `railway_switch_save_data` | - |
| +0x80 | web_node_branch_a | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_get_web_nodes`, `railway_switch_save_data` | - |
| +0x84 | web_node_branch_b | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_get_web_nodes`, `railway_switch_save_data` | - |
| +0x88 | rail_a_name | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_game_init`, `railway_switch_save_data` | - |
| +0xa8 | rail_b_name | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_game_init`, `railway_switch_save_data` | - |
| +0xc8 | start_at_b | H | `railway_switch_ctor`, `railway_switch_duplicate_from`, `railway_switch_load_data` | `railway_switch_game_init`, `railway_switch_save_data` | - |
| +0xcc | rotation | H | `railway_switch_game_init`, `railway_switch_update` | `railway_switch_set_coll_pos` | - |
| +0x10c | use_record | H | `railway_switch_ctor` (its `+0x11c..+0x13c` zeroed), `railway_switch_game_init` | `using_object_find_near_objects` and the other `using_object_*` walkers | removed by `railway_switch_game_done` |
| +0x144 | blade_angle | H | `railway_switch_ai`, `railway_switch_game_init`, `railway_switch_save_game_load` | `railway_switch_ai`, `railway_switch_save_game_save` | - |
| +0x148 | coll | H | `railway_switch_ctor` (zero), `railway_switch_game_init`, `railway_switch_set_coll_pos` | `collision_dyn_insert`, `collision_dyn_update`, `collision_dyn_erase` | - |
| +0x190 | state | H | `railway_switch_ai`, `railway_switch_game_init`, `railway_switch_save_game_load`, `railway_switch_start_switching` | `railway_switch_ai`, `railway_switch_game_done`, `railway_switch_save_game_load`, `railway_switch_save_game_save`, `railway_switch_start_switching` | §2a |
| +0x194 | switch_sound | H | `railway_switch_game_init` | `railway_switch_game_done`, `railway_switch_game_init`, `railway_switch_start_switching` | - |
| +0x198 | rail_a | H | `railway_switch_game_init` | `railway_switch_ai`, `railway_switch_game_init`, `railway_switch_save_game_load` | - |
| +0x19c | rail_b | H | `railway_switch_game_init` | `railway_switch_ai`, `railway_switch_game_init`, `railway_switch_save_game_load` | - |

### 2a. `state` — the four values

| Value | Meaning | Evidence |
|---|---|---|
| 0 | rest 0: `blade_angle` = `+blade_travel`, `rail_a` shown, common ↔ branch A | `railway_switch_ai` arm of state 2, `railway_switch_game_init` |
| 1 | rest 1: `blade_angle` = `-blade_travel`, `rail_b` shown, common ↔ branch B | `railway_switch_ai` arm of state 3, `railway_switch_game_init` |
| 2 | moving to rest 0: the angle ramps **up** toward `+blade_travel` | `railway_switch_start_switching` from 1 |
| 3 | moving to rest 1: the angle ramps **down** toward `-blade_travel` | `railway_switch_start_switching` from 0 |

Anything else in `railway_switch_start_switching` is ignored, so a switch in motion cannot be
re-triggered.

## 3. Method family

17 of 17 PS2 methods mapped (15 CONFIRMED, `Init` STRUCTURAL on the shared default, `Hit` a base stub);
3 PC-only starts: the unit's static-init pair and the scalar deleting destructor. Slots 15, 27 and 30
keep the `C_actor` defaults.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00508120 | `crt_static_init_guard_00508120` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00508140 | `crt_atexit_register_00508140` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x00508150 | `railway_switch_ctor` | instance | `__ct__9C_turnoutFv` | 0x0022bf30 | CONFIRMED | `actor_ctor`, vtable, kind `0x18`, the defaults of §1; one caller, `mission_create_actor` case `0x18` |
| 0x00508260 | `railway_switch_scalar_deleting_dtor` | instance | - | - | - | slot 16; MSVC-only |
| 0x00508280 | `railway_switch_dtor` | instance | `__dt__9C_turnoutFv` | 0x0022c060 | CONFIRMED | restores the vtable, `actor_dtor` |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__9C_turnoutF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0022d1a0 | base-stub | slot 31; the `Hit` default of 13 vtables; the PS2 body returns 0 |
| 0x004ae890 | `actor_subclass_init_require_frame` | `Init__9C_turnoutFP9I3D_frame` | 0x0022c0c0 | STRUCTURAL | slot 18; the shared `Init` of 6 vtables: `if (!frame) return false; return C_actor::Init(frame)` |
| 0x00508290 | `railway_switch_save_data` | `SaveData__9C_turnoutFP7C_chunk` | 0x0022c0f0 | CONFIRMED | slot 19; a version byte and the designer fields, in the order of §1a |
| 0x005083f0 | `railway_switch_load_data` | `LoadData__9C_turnoutFP7C_chunk` | 0x0022c2b0 | CONFIRMED | slot 20; the same fields back |
| 0x00508620 | `railway_switch_duplicate_from` | `DuplicateFrom__9C_turnoutFP7C_actor` | 0x0022c770 | CONFIRMED | slot 21; copies the designer fields |
| 0x005086b0 | `railway_switch_game_init` | `GameInit__9C_turnoutFv` | 0x0022c800 | CONFIRMED | slot 25; resolves the rail pieces, enters the rest position of `start_at_b`, adds that connection, builds the collision record, the sound and the action-menu record |
| 0x00508b30 | `railway_switch_game_done` | `GameDone__9C_turnoutFv` | 0x0022cc20 | CONFIRMED | slot 26; erases the collision record, releases the sound, withdraws the record, removes the current connection |
| 0x00508c00 | `railway_switch_ai` | `AI__9C_turnoutFUi` | 0x0022cd20 | CONFIRMED | slot 13; ramps the blade, at an end stop swaps the rail pieces and the connection (§5), then rebuilds the wagons' paths |
| 0x00508dd0 | `railway_switch_update` | `Update__9C_turnoutFUi` | 0x0022cfe0 | CONFIRMED | slot 14; rebuilds the rotation, the collision centre and the cached position / forward row |
| 0x00509000 | `railway_switch_start_switching` | `Start__9C_turnoutFv` | 0x0022d0f0 | CONFIRMED | state `0 → 3` or `1 → 2`, starts the sound; sole caller `human_use_actor` |
| 0x00509050 | `railway_switch_set_coll_pos` | `SetCollPos__9C_turnoutFv` | 0x0022d1b0 | CONFIRMED | collision centre = position + 0.8 × the rotated local Y axis |
| 0x00509170 | `railway_switch_get_web_nodes` | `NodesOK__9C_turnoutFRUiRUiRUi` | 0x0022d290 | CONFIRMED | resolves the three ids through `web_path_find_node_id`; false if any is -1 or two are equal |
| 0x00509200 | `railway_switch_save_game_get_size` | `SaveGameGetSize__9C_turnoutFv` | 0x0022d360 | CONFIRMED | slot 22; `actor_save_game_get_size() + 0x10` |
| 0x00509210 | `railway_switch_save_game_save` | `SaveGameSave__9C_turnoutFPPv` | 0x0022d380 | CONFIRMED | slot 23; record type 2: `state` at `+4`, `blade_angle` at `+0xc` |
| 0x00509250 | `railway_switch_save_game_load` | `SaveGameLoad__9C_turnoutFPPv` | 0x0022d3e0 | CONFIRMED | slot 24; removes the current connection, restores state and angle, shows the rail piece and adds the connection of the loaded state, calls slot 14 |

### 3d. Free helpers

The unit has no free helper.

## 4. The shipped turnouts

Across the 79 shipped `scene2.bin` files exactly two objects are of kind `0x18`, both in
`MISE15-PRISTAV`: frames `vrch` and `vrch2`, with the rail pieces `vyhbk3` / `vyhbk4` and
`vyhbk5` / `vyhbk`, web-node ids 1/2/3 and 4/5/6, and sound id `0x85` in their scene data. That
scene holds no `C_railway` and two `C_wagon`s (`vagcisterna3`, `vagcisterna4`), and the scene's
script `aktivace vagonu 1` looks `vrch` up with `findactor`, so the turnouts route the harbour
wagons. No tram network in the shipped game has a turnout.

## 5. How it rewires the graph

The class calls two `C_web_path` slots through `*(g_act_mission + 0x2c)`: slot 5,
`web_path_toggle_connection`, which deletes the connection both ways if it exists and otherwise adds
it both ways, and slot 13, `web_path_set_connection_one_way` with flag 0, which sets the one-way bit
`0x80` on the first node's entry and clears it on the second's when it was clear.

| Function | When | Calls |
|---|---|---|
| `railway_switch_game_init` | scene start, `start_at_b` = 0 | toggle(common, A), one-way(A, common) |
| `railway_switch_game_init` | scene start, `start_at_b` = 1 | toggle(common, B), one-way(B, common) |
| `railway_switch_ai` | state 2 reaches `+blade_travel` | toggle(common, A), toggle(common, B), one-way(A, common), `wagon_rebuild_all_forward_paths` |
| `railway_switch_ai` | state 3 reaches `-blade_travel` | toggle(common, A), toggle(common, B), one-way(B, common), `wagon_rebuild_all_forward_paths` |
| `railway_switch_game_done` | scene end | toggle(common, A) in state 0 or 3, else toggle(common, B) |
| `railway_switch_save_game_load` | load | the `game_done` toggle, then the rest of the loaded state; no wagon rebuild |

The authored graph therefore holds neither edge: `game_init` adds one, each end stop swaps it for the
other, and `game_done` takes it away. `railway_switch_get_web_nodes` guards each call: a lookup of -1
or two equal ids skip the graph edit, so the switch needs one common node and two distinct branches.

## 6. The vtable

Slots 0–12 are the `C_actor_track` interface and hold the defaults; §3c gives every slot this class
overrides.

**Savegame:** record type `2`, 16 bytes: `state` at `+4`, `blade_angle` at `+0xc`.

## 7. False function starts in the unit

Ghidra's aggressive instruction finder created four false function starts inside this unit; none
is a function. Three are ordinary epilogue arms. The fourth, `0x0050904b`, starts inside the
4-byte displacement of the `call rel32` at `0x00509049`: Ghidra resynchronised on the `90 90`
padding and decompiled straight through into the real function, so its rendering is a
**verbatim copy of its host's body**, the fingerprint of this shape.

Counted as a function start, it truncates `railway_switch_save_game_load` to **29 B**. The true
extent is 432 B (the size check against the other build is in §8).

## 8. PC vs PS2

**Method accounting, 17 PS2 methods:** 15 have their own PC body and match statement for statement;
`Init` folds onto `actor_subclass_init_require_frame` (6 vtables) and `Hit` onto
`stub_retfalse_args32_0048a4f0` (13 vtables). The PS2 `Init` body is
`if (!frame) return false; return C_actor::Init(frame)`, precisely the shared PC function, and PS2
`Hit__9C_turnoutF…` is an 8-byte stub returning 0. The PC additionally emits a scalar deleting
destructor with no PS2 counterpart.

**Size check on the false start (§7).** Against the 640 B PS2 `SaveGameLoad`, the truncated 29 B
`railway_switch_save_game_load` gives a ratio of 0.045, which reads as a mismatch; the true 432 B
extent gives 0.68.

**Field delta is not uniform, so do not rebase band-wise.** The two builds agree field for field in
content and differ in offset by `0x100` for the first pair and `0x104` for the designer block:

| PC | PS2 | Field |
|---|---|---|
| `+0x70` | `+0x170` | `blade_travel` |
| `+0x74` | `+0x174` | `blade_speed` |
| `+0x78` | `+0x17c` | `sound_id` |
| `+0x7c` | `+0x180` | `web_node_common` |
| `+0x80` | `+0x184` | `web_node_branch_a` |
| `+0x84` | `+0x188` | `web_node_branch_b` |
| `+0x88` | `+0x18c` | `rail_a_name` |
| `+0xa8` | `+0x1ac` | `rail_b_name` |
| `+0xc8` | `+0x1cc` | `start_at_b` |
| `+0x144` | `+0x178` | `blade_angle` |
| `+0x190` | `+0xc0` | `state` |
| `+0x194` | `+0xc4` | `switch_sound` |

The four ctor-seeded ids are **byte-identical on both builds**, which is what makes the mapping a
measurement rather than an alignment.

**Cross-build corroboration:** where the PC `GameDone` calls `using_object_del_object`, the PS2
calls `DelObject__14C_using_object` at the same statement position. That corroborates that
`using_object_del_object` is a shared helper MSVC folded `DelObject` onto (the *position* belongs
to `C_using_object`, the *symbol* to no class).

**`RecompileNewPath__7C_wagonFv`** is the PS2's tail in both end-stop arms of `AI`, where the PC calls
`wagon_rebuild_all_forward_paths`. Neither build's `SaveGameLoad` rebuilds the paths.

## 9. Related

`structs/actor.md` (the base) · `structs/web_path.md` (the graph it edits) · `structs/wagon.md`
(the vehicles it routes) · `structs/using_object.md` (the action-menu record) ·
`structs/collision.md` (the dyn-coll record) · `structs/entity_kind.md` §1 (the kind map).

## 10. Open items

1. **In game.** In `MISE15-PRISTAV`, throw a switch from the action menu and watch a wagon take the
   other branch, which confirms §5 end to end.
