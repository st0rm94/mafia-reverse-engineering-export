# `C_box_new` — the physics-base box that replaced `C_box`

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-10-05 15:41 +0200

A thin `C_PhThing` subclass: it holds a **six-field descriptor**, decides at mission start whether
the body behaves as a *box* or as a *wheel*, hands that descriptor to the shared physics base, and
does nothing else. All dynamics live in `C_PhThing` (`structs/physical_object.md`).

| | |
|---|---|
| PC prefix | **`box_new_`** |
| PC kind (`entity+0x10`) | **`0x23`** — not 9; kind 9 is the PC-only `C_box` (`structs/box.md`, §3a) |
| PC vtable | `.rdata 0x0063baa0` (39 slots) |
| PC translation unit | `[0x0045f160, 0x0045f930)` — 11 function starts, all accounted; CRT static-init helpers sit on both sides |
| Base class | **`C_PhThing`** → `C_actor`. `box_new_ctor` → `physical_object_ctor`, `box_new_init` → `physical_object_init`, `box_new_game_init` → `physical_object_game_init` |

## 1. Field layout

Confidence tags: **[H]** byte-proven from a writer *and* a reader · **[M]** one side traced.

The class's own state is six fields plus one flag. Every one of them has **four** independent
witnesses — `box_new_init` seeds it, `box_new_save_data` writes it, `box_new_load_data` reads it
back, and `box_new_duplicate_from` copies it — and all four agree on the order, which is why the
whole band is [H].

```c
typedef struct C_box_new {                /* extends C_PhThing */
    /* +0x00  */ void      *vptr;             // 0x0063baa0
    /* ...    */ C_PhThing  base;             // structs/physical_object.md
    /* +0x10  */ int        kind;             // [H] = 0x23
    /* +0x208 */ int        base_state_a;     // [M] C_PhThing field, zeroed by GameInit
    /* +0x20c */ float      base_state_b;     // [M] C_PhThing field, set to -1.0
    /* +0x210 */ int        base_state_c;     // [M] C_PhThing field, zeroed
    /* +0x3b0 */ char       is_box;           // [H] 1 = S_THING_BOX, 0 = S_THING_WHEEL
    /* +0x3b4 */ float      wheel_radius;     // [H] default 1.0   (wheel only)
    /* +0x3b8 */ float      wheel_param;      // [H] default 1.0   (wheel only) -> C_PhThing +0x21c
    /* +0x3bc */ float      mass;             // [H] default 10.0; overwritten from the model volume
    /* +0x3c0 */ float      friction;         // [H] default 0.5
    /* +0x3c4 */ float      damping;          // [H] default 0.01  (wheel only)
    /* +0x3c8 */ char       touched;          // [H] set by Collision and Hit, cleared by GameInit
} C_box_new;
```

**The mass is derived from the model, not authored.** `box_new_init` refreshes the frame's bound
volume, computes `dx * dy * dz` from the bbox at `frame+0xb0`, applies the frame scale, and when
`0.01 < volume < 1000.0` overwrites `+0x3bc` with `volume * 50.0`. Outside that band the 10.0
default stands — a degenerate or absurdly large model falls back rather than producing a zero or
runaway mass.

**Which fields reach the base depends on `+0x3b0`.** `box_new_game_init` builds one of two
descriptor structs on the stack:

| `+0x3b0` | struct | fields passed |
|---|---|---|
| set | `S_THING_BOX` | `+0x3bc` (mass), `+0x3c0` (friction) |
| clear | `S_THING_WHEEL` | `+0x3b4`, `+0x3b8`, `+0x3bc`, `+0x3c0`, `+0x3c4` |

So `wheel_radius` / `wheel_param` / `damping` are **wheel-only** — dead storage for a box-configured instance,
which still round-trips through the savegame.

### 1a. The descriptor layouts (PC)

Read from the stack stores in `box_new_game_init` and from the loads in
`physical_object_init_box` / `physical_object_init_wheel`:

| struct | offset | field | source | destination in `C_PhThing` |
|---|---|---|---|---|
| `S_THING_BOX` | `+0x00` | mass | `+0x3bc` | `+0x190` |
| `S_THING_BOX` | `+0x04` | initial velocity (vec3) | 0 | `+0x1e0` |
| `S_THING_BOX` | `+0x10` | friction | `+0x3c0` | `+0x1f8` |
| `S_THING_WHEEL` | `+0x00` | wheel radius | `+0x3b4` | `+0x220` |
| `S_THING_WHEEL` | `+0x04` | wheel parameter, read by `physical_object_move_wheel` (`structs/physical_object.md` §4b) | `+0x3b8` | `+0x21c` |
| `S_THING_WHEEL` | `+0x08` | mass | `+0x3bc` | `+0x190` |
| `S_THING_WHEEL` | `+0x0c` | initial velocity (vec3) | 0 | `+0x1e0` |
| `S_THING_WHEEL` | `+0x18` | friction | `+0x3c0` | `+0x1f8` |
| `S_THING_WHEEL` | `+0x1c` | damping | `+0x3c4` | `+0x224` |

A velocity below `1e-05` squared puts the body at rest (act state 1); `box_new_game_init`
always passes zero, so every `C_box_new` starts at rest.

## 2. Method coverage — 10/10 PS2 methods mapped; 11/11 PC starts accounted

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x0045f160` | `box_new_ctor` | `__ct__9C_box_newFv` | - | CONFIRMED |
| `0x0045f180` | `box_new_scalar_deleting_dtor` | - | 16 | STRUCTURAL ¹ |
| `0x0045f1a0` | `box_new_dtor` | `__dt__9C_box_newFv` | - | CONFIRMED |
| `0x0045f1b0` | `box_new_init` | `Init__9C_box_newFP9I3D_frame` | 18 | CONFIRMED |
| `0x0045f2c0` | `box_new_game_init` | `GameInit__9C_box_newFv` | 25 | CONFIRMED |
| `0x0045f390` | `thunk_physical_object_game_done` | `GameDone__9C_box_newFv` | 26 | CONFIRMED ² |
| `0x0045f3a0` | `box_new_load_data` | `LoadData__9C_box_newFP7C_chunk` | 20 | CONFIRMED |
| `0x0045f720` | `box_new_save_data` | `SaveData__9C_box_newFP7C_chunk` | 19 | CONFIRMED |
| `0x0045f890` | `box_new_duplicate_from` | `DuplicateFrom__9C_box_newFP7C_actor` | 21 | CONFIRMED |
| `0x0045f8f0` | `box_new_on_collision` | `Collision__9C_box_newF…` | 30 | CONFIRMED |
| `0x0045f910` | `box_new_hit` | `Hit__9C_box_newF…` | 31 | CONFIRMED |
| - | *(inherited)* | - | 13/14/15/22/23/24/27/38 | all point into the `C_PhThing` band |

¹ MSVC-only construct. ² A 5-byte `jmp` to `physical_object_game_done`, referenced from the vtable
at `.rdata 0x0063bb08`; the PS2 `GameDone__9C_box_newFv` is the same 8-byte tail-jump.

### 2a. Slot 21 = `DuplicateFrom` for the whole `C_actor` family

`box_new_duplicate_from` is **96 bytes on both builds**, copies the same six fields in the same
order after a base call, and sits in slot 21. In every sibling class that does not override it,
slot 21 holds a folded no-op stub: `C_bottle` has `thunk_stub_noop_args4_00461180` (`0x004773f0`,
a 5-byte `jmp` thunk), `C_box` has `stub_noop_args4_004776f0` (a real body). The family-wide slot
map is in `structs/actor_track.md`.

## 3. PC vs PS2

### 3a. The kind map: `C_box_new` is `0x23`, kind 9 is `C_box`

Two PC observations point at kind 9 and both fit `C_box`, a class the PS2 does not have:

- link order puts kind 9's vtable immediately after `C_bottle`'s, where `C_box_new` sits on PS2;
- `recorder_new_track` labels kind 9's recorder tracks `box%d`, i.e. "a box".

The PS2 constructors settle it: **`__ct__9C_box_newFv` writes `0x23`** to `+0x50`. Extracting that
write from every PS2 `__ct__*` gives the kind for all 33 PS2 classes that write it, and the table
cross-validates against the PC on **twelve** documented classes (actor=1, player=2, human=3,
car=4, don_ocas=0x13, bridge=0x14, airplane=0x16, DropOut=0x11, crash_obj=0x1a, entity=0x1b,
actor_shot=0x21, bottle=0xa). It also gives **kind `0x20` = `C_public_physics`** and **kind `0x25`
= `C_drop_in_weapon`**. `recorder_new_track` only knows the kinds the PC has.

The hierarchy is the same on both builds: `box_new_ctor` chains to `physical_object_ctor` exactly
as `__ct__9C_box_newFv` chains to `__ct__9C_PhThingFv`.

### 3b. Body-diff evidence, per method

| method | the falsifiable evidence |
|---|---|
| `box_new_init` | **five shared magic constants in order** — `1.0 / 1.0 / 10.0 / 0.5 / 0.01` at PC `+0x3b4..+0x3c4` and PS2 `+0x644..+0x654` — plus the identical `0.01 / 1000.0 / ×50.0` volume ladder |
| `box_new_game_init` | same preamble, same two-arm branch, same argument sets, same base call, same trailing flag clear; the PS2 test is **inverted** (`if (+0x640 == 0) InitWheel else InitBox`) |
| `box_new_save_data` / `_load_data` | the same six fields in the same order; PS2 writes version byte `2` |
| `box_new_duplicate_from` | same six fields, **96 B on both builds** |
| `box_new_on_collision` / `_hit` | `*(this+0x3c8) = 1` then the base handler; PS2 is `*(this+0x658) = 1` then the same |
| `box_new_ctor` / `_dtor` | base call + vtable + kind, statement-for-statement |

### 3c. The field-band mapping this class pins

| PC | PS2 | field |
|---|---|---|
| `+0x3b0` | `+0x640` | `is_box` |
| `+0x3b4` | `+0x644` | `wheel_radius` |
| `+0x3b8` | `+0x648` | `wheel_param` |
| `+0x3bc` | `+0x64c` | `mass` |
| `+0x3c0` | `+0x650` | `friction` |
| `+0x3c4` | `+0x654` | `damping` |
| `+0x3c8` | `+0x658` | `touched` |

A constant `+0x290` delta across the band — but **do not rebase other fields from it**
(`structs/actor.md` §1a: the PC↔PS2 deltas in this family are not constant, and two measured pairs
are negative).

### 3d. `C_PhThing` twins located through this class

`box_new_game_init`'s inverted branch locates two `C_PhThing` entry points, its tail a third, and the
collision/hit pair the last two. All five are body-diffed on both sides and CONFIRMED:

| located by | PC name | PS2 twin | what the diff turns on |
|---|---|---|---|
| tail of `box_new_game_init` | `physical_object_game_init` | `GameInit__9C_PhThingFv` | the `1e-08` scale ladder, the `0x40` written into the freshly zeroed dyn-coll object |
| the box arm of that branch | `physical_object_init_box` | `InitBox__9C_PhThingFP11S_THING_BOX` | extents `+0.05`, the six-comparison `2.0 / 0.5` aspect gate, `1/(m·L²/3)` |
| the wheel arm (PS2's test is inverted) | `physical_object_init_wheel` | `InitWheel__9C_PhThingFP13S_THING_WHEEL` | `0x3cf5c28f`, `0x3f866666`, and PS2's `/(0.03·0.9)` folded by MSVC into `×37.03704` |
| tail of `box_new_on_collision` | `physical_object_on_collision` | `Collision__9C_PhThingF…` | impulse `×1.1`, the deepest-hit record on `+0x30c ↔ +0x5e4` |
| tail of `box_new_hit` | `physical_object_hit` | `Hit__9C_PhThingF10E_hit_type…` | the identical `1e-08` normalise-with-degenerate-fallback tree, offsets `0.025 / 0.06 / 0.04` |

A statement-position match inside a confirmed twin is a lead, not a twin: the two `CollisionCount`
specialisations do **not** resolve to a 1:1 twin that way (`structs/physical_object.md` §6c).

### 3e. Descriptor field order

The PS2 puts the initial velocity (a 16-byte `S_vector`) FIRST in both descriptors:
`S_THING_BOX` = {velocity, mass `+0x10`, friction `+0x14`}, `S_THING_WHEEL` = {velocity,
radius `+0x10`, parameter `+0x14`, mass `+0x18`, friction `+0x1c`, damping `+0x20`}. The PC puts the
scalars that precede friction first and the 12-byte velocity after them (§1a). Both builds'
`GameInit` fill the velocity with zeros, and both builds' `InitBox` / `InitWheel` read each
field from its own build's offset, so the difference is a declaration order, not a behaviour.

## 4. Related

- `structs/physical_object.md` — `C_PhThing`, where all the dynamics live (26/26 methods named).
- `structs/box.md` — `C_box`, the PC-only class this one replaced (kind 9).
- `structs/entity.md` — the kind enum (`0x23`, `0x20`, `0x25`).
