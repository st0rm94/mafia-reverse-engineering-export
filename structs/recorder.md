# `C_recorder` — the keyframe track recorder and `.rep` / `.tck` loader

**Created** 2026-08-04 18:07 +0200 · **Modified** 2026-11-16 12:35 +0100

`C_recorder` owns every scene object's keyframe **track**: it creates them, names them, keeps the
object↔frame bindings needed to re-attach a track after a scene reload, and loads `.tck` track files
and `.rep` replays from disk, whose camera, fade, script, speech and subtitle keys it plays back on a
remappable clock. Its per-object client interface is `C_actor_track` (`structs/actor_track.md`), which
every `C_actor`-family class inherits. The class name is the developers' own (§5).

| | |
|---|---|
| PC prefix | `recorder_` |
| PC kind (`+0x10`) | - |
| PC vtable | `g_recorder_vt` @ `0x0063d0e8` (53 slots) |
| PC base class | - |
| PC instance size | `0x12c`, one instance: the singleton at `0x00656368` (§4); the next global starts at `0x00656494` |
| PC constructor | `recorder_ctor` @ `0x00545130` |
| PC translation unit | `[0x005407f0, 0x005469a0)`: every start named |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct C_recorder {                   // 0x12c bytes, the singleton
    /* +0x00 */ void *vtable;                  // [H] g_recorder_vt: 53 slots (§6)
    /* +0x04 */ uint8_t idle_alloc;            // [H] allocator byte of the idle client vector
    /* +0x08 */ C_actor **idle_begin;          // [H] clients with no key stream playing
    /* +0x0c */ C_actor **idle_end;            // [H] one past the last idle client
    /* +0x10 */ C_actor **idle_cap;            // [H] end of the allocation
    /* +0x14 */ uint8_t active_alloc;          // [H] allocator byte of the active client vector
    /* +0x18 */ C_actor **active_begin;        // [H] clients being stepped
    /* +0x1c */ C_actor **active_end;          // [H] one past the last active client
    /* +0x20 */ C_actor **active_cap;          // [H] end of the allocation
    /* +0x24 */ std_vector unread_vector_24;   // [U] built by recorder_ctor, freed by recorder_dtor; no reader (.text scan)
    /* +0x34 */ uint8_t objects_alloc;         // [H] allocator byte of the object table
    /* +0x38 */ S_anim_object **objects_begin; // [H] one per name-table row (§1g); recorder_rebuild_object_table
    /* +0x3c */ S_anim_object **objects_end;   // [H] one past the last element
    /* +0x40 */ S_anim_object **objects_cap;   // [H] end of the allocation
    /* +0x44 */ S_cam_key *camera_keys;        // [H] 0x40-byte records from the .rep (§1i)
    /* +0x48 */ S_cam_target_key *target_keys; // [H] 0x38-byte records from the .rep (§1j)
    /* +0x4c */ uint32_t camera_key_count;     // [H] entries in camera_keys
    /* +0x50 */ uint32_t target_key_count;     // [H] entries in target_keys
    /* +0x54 */ uint32_t camera_cursor;        // [H] the camera key the clock has passed
    /* +0x58 */ uint32_t target_cursor;        // [H] the target key the clock has passed
    /* +0x5c */ uint32_t fade_key_count;       // [H] entries in fade_keys
    /* +0x60 */ S_fade_key *fade_keys;         // [H] 0x20-byte records (§1k)
    /* +0x64 */ uint32_t fade_cursor;          // [H] the fade key the clock has passed
    /* +0x68 */ S_timeline_key *timeline_keys; // [H] 0xc-byte clock remap records (§8)
    /* +0x6c */ uint32_t timeline_key_count;   // [H] entries in timeline_keys
    /* +0x70 */ uint32_t timeline_cursor;      // [H] the timeline key source_clock_ms has passed
    /* +0x74 */ uint32_t speech_key_count;     // [H] entries in speech_keys
    /* +0x78 */ uint32_t speech_cursor;        // [H] the next speech key
    /* +0x7c */ void *speech_keys;             // [H] 0x24-byte records for speech_apply_recorder_key
    /* +0x80 */ S_recorder_names names;        // [H] embedded name table (§1a)
    /* +0x90 */ S_recorder_bindings bindings;  // [H] embedded binding table (§1b)
    /* +0xb4 */ uint32_t subtitle_key_count;   // [H] entries in subtitle_keys
    /* +0xb8 */ uint32_t subtitle_cursor;      // [H] the next subtitle key
    /* +0xbc */ void *subtitle_keys;           // [H] 0xc-byte records for hud_subtitles_add_from_record
    /* +0xc0 */ uint32_t script_key_count;     // [H] entries in script_keys
    /* +0xc4 */ int32_t script_cursor;         // [H] starts at -1
    /* +0xc8 */ S_script_key *script_keys;     // [H] 0x28-byte records run by mission_global_program_run (§1l)
    /* +0xcc */ uint32_t frame_event_count;    // [H] entries in frame_events
    /* +0xd0 */ int32_t frame_event_cursor;    // [H] starts at -1
    /* +0xd4 */ S_frame_event_key *frame_events; // [H] 0x28-byte records: show or hide a named frame (§1m)
    /* +0xd8 */ uint32_t morph_key_count;      // [H] entries in morph_keys
    /* +0xdc */ int32_t morph_cursor;          // [H] starts at -1
    /* +0xe0 */ void *morph_keys;              // [H] 0x2c-byte records for speech_morph_frame_list_add
    /* +0xe4 */ uint32_t mode;                 // [H] §8
    /* +0xe8 */ uint32_t prev_mode;            // [H] the mode of the last tick
    /* +0xec */ void *file_block_0;            // [H] first block of a loaded .rep; ctor 1 = none
    /* +0xf0 */ void *file_block_1;            // [H] second block of a loaded .rep
    /* +0xf4 */ void *file_block_2;            // [H] third block of a loaded .rep
    /* +0xf8 */ void *file_block_3;            // [H] fourth block of a loaded .rep
    /* +0xfc */ S_actor_track_pose captured_pose; // [H] filled by actor_track_apply_pose_at_time; no reader (.text scan); §1h
    /* +0x11c */ uint32_t source_clock_ms;      // [H] the clock before the timeline remap
    /* +0x120 */ uint32_t playback_clock_ms;    // [H] what every track apply compares (g_recorder_playback_clock_ms)
    /* +0x124 */ float clock_rate;              // [H] rate of the current timeline key; ctor 1.0
    /* +0x128 */ uint8_t camera_idle;           // [H] cleared while a camera key plays
    /* +0x129 */ uint8_t timeline_enabled;      // [H] gate of the remap; ctor 1
} C_recorder;
```

**Why the key lists are count + pointer pairs and not vectors:** they point into the four blocks a
`.rep` load allocates (`file_block_0` .. `_3`), which `recorder_end_sequence_and_restore_scene` frees
as one, so they never grow.

### 1a. `S_recorder_names` — the name ↔ id table

```c
typedef struct S_recorder_names {             // 0x10 bytes; recorder + 0x80
    /* +0x00 */ void *vtable;                    // [H] 0x0063d0d0, six serialisation slots
    /* +0x04 */ int32_t count;                   // [H] live rows
    /* +0x08 */ S_name_row *rows;                // [H] 0x38 bytes each (§1c)
    /* +0x0c */ uint32_t next_id;                // [H] ids start at 0x100
} S_recorder_names;
```

Its own vtable is a six-slot serialisation interface: `recorder_names_read_dta`, `_read_stdio`,
`_copy_from_block`, `_write_stdio`, `_copy_to_block`, `_serialized_size`.

### 1b. `S_recorder_bindings` — the track ↔ object binding table

```c
typedef struct S_recorder_bindings {          // 0x24 bytes; recorder + 0x90
    /* +0x00 */ void *key_arena;                 // [H] the loaded key block; rows with mode 1 keep it
    /* +0x04 */ uint8_t rows_alloc;              // [H] allocator byte of the rows
    /* +0x08 */ S_binding_row *rows_begin;       // [H] 0x58 bytes each (§1e)
    /* +0x0c */ S_binding_row *rows_end;         // [H] one past the last row
    /* +0x10 */ S_binding_row *rows_cap;         // [H] end of the allocation
    /* +0x14 */ uint8_t values_alloc;            // [H] allocator byte of the values
    /* +0x18 */ S_binding_value *values_begin;   // [H] 0x28 bytes each: {name, value} set by the script
    /* +0x1c */ S_binding_value *values_end;     // [H] one past the last value
    /* +0x20 */ S_binding_value *values_cap;     // [H] end of the allocation
} S_recorder_bindings;
```

`recorder_add_track_binding` is a member of this sub-object, not of `C_recorder`: its name is right and
its prefix is one level off.

### 1c. `S_name_row` — one row of the name table

```c
typedef struct S_name_row {                   // 0x38 bytes
    /* +0x00 */ int32_t id;                   // [H] from next_id, starts at 0x100
    /* +0x04 */ char    name[0x30];           // [H] a track or object name
    /* +0x34 */ int32_t value;                // [H] recorder_set_name_value
} S_name_row;
```

`name` is `0x30` bytes because `recorder_names_read_stdio` and `_copy_to_block` move whole `0x38`-byte
rows; no longer name fits a row.

### 1d. `S_track` — the track object

Allocated by `recorder_new_track`, `0x5c` bytes.

```c
typedef struct S_track {                      // 0x5c bytes
    /* +0x00 */ void    *cursor_current;      // [H] the key the clock is past
    /* +0x04 */ void    *cursor_next;         // [H] the key the clock is bracketed against
    /* +0x08 */ void    *cursor_last_type2;   // [H] the last key with type & 2
    /* +0x0c */ void    *unread_0c;           // [U] zeroed by the two initialisers and three loaders; no reader (class census)
    /* +0x10 */ void    *cursor_last_type3;   // [H] the last key with type & 3
    /* +0x14 */ int32_t  last_key_time;       // [H] the game's own name lastKeyTime; seeded -666
    /* +0x18 */ void    *first_chunk;         // [H] §7b
    /* +0x1c */ void    *current_chunk;       // [H] §7b
    /* +0x20 */ char     name[0x24];          // [H] act_%d, hum_%d, ... (§7a)
    /* +0x44 */ uint32_t key_size[4];         // [H] {8, size1, size2, size3}; indexed by key[1]
    /* +0x54 */ uint8_t  end_reached;         // [H] the clock passed last_key_time
    /* +0x58 */ C_actor *owner;               // [H] written by recorder_add_track
} S_track;
```

`key_size[4]`: entry 0 is the 8-byte chunk-link record (§7b); entries 1 to 3 are the key sizes of the
owner's class, the three arguments `recorder_new_track` receives from each `*_track_create` override.

### 1e. `S_binding_row` — one row of the binding table

```c
typedef struct S_binding_row {                // 0x58 bytes
    /* +0x00 */ uint32_t mode;                // [H] 1 = keys from the loaded file (§6c)
    /* +0x04 */ uint32_t owner_kind;          // [H] the actor's +0x10; a row is adopted only on a match
    /* +0x08 */ char     frame_name[0x24];    // [H] "<NONE>", else the bound frame's name
    /* +0x2c */ char     binding_name[0x24];  // [H] the lookup key, copied from track +0x20
    /* +0x50 */ C_actor *object;              // [H] cleared on detach
    /* +0x54 */ S_track *track;               // [H] also written into the object's +0x08
} S_binding_row;
```

Both name arrays are `0x24` bytes because `track + 0x20` is: `recorder_new_track` formats at most
`act_%d`-style names there and the row copies it whole.

### 1f. `S_key_chunk` — one chunk of a key stream

```c
typedef struct S_key_chunk {                  // 0x1800 bytes from operator_new
    /* +0x00 */ struct S_key_chunk *next;     // [H] 0 on the last
    /* +0x04 */ void    *soft_limit;          // [H] chunk + 0xff0
    /* +0x08 */ uint8_t  records[0x17f8];     // [H] the first record starts here (§7b)
} S_key_chunk;
```

`records` is the rest of the allocation; the allocator chains at `soft_limit`, so only the first
`0xfe8` bytes are ever used (§7b).

### 1g. `S_anim_object` — one object-table element

`recorder_load_anim_by_name` allocates it, `0x48` bytes, opens the animation set from `anims\%s` and
has `recorder_load_tck` fill the `.tck` record at `+0x04` (§5a; `out` there is this `+0x04`).

```c
typedef struct S_anim_object {                // 0x48 bytes
    /* +0x00 */ I3D_animation_set *anim_set;  // [H] released by recorder_release_object_table
    /* +0x04 */ uint32_t duration_ms;         // [H] .tck +0x1c
    /* +0x08 */ uint32_t sample_period_ms;    // [H] .tck +0x20
    /* +0x0c */ float    vec_a[3];            // [H] .tck +0x04
    /* +0x18 */ float    vec_b[3];            // [H] .tck +0x10
    /* +0x24 */ uint8_t  samples_a_alloc;     // [H] allocator byte
    /* +0x28 */ float   *samples_a_begin;     // [H] three floats per sample
    /* +0x2c */ float   *samples_a_end;       // [H] one past the last sample; recorder_load_tck appends here
    /* +0x30 */ float   *samples_a_cap;       // [H] end of the allocation
    /* +0x34 */ uint8_t  samples_b_alloc;     // [H] allocator byte
    /* +0x38 */ float   *samples_b_begin;     // [H] three floats per sample
    /* +0x3c */ float   *samples_b_end;       // [H] one past the last sample; recorder_load_tck appends here
    /* +0x40 */ float   *samples_b_cap;       // [H] end of the allocation
    /* +0x44 */ uint8_t  has_samples_b;       // [H] the second array is non-empty
} S_anim_object;
```

`vec_a[3]` and `vec_b[3]` are `.tck` vec3s. The two sample arrays are vectors because
`stl_insert_n_grow_stride_0xc` appends to them one 12-byte sample at a time.

### 1h. `S_actor_track_pose` — the captured pose at `+0xfc`

```c
typedef struct S_actor_track_pose {           // 0x20 bytes; recorder + 0xfc
    /* +0x00 */ uint8_t valid;                // [H] 1 when a key with bit 0x4000 plays; no reader (.text scan)
    /* +0x04 */ float   pos[3];               // [H] the frame's world position; no reader (.text scan)
    /* +0x10 */ float   rot[4];               // [H] the frame's world rotation as a quaternion; no reader (.text scan)
} S_actor_track_pose;
```

The member is the global `g_actor_track_captured_pose` (`0x00656464` = `0x00656368 + 0xfc`) of
`structs/actor_track.md` §6: the same bytes. `actor_track_apply_pose_at_time` fills it when the key
word it plays back has bit `0x4000`: `pos` from the frame's `+0x40..+0x48`, `rot` by `S_quat::Make` of
the frame's world matrix; `recorder_ctor` clears `valid`. Nothing reads it: every dword of the image
whose value falls in `0x00656464..0x00656483` is one of the eight stores (plus four text bytes at
`0x006512a9`), and no recorder method reads `+0xfc..+0x11b`.

### 1i. `S_cam_key` — one camera key

```c
typedef struct S_cam_key {                    // 0x40 bytes; camera_keys
    /* +0x00 */ uint32_t time_ms;             // [H] compared with source_clock_ms
    /* +0x04 */ uint32_t interp_time_ms;      // [H] the time base of the Hermite parameter
    /* +0x08 */ uint32_t flags;               // [H] bits 0..1 mark a bracket key (below)
    /* +0x0c */ float    pos[3];              // [H] the camera position
    /* +0x18 */ float    tangent_out[3];      // [H] the out tangent of this key
    /* +0x24 */ float    tangent_in[3];       // [H] the in tangent, read from the next key
    /* +0x30 */ float    roll;                // [H] the camera roll
    /* +0x34 */ float    fov;                 // [H] field of view; 666.0 ends the sequence
    /* +0x38 */ float    ease_in;             // [H] the first shape value of curve_ease_in_out
    /* +0x3c */ float    ease_out;            // [H] the second, read from the next key
} S_cam_key;
```

`pos`, `tangent_out` and `tangent_in` are one vec3 each. `recorder_tick_tracks` brackets
`source_clock_ms` between two keys and `recorder_interp_key_pos_and_pair` interpolates between them: the
parameter is `(clock - key.interp_time_ms) / (next.interp_time_ms - key.interp_time_ms)`, shaped by
`curve_ease_in_out(t, key.ease_in, next.ease_out)`; the Hermite terms use `key.pos`, `next.pos`,
`key.tangent_out` and `next.tangent_in`, and `roll` and `fov` are blended with the two position weights
alone. **`flags`:** the bracket search stops only on a key with bit 0 or bit 1 set; bit 1 on the start
key moves the start two keys on, on the end key the end one key on. A key whose `fov` is 666.0
(`0x44268000`) ends the sequence: `recorder_tick_tracks` calls slot 0 with mode 0.

### 1j. `S_cam_target_key` — one camera-target key

```c
typedef struct S_cam_target_key {             // 0x38 bytes; target_keys
    /* +0x00 */ uint32_t time_ms;             // [H] compared with source_clock_ms
    /* +0x04 */ uint32_t interp_time_ms;      // [H] the time base of the Hermite parameter
    /* +0x08 */ uint32_t flags;               // [H] as in S_cam_key
    /* +0x0c */ float    pos[3];              // [H] the point the camera looks at
    /* +0x18 */ float    tangent_out[3];      // [H] the out tangent of this key
    /* +0x24 */ float    tangent_in[3];       // [H] the in tangent, read from the next key
    /* +0x30 */ float    ease_in;             // [H] the first shape value of curve_ease_in_out
    /* +0x34 */ float    ease_out;            // [H] the second, read from the next key
} S_cam_target_key;
```

The record of §1i without `roll` and `fov`, interpolated the same way by `recorder_interp_key_pos`.

### 1k. `S_fade_key` — one fade key

```c
typedef struct S_fade_key {                   // 0x20 bytes; fade_keys
    /* +0x00 */ uint32_t time_and_flags;      // [H] low 24 bits the time, high bits the flags of §8
    /* +0x04 */ float    fade_in_ms;          // [H] the fade's length after this key's time (flag 0x80000000)
    /* +0x08 */ float    fade_out_ms;         // [H] the fade's length before this key's time (flag 0x40000000)
    /* +0x0c */ uint32_t colour;              // [H] the tint passed to hud_set_screen_tint
    /* +0x10 */ char     texture_name[0x10];  // [H] the overlay texture, with flag 0x20000000
} S_fade_key;
```

`texture_name` fills the record to its end.

### 1l. `S_script_key` — one script key

```c
typedef struct S_script_key {                 // 0x28 bytes; script_keys
    /* +0x00 */ uint32_t time_ms;             // [H] compared with source_clock_ms
    /* +0x04 */ char     command[0x24];       // [H] the script text mission_global_program_run runs
} S_script_key;
```

### 1m. `S_frame_event_key` — one frame-event key

```c
typedef struct S_frame_event_key {            // 0x28 bytes; frame_events
    /* +0x00 */ uint32_t time_ms;             // [H] compared with source_clock_ms
    /* +0x04 */ uint32_t hide;                // [H] 0 shows the frame, 1 hides it
    /* +0x08 */ char     frame_name[0x20];    // [H] looked up by name in the mission scene
} S_frame_event_key;
```

`command` and `frame_name` fill their records to the end.

## 2. Field evidence

`C_recorder`: a register-tag census of every `this + N` access in its methods (ECX on entry, followed
through moves and stack slots), of `mov reg, 0x656368`, and of absolute operands inside the singleton.
The name and binding methods are excluded here (their receiver is the sub-object) and counted in their
own tables. Records: the bodies of the functions named. The key records of §1i-§1m are file data that
`recorder_load_replay_file` reads whole into the blocks of `file_block_0`..`_3`, so it is every one of
their fields' writer; `recorder_save_replay_file` writes the blocks back whole.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `recorder_ctor`, `recorder_tick_tracks` | `recorder_add_track`, `recorder_attach_existing_track`, `recorder_attach_object_binding`, `recorder_bind_object_by_name`, `recorder_binding_block_write`, `recorder_end_sequence_and_restore_scene`, `recorder_get_or_create_name_id`, `recorder_load_anim_by_name`, `recorder_load_anim_for_object`, `recorder_load_replay_file`, `recorder_open_speech_sound_for_text_id`, `recorder_rebuild_object_table`, `recorder_release_all_key_streams`, `recorder_set_track_active`, `recorder_stop`, `recorder_tick_tracks`, `recorder_unbind_object_by_name` | - |
| +0x04 | idle_alloc | H | `recorder_ctor` | `recorder_add_track`, `recorder_attach_existing_track`, `recorder_end_sequence_and_restore_scene`, `recorder_set_track_active` | - |
| +0x08 | idle_begin | H | `recorder_ctor`, `recorder_dtor` | `recorder_bindings_compact_and_rebind`, `recorder_ctor`, `recorder_dtor`, `recorder_has_track`, `recorder_has_track_in_either_list`, `recorder_load_replay_file`, `recorder_release_all_key_streams`, `recorder_remove_track`, `recorder_set_clock`, `recorder_set_mode`, `recorder_set_track_active`, `recorder_snapshot_track_list`, `recorder_tick_tracks` | §4 |
| +0x0c | idle_end | H | `recorder_ctor`, `recorder_dtor`, `recorder_remove_track`, `recorder_set_track_active` | `recorder_attach_existing_track`, `recorder_bindings_compact_and_rebind`, `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_has_track`, `recorder_has_track_in_either_list`, `recorder_load_replay_file`, `recorder_release_all_key_streams`, `recorder_remove_track`, `recorder_set_clock`, `recorder_set_mode`, `recorder_set_track_active`, `recorder_snapshot_track_list`, `recorder_tick_tracks` | - |
| +0x10 | idle_cap | H | `recorder_ctor`, `recorder_dtor` | `recorder_add_track`, `recorder_attach_existing_track` (by address, `std_vector4_insert_n`), `recorder_set_track_active` (by address, `std_vector4_insert_n`) | - |
| +0x14 | active_alloc | H | `recorder_ctor` | `recorder_end_sequence_and_restore_scene`, `recorder_set_track_active` | - |
| +0x18 | active_begin | H | `recorder_ctor`, `recorder_dtor` | `recorder_bindings_compact_and_rebind`, `recorder_ctor`, `recorder_dtor`, `recorder_has_track`, `recorder_has_track_in_either_list`, `recorder_release_all_key_streams`, `recorder_remove_track`, `recorder_set_clock`, `recorder_set_mode`, `recorder_set_track_active`, `recorder_snapshot_track_list`, `recorder_tick_tracks` | §4 |
| +0x1c | active_end | H | `recorder_ctor`, `recorder_dtor`, `recorder_remove_track`, `recorder_set_track_active` | `recorder_bindings_compact_and_rebind`, `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_has_track`, `recorder_has_track_in_either_list`, `recorder_release_all_key_streams`, `recorder_remove_track`, `recorder_set_clock`, `recorder_set_mode`, `recorder_set_track_active`, `recorder_snapshot_track_list`, `recorder_tick_tracks` | - |
| +0x20 | active_cap | H | `recorder_ctor`, `recorder_dtor` | `recorder_set_track_active` (by address, `std_vector4_insert_n`) | - |
| +0x24 | unread_vector_24 | U | `recorder_ctor`, `recorder_dtor` | `recorder_ctor`, `recorder_dtor` | the two loads only build and free it; no other access, and no dword of the image points into `0x0065638c..0x0065639b` (`.text` scan) |
| +0x34 | objects_alloc | H | `recorder_ctor` | `recorder_ctor`, `recorder_rebuild_object_table` | - |
| +0x38 | objects_begin | H | `recorder_dtor` | `recorder_dtor`, `recorder_get_object_by_id`, `recorder_get_object_by_name`, `recorder_release_object_table` | - |
| +0x3c | objects_end | H | `recorder_dtor`, `recorder_release_object_table` | `recorder_get_object_by_id`, `recorder_get_object_by_name`, `recorder_get_or_create_name_id`, `recorder_rebuild_object_table`, `recorder_release_object_table` | - |
| +0x40 | objects_cap | H | `recorder_dtor` | - | - |
| +0x44 | camera_keys | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x48 | target_keys | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_load_replay_file`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x4c | camera_key_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_load_replay_file`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x50 | target_key_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_load_replay_file`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x54 | camera_cursor | H | `recorder_load_replay_file`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0x58 | target_cursor | H | `recorder_load_replay_file`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0x5c | fade_key_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x60 | fade_keys | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0x64 | fade_cursor | H | `recorder_ctor`, `recorder_load_replay_file`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0x68 | timeline_keys | H | `recorder_load_replay_file` | `recorder_advance_clock`, `recorder_end_sequence_and_restore_scene`, `recorder_save_replay_file`, `recorder_set_clock` | - |
| +0x6c | timeline_key_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_advance_clock`, `recorder_save_replay_file`, `recorder_set_clock` | - |
| +0x70 | timeline_cursor | H | `recorder_advance_clock`, `recorder_ctor`, `recorder_load_replay_file`, `recorder_rewind_timeline_cursor`, `recorder_set_clock` | `recorder_advance_clock`, `recorder_set_clock` | - |
| +0x74 | speech_key_count | H | `recorder_ctor`, `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_size`, `recorder_replay_state_write`, `recorder_tick_tracks` | - |
| +0x78 | speech_cursor | H | `recorder_replay_state_read`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0x7c | speech_keys | H | `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_write`, `recorder_tick_tracks` | - |
| +0x80 | names | H | `recorder_ctor` (by address, `recorder_names_ctor`, `recorder_names_clear`), `recorder_end_sequence_and_restore_scene` (by address, `recorder_names_reset`), `recorder_load_replay_file` (by address, the table's `recorder_names_read_dta` slot) | `recorder_bind_object_by_name`, `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_get_object_by_name`, `recorder_get_or_create_name_id`, `recorder_get_or_create_name_id_direct`, `recorder_load_replay_file`, `recorder_rebuild_object_table`, `recorder_save_replay_file`, `recorder_unbind_object_by_name` | - |
| +0x90 | bindings | H | `recorder_ctor` (by address, `recorder_bindings_init`), `recorder_add_track` (by address, `recorder_add_track_binding`), `recorder_attach_object_binding` (by address, `recorder_bindings_attach_object`), `recorder_remove_track` (by address, `recorder_bindings_remove_for_object`), `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file`, `recorder_release_all_key_streams` (by address, `recorder_bindings_clear_all`), `recorder_dtor` | `recorder_add_track`, `recorder_attach_object_binding`, `recorder_binding_block_size`, `recorder_binding_block_write`, `recorder_ctor`, `recorder_dtor`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file`, `recorder_release_all_key_streams`, `recorder_remove_track`, `recorder_save_replay_file` | - |
| +0xb4 | subtitle_key_count | H | `recorder_ctor`, `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_size`, `recorder_replay_state_write`, `recorder_set_clock`, `recorder_tick_tracks` | - |
| +0xb8 | subtitle_cursor | H | `recorder_replay_state_read`, `recorder_set_clock`, `recorder_tick_tracks` | `recorder_set_clock`, `recorder_tick_tracks` | - |
| +0xbc | subtitle_keys | H | `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_write`, `recorder_set_clock`, `recorder_tick_tracks` | - |
| +0xc0 | script_key_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0xc4 | script_cursor | H | `recorder_ctor`, `recorder_load_replay_file`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0xc8 | script_keys | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0xcc | frame_event_count | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0xd0 | frame_event_cursor | H | `recorder_ctor`, `recorder_load_replay_file`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0xd4 | frame_events | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_save_replay_file`, `recorder_tick_tracks` | - |
| +0xd8 | morph_key_count | H | `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_size`, `recorder_replay_state_write`, `recorder_tick_tracks` | - |
| +0xdc | morph_cursor | H | `recorder_ctor`, `recorder_replay_state_read`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0xe0 | morph_keys | H | `recorder_ctor`, `recorder_replay_state_read` | `recorder_end_sequence_and_restore_scene`, `recorder_replay_state_read`, `recorder_replay_state_write`, `recorder_tick_tracks` | - |
| +0xe4 | mode | H | `recorder_ctor`, `recorder_set_mode` | `recorder_advance_clock`, `recorder_get_mode`, `recorder_is_sequence_running`, `recorder_set_clock`, `recorder_set_mode`, `recorder_sync_stream_to_clock`, `recorder_tick_tracks` | - |
| +0xe8 | prev_mode | H | `recorder_ctor`, `recorder_set_mode`, `recorder_tick_tracks` | `recorder_set_mode`, `recorder_tick_tracks` | - |
| +0xec | file_block_0 | H | `recorder_ctor`, `recorder_load_replay_file` | `recorder_load_replay_file`, `recorder_release_all_key_streams` | §8 |
| +0xf0 | file_block_1 | H | `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | - |
| +0xf4 | file_block_2 | H | `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | - |
| +0xf8 | file_block_3 | H | `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | `recorder_end_sequence_and_restore_scene`, `recorder_load_replay_file` | - |
| +0xfc | captured_pose | H | `actor_track_apply_pose_at_time`, `recorder_ctor` | - | an `S_actor_track_pose` (§1h); no reader (`.text` scan) |
| +0x11c | source_clock_ms | H | `recorder_advance_clock`, `recorder_ctor`, `recorder_load_replay_file`, `recorder_set_clock` | `recorder_advance_clock`, `recorder_get_source_clock_ms`, `recorder_tick_tracks` | - |
| +0x120 | playback_clock_ms | H | `recorder_advance_clock`, `recorder_ctor`, `recorder_load_replay_file`, `recorder_set_clock` | `actor_track_apply_key_texture_anim`, `actor_track_apply_pose_at_clock`, `car_track_apply_pose_at_clock`, `car_track_apply_pose_at_time`, `folded_track_apply_rigid_pose_at_clock`, `human_track_apply_pose_at_clock`, `recorder_get_playback_clock_ms`, `recorder_set_clock`, `recorder_tick_tracks` | - |
| +0x124 | clock_rate | H | `recorder_advance_clock`, `recorder_ctor`, `recorder_set_clock` | - | - |
| +0x128 | camera_idle | H | `recorder_ctor`, `recorder_end_sequence_and_restore_scene`, `recorder_set_mode`, `recorder_tick_tracks` | `recorder_tick_tracks` | - |
| +0x129 | timeline_enabled | H | `recorder_ctor`, `recorder_set_timeline_enabled` | `recorder_advance_clock`, `recorder_get_timeline_enabled`, `recorder_set_clock` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | vtable | H | `recorder_names_ctor` | virtual dispatch from `recorder_save_replay_file`, `recorder_load_replay_file` | - |
| +0x04 | count | H | `recorder_names_ctor`, `recorder_names_append`, `recorder_names_clear`, `recorder_names_copy_from_block`, `recorder_names_reset` | `recorder_names_find_id`, `recorder_names_find_name_by_id`, `recorder_names_get_count`, `recorder_names_copy_to_block` and the other row walkers | `recorder_names_get_count` reads it, a signed loop bound in `recorder_rebuild_object_table` |
| +0x08 | rows | H | `recorder_names_ctor`, `recorder_names_append`, `recorder_names_clear`, `recorder_names_copy_from_block`, `recorder_names_read_dta`, `recorder_names_read_stdio`, `recorder_names_reset` | the same walkers | - |
| +0x0c | next_id | H | `recorder_names_ctor`, `recorder_names_alloc_id`, `recorder_names_copy_from_block` | `recorder_names_alloc_id`, `recorder_names_copy_to_block`, `recorder_names_read_dta`, `recorder_names_read_stdio`, `recorder_names_write_stdio` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | key_arena | H | `recorder_bindings_init`, `recorder_bindings_release_key_arena`, `recorder_bindings_adopt_key_arena` | `recorder_bindings_has_key_arena`, `recorder_bindings_release_key_arena` | - |
| +0x04 | rows_alloc | H | `recorder_bindings_init` | - | - |
| +0x08 | rows_begin | H | `recorder_bindings_free_vectors`, `recorder_bindings_init` | `recorder_bindings_clear_all`, `recorder_bindings_compact_and_rebind`, `recorder_bindings_detach_object`, `recorder_bindings_find_by_name`, `recorder_bindings_find_by_name_pair`, `recorder_bindings_remove_for_object` | - |
| +0x0c | rows_end | H | `recorder_bindings_clear_all`, `recorder_bindings_compact_and_rebind`, `recorder_bindings_free_vectors`, `recorder_bindings_init`, `recorder_bindings_remove_for_object` | `recorder_bindings_bind_track_by_name`, `recorder_bindings_clear_all`, `recorder_bindings_compact_and_rebind`, `recorder_bindings_detach_object`, `recorder_bindings_find_by_name` | - |
| +0x10 | rows_cap | H | `recorder_bindings_free_vectors`, `recorder_bindings_init` | `recorder_add_track_binding`, `recorder_bindings_bind_track_by_name` | - |
| +0x14 | values_alloc | H | `recorder_bindings_init` | - | - |
| +0x18 | values_begin | H | `recorder_bindings_free_vectors`, `recorder_bindings_init` | `recorder_bindings_clear_values`, `recorder_bindings_load_file` | - |
| +0x1c | values_end | H | `recorder_bindings_clear_values`, `recorder_bindings_free_vectors`, `recorder_bindings_init` | `recorder_bindings_clear_values`, `recorder_bindings_load_file` | - |
| +0x20 | values_cap | H | `recorder_bindings_free_vectors`, `recorder_bindings_init` | `recorder_bindings_add_value` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | id | H | `recorder_names_append` | `recorder_names_find_id`, `recorder_names_find_name_by_id` | - |
| +0x04 | name | H | `recorder_names_append` | `recorder_names_find_id`, `recorder_names_find_name_by_id` | - |
| +0x34 | value | H | `recorder_names_set_value_by_id` | `recorder_names_get_value_by_id` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | cursor_current | H | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_track_rewind_cursors`, `recorder_track_step_keys_to_clock`, `recorder_key_stream_alloc`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `actor_track_apply_pose_at_clock`, `recorder_track_step_keys_to_clock`, `recorder_key_stream_alloc` | while recording, the record under the cursor |
| +0x04 | cursor_next | H | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_track_rewind_cursors`, `recorder_track_step_keys_to_clock`, `recorder_key_stream_alloc`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `actor_track_apply_pose_at_clock`, `recorder_track_step_keys_to_clock` | - |
| +0x08 | cursor_last_type2 | H | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_track_rewind_cursors`, `recorder_track_step_keys_to_clock`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_track_step_keys_to_clock` | - |
| +0x0c | unread_0c | U | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | - | no reader in the recorder methods (class census); the `*_track_*` overrides of `structs/actor_track.md` were not read for it |
| +0x10 | cursor_last_type3 | H | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_track_rewind_cursors`, `recorder_track_step_keys_to_clock`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_track_rewind_cursors` | the rewind copies it into `+0x08`; `recorder_track_step_keys_to_clock` writes it under the same key test as `+0x08`, so after a step the two hold the same key |
| +0x14 | last_key_time | H | `recorder_new_track`, `recorder_track_reset_fields`, `recorder_key_stream_alloc`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_track_step_keys_to_clock`, `human_track_dump_keys`, `recorder_key_stream_size` | sentinel `0xfffffd66` |
| +0x18 | first_chunk | H | `recorder_new_track`, `recorder_key_stream_new_chunk`, `recorder_key_stream_clear`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_key_stream_size`, `recorder_track_rewind_cursors`, `actor_track_key_offset_of` | - |
| +0x1c | current_chunk | H | `recorder_new_track`, `recorder_key_stream_alloc`, `recorder_track_reset_fields`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_key_stream_alloc` | - |
| +0x20 | name | H | `recorder_new_track`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_add_track_binding`, `recorder_save_replay_file` | - |
| +0x44 | key_size | H | `recorder_new_track`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_key_stream_alloc`, `recorder_key_stream_size`, `actor_track_key_offset_of` | - |
| +0x54 | end_reached | H | `recorder_track_step_keys_to_clock`, `recorder_track_rewind_cursors`, `recorder_load_replay_file`, `recorder_binding_block_read`, `recorder_bindings_load_file` | `recorder_track_step_keys_to_clock` | - |
| +0x58 | owner | H | `recorder_add_track` | `recorder_save_replay_file` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | mode | H | `recorder_bindings_bind_track_by_name` | `recorder_bindings_release_key_arena`, `recorder_bindings_clear_all` | - |
| +0x04 | owner_kind | H | `recorder_bindings_bind_track_by_name` | `recorder_bindings_attach_object`, `recorder_save_replay_file` | - |
| +0x08 | frame_name | H | `recorder_bindings_attach_object`, `recorder_bindings_detach_object` | `recorder_save_replay_file` | - |
| +0x2c | binding_name | H | `recorder_bindings_bind_track_by_name` | `recorder_bindings_find_by_name` | - |
| +0x50 | object | H | `recorder_bindings_attach_object`, `recorder_bindings_detach_object` | `recorder_bindings_clear_all`, `recorder_save_replay_file` | - |
| +0x54 | track | H | `recorder_bindings_bind_track_by_name` | `recorder_save_replay_file`, `recorder_bindings_clear_all` | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | next | H | `recorder_key_stream_new_chunk` | `recorder_key_stream_clear`, `recorder_key_stream_size`, `actor_track_key_offset_of` | - |
| +0x04 | soft_limit | H | `recorder_key_stream_new_chunk` | `recorder_key_stream_alloc` | - |
| +0x08 | records | H | `recorder_key_stream_alloc` | `recorder_track_rewind_cursors`, `recorder_key_stream_size` | - |


| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | anim_set | H | `recorder_load_anim_by_name` | `recorder_release_object_table`, the callers of `recorder_get_object_by_id` | - |
| +0x04 | duration_ms | H | `recorder_load_tck` | the `*_track_apply_*` callers of `recorder_get_object_by_id` | - |
| +0x08 | sample_period_ms | H | `recorder_load_tck` | the same | - |
| +0x0c | vec_a | H | `recorder_load_tck` | the same | - |
| +0x18 | vec_b | H | `recorder_load_tck` | the same | - |
| +0x24 | samples_a_alloc | H | `recorder_load_anim_by_name` | - | - |
| +0x28 | samples_a_begin | H | `recorder_load_anim_by_name`, `recorder_load_tck`, `recorder_tck_record_free_vectors`, `recorder_anim_object_free_vectors` | `recorder_tck_record_free_vectors`, `recorder_anim_object_free_vectors` | - |
| +0x2c | samples_a_end | H | `recorder_load_anim_by_name`, `recorder_load_tck` | `recorder_load_tck` | - |
| +0x30 | samples_a_cap | H | `recorder_load_anim_by_name`, `recorder_load_tck` | `recorder_load_tck` | - |
| +0x34 | samples_b_alloc | H | `recorder_load_anim_by_name` | - | - |
| +0x38 | samples_b_begin | H | `recorder_load_anim_by_name`, `recorder_load_tck`, `recorder_tck_record_free_vectors`, `recorder_anim_object_free_vectors` | `recorder_tck_record_free_vectors`, `recorder_anim_object_free_vectors` | - |
| +0x3c | samples_b_end | H | `recorder_load_anim_by_name`, `recorder_load_tck` | `recorder_load_tck` | - |
| +0x40 | samples_b_cap | H | `recorder_load_anim_by_name`, `recorder_load_tck` | `recorder_load_tck` | - |
| +0x44 | has_samples_b | H | `recorder_load_tck` | `human_track_apply_pose_at_clock`, `human_track_apply_pose_at_time`, `human_track_get_rot_at_time` | a byte test on the object `recorder_get_object_by_id` returns |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | valid | H | `actor_track_apply_pose_at_time`, `recorder_ctor` | - | 1 when the key word has bit `0x4000`; no reader (`.text` scan) |
| +0x04 | pos | H | `actor_track_apply_pose_at_time` | - | the frame's `+0x40..+0x48`; no reader (`.text` scan) |
| +0x10 | rot | H | `actor_track_apply_pose_at_time` | - | `S_quat::Make` of the frame's world matrix; no reader (`.text` scan) |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | the bracket search |
| +0x04 | interp_time_ms | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | - |
| +0x08 | flags | H | `recorder_load_replay_file` | `recorder_tick_tracks` | §1i |
| +0x0c | pos | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | - |
| +0x18 | tangent_out | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | - |
| +0x24 | tangent_in | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | taken from the next key |
| +0x30 | roll | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | - |
| +0x34 | fov | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair`, `recorder_tick_tracks` | the tick tests it for 666.0 |
| +0x38 | ease_in | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | - |
| +0x3c | ease_out | H | `recorder_load_replay_file` | `recorder_interp_key_pos_and_pair` | taken from the next key |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | the bracket search |
| +0x04 | interp_time_ms | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | - |
| +0x08 | flags | H | `recorder_load_replay_file` | `recorder_tick_tracks` | §1i |
| +0x0c | pos | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | - |
| +0x18 | tangent_out | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | - |
| +0x24 | tangent_in | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | taken from the next key |
| +0x30 | ease_in | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | - |
| +0x34 | ease_out | H | `recorder_load_replay_file` | `recorder_interp_key_pos` | taken from the next key |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_and_flags | H | `recorder_load_replay_file` | `recorder_tick_tracks` | §8 |
| +0x04 | fade_in_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | read with flag `0x80000000` |
| +0x08 | fade_out_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | read with flag `0x40000000` |
| +0x0c | colour | H | `recorder_load_replay_file` | `recorder_tick_tracks` | passed to `hud_set_screen_tint` |
| +0x10 | texture_name | H | `recorder_load_replay_file` | `recorder_tick_tracks` (by address, `hud_set_overlay_texture`) | with flag `0x20000000` |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | - |
| +0x04 | command | H | `recorder_load_replay_file` | `recorder_tick_tracks` (by address, `mission_global_program_run`) | - |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | time_ms | H | `recorder_load_replay_file` | `recorder_tick_tracks` | - |
| +0x04 | hide | H | `recorder_load_replay_file` | `recorder_tick_tracks` | - |
| +0x08 | frame_name | H | `recorder_load_replay_file` | `recorder_tick_tracks` (by address, the mission scene's frame lookup) | - |

## 3. Method family

PC-only by measurement (§10), so the tables have no PS2 columns. All 53 vtable slots are placed; §6
gives them in slot order. The unit also holds `curve_ease_in_out` (its callers are the interpolators
below and `actor_track_*`) and six STL template helpers.

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x005407f0 | `crt_static_init_guard_005407f0` | the translation unit | - | C++ static-initialization entry for one translation unit |
| 0x00540810 | `crt_atexit_register_00540810` | the translation unit | - | registers this translation unit's static destructor |
| 0x00540830 | `recorder_static_zero_globals` | the file-scope globals | - | zeroes the file-scope zero vector and its neighbours (§9) |
| 0x00541a30 | `recorder_track_snapshot_static_dtor` | the snapshot vector | - | crt_atexit body: frees the track snapshot vector |
| 0x00543a00 | `recorder_names_ctor` | `names` | - | `this = +0x80` constructs the name↔id table |
| 0x00545110 | `recorder_singleton_static_init` | the singleton | - | `.CRT$XCU` entry of the singleton |
| 0x00545120 | `recorder_singleton_construct` | the singleton | - | `mov ecx, 0x656368 ; jmp recorder_ctor` |
| 0x00545130 | `recorder_ctor` | instance | - | the defaults of §1; `recorder_names_ctor`, `recorder_bindings_init` |
| 0x00545320 | `recorder_bindings_free_vectors` | `bindings` | - | `this = +0x90` frees the binding table's two vectors |
| 0x00545350 | `recorder_singleton_register_atexit` | the singleton | - | registers `recorder_singleton_destruct` |
| 0x00545360 | `recorder_singleton_destruct` | the singleton | - | `mov ecx, 0x656368 ; jmp recorder_dtor` |
| 0x00545370 | `recorder_dtor` | instance | - | frees the four vectors and both tables' vectors |
| 0x00545460 | `recorder_bindings_init` | `bindings` | - | `this = +0x90` zero-initialises the binding table |

### 3b. Accessors

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00540fb0 | `recorder_get_mode` | - | slot 1; returns the recorder mode |
| 0x005411d0 | `recorder_get_source_clock_ms` | - | slot 3; returns the un-remapped source clock |
| 0x005411e0 | `recorder_get_playback_clock_ms` | - | slot 4; returns the timeline-remapped playback clock |
| 0x00544af0 | `recorder_rewind_timeline_cursor` | - | slot 23; resets the timeline segment cursor |
| 0x00544b00 | `recorder_set_timeline_enabled` | - | slot 27; sets the timeline-remap flag |
| 0x00544b10 | `recorder_get_timeline_enabled` | - | slot 28; reads the timeline-remap flag |

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00540920 | `recorder_interp_key_pos_and_pair` | - | Hermite-interpolates a position plus a trailing float pair between two keyframes |
| 0x00540ac0 | `recorder_interp_key_pos` | - | Hermite-interpolates a position between two keyframes |
| 0x00540c30 | `recorder_new_track` | - | slot 5; allocates a `0x5c`-byte track and names it by kind (§7a) |
| 0x00540db0 | `recorder_track_reset_fields` | - | slot 6; resets a track record to its constructed state |
| 0x00540de0 | `recorder_set_mode` | - | slot 0; switches the recorder between stopped, record and playback |
| 0x00540fc0 | `recorder_set_clock` | - | slot 2; set the recorder clock to an absolute time and prune finished tracks |
| 0x005411f0 | `recorder_add_track` | - | slot 8; the target of every `*_track_create` override: gives an object a track, registers it, adds its binding row |
| 0x005413f0 | `recorder_attach_existing_track` | - | slot 7; appends a track the caller built to the active vector, clears its cursor, sets flag `0x10`, drives the playback / record slots |
| 0x00541450 | `recorder_remove_track` | - | slot 9; unregisters a track |
| 0x00541520 | `recorder_release_all_key_streams` | - | slot 10; frees the key data of every registered track |
| 0x00541620 | `recorder_has_track` | - | slot 12; tests whether a track is registered |
| 0x00541680 | `recorder_set_track_active` | - | slot 13; moves a track between the active and idle vectors |
| 0x005417b0 | `recorder_has_track_in_either_list` | - | slot 14; tests whether a track is in either track vector |
| 0x00541810 | `recorder_snapshot_track_list` | - | slot 11; copies both track vectors into the file-scope snapshot vector |
| 0x00541a50 | `recorder_tick_tracks` | - | per-frame track stepper |
| 0x00542670 | `recorder_track_rewind_cursors` | - | slot 24; rewinds a client's track cursors to the first key, clears the end-reached flag |
| 0x005426d0 | `recorder_track_step_keys_to_clock` | - | advances one track's key cursors up to the playback clock |
| 0x00542790 | `recorder_track_stop_and_rewind` | - | stops a track and rewinds its key cursors |
| 0x005427b0 | `recorder_track_seek_to_clock` | - | seeks one track to the current playback clock |
| 0x005427f0 | `recorder_load_replay_file` | - | slot 25; loads a `.rep` replay file and starts playback |
| 0x00543040 | `recorder_save_replay_file` | - | slot 26; writes the recorder state out as a `.rep` file |
| 0x00543454 | `recorder_write_replay_blocks` | - | writes the replay's blocks to the open file |
| 0x00543540 | `recorder_advance_clock` | - | advances the recorder clock by one frame delta and remaps it through the timeline |
| 0x00543620 | `recorder_binding_block_size` | - | slot 40; returns the serialised size of the binding rows |
| 0x005436b0 | `recorder_binding_block_write` | - | slot 41; flattens the binding rows and their key streams into a block |
| 0x00543840 | `recorder_binding_block_read` | - | slot 42; rebuilds the binding rows and their key streams from a flat block |
| 0x00543980 | `recorder_bind_object_by_name` | - | slot 43; attaches the object registered under a name |
| 0x005439c0 | `recorder_unbind_object_by_name` | - | slot 44; detaches the object registered under a name |
| 0x00543a20 | `recorder_names_alloc_id` | - | `this = +0x80` allocates the next name id |
| 0x00543a30 | `recorder_names_append` | - | `this = +0x80` appends a {id, name} row to the name table |
| 0x00543b50 | `recorder_names_find_id` | - | `this = +0x80` resolves a name to its id |
| 0x00543bd0 | `recorder_names_find_name_by_id` | - | `this = +0x80` returns the name of the row with a given id |
| 0x00543c10 | `recorder_names_get_value_by_id` | - | `this = +0x80` reads the value stored on the row with a given id |
| 0x00543c50 | `recorder_names_set_value_by_id` | - | `this = +0x80` stores a value on the row with a given id |
| 0x00543c90 | `recorder_names_get_count` | - | `this = +0x80` returns the number of rows in the name table |
| 0x00543ca0 | `recorder_names_clear` | - | `this = +0x80` frees the name table's row array when it holds rows |
| 0x00543cd0 | `recorder_names_copy_from_block` | - | `this = +0x80` rebuilds the name table from a flat memory block |
| 0x00543d50 | `recorder_names_copy_to_block` | - | `this = +0x80` copies the name table into a flat memory block |
| 0x00543db0 | `recorder_names_read_stdio` | - | `this = +0x80` reads the name table through the stdio reader |
| 0x00543e50 | `recorder_names_write_stdio` | - | `this = +0x80` writes the name table through the stdio writer |
| 0x00543ec0 | `recorder_names_read_dta` | - | `this = +0x80` reads the name table from a `.dta` handle |
| 0x00543f60 | `recorder_names_reset` | - | `this = +0x80` frees the name table's row array unconditionally |
| 0x00543f80 | `recorder_names_serialized_size` | - | `this = +0x80` returns the serialised size of the name table |
| 0x00543fa0 | `recorder_get_or_create_name_id` | - | slot 15; resolves a name to an id, building the object when it is new |
| 0x00544030 | `recorder_load_anim_by_name` | - | slot 16; loads an animation set by name into a new record |
| 0x00544190 | `recorder_get_object_by_id` | - | slot 18; returns the owned object at an index |
| 0x005441c0 | `recorder_get_object_by_name` | - | slot 17; returns the owned object registered under a name |
| 0x00544200 | `recorder_get_name_for_id` | - | slot 19 and 35; forwards to the name table's name lookup |
| 0x00544210 | `recorder_load_tck` | - | slot 20; C_recorder — load a `.tck` keyframe-track file |
| 0x00544680 | `recorder_load_anim_for_object` | - | slot 21; loads the animation clip an object's track needs |
| 0x005447d0 | `recorder_tck_record_free_vectors` | - | `this = none` ECX is the `.tck` record at `S_anim_object + 0x04`; frees the two sample vectors of a `.tck` record (§1g); only reference the unwind funclet `Unwind_006369c0`; the receiver is inferred from the offsets |
| 0x00544800 | `recorder_rebuild_object_table` | - | slot 22; re-derives the owned-object vector from the name table |
| 0x005449f0 | `recorder_release_object_table` | - | destroys every object the recorder owns and empties the table |
| 0x00544a60 | `recorder_anim_object_free_vectors` | - | frees the same two vectors from the element (§1g); sole caller `recorder_release_object_table` |
| 0x00544a90 | `recorder_get_or_create_name_id_direct` | - | slot 34; looks a name up in the name table, appending it when absent |
| 0x00544ad0 | `recorder_get_name_value` | - | slot 36; forwards to the name table's value getter |
| 0x00544ae0 | `recorder_set_name_value` | - | slot 37; forwards to the name table's value setter |
| 0x00544b20 | `recorder_clear_binding_values` | - | slot 29; forwards to the binding table's value clear |
| 0x00544b30 | `recorder_add_binding_value` | - | slot 30; forwards to the binding table's value append |
| 0x00544b40 | `recorder_load_bindings_file` | - | slot 31; forwards to the binding table's file loader |
| 0x00544b50 | `recorder_detach_object_binding` | - | slot 32; forwards to the binding table's detach |
| 0x00544b60 | `recorder_attach_object_binding` | - | slot 33; attaches an object to its binding row and notifies the recorder |
| 0x00544b90 | `recorder_compact_bindings` | - | slot 38; forwards to the binding table's compact-and-rebind |
| 0x00544ba0 | `recorder_end_sequence_and_restore_scene` | - | slot 39; ends a sequence and puts the scene back the way it was |
| 0x00544eb0 | `recorder_replay_state_write` | - | flattens the recorder's three replay arrays into a block |
| 0x00544f80 | `recorder_replay_state_read` | - | restores the recorder's three replay arrays from a flat block |
| 0x005450e0 | `recorder_replay_state_size` | - | returns the byte size of the recorder's replay state block |
| 0x005454e0 | `recorder_bindings_has_key_arena` | - | `this = +0x90` `cmp dword ptr [ecx], 0 ; setne al ; ret` — three instructions |
| 0x005454f0 | `recorder_bindings_adopt_key_arena` | - | `this = +0x90` adopts a key-data arena when the binding table has none |
| 0x00545520 | `recorder_bindings_find_by_name` | - | `this = +0x90` finds a binding row by name |
| 0x005455d0 | `recorder_bindings_find_by_name_pair` | - | `this = +0x90` finds a binding row by name, qualified by a second name |
| 0x005456e0 | `recorder_bindings_remove_for_object` | - | `this = +0x90` removes the binding row owned by one object |
| 0x005457b0 | `recorder_bindings_clear_all` | - | `this = +0x90` releases every binding row, then the shared key arena |
| 0x00545840 | `recorder_bindings_release_key_arena` | - | `this = +0x90` frees the binding table's key arena when no row uses it |
| 0x005458b0 | `recorder_bindings_compact_and_rebind` | - | `this = +0x90` drops bound rows and re-binds every recorder client |
| 0x005459c0 | `recorder_bindings_clear_values` | - | `this = +0x90` empties the binding value list |
| 0x005459f0 | `recorder_bindings_add_value` | - | `this = +0x90` appends a {name, value} row to the binding value list |
| 0x00545c30 | `recorder_bindings_bind_track_by_name` | - | `this = +0x90` upserts the row of a binding name; mode 0 and a new track first clear the recorded keys |
| 0x00545f50 | `recorder_add_track_binding` | - | `this = +0x90` C_recorder — record how a track binds to a scene frame |
| 0x005461c0 | `recorder_bindings_load_file` | - | `this = +0x90` loads a binding file into the binding table |
| 0x00546430 | `recorder_bindings_detach_object` | - | `this = +0x90` detaches an object from every binding row that names it |
| 0x005464a0 | `recorder_bindings_attach_object` | - | `this = +0x90` attaches a live object to its named binding row |
| 0x00546510 | `recorder_open_speech_sound_for_text_id` | - | slot 45; resolves and opens the speech wav for a textdb string id |
| 0x005465f0 | `recorder_get_constant_5` | - | slot 46; `return 5`; its only reference is the slot, and no recorder call dispatches `+0xb8` |
| 0x00546600 | `recorder_set_overlay_texture` | - | slot 47; sets the cutscene overlay/letterbox texture |
| 0x00546630 | `recorder_rebind_speaker_channel` | - | slot 48; re-points a speech channel at a new model and restarts the line |
| 0x00546660 | `recorder_tick_speech` | - | slot 49; forwards the frame delta to the speech tick |
| 0x00546670 | `recorder_open_synced_stream` | - | slot 50; opens the streamed audio track that follows the recorder clock |
| 0x005466d0 | `recorder_set_stream_time_base` | - | slot 51; sets the time base the synced stream is measured against |
| 0x005466e0 | `recorder_sync_stream_to_clock` | - | slot 52; keeps the synced audio stream aligned with the playback clock |
| 0x00546790 | `recorder_stop` | - | stops the running sequence |
| 0x005467c0 | `recorder_is_sequence_running` | - | reports whether a recorded sequence is currently running |
| 0x00548890 | `recorder_key_stream_clear` | - | C_recorder — drop a track's key data |
| 0x005488a0 | `recorder_key_stream_size` | - | C_recorder — total byte size of a track's key stream |
| 0x005489b0 | `recorder_key_stream_new_chunk` | - | `__stdcall`; allocates one `0x1800`-byte chunk (§7b) |
| 0x005489e0 | `recorder_key_stream_alloc` | - | `__stdcall`; the bump allocator of §7b; eleven callsites, all `*_track_record_key` |

### 3d. Free helpers

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x00540850 | `curve_ease_in_out` | - | evaluates a quadratic ease-in / linear / ease-out curve |
| 0x005467e0 | `stl_vector_ctor_with_allocator` | - | constructs an empty vector, copying the allocator |
| 0x00546840 | `stl_copy_stride_0x58` | - | copies a range of 0x58-byte elements forward |
| 0x00546880 | `stl_fill_n_stride_0x58` | - | fills n 0x58-byte elements with one value |
| 0x005468b0 | `stl_copy_stride_0x28` | - | copies a range of 0x28-byte elements forward |
| 0x005468f0 | `stl_fill_n_stride_0x28` | - | fills n 0x28-byte elements with one value |
| 0x00546920 | `stl_copy_stride_0x24_00546920` | - | copies a range of 0x24-byte elements forward |
| 0x00546960 | `stl_construct_stride_0x58` | - | copy-constructs one 0x58-byte element |
| 0x00546980 | `stl_construct_stride_0x28` | - | copy-constructs one 0x28-byte element |

## 4. The singleton

**`0x00656368`**, proven two ways:

* `recorder_singleton_construct` is `mov ecx, 0x656368 ; jmp recorder_ctor`, and
  `recorder_singleton_destruct` is `mov ecx, 0x656368 ; jmp recorder_dtor`.
* A whole-`.text` disassembly scan finds **59 `mov ecx, 0x656368` sites**: a singleton loaded as an
  immediate gives the receiver *and* the full method census in one scan. The heaviest consumers are
  `engine_game_loop` (8), `program_process_command` (7), `game_done`, `mission_close` and
  `actor_dtor`; the rest are the `C_actor_track` slot implementations and their per-class overrides.

**The singleton has no symbol in the decomp.** An immediate loaded straight into ECX produces no
identifier in Ghidra's output: the callsite renders as `recorder_add_track(obj, 0x5c, 0, 0)` with the
receiver invisible.

The object is polymorphic (`recorder_add_track` dispatches through `[this]`, slots 5 and 12), but no
derived recorder exists: the vtable has exactly one reference in the whole file, the store in
`recorder_ctor`, and `recorder_ctor`'s only referrer is the singleton's `jmp`.

**The two client vectors.** `recorder_set_track_active` moves a client between them: mode 1 takes it
out of the idle vector and appends it to the active vector, clearing the client's flag `0x10`; mode 0
does the reverse and sets the flag. `recorder_has_track_in_either_list` scans both. Each entry's
`+0x08` is its track, `+0x10` its kind. `recorder_bindings_compact_and_rebind` is the only function that
reaches them by absolute address (§9); everywhere else they are `this`-relative.

## 5. What names the class

`recorder_load_tck` emits all of these through `dbgPrintf`:

```
C_recorder:LoadTCK(%s)          C_recorder:tck opened...
C_recorder:version = 4          C_recorder:wrong version
C_recorder:LoadTCK() done ok    C_recorder:LoadTCK() done failed
```

Shipping `.tck` files include `Ddeath.tck`, `DturnL.tck`, `DturnR.tck`, `Nuda01.tck`, `Nuda02.tck`,
`turn1.tck`. The format is **version 4**; anything else is rejected.

### 5a. The `.tck` layout

`recorder_load_tck` reads the file front to back into its caller's record:

| Offset | Size | Field | Evidence |
|---|---|---|---|
| `+0x00` | 4 | version, must be 4 | the gate of `recorder_load_tck` |
| `+0x04` | 12 | a vec3 | stored at `out+0x08` |
| `+0x10` | 12 | a vec3 | stored at `out+0x14` |
| `+0x1c` | 4 | the duration, ms | stored at `out+0x00` |
| `+0x20` | 4 | the sample period, ms (20 in 244 files) | stored at `out+0x04` |
| `+0x24` | 4 | N, the first array's count | - |
| `+0x28` | 12 × N | the first array, three floats per sample | appended through the vector end `out+0x28` (begin `out+0x24`) |
| next | 4 | M, the second array's count | `out+0x40` = 1 when M is non-zero |
| next | 12 × M | the second array | appended through the vector end `out+0x38` (begin `out+0x34`) |

In every shipped version-4 file N = duration / period + 1, and M is 0 or N. Of the 293 shipped `.tck`
files, 283 are version 4: 224 end exactly after the second array, and 59 carry further whole 12-byte
records the loader never reads. The other 10 (five of version 2, five that open with a duration-like
value) print `C_recorder:wrong version` and load nothing.

## 6. The vtable

Slot indices are relative to the vtable at `0x0063d0e8`. A plain run-of-code-pointers scan of `.rdata`
merges it with the adjacent six-slot `0x0063d0d0` table (no separator between them), which reports
every slot 6 too high. §3 names each slot on its function row; slots 19 and 35 hold one function
(`recorder_get_name_for_id`), and slots 29 to 33, 36 to 38 are forwarders (`add ecx, 0x90` or `0x80`)
into the two embedded tables. Slot 34 is `lea esi, [ecx+0x80]`.

### 6a. Owner proof for the four one-line accessors

Identical-COMDAT folding is active in this link, so a one-line accessor needs its own owner proof. All
four keep the `recorder_` prefix:

| Accessor | Body | What proves the owner |
|---|---|---|
| `recorder_get_mode` | `mov eax,[ecx+0xe4]` | 0 callsites, 1 dataref: `g_recorder_vt` slot 1 |
| `recorder_get_source_clock_ms` | `mov eax,[ecx+0x11c]` | 0 callsites, 1 dataref: slot 3 |
| `recorder_get_playback_clock_ms` | `mov eax,[ecx+0x120]` | 0 callsites, 1 dataref: slot 4 |
| `recorder_names_get_count` | `mov eax,[ecx+4]` | 2 callsites, receiver `lea ecx,[ebx+0x80]` = the name table |

**A dataref count of exactly 1 is a single-owner proof.** The function appears in one class's vtable
and nowhere else, so nothing can have been folded into it, and it is the only proof available for the
first three, which nothing calls directly.

Two of these bodies are byte-identical to accessors of other classes and are still not folds:
`recorder_get_mode` has the same three bytes as `game_get_player_actor` (`0x00425530`), and
`recorder_names_get_count` the same as `std_vector_begin` (`0x005a17c0`). Two addresses, two owners:
surviving duplicates from the non-COMDAT population that could not fold.

### 6b. `recorder_set_clock` takes the time as a parameter

`recorder_set_clock` (slot 2) opens with `folded_return_this()`, whose body is `mov eax, ecx; ret`. Its
ECX at that callsite is `mov ecx, [esp+0x14]`, and the frame arithmetic settles what that is:

```
00540fc0  sub esp, 8          ; locals
00540fc3  push ebp            ; -> retaddr at [esp+0xc], arg1 at [esp+0x10]
00540fc4  push esi            ; -> retaddr at [esp+0x10], arg1 at [esp+0x14]
00540fc5  mov esi, ecx        ; esi = this (C_recorder)
00540fc7  mov ecx, [esp+0x14] ; <- the first STACK ARGUMENT
00540fcc  call folded_return_this
00540fd7  mov [esi+0x11c], eax
```

So the clock value is an **incoming parameter Ghidra dropped from the signature**. Nothing reads a
system timer.

The sibling settles what the function is: `recorder_advance_clock` does `+0x11c += *delta`; this one
does `+0x11c = param`. Both then run the identical remap of `source_clock_ms` through `timeline_keys`
into `playback_clock_ms` / `clock_rate` and stop every track whose slot-3 test fails at that time. The
pair differs only by **absolute versus delta**. Nothing dispatches the slot: it has no direct callsite,
its only reference is the vtable entry, and each of the 59 `mov ecx, 0x656368` immediates of §4 is
followed by a direct call to another member.

### 6c. Binding-table teardown

| Function | Range | What it does |
|---|---|---|
| `recorder_bindings_clear_all` | `[0x005457b0, 0x00545838)`, no `ret` | walks the rows: a row with mode 0 has its key stream cleared, and a bound scene object is unlinked; then empties the row vector and ends in `jmp 0x00545840` |
| `recorder_bindings_release_key_arena` | `[0x00545840, 0x005458a6)`, one `ret` | when `key_arena` is set, counts the rows with mode 1; with none, frees the arena and clears the pointer |

Both take the table in ECX. The closing `jmp` is the release's only reference, so it has no direct
caller of its own.

## 7. The key stream

### 7a. Track naming, and what it proves

`recorder_new_track` switches on the owner's **kind** and post-increments one counter per kind:

| Kind | Prefix | Counter global |
|---|---|---|
| 1 (actor) | `act_%d` | `g_recorder_track_seq_actor` |
| 2 / 3 / 0x1b (player, human, entity) | `hum_%d` | `g_recorder_track_seq_human` |
| 4 (car) | `car_%d` | `g_recorder_track_seq_car` |
| 9 (box) | `box_%d` | `g_recorder_track_seq_box` |
| 0x11 (drop-out) | `drp_%d` | `g_recorder_track_seq_dropout` |
| anything else | `unk_%d` | `g_recorder_track_seq_unknown` |

These are exactly the documented `C_entity` kind values, which confirms that the value
`recorder_add_track` forwards is the object's `+0x10` kind field. The counters are **sequence numbers**,
not populations: nothing decrements them.

### 7b. A chain of chunks, with no separate allocator object

`recorder_key_stream_alloc` and `recorder_key_stream_new_chunk` are `__stdcall` free functions (`ret 8`
and `ret 4`, so the receiver is pushed, not in ECX), and the object they are handed is the track.
Byte-proven at the call site:

```
car_track_record_key:   mov  eax, [esi + 8]     ; esi = this (C_car); [this+8] = the track
                        push 2                  ; the key type
                        push eax                ; the track
                        call recorder_key_stream_alloc
```

All **eleven** `recorder_key_stream_alloc` call sites are the four `*_track_record_key` overrides
(`human_` x7, `car_` x2, `box_` x1, `dropout_` x1), and every one passes `actor+0x08`. That is the same
object `recorder_key_stream_clear` and `recorder_key_stream_size` receive: their call sites load
`mov ecx, [row + 0x54]`, the track field of the binding row.

The allocator chains as soon as `cursor + size(new key) >= chunk + 0xff0`, and the overflow sentinel
lands below that limit, so nothing ever writes `[chunk + 0xff4, chunk + 0x1800)`: about 2 KB of each
chunk stays unused. Why the allocation is `0x1800` is not recoverable from the binary.

**Why only two functions.** Allocation needs exactly bump + grow; `recorder_key_stream_new_chunk` has
two call sites and both are inside its parent. The rest of the interface lives on the track:
`recorder_key_stream_clear` (free), `recorder_key_stream_size` / `actor_track_key_stream_size` (walk),
`actor_track_key_offset_of` (index) and slot 10 `recorder_release_all_key_streams` (bulk free).

**The alloc advances past the record under the cursor, not the one being asked for.** It reads that
record's type out of `record+0x04`, looks the stride up in `key_size`, and only then checks whether the
new type fits. On overflow it stamps **`0xEEEEEEEE`** at the old cursor: the continuation sentinel
`recorder_key_stream_size` and `actor_track_key_offset_of` walk, and the reason `key_size[0]` is the
8-byte chunk-link record (`structs/actor_track.md` §6).

## 8. The playback tick

**`mode`** (`recorder_set_mode`, `recorder_tick_tracks`): 0 stop · 1 play: every client's slot 7
(`actor_track_set_playback`) on, flag `0x10` set · 2 record: the active clients' slot 6
(`actor_track_set_recording`) on, the idle clients play · 3 play: only the tick has a branch for it, no
`recorder_set_mode` arm · 4 idle, the ctor's value.

`recorder_tick_tracks` returns at once in mode 4. Otherwise it works in this order, on two clocks: the
camera, target, fade, script and frame-event keys (§1i-§1m) are compared with `source_clock_ms`; the
clients and the speech, subtitle and morph keys with `playback_clock_ms`; slot 52 receives
`source_clock_ms`:

| Keys | Stride | What the tick does |
|---|---|---|
| `camera_keys` + `target_keys` | `0x40` + `0x38` | modes 1 and 3: interpolates position and look target (`recorder_interp_key_pos_and_pair`, `recorder_interp_key_pos`), and poses the mission camera: position, direction + roll (key `[0xc]`), field of view (key `[0xd]`). A key whose `[0xd]` is 666.0 ends the sequence (`slot 0`, mode 0) |
| `fade_keys` | `0x20` | low 24 bits of `[0]` = the time, high bits = flags (`0x80000000` fade in, `0x40000000` fade out, `0x20000000` a texture name at `+0x10`, `0x10000000` hold, `0x8000000` unmodulated); `hud_set_overlay_texture`, `hud_set_screen_tint` with the colour `[3]` |
| `script_keys` | `0x28` | `mission_global_program_run(key + 4)` when the time passes |
| `frame_events` | `0x28` | finds the frame named at `key + 8`, `key + 4` = 0 shows it, 1 hides it |
| the clients | - | per `mode` the slots `+0x18` / `+0x1c` / `+0x20` and `recorder_track_step_keys_to_clock` / `_seek_to_clock` / `_stop_and_rewind` |
| `speech_keys`, `subtitle_keys`, `morph_keys` | `0x24`, `0xc`, `0x2c` | `speech_apply_recorder_key`, `hud_subtitles_add_from_record`, `speech_morph_frame_list_add`, one per tick |

**`timeline_keys`** are `{source_ms, target_ms, rate}`: `recorder_advance_clock` advances the cursor up
to `source_clock_ms`, sets `playback_clock_ms` = `target_ms` + the scaled rest and `clock_rate` = `rate`,
and rescales the delta it hands back. With `timeline_enabled` clear or no keys, the two clocks are equal.

**`file_block_0`** doubles as the *keys are owned by the file* flag: `recorder_release_all_key_streams`
frees the clients' streams only when it is not 1, the ctor's value.

## 9. Globals

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| - | `0x00656368` | the `C_recorder` singleton (no symbol, §4) | `recorder_ctor` | §4 |
| `g_recorder_idle_clients_begin` / `_end`, `g_recorder_active_clients_begin` / `_end` | `0x00656370`..`0x00656384` | the singleton's own `idle_begin` / `_end` and `active_begin` / `_end` | `recorder_ctor`, `recorder_dtor`, `recorder_remove_track`, `recorder_set_track_active` | `recorder_bindings_compact_and_rebind` |
| `g_recorder_playback_clock_ms` | `0x00656488` | the singleton's `playback_clock_ms` | `recorder_advance_clock`, `recorder_set_clock`, `recorder_load_replay_file` | every track apply |
| `g_zero_vector_00656494` | `0x00656494` | a file-scope zero `S_vector` | `recorder_static_zero_globals` | - |
| `g_recorder_track_seq_*` | `0x006564c4`..`0x006564d8` | the six per-kind name counters (§7a) | `recorder_new_track` | `recorder_new_track` |

`g_recorder_playback_clock_ms` has no initial bytes in `Game.exe`. `.data` is `VA 0x00646000, virtual
size 0xb4e7c, raw size 0x10000`, so the section's file bytes stop at VA `0x00656000`; `0x00656488` lies
in the **BSS tail**, zero-initialised at load. File offset `0x256488` belongs to the next section's raw
data (`.rsrc`, raw `0x256000`), so bytes read there say nothing about this global.

## 10. PC vs PS2

**`C_recorder` does not exist on PS2.** No symbol carries the class, and none of `lastKeyTime`,
`animflags`, `animid`, `.dump` or any `C_recorder:` string appears anywhere in `SLUS_206.71`.
`__vt__7C_actor` on PS2 holds only `C_actor`'s own 27 methods, with no client interface before them.

The port kept the **data and the readers** and cut the **producer**: `.tck` files still ship, and PS2
retains `LoadTCK__5C_dogFv`, `GetPlacementFromTCK__5C_car`, `GetPlacementFromTCK1__5C_car`,
`GetPlacementFromTCK__28C_animations_machine_de_luxe`, `GetEndPointOfTCK__…` and
`GetMixingPointOfTCK__…`.

Every function and global **of `C_recorder` itself** is PC-only by measurement, not by search failure.
The claim covers only those symbols, not the address range: the 16-channel speech bank follows this unit
and **does** exist on PS2, as free functions in `g_dabing.cpp`, three of them byte-identical
(`structs/speech.md` §4). Here the PC build has a subsystem the PS2 port removed: the reverse of the
usual direction, where the PS2 build adds subsystems.

## 11. Related

`structs/actor_track.md` (the client interface, the override table, the key sizes) ·
`structs/speech.md` (the three bridge slots) · `structs/hud.md` (the tint and overlay the fade keys
drive) · `structs/chunk.md`.

## 12. Open items

1. **The speech, subtitle and morph keys.** Of `speech_keys` (`0x24` bytes), `subtitle_keys` (`0xc`) and
   `morph_keys` (`0x2c`) only `+0x00`, the time, is decoded; the rest is read by
   `speech_apply_recorder_key`, `hud_subtitles_add_from_record` and `speech_morph_frame_list_add`, which
   were not read for it.
