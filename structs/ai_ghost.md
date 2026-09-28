# `ai_ghost` — the per-entity obstacle memory ("ghosts")

**Created** 2026-08-02 11:32 +0200 · **Modified** 2026-09-17 13:47 +0200

`ai_ghost` is **how an NPC remembers that something solid was in its way.** While walking a route,
`ai_path_tick` records a "ghost" for every collidable object it meets on the current path segment:
the object's world position at the moment it was seen, plus an expiry stamp and a drift tolerance.
The record survives for a few hundred milliseconds to a couple of seconds, and dies early if the
real object moves away from where it was remembered. It is a short-term spatial memory, not a
perception system — `ai_sensors` does perception.

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
`C_entity`'s own AI helpers. The consumer `entity_expire_ghost_trail` is a **`C_entity` method, not
an `ai_ghost` method** (§3).

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
                                             //     from `pos` before the memory is discarded
    /* +0x14 */ C_actor  *actor;             // [H] who the remembered object belongs to; also
                                             //     half the dedup key and the Invalidate key
    /* +0x18 */ void     *coll_object;       // [H] tDynamicCollObject* - the specific collidable
                                             //     part; the other half of the dedup key
} ai_ghost_actor;                            // 0x1c
```

The type names `ai_ghost_actor` and `tDynamicCollObject` come from the PS2 symbols (§4); the PC
decomp shows only stride-0x1c rows.

The field map is proven **three times independently**: `ai_ghost_add_or_refresh` (dedup key pair
`+0x14`/`+0x18`, refreshed field `+0x0c`), `ai_ghost_invalidate_actor` (key `+0x14`, 7-dword
shift) and `entity_expire_ghost_trail` (expiry `+0x0c`, radius² `+0x10`, actor `+0x14` whose
`+0x70`… is an `I3D_frame`). All three agree.

### 1b. `ai_ghost` — the vector head, at `C_entity+0x1268`

```c
typedef struct ai_ghost {                    // at C_entity+0x1268
    /* +0x00 */ void     *_alloc;            // [M] MSVC std::vector allocator member; never read
                                             //     by game code
    /* +0x04 */ ai_ghost_actor *begin;       // [H] == C_entity+0x126c
    /* +0x08 */ ai_ghost_actor *end;         // [H] count = (end - begin) / 0x1c
    /* +0x0c */ ai_ghost_actor *cap;         // [H]
    /* +0x10 */ uint8_t   changed_flag;      // [H] set to 1 whenever the list is modified or
                                             //     force-cleared; polled and cleared by
                                             //     entity_expire_ghost_trail, which then rebuilds
                                             //     the derived collision-object list at
                                             //     C_entity+0x698 (see §3a)
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
² A human is several collidable parts, so remembering a person means remembering all of them.

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

**Producers — the path follower.** `ai_path_tick` and `ai_path_revalidate_for_actor` record a
ghost for each collidable object found on the current path segment, dispatching on the object's
type field (`obj+0x10`) with **type-dependent expiry and drift tolerance**:

| object type | call | expiry | drift radius² |
|---|---|---|---|
| 4 (vehicle) | `ai_ghost_add_or_refresh` | clock + **500 ms** | **0.09** (0.3²) |
| 6 | `ai_ghost_add_or_refresh` | clock + **800 ms** | **0.25** (0.5²) |
| 0x1b (entity/human) | `ai_ghost_add_human_colliders` | clock + **2000 ms** | **1.0** (1.0²) |
| *(revalidate path)* | either, per type | clock + **1000 ms** | 0.09 / 1.0 |

A parked car is remembered briefly and forgotten the moment it rolls 0.3 units; a person is
remembered four times longer and allowed a whole unit of movement first. That gradient is the
clearest statement of what the structure is for.

**Consumer — the per-frame expiry.** `entity_expire_ghost_trail` (VA 0x0043a220) runs once per
frame from `entity_ai` and drops a record when **either** the clock passes `+0x0c` **or** the real
object has moved further than `+0x10` from the remembered `pos`. If anything was dropped it sets
`changed_flag`, then clears it and rebuilds the derived list at **`C_entity+0x698`** (§3a). It is a
`C_entity` method documented here because it lives in the same TU and the same PS2 order run.

**Scrubber.** `entity_release_target` calls `ai_ghost_invalidate_actor` when a target dies or is
despawned, so no entity keeps remembering an object that no longer exists.

### 3a. `C_entity+0x698` — the derived collision-object list

When `changed_flag` (`ai_ghost+0x10`) is set, `entity_expire_ghost_trail` clears the flag and then:

```c
begin = *(this + 0x69c);                    // stride-4 std::vector at C_entity+0x698
release_range(begin, *(this + 0x6a0));      // drop every element
*(this + 0x6a0) = begin;                    // end = begin  -> the list is now empty
for (i = ghost_count - 1; i >= 0; i--)      // rebuild, walking the ghost vector backwards
    std_vector4_insert_n(*(this + 0x6a0), 1, ghost_record(i) + 0x18);
```

```c
    /* +0x698 */ allocator_t             derived_alloc;  // [M] MSVC std::vector member
    /* +0x69c */ void                  **derived_begin;  // [H] stride 4
    /* +0x6a0 */ void                  **derived_end;    // [H]
```

The pushed value is **`ai_ghost_actor+0x18` — the `tDynamicCollObject *`** (§1a). So `+0x698` is
a **flat list of the collision objects currently remembered as ghosts**, one entry per live ghost
record, kept in sync lazily: it is only rebuilt on the frames where the ghost list actually
changed. That is what `changed_flag` exists for.

`+0x700` is not part of this: it is a **C_human** field, occurring ~25 times in the C_human band
always paired with `+0x6f8` and `+0x704` as movement/anim gates, and `entity_expire_ghost_trail`
never touches it.

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
- `structs/ai_path.md` — `ai_path_tick` / `ai_path_revalidate_for_actor`, the producers.
- `structs/ai_sensors.md` — the actual perception system; `ai_ghost` is not one.
- `structs/ai_fight_manager.md` — a neighbouring AI class.

---

## 6. Open items

1. **Who reads `C_entity+0x698`** is [U]. Its content is known (§3a), but no consumer of the list
   is traced; that is what stands between knowing the layout and knowing what ghosts do.
2. **`ai_ghost_actor+0x00..+0x08` is asserted to be a position from the *writer* side only.** The
   snapshot comes from the actor's `I3D_frame` and the expiry test reads it back as one, so the
   reading is safe — but no consumer treats it as a position for anything other than the drift
   test.
3. **Object type 6 is unidentified.** Types 4 (vehicle) and 0x1b (entity) are documented in
   `structs/entity.md` §3; 6 is not, and it gets its own expiry/radius pair, so it is a real
   distinct case.
4. **Nothing links ghosts to blocked navigation.** `entity_add_blocked_con` (blocked nav-graph
   edges, `C_entity+0x119c`) sits in the same translation unit and answers a similar-sounding
   question. Whether the two systems interact is **[U]** — do not assume they do.
