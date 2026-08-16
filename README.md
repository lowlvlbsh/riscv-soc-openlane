# PWM peripheral — RTL to GDSII on two open PDKs

A centre-aligned PWM peripheral carried from register-transfer level through
synthesis, placement, routing and physical signoff on two independent process
design kits, using an entirely open toolchain.

Alongside it: a dead-time interlock, a CORDIC Park rotator, a space-vector
modulator, a single-cycle RV32I integer core, and fixed-point control
firmware — all verified in simulation, none hardened.

Toolchain: iverilog, Yosys, OpenLane, Magic, Netgen, KLayout.

---

## Physical implementation — `pwm.v`

| Check | sky130A | gf180mcuC |
|---|---|---|
| Standard cell library | sky130_fd_sc_hd | gf180mcu_fd_sc_mcu7t5v0 |
| Clock period | 15.0 ns | 24.0 ns |
| Cells | 85 | 97 |
| Flip-flops | 10 | 10 |
| Chip area | — | 2078.85 µm² |
| Design rule check | **0 violations** | NP.8a — see [DEFECTS.md](DEFECTS.md) |
| Layout versus schematic | **0 errors** | **0 errors** |
| XOR difference | **0** | **0** |
| Total negative slack | 0.00 | 0.00 |
| Worst setup slack | +10.50 | +16.76 |
| Worst hold slack | +0.33 | +0.74 |

Both configurations differ in exactly four fields — clock period, PDK, cell
library, top routing layer. Core utilisation, aspect ratio, placement density,
congestion policy and antenna handling are identical, so any difference in
outcome is attributable to the process.

**Pinned revisions**

```
OpenLane    ff5509f65b17bfa4068d5336495ab1718987ff69
open_pdks   0fe599b2afb6708d281543108caf8310912f54af
```

The flip-flop count is identical across processes, as expected — sequential
elements come from the RTL, not the technology. The 14% difference in
combinational cells reflects what each library offers: the sky130 mapping uses
complex and-or-invert cells, the gf180 mapping uses simpler gates plus more
inverters.

---

## Simulation — verified, not hardened

| Module | Function | Verification |
|---|---|---|
| `rv32i_core.v` | Single-cycle RV32I integer core | Runs a GCC-compiled image; 14 outputs checked against independently known values |
| `deadtime_interlock.v` | Complementary gate pair with enforced blanking | 5000 random toggles; non-overlap invariant checked every cycle |
| `cordic_park.v` | Park rotation by shift-and-add, 14 stages, no multiplier | 360 angles against real arithmetic; worst-case error in LSB |
| `svpwm_gen.v` | Space-vector PWM by zero-sequence injection | One electrical revolution, three properties |
| `q15.h`, `tim1_pwm.c` | Saturating fixed-point PI with anti-windup; zero-HAL timer setup | Host-side test with measured recovery ratio |

The RV32I core implements LUI, AUIPC, JAL, JALR, all six branches, all five
loads, all three stores, all nine register-immediate and all ten
register-register ALU operations. It does not implement FENCE, ECALL, EBREAK,
CSR, or the M/A/F extensions.

Testbench results are recorded in `results/` where the run has been captured.
Where a threshold is stated in a testbench but the run output is not yet
recorded, that is noted rather than quoted.

---

## Reproducing

```bash
make sim        # iverilog
make formal     # SymbiYosys
make openlane PDK=sky130A
make openlane PDK=gf180mcuC
```

---

## Layout

```
rtl/         PWM, interlock, CORDIC, SVPWM
cpu/         RV32I core, program, linker script, startup
fw/          STM32F405 firmware — Q15 math, zero-HAL timer
tb/          testbenches
spice/       circuit decks
openlane/    per-PDK configuration
results/     synthesis and signoff reports
DEFECTS.md   defects found, with mechanisms
TOOLCHAIN.md exact tool and PDK revisions
```

---

## Why this exists

Most undergraduate RTL work stops at simulation or an FPGA bitstream. This
carries a design to physical signoff on two different processes, which exposes
problems simulation does not: rule-deck behaviour, library differences, and
the distinction between a design defect and a toolchain defect.

The defects file is the point. A flow producing no failures has not been
pushed hard enough to be informative.

---

## Scope

No physical measurement. No commercial EDA tool. No hardware-in-the-loop,
thermal or efficiency work. Every figure above comes from a simulator or a
toolchain report, and the report each came from is named in the repository
history.
