# `C_PhThing` — the physics-body base ("**Ph**ysical **Thing**")

**Created** 2026-08-05 17:00 +0200 · **Modified** 2026-09-17 22:52 +0200

The shared rigid-body base behind everything in the game that falls, rolls, tumbles and
collides on its own: dropped car parts, crash debris, boxes, dropped weapons and scripted
physics props. The base itself is **abstract** — the script commands `CREATE_PHYSICALOBJECT` /
`DESTROY_PHYSICALOBJECT` (dispatcher cases `0xcc` / `0xcd`) construct the subclass
**`C_public_physics`** (kind `0x20`, `structs/public_physics.md`), not this class directly.

| | |
|---|---|
| PC prefix | **`physical_object_`** |
| Base class | `C_actor` (`structs/actor.md`); `physical_object_ctor` → `actor_ctor` |
| PC kind | none of its own — abstract; each subclass writes its own kind |
| PC vtable | `.rdata 0x0063c840` (39 slots) |
| PC translation unit | `[0x004be840, 0x004c6b50)` — **35 function starts**, bounded at both ends by a CRT static-init/atexit pair |
| PS2 class | `C_PhThing`, 26 methods, all mapped (§3) |

The PC prefix states what the class *is* rather than transliterating the PS2 tag: `PhThing`
tells a reader nothing, and the expansion is not a guess (§1).

## 1. What "Ph" means — the evidence

| evidence | what it establishes |
|---|---|
| **`CREATE_PHYSICALOBJECT` / `DESTROY_PHYSICALOBJECT`** — script command names present in **both** binaries | the team's own designer-facing word for these objects is **PHYSICAL OBJECT**, i.e. the *adjective* |
| the Mafia Script Data Reference glosses `destroy_physicalobject xx` as *"removes physical object xx"* (Czech *"Odstraní fyzikální objekt xx"*) | independent confirmation of the same reading |
| the sibling class is spelled **`C_public_physics`** in full | when the team means the *noun* "physics" they write it out ⇒ the abbreviated `Ph` carries the adjective |
| `Thing` ≈ `Object`; the file is **`C_PhThings.cpp`** (plural) | "physical things" — a family, which is what the five subclasses are |
| `fyz` / `Fyz` are **absent from both binaries** | not a Czech abbreviation of *fyzikální*, despite the studio being Czech |
| the only other `Ph*` symbol is `C_PData_Phase`, where `Ph` merely starts "Phase" | no competing expansion ("Phase Thing", "Phantom") survives the method set |
| method set: `MoveBox`, `MoveWheel`, `GetRVelMassJ`, `ExternalImpuls`, `CollisionFilter`, `InitBox`, `InitWheel` | rigid-body dynamics — the domain matches |

**`C_PhThing` = "Physical Thing".**

The two dispatcher cases are byte-confirmed from the game's own keyword table
(`script-mapping/artifacts/opcode-table.md`) and written up in
`script-mapping/artifacts/opcodes-verified.md`. Case `0xcc` must construct one of the five
subclasses below, selected by one of its four arguments (`create_physicalobject ww, xx, yy, zz`
per the Mafia Script Data Reference); reading that one case would confirm the whole kind→class
mapping from the script side, independently of the kind byte (§8).

## 2. The five subclasses — a closed set on both builds

Counting callers of the base constructor gives a closed candidate space on each side, which
is far cheaper than chasing one class at a time. Both builds have **exactly five**, and all
five have a named PC method set and a body-diffed constructor.

| PC ctor | PC vtable | kind | PS2 subclass | evidence / write-up |
|---|---|---|---|---|
| `dropout_ctor` (0x0048a530) | 0x0063c320 | `0x11` | `C_DropOut` | `recorder_new_track` prefix `drp%d`; `structs/dropout.md`, 12/12 PS2 methods |
| `crash_obj_ctor` (0x004786c0) | 0x0063bf68 | `0x1a` | `C_crash_obj` | `structs/crash_obj.md`, 19/19 PS2 methods |
| `box_new_ctor` (0x0045f160) | 0x0063baa0 | `0x23` | `C_box_new` | PS2 ctor writes `0x23` at `+0x50`; `structs/box_new.md` |
| `public_physics_ctor` (0x004cde20) | 0x0063c9b8 | `0x20` | `C_public_physics` | PS2 ctor writes `0x20`; `structs/public_physics.md`, 14/14 PS2 methods. The `create_physicalobject` class: the general-purpose subclass that turns any existing scene frame into a physics prop ¹ |
| `drop_in_weapon_ctor` (0x0048a0d0) | 0x0063c280 | `0x25` | `C_drop_in_weapon` | PS2 ctor writes `0x25`; `structs/drop_in_weapon.md`, 14/14 PS2 methods; also in `structs/entity.md` |

¹ `C_public_physics` was identified not from either binary but from the shipped Czech script
reference, whose 30-object cap for `create_physicalobject` is the literal `0x1e < size` gate in
`public_physics_game_init`. Three of its vtable slots are shared with `C_drop_in_weapon` by
`/OPT:ICF`: `0x0063c9b8` slots 13 / 15 / 18 hold `thunk_physical_object_ai_tick`,
`thunk_physical_object_net_apply_remote_state` and `thunk_physical_object_init`, the same three
5-byte forwarders, so `C_public_physics` adds nothing to `AI`, `NetDirect` or `Init` either.

## 3. Method coverage — 26/26 PS2 methods mapped, 35/35 PC starts accounted

| PC VA | PC name | slot | PS2 twin | status |
|---|---|---|---|---|
| 0x004be840 | `physical_object_build_world_inertia_tensor` | - | - | STRUCTURAL — TU-static helper |
| 0x004be910 | `physical_object_ctor` | - | `__ct__9C_PhThingFv` | CONFIRMED |
| 0x004bea00 | `physical_object_scalar_deleting_dtor` | 16 | - | STRUCTURAL — MSVC-only construct |
| 0x004bea20 | `physical_object_dtor` | - | `__dt__9C_PhThingFv` | CONFIRMED |
| 0x004bea60 | `physical_object_init` | 18 | `Init__9C_PhThingFP9I3D_frame` | CONFIRMED |
| 0x004bebd0 | `physical_object_game_init` | 25 | `GameInit__9C_PhThingFv` | CONFIRMED |
| 0x004beff0 | `physical_object_update_rest_state` | - | `FindActors__9C_PhThingFv` | CONFIRMED |
| 0x004bf0c0 | `physical_object_game_done` | 26 | `GameDone__9C_PhThingFv` | CONFIRMED |
| 0x004bf120 | `physical_object_ai_tick` | 13 | `AI__9C_PhThingFUi` | CONFIRMED |
| 0x004bf260 | `physical_object_update` | 14 | `Update__9C_PhThingFUi` | CONFIRMED |
| 0x004bfa80 | `physical_object_on_act_state_changed` | 27 | `ChangeState__9C_PhThingFv` | CONFIRMED |
| 0x004bfae0 | `physical_object_move_wheel` | - | `MoveWheel__9C_PhThingFf` | CONFIRMED |
| 0x004c1dd0 | `physical_object_move_box` | - | `MoveBox__9C_PhThingFf` | CONFIRMED |
| 0x004c2a40 | `physical_object_integrate_transform` | - | `MoveCount__9C_PhThingFfRC8S_vector` | CONFIRMED |
| 0x004c31a0 | `physical_object_accumulate_contact_impulse_diag` | - | `CollisionCount__9C_PhThingF…` | STRUCTURAL — half of a 1→2 split, §6c |
| 0x004c3890 | `physical_object_accumulate_contact_impulse_matrix` | - | `CollisionCount__9C_PhThingF…` | STRUCTURAL — half of a 1→2 split, §6c |
| 0x004c4000 | `physical_object_init_wheel` | - | `InitWheel__9C_PhThingFP13S_THING_WHEEL` | CONFIRMED |
| 0x004c41a0 | `physical_object_init_box` | - | `InitBox__9C_PhThingFP11S_THING_BOX` | CONFIRMED |
| 0x004c4880 | `physical_object_on_collision` | 30 | `Collision__9C_PhThingF…` | CONFIRMED |
| 0x004c4b10 | `physical_object_hit` | 31 | `Hit__9C_PhThingF10E_hit_type…` | CONFIRMED |
| 0x004c5260 | `physical_object_get_contact_terms` | - | `GetRVelMassJ__9C_PhThingF…` | CONFIRMED |
| 0x004c53f0 | `physical_object_apply_impulse_at_point` | - | `ExternalImpuls__9C_PhThingF…` | CONFIRMED |
| 0x004c5590 | `physical_object_remove_surrounding_actor` | - | `InvalidateActor__9C_PhThingFP7C_actor` | CONFIRMED |
| 0x004c5640 | `physical_object_collision_filter` | - | `CollisionFilter__9C_PhThingF…` | CONFIRMED |
| 0x004c5670 | `physical_object_refresh_world_transform` | - | `UpdateImportantVariables__9C_PhThingFv` | CONFIRMED |
| 0x004c5750 | `physical_object_save_game_get_size` | 22 | `SaveGameGetSize__9C_PhThingFv` | CONFIRMED |
| 0x004c5760 | `physical_object_save_game_save` | 23 | `SaveGameSave__9C_PhThingFPPv` | CONFIRMED ¹ |
| 0x004c5940 | `physical_object_save_game_load` | 24 | `SaveGameLoad__9C_PhThingFPPv` | CONFIRMED |
| 0x004c5d90 | `physical_object_broadcast_state_net_event_msg` | - | - | STRUCTURAL — PC-only, §5 |
| 0x004c5e40 | `physical_object_post_state_net_event_msg` | - | - | STRUCTURAL — PC-only, §5 |
| 0x004c5fd0 | `physical_object_net_restart_local_sim` | - | - | STRUCTURAL — PC-only, §5 |
| 0x004c6150 | `physical_object_apply_net_event_msg` | 38 | - | STRUCTURAL — PC-only, §5 |
| 0x004c6380 | `physical_object_net_snapshot_state` | - | - | STRUCTURAL — PC-only, §5 |
| 0x004c6480 | `physical_object_net_apply_remote_state` | 15 | `NetDirect__9C_PhThingFUi` | STRUCTURAL — §6d |
| 0x004c6b20 | `i3d_frame_set_direction` | - | - | foreign — an `I3D_frame` method sitting in this TU ² |

¹ Its body is 480 B against the PS2's 564 B (ratio 0.85). A mid-body split at `0x004c5777` was
a disassembler artifact, not a function: `jne 0x004c5777` at `0x004c5770` is this function's
`if (cVar2 == '\0') return 0;` early exit, the bytes there are `mov word ptr [ebx], 4` — the
savegame version tag — and the address has 0 `E8` callers and 0 datarefs.

² Receiver byte-proven an `I3D_frame`: `mov ecx,[ebp+0x68]` (= `this->frame`) from
`physical_object_move_wheel`, and `mov ecx,[esi+0x60]` from `camera_apply_desired_pos`, outside
this TU.

**How the 35 starts reconcile:** 27 twin-mapped (26 PS2 methods, `CollisionCount` → 2) + 1 MSVC
scalar deleting dtor + 1 TU-static helper + 5 PC-only network members + 1 foreign helper.

### 3a. Slots 19/20/21 are folded no-op stubs

`SaveData`, `LoadData` and `DuplicateFrom` are **not overridden** by this class — each slot
holds a shared folded stub. That is consistent with `structs/box_new.md` §2a, where
`C_box_new` *does* override 19/20/21 with real bodies.

## 4. Field layout

Confidence: **[H]** a writer *and* a reader traced · **[M]** one side traced · **[U]** unread.
Everything below is **PC**; the cross-build mapping is in §6.

### 4a. Inherited from `C_actor` (used pervasively here)

| offset | field | conf |
|---|---|---|
| `+0x10` | `kind` — written by each subclass ctor | H |
| `+0x1c` | `act_state` — 0 awake, 1 asleep | H |
| `+0x20` | non-zero ⇒ **this body is simulated by a remote peer** | H |
| `+0x24` `+0x28` `+0x2c` | world position (mirrored into the I3D frame) | H |
| `+0x30` `+0x34` `+0x38` | forward axis, renormalised every integration step | H |
| `+0x68` | `I3D_frame *` — the scene frame this body drives | H |

### 4b. The body's own state

| offset | field | conf |
|---|---|---|
| `+0x6d` | network snapshot valid | M |
| `+0x70` | **body type — 1 = wheel, otherwise box.** The selector `physical_object_ai_tick` and both `Move*` functions branch on | H |
| `+0x75` | awake flag (the rest-timer latch) | H |
| `+0x76` | **use the FULL inverse-inertia matrix** rather than the diagonal. Decided by `physical_object_init_box`; cleared by `physical_object_init_wheel`. Selects between the two contact solvers — §6c | H |
| `+0x77` | box configured | M |
| `+0x78` | local-space centre offset (→ `+0x84` each refresh) | M |
| `+0x84` `+0x88` `+0x8c` | **world centre of mass** — the origin every contact arm is measured from | H |
| `+0x90` `+0x94` `+0x98` | last latched position for the "has it moved?" rest test | H |
| `+0x9c` | age in ms since the last significant motion; sleeps at 4000 | H |
| `+0xa0` … `+0xb4` | the world basis rows, refreshed from the frame | H |
| `+0xb8` | **`tDynamicCollObject`** (0x12 dwords) — registered in `g_collision`, kind `0x40` | H |
| `+0x10c` | cached world matrix (16 floats) | H |
| `+0x14c` | cached **inverse** world matrix (16 floats) | H |
| `+0x18c` | **max sub-step length in seconds (0.028f)** — the fixed-Δt granularity | H |
| `+0x190` | **mass** | H |
| `+0x194` `+0x198` `+0x19c` | **inverse inertia, diagonal form** | H |
| `+0x1a0` | **inverse inertia, full world tensor** (16 floats) — precomposed by `physical_object_build_world_inertia_tensor` | H |
| `+0x1e0` `+0x1e4` `+0x1e8` | **linear velocity** | H |
| `+0x1ec` `+0x1f0` `+0x1f4` | **angular velocity** | H |
| `+0x1fc` | restitution, default **1.05** | H |
| `+0x200` | friction, default **0.03** | H |
| `+0x204` | derived wheel constant, `width × mass × 37.03704` | H |
| `+0x208` `+0x20c` `+0x210` | wheel contact offset, `(0,−1,0) × mass × width` | H |
| `+0x214` | wheel width | H |
| `+0x220` | wheel radius | H |
| `+0x2ec` | at-rest lifetime accumulator, in seconds | H |
| `+0x2f0` | at-rest lifetime limit (0.6 by default) | H |
| `+0x2f4` | frame-clamp ratio `83.0 / Δt`, consumed by the network layer | H |
| `+0x2f8` | effect object allocated through `g_i3d_driver` slot 0x4c | M |
| `+0x2fc` `+0x300` `+0x304` | position of the frame's **deepest** hit | H |
| `+0x308` `+0x30c` | the −1-initialised id pair (`+0x30c` gates the deepest-hit record) | H |
| `+0x32c` `+0x330` | deepest-hit sub-id and magnitude | H |
| `+0x340` `+0x344` `+0x348` | **the `S_SURR_ACT` vector** — begin / end / capacity | H |

### 4c. `S_SURR_ACT` — the riding-actor record, 0x14 bytes

The PS2 template argument names the type (`gif::vector<Q29C_PhThing10S_SURR_ACT,8>`). Four
independent walks agree on the layout — `physical_object_update` (TTL tick + expiry),
`physical_object_update_rest_state` (count), `physical_object_remove_surrounding_actor`
(swap-with-last) and `physical_object_move_box` (append):

```c
typedef struct S_SURR_ACT {      /* 0x14 bytes on the PC */
    /* +0x00 */ float     pos[3];    // [H] where the actor was last seen standing
    /* +0x0c */ C_actor  *actor;     // [H] the riding actor — the removal key
    /* +0x10 */ int       ttl_ms;    // [H] counted down by the frame delta; <0 ⇒ dropped
} S_SURR_ACT;
```

**What it is for:** a body that has come to rest may not fall asleep while somebody is
standing on it, and an actor that walks away must stop pinning it awake. The vector is the
memory that makes both possible.

## 5. The network replication layer (PC-only)

Six functions, all inside this TU, all with the class base as receiver (byte-proven by an ECX
census of their callsites), none with a PS2 counterpart. They form one closed loop:

```
physical_object_game_init ─┐
                           ├─► net_snapshot_state          latch pos/quat/velocities + timeGetTime
net_session_transfer_actor_ownership (ownership) ──┘

update / on_act_state_changed ─► broadcast_state_net_event_msg ─► post_state_net_event_msg
                                                                        │  type 0x1b
   slot 38  apply_net_event_msg  ◄─────────────────────────────────────┘
                                 unpacks into the same band, derives the correction velocity
   slot 15  net_apply_remote_state   extrapolates between messages, 5.0-unit snap-back
   net_session_transfer_actor_ownership (gain) ─► net_restart_local_sim
                                 snapshot velocities → live velocities
```

### 5a. The snapshot band

| offset | field |
|---|---|
| `+0x34c` `+0x350` `+0x354` | last received / latched world position |
| `+0x358` `+0x35c` `+0x360` | last received / latched **linear** velocity |
| `+0x364` `+0x368` `+0x36c` | correction accumulator (zeroed on snapshot) |
| `+0x370` `+0x374` `+0x378` | **smoothing correction velocity**, derived on receive |
| `+0x37c` `+0x380` `+0x384` | last received / latched **angular** velocity |
| `+0x388` … `+0x394` | last received / latched orientation quaternion |
| `+0x3a0` | flag word — bit `0x8` suppress rotation blend, bit `0x40` collision event already posted this frame |
| `+0x3a4` | `timeGetTime()` at the last snapshot / message |
| `+0x3a8` `+0x3ac` | elapsed / limit of the 3000 ms hand-back timer |

### 5b. How the correction velocity is derived

`physical_object_apply_net_event_msg` computes

```
correction = (remote_pos + peer_latency_ms * remote_vel * 0.001 - local_pos)
             / (session_update_interval_ms * 0.001)
```

reading the latency from `net_session_get_peer_clock_ms` and the interval from
`net_session_game_settings_addr()+0x18`. `physical_object_net_apply_remote_state` then advances
the body on `correction + remote_vel` while the message is fresh and on `remote_vel` alone once
it is stale — snapping back to `+0x34c` outright if the error ever exceeds **5.0** units.

### 5c. Message type `0x1b` is shared, not `C_box`'s own

`physical_object_post_state_net_event_msg` builds a 0x18-byte message with the descriptor
`g_event_msg_vt_physics_state` (`.rdata 0x0063ba90`) and **type `0x1b`** — the same descriptor
and the same type `box_post_state_net_event_msg` uses, so the two classes share their
replication message. `event_msg_physics_state_ctor` (`0x005a1860`) is that message class's
**generic** constructor and takes the type as a **parameter**, `(this, type, guid, flag)`.

The descriptor's slots 2/3 are `event_msg_physics_state_serialize` / `_deserialize`, named for
the payload rather than for either owner. Payload: the actor-state base
(`{vec3 → i16, 3× i16, u32 guid, u8}`) plus a 16-byte quaternion at `+0x1c`, three quantised
velocity words, and a flag byte at `+0x38`. The receive half is
`physical_object_apply_net_event_msg`, vtable slot 38. Full write-up:
`structs/net_event_msg.md`.

## 6. PC vs PS2

### 6a. Field pairs proven by the 21 body diffs

| PC | PS2 | field |
|---|---|---|
| `+0x1c` | `+0x5c` | act_state |
| `+0x20` | `+0x60` | remotely simulated |
| `+0x24`.. | `+0x00`.. | world position |
| `+0x30`.. | `+0x10`.. | forward axis |
| `+0x68` | `+0x70` | `I3D_frame *` |
| `+0x70` | `+0x590` | body type (1 = wheel) |
| `+0x75` | `+0x614` | awake flag |
| `+0x77` | `+0x615` | box configured |
| `+0x84`.. | `+0x90`.. | world centre |
| `+0x90`.. | `+0xa0`.. | rest-test latched position |
| `+0x9c` | `+0x594` | age |
| `+0xa0`.. | `+0xb0`.. | world basis rows |
| `+0xb8` | `+0x530` | `tDynamicCollObject` |
| `+0x10c` | `+0xe0` | world matrix |
| `+0x14c` | `+0x120` | inverse world matrix |
| `+0x18c` | `+0x59c` | max sub-step (0.028f) |
| `+0x190` | `+0x5a0` | mass |
| `+0x194`.. | `+0x1e0`.. | inverse inertia diagonal |
| `+0x1e0`.. | `+0x1f0`.. | linear velocity |
| `+0x1ec`.. | `+0x200`.. | angular velocity |
| `+0x1fc` | `+0x5a8` | restitution (1.05) |
| `+0x200` | `+0x5ac` | friction (0.03) |
| `+0x204` | `+0x5b0` | derived wheel constant |
| `+0x214` | `+0x5b4` | wheel width |
| `+0x21c` | `+0x5bc` | wheel tunable |
| `+0x220` | `+0x5c0` | wheel radius |
| `+0x2ec` | `+0x5d0` | at-rest lifetime |
| `+0x2f0` | `+0x5d4` | lifetime limit |
| `+0x2f4` | `+0x5d8` | frame-clamp ratio |
| `+0x2f8` | `+0x5dc` | effect object |
| `+0x308` / `+0x30c` | `+0x5e0` / `+0x5e4` | the −1-initialised id pair |
| `+0x310` | `+0x5f4` | deepest-hit sub-id |
| `+0x32c` | `+0x610` | deepest-hit flag |
| `+0x330` | `+0x5e8` | deepest-hit magnitude |
| `+0x340`.. | `+0x400`.. | the `S_SURR_ACT` vector |

The deltas are **not constant** (`+0x478`, `+0x410`, `+0x3a0`, `+0x2e4`, `+0xc0`, and several
negative) — map fields by ROLE, never by rebasing (`structs/actor.md` §1a). `S_SURR_ACT` is
**0x14 bytes on the PC and 0x20 on the PS2**, with the actor pointer at `+0x0c` / `+0x10`
respectively; every walk in the class encodes the stride literally, so this is one of the few
places a naive rebase would silently corrupt an index.

### 6b. A 2× tuning delta, witnessed three independent times

| what | PC | PS2 | seen in |
|---|---|---|---|
| age cap before forced sleep | `0xfa1` (4001) | `0x7d1` (2001) | `update_rest_state`, `ai_tick` |
| `Update`'s own sleep gate | 4000 | 2000 | `update` |

Three functions, one ratio. Whatever this counter counts, the PC build allows twice as much
of it before a body is put to sleep.

### 6c. `CollisionCount` is ONE PS2 method and TWO PC functions

`physical_object_accumulate_contact_impulse_diag` and `_matrix` have **identical 7-parameter
signatures** and are selected by `if (*(this+0x76) == 0) … else …` inside
`physical_object_move_box`:

- the **diag** arm multiplies by the scalar `+0x194`;
- the **matrix** arm goes through `S_vector::operator*(S_matrix *)` on the `+0x1a0` tensor;
- PS2's single `CollisionCount` compiles **only** the matrix sandwich
  (`__amu__(+0x1a0)` → `_vmul(+0x1e0)` → `__amu__(+0x160)`).

Two levers disagree and are deliberately not forced:

| lever | points at |
|---|---|
| algorithm — which arm PS2 actually compiles | `_matrix` |
| callsite count — 3 on PS2, 3 for `_diag` (2 in `move_wheel`, 1 in `move_box`) | `_diag` |
| size ratio — 0.99 and 1.06, both inside the 1.0–1.6 band | inconclusive |

Both rows therefore carry the same PS2 twin, both graded **STRUCTURAL**, and both are named
after the inertia model they use; a CONFIRMED tag on either would assert a choice that has not
been made.

**The PC also precomputes what the PS2 recomposes.** PC `+0x1a0` holds the finished world
tensor, built once per step by `physical_object_build_world_inertia_tensor`; the PS2 body
rebuilds the sandwich from `+0x160` / `+0x1a0` / `+0x1e0` at every contact. That is why the
PC has a TU-static helper with no PS2 counterpart.

### 6d. `NetDirect` — a PS2 stub against 1696 PC bytes

`.rdata 0x0063c840` **slot 15** holds `physical_object_net_apply_remote_state` (1696 B).
`NetDirect__9C_PhThingFUi` is an **8-byte empty body** on the PS2.

The mapping is byte-verified by slot-role transfer: `C_bottle`'s vtable `0x0063b870` slot 15
is `bottle_net_apply_remote_state` = `NetDirect__8C_bottleFUi`, graded STRUCTURAL in
`structs/bottle.md` §3a for exactly this reason. Same slot, same role, same signature.

This is the fourth instance of the gutted direction after `C_bottle`'s network layer,
`C_box`'s, and this class's own sender/receiver pair. **A PS2 method count is an upper bound
everywhere except in the replication subsystem, where it is a lower one.** An empty PS2 body
means the twin can be matched as a **symbol** and never as a **body** — hence STRUCTURAL.

### 6e. Three arms the PC body does not have

- `physical_object_collision_filter` — PS2 additionally early-outs on `+0x617` and rejects
  kinds `0x13` / `0x1b` / `2` explicitly.
- `physical_object_remove_surrounding_actor` — PS2 additionally clears a 14-slot pointer
  table at `+0x4f0`.
- `physical_object_refresh_world_transform` — PS2 performs a third matrix copy and a
  zeroing tail.

None of these changes what the PC function *is*; they are recorded so that a future size
comparison does not read the ratio as a mismatch.

### 6f. The PC's `C_box` has no PS2 counterpart and does not use this base

`structs/box.md` documents a **PC-only** rigid-body class (kind 9) that duplicates this
base's job in 55 functions of its own and derives straight from `C_actor`. The PS2 source
manifest has no `C_box.cpp`. Read together, the two classes are a refactor caught in the
act: `C_box` is the pre-`C_PhThing` implementation, `C_box_new` is the same feature rebuilt
on the shared base, and the port shipped only the second.

## 7. Related

`structs/box_new.md` (the subclass that supplied the first confirmed twins here) ·
`structs/box.md` (the PC-only class that does the same job **without** this base) ·
`structs/dropout.md` · `structs/crash_obj.md` · `structs/drop_in_weapon.md` ·
`structs/public_physics.md` · `structs/bottle.md` §3a (the anchor for the `NetDirect` slot-15
role) · `structs/entity.md` (the kind enum) · `structs/actor.md` (the base of this base) ·
`structs/net_event_msg.md`.

## 8. Open items

| item | state |
|---|---|
| the `create_physicalobject` dispatcher case body | case `0xcc` is confirmed, but the case body has not been read; it would confirm the kind→class mapping from the script side |
| `+0x3a0` bit meanings | bits `0x1`, `0x8`, `0x10`, `0x40` all have writers; only `0x40` (collision event posted) has a confirmed reader |
| `+0x1f8` / `+0x21c` / `+0x224` | wheel tunables copied straight out of `S_THING_WHEEL`; no consumer traced |
| `S_THING_BOX` / `S_THING_WHEEL` | the PS2 mangled names give the type names; the field *order inside the structs* differs between the two builds' stack layouts and is not reconciled |
