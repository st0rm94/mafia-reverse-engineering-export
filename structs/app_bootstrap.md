# The Win32 bootstrap translation unit — `winmain`, the host helpers and the CD check

**Created** 2026-08-23 11:15 +0200 · **Modified** 2026-09-27 17:33 +0200

`[0x00623e10, 0x006242b0)` — **`Winstuff.cpp`** (§6).

This file documents a **translation unit, not a class.** Every function in it is a free function —
no `this`, no vtable, no receiver — so there is no `C_*` type to reconstruct and no vtable coverage
table to keep. It is the PC's Win32 host-services file: it brings the process up, wraps the four
Win32 APIs the engine needs, and implements the CD check. [`speech.md`](speech.md) is the other
TU-scoped doc in this directory (`g_dabing.cpp`).

§5a–§5c cover the neighbouring `Main.cpp`, where `winmain`'s callee `engine_main` lives.

---

## 1. The TU map

Bounded by CRT static-init/atexit pairs at `0x00623e10`/`0x00623e30` and `0x00623ea0`/`0x00623ec0`
on one side and `0x006242b0`/`0x006242d0` on the other. The second pair's initializer only sets
bit 0 of `DAT_006f9ae0` and registers a no-op atexit destructor, so the TU owns one file-scope
static with a trivial destructor.

| VA | name | role | callsites |
|---|---|---|---|
| `0x00623e40` | `winmain` | `GetModuleFileNameA` → strip the filename → `crt_chdir` → `engine_main` | - ¹ |
| `0x00623ed0` | `win_message_box` | modal message box with a mapped button code and a normalized result | 1 |
| `0x00623f80` | `win_load_string_resource` | `LoadStringA` forwarder | 1 |
| `0x00623fa0` | `win_set_app_icon` | `LoadIconA` + `SetClassLongA(GCL_HICON)` | 1 |
| `0x00623fc0` | `cd_find_game_file` | resolve a data file: local install first, then the disc | 4 |
| `0x00624070` | `cd_require_disc` | the blocking retry loop around the drive scan | 2 |
| `0x006240b0` | `cd_file_exists` | `FindFirstFileA` existence test | 1 |
| `0x006240e0` | `cd_scan_drives_for_disc` | the drive scan itself — **cracked in `Game.exe`**, see §5 | 1 |

¹ Called by `crt_startup`.

`std_string_reserve` (`0x00624190`), immediately after, is **not** in this TU — it is a refcounted
string-buffer resize belonging to the string-class TU.

Callsite counts are byte scans of `.text` in `Game.exe` for `E8 rel32` calls, not decomp greps.

---

## 2. The CD chain

```
cd_require_disc
  └─ cd_scan_drives_for_disc          for letter in 'C'..'Z'
       └─ cd_file_exists                  "<letter>:\MafiaGame\a1.dta" ?
```

**`cd_scan_drives_for_disc`** stamps the letter into `"X:\"` (`DAT_0064d928`), calls
`GetDriveTypeA` (**result discarded**), then `GetVolumeInformationA` into a 0x80-byte volume-name
buffer and a 0x80-byte filesystem-name buffer. That call is tested only for success — media present
and readable — and **no volume label is ever compared.** The disc is identified purely by carrying
`MafiaGame\a1.dta` (format string `0x00651b0c`). Returns 1 on the first hit, 0 after `'Z'`.

**`cd_require_disc`** does not check, it **blocks**: on failure it calls `g_engine` vtable `+0x3c`
(releasing the display so a modal box can appear), shows
`MessageBoxA("Please insert MAFIA CD 1", 0x15 = MB_RETRYCANCEL|MB_ICONHAND)`, and retries until the
disc appears or the user cancels, in which case it returns 0. Its two callers are `engine_main` and
`mission_open` (callsite `0x005fff62`).

**`cd_find_game_file`** is a separate path and is *not* part of that chain. It resolves one named
file: it first tries the bare name in the working directory with `CreateFileA`, and only on failure
walks `'C'`..`'Z'` building `"X:\MafiaGame\%s"` (`0x00651ae0`); this walk also calls
`GetDriveTypeA` and discards the result. The **resolved** path is written back into the caller's
buffer. Its four callers and their arguments, recovered from the disassembly because Ghidra hides
the `__fastcall` operands:

| callsite | argument | what the caller does with the resolved path |
|---|---|---|
| `0x005c6539` (`program_process_command`) | `"intro.avi"` | `g_display` vtable `+0x70` |
| `0x005fa411` (`engine_init`) | `"a1.dta"` | `dtaCreate(buf)`, stored in `DAT_006f92a8` |
| `0x005fb214` (`engine_init`) | `"logo1.avi"` | `g_display` vtable `+0x00` |
| `0x005fb25c` (`engine_init`) | `"logo2.avi"` | `g_display` vtable `+0x00` |

Every caller consumes the **buffer**, not just the boolean — which is what proves it is a path
resolver and not a predicate. The set of names is the assets that live on the disc rather than in
the install: the two logo videos, the intro video, and one archive.

---

## 3. The Win32 host helpers

**`win_message_box`** maps an internal button-set code to a Win32 type — `0`/default `MB_OK`,
`1`→`MB_YESNO`, `2`→`MB_YESNOCANCEL`, `3`→`MB_RETRYCANCEL`, `4`→`MB_OKCANCEL` — takes its parent HWND
from `g_engine` vtable `+0x10` (NULL when `g_engine` is null), and normalizes the result:
`IDRETRY`/`IDYES` → 1, `IDNO` → 0, `IDCANCEL` → 2, anything else → -1. Its single caller reports a
sound error (`"Sound type I3DSOUND_VOLUME not exist."` / `"ConstructSound"`). `cd_require_disc` does
**not** use it — it calls `MessageBoxA` directly with a literal `0x15`.

**`win_load_string_resource`** is a bare forwarder to `LoadStringA`. Its one caller, `engine_init`,
fetches **string id 3** into a 128-byte buffer and hands it to `g_display` vtable `+0x18`.

**`win_set_app_icon`** calls `LoadIconA(hInstance, MAKEINTRESOURCE(1))` and, on success,
`SetClassLongA(hWnd, GCL_HICON, hIcon)`. `engine_init` passes the HWND it has just fetched from
`g_display` vtable `+0x10`.

### 3a. The resources this TU reads

From `.rsrc`, which is **byte-identical in retail and in both cracked builds**:

| resource | value |
|---|---|
| `STRING/#1/#1029` id 1 | `Software\LS3D game reg\Setup` |
| `STRING/#1/#1029` id 2 | `Mafia` |
| **`STRING/#1/#1029` id 3** | **`Mafia`** — what `win_load_string_resource` fetches |
| `ICON/#1/#1029` | the icon `win_set_app_icon` installs |

---

## 4. Strings

| VA | content |
|---|---|
| `0x0064d928` | `X:\` — the root-path template both drive walks stamp a letter into |
| `0x00651ae0` | `X:\MafiaGame\%s` — `cd_find_game_file`'s disc path |
| `0x00651af0` | `Please insert MAFIA CD 1` |
| `0x00651b0c` | `%c:\MafiaGame\a1.dta` — the disc's identifying file |

---

## 5. `cd_scan_drives_for_disc` is cracked in `exe-re/Game.exe`

The four bytes at `0x006240e0` in our binary are `33 C0 40 C3` (`xor eax,eax; inc eax; ret`) — a
no-CD stub written over the real prologue `81 EC 8C 01` (`sub esp,0x18c`), so the check always
succeeds.

Consequences that matter when reading this TU:

* the decomp shows the **original** function; `Game.exe` does not. That divergence is registered in
  [`basic/artifacts/crack.md`](../basic/artifacts/crack.md) §5c;
* the restored binary `exe-re/Game-uncracked-code.exe` matches the decomp exactly;
* the body's `stack0x...` pseudo-locals are an artifact of the missing prologue, not of the
  function — they are ordinary locals in the 0x18c frame;
* the crack split the function three ways for Ghidra: the stub at `0x006240e0`, a fragment at
  `0x006240e6`, and one of its two epilogues at `0x0062417f`. The decomp carries it as one function.

Full provenance, including SafeDisc 2.70.030 and both cracker banners:
[`basic/artifacts/crack.md`](../basic/artifacts/crack.md).

---

## 5a. The three translation units around `engine_main`

The `.CRT$XCU` initializer table at `.data:0x00646264` gives the object-file boundaries; the spans
and the file names come from the PS2 side (§6).

| PC range | translation unit | what is in it |
|---|---|---|
| `[0x005dc190, 0x005f79a0)` | `g_game.cpp` | `C_game` |
| `[0x005f79a0, 0x005f7bc0)` | **`Load_progress.cpp`** | 2 guard-only initializers + `load_progress_callback` |
| `[0x005f7bc0, 0x005fe4f0)` | **`Main.cpp`** | 9 file-scope singletons, then `engine_main`, `engine_init`, `ls3d_setup_set_defaults`, the `C_I3D_cache<T>` implementation and the force-feedback manager |
| `[0x005fe4f0, 0x00608390)` | **`Mission.cpp`** | `C_mission`, the scene loader, the `display_*` accessors |

The PC link order across this stretch is alphabetical by source file — `g_game` < `Load_progress`
< `Main` < `Mission`.

### `Main.cpp`'s initializer block

`Main.cpp` is identified by matching its initializer block object-for-object, in order, against the
PS2's `__sinit_Main.cpp` (§6); a boundary alone proves nothing about a span.

| PC `.CRT$XCU` slot | PC object |
|---|---|
| `0x005f7bc0` | *(guard-only, ctor elided)* |
| `0x005f7bf0` | `g_ls3d_setup` `0x006f952c` (§5b) |
| `0x005f7cd0` | `g_mission` `0x006f9440` |
| `0x005f7d10` | `g_i3d_model_cache` `0x006f9418` |
| `0x005f7ed0` | `g_i3d_sound_cache` `0x006f93f0` |
| `0x005f8080` | `g_i3d_anim_cache` `0x006f93c8` |
| `0x005f8230` | `g_input` `0x006f92e0` |
| `0x005f8270` | `g_joy_device_list` `0x006f92cc` |
| `0x005f82d0` | the screen-fade object `0x006f9278` (`structs/screen_fade.md`) |
| `0x005f8330` + `0x005f8350` | two `S_vector`s, empty ctors |
| `0x005f8370` | the force-feedback manager `0x006f91e0` |

## 5b. `g_ls3d_setup` — the persisted engine settings block

`0x006f952c`, **0x3f bytes**, written to and read from the registry value `LS3D_setup` under
`HKLM\Software\Illusion Softworks\Mafia` — both strings byte-read from `Game.exe`, and `engine_init`
does `push 0x3f; push 0x6f952c` into `RegSetValueExA` at `0x005fa219`. The name is the game's own
name for the block; `display_` is taken by the `C_mission + 0x90` sub-object (38 callsites).

| offset | field | default | evidence |
|---|---|---|---|
| `+0x00` | version tag | `0x14` | `engine_main` refuses the saved blob unless it matches |
| `+0x01` | adapter index | 0 | passed as element [1] of the `GetIGraph()` init struct |
| `+0x02` | `g_ls3d_setup_width` | 800 | the RIGHT member of two `SetRect` calls |
| `+0x06` | `g_ls3d_setup_height` | 600 | the BOTTOM member of the same two |
| `+0x0a` | `g_ls3d_setup_bpp` | 32 | copied into the `IGraph` init struct |
| `+0x0e` | `g_ls3d_setup_field_0x0e` | 0 | a dword copied into the `IGraph` init struct beside the bpp |
| `+0x12` | `g_ls3d_setup_field_0x12` | 0 | compared with 4 and 2 → mode bits `0x2000` / `0x1000` ¹ |
| `+0x13` | `g_ls3d_setup_field_0x13` | 0 | mode bit `0x4000` ¹ |
| `+0x14` | `g_ls3d_setup_field_0x14` | 0 | mode bit `0x8000` ¹ |
| `+0x15` | `g_ls3d_setup_field_0x15` | 0 | mode bit `0x2` ¹ |
| `+0x16` | `g_ls3d_setup_field_0x16` | 1 | mode bit `0x10` ¹ |
| `+0x17` | `g_ls3d_setup_field_0x17` | 0 | mode bit `0x100` ¹ |
| `+0x18` | `g_ls3d_setup_flag_bytes_0x18` | 0, 0, 1, 1 | four bytes with different subjects: byte 0 → **I3D driver state `0x15`**, byte 1 → mode bit `0x4` ¹, byte 2 → driver state `0x22`, byte 3 → driver state `0x19` (`engine_init`) |
| `+0x1c` | `g_ls3d_setup_flag_bytes_0x1c` | - | written to **I3D driver state `0x11`** by `engine_init` ² |
| `+0x1d` | byte 1 of the `+0x1c` group | 0 | **gates the sound driver**: `engine_init` calls `ISndGetDriver()` only when it is 0, so non-zero = sound disabled |
| `+0x1e` | byte 2 of the `+0x1c` group | - | contributes bit `0x10` to the sound-driver init flags |
| `+0x1f` | byte 3 of the `+0x1c` group | - | contributes bit `0x20` to the IGraph mode word |
| `+0x20` | `g_ls3d_setup_field_0x20` | 0 | contributes bit `0x20` to the sound-driver init flags |
| `+0x21` | `g_ls3d_setup_field_0x21` | 1 | written only by the two default fillers; no reader located |
| `+0x22` | `g_ls3d_setup_field_0x22` | 0 | a dword, written only by the two default fillers; no reader located |
| `+0x26` | `g_ls3d_setup_wbuffer_enabled` | 1 | the **W-buffer** setting ³ |
| `+0x27 … +0x3b` | the six quality/scale mirrors | 1.0, 1.0, 2, 2, 3, 2 | the persisted copy of `C_mission + 0x90` — `structs/mission.md` §2 |

¹ The `IGraph` mode word `engine_init` assembles: `+0x12` vs 4 and 2 → `0x2000`/`0x1000`,
`+0x13` → `0x4000`, `+0x14` → `0x8000`, `+0x15` → `2`, `+0x16` → `0x10`, `+0x17` → `0x100`,
`+0x19` → `4`, `+0x1f` → `0x20`, plus the constant `0x10090`. What each bit selects is not decoded,
which is why these fields keep offset names.
² `engine_igraph_error_callback` clears it before re-saving the blob on a video-memory overflow, so
it selects something video-memory-hungry. The `+0x1c` group keeps a mechanical offset name because
its four bytes have different subjects (two graphics, two sound).
³ Written 1 by both `ls3d_setup_static_init` and `ls3d_setup_set_defaults`; its only reader is
`engine_set_wbuffer`, which ANDs a caller's request with it before setting I3D render state 4.

Defaults are written twice by the same source function: `ls3d_setup_static_init` (`0x005f7c00`)
fills the global, `ls3d_setup_set_defaults` (`0x005fa2a0`) fills a stack copy for `engine_main` to
validate the registry blob against.

## 5c. The `engine_*` free-function family

`Main.cpp`'s code half `[0x005f83b0, 0x005fb6e0)` holds twelve free functions. `engine_` is a
**translation-unit** prefix, not a class prefix — as are `load_progress_` (2 functions),
`ls3d_setup_` (2), and this file's own `win_` / `cd_`. None of them is a `C_game` or `C_mission`
member.

| PC VA | PC name | PS2 twin | role |
|---|---|---|---|
| `0x005f83b0` | `engine_set_wbuffer` | `SetWBuffer__Fb` | I3D render state 4, gated by the persisted setting |
| `0x005f83e0` | `joy_devices_init` | - | `ijoy.dll` bring-up; fills `g_joy_device_list` |
| `0x005f8660` | `joy_devices_done` | - | releases them |
| `0x005f86c0` | `engine_draw_overlay_callback` | `cbScene__F20SCN_CALLBACK_MESSAGEUiUi` | scene callback **id 6** |
| `0x005f8700` | `engine_new_mission` | `InternMissionInit__Fv` | the queued mission change; `structs/mission.md` §7 |
| `0x005f9540` | `engine_game_loop` | `GameLoopSingle__Fv` | the single-player in-game loop (a `do { } while (true)` loop) |
| `0x005f98e0` | `engine_igraph_error_callback` | - | IGraph error hook, out-of-memory only ¹ |
| `0x005f99b0` | `engine_main` | `main` | the real main; called by `winmain` |
| `0x005fa2a0` | `ls3d_setup_set_defaults` | - | writes the defaults into a stack copy |
| `0x005fa320` | `engine_init` | `InitSystem__FPvRb` | dta, IGraph, I3D driver, mission, sound, joysticks |
| `0x005fb330` | `engine_show_init_error` | `InitErrorMessageBox__F11LS3D_RESULTi` | the 21-arm `LS3D_RESULT` message box |
| `0x005fb4f0` | `engine_shutdown` | `CloseSystem__Fv` | the mirror of `engine_init` |

A `-` twin is PC-only.

¹ Zero `E8` callers and zero `.rdata` datarefs, yet live: `engine_init` stores its address at
`0x005fa5ae` (`&engine_igraph_error_callback`).

## 6. PC vs PS2

**The Win32 code is PC-only, and that is a measurement rather than an assumption.** `SLUS_206.71`
contains no `WinMain`, `LoadString`, `LoadIcon`, `SetClassLong`, `GetDriveType`,
`GetVolumeInformation`, and no drive-letter scan of any kind. Its only disc-related symbols are
Sony's SDK — `sceCdSearchFile`, `sceCdNcmdDiskReady`, `sceCdLayerSearchFile`, `_Cdvd_cbLoop` —
the console's driver layer, not a game-level disc check. A PS2 title has no drive letters, no
`HWND`, and no Win32 resources, so none of these functions can have a counterpart.

**The TU name.** The PS2 symbol table carries `__sinit_Winstuff.cpp` at `0x004d10f0`, the entry
immediately after `__sinit_VE_Debris.cpp` — and the PC `.CRT$XCU` slot that opens this TU
(`0x006462f8` -> `0x00623e10`) is likewise the one immediately after `VE_Debris.cpp`'s. That is a
five-of-five ordered lockstep across the two build systems ([`glass_shatter.md`](glass_shatter.md)
§6). The FILE exists on both builds; on the PS2 it holds console platform code instead of Win32
code.

**`MessageBox__FPvPCcPCcUi` is not a twin of `win_message_box`.** It takes four parameters to
`win_message_box`'s three, and its entire body is `LSI_dbgExit(0x4b67e8, 1); return 0;` — while
`LSI_dbgExit__FPCcb` is itself an empty stub. A gutted stub with a different arity is not evidence
of a twin.

**`Main.cpp` lockstep (§5a).** The PC link order `g_game` < `Load_progress` < `Main` < `Mission` is
exactly the PS2 `__sinit_` order with the PS2-only `MemoryCard.cpp` omitted. The PS2 counterparts of
the initializer block, in order:

| PC object | PS2 statement |
|---|---|
| guard-only initializer | - *(PC-only)* |
| `g_ls3d_setup` | - *(PC-only)* |
| `g_mission` | `__ct__9C_missionFv(0x509b10)` |
| `g_i3d_model_cache` | `C_I3D_model_cache` `0x50a880` |
| `g_i3d_sound_cache` | `C_I3D_sound_cache` `0x50a8c0` |
| `g_i3d_anim_cache` | `C_I3D_anim_cache` `0x50a900` |
| `g_input` | `__ct__5C_vidFv(0x4dc920)` |
| `g_joy_device_list` | `vector<P5C_Joy>` `0x50a948` |
| the screen-fade object | `silver_screen` `0x50a9a0` |
| two `S_vector`s | `__construct_array(0x50aa20, __ct__8S_vectorFv, 0, 0x10, 2)` |
| the force-feedback manager | `__ct__12C_ff_managerFv(0x50aa50)` |

**`g_ls3d_setup` `+0x1c`.** The PS2's `InitSystem` writes I3D driver state `0x11` from its own
settings block too, alongside `0x15` / `0x19` / `0x22` fed by the `+0x18` group.

**`engine_draw_overlay_callback`** is scene callback id 6 on both builds.

Related: [`basic/artifacts/dll.md`](../basic/artifacts/dll.md) — SafeDisc's own import table in the
retail exe names `LS3DF.dll`, `IJoy.dll`, `Inet.dll` and `rw_data.dll` in the developers' original
casing, independently corroborating that document's four Illusion DLLs.

---

## 7. Open items

| item | state |
|---|---|
| `g_engine` vtable slots used here | undocumented: `+0x10` (returns the HWND), `+0x18` (consumes the app-name string), `+0x3c` (called before a modal box) |
| `GetDriveTypeA` in both drive walks | its result is discarded in `cd_find_game_file` and in `cd_scan_drives_for_disc`; no purpose found |
