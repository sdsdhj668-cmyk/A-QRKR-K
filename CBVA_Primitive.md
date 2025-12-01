# Cryptographic Byte-Volume Accounting (CBVA) — Primitive Specification v1.0

## 1. Purpose
CBVA defines the mandatory mechanism for measuring, attesting, and enforcing the maximum ciphertext volume a cryptographic key may protect before forced rotation. It provides the volume-based defense in the A-QRKR-K v3.0 Control Law.

## 2. Definitions
| Symbol | Name | Definition |
|--------|------|------------|
| kVC | Key Volume Counter | Monotonic counter tracking total bytes encrypted by key k_e. |
| Vbudget | Volume Budget | Maximum allowed ciphertext volume for a single key. |
| Rthroughput | Throughput Rate | Measured rate at which ciphertext is produced through the session. |

## 3. Requirements

### 3.1 Trusted Hardware Enforcement
A conformant system MUST maintain kVC within a Hardware Security Module (HSM), a Trusted Platform Module (TPM), or an embedded secure enclave with monotonic counters. The counter MUST NOT be modifiable, reset, or overwritten by software.

### 3.2 Monotonic Counter Integrity
The counter MUST satisfy:
kVC(t2) ≥ kVC(t1) for all t2 > t1.
Any violation indicates tampering and MUST trigger a Hard Fail-State (immediate rotation and alert).

### 3.3 Signed Assertion Layer (SAL) Compliance
Every reported counter value MUST include:
- a cryptographic signature from the hardware root-of-trust,
- a timestamp,
- a monotonic sequence number.
The A-QRKR-K Controller MUST verify the signature before using the value.

## 4. Rotation Trigger
Rotation MUST occur if:
kVC ≥ Vbudget
This trigger operates independently of detection probability, kill-chain time, or cryptanalytic break time. No key may ever protect more ciphertext than the policy allows.

## 5. Computation of ΔtSNDL_max
For informational display (not enforcement), the controller MAY compute:
ΔtSNDL_max = (Vbudget − kVC) / Rthroughput
This reflects the remaining lifetime based on volume but MUST NOT override the hard trigger in Section 4.

## 6. Security Guarantees
CBVA provides:
1. Inviolable Volume Limit — attackers cannot use covert exfiltration to bypass the key’s maximum allowed data protection window.
2. Independence from Network Monitoring — the constraint is enforced at the cryptographic boundary, not the network boundary.
3. Global Predictability — any CBVA-compliant system behaves identically under volume stress.

## 7. Required Interface
A CBVA-compliant HSM/TPM/enclave MUST expose:
get_volume_counter() → { kVC, signature, timestamp, seq }

## End of CBVA_Primitive.md
