# `C_detector` — the script trigger volume (kind `5`)

**Created** 2026-08-13 00:24 +0200 · **Modified** 2026-09-17 17:53 +0200

PC prefix **`detector_`**.

| | |
|---|---|
| PC kind (`entity+0x10`) | **`5`** |
| PC vtables | `.rdata 0x0063c008` (39 slots) **and** `0x0063c0a4` (10 slots, the `C_program` sub-object) |
| PC constructor | `detector_ctor` @ `0x004793b0` |
| PC instance size | `0x128` (entity factory `case 5`) |
| PC translation unit | `[0x004793b0, 0x0047aae0)` — 25 function starts, all named, plus 5 functions at the head of the next unit (§2) |
| Base classes | **`C_actor`** at `+0x00` **and `C_program` at `+0x70`** |
| PS2 class | `C_detector`, 28 methods + 8 `@128@` thunks; 25 mapped, 2 PS2-port-only, `NetDirect` unmapped (§5, §7) |

## 1. What it is

An invisible volume that runs a script when the player enters it, uses it, or hits it. It is the
class behind the game's own `detector_*` script vocabulary, which is why the PC prefix is
`detector_`. Counted over the shipped mission scripts:

| script command | uses | implemented by |
|---|---|---|
| `detector_inrange` | 1123 | `detector_program_process`, instruction `0x1c` |
| `detector_issignal` | 567 | `detector_get_signal` — reads the BYTE at `+0xdc` |
| `detector_setsignal` | 517 | `detector_set_signal` — writes the BYTE at `+0xdc` and clears `+0x65` |
| `detector_waitforuse` | 505 | `detector_program_process` `0x1f` → `detector_create_using_object`; released by `detector_use` |
| `detector_waitforhit` | 96 | `detector_program_process` `0x20` → `detector_create_colls`; released by `detector_hit` |

Whole shipped scripts are named after it — mission `MISE05-MESTO`, script `detector_bertone`;
mission `MISE07B-CHULIGANI`, script `detector_konec`.

## 2. It has TWO base classes

`detector_ctor` installs **two** vtables: `0x0063c008` at `+0x00` (the `C_actor` chain) and
`0x0063c0a4` at `+0x70` — the `C_program` sub-object. Three independent proofs:

1. the two vptr writes in the constructor, and the matching pair of restores in the destructor;
2. every `C_program`-facing method takes `this = detector + 0x70` (`detector_duplicate_from`
   passes `src + 0x70`, null-guarded);
3. `detector_program_process` reaches the owning detector's frame as **`this - 8`** —
   `+0x70 - 8 = +0x68`, the `C_actor` frame pointer.

The `C_program` sub-vtable's slots are the script-VM interface:

| sub-slots | filled by |
|---|---|
| 5, 6, 8 | 5-byte forwarders emitted in this TU (`0x0047a810`, `0x0047a820`, `0x0047a830`) |
| 7 | the class's own `detector_program_process` |
| 0, 2, 3, 4 | MSVC adjustor thunks at the head of the NEXT translation unit — `0x0047ab10`/`ab20`/`ab30`/`ab40`, each byte-proven `sub ecx, 0x70 ; jmp <detector method>` |

A fifth `C_detector` function sits there too, `detector_get_program` (§7a). COMDAT placement means
a translation-unit sweep files all five under `C_dog`; the same placement puts
`crash_obj_get_material_id` at the head of THIS unit.

## 3. `DYNB` — the trigger volume is named in the model

`detector_create_colls` walks the bound frame's subtree twice through the engine's frame
enumerator: first with `detector_count_dynb_frames` to size the arrays, then with
`detector_create_dynb_coll` to build them. Both callbacks accept a child frame only if

* its name is longer than four characters, **and**
* its first four bytes are the literal **`DYNB`** (`0x0064b8dc`), compared as one dword.

So a level designer marks a trigger's shape by naming dummy frames `DYNB…` in the model. When a
model has no `DYNB` child at all, the class falls back to a single volume around the frame's own
bound volume. Each accepted frame becomes one `0x60`-byte `tDynamicCollObject`, and
`detector_update` re-derives its centre, radius, world matrix and inverse every frame, so a trigger
glued to a moving frame keeps up.

Both callbacks have **zero textual callsites** — they are passed to the enumerator as function
pointers.

## 4. Field layout

```c
typedef struct C_detector {              /* 0x128 bytes */
    /* +0x000 */ C_actor  base;          // [H] kind 5 at +0x10, frame at +0x68
    /* +0x070 */ C_program program;      // [H] the script VM sub-object
    /* +0x0b4 */ C_detector *self;       // [H] back-link written by GameInit
    /* +0x0d8 */ uint  inrange_mask;     // [H] actor-type mask for detector_inrange
    /* +0x0dc */ uint8_t signal;         // [H] the script signal byte
    /* +0x0dd */ bool  waiting_for_use;  // [H] set by CreateUO, cleared by detector_use
    /* +0x0de */ bool  waiting_for_hit;  // [H] set by Process 0x20, cleared by Hit
    /* +0x0df */ bool  volumes_live;     // [H] gates detector_update and the Hit event
    /* +0x0e0 */ S_using_object uo;      // [H] embedded record: pos, +0xec radius 1.5f,
                                         //     +0xfc owner, +0x100 anchor frame,
                                         //     +0x10c action id, +0x114 pos ptr
    /* +0x118 */ tDynamicCollObject **colls;  // [H]
    /* +0x11c */ S_matrix *world;        // [H] 0x40 stride, one per volume
    /* +0x120 */ S_matrix *world_inv;    // [H] 0x40 stride, parallel array
    /* +0x124 */ int   coll_count;       // [H]
} C_detector;
```

### 4a. Field evidence

| offset | field | conf | writer(s) | reader(s) | notes |
|---|---|---|---|---|---|
| `+0xd8` | `inrange_mask` | H | `detector_init` (0), `detector_load_data`, `detector_duplicate_from`, `detector_save_game_load` | `detector_save_data`, `detector_save_game_save`, `detector_program_process` ¹ | designer data: the only field in the scene chunk |
| `+0xdc` | `signal` | H | `detector_set_signal`, `detector_game_init` (0), `detector_save_game_load` | `detector_get_signal`, `detector_save_game_save` | `detector_set_signal` also clears `+0x65`; savegame load clears it when the byte is non-zero |
| `+0xdd` | `waiting_for_use` | H | `detector_create_using_object`, `detector_use`, `detector_game_init` (0), `detector_save_game_load` | `detector_use`, `detector_save_game_save` | set on load ⇒ `detector_create_using_object` |
| `+0xde` | `waiting_for_hit` | H | `detector_program_process` `0x20`, `detector_hit`, `detector_game_init` (0), `detector_save_game_load` | `detector_hit`, `detector_save_game_save`, `detector_program_process` | set on load ⇒ `detector_create_colls(0)` |
| `+0xdf` | `volumes_live` | H | `detector_game_init` (0), `detector_program_process` `0x154`/`0x155` | `detector_update`, `detector_hit`, `detector_program_process` | |
| `+0xe0` | `uo` | H | `detector_create_using_object` | `using_object_add_object` (`C_game`'s using-object list) | `+0xec` = `0x3fc00000` = 1.5f |
| `+0x118`..`+0x124` | the volume arrays | H | `detector_create_colls`, `detector_create_dynb_coll`, `detector_erase_colls` | `detector_update` | `0x40`-byte matrix stride |

¹ The sub-object reads it as `this+0x68` (`+0x70 + 0x68 = +0xd8`).

The savegame record is `0xc` bytes: version byte 1, the `+0xd8` dword, the `+0xdc`/`+0xdd`/`+0xde`
bytes and the using-object id at `+0x10c`.

## 5. Method coverage

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x004793b0` | `detector_ctor` | `__ct__10C_detectorFv` | - | CONFIRMED |
| `0x00479440` | `detector_scalar_deleting_dtor` | - | 16 | STRUCTURAL ¹ |
| `0x00479460` | `detector_dtor` | `__dt__10C_detectorFv` | - | CONFIRMED |
| `0x004794c0` | `detector_init` | `Init__10C_detectorFP9I3D_frame` | 18 | CONFIRMED |
| `0x004794f0` | `detector_duplicate_from` | `DuplicateFrom__10C_detectorFP7C_actor` | 21 | CONFIRMED |
| `0x00479530` | `detector_game_init` | `GameInit__10C_detectorFv` | 25 | CONFIRMED |
| `0x004795b0` | `detector_game_done` | `GameDone__10C_detectorFv` | 26 | CONFIRMED |
| `0x004795f0` | `detector_save_data` | `SaveData__10C_detectorFP7C_chunk` | 19 | CONFIRMED |
| `0x00479650` | `detector_load_data` | `LoadData__10C_detectorFP7C_chunk` | 20 | CONFIRMED |
| `0x00479740` | `detector_save_game_get_size` | `SaveGameGetSize__10C_detectorFv` | 22 | CONFIRMED |
| `0x00479760` | `detector_save_game_save` | `SaveGameSave__10C_detectorFPPv` | 23 | CONFIRMED |
| `0x004797d0` | `detector_save_game_load` | `SaveGameLoad__10C_detectorFPPv` | 24 | CONFIRMED |
| `0x004798a0` | `detector_ai` | `AI__10C_detectorFUi` | 13 | CONFIRMED |
| `0x004798e0` | `detector_update` | `Update__10C_detectorFUi` | 14 | CONFIRMED |
| `0x00479b50` | `detector_hit` | `Hit__10C_detectorF…` | 31 | CONFIRMED |
| `0x00479be0` | `detector_create_colls` | `CreateColls__10C_detectorFb` | - | CONFIRMED |
| `0x0047a150` | `detector_count_dynb_frames` | `cbDummyCount__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi` | - | CONFIRMED |
| `0x0047a1a0` | `detector_create_dynb_coll` | `cbDummyCreate__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi` | - | CONFIRMED |
| `0x0047a610` | `detector_erase_colls` | `EraseColls__10C_detectorFv` | - | CONFIRMED |
| `0x0047a6b0` | `detector_create_using_object` | `CreateUO__10C_detectorFib` | - | CONFIRMED |
| `0x0047a790` | `detector_use` | `UseDetector__10C_detectorFv` | - | CONFIRMED |
| `0x0047a810` | `thunk_program_decode_params` | `DecodeParams__10C_detectorFPcRQ29C_program13S_instruction` | prog 5 | STRUCTURAL ² |
| `0x0047a820` | `thunk_program_decode_instruction` | `DecodeInstruction__10C_detectorFPcRi` | prog 6 | STRUCTURAL ² |
| `0x0047a830` | `thunk_stub_noop_args8_0060b1a0` | `DecompileCommand__10C_detectorFiPc` | prog 8 | STRUCTURAL ² |
| `0x0047a840` | `detector_program_process` | `Process__10C_detectorFUi` | prog 7 | CONFIRMED |
| `0x0047ab10` | `thunk_detector_scalar_deleting_dtor` | `@128@__dt__10C_detectorFv` | prog 0 | STRUCTURAL |
| `0x0047ab20` | `thunk_detector_save_game_get_size` | `@128@SaveGameGetSize__10C_detectorFv` | prog 2 | STRUCTURAL |
| `0x0047ab30` | `thunk_detector_save_game_save` | `@128@SaveGameSave__10C_detectorFPPv` | prog 3 | STRUCTURAL |
| `0x0047ab40` | `thunk_detector_save_game_load` | `@128@SaveGameLoad__10C_detectorFPPv` | prog 4 | STRUCTURAL |
| `0x0047ab50` | `detector_get_program` | `GetProgram__10C_detectorFv` | 35 | CONFIRMED |
| `0x005c8d00` | `detector_get_signal` | - | - | STRUCTURAL ³ |
| `0x005c8d10` | `detector_set_signal` | - | - | STRUCTURAL ³ |

¹ MSVC-only. ² Named after the **target** the forwarder jumps to (`program_decode_params`
`0x005afe70`, `program_decode_instruction` `0x005b1b60`, a shared no-op stub). Each has exactly one
dataref, in this class's sub-vtable, so each is this class's own override. ³ Header inlines, called
from the base dispatcher's `detector_issignal` / `detector_setsignal` arms under a `kind == 5` gate;
no PS2 symbol (§7).

The `C_program` base methods behind the positionally matched calls in these bodies are
`program_ctor`, `program_dtor`, `program_save_source_code`, `program_load_source_code`,
`program_init`, `program_done`, `program_duplicate_from` and `program_clear_after_end`
(`structs/program.md`).

## 6. `detector_program_process` — the class's script instructions

| code | script command | what the case does |
|---|---|---|
| `0x1c` | `detector_inrange` | counts scriptable actors (kind `0x1b` or `2`) within a radius whose human-type mask (`+0x5fc`) intersects `+0xd8`, writes the count into a script register, then advances |
| `0x1f` | `detector_waitforuse` | releases any previous using-object, calls `detector_create_using_object`, blocks |
| `0x20` | `detector_waitforhit` | arms the hit path (`detector_hit` releases it) |
| `0x25` | `program_storage` | parks the actor in act-state 2 while program `+0x40` is 0; does not advance |
| `0x154` | `detector_createdyncoll` | no-op unless both `+0xde` and `+0xdf` are clear; sets `+0xdf`, then erase + create colls in mode 1 |
| `0x155` | `detector_erasedyncoll` | when `+0xde` is clear and `+0xdf` set: clears `+0xdf`, then erase colls |

Anything else falls through to the `C_program` base.

### 6a. Why these six commands have a default-arm slot in the base dispatcher

`C_program::Process` is **virtual**. The base `program_process_command` jump table points all six
opcodes above at its default arm precisely because this derived class owns them — `detector_waitforuse`
and `detector_waitforhit` are used in 30+ shipped scripts and work in game. Of the 15 opcodes whose
base slot is the default arm, these six are implemented here and the other **9** are genuinely
unimplemented.

**How the wait works.** The base default arm at `0x005c76da` is literally
`push 0; mov ecx,ebp; call program_next_command` — a no-op that advances the program counter. The
`0x1f` and `0x20` cases here **do not call `program_next_command`**, so the program counter stays
on the same instruction and the script re-executes it every tick until `detector_use` /
`detector_hit` releases it. Blocking is the absence of the advance, which nothing in the base
dispatcher can express.

`entity_process_command` is the OTHER override of the same virtual — it and this function are the
only two in the image that tail into `program_process_command` — but it intercepts only the
`0x1000+` AI-action instruction space, never a script opcode.

## 7. PC vs PS2

* **The `C_program` base sits at PC `+0x70` and PS2 `+0x80`.** PS2 spells the relationship with
  eight `@128@` adjustor thunks (128 = `0x80`) — `__dt`, `SaveGameGetSize`, `SaveGameSave`,
  `SaveGameLoad`, `DecodeParams`, `DecodeInstruction`, `DecompileCommand`, `Process`; the PC uses
  two vtable installs, four adjustor thunks and plain forwarders. Same relationship, two ABIs.
* **Field delta is not uniform.** Measured from both builds' `GameInit` and `SaveGameSave`:

  | field | PC | PS2 | delta |
  |---|---|---|---|
  | back-link | `+0xb4` | `+0xb8` | `+0x4` |
  | `inrange_mask` | `+0xd8` | `+0x108` | `+0x30` |
  | `signal` / `waiting_for_use` / `waiting_for_hit` / `volumes_live` | `+0xdc`..`+0xdf` | `+0x10c`..`+0x10f` | `+0x30` |
  | `uo` | `+0xe0` | `+0x110` | `+0x30` |
  | using-object id | `+0x10c` | `+0x144` | `+0x38` |
  | volume arrays (GameInit zeroes) | `+0x118`/`+0x11c`/`+0x120`/`+0x124` | `+0x100`/`+0xe0`/`+0xe4`/`+0x104` | - |
  | actor flag cleared by `set_signal` | `+0x65` | `+0x69` | `+0x4` |

  The PS2 savegame record is `0x10` bytes against the PC's `0xc`.
* **`DYNB` compare:** one dword compare on the PC, `memcmp(name, "DYNB", 4)` on PS2.
* **Two PS2 methods are PS2-port-only:** `ChangeSceneInit__10C_detectorFv` and
  `ChangeSceneDone__10C_detectorFv`, the city-streaming pair; no unclaimed PC start remains in the
  TU for either.
* **`NetDirect__10C_detectorFUi`** is an empty body on PS2 and has no PC row.
* **`detector_get_signal` / `detector_set_signal` have no PS2 symbol:** PS2 inlines both into its
  own `Process` arms (`DETECTOR_ISSIGNAL` `0x1b`, `DETECTOR_SETSIGNAL` `0x1c`), which read and write
  `+0x10c` / `+0x69` exactly as the PC does.
* **The awaited instruction code differs by build** — `detector_hit` waits on `0x20` on the PC and
  `0x22` on PS2. The two script opcode tables are NOT shared; do not carry an opcode number across
  builds.
* **`AddObject__14C_using_object` is inlined on the PC** onto the shared helper
  `using_object_add_object` (50 callsites).

### 7a. Vtable slot 35 = `GetProgram`, for the whole actor family

**Slot 35 is `GetProgram`** — a `C_actor` virtual whose default answer is NULL. The shared stub
`stub_ret0_cdecl_00477700` (`return 0`) occupies it in **33** sibling vtables; only a class that
owns a `C_program` overrides it. PS2 has exactly three such symbols and they close the set:

| class | PS2 body | its `C_program` base | skew |
|---|---|---|---|
| `C_actor` | `return 0;` | - | the default |
| `C_detector` | `return this + 0x88;` | PS2 `+0x80` | **+8** |
| `C_entity` | `return this + 0xbf8;` | PS2 `+0xbf0` | **+8** |

On the PC the skew is **`0xc`**: `entity_get_program` returns `entity+0xaa8` over a base at
`+0xa9c`, and `detector_get_program` returns `+0x7c` over a base at `+0x70`.

`detector_get_program` (`0x0047ab50`) is **not folded**: it has 0 callsites and exactly 1 dataref —
the single `.rdata` word `0x0063c094` — so it sits in one vtable only and no second class can reach
it.

## 8. Related

`structs/program.md` (the `C_program` base) · `structs/entity.md` §1 (the kind enum) ·
`structs/crash_obj.md` (the previous TU) · `structs/using_object.md` (the `S_using_object` record) ·
`basic/artifacts/player-interactions.md` (the using-object menu) ·
`script-mapping/artifacts/opcode-table.md` (the base dispatcher's opcode table).

## 9. Open items

| item | state |
|---|---|
| `detector_inrange`'s mask `+0xd8` | compared against the human-type mask at `+0x5fc`; the bit meanings are untraced |
| the volume-array PS2 offsets | the PS2 `GameInit` zeroes `+0x100`/`+0xe0`/`+0xe4`/`+0x104`, not a band-shifted copy of PC `+0x118`..`+0x124`; the per-field mapping is not established |
| `NetDirect` | the PS2 body is empty; whether the PC folds it onto a shared stub slot is not measured |
