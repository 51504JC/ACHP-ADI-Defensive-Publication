# ACHP-ADI: Deterministic Handling and Attestation of Unresolved Decision States in Probabilistic AI Systems

**A Technical Whitepaper**
Author: Richard Davis
Date: 2026-03-23
Version: 4.2

---

## 1. Problem Statement

Probabilistic inference engines — neural network classifiers, Bayesian ensemble systems, sensor fusion pipelines, risk scoring models — produce probability distributions over candidate outputs. Downstream systems act on these distributions by selecting the highest-ranked candidate (argmax) and dispatching the corresponding action: a robotic actuator moves, an API responds, a treatment is applied, a policy is enforced.

This inference-to-action pipeline has no structural mechanism for handling ambiguity. When the model's output distribution is ambiguous — when the top candidates are closely spaced and no single output commands sufficient confidence — the system selects one output and dispatches the action as if the model were certain. The result is silent overconfidence: the downstream consumer has no indication that the decision was marginal. The system log shows an input and an output, but not that two or more candidates were nearly tied. No structural mechanism exists to prevent action dispatch when the system's own metrics indicate insufficient separation.

This paper describes ACHP-ADI, a control architecture that addresses this problem through a fundamentally different approach: when decision ambiguity exceeds defined thresholds, no decision object is ever instantiated within the system, and the system instead generates a cryptographic attestation of that non-existence.

---

## 2. Failure of Prior Systems

### 2.1 Threshold-Based Classification Rejection

Chow (1970) established that a classifier may optimally reject inputs when the maximum posterior probability falls below a cost-derived threshold. This work was extended by Geifman and El-Yaniv (2017, 2019) with SelectiveNet and the broader selective classification literature, which introduced learned rejection thresholds and margin-based abstention mechanisms.

These methods share a structural property: the rejection is internal to the classifier. The classifier outputs "reject" as one possible outcome within its own output space. The rejection is a decision — a specific, instantiated output that the classifier produces. No structured attestation record is generated. No cryptographic signing occurs. No hash-chained audit log is maintained. No downstream action dispatch interface is gated on the rejection. No non-override invariant is enforced at the protocol level. The rejection is a classifier behavior, not a system-level control state.

Critically, in all rejection-based systems, a decision object exists. The "reject" output is that object. It is produced by the inference engine, transmitted downstream, and consumed by the next system component. The system acts on a decision — the decision to reject.

### 2.2 Certified Control

Jackson et al. (MIT CSAIL / Toyota Research Institute, 2019-2020) proposed a certified control architecture for autonomous vehicles in which a main controller generates a structured artifact containing a proposed action and supporting sensor evidence. An independent certifier evaluates this artifact against safety criteria. If evaluation passes, the action is transmitted to actuators. If evaluation fails, a safe fallback controller takes over. The certifier has prioritized access to actuators, achieving structural non-bypassability.

This architecture teaches three relevant concepts: structural separation of the decision engine from an independent authority; non-bypassable gating of action execution; and fallback on rejection. However, the certified control architecture requires the existence of a decision object — the structured artifact containing the proposed action. The certifier evaluates physical safety conditions (obstacle geometry, lane compliance), not the internal structure of a probability distribution. When the certifier rejects, no structured attestation record is produced, no hash-chained audit trail is maintained, and no mechanism exists for deterministic replay of the rejection conditions.

The certified control architecture is fundamentally a gate: a decision object is produced, evaluated, and either passed or blocked. The decision object exists throughout the process.

### 2.3 Cryptographic Audit Logging

Blockchain-based and hash-chained logging systems for AI inference decisions (2021-2025) create tamper-evident records of model outputs and actions dispatched. These systems log decisions that were made. They record the input, the model, and the selected output. They are observational and post-hoc: they do not refuse to log a decision when ambiguity is present, do not gate action execution on the content of the log, and do not prevent forced decisions from occurring. The logging is an overlay on an existing decision pipeline, not a control mechanism that governs the pipeline.

### 2.4 Attested Execution and TEE Systems

Trusted Execution Environment (TEE) attestation systems (Intel SGX, AMD SEV-SNP, ARM TrustZone, IETF RATS) require cryptographic attestation as a precondition for code execution or secret release. The attestation token is a signed, structured record that must be verified before the relying party permits access.

These systems attest a positive condition: that the execution environment is valid, that code measurements match expected values, that the platform is trustworthy. They do not attest a negative condition: that a decision could not be made. The attestation domain is environment integrity, not probabilistic decision quality. TEE systems do not analyze probability distributions, compute decision margins, or produce attestation records of decision ambiguity.

### 2.5 Common Structural Limitation

All prior systems require a decision object — a proposed action, a class label, a reject output, a log entry — to exist within the system. Their mechanisms operate on this object: evaluating it, filtering it, logging it, gating it. The decision object is the operative element. Without it, none of these systems can function.

---

## 3. Architecture Overview

ACHP-ADI comprises four structural layers arranged in a linear, non-bypassable pipeline. No alternate execution path permits an inference engine output to reach an action executor without traversal through the Authority-State Derivation Layer and the Action Dispatch Interface.

**Layer 1 — Inference Engine.** A probabilistic model that produces a probability distribution P over n candidate classes or actions (n >= 2). The Inference Engine has no direct connection to any action executor. It does not determine whether its own output is of sufficient quality for action authorization.

**Layer 2 — Authority-State Derivation Layer.** Receives P from the Inference Engine. Computes decision metrics: the maximum probability (p_max), the second-largest probability (p_second), and the decision margin (p_max - p_second). Applies fixed, lane-configurable thresholds (T_decide, T_sep) to partition P into one of three mutually exclusive, exhaustive regions. For every evaluation, produces exactly one of two outputs: an attested execution-authority state (for Strong or Weak regions) or a negative attestation artifact (for Conflict region).

**Layer 3 — Action Dispatch Interface (ADI).** The sole structural pathway between the Authority Layer and all action executors. Operates as a state-driven emitter: produces action commands exclusively by mapping attested execution-authority states to actions through a fixed correspondence. Has no input path from inference engine outputs. Is structurally absent the computational primitives necessary to derive actions from inference data. Upon receiving a negative attestation artifact, blocks all action paths and invokes fallback.

**Layer 4 — Action Executors.** Receive commands exclusively from the ADI. Have no direct access to the Inference Engine or Authority Layer.

---

## 4. Topological Non-Instantiation

The defining architectural property of ACHP-ADI is topological non-instantiation of decision objects.

When the Authority-State Derivation Layer determines that a probability distribution enters the Conflict region (p_max < T_decide AND decision margin < T_sep), the system enters a conflict condition in which no decision object is ever instantiated — even momentarily. This prohibition applies across all stages of computation, including transient, intermediate, cached, register-level, speculative, and ephemeral machine states. No executable, classifiable, or derivable decision representation is ever formed, materialized, stored, transmitted, or recoverable within any component of the system.

This is not a security measure applied to a decision that exists. It is a structural property of the machine in which the relevant class of data is rendered non-existent by design. The system constrains the set of representable machine states such that executable decision representations cannot be formed under the conflict condition.

The non-instantiation property is enforced through the convergence of multiple structural mechanisms:

**Memory isolation.** Inference-domain data resides in memory regions that are not addressable by the execution interface. The processor's memory management unit (MMU) enforces separation at the hardware level. Page table entries for inference memory are absent from the execution interface's address space. Any attempted access generates a non-maskable hardware fault.

**Computational incapability.** The execution interface either (a) lacks hardware arithmetic and statistical processing units (implemented as a fixed-function logic block without ALU, FPU, or instruction decoder), or (b) operates within an address space from which all inference-domain data is structurally excluded. In both cases, no computational pathway from inference data to action emission exists within the execution interface's operational domain.

**One-way transformation.** Authority-state tokens are produced via a cryptographically one-way, non-invertible transformation. The tokens contain no probability distribution, no inference output, and no data from which these could be recovered. Reconstruction of inference data from the token is computationally infeasible.

**State destruction.** Upon conflict, all transient computation states associated with inference are irreversibly discarded through secure zeroization, including processor registers (via explicit XOR clearing), speculative execution traces (via LFENCE/CSDB barriers), and cache hierarchies (via CLFLUSH/DC CIVAC invalidation). No residual or reconstructable decision representation remains accessible.

This is a topological constraint on machine operation, not a post hoc control over data that has already been created.

---

## 5. Negative Attestation

When the conflict condition is entered, the system generates a negative attestation artifact — a cryptographically signed, structured record documenting that no decision could be produced for the corresponding input.

The negative attestation is structurally distinct from all prior attestation models:

- TEE remote attestation attests a positive condition (environment is valid). Negative attestation attests a negative condition (decision could not be made).
- Certified control certificates attest that a proposed action is safe. Negative attestation attests that no action was proposed.
- Audit logs record decisions that were made. Negative attestation records that a decision was not made.

The negative attestation artifact contains: the full probability distribution P; the computed p_max, p_second, and decision margin; the applied thresholds T_decide and T_sep; a cryptographic hash of the raw input data; timestamps (monotonic and UTC); device and lane identifiers; chain linkage (hash of preceding receipt); status indicator UNRESOLVED_ATTESTED; cryptographic signature; and signature algorithm identifier.

The artifact constitutes the exclusive persistent representation of the system state for the corresponding input. Upon its generation, all other copies of inference data are destroyed. No parallel, duplicate, cached, or derived state representation for the same input may exist, persist, or influence the execution interface.

---

## 6. Execution Gating

The Action Dispatch Interface (ADI) enforces a structural invariant: no action may be dispatched without prior issuance of an attested execution-authority state by the Authority-State Derivation Layer.

The ADI is not a conditional gate. A conditional gate receives two inputs: the data to be gated and the control signal determining whether to pass it. The ADI receives one input: an attested execution-authority state (or a negative attestation artifact). It has no input path that receives, carries, or is derived from any decision object, candidate action, inference engine output, or probability distribution. No data path exists from inference outputs to action emission within or through the ADI.

The ADI operates as a state-driven emitter: it maps attested states to action commands through a fixed correspondence table. When it receives a negative attestation artifact, it blocks all action paths for the corresponding input and invokes the lane-defined fallback behavior. The blocking occurs prior to and instead of any action dispatch.

The structural absence of computational primitives is enforced through two independently sufficient mechanisms:

- **ISA-level incapability**: The ADI is implemented as a fixed-function logic block (FPGA or ASIC) containing no general-purpose ALU, no FPU, no instruction decoder, and no hardware capable of computing statistical aggregates. The incapability is physical and immutable.

- **Address-space structural exclusion**: The ADI executes within a process address space from which all inference-domain data is absent. The MMU generates hardware faults on any attempted access. Even though the underlying processor may possess arithmetic instructions, those instructions cannot operate on inference data because the data cannot be loaded into any register accessible to the ADI.

Both mechanisms achieve the same structural result: no computational pathway from inference-domain data to action emission exists within the ADI's operational domain.

---

## 7. Decision Region Partitioning

The Authority-State Derivation Layer partitions the probability simplex into three mutually exclusive, exhaustive regions using two fixed thresholds:

**T_decide** (confidence threshold): Constrained to the open interval (0.5, 1). Defines the minimum p_max for a Strong Decision.

**T_sep** (separation threshold): Constrained to the open interval (0, 1 - T_decide). Defines the minimum decision margin for a Weak/Flagged Decision when p_max < T_decide.

**Region 1 — Strong Decision:** p_max >= T_decide. The Authority Layer issues an attested execution-authority state authorizing action for the top-ranked candidate.

**Region 2 — Weak/Flagged Decision:** p_max < T_decide AND decision margin >= T_sep. The Authority Layer issues an attested execution-authority state with a mandatory reduced-confidence indicator.

**Region 3 — Conflict (UNRESOLVED_ATTESTED):** p_max < T_decide AND decision margin < T_sep. No execution-authority state is issued. Negative attestation artifact is generated. All action paths are blocked.

Proof of exhaustiveness: For any distribution P with n >= 2, p_max has a definite value. Either p_max >= T_decide (Strong) or p_max < T_decide. In the latter case, the decision margin is a non-negative real number, so either it is >= T_sep (Weak) or < T_sep (Conflict). The regions are pairwise disjoint by construction.

Boundary assignments: p_max = T_decide assigns to Strong Decision (>= operator). Decision margin = T_sep assigns to Weak Decision (>= operator).

---

## 8. Replayability and Audit

The negative attestation artifact enables deterministic replay of the decision boundary: an independent party, using only the data contained in the artifact and without access to the inference engine, model weights, raw input data, or any internal system state, can:

1. Extract the recorded probability distribution P and thresholds T_decide, T_sep.
2. Recompute p_max, p_second, and the decision margin from P.
3. Confirm that p_max < T_decide and decision margin < T_sep.
4. Conclude that the conflict region was correctly entered.

This replay is deterministic: given the same recorded distribution and thresholds, the result is always the same. It is distinct from event replay (replaying a sequence of actions) and from chain integrity verification (verifying hash linkage and signatures).

The full audit process comprises: chain integrity verification (signature and hash linkage checks across the append-only log); deterministic replay of the decision boundary for each artifact; and confirmation that no action dispatch occurred for inputs whose evaluations entered the conflict region.

Replay verification confirms internal consistency of recorded data. It does not independently verify that the recorded thresholds were the thresholds applied at decision time. This limitation is explicitly disclosed. Binding decision-time thresholds to receipt content requires trusted execution or hardware attestation, which is outside the scope of the ACHP-ADI core protocol.

---

## 9. Invariants

A conforming implementation of ACHP-ADI satisfies the following invariants:

**INV-1 (No Forced Output).** No action-triggering output is emitted from any execution path that has entered the Conflict region for that evaluation.

**INV-2 (One Receipt Per Conflict).** Every entry into the Conflict region produces exactly one negative attestation artifact.

**INV-3 (Signed Attestation).** Every artifact is cryptographically signed or flagged as signature-pending with a queued retry.

**INV-4 (Append-Only Chain).** Signed artifacts are stored in an append-only, hash-linked sequence. No artifact may be removed, modified, or reordered.

**INV-5 (Non-Override).** No downstream component may override a Conflict determination to produce an action dispatch for the same input.

**INV-6 (No Action Without Prior Attestation).** The ADI dispatches an action only upon prior receipt of a valid attested execution-authority state.

**INV-7 (Conflict Forces Fallback).** When Conflict is entered, the ADI invokes the lane-defined fallback behavior.

**INV-8 (No Alternate Execution Path).** No execution path permits an inference engine output to reach an action executor without traversal through the Authority Layer and the ADI.

**INV-9 (Conflict State Externality).** The Conflict State is structurally external to the inference engine's output space. No output of the inference engine represents or substitutes for it.

---

## 10. Embodiment Domains

The architecture applies across multiple domains:

**Agriculture / Environmental Monitoring.** Edge devices classify sensor data into condition categories. When classification is ambiguous (e.g., Healthy vs. Pest Risk with narrow margin), the system blocks automated treatment and alerts operators. Receipts are forwarded to regulatory audit servers upon network reconnect.

**Robotic Path Planning.** Sensor fusion produces conflicting obstacle classifications. When top candidates are nearly tied, the system halts motors, assumes a safe pose, and awaits teleoperation. The receipt chain provides post-incident forensic evidence that the system correctly refused to act on ambiguous data.

**API Risk Assessment.** A probabilistic risk model evaluates incoming requests. When risk tiers are closely spaced, the system blocks automated approval, routes to human review, and produces a regulatory-grade audit record.

**Disaster Response.** Multi-source sensor fusion with conflicting environmental readings triggers conservative emergency protocols and produces auditable evidence of the decision boundary.

---

## 11. Core Distinction

All prior systems operate on decision objects. The decision object is the structural prerequisite for their function: certifiers evaluate it, rejection mechanisms produce it, audit systems log it, gates pass or block it.

ACHP-ADI is defined by the structural non-existence of the decision object. No decision object is ever instantiated, even momentarily. The system produces a negative attestation of that non-existence, cryptographically signs and chains it, and gates all downstream execution through an interface that is structurally incapable of deriving actions from the inference data that could have produced a decision. This is a topological constraint on machine operation — a reconfiguration of the computer's operational state space — not a policy applied to data that exists.

---

*End of whitepaper.*
