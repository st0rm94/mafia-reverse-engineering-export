# The input manager — `g_input` @ `0x006f92e0`

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-10-04 23:41 +0200

PC prefix **`input_`**. Translation unit `[0x005d14a0, 0x005d3a60)`: 21 starts, all named.

The PC's keyboard / mouse / joystick binding layer. It owns the 60 rebindable **actions**
the Controls screen exposes, resolves them into two per-action float tables once a frame,
and hands those tables to the player controller, the camera, the script VM and the GUI.

---

## 1. Layout

```c
typedef struct g_input {                 /* 0xe8 bytes */
/* +0x00 */ void         *engine;        // [H] g_engine — the LS3D object that owns
                                         //     keyboard + mouse (slot 0xa0 = IsKeyDown)
/* +0x04 */ unsigned char joy_alloc;     // [H] std::vector allocator byte
/* +0x08 */ S_input_joy_state *joy_begin;// [H] one 0x8c record per enumerated joystick
/* +0x0c */ S_input_joy_state *joy_end;
/* +0x10 */ S_input_joy_state *joy_cap;
/* +0x14 */ S_input_binding   capture;   // [H] scratch record filled by input_capture_binding
/* +0x20 */ char          keys[0x40];    // [H] the keyboard scancodes any binding uses
/* +0x60 */ char          key_refs[0x40];// [H] per-slot refcount for the table above
/* +0xa0 */ float         mouse_range_x; // [H] deflection denominator; ctor 20.0f, live 400
/* +0xa4 */ float         mouse_range_y;
/* +0xa8 */ float         mouse_range_z; // (wheel)
/* +0xac */ unsigned char bind_alloc;
/* +0xb0 */ S_input_binding *bind_begin; // [H] g_input_bindings_begin — 60 entries
/* +0xb4 */ S_input_binding *bind_end;   // [H] g_input_bindings_end
/* +0xb8 */ S_input_binding *bind_cap;
/* +0xbc */ unsigned char val_alloc;
/* +0xc0 */ float        *value_begin;   // [H] per-action SAMPLED value, parallel to bind_*
/* +0xc4 */ float        *value_end;
/* +0xc8 */ float        *value_cap;
/* +0xcc */ unsigned char once_alloc;
/* +0xd0 */ float        *oneshot_begin; // [H] per-action ONE-SHOT value
/* +0xd4 */ float        *oneshot_end;
/* +0xd8 */ float        *oneshot_cap;
/* +0xe0 */ unsigned int  prev_keys_lo;  // [H] previous 64-bit key mask, for edge detection
/* +0xe4 */ unsigned int  prev_keys_hi;
} g_input;                               /* ends exactly at 0x006f93c8 */
```

**The size is bounded, not inferred.** `g_joy_device_list_cap` ends at `0x006f92dc`
and `g_i3d_anim_cache` begins at `0x006f93c8`; `0x006f93c8 − 0x006f92e0 = 0xe8`, which is
exactly where `+0xe4` ends. Independently, `0x006f9390 − 0x006f92e0 = 0xb0` — and
`0x006f9390` carries its own Ghidra symbol, so the binding-vector offset is a measurement
and not a count.

**The object has almost no textual existence.** Every method is `__thiscall`, so the receiver
travels in ECX and a text search of the decomp for `0x006f92e0` finds only the static
constructor/destructor thunks (§3). The two members that carry Ghidra symbols (`+0xb0`, `+0xb4`)
are Ghidra rendering two members of a fixed-address singleton as standalone globals.

### 1a. `S_input_binding` — 0xc bytes, 60 of them

```c
typedef struct S_input_binding {
/* +0x00 */ int    mode;         // 0 = whole/digital, 1 = positive half-axis,
                                 //                    2 = negative half-axis
/* +0x04 */ short  device_kind;  // 1 = keyboard, 2 = mouse, 3 = joystick
/* +0x06 */ unsigned short code; // see the encoding table below
/* +0x08 */ int    device_index; // index into joy_begin[]; only meaningful for kind 3
} S_input_binding;
```

The name is a **PC invention**. The *contents* are proven by the shipped strings that
`input_binding_get_label` returns:

| kind | code | what it is |
|---|---|---|
| 1 | DIK scancode | keyboard. `0x2a`/`0x36` Shift, `0x1d`/`0x9d` Ctrl, `0x38`/`0xb8` Alt, `0x01` Escape, `0x0e` Backspace, `0x1c` Return … |
| 2 | `1` / `2` / `4` | mouse button left / right / middle |
| 2 | `0x100` / `0x200` / `0x300` | mouse axis X / Y / Z (wheel) |
| 3 | `< 0x100` | joystick axis index 0..7 — the name comes from the device's vtable slot `0x5c` |
| 3 | `0x100`..`0x1ff` | joystick button `code − 0x100` — device vtable slot `0x64` |
| 3 | `>= 0x200` | POV hat: hat index `(code >> 4) & 0xf`, direction mask `code & 0xf` — device vtable slot `0x6c` |

A `device_index` that no longer names a live joystick renders as the shipped string
**"Not attached joy"**, which is what proves `+0x08` is an index into the `0x8c` vector and
not a handle.

### 1b. `S_input_joy_state` — 0x8c bytes, one per joystick

```c
typedef struct S_input_joy_state {
/* +0x00 */ void *device;          // ijoy.dll device object; addref'd on insert (vtable 1)
/* +0x04 */ S_joy_block cur;       // filled by device vtable slot 0x50
/* +0x48 */ S_joy_block prev;      // filled by device vtable slot 0x54
} S_input_joy_state;               /* 0x04 + 0x44 + 0x44 = 0x8c */

typedef struct S_joy_block {       /* 0x44 bytes */
/* +0x00 */ float axis[8];         // 8 analog axes
/* +0x20 */ float pov[4];          // 4 POV hat angles, in radians
/* +0x30 */ char  pov_centered[4]; // non-zero => that hat is centred
/* +0x34 */ unsigned int buttons[4]; // 128-bit button mask; Game.exe reads only word 0
} S_joy_block;
```

Also a PC invention. The block boundary is exact: `input_update` calls device slot `0x50`
with `record+0x04` and slot `0x54` with `record+0x48`, and `0x48 + 0x44 = 0x8c`.
The POV angle thresholds `input_update` and `input_capture_binding` compare against —
`0.1`, `1.5`, `1.6`, `3.12`, `3.16`, `4.69`, `4.72` — are radians around 0, π/2, π and
3π/2, which is how a hat angle becomes a 4-bit direction mask.

The block is the device state `ijoy.dll` keeps at `device+0x130`: device slot `0x50` copies those
`0x44` bytes out, and the DLL's own button test picks word `index >> 5` of the four for buttons
0..127. `input_update` and `input_capture_binding` read only word 0 (`1 << (code & 0x1f)`, and the
capture loop stops at 32), so a joystick button above 31 cannot be bound.

### 1c. The capture filter

`input_capture_binding` takes two stack arguments (it returns with `ret 8`): a device FILTER and a
poll flag. Each set filter bit skips one input source:

| bit | source skipped |
|---|---|
| `0x01` | mouse axes |
| `0x02` | joystick axes |
| `0x04` | mouse buttons |
| `0x08` | joystick buttons |
| `0x10` | POV hats |
| `0x20` | keyboard |

Its one caller, `gui_key_capture_tick`, passes `0` for an Aiming slot (`0x12` / `0x13`) and `1` for
any other slot, so only the Aiming actions accept a mouse axis. When the screen's byte at `+0x4c` is
set it passes `0x3c` / `0x3d` instead, which leaves only the axes. The decompiler renders the filter
as the return value of an engine call and the parameter as unused, because it loses the stack depth
across the engine's callee-cleans virtual calls; in the disassembly all five reads of the filter sit
at stack depth `0x1c`, where the first argument lives.

---

## 2. The 60 actions

The binding vector is indexed by **action slot 0..0x3b**, and both value tables are
parallel to it. The slot→label map is decoded in `structs/loadsave.md` §7a from the 60-dword
table at `0x0064d668`; the profile stores the bindings themselves as one `0x2d0` block at profile
`+0x1f4`.

Every decoded consumer matches its label exactly:

| slot | textdb label | consumer |
|---|---|---|
| `0x12` / `0x13` | 312 **Aiming** (primary + secondary) | `camera_apply_desired_pos` and `camera_apply_desired_pos_car` call `input_get_action_value(0x13, 0)` / `(0x12, 0)` |
| `0x1e` | 325 **Objectives** | `player_ai` reads the sampled table with a `> 0.25` gate → `loadsave_objectives_display()` |
| `0x3a` | 310 **Change camera** | `player_ai` reads the one-shot table → `camera_set_mode(-1, 1)` |
| `0x01` / `0x26` | 301 **Forwards** / 332 **Steer Left** | `force_feedback_manager_game_init` reads both bindings to find the physical axis for the on-foot and steering effects |
| `0x21`..`0x28`, `0x37` / `0x38` | 330 **Accelerate**, 331 **Brake / Reverse**, 332 **Steer Left**, 333 **Steer Right**, 342 **Use Clutch** (each primary + secondary) | `player_ai_drive` reads the sampled table at `+0x84`..`+0xa0` and `+0xdc`/`+0xe0` and arbitrates each primary/secondary pair (`structs/player.md` §5a) |

**The two tables.** `+0xc0` holds the value sampled from the CURRENT device state;
`+0xd0` holds the one-shot value — for a key it is gated by the auto-repeat counter, for a
mouse button it is a rising-edge latch, for an axis it is simply the same analog value.
`input_get_action_values(out, sel)` and `input_get_action_value(i, sel)` both take the
selector as bit 0. Callers characteristically take **both** tables, one call per selector.

---

## 3. Lifecycle

```
Main.cpp static init -> input_static_ctor()          mov ecx, 0x006f92e0; jmp input_ctor
                     -> input_register_atexit_dtor() crt_atexit(&input_static_dtor)
engine_init      -> input_init_devices(g_engine, g_joy_device_list, count)
loadsave_apply_key_bindings(profile)
                 -> input_clear_bindings()
                 -> input_set_binding(profile + 0x1f4 + i*0xc, i)   x 60
                 -> input_set_mouse_axis_range(400, 400, 400)
engine_game_loop       -> input_update(poll)            once a frame
                 -> input_get_action_values(...)  x2, then the consumers index them
gui_key_capture_ctor -> input_poll_joysticks()    prime, so a stale press is not captured
gui_key_capture_tick -> input_capture_binding()   returns the scratch record at +0x14
at exit          -> input_static_dtor()           mov ecx, 0x006f92e0; jmp input_dtor
                                                  -> input_release_all()
```

**Construction and teardown.** `g_input` is a file-scope object of `Main.cpp`.
`input_static_ctor` (`0x005f8240`) and `input_static_dtor` (`0x005f8260`) reach `input_ctor` and
`input_dtor` only by **`E9` tail jumps**, so a scan for `E8` calls and for the address as an
absolute 4-byte value finds zero references to `0x005d14d0` / `0x005d1560`. `input_release_all` has
exactly one caller, `input_dtor`. The TU's own `.CRT$XCU` guard at `0x005d14a0` sets one bit in
`g_crt_static_init_flags` and registers a no-op `atexit`, with no constructor call.

`input_ctor` zeroes the four vectors and seeds the mouse ranges with 20.0f; `input_init_devices`
initialises both key tables and the previous-key mask; `loadsave_apply_key_bindings` overwrites
the mouse ranges with 400.

---

## 4. Field evidence

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| `+0x00` | `engine` | H | `input_init_devices` | `input_update`, `input_capture_binding`, `input_set_binding` | ¹ |
| `+0x08` | joystick-state vector | H | `input_init_devices`, `input_release_all` | `input_update`, `input_poll_joysticks`, `input_capture_binding`, `input_binding_get_label` | stride `0x8c`; the "Not attached joy" bound check divides by it |
| `+0x14` | capture scratch | H | `input_capture_binding` | `gui_key_capture_tick` | the only use of this field |
| `+0x20` / `+0x60` | key-of-interest table + refcounts | H | `input_set_binding`, `input_clear_bindings`, `input_init_devices` | `input_update` | republished to the engine as `(buf, 0x40)` whenever it changes |
| `+0xa0`..`+0xa8` | mouse axis ranges | H | `input_ctor`, `input_set_mouse_axis_range` | `input_update` | clamp-and-divide denominators; larger = less sensitive |
| `+0xb0` | binding vector | H | `input_set_binding`, `input_clear_bindings` | `input_get_binding`, `input_update`, `engine_game_loop` | `engine_game_loop` derives the action count as `(end − begin)/0xc` |
| `+0xc0` | sampled value table | H | `input_update` | `input_get_action_values`, `input_get_action_value` | selector 0 |
| `+0xd0` | one-shot value table | H | `input_update` | `input_get_action_values`, `input_get_action_value` | selector 1 |
| `+0xe0`/`+0xe4` | previous key mask | H | `input_init_devices` (−1), `input_update` | `input_update` | edge detection for keyboard bindings |

¹ `engine_init` passes `[g_engine]` (disassembled callsite). Engine slots used: `0x2c`/`0x94`/`0xcc`
poll, `0xa4` publish key table, `0xa8` key mask, `0xb8`/`0x98` capture, `0xd0`/`0xd4`/`0xd8` mouse
deltas, `0xe8` key repeat.

---

## 5. PC coverage

| PC VA | PC name | PS2 twin | Status | Role |
|---|---|---|---|---|
| 0x005d14a0 | `crt_static_init_guard_005d14a0` | - | STRUCTURAL | TU-opening guard; contains no ctor call |
| 0x005d14c0 | `crt_atexit_register_005d14c0` | - | STRUCTURAL | registers a no-op dtor |
| 0x005d14d0 | `input_ctor` | - | STRUCTURAL | reached only by `input_static_ctor`'s tail jump |
| 0x005d1560 | `input_dtor` | - | STRUCTURAL | reached only by `input_static_dtor`'s tail jump |
| 0x005d1610 | `input_release_all` | - | STRUCTURAL | release every device, empty every vector |
| 0x005d1740 | `input_set_mouse_axis_range` | - | CONFIRMED | three floats at `+0xa0` |
| 0x005d1770 | `input_init_devices` | - | CONFIRMED | binds `g_engine` + `g_joy_device_list` |
| 0x005d1a70 | `input_clear_bindings` | - | CONFIRMED | the reset half of the profile apply |
| 0x005d1b20 | `input_get_action_values` | - | CONFIRMED | base + count of one value table |
| 0x005d1bc0 | `input_get_action_value` | - | CONFIRMED | one action, one table |
| 0x005d1c30 | `input_set_binding` | - | CONFIRMED | install or clear one action |
| 0x005d1ec0 | `input_get_binding` | - | CONFIRMED | copy one `S_input_binding` out |
| 0x005d1f30 | `input_update` | - | CONFIRMED | the per-frame resolve |
| 0x005d2ad0 | `input_poll_joysticks` | - | CONFIRMED | prime before capture |
| 0x005d2b40 | `input_capture_binding` | - | CONFIRMED | first control pressed that the filter allows → the scratch record (§1c) |
| 0x005d3180 | `input_binding_get_label` | - | CONFIRMED | the shipped label strings |
| 0x005d3990 | `input_joy_states_copy_range` | - | STRUCTURAL | member helper ¹ |
| 0x005d39d0 | `input_joy_states_fill_n` | - | STRUCTURAL | member helper ¹ |
| 0x005d3a10 | `input_joy_states_copy_one` | - | STRUCTURAL | member helper ¹ |
| 0x005d3a30 | `crt_static_init_guard_005d3a30` | - | STRUCTURAL | TU-closing guard |
| 0x005d3a50 | `crt_atexit_register_005d3a50` | - | STRUCTURAL | its `atexit` partner |

¹ **Why the three `0x8c` helpers carry a class prefix while `copy_range_stride_0xc` does not.**
Identical-code folding did happen in this link, so "all callers in one TU" alone cannot prove an
owner. What does: each of the three bodies is **byte-unique** in `.text`; each carries the **`0x8c`
`S_input_joy_state` stride**, this class's record and nothing else's; each has **0 datarefs**; and
the image-wide `E8`/`E9`/dataref census returns 3, 2 and 1 referrers respectively with **no foreign
owner** — nothing was folded into these addresses. `copy_range_stride_0xc` has 42 callsites across
16 owners and keeps its mechanical name, which asserts no owner.

---

## 6. PC vs PS2

**Class-level counterpart: `C_vid` (STRUCTURAL).** `g_input` is constructed by `Main.cpp`'s
initializer block; matching that block against PS2 `__sinit_Main.cpp` object-for-object puts
`g_input` in the same slot as `__ct__5C_vidFv(0x4dc920)`, between the animation cache and the
joystick vector on both builds. `C_vid`'s method set is an action-indexed input manager, as the PC
class is: `Reset`, `Update(ms)`, `ReadKey`, `TestKey(i)`, `GetIcon(i)`, `ControlerIsOK`,
`IsSameAB` (the PS2's A/B button-swap option).

The grade is capped at STRUCTURAL: `__ct__5C_vidFv` and `input_ctor` disagree — the PS2 constructor
zeroes five class-static globals and a `0x40`-byte control array and writes `*this = 0`; the PC one
builds four member vectors and three mouse-range floats and writes `*this = 0`. They are the same
role on different input hardware, so **no method-level twin is expected** and none is stamped.

**No method-level PS2 twin exists for any member of this TU.** Searched:

* the PS2 class column for `key|joy|pad|input|contro|axis|button` — only `LSIInput`,
  `LSIInputDevice`, `GM_OptionsControls`, `GM_TextInput` and one `std::vector<P5C_Joy>` destructor;
* the PS2 free-function list for the same words — only the `scePad*` SDK entry points;
* type names `bind`, `S_key`, `S_control`, `S_input` — nothing;
* the `LSIInput` consumer trace: `LSIInputCreate__FPvUiUiPPvPv` is called from
  `Init__6IGraphFP16IGRAPH_INIT_DESC`, i.e. inside the engine, which on PC lives in `ls3df.dll`.

The PS2 reads its pad through `scePadRead` and `C_vid::ReadKey`/`TestKey`, and ships no
keyboard-scancode name table at all — the PC needs this code because it has a keyboard, a mouse
and DirectInput joysticks. `ijoy.dll` supplies the DEVICE objects while this game-side wrapper
*holds* them (§8).

**`G_ConsoleControls` is a false friend.** That PS2 struct's "Console" means *the games
console*, i.e. the pad — not the on-screen text console documented in `structs/console.md`, which
lives in the very next translation unit. Do not let that word pull this class toward a `console_`
prefix. `G_ConsoleControls` has no methods and appears only as the parameter type of
`C_player::AI_walk`, `AI_drive`, `AI_car_fire` and `GoToCar_Process`. On PC those four take the two
value-table pointers directly — a factoring difference, not a missing function.

---

## 7. Related

`structs/console.md` (the next TU — and the `Console` false friend) · `structs/loadsave.md` §7/§7a
(the profile block and the 60 action labels) · `structs/force_feedback.md` (the only consumer of
`input_get_binding`) · `structs/camera.md` (the Aiming consumers) · `structs/app_bootstrap.md`
(`Main.cpp` static init) · `basic/artifacts/dll.md` (`ijoy.dll`).

---

## 8. Open items

| item | state |
|---|---|
| whether part of the PC-side input code lives in a DLL | only half-excluded: `ijoy.dll` supplies the DEVICE objects while this game-side wrapper holds them (§6) |
| in game: binding conflicts | `loadsave_binding_find_conflict` walks only the band a slot belongs to (`0..0x20` on foot, `0x21..0x3b` in car), so binding the same key to one on-foot and one in-car action should be allowed; untested |
| in game: slot `0x20` | textdb id **0** — the last on-foot slot is unused; nothing should appear for it on the Controls screen; untested |
