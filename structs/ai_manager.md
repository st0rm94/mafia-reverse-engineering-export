# `ai_manager` — the AI root object

**Created** 2026-08-03 16:30 +0200 · **Modified** 2026-11-03 23:17 +0100

`ai_manager` is **the single object that owns every AI subsystem.** It is not a manager in the behavioural
sense: it has no tick logic of its own on PC and makes no decisions. It is a container, one file-scope C++
object, constructed by the CRT before `main` and destroyed by `atexit`, whose members are the AI singletons
documented in their own files.

```
  ai_manager                    at 0x00655218
    +0x00  ai_check_builder     at 0x00655218        structs/ai_check.md (no state)
    +0x04  check_seg_clusters   at 0x0065521c        structs/check_seg.md
    +0x0c  ai_wayfind           at 0x00655224        structs/ai_wayfind.md
    +0x90  police_board         at 0x006552a8        structs/police-board.md
    +0xc4  ai_groups            at 0x006552dc        structs/ai_group.md
    +0xec  ai_follow_manager    at 0x00655304        structs/ai_follow_manager.md
    +0x100 ai_fight_manager     at 0x00655318        structs/ai_fight_manager.md
    +0x110 one member: two std::vectors, +0x110 and +0x120 (§6b)
```

| | |
|---|---|
| PC prefix | `ai_manager_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x130` (`0x00655348` is the next global) |
| PC constructor | `ai_manager_ctor` @ `0x00443720` |
| PC translation unit | `[0x00443510, 0x00443bb0)`, guard to guard: 17 starts, all named |
| PS2 class | `ai_manager` (tag `10ai_manager`), 5 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Offsets from `0x00655218`. Every vector inside a member is the MSVC four-word shape `{allocator, begin, end,
cap}`, which is why `ai_manager_ctor` writes a byte then three zeroed pointers, eight times; the byte is the
allocator member, not a flag.

```c
typedef struct ai_manager {                   // file-scope singleton at 0x00655218, 0x130 bytes
    /* +0x00 */ ai_check_builder check_builder; // [H] no fields, padded to 4 (§4)
    /* +0x04 */ check_seg_clusters clusters;  // [H] the map's segment store
    /* +0x0c */ ai_wayfind wayfind;           // [H] the route planner, 0x84 bytes (§6a)
    /* +0x90 */ police_board board;           // [H] the police squad board
    /* +0xc4 */ ai_groups groups;             // [H] the AI groups
    /* +0xec */ ai_follow_manager follow;     // [H] the follow groups
    /* +0x100 */ ai_fight_manager fight;      // [H] the fight parties
    /* +0x110 */ std_vector unread_vector_110; // [U] built by ai_manager_ctor, freed by ai_manager_dtor; no reader (.text scan); §6b
    /* +0x120 */ std_vector unread_vector_120; // [U] built by ai_manager_ctor, freed by ai_manager_dtor; no reader (.text scan); §6b
} ai_manager;
```

## 2. Field evidence

Each member's own cells are in its own doc. A cell here names the member's method family, plus what
`ai_manager_ctor` and `ai_manager_dtor` touch; the receiver lists come from a scan of `.text` for each
member's absolute address.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | check_builder | H | - | `ai_check_builder_get_dir8_index`, `ai_check_builder_test_space_8dir`, `vec3_rotate_by_angle_in_place` | no fields; the address is only loaded as `this` (§4) |
| +0x4 | clusters | H | `check_seg_clusters_*` | `check_seg_clusters_*`, `ai_wayfind_plan_route_via_corner`, `ai_loc_update`, `ai_path_find_segment_for_node`, `entity_save_action_man`, `entity_load_action_man` | `structs/check_seg.md`; `ai_manager_ctor` does not touch it |
| +0xc | wayfind | H | `ai_wayfind_*`, `ai_manager_ctor` (the route vector at `+0x58`) | `ai_wayfind_*`, `ai_manager_dtor` | `structs/ai_wayfind.md`; `mov ecx, 0x655224` in `entity_action_tick` 92x, `entity_process_command` 8x, `entity_script_tick` 5x, `dog_replan_path`, `dog_set_destination_and_plan`, `ai_fight_modes_tick`, `ai_loc_update` |
| +0x90 | board | H | `police_board_*`, `ai_manager_ctor` | `police_board_*`, `ai_manager_dtor` | `structs/police-board.md`; the vector head at `+0xa0` |
| +0xc4 | groups | H | `ai_groups_*`, `ai_manager_ctor` | `ai_groups_*`, `ai_manager_dtor` | `structs/ai_group.md`; the vector heads at `+0xcc` and `+0xdc` |
| +0xec | follow | H | `ai_follow_manager_*`, `ai_manager_ctor` | `ai_follow_manager_*`, `ai_manager_dtor` | `structs/ai_follow_manager.md` |
| +0x100 | fight | H | `ai_fight_manager_*`, `ai_manager_ctor` | `ai_fight_manager_*`, `ai_manager_dtor` | `structs/ai_fight_manager.md`; one vector ending at `+0x110` |
| +0x110 | unread_vector_110 | U | `ai_manager_ctor` | `ai_manager_dtor` | no function outside the ctor and dtor touches `0x00655328..0x00655347` (`.text` scan); Open item 1 |
| +0x120 | unread_vector_120 | U | `ai_manager_ctor`, `ai_manager_dtor` | `ai_manager_dtor` | the same; freed through `stub_noop_args8_00575c00`, so its elements have no destructor |

## 3. Method family

5 of 5 PS2 methods accounted for: 1 with a PC function, 2 inlined into `C_game` methods and 2 with no PC
counterpart, all in §7. The destructors `ai_wayfind_dtor`, `police_board_dtor`, `ai_groups_dtor`,
`ai_follow_manager_dtor` and `ai_fight_manager_dtor` sit in this unit and are rows of their own classes' docs.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443510 | `crt_static_init_guard_00443510` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00443530 | `crt_atexit_register_00443530` | the translation unit | - | - | - | registers the shared no-op static destructor |
| 0x00443540 | `ai_manager_static_init` | `0x00655218` | - | - | STRUCTURAL | `.CRT$XCU` entry: `call ai_manager_static_ctor ; jmp ai_manager_register_atexit_dtor` |
| 0x00443550 | `ai_manager_static_ctor` | `0x00655218` | - | - | CONFIRMED | `mov ecx, 0x655218 ; jmp ai_manager_ctor` |
| 0x00443560 | `ai_manager_register_atexit_dtor` | `0x00655218` | - | - | CONFIRMED | `push ai_manager_static_dtor ; call atexit` |
| 0x00443570 | `ai_manager_static_dtor` | `0x00655218` | - | - | CONFIRMED | `mov ecx, 0x655218 ; jmp ai_manager_dtor`, the pointer handed to `atexit` |
| 0x00443580 | `ai_manager_dtor` | `0x00655218` | `__dt__10ai_managerFv` | 0x0036ac50 | CONFIRMED | tears the members down in reverse construction order |
| 0x00443720 | `ai_manager_ctor` | `0x00655218` | - | - | CONFIRMED | default-constructs the eight `std::vector` members and calls nothing else of the members; a constructor, not the `Init` method (§5) |

### 3b. Accessors

None: the class has no field access outside its construction.

### 3c. Behaviour

None: the class has no behaviour on PC (§5).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00443b00 | `std_vector_construct_empty` | - | - | - | the empty-allocator call `ai_manager_ctor` makes once |
| 0x00443b20 | `ai_group_vector_destroy` | - | - | - | frees the `ai_groups` vector at `+0xcc` for `ai_manager_dtor` |
| 0x00443b60 | `police_action_destroy_range` | - | - | - | destroys the board's action records for `ai_manager_dtor` |
| 0x00443b90 | `vec_owner_scalar_deleting_dtor` | - | - | - | the element destructor of the follow-group and fight-party vectors (`ai_manager_dtor`, stride `0x38` and `0x48`) |

## 4. How the object is identified

`ai_manager_static_ctor` is six bytes:

```
0x00443550   mov  ecx, 0x655218
0x00443555   jmp  ai_manager_ctor
```

The receiver is `0x00655218`. A scan of `.text` for that immediate finds, outside `ai_manager_static_ctor`
and `ai_manager_static_dtor`, only the `this` of the stateless `ai_check_builder` member at `+0x00`:
`vec3_rotate_by_angle_in_place` 3x from `entity_action_tick`, `ai_check_builder_test_space_8dir` from
`ai_space_coverage_compute`, and `ai_check_builder_get_dir8_index` from `ai_space_coverage_tick` and
`ai_space_coverage_prefer_dir`. No method reads it. Every other member is referenced by its own absolute
address, never through the root.

### 4a. The unwind funclet block is the member map

The MSVC exception-unwind block at `0x006354f0`–`0x0063556b` is the unwind table for this ctor / dtor pair.
Each stub is six bytes: load a frame local, add a member offset, jump to that member's destructor.
`ai_manager_dtor`'s prologue pins the two locals — `[ebp-0x14]` = `this`, `[ebp-0x10]` = the member cursor
— and the whole block decodes at once:

| stub VA | computes | absolute | destructor | member singleton |
|---|---|---|---|---|
| `0x006354f0` | `this + 0x0c` | `0x00655224` | `ai_wayfind_dtor` | `ai_wayfind` |
| `0x006354fb` | `this + 0x90` | `0x006552a8` | `police_board_dtor` | `police_board` |
| `0x00635509` | `this + 0xc4` | `0x006552dc` | `ai_groups_dtor` | `ai_groups` |
| `0x00635517` | `this + 0xec` | `0x00655304` | `ai_follow_manager_dtor` | `ai_follow_manager` |
| `0x00635525` | `this + 0x100` | `0x00655318` | `ai_fight_manager_dtor` | `ai_fight_manager` |

Five independently byte-proven singletons are member offsets of one object.

An unwind funclet block is a **member map of the enclosing class**, offsets and destructors, recoverable
without reading a destructor body. Adjustor thunks give the inheritance graph; unwind funclets give the
composition graph. It also tells a class destructor from a member vector's: a funclet passes the class base,
and MSVC emits one funclet per member of the enclosing class; if the symbol has exactly one entry point there
is no COMDAT fold, so it is that member's destructor. A normal-path call passing `base+N` means the opposite
(that is why `ai_group_vector_destroy` is a vector destructor, not the `ai_groups` class destructor).

## 5. The lifecycle sits in `C_game`

`game_init` (`0x005dc240`) and `game_done` (`0x005df6b0`) each contain a contiguous block that is the
class's lifecycle:

```
game_init:                                   game_done:
police_board_reset();                        police_board_end_all_pursuits();
ai_groups_clear_all_groups();                ai_follow_manager_destroy();
ai_follow_manager_clear_all_groups();        ai_fight_manager_destroy();
ai_fight_manager_clear_all_parties();        ai_groups_destroy();
                                             check_seg_clusters_clear();
```

The AI block is 4 statements of 1682 in `game_init` and 5 of 597 in `game_done`. The callers keep their
`game_` names: the position of those statements belongs to `ai_manager`, the symbol to `C_game`. The finding
is recorded as a comment in the decomp, not as a name.

## 6. Members worth a note

### 6a. `+0x58` belongs to `ai_wayfind`

`ai_wayfind_dtor` frees `(root+0x0c)+0x50` = `root+0x5c`. That is exactly the vector `ai_manager_ctor`
default-constructs at head `root+0x58`, so the head is `ai_wayfind+0x4c`, that class's own and only owned
allocation (`structs/ai_wayfind.md` §1).

### 6b. `+0x110` — one member with two vectors

`ai_manager_ctor` builds two vectors, at `+0x110` and `+0x120`, and `ai_manager_dtor` tears both down through
one member cursor: `lea ebp,[esi+0x110]` is stored to the frame local the unwind funclets read (`[ebp-0x10]`);
the `+0x120` vector is released through `ebp+0x10` (element range destroy `stub_noop_args8_00575c00`, so
trivially destructible elements) and the `+0x110` vector by `vector_dispose_at_0x04` with the cursor in ECX.
The funclets at `0x00635533` and `0x00635546` rerun that dispose through the same cursor. That is one class
whose members are two vectors, the idiom `police_board` shows at `+0x90`. The member does not belong to
`ai_fight_manager`, which is one vector ending at `+0x110`. With no code reaching either vector, their element
types and the member's identity stay open (§7, Open item 1).

## 7. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x0036a5e0 | `Init__10ai_managerFv` | inlined into `game_init` (§5) |
| 0x0036a760 | `Destroy__10ai_managerFv` | inlined into `game_done` (§5) |
| 0x0036a6f0 | `Tick__10ai_managerFUi` | it drives five things, `ai_framerater`, `ai_sight_manager` (2x), `ai_panic_manager` and `ai_unlock_manager`, all PS2-port-only or refactored away on PC, so nothing is left for a PC tick to do |
| 0x00368950 | `ResetCounting__10ai_managerFv` | it writes `root+0x248` = 4 and `root+0x24c` = 1, past the PC object's end; the two counters are per-frame permits for expensive AI work, the budget half of the PS2's AI throttling layer |

- **Root address: PC `0x00655218` / PS2 `0x509080`.** PS2's is passed as an immediate at both `Init` and
  `Destroy` callsites inside `C_game`: `Init__6C_gameFbb` calls `Init__10ai_managerFv(0x509080)`,
  `Done__6C_gameFv` calls `Destroy__10ai_managerFv(0x509080)`. Same set and same order as the two PC blocks of
  §5, so factoring, not a missing method.
- **The twin of `ai_manager_dtor`.** 416 / 888 B = 0.47; PS2 also destroys 7 sub-objects PC does not have. It
  inlines everything into `__dt__10ai_managerFv`, so no `__dt__9ai_groups` / `__dt__17ai_follow_manager` /
  `__dt__16ai_fight_manager` / `__dt__10ai_wayfind` / `__dt__18ai_police_nastenka` symbol exists, while the PC
  out-lines each member destructor, reachable only through the unwind table.
- **Construction.** GCC uses `.ctors`, so the static quartet has no PS2 symbol, and it emitted the
  compiler-generated constructor inline, so `__ct__10ai_manager` does not exist.
- **Offset anchors.** Most offsets drift because an MSVC `std::vector` is one word larger than PS2's
  `gif::vector`. PS2 `Destroy__10ai_managerFv` calls `Clear__18check_seg_clustersFv(param_1 + 4)`: `+0x04` on
  both builds. Every PS2 member offset below is the `this` its methods receive: `ai_check_builder` `0x509080`,
  `check_seg_clusters` `0x509084` (17 calls), `ai_wayfind` `0x509090` (48), the board `0x509130` (67),
  `ai_follow_manager` `0x50917c` (29), `ai_fight_manager` `0x50918c` (11), `ai_sight_manager` `0x509198` (16),
  `ai_panic_manager` `0x5091b8` (8). PS2 `Init`'s `reserve__…(param_1 + 0x90, 100)` is `ai_wayfind`'s own
  vector (`ai_wayfind+0x80`), not the board. The drift is the documented amount: PS2
  `ai_groups→ai_follow_manager` = `0x20`, PC = `0x28`; PS2 `ai_follow_manager→ai_fight_manager` = `0x10`, PC =
  `0x14`.

  | member | PS2 | PC |
  |---|---|---|
  | `ai_check_builder` | +0x00 | +0x00 |
  | `check_seg_clusters` | +0x04 | +0x04 |
  | `ai_wayfind` | +0x10 | +0x0c |
  | `police_board` | +0xb0 | +0x90 |
  | `ai_groups` | +0xdc | +0xc4 |
  | `ai_follow_manager` | +0xfc | +0xec |
  | `ai_fight_manager` | +0x10c | +0x100 |
  | `ai_sight_manager` | +0x118 | +0x110 [M] |
  | `ai_panic_manager` | +0x138 | - |
  | `ai_framerater` | +0x148 | - |
  | `ai_spageman` x5 | +0x14c / +0x17c / +0x1ac / +0x1dc / +0x20c | - |
  | `ai_unlock_manager` | +0x23c | - (refactored to the `ai_groups+0x00` scalar) |
  | counting fields | +0x248 / +0x24c | - |

  `-` means no PC member.
- **The PC member at `+0x110` sits where the PS2 puts `ai_sight_manager`.** `ai_fight_manager` is one vector on
  both builds (PS2 `root+0x10c → +0x118` is `0xc`, one `gif::vector`), so the next member follows it directly:
  PC `+0x110`, PS2 `+0x118`. PS2 `ai_sight_manager` holds two pointer vectors (entities and corpses: `Next` /
  `NextCorpses`, `Tick` / `TickCorpses`) and two cursor words that `Init` sets to `-1`. The PC member holds two
  vectors, no cursors and no code. The identification rests on position and shape only [M].
- **The two counters.** Consumers gate on `== 0` / `< 1` and spend by decrement; `ai_framerater::Tick`
  replenishes `+0x248` by frame-time tier (tier 0 → `-1`, tier 3 and up → `+2`, tier 4 → `+4` total).
- **Object size: PC `0x130` / PS2 about `0x250`.** The whole difference is the PS2-only tail, an independent
  corroboration of the separate PS2-only findings for each of those subsystems.

## 8. Related

- `structs/check_seg.md`, `structs/ai_wayfind.md`, `structs/police-board.md`, `structs/ai_group.md`,
  `structs/ai_follow_manager.md`, `structs/ai_fight_manager.md` — the six members with state.
- `structs/ai_check.md` — the stateless member at `+0x00`.

## 9. Open items

| Item | State |
|---|---|
| 1. `unread_vector_110`, `unread_vector_120` (§6b) | no PC code reaches either vector, so no PC artifact can name their element types; the identity as the `ai_sight_manager` slot stays [M] |
