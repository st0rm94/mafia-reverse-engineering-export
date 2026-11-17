# `C_box_new` — the physics-base box that replaced `C_box`

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-11-02 17:43 +0100

A thin `C_PhThing` subclass: it holds a **six-field descriptor**, decides at mission start whether
the body behaves as a *box* or as a *wheel*, hands that descriptor to the shared physics base, and
does nothing else. All dynamics live in `C_PhThing` (`structs/physical_object.md`). Kind 9 is not
this class but the PC-only `C_box` (`structs/box.md`; the kind map is §5a).

| | |
|---|---|
| PC prefix | `box_new_` |
| PC kind (`+0x10`) | `0x23`, stamped by `box_new_ctor` |
| PC vtable | `0x0063baa0`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_PhThing` (`structs/physical_object.md`), itself a `C_actor` |
| PC instance size | `0x3cc` (`operator_new(0x3cc)` in `mission_create_actor` case `0x23`) |
| PC constructor | `box_new_ctor` @ `0x0045f160` |
| PC translation unit | `[0x0045f130, 0x0045f930)`: the unit's CRT pair, then the ten `box_new_*` functions and one thunk; the next unit's guard `crt_static_init_guard_0045f930` closes it (that unit is `C_bridge`, `bridge_ctor` at `0x0045f980`) |
| PS2 class | `C_box_new` (tag `9C_box_new`), 10 methods: 10 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_PhThing` base; offsets are PC.

```c
typedef struct C_box_new {                  // 0x3cc bytes; PC offsets
    /* +0x000 */ C_PhThing base;                // [H] the C_PhThing band (structs/physical_object.md)
    /* +0x3b0 */ uint8_t   is_box;              // [H] 1 = S_THING_BOX, 0 = S_THING_WHEEL (§4)
    /* +0x3b4 */ float     wheel_radius;        // [H] 1.0 default; wheel only -> C_PhThing +0x220
    /* +0x3b8 */ float     wheel_width;         // [H] 1.0 default; wheel only -> C_PhThing +0x21c
    /* +0x3bc */ float     mass;                // [H] 10.0 default; derived from the model volume
    /* +0x3c0 */ float     friction;            // [H] 0.5 default; both shapes -> C_PhThing +0x1f8
    /* +0x3c4 */ float     damping;             // [H] 0.01 default; wheel only -> C_PhThing +0x224
    /* +0x3c8 */ uint8_t   touched;             // [M] set by a collision or a hit, cleared by GameInit; no reader (.text scan)
} C_box_new;
```

The six descriptor fields `+0x3b0..+0x3c4` have four independent witnesses that agree on the
order: `box_new_init` seeds them, `box_new_save_data` writes them, `box_new_load_data` reads them
back and `box_new_duplicate_from` copies them. Which of them reach the base depends on `is_box`
(§4), so `wheel_radius`, `wheel_width` and `damping` are dead storage for a box-configured
instance that still round-trips through the definition record.

## 2. Field evidence

Measured by a census of every operand with each displacement in `.text`, attributed to its
containing function.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `box_new_ctor` (through `physical_object_ctor`; kind `+0x10` = `0x23`), `box_new_game_init` (C_PhThing `+0x208` / `+0x20c` / `+0x210` = 0 / -1.0 / 0, which `physical_object_init_box` and `physical_object_init_wheel` then rewrite) | the `C_PhThing` methods of `structs/physical_object.md`; `box_new_save_data` / `box_new_load_data` (C_PhThing `+0x308`, `+0x338`, `+0x30c`, §4) | the inherited band |
| +0x3b0 | is_box | H | `box_new_init` (= 1), `box_new_load_data`, `box_new_duplicate_from` | `box_new_game_init`, `box_new_save_data`, `box_new_duplicate_from` | selects the descriptor (§4) |
| +0x3b4 | wheel_radius | H | `box_new_init` (= 1.0), `box_new_load_data`, `box_new_duplicate_from` | `box_new_game_init` (wheel arm), `box_new_save_data`, `box_new_duplicate_from` | `S_THING_WHEEL +0x00` |
| +0x3b8 | wheel_width | H | `box_new_init` (= 1.0), `box_new_load_data`, `box_new_duplicate_from` | `box_new_game_init` (wheel arm), `box_new_save_data`, `box_new_duplicate_from` | `S_THING_WHEEL +0x04`. `physical_object_move_wheel` offsets the wheel's contact point by half of it along the body's first basis row (C_PhThing `+0xa0`), signed by the side of the contact, which makes it the tyre width |
| +0x3bc | mass | H | `box_new_init` (= 10.0, then volume × 50.0), `box_new_load_data` (clamp), `box_new_duplicate_from` | `box_new_game_init` (both arms), `box_new_save_data`, `box_new_load_data`, `box_new_duplicate_from` | `box_new_init` refreshes the frame's bound volume, takes the volume of the box at `frame+0xb0`, applies the frame scale, and when `0.01 < volume < 1000.0` overwrites the default with `volume × 50.0`. `box_new_load_data` clamps a loaded value to `[0.01, 100000.0]` |
| +0x3c0 | friction | H | `box_new_init` (= 0.5), `box_new_load_data`, `box_new_duplicate_from` | `box_new_game_init` (both arms), `box_new_save_data`, `box_new_duplicate_from` | the tangential factor of both `C_PhThing` contact solvers (`structs/physical_object.md`) |
| +0x3c4 | damping | H | `box_new_init` (= 0.01), `box_new_load_data`, `box_new_duplicate_from` | `box_new_game_init` (wheel arm), `box_new_save_data`, `box_new_duplicate_from` | `S_THING_WHEEL +0x1c`; `physical_object_move_wheel` divides it by the radius |
| +0x3c8 | touched | M | `box_new_game_init` (= 0, both arms), `box_new_on_collision` (= 1), `box_new_hit` (= 1) | - | no reader (`.text` scan): every other `+0x3c8` operand in `.text` is a `C_vehicle` access or a `crash_obj_ctor` store (§7) |

## 3. Method family

Coverage: all **10** PS2 `9C_box_new` methods are in the tables below, **10 CONFIRMED twins**.
The scalar-deleting destructor has no PS2 twin; the CRT pair of §3a belongs to the translation
unit. Slots 13–15, 22–24, 27 and 38 are inherited from `C_PhThing`; the other slots are the
`C_actor` defaults (`structs/actor.md` §3c).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0045f130 | `crt_static_init_guard_0045f130` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x0045f150 | `crt_atexit_register_0045f150` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0045f160 | `box_new_ctor` | instance | `__ct__9C_box_newFv` | 0x00124c50 | CONFIRMED | `physical_object_ctor`, vtable `0x0063baa0`, kind `0x23` |
| 0x0045f180 | `box_new_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `box_new_dtor`, vtable slot 16 ¹ |
| 0x0045f1a0 | `box_new_dtor` | instance | `__dt__9C_box_newFv` | 0x00124c90 | CONFIRMED | restores the vtable and tail-calls the `C_PhThing` destructor; the class owns no heap state |

¹ No standalone PS2 twin: GCC carries the free flag inside `__dt__9C_box_newFv`.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0045f1b0 | `box_new_init` | `Init__9C_box_newFP9I3D_frame` | 0x00124cf0 | CONFIRMED | slot 18; `physical_object_init`, then the six defaults and the volume-derived mass (§2) |
| 0x0045f2c0 | `box_new_game_init` | `GameInit__9C_box_newFv` | 0x00124e70 | CONFIRMED | slot 25; builds `S_THING_BOX` or `S_THING_WHEEL` on the stack, calls `physical_object_init_box` / `physical_object_init_wheel`, then `physical_object_game_init`, then clears `touched` (§4) |
| 0x0045f390 | `thunk_physical_object_game_done` | `GameDone__9C_box_newFv` | 0x00124f40 | CONFIRMED | slot 26; a 5-byte `jmp` to `physical_object_game_done`, as the 8-byte PS2 body is a tail-jump to `GameDone__9C_PhThingFv` ¹ |
| 0x0045f3a0 | `box_new_load_data` | `LoadData__9C_box_newFP7C_chunk` | 0x00124f50 | CONFIRMED | slot 20; reads the definition record (§4) |
| 0x0045f720 | `box_new_save_data` | `SaveData__9C_box_newFP7C_chunk` | 0x00125a60 | CONFIRMED | slot 19; writes version byte 2, the six fields, then C_PhThing `+0x308`, `+0x338`, `+0x30c` |
| 0x0045f890 | `box_new_duplicate_from` | `DuplicateFrom__9C_box_newFP7C_actor` | 0x00125c30 | CONFIRMED | slot 21; the base call, then the six fields in order ² |
| 0x0045f8f0 | `box_new_on_collision` | `Collision__9C_box_newFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00125c90 | CONFIRMED | slot 30; `touched = 1`, then `physical_object_on_collision` |
| 0x0045f910 | `box_new_hit` | `Hit__9C_box_newF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00125ca0 | CONFIRMED | slot 31; `touched = 1`, then `physical_object_hit` |

¹ Ghidra names a thunk after its target, so the function's Ghidra id is `thunk_FUN_004bf0c0`,
not a `FUN_0045f390`.
² Slot 21 is `DuplicateFrom` for the whole `C_actor` family. In a class that does not override it
the slot holds a folded no-op: `C_bottle` has `thunk_stub_noop_args4_00461180` (`0x004773f0`), a
5-byte `jmp` thunk; `C_box` has `stub_noop_args4_004776f0`, a real body. The family slot map is in
`structs/actor_track.md`.

### 3d. Free helpers

The translation unit holds no free helper.

## 4. The two descriptors and the definition record

`box_new_game_init` builds one of two descriptor structs on the stack and hands it to the base:

| `is_box` | descriptor | fields passed |
|---|---|---|
| set | `S_THING_BOX` | `mass`, `friction` |
| clear | `S_THING_WHEEL` | `wheel_radius`, `wheel_width`, `mass`, `friction`, `damping` |

Their PC layouts, read from the stack stores in `box_new_game_init` and the loads in
`physical_object_init_box` / `physical_object_init_wheel`:

| Struct | Offset | Field | Source | Destination in `C_PhThing` |
|---|---|---|---|---|
| `S_THING_BOX` | `+0x00` | mass | `+0x3bc` | `+0x190` |
| `S_THING_BOX` | `+0x04` | initial velocity (vec3) | 0 | `+0x1e0` |
| `S_THING_BOX` | `+0x10` | friction | `+0x3c0` | `+0x1f8` |
| `S_THING_WHEEL` | `+0x00` | wheel radius | `+0x3b4` | `+0x220` |
| `S_THING_WHEEL` | `+0x04` | wheel width | `+0x3b8` | `+0x21c` |
| `S_THING_WHEEL` | `+0x08` | mass | `+0x3bc` | `+0x190` |
| `S_THING_WHEEL` | `+0x0c` | initial velocity (vec3) | 0 | `+0x1e0` |
| `S_THING_WHEEL` | `+0x18` | friction | `+0x3c0` | `+0x1f8` |
| `S_THING_WHEEL` | `+0x1c` | damping | `+0x3c4` | `+0x224` |

A velocity whose square is below `1e-05` puts the body at rest (act state 1); `box_new_game_init`
always passes zero, so every `C_box_new` starts at rest.

**The definition record** is what `LoadData` / `SaveData` carry over a `C_chunk` (a scene's
`scene2.bin` actor definition, chunk `0xAE24`). `box_new_load_data` reads a version byte, then:

| version | fields |
|---|---|
| 0 | the six fields, each read only while the record still holds enough bytes |
| 1 | the six fields, then C_PhThing `+0x308` (the impact-sound column) |
| 2 | the six fields, then C_PhThing `+0x308`, `+0x338` (impact sound may restart) and `+0x30c` |

`box_new_save_data` writes version 2.

## 5. PC vs PS2

### 5a. The kind map: `C_box_new` is `0x23`, kind 9 is `C_box`

Two PC observations point at kind 9, and both fit `C_box`, a class the PS2 does not have: link
order puts kind 9's vtable immediately after `C_bottle`'s, where `C_box_new` sits on PS2; and
`recorder_new_track` labels kind 9's recorder tracks `box%d`.

The PS2 constructors settle it: **`__ct__9C_box_newFv` writes `0x23`** to `+0x50`. Extracting that
write from every PS2 `__ct__*` gives the kind of all 33 PS2 classes that write it, and the table
cross-validates against the PC on **twelve** documented classes (actor=1, player=2, human=3,
car=4, don_ocas=0x13, bridge=0x14, airplane=0x16, DropOut=0x11, crash_obj=0x1a, entity=0x1b,
actor_shot=0x21, bottle=0xa). It also gives **kind `0x20` = `C_public_physics`** and **kind
`0x25` = `C_drop_in_weapon`**. `recorder_new_track` only knows the kinds the PC has.

The hierarchy is the same on both builds: `box_new_ctor` chains to `physical_object_ctor` exactly
as `__ct__9C_box_newFv` chains to `__ct__9C_PhThingFv`.

### 5b. Body-diff evidence, per method

| Method | The falsifiable evidence |
|---|---|
| `box_new_init` | **five shared magic constants in order** — `1.0 / 1.0 / 10.0 / 0.5 / 0.01` at PC `+0x3b4..+0x3c4` and PS2 `+0x644..+0x654` — plus the identical `0.01 / 1000.0 / ×50.0` volume ladder |
| `box_new_game_init` | the same preamble, two-arm branch, argument sets, base call and trailing flag clear; the PS2 test is **inverted** (`if (+0x640 == 0) InitWheel else InitBox`) |
| `box_new_save_data` / `box_new_load_data` | the same six fields in the same order; the PS2 writes version byte 2 too |
| `box_new_duplicate_from` | the same six fields, **96 bytes on both builds** |
| `box_new_on_collision` / `box_new_hit` | `*(this+0x3c8) = 1`, then the base handler; the PS2 is `*(this+0x658) = 1`, then the same. A search of the PS2 decomp for `+ 0x658)` finds these two writers and the `GameInit` clear and no reader |
| `box_new_ctor` / `box_new_dtor` | base call + vtable + kind, statement for statement |
| `thunk_physical_object_game_done` | an 8-byte PS2 tail-jump against a 5-byte PC `jmp`, both to the `C_PhThing` `GameDone` |

### 5c. The field-band mapping this class pins

| PC | PS2 | Field |
|---|---|---|
| `+0x3b0` | `+0x640` | `is_box` |
| `+0x3b4` | `+0x644` | `wheel_radius` |
| `+0x3b8` | `+0x648` | `wheel_width` |
| `+0x3bc` | `+0x64c` | `mass` |
| `+0x3c0` | `+0x650` | `friction` |
| `+0x3c4` | `+0x654` | `damping` |
| `+0x3c8` | `+0x658` | `touched` |

A constant `+0x290` delta across the band — but **do not rebase other fields from it**
(`structs/actor.md` §8: the PC↔PS2 deltas in this family are not constant, and two measured pairs
are negative).

### 5d. `C_PhThing` twins located through this class

`box_new_game_init`'s inverted branch locates two `C_PhThing` entry points, its tail a third, and
the collision / hit pair the last two. All five are body-diffed on both sides and CONFIRMED:

| Located by | PC name | PS2 twin | What the diff turns on |
|---|---|---|---|
| the tail of `box_new_game_init` | `physical_object_game_init` | `GameInit__9C_PhThingFv` | the `1e-08` scale ladder, the `0x40` written into the freshly zeroed dyn-coll object |
| the box arm of that branch | `physical_object_init_box` | `InitBox__9C_PhThingFP11S_THING_BOX` | extents `+0.05`, the six-comparison `2.0 / 0.5` aspect gate, `1/(m·L²/3)` |
| the wheel arm (the PS2 test is inverted) | `physical_object_init_wheel` | `InitWheel__9C_PhThingFP13S_THING_WHEEL` | `0x3cf5c28f`, `0x3f866666`, and the PS2's `/(0.03·0.9)` folded by MSVC into `×37.03704` |
| the tail of `box_new_on_collision` | `physical_object_on_collision` | `Collision__9C_PhThingF…` | impulse `×1.1`, the deepest-hit record on `+0x30c ↔ +0x5e4` |
| the tail of `box_new_hit` | `physical_object_hit` | `Hit__9C_PhThingF10E_hit_type…` | the identical `1e-08` normalise-with-degenerate-fallback tree, offsets `0.025 / 0.06 / 0.04` |

A statement-position match inside a confirmed twin is a lead, not a twin: the two
`CollisionCount` specialisations do **not** resolve to a 1:1 twin that way
(`structs/physical_object.md` §7c).

### 5e. Descriptor field order

The PS2 puts the initial velocity (a 16-byte `S_vector`) FIRST in both descriptors:
`S_THING_BOX` = {velocity, mass `+0x10`, friction `+0x14`}, `S_THING_WHEEL` = {velocity, radius
`+0x10`, width `+0x14`, mass `+0x18`, friction `+0x1c`, damping `+0x20`}. The PC puts the scalars
that precede friction first and the 12-byte velocity after them (§4). Both builds' `GameInit` fill
the velocity with zeros, and both builds' `InitBox` / `InitWheel` read each field from their own
offset, so the difference is a declaration order, not a behaviour.

## 6. Related

- `structs/physical_object.md` — `C_PhThing`, where all the dynamics live.
- `structs/box.md` — `C_box`, the PC-only class this one replaced (kind 9).
- `structs/entity_kind.md` — the kind enum (`0x23`, `0x20`, `0x25`).
- `structs/dropout.md` — `S_DROPOUT_BOX`, which extends `S_THING_BOX` (§4).

## 7. Open items

1. **`touched` (`+0x3c8`) has no reader** on either build (§2, §5b). What the flag was meant to
   gate is not recoverable from the binary. Searched: every `.text` operand with this displacement,
   by whether its base register holds the function's receiver, and the functions testing kind
   `0x23` or calling a `box_new_` method. That search cannot see a pointer used with no kind test.
