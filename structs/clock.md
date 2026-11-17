# `C_clock` — the working clock actor

**Created** 2026-08-12 20:08 +0200 · **Modified** 2026-10-28 16:49 +0100

An actor whose model is a clock: it resolves three named child frames as clock hands, seeds them
from the **machine's real wall clock** when the scene starts, and rotates them forward once per
elapsed REAL minute (`clock_ai_tick`'s argument is the frame delta in ms). It is one of the smallest
complete classes in the binary: five methods of its own and one destructor split.

| | |
|---|---|
| PC prefix | `clock_` |
| PC kind (`+0x10`) | `0x22`, stamped by `clock_ctor` |
| PC vtable | `g_clock_vt` @ `0x0063beb8`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x88` (`operator_new(0x88)` in `mission_create_actor` `case 0x22`; the last field ends at `0x88`) |
| PC constructor | `clock_ctor` @ `0x00477330` |
| PC translation unit | `[0x00477300, 0x004776c0)`: the unit's CRT pair, then the six `clock_*` functions and two thunks; the next unit's guard `crt_static_init_guard_004776c0` opens `C_crash_obj` |
| PS2 class | `C_clock` (tag `7C_clock`), 11 methods: 6 CONFIRMED twins, 5 base stubs, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC.

```c
typedef struct C_clock {                    // 0x88 bytes; PC offsets
    /* +0x00 */ C_actor    base;            // [H] the C_actor band (structs/actor.md)
    /* +0x70 */ I3D_frame *hand_hour;       // [H] child frame "Mala"; required
    /* +0x74 */ I3D_frame *hand_minute;     // [H] child frame "Velka"; required
    /* +0x78 */ I3D_frame *hand_second;     // [H] child frame "Sekundova"; optional (§4)
    /* +0x7c */ uint32_t   ms_accum;        // [H] milliseconds inside the current minute
    /* +0x80 */ float      angle_hour;      // [H] radians
    /* +0x84 */ float      angle_minute;    // [H] radians
} C_clock;
```

`hand_second` is the needle of the **seconds sub-dial** (§4a), one revolution per 60 s; two of the
three shipped clock models have no such frame, so it may stay null. The three hands are looked up
under the actor's own frame (`C_actor+0x68`).

## 2. Field evidence

Measured by a census of every operand with each displacement in the `clock_*` functions and of the
decomp's `this+N` accesses; no function outside the class touches the band.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `clock_ctor` (vtable `g_clock_vt`, kind `+0x10` = `0x22`), `clock_game_init` (`+0x65` = 1, the `C_actor` save-exclusion byte: the savegame skips the clock) | the `C_actor` methods of `structs/actor.md`; `clock_init` (`+0x68`, the lookup root) | the inherited band |
| +0x70 | hand_hour | H | `clock_init` | `clock_init`, `clock_game_init`, `clock_ai_tick` | `clock_init` fails without it |
| +0x74 | hand_minute | H | `clock_init` | `clock_init`, `clock_game_init`, `clock_ai_tick` | `clock_init` fails without it |
| +0x78 | hand_second | H | `clock_init` | `clock_game_init`, `clock_ai_tick` | every use is guarded by `if (hand_second != 0)` |
| +0x7c | ms_accum | H | `clock_game_init` (= 0), `clock_ai_tick` | `clock_ai_tick` | wraps at 60000 |
| +0x80 | angle_hour | H | `clock_game_init` (= 0, then the seed), `clock_ai_tick` | `clock_ai_tick` | §5 |
| +0x84 | angle_minute | H | `clock_game_init` (= 0, then the seed), `clock_ai_tick` | `clock_game_init`, `clock_ai_tick` | §5 |

## 3. Method family

Coverage: all **11** PS2 `7C_clock` methods are in the tables below: **6 CONFIRMED** twins
(`{ctor}`, `{dtor}`, `Init`, `GameInit`, `AI`, and `GameDone` on its thunk), and
**5 base stubs** whose vtable slots hold shared PC functions (`DuplicateFrom`, `Update`,
`SaveGameGetSize`, `SaveGameSave`, `SaveGameLoad`). The PS2 destructor is split in two on the PC
(§6b). The slots the class does not override are the `C_actor` defaults (`structs/actor.md` §3c).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00477300 | `crt_static_init_guard_00477300` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00477320 | `crt_atexit_register_00477320` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x00477330 | `clock_ctor` | instance | `__ct__7C_clockFv` | 0x001478c0 | CONFIRMED | `actor_ctor`, vtable `0x0063beb8`, kind `0x22` |
| 0x00477350 | `clock_scalar_deleting_dtor` | instance | `__dt__7C_clockFv` | 0x00147900 | STRUCTURAL | vtable slot 16; the `operator delete` arm of the split destructor (§6b) |
| 0x00477370 | `clock_dtor` | instance | `__dt__7C_clockFv` | 0x00147900 | CONFIRMED | restores the vtable and chains the `C_actor` destructor (§6b) |

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00477380 | `clock_init` | `Init__7C_clockFP9I3D_frame` | 0x00147960 | CONFIRMED | slot 18; `actor_init`, then the three by-name child lookups (§4); succeeds on `hand_hour && hand_minute` |
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | `DuplicateFrom__7C_clockFP7C_actor` | 0x00147a20 | base-stub | slot 21; a 5-byte `jmp` to the shared one-argument no-op; the PS2 body is an empty return. The same thunk fills `C_bottle`'s slot 21 (`structs/bottle.md` §3c) ¹ |
| 0x00477400 | `clock_game_init` | `GameInit__7C_clockFv` | 0x00147a30 | CONFIRMED | slot 25; zeroes the angles and `ms_accum`, `actor_game_init`, raises `+0x65`, seeds the angles from `GetLocalTime` and poses the hands (§5) |
| 0x00477540 | `thunk_actor_game_done` | `GameDone__7C_clockFv` | 0x00147bb0 | CONFIRMED | slot 26; a 5-byte `jmp` to `actor_game_done`, as the 8-byte PS2 body is a tail-jump to the `C_actor` `GameDone` ¹ |
| 0x00477550 | `clock_ai_tick` | `AI__7C_clockFUi` | 0x00147bc0 | CONFIRMED | slot 13; accumulates the frame delta, steps the minute (and on a wrap the hour) once per 60000 ms, poses the hands (§5) |
| 0x004776f0 | `stub_noop_args4_004776f0` | `Update__7C_clockFUi` | 0x00147dd0 | base-stub | slots 14, 15, 36, 37; the shared one-argument no-op of the `C_actor` family (§6a) |
| 0x00477700 | `stub_ret0_cdecl_00477700` | `SaveGameGetSize__7C_clockFv` | 0x00147de0 | base-stub | slots 22, 35; returns 0, so the savegame stores nothing for the clock (§6a) |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveGameSave__7C_clockFPPv` | 0x00147df0 | base-stub | slots 19, 20, 23, 24; returns true (§6a) |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveGameLoad__7C_clockFPPv` | 0x00147e00 | base-stub | the same function as the row above, slot 24 (§6a) |

¹ Ghidra names a thunk after its target: the Ghidra ids are `thunk_FUN_00461180` and
`thunk_FUN_0044bd90`.

### 3d. Free helpers

The translation unit holds no free helper.

## 4. The shipped assets are the label

`clock_init` resolves three child frames **by name**: `Mala`, `Velka`, `Sekundova`, the
**hour / minute / second hand**. The three literals are `s_clock_hand_hour`,
`s_clock_hand_minute` and `s_clock_hand_second`.

A census of all **3219** shipped models finds exactly three files carrying those frame names:

| model | `Mala` | `Velka` | `Sekundova` |
|---|---|---|---|
| `9hodinky.4ds` (*hodinky* = watch) | ✔ | ✔ | ✔ |
| `xhodiny 4.4ds` (*hodiny* = clock) | ✔ | ✔ | - |
| `rucky.4ds` (*rucky* = clock hands) | ✔ | ✔ | - |

**Where clocks are placed.** A walk of the object definitions of all 79 shipped `scene2.bin`
files finds four of kind `0x22`, and no other code creates one:

| scene | frame | model |
|---|---|---|
| `00MENU` | `9hodinky` | `9hodinky.i3d`: the watch in the main-menu scene |
| `MISE19-BANKA` | `xhodiny 1` | `xhodiny 4.i3d` |
| `MISE19-BANKA` | `xhodiny 1954` | `xhodiny 4.i3d` |
| `MISE19-BANKA` | `xhodiny 5442` | `xhodiny 4.i3d` |

No scene places a `rucky` clock.

The census **explains the code's shape**. `clock_init` returns success on `Mala && Velka` alone, and
both `clock_game_init` and `clock_ai_tick` wrap every second-hand operation in
`if (hand_second != 0)`, because two of the three shipped clock models have no `Sekundova` frame.
Body-reading alone does not give that; the label lives in the game's assets, the same lever as
`structs/car_extended.md` §4 (the `~` tag round-trip).

### 4a. The dial layout

In-game observation by the player-tester:

> *"every manual clock is 12h including that in game. Furthermore there is another small dial
> inside big (hour+minutes) dial. Smaller dial (one needle) is for seconds where whole round
> (like 12 h on big) is 60 s."*

The face is a **12-hour main dial carrying two hands** (`Mala` = hour, `Velka` = minute) plus a
**separate small sub-dial set inside it carrying a single needle** (`Sekundova`), one revolution of
which is **60 seconds**. There is no 24-hour dial anywhere in this game.

* **The second hand is a sub-dial, not a third hand on the main face.** That is why it is a separate
  frame with its own independent rotation, and why the cheaper models simply omit it: a sub-dial is
  a distinct piece of geometry.
* **`clock_ai_tick`'s second-hand arithmetic matches it.** `π − ms·2π/60000` is one full revolution
  per 60 000 ms = 60 s, the sub-dial described.
* **The dial period is 12 hours**, which is what §5a turns on.

## 5. Behaviour — the arithmetic

**`clock_game_init`** (vtable slot 25) zeroes `+0x80`, `+0x84`, `+0x7c` (in that order), chains
`actor_game_init`, raises `+0x65` (the `C_actor` save-exclusion byte), then calls the Win32
**`GetLocalTime`** and converts the real time of day into radians:

```
angle_hour   = wHour   * 2 * pi/12       // 30 deg per hour  -> a 12-hour dial
angle_minute = wMinute * 2 * pi/60       //  6 deg per minute
```

**`clock_ai_tick`** (vtable slot 13) accumulates elapsed milliseconds and, once a full minute
(`60000` ms) has passed:

```
angle_minute += pi/30                    //  6 deg per minute
if (angle_minute > 2*pi) { angle_minute -= 2*pi; angle_hour += pi/12; ... }
```

Both then pose the frames about the axis `(0, 1, 0)`:

```
hand_hour   <- pi - (angle_hour + angle_minute/12)     // the hour hand creeps with the minutes
hand_minute <- pi - angle_minute
hand_second <- pi - ms_accum * 2*pi/60000              // every tick; one revolution per 60 s
```

### 5a. The hour hand's per-hour step is half the 12-hour dial's

On a 12-hour dial the hour hand must travel **30° per hour**. The two halves of this class disagree
about that:

| | expression | per elapsed hour | right for a 12 h dial? |
|---|---|---|---|
| `clock_game_init` seed | `wHour × 2 × π/12` | 30° | yes |
| `clock_game_init` creep term | `+ angle_minute/12` | +30° across the hour | yes |
| `clock_ai_tick` step | `angle_hour += π/12` | **15°** | **no: half** |

The displayed angle is `π − (angle_hour + angle_minute/12)` in both functions, so the visible
behaviour is precise:

* **Within any given hour the hand is exactly right**: the `angle_minute/12` creep carries it
  smoothly through the full 30°.
* **At each hour boundary the creep term resets to 0 (−30°) while `angle_hour` gains only +15°**,
  so the hour hand **jumps 15° backwards** and is 15° behind from then on. After *n* whole hours in
  the same scene it lags `n × 15°`.

The PS2 build has the same arithmetic (§6d).

**Why it ships unnoticed:** `ms_accum` counts **real** milliseconds and `clock_game_init` re-seeds
from `GetLocalTime` on **every scene load**. The error is therefore exactly **zero for the first
hour of continuous play in one scene**, and a mission rarely runs that long. The minute hand and the
second sub-dial are right at all times.

## 6. PC vs PS2

### 6a. The PS2 stubs and the shared PC functions they fold onto

Six of the eleven PS2 methods are 8-byte bodies. A stub survives as its own PC function only when
it returns a *distinct constant*; the others fill their slots with functions the whole `C_actor`
family shares:

| PS2 stub | slot | PC function in the slot |
|---|---|---|
| `Update__7C_clockFUi` | 14 | `stub_noop_args4_004776f0` (225 datarefs) |
| *(no PS2 `NetDirect` for this class)* | 15 | `stub_noop_args4_004776f0` |
| `DuplicateFrom__7C_clockFP7C_actor` | 21 | `thunk_stub_noop_args4_00461180` |
| `SaveGameGetSize__7C_clockFv` | 22 | `stub_ret0_cdecl_00477700` |
| `SaveGameSave__7C_clockFPPv` | 23 | `stub_rettrue_args4_00477710` |
| `SaveGameLoad__7C_clockFPPv` | 24 | `stub_rettrue_args4_00477710` |
| `GameDone__7C_clockFv` | 26 | `thunk_actor_game_done` |

Slots 19 and 20 (`SaveData`, `LoadData`) hold `stub_rettrue_args4_00477710` too; the PS2 class
declares neither. Slot 21 = `DuplicateFrom` is confirmed independently in
`structs/car_extended.md`, whose slot-21 thunk maps to `DuplicateFrom__8C_car_exFP7C_actor`.

Size ratios of the three real twins: 0.58 (`Init`), 0.84 (`GameInit`), 0.71 (`AI`), all inside the
healthy band, the shortfall explained by MSVC constant-folding the π expressions the PS2 computes
at run time (§6d).

### 6b. The 1 → 2 destructor split

The PS2 fuses destructor and delete into one `__dt__(this, short)`; MSVC splits them. Both PC rows
therefore carry the **same twin**, and each is named after **its discriminator**: `clock_dtor` is
the vtable-restore and base-chain arm, `clock_scalar_deleting_dtor` is the arm that adds the
`operator delete` under the flag bit. The split arm is graded STRUCTURAL because no single body
agreement can be asserted for half of a fused method.

### 6c. Field bridge — a flat +0x10 on the class's own band

| PC | PS2 | Field |
|---|---|---|
| `+0x00` | `+0x78` | vtable |
| `+0x10` | `+0x50` | kind |
| `+0x65` | `+0x69` | save-exclusion byte |
| `+0x68` | `+0x70` | the actor frame |
| `+0x70` | `+0x80` | `hand_hour` |
| `+0x74` | `+0x84` | `hand_minute` |
| `+0x78` | `+0x88` | `hand_second` |
| `+0x7c` | `+0x8c` | `ms_accum` |
| `+0x80` | `+0x90` | `angle_hour` |
| `+0x84` | `+0x94` | `angle_minute` |

### 6d. Shared constants — and why a naive grep misses them

The three frame names are **byte-identical** across builds, read out of both binaries: PC
`0x0064b8c0` / `0x0064b8b8` / `0x0064b8ac`, PS2 `0x4ab4e0` / `0x4ab4e8` / `0x4ab4f0`, both laid out
`Sekundova`, `Velka`, `Mala`.

Every float in the PC bodies is the PS2 expression **constant-folded by MSVC**, so searching
Game.exe for the PS2's literals finds nothing:

| PC constant | PS2 spelling |
|---|---|
| `0.2617994` | `3.1415927 / 12.0` |
| `0.05235988` | `3.1415927 / 60.0` |
| `0.10471976` | (the per-minute step) |
| `0.083333336` | `/ 12.0` |
| `0.00010471976` | `(ms / 60000.0) * 2.0 * 3.1415927` |

`60000` and `6.2831855` are shared verbatim.

The §5a step is on both builds: the PS2 `GameInit__7C_clockFv` uses the same `<<1 … /12.0` and
`AI__7C_clockFUi` the same `0.2617994`.

### 6e. Frame-lookup vtable slot

The by-name child lookup is called through the actor frame's vtable: **PC `+0x38`**, PS2 `+0x50`.
Same call, same `(name, 1)` arguments, same three-in-a-row usage.

### 6f. Position and the twin levers

| lever | result |
|---|---|
| PC `.rdata` vtable order | `0x0063be18` (`C_cartridge`) → **`0x0063beb8`** → `0x0063bf68` (`C_crash_obj`) |
| link order by source file | `C_cartridge.cpp` → **`c_clock.cpp`** → `C_crash_obj.cpp` (the sort is case-insensitive) |
| kind byte | `__ct__7C_clockFv` writes `0x22` at PS2 `+0x50`, where the PC ctor writes it at `+0x10` |
| body agreement | `Init`, `GameInit` and `AI` are statement-for-statement identical across builds |
| `GameDone` | the 8-byte PS2 body is a tail-jump to the `C_actor` `GameDone`; the PC slot is a 5-byte `jmp` to `actor_game_done` |

Every §1 field is `[H]` because each one is written and read by a body that agrees
statement-for-statement with its PS2 twin (§6c).

## 7. Related

- `structs/actor.md` — `C_actor`, the base class, and the shared slot stubs.
- `structs/entity_kind.md` — kind `0x22`.
- `structs/bottle.md` — the other class whose slot 21 is `thunk_stub_noop_args4_00461180`.
- `structs/cartridge.md`, `structs/crash_obj.md` — the neighbouring classes in vtable and link
  order.

## 8. Open items

1. **The §5a hour-hand lag in game** is untested: stay in one scene for more than an hour of real
   time near a clock. After the first hour boundary the hour hand should sit 15° behind, and 30°
   after two.
