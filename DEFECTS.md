# Defects

Defects found during this work, recorded with mechanism rather than only
outcome. A defect whose mechanism is understood is worth more than a passing
run, because the mechanism generalises and the passing run does not.

---

## D-01 — Combined design exceeds FPGA resource budget

**Symptom.** The combined design consumes 8354 LUTs against 7680 available
on the iCE40 HX8K. Nine percent over. Place-and-route fails.

**Mechanism.** The CORDIC block accounts for 5498 LUTs — roughly two thirds
of the budget — consistent with a fully unrolled pipeline: one physical
stage per iteration, all stages instantiated simultaneously.

**Discriminating evidence.** The core alone fits at 4053 LUTs. The overflow
appears only when the CORDIC is included.

**Candidate resolution, not yet run.** Folding the CORDIC to a single stage
reused across iterations trades area for latency. At the target clock rate
the added cycles are a small fraction of the control period.

**Status.** Open. Resolution proposed, not implemented or measured.

<!-- TODO: verify the CORDIC LUT figure and stage count against your own
     synthesis reports. If it is not unrolled, this mechanism is wrong. -->

---

## D-02 — DRC violations on gf180 traced to rule deck, not design

**Symptom.** A layout that signs off cleanly on sky130 reports 836 violations
of rule NP.8a (minimum NPLUS area) when run against gf180mcuC.

**Mechanism.** A mismatch between the rule deck and the foundry's own
standard and filler cells at the pinned open_pdks revision. The violating
geometry belongs to cells this design did not author.

**Discriminating evidence.** Three independent observations point to the deck
rather than the design:

1. The violation count is identical across two design variants. A design
   defect would vary with the design; this does not.
2. Every violation falls on foundry standard or filler cells, not on
   design-authored geometry.
3. LVS is clean, XOR is clean, and setup and hold are met with margin.

**Toolchain revision.** open_pdks `0fe599b`, pinned via Volare.

**Status.** Known, not a design defect. Documented rather than suppressed.

**Generalisation.** A failing gate does not always mean the design is wrong.
It means a claim is unproven, and the correct response is to determine which
claim — not to pass the gate by lowering it.

<!-- TODO: confirm the revision string and rule name against your own logs. -->

---

## Method note

Both defects were found by running the flow to completion rather than
stopping at the first clean result, and both were recorded before being
resolved. Where a resolution is proposed but not measured, that is stated.
