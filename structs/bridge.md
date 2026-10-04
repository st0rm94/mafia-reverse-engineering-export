# `C_bridge` — the city's two drawbridges

**Created** 2026-08-06 14:15 +0200 · **Modified** 2026-09-30 23:08 +0200

The city's two lifting road bridges: the **West Marshall Bridge** (Central Island - Little Italy)
and the **East Marshall Bridge** (Central Island - Downtown). Each is one `C_bridge` entity that
drives both leaves of its deck: the frame it is bound to and a second leaf it finds at `Init`
time. The two instances share a global bank (§2), and the East Marshall instance also runs the
steamboats whose approach makes either bridge rise (§2a, §2b).

| | |
|---|---|
| PC prefix | **`bridge_`** |
| PC kind (`entity+0x10`) | **`0x14`** (`structs/entity.md`) |
| PC vtable | `.rdata 0x0063bb40` (39 slots) |
| PC translation unit | `[0x0045f970, 0x00462a90)` — 20 function starts; CRT static-init/atexit helpers on both sides |
| Base class | **`C_actor`** (`bridge_ctor` → `actor_ctor`, `bridge_init` → `actor_init`, `bridge_game_init` → `actor_game_init`, `bridge_game_done` → `actor_game_done`) |
| PS2 class | `C_bridge`, 16 methods, all accounted (§3) |

One consumer outside the class: `human_find_blocking_drawbridge` refuses to let the player leave a
car when a `C_bridge` in state 3-5 is within 100 units — exactly the **raising / up / lowering**
states of §1b.

## 1. Field layout

Confidence tags: **[H]** byte-proven from a writer *and* a reader · **[M]** one side traced ·
**[U]** unknown.

Every field in the `+0x13c`..`+0x190` band has **four** independent witnesses — `bridge_ctor` seeds
it, `bridge_game_init` re-seeds or clamps it, `bridge_ai` reads it every tick, and the PS2 twin
writes the same value at the same relative offset (§6b) — which is why the whole band is [H].

```c
typedef struct C_bridge {                 /* extends C_actor */
    /* +0x00  */ void      *vptr;             // 0x0063bb40
    /* +0x10  */ int        kind;             // [H] = 0x14
    /* +0x68  */ I3D_frame *frame;            // [H] C_actor field — the first leaf's frame
    /* +0x74  */ S_BridgeCol *coll_a_begin;   // [H] deck collision panels, stride 0x84
    /* +0x78  */ S_BridgeCol *coll_a_end;     // [H]
    /* +0x7c  */ S_BridgeCol *coll_a_cap;     // [H]
    /* +0x84  */ S_BridgeCol *coll_b_begin;   // [H] second panel set (the second leaf)
    /* +0x88  */ S_BridgeCol *coll_b_end;     // [H]
    /* +0x8c  */ S_BridgeCol *coll_b_cap;     // [H]
    /* +0x90  */ I3D_frame *paired_frame;     // [H] the second leaf, found by bridge_init
    /* +0x94  */ I3D_sound *machinery_sound;  // [H] created by bridge_game_init
    /* +0x98  */ I3D_sound *siren_sound;      // [H] created by bridge_game_init
    /* +0xb4  */ tDynamicCollObject *dyn_coll;// [H] inserted while raised, erased when down
    /* +0x138 */ int        act_state;        // [H] C_actor act state: 1 while cycling, 7 idle
    /* +0x13c */ char       frozen;           // [H] set by bridge_shut_down; forces every
                                              //     bridge_ai state to take its exit branch
    /* +0x140 */ int        bridge_index;     // [H] 0 or 1 — indexes the global bank (§2)
    /* +0x144 */ int        state;            // [H] the 6-state cycle, §1b
    /* +0x148 */ float      angle;            // [H] current deck angle, radians
    /* +0x14c */ float      angle_applied;    // [H] last angle written to the frame
    /* +0x150 */ int        phase_timer_ms;   // [H] counts down within the current state
    /* +0x154 */ int        blink_timer_ms;   // [H] warning-light blink, re-armed at 600 ms
    /* +0x158 */ char       blink_on;         // [H] toggled by bridge_update
    /* +0x15c */ uint       hold_ms;          // [H] bridge_ai returns while non-zero
    /* +0x160 */ int        idle_period_ms;   // [H] default 120000
    /* +0x164 */ int        warn_ms;          // [H] default   4000
    /* +0x168 */ int        raise_ms;         // [H] default  30000
    /* +0x16c */ int        up_dwell_ms;      // [H] default  10000
    /* +0x170 */ int        lower_ms;         // [H] default  20000
    /* +0x174 */ char       reverse_dir;      // [H] when clear, bridge_ai negates the angle
    /* +0x178 */ float      angle_max;        // [H] 0x3f5f66f3 = 0.8727 rad = 50 degrees
    /* +0x17c */ int        snd_wait_clear;   // [H] 0x71, played entering state 2
    /* +0x180 */ int        snd_raising;      // [H] 0x72, played entering state 3
    /* +0x184 */ int        snd_up;           // [H] 0x73, played entering state 4
    /* +0x188 */ int        snd_lowering;     // [H] 0x74, played entering state 5
    /* +0x18c */ int        snd_machinery_go; // [H] 0xb6, played on the machinery emitter
    /* +0x190 */ int        snd_siren;        // [H] 0xb5, played on the siren emitter
} C_bridge;
```

**The five durations are the whole behaviour.** `bridge_ctor` seeds them and `bridge_game_init`
clamps four of them to a minimum of 1 ms, so a zero left in the mission data cannot stall the cycle
— it degrades to an instant transition instead.

### 1a. `S_BridgeCol` — the deck collision panels

Element **stride 0x84** (132 bytes). Two vectors, `+0x74` and `+0x84`, filled by
`bridge_game_init` from `collision_find_hits_for_frame` — one set per leaf — and
re-transformed into the deck's current frame space every tick by `bridge_update`. That is how the
road surface keeps colliding correctly while it tilts. `bridge_game_init` also builds the child
frames, calls `S_matrix::Inverse`, and creates a midpoint proxy of bounding radius 20.0.

The element TYPE is not derivable from the PC decomp; it comes from the PS2 template argument
(§6c).

### 1b. The state machine (`+0x144`)

| state | name | leaves when | on leaving |
|---|---|---|---|
| 0 | idle | `g_bridge_raise_request_mask[idx]` becomes non-zero (a steamboat is approaching — §2a) | arm `+0x150` = `warn_ms`, blink `+0x154` = 600 ms, play `snd_siren` on the siren emitter, act state 1 |
| 1 | warn | timer expires **and** `traffic_car_get_bridge_car_count` < 1 | play `snd_machinery_go`; if cars are still on the deck, re-check in 1000 ms |
| 2 | wait-for-clear | the machinery emitter reports idle | arm the raise ramp over `raise_ms`, play `snd_wait_clear`, `collision_dyn_insert(+0xb4)` |
| 3 | raising | timer expires | angle ramps 0 → `angle_max`; then hold for `up_dwell_ms`, play `snd_raising` |
| 4 | up | timer expires **and** `g_bridge_raise_request_mask[idx]` has cleared | publish `is_up[idx] = 1`; ramp down over `lower_ms`, play `snd_up` |
| 5 | lowering | timer expires | angle ramps back to 0; return to state 0 after `idle_period_ms`, silence the siren, act state 7, `collision_dyn_erase(+0xb4)`, play `snd_lowering` |

`+0x13c` (`frozen`) short-circuits every one of these — each state's exit condition is
`timer expired **or** frozen`.

**The dynamic collision object is only registered while the bridge is up** (inserted entering state
3, erased leaving state 5). While the deck is flat it is ordinary static world geometry.

`traffic_car_get_bridge_car_count` (`0x00506760`), state 1's gate, is a `C_traffic_car` static, not
a `C_bridge` member.

## 2. The shared global bank

Two `C_bridge` instances exist, one per drawbridge, and they coordinate through fixed globals
rather than pointers to each other. `bridge_init` finds each bridge's second leaf by incrementing
the last character of its frame name and assigns the index: **frame name starting with `'L'` ⇒
index 0** (the West Marshall Bridge, leaves `LLsklap01` / `LLsklap02`), **otherwise index 1**
(the East Marshall Bridge, `sklapx01` / `sklapx02`); the index-1 instance clears the whole bank.
Which frame is which bridge is the user's identification; the steamboat routes of §2b agree, as
index 0 is requested only by routes 0 and 1 and index 1 only by routes 2 and 3, at crossing
points about 870 units apart. Every named global below is used only by `bridge_*` functions.

| address | name | shape | meaning |
|---|---|---|---|
| `0x0064b5ec` | `g_bridge_first` | byte | raised by `bridge_dtor` and `bridge_game_done` [M] |
| `0x00655744` | `g_bridge_is_up` | int[2] | published by `bridge_ai` — 1 in state 4, 0 otherwise — and read by `bridge_update` as a gate on the approach logic [H] |
| `0x0065574c` | `g_bridge_raise_request_mask` | int[2] | **a bitmask of requesting steamboats, not a flag** ¹ [H] |
| `0x00655758` | `g_steamboat_slots` | record[2], stride 0x24 | the steamboat slot array — §2a [H] |
| `0x006557a0` | `g_steamboat_segment_time` | float[28] | per-segment traversal time, indexed `route*7 + segment` [H] |
| `0x00655810` | `g_steamboat_nodes` | node[4][7], stride 0x18 | the route node table loaded by `bridge_load_data` — §4 [H] |
| `0x00655730` | `g_steamboat_shared_section_claim` | int[3] | `[0]` = the boat (index + 1) in the channel under Terranova Bridge; `[1]` and `[2]` are only ever cleared (§2b) [H] |
| `0x0065573c` | `g_bridge_passage_claim` | int[2] | the boat (index + 1) in the channel under drawbridge `i` (§2b) [H] |

¹ `bridge_update` clears it and ORs `1 << steamboat_index` per approaching boat; `bridge_ai` state 0
tests `!= 0`.

`0x006557a0 + 28*4 == 0x00655810` exactly — the segment-time table and the node table are
contiguous, and that arithmetic fixes both element counts.

### 2a. The steamboat slot record (stride `0x24`, 2 entries)

| off | name | meaning |
|---|---|---|
| `+0x00` | `g_steamboat_slots` | **route id**; seeded `0xffff` by `bridge_init`; the `switch` subject in `bridge_update`, and `id * 7` indexes `g_steamboat_nodes` — that multiply proves the 4-routes-of-7-nodes shape |
| `+0x04` | `g_steamboat_slot_segment` | current segment index; wraps at `> 5` (6 segments over 7 nodes) |
| `+0x08` | `g_steamboat_slot_segment_prev` | written from `+0x04` each tick |
| `+0x0c` | `g_steamboat_slot_t` | float, 0..1 along the current segment |
| `+0x10` | `g_steamboat_slot_frame` | the boat's `I3D_frame`; parent of the figures, released by `bridge_game_done` |
| `+0x14` | `g_steamboat_slot_speed_scale` | float divisor feeding the `t` advance |
| `+0x18` | `g_steamboat_slot_figure_a` | `Tommy.i3d` / `Leonardo` — §4a |
| `+0x1c` | `g_steamboat_slot_figure_b` | `czena06.i3d` / `Kate` — §4a |
| `+0x20` | `g_steamboat_slot_sound` | the boat's sound object: `bridge_game_init` creates it (`sound_table_construct_sound(..., 0x146)`, range 20..50); `bridge_update` drives it |

**The writer of the raise request is `bridge_update`.** Its second half — gated `+0x140 != 0`, so
only the index-1 bridge runs it — **is the steamboat simulation.** It walks the slot array, advances
each boat along its route, and from the boat's route id and position sets the request bits:

```
snapshot both mask elements · clear both · per boat:
    bVar11 = (byte)slot_index
    g_bridge_raise_request_mask |= 1 << (bVar11 & 0x1f)
```

So **the bridge raises itself because a boat is approaching**, and the whole mechanism is inside this
translation unit.

### 2b. The raise windows and the channel claims

A boat's position is `segment + t` along its route (7 nodes, so 0.0 .. 6.0). Each tick
`bridge_update` first drops every claim the boat holds, then, per route:

| route | raises | raise-request window | `g_bridge_passage_claim` window | Terranova claim window |
|---|---|---|---|---|
| 0 | index 0 (West Marshall) | 0.72 .. 1.3 | `[0]` 0.6 .. 2.0 | 4.0 .. 5.0 |
| 1 | index 0 (West Marshall) | 4.0 .. 5.2 | `[0]` 4.0 .. 5.3 | - |
| 2 | index 1 (East Marshall) | 4.4 .. 5.2 | `[1]` 4.2 .. 5.4 | 0.5 .. 1.5 |
| 3 | index 1 (East Marshall) | 0.3 .. 1.3 | `[1]` 0.3 .. 2.0 | 4.0 .. 5.5 |

A claim is taken only when it is free; a boat inside the first part of a window whose claim
another boat holds, or whose drawbridge has not published `g_bridge_is_up` yet near the crossing
(0.92 .. 1.0, 4.79 .. 5.0, 4.85 .. 5.0, 0.75 .. 1.0 for routes 0..3), is held back. So
`g_bridge_passage_claim[i]` keeps the channel under drawbridge `i` to one boat, and
`g_steamboat_shared_section_claim[0]` does the same for a third channel shared by routes 0, 2
and 3. Mapped through the node positions of `tables\parnik.def`, that third window lies north
of both drawbridges (x ≈ -1200, z ≈ 450 .. 1000), which is the channel under **Terranova
Bridge** (user identification). Route 1 passes the same channel between its nodes 1 and 2 and
takes no claim.

## 3. Method coverage — 16/16 PS2 methods accounted; 20/20 PC starts accounted

Ordered by PC VA. Every PC function in the translation unit gets a row, including those with no PS2
twin (`-` in the PS2 columns); PS2 methods with no PC function of their own follow at the bottom.

| PC VA | PC name | PS2 twin | slot | status |
|---|---|---|---|---|
| `0x0045f970` | `stub_noop_cdecl_0045f970` | - | - | shared stub, not a class member |
| `0x0045f980` | `bridge_ctor` | `__ct__8C_bridgeFv` | - | CONFIRMED |
| `0x0045fa70` | `bridge_scalar_deleting_dtor` | - | 16 | STRUCTURAL ¹ |
| `0x0045fa90` | `bridge_dtor` | `__dt__8C_bridgeFv` | - | CONFIRMED |
| `0x0045fae0` | `bridge_init` | `Init__8C_bridgeFP9I3D_frame` | 18 | CONFIRMED |
| `0x0045fc20` | `bridge_ai` | `AI__8C_bridgeFUi` | 13 | CONFIRMED |
| `0x004601b0` | `bridge_update` | `Update__8C_bridgeFUi` | 14 | CONFIRMED |
| `0x00461180` | `stub_noop_args4_00461180` | `NetDirect__8C_bridgeFUi` | 15 | folded stub (§5) |
| `0x00461190` | `bridge_game_init` | `GameInit__8C_bridgeFv` | 25 | CONFIRMED |
| `0x00462280` | `bridge_game_done` | `GameDone__8C_bridgeFv` | 26 | CONFIRMED |
| `0x00462330` | `stub_rettrue_args4_00462330` | `SaveData__8C_bridgeFP7C_chunk`, `SaveGameLoad__8C_bridgeFPPv` | 19, 24 | folded stub (§5) |
| `0x00462340` | `bridge_load_data` | `LoadData__8C_bridgeFP7C_chunk` | 20 | CONFIRMED |
| `0x00462420` | `stub_ret0_cdecl_00462420` | `SaveGameGetSize__8C_bridgeFv` | 22 | folded stub (§5) |
| `0x00462430` | `bridge_save_game_save` | `SaveGameSave__8C_bridgeFPPv` | 23 | CONFIRMED |
| `0x00462450` | `bridge_shut_down` | `ShutDown__8C_bridgeFb` | - | CONFIRMED ² |
| `0x00462480` | `bridge_collision` | `Collision__8C_bridgeFP7C_actor…P9I3D_frame` | 30 | CONFIRMED |
| `0x004624e0` | `bridge_set_steamboat_figures` | - | - | STRUCTURAL ³ |
| `0x004626b0` | `string_assign_cstr` | - | - | FOREIGN — a `C_string` method emitted in this TU |
| `0x00462730` | `bridge_col_vector_insert_n` | - | - | STRUCTURAL (§6c) |
| `0x00462a60` | `bridge_col_vector_copy` | - | - | STRUCTURAL (§6c) |
| - | (`DuplicateFrom`) | `DuplicateFrom__8C_bridgeFP7C_actor` | 21 | folded stub (§5) ⁴ |

¹ MSVC-only construct. ² Not a vtable slot on either build. ³ A class member with no PS2 twin;
PC-only because the PS2 has no cheat system (§6a). ⁴ `stub_noop_args4_004776f0`, outside this TU.

## 4. The steamboat rides in this class's translation unit

`bridge_load_data` does not load bridge data. It loads **`tables\parnik.def`** — a
`4 routes x 7 nodes x 0x18 bytes` table at `0x00655810`, each node a position vec3 followed by a
direction vec3, defaulting to `{(0,0,0), (1,0,0)}` when the file is absent.

The node element type is `G_ParnikUzel` (*parník uzel* = steamboat node), a record type declared in
the same source file as `C_bridge` (§6a). A class defined in another class's source file links with
that file — the same mechanism that puts `ai_loc` / `ai_blob` / `ai_blob_log` next to `C_human`.

### 4a. `bridge_set_steamboat_figures` — the PARNIK cheat's figures

The PARNIK cheat (`game_cheat_check` case 0xb) decorates the steamboats with the Titanic easter egg,
and it does so **through this class**.

**Receiver, byte-proven.** `game_cheat_check` walks the `C_game` entity vector and filters on the
entity kind before calling:

```
mov ecx, [edi+4]                 ; entity vector begin
mov ecx, [ecx + esi*4]           ; element
cmp dword ptr [ecx + 0x10], 0x14 ; kind == C_bridge  (structs/entity.md)
jne skip
push edx                         ; the cheat's toggle bit
call bridge_set_steamboat_figures
```

A kind gate is the caller stating the receiver class outright — stronger evidence than an ECX
fingerprint. The body agrees independently: it opens with `if (+0x140 != 0)`, the `bridge_index`
test, so only the index-1 bridge acts (the same index-1-owns-the-shared-state convention
`bridge_init` establishes).

**What it does.** For each of the 2 records in the global bank, with `enable != 0` it loads
`Tommy.i3d` textured **`Leonardo`** into `record+0x18` and `czena06.i3d` textured **`Kate`** into
`record+0x1c`, parents both to the steamboat frame at `record+0x10`, and places them at local
`(0, 4.7, 37.0)` and `(0, 4.7, 37.4)` — the bow, one just behind the other. With `enable == 0` it
releases both and zeroes the slots. The retextured Leonardo (DiCaprio) and Kate (Winslet) in the bow
pose parody the Titanic "I'm flying" scene; user-confirmed in game.

This function also fixes the bank's field map: §2a's `+0x10`/`+0x18`/`+0x1c` triple is exactly what
`bridge_game_done` releases, and this function fills all three.

## 5. Five methods are folded onto shared stubs

The `NetDirect`, `SaveData`, `SaveGameGetSize`, `SaveGameLoad` and `DuplicateFrom` vtable slots all
point at shared behaviour-free stubs owned by no class (`stub_noop_args4_00461180` has 30 vtable
users). This is read at the vtable slots themselves. A static drawbridge has no per-instance state
to replicate. The PS2 side of the five is in §6f.

## 6. PC vs PS2

### 6a. Link order and the source file

PS2 emits classes alphabetically by source file, and the sequence around this class is
`C_bottle → C_box_new → C_bridge → G_ParnikUzel → C_car`. The PC `.rdata` vtable sequence carries
the same order, which places this vtable's owner before any body is read. A PS2 `__sinit_` symbol
confirms `C_bridge.cpp` is compiled into the PS2 build.

`G_ParnikUzel` immediately follows `C_bridge` and has only an 8-byte `{ctor}` — a plain record type
declared inside `C_bridge.cpp`, which is why the steamboat node table loads from a `C_bridge` method
on both builds (§4).

`bridge_set_steamboat_figures` has no PS2 twin: the PS2 build has no cheat system, and nothing in its
`C_bridge.cpp` TU corresponds. It is therefore graded STRUCTURAL: with no PS2 body in existence,
CONFIRMED would assert a body agreement that cannot exist.

### 6b. The field delta is a flat +8

Across the whole `+0x13c`..`+0x190` band the PS2 offset is the PC offset **+ 8**:

| what | PC | PS2 |
|---|---|---|
| `frozen` | +0x13c | +0x144 |
| `bridge_index` | +0x140 | +0x148 |
| `state` | +0x144 | +0x14c |
| `phase_timer_ms` | +0x150 | +0x158 |
| `idle_period_ms` | +0x160 | +0x168 |
| `angle_max` | +0x178 | +0x180 |

Twelve twins diffed independently all land on that same delta, so they corroborate each other.

**Do not rebase other offsets off it.** In the low band the bases differ: PC `+0x74` (collision
vector A) ↔ PS2 `+0x80`; PC `+0x90` (paired frame) ↔ PS2 `+0x98`; PC `+0xb4` (dyn coll) ↔ PS2
`+0xbc`.

### 6c. What `S_BridgeCol` is

The PC decomp gives a stride and an access pattern, never an element type. The PS2 symbol
`reserve__Q23gif34vector<Q28C_bridge11S_BridgeCol,8>Fi` gives it directly:
**`gif::vector<C_bridge::S_BridgeCol, 8>`**. Its body copies 9 quadwords = `0x90` per element against
the PC's `0x84` — the usual `S_vector` 16-vs-12 padding, ratio 0.92.

MSVC and GCC out-lined **different members** of that template: the PS2 has `reserve`, the PC has
`_Insert_n` and `std::copy`. That is why `bridge_col_vector_insert_n` / `bridge_col_vector_copy` are
graded STRUCTURAL — there is no counterpart body to diff, only a confirmed element type plus a
callsite census (2/2 each, both inside `bridge_game_init`, found by a byte scan of `.text` for `E8`
calls since Ghidra does not render them textually).

### 6d. PS2 names this class's globals in the clear

`globalFirstBridge`, `bridgeIsUp`, `bridgeNeedUp`, `bridgeObsazen`, `mostObsazen` (*obsazen* =
occupied, *most* = bridge), `parnik`, `uzly` (= nodes). Those symbols identify the PC global bank in
§2: `bridgeObsazen` is `g_bridge_passage_claim` and `mostObsazen` is
`g_steamboat_shared_section_claim`, and the PS2 `Update__8C_bridgeFUi` uses the same route
windows as §2b, value for value.

### 6e. Cross-build evidence that settles PC-side questions

| PC question | settled by |
|---|---|
| what the `0x84`-stride vector holds | the `gif::vector<C_bridge::S_BridgeCol,8>` template arg |
| what the `0x00655744` / `0x0065574c` arrays are | PS2 `bridgeIsUp` / `bridgeNeedUp`, indexed identically |
| what `bridge_load_data` loads | PS2's destination array is named `uzly`, and `G_ParnikUzel` links right after this class |
| that `bridge_ai` state 1's gate is a traffic query | PS2 calls `GetBridgeCarCount__13C_traffic_car` (twin of `traffic_car_get_bridge_car_count`, CONFIRMED) where the PC calls `traffic_car_get_bridge_car_count` |

### 6f. The five folded methods (§5)

All five have **8-byte (empty) PS2 bodies**. The PC fold of §5 is read at the vtable slots, not
inferred from those empty bodies — an empty-looking slot can hold real code in another class
(`C_PhThing`'s slot 15 holds 1696 bytes of dead-reckoning code). A static drawbridge has no
per-instance state to replicate, so an emptied-replication reading does not apply here.
