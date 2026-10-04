# The lightness subsystem — `C_game`'s per-object shading, shoot lights and light cache

**Created** 2026-09-02 19:51 +0200 · **Modified** 2026-10-04 16:36 +0200

Not a class: three related **features of `C_game`**, built from one record type, one engine
class and ten PC functions in the `g_game.cpp` translation-unit tail `[0x005f1090, 0x005f79a0)`.
It gets its own document for the same reason `structs/fake_shadow.md` does — it spans three
`C_game` field bands, and it is the largest PC-vs-PS2 content difference in this half of the TU:
**the PS2 gutted all three features to empty stubs and the PC ships every one of them in full**
(§6).

## 1. Three features, one TU band

| feature | what it does | PC functions |
|---|---|---|
| **object lightness** | samples the scene's ambient light at each registered object and fades the object's shading toward it | `game_lightness_add` `_del` `_clear` `_tick` `_apply_recursive` |
| **shoot lights** | a small pool of short-lived dynamic lights for muzzle flashes, explosions and fires | `game_add_shoot_light` |
| **light cache** | collects every `I3D_light` in the scene once, then enables/disables them by camera distance | `game_use_light_cache_system` `game_light_cache_collect_recursive` `game_recompute_light_cache` |

All ten PC functions sit in `[0x005f1090, 0x005f19c0]`, in PS2 emission order, immediately
after `game_savegame_load_before` and immediately before the media (IShow) family.

## 2. Object lightness

### 2a. The record

`game_lightness_add` allocates it with `operator_new(0x24)`. Layout read directly from the
constructor and confirmed by all five consumers:

| offset | size | field | notes |
|---|---|---|---|
| `+0x00` | 4 | `C_actor *owner` | the actor the frame belongs to |
| `+0x04` | 4 | `I3D_frame *frame` | **the key** — `game_lightness_del` erases on this |
| `+0x08` | 2 | `ushort flags` | bit 0 = apply recursively AND skip the tag filter |
| `+0x0c` | 12 | `S_vector offset` | sampling offset from the frame origin |
| `+0x18` | 4 | `float current` | the value actually applied this frame; eased toward `+0x1c` |
| `+0x1c` | 4 | `float target` | the last sampled ambient value |
| `+0x20` | 4 | `int last_sample_ms` | game clock at the last sample; drives the 250 ms gate |

Both floats are seeded to `1.0f` and the timestamp to `0`.

### 2b. Where the records live

| `C_game` offset | what |
|---|---|
| `+0x154` | vector begin |
| `+0x158` | vector end |
| `+0x15c` | vector capacity |
| `+0x160` | the round-robin sampling cursor, reset when it passes the end |

### 2c. The tag whitelist

Unless bit 0 of the record flags is set, `game_lightness_add` accepts a frame only if its
visual type at `I3D_frame+0x110` is **1** and its FourCC tag at `I3D_frame+0x1d4` is one of:

`LMAP` `MMAP` `MRPH` `PRTB` `PRTE` `SNGM` `LENS` `LNDP` `SMMR` `OBJ_`

`game_lightness_apply_recursive` applies a narrower set when it walks a subtree — `MMAP`
`LMAP` `MRPH` `SNGM` `SMMR` `OBJ_` — i.e. it drops the two particle tags and the two lens
tags, which are not shaded by ambient light.

### 2d. What the tick does

`game_tick` calls `game_lightness_tick` only while the enable byte `C_game+0x164` is set, the
vector is non-empty and the detail setting is 2 or more (`structs/game.md` §2). Only the script
command `actorlightnesson` sets the byte, `game_init` clears it on every scene, and no shipped
mission script calls the command, so in the retail game the pass does not run.

`game_lightness_tick(ms)` runs two passes over the same vector:

1. **Sample one record per frame.** The cursor at `+0x160` selects it. The record is sampled
   only when *both* gates pass: more than **250 ms** since its last sample, and the frame is
   within **40 units** of the scene camera (the body tests the squared distance against
   `1600.0`). The sample itself is the `I3D_scene` virtual at `+0x68` — the engine's
   `GetLightness` — and the answer is stored as the record's `target`.
2. **Fade every record.** `current` moves toward `target` at `ms * 0.004` per millisecond
   (so a full 0→1 swing takes 250 ms, matching the re-test interval), snapping when the gap
   falls below `1e-08`. The result is applied through the frame's `SetHRLightness` virtual at
   `+0x80`, or through `game_lightness_apply_recursive` when the record's recursive flag is
   set.

**The cost is one scene light query per frame in total, not per object.** With N registered
objects each is re-sampled every N frames, which is why the round-robin cursor exists.

### 2e. Who registers and unregisters

| adds | removes |
|---|---|
| `human_game_init` | `human_game_done` |
| `dog_game_init` | `dog_death` |
| `human_change_model` | `human_change_model` |
| `human_activate` | `human_deactivate` |
| - | `human_death` |

An exact mirror except for `human_death`, which has no add-side partner because the corpse
keeps the registration its living form made.

## 3. `game_lightness_apply_recursive` is not an `I3D_frame` method

Its receiver in ECX is a **frame**, not the `C_game` singleton — an ECX census of its callsites
grades 2 SAFE / 0 UNSAFE: `mov ecx,[eax+4]` (the record's frame) plus the recursive self-call. It
still carries the `game_` prefix because it is a file-static of `g_game.cpp`, like the TU's two
other recursive frame walkers `game_invalidate_frame_recursive` and
`game_free_frame_placement_recursive`, whose ECX is a frame too.

It has no PS2 twin (§6c).

## 4. Shoot lights

### 4a. The pool

Ten records of stride `0xc` at `C_game+0x2a48`, spanning `+0x2a48`..`+0x2ac0`:

| offset in record | field |
|---|---|
| `+0x00` | `I3D_light *` — the pre-created light object |
| `+0x04` | `int lifetime_ms` — counted down every frame; `< 1` means the slot is free |
| `+0x08` | `ushort flags` — bit 0: flicker, then fade over the last 500 ms (§4c) |

### 4b. What `game_add_shoot_light` does

Scans the pool from slot 9 downward for the first record whose timer is `< 1`, then on that
light: sets the range through vtable `+0x64` with `(range * 0.5, range)`; writes the world
position into the frame at `+0x80`..`+0x88`; rewrites the frame flags
`& 0xfffffedf | 0x40000000` (the project-wide "frame is on and dirty" idiom); re-links it into
the scene sector; gives it an intensity of `rand() * 0.4 + 0.8`; and stores the caller's
lifetime and flags in the record.

| caller | range | lifetime | flags |
|---|---|---|---|
| `human_shooting` (muzzle flash) | 7.0 | 50 ms | 0 |
| `actor_shot_update` | 7.0 | 50 ms | 0 |
| `game_new_explosion` | 12.0 | 100 ms | 0 |
| `game_new_fire` | 10.0 | the fire's own lifetime | 1 |

### 4c. The per-frame tick is inlined into `game_tick`

`game_tick` walks all ten records every frame. For a slot with lifetime left it subtracts the
frame time; at zero it notifies a kind-5 parent, unlinks the light and switches it off. While the
slot lasts and flag bit 0 is set, the intensity is re-rolled to `rand() * 0.4 + 0.8` every frame
while more than 500 ms remain, and set to `lifetime * 0.002` below that, a linear fade. A slot
without the flag keeps its creation intensity until it goes out.

A displacement sweep for `0x2a48`, `0x2a4c` or `0x2ac4` does not see this loop: it starts at
`lea esi,[ebp+0x2ac0]`, the END of the pool, and reads each record at negative offsets
(`[esi-8]` the lifetime, `[esi+8]` after the step the flags).

## 5. The light cache

| `C_game` offset | what |
|---|---|
| `+0x16c` / `+0x170` / `+0x174` | the `std::vector<I3D_light *>` — begin / end / capacity |
| `+0x178` | the re-test countdown; `0x7e7` (2023 ms) when armed, `-1` when idle |

`game_use_light_cache_system` empties the vector, sets the countdown to `-1` and hands the
scene root to `game_light_cache_collect_recursive`, which walks the frame tree (`+0x124` first
child, `+0x12c` next sibling) appending every child of visual type **2** whose sub-type is
**1 or 2** — the two `I3D_light` kinds. The element type comes from the PS2 template argument
`std::vector<P9I3D_light>` (§6).

`game_recompute_light_cache` then runs the distance pass: for each cached light it compares the
squared distance to the scene camera against `(light_radius + camera_range + 15.0)²` and enables
or disables the light through its own vtable `+0x24`. The countdown is cleared on both sides of
the pass, so every call recomputes.

Both entry points are reached from `program_process_command`, i.e. they are script commands.

## 6. PC vs PS2

### 6a. Three methods are empty stubs on the PS2

Verified by reading the eight bytes of each body, not inferred from the symbol size (an 8-byte
MIPS symbol can mean several things):

| PS2 symbol | size | body |
|---|---|---|
| `LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector` | 8 | `return;` |
| `LightnessTick__6C_gameFUi` | 8 | `return;` |
| `AddShootLight__6C_gameFR8S_vectorfiUs` | 8 | `return;` |

`LightnessDel` / `LightnessClear` are real, `ShootLightTick` is real (388 B), and the three
light-cache methods are real. **Here the PS2 method count is a lower bound, not an upper one.**
A row whose PS2 side is one of the three stubs can be matched as a *symbol* but never as a
*body*, so it is graded STRUCTURAL.

### 6b. The PC↔PS2 map

| PC VA | PC name | PS2 symbol | PS2 VA | grade |
|---|---|---|---|---|
| `0x005f10d0` | `game_lightness_add` | `LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector` | `0x002ff510` | STRUCTURAL (stub) |
| `0x005f1370` | `game_lightness_del` | `LightnessDel__6C_gameFP9I3D_frame` | `0x002ff520` | CONFIRMED |
| `0x005f1400` | `game_lightness_clear` | `LightnessClear__6C_gameFv` | `0x002ff5c0` | CONFIRMED |
| `0x005f1460` | `game_lightness_tick` | `LightnessTick__6C_gameFUi` | `0x002ff620` | STRUCTURAL (stub) |
| `0x005f1750` | `game_lightness_apply_recursive` | - | - | PC-only (§6c) |
| `0x005f17c0` | `game_add_shoot_light` | `AddShootLight__6C_gameFR8S_vectorfiUs` | `0x002ff630` | STRUCTURAL (stub) |
| - | inlined into `game_tick` | `ShootLightTick__6C_gameFUi` | `0x002ff640` | inlined (§6d) |
| `0x005f1900` | `game_use_light_cache_system` | `UseLightCacheSystem__6C_gameFv` | `0x002ff7d0` | CONFIRMED |
| `0x005f1950` | `game_light_cache_collect_recursive` | `cbLight__Q26C_game19S_hlp$18015game_cpp…` | `0x002ff8c0` | CONFIRMED |
| `0x005f19c0` | `game_recompute_light_cache` | `RecomputeLightCache__6C_gameFv` | `0x002ffa50` | CONFIRMED, 2→1 fold |
| - | - | `LightCacheTick__6C_gameFUi` | `0x002ffa90` | folded into the row above (§6e) |

### 6c. Why `game_lightness_apply_recursive` has no twin

It is **not** the twin of the PS2's `SetLightnessHR__FP9I3D_frame` at `0x004251d0`, a free
function whose only callers are `I3D_driver::SetLightness` and one sibling. That address band is
`I3d_driver.cpp`, the LS3D engine, which the PS2 compiled into the ELF and the PC ships in
`ls3df.dll` — so the PC's copy of *that* function is in a DLL, and no search of `Game.exe` can
find it. The function in `Game.exe` is a separate game-side helper that exists only because the
PC implements a tick the PS2 stubbed out. Two differences confirm they are not the same
function: the PS2 gates on flag `0x800` of `I3D_frame+0x1d0` where the PC uses the FourCC
whitelist, and the PS2 recurses over the raw sibling chain where the PC uses the child-count and
`GetChild` virtuals.

### 6d. `ShootLightTick` is inlined into `game_tick` on the PC

The PS2 method is 388 bytes, and it is what identified `game_add_shoot_light`: it walks **2**
records of the identical `{I3D_light *, int lifetime, ushort flags}` stride-`0xc` layout at
`C_game+0xe0c`. The PC copy inlined into `game_tick` (§4c) matches it statement for statement —
the countdown, the parent notification and switch-off on expiry, and the bit-0 branch with
`rand() * 0.4 + 0.8` above 500 ms and `lifetime * 0.002` below — and differs only in the pool
size, **10** slots against **2**. With `AddShootLight` stubbed, the PS2 tick never finds a lit
slot.

### 6e. The factoring difference in `game_recompute_light_cache`

PS2 `RecomputeLightCache` is three statements — `+0x1c4 = -1; LightCacheTick(1); +0x1c4 = -1;`.
MSVC spliced the whole 388-byte tick between them, so the PC has no separate tick function: a
**2 → 1 fold**, 336 B against 56 + 388. The site is bannered in the decomp. The consequence is
behavioural: on the PS2 the countdown at `+0x1c4` throttles the distance pass to once every ~2 s
during normal ticking, while the only PC entry point clears the countdown on both sides of the
pass and therefore always recomputes.

## 7. Related

`structs/game.md` (the owning class) · `structs/fake_shadow.md` (the sibling `C_game` rendering
feature) · `basic/artifacts/dll.md` (why `SetLightnessHR` is not in `Game.exe`).

## 8. Open items

| item | state |
|---|---|
| `game_lightness_add`'s `kind`/`flags` parameters | the mangled PS2 signature carries a `ushort`; only bit 0 is decoded |
| object lightness in game | not tested. In the retail game no actor should darken, because the pass is gated off (§2d); with `actorlightnesson` executed, an actor in a dark alley should darken over ~0.25 s once within 40 units of the camera and stop updating beyond it |
| which missions arm the light cache | it is armed only by a script command; the missions that use it are untraced |
