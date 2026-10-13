# The seagull flock — entity kind `0x17`, PC-only, never constructed

**Created** 2026-08-16 22:24 +0200 · **Modified** 2026-10-11 10:44 +0200

A complete, working actor class that the shipped game never builds.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x17`** |
| PC vtable | `.rdata 0x0063cc98` |
| PC constructor | `seagull_flock_ctor` @ `0x004ea1d0` |
| PC instance size | `0x244` (entity factory `case 0x17`) |
| PC translation unit | `[0x004ea1d0, 0x004ec180)` — 11 function starts, all named |
| Base class | `C_actor` |
| PS2 class | none (§4) |

The class name is **ours**. Neither binary names it: the PS2 has no such class at all, and the PC
has no RTTI. It is called the seagull flock because the only model it loads is `racek.i3d` —
*racek* is Czech for seagull.

## 1. What it does

One actor owns a whole flock. `seagull_flock_game_init` builds the wander box and the clipping
planes from the navigation web (§1c), then loops **0x32 = 50** times: creates an I3D object, picks
a random position inside the box and a random normalised 2-D heading, loads `racek.i3d`, and
pushes a `0x5c`-byte bird record into the vector at `+0x114`/`+0x118` (capacity `+0x11c`).

`seagull_flock_ai` then runs each bird as a two-state wanderer:

* **no target** — pick a random point inside the box, and **reject it if it lies on the positive
  side of any of the `+0x240` clipping planes stored from `+0x120`** (four floats each), retrying
  until one passes. Then randomise a normalised 2-D heading, move the frame there and set state 2.
* **state 2** — steer and bank toward the target using the frame's quaternion.

The plane list is the outline the wander targets must respect; §1c says where it comes from.

`seagull_flock_hit` (vtable slot 31) walks the flock computing each bird's perpendicular distance to
an incoming ray, and downs the nearest one under 1.0 — state **4**, 2000 ms timer. **The birds are
shootable**, and that is the only reason the flock needs to be a `C_actor` rather than a particle
effect.

### 1a. Field map

| offset | meaning |
|---|---|
| `+0x10` | entity kind — `0x17` |
| `+0xf8` .. `+0x10c` | the wander box: min xyz then max xyz |
| `+0x114` / `+0x118` / `+0x11c` | bird vector: begin / end / capacity, stride `0x5c` |
| `+0x120` .. | the clipping planes, 4 floats each |
| `+0x240` | plane count |

Per bird (`0x5c` bytes): `+0x00` the `I3D_frame`, `+0x04` state, `+0x08`.. heading and target, with
the hit timer near the end of the record.

### 1b. Vtable `0x0063cc98`

| slot | function | note |
|---|---|---|
| 13 | `seagull_flock_ai` | - |
| 14, 15 | shared `ret 4` stub | no Update, no NetDirect |
| 16 | `seagull_flock_scalar_deleting_dtor` | - |
| 18 | shared `actor_subclass_init_require_frame` | - |
| 19, 20 | shared SaveData/LoadData stub | - |
| 25 | `seagull_flock_game_init` | - |
| 26 | `seagull_flock_game_done` | - |
| 30 | `seagull_flock_destroy_on_collision` | `Collision`: a contact releases every bird (§3) |
| 31 | `seagull_flock_hit` | - |

The TU's other three starts are helpers: `vec3_cross_of_edges`, `vec_count_stride_0x5c`,
`stl_fill_n_stride_0x5c`.

### 1c. Where the box and the planes come from

`seagull_flock_game_init` resets the box at `+0xf8`..`+0x10c` to an empty one (`+FLT_MAX` minimum,
`-FLT_MAX` maximum) and asks `web_path_find_nearest_node` for the node nearest the actor's frame
with filter mask `8`, which selects the category-2 nodes (`structs/web_node.md` §1a). From that
node it walks the navigation web:

1. the current node must have exactly two connections, or the walk ends;
2. the next node is the neighbour that is not the one the walk came from;
3. the edge from the current node to the next adds one vertical plane, whose normal is the cross
   product of the edge with the vertical, and both endpoints grow the box;
4. the walk ends when it returns to the start node.

The walk indexes the node array with the neighbour's id (`id * 0x1e`), although connections store
ids, not indices (`structs/web_node.md` §1). After the walk, planes 0 and 1 are overwritten with a
floor and a ceiling plane taken from the box's height range; the plane count at `+0x240` stays as
the walk set it.

**No shipped web holds the closed ring the walk is built for.** Walking the node chunk of all 71
shipped `check.bin` files (32 of them carry category-2 nodes) from every category-2 node finds no
closed ring of two-connection nodes; a ring planted in a test file is found by the same walk. A
flock built in a shipped mission would therefore get an open chain of planes, not a closed
outline. What the flock was meant for is not recoverable from the binary or the data.

## 2. Nothing constructs it

Two routes build an actor in this engine. Both dead-end — the same shape as `bendable_plant.md` §5.

**(a) Direct C++ construction.** `seagull_flock_ctor` has **exactly one callsite in the whole
decomp, and it is `case 0x17:` of the entity factory.** No direct creator exists.

**(b) The factory `mission_create_actor(kind)`.** Its eight callsites are enumerated in
`bendable_plant.md` §5 and none produces `0x17`: `mission_open_bin` takes the kind from a scene's
`0xAE22` object kind and `game_load_differences` from a `.chg` difference file, and neither data
set ever names `0x17`; the savegame only replays actors a running game created,
`program_process_command` copies an existing actor's kind, and the three remaining sites push
hard-coded literals (2, 4, 4).

**The scene census.** Special type `0x17` occurs **0 times across all 79 shipped `scene2.bin`**,
`missions/INTRO` included — whose special types are only `{1, 2, 5}` — and none of the 92 `.chg`
difference files holds it either. A per-scene census is a
**lower bound**, not a verdict: ten certainly-live kinds appear in no scene either, because C++ code
spawns them. It is (a) that carries the weight here. The census still reproduces every known figure
as a control (`0x1c` = 26 in MISE06-AUTODROM, `0x8` = 810 across 25 scenes, `0x18` and `0x1e` = 2
each in MISE15-PRISTAV, `0x1d` = 0).

**No asset reference.** `racek.i3d` is the **only** gull/racek string in the entire 2.4 MB
`Game.exe`, and its only reference is `seagull_flock_game_init`. No scene file anywhere references
`racek.i3d`, `racek.4ds`, `gull.i3d` or `gull.4ds` — even though `models/racek.4ds`,
`models/gull.4ds`, `maps/gull.bmp` and `maps/2gull.bmp` all ship, as orphan assets, as do two gull animations (§2b).

⇒ **Never constructed in the shipped game:** every route that builds an actor is enumerated, and
none produces kind `0x17` from the shipped code and data. Whether the designers meant to use it is
not recoverable from the binary.

### 2a. What you actually hear — and why that is not this class

In-game observation by the player-tester:

> *"In MISE14-MESTO/racek1-2 and MISE15-PRISTAV/racek1-4 you can hear them but they don't actually
> spawn."*

The scenes agree. All **14** `racek*` objects across MISE14-MESTO, MISE15-PRISTAV, MISE17-VEZENI and
MISE18-PRISTAV are scene object **type 4 with a `0x4060` sound sub-chunk** — `14m_gull1.wav`,
`14m_gull3.wav`, `18_gull2/3/4/5.wav`, `17_bird3..6.wav`. Their special-object record is **type 5 =
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

### 2b. The two gull models

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

## 3. Slot 30: a contact destroys the flock

`seagull_flock_destroy_on_collision` (`0x004eb100`) occupies vtable **slot 30**, the `Collision`
slot of this actor family, and has that slot's signature: six stack arguments (`ret 0x18`) and a
bool in AL, like the shared `stub_retfalse_args24_00482690` that fills slot 30 in the vtables
without an override. The engine calls slot 30 with six arguments from the contact paths of
`box_wake_resting_bodies`, `car_detect_wheel`, `human_movement_intern`, `railway_collide` and
`wagon_ai`. The body ignores the arguments, releases every bird frame, empties the bird vector
and returns false: a contact with the flock removes all its birds. It has **zero `E8` callsites
and zero `LAB_` references**; the vtable dataref is its only reference.

## 4. PC vs PS2

**There is no PS2 side**, with evidence in both directions:

* **No PS2 constructor writes `0x17`.** A sweep over every PS2 `__ct__*` symbol for its `+0x50`
  kind write returns 33 rows; none is this kind. The neighbours are accounted for on both sides
  (`0x16` `C_airplane`, `0x18` `C_turnout`).
* **`racek` occurs 0 times in the PS2 decomp** and 0 times in the PS2 symbol table.
* **No PS2 source file fits.** The PS2 `__sinit_` source-file manifest has nothing between
  `C_rail_traffic.cpp`/`C_railway.cpp` and `C_rmorf.cpp`/`C_shvestky.cpp` that could hold it.
* **The PS2 data ships none of the gull assets** of §2b: no file named `racek*` or `gull*` exists
  in it, and the word occurs only as sound-file names inside its scene files.

⇒ Every row in this class is graded from the PC body alone, and **no `PS2 twin:` line is stamped on
any of the 11 functions.**

**Related:** `structs/entity.md` §1 (kind `0x17`) · `structs/bendable_plant.md` §5 (the same shape:
nothing constructs it) · `structs/railway.md` (the TU immediately before this one).

## 5. Open items

| item | state |
|---|---|
| the intro footage | seagulls reportedly appear in the intro; `intro.avi` is pre-rendered and `missions/INTRO` has no `0x17`, so any birds there came from a build or scene we do not have — one viewing would confirm what is on screen |
