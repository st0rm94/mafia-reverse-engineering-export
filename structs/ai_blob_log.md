# `ai_blob_log` / `ai_blob` — the scent trail

**Created** 2026-07-16 14:59 +0200 · **Modified** 2026-09-29 09:31 +0200

An `ai_blob_log` is a **scent trail**: a 16-slot ring of `ai_blob` records, each one a position the
owning human occupied, dropped at most once every 500 ms. It is the *trail* producer of Mafia's
**smell** sense — the sense `ai_sensors` scores alongside sight and hearing
(`structs/ai_sensors.md`), and the one a pursuing character follows via
`entity_trace_human_scent`.

```
  ai_sensors (sight / sound / SMELL)          structs/ai_sensors.md
        ^
  entity_trace_human_scent  (the follower)    reads the trail, LOS-tests each blob
        ^
  ai_blob_log  (one per C_human, @ +0x758)    THIS FILE - the trail producer
```

| | |
|---|---|
| PC prefix | **`ai_blob_log_*`** |
| Owner | embedded in **`C_human` at `+0x758`** ¹ |
| PC cluster | `[0x0048ee60, 0x0048f330)`, one translation unit shared with `ai_loc` |
| Method coverage | `ai_blob_log` 3/3 mapped; `ai_blob`'s 2 PS2 methods have no PC code (§4) |

¹ `lea ecx,[reg+0x758]` at 1/1 `ai_blob_log_add_blob`, 3/3 `ai_blob_log_clear` and 3/4
`ai_blob_log_get_blob` callsites, found by walking back from each call to the ECX write (the fourth
is a `mov ecx, ebx` — an address already computed, which that walk cannot follow). The neighbour
`ai_loc` is `0x34` bytes at `+0x724`, and `0x724 + 0x34 = 0x758` exactly (`structs/ai_loc.md`).

**Confidence tags:** **[H]** body-read on both builds · **[M]** inferred · **[U]** not observed.

---

## 1. Object layout (PC offsets, `0x244` bytes)

```c
typedef struct ai_blob_log {                 // 0x244 bytes; embedded in C_human at +0x758
    /* +0x000 */ ai_blob  ring[16];          // [H] 16 slots x 0x24 = 0x240 - see §1a
    /* +0x240 */ uint32_t cursor;            // [H] index of the NEWEST blob; 0xffffffff = the
                                             //     ring has never been written. Not masked in
                                             //     storage - it increments without bound and is
                                             //     masked with & 0xf at every use
} ai_blob_log;                               // 0x244
```

The slot count is proven twice over: `& 0xf` in `ai_blob_log_add_blob`, and
`ai_blob_log_clear` zeroing exactly `0x240` bytes before writing the cursor sentinel.

### 1a. `ai_blob` — one scent blob (`0x24` bytes)

```c
typedef struct ai_blob {                     // 0x24 bytes (PC)
    /* +0x00 */ float     pos[3];            // [H] the owner frame's origin with **y + 1.0** -
                                             //     blobs float one unit above the ground
    /* +0x0c */ float     dir[3];            // [H] normalised: the owner frame's forward vector
                                             //     for the first blob of a trail, otherwise the
                                             //     normalised delta from the previous blob, i.e.
                                             //     the direction the character was MOVING
    /* +0x18 */ uint32_t  emitted_ms;        // [H] game clock at emission - drives the 500 ms
                                             //     rate limit and any staleness test
    /* +0x1c */ uint32_t  flags;             // [H] see §1b
    /* +0x20 */ C_human  *owner;             // [H] the emitting human; ALSO the slot-occupied
                                             //     test - add_blob reads the PREVIOUS slot's
                                             //     owner to decide first-blob vs delta
} ai_blob;                                   // 0x24
```

**`dir` is a movement direction, not a facing direction**, except for the first blob of a trail
where no delta exists yet and the frame's forward vector stands in.

The clock is `C_game + 0x2b0c` (`*(g_act_mission+0x24) + 0x2b0c`).

### 1b. The blob flag word at `+0x1c`

Built at emission from the owner's state, and **never read**. The only reader interface is
`ai_sensors_get_smelled_target_info`'s flags out-parameter, and all 9 of its callers (9 `E8`, no tail
jump) pass `0` for it; `entity_trace_human_scent` uses the record as a position only.

| Bit | Written when |
|---|---|
| `0x02` | the owner's byte at `+0x1e4` is non-zero — the **crouched** flag per `structs/human.md` [H] |
| `0x04` | the owner's byte at `+0x1ec` is non-zero — the **shooting** latch per `structs/human.md` [H] |
| `0x01` / `0x08` | selected by a `switch` on the owner's animation id (`owner+0x70 & 0x1ff`) [H] ¹ |

¹ PC cases `5`, `0x5e`/`0x5f`, `0x60`…

---

## 2. Method family

| PC VA | PC name | PS2 twin | Status | Role / decisive evidence |
|---|---|---|---|---|
| 0x0048ee60 | `ai_blob_log_add_blob` | `Blob__11ai_blob_logFPv` | CONFIRMED | **The trail producer** ¹ |
| 0x0048f2c0 | `ai_blob_log_get_blob` | `GetBlob__11ai_blob_logFUi` | CONFIRMED | Heap-clones the Nth-newest blob ² |
| 0x0048f310 | `ai_blob_log_clear` | `Clear__11ai_blob_logFv` | CONFIRMED | Wipes the ring and resets the cursor to `0xffffffff` ³ |
| - | - | `__nw__7ai_blobFUi` | PS2-ONLY | §4 |
| - | - | `__dl__7ai_blobFPv` | PS2-ONLY | §4 |

Every mapped row is body-diffed on both builds.

¹ Shared 500 ms rate limit, the same `0xffffffff` empty-ring sentinel, the same `& 0xf` mask, the
same y + 1.0 lift, the same first-blob-vs-delta branch with an identical normalise sign cascade,
the same flag word from the same two owner bytes. Size ratio 1128/1120 = **1.007**.
² Confirmed by two independent body-diffs of its callers (`TraceHuman`, `SpawnPolice`). It is an
`ai_blob_log` method; no PC function belongs to `ai_blob`.
³ The two constants are the same quantity per build (§4). Size ratio 52/32 = **1.63**.

---

## 3. How the scent chain fits together

There are **two producers** of scent records, and they are architecturally different:

| producer | trigger | destination |
|---|---|---|
| `ai_blob_log_add_blob` | **passive**, one blob per human per 500 ms | that human's own 16-slot ring — a TRAIL left on the ground |
| **`ai_group_send_sniff_event`** | **on demand**, one blob per call | **one specific entity's smell-contact table**, via entity event `0xb` |

`ai_group_send_sniff_event` (`structs/ai_group.md` §3) never touches an `ai_blob_log`. It
allocates a bare `ai_blob` (`operator_new(0x24)`) and hands it straight to
`entity_dispatch_event(entity, 0, 0xb, blob)`. Case `0xb` wraps it in a 0x1c-byte entry and upserts
it through **`ai_sensors_new_smell_contact`**. One lays a trail that any pursuer may later find;
the other *tells one specific character* about a scent, so a group sharing a lead with a member
does not need the member to walk over the ground where the trail is. A record type can therefore
have producers outside the class that owns the container.

**Who lays a trail.** `ai_blob_log_add_blob` has exactly one caller: a byte scan of `.text` finds
one `E8` call (`0x004930df`), no tail jump and no pointer to it in `.rdata`/`.data`. The caller is
`human_update` (`0x00492ff0`), with `lea ecx,[esi+0x758]`, immediately before `ai_loc_update` on
its main per-frame path. That path is not reached when the `+0xa58` freeze countdown is still
positive or when `C_human+0x04 & 5` is non-zero (that branch runs `human_ai` and returns). `+0x04 & 5`
is the recorded-track playback mask (`structs/actor_track.md` §3a), and nothing ever sets `+0xa58`
positive (`structs/human.md`), so **every human lays a trail except while a recorded track drives its
pose**.

| Step | Function | What it does |
|---|---|---|
| produce (trail) | **`ai_blob_log_add_blob`** | one blob per human per 500 ms, at frame origin `y+1.0`, from `human_update` |
| produce (direct) | **`ai_group_send_sniff_event`** | one blob on demand, posted to one entity as event `0xb` → `ai_sensors_new_smell_contact` |
| reset | **`ai_blob_log_clear`** | called beside `ai_loc_init` from `human_game_init` / `human_activate`, so a re-activated character never inherits an old trail |
| read | `ai_blob_log_get_blob` | heap-clones the Nth-newest blob (caller frees) |
| follow | `entity_trace_human_scent` | walks up to 16 blobs, raycasts actor→blob on collision layer `0x100000`, returns the first with clear LOS |
| score | `ai_sensors` smell band | the sense that consumes the result; **smell reuses the SIGHT stat** (`structs/ai_sensors.md`) |

**The gameplay reading:** a character following a scent does not walk to where the target *is* —
it walks to the most recent trail point it can actually *see*. That is why the trail stores a
movement direction per blob and why the follower LOS-tests each one.

`C_dog` is **not** a smell consumer: it derives from `C_actor`, so it has no `ai_sensors` at all
(`structs/dog.md` §2).

---

## 4. PC vs PS2

- **Blob stride: PC `0x24` / PS2 `0x30`.** The record holds **three** vectors (`pos`, `dir`, and
  PS2's padding of each), and a PS2 `S_vector` is 4 floats against PC's 3 — so the delta is
  exactly 3 × 4 = 12 bytes. That arithmetic working out is an independent check on the layout.
- **Ring size: PC `0x240` / PS2 `0x300`**, both = 16 × the stride. **Object: PC `0x244` / PS2
  `0x304`**, both = ring + a 4-byte cursor.
- **Field offsets** (`pos` `+0x00` on both; PC `dir` `+0x0c` ↔ PS2 `+0x10`; PC `emitted_ms`
  `+0x18` ↔ PS2 `+0x20`; PC `flags` `+0x1c` ↔ PS2 `+0x24`; PC `owner` `+0x20` ↔ PS2 `+0x28`).
- **Owner offset: PC `C_human+0x758` ↔ PS2 `+0x7c0`** (delta `0x68`). The adjacent `ai_loc` has
  delta `0x5c` — **the two deltas differ**, so this band does not rebase by a constant; use each
  class's own measured pair.
- **Clock: PC `*(g_act_mission+0x24) + 0x2b0c` ↔ PS2 `*(act_mission+0x20) + 0xe54`.**
- **The 500 ms rate limit, the y+1.0 lift and the `dir` rule are identical**, including the
  normalise sign cascade for the degenerate cases — the shipped scent behaviour is the same on both
  platforms.
- **`ai_blob_log_clear`: PC inlines `memset`** (MSVC compiles it as a `/Oi` intrinsic) where PS2
  calls libc `memset`. That is the whole of the 1.63 ratio.
- **The animation-id switch arms (§1b) are NOT comparable across builds**: PC cases `5`,
  `0x5e`/`0x5f`, `0x60`… against PS2 `0x3d`, `0x3e`…. The anim tables are rebased per build, so this
  is not a behavioural difference and the arms are not usable as matching evidence.
- **`ai_group_send_sniff_event`** allocates the blob with `__nw__7ai_blob(0x30)` on PS2, against PC
  `operator_new(0x24)` — the same two strides.
- **`ai_blob`'s two methods are PS2-only.** `__nw__7ai_blobFUi` is a one-line tail call to
  **`Alloc__11ai_spagemanFv`** — the custom pool allocator the PS2 port added for the console's
  32 MB of RAM; its only caller is that PS2-only subsystem. `ai_path`'s class-specific
  `operator new` / `operator delete` are the same case: an `ai_*` class whose only unmapped PS2
  members are `__nw__`/`__dl__` is complete on PC.

---

## 5. Cross-references

- `structs/ai_sensors.md` — the three-sense system that consumes smell; smell reuses the sight stat.
- `structs/ai_group.md` §3 — `ai_group_send_sniff_event`, the direct scent producer.
- `structs/ai_loc.md` — the adjacent sub-object at `C_human+0x724`, same translation unit.
- `structs/human.md` — the owner class and its sub-object offsets.
- `structs/dog.md` §2 — why `C_dog` is not a scent consumer.

