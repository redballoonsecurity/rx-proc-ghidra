# rx-proc-ghidra

This Ghidra module adds support for disassembling and decompiling Renesas RX architecture binaries.

## Languages

| Language ID      | Instruction set | Example devices                         |
|------------------|-----------------|-----------------------------------------|
| `RX:LE:32:RXv2`  | RXv1 / RXv2     | RX62T, RX63N, RX65N, RX66T, ...         |
| `RX:LE:32:RXv3`  | RXv3            | RX72N, RX72M, RX66N, RX671, ...         |

RXv3 is a superset of RXv2, so ELF files with `e_machine == EM_RX` (173) are imported as
`RX:LE:32:RXv3` by default. Choose `RX:LE:32:RXv2` manually to restrict decoding to RXv2.

Source files (`data/languages`):

* `rx62t.slaspec` / `rx62t.sinc` - registers and the RXv1/RXv2 instruction set, including the
  DSP/accumulator instructions (`MULHI`, `MACLO`, `EMACA`, `MSBHI`, `MVFACGU`, `RACL`, ...).
* `rxv3.slaspec` / `rxv3.sinc` - RXv3 additions: the double-precision FPU (`DR0`-`DR15`,
  `DPSW`/`DCMR`/`DECNT`/`DEPC`, `DMOV`, `DADD`..`DDIV`, `DCMPcm`, `DTOI`, `ITOD`, `DPUSHM`, ...),
  three-operand `XOR`, `BFMOV`/`BFMOVZ` and `SAVE`/`RSTR`. `rxv3.slaspec` defines `RXV3` and
  includes `rx62t.slaspec`, which then pulls in `rxv3.sinc`.

* `rx.dwarf` / `rxv3.dwarf` - DWARF register mappings (0 = SP, 1-15 = R1-R15, 16 = PC, and for
  RXv3 32-47 = DR0-DR15) used by Ghidra's DWARF importer for function signatures and parameters.

Reference: *RX Family RXv3 Instruction Set Architecture User's Manual: Software* (R01US0316EJ).

## DWARF notes (CC-RX)

* Set *Analysis > DWARF > Default Calling Convention* to `__fastcall` before analyzing. Otherwise
  Ghidra leaves functions without parameters at an `unknown` calling convention (decompiler warning
  "Unknown calling convention -- yet parameter storage is locked").
* Local variables are not imported from CC-RX debug info: CC-RX's location lists for locals start
  after the function prologue, and Ghidra's DWARF importer only looks up locals at the function entry
  address. Function names, signatures, parameters, types and globals are imported.

## Building

`gradle -PGHIDRA_INSTALL_DIR=<ghidra_install_dir>`

The extension zip is written to `dist/`. To check the SLEIGH alone:
`<ghidra_install_dir>/support/sleigh data/languages/rxv3.slaspec data/languages/rxv3.sla`

## Installation

In Ghidra: *File > Install Extensions > +*, pick the zip from `dist/`, and restart Ghidra. Or unzip it
into `<user settings dir>/Extensions`.
