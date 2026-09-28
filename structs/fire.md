# `C_fire` — the timed fire effect

**Created** 2026-09-04 00:21 +0200 · **Modified** 2026-09-17 13:43 +0200

PC prefix **`fire_`**. Hosted by the `tools.cpp` translation unit.

A self-retiring fire: a particle effect plus, above a lifetime threshold, a looping sound,
both bound to a parent `I3D_frame`. It counts its own lifetime down and tears itself down
when it expires. All four methods are mapped.

---

## 1. What is measured

```c
typedef struct C_fire {
/* +0x00 */ S_vector      pos;             // [H] world position, copied by fire_init
/* +0x0c */ I3D_frame    *parent;          // [H] the frame the particle and sound bind to
/* +0x10 */ int           lifetime_ms;     // [H] counted down by fire_tick; < 0 = expired
/* +0x14 */ float         radius_sq;       // [H] fire_init stores radius * radius
/* +0x18 */ float         f18;             // [M] a caller-supplied float, no decoded reader
/* +0x1c */ uint          reemit_accum_ms; // [H] accumulates dt; reset on each re-emit
/* +0x20 */ uint          reemit_period_ms;// [H] ctor 200; fire_init rand_range() + 450
/* +0x24 */ void         *sound;           // [H] driver object type 4, only above 2000 ms
/* +0x28 */ char          flag28;          // [M] a caller-supplied bool
/* +0x2c */ int           particle_id;     // [H] particle effect type 0x15; -1 = none
} C_fire;
```

**The lifetime threshold is a real gameplay gate.** `fire_init` only allocates the looping
sound when the requested lifetime is **2000 ms or more** (`1999 < lifetime`), so a short-lived
fire is silent by construction.

**The re-emit period is randomised per fire.** The constructor seeds `200`, then `fire_init`
overwrites it with `rand_range() + 450` ms — so two fires lit at the same instant drift apart
instead of pulsing in lockstep.

---

## 2. PC coverage — 4 of 4

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x0061b650 | `fire_ctor` | `__ct__6C_fireFv` | 0x003ad670 | CONFIRMED | zero the re-emit accumulator, seed the period at 200 ms |
| 0x0061b670 | `fire_dtor` | `__dt__6C_fireFv` | 0x003ad690 | CONFIRMED | release the looping sound if there is one |
| 0x0061b6a0 | `fire_init` | `Init__6C_fireFR8S_vectorP9I3D_frameiffb` | 0x003ad730 | CONFIRMED | store the placement, spawn particle type `0x15`, and the sound above 2000 ms |
| 0x0061b850 | `fire_tick` | `Tick__6C_fireFUi` | 0x003ad9c0 | CONFIRMED | count down, fade the sound over the last 2000 ms, tear down on expiry |

`fire_init`'s arity matches the PS2 mangled signature exactly:
`(S_vector &, I3D_frame *, int, float, float, bool)`.

---

## 3. Consumers

* **`game_new_fire`** — the factory. It allocates `0x30` bytes, calls `fire_ctor` and then
  `fire_init`, adds a shoot light at the position and appends the fire to the game's fire
  vector.
* Nothing else in the `[0x00617a90, 0x0061c3f0]` range touches the class.

---

## 4. PC vs PS2

* The PS2 record is **4 bytes wider from `+0x0c` on**, because its `S_vector` is 16 bytes
  and the PC's is 12. PS2 offsets: parent `+0x10`, lifetime `+0x14`, radius² `+0x18`,
  accumulator `+0x20`, period `+0x24`, sound `+0x28`, flag `+0x2c`, particle `+0x30`.
  **Do not carry a PS2 offset into a PC expression.**
* `fire_init` on the PS2 transforms the incoming position by the parent frame's rotation
  before storing it; the PC does the equivalent inside the sound-placement block instead.
  A factoring difference, not a behavioural one.
* `rnd__Fi(100)` on the PS2 against `rand_range()` on the PC — the same `+ 0x1c2` (450)
  offset on both, which is the shared constant that pins the pair.
* Everything else is statement-for-statement: the same `radius * radius`, the same particle
  effect type `0x15`, the same 2000 ms sound gate, the same driver object type 4, the same
  three-call sound teardown with `(0,1)` then `(0,0)`.

---

## 5. Related

`structs/game.md` (`game_new_fire`) · `structs/particle.md` (effect type `0x15`) ·
`structs/def_tables.md` (`particles.def`, which names the effect).

---

## 6. Open items

* **`+0x18` and `+0x28` are [M]** — `fire_init` stores a caller-supplied float and a
  caller-supplied bool and nothing in this cluster reads either. Decoding
  `game_new_fire`'s callers would name both.
* **What lights a fire** is untraced beyond `game_new_fire`. Molotovs (`C_grenade` carries
  a second payload flag, `structs/grenade.md`), car explosions and scripted set-pieces are
  the candidates.
* **In game, untested:** a fire lit with a lifetime under 2 seconds should be **silent**, and
  every fire's flame should re-emit on its own slightly-random ~0.45-0.7 s cycle rather than in
  sync with its neighbours.
* **In game, untested:** a fire's sound should fade out over its last two seconds rather than
  cutting.
