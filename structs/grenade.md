# `C_grenade` — the thrown grenade AND the Molotov (kind `0xe`)

**Created** 2026-08-13 20:11 +0200 · **Modified** 2026-09-17 13:45 +0200

PC prefix **`grenade_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0xe`** |
| PC vtable | `.rdata 0x0063c3c0` (39 slots) |
| PC constructor | `grenade_ctor` @ `0x0048af30` |
| PC instance size | **`0x160`** — `operator_new(0x160)` at its single creation site |
| PC translation unit | `[0x0048af30, 0x0048b5b0)` — 14 function starts, all named |
| Base class | **`C_bottle`** (`structs/bottle.md`), which derives from `C_actor` |
| PS2 class | `C_grenade`, 15 methods, 15/15 accounted |

---

## 1. What it is — and why "grenade" understates it

A thrown explosive projectile. `C_bottle` supplies the physics — the arc, the bounce, the settle —
and this class adds exactly three things on top: an **impact sound keyed to the surface it hits**, a
**fuse** with two independent trigger modes, and a **payload** that fires when the fuse does.

**The payload is where the class turns out to be two weapons.** `grenade_ai_tick` has two terminal
arms and they are independent flags, not an enum:

| flag | payload | parameters |
|---|---|---|
| `+0x141` | `game_new_explosion` | radius `+0x144`, damage `+0x148` |
| `+0x14c` | `game_new_small_fires` + particle 8 + `game_new_fire` | the triple `+0x150` / `+0x154` / `+0x158` |

That matches the creator exactly. `game_new_grenade` (= `NewGrenade__6C_game`, CONFIRMED) loads
**`2grenade.i3d` for types 0/1 and `2mol.i3d` for types 2/3** — one class, one code path, and the
designer flags in the savegame record decide whether the thing that lands is a fragmentation grenade
or a Molotov cocktail.

The prefix records the developers' class name, read off the mangled tag (the prefix map is
per-class). It is **not** a claim that every instance is a grenade.

### 1a. How the class is identified — four independent levers

1. **The vtable order in `.rdata`.** Between `C_DropOut` (`0x0063c320`) and `C_human` (`0x0063c468`)
   there is exactly one vtable: `0x0063c3c0`, kind `0xe`. §6 explains why `C_ff_manager`, which sits
   between them in the PS2 file order, cannot appear in a vtable run at all.
2. **The kind byte** — an independent check against `structs/entity.md` §1.
3. **The creator carries a CONFIRMED twin.** `game_new_grenade` is the ctor's only caller, so the
   class name is read off a confirmed twin, not inferred from link order.
4. **The instance size closes the field map.** `operator_new(0x160)`, and the class's own band ends
   at `+0x15c + 4 = 0x160` exactly.

---

## 2. Field layout

Only the class's OWN band is listed; everything below `+0x138` belongs to `C_bottle` or `C_actor`
(`structs/bottle.md`, `structs/actor.md`). Confidence: [H] byte-proven, [M] measured with one
reading, [U] unknown.

```c
typedef struct C_grenade {          /* size 0x160, base C_bottle */
    /* ... C_actor + C_bottle, 0x00 .. 0x137 ... */
    /* +0x138 */ char  fuse_timed_armed;    // [H] GameInit clears; save byte 1
    /* +0x13c */ int   fuse_ms_remaining;   // [H] AI subtracts the frame delta; fires at < 0
    /* +0x140 */ char  fuse_on_impact;      // [H] arms the +0x100 impact counter instead
    /* +0x141 */ char  do_explosion;        // [H] selects the game_new_explosion arm
    /* +0x144 */ float explosion_radius;    // [H] arg 4 of game_new_explosion
    /* +0x148 */ float explosion_damage;    // [H] arg 5 of game_new_explosion
    /* +0x14c */ char  do_fire;             // [H] selects the game_new_fire arm
    /* +0x150 */ int   fire_kind;           // [M] arg 4 of game_new_fire
    /* +0x154 */ float fire_param_a;        // [M] arg 5 of game_new_fire
    /* +0x158 */ float fire_param_b;        // [M] arg 6 of game_new_fire
    /* +0x15c */ char  net_direct_enabled;  // [H] ctor sets 1; the NetDirect gate
} C_grenade;                        /* 0x160 */
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x138 | `fuse_timed_armed` | H | `grenade_game_init`, `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | byte 1 of the save record |
| +0x13c | `fuse_ms_remaining` | H | `grenade_ai_tick`, `grenade_save_game_load` | `grenade_ai_tick` | counts down by the frame delta |
| +0x140 | `fuse_on_impact` | H | `grenade_game_init`, `grenade_save_game_load` | `grenade_ai_tick`, `grenade_on_collision`, `grenade_hit` | gates the `+0x100` counter |
| +0x141 | `do_explosion` | H | `grenade_game_init`, `grenade_save_game_load` | `grenade_ai_tick` | - |
| +0x144 | `explosion_radius` | H | `grenade_save_game_load` | `grenade_ai_tick` | designer-set; no other PC writer located |
| +0x148 | `explosion_damage` | H | `grenade_save_game_load` | `grenade_ai_tick` | designer-set; no other PC writer located |
| +0x14c | `do_fire` | H | `grenade_game_init`, `grenade_save_game_load` | `grenade_ai_tick` | - |
| +0x150 | `fire_kind` | M | `grenade_save_game_load` | `grenade_ai_tick` | forwarded to `game_new_fire` un-inspected |
| +0x154 | `fire_param_a` | M | `grenade_save_game_load` | `grenade_ai_tick` | forwarded to `game_new_fire` un-inspected |
| +0x158 | `fire_param_b` | M | `grenade_save_game_load` | `grenade_ai_tick` | forwarded to `game_new_fire` un-inspected |
| +0x15c | `net_direct_enabled` | H | `grenade_ctor`, `grenade_apply_net_event_msg` | `grenade_net_apply_remote_state` | the ctor sets it; the type-`0x16` message clears/sets it |

**Base fields this class drives** (in `C_bottle`/`C_actor` territory, listed because the class's
behaviour is unreadable without them):

| offset | role here |
|---|---|
| `+0x1c` | act-state — **2 = spent**, tested by both `grenade_on_collision` and `grenade_hit` |
| `+0x80` | the float compared against `50.0` to decide the projectile retires instead of firing |
| `+0x100` | the impact counter the `+0x140` fuse mode reads |
| `+0x104` | the impact-sound enable |
| `+0x12c` | the surface-material id indexing `g_surface_material_table` |
| `+0x108`..`+0x110` | the pose mirror the net message snaps from |

---

## 3. Method coverage — 15/15 PS2 methods accounted; 14/14 PC starts named

Ordered by PC VA.

| PC VA | PC name | slot | PS2 twin | PS2 size | Status |
|---|---|---|---|---|---|
| `0x0048af30` | `grenade_ctor` | - | `__ct__9C_grenadeFv` | 72 | CONFIRMED |
| `0x0048af60` | `grenade_scalar_deleting_dtor` | 16 | - | - | PC-only ¹ |
| `0x0048af80` | `grenade_dtor` | - | `__dt__9C_grenadeFv` | 96 | CONFIRMED |
| `0x0048af90` | `bottle_subclass_init_require_frame` | 18 | `Init__9C_grenadeFP9I3D_frame` | 44 | ICF fold ² |
| `0x0048afb0` | `grenade_ai_tick` | 13 | `AI__9C_grenadeFUi` | 1756 | CONFIRMED |
| `0x0048b310` | `grenade_game_init` | 25 | `GameInit__9C_grenadeFv` | 56 | CONFIRMED |
| `0x0048b340` | `grenade_game_done` | 26 | `GameDone__9C_grenadeFv` | 80 | CONFIRMED |
| `0x0048b360` | `grenade_save_game_get_size` | 22 | `SaveGameGetSize__9C_grenadeFv` | 32 | CONFIRMED |
| `0x0048b370` | `grenade_save_game_save` | 23 | `SaveGameSave__9C_grenadeFPPv` | 168 | CONFIRMED |
| `0x0048b400` | `grenade_save_game_load` | 24 | `SaveGameLoad__9C_grenadeFPPv` | 184 | CONFIRMED |
| `0x0048b490` | `grenade_on_collision` | 30 | `Collision__9C_grenadeF…` | 76 | CONFIRMED |
| `0x0048b4c0` | `grenade_hit` | 31 | `Hit__9C_grenadeF…` | 72 | CONFIRMED |
| `0x0048b510` | `grenade_apply_net_event_msg` | 38 | - | - | PC-only ³ |
| `0x0048b590` | `grenade_net_apply_remote_state` | 15 | `NetDirect__9C_grenadeFUi` | 8 | STRUCTURAL ³ |
| - | - | 14 | `Update__9C_grenadeFUi` | 8 | shared ⁴ |
| - | - | 19 | `SaveData__9C_grenadeFP7C_chunk` | 8 | shared ⁴ |
| - | - | 20 | `LoadData__9C_grenadeFP7C_chunk` | 8 | shared ⁴ |

¹ MSVC construct. ² §3a. ³ §4 — the PS2 `NetDirect` body is empty. ⁴ No PC function of its own:
PC slot 14 = `thunk_bottle_update` (PS2 `Update` is a tail jump); slots 19 and 20 =
`stub_rettrue_args4_00477710` (PS2 `return 1;`).

**The shared occupants carry no `PS2 twin:` line.** `stub_rettrue_args4_00477710` sits in 37
vtables; a docblock cannot say "one of thirty-seven", a slot table can. The mapping lives here only.

### 3a. Slot 18 is an ICF fold across three `C_bottle` subclasses

A scan of `.rdata` for `0x0048af90` as a pointer finds **three** references, all at slot 18:

| vtable | class |
|---|---|
| `0x0063be18` | `C_cartridge` (`structs/cartridge.md`) |
| `0x0063c3c0` | **`C_grenade`** |
| `0x0063cd80` | `C_smoke_trail`, kind `0xf`, ctor `smoke_trail_ctor` @ `0x004f60e0` (`structs/smoke_trail.md`) |

All three are `C_bottle` subclasses, the body is `frame ? bottle_init(frame) : false`, and the PS2
side has three separate 44-byte bodies that are identical to each other — which is precisely why
`/OPT:ICF` merged them. So the function belongs to **no single class** and is named for the family:
`bottle_subclass_init_require_frame`. The name `grenade_init` would assert one owner out of three —
the ICF-fold trap `structs/actor_track.md` §2b describes.

---

## 4. The network path

`grenade_apply_net_event_msg` (slot 38) has **no PS2 symbol at all**, and
`grenade_net_apply_remote_state` (slot 15) maps to an **empty 8-byte** PS2 body. The PS2 port emptied
the per-class replication bodies (`structs/bottle.md` §3a), so here the PS2 method count is a
**lower** bound and the PC code is real.

The two are wired to each other through `+0x15c`: the ctor sets it, the type-`0x16` message clears it
while a remote pose snap is being applied and restores it afterwards, and
`grenade_net_apply_remote_state` refuses to run the `C_bottle` dead-reckoning integrator while it is
clear.

Message types this class handles itself, on top of whatever `C_bottle` does:

| type | sub-type | effect |
|---|---|---|
| `0x13` | payload state 2 | hand the actor to `game_remove_temporary_actor` (retire it) |
| `0x16` | `5` | snap `+0x24`..`+0x2c` from the mirror at `+0x108`..`+0x110`, run the frame-update virtual, clear `+0x15c` |
| `0x16` | anything else | set `+0x15c` back to 1 |

**The spawn message, type `0x17`.** `game_new_grenade` — which builds Molotovs as well as grenades
(§1) — is the **sole producer** of descriptor `g_event_msg_vt_grenade_spawn` (`.rdata 0x0063e2f4`).
It constructs the message with `event_msg_actor_state_ctor(0x17, …)` and then overwrites the vtable
with its own descriptor, so a thrown object is **spawned over the network as type `0x17`**, carrying
the actor-state base plus one dword at payload `+0x1c`. The receive half is
`grenade_apply_net_event_msg` (slot 38). Full write-up: `structs/net_event_msg.md`.

---

## 5. Three `C_game` methods this class's AI calls

A statement-level diff of `grenade_ai_tick` against `AI__9C_grenadeFUi`, auditing every callee at
every position, gives three `C_game` twins. All three are recorded in `structs/game.md`; listed here
because this class supplies the evidence.

| PC function | PS2 twin |
|---|---|
| `game_new_explosion` | `NewExplosion__6C_gameFP7C_actorR8S_vectorffbbbi` |
| `game_new_fire` | `NewFire__6C_gameFR8S_vectorP9I3D_frameiffb` |
| `game_remove_temporary_actor` | `RemoveTemporaryActor__6C_gameFP7C_actor` |

`game_new_explosion` is also called from `crash_obj_ai` with a matching argument list; two unrelated
classes calling it in the same PS2-mirrored position is what makes the twin firm.

`game_new_fire` builds a `C_fire` (`structs/fire.md`).

---

## 6. PC vs PS2

* **The class's own band is a constant `+0x58` from the PS2 layout** — PC `+0x138`..`+0x15c` <-> PS2
  `+0x190`..`+0x1b4` — and the base bands differ by `+0x64` (`+0x84` <-> `+0xe8`) and `+0x40`
  (`+0x1c` <-> `+0x5c`). Do **not** rebase band-wise across the boundary.
* **The savegame record differs in size and that is real:** PC advances the stream by `0x1d`, PS2 by
  `0x20`. Counting the fields the writer actually emits gives exactly `0x1d` bytes, so the PS2 record
  is padded to a 4-byte multiple and the PC one is packed. A savegame is not portable between builds
  anyway.
* **Both builds call the `C_actor` base saver, not the `C_bottle` one**, in
  `SaveGameSave`/`SaveGameLoad`. Unusual, matched exactly, and a strong anchor.
* **`C_ff_manager` is not missing from the PC vtable run.** The PS2 file order is
  `C_DropOut.cpp -> C_ff_manager.cpp -> C_grenade.cpp -> C_human.cpp`, so the vtable sequence appears
  to skip a class. It does not: `__vt__12C_ff_manager` occurs **0 times in the PS2 decomp**, i.e. the
  class is non-polymorphic on *both* builds and can never appear in a vtable run — the same holds for
  `C_driver` and `C_animations_machine_de_luxe`. See `structs/force_feedback.md` for the class.
* **PS2 `Update` is a tail jump, not a stub.** 8 bytes on MIPS is either `jr ra; nop` or a tail
  call, and Ghidra renders the tail call as its TARGET's body. The PC mirrors it exactly, with a
  5-byte `jmp` thunk in slot 14.

---

## 7. Related

`structs/bottle.md` (the base) · `structs/cartridge.md` and `structs/smoke_trail.md` (share the
slot-18 fold) · `structs/entity.md` §1 (the kind map) · `structs/game.md` (the three factories) ·
`structs/fire.md` (the Molotov payload's object) · `structs/actor_track.md` §2b (the ICF-fold trap) ·
`structs/net_event_msg.md`.

---

## 8. Open items

| Item | State |
|---|---|
| `+0x150` / `+0x154` / `+0x158` | forwarded to `game_new_fire` un-inspected on both builds; their meanings come from `C_fire`, not from here |
| `+0x144` / `+0x148` | designer-set; no writer other than the savegame loader is located, so where the radius and damage come from is untraced |
| what `+0x80 <= 50.0` measures | a `C_bottle` base float. The retire gate is certain; the quantity is not |
| where `C_ff_manager`'s PC code lives | not settled by this class's evidence; nothing here supports a PS2-port-only tag either |
| in-game, untested | a Molotov and a grenade should differ ONLY in these flags. Throwing each near a wall and watching whether fire and blast can ever co-occur would settle whether the two arms are exclusive in the data |
