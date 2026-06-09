# Caduceus v0.1 — Interplanetary Connectivity Substrate
## caduceus-bench v1.1 specification

| | |
|---|---|
| **Document ID** | CADUCEUS-002 |
| **Revision** | A |
| **Methodology** | ZCS-6 (Phase 1 + Phase 2 — first revision of bench) |
| **Predecessor** | CADUCEUS-001 (v1.0, found gap at Phase 3 sweep) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Pre-commit draft. Subject to Phase 3 re-sweep before LEDGER #0004 eligibility. |
| **Delta source** | CADUCEUS-PHASE3-001 (sweep report on v1.0) |
| **Cross-references** | CADUCEUS-001 §1–§2 retained without change; only delta sections restated here |

---

## 0. Scope statement (v1.1 additions in **bold**)

This document is a **specification and benchmark pre-commit**, not an implementation.

This document treats "interplanetary internet" as shorthand for **delay-tolerant store-and-forward bundle fabric** (DTN / BP7). TCP/IP is not interplanetary-useful.

This document treats "teleportation" in only one physical sense — **quantum-state teleportation for QKD**. Matter teleportation and FTL information transfer remain out of scope.

**[NEW v1.1] EPHEMERIS firmware integrity and tamper-evident packaging are assumed and are out-of-scope of caduceus-bench.** EPHEMERIS supply-chain attestation is a parallel requirement to be addressed by VBX-01 v0.3+ firmware signing and packaging discipline, not by Caduceus alone.

**[NEW v1.1] Caduceus enforces key-authority asymmetry.** PHRONESIS units hold per-suit telemetry-signing keys only. Master command-authority keys are held by ground signers and are never deployed to suits. Suit capture is bounded to per-suit telemetry forgery; suit capture cannot escalate to command authority over other crew. Ground key compromise remains a total-breach scenario; mitigation is operational (multi-party master-key custody), not bench-asserted.

---

## 1. Architectural framing

Unchanged from CADUCEUS-001 §1. The six-attribute whitespace map (§1.3) holds.

---

## 2. ZCS-6 Phase 1 — Whitespace

Unchanged from CADUCEUS-001 §2.

---

## 3. ZCS-6 Phase 2 — caduceus-bench v1.1 (draft, pre-commit)

**Seventeen assertions across four layers.** v1.0 had twelve; v1.1 adds five and revises seven. Pre-commit cryptographic discipline unchanged: SHA-256 manifest hash, Ed25519 signature with Visionblox release key, Aletheia ledger entry, OpenTimestamps proof. **No silent edits — versioned re-commits only.**

### 3.1 Physical layer (P-class) — unchanged

| ID | Assertion | Threshold |
|---|---|---|
| P1 | RF link-budget margin on S-band uplink at Earth–Mars maximum-conjunction range | ≥ 3 dB above Shannon-rate floor |
| P2 | Optical pointing accuracy when optical mode is engaged | ≤ 1 µrad RMS over 60 s |
| P3 | Multi-mode reconfiguration time (S → UHF, optical → S) | ≤ 5 s from loss detection to reacquired bundle delivery |

### 3.2 Bundle layer (B-class) — B2 revised

| ID | Assertion | Threshold | Δ |
|---|---|---|---|
| B1 | BP7 (RFC 9171) wire-format conformance, 1 KB and 1 MB round-trip through ION-DTN or μPCN | 100% bit-exact, all required fields parsed | — |
| **B2** | **Store-and-forward integrity through ≥ 3 hops with simulated 60-min connectivity outage between hops 2 and 3, AND with bundle fragmentation invoked under BPSec (RFC 9172) using asymmetric keys rooted in the Aletheia key infrastructure (NOT a separate BPSec key infrastructure). Receiver produces zero side-effects (no display, no alert, no log-as-acted, no command-execution, no gate-decision-logged-as-pending) until full reassembly AND full signature verification.** | **100% custody-transferred bundles delivered; 100% partial-fragment side-effects blocked; 0% BPSec-key-isolated** | **revised** |
| B3 | Custody-transfer semantics under intermittent loss — TTL expiry → explicit failure notification | 100% expired bundles produce custody-failure status report | — |

### 3.3 Trust layer (T-class) — major v1.1 expansion

| ID | Assertion | Threshold | Δ |
|---|---|---|---|
| **T1** | **Every outbound bundle carries an Ed25519 signature (constant-time implementation, e.g., libsodium ref10-hardened) over (bundle_id ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce) AND a BP7 lifetime field. Receiver MUST verify (signature, lifetime-not-expired, nonce-not-seen-in-set, prev_hash-matches-last-accepted-within-authority-stream).** | **100% outbound signed; 100% receiver four-checks executed; 100% constant-time on both sign and verify** | **revised** |
| **T2** | **Every inbound bundle is verified against (a) the current local Civium authority graph AND (b) a time-bound validity assertion ("this authority is valid for this scope until TAI epoch X") signed as part of the authority assertion. T2 passes only if BOTH (a) and (b) succeed.** | **100% gate decisions logged with both checks; 0% pre-gate side effects** | **revised** |
| T3 | Signature failure OR gate denial deterministically falls back to HF-9 — no LLM-mediated retry | 100% fail-safe transitions logged on Aletheia chain; ≤ Mercury-WCET bound | — |
| **T4 [NEW]** | **Replay-prevention floor: receiver maintains a nonce-seen-set per authority stream with retention ≥ max-TTL + grace. Bundles failing any of the four T1 checks deterministically fall back to HF-9. Nonce-seen-set has bounded memory (per §6.4 cap) with explicit eviction policy: LRU among expired-TTL nonces only; live-TTL nonces never evicted.** | **100% replay attempts logged; 0% memory-exhaustion-induced eviction of live-TTL nonces** | **new** |
| **T5 [NEW]** | **Link-integrity floor: ACI's conformal bandwidth on (BER, SNR, RTT) is bounded above by a mission-policy maximum AND below by a physics-minimum (link natural-variance floor). Bandwidth exceeding the upper bound raises a CORE-class alert (link operating below ops-safe integrity). Bandwidth attempting to compress below the physics-minimum is rejected as adversarial. Alpha-update has a lower bound preventing indefinite band expansion.** | **100% bandwidth excursions alerted; 0% sub-physics-floor band compression accepted** | **new** |
| **T6 [NEW]** | **Ingress validity floor: receiver MUST verify F1 (URN + timestamp envelope) before any T-layer verification, B-layer custody decision, or render-layer pass. F1-rejected bundles are quarantined: logged with full content, alert raised, no side effects. Quarantine class distinct from signature-failure class.** | **100% F1 checks executed before T/B/render; 100% out-of-envelope bundles quarantined** | **new** |
| **T7 [NEW]** | **Revocation propagation: revocation events are themselves Caduceus bundles on a dedicated authority-revocation stream, signed by ground master keys with their own freshness tokens (revocation includes "auth X revoked at TAI epoch Y, valid from there forward"). Receiver MUST process revocation bundles before any command-class bundle from the revoked authority. If local authority graph has not received an authoritative refresh within mission-policy interval (24 h ground-link-available, 7 days connectivity-degraded), conservative defaults apply per enumerated table (§3.6).** | **100% revocations processed in priority lane; 100% stale-graph defaults applied per §3.6; 0% revoked-authority bundles passed during propagation window** | **new** |

### 3.4 Frame layer (F-class) — F1 and F2 revised; F4 added

| ID | Assertion | Threshold | Δ |
|---|---|---|---|
| **F1** | **Every bundle carries a (TAI_microseconds, vbx-body-URN) tuple in a BP7 extension block, where the URN conforms to Caduceus-BodyID v1.0 (§3.5) AND the TAI timestamp is within the validated envelope: 1972-01-01T00:00:00 TAI (inclusive) ≤ t < 2101-01-01T00:00:00 TAI (exclusive). Out-of-envelope timestamps cause F1 rejection at ingress, before any T-layer or B-layer processing.** | **100% bundles carry tuple; URN parses; timestamp in envelope; out-of-envelope quarantined** | **revised** |
| **F2** | **All bundles whose payload class is in {command, telemetry-acted, advisory-medical, advisory-safety, configuration, status-actionable} are rejected if the URN's (body, frame) is not in the local PHRONESIS authorized-context set. Display-only and crew-informational classes are not gated by F2 but are gated by F4.** | **100% unauthorized cross-context bundles rejected in gated classes** | **revised** |
| F3 | EPHEMERIS independently verifies the URN field on its display ring; mismatch with PHRONESIS-displayed URN triggers alert | ≤ 100 ms mismatch detection latency over UWB peer link | — |
| **F4 [NEW]** | **Frame-context interlock for non-command bundles: for every bundle delivered to a non-display side-effect path (gate, log-as-acted, command-execution, alert-state-change), receiver MUST first verify URN-context match against local operational context. Mismatch raises crew alert AND blocks the side-effect path until crew confirms explicitly via EPHEMERIS peer (read-and-attest acknowledgment). Confirmation requests are rate-limited and clustered: ≤ 1 active confirmation request per crew member at a time; similar mismatches batched into a single confirmation; novel-class mismatches escalate independently of recent dismissals.** | **100% side-effect paths interlocked; ≤ 1 active confirmation/crew/time; 0% confirmation-fatigue cascade documented in simulated 90-day adversarial flood** | **new** |

### 3.5 Caduceus-BodyID v1.1 — namespace (revised from v1.0)

URN scheme unchanged: `vbx-body:v1.0:<path>?frame=<frame_id>[&epoch=<rfc3339-utc>]`

[NEW v1.1] **Parsing requirements (normative):**

1. **Grammar enforcement.** URN parsing MUST enforce the EBNF grammar strictly. Grammar violations cause rejection at ingress. Examples of rejected URNs: empty path levels (`vbx-body:v1.0:earth/leo//iss?frame=`), level1 not in the registered top-level set, more than three path levels, missing frame query parameter, malformed epoch.
2. **Unicode NFC + ASCII-only canonical form.** Path and frame fields MUST be NFC-normalized and ASCII-only. Look-alike attacks (Cyrillic "e", Greek "o", etc.) rejected. The v1.0 registry is entirely ASCII; this requirement does not exclude any registered body or frame.
3. **Registry strict-match.** Body and frame lookups against the v1.x registry require exact-string match after NFC + ASCII canonicalization. Partial-match, substring-match, fuzzy-match all rejected.
4. **Atomic registry transition.** v1.x → v1.(x+1) namespace bumps are atomic on the Aletheia ledger. Operations during the commit window use the prior-version registry until commit finalizes. No partial-update reads.

Initial frame registry unchanged from v1.0 (§3.5 of CADUCEUS-001): 13 frames including UNDEFINED for Saturn.

Wire-format binding unchanged: BP7 extension block (block-type-code TBD via CCSDS reserved-range request), UTF-8 length-prefixed, 256-byte cap. Overflow → hard rejection.

### 3.6 [NEW v1.1] Conservative-defaults table for stale authority graph (T7 binding)

When the local Civium authority graph has not received authoritative refresh within the mission-policy interval, T7 mandates conservative defaults per the following enumeration:

| Bundle class | Default under stale graph (ground-link-available context, 24 h interval exceeded) | Default under stale graph (connectivity-degraded context, 7-day interval exceeded) |
|---|---|---|
| Command (safety-critical, e.g., EVA termination) | Require dual-authority confirmation: bundle + independent revocation-check via dedicated channel | Reject; queue for refresh; alert crew |
| Command (non-safety, e.g., schedule update) | Drop trust tier by one (require additional crew confirmation via EPHEMERIS) | Drop two tiers; require crew + ground refresh before action |
| Telemetry-acted (e.g., biomed alert that triggers automated dosing) | Drop trust tier by one | Reject; queue for refresh |
| Advisory-medical / advisory-safety | Display normally with stale-graph banner | Display normally with persistent stale-graph banner |
| Configuration | Reject; queue for refresh | Reject; queue for refresh |
| Status-actionable | Drop trust tier by one | Reject; queue for refresh |
| Emergency-class (life-safety override) | Bypass T7 gating (HF-9 floors apply) | Bypass T7 gating (HF-9 floors apply) |
| Revocation bundles themselves | **Process at highest priority regardless of staleness** (otherwise revocation can never recover from stale state) | **Process at highest priority regardless of staleness** |
| Display-only / informational | Display normally with stale-graph banner | Display normally with persistent stale-graph banner |

The table is normative and forms part of caduceus-bench v1.1's locked content. Additions or modifications require a v1.x bump.

---

## 4. ZCS-6 Phase 3 — Adversarial sweep (queued for re-execution against v1.1)

Twelve attacks (8 original + 4 newly discovered in CADUCEUS-PHASE3-001) MUST be re-run against v1.1. **Each new assertion introduces new attack surface.** Re-sweep is the next gate before LEDGER #0004 eligibility.

[See CADUCEUS-PHASE3-002 for the v1.1 re-sweep results.]

---

## 5. ZCS-6 Phase 4 — Deterministic defense (commit plan)

Unchanged from CADUCEUS-001 §5. Commit happens only after a clean Phase 3 sweep — v1.0 did not qualify; v1.1 must be re-swept.

---

## 6. ZCS-6 Phase 5 — Substrate sketch (v1.1 expansions)

### 6.1 PHRONESIS-side modules — expanded floor map

| HF | Function | Backing module | Δ from v1.0 |
|---|---|---|---|
| HF-16 | Outbound bundle attestation | Aletheia + Civium | **+ constant-time Ed25519 mandatory** |
| HF-17 | Inbound verification + Civium authority gate | Civium + MVCI | **+ authority-graph freshness tracker; + conservative-default escalation logic per §3.6; + revocation-stream dedicated handling** |
| HF-18 | ACI drift monitor on link characteristics | Aletheia | **+ alpha lower bound; + max-bandwidth alert; + sub-physics-floor rejection** |
| HF-19 | Frame disambiguation on bundle headers | EPHEMERIS interop | **+ ingress-time envelope check (T6); + grammar/Unicode validation per §3.5** |
| HF-20 | Mercury-bounded WCET on safety-critical receive path | Mercury kernel | **+ comm-peak / compute-peak non-concurrence interlock** |
| **HF-21 [NEW]** | **Nonce-seen-set management** | Aletheia | **Bounded-memory replay-prevention store per T4** |
| **HF-22 [NEW]** | **Frame-context interlock for non-command paths** | Civium + EPHEMERIS | **Per F4; confirmation rate-limit and clustering** |

### 6.2 EPHEMERIS-side peer — unchanged from CADUCEUS-001 §6.2

[v0.2+ stretch] dual-peer attestation across two crew EPHEMERIS units for high-stakes ops.

### 6.3 Reference open-source dependencies — additions

[NEW v1.1] **libsodium** specifically required for constant-time Ed25519 (T1). Other Ed25519 implementations rejected unless audited for constant-time guarantees. **BPSec (RFC 9172)** reference implementation required as part of B2 substrate.

### 6.4 Mass and power budget cap — unchanged

4.0 kg total comm subsystem, 50 W steady, 100 W peak. Locked at v1.0 commit, carried forward to v1.1. Nonce-seen-set memory (HF-21) and revocation-stream tracker (HF-17 addition) sit in the existing compute budget on Orin NX; no mass impact.

---

## 7. ZCS-6 Phase 6 — Anticipated vertical-integration attacks (queued)

Unchanged from CADUCEUS-001 §7. Additional v1.1-specific Phase 6 candidates:

- Sustained confirmation-fatigue attack against F4 over a simulated 90-day mission window.
- Nonce-seen-set memory-exhaustion under live-TTL flooding.
- Stale-graph default-table boundary exploits (e.g., a bundle classified ambiguously between two table rows).
- Revocation-stream DoS via rate-limit exhaustion.

---

## 8. Cross-references — additions

| Component | Reference | Relationship |
|---|---|---|
| libsodium | Open source | Required cryptographic substrate per T1 |
| BPSec (RFC 9172) | IETF | Required for B2 fragmentation security |
| Aletheia DAC v0.3+ | Pending | Must extend chain semantics for cross-key-epoch forward-verification per T1 |

---

## 9. Open items carried forward to v0.2 and beyond — additions

(Items 1–11 unchanged from CADUCEUS-001 §9.)

12. **Aletheia DAC v0.3** — extend chain semantics to support cross-key-epoch forward-verification required by T1 revision. Coordinate with VBX-ISPS Aletheia track.
13. **BPSec key-rooting in Aletheia infrastructure** — formalize the Aletheia → BPSec key derivation specifically required by B2; defensive arXiv post-commit.
14. **Conservative-defaults table validation** — §3.6 table must be reviewed by mission ops counterpart (NASA OSMA, ESA Inspector General, or equivalent) before LEDGER commit if possible. If unavailable, internal-probe per Move 3 minimum.

---

## 10. Honest scope statement (recap, v1.1 update)

- v1.1 is Phase 2 first revision. Phase 3 re-sweep against v1.1 is the gate before LEDGER eligibility.
- v1.1 has 17 assertions vs v1.0's 12 (+5 new, 7 revised, 5 unchanged).
- Conservative-defaults table (§3.6) is normative content of the commit, not advisory.
- §0 additions (EPHEMERIS supply-chain assumption, key-authority asymmetry) are scope-narrowing and must accompany any external description of Caduceus to prevent overclaim.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-002 Rev A · caduceus-bench v1.1 (pre-commit, awaiting re-sweep)*
