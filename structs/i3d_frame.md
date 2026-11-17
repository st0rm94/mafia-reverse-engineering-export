# `I3D_frame` — the LS3D scene node, its derived frames and the mesh chain

**Created** 2026-08-30 19:07 +0200 · **Modified** 2026-11-16 16:16 +0100

`I3D_frame` is the LS3D engine's **scene-graph node**: a transform, a name, a sector membership, a
frame type and a four-character class tag. Every `C_actor` owns one at `actor+0x68`
(`folded_get_dword_0x68`, 253 callsites), and the mission script VM holds them directly in its
frame register file (`C_program+0x50`, typed by the signature
`SetFrameReg__9C_programFiP9I3D_frame`). The scene itself (`I3D_scene`), the camera, the sound,
the dummy and the lightmapped visual are frames too, and a visual frame reaches its mesh through a
three-class chain; their fields are documented here as sub-structures (§1a–§1j).

**The classes are not in `Game.exe`.** They are implemented in **`ls3df.dll`**, which exports only
three `I3D_frame` methods (`?UpdateWMatrixProc@I3D_frame@@AAEXXZ`, `?UpdateHRBoundVol@…`,
`?UpdateHRBoundVolW@…`). What lives in `Game.exe` is the set of **header inlines** MSVC out-lined
into whichever translation unit used them (`basic/artifacts/dll.md`): there is no vtable to dump
and no constructor to diff, and the method table of §3 is a table of accessors.

| | |
|---|---|
| PC prefix | `i3d_frame_`, `i3d_scene_`, `i3d_camera_`, `i3d_sound_`, `i3d_dummy_`, `i3d_mesh_object_`, `i3d_mesh_level_`, `i3d_face_group_` |
| PC kind (`+0x10`) | - |
| PC vtable | - (inside `ls3df.dll`) |
| PC base class | - |
| PC instance size | - (allocated by `ls3df.dll`) |
| PC constructor | - |
| PC translation unit | `[0x005c82a0, 0x005c8640)`, the out-of-line tail of the `C_program` unit (`structs/program.md` §8), holds 22 of the functions of §3; the other 14 sit in their callers' units |
| PS2 class | `I3D_frame` (tag `9I3D_frame`), 48 methods, and the derived and linked classes of §1a–§1j; none has a PC function (§9) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the offsets `Game.exe` reaches are listed; the gaps are fields only `ls3df.dll` touches.

```c
typedef struct I3D_frame {
    /* +0x08  */ C_actor **owner_cell;      // [H] a cell holding the actor that owns the frame; null or an empty cell = no owner
    /* +0x10  */ S_matrix world_matrix;     // [H] the world matrix, 0x40 bytes; refreshed on demand
    /* +0x50  */ float    local_rot[3][4];  // [H] the local rotation x scale, three padded rows
    /* +0x80  */ S_vector position;         // [H] the local position
    /* +0x90  */ S_vector scale;            // [H] the local scale
    /* +0x9c  */ S_quat   rotation;         // [H] the local rotation quaternion
    /* +0xac  */ uint32_t flags;            // [H] validity and dirty bits (§4)
    /* +0xb0  */ S_vector bound_min;        // [H] the hierarchy bound box, minimum corner
    /* +0xbc  */ S_vector bound_max;        // [H] the hierarchy bound box, maximum corner
    /* +0xc8  */ S_vector bound_center;     // [H] the hierarchy bound volume's centre
    /* +0xd4  */ float    unk_d4;           // [U] copied with the volume; meaning in ls3df.dll
    /* +0x100 */ char    *name;             // [H] null -> g_empty_string
    /* +0x110 */ int32_t  frame_type;       // [H] 1..17, LS3DF.dll's FRAME_* names (§5b)
    /* +0x11c */ int32_t  child_count;      // [H] the number of child frames
    /* +0x120 */ I3D_frame *parent;         // [H] null for a root frame (§6)
    /* +0x13c */ void    *sector;           // [H] the visibility cell the frame is in
    /* +0x154 */ char    *model_name;       // [H] the model file name; null -> g_empty_string
    /* +0x1d4 */ uint32_t class_tag;        // [H] the FourCC class tag (§5a)
} I3D_frame;
```

**The world matrix and the two rows read from it.** `i3d_frame_get_world_matrix_ptr` returns
`+0x10` after refreshing it, so the matrix spans `+0x10..+0x4f`: its translation row `+0x40..+0x48`
is the world POSITION (`i3d_frame_get_world_position_ptr`, `actor_get_world_position`), and its
third row `+0x30..+0x38` the world FORWARD direction (`i3d_frame_get_forward_normalized`).

**`local_rot[3][4]` — why three by four.** It is the 3×3 local orientation scaled by `scale`,
stored as three rows of four floats (x, y, z and an unused fourth) so each row is 16 bytes, which
is why `position` starts exactly `0x30` later. `i3d_frame_build_scaled_rot_matrix` and
`i3d_frame_get_scaled_rot_matrix_ptr` rebuild it lazily from `rotation` and `scale` (flag `0x4`
marks it valid), and `i3d_frame_set_direction` writes it through `S_matrix::SetDir3` (flag `0x10`).

**The bound volume `+0xb0..+0xd7`.** Ten floats that `ls3df.dll` computes for the frame and its
subtree in `I3D_frame::UpdateHRBoundVol`; the game calls it whenever flag `0x40` is clear (§4), at 59
sites in 40 functions, and then reads the box corners or the centre (§2). Two functions copy all ten
floats at once.

### 1a. `I3D_scene` — the scene root, a frame

`I3D_scene` is an `I3D_frame` with part of the base API suppressed (§9). That is why the scene and
a frame can be compared on the *same* offset `+0x13c`.

```c
typedef struct I3D_scene {                   // : I3D_frame
    /* +0x17c */ I3D_frame *active_camera;   // [M] the camera the scene renders through
    /* +0x210 */ I3D_frame *root_frame;      // [H] the scene's root frame
} I3D_scene;
```

**The singleton is `C_mission+0x10`** (`structs/mission.md` §1), identified by
`i3d_scene_set_frame_sector_pos`, whose receiver is that pointer. Its vtable slot `+0xbc` is
`SetFrameSectorPos(I3D_frame *, const S_vector &)`.

### 1b. `I3D_camera` — a camera frame

```c
typedef struct I3D_camera {                  // : I3D_frame, frame type 3
    /* +0x140 */ float fov;                  // [H] the field of view, radians
} I3D_camera;
```

### 1c. `I3D_dummy` — a dummy frame and its box

```c
typedef struct I3D_dummy {                   // : I3D_frame, frame type 6
    /* +0x140 */ S_vector bbox_min;          // [H] the bounding box's minimum corner
    /* +0x14c */ S_vector bbox_max;          // [H] its maximum corner
} I3D_dummy;
```

The `0x18`-byte box is read out of the scene file (chunk `0x4050`, `structs/mission.md` §6a) and
stored by `i3d_dummy_set_bound_box`; its readers use it as a min / max corner pair, among them the
car's `"TANK"` frame radius (`structs/car.md` §4a) and the `~DCC` boxes, which are built only on
type-6 frames (`structs/car_extended.md` §5c).

### 1d. `I3D_sound` — a sound frame

```c
typedef struct I3D_sound {                   // : I3D_frame, frame type 4
    /* +0x148 */ uint32_t unk_148;           // [U] set to 0x80 before the sound is linked; meaning in ls3df.dll
    /* +0x160 */ C_string file_name;         // [H] the file name the sound is opened with
    /* +0x18c */ int32_t volume;             // [H] read under the frame-type-4 gate
    /* +0x198 */ uint32_t open_flags;        // [H] the flags the sound is opened with
} I3D_sound;
```

### 1e. `I3D_lit_object` — a lightmapped visual (`'LMAP'`)

```c
typedef struct I3D_lit_object {              // : I3D_frame, frame type 1, tag 'LMAP'
    /* +0x21c */ int32_t      active_level;  // [H] the lightmap level shown
    /* +0x220 */ S_lmap_level levels[];      // [H] one 0x38-byte record per lightmap level
} I3D_lit_object;
```

**`levels[]` — why open-ended.** `ls3df.dll` indexes the table as
`[frame+0x21c] * 0x38 + 0x220`, and entry *i* is lightmap level *i*. The table's capacity is set
inside `ls3df.dll` and `Game.exe` never reads it; every shipped `setlmlevel` call passes level 0 or
1 (for example mission `MISE11-VILA`, script `9lustr`, a chandelier), so the game uses two levels.

An `'LMAP'` frame is a visual too: it carries its mesh object at `+0x1e0`, as an `'OBJ_'` frame does
(§1j).

### 1f. `S_lmap_level` — one lightmap level, 0x38 bytes

```c
typedef struct S_lmap_level {
    /* +0x00 */ uint8_t present;             // [H] the level exists
    /* +0x10 */ void   *lod_maps[10];        // [H] one lightmap per level of detail
} S_lmap_level;
```

**`lod_maps[10]` — why ten.** The record ends at `0x38` = `0x10 + 10 × 4`, and `ls3df.dll` walks the
pointers ten at a time from record `+0x10` when it clears or queries a level: entry *i* is the
lightmap of mesh level-of-detail *i*.

### 1g. `I3D_mesh_object` — the mesh of a visual frame

```c
typedef struct I3D_mesh_object {             // reached through I3D_object+0x1e0 (§1j, §8)
    /* +0x00 */ void   *vtable;              // [H] virtual 1 = GetLOD(int)
    /* +0x08 */ int32_t lod_count;           // [H] the number of levels of detail
} I3D_mesh_object;
```

`I3D_object+0x1e0` holds the mesh object of a type-1 visual frame, tagged `'OBJ_'` or `'LMAP'` (§1j).

### 1h. `I3D_mesh_level` — one level of detail

```c
typedef struct I3D_mesh_level {
    /* +0x00 */ void   *vtable;              // [H] 1 LockVertices, 2 UnlockVertices, 5 GetFGroup
    /* +0x18 */ int32_t face_group_count;    // [H] the number of face groups
    /* +0x1c */ void   *face_groups;         // [H] the face-group list head
} I3D_mesh_level;
```

### 1i. `I3D_face_group` — one material's faces

```c
typedef struct I3D_face_group {
    /* +0x00 */ void         *vtable;        // [H] virtual 3 = GetFaces(u32) -> u16 triples
    /* +0x04 */ I3D_material *material;      // [H] the material
    /* +0x08 */ int32_t       face_count;    // [H] the number of faces
} I3D_face_group;
```

### 1j. `I3D_object` — a visual frame and its mesh

```c
typedef struct I3D_object {                  // : I3D_frame, frame type 1, tag 'OBJ_' or 'LMAP'
    /* +0x1e0 */ I3D_mesh_object *mesh_object; // [H] the visual's mesh (§1g, §8)
} I3D_object;
```

Both tags carry the field: `i3d_frame_get_mesh_object_ptr` is called on a type-1 `'OBJ_'` frame by a
script-VM case of `program_process_command`, and on a type-1 frame tagged `'LMAP'` or `'OBJ_'` by the
uv-scroll chunk `0xAE03` of `mission_open_bin`; `glass_shatter_cut_mesh` loads it directly.

## 2. Field evidence

Measured by body reads of every accessor of §3 and of the script-VM cases and functions named in
the notes. The classes are written inside `ls3df.dll`; a `Game.exe` writer is named where one
exists. Direct (non-accessor) accesses are recorded where a body read found them; no `.text` census
of them was made (Open item 2).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x08 | owner_cell | H | `game_box_add` (zeroes the cell after `mission_unload_actor`, and frees it), `i3d_model_cache_cb_null_data` (stores 0 into `+0x08` of every frame of a model the model cache duplicates) | `game_add_frame_placement`, `game_box_add`, `game_box_count_owned_on_car`, `game_box_place_at_next_dest`, `game_clear_differences`, `game_light_cache_collect_recursive` | every reader tests the pointer and then the cell: `game_add_frame_placement` registers only a frame with no owner, `game_clear_differences` invalidates and deletes the owner, the two box functions walk the parents until the owner is a car, and the light-cache walk skips an owned subtree (`structs/lightness.md` §7). The code that allocates and fills the cell was not read |
| +0x10 | world_matrix | H | `ls3df.dll` (`I3D_frame::UpdateWMatrixProc`) | `i3d_frame_get_world_matrix_ptr`, `i3d_frame_get_world_position_ptr`, `i3d_frame_get_forward_normalized`, `i3d_frame_get_world_rotation_quat`, `i3d_frame_get_scale` | refreshed while flag `0x20` is clear |
| +0x50 | local_rot | H | `i3d_frame_build_scaled_rot_matrix`, `i3d_frame_get_scaled_rot_matrix_ptr`, `i3d_frame_set_direction` | `i3d_frame_get_rotation_ptr` (`S_quat::Make`), `i3d_frame_get_scaled_rot_matrix_ptr` | §1 |
| +0x80 | position | H | `i3d_frame_set_position`, `i3d_frame_set_position_xyz_ptrs`; direct stores under the write guard in `cache_base_block_activate_block`, `speech_channel_speak_line`, `speech_channel_set_look_at_frame`, `speech_update_look_at_targets` | `i3d_frame_get_local_position_ptr` | - |
| +0x90 | scale | H | `i3d_frame_set_scale`, `cache_base_block_activate_block` (a placed instance's scale, and the scale of its type-1 children) | `i3d_frame_get_scale`, `folded_get_ptr_0x90`, `i3d_frame_build_scaled_rot_matrix`, `cache_base_block_walk_block_objects` (`+0x94` of a root frame) | - |
| +0x9c | rotation | H | `i3d_frame_set_orientation`, `i3d_frame_set_scale` (materialises it first), `i3d_frame_get_rotation_ptr` (lazily from `local_rot`), `cache_base_block_activate_block` (a placed instance's rotation) | `i3d_frame_get_rotation_ptr`, `i3d_frame_build_scaled_rot_matrix` | 4 floats `+0x9c..+0xa8` |
| +0xac | flags | H | `i3d_frame_set_position_xyz_ptrs`, `i3d_frame_set_orientation`, `i3d_frame_set_scale`, `i3d_frame_set_direction`, `i3d_frame_get_rotation_ptr`, the `C_human` weapon-effect family, `drop_in_weapon` slot 18 | `i3d_frame_is_on`, `i3d_frame_get_scaled_rot_matrix_ptr`, `i3d_frame_set_position_xyz_ptrs`, `i3d_frame_get_scale`, `i3d_frame_get_bound_points_2d`, `cache_base_block_walk_block_objects`, `speech_update_look_at_targets`, the 59 bound-volume refresh sites (bit `0x40`) | §4 |
| +0xb0 | bound_min | H | `ls3df.dll` (`I3D_frame::UpdateHRBoundVol`, called by the 59 refresh sites in 40 functions while flag `0x40` is clear) | `i3d_frame_get_bound_points_2d` (x / z inflated by 0.7, mid-height from y), `cache_base_block_walk_block_objects` (the 2.0 prune, scaled by `+0x94`), `door_game_init`, `box_new_init` and the others of the 40 | `i3d_copy_bound_volume` and `railway_game_init` copy all ten floats `+0xb0..+0xd7` |
| +0xbc | bound_max | H | `ls3df.dll`, as `bound_min` | the readers of `bound_min` | - |
| +0xc8 | bound_center | H | `ls3df.dll`, as `bound_min` | `door_ai` (transforms it into its collision centre, `structs/door.md`), `i3d_copy_bound_volume`, `railway_game_init` | - |
| +0xd4 | unk_d4 | U | `ls3df.dll`, as `bound_min` | `i3d_copy_bound_volume`, `railway_game_init` (the whole-volume copies) | no `Game.exe` reader of this float alone at the 59 refresh sites; meaning in `ls3df.dll` (Open item 4) |
| +0x100 | name | H | `ls3df.dll` | `i3d_frame_get_name` | every disassembled callsite is `call folded_get_dword_0x68; mov ecx,eax; call i3d_frame_get_name` |
| +0x110 | frame_type | H | `ls3df.dll` | `i3d_frame_get_type_ptr` (41 callsites), `mission_open_bin`, the script-VM cases of §5b | §5b |
| +0x11c | child_count | H | `ls3df.dll` | `i3d_frame_get_child_count_ptr` | - |
| +0x120 | parent | H | `ls3df.dll` | `i3d_frame_get_parent_ptr` (30 uses), `i3d_frame_get_scale`, the script-VM cases `0x13b`, `0xf6`, `0xda` | §6 |
| +0x13c | sector | H | `ls3df.dll` | `game_tick` (copies the scene's into `C_mission+0x64`), `pedestrian_tick` (compares a frame's with it) | a frame is in the camera's sector when its `+0x13c` equals the scene's |
| +0x154 | model_name | H | `ls3df.dll` | `i3d_frame_get_model_name`, `mission_actor_cache_store`, `mission_model_cache_store` | its consumer `actor_duplicate` (case `0xda`) strips the directory and opens the model through `i3d_cache_model_open` |
| +0x1d4 | class_tag | H | `ls3df.dll` | `i3d_frame_get_tag_ptr`, and direct reads as `param_1[0x75]` (§5a) | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x17c | active_camera | M | `ls3df.dll` | `i3d_scene_get_active_camera_ptr`, `mission_update_settings`, `mission_open_bin`, `game_tick`, `cache_base_block_tick` (the streaming reference position), `speech_channel_speak_line` (the 12-unit subtitle distance) | the reading rests on the accessor's name and on `mission_open_bin` applying the camera chunks `0x3010`..`0x3013` through it |
| +0x210 | root_frame | H | `ls3df.dll` | `i3d_scene_get_root_frame_ptr` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x140 | fov | H | `ls3df.dll` | `i3d_camera_get_fov_ptr` | its 180/π conversion matches the PS2's (§9) |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x140 | bbox_min | H | `i3d_dummy_set_bound_box` | the `"TANK"` radius of `structs/car.md` §4a, the `~DCC` boxes of `structs/car_extended.md` §5c | §1c |
| +0x14c | bbox_max | H | `i3d_dummy_set_bound_box` | the same | §1c |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x148 | unk_148 | U | `speech_channel_speak_line` (`0x80`, before it links the sound) | `ls3df.dll` | meaning in `ls3df.dll` (Open item 4) |
| +0x160 | file_name | H | `ls3df.dll` | `i3d_sound_cache_duplicate` (through `string_ensure_buffer`, the file name of `Open`, slot `0x50`) | a `C_string` |
| +0x18c | volume | H | `ls3df.dll` | `i3d_sound_get_volume_ptr`, `sound_getvolume` (case `0x13f`) | - |
| +0x198 | open_flags | H | `ls3df.dll` | `i3d_sound_cache_duplicate` (the flags of `Open`, with `file_name`) | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x21c | active_level | H | `ls3df.dll` (frame slot `+0xa0`, called by `setlmlevel` case `0x1e` and `program_switch_light_map`) | `ls3df.dll` | §7 |
| +0x220 | levels | H | `ls3df.dll` | `i3d_frame_get_lmap_entry_ptr`, `program_switch_light_map`, the `setlmlevel` arm | §7 |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | present | H | `ls3df.dll` | `program_switch_light_map`, the `setlmlevel` arm | tested before a switch |
| +0x10 | lod_maps | H | `ls3df.dll` | `ls3df.dll` | ten per record (§1f) |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `ls3df.dll` | `glass_shatter_cut_mesh`, `uv_scroll_channel_tick`, the code at `0x004fc8fd` | `GetLOD` |
| +0x08 | lod_count | H | `ls3df.dll` | `i3d_mesh_object_get_lod_count_ptr` | the loop bound of `GetLOD` at `0x004fc8fd` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `ls3df.dll` | `glass_shatter_cut_mesh`, the code at `0x004fc8fd` | `LockVertices`, `UnlockVertices`, `GetFGroup` |
| +0x18 | face_group_count | H | `ls3df.dll` | `i3d_mesh_level_get_face_group_count_ptr` | the loop bound of `GetFGroup` at `0x004fc8fd` |
| +0x1c | face_groups | H | `ls3df.dll` | `ls3df.dll` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `ls3df.dll` | `glass_shatter_cut_mesh` | `GetFaces` |
| +0x04 | material | H | `ls3df.dll` | `i3d_face_group_get_material_ptr`, `glass_shatter_cut_mesh` (`I3D_material::Duplicate`) | - |
| +0x08 | face_count | H | `ls3df.dll` | `glass_shatter_cut_mesh` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x1e0 | mesh_object | H | `ls3df.dll` | `i3d_frame_get_mesh_object_ptr` (a `program_process_command` case gated on type 1 and `'OBJ_'`; the uv-scroll chunk `0xAE03` of `mission_open_bin`, gated on type 1 and `'LMAP'` or `'OBJ_'`), `glass_shatter_cut_mesh` (a direct load) | §1j, §8 |

## 3. Method family

Coverage: no PS2 method of these classes has a PC function (§9). The tables hold the 34 header
inlines `Game.exe` carries and two free functions, one of them with a PS2 twin; the `Role` cell names the
receiver class where it is not `I3D_frame`. Every row is an out-of-line copy of a header inline,
so a `_ptr` name means the inline returns a reference. The rows from `0x005c82a0` to `0x005c8630`
sit in the `C_program` unit's tail.

### 3a. Construction & destruction

The classes are constructed and destroyed inside `ls3df.dll`; `Game.exe` holds no such code.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004251f0 | `i3d_frame_get_local_position_ptr` | - | - | - | `&position` |
| 0x00528b70 | `i3d_frame_get_parent_ptr` | - | - | - | `&parent`, 30 uses (§6) |
| 0x005c82a0 | `i3d_frame_get_type_ptr` | - | - | - | `&frame_type`, 41 callsites, the busiest in the tail |
| 0x005c8360 | `folded_get_ptr_0x90` | - | - | - | `&scale`, the `GetScale` role; the symbol is ICF-folded and also serves `C_mission` (`&settings`), so the name stays mechanical; 4 callsites in `program_process_command` |
| 0x005c8530 | `i3d_frame_get_child_count_ptr` | - | - | - | `&child_count` |
| 0x005c8560 | `i3d_mesh_level_get_face_group_count_ptr` | - | - | - | `I3D_mesh_level`: `&face_group_count` |
| 0x005c8580 | `i3d_frame_get_tag_ptr` | - | - | - | `&class_tag` |
| 0x005c8590 | `i3d_frame_get_mesh_object_ptr` | - | - | - | `I3D_object`: `&mesh_object` (§1j) |
| 0x005c85a0 | `i3d_camera_get_fov_ptr` | - | - | - | `I3D_camera`: `&fov` |
| 0x005c85e0 | `i3d_scene_get_active_camera_ptr` | - | - | - | `I3D_scene`: `&active_camera` |
| 0x005c85f0 | `i3d_scene_get_root_frame_ptr` | - | - | - | `I3D_scene`: `&root_frame` |
| 0x005c8630 | `i3d_sound_get_volume_ptr` | - | - | - | `I3D_sound`: `&volume`, called under the frame-type-4 gate |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00425200 | `i3d_frame_get_world_position_ptr` | - | - | - | refreshes the world matrix, returns its translation row `+0x40` |
| 0x00425220 | `i3d_frame_get_forward_normalized` | - | - | - | normalises the world matrix's third row `+0x30..+0x38` |
| 0x00453900 | `i3d_frame_build_scaled_rot_matrix` | - | - | - | rebuilds `local_rot` from `rotation` and `scale`; the void sibling of `i3d_frame_get_scaled_rot_matrix_ptr` |
| 0x0045f000 | `i3d_frame_set_position` | - | - | - | writes `position` from one `S_vector`, gated by the write guard of §4 |
| 0x0045f040 | `i3d_frame_set_orientation` | - | - | - | copies 4 dwords into `rotation`, sets flag `0x8` |
| 0x0045f090 | `i3d_frame_get_scale` | - | - | - | the **WORLD** scale: `scale` when the frame has no parent, else extracted from the world matrix (§6) |
| 0x004c6b20 | `i3d_frame_set_direction` | - | - | - | `S_matrix::SetDir3` into `local_rot`, then `flags = flags & 0xfffffed3 \| 0x40000010` (rotation source = the matrix); callers `physical_object_move_wheel`, `camera_apply_desired_pos` |
| 0x004f46c0 | `i3d_frame_get_world_matrix_ptr` | - | - | - | refreshes, returns `&world_matrix` |
| 0x005289c0 | `i3d_frame_get_local_direction_normalized` | - | - | - | the normalised local direction |
| 0x005c82b0 | `i3d_frame_set_position_xyz_ptrs` | - | - | - | writes `position` from three separate `float *` arguments |
| 0x005c8300 | `i3d_frame_set_scale` | - | - | - | writes `scale`, materialising `rotation` first |
| 0x005c8370 | `i3d_frame_get_rotation_ptr` | - | - | - | lazily `S_quat::Make(rotation, local_rot)`, returns `&rotation` |
| 0x005c83a0 | `i3d_frame_get_world_rotation_quat` | - | - | - | copies a quaternion out of the world matrix |
| 0x005c83f0 | `i3d_frame_get_scaled_rot_matrix_ptr` | - | - | - | rebuilds `local_rot` when flag `0x4` is clear, returns it |
| 0x005c8510 | `i3d_frame_is_on` | - | - | - | `flags & 1`, the frame's on state (§4) |
| 0x005c8520 | `i3d_frame_get_name` | - | - | - | `name`, or `g_empty_string` |
| 0x005c8550 | `i3d_face_group_get_material_ptr` | - | - | - | `I3D_face_group`: the material |
| 0x005c8570 | `i3d_mesh_object_get_lod_count_ptr` | - | - | - | `I3D_mesh_object`: the level-of-detail count |
| 0x005c85b0 | `i3d_frame_get_lmap_entry_ptr` | - | - | - | `I3D_lit_object`: `levels + i * 0x38` (§7) |
| 0x005c85d0 | `i3d_frame_get_model_name` | - | - | - | `model_name`, or `g_empty_string`; its one consumer is `actor_duplicate` (case `0xda`) |
| 0x005c8600 | `i3d_scene_set_frame_sector_pos` | - | - | - | `I3D_scene`: forwards to the scene's virtual `SetFrameSectorPos` at slot `+0xbc` |
| 0x00606370 | `i3d_dummy_set_bound_box` | - | - | - | `I3D_dummy`: stores the `0x18`-byte box of scene chunk `0x4050`; its one callsite, in `mission_open_bin`, is guarded on frame type 6 |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004029c0 | `i3d_frame_get_bound_points_2d` | `GetBoundPoints2D__FRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x00357130 | CONFIRMED | the free overload: the four world-space corners of a frame's 2D bound rectangle, inflated by 0.7 at mid-height; the footprint helper of the `ai_path_find_spec_node_*` searches |
| 0x00473f90 | `i3d_frame_match_deform_roof` | - | - | - | a frame-enumeration predicate: appends each frame whose name carries a `[DEFORM_ROOF]` tag to the caller's vector; passed by `car_set_roof_visible` (Open item 3) |

## 4. The flag word at `+0xac`

| Bit | Meaning | Evidence |
|---|---|---|
| `0x1` | **[H]** on (the frame is shown) | `i3d_frame_is_on` returns it; the script command `frm_ison` (case `0xf`) reports exactly this bit, and the sound-fade case calls frame slot `+0x24` (the slot `frm_seton` uses) with 1 when it is clear |
| `0x4` | `local_rot` valid | `i3d_frame_get_scaled_rot_matrix_ptr` returns early when set |
| `0x8` | `rotation` valid | set by `i3d_frame_get_rotation_ptr` and `i3d_frame_set_orientation` |
| `0x10` | the rotation source is the matrix, not the quaternion | selects which branch rebuilds the 3×3; set by `i3d_frame_set_direction` |
| `0x20` | world matrix valid | the refresh is done by `I3D_frame::UpdateWMatrixProc` |
| `0x40` | **[H]** the bound volume `+0xb0..+0xd7` is valid | the game calls `I3D_frame::UpdateHRBoundVol` whenever it is clear, at 59 sites in 40 functions |
| `0x80` (low byte sign) | a write guard | tested by `i3d_frame_set_position` and `i3d_frame_set_position_xyz_ptrs` before writing |
| `0x100` | **[U]** cleared by every setter; meaning in `ls3df.dll` | the two composite rewrites below |
| `0x40000000` | **[U]** set by every setter; meaning in `ls3df.dll` | the two composite rewrites below |

Two composite rewrites recur across the engine and are useful as **shared magic constants** when
matching functions across builds:

* `flags &= 0xfffffedf; flags |= 0x40000000;` — **position changed**
  (`i3d_frame_set_position_xyz_ptrs`, and the whole `C_human` weapon-effect family).
* `flags &= 0xfffffecb; flags |= 0x40000008;` — **rotation / scale changed**
  (`i3d_frame_set_scale`, `drop_in_weapon` slot 18).

## 5. Frame type and class tag

### 5a. The FourCC tag at `+0x1d4`

Read directly at many sites as `param_1[0x75]` and through `i3d_frame_get_tag_ptr`. Values observed
in `Game.exe`:

| Tag | Dword | What |
|---|---|---|
| `'OBJ_'` | `0x5f4a424f` | a model / object visual — the tag the I3D driver is asked for at `(*g_i3d_driver + 0x48)(g_i3d_driver, 0x5f4a424f)` |
| `'LMAP'` | `0x50414d4c` | a lightmapped visual, `I3D_lit_object` (§1e, §7) |
| `'SNGM'` | `0x4d474e53` | - |
| `'BBRD'` | `0x44524242` | - |
| `'MRPH'` | `0x4850524d` | - |
| `'SMMR'` | `0x524d4d53` | - |
| `'LENS'` | `0x534e454c` | - |

The frame type and this tag are tested **together**: `type == 1 && tag == 'OBJ_'` is the standard
"is this a real model?" gate (`structs/player.md`, `structs/entity.md`, `mission_open_bin`).

### 5b. The frame types at `+0x110`

**The names come from `ls3df.dll`.** Its frame-type-name function (`0x10019760` in the DLL) is a
jump table indexed by `type − 1` whose arms return the strings `FRAME_VISUAL` … `FRAME_EMITOR`, in
this order: 1 VISUAL · 2 LIGHT · 3 CAMERA · 4 SOUND · 5 SECTOR · 6 DUMMY · 7 TARGET · 8 USER ·
9 MODEL · 10 JOINT · 11 VOLUME · 12 OCCLUDER · 13 SCENE · 14 AREA · 15 SHADOW · 16 LANDSCAPE ·
17 EMITOR.

The rows below are the types the game's own code tests. Three are pinned by a script-VM dispatcher
case that REFUSES to run unless the frame carries the type, which makes the value a guard the
engine itself enforces; the scene-file arms of `mission_open_bin` (`structs/mission.md` §6a) gate
on several more.

| Value | What | How it is established |
|---:|---|---|
| `1` | **VISUAL**, a model / object frame | tested together with the `'OBJ_'` tag (§5a) |
| `2` | **LIGHT** | scene chunk `0x4040` (the light block) is applied only to a type-2 frame |
| `4` | **SOUND** | `sound_getvolume` (dispatcher case `0x13f`) bails unless the type is 4, then reads through `i3d_sound_get_volume_ptr` |
| `5` | **SECTOR** | `ispointinsector` (case `0x13c`) bails unless the type is 5, then calls the frame's own containment vtable slot `+0x68`. `frm_setpos` (case `0xf6`) tests the same value on the PARENT when deciding whether to re-home a frame |
| `6` | **DUMMY**, carries its bounding box (§1c) | `i3d_dummy_set_bound_box` stores it from scene chunk `0x4050` |
| `9` | **MODEL**, a model / driver frame | `actor_duplicate` (case `0xda`) requires it before cloning; the I3D driver is asked for type 9; `mission_actor_cache_store` parks only type-9 frames |
| `0xc` | **OCCLUDER** | scene chunk `0x4083` (a vertex and triangle list) is applied only to a type-12 frame |
| `0xe` | **AREA**, a scripted trigger volume | `ispointinarea` (case `0x13d`) bails unless the type is `0xe`, then calls containment vtable slot `+0x5c` |

A sector and an area are DIFFERENT objects with different containment slots. The shipped scripts
use the area form 36 times and the sector form not at all.

## 6. `+0x120` is the PARENT frame pointer

Four readers agree, and no other reading survives all four:

1. **`i3d_frame_get_scale` (`0x0045f090`)**: a two-armed branch that only makes sense for a parent.
   `+0x120 == 0` returns `scale` unchanged; `+0x120 != 0` refreshes the world matrix when the `0x20`
   valid bit is clear and extracts the scale out of it. **For a root frame world scale IS local
   scale**, the identity the null arm encodes.
2. **`frm_getparent`, dispatcher case `0x13b`**: the script command the Mafia Script Data
   Reference describes as "returns the parent of child xx" clears its destination register and then
   stores exactly this pointer.
3. **`frm_setpos`, case `0xf6`**: after moving a frame it re-registers it with its sector, but skips
   that whenever `+0x120` is non-null AND the object it points at has frame type != 5. Read as a
   parent this is "a frame parented to a non-sector inherits sector membership from its parent";
   read as anything else it is unexplainable.
4. **`actor_duplicate`, case `0xda`**: passes `*(+0x120)` of the source frame as the *parent*
   argument to the new driver object's vtable slot `+0x2c`.

## 7. The lightmap levels of an `'LMAP'` frame

A type-1 frame tagged `'LMAP'` carries one `S_lmap_level` per lightmap level at `+0x220` and the
index of the active level at `+0x21c` (§1e, §1f). The level is switched through frame slot
`+0xa0`: the script command `setlmlevel` (case `0x1e`) calls it for one frame and
`program_switch_light_map` for a whole subtree. Both test the level's `present` byte before
switching.

## 8. The mesh chain reached through `+0x1e0`

`+0x1e0` does not hold a generic "visual". It holds an **`I3D_mesh_object`**, and three one-line
header inlines in the `C_program` tail walk from it down to a material:

```
I3D_object (type 1, tag 'OBJ_' or 'LMAP')
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

**How the classes are established.** `glass_shatter_cut_mesh` walks the identical chain, and its
CONFIRMED twin names every hop (§9). The two counts fall out of `0x004fc8fd`, where each is the loop
bound of the indexed getter it guards.

## 9. PC vs PS2

- **The PS2 has no DLLs**, so it compiled LS3D in and its symbol table carries the classes' methods
  in the clear. That is the ONLY place their own method names exist (`basic/artifacts/dll.md`).
- **`I3D_scene::SetFrameSectorPos` ships in two overloads on the PS2**: the 220-byte virtual
  `…FP9I3D_frameRC8S_vector`, which does `GetSector(pos)` then `frame->LinkTo(sector, 1)`, and a
  48-byte inline `…RC8S_vectorRC8S_vector`. The PC's `i3d_scene_set_frame_sector_pos` is the thin
  inline forwarding to the virtual at slot `+0xbc`.
- **The FourCC and the frame type are VIRTUALS on the PS2 and plain FIELDS on the PC.** The PS2
  reads them as `(**(code **)(*frame + 0xc))(frame) == 0x5f4a424f`; the PC reads
  `*(frame + 0x1d4)`. A cross-build match in this area must not expect a call on both sides.
- The PS2 pair `SetModelName__9I3D_frame` (384 B) and `ResetModelName__9I3D_frame` (188 B) agrees
  with the `model_name` reading of `+0x154`: the frame carries exactly two name strings and the PS2
  class has exactly two name setters.
- `I3D_frame::GetDir` (380 B), `GetWorldPos` (56 B), `GetWorldDir` (348 B), `UpdateMatrix` (404 B)
  and `UpdateWMatrix` (44 B) are the PS2's own out-of-lined copies of the same header inlines §3
  lists, in a different host unit; no twin is stamped on them.
- **Field deltas for the mesh chain:** PC `face_group + 0x04` is PS2 `face_group + 0x00`, and PC
  `+0x08` (face count) is PS2 `+0x04`, because those classes are non-polymorphic on the PS2 (no
  `__vt__` symbol exists for any of them) and fully virtual on the PC.
- **PC vtable indices are NOT PS2 vtable indices** for this family. `I3D_frame`'s PC table is called
  at `+0x90` (index 36) by `game_scratch_recompile`, while `__vt__9I3D_frame` has only 29 virtuals:
  the PC interface is strictly larger. The one place the two agree slot for slot is `IShow`
  (`basic/artifacts/script-opcode-dispatch.md` §10), a COM-style interface on both builds.
- **`I3D_camera+0x140`** is PS2 `camera+0x120`; **`I3D_sound+0x18c`** is PS2 `frame+0x16c`;
  **`I3D_scene+0x210`** is PS2 `scene+0x1d8`.
- **`I3D_scene` derives from `I3D_frame` (§1a).** The PS2 `I3D_scene` class ships **eight-byte stub
  overrides** of exactly the `I3D_frame` virtuals a scene root has no use for (`LinkTo`, `SetDir`,
  `SetName`, `SetOn`, `Update`, `SetCallback`, `Duplicate`), which is what a derived class that
  suppresses its base's API looks like.
- **Flag bit `0x1` (§4):** the PS2 class's `SetOn__9I3D_frameFb` sets and clears exactly this bit of
  its flag word (PS2 `+0xe0`, the word both builds test for bit `0x20` before `UpdateWMatrixProc`).
- **The lightmap table (§7)** is PS2 `I3D_lit_object`'s: one `0x1c`-byte record per level at
  `+0x214` (the present byte, then three per-LOD lightmap pointers at record `+0x10`), the active
  level at `+0x210`, selected by `SetActiveLevel__14I3D_lit_objectFi` (PS2 vtable slot 58, reached
  on the PC at `+0xa0`). `SwitchLightMap__Q29C_program24S_help_4741C_program_cppFP9I3D_framei` tests
  the same present byte. The PC keeps ten LOD pointers per level where the PS2 keeps three.
- **Three accessors of §3:** the PS2 reaches `i3d_frame_get_child_count_ptr`'s quantity through the
  `GetNumChilds` virtual; `i3d_camera_get_fov_ptr`'s 180/π conversion is the same on both builds;
  the pointer `i3d_scene_get_root_frame_ptr` returns is passed to the same link virtual as the PS2's
  scene root.
- **The mesh chain (§8) is named from the PS2.** The PS2 `STREAM_CONNECT` arm is an empty stub, but
  `glass_shatter_cut_mesh` walks the identical chain and its PS2 twin `CutMesh__19C_Break_plain_glass…`
  is CONFIRMED, so laying the two side by side names every hop: `GetLOD__15I3D_mesh_objectCFi`,
  `LockVertices__14I3D_mesh_levelFUi`, `GetFGroup__14I3D_mesh_levelFi`, and `I3D_material::Duplicate`
  taking `*(u32 *)fgroup`. The PS2 hands `+0x1e0` to `GetLOD__15I3D_mesh_objectCFi` in the twins of
  `uv_scroll_channel_tick` and `glass_shatter_cut_mesh`, at the same `+0x1e0` on both builds.

**PS2 methods with no PC function** (grouped by class; counts from the PS2 symbol table's class
column):

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| - | the 48 methods of `I3D_frame` | the class body is in `ls3df.dll`; `Game.exe` holds only header inlines (§3) |
| - | the 73 methods of `I3D_scene` | the same |
| - | the 14 methods of `I3D_camera` | the same |
| - | the 27 methods of `I3D_sound` | the same |
| - | the 6 methods of `I3D_dummy` | the same |
| - | the 21 methods of `I3D_lit_object` | the same |
| - | the 7 methods of `I3D_mesh_object` | the same |
| - | the 17 methods of `I3D_mesh_level` | the same |
| - | the 6 methods of `I3D_face_group` | the same |
| - | the 14 methods of `I3D_object` | the same |

## 10. Related

- `basic/artifacts/dll.md` — what `ls3df.dll` exports and why the class bodies are not in
  `Game.exe`.
- `structs/actor.md` — `C_actor+0x68` holds the frame.
- `structs/program.md` §8 — the translation unit that hosts most of these inlines.
- `structs/mission.md` — the `I3D_scene` at `C_mission+0x10` and the scene file.
- `basic/artifacts/script-opcode-dispatch.md` — the dispatcher cases that reach these fields.

## 11. Open items

1. **`I3D_scene+0x17c` is [M].** The active-camera reading rests on the accessor's name, the camera
   chunks and two readers that use it as the camera's position (`cache_base_block_tick`'s streaming
   reference, `speech_channel_speak_line`'s 12-unit subtitle distance); no `ls3df.dll` writer was
   read.
2. **Direct field accesses.** The evidence covers the accessors, the named callers and the direct
   accesses found in the `cache_base_block_*`, `i3d_cache_*` and `speech_*` bodies; a `.text` census
   of direct `I3D_frame` field accesses (they share offsets with many game classes) was not made.
3. **`i3d_frame_match_deform_roof` reads its candidate's name at `+0xc`.** Every other reader takes
   a frame's name from `+0x100` (§1); the object the enumeration of `car_set_roof_visible` (frame
   slot `+0x34`, mask `0x10001`) hands the predicate is not identified: the body takes `+0x0c` as a
   `char *`, and the callback's signature is inside `ls3df.dll`.
4. **Fields whose meaning is inside `ls3df.dll`.** `unk_d4` (the last float of the bound volume),
   `I3D_sound` `unk_148` and the flag bits `0x100` and `0x40000000` have no consumer in `Game.exe`;
   their meaning is inside the DLL, outside this binary.
