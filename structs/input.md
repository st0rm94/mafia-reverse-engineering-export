# `g_input` — the PC input manager

**Created** 2026-08-31 22:48 +0200 · **Modified** 2026-11-04 09:26 +0100

The PC's keyboard / mouse / joystick binding layer, one file-scope object at **`0x006f92e0`**. It
owns the 60 rebindable **actions** the Controls screen exposes, resolves them into two per-action
float tables once a frame, and hands those tables to the player controller, the camera, the script
VM and the GUI. The PS2 build has no counterpart class (§6); the name `g_input` and every type name
below are the project's.

| | |
|---|---|
| PC prefix | `input_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | `0xe8`, bounded by its neighbours (§1) |
| PC constructor | `input_ctor` @ `0x005d14d0`, reached by `input_static_ctor`'s tail jump |
| PC translation unit | `[0x005d14a0, 0x005d3a60)`: 21 starts, all named, between the unit's own CRT pairs; the static quartet sits in `Main.cpp` (§3a) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

```c
typedef struct g_input {                              // 0xe8 bytes @ 0x006f92e0
    /* +0x00 */ LS3D_engine *engine;                  // [H] g_engine: owns keyboard + mouse
    /* +0x04 */ std::vector<S_input_joy_state> joysticks; // [H] one 0x8c record per enumerated joystick (§2)
    /* +0x14 */ S_input_binding capture;              // [H] scratch record of the capture screen
    /* +0x20 */ uint8_t  keys[64];                    // [H] the keyboard scancodes any binding uses
    /* +0x60 */ uint8_t  key_refs[64];                // [H] how many bindings use keys[i]
    /* +0xa0 */ float    mouse_range_x;               // [H] deflection denominator, X axis
    /* +0xa4 */ float    mouse_range_y;               // [H] the same, Y axis
    /* +0xa8 */ float    mouse_range_wheel;           // [H] the same, wheel (Z)
    /* +0xac */ std::vector<S_input_binding> bindings; // [H] one per action slot, 60
    /* +0xbc */ std::vector<float> sampled_values;    // [H] per action, from the current state
    /* +0xcc */ std::vector<float> oneshot_values;    // [H] per action, edge / repeat gated
    /* +0xe0 */ uint64_t prev_key_mask;               // [H] last frame's 64-bit key mask
} g_input;
```

**The size is bounded, not inferred.** `g_joy_device_list_cap` ends at `0x006f92dc` and
`g_i3d_anim_cache` begins at `0x006f93c8`; `0x006f93c8 − 0x006f92e0 = 0xe8`, which is exactly where
`prev_key_mask` ends. `+0xdc..+0xdf` is never accessed: it is the alignment padding in front of
the 8-byte mask. Ghidra carries two members of this global object as standalone symbols,
`g_input_bindings_begin` (`0x006f9390` = `+0xb0`) and `g_input_bindings_end` (`+0xb4`).

**`keys[64]` and `key_refs[64]` — why 64.** The engine reports keyboard state as a 64-bit mask over
the table it was last given: `input_set_binding` publishes `keys` to the engine through its slot
`+0xa4` as `(keys, 0x40)`, `input_update` reads the mask back through slot `+0xa8`, finds a key
binding's scancode at index *i* of `keys` and tests bit *i* of the mask. So entry *i* is "the
scancode the engine reports in bit *i*", and 64 is the mask's width; a 65th distinct key cannot be
bound (`input_set_binding` refuses when no free entry is left). `key_refs[i]` counts the bindings
that use `keys[i]`; the entry is cleared and the table republished when it drops to 0, so two
actions bound to one key share an entry.

**The three mouse ranges** are the clamp-and-divide denominators of the three mouse axes in the
order of the binding codes `0x100` / `0x200` / `0x300` (§1a); larger means less sensitive.

**The object has almost no textual existence.** Every method is `__thiscall`, so the receiver
travels in ECX and a text search of the decomp for `0x006f92e0` finds only the static constructor
and destructor thunks.

### 1a. `S_input_binding` — one action's binding, 0xc bytes

```c
typedef struct S_input_binding {
    /* +0x00 */ int32_t  mode;          // [H] 0 = whole / digital, 1 = positive half-axis, 2 = negative half-axis
    /* +0x04 */ int16_t  device_kind;   // [H] 1 = keyboard, 2 = mouse, 3 = joystick
    /* +0x06 */ uint16_t code;          // [H] scancode / button / axis / hat, by kind (below)
    /* +0x08 */ int32_t  device_index;  // [H] index into joysticks; only meaningful for kind 3
} S_input_binding;
```

The contents are proven by the shipped strings that `input_binding_get_label` returns:

| Kind | Code | What it is |
|---|---|---|
| 1 | DIK scancode | keyboard: `0x2a`/`0x36` Shift, `0x1d`/`0x9d` Ctrl, `0x38`/`0xb8` Alt, `0x01` Escape, `0x0e` Backspace, `0x1c` Return … |
| 2 | `1` / `2` / `4` | mouse button left / right / middle |
| 2 | `0x100` / `0x200` / `0x300` | mouse axis X / Y / Z (wheel) |
| 3 | `< 0x100` | joystick axis index 0..7, named by the device's vtable slot `0x5c` |
| 3 | `0x100`..`0x1ff` | joystick button `code − 0x100`, named by device vtable slot `0x64` |
| 3 | `>= 0x200` | POV hat: hat index `(code >> 4) & 0xf`, direction mask `code & 0xf`, named by device vtable slot `0x6c` |

A `device_index` that no longer names a live joystick renders as the shipped string **"Not
attached joy"**, which is what proves `+0x08` is an index into `joysticks` and not a handle.

### 1b. `S_input_joy_state` — one joystick, 0x8c bytes

```c
typedef struct S_input_joy_state {
    /* +0x00 */ IJoy_device *device;   // [H] the ijoy.dll device object; addref'd on insert
    /* +0x04 */ S_joy_block  cur;      // [H] this frame's state
    /* +0x48 */ S_joy_block  prev;     // [H] the reference state edges are measured against
} S_input_joy_state;                   // 0x04 + 0x44 + 0x44 = 0x8c
```

`input_update` fills `cur` through device slot `0x50` and `prev` through slot `0x54`;
`input_poll_joysticks` fills `prev` through slot `0x50`, so the capture screen starts from a fresh
snapshot and a control held down when the screen opens is not captured.

### 1c. `S_joy_block` — the device state, 0x44 bytes

```c
typedef struct S_joy_block {
    /* +0x00 */ float    axis[8];           // [H] the analog axes, binding codes 0..7
    /* +0x20 */ float    pov[4];            // [H] the POV hat angles, radians
    /* +0x30 */ uint8_t  pov_centered[4];   // [H] non-zero => that hat is centred
    /* +0x34 */ uint32_t buttons[4];        // [H] a 128-bit button mask
} S_joy_block;
```

The block is the device state `ijoy.dll` keeps at `device+0x130`; device slot `0x50` copies those
`0x44` bytes out. Its arrays follow the DLL's format, and the binding codes address them:

* **`axis[8]`** — a joystick axis binding's code is the axis index, `< 0x100`, and the update loop
  selects one of eight float slots `+0x04..+0x20` of the record by it.
* **`pov[4]` / `pov_centered[4]`** — a hat binding carries the hat index in bits 4..7 of the code;
  four hats are stored, angle and centred flag side by side. The thresholds `input_update` and
  `input_capture_binding` compare an angle against — `0.1`, `1.5`, `1.6`, `3.12`, `3.16`, `4.69`,
  `4.72` — are radians around 0, π/2, π and 3π/2, which is how an angle becomes the 4-bit
  direction mask of the code.
* **`buttons[4]`** — 128 buttons, button *n* in bit `n & 0x1f` of word `n >> 5` (the DLL's own
  button test). `input_update` and `input_capture_binding` read only word 0 (`1 << (code & 0x1f)`,
  and the capture loop stops at 32), so a joystick button above 31 cannot be bound.

### 1d. The capture filter

`input_capture_binding` takes two stack arguments (it returns with `ret 8`): a device FILTER and a
poll flag. Each set filter bit skips one input source:

| Bit | Source skipped |
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
across the engine's callee-cleans virtual calls; in the disassembly all five reads of the filter
sit at stack depth `0x1c`, where the first argument lives.

## 2. Field evidence

Measured by a census of every `this+N` access in the `input_*` functions, a census of their
callers, and body reads of `input_ctor`, `input_set_binding`, `input_clear_bindings`,
`input_poll_joysticks` and the device-slot calls of `input_update` / `input_capture_binding`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | engine | H | `input_ctor` (0), `input_init_devices` | `input_update`, `input_capture_binding`, `input_set_binding` | ¹ |
| +0x04 | joysticks | H | `input_ctor`, `input_init_devices`, `input_release_all`, `input_dtor` | `input_update`, `input_poll_joysticks`, `input_capture_binding`, `input_binding_get_label`, `input_set_binding`, `input_release_all` | the alloc byte, then begin, end, cap; stride `0x8c`; the "Not attached joy" bound check divides by it |
| +0x14 | capture | H | `input_capture_binding` | `gui_key_capture_tick` (the returned record) | the only use of the field |
| +0x20 | keys | H | `input_set_binding`, `input_clear_bindings`, `input_init_devices` | `input_update`, `input_set_binding` | republished to the engine as `(keys, 0x40)` whenever it changes |
| +0x60 | key_refs | H | `input_set_binding`, `input_clear_bindings`, `input_init_devices` | `input_set_binding` | - |
| +0xa0 | mouse_range_x | H | `input_ctor` (20.0), `input_set_mouse_axis_range` | `input_update` | `loadsave_apply_key_bindings` sets 400 |
| +0xa4 | mouse_range_y | H | `input_ctor` (20.0), `input_set_mouse_axis_range` | `input_update` | as `+0xa0` |
| +0xa8 | mouse_range_wheel | H | `input_ctor` (20.0), `input_set_mouse_axis_range` | `input_update` | as `+0xa0` |
| +0xac | bindings | H | `input_ctor`, `input_dtor`, `input_release_all`, `input_clear_bindings` (`end = begin`), `input_set_binding` (grows it, copies the record) | `input_set_binding`, `input_get_binding`, `input_update`, `engine_game_loop` | `engine_game_loop` derives the action count as `(end − begin) / 0xc` |
| +0xbc | sampled_values | H | `input_ctor`, `input_dtor`, `input_release_all`, `input_clear_bindings`, `input_set_binding` (grows it), `input_update` | `input_get_action_values`, `input_get_action_value` | selector 0 (§2a) |
| +0xcc | oneshot_values | H | `input_ctor`, `input_dtor`, `input_release_all`, `input_clear_bindings`, `input_set_binding` (grows it), `input_update` | `input_get_action_values`, `input_get_action_value` | selector 1 (§2a) |
| +0xe0 | prev_key_mask | H | `input_init_devices` (all ones), `input_update` | `input_update` | edge detection for keyboard bindings |

¹ `engine_init` passes `[g_engine]` (disassembled callsite). Engine slots used: `0x2c` / `0x94` /
`0xcc` poll, `0xa4` publish the key table, `0xa8` key mask, `0xb8` / `0x98` capture, `0xd0` / `0xd4`
/ `0xd8` mouse deltas, `0xe8` key repeat.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | mode | H | `input_set_binding` (copies the caller's record), `input_capture_binding` | `input_update`, `input_set_binding` | §1a; see Open item 3 |
| +0x04 | device_kind | H | `input_set_binding`, `input_capture_binding` | `input_update`, `input_set_binding`, `input_binding_get_label` | - |
| +0x06 | code | H | `input_set_binding`, `input_capture_binding` | `input_update`, `input_set_binding`, `input_binding_get_label` | - |
| +0x08 | device_index | H | `input_set_binding`, `input_capture_binding` | `input_update`, `input_set_binding`, `input_binding_get_label` | - |

`input_get_binding` copies the whole record out for `force_feedback_manager_game_init` and
`force_feedback_manager_create_effect`.

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | device | H | `input_init_devices` (addref through the device's vtable slot 1) | `input_update`, `input_poll_joysticks`, `input_capture_binding`, `input_binding_get_label`, `input_set_binding`, `input_release_all` | - |
| +0x04 | cur | H | the device's slot `0x50`, called by `input_update` and `input_capture_binding` | `input_update`, `input_capture_binding` | - |
| +0x48 | prev | H | the device's slot `0x54` from `input_update`; slot `0x50` from `input_poll_joysticks` | `input_update`, `input_capture_binding` | §1b |

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x00 | axis | H | `ijoy.dll` (device slot `0x50` / `0x54`) | `input_update`, `input_capture_binding` | - |
| +0x20 | pov | H | `ijoy.dll` | `input_update`, `input_capture_binding` | - |
| +0x30 | pov_centered | H | `ijoy.dll` | `input_update`, `input_capture_binding` | - |
| +0x34 | buttons | H | `ijoy.dll` | `input_update`, `input_capture_binding` | word 0 only |

### 2a. The 60 actions

The binding vector is indexed by **action slot 0..0x3b**, and both value tables are parallel to
it. The slot → label map is decoded in `structs/loadsave.md` §9a from the 60-dword table at
`0x0064d668`; the profile stores the bindings themselves as one `0x2d0` block at profile `+0x1f4`.
Every decoded consumer matches its label:

| Slot | Textdb label | Consumer |
|---|---|---|
| `0x12` / `0x13` | 312 **Aiming** (primary + secondary) | `camera_apply_desired_pos` and `camera_apply_desired_pos_car` call `input_get_action_value(0x13, 0)` / `(0x12, 0)` |
| `0x1e` | 325 **Objectives** | `player_ai` reads the sampled table with a `> 0.25` gate → `loadsave_objectives_display()` |
| `0x3a` | 310 **Change camera** | `player_ai` reads the one-shot table → `camera_set_mode(-1, 1)` |
| `0x01` / `0x26` | 301 **Forwards** / 332 **Steer Left** | `force_feedback_manager_game_init` reads both bindings to find the physical axis for the on-foot and steering effects |
| `0x21`..`0x28`, `0x37` / `0x38` | 330 **Accelerate**, 331 **Brake / Reverse**, 332 **Steer Left**, 333 **Steer Right**, 342 **Use Clutch** (each primary + secondary) | `player_ai_drive` reads the sampled table at `+0x84`..`+0xa0` and `+0xdc` / `+0xe0` and arbitrates each primary / secondary pair (`structs/player.md` §5a) |

**The two value tables.** `sampled_values` holds the value read from the CURRENT device state;
`oneshot_values` holds the one-shot value: for a key it is gated by the auto-repeat counter, for a
mouse button it is a rising-edge latch, for an axis it is the same analog value.
`input_get_action_values(out, sel)` and `input_get_action_value(i, sel)` both take the selector as
bit 0, and callers characteristically take both tables, one call per selector. The 13 callsites of
`input_get_action_values` are in `player_ai`, `gui_menu_execute_modal`, `race_game_loop`,
`race_game_loop_net`, `program_process_command` and `engine_game_loop`.

## 3. Method family

Coverage: the class is PC-only (§6); all 25 functions of its scope are in the tables below.

### 3a. Construction & destruction

| PC VA | PC name | Object | Status | Role |
|---|---|---|---|---|
| 0x005d14a0 | `crt_static_init_guard_005d14a0` | the translation unit | - | the unit's opening `.CRT$XCU` initializer: sets one bit of `g_crt_static_init_flags` and registers the pair below; no constructor call |
| 0x005d14c0 | `crt_atexit_register_005d14c0` | the translation unit | - | registers a do-nothing destructor with `crt_atexit` |
| 0x005d14d0 | `input_ctor` | instance | - | zeroes the four vectors and `+0x00`, seeds the three mouse ranges with 20.0 |
| 0x005d1560 | `input_dtor` | instance | - | `input_release_all`, then frees the vectors |
| 0x005d3a30 | `crt_static_init_guard_005d3a30` | the translation unit | - | the unit's closing initializer |
| 0x005d3a50 | `crt_atexit_register_005d3a50` | the translation unit | - | its `atexit` partner |
| 0x005f8230 | `input_static_init` | `g_input` | - | `Main.cpp`'s `.CRT$XCU` entry |
| 0x005f8240 | `input_static_ctor` | `g_input` | - | `mov ecx, 0x006f92e0; jmp input_ctor` |
| 0x005f8250 | `input_register_atexit_dtor` | `g_input` | - | `crt_atexit(&input_static_dtor)` |
| 0x005f8260 | `input_static_dtor` | `g_input` | - | `mov ecx, 0x006f92e0; jmp input_dtor` |

`input_ctor` and `input_dtor` are reached only by those two **`E9` tail jumps**, so a scan for `E8`
calls and for the address as an absolute 4-byte value finds zero references to `0x005d14d0` /
`0x005d1560`.

### 3b. Accessors

The class has no accessor of its own.

### 3c. Behaviour

| PC VA | PC name | Status | Role |
|---|---|---|---|
| 0x005d1610 | `input_release_all` | - | releases every device and empties every vector; its one caller is `input_dtor` |
| 0x005d1740 | `input_set_mouse_axis_range` | - | writes the three floats at `+0xa0` |
| 0x005d1770 | `input_init_devices` | - | binds `g_engine` and `g_joy_device_list`, clears both key tables, sets the previous-key mask to all ones; called by `engine_init` |
| 0x005d1a70 | `input_clear_bindings` | - | empties the binding and value vectors and both key tables, the reset half of the profile apply |
| 0x005d1b20 | `input_get_action_values` | - | base and count of one value table |
| 0x005d1bc0 | `input_get_action_value` | - | one action, one table |
| 0x005d1c30 | `input_set_binding` | - | installs or clears one action's binding, growing the three parallel vectors to the slot and maintaining the key table |
| 0x005d1ec0 | `input_get_binding` | - | copies one `S_input_binding` out |
| 0x005d1f30 | `input_update` | - | the per-frame resolve into both value tables |
| 0x005d2ad0 | `input_poll_joysticks` | - | refreshes each joystick's `prev` block before a capture (§1b); called by `gui_key_capture_ctor` |
| 0x005d2b40 | `input_capture_binding` | - | the first control pressed that the filter allows → the scratch record (§1d) |
| 0x005d3180 | `input_binding_get_label` | - | the shipped label strings; called by the key-binding widget and the two edit-field key handlers |
| 0x005d3990 | `input_joy_states_copy_range` | - | copy-assigns a range of `0x8c` records ¹ |
| 0x005d39d0 | `input_joy_states_fill_n` | - | copy-constructs N `0x8c` records ¹ |
| 0x005d3a10 | `input_joy_states_copy_one` | - | copy-constructs one `0x8c` record ¹ |

¹ **Why the three `0x8c` helpers carry the class prefix while `copy_range_stride_0xc` does not.**
Identical-code folding did happen in this link, so "all callers in one unit" alone cannot prove an
owner. What does: each of the three bodies is **byte-unique** in `.text`; each carries the `0x8c`
`S_input_joy_state` stride, this class's record and nothing else's; each has **0 datarefs**; and
the image-wide `E8` / `E9` / dataref census returns 3, 2 and 1 referrers with **no foreign owner**.
`copy_range_stride_0xc` has 42 callsites across 16 owners and keeps its mechanical name.

### 3d. Free helpers

The translation unit holds no free helper.

## 4. Lifecycle

```
Main.cpp static init -> input_static_ctor()          mov ecx, 0x006f92e0; jmp input_ctor
                     -> input_register_atexit_dtor() crt_atexit(&input_static_dtor)
engine_init          -> input_init_devices(g_engine, g_joy_device_list, count)
loadsave_apply_key_bindings(profile)
                     -> input_clear_bindings()
                     -> input_set_binding(profile + 0x1f4 + i*0xc, i)   x 60
                     -> input_set_mouse_axis_range(400, 400, 400)
engine_game_loop     -> input_update(poll)            once a frame
                     -> input_get_action_values(...)  x2, then the consumers index them
gui_key_capture_ctor -> input_poll_joysticks()        prime, so a stale press is not captured
gui_key_capture_tick -> input_capture_binding()       returns the scratch record at +0x14
at exit              -> input_static_dtor()           mov ecx, 0x006f92e0; jmp input_dtor
                                                      -> input_release_all()
```

`input_ctor` seeds the mouse ranges with 20.0; `loadsave_apply_key_bindings` overwrites them with
400.

## 5. `ijoy.dll` and the engine

The game-side wrapper *holds* devices that `ijoy.dll` supplies (`basic/artifacts/dll.md`): the
device vtable slots it calls are `0x30` (an axis setup call from `input_set_binding`), `0x50` /
`0x54` (copy the state block out), `0x58` (poll), `0x5c` / `0x64` / `0x6c` (axis / button / hat
names). Keyboard and mouse go through the LS3D engine object at `+0x00` (§2 ¹).

## 6. PC vs PS2

**No PS2 class corresponds method for method.** `g_input` is constructed by `Main.cpp`'s initializer
block; matching that block against PS2 `__sinit_Main.cpp` object for object puts `g_input` in the
same slot as `__ct__5C_vidFv` (`0x004dc920`), between the animation cache and the joystick vector on
both builds. `C_vid`'s method set is an action-indexed input manager, as the PC class is: `Reset`,
`Update(ms)`, `ReadKey`, `TestKey(i)`, `GetIcon(i)`, `ControlerIsOK`, `IsSameAB` (the PS2's A/B
button-swap option). The two constructors disagree — the PS2 one zeroes five class-static globals
and a `0x40`-byte control array and writes `*this = 0`; the PC one builds four member vectors and
three mouse-range floats and writes `*this = 0` — so `C_vid` is the same role on different input
hardware, and no method twin is expected or stamped.

**How the absence of method twins was established.** Searched:

* the PS2 class column for `key|joy|pad|input|contro|axis|button`: only `LSIInput`,
  `LSIInputDevice`, `GM_OptionsControls`, `GM_TextInput` and one `std::vector<P5C_Joy>` destructor;
* the PS2 free-function list for the same words: only the `scePad*` SDK entry points;
* type names `bind`, `S_key`, `S_control`, `S_input`: nothing;
* the `LSIInput` consumer trace: `LSIInputCreate__FPvUiUiPPvPv` is called from
  `Init__6IGraphFP16IGRAPH_INIT_DESC`, inside the engine, which on the PC lives in `ls3df.dll`.

The PS2 reads its pad through `scePadRead` and `C_vid::ReadKey` / `TestKey`, and ships no
keyboard-scancode name table at all; the PC needs this code because it has a keyboard, a mouse and
DirectInput joysticks.

**`G_ConsoleControls` is a false friend.** That PS2 struct's "Console" means *the games console*,
the pad, not the on-screen text console documented in `structs/console.md`, which lives in the
very next translation unit. `G_ConsoleControls` has no methods and appears only as the parameter
type of `C_player::AI_walk`, `AI_drive`, `AI_car_fire` and `GoToCar_Process`. On the PC those four
take the two value-table pointers directly, a factoring difference, not a missing function.

## 7. Related

- `structs/console.md` — the next translation unit, and the `Console` false friend.
- `structs/loadsave.md` §9 / §9a — the profile block and the 60 action labels.
- `structs/force_feedback.md` — the only consumer of `input_get_binding`.
- `structs/camera.md` — the Aiming consumers.
- `structs/app_bootstrap.md` — `Main.cpp`'s static init.
- `basic/artifacts/dll.md` — `ijoy.dll`.

## 8. Open items

1. **Binding conflicts, in game.** `loadsave_binding_find_conflict` walks only the band a slot
   belongs to (`0..0x20` on foot, `0x21..0x3b` in car), so binding the same key to one on-foot and
   one in-car action should be allowed.
2. **Slot `0x20`, in game.** Its textdb label id is **0**: the last on-foot slot is unused, and
   nothing should appear for it on the Controls screen.
3. **Binding mode 3.** For a joystick binding, `input_set_binding` passes flag 4 to the device's
   slot `0x30` when `mode` is 3 and the code is an axis; the modes the capture and the update
   handle are 0, 1 and 2, and no writer of 3 was located.
