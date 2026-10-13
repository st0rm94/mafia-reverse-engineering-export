# `C_railway` — the ambient rail-traffic system (entity kind 8)

**Created** 2026-08-16 22:24 +0200 · **Modified** 2026-10-07 21:57 +0200

| | |
|---|---|
| PC kind (`entity+0x10`) | **`8`** |
| PC vtable | `.rdata 0x0063cbd0` |
| PC constructor | `railway_ctor` @ `0x004d74e0` |
| PC instance size | `0x36c` (entity factory `case 8`) |
| PC translation unit | `[0x004d35f0, 0x004ea1a0)` — 127 function starts, all named |

`C_railway` is the trams **and** the elevated train — the split against `C_wagon` is
documented in `entity.md` §3 and is not repeated here.

**The track switches are a separate class.** `C_turnout` (entity kind `0x18`, PC prefix
`railway_switch_`, `structs/railway_switch.md`) is the turnout: it swings a blade, shows one of
two rail-piece frames, and at each end-stop disconnects both `C_web_path` edges from a common node
and reconnects one, and its AI then calls `wagon_rebuild_all_forward_paths`, which rebuilds the
path of every `C_wagon` (kind `0x1e`). The two shipped turnouts are in `MISE15-PRISTAV`, a scene
with no `C_railway`, so they route the mission wagons, not the trams. `railway_switch_` extends
this class's
`railway_` prefix, so a prefix match on `railway_` also catches the 16 turnout functions.

---

## 1. What the class actually is

A rail vehicle is an engine plus *n* trailing vagons, running a route of `C_web_node`s,
carrying passengers who walk to seats, with doors that open at stops.

| concern | where it lives |
|---|---|
| the route ahead / behind | node vectors at `+0x80`/`+0x84` and `+0xac`/`+0xb0` |
| the vagons | pointer array at `+0xc4`, count at `+0x1a8`; index 0 is the engine's own frame |
| the passengers | pointer list at `+0x254`/`+0x258`; the driver is EMBEDDED at `+0x2b0` |
| the seats | `S_RailSeats`, one bank per vagon |
| the doors | `rail_doors`, two leaf banks |
| the windows | `rail_glass` / `rail_window`, `0x14` bytes per window |
| the boarding points | `rail_entries`, two banks: inner `+0x270`, outer `+0x274` |
| sound | three slots at `+0x28c`, `+0x290`, `+0x294`: running loop, bell or door, curve squeal (§3b) |
| run state | `+0xdc`, written only by `railway_set_state` |
| speed | `+0xd4` |
| boarding lock | `+0x288`, held/released by `railway_lock` / `railway_unlock` |
| the vehicle ahead | `+0x160`, found by `railway_find_vehicle_ahead` |
| material id | `+0x2ac` |

### 1a. Field map (PC)

| offset | meaning | established by |
|---|---|---|
| `+0x68` | the actor's own `I3D_frame` (the engine) | `railway_get_vagon` index 0 |
| `+0x80` / `+0x84` | forward node vector, begin / end | `railway_build_forward_node_vector` |
| `+0xac` / `+0xb0` | back node vector, begin / end | `railway_build_back_node_vector` |
| `+0xbc` | vagon position array (stride `0xc`) | `railway_get_nearest_vagon` |
| `+0xc4` | vagon frame array | `railway_get_vagon` |
| `+0xd4` | current speed | `railway_is_in_stop`, `railway_spring_process` |
| `+0xdc` | run state | `railway_set_state` |
| `+0xe0` | saved state while holding behind another vehicle | `railway_hold_behind_vehicle_ahead` |
| `+0x160` | the rail vehicle ahead on the track | `railway_find_vehicle_ahead` |
| `+0x1a8` | vagon count | `railway_duplicate_from`, `railway_save_data` |
| `+0x1b4` | speed scale (designer) | `railway_duplicate_from` |
| `+0x1d8` | the elevated-train flag: 1 when the model file is `metro.i3d`, 0 for a tram; it selects the sound ids (§3b) and is tested by `railway_is_in_stop`, `railway_ai` and `railway_update` | `railway_game_init` |
| `+0x1e0` / `+0x1e4` | boarding-entry count / array (stride `0x18`) | `railway_get_outer_entry_at` |
| `+0x254` / `+0x258` | passenger list, begin / end | `railway_do_panic`, `railway_remove_person` |
| `+0x270` / `+0x274` | inner / outer published entry banks (stride `0x38`) | `railway_gen_entries` |
| `+0x288` | boarding-lock counter | `railway_lock` / `railway_unlock` |
| `+0x28c`..`+0x294` | the three sound objects | `railway_game_init`, `railway_on_act_state_changed` |
| `+0x29c`..`+0x2a4` | the three `sounds.dat` ids, one per sound object (§3b) | `railway_game_init` |
| `+0x2a8` | in-range state (0 or 2 = eligible) | `railway_any_passenger_wants_off` |
| `+0x2ac` | collision material id | `railway_get_material_id` |
| `+0x2b0` | the DRIVER, an embedded `S_rail_human` | `railway_ctor`, `railway_do_panic` |

---

## 2. The five helper types in the same file

### 2a. `S_RailSeats` — the seat banks (`rail_seats_`)

A bank is an array of **rows**, stride `0x38`. A row is a *bench*: it stores a start point
at `+0x00` and an end point at `+0x0c`, and seat positions are **linearly interpolated
between them** by the seat's offset within the row (`rail_seats_get_seat_pos`). The row
also carries a facing vector at `+0x18`, its base seat index at `+0x24`, its seat count at
`+0x28`, its carriage index at `+0x2c`, and two parallel arrays: occupancy at `+0x30` and owner at
`+0x34`.

Occupancy is a three-state value, which is what the three mutators are for:

| value | meaning | set by |
|---|---|---|
| 0 | free | `rail_seats_empty_seat` |
| 1 | reserved — a passenger is walking to it | `rail_seats_reserve_seat` |
| 2 | seated | `rail_seats_occupy_seat` |

🔑 `rail_seats_find_free_seat` scans rows and seats from a **shared rotating cursor**
(`g_rail_seat_search_cursor`), incremented on every successful pick. That is why boarding
passengers spread through a carriage instead of all choosing the same bench. It returns
`0xffff` when nothing is free.

`rail_seats_duplicate_rows` replicates the row block once per carriage (the engine plus each
vagon), so every carriage shares one layout, and `rail_seats_init` then stamps each copy's `+0x2c`
with its carriage index: the row index divided by the rows per carriage, 0 being the engine.
`rail_seats_find_free_seat` matches the index exactly; `rail_seats_count_free` takes `-1` for any
carriage.

### 2b. `S_rail_human` — a passenger (`rail_passenger_`)

| offset | meaning |
|---|---|
| `+0x04` | the passenger's `I3D_frame` |
| `+0x08` | `C_animations_machine_de_luxe*` — a heap machine; `S_rail_human` is one of that class's six owners (`structs/anim_machine.md` §1) |
| `+0x10` | last applied animation state |
| `+0x14` | current state |
| `+0x18` | the carriage index: `railway_game_init` draws it with `rand_range(vagon count + 1)` and the seat search matches it; `railway_do_panic` panics the passengers whose index equals the carriage `railway_hit` was hit in |
| `+0x20` | animation-finished flag |
| `+0x3c` / `+0x40` | timers |
| `+0x44` | saved state while panicking |
| `+0x4c` | the inverse transform for the collision record |
| `+0x8c` | the dynamic-collision record (type `0x40`) |

State values that matter: **`0x11`** dead, **`0x12`** inactive/alighted (both excluded by
`railway_any_passenger_wants_off`), **`0x15`** and **`0x16`** the two panic states.

Passengers are solid — `rail_passenger_create_coll` publishes a `0x48`-byte
dynamic-collision record and `rail_passenger_update_coll` keeps it in step — so you can
bump into somebody standing in a tram.

`rail_passenger_is_woman` resolves the frame's **`back1`** child and reports whether the
model name's first letter is `'B'`. (The ambient pedestrian crowd reads a height scale off
the same child frame.)

### 2c. `rail_doors` — the doors (`rail_doors_`)

Two leaf banks (one per side) sharing a single angle. States: **0** closed, **1** open,
**2** closing, **3** opening. `rail_doors_tick` integrates the angle at
`dt * 0.001 * speed` and clamps at **π/2**; `rail_doors_place` builds one quaternion per
bank and stamps it onto every leaf frame with the flag rewrite
`(flags & 0xfffffecb) | 0x40000008`.

### 2d. `rail_glass` / `rail_window` — breakable windows

A window is `0x14` bytes seeded `{1, 0, -1, 0, 0}`. `rail_glass_game_init` clones the
source material per vehicle so one broken window does not crack every tram in the city.
**Two hits break a window**: `rail_window_hit` state 1 swaps in the cracked material,
state 2 calls the glass-breaking helper with the sound literal `lustr1.wav`.

### 2e. `rail_entries` — the boarding points

Entries are `0x18`-byte records (a start point, an end point and flags). There are two
banks, **inner** (`+0x270`) and **outer** (`+0x274`), each holding
`(vagon count + 1) × entry count` published records — `railway_gen_entries` allocates both.

The public accessor is `railway_get_entry_for_actor`: an actor of kind 2, 3 or `0x1b`
that is **riding a vehicle** gets an interior entry, everyone else the street side. The
four `railway_add_*_entries` / `railway_remove_*_entries` siblings publish and retract the
banks as passengers board and alight.

---

## 3. Behaviour worth knowing

* **Departures are rate-limited to one per 60 000 ms per stop.** `railway_is_on_schedule`
  keeps a file-scope `{node, last-departure}` log shared by every vehicle.
* **A tram holds back when the vehicle ahead is within 40 units.**
  `railway_find_vehicle_ahead` walks the rail graph forward and records the next vehicle
  at `+0x160`; `railway_hold_behind_vehicle_ahead` saves the current state and forces
  state `0xc`.
* **The flange squeals on a bend.** `railway_update_curve_squeal_sound` turns on sound
  slot 3 when the track bends between ~10° and ~80° within 5 units of the next node, and
  scales it by speed.
* **It does not rain inside a tram.** `railway_set_rain_dummy` moves the shared
  `g_rail_rain_dummy` volume into and out of the scene's rain-exclusion list.
* **Shooting a tram breaks its windows and kills passengers.** `railway_hit` breaks up to
  three windows, kills up to three passengers and panics the rest.
* **Everything stops for a cutscene.** `railway_ai` returns immediately while
  `g_ambient_ai_suspended` is set.

### 3a. The globals

All four file-scope vectors have an MSVC static-init/atexit group in
`[0x004d7300, 0x004d74e0)`; three carry a bit in `g_rail_static_teardown_flags`.

| global | what it is |
|---|---|
| `g_railway_list_*` | every live rail vehicle |
| `g_rail_semaphore_list_*` | the rail signals |
| `g_rail_schedule_log_*` | `{node, last-departure-ms}` pairs |
| `g_rail_clamp_candidates_*` | never filled: the file holds 11 references to its four addresses — the zero-init, the atexit free and the one reader, `railway_clamp_movement_against_traffic` — so as shipped that loop never runs |
| `g_rail_rain_dummy` | the shared rain-exclusion volume |
| `g_rail_seat_search_cursor` | the rotating seat-search cursor |
| `g_rail_passenger_anims` | the passenger animation table, 12 bytes per state |

### 3b. The three sound slots

`railway_game_init` builds sound object *i* at `+0x28c + 4i` from the `sounds.dat` id at
`+0x29c + 4i`; the ids depend on the elevated-train flag `+0x1d8`:

| slot | object | tram | elevated train | played by |
|---|---|---|---|---|
| 1 | `+0x28c` | 158 `salina_jizda` (`tram1.wav`) | 157 `nadzemka_jizda` (`railway.wav`) | the running loop: `railway_ai`, `railway_update`, `railway_set_state` |
| 2 | `+0x290` | 160 `salina_cink` (`tram2.1.wav`, `tram2.2.wav`) | 291 `nadzemka_dvere (open)` (`t-door.wav`) | `railway_ai` rebuilds and starts it in its door and departure states |
| 3 | `+0x294` | 161 `salina_zatacka` (`tram3.1.wav` .. `tram3.3.wav`) | 161, the same | `railway_update_curve_squeal_sound` |

---

## 4. Method coverage

**127/127 PC starts named.** The PS2 method accounting is in §6.

**The frame-enumeration callbacks.** Four free functions are handed to the scene's frame
enumerator (vtable `+0x34`). `railway_enumerate_scene_frames` passes two, both with filter
`0x21`: `railway_cb_enum_dummies` sorts the train model's named frames (`RAIL_COLL`,
`[DEFORM GLASS]`, and the numbered `RAIL_ENTRY` / `RAIL_DOOR` / `RAIL_ROW` tags) into a
`raildummy` vector, and `railway_cb_enum_dummies_vagons` collects each vagon's `[DEFORM GLASS]`
frames (the loop over `+0xc4`). `railway_game_init` passes `railway_cb_enum_lights` (filter
`0xffff`) and `railway_game_done` passes `railway_cb_enum_destroy`. `railway_cb_enum_dummies`
(`0x004e4c50`, 16 name arms) is reached only through that address, so it has no direct caller.
Four of its `copy_range_stride_0x10` calls are dead code: each sits behind `mov ecx,edi; sub ecx,edi;
cmp ecx,1; jae`, which subtracts a register from itself, so the "shift the tail" branch of the
inlined `vector::insert(end(), 1, x)` never runs.

---

## 5. Open items

| item | state |
|---|---|
| `RestoreWindows` | no distinct PC function. Its restore-all loop exists once on the PC, inside `rail_glass_game_done`, so a fold there is the likely explanation; not proven |
| in game | does a tram actually wait 60 s at a stop, and does the squeal fire on the sharp turns? Both are one observation each |
| in game | shoot a tram window — two hits should break it, and a third shot should kill a passenger |

---

## 6. PC vs PS2

**How the twins are established.** The PC callee sequence of `railway_ai` and the PS2 callee
sequence of `AI__9C_railwayFUi` run in step for **35 consecutive calls with zero
inversions**. That single measurement pinned `Place(i)`, `Collide`, `GetSafeSpeed`,
`DoBrousicka`, `MustStop`, `SetRLWState`, `PreduceMovement`, `ExportEntries`, the four
entry-bank siblings and both door verbs, and it positively proved five inlinings — a
statement-identical caller acting as a callee oracle at a scale no order alignment could
reach. `railway_place_on_track` then repeated it one level down: its
five-call tail is `FindBackPos, SpringProcess, FindBackPos, FindBackPos, SpringProcess`,
matching PS2 `Place(i)` exactly.

**Method accounting.** 76/76 `C_railway` PS2 methods accounted, plus `S_RailSeats` 11/11,
`S_rail_human` 10/10, `rail_doors` 7/7, `rail_glass` 3/4, `rail_window` 2/3, `rail_entries` 2/2.
The per-function PC↔PS2 twins are in the function registry, `Game-ghidra.exe.functions.md`.
What is *not* a simple 1:1 is listed here:

| PS2 method | status |
|---|---|
| `CheckDistance`, `GetCheckpoints`, `IsStopPoint`, `InRangeActivate`, `OutOfRangeDeactivate` | **inlined on PC** — proven positively, each is the only call missing at its mirrored position. Marked with `// ===== INLINED ON PC:` comments at those sites in the decomp |
| `PreduceMovementCar` | **partly inlined**: only its swept-sphere kernel stayed out of line, as `railway_sweep_sphere_along_curve`. One source function, two PC symbols ⇒ STRUCTURAL |
| `RestoreWindows` | **not located** (§5) |
| `Place(int, C_web_node*)`, `GeneratePersonTrafic`, `FreePersonTrafic`, `ReGenerate` | **no PC counterpart**: on the PS2 they are reached only from `C_rail_generator` ¹ |
| `MemWrite`, `MemRead` | **present on PC** as the shared `entity_mem_write` / `entity_mem_read`, called ×14 each from the SaveGame pair on both builds |

¹ `C_rail_generator::AI` calls `ReGenerate`, which calls `Place(node)` and `GeneratePersonTrafic`;
`C_rail_generator::GetFreeRailHuman` calls `FreePersonTrafic`. None of the four is virtual, and
`C_rail_generator` (entity kind `0x26`) exists only on the PS2 (`structs/entity.md` §7), so the
band has no caller a PC function could mirror.

**Field deltas measured** (do NOT extrapolate — this class is not a uniform shift):

| PC | PS2 | field |
|---|---|---|
| `+0x68` | `+0x70` | the actor's frame |
| `+0xc4` | `+0x148` | vagon array |
| `+0xd4` | `+0x340` | speed |
| `+0xdc` | `+0x310` | run state |
| `+0x1a8` | `+0x334` | vagon count |
| `+0x1b4` | `+0x34c` | speed scale |
| `+0x1d8` | `+0x368` | the stop-test flag |
| `+0x254`/`+0x258` | `+0x2ec`/`+0x2e8` | passenger list (PS2 stores a count, the PC an end pointer) |
| `+0x288` | `+0x31c` | boarding lock |
| `+0x28c` | `+0x16c` | first sound object |
| `+0x2ac` | `+0x330` | material id |
| `+0x2b0` | `+0x210` | the embedded driver |

`S_RailSeats` row stride is **0x38 on PC against 0x50 on PS2**, with the occupancy/owner
arrays at `+0x30`/`+0x34` against `+0x3c`/`+0x40`.

**Static class members.** Two of this class's globals are PS2 static members rather than
instance fields, which is why one object serves every vehicle:
`rain_dummy__9C_railway` (→ `g_rail_rain_dummy`) and **`salina_v_kine__9C_railway`**
(→ `g_ambient_ai_suspended`). The latter is Brno slang — *šalina* = tram, "v kině" = in
the cinema — so the flag's origin is literally "stop the trams, a cutscene is playing".
The PC name is kept because its consumer census is much wider than rail.

**Toolchain traps in this TU.** PS2 `Hit`, `Init__11S_RailSeats` and `GameInit` are
dominated by VU intrinsics and `GetCheckpoints`, so a callee-multiset comparison cannot
work on them by construction; they are graded from vtable slot, signature and
the surviving distinctive callee instead. PS2 `strcpy`×6 in `GameInit` is MSVC `string_*`
on the PC.

**Related:** `structs/entity.md` §1 (kind 8) and §3 (`C_railway` vs `C_wagon`) ·
`structs/seagull_flock.md` (the TU immediately after this one) ·
`structs/anim_machine.md` (`S_rail_human` is one of its six owners) ·
`structs/railway_switch.md` (the turnout).
