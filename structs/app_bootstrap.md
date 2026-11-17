# The Win32 bootstrap translation unit — `winmain`, the host helpers and the CD check

**Created** 2026-08-23 11:15 +0200 · **Modified** 2026-11-17 09:20 +0100

This file documents **translation units, not a class.** Every function of its own unit is a free function, with no `this`,
no vtable and no receiver, so there is no `C_*` type to reconstruct and no vtable coverage table to keep. It is the PC's Win32
host-services code: it brings the process up, wraps the four Win32 APIs the engine needs, and implements the CD check.

It also hosts the neighbouring `Main.cpp`, where `winmain`'s callee `engine_main` lives, and `Load_progress.cpp`: their static-init
and CRT rows are listed here, and rows a class owns stay in that class's doc as well.

| | |
|---|---|
| PC prefix | `win_`, `cd_`, `engine_`, `ls3d_setup_`, `load_progress_`, `joy_devices_`, `joy_device_list_` |
| PC kind (`+0x10`) | - |
| PC vtable | - |
| PC base class | - |
| PC instance size | - |
| PC constructor | - |
| PC translation unit | `[0x00623e10, 0x006242b0)` `Winstuff.cpp` and the object after it (§4); hosted: `[0x005f79a0, 0x005f7bc0)` `Load_progress.cpp`, `[0x005f7bc0, 0x005fe4f0)` `Main.cpp` |
| PS2 class | free functions of `Main.cpp` and `Load_progress.cpp`; the Win32 unit has none (§11) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Data layout

| Global | VA | Role | Writer(s) | Reader(s) |
|---|---|---|---|---|
| `g_load_progress_screen` | `0x006f91dc` | the load-progress screen `load_progress_callback` creates and destroys | `load_progress_callback` | `load_progress_callback` |
| `g_load_progress_last_reported` | `0x006f91d8` | the last fraction reported, the throttle of `load_progress_callback` | `load_progress_callback` | `load_progress_callback` |
| `g_joy_device_list` | `0x006f92d0` | the joystick vector of `Main.cpp`'s initializer block, whose allocator byte `g_joy_device_list_alloc` is the object at `0x006f92cc` | `joy_device_list_static_ctor`, `joy_device_list_static_dtor`, `joy_devices_init` | 10 functions |
| `g_engine` | `0x006f9528` | the `IGraph` object `LS3DF.dll`'s `GetIGraph()` returns (§6a) | `engine_init`, `engine_shutdown` | 76 functions |
| `g_ls3d_setup` | `0x006f952c` | the persisted settings block, §1a; each field has its own symbol, so this row counts the block's own uses | - | `engine_igraph_error_callback`, `engine_init`, `engine_main`, `ls3d_setup_static_init`, `thunk_ls3d_setup_static_init` |

### 1a. `S_ls3d_setup` — the persisted engine settings block, `0x3f` bytes

`0x006f952c`, packed, written to and read from the registry value `LS3D_setup` under `HKLM\Software\Illusion Softworks\Mafia`. The
fields are the globals `g_ls3d_setup_<field>`; the name is the game's own name for the block, and `display_` is taken by the
`C_mission + 0x90` sub-object. The **bold** words of §2a are the captions of the setup program's dialog (§9). The block holds no array.

```c
#pragma pack(1)
typedef struct S_ls3d_setup {
    /* +0x00 */ uint8_t version_tag;             // [H] 0x14; stamped by both default fillers, read by the test of §9
    /* +0x01 */ uint8_t adapter_index;           // [H] the setup program's display-adapter list
    /* +0x02 */ int32_t width;                   // [H] screen width, default 800
    /* +0x06 */ int32_t height;                  // [H] screen height, default 600
    /* +0x0a */ int32_t bpp;                     // [H] bits per pixel, default 32
    /* +0x0e */ int32_t refresh_rate;            // [H] the refresh-rate list, default 0
    /* +0x12 */ uint8_t antialiasing;            // [H] fullscene antialiasing: 2 or 4, or 0 for off
    /* +0x13 */ uint8_t multipass_rendering;     // [H] mode bit 0x4000
    /* +0x14 */ uint8_t clip_always;             // [H] mode bit 0x8000
    /* +0x15 */ uint8_t fullscreen;              // [H] mode bit 0x2
    /* +0x16 */ uint8_t hw_transform_lighting;   // [H] mode bit 0x10, default 1
    /* +0x17 */ uint8_t vsync;                   // [H] mode bit 0x100
    /* +0x18 */ uint8_t low_detail_textures;     // [H] I3D driver state 0x15
    /* +0x19 */ uint8_t triple_buffering;        // [H] mode bit 0x4
    /* +0x1a */ uint8_t truecolor_lightmaps;     // [H] I3D driver state 0x22, default 1
    /* +0x1b */ uint8_t compressed_textures;     // [H] I3D driver state 0x19, default 1
    /* +0x1c */ uint8_t truecolor_textures;      // [H] I3D driver state 0x11
    /* +0x1d */ uint8_t sound_disabled;          // [H] the inverse of the Sound check box
    /* +0x1e */ uint8_t eax_disabled;            // [H] sound-driver init flag 0x10, default 1
    /* +0x1f */ uint8_t suspend_when_inactive;   // [H] mode bit 0x20, default 1
    /* +0x20 */ uint8_t software_mixing_only;    // [H] sound-driver init flag 0x20
    /* +0x21 */ uint8_t speaker_config;          // [H] the speaker list, default 1; read by the setup program only (§9)
    /* +0x22 */ int32_t language;                // [H] the language list, default 0; read by the setup program only (§9)
    /* +0x26 */ uint8_t wbuffer_enabled;         // [H] the W Buffer check box, default 1
    /* +0x27 */ float details_scale;             // [H] persisted mirror of C_mission + 0x90, default 1.0
    /* +0x2b */ float draw_distance_scale;       // [H] persisted mirror of C_mission + 0x94, default 1.0
    /* +0x2f */ int32_t effects_quality;         // [H] persisted mirror of C_mission + 0x9c, default 2
    /* +0x33 */ int32_t sounds_quality;          // [H] persisted mirror of C_mission + 0xa0, default 2
    /* +0x37 */ int32_t shadow_quality_idx;      // [H] persisted mirror of C_mission + 0xa4, default 3
    /* +0x3b */ int32_t particles_quality;       // [H] persisted mirror of C_mission + 0xa8, default 2
} S_ls3d_setup;
#pragma pack()
```

## 2. Field evidence

Every cell is the census of accesses to `[0x006f952c, 0x006f956b)` in `.text`. Besides these, `engine_main` writes every field by the
block copy at `0x005f9ab2` (15 dwords, a word and a byte), and `engine_main` (`0x005fa219`) and `engine_igraph_error_callback`
(`0x005f9927`) read every field when they pass the block to `RegSetValueExA`.

### 2a. `S_ls3d_setup`

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x0 | `version_tag` | H | `ls3d_setup_static_init` | `engine_main` | the `0x14` test is on this global, not on the registry copy (§9) |
| +0x1 | `adapter_index` | H | `ls3d_setup_static_init` | `engine_init` | passed as element [1] of the `GetIGraph()` init struct |
| +0x2 | `width` | H | `ls3d_setup_static_init` | `engine_init` | the RIGHT member of two `SetRect` calls |
| +0x6 | `height` | H | `ls3d_setup_static_init` | `engine_init` | the BOTTOM member of the same two |
| +0xa | `bpp` | H | `ls3d_setup_static_init` | `engine_init` | copied into the `IGraph` init struct |
| +0xe | `refresh_rate` | H | `ls3d_setup_static_init` | `engine_init` | the **refresh rate** list; a dword copied into the `IGraph` init struct beside `bpp` |
| +0x12 | `antialiasing` | H | `ls3d_setup_static_init` | `engine_init` | the **fullscene antialiasing** list; compared with 4 and 2 -> mode bits `0x2000` / `0x1000` ¹ |
| +0x13 | `multipass_rendering` | H | `ls3d_setup_static_init` | `engine_init` | **Multipass rendering** ¹ |
| +0x14 | `clip_always` | H | `ls3d_setup_static_init` | `engine_init` | **Clip always** ¹ |
| +0x15 | `fullscreen` | H | `ls3d_setup_static_init` | `engine_init` | **Fullscreen** ¹ |
| +0x16 | `hw_transform_lighting` | H | `ls3d_setup_static_init` | `engine_init` | **Hardware Transform & Lighting** ¹ |
| +0x17 | `vsync` | H | `ls3d_setup_static_init` | `engine_init` | **Vertical synchronization** ¹ |
| +0x18 | `low_detail_textures` | H | `ls3d_setup_static_init` | `engine_init` | **Low detail** textures |
| +0x19 | `triple_buffering` | H | `ls3d_setup_static_init` | `engine_init` | **Triple buffering** ¹ |
| +0x1a | `truecolor_lightmaps` | H | `ls3d_setup_static_init` | `engine_init` | **Truecolor** lightmaps |
| +0x1b | `compressed_textures` | H | `ls3d_setup_static_init` | `engine_init` | **Compressed** textures |
| +0x1c | `truecolor_textures` | H | `engine_igraph_error_callback`, `ls3d_setup_static_init` | `engine_init` | **Truecolor** textures ² |
| +0x1d | `sound_disabled` | H | `ls3d_setup_static_init` | `engine_init`, `game_media_create` | the **Sound** check box: `engine_init` calls `ISndGetDriver()` only when it is 0, and `game_media_create` reads it too |
| +0x1e | `eax_disabled` | H | `ls3d_setup_static_init` | `engine_init` | **Disable EAX** |
| +0x1f | `suspend_when_inactive` | H | `ls3d_setup_static_init` | `engine_init` | **Suspend when inactive** ¹ |
| +0x20 | `software_mixing_only` | H | `ls3d_setup_static_init` | `engine_init` | **Software mixing only** |
| +0x21 | `speaker_config` | H | `ls3d_setup_static_init` | - | the **speaker configuration** list; a scan of `.text` for `0x006f954d` finds only the default filler's store, so `Game.exe` never reads it; `Setup.exe`'s dialog reads and writes it (§9) |
| +0x22 | `language` | H | `ls3d_setup_static_init` | - | the **language** list; the same scan for `0x006f954e..0x006f9551` finds only the default filler's store; `engine_init` takes the language from `tables\version.txt`, not from this field; `Setup.exe` reads and writes it (§9) |
| +0x26 | `wbuffer_enabled` | H | `ls3d_setup_static_init` | `engine_set_wbuffer` | the **W Buffer** check box ³ |
| +0x27 | `details_scale` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | `structs/mission.md` §1a; restored through `mission_settings_set_details_scale` |
| +0x2b | `draw_distance_scale` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | restored through `mission_settings_set_draw_distance_scale` |
| +0x2f | `effects_quality` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | restored through `mission_settings_set_effects_quality` |
| +0x33 | `sounds_quality` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | restored through `mission_settings_set_sounds_quality` |
| +0x37 | `shadow_quality_idx` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | restored through `mission_settings_set_shadow_quality` |
| +0x3b | `particles_quality` | H | `engine_main`, `ls3d_setup_static_init` | `engine_main` | restored through `mission_settings_set_particles_quality` |

¹ The `IGraph` mode word `engine_init` assembles: `antialiasing` vs 4 and 2 -> `0x2000` / `0x1000`, `multipass_rendering` -> `0x4000`,
`clip_always` -> `0x8000`, `fullscreen` -> `2`, `hw_transform_lighting` -> `0x10`, `vsync` -> `0x100`, `triple_buffering` -> `4`,
`suspend_when_inactive` -> `0x20`, plus the constant `0x10090`.
² `engine_igraph_error_callback` clears it before re-saving the blob on a video-memory overflow; the setup program likewise warns that
truecolor textures are recommended only for cards with 64 MB of video memory or more.
³ Written 1 by both `ls3d_setup_static_init` and `ls3d_setup_set_defaults`; its only reader is `engine_set_wbuffer`, which ANDs a caller's
request with it before setting I3D render state 4.

## 3. Method family

The Win32 unit has no PS2 counterpart (§11); every PS2 function of `Load_progress.cpp` and of `Main.cpp`'s code half that this doc lists
has its row below, and `C_I3D_cache<T>`'s members stay in `structs/i3d_cache.md`. Callsite counts are byte scans of `.text` for
`E8 rel32` calls, not decomp greps. A `-` twin is PC-only.

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x005f79a0 | `crt_static_init_guard_005f79a0` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard of the shared flag bit, then its registrar; the first of `Load_progress.cpp`'s two |
| 0x005f79c0 | `crt_atexit_register_005f79c0` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |
| 0x005f79d0 | `crt_static_init_guard_005f79d0` | `g_crt_static_init_flags` | - | - | - | the second guard of `Load_progress.cpp` |
| 0x005f79f0 | `crt_atexit_register_005f79f0` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |
| 0x005f7bc0 | `crt_static_init_guard_005f7bc0` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard opening `Main.cpp` |
| 0x005f7be0 | `crt_atexit_register_005f7be0` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |
| 0x005f7bf0 | `thunk_ls3d_setup_static_init` | `g_ls3d_setup` | - | - | - | the `.CRT$XCU` slot's jump to the initializer |
| 0x005f7c00 | `ls3d_setup_static_init` | `g_ls3d_setup` | - | - | - | fills the global with the defaults of §1a |
| 0x005f7cd0 | `mission_static_init` | `g_mission` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f7ce0 | `mission_static_ctor` | `g_mission` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f7cf0 | `mission_register_atexit_dtor` | `g_mission` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f7d00 | `mission_static_dtor` | `g_mission` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f7d10 | `i3d_model_cache_static_init` | `g_i3d_model_cache` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f7d20 | `i3d_model_cache_static_ctor` | `g_i3d_model_cache` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f7d30 | `i3d_model_cache_register_atexit_dtor` | `g_i3d_model_cache` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f7d40 | `i3d_model_cache_static_dtor` | `g_i3d_model_cache` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f7d50 | `i3d_model_cache_dtor` | `g_i3d_model_cache` | `__dt__17C_I3D_model_cacheFv` | 0x00378430 | CONFIRMED | destroys the cache |
| 0x005f7e70 | `i3d_model_cache_ctor` | `g_i3d_model_cache` | - | - | - | constructs the cache |
| 0x005f7eb0 | `i3d_model_cache_scalar_deleting_dtor` | `g_i3d_model_cache` | - | - | - | the MSVC scalar deleting destructor |
| 0x005f7ed0 | `i3d_sound_cache_static_init` | `g_i3d_sound_cache` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f7ee0 | `i3d_sound_cache_static_ctor` | `g_i3d_sound_cache` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f7ef0 | `i3d_sound_cache_register_atexit_dtor` | `g_i3d_sound_cache` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f7f00 | `i3d_sound_cache_static_dtor` | `g_i3d_sound_cache` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f7f10 | `i3d_sound_cache_dtor` | `g_i3d_sound_cache` | `__dt__17C_I3D_sound_cacheFv` | 0x003781a0 | CONFIRMED | destroys the cache |
| 0x005f8020 | `i3d_sound_cache_ctor` | `g_i3d_sound_cache` | - | - | - | constructs the cache |
| 0x005f8060 | `i3d_sound_cache_scalar_deleting_dtor` | `g_i3d_sound_cache` | - | - | - | the MSVC scalar deleting destructor |
| 0x005f8080 | `i3d_anim_cache_static_init` | `g_i3d_anim_cache` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f8090 | `i3d_anim_cache_static_ctor` | `g_i3d_anim_cache` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f80a0 | `i3d_anim_cache_register_atexit_dtor` | `g_i3d_anim_cache` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f80b0 | `i3d_anim_cache_static_dtor` | `g_i3d_anim_cache` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f80c0 | `i3d_anim_cache_dtor` | `g_i3d_anim_cache` | `__dt__16C_I3D_anim_cacheFv` | 0x00377fa0 | CONFIRMED | destroys the cache |
| 0x005f81d0 | `i3d_anim_cache_ctor` | `g_i3d_anim_cache` | - | - | - | constructs the cache |
| 0x005f8210 | `i3d_anim_cache_scalar_deleting_dtor` | `g_i3d_anim_cache` | - | - | - | the MSVC scalar deleting destructor |
| 0x005f8230 | `input_static_init` | `g_input` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f8240 | `input_static_ctor` | `g_input` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f8250 | `input_register_atexit_dtor` | `g_input` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f8260 | `input_static_dtor` | `g_input` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f8270 | `joy_device_list_static_init` | `g_joy_device_list` | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f8280 | `joy_device_list_static_ctor` | `g_joy_device_list` | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f82a0 | `joy_device_list_register_atexit_dtor` | `g_joy_device_list` | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f82b0 | `joy_device_list_static_dtor` | `g_joy_device_list` | - | - | - | loads the object and tail-jumps to its destructor |
| 0x005f82d0 | `screen_fade_static_init` | the screen-fade object | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f82e0 | `screen_fade_static_ctor` | the screen-fade object | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f8310 | `screen_fade_register_atexit_dtor` | the screen-fade object | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f8330 | `thunk_stub_noop_cdecl_005f8340` | - | - | - | - | the `.CRT$XCU` slot of the first `S_vector`, whose constructor is empty |
| 0x005f8340 | `stub_noop_cdecl_005f8340` | - | - | - | - | the empty constructor |
| 0x005f8350 | `thunk_stub_noop_cdecl_005f8360` | - | - | - | - | the slot of the second `S_vector` |
| 0x005f8360 | `stub_noop_cdecl_005f8360` | - | - | - | - | the empty constructor |
| 0x005f8370 | `force_feedback_manager_static_init` | the force-feedback manager | - | - | - | the `.CRT$XCU` initializer: the constructor call, then the destructor's registrar |
| 0x005f8380 | `force_feedback_manager_static_ctor` | the force-feedback manager | - | - | - | loads the object and tail-jumps to its constructor |
| 0x005f8390 | `force_feedback_manager_register_atexit_dtor` | the force-feedback manager | - | - | - | `crt_atexit(static_dtor)` |
| 0x005f83a0 | `force_feedback_manager_static_dtor` | the force-feedback manager | - | - | - | loads the object and tail-jumps to its destructor |
| 0x00623e10 | `crt_static_init_guard_00623e10` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard opening the unit of `winmain` |
| 0x00623e30 | `crt_atexit_register_00623e30` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |
| 0x00623ea0 | `crt_static_init_guard_00623ea0` | `g_crt_static_init_flags` | - | - | - | `.CRT$XCU` guard opening the unit of the Win32 helpers |
| 0x00623ec0 | `crt_atexit_register_00623ec0` | `g_crt_static_init_flags` | - | - | - | `crt_atexit(crt_atexit_noop_dtor_0060b700)` |

### 3b. Accessors

These units have no accessor.

### 3c. Behaviour

These units have no function with a receiver; `engine_init`'s two registers are `__fastcall` arguments (§10).

### 3d. Free helpers

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x005f7a00 | `load_progress_callback` | `cbLoadScene__F21LS3D_CALLBACK_MESSAGEUiUiPv` | 0x00372e40 | CONFIRMED | The LS3D load callback: creates the screen in phase 0, reports the fraction with a `0.02` throttle in phase 1, destroys it in phase 2 |
| - | - | `InitProgressScreen__Fv` | 0x00372dc0 | inlined | inlined into `load_progress_callback`'s phase-0 arm, which allocates and presents the screen |
| 0x005f83b0 | `engine_set_wbuffer` | `SetWBuffer__Fb` | 0x00372fc0 | CONFIRMED | I3D render state 4, gated by the persisted setting |
| 0x005f83e0 | `joy_devices_init` | - | - | - | `ijoy.dll` bring-up; fills `g_joy_device_list` |
| 0x005f8660 | `joy_devices_done` | - | - | - | releases them |
| 0x005f86c0 | `engine_draw_overlay_callback` | `cbScene__F20SCN_CALLBACK_MESSAGEUiUi` | 0x00373020 | CONFIRMED | scene callback **id 6** |
| 0x005f8700 | `engine_new_mission` | `InternMissionInit__Fv` | 0x00374250 | CONFIRMED | the queued mission change; `structs/mission.md` §7 |
| 0x005f9540 | `engine_game_loop` | `GameLoopSingle__Fv` | 0x00375090 | CONFIRMED | the single-player in-game loop |
| 0x005f98e0 | `engine_igraph_error_callback` | - | - | - | IGraph error hook, out-of-memory only ¹ |
| 0x005f99b0 | `engine_main` | - | - | - | the real main; called by `winmain` (§9) |
| 0x005fa2a0 | `ls3d_setup_set_defaults` | - | - | - | writes the defaults into the stack buffer the registry value then overwrites |
| 0x005fa320 | `engine_init` | `InitSystem__FPvRb` | 0x00375780 | CONFIRMED | dta archives, IGraph, sound, I3D driver, mission, joysticks, the three cache directories, tables, language, network session, logos, load-save init (§10) |
| 0x005fb330 | `engine_show_init_error` | `InitErrorMessageBox__F11LS3D_RESULTi` | 0x00375510 | CONFIRMED | the 21-arm `LS3D_RESULT` message box |
| 0x005fb4f0 | `engine_shutdown` | `CloseSystem__Fv` | 0x00376df0 | CONFIRMED | the mirror of `engine_init` |
| 0x00623e40 | `winmain` | - | - | - | `GetModuleFileNameA`, strips the file name, `crt_chdir`, `engine_main`; called by `crt_startup` |
| 0x00623ed0 | `win_message_box` | - | - | - | modal message box with a mapped button code and a normalized result; 1 callsite (§6) |
| 0x00623f80 | `win_load_string_resource` | - | - | - | `LoadStringA` forwarder; 1 callsite |
| 0x00623fa0 | `win_set_app_icon` | - | - | - | `LoadIconA` + `SetClassLongA(GCL_HICON)`; 1 callsite |
| 0x00623fc0 | `cd_find_game_file` | - | - | - | resolves a data file: local install first, then the disc; 4 callsites (§5) |
| 0x00624070 | `cd_require_disc` | - | - | - | the blocking retry loop around the drive scan; 2 callsites |
| 0x006240b0 | `cd_file_exists` | - | - | - | `FindFirstFileA` existence test; 1 callsite |
| 0x006240e0 | `cd_scan_drives_for_disc` | - | - | - | the drive scan itself, **cracked in `Game.exe`** (§8); 1 callsite |
| 0x00624190 | `std_string_reserve` | - | - | - | a refcounted string-buffer resize, emitted in this object; 3 callsites |

¹ Zero `E8` callers and zero `.rdata` datarefs, yet live: `engine_init` stores its address at `0x005fa5ae`
(`&engine_igraph_error_callback`).

## 4. The translation units

The `.CRT$XCU` table holds three consecutive guard slots here: `0x006462f8` -> `0x00623e10`, `0x006462fc` -> `0x00623ea0` and `0x00646300`
-> `0x006242b0`. Each guard only sets bit 0 of the shared `g_crt_static_init_flags` and registers a no-op destructor
(`structs/crt_runtime_tail.md` §8e), and two adjacent slots guarding that bit are two consecutive units. So `winmain` sits in its own object,
`[0x00623e10, 0x00623ea0)`, and the `win_` / `cd_` helpers and `std_string_reserve` in the next, `[0x00623ea0, 0x006242b0)`. The file name
`Winstuff.cpp` (§11) belongs to the first slot, and nothing names the second (Open item 1).

### 4a. The three translation units around `engine_main`

The `.CRT$XCU` initializer table at `.data:0x00646264` gives the object-file boundaries; the spans
and the file names come from the PS2 side (§11).

| PC range | translation unit | what is in it |
|---|---|---|
| `[0x005dc160, 0x005f79a0)` | `g_game.cpp` | `C_game` |
| `[0x005f79a0, 0x005f7bc0)` | **`Load_progress.cpp`** | 2 guard-only initializers + `load_progress_callback` |
| `[0x005f7bc0, 0x005fe4f0)` | **`Main.cpp`** | 9 file-scope singletons, then `engine_main`, `engine_init`, `ls3d_setup_set_defaults`, the `C_I3D_cache<T>` implementation and the force-feedback manager |
| `[0x005fe4f0, 0x00608390)` | **`Mission.cpp`** | `C_mission`, the scene loader, the `display_*` accessors |

The PC link order across this stretch is alphabetical by source file — `g_game` < `Load_progress`
< `Main` < `Mission`.

**`Main.cpp`'s initializer block.**

`Main.cpp` is identified by matching its initializer block object-for-object, in order, against the
PS2's `__sinit_Main.cpp` (§11); a boundary alone proves nothing about a span.

| PC `.CRT$XCU` slot | PC object |
|---|---|
| `0x005f7bc0` | *(guard-only, ctor elided)* |
| `0x005f7bf0` | `g_ls3d_setup` `0x006f952c` (§9) |
| `0x005f7cd0` | `g_mission` `0x006f9440` |
| `0x005f7d10` | `g_i3d_model_cache` `0x006f9418` |
| `0x005f7ed0` | `g_i3d_sound_cache` `0x006f93f0` |
| `0x005f8080` | `g_i3d_anim_cache` `0x006f93c8` |
| `0x005f8230` | `g_input` `0x006f92e0` |
| `0x005f8270` | `g_joy_device_list` `0x006f92cc` |
| `0x005f82d0` | the screen-fade object `0x006f9278` (`structs/screen_fade.md`) |
| `0x005f8330` + `0x005f8350` | two `S_vector`s, empty ctors |
| `0x005f8370` | the force-feedback manager `0x006f91e0` |

## 5. The CD chain

```
cd_require_disc
  └─ cd_scan_drives_for_disc          for letter in 'C'..'Z'
       └─ cd_file_exists                  "<letter>:\MafiaGame\a1.dta" ?
```

**`cd_scan_drives_for_disc`** stamps the letter into `"X:\"` (`s_drive_root_template_0064d928`), calls
`GetDriveTypeA` (**result discarded** — EAX is overwritten by the next call without a read), then `GetVolumeInformationA` into a 0x80-byte volume-name
buffer and a 0x80-byte filesystem-name buffer. That call is tested only for success — media present
and readable — and **no volume label is ever compared.** The disc is identified purely by carrying
`MafiaGame\a1.dta` (format string `0x00651b0c`). Returns 1 on the first hit, 0 after `'Z'`.

**`cd_require_disc`** does not check, it **blocks**: on failure it presents a frame through
`g_engine` slot `+0x3c` (§6a), shows
`MessageBoxA("Please insert MAFIA CD 1", 0x15 = MB_RETRYCANCEL|MB_ICONHAND)`, and retries until the
disc appears or the user cancels, in which case it returns 0. Its two callers are `engine_main` and
`mission_open` (callsite `0x005fff62`).

**`cd_find_game_file`** is a separate path and is *not* part of that chain. It resolves one named
file: it first tries the bare name in the working directory with `CreateFileA`, and only on failure
walks `'C'`..`'Z'` building `"X:\MafiaGame\%s"` (`0x00651ae0`); this walk also calls
`GetDriveTypeA` and discards the result in the same way. The **resolved** path is written back into the caller's
buffer. Its four callers and their arguments, recovered from the disassembly because Ghidra hides
the `__fastcall` operands:

| callsite | argument | what the caller does with the resolved path |
|---|---|---|
| `0x005c6539` (`program_process_command`) | `"intro.avi"` | queries the screen size (`g_engine` slots `+0x6c` / `+0x70`), creates a video player through `g_engine` slot `+0x00` and opens the path with the player's `+0x0c` |
| `0x005fa411` (`engine_init`) | `"a1.dta"` | `dtaCreate(buf)`, stored in `g_dta_archives[3]` |
| `0x005fb214` (`engine_init`) | `"logo1.avi"` | creates a video player through `g_engine` slot `+0x00` and opens the path with the player's `+0x0c` |
| `0x005fb25c` (`engine_init`) | `"logo2.avi"` | the same as `logo1.avi` |

Every caller consumes the **buffer**, not just the boolean — which is what proves it is a path
resolver and not a predicate. The set of names is the assets that live on the disc rather than in
the install: the two logo videos, the intro video, and one archive.

## 6. The Win32 host helpers

**`win_message_box`** maps an internal button-set code to a Win32 type — `0`/default `MB_OK`,
`1`→`MB_YESNO`, `2`→`MB_YESNOCANCEL`, `3`→`MB_RETRYCANCEL`, `4`→`MB_OKCANCEL` — takes its parent HWND
from `g_engine` slot `+0x10`, the main-window handle (NULL when `g_engine` is null), and normalizes the result:
`IDRETRY`/`IDYES` → 1, `IDNO` → 0, `IDCANCEL` → 2, anything else → -1. Its single caller reports a
sound error (`"Sound type I3DSOUND_VOLUME not exist."` / `"ConstructSound"`). `cd_require_disc` does
**not** use it — it calls `MessageBoxA` directly with a literal `0x15`.

**`win_load_string_resource`** is a bare forwarder to `LoadStringA`. Its one caller, `engine_init`,
fetches **string id 3** into a 128-byte buffer and hands it to `g_engine` slot `+0x18`, which sets the application's window name.

**`win_set_app_icon`** calls `LoadIconA(hInstance, MAKEINTRESOURCE(1))` and, on success,
`SetClassLongA(hWnd, GCL_HICON, hIcon)`. `engine_init` passes the HWND it has just fetched from
`g_engine` slot `+0x10`.

### 6a. `g_engine` — the `IGraph` interface

`g_engine` (`0x006f9528`) is the object `engine_init` receives from `LS3DF.dll`'s `GetIGraph()`. The
slots this TU and its callers use are read off the DLL itself: `GetIGraph` returns a singleton whose
constructor installs the vtable at `LS3DF.dll:0x10099498`, and each slot's body says what it is.

| slot | what the `LS3DF.dll` body does | used by |
|---|---|---|
| `+0x00` | creates a video-player object (no argument; the caller opens a path with the object's `+0x0c`) | `engine_init` (the two logos), `program_process_command` (the intro) |
| `+0x10` | returns the main-window handle (a global the constructor zeroes) | `win_message_box`, `engine_init` (for `win_set_app_icon`) |
| `+0x14` | returns the child-window handle (a second global the constructor zeroes) | - |
| `+0x18` | sets the application name: compares the argument with the stored name, copies it and sets the window text | `engine_init` |
| `+0x3c` | presents the frame: a once-per-second frame counter, the device's present call, and on a lost device a sleep-250-ms-and-reset loop | `cd_require_disc` |

### 6b. The resources this TU reads

From `.rsrc`, which is **byte-identical in retail and in both cracked builds**:

| resource | value |
|---|---|
| `STRING/#1/#1029` id 1 | `Software\LS3D game reg\Setup` |
| `STRING/#1/#1029` id 2 | `Mafia` |
| **`STRING/#1/#1029` id 3** | **`Mafia`** — what `win_load_string_resource` fetches |
| `ICON/#1/#1029` | the icon `win_set_app_icon` installs |

## 7. Strings

| VA | content |
|---|---|
| `0x0064d928` | `X:\` — the root-path template both drive walks stamp a letter into |
| `0x00651ae0` | `X:\MafiaGame\%s` — `cd_find_game_file`'s disc path |
| `0x00651af0` | `Please insert MAFIA CD 1` |
| `0x00651b0c` | `%c:\MafiaGame\a1.dta` — the disc's identifying file |

## 8. `cd_scan_drives_for_disc` is cracked in `exe-re/Game.exe`

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

## 9. `g_ls3d_setup` — how the block is loaded and saved

Both strings are byte-read from `Game.exe`. `engine_main` passes the block to `RegSetValueExA` with `push 0x3f; push 0x6f952c` at
`0x005fa219`, and `engine_igraph_error_callback` does the same at `0x005f9927`. The six quality and scale mirrors are the persisted copy of
`C_mission + 0x90`, `structs/mission.md` §1a.

Defaults are written twice by the same source function: `ls3d_setup_static_init` (`0x005f7c00`) fills the global, and
`ls3d_setup_set_defaults` (`0x005fa2a0`) fills the stack buffer that `RegQueryValueExA` then overwrites.

**The setup program names every byte.** `Setup.exe`, shipped with the game, writes the same `0x3f`-byte value to the same registry key. Its
dialog (resource 101) binds one control to each byte: the code that fills the dialog from the block and the code that copies the dialog
back into it agree for all fifteen check boxes, and each list (adapter, resolution, bit depth, refresh rate, antialiasing, speaker
configuration, language) stores the selected item's value. The **bold** names of §2a are the captions of the dialog template; the Sound and
Fullscreen check boxes sit in the titles of their group boxes.

**A missing blob opens the setup program.** When the registry key, the value or its `0x3f`-byte size is missing, `engine_main` starts
`setup.exe` through `crt_spawnlp` with mode 1 and returns. Mode 1 does not wait (`crt_dospawn` waits only for mode 0), so the game process
ends while the setup program runs. In game, deleting the value and starting the game opens `setup.exe`.

**The blob's tag is not tested.** The test at `0x005f9aa5` is `cmp byte ptr [0x6f952c], 0x14`, on the global, which `ls3d_setup_static_init`
has set to `0x14`; the registry value is in the stack buffer at `[esp + 0x34]` at that point. So the test always passes, and the block copy
then overwrites the global with whatever tag the value holds (Open item 2).

## 10. The `engine_*` free-function family

`Main.cpp`'s code half `[0x005f83b0, 0x005fb6e0)` holds twelve free functions. `engine_` is a
**translation-unit** prefix, not a class prefix — as are `load_progress_` (2 functions),
`ls3d_setup_` (2), and this file's own `win_` / `cd_`. None of them is a `C_game` or `C_mission`
member.

`engine_init` is `__fastcall`: ECX = the instance handle, EDX = a flag set when the load-save init fails, on which `engine_main` goes straight to
`engine_shutdown`. `engine_game_loop` is a `do { } while (true)` loop.

## 11. PC vs PS2

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

**`Main.cpp` lockstep (§4a).** The PC link order `g_game` < `Load_progress` < `Main` < `Mission` is
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

**`engine_init` and `engine_shutdown` are ordered callee locksteps.** `InitSystem__FPvRb` makes
the same calls in the same order as `engine_init` (archives, the two rects, `GetIGraph`, the I3D
driver, the mission, the cursor, the three cache directories with their trailing backslash,
consoles, tyres, materials, particles, animation index, sound tables, the version file and the
language select, `G_LoadSaveClass::Init` writing the `bool&`), except that it creates the sound
driver after the mission; its `bool&` is the PC's EDX flag. `CloseSystem__Fv` matches
`engine_shutdown` the same way, load-save close first and `FreeTables` last. The PS2 `main` calls
`CloseSystem` from 12 places; `engine_main` calls `engine_shutdown` from one.

**The `IGraph` slots of §6a have PS2 names.** The PS2 statically links the same class
(`__vt__6IGraph`), but its vtable order differs from the PC DLL's (it starts with `Init`), so each
slot is matched by its BODY, not by position:

| PC slot | PS2 method | matched on |
|---|---|---|
| `+0x00` | `CreateIShow__6IGraphFv` | [M] shape only: no argument, returns the object the caller opens a video with |
| `+0x10` | `GetMainHWND__6IGraphFv` | returns `m_MainHWnd` |
| `+0x14` | `GetChildHWND__6IGraphFv` | returns the second window handle |
| `+0x18` | `SetAppName__6IGraphFPCc` | `strcmp` with `m_AppName`, `strncpy`, `SetWindowText(m_MainHWnd, …)` |
| `+0x3c` | `Present__6IGraphFv` | `m_ShowFPS` counter against `timeGetTime` (1000 ms), device present with four null arguments, the `D3DERR_DEVICELOST` (`0x88760868`) sleep-250 / reset loop |

## 12. Related

[`basic/artifacts/dll.md`](../basic/artifacts/dll.md): SafeDisc's own import table in the retail exe names `LS3DF.dll`, `IJoy.dll`, `Inet.dll` and
`rw_data.dll` in the developers' original casing, independently corroborating that document's four Illusion DLLs. `basic/artifacts/game-loop.md`
walks `engine_main`. `structs/speech.md` is another unit-scoped doc (`g_dabing.cpp`).

## 13. Open items

| Item | State |
|---|---|
| 1. The object `[0x00623ea0, 0x006242b0)` | its own `.CRT$XCU` slot and no source name; which file holds the Win32 helpers is not measured |
| 2. in game: a `LS3D_setup` value of the right size and a wrong first byte | §9 expects the game to start without opening `setup.exe`; not observed |
