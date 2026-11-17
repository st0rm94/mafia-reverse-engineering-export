# `C_racecam` — the race-track camera actor

**Created** 2026-08-16 07:38 +0200 · **Modified** 2026-11-01 17:14 +0100

A **camera placed on the race track**, and nothing else. The class holds one `0x18`-byte
`G_RaceCamera` record, fills it with defaults when the scene binds it to a frame, and hands that
record to the `G_Camera` singleton for the duration of the scene. All the camera work (choosing
which race camera is live, following the cars) happens inside `G_Camera` and inside `C_circuit`,
which pairs each camera node with the two track gates it sits between
(`circuit_determine_race_cams`). The prefix `racecam_` is distinct from the `race_` prefix of
`race_compare_position` and its siblings.

| | |
|---|---|
| PC prefix | `racecam_` |
| PC kind (`+0x10`) | `0x1c`, stamped by `racecam_ctor` |
| PC vtable | `g_racecam_vt` @ `0x0063cb00`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x88` (`operator_new(0x88)` in `mission_create_actor` `case 0x1c`; the record ends at `0x88`) |
| PC constructor | `racecam_ctor` @ `0x004d33d0` |
| PC translation unit | `[0x004d33a0, 0x004d35c0)`: the unit's CRT pair, then the nine `racecam_*` functions; the next unit's guard `crt_static_init_guard_004d35c0` opens the rail family |
| PS2 class | `C_racecam` (tag `9C_racecam`), 11 methods: 8 CONFIRMED twins, 3 base stubs, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC.

```c
typedef struct C_racecam {                   // 0x88 bytes; PC offsets
    /* +0x00 */ C_actor      base;           // [H] the C_actor band (structs/actor.md)
    /* +0x70 */ G_RaceCamera race_camera;    // [H] the record handed to G_Camera (§1a)
} C_racecam;
```

### 1a. `G_RaceCamera` — the record at `+0x70`

Offsets are relative to the record. The type name comes from the PS2 symbol
`AddRaceCamera__8G_CameraFP12G_RaceCamera`; the PC decompiler renders the record as six anonymous
dwords.

```c
typedef struct G_RaceCamera {                // 0x18 bytes; C_racecam + 0x70
    /* +0x00 */ I3D_frame *frame;            // [H] the camera frame
    /* +0x04 */ float      fov;              // [M] seeded 70.0f; never read (§4)
    /* +0x08 */ float      fov_alt;          // [M] seeded 70.0f; never read (§4)
    /* +0x0c */ float      zoom;             // [M] seeded 1.0f; never read (§4)
    /* +0x10 */ int32_t    gate_before;      // [H] index of the gate before the camera; -1 at the lap wrap
    /* +0x14 */ int32_t    gate_after;       // [H] index of the gate after the camera; 0 at the lap wrap
} G_RaceCamera;
```

The three float names read the seed values as a field of view in degrees and a zoom factor. That is
an interpretation: no instruction on either build reads the three fields (§4), so the binary cannot
confirm it. The two gate indices are the record's only fields any consumer uses besides the frame.

## 2. Field evidence

Measured by a census of every access in the nine `racecam_*` functions and of the `G_Camera`
functions that take the record (`camera_add_race_camera`, `camera_remove_race_camera`,
`camera_init_race_cameras`, the race-camera mode of `camera_tick`).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `racecam_ctor` (vtable `g_racecam_vt`, kind `+0x10` = `0x1c`), `racecam_game_init` (`+0x65` = 1, the `C_actor` save-exclusion byte) | the `C_actor` methods of `structs/actor.md`; `racecam_load_data` and `racecam_game_init` (`+0x68`, the actor frame) | the inherited band |
| +0x70 | race_camera | H | `racecam_init`, `racecam_load_data`, `racecam_duplicate_from`, `racecam_game_init` | `camera_add_race_camera`, `camera_remove_race_camera` (by address); `racecam_save_data` | §2a |

### 2a. `G_RaceCamera`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | frame | H | `racecam_init` (its `I3D_frame` argument), `racecam_load_data` and `racecam_game_init` (= the actor frame `+0x68`), `racecam_duplicate_from` | `camera_tick` (race-camera mode `0x14`), `camera_init_race_cameras` | `racecam_load_data` re-binds it after the read because a serialized pointer is stale |
| +0x04 | fov | M | `racecam_init` (70.0f), `racecam_load_data`, `racecam_duplicate_from` | - | ¹ |
| +0x08 | fov_alt | M | `racecam_init` (70.0f), `racecam_load_data`, `racecam_duplicate_from` | - | ¹ |
| +0x0c | zoom | M | `racecam_init` (1.0f), `racecam_load_data`, `racecam_duplicate_from` | - | ¹ |
| +0x10 | gate_before | H | `racecam_init` (0), `racecam_game_init` (0), `racecam_load_data`, `racecam_duplicate_from`, `camera_init_race_cameras` | `camera_tick` (race-camera mode `0x14`) | ² |
| +0x14 | gate_after | H | `racecam_init` (0), `racecam_game_init` (0), `racecam_load_data`, `racecam_duplicate_from`, `camera_init_race_cameras` | `camera_tick` (race-camera mode `0x14`) | ² |

¹ A record reaches code outside the class only through the `G_Camera` vector at `+0x7c` and
`g_active_race_camera`, and every reader there uses the frame and the two gate indices only: the
race-camera mode `0x14` of `camera_tick`, `camera_init_race_cameras`, `camera_remove_race_camera`
and the multiplayer caller of `camera_get_race_camera_at`. The three floats are constructed, loaded,
duplicated and saved, and nothing else. All 26 race cameras of mission 6 store 70.0, 70.0 and 1.0
in the scene file.

² `camera_init_race_cameras`, called from `racing_game_init`, walks the registered records, finds
the `C_circuit` race-camera node (`+0xcc` vector) nearest to each camera frame, and stores the index
(`gate + 0x1c`) of that node's two gates here, the lower-indexed gate first, as
`circuit_determine_race_cams` ordered them. When the first gate is gate 0 and the second is the last
gate, the pair is written as `-1` / `0`, so the camera straddles the start/finish line.

## 3. Method family

Coverage: all **11** PS2 `9C_racecam` methods are in the tables below: **8 CONFIRMED** twins
(`{ctor}`, `{dtor}`, `Init`, `SaveData`, `LoadData`, `DuplicateFrom`, `GameInit`, `GameDone`) and
**3 base stubs** (`SaveGameGetSize`, `SaveGameSave`, `SaveGameLoad`) whose vtable slots hold shared
PC functions. The class has no `AI`, `Update`, `Collision` or `Hit`; the slots it does not override
are the `C_actor` defaults (`structs/actor.md` §3c).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x004d33a0 | `crt_static_init_guard_004d33a0` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x004d33c0 | `crt_atexit_register_004d33c0` | the translation unit | - | - | - | its `atexit` registrar |
| 0x004d33d0 | `racecam_ctor` | instance | `__ct__9C_racecamFv` | 0x001db690 | CONFIRMED | `actor_ctor`, vtable `0x0063cb00`, kind `0x1c` |
| 0x004d33f0 | `racecam_scalar_deleting_dtor` | instance | - | - | - | vtable slot 16; the MSVC arm that adds `operator delete` under the flag bit, which the PS2 fuses into `__dt__` |
| 0x004d3410 | `racecam_dtor` | instance | `__dt__9C_racecamFv` | 0x001db6d0 | CONFIRMED | restores the vtable and chains `actor_dtor` |

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00477700 | `stub_ret0_cdecl_00477700` | `SaveGameGetSize__9C_racecamFv` | 0x001db9a0 | base-stub | slots 22, 35; returns 0, so the savegame stores nothing for the camera |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveGameSave__9C_racecamFPPv` | 0x001db9b0 | base-stub | slot 23; returns true |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveGameLoad__9C_racecamFPPv` | 0x001db9c0 | base-stub | the same function as the row above, slot 24 |
| 0x004d3420 | `racecam_init` | `Init__9C_racecamFP9I3D_frame` | 0x001db730 | CONFIRMED | slot 18; `actor_init`, then binds the frame and seeds the record (70, 70, 1.0, gates 0) |
| 0x004d3480 | `racecam_save_data` | `SaveData__9C_racecamFP7C_chunk` | 0x001db7a0 | CONFIRMED | slot 19; `dtaWrite` of the `0x18`-byte record |
| 0x004d34b0 | `racecam_load_data` | `LoadData__9C_racecamFP7C_chunk` | 0x001db7e0 | CONFIRMED | slot 20; `dtaRead` of `0x18` bytes when the chunk holds them, then the frame re-bind |
| 0x004d3530 | `racecam_duplicate_from` | `DuplicateFrom__9C_racecamFP7C_actor` | 0x001db890 | CONFIRMED | slot 21; the base `DuplicateFrom`, then a six-dword copy of the record |
| 0x004d3550 | `racecam_game_init` | `GameInit__9C_racecamFv` | 0x001db8f0 | CONFIRMED | slot 25; `actor_game_init`, `+0x65` = 1, re-binds the frame, zeroes the gates, `camera_add_race_camera` iff `g_racing != 0`, act state 2 |
| 0x004d35a0 | `racecam_game_done` | `GameDone__9C_racecamFv` | 0x001db960 | CONFIRMED | slot 26; `actor_game_done`, then `camera_remove_race_camera`, not gated on `g_racing` |

### 3d. Free helpers

The translation unit holds no free helper.

## 4. The camera in the game

**In the retail PC build the race cameras run in mission 6.** `racecam_game_init` gates the
registration on `g_racing != 0`. Mission 6 (MISE06-AUTODROM) builds `g_racing` through its
`racing_mission6_init` script command, and its `scene2.bin` carries 26 `C_racecam` actors. Because
`racecam_game_init` is the only caller of `camera_add_race_camera`, the cameras seen in game are
these actors registering (`basic/artifacts/racing.md` §8a, `structs/camera.md` §4b). **Confirmed in
game:** after the race finishes, the player watches the other racers from these trackside cameras.
In every other mission `g_racing` stays NULL; an actor there still constructs, loads and runs its
lifecycle, and never reaches the camera manager.

### 4a. The `G_Camera` bridge

| PC VA | PC name | PS2 twin | What it does |
|---|---|---|---|
| `0x005dbca0` | `camera_add_race_camera` | `AddRaceCamera__8G_CameraFP12G_RaceCamera` | duplicate-checked `push_back` onto the manager's vector at `+0x7c`; its only caller is `racecam_game_init` |
| `0x005dbe70` | `camera_remove_race_camera` | `RemoveRaceCamera__8G_CameraFP12G_RaceCamera` | clears `g_active_race_camera` if it is this record, then shift-erases; its only caller is `racecam_game_done` |

The receiver is byte-proven: `racecam_game_init` disassembles as
`mov edx,[0x0065115c]; mov ecx,[edx+0x24]; add ecx,0x4c`, which is `C_game + 0x4c`, the `G_Camera`
member (`structs/game.md`).

### 4b. The camera nodes in the navigation web

`circuit_load_gates_from_scene` pairs each camera with the track through the mission's navigation
web (`check.bin`): a race-camera node is a web node whose two-bit category (bits 2 and 3 of the
category word `C_web_node + 0x0c`, `structs/web_node.md` §2a) has the value 2, that is bit `0x8`. In
mission 6, 26 of the 860 nodes carry it, one per `C_racecam` actor, each with two connections; the
other 834 nodes have category value 1. The node file is produced by the developers' web editor,
which the PC build does not contain (§5), so nothing in the shipped files names the tool that set the
bit.

## 5. PC vs PS2

The class position is settled by five independent measurements:

| Lever | Result |
|---|---|
| PC `.rdata` vtable order | `0x0063ca58` (`C_pumper`) → **`0x0063cb00`** → `0x0063cbd0` (`C_railway`, kind 8) |
| link order by source file | `C_pumper.cpp` → **`C_racecam.cpp`** → the rail family |
| CRT static-init / atexit bounds | the unit opens at its own guard `0x004d33a0` and closes at the next guard `0x004d35c0` |
| kind byte | `__ct__9C_racecamFv` writes `0x1c` at PS2 `+0x50`, the value the PC ctor writes at `+0x10` (`structs/entity_kind.md` §1) |
| body agreement | all eight methods with a body agree statement for statement across builds |

* **Field delta:** a uniform `+0x10` inside the class band: `save_excluded` PC `+0x65` ↔ PS2
  `+0x69`, the actor frame `+0x68` ↔ `+0x70`, the `G_RaceCamera` record `+0x70..+0x84` ↔
  `+0x80..+0x94`.
* **The base stubs:** PS2 vtable slots 11–13 (`SaveGameGetSize`, `SaveGameSave`, `SaveGameLoad`)
  are 8-byte bodies returning 0, 1, 1; the PC slots 22–24 hold the shared stubs of §3c.
* **The scalar deleting destructor** is MSVC-only; GCC fuses it into `__dt__`'s flag arm.
* **Receiver path:** the PS2 reaches the `G_Camera` object as `*(act_mission + 0x20) + 0x70`.
* **The floats are unread on the PS2 too:** its `activeRaceCam` readers in `Tick__8G_CameraFUi` use
  the frame and the two gate indices only.
* **The node file writer** is `SaveNodes__10C_web_path`, a PS2 symbol with no PC counterpart
  (`structs/web_path.md` §7).

## 6. Related

- `structs/actor.md` — `C_actor`, the base class, and the shared slot stubs.
- `structs/camera.md` §4b — `G_Camera`'s race-camera mode.
- `structs/racing.md` — `C_circuit` and the sibling racing classes.
- `basic/artifacts/racing.md` — the racing mode.
- `structs/entity_kind.md` — kind `0x1c`.

## 7. Open items

1. **The three floats** (`fov`, `fov_alt`, `zoom`) are stored and never read on either build; what
   the developers' editor meant by them is not recoverable from the binary.
