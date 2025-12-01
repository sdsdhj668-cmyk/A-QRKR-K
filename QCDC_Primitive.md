
# Quantum-Computational Difficulty Certificate (QCDC) — Primitive Specification

## 1. Purpose
The Quantum-Computational Difficulty Certificate (QCDC) defines the minimum provable break-time (Tbreak_bound) for a cryptographic algorithm under quantum adversaries. It is the authoritative source of truth for all A-QRKR-K v3.0 break-time constraints and MUST be verifiable through the Signed Assertion Layer (SAL).

QCDC replaces vendor marketing claims with a formal, auditable, cryptographic complexity guarantee.

---

## 2. QCDC Requirements

A compliant QCDC MUST contain the following fields:

### 2.1 Algorithm Identifier
A unique string identifying the cryptographic algorithm, e.g.:
- “Kyber1024-Hybrid”
- “Dilithium-5-SAL”
- “AES-256 Post-Quantum Hybrid”

### 2.2 Quantum Complexity Model
A detailed description of the adversarial model, specifying:
- Gate model assumptions
- Error correction overhead assumptions
- Required qubit count
- Required logical depth
- Noise tolerance and decoherence parameters

This ensures the break-time estimate is tied to explicit, inspectable assumptions.

### 2.3 Break-Time Lower Bound (Tbreak_bound)
The certificate MUST include a provable LOWER BOUND on the time required for a quantum adversary to break the algorithm under the stated model.

This value MUST:
- Be expressed in seconds.
- Be derived using conservative, adversarial assumptions.
- Be NON-NEGOTIABLE once issued.

### 2.4 Proof Sketch or Reference
The QCDC MUST include either:
- A cryptanalytic proof sketch (as in NIST PQC submissions), OR
- A reference to published peer-reviewed work supporting the lower bound.

### 2.5 Certificate Lifetime
The QCDC MUST specify:
- The date of issue.
- The certificate expiration date.
- The conditions under which reissuance is required.

---

## 3. Signed Assertion Layer (SAL) Integration

Every QCDC MUST be delivered with a signed assertion proving its authenticity.

### 3.1 Signature Requirements
- MUST be digitally signed by the issuing vendor.
- MUST include a timestamp from a TSP-compliant clock.
- MUST be verifiable against a publicly auditable root-of-trust.

### 3.2 Controller Verification
The A-QRKR-K Controller MUST:
- Verify the QCDC signature.
- Reject ANY unsigned or unverifiable break-time value.
- Enforce a Hard Fail-State if signature verification fails.

No cryptographic break-time may be accepted without a valid SAL signature.

---

## 4. Operational Use

### 4.1 Runtime Binding
The Controller MUST bind:Tbreak_max := Tbreak_bound

where Tbreak_bound is derived directly from the verified QCDC artifact.

### 4.2 Triggering Rotation
If the epoch reaches Tbreak_max before any other constraint triggers, the key MUST be rotated immediately.

### 4.3 Forced Update
Upon QCDC expiration:
- All keys relying on that certificate MUST be rotated.
- A new QCDC MUST be provided before future keys can be generated.

---

## 5. Security Guarantees

When implemented correctly, the QCDC primitive guarantees:

- Break-time constraints are grounded in verifiable, externally issued evidence.
- No insider, vendor, or attacker can inflate Tbreak_bound without detection.
- All A-QRKR-K deployments share a common, auditable cryptanalytic foundation.
- The global ecosystem retains provable minimum security levels independent of implementation differences.

---

## End of qcdcprimitive.md
