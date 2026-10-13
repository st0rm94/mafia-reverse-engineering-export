# `soundArena` — the game-wide sound propagation ring

**Created** 2026-07-16 20:47 +0200 · **Modified** 2026-10-11 10:46 +0200

⚠️ **Not to be confused with `C_sound_table`** (`structs/sound_table.md`), which sits in the same
`tools.cpp` translation unit immediately above this one and shares the word "sound" and nothing else.
That class is the `tables\sounds.dat` bank — a registry of named sound DEFINITIONS. This one is a
per-mission propagation RING that carries emitted sounds to AI listeners. The two never touch.

`soundArena` is the **producer half of the hearing sense**: one ring of live sounds owned by the
`C_game` object. Anything that makes a noise emits into it; every listening NPC is walked against it
once per AI tick, and audible sounds become **sound contacts** on that NPC's `ai_sensors`
(`structs/ai_sensors.md`).

| | |
|---|---|
| PC prefix | **`sound_arena_*`** |
| PC location | **`C_game + 0x2b3c`**, address accessor `game_get_sound_arena` (`0x00425550`) ¹ |
| Owner | `C_game`, not per-entity |
| Method coverage | 4 PC functions, 4/4 PS2 methods mapped (§3) |

¹ Reached as `*(*(0x0065115c) + 0x24) + 0x2b3c` — byte-proven from the disassembly at the
`sound_arena_update_entity_sound_contacts` callsites (`mov eax,[0x65115c]; mov ecx,[eax+0x24];
add ecx,0x2b3c`).

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Field layout (PC offsets, relative to the `soundArena` base)

```c
typedef struct soundArena {              // at C_game+0x2b3c
    /* +0x000 */ sound_slot slots[16];   // [H] THE LIVE-SOUND RING - 16 slots, stride 0x20.
                                         //     sound_arena_init zeroes exactly 0x80 dwords = 0x200
                                         //     bytes from the base; sound_arena_emit_sound and
                                         //     sound_arena_invalidate_actor walk the same 16.
    /* +0x200 */ uint32_t cursor;        // [H] the write cursor: a new sound takes slot
                                         //     (cursor + 1) & 0xf
    /* +0x204 */ uint32_t next_gc_at;    // [H] throttle stamp: the next game-clock time at which
                                         //     sound_arena_emit_sound will run its GC sweep.
} soundArena;                            // 0x208 bytes

typedef struct sound_slot {              // 0x20 bytes
    /* +0x00 */ float    pos[3];         // [H] where the sound was emitted
    /* +0x0c */ uint32_t type;           // [H] 1 = a step, 2 = gunfire (§4); 0 = free slot
    /* +0x10 */ uint32_t time;           // [H] emission time on the game clock
    /* +0x14 */ float    intensity;      // [H] the intensity at the source
    /* +0x18 */ float    range;          // [H] the distance at which the intensity reaches 0
    /* +0x1c */ C_actor *source;         // [H] the emitting actor
} sound_slot;
```

**The per-slot record is read from the disassembly of all three slot walkers.**
`sound_arena_emit_sound` writes all six fields, `sound_arena_update_entity_sound_contacts` reads
all six, and `sound_arena_invalidate_actor` matches `+0x1c` and clears `+0x0c` and `+0x1c`; the GC
sweep clears the same two on a slot older than 2000 ms.

**The clock** is `C_game + 0x2b0c` — the same clock `ai_sensors_can_hear` and
`ai_sensors_build_smell_contact_from_blob` read.

## 2. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x000 | slots[16] | H | `sound_arena_init` (zeroes 0x200 B), `sound_arena_emit_sound` | `sound_arena_update_entity_sound_contacts`, `sound_arena_invalidate_actor` | 16 × 0x20 |
| slot +0x00 | pos | H | `sound_arena_emit_sound` | `sound_arena_update_entity_sound_contacts` | the origin of the distance attenuation |
| slot +0x0c | type | H | `sound_arena_emit_sound`; cleared by the GC and by `sound_arena_invalidate_actor` | `sound_arena_update_entity_sound_contacts` (0 = skip), `sound_arena_emit_sound` (the find key) | 0 = free slot |
| slot +0x10 | time | H | `sound_arena_emit_sound` | `sound_arena_emit_sound` (GC) | expiry compared against the game clock, deadline **2000 ms** |
| slot +0x14 | intensity | H | `sound_arena_emit_sound` (its third argument) | `sound_arena_update_entity_sound_contacts` | the intensity at the source |
| slot +0x18 | range | H | `sound_arena_emit_sound` (its second argument) | `sound_arena_update_entity_sound_contacts` | the attenuation divisor |
| slot +0x1c | source | H | `sound_arena_emit_sound`; cleared by the GC and by `sound_arena_invalidate_actor` | `sound_arena_update_entity_sound_contacts` (skips the listener's own sounds), `sound_arena_emit_sound` and `sound_arena_invalidate_actor` (the key) | the emitting `C_actor*`; zeroed when that actor dies |
| +0x200 | cursor | H | `sound_arena_init` (0), `sound_arena_emit_sound` | `sound_arena_emit_sound` | the write cursor, advanced mod 16 |
| +0x204 | next_gc_at | H | `sound_arena_init` (0), `sound_arena_emit_sound` (clock + **2000**) | `sound_arena_emit_sound` | GC throttle — the sweep runs at most once per 2000 ms |

## 3. Method family — PC ↔ PS2 (4/4 CONFIRMED)

Every row body-diffed on both builds.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061d820 | `sound_arena_emit_sound` | `SoundEmit__10soundArenaFR8S_vectorffUiP7C_actor` | 0x003b2390 | CONFIRMED | **The producer.** Throttled GC, then find-or-add the slot keyed by (type, source) ¹ |
| 0x0061d970 | `sound_arena_update_entity_sound_contacts` | `GetSounds__10soundArenaFP8C_entity` | 0x003b2550 | CONFIRMED | **The listener side.** For one entity: attenuate each live sound by distance and register or cancel a sound contact ² |
| 0x0061dac0 | `sound_arena_init` | `Init__10soundArenaFv` | 0x003b2780 | CONFIRMED | Zeroes the ring (0x80 dwords), then clears `+0x200` and `+0x204` ³ |
| 0x0061dae0 | `sound_arena_invalidate_actor` | `InvalidateActor__10soundArenaFP7C_actor` | 0x003b27c0 | CONFIRMED | Zeroes the source + live/type fields of every slot whose source (`+0x10`) matches ⁴ |

¹ Clears slots older than **2000 ms**, restamps `+0x204` to clock + **2000**. Twin evidence: the
constant 2000 used twice in identical roles, 16 slots, 6 parameters on both builds.
² `intensity − dist × intensity / range`; audible iff it exceeds
`1.05 − entity_get_hearing_sensitivity()`, then clamped to 1.0 and handed to
`ai_sensors_new_sound_contact` as the record `{time, type, pos, source, intensity}`
(`structs/ai_sensors.md`); otherwise `ai_sensors_cancel_sound_contact(source, type)`. Twin evidence: the constant 1.05,
the identical attenuation formula, and 19 calls from the entity AI tick on both builds.
³ `+0x204` is the GC stamp `sound_arena_emit_sound` reads, which pins this as the arena's init.
⁴ A dead actor's sounds cannot outlive it. Mirrors `entity_release_target` at the entity level.

## 4. The hearing chain end-to-end

```
  <something makes a noise>
        |
        v
  sound_arena_emit_sound(pos, range, intensity, type, source)     [C_game+0x2b3c]
        |   GC slots older than 2000 ms (throttled by +0x204), then find-or-add by (type, source)
        v
  entity_ai  --(x19, once per listening entity)-->  sound_arena_update_entity_sound_contacts(entity)
        |   heard = intensity - (dist*intensity)/range
        |   audible iff heard > 1.05 - entity_get_hearing_sensitivity()
        |                                   ^-- the NPC's ai_character HEARING stat (+0xebc)
        +-- audible ---> ai_sensors_new_sound_contact      -> writes C_entity+0xc74 (type 2 = GUNFIRE)
        +-- inaudible -> ai_sensors_cancel_sound_contact   -> zeroes the contact's type
```

**Gunfire is heard when it is loud enough *relative to that NPC's hearing stat* — not at a fixed
radius.** This chain is the writer of the `+0xc74` heard-event ring (`structs/entity.md` §5,
`structs/ai_sensors.md` §3).

### The emitters

The emitter set is closed: a byte scan of `.text` for `E8`/`E9` branches to `0x0061d820` finds two
`E8` calls and no tail jump, and a scan of `.rdata`/`.data` for the address as a pointer finds none.

| callsite | caller | type | range | intensity | source |
|---|---|---|---|---|---|
| `0x0048d852` | `human_notify_callback` | 1 | step value × 10.0 | 0.5 | the walking human ¹ |
| `0x005e479e` | `game_new_shoot` | 2 | 50.0 | 1.0 | the shooter |

¹ Emitted right after `human_notify_callback` plays the surface-material sound
(`g_surface_material_table`) for an animation notify, next to its `r_foot` frame path — consistent
with footsteps.

The argument order is read from the push sequence at both callsites and from the consumer, which
divides by the slot's `+0x18` and scales by its `+0x14`: gunfire reaches 50 units at full
intensity, a step `step value × 10` units at half.

Type 2 = GUNFIRE agrees with `entity_has_heard_gunfire` and with the `ai_sensors_new_sound_contact`
event dispatch (type 1 ⇒ entity event 6, type 2 ⇒ entity event 5). `game_new_shoot` also calls
`pedestrian_manager_new_panic_at_position` right after the emit.

## 5. PC vs PS2

- **PS2 location:** `*(act_mission + 0x20) + 0xe80`; the clock is `*(act_mission+0x20) + 0xe54`.
- **PS2 class tag:** `10soundArena`, 4 methods, all mapped (§3).

## 6. Cross-references

- `structs/ai_sensors.md` §3 — the consumer side; the full producer→contact chain.
- `structs/ai_character.md` — the HEARING stat that gates audibility.
- `structs/entity.md` §5 — the `+0xc74` heard-event ring this feeds.
- `structs/game.md` — the owning `C_game` object (clock `+0x2b0c`, `pursuit_event_manager` `+0x2b18`).
- `structs/pursuit-event-manager.md` — the other known `C_game+0x2b__` sub-object.
