# RV32I SoC — RTL to GDSII on open PDKs

A RISC-V RV32I core and a PWM peripheral, taken from RTL through synthesis,
place-and-route, and physical signoff on two open process design kits.

Built with an entirely open toolchain: iverilog, Yosys, nextpnr, SymbiYosys,
OpenLane, Magic, Netgen, KLayout.

**Status:** functional core verified against the RISC-V compliance suite;
PWM peripheral hardened to clean signoff on sky130 and gf180. Two defects
found and documented — see [DEFECTS.md](DEFECTS.md).

---

## Results

### RV32I core — functional verification

| Metric | Result |
|---|---|
| riscv-tests compliance | 14 / 14 pass |
| Test program: sum 1..100 | 5050 (correct) |
| Test program: fib(20) | 6765 (correct) |
| Execution length | 693 cycles |

### FPGA implementation — Lattice iCE40 HX8K

| Metric | Result |
|---|---|
| LUTs used (core) | 4053 |
| LUTs available | 7680 |
| Toolchain | Yosys + nextpnr + icestorm |

### ASIC signoff — PWM peripheral

| PDK | DRC | LVS | Status |
|---|---|---|---|
| sky130A | clean | clean | signed off |
| gf180mcuC | see DEFECTS.md | clean | see DEFECTS.md |

PDK revisions pinned via Volare and recorded in TOOLCHAIN.md so runs are
reproducible.

### Verification

- **Mutation testing** on the core testbench — faults injected into the RTL
  to measure whether the testbench detects them, rather than inferring
  coverage from passing tests.
- **Formal verification** with SymbiYosys — bounded model checking and
  induction on selected properties.

<!-- TODO: add your mutation testing numbers.
     e.g. "N mutants injected, M detected, detection rate X%" -->

---

## Reproducing

```bash
make sim        # simulation
make formal     # SymbiYosys
make fpga       # Yosys + nextpnr, iCE40 HX8K
make openlane PDK=sky130A
make openlane PDK=gf180mcuC
```

<!-- TODO: adjust to your actual build targets -->

---

## Repository layout

```
rtl/         core and peripheral RTL
tb/          testbenches, mutation testing harness
formal/      SymbiYosys property checks
synth/       Yosys + nextpnr scripts, iCE40 constraints
openlane/    per-PDK configuration
results/     synthesis reports, signoff logs
DEFECTS.md   defects found, with mechanisms
TOOLCHAIN.md exact tool versions
```

---

## Why this exists

Most undergraduate RTL work stops at simulation, or at an FPGA bitstream.
This project carries a design through to physical signoff on two different
processes, which exposes a class of problem simulation does not: resource
limits, rule-deck behaviour, and the difference between a design defect and
a toolchain defect.

The defects file is the point. A flow that produces no failures has not been
pushed hard enough to be informative.

---

## License

<!-- TODO: MIT or Apache-2.0 -->
