# `police_manager` — reconstruction notes

**Created** 2026-07-27 22:03 +0200 · **Modified** 2026-09-18 15:31 +0200

The companion to [`police_manager.md`](police_manager.md), which is the spec: it holds the
struct layouts, the per-field evidence, the tick scheduler and the PC↔PS2 twin table. This
file holds what does not belong in a layout spec — the naming conventions that keep the
police vocabulary apart, the behaviour of the reserve-pool deploy helpers, the signatures
that the decompiler cannot show, and the two readings that a naive pass of the decomp
produces and that the bytes refute.

## 1. Naming conventions

The police subsystem has three vocabularies that are easy to collapse into one another.
They are kept apart deliberately.

| concept | name | why |
|---|---|---|
| the struct — a block of police data embedded in the `C_game` object | `police_manager` | names the memory layout; carries no claim about how it is reached |
| the function family that operates on it | `police_manager_*` (with the underscore) | matches the `s_POLICEMANAGER_*` strings in `.rdata` and the `POLICEMANAGER_*` script-command identifiers; describes the behaviour |
| the script-command identifiers `policemanager_on`, `policemanager_setspeed` | unchanged | baked into shipped game data, so not ours to rename |
| the officer-squad object and its family | `police_action` / `police_action_*`, `police_board` / `police_board_*` | a **separate** runtime object (`structs/police-action.md`, `structs/police-board.md`); its helpers never receive a `police_manager` |
| engine-wide helpers whose shape is generic but whose data is police-specific | `police_manager_npc_list_push` / `_erase` / `_tick` | the code shape is an ordinary vector op over a `+0x80`/`+0x84`/`+0x88` triple, but every located caller passes the player's `police_manager`, so the prefix states the data, not the shape |
| engine-wide helpers whose data is not police-specific either | neutral, e.g. `std_vector8_insert_at` | receiver discipline is byte-recovered rather than encoded in the signature; the same address serves unrelated owners, so no class-qualified name is allowed |

## 2. The reserve-pool deploy helpers

`police_manager_apply_state` populates the three reserve pools; six helpers consume them.
Each bucket has a pool-pop helper and a deploy wrapper, and the wrappers differ from one
another in ways the pool table in the spec does not carry.

| bucket | pool-pop helper | deploy wrapper | selection | wrapper's own work |
|---|---|---|---|---|
| police_vehicles (`+0x50`) | `police_manager_vehicle_pool_pop_random` | `police_manager_deploy_vehicle_from_pool` | uniform random | cycles a 3-step vehicle-controller state machine on `entity+0x20c`, then a radar-add tail |
| spike_strips (`+0x60`) | `police_manager_spike_strip_pool_pop` | `police_manager_deploy_temp_spike_strip` | LIFO — first claimed walking backward | a downward `collision_test_line_vertical_thunk` navmesh snap on the spawn pose |
| foot_cops (`+0x70`) | `police_manager_officer_pool_pop_random` | `police_manager_deploy_npc_from_pool` | uniform random | equips ONE random weapon (below), then `inventory_select_by_id(10, 0)` and `human_change_weapon_model` |

All three wrappers share the same placement sequence: write the position into the frame's
`+0x80`/`+0x84`/`+0x88` translation slots, set `frame+0xac = (old & 0xfffffedf) | 0x40000000`
(the world-pose-dirty bits), call the frame's `vt[+0xc]` orientation setter and `vt[+0x18]`
finalise, then register through `game_add_temporary_actor_intern`. Each returns the deployed
entity, or 0 when its pool is empty.

### 2a. The two foot-cop loadouts

`police_manager_deploy_npc_from_pool` flips a coin through `rand_range` and equips exactly
one weapon:

| roll | weapon id | byte-4 | reading |
|---|---|---|---|
| `rand_range() == 0` | `0x0b` | 8 | pump-action shotgun, 8 shells |
| `rand_range() != 0` | `0x0a` | `0x32` (50) | Thompson 1928 SMG, the 50-round drum-magazine variant |

Both ids are resolved against `tables/predmety.def` (`def-files-docs/predmety.md`), and the
50-round capacity corroborates the Thompson identification independently of the id.

The **pursuit-vehicle-with-occupants** workflow is a different composite and does **not** go
through that wrapper. It materialises a chase patrol car at the player out of:

1. one `police_manager_deploy_vehicle_from_pool` call (the vehicle host);
2. two raw `police_manager_officer_pool_pop_random` pops — bypassing the deploy wrapper
   precisely because this workflow wants a two-weapon loadout;
3. four `inventory_insert_item` calls — a pump shotgun and an S&W M&P revolver per NPC.

So a foot cop deployed on his own carries one weapon and a foot cop riding a patrol car
carries two, and the two paths do not share a code path.

## 3. Signatures the decompiler cannot show

* **`police_manager_deploy_vehicle_from_pool`** is rendered as `int (void)`, because its
  arguments arrive in the caller's preserved registers and stack and surface in the body as
  `unaff_ESI` / `unaff_retaddr`. The signature reconstructed from its three callsites is
  `int __thiscall police_manager_deploy_vehicle_from_pool(police_manager *this, vec3 *pos,
  vec3 *dir, char flag)`. All three callsites pass `(&pos, &dir, 0_or_1)`.
* **Receiver discipline is byte-recovered, not decomp-visible,** at the `police_manager`
  callsites that Ghidra prints with no arguments at all — the script-dispatcher `npc_list`
  cases and the `game_init` call of `police_manager_game_init` among them. The receiver is
  read out of the disassembly at the callsite, where the chain is
  `act_mission_get_game` → `game_get_police_block` → the call.

## 4. Two readings the bytes refute

* **Nothing in `police_manager_dtor` destroys elements.** The destructor walks all seven
  `std::vector` triples in reverse offset order — `+0xa0`, `+0x90`, `+0x80`, `+0x70`,
  `+0x60`, `+0x50`, `+0x40` — and for each does `operator_delete_wrapper(begin)` plus a
  zeroing of begin/end/cap. The records vector at `+0x40` is preceded by a call to
  `stub_noop_args8_00575c00(begin, end)`, and **that function is an empty `ret 8`** — no
  `operator delete`, no write, no read, no instruction at all besides the return. It is
  16-aligned and NOP-padded, with 511 direct `E8` callers and 21 vtable slots, verified
  against the `Game.exe` bytes rather than against decompiler output. So the records vector
  is not special: every one of the seven is freed as a bare array. Any conclusion that
  `police_manager` owns its `pursuit_record` elements rests on that call and does not
  survive it. The same no-op sits in `police_manager_game_init`, where it equally does not
  drain the records vector — the `end = begin` write does that alone, and `cap` at `+0x48`
  stays valid across a soft reset because nothing is freed, not because a buffer is recycled.
* **`+0x6c` is not a field.** `police_manager_apply_state` computes `local_80 = this + 0x6c`,
  which is the compiler picking a convenient base for the foot_cops triple at
  `+0x70`/`+0x74`/`+0x78` — each vector helper sees its triple at `receiver+0x04/+0x08/+0x0c`.
  The same shape appears at `+0x4c` for police_vehicles and `+0x5c` for spike_strips.

## 5. Runtime-validation checklist

Static reconstruction is complete; the items below are what an attached ReClass.NET or
Cheat Engine session would settle. The Cheat Engine pointer paths are in
`police_manager.md` §5.

1. Resolve the `C_game` object, step `+0x2DA0`, and watch `+0x00 enabled` toggle across
   police-state transitions — that confirms the struct anchor.
2. Walk the records vector at `+0x40`, take one `pursuit_record *` element and lay out the
   `0x80`-byte struct against `police_manager.md` §3.
3. Trigger a pursuit and watch the five `(accumulator, period_ms)` pairs at `+0x04` .. `+0x30`.
   The accumulator halves should climb monotonically between overflows; the two pairs with no
   located reader should stay at their reseeded values.
4. Trigger a pursuit and observe which of the pools populate; then run a soft reset and
   confirm that only foot_cops and the two renderer arrays survive it.
5. Break on `police_manager_npc_list_push` with the player's manager as receiver and dump the
   NPC handles being pushed, to settle which category of NPC the list holds.
6. Break on a pool-pop helper and read the second 4 bytes of the stride-8 element, which the
   pop helpers treat as a claim flag but which the populate pass writes specific non-zero
   values into.

## 6. Cross-references

* [`police_manager.md`](police_manager.md) — the spec: `police_manager` and `pursuit_record`
  layouts, field evidence, the per-frame tick scheduler, the PC↔PS2 twin table, open items.
* `structs/police-action.md` and `structs/police-board.md` — the officer-squad system.
* `police-system/artifacts/police-system.md` — runtime behaviour and the call chain.
* `police-system/artifacts/police-manager.md` — the `police_manager_tick` phases and the
  noticing pipeline.
* `police-system/artifacts/offences.md` — the offence bit map and escalation.
* `script-mapping/artifacts/opcodes-verified.md` — the police script commands.
* `def-files-docs/predmety.md` — the weapon-id table behind §2a.
* `ghidra/Game-ghidra.exe.functions.md` — the function registry.

## 7. Open items

| item | state |
|---|---|
| the `flag` argument of `police_manager_deploy_vehicle_from_pool` | 0 or 1; the two pursuit-workflow callsites pass 1 and the standalone deploy passes 0. Reads as "is this spawn bound to a pursuit", unverified |
| the second half of each stride-8 pool element | the pop helpers treat it as a claim flag, but the populate pass writes specific non-zero values; needs a runtime breakpoint (§5.6) |
| the NPC category on `npc_list` | officers of the tracked records, script-added slots, or a mixed set — static evidence admits several readings (§5.5) |
| `pursuit_record + 0x28` | a reader exists (`police_manager_get_num_of_offences`) and the only located writer is the initialising one in the register-actor path; no mutation writer found |
| the script-command names for dispatcher cases `0x16b` / `0x171` | the add/remove pair on `npc_list` is byte-decoded, but neither case body carries a string literal, so the canonical command names are unresolved |
| the renderer-array consumer | `police_manager_apply_state` and the middle tick populate `rndr_arr_a` (`+0x90`) and `rndr_arr_b` (`+0xA0`); the consumer that draws from them is not located |
