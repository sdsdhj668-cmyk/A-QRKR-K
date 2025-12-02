# SAL Primitive — Signed Assertion Layer (A-QRKR-K v3.0)
## 1. Overview
The Signed Assertion Layer (SAL) is the mandatory integrity mechanism for all control-critical values consumed by the A-QRKR-K Controller. SAL ensures no attacker or insider can falsify, backdate, replay, or modify the inputs that determine Δtactual, QRS, Tbreak_bound, or CBVA KeyVolumeCounter. Without SAL, the framework is vulnerable to silent parameter tampering. With SAL, all inputs become cryptographically non-repudiable.

## 2. Scope
SAL applies to all parameters that directly influence Δtactual. These MUST be signed, timestamped, monotonic, and verifiable:
- τkill_pen (QRS)
- Tbreak_bound (from QCDC)
- KeyVolumeCounter (kVC from CBVA)
- P(K) and posture uncertainty values
- Any configuration affecting Δtactual or Δttarget

## 3. Core Requirements (Normative)
### 3.1 Signing Requirements
All control inputs MUST be:
- Signed using an asymmetric keypair in a hardware root-of-trust (HSM/TPM/TEE)
- Timestamped using a TSP-compliant time source
- Bound to a monotonic nonce
- Encoded in canonical, deterministic JSON

### 3.2 Verification Requirements
The A-QRKR-K Controller MUST:
- Verify signatures against the issuer’s certificate chain
- Verify timestamp freshness within allowed drift
- Verify monotonic nonces
- Reject unsigned, expired, malformed, or replayed assertions
- Trigger Hard Fail-State on any verification failure

## 4. Assertion Formats
### 4.1 QRS Assertion (τkill_pen)
{
  "type": "QRS_ASSERTION",
  "value_seconds": <float>,
  "uncertainty_seconds": <float>,
  "timestamp": "<UTC ISO8601>",
  "nonce": <uint64>,
  "signature": "<base64>"
}

### 4.2 QCDC Assertion (Tbreak_bound)
{
  "type": "QCDC_ASSERTION",
  "algorithm_id": "<pqc_algorithm>",
  "lower_bound_seconds": <float>,
  "proof_hash": "<hex>",
  "timestamp": "<UTC ISO8601>",
  "nonce": <uint64>,
  "signature": "<base64>"
}

### 4.3 CBVA Assertion (KeyVolumeCounter)
{
  "type": "CBVA_VOLUME_ASSERTION",
  "key_id": "<UUID>",
  "volume_bytes": <uint64>,
  "volume_hash": "<hex>",
  "timestamp": "<UTC ISO8601>",
  "nonce": <uint64>,
  "signature": "<base64>"
}

## 5. Root-of-Trust Requirements
- Signing keys MUST be hardware-protected
- Public certificates MUST be distributed securely
- Certificate rotation MUST follow SAL rules
- Revocation MUST be supported via CRL/OCSP-equivalent

## 6. Replay and Rollback Protections
SAL enforces:
- Monotonic nonces (rollback triggers Hard Fail-State)
- Timestamp boundaries (stale assertions rejected)
- CBVA log hashing (prevents volume counter reset)
- QCDC proof hashing (prevents cryptanalytic bound fraud)

## 7. Failure Behavior
Hard Fail-State MUST be triggered if:
- Signature verification fails
- Timestamp is outside drift tolerance
- Nonce decreases or is reused
- Volume hash mismatch occurs
- QCDC hash mismatch occurs
- Root-of-trust chain is invalid

Fail-State responses MAY include:
- Immediate key rotation
- Disabling compromised KMS/HSM channels
- Broadcasting updated QRS posture
- Forcing QR-TLS renegotiation

## 8. Role of SAL in A-QRKR-K Architecture
SAL guarantees:
- Tbreak_bound cannot be forged
- kVC cannot be reset
- τkill_pen cannot be manipulated
- Posture inputs are cryptographically authenticated
- QR-TLS peers can trust resilience declarations (QRS)

With SAL, all A-QRKR-K primitives become tamper-evident and verifiable.

## 9. License
Released under the MIT License.
