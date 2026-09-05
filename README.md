# Marvel: Ultimate Alliance — Static Recompilation

> Turning Raven Software's *Marvel: Ultimate Alliance* (2006, `BLUS30010`) from a
> PS3 disc binary into a native executable — no emulator underneath.

This project takes the disc's own `EBOOT.BIN`, disassembles every PowerPC
function, lifts them to C++, and links the result against
[ps3recomp](https://github.com/sp00nznet/ps3recomp) — clean-room HLE runtime
libraries that stand in for the PS3 operating system. Same approach as
[vf5](https://github.com/sp00nznet/vf5),
[twistedmetal](https://github.com/sp00nznet/twistedmetal) and
[simpsonsarcade-ps3](https://github.com/sp00nznet/simpsonsarcade-ps3).

**You supply your own disc.** No game binary, asset or key is committed here.

## Why this title

Two reasons, and the second is the one that made it worth doing.

**It is unusually clean.** A corpus sweep over the retail library ranked titles
by their *real* import table, and this one came second only to Virtua Fighter 5:

| | imports | libs | cellSpurs/Sync | network | exec |
|---|---|---|---|---|---|
| Virtua Fighter 5 | 107 | 10 | 5 | 3 | 7.9 MB |
| **Marvel: Ultimate Alliance** | **126** | **11** | **12** | 25 | 13.8 MB |
| Marvel: Ultimate Alliance 2 | 276 | 19 | 38 | 87 | 18.1 MB |

Its entire OS surface is eleven libraries:

```
sysPrxForUser 22   cellGcmSys 20   sys_net 19   cellSysutil 16   cellAudio 11
sys_fs 9   sys_io 8   cellSpurs 7   cellNetCtl 6   cellSync 5   cellSysmodule 3
```

No `cellSaveData`, no `cellGame`, no font or image decoders, no trophy or
commerce APIs. `PARAM.SFO` says `PS3_SYSTEM_VER 00.96` — this is a **launch-window
title**, built when the SDK barely existed, and it shows. The sequel, built three
years later against a mature SDK, has more than double the import surface and
three times the SPU footprint; it is the harder target despite being the same
studio and engine lineage.

The 25 network imports are online co-op against servers that no longer exist —
stub-shaped work, not emulation work.

**It is otherwise unplayable.** The PC release differs from the console versions,
and the 2016 remaster — which is a different game in several respects — has been
delisted. For the version that shipped on PS3, there is currently no way to play
it at all.

## Status

Analysis only. Nothing is lifted yet and nothing builds.

| Phase | State |
|---|---|
| Disc inventory | **done** — `BLUS30010`, disc game, firmware 00.96 |
| `EBOOT.BIN` → plain ELF | **done** — non-NPDRM disc SELF, 13.8 MB ELF |
| Import / NID analysis | **done** — 126 imports across 11 libraries |
| Function boundary detection | **done** — **38,648 functions**, every `.opd` descriptor verified as a function start |
| SPU image extraction | not started |
| PPU lifting | not started |
| Build & link | not started |
| Boot | not started |

For scale: Simpsons Arcade lifted 14,754 functions and is playable; Twisted Metal
31,032 and rasterises. 38,648 is a large but ordinary lift.

## Reproducing the analysis

```bash
# your own disc -> the plain ELF
python ../twistedmetal/tools/decrypt_self.py input/EBOOT.BIN -o input/EBOOT.elf \
       --keys ../ps3sce/data/keys

P=../ps3recomp/tools
python $P/elf_parser.py    input/EBOOT.elf --imports > meta/imports.json
python $P/find_functions.py input/EBOOT.elf --output meta/functions.json
```

## Related

- **[ps3recomp](https://github.com/sp00nznet/ps3recomp)** — the runtime this builds against
- **[vf5](https://github.com/sp00nznet/vf5)** — the other clean disc title, and the benchmark this was measured against

## Legal

No proprietary Sony or Activision code, assets or keys are in this repository.
Clean-room tooling only; everything derived from the game is generated locally
from a disc you supply.

Licensed under the [MIT License](LICENSE).
