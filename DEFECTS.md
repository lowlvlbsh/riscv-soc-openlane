# Defects

Defects found during this work, recorded with mechanism rather than only
outcome, and with the evidence that distinguishes the stated cause from the
alternatives. Where the evidence supports a conclusion but does not establish
it, that is said.

---

## D-01 — gf180 design rule violations on a layout that is otherwise clean

**Symptom.** A layout that reports a design rule count of zero on sky130A
reports violations of rule NP.8a — minimum NPLUS area, 35 µm² — when the same
RTL is run through the same flow against gf180mcuC.

Report path: `runs/gf180c_run/reports/signoff/drc.rpt`
Geometry entries: 836

**Evidence pointing away from a design defect.** Three independent
observations:

1. The violation geometry is identical across two independent gf180 runs
   (`gf180_run` and `gf180c_run`), with matching leading coordinates. A
   placement-dependent design defect would be expected to vary.
2. Every other signoff check on the same layout passes. Layout versus
   schematic reports zero errors. XOR reports zero differences. Setup and hold
   are met with positive margin (+16.76 and +0.74).
3. The identical RTL, flow, utilisation and density settings produce a design
   rule count of zero on sky130A.

**Mechanism, established.** The implant layer is not drawn in any library
cell. A layer census across all 229 cells in `gf180mcu_fd_sc_mcu7t5v0`
returns nwell, pwell, metal1, polysilicon and diffusion layers (`mvndiff`,
`mvpdiff` and their contacts) — and no implant layer at all.

The N-plus geometry evaluated by the design rule checker is therefore
generated at GDS-write time by Magic's technology file. It exists in neither
the design nor the cell library source. It is produced by one component of
open_pdks `0fe599b2afb6708d281543108caf8310912f54af` and rejected by another
component of the same pinned revision.

Method:

```bash
P=~/.volare/ciel/gf180mcu/versions/0fe599b2.../gf180mcuD/libs.ref/gf180mcu_fd_sc_mcu7t5v0/mag
grep -ho '<< .* >>' $P/*.mag | sort | uniq -c | sort -rn
```

Returns 229 counts each for nwell, pwell, metal1, labels and properties;
219 each for polysilicon, polycontact and the medium-voltage diffusion and
transistor layers; and no implant layer.

Supporting observation: `gf180mcu_fd_sc_mcu7t5v0__fill_1`, the narrowest
filler, has a fixed bounding box of 56 by 392 internal units — roughly
0.56 by 3.92 um, about 2.2 um2 in total. The rule requires 35 um2. No
filler cell of that size could satisfy it under any circumstances, and it
does not carry the layer in question in the first place.

**Toolchain revision.** open_pdks `0fe599b2afb6708d281543108caf8310912f54af`,
pinned via Volare.

**Status.** Mechanism established. Neither a design defect nor a library
defect: a geometry-generation rule and a rule deck disagreeing within one
pinned PDK revision.

**Generalisation.** A failing gate does not always mean the design is wrong.
It means a claim is unproven, and the correct response is to determine which
claim — not to pass the gate by lowering it.

---

## D-02 — Undriven output port reported by synthesis

**Symptom.** The sky130 synthesis check pass reports:

```
Warning: Wire pwm.\pwm_out is used but has no driver.
Found and reported 1 problems.
```

**Why it matters.** An output port used but undriven normally indicates a
missing assignment or a name mismatch between the port declaration and the
logic intended to drive it. An undriven net is optimised away and leaves no
downstream trace, so the flow completing and timing closing does not resolve
it.

**Status.** Open, not investigated.

---

## Method note

Both entries were recorded before being resolved. Where a resolution is
proposed but not measured, and where evidence supports a conclusion without
establishing it, that is stated rather than smoothed over.
