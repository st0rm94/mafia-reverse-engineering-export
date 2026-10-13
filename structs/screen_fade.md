# The screen-fade object — a cinematic letterbox that is never started

**Created** 2026-09-03 00:36 +0200 · **Modified** 2026-10-07 21:57 +0200

A `0x14`-byte file-scope object at `0x006f9278`, constructed in the `Main.cpp` initializer block
(`[0x005f79a0, 0x005f8370]`), with a two-method class: a per-frame tick and a destructor. It
interpolates one parameter between 0 and 1 at a settable speed and direction, and it carries the
constant **1/1.85** — the cinematic aspect ratio. In the shipped PC build **nothing ever starts it**.

---

## 1. Field layout

```c
typedef struct {                    /* at 0x006f9278, sizeof = 0x14 */
/* +0x00 */ char  active;           // [H] g_fade_active — gates the per-frame tick
/* +0x01 */ char  animating;        // [H] set while t is moving; cleared at either end
/* +0x04 */ float speed;            // [H] g_screen_fade_speed, ctor writes 1.0f
/* +0x08 */ float t;                // [H] g_screen_fade_t, clamped to [0, 1]
/* +0x0c */ int   direction;        // [H] +1 / -1, multiplied into the step
/* +0x10 */ float aspect;           // [H] g_screen_fade_aspect, 0x3f0a60dd = 1/1.85; ctor writes
                                    //     it, nothing reads it
} S_screen_fade;
```

### 1a. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x00` | `active` | H | `screen_fade_static_ctor`, `screen_fade_tick` | `engine_game_loop`, `race_game_loop`, `race_game_loop_net` | `if (g_fade_active) screen_fade_tick(dt)` in all three loops; the tick clears it when `t` reaches 0 |
| `+0x01` | `animating` | H | `screen_fade_tick` | `screen_fade_tick` | the tick's own gate; **nothing sets it** |
| `+0x04` | `speed` | H | `screen_fade_static_ctor` | `screen_fade_tick` | `1.0f` |
| `+0x08` | `t` | H | `screen_fade_static_ctor`, `screen_fade_tick` | `screen_fade_tick` | `t += (dt/1000) * speed * direction`, clamped |
| `+0x0c` | `direction` | H | - | `screen_fade_tick` | **no writer on either build** |
| `+0x10` | `aspect` | H | `screen_fade_static_ctor` | - | `1/1.85`; no reader on either build (§3) |

---

## 2. PC functions

| PC VA | PC name | role |
|---|---|---|
| `0x005f82d0` | `screen_fade_static_init` | `.CRT$XCU` entry |
| `0x005f82e0` | `screen_fade_static_ctor` | writes `active=0`, `speed=1.0f`, `t=0`, `aspect=1/1.85` |
| `0x005f8310` | `screen_fade_register_atexit_dtor` | `atexit(&screen_fade_static_dtor)` |
| `0x005f8320` | `screen_fade_static_dtor` | a one-byte `ret`; a data symbol, not a Ghidra function ¹ |
| `0x0061b5e0` | `screen_fade_tick` | the per-frame integrator |

¹ `0x005f8320` is 16-aligned and reachable only as the `atexit` argument, but Ghidra never made a
function there, so it exists in the decomp only as a data symbol and has no function-registry row.
It is **not** folded onto `crt_atexit_noop_dtor_0060b700`, the shared do-nothing destructor every
other registrar in this translation unit passes.

---

## 3. Nothing in the shipped PC build starts the fade

A measurement over a closed set:

1. **The object's whole method set is two functions**, `screen_fade_tick` and
   `screen_fade_static_dtor` (§2); no `Start(direction, speed)` member exists to have been inlined
   away (§4).
2. **A byte scan of the whole file for every address of the object, `0x006f9278` ..
   `0x006f928b`, returns ten references**: the constructor's four field writes, and in each of
   `engine_game_loop`, `race_game_loop` and `race_game_loop_net` one `test` (or `cmp`) of
   `[0x6f9278]` followed by `mov ecx, 0x6f9278; call screen_fade_tick`. A starter or a reader
   would have to materialise one of those addresses and would appear in the scan.
3. **`+0x01` and `+0x0c` therefore have no writer and `+0x10` has no reader**, so
   `screen_fade_tick` never runs its body: it returns on the `animating` gate every frame.

Same family as the seagull flock and `g_console_unused_vector` — code that ships, runs and does
nothing. Every statement above is about `Game.exe`'s bytes; the in-game confirmation is in §5.

---

## 4. PC vs PS2

The class is PS2 **`C_silver_screen`**, and its object is `silver_screen` at `0x0050a9a0`,
constructed by `__sinit_Main.cpp` in the same lockstep slot as the PC's. The PC name is
behaviour-derived; the PS2 noun is what identifies the `1/1.85` constant as a *letterbox* aspect
rather than an arbitrary float.

**Twin table** (PC VA order, PC name first):

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| `0x005f82e0` | `screen_fade_static_ctor` | the `silver_screen` block of `__sinit_Main.cpp` | CONFIRMED |
| `0x005f8320` | `screen_fade_static_dtor` | `__dt__15C_silver_screenFv` | STRUCTURAL |
| `0x0061b5e0` | `screen_fade_tick` | `Tick__15C_silver_screenFUi` | CONFIRMED |

### 4a. What the two ticks share

Statement for statement, at identical offsets:

```
if (this[1]) {
    t = this[+8] + (dt / 1000.0) * this[+4] * (float)(int)this[+0xc];
    this[+8] = t;
    if (t > 1.0)      { this[+8] = 1.0f; this[1] = 0; }
    else if (t < 0.0) { this[+8] = 0.0f; this[0] = 0; this[1] = 0; }
}
```

### 4b. The constructor block

PC writes `[0x6f9278] = 0`, `[+4] = 0x3f800000`, `[+8] = 0`, `[+0x10] = 0x3f0a60dd`.
PS2 writes `silver_screen = 0`, `[0x50a9a4] = 0x3f800000`, `[0x50a9a8] = 0`,
`[0x50a9b0] = 0x3f0a60dd`. **Four constants, four matching offsets, including the non-round
`0x3f0a60dd`.**

**The method set.** The PS2 class has exactly two symbols, `__dt__15C_silver_screenFv` and
`Tick__15C_silver_screenFUi`, so there is no `Start(direction, speed)` member on either build to
have been inlined away. The PS2 references to the object (address halves built with `lui 0x51`)
sit only in `GameLoopSingle__Fv`, `GameLoopSingle1__Fv` and `__sinit_Main.cpp`: the two loops'
test-and-tick and the constructor's writes, including the one to `+0x10`. The PS2 has no starter
and no reader of the aspect either.

---

## 5. Open items

| item | state |
|---|---|
| in game | does any cutscene or mission transition letterbox the screen to 1.85:1, or fade through a full-screen curtain? If never, §3 is confirmed; if it does, the writer of `+0x01` / `+0x0c` is the thing to find |
