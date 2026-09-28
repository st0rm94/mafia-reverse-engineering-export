# Cheat-code system — *Mafia: The City of Lost Heaven* (Game.exe, v1.0)

**Created** 2026-06-12 19:40 +0200 · **Modified** 2026-09-23 15:58 +0200

The typed cheat-code subsystem. Entry point is `game_cheat_check` (VA `0x005e3a90`). All
offsets/VAs are v1.0 (PE image base `0x00400000`).

---

## 1. How the cheat checker works

`game_cheat_check(this, typed_char)` is called once per frame from `engine_game_loop` (and from
`race_game_loop`), right after `input_update`, when the second `g_engine` vtable `+0x98` state
check (`state_recheck`) is true. `this` is the active screen/UI object; `typed_char` is the
latest key.

Per call:

1. **Accumulate** — the typed char is upper-cased (`a..z → A..Z`) and appended to
   a rolling text buffer at `this+0x2ef4`. The buffer is cleared once it grows
   past 30 chars.
2. **Obfuscated storage** — each char is stored **bitwise-complemented (`~c`)**,
   NUL-terminated by a plain `0x00`. The cheat-code table is stored the same way,
   so the codes never appear as plaintext in the binary (anti-extraction trick).
3. **Match** — the buffer tail is compared against each table entry. The code
   table is walked **backwards** in `0x24`-byte (36-byte) strides from
   `s_hole3_i3d_00650a08` down to `&g_game_cheat_code_table` (BOXER) — **8 entries** (§3).
4. **Dispatch** — on a full-length match the entry's **cheat id** is read from the
   parallel id-table (`id = *(int*)(slot*0x24 + 0x650908)`). `id == -1` = inactive
   slot. For `id >= 0`: `hud_fade_set(0,500,0)` (acknowledgement black flash) fires,
   the buffer is cleared, and the effect is selected by `switch(id)`. Returns
   `true` if a cheat fired, else `false`.

### De-obfuscation recipe (Python)
```python
# code strings: chars are ~complemented, NUL-terminated.
def read_code(va):           # va = code string VA in .data (0x6508e8 + n*0x24)
    o = va2off(va)
    out = []
    while data[o] != 0:      # raw 0x00 terminates
        out.append(chr((~data[o]) & 0xff)); o += 1
    return ''.join(out)
```

---

## 2. The 8 cheat codes (v1.0)

Codes are **case-insensitive** (the checker upper-cases input). Type the letters
with no spaces. Order below is the in-memory table order (slot 0 = BOXER … slot 7
= ROZMAZ).

| Code (keys to type) | id | `switch` case | Effect | Source of truth |
|---|---|---|---|---|
| **`BOXER`** | `4` | `case 0x4` | **Big guns** — doubles the scale of the player's `r_hand`/`l_hand` frames, so the hands and the weapon parented to them render oversized. Toggles flag bit `0x10`. | in-game + decomp |
| **`PARNIK`** | `0xb` (11) | `case 0xb` | **Titanic easter egg** — spawns the player model (`Tommy.i3d`, retextured **`Leonardo`**) plus an old-lady model (`czena06.i3d`, retextured **`Kate`**) in the bow pose on the moving city ships, parodying the *Titanic* "I'm flying" scene. Toggles flag bit `0x80`. | in-game + decomp ¹ |
| **`JIZDA`** | `0xc` (12) | `case 0xc` | **Removes the convertible roof** — on a roadster-type car, hides every model frame tagged `[DEFORM_ROOF…]`. | in-game + decomp |
| **`KREFJEZIVOT`** | `0xd` (13) | `case 0xd` | **Heal** — sets player health to 100 and copies 5 entity dwords `+0x604..+0x614` → `+0x644..+0x654`. Gated by the `this+0x3664` one-shot flag. | decomp ² |
| **`MUNICAK`** | `0xe` (14) | `case 0xe` | **Infinite ammo** — toggles flag bit `0x100` in `this+0x2f14` ³. | in-game + decomp ⁴ |
| **`CORNUFSE`** | `7` | - | **No effect in v1.0.** In **v1.2** it lets the player steal any car without learning the steal mini-progression first. | in-game (v1.2) |
| **`POLIFUCK`** | `8` | - | **No effect in v1.0.** | decomp |
| **`ROZMAZ`** | `0xf` (15) | - | **No effect in v1.0.** CZ *rozmaz* = blur/smear. | decomp |

¹ Model and texture names byte-verified. ² CZ "krev je život" = *blood is life*.
³ While set, the **reserve** ammo pool (the `y` in the HUD's `x/y` readout = rounds outside the
current magazine) is **not decremented on reload**. Only effective when `y != 0`.
⁴ CZ *munice* = ammo.

### The id ↔ case mapping
- The `switch` in `game_cheat_check` implements cases **4, 0xb, 0xc, 0xd, 0xe** — all 5
  map to a real v1.0 code (BOXER, PARNIK, JIZDA, KREFJEZIVOT, MUNICAK).
- Ids **7, 8, 0xf** (CORNUFSE / POLIFUCK / ROZMAZ) have **no case** → they match
  and `return true` (triggering the generic post-cheat refresh in the caller) but
  carry no in-function effect in v1.0. Consistent with CORNUFSE being a v1.2
  feature.

---

## 3. Data layout (`.data`)

The table has 8 entries at a 36-byte (`0x24`) stride; the code strings occupy
`0x6508e8 … 0x6509e4` (slot 0 = `BOXER` at `0x6508e8`, slot 7 = `ROZMAZ` at
`0x6509e4`). The id (`int`) for slot *n* lives at `0x650908 + n*0x24`.

| Field | Address (slot n) | Notes |
|---|---|---|
| code string | `0x6508e8 + n*0x24` | `~`-complemented chars, NUL-terminated |
| cheat id (`int`) | `0x650908 + n*0x24` | `-1` = inactive |

- Loop start anchor: `s_hole3_i3d_00650a08` (the `hole1/2/3.i3d` strings above the
  table are unrelated neighbours; the walk pre-decrements `0x24` so the first
  string compared is slot 7 / `ROZMAZ` at `0x6509e4`).
- The walk ends at slot 0 (`BOXER`, `&g_game_cheat_code_table`): that entry is **compared
  first, then** — if it didn't match — its address doubles as the loop's stop
  condition (`pcVar3 == &g_game_cheat_code_table → return false`). It is **not** a dead
  sentinel; it is the BOXER cheat, so there are 8 cheats, not 7.
- Shared empty-string used to clear the buffer: `g_empty_string`.

---

## 4. Handler functions

### 4.1 `bridge_set_steamboat_figures` (`0x004624e0`) — PARNIK / case 0xb

- `__thiscall(C_bridge *this, char enable)`. It is a **`C_bridge` method**: `game_cheat_check`
  walks the C_game entity list and tests `cmp [ecx+0x10], 0x14` (kind == C_bridge) before
  calling, and the body only acts when `bridge_index` (`+0x140`) is non-zero. The steamboat
  node table it decorates is loaded by `bridge_load_data` from `tables\parnik.def`. See
  `structs/bridge.md`.
- Iterates the global ship-figure bank at `g_steamboat_slot_figure_a` (stride 9 dwords).
  For each slot, when `enable != 0` it:
  - loads `Tommy.i3d` into slot[0], applies texture `Leonardo`, parents it to the
    ship anchor frame (`slot[-2]`), positions it (`x=0, y≈4.7, z≈37.0`);
  - loads `czena06.i3d` into slot[1], applies texture `Kate`, parents + positions
    it (`z≈37.4`) — the second bow figure.
  - When `enable == 0` it releases both models (vtable dtor) and zeroes the slots.
- The Leonardo/Kate figure pair on the city ships is the *Titanic* bow-scene easter egg.

### 4.2 `car_set_roof_visible` (`0x00472de0`) — JIZDA / case 0xc

- `__fastcall(I3D_model *this, char hide)` → returns `1` if matching frames found.
- `game_cheat_check` calls it only when the player (`this+0xe4`) sits in a vehicle
  (`player+0x98`) of kind 4.
- Enumerates the vehicle model's frames (`this` vtable `+0x34`, filter `"*"`),
  using the predicate `i3d_frame_match_deform_roof` to collect frames whose bracket-tag is
  `DEFORM_ROOF`, then sets each collected frame's visibility to `hide == 0` — puts on / takes
  off the convertible roof.

### 4.3 `i3d_frame_match_deform_roof` (`0x00473f90`) — roof predicate

- Frame-enumeration callback. Parses the frame name for a `[ … ]` tag and,
  when present and ≥ length, `strncmp`s it against `s_DEFORM_ROOF_0064b878`
  (`"DEFORM_ROOF"`, 0xb chars); on a match, appends the frame to the output
  vector (`param_2`). Its consumer is `car_set_roof_visible`.

### 4.4 `player_set_hand_scale` (`0x004cda20`) — BOXER / case 4

- `__fastcall(actor *this, bool big)`, reached from `game_cheat_check` `case 4` with flag bit
  `0x10` of `this+0x2f14` as the bool, only when the player pointer (`+0xe4`) is set.
- Looks up the player model's `r_hand` (`s_r_hand_0064bcf8`) and `l_hand`
  (`s_l_hand_0064bcf0`) frames via the model accessor (`this+0x68` vtable `+0x38`, search
  depth `0xffff`).
- Builds a scale vector of `1.0f,1.0f,1.0f`, overwritten with `2.0f,2.0f,2.0f` when the bool
  (at `[esp+0x10]`) is set, and stores it at `frame+0x90..+0x98` — the local **scale** — on
  each hand frame. It seeds the orientation matrix if absent (`S_quat::Make`), rewrites the
  flags at `frame+0xac` (`& 0xfffffecb | 0x40000008`), then commits (vtable `+0x18`).
- Ghidra's decompilation does not show the bool or the scale vector (they reach the stores
  through registers it does not track); the disassembly does.
- **In-game effect:** the player's hands — and the weapon parented to them, which inherits the
  scale — render enlarged ("big guns").

---

## 5. Cross-references
- `game_cheat_check` docblock: in `ghidra/Game-ghidra.exe.annotated.c` (anchor: the
  function `game_cheat_check`, VA `0x005e3a90`).
- Main-loop call site: `basic/artifacts/game-loop.md` (`state_recheck` / `screen_obj`).
- `structs/bridge.md` — the `C_bridge` receiver of the PARNIK handler.

## 6. Open items
- `POLIFUCK` (`8`) and `ROZMAZ` (`0xf`) — no handler in v1.0; whether v1.2 adds `switch`
  cases (as it does for `CORNUFSE`) is unchecked.
- The layout of the rest of each `0x24`-byte table entry (bytes other than the code string and
  the id) is undocumented.
