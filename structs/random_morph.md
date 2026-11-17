# `C_rmorf` — the random-morph tickable module

**Created** 2026-08-31 15:07 +0200 · **Modified** 2026-11-04 09:27 +0100

A small engine-level module with a three-slot vtable that animates **vertex-morph channels on named
scene models at random**. The per-mission file `Missions\<mission>\rmorf.bin` lists sets of frame
names; every period the module picks a random subset of each set's morph channels, gives them random
weights and eases the current weights towards them. The PS2 class is `C_rmorf` (*morf* = morph); the
PC name states the behaviour. Each loaded set is an `RMRF_S_SETREF` record (§1a), prefix
`random_morph_set_`.

| | |
|---|---|
| PC prefix | `random_morph_set_` (the set record), `random_morph_` (the module) |
| PC kind (`+0x10`) | - |
| PC vtable | `g_random_morph_vt` @ `0x0063eaec`, 3 slots (§4) |
| PC base class | - |
| PC instance size | `0x08` |
| PC constructor | `random_morph_ctor` @ `0x005ca4b0` |
| PC translation unit | `[0x005ca290, 0x005cad40)`: 17 starts and one catch funclet, all named; the next unit's guard `crt_static_init_guard_005cad40` opens `C_shvestky` |
| PS2 class | `C_rmorf` (tag `7C_rmorf`), 5 methods, and its set record `RMRF_S_SETREF` (tag `13RMRF_S_SETREF`), 2 methods; all mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_rmorf {                    // 0x08 bytes
    /* +0x00 */ void          **vtbl;       // [H] g_random_morph_vt, 3 slots
    /* +0x04 */ RMRF_S_SETREF  *sets;       // [H] head of the loaded-set list, 0 when idle
} C_rmorf;
```

**Two instances, one of them live.** A byte scan of `.text` and a census of every `lea ecx,[reg +
0x2bc]` give a closed set of references:

| Instance | References |
|---|---|
| `C_game + 0x2bc` | `game_ctor` (`random_morph_ctor`), `game_init` (slot 1), `game_tick` (slot 0, every frame, with the frame delta), `game_done` (slot 2), `game_dtor` (`random_morph_dtor`) |
| the file-scope object at `0x006efdd0` | `random_morph_static_ctor` and `random_morph_static_dtor` only |

So the module that runs is the `C_game` member; the static object is constructed and destroyed and
nothing else addresses it.

### 1a. `RMRF_S_SETREF` — one loaded set, `0x430` bytes

```c
typedef struct RMRF_S_SETREF {              // 0x430 bytes
    /* +0x000 */ int32_t        frame_count;      // [H] N frames driven together
    /* +0x004 */ I3D_frame    **frames;           // [H] array of N resolved scene frames
    /* +0x008 */ RMRF_S_SETREF *next;             // [H] the singly-linked list
    /* +0x00c */ uint32_t       channel_count;    // [H] morph channels per row
    /* +0x010 */ uint32_t       row_count;        // [H] rows of channels
    /* +0x014 */ uint32_t       timer_ms;         // [H] the frame-delta accumulator
    /* +0x018 */ uint32_t       period_ms;        // [H] the current period; re-drawn each cycle
    /* +0x01c */ float          weight[16][8];    // [H] the current channel weights
    /* +0x21c */ float          delta[16][8];     // [H] the per-period increment towards the target
    /* +0x41c */ int32_t        mode;             // [H] low byte < 0 = cycle; 1 = eased draw; else linear draw (§5b)
    /* +0x420 */ int32_t        period_base_ms;   // [H] default 1000
    /* +0x424 */ int32_t        period_spread_ms; // [H] default 1; rand() % this is added
    /* +0x428 */ int32_t        pick_base;        // [H] default 1; channels chosen per period
    /* +0x42c */ int32_t        pick_spread;      // [H] default 1; rand() % this is added
} RMRF_S_SETREF;
```

**Why 16 × 8.** The record stores one weight and one delta per (row, channel) pair in static arrays of
16 rows of 8 channels. `random_morph_set_load_raw` takes `channel_count` from the first frame's morph
interface (`I3D_frame + 0x140`, vtable `+0x00`) and `row_count` as the largest value any frame's
interface reports at vtable `+0x04`; it stores both without a clamp. `random_morph_tick` walks
`row_count` rows of `channel_count` channels, keeps an 8-entry per-channel mark array and a 128-float
(16 × 8) scratch copy on its stack, and pushes each weight through vtable `+0x08` as
`(row, channel, weight)`. The array sizes are therefore the capacity the developers allowed; no
instruction checks a model against them.

The list is singly linked: `random_morph_set_load_raw` prepends each new record (`next` = the old
head) and `random_morph_set_destroy` recurses on `next` rather than indexing.

## 2. Field evidence

Measured by body reads of every function of the unit.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtbl | H | `random_morph_ctor`, `random_morph_dtor` | `game_tick`, `game_init`, `game_done` (the three slots, §4) | - |
| +0x04 | sets | H | `random_morph_ctor` (0), `random_morph_mission_start` (through `random_morph_set_load_raw`, which prepends), `random_morph_mission_done` and `random_morph_dtor` (0) | `random_morph_tick`, `random_morph_mission_done`, `random_morph_dtor` | the list head |

### 2a. `RMRF_S_SETREF`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | frame_count | H | `random_morph_set_load_raw` (from the file); 0 on teardown by `random_morph_set_destroy`, `random_morph_mission_done`, `random_morph_dtor` | `random_morph_set_load_raw`, `random_morph_tick` | - |
| +0x004 | frames | H | `random_morph_set_load_raw` (`operator_new(N * 4)`); freed by the three teardown paths | `random_morph_tick` | each entry found by name through `*(g_act_mission + 0x10)` vtable `+0x58` |
| +0x008 | next | H | `random_morph_set_load_raw` (the old head); 0 on teardown | `random_morph_tick`, the teardown paths | - |
| +0x00c | channel_count | H | `random_morph_set_load_raw` | `random_morph_tick` | the inner loop bound |
| +0x010 | row_count | H | `random_morph_set_load_raw` | `random_morph_tick` | the outer loop bound |
| +0x014 | timer_ms | H | `random_morph_set_load_raw` (0), `random_morph_tick` | `random_morph_tick` | wraps modulo `period_ms` |
| +0x018 | period_ms | H | `random_morph_set_load_raw` (0), `random_morph_tick` | `random_morph_tick` | `period_base_ms + rand() % period_spread_ms` |
| +0x01c | weight | H | `random_morph_set_load_raw` (channel 0 of every row = 1.0f, the rest 0), `random_morph_tick` | `random_morph_tick` | - |
| +0x21c | delta | H | `random_morph_set_load_raw` (0), `random_morph_tick` | `random_morph_tick` | - |
| +0x41c | mode | H | `random_morph_set_load_raw` (from the file) | `random_morph_tick` | §5b |
| +0x420 | period_base_ms | H | `random_morph_set_load_raw` (1000, then the file) | `random_morph_tick` | - |
| +0x424 | period_spread_ms | H | `random_morph_set_load_raw` (1, then the file) | `random_morph_tick` | used only when above 1 |
| +0x428 | pick_base | H | `random_morph_set_load_raw` (1, then the file) | `random_morph_tick` | a pick of 0 becomes 1; capped at `channel_count` |
| +0x42c | pick_spread | H | `random_morph_set_load_raw` (1, then the file) | `random_morph_tick` | used only when above 1 |

## 3. Method family

Coverage: all **5** PS2 `7C_rmorf` methods and both `13RMRF_S_SETREF` methods are CONFIRMED twins.
The unit also emits four MSVC `basic_string` members (§3d) and its CRT scaffolding.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005ca290 | `crt_static_init_guard_005ca290` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x005ca2b0 | `crt_atexit_register_005ca2b0` | the translation unit | - | - | - | registers `crt_atexit_noop_dtor_0060b700` |
| 0x005ca2c0 | `random_morph_static_init` | the object at `0x006efdd0` | - | - | - | the `.CRT$XCU` entry: the ctor, then the `atexit` registration |
| 0x005ca2d0 | `random_morph_static_ctor` | the object at `0x006efdd0` | - | - | - | `mov ecx, 0x6efdd0; jmp random_morph_ctor` |
| 0x005ca2e0 | `random_morph_register_atexit_dtor` | the object at `0x006efdd0` | - | - | - | `atexit` of the static dtor |
| 0x005ca2f0 | `random_morph_static_dtor` | the object at `0x006efdd0` | - | - | - | `random_morph_dtor` on the static object |
| 0x005ca4b0 | `random_morph_ctor` | instance | `__ct__7C_rmorfFv` | 0x001f8c70 | CONFIRMED | installs the vtable, clears `sets`; called by `random_morph_static_ctor` and `game_ctor` |
| 0x005ca4c0 | `random_morph_dtor` | instance | `__dt__7C_rmorfFv` | 0x001f8c90 | CONFIRMED | restores the vtable, tears down the head record (one level inlined) and recurses on its tail |
| 0x005ca510 | `random_morph_set_destroy` | a set record | `__dt__13RMRF_S_SETREFFv` | 0x001f8d40 | CONFIRMED | frees `frames`, recurses on `next`, deletes itself on the flag bit |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005ca300 | `random_morph_set_load_raw` | `LoadRAWSet__13RMRF_S_SETREFFPP13RMRF_S_SETREFPc` | 0x001f89f0 | CONFIRMED | a static member: allocates and seeds one record, prepends it to the list whose head the caller passes, resolves its frames and returns the advanced file cursor (§5) |
| 0x005ca560 | `random_morph_mission_start` | `Mission_Start__7C_rmorfFv` | 0x001f8e00 | CONFIRMED | slot 1; formats `Missions\%s\rmorf.bin`, checks the header and loads every record (§5) |
| 0x005ca790 | `random_morph_mission_done` | `Mission_Done__7C_rmorfFv` | 0x001f8ff0 | CONFIRMED | slot 2; tears down the whole list (head inlined, then `random_morph_set_destroy`) |
| 0x005ca7e0 | `random_morph_tick` | `Tick__7C_rmorfFi` | 0x001f9070 | CONFIRMED | slot 0; per set: advance the timer, re-draw the period and the targets, ease the weights and push them to every frame (§5b) |

### 3d. Free helpers

The four MSVC `basic_string<char>` members emitted into this object file because this is where they
were first used; they are not members of either class, and `_tidy` and `_grow_buffer` have callers in
the CRT band at `0x0062xxxx`.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005ca680 | `std_string_ctor_formatted` | `__ct__7fstringFPCce` | 0x001f8f60 | CONFIRMED | formats into a 1000-byte stack buffer and assigns it to a string; its only caller is `random_morph_mission_start` (§6) |
| 0x005caba0 | `std_string_dtor` | `__dt__Q23std59basic_string<>Fv` | 0x00104320 | STRUCTURAL | reached only from three unwind funclets; the two builds ship different Standard Libraries |
| 0x005cabe0 | `std_string_tidy` | - | - | - | 12 callsites, mostly in the CRT band |
| 0x005cac30 | `std_string_grow_buffer` | - | - | - | 4 callsites; its true extent is `[0x005cac30, 0x005cad40)` ¹ |

¹ `0x005cacb0` inside it is not a function start: it is reached by an unconditional `jmp 0x5cacb0`
0x29 bytes earlier and by the catch-resume fall-through at `0x005cacaa`, with zero `E8` callers and
zero datarefs. `Catch_005cac89` is a real EH catch funclet, with a dataref in `.rdata:0x006434d4`
(the MSVC FuncInfo table).

## 4. The vtable — `g_random_morph_vt`, three slots

Read from the vtable order in `.rdata`:

| Slot | PC VA | PC name | PS2 twin |
|---|---|---|---|
| 0 | 0x005ca7e0 | `random_morph_tick` | `Tick__7C_rmorfFi` |
| 1 | 0x005ca560 | `random_morph_mission_start` | `Mission_Start__7C_rmorfFv` |
| 2 | 0x005ca790 | `random_morph_mission_done` | `Mission_Done__7C_rmorfFv` |

The dword after the table (`DAT_0063eaf8`) is **not** a fourth slot: it is zero, and
`random_morph_mission_start` takes its address as an empty C string when the formatted path comes
back empty. It is not the project's `g_empty_string` (`0x00655214`). Slots 1 and 2 are the only
references to those two functions anywhere in the image; they have no direct caller.

## 5. The `rmorf.bin` file format

`Missions\<mission>\rmorf.bin`, opened through the `.dta` layer.

| Offset | Size | Field | Evidence |
|---|---|---|---|
| +0x00 | 4 | total file size in bytes, header included | `random_morph_mission_start` |
| +0x04 | 4 | version, **float `0.1`**; anything else is rejected | `random_morph_mission_start` |
| +0x08 | 4 | record count | `random_morph_mission_start` |
| +0x0c | size − 0x0c | the payload, read into one heap buffer and walked once | `random_morph_set_load_raw` |

Each record in the payload is `{ int frame_count; int mode; int period_base; int period_spread;
int pick_base; int pick_spread; char name[] * frame_count; }`: six ints, then the frame names.
`random_morph_set_load_raw` copies the six ints to the set record (`+0x00` count, `+0x41c` mode,
`+0x420` / `+0x424` period, `+0x428` / `+0x42c` pick) and then walks the names, which are
NUL-terminated and variable length; that is why it returns the advanced cursor rather than taking an
index, and `random_morph_mission_start` passes the returned cursor straight into the next call.

**Round trip:** every shipped file parses exactly under this layout and ends with one zero byte per
record that the header's size counts and the loader never reads.

### 5a. The shipped files

Twelve missions ship an `rmorf.bin`. One entry per record, as frames · mode · period base / spread ·
pick base / spread:

| Mission | Records |
|---|---|
| `MISE02-ULICKA` | 9 · 0 · 1000 / 1 · 1 / 1 |
| `MISE04-MOTOREST` | 1 · 0 · 1000 / 201 · 1 / 1; 1 · 0 · 800 / 601 · 1 / 1 |
| `MISE06-AUTODROM` | 4 · 128 · 100 / 1 · 1 / 1 |
| `MISE07-SARA` | 3 · 128 · 1000 / 1 · 1 / 1 |
| `MISE09-KRAJINA` | 1 · 0 · 150 / 51 · 0 / 1 |
| `MISE09-PREJIMKA` | 1 · 0 · 100 / 301 · 1 / 1; 1 · 128 · 200 / 601 · 1 / 1; 2 · 0 · 400 / 1001 · 1 / 1; 8 · 0 · 800 / 401 · 1 / 1; 6 · 0 · 500 / 1001 · 1 / 1 |
| `MISE10-LETISTE` | 2 · 128 · 160 / 1 · 1 / 1 |
| `MISE13-ZRADCE` | 9 · 129 · 1000 / 1001 · 1 / 1 |
| `MISE14-PARNIK` | 1 · 128 · 170 / 1 · 1 / 1; 1 · 128 · 200 / 1 · 1 / 1 |
| `MISE15-PRISTAV` | 1 · 1 · 2000 / 1 · 1 / 1 |
| `MISE16-LETISTE` | 2 · 128 · 160 / 1 · 1 / 1 |
| `MISE18-PRISTAV` | 1 · 128 · 1000 / 1 · 1 / 1 |

The two MISE09 files name what they drive. `MISE09-KRAJINA`: `Truck.Mesh00`. `MISE09-PREJIMKA`: two
laundry frames (`pradlo mor2.Rectangle02`, `pradlo mor.Rectangle02`, one per record), two juniper
frames (`s_jalovec00.Box14`, `s_jalovec.Box14`) and 14 tree frames (`s_oranz*` / `s_barevny*`,
8 + 6), which the same mission's `scene2.bin` places.

### 5b. One period of `random_morph_tick`

Per set, `timer_ms` accumulates the frame delta. When it reaches `period_ms` it wraps, a new period
`period_base_ms + rand() % period_spread_ms` is drawn, every weight takes its full delta
(`weight += delta`), and the set picks its next deltas in one of two arms, selected by the low byte
of `mode`:

| Arm | Modes | New deltas |
|---|---|---|
| draw | low byte `>= 0` (0, 1) | choose `pick_base + rand() % pick_spread` channels per row (at least 1, at most `channel_count`), taking channels whose weight is zero first and active ones only to fill the count; give each a random integer 1..10, normalise by their sum, and set `delta = target − weight` (unchosen channels head to 0) |
| cycle | low byte `< 0` (128, 129) | `delta[k+1] = weight[k] − weight[k+1]` and `delta[0] = weight[last] − weight[0]`, so the weight pattern moves one channel along per period with no new draw |

Every frame, wrap or not, the tick computes `s = timer_ms / period_ms`, replaces it with
`(1 − cos(π · s)) / 2` when the whole dword `mode == 1`, and pushes `weight + s · delta` into every
frame's morph interface through `I3D_frame + 0x140`, vtable `+0x04` (that frame's row count) and
`+0x08` (set row *i*, channel *j*). So mode 0 ramps linearly, mode 1 eases, and 128 and 129 cycle
with a linear ramp. No string, symbol or script of either build names the modes, so what a designer
called them is not recoverable.

## 6. PC vs PS2

* **Source file.** The PS2 ELF names it `C_rmorf.cpp` in its `__sinit_` manifest, and the object's
  own string is `randomMorfTickableModule`. The PC has no such string; the class was identified from
  the shipped `Missions\%s\rmorf.bin` path, which both builds share verbatim.
* **Link order.** The PS2 emits `C_rmorf.cpp` immediately before `C_shvestky`
  (`0x001f8c70` → `0x001f99b0`); the PC does the same (`0x005ca290` → `0x005cad40`). `C_rmorf.cpp`
  sorts before `C_shvestky.cpp`, so this is the alphabetical-by-source-file link order agreeing
  across both toolchains.
* **Record layout is identical.** `+0x1c` weights, `+0x21c` deltas, `+0x41c` mode, `+0x420` /
  `+0x424` period, `+0x428` / `+0x42c` pick: the same offsets on both builds, which made the `Tick`
  diff decisive.
* **vptr placement differs:** the PS2 puts the vtable at `+0x04` and the list head at `+0x00`; the PC
  does the reverse. Same two fields.
* **`fstring`.** The PS2 wraps "format into a string" in a class it calls `fstring`, deriving from
  `std::basic_string<char>`; MSVC inlines the same three calls (default ctor, `vsprintf`, `assign`)
  into one function. Both use a **1000-byte** stack scratch buffer, the shared constant that made the
  match falsifiable.
* **The morph interface offset:** the PC reaches the channel interface through `I3D_frame + 0x140`;
  the PS2 offset is not measured.

## 7. Related

- `structs/game.md` — the owning `C_game` (`+0x2bc`) and its `game_init` / `game_tick` /
  `game_done`.
- `structs/i3d_frame.md` — `I3D_frame`, whose `+0x140` interface carries the morph channels.
- `structs/police_manager.md` — `C_shvestky`, the next unit.

## 8. Open items

1. **What morphs in game has not been observed.** The MISE09 files name laundry, juniper, tree and
   truck frames (§5a); the other ten files are not matched to their models. **In game:** load a
   mission with an `rmorf.bin` and look for a model that deforms on a ~1 s random cycle; a mode-128
   set should show its pattern stepping along rather than re-drawing.
2. **The PS2 offset of the morph interface** (`I3D_frame + 0x140` on the PC) is not measured.
