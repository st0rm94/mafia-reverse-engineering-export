# `GM_Carcyclopedia` — the main-menu Carcyclopedia screen

**Created** 2026-08-23 20:50 +0200 · **Modified** 2026-11-04 16:45 +0100

The **Carcyclopedia**, the in-game car encyclopedia reached from the main menu: one car at a time on
a rotating turntable with its name, colour name and description; the player pages through every car
the profile has unlocked and cycles its paint colours. In-game label: textdb **110 / 4083 / 4087** =
`"Carcyclopedia"` (de *Autozyklopädie*, cz *Auto encyklopedie*). The class is `GM_Carcyclopedia` on both builds.

| | |
|---|---|
| PC prefix | `carcyclopedia_` |
| PC kind (`+0x10`) | - |
| PC vtable | `0x0063d378`, 6 slots (§5) |
| PC base class | `GM_Menu` (`structs/gui_screen.md`): the ctor calls `gui_menu_ctor` first, the dtor ends in `gui_menu_dtor` |
| PC instance size | `0x9c` (`operator_new(0x9c)` in `engine_main`) |
| PC constructor | `carcyclopedia_ctor` @ `0x00561b70` |
| PC translation unit | `[0x00561b40, 0x00563b00)`, guard to guard, hosted here; it also holds the multiplayer person picker and one `GM_Menu` default (§7) |
| PS2 class | `GM_Carcyclopedia` (tag `16GM_Carcyclopedia`), 9 methods: 5 twins, 3 folded into one PC function, 1 with none; `g_m_carc.cpp` also holds the PS2-only `G_CarcCache` (§8) |

**Confidence tags:** **[H]** body-read + writer + reader · **[M]** inferred · **[U]** not observed.

## 1. Base field layout

Only the class's own band; `+0x00..+0x44` is the `GM_Menu` base (`structs/gui_screen.md` §1). The
class uses two base fields by name: `+0x18`, the PARENT screen, to which `carcyclopedia_on_exchange`
forwards the rebuild ¹, and the framework flag word `+0x40`, whose bit `0x00010000` the ctor clears.

```c
typedef struct GM_Carcyclopedia {             // 0x9c bytes; GM_Menu base +0x00..+0x44
    /* +0x48 */ uint32_t     selected;        // [H] the selected car, an index into both vectors
    /* +0x4c */ uint32_t     color;           // [H] the paint-colour index
    /* +0x50 */ I3D_model   *car_model;       // [H] the showroom car, driver object type 9
    /* +0x54 */ uint32_t     unused_54;       // [U] no access; no reader (.text scan)
    /* +0x58 */ I3D_frame   *turntable;       // [H] the NGon02 frame
    /* +0x5c */ I3D_frame   *cam_near;        // [H] the autosalon frame: camera at blend 1
    /* +0x60 */ I3D_frame   *cam_far;         // [H] the autosalon2 frame: camera at blend 0
    /* +0x64 */ float        yaw;             // [H] turntable yaw, degrees, wrapped into [0, 360)
    /* +0x68 */ float        unread_68;       // [U] 60.0 by carcyclopedia_ctor; no reader (.text scan)
    /* +0x6c */ float        blend;           // [H] camera blend 0..1; ctor 1.0
    /* +0x70 */ float        spin_speed;      // [H] turntable degrees per second; ctor 16.666
    /* +0x74 */ uint32_t     view_mode;       // [H] 1 = list mode, 2 = show mode (§2, §4)
    /* +0x78 */ uint8_t      ids_alloc;       // [H] allocator byte of the car-id vector
    /* +0x7c */ uint32_t    *ids_begin;       // [H] packed car ids
    /* +0x80 */ uint32_t    *ids_end;         // [H] one past the last car id
    /* +0x84 */ uint32_t    *ids_cap;         // [H] end of the id vector's storage
    /* +0x88 */ uint8_t      names_alloc;     // [H] allocator byte of the car-name vector
    /* +0x8c */ char       **names_begin;     // [H] display names, index-parallel to the ids
    /* +0x90 */ char       **names_end;       // [H] one past the last display name
    /* +0x94 */ char       **names_cap;       // [H] end of the name vector's storage
    /* +0x98 */ uint8_t      roof_shown;      // [H] ctor 0; the roof toggle
} GM_Carcyclopedia;
```

The two vectors are **index-parallel**, which is why every consumer takes the single selection
index `+0x48`.

¹ The field holds the parent, not a child: the framework's slot-4 default `gui_menu_close` assigns
`this+0x18` to `g_gui_active_screen` on close and then notifies it through slot 3; every screen
constructor in this family is called as `ctor(g_gui_active_screen, …)`; nothing in this class
releases a child screen through it (`structs/multiplayer.md` §10).

## 2. Field evidence

Measured by a census of every `this + N` access in the `carcyclopedia_*` functions plus their
disassembly (the allocator bytes are byte stores the text census does not see).

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x48 | selected | H | `carcyclopedia_ctor`, `carcyclopedia_tick` | `carcyclopedia_refresh_display`, `carcyclopedia_tick` | passed to `carcyclopedia_refresh_car` |
| +0x4c | color | H | `carcyclopedia_ctor`, `carcyclopedia_tick`, `carcyclopedia_refresh_car` | `carcyclopedia_tick`, `carcyclopedia_refresh_car` | clamped against `car_database_get_max_color_index`; cycled by the left / right arrows |
| +0x50 | car_model | H | `carcyclopedia_ctor` | `carcyclopedia_ctor`, `carcyclopedia_on_close`, `carcyclopedia_refresh_car` | reloaded per car by `carcyclopedia_refresh_car`, released by `carcyclopedia_on_close` |
| +0x54 | unused_54 | U | - | - | a 4-byte gap: no access in the class and no hit through a holder of the screen (.text scan) |
| +0x58 | turntable | H | `carcyclopedia_ctor` | `carcyclopedia_ctor`, `carcyclopedia_tick` | frame-by-name lookup with mask `0xffff`; the tick sets its direction from `yaw` |
| +0x5c | cam_near | H | `carcyclopedia_ctor` | `carcyclopedia_ctor`, `carcyclopedia_tick` | the ctor places the camera at its world position and aims it along its normalised forward row; the tick reads its world position (`frame+0x40`) and forward row (`frame+0x30`) |
| +0x60 | cam_far | H | `carcyclopedia_ctor` | `carcyclopedia_tick` | the ctor only stores it; the tick reads its world position (`frame+0x40`) and forward row (`frame+0x30`) |
| +0x64 | yaw | H | `carcyclopedia_ctor`, `carcyclopedia_tick` | `carcyclopedia_tick` | `sin` / `cos` of `yaw * 0.017453292` |
| +0x68 | unread_68 | U | `carcyclopedia_ctor` | - | `0x42700000` = 60.0, the ctor's store at `0x00561d0d` is the only access (.text scan) ² |
| +0x6c | blend | H | `carcyclopedia_ctor`, `carcyclopedia_tick` | `carcyclopedia_tick` | §4 |
| +0x70 | spin_speed | H | `carcyclopedia_ctor`, `carcyclopedia_tick` | `carcyclopedia_tick` | §4 |
| +0x74 | view_mode | H | `carcyclopedia_ctor` (1), `carcyclopedia_tick` | `carcyclopedia_on_exchange`, `carcyclopedia_tick` | 1 = list mode, 2 = show mode. The tick toggles it on key `0x23` in either mode and on `0x1c` in show mode, then rebuilds through vtable slot 1; `carcyclopedia_on_exchange` passes it as the `gate` argument of `gui_menu_build_components_from_design`. Design `0xd` of `menu.def`: gate 1 holds the list (`0x14`), "Select Car" and textdb 559 "H - show mode"; gate 2 holds only textdb 560 "Cursor to move, H - list mode" (§4) |
| +0x78 | ids_alloc | H | `carcyclopedia_ctor` | - | the header `car_database_collect_cars_by_carcyclopedia_mask` fills |
| +0x7c | ids_begin | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `std_vector4_insert_n` (through `+0x78`, called by `car_database_collect_cars_by_carcyclopedia_mask`) | `carcyclopedia_dtor`, `carcyclopedia_tick`, `carcyclopedia_build_car_list`, `carcyclopedia_refresh_car` | freed second by the dtor |
| +0x80 | ids_end | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `car_database_collect_cars_by_carcyclopedia_mask`, `std_vector4_insert_n` (both through `+0x78`) | `carcyclopedia_tick`, `carcyclopedia_build_car_list`, `carcyclopedia_refresh_car` | - |
| +0x84 | ids_cap | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `std_vector4_insert_n` (through `+0x78`, called by `car_database_collect_cars_by_carcyclopedia_mask`) | `std_vector4_insert_n` (through `+0x78`) | - |
| +0x88 | names_alloc | H | `carcyclopedia_ctor` | - | `carcyclopedia_refresh_display` hands its address to the list widget |
| +0x8c | names_begin | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `carcyclopedia_build_car_list` | `carcyclopedia_dtor`, `carcyclopedia_build_car_list` | freed FIRST by the dtor |
| +0x90 | names_end | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `carcyclopedia_build_car_list` | `carcyclopedia_build_car_list` | - |
| +0x94 | names_cap | H | `carcyclopedia_ctor`, `carcyclopedia_dtor`, `carcyclopedia_build_car_list` | `carcyclopedia_build_car_list` | - |
| +0x98 | roof_shown | H | `carcyclopedia_ctor`, `carcyclopedia_tick` | `carcyclopedia_tick`, `carcyclopedia_refresh_car` | widget command `0x79` flips it and calls `car_set_roof_visible`; `carcyclopedia_refresh_car` re-applies it to every car it shows |

² A sweep of the class's own code finds no other access, and the `GM_Menu` base fields end at
`+0x44`, so no base method reads it either.

## 3. Method family

5 of the 9 PS2 `GM_Carcyclopedia` methods are twins (two PC functions share `OnCreate`), three more
are folded into `carcyclopedia_tick`, and `OnBeforeCreate` has no PC function (§8i).

### 3a. Construction & destruction

| PC VA | PC name | Object | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|---|
| 0x00561b40 | `crt_static_init_guard_00561b40` | the translation unit | - | - | - | the unit's `.CRT$XCU` initializer |
| 0x00561b60 | `crt_atexit_register_00561b60` | the translation unit | - | - | - | its `atexit` registrar |
| 0x00561b70 | `carcyclopedia_ctor` | instance | `OnCreate__16GM_CarcyclopediaFv` | 0x002c05a0 | CONFIRMED | `gui_menu_ctor`, the vtable, the frame lookups, the showroom model, the defaults of §2; 1120 B against 1348 |
| 0x00561fd0 | `carcyclopedia_scalar_deleting_dtor` | instance | - | - | - | slot 0; an MSVC-only shape |
| 0x00561ff0 | `carcyclopedia_dtor` | instance | `__dt__16GM_CarcyclopediaFv` | 0x002c1a80 | CONFIRMED | frees the name vector, then the id vector, then `gui_menu_dtor` (§8a) |

### 3b. Accessors

The class has no accessor.

### 3c. Behaviour

| PC VA | PC name | PS2 twin | PS2 VA | Status | Role |
|---|---|---|---|---|---|
| 0x00562040 | `carcyclopedia_on_close` | `OnClose__16GM_CarcyclopediaFv` | 0x002c0c00 | CONFIRMED | slot 2; releases the showroom model |
| 0x00562080 | `carcyclopedia_on_exchange` | `OnExchange__7GM_MenuFv` | 0x002c1b60 | STRUCTURAL | slot 1; rebuilds in place (language / resolution change) and forwards to the parent; the PS2 twin is the framework default the class does not override |
| 0x005620b0 | `carcyclopedia_refresh_display` | - | - | - | refills the list widget `0x14` and refreshes the car; PC-only factoring, the PS2 does it inline |
| 0x00562120 | `carcyclopedia_tick` | `OnTick__16GM_CarcyclopediaFUi` | 0x002c0c70 | CONFIRMED | slot 5; per-frame turntable and camera, keys, mouse and widget commands: a 4 → 1 fold (§8d) |
| - | - | `OnKey__16GM_CarcyclopediaFUcUi` | 0x002c1180 | inlined | folded into `carcyclopedia_tick`: the arrow keys |
| - | - | `OnClick__16GM_CarcyclopediaFUi` | 0x002c1480 | inlined | folded into `carcyclopedia_tick`: the widget commands |
| - | - | `OnMouse__16GM_CarcyclopediaFiiiUiUi` | 0x002c1500 | inlined | folded into `carcyclopedia_tick`: the drag on the turntable and the camera |
| 0x00562bb0 | `carcyclopedia_build_car_list` | `OnCreate__16GM_CarcyclopediaFv` | 0x002c05a0 | CONFIRMED | the list-building half of `OnCreate`: the unlocked-car mask (§8e), the collector, then one name per id |
| 0x00562da0 | `carcyclopedia_refresh_car` | `RefreshCar__16GM_CarcyclopediaFiib` | 0x002c1720 | CONFIRMED | loads the selected car in the selected colour and fills the widgets of §6 |

### 3d. Free helpers

The class calls no free helper of its unit.

## 4. How the screen runs

`engine_main` reaches it from main-menu state **`0x19`**, the entry immediately after the credits
(`0x17` → `loadsave_execute_credits`):

```
mission_close()
mission_open("autosalon", 0, 0xffffffff, 1)
collision_load_level("missions\autosalon\tree.klz", 1)
game_init()
operator_new(0x9c) -> carcyclopedia_ctor(obj, 0, 0)
gui_menu_execute_modal(1)
mission_close()
```

🔑 **`autosalon` is the SCENE, not the class.** The screen runs inside a car-showroom set whose scene
directory is `missions\autosalon\`; the frames `autosalon` and `autosalon2` are the two camera
placements the screen blends between, and `NGon02` is the turntable.

Per frame `carcyclopedia_tick` adds `spin_speed · dt` to `yaw`, wraps it into `[0, 360)` and sets
the turntable's direction from it; it relaxes `spin_speed` back toward 16.67 and sets it from a
horizontal drag. It clamps `blend`, moves it on a vertical drag (0.005 per pixel), and places the
active camera (`I3D_scene+0x17c`) between the `cam_far` (0) and `cam_near` (1) positions.

The arrow keys depend on `view_mode` (§2), which key `0x23` (H) toggles in either mode and `0x1c`
(Enter) leaves in show mode:

| `view_mode` | Up / down (`0xc8` / `0xd0`) | Left / right (`0xcb` / `0xcd`) | Widgets shown |
|---|---|---|---|
| 1, list mode | previous / next car | previous / next colour | the car list `0x14`, "Select Car", textdb 559 "H - show mode" |
| 2, show mode | move `blend` | set `spin_speed` to −100 / +100 and turn the turntable | textdb 560 "Cursor to move, H - list mode" |

## 5. The vtable — and the framework's slot layout

The PC screen vtable at `0x0063d378` is **six slots**. Three sibling tables in the neighbouring
cluster (`0x0063d398`, `0x0063d3b4`, `0x0063d3cc`) have the same shape, which makes the layout a
measurement rather than a guess:

| Slot | This class | Meaning |
|---|---|---|
| 0 | `carcyclopedia_scalar_deleting_dtor` | MSVC `delete` entry |
| 1 | `carcyclopedia_on_exchange` | re-create in place (language / resolution change) |
| 2 | `carcyclopedia_on_close` | teardown |
| 3 | `gui_menu_on_close_child` (framework default, 32 vtables) | not overridden here |
| 4 | `gui_menu_close` (framework default, 49 vtables) | not overridden here |
| 5 | `carcyclopedia_tick` | per-frame update, keyboard and widget commands |

Slot 1's framework default is `gui_menu_on_exchange_forward_to_parent`, which occupies that slot in
**45 vtables**, so a function in this VA range is not automatically this class's (§7).

## 6. The widget ids the screen drives

| Element | Set by | Content |
|---|---|---|
| `0x14` | `carcyclopedia_refresh_display`, `carcyclopedia_tick` | the scrolling car list; its backing list is the name vector `+0x88` |
| `0x32` | `carcyclopedia_refresh_car` | the car name (`car_database_get_car_name`) |
| `0x33` | `carcyclopedia_refresh_car` | the colour name (`car_database_get_color_name_id`) |
| `0x34` | `carcyclopedia_refresh_car` | the description string id, the detail record's `+0xc4` (`structs/car_database.md` §1b) |
| `0x66` | `carcyclopedia_refresh_car` | the repaint control, **enabled only when the car has more than one colour** |

Design-table entry: **id `0xd`** of `g_gui_screen_design_table`, used by both the ctor and the
rebuild.

## 7. ⚠️ Two things in this unit that are NOT this class

A function inside a class's VA span is not automatically that class's. Both were settled by
counting the function's address as a vtable slot across `.rdata`.

| VA | What it is | Why it is not `carcyclopedia_` |
|---|---|---|
| `0x005634a0` `gui_menu_on_exchange_forward_to_parent` | a `GM_Menu` framework default | **45 vtable datarefs**: it occupies slot 1 of nearly every screen in the game, so it belongs to the framework (`structs/gui_screen.md`) |
| `0x00562fa0` / `0x005634b0` / `0x005634d0` / `0x005639e0` `gui_mp_person_select_ctor` / `_on_close` / `_tick` / `_show_model` | the **multiplayer driver-appearance picker** ³ | a separate class: vtable `0x0063d398`, design id `0x2c`, a `0x68`-byte object (`structs/multiplayer.md` §6) |

³ Opened by the multiplayer lobby (`gui_mp_lobby_tick`, `0x00564320`) from its **`Person:`** button,
component `0x2afa`, textdb 662 (cz *Postava:*); the index it returns lands in `g_net_session+0x228`. It lists all 66
`tables\playerindex.def` **character** models (`civilNN.i3d`), the table at `G_CarDatabase+0x0c`
read by `car_database_get_player_model` (`structs/car_database.md` §1c). It previews the selection
walking on the showroom set, which is why it binds `walk1.i3d`.

## 8. PC vs PS2

Everything cross-build lives here.

### 8a. What identified the class

Four independent lines, none of them the `autosalon` scene name:

1. **The destructor is a lockstep.** `carcyclopedia_dtor` (80 B) and `__dt__16GM_CarcyclopediaFv`
   (184 B) free the SECOND 3-dword vector first (PC `+0x8c/+0x90/+0x94` ↔ PS2 `+0x68/+0x6c/+0x70`),
   then the first (PC `+0x7c/+0x80/+0x84` ↔ PS2 `+0x5c/+0x60/+0x64`), zeroing each triple, then
   tail-call the base destructor.
2. **`RefreshCar` is byte-level.** The empty-list fallback to car id 1, the max-colour clamp that
   zeroes the colour index, `GetIDByParams(id, 0, colour)` ↔ `car_database_pack_id(id, 0, colour)`
   with the same middle `0`, and the same four element ids `0x66 / 0x32 / 0x33 / 0x34` in the same
   order, with the description read from detail offset `0xc4` (the PS2 renders it as `200`).
3. **The string block matches.** PS2 `g_m_carc.cpp` carries `models\%s`, `CarcCar`, `NGon02`,
   `autosalon`, `autosalon2`, `carcyclopedia` as one contiguous run; the PC has the same six literals
   in the same unit.
4. **The shipped label.** textdb 110 / 4083 / 4087 = `"Carcyclopedia"`.

### 8b. 🔑 The base class falls out of the dtor

`__dt__16GM_CarcyclopediaFv` tail-calls `__dt__7GM_MenuFv`, and the PC dtor tail-calls
`gui_menu_dtor` (`0x0056af10`) in the same position, so **`gui_menu_dtor` is `GM_Menu::~GM_Menu`**,
and `gui_menu_ctor` (`0x0056ae50`), called first by the PC constructor, is the **`GM_Menu`
constructor**. The framework's own translation unit, `g_menu.cpp` at `[0x0056adc0, 0x00575c20)`, is
therefore **`GM_Menu` / `GM_Component`**; the neighbouring `[0x00566280, 0x0056adc0)` is the racing
screens' unit (`structs/racing_menus.md`). Survey of the screen framework: `structs/gui_screen.md`.

### 8c. 🔴 `G_CarcCache` is PS2-only

Nine of the 28 PS2 symbols in `g_m_carc.cpp` are `G_CarcCache`: an `I3D_model` cache keyed by model
name and by car id + colour, with a size limit. **The PC has no counterpart.** Where the PS2 calls
`GetModel__11G_CarcCache` the PC calls `i3d_cache_model_open` directly, and where the PS2 calls
`Clear__11G_CarcCache` in `OnClose` the PC chains straight to the framework default. The PS2's
`RefreshCar` also takes two extra parameters (`int, int, bool`) that exist only to drive
`CacheModelByIdx` / `CacheModelByColor`; the PC's takes one. This is a memory constraint, not a
feature difference.

### 8d. The 4 → 1 fold in slot 5

The PS2 splits the screen's per-frame work across four virtuals; the PC has one slot for it. Sizes:
`OnTick` 1296 + `OnKey` 768 + `OnMouse` 428 + `OnClick` 120 = **2612 B** against **2704 B** for
`carcyclopedia_tick`, a ratio of **1.04**. Both builds wrap the yaw into `[0, 360)` and convert with
the same `0.017453292`, and both funnel every selection or colour change through `RefreshCar`.
`carcyclopedia_refresh_display` is PC-only factoring of statements the PS2 runs inline.

### 8e. Where the unlocked-car mask comes from

The PS2 takes it from `FreerideGetEnabledCars__15G_LoadSaveClassFv` (`{ return *(this + 0x2b8); }`).
The PC calls `loadsave_freeride_get_enabled_cars` (`0x0055ffb0`, `{ return *(this + 0x9c); }`) on
the **same singleton**, `0x00671fb0`, the `G_LoadSaveClass` object, at the **same point**, ORs in
`0x2000000` and passes the result to the same car-database collector. That edge holds on both
builds, which makes `loadsave_freeride_get_enabled_cars` the PC twin of `FreerideGetEnabledCars`
(`structs/loadsave.md` §10e).

### 8f. The other symbols of `g_m_carc.cpp`

The seven 8–16-byte `GM_Menu` stubs the PS2 emits into this file by COMDAT (`OnBeforeExchange`,
`OnExchange`, `OnCreate`, `OnCreateChild`, `OnCloseChild`, `OnChange`, `MartinovaKlavesa`),
`IsOverActive__12GM_ComponentFff` and two `gif::vector<…,8>::reserve` instantiations are framework
and container code, not class methods; the PC emits them elsewhere. None of them is claimed inlined.

### 8g. The framework flag word

PC `+0x40` is PS2 `+0x24`: both constructors clear bit `0x00010000` in it.

### 8h. The two mode fields on the PS2

The PS2 `OnCreate` stores the same 60.0 at its `+0x4c`, and none of the other eight PS2 methods
reads it (a text search of their bodies). The PS2 keeps the list / show switch as a byte at
`+0x58` that feeds `SetHidden(1)` / `SetHidden(2)`.

### 8i. PS2-only methods

| PS2 VA | PS2 method | Why there is no PC function |
|---|---|---|
| 0x002bfcd0 | `Clear__11G_CarcCacheFv` | `G_CarcCache` is PS2-only (§8c) |
| 0x002bfd70 | `RemoveFirst__11G_CarcCacheFi` | the same |
| 0x002bfe20 | `LimitSize__11G_CarcCacheFv` | the same |
| 0x002bfe70 | `HaveModel__11G_CarcCacheFPc` | the same |
| 0x002bff20 | `GetModel__11G_CarcCacheFPc` | the same; the PC calls `i3d_cache_model_open` at its call position |
| 0x002c01d0 | `GetModelById__11G_CarcCacheFUiUi` | the same |
| 0x002c0250 | `CacheModelByIdx__11G_CarcCacheFUiib` | the same |
| 0x002c0460 | `CacheModelByColor__11G_CarcCacheFUii` | the same |
| 0x002c1b40 | `OnBeforeCreate__16GM_CarcyclopediaFv` | 12 B, returns the scene name `"carcyclopedia"`; the PC loads the `autosalon` scene from `engine_main` instead (§4) |
| 0x002c1bc0 | `__dt__11G_CarcCacheFv` | `G_CarcCache` is PS2-only (§8c) |

## 9. Related

- `structs/gui_screen.md` — the `GM_Menu` base and the screen framework.
- `structs/car_database.md` — the car catalogue the screen reads.
- `structs/multiplayer.md` — the person picker that shares the unit.
- `structs/loadsave.md` — the unlocked-car mask.
- `structs/car.md` — `car_set_roof_visible`, the roof toggle.

## 10. Open items

1. **The repaint control in game:** does element `0x66` grey out for a single-colour car? Not tested.
2. **The car list in game:** does it contain exactly the cars unlocked for Free Ride? Not tested.
