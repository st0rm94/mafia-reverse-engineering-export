# `C_box` — the PC-only legacy rigid-body box

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-09-24 12:22 +0200

A `C_actor` subclass carrying **its own complete rigid-body solver**: force accumulation,
semi-implicit integration into a double-buffered state, an OBB-vs-OBB separating-axis test, a
30-slot warm-started contact cache, sequential impulse resolution, write-back to the I3D frame,
plus savegame, recorder tracks and network replication. At ~36 KB across 55 function starts it
is the largest single-class translation unit in the actor family.

| | |
|---|---|
| PC prefix | **`box_`** |
| Base class | **`C_actor`** (`structs/actor.md`) directly — *not* `C_PhThing`; this class predates the physics base and duplicates its job (§5c) |
| PC kind (`+0x10`) | **`9`** |
| PC vtable | `.rdata 0x0063b980` (39 slots) |
| PC translation unit | `[0x004564a0, 0x0045f130)` — **55 function starts**, bounded at both ends by a CRT static-init/atexit pair |
| PS2 class | **none** — the class does not exist in the PS2 build (§5a) |

Because there is no PS2 body to diff, **every method below is graded STRUCTURAL and its PS2-twin
cell is `-`**: a CONFIRMED tag would assert a body agreement that cannot exist.

**The class NAME is inferred; the PREFIX is not.** `box_` rests on the developers' own string:
`recorder_new_track` names this kind's recorder tracks `box%d` (`s_box_d_0064c9e0`). `C_box` as
the class name rests on the link-order argument (§5b); if the name were falsified the prefix
would still stand.

`C_box` is not dead code: it has a live vtable, a kind value the recorder formats, a savegame
chunk id, and a matching network sender and receiver. Whether any mission instantiates it is
open (§7).

## 1. Field layout

Confidence tags: **[H]** byte-proven from a writer *and* a reader · **[M]** one side traced ·
**[U]** written but no consumer traced.

The two backbones of this table are `box_init` (which seeds every tunable with a literal)
and `box_accumulate_forces` (which consumes mass, inverse mass, damping and inertia in one
expression), so the physics band has two independent witnesses throughout.

```c
typedef struct C_box {                    /* extends C_actor */
    /* +0x00  */ void      *vptr;             // 0x0063b980
    /* ...    */ C_actor    base;
    /* +0x10  */ int        kind;             // [H] = 9
    /* +0x20  */ char       is_remote;        // [H] C_actor base; gates the replication mirror
    /* +0x24  */ S_vector   pos;              // [H] C_actor base, refreshed by box_game_init
    /* +0x68  */ I3D_frame *frame;            // [H] bound scene frame; Init rejects type != 9
    /* +0x70  */ float      half_extent[3];   // [H] written by box_recompute_dimensions_from_frame
    /* +0x7c  */ int        solver_phase;     // [M] set to 2 by both contact passes
    /* +0x80  */ int        state_read_idx;   // [H] index of the live state buffer
    /* +0x84  */ int        state_write_idx;  // [H] index of the back buffer
    /* +0xe8  */ C_box     *resting_on[];     // [H] boxes stacked on this one
    /* +0xfc  */ int        resting_count;    // [H] cleared by box_wake_resting_bodies
    /* +0x104 */ int        sleep_counter;    // [H] set to 5 on any external impulse
    /* +0x110 */ float      inv_mass;         // [H] every impulse denominator
    /* +0x114 */ float      mass;             // [H] the gravity term is g * mass
    /* +0x124 */ float      inertia_diag[3];  // [H] +0x124/+0x128/+0x12c, angular damping
    /* +0x130 */ S_box_state state[2];        // [H] stride 200 (0xc8); see §1a
    /* +0x388 */ float      linear_damping;   // [H] default 0.32 (0x3ea3d70a)
    /* +0x38c */ float      angular_damping;  // [H] default 0.5
    /* +0x390 */ S_vector   force_accum;      // [H] box_add_impulse_at_point -> box_accumulate_forces
    /* +0x39c */ S_vector   torque_accum;     // [H] r x F, same producer/consumer pair
    /* +0x3a8 */ char       impulse_pending;  // [M] cleared by box_add_impulse_at_point
    /* +0x3b0 */ int        step_counter;     // [M] bumped by box_integrate_state
    /* +0x3b4 */ S_vector   contact_pos[30];  // [H] the contact ring; see §1b
    /* +0x51c */ float      contact_depth[30];// [H] matched to 0.01 by the slot acquire
    /* +0x60c */ int        contact_cursor;   // [H] ring head, modulo 0x1e
    /* +0x610 */ int        contact_count;    // [H] live entries
    /* +0x674 */ S_vector   contact_normal[]; // [M] stride 0xc, read by box_resolve_contacts
    /* +0x9f4 */ int        resolve_scratch;  // [U] cleared by both contact passes
    /* +0x9f8 */ float      resolve_best;     // [U] seeded to -1.0 by box_collect_contacts
    /* +0xb68 */ char       awake;            // [H] 0 = asleep; set by every wake path
    /* +0xb69 */ char       just_woke;        // [H] read by box_track_record_key
    /* +0xc34 */ tDynamicCollObject dyn_coll; // [H] 0x12 dwords; collision_dyn_erase target
    /* +0xc7c */ I3D_frame *dyn_coll_frame;   // [H] the `dyn*` frame it was built from
    /* +0xc80 */ float      tunable_a;        // [H] default 50.0
    /* +0xc84 */ float      tunable_b;        // [H] default 0.4
    /* +0xc88 */ float      tunable_c;        // [H] default 2.5
    /* +0xc8c */ float      restitution;      // [H] default 0.6; every solver uses -(e+1)
    /* +0xc90 */ char       tensor_valid;     // [M] gates box_build_inverse_inertia_tensor
    /* +0xc94 */ float      bound_radius;     // [H] |half_extent|, from box_game_init
    /* +0xc9c */ S_vector   net_pos;          // [H] replication band, see §1c
    /* +0xca8 */ S_vector   net_lin_vel;      // [H]
    /* +0xcb4 */ S_quat     net_rot;          // [H]
    /* +0xcc4 */ S_quat     net_rot_prev;     // [M] seeded equal to net_rot by box_game_init
    /* +0xcd4 */ int        net_pending;      // [M] cleared by the sender
    /* +0xcd8 */ uint       net_flags;        // [H] bit map in §1c
    /* +0xcdc */ int        net_owner;        // [U] written by Init, no reader traced
} C_box;                                       /* >= 0xce0 bytes */
```

### 1a. The double-buffered state record (`+0x130`, stride 200 = 0xc8)

Two of these exist. `box_integrate_state` reads the one indexed by `+0x80` and writes the
one indexed by `+0x84`; `box_write_state_to_frame` publishes whichever it is handed. That is
what lets the network layer hold a previous state to interpolate from while the solver runs.

| offset in record | field | evidence |
|---|---|---|
| `+0x00` | `S_vector pos` (centre of mass) | every `r = point - com` in the three solvers |
| `+0x0c` | `float basis[16]` (the 4×4 orientation) | copied wholesale by `box_game_init`; clamped by `box_clamp_state_matrix` |
| `+0x4c` | `S_vector lin_vel` | written by `box_apply_impulse_pair_at_point` |
| `+0x58` | `S_vector ang_vel` (local) | same writer, second half |
| `+0x64` | `S_vector force` | destination of `box_accumulate_forces` |
| `+0x70` | `S_vector torque` | same |
| `+0xbc` | `S_vector ang_vel_world` | recomputed through `S_vector::operator*` after every impulse |

### 1b. The contact cache — 30 slots, warm-started

`box_contact_slot_acquire` searches the ring `(cursor + i) % 0x1e` for a contact **within
1e-4 of the given position whose stored depth is within 0.01 of the given depth** and returns
that slot; only if none matches does it reuse an all-zero slot or append. Reusing the slot
across frames is what makes a stack of boxes settle instead of jittering — the standard
warm-starting trick for a sequential-impulse solver. `box_contact_slot_release` zeroes a slot
by index.

### 1c. The replication band `+0xc9c..+0xcd8`

Written by `box_game_init` (seed), `box_write_state_to_frame` (mirror, only when `+0x20` says
this machine is authoritative) and `box_apply_net_event_msg` (remote update); read by
`box_net_apply_remote_state` for dead reckoning.

| bit of `+0xcd8` | set by | meaning |
|---|---|---|
| `0x1` | `box_on_collision` | collided this frame — state is dirty, broadcast it |
| `0x2` | `box_apply_net_event_msg` type `0x13` | payload word 1 == 1 (§4) |
| `0x8` | `box_apply_net_event_msg` type `0x1b` | a remote state arrived |
| `0x10` | same, from the payload's sleep byte | remote body is asleep (cleared when awake) |
| `0xfffffffe` (OR) | `box_init` | all bits but 0 set at construction |

## 2. Method coverage — 55/55 function starts

Every row is STRUCTURAL: PC body read plus a byte-proven anchor (a vtable slot role transferred
across three sibling classes, a field-offset + callsite census, or a proven internal call chain).
The four CRT boundary functions are not listed.

| PC VA | PC name | vtable slot | role |
|---|---|---|---|
| 0x004564a0 | `box_static_init_gravity` | - | TU static init: gravity `(0, -9.81, 0)` |
| 0x004564c0 | `box_wake_resting_bodies` | - | wake the stack above this box |
| 0x00456530 | `box_add_impulse_at_point` | - | public "push this box" entry |
| 0x00456630 | `box_get_contact_state_at_point` | - | public query used by `car_get_collision_info` ¹ |
| 0x00456720 | `box_solve_impulse_vs_box` | - | two-body contact solver |
| 0x00456b00 | `box_solve_impulse_vs_world` | - | one-body solver vs static geometry |
| 0x00456e60 | `box_apply_impulse_pair_at_point` | - | applies a resolved impulse to the state |
| 0x00456fb0 | `box_on_collision` | **30** | Collision |
| 0x00457350 | `box_hit` | **31** | Hit |
| 0x00457540 | `box_accumulate_forces` | - | gravity + damping into the accumulators |
| 0x00458000 | `vec3_mul_componentwise` | - | foreign math helper ¹ |
| 0x00458030 | `matrix3_rotate_by_angular_velocity` | - | foreign math helper ¹ |
| 0x004580a0 | `matrix4_from_transposed_basis` | - | foreign math helper ¹ |
| 0x00458130 | `matrix3_orthonormalize` | - | foreign math helper ¹ |
| 0x004585b0 | `box_integrate_state` | - | the integrator |
| 0x00458830 | `box_solve_impulse_vs_static` | - | third solver variant |
| 0x00458a60 | `box_contact_slot_acquire` | - | contact cache, acquire |
| 0x00458bc0 | `box_contact_slot_release` | - | contact cache, release |
| 0x00458c40 | `stub_noop_cdecl_00458c40` | - | folded stub |
| 0x00458c60 | `stub_noop_cdecl_00458c60` | - | folded stub |
| 0x00458c70 | `box_build_inverse_inertia_tensor` | - | world-space inverse inertia |
| 0x004592f0 | `obb_vs_obb_separating_axis_test` | - | foreign geometry helper (15-axis SAT) ¹ |
| 0x00459820 | `box_generate_contacts_vs_box` | - | contact generation |
| 0x00459cf0 | `vec3_equals` | - | shared helper, 3 unrelated modules ¹ |
| 0x00459d30 | `box_collect_contacts` | - | fills the resolve scratch list (§3) |
| 0x0045a8c0 | `box_resolve_contacts` | - | consumes it and applies impulses |
| 0x0045ae00 | `box_ctor` | - | ctor; kind 9, vtable 0x0063b980 |
| 0x0045ae80 | `box_scalar_deleting_dtor` | **16** | MSVC sdd |
| 0x0045aea0 | `identity_thunk` | - | shared identity thunk |
| 0x0045aeb0 | `box_init` | **18** | Init; frame type gate == 9 |
| 0x0045afb0 | `box_recompute_dimensions_from_frame` | - | clamps tunables, derives half-extents |
| 0x0045b300 | `box_dtor` | - | dtor |
| 0x0045b360 | `box_build_dyn_collision_from_frame` | - | the `dyn*` frame-name convention ² |
| 0x0045baf0 | `box_game_init` | **25** | GameInit |
| 0x0045c0d0 | `box_game_done` | **26** | GameDone |
| 0x0045c0f0 | `box_save_data` | **19** | SaveData, chunk id `0x11` |
| 0x0045c150 | `box_load_data` | **20** | LoadData |
| 0x0045c240 | `box_ai_tick` | **13** | the physics tick — the TU's apex |
| 0x0045dcf0 | `box_write_state_to_frame` | - | publish to the I3D frame |
| 0x0045ddf0 | `box_track_create` | **9** | `recorder_add_track(0x28, 0x28, 0x30)` |
| 0x0045de10 | `box_track_record_key` | **0** | record a key; `-1` = nothing recorded |
| 0x0045e1d0 | `box_clamp_state_matrix` | - | clamp a 4×4 state matrix to ±1000 |
| 0x0045e220 | `box_post_state_net_event_msg` | - | **sender**, message type `0x1b` (§4) |
| 0x0045e280 | `box_apply_net_event_msg` | **38** | **receiver**, types `0x13` + `0x1b` (§4) |
| 0x0045e5a0 | `box_net_apply_remote_state` | **15** | `NetDirect` — dead reckoning |
| 0x0045f000 | `i3d_frame_set_position` | - | foreign I3D helper |
| 0x0045f040 | `i3d_frame_set_orientation` | - | foreign I3D helper |
| 0x0045f090 | `i3d_frame_get_scale` | - | foreign I3D helper, 3 unrelated callers ¹ |
| 0x0045f0f0 | `stub_zero_vec3_out` | **4** | ICF fold, 2 vtables — owned by no class ¹ |
| 0x0045f110 | `stub_zero_quat_out` | **5** | ICF fold, 2 vtables — owned by no class ¹ |

¹ See §2a. ² `box_build_dyn_collision_from_frame` matches frame names against `s_dyn_0064b5e8`
(`dyn`, a 3-character compare).

### 2a. The foreign symbols inside the TU

Graded by offset first, callsite census second, each member of a family separately; adjacency
proves nothing in either direction.

| symbol | verdict | what decides it |
|---|---|---|
| `box_get_contact_state_at_point` | **C_box method** despite an external caller | indexes `this+0x130 + this[0x80]*200`, the class's own state block |
| `vec3_mul_componentwise` · `matrix3_rotate_by_angular_velocity` · `matrix4_from_transposed_basis` · `matrix3_orthonormalize` · `obb_vs_obb_separating_axis_test` | **no class** | no receiver at all; pure value transforms |
| `vec3_equals` | **no class** | 5 callsites across 3 unrelated modules incl. `car_net_apply_remote_state` |
| `i3d_frame_get_scale` | **I3D, not C_box** | 3 callers in 3 modules; sits beside its `i3d_frame_set_*` siblings |
| `stub_zero_vec3_out` / `stub_zero_quat_out` | **no class — ICF fold** | each occupies the **same slot in two vtables** (`0x0063b720` C_actor_shot and `0x0063b980` C_box): the linker merged two classes' identical overrides (`structs/actor_track.md` §2b) |

### 2b. The network pair verifies itself with no cross-build side

Same shape as `structs/bottle.md` §2a, and the same grading argument. `box_ai_tick` and
`box_apply_net_event_msg` **both** re-broadcast through `box_post_state_net_event_msg` under
the byte-identical host-role gate `g_net_session+0x100 == 7`. Two independent code paths
agreeing on a non-obvious guard is usable evidence when there is no PS2 body to diff.

## 3. The solver's global scratch arrays

The TU owns 16 globals in one contiguous band that tiles with no gaps (only two 4-byte alignment
pads):

| address | name | type | role |
|---|---|---|---|
| `0x00655378` | `g_box_resolve_count` | `int` | resolve-list count, capped at 0x28 (40) by `box_collect_contacts`; loop bound in `box_resolve_contacts` |
| `0x00655380` | `g_box_resolve_slot_index` | `int[40]` | contact-ring slot index of each queued contact |
| `0x00655420` | `g_box_resolve_partner_index` | `int[40]` | partner index of the same entry |
| `0x006554c0` | `g_box_depenetration_offset` | vec3 | `depth × contact_normal`; one writer (`box_collect_contacts`, gated `-0.4 <= depth`), one reader (`box_ai_tick`, which subtracts it from the body position) |
| `0x006554d0` | `g_box_contact_dyn_object` | `ptr[30]` | the hit `tDynamicCollObject` (record `+0x1c`), `0` for a static hit |
| `0x00655548` | `g_box_contact_plane_d` | `float[30]` | the contact plane **constant `d`** (record `+0x0c`) |
| `0x006555c0` | `g_box_contact_normal` | interleaved vec3 ×29 | the contact plane's **normal** (record `+0x00`) |
| `0x0065571c` | `g_box_scratch_count` | `int` | shared index of the three contact arrays |
| `0x00655720` | `g_box_gravity` | vec3 | `(0, -9.81, 0)` (`0xc11cf5c3`), seeded by `box_static_init_gravity`; `× mass` in `box_accumulate_forces`, also read by `box_ai_tick` (`× 0.1`) and `box_net_apply_remote_state` |

**The contact arrays.** `box_ai_tick` calls `collision_test_sphere_slide` and copies each
returned contact record into the three parallel arrays while `g_box_scratch_count < 0x1d`, so at
most **29** entries are written. The normal array has room for exactly 29 vec3s before the
count; the other two have room for 30. Ghidra shows `plane_d` and `dyn_object` as the raw
addresses `0x00655544` / `0x006554cc` because they are indexed with the post-increment count,
i.e. the base is element `[-1]`.

The consumer fixes the meaning: the read loop computes

```
n.x*p.x + n.y*p.y + n.z*p.z + plane_d[i] < 0.01
```

over the box's own centre — the plane equation `dot(n, p) + d`. The contact POINT exists at
record `+0x10`, and `box_ai_tick` never copies it. The records are produced by `collision_slide_sphere`,
a PC-only solver whose only consumer is this class.

## 4. Replication — types `0x1b` and `0x13`

| type | descriptor | sender | payload | `box_apply_net_event_msg` |
|---|---|---|---|---|
| `0x1b` | `g_event_msg_vt_physics_state` (`.rdata 0x0063ba90`) | `box_post_state_net_event_msg`, and `physical_object_post_state_net_event_msg` (`structs/physical_object.md` §5c) | actor-state base + 16-byte quaternion at `+0x1c` + three velocity floats quantised through `net_encode_fixed16` + flag byte at `+0x38` | fills the replication band, sets bit `0x8`, bit `0x10` from the sleep byte |
| `0x13` | `g_event_msg_vt_actor_act_state` (`.rdata 0x0063b6dc`) | `actor_set_act_state`, host role only | `{actor guid, act state}` | sets bit `0x2` when the act state is 1 |

Type `0x1b` is the shared **rigid-body state update**, not `C_box`'s own: `C_box` and `C_PhThing`
install the same descriptor, and `event_msg_physics_state_ctor` (`0x005a1860`) takes the type
as a parameter. Its serialize/deserialize pair is therefore named for the payload:
`event_msg_physics_state_serialize` / `_deserialize`. Type `0x13` is the generic act-state
change every actor posts; `bottle_apply_net_event_msg` handles it too. A PC-only class
replicating through a PC-only subsystem is consistent with §5a. Full write-up:
`structs/net_event_msg.md`.

## 5. PC vs PS2

### 5a. The class is absent from the PS2 build, proven by the port's own file manifest

The PS2 ELF carries its source-file list in the clear: **173 `__sinit_<file>.cpp` symbols**.
The relevant run reads

```
... C_blood.cpp   C_bottle.cpp   C_box_new.cpp   C_bridge.cpp   C_car.cpp ...
```

with **no `C_box.cpp`**. The file was not renamed and not inlined — it was never compiled
into the port. Three weaker checks agree: no `5C_box` mangled tag in the symbol table, no
`__vt__5C_box` among the vtable data symbols, and no PS2 constructor writes `9` to its kind
field `+0x50`. The PC has **more** code than the PS2 symbol table accounts for — the same
direction as `C_bottle`'s network layer.

### 5b. Why the name `C_box` is the inference it is

Three things point at it and nothing points away:

1. **Link order.** Aligning the vtables in `.rdata` with the PS2 source-file order places this
   vtable strictly between `C_bottle`'s `0x0063b870` and `C_box_new`'s `0x0063baa0`, and the
   PS2 manifest shows the sort is case-insensitive by file name. `C_box.cpp` is the only
   plausible name that sorts between `C_bottle.cpp` and `C_box_new.cpp` (a prefix sorts before
   its extension).
2. **The successor is literally called `C_box_new`** — a name that presupposes a `C_box`.
3. **The developers' own word.** `recorder_new_track` labels this kind's tracks `box%d`.

### 5c. What the PS2 replaced it with

`C_box_new` (`structs/box_new.md`) does the same job in **12 functions instead of 55**,
because all the dynamics moved into the shared base `C_PhThing`
(`structs/physical_object.md`). The refactor is visible in the numbers: this class carries
three bespoke impulse solvers, its own SAT and its own contact cache; `C_box_new` carries a
six-field descriptor and hands it to `InitBox` / `InitWheel`.

## 6. Related

`structs/box_new.md` (the class that replaced it, and the one the PS2 kept) ·
`structs/physical_object.md` (`C_PhThing`, the physics base `C_box` does **not** use) ·
`structs/actor.md` / `structs/actor_track.md` (the base class and the slot 0–12 recorder
interface `box_track_create` / `box_track_record_key` implement) · `structs/bottle.md` (the
neighbouring class, and the template for the network-pair argument in §2b) ·
`structs/net_event_msg.md`.

## 7. Open items

| item | state |
|---|---|
| `+0x9f4` / `+0x9f8` / `+0xcdc` | [U] — written, no consumer traced |
| `+0x674` contact-normal array | stride `0xc` is measured but the element count is not; `box_resolve_contacts` indexes it from the resolve list |
| is `C_box` instantiated by any mission? | needs an in-game observation or a mission-file scan; the `dyn*` frame-name convention of `box_build_dyn_collision_from_frame` is what to search the level data for |
