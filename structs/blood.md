# `C_blood` / `C_blood_manager` — the blood-drop and decal system

**Created** 2026-08-30 15:50 +0200 · **Modified** 2026-09-17 13:43 +0200

PC prefixes **`blood_`** and **`blood_manager_`**.

| | |
|---|---|
| PS2 classes | `C_blood` (tag `7C_blood`), `C_blood_manager` (tag `15C_blood_manager`) |
| Manager location | **embedded in `C_game` at `+0x2d90`** — byte-proven on 12 of 12 callsites |
| PC functions | 6, all with CONFIRMED PS2 twins (§6) |
| PC translation unit | `[0x005adf80, 0x005ae480)` — 8 starts including the CRT guard pair |

## 1. What it is — and what it is not

A `C_blood` is an **invisible ballistic marker**. The record is 0x24 bytes — a position, a
velocity, a size scale, a decal texture index and a flag — and it holds **no `I3D_frame`
and no model**. Nothing draws it. Its only visible output is the **decal it places on
whatever it lands on**.

That is why there is no visible animation of blood falling: the drop is launched from the
wound, falls under gravity, hits static geometry usually within a frame or two, stamps a
decal and dies.

Each spawn call emits **one** drop from the body's current position; nothing in the code lays
a path of drops.

In-game observation by the user:
> * blood appears for a moment around the point where the enemy was hit — that is a
>   separate particle effect, **not** this system;
> * a blood stain then appears **on the ground under the enemy's current position**, using
>   `2blood3.bmp`;
> * **there is no trail** of stains behind a wounded NPC that walks;
> * there is no visible falling animation.

## 2. Object layouts (PC)

```c
typedef struct C_blood {          // 0x24 bytes, heap, owned by the manager's vector
    /* +0x00 */ float    pos[3];      // [H] current position, integrated by blood_tick
    /* +0x0c */ float    vel[3];      // [H] velocity; vel.y takes gravity every tick
    /* +0x18 */ float    size;        // [H] decal size scale
    /* +0x1c */ int      texture;     // [H] index into C_game+0x29bc, the 10 decal textures
    /* +0x20 */ uint8_t  flag;        // [M] passed straight through to the decal placer
} C_blood;

typedef struct C_blood_manager {  // embedded at C_game + 0x2d90
    /* +0x00 */ uint8_t  alloc;       // [H] STL allocator byte
    /* +0x04 */ C_blood **begin;      // [H] vector<C_blood*> — the live drops
    /* +0x08 */ C_blood **end;        // [H]
    /* +0x0c */ C_blood **cap;        // [H]
} C_blood_manager;                    // 0x10 bytes
```

The `C_blood` constructor has no address of its own: `blood_manager_add_blood` calls
`folded_return_this` (`0x005575f0`, body `mov eax,ecx; ret`) on the `operator_new(0x24)` result.
That body is a three-owner fold — 9 callsites take the `G_LoadSaveClass` singleton, 1 is this
`C_blood` constructor, and 1 takes a stack parameter in `recorder_sync_clock_from_system`.

## 3. The physics

`blood_tick(drop, dt_seconds)`:

1. `vel.y -= dt * 10.0` — gravity, 10 units/s².
2. **If `vel.y > 20.0`, return false** and the drop is destroyed.
3. Line-test `pos → pos + vel*dt` against **static** collision.
4. **Hit:** place a decal at the impact point, offset by half the surface normal, with a
   random size `0.5 + rand * drop.size * 0.3` and a random rotation `rand * 2π`, using the
   drop's texture index. Return false — the drop is done.
5. **Miss:** `pos += vel*dt`, return true.

`blood_manager_tick` runs the swap-and-pop loop over every live drop each frame from
`game_tick`.

## 4. Who spawns drops, and with which texture

`blood_manager_add_blood(pos, dir, size, texture, flag)` — 10 callsites, every one a wound,
death or bleeding path, all reaching the manager as
`mov ecx,[reg+0x24]; add ecx,0x2d90` (the `C_game` singleton plus the embed offset):

| caller | texture | flag |
|---|---|---|
| `human_hit` (×2) | 0 — `2blood.bmp` | 0 |
| `dog_hit` (×2) | 0 | 0 |
| `pedestrian_hit` | 0 | 0 |
| `human_force_death` | 0 | 0 |
| `human_update` | 7 — `2blood3.bmp` | 1 |
| `human_movement` | 7 | 1 |
| `dog_enter_state` | 7 | 1 |
| `dog_update` | 7 | 1 |

Texture **0** is the hit/death spray; texture **7** is the stain that appears under a
wounded body — which is what the in-game observation reports.

## 5. The ten decal textures — `C_game + 0x29bc`

Loaded in `game_init`, released in `game_done`. The decal pool itself is **100 records of
0x58 bytes** at `C_game + 0x75c .. +0x29bc`, with the ring cursor at `+0x29e4`.

| index | asset | what it is |
|---|---|---|
| 0 | `2blood.bmp` | blood |
| 1 | `2flek.bmp` | Czech *flek* — a stain |
| 2 | `2dirbet.bmp` | dirt on *beton* (concrete) |
| 3 | `2dirdre.bmp` | dirt on *dřevo* (wood) |
| 4 | `2dirple.bmp` | dirt on *plech* (sheet metal) |
| 5 | `2dirsklo.bmp` | dirt on *sklo* (glass) |
| 6 | `2bfootprint.bmp` | a bloody footprint |
| 7 | `2blood3.bmp` | blood, variant 3 |
| 8 | `2dogfoot.bmp` | a dog footprint |
| 9 | `skrob.tga` | a scrape/scratch |

Indices 2–5 are per-surface bullet-impact marks, i.e. the same 100-slot decal pool serves
gunfire as well as blood. Only 0 and 7 are reachable through `C_blood`.

## 6. PC vs PS2

| PC VA | PC name | PS2 twin | PS2 size | PC size | status |
|---|---|---|---|---|---|
| 0x005adfb0 | `blood_tick` | `Tick__7C_bloodFf` | 472 | 369 | CONFIRMED |
| 0x005ae130 | `blood_init` | `Init__7C_bloodFR8S_vectorR8S_vectorfib` | 32 | 71 | CONFIRMED |
| 0x005ae180 | `blood_manager_game_init` | `GameInit__15C_blood_managerFv` | 8 | 34 | CONFIRMED |
| 0x005ae1b0 | `blood_manager_game_done` | `GameDone__15C_blood_managerFv` | 96 | 77 | CONFIRMED |
| 0x005ae200 | `blood_manager_tick` | `Tick__15C_blood_managerFUi` | 244 | 150 | CONFIRMED |
| 0x005ae2a0 | `blood_manager_add_blood` | `AddBlood__15C_blood_managerFR8S_vectorR8S_vectorfib` | 196 | 473 | CONFIRMED |
| - | - | `__ct__7C_bloodFv` | 8 | - | folded on PC into `folded_return_this` (§2) |
| - | - | `__dt__7C_bloodFv` | 68 | - | inlined on PC as a bare `operator_delete_wrapper` |
| - | - | `__ct__15C_blood_managerFv` | 52 | - | not located ¹ |
| - | - | `__dt__15C_blood_managerFv` | 100 | - | not located ¹ |

¹ Must be inside `game_ctor` / `game_dtor`, because the manager is embedded (§8).

* **Record size differs:** PC 0x24 against PS2 0x30, because the PS2 pads `S_vector` to 16
  bytes. The manager differs too: the PS2 keeps a count + array, the PC a `std::vector`.
* **The `vel.y > 20.0` test** (§3 step 2): the PS2 decompiles it cleanly as `if (20.0 < fVar4)`,
  which settles Ghidra's x87 comparison output on the PC side.
* The PS2 also carries `BloodStep__5C_dogFUi` on `C_dog`, which is the caller behind the two
  `dog_*` texture-7 rows in §4.

## 7. Related

`structs/game.md` (the `+0x2d90` embed and the `+0x29bc` texture table) · `structs/dog.md`
(`BloodStep`) · `structs/human.md` · `basic/artifacts/hud_elements.md` §10 (the screen blood
overlay, a different consumer of `2blood.bmp`).

## 8. Open items

| item | state |
|---|---|
| the manager's constructor and destructor | not located; they must be inside `game_ctor` / `game_dtor` because the manager is embedded |
| `C_blood+0x20` flag byte | passed straight through to the decal placer; its effect there is not decoded |
| the 100-slot decal pool at `C_game + 0x75c` | no struct doc; record layout, ring behaviour and the bullet-impact producers (indices 2–5) are undocumented |
| bloody footprints (index 6) and dog prints (index 8) | nothing in `C_blood` reaches them, so another producer must exist; not checked in game whether they ever appear |
