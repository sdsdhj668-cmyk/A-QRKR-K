# A-QRKR-K v3.0 Specification  
Active Quantum Risk Killed Rotation Key Management Standard

Status: Draft 1.0  
Intended Use: Open technical standard for community adoption

## 1. Introduction

A-QRKR-K defines a key management control law that guarantees a hard upper bound on the useful exposure time of any compromised key in the presence of quantum capable adversaries.

The standard does not replace post-quantum cryptography. It wraps any cryptographic stack in a mathematically governed key lifecycle that:

- Limits worst case exposure time per key
- Limits ciphertext volume per key
- Enforces rotation based on measurable, attested parameters
- Provides an auditable Quantum Resilience Score (QRS)

This document defines:

- Core symbols and governance inputs
- The Full Cycle Constraint on maximum exposure time
- The control law for selecting the target epoch length
- The required primitives: SER, CBVA, QCDC, SAL
- Conformance requirements for implementations

## 2. Normative Language

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,  
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**  
are to be interpreted as described in RFC 2119.

## 3. Symbols and Governance Inputs

The A-QRKR-K Controller operates with the following primitives.

### 3.1 Governance Parameters

These are set by policy and external certification.

- `BQRC` - Quantum Risk Budget. Maximum allowed worst case exposure time for a single key, in seconds.
- `Rharm` - Harm Rate. Expected loss per second when a key is actively misused. Can be financial or mission impact units.
- `Vbudget` - Volume Budget. Maximum total ciphertext bytes that any key is allowed to protect.
- `Tbreak_bound` - Break Bound. Lower bound on time required for a quantum capable adversary to break the underlying cryptography. Derived from QCDC.
- `α_U` - Uncertainty Penalty Factor. Dimensionless multiplier (for example 2.0 or 3.0) used to penalize uncertain measurements.

### 3.2 Measured Posture Inputs

These are measured or inferred from the deployed environment.

- `P_K` - Probability that an active attack on a given key will be detected and trigger a kill state.
- `U_P` - Uncertainty on `P_K` (for example standard deviation or confidence interval width).
- `τ_kill` - Expected time between attack detection and effective key kill, in seconds.
- `U_τ` - Uncertainty on `τ_kill`.
- `R_throughput` - Observed throughput of ciphertext protected by the key, in bytes per second.
- `KeyVolumeCounter` (`kVC`) - Monotonic counter of bytes encrypted under the key, maintained by CBVA.

### 3.3 Penalized Inputs

The controller MUST derive worst case, penalized values:

- `P_K_pen = max(0, P_K - α_U * U_P)`
- `τ_kill_pen = τ_kill + α_U * U_τ`

These penalized values are used in all risk constraints.

## 4. Threat Model Summary

A-QRKR-K assumes:

- An adversary with access to a cryptographically relevant quantum computer
- Ability to break public key systems (Shor class attacks) and weaken symmetric systems (Grover class)
- Ability to conduct Harvest Now Decrypt Later campaigns
- Ability to mount active attacks using compromised keys once they are broken
- Ability to time key misuse within a rotation epoch to maximize exposure
- Potential partial compromise or misconfiguration of detection systems

The standard does not assume:

- Perfect detection
- Perfect volume monitoring
- Honest vendors by default

Instead, it assumes hostile conditions and uses attested inputs and worst case bounds.

Full details are in `Threat_Model.md`.

## 5. Core Functional: Full Cycle Maximum Exposure

### 5.1 Maximum Exposure Definition

For a single key `k_e` active over epoch length `Δt`, the worst case maximum useful exposure time `T_max` is:

- With probability `P_K_pen`, an active attack is detected and stopped in time `τ_kill_pen`
- With probability `(1 - P_K_pen)`, detection fails, and a hostile adversary times the attack at the start of the epoch, achieving up to `Δt` of undetected use

Therefore:

`T_max = P_K_pen * τ_kill_pen + (1 - P_K_pen) * Δt`

### 5.2 Risk Budget Constraint

The Quantum Risk Budget `BQRC` defines the maximum allowed `T_max` per key:

`T_max ≤ BQRC`

Substituting and solving for `Δt`:

`P_K_pen * τ_kill_pen + (1 - P_K_pen) * Δt ≤ BQRC`

`(1 - P_K_pen) * Δt ≤ BQRC - P_K_pen * τ_kill_pen`

If `P_K_pen < 1`:

`Δt ≤ Δt_active_max_pen`

where

`Δt_active_max_pen = (BQRC - P_K_pen * τ_kill_pen) / (1 - P_K_pen)`

### 5.3 Feasibility and Hard Fail State

If

`BQRC < P_K_pen * τ_kill_pen`

then the numerator is negative and no positive epoch length satisfies the constraint. In that case:

- The controller MUST set `Δt_active_max_pen = 0`
- The system MUST enter a Hard Fail State for that key class
- New sessions MUST NOT be established using that policy until either `BQRC` is increased or `τ_kill_pen` is reduced

## 6. Volume Constraint (CBVA)

Key volume is enforced by the CBVA primitive.

### 6.1 Volume Limit

For each key `k_e` the KMS or HSM MUST maintain `KeyVolumeCounter` as a monotonic counter of bytes encrypted under the key.

The volume constraint is:

`KeyVolumeCounter ≤ Vbudget`

### 6.2 Derived Time Limit

Given observed throughput `R_throughput`, the remaining volume headroom is:

`V_remaining = Vbudget - KeyVolumeCounter`

If `V_remaining ≤ 0`, the key MUST be rotated immediately.

If `V_remaining > 0`, the volume based time bound is:

`Δt_volume_max = V_remaining / R_throughput`

This is the latest safe time until volume budget is exhausted.

## 7. Break Time Constraint (QCDC)

The QCDC primitive provides `Tbreak_bound`, the certified lower bound on break time.

The break time constraint is:

`Δt_break_max = Tbreak_bound`

The controller MUST NOT set the target epoch length greater than `Tbreak_bound`.

## 8. Combined Control Law

The controller computes the target epoch length `Δt_target` as:

`Δt_target = min(Δt_active_max_pen, Δt_volume_max, Δt_break_max)`

If any term is undefined or invalid:

- If `Δt_active_max_pen` is zero due to infeasibility, the Hard Fail State rules apply
- If `Δt_volume_max` is undefined due to missing volume data, the system MUST treat that as zero for safety
- If `Δt_break_max` is missing, the system MUST treat it as zero until a valid QCDC is available

## 9. Stochastic Epoch Rotation (SER)

A-QRKR-K does not expose a fixed epoch length to the adversary. Instead, the controller uses a stochastic scheduler.

The KMS or controller MUST:

- Sample a random variable `X` from a distribution with mean equal to `Δt_target`
- Use `X` as the actual rotation delay for that key

Example implementation:

- Choose `X` uniformly from `[Δt_target / 2, 3 * Δt_target / 2]`
- Or choose `X` from an exponential or bounded distribution suitable for the environment

Requirements:

- `E[X] = Δt_target`
- The actual rotation time MUST be unpredictable to an external observer
- The controller MUST enforce an absolute upper bound (for example `2 * Δt_target`) to avoid extreme tails

SER reduces the ability of an adversary to time attacks relative to rotation while the Full Cycle Constraint protects against worst case timing even if the distribution is partially known.

Details are in `SER_Primitive.md`.

## 10. Signed Assertion Layer (SAL)

The control law relies on three critical inputs:

- `Tbreak_bound` from QCDC
- `KeyVolumeCounter` from CBVA
- Posture measurements that feed `P_K_pen` and `τ_kill_pen` (for example output from the Kill Chain Tracer)

The SAL primitive defines how these inputs are signed and verified.

At minimum:

- QCDC artifacts MUST be digitally signed by the issuing entity and verified against a root of trust
- KMS or HSM MUST sign volume counter records in a monotonic log
- Posture measurements that feed QRS MUST be backed by signed records, or by a verified measurement pipeline

Details are in `SAL_Primitive.md`.

## 11. Quantum Resilience Score (QRS)

The Quantum Resilience Score is defined as:

`QRS = τ_kill_pen`

In words: the worst case, uncertainty penalized kill chain time, in seconds.

Implementations:

- MUST compute and expose QRS for each key policy class
- SHOULD expose QRS for reporting, audit, and procurement
- MAY set procurement thresholds such as "QRS ≤ 5 seconds" for critical systems

The QRS is not itself part of the control law, but is derived from the same inputs and enables external comparison and governance.

## 12. QR-TLS Interoperability

When two parties establish a secure session under QR-TLS:

- Each side computes its local `Δt_target` and `QRS`
- The session epoch MUST be set to

`Δt_session = min(Δt_target_A, Δt_target_B)`

- SER MAY then be applied independently on each side as long as each respects the session bound

The QR-TLS specification is in `QR-TLS_v1.0_Spec.md`.

## 13. Conformance Summary

A system conforms to A-QRKR-K v3.0 if it:

1. Implements the Full Cycle Constraint with penalized `P_K_pen` and `τ_kill_pen`
2. Enforces volume limits using CBVA
3. Respects `Tbreak_bound` via QCDC
4. Uses SER for unpredictable rotation
5. Protects all critical inputs via SAL
6. Computes and exposes QRS for each key policy
7. Uses the combined control law to set `Δt_target` for each key or key class

---
