# `police_board` — the pursuit board singleton

**Created** 2026-05-26 11:43 +0200 · **Modified** 2026-09-21 23:38 +0200

The pursuit **board** singleton that owns and drives all active `police_action` squads. Functions
whose receiver is this singleton carry the `police_board_*` prefix; the contained per-squad
`police_action` objects and their `police_action_*` methods live in
[`police-action.md`](police-action.md). Several board-receiver helpers *operate on* a squad (e.g.
`police_board_end_pursuit`, `police_board_update_sighting`, `police_board_create_and_engage`) —
they still take the board as `this` and walk/mutate the matching squad.

**Relationship to `police_manager.md`:** these are **two genuinely separate structs**.
`police_manager` is the per-actor sub-object embedded at `actor + 0x2da0` (size 0xB0); this is a
global singleton. They happen to have unrelated content at overlapping offsets (e.g. both have a
`+0x24` field with completely different meanings) — they are NOT views into the same memory.

**Status:** reconstructed from `Game.exe` disassembly + decomp static analysis. No runtime
validation.

---

## 1. Identity & evidence

**Location:** **global singleton at VA `0x006552a8`**, which is also **`ai_manager + 0x90`** — the
board is a member of the `ai_manager` root object (`structs/ai_manager.md`).

**Allocation:** static — lives in the `.bss` / `.data` region. No `operator_new`; the singleton is
constructed in place at engine init.

**Evidence:**
- A byte scan of `Game.exe` finds 89 `MOV ECX, 0x006552a8` immediate loads (opcode pattern
  `B9 A8 52 65 00`) and **zero** references via any other addressing mode (no `[reg+disp]`, no
  `LEA`, no absolute `[imm32]`). Every access is `this`-pointer setup for a `__fastcall` /
  `__thiscall` call.
- 26 distinct methods are called on it through that pattern (§3).
- The 89 references cluster in the `0x004469e0..0x004487f0` VA range plus a handful of high-VA
  callers — consistent with a single singleton owned by the police subsystem.

**Not embedded in any other struct** in the `police_manager` sense: its address is baked into the
binary as an immediate, not computed via base + offset.

---

## 2. Field layout

**Size:** at least `0x34`. Fields are observed up to `+0x30`; no high-confidence field exists past
it, and the next BSS landmark needed to bound the size has not been identified.

**Confidence tags:** **[H]** disasm or body-read evidence with writer + reader · **[M]** body-read
evidence, semantics partly inferred · **[U]** never observed accessed.

```c
// Global singleton at VA 0x006552a8 (= ai_manager + 0x90).

typedef struct police_action police_action;   // 0x6c bytes, see structs/police-action.md

typedef struct police_board {
    /* +0x00 */ ai_animitem_manager anim_items;  // [H] embedded manager; its vector head is
                                                 //     +0x04/+0x08 (§5)
    /* +0x04 */ ai_anim_item*  items_begin;      // [H] stride 0x10 (§5)
    /* +0x08 */ ai_anim_item*  items_end;        // [H]
    /* +0x0c */ uint32_t       _unk_0c;          // [U]
    /* +0x10 */ uint32_t       _unk_10;          // [U]
    /* +0x14 */ police_action* squads_begin;     // [H] begin of the pursuit-squad array, stride
                                                 //     0x6c
    /* +0x18 */ police_action* squads_end;       // [H] one past last; count = (end-begin)/0x6c
    /* +0x1c */ uint32_t       squads_cap;       // [M] presumed capacity (vector triple); never
                                                 //     observed in a body read
    /* +0x20 */ uint32_t       session_id_counter; // [H] monotonic pursuit-session id
    /* +0x24 */ uint32_t       arrest_settle_timer; // [H] post-arrest settling grace timer (ms)
    /* +0x28 */ float          police_speed_factor; // [H] also addressed absolutely as
                                                 //     VA 0x006552d0
    /* +0x2c */ uint8_t        busted_cutscene_active; // [H] an arrest/fine capture cutscene is
                                                 //     animating
    /* +0x2d */ uint8_t        _pad[0x03];       // [U] padding inferred
    /* +0x30 */ actor*         tracked_npc;      // [H] suspend/resume "keep" exclusion
} police_board;
```

### 2a. Field hot-points

| Offset | Type | Field | Conf | Reset value | Notes |
|---|---|---|---|---|---|
| +0x04 / +0x08 | ptr | items_begin / items_end | H | end = begin | the embedded `ai_animitem_manager`'s `ai_anim_item` vector, stride 0x10 (§5) |
| +0x14 | ptr | squads_begin | H | unchanged | stride-0x6c vector of `police_action` |
| +0x18 | ptr | squads_end | H | = begin (cleared) | end pointer of the same vector |
| +0x1c | ptr | squads_cap | M | not written by the reset | inferred from the vector convention |
| +0x20 | u32 | session_id_counter | H | 0 | bumped by `police_board_increment_counter` ¹ |
| +0x24 | u32 | arrest_settle_timer (ms) | H | 0xfffffff | post-arrest settle grace, clamped to 12000 on ARREST only ² |
| +0x28 | float | police_speed_factor | H | 1.0f | the same field as absolute-addressed `DAT_006552d0` ³ |
| +0x2c | u8 | busted_cutscene_active | H | 0 | "a capture cutscene (arrest or fine) is animating" ⁴ |
| +0x30 | actor* | tracked_npc | H | 0 | set by `police_board_set_tracked_npc`; the suspend/resume pair `police_board_suspend_npcs` / `police_board_resume_npcs` use it as a "keep" exclusion when adjusting hold-timer offsets on all type-0x1b NPCs |

¹ `*(int*)(this+0x20) += 1`; the function returns the post-increment value in EAX despite Ghidra's
`void` declaration. The returned value is stamped into each newly-constructed `police_action`'s
`+0x30` slot as its unique pursuit-session id, which then propagates to every squad officer's
`actor+0xf54` slot via `entity_spawn_police_response`.
² Probed by `police_board_arrest_settle_is_zero` (true iff `+0x24 == 0`); clamped DOWN to 12000 by
`police_board_clamp_arrest_settle_12s` (writes 12000 only when the value exceeds it); decremented
every frame by `police_board_tick` (unconditional, floored at 0). The reset value `0xfffffff`
(≈ 74.6 h) is an effectively-never sentinel. The clamp's SOLE callsite is the ARREST branch of the
arrest/fine cutscene (`entity_action_tick` case 0x21 substate 0xb); the FINE branch does not clamp.
So `+0x24` only becomes meaningful (≤ 12 s) after an arrest, and it gates when the arrest end-screen
menu appears (consumed by `police_board_arrest_settle_is_zero` in `police_manager_tick` and
`program_process_command` case 0x33). It does NOT touch `pursuit_record+0x50`
(`wanted_substate_flag`) or the on-screen wanted bar.
³ See §2b.
⁴ Writer `police_board_set_busted_cutscene_active` (`*(u8*)(board+0x2c) = value`), reader
`police_board_is_busted_cutscene_active` (`!= 0`). Set to 1 ONLY by the arrest/fine cutscene
(`entity_action_tick` case 0x21), fired right AFTER the squad is disbanded
(`police_board_disband_by_target`), on both the arrest and the fine path; set to 0 when the
cutscene animation completes (only 2 writer callsites, both in case 0x21). So it does not mean "a
squad is chasing" — it means "the player is being captured right now". `police_manager_tick` reads
it as the gate that SUPPRESSES its live per-suspect block (clearing the player's record in the
else) while the cutscene owns the player; `POLICEITCHFORPLAYER` (case 0x12c) also reads it.
Distinct from the per-squad active bit `police_action+0x50 & 0x01` (`police-action.md` §2).

### 2b. `+0x28` is `g_police_speed_factor` at VA `0x006552d0`

One field, two access modes:

- from singleton methods (e.g. `police_board_reset`): `MOV [ECX+0x28], 0x3f800000` (ECX-relative,
  1.0f default);
- from the script-command write path (case 0x173 `POLICE_SPEED_FACTOR`) and the police-chase
  consumer `pursuit_event_dispatch_pursuit_vehicle` (8 callsites, which multiply a speed coefficient
  by the factor, clamped to `[0.0, 1.0]`): FPU absolute addressing — a scan of `Game.exe` finds 11
  references to VA `0x006552d0`, all FPU opcodes (FMUL ×6, FST ×3, FSTP ×2).

That dual pattern is the canonical compiler-emitted shape for "field on a singleton with a fixed BSS
address": base+offset in iteration/reuse contexts, absolute addressing for standalone float ops.
There is no separate `g_police_speed_factor` global; `DAT_006552d0` is an alias for
`police_board + 0x28`. Its use in a pursuit is described in
[`police-system/artifacts/boosting.md`](../police-system/artifacts/boosting.md) §2b.

### 2c. Three structs that are not this one

Offsets attributed to "the manager" in early work belong to two other objects:

| offsets | where they really live |
|---|---|
| `+0x4/+0x8`, `+0x14/+0x18`, `+0x24/+0x28` task lists | the per-NPC task subsystem at **`NPC + 0x1164`** ¹ |
| `+0x40/+0x44/+0x48` per-target tracking array | `police_manager` (`structs/police_manager.md` §3) |
| `+0x10b8`/`+0x10bc` (stride 0xc) and `+0x10c4` | `C_entity` ² |

¹ Every call to `ai_action_manager_clear_all` / `ai_action_manager_invalidate_actor` /
`ai_action_manager_retire_top_modifier` in `Game.exe` is preceded by `LEA ECX, [reg + 0x1164]` (the
only ECX-setup pattern across 23 callers). The enclosing `entity_reset` docblock confirms the base
register holds the NPC entity. The three lists are stride 4 (4-byte task pointers), entirely
separate from this singleton's stride-0x6c squad array at `+0x14/+0x18` and its uint timer at
`+0x24`.
² `entity+0x10b8`/`+0x10bc` is `C_entity`'s **substate-lock table** —
`std::vector<ai_substate_lock>`, stride 0xc, rows `{key, p1 (uint), p2 (float)}` — and
`entity+0x10c4` is `locked_state`, where `0x29a` (= 666) means **NO LOCK**; 666 is the project-wide
"none" sentinel. The accessors are `entity_unlock_substate` (`0x00434550`), `entity_lock_substate`,
`entity_has_locked_substate`, `entity_has_locked_state`, and the persistence pair
`entity_save_state` / `entity_load_state`. `structs/entity.md` §4/§5 has the detail. The
attribution is byte-level: `entity_unlock_substate` operates on `[param_1 + 0x10b8]`/`[+0x10bc]`
and all 5 of its callers pass an officer as `this` — 3 inside `entity_weapon_choose`
(`0x00432550`, whose `__fastcall (int officer)` signature fixes the receiver), 2 more in
`ai_fight_manager_disband_followship` (`0x0043f730`) with ESI sourced from an officer-typed `this`
— and a scan finds ZERO `MOV ECX, 0x006552a8` loads followed by a call to it within 32 bytes.
`entity_save_state` / `entity_load_state` (`0x00435600` / `0x004358f0`) serialise that table on the
same `this` as the per-officer fields `+0xf40` (state) / `+0xf44` (substate) / `+0xf08` (flags) /
`+0xf4c` (role) / `+0xf50` / `+0xf58` / `+0xf5c`, and `entity_get_state_size` sums `nlocks * 0xc`
into the savegame "State" block. The table carries, among others, key 3 = the pursuit-approach
sub-state published each tick by `ai_fight_manager_tick`.

---

## 3. Methods called on the singleton (26 distinct callees)

All 26 are reached via `MOV ECX, 0x006552a8; CALL <method>`. Counts are the number of distinct
`MOV ECX + CALL` immediate pairs in `Game.exe` (a function called from two places counts twice).

| VA | Function | Calls | Notes |
|---|---|---|---|
| 0x004469e0 | `police_board_reset` | 1 | clears the squad array + writes the five header fields (§6) |
| 0x00446b00 | `police_board_increment_counter` | - ¹ | bumps `+0x20` |
| 0x00446b10 | `police_board_find_group_by_officer` | 10 | walks squads, returns the one containing this officer in its inner subarray |
| 0x00446ba0 | `police_board_find_group_by_session_id` | 4 | walks squads, returns the one whose `+0x30` matches the supplied session id |
| 0x00446c00 | `police_board_create_and_engage` | 1 | allocates a new 0x6c-byte `police_action`, stamps it with the `+0x20`-derived session id |
| 0x004474a0 | `police_board_end_pursuit` | 2 | frees a `police_action` squad + collapses the end pointer down |
| 0x00447610 | `police_board_invalidate_actor` | 4 | walks squads, applies state transitions when a tracked NPC dies |
| 0x004477b0 | `police_board_tick` | 1 | the per-frame tick: decrements `+0x24` by dt and ticks each squad |
| 0x00447820 | `police_board_arrest_settle_is_zero` | 2 | tests `+0x24 == 0` (arrest-settle done) |
| 0x00447830 | `police_board_raise_gun_escalation` | 4 | loud GUN alarm fan-out (raises committed `+0x3c` = 3) ² |
| 0x00447950 | `police_board_raise_handcuffs_escalation` | 2 | loud HANDCUFFS alarm fan-out (raises committed `+0x3c` = 2) ² |
| 0x00447a90 | `police_board_disband_by_target` | 1 | disbands all squads targeting an NPC |
| 0x00447b00 | `police_board_clamp_arrest_settle_12s` | 1 | caps `+0x24` at 12000 ms (on ARREST only) |
| 0x00447b10 | `police_board_promote_handcuffs_escalation` | 1 | silent commit half of the handcuffs raise (squad pair (4,5)) |
| 0x00447bb0 | `police_board_promote_gun_escalation` | 1 | silent commit half of the gun raise (squad pair (0,1)) |
| 0x00447c40 | `police_board_update_sighting` | 10 | refreshes the per-squad sighting timestamp |
| 0x00448150 | `police_board_target_matches` | 1 | predicate: any squad has target == npc |
| 0x004481c0 | `police_board_refresh_target_vehicle` | 1 | refreshes the cached vehicle pointer for a squad |
| 0x00448230 | `police_board_has_recent_sighting` | 2 | predicate: any squad sighted within the last 2000 ms |
| 0x00448350 | `police_board_broadcast_for_target` | 1 | fans event 0x17 to every squad targeting an NPC |
| 0x004483b0 | `police_board_is_busted_cutscene_active` | 2 | reads `board+0x2c` (`!= 0`); a single byte read, does NOT walk squads |
| 0x004483c0 | `police_board_set_busted_cutscene_active` | 2 | writes `board+0x2c` (1 = cutscene started, 0 = ended); both callsites are in `entity_action_tick` case 0x21 |
| 0x004483d0 | `police_board_is_actor_in_pursuit` | 3 | predicate: is `actor` (kind 0x1b NPC or **2 = player**) the tracked target (`squad+0x60`) or an officer (`squad+0x4`/`+0x8`) of any squad? |
| 0x00448620 | `police_board_has_active_group_for_target` | 2 | true if a squad has target == npc AND flag bit 0 set |
| 0x00448680 | `police_board_set_tracked_npc` | 1 | stores npc at `+0x30` |
| 0x00448690 | `police_board_end_all_pursuits` | 1 | ends every squad's pursuit + clears the anim-item vector (§5) |
| 0x004486f0 | `police_board_release_all_group_vehicles` | 1 | deactivates each squad's vehicle (`+0x5c`) and detaches officers. Called only by `program_process_command` case 0x183 = `M7_ZASTAVZKURVENEPOLISE` |
| 0x004487f0 | `thunk_ai_animitem_manager_clear` | 2 | JMP thunk into `ai_animitem_manager_clear` (`0x00448800`), §5 |

¹ Not called through the singleton pattern: `police_board_create_and_engage` calls it with the
board already in ECX.
² The escalation tiers follow `PM_SHOWSYMBOL`: 2 = handcuffs, 3 = gun.

**What event `0x17` does to the officers that receive it.** `police_board_broadcast_for_target`
hands each matching squad to `police_action_broadcast_event(squad, sender, 0x17, target)`, which
walks the squad's officer roster at `squad+0x04`/`+0x08` and calls
`entity_dispatch_event(sender, 0x17, target)` on every officer except the sender. That handler's
`case 0x17` does exactly one thing: if the receiving officer's base job is an `ai_action` of type
`0x21` whose payload `+0x58` is this same target, and the sender is a different entity, it pushes
that action's `+0x50` deadline out to `mission_clock + 1000` ms. Every other path returns without
writing anything. So `0x17` is a **deconfliction signal** — "somebody else is already working this
target, hold yours for a second" — and it carries no audio, no state change and no escalation. Its
one producer is `entity_action_tick` case `0x21`.

**All of these take the board as `this`.** Those that act "per squad"
(`police_board_update_sighting`, `police_board_create_and_engage`, `police_board_end_pursuit`,
`police_board_raise_gun_escalation`, `police_board_target_matches`, …) are called once per action
and walk all matching squads. The genuinely per-squad `police_action_*` methods live in
[`police-action.md`](police-action.md) §4.

---

## 4. Ownership and teardown

The board is a member of the `ai_manager` root object at **`ai_manager + 0x90`** (full picture:
`structs/ai_manager.md`). The board's own head at `+0x00` is a `std::vector` head (§5).

| PC VA | PC name | what it does |
|---|---|---|
| 0x00443860 | `police_board_dtor` | walks the `police_action` array at `+0x14`/`+0x18` (stride 0x6c); per squad drains and frees its inner vector at `action+0x04` (stride 4); then frees the array buffer at `+0x14` and the vector at `+0x04` |
| 0x00443b60 | `police_action_destroy_range` | stride `0x6c` range destroy; both callers are board paths (`ai_manager_dtor`'s police_board phase and `police_board_create_and_engage`) |

The destructor's receiver is byte-proven by the unwind funclet `add ecx, 0x90 ; jmp <here>`, and it
has exactly one entrypoint (so no COMDAT fold) — it is the class destructor. The normal path never
calls it: `ai_manager_dtor` (`0x00443580`) inlines the same teardown, which is why an `E8` caller
scan reports zero callers.

`vec_owner_scalar_deleting_dtor` (`0x00443b90`) is **not** part of this family: its body is
`vector_dispose_at_0x04(); if (flag&1) delete this;`, and its two callsites are the stride-`0x48`
(`ai_fight_followship`) and stride-`0x38` (`ai_follow_group`) loops inside `ai_manager_dtor` — two
element types, so MSVC COMDAT-folded it and it belongs to no class.

---

## 5. `ai_animitem_manager` at `police_board+0x00` and the `+0x04` vector

The `ai_animitem_manager` class (2 methods) is not free-floating: both its methods are called on the
`police_board` `this` **unmodified**, so it sits at **`police_board+0x00`**. Its element type is
`ai_anim_item`, stride `0x10`, whose element `+0xc` holds a polymorphic object pointer.

| PC name | Called from | What it does |
|---|---|---|
| `ai_animitem_manager_init` (`0x00448900`) | `police_board_reset` | empties the item vector (`end = begin`), no teardown |
| `ai_animitem_manager_clear` (`0x00448800`) | `police_board_end_all_pursuits`, plus 2 callsites through the thunk at `0x004487f0` | reverse-walks the items, releasing each element's `+0xc` object via **three vtable calls** (PC slots `+0x2c`, `+0x24`, `+0x00` = dtor, arg counts 3/2/1), nulls the slot, then erases |

Each entry's `+0x00`/`+0x04`/`+0x08` are three payload dwords, copied as-is during the
shift-compact tail of `ai_animitem_manager_clear`.

Each method has exactly **one** callsite, which is what licenses the class attribution rather than a
shared-helper reading.

**Where entries are pushed is unknown.** A scan of `Game.exe` finds **zero absolute references** to
`singleton+0x04` (`0x006552ac`) or `singleton+0x08` (`0x006552b0`), and no inlined push via
`ADD [singleton+0x08], imm8`; all vector-field accesses are ECX-relative, so the push-side helper
must be inside one of the 26 methods of §3 (§8).

---

## 6. PC vs PS2

**Class names.** The board is PS2 `ai_police_nastenka` (*nástěnka* = "notice board"); the contained
squad is `ai_police_action`; the embedded item manager is `ai_animitem_manager`
(`Init__19ai_animitem_managerFv` / `Clear__19ai_animitem_managerFv`), typed by the container lever
`erase__…FP12ai_anim_item`.

**Twin table** (PC VA order, PC name first):

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| 0x004469e0 | `police_board_reset` | `ZacatekSmeny__18ai_police_nastenkaFv` ("start of shift") | CONFIRMED |
| 0x00446c00 | `police_board_create_and_engage` | `MamProVasAkciHosi__18ai_police_nastenka` | CONFIRMED |
| 0x004474a0 | `police_board_end_pursuit` | `AkceDokoncenaPlukovniku__18ai_police_nastenka` | CONFIRMED |
| 0x00447610 | `police_board_invalidate_actor` | `ZinvalidujAktora__18ai_police_nastenka` | CONFIRMED |
| 0x004477b0 | `police_board_tick` | `Tick__18ai_police_nastenka` | CONFIRMED |
| 0x004483d0 | `police_board_is_actor_in_pursuit` | `SemTrefenaLocklaKurva__18ai_police_nastenka` ¹ | CONFIRMED |
| 0x00448690 | `police_board_end_all_pursuits` | `KonecSmeny__18ai_police_nastenkaFv` ("end of shift") | CONFIRMED |
| 0x004486f0 | `police_board_release_all_group_vehicles` | `ZastavMotorizovaneJednotky__18ai_police_nastenka` | CONFIRMED |
| 0x00443860 | `police_board_dtor` | - ² | STRUCTURAL |
| 0x00448800 | `ai_animitem_manager_clear` | `Clear__19ai_animitem_managerFv` | CONFIRMED |
| 0x00448900 | `ai_animitem_manager_init` | `Init__19ai_animitem_managerFv` | CONFIRMED |

¹ A developer joke ("hit-and-locked, dammit"); the body tests squad membership, not a hit/lock flag.
² Inlined into `__dt__10ai_managerFv` on PS2.

**The board's offset inside `ai_manager` is identical on both builds:** PS2
`Init__10ai_managerFv` opens `reserve__…(param_1 + 0x90, 100)`, and PC `police_board+0x00` is
exactly a `std::vector` head. The reserve also says the board is sized for **100** entries.

**Reset / end-of-shift diff.** `police_board_reset`'s five-field scalar block matches
`ZacatekSmeny` exactly, in the same order, with the same constants, at a uniform **+0x08** rebase:

| PS2 | PC | value |
|---|---|---|
| +0x18 | +0x20 | `0` |
| +0x1c | +0x24 | `0xfffffff` — a distinctive 28-bit mask |
| +0x24 (byte) | +0x2c (byte) | `0` — a byte among dwords |
| +0x20 | +0x28 | `0x3f800000` (1.0f) |
| +0x28 | +0x30 | `0` |

The vector triple rebases identically (PS2 `+0xc`/`+0x10` ↔ PC `+0x14`/`+0x18`) and the element
stride is each build's `police_action` record (**PC `0x6c`**, PS2 `0x80`). **The `+0x08` delta
decomposes exactly:** `ai_animitem_manager` (at `police_board+0x00`) grows `0xc` → `0x10` because
an MSVC `std::vector` carries an allocator word, **and** the board's own vector gains the same word
— 4 + 4 = 8.

`police_board_end_all_pursuits` ↔ `KonecSmeny` is the mirror at the other end of the scene: PS2
`Done__6C_gameFv` calls it exactly where PC `game_done` calls this function. Both bodies iterate the
`police_action` vector **in reverse**, call `police_board_end_pursuit` ↔ `AkceDokoncenaPlukovniku`
on every record, and finish with the anim-item clear at the same position.

**Build differences.** PS2 `ZacatekSmeny` *frees* the record buffer (`__dl__`) and zeroes
begin/end/cap; PC `police_board_reset` only resets `end = begin`, keeping the capacity. Vtable
**slot numbers** differ for the anim-item teardown (PS2 `0x40`/`0x38`/`0x8` vs PC
`0x2c`/`0x24`/`0x0`) — vtables do not slot-align across builds (`structs/actor.md`); the order and
arity are what match.

---

## 7. Cross-references

- [`police-action.md`](police-action.md) — the contained `police_action` squad struct (0x6C bytes)
  owned by this board at `+0x14`/`+0x18`: field evidence, the squad FSM, the `police_action_*`
  family and the Cheat Engine layout. Each squad is created by `police_board_create_and_engage` and
  ticked by `police_action_tick`, driven from `police_board_tick`.
- [`police_manager.md`](police_manager.md) — the other struct people call "the manager": per-actor
  sub-object at `actor + 0x2da0`. Distinct from this singleton.
- [`ai_manager.md`](ai_manager.md) — the root object this board is a member of.
- [`entity.md`](entity.md) §4/§5 — the substate-lock table and `locked_state` (§2c).
- `police-system/artifacts/police-system.md` — diagrams and flow descriptions of the pursuit-squad lifecycle.
- `ghidra/Game-ghidra.exe.functions.md` — the rows for the methods called on this singleton.

---

## 8. Open items

| item | state |
|---|---|
| upper bound on the struct size | confirmed to `+0x34`; the next BSS landmark after `0x006552d0` has not been identified |
| `+0x0c` / `+0x10` | never observed accessed |
| `+0x1c` (`squads_cap`) | presumed capacity from the vector triple; no body read touches it, and `police_board_reset` does not write it |
| what the `ai_anim_item` entries represent | "3 payload dwords + a polymorphic object with a 3-method cleanup vtable" matches an event/subscriber/handler list, but no writer is identified |
| where items are pushed into the `+0x04` vector | must be inside one of the 26 methods of §3 (all access is ECX-relative). Candidates to body-read for the alloc+emplace pattern (`operator_new(0x10)` → init 3 dwords + object pointer at `+0x0c` → store at `*end` → bump end): `police_board_raise_gun_escalation`, `police_board_raise_handcuffs_escalation`, `police_board_invalidate_actor`, `police_board_create_and_engage`, `police_board_set_tracked_npc` |
| runtime validation | none of the layout has been confirmed in game |
