# Caduceus v0.1 — Interplanetary Connectivity Substrate
## caduceus-bench v1.2 specification (LEDGER #0004 candidate)

| | |
|---|---|
| **Document ID** | CADUCEUS-003 |
| **Revision** | A |
| **Methodology** | ZCS-6 (Phase 1 + Phase 2 — second revision of bench) |
| **Predecessor** | CADUCEUS-002 (v1.1, found 8 tightening gaps at Phase 3 re-sweep) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **LEDGER #0004 candidate.** Final Phase 3 sweep (§4) returns clean; eligible for cryptographic commit subject to Move 3 diversified-internal probe. |
| **Delta source** | CADUCEUS-PHASE3-002 (re-sweep report on v1.1) |

---

## 0. Scope statement

Unchanged from CADUCEUS-002 §0. EPHEMERIS supply-chain assumption and key-authority asymmetry remain normative.

---

## 1–2. Architectural framing and Phase 1 whitespace

Unchanged from CADUCEUS-001 §§1–2. The six-attribute whitespace map holds. Phase 1 substrate baseline (DTN/BP7, CCSDS, DSN, DSOC, LunaNet, Mars Relay Network, Yin 2017 Micius QKD) unchanged.

---

## 3. ZCS-6 Phase 2 — caduceus-bench v1.2 (final pre-commit form)

**Seventeen assertions, four layers.** Same count as v1.1; v1.2 tightens 6 of them and adds normative content to §3.5 and §3.6.

### 3.1 Physical layer (P-class) — unchanged

P1, P2, P3 as in v1.1 §3.1.

### 3.2 Bundle layer (B-class) — unchanged from v1.1

B1, B2 (BPSec + Aletheia-rooted keys), B3 as in v1.1 §3.2.

### 3.3 Trust layer (T-class) — T1, T4, T5, T7 revised

| ID | Assertion | Threshold | Δ from v1.1 |
|---|---|---|---|
| **T1** | **Every outbound bundle carries an Ed25519 signature (constant-time, e.g., libsodium ref10-hardened) over the canonical signed tuple: (bundle_id ∥ payload_class ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce) AND a BP7 lifetime field. Receiver MUST verify (signature, lifetime-not-expired, nonce-not-seen-in-set, prev_hash-matches-last-accepted-within-authority-stream) AS AN ATOMIC SEQUENCE — implementations MUST hold an exclusive lock on the per-authority-stream nonce-seen-set + prev_hash pointer from check 2 through check 4 inclusive.** | **100% outbound signed; 100% atomic verification per bundle; 100% constant-time crypto** | **revised: +payload_class signed; +atomicity clause** |
| T2 | Every inbound bundle is verified against (a) the current local Civium authority graph AND (b) a time-bound validity assertion signed as part of the authority assertion. T2 passes only if BOTH succeed. | 100% gate decisions logged with both checks; 0% pre-gate side effects | — |
| T3 | Signature failure OR gate denial deterministically falls back to HF-9. | 100% fail-safe transitions logged; ≤ Mercury-WCET bound | — |
| **T4** | **Replay-prevention floor: nonce-seen-set is PARTITIONED PER AUTHORITY STREAM with bounded per-stream quotas. Total: max(N_streams × per-stream-quota) ≤ total memory cap (per §6.4). Per-stream quota exceeded by a single stream causes that stream's oldest live-TTL nonces to be evicted with explicit alert ("degraded-replay-protection" state for that stream); other streams unaffected. Global memory exhaustion across all streams triggers HF-9 fail-safe.** | **100% replay attempts logged; 0% cross-stream contamination of nonce-set; degraded-replay-protection alerts surface to crew within 100 ms** | **revised: per-stream quotas** |
| **T5** | **Link-integrity floor: ACI's conformal bandwidth on (BER, SNR, RTT) is bounded above by mission-policy maximum and below by max(empirical-observed-floor, theoretical-floor). Theoretical-floor is derived from first-principles link characteristics (thermal noise, antenna pattern, propagation distance) and registered as a per-link-configuration constant in the substrate. Empirical calibration cannot drive the effective floor below the theoretical bound. Floor updates require v1.x bump.** | **100% bandwidth excursions alerted; 0% sub-theoretical-floor band acceptance** | **revised: theoretical-floor binding** |
| T6 | Ingress validity floor: receiver MUST verify F1 before any T-layer, B-layer, or render-layer processing. F1-rejected bundles quarantined. | 100% F1 checks before T/B/render; 100% out-of-envelope quarantined | — |
| **T7** | **Revocation propagation: revocation events are dedicated authority-revocation-stream bundles, signed by ground master keys, carrying freshness tokens. Receiver MUST process revocations in the priority lane before any command-class bundle from the revoked authority. Stale-graph defaults per §3.6. AUTHORITY RENEWAL (issuance of a successor authority for the same scope) MUST be accompanied by an explicit revocation of the predecessor signed by the same ground master key. Receiver MUST treat any active successor as an implicit revocation of predecessors in the same scope — succession-implies-revocation default applies even before the explicit revocation bundle has propagated.** | **100% revocations processed in priority lane; 100% successions implicitly revoke predecessors; 0% predecessor-key acceptance after successor is local** | **revised: +succession rule** |

### 3.4 Frame layer (F-class) — F2 and F4 revised

| ID | Assertion | Threshold | Δ from v1.1 |
|---|---|---|---|
| F1 | Every bundle carries a (TAI_microseconds, vbx-body-URN) tuple in BP7 extension block, URN conforms to Caduceus-BodyID (§3.5), timestamp in envelope [1972-01-01, 2101-01-01) TAI. Out-of-envelope rejected at ingress. | 100% bundles carry valid tuple; out-of-envelope quarantined | — |
| **F2** | **All bundles whose payload_class is in {command, telemetry-acted, advisory-medical, advisory-safety, configuration, status-actionable} are rejected if URN's (body, frame) is not in the local authorized-context set. Bundles whose payload_class is NOT in the enumerated set (and not in {display-only, informational, emergency-class, revocation}) are rejected as malformed — F2 implicitly closes the class namespace. payload_class field is signed per T1 — receiver MUST verify class-field signed-integrity before F2 check.** | **100% unauthorized cross-context bundles rejected; 100% unenumerated-class bundles rejected; 0% post-signature class manipulation accepted** | **revised: +class signed-integrity verification; +closed class namespace** |
| F3 | EPHEMERIS independently verifies URN; mismatch with PHRONESIS-displayed URN triggers alert. | ≤ 100 ms detection latency over UWB | — |
| **F4** | **Frame-context interlock for non-command bundles: side-effect paths require URN-context verification; mismatch raises crew alert AND blocks side-effect until crew confirms via EPHEMERIS. Clustering criterion is ROOT-CAUSE-BASED: two mismatches are clustered iff they share the (mismatch-class, expected-body, actual-body, expected-frame, actual-frame) tuple — surface features (URN microvariations, timestamps) do NOT defeat clustering. Hard rate limit independent of clustering: ≤ K confirmations per crew member per hour (default K=6). Bundles in mismatched-frame exceeding K are rejected outright until next authority-graph refresh — no further confirmations issued in that window.** | **100% side-effect paths interlocked; root-cause clustering verified; hard-rate-limit enforced; confirmation-fatigue 0-cascade under 90-day adversarial flood simulation** | **revised: root-cause clustering + hard rate limit** |

### 3.5 Caduceus-BodyID v1.0 — namespace (interplanetary-transition rules added)

URN scheme unchanged: `vbx-body:v1.0:<path>?frame=<frame_id>[&epoch=<rfc3339-utc>]`

**Parsing requirements (unchanged from v1.1):** grammar enforcement, NFC + ASCII-only canonical, strict-match against registry, atomic transition on Aletheia ledger.

**[NEW v1.2] Interplanetary-transition retention window.** Registry version is carried in the URN scheme itself (`vbx-body:v1.0:` vs `vbx-body:v1.1:`). During registry transition windows, receivers MUST accept and parse URNs using both the prior and current registry versions for a mission-policy interval (default: 2 × max-RTT for the slowest active link; minimum 60 minutes). After the interval, prior-version URNs are rejected. Transition windows themselves are announced as a special bundle class (`registry-version-transition`) on the priority lane, signed by the registry master key, with explicit start and end TAI epochs.

Initial frame registry and wire-format binding unchanged from v1.0/v1.1.

### 3.6 Conservative-defaults table (v1.2 — precedence rule added)

Table content unchanged from v1.1 §3.6. **[NEW v1.2] Precedence rule:**

"When a bundle's payload_class matches multiple table rows, the **most restrictive default** applies. Precedence ordering from most-restrictive to least-restrictive:

`Configuration > Command (safety-critical) > Telemetry-acted > Command (non-safety) > Status-actionable > Advisory-medical = Advisory-safety > Display-only/informational`

Emergency-class and Revocation bundles bypass this precedence and apply their own special-case rows. Precedence ordering is part of the v1.x-locked bench content."

---

## 4. ZCS-6 Phase 3 — Final sweep against v1.2 (embedded convergence statement)

### 4.1 Sweep methodology

Re-running all 20 attacks (12 original from CADUCEUS-PHASE3-001 + 8 new from CADUCEUS-PHASE3-002) against v1.2. Plus adversarial probing for any *new* attack surface introduced by v1.2's tightening.

### 4.2 Sweep results

| Attack class | # | v1.2 status | Closing mechanism |
|---|---|---|---|
| Original 12 attacks (#1–12) | 12 | **ALL CLOSED** (carried from v1.1) | T1, T4, T5, T6, T7, F1, F2, F4, B2, §3.5, §3.6, §0 scope notes |
| TOCTOU on T1 checks (#21) | 1 | **CLOSED** | T1 v1.2 atomicity clause |
| Authority succession (#22) | 1 | **CLOSED** | T7 v1.2 succession-implies-revocation rule |
| Live-TTL nonce flooding (#23) | 1 | **CLOSED** | T4 v1.2 per-stream quotas + degraded-replay alert |
| Physics-floor calibration (#24) | 1 | **CLOSED** | T5 v1.2 theoretical-floor binding |
| Class-claim spoofing (#25) | 1 | **CLOSED** | T1 v1.2 (+payload_class signed) + F2 v1.2 (signed-integrity + closed namespace) |
| Confirmation-clustering bypass (#26) | 1 | **CLOSED** | F4 v1.2 root-cause clustering + hard rate limit |
| Registry transition at light-time (#27) | 1 | **CLOSED** | §3.5 v1.2 dual-version retention + transition bundle class |
| Class ambiguity (#28) | 1 | **CLOSED** | §3.6 v1.2 precedence ordering |

### 4.3 New attack surface introduced by v1.2

Probing the v1.2 deltas for new attack vectors:

| Probe | Attack surface | Verdict |
|---|---|---|
| T1 atomicity → lock contention | Adversary floods to force lock starvation across streams | **NOT MATERIAL** — per-stream locks (per T4 partitioning); cross-stream contention bounded; HF-20 Mercury-WCET enforces bounded acquisition time |
| T4 per-stream quotas → quota exhaustion as DoS signal | Adversary exhausts a benign stream's quota to force "degraded-replay-protection" alert noise | **NOT MATERIAL** — alert is per-stream and labeled; does not cascade or affect other streams; alert fatigue mitigated by F4's hard rate limit applying to ALL crew alerts |
| T5 theoretical-floor → floor too lax | Adversary exploits gap between theoretical floor and ops-realistic floor | **NOT MATERIAL** — theoretical floor is LOWER bound; empirical floor when valid is still binding; attack requires both theoretical and empirical to be wrong simultaneously |
| T7 succession-implies-revocation → premature succession | Adversary issues bogus successor to revoke a legitimate authority | **NOT MATERIAL** — successor must be signed by same ground master key as predecessor; if master key compromised, total breach (out-of-bench condition per §0) |
| F2 closed class namespace → legitimate new class blocked | Future genuinely new class additions force benchmark revision | **ACCEPTED COST** — namespace closure is the point; new classes are v1.x-bump events, not runtime additions |
| F4 hard rate limit → legitimate confirmation overflow | High-tempo ops with genuine mismatches blocked at K | **ACCEPTED COST** — K is mission-policy-set; legitimate ops at K=6/hr/crew is sufficient for crewed mission tempo; emergency-class bypasses |
| §3.5 dual-version window → ambiguous URN interpretation | Adversary exploits window boundary to send bundle interpretable as either v1.0 or v1.1 | **NOT MATERIAL** — URN scheme prefix is unambiguous (`v1.0:` vs `v1.1:`); a bundle is one version or the other, not both; receiver's accept-both-during-window does not blur the per-bundle interpretation |
| §3.6 precedence → over-restriction | Bundles with multi-row class receive the most-restrictive default even when context permits less | **ACCEPTED COST** — over-restriction is conservative-defaults intent under stale-graph; crew confirmation path remains open via F4 |

**No material new attack surface introduced by v1.2 tightenings.**

### 4.4 Convergence verdict

**caduceus-bench v1.2 passes ZCS-6 Phase 3.**

All 20 attacks across two sweep iterations are closed. The 8 new attack surfaces introduced by v1.2's tightening are either not-material (5 cases) or accepted-cost design choices with explicit justification (3 cases).

**v1.2 is eligible for LEDGER #0004 commit subject to Move 3 diversified-internal probe.**

---

## 5. ZCS-6 Phase 4 — Deterministic defense (commit plan)

Unchanged from CADUCEUS-001 §5. Commit happens after Move 3 diversified-internal probe (CADUCEUS-PROBE-001) returns without material findings. Steps:

1. Freeze CADUCEUS-003 markdown (this document) as the committed manifest.
2. SHA-256 hash.
3. Ed25519 sign with Visionblox release key.
4. Append to Aletheia chain as **LEDGER #0004**.
5. OpenTimestamps proof on the ledger entry.
6. Publish manifest hash + signature publicly via defensive arXiv preprint or SSRN companion before any v0.1 substrate code is written.

---

## 6. ZCS-6 Phase 5 — Substrate sketch (v1.2 updates)

PHRONESIS-side floor map unchanged in count from v1.1 (HF-16 through HF-22). v1.2 adds implementation detail to substrate-side requirements:

| HF | v1.2 additions |
|---|---|
| HF-16 | payload_class included in canonical signed tuple; atomicity lock around signing operation |
| HF-17 | Per-stream nonce-set partitioning; succession-implies-revocation rule logic; registry-version dual-retention buffer; atomicity lock around four-check sequence |
| HF-18 | Theoretical-floor constants per link configuration; floor max(empirical, theoretical) logic |
| HF-19 | Closed class namespace enforcement; class signed-integrity verification before F2 |
| HF-20 | Lock-acquisition bounded WCET (Mercury kernel enforcement) |
| HF-21 | Per-stream quota accounting; degraded-replay-protection alert path |
| HF-22 | Root-cause clustering logic; hard-rate-limit enforcement; mismatched-frame-state-tracking |

§6.4 mass and power budget cap (4.0 kg / 50 W / 100 W) unchanged.

---

## 7–8. Phase 6 anticipated attacks and cross-references

Unchanged from CADUCEUS-002 §§7–8.

---

## 9. Open items carried to v0.2 and beyond

Items 1–14 from CADUCEUS-002 §9, plus:

15. **Move 3 diversified-internal probe** — required gate before Phase 4 commit; see CADUCEUS-PROBE-001.
16. **Attack 10 standalone IP filing** — interplanetary-RTT-aware authority propagation; see CADUCEUS-IP-001. Independent of LEDGER #0004 timing.
17. **External red-team engagement** — internal probe is necessary but not sufficient; pursue NASA OSMA / mission-assurance reviewer or equivalent external party post-commit for v1.3 hardening if needed.
18. **Aletheia ↔ BPSec key-derivation formalization** — required per B2 v1.1+; defensive arXiv post-commit.

---

## 10. Honest scope statement (final)

- v1.2 closes 20 attacks across two Phase 3 iterations. The architecture (whitespace map §1.3) is intact and the assertion set is dense.
- v1.2 is the LEDGER #0004 commit candidate. Move 3 diversified-internal probe is the next gate.
- Two interplanetary-specific findings (Attack 10: authority staleness; Attack 27: registry transition at light-time) emerged through the sweep process. Both are patent-defensible novelties that would not appear in Earth-only deployments. Worth claiming.
- Three categories of attack remain explicit out-of-bench risks: ground master-key compromise (operational mitigation, not bench-asserted), EPHEMERIS supply-chain integrity (parallel requirement on VBX-01), PHRONESIS Secure Element tamper (hardware-side, addressed by HF-8 + key-authority asymmetry).
- External red-team engagement remains a recommended-but-not-blocking step for LEDGER commit. Internal probe (Move 3) is the required gate.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-003 Rev A · caduceus-bench v1.2 (LEDGER #0004 candidate, awaiting Move 3 gate)*
