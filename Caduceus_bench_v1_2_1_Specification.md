# Caduceus v0.1 — Interplanetary Connectivity Substrate
## caduceus-bench v1.2.1 specification (LEDGER #0004 commit manifest)

| | |
|---|---|
| **Document ID** | CADUCEUS-004 |
| **Revision** | A |
| **Methodology** | ZCS-6 (Phase 1 + Phase 2 — final commit form) |
| **Predecessor** | CADUCEUS-003 (v1.2, found 5 enablement/scope-clarity items at Move 3 diversified-internal probe) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **LEDGER #0004 commit manifest.** All Category A tightenings from CADUCEUS-PROBE-001 applied. v1.2's Phase 3 convergence (§4 below) carries forward unchanged — tightenings introduce no new attack surface per probe synthesis §4.3. |
| **Delta source** | CADUCEUS-PROBE-001 Category A items: P1.3, P1.4, P3.6, P3.9, P3.4/P3.5 acknowledgment |

---

## 0. Scope statement

This document is a **commit-ready specification**, not an implementation.

"Interplanetary internet" is shorthand for **delay-tolerant store-and-forward bundle fabric** (DTN/BP7). TCP/IP is not interplanetary-useful.

"Teleportation" is treated in only one physical sense — **quantum-state teleportation for QKD**. Matter teleportation and FTL information transfer remain out of scope. Quantum-state teleportation interplanetary QKD is [SPECULATIVE] and deferred to v3.x horizon.

**EPHEMERIS firmware integrity and tamper-evident packaging are assumed** and are out-of-scope of caduceus-bench. EPHEMERIS supply-chain attestation is a parallel requirement on VBX-01 v0.3+ firmware signing.

**Caduceus enforces key-authority asymmetry.** PHRONESIS units hold per-suit telemetry-signing keys only. Master command-authority keys are held by ground signers and are never deployed to suits. Suit capture is bounded to per-suit telemetry forgery; suit capture cannot escalate to command authority over other crew. Ground master-key compromise remains a total-breach scenario; mitigation is operational (multi-party master-key custody), not bench-asserted.

**[NEW v1.2.1] Fault-tolerance scope.** caduceus-bench v1.2.1 specifies a **single-fault-tolerant (SFT)** authority verification architecture. NASA crewed-mission practice requires dual-fault-tolerance (DFT) for command-authority-critical paths. DFT for Caduceus is provided by v0.2+ substrate work specifically through three additional mechanisms, none of which are asserted by the bench but all of which are documented as preconditions for any crewed deployment:

1. **Dual-peer EPHEMERIS attestation** — two crew EPHEMERIS units must independently agree on (URN, signature) for high-stakes ops; replaces the v1.x single-peer model.
2. **Triple-redundant Aletheia chain storage** — chain state replicated across three physical storage media (e.g., Orin NX flash + Secure Element + EPHEMERIS-side cache) with quorum-recovery semantics.
3. **Authority-graph redundant storage** — local Civium authority graph replicated with the same triple-redundant pattern, quorum-verified before any T2 gate decision.

The bench commits in v1.2.1 to SFT. Crewed-deployment readiness requires the three v0.2+ substrate-side DFT mechanisms.

---

## 1–2. Architectural framing and Phase 1 whitespace

Unchanged from CADUCEUS-001 §§1–2. The six-attribute whitespace map (§1.3) holds.

---

## 3. ZCS-6 Phase 2 — caduceus-bench v1.2.1 (final pre-commit form)

**Seventeen assertions, four layers.** Same count as v1.2; v1.2.1 tightens 3 of them (T3, T5, F4) and adds enablement detail to B2.

### 3.1 Physical layer (P-class) — unchanged

P1, P2, P3 as in CADUCEUS-001 §3.1.

### 3.2 Bundle layer (B-class) — B2 enablement tightening

| ID | Assertion | Threshold | Δ from v1.2 |
|---|---|---|---|
| B1 | BP7 (RFC 9171) wire-format conformance | 100% bit-exact, all required fields parsed | — |
| **B2** | **Store-and-forward integrity through ≥ 3 hops with simulated 60-min connectivity outage between hops 2 and 3, AND with bundle fragmentation invoked under BPSec (RFC 9172) using session keys derived from Aletheia per-stream signing keys via HKDF (RFC 5869) — specifically: BPSec session keys = HKDF-Expand(HKDF-Extract(salt = stream_id ∥ epoch_tai, IKM = Aletheia_stream_signing_key), info = "caduceus-bpsec-v1" ∥ purpose_label, L = 32 bytes). Per-session key derivation is one-way; compromise of a BPSec session key does not compromise the Aletheia parent signing key. No separate BPSec key infrastructure is maintained — Aletheia is the single trust anchor. Receiver produces zero side-effects until full reassembly AND full signature verification.** | **100% custody-transferred bundles delivered; 100% partial-fragment side-effects blocked; 0% BPSec session-key compromise propagating to parent Aletheia key under simulated key-leak fuzz tests** | **revised: enablement-grade HKDF derivation specified** |
| B3 | Custody-transfer semantics under intermittent loss → explicit failure notification | 100% expired bundles produce custody-failure status report | — |

### 3.3 Trust layer (T-class) — T3 and T5 tightened

| ID | Assertion | Threshold | Δ from v1.2 |
|---|---|---|---|
| T1 | Constant-time Ed25519 signature over canonical signed tuple (bundle_id ∥ payload_class ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce) + BP7 lifetime; four-check atomic verification at receiver | 100% outbound signed; 100% atomic verification; 100% constant-time | — |
| T2 | Inbound verification against current authority graph AND time-bound validity assertion; both must succeed | 100% gate decisions logged with both checks; 0% pre-gate side effects | — |
| **T3** | **Signature failure OR gate denial deterministically transitions to HF-9 (hard safety floor). HF-9 normative behavior, inlined here for self-containment of this commit manifest:** **(a) receiver halts all non-life-support comm processing on the affected authority stream; (b) alert surfaced to crew via EPHEMERIS haptic + LED AND PHRONESIS HUD AND backup audio; (c) HF-9 entry is logged to Aletheia chain with full bundle context, failure cause, and TAI epoch; (d) exit from HF-9 requires explicit two-source confirmation — either (crew + ground) OR (crew + second-EPHEMERIS dual-peer), confirmation cryptographically signed; (e) life-support subsystems (O2/CO2/thermal control) are not affected by HF-9 entry and continue per their own safety floors per VBX-ISPS-001; (f) HF-9 state is reversible per (d) but never auto-reversible; (g) repeated HF-9 entries within a mission-policy interval trigger escalation alert independent of individual entry resolutions.** No LLM-mediated retry path. | 100% fail-safe transitions logged; ≤ Mercury-WCET bound from failure detection to floor entry; 0% auto-reversal; 100% two-source confirmation required for exit | **revised: HF-9 behavior inlined for self-contained commit manifest** |
| T4 | Per-stream-partitioned nonce-seen-set with bounded per-stream quota; per-stream eviction with degraded-replay-protection alert; global exhaustion → HF-9 | 100% replay attempts logged; 0% cross-stream contamination; degraded-replay alerts surface ≤ 100 ms | — |
| **T5** | **Link-integrity floor: ACI's conformal bandwidth on (BER, SNR, RTT) is bounded above by mission-policy maximum and below by max(empirical-observed-floor, theoretical-floor). Theoretical-floor is computed from first-principles link characteristics per the procedure in §3.7 [NEW] and registered as a per-link-configuration constant; empirical calibration cannot drive the effective floor below the theoretical bound.** Floor updates require v1.x bump. | 100% bandwidth excursions alerted; 0% sub-theoretical-floor band acceptance; 100% theoretical-floor derivation reproducible per §3.7 | **revised: theoretical-floor derivation procedure specified in §3.7** |
| T6 | Ingress validity floor: F1 verified before any T/B/render processing; out-of-envelope quarantined | 100% F1 checks before T/B/render; 100% out-of-envelope quarantined | — |
| T7 | Revocation propagation via dedicated priority-lane stream with freshness tokens; stale-graph defaults per §3.6; succession-implies-revocation rule | 100% revocations in priority lane; 100% successions implicitly revoke predecessors; 0% predecessor acceptance after successor local | — |

### 3.4 Frame layer (F-class) — F4 tightened with audit logging

| ID | Assertion | Threshold | Δ from v1.2 |
|---|---|---|---|
| F1 | (TAI_microseconds, vbx-body-URN) tuple in BP7 extension; envelope [1972-01-01, 2101-01-01) TAI; out-of-envelope quarantined at ingress | 100% bundles carry valid tuple; out-of-envelope quarantined | — |
| F2 | Action-inducing bundles gated by URN-context match against authorized set; class signed-integrity verified; unenumerated classes rejected | 100% unauthorized cross-context rejected; 100% unenumerated-class rejected; 0% post-signature class manipulation accepted | — |
| F3 | EPHEMERIS independently verifies URN; mismatch with PHRONESIS-displayed URN triggers alert | ≤ 100 ms mismatch detection latency over UWB | — |
| **F4** | **Frame-context interlock for non-command bundles: side-effect paths require URN-context verification; mismatch raises crew alert AND blocks side-effect until crew confirms via EPHEMERIS peer. Root-cause clustering: shared (mismatch-class, expected-body, actual-body, expected-frame, actual-frame) tuple. Hard rate limit: ≤ K confirmations per crew member per hour (default K=6, mission-policy-set per §3.8 [NEW] calibration guidance). Bundles in mismatched-frame exceeding K rejected outright until next authority-graph refresh. CREW CONFIRMATION EVENTS — BOTH GRANTS AND DENIALS — ARE RECORDED AS ALETHEIA CHAIN ENTRIES INCLUDING: full bundle context, crew member identifier, EPHEMERIS device identifier, confirmation TAI timestamp, confirmation outcome (grant/deny/timeout). Audit-trail completeness 100% — no confirmation event omitted from chain.** | 100% side-effect paths interlocked; ≤ K confirmations/crew/hour; 0% confirmation-fatigue cascade in 90-day adversarial flood simulation; **100% confirmation events appear in Aletheia chain** | **revised: explicit confirmation-event audit logging** |

### 3.5 Caduceus-BodyID v1.0 — namespace

Unchanged from CADUCEUS-003 §3.5: URN scheme, grammar enforcement, Unicode NFC + ASCII-only canonical, strict registry match, atomic transitions, dual-version retention window (default 2 × max-RTT, minimum 60 min), registry-version-transition bundle class on priority lane.

### 3.6 Conservative-defaults table

Unchanged from CADUCEUS-003 §3.6: full enumeration table, precedence ordering from most-restrictive to least-restrictive, Emergency-class and Revocation bypass rules.

### 3.7 [NEW v1.2.1] Theoretical-floor derivation procedure (enablement for T5)

The T5 theoretical-floor for each tracked link metric is derived from first-principles link physics as follows:

**SNR theoretical-floor (dB):**

```
SNR_theoretical_floor = Pr - N0_total
where:
  Pr        = received signal power (dBW) = Pt + Gt - L_path + Gr - L_system - L_pointing - L_atmospheric
  Pt        = transmit power (dBW)
  Gt, Gr    = transmit and receive antenna gains (dBi)
  L_path    = free-space path loss (dB) = 20*log10(4*pi*d/lambda)
            = 20*log10(d_km) + 20*log10(f_MHz) + 32.45 dB
  d         = slant range (km)
  f         = carrier frequency (MHz)
  L_system  = receiver implementation loss (typ. 2-3 dB)
  L_pointing = antenna mispointing loss (function of pointing accuracy P2 budget)
  L_atmospheric = atmospheric attenuation (function of body atmosphere; ~0 dB for vacuum, increases for Mars, etc.)
  N0_total  = total noise spectral density (dBW/Hz) = 10*log10(k*T_system) where
              k = 1.380649e-23 J/K (Boltzmann constant)
              T_system = system noise temperature (Kelvin) = T_antenna + T_receiver*(1 - 1/G_LNA)
              T_antenna varies by pointing direction (cold sky ~10K; Sun-pointed >>1000K)
              T_receiver depends on LNA + receiver chain
```

For each operational link configuration (frequency, antenna characteristics, link geometry), the theoretical SNR floor is computed and stored as a configuration constant.

**BER theoretical-floor:**

For coherent demodulation, BER at the Shannon-bound is a function of Eb/N0 and modulation scheme:

```
BER_theoretical_floor:
  QPSK uncoded:    erfc(sqrt(Eb/N0)) / 2 (lower bound; assumes ideal demodulation)
  QPSK with coding (e.g., LDPC rate-1/2): improves by ~5-6 dB at BER=1e-6 operating point
```

The theoretical-floor BER for a given configured Eb/N0 budget is the lowest BER physically achievable under that budget; any observed BER below this is anomalous and triggers T5's adversarial-rejection logic.

**RTT theoretical-floor:**

```
RTT_theoretical_floor = 2 * range / c
where:
  range = current line-of-sight distance to peer (m)
  c     = 299,792,458 m/s (vacuum speed of light)
```

Plus any unavoidable processing delays at intermediate relays (relay buffering, retransmission timeouts, queueing). The theoretical floor is the absolute minimum; any observed RTT below this is impossible and indicates timing manipulation.

**Registration and update:** the theoretical floors for each link configuration are stored as `caduceus_link_config_v1_0` constants in the substrate repo, signed by the v1.x release key, and committed alongside the bench. Updates require a v1.x bump with delta justification. Empirical observations are tracked separately and may not override the theoretical floors — they may only ratchet upward (tighter empirical floor than theoretical is allowed; looser is not).

### 3.8 [NEW v1.2.1] F4 K calibration guidance

The hard rate limit K=6 confirmations/crew/hour is a default; mission-policy may set K per operational context. **Calibration guidance (advisory, not normative):**

- Apollo and Artemis EVA cognitive-load data (where accessible) suggest crew can sustain ~4–10 routine confirmation events per hour during nominal EVA operations without cognitive overload, with higher tolerance during ground-rest periods and lower tolerance during high-workload moments (e.g., during translation, equipment swap, contingency response).
- Mission-policy should set K per phase: K_routine ≥ K_high_workload, with K_emergency_phase set high or unlimited for emergency-class bypasses.
- Empirical validation against actual EVA workload data and ground-simulation telemetry should precede crewed deployment. The bench commit does not depend on this empirical work; the bench commits to the rate-limit *mechanism*, with the *value* of K being mission-policy-set.

This calibration guidance is non-normative and may be revised without a v1.x bump.

---

## 4. ZCS-6 Phase 3 — Final sweep convergence (carried forward from CADUCEUS-003 §4)

All 20 attacks across two sweep iterations remain closed. The 5 v1.2.1 tightenings introduce no new attack surface:

| v1.2.1 tightening | New attack surface analysis | Verdict |
|---|---|---|
| B2 HKDF derivation specified | HKDF (RFC 5869) is mature; per-session key derivation isolates compromise | NOT MATERIAL |
| T3 HF-9 behavior inlined | Inlining changes documentation, not behavior; HF-9 was already normative via VBX-ISPS-001 reference | NOT MATERIAL |
| T5 theoretical-floor derivation specified | More explicit derivation reduces implementation ambiguity; does not change rejection logic | NOT MATERIAL |
| F4 confirmation audit logging | Audit logging strengthens forensic capability; adds chain write load that fits within existing chain throughput budget | NOT MATERIAL |
| §0 SFT scope acknowledgment | Documentation change only; SFT was implicit in v1.2 and is now explicit | NOT MATERIAL |

**caduceus-bench v1.2.1 passes ZCS-6 Phase 3.** All previous attack closures hold. v1.2.1 is the LEDGER #0004 commit manifest.

---

## 5. ZCS-6 Phase 4 — Deterministic defense (commit plan)

1. Freeze CADUCEUS-004 markdown (this document) as the committed manifest.
2. Compute SHA-256 hash of the canonical UTF-8 byte sequence.
3. Ed25519 sign the hash with the Visionblox release key (constant-time implementation, per T1).
4. Append to Aletheia chain as **LEDGER #0004**, sequential after VBX-ISPS LEDGERs #0001–#0003.
5. Submit OpenTimestamps proof on the ledger entry.
6. **Sequence-critical:** file PPAs (CADUCEUS-IP-001 and follow-on Attack 27 + F4 root-cause clustering PPAs) **before** any arXiv defensive preprint, to preserve Paris Convention foreign-filing rights.
7. Publish arXiv defensive preprint after PPA filings, before any v0.1 substrate code is written.

---

## 6. ZCS-6 Phase 5 — Substrate sketch

Unchanged from CADUCEUS-003 §6. PHRONESIS-side floor map HF-16 through HF-22 carries forward. v1.2.1 adds implementation requirements:

- HF-17: HKDF-Expand/Extract implementation for B2 key derivation; HF-9 entry/exit state machine per T3 inlined behavior
- HF-18: theoretical-floor constants per link configuration loaded at boot; max(empirical, theoretical) floor selection logic
- HF-22: confirmation-event Aletheia chain write path per F4 audit logging

§6.4 mass and power budget cap (4.0 kg / 50 W steady / 100 W peak TX) unchanged.

---

## 7. ZCS-6 Phase 6 — Anticipated vertical-integration attacks (queued)

Unchanged from CADUCEUS-002 §7 plus the v1.1-specific candidates. Additional v1.2.1-specific candidates:

- HKDF key-derivation correctness under arbitrary stream_id + epoch inputs
- HF-9 exit two-source confirmation under adversarial confirmation-stream manipulation
- Confirmation-event chain-write throughput under sustained F4 mismatch flood

---

## 8. Cross-references

Unchanged from CADUCEUS-002 §8. Additions:

| Component | Reference | Relationship |
|---|---|---|
| HKDF | RFC 5869 | Required for B2 BPSec session-key derivation from Aletheia parent keys |
| VBX-ISPS-001 §HF-9 | VBX-ISPS blueprint | Source of HF-9 behavior now inlined in T3 |

---

## 9. Open items carried to v0.2 and beyond

Items 1–18 from CADUCEUS-002 §9 / CADUCEUS-003 §9. v1.2.1 closes the following open items via the tightenings:

- ~~Open item #5 (CCSDS reserved-range allocation request) — unchanged, still open~~
- ~~Open item #13 (BPSec key-rooting in Aletheia infrastructure) — CLOSED in B2 v1.2.1 with HKDF derivation~~

Remaining open items unchanged. New open items:

19. **Theoretical-floor configuration constants** — derive and register per-link-configuration constants for each Caduceus operating link (Earth-Mars S-band, Earth-Mars X-band, Earth-Moon UHF, Earth-Moon optical) before v0.1 substrate operation. ~1–2 weeks of radio engineering work.
20. **HF-9 exit two-source confirmation protocol** — specify the cryptographic structure of the two-source confirmation per T3 (e.g., is it a joint-signed bundle? Two independent bundles? Threshold scheme?). v0.2 substrate work.
21. **Empirical K calibration** — per §3.8 guidance; mission-policy track, not benchmark track.

---

## 10. Honest scope statement (final, commit form)

- v1.2.1 is the **commit manifest for LEDGER #0004**.
- All Category A tightenings from CADUCEUS-PROBE-001 are applied; no Category A items remain open.
- Category B items (prior-art search, TESS check, PPA filings, K calibration) and Category C items (Lean 4 / TLA+ formal-methods, dual-peer EPHEMERIS, external review engagement) are explicitly **post-commit work**.
- v1.2.1 specifies a single-fault-tolerant authority architecture. Crewed deployment requires v0.2+ substrate dual-fault-tolerance mechanisms (three named in §0).
- Two interplanetary-specific patent-defensible novelties (Attack 10, Attack 27) and one general novelty (F4 root-cause clustering with hard rate limit under high-RTT mission ops) are identified for PPA filing **before** arXiv defensive preprint, to preserve foreign-filing rights.
- LEDGER #0004 commit is not auto-executed by this document. The commit is a deliberate next step requiring user confirmation, PPA filing sequencing, and execution of §5 steps 1–7.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-004 Rev A · caduceus-bench v1.2.1 (LEDGER #0004 commit manifest)*
*Status: commit-ready pending PPA filing sequencing*
