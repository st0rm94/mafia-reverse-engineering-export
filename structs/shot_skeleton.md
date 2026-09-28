# `C_ShotSkeleton` — the per-limb hit locator (PC prefix `shot_skeleton_*`)

**Created** 2026-08-26 11:13 +0200 · **Modified** 2026-09-17 13:44 +0200

`C_ShotSkeleton` answers one question: *when a shot ray passes near a person, which body part
did it hit?* It does that by wrapping the character's animated skeleton in **ten capsules**,
each stretched between a pair of named joints, and returning the **body-part id** of the
nearest capsule the ray intersects.

| | |
|---|---|
| PC translation unit | `[0x005922c0, 0x005929c0)`, delimited at both ends by its own MSVC static-init pair |
| PC methods | 2 (§4) |
| Size | `0x130`, embedded in `C_human` at `+0x2dc` (§1) |
| Destructor | none — the object holds raw `I3D_frame *` it does not own |

## 1. Where the object lives

🔑 **`C_human` EMBEDS one at `+0x2dc`.** There is no separate allocation and no pointer: the
skeleton is part of every person.

The decomp cannot show this — both methods are `__thiscall`, so the receiver travels in ECX
and Ghidra renders the calls with no receiver argument at all. The ECX load in front of each of the
six callsites of the two methods, read from the disassembly, settles it:

| callsite in | ECX form | reading |
|---|---|---|
| `human_game_init` | `lea ecx,[ebp + 0x2dc]` | EMBEDDED at `C_human+0x2dc` |
| `human_change_model` | `lea ecx,[esi + 0x2dc]` | same |
| `human_ai` | `lea ecx,[edi + 0x2dc]` | same |
| `game_tick_shoot` | `lea ecx,[edi + 0x2dc]` | same |
| `pedestrian_crowd_hit` ×2 | `lea ecx,[esp + 0x48]` / `[esp + 0x50]` | a STACK temporary — the ambient crowd builds one on the fly rather than owning one |

**Corroboration from a different direction.** The object is `0x40` (16 joint handles) +
10 × `0x18` (the capsule table) = **`0x130`** bytes, so it spans `+0x2dc..+0x40c` — and
`+0x40c` is the documented `movement_mode` field. The size and the neighbour agree without
either being derived from the other. This is the same shape of argument that pinned the
`anim_machine` at `C_human+0xbc` (`structs/anim_machine.md` §1).

## 2. Layout

```c
typedef struct C_ShotSkeleton {            /* 0x130 bytes */
    /* +0x00 */ I3D_frame *joint[16];      // [H] resolved at Init from g_shot_skeleton_joint_names
    /* +0x40 */ S_bone     bone[10];       // [H] a verbatim copy of g_shot_skeleton_template_bones
} C_ShotSkeleton;

typedef struct S_bone {                    /* 0x18 bytes */
    /* +0x00 */ uint32_t joint_a;          // [H] index into joint[]
    /* +0x04 */ uint32_t joint_b;          // [H] index into joint[]
    /* +0x08 */ float    t_start;          // [H] parametric start along the a->b segment
    /* +0x0c */ float    t_end;            // [H] parametric end
    /* +0x10 */ float    radius;           // [H] capsule radius, world units
    /* +0x14 */ uint32_t body_part;        // [H] 1..6, the value TestCollision returns
} S_bone;
```

The bone table is COPIED into every instance rather than shared, so an instance could in
principle be retuned per character; nothing located does so.

## 3. The joint names and the capsule table (decoded from `Game.exe`)

`g_shot_skeleton_joint_names` (`0x0064dfc0`) is 16 `char *`, resolved at Init through the
model's `I3D_frame` find-by-name virtual with a `0xffff` search mask:

| # | name | | # | name | | # | name |
|---|---|---|---|---|---|---|---|
| 0 | `r_hand` | | 6 | `l_hand` | | 12 | `base` |
| 1 | `r_elbow` | | 7 | `l_elbow` | | 13 | `back1` |
| 2 | `r_arm` | | 8 | `l_arm` | | 14 | `back3` |
| 3 | `r_foot` | | 9 | `l_foot` | | 15 | `neck` |
| 4 | `r_shin` | | 10 | `l_shin` | | | |
| 5 | `r_thigh` | | 11 | `l_thigh` | | | |

`g_shot_skeleton_template_bones` (`0x0064e000`) is the ten capsules:

| # | from → to | t_start | t_end | radius | body part |
|---|---|---|---|---|---|
| 0 | `r_hand` → `r_elbow` | -0.5 | 1.2 | 0.06 | **1** right arm |
| 1 | `l_hand` → `l_elbow` | -0.5 | 1.2 | 0.06 | **2** left arm |
| 2 | `r_elbow` → `r_arm` | 0.0 | 1.3 | 0.08 | **1** right arm |
| 3 | `l_elbow` → `l_arm` | 0.0 | 1.3 | 0.08 | **2** left arm |
| 4 | `r_foot` → `r_shin` | -0.3 | 1.1 | 0.10 | **3** right leg |
| 5 | `l_foot` → `l_shin` | -0.3 | 1.1 | 0.10 | **4** left leg |
| 6 | `r_shin` → `r_thigh` | 0.1 | 1.2 | 0.12 | **3** right leg |
| 7 | `l_shin` → `l_thigh` | 0.1 | 1.2 | 0.12 | **4** left leg |
| 8 | `back1` → `back3` | -0.5 | 1.2 | 0.17 | **5** torso |
| 9 | `back3` → `neck` | 0.5 | 3.0 | 0.12 | **6** head / neck |

Two things worth noticing. The extents are deliberately **not** `[0, 1]`: a capsule usually
over-runs its bone at both ends (`-0.5 .. 1.2`) so the gaps between limbs stay covered. And
the head capsule runs `0.5 .. 3.0` along `back3 → neck`, i.e. it starts halfway up the neck
bone and extends to **three times** its length — that is how a head with no head joint gets a
hit volume.

The body-part ids are what the shooting code consumes; the class itself never interprets them.

## 4. The two methods

| PC VA | PC name | what it does |
|---|---|---|
| `0x005922f0` | `shot_skeleton_init` | resolve all 16 joints from the model, then copy the 10-capsule template in |
| `0x00592350` | `shot_skeleton_test_collision` | test a ray against all 10 capsules; return the body part of the nearest hit, or 0 |

`shot_skeleton_test_collision(start, dir, best)` walks the capsules in order. For each it
refreshes both joints' world matrices (behind the `I3D_frame` `& 0x20` dirty flag), builds the
bone axis as `joint_b.pos - joint_a.pos`, crosses it with the ray direction, normalises with a
degenerate-safe ladder (if `|len²| < 1e-08`, snap to ±1 on whichever component is non-zero),
and offsets by the capsule radius to get the two side planes. A candidate is kept only when
its squared parameter beats the running best — `param_4` seeds that best, so a caller can cap
the search distance. The return is `bone[i].body_part`, so **0 means no hit**.

The `+0.01 * i` term added to the hit parameter inside the loop is a tie-break bias, not a
distance: it makes later capsules lose to earlier ones at equal range, which is what keeps a
torso hit from stealing an arm hit.

## 5. PC vs PS2

The class name, the two method names and both data-table names come from the PS2 symbol
table; the PC decomp could not have supplied any of them. The PC prefix drops the `C_` and
lowercases the rest.

| PC name | PS2 twin | PS2 VA | Status |
|---|---|---|---|
| `shot_skeleton_init` | `Init__14C_ShotSkeletonFP9I3D_frame` | `0x002e73e0` | CONFIRMED |
| `shot_skeleton_test_collision` | `TestCollision__14C_ShotSkeletonFRC8S_vectorRC8S_vectorf` | `0x002e74c0` | CONFIRMED |
| `g_shot_skeleton_joint_names` | `jointName` | - | CONFIRMED |
| `g_shot_skeleton_template_bones` | `vzoroveKosti` | - | CONFIRMED |

**2 PS2 methods, 2 PC twins — the class is CLOSED.** The source file is `g_shot.cpp`, from the
PS2 `__sinit_` manifest. No `C_ShotSkeleton` destructor exists on either build.

`vzoroveKosti` is Czech for *"template bones"*, which is exactly what the table is — the PC
name states that rather than transliterating.

**Build differences in the bodies, all cosmetic:**
* `Init` resolves joints through `I3D_frame` vtable slot `+0x38` on PC and `+0x50` on PS2 —
  the same virtual at a different index, since the two builds' `I3D_frame` vtables do not
  slot-align.
* `TestCollision` refreshes the second joint with the flag-guarded `UpdateWMatrixProc` on PC
  and the unguarded `UpdateWMatrix` on PS2. Same effect, one redundant test.
* PS2 does the vector normalise on the VU (`vrsqrt`/`vmulq` intrinsics) where PC uses x87.
  The degenerate ladder either side of it is identical.

## 6. Related

`structs/human.md` §4d (the embedding, from the `C_human` side) ·
`structs/anim_machine.md` §1 · `ghidra/Game-ghidra.exe.functions.md` (the function registry).

## 7. Open items

| item | state |
|---|---|
| consumer of the returned body-part id | `human_ai` and `game_tick_shoot` both call `TestCollision`; what they do with 1..6 (damage scaling, a hit reaction, a sound variant) is unread |
| how `pedestrian_crowd_hit` fills its stack instance | it must fill it itself; whether it calls `shot_skeleton_init` on the temporary or copies a prepared one is unchecked |
