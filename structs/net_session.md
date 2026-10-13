# `g_net_session` — the multiplayer session object

**Created** 2026-08-16 18:02 +0200 · **Modified** 2026-10-05 15:41 +0200

| | |
|---|---|
| Symbol | `g_net_session` (`DAT_006efbac`), 273 references in the annotated decomp |
| Shape | a **712-byte heap singleton** — `operator_new(0x2c8)` in `engine_init`, destroyed by `engine_shutdown`, held in its own global |
| PC prefix | **`net_session_`** for every method of this object; `net_` is the free/subsystem root (`net_encode_fixed16`, `net_decode_fixed16`, `net_find_actor_by_guid`, `net_deactivate_actors_of_peer`, and the four free session drivers in `structs/race_session.md`) |
| PS2 | **nothing** — the PS2 build has no networking (§6) |

**It is not `C_game` and not a sub-object of one.** `C_game` is `*(g_act_mission+0x24)`, owned by
`C_mission` (`structs/game.md`). The two are easy to conflate because both have a meaningful
`+0x204`: on `C_game` that is the current taxi-passenger candidate actor
(`game_is_valid_taxi_passenger`), here it is the embedded local player record. Everything this object
owns is networking — connection `+0x108`, INet driver `+0x10c`, connection settings `+0x110`, game
settings `+0x1ac`, peers `+0x268`, send queues, lobby flags, AI racer slots `+0x294`, position grid
`+0x14`.

## 1. Field map

| offset | meaning | evidence |
|---|---|---|
| `+0x14` | position grid | - |
| `+0x20` | an embedded `C_console` (`0x3c` B), constructed and destroyed with the session but never registered, so nothing draws it | `net_session_ctor` `lea ecx,[esi+0x20]` → `console_ctor`; `net_session_dtor` mirrors it (`structs/console.md`) |
| `+0x5c` | a second embedded `C_console`, registered with flags **3** = always render + cleared every frame — the multiplayer on-screen message overlay | `net_session_ctor` → `console_ctor` + `console_register`; `net_session_dtor` → `console_unregister` + `console_dtor` |
| `+0x98` | frame delta accumulated by the ping timer | `net_session_ping_tick` |
| `+0xa5` | cleared by transport id `0xffff0014` | `net_session_receive_messages` |
| `+0xb0` | container base of the relevance-filtered peer vector (begin/end at `+0xb4`/`+0xb8`) | published by `net_session_get_relevant_peer_list`, filled by `net_session_partition_peers_by_relevance` |
| `+0xb4` | walked to post one state message per peer | `physical_object_broadcast_state_net_event_msg` |
| `+0xd4`/`+0xd8`/`+0xdc` | `vector<provider*>` — the enumerated connection providers, records `{const char *name, int driver_index, int type}` (`0xc` B) | `net_session_enumerate_providers` |
| `+0x100` | **session state** ¹ | `net_session_set_state`, `net_session_start`, `net_session_init_provider`, `net_session_is_active`, `net_multiplayer_run` |
| `+0x104` | **the state the load screen waits for**: `race_net_load_sync` sets 6, then 7, and `gui_load_progress_tick` closes the screen once `+0x100` equals it; 8 = none (§3c) | `net_session_ctor` and `net_session_leave` (8), `race_net_load_sync`, `gui_load_progress_tick` |
| `+0x108` | the **INet connection object**, the target of every session-level forward ² | `net_session_set_name`, `net_session_set_password`, `net_session_create_as_host`, `net_session_list_available` |
| `+0x10c` | the **INet driver object** from `inet.dll`'s `_CreateINetBase@4`; vtable `+0x14`/`+0x18`/`+0x1c` enumerate providers, `+0x20`/`+0x24` enumerate sessions | `net_session_init_provider`, `net_session_enumerate_providers`, `gui_mp_settings_ctor` |
| `+0x110` | the `0x9c`-byte network-settings block (§4a) | `net_session_settings_addr` returns `this+0x110`; `gui_mp_settings_ctor` copies exactly `0x27` dwords |
| `+0x1ac` | the embedded game-settings block; `+0x38` of it is `+0x1e4` | `net_session_game_settings_addr` |
| `+0x1e4` | **session game mode** — 1 Deathmatch, 2 Team Deathmatch, 3 Single Race, 4 Championship. The disconnect handler's `3..4` gate is a test for a race mode | `net_session_remove_peer`, `net_session_game_settings_ctor`, `net_session_leave` (zeroes it) |
| `+0x1fc` | the last error's textdb id — **900** = "Error in Direct Play initialization!"; `0x38c` when the session dies | `net_session_init_provider`, `net_session_create_connection`, `net_session_receive_messages` |
| `+0x200` | **this machine hosts the session.** `net_session_post_event_msg` sets a message's broadcast flag `[5]` only on the host, for a message addressed to the host itself (`[4]` equals `+0x274`); the receive arms re-post a message for the other peers only when it is set; `net_session_return_to_lobby` picks state 4 when set and 3 when clear; `race_net_load_sync` and `gui_mp_lobby_ctor` take their host or client branch on it | `net_session_post_event_msg`, `net_session_return_to_lobby`, `race_net_load_sync`, `gui_mp_lobby_ctor`, `racing_set_damage_enabled`, `net_session_remove_ai_racer`, `racing_tick` phases 2/3 |
| `+0x204` | **the local player's record, embedded** (not a pointer), so its name field is at `+0x208` | the session banner prints `+0x208`; `racing_do_gates` and `opponent_do_recover` compare the ADDRESS `g_net_session + 0x204`; the latency deserializer skips this peer |
| `+0x228` | the local player's **driver-appearance model index** into `tables\playerindex.def` | written by `gui_mp_lobby_on_child_result`; read by `gui_mp_lobby_ctor` / `_tick` through `car_database_get_player_model` |
| `+0x230` | **the local player's slot**: `+0x2c` of the record embedded at `+0x204`. The host takes slot 0; a client takes one past the highest slot in the received roster, the rule the host's join arm uses to assign it. Compared against a record's `+0x2c` on disconnect and against `+0x1b` of a wire record | `net_session_create_as_host`, the type-3 receive arm, `net_session_remove_peer` |
| `+0x252` | the local player's **ready** flag; greys the Garage and Person buttons, toggled by a type-`0x2e` receive | `gui_mp_lobby_tick`, `net_session_set_ready_flag` |
| `+0x268` / `+0x26c` | `vector<player*>` — **the human players**, the local one included | `racing_place_cars`, `racing_activate_racers`, `net_session_average_remote_latency`, `net_session_find_peer_by_id`, the disconnect handler |
| `+0x274` | **the host's peer id**: the host stores its own id, a client the id of the first roster record, which is the host's. Most producers address a message to it in `[4]`; on the host such a message is relayed to every other peer (§3b) | `net_session_create_as_host`, the type-3 receive arm, `net_session_post_event_msg` |
| `+0x27c` | ping accumulator; fires past 4000 ms | `net_session_ping_tick` |
| `+0x280` | **the timeout watchdog is armed**: `net_session_ping_tick` does nothing while it is clear. Set by `net_session_create_as_host` and by the type-`0xa` acknowledgement; cleared by the constructor, `net_session_leave` and `net_session_disable_timeout_watchdog` | `net_session_ping_tick` |
| `+0x289` | **ready-toggle request in flight** — set by `net_session_request_ready_flag` when a client posts type `0x2f`, cleared when the host's type-`0x2e` echo returns for our own peer; read twice by `gui_mp_lobby_tick` to stop a client toggling ready while a request is outstanding | `net_session_request_ready_flag`, `gui_mp_lobby_tick` |
| `+0x294` / `+0x298` | `vector<ai_slot*>` — **the AI opponents** configured for the session | `racing_add_racer_for_ai_slot`, `net_session_add_ai_racer`, `net_session_remove_ai_racer` |
| `+0x2a0` / `+0x2a4` | mid-buffer SUSPEND request and the saved parse offset — the dispatcher can stop part-way through a payload and resume next frame | `net_session_receive_messages` |
| `+0x2c0` | accumulated microseconds spent inside the transport, measured with `QueryPerformanceCounter` around every call | `net_session_receive_messages` |

¹ The state is a stage, written by `net_session_set_state` and by the session's own methods:

| value | meaning | writers |
|---|---|---|
| 0 | no session: no INet provider, or a step failed, with the error's textdb id in `+0x1fc` | `net_session_init_provider` (textdb 900), `net_session_create_connection`, `net_session_connect_selected_provider`, `net_session_create_as_host`, `net_session_join`, `net_session_list_available` (textdb `0x388`), `net_session_ping_tick`, `net_session_remove_peer`, transport id `0xffff0016`, a scene mismatch in the type-`0xb` arm (textdb `0x391`) |
| 1 | idle: a provider exists, no session | `net_session_ctor`, `net_session_leave`, `net_multiplayer_run` after a failed host or join |
| 2 | the selected connection provider is connected | `net_session_connect_selected_provider` |
| 3 | in a session's lobby as a client | `net_session_join`, `net_session_return_to_lobby` |
| 4 | in a session's lobby as the host | `net_session_create_as_host`, `net_session_return_to_lobby` |
| 5 | the game is starting | `net_session_start` (host), the type-6 arm (client) |
| 6 | every player has the same scene loaded | `race_net_load_sync` and the type-`0xb` arm (§3c) |
| 7 | the load handshake is complete and the game runs | `race_net_load_sync`, the type-`0xd` arm (host) and the type-`0xe` arm (client) (§3c) |

Every machine waits for 7 before the race runs, so 7 means "a networked game is running" on host
and client alike. `net_multiplayer_run` refuses on 0; `gui_mp_lobby_tick` closes the lobby with
code `0x3d` on 5 and `0x3f` on 1; the networked physics gates (`box_ai_tick`,
`bottle_post_collision_net_event_msg`, `opponent_do_recover` and others) test `== 7`.
² 8 vtable slots are used: `+0x14`/`+0x18`/`+0x1c` enumerate sessions, `+0x20` set session name,
`+0x24` set password, `+0x34` create session, `+0x50` local player id, `+0x54` set player name.
Layout inside `INet.dll`: §4c.

**The participant cap is 21.** `net_session_add_ai_racer` refuses once `players + AI >= 0x15` — the
same 21 as the championship results table (`championship+0x00`, stride 8) and the round's
final-positions array (`champ_circuit+0x0c`).

## 2. The three record types

### 2a. Human player record — `+0x268` vector, local one embedded at `+0x204`

| offset | meaning | evidence |
|---|---|---|
| `+0x04` | `char name[0x20]` — the name the player typed | `racing_add_racer_for_player` copies `0x20` bytes into the racer; the banner prints `g_net_session+0x208` |
| `+0x20` | driver-name table index | `racing_set_player_racer_model`'s sibling path |
| `+0x24` | chosen car model id | `racing_add_racer_for_player`, `racing_bind_local_player_racer` |
| `+0x28` | per-player latency, averaged over **remote** players only (`+0x204` skipped, divisor `count - 1`) | `net_session_average_remote_latency`, whose result is the pre-start pacing period in `racing_tick` phase 2 |
| `+0x2c` | **player id / slot byte** — the disconnect handler's match key and the sort key of `race_compare_participant_slot` | `net_session_remove_peer` |
| `+0x4c` | the player acknowledged the actor-GUID table (§3c) | set by the type-`0xd` arm, the local record's (`session+0x250`) by `race_net_load_sync` |
| `+0x4d` | the player has the scene loaded (§3c); also tested in the session listing | set by the type-`0xb` arm, the local record's (`session+0x251`) by `race_net_load_sync`; the `"%s: players: %i / max: %i"` path |

### 2b. AI opponent slot — `+0x294` vector, `0x2c` bytes

| offset | meaning | evidence |
|---|---|---|
| `+0x00` | `char name[0x20]`, filled from the driver-name table, not typed | `net_session_add_ai_racer` copies the name back out of the created racer |
| `+0x20` | **driver-name table index**, written back by `racing_add_racer_for_ai_slot` so every machine draws the same name | that write-back is why the function takes a "draw a new one" flag |
| `+0x24` | car model id, chosen by whoever added the slot | `net_session_add_ai_racer` sets it before creating the racer |
| `+0x28` | the slot's own index; the sort key of `race_compare_participant_slot` for AI entrants | `net_session_add_ai_racer` stores the current vector length |

### 2c. Peer record — `0x60` bytes (`operator_new(0x60)` in the type-3 and type-4 receive arms)

| offset | meaning | evidence |
|---|---|---|
| `+0x00` | peer id | `net_session_find_peer_by_id` compares it |
| `+0x04` | display name | the chat receiver formats `"%s : %s"` with it |
| `+0x28` | **mean half round-trip time** | `net_session_peer_update_latency` stores the mean of the `+0x34..+0x38` window |
| `+0x2c` | player slot, assigned by the host as one past the highest slot in use; the local player's is `session+0x230` | the roster message carries it |
| `+0x34` / `+0x38` | a 10-entry rolling window of RTT samples, trimmed by shifting | `net_session_peer_update_latency` |
| `+0x40` / `+0x44` | timeout countdown pair, rearmed to 8000 ms | `net_session_peer_update_latency` |
| `+0x4e` | the peer's **lobby ready flag** — written only by the type-`0x2e` / type-`0x2f` receive arms, ANDed across peers by `net_session_all_peers_ready`, rendered by `gui_widget_net_player_set_row` | `net_session_all_peers_ready`, `net_session_can_start` |

**The racer back-pointers.** A `C_opponent` carries exactly one of the first two:
`opponent+0x50` → a human player record, `opponent+0x54` → an AI slot. That is how
`racing_find_opponent_by_player` / `_by_ai_slot`, `racing_remove_racer_for_player` / `_for_ai_slot`
and `race_compare_participant_slot` tell them apart, and how `racing_place_cars` builds a networked
grid in session order rather than array order.

### 2d. Who adds and removes records

A census of every write to `+0x264..+0x26c` and `+0x290..+0x298` in `Game.exe` closes the set:

| vector | adds | removes |
|---|---|---|
| human players `+0x268` | `net_session_create_as_host` (the local record), the type-`0xffff0007` arm (host), the type-3 roster arm and the type-4 arm (client) | `net_session_remove_peer` (from the type-`0xffff0009`, type-5 and type-`0xb` arms, `net_session_ping_tick` and the lobby's kick), `net_session_leave` |
| AI opponents `+0x294` | `net_session_add_ai_racer`, the type-`0x32` arm | `net_session_remove_ai_racer`, `net_session_clear_ai_racers` |

## 3. The message layer

### 3a. Receive

`net_session_receive_messages` (`0x0059da60`) is the partner of `net_session_send_queued_messages`;
`net_session_tick` calls them in that order. It drains the INet transport at `+0x108` and dispatches
on the system id:

| id | meaning |
|---|---|
| `0xffff0007` | a peer joined — allocate the `0x60`-byte record, assign the next free slot, post the roster and player-add messages, push the join banner (textdb `0x38e` + `0x390`), and in a race send the racing setup |
| `0xffff0009` | a peer left — `net_session_remove_peer` |
| `0xffff0011` | a payload — handed to `net_session_handle_event_msg` |
| `0xffff0014` | clears `+0xa5` |
| `0xffff0016` | the session died — `+0x100 = 0`, textdb `0x38c` into `+0x1fc` |

`net_session_handle_event_msg` (`0x0059e120`, ~1900 lines) is the payload dispatcher: it switches on
`*wire - 1` (the table is biased by one because type 0 has no arm), builds the matching message class
on the stack, calls its slot-3 deserializer, applies the result, and — where the local peer is the
host — rebuilds the message with `event_msg_set_body_from_wire` and re-posts it. It is the single
consumer of 27 of the 39 message descriptors (`structs/net_event_msg.md`).

### 3b. Send

Every outbound message is an `operator_new(0x18)` record with the type in `[2]` and the addressee's
peer id in `[4]`, for most producers `g_net_session+0x274`, the host. It is posted through
`net_session_post_event_msg`; on the host, a message addressed to the host is relayed to every other
peer. Type `5`, the type-`7` echo and the host's type-`0xb` answer are sent at once by
`net_session_send_event_msg_direct`. Every type and its producer is in `structs/net_event_msg.md` §4;
the lobby and race-control types:

| type | vtable | posted by | meaning |
|---|---|---|---|
| `0x05` | `g_event_msg_vt_base` | the disconnect handler | a player left |
| `0x0f` | `g_event_msg_vt_base` | `racing_tick` phase 2 | pre-start sync tick |
| `0x2e` / `0x2f` | `g_event_msg_vt_slot_flag` | `net_session_set_ready_flag` / `net_session_request_ready_flag` | lobby ready echo / request |
| `0x32` | `g_event_msg_vt_ai_racer_add` | `net_session_add_ai_racer` | an AI opponent was added |
| `0x33` | `g_event_msg_vt_ai_racer_remove` | `net_session_remove_ai_racer` | an AI opponent was removed |
| `0x38` | `g_event_msg_vt_race_finish` | `racing_do_gates` | a racer finished (slot, time, place) |
| `0x3b` / `0x3c` | `g_event_msg_vt_base` | `racing_set_damage_enabled` | damage on / off |
| `0x3d` | `g_event_msg_vt_base` | `opponent_do_recover` | a car respawned |

### 3c. The load handshake

`race_net_load_sync` runs on every machine once a networked race scene is loaded, and walks the
session through states 6 and 7:

1. **Scene.** A client posts type `0xb` with its scene identity (`C_mission+0x00..+0x07`) and
   waits for 6. The host sets its own record's `+0x4d`; each arriving `0xb` sets that player's.
   Once every `+0x4d` is set the host enters 6 and posts its own `0xb`, and a client enters 6
   when the host's identity matches its own, or 0 with textdb `0x391` when it does not.
2. **Actor GUIDs.** The host builds the remap table (`net_session_build_actor_guid_table`),
   applies it, posts it as type `0xc` and sets its own `+0x4c`. Each client applies it
   (`actor_apply_net_guid_table`) and answers with type `0xd`, which sets that player's `+0x4c`.
   Once every `+0x4c` is set the host posts type `0xe` and enters 7; a client enters 7 when the
   `0xe` arrives. A host alone in the session enters 7 at once.
3. **Wait.** Both sides hold the load screen open until `+0x100` reaches `+0x104` = 7.

## 4. The DirectPlay / `inet.dll` layer

The transport half of the session, reached from the multiplayer front end
(`structs/multiplayer.md`).

| PC VA | PC name | role |
|---|---|---|
| `0x00597190` | `net_session_init_provider` | create the INet driver — the gate that decides whether multiplayer is reachable at all |
| `0x005974c0` | `net_session_enumerate_providers` | refill `+0xd4`/`+0xd8`; keep types 1, 2, 4 |
| `0x00597710` | `net_session_settings_addr` | `return this + 0x110` |
| `0x005977e0` | `net_session_settings_apply_saved` | apply `g_net_settings_saved` |
| `0x00597800` | `net_session_settings_apply` | copy a `0x9c`-byte block into the session |
| `0x005978b0` | `net_session_selected_provider_is_valid` | gates every Multiplayer-menu item; matches `+0x130` of the settings block against a provider record's `+0x08` |
| `0x005978f0` | `net_session_find_provider_index_by_type` | type → the driver's provider index, or `0xffffffff` |

### 4a. The settings block at `+0x110`

`0x9c` bytes, copied wholesale by `net_session_settings_apply`. The module-global copy is
`g_net_settings_saved` (`0x006724d0`); `net_session_settings_apply_saved` pushes it into
`net_session_settings_apply` with the receiver forwarded from `net_session_init_provider`.

| offset | field | edited by |
|---|---|---|
| `+0x110` | `char player_name[0x20]` — when empty, `net_session_init_provider` runs `net_session_settings_init_defaults`, which copies in the active profile name (`g_loadsave_active_profile_name`) | settings component 10, textdb 623 "Player Name:" |
| `+0x130` | connection-provider TYPE (1 / 2 / 4) | combo `0xb`, textdb 624 |
| `+0x134` | provider sub-selection | combo `0xc` |
| `+0x138` | `char ip_address[0x20]` | component `0x70`, textdb 630 |
| `+0x158` | `char port[0x20]` | component `0x71`, textdb 631 |
| `+0x178` | `char serial_port[0x20]` | component `0xd4` |
| `+0x198`..`+0x1a8` | five dwords — COM port, baud rate, stop bits, parity, flow control | combos `0x32c`..`0x330`, textdb 634–638 |

`net_session_settings_save` copies the block the other way, field by field, into the saved copy,
whose fields carry their own names: `g_net_settings_saved` (the player name),
`g_net_settings_saved_provider_type`, `_provider_index`, `_ip_address`, `_port`, `_serial_port`,
and the five serial dwords `g_net_settings_saved_com_port`, `_baud_rate`, `_stop_bits`, `_parity`,
`_flow_control` (`0x00672558`..`0x00672568`).

### 4b. The two 11-byte connection forwarders

`net_session_set_name` (`0x005988e0`) and `net_session_set_password` (`0x00598a30`) are the same
eleven bytes, differing only in the vtable slot:

```
mov ecx, [this + 0x108]      ; the INet connection object
mov eax, [ecx]               ; its vtable
jmp  dword ptr [eax + 0x20]  ; set_name       — slot 8
jmp  dword ptr [eax + 0x24]  ; set_password   — slot 9
```

Ghidra renders both defs `__fastcall(int param_1)` and drops the ECX `this`, so the rendered
`param_1` is the pushed STRING, not the receiver. The callsite proves it — at `0x005961b0` inside
`net_host_session_start`: `mov ecx,[g_net_session]; push offset "GALAXIAN BEST ARCADE EVER!"`.

### 4c. What the connection does with them

Position alone cannot separate "password" from "session description"; the shipped `INet.dll` — which
creates the `+0x108` object — settles it:

| INet connection field | written by | ends up in |
|---|---|---|
| `conn+0x2c` (ANSI) / `conn+0x38` (wide) | vtable slot 8 (`+0x20`) | `DPN_APPLICATION_DESC.pwszSessionName` = `desc+0x30` |
| `conn+0x30` (ANSI) / `conn+0x3c` (wide) | vtable slot 9 (`+0x24`) | `DPN_APPLICATION_DESC.pwszPassword` = `desc+0x34` |
| `conn+0x40` | the create-session slot (`+0x34`) | `desc+0x28` = `dwMaxPlayers` |
| `conn+0x44` | the create-session slot (`+0x34`) | the "a password is required" flag |

Both slots tail-call one shared applier at `INet.dll!0x100028c0`, which zero-fills a `0x48`-byte
`DPN_APPLICATION_DESC` (`dwSize = 0x48`), fills the fields above and hands it to DirectPlay8's
`SetApplicationDesc`. `conn+0x3c` reaches `desc+0x34` only when `conn+0x44` is set.

**The password is dormant in retail.** `conn+0x44` is written in exactly one place — the
create-session slot — and only on the branch that also ORs `0x80` (`DPNSESSION_REQUIREPASSWORD`) into
`desc+0x04`. `net_session_create_as_host` calls that slot as `(&g_net_directplay_app_guid, 0)`, so the
branch is never taken: the joke password `"GALAXIAN BEST ARCADE EVER!"` is stored on the connection
and never advertised. Nothing in the PC build ever asks for a session password.

## 5. Related

`structs/game.md` (the singleton this is not) · `structs/multiplayer.md` (the front end) ·
`structs/race_session.md` (the free session drivers) · `structs/net_event_msg.md` (the message
classes) · `structs/racing.md` · `structs/console.md` · `basic/artifacts/dll.md` (the five Illusion
DLLs, `inet.dll` among them).

## 6. PC vs PS2

The PS2 build ships no networking and no `inet.dll`, so **every field and every function in this
document is PC-only**. Two consequences invert the usual expectation:

* **Two racing methods are LARGER on PC**, and the excess is exactly the broadcast.
  `SetDamage__8C_racingFb` is an 8-byte inline setter on PS2 and 256 bytes on PC;
  `DoRecover__10C_opponentFR8S_vectorR8S_vector` is 348 bytes on PS2 and 464 on PC. Everywhere else
  in this project a PC function smaller than its twin is the norm.
* **`racing_tick` has two race phases PS2 does not have.** Phases 2 and 3 (`+0x2a4`) are the
  networked pre-start handshake, absent from PS2's `Tick` — one of the observations behind
  `structs/racing.md`'s reading that the PC is the earlier revision: the divergence runs both ways.

The absence of `inet.dll` on PS2 is the fifth documented reason a method exists on one build only —
here in the reverse direction.
