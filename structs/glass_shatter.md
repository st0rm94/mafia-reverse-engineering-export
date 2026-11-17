# `C_Break_plain_glass` — the glass shatter engine

**Created** 2026-09-04 13:25 +0200 · **Modified** 2026-11-16 21:04 +0100

The engine that turns a window into falling fragments. It cuts the pane's mesh into loose
triangles, spawns each as an `'OBJ_'` frame with its own velocity and spin, ticks them under
gravity until they come to rest, and then fuses the settled shards of that pane back into a
single static object so a broken window does not cost a draw call per fragment for the rest
of the mission.

It is a **`C_game` sub-object at `C_game + 0x208`**, not a singleton with a global — which
is why the whole translation unit contains **zero data symbols**.

| | |
|---|---|
| PC prefix | `glass_shatter_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x38`, embedded at `C_game + 0x208` |
| PC constructor | -: no constructor of its own; `game_ctor` initialises it inline and `glass_shatter_init` does the work |
| PC translation unit | `[0x0061e0d0, 0x00622230)`, `VE_BreakageGlass.cpp` (§7), guard to guard; `glass_shatter_dtor` is emitted far outside the range |
| PS2 class | `C_Break_plain_glass`, 6 methods, plus the free `PrepareGlassSingleObject__FR6S_DICE` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_Break_plain_glass {
    /* +0x00 */ char        restore_alloc;  // [H] allocator byte; its address is the restore vector's receiver
    /* +0x04 */ I3D_frame **restore_begin;  // [H] panes kept for restore (§4): begin
    /* +0x08 */ I3D_frame **restore_end;    // [H] panes kept for restore: end
    /* +0x0c */ I3D_frame **restore_cap;    // [H] panes kept for restore: capacity
    /* +0x10 */ char        pane_alloc;     // [H] allocator byte; pinned by `lea ecx,[eax+0x10]` at the insert
    /* +0x14 */ S_DICE     *pane_begin;     // [H] the live broken panes: begin
    /* +0x18 */ S_DICE     *pane_end;       // [H] the live broken panes: end
    /* +0x1c */ S_DICE     *pane_cap;       // [H] the live broken panes: capacity
    /* +0x20 */ S_DICE     *pool;           // [H] the ring of pane slots, allocated by glass_shatter_init
    /* +0x24 */ int         ring_cursor;    // [H] advanced modulo pool_slots when a pane retires (§2)
    /* +0x28 */ int         pool_slots;     // [H] the Init argument: broken panes alive at once; game_init 5
    /* +0x2c */ float       gravity_x;      // [H] 0.0
    /* +0x30 */ float       gravity_y;      // [H] -8.0 (0xc1000000)
    /* +0x34 */ float       gravity_z;      // [H] 0.0
} C_Break_plain_glass;                      // embedded at C_game + 0x208
```

The two vectors store only begin / end / capacity, and neither has a set count. `pool` is a heap array of
`pool_slots` records, and each entry is a recycled `S_DICE`.

### 1a. `S_DICE` — one broken pane (`0x18`)

```c
typedef struct S_DICE {
    /* +0x00 */ char        merged_alloc;   // [H] MSVC allocator byte of the vector below
    /* +0x04 */ I3D_frame **merged_begin;   // [H] the fused frames, one per distinct material: begin
    /* +0x08 */ I3D_frame **merged_end;     // [H] the fused frames: end
    /* +0x0c */ I3D_frame **merged_cap;     // [H] the fused frames: capacity
    /* +0x10 */ S_DICE_OBJ *shards;         // [H] the loose shard array
    /* +0x14 */ int         shard_count;    // [H] entries in shards
} S_DICE;
```

The record owns a heap buffer, which is why its vector cannot block-copy: the insert path
goes through `glass_shatter_pane_construct_copy` and `glass_shatter_pane_copy_assign`
rather than a `memcpy`.

`glass_shatter_done` reads the two arms in exactly that order: if `merged_*` is non-empty
the pane has been fused and only those frames are released; otherwise the raw
shard array is walked.

### 1b. `S_DICE_OBJ` — one shard (`0x34`)

```c
typedef struct S_DICE_OBJ {
    /* +0x00 */ I3D_frame  *frame;          // [H] the 'OBJ_' frame carrying this triangle
    /* +0x04 */ float       vel_x;          // [H] impact direction x a random -0.20..+0.79 factor
    /* +0x08 */ float       vel_y;          // [H] the same, y
    /* +0x0c */ float       vel_z;          // [H] the same, z
    /* +0x10 */ float       spin_angle;     // [H] integrated by spin_rate each tick
    /* +0x14 */ float       spin_rate;      // [H] (rand() % 0x78 + 0x14) * 0.1 = 2.0..13.9
    /* +0x18 */ S_vector    contact_point;  // [H] out-param of the downward collision probe (§2b)
    /* +0x24 */ S_vector    contact_normal; // [H] the plane the shard rests on (§2b)
    /* +0x30 */ char        animating;      // [H] 1 while live; cleared at the last placement (§2b)
    /* +0x31 */ char        falling;        // [H] 1 while in motion (§2b)
    /* +0x32 */ char        probed;         // [H] the ground-probe latch (§2b)
    /* +0x33 */ char        pad33;          // [H] tail padding: no instruction addresses it
} S_DICE_OBJ;
```

The three bytes are all seeded to **1** by `glass_shatter_break_frame`, so a fresh shard is
animating, falling and un-probed.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | restore_alloc | H | - | `glass_shatter_break_frame` (by address, `std_vector4_insert_at`) | the receiver of the vector helper (`mov ecx, ebp`); the byte itself is never stored |
| +0x4 | restore_begin | H | `glass_shatter_break_frame` (by address, `std_vector4_insert_at`), `glass_shatter_dtor` | `glass_shatter_break_frame` (by address, `std_vector4_insert_at`), `glass_shatter_done`, `glass_shatter_dtor` | - |
| +0x8 | restore_end | H | `glass_shatter_break_frame`, `glass_shatter_done`, `glass_shatter_dtor` | `glass_shatter_break_frame`, `glass_shatter_done` | - |
| +0xc | restore_cap | H | `glass_shatter_break_frame` (by address, `std_vector4_insert_at`), `glass_shatter_dtor` | `glass_shatter_break_frame` (by address, `std_vector4_insert_at`) | - |
| +0x10 | pane_alloc | H | `glass_shatter_break_frame` (by address, `glass_shatter_pane_list_insert_n`) | `glass_shatter_break_frame` | the receiver of the vector helper; the byte itself is never stored |
| +0x14 | pane_begin | H | `glass_shatter_dtor`, `glass_shatter_pane_list_insert_n` | `glass_shatter_done`, `glass_shatter_dtor`, `glass_shatter_init`, `glass_shatter_tick` | - |
| +0x18 | pane_end | H | `glass_shatter_done`, `glass_shatter_dtor`, `glass_shatter_init`, `glass_shatter_pane_list_insert_n`, `glass_shatter_tick` | `glass_shatter_done`, `glass_shatter_dtor`, `glass_shatter_init`, `glass_shatter_tick` | - |
| +0x1c | pane_cap | H | `glass_shatter_dtor`, `glass_shatter_pane_list_insert_n` | `glass_shatter_pane_list_insert_n` | - |
| +0x20 | pool | H | `glass_shatter_init` | `glass_shatter_done`, `glass_shatter_tick` | - |
| +0x24 | ring_cursor | H | `glass_shatter_init`, `glass_shatter_tick` | `glass_shatter_tick` | `glass_shatter_init` seeds it to `pool_slots`, so the first advance wraps to 0 |
| +0x28 | pool_slots | H | `glass_shatter_init` | `glass_shatter_done`, `glass_shatter_init`, `glass_shatter_tick` | - |
| +0x2c | gravity_x | H | `glass_shatter_init` | `glass_shatter_init`, `glass_shatter_tick` | the write is the init's float constant |
| +0x30 | gravity_y | H | `glass_shatter_init` | `glass_shatter_tick` | - |
| +0x34 | gravity_z | H | `glass_shatter_init` | `glass_shatter_tick` | - |

### 2a. `S_DICE`

The cells of §2a and §2b come from the bodies: the functions walk both records with a moving pointer.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | merged_alloc | H | `folded_vector_ctor_at_0x04_0044a120` (each pool slot, through `crt_ehvec_ctor` in `glass_shatter_init`), `glass_shatter_pane_construct_copy` (the record copy) | `glass_shatter_pane_construct_copy` (the copy's source) | copied with the record; nothing tests it |
| +0x4 | merged_begin | H | `glass_shatter_dtor`, `glass_shatter_merge_settled_shards` | `glass_shatter_done`, `glass_shatter_dtor`, `glass_shatter_tick` | - |
| +0x8 | merged_end | H | `glass_shatter_dtor`, `glass_shatter_merge_settled_shards` | `glass_shatter_done`, `glass_shatter_dtor`, `glass_shatter_tick` | - |
| +0xc | merged_cap | H | `glass_shatter_dtor`, `glass_shatter_merge_settled_shards` | `glass_shatter_merge_settled_shards` | - |
| +0x10 | shards | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_done`, `glass_shatter_merge_settled_shards`, `glass_shatter_tick` | - |
| +0x14 | shard_count | H | `glass_shatter_break_frame` | `glass_shatter_done`, `glass_shatter_merge_settled_shards`, `glass_shatter_tick` | - |

### 2b. `S_DICE_OBJ`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | frame | H | `glass_shatter_break_frame` | `glass_shatter_done`, `glass_shatter_merge_settled_shards`, `glass_shatter_tick` | - |
| +0x4 | vel_x | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | - |
| +0x8 | vel_y | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | - |
| +0xc | vel_z | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | - |
| +0x10 | spin_angle | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | - |
| +0x14 | spin_rate | H | `glass_shatter_break_frame` | `glass_shatter_tick` | - |
| +0x18 | contact_point | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | written through a pointer by the probe; its y is lifted 0.05 above the surface |
| +0x24 | contact_normal | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | written through a pointer by the probe; the tick dots the position against it to detect landing |
| +0x30 | animating | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | all shards at 0 is what triggers the pane's fuse-and-recycle |
| +0x31 | falling | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | 0 makes the next tick take the final-placement branch that clears `+0x30` |
| +0x32 | probed | H | `glass_shatter_break_frame`, `glass_shatter_tick` | `glass_shatter_tick` | set after a probe, cleared on a bounce so the next descent re-probes |
| +0x33 | pad33 | H | - | - | no access; the float members round the record to `0x34` |

## 3. Method family

6 of 6 PS2 methods and the free `PrepareGlassSingleObject` mapped. The container helpers are graded by their own ECX
census: a helper whose every safe receiver sits in this unit takes the `glass_shatter_` family name; one used by several
unrelated modules takes a mechanical name. On the PS2 the containers are other functions (§7), so none of them carries
a twin. For the stride-6 (`I3D_triface`) family, censusing the leaf primitives as well as the members is what proves the
whole family has one owner.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00607450 | `glass_shatter_dtor` | instance | - | - | - | called by `game_dtor`: frees each pane's merged vector, then the pane vector, then the restore vector. The PS2 build has no destructor symbol for the class |
| 0x0061e0d0 | `crt_static_init_guard_0061e0d0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer, slot `0x006462f0` |
| 0x0061e0f0 | `crt_atexit_register_0061e0f0` | the translation unit | - | - | - | registers the unit's static destructor |

### 3b. Accessors

None.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061e100 | `glass_shatter_merge_settled_shards` | `PrepareGlassSingleObject__FR6S_DICE` | 0x003b62c0 | CONFIRMED | `this = S_DICE` fuse one pane's settled shards, one frame per material. A free function on both builds |
| 0x0061e8f0 | `glass_shatter_init` | `Init__19C_Break_plain_glassFi` | 0x003b70e0 | CONFIRMED | gravity `(0, -8, 0)`, allocate the ring pool |
| 0x0061ea60 | `glass_shatter_done` | `Done__19C_Break_plain_glassFv` | 0x003b7170 | CONFIRMED | release every live pane, free the pool |
| 0x0061ecd0 | `glass_shatter_tick` | `Tick__19C_Break_plain_glassFUi` | 0x003b75c0 | CONFIRMED | integrate every shard; retire settled panes |
| 0x0061fb20 | `glass_shatter_pane_copy_assign` | - | - | - | `this = S_DICE` 1 SAFE receiver, in `glass_shatter_done`: one owner |
| 0x0061fc90 | `glass_shatter_break_frame` | `BreakGlass__19C_Break_plain_glassFP9I3D_framePC8S_vectorPCcb` | 0x003b89e0 | CONFIRMED | the public entry point |
| 0x006202a0 | `glass_shatter_cut_mesh` | `CutMesh__19C_Break_plain_glassFP9I3D_framePQ23gif23vector<11I3D_triface,8>PQ23gif22vector<10I3D_vertex,8>P12I3D_materialR8S_matrix` | 0x003b9370 | CONFIRMED | split the visible faces into loose triangles |
| 0x00621130 | `glass_shatter_prepare_object` | `PrepareObject__19C_Break_plain_glassFiPQ23gif23vector<11I3D_triface,8>PQ23gif22vector<10I3D_vertex,8>P12I3D_materialP10I3D_object` | 0x003bb170 | CONFIRMED | build one shard's `I3D_object` |
| 0x00621850 | `glass_shatter_triface_list_insert_at` | - | - | - | `this = none` 1 SAFE receiver, in `glass_shatter_cut_mesh`: one owner |
| 0x00621a50 | `glass_shatter_triface_list_size` | - | - | - | `this = none` 1 SAFE receiver, in the insert above: one owner |
| 0x00621a70 | `glass_shatter_pane_list_insert_n` | - | - | - | `this = +0x10` 1 SAFE receiver, `lea ecx,[eax+0x10]` in `glass_shatter_break_frame`: one owner |
| 0x00621d70 | `glass_shatter_triface_list_insert_n` | - | - | - | `this = none` 2 SAFE receivers, both in `glass_shatter_cut_mesh`: one owner |
| 0x006220e0 | `glass_shatter_triface_list_uninit_copy` | - | - | - | `this = none` 3 SAFE receivers, all in the insert: one owner |
| 0x00622120 | `glass_shatter_triface_list_fill_n` | - | - | - | `this = none` 1 SAFE receiver, in the insert: one owner |
| 0x00622160 | `glass_shatter_pane_construct_copy` | - | - | - | `this = S_DICE` 6 SAFE receivers, all in `glass_shatter_pane_list_insert_n`: one owner |
| 0x00622210 | `glass_shatter_triface_list_construct_copy` | - | - | - | `this = none` 2 SAFE receivers, both in the insert: one owner |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00621660 | `std_vector4_capacity` | - | - | - | 7 SAFE receivers in six unrelated modules: shared, mechanical name |
| 0x00621680 | `std_allocator_deallocate` | - | - | - | 1 dataref (`C_chunk` vtable) and 1 foreign caller: shared, mechanical name |
| 0x00621690 | `std_vector20_insert_at` | - | - | - | 4 SAFE receivers in two modules (with `vehicle_init_lights`): shared, mechanical name |
| 0x00621830 | `std_vector20_size` | - | - | - | 4 SAFE receivers in two modules (with `game_box_save_info`): shared, mechanical name |
| 0x00622070 | `stl_uninit_copy_stride_0x20` | - | - | - | 3 SAFE and 4 UNSAFE receivers in two modules: shared, mechanical name |
| 0x006220b0 | `stl_fill_n_stride_0x20` | - | - | - | 3 SAFE receivers in two modules: shared, mechanical name |

## 4. The life of a broken window

```
<caller>  ->  glass_shatter_break_frame(frame, dir, sound, keep)
                 |- gate: frame+0x110 == 1 && frame+0x1d4 == 'OBJ_'   (`structs/i3d_frame.md` §5a)
                 |- glass_shatter_cut_mesh      -> loose triangles + the pane's material
                 |- per shard: driver 'OBJ_' frame
                 |             glass_shatter_prepare_object   -> its two-triangle geometry
                 |             seed velocity, spin, contact plane
                 |- glass_shatter_pane_list_insert_n           -> the pane goes live
                 `- game_play_3d_sound_by_name(sound)          default "lustr1.wav"

game_tick ->  glass_shatter_tick(dt)
                 |- per shard: gravity, probe, bounce, integrate, write the frame transform
                 `- when every shard is settled:
                       glass_shatter_merge_settled_shards      -> one frame per material
                       ring_cursor = (ring_cursor + 1) % pool_slots
                       the slot the cursor now points at is recycled
```

**The ring is why old debris disappears.** Only `pool_slots` (5) panes can be broken at once;
breaking the next one silently retires the oldest. Confirmed in game.

**The fourth argument restores the pane at scene end.** `glass_shatter_break_frame` always
switches the original pane frame off (frame vtable `+0x24`, `SetOn(0)`). With the fourth argument
set it also bumps the frame's reference count and keeps it in the `+0x04` list;
`glass_shatter_done`, whose only caller is `game_done`, switches every kept frame back on
(`SetOn(1)`) and releases it. Six of the seven callers (§5) pass 1; `crash_obj_ai` passes 0, and
the frame it breaks is the crash object's own frame at `+0x68`.

## 5. Callers of `glass_shatter_break_frame`

All seven reach the sub-object as `[obj + 0x24] + 0x208` or `lea ecx,[reg + 0x208]`:

| caller | what breaks |
|---|---|
| `car_break_glass` | a car window ¹ |
| `door_hit` | a door pane |
| `rail_window_hit` | a tram / metro window |
| `crash_obj_ai` | a breakable prop's glass |
| `human_ai` | glass an NPC walks or falls through |
| `game_tick_shoot` | a bullet impact |
| `game_new_explosion` | an explosion's blast radius |

¹ The frame comes from the stride-0x34 window list at `C_car + 0x308`; default sound
`"lustr1.wav"`.

So the class is **not** limited to non-vehicle glass (§6).

## 6. What "plain" means

The class name is `C_Break_plain_glass`. The PC prefix **drops the adjective**, because no
measurement gives it a referent:

* It is **not** "flat pane glass, as opposed to vehicle glass": `car_break_glass` routes car
  windows straight into this class (§5).
* The entry gate is **not** a planarity test. `frame+0x110 == 1 && frame+0x1d4 == 'OBJ_'` is
  the project-wide "is this a real model?" gate (`structs/i3d_frame.md` §5a); it rejects `'LMAP'`,
  `'BBRD'`, `'MRPH'` and `'LENS'` frames and says nothing about flatness.
* The cutter never uses a plane primitive. `S_plane::IntersectionLine` has exactly **2
  callers image-wide and both are inside `physical_object_move_wheel`**.
  `glass_shatter_cut_mesh` takes face 0's normal as a reference and keeps faces that dot
  positive against it — it *assumes* a single-sided sheet and never verifies one.
* **"plain" contrasts with nothing in either binary** (§7).

The nearest piece of shipped vocabulary is the literal **`"[DEFORM GLASS]"`**
(`0x0064c630`), a frame name in the railway model block, read only by
`railway_enumerate_scene_frames` and `railway_cb_enum_dummies`. That makes *"plain = the
not-deform kind"* the most plausible reading — plausible, not measured, and
`rail_window_hit` shatters through this class anyway.

No shipped file contains the word either — no PC string, no table, no DLL — so what the
developers meant by it cannot be recovered from the game.

Both words the prefix keeps are backed: every caller is a glass pane and the default sound is
`lustr1.wav` (Czech *lustr* = chandelier), and the body cuts a mesh into shards that fall,
settle and merge — shattering, which is precisely what `vehicle_deform_glass`
(`structs/vehicle.md`) does not do.

## 7. PC vs PS2

**Translation unit.** `VE_BreakageGlass.cpp`, named by the PS2's own
`__sinit_VE_BreakageGlass.cpp` (`0x004d0a10`). The PC `.CRT$XCU` slot `0x006462f0` opens it
at `0x0061e0d0`; the PS2 manifest places it between `tshift.cpp` and `VE_Debris.cpp`, and
the PC boundaries agree — a five-of-five ordered lockstep.

**Free function.** `PrepareGlassSingleObject__FR6S_DICE` has no class tag (`__F`), so
`glass_shatter_merge_settled_shards` is a free function on both builds.

**"plain".** A closed-set check over the PS2 symbol table with the class column ignored finds
`C_Break_plain_glass` to be the only `C_Break_*` class and the only symbol in either build
containing the word.

**Gravity.** The `-8.0` literal (`0xc1000000`) is shared with the PS2.

**Record sizes.** PS2 pads for VU alignment: `S_DICE` `0x10` against the PC's `0x18`
(the PC's MSVC `std::vector` carries a padded allocator byte), `S_DICE_OBJ` `0x40` against
`0x34`.

**Containers.** PS2 uses Metrowerks `gif::vector<T,8>`, which grows through `reserve`;
MSVC grows inside `insert`. `reserve__Q23gif17vector<6S_DICE,8>Fi`,
`…<10I3D_vertex,8>` and `…<11I3D_triface,8>` are therefore **structural correspondents** of
the PC helpers in §3, not twins, and none of them carries a PS2 twin.

**The shard's default constructor is out of line on both builds.** `__ct__10S_DICE_OBJFv`
(8 B, `0x003b9360`) is `return this`. `glass_shatter_break_frame` default-constructs the new
shard array with a counted loop that calls `identity_thunk` (`mov eax, ecx; ret`) once per shard,
stepping `0x34` — the same empty body. That address also serves `identity_thunk`'s other
receivers, so it is shared and carries no twin line; the decomp site carries an inline banner.

**Init.** `glass_shatter_init` allocates the ring pool. The PS2 has no pool at all: its `Init`
zeroes the pool pointer `+0x28`, the cursor `+0x2c` and the slot count `+0x30` and never reads its
count argument (`C_game::Init` passes 5), and no other PS2 function stores to `+0x28` or `+0x30`.
The PS2 `Tick` still indexes `+0x28` when a pane retires.

**Element type names** come from the PS2 template arguments. The PC decomp can give a stride
but never a type, so `I3D_triface` (6 B) and `I3D_vertex` (0x20 B) are PS2-supplied facts about
a PC data structure.

**The size ratios** run 0.53–0.73, consistent with x86 against MIPS throughout; nothing in
this TU is an outlier.

## 8. Related

* `structs/game.md` — the owner; this class is the sub-object at `C_game + 0x208`.
* `structs/debris.md` — the sibling TU, `C_Debris` at `C_game + 0x240`. The two are
  consecutive object files and share the `'OBJ_'` spawn machinery.
* `structs/i3d_frame.md` §5a — the `+0x110` / `+0x1d4` model gate this class uses.
* `structs/car.md` — `car_break_glass` and the window list at `C_car + 0x308`.
* `structs/vehicle.md` — `vehicle_deform_glass`, the *other* glass effect: it cracks
  a windshield in place and never spawns a shard.

## 9. Open items

Nothing is open.
