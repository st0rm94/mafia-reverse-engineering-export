# `ai_blob_log` / `ai_blob` — the scent trail

**Created** 2026-07-16 14:59 +0200 · **Modified** 2026-11-01 09:28 +0100

An `ai_blob_log` is a **scent trail**: a 16-slot ring of `ai_blob` records, each a position the owning human occupied,
dropped at most once every 500 ms. It is the trail producer of the **smell** sense that `ai_sensors` scores beside sight
and hearing (`structs/ai_sensors.md`), and a pursuing character follows it through `entity_trace_human_scent`.

```
  ai_sensors (sight / sound / SMELL)          structs/ai_sensors.md
        ^
  entity_trace_human_scent  (the follower)    reads the trail, LOS-tests each blob
        ^
  ai_blob_log  (one per C_human, at +0x758)   THIS FILE - the trail producer
```

| | |
|---|---|
| PC prefix | `ai_blob_log_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x244`, embedded at `C_human + 0x758` |
| PC constructor | - |
| PC translation unit | `[0x0048b5b0, 0x004ae800)`, the `C_human` unit (`structs/human.md` §4); this class is `[0x0048ee60, 0x0048f330)` inside it, next to `ai_loc` |
| PS2 class | `ai_blob_log` (tag `11ai_blob_log`), 3 methods; `ai_blob` (tag `7ai_blob`), 2 methods |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

**Why embedded at `C_human + 0x758`.** Every callsite with a readable receiver loads `lea ecx, [reg + 0x758]`:
`ai_blob_log_clear` from `human_game_init`, `human_activate` and `engine_new_mission`, `ai_blob_log_get_blob` from
`ai_sensors_cancel_visual_contact`, `entity_process_command` and `entity_spawn_police_response`. The neighbour `ai_loc` is
`0x34` bytes at `+0x724`, and `0x724 + 0x34 = 0x758` (`structs/ai_loc.md`).

```c
typedef struct ai_blob_log {        // 0x244 bytes, embedded at C_human+0x758
    /* +0x000 */ ai_blob  ring[16]; // [H] the trail: 16 slots x 0x24 = 0x240, oldest overwritten first
    /* +0x240 */ uint32_t cursor;   // [H] index of the newest blob; 0xffffffff = never written; masked with & 0xf at each use
} ai_blob_log;
```

**Why 16.** `ai_blob_log_add_blob` masks the cursor with `& 0xf`, `ai_blob_log_clear` zeroes exactly `0x90` dwords before
writing the cursor sentinel, and `entity_trace_human_scent` walks `n < 0x10`. Each entry is one emitted position; the
cursor is never masked in storage, so `ai_blob_log_get_blob` can tell that a trail is shorter than the index asked for.

### 1a. `ai_blob` — one scent blob (`0x24` bytes)

```c
typedef struct ai_blob {
    /* +0x00 */ S_vector  pos;        // [H] the owner frame's origin with y + 1.0
    /* +0x0c */ S_vector  dir;        // [H] normalised: the delta from the previous blob, or the frame forward for a first blob
    /* +0x18 */ uint32_t  emitted_ms; // [H] game clock at emission, C_game+0x2b0c; drives the 500 ms rate limit
    /* +0x1c */ uint32_t  flags;      // [H] the owner's state at emission (§2b)
    /* +0x20 */ C_human  *owner;      // [H] the emitting human; the previous slot's value decides first blob or delta
} ai_blob;
```

`dir` is a movement direction, except for the first blob of a trail, where no delta exists and the frame's forward vector
stands in. `ai_group_send_sniff_event` also builds these records outside any ring (§4).

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | ring | H | `ai_blob_log_add_blob`, `ai_blob_log_clear` | `ai_blob_log_add_blob`, `ai_blob_log_get_blob` | the census sees only slot 0; the bodies index `+ (i & 0xf) * 0x24` |
| +0x240 | cursor | H | `ai_blob_log_add_blob`, `ai_blob_log_clear` | `ai_blob_log_add_blob`, `ai_blob_log_get_blob` | - |

### 2a. `ai_blob`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | pos | H | `ai_blob_log_add_blob`, `ai_group_send_sniff_event` | `ai_blob_log_add_blob`, `ai_blob_log_get_blob`, `ai_sensors_get_smelled_target_info`, `entity_trace_human_scent` | `ai_blob_log_get_blob` copies the whole record |
| +0xc | dir | H | `ai_blob_log_add_blob`, `ai_group_send_sniff_event` | `ai_blob_log_get_blob`, `ai_sensors_get_smelled_target_info` | - |
| +0x18 | emitted_ms | H | `ai_blob_log_add_blob`, `ai_group_send_sniff_event` | `ai_blob_log_add_blob`, `ai_blob_log_get_blob`, `ai_sensors_get_smelled_target_info` | - |
| +0x1c | flags | H | `ai_blob_log_add_blob`, `ai_group_send_sniff_event` | `ai_blob_log_get_blob`, `ai_sensors_get_smelled_target_info` | `ai_group_send_sniff_event` writes 0; no reader uses the value (§2b) |
| +0x20 | owner | H | `ai_blob_log_add_blob`, `ai_group_send_sniff_event` | `ai_blob_log_add_blob`, `ai_blob_log_get_blob` | - |

### 2b. `flags` — the blob flag word

Built at emission from the owner's state, and read by nothing that uses it: `ai_sensors_get_smelled_target_info` is the
only reader interface, its 9 callers (`entity_action_tick` ×6, `ai_logs_reset_stale_logs`, `ai_action_new_hostile_coverage`,
`police_board_update_sighting`) pass 0 for that out-parameter, and `entity_trace_human_scent` uses the record as a position.

| Value | Meaning | Evidence |
|---|---|---|
| `0x02` | the owner's byte `+0x1e4` is non-zero, the crouched flag | `ai_blob_log_add_blob`, `structs/human.md` |
| `0x04` | the owner's byte `+0x1ec` is non-zero, the shooting latch | `ai_blob_log_add_blob`, `structs/human.md` |
| `0x01` | the owner's animation id `+0x70 & 0x1ff` is 5 or `0x60` | the `switch` at the end of `ai_blob_log_add_blob` |
| `0x08` | the same id is `0x5e`, `0x5f` or `0x60` | the same `switch` |

## 3. Method family

3 of 3 `ai_blob_log` methods mapped; `ai_blob`'s 2 have no PC code (§5).

### 3a. Construction & destruction

No row: the log has no constructor of its own; `ai_blob_log_clear` is its initialiser.

### 3b. Accessors

No row.

### 3c. Behaviour

#### `ai_blob_log`

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0048ee60 | `ai_blob_log_add_blob` | `Blob__11ai_blob_logFPv` | 0x001653c0 | CONFIRMED | The trail producer: one blob per 500 ms, called only by `human_update` |
| 0x0048f2c0 | `ai_blob_log_get_blob` | `GetBlob__11ai_blob_logFUi` | 0x00165830 | CONFIRMED | Heap-clones the blob `cursor - n` (caller frees); 0 for an empty ring or `n > cursor` |
| 0x0048f310 | `ai_blob_log_clear` | `Clear__11ai_blob_logFv` | 0x001658c0 | CONFIRMED | Wipes the ring and sets the cursor to `0xffffffff`; called by `human_game_init`, `human_activate`, `engine_new_mission`, and by a tail jump from `human_reset_pose_and_scent_trail` |

### 3d. Free helpers

No row: the class calls no helper of the unit.

## 4. How the scent chain fits together

There are **two producers** of scent records:

| Producer | Trigger | Destination |
|---|---|---|
| `ai_blob_log_add_blob` | passive, one blob per human per 500 ms | that human's own 16-slot ring, a trail left on the ground |
| `ai_group_send_sniff_event` | on demand, one blob per call | one entity's smell-contact table, through entity event `0xb` |

`ai_group_send_sniff_event` (`structs/ai_group.md` §5) never touches an `ai_blob_log`. It allocates a bare `ai_blob` and hands
it to `entity_dispatch_event(entity, 0, 0xb, blob)`, whose case `0xb` wraps it in a `0x1c`-byte entry and upserts it through
`ai_sensors_new_smell_contact`. `ai_fight_manager_tick` builds the same record for a car party's members
(`structs/ai_fight_manager.md` §4).

**Who lays a trail.** `ai_blob_log_add_blob` has one caller: a scan of `.text` finds one `E8` call (`0x004930df`), no tail jump and
no pointer to it. The caller is `human_update` (`0x00492ff0`), immediately before `ai_loc_update` on its main per-frame path.
That path is not reached while the `+0xa58` countdown is positive or while `C_human+0x04 & 5` is non-zero (that branch runs
`human_ai` and returns). `+0x04 & 5` is the recorded-track playback mask (`structs/actor_track.md` §2a), and nothing ever sets
`+0xa58` positive (`structs/human.md`), so **every human lays a trail except while a recorded track drives its pose**.

| Step | Function | What it does |
|---|---|---|
| produce (trail) | `ai_blob_log_add_blob` | one blob per human per 500 ms, at frame origin y + 1.0 |
| produce (direct) | `ai_group_send_sniff_event` | one blob on demand, posted to one entity as event `0xb` |
| reset | `ai_blob_log_clear` | a re-activated character never inherits an old trail |
| read | `ai_blob_log_get_blob` | heap-clones the Nth-newest blob |
| follow | `entity_trace_human_scent` | walks up to 16 blobs, tests a line from the actor to each blob on collision layer `0x100000`, and returns the first with a clear line |
| score | `ai_sensors` smell band | the sense that consumes the result; smell reuses the sight stat (`structs/ai_sensors.md`) |

**The reading:** a character following a scent walks to the most recent trail point it can see, not to where the target is.

`C_dog` is not a smell consumer: it derives from `C_actor`, so it has no `ai_sensors` (`structs/dog.md` §4).

## 5. PC vs PS2

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x001653a0 | `__nw__7ai_blobFUi` | a one-line tail call to `Alloc__11ai_spagemanFv`, the PS2-only pool allocator; the PC calls `operator_new` |
| 0x001653b0 | `__dl__7ai_blobFPv` | its pair; the PC calls `operator_delete` |

- **Blob stride: PC `0x24`, PS2 `0x30`.** The record holds two vectors, and a PS2 `S_vector` is 4 floats against the PC's 3, which is
  `+0x08`; the PS2 record then ends 4 bytes later than the PC's after `owner`.
- **Ring size: PC `0x240`, PS2 `0x300`**, both 16 × the stride. **Object: PC `0x244`, PS2 `0x304`**, both ring + cursor.
- **Field offsets:** `pos` `+0x00` on both; `dir` PC `+0x0c`, PS2 `+0x10`; `emitted_ms` PC `+0x18`, PS2 `+0x20`; `flags` PC `+0x1c`,
  PS2 `+0x24`; `owner` PC `+0x20`, PS2 `+0x28`.
- **Owner offset:** PC `C_human+0x758`, PS2 `+0x7c0` (delta `0x68`). The adjacent `ai_loc` has delta `0x5c`, so this band does not
  rebase by a constant.
- **Clock:** PC `*(g_act_mission+0x24) + 0x2b0c`, PS2 `*(act_mission+0x20) + 0xe54`.
- **The 500 ms rate limit, the y + 1.0 lift and the `dir` rule are identical**, including the normalise sign cascade.
- **`ai_blob_log_clear`:** the PC inlines the fill, the PS2 calls libc `memset`; that is the whole of the
  52 / 32 = 1.63 size ratio. The two constants are the same quantity per build.
- **`ai_blob_log_add_blob`:** 1128 / 1120 = 1.007, with the same flag word from the same two owner bytes.
- **`ai_blob_log_get_blob`** is confirmed by two caller diffs (`TraceHuman`, `SpawnPolice`).
- **The animation-id arms (§2b) are not comparable across builds**: PC `5`, `0x5e`, `0x5f`, `0x60` against PS2 `0x3d`, `0x3e`…; the
  anim tables are rebased per build.
- **`ai_group_send_sniff_event`** allocates the blob with `__nw__7ai_blob(0x30)` on the PS2, against PC `operator_new(0x24)`.
- An `ai_*` class whose only unmapped PS2 members are `__nw__` / `__dl__` is complete on the PC; `ai_path` is the same case.

## 6. Related

- `structs/ai_sensors.md` — the three-sense system that consumes smell.
- `structs/ai_group.md` §5 — `ai_group_send_sniff_event`, the direct scent producer.
- `structs/ai_loc.md` — the adjacent sub-object at `C_human+0x724`, same unit.
- `structs/human.md` — the owner class.
- `structs/dog.md` §4 — why `C_dog` is not a scent consumer.

## 7. Open items

None.
