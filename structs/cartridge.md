# `C_cartridge` — the ejected shell casing

**Created** 2026-08-12 20:08 +0200 · **Modified** 2026-11-15 18:48 +0100

A `C_bottle` subclass: the **spent cartridge case flicked out of a firing weapon**. It inherits the whole ballistic arc
from `C_bottle` and adds only a landing sound and a retirement rule. `cartridge_` maps the PS2 class tag `11C_cartridge` onto a
lowercase prefix; the English word is also what the thing is. The prefix is not shadowed by `car_`.

| | |
|---|---|
| PC prefix | `cartridge_` |
| PC kind (`+0x10`) | `0x10` |
| PC vtable | `g_cartridge_vt` @ `0x0063be18` (39 slots) |
| PC base class | `C_bottle` (`structs/bottle.md`) |
| PC instance size | `0x13c` (`operator_new` in `game_init`) |
| PC constructor | `cartridge_ctor` @ `0x00476f80` |
| PC translation unit | `[0x00476f50, 0x00477300)`, guard to guard: 10 starts, all named |
| PS2 class | `C_cartridge`, data only, no member functions (§7) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class adds **one** field; `+0x000..+0x137` is `C_bottle` (`structs/bottle.md` §1).

```c
typedef struct C_cartridge {                  // 0x13c bytes; C_bottle base +0x000..+0x137
    /* +0x138 */ uint8_t type;                // [H] 1 = shotgun shell, 0 = any other gun
} C_cartridge;
```

## 2. Field evidence

Measured by a register-tag census of the class's functions and by reading every body.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x138 | type | H | `cartridge_set_type` | `cartridge_ai_tick` | 1 = shotgun shell (§4a); selects the landing-sound column and halves `+0x88`; a bool per the spawner's PS2 signature (§7c) |

### 2a. Inherited fields this class re-purposes or pins down

| Field (`bottle.md` name) | What `C_bottle` does | What `C_cartridge` does |
|---|---|---|
| `+0x6d` `may_broadcast` | ctor sets 1 | `cartridge_ctor` clears it: a casing never replicates |
| `+0xfc` `coll_registered` | ctor sets 1 | `cartridge_ctor` clears it: no dynamic-collision proxy is registered |
| `+0xa4` `radius` | `bottle_game_init` computes it from the model's bound volume | `cartridge_game_init` overwrites it with `0.1f` |
| `+0xa8` `frame_axis` | the second axis passed to `SetDir3`, tumbled each tick | set to `(0, 1, 0)` |
| `+0x84` `landed_sets_state` | chooses act-state 1 vs 2 at flight end | cleared |
| `+0x88` `restitution` | speed kept per bounce, GameInit = `0.4f` | `cartridge_set_type` sets `0.2f` for a shotgun shell, so a shell bounces lower |
| `+0x100` `bounce_count` | +1 per ground contact in `bottle_ai_tick` | `cartridge_ai_tick` retires the casing past 15 bounces |
| `+0x104` `on_ground` | the ground ray hit this tick | forces the landing-sound path |
| `+0x134` `ray_all_layers` | clear: the ground ray tests mask 2 | `cartridge_game_init` sets it, so a casing's ground ray tests every layer |
| `+0x12c` | part of the saved tail | the surface material id of the last contact, the index into `g_surface_material_table` |

## 3. Method family

PC-only by measurement: the PS2 has no `C_cartridge` member function (§7a), so every twin cell is `-`. Every override of the
`C_bottle` vtable has a row: slots 13–16, 18, 22–26, 30 and 31. A derived class cannot "un-override" its base, so the foreign
symbols in slots 15, 18, 22–24, 30 and 31 are empty or trivial overrides that identical-COMDAT-folding merged onto other
classes' identical ones. `game_new_cartridge` (`structs/game.md`) exists only to serve this class (§4a).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00476f50 | `crt_static_init_guard_00476f50` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00476f70 | `crt_atexit_register_00476f70` | the translation unit | - | - | - | registers the unit's no-op static destructor |
| 0x00476f80 | `cartridge_ctor` | instance | - | - | CONFIRMED | `bottle_ctor`, vtable, clears `+0xfc` and `+0x6d`, kind `0x10`; called from `game_init` for each pool element |
| 0x00476fb0 | `cartridge_scalar_deleting_dtor` | instance | - | - | CONFIRMED | slot 16 |
| 0x00476fd0 | `cartridge_dtor` | instance | - | - | CONFIRMED | restores the vtable, `bottle_dtor` |

### 3b. Accessors

None: `cartridge_set_type` also writes `+0x88`.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00462330 | `stub_rettrue_args4_00462330` | - | - | - | slots 23 and 24; a shared stub |
| 0x00462420 | `stub_ret0_cdecl_00462420` | - | - | - | slot 22; a shared stub: a casing has no savegame data |
| 0x00476fe0 | `cartridge_ai_tick` | - | - | CONFIRMED | slot 13; `bottle_ai_tick`, then the landing sound and the retirement rule (§5) |
| 0x004771f0 | `thunk_bottle_update` | - | - | STRUCTURAL | slot 14; a 5-byte `jmp bottle_update` |
| 0x00477200 | `cartridge_game_init` | - | - | CONFIRMED | slot 25; `bottle_game_init`, then the overrides of §2a and `+0x65` = 1 |
| 0x00477260 | `cartridge_game_done` | - | - | CONFIRMED | slot 26; copies the frame's position, rotation and parent back to the actor, `bottle_game_done` |
| 0x004772e0 | `cartridge_set_type` | - | - | STRUCTURAL | non-virtual; stores `type`, and a non-zero type sets `+0x88` to 0.2 |
| 0x00482690 | `stub_retfalse_args24_00482690` | - | - | - | slot 30, `Collision`; a shared stub |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | - | - | - | slot 31, `Hit`; a shared stub |
| 0x0048af90 | `bottle_subclass_init_require_frame` | - | - | STRUCTURAL | slot 18; `frame ? bottle_init(frame) : false`, a fold shared with `C_grenade` and `C_smoke_trail` (`structs/grenade.md` §4c); its twins belong to those classes |
| 0x004f61d0 | `thunk_bottle_net_apply_remote_state` | - | - | STRUCTURAL | slot 15; a jump to `bottle_net_apply_remote_state`, shared with `C_smoke_trail` (2 datarefs) |

### 3d. Free helpers

None.

## 4. How the class was identified

The class sits in the PC `.rdata` vtable order directly after `C_car_ex`. Five levers agree, and no two of them share a premise;
levers 3 and 4 are read on the PS2 and sit in §7e:

| # | Lever | Result |
|---|---|---|
| 1 | PC `.rdata` vtable order | `0x0063bd68` (`C_car_ex`) → `0x0063be18` → `0x0063beb8` (`C_clock`) |
| 2 | link order by source file | `C_car_ex.cpp` → `C_cartridge.cpp` → `c_clock.cpp` |
| 5 | PC-side, independent of 1–3 | the ctor chains `bottle_ctor` and writes kind `0x10`; `operator_new(0x13c)` against `C_bottle`'s `operator_new(0x138)` |

Lever 5's arithmetic is the sharpest of the set: the subclass adds exactly one dword, and `+0x138` is the one field the class
uses. The ctor census of `C_bottle` subclasses finds two more besides this one: `grenade_ctor` (`0x0048af30`) and `smoke_trail_ctor`
(`0x004f60e0`), matching PS2's `C_grenade` and `C_smoke_trail`.

### 4a. The two pools

`game_init` builds two ring buffers on the `C_game` singleton from two shipped models (*nabojnice* is Czech for a cartridge case):

| Pool | Model | Entries | Array | Cursor |
|---|---|---|---|---|
| A | `2nabojnice.i3d` | 100 | `C_game+0x3a8` | `C_game+0x538` |
| B | `2nabojniceb.i3d` | 30 | `C_game+0x53c` | `C_game+0x5b4` |

**Pool B is the shotgun shell.** The type the spawner receives selects the pool: `human_ai` passes bit 1 of the fired weapon's
`predmety.def` flag word `+0x20`, and only the pump shotgun (id 11), the sawed-off shotgun (id 12) and the unused silent pump shotgun
(id 34) carry that bit. Every other gun ejects into pool A, and so do the revolver reload (`human_start_weapon_effect`, six casings
from the `cylinder` frame) and `game_car_gun_tick`.

Both are built through the frame named `Line01`, each element `operator_new(0x13c)` + `cartridge_ctor`. `game_new_cartridge` takes the
next slot, wraps at the pool size, and never allocates at runtime. `game_done` and `game_done_intern` release both pools.

## 5. What `cartridge_ai_tick` does

1. Chain `bottle_ai_tick`, the inherited ballistics.
2. **Spent?** (`flight_speed` < 0.01 and `fall_accum` < 0.01), or `fall_accum` > 50.0, or `+0x100` > 15.
3. If spent, or `+0x104` is set, and the player frame is within 100.0 squared units: walk the three `g_cartridge_sounds` slots for a free
   one, build a sound from `g_surface_material_table[+0x12c].{+0x28 | +0x2c}`, the column chosen by `type`, place the emitter at the
   casing's position, rewrite the emitter frame's flag word `& 0xfffffedf | 0x40000000` (the project's documented sound-emitter
   fingerprint) and attach it to the mission scene.
4. If spent: `game_invalidate_actor` + `game_remove_temporary_actor`, so the slot returns to the pool; the act-state 2 that
   `game_remove_temporary_actor` writes is the "spent" state.

## 6. The vtable — `0x0063be18`, 39 slots

Slot-for-slot aligned with `C_bottle`'s `0x0063b870`; §3 lists the overrides. Holder counts of the shared symbols: `0x00482690` sits in
16 vtables, `0x0048a4f0` in 13, `0x00462330` in 14.

## 7. PC vs PS2

### 7a. The PS2 side of this class is data only

PS2 emits no `C_cartridge` member function at all. Three structurally different searches agree (a plain text grep of the PS2 decomp can
miss a definition whose line Ghidra wrapped, so the symbol table and the VA span are checked too):

1. the only `11C_cartridge` token in the entire PS2 decomp is `sounds__11C_cartridge`;
2. the PS2 symbol table has zero rows whose class is `C_cartridge`; the only cartridge-bearing symbols in the whole build are
   `ClearCartridges__6C_gameFv`, `NewCartidge__6C_game...` and `__sinit_C_cartridge.cpp`;
3. the PS2 VA span between `C_car_ex`'s last method (`0x146790`) and `C_clock`'s first (`0x1478c0`) holds only `C_car_ex` container helpers.

### 7b. Why: the port gutted the spawner

`NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` is an **8-byte `{ return; }` stub** on PS2, against 1488 bytes on the PC. Nothing on the
PS2 ever creates a casing, so the linker had nothing to keep. The port did not delete the concept: it ships `ClearCartridges__6C_gameFv`
(192 B) and both pools, at 10 and 5 entries against the PC's 100 and 30, and it ships the class's static sound array.

So in this subsystem the PS2 method count is a lower bound, not an upper bound.

### 7c. The one twin that can be graded, and why it is STRUCTURAL

`game_new_cartridge` ↔ `NewCartidge__6C_game...`. The symbol match is exact: the PS2 signature `FRP9I3D_frameR8S_vectorbb` =
`(I3D_frame *&, S_vector &, bool, bool)` maps argument for argument onto the PC function, in order, including the double indirection on the
frame (the PC body reads `*param_2`). The trailing `bb` is why `+0x138` is a bool. But an empty PS2 body confirms the symbol and never the
bodies, so the grade stays STRUCTURAL.

### 7d. Field bridge

With no PS2 method bodies there is no PS2 field evidence for this class. The only cross-build datum is `g_cartridge_sounds` ↔
`sounds__11C_cartridge`, which is a static, not an instance field.

### 7e. The two PS2 identification levers (§4)

| # | Lever | Result |
|---|---|---|
| 3 | the source-file manifest | `__sinit_C_cartridge.cpp` exists, so the file was compiled into the build |
| 4 | a static class member | `sounds__11C_cartridge` ↔ PC `g_cartridge_sounds`, 3 slots, same walk in the same teardown position |

## 8. Related

- `structs/bottle.md` — the base class.
- `structs/grenade.md`, `structs/smoke_trail.md` — the other two `C_bottle` subclasses.
- `structs/game.md` — `game_new_cartridge` and the two pools.

## 9. Open items

| Item | State |
|---|---|
| `human_ai`'s weapon-flag bit and the `predmety.def` ids of §4a | taken as written, not re-read for this revision |
