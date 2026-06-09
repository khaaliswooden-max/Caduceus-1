# Caduceus-bench v1.1 — ZCS-6 Phase 3 Re-Sweep Report

| | |
|---|---|
| **Document ID** | CADUCEUS-PHASE3-002 |
| **Revision** | A |
| **Target** | caduceus-bench v1.1 (CADUCEUS-002) |
| **Methodology** | ZCS-6 Phase 3 (adversarial sweep, second iteration) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **VERDICT: REVISION REQUIRED → caduceus-bench v1.2.** v1.1 closes all 12 original v1.0 attacks but opens 8 new attack vectors against the new and revised assertions. |

---

## Executive verdict

caduceus-bench v1.1 closes all 12 attacks from CADUCEUS-PHASE3-001. **But the new and revised assertions introduce 8 new attack vectors, all material.**

This is the expected ZCS-6 pattern — every defense surface is also an attack surface. The 8 new vectors are *tightening* attacks rather than *structural* attacks — they exploit imprecision in the v1.1 assertions, not architectural gaps. The required deltas for v1.2 are predominantly assertion clarifications and detail additions, not new floors.

After v1.2 tightening, convergence is the expected outcome. v1.2 should be the LEDGER #0004 candidate.

---

## Original 12 attacks vs v1.1

| # | Attack | Status against v1.1 |
|---|---|---|
| 1 | Replay across long RTT | **CLOSED** by T1 + T4 |
| 2 | Frame confusion (broadened) | **CLOSED** by F2 (revised) + F4 |
| 3 | Bundle fragmentation | **CLOSED** by B2 (BPSec + no-partial-side-effects) |
| 4 | Trust-anchor compromise at relay | **CLOSED** (was conditional; condition met by T1+T4) |
| 5 | ACI thrashing + slow-drift | **CLOSED** by T5 |
| 6 | EPHEMERIS-PHRONESIS MitM | **CLOSED** (assumption flagged in §0) |
| 7 | A1d seam | **CLOSED** by F1 (revised) + T6 |
| 8 | Namespace exhaustion | **CLOSED** by §3.5 v1.1 (grammar + Unicode + atomic) |
| 9 | Slow-drift ACI (folded into 5) | **CLOSED** |
| 10 | Authority-graph staleness | **CLOSED** by T2 (revised) + T7 + §3.6 |
| 11 | Side-channel on Ed25519 | **CLOSED** by T1 (constant-time mandate) |
| 12 | Cold-boot / suit-capture | **NOTED** in §0 (key-authority asymmetry) |

**All 12 original attacks closed in v1.1.** The architectural posture is sound.

---

## New attacks discovered against v1.1

### Attack 21 — TOCTOU between T1's four receiver checks

**Mechanism.** T1 requires receiver to verify (signature, lifetime, nonce-not-seen, prev_hash). If these are executed sequentially on a shared mutable state (the nonce-seen-set and the prev_hash pointer), an adversary may exploit the window between check 2 (lifetime) and check 3 (nonce-set query) to insert a duplicate nonce — or the window between check 3 and check 4 to manipulate the prev_hash pointer.

**Probe.** v1.1 T1 does not assert atomicity of the four checks. A naive implementation processing two bundles concurrently could allow both to pass nonce-not-seen against the same fresh nonce.

**Verdict.** **GAP.**

**Delta for v1.2.** T1 detail clause: "The four receiver checks (signature, lifetime, nonce-not-seen, prev_hash) MUST be executed atomically per bundle. Implementations MUST hold an exclusive lock on the per-authority-stream nonce-seen-set + prev_hash pointer from check 2 through check 4 inclusive."

---

### Attack 22 — Time-bound authority succession ambiguity

**Mechanism.** T2 says authority is "valid for this scope until TAI epoch X." Authority A is issued with X = T+10y. At T+2y, authority A is renewed as authority A' (different key, same role). Without explicit succession semantics, A is still valid until T+10y per its original X — A and A' coexist. If A's key is compromised at T+5y, A's bundles still pass T2 until T+10y, despite A' having been issued.

**Probe.** v1.1 T7 addresses revocation but not succession-without-revocation. A clean renewal that does not explicitly revoke the predecessor leaves the predecessor active.

**Verdict.** **GAP.**

**Delta for v1.2.** T7 expansion: "Authority renewal (issuance of a successor authority for the same scope) MUST be accompanied by an explicit revocation bundle for the predecessor, signed by the same ground master key that issued the successor. Receiver MUST treat any active successor as an implicit revocation of predecessors in the same scope, even if the explicit revocation bundle has not yet propagated — succession-implies-revocation default rule."

---

### Attack 23 — Live-TTL nonce-set flooding

**Mechanism.** T4 says live-TTL nonces are never evicted from the nonce-seen-set. Adversary (or compromised authority) floods with max-TTL bundles, each consuming a slot. The set grows toward the §6.4-bounded memory cap. Eventually new legitimate bundles are rejected because the set is full of live-TTL adversarial nonces.

**Probe.** v1.1 T4 specifies bounded memory and live-TTL non-eviction. It does not specify per-authority-stream quotas. A single compromised authority can monopolize the global set.

**Verdict.** **GAP.**

**Delta for v1.2.** T4 revised: "Nonce-seen-set is partitioned per authority stream. Each stream has a bounded quota: max(N_streams × per-stream-quota) ≤ total memory cap. Per-stream quota exceeded by a single stream causes that stream's oldest live-TTL nonces to be evicted with explicit alert (degraded-replay-protection state for that stream); other streams remain unaffected. Global memory exhaustion across all streams triggers HF-9 fail-safe."

---

### Attack 24 — Physics-floor calibration attack against T5

**Mechanism.** T5 specifies ACI's conformal bandwidth must be bounded below by a "physics-minimum (link natural-variance floor)." If the floor is estimated empirically from link observation, an adversary can clean the channel (suppress all noise) during a calibration period, training PHRONESIS's physics-floor estimate to a too-tight value. After calibration, legitimate operational variation falls outside the calibrated floor and gets rejected.

**Probe.** v1.1 T5 says "link natural-variance floor" without specifying empirical vs theoretical. A naive implementation will use empirical-only.

**Verdict.** **GAP.**

**Delta for v1.2.** T5 revised: "Physics-minimum floor is max(empirical-observed-floor, theoretical-floor) where theoretical-floor is derived from first-principles link characteristics (thermal noise, antenna pattern, propagation distance). Empirical calibration cannot drive the floor below the theoretical bound. Theoretical-floor is registered as a per-link-configuration constant in the substrate; updates require a v1.x bump."

---

### Attack 25 — Class-claim spoofing

**Mechanism.** F2 enumerates gated payload classes: {command, telemetry-acted, advisory-medical, advisory-safety, configuration, status-actionable}. The class field is presumably in the bundle. Adversary marks an action-inducing bundle with a non-enumerated or display-only class to bypass F2 gating. Worse: adversary marks a command bundle as "display-only" to bypass F2 entirely while still triggering the receiver's downstream processing (which may treat any signed bundle as actionable).

**Probe.** v1.1 T1 lists signed fields: (bundle_id ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce). **Class is not in the signed set.** An attacker who can modify the class field after signature (or who controls bundle origination) can spoof class.

**Verdict.** **GAP.**

**Delta for v1.2.** Two coupled revisions:

- T1 revised: signed-fields list extended to include payload_class. New signed tuple: (bundle_id ∥ payload_class ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce).
- F2 revised: payload_class field MUST be in the enumerated set; non-enumerated classes are F2-rejected (not F2-passed); class signed-integrity verified before F2 check.

---

### Attack 26 — Confirmation-clustering bypass on F4

**Mechanism.** F4 clusters "similar" mismatches into a single confirmation request. Adversary deliberately constructs each bundle to differ in surface features (different URN microvariation, different timestamp, different payload type) such that the clustering heuristic treats each as novel. Each requires its own confirmation. Crew is back to confirmation-fatigue.

**Probe.** v1.1 F4 says "similar mismatches batched" without specifying the similarity criterion. A surface-feature-based clustering is the default and is exploitable.

**Verdict.** **GAP.**

**Delta for v1.2.** F4 revised: "Clustering criterion MUST be based on root cause (specific URN field mismatch: body, frame, or both), not surface features. Two mismatches are clustered if and only if they share the same (mismatch-class, expected-body, actual-body, expected-frame, actual-frame) tuple. PLUS: hard rate limit independent of clustering: ≤ K confirmations per crew member per hour (K mission-policy-set, default 6). Bundles in mismatched-frame state that exceed K are rejected outright until next graph refresh — no further confirmation requests are issued in that window."

---

### Attack 27 — Registry transition at light-time distance

**Mechanism.** §3.5 v1.1 says namespace transitions are atomic on the Aletheia ledger. Aletheia ledger updates propagate at light-speed. PHRONESIS on Mars receives the v1.0 → v1.1 transition up to 22 minutes after Earth-side completion. During the window:

- Earth uses v1.1 namespace. Bundles signed and sent reference v1.1 URNs and registry-specific behaviors.
- Mars PHRONESIS still uses v1.0 registry. Bundles arriving with v1.1-specific features are rejected as malformed.

Worse: a bundle sent from Mars during the window will use v1.0 references; ground systems on Earth (now on v1.1) may reject it if backward compatibility is not retained.

**Probe.** v1.1 §3.5 atomic-registry-transition rule assumes a single trust-domain commit. The interplanetary commit is inherently non-atomic in time.

**Verdict.** **GAP.**

**Delta for v1.2.** §3.5 revised: "Registry version is carried in the URN scheme itself (`vbx-body:v1.0:` vs `vbx-body:v1.1:`). During registry transition windows, receivers MUST accept and parse URNs using both the prior and current registry versions for a mission-policy interval (default: 2 × max-RTT for the slowest active link, minimum 60 minutes). After the interval, the prior version is rejected. Transition windows are themselves announced as a special bundle class (registry-version-transition) on the priority lane, signed by the registry master key, with explicit start and end TAI epochs."

---

### Attack 28 — Class ambiguity in §3.6 conservative-defaults table

**Mechanism.** §3.6 enumerates payload classes per row. A real bundle may fit multiple rows. Example: a medical advisory that triggers automated dosing is both "advisory-medical" and "telemetry-acted." The two rows specify different defaults under stale graph.

**Probe.** v1.1 §3.6 has no precedence rule. Implementation will pick one row arbitrarily — likely the first match — which may be the more permissive row.

**Verdict.** **GAP.**

**Delta for v1.2.** §3.6 addition: "When a bundle's class matches multiple table rows, the most restrictive default applies (highest tier-drop, strictest action). Precedence ordering from most-restrictive to least-restrictive: Configuration > Command (safety-critical) > Telemetry-acted > Command (non-safety) > Status-actionable > Advisory-medical = Advisory-safety > Display-only/informational. Emergency-class and Revocation bundles bypass this precedence and apply their own special-case rows. Precedence ordering is part of the v1.x-locked bench content."

---

## New attacks summary

| # | Attack | Verdict | v1.2 delta class |
|---|---|---|---|
| 21 | TOCTOU on T1 checks | GAP | T1 atomicity clause |
| 22 | Authority succession ambiguity | GAP | T7 succession-implies-revocation rule |
| 23 | Live-TTL nonce-set flooding | GAP | T4 per-stream quotas |
| 24 | Physics-floor calibration | GAP | T5 theoretical + empirical |
| 25 | Class-claim spoofing | GAP | T1 + F2 (class signed and enumerated) |
| 26 | Confirmation-clustering bypass | GAP | F4 root-cause clustering + hard rate limit |
| 27 | Registry transition at light-time | GAP | §3.5 dual-version retention window |
| 28 | Class ambiguity in §3.6 | GAP | §3.6 precedence ordering |

**All 8 are tightening attacks** (assertion precision, not architectural gaps). The fix scope is bounded — no new floors required, just clarification of existing ones.

---

## Required changes for caduceus-bench v1.2

**T-layer:**
- T1 expanded signed tuple (+ payload_class) + atomicity clause
- T4 per-stream quotas
- T5 theoretical-floor binding
- T7 succession-implies-revocation rule

**F-layer:**
- F2 class field signed-integrity + enumerated-only acceptance
- F4 root-cause clustering + hard rate limit

**§3.5 namespace:**
- Dual-version retention window for interplanetary atomic transitions
- Registry-version-transition bundle class on priority lane

**§3.6 table:**
- Precedence ordering for ambiguous-class bundles

**§0 scope (no change needed).**

**Substrate-side (v0.2 build) downstream impacts:**
- HF-16/17 atomicity locking around nonce-set and prev_hash operations
- HF-17 per-stream nonce-set partitioning; registry-version dual-retention buffer
- HF-18 theoretical-floor constants per link configuration
- HF-22 root-cause clustering logic + hard rate limit enforcement

---

## Convergence projection for v1.2

All 8 deltas are clarifications, not new assertions. After v1.2:

- Total assertion count: 17 (unchanged from v1.1; v1.2 tightens existing ones)
- §3.5 and §3.6 gain normative additions but no new sections
- The next Phase 3 sweep against v1.2 will probe these tightenings — most likely outcome is convergence (no material gaps) given the spec is now closing toward a fixed point at the assertion-precision layer.

**Projected outcome: caduceus-bench v1.2 passes Phase 3 cleanly and is eligible for LEDGER #0004 commit.**

If v1.2 sweep finds material gaps (unexpected per stopping rule), reassessment required — that would be a substrate-architecture signal, not a bench-iteration signal.

---

## Honest scope statement

- Confidence on attacks 21–28: HIGH (all are well-understood patterns in distributed-systems and security-protocol literature, adapted to the interplanetary RTT context).
- Attack 27 (registry transition at light-time) is the second interplanetary-specific finding in this sweep series; like Attack 10 (authority staleness), it would not appear in Earth-only deployments. Worth flagging as part of the prior-art anchor for any patent filing on interplanetary compliance fabrics.
- v1.2 should still receive diversified-internal probe (Move 3) before LEDGER commit — internal sweep is necessary but not sufficient.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-PHASE3-002 Rev A · Phase 3 re-sweep on v1.1*
*Verdict: REVISION REQUIRED → caduceus-bench v1.2*
