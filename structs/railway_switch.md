# `C_turnout` — entity kind `0x18`, the railway turnout (points / track switch)

**Created** 2026-08-17 15:33 +0200 · **Modified** 2026-09-17 13:44 +0200

| | |
|---|---|
| PS2 class | `C_turnout` (`9C_turnout`), source file `C_turnout.cpp` |
| PC prefix | **`railway_switch_`** |
| PC constructor | `railway_switch_ctor` @ `0x00508150` |
| PC vtable | `0x0063cee0` |
| PC kind (`entity+0x10`) | **`0x18`** |
| PC translation unit | `[0x00508150, 0x00509400)` — 16 function starts, all named |
| PS2 methods | 17/17 accounted: 15 with their own PC body, `Init` and `Hit` folded onto shared defaults (§7) |

## 1. What a "turnout" is

A **turnout** is the railway engineering term for the movable track junction — the points,
a track switch; Czech *výhybka*. It decides which way a tram goes, and this class implements
exactly that:

* **A swinging blade.** `+0x144` is an angle ramped by `+0x74` rad/ms toward `±(+0x70)`.
  The constructor seeds **0.5 rad** of travel at **0.004 rad/ms**, and every AI tick pushes
  the current angle into the frame's direction.
* **Two alternative rail pieces.** Two 32-byte frame-name literals at `+0x88` and `+0xa8`
  are resolved at `GameInit` into the objects at `+0x198` and `+0x19c`, and exactly one is
  shown per rest position.
* 🔑 **It rewires the navigation graph.** Three `C_web_path` node ids live at `+0x7c`,
  `+0x80` and `+0x84`. At each end-stop the AI disconnects **both** edges from the common
  node and reconnects **exactly one**, then tail-calls `wagon_rebuild_all_forward_paths` so the
  tram recomputes its route. The switch changes where trams can physically go.
* **The player can throw it.** `railway_switch_game_done` withdraws an `S_using_object`
  from the action-menu registry, so the switch is an interactable object.
* **It has a collision body**, a dyn-coll record at `+0x148` whose centre is offset
  `0.8 ×` the local Y axis from the frame (`railway_switch_set_coll_pos`).
* **It makes a noise** — a sound object at `+0x194`, opened at `GameInit` from the sound id at
  `+0x78` and started by `railway_switch_start_switching`.

## 2. State machine

| `+0x190` | meaning |
|---|---|
| `0` | rest position A |
| `1` | rest position B |
| `2` | moving A → B (angle ramping **up** toward `+(+0x70)`) |
| `3` | moving B → A (angle ramping **down** toward `−(+0x70)`) |

`railway_switch_start_switching` is the public verb: `0 → 3` or `1 → 2`, and it starts the
sound. Anything else is ignored, so a switch already in motion cannot be re-triggered.

`railway_switch_get_web_nodes` guards the rewiring: it resolves the three ids through
`C_web_path::FindNodeID` and returns **false** if any lookup failed (`-1`) **or if any two
ids are equal** — the switch requires one common node and two genuinely distinct branches.

## 3. Field map (PC)

| offset | what | evidence |
|---|---|---|
| `+0x10` | entity kind `0x18` | ctor |
| `+0x24`..`+0x2c` | cached world position | `railway_switch_update` |
| `+0x30`..`+0x38` | cached, normalised rotation axis | `railway_switch_update` |
| `+0x68` | the switch's `I3D_frame` | throughout |
| `+0x70` | blade travel, radians (ctor `0.5`) | ctor + AI clamp |
| `+0x74` | blade speed, rad/ms (ctor `0.004`) | ctor + AI ramp |
| `+0x78` | the sound id (ctor `0x85`) | `GameInit` passes it to `sound_table_construct_sound` |
| `+0x7c` / `+0x80` / `+0x84` | the three web-node ids (ctor `0x7b`, `0x7c`, `0x7d`) | `railway_switch_get_web_nodes` |
| `+0x88` (0x20 B) | frame name A | ctor literal, `GameInit` lookup |
| `+0xa8` (0x20 B) | frame name B | ctor literal, `GameInit` lookup |
| `+0xc8` | the mirror flag | SaveData / DuplicateFrom / `GameInit` branch |
| `+0xcc` (0x40 B) | the 4×4 rotation matrix | `Update` rebuilds it each frame |
| `+0xd0`-ish | the published `S_using_object` | `GameDone` withdraws it |
| `+0x10c` | the path node the switch holds | `GameDone` releases it |
| `+0x144` | the current blade angle | AI ramp, savegame `+0xc` |
| `+0x148` | the `tDynamicCollObject` | insert / update / erase |
| `+0x158`..`+0x160` | the collision centre | `railway_switch_set_coll_pos` |
| `+0x190` | the state (§2) | savegame `+4` |
| `+0x194` | the switching sound | `GameInit` / `GameDone` / `Start` |
| `+0x198` / `+0x19c` | the two rail-piece objects | `GameInit` resolves, AI toggles |

**Savegame:** record type `2`, 16 bytes = the state at `+4` and the blade angle at `+0xc`.

## 4. Vtable `0x0063cee0`

Slots 0–12 are the `C_actor_track` base. The class's own:

| slot | function | note |
|---|---|---|
| 13 | `railway_switch_ai` | - |
| 14 | `railway_switch_update` | - |
| 16 | `railway_switch_scalar_deleting_dtor` | MSVC-only |
| 18 | `actor_subclass_init_require_frame` | 🔑 shared by 6 vtables — the family's `Init` default ¹ |
| 19–21 | `railway_switch_save_data` / `_load_data` / `_duplicate_from` | - |
| 22–24 | the `railway_switch_save_game_*` triple | - |
| 25 / 26 | `railway_switch_game_init` / `_game_done` | - |
| 31 | `stub_retfalse_args32_0048a4f0` | 🔑 shared by 13 vtables — the family's `Hit` default ¹ |

¹ A shared realisation of the class's method, not an exclusive twin (§7).

## 5. False function starts in the TU

Ghidra's aggressive instruction finder created four false function starts inside this TU; none
is a function. Three are ordinary epilogue arms. The fourth, `0x0050904b`, starts inside the
4-byte displacement of the `call rel32` at `0x00509049`: Ghidra resynchronised on the `90 90`
padding and decompiled straight through into the real function, so its rendering is a
**verbatim copy of its host's body** — that duplicate rendering is the fingerprint of this shape.

Counted as a function start, it truncates `railway_switch_save_game_load` to **29 B** against a
640 B PS2 method (ratio 0.045, which reads as a mismatch). The true extent is 432 B (0.68).

## 7. PC vs PS2

**Method accounting — 17 PS2 methods:** 15 have their own PC body and match statement for
statement; `Init` folds onto `actor_subclass_init_require_frame` (6 vtables) and `Hit` onto
`stub_retfalse_args32_0048a4f0` (13 vtables). The PS2 `Init` body is
`if (!frame) return false; return C_actor::Init(frame)`, precisely the shared PC function, and
PS2 `Hit__9C_turnout` is an 8-byte stub. The PC additionally emits a scalar deleting destructor
with no PS2 counterpart. Nothing in this class is PS2-port-only and nothing is inlined away.

**Field delta is NOT uniform — do not rebase band-wise.** The two builds agree field for
field in content and differ in offset by `0x100` for the first pair and `0x104` for the
rest:

| PC | PS2 | field |
|---|---|---|
| `+0x70` | `+0x170` | blade travel |
| `+0x74` | `+0x174` | blade speed |
| `+0x78` | `+0x17c` | sound id (`0x85`) |
| `+0x7c` | `+0x180` | web id 1 (`0x7b`) |
| `+0x80` | `+0x184` | web id 2 (`0x7c`) |
| `+0x84` | `+0x188` | web id 3 (`0x7d`) |
| `+0x88` | `+0x18c` | frame name A |
| `+0xa8` | `+0x1ac` | frame name B |
| `+0xc8` | `+0x1cc` | mirror flag |
| `+0x144` | `+0x178` | blade angle |
| `+0x190` | `+0xc0` | state |

The four ctor-seeded ids are **byte-identical on both builds**, which is what makes the mapping
above a measurement rather than an alignment.

**Cross-build corroboration:** where the PC `GameDone` calls `using_object_del_object`, the PS2
calls `DelObject__14C_using_object` at the same statement position. That corroborates that
`using_object_del_object` is a shared helper MSVC folded `DelObject` onto (the *position* belongs
to `C_using_object`, the *symbol* to no class), and it establishes that a turnout is
player-operable.

**PS2 tails into `RecompileNewPath__7C_wagonFv`** where the PC tails into
`wagon_rebuild_all_forward_paths`, in both `AI` and the savegame-load path.

## 8. Open items

| item | state |
|---|---|
| which physical switches exist in the shipped scenes | no census of kind-`0x18` special objects taken; it would give the number and location of turnouts in the tram network |
| in game | throw a switch from the action menu and watch a tram take the other branch — confirms the §1 reading end to end |
