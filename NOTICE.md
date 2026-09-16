# NOTICE — third-party components and attributions

This repository's own RTL and documentation are **GPL-3.0-or-later** (see
`LICENSE`). The tree is not uniformly GPL-3.0: this file names every
third-party component compiled into the bitstream, its authors and its
licence, with the paths as they exist here. File headers are preserved in
place and are authoritative where they differ from this summary.

## Compiled into the bitstream

### Vendored, whole files present in the tree

| Component | Author(s) | License | Location |
|---|---|---|---|
| MiSTer framework (`sys_top`, `hps_io`, `arcade_video`, `video_freak`, `ascal`, PLL reconfiguration, …) | Till Harbaum, Alexey Melnikov (sorgelig), MiSTer-devel contributors | GPL-2.0-or-later or GPL-3.0-or-later, per file header | `sys/` |
| fx68k — cycle-exact, microcode-level 68000 (**the main CPU**), upstream commit `0602ee46`, whole repository | Jorge Cwik (ijor), <https://github.com/ijor/fx68k> | GPL-3.0-or-later | `rtl/lib/fx68k/` |
| T65 6502 soft core (the JSA I sound CPU) | Daniel Wallner, Mike Johnson, Wolfgang Scherr, Morten Leikvoll (OpenCores / FPGAArcade) | BSD-style, three clauses; the "redistributions in synthesized form" clause is satisfied by this notice | `rtl/lib/T65/` |
| `T65_wrap.vhd` — flat-port wrapper around T65, written for the Arcade-Toobin_MiSTer core | Arcade-Toobin_MiSTer authors | Same terms as T65 | `rtl/lib/T65/T65_wrap.vhd` |
| jt51 YM2151 (OPM) FM core | Jose Tejada (jotego), <https://github.com/jotego/jt51> | GPL-3.0-or-later | `rtl/lib/jt51/` |
| `analog_hsize.sv` — analog-VGA horizontal resampler (CRT H-Size / H-Position), from Arcade-Raiden_MiSTer by way of Arcade-Toobin_MiSTer; two small adaptations are listed in its header | Umberto Parisi (rmonic79) | GPL-3.0-or-later | `rtl/video/analog_hsize.sv` |
| Altera/Intel PLL megafunction instances | Intel Corporation (generated IP) | Intel FPGA IP licence, as generated | `rtl/pll*`, `sys/pll*` |

Everything under `rtl/lib/` is byte-for-byte upstream; see `rtl/lib/README.md`
for the provenance of each copy.

### Derived work — Xybots RTL written from, or structurally based on, GPL-3.0 cores

Each of these files names its origin in its header. All are GPL-3.0-or-later,
the licence this repository ships under.

| Xybots file(s) | Derived from |
|---|---|
| `rtl/main/xybots_eeprom_2804.sv`, `xybots_nvram_io.sv`, `xybots_main_bus.sv` (structure) | [Arcade-Toobin_MiSTer](https://github.com/MiSTer-devel/Arcade-Toobin_MiSTer) |
| `rtl/mem/xybots_sdram.sv`, `xybots_sdram_loader.sv`, `xybots_rom_loader.sv`, `xybots_gfx_mem.sv`, `xybots_prog_rom.sv` | Arcade-Toobin_MiSTer |
| `rtl/sound/*.sv` (the JSA I board, without POKEY and TMS5220) | Arcade-Toobin_MiSTer `toobin_jsa*.sv`, `toobin_sound_comm.sv`, `toobin_ym.sv` |
| `rtl/xybots_core.sv`, `Arcade-Xybots.sv`, `Xybots.sdc` | Arcade-Toobin_MiSTer glue and timing-constraint patterns |
| `rtl/video/xybots_analog_adjust.sv` | Arcade-Toobin_MiSTer, which follows Arcade-Raiden_MiSTer |
| `rtl/video/xybots_syngen.sv`, `xybots_mo_linebuf.sv` | Behavioural references for the same Atari customs in [Arcade-Gauntlet_MiSTer](https://github.com/MiSTer-devel/Arcade-Gauntlet_MiSTer) (`SYNGEN.vhd`, `LINEBUF.vhd`, `PROM_5E.vhd`) by d18c7db |

`rtl/main/xybots_slapstic.sv` is **not** derived from the Gauntlet / Atari
System 1 `SLAPSTIC.vhd`: it was written from the SP-313 schematic (sheet 2)
and checked against the state machine in MAME's `slapstic.cpp`.

## Reference material (read for behaviour, never compiled or redistributed)

MAME **0.288** (BSD-3-Clause, copyright Aaron Giles and the MAME contributors)
was used as a functional oracle: `xybots.cpp`, `atarijsa.cpp`, `atarimo.cpp`
and `slapstic.cpp`. The Atari SP-313 schematic package, the Xybots operator
manual and all ROM images were inputs to the design and are **never committed
or redistributed** by this project.

## Trademarks and content

This project distributes no ROM data and no copyrighted artwork. *Xybots* is a
trademark of its rights holders. Use only with software you are legally
entitled to.
