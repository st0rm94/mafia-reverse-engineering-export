# `C_rmorf` — the random-morph tickable module

**Created** 2026-08-31 15:07 +0200 · **Modified** 2026-10-07 21:57 +0200

The translation unit `[0x005ca290, 0x005cad40)`, the second of the three translation units in
`[0x005c9720, 0x005d14a0)`.

PC prefix **`random_morph_`** (the record class is `random_morph_set_`). The PS2 class is
`C_rmorf` (§7) — "morf" is Czech-flavoured shorthand for *morph*, which is what the class
actually does, so the PC name states the behaviour rather than transliterating the
abbreviation.

---

## 1. What it is

A **tickable module**: a small engine-level object with a three-slot vtable that the frame
loop and the mission lifecycle drive. It animates **vertex-morph channels on named scene
models at random** — the per-mission file `Missions\<mission>\rmorf.bin` lists sets of model
names, and every so often the module picks a random subset of each set's morph channels,
gives them random weights, and eases the current weights towards them.

There are **two classes in the translation unit**:

| class | PC prefix | what it is |
|---|---|---|
| `C_rmorf` | `random_morph_` | the module singleton; owns the list, ticks it |
| `RMRF_S_SETREF` | `random_morph_set_` | one loaded *set*: N frames driven together, with their own timing and weights |

`RMRF_S_SETREF` records form a **singly-linked list**, not a vector — proven by
`random_morph_set_destroy` recursing on `+0x08` rather than indexing.

---

## 2. `C_rmorf` — field layout

```c
typedef struct C_rmorf {           /* 0x08 bytes */
/* +0x00 */ void**   vtbl;         // [H] 3 slots, g_random_morph_vt
/* +0x04 */ RMRF_S_SETREF* sets;   // [H] head of the loaded-set list, 0 when idle
} C_rmorf;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtbl | H | `random_morph_ctor`, `random_morph_dtor` | the frame loop / mission lifecycle | 3 slots; see §3 |
| +0x04 | sets | H | `random_morph_set_load_raw` (prepend), `random_morph_mission_done` / `_dtor` (clear) | `random_morph_tick` | list head |

The object is a **file-scope static**, constructed by the `.CRT$XCU` initializer at
`0x005ca2c0` and destroyed through `atexit`. One further external caller constructs a
`C_rmorf` as well — the tickable-module registry — which is why `random_morph_ctor` has two
callers rather than one.

---

## 3. The vtable — `g_random_morph_vt`, three slots

Read from the vtable order in `.rdata`:

| slot | PC function | role |
|---|---|---|
| 0 | `random_morph_tick` | per-frame advance, takes the delta in ms |
| 1 | `random_morph_mission_start` | load `rmorf.bin` for the mission that is starting |
| 2 | `random_morph_mission_done` | drop every loaded set |

The dword immediately after the table (`DAT_0063eaf8`) is **not** a fourth slot: it is
zero, and `random_morph_mission_start` takes its address as an empty C string when the
formatted path comes back empty. It is NOT the project's `g_empty_string` (`0x00655214`).

Slots 1 and 2 are the only references to those two functions anywhere in the image — they
have no direct caller.

---

## 4. `RMRF_S_SETREF` — field layout (0x430 bytes)

```c
typedef struct RMRF_S_SETREF {     /* 0x430 bytes */
/* +0x000 */ int      frame_count;      // [H] N models driven together
/* +0x004 */ I3D_frame** frames;        // [H] array of N resolved scene frames
/* +0x008 */ RMRF_S_SETREF* next;       // [H] singly-linked list
/* +0x00c */ uint     channel_count;    // [H] morph channels per row (<= 8)
/* +0x010 */ uint     row_count;        // [H] rows / morph targets (<= 16)
/* +0x014 */ uint     timer_ms;         // [H] accumulates the frame delta
/* +0x018 */ uint     period_ms;        // [H] current period; re-drawn each cycle
/* +0x01c */ float    weight[16][8];    // [H] current channel weights
/* +0x21c */ float    delta[16][8];     // [H] per-cycle increment towards the target
/* +0x41c */ int      mode;             // [H] 1 = cosine ease, low byte <0 = additive, else linear
/* +0x420 */ int      period_base_ms;   // [H] default 1000
/* +0x424 */ int      period_spread_ms; // [H] default 1; rand() % this is added
/* +0x428 */ int      pick_base;        // [H] default 1; channels chosen per cycle
/* +0x42c */ int      pick_spread;      // [H] default 1; rand() % this is added
} RMRF_S_SETREF;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | frame_count | H | `random_morph_set_load_raw` | `random_morph_tick` | read from the file |
| +0x004 | frames | H | `random_morph_set_load_raw` | `random_morph_tick`, `random_morph_set_destroy` | `operator_new(N*4)` |
| +0x008 | next | H | `random_morph_set_load_raw` | `random_morph_set_destroy`, `random_morph_tick` | list link |
| +0x00c | channel_count | H | `random_morph_set_load_raw` (morph iface slot 0) | `random_morph_tick` | inner loop bound |
| +0x010 | row_count | H | `random_morph_set_load_raw` (max of morph iface slot 4) | `random_morph_tick` | outer loop bound |
| +0x014 | timer_ms | H | `random_morph_tick` | `random_morph_tick` | wraps modulo `period_ms` |
| +0x018 | period_ms | H | `random_morph_tick` | `random_morph_tick` | re-drawn every cycle |
| +0x01c | weight | H | `random_morph_set_load_raw` (seed), `random_morph_tick` | `random_morph_tick` | row 0 of each set seeded to 1.0f |
| +0x21c | delta | H | `random_morph_set_load_raw` (zero), `random_morph_tick` | `random_morph_tick` | |
| +0x41c | mode | H | `random_morph_set_load_raw` (from the file) | `random_morph_tick` | the three arms: §5a |
| +0x420..+0x42c | timing / pick tuning | H | `random_morph_set_load_raw` (from the file) | `random_morph_tick` | defaults 1000 / 1 / 1 / 1 |

**The per-cycle algorithm** (`random_morph_tick`, byte-identical across builds): mark each
channel whose current weight is above zero; choose `pick_base + rand() % pick_spread`
channels, clamped to the channel count; give each chosen channel a random integer weight
1..10; normalise by the total; set `delta = target − current` so the weights arrive over the
period. Then push the interpolated weights into every frame's morph interface through
`I3D_frame + 0x140`, vtable slots `+4` (count) and `+8` (set channel *i*,*j*).

---

## 5. The `rmorf.bin` file format

`Missions\<mission>\rmorf.bin`, opened through the `.dta` layer.

| Offset | Size | Field |
|---|---|---|
| +0x00 | 4 | total file size in bytes, header included |
| +0x04 | 4 | version, **float `0.1`** — anything else is rejected outright |
| +0x08 | 4 | record count |
| +0x0c | size−0x0c | the payload, read into one heap buffer and walked once |

Each record in the payload is `{ int frame_count; int mode; int period_base; int period_spread;
int pick_base; int pick_spread; char name[] * frame_count; }` — six ints, then the frame names.
`random_morph_set_load_raw` copies the six ints to the set record (`+0x00` count, `+0x41c` mode,
`+0x420` / `+0x424` period, `+0x428` / `+0x42c` pick) and then walks the names, which are
NUL-terminated and variable length; that is why it returns the advanced cursor rather than taking
an index, and `random_morph_mission_start` passes the returned cursor straight into the next call.

Every shipped file parses exactly under this layout and ends with one zero byte per record that the
header's size counts and the loader never reads.

### 5a. The shipped files

Twelve missions ship an `rmorf.bin`. One entry per record, as frames · mode · period base / spread
· pick base / spread:

| mission | records |
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

**Mode.** `random_morph_tick` tests the mode twice, at two widths: the additive arm fires when the
LOW BYTE is negative (`(char)mode < 0`), the eased arm when the whole dword equals 1. So the shipped
values split three ways: 0 ramps linearly, 1 eases, and 128 and 129 take the additive arm. The
eased arm applies `(1 - cos(pi * t)) / 2`, and the additive arm accumulates the deltas without
picking a new target. No string, symbol or script of either build names the modes, so what a
designer called them is not recoverable.

---

## 6. Coverage — PC first

| PC VA | PC name | PS2 twin | Status |
|---|---|---|---|
| 0x005ca2c0 | `random_morph_static_init` | - | - |
| 0x005ca2d0 | `random_morph_static_ctor` | - | - |
| 0x005ca2e0 | `random_morph_register_atexit_dtor` | - | - |
| 0x005ca2f0 | `random_morph_static_dtor` | - | - |
| 0x005ca300 | `random_morph_set_load_raw` | `LoadRAWSet__13RMRF_S_SETREFFPP13RMRF_S_SETREFPc` | CONFIRMED |
| 0x005ca4b0 | `random_morph_ctor` | `__ct__7C_rmorfFv` | CONFIRMED |
| 0x005ca4c0 | `random_morph_dtor` | `__dt__7C_rmorfFv` | CONFIRMED |
| 0x005ca510 | `random_morph_set_destroy` | `__dt__13RMRF_S_SETREFFv` | CONFIRMED |
| 0x005ca560 | `random_morph_mission_start` | `Mission_Start__7C_rmorfFv` | CONFIRMED |
| 0x005ca680 | `std_string_ctor_formatted` | `__ct__7fstringFPCce` | CONFIRMED |
| 0x005ca790 | `random_morph_mission_done` | `Mission_Done__7C_rmorfFv` | CONFIRMED |
| 0x005ca7e0 | `random_morph_tick` | `Tick__7C_rmorfFi` | CONFIRMED |
| 0x005caba0 | `std_string_dtor` | `__dt__Q23std59basic_string<>Fv` | STRUCTURAL |
| 0x005cabe0 | `std_string_tidy` | - | - |
| 0x005cac30 | `std_string_grow_buffer` | - | - |

The first four rows are PC CRT scaffolding; the four `std_string_*` rows are shared CRT template
members, not members of either class (below).

**5/5 `C_rmorf` methods and 2/2 `RMRF_S_SETREF` methods are matched. The class is CLOSED.**

The four `std_string_*` helpers are **not** members of either class. They are MSVC
`basic_string<char>` members emitted into this object file because this is where they were
used, so they are not named after the host TU. `_tidy` (13 refs) and `_grow_buffer` (6 refs) have
callers in the CRT band at `0x0062xxxx`, so they are genuinely shared.

* **`std_string_grow_buffer`'s true extent is `[0x005cac30, 0x005cad40)`.** The address
  `0x005cacb0` inside it is not a function start: it is reached by an unconditional
  `jmp 0x5cacb0` 0x29 bytes earlier and by the catch-resume fall-through at `0x005cacaa`, with
  zero `E8` callers and zero datarefs.
* `Catch_005cac89` is a real EH catch funclet — it carries a dataref in `.rdata:0x006434d4` (the
  MSVC FuncInfo table).

---

## 7. PC vs PS2

* **Source file.** The PS2 ELF names it `C_rmorf.cpp` in its `__sinit_` manifest, and the
  object's own string is `randomMorfTickableModule`. The PC has no such string; the class was
  identified from the shipped `Missions\%s\rmorf.bin` path, which both builds share verbatim.
* **Link order.** The PS2 emits `C_rmorf.cpp` immediately before `C_shvestky`
  (`0x001f8c70` → `0x001f99b0`); the PC does exactly the same
  (`0x005ca290` → `0x005cad40`). `C_rmorf.cpp` sorts before `C_shvestky.cpp`, so this is the
  alphabetical-by-source-file link order agreeing across both toolchains.
* **Record layout is IDENTICAL.** `+0x1c` weights, `+0x21c` deltas, `+0x41c` mode,
  `+0x420`/`+0x424` period, `+0x428`/`+0x42c` channel count — the same offsets on both
  builds, which is unusual and is what made the `Tick` diff decisive.
* **vptr placement differs**, as always: the PS2 puts the vtable at `+0x04` and the list head
  at `+0x00`; the PC does the reverse. Same two fields.
* **`fstring`.** The PS2 wraps "format into a string" in a class it calls `fstring`, deriving
  from `std::basic_string<char>`; MSVC inlines the same three calls (default ctor, `vsprintf`,
  `assign`) into one function. Both use a **1000-byte** stack scratch buffer, which is the
  shared constant that made the match falsifiable.
* **The morph interface offset**: the channel interface hangs off `I3D_frame + 0x140` on the
  PC; the PS2 side is open (§8).

---

## 8. Open items

* **What actually morphs in game has not been observed.** The MISE09 files name laundry,
  juniper, tree and truck frames (§5a); the other ten are not yet matched to their models. **In game:**
  load a mission with an `rmorf.bin` and look for a model that deforms on a ~1 s random cycle.
* **The PS2 offset of the morph interface is not measured.** The PC reaches the channel interface
  through `I3D_frame + 0x140` (§4); the PS2 side of that offset is unmeasured.
