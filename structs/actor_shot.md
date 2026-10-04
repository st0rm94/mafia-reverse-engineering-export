# `C_actor_shot` — the timed weapon-fire actor

**Created** 2026-08-04 12:05 +0200 · **Modified** 2026-10-04 11:53 +0200

A `C_actor` subclass that **fires a weapon on a timer**: a placed scene object which, every N
milliseconds, spawns a muzzle particle and issues a shot. N is not authored — it is read out of
the selected weapon's own phase table, so the object fires at that weapon's natural cadence.

| | |
|---|---|
| PC prefix | **`actor_shot_`** |
| Base class | **`C_actor`** (`structs/actor.md`) — ctor calls `actor_ctor`, dtor tail-calls `actor_dtor`, `init` / `game_init` / `game_done` each chain to the base first |
| PC kind (`entity+0x10`) | **`0x21`** ¹ |
| PC vtable | `.rdata 0x0063b720` |
| PC translation unit | `[0x0044e9a0, 0x0044eebc)` ² — 8 functions, all named |

¹ Byte-proven from the ctor, and corroborated from the other side: `actor_game_init`
special-cases exactly `kind != 1 && kind != 0x21` in two places, so the base class already
knows about this subclass.
² One MSVC translation unit, bracketed by the CRT static-init pairs at `0x0044e970` (above) and
`0x0044ef20` (below, which opens `C_airplane`).

**Where instances come from.** The only constructor call is `mission_create_actor` case `0x21`
(`operator_new(0x8c)`). Of that factory's eight callers, two take the kind from shipped content:
`mission_open_bin` from a scene's `scene2.bin` object definition and `game_load_differences` from a
`.chg` difference file (`structs/chunk.md` §4b); the savegame restore and the script's actor
duplication copy the kind of an existing actor, and the garage and free-ride setups pass literal
kinds. **No shipped scene places one** — all 79 `scene2.bin` files carry 13 631 object definitions
(chunk `0x4c53` → `0xAE20` → `0xAE21` → kind in `0xAE22`) and none has kind `0x21` — **but 38 of
the 92 `.chg` files do**: 77 actor records (chunk 200) and 45 frame bindings (chunk 400) of kind
`0x21`, created when a script loads that difference file with `loaddifferences`.

---

## 1. Field layout

```c
typedef struct C_actor_shot {          /* extends C_actor */
    /* ...  */ C_actor      base;
    /* +0x10 */ int          kind;          // [H] = 0x21 (C_actor field)
    /* +0x68 */ I3D_frame   *frame;         // [H] the actor's frame (C_actor field)
    /* +0x70 */ void        *emitter;       // [H] renderer object, factory type 4;
                                            //     created in game_init, released in game_done
    /* +0x74 */ int          weapon_id;     // [H] index into g_game_item_table
    /* +0x78 */ int          interval_ms;   // [H] countdown to the next shot; -1 = re-arm now
    /* +0x7c */ char         enabled;       // [H] firing gate; update() returns early when 0
    /* +0x84 */ char         use_frame_dir; // [H] aim along the frame forward vector
    /* +0x88 */ int          particle_handle; // [H] muzzle particle, -1 = none
} C_actor_shot;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x10 | kind | H | `actor_shot_ctor` (= 0x21) | `folded_get_dword_0x10` (its `C_actor::GetKind` role), `actor_game_init` | the engine's stand-in for RTTI, which the PC build lacks |
| +0x70 | emitter | H | `actor_shot_game_init`, `actor_shot_game_done` | `actor_shot_update` | renderer factory type 4, parented to the frame at the origin |
| +0x74 | weapon_id | H | `actor_shot_set_weapon` | `actor_shot_update` | indexes `g_game_item_table`, stride 0xbc |
| +0x78 | interval_ms | H | `actor_shot_game_init` (=-1), `actor_shot_update`, `actor_shot_track_apply_key_effects` (=-1) | `actor_shot_update` | re-armed from the weapon's phase table, not from an authored value |
| +0x7c | enabled | H | `actor_shot_game_init` (=0), `actor_shot_track_apply_key_effects` (key flag bit 12) | `actor_shot_update` | - |
| +0x84 | use_frame_dir | H | `actor_shot_track_apply_key_effects` (key flag bit 15) | `actor_shot_update` | selects frame-forward aiming. Its only writer is the replay of a recorded key (§2), so outside playback it keeps its initial value |
| +0x88 | particle_handle | H | `actor_shot_init` (=-1), `actor_shot_track_apply_key_effects` | `actor_shot_track_apply_key_effects`, `actor_shot_game_done` | the muzzle particle: the key replay creates it with `particle_manager_create_particle` and every holder frees it with `particle_manager_destroy_particle` |

---

## 2. Method coverage — 12 PS2 methods, 8 PC functions

PC-first, ordered by PC VA. PS2 has 12 symbols; 5 of them are 8-byte folded stubs (`AI`,
`NetDirect`, `SaveGameGetSize`, `SaveGameSave`, `SaveGameLoad`) that inherit the C_actor
behaviour, so the class is **7 real methods + 5 stubs**, and the PC block is a **forced 7 ↔ 7**.

| PC VA | PC name | PS2 twin (mangled) | PS2 VA | PS2 size | Status | Role |
|---|---|---|---|---|---|---|
| 0x0044e9a0 | `actor_shot_ctor` | `__ct__12C_actor_shotFv` | 0x00118cc0 | 60 | CONFIRMED | `actor_ctor` + vtable `0x0063b720` + kind `0x21` |
| 0x0044e9c0 | `actor_shot_scalar_deleting_dtor` | - | - | - | STRUCTURAL | MSVC scalar deleting dtor ¹ |
| 0x0044e9e0 | `actor_shot_dtor` | `__dt__12C_actor_shotFv` | 0x00118d00 | 96 | CONFIRMED | tail-calls `actor_dtor`; the class owns no heap state of its own |
| 0x0044e9f0 | `actor_shot_init` | `Init__12C_actor_shotFP9I3D_frame` | 0x00118d60 | 76 | CONFIRMED | `actor_init` then `particle_handle = -1`; 0 on a null/failed frame bind |
| 0x0044ea30 | `actor_shot_update` | `Update__12C_actor_shotFUi` | 0x00118dc0 | 680 | CONFIRMED | the timed-fire tick — see §3 |
| 0x0044ed90 | `actor_shot_game_init` | `GameInit__12C_actor_shotFv` | 0x00119080 | 224 | CONFIRMED | `actor_game_init`, create the emitter, `actor_shot_set_weapon(0)`, exclude the actor from the savegame (`C_actor+0x65`) |
| 0x0044ee50 | `actor_shot_game_done` | `GameDone__12C_actor_shotFv` | 0x00119160 | 88 | CONFIRMED | `actor_game_done`, release the emitter, `particle_manager_destroy_particle` |
| 0x0044ee90 | `actor_shot_set_weapon` | `SetWeapon__12C_actor_shotFi` | 0x001191f0 | 204 | CONFIRMED | store `weapon_id`, build the fire sound from `g_game_item_table` |
| - | *(C_actor base)* | `AI__12C_actor_shotFUi` | 0x00118db0 | 8 | base-stub | folded |
| - | *(C_actor base)* | `NetDirect__12C_actor_shotFUi` | 0x00119070 | 8 | base-stub | folded |
| - | *(C_actor base)* | `SaveGameGetSize__12C_actor_shotFv` | 0x001191c0 | 8 | base-stub | folded |
| - | *(C_actor base)* | `SaveGameSave__12C_actor_shotFPPv` | 0x001191d0 | 8 | base-stub | folded |
| - | *(C_actor base)* | `SaveGameLoad__12C_actor_shotFPPv` | 0x001191e0 | 8 | base-stub | folded |

¹ No standalone PS2 twin: GCC carries the free flag inside `__dt__12C_actor_shotFv`.

**Two vtable-slot overrides live outside this TU**, in the COMDAT band at
`0x0044a650`–`0x0044b5c0`:

| PC VA | PC name | Slot | `.rdata` | Role |
|---|---|---|---|---|
| 0x0044a670 | `actor_shot_track_create` | 9 | 0x0063b744 | declares one `0x10`-byte key type (C_actor's default declares `0x5c`) |
| 0x0044a690 | `actor_shot_track_apply_key_effects` | 1 | 0x0063b724 | key bit 0 ⇒ destroy + respawn the muzzle particle; bit 1 alone ⇒ destroy it; neither ⇒ bit `0x2000` calls `actor_shot_set_weapon`, the countdown re-arms, and bits 12 / 15 are copied into `enabled` / `use_frame_dir` ¹ |

¹ So **a recorded clip can change this actor's weapon, and switch its firing and its aiming
mode, mid-clip**.

The band is the **`C_actor_track`** interface: the 13-virtual base (slots **0–12**) every
`C_actor`-family class inherits, whose job is to be a client of the `C_recorder` keyframe-track
system; slots 13/14/15 are `C_actor`'s own `AI` / `Update` / `NetDirect`
(`structs/actor_track.md` · `structs/recorder.md`). Both overrides sit in the band rather than in
this TU because they are defined inline in a header, so the linker groups them as COMDATs — which
is why emission order says nothing about them.

---

## 3. What `actor_shot_update` does

```
if (!enabled || !frame)          return;
if (interval_ms >= 0)          { interval_ms -= dt; return; }   // still counting down
interval_ms = 0;
for (p = &g_game_item_table[weapon_id].phases; p->type != 3; p++)
    if (p->type != 1) interval_ms += p->duration_u16;           // the weapon's own cadence
... drive the emitter, spawn the muzzle particle, game_new_shoot ...
```

The phase table is a stride-8 `{u32 type, u16 duration, u16 pad}` run terminated by
`type == 3`; `type == 1` phases are skipped. So the fire interval is **derived from the
weapon**, which is why the class needs no authored rate.

---

## 4. PC vs PS2

**The firing algorithm is a straight port** — it is byte-exact on both builds, including the
terminator `3` and the skip-`type == 1` rule. The key-replay override of §2 is PC-only, because the
PS2 has no `C_recorder`: PS2 `Update__12C_actor_shotFUi` reads the `use_frame_dir` twin at `+0x94`
and nothing on the PS2 writes it. The mechanical differences:

| | PS2 | PC |
|---|---|---|
| `g_game_item_table` record stride | 0xf0 | 0xbc |
| phase-array offset inside the record | +0xc0 | +0x8c |
| the class's own fields | `+0x80` emitter · `+0x84` weapon · `+0x88` countdown · `+0x8c` enabled · `+0x94` use_frame_dir | each **0x10 lower** |
| the inherited frame pointer | `+0x70` | `+0x68` — **0x08** lower, the `C_actor` pair of `structs/actor.md` §6 |

The `C_actor` base itself has no single delta (`structs/actor.md` §6); the subclass's own fields
simply start 0x10 lower on the PC.

**The pair that identified the whole class** — and, in the same step, two `C_actor` methods:

```
PS2  GameInit__12C_actor_shotFv : GameInit__7C_actorFv() ; ... ; SetWeapon(this, 0)
PC   actor_shot_game_init       : actor_game_init()      ; ... ; actor_shot_set_weapon(0)

PS2  GameDone__12C_actor_shotFv : GameDone__7C_actorFv() ; release +0x80 ; +0x80 = 0
PC   actor_shot_game_done       : actor_game_done()      ; release +0x70 ; +0x70 = 0
```

The second pair is what proves that `actor_game_done` is `GameDone__7C_actorFv`: a
statement-aligned twin audits every callee at every position, and here the audited callee is in a
different class.

---

## 5. Related

 `structs/actor.md` (the derived root, the `S_posscoll_frame` table, the field
pairs) · `structs/actor_track.md` (the base class and its 13 vtable slots) ·
`structs/recorder.md` (the `C_recorder` keyframe system this class is a client of) ·
`structs/def_tables.md` (`predmety.def`, the item table `weapon_id` indexes).
