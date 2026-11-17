# `C_actor` — the root actor base class

**Created** 2026-07-12 09:41 +0200 · **Modified** 2026-11-15 18:48 +0100

The root of the game's actor hierarchy: `C_actor` ← `C_human` ← { `C_entity`, `C_player` }, with
`C_car` / `C_traffic_car` / `C_railway` / `C_wagon`, spike-strips and props deriving from it as well.
It owns the transform/scene-graph frame, global-actor registry membership, world position and
direction, the per-frame `Tick`, the collision broad-phase entry points and the SaveGame plumbing.

| | |
|---|---|
| PC prefix | `actor_` |
| PC kind (`+0x10`) | `1`, stamped by `actor_ctor` |
| PC vtable | `g_actor_vt` @ `0x0063b640`, 39 slots: 0–12 the `C_actor_track` interface (§4), 13–38 `C_actor`'s own |
| PC base class | `C_actor_track` (`structs/actor_track.md`) |
| PC instance size | `0x70` (`operator_new(0x70)` in `mission_create_actor` case `1`) |
| PC constructor | `actor_ctor` @ `0x0044b6d0` |
| PC translation unit | `C_actor.cpp` `[0x0044a620, 0x0044e970)` (§6) |
| PS2 class | `C_actor` (tag `7C_actor`), 47 methods: 30 CONFIRMED twins and 17 base stubs, all mapped in §3 |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the `C_actor` **base** fields are listed; subclasses extend the object far beyond these and
their fields belong in the respective class doc. Offsets are PC (`Game.exe`, image base
`0x00400000`).

```c
typedef struct C_actor {                    // 0x70 bytes; PC offsets
    /* +0x00 */ void      *vtable;              // [H] g_actor_vt; ctor sets, dtor restores
    /* +0x04 */ uint32_t   track_flags;         // [H] C_actor_track mode flags, a dword (§4)
    /* +0x08 */ void      *track;               // [H] the C_recorder keyframe track
    /* +0x0c */ uint16_t   ref_count;           // [H] reference count; ctor = 1
    /* +0x10 */ int32_t    kind;                // [H] class discriminator; 1 for a bare C_actor
    /* +0x14 */ int32_t    class_id;            // [H] branch tag: 1 base, 2 C_human, 3 C_car_ex
    /* +0x18 */ uint32_t   net_guid;            // [H] network GUID; top byte = owning peer (§7)
    /* +0x1c */ int32_t    act_state;           // [H] 0 active, 1 inactive, 2 switched off (§2b)
    /* +0x20 */ uint8_t    ai_driven;           // [H] tick selector: 0 Update, non-zero AI
    /* +0x24 */ float      cached_world_pos[3]; // [H] cached world position x/y/z
    /* +0x30 */ float      cached_forward[3];   // [H] cached forward row of the frame
    /* +0x3c */ float      position[3];         // [H] placement position
    /* +0x48 */ float      rotation[4];         // [H] placement quaternion
    /* +0x58 */ void      *frame_parent;        // [H] saved frame parent (placement round trip)
    /* +0x5c */ uint8_t    frame_visible;       // [H] saved frame visibility byte
    /* +0x5d */ uint8_t    alive;               // [H] non-zero = alive
    /* +0x5e */ uint8_t    death_processed;     // [H] once-only death latch
    /* +0x5f */ uint8_t    in_camera_sector;    // [H] 1 = same visibility cell as the camera (§2c)
    /* +0x60 */ void      *pending_track_event; // [H] recorder event record queued for the next key
    /* +0x64 */ uint8_t    owns_frame_owner_block; // [H] actor_game_init allocated the owner block
    /* +0x65 */ uint8_t    save_excluded;       // [H] 1 = the savegame skips this actor
    /* +0x66 */ uint8_t    scene_placed;        // [H] 1 = placed by the scene .bin loader
    /* +0x68 */ I3D_frame *frame;               // [H] scene-graph frame, bound by actor_init
    /* +0x6c */ uint8_t    owns_frame;          // [H] 1 = actor_dtor releases the frame
    /* +0x6d */ uint8_t    may_broadcast;       // [H] 1 = this actor may post its state to the session
} C_actor;
```

Global side-state (not per-instance) is in §5.

### 1a. `C_actor::S_posscoll_frame` — one broad-phase collision candidate

The element of the file-scope candidate table of §5a.

```c
typedef struct S_posscoll_frame {           // 8 bytes
    /* +0x00 */ I3D_frame *frame;               // [H] the candidate's transform frame
    /* +0x04 */ float      radius;              // [H] its bounding radius; callers pass 10.0f
} S_posscoll_frame;
```

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `actor_ctor`, `actor_dtor` | `actor_tick`, `actor_release` | `g_actor_vt`; read through virtual dispatch |
| +0x04 | track_flags | H | `actor_ctor` (=0) | `actor_tick` | bits 0 and 2 suppress the per-frame Update/AI dispatch; owned by `C_actor_track` (§4) |
| +0x08 | track | H | `actor_ctor` (=0) | the slot 0–12 interface | the `C_recorder` keyframe track (§4) |
| +0x0c | ref_count | H | `actor_ctor` (=1) | `actor_release` | 16-bit. `actor_release` pre-decrements and, at zero, invokes vtable slot 16 (`+0x40`) — the scalar deleting destructor — with the delete flag |
| +0x10 | kind | H | `actor_ctor`, `human_ctor`, `entity_ctor`, `player_ctor` | `folded_get_dword_0x10` | 1 actor, 2 player, 3 human, 4 car, 0x13 `C_don_ocas` (the ambient-crowd stand-in actor), 0x14 `C_bridge`, 0x1b entity — `structs/entity_kind.md` §1. The engine's stand-in for RTTI, the build carrying no RTTI at all, which is why the reader has 346 direct callsites. The reader symbol is ICF-folded and also serves `C_mission` and `C_sound_table` (§3b) |
| +0x14 | class_id | H | `actor_ctor` (=1), `human_ctor` (=2), `car_extended_ctor` (=3) | `actor_get_class_id` | a branch tag with exactly three values — **1** every class that does not restamp it, **2** `C_human` (inherited by `C_entity` and `C_player`), **3** `C_car_ex`. The three ctor stores are every store at this offset in the binary's constructors. Its one reader tests `kind == 4 && +0x14 == 3` to select a truck |
| +0x18 | net_guid | H | `actor_init` (`++C_mission+0x08`, `structs/mission.md` §1), `actor_apply_net_guid_table`, `actor_save_game_load_guid` | `actor_apply_net_guid_table`, the event-message layer | `actor_init` draws it from a per-mission counter; top byte = owning peer slot (§7) |
| +0x1c | act_state | H | `actor_init` (=0), `actor_game_init` (=0), `actor_set_act_state`, `actor_rebirth` (=0), `actor_save_game_load` | `actor_rebirth`, the tag-3 savegame record | the three-value enum of §2b |
| +0x20 | ai_driven | H | `actor_ctor` (=0), `actor_apply_net_guid_table` | `actor_tick` | `actor_tick` dispatches the Update slot when this byte is zero and the AI slot otherwise; the network layer sets it for every actor this peer does not own (§7) |
| +0x24 | cached_world_pos | H | `actor_game_init`, `car_update`, `actor_set_cached_world_pos`, `entity_action_tick` (case `0x21` substate `0xa`: the arrest spot) | `actor_get_cached_world_pos`, `actor_get_world_position` (when the actor has no frame), `garage_game_done`, `entity_action_tick` (case `0x25`) | three floats; the fast path against the frame recompute. `car_update` refreshes it on every tick of a locally simulated car (`+0x20 == 0`) from the body position `car+0x40c`, so a car's copy is current. `actor_set_cached_world_pos` is an out-of-line inline member hosted by the `C_program` translation unit (`structs/program.md` §8); its three callsites all take the receiver out of the script VM's actor register file |
| +0x30 | cached_forward | H | `actor_game_init`, `actor_set_direction`, `entity_action_tick` (case `0x21` substate `0xa`: the negated approach direction) | `actor_get_direction`, `entity_action_tick` (case `0x25`) | three floats; the frame's forward row, cached. The `actor_setdir` arm normalises a direction, stores it here, reads it straight back through `actor_get_direction` and hands it to the frame's world-forward virtual |
| +0x3c | position | H | `actor_game_init`, `actor_internal_placement_reset` | `actor_reset_placement`, `actor_game_done` | three floats. The placement round trip: `actor_reset_placement` pushes position and quaternion into the frame's local matrix with `S_quat::Normalize` and the dirty masks; `actor_internal_placement_reset` reads them back |
| +0x48 | rotation | H | `actor_game_init`, `actor_internal_placement_reset` | `actor_reset_placement`, `actor_game_done` | the placement quaternion of the same round trip, normalised on push |
| +0x58 | frame_parent | H | `actor_game_init`, `actor_internal_placement_reset` | `actor_reset_placement`, `actor_game_done` | the frame's parent, for the placement round trip |
| +0x5c | frame_visible | H | `actor_game_init` (bit 0 of the frame's flags) | `actor_reset_placement`, `actor_game_done` | the visibility byte of the same round trip |
| +0x5d | alive | H | set (=1): the `C_actor` init, `human_activate`; cleared: seven functions; restored: `actor_save_game_load` | `actor_is_alive`, `entity_is_tickable`, `entity_script_tick`, `entity_suspend_ai_and_hide`, `actor_tick`, `entity_process_command` | non-zero = alive ¹ |
| +0x5e | death_processed | H | set (=1): `actor_tick`, `human_death`, `human_hit`, `car_explosion`; cleared (=0): `actor_game_init`, `human_activate`, `car_game_init`, `car_reset`; restored: `actor_save_game_load` | `actor_is_death_processed`, `police_manager_count_officers_alive_for_target` | the death has been handled, so the teardown runs exactly once ² |
| +0x5f | in_camera_sector | H | `actor_game_init` (= 0), `actor_tick` | `actor_is_in_camera_sector`, `car_update`, `car_extended_update` | recomputed every tick (§2c); `car_update` copies it into `C_Vehicle +0x360` every frame (`structs/vehicle.md` §2) |
| +0x60 | pending_track_event | H | `actor_ctor` (=0), `human_track_queue_anim_event` (allocates the record once, fills it, stores the pointer) | `human_track_record_key` (encodes the record into the next key and clears its byte 0) | a `0x48`-byte record: byte 0 pending flag, `+0x04` value, `+0x08` target frame name, `+0x2c` position, `+0x38` quaternion; filled only while the track records (`+0x04` bit `0x2`). Measured by a census of `this+0x60` over the methods of every `C_actor`-derived class |
| +0x64 | owns_frame_owner_block | H | `actor_game_init` (1 when it allocated the block, else 0) | `actor_game_done` | the allocate/free pair across GameInit and GameDone |
| +0x65 | save_excluded | H | set (=1): `actor_game_init` (`kind == 1`), the `game_init` of every runtime class ³, `car_explosion`, the police-car pool and pursuit-car spawners, `game_load_differences`; set from an argument: `actor_set_save_excluded`; cleared (=0): `detector_set_signal`, `detector_save_game_load`, `human_use_actor` | `entity_save_logs`, `entity_save_vis_logs`, `entity_save_sensors`, `entity_get_logs_size`, `pursuit_event_save_game_get_size`, `pursuit_event_save_game_save`, `loadsave_savegame_save` | 1 = the savegame skips this actor ³ |
| +0x66 | scene_placed | H | `actor_ctor` (=0), `actor_mark_scene_placed` (=1) | `game_done` | 1 = a FILE-PLACED actor, not one the script, the traffic generator or the pedestrian crowd created ⁴ |
| +0x68 | frame | H | `actor_ctor` (=0), `actor_init` (binds the frame argument, refusing one another actor owns); released and cleared by `human_game_done`, `car_game_done`, `dropout_game_done`; rebound by `box_init`, `dropout_init` | `folded_get_dword_0x68`, `actor_get_world_position`, `actor_get_forward_normalized`, `entity_process_command` | the transform frame ⁵ |
| +0x6c | owns_frame | H | `actor_init` (=0), `actor_mark_created_at_runtime` (from its argument) | `actor_dtor` | `actor_dtor` releases the frame through its vtable slot 0 only when this byte is set and the frame is non-null. Set on a freshly duplicated actor whose slot-18 `Init` returned true, immediately before `game_add_temporary_actor`. The complement of `+0x66` |
| +0x6d | may_broadcast | H | 1: `bottle_ctor`, `grenade_ctor`, `box_init`, `physical_object_net_snapshot_state`, `car_net_snapshot_state`; 0: `cartridge_ctor`, `smoke_trail_ctor` | `actor_set_act_state`, `bottle_ai_tick`, `bottle_set_flight_impulse`, `bottle_on_collision`, `bottle_hit`, `bottle_apply_net_event_msg`, `box_ai_tick`, `car_update`, `physical_object_update` | `actor_ctor` does not write it. `actor_set_act_state`, `box_ai_tick`, `car_update` and `physical_object_update` post a state message only under `g_net_session+0x100 == 7` and this byte. Writers closed by a byte scan of `.text`, reading each base register |

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
`actor_set_save_excluded` has one callsite, the script `DAN_INTERNAL_1` arm (case `0x185`), on the
car that arm spawns. Every reader is save code, and `loadsave_savegame_save` writes a kind-4 or
kind-8 actor only when the byte is zero. The clears are the objects whose state has changed and
must therefore be saved.
⁴ **`+0x66`** — set by `mission_open_bin` on every actor it creates from the scene `.bin` whose
slot-18 `Init` succeeded. Its one reader is `game_done`, which walks the mission actor vector and,
for every flagged actor of kind 4, either releases it (act state 2) or hands it to
`mission_active_actor_store` so it survives the scene teardown. Found by an image-wide disassembly
scan for displacement `0x66`: 17 sites, of which exactly three touch a `C_actor` — the two writers
and that reader. The `C_program` run at `+0x64`/`+0x66`/`+0x67` and the `C_Vehicle` word accesses
are different classes.
⁵ **`+0x68`** — `folded_get_dword_0x68` is the raw slot read (253 direct callsites); the other two
readers resolve the world matrix through it. The actor's displayed name is the FRAME's name
(`frame+0x100`, `i3d_frame_get_name`, `structs/i3d_frame.md` §1), not a `C_actor` field.

### 2a. `C_actor::S_posscoll_frame`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | frame | H | `actor_add_frame_for_possible_coll` | `actor_is_possible_coll_all`, `actor_del_frame_for_possible_coll` (search key) | the candidate's `I3D_frame *`; the upsert and the swap-pop both search the table by it |
| +0x04 | radius | H | `actor_add_frame_for_possible_coll` | `actor_is_possible_coll_all` | the bounding radius; every caller passes `10.0f` |

### 2b. `+0x1c` — the `act_state` enum, decoded from the script side

`actor_set_act_state` is called with a bare integer all over the engine and nothing on the C++ side
names the values. The script language does: `act_setstate xx, yy` takes a KEYWORD, and the two halves
of the script VM together give the complete mapping. `program_decode_params`' opcode-`0x19` arm
resolves the keyword at LOAD time and `program_process_command`'s `case 0x19` then **remaps it** —
the two numberings differ, which is why neither half alone reads as an enum.

| Value | Meaning | Evidence |
|---|---|---|
| 0 | the actor is active | script keyword `active`, decoder payload `argv[1]` = 1 |
| 1 | the actor is inactive | script keyword `inactive`, payload 2 |
| 2 | the actor is switched off; the Mafia Script Data Reference adds that a corpse stays in the world | script keyword `off` and any unrecognised word, payload 0 |

Anything the decoder does not recognise falls to payload 0 → state **2**, so a typo in a script
switches the actor OFF rather than erroring.

This is the enum behind three otherwise-unexplained constants elsewhere: `actor_rebirth`'s
`if (act_state == 2)` gate, which revives an actor that was switched off; `program_clear_after_end`'s
`SetActState(owner, 2)` when a script ends; and the `C_detector` override's `PROGRAM_STORAGE` arm,
which parks its actor in state 2 (`structs/detector.md` §4). The type name is `E_act_state`, from
the PS2 symbol `SetActState__7C_actorFQ27C_actor11E_act_state`; the PC decomp has no symbol for it.

### 2c. `+0x5f` — the camera's sector

`actor_tick` writes
`+0x5f = (frame->+0x110 != 9 && frame->+0x110 != 1) ? 1 : (frame->+0x13c == view->+0x13c)`. Three
independent confirmations that it means "the camera's sector":

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

`+0x13c` is the sector slot the two objects are compared on; the noun *sector* is backed by a
symbol (§8). Consumers: `car_update` and `car_extended_update` gate expensive per-frame work on
`+0x5f` **plus** a 400-unit distance test — the classic "same visibility cell, and near" cull.

## 3. Method family

Coverage: all **47** PS2 `7C_actor` methods are in the tables below — **30 CONFIRMED twins** and
**17 base stubs**. 15 of the stubs sit on a shared PC stub at the matching vtable slot (PC slot =
PS2 index + 13, §8); `ChangeSceneInit` and `ChangeSceneDone` have no PC slot. A further **42** PC
functions carry the `actor_` prefix, are file-scope lifecycle code, or are free helpers of the
translation unit (§6); of these, `actor_apply_explosion_damage` and the seven free helpers with
a `rnd`/`AngleTo2`/anim-index symbol have a PS2 twin that is a free function.

A class's PC code is not one contiguous cluster: `actor_release` and `folded_get_dword_0x68` sit
roughly 130 KB below the TU, inside the inline-accessor block MSVC emitted after
`entity_action_tick` — an inline member lands in whichever `.obj` used it. Both have zero direct
`E8` callsites; they are pure vtable slots or inline reads, found by counting `.rdata` datarefs
(35 vtables for `Release`) and reading the slot position. When a method is not located in the
cluster, check whether it is virtual-only before concluding it is inlined.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0044a620 | `crt_static_init_guard_0044a620` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets the shared static-init flag and tail-jumps to the registrar below |
| 0x0044a640 | `crt_atexit_register_0044a640` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0044b5d0 | `actor_global_list_static_init` | `g_actor_global_list_begin` | - | - | - | CRT static-init entry of the global actor list (§5b) |
| 0x0044b5e0 | `actor_global_list_ctor` | `g_actor_global_list_begin` | - | - | - | zeroes begin / end / capacity |
| 0x0044b600 | `actor_global_list_register_atexit_dtor` | `g_actor_global_list_begin` | - | - | - | `crt_atexit(&actor_global_list_dtor)` |
| 0x0044b610 | `actor_global_list_dtor` | `g_actor_global_list_begin` | - | - | - | exit destructor; guard = **bit 0** of `g_actor_static_vector_exit_guard` |
| 0x0044b650 | `actor_posscoll_table_static_init` | `g_actor_posscoll_table_begin` | - | - | - | CRT static-init entry of the candidate table (§5a) |
| 0x0044b660 | `actor_posscoll_table_ctor` | `g_actor_posscoll_table_begin` | - | - | - | zeroes begin / end / capacity and clears the allocator byte `g_actor_posscoll_table_alloc` |
| 0x0044b680 | `actor_posscoll_table_register_atexit_dtor` | `g_actor_posscoll_table_begin` | - | - | - | `crt_atexit(&actor_posscoll_table_dtor)` |
| 0x0044b690 | `actor_posscoll_table_dtor` | `g_actor_posscoll_table_begin` | - | - | - | exit destructor; guard = **bit 1** of `g_actor_static_vector_exit_guard` |
| 0x0044b6d0 | `actor_ctor` | instance | `__ct__7C_actorFv` | 0x001142b0 | CONFIRMED | base ctor: kind = 1, vtable `g_actor_vt`, `g_actor_global_refs++`, joins the global actor list through `actor_add_to_global_list` |
| 0x0044b720 | `actor_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `actor_dtor`; vtable slot 16, the slot `actor_release` calls at zero references |
| 0x0044b740 | `actor_dtor` | instance | `__dt__7C_actorFv` | 0x00114320 | CONFIRMED | base dtor: unregister, then drop the reference |
| 0x005a17e0 | `actor_state_event_payload_ctor` | the `0x1c` actor-state event payload | - | - | - | ctor of the actor-snapshot payload; its field offsets match the actor-state serializer (`structs/net_event_msg.md`) |

### 3b. Accessors

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004253a0 | `folded_get_dword_0x10` | - | - | - | loads `+0x10`, the kind; **346 direct callsites**. ICF-folded: one address serving three accessors — the `C_actor` kind (about 317 callsites), `C_mission`'s `I3D_scene` at `+0x10` (29 callsites, `structs/mission.md` §3) and `C_sound_table` vtable slot 9, reached only through its vtable (`structs/sound_table.md`). Its consumers compare the result against 2/3/4/8/0x1b, and at `0x00415747`, `0x0041b97a`, `0x0042d8c9` and `0x005c5da5` the receiver is `actor_get_vehicle()`'s return with `cmp eax,4` as the next instruction |
| 0x00425410 | `folded_get_dword_0x68` | `GetFrame__7C_actorFv` | 0x001b3260 | CONFIRMED | loads the frame slot `+0x68`; **253 direct callsites** |
| 0x00425430 | `actor_is_alive` | - | - | - | loads the alive byte `+0x5d` |
| 0x00425730 | `actor_get_vehicle` | - | - | - | loads `+0x98`, a field past the `C_actor` band: the `C_human` vehicle link (`structs/human.md` §1); about 129 callsites |
| 0x004f57c0 | `actor_is_death_processed` | - | - | - | loads the latch `+0x5e`. An out-of-line inline member hosted by the `pursuit_event` translation unit; 6 callsites, all receivers disassembled and all `C_actor`-derived — the occupant returned by `car_get_owner`, and actors indexed out of the script VM's register file |
| 0x004f57d0 | `actor_is_in_camera_sector` | - | - | - | loads the sector byte `+0x5f` (§2c); same host unit; 2 callsites, both `mov al,[ecx+0x2155]` immediately before the call, so `this` is the chased `C_car` |
| 0x005c86e0 | `actor_get_class_id` | - | - | - | loads the class id `+0x14` |
| 0x005c8730 | `actor_mark_created_at_runtime` | - | - | - | stores its byte argument to `+0x6c`; called on a freshly duplicated actor before `game_add_temporary_actor` |
| 0x005c8740 | `actor_set_save_excluded` | - | - | - | stores its byte argument to `+0x65` |
| 0x006065c0 | `actor_mark_scene_placed` | - | - | - | stores 1 to `+0x66` |

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x004253b0 | `actor_release` | `Release__7C_actorFv` | 0x00117b70 | CONFIRMED | drops one reference: pre-decrements the short `ref_count` at `+0x0c` and, at zero, calls the scalar deleting destructor via vtable slot 16 (`+0x40`). **0 direct `E8` callsites, 35 `.rdata` datarefs** — the 35-vtable census is what identifies it as a `C_actor` base virtual |
| 0x004253d0 | `actor_get_cached_world_pos` | - | - | - | copies the three floats `+0x24..+0x2c` to an out-vector |
| 0x004253f0 | `actor_set_direction` | - | - | - | three raw float stores into `+0x30/+0x34/+0x38` |
| 0x00425420 | `stub_ret0_args4_00425420` | `GetMaterialID__7C_actorFP9I3D_frame` | 0x00117c00 | base-stub | slot 32; returns 0; one argument on both builds. An ICF-folded stub shared with other vtables |
| 0x0044b780 | `actor_init` | `Init__7C_actorFP9I3D_frame` | 0x001143e0 | CONFIRMED | `(this, frame)`; returns 0 if the frame is null, then attaches via the `frame+8` child list |
| 0x0044b810 | `actor_internal_placement_reset` | `InternalPlacementReset__7C_actorFv` | 0x001144e0 | CONFIRMED | reads the frame local matrix (`frame+0x80`) back into the actor's `+0x3c`; the inverse of `ResetPlacement` |
| 0x0044b890 | `actor_reset_placement` | `ResetPlacement__7C_actorFv` | 0x00114570 | CONFIRMED | actor pos `+0x3c` + quat `+0x48` → frame local matrix + `S_quat::Normalize` + dirty masks. Statement for statement with PS2 |
| 0x0044b950 | `actor_game_init` | `GameInit__7C_actorFv` | 0x00114670 | CONFIRMED | installs the frame's owner backlink (`frame+8`, recording `+0x64 = 1` when it allocated it), reads the frame placement into the actor, caches world pos `+0x24` and forward `+0x30`, sets ALIVE `+0x5d = 1`, and for kind 1 runs debustize + noshadow |
| 0x0044bd90 | `actor_game_done` | `GameDone__7C_actorFv` | 0x00114cc0 | CONFIRMED | scene teardown: `ResetPlacement`'s body PLUS a leading frame-vtable `(frame,1)` call and a trailing `if (+0x64) { free the frame+8 owner block }` — the exact undo of what `actor_game_init` allocates. Proven by the statement-aligned pair `actor_shot_game_done` ↔ `GameDone__12C_actor_shotFv`, whose first statement is the call to this function on both builds |
| 0x0044bea0 | `actor_tick` | `Tick__7C_actorFUi` | 0x00114e70 | CONFIRMED | base per-frame tick: sector byte `+0x5f` from the frame type, Update-vs-AI vtable dispatch on `+0x20`, then the once-only death latch `+0x5d`/`+0x5e` → Death. This diff established PS2 `+0x60`/`+0x61`/`+0x62`/`+0x63` ↔ PC `+0x20`/`+0x5d`/`+0x5e`/`+0x5f` |
| 0x0044bf40 | `actor_ref_save_get_size` | `Actor_SaveGameGetSize__7C_actorFP7C_actor` | 0x00114fb0 | CONFIRMED | size of an actor REFERENCE record: frame name (`frame+0x100`, fallback `g_empty_string`) strlen + 8. It NULL-CHECKS ITS ARGUMENT, which is what separates this static helper from the virtual sizer |
| 0x0044bfa0 | `actor_ref_save_write` | `Actor_SaveGameSave__7C_actorFPPvP7C_actor` | 0x00115070 | CONFIRMED | writes an actor REFERENCE record `{namelen, kind, name}` and advances the cursor by `len + 8`. Two parameters `(stream, actor)` match `FPPvP7C_actor`, not the one-parameter virtual |
| 0x0044c050 | `actor_ref_save_resolve` | `Actor_SaveGameLoad__7C_actorFPPv` | 0x001151a0 | CONFIRMED | reads `{len, kind}`, scans the global actor list for a matching frame name + kind and returns the live `C_actor *`. Byte-exact; it is what identifies the global list (§5b) |
| 0x0044c120 | `actor_save_game_get_size` | `SaveGameGetSize__7C_actorFv` | 0x00115320 | CONFIRMED | vtable slot 22: `0xd` plus, per type-9 frame child, model basename + frame name + parent name + `0x28`; gated on kind 4 |
| 0x0044c220 | `actor_save_game_save` | `SaveGameSave__7C_actorFPPv` | 0x001154b0 | CONFIRMED | vtable slot 23; writes the tag-3 header plus one block per type-9 child. Sizer, writer and reader agree field for field |
| 0x0044c4b0 | `actor_save_game_load` | `SaveGameLoad__7C_actorFPPv` | 0x00115850 | CONFIRMED | reads a type-3 record from the `*param_2` stream, restores `+0x18`/`+0x5d`/`+0x5e`, calls `actor_set_act_state` and reloads the model via `i3d_cache_model_open`; `__thiscall(this, stream)` |
| 0x0044c730 | `actor_save_game_load_guid` | `SaveGameLoadGuid__7C_actorFPPv` | 0x00116020 | CONFIRMED | identity-only pre-pass: `if (tag == 3) +0x18 = guid`; the cursor is NOT advanced |
| 0x0044c750 | `actor_set_act_state` | `SetActState__7C_actorFQ27C_actor11E_act_state` | 0x00116080 | CONFIRMED | writes the act-state enum at `+0x1c` (§2b) |
| 0x0044c820 | `actor_rebirth` | `Rebirth__7C_actorFv` | 0x001160c0 | CONFIRMED | `if (act_state +0x1c == 2) { +0x1c = 0; scene-dirty = 1; }` — brings a pooled actor back to life. Byte-exact; 8 direct callsites including all three pool-deploy wrappers |
| 0x0044c840 | `actor_is_possible_coll_all` | `IsPossibleCollAll__7C_actorFR8S_vectorR8S_vectorfR8S_vectorPP7C_actor` | 0x001160f0 | CONFIRMED | swept broad-phase against every `S_posscoll_frame` entry (§5a) — the table's only reader. Callee counts: PS2 `S_vector::Dot` ×8 ↔ PC `vec3_dot_product` ×7; PS2 `S_vector` ctor ×4 ↔ `vec3_set_xyz` ×3 + `vec3_sub` ×1; `UpdateWMatrixProc` ×1 ↔ ×1. 5 parameters on both sides |
| 0x0044cf90 | `stub_retfalse_args40_0044cf90` | `IsPossibleColl__7C_actorFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor` | 0x00116950 | base-stub | slot 33; returns false; folded across 27 vtables. `ret 0x28` = 10 argument dwords against 6 PS2 parameters — the count matches if both `S_vector` parameters travel by value [M] |
| 0x0044cfa0 | `actor_get_bound_points_2d` | `GetBoundPoints2D__7C_actorFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame` | 0x001169a0 | CONFIRMED | vtable slot 34. The PS2 template argument is `vector<S_vector2>` and every PC helper is stride-8 (`std_vector8_insert_at`, `stl_copy_one_stride8` ×3, `vec_count_stride_0x8`, `stl_fill_n_stride8`) — two floats per point. Plus `UpdateHRBoundVol` ×2 |
| 0x0044d6d0 | `actor_debustize_frame` | `DebustizeFrame__7C_actorFP9I3D_frame` | 0x00116ed0 | CONFIRMED | frame-child recursion; conditional frame-vtable call at `+0x5c`. Pairs with the row below |
| 0x0044d710 | `actor_no_shadow_frame` | `NoShadowFrame__7C_actorFP9I3D_frame` | 0x00116f70 | CONFIRMED | identical frame-child recursion; frame-vtable `+0x54` |
| 0x0044d750 | `actor_add_to_global_list` | `AddActorToGlobal__7C_actorFP7C_actor` | 0x00117010 | CONFIRMED | appends the actor to the global vector (§5b); called from `actor_ctor` |
| 0x0044d950 | `actor_del_from_global_list` | `DelActorFromGlobal__7C_actorFP7C_actor` | 0x00117270 | CONFIRMED | searches and removes `param_1` from that vector; the inverse of the row above and adjacent to it in address order |
| 0x0044d9b0 | `actor_is_in_global_list` | - | - | - | backward membership scan of the global actor list |
| 0x0044d9e0 | `actor_get_world_position` | `GetWorldPos__7C_actorFv` | 0x001172f0 | CONFIRMED | frame `+0x68` → `UpdateWMatrixProc` → matrix `+0x40` |
| 0x0044da40 | `actor_get_forward_normalized` | `GetWorldDir__7C_actorFv` | 0x00117350 | CONFIRMED | same frame, forward vector |
| 0x0044dbd0 | `actor_add_frame_for_possible_coll` | `AddFrameForPossibleColl__7C_actorFP9I3D_framef` | 0x001173c0 | CONFIRMED | upserts `{I3D_frame*, radius}` into the table of §5a; callers pass radius `10.0f` |
| 0x0044dde0 | `actor_del_frame_for_possible_coll` | `DelFrameForPossibleColl__7C_actorFP9I3D_frame` | 0x00117450 | CONFIRMED | swap-pops the frame out of that table — the same distinctive shape on both builds |
| 0x0044e1b0 | `actor_post_hit_event` | - | - | - | packs the `Hit` argument list into a `0x38` payload, wraps it as message type `0x1c`, serialises and posts (§7) |
| 0x0044e300 | `actor_post_collision_event` | - | - | - | packs the `Collision` argument list into a `0x30` payload, message type `0x1d` (§7) |
| 0x0044e8a0 | `actor_get_actor_speed` | `GetActorSpeed__7C_actorFR8S_vector` | 0x00117bf0 | CONFIRMED | writes the zero vector; `C_human` and `C_car` override it. Vtable **slot 28 in 29 vtables**, **0 direct `E8` callsites** |
| 0x00461180 | `stub_noop_args4_00461180` | `DuplicateFrom__7C_actorFP7C_actor` | 0x001143d0 | base-stub | slot 21; does nothing; one argument. The slot-21 map across the family is `structs/actor_track.md` §4c |
| 0x004776f0 | `stub_noop_args4_004776f0` | `AI__7C_actorFUi` · `Update__7C_actorFUi` · `NetDirect__7C_actorFUi` · `cbCollTester__7C_actorFP18tDynamicCollObject` · `SoundsChange__7C_actorFi` | 0x00114f90 · 0x00114f80 · 0x00114fa0 · 0x00117c20 · 0x00117c30 | base-stub | slots 13, 14, 15, 36, 37 — a **5 → 1 fold**: five one-argument base virtuals that do nothing, ICF-folded onto one stub. `C_human` / `C_entity` override `AI`; `actor_tick` dispatches `AI` or `Update` on `+0x20` |
| 0x00477700 | `stub_ret0_cdecl_00477700` | `GetProgram__7C_actorFv` | 0x00117c10 | base-stub | slot 35; returns 0; `C_entity` overrides it to return its `C_program` |
| 0x00477710 | `stub_rettrue_args4_00477710` | `SaveData__7C_actorFP7C_chunk` · `LoadData__7C_actorFP7C_chunk` | 0x00117bc0 · 0x00117bd0 | base-stub | slots 19 and 20 — a **2 → 1 fold**; returns true; one argument |
| 0x00482690 | `stub_retfalse_args24_00482690` | `Collision__7C_actorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame` | 0x00116060 | base-stub | slot 30; returns false; `ret 0x18` = 6 argument dwords, the PS2 parameter count; overridden per subclass |
| 0x0048a4f0 | `stub_retfalse_args32_0048a4f0` | `Hit__7C_actorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame` | 0x00116070 | base-stub | slot 31; returns false; `ret 0x20` = 8 argument dwords, the PS2 parameter count; overridden per subclass |
| 0x004ae890 | `actor_subclass_init_require_frame` | - | - | - | the slot-18 `Init` fold shared by **6** `C_actor`-subclass vtables; `C_actor`'s own slot 18 is `actor_init`. Sibling of `bottle_subclass_init_require_frame` |
| 0x00574a40 | `stub_noop_cdecl_00574a40` | `Death__7C_actorFv` | 0x00116050 | base-stub | slot 29; does nothing; `actor_tick` reaches it through the death latch |
| 0x005c86f0 | `actor_set_cached_world_pos` | - | - | - | writes the three floats `+0x24..+0x2c`; the byte-exact mirror of `actor_get_cached_world_pos`; hosted by the `C_program` translation unit (`structs/program.md` §8) |
| 0x005c8710 | `actor_get_direction` | - | - | - | returns the cached forward vector `+0x30..+0x38` |
| 0x005c8750 | `actor_get_global_list` | - | - | - | returns the global actor list (§5b) |
| 0x005e7070 | `actor_apply_explosion_damage` | `_explosion__FP7C_actorfP7C_actorR8S_vectorff` | 0x002f5230 | CONFIRMED | applies an explosion's damage to an actor; the PS2 twin is a free function with a `C_actor *` first parameter |
| 0x005e9490 | `actor_apply_net_guid_table` | - | - | - | re-stamps world actor GUIDs from the type-`0xc` table and marks remote-owned actors at `+0x20` (§7) |
| 0x0060b700 | `crt_atexit_noop_dtor_0060b700` | `ChangeState__7C_actorFv` | 0x00117be0 | base-stub | slot 27; a bare `ret` shared by every do-nothing slot and unit destructor of its shape (149 data references) |
| - | - | `ChangeSceneInit__7C_actorFv` | 0x00117c40 | base-stub | no PC slot: the PC vtable ends at slot 38, whose stub (`stub_noop_args8_00575c00`) takes 2 arguments against PS2's none (§8) |
| - | - | `ChangeSceneDone__7C_actorFv` | 0x00117c50 | base-stub | no PC slot (§8) |

### 3d. Free helpers

The `C_actor.cpp` translation unit also holds a handful of free functions. Every one is
generic-named because it is used far beyond the class; they are listed here because they live
in this unit.

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0044a780 | `stub_retminus1_args4_0044a780` | - | - | - | slot 0 default of the `C_actor_track` interface (`structs/actor_track.md` §4), in this unit's COMDAT band; returns -1 |
| 0x0044de50 | `rand_range` | `rnd__Fi` | 0x001174f0 | CONFIRMED | byte-exact `n < 1 ? 0 : rand() % n`; 208 callsites across every family |
| 0x0044de70 | `rand_float` | `rnd__Fv` | 0x00117540 | CONFIRMED | `rand()/RAND_MAX` |
| 0x0044de90 | `vec2_signed_angle_diff` | `AngleTo2__FRC9S_vector2RC9S_vector2` | 0x00117580 | CONFIRMED | byte-exact `atan2(b) - atan2(a)` wrapped into ±π |
| 0x0044ded0 | `vec3_rotate_around_y_xz_unit_in_place` | - | - | - | rotates a vector about the world Y axis in place, then flattens Y and normalises on X/Z |
| 0x0044e030 | `vec3_rotate_around_y_in_place` | - | - | - | rotates a vector about the world Y axis in place, no flatten |
| 0x0044e090 | `compare_anim_index` | `compare_anim_index__FPCvPCv` | 0x00117980 | CONFIRMED | three-way unsigned compare of the first dword; the `crt_qsort` comparator of the index below |
| 0x0044e0b0 | `anim_index_init` | `InitAnimIndex__Fv` | 0x001179b0 | CONFIRMED | builds the sorted `{anim_id, ordinal}` index over `g_anim_info_table` |
| 0x0044e150 | `anim_index_free` | `DoneAnimIndex__Fv` | 0x00117a70 | CONFIRMED | frees that index |
| 0x0044e160 | `anim_index_of_id` | `GetAnimIndex__FUi` | 0x00117a80 | CONFIRMED | binary search: animation id → table ordinal, -1 when absent |

**The anim-index trio** builds and searches a sorted `{anim_id, ordinal}` index over the static
table at `g_anim_info_table`, which maps an animation id to its `.i3d` clip filename (`run1.i3d`,
`walk1.i3d`, `strafRL1.i3d`, `!plachteni.i3d`, `breath02a.i3d`, …). Its element is an 8-byte
`S_anim_info` record with the id at `+0x00` (§8).

## 4. The PC base class `C_actor_track` — vtable slots 0–12

On the PC, `C_actor`'s own virtuals **do not start at slot 0**. Slots **0–12** of every
`C_actor`-family vtable hold a 13-method interface belonging to a base class, `C_actor_track`
(`structs/actor_track.md`): the per-object client of the `C_recorder` keyframe-track system
(`structs/recorder.md`). `C_actor`'s own virtuals occupy slots **13–38**. Slots 13/14/15 are
`C_actor`'s own `AI` / `Update` / `NetDirect`, all ICF-folded onto the single stub
`stub_noop_args4_004776f0`. `C_actor` is therefore the root of the *game* hierarchy and of the kind
enum, but not of the vtable.

The base is **data-free as far as the PC is concerned**: `actor_ctor` writes the derived vptr at
`+0x00` and calls no base constructor, because there is none to call. It does own the fields the
interface touches — `+0x04` mode flags, `+0x08` track, `+0x60` pending event record, `+0x68`
`I3D_frame*` (`structs/actor_track.md` §1).

**The whole subsystem is PC-only** (§8).

## 5. The file-scope globals

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_actor_posscoll_table_alloc` | 0x00655348 | allocator byte of the candidate table | `actor_posscoll_table_ctor` | - |
| `g_actor_posscoll_table_begin` | 0x0065534c | begin of the candidate table (§5a) | `actor_posscoll_table_ctor`, `actor_posscoll_table_dtor`, `actor_add_frame_for_possible_coll` (regrowth) | `actor_is_possible_coll_all`, `actor_add_frame_for_possible_coll`, `actor_del_frame_for_possible_coll`, `game_init` |
| `g_actor_posscoll_table_end` | 0x00655350 | end of the candidate table | `actor_posscoll_table_ctor`, `actor_posscoll_table_dtor`, `actor_add_frame_for_possible_coll`, `actor_del_frame_for_possible_coll`, `game_init` (empties the table) | `actor_is_possible_coll_all` and the same writers |
| `g_actor_posscoll_table_cap` | 0x00655354 | capacity end of the candidate table | `actor_posscoll_table_ctor`, `actor_posscoll_table_dtor`, `actor_add_frame_for_possible_coll` | `actor_add_frame_for_possible_coll` |
| `g_actor_static_vector_exit_guard` | 0x00655358 | ONE guard byte shared by both exit destructors: bit 0 the global list, bit 1 the candidate table | `actor_global_list_dtor`, `actor_posscoll_table_dtor` | the same two |
| `g_actor_global_list_alloc` | 0x0065535c | allocator byte of the global actor list, and the first byte of its vector object | `actor_global_list_ctor` | `actor_get_global_list` and `loadsave_savegame_save` (take the vector object's address) |
| `g_actor_global_list_begin` | 0x00655360 | begin of the global actor list (§5b) | `actor_global_list_ctor`, `actor_global_list_dtor`, `actor_add_to_global_list` | `actor_ref_save_resolve`, `actor_is_in_global_list`, `actor_del_from_global_list` and 17 iterators elsewhere ⁶ |
| `g_actor_global_list_end` | 0x00655364 | end of the global actor list | the same three, and `actor_del_from_global_list` | the same readers |
| `g_actor_global_list_cap` | 0x00655368 | capacity end of the global actor list | `actor_global_list_ctor`, `actor_global_list_dtor`, `actor_add_to_global_list` | `actor_add_to_global_list` |
| `g_actor_global_refs` | 0x0065536c | live `C_actor` count | `actor_ctor` (+1), `actor_dtor` (−1), `mission_open` (reset) | `actor_ctor`, `actor_dtor` |

⁶ The 17 iterators: `game_done`, `game_get_actor_from_guid`, `game_invalidate_actor`,
`game_stop_all_sounds`, `game_tick_car_sound_manager`, `game_car_gun_tick`,
`human_can_special_stroke`, `human_do_box_cancel`, `human_find_blocking_drawbridge`,
`opponent_test_node_vs_cars`, `pedestrian_tick`, `police_manager_game_done`,
`police_manager_npc_list_tick`, `traffic_car_game_init`, `using_object_save_game_get_size`,
`using_object_save_game_save`, `wagon_rebuild_all_forward_paths`. Measured by attributing every
decomp reference to these globals to its containing function.

Both vectors are built and torn down by an MSVC static-init quartet (§3a), named per global
because the object each one manages is identified. The two exit destructors **share one guard
byte, one bit each** — that is how the pairs are told apart.

### 5a. The broad-phase collision-candidate table

A file-scope `std::vector` of `S_posscoll_frame` records (§1a). **Who writes it:**
`actor_add_frame_for_possible_coll` (search-then-upsert), `actor_del_frame_for_possible_coll`
(search-then-**swap-pop**, so order is never preserved) and `game_init`, which empties it.
**Who reads it:**
`actor_is_possible_coll_all`, the table's only reader. The PC decomp alone can only say "a
global stride-8 vector"; the element type's name comes from a PS2 symbol (§8).

### 5b. The global actor list

`g_actor_global_list_begin` / `_end` / `_cap` is the **global actor list**, identified by
`actor_ref_save_resolve`, which scans it for a saved actor reference. Written by
`actor_add_to_global_list` (called from `actor_ctor`) and `actor_del_from_global_list`, searched
by `actor_ref_save_resolve` and `actor_is_in_global_list`, returned whole by
`actor_get_global_list`.

## 6. The translation unit

`C_actor.cpp` is `[0x0044a620, 0x0044e970)`: it opens with its CRT static-init pair
(`crt_static_init_guard_0044a620` / `crt_atexit_register_0044a640`) and the next unit's guard,
`crt_static_init_guard_0044e970`, closes it. Inside it:

* the COMDAT band `[0x0044a650, 0x0044b5c0)` of the `C_actor_track` interface
  (`structs/actor_track.md`), which also holds `C_actor_shot`'s two track overrides
  `actor_shot_track_create` and `actor_shot_track_apply_key_effects`;
* the two static-init quartets, then the class's own methods `0x0044b6d0`–`0x0044e8a0`;
* the free-helper tail of §3d;
* foreign COMDATs that are not `C_actor` methods: `i3d_cache_sound_open` and the cache-item
  helpers `i3d_cache_items_destroy_one`, `i3d_cache_items_destroy_range`,
  `i3d_cache_items_uninit_fill_n` and `i3d_cache_items_copy_assign_range`
  (`structs/i3d_cache.md` §3c).

The next units are `C_actor_shot` (`actor_shot_ctor` at `0x0044e9a0`) and `C_airplane`
(`airplane_ctor` at `0x0044ef50`), subclasses documented in their own files.

## 7. Actor identity on the network

Two `C_actor` fields are decoded by the event-message layer.

**`actor+0x18` is the network GUID**, and its **top byte is the owning peer's slot**.
`actor_apply_net_guid_table` (`0x005e9490`) walks the type-`0xc` remap table as
`{count, (old_guid, new_guid) × count}`, finds each world actor in `g_act_mission+0x38..0x3c` by its
current `+0x18` and re-stamps it. This agrees with `actor_save_game_load_guid`, which records
`+0x18 = guid`, and with `bottle_post_collision_net_event_msg`'s owner-byte swap through
`guid >> 24`.

**`actor+0x20` is set for a remotely-owned actor.** The same function writes
`(actor+0x10 != 0x1b) && (g_net_session+0x230 != (old_guid >> 24))` — "this peer does not own this
actor", with kind `0x1b` (`C_entity`) excluded. It is the same byte `actor_tick` reads as its
per-frame slot selector (§2), so a non-owned actor is dispatched through the AI slot rather than
Update. `car_break_glass` reads it to decide whether to originate a glass-break message at all.

`actor_post_hit_event` and `actor_post_collision_event` are the two `C_actor` producers in the layer,
posting types `0x1c` and `0x1d`; their payload helpers are `hit_event_payload_fill`,
`hit_event_payload_serialize` and `hit_event_payload_deserialize` (`structs/net_event_msg.md` §5).

## 8. PC vs PS2

* **The vtable is offset by a whole base class.** PC slot = PS2 vtable index + 13, because PC has the
  `C_actor_track` interface in slots 0–12 and PS2 has no such class (§4); no `C_recorder` symbol or
  string exists in `SLUS_206.71`. The formula is anchored by nine CONFIRMED twins (`Release` 17,
  `Init` 18, the `SaveGame` trio 22/23/24, `GameInit` 25, `GameDone` 26, `GetActorSpeed` 28,
  `GetBoundPoints2D` 34).
* **The base stubs, slot by slot.** PS2 `__vt__7C_actor` holds 27 methods. Indices 0–24 pair with
  PC slots 13–37 by the formula above; every one of those PC slots that is not a CONFIRMED twin
  holds a shared ICF-folded stub, and each stub's stack-argument count (`ret N`) agrees with the
  PS2 parameter count (`IsPossibleColl` matches if its two `S_vector` parameters travel by value).
  Indices 25 and 26, `ChangeSceneInit` / `ChangeSceneDone`, have no PC slot: PC slot 38 holds
  `stub_noop_args8_00575c00`, a two-argument stub, against PS2's no-argument `ChangeSceneInit`,
  and slot 39 is the start of the next vtable.
* **The vptr sits in a different place.** PC keeps it at `+0x00`, and `actor_ctor` calls no base
  ctor. PS2's `C_actor` has no polymorphic base, so GCC 2.x puts the vptr *after* the data:
  `__ct__7C_actorFv` writes its vtable to `this+0x78`, and the destructor restores exactly one vptr
  and calls no base destructor (`structs/actor_track.md` §7).
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
| `+0x04` | - | - | track mode flags | PC-only: the `C_recorder` client state (`structs/actor_track.md` §7) |
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
| `+0x6c` | `+0x74` | `+0x08` | owns-its-frame flag | the `ACTOR_DUPLICATE` arm, statement-identical; both dtors release the frame only when it is set. PS2 sets `actor+0x74 = 1` immediately before `AddTemporaryActor`, statement-identical with `actor_mark_created_at_runtime` |
| `+0x6d` | `+0x75` | `+0x08` | may_broadcast | the ctors: 1 on `C_bottle` and `C_grenade`, 0 on `C_smoke_trail`, on both builds; a search of the PS2 decomp for `+ 0x75)` finds no reader on an actor |
| - | `+0x48` | - | a heap array | PS2-only: `__dt__7C_actorFv` frees it with `delete[]`; the PC dtor has no such step |
| - | `+0x6b` | - | a byte | PS2-only: zeroed by the PS2 ctor, not by the PC one |
| - | `+0x6c` | - | a byte | PS2-only: zeroed by the ctor, set to 2 by `C_game::VyhjebejTemporaryho` |

* **Both builds treat `+0x5d` non-zero as alive** — the polarity is not inverted.
* **`ShowCarEnergy` corroborates `+0x5e ↔ +0x62`.** PS2 `ShowCarEnergy__5C_carFv` opens
  `if (this->+0x62) return 0.0f;` exactly where PC `car_get_condition_fraction` reads `+0x5e` — a
  second, independent body confirming both the pair and the semantics: a finalised actor reports
  zero condition. The setter is `actor_tick`.
* **The PS2 has no out-of-line accessors** for the fields of §3b or for `actor_set_direction`,
  `actor_get_direction`, `actor_get_cached_world_pos`, `actor_set_cached_world_pos` and
  `actor_get_global_list`: it reads and writes the fields inline. `actor_get_global_list` walks
  `global_actors__7C_actor`, a static member of `C_actor` that PS2 `Actor_SaveGameLoad` scans
  exactly as `actor_ref_save_resolve` scans `g_actor_global_list_begin`.
* **The networking subsystem is PC-only**, so `actor_apply_net_guid_table`, the two event posters
  and `actor_state_event_payload_ctor` have no PS2 twin.
* **The element type of §5a comes from a PS2 symbol.** The insert symbol
  `insert__…FPQ27C_actor16S_posscoll_frame…` names `C_actor::S_posscoll_frame` outright.
* **The noun *sector* of §2c is backed by a symbol:** PS2
  `SetFrameSectorPos__9I3D_sceneCFP9I3D_frameRC8S_vector` calls the scene's `GetSector(pos)` and
  then `frame->LinkTo(sector, 1)`, and its PC twin `i3d_scene_set_frame_sector_pos` (`0x005c8600`)
  is called on `*(g_act_mission+0x10)`.
* **Static construction differs by toolchain.** None of the eight lifecycle functions of §3a that
  manage the two file-scope vectors has a PS2 twin, because GCC uses `__sti__` routines and the
  `.ctors` section instead of MSVC's static-init quartet. A toolchain artifact, not a missing
  method.
* **The translation unit's name** comes from the PS2 symbol `S_hlp$2695C_actor_cpp`, and the
  free-helper alignment of §3d is monotone on both builds.
* **`S_anim_info`'s internal field order differs** between builds (§3d): PS2 reads the id at
  `+0x04`, PC at `+0x00`, while the algorithm over it does not. The type name comes from PS2
  (`std::vector<11S_anim_info>`); the PC decomp shows only a stride-8 record.
* **The identical-sibling trap in the free-helper tail (§3d).** PS2 has two byte-identical
  `rand()/RAND_MAX` functions — `rnd__Fv` and `RandFloat` at `0x00437f60`, roughly 3 MB away. A
  body-diff cannot separate them; what identifies `rnd__Fv` is the emission position plus the
  `rnd__Fi`/`rnd__Fv` **overload pairing**, which matches PC's adjacent `rand_range` /
  `rand_float`. `rand_float`'s scale constant differs only because MSVC's `RAND_MAX` is `0x7fff`.

## 9. Related

- `structs/actor_track.md` — the PC-only base class holding vtable slots 0–12; `structs/recorder.md`
  — the keyframe-track system it is a client of.
- `structs/entity.md` — the kind-value enum and per-type properties (kind 1 = `C_actor`);
  `structs/human.md` — the `C_human` layer above this one.
- `structs/i3d_frame.md` — the `I3D_frame` at `+0x68`, including the frame name at `frame+0x100`.
- `structs/program.md` §8 — the `C_program` translation unit that hosts `actor_set_cached_world_pos`.
- `structs/mission.md`, `structs/sound_table.md` — the other two classes `folded_get_dword_0x10` serves.
- `structs/net_event_msg.md` §5 — the event-message layer of §7.
- `structs/driver.md`, `structs/vehicle.md`, `structs/car.md` — `C_actor` subclasses.
- `basic/artifacts/game-loop.md` — where `actor_tick` is driven from.

## 10. Open items

The class has no open item.
