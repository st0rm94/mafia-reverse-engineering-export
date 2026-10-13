# The network event message — the PC's multiplayer replication layer

**Created** 2026-08-26 20:52 +0200 · **Modified** 2026-10-05 15:41 +0200

PC prefix **`event_msg_`**. Core range `[0x005929c0, 0x00594740)`. **PC-only** — see §7.

## 1. The class

One heap object, `operator_new(0x18)`, built by `event_msg_ctor` and its twelve per-descriptor
subclass constructors; destroyed by `event_msg_dtor` (`0x0059da00`), which sits in a one-function
COMDAT of its own.

| off | size | field | evidence |
|---|---|---|---|
| `+0x00` | 4 | **vtable** (the "descriptor") | every ctor's last store |
| `+0x04` | 4 | **body length** on the write side / **read cursor** on the read side | `event_msg_serialize_bytes` advances it; `event_msg_deserialize_bytes` advances it |
| `+0x08` | 4 | **message type** | `piVar[2] = 0x28` in every producer; slot 0 appends it, slot 1 reads it back |
| `+0x0c` | 4 | **owned body buffer** | `crt_realloc`'d by `event_msg_serialize_bytes`; NULL at construction |
| `+0x10` | 4 | **addressee peer id** | most producers store `g_net_session+0x274`, the host's peer id; the per-peer senders store the peer's own id. `net_session_send_event_msg_direct` sends to it, and a relaying `net_session_post_event_msg` skips it |
| `+0x14` | 1 | **relay flag** | set to 1 by the host when re-broadcasting |

**The vtable is 4 slots:**

| slot | role | base implementation |
|---|---|---|
| 0 | `Serialize(payload, len)` | `event_msg_serialize_header` — append the type, then a raw blob |
| 1 | `Deserialize(dst, wire, len)` | `event_msg_deserialize_header` — read the type out of `wire`, cursor = 4, then a raw blob |
| 2 | `SerializePayload(payload)` | per-message override |
| 3 | `DeserializePayload(dst, wire)` | per-message override |

Two descriptors override nothing and run on the base pair directly: `g_event_msg_vt_base` and
`g_event_msg_vt_race_camera`.

### 1a. The write / read primitives

| function | direction | note |
|---|---|---|
| `event_msg_serialize_bytes` | write | `realloc(buf, len+n)`, `rep movs`, `len += n`. **99 callsites, all in-TU** |
| `event_msg_serialize_bytes_at` | write | at an absolute offset, does NOT commit `len` — the back-patch primitive |
| `event_msg_serialize_cstring` | write | `strlen + 1`, terminator included |
| `event_msg_deserialize_bytes` | read | `src = base + cursor`; the base is **caller-supplied**, never `this->buf` |
| `event_msg_deserialize_cstring` | read | advances by `strlen + 1` |
| `event_msg_set_body_from_wire` | neither | adopts a received buffer verbatim; **27 callsites, all the host relay path** |

## 2. Direction of the verbs — the evidence

`event_msg_serialize_header` (slot 0), disassembled:

```
0x00592b40  mov esi, ecx ; push 4 ; lea eax,[esi+8] ; push eax ; call 0x5929f0
```

and `0x5929f0` (`event_msg_serialize_bytes`) is `realloc` + `rep movs` + `len += n`. It **writes**.
Its partner `event_msg_deserialize_header` reads `this->type` out of a *supplied* buffer.

Three independent confirmations, no two sharing a premise:

1. **Every producer calls slot 2, then posts** —
   `(**(code**)(*msg + 8))(payload); net_session_post_event_msg(msg);`
2. **The string helpers split cleanly by side.** `event_msg_serialize_cstring` has three callers,
   all payload writers; `event_msg_deserialize_cstring` has three, all payload readers.
3. **The hit producer agrees.** `actor_post_hit_event` wraps a payload as message type `0x1c`,
   serialises it and posts it — and the payload function it calls is the slot-2 writer.

## 3. The descriptor roster — a closed set of 39

Scanning `Game.exe` for the address of `event_msg_serialize_header` as a pointer returns exactly 39
dwords, every one the slot 0 of a descriptor. Ordered by address.

| descriptor | name | type(s) | payload class |
|---|---|---|---|
| `0x0063b6dc` | `g_event_msg_vt_actor_act_state` | `0x13` | two dwords |
| `0x0063b6fc` | `g_event_msg_vt_hit` | `0x1c` | 0x38-byte hit payload |
| `0x0063b70c` | `g_event_msg_vt_collision` | `0x1d` | 0x30-byte collision payload |
| `0x0063b92c` | `g_event_msg_vt_bottle_state` | `0x16` | actor state + `u8` + `u32` |
| `0x0063b93c` | `g_event_msg_vt_bottle_collision` | `0x15` | actor state + `u8` |
| `0x0063ba90` | `g_event_msg_vt_physics_state` | `0x1b` | actor state + quat + 3 quantised velocities + `u8` |
| `0x0063c21c` | `g_event_msg_vt_door_state` | `0x34` | `{u32 guid, u8 open}` |
| `0x0063e0bc` | `g_event_msg_vt_dword_pair_reversed` | `0x29`, `0x2a`, `0x36` | `{value, guid}` |
| `0x0063e130` | `g_event_msg_vt_u8` | `0x11`, `0x12`, `0x39` | one byte |
| `0x0063e140` | `g_event_msg_vt_base` | many | none — the base pair |
| `0x0063e148` | `g_event_msg_vt_actor_guid_table` | `0xc` | GUID pairs. **Send-only** |
| `0x0063e158` | `g_event_msg_vt_scene_load` | `0xb` | 8 B `{mission name, param}` (§3b) |
| `0x0063e178` | `g_event_msg_vt_chat` | `1` | `{char text[200], u8}` |
| `0x0063e188` | `g_event_msg_vt_session_start` | `6` | none |
| `0x0063e198` | `g_event_msg_vt_ping` | `7` | two dwords (timestamps) |
| `0x0063e1a8` | `g_event_msg_vt_slot_flag` | `0x2e`, `0x2f` | `{u8 flag, u8 slot}` |
| `0x0063e1b8` | `g_event_msg_vt_session_setting` | `0x31` | `{u8 selector, u32 value, u8 slot}` |
| `0x0063e1c8` | `g_event_msg_vt_ai_racer_add` | `0x32` | `{char name[0x20], u32, u32}` |
| `0x0063e1d8` | `g_event_msg_vt_ai_racer_remove` | `0x33` | `{u32 index}` |
| `0x0063e1e8` | `g_event_msg_vt_timeout_watchdog` | `9` | one byte (§3b) |
| `0x0063e1f8` | `g_event_msg_vt_scene_done` | `0x10` | 16 B `{slot, mission name, param}` (§3b) |
| `0x0063e208` | `g_event_msg_vt_actor_owner` | `0x26` | 8 B `{actor guid, new owner slot}` (§3b) |
| `0x0063e224` | `g_event_msg_vt_actor_state` | `0x14`, `0x21`, `0x23` ¹ | the actor-state base |
| `0x0063e234` | `g_event_msg_vt_car_state` | `0x1e`, `0x1f` | physics state + car extras |
| `0x0063e254` | `g_event_msg_vt_car_glass_break` | `0x27` | `{guid, part code, part dword, vec3}` |
| `0x0063e264` | `g_event_msg_vt_car_hit` | `0x2c` | two dwords |
| `0x0063e274` | `g_event_msg_vt_car_part_dropout` | `0x2d` | variable block |
| `0x0063e284` | `g_event_msg_vt_car_damage` | `0x28` | two back-patched runs |
| `0x0063e294` | `g_event_msg_vt_car_collision_damage` | `0x2b` | `{vec3, vec3, u32}` queue |
| `0x0063e2a4` | `g_event_msg_vt_car_explosion` | `0x24` | `{value, guid}` |
| `0x0063e2b4` | `g_event_msg_vt_player_add` | `4` | one peer record |
| `0x0063e2c4` | `g_event_msg_vt_player_list` | `3` | the peer roster + session mode |
| `0x0063e2d4` | `g_event_msg_vt_fire` | `0x1a` | explosion payload + one dword |
| `0x0063e2e4` | `g_event_msg_vt_explosion` | `0x19` | `{vec3, u32, u32}` |
| `0x0063e2f4` | `g_event_msg_vt_grenade_spawn` | `0x17` | actor state + `u32` |
| `0x0063e304` | `g_event_msg_vt_race_finish` | `0x37`, `0x38` | `{u8 slot, u8 place, u32 clock}` |
| `0x0063e314` | `g_event_msg_vt_unsent_type_0x25` | `0x25` | **never sent** (§3a) |
| `0x0063e324` | `g_event_msg_vt_peer_latency_table` | `8` | one dword per peer |
| `0x0063e334` | `g_event_msg_vt_race_camera` | `0x3a` | none — sent via slot 0 as an 8-byte blob |

¹ Received as type `0x14`, and sent as the reduced car snapshot `0x21` / `0x23` (§4); its constructor
takes the type as a parameter and serves as the base of the actor-state subclasses.

The dispatcher's case labels are the type minus one (§6), so the type-`0x10` arm is `case 0xf` and
the `g_event_msg_vt_u8` arms are `case 0x10` / `case 0x11`.

### 3a. Two descriptors worth a second look

**`g_event_msg_vt_unsent_type_0x25`.** A scan of the image for the address as a pointer finds one
reference: `0x0059f3d1`, inside `net_session_handle_event_msg`. There is no producer and no
constructor — and the receiving arm (`case 0x24`) parses the payload into a stack local and falls
through to `break` without reading it. Both halves are inert (§9).

**`g_event_msg_vt_actor_guid_table` is send-only by construction** — slot 3 of its descriptor is the
shared stub `stub_ret0_args8_00575a60`. The receive side is a separate free function,
`actor_apply_net_guid_table`, called by the SENDER right after serializing.

### 3b. The four session descriptors

Each has exactly one producer, and the producer names the descriptor.

| descriptor | type | payload | sole producer |
|---|---|---|---|
| `g_event_msg_vt_scene_load` | `0xb` | 8 B `{mission name, param}` | `race_net_load_sync` |
| `g_event_msg_vt_timeout_watchdog` | `9` | 1 B | `net_session_enable_timeout_watchdog` / `net_session_disable_timeout_watchdog` |
| `g_event_msg_vt_scene_done` | `0x10` | 16 B `{slot, mission name, param}` | `net_session_post_scene_done` |
| `g_event_msg_vt_actor_owner` | `0x26` | 8 B `{actor guid, new owner slot}` | `net_session_transfer_actor_ownership` |

The `0xb` / `0x10` pair is a matched announce/withdraw: `race_net_load_sync` posts `0xb` when this
machine has a scene, `game_done` posts `0x10` when it lets it go, and the type-`0x10` receive arm
calls `net_deactivate_actors_of_peer` for the departing slot.

### 3c. Payload default constructors

Three functions right beside the dispatcher are the default constructors of the payload structs the
arm below each one deserialises into:

| function | dispatcher case | type |
|---|---|---|
| `event_msg_car_damage_payload_init` (`0x005a18f0`) | `0x27` | `0x28` |
| `event_msg_dword_pair_payload_init_005a1930` (`0x005a1930`) | `0x28` / `0x29` / `0x35` | `0x29` / `0x2a` / `0x36` |
| `event_msg_car_collision_damage_payload_init` (`0x005a1970`) | `0x2a` | `0x2b` |

**The family has byte-identical duplicates because `/OPT:ICF` was not applied to them.**
`event_msg_dword_pair_payload_init_005a1930` is byte-identical to `event_msg_dword_pair_payload_init`
(`0x00594650`), and `event_msg_car_damage_payload_init` to `event_msg_door_state_payload_init` (`0x005946c0`).
Filtering the image's byte-identical function groups down to bodies with no relative branch or call
leaves four genuinely foldable pairs, all unfolded.

`folded_get_dword_0x18` (`0x005a17d0`) is not a payload helper: it is a one-line `*(this+0x18)`
getter serving receivers of two owner classes. At two of its three callsites it reads the `C_actor`
network GUID (top byte = owning peer slot); at the third, in `program_process_command`, it reads
`+0x18` of `C_mission`'s settings block, the particles quality (`structs/mission.md` §2). The two
receivers share no base class and the body occurs once in the image, so the one address is
identical-code folding of two inline accessors. The unfolded duplicates above do not contradict
it: they are ordinary functions of this translation unit, and only inline functions, which
every object file emits on its own, were folded in this build. That split is inferred, not
measured; it is the one model that fits both observations.

## 4. Message types, by feature

| type | what it carries | producer |
|---|---|---|
| `1` | chat text | `net_session_send_chat_message` |
| `3` | the whole peer roster + session mode | `net_session_receive_messages` |
| `4` | one peer joining | `net_session_receive_messages` |
| `5` | a player left; the receiver removes the peer | `net_session_remove_peer` |
| `6` | go in-game (`session+0x100 = 5`) | `net_session_start` |
| `7` | clock-sync ping | `net_session_ping_tick` |
| `8` | the peer latency table | dispatcher |
| `9` | timeout watchdog on / off | `net_session_enable_timeout_watchdog` / `_disable_timeout_watchdog` |
| `0xa` | the watchdog-on message is acknowledged: the receiver sets `g_net_session+0x280`, which arms `net_session_ping_tick` | the type-`9` receive arm |
| `0xb` | the scene identity | `race_net_load_sync` |
| `0xc` | the actor GUID remap table | `race_net_load_sync` |
| `0xd` | a client acknowledges the actor GUID table | the type-`0xc` receive arm |
| `0xe` | every player is synchronised; a client enters the running state | `race_net_load_sync`, the type-`0xd` receive arm |
| `0xf` | pre-start synchronisation: the receiver calls `racing_set_phase(0)` (the start lights) and `racing_reset_phase_timer(0)` | `racing_tick` phase 2 |
| `0x10` | a peer's scene released | `net_session_post_scene_done` |
| `0x11` | a human racer is out: the receiver kills the racer bound to that player slot | `racing_tick` (car wrecked or driver dead), `race_game_loop_net` (the in-race menu's code `0x23`) |
| `0x12` | an AI racer is out | `racing_tick` |
| `0x13` | an actor's act state changed | `actor_set_act_state` |
| `0x14` | a state snapshot applied to kind-`0x15` actors (`C_dog`) | none: only its receive arm and the host's relay of it build this type |
| `0x15` / `0x16` | `C_bottle` collision / state | `bottle_post_collision_net_event_msg` / `bottle_post_state_net_event_msg` |
| `0x17` | thrown object spawned | `game_new_grenade` |
| `0x19` / `0x1a` | explosion / fire | `game_new_explosion` / `game_new_fire` |
| `0x1b` | rigid-body snapshot | `box_post_state_net_event_msg` / `physical_object_post_state_net_event_msg` |
| `0x1c` / `0x1d` | hit / collision event | `actor_post_hit_event` / `actor_post_collision_event` |
| `0x1e` / `0x1f` | `C_car` snapshot | `car_post_state_net_event_msg` |
| `0x20`–`0x23` | the reduced `C_car` snapshot for far peers, applied to kind-4 actors through vtable `+0x98`. Bit 1 selects the payload: the velocity scaled by `car+0x20b4` while the car moves (speed squared at least 0.001), the forward axis `car+0xd28` when it is nearly at rest. Bit 0 is bit `0x100` of `car+0x2198`, and the sender runs only while that bit is set, so only `0x21` and `0x23` are sent; the host relays both | `car_post_reduced_state_net_event_msg`, from `car_broadcast_state_net_event_msg` |
| `0x24` | car explosion | `car_post_explosion_net_event_msg` |
| `0x25` | never sent | - |
| `0x26` | actor ownership transferred | `net_session_transfer_actor_ownership` |
| `0x27` | car glass broken | `car_post_break_glass_net_event_msg` |
| `0x28` | car damage table | `car_post_dirty_net_event_msgs` |
| `0x29` / `0x2a` | two car scalars (`car+0x2a4`, `car+0x2dc`) | `car_post_dirty_net_event_msgs` |
| `0x2b` | queued collision damage | `car_post_collision_damage_net_event_msg` |
| `0x2c` | car hit | `car_post_hit_net_event_msg` |
| `0x2d` | car part detached | `car_post_part_dropout_net_event_msg` |
| `0x2e` / `0x2f` | two lobby ready flags | `net_session_set_ready_flag` / `net_session_request_ready_flag` |
| `0x31` | a lobby setting changed | `net_session_broadcast_setting` |
| `0x32` / `0x33` | AI opponent added / removed | `net_session_add_ai_racer` / `net_session_remove_ai_racer` |
| `0x34` | door opened / closed | `door_set_state` |
| `0x36` | loading progress | `gui_load_progress_set_progress_game_init` / `_mission_open` |
| `0x37` / `0x38` | a racer finished | `racing_do_gates` |
| `0x39` | the local racer touched a death-collision volume. The sender kills its own racer (`opponent_set_death`). The receiving dispatcher has no arm for this type (its table entry is the exit), so peers never apply it; the type-`0x11` arm holds a `== 0x39` branch (spawn particle effect `0x25` when the particles quality is above 0) that this routing never reaches | `race_game_loop_net` (`C_game+0x2ad8`) |
| `0x3a` | race camera selected | inside `camera_tick`, which also writes `g_racing+0x98` |
| `0x3b` / `0x3c` | racing damage on / off | `racing_set_damage_enabled`; `net_session_receive_messages` sends the current one to a joining peer |
| `0x3d` | a racer's car recovered onto the track, relayed by the host | `opponent_do_recover` |
| `0x3e` / `0x3f` | an AI / human racer passed a gate; at the first gate the lap count rises and the best lap is tested | `racing_do_gates` |
| `0x40` / `0x41` | an AI / human racer's gate passage is taken back; at the first gate the lap count drops | `racing_do_gates` |

Every number from `1` to `0x41` has a producer except `2`, `0x14`, `0x18`, `0x25`, `0x30` and `0x35`,
and of the reduced car snapshot only `0x21` and `0x23` are sent. Every message is posted through
`net_session_post_event_msg`, except type `5`, the type-`7` echo and the host's type-`0xb` answer,
which `net_session_send_event_msg_direct` sends at once.

## 5. The wire format

A message on the wire is `[u32 type][payload]`. Everything past the type is written by the class's
slot 2 in field order; there is no length prefix and no alignment padding, so the two sides agree
only because slot 2 and slot 3 are written as mirrors.

**Quantisation.** Position and velocity triples travel as 16-bit sign-magnitude fixed point through
`net_encode_fixed16` / `net_decode_fixed16`: `(int << fracbits) | frac`, with the sign in bit
`0x8000` rather than two's complement.

**Ownership.** The top byte of an actor GUID is the owning peer's slot —
`actor_apply_net_guid_table` computes `g_net_session+0x230 != (guid >> 24)` to mark every actor this
peer does not own at `actor+0x20`, excluding kind `0x1b` (`C_entity`).

## 6. The dispatcher

`net_session_handle_event_msg` (`0x0059e120`, ~1900 lines) is `switch (*wire - 1)`: every case label
is the message type minus one. Each arm builds the matching class on the stack, calls its slot 3,
applies the result, and where the local peer is the host rebuilds the message with
`event_msg_set_body_from_wire` and re-posts it.

It is the single consumer of 27 of the 39 descriptors. Its jump table has `0x41` entries; those of
types `2`, `0x18`, `0x30`, `0x35` and `0x39` point at the exit.

## 7. PC vs PS2

**This entire subsystem is PC-only.** Four measurements, not a failed grep:

1. **The PS2 `__sinit_` source-file manifest** (173 entries, in the clear) contains **no networking
   source file** — while `g_shot.cpp`, `g_textdb.cpp`, `g_material.cpp` and the five collision files
   are all there.
2. **`NetDirect` was gutted for the port**: of the 21 PS2 classes that have one, **19 have an 8-byte
   (empty) body**; only `C_player` (16 B) and `C_door` (68 B) differ. The PS2 method count is a lower
   bound here — it loses real PC code.
3. **`GM_NetPlayer` is not a network class.** Its method set is `Draw` / `Key` / `GetType` / `Make` /
   `MakeDup` — a `GM_Component` GUI widget.
4. A `Read`/`Write`/`Serialize`/`Pack` method census over the whole PS2 symbol table returns two
   unrelated hits.

Consequence: **every PS2-twin cell in this subsystem is `-`, and that dash is a measurement.** There
is also nothing here to mark as "inlined on PC" — there is no PS2 side to have inlined it.

## 8. Related

`structs/net_session.md` (the `g_net_session` session object and its peer records) ·
`structs/car.md` (the `C_car` breakable-part and damage bands) · `structs/door.md` · `structs/bottle.md` ·
`structs/box.md` · `structs/physical_object.md` · `structs/racing.md` · `structs/camera.md`.

## 9. Open items

| item | state |
|---|---|
| type `0x25` in game | does a second peer ever send it? (§3a) |
