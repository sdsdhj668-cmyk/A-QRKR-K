# Governance Guide — A-QRKR-K v3.0

## 1. Purpose
This document defines the governance responsibilities, policy inputs, and operational controls required to configure and maintain a compliant A-QRKR-K v3.0 deployment. Governance inputs are not derived from computation; they are human-defined, board-approved, and regulator-auditable.

## 2. Core Governance Inputs

### 2.1 Quantum Risk Budget (BQRC)
BQRC defines the maximum acceptable exposure time (in seconds) that any cryptographic key may remain usable under worst-case adversarial conditions.  
Requirements:
- MUST be defined and approved by executive risk governance.
- MUST reflect the maximum tolerable loss given the Harm Rate (Rharm).
- SHOULD be derived from financial risk analysis and incident loss history.

### 2.2 Volume Budget (Vbudget)
Vbudget defines the maximum ciphertext volume (in bytes) that a key may protect before mandatory rotation.  
Requirements:
- MUST be set based on data classification, regulatory sensitivity, and breach impact.
- MUST align with the CBVA primitive.
- SHOULD be updated with changes to workload patterns.

### 2.3 Harm Rate (Rharm)
Rharm is the expected cost per second of key misuse (fraud, unauthorized actions, system degradation).  
Requirements:
- MUST be estimated by finance and security jointly.
- MUST be used to validate BQRC with: ExpectedLoss ≤ Rharm × BQRC.
- SHOULD be recalculated annually or after any major architectural change.

### 2.4 Uncertainty Penalty (αU)
αU penalizes uncertainty in detection probability and kill-chain measurement.  
Requirements:
- MUST be defined by security governance (typical values 2–4).
- MUST be applied directly to PK and τkill.
- SHOULD err on the side of over-penalization to maintain conservatism.

### 2.5 Break Bound (Tbreak_bound) from QCDC
Break Bound defines the minimum time a quantum adversary requires to break the hybrid key.  
Requirements:
- MUST be sourced from a signed QCDC (Quantum-Computational Difficulty Certificate).
- MUST NOT be defined internally.
- MUST be verifiable through the Signed Assertion Layer (SAL).

---

## 3. Governance Responsibilities

### 3.1 Executive Risk Board
The board MUST:
- Approve BQRC.
- Approve Vbudget.
- Validate Rharm as part of financial exposure modeling.
- Mandate adoption of SAL and QCDC in procurement.

### 3.2 CISO / Security Leadership
Security leaders MUST:
- Configure αU for the environment.
- Ensure kill-chain measurement architecture is compliant.
- Oversee deployment and monitoring of CBVA.
- Maintain QR-TLS interoperability for external partners.

### 3.3 Engineering & DevOps
Engineering MUST:
- Implement the A-QRKR-K Controller.
- Ensure integration with HSM/TPM/secure enclave for CBVA.
- Maintain reliable Rthroughput and PK measurement pipelines.
- Ensure SER (Stochastic Epoch Rotation) is correctly implemented.

### 3.4 Compliance & Regulators
Compliance MUST:
- Audit QRS (Quantum Resilience Score).
- Verify SAL signatures for QCDC and CBVA artifacts.
- Monitor adherence to BQRC and Vbudget.
- Produce annual certification reports for external regulators.

---

## 4. Governance Cycles

### 4.1 Annual Review Cycle
The following inputs MUST be reevaluated annually:
- BQRC
- Vbudget
- Rharm
- αU
- Vendor QCDC claims

### 4.2 Continuous Measurement Cycle
The following MUST be measured in real time:
- PK estimate and UP (uncertainty)
- τkill (kill-chain time)
- kVC (Key Volume Counter)
- TSP-compliant time synchronization
- Throughput rate (Rthroughput)

### 4.3 Rotation Trigger Compliance
The system MUST enforce rotation immediately when:
- The Δtactive_maxpen bound is reached.
- The CBVA volume limit triggers (kVC ≥ Vbudget).
- The QCDC bound triggers (Δtbreak_max reached).
- The Tmax functional reaches BQRC under SER.

---

## 5. Governance Guarantees

When governance is properly configured:
- Maximum cryptographic exposure is mathematically bounded.
- Risk is quantifiable, insurable, and regulator-auditable.
- Insider threats cannot bypass volume or break-time limits.
- Interoperability (QR-TLS) ensures weakest-party protection.
- Compliance creates a self-healing upward security pressure.

---

## End of GovernanceGuide.md
