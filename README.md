# pc-escapefrommonstermanor

Toward a native PC port of **Escape from Monster Manor** (Studio 3DO /
Electronic Arts, 3DO, 1993; the European release, mastered in 1994), a
first-person shooter from the console's launch year: a haunted manor's
attics, dungeons, caves, catacombs and graveyard in real-time 3D, twelve
levels, ghosts, zombies and spiders to shoot and a talisman's pieces to
find. The goal is the game
running natively on PC by **static recompilation**: the game's own ARM60
code turned into C++ and run on a reimplementation of the 3DO's OS,
booting the disc as the console boots it, in a window in real time, with
its sound.

It is the fourth port built on
[3dokit](https://github.com/vs-sr-dev/3dokit), the game-agnostic 3DO
toolkit -- after [pc-immercenary](https://github.com/vs-sr-dev/pc-immercenary)
(Portfolio 23.10), [pc-crashnburn](https://github.com/vs-sr-dev/pc-crashnburn)
(the 1993 launch OS) and [pc-doctorhauzer](https://github.com/vs-sr-dev/pc-doctorhauzer)
(20.21) -- and the first on Portfolio 21.10. 3dokit is taken here as a
submodule: clone with `--recursive`, or run `git submodule update --init`.

## BYOA — Bring Your Own Assets

This repository contains **documentation and tools only**. No game data, no
executables, no assets, no OS code. You need your own original disc. The
work is done on the European release, one data track, as a .cue/.bin set
(raw MODE1/2352, 230,700 sectors; a CHD converts with MAME's `chdman
extractcd`). Hashes, sizes and addresses are written down here; bytes of
the product are not.

## Layout

    docs/            what the kit reads on this disc, the OS version, the boot, the sessions
    replays/         presses recorded in the window (pfboot --pad options), to replay
    3dokit/          game-agnostic 3DO toolkit (submodule)
    iso/, build/     your disc and everything derived from it (ignored by git)

## The route

The same route three ports have already walked (pc-crashnburn's
`docs/00-sessions.md`, PC-Immercenary's `docs/30-pipeline-pivot.md`,
pc-doctorhauzer's `TODO.md`):

1. **Recompile**: `3dokit.recomp` turns `/LaunchMe` (the whole game, one
   program) into C++, a function at a time, checked by the recompiler's
   self-test against an ARM60 interpreter.
2. **Boot it on the Portfolio runtime** (`pfboot`), one OS call at a time:
   each call the runtime does not answer yet stops the run; it is read in
   *this disc's* own OS code (the kernel, the folios, the shell are all on
   the disc) and implemented in the kit as that code does it, with its
   address in a comment, and checked against the other games on the kit.
3. **Compare** with the console and the Phoenix emulator; then play, record
   the presses, and replay them to guard every later change.

## Tools

The Python tools need Python 3.8+, and capstone for the code; run them from
the repository root. Building the C++ needs CMake, Ninja, clang and SDL3
(MSYS2's mingw64).

```sh
D=iso/efmm.cue
export PYTHONIOENCODING=utf-8

# the disc: volume, ROM tags, every copy compared; extract it
python -m 3dokit.disc "$D"
python -m 3dokit.disc "$D" --verify
python -m 3dokit.disc "$D" --extract build/disc

# the executables, the OS images, the pictures, the sound
python -m 3dokit.aif --scan build/disc
python -m 3dokit.aif --decompress build/disc/System/Folios/graphix build/os/graphix.bin
python -m 3dokit.cel --check build/disc
python -m 3dokit.stream --scan build/disc
python -m 3dokit.audio --scan build/disc
python -m 3dokit.dsp build/disc/System/Audio/dsp --verify

# the code
python -m 3dokit.arm build/disc/LaunchMe --names
python -m 3dokit.portfolio build/disc/LaunchMe --sites
python -m 3dokit.recomp.discover build/disc/LaunchMe --report

# the recompiler and its self-test
python -m 3dokit.recomp --out build/recomp --optest "launchme=build/disc/LaunchMe"
python -m 3dokit.recomp.selftest --image launchme=build/disc/LaunchMe --auto \
  --out build/recomp/selftest/launchme.txt
cmake -S build/recomp -B build/recomp-build -G Ninja -DCMAKE_CXX_COMPILER=clang++
ninja -C build/recomp-build
build/recomp-build/selftest build/recomp/selftest/*.txt

# the disc as the console starts it, every OS call traced; A at the menu
build/recomp-build/pfboot build/disc --boot --trace 1 --max-calls 300000 --pad a@9000x1
build/recomp-build/pfboot build/disc --boot --trace 0 --max-calls 300000 --pad a@9000x1 \
  --frames build/frames --frames-at 0-10000/30

# in a window, with the pad (arrows, Z X C, Enter); the user's first play replayed headless
build/recomp-build/pfboot build/disc --boot --trace 0 --window --record build/play.txt
build/recomp-build/pfboot build/disc --boot --trace 0 --max-calls 2000000 $(cat replays/attic-1.txt)
```

## Status

**Session 1**: the repository; the disc read by the kit (256 files; four
things it taught the kit: a DataStream with no header, a cel whose
preamble opens its pixel data, a function reached only by a pointer, and
two OS calls); the OS read -- **Portfolio 21.10** (kernel and Operator
21.10, File folio 21.11), and a GRAPHIX 20.45 that is not Doctor Hauzer's
20.45 (another build, two vectors more): a version alone does not name a
folio's code. `LaunchMe` recompiled whole (617 functions, no seeds), the
self-test at 0 failures, and the first boot: the 3DO and EA logos, the
title, the intro film over the graveyard, **the options tombstone**; with
A, `Start Game` loads the first level, `Attic Simple`, and fades its 3D
view in -- to call 196,030, `CreateEnvelope`.

**Session 2**: the user's Phoenix captures of the title, the tombstone and
the first level match the runtime's frames of the same moments; the audio
folio's envelopes, read in AUDIOFOLIO 21.10 and put in the kit, and the
level fades in with its music and runs on with no stop, past field
20,000; GRAPHIX build 419's font addresses, checked on the folio's own
code. **The first level plays in the window.** One difference from the
console, its own and wanted: the monsters animate at twice the FZ-10's
pace (and Phoenix's), the cel engine's time not yet charged -- next. See
`TODO.md`.

## Documentation

| | |
|---|---|
| [TODO](TODO.md) | the next session's work, and the history of sessions |
| [01-the-disc-on-the-kit](docs/01-the-disc-on-the-kit.md) | what 3dokit reads on this disc, and what it learnt from it |
| [02-the-os-version](docs/02-the-os-version.md) | Portfolio 21.10: each image's version and build, GRAPHIX 20.45.419, OPERAMATH 21.10, GRAFMATH |
| [03-the-first-boot](docs/03-the-first-boot.md) | the run to the menu and into the first level; the stops and what they asked |
| [04-phoenix-envelopes-font](docs/04-phoenix-envelopes-font.md) | the runtime against Phoenix; the audio folio's envelopes; GRAPHIX build 419's font |
| [10-3dokit](docs/10-3dokit.md) | what this port gave 3dokit |

## Licence

MIT -- see [LICENSE](LICENSE). Escape from Monster Manor is © 1993 The 3DO
Company; this repository contains none of it.
