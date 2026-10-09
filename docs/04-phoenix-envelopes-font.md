# Phoenix, the envelopes, GRAPHIX build 419

Session 2: the runtime's frames against the user's Phoenix screenshots,
the audio folio's envelopes (the stop of session 1, call 196,030), and
GRAPHIX 20.45's font keyed by build.

## 1. Phoenix and the runtime, the same moments

The user's three Phoenix captures (`D:\Tools\phoenix28\ph-win64\3DO`,
`efmm@panafz10@ven ottobre 9 2026 15-23-41/47/56`, 953x686) against the
runtime's frames (`--frames`, 320x240): each capture scaled to 320x240
(bilinear) and compared pixel by pixel, the root mean square of the
difference over the three channels, and the mean difference per channel
(R, G, B):

| moment | runtime frame | RMS | mean difference |
|---|---|---|---|
| the title (15-23-41) | field 880, the best of every 10th field to 8,640 | 3.8 | 0.52, -0.03, -0.13 |
| the options tombstone (15-23-47) | field 8,640 | 5.6 | 0.31, -0.08, 0.00 |
| the first level after A on Start Game (15-23-56) | field 10,000, `--pad a@9000x1` | 3.9 | -0.15, -0.01, -0.09 |

What is left is the capture's JPEG and its scaling: positions, sizes and
colours are the same -- the tombstone's corner pixel (32, 26, 26) in both,
the level's background (26, 26, 26) in both, the walls' top and bottom at
the same heights of the frame. Nothing to ask the user about.

Before the envelopes the runtime stopped during the level's fade-in (field
9,320, at 1/25 of its brightness: a uniform scale on all three channels,
24.4, 25.2, 25.2 against Phoenix); with them it goes on (below).

## 2. The envelopes (AUDIOFOLIO 21.10.433)

### What the game does

Right after `ConnectInstruments(37 "Output" -> 556 "Amplitude")` (37 an
`envelope.dsp`), the game's 0xfc08 makes three points in its data,
`DataTimePair`s {time, data}: (0, 0), (200, its volume), (400, 0) -- the
volume 14,336 in this run -- and calls `CreateEnvelope(points, 3, 1, 1)`
(a sustain loop of the one point 1), `AttachEnvelope(37, envelope, "Env")`
and `StartInstrument(37)`; then it plays its music, and at the end stops
the instrument, `DetachEnvelope`, `DeleteEnvelope`.

### What the folio does

The four vectors are glue: `CreateEnvelope` (-136, 0x4dbc) is `CreateItem`
of node type 6 with `AF_TAG_ADDRESS`, `AF_TAG_FRAMES`, `AF_TAG_SUSTAINBEGIN`,
`AF_TAG_SUSTAINEND`; `AttachEnvelope` (-128, 0x564c) `CreateItem` of an
attachment with `AF_TAG_INSTRUMENT`, `AF_TAG_ENVELOPE` and the hook's name;
`DetachEnvelope` (-132) `DeleteItem`; `DeleteEnvelope` (-140, 0x55cc) frees
the points when its flag 1 says they are the folio's, then `DeleteItem`.

* **The envelope** (0x5558; defaults 0x4700, tags 0x4e10): the points and
  their count (1 or more), the sustain and release loops (-1: none), the
  times jumped back with (`AF_TAG_SUSTAINTIME` 52, `AF_TAG_RELEASETIME` 53),
  microseconds a unit (`AF_TAG_MICROSPERUNIT` 54, 1,000), the release jump
  (58), flags (bits 0 and 1); frames a unit = DivUF16(microseconds x 44,100,
  1,000,000), 44.1 in 16.16.
* **The attachment** (0x5bd4, its envelope part 0x5e14): the hook must be
  named; the instrument's knobs `<hook>.incr` and `<hook>.request` and its
  variables `<hook>.target` and `<hook>.current` (0x5058; envelope.dsp's
  `Env.*`), names compared without regard to case as 23.10 does (0xb29c);
  a block of 0x58 bytes holds the step's state and a node of the folio's
  timer list; the envelope's flag 2 becomes the attachment's
  `AF_ATTF_FATLADYSINGS`.
* **Start** (0x5420, from `StartInstrument` after the FIFOs' attachments):
  `.current` = the first point's data, `.target` = one less (so the first
  `.request` starts a segment), then the first chunk.
* **The step** (0x5234, run by the timer list's daemon): a segment of
  `time x frames a unit` frames goes in chunks of at most 0x8000; `.incr` =
  0x8000 over the frames rounded up, a chunk covers 0x8000 / `.incr`; with
  more than 400 frames left the chunk goes part of the way. `.incr` and
  `.request` are written into the DSP's memory (knobs, 0x9758; the
  variables through head.dsp's semaphore, 0x9b60); envelope.dsp ramps from
  `.current` to `.request` by `.incr` a frame. Past a sustain loop's end
  while unreleased: back to its begin, and a loop of one point holds with
  no timer. The next timer is the chunk's frames later in the clock's
  ticks, at least 1. Past the last point: `StopInstrument` with
  `AF_ATTF_FATLADYSINGS`, then the attachment's cue.
* **Release** (0x54a8) and **stop** (0x5548) from `ReleaseInstrument` and
  the instrument's DSP stop; deletions (0x6050, 0x5624, and the
  instrument's 0x8b50: the FIFOs' attachments, then the envelopes').

Two things the original does that the runtime does too: a sustain or
release index out of range ends the creation with the result of the
points' readability check, 0 -- the envelope is made, its later fields at
their defaults (23.10 returns `AF_ERR_BADTAGVAL` there) -- and the release
loop's end is compared with the sustain loop's begin, not its own.

### Across versions

The code compared instruction by instruction, registers and constants put
aside: **21.10 and 23.10 are the same; 20.27 (Doctor Hauzer) is the same
but for the minimum of one tick, which 21.10 adds**; 20.19 (Crash 'n Burn)
is another version (`.target` one more, frames = time x 44 with no time
scale, no minimum tick, no cue at the end). Neither of those two games
makes an envelope: the runtime does 20.27 on and stops on 20.19's. 21.10's
node database (0xba48) is its own as well: a template is 0x70 bytes, as in
23.10, a sample 0x98, as in 20.27; the runtime now has that table.

### In the run

`pfboot build/disc --boot --pad a@9000x1 --trace 2`:

```
envelope 561: 3 points at 0x22bd4
attach envelope 561 to instrument 37's "Env"
StartInstrument 37 (envelope.dsp)
envelope attachment 562: to 0 by 0x8000 over 1 frames
envelope attachment 562: to 13316 by 0x4 over 8192 frames
envelope attachment 562: to 14336 by 0x35 over 618 frames
```

200 units of 44.1 frames is 8,819 frames: 8,192 by 4 to the value that far
along, then 618 by 0x35 to the point itself, and the sustain holds there.
`envelope.dsp` is the 23.10 model's code, so it runs as the native model.
In the WAV (`--wav`) the level's music comes in at 155.92 s of guest time
and rises for about 0.2 s, from an RMS of 30 to about 220.

The run goes on through the level: 1,500,000 calls reach past field 20,000
(the monster in the corridor moving), with no stop -- 2 min 15 s on the
host with a frame written every 50 fields.

## 3. GRAPHIX 20.45: build 72 and build 419

`pf_font.cpp`'s table is now keyed by version **and build**, the build read
from the image's own build line once the image is laid
(`graphix 20.45.419 05/10/94 22:11:11 stan port1_3`). Build 419's addresses,
each read from its code: the font code is build 72's 0x2c4 on (identical but
for `cmp`/`teq`, every field offset included: the GrafFolio's font fields
still at 0xd0), its data 0x544 on (the characters' records 0x843c, the
built-in Font 0x6fe0, the current font's copies 0x6f28 and 0x6ee4, the
tree's head and butt 0x6f34 and 0x6f60, the root 0x6ee0, the missing
character's word 0x83d4), and on their own the PLUT 0x878c, the list's name
0x3b14, KernelBase's word 0x68c4 and GrafBase's 0x6a38.

Checked on the folio's own code: `pfcheck --font-start` rebuilds the font
as GRAPHIX's start leaves it and compares the whole memory with the
runtime's -- build 419 (a snapshot of this game's call 24): **0 bytes
differ**; build 72 (Doctor Hauzer): 0 as before. `pfcheck` tells the builds
apart by their words and stands in for each one's own glue (it had build
72's addresses written in).

Other places keyed by version: `pf_err.cpp` (kernel 20.21 and File folio
20.30, neither on this disc); `pf_graphics.cpp`'s node sizes ("20.45 and
on": both builds' node databases are the same, 0x74, 0x7c, 0x84, 0x44,
0x1c).

## The monsters' animation: twice the console's pace

Played in the window (the user's presses, `replays/attic-1.txt`, replay it
headless past field 6,000): the level
plays, the music comes in, no stop. The monsters' animation cycles look
fast; on Phoenix as fast as here; on the user's FZ-10 **about half as
fast**, the rest of the game the same. The runtime presents a new frame
every field in the level (60 `DisplayScreen`s each second of guest time,
from 157 s on): its clock counts the ARM60's clocks but not the cel
engine's work (`DrawCels` takes no guest time) nor the bus it shares, and
the console's MADAM needs about two fields for a frame of this level. The
monsters' animation steps once a frame; walking and turning are paced by
time (the game prints `throttleshift: 4` as the level starts), so they do
not change. The same thing as Total Eclipse's preview on Crash 'n Burn's
disc (pc-crashnburn's `docs/08-oracle.md`). The user's choice: the
console's pace is the developers', to be put in -- the kit's next work,
the cel engine's time (`TODO.md`).

## Not yet

This game has made no font call so far; `ResetCurrentFont` and the others
on build 419 are checked at the start only, the calls when it makes one.
The other envelope calls (`StartAttachment` and the rest on an envelope's
attachment, `LinkAttachments`, `AF_TAG_TIME_SCALE`) stop with "not yet".
