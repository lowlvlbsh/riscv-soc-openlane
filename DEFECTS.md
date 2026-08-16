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

**What is not yet established.** The decisive question is whether the
violating geometry belongs to foundry-supplied standard or filler cells rather
than to design-authored geometry. That requires opening the layout at the
reported coordinates and identifying the cell:

```bash
klayout runs/gf180c_run/results/signoff/pwm.gds
# navigate to (7.05, 88.13) µm
```

Until that is done, the honest statement is: a rule violation appearing on one
process and not the other, on a layout passing every other check, with a cause
consistent with a deck or library issue but not confirmed.

**Toolchain revision.** open_pdks `0fe599b2afb6708d281543108caf8310912f54af`,
pinned via Volare.

**Status.** Open. Investigation step identified above.

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
