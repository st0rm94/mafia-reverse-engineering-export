# `C_cartridge` — the ejected shell casing (PC prefix `cartridge_`)

**Created** 2026-08-12 20:08 +0200 · **Modified** 2026-09-17 13:44 +0200

| | |
|---|---|
| PC kind (`entity+0x10`) | **`0x10`** |
| PC vtable | `.rdata 0x0063be18` (39 slots) |
| PC instance size | `0x13c` |
| PC translation unit | `[0x00476f80, 0x00477300)` — 8 real starts, all named |
| Base class | **`C_bottle`** (`structs/bottle.md`) |
| PS2 class | `C_cartridge` — data only, no member functions (§5) |

**Naming.** `cartridge_` maps the PS2 class tag `11C_cartridge` onto a lowercase subsystem
prefix. The English word is also what the thing *is* — a spent cartridge case. `cartridge_` is
not shadowed by `car_` (`"cartridge_".startswith("car_")` is false).

---

## 1. What the class is, and how it was identified

A `C_bottle` subclass: the **spent cartridge case flicked out of a firing weapon**. It
inherits the whole ballistic arc from `C_bottle` and adds only a landing sound and a
retirement rule.

The class sits in the PC `.rdata` vtable order directly after `C_car_ex`. Five levers agree, and
no two of them share a premise:

| # | lever | result |
|---|---|---|
| 1 | PC `.rdata` vtable order | `0x0063bd68` (`C_car_ex`) → **`0x0063be18`** → `0x0063beb8` (`C_clock`) |
| 2 | link order by source file | `C_car_ex.cpp` → **`C_cartridge.cpp`** → `c_clock.cpp` |
| 3 | the source-file manifest | `__sinit_C_cartridge.cpp` exists ⇒ the file was **compiled into the build** |
| 4 | a static class member | `sounds__11C_cartridge` ↔ PC `g_cartridge_sounds`, 3 slots, same walk in the same teardown position |
| 5 | PC-side, independent of 1–3 | the ctor chains `bottle_ctor` and writes kind **`0x10`**; `operator_new(0x13c)` against `C_bottle`'s `operator_new(0x138)` |

Lever 5's arithmetic is the sharpest of the set: the subclass adds **exactly one dword**, and
`+0x138` is the one field the class actually uses. Nothing else can be hiding.

### 1a. The two pools

`game_init` builds two ring buffers on the `C_game` singleton from two shipped models — the
Czech *nábojnice* is a cartridge case:

| pool | model | entries | array | cursor |
|---|---|---|---|---|
| A | `2nabojnice.i3d` | **100** | `C_game+0x3a8` | `C_game+0x538` |
| B | `2nabojniceb.i3d` | **30** | `C_game+0x53c` | `C_game+0x5b4` |

Both are built through the frame named `Line01`, each element `operator_new(0x13c)` +
`cartridge_ctor`. `game_new_cartridge` takes the next slot, wraps at the pool size, and never
allocates at runtime. `game_done` and `game_done_intern` release both pools.

---

## 2. Field layout (PC)

The class adds **one** field. Everything else is `C_bottle`'s, and the interesting part is
which inherited fields it *overrides the meaning of*.

```c
struct C_cartridge /* : C_bottle, 0x13c bytes */ {
    /* ...0x000..0x137 : C_bottle — see structs/bottle.md §1 ... */
    /* +0x138 */ char type;   // [H] THE ONLY OWN FIELD. Set at eject time by
                              //     cartridge_set_type; selects the landing-sound column
                              //     AND halves +0x88. A BOOL per the spawner's PS2
                              //     signature (§5c).
};
```

### 2a. Inherited fields this class re-purposes or pins down

| field (`bottle.md` name) | what `C_bottle` does | what `C_cartridge` does |
|---|---|---|
| `+0x6d` `may_broadcast` | ctor sets **1** | `cartridge_ctor` **clears it** — a casing never replicates |
| `+0xfc` `coll_registered` | ctor sets **1** | `cartridge_ctor` **clears it** — no dynamic-collision proxy is registered |
| `+0xa4` `radius` | `bottle_game_init` computes it from the model's bound volume | `cartridge_game_init` **overwrites it with a fixed `0.1f`** |
| `+0xa8..+0xb0` | `[U]` "unknown vector" | set to **`(0, 1, 0)`** |
| `+0x84` `landed_sets_state` | chooses act-state 1 vs 2 at flight end | cleared |
| `+0x88` | `[M]`, GameInit = `0.4f`, no reader traced | second writer: `cartridge_set_type` halves it to `0.2f` for type B; no reader (§6) |
| `+0x100` | `[U]`, saved, no reader traced | read by `cartridge_ai_tick`, which retires the casing once it passes **15** |
| `+0x104` | `[U]`, saved | read as a gate that forces the landing-sound path |
| `+0x12c` | part of "the saved tail" | the **surface material id** of the last contact — the index into `g_surface_material_table` |

---

## 3. Method coverage — PC first

| PC VA | PC name | vtable slot | PS2 twin | grade |
|---|---|---|---|---|
| 0x00476f80 | `cartridge_ctor` | - (factory) | - | CONFIRMED |
| 0x00476fb0 | `cartridge_scalar_deleting_dtor` | 16 | - | CONFIRMED |
| 0x00476fd0 | `cartridge_dtor` | - | - | CONFIRMED |
| 0x00476fe0 | `cartridge_ai_tick` | 13 | - | CONFIRMED |
| 0x004771f0 | `thunk_bottle_update` | 14 | - | STRUCTURAL |
| 0x00477200 | `cartridge_game_init` | 25 | - | CONFIRMED |
| 0x00477260 | `cartridge_game_done` | 26 | - | CONFIRMED |
| 0x004772e0 | `cartridge_set_type` | - (non-virtual) | - | STRUCTURAL |

Plus one off-family function that exists only to serve this class:

| PC VA | PC name | PS2 twin | grade |
|---|---|---|---|
| 0x005e7a40 | `game_new_cartridge` | `NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` | STRUCTURAL |

Every PS2 cell in the first table is `-` because PS2 has no `C_cartridge` member function at all
(§5a) — a measured result, not an unsearched gap.

### 3a. What `cartridge_ai_tick` does

1. Chain `bottle_ai_tick` — the inherited ballistics.
2. **Spent?** `(flight_speed < 0.01 && fall_accum < 0.01)` ‖ `fall_accum > 50.0` ‖ `+0x100 > 15`.
3. If spent, or `+0x104` is set, and the player frame is within **100.0** squared units:
   round-robin the three `g_cartridge_sounds` slots for a free one, build a sound from
   `g_surface_material_table[+0x12c].{+0x28 | +0x2c}` — the column chosen by `+0x138` — place
   the emitter at the casing's position, rewrite the emitter frame's flag word
   `& 0xfffffedf | 0x40000000` (the project's documented sound-emitter fingerprint) and attach
   it to the mission scene.
4. If spent: `game_invalidate_actor` + `game_remove_temporary_actor` — the slot returns to the
   pool; the act-state 2 that `game_remove_temporary_actor` writes is the "spent" state.

---

## 4. The vtable — `0x0063be18`, 39 slots

Slot-for-slot aligned with `C_bottle`'s `0x0063b870`. Real overrides: **13 (AI)**,
**16 (sdd)**, **25 (GameInit)**, **26 (GameDone)**. Slot 14 (Update) holds a 5-byte jmp to
`bottle_update`.

Slots **15, 18, 22, 23, 24, 30, 31** hold symbols that are *not* `C_bottle`'s. A derived class
cannot "un-override" its base: these are **empty overrides that identical-COMDAT-folding merged
onto other classes' identical ones** — `0x00482690` sits in 16 vtables, `0x0048a4f0` in 13,
`0x00462330` in 14.

| slot | PC VA | PC name | holders |
|---|---|---|---|
| 15 | `0x004f61d0` | `thunk_bottle_net_apply_remote_state` | 2 datarefs |
| 18 | `0x0048af90` | `bottle_subclass_init_require_frame` | 3 ¹ |

¹ `C_cartridge` (`0x0063be18`), `C_grenade` (`0x0063c3c0`) and `C_smoke_trail` (`0x0063cd80`, PC
ctor `smoke_trail_ctor` @ `0x004f60e0`). Body: `frame ? bottle_init(frame) : false`. The mapping
is one-to-three, so it carries no single PS2 twin; see `structs/grenade.md` §3a.

The ctor census of `C_bottle` subclasses finds two more besides this one: `grenade_ctor`
(`0x0048af30`) and `smoke_trail_ctor` (`0x004f60e0`), matching PS2's `C_grenade` and
`C_smoke_trail`.

---

## 5. PC vs PS2

### 5a. The PS2 side of this class is data only

PS2 emits **no `C_cartridge` member function at all**. Three structurally different searches
agree (a plain text grep of the PS2 decomp can miss a definition whose line Ghidra wrapped, so
the symbol table and the VA span are checked too):

1. the only `11C_cartridge` token in the entire PS2 decomp is `sounds__11C_cartridge`;
2. the PS2 symbol table has **zero** rows whose class is `C_cartridge` — the only
   cartridge-bearing symbols in the whole build are `ClearCartridges__6C_gameFv`,
   `NewCartidge__6C_game...` and `__sinit_C_cartridge.cpp`;
3. the PS2 VA span between `C_car_ex`'s last method (`0x146790`) and `C_clock`'s first
   (`0x1478c0`) holds only `C_car_ex` container helpers.

### 5b. Why — the port gutted the spawner

`NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb` is an **8-byte `{ return; }` stub** on PS2,
against 1488 bytes on the PC. Nothing on the PS2 ever creates a casing, so the linker had
nothing to keep. The port did *not* delete the concept: it still ships
`ClearCartridges__6C_gameFv` (192 B) and both pools — at **10** and **5** entries against the
PC's 100 and 30 — and it still ships the class's static sound array.

⇒ In this subsystem **the PS2 method count is a lower bound**, not an upper bound.

### 5c. The one twin that can be graded, and why it is STRUCTURAL not CONFIRMED

`game_new_cartridge` ↔ `NewCartidge__6C_game...`. The **symbol** match is exact: the PS2
signature `FRP9I3D_frameR8S_vectorbb` = `(I3D_frame *&, S_vector &, bool, bool)` maps argument
for argument onto the PC function, in order, including the double indirection on the frame
(the PC body reads `*param_2`). The trailing `bb` is why `+0x138` is a bool. But an empty PS2 body
confirms the *symbol* and never the *bodies*, so the grade stays STRUCTURAL.

### 5d. Field bridge

Not derivable: with no PS2 method bodies there is no PS2 field evidence for this class. The
only cross-build datum is `g_cartridge_sounds` ↔ `sounds__11C_cartridge`, which is a static,
not an instance field.

---

## 6. Open items

* **`+0x88` has no reader.** This class supplies its second writer (0.4f → 0.2f for type B), but
  nothing in either build consumes it. A byte scan of `.text` for the displacement is the next
  lever.
* **What the two casing variants are is unnamed on both builds.** `+0x138` demonstrably
  selects pool + model + landing-sound column; nothing says whether B is a shotgun shell, a
  rifle case, or simply a second art asset. That is why `cartridge_set_type` is graded
  STRUCTURAL and the word *type* is marked `[M]`.
* **`+0x100` and `+0x104` semantics.** The reader is known; the *meaning* of the counter
  (bounces? contacts? ticks?) is not.
