# `g_net_session` — the multiplayer session object

**Created** 2026-08-16 18:02 +0200 · **Modified** 2026-11-17 10:40 +0100

A **712-byte heap singleton**, `operator_new(0x2c8)` in `engine_init`, destroyed by `engine_shutdown`, held in the global `g_net_session` (273 references in
the annotated decomp). **It is not `C_game` and not a sub-object of one.** `C_game` is `*(g_act_mission+0x24)`, owned by `C_mission` (`structs/game.md`). The two are easy
to conflate because both have a meaningful `+0x204`: on `C_game` that is the current taxi-passenger candidate actor (`game_is_valid_taxi_passenger`), here it is the embedded local
player record. Everything this object owns is networking.

| | |
|---|---|
| PC prefix | `net_session_` for every method of this object; `net_` for the free functions of the unit |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x2c8` |
| PC constructor | `net_session_ctor` |
| PC translation unit | `[0x00596c50, 0x0059ab90)` and `[0x0059da30, 0x005a1b00)`, guard to guard |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

### 1a. The session

```c
typedef struct S_net_session {                    // 0x2c8 bytes, one heap instance, g_net_session
    /* +0x000 */ uint8_t  send_queue_alloc;       // [H] MSVC's empty allocator member of the send queue
    /* +0x004 */ S_net_message **send_queue_begin; // [H] messages awaiting the transport, stride 4
    /* +0x008 */ S_net_message **send_queue_end;  // [H] count = (end - begin) >> 2
    /* +0x00c */ S_net_message **send_queue_cap;  // [H] one past the allocation
    /* +0x010 */ uint8_t  grid_alloc;             // [H] the empty allocator of the position grid
    /* +0x014 */ float  (*grid_begin)[3];         // [H] the position grid's points, stride 0xc
    /* +0x018 */ float  (*grid_end)[3];           // [H] count = (end - begin) / 0xc
    /* +0x01c */ float  (*grid_cap)[3];           // [H] one past the allocation
    /* +0x020 */ C_console hidden_console;        // [H] 0x3c bytes; constructed and destroyed, never registered, so nothing draws it
    /* +0x05c */ C_console message_console;       // [H] 0x3c bytes; registered with flags 3, the on-screen message overlay
    /* +0x098 */ uint32_t frame_ms;               // [H] this frame's delta, stored by net_session_tick
    /* +0x09c */ uint32_t peer_buffer_limit;      // [H] 1000; a peer's send buffer past this goes out as its own message
    /* +0x0a0 */ uint32_t transport_bytes_limit;  // [H] 0x5dc = 1500; sending stops when the transport's backlog reaches it
    /* +0x0a4 */ uint8_t  transport_count_limit;  // [H] 0xf; sending stops when the transport's message count reaches it
    /* +0x0a5 */ uint8_t  send_paused;            // [H] stops sending; every writer stores 0 (§2)
    /* +0x0a8 */ uint32_t pause_elapsed_ms;       // [H] net_session_tick's accumulator for send_paused
    /* +0x0ac */ uint32_t pause_timeout_ms;       // [H] 0 = no timer; past it the tick clears send_paused
    /* +0x0b0 */ uint8_t  relevant_peers_alloc;   // [H] the empty allocator; this byte's address is the container the accessor publishes
    /* +0x0b4 */ S_net_peer **relevant_peers_begin; // [H] the peers a state message goes to, stride 4
    /* +0x0b8 */ S_net_peer **relevant_peers_end; // [H] count = (end - begin) >> 2
    /* +0x0bc */ S_net_peer **relevant_peers_cap; // [H] one past the allocation
    /* +0x0c0 */ uint8_t  other_peers_alloc;      // [H] the empty allocator of the other partition
    /* +0x0c4 */ S_net_peer **other_peers_begin;  // [H] the peers net_session_partition_peers_by_relevance left out, stride 4
    /* +0x0c8 */ S_net_peer **other_peers_end;    // [H] count = (end - begin) >> 2
    /* +0x0cc */ S_net_peer **other_peers_cap;    // [H] one past the allocation
    /* +0x0d0 */ uint8_t  providers_alloc;        // [H] the empty allocator of the provider list
    /* +0x0d4 */ S_net_provider **providers_begin; // [H] the enumerated connection providers, stride 4
    /* +0x0d8 */ S_net_provider **providers_end;  // [H] count = (end - begin) >> 2
    /* +0x0dc */ S_net_provider **providers_cap;  // [H] one past the allocation
    /* +0x0e0 */ uint8_t  session_list_alloc;     // [H] the empty allocator of the session list
    /* +0x0e4 */ char   **session_list_begin;     // [H] the formatted "%s: players: %i / max: %i" lines, stride 4
    /* +0x0e8 */ char   **session_list_end;       // [H] count = (end - begin) >> 2
    /* +0x0ec */ char   **session_list_cap;       // [H] one past the allocation
    /* +0x0f0 */ uint8_t  message_log_alloc;      // [H] the empty allocator of the message log
    /* +0x0f4 */ char   **message_log_begin;      // [H] the last messages as heap strings, stride 4, trimmed to 6
    /* +0x0f8 */ char   **message_log_end;        // [H] count = (end - begin) >> 2
    /* +0x0fc */ char   **message_log_cap;        // [H] one past the allocation
    /* +0x100 */ uint32_t state;                  // [H] the session stage, section 2a
    /* +0x104 */ uint32_t load_wait_state;        // [H] the state the load screen waits for; 8 = none (section 5c)
    /* +0x108 */ INet_connection *connection;     // [H] the INet connection, the target of every session-level forward
    /* +0x10c */ INet_driver *driver;             // [H] the INet driver from inet.dll's _CreateINetBase@4
    /* +0x110 */ S_net_settings settings;         // [H] the connection settings, 0x9c bytes, section 1b
    /* +0x1ac */ S_net_game_settings game_settings; // [H] the game settings, 0x50 bytes, section 1c
    /* +0x1fc */ uint32_t last_error_id;          // [H] the last error's textdb id; 900 = "Error in Direct Play initialization!"
    /* +0x200 */ uint8_t  is_host;                // [H] this machine hosts the session
    /* +0x204 */ S_net_peer local_player;         // [H] the local player's record, embedded, 0x60 bytes, section 1d
    /* +0x264 */ uint8_t  players_alloc;          // [H] the empty allocator of the player vector
    /* +0x268 */ S_net_peer **players_begin;      // [H] the human players, the local one included, stride 4
    /* +0x26c */ S_net_peer **players_end;        // [H] count = (end - begin) >> 2
    /* +0x270 */ S_net_peer **players_cap;        // [H] one past the allocation
    /* +0x274 */ uint32_t host_peer_id;           // [H] the host's peer id: its own on the host, the first roster record's on a client
    /* +0x278 */ uint32_t last_transport_error;   // [M] the connection's +0xc result after a failed call; no reader
    /* +0x27c */ uint32_t ping_elapsed_ms;        // [H] the ping accumulator; fires past 4000 ms
    /* +0x280 */ uint8_t  watchdog_armed;         // [H] net_session_ping_tick does nothing while it is clear
    /* +0x284 */ uint32_t wait_timer_start;       // [H] timeGetTime() at the reset; the wait expires after 25000 ms
    /* +0x288 */ uint8_t  roster_dirty;           // [H] set by a transport arm, read and cleared by net_session_take_roster_dirty_flag
    /* +0x289 */ uint8_t  ready_request_pending;  // [H] a client's ready toggle is in flight
    /* +0x28c */ uint32_t last_send_handle;       // [H] the transport's handle of the last asynchronous send
    /* +0x290 */ uint8_t  ai_slots_alloc;         // [H] the empty allocator of the AI slot vector
    /* +0x294 */ S_net_ai_slot **ai_slots_begin;  // [H] the AI opponents configured for the session, stride 4
    /* +0x298 */ S_net_ai_slot **ai_slots_end;    // [H] count = (end - begin) >> 2
    /* +0x29c */ S_net_ai_slot **ai_slots_cap;    // [H] one past the allocation
    /* +0x2a0 */ uint8_t  suspend_parse;          // [H] the dispatcher stops part-way through a payload
    /* +0x2a4 */ uint32_t resume_offset;          // [H] where the next frame resumes that payload
    /* +0x2a8 */ uint32_t unread_2a8;             // [U] zeroed by the constructor; never read
    /* +0x2ac */ uint32_t events_handled;         // [H] net_session_handle_event_msg counts one per call
    /* +0x2b0 */ uint32_t unread_2b0;             // [U] zeroed by the constructor; never read
    /* +0x2b4 */ uint32_t unread_2b4;             // [U] zeroed by the constructor; never read
    /* +0x2b8 */ uint32_t payloads_received;      // [H] net_session_receive_messages counts one per payload
    /* +0x2bc */ uint32_t batched_flush_count;    // [H] counts the flushes of two or more messages; never read
    /* +0x2c0 */ uint32_t receive_transport_ticks; // [H] QueryPerformanceCounter ticks spent inside the receive transport
    /* +0x2c4 */ uint32_t send_transport_ticks;   // [H] the same around every send call
} S_net_session;
```

**The participant cap is 21.** `net_session_add_ai_racer` refuses once `players + AI >= 0x15`, the same 21 as the championship results table (`championship+0x00`, stride 8) and the
round's final-positions array (`champ_circuit+0x0c`).

**The peer lists.** All six `begin` / `end` / `cap` triples are MSVC vectors, 16 bytes each with the allocator byte, which is why their allocator bytes lie `0x10` apart.

### 1b. `S_net_settings` — the settings block

`0x9c` bytes, copied wholesale by `net_session_settings_apply`. The module-global copy is `g_net_settings_saved` (`0x006724d0`), which is `G_LoadSaveClass+0x520`.

```c
typedef struct S_net_settings {                   // 0x9c bytes; at session+0x110
    /* +0x00 */ char     player_name[0x20];       // [H] when empty, net_session_init_provider copies in the active profile name
    /* +0x20 */ uint32_t provider_type;           // [H] the connection-provider TYPE, 1 / 2 / 4
    /* +0x24 */ uint32_t provider_index;          // [H] the provider sub-selection
    /* +0x28 */ char     ip_address[0x20];        // [H] the address field
    /* +0x48 */ char     port[0x20];              // [H] the port field
    /* +0x68 */ char     serial_port[0x20];       // [H] the serial port field
    /* +0x88 */ uint32_t com_port;                // [H] the COM port combo
    /* +0x8c */ uint32_t baud_rate;               // [H] the baud rate combo
    /* +0x90 */ uint32_t stop_bits;               // [H] the stop bits combo
    /* +0x94 */ uint32_t parity;                  // [H] the parity combo
    /* +0x98 */ uint32_t flow_control;            // [H] the flow control combo
} S_net_settings;
```

The four strings are `0x20` bytes each because the settings screen's components edit that many and `net_session_settings_save` copies each as its own field.

### 1c. `S_net_game_settings`

```c
typedef struct S_net_game_settings {              // 0x50 bytes; at session+0x1ac
    /* +0x00 */ uint32_t unread_00;               // [U] the constructor writes 0; never read
    /* +0x04 */ uint32_t unused_04;               // [U] no access located
    /* +0x08 */ uint32_t unread_08;               // [U] the constructor writes 4000; never read
    /* +0x0c */ uint32_t unread_0c;               // [U] the constructor writes 0x50 = 80; never read
    /* +0x10 */ uint32_t bottle_send_period_ms;   // [H] 0xca = 202; bottle_ai_tick broadcasts when ms_since_broadcast exceeds it
    /* +0x14 */ uint32_t car_send_period_ms;      // [H] 0x96 = 150; car_update broadcasts the car's state past it
    /* +0x18 */ uint32_t object_send_period_ms;   // [H] 200; physical_object_update and two more read it
    /* +0x1c */ uint32_t box_send_period_ms;      // [H] 200; box_ai_tick posts the box's state past it
    /* +0x20 */ uint32_t unused_20[6];            // [U] no access located
    /* +0x38 */ uint32_t game_mode;               // [H] session+0x1e4: 1 Deathmatch, 2 Team Deathmatch, 3 Single Race, 4 Championship
    /* +0x3c */ uint8_t  mode_list_alloc;         // [H] the empty allocator of the vector below
    /* +0x40 */ uint32_t *mode_list_begin;        // [H] the constructor pushes 0, 1, 2, 3; only the destructor reads it
    /* +0x44 */ uint32_t *mode_list_end;          // [H] count = (end - begin) >> 2
    /* +0x48 */ uint32_t *mode_list_cap;          // [H] one past the allocation
    /* +0x4c */ uint32_t menu_mode_mask;          // [H] 0x18; bits 1..4 enable the Multiplayer menu's items 0x65..0x68
} S_net_game_settings;
```

`unused_20` is six dwords because it fills the gap between `+0x20` and `game_mode`. `0x18` sets only bits 3 and 4, so `gui_mp_menu_refresh_mode_items` leaves items `0x67` and `0x68` on.

### 1d. `S_net_peer` — a human player's record

The same `0x60`-byte record serves a peer and the local player: `operator_new(0x60)` in the type-3 and type-4 receive arms, and embedded at `session+0x204`, which is why `session+0x228` is the record's
`+0x24` and `session+0x230` its `+0x2c`.

```c
typedef struct S_net_peer {                       // 0x60 bytes
    /* +0x00 */ uint32_t peer_id;                 // [H] net_session_find_peer_by_id compares it
    /* +0x04 */ char     name[0x20];              // [H] the name the player typed
    /* +0x24 */ uint32_t driver_model;            // [H] index into tables\playerindex.def; 0x3a by default
    /* +0x28 */ uint32_t mean_half_rtt;           // [H] the mean of the sample window
    /* +0x2c */ uint8_t  slot;                    // [H] the player's slot byte, the disconnect match key
    /* +0x30 */ uint8_t  rtt_samples_alloc;       // [H] the empty allocator of the sample vector
    /* +0x34 */ uint32_t *rtt_samples_begin;      // [H] the rolling RTT window, trimmed to 10 entries
    /* +0x38 */ uint32_t *rtt_samples_end;        // [H] count = (end - begin) >> 2
    /* +0x3c */ uint32_t *rtt_samples_cap;        // [H] one past the allocation
    /* +0x40 */ uint32_t timeout_elapsed_ms;      // [H] advanced by net_session_ping_tick
    /* +0x44 */ uint32_t timeout_ms;              // [H] 8000 after a sample, 2000 on a new ping; 0 = disarmed
    /* +0x48 */ float    load_progress;           // [H] the player's loading progress, shown by the load screen
    /* +0x4c */ uint8_t  guid_table_acked;        // [H] the player acknowledged the actor-GUID table (section 5c)
    /* +0x4d */ uint8_t  scene_loaded;            // [H] the player has the scene loaded (section 5c)
    /* +0x4e */ uint8_t  lobby_ready;             // [H] the lobby ready flag; for the local record this is session+0x252
    /* +0x50 */ uint8_t *send_buffer;             // [H] outgoing bytes, malloc'd; its first dword is a count
    /* +0x54 */ uint32_t send_buffer_cap;         // [H] 4 initially; grown by net_session_peer_append_message_bytes
    /* +0x58 */ uint32_t send_buffer_used;        // [H] starts at 4; past peer_buffer_limit the buffer goes out
    /* +0x5c */ uint32_t send_buffer_messages;    // [H] the messages appended since the last flush
} S_net_peer;
```

**The racer back-pointers.** A `C_opponent` carries exactly one of two: `opponent+0x50` a human player record, `opponent+0x54` an AI slot. That is how `racing_find_opponent_by_player` / `_by_ai_slot`,
`racing_remove_racer_for_player` / `_for_ai_slot` and `race_compare_participant_slot` tell them apart, and how `racing_place_cars` builds a networked grid in session order rather than array order.

### 1e. `S_net_ai_slot` — an AI opponent

```c
typedef struct S_net_ai_slot {                    // 0x2c bytes
    /* +0x00 */ char     name[0x20];              // [H] filled from the driver-name table, not typed
    /* +0x20 */ uint32_t driver_name_index;       // [H] written back by racing_add_racer_for_ai_slot so every machine draws the same name
    /* +0x24 */ uint32_t car_model;               // [H] chosen by whoever added the slot
    /* +0x28 */ uint8_t  slot_index;              // [H] the current vector length when added; the sort key for AI entrants
} S_net_ai_slot;
```

### 1f. `S_net_provider` — one enumerated connection provider

```c
typedef struct S_net_provider {                   // 0xc bytes
    /* +0x00 */ const char *name;                 // [H] the driver's provider name
    /* +0x04 */ uint32_t driver_index;            // [H] the driver's index of the provider
    /* +0x08 */ uint32_t type;                    // [H] net_session_enumerate_providers keeps types 1, 2, 4
} S_net_provider;
```

## 2. Field evidence

Cells come from a census of the class's functions over the receiver, read against the bodies, with the external users the old field map proved added by hand. The event-message arms of
`net_session_handle_event_msg` count as that function. `gui_`, `racing_` and other outside users beyond those named here were not swept; Open item 1.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | send_queue_alloc | H | `net_session_ctor` | - | an empty class; the `cmpsb byte ptr [esi]` decoding at `0x0059e0f0`, after the return of `net_session_receive_messages`, is its jump table, not a read |
| +0x4 | send_queue_begin | H | `net_session_ctor` | `net_session_send_queued_messages` | - |
| +0x8 | send_queue_end | H | `net_session_ctor`, `net_session_send_queued_messages` | `net_session_post_event_msg`, `net_session_send_queued_messages` | - |
| +0xc | send_queue_cap | H | `net_session_ctor` | - | - |
| +0x10 | grid_alloc | H | `net_session_ctor` | `net_session_build_position_grid`, `net_session_dtor` | an empty class |
| +0x14 | grid_begin | H | `net_session_ctor` | `net_session_decode_position`, `net_session_encode_position` | - |
| +0x18 | grid_end | H | `net_session_ctor` | `net_session_build_position_grid`, `net_session_decode_position`, `net_session_encode_position` | - |
| +0x1c | grid_cap | H | `net_session_ctor` | - | - |
| +0x20 | hidden_console | H | `net_session_ctor` (by address, `console_ctor`), `net_session_dtor` (by address, `console_dtor`) | `net_session_ctor`, `net_session_dtor` | `net_session_ctor` `lea ecx,[esi+0x20]` to `console_ctor`; `net_session_dtor` mirrors it (`structs/console.md`) |
| +0x5c | message_console | H | `net_session_ctor` | `net_session_ctor`, `net_session_dtor` | the `console_*` fields inside it are `structs/console.md`'s; `net_session_ctor` then `console_register` |
| +0x98 | frame_ms | H | `net_session_tick`, `race_game_loop_net` | `bottle_ai_tick`, `net_session_ping_tick` | `bottle_ai_tick` adds it to its own timer |
| +0x9c | peer_buffer_limit | H | `net_session_ctor` | `net_session_post_event_msg` | - |
| +0xa0 | transport_bytes_limit | H | `net_session_ctor` | `net_session_peer_flush_send_buffer`, `net_session_send_queued_messages` | `net_session_peer_flush_send_buffer` too |
| +0xa4 | transport_count_limit | H | `net_session_ctor` | `net_session_peer_flush_send_buffer`, `net_session_send_queued_messages` | `net_session_peer_flush_send_buffer` too |
| +0xa5 | send_paused | H | `net_session_ctor`, `net_session_receive_messages`, `net_session_tick`, `race_game_loop_net` | `net_session_peer_flush_send_buffer`, `net_session_send_queued_messages` | every writer stores 0: the constructor, the send-complete arm (`0xffff0014`) when the completed handle equals `+0x28c`, and the pause timer of `net_session_tick` and `race_game_loop_net`; a scan of `.text` finds no non-zero writer |
| +0xa8 | pause_elapsed_ms | H | `net_session_ctor`, `net_session_tick`, `race_game_loop_net` | `net_session_tick`, `race_game_loop_net` | `race_game_loop_net` runs the tick's pause timer inline |
| +0xac | pause_timeout_ms | H | `net_session_ctor`, `net_session_tick`, `race_game_loop_net` | `net_session_tick`, `race_game_loop_net` | every writer stores 0, so the pause timer never runs |
| +0xb0 | relevant_peers_alloc | H | `net_session_ctor` | `net_session_dtor`, `net_session_get_relevant_peer_list`, `net_session_partition_peers_by_relevance` | `net_session_get_relevant_peer_list` returns this address |
| +0xb4 | relevant_peers_begin | H | `net_session_ctor` | `net_session_partition_peers_by_relevance`, `physical_object_broadcast_state_net_event_msg` | `physical_object_broadcast_state_net_event_msg` walks it |
| +0xb8 | relevant_peers_end | H | `net_session_ctor` | `net_session_partition_peers_by_relevance`, `physical_object_broadcast_state_net_event_msg` | - |
| +0xbc | relevant_peers_cap | H | `net_session_ctor` | - | - |
| +0xc0 | other_peers_alloc | H | `net_session_ctor` | `net_session_dtor`, `net_session_partition_peers_by_relevance` | an empty class |
| +0xc4 | other_peers_begin | H | `net_session_ctor` | `net_session_dtor`, `net_session_partition_peers_by_relevance` | - |
| +0xc8 | other_peers_end | H | `net_session_ctor` | `net_session_dtor`, `net_session_partition_peers_by_relevance` | - |
| +0xcc | other_peers_cap | H | `net_session_ctor` | - | - |
| +0xd0 | providers_alloc | H | `net_session_ctor` | `net_session_dtor`, `net_session_enumerate_providers` | an empty class |
| +0xd4 | providers_begin | H | `net_session_ctor` | `net_session_connect_selected_provider`, `net_session_dtor`, `net_session_enumerate_providers`, `net_session_find_provider_index_by_type`, `net_session_selected_provider_is_valid` | - |
| +0xd8 | providers_end | H | `net_session_ctor` | `net_session_connect_selected_provider`, `net_session_dtor`, `net_session_enumerate_providers`, `net_session_find_provider_index_by_type`, `net_session_selected_provider_is_valid` | - |
| +0xdc | providers_cap | H | `net_session_ctor` | - | - |
| +0xe0 | session_list_alloc | H | `net_session_ctor` | `net_session_dtor`, `net_session_list_available` | an empty class |
| +0xe4 | session_list_begin | H | `net_session_ctor` | `net_session_dtor`, `net_session_list_available` | - |
| +0xe8 | session_list_end | H | `net_session_ctor` | `net_session_dtor`, `net_session_list_available` | - |
| +0xec | session_list_cap | H | `net_session_ctor` | - | - |
| +0xf0 | message_log_alloc | H | `net_session_ctor` | `net_session_dtor`, `net_session_handle_event_msg`, `net_session_send_chat_message`, `net_session_trim_message_log` | an empty class |
| +0xf4 | message_log_begin | H | `net_session_ctor` | `net_session_clear_message_log`, `net_session_dtor`, `net_session_trim_message_log` | - |
| +0xf8 | message_log_end | H | `net_session_clear_message_log`, `net_session_ctor` | `net_session_clear_message_log`, `net_session_dtor`, `net_session_handle_event_msg`, `net_session_send_chat_message` | - |
| +0xfc | message_log_cap | H | `net_session_ctor` | `net_session_push_system_message` | - |
| +0x100 | state | H | `net_multiplayer_run`, `net_session_connect_selected_provider`, `net_session_create_as_host`, `net_session_create_connection`, `net_session_ctor`, `net_session_handle_event_msg`, `net_session_init_provider`, `net_session_join`, `net_session_leave`, `net_session_list_available`, `net_session_ping_tick`, `net_session_receive_messages`, `net_session_remove_peer`, `net_session_return_to_lobby`, `net_session_set_state`, `net_session_start`, `race_net_load_sync` | `bottle_post_collision_net_event_msg`, `box_ai_tick`, `gui_load_progress_tick`, `gui_mp_lobby_tick`, `net_multiplayer_run`, `net_session_handle_event_msg`, `net_session_is_active`, `net_session_push_system_message`, `net_session_receive_messages`, `net_session_remove_peer`, `net_session_send_chat_message`, `opponent_do_recover`, `race_net_load_sync` | §2a; every `== 7` gate reads it |
| +0x104 | load_wait_state | H | `net_session_ctor`, `net_session_leave`, `race_net_load_sync` | `gui_load_progress_tick` | §5c |
| +0x108 | connection | H | `net_session_ctor` | `net_session_create_as_host`, `net_session_create_connection`, `net_session_join`, `net_session_leave`, `net_session_list_available`, `net_session_receive_messages`, `net_session_return_to_lobby`, `net_session_send_event_msg_direct`, `net_session_send_queued_messages`, `net_session_set_name`, `net_session_set_password`, `net_session_start` | the vtable slots are in §6c |
| +0x10c | driver | H | `net_session_ctor` | `gui_mp_settings_ctor`, `net_session_connect_selected_provider`, `net_session_create_connection`, `net_session_enumerate_providers`, `net_session_init_provider`, `net_session_settings_init_defaults` | vtable `+0x14`/`+0x18`/`+0x1c` enumerate providers, `+0x20`/`+0x24` enumerate sessions |
| +0x110 | settings | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `gui_mp_settings_ctor`, `net_session_connect_selected_provider`, `net_session_create_as_host`, `net_session_create_connection`, `net_session_init_provider`, `net_session_join`, `net_session_selected_provider_is_valid`, `net_session_settings_addr`, `net_session_settings_apply`, `net_session_settings_init_defaults`, `net_session_settings_save` | §2b |
| +0x1ac | game_settings | H | `net_session_leave` | `net_session_ctor`, `net_session_dtor`, `net_session_game_settings_addr`, `net_session_local_player_is_configured`, `net_session_receive_messages`, `net_session_remove_peer` | §2c |
| +0x1fc | last_error_id | H | `net_session_connect_selected_provider`, `net_session_create_as_host`, `net_session_create_connection`, `net_session_ctor`, `net_session_init_provider`, `net_session_join`, `net_session_list_available`, `net_session_ping_tick`, `net_session_receive_messages`, `net_session_remove_peer` | - | `0x38c` when the session dies |
| +0x200 | is_host | H | `net_session_create_as_host`, `net_session_join` | `gui_mp_lobby_ctor`, `net_session_broadcast_setting`, `net_session_handle_event_msg`, `net_session_is_host`, `net_session_ping_tick`, `net_session_post_event_msg`, `net_session_remove_ai_racer`, `net_session_return_to_lobby`, `net_session_send_chat_message`, `race_net_load_sync`, `racing_set_damage_enabled`, `racing_tick` | ¹ |
| +0x204 | local_player | H | `gui_mp_lobby_on_child_result`, `net_session_create_as_host`, `net_session_handle_event_msg`, `net_session_join`, `net_session_leave`, `net_session_set_ready_flag`, `race_net_load_sync` | `gui_mp_lobby_ctor`, `gui_mp_lobby_tick`, `net_session_average_remote_latency`, `net_session_broadcast_setting`, `net_session_create_as_host`, `net_session_ctor`, `net_session_dtor`, `net_session_get_peer_clock_ms`, `net_session_handle_event_msg`, `net_session_join`, `net_session_leave`, `net_session_local_player_is_configured`, `net_session_partition_peers_by_relevance`, `net_session_post_scene_done`, `net_session_remove_peer`, `net_session_request_ready_flag`, `net_session_send_chat_message`, `net_session_send_racing_setup_to_peer`, `net_session_set_ready_flag`, `net_session_start`, `net_session_transfer_actor_ownership`, `net_session_validate_car_for_championship`, `opponent_do_recover`, `race_net_load_sync`, `racing_do_gates` | §2d; `racing_do_gates` and `opponent_do_recover` compare the address |
| +0x264 | players_alloc | H | `net_session_ctor` | `net_session_create_as_host`, `net_session_dtor`, `net_session_handle_event_msg`, `net_session_receive_messages` | an empty class |
| +0x268 | players_begin | H | `net_session_ctor`, `net_session_dtor` | `net_session_add_ai_racer`, `net_session_all_peers_ready`, `net_session_average_remote_latency`, `net_session_build_actor_guid_table`, `net_session_can_start`, `net_session_disable_timeout_watchdog`, `net_session_dtor`, `net_session_find_peer_by_id`, `net_session_find_peer_by_slot`, `net_session_get_peer_clock_ms`, `net_session_handle_event_msg`, `net_session_leave`, `net_session_partition_peers_by_relevance`, `net_session_ping_tick`, `net_session_post_event_msg`, `net_session_receive_messages`, `net_session_remove_peer`, `net_session_return_to_lobby`, `net_session_send_event_msg_direct`, `net_session_send_queued_messages`, `racing_activate_racers`, `racing_place_cars` | - |
| +0x26c | players_end | H | `net_session_ctor`, `net_session_dtor`, `net_session_leave`, `net_session_remove_peer` | `net_session_add_ai_racer`, `net_session_all_peers_ready`, `net_session_average_remote_latency`, `net_session_build_actor_guid_table`, `net_session_can_start`, `net_session_disable_timeout_watchdog`, `net_session_dtor`, `net_session_find_peer_by_id`, `net_session_find_peer_by_slot`, `net_session_get_peer_clock_ms`, `net_session_handle_event_msg`, `net_session_leave`, `net_session_partition_peers_by_relevance`, `net_session_ping_tick`, `net_session_post_event_msg`, `net_session_remove_peer`, `net_session_return_to_lobby`, `net_session_send_event_msg_direct`, `net_session_send_queued_messages`, `racing_activate_racers`, `racing_place_cars` | - |
| +0x270 | players_cap | H | `net_session_ctor`, `net_session_dtor` | - | - |
| +0x274 | host_peer_id | H | `net_session_create_as_host`, `net_session_handle_event_msg` | `net_session_add_ai_racer`, `net_session_disable_timeout_watchdog`, `net_session_enable_timeout_watchdog`, `net_session_handle_event_msg`, `net_session_ping_tick`, `net_session_post_event_msg`, `net_session_post_scene_done`, `net_session_remove_ai_racer`, `net_session_remove_peer`, `net_session_request_ready_flag`, `net_session_send_chat_message`, `net_session_send_event_msg_direct`, `net_session_set_ready_flag`, `net_session_transfer_actor_ownership` | most producers address a message to it in `[4]`; on the host such a message is relayed to every other peer (§5b) |
| +0x278 | last_transport_error | M | `net_session_join`, `net_session_peer_flush_send_buffer`, `net_session_send_event_msg_direct`, `net_session_send_queued_messages` | - | Open item 2 |
| +0x27c | ping_elapsed_ms | H | `net_session_ctor`, `net_session_ping_tick` | `net_session_ping_tick` | - |
| +0x280 | watchdog_armed | H | `net_session_create_as_host`, `net_session_ctor`, `net_session_disable_timeout_watchdog`, `net_session_enable_timeout_watchdog`, `net_session_handle_event_msg`, `net_session_leave` | `net_session_ping_tick` | the type-`0xa` acknowledgement sets it too |
| +0x284 | wait_timer_start | H | `net_session_reset_wait_timer` | `net_session_wait_timer_expired` | - |
| +0x288 | roster_dirty | H | `net_session_ctor`, `net_session_handle_event_msg`, `net_session_receive_messages`, `net_session_take_roster_dirty_flag` | `net_session_take_roster_dirty_flag` | - |
| +0x289 | ready_request_pending | H | `net_session_ctor`, `net_session_handle_event_msg`, `net_session_request_ready_flag` | `gui_mp_lobby_tick` | cleared when the host's type-`0x2e` echo returns for our own peer |
| +0x28c | last_send_handle | H | `net_session_peer_flush_send_buffer`, `net_session_send_event_msg_direct`, `net_session_send_queued_messages` (by address, the connection's send slot `+0x5c`) | `net_session_receive_messages` | the transport writes the send's handle through the out-parameter; the send-complete arm (`0xffff0014`, DirectPlay8's send complete) compares the completed handle with it and clears `send_paused` |
| +0x290 | ai_slots_alloc | H | `net_session_ctor` | `net_session_add_ai_racer` | an empty class |
| +0x294 | ai_slots_begin | H | `net_session_ctor`, `net_session_dtor` | `net_session_add_ai_racer`, `net_session_broadcast_setting`, `net_session_clear_ai_racers`, `net_session_dtor`, `net_session_receive_messages`, `net_session_remove_ai_racer`, `racing_add_racer_for_ai_slot` | - |
| +0x298 | ai_slots_end | H | `net_session_clear_ai_racers`, `net_session_ctor`, `net_session_dtor`, `net_session_remove_ai_racer` | `net_session_broadcast_setting`, `net_session_clear_ai_racers`, `net_session_receive_messages`, `net_session_remove_ai_racer` | - |
| +0x29c | ai_slots_cap | H | `net_session_ctor`, `net_session_dtor` | - | - |
| +0x2a0 | suspend_parse | H | `net_session_ctor`, `net_session_handle_event_msg`, `net_session_receive_messages` | `net_session_receive_messages` | - |
| +0x2a4 | resume_offset | H | `net_session_ctor`, `net_session_receive_messages` | `net_session_receive_messages` | - |
| +0x2a8 | unread_2a8 | U | `net_session_ctor` | - | no other access in the class or by a `g_net_session` load in `.text` |
| +0x2ac | events_handled | H | `net_session_ctor`, `net_session_handle_event_msg` | `net_session_handle_event_msg` | - |
| +0x2b0 | unread_2b0 | U | `net_session_ctor` | - | no other access in the class or by a `g_net_session` load in `.text` |
| +0x2b4 | unread_2b4 | U | `net_session_ctor` | - | no other access in the class or by a `g_net_session` load in `.text`; the `+0x2b4` hits in `net_session_broadcast_setting` and the dispatcher are stack locals |
| +0x2b8 | payloads_received | H | `net_session_ctor`, `net_session_receive_messages` | `net_session_receive_messages` | - |
| +0x2bc | batched_flush_count | H | `net_session_ctor`, `net_session_peer_flush_send_buffer` | `net_session_peer_flush_send_buffer` | incremented only on the branch that sends a buffer of two or more messages with its count dword; no reader besides that increment |
| +0x2c0 | receive_transport_ticks | H | `net_session_receive_messages` | `net_session_receive_messages` | - |
| +0x2c4 | send_transport_ticks | H | `net_session_peer_flush_send_buffer`, `net_session_send_queued_messages` | `net_session_peer_flush_send_buffer`, `net_session_send_queued_messages` | - |

¹ `net_session_post_event_msg` sets a message's broadcast flag `[5]` only on the host, for a message addressed to the host itself (`[4]` equals `+0x274`); the receive arms re-post a message for the other peers only when it is
set; `net_session_return_to_lobby` picks state 4 when set and 3 when clear; `race_net_load_sync` and `gui_mp_lobby_ctor` take their host or client branch on it.

### 2a. `state`, the session stage

The state is a stage, written by `net_session_set_state` and by the session's own methods:

| value | meaning | writers |
|---|---|---|
| 0 | no session: no INet provider, or a step failed, with the error's textdb id in `+0x1fc` | `net_session_init_provider` (textdb 900), `net_session_create_connection`, `net_session_connect_selected_provider`, `net_session_create_as_host`, `net_session_join`, `net_session_list_available` (textdb `0x388`), `net_session_ping_tick`, `net_session_remove_peer`, transport id `0xffff0016`, a scene mismatch in the type-`0xb` arm (textdb `0x391`) |
| 1 | idle: a provider exists, no session | `net_session_ctor`, `net_session_leave`, `net_multiplayer_run` after a failed host or join |
| 2 | the selected connection provider is connected | `net_session_connect_selected_provider` |
| 3 | in a session's lobby as a client | `net_session_join`, `net_session_return_to_lobby` |
| 4 | in a session's lobby as the host | `net_session_create_as_host`, `net_session_return_to_lobby` |
| 5 | the game is starting | `net_session_start` (host), the type-6 arm (client) |
| 6 | every player has the same scene loaded | `race_net_load_sync` and the type-`0xb` arm (§5c) |
| 7 | the load handshake is complete and the game runs | `race_net_load_sync`, the type-`0xd` arm (host) and the type-`0xe` arm (client) (§5c) |

Every machine waits for 7 before the race runs, so 7 means "a networked game is running" on host
and client alike. `net_multiplayer_run` refuses on 0; `gui_mp_lobby_tick` closes the lobby with
code `0x3d` on 5 and `0x3f` on 1; the networked physics gates (`box_ai_tick`,
`bottle_post_collision_net_event_msg`, `opponent_do_recover` and others) test `== 7`.

### 2b. `S_net_settings`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | player_name | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_create_as_host`, `net_session_init_provider`, `net_session_join`, `net_session_settings_apply`, `net_session_settings_save` | settings component 10, textdb 623 "Player Name:" |
| +0x20 | provider_type | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_selected_provider_is_valid`, `net_session_settings_save` | combo `0xb`, textdb 624 |
| +0x24 | provider_index | H | `net_session_settings_apply` | `net_session_create_connection`, `net_session_settings_save` | combo `0xc` |
| +0x28 | ip_address | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | component `0x70`, textdb 630 |
| +0x48 | port | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | component `0x71`, textdb 631 |
| +0x68 | serial_port | H | `net_session_settings_apply` | `net_session_settings_save` | component `0xd4` |
| +0x88 | com_port | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | combo `0x32c`, textdb 634 |
| +0x8c | baud_rate | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_settings_save` | combo `0x32d`, textdb 635 |
| +0x90 | stop_bits | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | combo `0x32e`, textdb 636 |
| +0x94 | parity | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | combo `0x32f`, textdb 637 |
| +0x98 | flow_control | H | `net_session_settings_apply`, `net_session_settings_init_defaults` | `net_session_connect_selected_provider`, `net_session_settings_save` | combo `0x330`, textdb 638 |

### 2c. `S_net_game_settings`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | unread_00 | U | `net_session_game_settings_ctor` | - | - |
| +0x4 | unused_04 | U | - | - | no access by the 37 callers of `net_session_game_settings_addr` or by a `g_net_session` load in `.text` |
| +0x8 | unread_08 | U | `net_session_game_settings_ctor` | - | `car_apply_net_event_msg` reads `+0x14` at all three of its accessor calls, never `+0x08` |
| +0xc | unread_0c | U | `net_session_game_settings_ctor` | - | - |
| +0x10 | bottle_send_period_ms | H | `net_session_game_settings_ctor` | `bottle_ai_tick` | - |
| +0x14 | car_send_period_ms | H | `net_session_game_settings_ctor` | `car_apply_net_event_msg`, `car_update` | `car_update` broadcasts when the car's timer `+0x215c` passes it; `car_apply_net_event_msg` derives the collision hold (half of it) and the reduced-snapshot interval (three times it) from it |
| +0x18 | object_send_period_ms | H | `net_session_game_settings_ctor` | `physical_object_apply_net_event_msg`, `physical_object_net_apply_remote_state`, `physical_object_update` | `structs/physical_object.md` |
| +0x1c | box_send_period_ms | H | `net_session_game_settings_ctor` | `box_ai_tick` | `box_ai_tick` posts the box's state when its timer passes it |
| +0x20 | unused_20 | U | - | - | no access by the accessor's callers or by a `g_net_session` load in `.text` |
| +0x38 | game_mode | H | `net_session_game_settings_ctor`, `net_session_handle_event_msg`, `net_session_leave` | `event_msg_player_list_serialize`, `game_tick`, `gui_load_progress_tick`, `gui_mp_lobby_ctor`, `gui_mp_menu_refresh_mode_items`, `net_host_session_start`, `net_multiplayer_run`, `net_session_create_as_host`, `net_session_local_player_is_configured`, `net_session_receive_messages`, `net_session_remove_peer`, `net_wait_for_session_mode` | the disconnect handler's `3..4` gate is a test for a race mode |
| +0x3c | mode_list_alloc | H | `net_session_game_settings_ctor` | - | an empty class |
| +0x40 | mode_list_begin | H | `net_session_game_settings_ctor`, `net_session_game_settings_dtor` | `net_session_game_settings_dtor` | only the destructor reads the vector |
| +0x44 | mode_list_end | H | `net_session_game_settings_ctor`, `net_session_game_settings_dtor` | `net_session_game_settings_dtor` | - |
| +0x48 | mode_list_cap | H | `net_session_game_settings_ctor`, `net_session_game_settings_dtor` | `net_session_game_settings_dtor` | - |
| +0x4c | menu_mode_mask | H | `net_session_game_settings_ctor` | `gui_mp_menu_ctor`, `gui_mp_menu_refresh_mode_items` | - |

### 2d. `S_net_peer`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | peer_id | H | `net_session_create_as_host`, `net_session_join`, `net_session_peer_ctor`, `net_session_peer_ctor_unset` | `net_session_find_peer_by_id`, `net_session_peer_id_for_actor_guid`, `net_session_post_event_msg` | - |
| +0x4 | name | H | `net_session_create_as_host`, `net_session_join`, `net_session_peer_ctor`, `net_session_peer_ctor_unset` | `net_session_handle_event_msg`, `racing_add_racer_for_player` | the chat receiver formats `"%s : %s"` with it |
| +0x24 | driver_model | H | `gui_mp_lobby_on_child_result`, `net_session_create_as_host`, `net_session_join`, `net_session_leave`, `net_session_peer_ctor`, `net_session_peer_ctor_unset` | `gui_mp_lobby_ctor`, `gui_mp_lobby_tick`, `net_session_broadcast_setting`, `net_session_local_player_is_configured`, `racing_add_racer_for_player` | `racing_add_racer_for_player` copies it to `opponent+0x20`; 0 reads as not configured |
| +0x28 | mean_half_rtt | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_update_latency` | `net_session_average_remote_latency`, `net_session_get_peer_clock_ms` | `net_session_average_remote_latency` skips the local record |
| +0x2c | slot | H | `net_session_create_as_host`, `net_session_handle_event_msg`, `net_session_peer_ctor`, `net_session_peer_ctor_unset` | `net_session_broadcast_setting`, `net_session_find_peer_by_slot`, `net_session_get_peer_clock_ms`, `net_session_post_scene_done`, `net_session_remove_peer`, `race_compare_participant_slot` | the host takes 0; a client one past the highest in the roster |
| +0x30 | rtt_samples_alloc | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset` | - | an empty class |
| +0x34 | rtt_samples_begin | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_dtor` | `net_session_peer_dtor`, `net_session_peer_update_latency` | - |
| +0x38 | rtt_samples_end | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_dtor`, `net_session_peer_update_latency` | `net_session_peer_update_latency` | - |
| +0x3c | rtt_samples_cap | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_dtor` | - | - |
| +0x40 | timeout_elapsed_ms | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_update_latency`, `net_session_ping_tick` | `net_session_ping_tick` | - |
| +0x44 | timeout_ms | H | `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_update_latency`, `net_session_ping_tick` | `net_session_ping_tick` | - |
| +0x48 | load_progress | H | `gui_load_progress_set_progress_game_init`, `gui_load_progress_set_progress_mission_open`, `net_session_handle_event_msg`, `net_session_leave`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `race_net_load_sync`, `race_open_menu_scene` | `gui_load_progress_tick` | the type-`0x36` arm stores a peer's value by its slot and sets `roster_dirty`; the two setters store the local value into `session+0x24c`; `net_session_leave` clears it, and `race_net_load_sync` and `race_open_menu_scene` clear every peer's with `scene_loaded`; the tick shows each player's value in component `2000 + i` |
| +0x4c | guid_table_acked | H | `net_session_handle_event_msg`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `race_net_load_sync` | `race_net_load_sync` | for the local record this is `session+0x250` |
| +0x4d | scene_loaded | H | `net_session_handle_event_msg`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `race_net_load_sync`, `race_open_menu_scene` | `race_net_load_sync` | also tested in the session listing; for the local record this is `session+0x251` |
| +0x4e | lobby_ready | H | `net_session_handle_event_msg`, `net_session_leave`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_set_ready_flag` | `gui_mp_lobby_tick`, `gui_widget_net_player_set_row`, `net_session_all_peers_ready` | greys the lobby's Garage and Person buttons |
| +0x50 | send_buffer | H | `net_session_peer_append_message_bytes`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_dtor` | `net_session_peer_append_message_bytes`, `net_session_peer_dtor`, `net_session_peer_flush_send_buffer`, `net_session_post_event_msg` | - |
| +0x54 | send_buffer_cap | H | `net_session_peer_append_message_bytes`, `net_session_peer_ctor`, `net_session_peer_ctor_unset` | `net_session_peer_append_message_bytes` | - |
| +0x58 | send_buffer_used | H | `net_session_peer_append_message_bytes`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_flush_send_buffer` | `net_session_peer_append_message_bytes`, `net_session_peer_flush_send_buffer`, `net_session_post_event_msg` | - |
| +0x5c | send_buffer_messages | H | `net_session_peer_append_message_bytes`, `net_session_peer_ctor`, `net_session_peer_ctor_unset`, `net_session_peer_flush_send_buffer` | `net_session_peer_append_message_bytes`, `net_session_peer_flush_send_buffer` | below 2 the flush sends the message without the count dword |

### 2e. `S_net_ai_slot`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `net_session_add_ai_racer`, `net_session_handle_event_msg` | `racing_add_racer_for_ai_slot` | `net_session_add_ai_racer` copies the name back out of the created racer |
| +0x20 | driver_name_index | H | `racing_add_racer_for_ai_slot` | `racing_add_racer_for_ai_slot` | that write-back is why the function takes a "draw a new one" flag |
| +0x24 | car_model | H | `net_session_add_ai_racer` | `net_session_broadcast_setting`, `racing_add_racer_for_ai_slot` | - |
| +0x28 | slot_index | H | `net_session_add_ai_racer` | `race_compare_participant_slot` | - |

### 2f. `S_net_provider`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | name | H | `net_session_enumerate_providers` | `net_session_dtor` | - |
| +0x4 | driver_index | H | `net_session_enumerate_providers` | `net_session_connect_selected_provider`, `net_session_find_provider_index_by_type` | - |
| +0x8 | type | H | `net_session_enumerate_providers` | `net_session_connect_selected_provider`, `net_session_find_provider_index_by_type`, `net_session_selected_provider_is_valid` | matched against `provider_type` |

## 3. Method family

Every function here is PC-only (§7). The four free session drivers with the `net_` prefix, `net_wait_for_session_mode`, `net_host_session_start`, `net_open_session_browser` and `net_multiplayer_run`, live in `structs/race_session.md`'s unit.

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x00596c50 | `crt_static_init_guard_00596c50` | - | - | The first unit's opening guard |
| 0x00596c70 | `crt_atexit_register_00596c70` | - | - | The guard's `atexit` helper |
| 0x00596c80 | `net_session_peer_ctor` | instance | - | `this = S_net_peer` A peer from an id and a name |
| 0x00596e90 | `net_session_peer_dtor` | instance | - | `this = S_net_peer` Frees the sample vector and the send buffer |
| 0x00596ed0 | `net_session_ctor` | instance | - | Constructs the members and registers the overlay |
| 0x005971f0 | `net_session_dtor` | instance | - | Destroys the members |
| 0x00598f20 | `net_session_game_settings_ctor` | instance | - | `this = S_net_game_settings` Seeds the periods, the mode list and the menu mask |
| 0x00599130 | `net_session_game_settings_dtor` | instance | - | `this = S_net_game_settings` Frees the mode list |
| 0x0059da30 | `crt_static_init_guard_0059da30` | - | - | The second unit's opening guard |
| 0x0059da50 | `crt_atexit_register_0059da50` | - | - | The guard's `atexit` helper |

### 3b. Accessors

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00597710 | `net_session_settings_addr` | - | `return this + 0x110` |
| 0x00597720 | `net_session_game_settings_addr` | - | `return this + 0x1ac` |
| 0x005977f0 | `net_session_set_state` | - | `+0x100 = state` |
| 0x005a14b0 | `net_session_is_host` | - | `return +0x200` |
| 0x005a14c0 | `net_session_get_relevant_peer_list` | - | `return &+0xb0` |

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00596d00 | `net_session_peer_ctor_unset` | - | `this = S_net_peer` The embedded local record's constructor |
| 0x00596d80 | `net_session_peer_update_latency` | - | `this = S_net_peer` 10-sample rolling mean of half-RTT into `peer+0x28`; rearms the 8000 ms peer timeout |
| 0x00596e20 | `net_session_peer_append_message_bytes` | - | `this = S_net_peer` Appends a message to a peer's send buffer |
| 0x00597130 | `net_session_create_connection` | - | Creates the connection at `+0x108` through the driver; state 0 and error 900 on failure |
| 0x00597190 | `net_session_init_provider` | - | Creates the INet driver, the gate that decides whether multiplayer is reachable at all |
| 0x005974c0 | `net_session_enumerate_providers` | - | Refills the provider list; keeps types 1, 2, 4 |
| 0x00597730 | `net_session_settings_save` | - | Copies the block into `g_net_settings_saved`, field by field |
| 0x005977e0 | `net_session_settings_apply_saved` | - | Applies `g_net_settings_saved` |
| 0x00597800 | `net_session_settings_apply` | - | Copies a `0x9c`-byte block into the session |
| 0x005978b0 | `net_session_selected_provider_is_valid` | - | Gates every Multiplayer-menu item |
| 0x005978f0 | `net_session_find_provider_index_by_type` | - | Type to the driver's index, or `0xffffffff` |
| 0x00597930 | `net_session_connect_selected_provider` | - | The gate `net_host_session_start` and `net_open_session_browser` run first |
| 0x00597ab0 | `net_session_create_as_host` | - | Creates the session as host; sets `+0x200`, `+0x274` and state 4 |
| 0x00597ba0 | `net_session_list_available` | - | Formats the enumerated sessions into the list at `+0xe4` |
| 0x00597e00 | `net_session_join` | - | Joins a session as a client; state 3 |
| 0x00597eb0 | `net_session_send_queued_messages` | - | Sends the queue, then flushes every peer's buffer |
| 0x005980b0 | `net_session_peer_flush_send_buffer` | - | Sends a peer's buffer, with or without the count dword |
| 0x00598360 | `net_session_send_event_msg_direct` | - | Sends one message at once, without buffering |
| 0x00598460 | `net_session_post_event_msg` | - | The single exit of every producer: relays on the host, buffers for the addressee |
| 0x005986f0 | `net_session_send_chat_message` | - | Posts type 1 |
| 0x00598860 | `net_session_find_peer_by_id` | - | Lookup by `+0x00`; 0 on a miss |
| 0x00598890 | `net_session_find_peer_by_slot` | - | Lookup by `+0x2c` |
| 0x005988e0 | `net_session_set_name` | - | Forwards to connection slot 8 (§6b) |
| 0x005988f0 | `net_session_leave` | - | The teardown every exit path out of the multiplayer menu runs |
| 0x005989b0 | `net_session_start` | - | Posts type 6; sets state 5 |
| 0x00598a30 | `net_session_set_password` | - | Forwards to connection slot 9 (§6b) |
| 0x00598a40 | `net_session_trim_message_log` | - | Trims the log to 6 lines |
| 0x00598aa0 | `net_session_ping_tick` | - | Posts type 7 every 4000 ms; expires peer timeouts |
| 0x00598be0 | `net_session_get_peer_clock_ms` | - | A peer's clock from the `+0x268` vector, by slot |
| 0x00598c50 | `net_session_average_remote_latency` | - | The mean of the remote players' `+0x28`; the local record is skipped |
| 0x00598d50 | `net_session_build_actor_guid_table` | - | Builds the type-`0xc` actor GUID remap table |
| 0x00598ea0 | `net_session_clear_message_log` | - | Frees the log's lines and empties it |
| 0x00598f00 | `net_session_peer_id_for_actor_guid` | - | The peer whose slot is the GUID's top byte |
| 0x00599170 | `net_session_set_ready_flag` | - | Posts type `0x2e` |
| 0x005991e0 | `net_session_request_ready_flag` | - | A client's ready toggle: posts type `0x2f` and sets `+0x289` |
| 0x00599250 | `net_session_all_peers_ready` | - | ANDs peer `+0x4e` over every player |
| 0x00599280 | `net_session_broadcast_setting` | - | Posts type `0x31`, the lobby's setting broadcast |
| 0x00599420 | `net_session_get_car_model_for_player` | - | The racer's car model for a player record, through `g_racing` |
| 0x00599490 | `net_session_return_to_lobby` | - | The state transition between a finished race and the lobby |
| 0x005994f0 | `net_session_all_players_finished_race` | - | Gates both results screens in a networked race |
| 0x00599540 | `net_session_reset_wait_timer` | - | Paired with `net_session_wait_timer_expired` (25 000 ms) |
| 0x00599560 | `net_session_wait_timer_expired` | - | The 25 s cap on waiting for the host to choose a mode |
| 0x00599580 | `net_session_is_active` | - | Tests the state `+0x100`; 43 callsites in 33 functions |
| 0x005995a0 | `net_session_can_start` | - | Enables the lobby's Start item |
| 0x005995d0 | `net_session_local_player_is_configured` | - | Why the lobby refuses to mark you ready without a car |
| 0x00599610 | `net_session_remove_peer` | - | The disconnect handler |
| 0x005997b0 | `net_session_take_roster_dirty_flag` | - | Reads and clears `+0x288` |
| 0x005997c0 | `net_session_push_system_message` | - | The non-chat half of the message log |
| 0x005999d0 | `net_session_tick` | - | Stores the delta, receives, sends, pings, runs the pause timer |
| 0x00599a40 | `net_session_add_ai_racer` | - | Adds an AI slot and posts type `0x32` |
| 0x00599b50 | `net_session_remove_ai_racer` | - | Removes an AI slot and posts type `0x33` |
| 0x00599c30 | `net_session_disable_timeout_watchdog` | - | Clears `+0x280` and every peer's timeout; posts type 9 |
| 0x00599cc0 | `net_session_enable_timeout_watchdog` | - | The mirror of `net_session_disable_timeout_watchdog` |
| 0x00599d20 | `net_session_clear_ai_racers` | - | The AI-slot half of the session teardown |
| 0x00599d80 | `net_session_validate_car_for_championship` | - | Runs whenever the championship selection changes |
| 0x00599e50 | `net_session_post_scene_done` | - | Posts type `0x10` with the scene identity and the local slot |
| 0x00599fa0 | `net_session_settings_init_defaults` | - | Writes the default settings block |
| 0x0059a070 | `net_session_transfer_actor_ownership` | - | Reassigns an actor's GUID to a slot; optionally posts type `0x26` |
| 0x0059a150 | `net_session_partition_peers_by_relevance` | - | Refills `+0xb4` and `+0xc4` from the players |
| 0x0059a520 | `net_session_build_position_grid` | - | Rebuilds the grid over the scene bounds; tail-called |
| 0x0059a810 | `net_session_decode_position` | - | The decoder half of the position codec |
| 0x0059a8c0 | `net_session_encode_position` | - | The encoder half of the position codec |
| 0x0059da60 | `net_session_receive_messages` | - | Drains the transport, dispatches on the system id (§5a) |
| 0x0059e120 | `net_session_handle_event_msg` | - | The payload dispatcher, about 1900 lines (§5a) |
| 0x005a1510 | `net_session_send_racing_setup_to_peer` | - | Only reachable from the join arm of the receive pump |

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00598cd0 | `net_find_actor_by_guid` | - | The scene actor with a GUID |
| 0x00599ed0 | `net_deactivate_actors_of_peer` | - | Deactivates a leaving peer's actors |
| 0x0059aaa0 | `net_encode_fixed16` | - | float to i16, sign and magnitude; the quantiser of the actor-state payloads |
| 0x0059ab10 | `net_decode_fixed16` | - | i16 -> float; the inverse of `net_encode_fixed16` |
| 0x005a14f0 | `matrix4_element_addr` | - | Shared address helper of the second unit |
| 0x005a17b0 | `vec_index_addr_stride_0x4` | - | Shared address helper |
| 0x005a17c0 | `std_vector_begin` | - | Shared vector accessor |
| 0x005a17d0 | `folded_get_dword_0x18` | - | A shared accessor fold, not this class's |

## 4. Who adds and removes records

A census of every write to `+0x264..+0x26c` and `+0x290..+0x298` in `Game.exe` closes the set:

| vector | adds | removes |
|---|---|---|
| human players `+0x268` | `net_session_create_as_host` (the local record), the type-`0xffff0007` arm (host), the type-3 roster arm and the type-4 arm (client) | `net_session_remove_peer` (from the type-`0xffff0009`, type-5 and type-`0xb` arms, `net_session_ping_tick` and the lobby's kick), `net_session_leave` |
| AI opponents `+0x294` | `net_session_add_ai_racer`, the type-`0x32` arm | `net_session_remove_ai_racer`, `net_session_clear_ai_racers` |

## 5. The message layer

### 5a. Receive

`net_session_receive_messages` (`0x0059da60`) is the partner of `net_session_send_queued_messages`;
`net_session_tick` calls them in that order. It drains the INet transport at `+0x108` and dispatches
on the system id:

| id | meaning |
|---|---|
| `0xffff0007` | a peer joined — allocate the `0x60`-byte record, assign the next free slot, post the roster and player-add messages, push the join banner (textdb `0x38e` + `0x390`), and in a race send the racing setup |
| `0xffff0009` | a peer left — `net_session_remove_peer` |
| `0xffff0011` | a payload — handed to `net_session_handle_event_msg` |
| `0xffff0014` | a send completed: clears `+0xa5` when the handle is `+0x28c`'s |
| `0xffff0016` | the session died — `+0x100 = 0`, textdb `0x38c` into `+0x1fc` |

`net_session_handle_event_msg` (`0x0059e120`, ~1900 lines) is the payload dispatcher: it switches on
`*wire - 1` (the table is biased by one because type 0 has no arm), builds the matching message class
on the stack, calls its slot-3 deserializer, applies the result, and — where the local peer is the
host — rebuilds the message with `event_msg_set_body_from_wire` and re-posts it. It is the single
consumer of 27 of the 39 message descriptors (`structs/net_event_msg.md`).

### 5b. Send

Every outbound message is an `operator_new(0x18)` record with the type in `[2]` and the addressee's
peer id in `[4]`, for most producers `g_net_session+0x274`, the host. It is posted through
`net_session_post_event_msg`; on the host, a message addressed to the host is relayed to every other
peer. Type `5`, the type-`7` echo and the host's type-`0xb` answer are sent at once by
`net_session_send_event_msg_direct`. Every type and its producer is in `structs/net_event_msg.md` §7;
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

### 5c. The load handshake

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

## 6. The DirectPlay / `inet.dll` layer

The transport half of the session, reached from the multiplayer front end
(`structs/multiplayer.md`).

### 6a. The saved settings

`net_session_settings_save` copies the block the other way, field by field, into the saved copy,
whose fields carry their own names: `g_net_settings_saved` (the player name),
`g_net_settings_saved_provider_type`, `_provider_index`, `_ip_address`, `_port`, `_serial_port`,
and the five serial dwords `g_net_settings_saved_com_port`, `_baud_rate`, `_stop_bits`, `_parity`,
`_flow_control` (`0x00672558`..`0x00672568`).

### 6b. The two 11-byte connection forwarders

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

### 6c. What the connection does with them

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

The eight used vtable slots of the connection: `+0x14`/`+0x18`/`+0x1c` enumerate sessions, `+0x20` set session name, `+0x24` set password, `+0x34` create session, `+0x50` local player id, `+0x54` set player name.

## 7. PC vs PS2

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

## 8. Related

`structs/game.md` (the singleton this is not) · `structs/multiplayer.md` (the front end) · `structs/race_session.md` (the free session drivers) · `structs/net_event_msg.md` (the message classes) · `structs/racing.md` · `structs/console.md` · `basic/artifacts/dll.md` (the five Illusion DLLs, `inet.dll` among them).

## 9. Open items

| # | Item | State |
|---|---|---|
| 1 | Outside users | `gui_`, `racing_` and other users of the session beyond those §2 names were not swept, so their cells are a lower bound; `race_game_loop_net` is swept |
| 2 | `last_transport_error` | the value is the connection's slot `+0xc` result; that slot in `INet.dll` is not read, so the meaning stays [M] |
