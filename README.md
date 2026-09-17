-=(Xybots_Senhor notes)=-

Tested: Working Video 720p, 1080p & Sound.

___
# Xybots for MiSTer FPGA

A hardware recreation of Atari Games' 1987 arcade title **Xybots**, implemented
as an FPGA core for the
[MiSTer](https://github.com/MiSTer-devel/Main_MiSTer/wiki) platform.

This is not a software port. The core reconstructs the original PCB set — the
game board (68000 + SLAPSTIC 137412-107) and the Atari **JSA I** stand-alone
audio board (6502 + YM2151) — from Atari's own schematic package, **SP-313**,
with MAME 0.288 used as a functional oracle where the drawings are silent.
"Sheet N" anywhere in this repository means sheet N of that package. Where a
schematic and MAME disagree, the schematic wins and the disagreement is
recorded: see [Where this core differs from MAME](#where-this-core-differs-from-mame).

Xybots is its own platform. It is not Atari System 1 or System 2, even though it
shares the SYNGEN video-sync custom with System 1 and the JSA I audio board with
Blasteroids, Vindicators and Skull & Crossbones.

## Supported ROM sets

All five MAME sets run on identical RTL — they differ only in the main CPU ROMs
(and, for `xybots0`, in three tile ROMs and the sound ROM's filename).

| MRA | MAME set | Region |
| --- | --- | --- |
| `Xybots.mra` | `xybots` | World, rev 2 |
| `_alternatives/_Xybots/Xybots (rev 1).mra` | `xybots1` | World, rev 1 |
| `_alternatives/_Xybots/Xybots (rev 0).mra` | `xybots0` | World, rev 0 |
| `_alternatives/_Xybots/Xybots (German, rev 3).mra` | `xybotsg` | Germany |
| `_alternatives/_Xybots/Xybots (French, rev 3).mra` | `xybotsf` | France |

## Requirements

| Item | Requirement |
| --- | --- |
| Board | DE10-Nano (Cyclone V `5CSEBA6U23I7`) |
| SDRAM | **Required** — a standard 32 MB module (MT48LC16M16-class) |
| Display | Horizontal (ROT0) cabinet, **336 × 240 visible at 59.695 Hz** |
| ROMs | Supplied by you from a MAME archive; none are included here |

**The SDRAM module is not optional.** The 256 KB of playfield tile graphics and
the 448 KB of motion-object graphics are streamed from it at runtime and there is
no block-RAM fallback for them. (The 192 KB 68000 program ROM, the 64 KB sound
ROM, the 8 KB character ROM, video RAM, colour RAM and the two motion-object line
buffers *are* on-chip, which is why the core needs only 768 KB of the module.)
The controller is addressed for the usual 4-bank × 13-bit-row × 9-bit-column
part, so a legacy 8 MB module will not work even though little of the module is
used.

**Refresh rate.** This core runs the raster at **456 × 263 counts, 59.695 Hz**,
derived from the real Atari sync PROM rather than from MAME's published 262 /
59.923 — see [the divergence table](#where-this-core-differs-from-mame). If a
display refuses to lock, that is the first thing to check.

## Installation

1. Copy `releases/Xybots_YYYYMMDD*.rbf` to `_Arcade/cores/` on the SD card.
2. Copy `releases/Xybots.mra` to `_Arcade/`, and any wanted alternates from
   `releases/_alternatives/` to `_Arcade/_alternatives/`.
3. Place the matching MAME ROM archive (`xybots.zip`, etc.) in `games/mame/`.

**No ROM data is included in this repository, and none ever will be.** The MRA
builds the correct memory image from an archive you already own.

## Controls

Each player has an 8-way stick with an **independently rotating knob** on top,
plus a Fire button and a Start button. The knob is two ordinary switches, not a
stick direction, so it maps to two buttons.

| Button | Function |
| --- | --- |
| D-pad | 8-way stick (move) |
| A | Fire |
| L | Turn Left |
| R | Turn Right |
| Start | Start |
| Select | Coin |

The operator manual's Switch Test page calls the knob switches *Turn Left* /
*Turn Right*; MAME names the same two inputs *Twist Left* / *Twist Right*. Coins
are read on the JSA audio board, and the two chutes are wired **swapped** — as
they are on the real board.

## Options

Xybots has **no DIP switches**. Every operator setting — difficulty, lives,
coinage, bookkeeping — lives in the on-board Xicor X2804 EEPROM and is edited
from the game's own self-test, reached with the OSD **Service** toggle. Settings
and high scores persist through MiSTer's *Save Settings*; the MRA declares a
512-byte NVRAM at index 2.

The OSD also offers Aspect ratio, Orientation (Original / Flip) and Scale, and an
**Analog alignment** page — CRT H-Size, CRT H-Position, Analog VGA H-Shift and
V-Shift, the same four controls as the [Arcade-Toobin_MiSTer](https://github.com/MiSTer-devel/Arcade-Toobin_MiSTer) and [Arcade-Klax_MiSTer](https://github.com/MiSTer-devel/Arcade-Klax_MiSTer) cores — for centring
and sizing the picture on an analog CRT. The page acts on the output side only
(`rtl/video/xybots_analog_adjust.sv`): the native 456 x 263 raster, the 7.159 MHz
dot clock and the 59.695 Hz refresh are untouched, and at the zero settings the
stage is a bit-exact bypass, so the default picture is the hardware-validated one.

## Hardware baseline

| Block | Implementation |
| --- | --- |
| Main CPU | Motorola 68000 @ 7.159090 MHz (14.318181 MHz ÷ 2), big-endian 16-bit — implemented with **fx68k**, a cycle-exact microcode-level 68000 |
| Protection | Atari **SLAPSTIC 137412-107** at 14B, banking ROM `0x8000-0x9FFF` in 4 × 8 KB pages |
| Program ROM | 192 KB (17CD / 19CD 64 KB each + 17B / 19B 32 KB each), in block RAM |
| Sound board | Atari JSA I: 6502 @ 1.7897725 MHz + YM2151 @ 3.579545 MHz. **POKEY and TMS5220 are not populated** on this board and are not implemented |
| Raster | SYNGEN 11E/F: 456 × 263 total, **336 × 240 visible**, 7.159090 MHz pixel clock, 59.695 Hz |
| Playfield | 64 × 32 map of 8 × 8 4 bpp tiles. **No scrolling** — Xybots has no scroll register |
| Alphanumerics | 64 × 32 map of 8 × 8 2 bpp characters, drawn on top |
| Motion objects | 64-entry × 4-word table at `0x802E00`, **not linked, no SLIP**, per-line vertical match, ping-pong line buffers, 4-bit priority per object |
| Priority | The 74S85 magnitude comparator at 12J (not a PAL — sheet 8 is unambiguous) + the GPC custom at 14K form the colour-RAM address |
| Palette | 1024 × 16-bit colour RAM, I:R:G:B 4:4:4:4, through the sheet-9 resistor-ladder DACs |
| NVRAM | Xicor X2804 parallel EEPROM, one-shot unlock strobe, lock-after-write |
| Watchdog | The sheet-2 LS10 network, cleared by a write to `0x806A00` |
| Interrupts | IRQ1 = the programmable **scanline** interrupt (VIRQ), IRQ2 = the sound board |

The whole core runs in a single **57.272727 MHz** `clk_sys` domain with integer
clock enables, which is how the real board works: the JSA crystal (3.579545 MHz)
is exactly a quarter of the video crystal (14.318181 MHz), so every rate on both
PCBs is an integer division of `clk_sys`. There are no fractional enables
anywhere.

Resource use on the DE10-Nano's Cyclone V (`5CSEBA6U23I7`), for the newest
bitstream in `releases/`: **14,659 / 41,910 ALMs (35 %)**, 400 / 553 M10K blocks
(72 %), 3.1 / 5.7 Mbit of block memory (55 %), 52 / 112 DSP blocks. Quartus
17.0.2 with multi-corner analysis on: **every slack entry positive, TNS 0.000 on
every clock and every check** (setup, hold, recovery, removal) across all 144
corner entries, worst hold +0.080 ns on the core PLL at the fast −40 °C corner.

## Accuracy and verification

This core was developed with AI assistance. Because that raises a fair question
about whether the result was actually checked, here is the verification record.
Every claim below has a named gate behind it. Like the other MiSTer arcade
repositories, this one ships the core and its release files only; the
simulation benches, MAME capture scripts and design notes are kept in the
development tree and are not part of this repository.

**Simulation suite** — 38 registered testbenches (Icarus Verilog, plus GHDL +
Yosys netlists of the vendored VHDL CPUs for the cosims), each printing
`PASS`/`FAIL`, each paired with a warning-clean Verilator lint. Gates are
required to be **non-vacuous**: a gate must demonstrate that its inputs can
distinguish outcomes, and several run a deliberately broken mutant first and fail
if the mutant passes.

**Programmed logic** — the one PAL the core depends on, the JSA sound board's
address decoder (Atari 136056-2101), is implemented from its fuse map rather
than from the memory map. The fuse map is evaluated independently in Python and
the RTL transcription is held to it over all 1,048,576 input combinations; the
reading of that chip the schematic alone suggests is run as a mutant and
rejected. The game PCB itself carries no PALs.

**Video timing** — the SYNGEN raster is reconstructed from the real Atari System 1
sync PROM 136032-102 and checked by an exhaustive one-frame bench that rebuilds
its expectations from an independent model. Of eleven timing mutants, nine are
killed and the other two were proved equivalent.

**Main CPU boot** — the 68000 bus is co-simulated against MAME 0.288 through
reset, the RAM and ROM tests, the SLAPSTIC checksum walk, 143 EEPROM writes and
the scanline-interrupt path: **all 42,057 bus events of the comparison window
and 100,000 program counters with zero divergence.** The game's register-only
EEPROM delay loop costs **959,888 `clk_sys` per pass, exactly the 68000
timing-table figure** — an earlier build on a non-cycle-exact soft core ran it
28 % fast, which is why the core now uses fx68k and nothing else. The SLAPSTIC
107 is proved equivalent to MAME's state machine over the full alt / bit / add
sequences.

**Interrupt entry** — a dedicated gate takes both interrupt levels, measures the
68000 E clock (80 / 32 / 48 `clk_sys` = CPU-clock ÷ 10) and bounds the `/VPA`
autovector cycle on both sides. It runs the pre-fix CPU as a mutant every time
and fails if the mutant does not hang. This gate exists because an earlier build
*did* hang on its first interrupt on real hardware; that bitstream was withdrawn.

**Sound** — the JSA I board is compared to the oracle instruction by
instruction: **590 / 590 6502 I/O accesses and 686 / 686 YM2151 register writes
in order**, with the periodic interrupt measured at 249.0 Hz against the board's
computed 249.69 Hz over one second.

**Pixel-exact video gates** — MAME is instrumented to capture complete video
state (alpha, playfield and motion-object RAM plus the palette) at chosen frames.
Those states are replayed through the real core RTL and compared to MAME's own
rendered bitmap pixel for pixel. The alpha layer, the playfield layer, the
motion-object engine and the full compositor each have their own gate; the
compositor is **bit-identical to MAME's `golden.ppm` on attract, gameplay and
self-test frames**, including 625 pixels decided by motion objects.

**Full-core cosim** — the whole integrated core boots in Verilator with the real
ROM image streamed in over its own download port, read back out of an SDRAM chip
model by region hash, and renders a captured gameplay frame pixel-exactly.

**Memory path** — every ROM region is reproducible by hash, and the MRA byte
stream is proved to reconstruct the download image. The interleave lane map is
proved non-vacuously: the reversed map produces a different, wrong hash.

**Real hardware.** Verified on a DE10-Nano with a 32 MB SDRAM module: the SDRAM
read-capture phase was swept across eight bitstreams (good window 225°–315°, the
shipping value is the 270° centre), and boot, attract, the self-test Switch Test
with all sixteen inputs, coin crediting, sound, EEPROM persistence across a power
cycle and long play sessions all pass.

## Where this core differs from MAME

MAME is this project's functional oracle and every gate above measures the core
*against* it, so this is a narrow list, not a general claim. It is where the
SP-313 schematics or the sound board's decoder PAL document something MAME's
model does not — and in two of the seven cases MAME's own source flags the gap.

| Behaviour | MAME | This core |
| --- | --- | --- |
| Lines per frame | `set_raw(..., 262, ...)`, 59.923 Hz, above a comment saying these are *"from published specs, not derived"* and that the board uses a SYNGEN chip | **263 lines, 59.695 Hz**, derived from the real System 1 sync PROM 136032-102, whose table only closes if `/VRES` loads the V counter with all-ones |
| IRQ1 | raised once per frame from `screen_vblank()` | the board's **programmable scanline interrupt**: alpha-RAM word bit 10, latched as the alpha row is fetched, gated by `V[2:0] == 7 & HBLANK & ~VBLANK`, held until a write to `0x806B00` |
| Motion objects per line | draws all 64 entries | the `/DIE` write window admits entries **0…55**; 57 entries are *fetched* per line, 56 are displayable. Entry 56 is fetched and discarded |
| Palette DAC | `IRGB_4444` — two 4-to-8-bit expansions multiplied and truncated, full scale 254 | the **sheet-9 resistor ladders**, computed in exact rational arithmetic |
| `0x806D00` | not decoded at all | decoded (LS138 Y5) but **not connected** — the drawing shows a bare stub |
| Sound CPU: YM2151 select | `0x2000-0x2001` | **`0x2000-0x27FF`** — the decoder PAL 136056-2101 sees neither SA10..SA1 nor anything the YM2151 (which has only A0) could use, so the chip mirrors across the block |
| Sound CPU: `/IRQACK` at `0x2806` | read **or** write | **read only** — the same PAL drops the LS138's enable for writes to that half of the I/O block |

Notes on the last five:

**Motion objects.** The game's own object list ends at entry 55 and parks entry
56 at X = 480, and entries 57…63 were never used in 3,596 captured frames, so the
difference is not observable in this game — but the engine implements the window
the schematic draws, not a model of it.

**The DAC.** Over the whole 256-entry product table the two curves differ by at
most **4 counts of 255** (mean absolute difference 0.82; 99 of 256 entries are
identical). None of it is visible on a display. A `DAC_MAME_COMPAT` parameter
selects MAME's curve so the frame-CRC gates can compare bit for bit; the shipping
build uses the schematic's.

**`0x806D00`.** The game really does write `0x20`, `0x24` and `0x17` there, at
three sites in the program ROM. The decoder output exists on sheet 2 and goes
nowhere, so the writes terminate like any other decoded-but-unconnected access
and have no effect. MAME does not map the address at all, and reaches the same
outcome by a different route.

**The sound decoder.** Both rows come from the fuse map of the JSA board's
address-decoder PAL, which also resolved a long-standing puzzle in the drawing
(the LS138's active-high enable is labelled `/SRD`: inside the I/O block that
signal is the PAL's read/write code for the decoder, not a read strobe). Neither
row is observable in this game: the sound program addresses the YM2151 at
`0x2000`/`0x2001` only and only ever *reads* `0x2806`, and MAME taps over attract
and gameplay recorded zero accesses of either kind.

One more difference worth naming, which is **not** a schematic conflict: the JSA
I sound driver's effect pitch comes from an 8-bit Galois LFSR that the 6502 steps
once per idle-loop pass — a classic seed-from-idle-time random generator. Its
value therefore depends on cycle-level idle timing, and the sound gate accepts one
LFSR-transposed key code out of 686 YM writes for that reason. Every other
register write matches in order and in value.

## Open items

**1. A SYNGEN dump would settle the 263-line count directly.** The vertical
total is derived from the System 1 sync PROM and the discrete System 1 circuit
that the SYNGEN custom replaced; the custom itself has not been decapped or
dumped. If you can read one out, that is the most valuable contribution this
core can receive.

**2. Every new bitstream gets a glance at the graphics on a board.** The SDRAM
controller captures read data open-loop — it latches the data pins a fixed number
of clocks after the read command, and the phase of the clock sent to the chip is
a constant chosen on hardware. That is how MiSTer cores handle SDR SDRAM in
general; the shipping value here (270°) is the centre of a window swept across
eight bitstreams, and the shipping fit is hardware-verified. The margin is a
property of the placement as well as the phase, though: one fit inside the
measured-good window still failed. A possible improvement, standard in DDR
controllers but not used by any MiSTer SDR core the author knows of, would be a
self-calibrating capture — write a known pattern at initialisation, read it back
across the capture taps, and lock onto the centre of the window the board
actually has. It is not planned; it is noted here so the trade-off is on record.

## Building

Quartus Prime **17.0.2** (Lite or Standard) — the version MiSTer standardises on;
`Xybots.qsf` pins it. Open `Xybots.qpf` and run a full compile; the bitstream
lands in `output_files/Xybots.rbf`, already named for the `<rbf>Xybots</rbf>` the
MRAs ask for, so there is no rename step.

Source files are listed in `files.qip`, which Quartus reads but cannot edit — add
and remove entries by hand. **Never add files through the Quartus GUI**: it
rewrites the `.qsf`.

## Attribution

This core's own RTL, and this tree, are **GPL-3.0-or-later** (`LICENSE`).
Individual third-party components keep their own notices, which must be
preserved; `NOTICE.md` is the authoritative list.

| Component | Author | License |
| --- | --- | --- |
| MiSTer framework (`sys/`) | Till Harbaum, Alexey Melnikov (sorgelig) and MiSTer-devel contributors | GPL-2.0-or-later / GPL-3.0-or-later per file |
| fx68k (cycle-exact 68000, the main CPU) | Jorge Cwik (ijor), <https://github.com/ijor/fx68k> | GPL-3.0-or-later |
| analog_hsize (CRT H-Size / H-Position resampler) | Umberto Parisi (rmonic79), Arcade-Raiden_MiSTer | GPL-3.0-or-later |
| T65 (6502) | Daniel Wallner, with fixes by Mike Johnson, Wolfgang Scherr and Morten Leikvoll (OpenCores / FPGAArcade) | BSD-style, three clauses |
| jt51 (YM2151) | Jose Tejada (jotego) | GPL-3.0-or-later |
| Altera/Intel PLL megafunction | Intel Corporation (generated IP) | Intel FPGA IP licence, as generated |

The T65, jt51, SDRAM and JSA I building blocks, the MiSTer glue, the analog
alignment stage and the verification methodology come from the [Arcade-Toobin_MiSTer](https://github.com/MiSTer-devel/Arcade-Toobin_MiSTer)
core; the analog alignment page matches the one in [Arcade-Klax_MiSTer](https://github.com/MiSTer-devel/Arcade-Klax_MiSTer). The SYNGEN and
line-buffer behaviour was cross-checked against d18c7db's [Arcade-Gauntlet_MiSTer](https://github.com/MiSTer-devel/Arcade-Gauntlet_MiSTer) core,
which models the same Atari customs. The SLAPSTIC is *not* derived from those
cores: it was written from SP-313 sheet 2 and verified against MAME's state
machine.

Thanks to the MAME team, whose `xybots.cpp`, `atarijsa.cpp`, `atarimo.cpp` and
`slapstic.cpp` served as the functional oracle throughout, and to whoever
preserved the Atari SP-313 schematic package and the operator manual.

Xybots and its ROMs, artwork and manuals are © Atari Games and its successors.
This project contains no copyrighted ROM data.
