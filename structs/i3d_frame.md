# `I3D_frame` — the LS3D scene node (and `I3D_scene`, which derives from it)

**Created** 2026-08-30 19:07 +0200 · **Modified** 2026-09-17 22:52 +0200

`I3D_frame` is the LS3D engine's **scene-graph node**: a transform, a visual, a name, a sector
membership and a four-character type tag. Every `C_actor` owns one at `actor+0x68`
(`folded_get_dword_0x68`, 253 callsites), and the mission script VM can hold them directly in its
`flt`-file sibling, the **frame register file** (`C_program+0x50`, typed by the signature
`SetFrameReg__9C_programFiP9I3D_frame`).

**The class itself is not in `Game.exe`.** `I3D_frame` is implemented in **`ls3df.dll`**, which
exports only three of its methods (`?UpdateWMatrixProc@I3D_frame@@AAEXXZ`, `?UpdateHRBoundVol@…`,
`?UpdateHRBoundVolW@…`). What lives in `Game.exe` is the set of **header inlines** MSVC out-lined
into whichever translation unit used them. That is why this file is a map of ACCESSORS and OFFSETS
and not a method table: there is no vtable to dump and no PC symbol to match
(`basic/artifacts/dll.md`).

Confidence: **[H]** byte-read / body-read · **[M]** inferred · **[U]** not observed.

## 1. Layout

```c
typedef struct I3D_frame {
    /* +0x40 */ S_matrix world;         // [H] world matrix; +0x40..+0x48 is its TRANSLATION row,
                                        //     i.e. the world POSITION (actor_get_world_position),
                                        //     and +0x30..+0x38 the FORWARD row of the PREVIOUS
                                        //     matrix block (i3d_frame_get_forward_normalized)
    /* +0x50 */ S_matrix local_rot3;    // [H] the local 3x3, rebuilt lazily from rot * scale
    /* +0x80 */ S_vector pos;           // [H] local position (i3d_frame_set_position_xyz_ptrs)
    /* +0x90 */ S_vector scale;         // [H] local scale (i3d_frame_set_scale / _get_scale_ptr)
    /* +0x9c */ S_quat   rot;           // [H] rotation quaternion, 4 floats +0x9c..+0xa8
    /* +0xac */ uint     flags;         // [H] validity + dirty bits, see §2
    /* +0x100*/ char    *name;          // [H] null -> g_empty_string (i3d_frame_get_name)
    /* +0x110*/ int      visual_type;   // [H] 1 = model ('OBJ_'), 4 = SOUND, 5 = SECTOR,
                                        //     6 = the +0x140 block, 9 = a model/driver frame,
                                        //     0xe = AREA (a trigger volume). See §3a.
    /* +0x11c*/ int      child_count;   // [H] i3d_frame_get_child_count_ptr
    /* +0x120*/ I3D_frame *parent;      // [H] the PARENT frame, null for a root frame; see §3b
    /* +0x13c*/ void    *sector;        // [H] the visibility cell; frame->+0x13c == scene->+0x13c
                                        //     is the "same sector as the camera" predicate
    /* +0x140*/ int      _140;          // [H] on an I3D_camera this is the FOV
                                        //     (i3d_camera_get_fov_ptr); on a visual-type-6 frame
                                        //     it is a 0x18-byte block read out of the scene .bin
                                        //     (i3d_frame_set_field_0x140_block) - see §10
    /* +0x154*/ char    *model_name;    // [M] null -> g_empty_string (i3d_frame_get_model_name)
    /* +0x18c*/ int      sound_volume;  // [H] reached only when visual_type == 4
                                        //     (i3d_sound_get_volume_ptr)
    /* +0x1d4*/ uint     fourcc;        // [H] class tag, see §3
    /* +0x1e0*/ I3D_mesh_object *mesh;  // [H] the MESH OBJECT, valid when type 1 / 'OBJ_'; see §9
    /* +0x220*/ /* 0x38-byte records */ // [M] reached only under the 'LMAP' tag
} I3D_frame;
```

### 1a. `I3D_scene` derives from `I3D_frame`

The PS2 `I3D_scene` class ships **eight-byte stub overrides** of exactly the `I3D_frame` virtuals a
scene root has no use for — `LinkTo`, `SetDir`, `SetName`, `SetOn`, `Update`, `SetCallback`,
`Duplicate` — which is what a derived class that suppresses its base's API looks like. That is why
the scene and a frame can be compared on the *same* offset `+0x13c`.

```c
typedef struct I3D_scene /* : I3D_frame */ {
    /* … I3D_frame … */
    /* +0x17c */ I3D_frame *active_camera; // [M] i3d_scene_get_active_camera_ptr
    /* +0x210 */ I3D_frame *root_frame;    // [H] i3d_scene_get_root_frame_ptr
} I3D_scene;
```

**The singleton is `*(g_act_mission + 0x10)`**, identified by `i3d_scene_set_frame_sector_pos`,
whose receiver is that pointer. Its vtable slot `+0xbc` is
`SetFrameSectorPos(I3D_frame *, const S_vector &)`.

## 2. The flag word at `+0xac`

| bit | meaning | evidence |
|---|---|---|
| `0x1` | **[M]** "on" / enabled | `i3d_frame_is_on` returns it; the PS2 class has `SetOn__9I3D_frame`. No consumer read — the measurement is exact, the noun is not |
| `0x4` | local matrix valid | `i3d_frame_get_scaled_rot_matrix_ptr` returns early when set |
| `0x8` | rotation quaternion valid | set by `i3d_frame_get_rotation_ptr` and `i3d_frame_set_orientation` |
| `0x10` | rotation source is the matrix, not the quaternion | selects which branch rebuilds the 3x3 |
| `0x20` | world matrix valid | cleared work is done by `I3D_frame::UpdateWMatrixProc` |
| `0x80` (low byte sign) | guard tested by `i3d_frame_set_position_xyz_ptrs` before writing | |

Two composite rewrites recur across the engine and are useful as **shared magic constants** when
matching functions across builds:

* `flags &= 0xfffffedf; flags |= 0x40000000;` — **position changed**
  (`i3d_frame_set_position_xyz_ptrs`, and the whole `C_human` WeaponEffect family).
* `flags &= 0xfffffecb; flags |= 0x40000008;` — **rotation / scale changed**
  (`i3d_frame_set_scale`, `drop_in_weapon` slot 18).

## 3. The FourCC tag at `+0x1d4`

The frame's class tag, read directly at many sites as `param_1[0x75]` and through
`i3d_frame_get_tag_ptr`. Values observed in `Game.exe`:

| tag | dword | what |
|---|---|---|
| `'OBJ_'` | `0x5f4a424f` | a model / object frame — the tag the I3D driver is asked for at `(*g_i3d_driver + 0x48)(g_i3d_driver, 0x5f4a424f)` |
| `'LMAP'` | `0x50414d4c` | a lightmap frame; carries the `0x38`-stride table at `+0x220` |
| `'SNGM'` | `0x4d474e53` | - |
| `'BBRD'` | `0x44524242` | - |
| `'MRPH'` | `0x4850524d` | - |
| `'SMMR'` | `0x524d4d53` | - |
| `'LENS'` | `0x534e454c` | - |

The `visual_type` at `+0x110` and this tag are tested **together** — `type == 1 && tag == 'OBJ_'`
is the standard "is this a real model?" gate (`structs/player.md`, `structs/entity.md`,
`mission_open_bin`).

### 3a. The `visual_type` values at `+0x110`

Three of these are pinned by a script-VM dispatcher case that REFUSES to run unless the frame
carries the type, which makes the value a guard the engine itself enforces.

| value | what | how it is established |
|---:|---|---|
| `1` | model / object frame | tested together with the `'OBJ_'` tag (§3) |
| `4` | **SOUND** | `sound_getvolume` (dispatcher case `0x13f`) bails unless the type is 4, then reads through `i3d_sound_get_volume_ptr` |
| `5` | **SECTOR** | `ispointinsector` (case `0x13c`) bails unless the type is 5, then calls the frame's own containment vtable slot `+0x68`. `frm_setpos` (case `0xf6`) tests the same value on the PARENT when deciding whether to re-home a frame |
| `6` | the `0x18`-byte `+0x140` block read out of the scene `.bin` | `i3d_frame_set_field_0x140_block` (§4) |
| `9` | a model / driver frame | `actor_duplicate` (case `0xda`) requires it before cloning; the I3D driver is asked for type 9 |
| `0xe` | **AREA** — a scripted trigger volume | `ispointinarea` (case `0x13d`) bails unless the type is `0xe`, then calls containment vtable slot `+0x5c` |

A sector and an area are DIFFERENT objects with different containment slots. The shipped scripts
use the area form 36 times and the sector form not at all.

### 3b. `+0x120` is the PARENT frame pointer

Four readers agree, and no other reading survives all four:

1. **`i3d_frame_get_scale` (`0x0045f090`)** — a two-armed branch that only makes sense for a
   parent: `+0x120 == 0` returns the local scale triple at `+0x90` unchanged; `+0x120 != 0`
   refreshes the world matrix when the `0x20` valid bit is clear and extracts the scale out of it.
   **For a root frame world scale IS local scale** — the identity the null arm encodes.
2. **`frm_getparent`, dispatcher case `0x13b`** — the script command the Mafia Script Data
   Reference describes as "returns the parent of child xx" clears its destination register and then
   stores exactly this pointer.
3. **`frm_setpos`, case `0xf6`** — after moving a frame it re-registers it with its sector, but
   skips that whenever `+0x120` is non-null AND the object it points at has visual type != 5. Read
   as a parent this is "a frame parented to a non-sector inherits sector membership from its
   parent"; read as anything else it is unexplainable.
4. **`actor_duplicate`, case `0xda`** — passes `*(+0x120)` of the source frame as the *parent*
   argument to the new driver object's vtable slot `+0x2c`.

## 4. The accessors that live in `Game.exe`

All of these are out-of-line copies of header inlines, so a `_ptr` name means the inline returns a
reference. The rows in bold sit in the `C_program` translation unit's out-of-line tail
(`structs/program.md` §6).

| PC VA | PC name | reads / writes | notes |
|---|---|---|---|
| `0x004029c0` | `i3d_frame_get_bound_points_2d` | - | the free `GetBoundPoints2D` overload: 4 world-space corners of the 2D bound rectangle |
| `0x004251f0` | `i3d_frame_get_local_position_ptr` | `+0x80` | |
| `0x00425200` | `i3d_frame_get_world_position_ptr` | `+0x40` | refreshes the world matrix first |
| `0x00425220` | `i3d_frame_get_forward_normalized` | `+0x30..+0x38` | normalises the forward row |
| `0x00453900` | `i3d_frame_build_scaled_rot_matrix` | `+0x50`, `+0x90` | void sibling of `0x005c83f0` |
| `0x0045f000` | `i3d_frame_set_position` | `+0x80` | takes one `S_vector` |
| `0x0045f040` | `i3d_frame_set_orientation` | `+0x9c` | 4 dwords + flag bit `0x8` |
| `0x0045f090` | `i3d_frame_get_scale` | `+0x90` | the **WORLD** scale: copies the local triple out when the frame has no PARENT (`+0x120` null), else extracts it from the world matrix (§3b) |
| `0x004f46c0` | `i3d_frame_get_world_matrix_ptr` | `+0x10` | refreshes first |
| `0x005289c0` | `i3d_frame_get_local_direction_normalized` | - | |
| `0x00528b70` | `i3d_frame_get_parent_ptr` | `+0x120` | the PARENT accessor, 30 uses (§3b) |
| **`0x005c82a0`** | **`i3d_frame_get_type_ptr`** | `+0x110` | 41 callsites — the busiest in the tail |
| **`0x005c82b0`** | `i3d_frame_set_position_xyz_ptrs` | `+0x80..+0x88` | three separate `float *` args |
| **`0x005c8300`** | **`i3d_frame_set_scale`** | `+0x90..+0x98` | materialises the quaternion first |
| **`0x005c8360`** | **`folded_get_ptr_0x90`** | `+0x90` | its `I3D_frame::GetScale` role; the symbol is folded and also serves `C_mission`, so the name stays mechanical |
| **`0x005c8370`** | **`i3d_frame_get_rotation_ptr`** | `+0x9c` | lazily `S_quat::Make(rot, local_rot3)` |
| **`0x005c83a0`** | `i3d_frame_get_world_rotation_quat` | `+0x40` | copies out a quaternion |
| **`0x005c83f0`** | **`i3d_frame_get_scaled_rot_matrix_ptr`** | `+0x50` | the `_ptr` sibling of `0x00453900` |
| **`0x005c8510`** | **`i3d_frame_is_on`** | `+0xac & 1` | the noun is [M] |
| **`0x005c8520`** | **`i3d_frame_get_name`** | `+0x100` | the name is an `I3D_frame` field, not a `C_actor` one: every disassembled callsite is `call folded_get_dword_0x68; mov ecx,eax; call <it>` |
| **`0x005c8530`** | **`i3d_frame_get_child_count_ptr`** | `+0x11c` | the PS2 reaches the same quantity through the `GetNumChilds` virtual |
| **`0x005c8580`** | **`i3d_frame_get_tag_ptr`** | `+0x1d4` | |
| **`0x005c8590`** | **`i3d_frame_get_mesh_object_ptr`** | `+0x1e0` | |
| **`0x005c85a0`** | **`i3d_camera_get_fov_ptr`** | `+0x140` | the receiver is an `I3D_camera`, so the field is a CAMERA field reached through the frame base; same 180/pi conversion on both builds |
| **`0x005c85b0`** | **`i3d_frame_get_lmap_entry_ptr`** | `+0x220 + i*0x38` | noun from the caller's `'LMAP'` gate |
| **`0x005c85d0`** | **`i3d_frame_get_model_name`** | `+0x154` | [M] — see §5 |
| **`0x005c85e0`** | **`i3d_scene_get_active_camera_ptr`** | `+0x17c` | |
| **`0x005c85f0`** | **`i3d_scene_get_root_frame_ptr`** | `+0x210` | passed to the same link virtual as the PS2's scene root |
| **`0x005c8600`** | **`i3d_scene_set_frame_sector_pos`** | scene vtable `+0xbc` | |
| **`0x005c8630`** | **`i3d_sound_get_volume_ptr`** | `+0x18c` | under the frame-type-4 gate |
| `0x00606370` | `i3d_frame_set_field_0x140_block` | `+0x140` | stores a `0x18`-byte block read straight out of the scene `.bin`, so `+0x140` is at least `0x18` bytes wide and is FILE DATA. Its one callsite, in `mission_open_bin`, is guarded on `i3d_frame_get_type_ptr` returning visual type **6** |

## 5. PC vs PS2

- **The PS2 has no DLLs**, so it compiled LS3D in and its symbol table carries 48 `I3D_frame`
  methods and 71 `I3D_scene` methods in the clear. That is the ONLY place the class's own method
  names exist (`basic/artifacts/dll.md`).
- **`I3D_scene::SetFrameSectorPos` ships in two overloads on the PS2** — the 220-byte virtual
  `…FP9I3D_frameRC8S_vector`, which does `GetSector(pos)` then `frame->LinkTo(sector, 1)`, and a
  48-byte inline `…RC8S_vectorRC8S_vector`. The PC's `i3d_scene_set_frame_sector_pos` is the thin
  inline forwarding to the virtual at slot `+0xbc`.
- **The FourCC and the visual type are VIRTUALS on the PS2 and plain FIELDS on the PC.** PS2 reads
  them as `(**(code **)(*frame + 0xc))(frame) == 0x5f4a424f`; the PC reads `*(frame + 0x1d4)`. A
  cross-build match in this area must not expect a call on both sides.
- The `model_name` reading of `+0x154` rests on the PS2 pair `SetModelName__9I3D_frame` (384 B) and
  `ResetModelName__9I3D_frame` (188 B): the frame carries exactly two name strings and the PS2
  class has exactly two name setters. It is **[M]**, not byte-proven.
- `I3D_frame::GetDir` (380 B), `GetWorldPos` (56 B), `GetWorldDir` (348 B), `UpdateMatrix` (404 B)
  and `UpdateWMatrix` (44 B) are the PS2's own out-of-lined copies of the same header inlines this
  file lists — the same construct, a different host translation unit.
- **Field deltas for the mesh chain:** PC `face_group + 0x04` is PS2 `face_group + 0x00`, and PC
  `+0x08` (face count) is PS2 `+0x04`, because those classes are non-polymorphic on the PS2 (no
  `__vt__` symbol exists for any of them) and fully virtual on the PC (§9).
- **PC vtable indices are NOT PS2 vtable indices** for this family. `I3D_frame`'s PC table is
  called at `+0x90` (index 36) by `game_scratch_recompile`, while `__vt__9I3D_frame` has only 29
  virtuals — the PC interface is strictly larger. The one place the two agree slot for slot is
  `IShow` (`basic/artifacts/script-opcode-dispatch.md` §10), a COM-style interface on both builds.
- **`I3D_camera+0x140`** is PS2 `camera+0x120`; **`+0x18c`** (sound volume) is PS2 `frame+0x16c`;
  **`I3D_scene+0x210`** is PS2 `scene+0x1d8`.

## 6. Related

- `basic/artifacts/dll.md` — what `ls3df.dll` exports and why the class body is not in `Game.exe`.
- `structs/actor.md` — `C_actor+0x68` holds the frame.
- `structs/program.md` §6 — the translation unit that hosts most of these inlines.
- `basic/artifacts/script-opcode-dispatch.md` — the dispatcher cases that reach these fields.

## 9. The three mesh classes reached through `+0x1e0`

`+0x1e0` does not hold a generic "visual". It holds an **`I3D_mesh_object`**, and three one-line
header inlines in the `C_program` tail walk from it down to a material:

```
I3D_frame (type 1, tag 'OBJ_')
  +0x1e0 -> I3D_mesh_object
              +0x08          lod_count          i3d_mesh_object_get_lod_count_ptr
              virtual 1      GetLOD(int)     -> I3D_mesh_level
                               +0x18        face_group_count
                                            i3d_mesh_level_get_face_group_count_ptr
                               +0x1c        face-group list head
                               virtual 1    LockVertices(u32) -> vertex array, stride 0x20
                               virtual 2    UnlockVertices()
                               virtual 5    GetFGroup(int) -> I3D_face_group
                                              +0x04  I3D_material *
                                                     i3d_face_group_get_material_ptr
                                              +0x08  face count
                                              virtual 3  GetFaces(u32) -> u16 triples
```

**How the classes are established**, given that the PS2 `STREAM_CONNECT` arm is an empty stub:
`glass_shatter_cut_mesh` walks the identical chain and its PS2 twin
`CutMesh__19C_Break_plain_glass…` is CONFIRMED, so laying the two side by side names every hop —
`GetLOD__15I3D_mesh_objectCFi`, `LockVertices__14I3D_mesh_levelFUi`, `GetFGroup__14I3D_mesh_levelFi`,
and `I3D_material::Duplicate` taking `*(u32 *)fgroup`. The two counts then fall out of `0x004fc8fd`,
where each is the loop bound of the indexed getter it guards. The PS2 hands `+0x1e0` to
`GetLOD__15I3D_mesh_objectCFi` in the twins of `uv_scroll_channel_tick` and
`glass_shatter_cut_mesh`, at the same `+0x1e0` on both builds.

## 10. Open items

| item | state |
|---|---|
| the `0x38`-stride table at `+0x220` | [U]; reached only under the `'LMAP'` tag |
| `i3d_frame_is_on`'s noun | the bit is measured, the name "on / enabled" is [M] |
| the `0x18`-byte `+0x140` block | file data from the scene `.bin` on a visual-type-6 frame; what it means is [U] |
