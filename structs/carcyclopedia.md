# `GM_Carcyclopedia` — the main-menu Carcyclopedia screen

**Created** 2026-08-23 20:50 +0200 · **Modified** 2026-10-01 08:56 +0200

PC prefix **`carcyclopedia_`**.

| | |
|---|---|
| PC translation unit | `[0x00561b70, 0x00563b30)` — 9 function starts, all named |
| PC vtable | `.rdata 0x0063d378` (6 slots) |
| PC object size | `0x9c` |
| Base class | **`GM_Menu`** (`structs/gui_screen.md`) |
| PS2 file | `g_m_carc.cpp` (`0x002bfcd0`–`0x002c1c40`, 28 symbols) |

## 1. What it is

The **Carcyclopedia** — the in-game car encyclopedia, reached from the main menu. In-game
label: textdb **110 / 4083 / 4087** = `"Carcyclopedia"` (de *Autozyklopädie*, cz *Auto
encyklopedie*). It shows one car at a time on a rotating pedestal with its name, colour
name and description, and lets you page through every car your profile has unlocked and
cycle its paint colours.

`engine_main` reaches it from main-menu state **`0x19`**, the entry immediately after the
credits (`0x17` → `loadsave_execute_credits`):

```
mission_close()
mission_open("autosalon", 0, 0xffffffff, 1)
collision_load_level("missions\autosalon\tree.klz", 1)
game_init()
operator_new(0x9c) -> carcyclopedia_ctor(obj, 0, 0)
gui_menu_execute_modal(1)
mission_close()
```

🔑 **`autosalon` is the SCENE, not the class.** The screen runs inside a car-showroom set
whose scene directory is `missions\autosalon\`; the frames `autosalon` and `autosalon2`
are the two camera placements the screen blends between, and `NGon02` is the turntable. The
class is `GM_Carcyclopedia` on both builds.

## 2. Field map of the 0x9c-byte screen object

Conf: **H** measured on ≥2 independent sites · **M** one site, unambiguous · **U** unproven.

| Offset | Field | Conf | Evidence |
|---|---|---|---|
| `+0x00` | vptr → `0x0063d378` | H | installed by `carcyclopedia_ctor`; 6 slots (§3) |
| `+0x18` | **the PARENT screen** (`GM_Menu` base field) | H | the framework's slot-4 default `gui_menu_close` assigns `this->+0x18` to `g_gui_active_screen` on close and then notifies it through slot 3; every screen constructor in this family is called as `ctor(g_gui_active_screen, …)`; `carcyclopedia_on_exchange` forwards the rebuild UP to it ¹ |
| `+0x40` | framework flag word | H | ctor clears bit `0x00010000` (`&= 0xfffeffff`) (§6g) |
| `+0x48` | **selected car index** | H | written by `carcyclopedia_tick`, read by `carcyclopedia_refresh_display`, passed to `carcyclopedia_refresh_car` |
| `+0x4c` | **paint-colour index** | H | clamped against `car_database_get_max_color_index` in `carcyclopedia_refresh_car`; cycled by the left/right arrows |
| `+0x50` | showroom car model (I3D object) | H | created by the ctor through the I3D driver factory (type `9`); reloaded per car; released by `carcyclopedia_on_close` |
| `+0x58` | the `NGon02` frame — **the turntable** | H | ctor, frame-by-name lookup with mask `0xffff`; `carcyclopedia_tick` sets its direction from the yaw `+0x64` every frame |
| `+0x5c` | the `autosalon` frame — camera placement at blend 1 | H | ctor; `carcyclopedia_tick` reads its world position (`+0x6c`) |
| `+0x60` | the `autosalon2` frame — camera placement at blend 0 | H | ctor, which re-normalises its basis right after the lookup; `carcyclopedia_tick` reads its world position and axis (`+0x6c`) |
| `+0x64` | **turntable yaw, degrees** | H | `carcyclopedia_tick` wraps it into `[0, 360)` and feeds `sin`/`cos` of `yaw * 0.017453292` |
| `+0x68` | `60.0f`, **write-only** | H | the ctor stores `0x42700000`; a sweep of the class's own code finds no other access, and the `GM_Menu` base fields end at `+0x44` |
| `+0x6c` | **camera blend**, 0..1 | H | ctor `1.0f`; `carcyclopedia_tick` clamps it, moves it on the up / down arrow keys and a vertical drag (0.005 per pixel), and places the active camera (`I3D_scene+0x17c`) between the `+0x60` (0) and `+0x5c` (1) frame positions |
| `+0x70` | **turntable spin speed**, degrees per second | H | ctor `16.666f`; `carcyclopedia_tick` adds `speed · dt` to the yaw `+0x64`, relaxes the speed back toward 16.67, sets it from a horizontal drag, and sets +100 / −100 while the right / left arrow key is held |
| `+0x74` | framework creation flag | M | set to 1 by the ctor and re-passed to the element builder by `carcyclopedia_on_exchange` |
| `+0x78` | `std::vector<car_id>` object | H | MSVC `{allocator, first, last, end}`; the allocator byte is written by the ctor |
| `+0x7c`/`+0x80`/`+0x84` | that vector's begin / end / cap | H | filled by `carcyclopedia_build_car_list`; freed second by `carcyclopedia_dtor` |
| `+0x88` | `std::vector<char*>` object (car names) | H | same shape |
| `+0x8c`/`+0x90`/`+0x94` | that vector's begin / end / cap | H | filled by `carcyclopedia_build_car_list`; freed FIRST by `carcyclopedia_dtor` |
| `+0x98` | **roof toggle** (byte) | H | ctor 0; widget command `0x79` flips it and calls `car_set_roof_visible` with it, and `carcyclopedia_refresh_car` re-applies it to every car it shows |

¹ The field holds the parent, not a child: nothing in this class releases a child screen
through it. `structs/multiplayer.md` §7.

The two vectors are **index-parallel**, which is why every consumer takes a single
selection index.

## 3. The vtable at `0x0063d378` — and the framework's slot layout

The PC screen vtable is **six slots**. Three sibling tables in the neighbouring cluster
(`0x0063d398`, `0x0063d3b4`, `0x0063d3cc`) have the same shape, which is what makes the
layout a measurement rather than a guess:

| Slot | This class | Meaning |
|---|---|---|
| 0 | `carcyclopedia_scalar_deleting_dtor` | MSVC `delete` entry |
| 1 | `carcyclopedia_on_exchange` | re-create in place (language / resolution change) |
| 2 | `carcyclopedia_on_close` | teardown |
| 3 | `gui_menu_on_close_child` *(framework default, 32 vtables)* | not overridden here |
| 4 | `gui_menu_close` *(framework default, 49 vtables)* | not overridden here |
| 5 | `carcyclopedia_tick` | per-frame update + keyboard + widget commands |

Slot 1's framework default is `gui_menu_on_exchange_forward_to_parent`, which occupies that
slot in **45 vtables** — so a function sitting in this VA range is not automatically this
class's (§5). Its PS2 twin is `OnExchange__7GM_MenuFv` [STRUCTURAL].

## 4. Coverage — PC ↔ PS2

**9 of 9 PC starts named.** Ordered by PC VA, PC name first.

| PC VA | PC name | PC size | PS2 method | PS2 size | Grade |
|---|---|---|---|---|---|
| 0x00561b70 | `carcyclopedia_ctor` | 1120 | `OnCreate__16GM_CarcyclopediaFv` | 1348 | CONFIRMED |
| 0x00561fd0 | `carcyclopedia_scalar_deleting_dtor` | 32 | - | - | STRUCTURAL ¹ |
| 0x00561ff0 | `carcyclopedia_dtor` | 80 | `__dt__16GM_CarcyclopediaFv` | 184 | CONFIRMED |
| 0x00562040 | `carcyclopedia_on_close` | 64 | `OnClose__16GM_CarcyclopediaFv` | 100 | CONFIRMED |
| 0x00562080 | `carcyclopedia_on_exchange` | 48 | `OnExchange__7GM_MenuFv` | 16 | STRUCTURAL ² |
| 0x005620b0 | `carcyclopedia_refresh_display` | 112 | - | - | CONFIRMED ³ |
| 0x00562120 | `carcyclopedia_tick` | 2704 | `OnTick` + `OnKey` + `OnMouse` + `OnClick` | 2612 | CONFIRMED ⁴ |
| 0x00562bb0 | `carcyclopedia_build_car_list` | 496 | `OnCreate__16GM_CarcyclopediaFv` | - | CONFIRMED ⁵ |
| 0x00562da0 | `carcyclopedia_refresh_car` | 512 | `RefreshCar__16GM_CarcyclopediaFiib` | 864 | CONFIRMED |

¹ MSVC-only. ² The framework default, not a class method. ³ PC-only factoring; the PS2
inlines it. ⁴ A 4 → 1 fold (§6d). ⁵ Twins the list-building half of `OnCreate`.

### 4a. The widget ids this screen drives

Byte-identical on both builds, which is most of why `carcyclopedia_refresh_car` graded
CONFIRMED:

| Element | Set by | Content |
|---|---|---|
| `0x14` | `carcyclopedia_refresh_display`, `carcyclopedia_tick` | the scrolling car list; its backing list is the name vector at `+0x88` |
| `0x32` | `carcyclopedia_refresh_car` | the car name (`car_database_get_car_name`) |
| `0x33` | `carcyclopedia_refresh_car` | the colour name (`car_database_get_color_name_id`) |
| `0x34` | `carcyclopedia_refresh_car` | the description string id, read from the detail record's `+0xc4` |
| `0x66` | `carcyclopedia_refresh_car` | the repaint control — **enabled only when the car has more than one colour** |

Design-table entry: **id `0xd`** of `g_gui_screen_design_table`, used by both the ctor and
the rebuild.

## 5. ⚠️ Two things in this VA range that are NOT this class

The mirror-image trap: a function inside a class's VA span is not automatically that
class's. Both were settled by counting the function's address as a vtable slot across
`.rdata`.

| VA | What it is | Why it is not `carcyclopedia_` |
|---|---|---|
| `0x005634a0` `gui_menu_on_exchange_forward_to_parent` | a `GM_Menu` framework default | **45 vtable datarefs.** It occupies slot 1 of nearly every screen in the game, so it belongs to the framework, not here |
| `0x00562fa0` / `0x005634b0` / `0x005634d0` / `0x005639e0` `gui_mp_person_select_ctor` / `_on_close` / `_tick` / `_show_model` | the **multiplayer driver-appearance picker** ¹ | a separate class: vtable `0x0063d398`, design id `0x2c`, a `0x68`-byte object |

¹ Opened by the multiplayer LOBBY (`gui_mp_lobby_tick`, `0x00564320`) from its **`Person:`**
button, component `0x2afa`, textdb 662 (cz *Postava:*); the index it returns lands in
`g_net_session+0x228`. It lists all 66 `tables\playerindex.def` **character** models
(`civilNN.i3d`) — the table at `G_CarDatabase+0x0c`, read by
`car_database_get_player_model` (`GetPlayerModel__13G_CarDatabaseFUi`), whose fallback
string at `0x0064d82c` is `"-"`. It previews the selection walking on the `carcyclopedia`
showroom set, which is why it binds `walk1.i3d`. Full write-up: `structs/multiplayer.md`
§3; the character table itself: `structs/car_database.md` §1.

## 6. PC vs PS2

Everything cross-build lives here.

### 6a. What identified the class

Four independent lines, none of them the `autosalon` scene name:

1. **The destructor is a lockstep.** `carcyclopedia_dtor` (80 B) and
   `__dt__16GM_CarcyclopediaFv` (184 B) free the SECOND 3-dword vector first (PC
   `+0x8c/+0x90/+0x94` ↔ PS2 `+0x68/+0x6c/+0x70`), then the first (PC `+0x7c/+0x80/+0x84`
   ↔ PS2 `+0x5c/+0x60/+0x64`), zeroing each triple, then tail-call the base destructor.
2. **`RefreshCar` is byte-level.** The empty-list fallback to car id 1, the max-colour
   clamp that zeroes the colour index, `GetIDByParams(id, 0, colour)` ↔
   `car_database_pack_id(id, 0, colour)` with the same middle `0`, and the same four
   element ids `0x66 / 0x32 / 0x33 / 0x34` in the same order, with the description read
   from detail offset `0xc4` (the PS2 renders it as `200`).
3. **The string block matches.** PS2 `g_m_carc.cpp` carries `models\%s`, `CarcCar`,
   `NGon02`, `autosalon`, `autosalon2`, `carcyclopedia` as one contiguous run; the PC has
   the same six literals in the same TU.
4. **The shipped label.** textdb 110 / 4083 / 4087 = `"Carcyclopedia"`.

### 6b. 🔑 The base class falls out of the dtor for free

`__dt__16GM_CarcyclopediaFv` tail-calls `__dt__7GM_MenuFv`, and the PC dtor tail-calls
`gui_menu_dtor` (`0x0056af10`) in the same position — so **`gui_menu_dtor` is
`GM_Menu::~GM_Menu`**, and `gui_menu_ctor` (`0x0056ae50`), called first by the PC
constructor, is the **`GM_Menu` constructor**. The framework's own translation unit,
`g_menu.cpp` at `[0x0056adf0, 0x00575c20)`, is therefore **`GM_Menu` / `GM_Component`**;
the neighbouring `[0x00566280, 0x0056adf0)` is the racing screens' translation unit
(`structs/racing_menus.md`). Survey of the screen framework: `structs/gui_screen.md`.

### 6c. 🔴 `G_CarcCache` is PS2-only

Nine of the 28 PS2 symbols in `g_m_carc.cpp` are `G_CarcCache` — `Clear`, `RemoveFirst`,
`LimitSize`, `HaveModel`, `GetModel`, `GetModelById`, `CacheModelByIdx`,
`CacheModelByColor` and its destructor — an `I3D_model` cache keyed by model name and by
car id + colour, with a size limit. **The PC has no counterpart.** Where the PS2 calls
`GetModel__11G_CarcCache` the PC calls `i3d_cache_model_open` directly, and where the PS2
calls `Clear__11G_CarcCache` in `OnClose` the PC chains straight to the framework default.
The PS2's `RefreshCar` also takes two extra parameters (`int, int, bool`) that exist only
to drive `CacheModelByIdx` / `CacheModelByColor`; the PC's takes one. This is a memory
constraint, not a feature difference — and it is why 9 of 28 PS2 symbols have no PC start.

### 6d. The 4→1 fold in slot 5

The PS2 splits the screen's per-frame work across four virtuals; the PC has one slot for
it. Sizes: `OnTick` 1296 + `OnKey` 768 + `OnMouse` 428 + `OnClick` 120 = **2612 B**
against **2704 B** for `carcyclopedia_tick`, a ratio of **1.04**. Both builds wrap the yaw
into `[0, 360)` and convert with the same `0.017453292`, and both funnel every selection or
colour change through `RefreshCar`.

### 6e. Where the unlocked-car mask comes from

The PS2 takes it from `FreerideGetEnabledCars__15G_LoadSaveClassFv`
(`{ return *(this + 0x2b8); }`). The PC calls `loadsave_freeride_get_enabled_cars`
(`0x0055ffb0`, `{ return *(this + 0x9c); }`) on the **same singleton** — `0x00671fb0`, the
`G_LoadSaveClass` object — at the **same point**, ORs in `0x2000000` and passes the result
to the same car-database collector. That is an edge holding on both builds, which makes
`loadsave_freeride_get_enabled_cars` the PC twin of `FreerideGetEnabledCars`
(`structs/loadsave.md` §9e).

### 6f. PS2 methods with no PC start — *not located*, no terminal tag

Beyond the nine `G_CarcCache` methods (§6c): `OnBeforeCreate__16GM_Carcyclopedia`
(12 B — returns the scene name `"carcyclopedia"`, which the PC handles by loading the
`autosalon` scene from `engine_main` instead), and the seven 8–16-byte `GM_Menu` stubs the
PS2 emits into this file by COMDAT (`OnBeforeExchange`, `OnExchange`, `OnCreate`,
`OnCreateChild`, `OnCloseChild`, `OnChange`, `MartinovaKlavesa`). None of these is claimed
inlined; they are simply framework code that the PC emits elsewhere.

### 6g. The framework flag word

PC `+0x40` is PS2 `+0x24`: both constructors clear bit `0x00010000` in it.

## 7. Open items

| item | state |
|---|---|
| does the repaint control grey out for single-colour cars (element `0x66`)? | not tested in game |
| does the car list contain exactly the cars unlocked for Free Ride? | not tested in game |
