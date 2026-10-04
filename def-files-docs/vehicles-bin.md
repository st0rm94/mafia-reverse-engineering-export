# `tables\vehicles.bin` — per-car physics / CARINIT data (exhaustive field map)

**Created** 2026-07-09 21:46 +0200 · **Modified** 2026-09-30 23:08 +0200

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
Region map of one block (offsets relative to the block's first byte):

| region | offsets | contents |
|---|---|---|
| Tuning header | 0x000-0x0E8 | RHAM rows 1-59 (4a) |
| Internal fields | 0x0EC-0x104 | 7 fields not shown by RHAM (4a) |
| Extended physics | 0x108-0x130 | RHAM rows 60-70 (4a) |
| Geometry/wheel/suspension | 0x134-0x7C8 | raw float block, unlabeled (4b); RHAM rows 71-74 are not individually pinned |
| Sound/string records | 0x7C9-0x1121 | filename slots + interleaved params (4c) |

Every field of the `blackha` (Falconer) block is enumerated below. In each table **boost** =
`**YES**` when the four bytes differ between level 0 and level 1, `-` when they are identical.

## 4a. Tuning header + extended physics (0x000-0x130)

| off | RHAM row | field | L0 | L1 | boost |
|---|---|---|---|---|---|
| +0x000 | 1 | Weight [inferred] | 1520 | 1520 | - |
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
| +0x0EC | - | internal (not shown in RHAM UI) | 45 | 45 | - |
| +0x0F0 | - | internal (not shown in RHAM UI) | 25 | 25 | - |
| +0x0F4 | - | internal (not shown in RHAM UI) | 10 | 10 | - |
| +0x0F8 | - | internal (not shown in RHAM UI) | 50 | 50 | - |
| +0x0FC | - | internal (not shown in RHAM UI) | 90 | 90 | - |
| +0x100 | - | internal (not shown in RHAM UI) | 16 | 16 | - |
| +0x104 | - | internal (not shown in RHAM UI) | 50 | 50 | - |
| +0x108 | 60 | Axis Z rotate | 0.5 | 0.5 | - |
| +0x10C | 61 | Center mass | 0.16 | 0.05 | **YES** |
| +0x110 | 62 | Axis X rotate | 0.6 | 0.52 | **YES** |
| +0x114 | 63 | Cohesion with road | 10 | 10 | - |
| +0x118 | 64 | Petrol tank | 60 | 60 | - |
| +0x11C | 65 | Fuel consumption | 20 | 20 | - |
| +0x120 | 66 | Front weigh | 68 | 52 | **YES** |
| +0x124 | 67 | Suspension forward (height) | 0.2 | 0.24 | **YES** |
| +0x128 | 68 | Shock-absorber forward | 1400 | 1400 | - |
| +0x12C | 69 | Shock-absorber rear | 2600 | 2600 | - |
| +0x130 | 70 | Forward control y/n [approx] | 2.588e-41 | 3.714e-41 | **YES** |

## 4b. Geometry / wheel / suspension / collision (0x134-0x7C8)

Raw per-field dump, not exposed by RHAM or MCVE; what the fields mean, and where RHAM rows 71-74
sit, is open (§9). The `idx` column is the field's dword index from the start of the block.

| off | idx | L0 | L1 | boost | note |
|---|---|---|---|---|---|
| +0x134 | 77 | 12 | 12 | - | value |
| +0x138 | 78 | 9 | 7 | **YES** | value |
| +0x13C | 79 | 0.9 | 0.9 | - | value |
| +0x140 | 80 | 40 | 40 | - | value |
| +0x144 | 81 | 1.3 | 1.4 | **YES** | value |
| +0x148 | 82 | 1.5 | 1.7 | **YES** | value |
| +0x14C | 83 | 1.3 | 1.4 | **YES** | value |
| +0x150 | 84 | 1.5 | 1.7 | **YES** | value |
| +0x154 | 85 | 0.01 | 0.01 | - | value |
| +0x158 | 86 | 12 | 12 | - | value |
| +0x15C | 87 | 9 | 9 | - | value |
| +0x160 | 88 | 0.9 | 0.9 | - | value |
| +0x164 | 89 | 40 | 40 | - | value |
| +0x168 | 90 | 1.1 | 1.1 | - | value |
| +0x16C | 91 | 1.35 | 1.35 | - | value |
| +0x170 | 92 | 1.1 | 1.1 | - | value |
| +0x174 | 93 | 1.35 | 1.35 | - | value |
| +0x178 | 94 | 0.2 | 0.2 | - | value |
| +0x17C | 95 | 100 | 100 | - | value |
| +0x180 | 96 | 3.587e-43 | 3.587e-43 | - | value |
| +0x184 | 97 | 6.129e-39 | 6.129e-39 | - | value |
| +0x188 | 98 | 6.129e-39 | 6.129e-39 | - | value |
| +0x18C | 99 | 6.048e-39 | 6.048e-39 | - | value |
| +0x190 | 100 | 6.030e-39 | 6.032e-39 | **YES** | value |
| +0x194 | 101 | 6.075e-39 | 6.075e-39 | - | value |
| +0x198 | 102 | -9.406e+26 | 6.090e-39 | **YES** | value |
| +0x19C | 103 | 5.714e-39 | 5.740e-39 | **YES** | value |
| +0x1A0 | 104 | 6.308e-39 | 6.308e-39 | - | value |
| +0x1A4 | 105 | 4.275e-17 | -0.770058 | **YES** | value |
| +0x1A8 | 106 | 1.009e-43 | 3.363e-44 | **YES** | value |
| +0x1AC | 107 | 5.992e-39 | 5.992e-39 | - | value |
| +0x1B0 | 108 | 2.279e+23 | 2.276e+23 | **YES** | value |
| +0x1B4 | 109 | 5.822e-39 | 5.822e-39 | - | value |
| +0x1B8 | 110 | 2.292e+23 | -2.036e+08 | **YES** | value |
| +0x1BC | 111 | 5.845e-39 | 4.466e-08 | **YES** | value |
| +0x1C0 | 112 | 2.264e+23 | -2.013e+08 | **YES** | value |
| +0x1C4 | 113 | 5.845e-39 | 4.466e-08 | **YES** | value |
| +0x1C8 | 114 | 9.232e-33 | 9.242e-33 | **YES** | value |
| +0x1CC | 115 | 5.523e-39 | 5.523e-39 | - | value |
| +0x1D0 | 116 | 5.992e-39 | 5.992e-39 | - | value |
| +0x1D4 | 117 | 2.279e+23 | 2.279e+23 | - | value |
| +0x1D8 | 118 | 5.822e-39 | 5.822e-39 | - | value |
| +0x1DC | 119 | -2.036e+08 | -2.036e+08 | - | value |
| +0x1E0 | 120 | -2.009e+08 | -2.009e+08 | - | value |
| +0x1E4 | 121 | -2.010e+08 | -2.010e+08 | - | value |
| +0x1E8 | 122 | -2.009e+08 | -2.009e+08 | - | value |
| +0x1EC | 123 | -2.010e+08 | -2.010e+08 | - | value |
| +0x1F0 | 124 | 5.721e-39 | 5.721e-39 | - | value |
| +0x1F4 | 125 | 3.578e-38 | 3.578e-38 | - | value |
| +0x1F8 | 126 | 3.587e-43 | 3.587e-43 | - | value |
| +0x1FC | 127 | 2.390e-41 | 2.390e-41 | - | value |
| +0x200 | 128 | 2.396e-41 | 2.396e-41 | - | value |
| +0x204 | 129 | 2.338e-41 | 2.338e-41 | - | value |
| +0x208 | 130 | 2.334e-41 | 2.334e-41 | - | value |
| +0x20C | 131 | 2.347e-41 | 2.347e-41 | - | value |
| +0x210 | 132 | 0 | 0 | - | unused (0) |
| +0x214 | 133 | 0 | 0 | - | unused (0) |
| +0x218 | 134 | 0 | 0 | - | unused (0) |
| +0x21C | 135 | 2.449e-41 | 2.449e-41 | - | value |
| +0x220 | 136 | 0 | 0 | - | unused (0) |
| +0x224 | 137 | 0 | 0 | - | unused (0) |
| +0x228 | 138 | 2.718e+23 | 2.718e+23 | - | value |
| +0x22C | 139 | 2.274e-41 | 2.274e-41 | - | value |
| +0x230 | 140 | 2.372e-41 | 2.372e-41 | - | value |
| +0x234 | 141 | 0 | 0 | - | unused (0) |
| +0x238 | 142 | 0 | 0 | - | unused (0) |
| +0x23C | 143 | 0 | 0 | - | unused (0) |
| +0x240 | 144 | 0 | 0 | - | unused (0) |
| +0x244 | 145 | 0 | 0 | - | unused (0) |
| +0x248 | 146 | 0 | 0 | - | unused (0) |
| +0x24C | 147 | 2.718e+23 | 2.718e+23 | - | value |
| +0x250 | 148 | 2.274e-41 | 2.274e-41 | - | value |
| +0x254 | 149 | 2.372e-41 | 2.372e-41 | - | value |
| +0x258 | 150 | 0 | 0 | - | unused (0) |
| +0x25C | 151 | 0 | 0 | - | unused (0) |
| +0x260 | 152 | 0 | 0 | - | unused (0) |
| +0x264 | 153 | 0 | 0 | - | unused (0) |
| +0x268 | 154 | 0 | 0 | - | unused (0) |
| +0x26C | 155 | 2.396e-41 | 2.396e-41 | - | value |
| +0x270 | 156 | 0 | 0 | - | unused (0) |
| +0x274 | 157 | 0 | 0 | - | unused (0) |
| +0x278 | 158 | -1.084e-19 | -1.084e-19 | - | value |
| +0x27C | 159 | -2.00002 | -2.00002 | - | value |
| +0x280 | 160 | -3.852e-34 | -3.852e-34 | - | value |
| +0x284 | 161 | 9.108e-44 | 9.108e-44 | - | value |
| +0x288 | 162 | 0 | 0 | - | unused (0) |
| +0x28C | 163 | 0 | 0 | - | unused (0) |
| +0x290 | 164 | 131072 | 131072 | - | value |
| +0x294 | 165 | 9.529e-44 | 9.529e-44 | - | value |
| +0x298 | 166 | 0 | 0 | - | unused (0) |
| +0x29C | 167 | 2.524e-29 | 2.524e-29 | - | value |
| +0x2A0 | 168 | 2.720e+23 | 2.720e+23 | - | value |
| +0x2A4 | 169 | 1.084e-19 | 1.084e-19 | - | value |
| +0x2A8 | 170 | 9.249e-44 | 9.249e-44 | - | value |
| +0x2AC | 171 | 0 | 0 | - | unused (0) |
| +0x2B0 | 172 | 0 | 0 | - | unused (0) |
| +0x2B4 | 173 | 0 | 0 | - | unused (0) |
| +0x2B8 | 174 | 0 | 0 | - | unused (0) |
| +0x2BC | 175 | 0 | 0 | - | unused (0) |
| +0x2C0 | 176 | 2.524e-29 | 2.524e-29 | - | value |
| +0x2C4 | 177 | 2.720e+23 | 2.720e+23 | - | value |
| +0x2C8 | 178 | 1.084e-19 | 1.084e-19 | - | value |
| +0x2CC | 179 | 9.249e-44 | 9.249e-44 | - | value |
| +0x2D0 | 180 | 0 | 0 | - | unused (0) |
| +0x2D4 | 181 | 0 | 0 | - | unused (0) |
| +0x2D8 | 182 | 0 | 0 | - | unused (0) |
| +0x2DC | 183 | 0 | 0 | - | unused (0) |
| +0x2E0 | 184 | -131072 | -131072 | - | value |
| +0x2E4 | 185 | 9.249e-44 | 9.249e-44 | - | value |
| +0x2E8 | 186 | 0 | 0 | - | unused (0) |
| +0x2EC | 187 | 0 | 0 | - | unused (0) |
| +0x2F0 | 188 | 20 | 20 | - | value |
| +0x2F4 | 189 | 6 | 6 | - | value |
| +0x2F8 | 190 | 17 | 17 | - | value |
| +0x2FC | 191 | 0 | 0 | - | unused (0) |
| +0x300 | 192 | 0 | 0 | - | unused (0) |
| +0x304 | 193 | 0 | 0 | - | unused (0) |
| +0x308 | 194 | 800 | 800 | - | value |
| +0x30C | 195 | 0 | 0 | - | unused (0) |
| +0x310 | 196 | 0 | 0 | - | unused (0) |
| +0x314 | 197 | 9 | 9 | - | value |
| +0x318 | 198 | 0.9 | 0.9 | - | value |
| +0x31C | 199 | 40 | 40 | - | value |
| +0x320 | 200 | 0 | 0 | - | unused (0) |
| +0x324 | 201 | 0 | 0 | - | unused (0) |
| +0x328 | 202 | 0 | 0 | - | unused (0) |
| +0x32C | 203 | 0 | 0 | - | unused (0) |
| +0x330 | 204 | 0 | 0 | - | unused (0) |
| +0x334 | 205 | 0 | 0 | - | unused (0) |
| +0x338 | 206 | 9 | 9 | - | value |
| +0x33C | 207 | 0.9 | 0.9 | - | value |
| +0x340 | 208 | 40 | 40 | - | value |
| +0x344 | 209 | 0 | 0 | - | unused (0) |
| +0x348 | 210 | 0 | 0 | - | unused (0) |
| +0x34C | 211 | 0 | 0 | - | unused (0) |
| +0x350 | 212 | 0 | 0 | - | unused (0) |
| +0x354 | 213 | 0 | 0 | - | unused (0) |
| +0x358 | 214 | 100 | 100 | - | value |
| +0x35C | 215 | 0 | 0 | - | unused (0) |
| +0x360 | 216 | 0 | 0 | - | unused (0) |
| +0x364 | 217 | 0 | 0 | - | unused (0) |
| +0x368 | 218 | 6.027e-39 | 6.027e-39 | - | value |
| +0x36C | 219 | 5.946e-39 | 5.946e-39 | - | value |
| +0x370 | 220 | 6.018e-39 | 6.018e-39 | - | value |
| +0x374 | 221 | 0 | 0 | - | unused (0) |
| +0x378 | 222 | 0 | 0 | - | unused (0) |
| +0x37C | 223 | 0 | 0 | - | unused (0) |
| +0x380 | 224 | 6.271e-39 | 6.271e-39 | - | value |
| +0x384 | 225 | 0 | 0 | - | unused (0) |
| +0x388 | 226 | 0 | 0 | - | unused (0) |
| +0x38C | 227 | 2.279e+23 | 2.279e+23 | - | value |
| +0x390 | 228 | 5.822e-39 | 5.822e-39 | - | value |
| +0x394 | 229 | 6.073e-39 | 6.073e-39 | - | value |
| +0x398 | 230 | 0 | 0 | - | unused (0) |
| +0x39C | 231 | 0 | 0 | - | unused (0) |
| +0x3A0 | 232 | 0 | 0 | - | unused (0) |
| +0x3A4 | 233 | 0 | 0 | - | unused (0) |
| +0x3A8 | 234 | 0 | 0 | - | unused (0) |
| +0x3AC | 235 | 0 | 0 | - | unused (0) |
| +0x3B0 | 236 | 2.279e+23 | 2.279e+23 | - | value |
| +0x3B4 | 237 | 5.822e-39 | 5.822e-39 | - | value |
| +0x3B8 | 238 | 6.073e-39 | 6.073e-39 | - | value |
| +0x3BC | 239 | 0 | 0 | - | unused (0) |
| +0x3C0 | 240 | 0 | 0 | - | unused (0) |
| +0x3C4 | 241 | 0 | 0 | - | unused (0) |
| +0x3C8 | 242 | 0 | 0 | - | unused (0) |
| +0x3CC | 243 | 0 | 0 | - | unused (0) |
| +0x3D0 | 244 | 6.133e-39 | 6.133e-39 | - | value |
| +0x3D4 | 245 | 0 | 0 | - | unused (0) |
| +0x3D8 | 246 | 0 | 0 | - | unused (0) |
| +0x3DC | 247 | 0 | 0 | - | unused (0) |
| +0x3E0 | 248 | 2.354e-41 | 2.354e-41 | - | value |
| +0x3E4 | 249 | 2.323e-41 | 2.323e-41 | - | value |
| +0x3E8 | 250 | 2.351e-41 | 2.351e-41 | - | value |
| +0x3EC | 251 | 0 | 0 | - | unused (0) |
| +0x3F0 | 252 | 0 | 0 | - | unused (0) |
| +0x3F4 | 253 | 0 | 0 | - | unused (0) |
| +0x3F8 | 254 | 2.449e-41 | 2.449e-41 | - | value |
| +0x3FC | 255 | 0 | 0 | - | unused (0) |
| +0x400 | 256 | 0 | 0 | - | unused (0) |
| +0x404 | 257 | 2.718e+23 | 2.718e+23 | - | value |
| +0x408 | 258 | 2.274e-41 | 2.274e-41 | - | value |
| +0x40C | 259 | 2.372e-41 | 2.372e-41 | - | value |
| +0x410 | 260 | 0 | 0 | - | unused (0) |
| +0x414 | 261 | 0 | 0 | - | unused (0) |
| +0x418 | 262 | 0 | 0 | - | unused (0) |
| +0x41C | 263 | 0 | 0 | - | unused (0) |
| +0x420 | 264 | 0 | 0 | - | unused (0) |
| +0x424 | 265 | 0 | 0 | - | unused (0) |
| +0x428 | 266 | 2.718e+23 | 2.718e+23 | - | value |
| +0x42C | 267 | 2.274e-41 | 2.274e-41 | - | value |
| +0x430 | 268 | 2.372e-41 | 2.372e-41 | - | value |
| +0x434 | 269 | 0 | 0 | - | unused (0) |
| +0x438 | 270 | 0 | 0 | - | unused (0) |
| +0x43C | 271 | 0 | 0 | - | unused (0) |
| +0x440 | 272 | 0 | 0 | - | unused (0) |
| +0x444 | 273 | 0 | 0 | - | unused (0) |
| +0x448 | 274 | 2.396e-41 | 2.396e-41 | - | value |
| +0x44C | 275 | 0 | 0 | - | unused (0) |
| +0x450 | 276 | 0 | 0 | - | unused (0) |
| +0x454 | 277 | -1.084e-19 | -1.084e-19 | - | value |
| +0x458 | 278 | -2.00002 | -2.00002 | - | value |
| +0x45C | 279 | -3.852e-34 | -3.852e-34 | - | value |
| +0x460 | 280 | 9.108e-44 | 9.108e-44 | - | value |
| +0x464 | 281 | 0 | 0 | - | unused (0) |
| +0x468 | 282 | 0 | 0 | - | unused (0) |
| +0x46C | 283 | 131072 | 131072 | - | value |
| +0x470 | 284 | 9.529e-44 | 9.529e-44 | - | value |
| +0x474 | 285 | 0 | 0 | - | unused (0) |
| +0x478 | 286 | 2.524e-29 | 2.524e-29 | - | value |
| +0x47C | 287 | 2.720e+23 | 2.720e+23 | - | value |
| +0x480 | 288 | 1.084e-19 | 1.084e-19 | - | value |
| +0x484 | 289 | 9.249e-44 | 9.249e-44 | - | value |
| +0x488 | 290 | 0 | 0 | - | unused (0) |
| +0x48C | 291 | 0 | 0 | - | unused (0) |
| +0x490 | 292 | 0 | 0 | - | unused (0) |
| +0x494 | 293 | 0 | 0 | - | unused (0) |
| +0x498 | 294 | 0 | 0 | - | unused (0) |
| +0x49C | 295 | 2.524e-29 | 2.524e-29 | - | value |
| +0x4A0 | 296 | 2.720e+23 | 2.720e+23 | - | value |
| +0x4A4 | 297 | 1.084e-19 | 1.084e-19 | - | value |
| +0x4A8 | 298 | 9.249e-44 | 9.249e-44 | - | value |
| +0x4AC | 299 | 0 | 0 | - | unused (0) |
| +0x4B0 | 300 | 0 | 0 | - | unused (0) |
| +0x4B4 | 301 | 0 | 0 | - | unused (0) |
| +0x4B8 | 302 | 0 | 0 | - | unused (0) |
| +0x4BC | 303 | -131072 | -131072 | - | value |
| +0x4C0 | 304 | 9.249e-44 | 9.249e-44 | - | value |
| +0x4C4 | 305 | 0 | 0 | - | unused (0) |
| +0x4C8 | 306 | 0 | 0 | - | unused (0) |
| +0x4CC | 307 | 20 | 20 | - | value |
| +0x4D0 | 308 | 6 | 6 | - | value |
| +0x4D4 | 309 | 17 | 17 | - | value |
| +0x4D8 | 310 | 0 | 0 | - | unused (0) |
| +0x4DC | 311 | 0 | 0 | - | unused (0) |
| +0x4E0 | 312 | 0 | 0 | - | unused (0) |
| +0x4E4 | 313 | 800 | 800 | - | value |
| +0x4E8 | 314 | 0 | 0 | - | unused (0) |
| +0x4EC | 315 | 0 | 0 | - | unused (0) |
| +0x4F0 | 316 | 9 | 9 | - | value |
| +0x4F4 | 317 | 0.9 | 0.9 | - | value |
| +0x4F8 | 318 | 40 | 40 | - | value |
| +0x4FC | 319 | 0 | 0 | - | unused (0) |
| +0x500 | 320 | 0 | 0 | - | unused (0) |
| +0x504 | 321 | 0 | 0 | - | unused (0) |
| +0x508 | 322 | 0 | 0 | - | unused (0) |
| +0x50C | 323 | 0 | 0 | - | unused (0) |
| +0x510 | 324 | 0 | 0 | - | unused (0) |
| +0x514 | 325 | 9 | 9 | - | value |
| +0x518 | 326 | 0.9 | 0.9 | - | value |
| +0x51C | 327 | 40 | 40 | - | value |
| +0x520 | 328 | 0 | 0 | - | unused (0) |
| +0x524 | 329 | 0 | 0 | - | unused (0) |
| +0x528 | 330 | 0 | 0 | - | unused (0) |
| +0x52C | 331 | 0 | 0 | - | unused (0) |
| +0x530 | 332 | 0 | 0 | - | unused (0) |
| +0x534 | 333 | 100 | 100 | - | value |
| +0x538 | 334 | 0 | 0 | - | unused (0) |
| +0x53C | 335 | 0 | 0 | - | unused (0) |
| +0x540 | 336 | 0 | 0 | - | unused (0) |
| +0x544 | 337 | 6.027e-39 | 6.027e-39 | - | value |
| +0x548 | 338 | 5.946e-39 | 5.946e-39 | - | value |
| +0x54C | 339 | 6.018e-39 | 6.018e-39 | - | value |
| +0x550 | 340 | 0 | 0 | - | unused (0) |
| +0x554 | 341 | 0 | 0 | - | unused (0) |
| +0x558 | 342 | 0 | 0 | - | unused (0) |
| +0x55C | 343 | 6.271e-39 | 6.271e-39 | - | value |
| +0x560 | 344 | 0 | 0 | - | unused (0) |
| +0x564 | 345 | 0 | 0 | - | unused (0) |
| +0x568 | 346 | 2.279e+23 | 2.279e+23 | - | value |
| +0x56C | 347 | 5.822e-39 | 5.822e-39 | - | value |
| +0x570 | 348 | 6.073e-39 | 6.073e-39 | - | value |
| +0x574 | 349 | 0 | 0 | - | unused (0) |
| +0x578 | 350 | 0 | 0 | - | unused (0) |
| +0x57C | 351 | 0 | 0 | - | unused (0) |
| +0x580 | 352 | 0 | 0 | - | unused (0) |
| +0x584 | 353 | 0 | 0 | - | unused (0) |
| +0x588 | 354 | 0 | 0 | - | unused (0) |
| +0x58C | 355 | 2.279e+23 | 2.279e+23 | - | value |
| +0x590 | 356 | 5.822e-39 | 5.822e-39 | - | value |
| +0x594 | 357 | 6.073e-39 | 6.073e-39 | - | value |
| +0x598 | 358 | 0 | 0 | - | unused (0) |
| +0x59C | 359 | 0 | 0 | - | unused (0) |
| +0x5A0 | 360 | 0 | 0 | - | unused (0) |
| +0x5A4 | 361 | 0 | 0 | - | unused (0) |
| +0x5A8 | 362 | 0 | 0 | - | unused (0) |
| +0x5AC | 363 | 6.133e-39 | 6.133e-39 | - | value |
| +0x5B0 | 364 | 0 | 0 | - | unused (0) |
| +0x5B4 | 365 | 0 | 0 | - | unused (0) |
| +0x5B8 | 366 | 0 | 0 | - | unused (0) |
| +0x5BC | 367 | 2.354e-41 | 2.354e-41 | - | value |
| +0x5C0 | 368 | 2.323e-41 | 2.323e-41 | - | value |
| +0x5C4 | 369 | 2.004 | 2.004 | - | value |
| +0x5C8 | 370 | -2.479e-41 | -2.479e-41 | - | value |
| +0x5CC | 371 | 2.00421 | 2.00421 | - | value |
| +0x5D0 | 372 | 2.479e-41 | 2.479e-41 | - | value |
| +0x5D4 | 373 | 2.327e-41 | 2.327e-41 | - | value |
| +0x5D8 | 374 | 2.278e-41 | 2.278e-41 | - | value |
| +0x5DC | 375 | 2.383e-41 | 2.390e-41 | **YES** | value |
| +0x5E0 | 376 | 2.314e-41 | 2.278e-41 | **YES** | value |
| +0x5E4 | 377 | -2.008e+08 | -2.008e+08 | - | value |
| +0x5E8 | 378 | 9.671e-27 | 9.671e-27 | - | value |
| +0x5EC | 379 | 0 | 0 | - | unused (0) |
| +0x5F0 | 380 | 0 | 0 | - | unused (0) |
| +0x5F4 | 381 | 9.957e-32 | 9.957e-32 | - | value |
| +0x5F8 | 382 | 0 | 0 | - | unused (0) |
| +0x5FC | 383 | -2.035e+08 | -2.035e+08 | - | value |
| +0x600 | 384 | -3.956e-23 | -3.956e-23 | - | value |
| +0x604 | 385 | 5.887e-39 | 5.887e-39 | - | value |
| +0x608 | 386 | 0 | 0 | - | unused (0) |
| +0x60C | 387 | 5.900e-39 | 5.900e-39 | - | value |
| +0x610 | 388 | 1.768e-38 | 1.768e-38 | - | value |
| +0x614 | 389 | 5.877e-39 | 5.877e-39 | - | value |
| +0x618 | 390 | 0.7 | 0.7 | - | value |
| +0x61C | 391 | 0.1 | 0.1 | - | value |
| +0x620 | 392 | 0.7 | 0.7 | - | value |
| +0x624 | 393 | 0.1 | 0.1 | - | value |
| +0x628 | 394 | 0.03 | 0.03 | - | value |
| +0x62C | 395 | 120 | 120 | - | value |
| +0x630 | 396 | 25 | 25 | - | value |
| +0x634 | 397 | 100 | 100 | - | value |
| +0x638 | 398 | 45 | 45 | - | value |
| +0x63C | 399 | 0.4 | 0.4 | - | value |
| +0x640 | 400 | 60 | 60 | - | value |
| +0x644 | 401 | 30 | 30 | - | value |
| +0x648 | 402 | 35 | 35 | - | value |
| +0x64C | 403 | 1.822e-44 | 1.822e-44 | - | value |
| +0x650 | 404 | 1.962e-44 | 1.962e-44 | - | value |
| +0x654 | 405 | 120 | 120 | - | value |
| +0x658 | 406 | 25 | 25 | - | value |
| +0x65C | 407 | 100 | 100 | - | value |
| +0x660 | 408 | 40 | 40 | - | value |
| +0x664 | 409 | 120 | 120 | - | value |
| +0x668 | 410 | 20 | 20 | - | value |
| +0x66C | 411 | 100 | 100 | - | value |
| +0x670 | 412 | 30 | 30 | - | value |
| +0x674 | 413 | 10 | 10 | - | value |
| +0x678 | 414 | 60 | 60 | - | value |
| +0x67C | 415 | 110.526 | 110.526 | - | value |
| +0x680 | 416 | 18.4211 | 18.4211 | - | value |
| +0x684 | 417 | 100 | 100 | - | value |
| +0x688 | 418 | 65 | 65 | - | value |
| +0x68C | 419 | 70 | 70 | - | value |
| +0x690 | 420 | 10 | 10 | - | value |
| +0x694 | 421 | 20 | 20 | - | value |
| +0x698 | 422 | 30 | 30 | - | value |
| +0x69C | 423 | 40 | 40 | - | value |
| +0x6A0 | 424 | 35 | 35 | - | value |
| +0x6A4 | 425 | 70 | 70 | - | value |
| +0x6A8 | 426 | 10 | 10 | - | value |
| +0x6AC | 427 | 100 | 100 | - | value |
| +0x6B0 | 428 | 15 | 15 | - | value |
| +0x6B4 | 429 | 25 | 25 | - | value |
| +0x6B8 | 430 | 15 | 15 | - | value |
| +0x6BC | 431 | 15 | 15 | - | value |
| +0x6C0 | 432 | 10 | 10 | - | value |
| +0x6C4 | 433 | 1 | 1 | - | value |
| +0x6C8 | 434 | 100 | 100 | - | value |
| +0x6CC | 435 | 0 | 0 | - | unused (0) |
| +0x6D0 | 436 | 1 | 1 | - | value |
| +0x6D4 | 437 | 1 | 1 | - | value |
| +0x6D8 | 438 | 1 | 1 | - | value |
| +0x6DC | 439 | 80 | 80 | - | value |
| +0x6E0 | 440 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6E4 | 441 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6E8 | 442 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6EC | 443 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6F0 | 444 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6F4 | 445 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6F8 | 446 | -4.316e+08 | -4.316e+08 | - | value |
| +0x6FC | 447 | -4.316e+08 | -4.316e+08 | - | value |
| +0x700 | 448 | 50 | 50 | - | value |
| +0x704 | 449 | 10 | 10 | - | value |
| +0x708 | 450 | 100 | 100 | - | value |
| +0x70C | 451 | 70 | 70 | - | value |
| +0x710 | 452 | 20 | 20 | - | value |
| +0x714 | 453 | 2 | 2 | - | value |
| +0x718 | 454 | 2 | 2 | - | value |
| +0x71C | 455 | 2 | 2 | - | value |
| +0x720 | 456 | 50 | 50 | - | value |
| +0x724 | 457 | 10 | 10 | - | value |
| +0x728 | 458 | 100 | 100 | - | value |
| +0x72C | 459 | 0 | 0 | - | unused (0) |
| +0x730 | 460 | 5 | 5 | - | value |
| +0x734 | 461 | 5 | 5 | - | value |
| +0x738 | 462 | 5 | 5 | - | value |
| +0x73C | 463 | 50 | 50 | - | value |
| +0x740 | 464 | 10 | 10 | - | value |
| +0x744 | 465 | 100 | 100 | - | value |
| +0x748 | 466 | 20 | 20 | - | value |
| +0x74C | 467 | 5 | 5 | - | value |
| +0x750 | 468 | 5 | 5 | - | value |
| +0x754 | 469 | 5 | 5 | - | value |
| +0x758 | 470 | 50 | 50 | - | value |
| +0x75C | 471 | 10 | 10 | - | value |
| +0x760 | 472 | 100 | 100 | - | value |
| +0x764 | 473 | 20 | 20 | - | value |
| +0x768 | 474 | 10 | 10 | - | value |
| +0x76C | 475 | 5 | 5 | - | value |
| +0x770 | 476 | 10 | 10 | - | value |
| +0x774 | 477 | 50 | 50 | - | value |
| +0x778 | 478 | 10 | 10 | - | value |
| +0x77C | 479 | 100 | 100 | - | value |
| +0x780 | 480 | 20 | 20 | - | value |
| +0x784 | 481 | 10 | 10 | - | value |
| +0x788 | 482 | 5 | 5 | - | value |
| +0x78C | 483 | 5 | 5 | - | value |
| +0x790 | 484 | 50 | 50 | - | value |
| +0x794 | 485 | 10 | 10 | - | value |
| +0x798 | 486 | 100 | 100 | - | value |
| +0x79C | 487 | 20 | 20 | - | value |
| +0x7A0 | 488 | 10 | 10 | - | value |
| +0x7A4 | 489 | 5 | 5 | - | value |
| +0x7A8 | 490 | 5 | 5 | - | value |
| +0x7AC | 491 | 9 | 9 | - | value |
| +0x7B0 | 492 | 27 | 27 | - | value |
| +0x7B4 | 493 | 5 | 5 | - | value |
| +0x7B8 | 494 | 0 | 0 | - | unused (0) |
| +0x7BC | 495 | 20 | 20 | - | value |
| +0x7C0 | 496 | 5.5 | 5.5 | - | value |
| +0x7C4 | 497 | 0.9 | 1.4 | **YES** | value |

## 4c. Sound / string records (0x7C9-0x1121)

Each sound is a **0x10-byte filename slot followed by about 8 float params**, whose meaning is
open (§9). Two clusters: **template/default** (0x7C9-0xC5F), then **car-specific**
(0xC63-0x1121, the set RHAM rows 75-100 expose). Only plausible params (`|x|` between 1e-3 and 1e6)
are printed; runs of sentinel/zero fill collapse into a single `padding` row.

| off | kind | value | boost | note |
|---|---|---|---|---|
| +0x7C9 | string | `engine_on.wav` | - | engine idle/run loop |
| +0x7D9 | string | `engine_off.wav` | - | engine stop |
| - | padding | (4 dwords sentinel/zero) | - | fill |
| +0x7F9 | string | `interier.wav` | - | interior sound |
| +0x809 | param | 0.55 / 0.55 | - | sound param (unknown) |
| +0x80D | param | 1.9 / 1.9 | - | sound param (unknown) |
| +0x811 | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0x815 | param | 100 / 100 | - | sound param (unknown) |
| +0x819 | param | 5 / 5 | - | sound param (unknown) |
| +0x81D | param | 100 / 100 | - | sound param (unknown) |
| +0x821 | param | 1 / 1 | - | sound param (unknown) |
| +0x825 | param | 0.2 / 0.2 | - | sound param (unknown) |
| - | padding | (50 dwords sentinel/zero) | - | fill |
| +0x8EE | string | `interier_X.wav` | - | interior sound |
| +0x8FE | param | 0.7 / 0.7 | - | sound param (unknown) |
| +0x902 | param | 1.3 / 1.3 | - | sound param (unknown) |
| +0x906 | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0x90A | param | 100 / 100 | - | sound param (unknown) |
| +0x90E | param | 5 / 5 | - | sound param (unknown) |
| +0x912 | param | 100 / 100 | - | sound param (unknown) |
| +0x916 | param | 1 / 1 | - | sound param (unknown) |
| +0x91A | param | 0.2 / 0.2 | - | sound param (unknown) |
| - | padding | (50 dwords sentinel/zero) | - | fill |
| +0x9E3 | string | `interier.wav` | - | interior sound |
| +0x9F3 | param | 0.55 / 0.55 | - | sound param (unknown) |
| +0x9F7 | param | 1.9 / 1.9 | - | sound param (unknown) |
| +0x9FB | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0x9FF | param | 100 / 100 | - | sound param (unknown) |
| +0xA03 | param | 5 / 5 | - | sound param (unknown) |
| +0xA07 | param | 100 / 100 | - | sound param (unknown) |
| +0xA0B | param | 1 / 1 | - | sound param (unknown) |
| +0xA0F | param | 0.2 / 0.2 | - | sound param (unknown) |
| - | padding | (48 dwords sentinel/zero) | - | fill |
| +0xAD3 | string | `interier_0.wav` | - | interior sound |
| +0xAE3 | param | 80 / 80 | - | sound param (unknown) |
| +0xAE7 | param | 1 / 1 | - | sound param (unknown) |
| +0xAEB | string | `interier_A.wav` | - | interior sound |
| +0xAFB | param | 0.9 / 0.9 | - | sound param (unknown) |
| +0xAFF | param | 1.1 / 1.1 | - | sound param (unknown) |
| +0xB03 | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0xB07 | param | 47 / 47 | - | sound param (unknown) |
| +0xB0B | param | 1 / 1 | - | sound param (unknown) |
| +0xB0F | string | `horn1.wav` | - | horn |
| - | padding | (4 dwords sentinel/zero) | - | fill |
| +0xB2F | string | `hand2.wav` | - | handbrake |
| +0xB3F | string | `gear1.wav` | - | gear change |
| +0xB4F | string | `gear1.wav` | - | gear change |
| +0xB5F | string | `slip1.wav` | - | tyre slip |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xB70 | string | `rzdy_loop.wav` | - | brakes loop (stored literally as `rzdy_loop.wav`; cf. Czech *brzdy* = brakes) |
| +0xB80 | string | `hit3.wav` | - | impact |
| - | padding | (4 dwords sentinel/zero) | - | fill |
| +0xB9F | string | `carglass.wav` | - | glass break |
| +0xBAF | string | `crashA1in.wav` | - | crash (interior) |
| +0xBBF | string | `crashA2in.wav` | - | crash (interior) |
| +0xBCF | param | 1.5 / 1.5 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xBD4 | string | `crashK1in.wav` | - | crash (interior) |
| +0xBE4 | string | `crashK2in.wav` | - | crash (interior) |
| +0xBF4 | param | 2.5 / 2.5 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xBF9 | string | `crashB1in.wav` | - | crash (interior) |
| +0xC09 | string | `crashB2in.wav` | - | crash (interior) |
| +0xC19 | param | 9 / 9 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xC1E | string | `crashC1in.wav` | - | crash (interior) |
| +0xC2E | string | `crashC2in.wav` | - | crash (interior) |
| +0xC3E | param | 35 / 35 | - | sound param (unknown) |
| - | padding | (9 dwords sentinel/zero) | - | fill |
| +0xC63 | string | `black_on.wav` | - | sound (car-specific) |
| +0xC73 | string | `black_off.wav` | - | sound (car-specific) |
| +0xC83 | string | `startbad1.wav` | - | engine bad-start |
| +0xC93 | string | `black_NPC.wav` | - | engine (NPC/external) |
| +0xCA3 | param | 0.65 / 0.65 | - | sound param (unknown) |
| +0xCA7 | param | 1.35 / 1.35 | - | sound param (unknown) |
| +0xCAB | param | 1 / 1 | - | sound param (unknown) |
| +0xCAF | param | 1 / 1 | - | sound param (unknown) |
| +0xCB3 | string | `black_1.wav` | - | sound (car-specific) |
| +0xCC3 | param | 0.5 / 0.5 | - | sound param (unknown) |
| +0xCC7 | param | 1.5 / 1.35 | **YES** | sound param (unknown) |
| +0xCCB | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0xCCF | param | 100 / 100 | - | sound param (unknown) |
| +0xCD3 | param | 6 / 6 | - | sound param (unknown) |
| +0xCD7 | param | 100 / 100 | - | sound param (unknown) |
| +0xCDB | param | 0.8 / 0.8 | - | sound param (unknown) |
| +0xCDF | param | 0.3 / 0.3 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xCE4 | string | `buick_2.wav` | - | sound (car-specific) |
| +0xCF4 | param | 0.55 / 0.55 | - | sound param (unknown) |
| +0xCF8 | param | 1.3 / 1.2 | **YES** | sound param (unknown) |
| +0xCFC | param | 45 / 45 | - | sound param (unknown) |
| +0xD00 | param | 100 / 100 | - | sound param (unknown) |
| +0xD04 | param | 97 / 97 | - | sound param (unknown) |
| +0xD08 | param | 100 / 100 | - | sound param (unknown) |
| +0xD0C | param | 1 / 1 | - | sound param (unknown) |
| +0xD10 | param | 0.3 / 0.3 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xD15 | string | `black_3.wav` | - | sound (car-specific) |
| +0xD25 | param | 0.6 / 0.6 | - | sound param (unknown) |
| +0xD29 | param | 1.3 / 1.2 | **YES** | sound param (unknown) |
| +0xD2D | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0xD31 | param | 100 / 100 | - | sound param (unknown) |
| +0xD35 | param | 5 / 5 | - | sound param (unknown) |
| +0xD39 | param | 100 / 100 | - | sound param (unknown) |
| +0xD3D | param | 1 / 1 | - | sound param (unknown) |
| +0xD41 | param | 0.2 / 0.2 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xD46 | string | `s1.wav` | - | sound (car-specific) |
| +0xD56 | param | 0.5 / 0.5 | - | sound param (unknown) |
| +0xD5A | param | 2 / 2 | - | sound param (unknown) |
| +0xD5E | param | 10 / 10 | - | sound param (unknown) |
| +0xD62 | param | 100 / 100 | - | sound param (unknown) |
| +0xD66 | param | 18 / 18 | - | sound param (unknown) |
| +0xD6A | param | 46 / 46 | - | sound param (unknown) |
| +0xD6E | param | 0.35 / 0.35 | - | sound param (unknown) |
| - | padding | (14 dwords sentinel/zero) | - | fill |
| +0xDA8 | string | `cheve_X.wav` | - | sound (car-specific) |
| +0xDB8 | param | 0.6 / 0.6 | - | sound param (unknown) |
| +0xDBC | param | 1.6 / 1.5 | **YES** | sound param (unknown) |
| +0xDC0 | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0xDC4 | param | 100 / 100 | - | sound param (unknown) |
| +0xDC8 | param | 24 / 24 | - | sound param (unknown) |
| +0xDCC | param | 100 / 100 | - | sound param (unknown) |
| +0xDD0 | param | 1 / 1 | - | sound param (unknown) |
| +0xDD4 | param | 0.3 / 0.3 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0xDD9 | string | `s1.wav` | - | sound (car-specific) |
| +0xDE9 | param | 0.7 / 0.7 | - | sound param (unknown) |
| +0xDED | param | 1.8 / 1.8 | - | sound param (unknown) |
| +0xDF1 | param | 10 / 10 | - | sound param (unknown) |
| +0xDF5 | param | 70 / 70 | - | sound param (unknown) |
| +0xDF9 | param | 30 / 30 | - | sound param (unknown) |
| +0xDFD | param | 40 / 40 | - | sound param (unknown) |
| +0xE01 | param | 0.42 / 0.42 | - | sound param (unknown) |
| - | padding | (38 dwords sentinel/zero) | - | fill |
| +0xE9D | string | `black_1.wav` | - | sound (car-specific) |
| +0xEAD | param | 0.5 / 0.5 | - | sound param (unknown) |
| +0xEB1 | param | 1.6 / 1.6 | - | sound param (unknown) |
| +0xEB5 | param | 0.1 / 0.1 | - | sound param (unknown) |
| +0xEB9 | param | 100 / 100 | - | sound param (unknown) |
| +0xEBD | param | 5 / 5 | - | sound param (unknown) |
| +0xEC1 | param | 100 / 100 | - | sound param (unknown) |
| +0xEC5 | param | 0.98 / 0.98 | - | sound param (unknown) |
| +0xEC9 | param | 0.7 / 0.7 | - | sound param (unknown) |
| +0xECD | string | `revers3.wav` | - | sound (car-specific) |
| +0xEDD | param | 0.7 / 0.7 | - | sound param (unknown) |
| +0xEE1 | param | 1.8 / 1.8 | - | sound param (unknown) |
| +0xEE5 | param | 30 / 30 | - | sound param (unknown) |
| +0xEE9 | param | 100 / 100 | - | sound param (unknown) |
| +0xEED | param | 80 / 80 | - | sound param (unknown) |
| +0xEF1 | param | 100 / 100 | - | sound param (unknown) |
| +0xEF5 | param | 1 / 1 | - | sound param (unknown) |
| +0xEF9 | param | 0.6 / 0.7 | **YES** | sound param (unknown) |
| +0xEFD | string | `buick_2.wav` | - | sound (car-specific) |
| +0xF0D | param | 0.6 / 0.6 | - | sound param (unknown) |
| +0xF11 | param | 1.82 / 1.82 | - | sound param (unknown) |
| +0xF15 | param | 35 / 35 | - | sound param (unknown) |
| +0xF19 | param | 100 / 100 | - | sound param (unknown) |
| +0xF1D | param | 70 / 70 | - | sound param (unknown) |
| +0xF21 | param | 100 / 100 | - | sound param (unknown) |
| +0xF25 | param | 1 / 1 | - | sound param (unknown) |
| +0xF29 | param | 0.4 / 0.4 | - | sound param (unknown) |
| - | padding | (24 dwords sentinel/zero) | - | fill |
| +0xF8D | string | `black_0.wav` | - | sound (car-specific) |
| +0xF9D | param | 35 / 35 | - | sound param (unknown) |
| +0xFA1 | param | 1 / 1 | - | sound param (unknown) |
| - | padding | (9 dwords sentinel/zero) | - | fill |
| +0xFC9 | string | `horn5.wav` | - | horn |
| - | padding | (4 dwords sentinel/zero) | - | fill |
| +0xFE9 | string | `hand2.wav` | - | handbrake |
| +0xFF9 | string | `gear2.wav` | **YES** | gear change |
| +0x1009 | string | `gear1.wav` | **YES** | gear change |
| +0x1019 | string | `slip1.wav` | - | tyre slip |
| +0x1029 | string | `break.wav` | **YES** | brakes |
| +0x1039 | string | `cep.wav` | **YES** | sound (car-specific) |
| +0x1049 | string | `pneu1.wav` | - | tyre |
| +0x1059 | string | `carglass.wav` | - | glass break |
| +0x1069 | string | `crashA1.wav` | - | crash (exterior) |
| +0x1079 | string | `crashA2.wav` | - | crash (exterior) |
| +0x1089 | param | 1.5 / 1.5 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0x108E | string | `crashK1.wav` | - | crash (exterior) |
| +0x109E | string | `crashK2.wav` | - | crash (exterior) |
| +0x10AE | param | 2.5 / 2.5 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0x10B3 | string | `crashB1.wav` | - | crash (exterior) |
| +0x10C3 | string | `crashB2.wav` | - | crash (exterior) |
| +0x10D3 | param | 9 / 9 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0x10D8 | string | `crashC1.wav` | - | crash (exterior) |
| +0x10E8 | string | `crashC2.wav` | - | crash (exterior) |
| +0x10F8 | param | 35 / 35 | - | sound param (unknown) |
| - | padding | (1 dword sentinel/zero) | - | fill |
| +0x10FD | string | `door04op.wav` | - | door open |
| +0x110D | string | `door04cl.wav` | - | door close |
| - | padding | (1 dword sentinel/zero) | - | fill |

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

More differing fields live in the geometry block (§4b — `+0x138`, `+0x144`-`+0x150`,
`+0x190`-`+0x1C8` among others) and in a few sound params (§4c). Every one of them carries
**YES** in the tables above.

## 6. Boostable cars (`level_count == 2`) — all 14

`blackha`, `buikFor`, `cad_ford`, `forVfor`, `bugatti`, `hudfor`, `arrow`, `polCad`, `poli`,
`polimFor`, `polimTud`, `black`, `buigang`, `chuligani`.

Body variants are separate records with independent level counts: `hudfor` (Terraplane Fordor) has
2 levels, `hudtu` (Tudor) has 1.

## 7. `vehicles.dat` — the field schema

3856 B. Header `u32 total (= 0xF0C)`, one further `u32`, then `u32 field_count (= 83)`, followed by
the per-field descriptors. Their byte-offset column steps by 4 across the CARINIT floats and each
carries a type code — `5` = float, `0x11` = other. The file is a layout/type table only; it holds no
per-car data.

## 8. Cross-references

- `police-system/artifacts/boosting.md` — how the pursuit system activates level 1.
- `structs/pursuit-event-manager.md` — the `C_sledovacka` chase system.
- `structs/car.md` and `structs/vehicle.md` — the runtime classes the block is loaded into.
- Engine entry points: `car_get_model_level_count` (`0x00465a90`), `car_set_model_level`
  (`0x00465ad0`), `vehicle_apply_carinit_params` (`0x005123d0`).

## 9. Open items

| item | state |
|---|---|
| the seven internal fields at `0x0EC`-`0x104` | byte-verified constants (45, 25, 10, 50, 90, 16, 50 for `blackha`), no consumer traced |
| RHAM rows 71-74 | Fuel consumption / Rear weigh / Rear suspension height / RWD & Roof flags fall inside `0x134`-`0x7C8` but are not pinned to individual offsets |
| the geometry block `0x134`-`0x7C8` | dumped field by field; the semantics of the individual entries are not established |
| the ~8 floats after each sound filename slot | read as volume / pitch / ranges / priority from their value shapes; no engine reader has been traced to confirm the order |
| `+0x000` "Weight" and `+0x130` "Forward control y/n" | RHAM's labels; the byte patterns are recorded, the meanings are inferred from the label alone |
