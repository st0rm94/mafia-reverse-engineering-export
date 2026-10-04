# `C_Vehicle` — the vehicle physics / drivetrain object

**Created** 2026-07-11 13:22 +0200 · **Modified** 2026-10-01 08:56 +0200

The physics and drivetrain component of a car: gears, throttle ramp, brake, steering, handbrake,
engine ignition, fuel, size and pivot geometry, wheels, body deformation, doors, windows, lights and
the whole vehicle sound bank. The chase AI (`C_driver`, `structs/driver.md`) drives a car entirely
through this object's setters.

| | |
|---|---|
| PC prefix | **`vehicle_`** |
| PC instance | **embedded at `C_car + 0x70`** — not a pointer field |
| PC code | four translation units spanning `[0x00509430, 0x0053c2b0)`, plus an inline-accessor block at `0x00425440`/`0x004f5a30`/`0x005c8740` (§6) |
| PC functions | **197** carrying the `vehicle_` prefix |
| PS2 class | `C_Vehicle`, **153 methods**: 133 with a PC twin, 20 without (§3a) |
| Owner | `C_car` — a **different class**, documented in `structs/car.md` (§5) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

**Why `C_car + 0x70` is the base.** `vehicle_set_engine_on` is reached as `lea ecx,[reg + 0x70]` at
11 of its 28 callsites (`ebx+0x70` ×6, `eax+0x70` ×2, plus `esi`, `edx`, `ebp`); `vehicle_set_power`
shows the same form; and `vehicle_get_engine_rpm` matches exactly, `lea ecx,[eax + 0x70]` with `eax`
the `C_car`. The fuel field this object gates ignition on lands at `C_car+0xd1c`, which is where the
HUD producer reads fuel.

**Two bases are in play, and they must not be mixed.** The fields below are `C_Vehicle`-relative;
the high-level readouts in `structs/car.md` §1a are `C_car`-relative.

| Fields | Base | How it is established |
|---|---|---|
| the drivetrain fields of §1 | the **`C_Vehicle`** (= `C_car+0x70`) | `+0xca8` ignition is read by `vehicle_set_engine_on`, whose `this` is byte-proven `C_car+0x70` |
| the high-level readouts (`car.md` §1a) | the **`C_car`** | the HUD instrument producer reads `+0x688` speed, `+0x648` gear and `+0x5b8` off an entity whose kind field `+0x10` is **4 = C_car**, with no `+0x70` step |

Some fields are therefore reachable under both spellings — `car+0x20c − 0x70 = C_Vehicle+0x19c` is
the light-state word, and `car+0x688 − 0x70 = C_Vehicle+0x618` is the scalar speed.

## 1. Drivetrain / control field layout (relative to the `C_Vehicle` base)

```c
typedef struct C_Vehicle {                  // partial; offsets relative to the C_Vehicle base = car+0x70
    /* +0x138 */ uint32_t gear_drive_state;     // [H] 0 & 2 = forward, 1 = reverse (SetPower/SetBrake/SetGear gating)
    /* +0x19c */ uint32_t light_state;          // [H] one bit per lamp/mode - see 1b
    /* +0x1D8 */ float    size_min[3];          // [H] bound-box min (vehicle_get_size(0))
    /* +0x1E4 */ float    size_max[3];          // [H] bound-box max (vehicle_get_size(1))
    /* +0x1F0 */ float    pivot_min[3];         // [H] pivot pos (vehicle_get_pivot_pos(0))
    /* +0x1FC */ float    pivot_max[3];         // [H] pivot pos (vehicle_get_pivot_pos(1))
    /* +0x218 */ float    steer_linearity;      // [H] steering linearity [0,1]
    /* +0x21C */ float    clutch_linearity;     // [M] clutch linearity (SetClutch reads)
    /* +0x224 */ uint32_t master_flags;         // [H] bit 3 unstable, bit 0x10000000 locked,
                                                //     bit 0x2000000 contact latch (see 2a);
                                                //     SetPower/SetBrake clear bit 0x1000000
    /* +0x364 */ void    *callbacks[14];        // [H] the owner callback bank, +0x364..+0x398 - see 8
    /* +0x420 */ float    gas_forward;          // [H] forward idle-gas channel
    /* +0x424 */ float    gas_reverse;          // [H] reverse-gas channel (SetPower param_3 == 0)
    /* +0x42C */ float    handbrake_force;      // [H] handbrake output scale (PC only)
    /* +0x430 */ float    handbrake_ramp_max;   // [H] handbrake ramp denominator
    /* +0x434 */ float    handbrake_ramp_accum; // [H] handbrake ramp accumulator
    /* +0x444 */ float    throttle_ramp_max;    // [H] throttle-ramp denominator
    /* +0x448 */ float    throttle_ramp_accum;  // [H] throttle-ramp accumulator
    /* +0x4AC */ uint8_t  horn_on;              // [H] see 1a
    /* +0x4AD */ uint8_t  siren_on;             // [H] see 1a
    /* +0x4AE */ uint8_t  sounds_enabled;       // [H] vehicle_enable_sounds; calls SoundOff when cleared
    /* +0x4B0 */ float    handbrake_out;        // [H] = (ramp_accum / ramp_max) * handbrake_force
    /* +0x50D */ uint8_t  camera_inside;        // [H] selects the interior or exterior sound handle
    /* +0x510 */ uint32_t shift_scratch;        // [M] cleared by SetGear on a successful shift
    /* +0x520 */ float    speed_limit;          // [H] AI speed limiter (+0.5 when > 0)
    /* +0x524 */ uint8_t  steer_smooth_enable;  // [H] set: SetSteer applies the linearity curve,
                                                //     clear: the raw angle - see 1c
    /* +0x525 */ uint8_t  accel_input_flag;     // [H] written like the others, never read - see 1c
    /* +0x526 */ uint8_t  ramp_bypass;          // [H] set: SetPower/SetBrake apply the pedal
                                                //     directly, clear: through the ease-in ramp
    /* +0x527 */ uint8_t  clutch_smooth_enable; // [H] set: SetClutch applies the linearity blend
    /* +0x528 */ void    *body_coll_heads;      // [H] vector<S_COLBODY_HEAD>, stride 0x74
    /* +0x540 */ uint32_t wheel_count;          // [H] length of the wheel array below
    /* +0x544 */ float    ramp_rate;            // [H] per-call accumulator step (throttle + handbrake)
    /* +0x548 */ uint8_t  clutch_gate;          // [M] 0 = idle/clutch-out path, != 0 = driven
    /* +0x5BC */ float    throttle_out_forward; // [H] from +0x420
    /* +0x5C0 */ float    throttle_out_reverse; // [H] from +0x424
    /* +0x5C4 */ float    engine_revs_per_sec;  // [H] engine speed in rev/SEC; vehicle_get_engine_rpm
                                                //     returns it * 60 when +0xca8 is set, else 0.
                                                //     Feeds the HUD tachometer.
    /* +0x5D8 */ int32_t  gear_effective;       // [H] SetPower ramps only when < 0 (neutral)
    /* +0x5E0 */ float    redline_revs_per_sec; // [H] CARINIT RedLineRPM / 60
    /* +0x5DC */ int32_t  gear_selected;        // [H] SetGear writes; range -1..gear_max
    /* +0x5E4 */ int32_t  gear_max;             // [H] max gear index / gear count
    /* +0x5EC */ float    gear_ratios[];        // [M] per-gear ratio table, indexed by gear
    /* +0x618 */ float    speed;                // [H] the scalar speed; = car+0x688
    /* +0x634 */ float    drivetrain_force_out; // [H] net wheel longitudinal force
                                                //     = (ramp_accum / ramp_max) * pedal; written by
                                                //     SetPower (neutral) AND SetBrake (in-gear)
    /* +0x65C */ float    clutch_out;           // [M] clutch pedal output
    /* +0x664 */ uint32_t flags_664;            // [M] SetGear bit ops
    /* +0x668 */ float    idle_revs_per_sec;    // [H] CARINIT idling rpm / 60 - see 2
    /* +0x68C */ float    steer_scale;          // [H] steering scale; +0x6a0 = angle * this
    /* +0x6A0 */ float    steer_out;            // [H] applied steering output
    /* +0x6B4 */ uint8_t  engine_on_mirror;     // [H] mirror of engine_on
    /* +0xC9C */ float    engine_timer;         // [M] 5000.0f, reset by the SetEngineOn force path
    /* +0xCA0 */ float    cut_time;             // [M] shift cut-time = int(5..50) / 1000
    /* +0xCA8 */ uint8_t  engine_on;            // [H] engine-running flag
    /* +0xCAC */ float    fuel_level;           // [H] ignition refused below 0.02
    /* +0xCB4 */ void   **wheels;               // [H] wheel pointer array - see 1a
    /* +0xCB8 */ float    basis_forward[3];     // [H] world basis, seeded (0,0,1)
    /* +0xCC4 */ float    basis_right[3];       // [H] world basis, seeded (1,0,0)
    /* +0xCD0 */ float    basis_up[3];          // [H] world basis, seeded (0,1,0)
    /* +0xCE4 */ uint32_t flags_ce4;            // [M] & 0x4000 selects the alt engine-start/gear path;
                                                //     bit 0x20 gates the handbrake sound;
                                                //     bit 0x200 gates the body-collision particles
    /* +0xD20 */ void     *door_records;        // [H] stride 0x24 - see 2a
    /* +0xDE8 */ S_CARINIT init;                // [H] 4388 bytes, the whole per-car tuning block
    /* +0x1F0C */ float   velocity[3];          // [H] the linear velocity; = car+0x1f7c
    /* +0x1F18 */ float   max_revs_per_sec;     // [H] CARINIT maximum rpm / 60 - see 2
    /* +0x1F1C */ float   fuel_capacity;        // [H] tank capacity (SetFuel clamp)
} C_Vehicle;                                    // partial
```

### 1a. Horn, siren, wheels and the world basis

These fields were pinned while naming the out-of-line inline members hosted by the `C_sledovacka`
translation unit. Every receiver is byte-proven `C_car + 0x70` by reading the `lea`/`add` that builds
`ecx` at each callsite.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x4ac` | horn_on | H | `vehicle_set_horn` | the sound layer | named by the PS2 build (§7) |
| `+0x4ad` | siren_on | H | reset paths only | `vehicle_get_siren` | directly under `+0x4ae sounds_enabled`, so the triple is complete |
| `+0x540` | wheel_count | H | `vehicle_apply_carinit_params` | `vehicle_get_wheel_count`, `vehicle_wheel_contact_solve`, `vehicle_game_done`, `vehicle_save_state`, `vehicle_load_state`, `vehicle_get_save_state_size` | length of the array below |
| `+0xcb4` | wheels[] | H | `vehicle_apply_carinit_params` (87 accesses — it builds them from the CARINIT record) | the same set, plus `vehicle_get_wheel_at` | array of wheel pointers. Element: `+0x04` `I3D_frame*`, `+0x1c` world hub position, `+0x10c` radius, `+0x120` flag word |
| `+0xcb8` | basis_forward (world) | H | `vehicle_reset_from_frame` | the wheel solver | seeded `(0,0,1)`, then multiplied by the body matrix |
| `+0xcc4` | basis_right (world) | H | `vehicle_reset_from_frame` | the wheel solver | seeded `(1,0,0)` |
| `+0xcd0` | basis_up (world) | H | `vehicle_reset_from_frame` | `vehicle_get_up_axis`; `vehicle_wheel_contact_solve` reads it 15× | seeded `(0,1,0)`. The contact point is `hub − radius · up`, which is the corroboration |
| `+0x5bc` | throttle_out_forward | H | the throttle path (from `+0x420`) | `vehicle_is_throttle_applied`, `vehicle_save_state`, `vehicle_load_state` | the gate `+0x634 > 0.9 && +0x5bc > 0.0` |
| `+0x5c0` | throttle_out_reverse | H | the throttle path (from `+0x424`) | the same | - |

**Wheel flag `+0x120 & 0x40000000` is what makes the chase AI give up:** `pursuit_event_tick` walks
the wheel array and, if any wheel carries that bit, drops the record to a terminal state.

`vehicle_get_forward_ptr` hands out `+0xcc4`. The axis is settled by a full three-term dot against
the velocity vector (`vel·fwd < 0.0` = "moving backwards") and by the seat-swap check, which negates
it per seat through `car_get_seat_property` and treats `dot < −0.25` as "the target is behind this
seat". It is not a row of the body matrix — that is `car+0xdd4`.

### 1b. Light state — the `+0x19c` flag word and its notify callback

Every exterior lamp on a car is one bit of a single flag word on the **`C_Vehicle`**, not on the
`C_car`. The same word is reachable as `C_car+0x20c`, which is the spelling `car_update` and
`car_deactivate` use.

| Bit | Meaning | Conf | Established by |
|---|---|---|---|
| `0x1` | LEFT turn indicator | H | script case `0xc2` `carlight_indic_l`; the Mafia Script Data Reference gives *"left indicator"*; confirmed in game |
| `0x2` | RIGHT turn indicator | H | case `0xc3`, resolver-pinned (`script-mapping/artifacts/opcodes-verified.md`); the Mafia Script Data Reference gives *"right indicator"*; confirmed in game |
| `0x8` | indicators driven **automatically from the steering angle** | H | gates both side setters; the per-frame light update owns the lamps while it is set |
| `0x80` | unidentified | U | script case `0xc1` `carlight_light`; the Mafia Script Data Reference guesses *"hazard lights ???"*, but testing it in game produced no visible effect. Its setter is therefore named `vehicle_set_light_bit_0x80` |
| `0x100` | brake lights | M | set when a float field is positive (brake pedal) |
| `0x800` | gates `0x100` | U | no script setter found |
| `0x1000` | reverse lights | M | set when a signed field is negative (reverse gear) |
| `0x10000` | main headlights | H | case `0xc0` `carlight_main`, confirmed in game; auto-driven by `car_update` while `C_game+0x2d44` (night mission) is set |
| `0x0ff00000` | script light-override sub-field | H | read and modified by `car_update`, cleared by `car_deactivate`; read by `vehicle_get_light_script_override`, written by `vehicle_set_light_script_override`. **Each override bit belongs to one lamp** — `0x100000` the indicators, `0x200000` the `0x80` lamp, `0x800000` the headlights — and each is set exactly when its `carlight_*` command is given the argument **2** (force) |

Bit `0x800` gates bit `0x100`, and bit `0x8` gates bits `0x1`/`0x2`; both are confirmed independently
by `driver_do_lights`, the chase AI's light logic, which also **flashes** bit `0x10000` on a timed
pattern after calling `car_set_auto_headlights_suppressed(1)`.

**The notify hook.** Every setter that actually changes a lamp calls the function pointer at
`+0x388` when it is non-null — that is how a light change wakes a sleeping car. `+0x388` is the
wake-up slot of the callback bank, so the light setters are consumers of it, not its owner
(§8a). `vehicle_set_auto_indicators_from_steering` is the one family member that does not notify.

### 1c. The four per-axis input flags `+0x524..+0x527`

One byte per driving axis. Set, the setter treats its input as an ANALOG value (shaped by the
linearity coefficient, or applied without the ramp); clear, as a digital full-scale value. The
same bytes are `C_car+0x594..+0x597`.

| Byte | Axis | Reader and effect | Writers |
|---|---|---|---|
| `+0x524` | steer | `vehicle_set_steer` (linearity `+0x218`), `vehicle_update_steering_wheels`; `car_update` runs force feedback only while it is set | ¹ |
| `+0x525` | accelerate | none located | ¹ |
| `+0x526` | brake | `vehicle_set_power` **and** `vehicle_set_brake` both skip the ramp `+0x444`/`+0x448` when set | ¹ |
| `+0x527` | clutch | `vehicle_set_clutch` (linearity `+0x21c`), `vehicle_tick_drivetrain` | ¹ |

¹ `player_ai_drive` writes all four every frame the player drives: 1 while that axis comes from its
SECONDARY key binding, 0 once the PRIMARY binding has taken over (`g_player_drive_*_secondary_latch`,
`structs/player.md`). `driver_tick` writes 0/0/0/0 or 1/1/1/0, `traffic_car_activate_elements`
1/1/1/1, `car_net_snapshot_state` 0 or 0/1/1/1; `vehicle_game_init` zeroes them and the save-state
pair round-trips them. The ramp gate for the THROTTLE is the brake axis's byte `+0x526`; `+0x525`
is written and never read.

## 2. Field evidence — drivetrain

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x138` | gear_drive_state | H | the drivetrain | `vehicle_set_power`, `vehicle_set_brake`, `vehicle_set_steer` | 0/2 forward, 1 reverse |
| `+0x420` / `+0x424` | gas forward / reverse | H | `vehicle_set_power`, `vehicle_set_brake`, `vehicle_set_engine_on` | the integrator | channel chosen by gear and by `param_3` |
| `+0x42c` / `+0x430` / `+0x434` / `+0x4b0` | handbrake force / ramp / accum / out | H | `vehicle_set_handbrake` | the drivetrain | `+0x4b0 = (accum / max) · force` |
| `+0x444` / `+0x448` / `+0x544` / `+0x526` | throttle ramp max / accum / rate / bypass | H | `vehicle_set_power`, `vehicle_set_brake` | the drivetrain | one shared ease-in |
| `+0x520` | speed_limit | H | `vehicle_set_speed_limit` | the drivetrain | `driver_tick` sets it to the lookahead distance |
| `+0x5d8` / `+0x5dc` / `+0x5e4` / `+0x5ec` | gear effective / selected / max / ratios | H | `vehicle_set_gear` writes `+0x5dc` | `vehicle_set_power` (`+0x5d8 < 0` = neutral), `vehicle_get_max_engine_torque_at_rpm` | - |
| `+0x634` | drivetrain_force_out | H | `vehicle_set_power` (neutral), `vehicle_set_brake` (in-gear) | the integrator | the net applied wheel force |
| `+0x5c4` / `+0x5e0` / `+0x668` / `+0x1f18` | engine revs / red-line / idle / maximum, all per second | H | `vehicle_update_motor_revs` writes the revs and clamps them to `+0x1f18`; `vehicle_apply_carinit_params` sets the other three from the CARINIT `RedLineRPM`, `idling rpm` and `maximum rpm` ÷ 60 (`def-files-docs/vehicles-bin.md`) | `vehicle_get_engine_rpm`, `vehicle_compute_moment`, `vehicle_update_strips`, `vehicle_update_engine_pitch`, `car_extended_update` | `vehicle_update_strips` and `car_extended_update` use the fraction `(revs − idle) / (max − idle)` |
| `+0x68c` / `+0x6a0` / `+0x218` / `+0x524` | steer scale / out / linearity / enable | H | `vehicle_set_steer`, `vehicle_set_steering_linearity` | the drivetrain | `+0x6a0 = angle · +0x68c`; `+0x524` in §1c |
| `+0x65c` / `+0x21c` / `+0x527` | clutch out / linearity / enable | H | `vehicle_set_clutch` | the drivetrain | `+0x527` in §1c |
| `+0xca8` / `+0x6b4` / `+0xc9c` | engine_on / mirror / timer | H | `vehicle_set_engine_on` | `vehicle_set_power`, `vehicle_set_brake`, `vehicle_set_gear` | ignition |
| `+0xcac` / `+0x1f1c` | fuel / capacity | H | `vehicle_set_fuel` | `vehicle_set_engine_on` (the `< 0.02` gate) | - |
| `+0xca0` | cut_time | M | `vehicle_set_cut_time` | the shift logic | `int(5..50) ÷ 1000` |
| `+0x4ae` | sounds_enabled | H | `vehicle_enable_sounds` | `vehicle_set_engine_on`, every sound member | calls `SoundOff` when cleared |
| `+0x1d8` / `+0x1e4` | size_min / max | H | the geometry init | `vehicle_get_size`, `driver_set_car` | bound extents |
| `+0x1f0` / `+0x1fc` | pivot_min / max | H | the geometry init | `vehicle_get_pivot_pos`, `driver_set_car` | - |

### 2a. Fields identified from the translation-unit sweeps

| Field | What | How |
|---|---|---|
| `+0x140` | the lock reference count | `vehicle_lock` ±1 |
| `+0x17c` / `+0x180` | the light **vector object** and its begin pointer; stride `0x20`, entry `+0x10` is the type (1/2 = projector) | byte-proven: `lea edi,[ebp + 0x17c]` inside `vehicle_apply_light_state`, and 285 of `vehicle_light_at`'s 301 callsites read that register. `vehicle_set_projector_size` walks it for six selector bits |
| `+0x18c` / `+0x190` | the **special light projector** vector object and its begin pointer; stride `0x1c`, entry `+0x00` is the `I3D_frame*`, `+0x18` a ushort state word | byte-proven: `lea ecx,[ebp + 0x18c]` at 10 of `vehicle_special_projector_at`'s 51 callsites. Searched by `vehicle_switch_special_projector_texture`, reset by `vehicle_init_damage`. The two vector objects are adjacent — `0x17c + 0x10 = 0x18c` |
| `+0x1e4` / `+0x1e8` / `+0x1ec` | the position-quantiser ranges (x, y, z) | `vehicle_decode_position` |
| `+0x1fc` / `+0x200` / `+0x204` | the position-quantiser origins | the same |
| `+0x28` | the contact latch (byte) | raised by `vehicle_wheel_contact_solve` and `vehicle_body_coll_resolve` when a contact's collision record carries flag `0x10000`; `vehicle_update_visuals` copies it into bit `0x2000000` of `+0x224` and clears it; `vehicle_game_init` clears it |
| `+0x224` | the master flag word — bit 3 unstable, bit `0x10000000` locked, bit `0x2000000` the copied contact latch | `vehicle_is_unstable`, `vehicle_lock`; `vehicle_update_visuals` sets `0x2000001` or clears bit 25; `vehicle_wheel_contact_solve` and `human_find_blocking_drawbridge` read bit 25 |
| `+0x22c` / `+0x230` | the engine-damage floor and the motor-force factor | `vehicle_apply_carinit_params` writes `+0x22c`; `car_hit` and `vehicle_apply_damage` write `+0x230 = +0x22c + (1 − +0x22c)·(pool/max)` over the pool below; `vehicle_compute_motor_force` multiplies by `+0x230`; `vehicle_init_damage` resets it to 1.0 |
| `+0x234` / `+0x238` | the **engine** damage pool, current / maximum | condition channel 1 (`structs/car.md` §1b). `vehicle_apply_damage` subtracts a hit above the threshold `+0x244`, reduced by the resistance percentage `+0x23c` and scaled by `+0x238 / +0x240`, clamps at 0, and latches `+0x66c = 1` once the pool is empty |
| `+0x26c` / `+0x270` | damage pool 0, current / maximum | condition channel 0 (`structs/car.md` §1b). `vehicle_apply_damage` applies the same formula with threshold `+0x27c`, resistance `+0x274` and divisor `+0x278` |
| `+0x288` | the **pristine-geometry backup**, a parallel `0x34`-stride copy of the deform-zone array | built by `vehicle_prepare_deform_zone`, restored from by `vehicle_init_deform`, measured by the three save-state functions |
| `+0x298` | the **deform-zone array**, stride `0x34`, one zone per model frame tagged `[DEFORM BODY]` … `[DEFORM ROOF]` (types 1..9, `structs/car.md` §4c); the model name is at part `+0x100`, the flags at part `+0x0e` | built by `vehicle_cb_enum_deform_zones` from `vehicle_init`'s frame walk; read by `vehicle_get_part_name_tag` |
| `+0x354` | the driver's head position | `vehicle_get_head_pos`, consumed by `camera_tick` and `camera_begin_mode` |
| `+0x414` / `+0x418` / `+0x41c` | the centre of mass | `vehicle_get_contact_impulse_terms` |
| `+0x49c` / `+0x4a0` / `+0x4a4` | the angular velocity | the same; `vehicle_settle` zeroes it on the way out |
| `+0x50d` | the camera-is-inside-the-car flag | every sound member selects its interior or exterior handle on it (§6e) |
| `+0x528` | the body-collision head vector — `vector<S_COLBODY_HEAD>`, stride `0x74` | `vehicle_game_init` passes it to `vehicle_clear_body_coll_add_track`; built by `vehicle_cb_enum_body_colls` |
| `+0x5e4` / `+0x5ec` | the gear count and the gear-ratio array | `vehicle_get_max_engine_torque_at_rpm` |
| `+0xd20` | the door/window record array, stride `0x24`; the record object hangs at `+0x20` | **one array serves BOTH doors and windows**: `vehicle_animate_doors` reads the door angle band on the object, while `vehicle_open_window`, `vehicle_animate_windows` and `vehicle_is_window_open` read the window band on the SAME object. A car has one record per door, so the door count, the window count and the count `vehicle_get_seat_count` returns all coincide (§11) |
| door `+0x24` / `+0x28` / `+0x2c` / `+0x30` | current angle / maximum angle / opening speed / flags | `vehicle_open_door`, `vehicle_set_max_door_angle`, `vehicle_set_door_open_speed` |
| `+0xd2c` | the WINDOW list — a `0x58`-stride vector, one record per `WINDOW*` frame | byte-proven `lea ecx,[ebx + 0xd2c]` in `vehicle_init`; consumed by `vehicle_open_window`, `vehicle_animate_windows`, `vehicle_is_window_open` |
| `+0xd40` / `+0xd44` | the **wheel vector** — `0x1c`-byte records, the second input to the mass computation | `vehicle_set_seat_mass` (§7) |
| `+0xd64` / `+0xda4` | the cached world matrix and its inverse | `vehicle_reset_from_frame` |
| `+0xde8` | **`S_CARINIT`, 4388 bytes** — the whole per-car tuning block | `vehicle_get_carinit_params` copies it out; the type comes from the PS2 parameter |
| `+0xde8` `+0xc84`..`+0xee4` | the `S_CARINIT` **sound-name table**, stride `0x10` | `vehicle_init_sounds` reads each entry as a filename — the vehicle sound names are data, not literals (§6e) |
| `+0xdf8` / `+0xdfc` | the rpm band the torque lookup accepts | `vehicle_get_max_engine_torque_at_rpm` |
| `+0x1f0c` / `+0x1f10` / `+0x1f14` | the linear velocity | `vehicle_get_contact_impulse_terms` |
| `+0x1f26` / `+0x1f38` | the special-collider mode byte and the stride-`0x48` vector OBJECT (`+0x1f3c` is that vector's begin pointer) | `vehicle_set_special_dynamic_colls` sets the mode, `vehicle_set_dynamic_colls_off` reads it; byte-proven `lea ecx,[ebp + 0x1f38]` in `vehicle_game_init` |

## 3. `C_Vehicle` method coverage — PC first

Every PC function carrying the `vehicle_` prefix gets a row, ordered by PC VA, including those with
no PS2 twin. Status: **CONFIRMED** = the bodies agree · **STRUCTURAL** = the mapping is certain from
position and content but the bodies cannot be compared · **PC-only** = no PS2 counterpart, because
the work is inlined there or the mechanism does not exist on that build. `-` means "none".

The twin column gives the PS2 name; unless it is written without a class, the method belongs to
`C_Vehicle`. `this` is always a `C_Vehicle` (= `C_car+0x70`).

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role / notes |
|---|---|---|---|---|---|
| 0x00425440 | `vehicle_get_speed` | - | - | PC-only | `return *(float *)(this + 0x618)` — the **scalar speed**, the field `car.md` §1a documents at `car+0x688` and the police speed gate reads. Consumers compare it against **0.5** ("is the target's car moving at all?") and **2.0** (`ai_fight_followship_must_car_hunt`'s "we need a vehicle to catch this"). 40 references, 24 of them in `pursuit_event_tick` |
| 0x00425450 | `vehicle_get_velocity_ptr` | - | - | PC-only | returns `&this->+0x1f0c` = `car+0x1f7c`, the velocity vector `car_recompute_velocity` derives each frame and `car_get_actor_speed` copies out |
| 0x00425460 | `vehicle_get_seat_count` | - | - | PC-only | `(end − begin) / 0x24` over the array at `+0xd20`/`+0xd24`; 0 when unallocated. Three consumers feed the index straight into `car_get_owner(i)`. The seat RECORDS — occupancy, occupant, placements — are a **different** array at `car+0x210c`, stride `0xa0`; both share one index space (§11) |
| 0x00425490 | `vehicle_get_forward_ptr` | - | - | PC-only | returns `&this->+0xcc4` = `car+0xd34..+0xd3c`, the world-space forward axis (§1a) |
| 0x004f5a30 | `vehicle_get_engine_on` | - | - | PC-only | - |
| 0x004f5a40 | `vehicle_set_clutch_gate` | - | - | PC-only | - |
| 0x004f5a50 | `vehicle_set_horn` | - | - | PC-only | writes `+0x4ac` |
| 0x004f5a60 | `vehicle_get_siren` | - | - | PC-only | reads `+0x4ad` |
| 0x004f5a70 | `vehicle_is_throttle_applied` | - | - | PC-only | `+0x634 > 0.9 && +0x5bc > 0.0` |
| 0x004f5aa0 | `vehicle_get_up_axis` | - | - | PC-only | returns `+0xcd0` |
| 0x004f5ab0 | `vehicle_get_wheel_count` | - | - | PC-only | reads `+0x540` |
| 0x004f5ac0 | `vehicle_get_wheel_at` | - | - | PC-only | indexes `+0xcb4` |
| 0x00509460 | `vehicle_tyre_list_static_ctor` | - | - | PC-only | §6a |
| 0x00509490 | `vehicle_tyre_list_static_dtor` | - | - | PC-only | §6a |
| 0x005094e0 | `vehicle_material_list_static_ctor` | - | - | PC-only | §6a |
| 0x00509510 | `vehicle_material_list_static_dtor` | - | - | PC-only | §6a |
| 0x00509560 | `vehicle_particle_list_static_ctor` | - | - | PC-only | §6a |
| 0x00509590 | `vehicle_particle_list_static_dtor` | - | - | PC-only | §6a |
| 0x005095e0 | `vehicle_projector_sector_list_static_ctor` | - | - | PC-only | §6a |
| 0x00509610 | `vehicle_projector_sector_list_static_dtor` | - | - | PC-only | §6a |
| 0x00509660 | `vehicle_sound_sector_list_static_ctor` | - | - | PC-only | §6a |
| 0x00509690 | `vehicle_sound_sector_list_static_dtor` | - | - | PC-only | §6a |
| 0x005096d0 | `vehicle_ctor` | `{ctor}` | 0x0022d660 | CONFIRMED | - |
| 0x00509bb0 | `vehicle_init` | `Init` | 0x0022daf0 | CONFIRMED | builds the wheel set, axles, materials and particle emitters from the model root. Pinned by call position: `car_init` calls it between `actor_init` and the callback-bank fill, exactly where PS2 `Init__5C_car` calls it, and the `return 0` failure handling is identical |
| 0x0050b6f0 | `vehicle_init_strip_and_smoke_emitters` | - | - | PC-only | the PS2 inlined this into `Init` (§9) |
| 0x0050cc10 | `vehicle_init_axes` | `InitAxis` | 0x00231650 | CONFIRMED | - |
| 0x0050d0c0 | `vehicle_init_lights` | `InitLights` | 0x00232000 | CONFIRMED | calls `vehicle_create_light_projector` six times |
| 0x0050db60 | `vehicle_dtor` | `{dtor}` | 0x002337b0 | CONFIRMED | - |
| 0x0050e0c0 | `vehicle_create_light_projector` | `CreateLightProjector` | 0x002341a0 | STRUCTURAL | the PS2 symbol is an **8-byte stub** against 1456 bytes of PC code, so position and the `PROJ` magic settle the mapping but the bodies cannot be compared |
| 0x0050e670 | `vehicle_create_special_light_projector` | `CreateSpecialLightProjector` | 0x002341b0 | CONFIRMED | - |
| 0x0050ee60 | `vehicle_deactivate` | `Deactivate` | 0x00234860 | CONFIRMED | the PC inlined `SetDynamicCollsOn` into it (§9) |
| 0x0050f140 | `vehicle_reset_from_frame` | `Reset1` | 0x00234ca0 | CONFIRMED | seeds the three world basis vectors (§1a), rebuilds every wheel's world position and contact point, and caches the world matrix and its inverse at `+0xd64`/`+0xda4` |
| 0x0050f7f0 | `vehicle_reset` | `Reset` | 0x002351d0 | CONFIRMED | - |
| 0x00510150 | `vehicle_game_init` | `GameInit` | 0x00235b70 | CONFIRMED | passes `+0x528` to `vehicle_clear_body_coll_add_track` |
| 0x00511890 | `vehicle_game_done` | `GameDone` | 0x002383d0 | CONFIRMED | releases the wheels, particles and sound handles. Pinned by call position: `car_game_done` calls it between `vehicle_enable_sounds(0)` and `actor_game_done`, exactly where PS2 `GameDone__5C_car` calls it |
| 0x00511b50 | `vehicle_set_tyres` | `SetTyres` | 0x00238820 | CONFIRMED | the PS2 parameter type names the element `S_TYRE` |
| 0x00511eb0 | `vehicle_close_tyres` | `CloseTyres` | 0x00238a10 | CONFIRMED | - |
| 0x00511f40 | `vehicle_set_materials` | `SetVehicleMaterials` | 0x00238aa0 | CONFIRMED | the PS2 parameter type names the element `S_VEH_MATERIAL` |
| 0x00511fb0 | `vehicle_close_materials` | `CloseVehicleMaterials` | 0x00238ba0 | CONFIRMED | - |
| 0x00511ff0 | `vehicle_set_particles` | `AddParticle` | 0x00239300 | STRUCTURAL | the PC fuses `CloseParticles` + N × `AddParticle` into one `Set`-shaped function, so no single body agreement can be asserted. The PS2 parameter names the element `S_VEH_PARTICLE`, stride `0x1c` |
| 0x005122b0 | `vehicle_close_particles` | `CloseParticles` | 0x00239680 | CONFIRMED | clears the vector `vehicle_set_particles` fills; 144 B against 368 B at the forced rank |
| 0x00512340 | `vehicle_get_wheel_cam_pos` | `GetWheelCamPos` | 0x00239930 | CONFIRMED | - |
| 0x005123d0 | `vehicle_apply_carinit_params` | `InitCar` | 0x002399f0 | CONFIRMED | builds the wheel array from the CARINIT record — 87 accesses to `+0xcb4` |
| 0x00517370 | `vehicle_get_carinit_params` | `GetInit` | 0x0023f380 | CONFIRMED | copies the `S_CARINIT` block out of `+0xde8` |
| 0x005173a0 | `vehicle_set_gear` | `SetGear` | 0x0023f3c0 | CONFIRMED | `+0x5dc`: select gear, range −1..gear_max |
| 0x00517460 | `vehicle_set_power` | `SetPower` | 0x0023f4e0 | CONFIRMED | `+0x634` (neutral) / `+0x420` (idle): AI engine power |
| 0x00517600 | `vehicle_set_brake` | `SetBrake` | 0x0023f650 | CONFIRMED | `+0x634` (in-gear): brake through the drivetrain force. Byte-exact (§4) |
| 0x00517790 | `vehicle_set_clutch` | `SetClutch` | 0x0023f7b0 | CONFIRMED | `+0x65c`: clutch pedal, linearity blend |
| 0x00517800 | `vehicle_set_steer` | `SetSteer` | 0x0023f830 | CONFIRMED | `+0x6a0 = angle · +0x68c` |
| 0x005178a0 | `vehicle_set_steering_linearity` | `SetSteeringLinearity` | 0x0023f8e0 | CONFIRMED | `+0x218`, range [0,1] |
| 0x005178e0 | `vehicle_set_engine_on` | `SetEngineOn` | 0x0023f930 | CONFIRMED | `+0xca8`: ignition, refused when fuel < 0.02 |
| 0x005179d0 | `vehicle_set_speed_limit` | `SetSpeedLimit` | 0x0023fac0 | CONFIRMED | `+0x520`: the AI speed limiter |
| 0x00517a10 | `vehicle_set_fuel` | `SetFuel` | 0x0023fb00 | CONFIRMED | `+0xcac`, clamped against `+0x1f1c` |
| 0x00517a40 | `vehicle_set_handbrake` | `SetHandbrake` | 0x0023fb30 | CONFIRMED | `+0x4b0` through the ramp — the parking brake (§4) |
| 0x00517ab0 | `vehicle_enable_sounds` | `EnableSounds` | 0x0023fb90 | CONFIRMED | `+0x4ae`; calls `SoundOff` when cleared |
| 0x00517ae0 | `vehicle_set_cut_time` | `SetCutTime` | 0x0023fbe0 | CONFIRMED | `+0xca0 = int(5..50) / 1000` |
| 0x00517b10 | `vehicle_apply_external_impulse` | `ExternalImpuls` | 0x0023fc30 | CONFIRMED | - |
| 0x00517f00 | `vehicle_apply_external_force` | `ExternalForce` | 0x0023fea0 | CONFIRMED | - |
| 0x005180a0 | `vehicle_do_collision` | `DoCollision` | 0x0023ffa0 | CONFIRMED | - |
| 0x00518500 | `vehicle_get_contact_impulse_terms` | `GetRVelMassJ` | 0x002407f0 | CONFIRMED | the contact-response query: centre of mass `+0x414`, angular velocity `+0x49c`, linear velocity `+0x1f0c` |
| 0x00518670 | `vehicle_get_head_pos` | `GetHeadPos` | 0x00240900 | CONFIRMED | returns `+0x354`, consumed by `camera_tick` and `camera_begin_mode` |
| 0x00518680 | `vehicle_get_max_engine_torque_at_rpm` | `_GetMaxMomentByRot` | 0x00240910 | CONFIRMED | reads the gear count and ratio table `+0x5e4`/`+0x5ec` and the accepted rpm band `+0xdf8`/`+0xdfc` |
| 0x00518730 | `vehicle_set_seat_mass` | `SetSeatMass` | 0x00240a30 | CONFIRMED | §7 |
| 0x00518a90 | `vehicle_encode_position` | `CodePos` | 0x00240ed0 | CONFIRMED | quantises against the ranges `+0x1e4` and origins `+0x1fc` |
| 0x00518b40 | `vehicle_decode_position` | `DecodePos` | 0x00241080 | CONFIRMED | the inverse |
| 0x00518c10 | `vehicle_get_part_name_tag` | `CodeIndexDan` | 0x002411e0 | CONFIRMED | over the body-part array `+0x298` |
| 0x00518d10 | `vehicle_find_part_by_name_tag` | `DecodeIndexDan` | 0x00241310 | CONFIRMED | the inverse |
| 0x00518d70 | `vehicle_encode_part_position` | - | - | PC-only | quantises the world position of one body part by walking its frame chain up to `+0x3c8`, then calls `vehicle_encode_position`. No PS2 method exists at this rank |
| 0x00518f70 | `vehicle_find_part_at_encoded_position` | - | - | PC-only | the inverse: decodes the packed position and returns the nearest matching body-part index, optionally filtered on the short tag at part `+0x0c` |
| 0x005195e0 | `vehicle_get_engine_rpm` | `GetRotations` | 0x002413a0 | CONFIRMED | `+0xca8 ? +0x5c4 * 60.0 : 0.0`. Its sole consumer is the HUD tachometer needle, normalised by `car+0x1f88 * 60` (`basic/artifacts/hud_elements.md` §6b) |
| 0x00519600 | `vehicle_is_unstable` | `IsVehicleUnstable` | 0x002413d0 | CONFIRMED | bit 3 of `+0x224` |
| 0x00519610 | `vehicle_get_size` | `GetVehicleSize` | 0x002413f0 | CONFIRMED | returns `+0x1d8` / `+0x1e4` |
| 0x00519630 | `vehicle_get_pivot_pos` | `GetVehiclePivotPos` | 0x00241410 | CONFIRMED | returns `+0x1f0` / `+0x1fc` |
| 0x00519650 | `vehicle_lock` | `LockVehicle` | 0x00241430 | CONFIRMED | `+0x140` ±1 and bit `0x10000000` of `+0x224` |
| 0x00519690 | `vehicle_set_projector_size` | `SetProjectorSize` | 0x00241490 | CONFIRMED | walks the light vector `+0x17c` for six selector bits |
| 0x00519a70 | `vehicle_open_door` | `OpenDoor` | 0x00241930 | CONFIRMED | over the record array `+0xd20` |
| 0x00519b10 | `vehicle_set_max_door_angle` | `SetMaxDoorAngle` | 0x00241ad0 | CONFIRMED | door `+0x28` |
| 0x00519b30 | `vehicle_set_door_open_speed` | `SetSpeedDoorOpen` | 0x00241b00 | CONFIRMED | door `+0x2c` |
| 0x00519b60 | `vehicle_set_special_dynamic_colls` | `SetSpecialDynamicColls` | 0x00241b40 | CONFIRMED | sets the mode byte `+0x1f26` |
| 0x00519d30 | `vehicle_set_dynamic_colls_off` | `SetDynamicCollsOn` | 0x00241de0 | CONFIRMED | a change-guarded toggle on `+0x118` that inserts or erases the vehicle's dynamic-collision records — either the single record at `+0x2ac` or, when `+0x1f26` is set, every `0x48`-byte record in the vector at `+0x1f38`. The PS2 method takes two bools; the PC folds the second into the `+0x1f26` read, and its single parameter is the INVERSE of the PS2 `on` (1 = erase), hence `_off`. Callers: `traffic_car_change_state` (×2) and `traffic_car_registry_set_enabled` (×1), `pursuit_event_tick`, `police_board_release_all_group_vehicles`, `program_process_command` |
| 0x00519e50 | `vehicle_update_cached_transform` | `UpdateImportantVariables` | 0x002420d0 | CONFIRMED | - |
| 0x0051a150 | `vehicle_clear_body_coll_add_track` | `ClearBodyCollAddTrack` | 0x00242420 | CONFIRMED | zeroes the add_track vector at `S_COLBODY_HEAD+0x3c` and at each `0x3c`-stride sphere `+0x1c`, recursively over the tree at `+0x528`. The sole caller on both builds is `GameInit`. `add_track` is the developers' field name; what accumulates there is open (§11) |
| 0x0051a250 | `vehicle_add_projector_lights_to_sectors` | `AddProjectorLightsToSectors` | 0x002424c0 | CONFIRMED | walks the light array `+0x180` (stride `0x20`, light pointer at elem `+0x0c`); for each light not also in the special-projector array `+0x190` (stride `0x1c`) it calls `I3D_sector` slot `0x50` on every entry of the projector-sector list |
| 0x0051a3b0 | `vehicle_remove_projector_lights_from_sectors` | `DeleteProjectorLightsFromSectors` | 0x002425f0 | CONFIRMED | the sibling above with sector slot `0x50` → `0x54`, plus a slot-`0x2c` reset per light |
| 0x0051a520 | `vehicle_register_projector_light_sector` | `AddSectorForProjectorLights` | 0x002426f0 | CONFIRMED | push-back-if-absent onto the projector-sector list |
| 0x0051a720 | `vehicle_register_sound_sector` | `AddSectorForSounds` | 0x00242790 | CONFIRMED | body-identical to the row above, over the sound-sector list |
| 0x0051a920 | `vehicle_settle` | `CalmVehicle` | 0x00242830 | CONFIRMED | zeroes the angular velocity `+0x49c` on the way out |
| 0x0051aac0 | `vehicle_get_save_state_size` | `GetVehicleStateSize` | 0x00243540 | CONFIRMED | the byte size of the vehicle savegame block, and the first term of `car_save_game_get_size`'s three-term sum. It also absorbs the PS2's separate deform sizer (§9) |
| 0x0051ad90 | `vehicle_save_state` | `SaveVehicleState` | 0x002435d0 | CONFIRMED | serialises the drivetrain, damage and wheel state; pinned by call position inside `car_save_game_save`, immediately after the base-class writer |
| 0x0051ba40 | `vehicle_load_state` | `LoadVehicleState` | 0x00243db0 | CONFIRMED | the inverse; `car_save_game_load` aborts when it returns 0 |
| 0x0051ccf0 | `vehicle_cb_enum_objects` | `CbEnumObj` | 0x00141260 | CONFIRMED | a free function, not a class method |
| 0x0051cdc0 | `vehicle_cb_enum_first_object` | `CbEnumObj1` | 0x001412b0 | CONFIRMED | a free function |
| 0x0051cdd0 | `vehicle_cb_enum_object_list` | `CbEnumObj6` | 0x00246df0 | CONFIRMED | a free function |
| 0x0051cf80 | `vehicle_cb_enum_body_colls` | `CbEnumBodyColl` | 0x00246f10 | CONFIRMED | a free function; builds the `+0x528` tree |
| 0x0051d280 | `vehicle_body_coll_attach_to_parent` | `IsParent` | 0x00246e40 | CONFIRMED | a free function |
| 0x0051d330 | `vehicle_colbody_head_dtor` | - | - | PC-only | - |
| 0x0051d3b0 | `vehicle_collect_coll_spheres` | `EnumColSphere` | 0x002471b0 | CONFIRMED | a free function |
| 0x0051d440 | `vehicle_cb_enum_coll_sphere` | `CbEnumCollsActiveSphere` | 0x00247240 | CONFIRMED | a free function |
| 0x0051d710 | `vehicle_prepare_body_colls` | `PrepareBodyColls` | 0x002473e0 | CONFIRMED | recursive walk of the `S_COLBODY_HEAD` tree baking each sphere into world space — `UpdateWMatrixProc` then two `S_vector::operator*=(S_matrix&)` per sphere on both builds, then the parent-frame arm |
| 0x0051d9a0 | `vehicle_cb_enum_deform_zones` | `CbEnumDeform` | 0x00247670 | CONFIRMED | a free function |
| 0x0051ddf0 | `vehicle_set_callback_collision_info` | `SetCallbackACI` | 0x00247fd0 | CONFIRMED | bank slot `+0x36c` (§8) |
| 0x0051de00 | `vehicle_set_callback_collision` | `SetCallbackCC` | 0x00247fe0 | CONFIRMED | bank slot `+0x364` (§8) |
| 0x0051de10 | `vehicle_set_callback_collision_quiet` | `SetCallbackCC1` | 0x00247ff0 | CONFIRMED | bank slot `+0x368` (§8) |
| 0x0051de20 | `vehicle_set_callback_actor_filter` | `SetCallbackAF` | 0x00248000 | CONFIRMED | bank slot `+0x370` (§8) |
| 0x0051de30 | `vehicle_set_callback_was_collision` | `SetCallbackWC` | 0x00248010 | CONFIRMED | bank slot `+0x374` (§8) |
| 0x0051de40 | `vehicle_set_callback_prepare_dropout` | `SetCallbackPD` | 0x00248020 | CONFIRMED | bank slot `+0x378` (§8) |
| 0x0051de50 | `vehicle_set_callback_prepare_wheel_dropout` | `SetCallbackPDW` | 0x00248030 | CONFIRMED | bank slot `+0x37c` (§8) |
| 0x0051de60 | `vehicle_set_callback_break_glass` | `SetCallbackBG` | 0x00248040 | CONFIRMED | bank slot `+0x380` (§8); fired by `vehicle_deform_glass` |
| 0x0051de70 | `vehicle_set_callback_wake_up` | `SetCallbackWU` | 0x00248050 | CONFIRMED | bank slot `+0x388` (§8, §8a); 3 callsites and no others |
| 0x0051de80 | `vehicle_set_callback_body_collision_filter` | `SetCallbackCFB` | 0x00248060 | CONFIRMED | bank slot `+0x38c` (§8) |
| 0x0051de90 | `vehicle_set_callback_wheel_collision_filter` | `SetCallbackCFW` | 0x00248070 | CONFIRMED | bank slot `+0x390` (§8) |
| 0x0051dea0 | `vehicle_set_callback_deform_part` | `SetCallbackDVP` | 0x00248080 | CONFIRMED | bank slot `+0x394` (§8) |
| 0x0051deb0 | `vehicle_set_callback_repair` | `SetCallbackVR` | 0x00248090 | CONFIRMED | bank slot `+0x398` (§8); fired by `vehicle_init_deform` |
| 0x0051dec0 | `vehicle_colbody_head_vector_destroy` | - | - | PC-only | container helper |
| 0x0051e100 | `vehicle_window_list_insert_at` | - | - | PC-only | container helper |
| 0x0051e310 | `vehicle_window_list_size` | - | - | PC-only | container helper |
| 0x0051e340 | `vehicle_colbody_head_vector_insert_n` | - | - | PC-only | container helper |
| 0x0051ef90 | `vehicle_special_coll_list_insert_n` | - | - | PC-only | container helper |
| 0x0051f330 | `vehicle_special_coll_list_copy_range` | - | - | PC-only | container helper |
| 0x0051f360 | `vehicle_colbody_head_copy` | - | - | PC-only | container helper |
| 0x0051f540 | `vehicle_deform_zone_copy` | - | - | PC-only | container helper |
| 0x0051f560 | `vehicle_colbody_sphere_vector_copy_assign` | - | - | PC-only | container helper |
| 0x0051f7a0 | `vehicle_colbody_head_scalar_deleting_dtor` | - | - | PC-only | container helper |
| 0x0051f7c0 | `vehicle_coll_sphere_list_copy_assign` | - | - | PC-only | container helper |
| 0x0051fa20 | `vehicle_colbody_head_vector_copy_assign` | - | - | PC-only | container helper |
| 0x0051fca0 | `vehicle_coll_sphere_list_copy_range` | - | - | PC-only | container helper |
| 0x0051fd00 | `vehicle_emit_wheel_skid_segment` | `Slaif` | 0x0024a710 | CONFIRMED | opens a skid-mark ring slot for one wheel |
| 0x005214e0 | `vehicle_end_wheel_skid_segment` | `SlaifEnd` | 0x0024c000 | CONFIRMED | clears the active byte of the current ring slot for one wheel and advances the cursor, wrapping. Same guard order and same three statements on both builds; ring stride `0x14`, cursor `+0x16c`, wheel array `+0xcb4` |
| 0x00521580 | `vehicle_prepare_deform_zone` | `PrepareDeformZone` | 0x0024c080 | CONFIRMED | partitions the body-part array `+0x298`, clones each mesh and fills the pristine backup `+0x288` |
| 0x005219a0 | `vehicle_init_deform_glass_texture` | `InitDeformGlassTexture` | 0x0024c3a0 | CONFIRMED | installs the cracked-glass texture pair on every type-3 (glass) part |
| 0x00521a40 | `vehicle_init_deform` | `InitDeform` | 0x0024c4a0 | CONFIRMED | restores the pristine vertices — **the repair path**; fires bank slot `+0x398` |
| 0x00521d30 | `vehicle_init_damage` | `InitDamage` | 0x0024cbc0 | CONFIRMED | resets the damage scalars, lights and projectors |
| 0x00521e60 | `vehicle_deform` | `Deform` | 0x0024d960 | CONFIRMED | the impact entry point; dispatches to the three below |
| 0x00522dd0 | `vehicle_switch_special_projector_texture` | `SwitchSpecialProjectorTexture` | 0x0024f830 | CONFIRMED | swaps a damaged projector's texture |
| 0x00522ec0 | `vehicle_deform_glass` | `DeformGlass` | 0x0024f970 | CONFIRMED | breaks one part's glass; fires bank slot `+0x380` |
| 0x00522f70 | `vehicle_apply_damage` | `DamageVehicle` | 0x0024fb50 | CONFIRMED | applies the scalar damage |
| 0x00523090 | `vehicle_update_axis` | `UpdateAxis` | 0x0024fcb0 | CONFIRMED | called ×2 per frame on both builds |
| 0x00523fa0 | `vehicle_animate_doors` | `DoDoors` | 0x00250680 | CONFIRMED | reads the door angle band of the `+0xd20` records |
| 0x005242d0 | `vehicle_open_window` | `OpenWindow` | 0x00250a50 | CONFIRMED | - |
| 0x00524320 | `vehicle_animate_windows` | `DoWindows` | 0x00250ab0 | CONFIRMED | - |
| 0x005244d0 | `vehicle_is_window_open` | `IsWindowOpen` | 0x00250df0 | CONFIRMED | - |
| 0x00524520 | `vehicle_apply_light_state` | `DoLights` | 0x00250e90 | CONFIRMED | the per-frame light consumer; reads the `+0x19c` word (§1b) and the light vector `+0x17c` |
| 0x00528b80 | `vehicle_update_strips` | `UpdateStrips` | 0x00256270 | CONFIRMED | - |
| 0x0052a1b0 | `vehicle_update_motor_particles` | `UpdateMotorParticles` | 0x00256c10 | CONFIRMED | - |
| 0x0052a670 | `vehicle_update_steering_wheels` | `UpdateSteeringWheels` | 0x002570a0 | CONFIRMED | called ×1 from BOTH `Tick` and `Update` on both builds |
| 0x0052aa30 | `vehicle_update_rotation_wheels` | `UpdateRotationWheels` | 0x00257770 | CONFIRMED | - |
| 0x0052af10 | `vehicle_update_wheel_particles` | `UpdateWheelParticles` | 0x00258120 | CONFIRMED | one of only two functions per build that touch the material-id lookup table |
| 0x0052b8a0 | `vehicle_update_exhaust_particles` | `UpdateExhaustParticles` | 0x00258960 | CONFIRMED | - |
| 0x0052bf20 | `vehicle_emit_body_collision_particles` | `UpdateBodyParticles` | 0x00258fa0 | CONFIRMED | emits one material-keyed impact particle at a contact point with the body's point velocity `v + ω × (p − com)`. Five shared constants settle it: the gate bit `0x200` of `+0xce4`; the byte-identical material lookup `abs(materials[conv_tbl[id] * 0x30 + 0x28])`; the six `SetParam` ids `0xd 0xe 0xf 0x10 0x11 0x12` in the same order; the `0x1d`/`0x1e` velocity vectors scaled by `+0x130`; and the frame-flag rewrite `& 0xfffffedf \| 0x40000000`. The PC restructured it into an immediate, rate-limited per-contact emitter called from `vehicle_body_coll_resolve` (§9) |
| 0x0052c350 | `vehicle_light_at` | - | - | PC-only | indexes the light vector `+0x180`; 301 callsites |
| 0x0052c360 | `vehicle_special_projector_at` | - | - | PC-only | indexes the special-projector vector `+0x190`; 51 callsites |
| 0x0052c3b0 | `vehicle_tick_drivetrain` | `Tick` | 0x002594c0 | CONFIRMED | the physics half of the frame |
| 0x0052d480 | `vehicle_update_visuals` | `Update` | 0x0025b4c0 | CONFIRMED | the visual and audio half; calls every `Do*`/`Update*` member of the effects unit |
| 0x0052dfa0 | `vehicle_compute_motor_force` | `MotorForce` | 0x0025d530 | CONFIRMED | drive force; calls `ComputeMoment` ×1 |
| 0x0052e220 | `vehicle_compute_moment` | `ComputeMoment` | 0x0025d8c0 | CONFIRMED | the mid-gap anchor of the kernel unit, size ratio exactly 1.00 |
| 0x0052e3c0 | `vehicle_update_motor_revs` | `MotorRot` | 0x0025da60 | CONFIRMED | revs and gearbox — **the engine model**; fires `GearSnd` ×1 |
| 0x0052e6d0 | `vehicle_substep_move` | `Engine` | 0x0025dff0 | CONFIRMED | despite the PS2 name this is **not** the engine model: it latches two counters, calls `vehicle_move` once at the full delta and then in a loop at `delta / n` |
| 0x0052ea80 | `vehicle_move` | `Move` | 0x0025e390 | CONFIRMED | the integrator; six float parameters |
| 0x005334e0 | `vehicle_wheel_contact_solve` | `DoWheelsCollision` | 0x00262850 | CONFIRMED | the wheel solver; reads `+0xcd0` 15× |
| 0x00535610 | `vehicle_body_coll_resolve` | `BodyCollision` | 0x00264de0 | CONFIRMED | the body-collision tree solver (§6d, §9) |
| 0x00537300 | `vehicle_body_coll_sphere_sort_list_static_dtor` | - | - | PC-only | §6d |
| 0x00537320 | `vehicle_body_coll_head_sort_list_static_dtor` | - | - | PC-only | §6d |
| 0x005373e0 | `vehicle_init_sounds` | `InitSounds` | 0x00267870 | CONFIRMED | opens the whole sound bank: about 21 calls to `vehicle_sound_slot_init`, each name taken from the `S_CARINIT` sound table at `+0xc84`..`+0xee4`. Called as `(this + 0xde8)`, which is where `S_CARINIT` is byte-proven, so the parameter type matches the PS2 signature exactly |
| 0x00537880 | `vehicle_add_sounds_to_sectors` | `AddSoundsToSectors` | 0x00267dc0 | CONFIRMED | iterates the sound-sector list calling `I3D_sector` slot `0x58` for each live handle — independent confirmation that the list is the SOUND one |
| 0x00537d50 | `vehicle_remove_sounds_from_sectors` | `DeleteSoundsFromSectors` | 0x00268720 | CONFIRMED | byte-identical to the row above with sector slot `0x58` → `0x5c` |
| 0x00538220 | `vehicle_update_engine_sound` | `EngineSnd` | 0x00269080 | CONFIRMED | gated on `+0x4ae`; checks the four engine handles `+0x6c8`/`+0x6cc`/`+0x9a8`/`+0x9ac` and either tears the bank down through `SoundOff` or starts the handle `+0x50d` selects |
| 0x005385b0 | `vehicle_update_engine_pitch` | `EngineFreq` | 0x00269570 | CONFIRMED | maps engine state onto playback frequency across the interior/exterior split; the largest body in the unit on both builds |
| 0x00539550 | `vehicle_update_environment_sound` | `EnvironmentalSnd` | 0x0026ac20 | CONFIRMED | level driven by the linear-velocity magnitude from `+0x1f0c`, clamped into the `[+0xc60, +0xc64]` band, driving handle `+0x9f8` |
| 0x005398d0 | `vehicle_update_flat_tyre_sound` | `CrashTyreSnd` | 0x0026b080 | CONFIRMED | level tracks \|dot(world forward `+0xcb8`, linear velocity)\| banded by `[+0xc78, +0xc80]` on handle `+0xa48` — the one slot `InitSounds` fills from a hard-coded literal, `PNEU_PUNC.WAV`, min 6.0 max 30.0. Called ×2 per frame on both builds |
| 0x00539c60 | `vehicle_update_skid_sound` | `SkidSnd` | 0x0026b4e0 | CONFIRMED | per-frame skid level from the slip magnitude |
| 0x00539f80 | `vehicle_play_gear_change_sound` | `GearSnd` | 0x0026b900 | CONFIRMED | picks one of two variants with a `rand()`-based coin flip per shift, then starts the interior or exterior handle |
| 0x0053a080 | `vehicle_play_handbrake_sound` | `HandBrake` | 0x0026baa0 | CONFIRMED | one-shot latched by `+0x1d4`, gated on bit `0x20` of `+0xce4` |
| 0x0053a120 | `vehicle_play_engine_start_fail_sound` | `EngineSndNFS` | 0x0026bba0 | CONFIRMED | plays the non-looping `S_CARINIT+0xc84` entry once |
| 0x0053a1b0 | `vehicle_play_horn_sound` | `HornSnd` | 0x0026bca0 | CONFIRMED | stops handles `+0x718`/`+0x9fc` on false, otherwise starts whichever `+0x50d` selects; the handle pair sits immediately below the siren's, matching `+0x4ac` horn / `+0x4ad` siren |
| 0x0053a280 | `vehicle_play_siren_sound` | `SirenSnd` | 0x0026bdf0 | CONFIRMED | the exact mirror of the horn at the next handle pair |
| 0x0053a350 | `vehicle_play_tyre_burst_sound` | `TyreBreakSnd` | 0x0026bf40 | CONFIRMED | gated on `+0x4ae`; starts handle `+0x738`/`+0xa1c` and POSITIONS the sound at the wheel taken from `+0xcb4`, with the frame-flag rewrite |
| 0x0053a410 | `vehicle_play_crash_sound` | `CrashSnd` | 0x0026c040 | CONFIRMED | picks a crash sound and level from the impact magnitude |
| 0x0053b0c0 | `vehicle_play_wheel_sound` | `WheelSnd` | 0x0026d0e0 | CONFIRMED | starts handle `+0x734`/`+0xa18` at the global volume scale |
| 0x0053b120 | `vehicle_play_door_sound` | `DoorSnd` | 0x0026d260 | CONFIRMED | gated on `+0x4ae`; selects interior or exterior by `+0x50d` |
| 0x0053b2c0 | `vehicle_stop_motor_sounds` | `MotorSoundOff` | 0x0026d4f0 | CONFIRMED | walks the handle bands `+0x6d4` (5), `+0x6e8` (5) and `+0x6fc`, calling sound slot `0x88` with `(0, 1)` |
| 0x0053b3f0 | `vehicle_stop_all_sounds` | `SoundOff` | 0x0026d720 | CONFIRMED | the same slot-`0x88` shape over the full band set |
| 0x0053b840 | `vehicle_set_global_sound_volume` | - | - | PC-only | one statement: writes `g_vehicle_sound_volume_scale` (`0x0064c7c0`), the master scale every sound call in the unit passes to volume slot `0x68`. Its only writer |
| 0x0053b850 | `vehicle_open_sound` | `OpenSnd` | 0x0026de10 | CONFIRMED | a three-argument forwarder to the shared `i3d_cache_sound_open`, matching the PS2 signature at the exact rank |
| 0x0053b880 | `vehicle_sound_slot_init` | `SoundInit` | 0x0026e2a0 | CONFIRMED | nine arguments at every callsite, matching the nine-parameter PS2 signature. Defaults: min distance 4.0, max 50.0, volume 0.3 |
| 0x0053bbf0 | `vehicle_release_sounds` | `ReleaseSounds` | 0x0026e800 | CONFIRMED | releases every handle band |
| 0x005c8740 | `vehicle_controller_set_field_0x65` | - | - | PC-only | - |
| 0x005c8780 | `vehicle_get_special_dynamic_colls` | - | - | PC-only | reads the mode byte `+0x1f26` |
| 0x005c8790 | `vehicle_set_velocity` | - | - | PC-only | writes `+0x1f0c` |
| 0x005c8840 | `vehicle_set_angular_velocity` | - | - | PC-only | writes `+0x49c` |
| 0x005c8880 | `vehicle_set_auto_indicators_from_steering` | - | - | PC-only | bit `0x8`; the only light setter that does NOT notify (§1b) |
| 0x005c88b0 | `vehicle_set_light_bit_0x80` | - | - | PC-only | bit `0x80`, script `carlight_light` / case `0xc1`; the lamp is unidentified, so the name records the bit (§1b) |
| 0x005c88e0 | `vehicle_set_left_indicator` | - | - | PC-only | bit `0x1`, gated on bit `0x8` clear, plus notify. Script `carlight_indic_l` / case `0xc2` |
| 0x005c8920 | `vehicle_set_right_indicator` | - | - | PC-only | bit `0x2`, gated on bit `0x8` clear, plus notify. Script `carlight_indic_r` / case `0xc3` |
| 0x005c8960 | `vehicle_set_headlights` | - | - | PC-only | bit `0x10000`, plus notify. Script `carlight_main` / case `0xc0` |
| 0x005c89a0 | `vehicle_set_light_script_override` | - | - | PC-only | writes the `0x0ff00000` sub-field (§1b) |
| 0x005c89e0 | `vehicle_get_light_script_override` | - | - | PC-only | reads it |
| 0x005c8a90 | `vehicle_controller_set_field_0x214c` | - | - | PC-only | - |

### 3a. PS2 methods with no PC twin

Twenty of the 153. Seven are accounted for by a fold or an inline on the PC side; thirteen are not
located (§11).

| PS2 VA | PS2 method | State |
|---|---|---|
| 0x002331c0 | `InitSpecialLightProjectors` | not located |
| 0x002335a0 | `CloseSpecialLightProjectors` | not located |
| 0x00233a80 | `CloseVehicle` | not located |
| 0x00234110 | `ReleaseBodyColls` | not located |
| 0x00238bb0 | `ClearParticleRef` | not located |
| 0x00238d60 | `LockParticle` | not located |
| 0x00239270 | `UnlockParticle` | not located |
| 0x002397f0 | `TickParticles` | not located |
| 0x00240370 | `DoCollisionStatic` | inlined into both PC callers (§9) |
| 0x00240ad0 | `ComputeMass` | inlined into `vehicle_set_seat_mass` (§7) |
| 0x00241a50 | `IsDoor` | not located (§11) |
| 0x00242c30 | `GetForwardForceMax` | not located |
| 0x00242c80 | `GetSideForceMax` | not located |
| 0x00242cd0 | `DuplicateDamageAndDeform` | not located (§11) |
| 0x00245290 | `GetVehicleDamageDeformSize` | folded into `vehicle_get_save_state_size` (§9) |
| 0x00245650 | `SaveVehicleDamageDeform` | folded into `vehicle_save_state` (§9) |
| 0x00245f70 | `LoadVehicleDamageDeform` | folded into `vehicle_load_state` (§9) |
| 0x002480a0 | `SetCallbackLP` | not located (§11) |
| 0x002480b0 | `SetCallbackCP` | not located |
| 0x0024cd30 | `UpdateDeformPart` | not located (§11) |

## 4. Power / brake delivery notes

* **`+0x634` is the net wheel longitudinal force**, written by BOTH `SetPower` (the neutral,
  `gear_effective < 0` path) and `SetBrake` (the in-gear path) through the same
  `(ramp_accum / ramp_max) · pedal` ramp. `driver_tick`'s tail calls
  `vehicle_set_power(speed_factor · scale)` and then `vehicle_set_brake(brake_out)`; the two cover
  mutually exclusive gear states, so they cooperate rather than clobber. `pedal == 0` means "cut and
  coast". There is no separate "SetGas".
* **The dedicated brake is `vehicle_set_brake`; the parking brake is `vehicle_set_handbrake`**
  (`+0x4b0`). The chase AI's `driver+0xdc` is the brake pedal and feeds `vehicle_set_brake` and the
  brake light (`driver_do_lights`).
* **Boosting is a `C_car` model-level swap, not a `C_Vehicle` field** —
  `police-system/artifacts/boosting.md`.

## 5. `C_car` is a different class

Everything above describes **`C_Vehicle`** (mangled `9C_Vehicle`), the physics and drivetrain
sub-object at `car+0x70`. The **`C_car`** entity that owns it (mangled `5C_car`, entity kind **4**)
is a separate class. Convention: `C_car` → `car_*`, `C_Vehicle` → `vehicle_*`, and a prefix is
assigned only after the receiver has been measured at the callsites.

All `C_car` field layout, the seat and owner API, model-level (boost) switching and the `C_car`
method coverage table are in `structs/car.md`. This file is `C_Vehicle`-only.

## 6. The four translation units

The class occupies four contiguous PC units mapping 1:1 onto the four PS2 `C_Vehicle*.cpp` source
files, bounded by the CRT static-init/atexit boundary pairs. Any "no PC twin" claim for a
`C_Vehicle` method has to say which of the four it searched.

| PC range | PS2 source file | PC starts | PC bytes | PS2 bytes | ratio |
|---|---|---|---|---|---|
| `[0x00509430, 0x0051fd00)` | `C_Vehicle.cpp` | 137 | 92 320 | 118 960 | 0.78 |
| `[0x0051fd00, 0x0052c3b0)` | `C_Vehicle_effects.cpp` | 29 | 50 864 | 60 848 | 0.84 |
| `[0x0052c3b0, 0x005373e0)` | `C_Vehicle_kernel.cpp` | 15 | 45 104 | 58 288 | 0.77 |
| `[0x005373e0, 0x0053c2b0)` | `C_Vehicle_sounds.cpp` | 26 | 20 176 | 31 320 | 0.64 |

The cluster is closed: 206 starts, all named. Not every `vehicle_` function lives inside it — the
one-line accessors at `0x00425440`, `0x004f5a30`..`0x004f5ac0` and `0x005c8740`..`0x005c8a90` are
out-of-line inline members MSVC emitted into whichever object first used them, so they land in the
`entity_action_tick`, `C_sledovacka` and script-VM units instead.

**The next class is `C_wagon`** (entity kind `0x1e`, `[0x0053c2b0, 0x005407f0)`, 19 starts), then
`C_recorder` at `0x00540820`. PC and PS2 link order therefore agree across five consecutive source
files — `C_turnout.cpp`, the four `C_Vehicle*.cpp`, `C_wagon.cpp`.

### 6a. The five file-scope vectors

The first unit opens with five identical MSVC static quartets: an init entry, a constructor plus an
atexit registrar, and a destructor guarded by one bit of `g_vehicle_static_dtor_guard`
(`0x00656318`).

| Guard bit | Begin pointer | What | Element |
|---|---|---|---|
| `0x01` | `g_vehicle_tyre_list_begin` | the tyre list | `S_TYRE`, stride `0x28` |
| `0x02` | `g_vehicle_materials_begin` | the material list | `S_VEH_MATERIAL`, stride `0x30` |
| `0x04` | `g_vehicle_particles_begin` | the particle list | `S_VEH_PARTICLE`, stride `0x1c` |
| `0x08` | `g_vehicle_projector_light_sectors_begin` | the sectors the vehicle's projector lights are registered with | `I3D_sector*` |
| `0x10` | `g_vehicle_sound_sectors_begin` | the sectors the vehicle's sounds are registered with | `I3D_sector*` |

The last two are each pinned by three functions: a registrar that push-backs uniquely
(`vehicle_register_projector_light_sector` / `vehicle_register_sound_sector`) and an add/remove pair
that walks it. The two pairs use adjacent `I3D_sector` vtable slots — `0x50`/`0x54` for lights,
`0x58`/`0x5c` for sounds — which corroborates that they are the same mechanism twice. All five
element types come from the PS2 parameter types of `SetTyres`, `SetVehicleMaterials` and
`AddParticle`.

Beside them sits `g_vehicle_material_index_by_id`, a 256-byte material-id → index table at
`0x00656208`. It is **not** `DAT_00656204 + 4`; that address is the particle vector's capacity
pointer, and anchoring on where the decompiler renders the memset base would place the table four
bytes early.

### 6b. `C_Vehicle.cpp` — the first unit

`[0x00509430, 0x0051fd00)`, 137 starts, anchored on a 12-for-12 in-order block: PC
`0x005173a0`…`0x00517ae0` against `SetGear`…`SetCutTime`. It maps against PS2 methods 0–98 only,
`{ctor}` … `SetCallbackCP`, so it is roughly one quarter of the class. It holds construction, the
geometry and CARINIT import, the whole setter family, the save-state triple, the body-collision tree
builders, the callback bank (§8) and the container helper tail.

### 6c. `C_Vehicle_effects.cpp` — deform, damage, doors, windows, lights

`[0x0051fd00, 0x0052c3b0)`, 29 starts. It holds body deformation and damage, the door and window
animation pair, the light-state consumer and the seven per-frame emitters.

**One caller in the kernel unit validates the whole unit.** `vehicle_update_visuals` calls every
`Do*`/`Update*` function here, and its PS2 twin calls the same set with the same multiplicities. Two
of those are falsifiable and both match: `UpdateAxis` ×2 and `CrashTyreSnd` ×2 on both builds.
`UpdateSteeringWheels` is additionally called ×1 from BOTH `Tick` and `Update` on BOTH builds — a
four-way cross-check pinning three functions at once.

**The deformation chain, end to end:**

| Stage | Function | What it does |
|---|---|---|
| build | `vehicle_prepare_deform_zone` | partitions the body-part array `+0x298`, clones each mesh and fills the pristine backup `+0x288` |
| texture | `vehicle_init_deform_glass_texture` | installs the cracked-glass texture pair on every type-3 (glass) part |
| reset | `vehicle_init_deform` | restores the pristine vertices — the repair path; fires bank slot `+0x398` |
| reset | `vehicle_init_damage` | resets the damage scalars, lights and projectors |
| impact | `vehicle_deform` | the entry point; dispatches to the three below |
| impact | `vehicle_deform_glass` | breaks one part's glass; fires bank slot `+0x380` |
| impact | `vehicle_apply_damage` | applies the scalar damage |
| impact | `vehicle_switch_special_projector_texture` | swaps a damaged projector's texture |

Two callback slots confirm two names without being asked to: the predictions come from the PS2 side
and the slots are looked up afterwards. `vehicle_set_callback_break_glass` writes `+0x380`, which
`vehicle_deform_glass` fires; `vehicle_set_callback_repair` writes `+0x398`, which
`vehicle_init_deform` fires — and that is also what settles `vehicle_init_deform` as the repair path.

**The seven per-frame emitters** (`vehicle_update_strips`, `_motor_particles`, `_steering_wheels`,
`_rotation_wheels`, `_wheel_particles`, `_exhaust_particles` and
`vehicle_emit_body_collision_particles`) are a forced 7-for-7 block. Only two functions on each build
touch the material-id lookup table, which is what separates the wheel emitter from its neighbours.

### 6d. `C_Vehicle_kernel.cpp` — the motor model and the integrator

`[0x0052c3b0, 0x005373e0)`, 15 starts. The unit aligns 9-for-9 against the PS2 with
`vehicle_compute_moment` sitting mid-gap at a size ratio of exactly **1.00**, the cleanest alignment
anywhere in the cluster.

| PC | PS2 | ratio | What it is |
|---|---|---|---|
| `vehicle_tick_drivetrain` | `Tick` | 0.53 | the physics half of the frame |
| `vehicle_update_visuals` | `Update` | 0.35 | the visual and audio half — the lever for §6c |
| `vehicle_compute_motor_force` | `MotorForce` | 0.71 | drive force; calls `ComputeMoment` ×1 |
| `vehicle_compute_moment` | `ComputeMoment` | 1.00 | the mid-gap anchor |
| `vehicle_update_motor_revs` | `MotorRot` | 0.55 | revs and gearbox; fires `GearSnd` ×1 |
| `vehicle_substep_move` | `Engine` | 1.03 | subdivides the frame and calls `Move` per sub-step |
| `vehicle_move` | `Move` | 1.08 | the integrator; six float parameters |
| `vehicle_wheel_contact_solve` | `DoWheelsCollision` | 0.88 | the wheel solver |
| `vehicle_body_coll_resolve` | `BodyCollision` | 0.89 | the body-collision tree solver |

The two per-frame drivers are named for their callee sets rather than for the PS2 labels: `Tick` and
`Update` read as synonyms while their callee sets are disjoint, so the PC names carry the split —
drivetrain against visuals.

**The two function-local static sort vectors.** `vehicle_body_coll_resolve` builds two MSVC
magic-statics on first call, both `vector<T8>`:

| Object | Begin | Entries index | atexit destructor |
|---|---|---|---|
| `g_vehicle_body_coll_sphere_sort_list` `0x00656344` | `0x00656348` | the `0x3c`-stride sphere array at `S_COLBODY_HEAD+0x00` | `vehicle_body_coll_sphere_sort_list_static_dtor` |
| `g_vehicle_body_coll_head_sort_list` `0x00656354` | `0x00656358` | the `0x74`-stride `S_COLBODY_HEAD` vector at `+0x528` | `vehicle_body_coll_head_sort_list_static_dtor` |

The PS2 instantiates `vector<S_SORT>` **and** `vector<S_COLBODY_EMITOR>` in this unit and nowhere
else, so those are the two element types — but which is which is not resolved (§11), and the PC
names deliberately describe the measured role instead of asserting a PS2 type. Both destructors are
taken by address only (`atexit(&…)`), so a census that counts `E8` call instructions reports zero
callsites and zero datarefs for them; that is the signature of an address-referenced function, not
evidence of dead code.

**Two more function-local statics.** `g_vehicle_body_coll_contact_normal` (`0x00656330`, one
`S_vector`) receives the contact normal from the body-contact surface filter (`+0x38c`, §8); the
response then uses it in dot products, cross products and projections against the body
velocity. `g_vehicle_body_coll_struck_actor` (`0x0065633c`) holds the **struck actor**: the contact record's `+0x44` when the record's
flag `0x40` is set, else 0 — `car_collision_filter_actor` reads the same field and switches on its
`+0x10` class kind. A zero skips the actor filter (`+0x370`) and the collision-info query
(`+0x36c`), and the value is passed on to the post-collision notification (`+0x374`).

### 6e. `C_Vehicle_sounds.cpp` — the sound bank

`[0x005373e0, 0x0053c2b0)`, 26 starts; 21 of the 22 PS2 methods are named, plus two PC-only
functions. Three structural facts:

* **The sound FILENAMES are data, not literals.** `vehicle_init_sounds` calls
  `vehicle_sound_slot_init` about 21 times, taking each name from the `S_CARINIT` table at
  `+0xc84`..`+0xee4` (stride `0x10`). Exactly ONE name is hard-coded in the binary —
  `PNEU_PUNC.WAV` (Czech *pneu punc* = tyre puncture) — which is what decoded
  `vehicle_update_flat_tyre_sound`.
* **Every sound has an interior/exterior handle pair**, selected by `+0x50d`. The interior handles
  are the `+0x6xx` band, the exterior ones the `+0x9xx`/`+0xaxx` band.
* **A master volume scale.** `g_vehicle_sound_volume_scale` (`0x0064c7c0`) is passed to the sound
  object's volume slot `0x68` by every call in the unit, and
  `vehicle_set_global_sound_volume` is its only writer.

Default 3D parameters are min distance **4.0**, max **50.0**, volume **0.3**; the puncture sound uses
6.0 / 30.0 and one slot uses 110.0. A refcounted-string constructor, `string_ensure_buffer`, is
parked in the unit's tail and is not a vehicle function.

## 7. The seat and wheel vectors, and the mass distribution

`vehicle_set_seat_mass` sets one seat's occupant mass and then recomputes the vehicle's centre of
mass. It is reached from `opponent_game_init` when a race entrant is seated, and from
`car_set_owner` and the pursuit code when anyone gets in or out.

| PC offset | What |
|---|---|
| `+0xd20` / `+0xd24` | the **seat vector** — `0x24`-byte records, occupant mass at record `+0x18` |
| `+0xd40` / `+0xd44` | the **wheel vector** — `0x1c`-byte records, the second input to the mass computation |

**The prefix was measured, not assumed.** All 10 callsites of `vehicle_set_seat_mass` build the
receiver as `<C_car> + 0x70` — `lea ecx,[esi+0x70]`, `mov ecx,[edx+0xc]; add ecx,0x70`, and from
`opponent_game_init` `mov ecx,[esi+0x28]; add ecx,0x70` — with 0 datarefs, so it is the embedded
`C_Vehicle` sub-object rather than a `C_car` method, and it is not a vtable slot.

## 8. The `C_Vehicle` callback bank, `+0x364`..`+0x398`

`C_Vehicle` does not implement collision policy; it calls back into its owner. The owner installs
thirteen function pointers into a contiguous bank during construction, through thirteen one-line
setters. For a `C_car` the installer is `car_init`.

| Offset | Setter | Role | What `car_init` installs |
|---|---|---|---|
| `+0x364` | `vehicle_set_callback_collision` | full collision reaction | `car_react_to_and_report_collision` |
| `+0x368` | `vehicle_set_callback_collision_quiet` | consequence-free reaction | `car_react_to_collision` |
| `+0x36c` | `vehicle_set_callback_collision_info` | response-parameter query | `car_get_collision_info` |
| `+0x370` | `vehicle_set_callback_actor_filter` | struck-object relevance filter | `car_collision_filter_actor` |
| `+0x374` | `vehicle_set_callback_was_collision` | post-collision notification | `car_on_collision_apply_damage` |
| `+0x378` | `vehicle_set_callback_prepare_dropout` | body-panel drop-out | `car_prepare_body_part_dropout` |
| `+0x37c` | `vehicle_set_callback_prepare_wheel_dropout` | wheel drop-out | `car_prepare_wheel_dropout` |
| `+0x380` | `vehicle_set_callback_break_glass` | glass shatter | `car_break_glass` |
| `+0x384` | - | never written by `car_init` — [U] | - |
| `+0x388` | `vehicle_set_callback_wake_up` | wake a sleeping car | NULL at init; `car_on_act_state_changed` later installs `car_wake_up` |
| `+0x38c` | `vehicle_set_callback_body_collision_filter` | body-contact surface filter | `car_on_body_surface_contact` |
| `+0x390` | `vehicle_set_callback_wheel_collision_filter` | wheel-contact surface filter | `car_collision_filter_wheel` |
| `+0x394` | `vehicle_set_callback_deform_part` | panel deformation | `car_add_deform_scratch` |
| `+0x398` | `vehicle_set_callback_repair` | repair / decal wipe | `car_clear_damage_decals` |

**Why the offsets are trustworthy:** each setter's body is a single `mov [this+N], arg`, and the
thirteen N values are consecutive. That proves the install order carries no skew, which is what makes
the bank usable as an identification lever for the callbacks themselves (§9).

### 8a. `C_Vehicle+0x388` — the wake-up slot, and why the light setters call it

Two things are true of this slot at once, and together they describe one complete mechanism.

| Claim | Status | Evidence |
|---|---|---|
| `+0x388` is the wake-up slot of the bank (`vehicle_set_callback_wake_up`) | CONFIRMED | its setter is the 9th of the 13 `car_init` installs; the 13 setters write consecutive offsets `+0x364`..`+0x398` (the other build agrees, §9) |
| the only handler ever registered there is the wake-up handler | CONFIRMED | the setter has exactly **3 callsites and no others**: one in `car_init` (NULL) and two in `car_on_act_state_changed`, which installs `car_wake_up` on entering act state 1 and clears it on state 0 (§9) |
| every light setter invokes it when a lamp actually changes | CONFIRMED | `vehicle_set_headlights` is literally `*(this+0x19c) = …; if (*(this+0x388)) (*(this+0x388))();`, and `vehicle_set_left_indicator`, `vehicle_set_right_indicator` and `car_unpack_light_state` do the same |

The light setters are therefore **consumers** of the slot, not its owner. Changing a lamp on a car
that is asleep — act state 1, so a handler is installed — wakes it; on a car that is awake the slot
is NULL and the call is skipped. `car_wake_up`'s whole body is `actor_set_act_state(0)`, which is
exactly what "a light changed, so this car is not dormant any more" should do. There is no second
notify mechanism behind the light bits.

## 9. PC vs PS2

All cross-build material for this class lives here; the twin tables of §3, §3a, §6 and §6d are the
mapping itself.

**Field pairs.** The two builds do not lay this class out as a uniform shift, so these are pairs, not
a rebase.

| PC | PS2 | What | Proof |
|---|---|---|---|
| `+0x140` | `+0x51c` | the lock reference count | `vehicle_lock` / `LockVehicle`, same bit `0x10000000` |
| `+0x14c` | `+0x528` | damage state | the `InitDamage` diff |
| `+0x180` | `+0x560` | the light vector begin pointer | the same |
| `+0x190` | `+0x574` | the special-projector vector begin pointer | the same |
| `+0x19c` | `+0x578` | the light-state flag word | `DoLights` reads it with the same bit numbers |
| `+0x224` | `+0x5c8` | the master flag word | `vehicle_is_unstable` reads bit 3 on both |
| `+0x230` | `+0x5d4` | damage scalar | the `InitDamage` diff |
| `+0x234` / `+0x238` | `+0x604` / `+0x608` | damage scalars | the same |
| `+0x26c` / `+0x270` | `+0x5d8` / `+0x5dc` | damage scalars | the same |
| `+0x298` stride `0x34`, flags `+0x0e` | `+0x624` stride `0x3c`, flags `+0x38` | the body-part array | the window and deform diffs |
| `+0x540` | `+0x768` | the wheel count | the `InitDamage` diff |
| `+0xcb4` | `+0xf0c` | the wheel array | the same, and the skid-ring diff |
| `+0xd20` stride `0x24` | `+0xf50` stride `0x30` | the door / seat record array, mass at record `+0x18` ↔ `+0x20` | both door readers index it; `SetSeatMass` on both |
| `+0xd40` / `+0xd44` | `+0xf68` / `+0xf6c` | the wheel vector, records `0x1c` ↔ `0x30` bytes | `SetSeatMass` on both |
| `+0xde8` (4388 B) | `+0xf78` (4672 B) | the `S_CARINIT` block | `GetInit` copies it out on both |
| `+0x16c` | `+0x214` | the skid-mark ring cursor, stride `0x14` ↔ `0x0c` | the `SlaifEnd` diff |
| `+0x130` | `+0x510` | the particle velocity scale | the `UpdateBodyParticles` diff |
| `+0xce4` | `+0xf14` | the flag word carrying the particle gate bit `0x200` | the same |
| `+0x618` | `+0x8f0` | the scalar speed | read inline on PS2 |
| `+0x5bc` / `+0x5c0` | `+0x804` / `+0x808` | the throttle outputs | the gate `+0x634 > 0.9 && +0x5bc > 0.0` is `+0x88c > 0.9 && +0x804 > 0.0` there |
| `+0x634` | `+0x88c` | the net wheel longitudinal force | `SetPower` and `SetBrake` on both |

**`DoLights` is the key cross-build lever for the light bits.** The PS2 method at `0x00250e90`
(21 472 B) reads the flag word at `+0x578` using the same bit numbers as the PC's `+0x19c`, which is
what fixes the PC bit semantics:

| PS2 `DoLights` behaviour | What it fixes on the PC side |
|---|---|
| the bit `0x8` block: `fabs(steer) > threshold(+0x58c)` lights the side matching the steering SIGN, else clears both | bit `0x8` is **automatic indicators**, not "indicators disabled" |
| `steer < 0` sets bit `0x1`; `steer > 0` sets bit `0x2` | corroborates LEFT = `0x1`, RIGHT = `0x2` |
| `0.0 < *(float *)(this+0x88c)` sets bit `0x100` | brake lights |
| `*(int *)(this+0x81c) < 0` sets bit `0x1000` | reverse lights |

**No PS2 twin exists for any of the one-line light setters of §1b, and that is a measurement.** All
153 PS2 `C_Vehicle` methods were enumerated; the entire sub-100-byte tail is 35 methods — the 15
`SetCallbackXX` registrars plus `CreateLightProjector`, `CloseVehicleMaterials`, `GetInit`,
`SetSteeringLinearity`, `SetSpeedLimit`, `SetFuel`, `SetHandbrake`, `EnableSounds`, `SetCutTime`,
`GetHeadPos`, `GetRotations`, `IsVehicleUnstable`, `GetVehicleSize`, `GetVehiclePivotPos`,
`LockVehicle`, `SetMaxDoorAngle`, `SetSpeedDoorOpen`, `GetForwardForceMax`, `GetSideForceMax` and
`OpenWindow` — and none of them touches a light bit. GCC inlined that work into `DoLights` and
`Process__9C_program`; MSVC out-lined it into the one-line members listed in §3.

**The callback bank as an identification lever.** PS2 `Init__5C_carFP9I3D_frame` calls
`SetCallbackXX__9C_Vehicle(this + 0x80, <literal method address>)` thirteen times, naming each
callback method outright. The PC `car_init` calls the thirteen setters of §8 with a literal function
pointer each time. Because the PC setters write consecutive offsets, the two sequences align without
any body diff — and slot 9 being NULL on **both** builds is a free alignment check. That single diff
named thirteen `C_car` callbacks and thirteen `C_Vehicle` setters at once.

**The wake-up slot on the PS2 (§8a).** The PS2's 9th bank install is
`SetCallbackWU__9C_VehicleFPFPCv_v(this + 0x80, 0)`, and PS2 `ChangeState__5C_carFv` installs
`Wake_Up__5C_carFPCv` at exactly the two points where `car_on_act_state_changed` installs and clears
`car_wake_up`.

**Factoring differences, all marked in the decomp at their sites.**

* `ComputeMass` is **inlined into `vehicle_set_seat_mass`**: the PC splices the whole weighted running
  mean over both vectors into that body, `156 + 1016 = 1172` against 864 = 0.74. It has no separate
  PC start and must not be hunted as a missing twin.
* `SetDynamicCollsOn` is **inlined into `vehicle_deactivate`**, which is why the two builds' callee
  multisets there look unlike each other.
* `CloseParticles` + N × `AddParticle` are **fused into `vehicle_set_particles`** — two PS2 methods
  onto one PC function, which is why that twin is STRUCTURAL.
* The reverse also happens: the PS2 **inlined `vehicle_init_strip_and_smoke_emitters` into `Init`**,
  so that PC function has no PS2 symbol at all. `6976 + 5408 = 12 384` against `Init`'s 13 560 = 0.91.
* `CreateLightProjector`'s PS2 symbol is an **8-byte stub** while the PC ships 1456 bytes of real
  code, so its twin is STRUCTURAL.
* `UpdateStrips` is 2.30× on the PC, because MSVC inlined the
  `LockVertices`/`UnlockVertices`/`UpdateBBox`/`UpdateBoundVolume` quartet the PS2 calls out of line
  four times each.

**The damage/deform save triple — three N→1 folds.** The PS2 has six save-state methods where the PC
has three:

| PC function | PS2 methods folded into it | ratio |
|---|---|---|
| `vehicle_get_save_state_size` | `GetVehicleStateSize` + `GetVehicleDamageDeformSize` | 0.66 |
| `vehicle_save_state` | `SaveVehicleState` + `SaveVehicleDamageDeform` | 0.75 |
| `vehicle_load_state` | `LoadVehicleState` + `LoadVehicleDamageDeform` | 0.57 |

This is settled by content rather than by size: all three PC functions walk the pristine-geometry
backup at `+0x288` and run the mesh lock/unlock vertex walk, and that array exists only for the
deform machinery, so a plain vehicle-state save has no reason to touch it. The ratios then
corroborate, and they are only coherent under the fold — 0.66 / 0.75 / 0.57 as pairs against
5.29 / 1.61 / 0.90 as singles. A 136-byte PS2 size query cannot become a 720-byte PC one; the extra
584 bytes are the deform measurement.

**`DoCollisionStatic` is inlined on the PC at both callsites.** The method has exactly two PS2
callsites, `Collide__9C_railwayFv` and `AI__7C_wagonFUi`, both in the kind-4 arm and both invoking it
on `car + 0x80`. On the PC both twins, `railway_collide` and `wagon_ai`, call
`vehicle_get_contact_impulse_terms` directly, once each, at the mirrored position, and neither calls
any separate response function. The fold runs the other way on the PS2: `DoCollisionStatic`'s callee
list there is pure vector-unit math (`__amu__8S_vector` ×4, `Cross` ×3) with no call to
`GetRVelMassJ`, so the PS2 inlined `GetRVelMassJ` into it while the PC inlined it into its callers.
The 2-for-2 is not an artifact of a shared callee: the symbol's other two PC callers,
`box_solve_impulse_vs_world` and `car_get_collision_info`, are the generic kind-dispatched query the
PS2 also has, and no third PC function mirrors a PS2 `DoCollisionStatic` call.

**`BodyCollision`'s receiver differs.** The PS2 declares it a `C_Vehicle` method taking an
`S_COLBODY_HEAD*`; on the PC, MSVC bound `this` to the head itself and callers pass `+0x528`. The
class membership is unchanged — only the receiver moved.

**The pristine geometry lives in different places.** PS2 `InitDeform` restores from a file-scope
`deform_orig`; the PC restores from the member vector at `+0x288`.

**Developers' names recovered from the PS2 statics.** `s_material_conv_tbl__9C_Vehicle` and
`v_particle__9C_Vehicle` are the original names for `g_vehicle_material_index_by_id` and
`g_vehicle_particles_begin`. `S_hlp$2695C_actor_cpp`-style source tags likewise supply the four
`C_Vehicle*.cpp` file names of §6.

**The container helper tails are not comparable by count:** 4 PC functions against 10 PS2 template
symbols in the kernel unit, and two of the four are shared across unrelated classes
(`std_vector74_size`, `std_vector8_erase_range`) and carry no class prefix.

**Two genuinely foreign functions sit inside the effects unit** and are correctly named:
`i3d_frame_get_local_direction_normalized` (3 callsites, 2 unrelated containers) and
`i3d_frame_get_parent_ptr` (29 callsites, 3 unrelated containers). MSVC emitted them into whichever
object first instantiated them.

**The `S_anim_info`-style internal ordering caveat applies to the record types too:** the element
layouts differ between builds even where the algorithm over them does not — the door record is
`0x24` bytes on the PC and `0x30` on the PS2, the wheel record `0x1c` against `0x30`, and the
skid-mark ring slot `0x14` against `0x0c`.

## 10. Related

- `structs/car.md` — the `C_car` entity that owns this object at `car+0x70`.
- `structs/driver.md` — the `C_driver` chase AI that calls these setters.
- `structs/pursuit-event-manager.md` — `C_sledovacka`, whose translation unit hosts the §1a
  accessors.
- `structs/wagon.md` — `C_wagon`, the next class in link order.
- `basic/artifacts/vehicle-movement.md` — the per-frame integrator, wheel solver and keyframe
  playback.
- `basic/artifacts/hud_elements.md` §6b — the tachometer that consumes `vehicle_get_engine_rpm`.
- `script-mapping/artifacts/opcodes-verified.md` — the `carlight_*` script cases of §1b.
- `police-system/artifacts/boosting.md`, `def-files-docs/vehicles-bin.md` — the CARINIT model levels.

## 11. Open items

| Item | State |
|---|---|
| light bit `0x80` | script case `0xc1` `carlight_light` sets it, but testing it in game produced no visible effect and no consumer explains it |
| light bit `0x800` | it gates bit `0x100` (brake lights); no script setter writes it and no producer has been located |
| wheel flag `+0x120 & 0x40000000` | the chase AI treats it as terminal, but the bit's own meaning — burst, destroyed — is not proven. The spike strip (`structs/spike_strip.md`) is the obvious suspect and has not been checked |
| `add_track`, the vector at `S_COLBODY_HEAD+0x3c` | `vehicle_clear_body_coll_add_track` zeroes it every `GameInit` and the PS2 supplies the field name, but what accumulates there is unknown: an offset census across the band finds no writer with a literal displacement |
| the two kernel sort vectors | the PS2 instantiates `vector<S_SORT>` and `vector<S_COLBODY_EMITOR>` in that unit and nowhere else, so those are the element types, but which vector has which is not resolved (§6d) |
| `vehicle_get_seat_count` | it counts the `+0xd20` record array, which also serves the doors and the windows. A car has one record per door, so all three counts coincide and no count has ever contradicted any of the three readings — but the name has not been settled against the PS2 body |
| bank slot `+0x384` | `car_init` never writes it, and no PC setter targets it; `SetCallbackLP` is the PS2 method whose slot it would be |
| `IsDoor` | no PC start located — the odd one out of an otherwise forced 4-for-5 door block |
| `UpdateDeformPart` | no PC start at its rank; a fold into `vehicle_deform` is the leading hypothesis, unverified |
| `DuplicateDamageAndDeform` | 2156 B on the PS2, no PC start; the copy-assign family in the first unit's tail is the leading lead for what a PC `Duplicate` would be built from |
| the other unlocated PS2 methods | listed in §3a; each is a candidate for another of the four units before it is a candidate for inlining |
