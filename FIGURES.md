# ACHP-ADI Figure Descriptions

**Version 4.2**
Date: 2026-03-23

These are text descriptions of the architectural diagrams referenced in the specification. They are intended for reproduction as formal figures in published materials.

---

## Figure 1 — System Architecture Pipeline

A block diagram showing four horizontally arranged layers connected by directed arrows indicating data flow.

**Left to right:**

Block 1: **Inference Engine**. Label: "Produces probability distribution P over n candidates." Output arrow labeled "P" points right to Block 2.

Block 2: **Authority-State Derivation Layer**. Label: "Computes p_max, p_second, decision margin. Partitions into Strong / Weak / Conflict regions." Two output arrows diverge from Block 2 to Block 3:
- Upper arrow labeled "Attested Execution-Authority State (Strong or Weak regions)"
- Lower arrow labeled "Negative Attestation Artifact (Conflict region)"

Block 3: **Action Dispatch Interface (ADI)**. Label: "State-driven emitter. No input path from inference data. Sole path to action executors." Two output arrows:
- Upper arrow labeled "Action Command" points right to Block 4 (for attested states).
- Lower arrow labeled "Fallback Signal" points down to a Fallback Handler box (for negative attestation).
- A vertical arrow labeled "Append to Chain" points down to a "Tamper-Evident Log" cylinder.

Block 4: **Action Executors**. Label: "Receive commands exclusively from ADI."

**Blocked paths** (shown as dashed lines with X marks):
- No direct path from Inference Engine to Action Executors.
- No direct path from Inference Engine to ADI.
- No path from any component to Action Executors that bypasses ADI.
- No path from Authority Layer to Action Executors that bypasses ADI.

---

## Figure 2 — Decision Region Partitioning

A two-dimensional plot with axes:
- Horizontal axis: Decision Margin (p_max - p_second), range 0 to 1.
- Vertical axis: p_max, range 0 to 1.

A horizontal dashed line at p_max = T_decide divides the plot into upper and lower regions.

In the lower region (p_max < T_decide), a vertical dashed line at Decision Margin = T_sep divides it into left and right sub-regions.

**Three labeled regions:**

Region 1 (upper): **Strong Decision**. Condition: p_max >= T_decide. Shaded in green or light fill. Label: "Emit argmax with full confidence."

Region 2 (lower-right): **Weak/Flagged Decision**. Condition: p_max < T_decide AND Decision Margin >= T_sep. Shaded in yellow or medium fill. Label: "Emit argmax with low-confidence flag."

Region 3 (lower-left): **Conflict (UNRESOLVED_ATTESTED)**. Condition: p_max < T_decide AND Decision Margin < T_sep. Shaded in red or dark fill. Label: "No decision object instantiated. Negative attestation emitted. All action paths blocked."

Boundary annotations:
- At p_max = T_decide: ">= binds to Strong Decision"
- At Decision Margin = T_sep: ">= binds to Weak Decision"

Example point plotted: P = [0.45, 0.42, 0.08, 0.05] with p_max = 0.45, margin = 0.03, clearly within the Conflict region with T_decide = 0.90, T_sep = 0.15.

---

## Figure 3 — Conflict Flow vs. Action Flow

A flowchart showing two diverging paths from the Authority-State Derivation Layer.

**Entry point:** "Authority Layer receives probability distribution P."

**Decision diamond:** "p_max >= T_decide OR (p_max < T_decide AND margin >= T_sep)?"

**Left path (Action Flow — Yes):**
1. "Authority Layer issues Attested Execution-Authority State"
2. Arrow to ADI: "ADI receives attested state"
3. "ADI maps state to action command via fixed correspondence"
4. "ADI dispatches Action Command to Action Executor"
5. "Action Executor performs action"

**Right path (Conflict Flow — No: p_max < T_decide AND margin < T_sep):**
1. "Authority Layer enters Conflict condition"
2. "No decision object instantiated at any computation stage"
3. "Authority Layer generates Negative Attestation Artifact"
4. "Artifact signed by cryptographic module"
5. Arrow to ADI: "ADI receives Negative Attestation Artifact"
6. "ADI blocks ALL action paths for this input"
7. "ADI appends signed artifact to tamper-evident chain"
8. "ADI invokes lane-defined fallback behavior"
9. "System enters fallback mode"

**Convergence point:** "Await next evaluation cycle."

**Key annotations:**
- On the Conflict path, between steps 1 and 2: "No executable, classifiable, or derivable decision representation is formed, materialized, stored, transmitted, or recoverable — even momentarily — including in registers, caches, and speculative execution states."
- On the ADI block in both paths: "No action without prior attested execution-authority state from Authority Layer."
- On the blocked-paths annotation: "No execution path, side-channel, duplicate pipeline, shadow process, or alternative signal route permits inference output to reach action executor without traversal through Authority Layer and ADI."

---

## Figure 4 — Execution Isolation Model

A diagram showing two isolated domains separated by a hardware boundary.

**Left domain: Inference Domain**
- Contains: Inference Engine, probability distribution P, intermediate computations (activations, logits, pre-softmax values), model weights.
- Memory region: "Inference Memory Region — mapped only in Inference Engine's page tables."
- Connected to Authority-State Derivation Layer via a one-directional arrow labeled "P (read access, time-limited)."

**Center: Authority-State Derivation Layer**
- Straddles the boundary. Has read access to Inference Domain (revoked after computation). Has write access to Execution Domain (to emit tokens).
- Performs: "Compute p_max, p_second, margin. Compare to T_decide, T_sep. Determine region."
- Output to right: "Authority-State Token (typed, signed, contains no probability data)" — or — "Negative Attestation Artifact."
- After emission: "Zeroization of inference state: three-pass memory overwrite, register clearing (XOR), speculation barrier (LFENCE/CSDB), cache invalidation (CLFLUSH/DC CIVAC)."

**Right domain: Execution Domain**
- Contains: Action Dispatch Interface (ADI), action command structures, fixed correspondence table (ROM).
- Memory region: "Execution Memory Region — mapped only in ADI's page tables. Inference memory region ABSENT from this address space."
- ADI composition (two alternatives shown):
  - Category A: "Fixed-function logic block. No ALU, no FPU, no instruction decoder. Token type register + ROM lookup + output register."
  - Category B: "Software process on general-purpose processor. Inference memory not mapped. MMU generates hardware fault on any access attempt."
- Output: "Action commands to Action Executors" — or — "Fallback signal."

**Hardware boundary between domains:**
- Labeled: "Hardware-enforced memory isolation. MMU page table separation. IOMMU DMA restriction. TEE/enclave encryption (optional)."
- Annotation: "No system call, IPC, shared memory, DMA, or any other mechanism bridges this boundary from Execution Domain to Inference Domain."
- Annotation at boundary: "Any attempted access from ADI to inference memory triggers non-maskable, non-recoverable hardware page fault."

---

*End of figure descriptions.*
