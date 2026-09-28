# `C_smoke_trail` — entity kind `0xf`, the damage smoke plume

**Created** 2026-08-17 11:24 +0200 · **Modified** 2026-09-18 05:40 +0200

PC prefix **`smoke_trail_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0xf`** |
| PC vtable | `.rdata 0x0063cd80` |
| PC constructor | `smoke_trail_ctor` @ `0x004f60e0` |
| PC translation unit | `[0x004f60e0, 0x004f6270)` — 8 function starts, all named |
| Base class | **`C_bottle`** |
| PS2 class | `C_smoke_trail` (`13C_smoke_trail`), source file `C_smoke_trail.cpp`, 15 methods, all accounted (§3) |

`C_smoke_trail` is a **`C_bottle` subclass** — the same family as `C_cartridge`
(`structs/cartridge.md`), `C_grenade` (`structs/grenade.md`) and the bottle itself
(`structs/bottle.md`). It is a short-lived, self-retiring actor that owns one
particle-effect object and removes itself from the world once the effect has faded.

Every twin that owns a PC function of its own was body-diffed; the seven that do not are shared
stubs or folds and are listed in §3 with the reason.

## 1. Identity

| Fact | Value | How we know |
|---|---|---|
| Entity kind | **`0xf`** | `smoke_trail_ctor` writes `0xf` to `+0x10`; the PS2 constructor writes the same to `+0x50`, the proven kind pair (`structs/entity.md` §1) |
| Vtable | `0x0063cd80` | installed by the constructor and re-installed by the destructor |
| Base class | `C_bottle` | every method chains its `bottle_*` counterpart first |
| PC translation unit | `[0x004f60e0, 0x004f6270)` | bounded by the CRT static-init/atexit pairs `0x004f60b0`/`0x004f60d0` and `0x004f6270`/`0x004f6290`, which is where the compiler emits one TU's boundary |
| PC starts | **8** | an enumeration of the function definitions in the annotated decomp over that range; none is a phantom of the aggressive instruction finder — each of the eight has direct callsites or a data reference, sits behind function-start padding, and begins with a prologue rather than a stack teardown |

## 2. Field map

Only the fields this class's own methods touch are listed; everything below `+0x84` is
`C_bottle`/`C_actor` state documented in those docs.

| PC offset | PS2 offset | Field | Conf | Evidence |
|---|---|---|---|---|
| `+0x10` | `+0x50` | entity kind = `0xf` | H | both constructors |
| `+0x65` | `+0x69` | alive byte, set to 1 by `GameInit` | H | `smoke_trail_game_init` and its twin |
| `+0x68` | `+0x70` | **the particle-effect object** | H | `smoke_trail_update` dispatches one virtual on it; `smoke_trail_game_done` calls three |
| `+0x6d` | `+0x75` | cleared by the constructor | U | ctor only; no reader located |
| `+0x7c` | `+0xe0` | fade term A | H | `smoke_trail_ai_tick` retires when it and `+0x80` are both `<= 1e-08` |
| `+0x80` | `+0xe4` | fade/age term B | H | same test, plus the independent `> 50.0` retire gate |
| `+0x84` | `+0xe8` | cleared by `GameInit` | U | no reader located |
| `+0xa4` | `+0xf0` | `0.1f` seed | M | `GameInit`; shared constant `0x3dcccccd` on both builds |
| `+0xa8` | `+0xb0` | cleared by `GameInit` | U | - |
| `+0xac` | `+0xb4` | `1.0f` seed | M | `GameInit`; shared constant `0x3f800000` |
| `+0xb0` | `+0xb8` | cleared by `GameInit` | U | - |
| `+0xfc` | `+0x160` | cleared by the constructor | U | ctor only |

Confidence: **[H]** measured on both builds · **[M]** measured, meaning open · **[U]** written,
no reader located.

The PC↔PS2 delta is **not uniform** across the class (`+0x10`→`+0x50` is `+0x40`,
`+0x7c`→`+0xe0` is `+0x64`, `+0xa4`→`+0xf0` is `+0x4c`). Do not rebase band-wise. Each row above
was matched by ROLE inside a body-diffed method.

## 3. Method coverage — 15/15 PS2 methods accounted

| PC VA | PC name | PS2 symbol | PS2 VA | Status | Notes |
|---|---|---|---|---|---|
| `0x004f60e0` | `smoke_trail_ctor` | `__ct__13C_smoke_trailFv` | `0x00211e90` | CONFIRMED | vtable + kind `0xf`; 48/68 = 0.71 |
| `0x004f6110` | `smoke_trail_scalar_deleting_dtor` | - | - | STRUCTURAL | MSVC-only; vtable slot 16, the family's sdd slot |
| `0x004f6130` | `smoke_trail_dtor` | `__dt__13C_smoke_trailFv` | `0x00211ee0` | CONFIRMED | the PS2 body also carries the `delete` arm MSVC splits into the sdd |
| `0x004f6140` | `smoke_trail_ai_tick` | `AI__13C_smoke_trailFUi` | `0x00211f70` | CONFIRMED | vtable slot 13; shared magics `1e-08` and `50.0`; 112/160 = 0.70 |
| `0x004f61b0` | `smoke_trail_update` | `Update__13C_smoke_trailFUi` | `0x00212010` | CONFIRMED | vtable slot 14; one virtual on the effect object |
| `0x004f61d0` | `thunk_bottle_net_apply_remote_state` | `NetDirect__13C_smoke_trailFUi` | `0x00212060` | STRUCTURAL | a 5-byte `jmp` to the `C_bottle` implementation; the PS2 symbol is an 8-byte tail jump |
| `0x004f61e0` | `smoke_trail_game_init` | `GameInit__13C_smoke_trailFv` | `0x00212070` | CONFIRMED | vtable slot 25; 96/84 = 1.14 |
| `0x004f6240` | `smoke_trail_game_done` | `GameDone__13C_smoke_trailFv` | `0x002120d0` | CONFIRMED | vtable slot 26; the same 3-call teardown |
| - | *(folded)* | `Init__13C_smoke_trailFP9I3D_frame` | `0x00211f40` | STRUCTURAL | realised by the shared slot-18 fold **`bottle_subclass_init_require_frame`** (`0x0048af90`) ¹ |
| - | *(shared stub)* | `SaveGameGetSize` / `SaveGameSave` / `SaveGameLoad` | `0x00212140`/`50`/`60` | STRUCTURAL | 8-byte PS2 stubs; the PC realisations are shared no-op stubs used by dozens of vtables |
| - | *(shared stub)* | `Collision` / `Hit` | `0x00212170`/`80` | STRUCTURAL | 8-byte `return 0;` on PS2 |
| - | *(shared stub)* | `SaveData` / `LoadData` | `0x00212190`/`a0` | STRUCTURAL | 8-byte `return 1;` on PS2 |

¹ A scan of the data sections for `0x0048af90` as a pointer shows 3 owning vtables:
`C_cartridge`, `C_grenade`, `C_smoke_trail`.

**None of the shared realisations carries a `PS2 twin:` line in its docblock.** Each is referenced
by many vtables, so a docblock field would assert an ownership the data-section scan refutes; the
many-to-one mapping belongs in this table.

## 4. Behaviour

**Construction.** `smoke_trail_ctor` chains `bottle_ctor`, installs vtable `0x0063cd80`,
stamps entity kind `0xf` and clears two fields. `Init` (the shared fold) refuses a null
frame and otherwise delegates to the `C_bottle` initialiser — so a smoke trail is always
parented to a model frame.

**Arming.** `smoke_trail_game_init` chains `bottle_game_init`, then seeds the fade band:
`+0xa4 = 0.1f`, `+0xac = 1.0f`, three cleared words, and the alive byte `+0x65 = 1`.

**Per frame.** `smoke_trail_update` advances the `C_bottle` half and then ticks the
particle-effect object at `+0x68` through its own vtable. `smoke_trail_ai_tick` chains
`bottle_ai_tick` and applies the **retire test**, which has two independent arms:

```c
if ((+0x7c <= 1e-08 && +0x80 <= 1e-08) || 50.0 < +0x80) {
    game_invalidate_actor(this, 0);
    game_remove_temporary_actor(this);
}
```

— i.e. *the plume has faded to nothing*, **or** *it has simply aged out*. Both constants
are byte-shared with the PS2 body. `game_remove_temporary_actor` is the deferred retire
queue (`structs/game.md`), so the actor disappears at the end of the frame rather than
mid-tick.

**Teardown.** `smoke_trail_game_done` chains `bottle_game_done` and then releases the
effect object with three virtual calls in a fixed order (PC slots `0x24`, `0x2c(0,0)`,
`0x00`).

## 5. PC vs PS2

* **Field deltas are not uniform** — see the note under §2.
* **Vtable slot numbering differs**: PC slot `0x14` ↔ PS2 slot `0x28` for the effect
  object's per-frame call, and PC `0x24`/`0x2c`/`0x00` ↔ PS2 `0x38`/`0x40`/`0x08` for the
  teardown triple. The ORDER is identical on both builds, which is what makes the mapping
  safe.
* **Seven of the fifteen PS2 methods are 8-byte symbols**, and each was read rather than
  counted: `NetDirect` is a tail jump (the PC mirrors it with a 5-byte `jmp` thunk), the
  `SaveGame*` triple and `SaveData`/`LoadData` return constants, and `Collision`/`Hit` return 0.
* **`Init` is a three-way ICF fold on the PC**, shared with `C_cartridge` and `C_grenade`;
  the PS2 emits it per class.
* MSVC splits the destructor into `__dt__` plus a scalar deleting destructor; the PS2
  toolchain emits one function with a flag argument. That is why the PC has 8 starts
  against 15 PS2 symbols and both counts are complete.

## 6. Related

`structs/bottle.md` — the base class, and the slot map this class inherits ·
`structs/cartridge.md` and `structs/grenade.md` — the sibling subclasses; all three share
`bottle_subclass_init_require_frame` · `structs/entity.md` §1 — the kind map, row `0xf` ·
`structs/game.md` — the deferred retire queue.

## 7. Open items

| item | state |
|---|---|
| what CREATES a smoke trail | the constructor's only located callsite is the entity factory, and nothing traced so far passes kind `0xf` to it, so the producer — presumably the car damage path, which is what a smoke plume is for — is not located |
| the particle-effect object's class | `+0x68` holds an object with at least vtable slots `0x14`, `0x24`, `0x2c`, `0x00`; not identified, likely the same effect type the `game_new_*` factories return (`structs/grenade.md` §5) |
| `+0x6d`, `+0x84`, `+0xa8`, `+0xb0`, `+0xfc` | written by the ctor or `GameInit`, no reader located on either build |
| what `+0x7c` and `+0x80` measure | the retire gate is certain (`1e-08` fade, `50.0` age); the physical quantities are inherited from `C_bottle` and are `[U]` there too (`structs/bottle.md` §1, the `+0x80` row) |
| in game | a badly damaged car should trail smoke that fades and stops after ~50 units of `+0x80`; one observation would confirm the class is live at all and pin the producer |
