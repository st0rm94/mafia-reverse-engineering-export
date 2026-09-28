# `C_clock` — the working clock actor (PC prefix `clock_`)

**Created** 2026-08-12 20:08 +0200 · **Modified** 2026-09-17 13:45 +0200

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x22`** |
| PC vtable | `.rdata 0x0063beb8` (39 slots) |
| PC constructor | `clock_ctor` @ `0x00477330` |
| PC instance size | **`0x88`** (entity factory `case 0x22`) |
| PC translation unit | `[0x00477330, 0x004776c0)` — 8 real function starts, all named |
| Base class | `C_actor` (`structs/actor.md`) |
| PS2 class | `C_clock`, 11 methods, all accounted for (§5a) |

The prefix `clock_` maps the PS2 class tag `7C_clock` onto a lowercase subsystem prefix. No existing
prefix is a proper prefix of it, and it is a proper prefix of none.

## 1. What the class is

An actor whose model is a clock: it resolves three named child frames as clock hands, seeds them
from the **machine's real wall clock** when the scene starts, and rotates them forward once per
elapsed REAL minute (`clock_ai_tick`'s argument is the frame delta in ms). It is one of the smallest
complete classes in the binary — five methods and one destructor split — with a clean PC↔PS2 twin
set.

Position is over-determined:

| lever | result |
|---|---|
| PC `.rdata` vtable order | `0x0063be18` (`C_cartridge`) → **`0x0063beb8`** → `0x0063bf68` (`C_crash_obj`) |
| link order by source file | `C_cartridge.cpp` → **`c_clock.cpp`** → `C_crash_obj.cpp` (the sort is case-insensitive) |
| kind byte | PC ctor writes `0x22` at `+0x10`; `__ct__7C_clockFv` writes `0x22` at PS2 `+0x50` |
| the entity factory | `case 0x22: operator_new(0x88)` — and the class's last field ends at exactly `0x88` |
| body agreement | `Init`, `GameInit` and `AI` are statement-for-statement identical across builds |

## 2. The shipped assets are the label

`clock_init` resolves three child frames **by name**: `Mala`, `Velka`, `Sekundova` — Czech
*malá / velká / sekundová ručička* = **hour / minute / second hand**.

A census of all **3219** shipped models finds exactly three files carrying those frame names:

| model | `Mala` | `Velka` | `Sekundova` |
|---|---|---|---|
| `9hodinky.4ds` (*hodinky* = watch) | ✔ | ✔ | ✔ |
| `xhodiny 4.4ds` (*hodiny* = clock) | ✔ | ✔ | - |
| `rucky.4ds` (*ručky* = clock hands) | ✔ | ✔ | - |

The census **explains the code's shape**. `clock_init` returns success on `Mala && Velka` alone, and
both `clock_game_init` and `clock_ai_tick` wrap every second-hand operation in `if (+0x78 != 0)`,
because two of the three shipped clock models have no `Sekundova` frame. Body-reading alone does not
give that; the label lives in the game's assets — the same lever as `car_extended.md` §1a (the `~`
tag round-trip).

### 2a. The dial layout

In-game observation by the player-tester:

> *"every manual clock is 12h including that in game. Furthermore there is another small dial
> inside big (hour+minutes) dial. Smaller dial (one needle) is for seconds where whole round
> (like 12 h on big) is 60 s."*

The face is a **12-hour main dial carrying two hands** (`Mala` = hour, `Velka` = minute) plus a
**separate small sub-dial set inside it carrying a single needle** (`Sekundova`), one revolution of
which is **60 seconds**. There is no 24-hour dial anywhere in this game.

* **The second hand is a sub-dial, not a third hand on the main face.** That is why it is a separate
  frame with its own independent rotation, and why the cheaper models simply omit it — a sub-dial is
  a distinct piece of geometry.
* **`clock_ai_tick`'s second-hand maths is exactly right.** `π − ms·2π/60000` is one full revolution
  per 60 000 ms = 60 s, precisely the sub-dial described.
* **The dial period is 12 hours**, which is what §4a turns on.

## 3. Field layout (PC)

```c
struct C_clock /* : C_actor, 0x88 bytes */ {
    /* +0x00 */ void      *vtable;         // 0x0063beb8
    /* +0x10 */ uint32_t   kind;           // 0x22
    /* +0x65 */ char       active;         // [H] raised by clock_game_init
    /* +0x68 */ I3D_frame *frame;          // [H] C_actor's own frame — the lookup root
    /* +0x70 */ I3D_frame *hand_hour;      // [H] child frame "Mala"       — REQUIRED
    /* +0x74 */ I3D_frame *hand_minute;    // [H] child frame "Velka"      — REQUIRED
    /* +0x78 */ I3D_frame *hand_second;    // [H] child frame "Sekundova" — the needle of the
                                           //     SECONDS SUB-DIAL (§2a), one revolution = 60 s.
                                           //     OPTIONAL: 2 of the 3 shipped models omit it.
    /* +0x7c */ uint32_t   ms_accum;       // [H] milliseconds inside the current minute
    /* +0x80 */ float      angle_hour;     // [H] radians
    /* +0x84 */ float      angle_minute;   // [H] radians
};
```

Every field is `[H]`: each one is written and read by a body that agrees statement-for-statement
with its PS2 twin (§5c).

## 4. Behaviour — the arithmetic, and one defect

**`clock_game_init`** (vtable slot 25) zeroes `+0x80`, `+0x84`, `+0x7c` (in that order), chains
`actor_game_init`, raises `+0x65`, then calls the Win32 **`GetLocalTime`** and converts the real
time of day into radians:

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
                                                      // = the sub-dial of §2a
```

### 4a. The hour hand's per-hour step is half the 12-hour dial's — identical on both builds

On a 12-hour dial the hour hand must travel **30° per hour**. The two halves of this class disagree
about that:

| | expression | per elapsed hour | correct for a 12 h dial? |
|---|---|---|---|
| `clock_game_init` seed | `wHour × 2 × π/12` | 30° | yes |
| `clock_game_init` creep term | `+ angle_minute/12` | +30° across the hour | yes |
| `clock_ai_tick` step | `angle_hour += π/12` | **15°** | **no — half** |

The displayed angle is `π − (angle_hour + angle_minute/12)` in both functions, so the visible
behaviour is precise:

* **Within any given hour the hand is exactly right** — the `angle_minute/12` creep carries it
  smoothly through the full 30°.
* **At each hour boundary the creep term resets to 0 (−30°) while `angle_hour` gains only +15°**,
  so the hour hand **jumps 15° backwards** and is 15° behind from then on. After *n* whole hours in
  the same scene it lags `n × 15°`.

**This is original behaviour, not a PC artifact** — both builds make the same decision (§5d).

**Why it ships unnoticed:** `ms_accum` counts **real** milliseconds and `clock_game_init` re-seeds
from `GetLocalTime` on **every scene load**. The error is therefore exactly **zero for the first
hour of continuous play in one scene**, and a mission rarely runs that long. The minute hand and the
second sub-dial are correct at all times.

## 5. PC vs PS2

### 5a. Method coverage — 11/11 PS2 methods accounted

| PC VA | PC name | slot | PS2 twin | PS2 size | grade |
|---|---|---|---|---|---|
| 0x00477330 | `clock_ctor` | - | `__ct__7C_clockFv` | 60 | CONFIRMED |
| 0x00477350 | `clock_scalar_deleting_dtor` | 16 | `__dt__7C_clockFv` | 96 | STRUCTURAL |
| 0x00477370 | `clock_dtor` | - | `__dt__7C_clockFv` | 96 | CONFIRMED |
| 0x00477380 | `clock_init` | 18 | `Init__7C_clockFP9I3D_frame` | 192 | CONFIRMED |
| 0x004773f0 | `thunk_stub_noop_args4_00461180` | 21 | `DuplicateFrom__7C_clockFP7C_actor` | 8 | STRUCTURAL ¹ |
| 0x00477400 | `clock_game_init` | 25 | `GameInit__7C_clockFv` | 380 | CONFIRMED |
| 0x00477540 | `thunk_actor_game_done` | 26 | `GameDone__7C_clockFv` | 8 | STRUCTURAL |
| 0x00477550 | `clock_ai_tick` | 13 | `AI__7C_clockFUi` | 516 | CONFIRMED |

¹ A shared stub filling the slot, not a class-specific body. Slot 21 = `DuplicateFrom` is confirmed
independently in `structs/car_extended.md`, whose slot-21 thunk maps to
`DuplicateFrom__8C_car_exFP7C_actor`.

Size ratios: 0.58 (`Init`), 0.84 (`GameInit`), 0.71 (`AI`) — all inside the healthy band, the
shortfall entirely explained by MSVC constant-folding the π expressions PS2 computes at run time.

**The six 8-byte PS2 stubs** fold onto shared PC symbols; a stub survives as its own PC function only
when it returns a *distinct constant*:

| PS2 stub | slot | PC symbol it folded onto |
|---|---|---|
| `Update__7C_clockFUi` | 14 | `stub_noop_args4_004776f0` (225 datarefs) |
| *(no PS2 `NetDirect` for this class)* | 15 | `stub_noop_args4_004776f0` |
| `DuplicateFrom__7C_clockFP7C_actor` | 21 | `thunk_stub_noop_args4_00461180` |
| `SaveGameGetSize__7C_clockFv` | 22 | `stub_ret0_cdecl_00477700` |
| `SaveGameSave__7C_clockFPPv` | 23 | `stub_rettrue_args4_00477710` |
| `SaveGameLoad__7C_clockFPPv` | 24 | `stub_rettrue_args4_00477710` |
| `GameDone__7C_clockFv` | 26 | `thunk_actor_game_done` |

### 5b. The 1 → 2 destructor split

PS2 fuses destructor and delete into one `__dt__(this, short)`; MSVC splits them. Both PC rows
therefore carry the **same twin**, and each is named after **its discriminator**: `clock_dtor` is
the vtable-restore + base-chain arm, `clock_scalar_deleting_dtor` is the arm that adds the
`operator delete` under the flag bit. The split arm is graded STRUCTURAL because no single body
agreement can be asserted for half of a fused method.

### 5c. Field bridge — a flat +0x10 on the class's own band

| PC | PS2 | | PC | PS2 |
|---|---|---|---|---|
| `+0x00` vtable | `+0x78` | | `+0x74` minute-hand frame | `+0x84` |
| `+0x10` kind | `+0x50` | | `+0x78` second-hand frame | `+0x88` |
| `+0x65` active | `+0x69` | | `+0x7c` ms accumulator | `+0x8c` |
| `+0x68` actor frame | `+0x70` | | `+0x80` hour angle | `+0x90` |
| `+0x70` hour-hand frame | `+0x80` | | `+0x84` minute angle | `+0x94` |

### 5d. Shared constants — and why a naive grep misses them

The three frame names are **byte-identical** across builds, read out of both binaries: PC
`0x0064b8c0` / `0x0064b8b8` / `0x0064b8ac`, PS2 `0x4ab4e0` / `0x4ab4e8` / `0x4ab4f0`, both laid out
`Sekundova`, `Velka`, `Mala`.

Every float in the PC bodies is the PS2 expression **constant-folded by MSVC**, so searching
Game.exe for PS2's literals finds nothing:

| PC constant | PS2 spelling |
|---|---|
| `0.2617994` | `3.1415927 / 12.0` |
| `0.05235988` | `3.1415927 / 60.0` |
| `0.10471976` | (the per-minute step) |
| `0.083333336` | `/ 12.0` |
| `0.00010471976` | `(ms / 60000.0) * 2.0 * 3.1415927` |

`60000` and `6.2831855` are shared verbatim.

The §4a defect is on both builds: PS2 `GameInit__7C_clockFv` uses the same `<<1 … /12.0` and
`AI__7C_clockFUi` the same `0.2617994`.

### 5e. Frame-lookup vtable slot

The by-name child lookup is called through the actor frame's vtable: **PC `+0x38`**, PS2 `+0x50`.
Same call, same `(name, 1)` arguments, same three-in-a-row usage.

## 6. Open items

| item | state |
|---|---|
| the §4a hour-hand lag in game | untested: stay in one scene for more than an hour of real time near a clock — after the first hour boundary the hour hand should sit 15° behind, and 30° after two |
| string-global names | inconsistent: `s_clock_hand_hour` (`0x0064b8c0`) sits beside Ghidra's own `s_Velka_0064b8b8` and `s_Sekundova_0064b8ac` |
| where clocks are placed | nothing beyond the generic entity factory creates a `C_clock`; which missions instantiate the three models is unmapped |
