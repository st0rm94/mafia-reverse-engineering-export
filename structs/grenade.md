# `C_grenade` — the thrown grenade AND the Molotov (kind `0xe`)

**Created** 2026-08-13 20:11 +0200 · **Modified** 2026-11-09 10:33 +0100

A thrown explosive projectile. `C_bottle` supplies the physics (the arc, the bounce, the settle);
this class adds an impact sound keyed to the ground surface, a fuse with two independent trigger
modes and a payload that fires with it. One class builds both thrown weapons: the creator's type
argument decides whether what lands is a fragmentation grenade or a Molotov cocktail (§4).

| | |
|---|---|
| PC prefix | `grenade_` |
| PC kind (`+0x10`) | `0xe`, stamped by `grenade_ctor` |
| PC vtable | `g_grenade_vt` @ `0x0063c3c0`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_bottle` (`structs/bottle.md`), which derives from `C_actor` |
| PC instance size | `0x160` (`operator_new(0x160)` in `game_new_grenade`; the last field ends at `0x160`) |
| PC constructor | `grenade_ctor` @ `0x0048af30` |
| PC translation unit | `[0x0048af00, 0x0048b5b0)`: the unit's CRT pair, then the thirteen `grenade_*` functions and the slot-18 fold; the next unit's guard `crt_static_init_guard_0048b5b0` opens `C_human` |
| PS2 class | `C_grenade` (tag `9C_grenade`), 15 methods, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_bottle` base; offsets are PC.

```c
typedef struct C_grenade {                  // 0x160 bytes; PC offsets
    /* +0x000 */ C_bottle base;             // [H] the C_bottle band, 0x000..0x137 (structs/bottle.md)
    /* +0x138 */ uint8_t  fuse_timed_armed; // [H] the timed fuse runs
    /* +0x13c */ int32_t  fuse_ms_remaining;// [H] counts down by the frame delta; fires below 0
    /* +0x140 */ uint8_t  fuse_on_impact;   // [H] fires on the first ground contact instead
    /* +0x141 */ uint8_t  do_explosion;     // [H] the payload is an explosion
    /* +0x144 */ float    explosion_radius; // [H] argument of game_new_explosion
    /* +0x148 */ float    explosion_damage; // [H] argument of game_new_explosion
    /* +0x14c */ uint8_t  do_fire;          // [H] the payload is a fire
    /* +0x150 */ int32_t  fire_lifetime_ms; // [H] argument of game_new_fire
    /* +0x154 */ float    fire_radius;      // [H] argument of game_new_fire
    /* +0x158 */ float    fire_damage_rate; // [H] argument of game_new_fire
    /* +0x15c */ uint8_t  net_direct_enabled; // [H] gates the C_bottle dead-reckoning step
} C_grenade;
```

The two fuse modes and the two payloads are four independent bytes, not two enums: the creator
sets one fuse and one payload (§4a), and `grenade_ai_tick` tests each byte on its own.

## 2. Field evidence

Measured by a census of every `this+N` access in the `grenade_*` functions and by a body read of
`game_new_grenade`, the only external writer; no other function touches the band.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `grenade_ctor` (`bottle_ctor`, vtable `g_grenade_vt`, kind `+0x10` = `0xe`, `may_broadcast` `+0x6d` = 1), `grenade_game_init` (`bottle_game_init`, then `C_bottle+0x84` = 0) | the `C_bottle` methods of `structs/bottle.md`; `grenade_ai_tick`, `grenade_on_collision`, `grenade_hit` (§4b) | the inherited band |
| +0x138 | fuse_timed_armed | H | `game_new_grenade` (types 0 and 2), `grenade_game_init` (= 0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | byte 1 of the save record, after the version byte |
| +0x13c | fuse_ms_remaining | H | `game_new_grenade` (5000), `grenade_ai_tick`, `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | `grenade_ai_tick` subtracts its frame-delta argument while `+0x138` is set |
| +0x140 | fuse_on_impact | H | `game_new_grenade` (types 1 and 3), `grenade_game_init` (= 0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_on_collision`, `grenade_hit`, `grenade_save_game_save` | while set, every collision and hit adds 1 to `C_bottle+0x100` (`bounce_count`), and `grenade_ai_tick` fires once it is above 0 |
| +0x141 | do_explosion | H | `game_new_grenade` (types 0 and 1), `grenade_game_init` (= 0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | - |
| +0x144 | explosion_radius | H | `game_new_grenade` (15.0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | - |
| +0x148 | explosion_damage | H | `game_new_grenade` (400.0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | - |
| +0x14c | do_fire | H | `game_new_grenade` (types 2 and 3), `grenade_game_init` (= 0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | - |
| +0x150 | fire_lifetime_ms | H | `game_new_grenade` (5000), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | the fire's lifetime; 2000 ms or more gives it a looping sound (`structs/fire.md` §1) |
| +0x154 | fire_radius | H | `game_new_grenade` (2.5), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | the radius inside which the fire hurts (`structs/fire.md` §1) |
| +0x158 | fire_damage_rate | H | `game_new_grenade` (50.0), `grenade_save_game_load` | `grenade_ai_tick`, `grenade_save_game_save` | damage per second of the fire's remaining lifetime (`structs/fire.md` §1) |
| +0x15c | net_direct_enabled | H | `grenade_ctor` (1), `grenade_apply_net_event_msg` (0 / 1) | `grenade_net_apply_remote_state` | §4d |

## 3. Method family

Coverage: all **15** PS2 `9C_grenade` methods are in the tables below: **13 twins**
(11 CONFIRMED, `NetDirect` STRUCTURAL, `Init` CONFIRMED on the slot-18 fold) and **3 base stubs**
(`Update`, `SaveData`, `LoadData`) whose slots hold shared PC functions. Two PC functions have no
PS2 symbol: the scalar-deleting destructor and the network-event handler of slot 38 (§5). The
slots the class does not override are the `C_bottle` / `C_actor` defaults (`structs/bottle.md` §3).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0048af00 | `crt_static_init_guard_0048af00` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x0048af20 | `crt_atexit_register_0048af20` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0048af30 | `grenade_ctor` | instance | `__ct__9C_grenadeFv` | 0x00162db0 | CONFIRMED | `bottle_ctor`, vtable `g_grenade_vt`, `may_broadcast` `+0x6d` = 1, `+0x15c` = 1, kind `0xe` |
| 0x0048af60 | `grenade_scalar_deleting_dtor` | instance | - | - | - | vtable slot 16; the MSVC scalar-deleting destructor around `grenade_dtor` |
| 0x0048af80 | `grenade_dtor` | instance | `__dt__9C_grenadeFv` | 0x00162e00 | CONFIRMED | restores the vtable and chains `bottle_dtor` |

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004771f0 | `thunk_bottle_update` | `Update__9C_grenadeFUi` | 0x00163570 | base-stub | slot 14; a 5-byte `jmp` to `bottle_update`, as the 8-byte PS2 body is a tail-jump to `C_bottle::Update`. The same thunk fills slot 14 of `g_cartridge_vt` (`structs/cartridge.md`) |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__9C_grenadeFP7C_chunk` | 0x00163850 | base-stub | slots 19 and 20; returns true, as the 8-byte PS2 body does |
| 0x00477710 | `stub_rettrue_args4_00477710` | `LoadData__9C_grenadeFP7C_chunk` | 0x00163860 | base-stub | the same function as the row above, slot 20 |
| 0x0048af90 | `bottle_subclass_init_require_frame` | `Init__9C_grenadeFP9I3D_frame` | 0x00162e60 | CONFIRMED | slot 18; `frame ? bottle_init(frame) : false`, one body that `C_cartridge`, `C_grenade` and `C_smoke_trail` share (ICF fold, §4c) |
| 0x0048afb0 | `grenade_ai_tick` | `AI__9C_grenadeFUi` | 0x00162e90 | CONFIRMED | slot 13; the impact sound, the fuse and the payload (§4) |
| 0x0048b310 | `grenade_game_init` | `GameInit__9C_grenadeFv` | 0x00163580 | CONFIRMED | slot 25; `bottle_game_init`, then clears `C_bottle+0x84` and the four mode bytes |
| 0x0048b340 | `grenade_game_done` | `GameDone__9C_grenadeFv` | 0x001635c0 | CONFIRMED | slot 26; `bottle_game_done`, then releases the actor's frame (`+0x68`) through two of its virtuals |
| 0x0048b360 | `grenade_save_game_get_size` | `SaveGameGetSize__9C_grenadeFv` | 0x00163610 | CONFIRMED | slot 22; the `C_bottle` size plus `0x1d` |
| 0x0048b370 | `grenade_save_game_save` | `SaveGameSave__9C_grenadeFPPv` | 0x00163630 | CONFIRMED | slot 23; the `C_actor` base saver, then a version byte (1) and the ten fuse and payload fields, `0x1d` bytes |
| 0x0048b400 | `grenade_save_game_load` | `SaveGameLoad__9C_grenadeFPPv` | 0x001636e0 | CONFIRMED | slot 24; the `C_actor` base loader, then the same `0x1d`-byte record |
| 0x0048b490 | `grenade_on_collision` | `Collision__9C_grenadeFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x001637a0 | CONFIRMED | slot 30; counts the contact for the impact fuse, refuses while spent (act state 2), else `bottle_on_collision` |
| 0x0048b4c0 | `grenade_hit` | `Hit__9C_grenadeF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x001637f0 | CONFIRMED | slot 31; the same impact count, then `bottle_hit` unless spent; returns 1 |
| 0x0048b510 | `grenade_apply_net_event_msg` | - | - | - | slot 38; the type-`0x13` / `0x16` handler on top of `bottle_apply_net_event_msg` (§4d) |
| 0x0048b590 | `grenade_net_apply_remote_state` | `NetDirect__9C_grenadeFUi` | 0x00163840 | STRUCTURAL | slot 15; runs `bottle_net_apply_remote_state` only while `+0x15c` is set; the PS2 body is empty (§5) |

### 3d. Free helpers

The translation unit holds no free helper.

## 4. The fuse and the payload

### 4a. The four creator types

`game_new_grenade` loads **`2grenade.i3d` for types 0/1 and `2mol.i3d` for types 2/3**, and when its
fourth argument is set writes the fuse and the payload itself:

| Type | Model | Fuse | Payload |
|---|---|---|---|
| 0 | `2grenade.i3d` | timed, 5000 ms | explosion, radius 15.0, damage 400.0 |
| 1 | `2grenade.i3d` | on impact | explosion, radius 15.0, damage 400.0 |
| 2 | `2mol.i3d` | timed, 5000 ms | fire, lifetime 5000 ms, radius 2.5, damage rate 50.0 |
| 3 | `2mol.i3d` | on impact | fire, lifetime 5000 ms, radius 2.5, damage rate 50.0 |

No type sets both payload bytes. Its three callers:

| Caller | Type | When |
|---|---|---|
| `human_notify_callback` | 0 when the thrown item is the grenade (`predmety.def` id 15), else 3 | a human's throw |
| `human_shooting` | 1 | the S&W model 27 Magnum (id 7) fires while `C_game+0x2f14` bit `0x4` is set |
| `net_session_handle_event_msg` | from the message | a remote spawn; the fourth argument is 0, so no payload is set locally |

The prefix records the developers' class name, read off the mangled tag; it is not a claim that
every instance is a grenade.

### 4b. `grenade_ai_tick`

1. **Impact sound.** While `C_bottle+0x104` (`on_ground`) is set and `C_bottle+0x12c`
   (`ground_material`) indexes `g_surface_material_table`, the surface's impact sound is built and
   played.
2. **Fuse.** The timed fuse subtracts the frame delta from `+0x13c` and fires below 0; the impact
   fuse fires once `C_bottle+0x100` (`bounce_count`) is above 0.
3. **Payload**, only while `C_bottle+0x80` (`fall_accum`, `structs/bottle.md` §1) is at most 50.0 —
   a grenade that has fallen for more than about 5 s retires without firing:
   * `+0x141` → `game_new_explosion` with `+0x144` / `+0x148`;
   * `+0x14c` → `game_new_small_fires(4)`, particle 8, then `game_new_fire` with `+0x150` /
     `+0x154` / `+0x158`, attached to the frame of `C_bottle+0x130` (`ground_dyn_obj`, the dynamic
     object the grenade landed on) when there is one.
4. In every firing path the actor is handed to `game_remove_temporary_actor`.

`grenade_on_collision` and `grenade_hit` test the act state `C_actor+0x1c`: **2 = spent**, and a
spent grenade neither collides nor takes hits.

### 4c. Slot 18 is an ICF fold across three `C_bottle` subclasses

A scan of `.rdata` for `0x0048af90` as a pointer finds **three** references, all at slot 18:

| Vtable | Class |
|---|---|
| `0x0063be18` | `C_cartridge` (`structs/cartridge.md`) |
| `0x0063c3c0` | `C_grenade` |
| `0x0063cd80` | `C_smoke_trail`, kind `0xf`, ctor `smoke_trail_ctor` @ `0x004f60e0` (`structs/smoke_trail.md`) |

All three are `C_bottle` subclasses and the body is `frame ? bottle_init(frame) : false`, so the
function belongs to no single class and is named for the family; `grenade_init` would assert one
owner out of three. The PS2 side of the fold is §5.

### 4d. The network path

`grenade_apply_net_event_msg` (slot 38) and `grenade_net_apply_remote_state` (slot 15) are wired to
each other through `+0x15c`: the ctor sets it, the type-`0x16` message clears it while a remote
pose snap is applied and restores it otherwise, and `grenade_net_apply_remote_state` runs the
`C_bottle` dead-reckoning step only while it is set.

| Type | Sub-type | Effect |
|---|---|---|
| `0x13` | payload state 2 | hand the actor to `game_remove_temporary_actor` (retire it) |
| `0x16` | `5` | copy `C_bottle+0x108` (`net_pos`) into the position `C_actor+0x24`..`+0x2c`, call vtable slot 14 (`Update`) with 1, clear `+0x15c` |
| `0x16` | anything else | set `+0x15c` back to 1 |

**The spawn message, type `0x17`.** `game_new_grenade` is the sole producer of descriptor
`g_event_msg_vt_grenade_spawn` (`.rdata 0x0063e2f4`). It constructs the message with
`event_msg_actor_state_ctor(0x17, …)` and then overwrites the vtable with its own descriptor, so a
thrown object is spawned over the network as type `0x17`, carrying the actor-state base plus one
dword at payload `+0x1c`. The receive half is `grenade_apply_net_event_msg`. Full write-up:
`structs/net_event_msg.md`.

### 4e. Three `C_game` methods the AI calls

A statement-level diff of `grenade_ai_tick` against its PS2 twin, auditing every callee at every
position, gives three `C_game` twins, all recorded in `structs/game.md`:

| PC function | PS2 twin |
|---|---|
| `game_new_explosion` | `NewExplosion__6C_gameFP7C_actorR8S_vectorffbbbi` |
| `game_new_fire` | `NewFire__6C_gameFR8S_vectorP9I3D_frameiffb` |
| `game_remove_temporary_actor` | `RemoveTemporaryActor__6C_gameFP7C_actor` |

`game_new_explosion` is also called from `crash_obj_ai` with a matching argument list; two
unrelated classes calling it in the same PS2-mirrored position is what makes the twin firm.
`game_new_fire` builds a `C_fire` (`structs/fire.md`).

## 5. PC vs PS2

* **The class band is a constant `+0x58` from the PS2 layout** — PC `+0x138`..`+0x15c` ↔ PS2
  `+0x190`..`+0x1b4` — and the base bands differ by `+0x64` (`+0x84` ↔ `+0xe8`) and `+0x40`
  (`+0x1c` ↔ `+0x5c`). Do not rebase band-wise across the boundary.
* **The savegame record differs in size, and that is real:** PC advances the stream by `0x1d`, PS2
  by `0x20`. The fields the writer emits add up to exactly `0x1d` bytes, so the PS2 record is padded
  to a 4-byte multiple and the PC one is packed.
* **Both builds call the `C_actor` base saver, not the `C_bottle` one**, in `SaveGameSave` /
  `SaveGameLoad`: unusual, matched exactly, and a strong anchor.
* **Slot 18 (§4c).** `Init__9C_grenadeFP9I3D_frame` and `Init__13C_smoke_trailFP9I3D_frame` are two
  separate 44-byte PS2 bodies, statement-identical to each other and to the PC fold; `C_cartridge`
  has no PS2 methods at all (`structs/cartridge.md`). `/OPT:ICF` merged the identical bodies on the
  PC.
* **The network path (§4d).** `grenade_apply_net_event_msg` (slot 38) has no PS2 symbol, and
  `grenade_net_apply_remote_state` (slot 15) maps to an empty 8-byte PS2 body. The PS2 port emptied
  the per-class replication bodies (`structs/bottle.md` §9a), so the PS2 method count is a lower
  bound for this class and the PC code is real.
* **`Update`, `SaveData` and `LoadData` are 8-byte PS2 bodies.** 8 bytes of MIPS is either
  `jr ra; nop` or a tail call; `Update` is a tail call to `C_bottle::Update`, mirrored by the
  5-byte PC thunk in slot 14, and the two chunk methods return 1, mirrored by the shared
  return-true stub.
* **`C_ff_manager` is not missing from the PC vtable run.** The PS2 file order is
  `C_DropOut.cpp -> C_ff_manager.cpp -> C_grenade.cpp -> C_human.cpp`, so the vtable sequence
  appears to skip a class. It does not: `__vt__12C_ff_manager` occurs 0 times in the PS2 decomp, so
  the class is non-polymorphic on both builds, as are `C_driver` and
  `C_animations_machine_de_luxe` (`structs/force_feedback.md`).
* **How the class was identified.** Four independent levers agree: the vtable order in `.rdata`
  (exactly one vtable between `C_DropOut`'s `0x0063c320` and `C_human`'s `0x0063c468`), the kind
  byte (`structs/entity_kind.md` §1), the CONFIRMED twin of the only creator `game_new_grenade`, and
  the instance size, which the class band closes exactly.

## 6. Related

- `structs/bottle.md` — the base.
- `structs/cartridge.md`, `structs/smoke_trail.md` — the other two owners of the slot-18 fold.
- `structs/entity_kind.md` §1 — kind `0xe`.
- `structs/game.md` — the three factories and `game_new_grenade`.
- `structs/fire.md` — the Molotov payload's object.
- `structs/net_event_msg.md` — the spawn and replication messages.
- `structs/force_feedback.md` — the `C_ff_manager` unit before this one.

## 7. Open items

1. **Can fire and blast co-occur in play?** No creator type sets both payload bytes (§4a);
   throwing each kind near a wall in game would show whether the two arms are exclusive in play.
