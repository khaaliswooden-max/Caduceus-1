# Caduceus-bench v1.0 — ZCS-6 Phase 3 Adversarial Sweep Report

| | |
|---|---|
| **Document ID** | CADUCEUS-PHASE3-001 |
| **Revision** | A (draft) |
| **Methodology** | ZCS-6 Phase 3 (adversarial sweep against benchmark) |
| **Target** | caduceus-bench v1.0 (draft, pre-commit) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **VERDICT: REVISION REQUIRED → caduceus-bench v1.1.** LEDGER #0004 should NOT be issued against v1.0. |

---

## Executive verdict

**caduceus-bench v1.0 does not pass ZCS-6 Phase 3.**

- 8 attacks were queued in CADUCEUS-001 §4.
- 6 attacks revealed gaps in the assertion set.
- 4 additional attacks were discovered during the sweep.
- 2 attacks survived (one conditional on another's fix).

The honest finding is that v1.0 has structural assertion holes — most notably an interplanetary-specific failure mode (Attack 10, authority-graph staleness) that was not in the original queue and that breaks the compliance-graded thesis if left unaddressed.

The right ZCS-6 move is to **revise to caduceus-bench v1.1** with explicit delta justification records, re-run Phase 3 against v1.1, then commit v1.1 as LEDGER #0004 — never v1.0. This is the same discipline applied successfully in the planet-time-bench v1.0 → v1.1 → v1.2 progression.

---

## Sweep methodology

For each attack:

1. **Mechanism** — concrete description of how the attack works.
2. **Targeted assertions** — which v1.0 assertions the attack tests.
3. **Stated defense (per §4 of CADUCEUS-001)** — what the v0.1 spec claimed would block the attack.
4. **Probe** — adversarial examination of whether the stated defense actually holds.
5. **Verdict** — SURVIVES / PARTIAL GAP / GAP, with explicit reasoning.
6. **Delta for v1.1** — if GAP, the specific assertion revision required.

The sweep is adversarial: the goal is to find Goodhart paths and assertion holes that would let a v0.1 substrate implementation *pass* caduceus-bench v1.0 while still being broken in deployment. A "this works in principle" stated defense is not sufficient — the assertion text must close the loop.

---

## Attack 1 — Replay across long RTT

**Mechanism.** Adversary captures a legitimately signed bundle (e.g., a "telemetry override accepted" status) during nominal ops. Months later, after a release-key rotation event, adversary re-injects the bundle. If T1 verifies against an archive of past-valid keys — which it must, to verify legitimate historical bundles in mission logs — the bundle passes T1.

**Targeted assertions.** T1 (signature verification), T2 (Civium gate).

**Stated defense.** Bundle TTL + nonce in signed payload + Aletheia prev_hash linkage.

**Probe.**

- **TTL.** BP7 has a lifetime field, set by sender. v1.0 does not require T1 to verify TTL. A bundle whose TTL has expired but whose signature is still valid would pass T1 as written. **GAP.**
- **Nonce in signed payload.** Including a nonce in what is signed makes each bundle's signature unique, but the signature still verifies on replay. Replay *detection* requires a receiver-side nonce-seen-set with a retention policy at least as long as max(TTL). Not in spec. **GAP.**
- **Aletheia prev_hash linkage.** v0.1 chain semantics imply forward verification, but T1 as written only verifies the *individual signature*, not the *chain*. Cross-key-epoch chain gaps are also unaddressed. **GAP.**

**Verdict.** **GAP.**

**Delta for v1.1.**

- **T1 revised:** "Every outbound bundle carries an Ed25519 signature over (bundle_id ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash ∥ nonce) AND a BP7 lifetime field. Receiver MUST verify (signature, lifetime-not-expired, nonce-not-seen, prev_hash-matches-last-accepted-within-authority-stream)."
- **New T4 (replay-prevention floor):** "Receiver maintains a nonce-seen-set per authority stream with retention ≥ max-TTL + grace. Bundles failing any of the four T1 checks deterministically fall back to HF-9. 100% of replay attempts logged on Aletheia chain."

---

## Attack 2 — Frame confusion (broadened)

**Mechanism.** Two variants.

- *Naive.* A UTC-timestamped bundle is delivered into a Mars-MTC operational context. Crew or system interprets the timestamp in the wrong frame.
- *Subtle.* A bundle carrying `vbx-body:v1.0:earth?frame=ITRF2020` URN is delivered to a Mars-based PHRONESIS. The command payload itself does not reference a body. F2 as written only addresses *command* bundles; telemetry, biomedical advisories, and status bundles slip through with frame confusion intact.

**Targeted assertions.** F1, F2, F3.

**Stated defense.** F1 + F2 + F3 combined; URN-mediated.

**Probe.**

- F1 ensures the URN is *present*. It does not assert what the receiver does with a URN that does not match local operational context.
- F2 says cross-body commands are rejected unless an authorized frame transformation exists. The text says **"command bundles"** — telemetry and advisories are not commands and are therefore not covered.
- F3 says EPHEMERIS alerts on URN mismatch. Alerts can be dismissed; alerts do not block action.

The deeper structural issue: v1.0 does not define what "act on" means for non-command bundles. Most interplanetary bundle traffic is not commands — it is telemetry, status, advisory, and informational. All of these can affect crew decisions and downstream automated action. **GAP.**

**Verdict.** **GAP.**

**Delta for v1.1.**

- **F2 revised:** "All bundles whose payload class is in {command, telemetry-acted, advisory-medical, advisory-safety, configuration, status-actionable} are rejected if the URN's (body, frame) is not in the local authorized-context set."
- **New F4 (frame-context interlock):** "For every bundle delivered to a non-display side-effect path (gate, log-as-acted, command-execution, alert-state-change), receiver MUST first verify URN-context match against local PHRONESIS operational context. Mismatch raises crew alert AND blocks the side-effect path until crew confirms explicitly via EPHEMERIS peer (read-and-attest acknowledgment)."

---

## Attack 3 — Bundle fragmentation under DTN custody transfer

**Mechanism.** BP7 supports fragmentation. A 1 MB bundle is fragmented to 4 × 256 KB. Each fragment carries fragment metadata and is custody-transferable independently. Two sub-attacks:

- *Partial-action.* Receiver implementation has any "fast-path" for partial-fragment processing (display preview, partial log entry, intent-prediction).
- *Reassembly spoofing.* Adversary sends a "duplicate" first fragment with a different payload after the legitimate first fragment is in flight. Receiver's reassembly logic may accept the spoofed fragment if cross-fragment binding is not enforced.

**Targeted assertions.** B1, B2.

**Stated defense.** Signature covers reassembled bundle; partial-fragment signatures are not valid command authorization.

**Probe.**

- The stated defense is correct in principle. v1.0 does not assert it explicitly. A receiver implementation could pass B1 and B2 as written while still leaking on partial-fragment processing.
- BP7 fragmentation has a well-known integrity solution: the Bundle Protocol Security Block (BPSec, RFC 9172). v1.0 does not invoke BPSec. **GAP.**
- Anti-spoofing for fragments requires that fragments with the same bundle-ID but different fragment-offsets be bound by the originator's signature, which BPSec provides.

**Verdict.** **PARTIAL GAP.**

**Delta for v1.1.**

- **B2 revised:** "Store-and-forward integrity through ≥ 3 hops with simulated 60-min connectivity outage between hops 2 and 3 AND with bundle fragmentation invoked (BPSec, RFC 9172, applied for cross-fragment integrity). Receiver produces zero side-effects (no display, no alert, no log-as-acted, no command-execution, no gate-decision-logged-as-pending) until full reassembly AND full signature verification."

---

## Attack 4 — Trust-anchor compromise at relay

**Mechanism.** Adversary compromises a Mars relay orbiter (physically or via supply-chain). Adversary has full bundle-forwarding control: drop, delay, modify.

**Targeted assertions.** T1, T2.

**Stated defense.** End-to-end Aletheia signature, not hop-by-hop; relay can drop but cannot forge.

**Probe.**

- End-to-end signature is sound. Adversary cannot forge new bundles purporting to be from authorized originators. ✓
- Adversary *can* drop, delay (to force TTL expiry — addressed if Attack 1 fix lands), replay (also addressed), reorder.
- Reordering breaks Aletheia chain forward-verification — receiver detects this if T1 revised per Attack 1 fix. ✓
- Bundle loss is a denial-of-service issue, not an integrity issue. Out of scope for the trust layer. Should be flagged for ACI monitoring (Attack 5 territory).

**Verdict.** **SURVIVES** *(conditional on Attack 1 fix landing).*

**Delta for v1.1.**

- No new assertion required, *but* add to v1.1 explanatory notes: "T1 + T4 close this attack, including chain-gap detection (forward-verification of prev_hash). Bundle drop is a DoS condition flagged by HF-18 (ACI) for crew awareness, not a trust-layer failure."

---

## Attack 5 — ACI thrashing (including slow-drift variant)

**Mechanism.** Two variants.

- *Noisy thrash.* Adversary injects calibrated noise into the RF channel to push ACI drift monitor into low-confidence band. If CORE escalates to HF-9 too aggressively, this becomes a denial-of-service attack on operations.
- *Slow-drift.* Over months, adversary slowly degrades link signal-to-noise. ACI's adaptive conformal bands widen to accommodate the gradual change. Eventually, real attacks falling within the now-widened bands do not trigger ACI alerts. The drift monitor has been *adversarially trained* to ignore the attack.

**Targeted assertions.** HF-18 (substrate floor), implied in caduceus-bench but not directly asserted.

**Stated defense.** Tunable alpha update (per VBX-ISPS bug fix #2 lesson); ACI does not directly gate authority — raises a CORE-monitored alert.

**Probe.**

- ACI as monitoring-not-gating is correct design. ✓
- v1.0 does not assert what CORE *does* with ACI alerts. Escalation policy is unspecified.
- v1.0 does not bound how wide ACI's conformal bands can grow. The slow-drift attack succeeds because there is no maximum-acceptable-bandwidth assertion. **GAP.**

**Verdict.** **GAP.**

**Delta for v1.1.**

- **New T5 (link-integrity floor):** "ACI's conformal bandwidth on (BER, SNR, RTT) is bounded above by a mission-policy maximum. Bandwidth exceeding the maximum raises a CORE-class alert that informs crew the link is operating below ops-safe integrity, independent of any individual bundle's verification. Alpha-update has a lower bound preventing indefinite band expansion."
- **HF-18 substrate update (v0.2 build):** Mercury kernel monitors ACI alpha and bandwidth; explicit alert classes (link-degraded, link-untrusted, link-down) with crew-visible escalation policy.

---

## Attack 6 — EPHEMERIS-PHRONESIS pairing MitM

**Mechanism.** Adversary places a UWB device between EPHEMERIS and PHRONESIS. Two variants.

- *Pairing-time MitM.* During initial pairing, adversary intercepts NFC pairing seed.
- *Run-time MitM.* After legitimate pairing, adversary tries to substitute bundles in the UWB peer stream.

**Targeted assertions.** F3.

**Stated defense.** Physical-proximity attestation (NFC pairing seed) + Ed25519 mutual auth + bundle re-signing by PHRONESIS for EPHEMERIS display.

**Probe.**

- NFC pairing seed ~4 cm range — interceptable with specialized hardware but operationally hard. ✓ (acceptable for v0.1)
- Ed25519 mutual auth after pairing closes the run-time MitM. ✓
- Bundle re-signing by PHRONESIS for EPHEMERIS display means EPHEMERIS trusts only PHRONESIS-local signatures, not remote ones. **This is the core defense** and it holds. ✓
- *However:* the stated defense assumes EPHEMERIS itself is trustworthy. If EPHEMERIS firmware is compromised (supply-chain attack), the entire crew-attestation-peer model collapses silently — EPHEMERIS displays "all signatures valid" while doing nothing.

**Verdict.** **SURVIVES** *(with noted assumption: EPHEMERIS supply-chain integrity is assumed; tamper-evident packaging + signed firmware on EPHEMERIS is a parallel requirement, not addressed by Caduceus alone).*

**Delta for v1.1.**

- No assertion change required, *but* add to v1.1 §0 scope statement: "EPHEMERIS firmware integrity and tamper-evident packaging are assumed and are out-of-scope of caduceus-bench. EPHEMERIS supply-chain attestation is a parallel requirement to be addressed by VBX-01 v0.3+ firmware signing and packaging discipline."
- **v0.2+ stretch:** dual-peer attestation. Mission commander's EPHEMERIS + crew member's EPHEMERIS must independently agree on (URN, signature). Two independent watches = redundant trust against single-EPHEMERIS compromise.

---

## Attack 7 — A1d seam attack

**Mechanism.** A1d (out-of-scope signaling) in planet-time-bench v1.2 was added to handle pre-1972 and post-2100 UTC inputs at the *render* layer. Adversary crafts a bundle with a timestamp in the out-of-scope range. If the comm fabric processes the bundle (signature verification, gate decision, audit log) *before* the renderer catches the out-of-scope timestamp, the bundle has caused side effects despite being out-of-scope.

**Targeted assertions.** F1.

**Stated defense.** F1 rejects un-tupled bundles; A1d signaling preserved.

**Probe.**

- F1 requires the URN tuple to be present. ✓
- F1 does not require the timestamp to be in the validated envelope. **GAP.**
- A1d is at the RENDER layer. Comm-fabric side effects (gate, log, alert, command) occur *before* render in the pipeline. By the time the renderer flags out-of-scope, the gate has already passed (or denied), the audit log has the bundle as "processed," and any cascading actions may already have started.

**Verdict.** **GAP.**

**Delta for v1.1.**

- **F1 revised:** "Every bundle carries a (TAI_microseconds, vbx-body-URN) tuple in a BP7 extension block, where the URN conforms to Caduceus-BodyID v1.0 AND the timestamp is within the planet-time-bench validated envelope (1972-01-01 ≤ TAI ≤ 2100-12-31). Out-of-envelope timestamps cause F1 rejection at ingress, before any T-layer or B-layer processing."
- **New T6 (ingress validity floor):** "Receiver MUST verify F1 (URN + timestamp envelope) before any T-layer verification, B-layer custody decision, or render-layer pass. F1-rejected bundles are quarantined: logged with their full content, alert raised, no side effects. Quarantine class is distinct from signature-failure class so that operational responses differ."

---

## Attack 8 — Namespace exhaustion / parser attacks

**Mechanism.** Adversary constructs adversarial URNs.

- *Length attack.* URN > 256 bytes, attempting length-prefix overflow.
- *Unregistered claim.* URN claims an unregistered body or frame.
- *Grammar violation.* `vbx-body:v1.0:earth/leo//iss?frame=` — empty path level, malformed.
- *Unicode look-alike.* `vbx-body:v1.0:еarth?frame=ITRF2020` where the first "e" is Cyrillic, not ASCII.
- *Registry race.* During v1.0 → v1.1 namespace transition, exploit timing window when registry is partially updated.

**Targeted assertions.** §3.5 wire-format binding, F1.

**Stated defense.** Strict registry check, hard 256-byte cap with rejection on overflow, no fallback parsing.

**Probe.**

- 256-byte cap with hard rejection: ✓
- Strict registry check on bodies and frames: ✓
- Grammar validation: not asserted in v1.0. A grammar-violating URN within registered bodies could still appear "registered" by a naive substring search. **GAP.**
- Unicode normalization: not asserted. Look-alike attacks work. **GAP.**
- Atomic registry transition: not asserted. Concurrent ops during commit window are undefined. **GAP.**

**Verdict.** **PARTIAL GAP.**

**Delta for v1.1.**

- **§3.5 wire-format binding revised:** "URN parsing MUST: (a) enforce the EBNF grammar strictly — grammar violations cause rejection at ingress; (b) apply Unicode NFC normalization and enforce ASCII-only canonical form on path and frame fields; (c) lookup registered bodies and frames against the v1.0 registry only — partial-match or substring-match is rejection; (d) registry updates (v1.x bumps) are atomic on the Aletheia ledger — operations during the commit window use the prior-version registry until commit finalizes."

---

## Newly discovered attacks

### Attack 9 — Slow-drift adversarial ACI training

Folded into Attack 5 above. Addressed by alpha-update lower bound and maximum-bandwidth alert (T5).

### Attack 10 — Authority-graph staleness across RTT

**Mechanism.** Civium authority graph is updated periodically. Updates originate at ground and must propagate to PHRONESIS. Propagation delay = link RTT (up to 44 min Earth-Mars). Adversary, knowing they are about to be revoked, sends a flurry of bundles signed under their soon-to-be-revoked authority during the propagation window. PHRONESIS's local Civium gate verifies against the *not-yet-updated* authority graph — bundles pass T2.

**Targeted assertions.** T2 (Civium gate).

**Probe.** v1.0 T2 says inbound bundles are verified and passed through the Civium authority gate. v1.0 does not address:

- Time-bound authority assertions (this authority is valid for this scope until TAI epoch X).
- Active revocation propagation (revocation as a bundle class with its own freshness semantics).
- Conservative defaults under stale authority graph.

This is an interplanetary-specific failure mode. On Earth, authority revocation propagation is sub-second. On Mars, it is minutes. The compliance-graded thesis fails during the propagation window without explicit revocation handling.

**Verdict.** **GAP** (and the most consequential one in this sweep — it would not have appeared in a non-interplanetary deployment).

**Delta for v1.1.**

- **T2 revised:** "Every inbound bundle is verified against (a) the current local Civium authority graph AND (b) a time-bound validity assertion (this authority is valid for this scope until TAI epoch X, where X is signed as part of the authority assertion)."
- **New T7 (revocation propagation):** "Revocation events are themselves Caduceus bundles, propagated end-to-end with Aletheia signature, on a dedicated authority-revocation stream. Receiver MUST verify revocation bundles before any command-class bundle from the revoked authority. If the local authority graph has not received an authoritative refresh within mission-policy interval (e.g., 24 h for ground-link-available ops, 7 days for connectivity-degraded ops), conservative defaults apply: increase Civium gate scrutiny class, surface authority-stale alert on EPHEMERIS."
- **HF-17 substrate update (v0.2 build):** authority-graph freshness tracker per stream, revocation-stream dedicated handling, conservative-default escalation logic.

### Attack 11 — Side-channel on Ed25519 signing

**Mechanism.** Aletheia signing operations execute on the Orin NX. Power-analysis or timing-channel attacks on the Ed25519 implementation could leak the release private key.

**Targeted assertions.** T1 (substrate-side).

**Probe.** v1.0 T1 specifies Ed25519 but does not require constant-time implementation. A naive Ed25519 implementation has timing characteristics that depend on key bits — exploitable via timing analysis. Power-analysis is a hardware-level concern but mitigable in software via constant-time arithmetic.

**Verdict.** **GAP** *(substrate-side, but benchmark must assert it).*

**Delta for v1.1.**

- **T1 detail:** "Ed25519 implementation MUST be constant-time (e.g., libsodium, ref10 hardened). Receiver-side verification MUST also be constant-time to defeat timing-based key-existence probing."
- **HF-16 / HF-17 substrate update (v0.2 build):** specify libsodium (or audited equivalent) as the binding cryptographic library; reject naive crypto implementations.

### Attack 12 — Cold-boot / suit-capture

**Mechanism.** Adversary physically captures a PHRONESIS unit (e.g., after crew incapacitation, or via theft of a between-mission stored suit). Adversary extracts the release private key from PHRONESIS storage.

**Targeted assertions.** T1 (substrate-side, hardware boundary).

**Probe.** Caduceus signs outbound bundles. If adversary extracts the suit's signing key, adversary can forge outbound telemetry as if from that suit. Two design responses:

- *Defensive:* secure-element-stored keys with tamper-evident enclosure. PHRONESIS blueprint shows a secure-element module (Ed25519 with shield); existing posture is partially aligned.
- *Asymmetric:* PHRONESIS holds signing capability for *outbound telemetry* only — never the *master authority key* for inbound commands. Commands originate from ground signers whose keys are not in any deployed suit. This asymmetry contains the blast radius of a suit capture: adversary can forge that suit's telemetry but cannot forge commands to other suits or to ground.

**Verdict.** **NOTED** *(hardware-side; benchmark may flag, substrate must implement).*

**Delta for v1.1.**

- **§0 scope statement addition:** "Caduceus enforces key-authority asymmetry. PHRONESIS units hold per-suit telemetry-signing keys only. Master command-authority keys are held by ground signers and are never deployed to suits. Suit capture is bounded to per-suit telemetry forgery; suit capture cannot escalate to command authority over other crew."
- **HF-16 substrate update (v0.2 build):** confirm Secure Element storage of telemetry-signing key; specify tamper-evident enclosure of PHRONESIS CORE; specify zeroization on detected tamper.

---

## Summary table

| # | Attack | Verdict | v1.1 delta class |
|---|---|---|---|
| 1 | Replay across long RTT | GAP | T1 revised; new T4 (replay-prevention floor) |
| 2 | Frame confusion (broadened) | GAP | F2 revised; new F4 (frame-context interlock) |
| 3 | Bundle fragmentation | PARTIAL GAP | B2 revised (explicit BPSec + no-partial-side-effects) |
| 4 | Trust-anchor compromise at relay | SURVIVES (cond.) | Explanatory note only |
| 5 | ACI thrashing + slow-drift | GAP | New T5 (link-integrity floor); HF-18 substrate update |
| 6 | EPHEMERIS-PHRONESIS MitM | SURVIVES (cond.) | §0 scope note; v0.2+ dual-peer attestation stretch |
| 7 | A1d seam | GAP | F1 revised; new T6 (ingress validity floor) |
| 8 | Namespace exhaustion | PARTIAL GAP | §3.5 wire-format revised (grammar + Unicode + atomic) |
| 9 | Slow-drift ACI training | (folded into 5) | — |
| 10 | Authority-graph staleness | GAP | T2 revised; new T7 (revocation propagation); HF-17 substrate update |
| 11 | Side-channel on Ed25519 | GAP | T1 detail (constant-time); HF-16/17 substrate update |
| 12 | Cold-boot / suit-capture | NOTED | §0 scope (key-authority asymmetry); HF-16 substrate update |

---

## Required changes for caduceus-bench v1.1

**T-layer (trust):**
- T1 revised (TTL + nonce + chain + constant-time)
- T2 revised (time-bound authority)
- New T4 (replay-prevention floor)
- New T5 (link-integrity floor / ACI bounds)
- New T6 (ingress validity floor)
- New T7 (revocation propagation)

**F-layer (frame):**
- F1 revised (timestamp envelope)
- F2 revised (broadened beyond commands)
- New F4 (frame-context interlock for non-command bundles)

**B-layer (bundle):**
- B2 revised (BPSec + no-partial-side-effects)

**§3.5 wire-format binding:**
- Grammar enforcement
- Unicode NFC + ASCII-only canonical
- Registry atomic transition

**§0 scope statement:**
- EPHEMERIS supply-chain assumption flagged
- Key-authority asymmetry stated

**Substrate-side (v0.2 build) downstream impacts:**
- HF-16: constant-time Ed25519 (libsodium), Secure Element tamper handling
- HF-17: revocation propagation, authority-graph freshness tracker, conservative defaults
- HF-18: ACI alpha lower bound, max-bandwidth alert classes
- New: dual-peer attestation (v0.2+ stretch)

---

## Re-running Phase 3 against v1.1

The standard ZCS-6 discipline applies: produce caduceus-bench v1.1 with the deltas above, re-run all 12 attacks against v1.1, and add adversarial probing for any *new* attack surface introduced by the new assertions (e.g., the revocation-stream itself is a new target for spoofing and DoS).

Only when v1.1 passes a clean Phase 3 sweep is it eligible for LEDGER #0004 commit.

---

## Honest scope statement

- This sweep is analytical, not empirical. No code was executed; no actual radios were attacked; no real ACI implementation was thrashed.
- Confidence on attacks 1–8: HIGH (these are well-understood attack classes in adjacent literature).
- Confidence on attacks 10–12: HIGH for attack 10 (interplanetary-specific, follows directly from first-principles RTT analysis); HIGH for attack 11 (well-understood class); MEDIUM-HIGH for attack 12 (depends on the specific Secure Element substrate chosen at v0.2).
- The sweep is not exhaustive. A registered patent practitioner, an external red-team, and a formal-methods reviewer would each likely surface additional gaps. v1.1 should be circulated for external probe before LEDGER commit.
- All "delta for v1.1" specifications above are first-principles drafts and may themselves require revision in the v1.1 draft cycle.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-PHASE3-001 Rev A · ZCS-6 Phase 3 sweep report*
*Target: caduceus-bench v1.0 (CADUCEUS-001) · Verdict: REVISION REQUIRED → v1.1*
