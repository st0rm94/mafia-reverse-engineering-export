# `C_actor` — the root actor base class

**Created** 2026-07-12 09:41 +0200 · **Modified** 2026-10-12 21:02 +0200

The root of the game's actor hierarchy: `C_actor` ← `C_human` ← { `C_entity`, `C_player` }, with
`C_car` / `C_traffic_car` / `C_railway` / `C_wagon`, spike-strips and props deriving from it as well.
It owns the transform/scene-graph frame, global-actor registry membership, world position and
direction, the per-frame `Tick`, the collision broad-phase entry points and the SaveGame plumbing.

| | |
|---|---|
| PC prefix | **`actor_`** (over-applied — see §3) |
| PC kind (`+0x10`) | **`1`**, stamped by `actor_ctor` |
| PC vtable | `g_actor_vt` — installed by `actor_ctor`, restored by `actor_dtor` |
| PC base class | **`C_actor_track`**, vtable slots 0–12 (§1a) — `C_actor`'s own 27 virtuals are slots 13–38 |
| PS2 class | `C_actor` (mangled tag `7C_actor`), **47 methods**: 30 with a PC twin, 17 base stubs (§4) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout (relative to the `C_actor` base)

Only the `C_actor` **base** fields are listed; subclasses extend the object far beyond these and
their fields belong in the respective class doc. Offsets are PC (`Game.exe`, image base
`0x00400000`). PS2 offsets are given by body-diffed ROLE, never by rebasing (§6).

```c
typedef struct C_actor {                    // partial; PC offsets
    /* +0x00 */ void     *vtable;           // [H] = g_actor_vt; ctor sets, dtor restores.
    /* +0x04 */ uint8_t   track_flags;      // [H] C_actor_track mode flags (§1a). actor_ctor = 0.
                                            //     actor_tick skips the Update/AI dispatch when
                                            //     bit 0 or bit 2 is set.
    /* +0x08 */ void     *track;            // [H] the C_actor_track keyframe track. actor_ctor = 0.
    /* +0x0c */ uint16_t  ref_count;        // [H] REFERENCE COUNT; ctor = 1. actor_release
                                            //     pre-decrements it and, at zero, calls the
                                            //     object's scalar deleting destructor through
                                            //     vtable slot 16 (+0x40).
    /* +0x10 */ int32_t   kind;             // [H] 1 for a bare C_actor; every subclass ctor
                                            //     overwrites it (human 3, player 2, car 4,
                                            //     entity 0x1b). The engine's stand-in for RTTI.
    /* +0x14 */ int32_t   class_id;         // [H] branch tag: 1 base, 2 C_human, 3 C_car_ex.
    /* +0x18 */ uint32_t  net_guid;         // [H] network GUID; its top byte is the owning peer's
                                            //     slot (§5). Restored by actor_save_game_load_guid.
    /* +0x1c */ int32_t   act_state;        // [H] 0 active, 1 inactive, 2 switched off (§1d).
    /* +0x20 */ uint8_t   ai_driven;        // [H] actor_tick's per-frame slot selector: 0 selects
                                            //     Update, non-zero selects AI. ctor = 0; the
                                            //     network layer writes it too (§5).
    /* +0x24 */ float     cached_world_pos[3]; // [H] cached world position x/y/z; the fast path
                                            //     against recomputing the frame matrix.
    /* +0x30 */ float     cached_forward[3];   // [H] the frame's forward row, cached.
    /* +0x3c */ float     position[3];      // [H] placement position; actor_reset_placement
                                            //     pushes it into the frame's local matrix.
    /* +0x48 */ float     rotation[4];      // [H] placement quaternion, normalized on push.
    /* +0x58 */ void     *frame_parent;     // [H] saved frame parent (placement round trip).
    /* +0x5c */ uint8_t   frame_visible;    // [H] saved frame visibility byte (same round trip).
    /* +0x5d */ uint8_t   alive;            // [H] non-zero = alive.
    /* +0x5e */ uint8_t   death_processed;  // [H] the once-only death latch (§1c).
    /* +0x5f */ uint8_t   in_camera_sector; // [H] recomputed every tick; 1 = same visibility cell
                                            //     as the camera (§1c).
    /* +0x60 */ void     *pending_track_event; // [H] the recorder event record queued for the
                                            //     next key (§1b). actor_ctor = 0.
    /* +0x64 */ uint8_t   owns_frame_owner_block; // [H] 1 = actor_game_init allocated the frame's
                                            //     owner block and actor_game_done must free it.
    /* +0x65 */ uint8_t   save_excluded;    // [H] 1 = the savegame skips this actor (runtime
                                            //     objects); actor_game_init sets kind == 1.
    /* +0x66 */ uint8_t   scene_placed;     // [H] 1 = placed by the scene .bin loader.
    /* +0x68 */ I3D_frame *frame;           // [H] scene-graph frame, bound by actor_init. World pos is
                                            //     the frame matrix +0x40 after the lazy
                                            //     UpdateWMatrixProc, gated on the frame+0xac
                                            //     dirty bit 0x20.
    /* +0x6c */ uint8_t   owns_frame;       // [H] 1 = actor_dtor releases the frame.
} C_actor;                                  // partial
```

Global side-state (not per-instance): `g_scene_ref_count`, bumped by the ctor and decremented by the
dtor, and the two file-scope vectors of §2.

### 1a. The PC base class `C_actor_track` — vtable slots 0–12

On the PC, `C_actor`'s own virtuals **do not start at slot 0**. Slots **0–12** of every
`C_actor`-family vtable hold a 13-method interface belonging to a base class, `C_actor_track`
(`structs/actor_track.md`): the per-object client of the `C_recorder` keyframe-track system
(`structs/recorder.md`). `C_actor`'s own 27 virtuals occupy slots **13–38**, so

> **PC slot = PS2 vtable index + 13**

anchored by eight confirmed twins in §4 (`Release` 17, `Init` 18, the `SaveGame` trio 22/23/24,
`GameInit` 25, `GameDone` 26, `GetActorSpeed` 28, `GetBoundPoints2D` 34). Slots 13/14/15 are
`C_actor`'s own `AI` / `Update` / `NetDirect`, all ICF-folded onto the single stub
`stub_noop_args4_004776f0`. `C_actor` is therefore the root of the *game* hierarchy and of the kind
enum, but not of the vtable.

The base is **data-free as far as the PC is concerned**: `actor_ctor` writes the derived vptr at
`+0x00` and calls no base constructor, because there is none to call. It does own the fields the
interface touches — `+0x04` mode flags, `+0x08` track, `+0x60` pending event record, `+0x68`
`I3D_frame*` (`structs/actor_track.md` §3).

**The whole subsystem is PC-only** (§6).

**The consequence for field mapping.** The PC↔PS2 deltas in this family are not constant, so fields
are mapped by ROLE, never by rebasing, in this class and in every class derived from it (§6).

### 1b. Field evidence — base

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `actor_ctor`, `actor_dtor` | - | `g_actor_vt`; read through virtual dispatch |
| +0x04 | track flags | H | `actor_ctor` (=0) | `actor_tick` | bits 0 and 2 suppress the per-frame Update/AI dispatch; owned by `C_actor_track` (§1a) |
| +0x08 | track pointer | H | `actor_ctor` (=0) | the slot 0–12 interface | the `C_recorder` keyframe track (§1a) |
| +0x0c | ref_count | H | `actor_ctor` (=1) | `actor_release` | 16-bit. `actor_release` pre-decrements and, at zero, invokes vtable slot 16 (`+0x40`) — the scalar deleting destructor — with the delete flag |
| +0x10 | kind | H | `actor_ctor`, `human_ctor`, `entity_ctor`, `player_ctor` | `folded_get_dword_0x10` | 1 actor, 2 player, 3 human, 4 car, 0x13 `C_don_ocas` (the ambient-crowd stand-in actor), 0x14 `C_bridge`, 0x1b entity — `structs/entity.md` §1. This is the engine's stand-in for RTTI, the build carrying no RTTI at all, which is why the reader has 364 references. The reader symbol is ICF-folded and also serves `C_mission` |
| +0x14 | class id | H | `actor_ctor` (=1), `human_ctor` (=2), `car_extended_ctor` (=3) | `actor_get_class_id` | the dword immediately after the kind: a branch tag with exactly three values — **1** every class that does not restamp it, **2** `C_human` (inherited by `C_entity` and `C_player`), **3** `C_car_ex`. The three ctor stores are every store at this offset in the binary's constructors. Its one reader tests `kind == 4 && +0x14 == 3` to select a truck |
| +0x18 | net GUID | H | `actor_init` (`++C_mission+0x08`, `structs/mission.md` §3), `actor_apply_net_guid_table`, `actor_save_game_load_guid` | `actor_apply_net_guid_table`, the event-message layer | `actor_init` draws it from a per-mission counter; top byte = owning peer slot (§5) |
| +0x1c | act_state | H | `actor_init` (=0), `actor_game_init` (=0), `actor_set_act_state`, `actor_rebirth` (=0), `actor_save_game_load` | `actor_rebirth`, the tag-3 savegame record | the three-value enum of §1d |
| +0x20 | AI-driven selector | H | `actor_ctor` (=0), `actor_apply_net_guid_table` | `actor_tick` | `actor_tick` dispatches the Update slot when this byte is zero and the AI slot otherwise; the network layer sets it for every actor this peer does not own (§5) |
| +0x24..+0x2c | cached_world_pos | H | `actor_game_init`, `car_update`, `actor_set_cached_world_pos` | `actor_get_cached_world_pos`, `actor_get_world_position` (when the actor has no frame), `garage_game_done` | the fast path against the frame recompute. `car_update` refreshes it on every tick of a locally simulated car (`+0x20 == 0`) from the body position `car+0x40c`, so a car's copy is current. `actor_set_cached_world_pos` is an out-of-line inline member hosted by the `C_program` translation unit (`structs/program.md` §6); its three callsites all take the receiver out of the script VM's actor register file |
| +0x30..+0x38 | cached forward | H | `actor_game_init`, `actor_set_direction` | `actor_get_direction` | the frame's forward row, cached. The `actor_setdir` arm normalises a direction, stores it here, reads it straight back through `actor_get_direction` and hands it to the frame's world-forward virtual |
| +0x3c / +0x48 | placement position / quaternion | H | `actor_internal_placement_reset` | `actor_reset_placement` | the placement round trip: `actor_reset_placement` pushes them into the frame's local matrix with `S_quat::Normalize` and the dirty masks; `actor_internal_placement_reset` reads them back |
| +0x58 / +0x5c | saved frame parent / visibility byte | H | `actor_reset_placement` | `actor_reset_placement` | saved and read back by the same body (the placement round trip) |
| +0x5d | alive flag | H | set (=1): the `C_actor` init, `human_activate`; cleared: seven functions; restored: `actor_save_game_load` | `actor_is_alive`, `entity_is_tickable`, `entity_script_tick`, `entity_suspend_ai_and_hide`, `actor_tick` | non-zero = alive ¹ |
| +0x5e | death-processed latch | H | set (=1): `actor_tick`, `human_death`, `human_hit`, `car_explosion`; cleared (=0): `actor_game_init`, `human_activate`, `car_game_init`, `car_reset`; restored: `actor_save_game_load` | `actor_is_death_processed`, `police_manager_count_officers_alive_for_target` | the death has already been handled, so the teardown runs exactly once ² |
| +0x5f | in-camera-sector byte | H | `actor_tick` | `actor_is_in_camera_sector`, `car_update`, `car_extended_update` | recomputed every tick (§1c) |
| +0x60 | pending track event | H | `actor_ctor` (=0), `human_track_queue_anim_event` (allocates the record once, fills it, stores the pointer) | `human_track_record_key` (encodes the record into the next key and clears its byte 0) | a `0x48`-byte record: byte 0 pending flag, `+0x04` value, `+0x08` target frame name, `+0x2c` position, `+0x38` quaternion; filled only while the track records (`+0x04` bit `0x2`). Measured by a census of `this+0x60` over the methods of every `C_actor`-derived class |
| +0x64 | owns the frame's owner block | H | `actor_game_init` (=1) | `actor_game_done` | the allocate/free pair across GameInit and GameDone |
| +0x65 | save_excluded | H | set (=1): `actor_game_init` (`kind == 1`), the `game_init` of every runtime class ³, `car_explosion`, the police-car pool and pursuit-car spawners, `game_load_differences`; cleared (=0): `detector_set_signal`, `detector_save_game_load`, `human_use_actor` | `entity_save_logs`, `entity_save_vis_logs`, `entity_save_sensors`, `entity_get_logs_size`, `pursuit_event_save_game_get_size`, `pursuit_event_save_game_save`, `loadsave_savegame_save` | 1 = the savegame skips this actor ³ |
| +0x66 | scene_placed | H | `actor_ctor` (=0), `actor_mark_scene_placed` (=1) | `game_done` | 1 = a FILE-PLACED actor, not one the script, the traffic generator or the pedestrian crowd created ⁴ |
| +0x68 | frame | H | `actor_ctor` (=0), `actor_init` (binds the frame argument, refusing one another actor owns); released and cleared by `human_game_done`, `car_game_done`, `dropout_game_done`; rebound by `box_init`, `dropout_init` | `folded_get_dword_0x68`, `actor_get_world_position`, `actor_get_forward_normalized` | the transform frame ⁵ |
| +0x6c | owns-its-frame flag | H | `actor_init` (=0), `actor_mark_created_at_runtime` (=1) | `actor_dtor` | `actor_dtor` releases the frame through its vtable slot 0 only when this byte is set and the frame is non-null. Set on a freshly duplicated actor whose slot-18 `Init` returned true, immediately before `game_add_temporary_actor`. The complement of `+0x66` |

¹ **`+0x5d`** — the seven clears are `human_force_death`, `human_hit`, `human_hit_in_car`,
`human_movement`, `human_throw_from_car_truck`, `human_die_in_exploding_car` and `actor_tick`; the
`C_actor` init writes `+0x5e=0; +0x5d=1; +0x5f=0`. Two of the clears sit immediately after
`*(this+0x644) = 0`, i.e. right after `cur_health` is zeroed. The 11 writers were found by a byte
scan of `.text` for `C6 /0` and `88 /r` at displacement `0x5d`; a decomp text search for
`+ 0x5d) =` returns zero.
² **`+0x5e`** — not a second alive flag. `actor_tick` sets it as
`if (+0x5d) skip; if (+0x5e) skip; +0x5e = 1`; `car_explosion` sets it on itself and on the `C_car`
it constructs. The writer list is closed by a disassembly of every function for byte stores at
displacement `0x5e`; the two other hits, in `car_pack_save_record` and (as a dword at `+0x5c`)
`car_save_game_save`, write save records, not actors.
³ **`+0x65`** — the runtime classes whose `game_init` sets it: `actor_shot`, `clock`, `detector`,
`dropout`, `spike_strip`, `bendable_plant`, `pedestrian_crowd`, `public_physics`, `racecam`,
`seagull_flock`, `smoke_trail`, `traffic_car`; `car_explosion` sets it on the `C_car` it constructs.
Every reader is save code, and `loadsave_savegame_save` writes a kind-4 or kind-8 actor only when
the byte is zero. The clears are the objects whose state has changed and must therefore be saved.
⁴ **`+0x66`** — set by `mission_open_bin` on every actor it creates from the scene `.bin` whose
slot-18 `Init` succeeded. Its one reader is `game_done`, which walks the mission actor vector and,
for every flagged actor of kind 4, either releases it (act state 2) or hands it to
`mission_active_actor_store` so it survives the scene teardown. Found by an image-wide disassembly
scan for displacement `0x66`: 17 sites, of which exactly three touch a `C_actor` — the two writers
and that reader. The `C_program` run at `+0x64`/`+0x66`/`+0x67` and the `C_Vehicle` word accesses
are different classes.
⁵ **`+0x68`** — `folded_get_dword_0x68` is the raw slot read (253 callsites); the other two readers
resolve the world matrix through it. The actor's displayed name is the FRAME's name (`frame+0x100`,
`i3d_frame_get_name`, `structs/i3d_frame.md` §1), not a `C_actor` field.

### 1c. `+0x5e` and `+0x5f` — the two accessors

Both are one-line out-of-line inline members hosted by the `C_sledovacka` translation unit. They do
not change §1b's field readings; they give them entry points.

| PC VA | Name | Field | Receiver evidence |
|---|---|---|---|
| `0x004f57c0` | `actor_is_death_processed` | `+0x5e` — the death-processed latch | 6 callsites, all receivers disassembled and all `C_actor`-derived: the occupant returned by `car_get_owner`, and actors indexed out of the script VM's register file |
| `0x004f57d0` | `actor_is_in_camera_sector` | `+0x5f` — the sector byte | 2 callsites, both `mov al,[ecx+0x2155]` immediately before the call, so `this` is the chased `C_car` |

**Why `+0x5f` is "the camera's sector", not "the main sector".** `actor_tick` writes
`+0x5f = (frame->+0x110 != 9 && frame->+0x110 != 1) ? 1 : (frame->+0x13c == view->+0x13c)`. Three
independent confirmations:

1. **`game_tick` contains the identical three-clause predicate, inlined**, as the gate deciding
   whether an actor is ticked at all.
2. It also **caches it**: `*(g_act_mission+0x64) = (*(g_act_mission+0x10))->+0x13c;` — and
   `pedestrian_tick` compares against that cache and stores the result in `pedestrian+0x50`. Three
   classes, one predicate.
3. `*(g_act_mission+0x10)` is the **`I3D_scene`**, and it DERIVES FROM `I3D_frame`, which is why
   `frame->+0x13c` and `view->+0x13c` are the same offset on both objects. Its vtable slot `0x54` is
   present/swap-buffers (`basic/artifacts/game-loop.md`), the active camera frame hangs off `+0x17c`
   (`i3d_scene_get_active_camera_ptr`), and slot `+0xbc` is
   `SetFrameSectorPos(I3D_frame *, const S_vector &)` — which is what named it
   (`structs/i3d_frame.md`).

The noun *sector* is backed by a symbol: PS2
`SetFrameSectorPos__9I3D_sceneCFP9I3D_frameRC8S_vector` calls the scene's `GetSector(pos)` and then
`frame->LinkTo(sector, 1)`, and its PC twin `i3d_scene_set_frame_sector_pos` (`0x005c8600`) is
called on `*(g_act_mission+0x10)`. `+0x13c` is the sector slot the two objects are compared on.

Consumers: `car_update` and `car_extended_update` gate expensive per-frame work on `+0x5f` **plus** a
400-unit distance test — the classic "same visibility cell, and near" cull.

### 1d. The `act_state` enum, decoded from the script side

`actor_set_act_state` is called with a bare integer all over the engine and nothing on the C++ side
names the values. The script language does: `act_setstate xx, yy` takes a KEYWORD, and the two halves
of the script VM together give the complete mapping. `program_decode_params`' opcode-`0x19` arm
resolves the keyword at LOAD time and `program_process_command`'s `case 0x19` then **remaps it** —
the two numberings differ, which is why neither half alone reads as an enum.

| script keyword | payload `argv[1]` | `act_state` (`+0x1c`) | meaning |
|---|---:|---:|---|
| `active` | 1 | **0** | the actor is active |
| `inactive` | 2 | **1** | the actor is inactive |
| `off`, and any unrecognised word | 0 | **2** | the actor is switched off; the Mafia Script Data Reference adds that a corpse stays in the world |

Anything the decoder does not recognise falls to 0 → state **2**, so a typo in a script switches the
actor OFF rather than erroring.

This is the enum behind three otherwise-unexplained constants elsewhere: `actor_rebirth`'s
`if (act_state == 2)` gate, which revives an actor that was switched off; `program_clear_after_end`'s
`SetActState(owner, 2)` when a script ends; and the `C_detector` override's `PROGRAM_STORAGE` arm,
which parks its actor in state 2 (`structs/detector.md` §1). PS2 names the type `E_act_state` in
`SetActState__7C_actorFQ27C_actor11E_act_state`; the PC decomp has no symbol for it.

## 2. The two file-scope globals

### 2a. `C_actor::S_posscoll_frame` — the broad-phase collision-candidate table

A file-scope `std::vector` of 8-byte records:

```c
struct S_posscoll_frame {   /* PS2 `Q27C_actor16S_posscoll_frame`, 8 bytes */
    /* +0x00 */ I3D_frame *frame;   /* [H] the candidate's transform frame          */
    /* +0x04 */ float      radius;  /* [H] its bounding radius; callers pass 10.0f  */
};
```

| Global | Role |
|---|---|
| `g_actor_posscoll_table_begin` | begin |
| `g_actor_posscoll_table_end` | end |
| `g_actor_posscoll_table_cap` | capacity end |
| `g_actor_posscoll_table_alloc` | the vector's allocator byte, cleared by `actor_posscoll_table_ctor` |

**Who writes it:** `actor_add_frame_for_possible_coll` (search-then-upsert) and
`actor_del_frame_for_possible_coll` (search-then-**swap-pop**, so order is never preserved).
**Who reads it:** `actor_is_possible_coll_all`, the table's only reader.

**The element type comes from the PS2 side and could not have come from the PC side.** The PC decomp
can only say "a global stride-8 vector"; the PS2 insert symbol
`insert__…FPQ27C_actor16S_posscoll_frame…` names the type outright.

### 2b. The global actor list

`g_actor_global_list_begin` / `g_actor_global_list_end` / `g_actor_global_list_cap` (begin / end /
capacity) is the **global actor list**, PS2 `global_actors__7C_actor` — identified by
`actor_ref_save_resolve`, which scans it exactly as PS2 `Actor_SaveGameLoad` scans its own. Written
by `actor_add_to_global_list` (called from `actor_ctor`) and `actor_del_from_global_list`, searched
by `actor_ref_save_resolve` and `actor_is_in_global_list`. The guard byte is
`g_actor_static_vector_exit_guard`.

### 2c. The two globals' lifecycle quartets

Both vectors are built and torn down by an MSVC static-init quartet, named per-global rather than
anonymously because the object each one manages is identified.

| role | global actor list | posscoll table |
|---|---|---|
| static-init entry | `actor_global_list_static_init` `0x0044b5d0` | `actor_posscoll_table_static_init` `0x0044b650` |
| ctor | `actor_global_list_ctor` `0x0044b5e0` | `actor_posscoll_table_ctor` `0x0044b660` |
| atexit registrar | `actor_global_list_register_atexit_dtor` `0x0044b600` | `actor_posscoll_table_register_atexit_dtor` `0x0044b680` |
| dtor | `actor_global_list_dtor` `0x0044b610` | `actor_posscoll_table_dtor` `0x0044b690` |
| exit guard | **bit 0** of `g_actor_static_vector_exit_guard` | **bit 1** of `g_actor_static_vector_exit_guard` |

The two dtors **share one guard byte, one bit each** — that is how the pairs are told apart. None of
the eight has a PS2 twin, and that is a measurement, not a gap: GCC implements file-scope static
construction through `__sti__` routines and the `.ctors` section, an entirely different mechanism
(§6).

## 3. The `actor_` prefix caveat

The PC `actor_` prefix is **over-applied**: of roughly 128 `actor_*` PC functions, many are `C_human`
methods (weapons, animation, movement — `actor_weapon_*`, `actor_set_movement_intent_*`,
`actor_set_anim_state_*`, `actor_play_*_animation`) or `C_entity` AI methods (`actor_perception_*`,
`actor_target_observation_*`, `actor_query_target_*`, `actor_relationship_tracker_*`). Those belong
to the derived classes and are not `C_actor` methods. §4 maps only the 47 genuine `7C_actor` PS2
methods to their PC implementation, regardless of the PC function's current prefix.

## 4. `C_actor` method family — PC ↔ PS2 (all 47 methods)

PC-first, ordered as the class reads. Status legend: **CONFIRMED** = body-diffed both sides ·
**base-stub** = the PS2 base implementation is a `0x8`/`0x16`-byte overridden virtual (returns 0 or
does nothing) with no distinct PC body, inlined or unnamed on PC.

Coverage: **30 of the 47 have a PC twin and all 30 are CONFIRMED**; the other 17 are base stubs.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0044b6d0 | `actor_ctor` | `__ct__7C_actorFv` | 0x001142b0 | CONFIRMED | base ctor: kind = 1, vtable `0063b640`, `g_scene_ref_count++` |
| 0x0044b740 | `actor_dtor` | `__dt__7C_actorFv` | 0x00114320 | CONFIRMED | base dtor: unregister, then drop the reference |
| 0x0044d9e0 | `actor_get_world_position` | `GetWorldPos__7C_actorFv` | 0x001172f0 | CONFIRMED | frame `+0x68` → `UpdateWMatrixProc` → matrix `+0x40` |
| 0x0044da40 | `actor_get_forward_normalized` | `GetWorldDir__7C_actorFv` | 0x00117350 | CONFIRMED | same frame, forward vector |
| 0x0044c750 | `actor_set_act_state` | `SetActState__7C_actorFQ27C_actor11E_act_state` | 0x00116080 | CONFIRMED | writes the act-state enum at `+0x1c` (§1d) |
| 0x0044d750 | `actor_add_to_global_list` | `AddActorToGlobal__7C_actorFP7C_actor` | 0x00117010 | CONFIRMED | appends the actor to the global vector (§2b); called from `actor_ctor` |
| 0x0044d950 | `actor_del_from_global_list` | `DelActorFromGlobal__7C_actorFP7C_actor` | 0x00117270 | CONFIRMED | searches and removes `param_1` from that vector; the inverse of the row above and adjacent to it in address order |
| 0x0044bea0 | `actor_tick` | `Tick__7C_actorFUi` | 0x00114e70 | CONFIRMED | base per-frame tick: sector byte `+0x5f` from the frame type, Update-vs-AI vtable dispatch on `+0x20`, then the once-only death latch `+0x5d`/`+0x5e` → Death. This diff established PS2 `+0x60`/`+0x61`/`+0x62`/`+0x63` ↔ PC `+0x20`/`+0x5d`/`+0x5e`/`+0x5f` |
| 0x00425410 | `folded_get_dword_0x68` | `GetFrame__7C_actorFv` | 0x001b3260 | CONFIRMED | returns the frame slot (PC `+0x68`, PS2 `+0x70`); **253 direct callsites** |
| 0x0044b780 | `actor_init` | `Init__7C_actorFP9I3D_frame` | 0x001143e0 | CONFIRMED | `(this, frame)`; returns 0 if the frame is null, then attaches via the `frame+8` child list |
| 0x0044b810 | `actor_internal_placement_reset` | `InternalPlacementReset__7C_actorFv` | 0x001144e0 | CONFIRMED | reads the frame local matrix (`frame+0x80`) back into the actor's `+0x3c`; the inverse of `ResetPlacement`, PS2 reading `frame+0xb4` |
| 0x0044b890 | `actor_reset_placement` | `ResetPlacement__7C_actorFv` | 0x00114570 | CONFIRMED | actor pos `+0x3c` + quat `+0x48` → frame local matrix + `S_quat::Normalize` + dirty masks. Statement for statement with PS2 |
| 0x0044b950 | `actor_game_init` | `GameInit__7C_actorFv` | 0x00114670 | CONFIRMED | installs the frame's owner backlink (`frame+8`, recording `+0x64 = 1` when it allocated it), reads the frame placement into the actor, caches world pos `+0x24` and forward `+0x30`, sets ALIVE `+0x5d = 1`, and for kind 1 runs debustize + noshadow |
| 0x0044bd90 | `actor_game_done` | `GameDone__7C_actorFv` | 0x00114cc0 | CONFIRMED | scene teardown: `ResetPlacement`'s body PLUS a leading frame-vtable `(frame,1)` call and a trailing `if (+0x64) { free the frame+8 owner block }` — the exact undo of what `actor_game_init` allocates. Proven by the statement-aligned pair `actor_shot_game_done` ↔ `GameDone__12C_actor_shotFv`, whose first statement is the call to this function on both builds |
| 0x004253b0 | `actor_release` | `Release__7C_actorFv` | 0x00117b70 | CONFIRMED | drops one reference: pre-decrements the short `ref_count` at `+0x0c` and, at zero, calls the scalar deleting destructor via vtable slot 16 (`+0x40`); PS2 does the same on `+0x4c` via vtable `+0x14`. **0 direct `E8` callsites, 35 `.rdata` datarefs** — the 35-vtable census is what identifies it as a `C_actor` base virtual |
| 0x0044c820 | `actor_rebirth` | `Rebirth__7C_actorFv` | 0x001160c0 | CONFIRMED | `if (act_state +0x1c == 2) { +0x1c = 0; scene-dirty = 1; }` — brings a pooled actor back to life. Byte-exact (PS2 `+0x5c` ↔ PC `+0x1c`); 8 direct callsites including all three pool-deploy wrappers |
| 0x0044c120 | `actor_save_game_get_size` | `SaveGameGetSize__7C_actorFv` | 0x00115320 | CONFIRMED | vtable slot 22: `0xd` plus, per type-9 frame child, model basename + frame name + parent name + `0x28`; gated on kind 4 |
| 0x0044c220 | `actor_save_game_save` | `SaveGameSave__7C_actorFPPv` | 0x001154b0 | CONFIRMED | vtable slot 23; writes the tag-3 header plus one block per type-9 child. Sizer, writer and reader agree field for field |
| 0x0044c4b0 | `actor_save_game_load` | `SaveGameLoad__7C_actorFPPv` | 0x00115850 | CONFIRMED | reads a type-3 record from the `*param_2` stream, restores `+0x18`/`+0x5d`/`+0x5e`, calls `actor_set_act_state` and reloads the model via `i3d_cache_model_open`; `__thiscall(this, stream)` |
| 0x0044c730 | `actor_save_game_load_guid` | `SaveGameLoadGuid__7C_actorFPPv` | 0x00116020 | CONFIRMED | identity-only pre-pass: `if (tag == 3) +0x18 = guid`; the cursor is NOT advanced. PS2 `+0x58` ↔ PC `+0x18` |
| 0x0044bf40 | `actor_ref_save_get_size` | `Actor_SaveGameGetSize__7C_actorFP7C_actor` | 0x00114fb0 | CONFIRMED | size of an actor REFERENCE record: frame name (`frame+0x100`, fallback `g_empty_string`) strlen + 8. It NULL-CHECKS ITS ARGUMENT, which is what separates this static helper from the virtual sizer above |
| 0x0044bfa0 | `actor_ref_save_write` | `Actor_SaveGameSave__7C_actorFPPvP7C_actor` | 0x00115070 | CONFIRMED | writes an actor REFERENCE record `{namelen, kind, name}` and advances the cursor by `len + 8`. Two parameters `(stream, actor)` match `FPPvP7C_actor`, not the one-parameter virtual |
| 0x0044c050 | `actor_ref_save_resolve` | `Actor_SaveGameLoad__7C_actorFPPv` | 0x001151a0 | CONFIRMED | reads `{len, kind}`, scans the global actor list for a matching frame name + kind and returns the live `C_actor *`. Byte-exact; it is what identifies the global list as PS2 `global_actors__7C_actor` |
| 0x0044c840 | `actor_is_possible_coll_all` | `IsPossibleCollAll__7C_actorF...` | 0x001160f0 | CONFIRMED | swept broad-phase against every `S_posscoll_frame` entry (§2a) — the table's only reader. Callee counts: PS2 `S_vector::Dot` ×8 ↔ PC `vec3_dot_product` ×7; PS2 `S_vector` ctor ×4 ↔ `vec3_set_xyz` ×3 + `vec3_sub` ×1; `UpdateWMatrixProc` ×1 ↔ ×1. 5 parameters on both sides |
| 0x0044cfa0 | `actor_get_bound_points_2d` | `GetBoundPoints2D__7C_actorF...` | 0x001169a0 | CONFIRMED | vtable slot 34. The PS2 template argument is `vector<S_vector2>` and every PC helper is stride-8 (`std_vector8_insert_at`, `stl_copy_one_stride8` ×3, `vec_count_stride_0x8`, `stl_fill_n_stride8`) — two floats per point. Plus `UpdateHRBoundVol` ×2 |
| 0x0044d6d0 | `actor_debustize_frame` | `DebustizeFrame__7C_actorFP9I3D_frame` | 0x00116ed0 | CONFIRMED | frame-child recursion; conditional frame-vtable call at `+0x5c`. Pairs with the row below |
| 0x0044d710 | `actor_no_shadow_frame` | `NoShadowFrame__7C_actorFP9I3D_frame` | 0x00116f70 | CONFIRMED | identical frame-child recursion; frame-vtable `+0x54` |
| 0x0044dbd0 | `actor_add_frame_for_possible_coll` | `AddFrameForPossibleColl__7C_actorFP9I3D_framef` | 0x001173c0 | CONFIRMED | upserts `{I3D_frame*, radius}` into the table of §2a; callers pass radius `10.0f` |
| 0x0044dde0 | `actor_del_frame_for_possible_coll` | `DelFrameForPossibleColl__7C_actorFP9I3D_frame` | 0x00117450 | CONFIRMED | swap-pops the frame out of that table — the same distinctive shape on both builds |
| 0x0044e8a0 | `actor_get_actor_speed` | `GetActorSpeed__7C_actorFR8S_vector` | 0x00117bf0 | CONFIRMED | writes the zero vector; `C_human` and `C_car` override it. Vtable **slot 28 in 29 vtables**, **0 direct `E8` callsites** |
| - | - | `GetProgram__7C_actorFv` | 0x00117c10 | base-stub | returns 0; `C_entity` overrides it to return its `C_program` |
| - | - | `AI__7C_actorFUi` | 0x00114f90 | base-stub | overridden by `C_human` / `C_entity` |
| - | - | `Update__7C_actorFUi` | 0x00114f80 | base-stub | - |
| - | - | `NetDirect__7C_actorFUi` | 0x00114fa0 | base-stub | - |
| - | - | `Death__7C_actorFv` | 0x00116050 | base-stub | - |
| - | - | `Collision__7C_actorF...` | 0x00116060 | base-stub | overridden per subclass |
| - | - | `Hit__7C_actorF10E_hit_type...` | 0x00116070 | base-stub | overridden per subclass |
| - | - | `IsPossibleColl__7C_actorF...` | 0x00116950 | base-stub | - |
| - | - | `DuplicateFrom__7C_actorFP7C_actor` | 0x001143d0 | base-stub | - |
| - | - | `ChangeState__7C_actorFv` | 0x00117be0 | base-stub | - |
| - | - | `GetMaterialID__7C_actorFP9I3D_frame` | 0x00117c00 | base-stub | - |
| - | - | `cbCollTester__7C_actorFP18tDynamicCollObject` | 0x00117c20 | base-stub | - |
| - | - | `SoundsChange__7C_actorFi` | 0x00117c30 | base-stub | - |
| - | - | `ChangeSceneInit__7C_actorFv` | 0x00117c40 | base-stub | - |
| - | - | `ChangeSceneDone__7C_actorFv` | 0x00117c50 | base-stub | - |
| - | - | `SaveData__7C_actorFP7C_chunk` | 0x00117bc0 | base-stub | - |
| - | - | `LoadData__7C_actorFP7C_chunk` | 0x00117bd0 | base-stub | - |

**A class's PC code is not one contiguous cluster.** Most of the rows above sit in
`0x0044a000`–`0x0044f200`, but `actor_release` (`0x004253b0`) and `folded_get_dword_0x68`
(`0x00425410`) are roughly 130 KB below it, inside the inline-accessor block MSVC emitted after
`entity_action_tick`: an inline member lands in whichever `.obj` used it, not in the class's own
`.obj`. Both are also invisible to any callsite-based hunt, because **both have zero direct `E8`
callsites** — they are pure vtable slots, found instead by counting `.rdata` datarefs (35 vtables for
`Release`) and reading the slot POSITION. When a PS2 method is not located after searching the
cluster, check whether it is virtual-only before concluding it is inlined.

These functions sit in the same VA cluster but are **not** `C_actor` methods:
`actor_global_list_static_init` / `_ctor` / `_register_atexit_dtor` / `_dtor` and their posscoll
counterparts (§2c), `actor_shot_ctor`, `airplane_ctor` and `airplane_scalar_deleting_dtor` (subclass
ctors/dtors), and the cache-item helpers `i3d_cache_items_uninit_fill_n`,
`i3d_cache_items_copy_assign_range` and `i3d_cache_items_destroy_one` (`structs/i3d_cache.md` §4b).

### 4a. PC-only `C_actor` accessors

The table above is scoped to the 47 `7C_actor` PS2 methods. These two PC functions are `C_actor`
accessors with **no** PS2 counterpart, because PS2 inlines them.

| PC VA | PC name | Reads | Notes |
|---|---|---|---|
| 0x004253a0 | `folded_get_dword_0x10` | `+0x10` | the kind discriminator, and the most-called accessor in this class at **364 references**. Its consumers compare the result against 2/3/4/8/0x1b, and at `0x00415747`, `0x0041b97a`, `0x0042d8c9` and `0x005c5da5` the receiver is `actor_get_vehicle()`'s return with `cmp eax,4` as the next instruction. The symbol is ICF-folded and also serves `C_mission` |
| 0x004253d0 | `actor_get_cached_world_pos` | `+0x24..+0x2c` | copies the cached position |

### 4b. The `C_actor.cpp` free-helper tail

The class's translation unit also holds a handful of free functions; PS2 carries
`S_hlp$2695C_actor_cpp` in its symbol table, naming the source file. Every slot is mapped, and the
alignment is monotone:

| PS2 VA | PS2 | PC VA | PC |
|---|---|---|---|
| 0x00117450 | `DelFrameForPossibleColl__7C_actor` | 0x0044dde0 | `actor_del_frame_for_possible_coll` |
| 0x001174f0 | `rnd__Fi` | 0x0044de50 | `rand_range` |
| 0x00117540 | `rnd__Fv` | 0x0044de70 | `rand_float` |
| 0x00117580 | `AngleTo2__FRC9S_vector2RC9S_vector2` | 0x0044de90 | `vec2_signed_angle_diff` |
| - | - | 0x0044ded0 | `vec3_rotate_around_y_xz_unit_in_place` |
| - | - | 0x0044e030 | `vec3_rotate_around_y_in_place` |
| 0x00117980 | `compare_anim_index__FPCvPCv` | 0x0044e090 | `compare_anim_index` |
| 0x001179b0 | `InitAnimIndex__Fv` | 0x0044e0b0 | `anim_index_init` |
| 0x00117a70 | `DoneAnimIndex__Fv` | 0x0044e150 | `anim_index_free` |
| 0x00117a80 | `GetAnimIndex__FUi` | 0x0044e160 | `anim_index_of_id` |

**The anim-index trio** builds and searches a sorted `{anim_id, ordinal}` index over the static table
at `g_anim_info_table`, which maps an animation id to its `.i3d` clip filename (`run1.i3d`,
`walk1.i3d`, `strafRL1.i3d`, `!plachteni.i3d`, `breath02a.i3d`, …). Its element is an 8-byte
`S_anim_info` record with the id at `+0x00` (§6).

## 5. Actor identity on the network

Two `C_actor` fields are decoded by the event-message layer.

**`actor+0x18` is the network GUID**, and its **top byte is the owning peer's slot**.
`actor_apply_net_guid_table` (`0x005e9490`) walks the type-`0xc` remap table as
`{count, (old_guid, new_guid) × count}`, finds each world actor in `g_act_mission+0x38..0x3c` by its
current `+0x18` and re-stamps it. This agrees with `actor_save_game_load_guid`, which already records
`+0x18 = guid`, and with `bottle_post_collision_net_event_msg`'s owner-byte swap
through `guid >> 24`.

**`actor+0x20` is set for a remotely-owned actor.** The same function writes
`(actor+0x10 != 0x1b) && (g_net_session+0x230 != (old_guid >> 24))` — "this peer does not own this
actor", with kind `0x1b` (`C_entity`) excluded. It is the same byte `actor_tick` reads as its
per-frame slot selector (§1b), so a non-owned actor is dispatched through the AI slot rather than
Update. `car_break_glass` reads it to decide whether to originate a glass-break message at all.

`actor_post_hit_event` and `actor_post_collision_event` are the two `C_actor` producers in the layer,
posting types `0x1c` and `0x1d`; their payload helpers are `hit_event_payload_fill`,
`hit_event_payload_serialize` and `hit_event_payload_deserialize` (`structs/net_event_msg.md` §2).
`actor_apply_net_guid_table` has no PS2 twin — the whole networking subsystem is PC-only.

## 6. PC vs PS2

* **The vtable is offset by a whole base class.** PC slot = PS2 vtable index + 13, because PC has the
  `C_actor_track` interface in slots 0–12 and PS2 has no such class (§1a): `__vt__7C_actor` holds
  exactly the 27 methods that map to PC slots 13–38 and nothing before them, and no `C_recorder`
  symbol or string exists in `SLUS_206.71`.
* **The vptr sits in a different place.** PC keeps it at `+0x00`, and `actor_ctor` calls no base
  ctor. PS2's `C_actor` has no polymorphic base, so GCC 2.x puts the vptr *after* the data:
  `__ct__7C_actorFv` writes its vtable to `this+0x78`, and the destructor restores exactly one vptr
  and calls no base destructor (`structs/actor_track.md` §5).
* **The field deltas are not constant**, and one of them is negative (see the table below). Map
  fields by ROLE, never by rebasing — the same caution applies to every class derived from
  `C_actor`, where measured pairs likewise include negative deltas.
* **The complete base layout, ordered by PC offset.** Every PC field of §1 is either paired with a
  PS2 field or measured as having none. The ctors (`actor_ctor` ↔ `__ct__7C_actorFv`), the
  `Init`s, the `GameInit`s and the dtors are each statement-aligned, and those four pairs are what
  pinned most rows. PS2 vectors are 16 bytes wide, which is why the vector rows have their own
  deltas.

| PC | PS2 | delta | Field | How it was pinned |
|---|---|---:|---|---|
| `+0x00` | `+0x78` | `+0x78` | vptr | the ctor's first store on each build |
| `+0x04` | - | - | track mode flags | PC-only: the `C_recorder` client state (`structs/actor_track.md` §5) |
| `+0x08` | - | - | track pointer | PC-only, same reason |
| `+0x0c` | `+0x4c` | `+0x40` | ref_count | `actor_release` / `Release__7C_actorFv`, same protocol (PS2 through vtable `+0x14`) |
| `+0x10` | `+0x50` | `+0x40` | kind | the ctor's stamp on both builds |
| `+0x14` | `+0x54` | `+0x40` | class id | the ctors: 1 on `C_actor`, 2 on `C_human`, 3 on `C_car_ex`, on both builds |
| `+0x18` | `+0x58` | `+0x40` | net GUID | `actor_init` ↔ `Init__7C_actorFP9I3D_frame` (`++act_mission+8`), and `actor_save_game_load_guid` |
| `+0x1c` | `+0x5c` | `+0x40` | act_state | `actor_rebirth`'s `== 2` gate; zeroed by both `Init`s and both `GameInit`s |
| `+0x20` | `+0x60` | `+0x40` | AI-driven selector | `actor_tick`'s vtable branch |
| `+0x24` | `+0x00` | `-0x24` | cached world position | the `GameInit`s' no-frame branch zeroes both |
| `+0x30` | `+0x10` | `-0x20` | cached forward | the `ACTOR_SETDIR` round trip, which uses `actor+0x10..+0x18` on PS2; the `GameInit`s set both to `(0, 0, 1)` |
| `+0x3c` | `+0x20` | `-0x1c` | placement position | `actor_reset_placement` ↔ `ResetPlacement__7C_actorFv` |
| `+0x48` | `+0x30` | `-0x18` | placement quaternion | the same body, before `S_quat::Normalize` |
| `+0x58` | `+0x40` | `-0x18` | frame parent | `actor_reset_placement`, statement-aligned |
| `+0x5c` | `+0x44` | `-0x18` | saved frame visibility byte | the same body |
| `+0x5d` | `+0x61` | `+0x04` | alive flag | `actor_tick`'s death latch |
| `+0x5e` | `+0x62` | `+0x04` | death-processed latch | the same latch; also `ShowCarEnergy`'s early return |
| `+0x5f` | `+0x63` | `+0x04` | in-camera-sector byte | the same body's three-clause predicate |
| `+0x60` | `+0x64` | `+0x04` | pending track event | the ctors zero one dword at this position; on the PC it is the recorder's queued event record, and no PS2 method reads PS2 `+0x64` |
| `+0x64` | `+0x68` | `+0x04` | owns the frame's owner block | the `GameInit` / `GameDone` allocate-free loop |
| `+0x65` | `+0x69` | `+0x04` | save_excluded | both `GameInit`s set it to `kind == 1`; PS2 `C_traffic_car::Krles` raises it on the car it creates, as PC `car_explosion` does; the PS2 readers are the same save routines (`SaveLogs__8C_entity`, `SaveSensors__8C_entity`, `SaveVisLogs__8C_entity`, `SaveGameSave__12C_sledovacka`, `SaveGameSave__15G_LoadSaveClass`) |
| `+0x66` | `+0x6a` | `+0x04` | scene_placed | PS2 `C_mission::OpenBin` sets it and `C_game::Done` reads it, as PC `mission_open_bin` / `game_done` do |
| `+0x68` | `+0x70` | `+0x08` | frame | `GetFrame__7C_actorFv`; bound by `actor_init` ↔ `Init__7C_actorFP9I3D_frame` |
| `+0x6c` | `+0x74` | `+0x08` | owns-its-frame flag | the `ACTOR_DUPLICATE` arm, statement-identical; both dtors release the frame only when it is set |
| - | `+0x48` | - | a heap array | PS2-only: `__dt__7C_actorFv` frees it with `delete[]`; the PC dtor has no such step |
| - | `+0x6b` | - | a byte | PS2-only: zeroed by the PS2 ctor, not by the PC one |
| - | `+0x6c` | - | a byte | PS2-only: zeroed by the ctor, set to 2 by `C_game::VyhjebejTemporaryho` |

* **Both builds treat `+0x5d` non-zero as alive** — the polarity is not inverted.
* **`ShowCarEnergy` corroborates `+0x5e ↔ +0x62`.** PS2 `ShowCarEnergy__5C_carFv` opens
  `if (this->+0x62) return 0.0f;` exactly where PC `car_get_condition_fraction` reads `+0x5e` — a
  second, independent body confirming both the pair and the semantics: a finalised actor reports
  zero condition. The setter is `actor_tick`.
* **Static construction differs by toolchain.** None of the eight lifecycle functions of §2c has a
  PS2 twin, because GCC uses `__sti__` routines and the `.ctors` section instead of MSVC's
  static-init quartet. A toolchain artifact, not a missing method.
* **`S_anim_info`'s internal field order differs** between builds (§4b): PS2 reads the id at
  `+0x04`, PC at `+0x00`, while the algorithm over it does not. The type name comes from PS2
  (`std::vector<11S_anim_info>`); the PC decomp shows only a stride-8 record.
* **The identical-sibling trap in the `C_actor.cpp` tail (§4b).** PS2 has two byte-identical
  `rand()/RAND_MAX` functions — `rnd__Fv` and `RandFloat` at `0x00437f60`, roughly 3 MB away. A
  body-diff cannot separate them; what identifies `rnd__Fv` is the emission position plus the
  `rnd__Fi`/`rnd__Fv` **overload pairing**, which matches PC's adjacent `rand_range` /
  `rand_float`.

## 7. Related

- `structs/actor_track.md` — the PC-only base class holding vtable slots 0–12; `structs/recorder.md`
  — the keyframe-track system it is a client of.
- `structs/entity.md` — the kind-value enum and per-type properties (kind 1 = `C_actor`);
  `structs/human.md` — the `C_human` layer above this one.
- `structs/i3d_frame.md` — the `I3D_frame` at `+0x68`, including the frame name at `frame+0x100`.
- `structs/program.md` §6 — the `C_program` translation unit that hosts `actor_set_cached_world_pos`.
- `structs/net_event_msg.md` §2 — the event-message layer of §5.
- `structs/driver.md`, `structs/vehicle.md`, `structs/car.md` — `C_actor` subclasses.
- `basic/artifacts/game-loop.md` — where `actor_tick` is driven from.

## 8. Open items

Nothing is open.
