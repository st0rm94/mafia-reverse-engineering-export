# `C_wagon` — entity kind `0x1e`, the tram / train wagon

**Created** 2026-08-18 21:09 +0200 · **Modified** 2026-10-11 15:07 +0200

| | |
|---|---|
| PS2 class | `C_wagon` (`7C_wagon`), source file `C_wagon.cpp` |
| PC prefix | `wagon_` |
| PC ctor | `wagon_ctor` (`0x0053c2b0`) |
| PC vtable | `0x0063cff0` |
| kind | `0x1e` |
| PC translation unit | `[0x0053c2b0, 0x005407f0)` — **19 PC starts, 17 728 bytes**, all named |
| Coverage | **18/18 PS2 methods twinned** (§3) |

The TU is a **forced 1:1**: 19 PC starts = 1 MSVC scalar deleting destructor + 18 PS2 methods, in
emission order.

---

## 1. How the class was identified

Two independent PC witnesses, neither of which needed a body diff (the source-file evidence is in
§8):

1. **`wagon_ctor` chains `actor_ctor`** and installs vtable `0x0063cff0`, so it is a
   `C_actor` subclass rather than anything in the `C_Vehicle` family.
2. **`wagon_rebuild_all_forward_paths` filters on the kind byte:** it scans
   `g_actor_global_list` for actors whose `+0x10` is **`0x1e`**.

## 2. Why it sits exactly here — the link-order corroboration

The PC link order is alphabetical by source file, and five consecutive files lead up to this
one (the other build's link order agrees, §8):

```
C_turnout.cpp  ->  C_Vehicle.cpp  ->  _effects.cpp  ->  _kernel.cpp  ->  _sounds.cpp  ->  C_wagon.cpp
0x00508150         0x00509430        0x0051fd00       0x0052c3b0      0x005373e0       0x0053c2b0
```

`C_recorder` follows at `0x005407f0` (see `structs/recorder.md`). So the continuous
vehicle cluster is `[0x00509430, 0x0053c2b0)` and **`C_wagon` is the first class after
it** — see `structs/vehicle.md` §6.

---

## 3. Method coverage

PC-first, ordered by PC VA; every row body-diffed.

| PC VA | PC name | size | PS2 method | size | Grade |
|---|---|---:|---|---:|---|
| 0x0053c2b0 | `wagon_ctor` | 176 | `{ctor}` | 240 | CONFIRMED |
| 0x0053c360 | `wagon_scalar_deleting_dtor` | 32 | - | - | STRUCTURAL |
| 0x0053c380 | `wagon_dtor` | 128 | `{dtor}` | 180 | CONFIRMED |
| 0x0053c400 | `wagon_init` | 32 | `Init` | 32 | CONFIRMED |
| 0x0053c420 | `wagon_save_game_get_size` | 80 | `SaveGameGetSize` | 64 | CONFIRMED |
| 0x0053c470 | `wagon_save_game_save` | 448 | `SaveGameSave` | 472 | CONFIRMED |
| 0x0053c630 | `wagon_save_game_load` | 1056 | `SaveGameLoad` | 944 | CONFIRMED |
| 0x0053ca50 | `wagon_extend_back_node_path` | 496 | `CreateBackVector` | 700 | CONFIRMED |
| 0x0053cc40 | `wagon_build_forward_node_path` | 1024 | `CreateForwardVector` | 1308 | CONFIRMED |
| 0x0053d040 | `wagon_node_is_stop_point` | 48 | `IsStopPoint` | 36 | CONFIRMED |
| 0x0053d070 | `wagon_distance_to_next_stop` | 608 | `MustStop` | 800 | CONFIRMED |
| 0x0053d2d0 | `wagon_find_point_on_back_path` | 1472 | `FindBackPos` | 1872 | CONFIRMED |
| 0x0053d890 | `wagon_apply_spring_sway` | 1040 | `SpringProcess` | 1040 | CONFIRMED |
| 0x0053dca0 | `wagon_ai` | 5824 | `AI` | 5316 | CONFIRMED |
| 0x0053f360 | `wagon_update` | 1136 | `Update` | 1328 | CONFIRMED |
| 0x0053f7d0 | `wagon_game_init` | 3520 | `GameInit` | 5288 | CONFIRMED |
| 0x00540590 | `wagon_cb_collect_wheel_frames` | 144 | `cbEnumFrame` (file-local) | 488 | CONFIRMED |
| 0x00540620 | `wagon_game_done` | 336 | `GameDone` | 376 | CONFIRMED |
| 0x00540770 | `wagon_rebuild_all_forward_paths` | 128 | `RecompileNewPath` | 208 | CONFIRMED |

`wagon_scalar_deleting_dtor` is MSVC-only and has no PS2 method. Size ratios run 0.62–1.33. Five
further PS2 methods have no own PC function (§8).

`wagon_rebuild_all_forward_paths` rebuilds **only** the forward vector `+0xac`/`+0xb0`; the back
path at `+0x80` is read for its first node and never touched. It is a member function whose body
does not use `this`.

---

## 4. The two node paths, and what mapped them

`wagon_save_game_get_size` is statement-identical across the builds and its two
`count * 4` terms are what pinned both vectors at once:

| Role | PC |
|---|---|
| **back path** (behind the wagon; `wagon_find_point_on_back_path` walks it) | `+0x80` / `+0x84` |
| **forward path** (look-ahead; rebuilt over the braking distance) | `+0xac` / `+0xb0` |

Both are capped at **100 entries**.

The node records live in the rail web-node table hanging off `g_act_mission+0x2c`, stride
**`0x1e`**:

| Offset | Field |
|---|---|
| `+0x00`..`+0x08` | position (3 floats) |
| `+0x0c` | flags — **bit 3 = this node is a tram stop** |
| `+0x0e` | node id (read by `wagon_get_last_stop_node_id`, §6a) |
| `+0x18` | edge array (8-byte entries; the flag byte at `+2` marks a reverse edge) |
| `+0x1d` | edge count |

## 5. Field map

`wagon_save_game_save` names the persisted band and `wagon_game_init` names the wheel arrays.

| Offset | Field | Evidence |
|---|---|---|
| `+0x10` | kind = `0x1e` | the filter in `wagon_rebuild_all_forward_paths` |
| `+0x68` | the body frame | index 0 of `wagon_apply_spring_sway` |
| `+0x70`..`+0x78` | a direction vector | `wagon_ai` re-seeds it when its length is zero; persisted (§5a) |
| `+0x80`/`+0x84` | back node path | §4 |
| `+0x8c` | back-path radius | the stop condition in `wagon_extend_back_node_path` |
| `+0x90`..`+0x98` | the wagon's position | `wagon_distance_to_next_stop` measures from it |
| `+0x9c`..`+0xa4` | a second position | advanced by the frame's displacement in `wagon_ai`; read by `wagon_update` |
| `+0xac`/`+0xb0` | forward node path | §4 |
| `+0xb8` | per-wheel spring phase array | `wagon_apply_spring_sway` advances `[i]` |
| `+0xc4` | **wheel frame array** | filled by `wagon_cb_collect_wheel_frames` |
| `+0xd0` | target speed | set to `+0x18c` on departure, to 0 when braking |
| `+0xd4` | speed | numerator of the braking distance; restarts from 0 on departure |
| `+0xd8` | halted at a stop | while set, `wagon_ai` counts `+0xdc` and departs after 10 000 ms |
| `+0xdc` | halt timer, ms | the same |
| `+0xe0` | dynamic collider | `collision_dyn_insert` / `_erase` |
| `+0x168` | forward-path state flag | `wagon_build_forward_node_path` sets and clears it; `wagon_game_init` seeds it to 1 |
| `+0x180` | wheel count | sizes the six arrays `wagon_game_init` allocates |
| `+0x18c` | cruising speed | the target speed a departure restores; the sway phase rate is `speed / +0x18c` |
| `+0x190` | deceleration | denominator of the braking distance `v² / 2a` |
| `+0x194` | the running sound, an `I3D_sound` | `wagon_game_init` creates it as driver object type 4 and loads `nadzemka.wav` through the sound cache; `wagon_ai` restarts it on departure; `wagon_game_done` stops and releases it through slots 0x2c / 0x24 / 0x00 |
| `+0x198` / `+0x19c` | script event: `C_program *`, event id | `wagon_set_event` (§6a) |
| `+0x1a0` | a countdown | `wagon_update` decrements it; at zero it empties the forward path and puts the wagon in act state 1 |
| `+0x1a4` | a linked actor | while set, `wagon_ai` does nothing else: the actor's `+0x1c` at 2 raises the wagon's script event and parks it in act state 2, at 0 drops the link |

**The wheels are named by a Czech string.** `wagon_cb_collect_wheel_frames` collects
every frame whose name suffix matches the 4-character literal **`"kolo"`**
(`s_wagon_wheel_frame_tag`, `0x0064c9b8`) — Czech for *wheel*. That single literal is what
makes `+0xc4` a wheel array rather than an anonymous frame list.

### 5a. The persisted record

`wagon_save_game_save` writes, after the `C_actor` block, a `0x5f`-byte header and then the node
ids of both paths:

| record offset | size | content |
|---|---|---|
| `+0x00` | 1 | version, 1 |
| `+0x01` | 0x18 | `+0x24`..`+0x38`, the cached position and forward vector (`structs/actor.md`) |
| `+0x19` | 0x0c | `+0x90`..`+0x98`, the position |
| `+0x25` | 0x0c | `+0x9c`..`+0xa4`, the second position |
| `+0x31` | 4 | `+0xd0`, the target speed |
| `+0x35` | 4 | `+0xd4`, the speed |
| `+0x39` | 1 | `+0xd8`, halted at a stop |
| `+0x3a` | 4 | `+0xdc`, the halt timer |
| `+0x3e` | 1 | `+0x168`, the forward-path state flag |
| `+0x3f` | 4 | `+0x1a0`, the countdown |
| `+0x43` | 4 | the savegame GUID (`actor+0x18`) of the linked actor `+0x1a4`, -1 when none |
| `+0x47` | 4 | the GUID of the actor owning the event program `+0x198`, -1 when none |
| `+0x4b` | 4 | `+0x19c`, the event id |
| `+0x4f` | 2 | forward-path node count |
| `+0x51` | 2 | back-path node count |
| `+0x53` | 0x0c | `+0x70`..`+0x78`, the direction vector |
| `+0x5f` | 4 per node | the forward-path node ids, then the back-path node ids |

## 6. Behaviour

The wagon drives itself along the rail web graph. Each frame `wagon_ai`:

* keeps the forward path filled over the **braking distance** `v² / 2a`;
* asks `wagon_distance_to_next_stop` how far the next stop-point node is and brakes when
  it is inside that distance;
* places the trailing bogie with `wagon_find_point_on_back_path`;
* applies a sinusoidal suspension sway to both ends with `wagon_apply_spring_sway`
  (phase advanced by `speed / +0x18c`, wrapped at 2π, offset `cos(phase) × 0.01`);
* resolves collisions — and in the **kind-4 arm** (it hit a `C_car`) the PC has
  `C_Vehicle::DoCollisionStatic` **inlined**, which is marked in the decomp.
* halts at a stop: after 10 000 ms halted (`+0xd8`, `+0xdc`) it departs — the target speed
  `+0xd0` becomes the cruising speed `+0x18c`, the speed `+0xd4` restarts from 0 and the running
  sound `+0x194` is restarted.

`wagon_rebuild_all_forward_paths` is the global re-plan: when a `C_turnout` is thrown, every
wagon in the world rebuilds its look-ahead path (`structs/railway_switch.md` §1).

### 6a. Script-facing members

| PC | what |
|---|---|
| `wagon_set_event` (`0x005c8ca0`) | stores `(C_program *, event id)` at `+0x198` / `+0x19c` — the `wagon_setevent` command |
| `wagon_get_last_stop_node_id` (`0x005c8cc0`) | first element of the back-path vector at `+0x80`, indexed into the mission road-node table (stride `0x1e`, id at `+0xe`) — the `wagon_getlastnode` command |

The FIRST entry of the back path is the wagon's last stop, which is why `+0x80` is only ever read
for its first node.

### 6b. Code shared with `C_railway`

Two members are compiled from the same source as their `C_railway` namesakes
(`structs/railway.md`):

| `C_wagon` | `C_railway` | how they compare |
|---|---|---|
| `wagon_apply_spring_sway` (1040 B) | `railway_spring_process` (1040 B) | the same 289 instructions; the only differences are the speed field (`+0x18c` here, `+0x1b4` there) and the sway amplitude constant (0.01 here, 0.03 there) |
| `wagon_find_point_on_back_path` (1472 B) | `railway_find_back_pos` (1424 B) | the instruction sequences agree to 92 % |

## 7. Off-family finds from this TU

| PC name | What it is |
|---|---|
| `game_get_actor_from_guid` (`0x005f0410`) | `C_game::GetActorFromGUID`; names **`actor+0x18` = the savegame GUID** |
| `crt_strnicmp` (`0x00634770`) | MSVC CRT `_strnicmp` (`structs/string.md`) |
| `char_to_lower_locale` (`0x00626378`) | the locale/MBCS sibling of `ascii_toupper` |

## 8. PC vs PS2

* **TU size.** PS2 `[0x0026f2d0, 0x00274450)` = 20 864 bytes against the PC's 17 728 (ratio
  **0.85**). GCC never emits the scalar deleting destructor, hence 19 PC starts for 18 methods.
* **Source file.** `__sinit_C_wagon.cpp` is in the PS2 source-file manifest, so the file was
  compiled into the PS2 build; the PS2 link order matches the PC chain in §2.
* **Five 8-byte PS2 stubs.** `SaveData`, `LoadData`, `DuplicateFrom`, `Hit` and `NetDirect` are
  genuine `return;` / `return 0;` bodies (not tail jumps); the PC folds them onto its shared vtable
  stubs.
* **`RecompileNewPath__7C_wagonFv`** is declared `(void)`, walks the global actor list filtering
  kind `0x1e`, clears the forward vector and calls `CreateForwardVector` with the same `v² / 2a`
  term — statement for statement `wagon_rebuild_all_forward_paths`.
* **`cbEnumFrame`** is a file-local helper struct method (`C_wagon::S_hlp`) on PS2.
* **Field delta.**

| Role | PC | PS2 |
|---|---|---|
| back path | `+0x80` / `+0x84` | `+0x104` / `+0x108` (the last-stop reader uses `+0x100` / `+0x104`) |
| forward path | `+0xac` / `+0xb0` | `+0x114` / `+0x118` |
| script event | `+0x198` / `+0x19c` | `+0x1cc` / `+0x1d0` |
| rail web-node stride | `0x1e` | `0x20` |

## 9. Cross-references

`structs/railway.md` (`C_railway`, the tram system) · `structs/railway_switch.md`
(`C_turnout`, which triggers the global re-plan) · `structs/vehicle.md` §6 (the cluster
map that ends at this class) and §9 (the `DoCollisionStatic` inlining) ·
`structs/recorder.md` (the TU immediately after) · `structs/entity.md` §1-3 (the kind map).

## 10. Open items

| item | state |
|---|---|
| in game: end stop | ride a tram to an end stop and watch whether the wagon reverses in place (`wagon_node_is_stop_point` / `wagon_distance_to_next_stop` / `wagon_extend_back_node_path`) |
| in game: switch | throw a switch: every wagon should re-plan at once |
