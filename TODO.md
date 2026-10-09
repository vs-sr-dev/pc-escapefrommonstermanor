# Next session

The top of this file is the next session's work; the history of sessions
is at the bottom. Read the top, not the history.

## Where the port stands

| | |
|---|---|
| the disc on the kit | 256 files, battery clean after this port's kit changes (`docs/01`) |
| the OS | Portfolio 21.10: kernel and Operator 21.10, File folio 21.11, GRAPHIX 20.45.419, AUDIOFOLIO 21.10.433, OPERAMATH 21.10.603, GRAFMATH 21.10.504 (`docs/02`) |
| `LaunchMe` recompiled | 617 functions, 28,505 instructions, no seeds; self-test 0 failures |
| `pfboot build/disc --boot` | the 3DO and EA logos, the title, the intro, **the options tombstone** (field 8,640), waiting for the pad |
| `... --pad a@9000x1` | Start Game: the first level, `Attic Simple`, loaded and fading in |
| the stop | call **196,030**, audio -136 `CreateEnvelope` |
| the kit | **b1b7739** in every port (`docs/10`): discovery's pointed leaves, `MulMat33Mat33_F16`, `FillRect`, CCBPRE clear in the file readers, streams with no header; regressed on every port, committed and pushed |
| this repository | committed locally, not published (until the game "works until shown otherwise") |

```sh
# from D:/Homebrew6 whenever the kit has uncommitted work (the build's TDK_RUNTIME is then the kit's);
# the build/recomp of session 1 points at D:/Homebrew6/3dokit/runtime, now at b1b7739 too
cd D:/Homebrew6 && PYTHONIOENCODING=utf-8 python -m 3dokit.recomp --out pc-escapefrommonstermanor/build/recomp \
  --optest "launchme=pc-escapefrommonstermanor/build/disc/LaunchMe"
export PATH=/c/msys64/mingw64/bin:$PATH                 # for cmake/ninja/clang only
cmake -S build/recomp -B build/recomp-build -G Ninja -DCMAKE_CXX_COMPILER=clang++ && ninja -C build/recomp-build
build/recomp-build/selftest build/recomp/selftest/*.txt
build/recomp-build/pfboot build/disc --boot --trace 1 --max-calls 300000 --pad a@9000x1 | tail   # 9 s
```

Session 1's scratchpad (`2040b407-.../scratchpad`): `build.sh [--selftest]`
(the port's build from the kit's working tree), `mbattery.sh KITPARENT OUT`
(the kit's battery over this disc), `discall.sh KITPARENT OUT` (discovery
over the 17 programs of the five discs), `newfuncs.py`, `whopoints.py`
(the functions one discovery has and another not, and who points at
them), `regbuild.sh VARIANT KITPARENT` and `regrun.sh VARIANT` (the other
ports' builds and their traces; `reg/out-old` is 5d23270's baseline, and
Doctor Hauzer's boot is in it), `readers.sh KITPARENT OUT` (cel, stream
and the Python/C check over the five discs), `syscmp.py` (System files
shared between discs), `odis.py`, `gvec.py` (from Doctor Hauzer's session
1), `sheet.py`, and `rt-inst/` + `mm-inst/` (a runtime instrumented to time
the cel engine; scratch only).

## The work, in order

### 1. Phoenix: the user's screenshots first

The user starts the next session after taking Phoenix screenshots (in
`D:\Tools\phoenix28\ph-win64\3DO`, named by the date as for the other
ports): the 3DO and EA logos, the title, the intro, the options
tombstone, and the first level. Compare them with the runtime's frames
(`--frames`, the same moments: the tombstone at field 8,640, the level
after `--pad a@9000x1`) before anything else -- colours, positions, the
fades. A difference is a question first (the original's own glitches
exist: pc-crashnburn's radar), not a runtime bug by default.

### 2. The envelopes (call 196,030)

`CreateEnvelope(points, 3, 1, 1)` after `ConnectInstruments(37 "Output" ->
556 "Amplitude")`. Read AUDIOFOLIO 21.10.433 (`build/os/audiofolio.bin`,
its vectors: `CreateEnvelope` -136, `DeleteEnvelope` -140, `AttachEnvelope`
-128, `DetachEnvelope` -132) -- the envelope's item, its points
(`DataTimePair`s), its attachment to an instrument's knob or envelope hook,
and the folio's envelope service that moves the knob over time (on the
audio clock, as the cues are) -- and Crash 'n Burn's 20.19 and Immercenary's
23.10 for the version differences. Then `PauseInstrument` and
`ResumeInstrument` are in the glue too.

### 3. GRAPHIX 20.45 is two folios

`pf_font.cpp`'s `kFonts` keys GRAPHIX's font addresses by version, read on
Doctor Hauzer's build 72; this disc's build 419 has the same version and
other addresses (`docs/02`). Before this game reaches a font call
(`ResetCurrentFont`, `DrawText16`, `DrawChar` are in its glue): key the
table by build (the unpacked image's length, 33,384 against 34,700, or its
build line) and read 419's own addresses. Look for other places that key
a folio's addresses by version alone (`pf_err.cpp` keys the kernel's).

### 4. On from there

* The first level in 3D, played: the window (`--window`), the pad, the
  gun, the map (`FillRect`, `DrawTo`), the monsters' cels; compare with
  Phoenix (the user's screenshots: the 3DO logo, the title, the tombstone,
  the attic).
* The save: `startopera` runs `$c/lmadm -a ram 3 0 nvram`; the game has a
  "Save game" screen and `CreateFile` in its glue. Recompile the disc's
  LMADM 21.10 and FORMAT (as Doctor Hauzer does), and read File folio
  21.11's linked-memory filesystem against 20.30's (`pf_file.cpp`).
* GRAFMATH 21.10.504: `startopera` starts it; `LaunchMe` does not open it
  so far.
* The speed of the 3D levels (the menu: about 375 frames a second on the
  host, `docs/03`); the cel engine's per-pixel path is the place to look
  if a level falls behind the console.

## Answered

* (Session 1) The kit's commit: done with the user's word (b1b7739),
  pushed, every port's submodule moved and pushed.
* (Session 1) This repository: committed locally only; published once the
  game "works until shown otherwise", as the other ports were.
* (Session 1) Phoenix screenshots: the user takes them before session 2.

## Questions for the user

* None open.

## Keep in mind

* **Chat in Italian, every file in English.** No game data, no OS bytes in
  any repository; hashes, sizes and addresses are fine.
* **The image is a CHD**: `iso/efmm.cue`/`.bin` come from it with
  `D:/Homebrew4/Pippin/mame/chdman.exe extractcd -i "Escape from Monster
  Manor (Europe).chd" -o iso/efmm.cue -ob iso/efmm.bin`.
* `PYTHONIOENCODING=utf-8` for the Python tools; Python with the normal
  PATH (msys's has no capstone); mingw64 on PATH only for cmake, ninja,
  clang.
* C++ or Python with `\n` or `\\` goes through Write or Edit, never a
  shell heredoc -- not even a quoted one.
* Always `--max-calls`; the menu makes 24 calls a frame, so a run that
  waits there piles them up. Never `--trace 1` a long run into a file.
* The OS images, unpacked (`build/os/`): `os_code_0_10000.bin` (kernel
  21.10), `os_code_1_20000.bin` (Operator 21.10), `os_code_2_0.bin` (File
  folio 21.11), `os_code_0_1e000.bin` (`misc_code`), `graphix.bin`,
  `audiofolio.bin`, `grafmath.bin`, `eventbroker.bin`, `shell.bin`,
  `operamath.bin` (not compressed: a copy). GRAPHIX 20.45.419's SWI table
  at 0x6a54 (52, SWI n at word 51 - n), its vectors at 0x6b24 (51, slot -4
  the last word): `python odis.py build/os/graphix.bin 0 ADDR N`,
  `python gvec.py build/os/graphix.bin 6b24 6bf0 Graphics`.

# History

## Session 1 (2026-10-09) -- the repository, the disc on the kit, 21.10, to the first level

* **The repository**: git, the BYOA `.gitignore` of the sister ports, MIT,
  the README, 3dokit as a submodule at its published URL (5d23270, then
  b1b7739). Committed locally at the session's end; not published.
* **The disc** (`docs/01`): the user's CHD to .cue/.bin; 256 files, `--verify`
  clean; the kit's battery: 40 AIF images, 63 DSP instruments, 45 AIFF
  files, 0 disagreements with capstone, discovery with 0 problems. Two
  readers' refusals turned into kit changes: `statscreen.cel`'s preamble
  in its pixel data (CCBPRE clear), and the four films' DataStreams with
  no `SHDR`.
* **The OS** (`docs/02`): Portfolio 21.10 (kernel and Operator at fixed
  addresses, as 20.21), File folio 21.11, and every image's build line;
  GRAPHIX 20.45 build 419 against Doctor Hauzer's build 72 -- the same
  version, other code, 51 vectors; OPERAMATH 21.10's tables; GRAFMATH, new.
* **The recompiler**: `LaunchMe` whole, no seeds; self-test 0 failures.
* **The boot** (`docs/03`): to the options tombstone, and with A into the
  first level. Three stops answered in the kit: the DataStream's leaf
  reached by a pointer (discovery's rule, which also finds four of the
  other ports' hand seeds), `MulMat33Mat33_F16`, `FillRect`. The fourth,
  `CreateEnvelope`, is the next session's.
* **The kit**: regressed on Crash 'n Burn, Immercenary, OMF2097 and Doctor
  Hauzer (traces and boots byte for byte, self-tests, the readers byte for
  byte); committed as b1b7739 and pushed with the user's word, every
  port's submodule moved (`docs/10`). This repository committed locally.
