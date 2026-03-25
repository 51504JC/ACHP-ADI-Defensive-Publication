# ACHP-ADI: Non-Instantiation Architecture for Probabilistic Decision Control

**Defensive Technical Publication**
Author: Richard Davis (GoBestow / ARC/R3P)
Publication Date: 2026-03-23
Version: 4.2

---

## What This Is

This repository constitutes a defensive technical publication of the Axiom Conflict Handling Protocol with Action Dispatch Interface (ACHP-ADI), a control architecture for computing systems that process probabilistic inference outputs. The disclosure establishes prior art and preserves the novelty of the described architecture for public benefit.

## The Problem

Computing systems that act on probabilistic inference outputs — classification models, sensor fusion pipelines, robotic controllers, API risk engines — share a structural deficiency. When the model's output distribution is ambiguous (the top candidates are closely spaced and no single output commands sufficient confidence), the system selects one output and dispatches the corresponding action anyway. This produces three failure modes: silent overconfidence (actions execute as if the model were certain), non-auditable ambiguity (no record that the decision was marginal), and forced execution (no structural mechanism prevents action dispatch under uncertainty).

Existing approaches address fragments of this problem. Threshold-based rejection (Chow, 1970) produces reject outputs within the classifier but does not gate downstream execution. Certified control (Jackson et al., MIT CSAIL) gates execution via an independent authority but does not evaluate probability distributions or produce auditable rejection records. Cryptographic audit logging records decisions that were made but does not prevent decisions from being forced.

All prior systems require a decision object to exist within the system. They then protect, evaluate, filter, log, or gate that object. The decision object is the operative element upon which every prior mechanism depends.

## The Breakthrough

ACHP-ADI is defined by the non-existence of the decision object.

When the decision margin between the top-ranked candidates in a probability distribution is insufficient, the system enters a conflict condition in which no decision object is ever instantiated — even momentarily. No executable decision object, intermediate action representation, candidate label, reject token, or any other data structure usable to initiate, derive, or approximate an action is formed, materialized, stored, transmitted, or recoverable at any stage of computation, including transient, intermediate, cached, register-level, speculative, or ephemeral machine states.

Instead of a decision, the system produces a negative attestation: a cryptographically signed artifact documenting that no decision could be made. This artifact is appended to a tamper-evident, hash-chained log and enables any auditor to perform deterministic replay of the decision boundary.

All downstream action dispatch passes through a non-bypassable execution interface (ADI) that operates as a state-driven emitter. The ADI has no data path from inference outputs to action emission. It is structurally absent the computational primitives necessary to derive actions from inference data. This is a topological constraint on machine operation, not a software policy.

## Architecture

The system comprises four structural layers in a linear, non-bypassable pipeline:

1. **Inference Engine** — Produces a probability distribution over candidate outputs. Has no direct connection to any action executor.
2. **Authority-State Derivation Layer** — Receives the distribution. Computes decision-quality metrics (decision margin = p_max - p_second). Partitions into three regions: Strong Decision, Weak/Flagged Decision, or Conflict (UNRESOLVED_ATTESTED). For Conflict, generates negative attestation artifact.
3. **Action Dispatch Interface (ADI)** — Sole path to action executors. Dispatches actions only from attested execution-authority states. Blocks all action paths on conflict. Structurally incapable of deriving actions from inference data.
4. **Action Executors** — Receive commands exclusively from the ADI.

## Why It Matters

- **Safety-critical systems**: Prevents ambiguous inference from causing uncontrolled actuation in robotics, autonomous vehicles, and industrial control.
- **Regulatory compliance**: Produces cryptographic proof that ambiguous decisions were not acted upon, with deterministic replay for audit.
- **Edge computing**: Operates offline-first with local tamper-evident logs synchronized when connectivity is available.
- **AI governance**: Converts probabilistic uncertainty into a verifiable, non-overridable control state.

## Repository Contents

| File | Description |
|---|---|
| `WHITEPAPER.md` | Full technical narrative of the architecture |
| `CLAIMS_REFERENCE.md` | Plain-English claim explanations and protection map |
| `NOVELTY_BRIEF.md` | Prior art comparison and novelty statement |
| `FIGURES.md` | Text descriptions of architectural diagrams |
| `ABSTRACT.md` | Publication abstract for indexing |
| `METADATA.json` | Structured publication metadata |
| `LICENSE.txt` | CC-BY-4.0 license |
| `SPECIFICATION.md` | Full v4.2 specification (canonical reference) |

## Full Specification

The canonical technical specification is `SPECIFICATION.md` in this repository. All other documents in this package are derived from and consistent with that specification. In case of any discrepancy, the specification governs.

## License

This work is licensed under Creative Commons Attribution 4.0 International (CC-BY-4.0). See `LICENSE.txt`.
