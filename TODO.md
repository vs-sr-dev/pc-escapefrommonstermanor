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
| `... --pad a@9000x1` | Start Game: the first level, `Attic Simple`, faded in, its music coming in through the envelope; **no stop**: 1,500,000 calls reach past field 20,000 (`docs/04`) |
| the window | **the first level plays** (the user, session 2): walking, the gun, the monsters, the music; the presses in `replays/attic-1.txt` replay it headless past field 6,000 |
| known difference | the monsters' animation runs at twice the FZ-10's pace (Phoenix's too): the cel engine takes no guest time, so the level shows 60 frames a second where the console shows about 30 (`docs/04`) |
| Phoenix | the title, the tombstone and the first level the same as the user's captures (RMS 3.8 to 5.6, the JPEG's; `docs/04`) |
| the kit | **45a81b0** in every port (`docs/10`): the envelopes, GRAPHIX's font keyed by build, `pfcheck` for build 419; regressed on every port, committed and pushed with the user's word |
| this repository | **published** at the end of session 2 (vs-sr-dev/pc-escapefrommonstermanor), the user's word: the first level plays |

```sh
# from D:/Homebrew6 whenever the kit has uncommitted work (the build's TDK_RUNTIME is then the kit's);
# the build/recomp points at D:/Homebrew6/3dokit/runtime, the kit's working tree
cd D:/Homebrew6 && PYTHONIOENCODING=utf-8 python -m 3dokit.recomp --out pc-escapefrommonstermanor/build/recomp \
  --optest "launchme=pc-escapefrommonstermanor/build/disc/LaunchMe"
export PATH=/c/msys64/mingw64/bin:$PATH                 # for cmake/ninja/clang only
cmake -S build/recomp -B build/recomp-build -G Ninja -DCMAKE_CXX_COMPILER=clang++ && ninja -C build/recomp-build
build/recomp-build/selftest build/recomp/selftest/*.txt
build/recomp-build/pfboot build/disc --boot --trace 1 --max-calls 400000 --pad a@9000x1 > build/t400k.txt   # 28 s
build/recomp-build/pfboot build/disc --boot --max-calls 1500000 --pad a@9000x1 --frames DIR --frames-at 9300-20000/50
```

Session 1's scratchpad (`2040b407-.../scratchpad`): `build.sh [--selftest]`
(the port's build from the kit's working tree), `mbattery.sh KITPARENT OUT`
(the kit's battery over this disc), `discall.sh KITPARENT OUT` (discovery
over the 17 programs of the five discs), `newfuncs.py`, `whopoints.py`
(the functions one discovery has and another not, and who points at
them), `regbuild.sh VARIANT KITPARENT` and `regrun.sh VARIANT` (the other
ports' builds and their traces; `reg/out-new` is b1b7739's baseline,
`reg/out-s3` the same from session 2's final tree), `readers.sh KITPARENT
OUT` (cel, stream and the Python/C check over the five discs), `syscmp.py`
(System files shared between discs), `odis.py`, `gvec.py`, `sheet.py`.

Session 2's scratchpad (`83783851-.../scratchpad`): `af2110.dis`,
`af_cnb.dis`, `af_dh.dis`, `af_im.dis` (the four AUDIOFOLIOs disassembled
whole: 21.10, 20.19, 20.27, 23.10), `norm.py FILE FROM TO` (a region's
instructions with addresses and constants put aside, for `diff` across
versions), `norm2.py` (the same keeping field offsets), `g72.dis` and
`g419.dis` (the two GRAPHIX builds' font code), the snapshots `snap419`,
`dh986` to `dh988` for `pfcheck`, and `h/` (the level's frames).

## The work, in order

### 1. The cel engine's time (the kit)

The monsters animate at twice the console's pace (`docs/04`): the runtime
charges the ARM60's clocks but not MADAM's work, so the level shows a
frame every field where the FZ-10 shows one every two. **The user's
choice (session 2): put it in** -- the developers' pace, and a point of
fidelity Phoenix does not have. It is the second game that shows it
(Total Eclipse's preview on Crash 'n Burn's disc, pc-crashnburn's
`docs/08-oracle.md`), so it is the kit's work: charge `DrawCels` the
cel engine's time (per cel and per pixel drawn, the bus it shares with the
CPU), from the hardware's documents and measured against the FZ-10 (the
user's eye now; a count of frames in a capture of the same moment would
be better). It moves every port's traces: each is to be checked against
its console, Crash 'n Burn's race start above all (the FZ-10 drops frames
there, the runtime does not).

### 2. The first level, played

The level now runs on its own with no stop. Next, in the window
(`--window`): the pad (walk, turn, the gun), the map (`FillRect`,
`DrawTo`), the monsters' cels, the sound; record the presses (`--record`)
and replay them to guard later changes; compare with Phoenix while
playing (the user's captures of the same moments). Any font call
(`ResetCurrentFont`, `DrawText16`, `DrawChar` are in the glue) runs on
build 419's addresses now: check its first snapshot with `pfcheck
--graphix` as Doctor Hauzer's were. The envelope calls not done stop with
"not yet" (`StartAttachment` and the rest on an envelope's attachment,
`LinkAttachments` on one, `AF_TAG_TIME_SCALE`): read them in 21.10 when
reached.

### 3. The save

`startopera` runs `$c/lmadm -a ram 3 0 nvram`; the game has a "Save
game" screen and `CreateFile` in its glue. Recompile the disc's LMADM
21.10 and FORMAT (as Doctor Hauzer does), and read File folio 21.11's
linked-memory filesystem against 20.30's (`pf_file.cpp`).

### 4. On from there

* The speed of the 3D level on the host, measured in the window against
  the console's 60 fields; the cel engine's per-pixel path is the place to
  look if it falls behind (the menu: about 375 frames a second, `docs/03`).
* GRAFMATH 21.10.504: `startopera` starts it; `LaunchMe` does not open it
  so far.

## Answered

* (Session 1) The kit's commit: done with the user's word (b1b7739),
  pushed, every port's submodule moved and pushed.
* (Session 1) This repository: committed locally only; published once the
  game "works until shown otherwise", as the other ports were.
* (Session 2) Phoenix: the title, the tombstone and the first level are
  the runtime's (`docs/04`); no difference to ask about.
* (Session 2) The kit's commit (45a81b0), every port moved; this
  repository published: the user's word, after playing the level.
* (Session 2) The monsters' double pace: the console's own (FZ-10 checked
  by the user); to be put in, through the cel engine's time.

## Questions for the user

* For the cel engine's time: can the FZ-10 give a count to measure against
  (a capture of the level at a known moment, the frames a second it
  shows)?

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

## Session 2 (2026-10-09) -- Phoenix, the envelopes, GRAPHIX build 419

* **Phoenix** (`docs/04`): the user's three captures (the title, the
  tombstone, the first level after A) against the runtime's frames of the
  same moments: RMS 3.8, 5.6 and 3.9, the mean differences under 1 --
  the same picture. Nothing to ask.
* **The envelopes** (`docs/04`): AUDIOFOLIO 21.10's envelope code read
  whole and compared with 20.19, 20.27 and 23.10 (21.10 = 23.10; 20.27
  the same but one minimum; 20.19 another version); in the kit: the item,
  the attachment to the hook, start, the chunks on the timer list,
  release, stop, deletions, the four vectors; 21.10's node database. The
  level fades in with its music and runs on, past field 20,000.
* **GRAPHIX build 419** (`docs/04`): the font table keyed by build, 419's
  addresses read from its code, checked with `pfcheck --font-start` (0
  bytes differ; `pfcheck` given build 419 too); the other version-keyed
  tables looked at.
* **The kit**: every port's traces byte for byte the same, twice;
  committed as 45a81b0 and pushed with the user's word, every port's
  submodule moved (`docs/10`).
* **The window**: the user played the first level; the presses kept in
  `replays/attic-1.txt`. The monsters animate at twice the FZ-10's pace
  (and Phoenix's): the cel engine's missing time, next.
* **The repository** published on GitHub.
