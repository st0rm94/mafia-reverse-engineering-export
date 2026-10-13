# tables\predmety.def

**Created** 2026-05-23 10:46 +0200 · **Modified** 2026-10-04 23:41 +0200

## Trivia

- Not every object is usable as a gun.
- Guns and throwables have limited ammo.
- Not all objects are used in the game.

## Item table

| ID | String | Melee | Gun | Throwable |
|---:|---|:---:|:---:|:---:|
| 0 | Empty Hands | ✓ | - | - |
| 1 | Special action | - | - | - |
| 2 | Knuckle-duster | ✓ | - | - |
| 3 | Knife | ✓ | - | - |
| 4 | Baseball bat | ✓ | - | - |
| 5 | Molotov cocktail | - | - | ✓ |
| 6 | Colt Detective Special | - | ✓ | - |
| 7 | S&W model 27 Magnum | - | ✓ | - |
| 8 | S&W model 10 M&P | - | ✓ | - |
| 9 | Colt 1911 | - | ✓ | - |
| 10 | Thompson 1928 | - | ✓ | - |
| 11 | Pump-action shotgun | - | ✓ | - |
| 12 | Sawed-off shotgun | - | ✓ | - |
| 13 | US rifle, M1903 Springfield | - | ✓ | - |
| 14 | Mosin-Nagant 1891/30 | - | ✓ | - |
| 15 | Grenade | - | - | ✓ |
| 16 | Key | - | - | - |
| 17 | Bucket | - | - | - |
| 18 | Flashlight | - | - | - |
| 19 | Documents | - | - | - |
| 20 | Steel bar | ✓ | - | - |
| 21 | Papers | - | - | - |
| 22 | Explosives ¹ | - | - | - |
| 23 | Door key | - | - | - |
| 24 | Safe key | - | - | - |
| 25 | Crowbar | ✓ | - | - |
| 26 | Tickets | - | - | - |
| 27 | Package | - | - | - |
| 28 | Wooden plank | ✓ | - | - |
| 29 | Bottle | - | - | - |
| 30 | Small key | - | - | - |
| 31 | Sword | ✓ | - | - |
| 32 | Dogs head | ✓ | - | - |
| 33 | Thompson 1928 with no sound ² | - | ✓ | - |
| 34 | Pump-action shotgun with no sound ² | - | ✓ | - |

¹ Cannot be equipped and is not throwable.
² Not used in the game.

## Flag word

The file is 35 records of `0xbc` bytes, one per ID above, and every record carries a 32-bit flag word
at `+0x20`. The item lists below are read from the shipped file; the effects are the code that tests
each bit.

| bit | items (ID) | effect |
|---|---|---|
| `0x01` | 1 | no reader located |
| `0x02` | 11, 12, 34 (the shotguns) | `human_ai` passes it as a flag argument of `game_new_cartridge`; it exempts the attacker's hit from the strong-hit rule of `human_hit` (damage above 80 with a hit type below 5) |
| `0x04` | 2, 3, 4, 20, 25, 28, 29, 31 (the melee weapons) | `entity_weapon_choose` asks for it; part of the any-weapon mask `0x24` |
| `0x20` | 5..15, 33, 34 (the firearms and the two throwables) | the ranged-weapon test of `ai_logs_tick`, the script predicates and `entity_weapon_choose`; part of `0x24` |
| `0x80` | 5, 15 (the throwables) | no reader located |
| `0x100` | 16, 19, 21, 22, 23, 24, 26, 27, 30 (keys and papers) | the inventory screen shows the row but disables it; `inventory_insert_new_item` tests it |
| `0x2000` | 4, 5, 15, 20, 25, 28, 31 | the gun shop shows no ammo counts for the row; `inventory_insert_new_item` puts the item straight into the hand |
| `0x4000` | 14 (the scoped Mosin-Nagant) | read by `player_set_sniper_mode`, `player_ai_walk` and `human_change_weapon_model`, the sniper-mode test |
| `0x40000` | 12 (the sawed-off shotgun) | the item may go into the big-weapon slot even without the carry permission |
| `0x200000` | 4, 10..14, 17, 20, 28, 31, 33, 34 (the big items) | routed to the single big-weapon slot ("Coat"), and only while the owner may carry one |
| `0x800000` | 2, 3, 5, 12, 14..34 | left out of the gun shop's catalogue |
| `0x2000000` | 6..10, 12..14, 17, 33 | `inventory_reload_magazine` takes its reserve-only path |

The two bits together, mask `0x24` (melee or ranged), are the "any weapon" test that the weapon cycle
and the hand slot use.
