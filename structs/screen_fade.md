# `C_silver_screen` — the screen-fade letterbox that nothing starts

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-11-01 19:11 +0100

A `0x14`-byte file-scope object at `0x006f9278` with a two-method class: a per-frame tick and a
destructor. It interpolates one parameter between 0 and 1 at a settable speed and direction, and it
carries the constant **1/1.85**, the cinematic aspect ratio. In the shipped PC build **nothing ever
starts it** (§4). The PC name is behaviour-derived; the class is PS2 `C_silver_screen` (§5).

| | |
|---|---|
| PC prefix | `screen_fade_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x14` (the object `g_fade_active` @ `0x006f9278` .. `0x006f928b`) |
| PC constructor | `screen_fade_static_ctor` @ `0x005f82e0` |
| PC translation unit | `[0x0061b050, 0x0061df00)`, the `tools.cpp` unit hosted by `structs/sound_table.md`, holds `screen_fade_tick`; the static quartet sits in the initializer block of `Main.cpp` (slot `0x005f82d0`, `structs/app_bootstrap.md` §4a) |
| PS2 class | `C_silver_screen` (tag `15C_silver_screen`), 2 methods, both mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_silver_screen {    // 0x14 bytes; the file-scope object at 0x006f9278
    /* +0x00 */ uint8_t active;     // [H] g_fade_active: gates the per-frame tick
    /* +0x01 */ uint8_t animating;  // [H] set while t is moving; cleared at either end
    /* +0x04 */ float   speed;      // [H] g_screen_fade_speed; the ctor writes 1.0f
    /* +0x08 */ float   t;          // [H] g_screen_fade_t, clamped to [0, 1]
    /* +0x0c */ int32_t direction;  // [H] +1 / -1, multiplied into the step
    /* +0x10 */ float   aspect;     // [H] g_screen_fade_aspect = 0x3f0a60dd = 1/1.85
} C_silver_screen;
```

Bytes `+0x02..+0x03` are alignment padding. The PS2 class name is what identifies `aspect` as a
letterbox ratio rather than an arbitrary float.

## 2. Field evidence

Measured by a byte scan of `Game.exe` for every address `0x006f9278` .. `0x006f928b` (ten references,
§4) and a body read of the two functions that touch the object.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | active | H | `screen_fade_static_ctor` (0), `screen_fade_tick` (0 when `t` falls below 0) | `engine_game_loop`, `race_game_loop`, `race_game_loop_net` | each loop runs `if (g_fade_active) screen_fade_tick(dt)` |
| +0x01 | animating | H | `screen_fade_tick` (0 at either end) | `screen_fade_tick` | the tick's own gate; no instruction sets it (§4) |
| +0x04 | speed | H | `screen_fade_static_ctor` (1.0f) | `screen_fade_tick` | - |
| +0x08 | t | H | `screen_fade_static_ctor` (0), `screen_fade_tick` | `screen_fade_tick` | `t += dt * 0.001 * speed * direction`, clamped |
| +0x0c | direction | H | - | `screen_fade_tick` | no writer on either build (§4) |
| +0x10 | aspect | H | `screen_fade_static_ctor` | - | no reader on either build (§4) |

## 3. Method family

Coverage: both PS2 `15C_silver_screen` methods are mapped: `Tick` CONFIRMED and the destructor
STRUCTURAL. The PC adds the MSVC static quartet of the file-scope object.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005f82d0 | `screen_fade_static_init` | `g_fade_active` | - | - | - | the `Main.cpp` `.CRT$XCU` entry: the ctor, then the `atexit` registration |
| 0x005f82e0 | `screen_fade_static_ctor` | `g_fade_active` | - | - | - | writes `active` 0, `speed` 1.0f, `t` 0, `aspect` 1/1.85; the PS2 writes the same four constants inline in `__sinit_Main.cpp` (§5b) |
| 0x005f8310 | `screen_fade_register_atexit_dtor` | `g_fade_active` | - | - | - | `crt_atexit(&screen_fade_static_dtor)` |
| 0x005f8320 | `screen_fade_static_dtor` | `g_fade_active` | `__dt__15C_silver_screenFv` | 0x00377ec0 | STRUCTURAL | a one-byte `ret` that Ghidra never made a function, so it exists only as a data symbol ¹; the PS2 body only deletes when its flag asks, which a static object never does |

¹ `0x005f8320` is 16-aligned and reachable only as the `atexit` argument. It is **not** folded onto
`crt_atexit_noop_dtor_0060b700`, the shared do-nothing destructor every other registrar of
`Main.cpp` passes.

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061b5e0 | `screen_fade_tick` | `Tick__15C_silver_screenFUi` | 0x003ad5b0 | CONFIRMED | the per-frame integrator (§5a); returns at once while `animating` is clear |

### 3d. Free helpers

The class has no free helper; the `tools.cpp` unit's helpers are listed in `structs/sound_table.md`.

## 4. Nothing in the shipped PC build starts the fade

A measurement over a closed set:

1. **The method set is two functions**, `screen_fade_tick` and `screen_fade_static_dtor` (§3), and
   the PS2 class has exactly the same two symbols, so no `Start(direction, speed)` member exists on
   either build to have been inlined away.
2. **A byte scan of the whole file for every address of the object, `0x006f9278` .. `0x006f928b`,
   returns ten references**: the constructor's four field writes, and in each of `engine_game_loop`,
   `race_game_loop` and `race_game_loop_net` one `test` (or `cmp`) of `[0x6f9278]` followed by
   `mov ecx, 0x6f9278; call screen_fade_tick`. A starter or a reader would have to materialise one
   of those addresses and would appear in the scan.
3. **`+0x01` and `+0x0c` therefore have no writer outside the tick and `+0x10` has no reader**, so
   `active` is always 0 after the constructor and the loops never call the tick; if they did, it
   would return on the `animating` gate.

Same family as the seagull flock and `g_console_unused_vector`: code that ships and does nothing.
Every statement above is about `Game.exe`'s bytes; the in-game check is Open item 1.

## 5. PC vs PS2

The object is PS2 `silver_screen` at `0x0050a9a0`, constructed by `__sinit_Main.cpp` in the same
slot of the initializer order as the PC's.

### 5a. What the two ticks share

Statement for statement, at identical offsets:

```
if (this[1]) {
    t = this[+8] + (dt / 1000.0) * this[+4] * (float)(int)this[+0xc];
    this[+8] = t;
    if (t > 1.0)      { this[+8] = 1.0f; this[1] = 0; }
    else if (t < 0.0) { this[+8] = 0.0f; this[0] = 0; this[1] = 0; }
}
```

### 5b. The constructor block

PC writes `[0x6f9278] = 0`, `[+4] = 0x3f800000`, `[+8] = 0`, `[+0x10] = 0x3f0a60dd`. PS2 writes
`silver_screen = 0`, `[0x50a9a4] = 0x3f800000`, `[0x50a9a8] = 0`, `[0x50a9b0] = 0x3f0a60dd`: four
constants at four matching offsets, including the non-round `0x3f0a60dd`.

### 5c. No starter on the PS2 either

The PS2 references to the object (address halves built with `lui 0x51`) sit only in
`GameLoopSingle__Fv`, `GameLoopSingle1__Fv` and `__sinit_Main.cpp`: the two loops' test-and-tick and
the constructor's writes, including the one to `+0x10`. The PS2 has no starter and no reader of the
aspect.

## 6. Related

- `structs/app_bootstrap.md` §4a — the `Main.cpp` initializer block that holds the static quartet.
- `structs/sound_table.md` — the host of the `tools.cpp` unit that holds `screen_fade_tick`.

## 7. Open items

1. **In game:** does any cutscene or mission transition letterbox the screen to 1.85:1, or fade
   through a full-screen curtain? If never, §4 is confirmed; if it does, the writer of `+0x01` /
   `+0x0c` is the thing to find.
