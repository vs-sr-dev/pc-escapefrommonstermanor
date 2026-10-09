# 01. The disc on the kit

What 3dokit reads on Escape from Monster Manor's disc, tool by tool, and
what it did not until this port taught it. Every number below was produced
in session 1 with 3dokit at 5d23270 plus the changes of `10-3dokit.md`,
the commands as in the README, `PYTHONIOENCODING=utf-8` throughout. The
battery's outputs are kept in `build/battery/`.

## The disc

The user's image is a CHD (`Escape from Monster Manor (Europe).chd`, SHA-1
2ae5594b8c7da329edb78f9425552e0d7be89e40, data SHA-1
ec23b5f59b1df31d1d7241ad20dc7f550f7f47a3), turned into a .cue/.bin with
MAME's `chdman extractcd` (0.287): one track, MODE1/2352, 230,700 sectors;
the .bin's SHA-1 is 3d16a9611aaad00faafb8d8a56d96825abdcfaa3.

```
efmm.cue: raw 2352, 230400 blocks of 2048 (image 230700), label 'CD-ROM',
  root 3 block(s) x7 copies; 256 files, 29 directories, 404.8 MiB
  rom tag 0f.0d  v0.0   -> System/Kernel/boot_code (5050 bytes, tag says 5168)
  rom tag 0f.07  v0.16  -> System/Kernel/os_code (85896 bytes)
  rom tag 0f.0c  v0.0   offset 0xaa777fd8 size 0x0
  rom tag 0f.02  v0.0   -> LaunchMe (244452 bytes, tag counts blocks from block 0)
```

* **`--verify`**: the root's seven copies identical; 45 entries stored more
  than once, 121 extra copies -- 112 identical, 8 directory copies that
  differ only in the case of names, 1 `rom_tags` copy with its own relative
  offsets, 0 different. 0 entries past the image's end.
* **The ROM tags** are Doctor Hauzer's in form: the launcher's tag counts
  from block 0, `boot_code`'s says 118 bytes more than the file (the same
  `boot_code` as Hauzer's, byte for byte), and the os_code tag still says
  v0.16 while the kernel inside is 21.10 (`02-the-os-version.md`).
* **The tree**: `LaunchMe` (the game), twelve level directories named by
  `LevelSequence` (`Attic Simple`, `Attic Level 2`, `Interior 1` and `2`,
  `Dungeon`, `Cave`, `Catacomb`, `Graveyard`, two of each), each holding a
  `FloorPlan` (text: the level's music, its walls file, then the map) and
  two `.loaf` files; `Streams/` (four films); `aiff/` (music and effects);
  the cels at the root; `mm.font`; and `loafanim`, a development tool left
  on the disc (below). `AppStartup` is the stock empty one.

## The executables

`aif --scan`: **40 AIF images, 5 compressed, 12 signed, 0 failing**.

| | `/LaunchMe` | `/loafanim` |
|---|---|---|
| file | 244,452 | 13,960 |
| read-only | 0x2054c | 0x2e18 |
| read-write | 0x71ec | 0x6e0 |
| zero-init | 0x26a74 (154 KB) | 0x4830 |
| relocations | 969 | 53 |
| 3DO binary header | none, as on the 1993 disc | program, stack 0x1000 |
| stack (header) | 0x2800 | 0x1000 |
| `arm60 --check` | 32,215 words agree with capstone, 0 disagree | 2,840 agree, 0 disagree |
| discovery | 617 functions, 24 switches, 0 problems | 60 functions, 0 switches, 0 problems |
| embedded names | 134, all library | 14 |

* **The compiler's names are the libraries'**: the DataStream and its
  subscribers (`DataAcqThread`, `CtrlSubscriberThread`,
  `SAudioSubscriberThread`, `CPakSubscriberThread`...), the Cinepak codec
  (`Decompresx`, `ExpandCodeBook`, `CreateCodexHandler`...), the item and
  memory pools, `MyAllocMem`. As on the three other ports, **the films are
  played by the game's own linked code**, recompiled with the rest. The
  game's own functions carry no names.
* **Code past `ro`**: `LaunchMe`'s code runs on past the read-only area
  (0x2054c) to 0x21440 -- three small routines reached by calls, then seven
  compiled functions with APCS prologues (0x205c4 on). Discovery finds them
  all; no seed is needed (unlike Doctor Hauzer's hand-written 3D code).
* **A debugger's tables**: the Cinepak library was linked with its ARM
  debugger (ASD) tables, which sit in the read-write data (0x339xx on:
  `CreateCodexHandler` and its line records). They name code addresses --
  words just after functions' last returns among them -- and matter to
  discovery (`10-3dokit.md`).
* **`loafanim`** is not the game's: its names (`anim2loaf`, `writeLOAF`,
  `getPDAT`, `getPLUT`, `usage`) make it the tool that wrote the levels'
  `.loaf` files out of anims. Nothing on the disc runs it and `LaunchMe`
  does not name it. Read, recompilable, not needed.
* **The OS surface** (`portfolio --sites`, `LaunchMe`): 293 SWI sites to 40
  entry points (Kernel 17, File 6, audio 15, Operamath 2) and 122 vector
  sites, every one attributed, to 122 slots: Graphics 42, audio 45, Kernel
  23 (21 with an SDK name), Operamath 8, File 4. lib3DO's glue, a superset
  of what the game reaches. Worth noting: `MulMat33Mat33_F16` (met, `03-the-first-boot.md`),
  `FillRect` and `DrawTo` (the map?), the audio folio's envelopes
  (`CreateEnvelope`, `AttachEnvelope`, `DetachEnvelope`, `DeleteEnvelope`:
  met), `PauseInstrument`/`ResumeInstrument`, `CreateFile` (the save), and
  the "mac" device looked for, as on Doctor Hauzer's disc.

## Pictures, films, sound

* **`cel --survey`/`--check`: 48 files, 341 frames, 341 decoded** (340 before
  this port: see below). 241 8-bit coded packed (the monsters' anims:
  `head.cels` 90, `spider.cels` 56, `george.cels` 51, `zombie.cels` 43, and
  `intrlev.cel`), 64 6-bit coded (the `DS n.C6` and `IL n.C6` pieces, the
  compass, the status screen; one literal), 22 2-bit coded literal
  (`MapGlyphs.cel` with BGND, `ray.cel`), 14 16-bit uncoded (packed: the
  credits, the `.Uncoded16` screens, `skull.cel`, and with BGND `gun.cel`
  and `LaurieProbstMadeMeUseThis`; literal: `optionscr.cel` -- the options
  tombstone -- and `statpanel.cel`). Every CCB is followed by an **`XTRA`**
  chunk (48), which the kit carries. `LaurieProbstMadeMeUseThis` is the one
  the game opens at its start.
* **`statscreen.cel`** (the status screen's frame, 6 bits packed) was the
  one frame the kit refused: its CCB has **CCBPRE clear**, so the preamble
  is the first word of its pixel data (0x00003484, the same as the CCB's
  PRE0 word). The 3DO Graphics Programmer's Guide is explicit -- "CCBPRE:
  1 = at end of CCB, 0 = at start of cel data", one word for a packed cel,
  two for a literal one -- and the runtime's cel engine already read it so;
  the kit's file readers (`cel`, `tdk_cel`) now do too.
* **`stream --scan` found no stream**: the four in `Streams/` begin with a
  `CTRL` chunk, not an `SHDR` -- **DataStreams with no stream header**, which
  none of the three other discs has. Walked with 128 KiB blocks (where the
  first block's `FILL` ends), all four read to their ends: **7 Cinepak films,
  2,369 frames** (`PublisherLogo` 280x200 at a scale of 30, the others
  320x240 at 15; `Intro` holds four films, 206 + 93 + 606 + 1,020 frames),
  and SDX2 sound in all four (mono in `PublisherLogo`, stereo in the rest,
  22,050 Hz). The kit now recognises such a stream (`10-3dokit.md`).
* **`audio --scan`: 45 AIFF files, 0 failing**: the music and inter-level
  tracks uncompressed 16-bit stereo at 44,100 Hz (22, up to 64 MB:
  `LongTitle.aiff`), the effects AIFC SDX2 mono (20 at 22,050 Hz,
  `SpiderExplodes.aifc` at 44,100), and `sinewave.aiff` twice (the System's
  and a copy in `aiff/`); 2 with a sustain loop.
* **`dsp --verify`: 63 instruments, 219 knobs, 669 relocations**, every file
  walked to its last byte. `dsp --used LaunchMe` names 20 of them.
* The runtime's check, in Python and in C (`check`, `tdkcheck`): the same
  341 frames and 4 streams, 0 lines differ.

## The recompiler

```
launchme   617 functions    28505 instructions   (no seeds)
self-test: optest 891 functions 10,580 vectors; launchme 157 functions 2,491 vectors -- 0 failures
```

## What the kit does not read here

The game's own formats, which the recompiled game reads itself: the
`LOAF` files (`.loaf`: a header with a count and a PIXC word, then the
level's wall and pickup pictures, written by `loafanim`), the `FloorPlan`
maps, `mm.font`, and `Disc label`.
