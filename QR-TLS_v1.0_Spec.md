# QR-TLS v1.0 Specification  
Quantum-Resilient Transport Layer Security Extension

## 1. Purpose
QR-TLS v1.0 defines the interoperability mechanism that allows two independent systems, each running the A-QRKR-K v3.0 Controller, to negotiate a shared session key lifespan (Δtsession) based on the strictest available security constraint. QR-TLS ensures that no TLS session ever exceeds the quantum-resilient limits of either participant.

## 2. Design Principles
QR-TLS is built on four foundational principles:
1. Least-Privilege Timing: The session key MUST adopt the smallest (strictest) rotation interval from either participant.
2. Transparency and Accountability: Both systems MUST disclose their local resilience parameters (QRS and Δtactual).
3. Cryptographic Authenticity: All resilience assertions MUST be signed via the Signed Assertion Layer (SAL).
4. Backward Compatibility: QR-TLS is an extension to TLS, not a replacement. Legacy TLS systems simply ignore the QR-TLS negotiation fields.

## 3. Normative Requirements
### 3.1 Required Local Inputs
Each participant MUST provide:
- Δtactual — its computed maximum safe epoch length, derived from the A-QRKR-K control law.
- QRS — the quantum resilience score (penalized kill-chain time, τkill_pen).
- SAL signature — cryptographic proof the values are authentic and untampered.

If any field is missing or fails signature validation, the handshake MUST be aborted.

## 4. QR-TLS Handshake Extensions
QR-TLS introduces three new handshake messages:
1. ClientHello_QR  
2. ServerHello_QR  
3. QRSessionConfirm

These messages encapsulate the resilience parameters required to compute the joint session constraint.

## 5. Message: ClientHello_QR
The client MUST send a ClientHello_QR message containing:
client_qrs: integer (ms)  
client_dtactual: integer (ms)  
client_sal_sig: bytes (SAL signature)

Requirements:
- The SAL signature MUST cover both values and a timestamp.
- The timestamp MUST be TSP-compliant to prevent replay.

## 6. Message: ServerHello_QR
The server MUST reply with:
server_qrs: integer (ms)  
server_dtactual: integer (ms)  
server_sal_sig: bytes

Requirements:
- Same SAL rules as ClientHello_QR.
- If the server cannot provide valid SAL, the handshake MUST fall back to classical TLS with a severe downgrade warning.

## 7. Joint Session Constraint
Both sides MUST independently compute:
Δtsession = min(client_dtactual, server_dtactual)

This rule is mandatory and non-negotiable. No QRS comparison, averaging, or policy override is permitted. Whichever side has the stricter (smaller) rotation interval dictates the session lifespan.

## 8. Message: QRSessionConfirm
Both sides exchange a QRSessionConfirm message containing:
agreed_dtsession: integer (ms)  
sal_sig: bytes

Requirements:
- Each side MUST sign the agreed Δtsession using its SAL key.
- A mismatch between the signatures MUST abort the handshake.
- The session key MUST NOT be used until both confirmations match.

## 9. Runtime Enforcement
### 9.1 Session Key Rotation
The TLS session key MUST be rotated according to Δtsession. Rotation MUST occur via SER (Stochastic Epoch Rotation):
- Actual epochs are drawn from a distribution X with E[X] = Δtsession.
- SER prevents adversarial timing attacks.

### 9.2 Session Termination
If the session exceeds Δtsession without rotation, BOTH sides MUST immediately terminate the connection.

### 9.3 SAL Enforcement
Every rotated key MUST carry:
- A CBVA volume counter,
- A SAL signature,
- A timestamp.

Any mismatch MUST trigger immediate session termination.

## 10. Security Properties
QR-TLS provides:
1. Mutual Security Floor Enforcement — The weaker party’s constraints always dictate session rotation.
2. Tamper-Proof Negotiation — SAL prevents manipulation of QRS or Δtactual.
3. Timing-Attack Immunity — SER removes the ability to time the compromise to epoch start.
4. Volume-Bounded Exposure — CBVA ensures stolen data per session is limited.
5. Quantum-Resilient Interoperability — Any two A-QRKR-K systems can securely interact without trust.

## 11. Failure and Downgrade Modes
### 11.1 SAL Failure
If either side cannot verify the SAL signature on any QR-TLS field:
- The handshake MUST abort OR
- Fall back to classical TLS with a severe downgrade flag.

### 11.2 QRS or Δtactual Omission
If either field is missing:
- QR-TLS MUST NOT proceed.

### 11.3 Inconsistent Δtsession
If the two parties compute different Δtsession:
- The session MUST NOT start.
- A mismatch alert MUST be logged.

## 12. Versioning & Extensibility
QR-TLS v1.0 MUST remain backward compatible with TLS 1.3. Future versions MAY introduce:
- Multi-party negotiation,
- Post-quantum cipher binding,
- Adaptive policy-based reduction of Δtsession.

## 13. End of QR-TLS v1.0 Specification
