# The speech / lip-sync subsystem (`speech_`)

**Created** 2026-08-19 18:06 +0200 · **Modified** 2026-11-16 16:16 +0100

This is the system that makes a character **say a line and move their mouth**. The Mafia Script Data
Reference describes it from the script side:

> `human_talk xx, yyyy, zz`: *"Character xx speaks line yyyy. The script continues while speaking, so it
> usually has to wait. … but the text must also have a sound with the same name yyyy"*
> `human_stoptalk xx`: *"Character xx stops speaking."*

That one paragraph names the whole thing: a **textdb string id** is also a **sound file name**, and the
script has to poll a *still speaking?* predicate because the call returns immediately.

| | |
|---|---|
| PC prefix | `speech_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0x44` per channel; the bank `g_speech_channels` @ `0x006564f0` holds 16 |
| PC constructor | - |
| PC translation unit | `[0x005469a0, 0x00548830)`, guard to guard: 20 starts, all named |
| PS2 class | free functions of `g_dabing.cpp` (`__sinit_g_dabing.cpp` in the source-file manifest), 13 functions |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

`g_speech_channels[i]`, `0 ≤ i < 16`. The Ghidra symbols on element 0's fields
(`g_speech_channel_time_ms`, `_state`, `_sound`, …) are **aliases used as array bases**, not separate
objects: every access is `alias[i * 0x11]`.

```c
typedef struct S_speech_channel {             // 0x44 bytes, stride of the bank
    /* +0x00 */ uint32_t *viseme_table;       // [H] the file of §5, loaded per line
    /* +0x04 */ uint32_t time_ms;             // [H] ms into the current line
    /* +0x08 */ uint32_t countdown_ms;        // [H] time left in states 3 and 4 (§2b)
    /* +0x0c */ uint32_t state;               // [H] §2a
    /* +0x10 */ uint32_t unread_10;           // [U] zeroed with the bank; no reader (class census)
    /* +0x14 */ I3D_sound *sound;             // [H] the voice clip from sounds\%08d.wav
    /* +0x18 */ I3D_frame *owner_frame;       // [H] the speaker's frame
    /* +0x1c */ I3D_frame *morph_frame;       // [H] the head morph frame, or 0
    /* +0x20 */ I3D_frame *neck_target;       // [H] the speaker's neck aim target
    /* +0x24 */ I3D_frame *eye_target;        // [H] the speaker's eye aim target
    /* +0x28 */ float    viseme_0;            // [H] current viseme value, byte 0 of the entry
    /* +0x2c */ float    viseme_1;            // [H] current viseme value, byte 1 of the entry
    /* +0x30 */ float    viseme_2;            // [H] current viseme value, byte 2 of the entry
    /* +0x34 */ float    viseme_3;            // [H] current viseme value, byte 3 of the entry
    /* +0x38 */ float    viseme_4;            // [H] current viseme value, byte 4 of the entry
    /* +0x3c */ float    viseme_5;            // [H] current viseme value, byte 5 of the entry
    /* +0x40 */ float    mouth_fade_in;       // [H] 0..1, ramps the mouth in at the start of a line
} S_speech_channel;
```

**Why 16 channels.** `speech_init_channels` zeroes `0x110` dwords and every accessor bounds-checks
`0 <= i < 0x10`; the bank is the only place a line can play, and when all 16 are busy
`speech_get_free_channel` returns 15 rather than failing. `unread_10` is zeroed with the bank
(`speech_init_channels`, the two release functions, `speech_invalidate_channel_for_actor`) and never
loaded.

**The viseme values** keep numbers because nothing in the game files names what each one moves.
`speech_channel_set_human` seeds a neutral set and reads it straight back to rebuild the three morph rows as
products of these values (e.g. row 0 = `(0, a·b, (1−a)·b, (1−b)(1−c), (1−b)·c)` with `a,b,c` =
`+0x28/+0x2c/+0x30`); `speech_tick_channels` writes each frame's values and rebuilds the rows the same way.

### 1a. `G_DabingMorphObj` — one timed face morph, `0x10` bytes

The element of the vector `g_speech_morph_objects_begin` / `_end` / `_cap` (allocator byte
`g_speech_morph_objects_alloc`); `speech_morph_frame_list_add` appends one per cutscene morph key.

```c
typedef struct G_DabingMorphObj {             // 0x10 bytes
    /* +0x00 */ I3D_frame *frame;             // [H] the morphed frame
    /* +0x04 */ uint32_t duration_ms;         // [H] how long the morph lasts
    /* +0x08 */ uint32_t elapsed_ms;          // [H] time into the morph
    /* +0x0c */ uint8_t  fade_out_channel_0;  // [H] morph group 0: the channel faded out, weight 1 - t
    /* +0x0d */ uint8_t  fade_in_channel_0;   // [H] morph group 0: the channel faded in, weight t
    /* +0x0e */ uint8_t  fade_out_channel_1;  // [H] morph group 1: the channel faded out, weight 1 - t
    /* +0x0f */ uint8_t  fade_in_channel_1;   // [H] morph group 1: the channel faded in, weight t
} G_DabingMorphObj;
```

**The four channel bytes.** `speech_morph_frame_list_add` copies them as one dword from the key's
`+0x28`. Each frame `speech_tick_channels` passes byte `+0x0c` (morph group 0) and `+0x0e` (group 1)
with the weight `1 − t`, and bytes `+0x0d` (group 0) and `+0x0f` (group 1) with `t`, where
`t = elapsed_ms / duration_ms`; the decomp hides these float arguments, so they are read from the
disassembly of the morph loop. When the morph ends the tick sets channels `+0x0d` / `+0x0f` to 1.0. So
a record cross-fades from one morph channel to another in each of two groups. The cutscene key it is
built from: `+0x00` start, `+0x04` duration, `+0x08` the frame name, `+0x28` the four channel bytes.

## 2. Field evidence

Read from all twenty bodies. An `alias[i * 0x11]` access is by name; a walking-pointer access is
`piVar[-n]` from the alias the loop starts at, resolved by hand in `speech_release_channels`,
`speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor`,
`speech_update_look_at_targets`, `speech_pause_all_channels` and `speech_tick_channels`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | viseme_table | H | `speech_channel_speak_line` | `speech_tick_channels`, `speech_channel_speak_line`, `speech_release_channels`, `speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor` | `tables\dat\%08d.dat`, §5 |
| +0x4 | time_ms | H | `speech_channel_speak_line`, `speech_tick_channels` | `speech_tick_channels`, `speech_channel_is_speaking` | one viseme per **40 ms**; the tick advances it by the frame delta without a clip, and otherwise copies the clip's play cursor |
| +0x8 | countdown_ms | H | `speech_channel_speak_line`, `speech_channel_stop`, `speech_tick_channels` | `speech_tick_channels` | §2b |
| +0xc | state | H | `speech_channel_set_human`, `speech_channel_set_model`, `speech_channel_speak_line`, `speech_channel_stop`, `speech_tick_channels` | `speech_get_free_channel`, `speech_channel_is_speaking`, `speech_channel_stop`, `speech_tick_channels`, `speech_pause_all_channels`, `speech_release_channels`, `speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor` | §2a |
| +0x10 | unread_10 | U | `speech_init_channels`, `speech_release_channels`, `speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor` (all zero it with the bank) | - | no reader (class census) |
| +0x14 | sound | H | `speech_channel_speak_line`, `speech_release_channels`, `speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor` | `speech_channel_speak_line`, `speech_channel_is_speaking`, `speech_channel_stop`, `speech_tick_channels`, `speech_pause_all_channels` | - |
| +0x18 | owner_frame | H | `speech_channel_set_human`, `speech_channel_set_model` | `speech_channel_speak_line`, `speech_channel_set_look_at_frame`, `speech_invalidate_channel_for_actor` | `C_human+0x68` or the model |
| +0x1c | morph_frame | H | `speech_channel_set_human`, `speech_channel_set_model` | `speech_tick_channels`, `speech_channel_speak_line`, `speech_release_channels_and_reset_morphs`, `speech_invalidate_channel_for_actor` | accepted only with `+0x110 == 1` and `+0x1d4 == 'SMMR'`, else 0 |
| +0x20 | neck_target | H | `speech_channel_set_human`, `speech_channel_set_model` | `speech_channel_set_look_at_frame`, `speech_update_look_at_targets` | `C_human+0x58c`, the model frame `targetN`; mask bit 1 |
| +0x24 | eye_target | H | `speech_channel_set_human`, `speech_channel_set_model` | `speech_channel_set_look_at_frame`, `speech_update_look_at_targets` | `C_human+0x588`, the model frame `targetE`; mask bit 0 |
| +0x28 | viseme_0 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 0.46 |
| +0x2c | viseme_1 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 0 |
| +0x30 | viseme_2 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 0.53 |
| +0x34 | viseme_3 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 1.0 |
| +0x38 | viseme_4 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 0.11 |
| +0x3c | viseme_5 | H | `speech_channel_set_human`, `speech_tick_channels` | `speech_channel_set_human`, `speech_tick_channels` | seeded 0.6 |
| +0x40 | mouth_fade_in | H | `speech_channel_speak_line`, `speech_tick_channels` | `speech_tick_channels` | zeroed by a new line; while under 1.0 state 2 adds `0.008` per ms, a 125 ms ramp |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | frame | H | `speech_morph_frame_list_add` | `speech_tick_channels` | a scene frame found by name, of the kind `+0x1d4 == 'MRPH'` |
| +0x4 | duration_ms | H | `speech_morph_frame_list_add` | `speech_tick_channels` | the key's `+0x04`; past it the record is erased |
| +0x8 | elapsed_ms | H | `speech_morph_frame_list_add`, `speech_tick_channels` | `speech_tick_channels` | seeded `clock - key +0x00`, so a late key starts part way |
| +0xc | fade_out_channel_0 | H | `speech_morph_frame_list_add` (the key's `+0x28`, one dword for the four bytes) | `speech_tick_channels` | weight `1 - elapsed / duration` |
| +0xd | fade_in_channel_0 | H | `speech_morph_frame_list_add` | `speech_tick_channels` | weight `elapsed / duration`; 1.0 when the morph ends |
| +0xe | fade_out_channel_1 | H | `speech_morph_frame_list_add` | `speech_tick_channels` | as `+0xc`, group 1 |
| +0xf | fade_in_channel_1 | H | `speech_morph_frame_list_add` | `speech_tick_channels` | as `+0xd`, group 1 |

### 2a. `state` — one channel

| Value | Meaning | Evidence |
|---|---|---|
| 0 | free | the bank zeroing, `speech_tick_channels` at the end of states 3 and 4; `speech_get_free_channel` scans for it |
| 1 | bound to an actor / model, silent | `speech_channel_set_human`, `speech_channel_set_model` |
| 2 | speaking with a voice clip, or with a viseme table and no clip | `speech_channel_speak_line`; `speech_channel_is_speaking` |
| 3 | stopping / finished | `speech_channel_stop`, the tick's timeout |
| 4 | speaking **without** a clip, timed by the text length | `speech_channel_speak_line` when no clip and no viseme table loaded and its third argument is set; `speech_channel_is_speaking` |
| 5 | **externally owned sound, never released** | `speech_channel_speak_line` when no clip loaded and its fourth argument is 5: it takes the mission sound `14_Coucou`; the guard in `speech_channel_stop`, `speech_release_channels`, `speech_pause_all_channels` |

State 5 is why those functions refuse to touch a channel: its sound object belongs to the mission's sound
list (`g_act_mission+0x10`), not to the channel.

### 2b. `countdown_ms` — the values

`200` from `speech_channel_speak_line`, or `strlen(text) × 30 + 1000` for a state-4 line; `1` from
`speech_channel_stop`. It counts down in states 3 and 4: `speech_tick_channels` subtracts the frame delta
and frees the channel at 0. A clip that ends, or a restart, leaves 200; a stop leaves 1, so the fade ends on
the next frame; a text-only line counts down its whole duration. In state 3 the tick also scales the mouth
weights by `countdown × 0.005`, a 200 ms fade.

## 3. Method family

11 of the 13 `Dabing*` functions twinned, all CONFIRMED; the other two are PS2-only (§7e). The four
static-init rows hold a `G_DabingMorphObj` vector. The subsystem has no `this`: every function takes the
channel index in ECX, which is MSVC's `__fastcall`, or no channel at all.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005469a0 | `speech_morph_list_static_init_entry` | `g_speech_morph_objects` | - | - | - | `.CRT$XCU` entry of the vector: calls the next two; the unit's first start |
| 0x005469b0 | `speech_morph_list_static_init` | `g_speech_morph_objects` | - | - | - | empties the `G_DabingMorphObj` vector |
| 0x005469d0 | `speech_morph_list_register_atexit` | `g_speech_morph_objects` | - | - | - | `atexit(&speech_morph_list_static_dtor)` |
| 0x005469e0 | `speech_morph_list_static_dtor` | `g_speech_morph_objects` | `__dt__Q23std74__vector_deleter<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>>Fv` | 0x0029d2a0 | STRUCTURAL | frees that vector |

### 3b. Accessors

None: the subsystem has no receiver.

### 3c. Behaviour

None: the subsystem has no receiver; see 3d.

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00546a00 | `speech_init_channels` | `DabingInit__Fv` | 0x0029ac40 | CONFIRMED | zeroes the bank and empties the morph vector; caller `game_init` |
| 0x00546a50 | `speech_release_channels` | `DabingDone__Fv` | 0x0029ac80 | CONFIRMED | releases every clip unless state 5, frees the viseme tables, zeroes the bank; caller `game_done` |
| 0x00546ae0 | `speech_get_free_channel` | `DabingGetFreeChanel__Fv` | 0x0029ad30 | CONFIRMED | downward scan from channel 15 for state 0; 15 when all are busy |
| 0x00546b10 | `speech_channel_set_human` | `DabingSetHuman__FiP7C_human` | 0x0029ad90 | CONFIRMED | binds an actor and seeds the morph weights |
| 0x00546ea0 | `speech_release_channels_and_reset_morphs` | - | - | - | the heavy teardown: weights to `(1,0,0,0,0)`, frees tables, releases sounds unless state 5; caller `recorder_end_sequence_and_restore_scene` |
| 0x00547050 | `speech_channel_set_model` | `DabingSetModel__FiP9I3D_model` | 0x0029b250 | CONFIRMED | binds by the frame literally named `"base"`; both target frames cleared |
| 0x00547210 | `speech_invalidate_channel_for_actor` | `DabingInvalidate__FP7C_human` | 0x0029b4e0 | CONFIRMED | frees and zeroes every channel owned by the actor's frame; caller `game_invalidate_actor` |
| 0x005473b0 | `speech_channel_speak_line` | `DabingSpeak__FiUibUi` | 0x0029b760 | CONFIRMED | **the `human_talk` implementation** (§6) |
| 0x00547960 | `speech_channel_set_look_at_frame` | - | - | - | points the eye and / or neck target at a frame, or hides them; caller `speech_apply_recorder_key` |
| 0x00547a80 | `speech_channel_is_speaking` | `DabingIsSpeaking__Fi` | 0x0029bfd0 | CONFIRMED | the predicate the script waits on (§6) |
| 0x00547b00 | `speech_channel_stop` | `DabingStop__Fi` | 0x0029c040 | CONFIRMED | **the `human_stoptalk` implementation**: fades the clip, countdown 1, state 3, unless state 5 |
| 0x00547b50 | `speech_update_look_at_targets` | - | - | - | per-frame refresh of the eye and neck targets; caller `recorder_tick_tracks` |
| 0x00547c10 | `speech_tick_channels` | `DabingTick__FUi` | 0x0029c0d0 | CONFIRMED | the per-frame lip-sync tick; callers `game_tick`, `recorder_tick_speech` |
| 0x00548420 | `speech_pause_all_channels` | `DabingPauseSounds__Fbb` | 0x0029ced0 | CONFIRMED | pause / resume all; callers `game_pause_all_sounds`, `game_play_all_sounds` |
| 0x005484b0 | `speech_apply_recorder_key` | - | - | - | applies one cutscene speech key: the speaker, a line, or the look-at frame; caller `recorder_tick_tracks` |
| 0x005485a0 | `speech_morph_frame_list_add` | - | - | - | appends a `G_DabingMorphObj` for a frame of the `'MRPH'` kind; caller `recorder_tick_tracks` |

## 4. How the subsystem is identified

1. **Separation from `C_recorder`.** A scan of `.text` for the immediate `mov ecx, 0x00656368` (the recorder
   singleton, 59 sites) proves that six functions in this address range are `C_recorder` methods; the rest
   of the range is not the recorder.
2. **The 16-slot bank's own shape.** Every accessor bounds-checks `0 <= i < 0x10`, the loops walk `0x11`
   dwords at a time, and `speech_release_channels` runs from `base + 0x14` up to `0x00656944`, exactly
   `16 × 0x44` later.
3. **The Mafia Script Data Reference** names the feature (quoted above).

A PC-only finding about `C_recorder` (`structs/recorder.md`) covers that class's symbols only, not this
bank that shares its address range: the speech bank has PS2 twins (§7).

## 5. The viseme table, `tables\dat\%08d.dat`

One file per textdb line: a `u32` entry count, a `u32` that is 1 in every file, then one 8-byte entry per
**40 ms** of speech. All 3403 non-empty shipped files (4 more are empty) are exactly `8 + 8 × count` bytes.
Bytes 0..5 of an entry are the six viseme parameters, each byte / 255, and every value 0..255 occurs in them;
byte 6 is 1 in 98 % of the 195 498 entries and byte 7 is a 0/1 flag.

## 6. What actually happens when a character speaks

1. The script issues `human_talk actor, textid`; `entity_process_command` and `program_process_command` hold
   the callers.
2. `speech_get_free_channel` scans **downward** from channel 15 for a free slot; when all 16 are busy it
   reuses channel 15 rather than failing.
3. `speech_channel_set_human` binds the actor: the frame at `C_human+0x68` becomes the owner,
   `+0x588`/`+0x58c` become the eye and neck aim targets (the model frames `targetE` and `targetN`; the model's
   `Reye` / `Leye` joints aim at `targetE`), and `+0x57c` is validated as a morph frame. All **3 × 5 morph
   channel weights** are first set to `(1, 0, 0, 0, 0)` through `frame+0x140` slot 8, then rebuilt from the six
   seeded values (§1). The channel goes to state 1.
4. `speech_channel_speak_line` loads two files keyed on the **same textdb id**: `sounds\%08d.wav` (the voice
   clip, when the voice volume is above 0.01) and `tables\dat\%08d.dat` (the viseme table). Its fourth argument
   picks the sound's 3-D attributes (values 1 to 4 and 6, with a default). With its third argument set it also
   adds the subtitle when the speaker is within 12 units of the active camera, or the argument is 3 or 4, or
   the channel has no owner; the subtitle lasts the clip's length + 500 ms.
5. `speech_tick_channels` runs every frame: it advances the viseme index at **40 ms per entry** and
   resynchronises against the clip's play cursor (sound slots `0x80`/`0x84`) with a 300 ms tolerance, writing
   the morph weights that move the mouth. It then runs the `G_DabingMorphObj` list.
6. The script polls `speech_channel_is_speaking`: true in state 2 or 4, or while the clip plays with
   `time_ms` behind its cursor and its slot `0x84` value under `time_ms + 300`. That is why the Mafia Script
   Data Reference says the script usually has to wait.
7. `speech_channel_stop` (`human_stoptalk`) fades the clip and moves the channel to state 3.

The gas-station attendant is the clearest live example: `pumper_choose_next_action` takes a free channel and
speaks a line three times over, and `pumper_ai` polls the predicate (`structs/pumper.md` §2a).

**In a recorded cutscene** the speech keys come from the track: `speech_apply_recorder_key` binds the speaker,
speaks the line, or points the speaker's eye and neck targets at a named scene frame through
`speech_channel_set_look_at_frame` (an empty name hides them), and `speech_update_look_at_targets` refreshes the
targets every frame. Nothing outside a recorded sequence moves them.

## 7. PC vs PS2

The PS2 source file is `g_dabing.cpp`; the subsystem is a set of free functions there. `C_recorder` is absent
from the PS2 build, but this bank is not. The twins are in §3d.

### 7a. Size comparison

| PC name | PC size | PS2 symbol | PS2 size | Ratio |
|---|---:|---|---:|---:|
| `speech_init_channels` | 80 | `DabingInit__Fv` | 52 | 1.54 |
| `speech_release_channels` | 144 | `DabingDone__Fv` | 164 | 0.88 |
| `speech_get_free_channel` | 48 | `DabingGetFreeChanel__Fv` | 84 | 0.57 |
| `speech_channel_set_human` | 912 | `DabingSetHuman__FiP7C_human` | 1204 | 0.76 |
| `speech_channel_set_model` | 448 | `DabingSetModel__FiP9I3D_model` | 644 | 0.70 |
| `speech_invalidate_channel_for_actor` | 416 | `DabingInvalidate__FP7C_human` | 628 | 0.66 |
| `speech_channel_speak_line` | 1456 | `DabingSpeak__FiUibUi` | 2148 | 0.68 |
| `speech_channel_is_speaking` | 128 | `DabingIsSpeaking__Fi` | 108 | 1.19 |
| `speech_channel_stop` | 80 | `DabingStop__Fi` | 140 | 0.57 |
| `speech_tick_channels` | 2064 | `DabingTick__FUi` | 3576 | 0.58 |
| `speech_pause_all_channels` | 144 | `DabingPauseSounds__Fbb` | 284 | 0.51 |

### 7b. The three byte-level confirmations

| | PS2 | PC |
|---|---|---|
| **the bank** | `memset(dabingChanel, 0, 0x480)` = 16 × 0x48 | zero `0x110` dwords = `0x440` = 16 × **0x44** |
| **free-channel scan** | `i = 0xf; off = 0x438; i--; off -= 0x48;` fallback `return 0xf` | `i = 0xf; p = &elem15.state; p -= 0x11; i--;` fallback `return 0xf` |
| **stop** | bounds, `state != 5`, sound `(0,1)`, flag = 1, `state = 3` | statement-identical |

The state-5 guard in the stop, release and pause functions is present on both builds.

### 7c. What the PS2 supplies

* **`dabingChanel`**: the developers' own name for the bank (and their spelling of "channel").
* **`G_DabingMorphObj`**: the element type of the vector at `0x006564e4`; the PC decomp only gives the stride.
* **`dabingPreData`**: the preload buffer freed by `DabingPreUnload`.

### 7d. A separate speech structure

PS2 `C_game` carries `AddHumanTalk` / `DelHumanTalk` / `DeathHumanTalk` over a
`std::vector<C_game::S_enemy_talk>` at `C_game+0x135c`, a **script-side registry of who is currently talking**,
distinct from the 16-channel bank. `DeathHumanTalk` calls `DabingStop` on the channel it drops, which is how the
two are wired together. The PC counterpart is the talk queue at `C_game+0x2f44`, whose three functions
`game_human_talk_add`, `game_human_talk_del` and `game_human_talk_stop_for_human` are the twins of the three PS2
methods (`structs/game.md` §4g); its element is the 8-byte `{human, speech id}` pair.

### 7e. The functions with no twin

* **The four PC functions without a PS2 twin are reached only from `C_recorder`**, which does not exist on the PS2
  build (`structs/recorder.md` §10): `speech_apply_recorder_key` and `speech_update_look_at_targets` are called
  only by `recorder_tick_tracks`, `speech_channel_set_look_at_frame` only by `speech_apply_recorder_key`, and
  `speech_release_channels_and_reset_morphs` only by `recorder_end_sequence_and_restore_scene`. A byte scan for
  `E8` / `E9` callsites and data references finds no other referrer. `speech_morph_frame_list_add` is called only
  by `recorder_tick_tracks` too, and the PS2 has no `Dabing*` function for it.

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| `0x0029cff0` | `DabingPreLoad__FPCc` | `C_game::Init` calls it with the mission name: it frees `dabingPreData`, reads a per-mission file into it and turns the file's stored offsets into pointers. `DabingSpeak` looks a line's viseme table up in that buffer; the PC has no such buffer, because `speech_channel_speak_line` opens the line's own `tables\dat\%08d.dat` every time. `speech_apply_recorder_key` sits at this function's emission rank, but its body applies a cutscene key and loads nothing |
| `0x0029d130` | `DabingPreUnload__Fv` | `C_game::Done` calls it to free the same buffer |

## 8. Related

`structs/recorder.md` (the `C_recorder` unit immediately below this one, and the caller of
`speech_apply_recorder_key` / `speech_tick_channels`) · `structs/pumper.md` §5 (the attendant, the clearest live
consumer) · `structs/human.md` (`+0x57c` morph frame, `+0x588`/`+0x58c` eye and neck targets) ·
`structs/game.md` §4g (the talk queue) · `script-mapping/artifacts/opcodes-verified.md`.

## 9. Open items

| Item | State |
|---|---|
| 1. in game | watch a character's mouth during `human_talk` and confirm the 40 ms viseme rate; check what happens when a 17th line is requested (the code reuses channel 15) |
