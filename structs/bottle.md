# `C_bottle` — the small thrown-physics-prop base class

**Created** 2026-08-05 12:38 +0200 · **Modified** 2026-10-12 19:38 +0200

A `C_actor` subclass: a small prop that flies a ballistic arc, tumbles, bounces off people,
reacts to being shot, and is replicated to other machines. Its own twenty functions are the
clearest worked example of the engine's actor contract — lifecycle, savegame, collision, damage
and network replication.

**It cannot be picked up, and a `C_bottle` is never thrown** (§6b): the class has no
`S_using_object`, no use handler and no `GetKind == 0xa` test anywhere in the image, and all four
callers of `bottle_set_flight_impulse` spawn a **subclass** (`C_grenade` — the molotov —
`C_cartridge`, `C_smoke_trail`). The throw API belongs to the family, not to the leaf. The two
instances the shipped game places are **tin cans** (§6a).

| | |
|---|---|
| PC prefix | `bottle_` |
| Base class | `C_actor` (`structs/actor.md`) ¹ |
| kind (`+0x10`) | **`0xa`** ² |
| PC vtable | `.rdata 0x0063b870` |
| PC translation unit | `[0x00453a50, 0x00456460)` — **20 function starts**, bounded at *both* ends by a CRT static-init/atexit pair (`0x00453a20` closes `C_airplane` and opens this one, `0x00456460` closes it) |
| Subclasses | `C_grenade`, `C_cartridge`, `C_smoke_trail` (§5) |

¹ `bottle_ctor` → `actor_ctor`, `bottle_dtor` → `actor_dtor`, and `bottle_init` /
`bottle_game_init` / `bottle_game_done` / the savegame trio all chain to their base first.
² Byte-proven from the ctor and corroborated twice: `structs/entity.md` documents kind `0xa`, and
`recorder_new_track`'s kind→prefix switch falls through to `unk_` for it, so the recorder does not
track bottles.

---

## 1. Field layout

Confidence tags: **[H]** byte-proven from a writer *and* a reader · **[M]** one side traced ·
**[U]** serialised but no consumer traced.

The savegame pair is the backbone of this table: `bottle_save_game_save` writes 22 fields in
a fixed order and `bottle_save_game_load` reads the same 22 back, so every offset below that
is marked *saved* has two independent witnesses.

```c
typedef struct C_bottle {                /* extends C_actor */
    /* +0x00  */ void      *vptr;            // 0x0063b870
    /* ...    */ C_actor    base;
    /* +0x10  */ int        kind;            // [H] = 0xa
    /* +0x18  */ int        guid;            // [H] object id; high byte selects the owner
    /* +0x1b  */ char       owner_slot;      // [H] peer slot; arg to net_session_get_peer_clock_ms
    /* +0x20  */ char       is_remote;       // [H] C_actor base: this machine is NOT authoritative
    /* +0x24  */ S_vector   pos;             // [H] C_actor base. saved
    /* +0x30  */ S_vector   dir_frame;       // [H] C_actor base, fed to S_matrix::SetDir3. saved
    /* +0x68  */ I3D_frame *frame;           // [H] C_actor base
    /* +0x6d  */ char       may_broadcast;   // [H] ctor = 1; half the host-role guard
    /* +0x70  */ S_vector   flight_dir;      // [H] normalised. saved
    /* +0x7c  */ float      flight_speed;    // [H] |impulse|. saved
    /* +0x80  */ float      fall_accum;      // [H] flight ends when this passes 50.0. saved
    /* +0x84  */ char       landed_sets_state;// [H] chooses act-state 1 vs 2 at flight end. saved
    /* +0x88  */ float      restitution;     // [H] speed kept per bounce. GameInit = 0.4f;
                                             //     cartridge_set_type halves it for type B
    /* +0x8c  */ S_vector   contact_normal;  // [H] ground normal of the last bounce. saved
    /* +0x98  */ S_vector   tumble_rate;     // [H] 3x rand_float; re-rolled on every hit
    /* +0xa4  */ float      radius;          // [H] half the bound-volume diagonal. saved
    /* +0xa8  */ S_vector   frame_axis;      // [H] second axis for SetDir3, tumbled each tick. saved
    /* +0xb4  */ tDynamicCollObject coll;    // [H] 0x48 bytes, zeroed by the ctor
    /* +0xc4  */ S_vector   coll_pos;        // [H] the proxy's position mirror
    /* +0xfc  */ char       coll_registered; // [H] ctor = 1; gates insert/update/erase
    /* +0x100 */ int        bounce_count;    // [H] +1 per ground contact. saved
    /* +0x104 */ char       on_ground;       // [H] the ground ray hit this tick. saved
    /* +0x108 */ S_vector   net_pos;         // [H] last received authoritative position
    /* +0x114 */ S_vector   net_vel;         // [H] its velocity
    /* +0x120 */ float      net_gravity;     // [H] the fall term applied while extrapolating
    /* +0x124 */ int        send_pending;    // [M] cleared after a broadcast
    /* +0x128 */ char       broadcast_on;    // [H] ctor = 1; gates the state broadcast
    /* +0x12c */ int        ground_material; // [H] surface material of the ground hit. saved
    /* +0x130 */ void      *ground_dyn_obj;  // [H] dynamic object the ground ray hit, else 0;
                                             //     C_grenade parents its fire to its frame
    /* +0x134 */ char       ray_all_layers;  // [H] ground-ray mask: 0 = mask 2, 1 = every layer
} C_bottle;
```

| Offset | Field | Conf | Writer(s) | Reader(s) | Notes |
|---|---|---|---|---|---|
| +0x10 | kind | H | `bottle_ctor` (= 0xa) | `folded_get_dword_0x10` (its `C_actor::GetKind` role), `bottle_on_collision` | the engine's stand-in for RTTI |
| +0x20 | is_remote | H | C_actor base | `bottle_on_collision`, `bottle_hit` | when set, both handlers stop simulating and report upstream instead |
| +0x70 / +0x7c / +0x80 | the flight triple | H | `bottle_set_flight_impulse`, `bottle_apply_net_event_msg` | `bottle_ai_tick`, `bottle_on_collision`, `bottle_hit` | direction (normalised), speed, fall accumulator. **The flight ends when `+0x80` passes 50.0** |
| +0x88 | restitution | H | `bottle_game_init` (0.4), `cartridge_set_type` (0.2 for type B) | `bottle_ai_tick` | on a bounce the new speed `+0x7c` is the old speed × `+0x88` |
| +0x8c | contact_normal | H | `bottle_ai_tick` (each bounce), `bottle_game_init` (clear) | `bottle_on_collision`, `bottle_hit`, the savegame pair | sent in the type-`0x15` collision message and added ×0.01 as a push-off |
| +0x98 | tumble_rate | H | `bottle_game_init`, `bottle_on_collision`, `bottle_hit` (3× `rand_float` each) | the render transform | re-rolled on *every* impact, which is why a bottle never tumbles the same way twice |
| +0xa4 | radius | H | `bottle_game_init` | collision | half the diagonal of the frame's hierarchical bound volume, after world scale |
| +0xa8 | frame_axis | H | `bottle_ai_tick` (rotated by the tumble each tick), `cartridge_game_init` (= (0, 1, 0)) | `bottle_update`, the savegame pair | passed to `S_matrix::SetDir3` together with `+0x30` |
| +0x100 / +0x104 / +0x12c / +0x130 | the ground-contact band | H | `bottle_ai_tick` (each ground ray); `grenade_on_collision` / `grenade_hit` also bump `+0x100` | `cartridge_ai_tick` (`+0x100` > 15 retires a casing; `+0x104` and `+0x12c` pick the landing sound), `grenade_ai_tick` (`+0x130`), the savegame pair | bounce count, ray hit this tick, the hit's surface material, the dynamic object hit (else 0). When a grenade's fire starts, `grenade_ai_tick` takes that object's frame (`+0x68`), converts the fire position into the frame's local space and passes the frame to `game_new_fire`, so the fire moves with whatever the projectile landed on. `+0x104` clears when the flight ends at speed < 3.0 on a surface within 0.5 rad of level |
| +0x134 | ray_all_layers | H | `cartridge_game_init` (= 1), `bottle_game_init` (clear) | `bottle_ai_tick` | selects the ground-ray mask: 2 when clear, every layer when set |
| +0xb4 / +0xfc | dyn-coll proxy + its flag | H | `bottle_ctor` (flag = 1), `bottle_update` | `bottle_game_done` (`collision_dyn_erase`) | the proxy position is mirrored from `+0x24` every frame |
| +0x108 / +0x114 / +0x120 | the dead-reckoning band | H | `bottle_apply_net_event_msg` (type 0x16) **only** | `bottle_net_apply_remote_state` **only** | one writer, one reader — which is what identifies both |
| +0x128 | broadcast_on | H | `bottle_ctor` (= 1), `bottle_ai_tick` (clear at flight end) | `bottle_post_state_net_event_msg` | a landed bottle stops being broadcast |

---

## 2. Method coverage — 20/20 named, 17/17 PS2 methods accounted

```
20 PC = 14 C_bottle methods + 1 MSVC scalar deleting dtor
      + 3 PC-only network functions + 2 foreign helpers
17 PS2 = 13 body-diffed + NetDirect (an 8-byte stub) + 3 folded 8-byte stubs
```

PC-first, ordered by PC VA.

| PC VA | PC name | Slot | PS2 twin (mangled) | PS2 size | Status | Role |
|---|---|---|---|---|---|---|
| 0x00453a50 | `chunk_read` | - | `Read__7C_chunkFPvi` | 216 | CONFIRMED | **foreign** — 212 callsites over 20+ functions; leads the TU only because MSVC emitted the out-of-line inline here |
| 0x00453ad0 | `bottle_ctor` | - | `__ct__8C_bottleFv` | 112 | CONFIRMED | base ctor → zero the proxy → vtable → kind 0xa → three flags |
| 0x00453b10 | `bottle_scalar_deleting_dtor` | 16 | - | - | STRUCTURAL | MSVC thunk; GCC folds the free flag into `__dt__` |
| 0x00453b30 | `bottle_dtor` | - | `__dt__8C_bottleFv` | 96 | CONFIRMED | restore vptr → `actor_dtor`; owns no heap memory |
| 0x00453b40 | `bottle_init` | 18 | `Init__8C_bottleFP9I3D_frame` | 84 | CONFIRMED | null-guard → `actor_init` → `actor_debustize_frame` |
| 0x00453b70 | `bottle_ai_tick` | 13 | `AI__8C_bottleFUi` | 3252 | CONFIRMED | the ballistic arc; ratio 1.01 |
| 0x00454840 | `bottle_update` | 14 | `Update__8C_bottleFUi` | 272 | CONFIRMED | transform → I3D frame; **both** shared frame masks |
| 0x00454920 | `bottle_net_apply_remote_state` | 15 | `NetDirect__8C_bottleFUi` | 8 | STRUCTURAL | dead reckoning + smoothing; the PS2 twin is an empty stub (§3a) |
| 0x00455160 | `bottle_game_init` | 25 | `GameInit__8C_bottleFv` | 768 | CONFIRMED | clears the flight band, rolls the tumble rate, measures the radius; ratio 1.04 |
| 0x00455480 | `vec3_div_scalar` | - | - | - | STRUCTURAL | **foreign** — 6 callsites / 5 functions, all stack receivers |
| 0x004554b0 | `bottle_game_done` | 26 | `GameDone__8C_bottleFv` | 64 | CONFIRMED | guarded `collision_dyn_erase` → base |
| 0x004554e0 | `bottle_save_game_get_size` | 22 | `SaveGameGetSize__8C_bottleFv` | 32 | CONFIRMED | base + 0x53 |
| 0x004554f0 | `bottle_save_game_save` | 23 | `SaveGameSave__8C_bottleFPPv` | 264 | CONFIRMED | version byte 1 + **22 stores in identical order** |
| 0x004555d0 | `bottle_save_game_load` | 24 | `SaveGameLoad__8C_bottleFPPv` | 436 | CONFIRMED | the version-gated mirror |
| 0x00455710 | `bottle_set_flight_impulse` | - | `AbsoluteImpulse__8C_bottleFR8S_vector` | 452 | CONFIRMED | launches the bottle; ratio 0.99. Non-virtual |
| 0x004558d0 | `bottle_on_collision` | 30 | `Collision__8C_bottleF…` | 708 | CONFIRMED | identical kind gate `{2, 10, 0x1b}`; ratio 1.06 |
| 0x00455bc0 | `bottle_hit` | 31 | `Hit__8C_bottleF10E_hit_type…` | 756 | CONFIRMED | identical hit-type gate `{0,1,2}` + `abs(y) × 15.0`; ratio 0.99 |
| 0x00455eb0 | `bottle_apply_net_event_msg` | 38 | - | - | STRUCTURAL | **PC-only.** Types 0x13 / 0x15 / 0x16 |
| 0x00456200 | `bottle_post_state_net_event_msg` | - | - | - | STRUCTURAL | **PC-only.** Type 0x16, descriptor `.rdata 0x0063b92c` |
| 0x00456310 | `bottle_post_collision_net_event_msg` | - | - | - | STRUCTURAL | **PC-only.** Type 0x15, descriptor `.rdata 0x0063b93c` |

**Inherited, not overridden** (the vtable points outside the TU): slot 17 `Release__7C_actorFv`,
slot 27 `ChangeState`, slot 28 `actor_get_actor_speed`, slot 32 `GetMaterialID`,
slot 34 `actor_get_bound_points_2d`.

### 2a. How the network trio closes on itself

The three functions verify each other, with no PS2 side needed:

- `bottle_post_state_net_event_msg` emits type **0x16**; `bottle_apply_net_event_msg`
  handles 0x16 and writes the `+0x108..+0x120` band.
- `bottle_net_apply_remote_state` is the band's **only** reader.
- `bottle_post_collision_net_event_msg` emits type **0x15**; the receiver handles 0x15 and
  ends with `actor_set_act_state(0)` — the same tail as `bottle_set_flight_impulse`, i.e. a
  peer's throw relaunches the bottle locally.
- `bottle_ai_tick` **and** `bottle_apply_net_event_msg` both re-broadcast through
  `bottle_post_state_net_event_msg` under the **byte-identical** guard
  `g_net_session+0x100 == 7 && this+0x6d != 0`. Two independent code paths agreeing on the
  host-role gate is what pins the whole loop.

The position quantise/dequantise pair the senders and the receiver use is
`net_session_encode_position` (`0x0059a8c0`) / `net_session_decode_position` (`0x0059a810`), members
of `g_net_session` (`structs/net_session.md`).

The receiver's third type, **`0x13`**, is the generic act-state change that `actor_set_act_state`
posts for every actor in the host role (`g_event_msg_vt_actor_act_state`, `structs/box.md` §4).

### 2b. The two bottle event messages

| type | descriptor | serialize / deserialize | payload |
|---|---|---|---|
| `0x16` | `g_event_msg_vt_bottle_state` (`0x0063b92c`) | `event_msg_bottle_state_serialize` / `_deserialize` | actor state + `u8 @+0x20` + `u32 @+0x1c` |
| `0x15` | `g_event_msg_vt_bottle_collision` (`0x0063b93c`) | `event_msg_bottle_collision_serialize` / `_deserialize` | actor state + `u8 @+0x1c` |

Both extend the shared **actor-state base payload** — `{vec3 quantised to i16, 3x i16, u32
guid, u8 flags}` — which `event_msg_actor_state_serialize` writes and four derived classes
chain to. Full write-up: `structs/net_event_msg.md`.

---

## 3. PC vs PS2

**The class is a straight port apart from networking.** Every gameplay constant matches:
the flight cut-off 50.0, the hit scale 15.0, the kind gate `{2, 10, 0x1b}`, the hit-type
gate `{0, 1, 2}`, and both I3D frame-flag masks.

**The field rebase is not a constant delta** — PC `+0x24` ↔ PS2 `+0x00` but PC `+0x12c` ↔
PS2 `+0x178` — because the `C_actor` base is larger on PS2 *and* its `S_vector` is a padded
vec4. **Map by RANK, never by subtracting.** The 22-store savegame pair gives the ranks
directly:

| PC | PS2 | what |
|---|---|---|
| +0x00 (vptr) | +0x78 | MSVC puts the vptr first, GCC 2.x after the data |
| +0x10 | +0x50 | kind |
| +0x24 | +0x00 | position |
| +0x30 | +0x10 | frame direction |
| +0x68 | +0x70 | I3D frame |
| +0x70 | +0x80 | flight direction |
| +0x7c | +0xe0 | flight speed |
| +0x80 | +0xe4 | fall accumulator |
| +0x84 | +0xe8 | landed-state selector |
| +0x8c | +0x90 | contact normal |
| +0xa4 | +0xf0 | radius |
| +0xa8 | +0xb0 | frame axis |
| +0xb4 | +0x100 | dynamic collision proxy |
| +0xfc | +0x160 | proxy-registered flag |
| +0x100 / +0x104 / +0x12c | +0x164 / +0x168 / +0x178 | bounce count, on-ground flag, ground material |

The savegame record is **0x53 bytes on the PC and 0x58 on PS2**: the PC packs the payload
byte-wise (store offsets 1, 5, 9, …) while PS2 writes dword-aligned slots.

### 3a. The PS2 port emptied the per-class network replication bodies

The PS2 build has no networking. Its ELF names eight IOP modules (`SIO2MAN`, `PADMAN`, `LIBSD`,
`SOUND`, `SOUND2`, `SDRDRV`, `MCMAN`, `MCSERV`) and none is a network module; `GM_NetPlayer` is a
`GM_Component` GUI widget, not a network class (`structs/net_event_msg.md` §7). What the PS2 keeps
is the `NetDirect` slot of the shared class interface, with **empty bodies**:

| PS2 `NetDirect` size | classes |
|---|---|
| **8 bytes (empty)** | **19** — `C_actor`, `C_actor_shot`, `C_airplane`, **`C_bottle`**, `C_bridge`, `C_crash_obj`, `C_detector`, `C_drop_in_weapon`, `C_entity`, `C_grenade`, `C_human`, `C_irenka`, `C_PhThing`, `C_public_physics`, `C_railway`, `C_smoke_trail`, `C_traffic_car`, `C_traffic_generator`, `C_wagon` |
| 16 bytes | `C_player` |
| 68 bytes | `C_door` |

So `C_bottle`'s PC vtable slot 15 holds **2112 bytes** of dead reckoning while
`NetDirect__8C_bottleFUi` is an 8-byte stub, and the PC's message receiver, its two senders and
the two event messages of §2b have **no PS2 symbol at all**. The same shape holds for `C_PhThing`, whose slot 15 holds a
**1696-byte** dead-reckoning body (`physical_object_net_apply_remote_state`) against the same
8-byte PS2 stub, plus five further PC-only replication members
(`structs/physical_object.md` §5), and for `C_box`.

⇒ **For this subsystem a PS2 method count is a LOWER bound, not the usual upper bound.** Both
directions exist — elsewhere the PS2 build *added* subsystems — so check which one you are in
before writing a terminal tag.

⇒ It is also why `bottle_net_apply_remote_state` is graded **STRUCTURAL and not CONFIRMED**.
The twin *symbol* is certain — vtable slot 15 is `NetDirect(Ui)` family-wide, byte-proven on
`C_airplane` whose slot 15 holds `stub_noop_args4_00461180` (`c2 04 00` = `ret 4`, exactly
one 4-byte argument, present in 30 vtables) — but an empty body cannot be body-diffed, and
"the symbol exists" is not the same evidence as "the bodies agree".

### 3b. The three PS2 methods with no distinct PC function

All folded onto shared PC stubs, none missing:

| PS2 method | Size | PC vtable slot | Folded onto |
|---|---|---|---|
| `DuplicateFrom__8C_bottleFP7C_actor` | 8 | 21 | `thunk_stub_noop_args4_00461180` (`0x004773f0`), a 5-byte JMP thunk ¹ |
| `SaveData__8C_bottleFP7C_chunk` | 8 | 19 | `stub_rettrue_args4_00477710` |
| `LoadData__8C_bottleFP7C_chunk` | 8 | 20 | **the same** `stub_rettrue_args4_00477710` |

¹ Ghidra names a thunk after its TARGET, so the slot resolves through the Ghidra symbol table,
not from the address: there is no `FUN_004773f0` id, but the thunk itself is a real function.

A bottle therefore carries **no mission-file state at all** — it is placed by the scene and
persists only through the savegame path. Two adjacent vtable slots sharing one stub is the
documented ICF fold (`structs/airplane.md` §3a).

### 3c. Corollary twin this class produced

| PC | PS2 | how |
|---|---|---|
| `chunk_read` | `Read__7C_chunkFPvi` | its own diff — the `max(0, remaining) < size` idiom |

### 3d. The subclass set on the PS2

`__ct__9C_grenadeFv` and `__ct__13C_smoke_trailFv` both chain `__ct__8C_bottleFv`, which
corroborates the §5 subclass shape independently. The shared slot-18 `Init` of §5 is three
separate 44-byte bodies on the PS2, identical to one another — which is why `/OPT:ICF` merged them
into `bottle_subclass_init_require_frame` on the PC.

---

## 5. The subclass set, and the slot-18 `Init` fold

`C_bottle` is a base; three PC subclasses are identified by kind byte:

| kind | class | PC ctor | doc |
|---|---|---|---|
| `0xe` | `C_grenade` | `grenade_ctor` `0x0048af30` | `structs/grenade.md` |
| `0xf` | `C_smoke_trail` | `smoke_trail_ctor` `0x004f60e0` | `structs/smoke_trail.md` |
| `0x10` | `C_cartridge` | `cartridge_ctor` `0x00476f40` | `structs/cartridge.md` |

A census of `bottle_ctor`'s callers finds exactly these three plus the two casing pools (PS2
corroboration: §3d).

**All three share ONE `Init` implementation**, `bottle_subclass_init_require_frame`
(`0x0048af90`), and that is a measurement: a scan of `.rdata` for the address as a pointer returns
exactly three datarefs — `0x0063be18` slot 18, `0x0063c3c0` slot 18, `0x0063cd80` slot 18. The body
is `frame ? bottle_init(frame) : false` — an `/OPT:ICF` merge of three identical bodies (§3d).
The PC symbol is therefore named for the FAMILY and carries **no `PS2 twin:` line**: a body shared by three classes has no single owner,
and naming it `grenade_init` would assert one (`structs/actor_track.md` §2c).

---

## 6. Where the shipped instances are, and what they do

### 6a. The scene census — `C_bottle` is scene-placed, exactly twice

Method: walk all **79** shipped `scene2.bin` (`0x4c53` root → `0xAE20` special-object block →
`0xAE21` object → `0xAE22` type dword, which is the value `mission_create_actor` switches on).

```
special type 0x0a (C_bottle):  2 occurrences, 1 scene
   MISE18-SALIERY   xplechovka3    0xAE24 payload EMPTY (6-byte header only)
   MISE18-SALIERY   xplechovka4    0xAE24 payload EMPTY (6-byte header only)
```

* **`plechovka` is Czech for *tin can*.** The models are `xplechovka3.i3d` / `xplechovka4.i3d`.
  The user's independent reading of the mesh — *"it looks like e.g. a paint tin can, not much
  like a bottle"* — agrees with the asset name. The class name is the developers' own symbol
  (`__ct__8C_bottleFv`) and stays; the **content** is cans.
* The empty `0xAE24` chunk independently confirms §3b: a bottle carries no mission-file state.
* `bottle_init` accepts any frame `actor_init` accepts, so both objects really do become live
  actors.
* **`MISE18-SALIERY` is textdb string 5490, "18-01 Just For Relaxation — Salieri Bar".**

**The sharpest control is inside the census itself.** Seventeen versions of the Salieri's bar
scene ship. `xplechovka3` and `xplechovka4` exist in all of them and are **plain static scene
meshes in sixteen** — only in `MISE18-SALIERY` are those two objects promoted to special type
`0x0a`. Somebody deliberately made two cans in the bar dynamic for one mission's opening.

A per-scene census is a **lower bound** on usage and can never prove absence — ten certainly-live
kinds appear in no scene either, because C++ code spawns them. Here it is used in the direction it
is valid in: to prove a **presence**. And `C_bottle` is alive a second way regardless, as the base
of three runtime classes (§5).

### 6b. Two things the class cannot do

| claim | measurement |
|---|---|
| picked up | **No.** No `S_using_object` in the layout (`C_door` has one at `+0x180`); no use/pick-up handler among the 20 functions; zero `using_object` / inventory / action-menu calls in the whole TU; and **zero `GetKind == 0xa` tests in the entire image** — all 346 `E8` callsites of `folded_get_dword_0x10` were decoded and the kind gates that exist are `0x1b`×108, `4`×84, `2`×80, `5`×10, `6`×8, `0x16`×4, `8`×3, … Nothing anywhere singles out a bottle by kind |
| thrown | **Not as a `C_bottle`.** All four callers of `bottle_set_flight_impulse` (`AbsoluteImpulse__8C_bottleFR8S_vector`) spawn a subclass: `game_new_grenade` (`C_grenade`, kind `0xe` — the molotov), `game_new_cartridge` ×2 (`C_cartridge`, `0x10`), `game_new_smoke_trail` (`C_smoke_trail`, `0xf`). No caller launches a kind-`0xa` instance |

A shipped can can therefore only be moved by **being shot** (`bottle_hit`: hit-type gate
`{0,1,2}`, incoming vector scaled ×15.0 with `abs(y)`, tumble re-rolled from 3× `rand_float`,
then `actor_set_act_state(0)` to start the arc) or by **being walked into** (`bottle_on_collision`:
the *other* actor's kind gated on `{2 C_player, 10 C_bottle, 0x1b C_entity}` — note a **car,
kind 4, is not in that gate**, so you cannot drive one over).

### 6c. In-game observation — the behaviour is not distinguishable from a static mesh

The player-tester, testing `MISE18-SALIERY` directly:

> *"`xplechovka3`/`xplechovka4` and `xplechovka`/`xplechovka2` are behaving the same. But this
> behavior is very limited — the same sound is emitted when hit with a gun (metal cing) … From
> an in-game perspective there is no difference between `xplechovka` and `xplechovka3`."*

**One half of that is fully explained by the static reading.** `xplechovka` and `xplechovka2`
are **not special objects in that scene at all** — the census returns only `xplechovka3` and
`xplechovka4` for type `0x0a`, and the scene's whole type histogram is
`{0x02, 0x04, 0x05, 0x06, 0x0a×2, 0x0c, 0x12, 0x1b, 0x23}`. So two of the four cans the user
shot are plain meshes. And **`C_bottle` never plays a sound**: there is not one sound call in any
of its twenty functions. The metal *cing* is the shared bullet-impact material sound, which a
static mesh produces too — identical audio is exactly what the code predicts, for all four.

**The other half is not:** `bottle_hit` writes the flight band and sets act-state 0, so a shot can
should visibly move, and the tester did not observe it move (§7).

⇒ **`C_bottle` is used, but its shipped presence is two tin cans whose extra behaviour no player
is likely to notice.** That is a different statement from `seagull_flock` and `bendable_plant`,
which have **zero** scene instances and no located construction path at all.

---

## 7. Open items

| item | state |
|---|---|
| in game: do the two dynamic cans move when shot? | shoot the four cans in `MISE18-SALIERY` one at a time and watch only for movement: `xplechovka3`/`xplechovka4` are the dynamic pair, `xplechovka`, `xplechovka2`, `xplechovka5`, `xplechovka6` are static. Candidates if they do not move: the launch is too small to read (the ×15 scale acts on an unmeasured hit vector); the dynamic pair was not the pair shot; the `Hit` never reaches them; or a degenerate bound volume leaves the collision proxy unhittable while `+0xa4` looks correct |

**Related:** `structs/actor.md` (base class) · `structs/airplane.md` (the preceding class in
the same link group) · `structs/entity.md` (the kind enum) · `structs/net_event_msg.md` ·
`structs/net_session.md`.
