# What this port gave 3dokit

3dokit (`3dokit/`, a submodule from
[vs-sr-dev/3dokit](https://github.com/vs-sr-dev/3dokit)) was started by
Immercenary's port and grew on Crash 'n Burn's and Doctor Hauzer's; Escape
from Monster Manor is its fourth game, and the first on Portfolio 21.10.
Each entry is a 3dokit commit and what this game asked of it. The
submodule was taken at **5d23270**; it is at **42a44c2**.

## Session 1

One commit, **b1b7739**, made and pushed with the user's word; every
port's submodule then moved to it (pc-crashnburn b62673f, pc-doctorhauzer
85a6d07, PC-Immercenary 59d8d45, all pushed; nothing moves in their
traces, so no new baseline). What it adds:

| file | What |
|---|---|
| `recomp/discover.py` | `_pointed_leaves`: a function with neither a prologue nor a name, reached only through a pointer -- a relocated word in a literal pool that reached code loads (`ldr rN, [pc, #k]`), naming the word just past a function's code, a word no function reaches -- is a function when its own descent is clean (no data met, no word of another function's), and so is what it calls. lib3DO's DataStream hands one over from `OpenStreamFile` (this disc's 0x1c4c8; Doctor Hauzer's 0x2ced4, a hand seed there) |
| `runtime/pf_math.cpp` | Operamath SWI 1, `MulMat33Mat33_F16` (OPERAMATH 21.10's Green 0x1f24; 20.53's 0x1ee0 the same): `MulManyVec3Mat33_F16`'s pipeline over the first matrix's three rows (`mul_many3`, shared with SWI 2) |
| `runtime/pf_graphics.cpp` | Graphics -136, `FillRect` (GRAPHIX 20.45's SWI 35: build 419's 0x2d8c, build 72's 0x2b5c): `DrawTo`'s one-pixel cel stretched over the rectangle |
| `cel.py`, `runtime/tdk_cel.c`, `tdk_cel.h` | a CCB with CCBPRE clear: its preamble taken from the start of its PDAT (one word packed, two literal), as the guide says and the runtime's cel engine already did -- no longer refused (`statscreen.cel`) |
| `stream.py`, `check.py`, `runtime/tdk_stream.c`, `tdk_stream.h`, `tdkcheck.c` | a DataStream with no `SHDR` -- its first chunk `CTRL`, `FILM` or `SNDS` -- recognised, its block size where its first block's `FILL` ends (this disc's four streams, 128 KiB); a stream with a header, or a marker table first, read as before |

### Discovery, checked by hand

Over the 17 programs of the five discs (Crash 'n Burn's `launchme` and
`Orion`; Immercenary's `p`, `p1e`, `launchme`, `CinepakSubroutine`,
`SpeechSubroutine`, `StorageTuner`; OMF2097's `LaunchMe`; Doctor Hauzer's
`launchme`, `sramtools`, `LMADM`, `FORMAT`; this disc's `LaunchMe`,
`loafanim`, `LMADM`, `FORMAT`), the rule adds **21 functions and changes
or loses none** (every program's function list before and after; 0
problems everywhere). Each was disassembled: sixteen are the C library's
`mov r0, #0; mov pc, lr` reached by pointer (Crash 'n Burn's 0x2f314 and
`Orion`'s 0x1bef0, two of that port's hand seeds, among them); Crash 'n
Burn's 0x25abc (a dispatch through a table, its third seed); Doctor
Hauzer's 0x2ced4 (its hand seed) and this disc's 0x1c4c8 (the DataStream
leaf); OMF2097's 0xd700 (an iterator) and 0x18e10 (a comparison for a
sort). A first form without the literal pool's
test also took five words of this disc's `LaunchMe` (0x1fb84, 0x1fc28,
0x1fcc0, 0x1fed8, 0x2013c: a second copy of an epilogue after a return),
named only from the Cinepak library's debugger tables in the read-write
data, which no code loads: the pool's test leaves them out.

### Regression

The old side built from this port's submodule (5d23270), the new from the
working tree, with session 1's scratchpad scripts (`regbuild.sh`,
`regrun.sh`, `readers.sh`, `discall.sh`, `newfuncs.py`, `whopoints.py`):

* **The generated C++**: Crash 'n Burn's the same byte for byte (its seeds
  already named the three functions; only `report.txt`'s origins move);
  Immercenary's, OMF2097's and Doctor Hauzer's each a few functions more.
* **The traces, byte for byte the same**: Crash 'n Burn's `launchme` to call
  234 (529 lines) and `--boot --max-calls 60000 --pad a@1300x1` (124,396
  lines); Immercenary's `p` (539,774 lines) and `--boot --max-calls 300000`
  with its replay's presses (810,154 lines); OMF2097's `LaunchMe` (107
  lines); Doctor Hauzer's `--boot --max-calls 400000 --pad start@400+6
  --pad start@11500+6` (893,210 lines, into its 3D rooms).
* **The self-test**, 0 failures: the optest on every build (891 functions,
  10,580 vectors), and the programs' own vectors generated with the new
  discovery -- Doctor Hauzer's `launchme` (246 functions, 3,812 vectors),
  Immercenary's `p` (382, 5,602), OMF2097's `LaunchMe` (108, 1,698), this
  disc's `LaunchMe` (157, 2,491).
* **The readers** (`cel --survey`, `cel --check`, `stream --scan`, and the
  runtime's check in Python and in C over the images) on Immercenary,
  OMF2097, Crash 'n Burn and Doctor Hauzer: **every output the same byte
  for byte**; on this disc 341 of 341 frames (340 before) and 4 streams (0
  before); Python and C agree on all five discs.
* Not rerun, and why: Crash 'n Burn's frames and `pfcheck` -- the cel
  engine, the allocator and every Graphics call `pfcheck` replays are
  untouched (a new slot and a new SWI only, which no other game calls:
  their traces above are the same).

## Session 2

One commit, **45a81b0**, made and pushed with the user's word; every
port's submodule then moved to it (pc-crashnburn 4198294, PC-Immercenary
00b73eb, pc-doctorhauzer 7c6e86c, all pushed; nothing moves in their
traces). What it adds:

| file | What |
|---|---|
| `runtime/pf_audio.cpp` | The audio folio's envelopes, AUDIOFOLIO 21.10's code (20.27's and 23.10's the same, 20.19's stops): `CreateEnvelope`, `DeleteEnvelope`, `AttachEnvelope`, `DetachEnvelope`; the envelope item (node type 6) and its tags; an attachment to an instrument's hook (`.incr`, `.request`, `.target`, `.current`); started by `StartInstrument`, stepped in chunks on the audio clock's timer list, released by `ReleaseInstrument`, stopped with the instrument's DSP side, deleted with it (`docs/04`). AUDIOFOLIO 21.10's own node database (a template 0x70). |
| `runtime/pf_font.cpp` | GRAPHIX's font table keyed by version and build, the build from the image's build line; build 419's addresses (`docs/04`) |
| `pfcheck.py` | `FontCall` for GRAPHIX 20.45 build 419 as well, the build picked by the folio's own words; the glue's stand-ins from each build's table (build 72's addresses were written in) |
| `README.md` | the runtime's and `pfcheck`'s rows |

Checked (session 1's scratchpad scripts, the new side from the working
tree):

* **The traces, byte for byte the same** as session 1's b1b7739 baseline,
  after the envelopes and again after the font: Crash 'n Burn's two (529
  and 124,396 lines), Immercenary's two (539,774 and 810,154), OMF2097's
  (107), Doctor Hauzer's boot into its 3D rooms (893,210).
* **`pfcheck` on GRAPHIX 20.45**: Doctor Hauzer's (build 72) start and
  its calls 986 `ResetCurrentFont`, 987 `GetCurrentFont` (0x4E69E4 both)
  and 988 `SetCurrentFontCCB`: 0 bytes differ; this disc's (build 419)
  start: 0 bytes differ.
* **This disc**: self-test 0 failures (1,048 functions, 13,071 vectors);
  the boot's trace to 400,000 calls the same before and after the font
  change (no font call yet); the run past field 20,000.

Then, with this repository published, **42a44c2**: the kit's README with
this port among the ports (Portfolio 21.10, what it asked, the first level
playable) and the cel engine's missing time among the known gaps; every
port's submodule moved to it. The README only: nothing to build or run.
