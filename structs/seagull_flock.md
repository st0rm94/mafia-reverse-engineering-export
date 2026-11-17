# `C_seagull_flock` — entity kind `0x17`, a shootable bird flock nothing constructs

**Created** 2026-08-16 22:24 +0200 · **Modified** 2026-11-09 20:59 +0100

A complete, working actor class that the shipped game never builds. The class name is ours: neither
binary names it, the PS2 has no such class and the PC has no RTTI. It is called the seagull flock
because the only model it loads is `racek.i3d` (*racek* is Czech for seagull).

| | |
|---|---|
| PC prefix | `seagull_flock_` |
| PC kind (`+0x10`) | `0x17` |
| PC vtable | `g_seagull_flock_vt` @ `0x0063cc98` (39 slots) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x244` (`operator_new` in `mission_create_actor` case `0x17`) |
| PC constructor | `seagull_flock_ctor` @ `0x004ea1d0` |
| PC translation unit | `[0x004ea1a0, 0x004ec180)`, guard to guard: 13 starts, all named |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x70` is the `C_actor` base. The class also uses the kind `+0x10`,
the savegame exclusion byte `+0x65` (set by `seagull_flock_game_init`) and the frame `+0x68`.

```c
typedef struct C_seagull_flock {              // 0x244 bytes; C_actor base +0x00..+0x70
    /* +0x70 */ tDynamicCollObject coll;      // [H] collision record over the box, 0x48 bytes, never inserted
    /* +0xb8 */ S_matrix coll_matrix;         // [H] identity matrix the collision record points at
    /* +0xf8 */ S_vector box_min;             // [H] wander box, minimum corner
    /* +0x104 */ S_vector box_max;            // [H] wander box, maximum corner
    /* +0x110 */ uint8_t birds_alloc;         // [H] allocator byte of the bird vector
    /* +0x114 */ S_seagull_bird *birds_begin; // [H] first bird
    /* +0x118 */ S_seagull_bird *birds_end;   // [H] one past the last bird
    /* +0x11c */ S_seagull_bird *birds_cap;   // [H] end of the vector's storage
    /* +0x120 */ float planes[18][4];         // [H] x, y, z, d of each outline plane
    /* +0x240 */ uint32_t plane_count;        // [H] planes the wander test reads
} C_seagull_flock;
```

**Why `planes` has 18 entries.** No constant says so: 18 is the room between `+0x120` and
`plane_count` at `+0x240`, and the walk that fills it (§5) has no bound check, so a longer walk would
run over `plane_count`. Each entry is one plane, written `normal.x, normal.y, normal.z, d`. Entries 0
and 1 end up as the floor and the ceiling, the rest are the vertical walls of the outline, one per edge
walked. The wander test reads `plane_count` entries; the steering test reads entries 2..5 only.

### 1a. `S_seagull_bird` — one bird, `0x5c` bytes

```c
typedef struct S_seagull_bird {               // 0x5c bytes, the vector's stride
    /* +0x00 */ I3D_frame *frame;             // [H] the bird's model frame
    /* +0x04 */ uint32_t state;               // [H] §2a
    /* +0x08 */ S_vector heading;             // [H] direction the aim point moves along
    /* +0x14 */ S_vector aim_point;           // [H] the point the frame steers toward
    /* +0x20 */ float    leg_seconds;         // [H] seconds until next_heading takes over
    /* +0x24 */ S_vector next_heading;        // [H] direction of the following leg
    /* +0x30 */ S_vector unread_30[3];        // [U] state 1 stores rows 0 and 2; no reader (class census)
    /* +0x54 */ uint32_t unused_54;           // [U] no access (class census)
    /* +0x58 */ int32_t  unread_58;           // [U] 50..450 at spawn, 2000 from the hit; no reader (class census)
} S_seagull_bird;
```

**Why `unread_30` has three rows.** Its size is measured, its meaning is not: state 1 writes `(1,0,0)` to
`+0x30` and `(0,0,1)` to `+0x48`, the first and third rows of a 3 x 3 block that ends at `+0x54`. Row 1
is never written, no function reads the block, and `unused_54` has no access in any function of the class.

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions and by reading
all eleven bodies. The bird record is read through a walking pointer, so its cells come from the bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x70 | coll | H | `seagull_flock_game_init` | `seagull_flock_ctor`, `seagull_flock_game_init` | type `0x40`, flags `0x40006`, the box's centre, half diagonal and corners, owner = this; **no function of the class passes it to `collision_dyn_insert`**, Open item 2 |
| +0xb8 | coll_matrix | H | `seagull_flock_game_init` (by address, `S_matrix::Identity`) | `seagull_flock_game_init` | the identity matrix; `coll` points at it twice |
| +0xf8 | box_min | H | `seagull_flock_game_init` | `seagull_flock_ai`, `seagull_flock_game_init` | reset to `+FLT_MAX`, grown by the walk (§5) |
| +0x104 | box_max | H | `seagull_flock_game_init` | `seagull_flock_ai`, `seagull_flock_game_init` | reset to `-FLT_MAX`, grown by the walk |
| +0x110 | birds_alloc | H | `seagull_flock_ctor` | `seagull_flock_game_init` | the vector's allocator byte |
| +0x114 | birds_begin | H | `seagull_flock_ctor`, `seagull_flock_dtor` | `seagull_flock_ai`, `seagull_flock_destroy_on_collision`, `seagull_flock_dtor`, `seagull_flock_game_done`, `seagull_flock_hit` | - |
| +0x118 | birds_end | H | `seagull_flock_ctor`, `seagull_flock_destroy_on_collision`, `seagull_flock_dtor`, `seagull_flock_game_done` | `seagull_flock_ai`, `seagull_flock_destroy_on_collision`, `seagull_flock_game_done`, `seagull_flock_hit` | - |
| +0x11c | birds_cap | H | `seagull_flock_ctor`, `seagull_flock_dtor` | - | - |
| +0x120 | planes | H | `seagull_flock_game_init` | `seagull_flock_ai`, `seagull_flock_game_init` | written through `lea [this + 0x120]` for the floor plane and through `this + (plane_count + 0x12) * 0x10` for the walls, so the census sees only the two floor / ceiling lines |
| +0x240 | plane_count | H | `seagull_flock_game_init` | `seagull_flock_ai`, `seagull_flock_game_init` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | frame | H | `seagull_flock_game_init` | `seagull_flock_ai`, `seagull_flock_destroy_on_collision`, `seagull_flock_game_done`, `seagull_flock_hit` | an `'OBJ_'` frame holding `racek.i3d` |
| +0x4 | state | H | `seagull_flock_ai`, `seagull_flock_game_init`, `seagull_flock_hit` | `seagull_flock_ai`, `seagull_flock_hit` | §2a |
| +0x8 | heading | H | `seagull_flock_ai`, `seagull_flock_game_init` | `seagull_flock_ai` | a unit vector with y = 0 at spawn; state 4 lowers its y as gravity |
| +0x14 | aim_point | H | `seagull_flock_ai`, `seagull_flock_game_init` | `seagull_flock_ai` | the point the frame steers toward |
| +0x20 | leg_seconds | H | `seagull_flock_ai`, `seagull_flock_game_init` | `seagull_flock_ai` | 1.0 at spawn; at most 100 |
| +0x24 | next_heading | H | `seagull_flock_ai`, `seagull_flock_game_init` | `seagull_flock_ai` | - |
| +0x30 | unread_30 | U | `seagull_flock_ai` | - | state 1 writes rows 0 and 2 as `(1,0,0)` and `(0,0,1)`; no reader (class census) |
| +0x54 | unused_54 | U | - | - | no access (class census) |
| +0x58 | unread_58 | U | `seagull_flock_game_init` | - | 50, 150, 250, 350, 450 by bird; `seagull_flock_hit` stores 2000 into the record one past the last bird; no reader (class census), Open item 3 |

### 2a. `state` — one bird

| Value | Meaning | Evidence |
|---|---|---|
| 1 | picks a new life: a random box point that passes the wander test, new heading and next heading, frame moved to one unit above the point; then state 2 | `seagull_flock_ai` |
| 2 | flying; the state of a new bird | `seagull_flock_ai`, the record built by `seagull_flock_game_init` |
| 4 | falling after a hit; on landing an explosion, then state 1 | `seagull_flock_hit`, `seagull_flock_ai` |

## 3. Method family

All 13 starts of the unit, a PC-only class, plus a row for each slot that overrides `C_actor` with a shared
stub or fold.

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x004ea1a0 | `crt_static_init_guard_004ea1a0` | the translation unit | - | the unit's `.CRT$XCU` initializer |
| 0x004ea1c0 | `crt_atexit_register_004ea1c0` | the translation unit | - | registers the shared no-op static destructor |
| 0x004ea1d0 | `seagull_flock_ctor` | instance | - | `actor_ctor`, zeroes the collision record, empties the bird vector, vtable, kind `0x17`; its one callsite is `mission_create_actor` case `0x17` |
| 0x004ea220 | `seagull_flock_scalar_deleting_dtor` | instance | - | slot 16 |
| 0x004ea240 | `seagull_flock_dtor` | instance | - | frees the bird vector's buffer, `actor_dtor` |

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | - | slots 14, 15 and 21: no Update, no NetDirect, no DuplicateFrom; a shared stub |
| 0x00462330 | `stub_rettrue_args4_00462330` | - | slots 19 and 20, SaveData and LoadData; a shared stub |
| 0x00477700 | `stub_ret0_cdecl_00477700` | - | slot 22, SaveGameGetSize; a shared stub: the flock is not saved |
| 0x00477710 | `stub_rettrue_args4_00477710` | - | slots 23 and 24, SaveGameSave and SaveGameLoad; a shared stub |
| 0x004ae890 | `actor_subclass_init_require_frame` | - | slot 18, the shared fold (`structs/spike_strip.md` §6) |
| 0x004ea280 | `seagull_flock_game_init` | - | slot 25; builds the box and planes (§5), then five birds (§4) |
| 0x004eb090 | `seagull_flock_game_done` | - | slot 26; `actor_game_done`, releases every bird frame, empties the vector |
| 0x004eb100 | `seagull_flock_destroy_on_collision` | - | slot 30, the `Collision` slot: releases every bird frame, empties the vector, returns false (§8) |
| 0x004eb160 | `seagull_flock_hit` | - | slot 31; the bird nearest an incoming ray, under 1.0, goes to state 4 (§4) |
| 0x004eb310 | `seagull_flock_ai` | - | slot 13; the three bird states (§4) |
| 0x0060b1a0 | `stub_noop_args8_0060b1a0` | - | slot 38; a shared stub |

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x004ec0a0 | `vec3_cross_of_edges` | - | `(a - b) x (c - b)`; called only by `seagull_flock_game_init` |
| 0x004ec120 | `vec_count_stride_0x5c` | - | element count of a `0x5c`-stride vector; also used by `vehicle_set_tyres` and `std_vector5c_insert_at` |
| 0x004ec150 | `stl_fill_n_stride_0x5c` | - | fills n `0x5c`-byte records; the same two other users |

## 4. What it does

One actor owns a whole flock. `seagull_flock_game_init` builds the wander box and the planes (§5), then
loops **five** times (a down counter from 5; the `0x32` next to it is the first value of `unread_58`,
which goes up by 100 per bird). Each pass creates an I3D object, picks a random position inside the box
and a random normalised 2-D heading, loads `racek.i3d`, and pushes one bird in state 2.

`seagull_flock_ai` runs each bird by its state (§2a):

* **State 1.** Pick a random point inside the box. **The point is accepted only when it lies on the
  non-negative side of all `plane_count` planes**; a negative result on any plane draws a new point.
  That is the only sign that terminates with the floor plane `(0, 1, 0, -min_y)`, which every point
  drawn from the box passes. Then two random headings, the frame moved there, state 2.
* **State 2.** The frame is aimed at `aim_point` through its direction slot `+0xc` and moves along its
  forward row at 1 unit per second. While `leg_seconds` lasts, `aim_point` moves along `heading` with
  its y held at 15.0. When it runs out, `next_heading` takes over and a new one is chosen from planes
  2..5: of those whose normal has a dot product above `1e-08` with the heading, the smallest
  `(n·p + d) / (n·h)` below 1000 sets the next `leg_seconds` (clamped to 0..100) and the next heading is
  `-2 × normal` plus up to 0.1 of noise per axis, normalised.
* **State 4.** The heading's y loses `10 × dt` and the frame moves along the heading. When the vertical
  line test hits the ground the frame lands, `game_new_explosion(this, point, 5.0, 5.0, 0, 1, 1, 0x12)`
  fires and the bird returns to state 1. **A shot bird comes back.**

`seagull_flock_hit` (slot 31) acts only when its first argument is 0. It measures each bird's distance
to the incoming ray and sends the nearest one under 1.0 to state 4, returning true. It also stores 2000 at
`[ebx + 0x58]`, where EBX is the loop pointer **after** the loop, one past the last bird and not the bird
it hit (Open item 3). The birds are shootable, and that is what a `C_actor` gives them that a particle
effect would not.

## 5. Where the box and the planes come from

`seagull_flock_game_init` resets the box to an empty one and asks `web_path_find_nearest_node` for the
node nearest the actor's frame with filter mask `8`, which selects the category-2 nodes
(`structs/web_node.md` §2a). From that node it walks the navigation web:

1. the current node must have exactly two connections, or the walk ends;
2. the next node is the neighbour that is not the one the walk came from;
3. the edge from the current node to the next adds one vertical plane, whose normal is the cross
   product of the edge with the vertical, and both endpoints grow the box;
4. the walk ends when it returns to the start node.

The walk indexes the node array with the neighbour's id (`id * 0x1e`), although connections store
ids, not indices (`structs/web_node.md` §1). After the walk, planes 0 and 1 are overwritten with a
floor and a ceiling plane taken from the box's height range; `plane_count` stays as the walk set it.

**No shipped web holds the closed ring the walk is built for.** Walking the node chunk of all 71
shipped `check.bin` files (32 of them carry category-2 nodes) from every category-2 node finds no
closed ring of two-connection nodes; a ring planted in a test file is found by the same walk. A
flock built in a shipped mission would therefore get an open chain of planes, not a closed
outline. What the flock was meant for is not recoverable from the binary or the data.

## 6. Nothing constructs it

Two routes build an actor in this engine. Both dead-end, the same shape as `bendable_plant.md` §6.

**(a) Direct C++ construction.** `seagull_flock_ctor` has **exactly one callsite in the whole
decomp, and it is `case 0x17:` of the entity factory.** No direct creator exists.

**(b) The factory `mission_create_actor(kind)`.** Its eight callsites are enumerated in
`bendable_plant.md` §6 and none produces `0x17`: `mission_open_bin` takes the kind from a scene's
`0xAE22` object kind and `game_load_differences` from a `.chg` difference file, and neither data
set ever names `0x17`; the savegame only replays actors a running game created,
`program_process_command` copies an existing actor's kind, and the three remaining sites push
hard-coded literals (2, 4, 4).

**The scene census.** Special type `0x17` occurs **0 times across all 79 shipped `scene2.bin`**,
`missions/INTRO` included, whose special types are only `{1, 2, 5}`, and none of the 92 `.chg`
difference files holds it either. A per-scene census is a **lower bound**, not a verdict: ten
certainly-live kinds appear in no scene either, because C++ code spawns them. It is (a) that carries
the weight here. The census still reproduces every known figure as a control (`0x1c` = 26 in
MISE06-AUTODROM, `0x8` = 810 across 25 scenes, `0x18` and `0x1e` = 2 each in MISE15-PRISTAV, `0x1d` = 0).

**No asset reference.** `racek.i3d` is the **only** gull/racek string in the entire 2.4 MB
`Game.exe`, and its only reference is `seagull_flock_game_init`. No scene file anywhere references
`racek.i3d`, `racek.4ds`, `gull.i3d` or `gull.4ds`, even though `models/racek.4ds`, `models/gull.4ds`,
`maps/gull.bmp` and `maps/2gull.bmp` all ship as orphan assets, as do two gull animations (§6b).

⇒ **Never constructed in the shipped game:** every route that builds an actor is enumerated, and
none produces kind `0x17` from the shipped code and data. Whether the designers meant to use it is
not recoverable from the binary.

### 6a. What you actually hear, and why that is not this class

In-game observation by the player-tester:

> *"In MISE14-MESTO/racek1-2 and MISE15-PRISTAV/racek1-4 you can hear them but they don't actually
> spawn."*

The scenes agree. All **14** `racek*` objects across MISE14-MESTO, MISE15-PRISTAV, MISE17-VEZENI and
MISE18-PRISTAV are scene object **type 4 with a `0x4060` sound sub-chunk** (`14m_gull1.wav`,
`14m_gull3.wav`, `18_gull2/3/4/5.wav`, `17_bird3..6.wav`). Their special-object record is **type 5 =
`C_detector`**, hosting a script whose entire body is:

```
label 10
  rnd 0, 8
  let flt[0] = flt[0] * 2900
  let flt[0] = flt[0] + 11000
  wait flt[0]
  frm_seton 0, 1
goto 10
```

A random 4-20 second wait, then switch a sound frame on. Nothing in any of the six scripts can
construct an actor. **Seagulls in Mafia are a sound, not a flock.**

The intro is consistent with this too: `intro.avi` is a pre-rendered file, and the in-engine
`missions/INTRO` scene contains no `0x17` and no gull model reference.

**What would overturn this:** any sighting in game of birds *flying in formation*, or a shot that
visibly downs one. That is the same falsifier `bendable_plant` carries.

### 6b. The two gull models

| file | content | used by |
|---|---|---|
| `models/racek.4ds` | 573 bytes: one unrigged mesh `Box02`, texture `2GULL.BMP` (128 x 64, 8-bit) | the flock: `seagull_flock_game_init` opens `racek.i3d`, and the birds move by their frames with no animation |
| `models/gull.4ds` | 11 621 bytes: a rigged mesh `Base` with the joints `Bone02`, `WingR1`..`WingR4`, `WingL1`..`WingL4`, `Tail1`, `TailR`, `TailL`; texture `GULL.BMP` (256 x 256, 24-bit) | nothing |
| `anims/racekLET.5DS`, `anims/racekPLCH.5DS` | two animations of the joints `Base`, `Head`, `WingR1`..`WingR4`, `WingL1`..`WingL4`, `Tail1`, `TailR`, `TailL` | nothing |

Twelve of the thirteen joints the two animations drive are named in `gull.4ds` (`Head` is not; the
model has a `Bone02`), so `gull.4ds` and the two clips are one animated bird, and `racek.4ds` is a
separate low-detail stand-in. The clip names' `LET` and `PLCH` are Czech abbreviations of flight
and gliding. No scene, difference file or table references `gull.4ds` or either animation, and
`Game.exe` contains no string naming them.

## 7. Vtable `0x0063cc98`

| Slot | PC VA | PC name | Note |
|---|---|---|---|
| 13 | `0x004eb310` | `seagull_flock_ai` | - |
| 14, 15 | `0x00461180` | `stub_noop_args4_00461180` | no Update, no NetDirect |
| 16 | `0x004ea220` | `seagull_flock_scalar_deleting_dtor` | - |
| 18 | `0x004ae890` | `actor_subclass_init_require_frame` | the shared fold |
| 19, 20, 21 | `0x00462330`, `0x00462330`, `0x00461180` | `stub_rettrue_args4_00462330` ×2, `stub_noop_args4_00461180` | the shared SaveData / LoadData / DuplicateFrom stubs |
| 22, 23, 24 | `0x00477700`, `0x00477710`, `0x00477710` | `stub_ret0_cdecl_00477700`, `stub_rettrue_args4_00477710` ×2 | the shared SaveGame stubs |
| 25 | `0x004ea280` | `seagull_flock_game_init` | - |
| 26 | `0x004eb090` | `seagull_flock_game_done` | - |
| 30 | `0x004eb100` | `seagull_flock_destroy_on_collision` | `Collision`: §8 |
| 31 | `0x004eb160` | `seagull_flock_hit` | - |

## 8. Slot 30: a contact destroys the flock

`seagull_flock_destroy_on_collision` occupies vtable **slot 30**, the `Collision` slot of this actor
family, and has that slot's signature: six stack arguments (`ret 0x18`) and a bool in AL, like the shared
`stub_retfalse_args24_00482690` that fills slot 30 in the vtables without an override. The engine calls
slot 30 with six arguments from the contact paths of `box_wake_resting_bodies`, `car_detect_wheel`,
`human_movement_intern`, `railway_collide` and `wagon_ai`. The body ignores the arguments, releases every
bird frame, empties the bird vector and returns false: a contact with the flock removes all its birds.
It has **zero `E8` callsites and zero `LAB_` references**; the vtable dataref is its only reference.

## 9. PC vs PS2

**There is no PS2 side**, with evidence in both directions:

* **No PS2 constructor writes `0x17`.** A sweep over every PS2 `__ct__*` symbol for its `+0x50`
  kind write returns 33 rows; none is this kind. The neighbours are accounted for on both sides
  (`0x16` `C_airplane`, `0x18` `C_turnout`).
* **`racek` occurs 0 times in the PS2 decomp** and 0 times in the PS2 symbol table.
* **No PS2 source file fits.** The PS2 `__sinit_` source-file manifest has nothing between
  `C_rail_traffic.cpp`/`C_railway.cpp` and `C_rmorf.cpp`/`C_shvestky.cpp` that could hold it.
* **The PS2 data ships none of the gull assets** of §6b: no file named `racek*` or `gull*` exists
  in it, and the word occurs only as sound-file names inside its scene files.

⇒ Every row in this class is graded from the PC body alone, and **no `PS2 twin:` line is stamped on
any of the 11 own functions.**

## 10. Related

`structs/entity_kind.md` §1 (kind `0x17`) · `structs/bendable_plant.md` §6 (the same shape: nothing
constructs it) · `structs/railway.md` (the unit immediately before this one) · `structs/web_node.md`
(the node categories) · `structs/collision.md` (`tDynamicCollObject`).

## 11. Open items

| Item | State |
|---|---|
| 1. the intro footage | seagulls reportedly appear in the intro; `intro.avi` is pre-rendered and `missions/INTRO` has no `0x17`, so any birds there came from a build or scene we do not have; one viewing would confirm what is on screen |
| 2. `coll` | filled by `seagull_flock_game_init` and never inserted by a function of the class, so what would call slot 30 for a flock is not located |
| 3. the 2000 of `seagull_flock_hit` | it lands one past the last bird, not on the bird hit, and `unread_58` has no reader; what the field was for is not recoverable |
