# `C_ff_manager` and the effect family — force feedback, and the effects the game plays

**Created** 2026-08-13 23:54 +0200 · **Modified** 2026-11-08 22:21 +0100

Two classes live here. **The manager** is the singleton at `0x006f91e0`: it owns the effect lists, hands out handles and arbitrates
device slots. **The effect** is a five-class polymorphic family of records, each wrapping one device-side effect. The device-side
objects themselves are **not in `Game.exe`**: they come from **`ijoy.dll`** (`basic/artifacts/dll.md`). The name `C_ff_manager` comes
from the PS2 symbol table; §9a says how far the two builds' classes agree.

| | |
|---|---|
| PC prefix | `force_feedback_manager_` for the manager, `force_feedback_effect_` for the effect family |
| PC kind (`+0x10`) | - |
| PC vtable | manager `-`; effects `g_force_feedback_effect_vt` @ `0x0063e394`, then periodic, condition, ramp, constant, 1 slot each |
| PC base class | - |
| PC instance size | manager `0x30`; effect `0x38` constant, `0x3c` ramp and condition, `0x44` periodic |
| PC constructor | `force_feedback_manager_ctor`, `force_feedback_effect_ctor`, `force_feedback_effect_periodic_ctor` |
| PC translation unit | `[0x005ae480, 0x005af290)`, guard to guard; the singleton's quartet lies in the `Main.cpp` unit, §5 |
| PS2 class | `C_ff_manager` (tag `12C_ff_manager`), 9 methods, a different implementation with no function twin (§9a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. The manager

```c
typedef struct C_ff_manager {                // 0x30 bytes, singleton at 0x006f91e0
    /* +0x00 */ uint8_t  idle_alloc;         // [H] MSVC's empty allocator member of the idle list
    /* +0x04 */ S_ff_effect **idle_begin;    // [H] the IDLE list, stride 4; every created effect starts here
    /* +0x08 */ S_ff_effect **idle_end;      // [H] count = (end - begin) >> 2
    /* +0x0c */ S_ff_effect **idle_cap;      // [H] one past the allocation
    /* +0x10 */ uint8_t  playing_alloc;      // [H] the empty allocator of the playing list
    /* +0x14 */ S_ff_effect **playing_begin; // [H] the PLAYING list, stride 4; an effect moves here once the device accepts it
    /* +0x18 */ S_ff_effect **playing_end;   // [H] count = (end - begin) >> 2
    /* +0x1c */ S_ff_effect **playing_cap;   // [H] one past the allocation
    /* +0x20 */ uint32_t next_handle;        // [H] handle = (*+0x20)++, stored in the new effect
    /* +0x24 */ uint32_t tick_accumulator;   // [H] += 1000 per tick
    /* +0x28 */ uint32_t tick_period;        // [H] constructed to 1000; past it the tick retries the pending effects
    /* +0x2c */ uint8_t  human_enabled;      // [H] g_force_feedback_human_enabled (0x006f920c), from binding 1
    /* +0x2d */ uint8_t  car_enabled;        // [H] g_force_feedback_car_enabled (0x006f920d), from binding 0x26
} C_ff_manager;
```

Loaded as an immediate: every call into the subsystem is preceded by `mov ecx, 0x6f91e0`, which makes the method set enumerable with one
scan of `.text` for that immediate (80 callsites). The tick adds 1000 against a period of 1000, so the retry runs on every second call.

### 1b. The effect record

```c
typedef struct S_ff_effect {                 // 0x38 / 0x3c / 0x44 bytes by family
    /* +0x00 */ void    *vtable;             // [H] one of the five of section 7
    /* +0x04 */ uint32_t unread_04[4];       // [U] zeroed by its own ctor; no reader (class census)
    /* +0x14 */ uint32_t type;               // [H] 0 Constant, 1 Ramp, 2-6 Periodic, 7-10 Condition, 0xc unsupported
    /* +0x18 */ uint32_t direction;          // [H] direction angle, a float bit pattern
    /* +0x1c */ uint32_t gain;               // [H] the base upload and force_feedback_manager_set_gain send it through device slot +0x1c
    /* +0x20 */ uint32_t duration_us;        // [H] microseconds
    /* +0x24 */ uint32_t priority;           // [H] never transmitted; section 6
    /* +0x28 */ uint8_t  device_selector;    // [H] 0 = the device of binding 0x26, else the device of binding 1
    /* +0x29 */ uint8_t  pending;            // [H] the device had no free slot
    /* +0x2c */ uint32_t handle;             // [H] what every manager method takes
    /* +0x30 */ ILSIInputEffect *device_effect; // [H] the ijoy effect object; its vtable is in the DLL
    /* +0x34 */ uint32_t param[4];           // [H] by family: 1 dword constant, 2 ramp, 4 periodic, 2 block pointers
} S_ff_effect;
```

**The arrays.** `unread_04` is four dwords because `force_feedback_effect_zero_member_at_0x04` zeroes exactly four, and nothing else
in `Game.exe` touches them: every effect access is in this unit or in `car_update`, which only asks
`force_feedback_effect_is_playing` (`+0x30`). It never reaches `ijoy.dll` either, because `force_feedback_effect_create_on_device`
passes literal zeros where a DirectInput-shaped call takes the envelope and the start delay. What the member was meant to hold is not
recoverable. `param` is four dwords, the most a setter fills (the periodic family, whose `0x44`-byte record ends with them); a
family uses as many as its setter fills, which is what the allocation sizes `0x38`, `0x3c`, `0x44` measure:

| Family | `param` | Set by |
|---|---|---|
| Constant | `[0]` magnitude | `force_feedback_manager_set_constant_magnitude` |
| Ramp | `[0]`, `[1]` range | `force_feedback_manager_set_ramp_range` |
| Periodic | `[0]` magnitude, `[1]` offset, `[2]` phase, `[3]` period | `force_feedback_manager_set_periodic_params` |
| Condition | `[0]`, `[1]` per-axis `S_ff_condition` blocks | `force_feedback_manager_set_condition_params` |

**The condition block.** `force_feedback_manager_set_condition_params` allocates one `0x18`-byte block per axis
(`operator_new(0x18)`) and fills each with a six-dword copy of the caller's array. The six are floats in the order of DirectInput's
`DICONDITION`, the reading §4's `LSIIEFFECT` = `DIEFFECT` supports; the values the callers pass fit it:

```c
typedef struct S_ff_condition {              // 0x18 bytes, one per axis
    /* +0x00 */ float offset;                // [M] DICONDITION offset; the signed steer ratio on axis 0
    /* +0x04 */ float positive_coefficient;  // [M] DICONDITION positive coefficient
    /* +0x08 */ float negative_coefficient;  // [M] DICONDITION negative coefficient
    /* +0x0c */ float positive_saturation;   // [M] DICONDITION positive saturation
    /* +0x10 */ float negative_saturation;   // [M] DICONDITION negative saturation
    /* +0x14 */ float dead_band;             // [M] DICONDITION dead band
} S_ff_condition;
```

`car_update` fills axis 0 of the lateral spring with {signed steer ratio, 1.0, 1.0, 0.8, 0.8, 0} and axis 1 with
{0, 1, 1, 1, 1, 0}; `car_create_force_feedback_effects` creates the effects with {0, 1, 1, 0, 0, 0} and zeros. The block is read on
the device side, in `ijoy.dll`, so the field order is [M] (Open item 3).

## 2. Field evidence

Cells come from a census of each class's functions, read against the bodies. The manager's effect list scans are read as `*(list + i * 4) + N`,
so the effect's cells also take the manager functions below.

### 2a. The manager

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | idle_alloc | H | `force_feedback_manager_ctor` | - | an empty class |
| +0x4 | idle_begin | H | `force_feedback_manager_ctor` | `force_feedback_manager_find_effect_by_handle`, `force_feedback_manager_find_highest_priority_pending`, `force_feedback_manager_game_done`, `force_feedback_manager_play_effect`, `force_feedback_manager_release_effect`, `force_feedback_manager_retry_pending_effects`, `force_feedback_manager_stop_effect` | - |
| +0x8 | idle_end | H | `force_feedback_manager_ctor`, `force_feedback_manager_game_done`, `force_feedback_manager_play_effect`, `force_feedback_manager_release_effect`, `force_feedback_manager_stop_effect` | `force_feedback_manager_create_effect`, `force_feedback_manager_find_effect_by_handle`, `force_feedback_manager_find_highest_priority_pending`, `force_feedback_manager_game_done`, `force_feedback_manager_play_effect`, `force_feedback_manager_release_effect`, `force_feedback_manager_retry_pending_effects`, `force_feedback_manager_stop_effect` | - |
| +0xc | idle_cap | H | `force_feedback_manager_ctor` | - | the vector helpers behind `create_effect` read it |
| +0x10 | playing_alloc | H | `force_feedback_manager_ctor` | - | an empty class |
| +0x14 | playing_begin | H | `force_feedback_manager_ctor` | `force_feedback_manager_find_effect_by_handle`, `force_feedback_manager_find_evictable_effect`, `force_feedback_manager_game_done`, `force_feedback_manager_release_effect`, `force_feedback_manager_stop_effect` | - |
| +0x18 | playing_end | H | `force_feedback_manager_ctor`, `force_feedback_manager_game_done`, `force_feedback_manager_release_effect`, `force_feedback_manager_stop_effect` | `force_feedback_manager_find_effect_by_handle`, `force_feedback_manager_find_evictable_effect`, `force_feedback_manager_game_done`, `force_feedback_manager_release_effect`, `force_feedback_manager_stop_effect` | - |
| +0x1c | playing_cap | H | `force_feedback_manager_ctor` | - | - |
| +0x20 | next_handle | H | `force_feedback_manager_create_effect`, `force_feedback_manager_game_init`, `force_feedback_manager_reset_handle_counter` | `force_feedback_manager_create_effect` | - |
| +0x24 | tick_accumulator | H | `force_feedback_manager_ctor`, `force_feedback_manager_tick` | `force_feedback_manager_tick` | - |
| +0x28 | tick_period | H | `force_feedback_manager_ctor` | `force_feedback_manager_tick` | - |
| +0x2c | human_enabled | H | `force_feedback_manager_game_init` | `force_feedback_manager_is_enabled`, `game_done`, `game_init` | `game_init` and `game_done` read the global (§5a); every human rumble site tests it |
| +0x2d | car_enabled | H | `force_feedback_manager_game_init` | `force_feedback_manager_is_enabled`, `game_done`, `game_init` | §5a |

### 2b. The effect record

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vtable | H | `force_feedback_effect_ctor`, `force_feedback_effect_periodic_ctor`, `force_feedback_manager_create_effect` | - | virtual dispatch: `force_feedback_manager_set_gain` and the other setters call slot 0 |
| +0x4 | unread_04 | U | `force_feedback_effect_zero_member_at_0x04` | - | no reader (class census); Open item 2 |
| +0x14 | type | H | `force_feedback_effect_ctor`, `force_feedback_effect_periodic_ctor` | `force_feedback_effect_create_on_device`, `force_feedback_effect_periodic_ctor`, `force_feedback_manager_set_condition_params`, `force_feedback_manager_set_constant_magnitude`, `force_feedback_manager_set_periodic_params`, `force_feedback_manager_set_ramp_range` | the periodic ctor stores `0xc` for a type outside 2..6 |
| +0x18 | direction | H | `force_feedback_effect_ctor`, `force_feedback_manager_set_direction` | `force_feedback_effect_create_on_device`, `force_feedback_effect_upload_common_params` | `human_do_jump` picks 0, π/2 or 3π/2 |
| +0x1c | gain | H | `force_feedback_effect_ctor`, `force_feedback_manager_set_gain` | `force_feedback_effect_create_on_device`, `force_feedback_effect_upload_common_params` | - |
| +0x20 | duration_us | H | `force_feedback_effect_ctor`, `force_feedback_manager_set_duration` | `force_feedback_effect_create_on_device`, `force_feedback_effect_upload_common_params`, `force_feedback_manager_find_highest_priority_pending` | only pending effects with `duration_us == 0` (infinite effects) are retried |
| +0x24 | priority | H | `force_feedback_effect_ctor` | `force_feedback_manager_find_evictable_effect`, `force_feedback_manager_find_highest_priority_pending`, `force_feedback_manager_play_effect` | §6; `play_effect` parks the newcomer when the victim's priority is at least its own and the victim still plays |
| +0x28 | device_selector | H | `force_feedback_effect_ctor` | `force_feedback_manager_create_effect` | - |
| +0x29 | pending | H | `force_feedback_effect_ctor`, `force_feedback_manager_play_effect` | `force_feedback_manager_find_highest_priority_pending`, `force_feedback_manager_retry_pending_effects` | - |
| +0x2c | handle | H | `force_feedback_manager_create_effect` | `force_feedback_manager_create_effect`, `force_feedback_manager_find_effect_by_handle` | `create_effect` returns it |
| +0x30 | device_effect | H | `force_feedback_effect_ctor`, `force_feedback_manager_create_effect` | `force_feedback_effect_condition_upload_params`, `force_feedback_effect_constant_upload_params`, `force_feedback_effect_create_on_device`, `force_feedback_effect_is_playing`, `force_feedback_effect_periodic_upload_params`, `force_feedback_effect_ramp_upload_params`, `force_feedback_effect_upload_common_params`, `force_feedback_manager_game_done` | `force_feedback_effect_is_playing` tests ijoy status `+0x58 == 5` |
| +0x34 | param | H | `force_feedback_effect_periodic_ctor`, `force_feedback_manager_create_effect`, `force_feedback_manager_set_condition_params`, `force_feedback_manager_set_constant_magnitude`, `force_feedback_manager_set_periodic_params`, `force_feedback_manager_set_ramp_range` | `force_feedback_effect_condition_upload_params`, `force_feedback_effect_constant_upload_params`, `force_feedback_effect_periodic_upload_params`, `force_feedback_effect_ramp_upload_params` | the real strength. `[0]` constant magnitude, ramp start, periodic magnitude or the axis-0 condition block; `[1]` ramp end, periodic offset or the axis-1 block; `[2]` periodic phase; `[3]` periodic period, which `car_update` re-sets each frame for the speed vibration |

### 2c. `S_ff_condition`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | offset | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; the caller's value is the signed steer ratio (§8b) |
| +0x04 | positive_coefficient | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; 1.0 from both callers |
| +0x08 | negative_coefficient | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; 1.0 from both callers |
| +0x0c | positive_saturation | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; 0.8 on axis 0, 1.0 on axis 1 from `car_update` |
| +0x10 | negative_saturation | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; as `positive_saturation` |
| +0x14 | dead_band | M | `force_feedback_manager_set_condition_params` | - | read in `ijoy.dll`; 0 from both callers |

## 3. Method family

**No function here carries a `PS2 twin:` line** (§9a), so every row's twin is `-`. Every other row of the unit is listed.

### 3a. Construction & destruction

#### The manager

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005ae480 | `crt_static_init_guard_005ae480` | - | - | - | - | The unit's opening guard |
| 0x005ae4a0 | `crt_atexit_register_005ae4a0` | - | - | - | - | The guard's `atexit` helper |
| 0x005ae6b0 | `force_feedback_manager_ctor` | instance | - | - | - | Zeroes both lists, period = 1000 |
| 0x005ae6f0 | `folded_vector_pair_free_at_0x04_0x14` | instance | - | - | - | The manager's destructor: frees both lists. A fold with the pursuit-event manager, whose two vectors sit at the same offsets (`structs/pursuit_event.md` §1) |
| 0x005f8370 | `force_feedback_manager_static_init` | `0x006f91e0` | - | - | - | `.CRT$XCU` slot `0x006462b0`, in the `Main.cpp` unit (§5) |
| 0x005f8380 | `force_feedback_manager_static_ctor` | `0x006f91e0` | - | - | - | `mov ecx, 0x6f91e0; jmp force_feedback_manager_ctor` |
| 0x005f8390 | `force_feedback_manager_register_atexit_dtor` | `0x006f91e0` | - | - | - | `atexit(&force_feedback_manager_static_dtor)` |
| 0x005f83a0 | `force_feedback_manager_static_dtor` | `0x006f91e0` | - | - | - | `mov ecx, 0x6f91e0; jmp folded_vector_pair_free_at_0x04_0x14` |

#### The effect family

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005ae4c0 | `force_feedback_effect_ctor` | instance | - | - | - | The base constructor |
| 0x005ae5f0 | `force_feedback_effect_periodic_ctor` | instance | - | - | - | The periodic constructor; `0xc` for a type outside 2..6 |

### 3b. Accessors

#### The manager

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005ae720 | `force_feedback_manager_reset_handle_counter` | - | - | - | `+0x20 = 0`; sole caller `engine_init` |

### 3c. Behaviour

#### The manager

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004a6070 | `force_feedback_manager_is_enabled` | - | - | - | `sel ? +0x2c : +0x2d`; an inline member out-lined into the `C_human.cpp` unit |
| 0x005ae730 | `force_feedback_manager_game_init` | - | - | - | Probes bindings 1 and `0x26`; sole caller `game_init` |
| 0x005ae7c0 | `force_feedback_manager_game_done` | - | - | - | Destroys every effect in both lists; sole caller `game_done` |
| 0x005ae890 | `force_feedback_manager_create_effect` | - | - | - | **The 11-type factory**; returns a handle |
| 0x005aeb10 | `force_feedback_manager_find_effect_by_handle` | - | - | - | Handle to record, searches both lists |
| 0x005aeb80 | `force_feedback_manager_set_constant_magnitude` | - | - | - | Gates `type == 0`, sets `+0x34` |
| 0x005aebc0 | `force_feedback_manager_set_ramp_range` | - | - | - | Gates `type == 1`, sets `+0x34`, `+0x38` |
| 0x005aec00 | `force_feedback_manager_set_periodic_params` | - | - | - | Gates `type` 2..6, sets `+0x34..+0x40` |
| 0x005aec60 | `force_feedback_manager_set_condition_params` | - | - | - | Gates `type` 7..10; `operator_new(0x18)` per axis into `+0x34`, `+0x38` |
| 0x005aece0 | `force_feedback_manager_set_gain` | - | - | - | Any type: `+0x1c` and the device |
| 0x005aed10 | `force_feedback_manager_set_direction` | - | - | - | Any type: `+0x18` and the device |
| 0x005aed40 | `force_feedback_manager_set_duration` | - | - | - | Any type: `+0x20` and the device |
| 0x005aed80 | `force_feedback_manager_play_effect` | - | - | - | Start + device-slot arbitration (§6) |
| 0x005af010 | `force_feedback_manager_release_effect` | - | - | - | Destroy by handle, then retry pending |
| 0x005af0e0 | `force_feedback_manager_stop_effect` | - | - | - | Playing to idle; the eviction primitive |
| 0x005af140 | `force_feedback_manager_find_evictable_effect` | - | - | - | Eviction-victim chooser (§6) |
| 0x005af200 | `force_feedback_manager_find_highest_priority_pending` | - | - | - | Retry-order chooser |
| 0x005af240 | `force_feedback_manager_tick` | - | - | - | Throttled heartbeat, retries pending; sole caller `game_tick` |
| 0x005af260 | `force_feedback_manager_retry_pending_effects` | - | - | - | Drains the pending queue |

#### The effect family

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005ae4b0 | `force_feedback_effect_zero_member_at_0x04` | - | - | - | Zeroes the 16-byte member `unread_04` |
| 0x005ae510 | `force_feedback_effect_upload_common_params` | - | - | - | Slot 0 of the base vtable; never dispatched, because the factory gives every effect a derived vtable |
| 0x005ae550 | `force_feedback_effect_is_playing` | - | - | - | ijoy status `+0x58 == 5`; 18 callsites |
| 0x005ae580 | `force_feedback_effect_create_on_device` | - | - | - | Creates the device effect from `+0x14`, `+0x18`, `+0x20`, `+0x1c` |
| 0x005ae5b0 | `force_feedback_effect_constant_upload_params` | - | - | - | Slot 0, constant |
| 0x005ae5d0 | `force_feedback_effect_ramp_upload_params` | - | - | - | Slot 0, ramp |
| 0x005ae660 | `force_feedback_effect_periodic_upload_params` | - | - | - | Slot 0, periodic |
| 0x005ae690 | `force_feedback_effect_condition_upload_params` | - | - | - | Slot 0, condition |

### 3d. Free helpers

The unit holds no free helper.

## 4. How the subsystem is identified

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
   **`LSIIEFFECT`** (= `DIEFFECT`) — see §9b. So the DirectInput reading is the developers'
   own vocabulary, not an inference.

## 5. Where the singleton is defined

The instance and the class live in different files. The singleton at `0x006f91e0` is a
**file-scope object of `Main.cpp`**, and its four CRT stubs sit at the very end of that TU's
initializer block. The four stubs are the `Main.cpp` rows of §3a. The cross-build placement is in §9c. The same TU defines the joy device vector. The receiver is
byte-proven from both stubs' `mov ecx, 0x6f91e0`; the dtor is reached only as
`atexit(&force_feedback_manager_static_dtor)`.

The class's 30 method bodies are implemented in the TU at `[0x005ae4b0, 0x005af290)` (§3c).

### 5a. What `+0x2d` distinguishes

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

## 6. Device-slot arbitration — why `+0x24` is a priority

The device has a finite number of effect slots. `force_feedback_manager_play_effect`
loops the download and, on each failure, evicts:

1. `force_feedback_manager_find_evictable_effect` picks a victim by a **two-tier rule** —
   an entry that is **no longer playing** is taken immediately, priority ignored;
   otherwise the entry with the **lowest** `+0x24`.
2. `force_feedback_manager_stop_effect` unloads it (the record survives, it just moves
   back to the idle list).
3. If no victim is available, or the victim outranks the newcomer, the effect is **parked
   pending** (`+0x29 = 1`) and `force_feedback_manager_retry_pending_effects` — driven by
   `..._release_effect` and by the tick — will try it again, highest `+0x24` first, and only an effect whose
   `duration_us` is 0 (an infinite effect).

🔴 **`+0x24` is a PRIORITY, not a magnitude.** It is never transmitted: the device create
call sends only `+0x14/+0x18/+0x20/+0x1c`, and the type overrides send only `+0x34…`. Its
only two readers are the two selectors above. The real strength is the float at `+0x34`.

## 7. The effect class family — five classes, one virtual each

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

## 8. The effects the game builds

### 8a. The four human effects — `human_create_force_feedback_effects`

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

### 8b. The five car effects — `car_create_force_feedback_effects`

`car_release_force_feedback_effects` releases all five.

| Global | Type | Priority | What drives its gain in `car_update` |
|---|---|---|---|
| `g_ff_car_effect_wheel_friction` | 10 (Condition) | 10 | accumulates over the wheel-contact array `car+0xd24` bounded by `car+0x5b0`, then falls with speed `car+0x688` (floor 0.34) |
| `g_ff_car_effect_lateral_spring` | 7 (Condition) | **300** | the condition offset is the angle between the velocity `car+0x1f7c` and `axis_forward` `car+0xd28` (π minus it when moving backwards) divided by `steer_lock_now` `car+0x6fc`, clamped to 1, signed by the velocity's side on `axis_right` `car+0xd34`; gain `min(speed × 0.5, 1) × wheel-friction weight × master × load` |
| `g_ff_car_effect_impact` | 0 (Constant), 220 ms | 201 | the change of the car-space velocity `car+0x1f7c` against `g_car_ff_prev_velocity` (horizontal part, × 0.2, clamped to 1.0) scales the gain; direction from `vec2_signed_angle_diff() + π` — a short directional jolt; played only when the clamped value exceeds 0.5 |
| `g_ff_car_effect_speed_vibration` | 2 (Periodic) | 200 | **period** re-set each frame from `(car+0x688 − 3.0) × 0.04761905` clamped 0..1; the gain carries the mean surface roughness of the wheel contacts (surface 2 / 8 0.3, 5 0.7, 6 0.4, 7 / 0xd / 0xe / 0xf 1.0, 10 0.2, 0x25 0.15) and the load |
| `g_ff_car_effect_steady_vibration` | 2 (Periodic) | 0 | constant 100 ms period; played only while the engine runs (`car+0xd18` = `C_Vehicle+0xca8`, `structs/car.md` §4a); gain `weight × master × 0.24`, re-sent only when the weight changes |

All five gains are `weight × g_ff_car_gain_master × <quantity>`, with per-effect weights
`g_ff_car_gain_*` and two change-detect caches `g_ff_car_last_gain_*`. The whole block is
gated on `car+0x594` = `C_Vehicle+0x524`, the steering axis's direct-input flag
(`structs/vehicle.md` §2c): for the player's car it is set while steering comes from the
SECONDARY key binding; otherwise all five are stopped.

### 8c. Where the weights come from — the Controls screen's Force Feedback sliders

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
| 246 Player Movement | 3006 | `g_force_feedback_strength` | `+0x514` | scales every human effect (§8a) |

Apart from that call, the five car sliders are referenced only by `gui_options_controls_on_exchange`
and `gui_options_controls_tick`, which push them into the widgets and read them back.

## 9. PC vs PS2

### 9a. PS2 has the same role in a different implementation — but **no function twin**

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

### 9b. The interfaces the PC keeps in a DLL, and the PS2 compiled in

`LSIInputDevice`, `ILSIInputEffect` and `LSIIEFFECT` **are PC class names** — they simply
live in `ijoy.dll`, not in `Game.exe`, so no PC function will ever be nameable for them.
The PS2, having no DLLs, statically links its own implementations, which is why those
symbols exist there. `basic/artifacts/dll.md` has the whole picture, including the general
consequence: a PS2 method with no PC twin may simply be in a DLL.

### 9c. Which source file the PC TU is

PS2's link order is `… C_DropOut · C_ff_manager · C_ff_manager::S_effect · C_grenade …`, and
`__sinit_Main.cpp`'s ninth slot is `__ct__12C_ff_managerFv(0x50aa50)` — the statement matching
the PC singleton's static ctor (§5), the ninth and last slot of a nine-of-nine ordered lockstep
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

### 9d. The PS2 methods

The PS2 destructor frees one vector where the PC's frees two, so `folded_vector_pair_free_at_0x04_0x14` carries no `C_ff_manager` twin. Every
method below is a role analogue at most.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00161f30 | `SetVibratoion__12C_ff_managerFUsUs` | the DualShock mixer of §9a, role analogue only |
| 0x00161fb0 | `__ct__12C_ff_managerFv` | the DualShock mixer of §9a, role analogue only |
| 0x00162000 | `__dt__12C_ff_managerFv` | the DualShock mixer of §9a, role analogue only; frees one vector |
| 0x00162070 | `GameInit__12C_ff_managerFv` | the DualShock mixer of §9a, role analogue only |
| 0x001620d0 | `GameDone__12C_ff_managerFv` | the DualShock mixer of §9a, role analogue only |
| 0x00162110 | `Tick__12C_ff_managerFUi` | the DualShock mixer of §9a, role analogue only |
| 0x00162580 | `SetConstVibration__12C_ff_managerFiUiUsUs` | the DualShock mixer of §9a, role analogue only |
| 0x001625f0 | `SetLinearVibration__12C_ff_managerFiUiUsUsUsUs` | the DualShock mixer of §9a, role analogue only |
| 0x00162680 | `ClearVibrations__12C_ff_managerFv` | the DualShock mixer of §9a, role analogue only |

## 10. Related

`basic/artifacts/dll.md` (`ijoy.dll`) · `structs/human.md` · `structs/car.md` · `structs/vehicle.md` §2c · `structs/loadsave.md` (the sliders' storage) ·
`structs/pursuit_event.md` §1 (the other owner of the destructor fold).

## 11. Open items

| # | Item | State |
|---|---|---|
| 1 | In game | whether force feedback does anything without a rumble-capable pad, and whether the CAR flag `+0x2d` corresponds to a second settings toggle |
| 2 | `unread_04` | no reader in the effect family's functions; a reader through another path is not ruled out, and its purpose is not recoverable from the binary |
| 3 | The condition block | `S_ff_condition` is typed [M] in `DICONDITION` order from the PC fill values (§1b); the code that reads it is in `ijoy.dll`, outside `Game.exe` |
