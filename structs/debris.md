# `C_Debris` — the debris and spark effect

**Created** 2026-09-04 13:25 +0200 · **Modified** 2026-10-01 14:19 +0200

PC prefix **`debris_`**, code range `[0x0061e100, 0x006243ac]`. Its own translation unit,
`VE_Debris.cpp`, immediately after `VE_BreakageGlass.cpp`.

Two effects in one class, driven by one tick:

* **debris pieces** — small two-sided billboard triangles that are thrown from an impact, tumble
  under gravity and are deleted when they fall out of the world;
* **flashes** — line-batch spark streaks with a head and a trailing tail, drawn directly rather
  than as scene objects.

Like the glass engine it is a **`C_game` sub-object, at `C_game + 0x240`** — no singleton, and
therefore **no data symbols anywhere in the translation unit**.

---

## 1. What is measured

```c
typedef struct C_Debris {
/* +0x00 */ char          debris_alloc;    // [M] MSVC allocator byte
/* +0x04 */ S_DEBRIS     *debris_begin;    // [H] the live debris pieces, stride 0x1c
/* +0x08 */ S_DEBRIS     *debris_end;      // [H]
/* +0x0c */ S_DEBRIS     *debris_cap;      // [H]
/* +0x10 */ char          flash_alloc;     // [M] MSVC allocator byte
/* +0x14 */ S_FLASH      *flash_begin;     // [H] the live spark streaks, stride 0x44
/* +0x18 */ S_FLASH      *flash_end;       // [H]
/* +0x1c */ S_FLASH      *flash_cap;       // [H]
/* +0x20 */ float         gravity_x;       // [H] 0.0
/* +0x24 */ float         gravity_y;       // [H] -(the Init argument)
/* +0x28 */ float         gravity_z;       // [H] 0.0
} C_Debris;                                 // embedded at C_game + 0x240
```

**The gravity is a parameter here and a hard-coded `-8.0` in `glass_shatter_init`.** That is the
discriminator that separates the two `Init` functions, and it is a real gameplay difference: glass
shards always fall at the same rate, debris falls at whatever the caller asks for.

### 1a. `S_DEBRIS` — one debris piece (`0x1c` bytes)

```c
typedef struct S_DEBRIS {
/* +0x00 */ I3D_frame    *frame;           // [H] the 'OBJ_' frame carrying the billboard
/* +0x04 */ float         vel_x;           // [H] randomised inside the caller's spread and
/* +0x08 */ float         vel_y;           //     scaled by the caller's speed
/* +0x0c */ float         vel_z;           // [H]
/* +0x10 */ float         spin_angle;      // [H] wrapped into [0, 2pi) every tick
/* +0x14 */ float         spin_rate;       // [H] +/-11.309733 x (rand x 0.3 + 0.7),
                                           //     alternating sign per piece
/* +0x18 */ char          live;            // [H] cleared when vel_y drops below -20.0;
                                           //     the compaction pass then erases the record
} S_DEBRIS;
```

### 1b. `S_FLASH` — one spark streak (`0x44` bytes)

Seventeen dwords. The decoded part:

| offset | field | conf | evidence |
|---|---|---|---|
| `+0x00`..`+0x08` | head velocity | [H] | gravity is accumulated into it each tick |
| `+0x0c`..`+0x14` | tail velocity | [H] | integrated separately, only after the arm gate |
| `+0x18`..`+0x20` | base velocity | [H] | added × dt to both endpoints every tick and never touched by gravity; copied from the descriptor's optional `+0x08` vector, zero when that pointer is null |
| `+0x24`..`+0x2c` | head position | [H] | fed to `line_batch_add` |
| `+0x30`..`+0x38` | tail position | [H] | the other end of the drawn segment |
| `+0x3c` | age (seconds) | [H] | accumulated by `dt`; the `>= 0.05` gate arms the tail |
| `+0x40` | lifetime | [H] | the compaction pass drops the record when age exceeds it |

### 1c. `S_FLASH_INIT` — the spawn descriptor (`0x28` bytes)

Read field by field by `debris_generate_flash`; every producer zeroes all ten dwords first.

| offset | field | unit | use |
|---|---|---|---|
| `+0x00` | count | streaks | one record per iteration |
| `+0x04` | `S_vector *` origin | world position | copied into the tail position; the head position is the origin jittered |
| `+0x08` | `S_vector *` base velocity | m/s | copied into `S_FLASH+0x18`; null = zero |
| `+0x0c` | `S_vector *` direction | unit vector | the axis of the orthogonal basis |
| `+0x10` | direction jitter | basis units | `(rand() % 200 - 100) × 0.01 ×` this along each of the two perpendicular axes, then normalised |
| `+0x14` | origin jitter | m | the same draw applied to the head position |
| `+0x18` | speed min | m/s | the speed actually used (§2) |
| `+0x1c` | speed max | m/s | never reached (§2) |
| `+0x20` | lifetime min | s | `min + (rand() % 100) × (max - min) × 0.01` |
| `+0x24` | lifetime max | s | the upper bound of the lifetime draw |

---

## 2. The two effects

```
car_explosion / car_on_collision_apply_damage / crash_obj_ai / human_ai
   -> debris_generate(pos, count, size, spread, speed)
        per piece: driver 'OBJ_' frame
                   debris_prepare_object  -> a two-sided billboard triangle
                   random scale (rand x 0.3 + 0.7), alternating +/-11.309733 spin
        appended to the stride-0x1c vector

airplane_hit / car_on_collision_apply_damage x2 / crash_obj_ai
   -> debris_generate_flash(S_FLASH_INIT *)
        builds an orthogonal basis around the descriptor's direction,
        jitters origin and direction with rand() % 200 - 100 scaled by 0.01,
        speed = descriptor minimum (the random term is always 0, see below)
        appended to the stride-0x44 vector

game_tick -> debris_tick(dt)            dt clamped to 1..0x53 ms
        pass 1  every piece: gravity, integrate, spin + 2pi wrap, write the frame
                              transform, clear `live` once vel_y < -20.0
        pass 2  compaction: release dead frames, debris_list_erase_at
        pass 3  every flash: integrate both endpoints, age, arm the tail past 0.05 s,
                              draw through line_batch_add, then compact by age
```

**A flash's tail does not exist for the first 50 ms.** The `0.05` gate is what makes a spark read
as a growing streak rather than a fixed-length line.

**Every flash flies at the descriptor's minimum speed.** The speed is
`(float)((rand() % 100) / 100) × (max - min) + min`, and the division is an integer one
(`idiv` by `0x64`, then `fild`), so the random term is always 0. The lifetime draw multiplies
before it divides and does vary.

### 2a. The four producers

| producer | count | direction | direction jitter | origin jitter | speed | lifetime | base velocity |
|---|---|---|---|---|---|---|---|
| `airplane_hit` | 20 + a random draw | the caller's vector | 2.0 | 0 | 8.0 (max 18.0) | 0.25..0.5 s | none |
| `car_on_collision_apply_damage`, impact above 5.0 | from the impact, at most 15 | up `(0, 1, 0)` | 1.0 | 0.2 | 6.0 (max 10.0) | 0.085..0.2 s | none |
| `car_on_collision_apply_damage`, impact above 2.0 with `param_7 == 0` | from the impact, at most 7 | the reverse of the `param_6` vector, tilted up by 0.5 and normalised | 1.0 | 0.2 | `4t + 2` (max `4t + 5`) | `0.085 + 0.1(1 - t)` .. `+ 0.075` s | the `param_6` vector |
| `crash_obj_ai`, flag bit `0x10` of `+0x3b0` | 20 + a random draw | up `(0, 1, 0)` | 1.0 | 0 | 4.0 (max 8.0) | 0.1..0.25 s | none |

`t` in the third row is a value `car_on_collision_apply_damage` computes on the x87 stack just
before the call.

The PC scales `rand()` by `1/32767` (`3.051851e-05`) and `0.3/32767` (`9.155553e-06`).

---

## 3. PC coverage — 6 of 6 class methods

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00622260 | `debris_init` | `Init__8C_DebrisFf` | 0x003bb970 | CONFIRMED | gravity `(0, -g, 0)`, clear both vectors |
| 0x006222f0 | `debris_done` | `Done__8C_DebrisFv` | 0x003bba40 | CONFIRMED | release every piece's frame, empty both lists |
| 0x006223a0 | `debris_tick` | `Tick__8C_DebrisFUi` | 0x003bbb80 | CONFIRMED | the three passes above |
| 0x00622a30 | `debris_generate` | `GenerateDebris__8C_DebrisFRC8S_vectorifffPC8S_vector` | 0x003bc110 | CONFIRMED | spawn a burst of pieces |
| 0x006230f0 | `debris_generate_flash` | `GenerateFlash__8C_DebrisFP12S_FLASH_INIT` | 0x003bc8c0 | CONFIRMED | spawn spark streaks |
| 0x00623700 | `debris_prepare_object` | `PrepareObject__8C_DebrisFP10I3D_objectfPC8S_vector` | 0x003bd2b0 | CONFIRMED | build one piece's two-sided billboard |

### 3a. Container helpers

| PC VA | PC name | receiver census | verdict |
|---|---|---|---|
| 0x00623ab0 | `debris_list_erase_at` | 1 safe site, in `debris_tick`'s compaction pass | one owner |
| 0x00623b00 | `std_vector44_insert_at` | 3 safe sites in two unrelated modules | shared, mechanical name |

`std_vector44_insert_at` is shared with the pursuit-event subsystem
(`pursuit_event_dispatch_pursuit_vehicle`, `pursuit_event_save_game_load`) because `S_FLASH` and
the pursuit-event `S_ELEMENT` are **both exactly `0x44` bytes**, so one COMDAT serves both.

MSVC inlines both POD copy constructors: the 7-dword `S_DEBRIS` copy is an open-coded loop in
`debris_generate`, and the `0x11`-dword `S_FLASH` copy is a block inside the
`std_vector44_insert_at` call in `debris_generate_flash`. Both are bannered at those debris-side
sites, not inside `stl_fill_n_stride_0x1c` or `std_vector44_insert_at`, because those helpers are
shared.

---

## 4. The billboard geometry

`debris_prepare_object` builds six vertices, indices `0,1,2,3,4,5`:

| vertex | position (x size) | normal |
|---|---|---|
| 0 | `( 0.000,  1.000, 0)` | `(0, 0, -1)` |
| 1 | `(-0.707, -0.707, 0)` | `(0, 0, -1)` |
| 2 | `( 0.707, -0.707, 0)` | `(0, 0, -1)` |
| 3 | `( 0.000,  1.000, 0)` | `(0, 0, +1)` |
| 4 | `( 0.707, -0.707, 0)` | `(0, 0, +1)` |
| 5 | `(-0.707, -0.707, 0)` | `(0, 0, +1)` |

Two coincident triangles with opposite normals — a fragment that is visible from either side while
it tumbles. The third parameter is the ambient/diffuse colour.

---

## 5. PC vs PS2

**Geometry.** `debris_prepare_object` is byte-identical across the builds, vertex for vertex. The
PS2 defaults the colour parameter to `0x3dcccccd` = 0.1 grey when the caller passes none.

**Translation unit.** `VE_Debris.cpp`, named by the PS2's own `__sinit_VE_Debris.cpp`
(`0x004d0d80`), the entry immediately after `VE_BreakageGlass.cpp`. The PC `.CRT$XCU` slot
`0x006462f4` opens it at `0x00622230`. Part of the five-of-five ordered lockstep recorded in
[`glass_shatter.md`](glass_shatter.md) §6.

**Record sizes.** PS2 pads for VU alignment: `S_DEBRIS` `0x20` against the PC's `0x1c`, `S_FLASH`
`0x60` against `0x44`.

**RNG range.** The PS2 divides `rand()` by `2.1474836e+09` where the PC uses `1/32767`. Same
formula, different RNG range — which is why the shared magic constants (`11.309733`, `0.7`, `0.3`)
match while the multipliers do not.

**The flash speed draw is integer division on both builds.** `GenerateFlash__8C_DebrisFP12S_FLASH_INIT`
computes `param_2[6] + (max - min) × (float)((rand() % 100) / 100)`, so the PS2's flashes also fly
at the minimum speed: a source-level property, not a compiler difference.

**Containers.** `reserve__Q23gif19vector<8S_DEBRIS,8>Fi` and
`reserve__Q23gif18vector<7S_FLASH,8>Fi` are Metrowerks `gif::vector` growth functions —
**structural correspondents** of the PC's MSVC insert helpers, not twins. No `PS2 twin:` line is
stamped on `debris_list_erase_at` or `std_vector44_insert_at`.

**Two PS2 functions MSVC inlined**, both POD copy constructors (PC sites in §3a):

| PS2 symbol | size | PC site |
|---|---|---|
| `__ct__8S_DEBRISFRC8S_DEBRIS` | 72 B @ 0x003bc730 | the open-coded 7-dword copy loop in `debris_generate` |
| `__ct__7S_FLASHFRC7S_FLASH` | 64 B @ 0x003bd0f0 | the `0x11`-dword block in `debris_generate_flash` |

Both are graded CANDIDATE-inlined — the direction is a toolchain difference (Metrowerks out-lines a
POD copy, MSVC splices it) and no image-wide census of MSVC's POD-ctor out-lining exists.

**Size ratios** run 0.55–1.18. The two above 1.0 (`debris_tick` 1.18, `debris_generate` 1.11) are
the passes MSVC unrolls hardest; nothing here is an outlier.

---

## 6. Open items

| item | state |
|---|---|
| in game: removal and tail delay | a debris piece is silently removed once its vertical velocity drops below -20 (falling faster than 20 units/s), and a spark's tail only appears after 50 ms; neither has been watched |
| in game: gravity | debris gravity is caller-supplied, so different impacts should throw fragments that fall at different rates, unlike glass shards (-8.0); not observed |

---

## 7. Cross-references

* [`game.md`](game.md) — the owner; this class is the sub-object at `C_game + 0x240`.
* [`glass_shatter.md`](glass_shatter.md) — the sibling TU, `C_Break_plain_glass` at
  `C_game + 0x208`. Consecutive object files; both spawn `'OBJ_'` frames and both are ticked from
  `game_tick`.
* [`line_batch.md`](line_batch.md) — `line_batch_add`, how a flash is actually drawn.
* [`pursuit-event-manager.md`](pursuit-event-manager.md) — the *other* consumer of
  `std_vector44_insert_at`; its `S_ELEMENT` is also `0x44` bytes.
