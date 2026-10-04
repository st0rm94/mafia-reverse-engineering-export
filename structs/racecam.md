# `C_racecam` — the race-track camera actor (kind `0x1c`)

**Created** 2026-08-16 07:38 +0200 · **Modified** 2026-09-30 23:08 +0200

PC prefix **`racecam_`** (distinct from the `race_` prefix of `race_compare_position` and its
siblings: the fifth character differs).

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x1c`** |
| PC vtable | `.rdata 0x0063cb00` (39 slots) |
| PC constructor | `racecam_ctor` @ `0x004d33d0` |
| PC translation unit | `[0x004d33d0, 0x004d35c0)` — 9 function starts, all named |
| Base class | **`C_actor`** (`structs/actor.md`) |
| PS2 class | `C_racecam`, 11 symbols, all accounted (§3) |
| Related | `basic/artifacts/racing.md` (the racing mode) · `structs/racing.md` (sibling classes) |

## 1. What it is

A **camera placed on the race track**, and nothing else. The class carries no behaviour: it holds
one `0x18`-byte `G_RaceCamera` record, fills it with designer defaults when the scene binds it to a
frame, and hands that record to the `G_Camera` singleton for the duration of the scene. All the
actual camera work — choosing which race camera is live, following the cars — happens inside
`G_Camera` and inside `C_circuit`, which pairs each camera node with the two track gates it sits
between (`circuit_determine_race_cams`).

It has no `AI`, no `Update`, no `Collision`, no `Hit`: eight methods, six of them lifecycle, 9 PC
functions in total.

The class position is fixed by five independent measurements:

| lever | result |
|---|---|
| PC `.rdata` vtable order | `0x0063ca58` (`C_pumper`) → **`0x0063cb00`** → `0x0063cbd0` (`C_railway`, kind 8) |
| link order by source file | `C_pumper.cpp` → **`C_racecam.cpp`** → the rail family |
| CRT static-init/atexit bounds | opens past `0x004d33a0`/`0x004d33c0`, closes at `0x004d35c0`/`0x004d35e0` |
| kind byte | the PC ctor writes `0x1c` at `+0x10`, matching `structs/entity.md` §1 (PS2 bridge in §4) |
| body agreement | all eight methods agree statement for statement across builds |

**In the retail PC build the race cameras run in mission 6.** `racecam_game_init` gates the
registration on `g_racing != 0`. Mission 6 (MISE06-AUTODROM) builds `g_racing` through its
`racing_mission6_init` script command, and its `scene2.bin` carries 26 `C_racecam` actors. Because
`racecam_game_init` is the only caller of `camera_add_race_camera`, the cameras seen in game are
these actors registering (`basic/artifacts/racing.md` §8a, `structs/camera.md` §3b). **Confirmed in game:**
after the race finishes, the player watches the other racers from these trackside cameras. In every
other mission `g_racing` stays NULL; an actor there still constructs, loads and runs its lifecycle,
and never reaches the camera manager.

## 2. Instance layout

Everything the class owns is the single record at `+0x70`.

```c
typedef struct C_racecam {           /* extends C_actor */
    /* +0x00 */ void      *vtable;   // [H] 0x0063cb00, 39 slots
    /* +0x10 */ uint32_t   kind;     // [H] 0x1c
    /* ...  */                       //     C_actor base
    /* +0x65 */ uint8_t    save_excluded; // [H] C_actor field; racecam_game_init sets 1
    /* +0x68 */ I3D_frame *own_frame;// [H] C_actor's frame

    /* ── G_RaceCamera, 0x18 bytes — the whole of C_racecam's own state ── */
    /* +0x70 */ I3D_frame *frame;    // [H] the camera frame
    /* +0x74 */ float      param_a;  // [M] designer default 70.0f
    /* +0x78 */ float      param_b;  // [M] designer default 70.0f
    /* +0x7c */ float      weight;   // [M] designer default 1.0f
    /* +0x80 */ int32_t    gate_a;   // [H] index of the gate before the camera; -1 at the lap wrap
    /* +0x84 */ int32_t    gate_b;   // [H] index of the gate after the camera; 0 at the lap wrap
} C_racecam;
```

Confidence: **[H]** measured · **[M]** measured, meaning inferred from defaults · **[U]** role
unknown.

The record's type name `G_RaceCamera` types a PC structure: the PC decompiler renders the record as
six anonymous dwords, while the PS2 mangled symbol `AddRaceCamera__8G_CameraFP12G_RaceCamera`
states the parameter type outright.

### 2a. Field evidence

| offset | field | conf | writer(s) | reader(s) | notes |
|---|---|---|---|---|---|
| `+0x70` | camera frame | H | `racecam_init`, `racecam_load_data`, `racecam_game_init`, `racecam_duplicate_from` | `camera_add_race_camera`, `camera_remove_race_camera` | seeded from the actor's frame `+0x68`; re-bound after load ¹ |
| `+0x74` | designer float A | M | `racecam_init` (70.0f), `racecam_load_data`, `racecam_duplicate_from` | - | no reader inside the class; consumed by `G_Camera` |
| `+0x78` | designer float B | M | same | - | same value, same writers (§5) |
| `+0x7c` | weight | M | `racecam_init` (1.0f), `racecam_load_data`, `racecam_duplicate_from` | - | named for its 1.0f default and position (§5) |
| `+0x80` | gate A | H | `racecam_init` (0), `racecam_game_init` (0), `racecam_load_data`, `racecam_duplicate_from`, `camera_init_race_cameras` | - | ² |
| `+0x84` | gate B | H | same | - | ² |

¹ `racecam_load_data` re-binds the frame after reading, because a serialized pointer is stale by
construction.

² `camera_init_race_cameras`, called from `racing_game_init`, walks the registered records, finds
the `C_circuit` race-camera node (`+0xcc` vector) nearest to each camera frame, and stores the index
(`gate+0x1c`) of that node's two gates here — the lower-indexed gate first, as
`circuit_determine_race_cams` ordered them. When the first gate is gate 0 and the second is the
last gate, the pair is written as `-1` / `0` instead, so the camera straddles the start/finish line.

The three floats have no located reader on either build; they are read inside `G_Camera` (§5).

## 3. Method coverage

| PC VA | PC name | PS2 twin | PS2 VA | PS2 size | status | role |
|---|---|---|---|---|---|---|
| `0x004d33d0` | `racecam_ctor` | `__ct__9C_racecamFv` | `0x001db690` | 60 | CONFIRMED | vtable + kind `0x1c` |
| `0x004d33f0` | `racecam_scalar_deleting_dtor` | - | - | - | STRUCTURAL ¹ | vtable slot 16 |
| `0x004d3410` | `racecam_dtor` | `__dt__9C_racecamFv` | `0x001db6d0` | 96 | CONFIRMED | vtable re-install + base dtor |
| `0x004d3420` | `racecam_init` | `Init__9C_racecamFP9I3D_frame` | `0x001db730` | 112 | CONFIRMED | binds the frame, seeds the record |
| `0x004d3480` | `racecam_save_data` | `SaveData__9C_racecamFP7C_chunk` | `0x001db7a0` | 64 | CONFIRMED | `dtaWrite` `0x18` B |
| `0x004d34b0` | `racecam_load_data` | `LoadData__9C_racecamFP7C_chunk` | `0x001db7e0` | 176 | CONFIRMED | `dtaRead` `0x18` B + frame re-bind |
| `0x004d3530` | `racecam_duplicate_from` | `DuplicateFrom__9C_racecamFP7C_actor` | `0x001db890` | 96 | CONFIRMED | six-dword copy |
| `0x004d3550` | `racecam_game_init` | `GameInit__9C_racecamFv` | `0x001db8f0` | 104 | CONFIRMED | register with `G_Camera` iff `g_racing != 0`; act-state 2 |
| `0x004d35a0` | `racecam_game_done` | `GameDone__9C_racecamFv` | `0x001db960` | 56 | CONFIRMED | unregister — **not** gated on `g_racing` |
| - | - | `SaveGameGetSize__9C_racecamFv` | `0x001db9a0` | 8 | STRUCTURAL ² | `return 0` |
| - | - | `SaveGameSave__9C_racecamFPPv` | `0x001db9b0` | 8 | STRUCTURAL ² | `return 1` |
| - | - | `SaveGameLoad__9C_racecamFPPv` | `0x001db9c0` | 8 | STRUCTURAL ² | `return 1` |

¹ MSVC-only; GCC folds it into `__dt`'s flag arm. ² Folded onto a PC stub shared by many vtables,
so the mapping lives only in this table.

8 PS2 methods with real bodies + 1 MSVC scalar deleting destructor = **9 PC starts**.

### 3a. The `G_Camera` bridge

| PC VA | PC name | PS2 twin | what it does |
|---|---|---|---|
| `0x005dbca0` | `camera_add_race_camera` | `AddRaceCamera__8G_CameraFP12G_RaceCamera` | duplicate-checked push_back onto the manager's vector at `+0x7c` |
| `0x005dbe70` | `camera_remove_race_camera` | `RemoveRaceCamera__8G_CameraFP12G_RaceCamera` | clears `g_active_race_camera` if it is this record, then shift-erases |

The receiver is byte-proven: `racecam_game_init` disassembles as
`mov edx,[0x0065115c]; mov ecx,[edx+0x24]; add ecx,0x4c` — `C_game+0x4c`, which `structs/game.md`
documents as the `G_Camera` sub-object.

## 4. PC vs PS2

* **Kind bridge:** `__ct__9C_racecamFv` writes `0x1c` at PS2 `+0x50`, the same value the PC ctor
  writes at `+0x10`.
* **Field delta:** a uniform `+0x10` inside the class band — `save_excluded` PC `+0x65` ↔ PS2 `+0x69`,
  `own_frame` `+0x68` ↔ `+0x70`, the `G_RaceCamera` record `+0x70..+0x84` ↔ `+0x80..+0x94`.
* **Receiver path:** PS2 reaches the `G_Camera` object as `*(act_mission+0x20)+0x70`.

## 5. Open items

| item | state |
|---|---|
| the two 70.0f floats and the 1.0f weight | designer defaults with no reader inside this class on either build; consumed by `G_Camera`, whose reader of the record is not traced |
| field of view against activation distance | which of the two 70.0f floats is which; the mission 6 cameras are the observable case (§1) |
| the scene side | `circuit_load_gates_from_scene` finds race-camera nodes by testing bit `0x8` of a web-path node's flag word, starting from node id `0x27c`; which scene authoring tool sets that bit, and what the node's sub-type `2` means, are not traced |
