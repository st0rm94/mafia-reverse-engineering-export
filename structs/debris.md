# `C_Debris` — the debris and spark effect

**Created** 2026-09-04 13:25 +0200 · **Modified** 2026-11-08 22:21 +0100

Two effects in one class, driven by one tick:

* **debris pieces**: small two-sided billboard triangles that are thrown from an impact, tumble
  under gravity and are deleted when they fall out of the world;
* **flashes**: line-batch spark streaks with a head and a trailing tail, drawn directly rather
  than as scene objects.

Like the glass engine it is a **`C_game` sub-object, at `C_game + 0x240`**: no singleton, and
therefore **no data symbols anywhere in the translation unit**.

| | |
|---|---|
| PC prefix | `debris_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x2c` (the gravity vector at `+0x20` is the last field; embedded at `C_game + 0x240`) |
| PC constructor | - (`game_ctor` builds the sub-object inline: the debris vector open-coded, the flash vector through `std_vector_construct_empty`) |
| PC translation unit | `[0x00622230, 0x00623e10)`: the unit's CRT pair, the six `debris_*` methods and two container helpers; the next unit's guard `crt_static_init_guard_00623e10` closes it. It follows `C_Break_plain_glass`'s unit |
| PS2 class | `C_Debris` (tag `8C_Debris`), 6 methods: 6 CONFIRMED twins, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_Debris {                   // 0x2c bytes; embedded at C_game + 0x240
    /* +0x00 */ uint8_t   debris_alloc;     // [M] the debris vector's MSVC allocator byte; never read (.text scan)
    /* +0x04 */ S_DEBRIS *debris_begin;     // [H] the live debris pieces, element 0x1c (§4a)
    /* +0x08 */ S_DEBRIS *debris_end;       // [H] one past the last debris piece
    /* +0x0c */ S_DEBRIS *debris_cap;       // [H] end of the debris vector's storage
    /* +0x10 */ uint8_t   flash_alloc;      // [M] the flash vector's MSVC allocator byte; never read (.text scan)
    /* +0x14 */ S_FLASH  *flash_begin;      // [H] the live spark streaks, element 0x44 (§4b)
    /* +0x18 */ S_FLASH  *flash_end;        // [H] one past the last spark streak
    /* +0x1c */ S_FLASH  *flash_cap;        // [H] end of the flash vector's storage
    /* +0x20 */ float     gravity_x;        // [H] gravity, x component: always 0.0
    /* +0x24 */ float     gravity_y;        // [H] gravity, y component: minus the Init argument
    /* +0x28 */ float     gravity_z;        // [H] gravity, z component: always 0.0
} C_Debris;
```

Two MSVC `std::vector`s and a gravity vector. Each vector is an allocator byte (padded to a dword)
and the begin / end / capacity pointers; it grows on demand, one element per live piece or
streak. **The gravity is a parameter here and a hard-coded `-8.0` in `glass_shatter_init`.** That
is the discriminator that separates the two `Init` functions, and it is a real gameplay
difference: glass shards always fall at the same rate, debris falls at whatever the caller asks
for.

## 2. Field evidence

Measured by a census of the decomp's `this+N` accesses inside the `debris_*` functions and of the
code that builds and frees the sub-object at `C_game + 0x240`. A helper that receives a vector by
its address writes and reads that vector's pointers through it.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | debris_alloc | M | `game_ctor` (inline) | - | an empty allocator, never read (`.text` scan): `game_*` code only takes `C_game + 0x240` by address |
| +0x04 | debris_begin | H | `game_ctor` (= 0), `debris_generate` (on a reallocation), `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `debris_init`, `debris_done`, `debris_tick`, `debris_generate`, `folded_vector_pair_free_at_0x04_0x14_00607500` | the push-back is open-coded in `debris_generate` |
| +0x08 | debris_end | H | `game_ctor` (= 0), `debris_init` (= begin), `debris_done`, `debris_generate`, `debris_list_erase_at`, `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `debris_done`, `debris_tick`, `debris_generate`, `debris_list_erase_at` | - |
| +0x0c | debris_cap | H | `game_ctor` (= 0), `debris_generate`, `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `debris_generate` | - |
| +0x10 | flash_alloc | M | `std_vector_construct_empty` (from `game_ctor`, receiver `+0x10`) | - | an empty allocator, never read (`.text` scan); the vector's address goes to `std_vector44_insert_at` (`debris_generate_flash`) |
| +0x14 | flash_begin | H | `std_vector_construct_empty` (= 0), `std_vector44_insert_at` (through `+0x10`), `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `debris_init`, `debris_done`, `debris_tick`, `folded_vector_pair_free_at_0x04_0x14_00607500` | - |
| +0x18 | flash_end | H | `std_vector_construct_empty` (= 0), `debris_init` (= begin), `debris_done`, `debris_tick` (compaction), `std_vector44_insert_at` (through `+0x10`), `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `debris_init`, `debris_done`, `debris_tick`, `debris_generate_flash` | - |
| +0x1c | flash_cap | H | `std_vector_construct_empty` (= 0), `std_vector44_insert_at` (through `+0x10`), `folded_vector_pair_free_at_0x04_0x14_00607500` (= 0) | `std_vector44_insert_at` (through `+0x10`) | - |
| +0x20 | gravity_x | H | `debris_init` (= 0) | `debris_tick` | - |
| +0x24 | gravity_y | H | `debris_init` (= −argument) | `debris_tick` | `game_init` passes the gravity |
| +0x28 | gravity_z | H | `debris_init` (= 0) | `debris_tick` | - |

## 3. Method family

Coverage: all **6** PS2 `8C_Debris` methods are in the tables below, **6 CONFIRMED twins**. The
class has no constructor of its own on either build (`game_ctor` builds it inline); its destructor
body is a folded two-vector free. The two container helpers have no PS2 twin (§7).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00607500 | `folded_vector_pair_free_at_0x04_0x14_00607500` | instance | - | - | - | the destructor body: frees and zeroes both vectors; reached from `game_dtor` with the receiver `C_game + 0x240`. A second copy of the two-vector body, whose other receiver is the `C_using_object` at `C_game + 0x29ec`, reached from unwind funclets (`structs/game.md`) |
| 0x00622230 | `crt_static_init_guard_00622230` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer (slot `0x006462f4`): sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00622250 | `crt_atexit_register_00622250` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00622260 | `debris_init` | `Init__8C_DebrisFf` | 0x003bb970 | CONFIRMED | gravity `(0, -g, 0)`, empties both vectors; called by `game_init` |
| 0x006222f0 | `debris_done` | `Done__8C_DebrisFv` | 0x003bba40 | CONFIRMED | releases every piece's frame, empties both vectors; called by `game_done` |
| 0x006223a0 | `debris_tick` | `Tick__8C_DebrisFUi` | 0x003bbb80 | CONFIRMED | the three passes of §5; called by `game_tick` |
| 0x00622a30 | `debris_generate` | `GenerateDebris__8C_DebrisFRC8S_vectorifffPC8S_vector` | 0x003bc110 | CONFIRMED | spawns a burst of pieces; four producers (§5) |
| 0x006230f0 | `debris_generate_flash` | `GenerateFlash__8C_DebrisFP12S_FLASH_INIT` | 0x003bc8c0 | CONFIRMED | spawns spark streaks; four producer sites (§5a) |
| 0x00623700 | `debris_prepare_object` | `PrepareObject__8C_DebrisFP10I3D_objectfPC8S_vector` | 0x003bd2b0 | CONFIRMED | builds one piece's two-sided billboard (§6); called by `debris_generate` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00623ab0 | `debris_list_erase_at` | - | - | - | the debris vector's erase, element 0x1c; one callsite, in `debris_tick`'s compaction pass: one owner |
| 0x00623b00 | `std_vector44_insert_at` | - | - | - | an element-0x44 vector insert; three callsites in two unrelated modules: `debris_generate_flash` and the pursuit-event subsystem (`pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_save_game_load`), because `S_FLASH` and the pursuit-event `S_ELEMENT` are **both exactly `0x44` bytes**, so one COMDAT serves both (also listed in `structs/pursuit_event.md`) |

MSVC inlines both POD copy constructors: the 7-dword `S_DEBRIS` copy is an open-coded loop in
`debris_generate`, and the `0x11`-dword `S_FLASH` copy is a block inside the
`std_vector44_insert_at` call in `debris_generate_flash`. Both are bannered at those debris-side
sites, not inside `stl_fill_n_stride_0x1c` or `std_vector44_insert_at`, because those helpers are
shared.

## 4. The records

### 4a. `S_DEBRIS` — one debris piece (`0x1c` bytes)

| Offset | Field | Conf | Evidence |
|---|---|---|---|
| +0x00 | `I3D_frame *frame` | H | the `'OBJ_'` frame carrying the billboard |
| +0x04 | `float vel_x` | H | randomised inside the caller's spread and scaled by the caller's speed |
| +0x08 | `float vel_y` | H | as `vel_x`; gravity is accumulated into it |
| +0x0c | `float vel_z` | H | as `vel_x` |
| +0x10 | `float spin_angle` | H | wrapped into `[0, 2π)` every tick |
| +0x14 | `float spin_rate` | H | `11.309733 × (r + 0.7)` on one piece and `-11.309733 × (r - 0.7)` on the next, with `r = rand / 32767`; the second range crosses zero, so the sign does not strictly alternate |
| +0x18 | `uint8_t live` | H | cleared when `vel_y` drops below −20.0; the compaction pass then erases the record |

### 4b. `S_FLASH` — one spark streak (`0x44` bytes)

Seventeen dwords:

| Offset | Field | Conf | Evidence |
|---|---|---|---|
| +0x00 | head velocity (vec3) | H | gravity is accumulated into it each tick |
| +0x0c | tail velocity (vec3) | H | integrated separately, only after the arm gate |
| +0x18 | base velocity (vec3) | H | added × dt to both endpoints every tick and never touched by gravity; copied from the descriptor's optional `+0x08` vector, zero when that pointer is null |
| +0x24 | head position (vec3) | H | fed to `line_batch_add` |
| +0x30 | tail position (vec3) | H | the other end of the drawn segment |
| +0x3c | age (seconds) | H | accumulated by `dt`; the `>= 0.05` gate arms the tail |
| +0x40 | lifetime | H | the compaction pass drops the record when age exceeds it |

### 4c. `S_FLASH_INIT` — the spawn descriptor (`0x28` bytes)

Read field by field by `debris_generate_flash`; every producer zeroes all ten dwords first.

| Offset | Field | Unit | Use |
|---|---|---|---|
| +0x00 | count | streaks | one record per iteration |
| +0x04 | `S_vector *` origin | world position | copied into the tail position; the head position is the origin jittered |
| +0x08 | `S_vector *` base velocity | m/s | copied into `S_FLASH+0x18`; null = zero |
| +0x0c | `S_vector *` direction | unit vector | the axis of the orthogonal basis |
| +0x10 | direction jitter | basis units | `(rand() % 200 - 100) × 0.01 ×` this along each of the two perpendicular axes, then normalised |
| +0x14 | origin jitter | m | the same draw applied to the head position |
| +0x18 | speed min | m/s | the speed actually used (§5) |
| +0x1c | speed max | m/s | never reached (§5) |
| +0x20 | lifetime min | s | `min + (rand() % 100) × (max - min) × 0.01` |
| +0x24 | lifetime max | s | the upper bound of the lifetime draw |

## 5. The two effects

```
car_explosion / car_on_collision_apply_damage / crash_obj_ai / human_ai
   -> debris_generate(pos, count, size, spread, speed)
        per piece: driver 'OBJ_' frame
                   debris_prepare_object  -> a two-sided billboard triangle
                   random scale (rand x 0.3 + 0.7), alternating 11.309733 / -11.309733 spin base
        appended to the element-0x1c vector

airplane_hit / car_on_collision_apply_damage x2 / crash_obj_ai
   -> debris_generate_flash(S_FLASH_INIT *)
        builds an orthogonal basis around the descriptor's direction,
        jitters origin and direction with rand() % 200 - 100 scaled by 0.01,
        speed = descriptor minimum (the random term is always 0, see below)
        appended to the element-0x44 vector

game_tick -> debris_tick(dt)            dt clamped to 1..0x53 ms
        pass 1  every piece: gravity, integrate, spin + 2pi wrap, write the frame
                              transform, clear `live` once vel_y < -20.0
        pass 2  compaction: release dead frames, debris_list_erase_at
        pass 3  every flash: integrate both endpoints, age, arm the tail past 0.05 s,
                              draw through line_batch_add, then compact by age
```

**A flash's tail does not exist for the first 50 ms.** The `0.05` gate is what makes a spark read
as a growing streak rather than a constant-length line.

**Every flash flies at the descriptor's minimum speed.** The speed is
`(float)((rand() % 100) / 100) × (max - min) + min`, and the division is an integer one (`idiv` by
`0x64`, then `fild`), so the random term is always 0. The lifetime draw multiplies before it
divides and does vary.

### 5a. The four flash producers

| producer | count | direction | direction jitter | origin jitter | speed | lifetime | base velocity |
|---|---|---|---|---|---|---|---|
| `airplane_hit` | 20 + a random draw | the caller's vector | 2.0 | 0 | 8.0 (max 18.0) | 0.25..0.5 s | - |
| `car_on_collision_apply_damage`, impact above 5.0 | from the impact, at most 15 | up `(0, 1, 0)` | 1.0 | 0.2 | 6.0 (max 10.0) | 0.085..0.2 s | - |
| `car_on_collision_apply_damage`, impact above 2.0 with `param_7 == 0` | from the impact, at most 7 | the reverse of the `param_6` vector, tilted up by 0.5 and normalised | 1.0 | 0.2 | `4t + 2` (max `4t + 5`) | `0.085 + 0.1(1 - t)` .. `+ 0.075` s | the `param_6` vector |
| `crash_obj_ai`, flag bit `0x10` of `+0x3b0` | 20 + a random draw | up `(0, 1, 0)` | 1.0 | 0 | 4.0 (max 8.0) | 0.1..0.25 s | - |

`t` in the third row is a value `car_on_collision_apply_damage` computes on the x87 stack just
before the call.

The PC scales `rand()` by `1/32767` (`3.051851e-05`) and `0.3/32767` (`9.155553e-06`).

## 6. The billboard geometry

`debris_prepare_object` builds six vertices, indices `0,1,2,3,4,5`:

| vertex | position (× size) | normal |
|---|---|---|
| 0 | `( 0.000,  1.000, 0)` | `(0, 0, -1)` |
| 1 | `(-0.707, -0.707, 0)` | `(0, 0, -1)` |
| 2 | `( 0.707, -0.707, 0)` | `(0, 0, -1)` |
| 3 | `( 0.000,  1.000, 0)` | `(0, 0, +1)` |
| 4 | `( 0.707, -0.707, 0)` | `(0, 0, +1)` |
| 5 | `(-0.707, -0.707, 0)` | `(0, 0, +1)` |

Two coincident triangles with opposite normals: a fragment that is visible from either side while
it tumbles. The third parameter is the ambient/diffuse colour.

## 7. PC vs PS2

**Geometry.** `debris_prepare_object` is byte-identical across the builds, vertex for vertex. The
PS2 defaults the colour parameter to `0x3dcccccd` = 0.1 grey when the caller passes none.

**Translation unit.** `VE_Debris.cpp`, named by the PS2's own `__sinit_VE_Debris.cpp`
(`0x004d0d80`), the entry immediately after `VE_BreakageGlass.cpp`. The PC `.CRT$XCU` slot
`0x006462f4` opens it at `0x00622230`. Part of the five-of-five ordered lockstep recorded in
`structs/glass_shatter.md` §7.

**Record sizes.** The PS2 pads for VU alignment: `S_DEBRIS` `0x20` against the PC's `0x1c`,
`S_FLASH` `0x60` against `0x44`.

**RNG range.** The PS2 divides `rand()` by `2.1474836e+09` where the PC uses `1/32767`. Same
formula, different RNG range, which is why the shared magic constants (`11.309733`, `0.7`, `0.3`)
match while the multipliers do not.

**The flash speed draw is integer division on both builds.**
`GenerateFlash__8C_DebrisFP12S_FLASH_INIT` computes
`param_2[6] + (max - min) × (float)((rand() % 100) / 100)`, so the PS2's flashes also fly at the
minimum speed: a source-level property, not a compiler difference.

**Containers.** `reserve__Q23gif19vector<8S_DEBRIS,8>Fi` and
`reserve__Q23gif18vector<7S_FLASH,8>Fi` are Metrowerks `gif::vector` growth functions:
**structural correspondents** of the PC's MSVC insert helpers, not twins. No `PS2 twin:` line is
stamped on `debris_list_erase_at` or `std_vector44_insert_at`.

**PS2 functions with no PC function of their own:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0037b0b0 | `__dt__Q23gif18vector<7S_FLASH,8>Fv` | the vector destructors: the PC frees both vectors in one folded body (§3a) |
| 0x0037b120 | `__dt__Q23gif19vector<8S_DEBRIS,8>Fv` | the same |
| 0x003bc730 | `__ct__8S_DEBRISFRC8S_DEBRIS` | a 72-byte POD copy ctor, inlined as the open-coded 7-dword copy loop in `debris_generate` |
| 0x003bc780 | `reserve__Q23gif19vector<8S_DEBRIS,8>Fi` | MSVC grows inside the insert: open-coded in `debris_generate` |
| 0x003bd0f0 | `__ct__7S_FLASHFRC7S_FLASH` | a 64-byte POD copy ctor, inlined as the `0x11`-dword block in `debris_generate_flash` |
| 0x003bd130 | `reserve__Q23gif18vector<7S_FLASH,8>Fi` | MSVC grows inside the insert: `std_vector44_insert_at` |

Both copy-ctor inlinings are measured on the call graph. The image does hold out-of-line
single-element copy helpers for other element types, and the only two whose copy size matches,
`stl_construct_stride_0x1c` (7 dwords) and `std_vector44_copy_one` (`0x11` dwords), are never
reached from this translation unit: the first is called only by AI, vehicle, HUD and UV-scroll
code, the second only by `std_vector44_insert_one`, which the debris code does not call.

**Size ratios** run 0.55–1.18. The two above 1.0 (`debris_tick` 1.18, `debris_generate` 1.11) are
the passes MSVC unrolls hardest; nothing here is an outlier.

## 8. Related

- `structs/game.md` — the owner; this class is the sub-object at `C_game + 0x240`.
- `structs/glass_shatter.md` — the sibling unit, `C_Break_plain_glass` at `C_game + 0x208`.
  Consecutive object files; both spawn `'OBJ_'` frames and both are ticked from `game_tick`.
- `structs/line_batch.md` — `line_batch_add`, how a flash is drawn.
- `structs/pursuit_event.md` — the other consumer of `std_vector44_insert_at`; its
  `S_ELEMENT` is also `0x44` bytes.
- `structs/crash_obj.md` — one of the producers.

## 9. Open items

1. **In game: removal and tail delay.** A debris piece is silently removed once its vertical
   velocity drops below −20 (falling faster than 20 units/s), and a spark's tail only appears after
   50 ms; neither has been watched.
2. **In game: gravity.** Debris gravity is caller-supplied, so different impacts should throw
   fragments that fall at different rates, unlike glass shards (−8.0); not observed.
