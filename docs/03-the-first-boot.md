# 03. The first boot: to the menu, into the first level

`LaunchMe` recompiled whole on the first try (617 functions, no seeds),
its self-test at 0 failures, and booted from the disc as the console boots
it (`pfboot build/disc --boot`). This is what it did, what stopped it, and
what each stop asked of the kit.

## The run

```
build/recomp-build/pfboot build/disc --boot --trace 1 --max-calls 4000000 --pad a@9000x1
```

* **The shell** carries out `startopera` and `AppStartup` and starts
  `$boot/LaunchMe`.
* **The start**: `LaunchMe` finds its directory and makes the `progdir`
  alias, opens the File, Graphics and Operamath folios, the timer (three
  times), SPORT and the audio folio; starts its threads -- "Joypad
  Reader", "SoundSpooler", "SoundSpoolerFader" (priority 180), the
  DataStream's own (106-111) -- reads `LevelSequence` and the cel
  `LaurieProbstMadeMeUseThis`, loads its effects (19 `.aifc` files), `mm.font`,
  `head.cels` and `zombie.cels` (its `kprintf`s: `vblankfreq = 60`,
  `scale: 0x24000`, `*** head ncels = 90`, `*** zombie ncels = 43`).
* **The opening**, frame by frame (`--frames`, one field in 30 looked at,
  `build/frames1`): "A 3DO Experience from Electronic Arts" over the 3DO
  logo; `Streams/PublisherLogo` -- the EA logo film, Cinepak decoded by the
  game's own codec into its screens --; the title, *Escape from Monster
  Manor*, on a stone wall; the smoke; the golden demon's medallion; and
  `Streams/Intro`'s fly-over of the graveyard to the manor's door, to the
  **options tombstone** (`optionscr.cel`) at field 8,640: Music On, Sound
  Effects On, View High Scores, View Credits, Play for keeps, Load Game,
  **Start Game** (lit), Replay Intro. There it waits for the pad, drawing
  its backdrop and some hundred glyphs a frame.
* **A at field 9,000** (`--pad a@9000x1`): the tombstone fades out, the
  game loads its first level from `LevelSequence` -- `Attic Simple`: its
  `FloorPlan` and `atticwalls.loaf`, found without case as the File folio
  finds them, the level's music `HauntOrgans.aiff` -- builds its 3D view
  (16-bit cels mapped by the projector) and fades it in: the attic's walls
  and the gun. At call **196,030** it stops at `CreateEnvelope`.

## The stops, and what they asked

### 1. A function reached only by a pointer (call 4,996)

Opening `Streams/PublisherLogo`, lib3DO's DataStream calls 0x1c4c8 through
a register: `ldr r1, [r0, #0x24]; ldr r0, [r0, #0x44]; b 0x1c9bc` -- three
instructions laid right after `_InitializeThread`'s last return and before
`OpenStreamFile`'s embedded name, a `static` function with no frame (so no
prologue and no name), whose address `OpenStreamFile` loads from its
literal pool (0x1c548). Doctor Hauzer met the same leaf at 0x2ced4 and
seeded it by hand; Crash 'n Burn had three like it among its 22 seeds.
Two games now: **discovery learnt the rule** (`10-3dokit.md`) -- a
relocated word in a literal pool that reached code loads, naming the word
just past a function's code, is a function when its descent is clean.
Over the 17 programs of the five discs it finds 21 such functions, each
checked by hand (sixteen the C library's `return 0` stub reached by
pointer, a sort's comparison, an iterator, a dispatch, the DataStream
leaf twice), among them Crash 'n Burn's seeds 0x25abc, 0x2f314 and
`Orion`'s 0x1bef0 and Doctor Hauzer's 0x2ced4; no function is lost or
changed. The pool matters: without it,
five words after functions' returns in `LaunchMe` -- named only by the
Cinepak library's debugger tables in the read-write data -- passed as
functions too.

### 2. `MulMat33Mat33_F16` (Operamath SWI 1, call 194,497)

The first thing the level asks: OPERAMATH 21.10's Green 0x1f24, a count
of 3 into `MulManyVec3Mat33_F16`'s pipeline -- the first matrix's three
rows through MADAM's matrix engine, each times the second, each output the
64-bit sum of the three products shifted down 16 (`02-the-os-version.md`).
In the kit as the 3x3 form of `MulMat44Mat44_F16`.

### 3. `FillRect` (Graphics -136, call 194,604)

GRAPHIX 20.45's SWI 35 (build 419's 0x2d8c, build 72's 0x2b5c: the same
code, only two branch offsets differ): `DrawTo`'s cel -- one 16-bit pixel of
`gc_FGPen` behind its preamble (0x16, 0), CCB flags 0x57260030, PIXC
0x1f401f40, width and height 1 -- laid on the folio's stack and stretched
over the rectangle: the corners put in order, the origin (left, top), HDX
the width right - left + 1 in 12.20, VDY the height bottom - top + 1 in
16.16, then `DrawCels`. The game sets its pen to 0x50a5 and fills, a
hundred calls after its first `MulMat33Mat33_F16`.

### 4. `CreateEnvelope` (audio -136, call 196,030): the next session

`CreateEnvelope(points, 3, 1, 1)`: an envelope of three points with its
sustain on the second, for the level's sound, right after the game
connects an instrument's "Output" to another's "Amplitude". The audio
folio's envelopes are not in the runtime: they are AUDIOFOLIO 21.10's
own (`CreateEnvelope`, `AttachEnvelope`, `DetachEnvelope`,
`DeleteEnvelope`, and the folio's envelope service that moves an
`envelope.dsp` instrument's knobs over time), to be read in this disc's
folio (`TODO.md`).

## Speed

The batch run is not real time: the guest's clock moves by the ARM60's
clocks and jumps ahead when every task waits. The whole run above, to the
first level's fade-in (196,030 calls, some 9,300 fields), takes 9 seconds.
Waiting in the menu, the game makes 24 OS calls a frame and draws its
full-screen 16-bit backdrop (320x240, PIXC 0x1f00) once a frame: a copy of
the runtime instrumented for the purpose counted 55,004 such frames in a
run of 146 s, about 375 a second, the backdrop about 2 ms of it each and
the glyphs nearly nothing -- the cel engine is most of the host's work
there, and the window has room to spare. A short cut in the projector for
unit squares (a cel at its own size) gave the same frames byte for byte
and only 6%: the time is in the per-pixel path, not the polygon; not kept.
The 3D levels are still to be measured.

A run that waits in the menu with no press accumulates calls fast (24 a
frame): give `--max-calls` with that in mind, and never `--trace 1` such a
run into a file.
