# Changelog

One section per export version (date tag). Counts are measured on the annotated decompilation
`ghidra/Game-ghidra.exe.annotated.c` against Ghidra's default auto-analysis of `Game.exe` 1.0.

## 2026.09.28 — initial export

- Annotated PC decompilation: 5,098 functions named, 0 `FUN_` left; 385 `EHHandler_*` and
  1,290 `Unwind_*` funclets named by owner.
- 2,651 functions mapped to their original PS2 developer method.
- ~5,400 docblocks, ~9,800 inline `//` comment lines, ~400 per-`case` docblocks.
- ~1,100 `g_*` globals and ~1,700 `s_*` strings named; raw `DAT_` tokens 1,253 → 14.
- Ghidra `WARNING` lines 2,688 → 3; records sorted by VA.
- Documentation: 116 struct/subsystem docs, 11 gameplay/engine write-ups, 5 police-system docs,
  5 data-format docs, 3 script-VM docs.
- Ghidra database `ghidra/Game.exe.gzf` (Ghidra 11.4.2) with all project names and data types.
