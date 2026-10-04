# `ai_ghost` — the per-entity collision exemptions ("ghosts")

**Created** 2026-08-02 11:32 +0200 · **Modified** 2026-09-29 12:39 +0200

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
  ai_ghost                 vector<ai_ghost_actor> + a changed flag   THIS FILE §1b
      |  holds a vector of
      v
  ai_ghost_actor           pos + expiry + radius² + actor + collider THIS FILE §1a
```

| | |
|---|---|
| PC prefix | **`ai_ghost_`** |
| PC cluster | `[0x0043a430, 0x0043a7b0]` ¹ |
| Owner offset | `C_entity+0x1268` (§1b) |
| PS2 class | `ai_ghost`, 5 methods, 5 mapped, all CONFIRMED by body diff (§2) |

¹ Between the `ai_space_coverage` block and `entity_add_blocked_con` — one translation unit with
`C_entity`'s own AI helpers. The per-frame driver `entity_tick_ghosts` is a **`C_entity` method,
not an `ai_ghost` method** (§3).

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. Layout

### 1a. `ai_ghost_actor` — the ghost record, 0x1c bytes (PC)

```c
typedef struct ai_ghost_actor {              // stride 0x1c on PC
    /* +0x00 */ float     pos[3];            // [H] the object's WORLD position at the moment it
                                             //     was remembered - snapshotted from the actor's
                                             //     I3D_frame, not re-read afterwards
    /* +0x0c */ uint32_t  expiry_ms;         // [H] absolute mission clock; the record dies once
                                             //     the clock passes it
    /* +0x10 */ float     drift_radius_sq;   // [H] squared distance the REAL object may move away
                                             //     from `pos` before the record is discarded
    /* +0x14 */ C_actor  *actor;             // [H] who the object belongs to; also half the
                                             //     dedup key and the Invalidate key
    /* +0x18 */ void     *coll_object;       // [H] tDynamicCollObject* - the specific collidable
                                             //     part; the other half of the dedup key, and the
                                             //     entry copied into the ignore list (§3a)
} ai_ghost_actor;                            // 0x1c
```

The type names `ai_ghost_actor` and `tDynamicCollObject` come from the PS2 symbols (§4); the PC
decomp shows only stride-0x1c rows.

The field map is proven **three times independently**: `ai_ghost_add_or_refresh` (dedup key pair
`+0x14`/`+0x18`, refreshed field `+0x0c`), `ai_ghost_invalidate_actor` (key `+0x14`, 7-dword
shift) and `entity_tick_ghosts` (expiry `+0x0c`, radius² `+0x10`, actor `+0x14` whose `+0x68`
is an `I3D_frame`). All three agree.

**`pos` is read by exactly one statement**: the drift test in `entity_tick_ghosts`, which
compares it against the actor frame's world translation (`frame+0x40/+0x44/+0x48`). The record
buffer is reached only through `+0x126c`, and a displacement census of `C_entity+0x1268..+0x1278`
over `.text` finds only the class's own methods, `entity_tick_ghosts`, the two producers and the
entity lifecycle functions (§3), so no other consumer exists.

### 1b. `ai_ghost` — the vector head, at `C_entity+0x1268`

```c
typedef struct ai_ghost {                    // at C_entity+0x1268
    /* +0x00 */ uint8_t   _alloc;            // [H] MSVC std::vector empty-allocator byte:
                                             //     entity_ctor stores it (mov [esi+0x1268],dl
                                             //     at 0x004115d5); never read by game code
    /* +0x04 */ ai_ghost_actor *begin;       // [H] == C_entity+0x126c
    /* +0x08 */ ai_ghost_actor *end;         // [H] count = (end - begin) / 0x1c
    /* +0x0c */ ai_ghost_actor *cap;         // [H]
    /* +0x10 */ uint8_t   changed_flag;      // [H] set to 1 whenever the list is modified or
                                             //     force-cleared; polled and cleared by
                                             //     entity_tick_ghosts, which then rebuilds the
                                             //     ignore list at C_entity+0x698 (§3a)
} ai_ghost;
```

The object base is `+0x1268`; `C_entity+0x126c` is the vector's **begin pointer** (base`+0x04`).
The base is fixed by `Invalidate`'s single PS2 callsite (§4).

---

## 2. Method families — PC ↔ PS2

Ordered by **PC VA**, PC name first. **5 of 5.**

| PC VA | PC name | PS2 twin | Status | Body-diffed | Role / decisive evidence |
|---|---|---|---|---|---|
| 0x0043a430 | `ai_ghost_add_or_refresh` | `Add__8ai_ghostFP7C_actorP18tDynamicCollObjectUif` | CONFIRMED | yes | Dedup-scans on `(+0x14, +0x18)` and on a hit only refreshes `+0x0c`; otherwise snapshots the frame position, appends, sets `changed_flag` ¹ |
| 0x0043a6e0 | `ai_ghost_add_human_colliders` | `AddHuman__8ai_ghostFP7C_humanUif` | CONFIRMED | yes | One ghost per element of the human's stride-4 collision-object array — count `human+0x690`, data `human+0x688` ² |
| 0x0043a730 | `ai_ghost_clear_all` | `Init__8ai_ghostFv` | CONFIRMED | yes | Clears to empty (`end = begin`), `changed_flag` ← **0**; the family's "`Init` is a teardown" idiom. Callers: `entity_game_init`, `entity_reset` |
| 0x0043a770 | `ai_ghost_clear_all_and_mark_changed` | `Reset__8ai_ghostFv` | CONFIRMED | yes | Identical **except `changed_flag` ← 1** (§2a). Callers: `entity_hit`, `entity_action_tick` |
| 0x0043a7b0 | `ai_ghost_invalidate_actor` | `Invalidate__8ai_ghostFP7C_actorP18tDynamicCollObject` | CONFIRMED | yes | Reverse walk, erases every record whose `+0x14` matches the departing actor. Exactly **one** callsite on each build (`entity_release_target` and its PS2 twin) |
| - | *no PC function* | `gif::vector<14ai_ghost_actor,1>::reserve` | STRUCTURAL | yes | **Not a method:** a compiler-emitted container helper that MSVC inlines into `ai_ghost_add_or_refresh` |

¹ Before the snapshot it forces the actor's world matrix current (gate `frame_flags & 0x20` →
`I3D_frame::UpdateWMatrixProc`, same magic and same callee on both builds).
² A human is several collidable parts, so exempting a person means exempting all of them.

### 2a. The identical-sibling resolution

`ai_ghost_clear_all` and `ai_ghost_clear_all_and_mark_changed` have **byte-identical bodies apart
from one constant**. Neither size, arity nor callee set separates them. Two independent
discriminators agree:

1. **A build-invariant constant** — `Init` writes `changed_flag = 0`, `Reset` writes `1`, on both
   builds; and both builds emit `Init` first. The same resolution splits `ai_group_can_receive`
   from `ai_group_can_cast`.
2. **Consumer position** — PC `entity_hit` calls `ai_ghost_clear_all_and_mark_changed` at the
   point where PS2 `Hit__8C_entity` calls `Reset`; PC's `ai_ghost_clear_all` callers are
   `entity_game_init` and `entity_reset`, matching PS2's game-init/reset callers of `Init`.

---

## 3. Who writes ghosts, and who reads them

**Producers — the path follower.** `ai_path_tick` (while the route is in state 2, following a
spec waypoint) and `ai_path_revalidate_for_actor` ghost the **blocker of the current spec
waypoint** — the actor at waypoint `+0x10`, with its collider at `+0x14` (`structs/ai_path.md`
§1b) — dispatching on the actor's kind (`actor+0x10`) with **kind-dependent expiry and drift
tolerance**:

| actor kind | call | expiry | drift radius² |
|---|---|---|---|
| 4 (`C_car`) | `ai_ghost_add_or_refresh` | clock + **500 ms** | **0.09** (0.3²) |
| 6 (`C_door`) | `ai_ghost_add_or_refresh` | clock + **800 ms** | **0.25** (0.5²) |
| 0x1b (`C_entity`) | `ai_ghost_add_human_colliders` | clock + **2000 ms** | **1.0** (1.0²) |
| *(revalidate path)* | either, per kind | clock + **1000 ms** | 0.09 / 1.0 |

A car is exempted briefly and loses its exemption the moment it rolls 0.3 units; a person keeps
it four times longer and may move a whole unit first. The spec-node searches also pick up a
closed door on the route, which `ai_path_update_spec_waypoints` then opens for a `C_entity`
walker (`structs/ai_path.md` §1, `+0x68`).

**The per-frame tick.** `entity_tick_ghosts` (VA 0x0043a220) runs once per frame from
`entity_ai` and drops a record when **either** the clock passes `+0x0c` **or** the real object
has moved further than `+0x10` from the remembered `pos`. If anything was dropped it sets
`changed_flag`, then clears it and rebuilds the ignore list at **`C_entity+0x698`** (§3a). It is a
`C_entity` method documented here because it lives in the same TU and the same PS2 order run.

**Scrubber.** `entity_release_target` calls `ai_ghost_invalidate_actor` when a target dies or is
despawned, so no entity keeps an exemption for an object that no longer exists.

### 3a. `C_entity+0x698` — the collision ignore list

When `changed_flag` (`ai_ghost+0x10`) is set, `entity_tick_ghosts` clears the flag and then:

```c
begin = *(this + 0x69c);                    // stride-4 std::vector at C_entity+0x698
release_range(begin, *(this + 0x6a0));      // drop every element
*(this + 0x6a0) = begin;                    // end = begin  -> the list is now empty
for (i = ghost_count - 1; i >= 0; i--)      // rebuild, walking the ghost vector backwards
    std_vector4_insert_n(*(this + 0x6a0), 1, ghost_record(i) + 0x18);
```

```c
    /* +0x698 */ uint8_t                 ignore_alloc;   // [H] MSVC empty-allocator byte
                                                         //     (human_ctor stores it)
    /* +0x69c */ void                  **ignore_begin;   // [H] stride 4
    /* +0x6a0 */ void                  **ignore_end;     // [H]
```

The pushed value is **`ai_ghost_actor+0x18` — the `tDynamicCollObject *`** (§1a), one entry per
live ghost record, kept in sync lazily: it is only rebuilt on the frames where the ghost list
changed. That is what `changed_flag` exists for.

**Its two readers.** `human_set_next_coll_owners` and `human_movement_intern` both concatenate
the human's own colliders (`+0x688`, count `+0x690`) and this list into the scratch vector at
`+0x6a8` (begin `+0x6ac`) and pass it to `collision_dyn_set_next_multiple_owners`. Every
`g_collision` query then walks that owner array and skips any object found in it, so the next
collision test ignores both the human's own body and every ghosted object. A displacement census
of `+0x698..+0x6a4` finds no other reader: the list is written by `human_ctor`, `human_dtor`,
`entity_game_init` / `entity_reset` (cleared) and `entity_tick_ghosts`, and read by those two.

`+0x700` is not part of this: it is a **C_human** field, occurring ~25 times in the C_human band
always paired with `+0x6f8` and `+0x704` as movement/anim gates, and `entity_tick_ghosts` never
touches it.

### 3b. Ghosts and blocked connections are separate mechanisms

`entity_add_blocked_con` (the `vector<blocked_con>` at `C_entity+0x119c`, `structs/entity.md`)
sits in the same translation unit and also answers "something is in the way", but the two share
no data. A census of both vectors finds no function touching both except `entity_ctor` and
`entity_dtor`:

| | ghost (`+0x1268`) | blocked connection (`+0x119c`) |
|---|---|---|
| what it holds | a collider to ignore, for 0.5–2 s | a nav-graph edge to avoid, for 30 s (1 s when the blocker is an entity) |
| written by | `ai_path_tick`, `ai_path_revalidate_for_actor` | `entity_add_blocked_con`, from the `blocked_con` the `ai_path_find_spec_node_*` searches leave at `ai_path+0x30` |
| consumed by | the movement collision query (§3a) | the route planner (`structs/ai_wayfind.md`) |

---

## 4. PC vs PS2

- **The type names are PS2 symbols.** The PS2 symbol table spells out
  **`gif::vector<14ai_ghost_actor,1>::reserve`**, and
  `Add__8ai_ghostFP7C_actorP18tDynamicCollObjectUif` types both pointer fields — `C_actor *` and
  **`tDynamicCollObject *`** — neither of which the PC side could supply.
- **How the cluster maps.** The six PC starts between `entity_is_in_danger` and
  `entity_suspend_ai_and_hide` line up 1:1 against `TickGhost__8C_entityFUi` + the five `ai_ghost`
  methods, once `gif::vector<14ai_ghost_actor,1>::reserve` — which sits inside the PS2 run — is
  discounted: MSVC inlines vector growth, so a compiler-emitted container helper has no PC
  function. The gap is forced against the PS2 entries that can have a PC function, not against
  the raw PS2 run length.
- **`TickGhost__8C_entityFUi` ↔ `entity_tick_ghosts`** [CONFIRMED]: the same reverse walk, the
  same time and drift tests (PS2 reads the frame translation at `frame+0x70`, the actor's frame
  at `actor+0x70`), the same shift-down erase and the same rebuild of the ignore list, which PS2
  keeps at `C_human+0x6fc` (count `+0x700`).
- **Object base: PC `C_entity+0x1268` / PS2 `C_entity+0x1460`.** Pinned by PS2
  `Invalidate(iVar2 + 0x1460, param_2, 0)`, the class's only `Invalidate` callsite, where the
  vector begins at base`+0x00`.
- **The vector head differs by one leading dword.** PS2 keeps `begin`/`end`/`cap` at
  `+0x00`/`+0x04`/`+0x08` and the flag at `+0x0c`, with no allocator member; PC keeps an MSVC
  `std::vector` allocator member at `+0x00` and pushes everything to `+0x04`/`+0x08`/`+0x0c`/`+0x10`.
  That single dword is the whole of the flat **+4** delta — do not rebase the record with it.
- **Record stride: PC `0x1c` / PS2 `0x20`.** The extra PS2 dword is **alignment padding after the
  position**: PC packs `pos` as a 12-byte vec3 then `expiry` at `+0x0c`, PS2 uses a 16-byte
  `S_vector` then `expiry` at `+0x10`. Every field from `expiry` on carries a flat +4. `Add`
  confirms this from the writer side — PS2 stores **four** dwords of position where PC stores
  three.
- **Human collider array:** PC `human+0x690` / `+0x688` ↔ PS2 `+0x6f4` / `+0x6ec`.
- **Clearing strategy differs.** PC clears in place (`end = begin`, allocation retained); PS2 frees
  the buffer via `__dl__FPv` and zeroes all three pointers. An allocator-strategy difference, not a
  behavioural one — it accounts for the 0.55–0.59 size ratios on `Init`/`Reset`.
- **`Invalidate` has an extra match arm on PS2** (a second key on `record+0x1c`, the
  `tDynamicCollObject*`). **Not a behavioural difference:** PS2's only callsite passes `0` for
  that parameter, so the arm is dead everywhere and MSVC eliminated it along with the parameter.
- **`gif::vector<14ai_ghost_actor,1>::reserve` has no PC function** because of factoring (MSVC
  inlines it into `ai_ghost_add_or_refresh`), not because it is PS2-only.
- **The expiry and radius constants are identical on both platforms** (500/800/2000/1000 ms;
  0.09/0.25/1.0), as is the `frame_flags & 0x20` world-matrix gate.
- **Size ratios (PC / PS2):**

  | PC name | PC B | PS2 B | ratio |
  |---|---|---|---|
  | `ai_ghost_add_or_refresh` | 688 | 456 + 292 (`reserve`) | 0.92 |
  | `ai_ghost_add_human_colliders` | 80 | 156 | 0.51 |
  | `ai_ghost_clear_all` | 64 | 108 | 0.59 |
  | `ai_ghost_clear_all_and_mark_changed` | 64 | 116 | 0.55 |
  | `ai_ghost_invalidate_actor` | 160 | 280 | 0.57 |

  The four small methods run systematically below 1.0 — MIPS prologue/epilogue and delay slots
  inflate small functions, and all four agree.
- **No PS2-only methods** in this class.

---

## 5. Related

- `structs/entity.md` §6c — the `C_entity` AI sub-object map.
- `structs/ai_path.md` — `ai_path_tick` / `ai_path_revalidate_for_actor`, the producers, and the
  spec waypoint that names the blocker.
- `structs/human.md` — the `+0x688`/`+0x698`/`+0x6a8` collider vectors.
- `structs/ai_sensors.md` — the actual perception system; `ai_ghost` is not one.
- `structs/ai_fight_manager.md` — a neighbouring AI class.
