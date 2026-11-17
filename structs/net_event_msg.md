# `S_event_msg` — the network event message, the PC's multiplayer replication layer

**Created** 2026-08-26 20:52 +0200 · **Modified** 2026-11-17 09:15 +0100

One heap object per message: `operator_new(0x18)`, built by `event_msg_ctor` and its twelve per-descriptor subclass
constructors, destroyed by `event_msg_dtor`, which sits in a one-function COMDAT of its own. The object carries a wire body; the
payload it writes or reads is a separate plain record on the caller's stack. PC-only (§10).

| | |
|---|---|
| PC prefix | `event_msg_`, plus `hit_event_payload_` and `collision_event_payload_` for two payload records |
| PC kind (`+0x10`) | - |
| PC vtable | 39 descriptors, a closed set (§6); base `g_event_msg_vt_base` @ `0x0063e140`, 4 slots |
| PC base class | - |
| PC instance size | `0x18` |
| PC constructor | `event_msg_ctor` |
| PC translation unit | `[0x005929c0, 0x00594740)`; the constructors sit beside the dispatcher in the unit of `structs/net_session.md` |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct S_event_msg {
    /* +0x00 */ void    **vtable;      // [H] the descriptor (§6), every constructor's last store
    /* +0x04 */ uint32_t  body_cursor; // [H] body length on the write side, read cursor on the read side
    /* +0x08 */ uint32_t  type;        // [H] the message type (§7)
    /* +0x0c */ uint8_t  *body;        // [H] the owned wire body, crt_realloc'd by event_msg_serialize_bytes; 0 at construction
    /* +0x10 */ uint32_t  addressee;   // [H] peer id: most producers store g_net_session+0x274, the host
    /* +0x14 */ uint8_t   relay;       // [H] set to 1 by the host when re-broadcasting
} S_event_msg;
```

The object has no array field. Its payload records have no vtable: a slot-2 function reads one and a slot-3 function fills one, and
the wire format (§8) writes each field in its own order. A `char` array holds a name of its constant wire length, and a `[3]` / `[4]`
holds an `S_vector` or a quaternion's components.

### 1a. `S_msg_actor_state` (`0x1c`)

The base of every actor snapshot; descriptors `actor_state` (types `0x14`, `0x21`, `0x23`), and the first `0x1c` bytes of 1b and 1d to 1f.

```c
typedef struct S_msg_actor_state {
    /* +0x00 */ float     velocity[3]; // [H] linear velocity times the object's net scale; 16-bit binary fraction on the wire
    /* +0x0c */ int16_t   position[3]; // [H] from net_session_encode_position
    /* +0x14 */ uint32_t  actor_guid; // [H] the sender's actor+0x18; top byte = owning peer slot (§8)
    /* +0x18 */ uint8_t   grid_index; // [H] the nearest position-grid point; position holds the offsets from it
} S_msg_actor_state;
```

### 1b. `S_msg_physics_state` (`0x39`)

Descriptor `physics_state` (type `0x1b`); `S_msg_actor_state` first.

```c
typedef struct S_msg_physics_state {
    /* +0x1c */ float     rotation[4]; // [H] the frame's quaternion, cached at I3D_frame+0x9c
    /* +0x2c */ float     angular_velocity[3]; // [H] angular velocity times the net scale; 16-bit binary fraction on the wire
    /* +0x38 */ uint8_t   send_reason; // [H] 0 periodic, 3 after a collision, 5 relayed to every peer (§2c)
} S_msg_physics_state;
```

### 1c. `S_msg_car_state` (`0x46`)

Descriptor `car_state` (types `0x1e`, `0x1f`); `S_msg_physics_state` first.

```c
typedef struct S_msg_car_state {
    /* +0x3c */ uint16_t  wheel_contact_bits; // [H] two bits per wheel from +0x120 flags 0x10 and 0x20
    /* +0x40 */ uint32_t  drive_state; // [H] gear, steering and engine revs, packed (§2d)
    /* +0x44 */ uint16_t  light_state; // [H] the light bits of car_pack_light_state
} S_msg_car_state;
```

### 1d. `S_msg_bottle_state` (`0x21`)

Descriptor `bottle_state` (type `0x16`); `S_msg_actor_state` first.

```c
typedef struct S_msg_bottle_state {
    /* +0x1c */ float     fall_accum; // [H] the bottle's fall_accum (bottle+0x80)
    /* +0x20 */ uint8_t   send_reason; // [H] the post's reason byte, 0..5 (§2e)
} S_msg_bottle_state;
```

### 1e. `S_msg_bottle_collision` (`0x1d`)

Descriptor `bottle_collision` (type `0x15`); `S_msg_actor_state` first.

```c
typedef struct S_msg_bottle_collision {
    /* +0x1c */ uint8_t   reply_reason; // [H] 3; the owner answers with a state message of this reason
} S_msg_bottle_collision;
```

### 1f. `S_msg_grenade_spawn` (`0x20`)

Descriptor `grenade_spawn` (type `0x17`); `S_msg_actor_state` first.

```c
typedef struct S_msg_grenade_spawn {
    /* +0x1c */ uint32_t  grenade_type; // [H] game_new_grenade's type: 0/1 grenade, 2/3 molotov
} S_msg_grenade_spawn;
```

### 1g. `S_msg_hit` (`0x38`)

Descriptor `hit` (type `0x1c`); filled by `hit_event_payload_fill` in its parameter order.

```c
typedef struct S_msg_hit {
    /* +0x00 */ uint32_t  target_guid; // [H] the hit actor's actor+0x18; top byte = owning peer slot
    /* +0x04 */ uint32_t  hit_type; // [H] E_hit_type, the first argument of the Hit virtual
    /* +0x08 */ S_vector  position; // [H] first vector argument
    /* +0x14 */ S_vector  direction; // [M] second vector argument
    /* +0x20 */ S_vector  normal; // [M] third vector argument
    /* +0x2c */ float     damage; // [M] the float argument
    /* +0x30 */ uint32_t  source_guid; // [H] the attacker's actor+0x18; the Hit virtual's attacker
    /* +0x34 */ uint32_t  hit_detail; // [M] the last argument
} S_msg_hit;
```

### 1h. `S_msg_collision` (`0x30`)

Descriptor `collision` (type `0x1d`); filled by `collision_event_payload_fill`.

```c
typedef struct S_msg_collision {
    /* +0x00 */ uint32_t  actor_guid; // [H] the colliding actor
    /* +0x04 */ uint32_t  other_guid; // [H] the other actor; the Collision virtual's first argument
    /* +0x08 */ S_vector  position; // [H] first vector argument
    /* +0x14 */ S_vector  direction; // [M] second vector argument
    /* +0x20 */ S_vector  normal; // [M] third vector argument
    /* +0x2c */ float     strength; // [M] the float argument
} S_msg_collision;
```

### 1i. `S_msg_explosion` (`0x14`)

Descriptor `explosion` (type `0x19`). The wire order swaps the last two fields.

```c
typedef struct S_msg_explosion {
    /* +0x00 */ S_vector  position; // [H] the explosion's position
    /* +0x0c */ float     damage; // [H] game_new_explosion's damage
    /* +0x10 */ float     radius; // [H] game_new_explosion's radius
} S_msg_explosion;
```

### 1j. `S_msg_fire` (`0x18`)

Descriptor `fire` (type `0x1a`); the three fields of 1i first, then one dword.

```c
typedef struct S_msg_fire {
    /* +0x00 */ S_vector  position; // [H] the fire's position
    /* +0x0c */ float     damage_rate; // [H] fire_init's damage_rate
    /* +0x10 */ float     radius; // [H] fire_init stores its square as radius_sq
    /* +0x14 */ int32_t   lifetime_ms; // [H] fire_init's lifetime_ms
} S_msg_fire;
```

### 1k. `S_msg_car_glass_break` (`0x18`)

Descriptor `car_glass_break` (type `0x27`).

```c
typedef struct S_msg_car_glass_break {
    /* +0x00 */ uint32_t  actor_guid; // [H] the car's actor+0x18
    /* +0x04 */ uint32_t  part_code; // [H] the encoded part position (§2l)
    /* +0x08 */ uint32_t  part_dword; // [H] dword +0x20 of the car's part record car+0x308
    /* +0x0c */ S_vector  position; // [M] the three remaining arguments
} S_msg_car_glass_break;
```

### 1l. `S_msg_car_collision_damage` (`0x1c`)

One queue entry; descriptor `car_collision_damage` (type `0x2b`) writes a `u32` guid, a `u8` count, then this many entries, read from `car_get_collision_damage_records`, whose count is the queue's length.

```c
typedef struct S_msg_car_collision_damage {
    /* +0x00 */ S_vector  impulse; // [H] the deformation's direction and strength
    /* +0x0c */ S_vector  position; // [H] the point the receiver deforms
    /* +0x18 */ uint32_t  damage_value; // [M] the entry's dword
} S_msg_car_collision_damage;
```

### 1m. `S_msg_car_part_dropout` (`0x10`)

Descriptor `car_part_dropout` (type `0x2d`). The block's size depends on `kind`: `0x20` bytes when `kind` is 1, else `0x18`. It has its own constructor and destructor, since it owns `block`.

```c
typedef struct S_msg_car_part_dropout {
    /* +0x00 */ uint32_t  actor_guid; // [H] constructor's first argument
    /* +0x04 */ uint32_t  dropout_guid; // [H] the new dropout's actor+0x18
    /* +0x08 */ uint8_t   kind; // [H] selects the block size
    /* +0x0a */ uint16_t  part_ref; // [H] the wheel index when kind is 1, else the encoded part position
    /* +0x0c */ void *    block; // [H] a _malloc copy of the part's data, 0x18 or 0x20 bytes
} S_msg_car_part_dropout;
```

### 1n. `S_msg_door_state` (`0x05`)

Descriptor `door_state` (type `0x34`). The wire writes `open` first.

```c
typedef struct S_msg_door_state {
    /* +0x00 */ uint32_t  door_guid; // [H] the door's actor+0x18
    /* +0x04 */ uint8_t   open; // [H] the new state
} S_msg_door_state;
```

### 1o. `S_msg_value_guid` (`0x08`)

Descriptor `dword_pair_reversed` (types `0x29`, `0x2a`, `0x36`). The wire writes `guid` first. The tyre burst (type `0x2c`) runs `event_msg_dword_pair_serialize` over the opposite layout, `{guid, wheel}`, filled by `car_post_tyre_burst_net_event_msg`.

```c
typedef struct S_msg_value_guid {
    /* +0x00 */ uint32_t  value; // [H] the scalar; §7 for each type
    /* +0x04 */ uint32_t  guid; // [H] the actor
} S_msg_value_guid;
```

### 1p. `S_msg_race_finish` (`0x09`)

Descriptor `race_finish` (types `0x37`, `0x38`).

```c
typedef struct S_msg_race_finish {
    /* +0x00 */ uint8_t   slot; // [H] the racer's slot
    /* +0x04 */ uint32_t  clock; // [H] the finish clock
    /* +0x08 */ uint8_t   place; // [H] the finishing place
} S_msg_race_finish;
```

### 1q. `S_msg_session_setting` (`0x09`)

Descriptor `session_setting` (type `0x31`). The wire writes `value` first.

```c
typedef struct S_msg_session_setting {
    /* +0x00 */ uint8_t   selector; // [H] which setting
    /* +0x04 */ uint32_t  value; // [H] the setting's value
    /* +0x08 */ uint8_t   slot; // [H] g_net_session+0x230, the sender's slot
} S_msg_session_setting;
```

### 1r. `S_msg_ai_racer_add` (`0x28`)

Descriptor `ai_racer_add` (type `0x32`). The payload is the `S_net_ai_slot` record of `structs/net_session.md` §1e; its `slot_index` (`+0x28`) is not on the wire, the receiver sets it.

```c
typedef struct S_msg_ai_racer_add {
    /* +0x00 */ char      name[0x20]; // [H] the racer's name
    /* +0x20 */ uint32_t  driver_name_index; // [H] the sender's driver-name index, kept by the receiver
    /* +0x24 */ uint32_t  car_model; // [H] the AI racer's car model
} S_msg_ai_racer_add;
```

### 1s. `S_msg_chat` (`0xc9`)

Descriptor `chat` (type `1`). The wire writes `sender_slot` first, then `text` as a C string.

```c
typedef struct S_msg_chat {
    /* +0x00 */ char      text[200]; // [H] the message
    /* +0xc8 */ uint8_t   sender_slot; // [H] g_net_session+0x230, the sender's slot
} S_msg_chat;
```

### 1t. `S_msg_slot_flag` (`0x02`)

Descriptor `slot_flag` (types `0x2e`, `0x2f`).

```c
typedef struct S_msg_slot_flag {
    /* +0x00 */ uint8_t   flag; // [H] the ready flag
    /* +0x01 */ uint8_t   slot; // [H] the peer slot
} S_msg_slot_flag;
```

### 1u. `S_msg_player` (`0x2d`)

Descriptor `player_add` (type `4`), and one row of `player_list` (type `3`), which writes a `u32` count, these rows, and the `u32` session mode. The payload is the `S_net_peer` record of `structs/net_session.md` §1d: the wire carries `peer_id`, `name` and `slot`, a `player_list` row also `lobby_ready` (`+0x4e`); `+0x24..+0x2b` are not on the wire.

```c
typedef struct S_msg_player {
    /* +0x00 */ uint32_t  peer_id; // [H] the peer
    /* +0x04 */ char      name[0x20]; // [H] the player's name; a C string on the wire
    /* +0x2c */ uint8_t   slot; // [H] the player's slot byte
} S_msg_player;
```

### 1v. `S_msg_two_dwords` (`0x08`)

Descriptors `actor_act_state` (`0x13`), `ping` (`7`) and `scene_load` (`0xb`) through `event_msg_blob8_serialize`; `actor_owner` (`0x26`) by `event_msg_blob8_serialize` as well.

```c
typedef struct S_msg_two_dwords {
    /* +0x00 */ uint32_t  first; // [H] by type: actor guid, send clock, mission name, actor guid
    /* +0x04 */ uint32_t  second; // [H] by type: act state, echo clock, parameter, new owner slot
} S_msg_two_dwords;
```

## 2. Field evidence

The payload cells are the function pair that moves the field, plus the producers read for §1. The arm of `net_session_handle_event_msg`
that consumes each record is listed in §7, and is not repeated per field.

### 2a. `S_event_msg`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | vtable | H | `event_msg_ctor` and every `event_msg_*_ctor` | the callers of slots 0 to 3 | - |
| +0x4 | body_cursor | H | `event_msg_ctor`, `event_msg_deserialize_bytes`, `event_msg_deserialize_cstring`, `event_msg_deserialize_header`, `event_msg_serialize_bytes`, `event_msg_set_body_from_wire` | `event_msg_deserialize_bytes`, `event_msg_serialize_bytes`, `event_msg_serialize_bytes_at`, every slot 2 and slot 3 function's return | - |
| +0x8 | type | H | `event_msg_ctor`, `event_msg_deserialize_header`, `event_msg_set_body_from_wire` | `event_msg_serialize_header` | - |
| +0xc | body | H | `event_msg_ctor`, `event_msg_dtor`, `event_msg_serialize_bytes`, `event_msg_set_body_from_wire` | `event_msg_dtor`, `event_msg_serialize_bytes`, `event_msg_serialize_bytes_at`, `net_session_post_event_msg` | - |
| +0x10 | addressee | H | `event_msg_ctor`, the producers | `net_session_post_event_msg`, `net_session_send_event_msg_direct` | - |
| +0x14 | relay | H | `event_msg_ctor`, `net_session_handle_event_msg` | `net_session_post_event_msg` | - |

### 2b. `S_msg_actor_state`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | velocity | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_actor_state_deserialize` | `event_msg_actor_state_serialize` | - |
| +0xc | position | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_actor_state_deserialize` | `event_msg_actor_state_serialize` | - |
| +0x14 | actor_guid | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_actor_state_deserialize` | `event_msg_actor_state_serialize` | - |
| +0x18 | grid_index | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_actor_state_deserialize` | `event_msg_actor_state_serialize` | `net_session_encode_position` returns the nearest grid point's index; every receiver passes it to `net_session_decode_position` |

### 2c. `S_msg_physics_state`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x1c | rotation | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_physics_state_deserialize` | `event_msg_physics_state_serialize` | - |
| +0x2c | angular_velocity | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_physics_state_deserialize` | `event_msg_physics_state_serialize` | - |
| +0x38 | send_reason | H | `box_post_state_net_event_msg`, `car_post_state_net_event_msg`, `physical_object_post_state_net_event_msg`, `event_msg_physics_state_deserialize` | `event_msg_physics_state_serialize` | `car_update` passes 0 or 3, `box_ai_tick` and the physical objects 0, 3 or 5, the ownership transfer 3; the type-`0x1b` arm relays reason 5 to every peer, else only to the relevance partition; `car_apply_net_event_msg` starts the collision hold on 3 |

### 2d. `S_msg_car_state`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x3c | wheel_contact_bits | H | `car_post_state_net_event_msg`, `event_msg_car_state_deserialize` | `event_msg_car_state_serialize` | - |
| +0x40 | drive_state | H | `car_post_state_net_event_msg`, `event_msg_car_state_deserialize` | `event_msg_car_state_serialize` | byte 0 `gear + 1`; byte 1 the steering magnitude `& 0x3f`, bit `0x40` when `car+0x714` is negative and bit `0x80` (gear 1 or more, or neutral with `car+0xd50` at most π/2; the receiver ignores it); high word the engine revs over `max_revs` × 65535. `car_apply_net_event_msg` restores `+0x648`, the steering through `vehicle_set_steer` and `+0x634` |
| +0x44 | light_state | H | `car_post_state_net_event_msg`, `event_msg_car_state_deserialize` | `event_msg_car_state_serialize` | `car_pack_light_state` on the sender, `car_unpack_light_state` on the receiver |

### 2e. `S_msg_bottle_state`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x1c | fall_accum | H | `bottle_post_state_net_event_msg`, `event_msg_bottle_state_deserialize` | `event_msg_bottle_state_serialize` | `bottle+0x80`; `bottle_apply_net_event_msg` stores it into `net_gravity` (`+0x120`) |
| +0x20 | send_reason | H | `bottle_post_state_net_event_msg`, `event_msg_bottle_state_deserialize` | `event_msg_bottle_state_serialize` | the post's argument, 0..5; on 3 or 4 `bottle_apply_net_event_msg` re-rolls the tumble |

### 2f. `S_msg_bottle_collision`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x1c | reply_reason | H | `bottle_post_collision_net_event_msg`, `event_msg_bottle_collision_deserialize` | `event_msg_bottle_collision_serialize` | `bottle_on_collision` passes 3; the owner's `bottle_apply_net_event_msg` posts a state message with this reason |

### 2g. `S_msg_grenade_spawn`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x1c | grenade_type | H | `game_new_grenade`, `event_msg_grenade_spawn_deserialize` | `event_msg_grenade_spawn_serialize` | the type-`0x17` arm passes it back to `game_new_grenade` |

### 2h. `S_msg_hit`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | target_guid | H | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | the type-`0x1c` arm tests its top byte against the local slot and finds the hit actor by it |
| +0x4 | hit_type | H | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | the Hit virtual's (slot 31) first argument |
| +0x8 | position | H | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | - |
| +0x14 | direction | M | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | - |
| +0x20 | normal | M | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | - |
| +0x2c | damage | M | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | - |
| +0x30 | source_guid | H | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | the arm passes the actor it resolves to as the Hit virtual's attacker |
| +0x34 | hit_detail | M | `actor_post_hit_event`, `hit_event_payload_fill`, `hit_event_payload_deserialize` | `hit_event_payload_serialize` | - |

### 2i. `S_msg_collision`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | actor_guid | H | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | - |
| +0x4 | other_guid | H | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | the type-`0x1d` arm passes the actor it resolves to as the Collision virtual's (slot 30) first argument |
| +0x8 | position | H | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | - |
| +0x14 | direction | M | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | - |
| +0x20 | normal | M | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | - |
| +0x2c | strength | M | `actor_post_collision_event`, `collision_event_payload_fill`, `collision_event_payload_deserialize` | `collision_event_payload_serialize` | - |

### 2j. `S_msg_explosion`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | position | H | `game_new_explosion`, `event_msg_explosion_deserialize` | `event_msg_explosion_serialize` | - |
| +0xc | damage | H | `game_new_explosion`, `event_msg_explosion_deserialize` | `event_msg_explosion_serialize` | the type-`0x19` arm passes it back to `game_new_explosion` as the damage |
| +0x10 | radius | H | `game_new_explosion`, `event_msg_explosion_deserialize` | `event_msg_explosion_serialize` | the type-`0x19` arm passes it back as the radius |

### 2k. `S_msg_fire`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | position | H | `game_new_fire`, `event_msg_fire_deserialize` | `event_msg_fire_serialize` | - |
| +0xc | damage_rate | H | `game_new_fire`, `event_msg_fire_deserialize` | `event_msg_fire_serialize` | the type-`0x1a` arm passes it to `game_new_fire`, which forwards it to `fire_init` as `damage_rate` |
| +0x10 | radius | H | `game_new_fire`, `event_msg_fire_deserialize` | `event_msg_fire_serialize` | forwarded to `fire_init`, which stores its square as `radius_sq` |
| +0x14 | lifetime_ms | H | `game_new_fire`, `event_msg_fire_deserialize` | `event_msg_fire_serialize` | forwarded to `fire_init` as `lifetime_ms` |

### 2l. `S_msg_car_glass_break`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | actor_guid | H | `car_post_break_glass_net_event_msg`, `event_msg_car_glass_break_deserialize` | `event_msg_car_glass_break_serialize` | - |
| +0x4 | part_code | H | `car_post_break_glass_net_event_msg`, `event_msg_car_glass_break_deserialize` | `event_msg_car_glass_break_serialize` | `car_apply_net_event_msg` finds the part with `vehicle_find_part_at_encoded_position(0x20, part_code, 3)` |
| +0x8 | part_dword | H | `car_post_break_glass_net_event_msg`, `event_msg_car_glass_break_deserialize` | `event_msg_car_glass_break_serialize` | - |
| +0xc | position | M | `car_post_break_glass_net_event_msg`, `event_msg_car_glass_break_deserialize` | `event_msg_car_glass_break_serialize` | - |

### 2m. `S_msg_car_collision_damage`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | impulse | H | `car_post_collision_damage_net_event_msg`, `car_queue_collision_damage_record`, `event_msg_car_collision_damage_deserialize` | `event_msg_car_collision_damage_serialize` | the queue's first vector; `car_apply_net_event_msg` deforms along its direction with its length as the strength |
| +0xc | position | H | `car_post_collision_damage_net_event_msg`, `car_queue_collision_damage_record`, `event_msg_car_collision_damage_deserialize` | `event_msg_car_collision_damage_serialize` | the queue's second vector; the point `vehicle_deform` deforms at |
| +0x18 | damage_value | M | `car_post_collision_damage_net_event_msg`, `car_queue_collision_damage_record`, `event_msg_car_collision_damage_deserialize` | `event_msg_car_collision_damage_serialize` | `vehicle_deform`'s third argument |

### 2n. `S_msg_car_part_dropout`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | actor_guid | H | `car_post_part_dropout_net_event_msg`, `event_msg_car_part_dropout_deserialize` | `event_msg_car_part_dropout_serialize` | - |
| +0x4 | dropout_guid | H | `car_post_part_dropout_net_event_msg`, `event_msg_car_part_dropout_deserialize` | `event_msg_car_part_dropout_serialize` | `car_apply_net_event_msg` stores it as the new dropout's `+0x18` |
| +0x8 | kind | H | `car_post_part_dropout_net_event_msg`, `event_msg_car_part_dropout_deserialize` | `event_msg_car_part_dropout_serialize` | - |
| +0xa | part_ref | H | `car_post_part_dropout_net_event_msg`, `event_msg_car_part_dropout_deserialize` | `event_msg_car_part_dropout_serialize` | kind 1: the wheel index; kinds 2..6: the encoded part position of part types 2, 5, 7, 4, 6 |
| +0xc | block | H | `car_post_part_dropout_net_event_msg`, `event_msg_car_part_dropout_deserialize` | `event_msg_car_part_dropout_serialize` | - |

### 2o. `S_msg_door_state`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | door_guid | H | `door_set_state`, `event_msg_door_state_deserialize` | `event_msg_door_state_serialize` | - |
| +0x4 | open | H | `door_set_state`, `event_msg_door_state_deserialize` | `event_msg_door_state_serialize` | - |

### 2p. `S_msg_value_guid`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | value | H | `car_post_dirty_net_event_msgs`, `gui_load_progress_set_progress_game_init`, `gui_load_progress_set_progress_mission_open`, `event_msg_dword_pair_reversed_deserialize` | `event_msg_dword_pair_reversed_serialize` | type `0x36`: the loading progress |
| +0x4 | guid | H | `car_post_dirty_net_event_msgs`, `gui_load_progress_set_progress_game_init`, `gui_load_progress_set_progress_mission_open`, `event_msg_dword_pair_reversed_deserialize` | `event_msg_dword_pair_reversed_serialize` | type `0x36`: the sender's slot, `g_net_session+0x230` |

### 2q. `S_msg_race_finish`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | slot | H | `racing_do_gates`, `event_msg_race_finish_deserialize` | `event_msg_race_finish_serialize` | - |
| +0x4 | clock | H | `racing_do_gates`, `event_msg_race_finish_deserialize` | `event_msg_race_finish_serialize` | - |
| +0x8 | place | H | `racing_do_gates`, `event_msg_race_finish_deserialize` | `event_msg_race_finish_serialize` | - |

### 2r. `S_msg_session_setting`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | selector | H | `net_session_broadcast_setting`, `event_msg_session_setting_deserialize` | `event_msg_session_setting_serialize` | - |
| +0x4 | value | H | `net_session_broadcast_setting`, `event_msg_session_setting_deserialize` | `event_msg_session_setting_serialize` | - |
| +0x8 | slot | H | `net_session_broadcast_setting`, `event_msg_session_setting_deserialize` | `event_msg_session_setting_serialize` | - |

### 2s. `S_msg_ai_racer_add`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `net_session_add_ai_racer`, `event_msg_ai_racer_add_deserialize` | `event_msg_ai_racer_add_serialize` | - |
| +0x20 | driver_name_index | H | `net_session_add_ai_racer`, `event_msg_ai_racer_add_deserialize` | `event_msg_ai_racer_add_serialize` | the type-`0x32` arm calls `racing_add_racer_for_ai_slot(record, 0)`, which keeps the sender's index |
| +0x24 | car_model | H | `net_session_add_ai_racer`, `event_msg_ai_racer_add_deserialize` | `event_msg_ai_racer_add_serialize` | `S_net_ai_slot`'s `car_model` |

### 2t. `S_msg_chat`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | text | H | `net_session_send_chat_message`, `event_msg_chat_deserialize` | `event_msg_chat_serialize` | - |
| +0xc8 | sender_slot | H | `net_session_send_chat_message`, `event_msg_chat_deserialize` | `event_msg_chat_serialize` | `g_net_session+0x230`; the chat arm finds the sender by it for `"%s : %s"` |

### 2u. `S_msg_slot_flag`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | flag | H | `net_session_request_ready_flag`, `net_session_set_ready_flag`, `event_msg_slot_flag_deserialize` | `event_msg_slot_flag_serialize` | - |
| +0x1 | slot | H | `net_session_request_ready_flag`, `net_session_set_ready_flag`, `event_msg_slot_flag_deserialize` | `event_msg_slot_flag_serialize` | - |

### 2v. `S_msg_player`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | peer_id | H | `net_session_receive_messages`, `event_msg_player_add_deserialize` | `event_msg_player_add_serialize` | - |
| +0x4 | name | H | `net_session_receive_messages`, `event_msg_player_add_deserialize` | `event_msg_player_add_serialize` | - |
| +0x2c | slot | H | `net_session_receive_messages`, `event_msg_player_add_deserialize` | `event_msg_player_add_serialize` | `S_net_peer`'s `slot` |

### 2w. `S_msg_two_dwords`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | first | H | `actor_set_act_state`, `net_session_ping_tick`, `net_session_transfer_actor_ownership`, `race_net_load_sync`, `event_msg_blob8_deserialize` | `event_msg_blob8_serialize` | - |
| +0x4 | second | H | `actor_set_act_state`, `net_session_ping_tick`, `net_session_transfer_actor_ownership`, `race_net_load_sync`, `event_msg_blob8_deserialize` | `event_msg_blob8_serialize` | - |

## 3. Method family

PC-only, so there is no twin column (§10). The roster of §6 is a closed set of 39 descriptors, and every slot 2 and slot 3 override is one row below; two descriptors run on the base pair and one has the shared `stub_ret0_args8_00575a60` at slot 3.

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x005929c0 | `crt_static_init_event_msg` | file-scope | - | MSVC static-init step of the unit |
| 0x005929e0 | `crt_atexit_register_005929e0` | file-scope | - | MSVC static-init step of the unit |
| 0x00594230 | `event_msg_car_part_dropout_payload_ctor` | payload record | - | Ctor of the type-0x2d payload; deep-copies a 0x18/0x20-byte block |
| 0x005942a0 | `event_msg_car_part_dropout_payload_dtor` | payload record | - | Dtor of the type-0x2d payload |
| 0x00594630 | `event_msg_dword_pair_payload_ctor` | payload record | - | Ctor of the two-dword payload; 1 callsite, in `car_post_explosion_net_event_msg` |
| 0x005946a0 | `event_msg_door_state_payload_ctor` | payload record | - | Ctor of the type-0x34 payload `{u32 door guid, u8 is_open}`; 1 callsite, in `door_set_state` |
| 0x00596c20 | `event_msg_ctor` | instance | - | the base constructor: type parameter, cursor and body 0 |
| 0x0059da00 | `event_msg_dtor` | instance | - | frees `body`; the one-function COMDAT |
| 0x005a1830 | `event_msg_actor_state_ctor` | instance | - | Ctor for descriptor `0x0063e224`; the type is a parameter |
| 0x005a1860 | `event_msg_physics_state_ctor` | instance | - | Ctor for descriptor `0x0063ba90` (type 0x1b) |
| 0x005a1890 | `event_msg_car_state_ctor` | instance | - | Ctor for descriptor `0x0063e234` (types 0x1e/0x1f) |
| 0x005a18c0 | `event_msg_car_glass_break_ctor` | instance | - | Ctor for descriptor `0x0063e254`; type 0x27 hard-coded |
| 0x005a1900 | `event_msg_car_damage_ctor` | instance | - | Ctor for descriptor `0x0063e284`; type 0x28 hard-coded |
| 0x005a1940 | `event_msg_dword_pair_reversed_ctor` | instance | - | Ctor for descriptor `0x0063e0bc`; the type is a parameter (0x29 / 0x2a / 0x36) |
| 0x005a1980 | `event_msg_car_collision_damage_ctor` | instance | - | Ctor for descriptor `0x0063e294`; type 0x2b hard-coded |
| 0x005a19b0 | `event_msg_tyre_burst_ctor` | instance | - | Ctor for descriptor `0x0063e264`; type 0x2c hard-coded |
| 0x005a19e0 | `event_msg_car_part_dropout_ctor` | instance | - | Ctor for descriptor `0x0063e274`; type 0x2d hard-coded |
| 0x005a1a10 | `event_msg_car_explosion_ctor` | instance | - | Ctor for descriptor `0x0063e2a4`; type 0x24 hard-coded |
| 0x005a1a40 | `event_msg_hit_ctor` | instance | - | Ctor for descriptor `0x0063b6fc`; type 0x1c hard-coded |
| 0x005a1a70 | `event_msg_collision_ctor` | instance | - | Ctor for descriptor `0x0063b70c`; type 0x1d hard-coded |
| 0x005a1aa0 | `event_msg_race_camera_ctor` | instance | - | Ctor for descriptor `0x0063e334`; type 0x3a hard-coded |
| 0x005a1ad0 | `event_msg_door_state_ctor` | instance | - | Ctor for descriptor `0x0063c21c`; type 0x34 hard-coded |

### 3b. Accessors

None: T2's accessor test finds no one-field function here.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x005929f0 | `event_msg_serialize_bytes` | - | The append primitive; 99 callsites, all in-TU |
| 0x00592a40 | `event_msg_serialize_bytes_at` | - | Back-patch write at an absolute body offset; 2 callsites, both in `event_msg_car_damage_serialize` |
| 0x00592a90 | `event_msg_deserialize_bytes` | - | The read primitive; 96 callsites, all in-TU |
| 0x00592ad0 | `event_msg_deserialize_cstring` | - | Reads a C string at the cursor; 3 callsites, all deserializers |
| 0x00592b10 | `event_msg_serialize_cstring` | - | Appends a C string incl |
| 0x00592b40 | `event_msg_serialize_header` | - | Descriptor base slot 0; shared by all three event-message vtables |
| 0x00592b70 | `event_msg_deserialize_header` | - | Descriptor base slot 1; the write mirror of slot 0 |
| 0x00592ba0 | `event_msg_set_body_from_wire` | - | Host relay: adopts a received buffer as the body and re-reads the type from it |
| 0x00592bf0 | `event_msg_chat_serialize` | - | Type 1, descriptor `0x0063e178` |
| 0x00592c20 | `event_msg_chat_deserialize` | - | Type 1, descriptor `0x0063e178`, slot 3 |
| 0x00592c60 | `event_msg_player_list_serialize` | - | Type 3, descriptor `0x0063e2c4` |
| 0x00592d10 | `event_msg_player_list_deserialize` | - | Type 3, descriptor `0x0063e2c4`, slot 3 |
| 0x00592e20 | `event_msg_player_add_serialize` | - | Type 4, descriptor `0x0063e2b4` |
| 0x00592e60 | `event_msg_player_add_deserialize` | - | Type 4, descriptor `0x0063e2b4`, slot 3 |
| 0x00592eb0 | `event_msg_session_start_serialize` | - | Type 6, descriptor `0x0063e188` |
| 0x00592ed0 | `event_msg_session_start_deserialize` | - | Type 6, descriptor `0x0063e188`, slot 3 |
| 0x00592ef0 | `event_msg_u8_serialize` | - | Generic one-byte payload class; descriptors `0x0063e130` + `0x0063e1e8`, 5 message types |
| 0x00592f20 | `event_msg_u8_deserialize` | - | Generic one-byte payload class, slot 3 |
| 0x00592f50 | `event_msg_blob8_serialize` | - | Generic 8-byte payload class; descriptors `0x0063e158` + `0x0063e208` |
| 0x00592f80 | `event_msg_blob8_deserialize` | - | Generic 8-byte payload class, slot 3 |
| 0x00592fb0 | `event_msg_blob16_serialize` | - | Generic 16-byte payload class; descriptor `0x0063e1f8` |
| 0x00592fe0 | `event_msg_blob16_deserialize` | - | Generic 16-byte payload class, slot 3 |
| 0x00593010 | `event_msg_dword_pair_deserialize` | - | Generic two-dword class, slot 3; descriptors `0x0063b6dc` + `0x0063e198` + `0x0063e264` |
| 0x00593050 | `event_msg_peer_latency_table_serialize` | - | Type 8, descriptor `0x0063e324` |
| 0x005930c0 | `event_msg_peer_latency_table_deserialize` | - | Type 8, descriptor `0x0063e324`, slot 3 |
| 0x00593130 | `event_msg_actor_guid_table_serialize` | - | Type 0xc, descriptor `0x0063e148` |
| 0x005931c0 | `event_msg_actor_state_serialize` | - | Descriptor `0x0063e224` **and the base of four derived payload classes** |
| 0x00593280 | `event_msg_actor_state_deserialize` | - | Descriptor `0x0063e224`, slot 3, and the base of four derived deserializers |
| 0x00593340 | `event_msg_bottle_state_serialize` | - | Type 0x16, descriptor `0x0063b92c` |
| 0x00593370 | `event_msg_bottle_state_deserialize` | - | Type 0x16, descriptor `0x0063b92c`, slot 3 |
| 0x005933b0 | `event_msg_bottle_collision_serialize` | - | Type 0x15, descriptor `0x0063b93c` |
| 0x005933e0 | `event_msg_bottle_collision_deserialize` | - | Type 0x15, descriptor `0x0063b93c`, slot 3 |
| 0x00593410 | `event_msg_grenade_spawn_serialize` | - | Type 0x17, descriptor `0x0063e2f4`, sole producer `game_new_grenade` |
| 0x00593440 | `event_msg_grenade_spawn_deserialize` | - | Type 0x17, descriptor `0x0063e2f4`, slot 3 |
| 0x00593470 | `event_msg_explosion_serialize` | - | Type 0x19, descriptor `0x0063e2e4`, sole producer `game_new_explosion` |
| 0x005934b0 | `event_msg_explosion_deserialize` | - | Type 0x19, descriptor `0x0063e2e4`, slot 3 |
| 0x00593500 | `event_msg_fire_serialize` | - | Type 0x1a, descriptor `0x0063e2d4`, sole producer `game_new_fire` |
| 0x00593530 | `event_msg_fire_deserialize` | - | Type 0x1a, descriptor `0x0063e2d4`, slot 3 |
| 0x00593560 | `event_msg_physics_state_serialize` | - | Type 0x1b, descriptor `0x0063ba90` |
| 0x005935f0 | `event_msg_physics_state_deserialize` | - | Type 0x1b, descriptor `0x0063ba90`, slot 3 |
| 0x00593690 | `event_msg_car_state_serialize` | - | Type 0x1e/0x1f, descriptor `0x0063e234` |
| 0x005936d0 | `event_msg_car_state_deserialize` | - | Type 0x1e/0x1f, descriptor `0x0063e234`, slot 3 |
| 0x00593720 | `event_msg_car_glass_break_serialize` | - | Type 0x27, descriptor `0x0063e254` |
| 0x00593770 | `event_msg_car_glass_break_deserialize` | - | Type 0x27, descriptor `0x0063e254`, slot 3 |
| 0x00593aa0 | `event_msg_dword_byte_serialize` | - | Type 0x25, descriptor `0x0063e314` |
| 0x00593ad0 | `event_msg_dword_byte_deserialize` | - | Type 0x25, descriptor `0x0063e314`, slot 3; its result is discarded by the only caller |
| 0x00593b10 | `event_msg_dword_pair_serialize` | - | Generic two-dword class; descriptors `0x0063b6dc` + `0x0063e198` + `0x0063e264` |
| 0x00593b40 | `event_msg_car_damage_serialize` | - | Type 0x28, descriptor `0x0063e284` |
| 0x00593cf0 | `event_msg_car_damage_deserialize` | - | Type 0x28, descriptor `0x0063e284`, slot 3 |
| 0x00593ef0 | `event_msg_dword_pair_reversed_serialize` | - | Generic two-dword class with a `{value, guid}` payload; descriptors `0x0063e0bc` + `0x0063e2a4` |
| 0x00593f20 | `event_msg_car_collision_damage_serialize` | - | Type 0x2b, descriptor `0x0063e294` |
| 0x00593fd0 | `event_msg_car_collision_damage_deserialize` | - | Type 0x2b, descriptor `0x0063e294`, slot 3 |
| 0x00594290 | `event_msg_car_part_dropout_payload_init` | - | Default ctor of the type-0x2d payload |
| 0x005942c0 | `event_msg_car_part_dropout_serialize` | - | Type 0x2d, descriptor `0x0063e274` |
| 0x00594330 | `event_msg_car_part_dropout_deserialize` | - | Type 0x2d, descriptor `0x0063e274`, slot 3 |
| 0x005943b0 | `event_msg_slot_flag_serialize` | - | Types 0x2e/0x2f, descriptor `0x0063e1a8` |
| 0x005943e0 | `event_msg_slot_flag_deserialize` | - | Types 0x2e/0x2f, descriptor `0x0063e1a8`, slot 3 |
| 0x00594420 | `event_msg_session_setting_serialize` | - | Type 0x31, descriptor `0x0063e1b8` |
| 0x00594460 | `event_msg_session_setting_deserialize` | - | Type 0x31, descriptor `0x0063e1b8`, slot 3 |
| 0x005944b0 | `event_msg_ai_racer_add_serialize` | - | Type 0x32, descriptor `0x0063e1c8` |
| 0x005944f0 | `event_msg_ai_racer_add_deserialize` | - | Type 0x32, descriptor `0x0063e1c8`, slot 3 |
| 0x00594540 | `event_msg_ai_racer_remove_serialize` | - | Type 0x33, descriptor `0x0063e1d8` |
| 0x00594570 | `event_msg_ai_racer_remove_deserialize` | - | Type 0x33, descriptor `0x0063e1d8`, slot 3 |
| 0x005945a0 | `event_msg_race_finish_serialize` | - | Types 0x37/0x38, descriptor `0x0063e304` |
| 0x005945e0 | `event_msg_race_finish_deserialize` | - | Types 0x37/0x38, descriptor `0x0063e304`, slot 3 |
| 0x00594650 | `event_msg_dword_pair_payload_init` | - | - |
| 0x00594660 | `event_msg_dword_pair_reversed_deserialize` | - | Generic `{value, guid}` class, slot 3; descriptors `0x0063e0bc` + `0x0063e2a4` |
| 0x005946c0 | `event_msg_door_state_payload_init` | - | Default ctor of the type-0x34 payload |
| 0x005946d0 | `event_msg_door_state_serialize` | - | Type 0x34, descriptor `0x0063c21c` |
| 0x00594700 | `event_msg_door_state_deserialize` | - | Type 0x34, descriptor `0x0063c21c`, slot 3 |
| 0x005a18f0 | `event_msg_car_damage_payload_init` | - | Closes one of the four NAMING-DEFERRED payload helpers listed in `structs/net_event_msg.md` §11 |
| 0x005a1930 | `event_msg_dword_pair_payload_init_005a1930` | - | Second emission of the same source inline; the first is `event_msg_dword_pair_payload_init` (0x00594650) |
| 0x005a1970 | `event_msg_car_collision_damage_payload_init` | - | Closes one of the four NAMING-DEFERRED payload helpers listed in `structs/net_event_msg.md` §11 |

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x005937d0 | `hit_event_payload_fill` | - | 14-dword field-by-field fill of the 0x38 hit payload |
| 0x00593840 | `hit_event_payload_serialize` | - | Tag-0x1c descriptor slot 2; the 0x38 field list |
| 0x005938c0 | `hit_event_payload_deserialize` | - | Tag-0x1c descriptor slot 3; the write mirror |
| 0x00593950 | `collision_event_payload_fill` | - | field-by-field fill of the `S_msg_collision` payload |
| 0x005939c0 | `collision_event_payload_serialize` | - | type-`0x1d` descriptor slot 2, the `S_msg_collision` field list |
| 0x00593a30 | `collision_event_payload_deserialize` | - | type-`0x1d` descriptor slot 3, the write mirror |

## 4. The write / read primitives

| function | direction | note |
|---|---|---|
| `event_msg_serialize_bytes` | write | `realloc(buf, len+n)`, `rep movs`, `len += n`. **99 callsites, all in-TU** |
| `event_msg_serialize_bytes_at` | write | at an absolute offset, does NOT commit `len` — the back-patch primitive |
| `event_msg_serialize_cstring` | write | `strlen + 1`, terminator included |
| `event_msg_deserialize_bytes` | read | `src = base + cursor`; the base is **caller-supplied**, never `this->buf` |
| `event_msg_deserialize_cstring` | read | advances by `strlen + 1` |
| `event_msg_set_body_from_wire` | neither | adopts a received buffer verbatim; **27 callsites, all the host relay path** |

**The vtable is 4 slots:**

| slot | role | base implementation |
|---|---|---|
| 0 | `Serialize(payload, len)` | `event_msg_serialize_header` — append the type, then a raw blob |
| 1 | `Deserialize(dst, wire, len)` | `event_msg_deserialize_header` — read the type out of `wire`, cursor = 4, then a raw blob |
| 2 | `SerializePayload(payload)` | per-message override |
| 3 | `DeserializePayload(dst, wire)` | per-message override |

Two descriptors override nothing and run on the base pair directly: `g_event_msg_vt_base` and
`g_event_msg_vt_race_camera`.

## 5. Direction of the verbs — the evidence

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

## 6. The descriptor roster — a closed set of 39

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
| `0x0063e140` | `g_event_msg_vt_base` | many | no payload: the base pair |
| `0x0063e148` | `g_event_msg_vt_actor_guid_table` | `0xc` | GUID pairs. **Send-only** |
| `0x0063e158` | `g_event_msg_vt_scene_load` | `0xb` | 8 B `{mission name, param}` (§6b) |
| `0x0063e178` | `g_event_msg_vt_chat` | `1` | `{char text[200], u8}` |
| `0x0063e188` | `g_event_msg_vt_session_start` | `6` | no payload |
| `0x0063e198` | `g_event_msg_vt_ping` | `7` | two dwords (timestamps) |
| `0x0063e1a8` | `g_event_msg_vt_slot_flag` | `0x2e`, `0x2f` | `{u8 flag, u8 slot}` |
| `0x0063e1b8` | `g_event_msg_vt_session_setting` | `0x31` | `{u8 selector, u32 value, u8 slot}` |
| `0x0063e1c8` | `g_event_msg_vt_ai_racer_add` | `0x32` | `{char name[0x20], u32, u32}` |
| `0x0063e1d8` | `g_event_msg_vt_ai_racer_remove` | `0x33` | `{u32 index}` |
| `0x0063e1e8` | `g_event_msg_vt_timeout_watchdog` | `9` | one byte (§6b) |
| `0x0063e1f8` | `g_event_msg_vt_scene_done` | `0x10` | 16 B `{slot, mission name, param}` (§6b) |
| `0x0063e208` | `g_event_msg_vt_actor_owner` | `0x26` | 8 B `{actor guid, new owner slot}` (§6b) |
| `0x0063e224` | `g_event_msg_vt_actor_state` | `0x14`, `0x21`, `0x23` ¹ | the actor-state base |
| `0x0063e234` | `g_event_msg_vt_car_state` | `0x1e`, `0x1f` | physics state + car extras |
| `0x0063e254` | `g_event_msg_vt_car_glass_break` | `0x27` | `{guid, part code, part dword, vec3}` |
| `0x0063e264` | `g_event_msg_vt_tyre_burst` | `0x2c` | `{guid, wheel}` |
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
| `0x0063e314` | `g_event_msg_vt_unsent_type_0x25` | `0x25` | **never sent** (§6a) |
| `0x0063e324` | `g_event_msg_vt_peer_latency_table` | `8` | one dword per peer |
| `0x0063e334` | `g_event_msg_vt_race_camera` | `0x3a` | no payload: sent via slot 0 as an 8-byte blob |

¹ Received as type `0x14`, and sent as the reduced car snapshot `0x21` / `0x23` (§7); its constructor
takes the type as a parameter and serves as the base of the actor-state subclasses.

The dispatcher's case labels are the type minus one (§9), so the type-`0x10` arm is `case 0xf` and
the `g_event_msg_vt_u8` arms are `case 0x10` / `case 0x11`.

### 6a. Two descriptors worth a second look

**`g_event_msg_vt_unsent_type_0x25`.** A scan of the image for the address as a pointer finds one
reference: `0x0059f3d1`, inside `net_session_handle_event_msg`. There is no producer and no
constructor — and the receiving arm (`case 0x24`) parses the payload into a stack local and falls
through to `break` without reading it. Both halves are inert (§12).

**`g_event_msg_vt_actor_guid_table` is send-only by construction** — slot 3 of its descriptor is the
shared stub `stub_ret0_args8_00575a60`. The receive side is a separate free function,
`actor_apply_net_guid_table`, called by the SENDER right after serializing.

### 6b. The four session descriptors

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

### 6c. Payload default constructors

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
`+0x18` of `C_mission`'s settings block, the particles quality (`structs/mission.md` §1a). The two
receivers share no base class and the body occurs once in the image, so the one address is
identical-code folding of two inline accessors. The unfolded duplicates above do not contradict
it: they are ordinary functions of this translation unit, and only inline functions, which
every object file emits on its own, were folded in this build. That split is inferred, not
measured; it is the one model that fits both observations.

## 7. Message types, by feature

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
| `0x2c` | a tyre burst: a shot hit a wheel; the receiver sets the wheel's burst bit `0x80000000` and plays the burst sound | `car_post_tyre_burst_net_event_msg` |
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

## 8. The wire format

A message on the wire is `[u32 type][payload]`. Everything past the type is written by the class's
slot 2 in field order; there is no length prefix and no alignment padding, so the two sides agree
only because slot 2 and slot 3 are written as mirrors.

**Quantisation.** Position and velocity triples travel as 16-bit sign-magnitude binary fractions through
`net_encode_fixed16` / `net_decode_fixed16`: `(int << fracbits) | frac`, with the sign in bit
`0x8000` rather than two's complement.

**Ownership.** The top byte of an actor GUID is the owning peer's slot —
`actor_apply_net_guid_table` computes `g_net_session+0x230 != (guid >> 24)` to mark every actor this
peer does not own at `actor+0x20`, excluding kind `0x1b` (`C_entity`).

## 9. The dispatcher

`net_session_handle_event_msg` (`0x0059e120`, ~1900 lines) is `switch (*wire - 1)`: every case label
is the message type minus one. Each arm builds the matching class on the stack, calls its slot 3,
applies the result, and where the local peer is the host rebuilds the message with
`event_msg_set_body_from_wire` and re-posts it.

It is the single consumer of 27 of the 39 descriptors. Its jump table has `0x41` entries; those of
types `2`, `0x18`, `0x30`, `0x35` and `0x39` point at the exit.

## 10. PC vs PS2

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

## 11. Related

`structs/net_session.md` (the `g_net_session` session object and its peer records) ·
`structs/car.md` (the `C_car` breakable-part and damage bands) · `structs/door.md` · `structs/bottle.md` ·
`structs/box.md` · `structs/physical_object.md` · `structs/racing.md` · `structs/camera.md`.

## 12. Open items

1. Type `0x25`: does a second peer ever send it (§6a)? In-game question.
2. `[M]` fields whose consumer is not read: `S_msg_hit` `direction`, `normal`, `damage`, `hit_detail` and `S_msg_collision` `direction`, `normal`, `strength` (the parameters of the Hit and Collision overrides); `S_msg_car_glass_break.position` (`vehicle_deform_glass`) and `S_msg_car_collision_damage.damage_value` (`vehicle_deform`'s third argument).
3. `S_msg_peer_latency_table` (`u32` count, one dword per peer, peer record `+0x28`), the actor GUID table and the car damage table (two back-patched runs) are variable blocks with no set record: their shape is the row of §6, not a typedef.
