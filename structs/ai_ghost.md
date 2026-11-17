# `ai_ghost` — the per-entity collision exemptions ("ghosts")

**Created** 2026-08-02 11:32 +0200 · **Modified** 2026-10-30 19:30 +0100

`ai_ghost` is **how an NPC walks through an object that stands on its detour.** While a route
follows a spec (detour) waypoint, `ai_path_tick` records a "ghost" for the blocker that waypoint
was built around — a car, a door or another entity: the object's world position at that moment,
plus an expiry stamp and a drift tolerance. Every live ghost's collider goes into the entity's
collision ignore list, which `human_movement_intern` and `human_set_next_coll_owners` hand to
`g_collision`, so the entity's movement collision skips that object. The record lives from half
a second to two seconds and dies early if the real object moves away from where it was seen. It
is a short-term collision exemption, not a perception system — `ai_sensors` does perception.

```
  C_entity  +0x1268
      |  embeds
      v
  ai_ghost                 vector<ai_ghost_actor> + a changed flag   section 1
      |  holds a vector of
      v
  ai_ghost_actor           pos + expiry + radius² + actor + collider section 1a
```

| | |
|---|---|
| PC prefix | `ai_ghost_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x14` (`C_entity+0x1268..+0x127c`) |
| PC constructor | - (`entity_ctor` builds the vector head in place) |
| PC translation unit | `[0x0040f700, 0x0043b000)`, the `C_entity` unit (`structs/entity.md` §8); this class is `[0x0043a430, 0x0043a850)` inside it |
| PS2 class | `ai_ghost` (tag `8ai_ghost`), 5 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Offsets from the object base, `C_entity+0x1268`. The vector head is an MSVC `std::vector`:
an allocator byte, then begin / end / capacity.

```c
typedef struct ai_ghost {                     // 0x14 bytes, at C_entity+0x1268
    /* +0x00 */ uint8_t records_alloc;        // [H] allocator byte of the record vector
    /* +0x04 */ ai_ghost_actor *records_begin; // [H] first record, C_entity+0x126c
    /* +0x08 */ ai_ghost_actor *records_end;  // [H] one past the last record
    /* +0x0c */ ai_ghost_actor *records_cap;  // [H] end of the vector's storage
    /* +0x10 */ uint8_t changed_flag;         // [H] the ignore list must be rebuilt (§4a)
} ai_ghost;
```

### 1a. `ai_ghost_actor` — one ghost, `0x1c` bytes

```c
typedef struct ai_ghost_actor {               // 0x1c bytes, the vector's stride
    /* +0x00 */ float    pos[3];              // [H] the object's world position when remembered
    /* +0x0c */ uint32_t expiry_ms;           // [H] absolute mission clock of the record's death
    /* +0x10 */ float    drift_radius_sq;     // [H] squared distance the object may move from pos
    /* +0x14 */ C_actor *actor;               // [H] the actor the object belongs to
    /* +0x18 */ tDynamicCollObject *coll_object; // [H] the collider exempted
} ai_ghost_actor;
```

**Why `pos` has three entries.** x, y, z of `frame+0x40..+0x48`. There is no other array in either
record.

## 2. Field evidence

Measured by a register-tag census of every `this + N` access in the class's functions, a displacement
census of `C_entity+0x1268..+0x1278` over `.text`, and reading every body. That census finds only the
class's methods, `entity_tick_ghosts`, `entity_ctor` and `entity_dtor`, plus the `lea` receivers in the
two producers and in `entity_action_tick`, `entity_game_init`, `entity_hit`, `entity_release_target` and
`entity_reset`, so no other consumer exists. The record is reached through a walking pointer, so its
cells come from the bodies.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | records_alloc | H | `entity_ctor` | - | the allocator byte: `mov [esi+0x1268],dl` at `0x004115d5`; no code reads it |
| +0x4 | records_begin | H | `ai_ghost_add_or_refresh`, `entity_ctor`, `entity_dtor` | `ai_ghost_add_or_refresh`, `ai_ghost_clear_all`, `ai_ghost_clear_all_and_mark_changed`, `ai_ghost_invalidate_actor`, `entity_dtor`, `entity_tick_ghosts` | `C_entity+0x126c` |
| +0x8 | records_end | H | `ai_ghost_add_or_refresh`, `ai_ghost_clear_all`, `ai_ghost_clear_all_and_mark_changed`, `ai_ghost_invalidate_actor`, `entity_ctor`, `entity_dtor`, `entity_tick_ghosts` | `ai_ghost_add_or_refresh`, `ai_ghost_clear_all`, `ai_ghost_clear_all_and_mark_changed`, `ai_ghost_invalidate_actor`, `entity_tick_ghosts` | count = (end - begin) / 0x1c |
| +0xc | records_cap | H | `ai_ghost_add_or_refresh`, `entity_ctor`, `entity_dtor` | `ai_ghost_add_or_refresh` | - |
| +0x10 | changed_flag | H | `ai_ghost_add_or_refresh`, `ai_ghost_clear_all`, `ai_ghost_clear_all_and_mark_changed`, `entity_tick_ghosts` | `entity_tick_ghosts` | set on an append, on the marked clear and on a tick that drops a record, not on a refresh and not by `ai_ghost_invalidate_actor`; `entity_ctor` leaves it unset, `entity_game_init` zeroes it through `ai_ghost_clear_all` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | pos | H | `ai_ghost_add_or_refresh` | `entity_tick_ghosts` | a snapshot of `frame+0x40..+0x48`, never re-read from the frame; read by exactly one statement, the drift test |
| +0xc | expiry_ms | H | `ai_ghost_add_or_refresh` | `entity_tick_ghosts` | the only field a refresh rewrites |
| +0x10 | drift_radius_sq | H | `ai_ghost_add_or_refresh` | `entity_tick_ghosts` | compared with the squared distance from `pos` |
| +0x14 | actor | H | `ai_ghost_add_or_refresh` | `ai_ghost_add_or_refresh`, `ai_ghost_invalidate_actor`, `entity_tick_ghosts` | half the dedup key, the invalidate key; its `+0x68` is the frame |
| +0x18 | coll_object | H | `ai_ghost_add_or_refresh` | `ai_ghost_add_or_refresh`, `entity_tick_ghosts` | the other half of the key, and the value copied into the ignore list |

## 3. Method family

5 of 5 PS2 methods mapped, all with a PC function; the one `gif::vector` helper is inlined. The per-frame
driver `entity_tick_ghosts` is a `C_entity` method and is listed in `structs/entity.md`. The unit's CRT
pair and its uncalled helpers stay with the host doc.

### 3a. Construction & destruction

None: `entity_ctor` and `entity_dtor` build and free the vector head in place.

### 3b. Accessors

None: every function does more than one load or store.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043a430 | `ai_ghost_add_or_refresh` | `Add__8ai_ghostFP7C_actorP18tDynamicCollObjectUif` | 0x00349970 | CONFIRMED | dedup-scans on `(actor, coll_object)`; a hit refreshes only `expiry_ms` and returns; otherwise forces the actor's world matrix current (`frame_flags & 0x20`), snapshots its position, appends and sets `changed_flag` |
| - | - | `reserve__Q23gif26vector<14ai_ghost_actor,1>Fi` | 0x00349b40 | inlined | a container helper with no PC function: its growth is open-coded in `ai_ghost_add_or_refresh` |
| 0x0043a6e0 | `ai_ghost_add_human_colliders` | `AddHuman__8ai_ghostFP7C_humanUif` | 0x00349c70 | CONFIRMED | one ghost per element of the human's stride-4 collider array (count `human+0x690`, data `human+0x688`), because a person is several collidable parts |
| 0x0043a730 | `ai_ghost_clear_all` | `Init__8ai_ghostFv` | 0x00349d10 | CONFIRMED | `end = begin`, `changed_flag = 0`; callers `entity_game_init`, `entity_reset` |
| 0x0043a770 | `ai_ghost_clear_all_and_mark_changed` | `Reset__8ai_ghostFv` | 0x00349d80 | CONFIRMED | the same with `changed_flag = 1` (§3e); callers `entity_hit`, `entity_action_tick` |
| 0x0043a7b0 | `ai_ghost_invalidate_actor` | `Invalidate__8ai_ghostFP7C_actorP18tDynamicCollObject` | 0x00349e00 | CONFIRMED | reverse walk, erases every record whose `actor` matches; does not set `changed_flag` (§4c); one callsite, `entity_release_target` |

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0043afe0 | `stl_construct_stride_0x1c` | - | - | - | copies one `0x1c` record while `ai_ghost_add_or_refresh` regrows the vector |

### 3e. The identical-sibling resolution

`ai_ghost_clear_all` and `ai_ghost_clear_all_and_mark_changed` have byte-identical bodies apart from one
constant, so neither size, arity nor callee set separates them. §5 records the two cross-build
discriminators that assign the twins.

## 4. Who writes ghosts, and who reads them

**Producers — the path follower.** `ai_path_tick` (while the route is in state 2, following a spec
waypoint) and `ai_path_revalidate_for_actor` ghost the **blocker of the current spec waypoint** — the
actor at waypoint `+0x10`, with its collider at `+0x14` (`structs/ai_path.md` §1b) — dispatching on the
actor's kind (`actor+0x10`):

| actor kind | call | expiry | drift radius² |
|---|---|---|---|
| 4 (`C_car`) | `ai_ghost_add_or_refresh` | clock + 500 ms | 0.09 (0.3²) |
| 6 (`C_door`) | `ai_ghost_add_or_refresh` | clock + 800 ms | 0.25 (0.5²) |
| `0x1b` (`C_entity`) | `ai_ghost_add_human_colliders` | clock + 2000 ms | 1.0 (1.0²) |
| any, in `ai_path_revalidate_for_actor` | either, per kind | clock + 1000 ms | 0.09 / 1.0 |

A car is exempted briefly and loses its exemption the moment it rolls 0.3 units; a person keeps it four
times longer and may move a whole unit first. The spec-node searches also pick up a closed door on the
route, which `ai_path_update_spec_waypoints` then opens for a `C_entity` walker (`structs/ai_path.md` §1,
`+0x68`).

**The per-frame tick.** `entity_tick_ghosts` (`0x0043a220`) runs once per frame from `entity_ai` and
drops a record when **either** the clock passes `expiry_ms` **or** the real object has moved further than
`drift_radius_sq` from `pos`. A drop sets `changed_flag`. It then acts on the flag (§4a).

### 4a. `C_entity+0x698` — the collision ignore list

When `changed_flag` is set, `entity_tick_ghosts` clears it and then:

```
begin = *(this + 0x69c);                    // stride-4 std::vector at C_entity+0x698
release_range(begin, *(this + 0x6a0));      // drop every element
*(this + 0x6a0) = begin;                    // end = begin: the list is now empty
for (i = ghost_count - 1; i >= 0; i--)      // rebuild, walking the ghost vector backwards
    std_vector4_insert_n(*(this + 0x6a0), 1, ghost_record(i) + 0x18);
```

The list is `+0x698` the allocator byte (`human_ctor` stores it), `+0x69c` begin, `+0x6a0` end, and the
pushed value is `coll_object`, one entry per live ghost. It is rebuilt only on frames where the flag is
set; that is what the flag exists for.

**Its two readers.** `human_set_next_coll_owners` and `human_movement_intern` both concatenate the human's
own colliders (`+0x688`, count `+0x690`) and this list into the scratch vector at `+0x6a8` (begin
`+0x6ac`) and pass it to `collision_dyn_set_next_multiple_owners`. Every `g_collision` query then walks
that owner array and skips any object found in it, so the next collision test ignores both the human's
own body and every ghosted object. A displacement census of `+0x698..+0x6a4` finds no other reader: the
list is written by `human_ctor`, `human_dtor`, `entity_game_init` / `entity_reset` (cleared) and
`entity_tick_ghosts`, and read by those two.

`+0x700` is not part of this: it is a `C_human` field, occurring about 25 times in the `C_human` band,
always paired with `+0x6f8` and `+0x704` as movement / anim gates, and `entity_tick_ghosts` never touches it.

### 4b. Ghosts and blocked connections are separate mechanisms

`entity_add_blocked_con` (the `vector<blocked_con>` at `C_entity+0x119c`, `structs/entity.md`) sits in the
same translation unit and also answers "something is in the way", but the two share no data. A census of
both vectors finds no function touching both except `entity_ctor` and `entity_dtor`:

| | ghost (`+0x1268`) | blocked connection (`+0x119c`) |
|---|---|---|
| what it holds | a collider to ignore, for 0.5–2 s | a nav-graph edge to avoid, for 30 s (1 s when the blocker is an entity) |
| written by | `ai_path_tick`, `ai_path_revalidate_for_actor` | `entity_add_blocked_con`, from the `blocked_con` the `ai_path_find_spec_node_*` searches leave at `ai_path+0x30` |
| consumed by | the movement collision query (§4a) | the route planner (`structs/ai_wayfind.md`) |

### 4c. `ai_ghost_invalidate_actor` leaves the ignore list alone

`entity_release_target` calls it when a target dies or is despawned, so no record keeps a pointer to an
actor that no longer exists. Its body has no store to `changed_flag`, so the ignore list still holds the
erased records' colliders until something else sets the flag: an append, a marked clear, or a tick that
drops another record.

## 5. PC vs PS2

- **The type names are PS2 symbols.** The PS2 symbol table spells out
  `reserve__Q23gif26vector<14ai_ghost_actor,1>Fi`, and
  `Add__8ai_ghostFP7C_actorP18tDynamicCollObjectUif` types both pointer fields — `C_actor *` and
  `tDynamicCollObject *` — neither of which the PC side could supply.
- **How the cluster maps.** The six PC starts between `entity_is_in_danger` and
  `entity_suspend_ai_and_hide` line up 1:1 against `TickGhost__8C_entityFUi` + the five `ai_ghost`
  methods, once `reserve`, which sits inside the PS2 run, is discounted: MSVC inlines vector growth, so
  a compiler-emitted container helper has no PC function.
- **`TickGhost__8C_entityFUi` ↔ `entity_tick_ghosts`** [CONFIRMED]: the same reverse walk, the same time
  and drift tests (PS2 reads the frame translation at `frame+0x70`, the actor's frame at `actor+0x70`), the
  same shift-down erase and the same rebuild of the ignore list, which PS2 keeps at `C_human+0x6fc` (count
  `+0x700`).
- **Object base: PC `C_entity+0x1268` / PS2 `C_entity+0x1460`.** Pinned by PS2
  `Invalidate(iVar2 + 0x1460, param_2, 0)`, the class's only `Invalidate` callsite, where the vector begins
  at base`+0x00`.
- **The vector head differs by one leading dword.** PS2 keeps `begin` / `end` / `cap` at `+0x00` / `+0x04` /
  `+0x08` and the flag at `+0x0c`, with no allocator member. That single dword is the whole of the flat
  **+4** delta — do not rebase the record with it.
- **Record stride: PC `0x1c` / PS2 `0x20`.** The extra PS2 dword is alignment padding after the position:
  PC packs `pos` as a 12-byte vec3 then `expiry` at `+0x0c`, PS2 uses a 16-byte `S_vector` then `expiry` at
  `+0x10`. Every field from `expiry` on carries a flat +4. `Add` confirms this from the writer side: PS2
  stores four dwords of position where PC stores three.
- **Human collider array:** PC `human+0x690` / `+0x688` ↔ PS2 `+0x6f4` / `+0x6ec`.
- **Clearing strategy differs.** PC clears in place (`end = begin`, allocation retained); PS2 frees the
  buffer via `__dl__FPv` and zeroes all three pointers. It accounts for the 0.55–0.59 size ratios on `Init`
  and `Reset`.
- **`Invalidate` has an extra match arm on PS2** (a second key on `record+0x1c`, the `tDynamicCollObject*`).
  PS2's only callsite passes `0` for that parameter, so the arm is dead everywhere and MSVC eliminated it
  along with the parameter.
- **The two identical siblings (§3e).** `Init` writes `changed_flag = 0` and `Reset` writes `1`, on both
  builds, and both builds emit `Init` first; the same resolution splits `ai_group_can_receive` from
  `ai_group_can_cast`. By consumer position, PC `entity_hit` calls `ai_ghost_clear_all_and_mark_changed`
  at the point where PS2 `Hit__8C_entity` calls `Reset`, and PC's `ai_ghost_clear_all` callers match PS2's
  game-init and reset callers of `Init`.
- **The expiry and radius constants are identical on both platforms** (500 / 800 / 2000 / 1000 ms; 0.09 /
  0.25 / 1.0), as is the `frame_flags & 0x20` world-matrix gate.
- **Size ratios (PC / PS2):**

  | PC name | PC B | PS2 B | ratio |
  |---|---|---|---|
  | `ai_ghost_add_or_refresh` | 688 | 456 + 292 (`reserve`) | 0.92 |
  | `ai_ghost_add_human_colliders` | 80 | 156 | 0.51 |
  | `ai_ghost_clear_all` | 64 | 108 | 0.59 |
  | `ai_ghost_clear_all_and_mark_changed` | 64 | 116 | 0.55 |
  | `ai_ghost_invalidate_actor` | 160 | 280 | 0.57 |

  The four small methods run systematically below 1.0: MIPS prologue / epilogue and delay slots inflate
  small functions, and all four agree.
- **No PS2-only methods** in this class.

## 6. Related

- `structs/entity.md` §7 — the `C_entity` AI sub-object map.
- `structs/ai_path.md` — `ai_path_tick` / `ai_path_revalidate_for_actor`, the producers, and the spec
  waypoint that names the blocker.
- `structs/human.md` — the `+0x688` / `+0x698` / `+0x6a8` collider vectors.
- `structs/ai_sensors.md` — the actual perception system; `ai_ghost` is not one.
- `structs/ai_fight_manager.md` — a neighbouring AI class.

## 7. Open items

| Item | State |
|---|---|
| the stale ignore-list entries after `ai_ghost_invalidate_actor` (§4c) | measured from the bodies; no in-game effect observed |
