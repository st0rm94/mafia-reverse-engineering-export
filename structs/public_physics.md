# `C_public_physics` — the `create_physicalobject` prop

**Created** 2026-08-15 10:06 +0200 · **Modified** 2026-10-05 21:51 +0200

The general-purpose `C_PhThing` subclass: the one that turns an **existing scene frame** into a
free-falling, rolling, colliding physics object. Its four siblings are each specialised to one kind
of debris (`C_DropOut` a car part, `C_crash_obj` a breakable prop, `C_drop_in_weapon` a dropped gun,
`C_box_new` a crate); this is the one anything may instantiate over any frame, which is what
"public" means in the developers' name.

| | |
|---|---|
| PC prefix | **`public_physics_`** ¹ |
| Base class | `C_PhThing` (`structs/physical_object.md`); `public_physics_ctor` → `physical_object_ctor` → `actor_ctor` |
| kind | `0x20` (`structs/entity.md` §1) |
| PC vtable | `.rdata 0x0063c9b8`, 39 slots |
| Instance size | `0x3b0`, byte-fixed by the `operator_new(0x3b0)` before every one of the five construction sites |
| PC translation unit | `[0x004cdda0, 0x004cdfd0)` — 11 own function starts plus 4 shared thunks, bounded at both ends by a CRT static-init/atexit pair; follows `C_player.cpp`, precedes `C_pumper.cpp` |

¹ The designers' own noun is *physical object* (the script command is `create_physicalobject`), but
`physical_object_` is the **base** class `C_PhThing`'s prefix, and any extension of it
(`physical_object_prop_`, …) would mix this class's members into the base's prefix.
`public_physics_` keeps the developers' class name and collides with nothing.

---

## 1. What it is — settled by the shipped designer documentation

The identification comes from the Mafia Script Data Reference, whose entry for
`create_physicalobject ww, xx, yy, zz` reads (translated):

> *"Turns object ww into a physical one. These objects interact badly with cars. … **At most 30
> physical children can be created from one object.** The object can be removed with
> destroy_physicalobject."*
> `ww` = frame ID · `xx` = initial motion · `yy` = 0/1 · `zz` = weight in kg

Two of those facts are checkable in the PC code and both check out:

| the reference says | the code does |
|---|---|
| turn **object ww** (a frame id) into a physical one | every creator takes an existing `I3D_frame`, wraps it, `Init(frame)`, `physical_object_init_box`, `game_add_temporary_actor` |
| at most **30** may exist | `public_physics_game_init`'s gate is literally `0x1e < size` — 30 — after which it retires the oldest |

**The 30 is the match that identifies the class.** It is a number with no other reason to appear,
sitting in a function whose only other job is a base-class chain.

The reference's warning that these objects *"interact badly with cars"* is a designer-facing caveat
about the physics; it is the only statement anywhere about the class's quality, and it is not derived
from the code.

## 2. The five creation sites

Every caller of `public_physics_ctor`:

| creator | what it turns into a prop |
|---|---|
| `human_free_hand` | whatever an NPC was holding, when he lets go |
| `pedestrian_drop_umbrella` | an ambient pedestrian's umbrella (`structs/pedestrian.md`) |
| `human_start_weapon_effect` | a weapon-effect object that outlives its effect |
| `program_process_command` case `0xcc` | `create_physicalobject` — the script path |
| `mission_create_actor` (`0x005fed70`) case `0x20` | the generic entity factory |

All five share one shape:

```
operator_new(0x3b0)
public_physics_ctor()
if (!vtable[18] /* Init */ (frame))  vtable[17] /* Release */ ();
else { physical_object_init_box(...); game_add_temporary_actor(this); }
```

⇒ The class is never placed by a scene file; it is always constructed at runtime over a frame that
already exists, so it appears in no `scene2.bin` special-data census.

## 3. The script commands

Both case bodies are read and cross-checked against the Mafia Script Data Reference, and annotated in
the decomp with `//` banners above the case labels.

| case | command | what it does |
|---|---|---|
| `0xcc` | `CREATE_PHYSICALOBJECT ww, xx, yy, zz` | `ww` the frame the prop wraps (the actor's `+0x68`) · `xx` the initial velocity, stored as `(0, -xx, 0)` at `+0x1e0`, so **>0 = down**; with `xx` = 0 the prop starts asleep (act state 1) until something hits it · `yy` non-zero: an impulse `(0, 0.01 × mass, 0)` at a random world point in `[-1, 1)³`, which wakes the prop · `zz` the mass in kg, at least 1.0, at `+0x190`; it also sets the inverse inertia `+0x194..+0x19c` and the size of the `yy` impulse |
| `0xcd` | `DESTROY_PHYSICALOBJECT xx` | scans the mission actor list for kind **`0x20`** with frame `xx` and retires it via `game_remove_temporary_actor` |

The keyword table stores `CREATE_PHYSICALOBJECT = 0xcc` and `DESTROY_PHYSICALOBJECT = 0xcd`, and the
stored opcode is the case label (`basic/artifacts/script-opcode-dispatch.md`).

The create arm hands `physical_object_init_box` an `S_THING_BOX` built on its stack
(`structs/physical_object.md` §1):

| block offset | value from the arm | stored by `physical_object_init_box` at |
|---|---|---|
| `+0x00` | the mass, `max(zz, 1.0)` | `+0x190` |
| `+0x04` .. `+0x0c` | the initial velocity `(0, -xx, 0)` | `+0x1e0` .. `+0x1e8` |
| `+0x10` | the friction, `0.45` | `+0x1f8` |

`physical_object_init_box` then sets act state 1 when the velocity's squared length is below
`1e-5`, and derives the box centre (`+0x78` .. `+0x80`) and the inverse inertia from the frame's
bounding box and the mass.

## 4. Method coverage — 14/14 PS2 methods accounted, 11/11 PC starts named

| PC function | VA | vtable slot | PS2 method | grade |
|---|---|---|---|---|
| `public_physics_ctor` | 0x004cde20 | - | `__ct__16C_public_physicsFv` | CONFIRMED |
| `public_physics_dtor` | 0x004cde60 | - | `__dt__16C_public_physicsFv` | CONFIRMED |
| `public_physics_scalar_deleting_dtor` | 0x004cde40 | 16 | - | STRUCTURAL ¹ |
| `public_physics_game_init` | 0x004cdeb0 | 25 | `GameInit__16C_public_physicsFv` | CONFIRMED |
| `public_physics_game_done` | 0x004cdf90 | 26 | `GameDone__16C_public_physicsFv` | CONFIRMED |
| `public_physics_on_collision` | 0x004cdfb0 | 30 | `Collision__16C_public_physicsF…` | CONFIRMED |
| `public_physics_hit` | 0x004cdfc0 | 31 | `Hit__16C_public_physicsF…` | CONFIRMED |
| `thunk_physical_object_ai_tick` | 0x004cde70 | 13 | `AI__16C_public_physicsFUi` | STRUCTURAL ² |
| `thunk_physical_object_update` | 0x004cde80 | 14 | `Update__16C_public_physicsFUi` | STRUCTURAL ² |
| `thunk_physical_object_net_apply_remote_state` | 0x004cde90 | 15 | `NetDirect__16C_public_physicsFUi` | STRUCTURAL ² |
| `thunk_physical_object_init` | 0x004cdea0 | 18 | `Init__16C_public_physicsFP9I3D_frame` | STRUCTURAL ² |
| `stub_rettrue_args4_00477710` | 0x00477710 | 19, 20 | `SaveData` / `LoadData` | STRUCTURAL ² (37 vtables) |
| `thunk_stub_noop_args4_00461180` | 0x004773f0 | 21 | `DuplicateFrom__16C_public_physicsFP7C_actor` | STRUCTURAL ² (9 vtables) |
| `physical_object_on_act_state_changed` (base) | 0x004bfa80 | 27 | `ChangeState__16C_public_physicsFv` | not overridden on PC ³ |

¹ MSVC-only. ² Shared body. Each of these symbols is referenced by 2–37 vtables (a scan of
`.rdata`/`.data` for the address as a pointer), so none of them carries a `PS2 twin:` line in its
docblock — that would assert a single-class ownership the pointer scan refutes; the mapping lives in
this table only. The six CONFIRMED overrides each have exactly **1** such reference. ³ §6.

Plus the TU's four non-method starts: `crt_static_init_004cdda0`, `public_physics_list_init`,
`crt_atexit_register_004cddd0`, `public_physics_list_free`.

## 5. The file-scope registry

`C_public_physics.cpp` owns one file-scope object, an MSVC `std::vector`:

| VA | name | role |
|---|---|---|
| `0x006560ec` | `g_public_physics_list_freed` | once-guard for the `atexit` teardown (bit `0x1`) |
| `0x006560f0` | `g_public_physics_list` | the vector object (allocator byte) |
| `0x006560f4` | `g_public_physics_list_begin` | begin |
| `0x006560f8` | `g_public_physics_list_end` | end — `(end-begin)>>2` is the live count |
| `0x006560fc` | `g_public_physics_list_cap` | capacity |

`public_physics_game_init` (gated on `C_game+0x40`) inserts `this` if absent; once `size > 30` it
calls `game_invalidate_actor(list[0])` + `game_remove_temporary_actor(list[0])` and shifts the vector
down — **a global FIFO of at most 30 props, oldest retired first.** That is the rule the designer
reference documents.

The element type is `C_public_physics*`, established by a census of the consumers: outside the TU's
own init/free pair, exactly **two** functions touch the vector — `public_physics_game_init` (the only
producer) and `game_init` (clears it wholesale at scene start). Nothing else is ever put in.

**Nothing removes an entry.** Only four functions touch the vector: its static initialiser and
`public_physics_list_free`, `public_physics_game_init` and `game_init`. A prop retired any other
way — `destroy_physicalobject`, or `game_remove_temporary_actor` from any caller — is released by
`game_remove_temporary_actor_now` while its pointer stays in the list. The stale pointer is used
again once it reaches `list[0]` with more than 30 entries: `public_physics_game_init` then hands
it to `game_invalidate_actor` and `game_remove_temporary_actor`, on freed memory. Four shipped
missions use `destroy_physicalobject` (`MISE15-MESTO`, `MISE15-PRISTAV`, `MISE15-SALIERY`,
`MISE17-VEZENI`); no mission's scripts contain more than eight `create_physicalobject` calls, so
reaching the case needs many runtime props (umbrellas, dropped hand items) in the same scene after
a removal.

## 6. PC vs PS2

**`ChangeState`.** PS2 overrides `ChangeState` (128 B) and the PC does **not** — vtable slot 27 holds
the base `physical_object_on_act_state_changed`. The PC's version of the same work is not inlined; it
is located elsewhere:

| | PS2 | PC |
|---|---|---|
| where | `ChangeState__16C_public_physics` | `public_physics_game_init` |
| when | per object, on a state change, gated on `+0x5c != 0 && +0x640 != 0` | on init, gated on `C_game+0x40` |
| what | `InternalPlacementReset` → `InvalidateActor` → `RemoveTemporaryActor` → `AddDeathHumanFade` | insert into `g_public_physics_list`; once `size > 30`, retire `list[0]` (§5) |

⇒ **PS2 retires a prop when it changes state and fades it out; the PC keeps a global FIFO of at most
30 and retires the oldest when a 31st is created.** Both spend the same two calls (`InvalidateActor`
+ `RemoveTemporaryActor`) — only the trigger differs.

**Scalar deleting destructor.** MSVC-only; GCC folds it into `__dt__`'s delete arm, so it can never
have a PS2 symbol.

**Emission order.** PS2 `C_public_physics.cpp`: ctor, dtor, AI, Update, NetDirect, Init,
DuplicateFrom, ChangeState, GameInit, GameDone, Collision, Hit, SaveData, LoadData —
`0x001d4a70`..`0x001d4c98`. The PC order is the same modulo the missing `ChangeState` and the inserted
scalar deleting destructor, which makes the TU a forced alignment.

`__sinit_C_public_physics.cpp` is **not** in the PS2 source-file manifest under that name; the class
sits between `C_program`'s helpers and `C_pumper` in the alphabetical run of class names, which is
what places it in the link order.

## 7. Related

`structs/physical_object.md` (the `C_PhThing` base and its five subclasses) · `structs/entity.md` §1
(the kind map; `0x20` is this class) · `script-mapping/artifacts/opcodes-verified.md` (cases `0xcc` /
`0xcd`).

## 8. Open items

| Item | State |
|---|---|
| the "interact badly with cars" caveat | Nothing in the code is traced to it; one in-game observation (drive into a `create_physicalobject` prop) would say what it looks like |
