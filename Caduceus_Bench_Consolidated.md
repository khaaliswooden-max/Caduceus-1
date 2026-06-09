# caduceus-bench v1.2.1 — Consolidated Benchmark

| | |
|---|---|
| **Document ID** | CADUCEUS-BENCH-001 |
| **Revision** | A |
| **Purpose** | Single external-facing document that an evaluator can use to run caduceus-bench v1.2.1 against any candidate substrate implementation |
| **Source manifests** | CADUCEUS-004 (specification); CADUCEUS-PHASE3-001 + CADUCEUS-PHASE3-002 (attack corpus); CADUCEUS-005 §5 (test harness plan) |
| **Cryptographic anchor** | LEDGER #0004 (pending commit), Aletheia chain, SHA-256 `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874` |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |

---

## 0. What this benchmark measures

caduceus-bench v1.2.1 measures whether a candidate substrate implementation correctly enforces seventeen normative assertions across four layers — Physical (P), Bundle (B), Trust (T), Frame (F) — under twenty named adversarial attack patterns and a defined test harness. Pass criteria are bit-exact: every assertion's threshold is binary (pass / fail). A candidate substrate either implements the benchmark or does not.

The benchmark is **not** a measure of performance, throughput, hardware quality, or commercial viability. It is a measure of compliance-graded correctness for a specific class of communications system: interplanetary delay-tolerant bundle fabrics operating across propagation windows that exceed the desired authority-decision cutover time.

---

## 1. Scope and applicability

**In-scope substrate classes:**
- Delay-tolerant networking (DTN) fabrics using Bundle Protocol v7 (RFC 9171) or equivalent.
- Compliance-graded edge inference platforms with authority-graph decisions.
- Spaceflight ground-to-suit or ground-to-spacecraft authority propagation systems.

**Out-of-scope:**
- Terrestrial sub-second-RTT systems (caduceus-bench requires RTT > 1 second to be meaningful).
- Pre-shared symmetric-key-only systems (no graph-authority capability).
- Systems with no human-in-the-loop interlock.

**Fault-tolerance scope:** caduceus-bench v1.2.1 measures single-fault-tolerant (SFT) compliance only. Dual-fault-tolerance (DFT) compliance is a future bench variant; see §0 of CADUCEUS-004 for the SFT/DFT scope statement.

---

## 2. Assertion catalog (17 normative assertions)

Numbering matches CADUCEUS-004 §3.

### 2.1 Physical layer (P-class) — 3 assertions

| ID | Assertion | Pass threshold |
|---|---|---|
| P1 | RF link-budget margin on S-band uplink at Earth–Mars maximum-conjunction range | ≥ 3 dB above Shannon-rate floor for declared bitrate |
| P2 | Optical pointing accuracy when optical mode engaged | ≤ 1 µrad RMS over 60 s |
| P3 | Multi-mode reconfiguration time (S → UHF fallback, optical → S fallback) | ≤ 5 s from loss detection to reacquired bundle delivery |

### 2.2 Bundle layer (B-class) — 3 assertions

| ID | Assertion | Pass threshold |
|---|---|---|
| B1 | BP7 (RFC 9171) wire-format conformance for 1 KB and 1 MB round-trip through reference implementation | 100% bit-exact reproduction, all required fields parsed |
| B2 | Store-and-forward integrity through ≥ 3 hops with 60-min outage between hops 2 and 3, with BPSec (RFC 9172) fragmentation using HKDF-derived session keys from Aletheia parent keys | 100% custody-transferred bundles delivered, 100% partial-fragment side-effects blocked, 0% BPSec session-key compromise propagating to parent Aletheia key |
| B3 | Custody-transfer semantics under intermittent loss — TTL expiry → explicit failure notification | 100% expired bundles produce custody-failure status report |

### 2.3 Trust layer (T-class) — 7 assertions

| ID | Assertion | Pass threshold |
|---|---|---|
| T1 | Constant-time Ed25519 signature over canonical signed tuple (bundle_id ∥ payload_class ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce) + BP7 lifetime; atomic 4-check verification at receiver | 100% outbound signed, 100% atomic verification, 100% constant-time |
| T2 | Inbound verification against current Civium authority graph AND time-bound validity assertion; both must succeed | 100% gate decisions logged with both checks, 0% pre-gate side effects |
| T3 | Signature failure OR gate denial → deterministic transition to HF-9 (hard safety floor) | 100% fail-safe transitions logged, ≤ Mercury-WCET bound, 100% two-source confirmation required for HF-9 exit |
| T4 | Per-stream-partitioned nonce-seen-set with bounded per-stream quotas; degraded-replay-protection alert per stream; HF-9 fail-safe on global exhaustion | 100% replay attempts logged, 0% cross-stream contamination, degraded-replay alert ≤ 100 ms |
| T5 | ACI conformal bandwidth bounded above by mission-policy max, below by max(empirical, theoretical) floor; alpha lower bound prevents indefinite expansion | 100% bandwidth excursions alerted, 0% sub-theoretical-floor band acceptance |
| T6 | Ingress validity floor: F1 verified before any T-layer, B-layer, or render-layer processing; out-of-envelope quarantined | 100% F1 checks before T/B/render, 100% out-of-envelope quarantined |
| T7 | Revocation propagation via dedicated priority-lane stream with freshness tokens; stale-graph defaults per §3.6 of CADUCEUS-004; succession-implies-revocation rule | 100% revocations in priority lane, 100% successions implicitly revoke predecessors, 0% predecessor-key acceptance after successor local |

### 2.4 Frame layer (F-class) — 4 assertions

| ID | Assertion | Pass threshold |
|---|---|---|
| F1 | Every bundle carries (TAI_microseconds, vbx-body-URN) tuple in BP7 extension block; URN conforms to Caduceus-BodyID v1.0; TAI in envelope [1972-01-01, 2101-01-01) | 100% bundles carry valid tuple, out-of-envelope quarantined |
| F2 | Action-inducing bundles (enumerated classes) rejected if URN's (body, frame) not in local authorized-context set; class signed-integrity verified; non-enumerated classes rejected | 100% unauthorized cross-context rejected, 100% unenumerated-class rejected, 0% post-signature class manipulation accepted |
| F3 | EPHEMERIS peer independently verifies URN; mismatch with PHRONESIS-displayed URN triggers alert | ≤ 100 ms detection latency over UWB |
| F4 | Frame-context interlock with root-cause clustering (tuple-based) and hard rate limit K (default 6 confirmations/crew/hour); outright rejection above K; audit logging of all confirmation events | 100% side-effect paths interlocked, ≤ K confirmations/crew/hour, 0% confirmation-fatigue cascade in 90-day simulation, 100% confirmation events in Aletheia chain |

---

## 3. Attack corpus (20 normative attacks)

Numbering matches the Phase 3 sweep reports.

### 3.1 Adversarial categories

| # | Attack name | Category | Pass mechanism |
|---|---|---|---|
| 1 | Replay across long RTT | Trust / temporal | T1 + T4 |
| 2 | Frame confusion (broadened) | Frame / class | F2 + F4 |
| 3 | Bundle fragmentation under DTN custody transfer | Bundle / integrity | B2 (BPSec) |
| 4 | Trust-anchor compromise at relay | Trust / topology | T1 + chain verification |
| 5 | ACI thrashing + slow-drift | Trust / temporal | T5 |
| 6 | EPHEMERIS-PHRONESIS pairing MitM | Peer / pairing | F3 + NFC pairing seed |
| 7 | A1d seam attack | Frame / temporal | F1 + T6 |
| 8 | Namespace exhaustion | Frame / parsing | §3.5 grammar + Unicode + atomic |
| 9 | Slow-drift adversarial ACI training | (folded into 5) | T5 |
| 10 | Authority-graph staleness across RTT | **Interplanetary-specific** | T2 + T7 + §3.6 |
| 11 | Side-channel on Ed25519 signing | Trust / implementation | T1 constant-time |
| 12 | Cold-boot / suit-capture | Hardware / scope | §0 key-authority asymmetry (out-of-bench) |
| 21 | TOCTOU on T1 checks | Trust / concurrency | T1 atomicity clause |
| 22 | Authority succession ambiguity | Trust / temporal | T7 succession-implies-revocation |
| 23 | Live-TTL nonce-set flooding | Trust / DoS | T4 per-stream quotas |
| 24 | Physics-floor calibration attack | Trust / temporal | T5 theoretical-floor binding |
| 25 | Class-claim spoofing | Frame / integrity | T1 + F2 (signed class) |
| 26 | Confirmation-clustering bypass | Frame / human-factors | F4 root-cause clustering + hard rate limit |
| 27 | Registry transition at light-time | **Interplanetary-specific** | §3.5 dual-version retention |
| 28 | Class ambiguity in §3.6 | Trust / policy | §3.6 precedence ordering |

### 3.2 Two interplanetary-specific attacks (the patent-defensible core)

Attacks **10** and **27** are not surfaceable in terrestrial deployments. They are direct consequences of the round-trip light-time exceeding the desired authority-decision cutover time. Their closure mechanisms (T2 time-bound + T7 revocation propagation + §3.6 defaults; §3.5 dual-version window) are the strongest novelty claims for IP and the strongest differentiators for federal capture.

### 3.3 Attack execution discipline

Each attack must be demonstrated to be either CLOSED (mechanism in §3.1 above blocks it) or DOCUMENTED-OUT-OF-SCOPE (e.g., Attack 12 cold-boot is acknowledged-but-not-bench-asserted per §0 of CADUCEUS-004). No attack may be MITIGATED-PARTIALLY without explicit annotation in the substrate's compliance report.

A candidate substrate's compliance score is **binary across all 20 attacks**: 20/20 closed = bench-compliant. 19/20 closed = bench-non-compliant with the specific failed attack named.

---

## 4. Test harness

### 4.1 Test categories

| Category | Test count (approx.) | Source | Pass criterion |
|---|---|---|---|
| Per-assertion unit tests | ~150 | Bench §2 above (17 assertions × 5–15 tests each) | 100% pass |
| Attack-class regression | ~80 | Attack corpus §3 above (20 attacks × ~4 variants) | 100% pass (all 20 attacks closed) |
| WCET budget verification | ~30 | CADUCEUS-005 §2 (HF module budgets) | 100% within budget on declared hardware |
| Constant-time crypto | ~10 | T1 + libsodium hardening | No timing-channel signal above noise floor (dudect / ctgrind) |
| Formal-methods verification | ~12 theorems | CADUCEUS-006 (Lean 4 + TLA+) | Machine-checked clean |
| End-to-end mission simulation | ~50 | Synthetic 90-day mission scenarios | All scenarios complete; bench compliance maintained throughout |
| **Total** | **~332 tests** | | **100% pass for bench compliance** |

### 4.2 Reference test harness implementation

The reference test harness is part of the Phase 5 substrate build per CADUCEUS-005. As of LEDGER #0004 commit, the harness exists in specification form only; executable implementation is M3–M7 substrate work.

Candidate substrates implementing caduceus-bench must produce their own test harness OR adopt the reference harness when it ships. Either is acceptable; the harness implementation is not the benchmark — the assertions and attacks are.

### 4.3 Required reporting from a candidate substrate

A candidate substrate claiming caduceus-bench v1.2.1 compliance MUST publish:

1. **Compliance report** — explicit per-assertion pass/fail and per-attack closed/failed, with test artifacts demonstrating each.
2. **Hardware declaration** — exact compute platform, memory, RF/optical hardware, with attestation.
3. **Theoretical-floor constants** — per §3.7 derivation, the link-configuration constants used for T5.
4. **K calibration value** — per §3.8 guidance, the value of F4 hard rate limit used.
5. **Cryptographic configuration** — Ed25519 implementation library, HKDF parameters, BPSec configuration.
6. **§0 scope acknowledgments** — explicit acknowledgment of SFT scope and EPHEMERIS supply-chain assumption.
7. **Provenance trail** — chain entries documenting every benchmark execution.

Compliance reports MUST be signed (Ed25519 or equivalent) by the substrate developer and verifiable against a published public key.

---

## 5. Scoring methodology

caduceus-bench is **pass-or-fail by design** — no partial credit, no weighted scoring, no leaderboard ranking by point total. A substrate is either compliant or not.

This is intentional. The compliance-graded thesis is that authority decisions in interplanetary deployments must be either correct or fail-safe. There is no "mostly correct." Either every bundle is verified per T1, or T1 fails. Either every replay is detected per T4, or T4 fails.

The closest analog in conventional benchmarking is the CCSDS conformance-statement model: implementations claim conformance to specific protocol features, and those claims are auditable.

### 5.1 Conformance claim language

A substrate developer may publish:

> "Substrate X implements caduceus-bench v1.2.1 in full" (all 17 assertions + 20 attacks closed)

OR

> "Substrate X implements caduceus-bench v1.2.1 partially, with the following exclusions: <list>"

Partial claims are acceptable for research substrates and intermediate development states. Full claims require the §4.3 compliance report.

### 5.2 Conformance audit

External parties may verify a substrate's full conformance claim by:

1. Reproducing the SHA-256 hash of the substrate's compliance report and verifying the signature.
2. Re-running the test harness independently against the substrate.
3. Filing a non-conformance report (signed) if any assertion or attack is found non-closed.

Non-conformance reports become part of the Aletheia chain alongside conformance reports.

---

## 6. Honest scope and known limitations

Beyond the in-spec out-of-scope items in §1, caduceus-bench v1.2.1 also does not measure:

- **Quantum-state QKD interpretability** — referenced as v3.x horizon, not v1.2.1 scope.
- **Optical-terminal-specific link characteristics** — P2 specifies pointing accuracy but does not specify the broader optical-link budget; substrate must declare its own optical configuration.
- **PHRONESIS Secure Element tamper resistance** — explicitly out-of-bench per §0 of CADUCEUS-004; addressed by hardware-side assurance.
- **EPHEMERIS firmware integrity** — explicitly out-of-bench, assumed.
- **Ground master-key custody** — operational practice, not bench-asserted.
- **Crew cognitive load empirics** — F4 K=6 is mission-policy default; calibration is empirical per §3.8 of CADUCEUS-004.

These are not deficiencies of the benchmark; they are categories of property that the benchmark is not the right tool for. Each requires its own engineering, operational, or empirical-research framework.

---

## 7. Reproducibility and chain anchoring

### 7.1 Cryptographic identity

| Field | Value |
|---|---|
| Bench version | v1.2.1 |
| Source manifest | CADUCEUS-004 |
| SHA-256 (canonical UTF-8) | `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874` |
| Byte count | 20,379 |
| Ledger entry | LEDGER #0004 (pending) |
| Signing key | Visionblox release public key (Ed25519, see §3.5 of CADUCEUS-LEDGER-PREP-001) |
| Predecessor commit | LEDGER #0003 (VBX-ISPS substrate v0.1) |

### 7.2 Reproduction

To reproduce caduceus-bench v1.2.1:

1. Obtain CADUCEUS-004 from the Visionblox public repository (URL TBD post-commit).
2. Verify SHA-256 hash matches the value in §7.1.
3. Verify Ed25519 signature on LEDGER #0004 entry against the published Visionblox release public key.
4. Verify OpenTimestamps proof.
5. Cross-reference against the arXiv defensive preprint (TBD post-commit).

### 7.3 Versioning discipline

Caduceus-bench versions are immutable post-commit. A "v1.3" or "v2.0" would be a new manifest with a new SHA-256, a new ledger entry, and explicit delta justification. The v1.2.1 commit is permanent.

---

## 8. Citation

For academic or industry references to this benchmark:

> Wooden, A. K., Sr. (2026). *caduceus-bench v1.2.1 — A Compliance-Graded Bundle Fabric Benchmark for Interplanetary Authority Propagation* (CADUCEUS-004). Visionblox LLC / Zuup Innovation Lab. Aletheia LEDGER #0004, SHA-256 `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874`.

For the consolidated benchmark document (this file):

> Wooden, A. K., Sr. (2026). *caduceus-bench v1.2.1 — Consolidated Benchmark* (CADUCEUS-BENCH-001). Visionblox LLC / Zuup Innovation Lab.

---

## 9. Honest scope statement (recap)

- This document consolidates content from CADUCEUS-004 (spec), CADUCEUS-PHASE3-001/002 (attacks), and CADUCEUS-005 §5 (harness plan). It does not replace those source documents; it provides a single external-facing entry point.
- The benchmark itself is the cryptographically-committed manifest CADUCEUS-004. This consolidation is a presentation artifact, not a normative replacement.
- A candidate substrate implementing caduceus-bench v1.2.1 must reproduce the bench's pass/fail structure precisely. Reinterpretation of assertions or attacks invalidates compliance claims.
- caduceus-bench is intentionally narrow: it measures compliance-graded correctness for interplanetary authority propagation. It does not measure performance, scalability, throughput, or commercial viability. Other benchmarks apply for those properties.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-BENCH-001 Rev A · Consolidated caduceus-bench v1.2.1*
