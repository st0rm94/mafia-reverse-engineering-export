# `ai_manager` — the AI root object

**Created** 2026-08-03 16:30 +0200 · **Modified** 2026-10-12 21:02 +0200

Static analysis of `Game-ghidra.exe.annotated.c` and `Game.exe`; PS2 material is in §4.

`ai_manager` is **the single object that owns every AI subsystem.** It is not a manager in the
behavioural sense — it has no tick logic of its own on PC and makes no decisions. It is a
container: one file-scope C++ object, constructed by the CRT before `main` and destroyed by
`atexit`, whose members are the AI singletons documented in their own files.

```
  ai_manager                    @ 0x00655218        THIS FILE
    +0x00  ai_check_builder     @ 0x00655218        structs/ai_check.md (no state)
    +0x04  check_seg_clusters   @ 0x0065521c        structs/check_seg.md
    +0x0c  ai_wayfind           @ 0x00655224        structs/ai_wayfind.md
    +0x58  (ai_wayfind's own vector — head ai_wayfind+0x4c)
    +0x90  police_board         @ 0x006552a8        structs/police-board.md
    +0xc4  ai_groups            @ 0x006552dc        structs/ai_group.md
    +0xec  ai_follow_manager    @ 0x00655304        structs/ai_follow_manager.md
    +0x100 ai_fight_manager     @ 0x00655318        structs/ai_fight_manager.md
    +0x110 one member: two std::vectors, +0x110 and +0x120 (§3b)
```

- **PC prefix:** `ai_manager_*`.
- **PC cluster:** `[0x00443540, 0x00443720]` for the lifecycle quartet plus the constructor.
- **PS2 method count:** 5, all accounted (§2).

**Confidence tags:** **[H]** byte-proven / body-read · **[M]** inferred · **[U]** not observed.

---

## 1. How the object is identified

`ai_manager_static_ctor` (`0x00443550`) is six bytes:

```
0x00443550   mov  ecx, 0x655218
0x00443555   jmp  ai_manager_ctor
```

The receiver is `0x00655218`. Outside the ctor/dtor thunks that address is loaded only as the
`this` of the stateless `ai_check_builder` member at `+0x00`: `vec3_rotate_by_angle_in_place` ×3
from `entity_action_tick`, `ai_check_builder_test_space_8dir` from `ai_space_coverage_compute`, and
`ai_check_builder_get_dir8_index` from `ai_space_coverage_tick` and `ai_space_coverage_prefer_dir`.
No method reads it. Every other member is referenced by its own absolute address, never through the
root.

### 1a. 🔑 The unwind funclet block is the member map

The MSVC exception-unwind block at `0x006354f0`–`0x0063556b` is the unwind table for this
ctor/dtor pair. Each stub is six bytes: *load a frame local, add a member offset, jump to that
member's destructor.* `ai_manager_dtor`'s prologue pins the two locals — `[ebp-0x14]` = `this`,
`[ebp-0x10]` = the member cursor — and the whole block decodes at once:

| stub VA | computes | absolute | destructor | member singleton |
|---|---|---|---|---|
| `0x006354f0` | `this + 0x0c` | `0x00655224` | `ai_wayfind_dtor` | **`ai_wayfind`** |
| `0x006354fb` | `this + 0x90` | `0x006552a8` | `police_board_dtor` | **`police_board`** |
| `0x00635509` | `this + 0xc4` | `0x006552dc` | `ai_groups_dtor` | **`ai_groups`** |
| `0x00635517` | `this + 0xec` | `0x00655304` | `ai_follow_manager_dtor` | **`ai_follow_manager`** |
| `0x00635525` | `this + 0x100` | `0x00655318` | `ai_fight_manager_dtor` | **`ai_fight_manager`** |

Five independently byte-proven singletons are member offsets of one object.

> 🔑 **Technique.** An unwind funclet block is a **member map of the enclosing class** — offsets
> *and* destructors — recoverable without reading a destructor body. Adjustor thunks give the
> **inheritance** graph; unwind funclets give the **composition** graph.
>
> It also tells a CLASS destructor from a member vector's: a funclet passes the **class base**,
> and MSVC emits one funclet per **member of the enclosing class**; if the symbol has exactly one
> entrypoint there is no COMDAT fold, so it is that member's destructor. A **normal-path** call
> passing `base+N` means the opposite (that is why `ai_group_vector_destroy` is a vector
> destructor, not the `ai_groups` class destructor).

---

## 2. Method families — PC ↔ PS2

Ordered by **PC VA**, PC name first. **5 of 5 PS2 methods accounted for:** 1 out-of-line +
2 inlined + 2 with no PC counterpart.

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / decisive evidence |
|---|---|---|---|---|---|
| 0x00443540 | `ai_manager_static_init` | - | STRUCTURAL | yes | CRT static-init entry: `call ai_manager_static_ctor ; jmp ai_manager_register_atexit_dtor` ¹ |
| 0x00443550 | `ai_manager_static_ctor` | - | **CONFIRMED** | yes | `mov ecx, 0x655218 ; jmp ai_manager_ctor` |
| 0x00443560 | `ai_manager_register_atexit_dtor` | - | **CONFIRMED** | yes | `push ai_manager_static_dtor ; call atexit` |
| 0x00443570 | `ai_manager_static_dtor` | - | **CONFIRMED** | yes | `mov ecx, 0x655218 ; jmp ai_manager_dtor` — the pointer handed to `atexit` |
| 0x00443580 | `ai_manager_dtor` | `__dt__10ai_managerFv` | **CONFIRMED** | yes | tears the members down in exact reverse-construction order, arm for arm ² |
| 0x00443720 | `ai_manager_ctor` | - ³ | **CONFIRMED** | yes | default-constructs the eight `std::vector` members; calls nothing (a ctor, not the `Init` method — §2a) |
| *(inlined)* | **`game_init`** | `Init__10ai_managerFv` | **CONFIRMED** | yes | §2a |
| *(inlined)* | **`game_done`** | `Destroy__10ai_managerFv` | **CONFIRMED** | yes | §2a |
| - | - | `Tick__10ai_managerFUi` | TERMINAL | yes | no PC counterpart (§4) |
| - | - | `ResetCounting__10ai_managerFv` | TERMINAL | yes | no PC counterpart (§4) |

¹ GCC uses `.ctors`, so no PS2 symbol. ² 416/888 B = 0.47 — PS2 also destroys 7 sub-objects PC
does not have. ³ GCC inlines the constructor.

### 2a. `Init` and `Destroy` are inlined into `C_game`

`game_init` (`0x005dc240`) and `game_done` (`0x005df6b0`) each contain a contiguous block that
*is* the corresponding `ai_manager` method:

```c
// game_init                                 // game_done
police_board_reset();                        police_board_end_all_pursuits();
ai_groups_clear_all_groups();                ai_follow_manager_destroy();
ai_follow_manager_clear_all_groups();        ai_fight_manager_destroy();
ai_fight_manager_clear_all_parties();        ai_groups_destroy();
                                             check_seg_clusters_clear();
```

Same set, same order as the PS2 methods — factoring, not a missing method. The AI block is
4 statements of 1682 in `game_init` and 5 of 597 in `game_done`.

🔑 **The callers keep their `game_` names.** The position of those statements belongs to
`ai_manager`; the symbol belongs to `C_game` (`game_init` = `Init__6C_gameFbb`,
`game_done` = `Done__6C_gameFv`, both CONFIRMED). The finding is recorded as a comment in the
decomp, not as a name.

---

## 3. Layout

```c
typedef struct ai_manager {              // file-scope singleton @ 0x00655218, 0x130 bytes
    /* +0x00 */ ai_check_builder  check_builder;     // [H] no fields; padded to 4 (§1)
    /* +0x04 */ check_seg_clusters clusters;         // [H] = 0x0065521c
    /* +0x0c */ ai_wayfind         wayfind;          // [H] = 0x00655224
    /* +0x58 */ std_vector         wayfind_vec;      // [H] IS ai_wayfind+0x4c — see §3a
    /* +0x90 */ police_board       board;            // [H] = 0x006552a8
    /* +0xc4 */ ai_groups          groups;           // [H] = 0x006552dc
    /* +0xec */ ai_follow_manager  follow;           // [H] = 0x00655304
    /* +0x100 */ai_fight_manager   fight;            // [H] = 0x00655318
    /* +0x110 */std_vector         tail_a;           // [H] ONE 0x20-byte member with two vectors
    /* +0x120 */std_vector         tail_b;           // [H]   (§3b); no PC code uses it
} ai_manager;                                     // 0x00655348 is the next global
```

Every `std_vector` here is the MSVC four-word shape `{allocator, begin, end, cap}` — which is
why `ai_manager_ctor` writes a byte then three zeroed pointers, eight times. The byte is the
allocator member, not a flag.

### 3a. `root+0x58` belongs to `ai_wayfind`

`ai_wayfind_dtor` frees `(root+0x0c)+0x50` = `root+0x5c`. That is exactly the vector
`ai_manager_ctor` default-constructs at head `root+0x58`. So the head is **`ai_wayfind+0x4c`**,
that class's own and only owned allocation — not an anonymous root member.

### 3b. `+0x110` — one member with two vectors

`ai_manager_ctor` builds two vectors, at `+0x110` and `+0x120`, and `ai_manager_dtor` tears both
down through ONE member cursor: `lea ebp,[esi+0x110]` is stored to the frame local the unwind
funclets read (`[ebp-0x10]`); the `+0x120` vector is released through `ebp+0x10` (element range
destroy `stub_noop_args8_00575c00`, so trivially destructible elements) and the `+0x110` vector by
`vector_dispose_at_0x04` with the cursor in ECX. The funclets at `0x00635533` and `0x00635546` rerun
that dispose through the same cursor. That is one class whose members are two vectors — the idiom
`police_board` shows at `+0x90`. The member does not belong to `ai_fight_manager`, which is one
vector ending at `+0x110`. No function outside the ctor and dtor touches `0x00655328`–`0x00655347`,
by absolute address or as a receiver, so the member has no PC code. Its likely identity is in §4 [M],
and no PC artifact can name it further: with no code reaching either vector, the element types and
the member's identity stay at [M].

---

## 4. PC vs PS2

- **Root address: PC `0x00655218` / PS2 `0x509080`** (PS2's is passed as an immediate at both
  `Init` and `Destroy` callsites inside `C_game`: `Init__6C_gameFbb` calls
  `Init__10ai_managerFv(0x509080)`, `Done__6C_gameFv` calls `Destroy__10ai_managerFv(0x509080)`).
  MSVC inlined the callee into the same callers (§2a).
- **`ai_manager` exists on both builds.** PC has a real out-of-line destructor; what it lacks is
  the PS2-only children the PS2 `Tick` exists to drive.
- **Offset anchors.** Most offsets drift because an MSVC `std::vector` is one word larger than
  PS2's `gif::vector`. PS2 `Destroy__10ai_managerFv` calls `Clear__18check_seg_clustersFv(param_1 + 4)`
  — **`+0x04` on both builds**. Every PS2 member offset below is the `this` its methods receive:
  `ai_check_builder` `0x509080`, `check_seg_clusters` `0x509084` (17 calls), `ai_wayfind` `0x509090`
  (48), the board `0x509130` (67), `ai_follow_manager` `0x50917c` (29), `ai_fight_manager` `0x50918c`
  (11), `ai_sight_manager` `0x509198` (16), `ai_panic_manager` `0x5091b8` (8). PS2 `Init`'s
  `reserve__…(param_1 + 0x90, 100)` is `ai_wayfind`'s own vector (`ai_wayfind+0x80`), not the board.
  The drift is the documented amount: PS2 `ai_groups→ai_follow_manager` = `0x20`, PC = `0x28`; PS2
  `ai_follow_manager→ai_fight_manager` = `0x10`, PC = `0x14`.

  | member | PS2 | PC |
  |---|---|---|
  | `ai_check_builder` | +0x00 | +0x00 |
  | `check_seg_clusters` | +0x04 | +0x04 |
  | `ai_wayfind` | +0x10 | +0x0c |
  | `police_board` | +0xb0 | +0x90 |
  | `ai_groups` | +0xdc | +0xc4 |
  | `ai_follow_manager` | +0xfc | +0xec |
  | `ai_fight_manager` | +0x10c | +0x100 |
  | `ai_sight_manager` | +0x118 | +0x110 [M] ¹ |
  | `ai_panic_manager` | +0x138 | - |
  | `ai_framerater` | +0x148 | - |
  | `ai_spageman` ×5 | +0x14c/+0x17c/+0x1ac/+0x1dc/+0x20c | - |
  | `ai_unlock_manager` | +0x23c | - ² |
  | counting fields | +0x248/+0x24c | - |

  `-` = no PC member; PS2-only unless noted. ¹ Two vectors, no code (§3b). ² Refactored to the
  `ai_groups+0x00` scalar.

- **The PC member at `+0x110` sits where the PS2 puts `ai_sight_manager`.** `ai_fight_manager`
  is one vector on both builds (PS2 `root+0x10c → +0x118` is `0xc`, one `gif::vector`), so the next
  member follows it directly: PC `+0x110`, PS2 `+0x118`. PS2 `ai_sight_manager` holds two pointer
  vectors (entities and corpses: `Next` / `NextCorpses`, `Tick` / `TickCorpses`) and two cursor
  words that `Init` sets to `-1`. The PC member holds two vectors, no cursors and no code. The
  identification rests on position and shape only [M].
- **`Tick__10ai_managerFUi`** drives exactly five things: `ai_framerater`, `ai_sight_manager`
  (×2), `ai_panic_manager`, and `ai_unlock_manager` — all PS2-port-only or refactored away on PC.
  There is nothing left for a PC tick to do, and no PC function exists.
- **`ResetCounting__10ai_managerFv`** writes `root+0x248` = 4 and `root+0x24c` = 1 — past the PC
  object's end (~`+0x130`). The two counters are **per-frame permits for expensive AI work**:
  consumers gate on `== 0` / `< 1` and spend by decrement; `ai_framerater::Tick` replenishes
  `+0x248` by frame-time tier (tier 0 → `−1`, tier ≥3 → `+2`, tier 4 → `+4` total). They are the
  budget half of the PS2's AI throttling layer and have no PC counterpart.
- **Object size: PC 0x130 / PS2 ~0x250.** The whole difference is the PS2-only tail — an
  independent corroboration of the separate PS2-only findings for each of those subsystems.
- **Destructor factoring.** PC out-lines each member destructor (reachable only through the
  unwind table; the normal path inlines them). PS2 inlines *everything* into
  `__dt__10ai_managerFv`, so no `__dt__9ai_groups` / `__dt__17ai_follow_manager` /
  `__dt__16ai_fight_manager` / `__dt__10ai_wayfind` / `__dt__18ai_police_nastenka` symbol exists.
- **Construction.** PS2 has no `__ct__10ai_manager` symbol — GCC emitted the compiler-generated
  constructor inline. PC keeps it out-of-line at `0x00443720`.

---

## 5. Cross-references

- `structs/check_seg.md` · `structs/ai_wayfind.md` · `structs/police-board.md` ·
  `structs/ai_group.md` · `structs/ai_follow_manager.md` · `structs/ai_fight_manager.md` — the
  six members.

---

## 6. Open items

Nothing is open.
