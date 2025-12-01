The Active-Quantum Risk-Killed Rotation Key Management Standard

Full Specification (Draft 1.0)
Status: Proposed Open Standard
License: MIT

⸻

1. Introduction

A-QRKR-K v3.0 defines the world’s first mathematically bounded, quantum-resilient key-lifecycle standard.
It guarantees that even if an adversary breaks encryption (including via CRQC-level quantum attacks), the maximum possible exposure time and maximum possible data volume per key are strictly bounded, verifiable, and auditable.

The core guarantee is enforced through:
	•	Tmax Functional (worst-case exposure control)
	•	Stochastic Epoch Rotation (SER)
	•	Cryptographic Byte-Volume Accounting (CBVA)
	•	Quantum-Computational Difficulty Certificates (QCDC)
	•	Signed Assertion Layer (SAL)
	•	QR-TLS Interoperability
	•	Quantum Resilience Score (QRS)

This spec defines the full architecture, constraints, primitives, and compliance rules.

⸻

2. Terminology & Normative Language

Terms MUST, SHOULD, MAY, REQUIRED, and OPTIONAL follow RFC 2119 usage.

⸻

3. Governance Inputs (Policy-Level Primitives)
4. Symbol
Name
Definition
Source
BQRC
Quantum Risk Budget
Max allowed attacker exposure per key (seconds).
Policy / Board
Vbudget
Volume Budget
Max ciphertext volume per key (bytes).
Policy / Board
Tbreak_bound
Break-Time Bound
Proven lower bound for cryptanalytic break time.
QCDC
αU
Uncertainty Penalty Factor
The variance multiplier applied to detection inputs.
Policy
4. Runtime Inputs (Measured Primitives)

These MUST be measured continuously:Symbol
Name
Definition
P(K)
Probability of Detection
Probability that an active misuse is detected.
τkill
Kill Chain Time
Time from detection to full kill-state completion.
kVC
Key Volume Counter
CBVA-tracked cumulative ciphertext volume.
Rthroughput
Throughput Rate
Bytes per second passing through the key.
Measurements MUST be uncertainty-penalized:P(K)pen = max(0, P(K) – αU · UP)
τkillpen = τkill + αU · Uτ

5. Core Security Functional

5.1 Tmax Functional (Mandatory Constraint)

To guarantee security against adversarial timing attacks, the maximum exposure time must satisfy:Tmax = P(K)pen · τkillpen + (1 – P(K)pen)

This MUST satisfy:Tmax ≤ BQRC
Solving for Δt:Δtactive_max = (BQRC – P(K)pen · τkillpen) / (1 – P(K)pen)
If:BQRC < P(K)pen · τkillpen

Then the system MUST enter Hard Fail-State:Δtactive_max = 0

6. Additional Hard Constraints

6.1 CBVA Functional (Volume Constraint)

The system MUST enforce a volume-based rotation trigger:kVC ≥ Vbudget → immediate rotation
The time-based projection:Δtvolume_max = (Vbudget – kVC) / Rthroughput

6.2 QCDC Constraint (Break-Time Constraint)

The QCDC MUST define:Δtbreak_max = Tbreak_bound

7. Final Control Law (Epoch Scheduling)

The target epoch is:Δttarget = min(Δtactive_max, Δtvolume_max, Δtbreak_max)

Stochastic Epoch Rotation (SER)

To prevent adversarial timing:

Δttarget MUST NOT be used directly.

Instead:Δt_actual = X
E[X] = Δttarget
X ~ Uniform(Δttarget/2, 3Δttarget/2)

Or any distribution with:
	•	Unpredictability
	•	Bounded support
	•	Mean = Δttarget

Rotation MUST occur at Δt_actual.

⸻

8. Signed Assertion Layer (SAL)

8.1 Purpose

SAL ensures all critical inputs are cryptographically authenticated.

8.2 Requirements

QCDC Attestation
	•	Tbreak_bound MUST be delivered in a signed artifact.
	•	The signature MUST chain to a root-of-trust.
	•	The Controller MUST reject unsigned or unverifiable QCDC files.

CBVA Integrity
	•	kVC MUST be signed by HSM/KMS using a monotonic counter (non-rollbackable).
	•	A discontinuity MUST trigger Hard-Fail rotation.

QRS Verification
	•	τkillpen MUST be delivered via a dual-path:

	1.	In-band: PAM-secured Tracer
	2.	Out-of-Band: GNSS-synchronized timestamp verifier

Both MUST match.

⸻

9. QR-TLS Interoperability

During session negotiation:Δtsession = min(Δtactual(A), Δtactual(B))

Each party MUST broadcast:
	•	QRS (τkillpen)
	•	Vbudget
	•	Tbreak_bound
	•	SAL-verified assertions

This allows session-level bounded exposure.

10. Quantum Resilience Score (QRS)QRS = τkillpen

11. Lower = more resilient.

QRS MUST be continuously measured and SAL-verified.

⸻

11. Compliance Levels

12. Level
Requirement
L0
Partial — missing constraints
L1
Full A-QRKR-K constraints
L2
Full + SAL
L3
Full + SAL + QR-TLS
L4
Full + SAL + QR-TLS + QCDC-Verified PQC Stack

12. Security Considerations
	•	Without SER, timing attacks are trivial.
	•	Without SAL, supply chain and insider attacks break the model.
	•	Without CBVA, SNDL volume limits cannot be enforced.
	•	Without QCDC, break-time constraints collapse.

All four are REQUIRED for full v3.0 compliance.

⸻

13. IANA Considerations

None.

⸻

14. License

MIT License.

⸻

End of Specification
