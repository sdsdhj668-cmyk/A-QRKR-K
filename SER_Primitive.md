# SER Primitive — Stochastic Epoch Rotation (A-QRKR-K v3.0)

## 1. Overview
Stochastic Epoch Rotation (SER) is the timing-randomization primitive that prevents adversaries from exploiting deterministic rotation intervals. Without SER, an attacker can time a key break to coincide with the start of an epoch and exploit the full Δt window. SER removes all predictability from rotation timing while preserving mathematical correctness via the Tmax constraint.

## 2. Purpose
SER ensures:
- Attackers cannot predict when the next key rotation will occur.
- The rotation interval remains centered around Δttarget but unpredictable in absolute time.
- The Tmax Functional (Full-Cycle Constraint) remains valid under hostile timing.
- Exposure never exceeds BQRC even if detection fails.

## 3. Functional Definition
SER defines the actual rotation time X for each epoch as a cryptographically random variable with:
- Expected value: E[X] = Δttarget
- Support interval: [Δttarget/2, 3Δttarget/2]
- Distribution: Uniform (or policy-approved alternative)

Formally:
X ~ Uniform(Δttarget/2, 3Δttarget/2)

This yields:
- Minimum possible X: Δttarget/2
- Maximum possible X: 3Δttarget/2
- No deterministic traceability
- Guaranteed unpredictability

## 4. Constraints
SER MUST:
- Preserve the BQRC exposure limit (i.e., Tmax ≤ BQRC at all times).
- Use a hardware-rooted CSPRNG.
- Produce outputs that cannot be predicted, influenced, or replayed.
- Ensure rotation times are not logged in advance or made inferable by system probes.

## 5. Normative Algorithm (Pseudocode)
function generate_epoch_length(Δt_target):
    lower = Δt_target / 2
    upper = 3 * Δt_target / 2
    X = CSPRNG_uniform(lower, upper)
    return X

## 6. Integration with A-QRKR-K Controller
1. Controller computes Δttarget using the Full-Cycle Constraint:
   Δttarget = min(Δt_active_max_pen, Δt_SNDL_max, Δt_break_max)
2. Controller calls SER to randomize:
   X = generate_epoch_length(Δttarget)
3. Controller schedules rotation at time X.
4. X MUST NOT violate the Tmax ≤ BQRC guarantee.

This makes rotation unpredictable while keeping exposure controlled.

## 7. Security Properties
SER provides:
- Immunity to timing attacks.
- Enforcement of the Full-Cycle Constraint under worst-case conditions.
- Elimination of epoch-based side channels.
- Protection from attacker reconnaissance of rotation schedules.
- Entropy-backed unpredictability.

## 8. Conformance Requirements
Implementations MUST:
- Use a hardware-based CSPRNG (TPM, HSM, TEE, etc.).
- Refresh entropy sources per epoch.
- Ensure distribution is uniform or policy-approved.
- Never reveal planned rotation times or intermediate SER values.
- Perform SER inside a trusted execution environment.

## 9. Failure Handling
If SER entropy fails or randomness degrades:
- Controller MUST default to Δt_active_max_pen.
- System MUST broadcast degraded posture in QRS.
- Repeated failures MUST trigger a Hard Fail-State (immediate rotation + posture lock).

## 10. License
Released under the MIT License.
