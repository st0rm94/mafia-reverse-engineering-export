# `tables\vehicles.bin` — per-car physics / CARINIT data (exhaustive field map)

**Created** 2026-07-09 21:46 +0200 · **Modified** 2026-10-12 20:59 +0200

The master **car parameter file**: one record per car model, each holding one or more **CARINIT
levels** (tuning stat blocks). A car with **two** levels is *boostable* — level 0 is the normal
block, level 1 the boosted one the pursuit system activates during a chase
(`car_set_model_level` → `vehicle_apply_carinit_params`; the chain is in
`police-system/artifacts/boosting.md`). The companion `vehicles.dat` (3856 B) is the field schema.

Field **names** come from the community editor **RHAM 3.0**; the older **MCVE** mislabels some —
field #2 is RHAM's "Power", not "MaxSpeedKMH", and it is *not* top speed. Both editors write
**level 0 only**. Every value below is byte-verified against the shipped file.

## 1. Container
```
u32 version = 1
u32 car_count = 89
car_record records[89]   // packed, no index; ends exactly at EOF 0x6F3D3
```

## 2. Record
```
u32  record_size;   // 0x1149 = 1 level, 0x226E = 2 levels
char name[0x20];    // "blackha", "poli", ...
u32  level_count;   // 1 = normal, 2 = boostable
level_block levels[level_count];
```
`level_count` is what runtime `car_get_model_level_count()` returns. Boostability is this field,
**not** the model files. The 14 two-level cars are listed in §6.

## 3. Level (CARINIT) block
```
u32 block_size = 0x1121 (4385)
u8  carinit[0x1121]
```
The block is the car's packed save record (`STCars`, `0x111d` bytes, packed without alignment)
followed by 4 bytes no reader touches. `car_load_model_level_tables` reads it and
`car_unpack_save_record` expands it into an `S_CARINIT` (`0x1124` bytes, aligned, some arrays
transposed); `car_pack_save_record` is the inverse, and the mission save stream stores the same
record. Every offset below is an offset into the packed block. The layout tiles all 103 shipped level
blocks with no overlap, and every 16-byte name slot holds a NUL-terminated filename or nothing.

| region | offsets | contents |
|---|---|---|
| Header | `0x000`-`0x11F` | RHAM rows 1-65 and seven steering angles (§4a) |
| Axle records | `0x120`-`0x5C5` | 10 records of `0x77` bytes, one per axle (§4b) |
| Scalar block | `0x5C6`-`0x7C7` | packed scalars (§4c) |
| Sound records | `0x7C9`-`0x111C` | filename slots and engine-sound records (§4d) |
| Unread | `0x7C8`, `0x111D`-`0x1120` | 0, and `00 00 00 18`, in every block |

Every field of the `blackha` (Falconer) block is enumerated below. In each table **boost** =
`**YES**` when the four bytes differ between level 0 and level 1, `-` when they are identical.

## 4a. Header (0x000-0x11F)

| off | RHAM row | field | L0 | L1 | boost |
|---|---|---|---|---|---|
| +0x000 | 1 | Weight: the body mass (`C_Vehicle+0x674`) | 1520 | 1520 | - |
| +0x004 | 2 | Power | 58.8 | 160 | **YES** |
| +0x008 | 3 | GearSpeedFactor | 78 | 88 | **YES** |
| +0x00C | 4 | RedLineRPM | 3000 | 3000 | - |
| +0x010 | 5 | maximum rpm | 3300 | 3300 | - |
| +0x014 | 6 | idling rpm | 800 | 800 | - |
| +0x018 | 7 | Max Power at idling RPM | 150 | 450 | **YES** |
| +0x01C | 8 | Min Power at redline RPM | 125 | 200 | **YES** |
| +0x020 | 9 | PowerBandKwMax | 90 | 200 | **YES** |
| +0x024 | 10 | PowerBandRPMlo | 1900 | 1900 | - |
| +0x028 | 11 | PowerBandRPMhi | 2800 | 2800 | - |
| +0x02C | 12 | Hit Points | 120 | 120 | - |
| +0x030 | 13 | Motor time | 0.28 | 0.4 | **YES** |
| +0x034 | 14 | SkidFactor | 250 | 550 | **YES** |
| +0x038 | 15 | Shift time | 0.2 | 0.2 | - |
| +0x03C | 16 | Motor speed | 0.3 | 0.3 | - |
| +0x040 | 17 | Down Force Front | 800 | 800 | - |
| +0x044 | 18 | Down Force Rear | 800 | 800 | - |
| +0x048 | 19 | GEAR 1 ON | 2600 | 2600 | - |
| +0x04C | 20 | GEARS | 4 | 4 | - |
| +0x050 | 21 | Reverse gear | 14 | 14 | - |
| +0x054 | 22 | Reverse shift up | 0 | 0 | - |
| +0x058 | 23 | Reverse shift down | 0 | 0 | - |
| +0x05C | 24 | GEAR 1 ratio | 15 | 15 | - |
| +0x060 | 25 | GEAR 1 shift up | 80 | 80 | - |
| +0x064 | 26 | GEAR 1 shift down | 0 | 0 | - |
| +0x068 | 27 | GEAR 2 ratio | 8 | 8 | - |
| +0x06C | 28 | GEAR 2 shift up | 78 | 78 | - |
| +0x070 | 29 | GEAR 2 shift down | 15 | 12 | **YES** |
| +0x074 | 30 | GEAR 3 ratio | 5.5 | 5.5 | - |
| +0x078 | 31 | GEAR 3 shift up | 80 | 80 | - |
| +0x07C | 32 | GEAR 3 shift down | 34 | 28 | **YES** |
| +0x080 | 33 | GEAR 4 ratio | 4.4 | 4.4 | - |
| +0x084 | 34 | GEAR 4 shift up | 100 | 100 | - |
| +0x088 | 35 | GEAR 4 shift down | 50 | 43 | **YES** |
| +0x08C | 36 | GEAR 5 ratio | 0 | 0 | - |
| +0x090 | 37 | GEAR 5 shift up | 0 | 0 | - |
| +0x094 | 38 | GEAR 5 shift down | 0 | 0 | - |
| +0x098 | 39 | GEAR 6 ratio | 0 | 0 | - |
| +0x09C | 40 | GEAR 6 shift up | 0 | 0 | - |
| +0x0A0 | 41 | GEAR 6 shift down | 0 | 0 | - |
| +0x0A4 | 42 | GEAR 7 ratio | 0 | 0 | - |
| +0x0A8 | 43 | GEAR 7 shift up | 0 | 0 | - |
| +0x0AC | 44 | GEAR 7 shift down | 0 | 0 | - |
| +0x0B0 | 45 | GEAR 8 ratio | 0 | 0 | - |
| +0x0B4 | 46 | GEAR 8 shift up | 0 | 0 | - |
| +0x0B8 | 47 | GEAR 8 shift down | 0 | 0 | - |
| +0x0BC | 48 | GEAR 9 ratio | 0 | 0 | - |
| +0x0C0 | 49 | GEAR 9 shift up | 0 | 0 | - |
| +0x0C4 | 50 | GEAR 9 shift down | 0 | 0 | - |
| +0x0C8 | 51 | GEAR 10 ratio | 0 | 0 | - |
| +0x0CC | 52 | GEAR 10 shift up | 0 | 0 | - |
| +0x0D0 | 53 | GEAR 10 shift down | 0 | 0 | - |
| +0x0D4 | 54 | 0,1-1 | 0.35 | 0.35 | - |
| +0x0D8 | 55 | Windage | 1.5 | 1.5 | - |
| +0x0DC | 56 | Front down speed | 1.2 | 1.2 | - |
| +0x0E0 | 57 | Front down | 1.4 | 1.4 | - |
| +0x0E4 | 58 | xRear down speed | 1.2 | 1.2 | - |
| +0x0E8 | 59 | Rear down | 1.4 | 1.4 | - |
| +0x0EC | - | steering lock at standstill, degrees, at most 50 (`C_Vehicle+0x684`) | 45 | 45 | - |
| +0x0F0 | - | steering lock at top speed, degrees, at most 50 (`+0x688`) | 25 | 25 | - |
| +0x0F4 | - | split between the two steered wheels at full lock, degrees, at most `+0x0EC` (`+0x6a8`) | 10 | 10 | - |
| +0x0F8 | - | turn-in rate at standstill, degrees per second (`+0x690`) | 50 | 50 | - |
| +0x0FC | - | return rate at standstill, degrees per second (`+0x694`) | 90 | 90 | - |
| +0x100 | - | turn-in rate at top speed, degrees per second (`+0x698`) | 16 | 16 | - |
| +0x104 | - | return rate at top speed, degrees per second (`+0x69c`) | 50 | 50 | - |
| +0x108 | 60 | Axis Z rotate | 0.5 | 0.5 | - |
| +0x10C | 61 | Center mass | 0.16 | 0.05 | **YES** |
| +0x110 | 62 | Axis X rotate | 0.6 | 0.52 | **YES** |
| +0x114 | 63 | Cohesion with road | 10 | 10 | - |
| +0x118 | 64 | Petrol tank | 60 | 60 | - |
| +0x11C | 65 | Fuel consumption | 20 | 20 | - |

The seven steering angles feed `vehicle_update_steering_wheels`. It interpolates the lock and both
rates between their standstill and top-speed values by `1 − speed / top speed`, where the top speed
`C_Vehicle+0x404` is the top-gear speed at maximum rpm. The return rate applies while the wheel moves
back toward the centre; the split is added to one steered wheel and subtracted from the other, scaled
by the current steering angle over the lock.

## 4b. Axle records (0x120-0x5C5)

Record *n* starts at `0x120 + 0x77·n`: 24 dwords, three bytes, five dwords. `vehicle_apply_carinit_params`
loops over the axle count `C_Vehicle+0x53c` and applies record *n* to wheels 2*n* and 2*n*+1 of the
wheel array `C_Vehicle+0xcb4`. `blackha` uses record 0 (front axle) and record 1 (rear axle); the other
eight hold defaults. The record offset is relative to the record start (front `+0x120`, rear `+0x197`);
the wheel field is the wheel object's offset, and the conversion is applied on load.

| record | type | wheel field | on load | front L0 | rear L0 | front L1 | rear L1 | note |
|---|---|---|---|---|---|---|---|---|
| `+0x00` | f32 | `+0x88` | ×1000 | 68 | 66 | 52 | 52 | the spring rate: the suspension force is `(+0x8c − extension) · +0x88`. RHAM *Front weigh* / *Rear weigh* |
| `+0x04` | f32 | `+0x8c` | at least 0.009 | 0.2 | 0.18 | 0.24 | 0.25 | the suspension travel: the extension at which the spring force reaches zero, also part of the contact probe's length. RHAM *Suspension forward (height)* / rear height |
| `+0x08` | f32 | `+0x98` | - | 1400 | 1400 | 1400 | 1400 | the BUMP damping: multiplies the compression speed while the wheel compresses. RHAM *Shock-absorber forward* |
| `+0x0C` | f32 | `+0x9c` | - | 2600 | 2600 | 2600 | 2600 | the REBOUND damping: multiplies the extension speed while the wheel extends. RHAM *Shock-absorber rear* |
| `+0x10` | i32 | `+0xa8` | tyre id, resolved to the tyre index `+0xac` | 18468 | 18468 | 26501 | 6335 | RHAM *Forward control y/n*; ids in `tables\tyres.bin` ¹ |
| `+0x14` | f32 | `+0xb8` | ×1000 | 12 | 12 | 12 | 12 | set 1 |
| `+0x18` | f32 | `+0xbc` | degrees, at least 1 | 9 | 9 | 7 | 6 | set 1 |
| `+0x1C` | f32 | `+0xc0` | - | 0.9 | 0.9 | 0.9 | 0.9 | set 1 |
| `+0x20` | f32 | `+0xc4` | km/h | 40 | 40 | 40 | 40 | set 1 |
| `+0x24` | f32 | `+0xd0` | stored as `+0xd4` minus the value | 1.3 | 1.3 | 1.4 | 1.6 | set 1 |
| `+0x28` | f32 | `+0xd4` | - | 1.5 | 1.5 | 1.7 | 1.9 | set 1 |
| `+0x2C` | f32 | `+0xd8` | stored as `+0xdc` minus the value | 1.3 | 1.3 | 1.4 | 1.6 | set 1 |
| `+0x30` | f32 | `+0xdc` | - | 1.5 | 1.5 | 1.7 | 1.9 | set 1 |
| `+0x34` | f32 | `+0xe0` | - | 0.01 | 0.01 | 0.01 | 0.01 | set 1 |
| `+0x38` | f32 | `+0xe4` | ×1000 | 12 | 12 | 12 | 12 | set 2 |
| `+0x3C` | f32 | `+0xe8` | degrees, at least 1 | 9 | 9 | 9 | 9 | set 2 |
| `+0x40` | f32 | `+0xec` | - | 0.9 | 0.9 | 0.9 | 0.9 | set 2 |
| `+0x44` | f32 | `+0xf0` | km/h | 40 | 40 | 40 | 40 | set 2 |
| `+0x48` | f32 | `+0xf4` | stored as `+0xf8` minus the value | 1.1 | 1.1 | 1.1 | 1.1 | set 2 |
| `+0x4C` | f32 | `+0xf8` | - | 1.35 | 1.35 | 1.35 | 1.35 | set 2 |
| `+0x50` | f32 | `+0xfc` | stored as `+0x100` minus the value | 1.1 | 1.1 | 1.1 | 1.1 | set 2 |
| `+0x54` | f32 | `+0x100` | - | 1.35 | 1.35 | 1.35 | 1.35 | set 2 |
| `+0x58` | f32 | `+0x104` | - | 0.2 | 0.2 | 0.2 | 0.2 | set 2 |
| `+0x5C` | f32 | `+0x11c` | percent, 0..100 | 100 | 100 | 100 | 100 | - |
| `+0x60` | u8 | flag `0x2` | - | 0 | 1 | 0 | 1 | driven: only driven wheels set the drive circumference `C_Vehicle+0x628` |
| `+0x61` | u8 | flag `0x1` | - | 1 | 0 | 1 | 0 | steered: `vehicle_update_steering_wheels` turns only these |
| `+0x62` | u8 | flag `0x4` | - | 0 | 1 | 0 | 1 | handbrake: `vehicle_move` applies `C_Vehicle+0x4b0` only to these |
| `+0x63` | f32 | `+0x118` | percent of `C_Vehicle+0x440` | 95 | 80 | 95 | 80 | even wheel of the axle |
| `+0x67` | f32 | `+0x118` | percent of `C_Vehicle+0x440` | 95 | 100 | 95 | 100 | odd wheel of the axle |
| `+0x6B` | f32 | `+0x170` | degrees, 1..90 | 27.5 | 11 | 27.5 | 11 | - |
| `+0x6F` | f32 | `+0x174` | degrees, 1..90, at most `+0x73` | 21 | 9 | 22 | 9 | - |
| `+0x73` | f32 | `+0x178` | degrees, 1..90 | 42 | 15 | 42 | 15 | - |

¹ `tables\tyres.bin` holds four tyre types: 42 `michellin`, 18468 `Normal`, 6335 `Racing`, 26501
`Semi racing`. `blackha` drives on Normal tyres at level 0 and on Semi racing (front) and Racing (rear)
when boosted.

RHAM rows 66-74 are fields of these records: *Front weigh*, *Suspension forward (height)*, both
*Shock-absorber* rows and *Forward control y/n* are front-axle fields `+0x120`-`+0x130` (the rear axle has
its own copies at `+0x197`-`+0x1A7`). Every axle record carries both damping values, so RHAM's
*forward* / *rear* shock-absorber labels name the two damping directions, not the two axles. The four
suspension fields are read by `vehicle_wheel_contact_solve` (`basic/artifacts/vehicle-movement.md` §3); *Rear weigh* is `+0x197`, the rear suspension height `+0x19B`,
the drive layout is the driven byte of each axle (`+0x180` front, `+0x1F7` rear), and the roof flag is
the scalar `+0x5F6` (§4c).

## 4c. Scalar block (0x5C6-0x7C7)

Packed: a field can start at any byte. **runtime field** = the `C_Vehicle` field
`vehicle_apply_carinit_params` writes directly, with the conversion applied on load; `-` = a field the
consumer reads through one of its array loops. A range row lists its fields' values in order.

| off | type | runtime field | L0 | L1 |
|---|---|---|---|---|
| `+0x5C6` .. `+0x5CE` | f32 ×3 | - | 2500 1900 2500 | 2500 1900 2500 |
| `+0x5D2` | f32 | `+0x440` ×1000 | 7 | 7 |
| `+0x5D6` | f32 | `+0x444` | 1 | 1 |
| `+0x5DA` | f32 | `+0x42c` ×1000 | 60 | 80 |
| `+0x5DE` | f32 | `+0x430` | 4 | 1 |
| `+0x5E2` | u8 | `+0x1bc` | 0 | 0 |
| `+0x5E3` | f32 | `+0x408` | 1 | 1 |
| `+0x5E7` | f32 | `+0x40c` | 1.1 | 1.1 |
| `+0x5EB` | i32 | `+0x2a4` | 20 | 20 |
| `+0x5EF` | u8 | `+0x2a8` | 0 | 0 |
| `+0x5F0` | u8 | `+0x2a9` | 0 | 0 |
| `+0x5F1` | u8 | - | 0 | 0 |
| `+0x5F2` | f32 | `+0x18` | 1 | 1 |
| `+0x5F6` | u8 | `+0x119` | 1 | 1 |
| `+0x5F7` | i32 | `+0x4dc` | 12 | 12 |
| `+0x5FB` | f32 | `+0x4e4` km/h | 35 | 35 |
| `+0x5FF` | f32 | `+0x4e8` | 0.8 | 0.8 |
| `+0x603` | f32 | `+0x4ec` | 2.4 | 2.4 |
| `+0x607` | f32 | `+0x344` | 0 | 0 |
| `+0x60B` .. `+0x60F` | f32 ×2 | - | 3 -4 | 3 -4 |
| `+0x613` | f32 | `+0x350` | 2 | 2 |
| `+0x617` | u8 | - | 0 | 0 |
| `+0x618` .. `+0x628` | f32 ×5 | - | 0.7 0.1 0.7 0.1 0.03 | 0.7 0.1 0.7 0.1 0.03 |
| `+0x62C` | f32 | `+0x240` km/h | 120 | 120 |
| `+0x630` .. `+0x634` | f32 ×2 | - | 25 100 | 25 100 |
| `+0x638` | f32 | `+0x23c` | 45 | 45 |
| `+0x63C` | f32 | `+0x22c` | 0.4 | 0.4 |
| `+0x640` | f32 | `+0x248` | 60 | 60 |
| `+0x644` | f32 | `+0x24c` | 30 | 30 |
| `+0x648` | f32 | `+0x250` km/h | 35 | 35 |
| `+0x64C` | i32 | `+0x254` | 13 | 13 |
| `+0x650` | i32 | - | 14 | 14 |
| `+0x654` | f32 | `+0x278` km/h | 120 | 120 |
| `+0x658` | f32 | `+0x27c` km/h | 25 | 25 |
| `+0x65C` | f32 | `+0x270` | 100 | 100 |
| `+0x660` | f32 | `+0x274` | 40 | 40 |
| `+0x664` .. `+0x678` | f32 ×6 | - | 120 20 100 30 10 60 | 120 20 100 30 10 60 |
| `+0x67C` | f32 | `C_car+0x1ff4` | 110.526 | 110.526 |
| `+0x680` | f32 | `C_car+0x1ff8` | 18.4211 | 18.4211 |
| `+0x684` | f32 | `C_car+0x1ffc` | 100 | 100 |
| `+0x688` | f32 | `C_car+0x2000` | 65 | 65 |
| `+0x68C` .. `+0x6DC` | f32 ×21 | - | 70 10 20 30 40 35 70 10 100 15 25 15 15 10 1 100 0 1 1 1 80 | 70 10 20 30 40 35 70 10 100 15 25 15 15 10 1 100 0 1 1 1 80 |
| `+0x700` | f32 | `+0x144` km/h | 50 | 50 |
| `+0x704` | f32 | `+0x148` km/h | 10 | 10 |
| `+0x708` | f32 | `+0x14c` | 100 | 100 |
| `+0x70C` | f32 | `+0x150` | 70 | 70 |
| `+0x710` | f32 | `+0x154` | 20 | 20 |
| `+0x714` .. `+0x7A8` | f32 ×38 | - | 2 2 2 50 10 100 0 5 5 5 50 10 100 20 5 5 5 50 10 100 20 10 5 10 50 10 100 20 10 5 5 50 10 100 20 10 5 5 | 2 2 2 50 10 100 0 5 5 5 50 10 100 20 5 5 5 50 10 100 20 10 5 10 50 10 100 20 10 5 5 50 10 100 20 10 5 5 |
| `+0x7AC` | f32 | `+0x208` km/h | 9 | 9 |
| `+0x7B0` | f32 | `+0x20c` km/h | 27 | 27 |
| `+0x7B4` | f32 | `+0x210` | 5 | 5 |
| `+0x7B8` | f32 | `+0x6b8` | 0 | 0 |
| `+0x7BC` | f32 | `+0x6bc` | 20 | 20 |
| `+0x7C0` | f32 | `+0x6c0` | 5.5 | 5.5 |
| `+0x7C4` | f32 | `+0x6c4` | 0.9 | 1.4 |

* `+0x5DA` / `+0x5DE` are the handbrake force and ramp (`C_Vehicle+0x42c` / `+0x430`).
* `+0x5E2` set (9 cars: the hot rods, `toyota`, `xedos`) makes `vehicle_body_coll_resolve` use the car's
  own friction `+0x5E3` and impulse factor `+0x5E7` instead of the surface material's
  (`def-files-docs/materialy.md` §1a).
* `+0x5F6` is the roof flag (`C_Vehicle+0x119`, = `C_car+0x189`). It is clear on the 27 open bodies
  (roadsters, phaetons, speedsters, tourers, pickups, hot rods, race cars) and set on the other 62;
  `car_on_collision_apply_damage` applies its hit-from-above occupant damage only to cars without it.
* `+0x617` set (`FThot`, `toyota`, `xedos`) switches off the surface bumps of
  `def-files-docs/materialy.md` §1a for the car (`C_Vehicle+0x1bd`).
* `+0x67C`-`+0x688` go to the `C_car`, not to the `C_Vehicle`; the first two are scaled by
  `C_car+0x6e0` and converted from km/h on load.

## 4d. Sound records (0x7C9-0x111C)

| offset | count | size | contents |
|---|---|---|---|
| `0x7C9` | 3 | `0x10` | filename slots |
| `0x7F9` | 5 | `0x31` | engine-sound records, set A |
| `0x8EE` | 5 | `0x31` | engine-sound records, set B |
| `0x9E3` | 5 | `0x30` | engine-sound records without the flag byte, set C |
| `0xAD3` | 1 | `0x18` | filename and two params |
| `0xAEB` | 1 | `0x24` | filename and five params |
| `0xB0F` | 10 | `0x10` | filename slots |
| `0xBAF` | 4 | `0x25` | crash records: two filenames, one param, one byte |
| `0xC43` | 6 | `0x10` | filename slots |
| `0xCA3` | 4 | 4 | params |
| `0xCB3` | 5 | `0x31` | engine-sound records, set D |
| `0xDA8` | 5 | `0x31` | engine-sound records, set E |
| `0xE9D` | 5 | `0x30` | engine-sound records without the flag byte, set F |
| `0xF8D` | 1 | `0x18` | filename and two params |
| `0xFA5` | 1 | `0x24` | filename and five params |
| `0xFC9` | 10 | `0x10` | filename slots |
| `0x1069` | 4 | `0x25` | crash records |
| `0x10FD` | 2 | `0x10` | filename slots |

An engine-sound record is a 16-byte filename, eight floats and (in sets A, B, D, E) a flag byte.
`vehicle_apply_carinit_params` validates the floats and `vehicle_update_engine_pitch` plays them. Rpm
values are percent of the range from idle to maximum rpm.

| param | meaning | load check |
|---|---|---|
| `p0` | pitch at the lowest audible rpm | at least 0.5 |
| `p1` | pitch at the highest audible rpm | above `p0`, at most 2.0 |
| `p2` | lowest audible rpm | 0..100 |
| `p3` | highest audible rpm | `p2`..100 |
| `p4` | rpm where the fade-in ends | 0..100 |
| `p5` | rpm where the fade-out starts | `p4`..100 |
| `p6` | volume at full weight | 0..1 |
| `p7` | volume at zero weight | 0..1 |
| flag | muted while the effective gear `C_Vehicle+0x5d8` is 0 | - |

The sound plays only while the rpm lies in [`p2`, `p3`]; its pitch runs linearly from `p0` to `p1`
across that band; its volume is `p7 + (p6 − p7)·w`, where *w* is the throttle for some sets and one
minus the throttle for others, scaled up linearly from `p2` to `p4` and down from `p5` to `p3`.

The `blackha` filenames (a slot whose first byte is NUL is empty even when text follows it):

| off | L0 | boost |
|---|---|---|
| `+0x7C9` | `engine_on.wav` | - |
| `+0x7D9` | `engine_off.wav` | - |
| `+0x7E9` | `engine_on_nf.wa` (cut to the slot's 15 characters) | - |
| `+0x7F9` | `interier.wav` | - |
| `+0x8EE` | `interier_X.wav` | - |
| `+0x9E3` | `interier.wav` | - |
| `+0xAD3` | `interier_0.wav` | - |
| `+0xAEB` | `interier_A.wav` | - |
| `+0xB0F` | `horn1.wav` | - |
| `+0xB2F` | `hand2.wav` | - |
| `+0xB3F` / `+0xB4F` | `gear1.wav` | - |
| `+0xB5F` | `slip1.wav` | - |
| `+0xB6F` / `+0xB7F` | empty (`\0rzdy_loop.wav`, `\0hit3.wav`; 52 other blocks name `brzdy_loop.wav` here) | - |
| `+0xB9F` | `carglass.wav` | - |
| `+0xBAF`-`+0xC2E` | `crashA1in` / `crashA2in`, `crashK1in` / `crashK2in`, `crashB1in` / `crashB2in`, `crashC1in` / `crashC2in` (`.wav`), params 1.5 / 2.5 / 9 / 35 | - |
| `+0xC63` / `+0xC73` | `black_on.wav` / `black_off.wav` | - |
| `+0xC83` | `startbad1.wav` | - |
| `+0xC93` | `black_NPC.wav` | - |
| `+0xCB3` / `+0xCE4` / `+0xD15` / `+0xD46` | `black_1.wav`, `buick_2.wav`, `black_3.wav`, `s1.wav` | `p1` of the first three |
| `+0xDA8` / `+0xDD9` | `cheve_X.wav`, `s1.wav` | `p1` of the first |
| `+0xE9D` / `+0xECD` / `+0xEFD` | `black_1.wav`, `revers3.wav`, `buick_2.wav` | `p7` of the second |
| `+0xF8D` | `black_0.wav` | - |
| `+0xFC9` | `horn5.wav` | - |
| `+0xFE9` | `hand2.wav` | - |
| `+0xFF9` / `+0x1009` | `gear2.wav` / `gear1.wav` | **YES** (empty at L1) |
| `+0x1019` | `slip1.wav` | - |
| `+0x1029` / `+0x1039` | `break.wav` / `cep.wav` | **YES** (empty at L1) |
| `+0x1049` | `pneu1.wav` | - |
| `+0x1059` | `carglass.wav` | - |
| `+0x1069`-`+0x10E8` | `crashA1` / `crashA2`, `crashK1` / `crashK2`, `crashB1` / `crashB2`, `crashC1` / `crashC2` (`.wav`), params 1.5 / 2.5 / 9 / 35 | - |
| `+0x10FD` / `+0x110D` | `door04op.wav` / `door04cl.wav` | - |

## 5. The boost (level 0 → level 1)

Boost raises **engine power, grip, throttle response and gear behaviour**, and leaves the fields
that set **top speed unchanged**: `RedLineRPM`, `maximum rpm`, `GEARS` and every gear ratio are
identical between the two levels, because top speed is derived rather than stored. The tuning
fields that differ, from §4a:

| field | L0 | L1 |
|---|---|---|
| Power | 58.8 | 160 |
| GearSpeedFactor | 78 | 88 |
| Max Power at idling RPM | 150 | 450 |
| Min Power at redline RPM | 125 | 200 |
| PowerBandKwMax | 90 | 200 |
| Motor time | 0.28 | 0.4 |
| SkidFactor | 250 | 550 |
| GEAR 2 shift down | 15 | 12 |
| GEAR 3 shift down | 34 | 28 |
| GEAR 4 shift down | 50 | 43 |
| Center mass | 0.16 | 0.05 |
| Axis X rotate | 0.6 | 0.52 |
| Front weigh | 68 | 52 |
| Suspension forward (height) | 0.2 | 0.24 |

More fields differ outside the header: in both axle records the spring rate `+0x00`, the suspension
travel `+0x04`, the tyre id `+0x10`, the set-1 angle `+0x18` and the set-1 pairs `+0x24`-`+0x30`, plus
the front `+0x6F`; in the scalar block the handbrake force and ramp `+0x5DA` / `+0x5DE` and `+0x7C4`;
in the sound records `p1` of four engine sounds and `p7` of one, and four filenames that are empty at
level 1 (§4b-§4d).

## 6. Boostable cars (`level_count == 2`) — all 14

`blackha`, `buikFor`, `cad_ford`, `forVfor`, `bugatti`, `hudfor`, `arrow`, `polCad`, `poli`,
`polimFor`, `polimTud`, `black`, `buigang`, `chuligani`.

Body variants are separate records with independent level counts: `hudfor` (Terraplane Fordor) has
2 levels, `hudtu` (Tudor) has 1.

## 7. `vehicles.dat` — the field schema

3856 B. Header `u32 total (= 0xF0C)`, one further `u32`, then `u32 field_count (= 83)`, followed by
the per-field descriptors. Their byte-offset column steps by 4 across the CARINIT floats and each
carries a type code — `5` = float, `0x11` = other. The file is a layout/type table only; it holds no
per-car data. No binary of the game opens it: the name occurs in none of the decrypted `Game.exe`
variants or shipped DLLs, and the engine reads `vehicles.bin` at the offsets of §4.

## 8. Cross-references

- `police-system/artifacts/boosting.md` — how the pursuit system activates level 1.
- `structs/pursuit-event-manager.md` — the `C_sledovacka` chase system.
- `structs/car.md` and `structs/vehicle.md` — the runtime classes the block is loaded into.
- `def-files-docs/materialy.md` §1a — the surface values the own-friction pair and the bump switch
  override.
- Engine entry points: `car_load_model_level_tables` (`0x00465290`), `car_unpack_save_record`,
  `car_get_model_level_count` (`0x00465a90`), `car_set_model_level` (`0x00465ad0`),
  `vehicle_apply_carinit_params` (`0x005123d0`), `vehicle_update_steering_wheels`,
  `vehicle_update_engine_pitch`.

## 9. Open items

| item | state |
|---|---|
| the axle-record fields `+0x14`-`+0x58` (the two parameter sets), `+0x5C` and the angles `+0x6B`-`+0x73` | runtime wheel fields and conversions are decoded (§4b); what each one means physically is not established |
| the scalar-block fields with `-` in §4c | read through the consumer's array loops; not traced field by field |
