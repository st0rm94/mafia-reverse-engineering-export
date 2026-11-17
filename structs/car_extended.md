# `C_car_ex` — the extended car: model-driven body parts and a ride-on collision body

**Created** 2026-08-12 15:55 +0200 · **Modified** 2026-11-04 16:47 +0100

A `C_car` subclass whose **model** carries extra `~`-tagged dummy frames. `car_extended_game_init`
walks the frame tree once (`car_extended_scan_frame_tags`) and turns those tags into runtime state;
`car_extended_update` animates it; `car_extended_game_done` restores every touched frame to its rest
pose. Two capabilities come out of that: **animated body parts** (a *shake* family `~C` and an
*exhaust* family `~E`), and a **ride-on dynamic-collision body** (`~DCC` boxes and a `~DCS` spot)
that `C_human`s board, are pushed by, are thrown off and burn on. `ex` = **extended**: the class is
not about exhausts (one member type) and not about trucks (three of its shipped users are not
trucks).

| | |
|---|---|
| PC prefix | `car_extended_` |
| PC kind (`+0x10`) | `4`, inherited from `C_car`: `car_extended_ctor` writes no kind. The class is told apart by `class_id` (`+0x14`) = 3 |
| PC vtable | `g_car_extended_vt` @ `0x0063bd68`, 41 slots (§6); second base `g_car_extended_vt_vehicle` @ `0x0063bd64` at `+0x70` |
| PC base class | `C_car` (`structs/car.md`) |
| PC instance size | `0x2254` (`operator_new(0x2254)` in `mission_create_actor` `case 0x24`) |
| PC constructor | `car_extended_ctor` @ `0x00474cf0` |
| PC translation unit | `[0x00474cc0, 0x00476f50)`: the unit's CRT pair, then the 16 `car_extended_*` functions of the range and one thunk; the next unit's guard `crt_static_init_guard_00476f50` opens `C_cartridge`. `car_extended_can_board` (`0x005c8c80`) is emitted outside the range, next to the script dispatcher |
| PS2 class | `C_car_ex` (tag `8C_car_ex`), 11 methods: 11 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_car` base (which ends at `0x21f4`); offsets are PC.

```c
typedef struct C_car_ex {                          // 0x2254 bytes; PC offsets
    /* +0x0000 */ C_car       base;                // [H] the C_car band; class_id (+0x14) = 3
    /* +0x21f4 */ uint8_t     shake_alloc;         // [H] the shake vector's allocator byte
    /* +0x21f8 */ S_clap     *shake_begin;         // [H] `~C` parts, element 0x38 bytes (§5a)
    /* +0x21fc */ S_clap     *shake_end;           // [H] one past the last shake part
    /* +0x2200 */ S_clap     *shake_cap;           // [H] end of the shake vector's storage
    /* +0x2204 */ uint8_t     exhaust_alloc;       // [H] the exhaust vector's allocator byte
    /* +0x2208 */ S_exh_top  *exhaust_begin;       // [H] `~E` parts, element 0x18 bytes (§5a)
    /* +0x220c */ S_exh_top  *exhaust_end;         // [H] one past the last exhaust part
    /* +0x2210 */ S_exh_top  *exhaust_cap;         // [H] end of the exhaust vector's storage
    /* +0x2214 */ uint8_t     dcs_present;         // [H] a `~DCS~` frame was found
    /* +0x2218 */ S_vector    dcs_position;        // [H] the cargo-bed rider spot, car-local
    /* +0x2224 */ S_vector    dcs_direction;       // [H] the rider's facing there
    /* +0x2230 */ void       *taken_out_list;      // [H] heap vector of objects lifted out of the grid
    /* +0x2234 */ uint8_t     dyn_alloc;           // [H] the dyn-coll vector's allocator byte
    /* +0x2238 */ S_dyn_coll *dyn_begin;           // [H] the `~DCC` boxes, element 0x10 bytes (§5c)
    /* +0x223c */ S_dyn_coll *dyn_end;             // [H] one past the last `~DCC` box
    /* +0x2240 */ S_dyn_coll *dyn_cap;             // [H] end of the dyn-coll vector's storage
    /* +0x2244 */ uint8_t     riders_alloc;        // [H] the rider vector's allocator byte
    /* +0x2248 */ C_human   **riders_begin;        // [H] persons carried outside any seat
    /* +0x224c */ C_human   **riders_end;          // [H] one past the last rider
    /* +0x2250 */ C_human   **riders_cap;          // [H] end of the rider vector's storage
} C_car_ex;
```

The band is **four MSVC `std::vector`s** and the `~DCS` spot. Each vector is a 16-byte object: an
allocator byte (the empty allocator still occupies a byte, padded to a dword), then the begin / end
/ capacity pointers. `car_extended_ctor` writes the four allocator bytes and zeroes the twelve
pointers; the class's helpers receive the vector by the address of its allocator byte. The
vectors grow on demand: one element per tagged frame (shake, exhaust, `~DCC`) or per boarded
person (riders).

`S_clap`, `S_exh_top`, `S_dyn_coll` are **PS2 type names that type a PC structure**: the PC code
gives the element sizes, and only the PS2 signatures say what the elements are. `taken_out_list`
points at a heap `std::vector<tDynamicCollObject *>` (`structs/collision.md`):
`car_extended_scan_frame_tags` allocates it (`operator_new(0x10)`) on the first `~DCC~` frame and
`car_extended_game_done` frees it.

## 2. Field evidence

Measured by a census of every operand with each displacement in `.text` and a census of the
decomp's `this+N` accesses, both attributed to the containing function. A helper that receives a
vector by its address writes and reads that vector's pointers through it; such a helper is named
with the address it is given.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0000 | base | H | `car_extended_ctor` (through `car_ctor`; `class_id` `+0x14` = 3, both vtables) | the `C_car` methods of `structs/car.md`; `actor_get_class_id`, `human_game_done` (`+0x14`) | `human_game_done` identifies the class as `kind == 4 && +0x14 == 3` (`structs/actor.md` §1) |
| +0x21f4 | shake_alloc | H | `car_extended_ctor` | - | the vector's address goes to `car_extended_shake_part_vector_insert_n` (`car_extended_scan_frame_tags`) and to `car_extended_game_done` |
| +0x21f8 | shake_begin | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_shake_part_vector_insert_n` (through `+0x21f4`) | `car_extended_game_init`, `car_extended_game_done`, `car_extended_update`, `car_extended_dtor` | - |
| +0x21fc | shake_end | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_game_init` (= begin), `car_extended_shake_part_vector_insert_n` (through `+0x21f4`), `car_extended_shake_part_vector_erase` (through `+0x21f4`, end = begin) | `car_extended_game_done`, `car_extended_update`, `car_extended_scan_frame_tags` (the insertion point, through `+0x21f4`) | `car_extended_game_done` hands begin / end to `car_extended_shake_part_vector_erase` |
| +0x2200 | shake_cap | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_shake_part_vector_insert_n` (through `+0x21f4`) | `car_extended_shake_part_vector_insert_n` (through `+0x21f4`) | - |
| +0x2204 | exhaust_alloc | H | `car_extended_ctor` | - | the vector's address goes to `stl_insert_n_stride_0x18` (`car_extended_scan_frame_tags`) and to `car_extended_game_done` |
| +0x2208 | exhaust_begin | H | `car_extended_ctor`, `car_extended_dtor`, `stl_insert_n_stride_0x18` (through `+0x2204`) | `car_extended_game_init`, `car_extended_game_done`, `car_extended_update`, `car_extended_dtor` | - |
| +0x220c | exhaust_end | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_game_init` (= begin), `car_extended_game_done` (the end returned by `car_extended_exhaust_part_vector_copy`), `stl_insert_n_stride_0x18` (through `+0x2204`) | `car_extended_game_done`, `car_extended_scan_frame_tags`, `car_extended_update` | - |
| +0x2210 | exhaust_cap | H | `car_extended_ctor`, `car_extended_dtor`, `stl_insert_n_stride_0x18` (through `+0x2204`) | `stl_insert_n_stride_0x18` (through `+0x2204`) | - |
| +0x2214 | dcs_present | H | `car_extended_game_init` (= 0), `car_extended_scan_frame_tags` (= 1 on the first `~DCS~` frame) | `car_extended_can_board`, `human_intern_use_car_truck` | ¹ |
| +0x2218 | dcs_position | H | `car_extended_scan_frame_tags` | `human_intern_use_car_truck` | copied into the boarding human's position `+0x24` before the human is parented to the car frame |
| +0x2224 | dcs_direction | H | `car_extended_scan_frame_tags` | `human_intern_use_car_truck` | copied into the boarding human's direction `+0x30` |
| +0x2230 | taken_out_list | H | `car_extended_game_init` (= 0), `car_extended_scan_frame_tags` (allocates it), `car_extended_game_done` (frees it, = 0) | `car_extended_can_board`, `car_extended_update`, `car_extended_game_done`, `car_extended_scan_frame_tags`, `human_intern_use_car_truck`, `human_throw_from_car_truck`, `human_game_done`, `human_movement_intern`, `human_test_move_coll_in_owner` | the car's `~DCC` boxes and a boarded rider's own dyn-colls are taken out to it (`collision_dyn_take_out_to_list`); a departing rider puts its own back (`collision_dyn_put_back_from_list`) |
| +0x2234 | dyn_alloc | H | `car_extended_ctor` | - | the vector's address goes to `std_vector10_insert_n` (`car_extended_scan_frame_tags`) |
| +0x2238 | dyn_begin | H | `car_extended_ctor`, `car_extended_dtor`, `std_vector10_insert_n` (through `+0x2234`) | `car_extended_game_init`, `car_extended_game_done`, `car_extended_dtor` | - |
| +0x223c | dyn_end | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_game_init` (= begin), `car_extended_game_done` (= begin), `std_vector10_insert_n` (through `+0x2234`) | `car_extended_game_done`, `car_extended_scan_frame_tags` | `car_extended_scan_frame_tags` inserts at the end |
| +0x2240 | dyn_cap | H | `car_extended_ctor`, `car_extended_dtor`, `std_vector10_insert_n` (through `+0x2234`) | `std_vector10_insert_n` (through `+0x2234`) | - |
| +0x2244 | riders_alloc | H | `car_extended_ctor` | - | `car_extended_add_carried_person` takes its address |
| +0x2248 | riders_begin | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_add_carried_person` (on a reallocation) | `car_extended_add_carried_person`, `car_extended_remove_carried_person`, `car_extended_update`, `car_extended_game_init`, `car_extended_dtor` | riders are persons outside any seat (`human+0xac == -1`) |
| +0x224c | riders_end | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_game_init` (= begin), `car_extended_add_carried_person`, `car_extended_remove_carried_person` | `car_extended_add_carried_person`, `car_extended_remove_carried_person`, `car_extended_update` | the push-back is open-coded in `car_extended_add_carried_person` |
| +0x2250 | riders_cap | H | `car_extended_ctor`, `car_extended_dtor`, `car_extended_add_carried_person` (on a reallocation) | `car_extended_add_carried_person` | - |

¹ `car_extended_can_board` returns `dcs_present && taken_out_list`, the gate the
`human_entertotruck` script command passes before it calls `human_intern_use_car_truck`;
`human_intern_use_car_truck` tests the same pair itself.

## 3. Method family

Coverage: all **11** PS2 `8C_car_ex` methods are in the tables below, **11 CONFIRMED twins**. Two
more twins are symbols of the class's helper types (the frame-tag callback of a file-local helper
struct, and one `gif::vector` instantiation). The scalar-deleting destructor, two vector helpers
and `car_extended_can_board` have no PS2 symbol. Eight vtable slots are overridden (§6).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00474cc0 | `crt_static_init_guard_00474cc0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00474ce0 | `crt_atexit_register_00474ce0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00474cf0 | `car_extended_ctor` | instance | `__ct__8C_car_exFv` | 0x00143a90 | CONFIRMED | `car_ctor`, then the four empty vectors, both vtables and `class_id` = 3 |
| 0x00474d90 | `car_extended_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `car_extended_dtor`, vtable slot 16 ¹ |
| 0x00474db0 | `car_extended_dtor` | instance | `__dt__8C_car_exFv` | 0x00143b40 | CONFIRMED | frees the four vectors in the reverse of the constructor's order, then tail-calls `car_dtor` |

¹ No standalone PS2 twin: GCC carries the free flag inside the destructor.

### 3b. Accessors

The class has no accessor of its own: `car_extended_can_board` reads two fields (§3c).

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00474e50 | `car_extended_init` | `Init__8C_car_exFP9I3D_frame` | 0x00143c40 | CONFIRMED | slot 18; a one-line forwarder to `car_init` that narrows the result to a bool, on both builds |
| 0x00474e70 | `car_extended_game_init` | `GameInit__8C_car_exFv` | 0x00143c60 | CONFIRMED | slot 25; `car_game_init`, empties the four vectors and the `~DCS` state, then scans the model frame tree when the actor has a frame (§4) |
| 0x004758e0 | `car_extended_game_done` | `GameDone__8C_car_exFv` | 0x00146300 | CONFIRMED | slot 26; restores every part to its rest pose; each `~DCC` box leaves `taken_out_list` (`collision_dyn_put_back_from_list`), is erased from the grid (`collision_dyn_erase`) and freed; frees `taken_out_list`, then `car_game_done` ¹ |
| 0x00475bc0 | `car_extended_load_data_chunk` | `LoadData__8C_car_exFP7C_chunk` | 0x00146650 | CONFIRMED | slot 20; a one-line bool forwarder to `car_load_data_chunk`: the class serialises nothing of its own |
| 0x00475be0 | `car_extended_save_data_chunk` | `SaveData__8C_car_exFP7C_chunk` | 0x00146670 | CONFIRMED | slot 19; a one-line bool forwarder to `car_save_data_chunk` |
| 0x00475c00 | `thunk_car_copy_carinit_from_actor` | `DuplicateFrom__8C_car_exFP7C_actor` | 0x00146690 | CONFIRMED | slot 21; a 5-byte `jmp` to `car_copy_carinit_from_actor`, as the 8-byte PS2 body is a tail-jump to the base ² |
| 0x00475c10 | `car_extended_add_carried_person` | `DCAddPerson__8C_car_exFP7C_human` | 0x001466a0 | CONFIRMED | null guard, a backward duplicate scan, then the push-back; `DC` = dynamic collision: riders on the body, not in a seat |
| 0x00475dd0 | `car_extended_remove_carried_person` | `DCDelPerson__8C_car_exFP7C_human` | 0x00146730 | CONFIRMED | a backward scan, then swap-with-last and shrink; the same algorithm on both builds |
| 0x00475e40 | `car_extended_update` | `Update__8C_car_exFUi` | 0x00146790 | CONFIRMED | slot 14; five blocks ³ |
| 0x005c8c80 | `car_extended_can_board` | - | - | - | `dcs_present && taken_out_list` (§2 ¹); the PS2 build inlines the test into its own `human_entertotruck` arm, so no symbol exists |

¹ Both builds share the flag masks `& 0xfffffedf | 0x40000000` and `& 0xfffffecb | 0x40000008`.
² Ghidra names a thunk after its target, so the function's Ghidra id is `thunk_FUN_00464f50`.
³ Burnt-corpse swap and eject (`ohorelec01.i3d`); the base `car_update`; a push of every rider
through its vtable; a tilt eject at 0.785 rad; then the shake / exhaust animation (§5a, §5b).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00474f60 | `car_extended_scan_frame_tags` | `cbEnumProp__Q28C_car_ex22S_hlp$2074C_car_ex_cppFP9I3D_frameP8C_car_ex` | 0x00144630 | CONFIRMED | the recursive frame enumeration callback `(frame, car)`: parses the `~E` / `~C` / `~DCS` / `~DCC` tags (§4); a static of a file-local helper struct on the PS2 |
| 0x00476bb0 | `car_extended_shake_part_vector_erase` | - | - | - | one callsite (`car_extended_game_done`), the shake range `[begin, end)`, element 0x38; the PS2 clears the vector inline in `GameDone` |
| 0x00476c00 | `car_extended_shake_part_vector_insert_n` | `push_back__Q23gif28vector<Q28C_car_ex6S_clap,8>FRCQ28C_car_ex6S_clap` | 0x00145d30 | STRUCTURAL | one callsite, receiver `+0x21f4`, element 0x38; an N->1 fold of the PS2's `push_back` (748 B) and `reserve` (308 B) of the shake vector |
| 0x00476f20 | `car_extended_exhaust_part_vector_copy` | - | - | - | one callsite (`car_extended_game_done`), element 0x18; a forward copy, not the generic `stl_copy_back_stride_0x18` ¹ |

¹ `stl_copy_back_stride_0x18` (`0x00474c10`, the tail of the previous unit) has the same element
size and copies in the opposite direction.

## 4. The `~` frame-tag grammar

Recovered from `car_extended_scan_frame_tags` and **round-tripped against the shipped models** in
`exe-re/game-sources/models/`: every tag the code can parse exists in the data, and every tag in the
data parses.

| tag | parsed as | shipped users |
|---|---|---|
| `~E~` | an **exhaust part**: `{frame, random 0..2π phase, rest quat}` appended to the exhaust vector | `traktor.4ds`, `valec.4ds`: the two vehicles with a vertical exhaust stack |
| `~C` `[R]` `<X\|Y\|Z>` `<value>` `~` | a **shake part** ¹ | `traktor.4ds` (`~CRX0.2~`, `~CRX0.35~`, `~CY3~`, `~CZ2~`), `fordtTud00K.4ds` (`~CX3~`, `~CX5~`) |
| `~DCS~` | the **cargo-bed rider spot**: sets `dcs_present`, `dcs_position`, `dcs_direction`, where `human_intern_use_car_truck` places a boarding rider | the 5 truck models, on a frame named **`misto`** (Czech: *place*) |
| `~DCC~` | a **dynamic-collision box**, only on frames of type 6 (dummy) ² | the 5 truck models, on `Dummy16` … `Dummy20` and **`kolize pla`** (*collision tarp*) |

¹ `R` ⇒ rotate by `<value>` **degrees** (`crt_atof` × `0.01745`); no `R` ⇒ translate by `<value>`
**millimetres** (`crt_atof` × `0.001`), with a ±0.4 random jitter on the other two axes. The
trailing `~` is found with `strchr` and replaced by a NUL.
² Builds the 0x48-byte record of §5c, takes it out of the world grid into `taken_out_list`
(`collision_dyn_take_out_to_list`) and appends a `S_dyn_coll` to the dyn-coll vector.

**Complete shipped user set:** `traktor`, `valec`, `fordtTud00K`, `TruckBxx00`, `truckmorph`,
`truckmorph00`, `trucksplachtou`, `FMVanimTruckB00`.

The four tags do not cross-match: `~DCS~` / `~DCC~` contain no `~C` or `~E` pair, so the probe order
(`~E` → `~C` → `~DCS` → `~DCC`) is not load-bearing. Shipped `~C` amplitudes are **0.2°** and
**3 mm**: idle vibration, not a door or a hood.

## 5. The part and collision records

### 5a. The shake and exhaust parts

The **shake part** (`S_clap`, 0x38 bytes):

| Offset | Field | Notes |
|---|---|---|
| +0x00 | `I3D_frame *frame` | the tagged frame |
| +0x04 | `uint8_t mode` | 0 = translate, 1 = rotate |
| +0x08 | `S_vector axis` | from the `X` / `Y` / `Z` character, ±0.4 jitter on the other two in translate mode |
| +0x14 | `float phase` | advanced by `(0.04 + 0.02·r)·dt`, r the engine-rev fraction (§5b), wrapped at 2π |
| +0x18 | `float amplitude` | degrees × 0.01745 or millimetres × 0.001 |
| +0x1c | `S_vector rest_pos` | the frame's position at scan time |
| +0x28 | `S_quat rest_rot` | the frame's rotation at scan time |

Applied each frame as `offset = sin(phase) · (1 − 0.5·r) · amplitude · axis + rest_pos` (mode 0) or
as a quaternion about `axis` composed with `rest_rot` (mode 1).

The **exhaust part** (`S_exh_top`, 0x18 bytes) is `{frame, phase, rest quat}`; its phase advances by
`dt · 0.006` and it rotates about local Z, with the swing scaled by `(1 − r)·π/2`.

### 5b. The engine-rev fraction r

`car_extended_update` drives both families from the engine state of the base car
(`structs/car.md` §1, `structs/vehicle.md` §1):

| Input | Field | Role |
|---|---|---|
| engine running | `car+0xd18` = `C_Vehicle+0xca8` | clear: only the exhaust parts are posed; set: both families animate |
| engine revs per second | `car+0x634` = `C_Vehicle+0x5c4` | the live value |
| idle revs per second | `car+0x6d8` = `C_Vehicle+0x668` | CARINIT `idling rpm` ÷ 60 |
| maximum revs per second | `car+0x1f88` = `C_Vehicle+0x1f18` | CARINIT `maximum rpm` ÷ 60 |

`r = (revs − idle) / (max − idle)`, clamped to 0..1: the normalisation `vehicle_update_strips`
applies too. As the engine revs up, the shake speeds up and its amplitude falls to half, and the
exhaust swing shrinks to nothing at maximum revs.

### 5c. The `~DCC` box

Each `~DCC~` frame produces one 0x10-byte `S_dyn_coll` element of the dyn-coll vector, built on the
stack by `car_extended_scan_frame_tags` and inserted with `std_vector10_insert_n`:

| Offset | Field | Notes |
|---|---|---|
| +0x00 | the 0x48-byte `tDynamicCollObject` below | freed by `car_extended_game_done` |
| +0x04 | the tagged `I3D_frame` | - |
| +0x08 | the 0x40-byte transform | freed by `car_extended_game_done` |
| +0x0c | the 0x40-byte inverse transform | freed by `car_extended_game_done` |

The 0x48-byte record: every one of its 18 dwords is written by `car_extended_scan_frame_tags` (the
allocation is zero-filled first). Word 0 is the dynamic-collision type, and `0x40` is the
transformed-box type of `structs/collision.md` §5c.

| Offset | Field | Source |
|---|---|---|
| +0x00 | type `0x40` | literal |
| +0x04 | the tagged `I3D_frame` | the scanned frame |
| +0x08 | query-exclusion mask, 0 | `collision_line_in_dynamics` skips an object when this word shares a bit with the query's `+0x3c` |
| +0x0c | grid-bucket back-link, 0 | set by `collision_dyn_insert` (`structs/collision.md` §4) |
| +0x10 | centre | the frame's local position `frame+0x80` |
| +0x1c | bounding radius | the length of the larger scaled box corner |
| +0x20 | zero fill | - |
| +0x24 / +0x30 | box min / max corner | the frame bounding box `frame+0x140` × the frame scale `frame+0x90` |
| +0x3c | pointer to the transform | the frame rotation plus its position |
| +0x40 | pointer to the inverse transform | `S_matrix::Inverse` of a copy |
| +0x44 | owner | the `C_car_ex` |

## 6. The vtable

`g_car_extended_vt` @ `0x0063bd68`, 41 slots: the same width and slot order as `C_car`'s
`0x0063bc08`, slot for slot. **Exactly 8 slots differ**, the 8 methods `C_car_ex` overrides:

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 14 | 0x00475e40 | `car_extended_update` | `Update__8C_car_exFUi` |
| 16 | 0x00474d90 | `car_extended_scalar_deleting_dtor` | - |
| 18 | 0x00474e50 | `car_extended_init` | `Init__8C_car_exFP9I3D_frame` |
| 19 | 0x00475be0 | `car_extended_save_data_chunk` | `SaveData__8C_car_exFP7C_chunk` |
| 20 | 0x00475bc0 | `car_extended_load_data_chunk` | `LoadData__8C_car_exFP7C_chunk` |
| 21 | 0x00475c00 | `thunk_car_copy_carinit_from_actor` | `DuplicateFrom__8C_car_exFP7C_actor` |
| 25 | 0x00474e70 | `car_extended_game_init` | `GameInit__8C_car_exFv` |
| 26 | 0x004758e0 | `car_extended_game_done` | `GameDone__8C_car_exFv` |

The `C_car` functions in the same slots are `car_update`, `car_scalar_deleting_dtor`, `car_init`,
`car_save_data_chunk`, `car_load_data_chunk`, `car_copy_carinit_from_actor`, `car_game_init` and
`car_game_done`. The second vtable at `+0x70` is the `C_car` / `C_Vehicle` multiple-inheritance
pair's second base.

## 7. PC vs PS2

**Class identity.** PS2 mangled tag `8C_car_ex`, source file `C_car_ex.cpp` (a PS2 `__sinit_`
symbol). PS2 vtable `__vt__8C_car_ex`; second base pointer at `+0x22d4`. Alphabetical link order
places it between `C_car` and `C_clock`; aligning that order with the PC's `.rdata` vtable sequence
gives the PC vtable `0x0063bd68`.

**Field bridge.**

| PC | PS2 | Note |
|---|---|---|
| `+0x14` | `+0x54` | `class_id`; one step off the kind bridge `+0x10` ↔ `+0x50` |
| `+0x21f4` | `+0x24d0` | shake-part vector; element 0x38 vs 0x40 ¹ |
| `+0x2204` | `+0x24dc` | exhaust-part vector; element 0x18 vs 0x20 |
| `+0x2214` / `+0x2218` / `+0x2224` | `+0x24b0` / `+0x2490` / `+0x24a0` | `~DCS` flag / position / direction |
| `+0x2230` | `+0x24b4` | `taken_out_list` |
| `+0x2234` | `+0x24b8` | the `S_dyn_coll` vector |
| `+0x2244` | `+0x24c4` | the rider vector (the PS2 keeps the count at `+0x24c8` and the data at `+0x24cc`) |

¹ The 8 extra bytes are the two `S_vector` members, 12 bytes on the PC against 16 on the PS2.

The PS2 `human_entertotruck` arm tests `car+0x24b0 && car+0x24b4`, the pair that maps to PC
`+0x2214` / `+0x2230`; the PC factors the same test out as `car_extended_can_board`.

**The two builds declare these members in a different order**: the PS2 is
`[spot pos, dir, flag, list, dyn_coll, persons, shake, exhaust]`, the PC is
`[shake, exhaust, flag, pos, dir, list, dyn_coll, persons]`. No twin in §3 depends on it (all rest
on shared constants and vtable slots).

**Factoring differences.**

* `GameInit__8C_car_exFv` is 2500 B against the PC's 236 B because the **PS2 inlines one copy of
  the tag scan into it**; the PC calls `car_extended_scan_frame_tags`.
* `cbEnumProp` is 5200 B against the PC's 2423 B because it carries the scan body **twice** (itself
  plus an inlined child pass): 2600 B per copy, a ratio of 0.93.
* The other twin sizes: ctor 148 / 160 B, dtor 151 / 252 B, `GameDone` 734 / 848 B, `DCDelPerson`
  111 / 88 B, `Update` 3428 / 2980 B (PC / PS2).
* The container helpers differ by template, not by behaviour: the PS2 uses `gif::vector<S_clap,8>`
  and `std::__vector_imp<S_dyn_coll>`, the PC uses MSVC `std::vector`, so the PC helper rows of §3d
  have no body agreement to assert. `car_extended_shake_part_vector_insert_n` (795 B) folds the
  PS2's `push_back<S_clap>` (748 B) and `reserve` (308 B).
* The PS2 calls newlib **`atof`** where the PC calls `crt_atof` (`0x00624ebb`); MSVC does not
  inline it, so the callee survives on both sides, and that statement alignment identifies
  `crt_atof`.

**PS2-side symbols with no separate PC function** (all realised by the PC's STL helpers of §3d, by
`std_vector10_insert_n`, or inline):

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00143b30 | `__ct__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1>FUi` | a template helper of the PS2 container; the PC's MSVC vector code holds it inline |
| 0x00145a80 | `__ct__Q28C_car_ex6S_clapFRCQ28C_car_ex6S_clap` | the shake-record copy ctor; the PC's MSVC vector code holds it inline |
| 0x00145ac0 | `reserve__Q23gif28vector<Q28C_car_ex6S_clap,8>Fi` | folded into `car_extended_shake_part_vector_insert_n` |
| 0x00145c00 | `reserve__Q23gif31vector<Q28C_car_ex9S_exh_top,8>Fi` | folded into `stl_insert_n_stride_0x18` |
| 0x00145d20 | `data__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv` | a template helper of the PS2 container; the PC's MSVC vector code holds it inline |
| 0x00146020 | `push_back__Q23gif31vector<Q28C_car_ex9S_exh_top,8>FRCQ28C_car_ex9S_exh_top` | realised by the shared `stl_insert_n_stride_0x18` |
| 0x00147340 | `insert__Q23std82__vector_imp<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>,0>FPQ28C_car_ex10S_dyn_collUiRCQ28C_car_ex10S_dyn_coll` | realised by the shared `std_vector10_insert_n` |
| 0x001477e0 | `swap__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1>FRQ310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1>` | a template helper of the PS2 container; the PC's MSVC vector code holds it inline |
| 0x00147800 | `clear__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv` | open-coded in `car_extended_game_done` (`dyn_end` = `dyn_begin`) |
| 0x00147850 | `__dt__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv` | a template helper of the PS2 container; the PC's MSVC vector code holds it inline |

## 8. Related

- `structs/car.md` — `C_car`, the base class (its §7 describes this subclass).
- `structs/vehicle.md` — `C_Vehicle`, the engine state the animation reads.
- `structs/actor.md` — `class_id` (`+0x14`).
- `structs/entity_kind.md` — the kind enum, which cannot see this class, and factory `case 0x24`.
- `structs/collision.md` — the dynamic-collision objects and the take-out / put-back pair.
- `structs/human.md` — `human_intern_use_car_truck`, `human_throw_from_car_truck`.

## 9. Open items

The class has no open item.
