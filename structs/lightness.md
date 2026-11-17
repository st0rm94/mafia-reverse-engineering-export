# The lightness subsystem — `C_game`'s per-object shading, shoot lights and light cache

**Created** 2026-09-02 19:51 +0200 · **Modified** 2026-11-15 18:46 +0100

Not a class: three related **features of `C_game`**, built from two record types, one engine class and nine PC functions in
`[0x005f10d0, 0x005f1b10)`, which follow `game_savegame_load_before` and precede the media family. It has its own document because
it spans three `C_game` field bands. On the PS2 the object-lightness tick and the shoot-light adder are empty stubs, while the PC ships
both; the light cache is live on both builds (§8).

| | |
|---|---|
| PC prefix | `game_lightness_`, `game_add_shoot_`, `game_use_light_cache_`, `game_light_cache_`, `game_recompute_light_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x005dc160, 0x005f79a0)`, `g_game.cpp`, hosted by `structs/game.md` |
| PS2 class | `C_game` (tag `6C_game`): 8 methods in the tables here, `ShootLightTick` in §8; the rest are in `structs/game.md` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

| Feature | What it does | PC functions |
|---|---|---|
| object lightness | samples the scene's ambient light for each registered actor frame and fades the frame's shading toward it | `game_lightness_add`, `game_lightness_del`, `game_lightness_clear`, `game_lightness_tick`, `game_lightness_apply_recursive` |
| shoot lights | a pool of short-lived dynamic lights for muzzle flashes, explosions and fires | `game_add_shoot_light` |
| light cache | collects every `I3D_light` in the scene once, then enables or disables them by camera distance | `game_use_light_cache_system`, `game_light_cache_collect_recursive`, `game_recompute_light_cache` |

The typedef and the evidence of the `C_game` fields are in `structs/game.md`; this table says what each does here.

| Offset | `C_game` field | Role |
|---|---|---|
| `+0x150` | `lightness_records` | allocator byte; begin `+0x154`, end `+0x158`, cap `+0x15c`. Elements are `S_lightness_record *` |
| `+0x160` | `lightness_cursor` | round-robin sampling cursor, reset when it passes the end |
| `+0x164` | `actor_lightness_on` | enables the pass (§4b) |
| `+0x168` | `light_cache` | allocator byte; begin `+0x16c`, end `+0x170`, cap `+0x174`. Elements are `I3D_light *` |
| `+0x178` | `light_cache_countdown` | ms until the next distance pass; reloaded with `0x7e7` (2023 ms) by the pass, -1 from `game_init` and from the two entry points of §7 |
| `+0x2a48` | `shoot_lights` | ten `S_shoot_light` records, stride `0xc`, ending at `+0x2ac0` |

### 1a. `S_lightness_record` — one registered frame, `0x24` bytes

`game_lightness_add` allocates it with `operator_new(0x24)`. The name is the project's: the PS2 has no symbol for the record, because
the methods that would use it are stubs (§8a). It holds no array.

```c
typedef struct S_lightness_record {
    /* +0x00 */ C_actor *owner;        // [H] the actor; its frame gives the sample point, its state gates the sample
    /* +0x04 */ I3D_frame *frame;      // [H] the shaded frame and the record's key: `game_lightness_del` erases on it
    /* +0x08 */ u16 flags;             // [H] bit 0: skip the tag list and the sample, apply through the recursive walker
    /* +0x0c */ S_vector offset;       // [H] added to the owner frame's world position to give the sample point
    /* +0x18 */ float current;         // [H] the value applied to the frame, eased toward `target`; seeded 1.0
    /* +0x1c */ float target;          // [H] the last sampled ambient value; seeded 1.0
    /* +0x20 */ int last_sample_ms;    // [H] game clock at the last sample, drives the 250 ms gate; seeded 0
} S_lightness_record;
```

### 1b. `S_shoot_light`

The record's typedef and evidence are `structs/game.md` §1k and §2k. The pool's count of ten is `game_add_shoot_light` and `game_tick`
looping over exactly ten slots, so a burst beyond ten live lights finds no free slot and adds nothing.

## 2. Field evidence

### 2a. `S_lightness_record`

The records are reached through the vector, so these cells come from reading the named bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | `owner` | H | `game_lightness_add` | `game_lightness_tick` | `owner+0x68` is the frame, `+0x5f` and `+0x1c` the gates of §4b |
| +0x04 | `frame` | H | `game_lightness_add` | `game_lightness_add`, `game_lightness_del`, `game_lightness_tick` | a second add for one frame returns at once |
| +0x08 | `flags` | H | `game_lightness_add` | `game_lightness_tick` | §4d |
| +0x0c | `offset` | H | `game_lightness_add` | `game_lightness_tick` | §4d lists the values passed |
| +0x18 | `current` | H | `game_lightness_add`, `game_lightness_tick` | `game_lightness_tick` | - |
| +0x1c | `target` | H | `game_lightness_add`, `game_lightness_tick` | `game_lightness_tick` | stored from the `fstp` after the scene call, at `0x005f165f` |
| +0x20 | `last_sample_ms` | H | `game_lightness_add`, `game_lightness_tick` | `game_lightness_tick` | compared with `frame_time_ms` |

## 3. Method family

8 of the 9 PS2 `C_game` methods of this feature have a row here, one of them `inlined`; `ShootLightTick` has no host in this doc (§8).
`game_light_cache_collect_recursive` is the twin of a helper-class method. `game_lightness_apply_recursive` is PC-only (§5). The three PS2 twins
that are the stubs of §8a can match only as symbols, so they are STRUCTURAL. The free helpers of the unit are `structs/game.md` §3d.

### 3a. Construction & destruction

The feature has no constructor: `game_ctor` and `game_init` set up its fields (`structs/game.md`).

### 3b. Accessors

`game_set_actor_lightness_flag` writes `actor_lightness_on`; its row is in `structs/game.md` §3b.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005f10d0 | `game_lightness_add` | `LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector` | 0x002ff510 | STRUCTURAL | Refuses a null or known frame and, unless flag bit 0 is set, a frame outside the tag list; appends a new record |
| 0x005f1370 | `game_lightness_del` | `LightnessDel__6C_gameFP9I3D_frame` | 0x002ff520 | CONFIRMED | Deletes the frame's record and moves the last element over it |
| 0x005f1400 | `game_lightness_clear` | `LightnessClear__6C_gameFv` | 0x002ff5c0 | CONFIRMED | Deletes every record and empties the vector; callers `game_init`, `game_done` |
| 0x005f1460 | `game_lightness_tick` | `LightnessTick__6C_gameFUi` | 0x002ff620 | STRUCTURAL | Samples one record per frame, then eases and applies every record (§4b) |
| 0x005f1750 | `game_lightness_apply_recursive` | - | - | - | Applies a value to a frame subtree; ECX is the frame (§5) |
| 0x005f17c0 | `game_add_shoot_light` | `AddShootLight__6C_gameFR8S_vectorfiUs` | 0x002ff630 | STRUCTURAL | Claims the highest free pool slot, places the light, switches it on and stores lifetime and flags (§6a) |
| 0x005f1900 | `game_use_light_cache_system` | `UseLightCacheSystem__6C_gameFv` | 0x002ff7d0 | CONFIRMED | Empties the cache, sets the countdown to -1 and walks the scene root (§7) |
| 0x005f1950 | `game_light_cache_collect_recursive` | `cbLight__Q26C_game19S_hlp$18015game_cppFP9I3D_frameRQ23std50vector<P9I3D_light,Q23std22allocator<P9I3D_light>>` | 0x002ff8c0 | CONFIRMED | Appends the light children of a frame subtree to the cache (§7) |
| 0x005f19c0 | `game_recompute_light_cache` | `RecomputeLightCache__6C_gameFv` | 0x002ffa50 | CONFIRMED | Clears the countdown, runs one distance pass, clears it again (§7) |
| - | - | `LightCacheTick__6C_gameFUi` | 0x002ffa90 | inlined | one of its two copies; the other is the `light_cache` block of `game_tick` (§8d) |

### 3d. Free helpers

The feature has no free helper of its own.

## 4. Object lightness

### 4a. The tag lists

Unless bit 0 of the record flags is set, `game_lightness_add` accepts a frame only if its visual type at `I3D_frame+0x110` is **1** and its
FourCC tag at `I3D_frame+0x1d4` is one of nine:

`LMAP` `MRPH` `PRTB` `PRTE` `SNGM` `LENS` `LNDP` `SMMR` `OBJ_`

`game_lightness_apply_recursive` applies to visual type 1 and five of them when it walks a subtree: `LMAP` `MRPH` `SNGM` `SMMR` `OBJ_`.

The sets are read from the compare chains at `0x005f1133` and `0x005f1768`. The decompiler renders `cmp eax,0x50414d4c; jg` as a test
against `0x50414d4d`, which spells another tag; that value is an immediate of no instruction in `Game.exe`.

### 4b. What the tick does

`game_tick` calls `game_lightness_tick` only while `actor_lightness_on` is set, the vector is non-empty and the detail setting
`*(g_act_mission + 0x9c)` is 2 or more. Only the script command `actorlightnesson` sets the byte, `game_init` clears it on every scene,
and no shipped mission script calls the command, so in the retail game the pass does not run.

`game_lightness_tick(ms)` runs two passes over the same vector.

1. **Sample one record per frame.** The cursor selects it, and the record is sampled only when all of these hold:
   * more than **250 ms** since its last sample;
   * its frame is within **40 units** of the scene camera (the body tests the squared distance against `1600.0`);
   * the owner's `in_camera_sector` (`+0x5f`) is set;
   * the owner's `act_state` (`+0x1c`) is 0, active;
   * flag bit 0 is clear.

   The sample point is `offset` plus the world position of the owner's frame (`owner+0x68`), and the sector is the first ancestor of that
   frame with visual type 5. The body switches the owner's frame off through its vtable `+0x24`, calls the `I3D_scene` virtual `+0x68`
   (`GetLightness`) with the point, the sector, 1 and the record's frame, stores the answer as `target`, and restores the frame's on bit.
   The timestamp is written as soon as the first two gates pass.
2. **Fade every record.** `current` moves toward `target` by `ms * 0.004` (a full 0 to 1 swing takes 250 ms), snapping when the step would
   overshoot and doing nothing when the gap is below `1e-08`. A changed value is applied through the frame's vtable `+0x80`
   (`SetHRLightness`), or through `game_lightness_apply_recursive` when flag bit 0 is set.

**The cost is at most one scene query per frame in total, not per object.** With N registered objects each is considered every N frames.

### 4c. Who registers and unregisters

| Adds | Removes |
|---|---|
| `human_game_init` | `human_game_done` |
| `dog_game_init` | `dog_death` |
| `human_change_model` | `human_change_model` |
| `human_activate` | `human_deactivate` |
| - | `human_death` |

`game_lightness_add` has these four direct calls and `game_lightness_del` these five, and neither has any other reference in `Game.exe`.
`human_death` removes the record with no add-side partner.

### 4d. The flags word in practice

Bit 0 is the only bit any code tests, in three places: `game_lightness_add` (skip the tag list), and twice in `game_lightness_tick` (skip
the sample; apply through `game_lightness_apply_recursive`).

Every caller passes **0**, read from the pushed operands at all four callsites:

| Caller | Flags | Sampling offset |
|---|---|---|
| `dog_game_init` | 0 | `(0, 0.5, 0.5)` |
| `human_game_init` | 0 | `(0, 1, 0)` |
| `human_change_model` | 0 | `(0, 1, 0)` |
| `human_activate` | 0 | `(0, 1, 0)` |

So the tag list always applies, and the recursive path is never selected by a record.

## 5. `game_lightness_apply_recursive` is not an `I3D_frame` method

Its receiver in ECX is a **frame**, not the `C_game` singleton; an ECX census of its callsites grades 2 safe, 0 unsafe: `mov ecx,[eax+4]`
(the record's frame) plus the recursive self-call. It keeps the `game_` prefix because it is a file-static of `g_game.cpp`, like the
unit's two other recursive frame walkers `game_invalidate_frame_recursive` and `game_free_frame_placement_recursive`, whose ECX is a frame
too. It walks the children through the child count `+0x11c` and the `GetChild` virtual `+0x30`.

## 6. Shoot lights

`game_init` creates the ten lights (driver object type 2, each switched off, lifetime -1).

### 6a. What `game_add_shoot_light` does

It scans the pool from slot 9 downward for the first record whose lifetime is below 1, then on that light: sets the range through vtable
`+0x64` with `(range * 0.5, range)`; unless bit 7 of the frame's flag byte `+0xac` is set, writes the position into the frame at
`+0x80`..`+0x88` and rewrites the frame flags `& 0xfffffedf | 0x40000000`; re-links it into the scene sector and tells a kind-5 parent through
its vtable `+0x50`; switches it on; gives it an intensity of `rand() * 0.4 + 0.8`; and stores the caller's lifetime and flags.

| Caller | Range | Lifetime | Flags |
|---|---|---|---|
| `human_shooting` (muzzle flash) | 7.0 | 50 ms | 0 |
| `actor_shot_update` | 7.0 | 50 ms | 0 |
| `game_new_explosion` | 12.0 | 100 ms | 0 |
| `game_new_fire` | 10.0 | the fire's own lifetime | 1 |

### 6b. The per-frame tick is inlined into `game_tick`

`game_tick` walks all ten records every frame. For a slot with lifetime left it subtracts the frame time; below 1 it tells a kind-5 parent
through its vtable `+0x54`, unlinks the light and switches it off. While the slot lasts and flag bit 0 is set, the intensity is re-rolled to
`rand() * 0.4 + 0.8` every frame while more than 500 ms remain, and set to `lifetime * 0.002` below that, a linear fade. A slot without the
flag keeps its creation intensity until it goes out.

A displacement sweep for the record fields does not see this loop: it starts at `lea esi,[ebp+0x2ac0]`, the END of the pool, and reads each
record at negative offsets.

## 7. The light cache

`game_use_light_cache_system` empties the vector, sets the countdown to -1 and hands the scene root to
`game_light_cache_collect_recursive`, which walks the frame tree (`+0x124` first child, `+0x12c` next sibling). It appends every child of
visual type **2** whose sub-type `+0x154` is **1 or 2**, the two `I3D_light` kinds, and it passes over a child that an actor owns: its
`owner_cell` (`+0x08`, `structs/i3d_frame.md`) is set and the cell holds an actor, so the lights of an actor's subtree stay out of the
cache. The element type comes from the PS2 template argument `std::vector<P9I3D_light>`.

The **distance pass** compares each cached light's squared distance to the scene camera against `(light_radius + camera_range + 15.0)²`, with
the radius at `light+0x16c` and the range at `camera+0x148`, and enables or disables the light through its vtable `+0x24`. It exists twice:

| Site | When it runs |
|---|---|
| the `light_cache` block of `game_tick` | every frame with a non-empty cache: a countdown below 1 reloads `0x7e7` and runs the pass, anything else subtracts the frame time, so a pass runs about every 2 s |
| `game_recompute_light_cache` | on demand: it sets the countdown to -1, runs the pass and sets -1 again, so the next frame's `game_tick` runs it once more |

Both entry points are reached from `program_process_command`. `game_use_light_cache_system` is the script command `use_lightcache`;
`game_recompute_light_cache` runs at the shared tail of the `camera_lock` / `camera_unlock` arms.

**Seven shipped scenes arm the cache**, found by a byte search of every shipped mission file: `FREERIDENOC`, `MISE01`, `MISE05-MESTO`,
`MISE07-SARA`, `MISE09-MESTO`, `MISE11-MESTO` and `MISE16-MESTO`. In each the `GameInitStart` script runs `game_nightmission 1` and then
`use_lightcache` on the next line. The other 17 night scenes, which also run `game_nightmission 1`, never arm it.

## 8. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002ff640 | `ShootLightTick__6C_gameFUi` | inlined into `game_tick`, which is listed in `structs/game.md` §3c (§8c) |

### 8a. Three methods are empty stubs on the PS2

Verified by reading the eight bytes of each body, not inferred from the symbol size:

| PS2 symbol | Size | Body |
|---|---|---|
| `LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector` | 8 | `return;` |
| `LightnessTick__6C_gameFUi` | 8 | `return;` |
| `AddShootLight__6C_gameFR8S_vectorfiUs` | 8 | `return;` |

`LightnessDel`, `LightnessClear`, `ShootLightTick` and the four light-cache methods have real bodies. **Here the PS2 method count is a lower
bound, not an upper one.** A row whose PS2 side is one of the three stubs can be matched as a symbol but never as a body, so it is STRUCTURAL.

The five PS2 calls of `LightnessAdd` pass flags 0 as well: `GameInit__5C_dogFv`, `GameInit__7C_humanFv`,
`_intern_ChangeModel__7C_humanFPcb`, `Activate__7C_humanFv` and `VystrcPero__7C_humanFv`.

### 8b. Why `game_lightness_apply_recursive` has no twin

It is **not** the twin of the PS2's `SetLightnessHR__FP9I3D_frame` at `0x004251d0`, a free function whose only callers are
`I3D_driver::SetLightness` and one sibling. That address band is `I3d_driver.cpp`, the LS3D engine, which the PS2 compiled into the ELF and
the PC ships in `ls3df.dll`, so the PC's copy of that function is in a DLL. Two differences confirm they are not the same function: the PS2
gates on flag `0x800` of `I3D_frame+0x1d0` where the PC uses the FourCC list, and the PS2 recurses over the raw sibling chain where the PC uses
the child-count and `GetChild` virtuals.

### 8c. `ShootLightTick`

The PS2 method is 388 bytes and walks **2** records of the identical `{I3D_light *, int lifetime, ushort flags}` stride-`0xc` layout at
`C_game+0xe0c`. The PC copy (§6b) matches it statement for statement: the countdown, the parent notification and switch-off on expiry, and
the bit-0 branch with `rand() * 0.4 + 0.8` above 500 ms and `lifetime * 0.002` below. It differs only in the pool size, **10** slots against
**2**. With `AddShootLight` stubbed, the PS2 tick never finds a lit slot.

### 8d. `LightCacheTick`

`LightCacheTick` has two PS2 callers: `Tick__6C_gameFUi` passes the frame time and `RecomputeLightCache__6C_gameFv` passes 1, between two stores
of -1 to its countdown `+0x1c4`. The PC has no function for it and holds both copies inline. The block in `game_tick` matches the PS2 body
statement for statement: the non-empty test, the `< 1` test, the `0x7e7` reload, the `+ 15.0` term, the enable or disable call and the `else`
that subtracts the frame time. The copy in `game_recompute_light_cache` drops the `< 1` test, because the store of -1 just above it decides that
test. So the countdown throttles the pass during normal ticking on both builds.

## 9. Related

`structs/game.md` (the owning class) · `structs/fake_shadow.md` (the sibling `C_game` rendering feature) · `structs/actor.md` (`in_camera_sector`,
`act_state`) · `basic/artifacts/dll.md` (why `SetLightnessHR` is not in `Game.exe`) · `Game-ghidra.exe.functions.md` (the function registry).

## 10. Open items

| Item | State |
|---|---|
| in game: object lightness | not tested. In the retail game no actor should darken, because the pass is gated off (§4b); with `actorlightnesson` executed, an active actor in a dark alley should darken over about 0.25 s once within 40 units of the camera and in its sector |
| the count of light-cache scenes (§7) | a search of the extracted scripts reproduces six of the seven and 22 night scenes; `MISE09-MESTO` is not among the extracted missions, so the seventh and the 24 rest on the byte search of the mission files, not repeated here |
