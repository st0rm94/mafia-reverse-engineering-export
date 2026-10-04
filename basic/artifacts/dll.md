# The Illusion DLLs — what is NOT in `Game.exe`

**Created** 2026-08-14 09:10 +0200 · **Modified** 2026-09-30 23:08 +0200

This documents the PC build. PS2 material is confined to §6.

---

## 1. Why this file exists

`Game.exe` is not the whole game. It statically imports from **four Illusion DLLs**, and
everything those DLLs implement is **absent from the decomp** — no body, no vtable, no
function to name, ever.

That matters beyond the DLLs themselves: a function the other build has and `Game.exe` lacks
may simply live in a DLL (§6).

---

## 2. The import table (measured from `Game.exe`'s PE directory)

| DLL | IAT | imports | what it is |
|---|---|---|---|
| `kernel32.dll` · `user32.dll` · `advapi32.dll` · `winmm.dll` | - | - | Windows |
| **`ls3df.dll`** | `0x0063b178` | **34** | **the LS3D engine** — renderer, math types, sound driver |
| **`ijoy.dll`** | `0x0063b018` | **1** | input + **force feedback** |
| **`inet.dll`** | `0x0063b028` | **1** | networking |
| **`rw_data.dll`** | `0x0063b230` | **8** | the **`.dta` archive** reader/writer |

Three of the four expose a single factory entry point and hand back a COM-style interface
object; everything after that is vtable dispatch, which is why the decomp shows long
chains of `(**(code **)(*obj + 0xNN))(...)` with no named callee.

The ten `ijoy.dll` / `inet.dll` / `rw_data.dll` import thunks at `[0x006248a8, 0x006248e4)` and
the 29 `ls3df.dll` thunks at `[0x006242e0, 0x006243ac)` keep their import spellings
(`S_vector::` / `S_matrix::` / `I3D_frame::` for the latter): the bodies are in the DLLs, so no
behaviour-derived name can be body-diffed, and the import string is the only authority for what
each entry point does (`structs/crt_runtime_tail.md` §3 and §7).

---

## 3. `ijoy.dll` — input and force feedback

**One import: `_CreateJoyBase@20`.**

```
joy_devices_init (0x005f83e0):
               _CreateJoyBase@20(&g_joy_base, param, mode, g_engine, 0)
               (*g_joy_base + 0x18)()      -> device count
               (*g_joy_base + 0x2c)(...)   -> per-device enumeration
               ... each device pushed into the device vector at 0x006f92d0 ...
               (*g_joy_base + 8)()         -> Release, on teardown
```

| PC global | Address | Role |
|---|---|---|
| `g_joy_base` | `0x006f956c` | the created base object |
| `g_joy_device_list` / `g_joy_device_list_end` / `g_joy_device_list_cap` | `0x006f92d0` / `d4` / `d8` | the input DEVICE vector ¹ |

¹ Filled by the `_CreateJoyBase@20` enumerator; 32 live references across 10 functions; elements
are reached only through device vtable slots `+0x08` / `+0x58` / `+0x68` / `+0x88`.

Known device-object vtable slots, all from force-feedback use sites:

| Slot | Use |
|---|---|
| `+0x58` | called on every device at shutdown |
| `+0x68` | returns a capability struct — `force_feedback_manager_game_init` takes an FF-support byte from it |
| `+0x88` | **create effect** — returns the object stored at effect `+0x30` |

Effect-object vtable slots (`ILSIInputEffect`), from the five
`force_feedback_effect_*upload*_params` bodies and `force_feedback_manager_play_effect`:

| Slot | Use |
|---|---|
| `+0x08` | release |
| `+0x0c` | download / start — returns false when the device has no free slot |
| `+0x10` | stop / unload |
| `+0x14` | create with (type, direction, duration, gain, 0, 0) |
| `+0x1c` | set gain |
| `+0x24` | set direction |
| `+0x2c` | set duration |
| `+0x3c` / `+0x40` / `+0x44` / `+0x48` | set constant / ramp / periodic / condition params |
| `+0x58` | status — `== 5` means playing |

Full subsystem: `structs/force_feedback.md`.

A **game-side wrapper** around a DLL object *is* in `Game.exe` and *is* nameable. The
force-feedback effect family looks like DLL code because it forwards to one — the tell that it is
not is that it holds the DLL object at `+0x30` and calls *through* it.

---

## 4. `ls3df.dll` — the LS3D engine

**34 imports, and they are MSVC-mangled C++ names**, so this DLL hands over real class and
method names. Ghidra demangles them into the decomp, which is why `S_matrix::Inverse` and
`I3D_frame::UpdateWMatrixProc` appear as named callees with no body anywhere.

| Group | Imports |
|---|---|
| `S_matrix` | `Inverse` · `Identity` · `SetRot` · `SetRot3` · `SetDir` · `SetDir3` · `GetScale` · `GetUScale` · `operator*` · `operator*=` |
| `S_vector` | `RotateByMatrix` · `RotateByNormMatrix` · `AngleTo` · `operator*` · `operator*=` |
| `S_quat` | `Make` (×2 overloads) · `Normalize` · `Inverse` · `GetDir` · `SetDir` · `RotationMatrix` · `RotateByMatrix` · `Slerp` |
| `S_plane` | `IntersectionLine` |
| `I3D_frame` | `UpdateWMatrixProc` · `UpdateHRBoundVol` · `UpdateHRBoundVolW` |
| `I3D_shadow` | no imports — only pointers cross the DLL boundary; the class behind the blob shadows ¹ |
| `I3D_visual` | `UpdateWBoundProc` |
| free | `I3DGetDriver` · `ISndGetDriver` · `GetIGraph` · `dbgPrintf` · `TEV` |

¹ Created by the driver factory with **object type `0xf`** — the only use of that type anywhere in
the PC decomp. `Open(const char*, u32)` at vtable `+0x50`, `SetTo(I3D_frame*)` at `+0x3c`,
`LinkTo(I3D_frame*, u32)` at `+0x2c`. See `structs/fake_shadow.md`.

* **The math types are engine types, not game types.** `S_matrix`, `S_vector`, `S_quat`,
  `S_plane` are all defined in the DLL. Any "unnamed math helper" in `Game.exe` is
  therefore game-side code operating *on* those types, never the types' own methods.
* **The mangling records the calling convention:** `QAG` / `QBG` = public
  (non-const / const) **`__stdcall`** member functions. Preserve that on any signature.
* `I3DGetDriver` / `ISndGetDriver` / `GetIGraph` are the three driver factories; the objects
  they return are the `g_renderer` / `g_i3d_driver` family at `0x006f9520`, `g_engine`
  (`0x006f9528`, the `IGraph` interface) and the sound driver. Those are DLL objects too: `Game.exe`
  holds none of their method bodies, but the shipped `LS3DF.dll` does, so a vtable slot can be
  identified from the DLL's own body — `structs/app_bootstrap.md` §3a does so for five `IGraph`
  slots.

---

## 5. `inet.dll` and `rw_data.dll`

**`inet.dll`** — one import, `_CreateINetBase@4`. Same factory shape. The whole networking
transport is outside `Game.exe`, while the *per-class replication bodies* are in the exe
(`structs/bottle.md` §3a).

**`rw_data.dll`** — the `.dta` archive layer, 8 imports:
`_dtaCreate@4` · `_dtaOpen@8` · `_dtaOpenWrite@8` · `_dtaClose@4` · `_dtaRead@12` ·
`_dtaWrite@12` · `_dtaSeek@12` · `_dtaSetDtaFirstForce@0`.

This is the API every asset loader in the game funnels through — `collision_load_level`
calls `dtaOpen(name, 0)`, and the `.dta` mounting order is what `_dtaSetDtaFirstForce@0`
controls.

---

## 6. PC vs PS2

The PS2 build has no DLLs: it statically links the equivalents of all four into the ELF, so **the
PS2 symbol table contains classes whose PC code is unreachable by construction**. A class with PS2
symbols and no PC code is therefore not automatically inlined, in another cluster, or
PS2-port-only — **it may simply live in a DLL on the PC**. This is a fifth explanation for "PS2
method, no PC twin" (§6a), and the only one that can never be resolved by looking harder inside
`Game.exe`.

| PS2 class | methods | PC location |
|---|---|---|
| `LSI` | 10 | `ls3df.dll` — `IDirect3D8`-shaped (`CreateDevice`, `GetAdapterIdentifier`, `_LSIPRESENT_PARAMETERS_`, `_LSICAPS8`) |
| `LSIDevice` | 83 | `ls3df.dll` — `IDirect3DDevice8`-shaped |
| `LSITexture` · `LSISurface` · `LSIVertexBuffer` · `LSIIndexBuffer` | 7 / 6 / 7 / 6 | `ls3df.dll` |
| `LSISound` · `LSISoundBuffer` · `LSISound3DListener` | 6 / 13 / 4 | `ls3df.dll` (via `ISndGetDriver`) |
| `LSIInput` | 6 | `ijoy.dll` — `IDirectInput8`-shaped (`CreateDevice`, `EnumDevices`, `RunControlPanel`, `CreateDeviceEx`) |
| **`LSIInputDevice`** | **17** | `ijoy.dll` — `IDirectInputDevice8` method-for-method, in DirectInput's own order ¹ |
| `ILSIInputEffect` | - | `ijoy.dll`; named only in `CreateEffect`'s signature. The object at effect `+0x30` |
| `LSIPad` | 4 | PS2-only — a DualShock wrapper with no PC counterpart |
| `I3D_shadow` | 17 | `ls3df.dll` ² |

¹ `GetCapabilities` · `EnumObjects` · `Get/SetProperty` · `Acquire`/`Unacquire` · `GetDeviceState`
· `SetDataFormat` · `SetCooperativeLevel` · `RunControlPanel` · `CreateEffect` · `EnumEffects` ·
`GetForceFeedbackState` · `SendForceFeedbackCommand` · `Poll` · `EnumEffectsInFile`.

² The object type `0xf` is the same constant on both builds. PS2 methods beyond the three slots in
§4: `Duplicate`, `SetRange`, `SetColor`, `SetTextureSize`, `SetOrthogonal`,
`SetMaxTextureDistance`, `SetReverseFactor`, `SetShadowLock`, `DeleteFromAnyShadow`, `UpdateV`,
`Close`.

**LSI = LS3D Interface**: Illusion's rename of the DirectX 8 API surface, so the same
engine source could target both platforms. The type names follow the same scheme —
`LSIIEFFECT` = `DIEFFECT`, `LSIIDEVCAPS` = `DIDEVCAPS`, `LSIIPROPHEADER` = `DIPROPHEADER`,
`LSIIDATAFORMAT` = `DIDATAFORMAT`, `_LSIFORMAT` = `D3DFORMAT`, and so on.

PS2 has `rw_data.cpp` in its source-file manifest, i.e. the `.dta` layer compiled in.

### 6a. When a PS2 method has no PC twin

Ask first **which side of a DLL boundary the class sits on**:

1. Is the class name `LSI*` / `ILSI*`, or does it wrap one? ⇒ it is in a DLL. Stop.
2. Does `Game.exe` import a factory for that subsystem (`_Create*Base@N`, `I3DGetDriver`,
   `ISndGetDriver`, `GetIGraph`)? ⇒ the implementation is behind it.
3. Only then work through the other four explanations: **inlined on PC** (the callee's
   distinctive constant sits inside the caller), **in another PC cluster** (search the whole
   binary), **virtual with no textual callsites** (find it in the vtable), or **PS2-port-only**
   (every PS2 caller is itself a PS2-only subsystem, e.g. city streaming).
