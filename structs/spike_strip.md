# `C_irenka` — the police spike strip (entity kind `0x1f`, PC prefix `spike_strip_`)

**Created** 2026-08-14 09:10 +0200 · **Modified** 2026-09-17 13:43 +0200

This documents the PC build. All cross-build material is confined to §6.

| | |
|---|---|
| **PS2 class** | `C_irenka` (13 methods) |
| **PC prefix** | `spike_strip_` |
| **kind** (`+0x10`) | **`0x1f`** |
| **PC vtable** | `0x0063c568`, 39 slots |
| **PC TU** | `[0x004ae830, 0x004af000)` — **10 starts, 0 unnamed** |
| **Base** | `C_actor` (the ctor chains `actor_ctor`) |
| **Asset** | `barrier.4ds` |

---

## 1. What it is

A static, deployable obstacle the police lay across a road. In gameplay a roadblock is a
**composite** — patrol cars (`police_manager` vec2) + spike strips (vec3) + foot cops with
Thompsons and shotguns (vec4) — and this class is only the strip. Its whole behaviour is:
be placed, register a 2-D crossing segment, and answer "did something drive over me".

It has no AI worth the name: `spike_strip_ai_tick` sets act-state 1 and returns.

---

## 2. Method coverage — 10 PC starts, 13/13 PS2 methods accounted

| PC VA | PC name | PS2 method | Grade |
|---|---|---|---|
| `0x004ae830` | `spike_strip_ctor` | `__ct__8C_irenkaFv` | CONFIRMED |
| `0x004ae860` | `spike_strip_deleting_dtor` | `__dt__8C_irenkaFv` | CONFIRMED |
| `0x004ae880` | `spike_strip_dtor` | `__dt__8C_irenkaFv` | CONFIRMED |
| `0x004ae890` | `actor_subclass_init_require_frame` | `Init__8C_irenkaFP9I3D_frame` | STRUCTURAL (§3) |
| `0x004ae8b0` | `spike_strip_ai_tick` | `AI__8C_irenkaFUi` | CONFIRMED |
| `0x004ae8c0` | `spike_strip_game_init` | `GameInit__8C_irenkaFv` | CONFIRMED |
| `0x004aeb70` | `spike_strip_game_done` | `GameDone__8C_irenkaFv` | CONFIRMED |
| `0x004aebb0` | `spike_strip_deactivate` | `Deactivate__8C_irenkaFv` | CONFIRMED |
| `0x004aebe0` | `spike_strip_activate` | - (PC-only) | CONFIRMED |
| `0x004aeda0` | `spike_strip_test_segment_crossing` | `ZapichameSi__8C_irenkaFRC8S_vectorRC8S_vector` | CONFIRMED |

The remaining PS2 methods are stubs whose PC realisations are **shared** functions; a body shared
by several classes has no single owner, so none of them carries a twin line:

| PS2 method | PC slot | PC realisation |
|---|---|---|
| `Update`, `NetDirect` | 14, 15 | `stub_noop_args4_00461180` (the shared `ret 4` stub) |
| `SaveGameGetSize` / `SaveGameSave` / `SaveGameLoad` | 22, 23, 24 | `stub_ret0_cdecl_00462420` / `stub_rettrue_args4_00462330` ×2 |

**No epilogue phantoms.** All 10 starts were tested for being an early-out / epilogue block of an
enclosing function (no `E8` callsite, no data reference, a branch into it, no alignment padding
before it, restore-fragment bytes): none qualifies.

### 2a. The vtable slot map (`0x0063c568`)

Identical in shape to the `C_dog` table, which is how `GameInit` was placed independently of the
body diff: **13** AI · **14** Update · **15** NetDirect · **16** sdd · **17** Release · **18** Init
· **19/20** SaveData/LoadData · **21** DuplicateFrom · **22/23/24** the SaveGame triple ·
**25 GameInit** · **26 GameDone** · **29** Death · **30** Collision. Slots 0–12 are the
`C_actor_track` keyframe-recorder band (`structs/actor_track.md`).

`spike_strip_deactivate`, `spike_strip_activate` and `spike_strip_test_segment_crossing` hold
**no** slot — they are non-virtual and called directly.

---

## 3. `Init` is a fold owned by no class

`actor_subclass_init_require_frame` (`0x004ae890`) is statement-identical to PS2's
`Init__8C_irenkaFP9I3D_frame` (`if (!frame) return false; return C_actor::Init(frame) != 0;`) —
but a scan of `.rdata` for the address as a pointer puts it at **slot 18 of six vtables**:
`0x0063b7c0` (`C_airplane`), `0x0063c0d0` (`C_dog`), `0x0063c568` (this class), plus `0x0063ca58`,
`0x0063cc98` and `0x0063cee0` (§7).

So it is named for the **family** and carries **no `PS2 twin:` line** — a docblock cannot say
"one of six". It is a second, disjoint identical-code fold of the same body as
`bottle_subclass_init_require_frame` (3 vtables, the `C_bottle` subclasses).

---

## 4. The activate / deactivate pair, and the placement pipeline

`police_manager_deploy_temp_spike_strip` (`0x005ce8d0`) is the only creator path:

1. `police_manager_spike_strip_pool_pop` takes a strip from `police_manager+0x60` (vec3);
2. the requested position is **ground-snapped** with `collision_test_line_vertical`, a
   `-5.0f` downward probe;
3. the position is written into the strip's `I3D_frame` at `+0x80/+0x84/+0x88` and the
   frame-move flag rewrite `& 0xfffffedf | 0x40000000` is applied;
4. **`spike_strip_activate`** commits it;
5. `actor_rebirth` + `game_add_temporary_actor_intern` publish it.

`spike_strip_activate` and `spike_strip_deactivate` are exact inverses:

| | activate | deactivate |
|---|---|---|
| scene | re-links the frame into the mission scene | unlinks (`+0x2c` with `0,0`) |
| visibility | frame vtable `+0x24` with **1** | frame vtable `+0x24` with **0** |
| bounds | `UpdateHRBoundVol`, endpoints `+0xfc..+0x108`, inverse matrix `+0xbc`, `+0x88 += 0.5` | - |
| collision | **`collision_dyn_insert(this+0x74)`** | **`collision_dyn_erase(this+0x74)`** |

`spike_strip_activate` is not `GameInit`: `spike_strip_game_init` (`0x004ae8c0`, slot 25) is.

---

## 5. Known fields

| Offset | Field | Conf | Notes |
|---|---|---|---|
| `+0x10` | kind = `0x1f` | H | written by the ctor |
| `+0x68` | the strip's `I3D_frame` | H | everything else derives from it |
| `+0x70` | deployed flag | H | cleared by `game_init`; gates the teardown in `game_done` |
| `+0x74`..`+0xbb` | the dynamic-collision object (0x48 B) | H | zeroed by ctor and `game_init`; `+0x74` = type `0x40`, `+0x78` = the frame |
| `+0x88` | bounds extent, inflated by `+0.5` | H | - |
| `+0x90` | bounding radius = max-axis scale × extent × 0.5 | H | - |
| `+0xb8` | back-pointer to the entity | H | - |
| `+0xbc`..`+0xfb` | inverse world matrix (4×4) | H | recomputed by activate |
| `+0xfc`..`+0x108` | the two world-space strip endpoints | H | what `spike_strip_test_segment_crossing` tests against |

---

## 6. PC vs PS2

* **Class name.** `C_irenka` — Czech *Irenka*, a developer's pet name; the police code calls the
  object an **Iris** (`FreeTempIris`, `S_road_block_iris`).
* **Field skew.** PC `+0x68/+0x70/+0x74/+0xb8` ↔ PS2 `+0x70/+0xc0/+0xd0/+0x118`; PS2's
  dyn-coll block is `0x60` bytes where the PC's is `0x48`, so **do not rebase band-wise**.
  PS2 writes the kind at `+0x50`, the PC at `+0x10`.
* **`ZapichameSi`** is Czech, roughly *"we're sticking it in"* — a developer joke, not a
  description. The PC name states the behaviour (`spike_strip_test_segment_crossing`).
* **The container lever names the pool record.** `C_shvestky::S_road_block_iris` is the
  element type of the vec3 at `police_manager+0x60` — independent corroboration of the whole
  spike-strip reading, and a type name `structs/police_manager.md` carries untyped.
* **PS2 has no acquire method.** `FreeTempIris__10C_shvestkyFP8C_irenka` exists;
  `GetTempIris` does not, although `GetTempCar` and `GetTempEnemy` do. So the PS2 deploy
  path is inlined or structured differently — which is consistent with `spike_strip_activate`
  having no PS2 rank.

---

## 7. Open items

| item | state |
|---|---|
| vtables `0x0063ca58`, `0x0063cc98`, `0x0063cee0` | carry `actor_subclass_init_require_frame` at slot 18; their classes are unidentified |
