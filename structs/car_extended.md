# `C_car_ex` — the EXTENDED car (PC prefix `car_extended_`)

**Created** 2026-08-12 15:55 +0200 · **Modified** 2026-10-01 08:56 +0200

| | |
|---|---|
| PC prefix | **`car_extended_`** — `ex` = **extended** |
| PC kind (`entity+0x10`) | **`4`** (unchanged from `C_car`); flavour `+0x14 = 3` |
| PC vtable | `.rdata 0x0063bd68` (41 slots, §4) |
| PC instance size | `0x2254` (`mission_create_actor` `case 0x24`) |
| PC translation unit | `[0x00474cc0, 0x00476f70]` — 20 real starts, 0 unnamed, 0 phantom starts |
| Base class | **`C_car`** (`structs/car.md`) |
| PS2 class | `C_car_ex`, 11 methods, all mapped (§3) |

The class is a `C_car` subclass that adds *model-driven* extras; it is not about exhausts (that is
one member type) and not about trucks (three of its shipped users are not trucks).

## 1. What the class is

A `C_car` whose **model** carries extra `~`-tagged dummy frames. `car_extended_game_init` walks the
frame tree once (`car_extended_scan_frame_tags`) and turns those tags into runtime state;
`car_extended_update` animates it; `car_extended_game_done` restores every touched frame to its rest
pose. Two independent capabilities come out of that:

1. **Animated body parts** — a *shake* family (`~C`) and an *exhaust* family (`~E`).
2. **A ride-on dynamic-collision body** (`~DCC` boxes + a `~DCS` spot) that `C_human`s register on,
   are pushed by, are thrown off, and burn on.

Runtime identity: the entity **kind stays 4** (it *is* a car for every kind test); the discriminator
is the C_actor-level flavour field **`+0x14 = 3`**. The entity factory builds it at
**`case 0x24: operator_new(0x2254)`**.

### 1a. The `~` frame-tag grammar — the whole class in one table

Recovered from `car_extended_scan_frame_tags`' body and **round-tripped against the shipped models**
in `exe-re/game-sources/models/`: every tag the code can parse exists in the data, and every tag in
the data parses.

| tag | parsed as | shipped users |
|---|---|---|
| `~E~` | an **exhaust part**: `{frame, random 0..2π phase, rest quat}` appended to the `+0x2204` vector | `traktor.4ds`, `valec.4ds` — the two vehicles with a vertical exhaust stack |
| `~C` `[R]` `<X\|Y\|Z>` `<value>` `~` | a **shake part** ¹ | `traktor.4ds` (`~CRX0.2~`, `~CRX0.35~`, `~CY3~`, `~CZ2~`), `fordtTud00K.4ds` (`~CX3~`, `~CX5~`) |
| `~DCS~` | the **cargo-bed rider spot**: sets flag `+0x2214`, position `+0x2218`, direction `+0x2224`, where `human_intern_use_car_truck` places a boarding rider | the 5 truck models, on a frame named **`misto`** (Czech: *place*) |
| `~DCC~` | a **dynamic-collision box**, only on frames of type 6 ² | the 5 truck models, on `Dummy16`…`Dummy20` and **`kolize pla`** (*collision tarp*) |

¹ `R` ⇒ rotate `<value>` **degrees** (`crt_atof` × `0.01745`); no `R` ⇒ translate `<value>`
**millimetres** (`crt_atof` × `0.001`), with a ±0.4 random jitter on the other two axes. The
trailing `~` is found with `strchr` and NUL'd.
² Allocates a 0x48-byte record of dynamic-collision type `0x40` (§2d), fills its box corners from
the frame AABB × frame scale, builds transform + inverse, appends it to `+0x2238` and hands it to
`collision_dyn_take_out_to_list`.

**Complete shipped user set:** `traktor`, `valec`, `fordtTud00K`, `TruckBxx00`, `truckmorph`,
`truckmorph00`, `trucksplachtou`, `FMVanimTruckB00`.

The four tags do not cross-match: `~DCS~`/`~DCC~` contain no `~C` or `~E` pair, so the probe order
(`~E` → `~C` → `~DCS` → `~DCC`) is not load-bearing. Shipped `~C` amplitudes are **0.2°** and
**3 mm** — idle vibration, not a door or a hood.

## 2. Field layout (PC)

`sizeof == 0x2254`; the base `C_car` ends at `0x21f4`.

```c
typedef struct C_car_ex /* extends C_car */ {
    /* +0x0000 */ C_car        base;          // [H] kind +0x10 stays 4; flavour +0x14 = 3
    /* +0x21f4 */ vec_alloc    _a0;           // [H] allocator byte (MSVC empty-base padding)
    /* +0x21f8 */ S_clap      *shake_begin;   // [H] `~C` parts, stride 0x38
    /* +0x21fc */ S_clap      *shake_end;     // [H]
    /* +0x2200 */ S_clap      *shake_cap;     // [H]
    /* +0x2204 */ vec_alloc    _a1;           // [H]
    /* +0x2208 */ S_exh_top   *exhaust_begin; // [H] `~E` parts, stride 0x18
    /* +0x220c */ S_exh_top   *exhaust_end;   // [H]
    /* +0x2210 */ S_exh_top   *exhaust_cap;   // [H]
    /* +0x2214 */ uint8_t      dcs_present;   // [H] a `~DCS~` frame was found
    /* +0x2218 */ S_vector     dcs_position;  // [H] the cargo-bed rider spot, car-local
    /* +0x2224 */ S_vector     dcs_direction; // [H] the rider's facing there
    /* +0x2230 */ void        *dyn_coll_list; // [H] the world dynamic-collision list
    /* +0x2234 */ vec_alloc    _a2;           // [H]
    /* +0x2238 */ S_dyn_coll **dyn_begin;     // [H] the class's own records, stride 0x10
    /* +0x223c */ S_dyn_coll **dyn_end;       // [H]
    /* +0x2240 */ S_dyn_coll **dyn_cap;       // [H]
    /* +0x2244 */ vec_alloc    _a3;           // [H]
    /* +0x2248 */ C_human    **riders_begin;  // [H] carried persons (outside any seat)
    /* +0x224c */ C_human    **riders_end;    // [H]
    /* +0x2250 */ C_human    **riders_cap;    // [H]
} C_car_ex;                                   // 0x2254
```

`S_clap`, `S_exh_top` and `S_dyn_coll` are **PS2 type names that type a PC structure**: the PC
decomp gives the stride; only the PS2 signature says what the elements are.

### 2a. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x14 | flavour = 3 | H | `car_extended_ctor` | `human_game_done` | gated as `kind == 4 && +0x14 == 3`; C_actor-level enum: C_actor = 1, C_human = 2, C_car_ex = 3 |
| +0x21f8 | shake-part vector | H | `car_extended_ctor`, `car_extended_scan_frame_tags`, `car_extended_shake_part_vector_insert_n` | `car_extended_update`, `car_extended_game_done`, `car_extended_shake_part_vector_erase` | stride 0x38 |
| +0x2208 | exhaust-part vector | H | `car_extended_ctor`, `car_extended_scan_frame_tags`, `stl_insert_n_stride_0x18` | `car_extended_update`, `car_extended_game_done`, `car_extended_exhaust_part_vector_copy` | stride 0x18 |
| +0x2214 | `~DCS` present | H | `car_extended_scan_frame_tags`, `car_extended_game_init` | `car_extended_can_board` | set to 1 on the first `~DCS~` frame ¹ |
| +0x2218 | rider-spot position | H | `car_extended_scan_frame_tags` | `human_intern_use_car_truck` | copied into the boarding human's position `+0x24` before the human is parented to the car frame |
| +0x2224 | rider-spot direction | H | `car_extended_scan_frame_tags` | `human_intern_use_car_truck` | copied into the boarding human's direction `+0x30` |
| +0x2230 | dynamic-collision list | H | `car_extended_scan_frame_tags`, `car_extended_game_init`, `car_extended_game_done` | `car_extended_update`, `human_game_done` | `human_game_done` returns a departing rider's own records to this list |
| +0x2238 | dyn-coll record vector | H | `car_extended_scan_frame_tags` | `car_extended_game_done` | stride 0x10; records are 0x48 bytes |
| +0x2248 | carried-person vector | H | `car_extended_add_carried_person`, `car_extended_remove_carried_person` | `car_extended_update` | `C_human*`; riders outside any seat (`+0xac == -1`) |

¹ `car_extended_can_board` returns `+0x2214 && +0x2230` and is the gate the `human_entertotruck`
script command passes before calling `human_intern_use_car_truck` (PS2 side in §5).

### 2b. The shake-part record (stride 0x38)

| Offset | Field | Notes |
|---|---|---|
| +0x00 | `I3D_frame *frame` | the tagged frame |
| +0x04 | `uint8_t mode` | 0 = translate, 1 = rotate |
| +0x08 | `S_vector axis` | from the `X`/`Y`/`Z` character, ±0.4 jitter on the other two in translate mode |
| +0x14 | `float phase` | advanced by `(0.04 + 0.02·r)·dt`, r the engine-rev fraction (§2c), wrapped at 2π |
| +0x18 | `float amplitude` | degrees×0.01745 or millimetres×0.001 |
| +0x1c | `S_vector rest_pos` | the frame's pose at scan time |
| +0x28 | `S_quat rest_rot` | the frame's rotation at scan time |

Applied each frame as `offset = sin(phase) · (1 − 0.5·r) · amplitude · axis + rest_pos` (mode 0) or
as a quaternion about `axis` composed with `rest_rot` (mode 1).

The **exhaust-part record** (stride 0x18) is `{frame*, phase, rest quat}`; its phase advances by
`dt·0.006` and it rotates about local Z, with the swing scaled by `(1 − r)·π/2`.

### 2c. The engine-rev fraction r

`car_extended_update` drives both families from the engine state of the base car
(`structs/car.md` §1, `structs/vehicle.md` §1):

| Input | Field | Role |
|---|---|---|
| engine running | `car+0xd18` = `C_Vehicle+0xca8` | clear: only the exhaust parts are posed; set: both families animate |
| engine revs per second | `car+0x634` = `C_Vehicle+0x5c4` | the live value |
| idle revs per second | `car+0x6d8` = `C_Vehicle+0x668` | CARINIT `idling rpm` ÷ 60 |
| maximum revs per second | `car+0x1f88` = `C_Vehicle+0x1f18` | CARINIT `maximum rpm` ÷ 60 |

`r = (revs − idle) / (max − idle)`, clamped to 0..1 — the same normalisation
`vehicle_update_strips` applies. As the engine revs up, the shake speeds up and its amplitude
falls to half, and the exhaust swing shrinks to nothing at maximum revs.

### 2d. The `~DCC` record (0x48 bytes)

Built by `car_extended_scan_frame_tags`; every one of its 18 dwords is written there (the
allocation is zero-filled first). Word 0 is the dynamic-collision type, and `0x40` is the
transformed-box type of `structs/collision.md` §1d.

| Offset | Field | Source |
|---|---|---|
| +0x00 | type `0x40` | literal |
| +0x04 | the tagged `I3D_frame` | the scanned frame |
| +0x08 | query-exclusion mask, 0 | `collision_line_in_dynamics` skips an object when this word shares a bit with the query's `+0x3c` |
| +0x0c | grid-bucket back-link, 0 | set by `collision_dyn_insert` (`structs/collision.md` §1) |
| +0x10 | centre | the frame's local position `frame+0x80` |
| +0x1c | bounding radius | the length of the larger scaled box corner |
| +0x20 | zero fill | - |
| +0x24 / +0x30 | box min / max corner | the frame bounding box `frame+0x140` × the frame scale `frame+0x90` |
| +0x3c | pointer to the 0x40-byte transform | the frame rotation plus its position |
| +0x40 | pointer to the 0x40-byte inverse transform | `S_matrix::Inverse` of a copy |
| +0x44 | owner | the `C_car_ex` |

## 3. Method coverage — PC first

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role / notes |
|---|---|---|---|---|---|
| `0x00474cf0` | `car_extended_ctor` | `__ct__8C_car_exFv` | `0x00143a90` | CONFIRMED | base ctor, then the four owned vectors + the flavour field `+0x14 = 3`; 148 vs 160 B |
| `0x00474d90` | `car_extended_scalar_deleting_dtor` | - | - | PC-only | vtable slot 16; MSVC-only construct, no PS2 counterpart by construction |
| `0x00474db0` | `car_extended_dtor` | `__dt__8C_car_exFv` | `0x00143b40` | CONFIRMED | frees the four owned vectors in exact reverse ctor order, then tail-calls `car_dtor`; 151 vs 252 B |
| `0x00474e50` | `car_extended_init` | `Init__8C_car_exFP9I3D_frame` | `0x00143c40` | CONFIRMED | slot 18; one-line bool-narrowing forwarder to `car_init` on both builds |
| `0x00474e70` | `car_extended_game_init` | `GameInit__8C_car_exFv` | `0x00143c60` | CONFIRMED | slot 25; empties the four vectors + the `~DCS` state, then scans the model frame tree; 236 vs 2500 B (§5) |
| `0x00474f60` | `car_extended_scan_frame_tags` | - | - | PC-only | recursive `~E` / `~C` / `~DCS` / `~DCC` frame-tag scanner ¹ |
| `0x004758e0` | `car_extended_game_done` | `GameDone__8C_car_exFv` | `0x00146300` | CONFIRMED | slot 26; restores every part to its rest pose, releases the dyn-coll records, then `car_game_done` ² |
| `0x00475bc0` | `car_extended_load_data_chunk` | `LoadData__8C_car_exFP7C_chunk` | `0x00146650` | CONFIRMED | slot 20; one-line bool forwarder; the class serialises nothing of its own |
| `0x00475be0` | `car_extended_save_data_chunk` | `SaveData__8C_car_exFP7C_chunk` | `0x00146670` | CONFIRMED | slot 19; one-line bool forwarder |
| `0x00475c00` | `thunk_car_copy_carinit_from_actor` | `DuplicateFrom__8C_car_exFP7C_actor` | `0x00146690` | CONFIRMED | slot 21; a pure tail-jump to the base on both builds (PC 5 B `jmp`, PS2 8 B `j` + delay slot) |
| `0x00475c10` | `car_extended_add_carried_person` | `DCAddPerson__8C_car_exFP7C_human` | `0x001466a0` | CONFIRMED | null guard, BACKWARD duplicate scan, append; `DC` = dynamic collision: riders on the body, not in a seat |
| `0x00475dd0` | `car_extended_remove_carried_person` | `DCDelPerson__8C_car_exFP7C_human` | `0x00146730` | CONFIRMED | backward scan then swap-with-last + decrement, the same algorithm; 111 vs 88 B |
| `0x00475e40` | `car_extended_update` | `Update__8C_car_exFUi` | `0x00146790` | CONFIRMED | slot 14; five blocks ³; 3440 vs 2980 B |
| `0x00476bb0` | `car_extended_shake_part_vector_erase` | - | - | PC-only | 1 callsite, receiver `+0x21f4`, stride 0x38; PS2 clears the same vector inline in `GameDone` |
| `0x00476c00` | `car_extended_shake_part_vector_insert_n` | - | - | PC-only | 1 callsite, receiver `+0x21f4`, stride 0x38; N->1 fold of PS2 template helpers (§5) |
| `0x00476f20` | `car_extended_exhaust_part_vector_copy` | - | - | PC-only | 1 callsite, `+0x2204`, stride 0x18; not the generic `stl_copy_back_stride_0x18` ⁴ |

¹ Its PS2 counterpart is the helper-class symbol
`cbEnumProp__Q28C_car_ex22S_hlp$2074C_car_ex_cppFP9I3D_frameP8C_car_ex`, which is not in the
`C_car_ex` method list. The grammar round-trips against the shipped `.4ds` models (§1a).
² Shared flag magic `& 0xfffffedf | 0x40000000` and `& 0xfffffecb | 0x40000008`; 734 vs 848 B.
³ Burnt-corpse swap + eject (`ohorelec01.i3d`), base `car_update`, per-rider vtable push, tilt eject
at 0.785 rad, then the shake/exhaust animation.
⁴ `0x00474c10`, 10 callsites / 7 owners — same stride, opposite direction.

## 4. The vtable — `0x0063bd68`, 41 slots

Identical width and slot order to `C_car`'s `0x0063bc08` — slot-for-slot aligned, no shift.
**Exactly 8 slots differ**, and they are exactly the 8 methods `C_car_ex` overrides:

| slot | `C_car` | `C_car_ex` |
|---|---|---|
| 14 (`+0x38`) | `car_update` | `car_extended_update` |
| 16 | `car_scalar_deleting_dtor` | `car_extended_scalar_deleting_dtor` |
| 18 | `car_init` | `car_extended_init` |
| 19 | `car_save_data_chunk` | `car_extended_save_data_chunk` |
| 20 | `car_load_data_chunk` | `car_extended_load_data_chunk` |
| 21 | `car_copy_carinit_from_actor` | `thunk_car_copy_carinit_from_actor` |
| 25 | `car_game_init` | `car_extended_game_init` |
| 26 | `car_game_done` | `car_extended_game_done` |

The second vtable at `+0x70` (`0x0063bd64`) is the `C_car` / `C_Vehicle` multiple-inheritance pair's
second base.

## 5. PC vs PS2

**Class identity.** PS2 mangled tag `8C_car_ex`, source file `C_car_ex.cpp` (a PS2 `__sinit_`
symbol). PS2 vtable `__vt__8C_car_ex`; second base pointer at `+0x22d4`. Alphabetical link order
places it between `C_car` and `C_clock`; aligning that order with the PC's `.rdata` vtable sequence
gives the PC vtable `0x0063bd68`.

**Field bridge.**

| PC | PS2 | delta / note |
|---|---|---|
| `+0x14` | `+0x54` | the flavour field; one step off the proven kind bridge `+0x10` ↔ `+0x50` |
| `+0x21f4` | `+0x24d0` | shake-part vector; stride 0x38 vs 0x40 ¹ |
| `+0x2204` | `+0x24dc` | exhaust-part vector; stride 0x18 vs 0x20 |
| `+0x2214` / `+0x2218` / `+0x2224` | `+0x24b0` / `+0x2490` / `+0x24a0` | `~DCS` flag / position / direction |
| `+0x2230` | `+0x24b4` | the dynamic-collision list |
| `+0x2234` | `+0x24b8` | `S_dyn_coll` vector |
| `+0x2244` | `+0x24c4` | carried-person vector (PS2 keeps count at `+0x24c8`, data at `+0x24cc`) |

¹ The +8 is exactly the two `S_vector` members, 12 B on PC vs 16 B on PS2, so the strides
reconcile.

The PS2 `human_entertotruck` arm tests `car+0x24b0 && car+0x24b4` — the pair that maps to PC
`+0x2214` / `+0x2230`.

**The two builds declare these members in a different order** — PS2 is
`[spot pos, dir, flag, list, dyn_coll, persons, shake, exhaust]`, PC is
`[shake, exhaust, flag, pos, dir, list, dyn_coll, persons]`. No twin in §3 depends on it (all rest
on shared constants and vtable slots).

**Factoring differences.**

* `GameInit__8C_car_exFv` is 2500 B against the PC's 236 B because **PS2 inlines one copy of the tag
  scan into it**; the PC calls out to `car_extended_scan_frame_tags`.
* `cbEnumProp` is 5200 B against the PC's 2305 B because it carries the scan body **twice** (self
  plus an inlined child pass). Per copy the ratio is ≈0.96, not the raw 0.44 — a size ratio outside
  the 1.0–1.6 band that means *factoring*.
* The container helpers differ by template, not by behaviour: PS2 uses `gif::vector<S_clap,8>` and
  `std::__vector_imp<S_dyn_coll>`, the PC uses MSVC `std::vector`, so the PC helper rows in §3 have
  no body agreement to assert. `car_extended_shake_part_vector_insert_n` (795 B) folds PS2's
  `push_back<S_clap>` (748 B) + `reserve` (308 B).
* PS2 calls newlib **`atof`** where the PC calls `crt_atof` (`0x00624ebb`); MSVC does not inline it,
  so the callee survives on both sides, and that statement alignment identifies `crt_atof`.

**PS2-side symbols with no separate PC function:**
`__ct__Q28C_car_ex6S_clapFRCQ28C_car_ex6S_clap` (the shake-record copy ctor), both
`reserve__Q23gif...` and both `push_back__Q23gif...` instantiations, and the
`__vector_deleter<S_dyn_coll>` triple — all folded into the PC's five STL helpers or emitted inline.
