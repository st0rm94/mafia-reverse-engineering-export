# `police_board` — the pursuit board singleton

**Created** 2026-05-26 11:43 +0200 · **Modified** 2026-11-08 17:25 +0100

The singleton that owns and drives every active `police_action` squad (`structs/police-action.md`).
Functions whose receiver is the board carry the `police_board_` prefix; several of them operate on
one squad (`police_board_end_pursuit`, `police_board_update_sighting`,
`police_board_create_and_engage`) but still take the board as `this` and walk the matching squads.
The board embeds a second, vestigial class at `+0x00`, `ai_animitem_manager` (§1a), whose two
methods are listed here with the receiver named in their Role.

It is not `police_manager`: that is the sub-object embedded once in `C_game` at `+0x2da0` (size
`0xb0`, `structs/police_manager.md`). The two have unrelated content at overlapping offsets (both
have a `+0x24` field with different meanings) and are not views of the same memory.

| | |
|---|---|
| PC prefix | `police_board_`, `ai_animitem_manager_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x34`: the next member of `ai_manager`, `ai_groups`, starts at `0x006552dc` = board `+0x34` (`structs/ai_manager.md`) |
| PC constructor | - (a static member of the `ai_manager` root, initialised by `ai_manager_ctor`; `police_board_reset` starts each scene) |
| PC translation unit | `[0x00443bb0, 0x00449ba0)`, shared with `police_action` and hosted here (§4); `police_board_dtor` sits in the preceding `ai_manager` unit |
| PS2 class | `ai_police_nastenka` (tag `18ai_police_nastenka`), 31 methods; `ai_animitem_manager` (tag `19ai_animitem_manager`), 2 methods; all mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

The board is the global singleton at VA **`0x006552a8`**, which is also **`ai_manager + 0x90`**
(`structs/ai_manager.md`). A byte scan of `Game.exe` finds 89 `MOV ECX, 0x006552a8` loads
(`B9 A8 52 65 00`) and no reference through any other addressing mode except the absolute float
alias of `+0x28` (§1c); 31 distinct methods are called through that pattern, and one more load
feeds the tail jump `jmp police_board_end_pursuit` at `0x00447788` inside
`police_board_invalidate_actor`.

```c
typedef struct police_board {                         // 0x34 bytes, at 0x006552a8
    /* +0x00 */ ai_animitem_manager anim_items;       // [H] §1a; always empty
    /* +0x10 */ std::vector<police_action> squads;    // [H] {alloc, begin, end, cap}, stride 0x6c
    /* +0x20 */ uint32_t session_id_counter;          // [H] the last pursuit-session id issued
    /* +0x24 */ uint32_t arrest_settle_timer_ms;      // [H] post-arrest settling time
    /* +0x28 */ float    police_speed_factor;         // [H] also addressed as VA 0x006552d0 (§1c)
    /* +0x2c */ uint8_t  busted_cutscene_active;      // [H] a capture cutscene is playing
    /* +0x30 */ C_actor *taxi_passenger;              // [H] the taxi passenger entity; parked with the suspend / resume pair
} police_board;
```

* **`squads`.** `police_board_create_and_engage` hands `board+0x10` to
  `police_action_vector_insert_at` / `police_action_vector_count`, which read `+0x04..+0x0c` of the
  object; the allocator word at `+0x10` is never accessed. `police_board_dtor`'s tidy zeroes both
  vector triples (`+0x04/+0x08/+0x0c` and `+0x14/+0x18/+0x1c`).
* **`arrest_settle_timer_ms`** ¹ resets to `0xfffffff` (about 74.6 h, an effectively-never
  value), is decremented every frame by `police_board_tick` (floored at 0), and is clamped down to
  12000 by `police_board_clamp_arrest_settle_12s`, whose only callsite is the ARREST branch of the
  arrest / fine cutscene (`entity_action_tick` case `0x21` substate `0xb`); the FINE branch does not
  clamp. So the field becomes meaningful (12 s or less) only after an arrest, and it gates when the
  arrest end-screen menu appears (`police_board_arrest_settle_is_zero` in `police_manager_tick` and
  `program_process_command` case `0x33`). It does not touch `pursuit_record+0x50` or the wanted bar.
* **`busted_cutscene_active`** means "the player is being captured right now", not "a squad is
  chasing": `police_board_set_busted_cutscene_active` raises it only in `entity_action_tick` case
  `0x21`, right after `police_board_disband_by_target`, on both the arrest and the fine path, and
  drops it when that cutscene ends (its only two callsites). `police_manager_tick` reads it as the
  gate that suppresses its live per-suspect block (clearing the player's record in the else) while
  the cutscene owns the player; `POLICEITCHFORPLAYER` (case `0x12c`) also reads it. Distinct from
  the per-squad active bit `police_action+0x50 & 1`.

¹ The session counter: `police_board_increment_counter` adds 1 and returns the new value in EAX
(Ghidra declares it `void`); the value becomes the new squad's `+0x30` and, through
`entity_spawn_police_response`, every squad officer's `+0xf54`.

### 1a. `ai_animitem_manager` — the embedded item manager, 0x10 bytes

```c
typedef struct ai_animitem_manager {          // 0x10 bytes, at police_board+0x00
    /* +0x00 */ std::vector<ai_anim_item> items;  // [H] {alloc, begin, end, cap}, stride 0x10
} ai_animitem_manager;
```

Both methods are called on the board's `this` unmodified, so the member sits at `+0x00`.
**Nothing ever adds an entry: the vector is always empty.** The board is reached only through the
89 immediate loads, `Game.exe` holds no absolute reference to `0x006552ac..0x006552b8`, and across
the 31 board methods, the two item methods and `police_board_dtor` the only writes to
`+0x04/+0x08/+0x0c` shrink the vector (`ai_animitem_manager_init`'s `end = begin`, the erase in
`ai_animitem_manager_clear`) or tidy it in the destructor. The member is vestigial.

### 1b. `ai_anim_item` — one item, 0x10 bytes

```c
typedef struct ai_anim_item {        // 0x10 bytes
    /* +0x00 */ uint32_t unread_00[3]; // [U] moved by the erase; no reader (class census)
    /* +0x0c */ void    *object;      // [M] a polymorphic object the clear releases
} ai_anim_item;
```

`unread_00` is three dwords because the erase in `ai_animitem_manager_clear` moves each later
element down as four dwords, of which only the fourth is ever dereferenced; what the three meant is
not recoverable, since no entry is ever created. `object` is released by three virtual calls
(slots `+0x2c`, `+0x24`, `+0x00` = the destructor, with 3 / 2 / 1 arguments) and nulled.

### 1c. `+0x28` is `g_police_speed_factor` at VA `0x006552d0`

One field, two access modes: singleton methods use `this`-relative addressing (`police_board_reset`
writes `MOV [ECX+0x28], 0x3f800000`, 1.0), while the script command `POLICE_SPEED_FACTOR` (case
`0x173`) and the chase consumer `pursuit_event_dispatch_pursuit_vehicle` (8 callsites, which
multiply a speed coefficient by the factor clamped to `[0.0, 1.0]`) use FPU absolute addressing — a
scan finds 11 references to `0x006552d0`, all FPU opcodes (`FMUL` ×6, `FST` ×3, `FSTP` ×2). That is
the compiler's usual shape for a field of a singleton at a known BSS address; there is no separate
global. Its use in a pursuit is in `police-system/artifacts/boosting.md` §2b.

## 2. Field evidence

Measured by a census of every `this+N` access in the `police_board_*` and `ai_animitem_manager_*`
functions, plus the absolute-address scan of §1c.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | anim_items | H | `ai_animitem_manager_init` (`end = begin`), `ai_animitem_manager_clear` (erase), `police_board_dtor` (zeroed) | `ai_animitem_manager_clear`, `police_board_dtor` | §1a |
| +0x10 | squads | H | `police_board_create_and_engage` (insert; the grow path writes the capacity), `police_board_end_pursuit` (`end -= 0x6c`), `police_board_reset` (`end = begin`), `police_board_dtor` | every board method that walks the squads; `police_action_vector_insert_at` (`[ecx+0xc]`, `ECX = board+0x10`) | stride `0x6c`, §1 |
| +0x20 | session_id_counter | H | `police_board_reset` (0), `police_board_increment_counter` (+1) | `police_board_increment_counter` | ¹ |
| +0x24 | arrest_settle_timer_ms | H | `police_board_reset` (`0xfffffff`), `police_board_tick` (−frame), `police_board_clamp_arrest_settle_12s` (12000) | `police_board_tick`, `police_board_arrest_settle_is_zero`, `police_board_clamp_arrest_settle_12s` | §1 |
| +0x28 | police_speed_factor | H | `police_board_reset` (1.0), the case `0x173` handler (absolute) | `pursuit_event_dispatch_pursuit_vehicle` (absolute) | §1c |
| +0x2c | busted_cutscene_active | H | `police_board_reset` (0), `police_board_set_busted_cutscene_active` | `police_board_is_busted_cutscene_active` | §1 |
| +0x30 | taxi_passenger | H | `police_board_reset` (0), `police_board_set_taxi_passenger`, `police_board_invalidate_actor` (0 when it dies) | `police_board_invalidate_actor`, `police_board_suspend_npcs`, `police_board_resume_npcs` | the setter's only caller is `pedestrian_wave_at_taxi`, with the taxi passenger `C_entity` it has just built (also stored at `C_game +0x204`); parked and un-parked with the other `C_entity` actors (§3c) |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | items | H | `ai_animitem_manager_init`, `ai_animitem_manager_clear`, `police_board_dtor` | `ai_animitem_manager_clear`, `police_board_dtor` | always empty (§1a) |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | unread_00 | U | `ai_animitem_manager_clear` (the erase moves it down) | - | no reader: no item is ever created (§1a, §1b) |
| +0x0c | object | M | `ai_animitem_manager_clear` (nulled) | `ai_animitem_manager_clear` (three virtual calls) | §1b |

## 3. Method family

Coverage: all **31** PS2 `ai_police_nastenka` methods and both `ai_animitem_manager` methods are
mapped — 32 CONFIRMED and 1 STRUCTURAL (`police_board_has_group_for_target`). One PC function has
no PS2 symbol, `police_board_dtor`, which the PS2 inlines into `__dt__10ai_manager`. The
genuinely per-squad methods are in `structs/police-action.md` §3.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00443860 | `police_board_dtor` | instance | - | - | - | walks the squads (stride `0x6c`), drains each roster, frees the squad buffer and the item vector. Its receiver is byte-proven by the unwind funclet `add ecx, 0x90 ; jmp <here>` and it has one entry point (no COMDAT fold); the normal path never calls it, because `ai_manager_dtor` inlines the same teardown |
| 0x00443bb0 | `crt_static_init_guard_00443bb0` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x00443bd0 | `crt_atexit_register_00443bd0` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004483c0 | `police_board_set_busted_cutscene_active` | `MakamSefe__18ai_police_nastenkaFb` | 0x00371290 | CONFIRMED | stores the byte `+0x2c` (PS2 `+0x24`); both callsites in `entity_action_tick` case `0x21` |
| 0x00448680 | `police_board_set_taxi_passenger` | `TuMasZakaznika__18ai_police_nastenkaFP8C_entity` | 0x00371690 | CONFIRMED | stores `+0x30` (PS2 `+0x28`); its only caller is `pedestrian_wave_at_taxi` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004469e0 | `police_board_reset` | `ZacatekSmeny__18ai_police_nastenkaFv` | 0x0036ee90 | CONFIRMED | scene start: `ai_animitem_manager_init`, empties the squads, writes the five header fields (§5) |
| 0x00446b00 | `police_board_increment_counter` | `JaChcuCislo__18ai_police_nastenkaFv` | 0x0036ef30 | CONFIRMED | `+0x20 += 1`, returns it; called with the board in ECX by `police_board_create_and_engage` |
| 0x00446b10 | `police_board_find_group_by_officer` | `CoVlastneRobimPlukovniku__18ai_police_nastenkaFP7C_actor` | 0x0036ef40 | CONFIRMED | the squad whose roster holds the officer; 10 callsites |
| 0x00446ba0 | `police_board_find_group_by_session_id` | `MamCisloDejAkci__18ai_police_nastenkaFUi` | 0x0036efe0 | CONFIRMED | the squad whose `+0x30` matches; 11 callsites, among them `entity_ai` and `entity_action_tick` |
| 0x00446c00 | `police_board_create_and_engage` | `MamProVasAkciHosi__18ai_police_nastenkaFP7C_actorUiRQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>bPv` | 0x0036f040 | CONFIRMED | builds a squad (mode → tier, §4 of `structs/police-action.md`), takes a session id, binds the roster, sends the officers into pursuit, fires the siren, inserts it; 5 callsites |
| 0x004474a0 | `police_board_end_pursuit` | `AkceDokoncenaPlukovniku__18ai_police_nastenkaFP16ai_police_action` | 0x0036ff80 | CONFIRMED | splices a squad out (per-element teardown, `end -= 0x6c`); 2 callsites and the tail jump of `police_board_invalidate_actor` |
| 0x00447610 | `police_board_invalidate_actor` | `ZinvalidujAktora__18ai_police_nastenkaFP7C_actor` | 0x00370160 | CONFIRMED | a tracked actor died: removes it from rosters, clears vehicles and `taxi_passenger`, disbands squads targeting it; 5 callsites |
| 0x004477b0 | `police_board_tick` | `Tick__18ai_police_nastenkaFUi` | 0x00370370 | CONFIRMED | decrements `arrest_settle_timer_ms`, runs `police_action_tick` for each squad |
| 0x00447820 | `police_board_arrest_settle_is_zero` | `PlukovnikuMasVChladkuPlajeraFrafera__18ai_police_nastenkaFv` | 0x00370420 | CONFIRMED | `+0x24 == 0` (PS2 `+0x1c == 0`); 2 callsites |
| 0x00447830 | `police_board_raise_gun_escalation` | `PlukovnikuRadsiHoSejmemNebudemSeSrat__18ai_police_nastenkaFP7C_actor` | 0x00370430 | CONFIRMED | the loud gun alarm: squads of the target go to tier 0 / 1 (committed police state 3); 5 callsites |
| 0x00447950 | `police_board_raise_handcuffs_escalation` | `PlukovnikuRadsiHoZatknemNebudemSeSrat__18ai_police_nastenkaFP7C_actor` | 0x00370590 | CONFIRMED | the loud handcuffs alarm: tier 4 / 5 (police state 2), refused from the gun tier; 4 callsites |
| 0x00447a90 | `police_board_disband_by_target` | `MamHoZmrdaUzHoNechte__18ai_police_nastenkaFP7C_actorP7C_actor` | 0x00370700 | CONFIRMED | disbands every squad targeting the actor |
| 0x00447b00 | `police_board_clamp_arrest_settle_12s` | `PlukovnikuMamZmrdaPlayera__18ai_police_nastenkaFv` | 0x003707a0 | CONFIRMED | caps `+0x24` at 12000 (PS2 `+0x1c`, the same constant); ARREST branch only |
| 0x00447b10 | `police_board_promote_handcuffs_escalation` | `PanoveZmenaZatkneteHoZmrda__18ai_police_nastenkaFP7C_actor` | 0x003707c0 | CONFIRMED | the silent commit half of the handcuffs raise (tier pair 4 / 5) |
| 0x00447bb0 | `police_board_promote_gun_escalation` | `PanoveZmenaZabijteHoZmrda__18ai_police_nastenkaFP7C_actor` | 0x003708c0 | CONFIRMED | the silent commit half of the gun raise (tier pair 0 / 1) |
| 0x00447c40 | `police_board_update_sighting` | `PanoveKokotPremistujePrdel__18ai_police_nastenkaFP7C_actorP7C_actorb` | 0x003709c0 | CONFIRMED | refreshes the sighting record of each matching squad (`structs/police-action.md` §1); 13 callsites |
| 0x00448150 | `police_board_target_matches` | `PanoveViditeZmrdaNeboSteTupci__18ai_police_nastenkaFP7C_actor` | 0x00370f20 | CONFIRMED | a squad covers the target in its current vehicle status (suppresses a duplicate) |
| 0x004481c0 | `police_board_refresh_target_vehicle` | `PanoveVytahneteBlokyAZapisteSiZmrda__18ai_police_nastenkaFP7C_actor` | 0x00370fb0 | CONFIRMED | re-syncs each matching squad's `+0x64` / `+0x68` to the target's vehicle |
| 0x00448230 | `police_board_has_recent_sighting` | `PanoveMateOciNaZmrdoviNeboCumitePoBabach__18ai_police_nastenkaFP7C_actor` | 0x00371030 | CONFIRMED | an officer of a squad on the target holds an entry for it in its own AI log (officer `+0xe88`) updated within 2000 ms; 4 callsites |
| 0x00448300 | `police_board_has_group_for_target` | `PanoveJdeteNekdoPoZmrdovi__18ai_police_nastenkaFP7C_actorb` | 0x00371160 | STRUCTURAL | any squad of any tier has the target; `police_manager_tick` picks its identity branch by it (`structs/police_manager.md` §8) |
| 0x00448350 | `police_board_broadcast_for_target` | `PanoveJaUzHoVarovalZmetka__18ai_police_nastenkaFP7C_actorP7C_actor` | 0x003711e0 | CONFIRMED | event `0x17` to every squad on the target (§6) |
| 0x004483b0 | `police_board_is_busted_cutscene_active` | `PanoveHrajeteSiNaPoldy__18ai_police_nastenkaFv` | 0x00371280 | CONFIRMED | `+0x2c != 0` (PS2 `+0x24`); one byte read, no squad walk |
| 0x004483d0 | `police_board_is_actor_in_pursuit` | `SemTrefenaLocklaKurva__18ai_police_nastenkaFP7C_actor` | 0x003712a0 | CONFIRMED | the actor (kind `0x1b`, or 2 = the player) is a squad's target (`+0x60`) or roster officer; 3 callsites |
| 0x00448470 | `police_board_suspend_npcs` | `VseckyUbozakyStopni__18ai_police_nastenkaFP7C_actorP7C_actor` | 0x00371360 | CONFIRMED | parks every `C_entity` of the mission and game actor lists by adding 10000 to its AI state `+0xf40` (`structs/entity.md`), skipping the two given actors and type group 2 (the police); also parks `taxi_passenger`, hides the ambient traffic and sets `g_ambient_ai_suspended`. Its one caller is `entity_action_tick` case `0x21`, when it captures the player |
| 0x00448550 | `police_board_resume_npcs` | `VseckyUbozakyRozejdi__18ai_police_nastenkaFP7C_actorP7C_actor` | 0x003714d0 | CONFIRMED | the inverse (subtracts 10000, shows the traffic, clears the flag) when the capture cutscene ends; its first loop does not skip type group 2 and its second does not skip the two given actors |
| 0x00448620 | `police_board_has_active_group_for_target` | `HosiPrkynkujete__18ai_police_nastenkaFP7C_actor` | 0x00371620 | CONFIRMED | a squad has the target (`+0x60`, PS2 `+0x64`) and the active bit (`+0x50 & 1`, PS2 `+0x54`); 2 callsites |
| 0x00448690 | `police_board_end_all_pursuits` | `KonecSmeny__18ai_police_nastenkaFv` | 0x003716a0 | CONFIRMED | scene end: ends every squad in reverse order, then the item clear (§5) |
| 0x004486f0 | `police_board_release_all_group_vehicles` | `ZastavMotorizovaneJednotky__18ai_police_nastenkaFv` | 0x00371720 | CONFIRMED | deactivates each squad's car (`+0x5c`) and detaches the officers; called only by `program_process_command` case `0x183` = `M7_ZASTAVZKURVENEPOLISE` |
| 0x004487f0 | `thunk_ai_animitem_manager_clear` | `SrackyPryc__18ai_police_nastenkaFv` | 0x00371880 | CONFIRMED | the board's item clear: a jump to `ai_animitem_manager_clear`, which the PS2 inlines statement for statement (reverse walk, three virtual calls, erase, clear); 2 callsites on each build |
| 0x00448800 | `ai_animitem_manager_clear` | `Clear__19ai_animitem_managerFv` | 0x00371890 | CONFIRMED | receiver `ai_animitem_manager`: reverse-walks the items, releases each `object` (§1b), erases; called by `police_board_end_all_pursuits` and through the thunk |
| 0x00448900 | `ai_animitem_manager_init` | `Init__19ai_animitem_managerFv` | 0x00371960 | CONFIRMED | receiver `ai_animitem_manager`: `end = begin`, no teardown; called by `police_board_reset` |

### 3d. Free helpers

The unit's free functions no class of it calls stay here (the unit's host):

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00448e00 | `crt_operator_delete_thunk_00448e00` | - | - | - | a jump thunk to the CRT `operator delete` |
| 0x00448fc0 | `ascii_toupper` | - | - | - | upper-cases one ASCII character; a string helper placed in this unit |

## 4. The translation unit

`[0x00443bb0, 0x00449ba0)`, from its own CRT guard to the next one. It holds the unit's CRT pair, the
seven per-squad methods of `police_action`, the 31 board methods, the two `ai_animitem_manager`
methods, the eight `std::vector<police_action>` functions (`structs/police-action.md` §7), and
COMDAT-placed functions of other families: `i3d_cache_model_open`, `i3d_cache_items_uninit_copy_range`,
`i3d_cache_items_copy_construct_one`, `i3d_cache_items_copy_assign_one` (`structs/i3d_cache.md`),
the string helpers `string_base_init`, `string_base_release`, `string_dtor`, `string_get_data`,
`string_concat_cstr`, `string_format` (`structs/string.md`), and the two free helpers of §3d. The
board's destructor and `police_action_destroy_range` sit in the preceding `ai_manager` unit
`[0x00443510, 0x00443bb0)`.

`vec_owner_scalar_deleting_dtor` (`0x00443b90`, in the `ai_manager` unit) belongs to neither police
class: its body is `vector_dispose_at_0x04(); if (flag & 1) delete this;`, and its two callsites are
the stride-`0x48` (`ai_fight_followship`) and stride-`0x38` (`ai_follow_group`) loops inside
`ai_manager_dtor` — two element types, one COMDAT-folded body.

## 5. Scene start and scene end

`police_board_reset` runs at scene start and writes the five header fields in this order: `+0x20` =
0, `+0x24` = `0xfffffff`, `+0x2c` = 0 (a byte among dwords), `+0x28` = 1.0, `+0x30` = 0; the squad
vector is only reset to `end = begin`, keeping its capacity. `police_board_end_all_pursuits` is the
mirror at scene end: `game_done` calls it, it iterates the squads **in reverse**, calls
`police_board_end_pursuit` on each, and finishes with the item clear.

## 6. What event `0x17` does

`police_board_broadcast_for_target` hands each matching squad to `police_action_broadcast_event(squad,
sender, 0x17, target)`, which calls `entity_dispatch_event(sender, 0x17, target)` on every roster
officer except the sender. That handler's `case 0x17` does one thing: if the receiving officer's
base job is an `ai_action` of type `0x21` whose payload `+0x58` is the same target, and the sender is
another entity, it pushes the action's `+0x50` deadline out to `mission_clock + 1000` ms. So `0x17`
is a **deconfliction signal** — "somebody else is working this target, hold yours for a second" —
with no audio, no state change and no escalation. Its one producer is `entity_action_tick` case
`0x21`.

## 7. Three structs that are not this one

| Offsets | Where they live |
|---|---|
| `+0x4/+0x8`, `+0x14/+0x18`, `+0x24/+0x28` task lists | the per-NPC task subsystem at **`NPC + 0x1164`** ¹ |
| `+0x40/+0x44/+0x48` per-target tracking array | `police_manager` (`structs/police_manager.md` §1) |
| `+0x10b8`/`+0x10bc` (stride `0xc`) and `+0x10c4` | `C_entity` ² |

¹ Every call to `ai_action_manager_clear_all` / `ai_action_manager_invalidate_actor` /
`ai_action_manager_retire_top_modifier` is preceded by `LEA ECX, [reg + 0x1164]` (the only ECX
set-up across 23 callers), and the `entity_reset` docblock confirms the base register holds the NPC.
The three lists are stride 4 (task pointers), unrelated to the board's stride-`0x6c` squads and its
timer at `+0x24`.

² `entity+0x10b8`/`+0x10bc` is `C_entity`'s substate-lock table, `std::vector<ai_substate_lock>`,
stride `0xc`, rows `{key, p1 (uint), p2 (float)}`, and `entity+0x10c4` is `locked_state`, where
`0x29a` (666, the project-wide "none" value) means no lock. The accessors are
`entity_unlock_substate` (`0x00434550`), `entity_lock_substate`, `entity_has_locked_substate`,
`entity_has_locked_state` and the persistence pair `entity_save_state` / `entity_load_state`
(`structs/entity.md` §1 / §2). The attribution is byte-level: `entity_unlock_substate` operates on
`[param_1 + 0x10b8]`/`[+0x10bc]`, all 5 of its callers pass an officer as `this` (3 in
`entity_weapon_choose`, `0x00432550`, whose `__fastcall (int officer)` signature fixes the
receiver; 2 in `ai_fight_manager_disband_followship`, `0x0043f730`, with ESI from an officer-typed
`this`), and a scan finds no `MOV ECX, 0x006552a8` followed by a call to it within 32 bytes.
`entity_save_state` / `entity_load_state` (`0x00435600` / `0x004358f0`) serialise the table on the
same `this` as the officer fields `+0xf40` (state), `+0xf44` (substate), `+0xf08` (flags), `+0xf4c`
(role), `+0xf50`, `+0xf58`, `+0xf5c`, and `entity_get_state_size` adds `nlocks * 0xc` to the
savegame "State" block. Among its keys, key 3 is the pursuit-approach sub-state that
`ai_fight_manager_tick` publishes each tick.

## 8. PC vs PS2

**Class names.** The board is `ai_police_nastenka` (the Czech word means "notice board"); the squad is
`ai_police_action`; the item manager is `ai_animitem_manager`, typed by the container lever
`erase__…FP12ai_anim_item`. Many method names are developer jokes; the PC names state behaviour
(`SemTrefenaLocklaKurva`, roughly "hit-and-locked", tests squad membership; `VseckyUbozakyStopni` /
`VseckyUbozakyRozejdi`, roughly "stop all the poor sods" / "send them on their way", park and
un-park the NPCs; `TuMasZakaznika`, "here is your customer", stores the taxi passenger).

**The `+0x08` rebase.** `police_board_reset`'s five-field block matches `ZacatekSmeny` in order and
constants at a uniform `+0x08`:

| PS2 | PC | Value |
|---|---|---|
| +0x18 | +0x20 | `0` |
| +0x1c | +0x24 | `0xfffffff`, a distinctive 28-bit mask |
| +0x24 (byte) | +0x2c (byte) | `0` |
| +0x20 | +0x28 | `0x3f800000` (1.0) |
| +0x28 | +0x30 | `0` |

The squad vector rebases the same way (PS2 `+0xc` / `+0x10` ↔ PC `+0x14` / `+0x18`) with each build's
squad stride (PC `0x6c`, PS2 `0x80`). The `+0x08` decomposes exactly: `ai_animitem_manager` grows
`0xc` → `0x10` because an MSVC `std::vector` carries an allocator word, and the board's own vector
gains the same word. The seven accessors twinned in §3 (`arrest_settle_is_zero`,
`clamp_arrest_settle_12s`, the busted-cutscene pair, `set_taxi_passenger`,
`has_active_group_for_target`, the item-clear thunk) read and write exactly these rebased fields.

**Matched bodies.** The statement-for-statement twins of the squad walks use each build's stride,
with the squad fields one dword apart (PC target `+0x60` and number `+0x30`, PS2 `+0x64` and
`+0x34`) and the same callees (`HosiKoncime` = `police_action_disband`, `EventVsem` =
`police_action_broadcast_event`, `GetLog` = `ai_logs_get_log_for_actor`). The sighting test looks
the target up in each officer's own log (PC officer `+0xe88`, PS2 `+0x1060`); its window is 2000 ms
on the PC and **3000 ms on the PS2**. The callers of the number lookup correspond too: `C_entity::AI`
and `ai_modifier::Tick` on the PS2, `entity_ai` and `entity_action_tick` on the PC. The suspend /
resume pair matches loop for loop (the mission actor list, then `C_game`'s temporary actors), with
the same exclusions and the same asymmetry in the resume half, `salina_v_kine` ↔
`g_ambient_ai_suspended`, and the taxi passenger at PS2 `+0x28` ↔ PC `+0x30`; the PS2 adds and subtracts
10000 through `NeserADejchej` / `NeserANedejchej` where the PC writes `+0xf40` directly.
`police_board_end_all_pursuits` ↔ `KonecSmeny`: PS2 `Done__6C_gameFv` calls it where PC `game_done`
calls this function.

**The board's offset inside `ai_manager` is the same on both builds:** PS2 `Init__10ai_managerFv`
opens `reserve__…(param_1 + 0x90, 100)`, so the board is sized for 100 squads.

**Build differences.**
- `ZacatekSmeny` frees the squad buffer (`__dl__`) and zeroes begin / end / cap; `police_board_reset`
  only resets `end = begin`.
- The anim-item virtual slots differ (PS2 `0x40` / `0x38` / `0x8` against PC `0x2c` / `0x24` /
  `0x0`): vtables do not slot-align across builds (`structs/actor.md`); order and arity match.
- `police_board_dtor` has no PS2 symbol: it is inlined into `__dt__10ai_managerFv`.
- The anim-item vector is empty on both builds: the PS2 instantiates only the constructor,
  destructor, deleter, `clear` and `erase` of `vector<ai_anim_item>`, no insert and no reserve.

## 9. Related

- `structs/police-action.md` — the squads: field evidence, the tier machine, the Cheat Engine
  layout and the vector instantiation.
- `structs/police_manager.md` — the sub-object at `C_game+0x2da0`; a distinct struct.
- `structs/ai_manager.md` — the root object this board is a member of.
- `structs/entity.md` — the officers, their AI state `+0xf40` and the substate-lock table.
- `police-system/artifacts/police-system.md` — the squad life cycle.
- `Game-ghidra.exe.functions.md` — the function registry.

## 10. Open items

1. **Runtime validation.** None of the layout has been confirmed in game.
