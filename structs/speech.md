# The speech / lip-sync subsystem (`speech_`)

**Created** 2026-08-19 18:06 +0200 · **Modified** 2026-09-24 21:22 +0200

| | |
|---|---|
| PC prefix | **`speech_`** |
| PC translation unit | `[0x005469a0, 0x00548830)` — 20 function starts, all named |
| Bank | `g_speech_channels` @ **`0x006564f0`**, 16 channels, stride **`0x44`** |
| PS2 | free functions in `g_dabing.cpp`, 12 of 13 symbols twinned (§5) |

This is the system that makes a character **say a line and move their mouth**. The Mafia Script
Data Reference describes it from the script side:

> `human_talk xx, yyyy, zz` — *"Character xx speaks line yyyy. The script continues while
> speaking, so it usually has to wait. … but the text must also have a sound with the same name
> yyyy"*
> `human_stoptalk xx` — *"Character xx stops speaking."*

That one paragraph names the whole thing: a **textdb string id** is also a **sound file
name**, and the script has to poll a *still speaking?* predicate because the call returns
immediately.

---

## 1. How the subsystem is identified

1. **Separation from `C_recorder`.** A scan of `.text` for the immediate
   `mov ecx, 0x00656368` (the recorder singleton, 59 sites) proves that six functions in this
   address range are `C_recorder` methods; the rest of the range is not the recorder.
2. **The 16-slot bank's own shape.** Every accessor bounds-checks `0 <= i < 0x10`, the
   loops walk `0x11` dwords at a time, and `speech_release_channels` runs from
   `base + 0x14` up to `0x00656944` — exactly `16 × 0x44` later.
3. **The Mafia Script Data Reference** names the feature (quoted above).

A PC-only finding about `C_recorder` (`structs/recorder.md`) covers that class's symbols only,
not this bank that shares its address range: the speech bank has PS2 twins (§5).

---

## 2. The channel record (`0x44` bytes)

Indexed `g_speech_channels[i]`, `0 ≤ i < 16`. The Ghidra symbols on element 0's fields are
**aliases used as array bases**, not separate objects — they are listed here so that stays
visible.

| Offset | PC symbol on element 0 | Meaning |
|---|---|---|
| `+0x00` | `g_speech_channels` | the viseme table loaded from `tables\dat\%08d.dat` |
| `+0x04` | `g_speech_channel_time_ms` | ms accumulated into the current line; one viseme per **40 ms**; zeroed by `speech_channel_speak_line` |
| `+0x08` | `g_speech_channel_stop_flag` | set to 1 by `speech_channel_stop`; `speech_channel_speak_line` writes `200`, or `strlen(text) × 30 + 1000` for a state-4 line ¹ |
| `+0x0c` | `g_speech_channel_state` | the channel state — see §2a |
| `+0x14` | `g_speech_channel_sound` | the voice clip loaded from `sounds\%08d.wav` |
| `+0x18` | `g_speech_channel_owner_frame` | the speaking actor's frame (`C_human+0x68`) |
| `+0x1c` | `g_speech_channel_morph_frame` | the head morph frame — `+0x110 == 1`, `+0x1d4 == 'SMMR'` |
| `+0x20` | `g_speech_channel_extra_frame_b` | from `C_human+0x58c`; mask bit 1; role unknown (§7) |
| `+0x24` | `g_speech_channel_extra_frame_a` | from `C_human+0x588`; mask bit 0; role unknown (§7) |
| `+0x28` | `g_speech_channel_field_0x28` | morph parameter, seeded `0.46` ² |
| `+0x2c` | `g_speech_channel_field_0x2c` | morph parameter, seeded `0` ² |
| `+0x30` | `g_speech_channel_field_0x30` | morph parameter, seeded `0.53` ² |
| `+0x34` | `g_speech_channel_field_0x34` | morph parameter, seeded `1.0` ² |
| `+0x38` | `g_speech_channel_field_0x38` | morph parameter, seeded `0.11` ² |
| `+0x3c` | `g_speech_channel_field_0x3c` | morph parameter, seeded `0.6` ² |
| `+0x40` | `g_speech_channel_field_0x40` | zeroed by `speech_channel_speak_line` |

¹ So `+0x08` also carries a duration, not only a stop flag; its symbol name is narrower than its
use (§7).
² Written by `speech_channel_set_human`, which then reads them straight back to rebuild the
three morph rows as products of these values (e.g. row 0 = `(0, a·b, (1−a)·b, (1−b)(1−c),
(1−b)·c)` with `a,b,c` = `+0x28/+0x2c/+0x30`). No other function reads them through these
symbols. The six fields are named by offset because no reader outside the binder reveals a
meaning.

### 2a. The state enum

| State | Meaning | Written by | Read by |
|---|---|---|---|
| `0` | free | the bank zeroing | `speech_get_free_channel` scans for it |
| `1` | bound to an actor / model, silent | `speech_channel_set_human`, `speech_channel_set_model` | - |
| `2` | speaking with a voice clip | `speech_channel_speak_line` | `speech_channel_is_speaking` |
| `3` | stopping / finished | `speech_channel_stop`, the tick's timeout | - |
| `4` | speaking **without** a clip — timed by the text length (`+0x08`) | `speech_channel_speak_line` when no clip and no viseme table loaded and `param_3` is set | `speech_channel_is_speaking` |
| `5` | **externally owned sound — never released** | `speech_channel_speak_line` when no clip loaded and `param_4 == 5`: it takes the mission sound `14_Coucou` instead | the guard in `speech_channel_stop`, `speech_release_channels`, `speech_pause_all_channels` |

State 5 is why three functions refuse to touch a channel: its sound object belongs to the
mission's sound list (`g_act_mission+0x10`), not to the channel.

---

## 3. What actually happens when a character speaks

1. The script issues `human_talk actor, textid`.
2. `speech_get_free_channel` scans **downward** from channel 15 for a free slot; when all
   16 are busy it reuses channel 15 rather than failing.
3. `speech_channel_set_human` binds the actor: the frame at `C_human+0x68` becomes the
   owner, `+0x588`/`+0x58c` become the two accessory frames, and `+0x57c` is validated as
   a morph frame (`frame+0x110 == 1`, `frame+0x1d4 == 'SMMR'`). All **3 × 5 morph channel
   weights** are first set to `(1, 0, 0, 0, 0)` through `frame+0x140` slot 8, then rebuilt from
   the six seeded parameters (§2 ²). The channel goes to state 1.
4. `speech_channel_speak_line` loads two files keyed on the **same textdb id**:
   `sounds\%08d.wav` (the voice clip) and `tables\dat\%08d.dat` (the viseme table).
   `recorder_open_speech_sound_for_text_id` is what resolves the clip — it refuses ids
   below `0x400`, subtracts `0x400`, and tries each prefix in the sound search-path list.
5. `speech_tick_channels` runs every frame: it advances the viseme index at **40 ms per
   entry** and resynchronises against the clip's play cursor (sound slots `0x80`/`0x84`)
   with a 300 ms tolerance, writing the morph weights that move the mouth.
6. The script polls `speech_channel_is_speaking` (state 2 or 4) — which is why the Mafia Script
   Data Reference says the script usually has to wait.
7. `speech_channel_stop` (`human_stoptalk`) fades the clip and moves the channel to state 3.

The gas-station attendant is the clearest live example: `pumper_choose_next_action` takes a
free channel and speaks a line three times over, and `pumper_ai` polls the predicate
(`structs/pumper.md` §2).

---

## 4. Method coverage

| PC VA | PC name | Role |
|---|---|---|
| `0x005469a0` | `speech_morph_list_static_init_entry` | the TU's static-init entry — marks the TU boundary |
| `0x005469b0` | `speech_morph_list_static_init` | zeroes the `G_DabingMorphObj` vector |
| `0x005469d0` | `speech_morph_list_register_atexit` | `atexit(&speech_morph_list_static_dtor)` |
| `0x005469e0` | `speech_morph_list_static_dtor` | frees that vector |
| `0x00546a00` | `speech_init_channels` | zeroes the bank; from `game_init` |
| `0x00546a50` | `speech_release_channels` | releases every clip; from `game_done` |
| `0x00546ae0` | `speech_get_free_channel` | downward scan from channel 15 |
| `0x00546b10` | `speech_channel_set_human` | binds an actor + seeds the morph weights |
| `0x00546ea0` | `speech_release_channels_and_reset_morphs` | the heavy teardown: weights to `(1,0,0,0,0)`, frees tables, releases sounds unless state 5 |
| `0x00547050` | `speech_channel_set_model` | binds by the frame literally named `"base"` |
| `0x00547210` | `speech_invalidate_channel_for_actor` | from `game_invalidate_actor` |
| `0x005473b0` | `speech_channel_speak_line` | **the `human_talk` implementation** |
| `0x00547960` | `speech_channel_attach_accessory_frame` | attaches an accessory frame to a channel |
| `0x00547a80` | `speech_channel_is_speaking` | the predicate the script waits on |
| `0x00547b00` | `speech_channel_stop` | **the `human_stoptalk` implementation** |
| `0x00547b50` | `speech_update_accessory_frames` | per-frame accessory refresh |
| `0x00547c10` | `speech_tick_channels` | the per-frame lip-sync tick |
| `0x00548420` | `speech_pause_all_channels` | pause / resume all |
| `0x005484b0` | `speech_preload_for_record` | binds a speaker from a recorder track key |
| `0x005485a0` | `speech_morph_frame_list_add` | appends a `G_DabingMorphObj` record |

---

## 5. PC vs PS2

The PS2 source file is `g_dabing.cpp` (`__sinit_g_dabing.cpp` in the source-file manifest); the
subsystem is a set of free functions there. `C_recorder` is absent from the PS2 build, but this
bank is not.

### 5a. The twin table

| PC name | PC size | PS2 symbol | PS2 size | ratio | Grade |
|---|---:|---|---:|---:|---|
| `speech_init_channels` | 80 | `DabingInit__Fv` | 52 | 1.54 | CONFIRMED |
| `speech_release_channels` | 144 | `DabingDone__Fv` | 164 | 0.88 | CONFIRMED |
| `speech_get_free_channel` | 48 | `DabingGetFreeChanel__Fv` | 84 | 0.57 | CONFIRMED |
| `speech_channel_set_human` | 912 | `DabingSetHuman__FiP7C_human` | 1204 | 0.76 | CONFIRMED |
| `speech_release_channels_and_reset_morphs` | 432 | - | - | - | open ¹ |
| `speech_channel_set_model` | 448 | `DabingSetModel__FiP9I3D_model` | 644 | 0.70 | CONFIRMED |
| `speech_invalidate_channel_for_actor` | 416 | `DabingInvalidate__FP7C_human` | 628 | 0.66 | CONFIRMED |
| `speech_channel_speak_line` | 1456 | `DabingSpeak__FiUibUi` | 2148 | 0.68 | CONFIRMED |
| `speech_channel_attach_accessory_frame` | 288 | - | - | - | open ¹ |
| `speech_channel_is_speaking` | 128 | `DabingIsSpeaking__Fi` | 108 | 1.19 | CONFIRMED |
| `speech_channel_stop` | 80 | `DabingStop__Fi` | 140 | 0.57 | CONFIRMED |
| `speech_update_accessory_frames` | 192 | - | - | - | open ¹ |
| `speech_tick_channels` | 2064 | `DabingTick__FUi` | 3576 | 0.58 | CONFIRMED |
| `speech_pause_all_channels` | 144 | `DabingPauseSounds__Fbb` | 284 | 0.51 | CONFIRMED |
| `speech_preload_for_record` | 240 | `DabingPreLoad__FPCc` | 308 | 0.78 | CONFIRMED |
| - | - | `DabingPreUnload__Fv` | 44 | - | open ² |

¹ No PS2 rank: PC-only or folded into a PS2 twin; not decided (§7).
² No PC start located; folded (inlined) on PC, not absent.

### 5b. The three byte-level confirmations

| | PS2 | PC |
|---|---|---|
| **the bank** | `memset(dabingChanel, 0, 0x480)` = 16 × 0x48 | zero `0x110` dwords = `0x440` = 16 × **0x44** |
| **free-channel scan** | `i = 0xf; off = 0x438; i--; off -= 0x48;` fallback `return 0xf` | `i = 0xf; p = &elem15.state; p -= 0x11; i--;` fallback `return 0xf` |
| **stop** | bounds, `state != 5`, sound `(0,1)`, flag = 1, `state = 3` | statement-identical |

The state-5 guard in the stop, release and pause functions is present on both builds.

### 5c. What the PS2 supplies

* **`dabingChanel`** — the developers' own name for the bank (and their spelling of
  "channel").
* **`G_DabingMorphObj`** — the element type of the vector at `0x006564e4`; the PC decomp only
  gives the stride.
* **`dabingPreData`** — the preload buffer freed by `DabingPreUnload`.

### 5d. A separate speech structure

PS2 `C_game` carries `AddHumanTalk` / `DelHumanTalk` / `DeathHumanTalk` over a
`std::vector<C_game::S_enemy_talk>` at `C_game+0x135c` — a **script-side registry of who
is currently talking**, distinct from the 16-channel bank. `DeathHumanTalk` calls
`DabingStop` on the channel it drops, which is how the two are wired together. The PC
counterpart is §7.

---

## 6. Cross-references

`structs/recorder.md` (the `C_recorder` TU immediately below this one, and the caller of
`speech_preload_for_record` / `speech_tick_channels`) · `structs/pumper.md` §2 (the
attendant, the clearest live consumer) · `structs/human.md` (`+0x57c` morph frame,
`+0x588`/`+0x58c` accessory frames) · `script-mapping/artifacts/opcodes-verified.md`.

---

## 7. Open items

| Item | State |
|---|---|
| The two accessory frames | `C_human+0x588` / `+0x58c` are bound to every channel and refreshed every frame; what they are is unknown. A frame-name dump on a live model would answer it |
| `+0x08` | written as `1` (stop), `200`, and a text-length duration; one name does not cover all three uses |
| `+0x28` … `+0x3c` | the morph-weight meaning of each seeded parameter |
| The three rank-less PC functions | `speech_release_channels_and_reset_morphs`, `speech_channel_attach_accessory_frame`, `speech_update_accessory_frames`. `DabingSetHuman` is 292 B larger than its PC twin, which suggests a fold and proves nothing |
| `DabingPreUnload` | 44 B on PS2, no PC start; where the PC frees `dabingPreData` is unlocated |
| `C_game::S_enemy_talk` | the PC counterpart of the PS2 talk registry (§5d) is unsearched |
| In game | watch a character's mouth during `human_talk` and confirm the 40 ms viseme rate; check what happens when a 17th line is requested (the code reuses channel 15) |
