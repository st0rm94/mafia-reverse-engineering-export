# `C_public_physics` — the `create_physicalobject` prop

**Created** 2026-08-15 10:06 +0200 · **Modified** 2026-10-30 12:24 +0100

The general-purpose `C_PhThing` subclass: the one that turns an **existing scene frame** into a
free-falling, rolling, colliding physics object. Its four siblings are each specialised to one kind
of debris (`C_DropOut` a car part, `C_crash_obj` a breakable prop, `C_drop_in_weapon` a dropped gun,
`C_box_new` a crate); this one may be built over any frame.

| | |
|---|---|
| PC prefix | `public_physics_` ¹ |
| PC kind (`+0x10`) | `0x20` (`structs/entity_kind.md` §1) |
| PC vtable | `g_public_physics_vt` @ `0x0063c9b8` (39 slots) |
| PC base class | `C_PhThing` (`structs/physical_object.md`), derived from `C_actor` |
| PC instance size | `0x3b0`, the operand of the `operator_new` before each of the five construction sites |
| PC constructor | `public_physics_ctor` @ `0x004cde20` |
| PC translation unit | `[0x004cdd70, 0x004cdfd0)`, guard to guard: 17 starts, all named |
| PS2 class | `C_public_physics` (tag `16C_public_physics`), 14 methods |

¹ The designers' noun is *physical object* (the script command is `create_physicalobject`), but
`physical_object_` is the prefix of the base class `C_PhThing`, and any extension of it would mix
this class's members into the base's prefix. `public_physics_` keeps the developers' class name.

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class adds no field: `operator_new(0x3b0)` is the size of `C_PhThing` itself
(`structs/physical_object.md` §1), so the typedef holds the base alone. The base fields its own
functions touch are the savegame exclusion byte `+0x65` and the frame `+0x68`.

```c
typedef struct C_public_physics {             // 0x3b0 bytes
    /* +0x00 */ C_PhThing base;               // [H] the whole object; no own field
} C_public_physics;
```

## 2. Field evidence

Measured by reading all eight own bodies; none accesses an offset at or past `+0x3b0`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `physical_object_ctor` | every `physical_object_*` function | `public_physics_game_init` raises `+0x65` to 1 and adds a reference at `frame + 4`; `public_physics_game_done` releases it through the frame's slot 0 |

## 3. Method family

14 of 14 PS2 methods mapped (6 CONFIRMED, 4 STRUCTURAL, 3 base stubs on PC slots, and `ChangeState`
in the PS2-only table of §8); 7 PC-only starts: the unit's two static-init pairs, the list pair and
the scalar deleting destructor. Slot 27 keeps the base `physical_object_on_act_state_changed`.

The four thunks and the two stubs are shared bodies: a scan of `.rdata` / `.data` for the address as
a pointer finds them in 2 to 37 vtables, so none carries a twin line in its docblock and the mapping
lives here. The four CONFIRMED overrides and the scalar deleting destructor have exactly one such
reference.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004cdd70 | `crt_static_init_guard_004cdd70` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer; bit 0 of `g_crt_static_init_flags`, then the registration below |
| 0x004cdd90 | `crt_atexit_register_004cdd90` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x004cdda0 | `crt_static_init_004cdda0` | `g_public_physics_list` | - | - | - | `.CRT$XCU` entry of the vector: calls the next two |
| 0x004cddb0 | `public_physics_list_init` | `g_public_physics_list` | - | - | - | static constructor: empties the vector |
| 0x004cddd0 | `crt_atexit_register_004cddd0` | `g_public_physics_list` | - | - | - | registers `public_physics_list_free` |
| 0x004cdde0 | `public_physics_list_free` | `g_public_physics_list` | - | - | - | static destructor, guarded by bit 0 of `g_public_physics_list_freed`: frees the buffer and clears the three pointers |
| 0x004cde20 | `public_physics_ctor` | instance | `__ct__16C_public_physicsFv` | 0x001d4a70 | CONFIRMED | `physical_object_ctor`, vtable, kind `0x20`; five callsites (§5) |
| 0x004cde40 | `public_physics_scalar_deleting_dtor` | instance | - | - | - | slot 16; MSVC emits it, the PS2 fuses it into `__dt__` |
| 0x004cde60 | `public_physics_dtor` | instance | `__dt__16C_public_physicsFv` | 0x001d4ab0 | CONFIRMED | restores the vtable, `physical_object_dtor` |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__16C_public_physicsFP7C_actor` | 0x001d4b50 | base-stub | slot 21; the default shared by 9 vtables |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__16C_public_physicsFP7C_chunk` | 0x001d4c80 | base-stub | slots 19 and 20 (`LoadData` is the next row); the default shared by 37 vtables |
| 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__16C_public_physicsFP7C_chunk` | 0x001d4c90 | base-stub | slot 20, the same stub |
| 0x004cde70 | `thunk_physical_object_ai_tick` | `AI__16C_public_physicsFUi` | 0x001d4b10 | STRUCTURAL | slot 13; a jump to `physical_object_ai_tick`, referenced by 2 vtables |
| 0x004cde80 | `thunk_physical_object_update` | `Update__16C_public_physicsFUi` | 0x001d4b20 | STRUCTURAL | slot 14; a jump to `physical_object_update`, referenced by 2 vtables |
| 0x004cde90 | `thunk_physical_object_net_apply_remote_state` | `NetDirect__16C_public_physicsFUi` | 0x001d4b30 | STRUCTURAL | slot 15; referenced by 3 vtables |
| 0x004cdea0 | `thunk_physical_object_init` | `Init__16C_public_physicsFP9I3D_frame` | 0x001d4b40 | STRUCTURAL | slot 18; a jump to `physical_object_init`, referenced by 2 vtables |
| 0x004cdeb0 | `public_physics_game_init` | `GameInit__16C_public_physicsFv` | 0x001d4be0 | CONFIRMED | slot 25; `physical_object_game_init`, `+0x65` = 1, one more reference on the frame; while `C_game` `running` is set, the list of §7 |
| 0x004cdf90 | `public_physics_game_done` | `GameDone__16C_public_physicsFv` | 0x001d4c20 | CONFIRMED | slot 26; `physical_object_game_done`, then releases the frame reference |
| 0x004cdfb0 | `public_physics_on_collision` | `Collision__16C_public_physicsFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001d4c60 | CONFIRMED | slot 30; forwards to `physical_object_on_collision` |
| 0x004cdfc0 | `public_physics_hit` | `Hit__16C_public_physicsF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001d4c70 | CONFIRMED | slot 31; forwards to `physical_object_hit` |

### 3d. Free helpers

The unit has no free helper.

## 4. What it is — settled by the designer documentation

The identification comes from the Mafia Script Data Reference, whose entry for
`create_physicalobject ww, xx, yy, zz` reads (translated):

> *"Turns object ww into a physical one. These objects interact badly with cars. … **At most 30
> physical children can be created from one object.** The object can be removed with
> destroy_physicalobject."*
> `ww` = frame ID · `xx` = initial motion · `yy` = 0/1 · `zz` = weight in kg

Two of those facts are checkable in the PC code and both check out:

| The reference says | The code does |
|---|---|
| turn **object ww** (a frame id) into a physical one | every creator takes an existing `I3D_frame`, wraps it, `Init(frame)`, `physical_object_init_box`, `game_add_temporary_actor` |
| at most **30** may exist | `public_physics_game_init`'s gate is `0x1e < size`, after which it retires the oldest |

The 30 identifies the class: it has no other reason to appear in a function whose only other job is
a base-class chain. Note the difference in scope: the code's cap is global to the list (§7), not per
source object as the reference words it.

The reference's warning that these objects *"interact badly with cars"* is a designer-facing caveat;
nothing in the code is traced to it (Open item 1).

## 5. The five creation sites

Every caller of `public_physics_ctor`:

| Creator | What it turns into a prop |
|---|---|
| `human_free_hand` | whatever an NPC was holding, when he lets go |
| `pedestrian_drop_umbrella` | an ambient pedestrian's umbrella (`structs/pedestrian.md`) |
| `human_start_weapon_effect` | a weapon-effect object that outlives its effect |
| `program_process_command` case `0xcc` | `create_physicalobject`, the script path |
| `mission_create_actor` case `0x20` | the generic entity factory |

The first four share one shape; the factory builds the actor and leaves `Init` to its caller:

```
operator_new(0x3b0)
public_physics_ctor()
if (!vtable[18] /* Init */ (frame))  vtable[17] /* Release */ ();
else { physical_object_init_box(...); game_add_temporary_actor(this); }
```

## 6. The script commands

Both case bodies are read and cross-checked against the Mafia Script Data Reference, and annotated in
the decomp with `//` banners above the case labels.

| Case | Command | What it does |
|---|---|---|
| `0xcc` | `CREATE_PHYSICALOBJECT ww, xx, yy, zz` | `ww` the frame the prop wraps (the actor's `+0x68`) · `xx` the initial velocity, stored as `(0, -xx, 0)` at `+0x1e0`, so **>0 = down**; with `xx` = 0 the prop starts asleep (act state 1) until something hits it · `yy` non-zero: an impulse `(0, 0.01 × mass, 0)` at a random world point in `[-1, 1)³`, which wakes the prop · `zz` the mass in kg, at least 1.0, at `+0x190`; it also sets the inverse inertia `+0x194..+0x19c` and the size of the `yy` impulse |
| `0xcd` | `DESTROY_PHYSICALOBJECT xx` | scans the mission actor list for kind **`0x20`** with frame `xx` and retires it via `game_remove_temporary_actor` |

The keyword table stores `CREATE_PHYSICALOBJECT = 0xcc` and `DESTROY_PHYSICALOBJECT = 0xcd`, and the
stored opcode is the case label (`basic/artifacts/script-opcode-dispatch.md`).

The create arm hands `physical_object_init_box` an `S_THING_BOX` built on its stack
(`structs/physical_object.md` §4):

| Block offset | Value from the arm | Stored by `physical_object_init_box` at |
|---|---|---|
| `+0x00` | the mass, `max(zz, 1.0)` | `+0x190` |
| `+0x04` .. `+0x0c` | the initial velocity `(0, -xx, 0)` | `+0x1e0` .. `+0x1e8` |
| `+0x10` | the friction, `0.45` | `+0x1f8` |

`physical_object_init_box` then sets act state 1 when the velocity's squared length is below
`1e-5`, and derives the box centre (`+0x78` .. `+0x80`) and the inverse inertia from the frame's
bounding box and the mass.

## 7. The file-scope list

`C_public_physics.cpp` owns one file-scope object, an MSVC `std::vector`:

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_public_physics_list_freed` | `0x006560ec` | once-guard of the `atexit` teardown (bit `0x1`) | `public_physics_list_free` | `public_physics_list_free` |
| `g_public_physics_list` | `0x006560f0` | the vector's allocator byte | `public_physics_list_init` | - |
| `g_public_physics_list_begin` | `0x006560f4` | begin | `public_physics_list_init`, `public_physics_list_free`, `game_init`, `std_vector4_insert_n` | `public_physics_game_init`, `public_physics_list_free` |
| `g_public_physics_list_end` | `0x006560f8` | end; `(end-begin)>>2` is the live count | `public_physics_list_init`, `public_physics_list_free`, `public_physics_game_init`, `game_init`, `std_vector4_insert_n` | `public_physics_game_init` |
| `g_public_physics_list_cap` | `0x006560fc` | capacity | `public_physics_list_init`, `public_physics_list_free`, `game_init`, `std_vector4_insert_n` | `std_vector4_insert_n` |

`public_physics_game_init`, gated on `C_game` `running` (`+0x40`, `structs/game.md` §1), inserts
`this` if absent; once `size > 30` it calls `game_invalidate_actor(list[0])` +
`game_remove_temporary_actor(list[0])` and shifts the vector down: **a global FIFO of at most 30
props, oldest retired first.** That gate also means a prop whose `GameInit` runs before the scene is
running is not listed.

The element type is `C_public_physics *`, established by a census of the consumers: outside the
unit's own init / free pair, exactly two functions touch the vector, `public_physics_game_init` (the
only producer) and `game_init` (clears it wholesale at scene start).

**Nothing removes an entry.** A prop retired any other way (`destroy_physicalobject`, or
`game_remove_temporary_actor` from any caller) is released by `game_remove_temporary_actor_now`
while its pointer stays in the list. The stale pointer is used again once it reaches `list[0]` with
more than 30 entries: `public_physics_game_init` then hands it to `game_invalidate_actor` and
`game_remove_temporary_actor`, on freed memory. Four shipped missions use `destroy_physicalobject`
(`MISE15-MESTO`, `MISE15-PRISTAV`, `MISE15-SALIERY`, `MISE17-VEZENI`); the highest
`create_physicalobject` count in one mission's scripts is 10 (`MISE15-PRISTAV`), so reaching the case
needs many runtime props (umbrellas, dropped hand items) in the same scene after a removal.

## 8. PC vs PS2

**`ChangeState`.** The PS2 overrides `ChangeState` (128 B) and the PC does **not**: slot 27 holds the
base `physical_object_on_act_state_changed`, which 4 vtables share. The PC's retirement is located
elsewhere:

| | PS2 | PC |
|---|---|---|
| Where | `ChangeState__16C_public_physicsFv` | `public_physics_game_init` |
| When | per object, on a state change, gated on `+0x5c != 0 && +0x640 != 0` | on init, gated on `C_game` `running` |
| What | `InternalPlacementReset` → `InvalidateActor` → `RemoveTemporaryActor` → `AddDeathHumanFade` | insert into `g_public_physics_list`; once `size > 30`, retire `list[0]` (§7) |

The PS2 retires a prop when it changes state and fades it out; the PC keeps a global FIFO of at most
30 and retires the oldest when a 31st is created. Both spend the same two calls (`InvalidateActor` +
`RemoveTemporaryActor`); only the trigger differs. The PS2 `GameInit` has no list: it stores `+0x69`
= 1 and adds the frame reference (a short at frame `+0xe4`).

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001d4b60 | `ChangeState__16C_public_physicsFv` | slot 27 keeps the base function; the PC retires props in `public_physics_game_init` instead |

**Scalar deleting destructor.** MSVC-only; GCC folds it into `__dt__`'s delete arm, so it can never
have a PS2 symbol.

**Emission order.** PS2 `C_public_physics.cpp`: ctor, dtor, AI, Update, NetDirect, Init,
DuplicateFrom, ChangeState, GameInit, GameDone, Collision, Hit, SaveData, LoadData
(`0x001d4a70`..`0x001d4c98`). The PC order is the same for the methods it has own bodies for, with
the scalar deleting destructor inserted, which makes the unit a forced alignment.

## 9. Related

`structs/physical_object.md` (the `C_PhThing` base and its five subclasses) · `structs/entity_kind.md`
§1 (the kind map; `0x20` is this class) · `script-mapping/artifacts/opcodes-verified.md` (cases `0xcc`
/ `0xcd`).

## 10. Open items

1. **The "interact badly with cars" caveat.** Nothing in the code is traced to it; one in-game
   observation (drive into a `create_physicalobject` prop) would say what it looks like.
