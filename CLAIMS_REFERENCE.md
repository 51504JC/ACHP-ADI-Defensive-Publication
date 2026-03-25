# ACHP-ADI Claims Reference

**Version 4.2 — Plain-English Explanations**
Date: 2026-03-23

---

## Independent Claims

### Claim 1 — Method

**What it says:** A computer-implemented method for controlling action execution in a system that processes probabilistic inference outputs. The method receives a probability distribution, computes a decision-quality metric (decision margin), compares it against fixed thresholds, and when both the maximum probability and the margin are below their respective thresholds, enters a conflict condition in which no decision object is ever instantiated at any stage of computation. The system then generates a negative attestation artifact, signs it, appends it to a tamper-evident chain, and transmits it to a non-bypassable execution interface that blocks all action paths and invokes fallback.

**What it protects:** The complete method — the sequence of receiving a distribution, computing metrics, determining that no action-authorizing state exists, ensuring no decision object is formed (including in transient/register/speculative states), generating a negative attestation, signing it, chaining it, and gating execution through an interface that has no computational pathway to inference data.

**Why it matters:** This is the foundational method claim. It defines the complete operational sequence and establishes that the conflict condition is not an inference engine output, that no decision object exists at any computation stage, and that the execution interface is structurally (not just policy) incapable of deriving actions from inference data. Any system performing this sequence infringes.

---

### Claim 2 — System

**What it says:** A computing system comprising: a probabilistic inference engine; an authority-state derivation layer; a cryptographic module; non-volatile memory with a tamper-evident log; a hardware-enforced memory isolation mechanism that prevents the execution interface from addressing inference-domain data (with hardware-level faults on attempted access); and a non-bypassable execution interface that operates as a state-driven emitter with no input path from inference data. The structural absence of computational primitives is enforced by either (i) the execution interface lacking a general-purpose ALU and FPU, or (ii) the execution interface operating in an address space from which inference data is excluded by hardware isolation.

**What it protects:** The physical system architecture. Recites specific hardware (memory isolation mechanism, hardware fault, fixed-function logic block) that makes this a machine claim, not an abstract idea on a generic computer.

**Why it matters:** This is the primary defense against Section 101 (abstract idea) challenges. The hardware-enforced memory isolation mechanism and the structural absence of computational primitives are physical machine constraints, not software policies.

---

### Claim 3 — Non-Transitory Medium

**What it says:** A non-transitory computer-readable medium storing instructions that cause a processor to perform the method of Claim 1.

**What it protects:** Software implementations distributed as firmware, binaries, or stored instructions.

**Why it matters:** Covers distribution of the method as a software product, complementing the method (Claim 1) and hardware (Claim 2) claims.

---

## Key Dependent Claims

### Claim 4 — Conflict State Externality and Reject-Class Exclusion

**What it protects:** The conflict condition cannot be represented by any modification, addition, or extension of the inference engine's output space — including adding a reject class, an abstention output, or an uncertainty indicator. No component receiving inference outputs may use them to derive, simulate, or approximate the conflict condition.

**Why it matters:** Closes the design-around path of adding a "reject" class to the inference engine and claiming it functions equivalently. The conflict condition is structurally external to the inference engine's output space by definition.

### Claim 5 — Distributional Ambiguity Trigger

**What it protects:** An optional entropy or other distributional ambiguity metric that forces conflict regardless of the decision margin. Covers the case where the top two candidates have adequate margin but probability mass is spread across many classes.

**Why it matters:** Prevents design-around by using entropy-based triggers instead of margin-based triggers. Both are covered.

### Claim 7 — Deterministic Replay

**What it protects:** The specific procedure: an independent party uses only the artifact data, recomputes metrics, confirms conflict was correctly entered, and confirms no action dispatch occurred.

**Why it matters:** Establishes that the artifact is self-contained for audit. No system access required.

### Claim 8 — Non-Override Invariant

**What it protects:** Once a negative attestation is received for an input, no subsequent signal from any component can cause action dispatch for that input. Parallel, cached, or derived state not produced by the authority layer is structurally inadmissible.

**Why it matters:** Prevents design-around via delayed override, cached decisions, or parallel processing paths.

### Claim 14 — Typed Authority-State Isolation

**What it protects:** The execution interface accepts only strictly typed tokens that contain no probability data and are decodable only through a fixed mapping table external to the inference engine.

**Why it matters:** Ensures the data flowing through the execution interface is informationally decoupled from inference outputs.

### Claim 15 — Non-Addressable Inference Memory

**What it protects:** Inference data resides in memory that is absent from the execution interface's address space. Page table entries do not exist. Hardware faults on access. No IPC, shared memory, DMA, or system call path can bridge the gap.

**Why it matters:** Establishes that the memory isolation is structural (pages absent) not policy (pages forbidden). This is the hardware-level enforcement of non-instantiation.

### Claim 16 — Structural Computational Isolation

**What it protects:** Two disjunctive arms: (a) the execution interface is a fixed-function logic block with no ALU, no FPU, no instruction decoder; or (b) the execution interface operates in an address space from which inference data is excluded by hardware isolation. Both are structural, not behavioral.

**Why it matters:** This is the strongest enforcement claim. It establishes that the execution interface is physically or architecturally incapable of deriving actions from inference data. The incapability does not depend on software configuration or policy.

### Claim 17 — Cryptographically One-Way Transformation

**What it protects:** Authority-state tokens are produced via a non-invertible transformation. Recovery of inference data from the token is computationally infeasible.

**Why it matters:** Prevents design-around via token analysis or side-channel inference of probability data from the token contents.

### Claim 19 — State Destruction

**What it protects:** Upon conflict, all transient inference state is irreversibly discarded, including processor registers, caches, buffers, and speculative execution traces. The negative attestation artifact becomes the only persistent state.

**Why it matters:** Closes the transient-state design-around: even register-level or speculative representations of inference data are destroyed. No reconstructable decision representation survives.

### Claim 20 — Replay Sufficiency

**What it protects:** The artifact contains everything needed to reconstruct the conflict condition without access to the inference engine, model weights, raw input, or any internal state.

**Why it matters:** Ensures the audit trail is self-contained and platform-independent.

---

## Claim Protection Map

| Claim | Protects Against | Design-Around It Blocks |
|---|---|---|
| 1 (Method) | Core sequence: distribution analysis, non-instantiation, attestation, gating | Reimplementing the method under different names |
| 2 (System) | Hardware architecture with memory isolation and structural incapability | Software-only or abstract implementations (Section 101 defense) |
| 4 (Externality) | Reject-class substitution | Adding "reject" to inference engine output space |
| 8 (Non-Override) | Post-conflict override attacks | Cached decisions, delayed authorization, parallel paths |
| 15 (Memory) | Memory-based side channels | IPC, shared memory, DMA bridges |
| 16 (Isolation) | Computational workarounds | Using ALU/FPU on smuggled inference data |
| 17 (One-Way) | Token analysis | Reverse-engineering probability data from tokens |
| 19 (Destruction) | Transient state recovery | Register dumps, cache inspection, speculative reads |
| 20 (Replay) | Audit dependency attacks | Claims that audit requires system access |

---

*End of claims reference.*
