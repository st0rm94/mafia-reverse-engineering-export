# `G_Camera` — the camera manager

**Created** 2026-09-01 22:14 +0200 · **Modified** 2026-11-16 21:19 +0100

The camera manager of the game: it owns the camera mode, drives the `I3D_camera`, follows the
player on foot and in a vehicle, and runs the dialogue, lock, look-to, sniper, death and race
cameras. PS2 class **`G_Camera`**, source file `g_camera.cpp`.

**It is not a singleton.** `G_Camera` is the sub-object **embedded at `C_game+0x4c`**, reached
through the address accessor `game_get_camera` (`0x00425510`), which is why every method is
`__thiscall` on that address and why a search for a global camera address never finds it. The
receiver is byte-proven twice: the instruction loading ECX before the single `E8` call to
`camera_get_race_camera_at` is `lea ecx, [eax + 0x4c]`, and the police arrest cutscene calls
`game_get_camera()` immediately before `camera_lock_at_human_pair`.

| | |
|---|---|
| PC prefix | `camera_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x98`: embedded at `C_game+0x4c`, and the next `C_game` member is at `+0xe4` (`structs/game.md`) |
| PC constructor | `camera_ctor` @ `0x005d48a0` |
| PC translation unit | `[0x005d4750, 0x005dc160)` (§6) |
| PS2 class | `G_Camera` (tag `8G_Camera`), 39 methods: 36 CONFIRMED twins, all body-diffed and mapped in §3; 3 PS2-only (§7a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

PC offsets relative to the `G_Camera` base (`C_game+0x4c`). No access to `+0x34..+0x4b` was
found in the `camera_*` functions.

```c
typedef struct G_Camera {                   // 0x98 bytes, embedded at C_game+0x4c
    /* +0x00 */ I3D_frame  *link_frame;         // [H] the frame camera_link_to_frame binds
    /* +0x04 */ I3D_camera *i3d_camera;         // [H] the camera being driven
    /* +0x08 */ C_human    *player;             // [H] the human the camera follows
    /* +0x0c */ C_car      *car;                // [H] the bound vehicle
    /* +0x10 */ uint32_t    mode;               // [H] the current mode (§4)
    /* +0x14 */ uint32_t    restore_mode;       // [H] the mode SetMode(-1) returns to
    /* +0x18 */ uint32_t    mode_flags;         // [H] the mode record's flag word
    /* +0x1c */ float      *mode_params;        // [H] the mode record's param[10] block
    /* +0x20 */ uint8_t     tick_suspended;     // [M] camera_tick returns at once while set
    /* +0x21 */ uint8_t     unused_21;          // [U] zeroed by camera_reset; nothing reads it
    /* +0x22 */ uint8_t     camera_inside;      // [H] mode flags bit 0; the player model is hidden while set
    /* +0x24 */ float       unread_24;          // [U] camera_reset stores 5.0; no reader (.text scan)
    /* +0x28 */ float       pitch;              // [H] game_tick's aim pitch every frame; read as half-turns
    /* +0x2c */ float       look_target;        // [H] look-around target angle, half-turns
    /* +0x30 */ float       look_current;       // [H] look-around current angle, half-turns
    /* +0x4c */ S_vector    up;                 // [H] the up vector the tick builds the view with
    /* +0x58 */ uint8_t     swing_enabled;      // [H] camera_set_swing
    /* +0x5c */ float       swing_amount;       // [H] camera_set_swing
    /* +0x60 */ I3D_frame  *swing_frame;        // [H] the swing reference frame
    /* +0x64 */ uint8_t     snap_next;          // [H] the next pose update cuts, not blends
    /* +0x68 */ vector      mode_stack;         // [H] G_CameraPushPop records (§1a)
    /* +0x78 */ vector      race_cameras;       // [H] G_RaceCamera pointers
    /* +0x88 */ I3D_frame  *look_to_frame;      // [H] the look-to target frame
    /* +0x8c */ float       aim_spread;         // [H] the player's shot_accuracy_spread; scales the sniper sway
    /* +0x90 */ float       sniper_fov;         // [M] 0.2 by camera_reset; the last scope FOV
    /* +0x94 */ uint8_t     free_look;          // [H] free look is enabled
} G_Camera;
```

The two vectors are the MSVC layout: an allocator byte, then begin / end / capacity
(`mode_stack` `+0x68` / `+0x6c` / `+0x70` / `+0x74`, `race_cameras` `+0x78` / `+0x7c` / `+0x80` /
`+0x84`). `camera_ctor` empties both; `camera_reset` empties `mode_stack` again.

**Why `mode_params` points at ten floats.** Each `g_camera_mode_table` record carries a `param`
block of ten floats (§4); `camera_begin_mode` and `camera_reset` store the address of the active
record's block here, and `camera_tick` reads entries 0-4 and 6-9 of it and `camera_apply_desired_pos` entries 8 and 9; entry 5 is never read. The count is the record
layout's; the meaning of each entry is per mode and is not decoded (§9).

### 1a. `G_CameraPushPop` — one saved mode

```c
typedef struct G_CameraPushPop {            // 0x10 bytes; mode_stack element
    /* +0x00 */ uint32_t mode;                  // [H] the saved mode
    /* +0x04 */ float    cam_param[3];          // [H] three saved I3D_camera parameters
} G_CameraPushPop;
```

`cam_param` holds the three values `camera_push_mode` reads from `frame+0x140` / `+0x144` /
`+0x148` of the driven `I3D_camera`; `camera_pop_mode` pushes them back through the camera's
vtable slots `0x50` and `0x58`. Three because the camera frame keeps exactly those three at
consecutive offsets. The type name is the PS2 template argument's.

### 1b. `G_CameraDialog` — one dialogue speaker

```c
typedef struct G_CameraDialog {             // 0x14 bytes; g_camera_dialog_actors element
    /* +0x00 */ C_human *speaker;               // [H] the speaker
    /* +0x10 */ int32_t  side;                  // [H] 1 or 2, alternating as speakers are added
} G_CameraDialog;
```

`+0x04..+0x0f` is not accessed by the camera code. The type name is the PS2 template argument's.

### 1c. `S_camera_mode_record` — one `g_camera_mode_table` record

```c
typedef struct S_camera_mode_record {       // 0x44 bytes; g_camera_mode_table element
    /* +0x00 */ int32_t  mode;                  // [H] the camera mode this record describes
    /* +0x04 */ int32_t  restore_mode;          // [H] the mode camera+0x14 takes from this record
    /* +0x08 */ float    param[10];             // [H] the mode's ten parameters; camera+0x1c points here
    /* +0x30 */ char     name[16];              // [H] the developers' label for the mode; no reader (.text scan)
    /* +0x40 */ uint32_t flags;                 // [H] the mode's flag word, copied to camera+0x18 (§4)
} S_camera_mode_record;
```

`g_camera_mode_table` (`0x00650450`) holds 17 of them (§4a). The decomp types the `param` block as
ten separate fields `param0`..`param9`, so the pointer that `camera_begin_mode` and `camera_reset`
store in `camera+0x1c` reads `&g_camera_mode_table[i].param0`.

## 2. Field evidence

Measured by a census of every `this+N` access in the `camera_*` functions and of every operand
with each displacement in them, then read in the bodies. The globals of §5 by a census of every
mention of each name in the decomp.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | link_frame | H | `camera_reset` (= 0), `camera_link_to_frame` | `camera_begin_mode` | the link also selects mode `0x11` |
| +0x04 | i3d_camera | H | `camera_reset` (= 0), `camera_set_i3d_camera` | `camera_tick`, `camera_apply_desired_pos`, `camera_apply_desired_pos_car`, `camera_begin_mode`, `camera_end_mode`, `camera_free_look`, `camera_free_look_base`, `camera_lock_at`, `camera_lock_at_human_pair`, `camera_push_mode`, `camera_pop_mode`, `camera_dialog_stop`, `camera_set_look_to`, `camera_set_sniper_fov` | every pose write goes to `frame+0x80..+0x88` |
| +0x08 | player | H | `camera_reset` (= 0), `camera_set_player` | `camera_tick`, `camera_apply_desired_pos`, `camera_begin_mode` | - |
| +0x0c | car | H | `camera_reset` (= 0), `camera_set_car` | `camera_get_car`, `camera_tick`, `camera_apply_desired_pos_car`, `camera_begin_mode`, `camera_end_mode` | `camera_end_mode` re-shows the driver model through it. The `car_setspeed` script arm reads it through `game_get_camera` and re-binds the camera when it equals the car just reset |
| +0x10 | mode | H | `camera_reset`, `camera_set_car`, `camera_set_mode`, `camera_set_climbing_into_car` | `camera_tick`, `camera_begin_mode`, `camera_end_mode`, `camera_enable_free_look`, `camera_lock_at`, `camera_lock_at_human_pair`, `camera_unlock`, `camera_push_mode`, `camera_dialog_look`, `camera_dialog_start`, `camera_dialog_stop`, `camera_is_in_dialog`, `camera_set_sniper_fov`, `camera_set_sniper_mode`, `camera_set_climbing_into_car` | every mode predicate tests it (§4) |
| +0x14 | restore_mode | H | `camera_reset`, `camera_begin_mode` (record `+0x04`) | `camera_set_mode` | `camera_set_mode(-1)` reads it back |
| +0x18 | mode_flags | H | `camera_begin_mode` (record `+0x40`) | `camera_tick`, `camera_begin_mode` | bit 1 hides the player model, bit 2 suppresses the fade reset and the `g_camera_look_from_eyes` clear (§4) |
| +0x1c | mode_params | H | `camera_reset`, `camera_begin_mode` | `camera_tick`, `camera_apply_desired_pos` | §1 |
| +0x20 | tick_suspended | M | `camera_reset` (= 0) | `camera_tick`, `camera_apply_desired_pos` | `camera_tick` returns at once while it is set, and also in mode `0x18` while it is clear. No instruction in `.text` sets it (§9) |
| +0x21 | unused_21 | U | `camera_reset` (= 0) | - | §9 |
| +0x22 | camera_inside | H | `camera_reset` (= 0), `camera_begin_mode` (record `flags` bit 0) | `human_intern_use_car`, `human_update`, `human_put_in_vehicle_at_seat`, `player_ai`, `human_intern_use_railway`, `player_set_sniper_mode` (all as `C_game +0x6e`) | no reader in the `camera_*` functions; the first four copy it into the car's `+0x57c` (`C_Vehicle +0x50c` `camera_inside_request`, `structs/vehicle.md` §2). The player model is hidden while it is set: `player_ai` hides it, and `player_set_sniper_mode` (entering and leaving the scope) and `human_intern_use_railway` (the player leaving a rail vehicle) switch the model frame on only while it is clear (frame virtual `+0x24`). Only the records `P In`, `CP Sniper`, `C RailwayIn` and mode `0x17` set it (§4a) |
| +0x24 | unread_24 | U | `camera_reset` (= 5.0) | - | no reader: none in the `camera_*` bodies, and no access to `C_game +0x70` in `.text` has a `C_game` base |
| +0x28 | pitch | H | `camera_reset` (= 0.5), `game_tick` (as `C_game +0x74`) | `camera_tick` | `game_tick` stores the player's `aim_pitch` minus `shot_accuracy_spread` × 0.1 (0.5 with no player) every frame, right before it calls `camera_tick`; `camera_tick` multiplies it by π and passes it to `sin` / `cos` in modes 1, 2, 3, 5, `0x11` and `0x17` |
| +0x2c | look_target | H | `camera_reset`, `camera_look_around` | `camera_tick` | half-turns; `camera_tick` moves `look_current` toward it at `dt × 0.008` |
| +0x30 | look_current | H | `camera_reset`, `camera_look_around`, `camera_tick` | `camera_look_around`, `camera_tick` | wrapped by ±2.0 in `camera_look_around` |
| +0x4c | up | H | `camera_tick` ((0, 1, 0) in several modes) | `camera_tick` (by address) | - |
| +0x58 | swing_enabled | H | `camera_reset` (= 0), `camera_set_swing` | `camera_tick`, `camera_apply_desired_pos`, `camera_pop_mode` | - |
| +0x5c | swing_amount | H | `camera_set_swing` | `camera_tick`, `camera_apply_desired_pos` | `camera_apply_desired_pos` tilts the up vector by `sin(g_camera_swing_phase · π) × swing_amount` |
| +0x60 | swing_frame | H | `camera_reset` (= 0), `camera_set_swing` | `camera_set_swing`, `camera_tick`, `camera_apply_desired_pos` | - |
| +0x64 | snap_next | H | `camera_set_car`, `camera_set_climbing_into_car`, `camera_tick`, `camera_snap_next_update` (= 1); `camera_begin_mode`, `camera_pop_mode(1)` and the end of the tick | `camera_tick`, `camera_apply_desired_pos` | ¹ |
| +0x68 | mode_stack | H | `camera_ctor`, `camera_dtor`, `camera_reset`, `camera_push_mode`, `camera_pop_mode` | `camera_push_mode`, `camera_pop_mode`, `camera_game_done`, `camera_dtor` | stride `0x10` (§1a) |
| +0x78 | race_cameras | H | `camera_ctor`, `camera_dtor`, `camera_add_race_camera`, `camera_remove_race_camera` | `camera_tick`, `camera_add_race_camera`, `camera_remove_race_camera`, `camera_init_race_cameras`, `camera_get_race_camera_at` | stride 4 (§4b) |
| +0x88 | look_to_frame | H | `camera_set_look_to` | `camera_tick` | - |
| +0x8c | aim_spread | H | `camera_reset` (= 0), `player_ai`, `player_add_shot_accuracy_penalty` (both as `C_game +0xd8`) | `camera_tick`, `camera_apply_desired_pos` | the two player writers store the player's `shot_accuracy_spread` (`structs/player.md`); in mode 5 the sniper sway amplitude is `(2 × aim_spread + 1) × (1 - crouch × 0.5) × 0.0015`; `camera_apply_desired_pos` offsets the look target by the player's forward vector × `aim_spread` in its swing branch only |
| +0x90 | sniper_fov | M | `camera_reset` (= 0.2), `camera_set_sniper_fov` | - | `camera_set_sniper_fov` also applies the value at once through the camera's FOV slot `0x50`; nothing reads the stored copy (§9) |
| +0x94 | free_look | H | `camera_set_car`, `camera_enable_free_look`, `camera_reset` (= 0) | `camera_apply_desired_pos`, `camera_apply_desired_pos_car`, `camera_enable_free_look` | - |

¹ `camera_apply_desired_pos` skips its blend with `g_camera_prev_pos` / `g_camera_prev_dir`, and
`camera_tick` forces the per-mode lerp factor to 1.0, zeroes `g_camera_stability_factor` and the
three in-car direction statics `g_camera_car_old_dir_y_mode_7_9` / `_0xa` / `_0xb` (§5), and drops
the velocity-lead term. `camera_snap_next_update` has a single caller, the end of `game_init`, so a
fresh scene does not drift in from the previous mission's pose.

### 2a. `G_CameraPushPop`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | mode | H | `camera_push_mode` | `camera_pop_mode` | - |
| +0x04 | cam_param | H | `camera_push_mode` | `camera_pop_mode` | §1a |

### 2b. `G_CameraDialog`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | speaker | H | `camera_dialog_start` | `camera_dialog_look`, `camera_tick` | - |
| +0x10 | side | H | `camera_dialog_start` | `camera_tick` | which shoulder the dialogue camera takes, with `g_camera_dialog_side` |

### 2c. `S_camera_mode_record`

The table is static `.data`; nothing in the image stores to it. A scan of the image for every dword
in `[0x00650450, 0x006508d4)` finds only `camera_begin_mode`'s pointers (to `+0x00`, `+0x04`,
`+0x08` and `+0x40` of record 0, then indexed by `0x44`) and `camera_reset`'s (to `+0x08` of
record 0); the other three hits are `call` displacements.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | mode | H | - | `camera_begin_mode` | the key of the record scan |
| +0x04 | restore_mode | H | - | `camera_begin_mode` | copied to `restore_mode` (`+0x14`) |
| +0x08 | param | H | - | `camera_begin_mode`, `camera_reset` (by address); `camera_tick`, `camera_apply_desired_pos` (through `mode_params`) | entries 0-4 and 6-9 are read, entry 5 never (§1) |
| +0x30 | name | H | - | - | no reader: no pointer into `+0x30..+0x3f` of any record |
| +0x40 | flags | H | - | `camera_begin_mode` | copied to `mode_flags` (`+0x18`); bit 0 to `camera_inside` (`+0x22`) |

## 3. Method family

Coverage: **36 of 39** PS2 `8G_Camera` methods are in the tables below, all **CONFIRMED** and
body-diffed; the other three (`CalcUpDown`, `HardLockAt`, `HardUnlock`) have no PC function (§7a).
`camera_get_race_camera_at` is PC-only. The static-init rows of §3a belong to the translation unit.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005d4750 | `crt_static_init_guard_005d4750` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x005d4770 | `crt_atexit_register_005d4770` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x005d4780 | `thunk_stub_noop_cdecl_005d4790` | the translation unit | - | - | - | a `.CRT$XCU` entry: a 5-byte `jmp` to the empty initializer below ¹ |
| 0x005d4790 | `stub_noop_cdecl_005d4790` | the translation unit | - | - | - | an empty dynamic initializer of a file-scope object |
| 0x005d47a0 | `thunk_stub_noop_cdecl_005d47b0` | the translation unit | - | - | - | the same shape ¹ |
| 0x005d47b0 | `stub_noop_cdecl_005d47b0` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x005d47c0 | `thunk_stub_noop_cdecl_005d47d0` | the translation unit | - | - | - | the same shape ¹ |
| 0x005d47d0 | `stub_noop_cdecl_005d47d0` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x005d47e0 | `thunk_stub_noop_cdecl_005d47f0` | the translation unit | - | - | - | the same shape ¹ |
| 0x005d47f0 | `stub_noop_cdecl_005d47f0` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x005d4800 | `camera_dialog_actors_static_init` | `g_camera_dialog_actors` | - | - | - | the vector's `.CRT$XCU` entry ² |
| 0x005d4810 | `camera_dialog_actors_static_ctor` | `g_camera_dialog_actors` | - | - | - | constructs the empty vector |
| 0x005d4830 | `camera_dialog_actors_register_atexit_dtor` | `g_camera_dialog_actors` | - | - | - | hands the destructor to `crt_atexit` by address |
| 0x005d4840 | `camera_dialog_actors_static_dtor` | `g_camera_dialog_actors` | - | - | - | frees the vector at exit; referenced only by that address push |
| 0x005d4860 | `thunk_stub_noop_cdecl_005d4870` | the translation unit | - | - | - | a `.CRT$XCU` entry (`0x00646268`) ¹ |
| 0x005d4870 | `stub_noop_cdecl_005d4870` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x005d4880 | `thunk_stub_noop_cdecl_005d4890` | the translation unit | - | - | - | a `.CRT$XCU` entry (`0x0064626c`) ¹ |
| 0x005d4890 | `stub_noop_cdecl_005d4890` | the translation unit | - | - | - | an empty dynamic initializer |
| 0x005d48a0 | `camera_ctor` | instance | `__ct__8G_CameraFv` | 0x00290d50 | CONFIRMED | empties both vectors, then `camera_reset` (§7e) |
| 0x005d4910 | `camera_dtor` | instance | `__dt__8G_CameraFv` | 0x00290df0 | CONFIRMED | frees `race_cameras` and `mode_stack` |

¹ Ghidra names a thunk after its target, so the six thunks carry `thunk_FUN_<target>` ids, not
`FUN_` ids of their own.
² The quartet is compiler-emitted CRT scaffolding. No PS2 counterpart can exist: the PS2 emits one
`__sinit_<file>.cpp` per source file rather than a per-object quartet. The spelling is the house
form of the other static-init families.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005c8ae0 | `camera_get_car` | - | - | - | returns `car` (`+0x0c`); in the `0x005c8xxx` helper tail, outside the unit |
| 0x005ee210 | `camera_snap_next_update` | - | - | - | `mov byte [ecx+0x64], 1; ret`; outside the unit; its one caller is the end of `game_init` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005d4950 | `camera_reset` | `Reset__8G_CameraFv` | 0x00290e80 | CONFIRMED | every field to its default; the mode from `g_camera_last_onfoot_mode` |
| 0x005d4a30 | `camera_game_done` | `GameDone__8G_CameraFv` | 0x00290f30 | CONFIRMED | empties `mode_stack` |
| 0x005d4a60 | `camera_set_i3d_camera` | `SetCamera__8G_CameraFP10I3D_camera` | 0x00290f70 | CONFIRMED | stores `i3d_camera` |
| 0x005d4a80 | `camera_set_player` | `SetPlayer__8G_CameraFP7C_human` | 0x00290fb0 | CONFIRMED | stores `player` |
| 0x005d4aa0 | `camera_set_car` | `SetCar__8G_CameraFP5C_car` | 0x00290ff0 | CONFIRMED | stores `car`, picks the vehicle or the on-foot mode, sets `snap_next` |
| 0x005d4af0 | `camera_link_to_frame` | `Link__8G_CameraFP9I3D_frame` | 0x00291060 | CONFIRMED | stores `link_frame` and picks mode `0x11` (or the last on-foot mode on unlink) |
| 0x005d4b20 | `camera_set_swing` | `SetSwing__8G_CameraFbfP9I3D_frame` | 0x002910b0 | CONFIRMED | the three swing fields |
| 0x005d4c40 | `camera_look_around` | `LookAround__8G_CameraFi` | 0x002911d0 | CONFIRMED | sets `look_target` from a selector (§7b) |
| 0x005d4cf0 | `camera_is_look_from_eyes` | `LookFromEyes__8G_CameraFv` | 0x002912a0 | CONFIRMED | reads the global `g_camera_look_from_eyes` |
| 0x005d4d10 | `camera_set_mode` | `SetMode__8G_CameraFUib` | 0x002912d0 | CONFIRMED | ends the old mode, stores the new one (or `restore_mode` for -1), begins it, latches the last on-foot / vehicle mode |
| 0x005d4d70 | `camera_set_climbing_into_car` | `LezeDoToyoty__8G_CameraFb` | 0x002913d0 | CONFIRMED | forces mode 7 while the player climbs in, and sets `snap_next` |
| 0x005d4dd0 | `camera_tick` | `Tick__8G_CameraFUi` | 0x002916c0 | CONFIRMED | the per-mode pose computation (§4, §5) |
| 0x005d9710 | `camera_apply_desired_pos_car` | `WantPosCar__8G_CameraFRC8S_vectorRC8S_vectorf` | 0x00296330 | CONFIRMED | the in-car blend into the `I3D_camera` |
| 0x005d9b90 | `camera_apply_desired_pos` | `WantPos__8G_CameraFRC8S_vectorRC8S_vectorRC8S_vectorf` | 0x002966f0 | CONFIRMED | the general blend, the swing tilt, the eye blend |
| 0x005da4b0 | `camera_begin_mode` | `Begin__8G_CameraFv` | 0x00297110 | CONFIRMED | finds the mode's record, stores `mode_params` / `mode_flags` / `restore_mode`, runs the mode's entry arm |
| 0x005daa60 | `camera_end_mode` | `End__8G_CameraFv` | 0x00297860 | CONFIRMED | the mode's exit arm (the driver model, the HUD after the scope) |
| 0x005daad0 | `camera_enable_free_look` | `EnableFreeLook__8G_CameraFbb` | 0x00297990 | CONFIRMED | sets `free_look`; restores mode 7 from modes `0xc`..`0xf` |
| 0x005dab30 | `camera_free_look` | `FreeLook__8G_CameraFff` | 0x002979f0 | CONFIRMED | applies the two free-look deltas |
| 0x005dae20 | `camera_free_look_base` | `FreeLookBase__8G_CameraFffR8S_vectorR8S_vectorR8S_vector` | 0x00297c20 | CONFIRMED | the shared free-look core (§7b) |
| 0x005db000 | `camera_lock_at` | `LockAt__8G_CameraFRC8S_vectorRC8S_vector` | 0x00297e80 | CONFIRMED | enters mode `0x16` at a given pose |
| 0x005db080 | `camera_lock_at_human_pair` | `CumNaNe__8G_CameraFP7C_humanP7C_human` | 0x00297f60 | CONFIRMED | enters mode `0x16` framing two humans; FOV π/4 through the camera's slot `0x50` |
| 0x005db5d0 | `camera_unlock` | `Unlock__8G_CameraFb` | 0x00298580 | CONFIRMED | leaves mode `0x16` |
| 0x005db5f0 | `camera_push_mode` | `PushMode__8G_CameraFv` | 0x002986c0 | CONFIRMED | appends a `G_CameraPushPop` (§7e) |
| 0x005db820 | `camera_pop_mode` | `PopMode__8G_CameraFb` | 0x00298730 | CONFIRMED | restores the last record; `pop_mode(1)` clears `snap_next` |
| 0x005db8e0 | `camera_dialog_look` | `DialogLook__8G_CameraFP7C_humanP7C_human` | 0x00298820 | CONFIRMED | sets `g_camera_dialog_look_from` / `_to` |
| 0x005db970 | `camera_dialog_start` | `DialogStart__8G_CameraFPP7C_humanUi` | 0x002988f0 | CONFIRMED | fills `g_camera_dialog_actors`, enters mode `0x13` |
| 0x005dbc50 | `camera_dialog_stop` | `DialogStop__8G_CameraFv` | 0x00298cd0 | CONFIRMED | leaves the dialogue mode |
| 0x005dbc90 | `camera_is_in_dialog` | `IsInDialog__8G_CameraFv` | 0x00298d80 | CONFIRMED | `mode == 0x13` |
| 0x005dbca0 | `camera_add_race_camera` | `AddRaceCamera__8G_CameraFP12G_RaceCamera` | 0x00298d90 | CONFIRMED | appends to `race_cameras` (§4b, §7e) |
| 0x005dbe70 | `camera_remove_race_camera` | `RemoveRaceCamera__8G_CameraFP12G_RaceCamera` | 0x00298e20 | CONFIRMED | erases from `race_cameras` |
| 0x005dbed0 | `camera_init_race_cameras` | `InitRaceCameras__8G_CameraFv` | 0x00298e90 | CONFIRMED | pairs the registered race cameras with the circuit's gates |
| 0x005dc000 | `camera_set_look_to` | `SetLookTo__8G_CameraFRC8S_vectorP9I3D_frame` | 0x00299050 | CONFIRMED | stores `look_to_frame`, enters mode `0x15` |
| 0x005dc070 | `camera_set_sniper_fov` | `SetSniperFov__8G_CameraFf` | 0x002990f0 | CONFIRMED | in mode 5, stores `sniper_fov` and applies it through the camera's slot `0x50` |
| 0x005dc0b0 | `camera_set_sniper_mode` | `SetSniperMode__8G_CameraFb` | 0x00299150 | CONFIRMED | pushes mode 5 or pops it |
| 0x005dc0f0 | `camera_get_race_camera_at` | - | - | - | PC-only; the race camera at an index; its one consumer is `net_session_partition_peers_by_relevance` (§7a) |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005dc120 | `copy_range_stride_0x14` | - | - | - | a shared STL copy with 22 callsites across at least six unrelated families, which is why it keeps a mechanical name; four of those callsites load `g_camera_dialog_actors_alloc`, the second proof of that vector's base address |

## 4. The mode map

Every mode value below is pinned by the function that sets or tests it, not inferred.

| Mode | Meaning | Pinned by |
|---|---|---|
| `1`..`3` | on foot | latched into `g_camera_last_onfoot_mode` by `camera_set_mode` |
| `5` | **sniper scope** | `camera_set_sniper_mode` pushes it; `camera_set_sniper_fov` only acts in it; `camera_end_mode` restores the HUD when leaving it |
| `6` | **death camera** | set by `human_death` and twice by `human_movement_intern`; `camera_tick` interpolates towards the victim's `neck` frame through `g_camera_death_interp` / `_pos` / `_dest_pos` |
| `7`..`0xf` | in a vehicle | latched into `g_camera_last_vehicle_mode`; `7` is also what `camera_set_climbing_into_car(true)` forces and what `camera_enable_free_look` restores from modes `0xc`..`0xf` |
| `0x11` | linked to a frame | set by `camera_link_to_frame`; suppresses the whole dialogue camera |
| `0x13` | **dialogue** | set by `camera_dialog_start`, tested by `camera_is_in_dialog` |
| `0x14` | race camera | set from `racing_set_race_camera_index` (§4b) |
| `0x15` | look-to | set by `camera_set_look_to` |
| `0x16` | **locked** | entered by `camera_lock_at` and `camera_lock_at_human_pair`, left by `camera_unlock` |
| `0x17` | **riding on a truck's cargo bed** | set by `human_intern_use_car_truck` when the human is the player, right after `camera_link_to_frame` binds the camera to the truck's frame; shares `camera_end_mode`'s teardown with `0x11` |
| `0x18` | **menu camera** — the owner drives the `I3D_camera` itself | set by `carcyclopedia_ctor` and `gui_menu_execute_modal`, each after `camera_push_mode`; `camera_tick` returns at once in this mode. It has no `g_camera_mode_table` record |

The table is complete: a byte scan of `.text` finds 17 calls and jumps to `camera_set_mode`, and
together with the direct stores to `+0x10` in `camera_reset`, `camera_set_car` and
`camera_set_climbing_into_car` they produce exactly the modes above.

**`g_camera_mode_table` (`0x00650450`) is the table behind all of this**: 17 records of `0x44`
bytes, scanned by `camera_begin_mode` up to `0x006508d4`; the record is `S_camera_mode_record`
(§1c, §2c).

`flags`: bit 0 goes to `camera_inside` (`camera+0x22`), bit 1 hides the player model, and bit 2 suppresses the fade
reset and the `g_camera_look_from_eyes` clear.

### 4a. The 17 records

| # | Mode | Restore | `name` | Flags |
|---|---|---|---|---|
| 0 | `0x01` | `0x00` | `P Behind` | `0x4` |
| 1 | `0x03` | `0x01` | `P In` | `0x5` |
| 2 | `0x06` | `0x00` | `P Behind` | `0x0` |
| 3 | `0x0b` | `0x0e` | `C Beh. Hellboy` | `0x0` |
| 4 | `0x07` | `0x08` | `C Beh. Hellboy` | `0x0` |
| 5 | `0x08` | `0x09` | `C Behind Emeth` | `0x0` |
| 6 | `0x09` | `0x0c` | `C Behind3` | `0x0` |
| 7 | `0x0c` | `0x0d` | `C Chase` | `0x2` |
| 8 | `0x0d` | `0x0e` | `C In` | `0x0` |
| 9 | `0x0e` | `0x07` | `C Over` | `0x0` |
| 10 | `0x11` | `0x00` | `C RailwayIn` | `0x1` |
| 11 | `0x17` | `0x00` | `P Behind` | `0x1` |
| 12 | `0x05` | `0x00` | `CP Sniper` | `0x1` |
| 13 | `0x15` | `0x00` | `C LookTo` | `0x8` |
| 14 | `0x16` | `0x00` | `C Lock` | `0x8` |
| 15 | `0x13` | `0x00` | `C Dialog` | `0x8` |
| 16 | `0x14` | `0x07` | `C Racing` | `0x0` |

The in-car records' `restore` values chain the camera-change key through the vehicle modes:
`0x07 -> 0x08 -> 0x09 -> 0x0c -> 0x0d -> 0x0e -> 0x07`.

**Dead code.** Modes `2`, `0xa` and `0x12` have arms in `camera_tick` (§5) but no record, and none
of the setters above ever produces them — the restore chain, the pushed modes and the two latches
only carry values that were set first — so those arms never run. Flag bit `0x8` (records 13..15)
has no reader: `camera_begin_mode` and `camera_tick` test bits `0x1`, `0x2` and `0x4` only, and a
scan of the whole of `.text` finds no `test` of bit `0x8` at `+0x18`.

### 4b. Race cameras in the retail game

The race-camera path runs in mission 6. Its `GameInitStart` script calls
`racing_mission6_init 5, 12, 0, 44`, and `race_mission6_init` constructs `g_racing`. The mission's
`scene2.bin` carries 26 `C_racecam` actors (kind `0x1c`). `camera_add_race_camera` has one caller,
`racecam_game_init`, which registers the actor's record only while `g_racing` is set, and
`camera_init_race_cameras` (called from `racing_game_init`) only pairs the registered
records with the circuit's gates. `racing_tick` then steps the camera to the next racer with
`racing_set_race_camera_index` (mode `0x14`) whenever its dwell timer expires.
`racecam_game_done` unregisters through `camera_remove_race_camera` without testing `g_racing`, so
in every other mission the removal runs against an empty vector. **Confirmed in game:** after the
race finishes, the player watches the other racers from these trackside cameras.

## 5. File-scope globals

Every one is pinned by a statement that matches on both builds.

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_camera_stability_factor` | `0x00650438` | settle weight, `+= dt × 0.003` / `-= dt × 0.01`, clamped to 1.0 | `camera_tick` | `camera_tick` |
| `g_camera_dialog_side` | `0x00650448` | ±1.0, which shoulder the dialogue camera sits over | `camera_dialog_look` | `camera_tick` |
| `g_camera_mode_table` | `0x00650450` | the 17 × `0x44` mode descriptor table (§4) | - | `camera_begin_mode`, `camera_reset` (by address) |
| `g_aiming_sensitivity_x` | `0x00672488` | Controls-screen slider, textdb 222 | `gui_options_controls_tick`, `gui_options_audio_video_tick` | `camera_free_look_base`, `camera_tick`, `player_ai_walk`, the Controls screen |
| `g_aiming_sensitivity_y` | `0x0067248c` | Controls-screen slider, textdb 223 | `gui_options_controls_tick`, `gui_options_audio_video_tick` | `camera_free_look_base`, `camera_tick`, `player_ai_walk`, the Controls screen |
| `g_mouse_control_inverted` | `0x006724ce` | Controls-screen toggle, textdb 257 / 255 / 256 | `gui_options_controls_tick` | `camera_free_look_base`, `player_ai_walk`, the Controls screen |
| `g_camera_frame_time_filter_ring` | `0x006f8f38` | 30 records × `0xc`, ends exactly at `0x006f90a0` | `camera_tick` (by address) | `camera_tick` (by address) |
| `g_camera_frame_time_filter_cursor` | `0x006f9148` | the ring cursor, wraps at `0x1d` | `camera_tick` | `camera_tick` |
| `g_camera_death_interp` | `0x006f90a0` | death-camera interpolation parameter | `camera_begin_mode`, `camera_tick` | `camera_tick` |
| `g_camera_death_dest_pos` | `0x006f90a4` | death-camera destination | `camera_begin_mode`, `camera_tick` | `camera_begin_mode`, `camera_tick` |
| `g_camera_death_pos` | `0x006f90b0` | death-camera current position | `camera_begin_mode` | `camera_begin_mode`, `camera_tick` |
| `g_camera_dialog_actors_alloc` / `_begin` / `_end` / `_cap` | `0x006f90bc` | the `G_CameraDialog` vector (§1b) | `camera_dialog_actors_static_ctor`, `camera_dialog_actors_static_dtor`, `camera_dialog_start` | `camera_dialog_look`, `camera_dialog_start`, `camera_dialog_actors_static_dtor` |
| `g_active_race_camera` | `0x006f90cc` | the active race camera | `camera_tick`, `camera_remove_race_camera` | `camera_tick`, `camera_remove_race_camera` |
| `g_camera_behind_pos` | `0x006f90d0` | the third-person pose | `camera_apply_desired_pos` | `camera_tick` |
| `g_camera_eye_dir` | `0x006f90dc` | the first-person view direction | `camera_tick` | `camera_tick` |
| `g_camera_eye_pos` | `0x006f90e8` | the first-person eye position, `+1.65` standing height | `camera_tick` | `camera_apply_desired_pos`, `camera_tick` |
| `g_camera_prev_distance` | `0x006f90fc` | last frame's distance | `camera_apply_desired_pos` | `camera_apply_desired_pos` |
| `g_camera_prev_dir` | `0x006f910c` | last frame's direction | `camera_apply_desired_pos`, `camera_tick` | `camera_apply_desired_pos`, `camera_tick` |
| `g_camera_prev_pos` | `0x006f9118` | last frame's position | `camera_apply_desired_pos`, `camera_tick` | `camera_apply_desired_pos`, `camera_tick` |
| `g_camera_swing_phase` | `0x006f9124` | `+= dt × 0.3`, wraps at 2.0; tilts the up vector by `sin(phase × π) × swing_amount` | `camera_tick` | `camera_apply_desired_pos`, `camera_tick` |
| `g_camera_crouch_eye_drop` | `0x006f9128` | eye height is `1.65 - this` | `camera_tick` | `camera_tick` |
| `g_camera_rot_in_act` | `0x006f912c` | zeroed by `camera_begin_mode` case `0xd` (head cam) | `camera_begin_mode` | - |
| `g_camera_look_from_eyes` | `0x006f9130` | the first-person request flag | `camera_begin_mode`, `camera_reset`, `camera_tick` | `camera_is_look_from_eyes`, `camera_tick` |
| `g_camera_eye_blend` | `0x006f9134` | weights the behind pose against the eye pose | `camera_tick` | `camera_apply_desired_pos`, `camera_tick` |
| `g_camera_dialog_look_from` / `_to` | `0x006f9138` / `0x006f913c` | the two speakers | `camera_dialog_look`, `camera_dialog_start`, `camera_dialog_stop` | `camera_tick` |
| `g_camera_free_look_yaw` / `_pitch` | `0x006f9140` / `0x006f9144` | free-look accumulators | `camera_enable_free_look`, `camera_free_look_base` | `camera_free_look_base` |
| `g_camera_car_sway_filtered` | `0x006f8f10` | modes `0xc` / `0xd` / `0xe`: the car's summed wheel vectors in car space, low-passed 0.9 old / 0.1 new; `y` forced to 1.0 before `SetDir3` | `camera_tick` | `camera_tick` |
| `g_camera_car_prev_velocity` | `0x006f8f1c` | a copy of `C_car+0x1f7c` written at the end of the same block every frame | `camera_tick` | - |
| `g_camera_tick_static_dtor_guard` | `0x006f8f28` | bit 0 / bit 1 guard registering the atexit destructors of the two vectors above | `camera_tick` | `camera_tick` |
| `g_camera_car_old_dir_y_mode_7_9` | `0x006f8f34` | modes 7/8/9: the look-direction Y, smoothed through `g_camera_stability_factor` | `camera_tick` | `camera_tick` |
| `g_camera_car_old_dir_y_mode_0xa` | `0x006f8f30` | mode `0xa`: the same Y, copied unsmoothed; the arm never runs (§4a) | `camera_tick` | - |
| `g_camera_car_old_dir_y_mode_0xb` | `0x006f8f2c` | mode `0xb`: smoothed like the mode 7/8/9 value | `camera_tick` | `camera_tick` |
| `g_camera_dir_yaw` | `0x006f9158` | mode 2: integrates the filtered mouse X × `g_aiming_sensitivity_x`, wrapped to [0, 2π); the arm never runs (§4a) | `camera_tick` | `camera_tick` |
| `g_camera_dir_pitch` | `0x006f915c` | mode 2: the pitch as a fraction of π, clamped to 0.01..0.99; the arm never runs (§4a) | `camera_tick` | `camera_tick` |
| `g_camera_sniper_sway_phase_pitch` | `0x006f9150` | mode 5: `+= dt × 2.5`, wrapped; its sine is added to the view's vertical component | `camera_tick` | `camera_tick` |
| `g_camera_sniper_sway_phase_yaw` | `0x006f9154` | mode 5: `+= dt × 3.0`, wrapped; its sine offsets the eye horizontally | `camera_tick` | `camera_tick` |
| `g_camera_orbit_yaw` | `0x006f914c` | mode `0x12`: `+= dt × 0.0002`, wrapped; the camera circles the target 2 m up; the arm never runs (§4a) | `camera_tick` | `camera_tick` |
| `g_camera_last_onfoot_mode` / `_last_vehicle_mode` | `0x006724a4` / `0x006724a8` | the last on-foot / vehicle mode (§4) | `camera_set_mode`, `loadsave_all_default` | `camera_reset`, `camera_set_car`, `camera_link_to_frame` (on foot); `camera_set_climbing_into_car` (vehicle) |

`DAT_005d96f0` / `DAT_005d9700` are **not globals**: they are code inside `.text`, the atexit
destructors of the two `camera_tick` static vectors, passed to `crt_atexit` by address under
`g_camera_tick_static_dtor_guard`.

## 6. The translation unit

`.CRT$XCU` at `.data:0x00646250` lists `0x5d4750, 0x5d4780, 0x5d47a0, 0x5d47c0, 0x5d47e0,
0x5d4800, 0x5d4860, 0x5d4880`, then `0x5dc160`, the next unit's guard. The unit is therefore
**`[0x005d4750, 0x005dc160)`**: its CRT pair, four empty static initializers, the
`g_camera_dialog_actors` quartet and two more empty initializers (all §3a), then the `G_Camera`
body `[0x005d48a0, 0x005dc160)` — 36 `G_Camera` methods, `camera_get_race_camera_at` and
`copy_range_stride_0x14`. The guard at `0x005d4720` before it opens a unit that holds nothing but
its own CRT pair.

**No phantom function starts.** All non-CRT starts of the body were tested for being a function's
early-exit block misread as a start (no `E8` callers, no data reference, branched into from
earlier code, no aligned padding, restore bytes instead of a prologue): every one is real, each
either 16-aligned with padding or carrying direct `E8` callers. The order alignment against the
PS2 corroborates it (§3: 38 PC functions against 39 PS2 methods, forced 1:1 in every gap).

## 7. PC vs PS2

### 7a. The three PS2 methods with no PC function, and the one PC function with no PS2 method

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x00291490 | `CalcUpDown__8G_CameraFRC8S_vector` | Its only PS2 caller is `AI_walk__8C_playerFUiP17G_ConsoleControls`. The PC twin of that caller, `player_ai_walk` (`0x004c6ed0`), carries the **same `[0.1, 0.85]` clamp at the same point**, fed from `g_aiming_sensitivity_y × mouse_delta` instead of from the aim frame's geometry: the PS2 computes an aim pitch from the crosshair frame because it is driving a gamepad; the PC reads the mouse. The computation does not exist on the PC, so this is not an inlining. A candidate, because its caller is not itself a PS2-only subsystem |
| 0x002985b0 | `HardLockAt__8G_CameraFRC8S_vectorRC8S_vector` | **Its consumer is absent too.** PS2 `Tick` opens a branch on `camera+0xa5` that copies the two hard-lock vectors from `+0xb0` and `+0xc0`, and PS2 `SetMode` gates its entire body on the same `+0xa5`. The PC's `camera_tick` has no reference to the corresponding `+0x95`, and `camera_set_mode` has no such gate. A feature whose flag, setters and only reader are all absent was never in the PC source |
| 0x002986b0 | `HardUnlock__8G_CameraFv` | the same feature as `HardLockAt` |

**`camera_get_race_camera_at` is the mirror case — a PC function with no PS2 method.** The PS2
gutted per-class networking (19 of its 21 `NetDirect` bodies are 8-byte stubs,
`structs/bottle.md` §9a), and this accessor's single consumer is
`net_session_partition_peers_by_relevance`. Here the PS2 method count is a **lower** bound.

### 7b. Behaviour differences

**The PC ships three more camera modes.** `camera_begin_mode` scans `g_camera_mode_table` to
`0x006508d4`, i.e. **17** records of `0x44`; the PS2's `Begin` loops `while (iVar6 < 0xe)`, i.e.
**14**. ✅ Confirmed in game by the user.

**Look-around is unrestricted on the PC.** PS2 `LookAround` opens with
`strcasecmp(<a mission name>, act_mission+0x5c)` and forces the selector to 0 on a match — a
per-mission kill switch. The PC has no such test. ✅ Confirmed in game by the user. The two builds
also disagree on units and on the case numbering: PC half-turns with a ±2.0 short-way wrap and
cases `{0, 1, 2, -1}`, PS2 radians with no wrap and cases `{0, 1, 2, 3}`, mapping PC
`{0, 1, 2, -1}` onto PS2 `{0, 2, 3, 1}`.

**Mode defaults.** Where the PS2 hard-codes mode `1`, the PC restores
`g_camera_last_onfoot_mode` — in `camera_reset`, `camera_set_car` and `camera_link_to_frame`
alike.

**Free-look input.** PS2 `FreeLookBase` reads its two deltas from two globals; the PC reads them
from `input_get_action_value(0x13)` and `(0x12)` — action slots labelled textdb 312 "Aiming" —
and scales them by `g_aiming_sensitivity_x` / `_y`, with `g_mouse_control_inverted` flipping the
pitch. None of that exists on the PS2.

**The graphics interface.** PC `camera_tick` reads `g_engine`'s vtable slots `0xd0` and `0xd4` to
fill the frame-time ring; the PS2 reads `igraph`'s `0xf0` and `0xf4` at the same statements.

### 7c. Layout differences

**Field-offset deltas.** `+0x10` mode is common. Above `+0x58` the PS2 runs `0x18` higher
(`+0x58/+0x5c/+0x60/+0x64` ↔ `+0x70/+0x74/+0x78/+0x7c`), and above `+0x6c` it runs `0x14` higher
(`+0x6c..+0x74` ↔ `+0x80..+0x88`, `+0x7c..+0x84` ↔ `+0x8c..+0x94`, `+0x88` ↔ `+0x98`,
`+0x90` ↔ `+0xa0`, `+0x94` ↔ `+0xa4`). The delta is **not constant** — do not rebase one offset
from another.

**`+0x0c` is byte-proven across builds** by the `car_setspeed` script arm: the PC reads
`camera+0xc`, the PS2 reads `*(int *)(game + 0x7c)`, and the PS2 camera is `C_game+0x70`, so
`game+0x7c` IS `camera+0xc`.

**`G_CameraDialog`** is `0x14` bytes on the PC and `0x30` on the PS2.

**Globals.** `g_camera_swing_phase` is PS2 `naklon` (twin CONFIRMED);
`g_camera_tick_static_dtor_guard` is a packed flag word (bits 0 and 1) where the PS2 keeps separate
`init$NNNN` statics. The `camera_tick` statics are PS2 function-local statics of `Tick`, each
matched by case label and statement:

| PC name | PS2 static | Matched by |
|---|---|---|
| `g_camera_car_old_dir_y_mode_0xb` | `oldDirY$2455` | case `0xb`, smoothed with `faktorStability` |
| `g_camera_car_old_dir_y_mode_0xa` | `oldDirY$2443` | case `10`, plain copy |
| `g_camera_car_old_dir_y_mode_7_9` | `oldDirY$2417` | cases `7` / `8` / `9` |
| `g_camera_orbit_yaw` | `rot$2351` | mode `0x12`, `+= dt × 0.0002`, 2π wrap |
| `g_camera_sniper_sway_phase_pitch` | `kl$2564` | mode 5, `+= dt × 2.5` |
| `g_camera_sniper_sway_phase_yaw` | `kl2$2567` | mode 5, `+= dt × 3.0` |
| `g_camera_dir_yaw` | `cameraDirRot$2605` | 2π wrap |
| `g_camera_dir_pitch` | `cameraDirUpDown$2608` | the same 0.01..0.99 clamp |
| `g_camera_car_sway_filtered` | `oldS$2498` | case `0xd`: the old value × 0.9 plus the scaled new one, `y` forced to 1.0 |
| `g_camera_car_prev_velocity` | `oldSpeed$2495` | case `0xd`: the car's velocity copied at the end of the block; no reader on either build |

### 7d. `ParheliaSetFov` and `S_auto_hide_machine`

**`ParheliaSetFov__17G_IndicatorsClassFP10I3D_cameraf` — PS2-only.** It has three PS2 callsites,
and at every one the PC calls the `I3D_camera`'s own FOV vtable slot `0x50` directly with the same
argument: `CumNaNe` → `camera_lock_at_human_pair` (π/4 = `0x3f490fdb`), `SetSniperFov` →
`camera_set_sniper_fov`, and `GM_Carcyclopedia::OnCreate` → `carcyclopedia_ctor`
(π/3 = `0x3f860a92`). The PS2 wrapper adjusts the field of view and the aspect ratio for Matrox
Parhelia multi-head output (`structs/hud.md` §7c); the PC has no such branch at any caller.

**`Tick`'s `S_auto_hide_machine` — PS2-only.** PS2 `Tick` calls
`Tick__19S_auto_hide_machineFUi(*(act_mission+0x20) + 0x1578, 100000)`, a `C_game` member whose
area data `C_mission::OpenBin` loads from `scene2.bin` chunk `0xae52`. The PC `mission_open_bin`
has no case for chunk `0xae52` (its arms above `0xae50` are `0xaef0` and `0xaef1`), so the
subsystem has no data path on the PC, and `camera_tick` calls nothing in its place.

### 7e. What MSVC inlined that the PS2 calls out of line

Marked in the decomp with `PS2-INLINED HERE` / `PS2 CALLS A REAL FUNCTION HERE` banners.

| PC function | The real PS2 call it absorbed |
|---|---|
| `camera_ctor` | two `compressed_pair_imp<>` constructors **and** `reserve__std::__vector_imp<>(3)` + `reserve__std::__vector_pod<>(15)` — the PC emits no reserve at all and starts both vectors empty |
| `camera_dtor` | `__dt__std::__vector_pod<>` and `__dt__std::__vector_deleter<G_CameraPushPop>` |
| `camera_push_mode` | `insert__std::__vector_imp<G_CameraPushPop,…>` (1180 B) — the whole grow path, which is the 549 B vs 104 B difference |
| `camera_pop_mode` | `first__compressed_pair_imp<>` |
| `camera_dialog_start` | `reserve__gif::vector<G_CameraDialog,8>` (348 B) |
| `camera_add_race_camera` | `data__std::__vector_pod<>` **and** `insert__std::__vector_pod<>` — the 462 B vs 136 B difference |
| `camera_lock_at_human_pair`, `camera_set_sniper_fov` | `ParheliaSetFov__17G_IndicatorsClass` → the `I3D_camera` FOV slot |

`copy_range_stride_0x14`, `fill_n_stride_0x14`, `vec_count_stride_0x14`, `stl_copy_dwords`,
`stl_fill_n_dwords`, `stl_construct_dword` and `stub_noop_args8_00575c00` are **not** inlinings
and **not** camera members — they are the shared STL primitives MSVC still calls out to, with
callers spread across the image. Naming any of them `camera_*` would be a false ownership claim.

## 8. Related

- `structs/game.md` §2 — the `+0x4c` field and `game_get_camera`.
- `structs/racecam.md` — `C_racecam`, whose `+0x70` record is the `G_RaceCamera`.
- `structs/hud.md` — `g_hud_flags` and the `ParheliaSetFov` row.
- `structs/input.md` — the action slots `0x12` / `0x13` the free-look path reads.
- `basic/artifacts/racing.md` — the racing mode; mission 6 is the retail path into `C_racing` (§4b).
- `structs/carcyclopedia.md` — the screen that uses menu mode `0x18`.

## 9. Open items

1. **In game:** the sniper scope is mode 5, and entering or leaving it flips two `g_hud_flags`
   bits (`0x20000` on entry, `0x20` on exit) — confirm the HUD really disappears under the scope
   and comes back.
2. **In game:** `camera_set_swing` arms a tilt driven by `sin(g_camera_swing_phase · π)`. What
   triggers the swing, and on which vehicles, is undecoded.
3. **Fields with one side missing:** `tick_suspended` (`+0x20`) is written only by `camera_reset`,
   so on the PC it stays 0 and its readers' other branch never runs; `sniper_fov` (`+0x90`) has no
   decoded reader; `unused_21` (`+0x21`) has none at all. The meaning of the ten `mode_params`
   entries is not decoded; entry 5 is never read (§1).
