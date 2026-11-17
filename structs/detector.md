# `C_detector` — the script trigger volume

**Created** 2026-08-13 00:24 +0200 · **Modified** 2026-11-08 22:29 +0100

An invisible volume that runs a script when the player enters it, uses it, or hits it. It is the
class behind the game's own `detector_*` script vocabulary (§4), an actor with a **second base
class**: a `C_program` script VM embedded at `+0x70` (§4a).

| | |
|---|---|
| PC prefix | `detector_` |
| PC kind (`+0x10`) | `5`, stamped by `detector_ctor` |
| PC vtable | `g_detector_vt` @ `0x0063c008`, 39 slots (the `C_actor` chain, §3), and the `C_program` sub-vtable @ `0x0063c0a4`, 10 slots, at `+0x70` (§4a) |
| PC base class | `C_actor` (`structs/actor.md`) at `+0x00`, and `C_program` (`structs/program.md`) at `+0x70` |
| PC instance size | `0x128` (`operator_new(0x128)` in `mission_create_actor` `case 5`) |
| PC constructor | `detector_ctor` @ `0x004793b0` |
| PC translation unit | `[0x00479370, 0x0047aae0)`: the unit's CRT pair, `crash_obj_get_material_id` (a header-inline body of the previous class), then the `detector_*` functions and three forwarders; the next unit's guard `crt_static_init_guard_0047aae0` opens `C_dog`, whose head holds this class's four adjustor thunks and `detector_get_program`. `detector_get_signal` and `detector_set_signal` (`0x005c8d00`, `0x005c8d10`) are emitted next to the script dispatcher |
| PS2 class | `C_detector` (tag `10C_detector`), 26 methods, 2 helper-struct statics and 8 `@128@` adjustor thunks: 24 methods, both statics and 4 thunks mapped in §3 (19 CONFIRMED methods and both statics, 3 STRUCTURAL forwarders, 1 base stub, 4 STRUCTURAL thunks); 2 PS2-port-only methods and 4 thunks with no PC function (§7) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The class's own band follows the two bases; offsets are PC.

```c
typedef struct C_detector {                      // 0x128 bytes; PC offsets
    /* +0x000 */ C_actor        base;            // [H] the C_actor band (structs/actor.md)
    /* +0x070 */ C_program      program;         // [H] the script VM, 0x68 bytes (structs/program.md)
    /* +0x0d8 */ uint32_t       inrange_mask;    // [H] human_type mask for detector_inrange (§2a)
    /* +0x0dc */ uint8_t        signal;          // [H] the script signal byte
    /* +0x0dd */ uint8_t        waiting_for_use; // [H] set by the use wait, cleared by detector_use
    /* +0x0de */ uint8_t        waiting_for_hit; // [H] set by the hit wait, cleared by detector_hit
    /* +0x0df */ uint8_t        volumes_live;    // [H] gates detector_update and the hit event
    /* +0x0e0 */ S_using_object uo;              // [H] the embedded using-object record
    /* +0x118 */ tDynamicCollObject **colls;     // [H] one collision volume per DYNB frame (§5)
    /* +0x11c */ S_matrix      *world;           // [H] one 0x40-byte world matrix per volume
    /* +0x120 */ S_matrix      *world_inv;       // [H] the inverse of each, a parallel array
    /* +0x124 */ int32_t        coll_count;      // [H] the number of volumes
} C_detector;
```

The three volume arrays are heap arrays of `coll_count` entries, one per `DYNB` frame of the
model, or one when the model has none (§5). `uo` is the record the using-object menu lists
(`structs/using_object.md` §1a): `detector_create_using_object` fills its position, radius `+0xec`
= 1.5, owner `+0xfc`, anchor frame `+0x100`, action id `+0x10c` and position pointer `+0x114`.

## 2. Field evidence

Measured by a census of the decomp's `this+N` accesses inside the `detector_*` functions and of
every operand with each displacement in their bytes. The `C_program`-facing functions take
`this = detector + 0x70` and reach the class fields 0x70 lower (`inrange_mask` as `this+0x68`, `waiting_for_hit` as `this+0x6e`,
`volumes_live` as `this+0x6f`).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | base | H | `detector_ctor` (through `actor_ctor`; vtable, kind `+0x10` = 5), `detector_game_init` (`+0x65` = 1 when the script has no instruction), `detector_set_signal` (`+0x65` = 0), `detector_save_game_load` | the `C_actor` methods; `detector_program_process` (the frame `+0x68`, as `this - 8`) | `+0x65` is the `C_actor` save-exclusion byte (`structs/actor.md` §2): a signalled detector is saved |
| +0x070 | program | H | `detector_ctor` (through `program_ctor`, sub-vtable `0x0063c0a4`), `detector_game_init` (`program_init`; `program.act_state_owner` `+0xb4` = the detector), `detector_load_data` (the script source), `detector_duplicate_from` | the `C_program` methods of `structs/program.md`; `detector_game_init` (the instruction vector `+0x80` / `+0x84`) | the embedded base |
| +0x0d8 | inrange_mask | H | `detector_init` (= 0), `detector_load_data`, `detector_duplicate_from`, `detector_save_game_load` | `detector_save_data`, `detector_save_game_save`, `detector_program_process` | designer data: the only class field in the scene chunk |
| +0x0dc | signal | H | `detector_set_signal`, `detector_game_init` (= 0), `detector_save_game_load` | `detector_get_signal`, `detector_save_game_save` | - |
| +0x0dd | waiting_for_use | H | `detector_create_using_object`, `detector_use`, `detector_game_init` (= 0), `detector_save_game_load` | `detector_use`, `detector_save_game_save`, `detector_save_game_load` | set on load ⇒ `detector_create_using_object` |
| +0x0de | waiting_for_hit | H | `detector_program_process` (case `0x20`), `detector_hit`, `detector_game_init` (= 0), `detector_save_game_load` | `detector_hit`, `detector_save_game_save`, `detector_save_game_load`, `detector_program_process` | set on load ⇒ `detector_create_colls(0)` |
| +0x0df | volumes_live | H | `detector_game_init` (= 0), `detector_program_process` (cases `0x154` / `0x155`) | `detector_update`, `detector_hit`, `detector_program_process` | - |
| +0x0e0 | uo | H | `detector_ctor` (its defaults), `detector_create_using_object` | `detector_create_using_object`, `detector_save_game_save` (the action id `+0x10c`), `using_object_add_object`, `using_object_del_object` (`detector_game_done`; both through its address) | registered in `C_game`'s using-object list |
| +0x118 | colls | H | `detector_game_init` (= 0), `detector_create_colls`, `detector_erase_colls` | `detector_update`, `detector_create_colls`, `detector_erase_colls` | - |
| +0x11c | world | H | `detector_game_init` (= 0), `detector_create_colls`, `detector_erase_colls` | `detector_update`, `detector_create_colls`, `detector_erase_colls` | - |
| +0x120 | world_inv | H | `detector_game_init` (= 0), `detector_create_colls`, `detector_erase_colls` | `detector_update`, `detector_create_colls`, `detector_create_dynb_coll`, `detector_erase_colls` | - |
| +0x124 | coll_count | H | `detector_game_init` (= 0), `detector_create_colls`, `detector_erase_colls` | `detector_update`, `detector_create_colls`, `detector_erase_colls` | - |

The savegame record is `0xc` bytes: version byte 1, the `+0xd8` dword, the `+0xdc` / `+0xdd` /
`+0xde` bytes and the using-object id at `+0x10c`.

### 2a. The `inrange_mask` bits

`detector_inrange` counts the actors (kind `0x1b` or `2`) within its radius whose `human_type`
(`C_human+0x5fc`, a single-bit mask over 11 types, `structs/human.md`) intersects `+0xd8`. The
mask bits are therefore `human_type` bits. Four of them have writers in the code that name them:

| Value | Meaning | Evidence |
|---|---|---|
| `0x1` | the player | `player_init` |
| `0x4` | police officers | `pedestrian_manager_spawn_cop_from_pedestrian`, `police_manager_apply_state` |
| `0x10` | gang members and mafia pursuers | `pedestrian_spawn_gang_on_hit`, `traffic_car_spawn_mafia_pursuit_car` |
| `0x80` | the taxi passenger | `pedestrian_wave_at_taxi` |

No code writer names the other bits. The mask values the shipped scenes store, over all 2779
type-5 objects in the 79 `scene2.bin` files (the mask is the dword after the version byte of the
object's data chunk):

| mask | objects | example |
|---|---|---|
| `0` | 2051 | `music` (`00MENU`); scripts that never call `detector_inrange` |
| `0x1` | 666 | `speed_end` (`EXTREME`) |
| `0x3` | 17 | `detector_konec` (`MISE01`) |
| `0x10` | 14 | `detector_kurak` (`MISE02-ULICKA`) |
| `0x81` | 11 | `CIL1` (`MISE02A-TAXI`) |
| `0x2` | 7 | `detector_pacidlo` (`MISE09-PREJIMKA`) |
| `0x9` | 3 | `misto_pro_bednu` (`EXTREME`) |
| `0x8` | 3 | `detector_autodrom` (`MISE05-MESTO`) |
| `0x11` | 2 | `car_detector` (`MISE07B-SALIERY`) |
| `0x80` | 2 | `zamceni_pokoje` (`MISE08-HOTEL`) |
| `0xb` | 1 | `biff_detector` (`MISE07B-CHULIGANI`) |
| `0x7f` | 1 | `xleseni` (`MISE13-RESTAURACE`) |
| `0x71` | 1 | `dog` (`MISE13-RESTAURACE`) |

`detector_inrange` returns a count, not a player test. `detector_konec` (`MISE01`, mask `0x3`)
waits for `flt[0]>1`, and `load mise04-motorest` (`MISE04-MESTO`) waits for `flt[0] = 3`: Tommy,
Paulie and Sam all inside the radius. `detector_hoboken` and `detector_island` (`MISE04-MESTO`)
wait for the same three.

## 3. Method family

Coverage: **24** of the 26 PS2 `10C_detector` methods are in the tables below: **19 CONFIRMED**,
**3 STRUCTURAL** (the `C_program` forwarders) and **1 base stub** (`NetDirect`); both helper-struct
statics are CONFIRMED twins (§3d), and 4 of the 8 `@128@` adjustor thunks have a PC thunk
(STRUCTURAL, §3c). `ChangeSceneInit` / `ChangeSceneDone` and the other four `@128@` thunks have no
PC function (§7). The scalar-deleting destructor has no PS2 twin; `detector_get_signal` and
`detector_set_signal` have no PS2 symbol.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00479370 | `crt_static_init_guard_00479370` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00479390 | `crt_atexit_register_00479390` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x004793b0 | `detector_ctor` | instance | `__ct__10C_detectorFv` | 0x0014a880 | CONFIRMED | `actor_ctor`, `program_ctor` at `+0x70`, both vtables, kind 5, the `uo` defaults |
| 0x00479440 | `detector_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `detector_dtor`, vtable slot 16 ¹ |
| 0x00479460 | `detector_dtor` | instance | `__dt__10C_detectorFv` | 0x0014a930 | CONFIRMED | restores both vtables, `program_dtor`, then the `C_actor` destructor |
| 0x0047ab10 | `thunk_detector_scalar_deleting_dtor` | instance | `@128@__dt__10C_detectorFv` | 0x0014c780 | STRUCTURAL | sub-vtable slot 0; `sub ecx, 0x70 ; jmp detector_scalar_deleting_dtor` |

¹ No standalone PS2 twin: GCC carries the free flag inside the destructor.

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0047ab50 | `detector_get_program` | `GetProgram__10C_detectorFv` | 0x0014c6f0 | CONFIRMED | slot 35; returns the address `this + 0x7c` (§7a); 0 callsites, one dataref |
| 0x005c8d00 | `detector_get_signal` | - | - | - | `return signal`; called from the base dispatcher's `detector_issignal` arm under a `kind == 5` gate. The PS2 build inlines it into its own `Process` arm, so no symbol exists |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00461180 | `stub_noop_args4_00461180` | `NetDirect__10C_detectorFUi` | 0x0014aeb0 | base-stub | slot 15; the shared `ret 4` no-op that slot 15 holds in every class whose PS2 `NetDirect` is an empty override (§7) |
| 0x004794c0 | `detector_init` | `Init__10C_detectorFP9I3D_frame` | 0x0014a9b0 | CONFIRMED | slot 18; `actor_init`, `inrange_mask` = 0 |
| 0x004794f0 | `detector_duplicate_from` | `DuplicateFrom__10C_detectorFP7C_actor` | 0x0014a9f0 | CONFIRMED | slot 21; copies `inrange_mask` and the program (`src + 0x70`, null-guarded) |
| 0x00479530 | `detector_game_init` | `GameInit__10C_detectorFv` | 0x0014aa40 | CONFIRMED | slot 25; `actor_game_init`, `program.act_state_owner` = this, `program_init`, the save exclusion for an empty script, clears the four bytes and the volume arrays |
| 0x004795b0 | `detector_game_done` | `GameDone__10C_detectorFv` | 0x0014aab0 | CONFIRMED | slot 26; unregisters `uo` (`using_object_del_object`), `actor_game_done`, `program_done`, `detector_erase_colls` |
| 0x004795f0 | `detector_save_data` | `SaveData__10C_detectorFP7C_chunk` | 0x0014ab00 | CONFIRMED | slot 19; the scene record: `inrange_mask` and the script source |
| 0x00479650 | `detector_load_data` | `LoadData__10C_detectorFP7C_chunk` | 0x0014ab90 | CONFIRMED | slot 20 |
| 0x00479740 | `detector_save_game_get_size` | `SaveGameGetSize__10C_detectorFv` | 0x0014acb0 | CONFIRMED | slot 22 |
| 0x00479760 | `detector_save_game_save` | `SaveGameSave__10C_detectorFPPv` | 0x0014acf0 | CONFIRMED | slot 23; the `0xc`-byte record of §2 |
| 0x004797d0 | `detector_save_game_load` | `SaveGameLoad__10C_detectorFPPv` | 0x0014ad90 | CONFIRMED | slot 24; restores the record and re-arms the waits |
| 0x004798a0 | `detector_ai` | `AI__10C_detectorFUi` | 0x0014aec0 | CONFIRMED | slot 13; runs the program (`program_call_process`) while its counter is inside the instruction vector, else `program_clear_after_end` |
| 0x004798e0 | `detector_update` | `Update__10C_detectorFUi` | 0x0014af00 | CONFIRMED | slot 14; re-derives each volume's centre, radius, world matrix and inverse while `volumes_live` |
| 0x00479b50 | `detector_hit` | `Hit__10C_detectorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x0014b2d0 | CONFIRMED | slot 31; releases the hit wait |
| 0x00479be0 | `detector_create_colls` | `CreateColls__10C_detectorFb` | 0x0014b370 | CONFIRMED | builds the volumes from the `DYNB` frames (§5) |
| 0x0047a610 | `detector_erase_colls` | `EraseColls__10C_detectorFv` | 0x0014bfd0 | CONFIRMED | erases and frees each collision object, then frees the three arrays |
| 0x0047a6b0 | `detector_create_using_object` | `CreateUO__10C_detectorFib` | 0x0014c0a0 | CONFIRMED | fills `uo` and registers it (`using_object_add_object`) |
| 0x0047a790 | `detector_use` | `UseDetector__10C_detectorFv` | 0x0014c1a0 | CONFIRMED | releases the use wait |
| 0x0047a810 | `thunk_program_decode_params` | `DecodeParams__10C_detectorFPcRQ29C_program13S_instruction` | 0x0014c220 | STRUCTURAL | sub-vtable slot 5; a 5-byte forwarder to `program_decode_params` ¹ |
| 0x0047a820 | `thunk_program_decode_instruction` | `DecodeInstruction__10C_detectorFPcRi` | 0x0014c230 | STRUCTURAL | sub-vtable slot 6; a 5-byte forwarder to `program_decode_instruction` ¹ |
| 0x0047a830 | `thunk_stub_noop_args8_0060b1a0` | `DecompileCommand__10C_detectorFiPc` | 0x0014c240 | STRUCTURAL | sub-vtable slot 8; a 5-byte forwarder to a shared no-op stub ¹ |
| 0x0047a840 | `detector_program_process` | `Process__10C_detectorFUi` | 0x0014c250 | CONFIRMED | `this = +0x70` sub-vtable slot 7; the class's script instructions (§6) |
| 0x0047ab20 | `thunk_detector_save_game_get_size` | `@128@SaveGameGetSize__10C_detectorFv` | 0x0014c770 | STRUCTURAL | sub-vtable slot 2; `sub ecx, 0x70 ; jmp` |
| 0x0047ab30 | `thunk_detector_save_game_save` | `@128@SaveGameSave__10C_detectorFPPv` | 0x0014c760 | STRUCTURAL | sub-vtable slot 3; `sub ecx, 0x70 ; jmp` |
| 0x0047ab40 | `thunk_detector_save_game_load` | `@128@SaveGameLoad__10C_detectorFPPv` | 0x0014c750 | STRUCTURAL | sub-vtable slot 4; `sub ecx, 0x70 ; jmp` |
| 0x005c8d10 | `detector_set_signal` | - | - | - | `+0x65` = 0, then `signal` = the argument; called from the base dispatcher's `detector_setsignal` arm under a `kind == 5` gate; inlined on the PS2 |

¹ Named after the **target** the forwarder jumps to (`program_decode_params` `0x005afe70`,
`program_decode_instruction` `0x005b1b60`, a shared no-op stub). Each has exactly one dataref, in
this class's sub-vtable, so each is this class's own override.

The `C_program` base methods behind the positionally matched calls in these bodies are
`program_ctor`, `program_dtor`, `program_save_source_code`, `program_load_source_code`,
`program_init`, `program_done`, `program_duplicate_from` and `program_clear_after_end`
(`structs/program.md`).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0047a150 | `detector_count_dynb_frames` | `cbDummyCount__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi` | 0x0014bf30 | CONFIRMED | the frame-enumerator callback that counts the `DYNB` frames (§5); zero textual callsites |
| 0x0047a1a0 | `detector_create_dynb_coll` | `cbDummyCreate__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi` | 0x0014b9e0 | CONFIRMED | the callback that builds one volume per `DYNB` frame (§5); zero textual callsites |

Both are statics of a file-local helper struct on the PS2, passed to the enumerator as function
pointers.

## 4. What it is

An invisible volume that runs a script. The class implements the game's own `detector_*` script
vocabulary, which is why the PC prefix is `detector_`. Counted over the shipped mission scripts:

| script command | uses | implemented by |
|---|---|---|
| `detector_inrange` | 1123 | `detector_program_process`, instruction `0x1c` |
| `detector_issignal` | 567 | `detector_get_signal`: reads the BYTE at `+0xdc` |
| `detector_setsignal` | 517 | `detector_set_signal`: writes the BYTE at `+0xdc` and clears `+0x65`, the `C_actor` save-exclusion byte, so a signalled detector is saved |
| `detector_waitforuse` | 505 | `detector_program_process` `0x1f` → `detector_create_using_object`; released by `detector_use` |
| `detector_waitforhit` | 96 | `detector_program_process` `0x20` → `detector_create_colls`; released by `detector_hit` |

Whole shipped scripts are named after it: mission `MISE05-MESTO`, script `detector_bertone`;
mission `MISE07B-CHULIGANI`, script `detector_konec`.

### 4a. It has TWO base classes

`detector_ctor` installs **two** vtables: `0x0063c008` at `+0x00` (the `C_actor` chain) and
`0x0063c0a4` at `+0x70`, the `C_program` sub-object. Three independent proofs:

1. the two vptr writes in the constructor, and the matching pair of restores in the destructor;
2. every `C_program`-facing method takes `this = detector + 0x70` (`detector_duplicate_from`
   passes `src + 0x70`, null-guarded);
3. `detector_program_process` reaches the owning detector's frame as **`this - 8`**:
   `+0x70 - 8 = +0x68`, the `C_actor` frame pointer.

The `C_program` sub-vtable's slots are the script-VM interface:

| sub-slots | filled by |
|---|---|
| 0, 2, 3, 4 | MSVC adjustor thunks at the head of the NEXT translation unit: `0x0047ab10` / `ab20` / `ab30` / `ab40`, each byte-proven `sub ecx, 0x70 ; jmp <detector method>` |
| 1 | `program_save_source_code`, the base's |
| 5, 6, 8 | 5-byte forwarders emitted in this unit (`0x0047a810`, `0x0047a820`, `0x0047a830`) |
| 7 | the class's own `detector_program_process` |
| 9 | the shared `stub_noop_args4_004776f0` |

A fifth `C_detector` function sits at the head of the next unit too, `detector_get_program` (§7a).
COMDAT placement means a translation-unit sweep files all five under `C_dog`; the same placement
puts `crash_obj_get_material_id` at the head of THIS unit.

## 5. `DYNB` — the trigger volume is named in the model

`detector_create_colls` walks the bound frame's subtree twice through the engine's frame
enumerator: first with `detector_count_dynb_frames` to size the arrays, then with
`detector_create_dynb_coll` to build them. Both callbacks accept a child frame only if

* its name is at least four characters long, **and**
* its first four bytes are the literal **`DYNB`** (`0x0064b8dc`), compared as one dword.

So a level designer marks a trigger's shape by naming dummy frames `DYNB…` in the model. When a
model has no `DYNB` child at all, the class falls back to a single volume around the frame's own
bound volume. Each accepted frame becomes one `0x48`-byte `tDynamicCollObject`, and
`detector_update` re-derives its centre, radius, world matrix and inverse every frame, so a trigger
glued to a moving frame keeps up.

## 6. `detector_program_process` — the class's script instructions

| code | script command | what the case does |
|---|---|---|
| `0x1c` | `detector_inrange` | counts scriptable actors (kind `0x1b` or `2`) within a radius whose human-type mask (`+0x5fc`) intersects `+0xd8` (§2a), writes the count into a script register, then advances |
| `0x1f` | `detector_waitforuse` | releases any previous using-object, calls `detector_create_using_object`, blocks |
| `0x20` | `detector_waitforhit` | arms the hit path (`detector_hit` releases it) |
| `0x25` | `program_storage` | parks the actor in act-state 2 while program `+0x40` is 0; does not advance |
| `0x154` | `detector_createdyncoll` | no-op unless both `+0xde` and `+0xdf` are clear; sets `+0xdf`, then erase + create colls in mode 1 |
| `0x155` | `detector_erasedyncoll` | when `+0xde` is clear and `+0xdf` set: clears `+0xdf`, then erase colls |

Anything else falls through to the `C_program` base.

### 6a. Why these six commands have a default-arm slot in the base dispatcher

`C_program::Process` is **virtual**. The base `program_process_command` jump table points all six
opcodes above at its default arm precisely because this derived class owns them:
`detector_waitforuse` and `detector_waitforhit` are used in 30+ shipped scripts and work in game.
Of the 15 opcodes whose base slot is the default arm, these six are implemented here and the other
**9** are unimplemented.

**How the wait works.** The base default arm at `0x005c76da` is literally
`push 0; mov ecx,ebp; call program_next_command`, a no-op that advances the program counter. The
`0x1f` and `0x20` cases here **do not call `program_next_command`**, so the program counter stays
on the same instruction and the script re-executes it every tick until `detector_use` /
`detector_hit` releases it. Blocking is the absence of the advance, which nothing in the base
dispatcher can express.

`entity_process_command` is the OTHER override of the same virtual (it and this function are the
only two in the image that tail into `program_process_command`), but it intercepts only the
`0x1000+` AI-action instruction space, never a script opcode.

## 7. PC vs PS2

* **The `C_program` base sits at PC `+0x70` and PS2 `+0x80`.** The PS2 spells the relationship
  with eight `@128@` adjustor thunks (128 = `0x80`): `__dt`, `SaveGameGetSize`, `SaveGameSave`,
  `SaveGameLoad`, `DecodeParams`, `DecodeInstruction`, `DecompileCommand`, `Process`. The PC uses
  two vtable installs, four adjustor thunks and plain forwarders. Same relationship, two ABIs.
* **The field delta is not uniform.** Measured from both builds' `GameInit`, `SaveGameSave`,
  `EraseColls` and `Update`:

  | field | PC | PS2 | delta |
  |---|---|---|---|
  | `program.act_state_owner` | `+0xb4` | `+0xb8` | `+0x4` |
  | `inrange_mask` | `+0xd8` | `+0x108` | `+0x30` |
  | `signal` / `waiting_for_use` / `waiting_for_hit` / `volumes_live` | `+0xdc`..`+0xdf` | `+0x10c`..`+0x10f` | `+0x30` |
  | `uo` | `+0xe0` | `+0x110` | `+0x30` |
  | using-object id | `+0x10c` | `+0x144` | `+0x38` |
  | `colls` | `+0x118` | `+0x100` | `-0x18` |
  | `world` | `+0x11c` | `+0xe0` | `-0x3c` |
  | `world_inv` | `+0x120` | `+0xe4` | `-0x3c` |
  | `coll_count` | `+0x124` | `+0x104` | `-0x20` |
  | `C_actor` save-exclusion byte, cleared by `set_signal` | `+0x65` | `+0x69` | `+0x4` |

  The volume-array rows come from `EraseColls`, which is statement-aligned on both builds (walk the
  count, erase and free each collision object, then free the three arrays in the same order), and
  from `Update`, which writes the world matrix and then its inverse. The PS2 savegame record is
  `0x10` bytes against the PC's `0xc`.
* **`DYNB` compare:** one dword compare on the PC, `memcmp(name, "DYNB", 4)` on the PS2.
* **`NetDirect__10C_detectorFUi`** is an empty body on the PS2. On the PC its slot (15) holds
  `stub_noop_args4_00461180` (`ret 4`), the word that slot 15 also holds in the vtables of
  `C_entity`, `C_human`, `C_player`, `C_airplane`, `C_bridge`, `C_railway`, `C_wagon`,
  `C_traffic_car` and `C_actor_shot`: the classes whose PS2 `NetDirect` is an empty override.
  `C_actor`'s own slot 15 holds the separate `ret 4` stub `stub_noop_args4_004776f0`.
* **`detector_get_signal` / `detector_set_signal` have no PS2 symbol:** the PS2 inlines both into
  its own `Process` arms (`DETECTOR_ISSIGNAL` `0x1b`, `DETECTOR_SETSIGNAL` `0x1c`), which read and
  write `+0x10c` / `+0x69` exactly as the PC does.
* **The awaited instruction code differs by build**: `detector_hit` waits on `0x20` on the PC and
  `0x22` on the PS2. The two script opcode tables are NOT shared; do not carry an opcode number
  across builds.
* **`AddObject__14C_using_object` is inlined on the PC** onto the shared helper
  `using_object_add_object` (50 callsites).

**PS2 functions with no PC function of their own:**

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0014c5c0 | `ChangeSceneInit__10C_detectorFv` | PS2-port-only: the city-streaming pair; no unclaimed PC start remains in the unit for either |
| 0x0014c650 | `ChangeSceneDone__10C_detectorFv` | the same |
| 0x0014c710 | `@128@DecompileCommand__10C_detectorFiPc` | the PC sub-vtable slot 8 points straight at the forwarder, which takes `this = detector + 0x70`: no adjustment needed |
| 0x0014c720 | `@128@Process__10C_detectorFUi` | the same for slot 7 (`detector_program_process`) |
| 0x0014c730 | `@128@DecodeInstruction__10C_detectorFPcRi` | the same for slot 6 |
| 0x0014c740 | `@128@DecodeParams__10C_detectorFPcRQ29C_program13S_instruction` | the same for slot 5 |

### 7a. Vtable slot 35 = `GetProgram`, for the whole actor family

**Slot 35 is `GetProgram`**: a `C_actor` virtual whose default answer is NULL. The shared stub
`stub_ret0_cdecl_00477700` (`return 0`) occupies it in **33** sibling vtables; only a class that
owns a `C_program` overrides it. The PS2 has exactly three such symbols and they close the set:

| class | PS2 body | its `C_program` base | skew |
|---|---|---|---|
| `C_actor` | `return 0;` | - | the default |
| `C_detector` | `return this + 0x88;` | PS2 `+0x80` | **+8** |
| `C_entity` | `return this + 0xbf8;` | PS2 `+0xbf0` | **+8** |

On the PC the skew is **`0xc`**: `entity_get_program` returns `entity+0xaa8` over a base at
`+0xa9c`, and `detector_get_program` returns `+0x7c` over a base at `+0x70`.

`detector_get_program` (`0x0047ab50`) is **not folded**: it has 0 callsites and exactly 1 dataref,
the single `.rdata` word `0x0063c094`, so it sits in one vtable only and no second class can reach
it.

## 8. Related

- `structs/program.md` — the `C_program` base.
- `structs/actor.md` — the `C_actor` base and the slot-35 default.
- `structs/entity_kind.md` — kind 5.
- `structs/crash_obj.md` — the previous unit.
- `structs/dog.md` — the next unit, whose head holds this class's thunks.
- `structs/using_object.md` — the `S_using_object` record.
- `basic/artifacts/player-interactions.md` — the using-object menu.
- `script-mapping/artifacts/opcode-table.md` — the base dispatcher's opcode table.

## 9. Open items

The class has no open item.
