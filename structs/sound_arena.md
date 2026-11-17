# `soundArena` — the game-wide sound propagation ring

**Created** 2026-07-16 20:47 +0200 · **Modified** 2026-11-08 22:21 +0100

The **producer half of the hearing sense**: one ring of live sounds embedded in the `C_game` object
at `+0x2b3c`. Anything that makes a noise emits into it; every listening NPC is walked against it
once per AI tick, and audible sounds become **sound contacts** on that NPC's `ai_sensors`
(`structs/ai_sensors.md`). Not to be confused with `C_sound_table` (`structs/sound_table.md`), the
`tables\sounds.dat` bank in the same translation unit, which shares the word "sound" and nothing
else.

| | |
|---|---|
| PC prefix | `sound_arena_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x208` (`sound_arena_init` writes `0x80` dwords and the two words after them) |
| PC constructor | - |
| PC translation unit | `[0x0061b050, 0x0061df00)`, the `tools.cpp` unit hosted by `structs/sound_table.md`; the four `sound_arena_*` functions are its last class run, before the `C_sound_table` container helpers |
| PS2 class | `soundArena` (tag `10soundArena`), 4 methods, all mapped (§3) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct soundArena {              // 0x208 bytes; at C_game + 0x2b3c
    /* +0x000 */ sound_slot slots[16];   // [H] the live-sound ring, stride 0x20
    /* +0x200 */ uint32_t   cursor;      // [H] the slot of the newest emission, mod 16
    /* +0x204 */ uint32_t   next_gc_at;  // [H] game-clock time of the next expiry sweep
} soundArena;
```

**Why sixteen slots.** The ring is a static array: `sound_arena_init` zeroes exactly `0x80` dwords
(16 × `0x20` bytes), `sound_arena_emit_sound` advances the cursor as `(cursor + 1) & 0xf`, and every
walker (`emit`, `update_entity_sound_contacts`, `invalidate_actor`) loops sixteen times. Each entry
is one live sound. A new sound that matches no slot's `(type, source)` always takes slot
`(cursor + 1) & 0xf`, free or not: a free slot has type 0 and never matches, so the ring is
overwritten in order rather than filled into its gaps.

The arena has no constructor: `C_game` holds it as raw storage and `sound_arena_init` clears it.
The game clock it compares against is `C_game + 0x2b0c`, the same clock `ai_sensors_can_hear` and
`ai_sensors_build_smell_contact_from_blob` read. The object is reached as
`*(*(0x0065115c) + 0x24) + 0x2b3c` (`mov eax,[0x65115c]; mov ecx,[eax+0x24]; add ecx,0x2b3c` at the
`sound_arena_update_entity_sound_contacts` callsites) or through `game_get_sound_arena`
(`structs/game.md`).

### 1a. `sound_slot` — one live sound

```c
typedef struct sound_slot {              // 0x20 bytes
    /* +0x00 */ float    pos[3];         // [H] where the sound was emitted (x, y, z)
    /* +0x0c */ uint32_t type;           // [H] 1 = a step, 2 = gunfire (§4); 0 = free slot
    /* +0x10 */ uint32_t time;           // [H] emission time on the game clock
    /* +0x14 */ float    intensity;      // [H] the intensity at the source
    /* +0x18 */ float    range;          // [H] the distance at which the intensity reaches 0
    /* +0x1c */ C_actor *source;         // [H] the emitting actor
} sound_slot;
```

A slot is keyed by `(type, source)`: a repeated emission from the same actor and type refreshes
`pos`, `time`, `intensity` and `range` in place instead of taking a new slot.

## 2. Field evidence

Measured from body reads of the four `sound_arena_*` functions and of the callers of each; no
other function addresses the arena's band.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | slots | H | `sound_arena_init` (zero), `sound_arena_emit_sound`, `sound_arena_invalidate_actor` | `sound_arena_emit_sound`, `sound_arena_update_entity_sound_contacts`, `sound_arena_invalidate_actor` | §2a |
| +0x200 | cursor | H | `sound_arena_init` (0), `sound_arena_emit_sound` | `sound_arena_emit_sound` | advanced only when no live slot matches |
| +0x204 | next_gc_at | H | `sound_arena_init` (0), `sound_arena_emit_sound` (clock + 2000) | `sound_arena_emit_sound` | the sweep runs at most once per 2000 ms |

### 2a. `sound_slot`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | pos | H | `sound_arena_emit_sound` | `sound_arena_update_entity_sound_contacts` | the origin of the distance attenuation |
| +0x0c | type | H | `sound_arena_emit_sound`; cleared by the sweep and by `sound_arena_invalidate_actor` | `sound_arena_emit_sound` (the key), `sound_arena_update_entity_sound_contacts` (0 = skip) | 0 = free slot |
| +0x10 | time | H | `sound_arena_emit_sound` | `sound_arena_emit_sound` (the sweep), `sound_arena_update_entity_sound_contacts` (copied into the contact) | a slot older than **2000 ms** is freed |
| +0x14 | intensity | H | `sound_arena_emit_sound` (its third argument) | `sound_arena_update_entity_sound_contacts` | - |
| +0x18 | range | H | `sound_arena_emit_sound` (its second argument) | `sound_arena_update_entity_sound_contacts` | the attenuation divisor |
| +0x1c | source | H | `sound_arena_emit_sound`; cleared by the sweep and by `sound_arena_invalidate_actor` | `sound_arena_emit_sound` and `sound_arena_invalidate_actor` (the key), `sound_arena_update_entity_sound_contacts` (skips the listener's own sounds) | zeroed when that actor is invalidated |

## 3. Method family

Coverage: all **4** PS2 `10soundArena` methods are CONFIRMED twins, every row body-diffed on both
builds.

### 3a. Construction & destruction

The class has no constructor or destructor: `sound_arena_init` (§3c) clears the storage that
`C_game` provides.

### 3b. Accessors

The class has no accessor; the address accessor `game_get_sound_arena` is a `C_game` method
(`structs/game.md`).

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061d820 | `sound_arena_emit_sound` | `SoundEmit__10soundArenaFR8S_vectorffUiP7C_actor` | 0x003b2390 | CONFIRMED | **the producer**: the throttled expiry sweep, then find-or-add the slot keyed by `(type, source)` ¹ |
| 0x0061d970 | `sound_arena_update_entity_sound_contacts` | `GetSounds__10soundArenaFP8C_entity` | 0x003b2550 | CONFIRMED | **the listener side**: for one entity, attenuate each live sound by distance and register or cancel a sound contact ² |
| 0x0061dac0 | `sound_arena_init` | `Init__10soundArenaFv` | 0x003b2780 | CONFIRMED | zeroes the ring (`0x80` dwords), then `cursor` and `next_gc_at`; called by `game_init`, `race_winners_replay_end` and a `program_process_command` arm |
| 0x0061dae0 | `sound_arena_invalidate_actor` | `InvalidateActor__10soundArenaFP7C_actor` | 0x003b27c0 | CONFIRMED | clears `type` and `source` of every slot whose `source` (`+0x1c`) is the actor; called by `game_invalidate_actor` ³ |

¹ The sweep frees slots older than **2000 ms** and restamps `+0x204` to clock + **2000**. Twin
evidence: the constant 2000 used twice in identical roles, 16 slots, the same argument list on both
builds.

² `heard = intensity − dist × intensity / range`; audible iff it exceeds
`1.05 − entity_get_hearing_sensitivity()`, then clamped to 1.0 and handed to
`ai_sensors_new_sound_contact` as the record `{time, type, pos, source, heard}`
(`structs/ai_sensors.md`); otherwise `ai_sensors_cancel_sound_contact(source, type)`. Twin evidence:
the constant 1.05 and the identical attenuation formula. Its callers are `entity_ai` (19 callsites)
and `entity_action_tick` (1).

³ A dead actor's sounds cannot outlive it; `entity_release_target` is the entity-level counterpart.

### 3d. Free helpers

The class has no free helper; the `tools.cpp` unit's helpers are listed in `structs/sound_table.md`.

## 4. The hearing chain end-to-end

```
  <something makes a noise>
        |
        v
  sound_arena_emit_sound(pos, range, intensity, type, source)     [C_game+0x2b3c]
        |   expire slots older than 2000 ms (throttled by +0x204), then find-or-add by (type, source)
        v
  entity_ai  --(x19, once per listening entity)-->  sound_arena_update_entity_sound_contacts(entity)
        |   heard = intensity - (dist*intensity)/range
        |   audible iff heard > 1.05 - entity_get_hearing_sensitivity()
        |                                   ^-- the NPC's ai_character HEARING stat (+0xebc)
        +-- audible ---> ai_sensors_new_sound_contact      -> writes C_entity+0xc74 (type 2 = GUNFIRE)
        +-- inaudible -> ai_sensors_cancel_sound_contact   -> zeroes the contact's type
```

**Gunfire is heard when it is loud enough relative to that NPC's hearing stat, not at a set radius.**
This chain is the writer of the `+0xc74` heard-event ring (`structs/entity.md` §2,
`structs/ai_sensors.md` §5).

### 4a. The emitters

The emitter set is closed: a byte scan of `.text` for `E8` / `E9` branches to `0x0061d820` finds two
`E8` calls and no tail jump, and a scan of `.rdata` / `.data` for the address as a pointer finds
none.

| Callsite | Caller | Type | Range | Intensity | Source |
|---|---|---|---|---|---|
| `0x0048d852` | `human_notify_callback` | 1 | step value × 10.0 | 0.5 | the walking human ¹ |
| `0x005e479e` | `game_new_shoot` | 2 | 50.0 | 1.0 | the shooter |

¹ Emitted right after `human_notify_callback` plays the surface-material sound
(`g_surface_material_table`) for an animation notify, next to its `r_foot` frame path: footsteps.

The argument order is read from the push sequence at both callsites and from the consumer, which
divides by the slot's `+0x18` and scales by its `+0x14`: gunfire reaches 50 units at full intensity,
a step `step value × 10` units at half.

Type 2 = gunfire agrees with `entity_has_heard_gunfire` and with the `ai_sensors_new_sound_contact`
event dispatch (type 1 ⇒ entity event 6, type 2 ⇒ entity event 5). `game_new_shoot` also calls
`pedestrian_manager_new_panic_at_position` right after the emit.

## 5. PC vs PS2

- **PS2 location:** `*(act_mission + 0x20) + 0xe80`; the clock is `*(act_mission + 0x20) + 0xe54`.
- **PS2 class tag:** `10soundArena`, 4 methods, all mapped (§3).

## 6. Related

- `structs/ai_sensors.md` §5 — the consumer side; the full producer-to-contact chain.
- `structs/ai_character.md` — the HEARING stat that gates audibility.
- `structs/entity.md` §2 — the `+0xc74` heard-event ring this feeds.
- `structs/game.md` — the owning `C_game` object (clock `+0x2b0c`, `pursuit_event_manager` `+0x2b18`).
- `structs/pursuit_event.md` — the `C_game` member just below this one.
- `structs/sound_table.md` — the host of the `tools.cpp` unit.

## 7. Open items

1. **In game:** an NPC with a low hearing stat should ignore gunfire at a distance where a keen NPC
   reacts (§4); the two thresholds have not been compared in play.
