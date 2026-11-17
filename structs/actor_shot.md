# `C_actor_shot` — the timed weapon-fire actor

**Created** 2026-08-04 12:05 +0200 · **Modified** 2026-10-29 23:02 +0100

A `C_actor` subclass that **fires a weapon on a timer**: a placed scene object which, every N
milliseconds, plays the weapon's fire sound, flashes a shoot light and either fires a shot along
its frame or spawns a particle effect. N is not authored — it is read out of the selected weapon's
own phase table, so the object fires at that weapon's natural cadence (§4). Where the instances
come from is §5.

| | |
|---|---|
| PC prefix | `actor_shot_` |
| PC kind (`+0x10`) | `0x21`, stamped by `actor_shot_ctor` ¹ |
| PC vtable | `0x0063b720`, 39 slots: 0–12 the `C_actor_track` interface, 13–38 the `C_actor` slots (§3) |
| PC base class | `C_actor` (`structs/actor.md`) |
| PC instance size | `0x8c` (`operator_new(0x8c)` in `mission_create_actor` case `0x21`) |
| PC constructor | `actor_shot_ctor` @ `0x0044e9a0` |
| PC translation unit | `[0x0044e970, 0x0044ef20)` (§6) |
| PS2 class | `C_actor_shot` (tag `12C_actor_shot`), 12 methods: 7 CONFIRMED twins and 5 base stubs, all mapped in §3 |

¹ Byte-proven from the ctor, and corroborated from the other side: `actor_game_init`
special-cases exactly `kind != 1 && kind != 0x21` in two places, so the base class
knows about this subclass.

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band follows the `C_actor` base; offsets are PC.

```c
typedef struct C_actor_shot {               // 0x8c bytes; PC offsets
    /* +0x00 */ C_actor    base;                // [H] the C_actor band (structs/actor.md §1)
    /* +0x70 */ I3D_sound *fire_sound;          // [H] the weapon's fire sound, driver factory type 4
    /* +0x74 */ int32_t    weapon_id;           // [H] index into g_game_item_table; -1 until set
    /* +0x78 */ int32_t    interval_ms;         // [H] countdown to the next shot; -1 = re-arm now
    /* +0x7c */ uint8_t    enabled;             // [H] firing gate; update returns early when 0
    /* +0x80 */ int32_t    shot_effect_id;      // [H] effect id from the frame's +0x0c string
    /* +0x84 */ uint8_t    use_frame_dir;       // [H] 1 = fire a shot along the frame forward
    /* +0x88 */ int32_t    particle_handle;     // [H] the key-replay particle; -1 = none
} C_actor_shot;
```

## 2. Field evidence

Measured by a census of every `this+N` access in the `actor_shot_*` methods, plus the readers
named in the notes.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | base | H | `actor_shot_ctor` (through `actor_ctor`; kind `+0x10` = `0x21`), `actor_shot_game_init` (`save_excluded` `+0x65` = 1) | `folded_get_dword_0x10` and `actor_game_init` (the kind); the methods of §3 (the frame `+0x68`); `actor_shot_track_apply_key_effects` (the track `+0x08`) | the inherited band; the kind is the engine's stand-in for RTTI, which the PC build lacks |
| +0x70 | fire_sound | H | `actor_shot_game_init` (creates it), `actor_shot_game_done` (releases it, = 0) | `actor_shot_update`, `actor_shot_set_weapon`, `actor_shot_game_done` | created through the driver's factory slot with type 4 and parented to the frame at the origin; `actor_shot_set_weapon` rebuilds it with `sound_table_construct_sound` from the item record's `+0x54`, and every firing cycle plays it ¹ |
| +0x74 | weapon_id | H | `actor_shot_game_init` (= -1), `actor_shot_set_weapon` | `actor_shot_update`, `actor_shot_set_weapon` (the change guard) | indexes `g_game_item_table`, stride `0xbc`. The -1 store makes the `actor_shot_set_weapon(0)` that follows it a real change |
| +0x78 | interval_ms | H | `actor_shot_game_init` (= -1), `actor_shot_update`, `actor_shot_track_apply_key_effects` (= -1) | `actor_shot_update` | re-armed from the weapon's phase table, not from an authored value (§4) |
| +0x7c | enabled | H | `actor_shot_game_init` (= 0), `actor_shot_track_apply_key_effects` (key flag bit 12) | `actor_shot_update` | - |
| +0x80 | shot_effect_id | H | `actor_shot_set_weapon` | `actor_shot_update` | `atoi` of the string at the frame's `+0x0c`, plus 1 when the night-mission byte `C_game+0x2d44` is set (`structs/game.md`). Passed to `game_new_shoot` with a shot, or to `particle_manager_create_particle` when no shot is fired and the id is positive (§4) |
| +0x84 | use_frame_dir | H | `actor_shot_track_apply_key_effects` (key flag bit 15) | `actor_shot_update` | selects a shot along the frame's forward vector over the effect-only branch. Its only writer is the replay of a recorded key (§3c), so outside playback it keeps its initial value |
| +0x88 | particle_handle | H | `actor_shot_init` (= -1), `actor_shot_track_apply_key_effects` | `actor_shot_track_apply_key_effects`, `actor_shot_game_done` | the key replay creates it with `particle_manager_create_particle`, and every holder frees it with `particle_manager_destroy_particle` |

¹ The type name is the PS2 signature's: `ConstructSound__13C_sound_tableFP9I3D_soundUi` takes the
same object (§7).

## 3. Method family

Coverage: all **12** PS2 `12C_actor_shot` methods are in the tables below — **7 CONFIRMED twins**
and **5 base stubs**, each stub on the PC stub its vtable slot holds. The scalar-deleting
destructor, the two `C_actor_track` overrides of §3c and the two track folds it shares with
`C_box` have no PS2 twin; the CRT pair of §3a belongs to the translation unit.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x0044e970 | `crt_static_init_guard_0044e970` | the translation unit | - | - | - | the unit's `.CRT$XCU` dynamic initializer: sets bit 0 of `g_crt_static_init_flags` and calls the registrar below |
| 0x0044e990 | `crt_atexit_register_0044e990` | the translation unit | - | - | - | registers the shared do-nothing unit destructor `crt_atexit_noop_dtor_0060b700` with `crt_atexit` |
| 0x0044e9a0 | `actor_shot_ctor` | instance | `__ct__12C_actor_shotFv` | 0x00118cc0 | CONFIRMED | `actor_ctor`, vtable `0x0063b720`, kind `0x21` |
| 0x0044e9c0 | `actor_shot_scalar_deleting_dtor` | instance | - | - | - | MSVC scalar-deleting destructor around `actor_shot_dtor`, vtable slot 16 ¹ |
| 0x0044e9e0 | `actor_shot_dtor` | instance | `__dt__12C_actor_shotFv` | 0x00118d00 | CONFIRMED | tail-calls `actor_dtor`; the class owns no heap state of its own |

¹ No standalone PS2 twin: GCC carries the free flag inside `__dt__12C_actor_shotFv`.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0044a670 | `actor_shot_track_create` | - | - | - | vtable slot 9, the `C_actor_track` key-type declaration: one `0x10`-byte key type (the `C_actor` default declares `0x5c`) ¹ |
| 0x0044a690 | `actor_shot_track_apply_key_effects` | - | - | - | vtable slot 1, the key replay: key bit 0 destroys and respawns the particle `+0x88`; bit 1 alone destroys it; with neither, bit `0x2000` calls `actor_shot_set_weapon`, the countdown re-arms, and bits 12 and 15 are copied into `enabled` and `use_frame_dir` ¹ ² |
| 0x0044e9f0 | `actor_shot_init` | `Init__12C_actor_shotFP9I3D_frame` | 0x00118d60 | CONFIRMED | `actor_init`, then `particle_handle = -1`; 0 on a null or failed frame bind |
| 0x0044ea30 | `actor_shot_update` | `Update__12C_actor_shotFUi` | 0x00118dc0 | CONFIRMED | vtable slot 14, the timed-fire tick (§4) |
| 0x0044ed90 | `actor_shot_game_init` | `GameInit__12C_actor_shotFv` | 0x00119080 | CONFIRMED | `actor_game_init`; `enabled = 0`, `weapon_id = -1`, `interval_ms = -1`; creates the fire sound; `actor_shot_set_weapon(0)`; excludes the actor from the savegame (`C_actor+0x65`) |
| 0x0044ee50 | `actor_shot_game_done` | `GameDone__12C_actor_shotFv` | 0x00119160 | CONFIRMED | `actor_game_done`, releases the fire sound, `particle_manager_destroy_particle` |
| 0x0044ee90 | `actor_shot_set_weapon` | `SetWeapon__12C_actor_shotFi` | 0x001191f0 | CONFIRMED | on a change: stores `weapon_id`, rebuilds the fire sound from `g_game_item_table`, and reads `shot_effect_id` from the frame |
| 0x0045f0f0 | `stub_zero_vec3_out` | - | - | - | vtable slot 4 of two vtables (`0x0063b720` `C_actor_shot` and `0x0063b980` `C_box`): an ICF fold of two classes' identical `C_actor_track` overrides, owned by neither; it sits in the `C_box` unit (`structs/box.md` §3c, `structs/actor_track.md` §4b) |
| 0x0045f110 | `stub_zero_quat_out` | - | - | - | vtable slot 5 of the same two vtables, the same fold |
| 0x00461180 | `stub_noop_args4_00461180` | `AI__12C_actor_shotFUi` · `NetDirect__12C_actor_shotFUi` | 0x00118db0 · 0x00119070 | base-stub | slots 13 and 15 — a **2 → 1 fold** onto the shared one-argument no-op; both PS2 bodies are a bare `return` |
| 0x00462330 | `stub_rettrue_args4_00462330` | `SaveGameSave__12C_actor_shotFPPv` · `SaveGameLoad__12C_actor_shotFPPv` | 0x001191d0 · 0x001191e0 | base-stub | slots 23 and 24 — a **2 → 1 fold**; returns true, as both PS2 bodies return 1. The class saves nothing |
| 0x00462420 | `stub_ret0_cdecl_00462420` | `SaveGameGetSize__12C_actor_shotFv` | 0x001191c0 | base-stub | slot 22; returns 0, as the PS2 body does — the savegame record is empty |
| 0x004776f0 | `stub_noop_args4_004776f0` | - | - | - | vtable slot 2, the `C_actor_track` pose playback at the clock: an empty override, so a recorded clip moves no frame of this actor; a generic no-op shared by 87 vtables |
| 0x00477710 | `stub_rettrue_args4_00477710` | - | - | - | vtable slot 3, the pose playback at a given time: returns true and applies nothing; shared by 17 vtables |

¹ Both overrides live outside this unit, in the COMDAT band `[0x0044a650, 0x0044b5c0)` of the
`C_actor_track` interface, the 13-virtual base (slots 0–12) every `C_actor`-family class
inherits and whose job is to be a client of the `C_recorder` keyframe-track system
(`structs/actor_track.md`, `structs/recorder.md`). They are defined inline in a header, so the
linker groups them as COMDATs and emission order says nothing about them.
² So **a recorded clip can change this actor's weapon, and switch its firing and its aiming
mode, mid-clip**.

The other slots of the vtable are `C_actor`'s: `actor_release` (17), `actor_get_actor_speed` (28),
`actor_get_bound_points_2d` (34) and the base stubs of `structs/actor.md` §3c.

### 3d. Free helpers

The translation unit holds no free helper.

## 4. The timed-fire tick

`actor_shot_update`:

```
if (!enabled || !frame)          return;
if (interval_ms >= 0)          { interval_ms -= dt; return; }   // still counting down
interval_ms = 0;
for (p = &g_game_item_table[weapon_id].phases; p->type != 3; p++)
    if (p->type != 1) interval_ms += p->duration_u16;           // the weapon's own cadence
play fire_sound; game_add_shoot_light(frame position, 7.0, 50, 0);
if (use_frame_dir)              game_new_shoot(..., frame forward, ..., shot_effect_id, frame, ...);
else if (shot_effect_id > 0)    particle_manager_create_particle(shot_effect_id, origin, +Z, ...);
```

The phase table is a stride-8 `{u32 type, u16 duration, u16 pad}` run at the item record's `+0x8c`,
terminated by `type == 3`; `type == 1` phases are skipped. So the fire interval is **derived from
the weapon**, which is why the class needs no authored rate. A shot is fired only when
`use_frame_dir` is set; otherwise each cycle spawns the effect alone.

## 5. Where instances come from

The only constructor call is `mission_create_actor` case `0x21` (`operator_new(0x8c)`). Of that
factory's eight callers, two take the kind from shipped content: `mission_open_bin` from a scene's
`scene2.bin` object definition and `game_load_differences` from a `.chg` difference file
(`structs/chunk.md` §4b); the savegame restore and the script's actor duplication copy the kind of
an existing actor, and the garage and free-ride setups pass literal kinds. **No shipped scene
places one** — all 79 `scene2.bin` files carry 13 631 object definitions (chunk `0x4c53` →
`0xAE20` → `0xAE21` → kind in `0xAE22`) and none has kind `0x21` — **but 38 of the 92 `.chg` files
do**: 77 actor records (chunk 200) and 45 frame bindings (chunk 400) of kind `0x21`, created when a
script loads that difference file with `loaddifferences`.

## 6. The translation unit

`[0x0044e970, 0x0044ef20)`: it opens with its CRT static-init pair
(`crt_static_init_guard_0044e970` / `crt_atexit_register_0044e990`), holds the eight
`actor_shot_*` functions `0x0044e9a0`–`0x0044ee90` and nothing else, and the next unit's guard,
`crt_static_init_guard_0044ef20`, closes it; that unit is `C_airplane` (`airplane_ctor` at
`0x0044ef50`). The previous unit is `C_actor.cpp` (`structs/actor.md` §6), whose COMDAT band holds
this class's two track overrides.

## 7. PC vs PS2

**The firing algorithm is a straight port** — it is byte-exact on both builds, including the
terminator `3` and the skip-`type == 1` rule. The key-replay override of §3c is PC-only, because
the PS2 has no `C_recorder`: PS2 `Update__12C_actor_shotFUi` reads the `use_frame_dir` twin at
`+0x94` and nothing on the PS2 writes it. The mechanical differences:

| | PS2 | PC |
|---|---|---|
| `g_game_item_table` record stride | 0xf0 | 0xbc |
| phase-array offset inside the record | +0xc0 | +0x8c |
| the class's own fields | `+0x80` fire sound · `+0x84` weapon · `+0x88` countdown · `+0x8c` enabled · `+0x90` shot effect · `+0x94` use_frame_dir | each **0x10 lower** |
| the inherited frame pointer | `+0x70` | `+0x68` — **0x08** lower, the `C_actor` pair of `structs/actor.md` §8 |

The `C_actor` base itself has no single delta (`structs/actor.md` §8); the subclass's own fields
simply start 0x10 lower on the PC. `SetWeapon__12C_actor_shotFi` pairs `+0x90` ↔ PC `+0x80`
statement for statement — the same `atoi` of the frame string and the same night-mission `+1`
(PS2 `C_game+0x1190`) — and it passes `+0x80` to `ConstructSound__13C_sound_tableFP9I3D_soundUi`,
which is what types the fire sound.

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

**The base stubs.** PS2 overrides `AI`, `NetDirect` and the three `SaveGame*` virtuals with 8-byte
bodies; the PC `C_actor_shot` vtable overrides the same slots (13, 15, 22, 23, 24) with shared
stubs whose return values match, so the class differs from the `C_actor` base in exactly those
slots on both builds.

**Method sizes** (bytes, PS2): `__ct__` 60, `__dt__` 96, `Init` 76, `Update` 680, `GameInit` 224,
`GameDone` 88, `SetWeapon` 204; each base stub 8.

## 8. Related

- `structs/actor.md` — the base class, the `S_posscoll_frame` table and the field pairs.
- `structs/actor_track.md` — the base interface and its 13 vtable slots; `structs/recorder.md` —
  the `C_recorder` keyframe system this class is a client of.
- `structs/def_tables.md` — `predmety.def`, the item table `weapon_id` indexes.
- `structs/sound_table.md` — `sound_table_construct_sound`, which builds the fire sound.

## 9. Open items

1. **The frame string at `+0x0c`.** `shot_effect_id` is parsed from a string the frame carries at
   `+0x0c`; which field of `I3D_frame` that is has not been decoded (`structs/i3d_frame.md`).
