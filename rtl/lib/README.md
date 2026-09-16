# Vendored third-party RTL — `rtl/lib/`

Everything in this directory is **third-party code copied verbatim**, headers
intact. Nothing here is edited: Xybots-specific behaviour lives in wrapper
modules under `rtl/main` and `rtl/sound`, never in these files. `NOTICE.md`
at the repository root is the authoritative list of third-party components
and their licences; this file records where each copy came from.

| Directory | Component | Upstream | License | Wrapper |
| --- | --- | --- | --- | --- |
| `fx68k/` | fx68k, Jorge Cwik's **cycle-exact**, microcode-level 68000 — the main CPU | <https://github.com/ijor/fx68k>, commit `0602ee4627b10f301298f2673d826cdd6baa9327` (2021-02-16) | GPL-3.0-or-later (`fx68k/LICENSE`) | `rtl/main/xybots_cpu.sv` |
| `T65/` | T65 6502 soft core — the JSA I sound CPU | Daniel Wallner / Mike Johnson / Wolfgang Scherr / Morten Leikvoll, OpenCores and FPGAArcade | BSD-style, three clauses (header of `T65.vhd`) | `rtl/sound/xybots_jsa.sv` |
| `jt51/` | JT51 Yamaha YM2151 (OPM) | Jose Tejada (jotego), <https://github.com/jotego/jt51> | GPL-3.0-or-later | `rtl/sound/xybots_ym.sv` |

The T65 and jt51 trees were taken from the
[Arcade-Toobin_MiSTer](https://github.com/MiSTer-devel/Arcade-Toobin_MiSTer)
core, in the state that core builds and ships with; fx68k was fetched directly
from its upstream repository. All eight files of the fx68k repository are
present. SHA-256 of the fx68k copy, as fetched:

```
a718bb869a8a969dbdda89facd76f8621f655f8a84e7afa9551b2785eee5656a  fx68k.sv
3f22006cbdb8cb661e5f54cc779f79bb697b587c86d586103478703fca38bcce  fx68kAlu.sv
07ceb3b1fbbd74f255c2a909151fe5efd38a575ba4f6aa47c6208050d2e27326  uaddrPla.sv
9d13082be0cf4b04bff887e6da23de84f9f2ee4321f3a2576b4f9540615dece6  microrom.mem
b3998009fb10605e8468cddf22fdd799a9792ba86f449de2e30615cfcefbeece  nanorom.mem
3972dc9744f6499f0f9b2dbf76696f2ae7ad8af9b23dde66d6af86c9dfb36986  LICENSE
c96263ece59f55f759c046ca8027b67a6805e9cb31995f0a1bc175a826e15f96  README.md
2a7d451e8f7ae8683e4ec4f40a9b3adf91fb5c8b5d569c0f4cdb2b5de090811d  fx68k.txt
```

## Notes

* **fx68k loads its microcode with relative `$readmemb` paths**
  (`"microrom.mem"` and `"nanorom.mem"`, `fx68k.sv` lines 2474 and 2485). The
  files are not edited, so `files.qip` carries
  `set_global_assignment -name SEARCH_PATH rtl/lib/fx68k` and Quartus resolves
  the paths against it. A build that silently fails to find them synthesises
  an **empty** microcode ROM and boots to a black screen, so check the
  Analysis & Synthesis log: `Info (10905)` must name both `.mem` files, and the
  RAM summary must show a 1024 × 17 and a 336 × 68 ROM initialised from them.
* `T65/T65_wrap.vhd` is not upstream T65. It is a thin flat-port wrapper,
  written for the Toobin' core, that keeps the T65 `DEBUG` record internal so
  the core can be netlisted for simulation. It carries the same BSD terms as
  the core it wraps.
* `jt51/hdl/deprecated/` and `jt51/hdl/filter/` are carried so the tree
  matches upstream; only `jt51/hdl/jt51.qip` is registered for synthesis, and
  it lists neither directory.
* **There is no POKEY core here, on purpose.** The Xybots JSA I board has the
  POKEY and TMS5220 sockets unpopulated, so the POKEY that the Toobin' core
  vendors is not copied at all. What the 6502 reads from the empty socket is
  modelled in `rtl/sound/xybots_jsa_bus.sv`.
* The vendored cores are not held to this repository's warning-clean lint
  bar. In the Quartus build, jt51's "has no driver or initial value" warnings
  on the `lfo_lut` / `sinetable` / `explut` write ports are upstream and
  expected.
