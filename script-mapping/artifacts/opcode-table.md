# Mafia script-VM - the complete opcode table (GENERATED)

**Created** 2026-09-07 22:05 +0200 · **Modified** 2026-10-12 20:58 +0200

Generated from `Game.exe`; regenerate it rather than editing it by hand.

Byte-extracted from the keyword table that `program_decode_instruction` (`0x005b1b60`) materialises on its own stack: **391 records of `struct { char name[0x20]; int opcode; }`, stride `0x24`**, recovered by emulating the unrolled initialiser. Full write-up: `basic/artifacts/script-opcode-dispatch.md`.

🔑 **The opcode IS the dispatcher case label.** `program_process_command` (`0x005bb2d0`) does `dec eax; cmp eax,0x185; ja default; jmp [eax*4 + 0x005c7700]`, so the jump table is biased by `-1` only because opcode 0 (`LABEL`) has no arm.

Validated on generation: 391 records, opcodes a bijection onto `0x000..0x186` (no gaps, no collisions, no duplicate names); 8 independently recorded case anchors match; every opcode `1..0x186` has a jump-table arm.

🔴 **A DEFAULT-ARM SLOT DOES NOT MEAN UNIMPLEMENTED.** `C_program::Process` is VIRTUAL, and `C_detector` overrides it: `detector_program_process` handles `DETECTOR_INRANGE`, `DETECTOR_WAITFORUSE`, `DETECTOR_WAITFORHIT`, `PROGRAM_STORAGE`, `DETECTOR_CREATEDYNCOLL`, `DETECTOR_ERASEDYNCOLL` itself and delegates everything else to the base. Their base-dispatcher slot points at the default arm precisely BECAUSE the derived class owns them. Two of them - `DETECTOR_WAITFORUSE` and `DETECTOR_WAITFORHIT` - are the *wait* primitives: the override deliberately does not advance the program counter, which is what makes the script block until `detector_use` / `detector_hit` releases it. See `structs/detector.md` §6a.

🔴 **AND A DEFAULT-ARM SLOT DOES NOT MEAN ONE THING EITHER.** The nine rows left after the override map split three ways by the LOAD-TIME argument decoder (`program_decode_params`: 384 of 391 opcodes have an arm, including a shared `return 1` arm for zero-argument commands, and the switch has no `default:` so an unmatched opcode returns 0 = a decode failure):

* **implemented as a MARKER (1)**: `EVENT` - a jump-target label, not a dispatched command. In game, `event` works.
* **compiles, then silently ignored (1)**: `HUMAN_CANDIE` - a parser arm exists, no dispatcher arm does. No error, no effect.
* **fails to COMPILE (7)**: `BEEP`, `DEBUG_TEXT`, `[BREAK]`, `[DEBUG_ME]`, `[DEBUG_STOP]`, `DEBUG_GETFRAMEINFO`, `[DAN_DEBUG]` - no parser arm at all. In game, `debug_text` throws a compilation error, which validates the split.

| Opcode | Dec | Command | Dispatcher case | Arm VA | Table index | Notes |
|---:|---:|---|---|---|---:|---|
| `0x000` | 0 | `LABEL` | `-` | `-` | 0 | no dispatcher arm - the jump table is indexed by `opcode - 1` |
| `0x001` | 1 | `GOTO` | `case 0x1` | `0x005bb356` | 1 | - |
| `0x002` | 2 | `GOSUB` | `case 0x2` | `0x005bb33f` | 2 | - |
| `0x003` | 3 | `RETURN` | `case 0x3` | `0x005bb3fe` | 3 | - |
| `0x004` | 4 | `WAIT` | `case 0x4` | `0x005bb3bc` | 4 | - |
| `0x005` | 5 | `BEEP` | `case 0x5` | `0x005c76da` | 5 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x006` | 6 | `END` | `case 0x6` | `0x005bb333` | 6 | - |
| `0x007` | 7 | `DIM_FLT` | `case 0x7` | `0x005bb495` | 7 | - |
| `0x008` | 8 | `LET` | `case 0x8` | `0x005bb528` | 8 | - |
| `0x009` | 9 | `IF` | `case 0x9` | `0x005bb601` | 9 | - |
| `0x00a` | 10 | `RND` | `case 0xa` | `0x005bb6b4` | 10 | - |
| `0x00b` | 11 | `DIM_FRM` | `case 0xb` | `0x005bb701` | 11 | - |
| `0x00c` | 12 | `DIM_ACT` | `case 0xc` | `0x005bb776` | 12 | - |
| `0x00d` | 13 | `FINDFRAME` | `case 0xd` | `0x005bb804` | 13 | - |
| `0x00e` | 14 | `FRM_SETON` | `case 0xe` | `0x005bba81` | 15 | - |
| `0x00f` | 15 | `FRM_ISON` | `case 0xf` | `0x005c74dd` | 386 | - |
| `0x010` | 16 | `FRM_LINKTO` | `case 0x10` | `0x005bbbc0` | 16 | - |
| `0x011` | 17 | `FINDACTOR` | `case 0x11` | `0x005bb871` | 14 | - |
| `0x012` | 18 | `CALL` | `case 0x12` | `0x005bbc56` | 17 | - |
| `0x013` | 19 | `IFFLTINRANGE` | `case 0x13` | `0x005bbd55` | 18 | - |
| `0x014` | 20 | `PLAYSOUND` | `case 0x14` | `0x005bbdaa` | 19 | - |
| `0x015` | 21 | `PLAYSOUNDSTOP` | `case 0x15` | `0x005bbe80` | 20 | - |
| `0x016` | 22 | `PLAYSOUNDEX` | `case 0x16` | `0x005bfec9` | 21 | - |
| `0x017` | 23 | `STOPSOUND` | `case 0x17` | `0x005bff43` | 22 | - |
| `0x018` | 24 | `GETSOUNDTIME` | `case 0x18` | `0x005bff9a` | 23 | - |
| `0x019` | 25 | `ACT_SETSTATE` | `case 0x19` | `0x005bbee5` | 24 | - |
| `0x01a` | 26 | `DETECTOR_ISSIGNAL` | `case 0x1a` | `0x005bbfa0` | 25 | - |
| `0x01b` | 27 | `DETECTOR_SETSIGNAL` | `case 0x1b` | `0x005bbf4f` | 26 | - |
| `0x01c` | 28 | `DETECTOR_INRANGE` | `case 0x1c` | `0x005c76da` | 27 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x01d` | 29 | `SETCOMPASS` | `case 0x1d` | `0x005bc005` | 28 | - |
| `0x01e` | 30 | `SETLMLEVEL` | `case 0x1e` | `0x005bc069` | 29 | - |
| `0x01f` | 31 | `DETECTOR_WAITFORUSE` | `case 0x1f` | `0x005c76da` | 30 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x020` | 32 | `DETECTOR_WAITFORHIT` | `case 0x20` | `0x005c76da` | 31 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x021` | 33 | `COMMANDBLOCK` | `case 0x21` | `0x005bc267` | 32 | - |
| `0x022` | 34 | `CALLSUBROUTINE` | `case 0x22` | `0x005bc580` | 33 | - |
| `0x023` | 35 | `SOUNDFADE` | `case 0x23` | `0x005bc59f` | 34 | - |
| `0x024` | 36 | `ENDOFMISSION` | `case 0x24` | `0x005bf4df` | 35 | - |
| `0x025` | 37 | `PROGRAM_STORAGE` | `case 0x25` | `0x005c76da` | 36 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x026` | 38 | `DOOR_LOCK` | `case 0x26` | `0x005bc111` | 37 | - |
| `0x027` | 39 | `DOOR_OPEN` | `case 0x27` | `0x005bc16d` | 38 | - |
| `0x028` | 40 | `DOOR_ENABLEUS` | `case 0x28` | `0x005bc20b` | 39 | - |
| `0x029` | 41 | `RECCLEAR` | `case 0x29` | `0x005bc283` | 40 | - |
| `0x02a` | 42 | `RECADDACTOR` | `case 0x2a` | `0x005bc292` | 41 | - |
| `0x02b` | 43 | `RECLOAD` | `case 0x2b` | `0x005bc2cf` | 42 | - |
| `0x02c` | 44 | `RECLOADFULL` | `case 0x2c` | `0x005bc33a` | 43 | - |
| `0x02d` | 45 | `RECWAITFOREND` | `case 0x2d` | `0x005bc492` | 44 | - |
| `0x02e` | 46 | `RECUNLOAD` | `case 0x2e` | `0x005bc4a9` | 45 | - |
| `0x02f` | 47 | `LOADCOLLTREE` | `case 0x2f` | `0x005bc64f` | 46 | - |
| `0x030` | 48 | `EXPLOSION` | `case 0x30` | `0x005bc698` | 47 | - |
| `0x031` | 49 | `TIMERON` | `case 0x31` | `0x005bc715` | 48 | - |
| `0x032` | 50 | `TIMEROFF` | `case 0x32` | `0x005bc704` | 49 | - |
| `0x033` | 51 | `CHANGE_MISSION` | `case 0x33` | `0x005bc75c` | 50 | - |
| `0x034` | 52 | `CAMERA_SETSWING` | `case 0x34` | `0x005bca5a` | 51 | - |
| `0x035` | 53 | `AIRPLANE_START` | `case 0x35` | `0x005bcabe` | 52 | - |
| `0x036` | 54 | `AIRPLANE_START2` | `case 0x36` | `0x005c0cac` | 53 | - |
| `0x037` | 55 | `DEBUG_TEXT` | `case 0x37` | `0x005c76da` | 54 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x038` | 56 | `GETACTORSDIST` | `case 0x38` | `0x005bcb21` | 55 | - |
| `0x039` | 57 | `GETCARDAMAGE` | `case 0x39` | `0x005bcc2b` | 56 | - |
| `0x03a` | 58 | `HUMAN_ADDWEAPON` | `case 0x3a` | `0x005bcca5` | 57 | - |
| `0x03b` | 59 | `HUMAN_CANADDWEAPON` | `case 0x3b` | `0x005bcd70` | 58 | - |
| `0x03c` | 60 | `HUMAN_DELWEAPON` | `case 0x3c` | `0x005bce1a` | 59 | - |
| `0x03d` | 61 | `HUMAN_ISWEAPON` | `case 0x3d` | `0x005bd0cd` | 60 | - |
| `0x03e` | 62 | `HUMAN_ACTIVATEWEAPON` | `case 0x3e` | `0x005bceae` | 117 | - |
| `0x03f` | 63 | `HUMAN_ANYWEAPONINHAND` | `case 0x3f` | `0x005bcf95` | 118 | - |
| `0x040` | 64 | `HUMAN_ANYWEAPONININVENTORY` | `case 0x40` | `0x005bd031` | 119 | - |
| `0x041` | 65 | `HUMAN_GETITEMINRHAND` | `case 0x41` | `0x005c7555` | 387 | - |
| `0x042` | 66 | `PRELOADMODEL` | `case 0x42` | `0x005bd171` | 61 | - |
| `0x043` | 67 | `LOADDIFFERENCES` | `case 0x43` | `0x005bd1b4` | 62 | - |
| `0x044` | 68 | `CLEARDIFFERENCES` | `case 0x44` | `0x005bd1da` | 63 | - |
| `0x045` | 69 | `{` | `case 0x45` | `0x005bd1f1` | 64 | - |
| `0x046` | 70 | `}` | `case 0x46` | `0x005bd20d` | 65 | - |
| `0x047` | 71 | `SETAIPRIORITY` | `case 0x47` | `0x005bd21a` | 66 | - |
| `0x048` | 72 | `EVENT` | `case 0x48` | `0x005c76da` | 67 | **IMPLEMENTED** - not a dispatched command but a jump-target MARKER: it compiles to an instruction of type `0x48` that `program_set_event` scans for and jumps to. The default arm (advance the pc) is the correct fall-through |
| `0x049` | 73 | `SETEVENT` | `case 0x49` | `0x005bd274` | 68 | - |
| `0x04a` | 74 | `EVENT_USE_CB` | `case 0x4a` | `0x005bd300` | 69 | - |
| `0x04b` | 75 | `ENEMY_CHANGEANIM` | `case 0x4b` | `0x005bd343` | 70 | - |
| `0x04c` | 76 | `HUMAN_CHANGEANIM` | `case 0x4c` | `0x005bd38f` | 71 | - |
| `0x04d` | 77 | `DIALOG_BEGIN` | `case 0x4d` | `0x005bd419` | 72 | - |
| `0x04e` | 78 | `DIALOG_END` | `case 0x4e` | `0x005bd4f1` | 73 | - |
| `0x04f` | 79 | `DIALOG_CAMSWITCH` | `case 0x4f` | `0x005bd55d` | 74 | - |
| `0x050` | 80 | `GAME_NIGHTMISSION` | `case 0x50` | `0x005bd644` | 75 | - |
| `0x051` | 81 | `HUMAN_LOOKTOACTOR` | `case 0x51` | `0x005bd670` | 76 | - |
| `0x052` | 82 | `HUMAN_LOOKTOFRAME` | `case 0x52` | `0x005bd767` | 77 | - |
| `0x053` | 83 | `GETMISSIONNUMBER` | `case 0x53` | `0x005bd865` | 78 | - |
| `0x054` | 84 | `SETMISSIONNUMBER` | `case 0x54` | `0x005bd89e` | 79 | - |
| `0x055` | 85 | `HUMAN_LINKTOHAND` | `case 0x55` | `0x005bd8e2` | 80 | - |
| `0x056` | 86 | `HUMAN_UNLINKFROMHAND` | `case 0x56` | `0x005bda53` | 81 | - |
| `0x057` | 87 | `CREATEWEAPONFROMFRAME` | `case 0x57` | `0x005bdd06` | 82 | - |
| `0x058` | 88 | `GARAGE_ADDCAR` | `case 0x58` | `0x005bdf1c` | 83 | - |
| `0x059` | 89 | `GARAGE_ADDCARIDX` | `case 0x59` | `0x005be067` | 84 | - |
| `0x05a` | 90 | `GARAGE_DELCAR` | `case 0x5a` | `0x005bdfc0` | 85 | - |
| `0x05b` | 91 | `GARAGE_GENERATECARS` | `case 0x5b` | `0x005be08f` | 86 | - |
| `0x05c` | 92 | `GARAGE_RELEASECARS` | `case 0x5c` | `0x005be09e` | 87 | - |
| `0x05d` | 93 | `GARAGE_CARMANAGER` | `case 0x5d` | `0x005be0ad` | 88 | - |
| `0x05e` | 94 | `PLAYER_LOCKCONTROLS` | `case 0x5e` | `0x005be0bc` | 89 | - |
| `0x05f` | 95 | `PERSON_PLAYANIM` | `case 0x5f` | `0x005be122` | 90 | - |
| `0x060` | 96 | `PERSON_STOPANIM` | `case 0x60` | `0x005be1b3` | 91 | - |
| `0x061` | 97 | `GUNSHOP_MENU` | `case 0x61` | `0x005be210` | 92 | - |
| `0x062` | 98 | `HUMAN_SET8SLOT` | `case 0x62` | `0x005be2b1` | 93 | - |
| `0x063` | 99 | `ENEMY_FORCESCRIPT` | `case 0x63` | `0x005be388` | 94 | - |
| `0x064` | 100 | `COMPAREOWNERWITHEX` | `case 0x64` | `0x005be3d7` | 95 | - |
| `0x065` | 101 | `HUMAN_SETPROPERTY` | `case 0x65` | `0x005be554` | 96 | - |
| `0x066` | 102 | `HUMAN_GETPROPERTY` | `case 0x66` | `0x005be47d` | 97 | - |
| `0x067` | 103 | `DISABLECOLLS` | `case 0x67` | `0x005be7bb` | 98 | - |
| `0x068` | 104 | `HUMAN_GETOWNER` | `case 0x68` | `0x005be85d` | 99 | - |
| `0x069` | 105 | `HUMAN_FORCE_SETTOCAR` | `case 0x69` | `0x005be92d` | 100 | - |
| `0x06a` | 106 | `CAR_SETDESTROYMOTOR` | `case 0x6a` | `0x005be9d1` | 101 | - |
| `0x06b` | 107 | `ACTOR_SETDIR` | `case 0x6b` | `0x005bea2d` | 102 | - |
| `0x06c` | 108 | `ACTOR_SETPOS` | `case 0x6c` | `0x005bece1` | 104 | - |
| `0x06d` | 109 | `ACTOR_SETPLACEMENT` | `case 0x6d` | `0x005bee53` | 103 | - |
| `0x06e` | 110 | `GARAGE_ENABLESTEAL` | `case 0x6e` | `0x005bf105` | 105 | - |
| `0x06f` | 111 | `SETTIMEOUTEVENT` | `case 0x6f` | `0x005bf125` | 106 | - |
| `0x070` | 112 | `EMITPARTICLE` | `case 0x70` | `0x005bf247` | 107 | - |
| `0x071` | 113 | `STOPPARTICLE` | `case 0x71` | `0x005bf327` | 108 | - |
| `0x072` | 114 | `SETPLAYERFIREEVENT` | `case 0x72` | `0x005bf1b9` | 109 | - |
| `0x073` | 115 | `FINDNEARACTOR` | `case 0x73` | `0x005bf36d` | 110 | - |
| `0x074` | 116 | `SETPLAYERHORNEVENT` | `case 0x74` | `0x005bf6fd` | 111 | - |
| `0x075` | 117 | `HUMAN_HOLSTER` | `case 0x75` | `0x005bf830` | 113 | - |
| `0x076` | 118 | `CAMERA_LOCK` | `case 0x76` | `0x005bf88d` | 114 | - |
| `0x077` | 119 | `CAMERA_UNLOCK` | `case 0x77` | `0x005bf914` | 115 | - |
| `0x078` | 120 | `SETMISSIONNAMEID` | `case 0x78` | `0x005bf946` | 116 | - |
| `0x079` | 121 | `WINGMAN_SETINDICATOR` | `case 0x79` | `0x005bf966` | 120 | - |
| `0x07a` | 122 | `WINGMAN_DELINDICATOR` | `case 0x7a` | `0x005bfaa8` | 213 | - |
| `0x07b` | 123 | `MODEL_PLAYANIM` | `case 0x7b` | `0x005bfc80` | 123 | - |
| `0x07c` | 124 | `MODEL_STOPANIM` | `case 0x7c` | `0x005bfd31` | 124 | - |
| `0x07d` | 125 | `MODEL_CREATE` | `case 0x7d` | `0x005bfb6f` | 121 | - |
| `0x07e` | 126 | `MODEL_DESTROY` | `case 0x7e` | `0x005bfc14` | 122 | - |
| `0x07f` | 127 | `HUMAN_CHANGEMODEL` | `case 0x7f` | `0x005bfd99` | 125 | - |
| `0x080` | 128 | `AUTOSAVEGAME` | `case 0x80` | `0x005bfe02` | 126 | - |
| `0x081` | 129 | `AUTOSAVEGAMEFULL` | `case 0x81` | `0x005bfe2d` | 127 | - |
| `0x082` | 130 | `HUMAN_DEATH` | `case 0x82` | `0x005bfe58` | 128 | - |
| `0x083` | 131 | `HUMAN_TALK` | `case 0x83` | `0x005c0020` | 129 | - |
| `0x084` | 132 | `HUMAN_ERASEAB` | `case 0x84` | `0x005c0112` | 130 | - |
| `0x085` | 133 | `HUMAN_CREATEAB` | `case 0x85` | `0x005c016a` | 131 | - |
| `0x086` | 134 | `POLICEMANAGER_ON` | `case 0x86` | `0x005c01c9` | 132 | - |
| `0x087` | 135 | `POLICEMANAGER_ADD` | `case 0x87` | `0x005c0203` | 133 | - |
| `0x088` | 136 | `POLICEMANAGER_DEL` | `case 0x88` | `0x005c032f` | 134 | - |
| `0x089` | 137 | `POLICEMANAGER_SETSPEED` | `case 0x89` | `0x005c2306` | 199 | - |
| `0x08a` | 138 | `POLICEMANAGER_FORCEARREST` | `case 0x8a` | `0x005c2485` | 204 | - |
| `0x08b` | 139 | `CAR_ENABLEUS` | `case 0x8b` | `0x005c03ed` | 135 | - |
| `0x08c` | 140 | `CAR_SETDOOROPEN` | `case 0x8c` | `0x005c0449` | 136 | - |
| `0x08d` | 141 | `HUMAN_SETFIRETARGET` | `case 0x8d` | `0x005c0515` | 137 | - |
| `0x08e` | 142 | `ISCARUSABLE` | `case 0x8e` | `0x005c05a3` | 142 | - |
| `0x08f` | 143 | `DOOR_SETOPENPOSE` | `case 0x8f` | `0x005c06a5` | 138 | - |
| `0x090` | 144 | `AIRPLANE_ISDESTROYED` | `case 0x90` | `0x005c0625` | 139 | - |
| `0x091` | 145 | `HUMAN_RETURNFROMPANIC` | `case 0x91` | `0x005c073c` | 140 | - |
| `0x092` | 146 | `CAR_GETSPEED` | `case 0x92` | `0x005c0798` | 141 | - |
| `0x093` | 147 | `CAR_SETSPEED` | `case 0x93` | `0x005c2563` | 206 | - |
| `0x094` | 148 | `CAR_FORCESTOP` | `case 0x94` | `0x005c2a02` | 212 | - |
| `0x095` | 149 | `CAR_BREAKMOTOR` | `case 0x95` | `0x005c0813` | 143 | - |
| `0x096` | 150 | `INVENTORY_PUSH` | `case 0x96` | `0x005c086f` | 144 | - |
| `0x097` | 151 | `INVENTORY_POP` | `case 0x97` | `0x005c08dc` | 145 | - |
| `0x098` | 152 | `FUCKINGBOX_ADD` | `case 0x98` | `0x005c0b39` | 149 | - |
| `0x099` | 153 | `FUCKINGBOX_RECOMPILE` | `case 0x99` | `0x005c0b84` | 150 | - |
| `0x09a` | 154 | `FUCKINGBOX_ADD_DEST` | `case 0x9a` | `0x005c0b9b` | 151 | - |
| `0x09b` | 155 | `HUMAN_THROWITEM` | `case 0x9b` | `0x005c0a2e` | 148 | - |
| `0x09c` | 156 | `FUCKINGBOX_GETNUMBOX` | `case 0x9c` | `0x005c0be6` | 152 | - |
| `0x09d` | 157 | `FUCKINGBOX_GETNUMDEST` | `case 0x9d` | `0x005c0c35` | 153 | - |
| `0x09e` | 158 | `FUCKINGBOX_MOVE` | `case 0x9e` | `0x005c0c84` | 154 | - |
| `0x09f` | 159 | `WAGON_SETEVENT` | `case 0x9f` | `0x005c0cf9` | 155 | - |
| `0x0a0` | 160 | `WAGON_GETLASTNODE` | `case 0xa0` | `0x005c0d4f` | 156 | - |
| `0x0a1` | 161 | `CAR_SETSPECCOLL` | `case 0xa1` | `0x005c0dc7` | 157 | - |
| `0x0a2` | 162 | `CAR_GETSPECCOLL` | `case 0xa2` | `0x005c0ef5` | 158 | - |
| `0x0a3` | 163 | `MISSION_OBJECTIVES` | `case 0xa3` | `0x005c0f72` | 159 | - |
| `0x0a4` | 164 | `MISSION_OBJECTIVESREMOVE` | `case 0xa4` | `0x005c0f92` | 160 | - |
| `0x0a5` | 165 | `MISSION_OBJECTIVESCLEAR` | `case 0xa5` | `0x005c0fb2` | 161 | - |
| `0x0a6` | 166 | `RACING_MISSION6_INIT` | `case 0xa6` | `0x005c0fc1` | 162 | - |
| `0x0a7` | 167 | `RACING_MISSION6_START` | `case 0xa7` | `0x005c0fe6` | 163 | - |
| `0x0a8` | 168 | `CAR_EXPLOSION` | `case 0xa8` | `0x005c1067` | 164 | - |
| `0x0a9` | 169 | `HUMAN_FROMCAR` | `case 0xa9` | `0x005c10b6` | 165 | - |
| `0x0aa` | 170 | `[BREAK]` | `case 0xaa` | `0x005c76da` | 166 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x0ab` | 171 | `CAR_REMOVE_DRIVER` | `case 0xab` | `0x005c11d5` | 167 | - |
| `0x0ac` | 172 | `SETMODELTOCAR` | `case 0xac` | `0x005c1237` | 168 | - |
| `0x0ad` | 173 | `GETENEMYAISTATE` | `case 0xad` | `0x005c130e` | 169 | - |
| `0x0ae` | 174 | `GET_PM_STATE` | `case 0xae` | `0x005c13a4` | 170 | - |
| `0x0af` | 175 | `GET_PM_CRASHTIME` | `case 0xaf` | `0x005c1459` | 171 | - |
| `0x0b0` | 176 | `GET_PM_FIRETIME` | `case 0xb0` | `0x005c260c` | 172 | - |
| `0x0b1` | 177 | `GET_PM_NUMPREDATORS` | `case 0xb1` | `0x005c26c4` | 173 | - |
| `0x0b2` | 178 | `GET_PM_HUMANSTATE` | `case 0xb2` | `0x005c75ef` | 388 | - |
| `0x0b3` | 179 | `USE_LIGHTCACHE` | `case 0xb3` | `0x005c1511` | 174 | - |
| `0x0b4` | 180 | `STREAM_CREATE` | `case 0xb4` | `0x005c1528` | 175 | - |
| `0x0b5` | 181 | `STREAM_CONNECT` | `case 0xb5` | `0x005c1711` | 176 | - |
| `0x0b6` | 182 | `STREAM_PLAY` | `case 0xb6` | `0x005c1589` | 177 | - |
| `0x0b7` | 183 | `STREAM_PAUSE` | `case 0xb7` | `0x005c161f` | 178 | - |
| `0x0b8` | 184 | `STREAM_STOP` | `case 0xb8` | `0x005c1671` | 179 | - |
| `0x0b9` | 185 | `STREAM_DESTROY` | `case 0xb9` | `0x005c16cd` | 180 | - |
| `0x0ba` | 186 | `STREAM_FADEVOL` | `case 0xba` | `0x005c181d` | 181 | - |
| `0x0bb` | 187 | `STREAM_SETLOOP` | `case 0xbb` | `0x005c186f` | 182 | - |
| `0x0bc` | 188 | `STREAM_SETPOS` | `case 0xbc` | `0x005c5c5b` | 331 | - |
| `0x0bd` | 189 | `STREAM_GETPOS` | `case 0xbd` | `0x005c5cd3` | 332 | - |
| `0x0be` | 190 | `SETTANKHITCOUNT` | `case 0xbe` | `0x005c18be` | 183 | - |
| `0x0bf` | 191 | `SETNOANIMHIT` | `case 0xbf` | `0x005c1915` | 184 | - |
| `0x0c0` | 192 | `CARLIGHT_MAIN` | `case 0xc0` | `0x005c1983` | 185 | - |
| `0x0c1` | 193 | `CARLIGHT_LIGHT` | `case 0xc1` | `0x005c1a63` | 186 | - |
| `0x0c2` | 194 | `CARLIGHT_INDIC_L` | `case 0xc2` | `0x005c1b43` | 187 | - |
| `0x0c3` | 195 | `CARLIGHT_INDIC_R` | `case 0xc3` | `0x005c1c17` | 188 | - |
| `0x0c4` | 196 | `CARLIGHT_INDIC_OFF` | `case 0xc4` | `0x005c1cdf` | 304 | - |
| `0x0c5` | 197 | `CAR_MUSTSTEAL` | `case 0xc5` | `0x005c1d79` | 189 | - |
| `0x0c6` | 198 | `CAR_SETPROJECTOR` | `case 0xc6` | `0x005c1edf` | 190 | - |
| `0x0c7` | 199 | `GARAGE_CANSTEAL` | `case 0xc7` | `0x005c1dd5` | 191 | - |
| `0x0c8` | 200 | `GARAGE_ADDLASTSTOLEN` | `case 0xc8` | `0x005c234d` | 200 | - |
| `0x0c9` | 201 | `GARAGE_SHOW` | `case 0xc9` | `0x005c235c` | 201 | - |
| `0x0ca` | 202 | `CAR_INWATER` | `case 0xca` | `0x005c1e60` | 192 | - |
| `0x0cb` | 203 | `HUMAN_FORCEFALL` | `case 0xcb` | `0x005c1f5b` | 193 | - |
| `0x0cc` | 204 | `CREATE_PHYSICALOBJECT` | `case 0xcc` | `0x005c1fc5` | 194 | - |
| `0x0cd` | 205 | `DESTROY_PHYSICALOBJECT` | `case 0xcd` | `0x005c4da6` | 294 | - |
| `0x0ce` | 206 | `WEATHER_RESET` | `case 0xce` | `0x005c2157` | 195 | - |
| `0x0cf` | 207 | `WEATHER_SETPARAM` | `case 0xcf` | `0x005c2170` | 196 | - |
| `0x0d0` | 208 | `GETTICKTIME` | `case 0xd0` | `0x005c227c` | 197 | - |
| `0x0d1` | 209 | `GETGAMETIME` | `case 0xd1` | `0x005c22b8` | 198 | - |
| `0x0d2` | 210 | `GETACTIVEPLAYER` | `case 0xd2` | `0x005c236b` | 202 | - |
| `0x0d3` | 211 | `GETANGLEACTORTOACTOR` | `case 0xd3` | `0x005c23a9` | 203 | - |
| `0x0d4` | 212 | `SETCITYTRAFFICVISIBLE` | `case 0xd4` | `0x005c2511` | 205 | - |
| `0x0d5` | 213 | `SHOWCARDAMAGE` | `case 0xd5` | `0x005c2774` | 207 | - |
| `0x0d6` | 214 | `CHARACTER_PUSH` | `case 0xd6` | `0x005c0954` | 146 | - |
| `0x0d7` | 215 | `CHARACTER_POP` | `case 0xd7` | `0x005c09c1` | 147 | - |
| `0x0d8` | 216 | `CONSOLE_ADDTEXT` | `case 0xd8` | `0x005c253e` | 208 | - |
| `0x0d9` | 217 | `CAMERA_SETRANGE` | `case 0xd9` | `0x005c27d0` | 209 | - |
| `0x0da` | 218 | `ACTOR_DUPLICATE` | `case 0xda` | `0x005c2804` | 210 | - |
| `0x0db` | 219 | `ACTOR_DELETE` | `case 0xdb` | `0x005c29cd` | 211 | - |
| `0x0dc` | 220 | `HUMAN_RESET` | `case 0xdc` | `0x005c2a5f` | 214 | - |
| `0x0dd` | 221 | `HUMAN_HAVEBOX` | `case 0xdd` | `0x005c2abc` | 215 | - |
| `0x0de` | 222 | `HUMAN_HAVEBODY` | `case 0xde` | `0x005c2b8b` | 216 | - |
| `0x0df` | 223 | `HUMAN_STOPTALK` | `case 0xdf` | `0x005c2c28` | 217 | - |
| `0x0e0` | 224 | `ENEMY_BLOCK` | `case 0xe0` | `0x005c2c93` | 218 | - |
| `0x0e1` | 225 | `ENEMY_UNBLOCK` | `case 0xe1` | `0x005c2c99` | 219 | - |
| `0x0e2` | 226 | `GET_REMOTE_FLOAT` | `case 0xe2` | `0x005c2cb4` | 220 | - |
| `0x0e3` | 227 | `GET_REMOTE_FRAME` | `case 0xe3` | `0x005c2d53` | 221 | - |
| `0x0e4` | 228 | `GET_REMOTE_ACTOR` | `case 0xe4` | `0x005c2dde` | 222 | - |
| `0x0e5` | 229 | `SET_REMOTE_FLOAT` | `case 0xe5` | `0x005c2e78` | 223 | - |
| `0x0e6` | 230 | `SET_REMOTE_FRAME` | `case 0xe6` | `0x005c2f19` | 224 | - |
| `0x0e7` | 231 | `SET_REMOTE_ACTOR` | `case 0xe7` | `0x005c2fba` | 225 | - |
| `0x0e8` | 232 | `GETACTORFRAME` | `case 0xe8` | `0x005c3056` | 226 | - |
| `0x0e9` | 233 | `ZATMYSE` | `case 0xe9` | `0x005c30b2` | 227 | - |
| `0x0ea` | 234 | `CARDAMAGEVISIBLE` | `case 0xea` | `0x005c30f1` | 228 | - |
| `0x0eb` | 235 | `[DEBUG_ME]` | `case 0xeb` | `0x005c76da` | 229 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x0ec` | 236 | `[DEBUG_STOP]` | `case 0xec` | `0x005c76da` | 230 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x0ed` | 237 | `IFPLAYERSTEALCAR` | `case 0xed` | `0x005c319b` | 231 | - |
| `0x0ee` | 238 | `INTRO_SUBTITLE_ADD` | `case 0xee` | `0x005c3244` | 232 | - |
| `0x0ef` | 239 | `SUBTITLE_ADD` | `case 0xef` | `0x005c6e0f` | 370 | - |
| `0x0f0` | 240 | `MATH_COS` | `case 0xf0` | `0x005c3268` | 233 | - |
| `0x0f1` | 241 | `MATH_SIN` | `case 0xf1` | `0x005c32a1` | 234 | - |
| `0x0f2` | 242 | `MATH_ABS` | `case 0xf2` | `0x005c32da` | 235 | - |
| `0x0f3` | 243 | `FRM_GETDIR` | `case 0xf3` | `0x005c330d` | 236 | - |
| `0x0f4` | 244 | `FRM_SETDIR` | `case 0xf4` | `0x005c339b` | 237 | - |
| `0x0f5` | 245 | `FRM_GETPOS` | `case 0xf5` | `0x005c3429` | 238 | - |
| `0x0f6` | 246 | `FRM_SETPOS` | `case 0xf6` | `0x005c34ba` | 239 | - |
| `0x0f7` | 247 | `FRM_GETSCALE` | `case 0xf7` | `0x005c358f` | 240 | - |
| `0x0f8` | 248 | `FRM_SETSCALE` | `case 0xf8` | `0x005c3620` | 241 | - |
| `0x0f9` | 249 | `FRM_GETROT` | `case 0xf9` | `0x005c46e3` | 242 | - |
| `0x0fa` | 250 | `FRM_SETROT` | `case 0xfa` | `0x005c4737` | 243 | - |
| `0x0fb` | 251 | `FRM_GETWORLDDIR` | `case 0xfb` | `0x005c47a0` | 244 | - |
| `0x0fc` | 252 | `FRM_GETWORLDPOS` | `case 0xfc` | `0x005c47f9` | 245 | - |
| `0x0fd` | 253 | `FRM_GETWORLDSCALE` | `case 0xfd` | `0x005c4866` | 246 | - |
| `0x0fe` | 254 | `FRM_GETWORLDROT` | `case 0xfe` | `0x005c48db` | 247 | - |
| `0x0ff` | 255 | `FRM_GETLOCALMATRIX` | `case 0xff` | `0x005c4956` | 248 | - |
| `0x100` | 256 | `FRM_GETWORLDMATRIX` | `case 0x100` | `0x005c49bc` | 249 | - |
| `0x101` | 257 | `VECT_SET` | `case 0x101` | `0x005c3d24` | 250 | - |
| `0x102` | 258 | `VECT_COPY` | `case 0x102` | `0x005c36ab` | 251 | - |
| `0x103` | 259 | `VECT_MUL_SCL` | `case 0x103` | `0x005c3d72` | 252 | - |
| `0x104` | 260 | `VECT_MUL_VECT` | `case 0x104` | `0x005c3715` | 253 | - |
| `0x105` | 261 | `VECT_MUL_QUAT` | `case 0x105` | `0x005c378a` | 254 | - |
| `0x106` | 262 | `VECT_MUL_MATRIX` | `case 0x106` | `0x005c37f0` | 255 | - |
| `0x107` | 263 | `VECT_ADD_VECT` | `case 0x107` | `0x005c3893` | 256 | - |
| `0x108` | 264 | `VECT_SUB_VECT` | `case 0x108` | `0x005c3908` | 257 | - |
| `0x109` | 265 | `VECT_CROSS` | `case 0x109` | `0x005c3981` | 258 | - |
| `0x10a` | 266 | `VECT_NORMALIZE` | `case 0x10a` | `0x005c3b66` | 259 | - |
| `0x10b` | 267 | `VECT_MAGNITUDE` | `case 0x10b` | `0x005c3a49` | 260 | - |
| `0x10c` | 268 | `VECT_INVERSE` | `case 0x10c` | `0x005c3cd0` | 261 | - |
| `0x10d` | 269 | `VECT_ANGLETO` | `case 0x10d` | `0x005c3ab8` | 262 | - |
| `0x10e` | 270 | `MATRIX_ZERO` | `case 0x10e` | `0x005c436b` | 263 | - |
| `0x10f` | 271 | `MATRIX_IDENTITY` | `case 0x10f` | `0x005c43a1` | 264 | - |
| `0x110` | 272 | `MATRIX_COPY` | `case 0x110` | `0x005c4012` | 265 | - |
| `0x111` | 273 | `MATRIX_INVERSE` | `case 0x111` | `0x005c43d8` | 266 | - |
| `0x112` | 274 | `MATRIX_MUL` | `case 0x112` | `0x005c406f` | 267 | - |
| `0x113` | 275 | `QUAT_COPY` | `case 0x113` | `0x005c40c0` | 268 | - |
| `0x114` | 276 | `QUAT_MAKE` | `case 0x114` | `0x005c448b` | 269 | - |
| `0x115` | 277 | `QUAT_SETDIR` | `case 0x115` | `0x005c44f3` | 270 | - |
| `0x116` | 278 | `QUAT_EXTRACT` | `case 0x116` | `0x005c455b` | 271 | - |
| `0x117` | 279 | `QUAT_INVERSE` | `case 0x117` | `0x005c440f` | 272 | - |
| `0x118` | 280 | `QUAT_GETROTMATRIX` | `case 0x118` | `0x005c411d` | 273 | - |
| `0x119` | 281 | `QUAT_ADD` | `case 0x119` | `0x005c417e` | 274 | - |
| `0x11a` | 282 | `QUAT_SUB` | `case 0x11a` | `0x005c41d8` | 275 | - |
| `0x11b` | 283 | `QUAT_MUL_SCL` | `case 0x11b` | `0x005c42db` | 276 | - |
| `0x11c` | 284 | `QUAT_MUL_QUAT` | `case 0x11c` | `0x005c4232` | 277 | - |
| `0x11d` | 285 | `QUAT_DOT` | `case 0x11d` | `0x005c45c3` | 278 | - |
| `0x11e` | 286 | `QUAT_NORMALIZE` | `case 0x11e` | `0x005c4454` | 279 | - |
| `0x11f` | 287 | `QUAT_ROTBYMATRIX` | `case 0x11f` | `0x005c4282` | 280 | - |
| `0x120` | 288 | `QUAT_SLERP` | `case 0x120` | `0x005c464a` | 281 | - |
| `0x121` | 289 | `COLL_TESTLINE` | `case 0x121` | `0x005c3e20` | 282 | - |
| `0x122` | 290 | `VLVP` | `case 0x122` | `0x005c3fa7` | 283 | - |
| `0x123` | 291 | `CTRL_READ` | `case 0x123` | `0x005c4a22` | 285 | - |
| `0x124` | 292 | `CTRL_READEX` | `case 0x124` | `0x005c4a76` | 286 | - |
| `0x125` | 293 | `GETFRAMEFROMACTOR` | `case 0x125` | `0x005c4ad0` | 287 | - |
| `0x126` | 294 | `ACTORUPDATEPLACEMENT` | `case 0x126` | `0x005c4b27` | 288 | - |
| `0x127` | 295 | `RACING_AUTOINVISIBLE` | `case 0x127` | `0x005c4bd4` | 289 | - |
| `0x128` | 296 | `SETPLAYERWFALLEVENT` | `case 0x128` | `0x005bf79c` | 112 | - |
| `0x129` | 297 | `CAR_INVISIBLE` | `case 0x129` | `0x005c4bef` | 290 | - |
| `0x12a` | 298 | `CAR_SWITCHSHOWENERGY` | `case 0x12a` | `0x005c4c6c` | 291 | - |
| `0x12b` | 299 | `DEBUG_GETFRAMEINFO` | `case 0x12b` | `0x005c76da` | 292 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x12c` | 300 | `POLICEITCHFORPLAYER` | `case 0x12c` | `0x005c4cc3` | 284 | - |
| `0x12d` | 301 | `NOANIMPRELOAD` | `case 0x12d` | `0x005c4d8d` | 293 | - |
| `0x12e` | 302 | `COMPAREACTORS` | `case 0x12e` | `0x005c4eb1` | 295 | - |
| `0x12f` | 303 | `SETNULLACTOR` | `case 0x12f` | `0x005c4e31` | 296 | - |
| `0x130` | 304 | `COMPAREFRAMES` | `case 0x130` | `0x005c4f00` | 297 | - |
| `0x131` | 305 | `SETNULLFRAME` | `case 0x131` | `0x005c4e61` | 298 | - |
| `0x132` | 306 | `SETFREERIDE` | `case 0x132` | `0x005c4e91` | 299 | - |
| `0x133` | 307 | `ENABLEMAP` | `case 0x133` | `0x005c4f75` | 300 | - |
| `0x134` | 308 | `FRAMEISTELEPHONE` | `case 0x134` | `0x005c4f9a` | 301 | - |
| `0x135` | 309 | `CAR_CALM` | `case 0x135` | `0x005c5008` | 302 | - |
| `0x136` | 310 | `BRIDGE_SHUTDOWN` | `case 0x136` | `0x005c5071` | 303 | - |
| `0x137` | 311 | `SETTRAFFSECTORSND` | `case 0x137` | `0x005c50cd` | 305 | - |
| `0x138` | 312 | `CITYMUSIC_OFF` | `case 0x138` | `0x005c5103` | 307 | - |
| `0x139` | 313 | `CITYMUSIC_ON` | `case 0x139` | `0x005c5107` | 306 | - |
| `0x13a` | 314 | `[DAN_DEBUG]` | `case 0x13a` | `0x005c76da` | 308 | no VM arm AND no `program_decode_params` arm - the argument decoder falls through its switch to `return 0`, so the command fails to COMPILE at script load |
| `0x13b` | 315 | `FRM_GETPARENT` | `case 0x13b` | `0x005c5160` | 309 | - |
| `0x13c` | 316 | `ISPOINTINSECTOR` | `case 0x13c` | `0x005c51c6` | 310 | - |
| `0x13d` | 317 | `ISPOINTINAREA` | `case 0x13d` | `0x005c524c` | 311 | - |
| `0x13e` | 318 | `GETACTIVECAMERA` | `case 0x13e` | `0x005c5120` | 312 | - |
| `0x13f` | 319 | `SOUND_GETVOLUME` | `case 0x13f` | `0x005c52f5` | 313 | - |
| `0x140` | 320 | `SOUND_SETVOLUME` | `case 0x140` | `0x005c5369` | 314 | - |
| `0x141` | 321 | `RACING_CHANGE_MODEL` | `case 0x141` | `0x005c5431` | 315 | - |
| `0x142` | 322 | `CITYCACHING_TICK` | `case 0x142` | `0x005c54fb` | 316 | - |
| `0x143` | 323 | `ACTORLIGHTNESSON` | `case 0x143` | `0x005c552c` | 317 | - |
| `0x144` | 324 | `CAR_LOCK` | `case 0x144` | `0x005c5545` | 318 | - |
| `0x145` | 325 | `CAR_LOCK_ALL` | `case 0x145` | `0x005c55c8` | 319 | - |
| `0x146` | 326 | `PM_SHOWSYMBOL` | `case 0x146` | `0x005c57a7` | 320 | - |
| `0x147` | 327 | `PM_SHOWPROGRESS` | `case 0x147` | `0x005c5925` | 321 | - |
| `0x148` | 328 | `PM_SETPROGRESS` | `case 0x148` | `0x005c596b` | 322 | - |
| `0x149` | 329 | `SETFILMMUSIC` | `case 0x149` | `0x005c564c` | 323 | - |
| `0x14a` | 330 | `GETFILMMUSIC` | `case 0x14a` | `0x005c5692` | 324 | - |
| `0x14b` | 331 | `HUMAN_GETSEATIDX` | `case 0x14b` | `0x005c56e0` | 325 | - |
| `0x14c` | 332 | `HUMAN_ENTERTOTRUCK` | `case 0x14c` | `0x005c5a11` | 326 | - |
| `0x14d` | 333 | `FLOATREG_PUSH` | `case 0x14d` | `0x005c5ae8` | 327 | - |
| `0x14e` | 334 | `FLOATREG_POP` | `case 0x14e` | `0x005c5b27` | 328 | - |
| `0x14f` | 335 | `AIRPLANESHOWDAMAGE` | `case 0x14f` | `0x005c5b66` | 329 | - |
| `0x150` | 336 | `SETNPCKILLEVENT` | `case 0x150` | `0x005c5bc2` | 330 | - |
| `0x151` | 337 | `CAR_GETSEATCOUNT` | `case 0x151` | `0x005c5d5c` | 333 | - |
| `0x152` | 338 | `FRM_SETALPHA` | `case 0x152` | `0x005c5dd9` | 334 | - |
| `0x153` | 339 | `HUMAN_THROWGRENADE` | `case 0x153` | `0x005c5ee0` | 335 | - |
| `0x154` | 340 | `DETECTOR_CREATEDYNCOLL` | `case 0x154` | `0x005c76da` | 336 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x155` | 341 | `DETECTOR_ERASEDYNCOLL` | `case 0x155` | `0x005c76da` | 337 | no arm in the BASE dispatcher - implemented by `detector_program_process`, the `C_detector` override of `C_program::Process` |
| `0x156` | 342 | `CAR_UNBREAKABLE` | `case 0x156` | `0x005c60fc` | 338 | - |
| `0x157` | 343 | `PHOBJ_IMPULS` | `case 0x157` | `0x005c615d` | 339 | - |
| `0x158` | 344 | `WEATHER_PREPAREBUFFER` | `case 0x158` | `0x005c622b` | 340 | - |
| `0x159` | 345 | `SUBTITLE` | `case 0x159` | `0x005c6253` | 341 | - |
| `0x15a` | 346 | `HUMAN_CANDIE` | `case 0x15a` | `0x005c76da` | 342 | compiles (it has a `program_decode_params` arm) but no dispatcher, base or derived, executes it - silently ignored at run time |
| `0x15b` | 347 | `VERSION_IS_GERMANY` | `case 0x15b` | `0x005c6262` | 343 | - |
| `0x15c` | 348 | `VERSION_IS_EDITOR` | `case 0x15c` | `0x005c6262` | 344 | - |
| `0x15d` | 349 | `GARAGE_NEZAHAZUJ` | `case 0x15d` | `0x005c6292` | 345 | - |
| `0x15e` | 350 | `POLICE_SUPPORT` | `case 0x15e` | `0x005c64a6` | 348 | - |
| `0x15f` | 351 | `PLAY_AVI_INTRO` | `case 0x15f` | `0x005c652d` | 349 | - |
| `0x160` | 352 | `CAMERA_SETFOV` | `case 0x160` | `0x005c6637` | 350 | - |
| `0x161` | 353 | `CAMERA_GETFOV` | `case 0x161` | `0x005c65ea` | 351 | - |
| `0x162` | 354 | `DOOR_GETSTATE` | `case 0x162` | `0x005c66b2` | 352 | - |
| `0x163` | 355 | `CAR_GETMAXLEVELS` | `case 0x163` | `0x005c672c` | 353 | - |
| `0x164` | 356 | `CAR_GETACTLEVEL` | `case 0x164` | `0x005c67a6` | 354 | - |
| `0x165` | 357 | `CAR_SETACTLEVEL` | `case 0x165` | `0x005c6820` | 355 | - |
| `0x166` | 358 | `FREERIDE_SCOREON` | `case 0x166` | `0x005c68a8` | 356 | - |
| `0x167` | 359 | `FREERIDE_SCORESET` | `case 0x167` | `0x005c6915` | 357 | - |
| `0x168` | 360 | `FREERIDE_SCOREGET` | `case 0x168` | `0x005c68d5` | 358 | - |
| `0x169` | 361 | `FREERIDE_SCOREADD` | `case 0x169` | `0x005c695e` | 359 | - |
| `0x16a` | 362 | `TAXIDRIVER_ENABLE` | `case 0x16a` | `0x005c69d0` | 360 | - |
| `0x16b` | 363 | `HUMAN_RETURNTOTRAFF` | `case 0x16b` | `0x005c69ef` | 361 | - |
| `0x16c` | 364 | `TIMER_GETINTERVAL` | `case 0x16c` | `0x005c6af1` | 362 | - |
| `0x16d` | 365 | `TIMER_SETINTERVAL` | `case 0x16d` | `0x005c6b5f` | 363 | - |
| `0x16e` | 366 | `FRM_GETNUMCHILDREN` | `case 0x16e` | `0x005c6c14` | 365 | - |
| `0x16f` | 367 | `FRM_GETCHILD` | `case 0x16f` | `0x005c6c72` | 366 | - |
| `0x170` | 368 | `HUMAN_WAITTOREADY` | `case 0x170` | `0x005c6baf` | 364 | - |
| `0x171` | 369 | `ACTOR_ADDDORICI` | `case 0x171` | `0x005c6d01` | 367 | - |
| `0x172` | 370 | `CAR_REPAIR` | `case 0x172` | `0x005c6d4f` | 368 | - |
| `0x173` | 371 | `POLICE_SPEED_FACTOR` | `case 0x173` | `0x005c6d9e` | 369 | - |
| `0x174` | 372 | `CARTRIDGE_INVALIDATE` | `case 0x174` | `0x005c6e2f` | 371 | - |
| `0x175` | 373 | `NPC_SHUTUP` | `case 0x175` | `0x005c6e46` | 372 | - |
| `0x176` | 374 | `FREERIDE_ENABLECAR` | `case 0x176` | `0x005c6e65` | 373 | - |
| `0x177` | 375 | `GETCARLINENUMFROMTABLE` | `case 0x177` | `0x005c6e85` | 374 | - |
| `0x178` | 376 | `ENEMY_GETHOSTILESTATE` | `case 0x178` | `0x005c6f12` | 375 | - |
| `0x179` | 377 | `HUMAN_SHUTUP` | `case 0x179` | `0x005c6fb6` | 376 | - |
| `0x17a` | 378 | `PUMPER_CANWORK` | `case 0x17a` | `0x005c7213` | 377 | - |
| `0x17b` | 379 | `GROUP_DISBAND` | `case 0x17b` | `0x005c7024` | 378 | - |
| `0x17c` | 380 | `INVENTORY_CLEAR` | `case 0x17c` | `0x005c72cc` | 380 | - |
| `0x17d` | 381 | `CAR_DISABLE_UO` | `case 0x17d` | `0x005c7254` | 379 | - |
| `0x17e` | 382 | `GETLASTSAVENUM` | `case 0x17e` | `0x005c733a` | 381 | - |
| `0x17f` | 383 | `FREERIDE_TRAFFSETUP` | `case 0x17f` | `0x005c737a` | 382 | - |
| `0x180` | 384 | `POCZKURVENYCHBEDENCAR` | `case 0x180` | `0x005c7389` | 383 | - |
| `0x181` | 385 | `HUMAN_GETACTANIMID` | `case 0x181` | `0x005c7411` | 384 | - |
| `0x182` | 386 | `CAR_SETAXIS` | `case 0x182` | `0x005c748e` | 385 | - |
| `0x183` | 387 | `M7_ZASTAVZKURVENEPOLISE` | `case 0x183` | `0x005c768b` | 389 | - |
| `0x184` | 388 | `ISVALIDTAXIPASSENGER` | `case 0x184` | `0x005c7697` | 390 | - |
| `0x185` | 389 | `DAN_INTERNAL_1` | `case 0x185` | `0x005c62e5` | 346 | - |
| `0x186` | 390 | `DAN_INTERNAL_2` | `case 0x186` | `0x005c642a` | 347 | - |

## PC vs PS2

385 of the 391 names appear verbatim in the PS2 `.rodata` copy of the same table; the 6 that do not are exactly the PC-only `REC*` recorder commands (`structs/recorder.md`).

## Open items

Nothing is open.

