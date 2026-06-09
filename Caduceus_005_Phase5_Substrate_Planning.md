# Caduceus v0.2 — Phase 5 Substrate Planning

| | |
|---|---|
| **Document ID** | CADUCEUS-005 |
| **Revision** | A (draft) |
| **Methodology** | ZCS-6 Phase 5 (substrate prototype plan against locked benchmark) |
| **Target bench** | caduceus-bench v1.2.1 (CADUCEUS-004, awaiting LEDGER #0004 commit) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Plan-of-work for v0.2 substrate build. Execution begins after LEDGER #0004 commit and PPA filings. |

---

## 0. Scope and ordering discipline

This document plans the **substrate prototype** that will be tested against caduceus-bench v1.2.1. ZCS-6 discipline: the bench is locked first (already done, CADUCEUS-004); the substrate builds to satisfy the bench; substrate-side gaps that surface during build are NOT cause to soften the bench — they are cause to revise the substrate plan.

Per the VBX-ISPS v0.1 → v0.2-α progression: Python substrate v0.1 passed 31/31 tests after fixing two real architectural bugs (Mercury Subleq control-flow defect; ACI alpha update sign error). Same discipline expected here.

---

## 1. Substrate architecture overview

### 1.1 Two-tier model

| Tier | Host | Compute class | Role |
|---|---|---|---|
| **PHRONESIS-side** | VBX-ISPS Jetson Orin NX (16 GB, shielded enclosure, 2 kg) | Edge AI / general-purpose | Bundle fabric terminator; runs all HF-16 through HF-22 floors |
| **EPHEMERIS-side** | VBX-01 (ARM Cortex-M class) | MCU | Crew-attestation peer; verifies signatures + URN; displays + audits |

### 1.2 Layered stack (PHRONESIS-side)

```
┌─────────────────────────────────────────────────────────┐
│  HF-22  Frame-context interlock (F4 enforcement)        │
├─────────────────────────────────────────────────────────┤
│  HF-21  Nonce-seen-set management (T4)                  │
├─────────────────────────────────────────────────────────┤
│  HF-17  Inbound verification + Civium gate (T1-T7)      │
│  HF-19  Frame disambiguation + URN parsing (F1-F3)      │
│  HF-16  Outbound bundle attestation (T1)                │
├─────────────────────────────────────────────────────────┤
│  HF-18  ACI drift monitor (T5)                          │
├─────────────────────────────────────────────────────────┤
│  HF-20  Mercury-bounded WCET enforcement                │
├─────────────────────────────────────────────────────────┤
│  BP7 (RFC 9171) + BPSec (RFC 9172)                      │
│  Reference impl: ION-DTN (NASA, open source)            │
├─────────────────────────────────────────────────────────┤
│  Existing PHRONESIS substrate                           │
│  Civium · MVCI · Aletheia DAC · Mercury kernel · Bus    │
│  ISO · Secure Element                                   │
├─────────────────────────────────────────────────────────┤
│  Jetson Orin NX hardware (shielded, 16 GB, ≤200 W peak) │
└─────────────────────────────────────────────────────────┘
```

EPHEMERIS-side runs a thin verification + display layer paired over UWB or BLE.

---

## 2. Module-by-module build plan

### 2.1 HF-16 — Outbound bundle attestation

| | |
|---|---|
| **Bench obligation** | T1 (sign canonical tuple) |
| **Implementation** | Rust crate `caduceus-attest` |
| **Dependencies** | libsodium (constant-time Ed25519), existing Aletheia signing service, payload_class enumeration crate |
| **Key implementation detail** | Construct canonical signed tuple via `serde_canonical_cbor` or equivalent deterministic serialization; pass to libsodium `crypto_sign_ed25519` |
| **WCET budget (per HF-20)** | ≤ 500 µs on Orin NX for 1 KB payload |
| **Estimated build effort** | 2 weeks (1 engineer) |
| **Test against bench** | Sign 10,000 bundles; verify signatures with independent verifier; constant-time analysis via `dudect` or `ctgrind` |

### 2.2 HF-17 — Inbound verification + Civium authority gate

| | |
|---|---|
| **Bench obligation** | T1 (4-check atomic verification), T2 (authority + time-bound), T3 (fail-safe to HF-9), T7 (revocation + succession-implies-revocation + stale-graph defaults) |
| **Implementation** | Rust crate `caduceus-gate` |
| **Dependencies** | `caduceus-attest`, existing Civium authority graph, existing MVCI gate framework, Mercury WCET-bounded scheduler |
| **Key implementation detail** | Per-authority-stream mutex for the 4-check sequence; separate revocation-stream priority handler; stale-graph timer per stream; conservative-defaults table from §3.6 hard-coded with cryptographic registry hash |
| **WCET budget (per HF-20)** | ≤ 2 ms for the 4-check sequence on Orin NX; ≤ 5 ms including gate decision |
| **Estimated build effort** | 6 weeks (1 engineer + 1 reviewer); largest module |
| **Test against bench** | Inject all 20 attack patterns from CADUCEUS-PHASE3-001/002; verify 100% closure per the bench compliance matrix |

### 2.3 HF-18 — ACI drift monitor

| | |
|---|---|
| **Bench obligation** | T5 (link-integrity floor, alpha-bounded, theoretical + empirical floor) |
| **Implementation** | Rust crate `caduceus-aci`; reuses existing Aletheia ACI module from VBX-ISPS v0.1 |
| **Dependencies** | Per-link theoretical-floor constants (per §3.7 derivation procedure, computed per link configuration), existing ACI v0.2 implementation |
| **Key implementation detail** | Alpha lower-bound = 1/(window_size × physics_floor_margin); theoretical-floor constants loaded at boot from signed configuration; max(empirical, theoretical) gating logic |
| **WCET budget (per HF-20)** | ≤ 100 µs per RF-symbol-batch update |
| **Estimated build effort** | 2 weeks (1 engineer) — most reuse of existing ACI |
| **Test against bench** | Slow-drift adversarial simulation over 10,000 hours of synthetic link telemetry; verify floor holds at theoretical bound |

### 2.4 HF-19 — Frame disambiguation + URN parsing

| | |
|---|---|
| **Bench obligation** | F1 (URN + envelope), F2 (closed class namespace + signed class), F3 (EPHEMERIS independent verification), §3.5 (parsing requirements, dual-version retention) |
| **Implementation** | Rust crate `caduceus-bodyid` |
| **Dependencies** | `vbx-body` URN parser (new), v1.0 registry as signed-data blob, EBNF parser combinator |
| **Key implementation detail** | EBNF strict-parse via `pest` or `nom`; NFC + ASCII-only normalization before lookup; dual-version registry pointer during transition windows; registry-version-transition bundle class handler |
| **WCET budget (per HF-20)** | ≤ 200 µs per URN parse + registry lookup |
| **Estimated build effort** | 3 weeks (1 engineer) |
| **Test against bench** | Namespace-exhaustion attack suite (Attack 8); registry-transition-at-light-time simulation (Attack 27); 1,000,000 URN parse fuzzing |

### 2.5 HF-20 — Mercury-bounded WCET enforcement

| | |
|---|---|
| **Bench obligation** | All assertions that require bounded execution time (T1, T3, T4, T6, F3, F4, B2) |
| **Implementation** | Integration layer; Mercury kernel provides the substrate |
| **Dependencies** | Existing Mercury Subleq kernel (Rust, with v0.2-β Lean 4 proof in progress) |
| **Key implementation detail** | Each HF module declares its WCET budget; HF-20 runtime enforces budgets via Mercury's deterministic 5-cycle counter; comm-peak / compute-peak non-concurrence interlock implemented as Mercury scheduling constraint |
| **WCET budget** | Itself — meta-floor |
| **Estimated build effort** | 4 weeks (1 engineer + Mercury collaboration); see CADUCEUS-006 for formal-methods composition |
| **Test against bench** | All HF modules' WCET budgets verified empirically; Mercury-bounded execution traced via instrumented runs |

### 2.6 HF-21 — Nonce-seen-set management

| | |
|---|---|
| **Bench obligation** | T4 (per-stream partitioned nonce-set with bounded quota) |
| **Implementation** | Rust crate `caduceus-nonce-set` |
| **Dependencies** | per-stream-quota config, LRU-among-expired-TTL eviction policy |
| **Key implementation detail** | Per-stream HashMap with TTL-indexed secondary structure; eviction never touches live-TTL nonces; degraded-replay-protection alert raised on per-stream quota exceedance |
| **WCET budget (per HF-20)** | ≤ 50 µs per nonce check |
| **Estimated build effort** | 2 weeks (1 engineer) |
| **Test against bench** | Live-TTL flood simulation (Attack 23); cross-stream isolation verification |

### 2.7 HF-22 — Frame-context interlock with root-cause clustering

| | |
|---|---|
| **Bench obligation** | F4 (root-cause clustering + hard rate limit + audit logging) |
| **Implementation** | Rust crate `caduceus-interlock` |
| **Dependencies** | EPHEMERIS peer protocol over UWB/BLE, Aletheia audit-write path, persistent state indicator |
| **Key implementation detail** | Root-cause tuple computed at every mismatch; clustering map keyed by tuple; sliding-window confirmation counter per crew member; outright-rejection branch when rate limit exceeded; audit-log every grant/deny/timeout |
| **WCET budget (per HF-20)** | ≤ 300 µs per mismatch detection + clustering decision |
| **Estimated build effort** | 4 weeks (1 engineer; UX collaboration for persistent indicator design) |
| **Test against bench** | 90-day adversarial flood simulation (Attack 26); confirmation-fatigue 0-cascade verification |

### 2.8 EPHEMERIS-side peer module

| | |
|---|---|
| **Bench obligation** | F3 (independent URN verification), F4 (crew-attestation), T1 (independent signature verification) |
| **Implementation** | C / embedded Rust on Cortex-M class MCU |
| **Dependencies** | embedded libsodium build; cached v1.0 namespace registry; UWB/BLE driver; NFC pairing |
| **Key implementation detail** | Receives peer-bundle copy from PHRONESIS containing (bundle_id, summary, signature, vbx-body-URN); independently verifies signature against cached release public key; independently parses URN; displays body + frame on watch face ring; alerts on signature failure, URN parse failure, or PHRONESIS-EPHEMERIS URN mismatch |
| **WCET budget** | Soft real-time; ≤ 100 ms from receive to display |
| **Estimated build effort** | 6 weeks (1 firmware engineer); largest EPHEMERIS-side delta |
| **Test against bench** | EPHEMERIS-PHRONESIS pairing MitM simulation (Attack 6); dual-peer attestation if v0.2+ stretch is in scope |

---

## 3. Reference open-source dependencies (locked at planning time)

| Component | Source | Version target | Role |
|---|---|---|---|
| **ION-DTN** | NASA / Github open source | latest stable (3.7.x as of planning) | BP7 reference implementation |
| **libsodium** | jedisct1/libsodium | 1.0.20+ | Constant-time Ed25519, SHA-256 |
| **HKDF** (via libsodium) | RFC 5869 | as in libsodium | BPSec session key derivation |
| **Rust toolchain** | rust-lang | stable + nightly for bench tooling | Substrate language |
| **`pest` or `nom`** | crates.io | latest stable | EBNF URN parser |
| **`serde` + `ciborium`** | crates.io | latest stable | Canonical CBOR serialization for signed tuples |
| **OpenTimestamps client** | opentimestamps.org | latest | Provenance ledger anchoring |
| **Mercury Subleq kernel** | Visionblox/Zuup internal | v0.2 (Lean 4 proof in progress) | Deterministic WCET substrate |
| **Aletheia DAC** | Visionblox/Zuup internal | v0.2 → v0.3 needed for cross-key-epoch | Provenance signing + chain |
| **Civium authority graph** | Visionblox/Zuup internal | v0.2 → needs revocation-stream extension | Authority decision substrate |

---

## 4. Build sequence and milestones

### 4.1 Milestone schedule

| Milestone | Weeks | Deliverables | Gate |
|---|---|---|---|
| **M0 — LEDGER commit complete** | 0 | CADUCEUS-004 committed; PPAs filed; arXiv preprint published | PPA filings confirmed |
| **M1 — Dependencies in place** | 1–3 | Aletheia v0.3 chain extensions; Civium revocation-stream extension; theoretical-floor constants computed per §3.7 for at least one link config | Aletheia v0.3 passes its own bench |
| **M2 — Outbound + parsing modules** | 3–6 | HF-16, HF-19, HF-21 implementations | Unit tests pass; constant-time analysis clean |
| **M3 — Verification + gate modules** | 5–11 | HF-17, HF-18 implementations | Phase 3 attack-suite execution; 100% closure verified |
| **M4 — WCET integration** | 9–13 | HF-20 Mercury integration; comm-peak/compute-peak interlock | All HF modules' WCET budgets verified empirically |
| **M5 — EPHEMERIS peer** | 7–13 | EPHEMERIS-side firmware; UWB/BLE pairing | Attack 6 simulation passes |
| **M6 — Frame-context interlock** | 11–15 | HF-22 implementation; persistent state indicator UX | 90-day adversarial flood simulation passes |
| **M7 — Full-stack integration test** | 14–16 | All HF modules integrated; bench v1.2.1 compliance suite executed end-to-end | **PHASE 3 RE-EXECUTION ON SUBSTRATE.** Any failures → architecture-level fix per ZCS-6 (no benchmark softening) |
| **M8 — Phase 6 vertical-integration attacks** | 17–20 | Adversarial probe against the substrate per CADUCEUS-003 §7 + §7 of CADUCEUS-004 | Substrate v0.2 → v0.3 iteration if Phase 6 finds material defects |

**Total duration estimate:** 20 weeks (5 months) for a 2-engineer team plus 1 firmware engineer for EPHEMERIS-side.

### 4.2 Critical-path analysis

- **M3 → M7 critical path** (verification + gate is the largest module and the deepest in dependencies).
- M4 (Mercury WCET integration) depends on Mercury v0.2-β Lean 4 proof reaching machine-checked state OR on accepting a "verified informal" baseline for the substrate build. See CADUCEUS-006 for the formal-methods composition decision.
- M1 dependencies (Aletheia v0.3, Civium revocation-stream) are parallel-buildable but each has its own bench-class discipline. **Recommendation:** scope these as sister projects with their own bench manifests; don't conflate with caduceus-bench.

---

## 5. Test harness and bench-compliance matrix

The test harness for v0.2 substrate compliance:

| Test class | Source | Test count (approximate) | Pass criteria |
|---|---|---|---|
| **Per-assertion unit tests** | Bench §3.1–§3.4 (17 assertions) | ~150 tests (5–15 per assertion) | 100% pass |
| **Attack-class regression** | CADUCEUS-PHASE3-001 (12 attacks) + CADUCEUS-PHASE3-002 (8 attacks) | 20 attack scenarios with multiple variants each, ~80 tests | 100% pass (all 20 attacks closed) |
| **WCET budget verification** | HF module budgets per §2 above | ~30 timing tests | 100% within budget on Orin NX representative load |
| **Constant-time crypto** | libsodium hardening | dudect + ctgrind on signing and verification paths | No timing-channel signal above noise floor |
| **Formal-methods verification** | CADUCEUS-006 (separate doc) | Lean 4 + TLA+ proofs of T1, T3, T7, F4, §3.6 | Machine-checked clean |
| **End-to-end mission simulation** | New: 90-day synthetic mission scenario | ~50 scenario tests | All scenarios complete; bench compliance maintained throughout |

**Total target test count:** ~330. Adapted from the VBX-ISPS 31/31 test pattern (smaller test surface there because VBX-ISPS bench had 6 assertions vs. caduceus-bench's 17).

---

## 6. Resource and team

### 6.1 Required staffing

| Role | FTE | Duration | Notes |
|---|---|---|---|
| Senior Rust engineer | 1.0 | 20 weeks | HF-17, HF-22 are largest modules; needs depth |
| Mid-level Rust engineer | 1.0 | 16 weeks | HF-16, HF-18, HF-19, HF-21 |
| Embedded firmware engineer | 0.5–1.0 | 12 weeks | EPHEMERIS-side peer |
| Formal-methods researcher | 0.5 | 24 weeks (parallel) | Lean 4 + TLA+ proof track; see CADUCEUS-006 |
| Radio engineer | 0.25 | 4 weeks | Theoretical-floor constants per §3.7 derivation |
| **Total** | **~3.25 FTE** | **20 weeks** | Approx. 65 person-weeks |

### 6.2 Cost order-of-magnitude

At loaded labor rate $200K/yr fully-burdened for senior, $130K/yr mid-level, $150K/yr embedded:
- ~$520K direct labor for the substrate build
- Add hardware/lab/testing infra: ~$50K (Orin NX dev kits, EPHEMERIS prototypes, radio test gear)
- Add Lean 4 / formal-methods contractor: ~$80K
- **Total**: ~$650K rough order

Federal-capture posture (CADUCEUS-CAPTURE-001) addresses how this is funded. SBIR Phase II ceiling ($1.25M typical) is sufficient.

---

## 7. §6.4 budget cap compliance (mass and power)

Locked cap: 4.0 kg total comm subsystem / 50 W steady / 100 W peak TX.

Substrate v0.2 software-only on existing HF-11 hardware: **passes cleanly**.
- 0 kg delta; existing HF-11 is the comm hardware.
- ~10–15 W compute delta on Orin NX for the full HF-16 through HF-22 stack (well within 120 W steady envelope).

Optical-terminal track (deferred to v0.3): requires partner engagement under §6.4 caps as exit criteria.

---

## 8. Open partner-engagement items

1. **Optical terminal partner** (v0.3+) — Mynaric, BridgeComm, Astrogate Labs. Must commit to ≤ 4 kg subsystem inclusive.
2. **Formal-methods collaboration** — Lean 4 / TLA+ practitioners. Mercury kernel team has the proof in progress; Caduceus composes on top. See CADUCEUS-006.
3. **Suit hardware partner** — ILC Dover / Collins Aerospace / equivalent. PHRONESIS hardware partnering remains open per VBX-ISPS-001 §HF-15 caveat.
4. **External mission-assurance review** — NASA OSMA-class reviewer for the v0.2 → v0.3 transition. Not gating commit but recommended for crewed-deployment readiness.

---

## 9. Honest scope statement

- This plan assumes successful LEDGER #0004 commit on CADUCEUS-004 and successful PPA filings. Both are prerequisites.
- 20-week duration is for an experienced team executing in parallel. Solo-developer execution would be ~3–4x longer.
- Aletheia v0.3 and Civium revocation-stream extensions are sister projects with their own discipline; this plan assumes they're available by M1. If not, M1–M3 dates slip.
- The 90-day adversarial flood simulation (M6) requires synthesizing mission-realistic adversarial bundle patterns. Existing red-team tooling may need extension.
- v0.2 is the first crewed-mission-class build; v0.3 (with optical terminal and dual-fault-tolerance) is the realistic crewed-deployment target. v0.2 enables LEO/cislunar crewed demonstrations; v0.3 enables Mars-class crewed deployment.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-005 Rev A · Phase 5 substrate planning*
