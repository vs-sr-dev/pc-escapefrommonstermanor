# 02. The OS version: Portfolio 21.10

3dokit's Portfolio runtime has met three OSes: Crash 'n Burn's of 1993,
Doctor Hauzer's 20.21 (April 1994) and Immercenary's 23.10. Escape from
Monster Manor's disc -- the European release, the game itself being one
of the 1993 launch titles -- carries a fourth, **21.10**, mastered in May
and June 1994: between Doctor Hauzer's and Immercenary's.

## The release, read

`os_code` at 0xa4, where the runtime's `pf_os_release()` reads the release,
holds **0x15 0x0a: 21.10**, and the kernel's own 3DO header says the same.
The ROM tag still says v0.16, as on Doctor Hauzer's disc.

| disc | os_code release | kernel | ROM tag 0x07 |
|---|---|---|---|
| Crash 'n Burn (1993-09) | 0 | no header | v0.16 |
| Doctor Hauzer (1994-04) | 20.21 | at 0x10000 | v0.16 |
| **Escape from Monster Manor (Europe, 1994-06)** | **21.10** | **at 0x10000** | **v0.16** |
| Immercenary (1995) | 23.10 | linked at 0, relocates itself | v23.10 |

`os_code` is three AIF images again, unpacked by their own decompressor
(`build/os/`):

| | at | version | unpacked | its build line |
|---|---|---|---|---|
| kernel | 0x10000, NOP at 0x04 | 21.10 | 52,740 | -- |
| Operator | 0x20000 | 21.10 | 29,432 | `operator.rom 21.10.494 05/10/94 22:16:35 stan port1_3` |
| File folio | 0, relocatable | **21.11** | 31,420 | `fs.boot 21.11.1 06/09/94 10:44:00 stan port1_3_1` |

So in its kernel's form this OS is still 1993's and 1994's -- fixed
addresses, no relocation -- as Doctor Hauzer's; Immercenary's 23.10 is the
first to relocate itself. `misc_code` is one image at 0x1e000 (3,056 bytes
unpacked), no header.

## Each image has its own version -- and its own build

| | Crash 'n Burn | Doctor Hauzer | **Monster Manor** | Immercenary |
|---|---|---|---|---|
| kernel | (none) | 20.21 | **21.10** | 23.10 |
| Operator | 20.15 | 20.18 | **21.10.494** | 23.10 |
| File folio | 20.19 | 20.30 | **21.11.1** | 23.10 |
| GRAPHIX | 20.31 | 20.45.72 | **20.45.419** | 23.10 |
| AUDIOFOLIO | 20.19 | 20.27 | **21.10.433** | 23.10 |
| OPERAMATH | 20.27 | 20.53 | **21.10.603** | 23.10 |
| GRAFMATH | -- | -- | **21.10.504** (new) | -- |
| eventbroker | 20.21 | 20.31 | **21.10.469** | 23.10 |
| shell | 20.26 | 20.33 | **21.10** | 23.10 |

The third number is the build, from each image's own build line
(`graphix 20.45.419 05/10/94 22:11:11 stan port1_3`); the 3DO header holds
only the first two. Every build line found here but the File folio's is
of 10 May 1994, by the same hand ("stan", the `port1_3` tree); the kernel,
the shell and `misc_code` carry none.

**GRAPHIX is 20.45 on both 1994 discs, and it is not the same folio.**
Doctor Hauzer's is build 72 (6 April 1994, 33,384 bytes unpacked), this
one build 419 (10 May, 34,700 bytes): the same version number, other
addresses -- and **51 user vectors where build 72 has 49** (the folio's
`CREATEFOLIO_TAG_NUSERVECS`, 0x33 against 0x31): `QueryGraphics` and
`QueryGraphicsList` filled in, as 23.10's, and two more slots,
`DisplayOverlay` (-200) and `SetVBLAttrs` (-204), still the folio's "not
implemented" (0x1438, `0xD55B911B`). Its SWI table is at 0x6a54 (52, SWI 0
the last word), its vector table at 0x6b24 (slot -4 the last), the node
database after them.

What that means for the runtime: **a version alone does not name a
folio's code.** Where the runtime holds a folio's addresses keyed by its
version -- `pf_font.cpp`'s `kFonts`, GRAPHIX 20.45's font tree, read on
build 72 -- this disc's build 419 would match the version and lay the
font from the wrong addresses. The game's glue names the font's calls
(`ResetCurrentFont`, `DrawText16`, `DrawChar`); none has been reached yet.
Before one is, `kFonts` wants keying by build (the image's length, or its
build line), and 419's own addresses (`TODO.md`).

## OPERAMATH 21.10 and GRAFMATH

**OPERAMATH 21.10.603** (uncompressed on the disc, so its addresses are the
file's) picks its SWI table when it starts as 20.27 and 20.53 do (0x4f0):
MADAM revision 0 ("Red") 0x2d14, revision 1 ("Green") 0x2d78 unless the
CPU flags say wirewrap, anything else the software routines at 0x2cb0;
25 SWIs each, SWI 0 the last word. Its Green `MulMat33Mat33_F16` (SWI 1,
0x1f24) is `mov r3, #3` into `MulManyVec3Mat33_F16`'s pipeline (0x1f28):
the first matrix's three rows through the engine, each times the second
(`03-the-first-boot.md`). 20.53's 0x1ee0 is the same code.

**GRAFMATH 21.10.504** is a folio the other three discs do not carry.
`startopera` starts it in the background (`bg $folios/grafmath`), before
the event broker and the audio folio. `LaunchMe` does not open it (its
items opened by name are Graphics, File, audio, Operamath, the timer,
SPORT and "mac"): the runtime does not need it so far.

## The startup

`System/Scripts/startopera` ($Id 1.16, 5 April 1994):

```
kill kprintf
alias audio $boot/System/Audio ... alias fonts {/rom2/System/Graphics/Fonts|$boot/System/Graphics/Fonts}
$c/lmadm -a ram 3 0 nvram
bg $folios/grafmath
$tasks/eventbroker
@$folios/audiofolio
#minmem
$boot/AppStartup
```

The shell carries it out as on the other discs; `lmadm` (LMADM 21.10, signed
and privileged) is passed over, since this build has no module for it, so
the NVRAM is not maintained (Doctor Hauzer recompiles its disc's own; this
game has a "Save game" screen and a `CreateFile` in its glue, so it will
want the same: `TODO.md`).
