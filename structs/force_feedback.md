# Force feedback — the manager, the effect class family, and the effects the game plays

**Created** 2026-08-13 23:54 +0200 · **Modified** 2026-10-04 11:49 +0200

Two classes live here and this file covers both:

| PC prefix | what it is | where |
|---|---|---|
| **`force_feedback_manager_`** | the singleton at **`0x006f91e0`** — owns the effect lists, hands out handles, arbitrates device slots | §2, §3 |
| **`force_feedback_effect_`** | a five-class polymorphic family of effect records, each wrapping one device-side effect | §4 |

The device-side objects themselves are **not in `Game.exe`** — they come from
**`ijoy.dll`**. See `basic/artifacts/dll.md`.

**Confidence tags:** **[H]** body-read on both ends · **[M]** inferred · **[U]** not observed.

---

## 1. How the subsystem is identified

1. **The factory is an 11-way polymorphic dispatch.** `force_feedback_manager_create_effect`
   switches on its type argument, `operator_new`s a different size per family and
   installs a different vtable. Eleven is exactly DirectInput's effect-type set —
   Constant, Ramp, five Periodic, four Condition — and the code uses **`0xc`** as an
   "unsupported type" sentinel that every setter tests.
2. **Durations are MICROSECONDS.** 120000 / 500000 / 200000 / 400000 on the human side,
   220000 / 100000 on the car side.
3. **One effect is steered by a DIRECTION ANGLE.** `human_do_jump` writes 0, `0x3fc90fdb`
   (π/2) or `0x4096cbe4` (3π/2) before playing its effect, choosing from the jump
   direction.
4. 🔑 **The import table names it outright.** `Game.exe` statically imports
   **`_CreateJoyBase@20` from `ijoy.dll`**, whose interfaces are `LSIInputDevice`
   (method-for-method `IDirectInputDevice8`), **`ILSIInputEffect`** and the parameter block
   **`LSIIEFFECT`** (= `DIEFFECT`) — see §6b. So the DirectInput reading is the developers'
   own vocabulary, not an inference.

---

## 2. The manager singleton `0x006f91e0`

Loaded as an immediate — every call into the subsystem is preceded by
`mov ecx, 0x6f91e0`, which makes the whole method set enumerable with one scan of `.text`
for that immediate (80 callsites).

| Offset | Field | Conf | Notes |
|---|---|---|---|
| `+0x00` | allocator byte of the idle vector | H | - |
| `+0x04` / `+0x08` / `+0x0c` | **IDLE list** — `vector<effect*>` begin/end/cap | H | every created effect starts here |
| `+0x10` | allocator byte of the playing vector | H | - |
| `+0x14` / `+0x18` / `+0x1c` | **PLAYING list** — `vector<effect*>` begin/end/cap | H | an effect moves here once the device accepts it |
| `+0x20` | next effect handle | H | `handle = (*+0x20)++`; reset by `..._reset_handle_counter` and by `..._game_init` |
| `+0x24` | tick accumulator | H | `+= 1000` per tick |
| `+0x28` | tick period | H | constructed to **1000** |
| `+0x2c` | **`g_force_feedback_human_enabled`** (`0x006f920c`) | H | filled from binding **1**; the flag every on-foot rumble site tests |
| `+0x2d` | **`g_force_feedback_car_enabled`** (`0x006f920d`) | H | filled from binding **0x26**; the CAR half of the pair (§2b) |

### 2a. Where the singleton is defined

The instance and the class live in different files. The singleton at `0x006f91e0` is a
**file-scope object of `Main.cpp`**, and its four CRT stubs sit at the very end of that TU's
initializer block:

| PC VA | PC name | body |
|---|---|---|
| `0x005f8370` | `force_feedback_manager_static_init` | `.CRT$XCU` slot `0x006462b0` |
| `0x005f8380` | `force_feedback_manager_static_ctor` | `mov ecx, 0x6f91e0; jmp force_feedback_manager_ctor` |
| `0x005f8390` | `force_feedback_manager_register_atexit_dtor` | `atexit(&…_static_dtor)` |
| `0x005f83a0` | `force_feedback_manager_static_dtor` | `mov ecx, 0x6f91e0; jmp force_feedback_manager_dtor` |

The cross-build placement is in §6c. The same TU defines the joy device vector. The receiver is
byte-proven from both stubs' `mov ecx, 0x6f91e0`; the dtor is reached only as
`atexit(&force_feedback_manager_static_dtor)`.

The class's 30 method bodies are implemented in the TU at `[0x005ae4b0, 0x005af290)` (§3).

### 2b. What `+0x2d` distinguishes

`force_feedback_manager_game_init` looks up binding **1** and binding **0x26** in the
input-binding table, and for each takes a capability byte from the ijoy device's vtable
`+0x68` — but it reads **two different fields** of the returned struct. On the producer
side alone, *"two devices"* and *"two capability bits of one device"* are both consistent.

The consumer census settles it. The two flags are read in exactly four places, and they
partition cleanly by subsystem:

```
game_init:  if (g_force_feedback_human_enabled) human_create_force_feedback_effects();
            if (g_force_feedback_car_enabled)   car_create_force_feedback_effects();
game_done:  if (g_force_feedback_human_enabled) human_release_force_feedback_effects();
            if (g_force_feedback_car_enabled)   car_release_force_feedback_effects();
```

`+0x2c` additionally gates every human rumble site in the engine; `+0x2d` is read
nowhere else at all. `force_feedback_manager_is_enabled(sel)` returns `+0x2d` when
`sel == 0`, and **that arm has zero callers** — its one caller passes `1`.

⇒ Binding **1** is an on-foot control and binding **0x26** a driving control, and the
two bytes are the force-feedback capability of the device bound to each.

---

## 3. The manager's method set

| VA | Function | Role |
|---|---|---|
| `0x004a6070` | `force_feedback_manager_is_enabled` | `sel ? +0x2c : +0x2d`. Inline member out-lined into the `C_human.cpp` TU |
| `0x005ae6b0` | `force_feedback_manager_ctor` | zeroes both lists, period = 1000 |
| `0x005ae6f0` | `force_feedback_manager_dtor` | frees both lists |
| `0x005ae720` | `force_feedback_manager_reset_handle_counter` | `+0x20 = 0`; sole caller `engine_init` |
| `0x005ae730` | `force_feedback_manager_game_init` | probes bindings 1 and 0x26; sole caller `game_init` |
| `0x005ae7c0` | `force_feedback_manager_game_done` | destroys every effect in both lists; sole caller `game_done` |
| `0x005ae890` | `force_feedback_manager_create_effect` | **the 11-type factory**; returns a handle |
| `0x005aeb10` | `force_feedback_manager_find_effect_by_handle` | handle → record, searches both lists |
| `0x005aeb80` | `force_feedback_manager_set_constant_magnitude` | gates `type == 0` → effect `+0x34` |
| `0x005aebc0` | `force_feedback_manager_set_ramp_range` | gates `type == 1` → effect `+0x34/+0x38` |
| `0x005aec00` | `force_feedback_manager_set_periodic_params` | gates `type` 2..6 → `+0x34..+0x40` (magnitude, offset, phase, **period**) |
| `0x005aec60` | `force_feedback_manager_set_condition_params` | gates `type` 7..10; `operator_new(0x18)` **per axis** → `+0x34/+0x38` |
| `0x005aece0` | `force_feedback_manager_set_gain` | any type → `+0x1c` and the device |
| `0x005aed10` | `force_feedback_manager_set_direction` | any type → `+0x18` and the device |
| `0x005aed40` | `force_feedback_manager_set_duration` | any type → `+0x20` and the device |
| `0x005aed80` | `force_feedback_manager_play_effect` | start + **device-slot arbitration** (§3a) |
| `0x005af010` | `force_feedback_manager_release_effect` | destroy by handle, then retry pending |
| `0x005af0e0` | `force_feedback_manager_stop_effect` | playing → idle; the eviction primitive |
| `0x005af140` | `force_feedback_manager_find_evictable_effect` | eviction-victim chooser (§3a) |
| `0x005af200` | `force_feedback_manager_find_highest_priority_pending` | retry-order chooser |
| `0x005af240` | `force_feedback_manager_tick` | throttled heartbeat → retry pending; sole caller `game_tick` |
| `0x005af260` | `force_feedback_manager_retry_pending_effects` | drains the pending queue |
| `0x005f8380` / `0x005f83a0` | `force_feedback_manager_static_ctor` / `_dtor` | static-init stubs, in the `Main.cpp` TU (§2a) |

### 3a. Device-slot arbitration — why `+0x24` is a priority

The device has a finite number of effect slots. `force_feedback_manager_play_effect`
loops the download and, on each failure, evicts:

1. `force_feedback_manager_find_evictable_effect` picks a victim by a **two-tier rule** —
   an entry that is **no longer playing** is taken immediately, priority ignored;
   otherwise the entry with the **lowest** `+0x24`.
2. `force_feedback_manager_stop_effect` unloads it (the record survives, it just moves
   back to the idle list).
3. If no victim is available, or the victim outranks the newcomer, the effect is **parked
   pending** (`+0x29 = 1`) and `force_feedback_manager_retry_pending_effects` — driven by
   `..._release_effect` and by the tick — will try it again, highest `+0x24` first.

🔴 **`+0x24` is a PRIORITY, not a magnitude.** It is never transmitted: the device create
call sends only `+0x14/+0x18/+0x20/+0x1c`, and the type overrides send only `+0x34…`. Its
only two readers are the two selectors above. The real strength is the float at `+0x34`.

---

## 4. The effect class family — five classes, one virtual each

`0x0063e394..0x0063e3a4` is **five consecutive single-entry vtables** — `0x0063e3a8` holds
`0x65727453` (`"Stre"`), string data, so the region ends there. A vtable dump that merges
adjacent runs renders it as nested tables, which it is not. Each install site is
byte-located:

| vtable | class | slot 0 = "push my parameters to the device" | installed by |
|---|---|---|---|
| `0x0063e394` | base | `force_feedback_effect_upload_common_params` (`0x005ae510`) | `..._effect_ctor` @ `0x005ae4f9` |
| `0x0063e398` | periodic | `force_feedback_effect_periodic_upload_params` (`0x005ae660`) | `..._periodic_ctor` @ `0x005ae621` |
| `0x0063e39c` | condition | `force_feedback_effect_condition_upload_params` (`0x005ae690`) | factory cases 7–10 @ `0x005ae9f2` |
| `0x0063e3a0` | ramp | `force_feedback_effect_ramp_upload_params` (`0x005ae5d0`) | factory case 1 @ `0x005ae946` |
| `0x0063e3a4` | constant | `force_feedback_effect_constant_upload_params` (`0x005ae5b0`) | factory case 0 @ `0x005ae8f6` |

All five slot-0 bodies have **0 direct callsites and exactly 1 dataref** — pure virtual
dispatch, one vtable each ⇒ one class each, no fold. Every typed setter writes its fields
then calls slot 0, so *setter → fields → slot 0 → the matching ijoy call* is a closed
evidence loop.

🔎 The base's slot 0 is **effectively dead**: the factory gives every constructed effect a
derived vtable, so the base default is never dispatched.

### 4a. The effect record layout

| Offset | Field | Conf | Notes |
|---|---|---|---|
| `+0x00` | vtable | H | one of the five above |
| `+0x04`..`+0x13` | **a 16-byte member with its own ctor** | U | written once, by `force_feedback_effect_zero_member_at_0x04` from `force_feedback_effect_ctor`, and read nowhere in `Game.exe`: every effect access is in this TU or in `car_update`, which only asks `force_feedback_effect_is_playing` (`+0x30`). It never reaches `ijoy.dll` either, because `force_feedback_effect_create_on_device` passes literal zeros where a DirectInput-shaped call takes the envelope and the start delay. What the member was meant to hold is not recoverable |
| `+0x14` | type | H | 0 Constant · 1 Ramp · 2–6 Periodic · 7–10 Condition · **0xc = unsupported** |
| `+0x18` | direction | H | - |
| `+0x1c` | gain | H | - |
| `+0x20` | duration (µs) | H | - |
| `+0x24` | **priority** | H | never transmitted; §3a |
| `+0x28` | device selector | H | picks which binding's device the factory creates on |
| `+0x29` | pending flag | H | set when the device had no free slot |
| `+0x2c` | handle | H | what every manager method takes |
| `+0x30` | the **ijoy** effect object | H | `ILSIInputEffect`; its vtable is in the DLL |
| `+0x34`… | type-specific params | H | 1 dword constant · 2 ramp · 4 periodic · 2 condition-block pointers |

Sizes confirm the layout: **0x38** constant, **0x3c** ramp, **0x44** periodic and condition.

Other members: `force_feedback_effect_ctor` (`0x005ae4c0`),
`force_feedback_effect_periodic_ctor` (`0x005ae5f0`),
`force_feedback_effect_zero_member_at_0x04` (`0x005ae4b0`),
`force_feedback_effect_is_playing` (`0x005ae550`, ijoy status `+0x58 == 5`, 18 callsites),
`force_feedback_effect_create_on_device` (`0x005ae580`).

---

## 5. The effects the game builds

### 5a. The four human effects — `human_create_force_feedback_effects`

| Global | VA | Type | Duration | **Priority** | Played by |
|---|---|---|---|---|---|
| `g_ff_effect_weapon` | `0x00655bf4` | 0 (constant) | 0.12 s | 30 | `human_change_weapon_model`, `human_shooting` |
| `g_ff_effect_jump` | `0x00655bf0` | 3 (periodic) | 0.50 s | 20 | `human_do_jump`, re-steered 0 / π/2 / 3π/2 |
| `g_ff_effect_hit` | `0x00655bec` | 0 (constant) | 0.20 s | 50 | `human_hit` |
| `g_ff_effect_landing` ¹ | `0x00655be8` | 1 (ramp) | 0.40 s | 40 | `human_movement_intern` |

¹ The byte census is human-only: three real operands in `human_movement_intern`, and the one
hit outside disassembles as `call 0x425b80`, an `E8` opcode coincidence. Ghidra renders two
`bridge_game_init` accesses at this address only because it folds the steamboat slot index to 32
(`0x00655768 + 32 × 0x24`); the instructions address `(&g_steamboat_slot_frame)[slot * 9]`.

Every one of those gains is scaled by **`g_force_feedback_strength`** (`0x006724c4`) — the
options slider, pushed into widget id `0xbbe`.

### 5b. The five car effects — `car_create_force_feedback_effects`

`car_release_force_feedback_effects` releases all five.

| Global | Type | Priority | What drives its gain in `car_update` |
|---|---|---|---|
| `g_ff_car_effect_wheel_friction` | 10 (Condition) | 10 | accumulates over the wheel-contact array `car+0xd24` bounded by `car+0x5b0`, then falls with speed `car+0x688` (floor 0.34) |
| `g_ff_car_effect_lateral_spring` | 7 (Condition) | **300** | condition blocks from the car's local lateral axis `car+0x1f80`; gain `car+0x1f84 × load` |
| `g_ff_car_effect_impact` | 0 (Constant), 220 ms | 201 | the change of the car-space velocity `car+0x1f7c` against `g_car_ff_prev_velocity` (horizontal part, × 0.2, clamped to 1.0) scales the gain; direction from `vec2_signed_angle_diff() + π` — a short directional jolt |
| `g_ff_car_effect_speed_vibration` | 2 (Periodic) | 200 | **period** re-set each frame from `(car+0x688 − 3.0) × 0.04761905` clamped 0..1 |
| `g_ff_car_effect_steady_vibration` | 2 (Periodic) | 0 | constant 100 ms period; played only while the engine runs (`car+0xd18` = `C_Vehicle+0xca8`, `structs/car.md` §1) |

All five gains are `weight × g_ff_car_gain_master × <quantity>`, with per-effect weights
`g_ff_car_gain_*` and two change-detect caches `g_ff_car_last_gain_*`. The whole block is
gated on `car+0x594` = `C_Vehicle+0x524`, the steering axis's direct-input flag
(`structs/vehicle.md` §1c): for the player's car it is set while steering comes from the
SECONDARY key binding; otherwise all five are stopped.

### 5c. Where the weights come from — the Controls screen's Force Feedback sliders

`car_set_force_feedback_gains` (`0x00472f00`) is the sole writer of the five gain weights, and its
one caller is `loadsave_apply_settings` (bit `8`), which passes five fields of the
`G_LoadSaveClass` singleton. Those fields are the five sliders of the Controls screen
(`tables/menu.def` design 10), each also a named global because the singleton sits at
`0x00671fb0`:

| Slider (textdb) | Component | Global | `G_LoadSaveClass` | Weight it sets |
|---|---|---|---|---|
| 241 Global Stick Volume | 3001 | `g_force_feedback_global_volume` | `+0x500` | `g_ff_car_gain_master` |
| 242 Suspension | 3002 | `g_force_feedback_suspension` | `+0x504` | the speed-vibration weight |
| 243 Collisions | 3003 | `g_force_feedback_collisions` | `+0x508` | the impact weight |
| 244 Revs. | 3004 | `g_force_feedback_revs` | `+0x50c` | the steady-vibration weight |
| 245 Tire Grip | 3005 | `g_force_feedback_tire_grip` | `+0x510` | the wheel-friction weight |
| 246 Player Movement | 3006 | `g_force_feedback_strength` | `+0x514` | scales every human effect (§5a) |

Apart from that call, the five car sliders are referenced only by `gui_options_controls_on_exchange`
and `gui_options_controls_tick`, which push them into the widgets and read them back.

---

## 6. PC vs PS2

### 6a. PS2 has the same role in a different implementation — but **no function twin**

PS2 ships **`C_ff_manager`**: `{ctor}` · `{dtor}` · `GameInit` · `GameDone` · `Tick(Ui)` ·
`SetConstVibration` · `SetLinearVibration` · `SetVibratoion` (sic) · `ClearVibrations`,
plus `__sinit_C_ff_manager.cpp` in the source-file manifest. It is the **DualShock
actuator mixer**: `Tick` ages a `std::vector<C_ff_manager::S_effect>` (stride **0x24**),
evaluates constant / ramp / triangle / sine envelopes **in software**, and pushes two
motor bytes through `scePadSetActDirect`. Its object is `{count, data, next_id,
last_played}` at `+0x04..+0x10`.

`C_ff_manager::S_effect` is a **POD** — its only member function anywhere in the binary is
a compiler-generated `operator=`. No ctor, no `__vt__`, no subclasses. Layout, read off
`SetConstVibration` and `Tick`: `+0x00` type · `+0x04` **priority** · `+0x08` handle ·
`+0x0c` duration · `+0x10` elapsed · `+0x14/+0x16` the two motor magnitudes ·
`+0x18/+0x1c` ramp deltas · `+0x20` period.

⇒ **The PC's five-class polymorphic effect family is PC-only; the effect RECORD is not**,
and both builds carry a priority field used for the same arbitration job.
⇒ 🔴 **No PC function here carries a `PS2 twin:` line.** The PS2 methods are role
analogues; stamping one would assert a body agreement nobody can check.
`IGraph::SetControlRumble(float,float)` is PS2's other rumble entry point.

### 6b. The interfaces the PC keeps in a DLL, and the PS2 compiled in

`LSIInputDevice`, `ILSIInputEffect` and `LSIIEFFECT` **are PC class names** — they simply
live in `ijoy.dll`, not in `Game.exe`, so no PC function will ever be nameable for them.
The PS2, having no DLLs, statically links its own implementations, which is why those
symbols exist there. `basic/artifacts/dll.md` has the whole picture, including the general
consequence: a PS2 method with no PC twin may simply be in a DLL.

### 6c. Which source file the PC TU is

PS2's link order is `… C_DropOut · C_ff_manager · C_ff_manager::S_effect · C_grenade …`, and
`__sinit_Main.cpp`'s ninth slot is `__ct__12C_ff_managerFv(0x50aa50)` — the statement matching
the PC singleton's static ctor (§2a), the ninth and last slot of a nine-of-nine ordered lockstep
between the two builds' `Main.cpp` initializer blocks, so the class exists on both builds and its
instance is defined in the same source file.

The PC TU's own file name is a **CANDIDATE**, not settled. `.text` holds at least two
alphabetical link runs of `C_*.cpp` objects, so a file's run membership is not decided by its
name. Run 2 reads `C_animations.cpp` → the `C_blood` / `C_blood_manager` TU
(`[0x005adfb0, 0x005ae480)`) → the FF TU → `C_program.cpp`, with the FF vtables in run 2's own
vtable sequence, monotone with run 2's `.text` order
(`0x0059a12a → 0x005a63ee → 0x005ae4f9`). `C_ff_manager` sorts inside that window
(`a < b < f < p`), so `C_ff_manager.cpp` is consistent with every measurement taken. The binary
cannot settle it further: `Game.exe` holds no source-file strings at all, and 26 PS2 source files
sort inside the same window (`C_blood.cpp` to `C_player.cpp`), so the alphabet does not single out
one name either.

---

## 7. Open items

| Item | State |
|---|---|
| in game | whether force feedback does anything without a rumble-capable pad, and whether the CAR flag `+0x2d` corresponds to a second settings toggle |
