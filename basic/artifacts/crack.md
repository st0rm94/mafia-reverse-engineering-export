# The Mafia PC executables — SafeDisc, the two cracks, and what is authentic

**Created** 2026-08-20 12:18 +0200 · **Modified** 2026-09-18 05:38 +0200

Read this before any byte-level claim about `Game.exe`, before any "the original binary does X"
statement, and before diffing the PC exes against each other.

> **The one-sentence version.** Everything this project decompiles comes from a **cracked, decrypted
> memory dump** of a **SafeDisc 2.70.030**-protected retail executable; the retail file itself is in
> the repo but its `.text` and `.data` are encrypted, so it can *authenticate* our copy's headers
> and `.rdata`/`.rsrc` but cannot supply a single byte of game code.

---

## 1. The five executables — always say which one

| file | size | MD5 | what it is |
|---|---|---|---|
| `Game.exe` | 2,486,272 | `92419435416499862cc981199566c14b` | **The project's subject.** A cracked no-CD build. Tag `GiMP` in the PE header |
| `Game-racing-mod.exe` | 2,486,272 | `3996f33e9d4becc48715f86921bbcb8d` | The community racing-menu mod. A *different* crack of the same dump. Tag `fLT!` (FairLight) |
| `Game-uncracked.exe` | 3,241,174 | `3a69149fae08c5689e6802f3e6f35b68` | **Genuine retail 1.0**, SafeDisc intact (§3) |
| `Game-uncracked-code.exe` | 2,486,272 | `009e1fcc05a9d58120099cee611c4748` | `Game.exe` with the four cracked code bytes restored and nothing else touched (§5c) |
| `game-sources/Game-norng.exe` | 2,486,272 | `3771ff05bebf9c8f9cd8b62eb3b07181` | FairLight's build plus one gameplay patch that kills the RNG (§7d) |

SHA-256:
`Game.exe` = `2641429b2bd855231f6d41de80736791885ae72d7aac8e8a8e5f33dd79be18df` ·
`Game-racing-mod.exe` = `1275d12a07940165479864307cd7b988f89d134de07c48e9eb35cd8c161a214f` ·
`Game-uncracked.exe` = `505985d1258cda59acaeb02528b7908e02429e0fed1712409be99ac9a4419d02` ·
`Game-uncracked-code.exe` = `d1694468d7b5903ea98deb7c095caebe94117e17d124a45b28900f696832e7ce` ·
`Game-norng.exe` = `56f13cea96cdedd2cd1dc3d365cfaf9856f7cee96505d124397b06605abf7801`

All of them are the **same build**: identical `.rsrc` (byte-for-byte), identical DOS stub +
Rich header, identical section names / VAs / virtual sizes, and a version resource reading
`Illusion Softworks` / `MAFIA` / FileVersion **`Master 1.0`** / OriginalFilename `Game.exe` /
`Copyright © 2001`.

---

## 2. The headline findings

1. **The retail executable is SafeDisc 2.70.030 protected** and its entry point is inside the
   protection stub, not in `.text`.
2. **Retail `.text` and `.data` are encrypted.** Our `Game.exe` is the decrypted image, dumped from
   memory at the original entry point. That is the only reason a decompilation exists at all.
3. **`Game.exe` is cracked in exactly ONE place in game code** — 4 bytes at VA `0x006240e0`, the
   CD check (§5c, §8).
4. **Every cracked PC exe carries a cracker banner in the PE header padding.** The `GiMP` banner and
   the FairLight one occupy *different* byte ranges, which is why a two-way diff makes each file look
   clean exactly where the other is dirty.
5. **`Where is the weed?` is a cracker artifact, not a developer easter egg** (§6).
6. The **import table in the cracked exes is a reconstruction**, and they share it byte-for-byte —
   so for `.text` they are **one witness, not two** (§7, §9).

---

## 3. The retail executable: SafeDisc 2.70.030

*Artifact: `Game-uncracked.exe` bytes.*

| evidence | value |
|---|---|
| SafeDisc signature @ file `0xfd4` | `BoG_ *90.0&!!  Yy>` |
| version dwords @ `0xff4` / `0xff8` / `0xffc` | `2` / `70` / `30` → **SafeDisc 2.70.030** |
| stub sections | `stxt774` @ VA `0x006fc000` (vsz `0x204f`) and `stxt371` @ VA `0x006ff000` (vsz `0x325c`), both characteristics `0xE0000020` = **code, read, write, execute** ¹ |
| PE `AddressOfEntryPoint` | RVA `0x002ff05e` → **VA `0x006ff05e`, inside `stxt371`** |
| PE `TimeDateStamp` | `0x3d51286b` = **2002-08-07 14:02:19 UTC** — the real build stamp |
| PE `CheckSum` | `0` (as linked) |
| DataDirectory\[1] Import | RVA `0x00301f70`, **inside `stxt371`** — SafeDisc's own 13-slot table |
| DataDirectory\[12] IAT | RVA `0x003020fa`, size `0x58` |
| overlay | **754,902 bytes** appended past the last section, entropy **7.77** — the encrypted payload |
| header padding `0x2f0..0xfd3` | **all zero.** No banner, no tag, no message |

¹ All 50 functions in `stxt371` are named and documented in `structs/safedisc_stub.md`.
`stxt774` holds no code at all: it is the encrypted payload.

The entry VA `0x006ff05e` is predictable from the stub alone: `stxt371` carries a plaintext
`entryp` marker followed by `55 8b ec 60 bb 5e f0 6f 00` = `push ebp; mov ebp,esp; pushad;
mov ebx, 0x006ff05e`. The retail PE points exactly there.

### 3a. What SafeDisc encrypts

Retail vs our `Game.exe`, section by section:

| section | differing bytes | entropy retail / ours | verdict |
|---|---|---|---|
| `.text` | **2,323,797 of 2,326,528** | **7.98** / 6.52 | **fully encrypted** — the only agreeing run is the 2,660 zero bytes past `vsize` |
| `.rdata` | 2,372 of 45,056 | 5.02 / 4.87 | **plaintext**, except the import machinery |
| `.data` | 65,320 of 65,536 | **7.07** / 4.73 | **encrypted** |
| `.rsrc` | **0** | - | untouched |
| `stxt774` | **0** | - | the stub itself |
| `stxt371` | 36 | - | six case-only edits (§5e) |

⇒ **The retail file cannot supply or verify any game-code byte.** For the missing prologue at
`0x006240e0` the *only* source is `Game-racing-mod.exe` (§8).

### 3b. What SafeDisc did to the imports

The game's real import descriptor array at **VA `0x00644fa8`** (file `0x244fa8`, `0xc8` bytes) is
**byte-identical in retail and in both cracked exes**. SafeDisc did *not* rebuild it — it left the
descriptors in place and merely repointed `DataDirectory[1]` at its own table in `stxt371`.

What SafeDisc *did* destroy is everything the descriptors point at:

| region | retail | ours |
|---|---|---|
| IAT `0x0063b000..0x0063b223` | encrypted | 140 resolved slots |
| INT mirrors `0x00645070` / `0x006450a0` / `0x00645274` | encrypted | rebuilt |
| hint/name blobs `0x006452c4..0x00645515`, `0x00645ae0..0x00645f3e` | encrypted (high-entropy) | plaintext `lstrcpyA`, `GetLocalTime`, `CreateDirectoryA`, … |

SafeDisc's own 13-slot table names the same nine DLLs in the developers' original casing —
`KERNEL32.dll`, `USER32.dll`, `ADVAPI32.dll`, `WINMM.dll`, **`LS3DF.dll`, `IJoy.dll`, `Inet.dll`,
`rw_data.dll`**, `IMAGEHLP.dll` — independently corroborating the four Illusion DLLs in
`basic/artifacts/dll.md`.

---

## 4. What a crack had to do — and why `Game.exe` is a memory dump

To make a standalone no-CD executable from the retail file, both groups had to:

1. run the retail exe so the SafeDisc stub decrypts `.text` and `.data` in memory;
2. dump the image at the **original entry point** (OEP);
3. set `AddressOfEntryPoint` back to the OEP — in both cracked exes it is RVA `0x00226f1a` →
   **VA `0x00626f1a` = `crt_startup`**, so the stub is orphaned and nothing enters it;
4. rebuild `DataDirectory[1]` (→ `0x00244fa8`) and `DataDirectory[12]` (→ IAT `0x0023b000`,
   size `0x254`) and reconstruct the IAT / INT / API name strings;
5. neutralise the in-game CD check (§5c and §7).

🥇 **Step 4's four values are corroborated from the other end.** They are not an inference:
SafeDisc keeps its own saved copy of them at `0x006ff009`
(`g_safedisc_saved_data_directories`), and the bytes read
`a8 4f 24 00 c8 00 00 00 00 b0 23 00 54 02 00 00` = Import RVA `0x00244fa8` size `0xc8`,
IAT RVA `0x0023b000` size `0x254` — byte for byte what §3b derives from the header diff. The
stub function `safedisc_restore_import_directory` writes them back at run time; the cracker
simply persisted the post-restore state. Two artifacts, two methods, one answer
(`structs/safedisc_stub.md` §9).

**Consequence worth remembering:** our `.data` is a *snapshot*, not the linked initial image. The OEP
dump makes it very likely to equal the file-initial values (nothing of the CRT has run yet at
`crt_startup`), but that is an inference, not a measurement, and the retail file cannot check it
because `.data` is encrypted. Any claim about a global's **initial** value inherits this caveat.

---

## 5. `Game.exe` — complete inventory of modifications

Ten sites. Only #8 changes what the game does.

| # | site | size | classification |
|---|---|---|---|
| 1 | PE `TimeDateStamp` @ file `0x110` | 4 B | `47 69 4d 50` = **`GiMP`**. Original recoverable: **`0x3d51286b`** |
| 2 | PE `CheckSum` @ file `0x160` | 4 B | `0x002673b9` — **recomputed and valid** for the cracked bytes. Retail value is `0` |
| 3 | ASCII banner @ `0x320..0x4d0` | 433 B | §5b. Retail is all-zero |
| 4 | `Where is the weed?` @ `0x501..0x512` | 18 B | §5b / §6. Retail is all-zero |
| 5 | `DataDirectory[1]` + `[12]` | 16 B | the un-wrapping (§4 step 4), not a crack |
| 6 | IAT / INT / API name strings | ~2.3 KB | **reconstruction** — retail is encrypted (§3b) |
| 7 | `stxt371` DLL-name casing, 6 sites | 36 B | §5e — dead code |
| 8 | **`.text` @ VA `0x006240e0`** | **4 B** | **the CD-check crack** (§5c, §8) |
| 9 | `.text` @ VA `0x00629aaa` | 2 B | reconstructed IAT operand. **Ours is the correct one** (§7) |
| 10 | `.text` @ VA `0x0040bc9d`, `0x0040c5bd` | 2 B | **unexplained — deliberately unattributed** (§5f) |

### 5b. The banner

Header padding `0x320..0x4d0`, CRLF-terminated lines, verbatim:

```
   ________.__   _____ __________  ___________________       _________
  /  _____/|__| /     \______   \/   _____/\______   \__ __/   _____/
 /   \  ___|  |/  \ /  \|     ___/\_____  \  |       _/  |  \_____  \
 \    \_\  \  /    Y    \    |    /        \ |    |   \  |  /        \
  \______  /__\____|__  /____|   /_______  / |____|_  /____/_______  /
         \/           \/                 \/         \/
```

The blob is **433 bytes**, `0x320..0x4d0`. It opens with a blank `\r\n\r\n` and every art line is
CRLF-terminated; line lengths in bytes are `0, 0, 70, 70, 70, 70, 70, 67, 0` (the last art line
carries 13 trailing spaces before its CRLF). The last byte of the blob is the `0a` at `0x4d0`.

The **measured** identity marker is the PE `TimeDateStamp`, which literally spells **`GiMP`**. The
figlet-style letterforms segment as `G i M P S R … S`; the exact release-group name is **not settled
by measurement** and is deliberately not asserted here.

#### `Where is the weed?` — the exact bytes

Widely believed to be a hidden developer message; §6 shows it is not. Its exact form:

```
  04f0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |................|
  0500  00 57 68 65 72 65 20 69 73 20 74 68 65 20 77 65  |.Where is the we|
  0510  65 64 3f 00 00 00 00 00 00 00 00 00 00 00 00 00  |ed?.............|
```

| property | value |
|---|---|
| file offset | `0x501..0x512`, **18 bytes** — maps to VA `0x00400501` in the header page |
| bytes | `57 68 65 72 65 20 69 73 20 74 68 65 20 77 65 65 64 3f` |
| exact text | `Where is the weed?` — capital `W`, rest lowercase, single spaces, ends `?` |
| terminators of its own | **none** — no `CR`, no `LF`, no embedded `NUL`, no quotes |
| byte before / after | `0x00` / `0x00` |
| references | **zero**, anywhere in the image |

**It is a separate block from the banner, not the banner's last line.** The banner is CRLF-formatted
text ending at `0x4d0`; then **48 zero bytes** (`0x4d1..0x500`); then this bare, NUL-delimited run
with no line endings at all; then zeros unbroken to `0xfd4`, where the SafeDisc marker sits. Two
oddities are recorded as measured and **not** attributed: that 0x30-byte gap, and the fact that the
string begins at the **odd** offset `0x501` rather than `0x500` (a 20-byte write of
`\0Where is the weed?\0` at `0x500` would produce exactly this, but that is a guess).

In `Game-uncracked.exe` the whole range `0x2f0..0xfd3` is zero — no banner, no string.

`Game-racing-mod.exe` carries its own banner at `0x2f0..0x34f` + `0x367..0x3af` reading
`FAiRLiGHT!`, plus `sb` at `0x106` and `TimeDateStamp` = `fLT!`. Because the two banners occupy
**disjoint** ranges, a two-way diff between the two cracked files shows each of them "clean"
exactly where the other is dirty — a trap for any header claim made from that diff alone.

### 5c. The `.text` crack — 4 bytes, not 6

| VA | retail-equivalent (from the mod) | ours |
|---|---|---|
| `0x006240e0` | `81 EC 8C 01` = `sub esp,0x18c` | `33 C0 40 C3` = `xor eax,eax; inc eax; ret` |

The following `00 00` — the high half of the `0x18c` immediate — is byte-identical in both files and
was never overwritten, which is why exactly four bytes had to change to turn a 171-byte function
into `return 1`.

#### 🔴 THE REGISTER OF DECOMP↔`Game.exe` DIVERGENCES — one entry, and it must stay one

The annotated decomp describes the **original** function at this VA, not the cracked bytes. This is
the *only* place in the decomp where that is true.

| VA | decomp says | `Game.exe` says | source of the original |
|---|---|---|---|
| `0x006240e0` | `sub esp,0x18c` (`cd_scan_drives_for_disc`) | `33 C0 40 C3` | `Game-racing-mod.exe`, copied verbatim |

The register is enforced by a **gate, not a note**: a check that fails if the list grows, if
`Game.exe` stops carrying the cracked bytes (which would make the exception stale), if
`Game-uncracked-code.exe` is missing or differs from `Game.exe` by anything other than those four
bytes, or if the decomp gains or loses a definition at that VA. It was verified by injecting a
fault, and it belongs after any Ghidra re-export and whenever `Game.exe` itself is replaced.

**`Game-uncracked-code.exe`** is `Game.exe` with exactly those four bytes restored and **nothing
else touched** — not the `TimeDateStamp`, not the `CheckSum`, not the banner. It matches the decomp
at this VA. `Game.exe` itself is never overwritten, because the analysis tooling reads it by that
exact path.

### 5e. The `stxt371` case edits

Six runs, 36 bytes, every one a letter-case change in a DLL-name string inside the orphaned stub:

| VA | retail | ours |
|---|---|---|
| `0x00702038` | `KERNEL` | `kernel` |
| `0x0070205e` | `USER` | `user` |
| `0x00702072` | `ADVAPI` | `advapi` |
| `0x00702088` | `WINMM.dll` … `LS3DF.dll` … `IJ…` | `winmm.dll` … `ls3df.dll` … `ij…` |
| `0x007020be` | `I` | `i` |
| `0x007020e4` | `IMAGEHLP` | `imagehlp` |

**Leading hypothesis, unverified:** `stxt371` is a **writable** section (`0xE0000020`), so the
SafeDisc stub could plausibly lowercase its own DLL-name table in place before `LoadLibrary`, and
the dump captured the post-run state. That would make these edits an artifact of *dumping*, not of
*patching*. Settling it would need the retail exe run under a debugger. The `.rdata` name strings
are lowercased too (`IMAGEHLP.dll` → `imagehlp.dll`, while the unused `MPR.dll` string keeps its
case), but `.rdata` is read-only, so those must have been written into the file by the import
rebuilder. Two mechanisms, one appearance.

### 5f. The two unexplained bytes — measured, not attributed

| VA | context | retail | ours | mod |
|---|---|---|---|---|
| `0x0040bc9d` | byte\[2] of the 5-byte run `0x0040bc9b..0x0040bc9f`, inter-function padding after `ret 0x14`, before a 16-aligned start | **encrypted — unreadable** | `a1` | `0b` |
| `0x0040c5bd` | byte\[2] of the 5-byte run `0x0040c5bb..0x0040c5bf`, same shape after `ret 0xc` | **encrypted — unreadable** | `a0` | `0a` |

Both runs are exactly 5 bytes, both differ in exactly byte\[2], both deltas are `−0x96`, and within
each file the two values are consecutive (`a1`/`a0`, `0b`/`0a`). **Zero absolute references anywhere
in the image.** Attributed to nothing — not to the crack, not to racing. The retail file **cannot**
settle them because `.text` is encrypted.

---

## 6. `Where is the weed?` — a cracker artifact, not a developer easter egg

The string is widely believed in the community to be a hidden message from Illusion Softworks. It is
not, and the measurement is unambiguous.

| measurement | result |
|---|---|
| occurrences in `Game.exe` | **exactly one**, at file `0x501`, in PE header padding — **not inside any section** |
| occurrences in `.text` / `.rdata` / `.data` / `.rsrc` of our exe | **zero** |
| occurrences in `Game-uncracked.exe` (retail) | **zero** (whole file, incl. plaintext `.rdata`/`.rsrc` and the 754 KB overlay) |
| occurrences in `Game-racing-mod.exe` | **zero** |
| occurrences in `Game-norng.exe` | **zero** |
| occurrences in all 12 `.dta` archives (`A0`–`AC`, **1.79 GB of shipped game data**) | **zero** |
| occurrences in the localized string tables under `def-files-docs/` | **zero** |
| UTF-16LE occurrences, any file | **zero** |
| retail header padding `0x2f0..0xfd3` | **all zero** |

A case-insensitive search for the bare word `weed` does hit `A0.dta` once and `A6.dta` six times, but
every hit is binary noise inside compressed data (`;;FFWWeeddOO`, `YUWEEDX`, `_tFFweeD`) — no hit is a
readable string, and the **full phrase occurs nowhere in 1.79 GB of game data**.

The decisive step is the comparison across builds. Our exe, FairLight's and the no-RNG build are **the
same decrypted image** — `.rsrc` identical, `.rdata` identical, `.text` differing in 13 bytes total —
so any string the developers had put in the program would appear in all of them. Only one has it, and
only in the header padding the other two leave zeroed. And the retail file, which predates every
crack, has zeros there.

**Why the belief is nonetheless reasonable:** the `GiMP`-tagged no-CD executable was the one almost
everybody actually ran for a decade. A string sitting in *your* `Game.exe`, in a slot no compiler
would use, reads exactly like a hidden developer joke.

**The nearest genuine thing** is unrelated dialogue: `textdb` string id **950034**,
`"You wanna leave me here in the weeds?"` — a real line, in the localized string table, in all four
shipped languages.

**Scope of the claim:** this is a statement about the **five executables and twelve archives listed
above**. A different Mafia build — the 1.1 patch executable, a demo, another territory release — has
not been examined. If someone can point at the string in a file whose provenance is retail, that is
new evidence and this section should be revisited.

---

## 7. `Game-racing-mod.exe` — the FairLight build

Differs from our `Game.exe` in **511 bytes over 19 contiguous runs**. 498 are header/banner; **13 are
in mapped sections, at 7 addresses**:

| VA | ours | mod | reading | which is original |
|---|---|---|---|---|
| `0x0040bc9d` | `a1` | `0b` | dead padding | **unknown** (§5f) |
| `0x0040c5bd` | `a0` | `0a` | dead padding | **unknown** (§5f) |
| `0x0057699d..e` | `9b 1b` | `cd 1c` | `push 0xf1b9b` → `push 0xf1ccd` (menu id) | **ours** — mod = the racing edit |
| `0x005769b7..8` | `9b 1b` | `cd 1c` | same, second call | **ours** |
| `0x0062407e` | `75` | `eb` | `jne` → `jmp` inside `cd_require_disc` | **ours** — mod = FairLight's crack |
| `0x006240e0..e3` | `33 c0 40 c3` | `81 ec 8c 01` | the stub vs the real prologue | **mod** — ours = the `GiMP` crack |
| `0x00629aaa..ab` | `[0x63b100]` | `[0x63b06c]` | `LCMapStringW` vs `CompareStringW` | **ours** — see §7a |

### 7a. `0x00629aaa` — a reconstruction error in the mod

The containing function sizes an A→W conversion with `MultiByteToWideChar`, converts, calls the W
API, then converts the result back with `WideCharToMultiByte(cp, 0x220 = WC_COMPOSITECHECK|WC_SEPCHARS, …)`.
That trailing back-conversion only exists if the W call **produced a string**, so the function is
MSVC's `__crtLCMapStringA` and the correct import is **`LCMapStringW`**. Since the API name table was
encrypted in retail (§3b), which slot holds which API was a *tool decision* during the rebuild — the
mod's rebuild got this one wrong. Not a deliberate patch on either side.

### 7b. The mod's checksum is broken

| file | stored `CheckSum` | recomputed | verdict |
|---|---|---|---|
| `Game.exe` | `0x002673b9` | `0x002673b9` | **match** |
| `Game-racing-mod.exe` | `0x0031b961` | `0x0026a440` | **mismatch** |

### 7c. The racing edit itself

Two `push` operands, redirecting `gui_main_menu_ctor`'s disable/delete calls from menu id `0xf1b9b`
to the harmless `0xf1ccd`, plus a `menu.def` carrying one extra `meti` record. Full write-up:
`basic/artifacts/racing.md` §§3–4.

### 7d. `Game-norng.exe` — FairLight plus one gameplay patch

`game-sources/Game-norng.exe` is **not** an independent build. It carries FairLight's
`TimeDateStamp` (`fLT!`), FairLight's banner, FairLight's `jne`→`jmp` at `0x0062407e`, FairLight's
intact `81 ec 8c 01` at `0x006240e0`, FairLight's `CompareStringW` operand at `0x00629aaa` and
FairLight's `0b` / `0a` padding bytes. It differs from our `Game.exe` in 513 bytes over 11 runs —
FairLight's 511, plus **one 6-byte patch of its own**:

| VA | ours | no-RNG build |
|---|---|---|
| `0x0044de65` | `8b c2 c3 90` + the first 2 bytes of the following `e8 09 52 fb ff` | `b8 00 00 00 00 c3` = `mov eax,0; ret` |

That is the tail of **`rand_range`** (`0x0044de50`), whose normal ending is
`call rand; cdq; idiv esi; pop esi; mov eax,edx; ret` — i.e. `rand() % range`. The patch makes it
return **0** unconditionally. The overwrite also clobbers two bytes of the `e8 09 52 fb ff` sequence
sitting in the gap before `rand_float` (`0x0044de70`); no function start is recorded there and the
build is in community use, so those bytes are presumed dead — **not verified**.

⚠️ **Do not byte-diff against this file to reason about retail.** It is two crack layers plus a
deliberate gameplay change.

---

## 8. What the stubbed function actually is

*Artifact: `Game-racing-mod.exe` for `[0x006240e0, 0x006240e6)`, `Game.exe` for the rest — verified
byte-identical across `[0x006240e6, 0x00624190)`.*

Extent `[0x006240e0, 0x0062418b)` = **171 bytes**, then five `nop` to the next 16-aligned start. Both
epilogues are `add esp,0x18c`, matching the recovered prologue.

For each drive letter `'C'` (`0x43`) … `'Z'` (`0x5a`):

1. copy `"X:\"` (`DAT_0064d928`) into a stack buffer and stamp the letter in;
2. `GetDriveTypeA` — result discarded;
3. `GetVolumeInformationA` with 0x80-byte volume-name and filesystem-name buffers — tested only for
   success, **no label comparison**;
4. on success, format `%c:\MafiaGame\a1.dta` (string at `0x00651b0c`) and call the `FindFirstFileA`
   existence helper at `0x006240b0`;
5. return **1** on the first hit, **0** after `'Z'`.

Sole caller `cd_require_disc` @ `0x00624070`: loops on 0 with
`MessageBoxA("Please insert MAFIA CD 1", 0x15 = MB_RETRYCANCEL|MB_ICONHAND)`, gives up on `IDCANCEL`.

**How the decomp carries it.** The annotated decomp holds **one** definition at `0x006240e0`,
`cd_scan_drives_for_disc`, covering the whole 171-byte function. Its body is Ghidra's own text over
the bytes at and after `0x006240e6`; only the docblock and the signature are hand-written, and the
docblock states that the four prologue bytes are **copied byte-for-byte** from
`Game-racing-mod.exe` rather than derived. Registry rows and evidence for it live in
`ghidra/Game-ghidra.exe.functions.md`.

⚠️ **A four-byte const-return stub at a function start is a crack fingerprint, and it is invisible to
the usual phantom heuristics.** The bytes `53 56` (`push ebx; push esi`) six bytes in *are* a
prologue, and nothing branches to that address — it is plain fall-through — so a tool that models
unreachable *epilogue* fragments rejects it. A crack orphan is a different shape and has to be
recognised by hand.

---

## 9. Confidence: the two cracked exes are ONE witness, not two

`.rdata` is **byte-identical** between `Game.exe` and `Game-racing-mod.exe` — all 45,056 bytes,
including the entire reconstructed 140-slot import table. Two crackers do not independently reproduce
a reconstruction byte-for-byte; they shared an unwrapping toolchain.

So the argument *"the two exes agree across `.text`, therefore the code is authentic"* is worth **one
witness**, not two. The single disagreement at `0x00629aaa` shows the toolchain had to *guess* at
least once, and the retail file cannot arbitrate because its ground truth is encrypted. Two
agreeing levers that share a premise are one lever.

---

## 10. How to re-measure any of this

Three read-only measurements underpin the sections above; each is reproducible from the files in §1.

| measurement | method |
|---|---|
| the crack inventory (§5, §7) | a full byte diff of two exes, every differing run attributed to its PE section + VA |
| the crack fingerprint (§8) | intersect two independent scans of `.text`: function starts whose first bytes are a ≤8-byte constant return, and function starts with **zero** incoming references. The reference census disassembles relative `E8`/`E9`/`Jcc` targets inside each function's own extent and adds every 4-byte little-endian value anywhere in the file that lands in `.text` |
| the import reconstruction (§3b) | resolve every IAT slot VA to DLL + API name and report duplicate-named slots |

The fingerprint scan takes a function inventory and an exe, so the same run works on any build —
which is what makes the control below meaningful:

| exe | `.text` starts | reference targets | unreferenced starts | const-return + orphan continuation |
|---|---|---|---|---|
| `Game.exe` | 6768 | 57589 | 16 | `0x006240e0` (`cd_scan_drives_for_disc`), bytes `33 c0 40 c3`, continuation at `0x006240e6` |
| `Game-racing-mod.exe` | 6768 | 57588 | 16 | **none** — the control |

The other 15 unreferenced starts are `crt_startup` (reached via the PE header, not by code) and CRT /
FPU / exception-handling helpers reached indirectly; none has a const-return predecessor.

---

## 11. Coda — leftover developer content nothing can reach

Found while parsing `.rsrc` for §5b. **Not crack artifacts**: `.rsrc` is byte-identical across
retail and both cracked builds, so all of this shipped from Illusion Softworks.

| resource | language block | contents |
|---|---|---|
| `DIALOG` **`IDD_PASSWORD`** | `#1029` = 0x405 **Czech** | `MAFIA` · `Password:` · `Cancel` · **`FOR INTERNAL USE OF TAKE2 ONLY!`** · **`THIS VERSION WILL EXPIRE ON xxxxxxxx.`** · `(C)1998-2002 ILLUSION SOFTWORKS` |
| `DIALOG` **`IDD_CRASH`** | `#1033` = 0x409 English | `Program crash...` · `Send crash info by e-mail` · `Close` |
| `MENU/#103` | `#1029` | 0x3e bytes, contents not decoded |

**Both are unreachable in the shipped product, and that is an import-table fact rather than a failed
search.** `Game.exe` imports **no dialog API at all** — no `DialogBoxParamA`, no
`CreateDialogParamA`, no `DialogBoxIndirectParamA`, no `EndDialog`. From `user32.dll` it imports
exactly two resource functions, `LoadStringA` (`0x0063b218`) and `LoadIconA` (`0x0063b21c`), which
are the two `structs/app_bootstrap.md` §3 documents. The Illusion DLLs cannot supply the gap either:
`LS3DF.dll` imports `LoadMenuA` but no dialog creator, and `IJoy.dll` / `INet.dll` import none.
Nothing in the shipped product can instantiate either dialog.

`IDD_PASSWORD` is a **press / review-build gate**: a password prompt plus a hard expiry notice whose
date is still the literal placeholder `xxxxxxxx`, meaning the retail build was produced from the
same source tree without the string ever being filled in. `IDD_CRASH` is a crash reporter that
would have mailed diagnostics home.

---

## 12. Open items

| item | state |
|---|---|
| the two padding bytes (§5f) | permanently unresolvable from these files — the retail `.text` that would arbitrate is encrypted |
| does `.data` in our exe equal the linked initial image? | an inference from the OEP dump (§4), not a measurement |
| the `stxt371` lowercasing mechanism (§5e) | needs the retail exe run under a debugger |
| the exact release-group name behind the `GiMP` tag (§5b) | the banner letterforms are not a measurement |
| can the retail `.text` be decrypted offline? | practically no — reproducing our `Game.exe` needs the disc or a SafeDisc emulator, which is how it was made in the first place |
| were either `.rsrc` dialog reachable in some *other* build (§11)? | unmeasured, deliberately: this file is a statement about the executables in §1 only. `LS3DF.dll`'s `LoadMenuA` import may relate to `MENU/#103`; not chased |
