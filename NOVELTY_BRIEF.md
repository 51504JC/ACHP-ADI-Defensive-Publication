# ACHP-ADI Novelty Brief

**Version 4.2**
Date: 2026-03-23

---

## 1. Novelty Statement

The novelty of ACHP-ADI resides in the conjunction of the following elements operating as an integrated protocol:

(a) Deterministic threshold-based partitioning of a probability distribution into decision regions using a decision-margin metric (p_max - p_second).

(b) Explicit refusal to instantiate any decision object in the Conflict region — not a low-confidence output, not a reject class, not an uncertainty score, not any data structure usable to derive an action — enforced across all computation stages including transient, register-level, speculative, and ephemeral machine states.

(c) Generation of a negative attestation artifact documenting that no decision could be produced, containing the full distribution and decision metadata sufficient for deterministic replay.

(d) Cryptographic signing of that artifact.

(e) Append-only, hash-chained, tamper-evident storage of signed artifacts.

(f) Execution gating through a non-bypassable interface that is structurally absent the computational primitives necessary to derive actions from inference data.

No individual element is claimed as independently novel. The claimed invention is the specific combination of (a) through (f). Each element is necessary; removing any one produces a materially different system that does not satisfy ACHP-ADI invariants.

---

## 2. Prior Art Comparison

### 2.1 Chow (1970) — Reject Option in Classification

**What it teaches:** A classifier may optimally reject inputs when the maximum posterior probability falls below a cost-derived threshold. Extended by Geifman and El-Yaniv (2017, 2019) with learned rejection mechanisms and margin-based abstention.

**What it does not teach:**

- The rejection in Chow is an output of the classifier. It is a decision object (the "reject" label). ACHP-ADI prohibits the instantiation of any decision object in the Conflict region.
- No structured attestation record is generated on rejection.
- No cryptographic signing or hash chaining of rejection events occurs.
- No downstream execution interface is gated on the rejection.
- No non-override invariant prevents a downstream component from overriding the rejection.
- The rejection is internal to the classifier's output space. ACHP-ADI's conflict condition is structurally external to the inference engine's output space (Claim 4).

**Core distinction:** Chow produces a decision (to reject). ACHP-ADI is defined by the non-existence of any decision.

### 2.2 Jackson et al. (MIT CSAIL, 2019-2020) — Certified Control

**What it teaches:** An independent certifier evaluates a controller-generated certificate containing a proposed action and sensor evidence. If the check fails, a safe fallback controller takes over. The certifier has prioritized actuator access (structural non-bypassability). Sensor evidence in the certificate is cryptographically signed.

**What it does not teach:**

- The certifier evaluates physical safety conditions, not the internal structure of a probability distribution for decision-margin sufficiency.
- The architecture requires the existence of a proposed action (the certificate). ACHP-ADI requires the non-existence of any proposed action in the Conflict region.
- No structured attestation record documents the rejection conditions.
- No hash-chained, tamper-evident log of rejection events is maintained.
- No deterministic replay of the rejection decision boundary is possible.
- The certifier attests a positive condition (action is safe). ACHP-ADI attests a negative condition (decision could not be made).
- The execution interface (actuator access) is a conditional gate that passes or blocks a proposed action. ACHP-ADI's execution interface is a state-driven emitter with no input path from inference data.

**Core distinction:** CSAIL certified control evaluates and gates a decision object. ACHP-ADI operates in a system where no decision object exists to evaluate or gate.

### 2.3 Zero-Trust / TEE Attestation Systems

**What they teach:** Cryptographic attestation as a precondition for execution or access. Signed, structured tokens. Environment integrity verification. Remote attestation protocols (Intel SGX, AMD SEV-SNP, IETF RATS).

**What they do not teach:**

- TEE attestation attests positive conditions (environment is valid, code measurements match). ACHP-ADI attests a negative condition (decision could not be made).
- TEE systems do not analyze probability distributions or compute decision-margin metrics.
- TEE systems gate on platform integrity, not on probabilistic decision quality.
- TEE systems do not produce hash-chained logs of attestation failures with sufficient content for deterministic replay of a decision boundary.
- TEE systems do not enforce that no decision object is instantiated within the machine.

**Core distinction:** TEE systems attest what IS. ACHP-ADI attests what IS NOT.

### 2.4 Cryptographic Audit Logging / Blockchain for AI

**What it teaches:** Tamper-evident, hash-chained logging of AI inference decisions. Blockchain-based audit trails for regulatory compliance. Signed records of model inputs, outputs, and actions dispatched.

**What it does not teach:**

- Audit logging records decisions that were made. ACHP-ADI records that a decision was not made (negative attestation).
- Audit logging is post-hoc and observational. ACHP-ADI's attestation is prescriptive and pre-execution: the receipt is generated as part of the control flow that blocks the action, not after the action is dispatched.
- Audit logging does not gate action execution. ACHP-ADI's execution interface dispatches actions only from attested authority states.
- Audit logging does not prevent forced decisions. ACHP-ADI structurally prevents decision objects from being instantiated.
- Audit logging does not enable deterministic replay of a decision boundary from a single record.

**Core distinction:** Audit systems log what happened. ACHP-ADI prevents what should not happen and produces cryptographic proof of that prevention.

### 2.5 Safety Interlocks and Industrial Control

**What they teach:** Pre-condition enforcement for action execution based on physical sensor states (door open/closed, pressure thresholds). Binary go/no-go decisions.

**What they do not teach:**

- Interlocks evaluate physical sensor states, not probability distributions.
- Interlocks do not compute decision-margin metrics from multi-valued distributions.
- Interlocks do not produce structured negative attestation records.
- Interlocks do not maintain hash-chained audit trails of interlock activations.
- Interlocks do not enforce that no decision object is instantiated within the system.

**Core distinction:** Interlocks gate on physical state. ACHP-ADI gates on the quality of a probabilistic inference, and structurally prevents the inference from producing an actionable output when quality is insufficient.

### 2.6 Standard Hardening (MMU, TEE, Zeroization, ISA)

**What it teaches:** Memory isolation via MMU/IOMMU. Trusted execution environments. Secure zeroization of sensitive data. Capability-based execution restrictions. Speculative execution barriers (LFENCE, CSDB). Cache invalidation (CLFLUSH, DC CIVAC).

**What it does not teach:**

- These mechanisms are applied in prior art to protect data that exists. ACHP-ADI applies them to enforce the non-existence of data.
- No prior art uses MMU isolation to structurally exclude inference data from an execution interface's address space for the purpose of preventing action derivation from probability distributions.
- No prior art uses zeroization to destroy transient inference state as part of a conflict-handling protocol that generates negative attestation artifacts.
- No prior art removes ALU/FPU from an execution interface specifically to prevent computation on probability distributions.

**Core distinction:** Hardening protects existing data. ACHP-ADI uses the same mechanisms to enforce the non-existence of data. The purpose, the architectural context, and the structural result are different.

---

## 3. Summary Table

| Property | Chow | CSAIL | TEE | Logging | Interlocks | ACHP-ADI |
|---|---|---|---|---|---|---|
| Decision object exists? | Yes (reject label) | Yes (certificate) | N/A | Yes (logged output) | N/A | **No** |
| Evaluates probability distribution? | Yes (p_max only) | No | No | No | No | **Yes (margin)** |
| Refuses to instantiate any decision? | No | No | No | No | No | **Yes** |
| Negative attestation? | No | No | No (positive) | No | No | **Yes** |
| Signed rejection record? | No | No | N/A | Sometimes | No | **Yes** |
| Hash-chained rejection log? | No | No | No | Sometimes | No | **Yes** |
| Deterministic replay of decision boundary? | No | No | No | No | No | **Yes** |
| Non-bypassable execution gating? | No | Yes | N/A | No | Partial | **Yes** |
| Execution interface structurally incapable? | N/A | No | N/A | N/A | N/A | **Yes** |
| Transient state destruction? | No | No | N/A | No | No | **Yes** |

---

## 4. The Fundamental Distinction

All prior systems instantiate a decision object and then act on it — evaluating, filtering, logging, gating, or rejecting it.

ACHP-ADI is defined by the non-existence of the decision object. No decision object is ever instantiated, even momentarily, at any stage of computation. The system attests that non-existence, signs the attestation, chains it, and gates all downstream execution through an interface that is structurally incapable of deriving actions from the inference data that could have produced a decision.

The cited art operates on decisions. This system is defined by their non-existence.

---

*End of novelty brief.*
