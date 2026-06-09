# PPA Filing Target — Interplanetary-RTT-Aware Compliance-Graded Authority Propagation

| | |
|---|---|
| **Document ID** | CADUCEUS-IP-001 |
| **Revision** | A (skeleton — practitioner-input required before filing) |
| **Filing class** | U.S. Provisional Patent Application (35 U.S.C. § 111(b)) |
| **Author / inventor** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Assignee (anticipated)** | Visionblox LLC |
| **Drafting date (UTC)** | 2026-06-08 |
| **Status** | **PPA SKELETON — not a final filing.** Requires review and refinement by a registered USPTO patent practitioner before filing. Drafted to capture priority for the Caduceus Attack 10 discovery and to give the practitioner a substantial starting point. |
| **Source disclosure** | CADUCEUS-PHASE3-001 §"Attack 10" (initial discovery); CADUCEUS-002 §3.3 (T2 + T7); CADUCEUS-002 §3.6 (conservative-defaults table); CADUCEUS-003 §3.3 (T7 v1.2 succession rule) |

---

## Honest scope statement (read first)

This document is a **drafting target for a registered USPTO patent practitioner**, NOT a filing-ready application. Claim language, enablement disclosure, and prior-art positioning are first-principles drafts. Final filing requires:

1. Practitioner review and revision of all claim language
2. Formal prior-art search across USPTO, EPO, WIPO, and academic literature
3. Practitioner judgment on § 101 abstract-idea risk and mitigation
4. Drawings prepared per 37 C.F.R. § 1.84 formal requirements (if drawings are filed with the PPA; PPAs do not require drawings but benefit from them for priority-establishment of figure-supported embodiments)

The user (Khaalis) has independently noted in the project context that **a registered patent practitioner must be engaged** before substantive filings. This document supports that engagement, not bypasses it.

---

## 1. Field of the invention

The invention relates to compliance-graded authority propagation in delay-tolerant networking systems, particularly in the context of crewed interplanetary missions where communication round-trip times between distributed compute nodes are bounded below by celestial-body separation divided by the speed of light, producing propagation windows that range from seconds (cislunar operations) to minutes (Earth-Mars) to multiple hours (outer Solar System operations).

---

## 2. Background of the invention

### 2.1 The propagation-window gap in existing authority systems

Conventional public-key authority infrastructures (PKI) including X.509 Certificate Revocation Lists (CRL), Online Certificate Status Protocol (OCSP), and similar mechanisms assume propagation latencies of seconds or less between authority issuers, revocation publishers, and authority consumers. This assumption is generally valid on terrestrial networks where the speed-of-light propagation budget for global revocation propagation is on the order of 250 ms (one-way Earth-circumference) plus network-processing overhead measured in additional hundreds of milliseconds.

Kerberos and related symmetric-key authority systems use time-bounded tickets but similarly assume rapid revocation. The Burrows-Abadi-Needham (BAN) logic formalization of authority assumes the time required for authority assertions to propagate is small relative to the lifetimes of those assertions.

In interplanetary contexts, this assumption fails. Earth-to-Mars one-way light-time varies from approximately 3 to 22 minutes depending on planetary alignment; round-trip times accordingly range from approximately 6 to 44 minutes. Outer-Solar-System missions (Jupiter, Saturn, beyond) experience one-way light-times of tens of minutes to hours. During these propagation windows, a revoked authority remains locally trusted at any compute node that has not yet received the revocation, and there is no physical mechanism by which the revocation can arrive faster.

Existing space-network standards including Consultative Committee for Space Data Systems (CCSDS) Space Data Link Security (SDLS) address this gap by using pre-shared symmetric keys without authority-graph semantics. This approach does not generalize to multi-party, multi-role, dynamically-updated authority structures characteristic of large crewed missions involving multiple ground organizations, mission-support contractors, and crew members with role-specific permissions.

### 2.2 The compliance-graded gap

Separately, compliance-graded systems — wherein authority decisions are gated not only by signature validity but also by an explicit authority-graph specifying which originator may issue which command class to which receiver in which context — are well-developed for terrestrial deployments (e.g., regulated-edge healthcare, financial transaction systems) but have not been extended to interplanetary deployments.

The unmet need is a system that combines compliance-graded authority decisions with explicit accommodation of bounded but non-zero propagation windows.

### 2.3 The interplanetary-specific failure mode

A compromised authority that knows it is about to be revoked has a propagation-window opportunity to issue malicious bundles that will pass authority verification at remote compute nodes which have not yet received the revocation. The duration of this opportunity scales with the worst-case propagation delay in the network — i.e., minutes to hours in interplanetary deployments versus milliseconds on Earth networks.

No prior art known to the inventors specifically addresses this failure mode through systematic combination of: time-bound authority assertions, dedicated revocation streams, stale-graph default policies, succession-implies-revocation rules, and precedence-ordered conservative defaults under ambiguous classification.

---

## 3. Summary of the invention

The invention provides a system and method for compliance-graded authority propagation across interplanetary distances. Distinguishing features of the invention include, in combination:

**(a)** Time-bound authority assertions wherein each authority is cryptographically signed by a master key and includes an explicit expiration time encoded in a precise time standard (preferably Temps Atomique International, TAI).

**(b)** A dedicated revocation stream cryptographically separated from the authority assertion stream, wherein revocation events are issued as discrete bundles signed by the same master key that issued the corresponding authority, and wherein the revocation stream is processed at higher priority than other bundle streams.

**(c)** A stale-graph default policy that, when a receiving compute node has not received an authoritative refresh of the local authority graph within a configurable mission-policy interval, applies enumerated conservative defaults specified per payload class.

**(d)** A succession-implies-revocation default rule wherein issuance of a successor authority for a given scope automatically revokes any predecessor authority for that scope at the receiving node, independent of and prior to receipt of an explicit revocation bundle.

**(e)** A precedence ordering for the stale-graph default policy that, when a bundle's payload class matches multiple default rows, applies the most-restrictive enumerated default.

The combination yields a system that maintains soundness of authority decisions during propagation windows whose duration exceeds the round-trip light-time between participating compute nodes — a circumstance not addressed by prior art authority propagation mechanisms.

---

## 4. Detailed description (enablement disclosure)

### 4.1 System architecture

A compliance-graded authority propagation system in accordance with the invention comprises:

- **A plurality of compute nodes** distributed across at least two celestial bodies (e.g., Earth ground station, Mars surface PHRONESIS-1 unit, Earth-Moon Lagrange relay). Each node executes a local authority verification engine.
- **A bundle fabric layer** preferably implemented as Bundle Protocol version 7 (RFC 9171, June 2022) or equivalent delay-tolerant networking protocol supporting custody transfer, store-and-forward, and bounded lifetime.
- **A cryptographic substrate** preferably using Ed25519 digital signatures with constant-time implementation (e.g., libsodium or equivalent audited library), supporting per-bundle attestation rooted in a master-key infrastructure.
- **A local authority graph** maintained at each compute node, storing authority assertions, time-bound validity epochs, scope specifications, and freshness timestamps.
- **A nonce-seen-set** per authority stream, partitioned with bounded per-stream quota.

### 4.2 Time-bound authority assertion structure

Each authority assertion comprises, at minimum:

- An authority identifier (preferably a public key or hash thereof)
- A scope specification (which bundle classes the authority may issue, to which recipients, in which contexts)
- A validity-from TAI epoch
- A validity-until TAI epoch
- A master-key signature over the above

### 4.3 Revocation stream

Revocation bundles travel on a dedicated stream prioritized above all other bundle classes for processing at the receiving node. Each revocation comprises, at minimum:

- A revocation-target authority identifier
- A revocation-effective TAI epoch
- A master-key signature

Receiving nodes process revocation bundles before any other inbound bundles in the same processing cycle. Revocation bundles are stored persistently as part of the local authority graph and survive subsequent bundle deliveries from the revoked authority.

### 4.4 Stale-graph default policy

A configurable mission-policy interval (default: 24 hours in ground-link-available operating context; 7 days in connectivity-degraded context) defines the maximum age of authoritative refresh of the local authority graph. When the interval is exceeded, the local node applies enumerated conservative defaults to all subsequent bundles, varying by payload class.

Example default table (non-limiting):

| Payload class | Ground-link-available stale (24h+) default | Connectivity-degraded stale (7d+) default |
|---|---|---|
| Command (safety-critical) | Require dual-authority confirmation | Reject; queue for refresh |
| Command (non-safety) | Drop trust tier by one (require additional crew confirmation) | Drop two tiers; require crew + ground refresh |
| Telemetry-acted | Drop trust tier by one | Reject; queue for refresh |
| Advisory-medical / advisory-safety | Display with stale-graph banner | Display with persistent stale-graph banner |
| Emergency-class | Bypass T7 gating | Bypass T7 gating |
| Revocation bundles | Process at highest priority regardless of staleness | Process at highest priority regardless of staleness |

### 4.5 Succession-implies-revocation rule

When a receiving node observes the arrival of an active successor authority for a given scope — defined as a new authority assertion whose scope-specification overlaps with that of a predecessor — the node automatically treats the predecessor as revoked at the receiving node, even if an explicit revocation bundle for the predecessor has not yet arrived. This rule closes a critical timing gap wherein an attacker who has compromised the predecessor key could exploit the propagation window between successor issuance and explicit revocation arrival.

### 4.6 Precedence ordering for ambiguous class

When a bundle's payload class matches multiple rows in the stale-graph default policy table, the most-restrictive default applies. A precedence ordering is established at the time of authority-graph initialization and is itself a master-key-signed system parameter. An example precedence ordering, non-limiting:

`Configuration > Command (safety-critical) > Telemetry-acted > Command (non-safety) > Status-actionable > Advisory-medical = Advisory-safety > Display-only/informational`

Emergency-class and Revocation bundles bypass this precedence ordering per their dedicated table rows.

### 4.7 Operating example (Earth-Mars context)

Assume an Earth ground station issues authority A to crew member C-1 at Mars surface PHRONESIS unit P-1, with validity-from = T_0 and validity-until = T_0 + 90 days. At T_0 + 30 days, ground discovers that authority A has been compromised and issues both: (i) a revocation bundle for A, and (ii) a successor authority A' for C-1.

In conventional PKI, the propagation delay from Earth to Mars (up to 22 minutes one-way) provides a window during which P-1 has not yet received either the revocation or the successor. During this window, the compromised A key can be used to issue malicious bundles to P-1 that pass authority verification.

In the inventive system:

1. The succession-implies-revocation rule (§ 4.5) ensures that as soon as A' arrives at P-1 (within the same propagation window), A is locally revoked at P-1 even before the explicit revocation bundle for A arrives.
2. If A' arrives before the explicit revocation bundle, P-1 has already locally revoked A; subsequent bundles signed by A fail verification.
3. If the explicit revocation bundle arrives first (e.g., via a different relay path with different propagation delay), it propagates through the priority lane and is processed before any bundles in standard streams.
4. During the worst-case window (neither A' nor explicit revocation has yet arrived), the stale-graph default policy (§ 4.4) is the additional defense: if the local authority graph has not refreshed within the mission-policy interval (which an attacker cannot extend), commands from any authority including A are subject to conservative defaults.

The combination provides defense-in-depth specifically calibrated for the interplanetary propagation context, addressing a failure mode that has not been previously addressed in the prior art.

---

## 5. Claims (DRAFT — practitioner refinement required)

### Independent claim 1 (system)

A system for compliance-graded authority propagation across distances at which round-trip light-time exceeds one second, comprising:

a plurality of compute nodes geographically distributed across at least two celestial bodies, each compute node maintaining a local authority graph;

an authority assertion stream wherein each authority is cryptographically signed by a master key and includes a time-bound validity assertion comprising a validity-from epoch and a validity-until epoch encoded in International Atomic Time;

a dedicated revocation stream cryptographically separated from the authority assertion stream, wherein revocation events are signed by the same master key that issued the corresponding authority, and wherein the revocation stream is processed at higher priority than non-revocation bundle streams at each compute node;

a stale-graph default policy module at each compute node, applying enumerated conservative defaults when the local authority graph has not received an authoritative refresh within a mission-policy interval, the conservative defaults varying by payload class;

a succession-implies-revocation rule at each compute node wherein issuance of a successor authority for a given scope automatically revokes predecessor authorities for that scope at the receiving compute node, independent of and prior to receipt of an explicit revocation bundle; and

a precedence ordering applied by the stale-graph default policy module when a bundle's payload class matches multiple conservative-default rows, wherein the most-restrictive default applies;

such that the system maintains soundness of authority decisions during propagation windows whose duration exceeds the round-trip light-time between participating compute nodes.

### Independent claim 2 (method)

A method for compliance-graded authority propagation across distances at which round-trip light-time exceeds one second, comprising:

issuing time-bound authority assertions from a master-key-controlled issuer, each authority comprising a validity-from epoch, a validity-until epoch, a scope specification, and a master-key signature;

propagating revocation events on a dedicated revocation stream prioritized above non-revocation bundle streams;

at a receiving compute node, applying enumerated conservative defaults to incoming bundles when the local authority graph has not received an authoritative refresh within a configurable mission-policy interval;

at the receiving compute node, treating issuance of a successor authority for a given scope as implicit revocation of predecessor authorities for that scope, independent of and prior to receipt of an explicit revocation bundle; and

when an incoming bundle's payload class matches multiple stale-graph default rows, applying the most-restrictive default per a configured precedence ordering.

### Dependent claims (sketches)

**Claim 3.** The system of claim 1 wherein the bundle fabric is conformant to Bundle Protocol version 7 (RFC 9171).

**Claim 4.** The system of claim 1 wherein the authority assertion signatures are Ed25519 signatures generated with constant-time implementation.

**Claim 5.** The system of claim 1 wherein the mission-policy interval is configurable per operating context, with a first interval applicable when a ground link is available and a second longer interval applicable when connectivity is degraded.

**Claim 6.** The system of claim 1 wherein the stale-graph default policy is itself cryptographically signed by the master key and is part of the locked authority-graph state.

**Claim 7.** The system of claim 1 wherein the system includes an alert mechanism that informs operators when the local authority graph staleness exceeds the mission-policy interval.

**Claim 8.** The system of claim 1 wherein the precedence ordering includes at least seven distinct payload-class categories.

**Claim 9.** The system of claim 1 wherein revocation bundles include freshness tokens that prevent replay of older revocations after a re-authorization event.

**Claim 10.** The system of claim 1 wherein the compute nodes include at least one node on a celestial-body surface and at least one node in an orbit around a celestial body.

**Claim 11.** The system of claim 1 wherein the at least two celestial bodies include Earth and at least one other body selected from the group consisting of Moon, Mars, asteroids, and outer-solar-system bodies.

**Claim 12.** The method of claim 2 further comprising recording authority decisions, revocation events, and stale-graph default applications in a hash-chained provenance ledger using Ed25519 signatures.

**Claim 13.** The system of claim 1 wherein the dedicated revocation stream is implemented as a higher-priority bundle class within a delay-tolerant networking fabric.

**Claim 14.** The system of claim 1 further comprising a fail-safe transition mechanism wherein authority-verification failure deterministically transitions the receiving compute node to a hard safety floor independent of any LLM-mediated decision.

**Claim 15.** The system of claim 1 wherein the conservative defaults include rejecting bundles whose payload class is not in an enumerated set.

---

## 6. Drawings to be prepared

| Figure | Subject |
|---|---|
| FIG. 1 | System overview: two compute nodes on two celestial bodies with bidirectional bundle fabric, including authority issuer at Earth ground and receiving node at Mars surface |
| FIG. 2 | Authority assertion structure (data layout) with time-bound validity fields |
| FIG. 3 | Revocation stream priority lane within bundle fabric architecture |
| FIG. 4 | Stale-graph default policy table (mapping payload class to default behavior) |
| FIG. 5 | Succession-implies-revocation state machine at receiving node |
| FIG. 6 | Timeline diagram: authority issuance, compromise, revocation, succession, and propagation window with propagation delays explicitly shown |
| FIG. 7 | Precedence-ordering flowchart for multi-row payload classes |
| FIG. 8 | Fail-safe transition state machine showing receiving-node behavior under verification failure |

Drawings need not be filed with the PPA but improve later non-provisional support.

---

## 7. Prior-art positioning (practitioner-input required)

Likely prior-art categories to search and distinguish:

| Category | Representative prior art | Distinguishing position |
|---|---|---|
| Symmetric-key authentication systems | Kerberos (Steiner et al. 1988; RFC 4120) | Distinguishes on dedicated revocation stream + stale-graph defaults + interplanetary propagation context; Kerberos has TTL but not the system combination |
| Formal authority logics | Burrows-Abadi-Needham logic (1989) | Theoretical foundation; not a system or method patent; cite as background |
| X.509 certificate revocation | X.509 CRL (RFC 5280), OCSP (RFC 6960) | Distinguishes on dedicated revocation stream architecture + propagation-window-aware defaults + succession-implies-revocation rule; X.509 assumes sub-second propagation |
| Space data link security | CCSDS SDLS Blue Book | Distinguishes on graph-authority semantics versus SDLS pre-shared symmetric model |
| Delay-tolerant networking | RFC 9171 (BP7), Bundle Protocol Security (BPSec, RFC 9172) | Bundle fabric is substrate, not novelty; distinguishes on compliance-graded authority decisions over the fabric |
| Formal-verified policy enforcement | Picachv (Hu et al. 2024); PrivPolicy | Different problem class — these enforce data-use policies, not authority propagation |
| Blockchain-style hash-chained provenance | Bitcoin, Cardano, etc. | Provenance substrate is conceptually similar (Aletheia DAC is the user's own prior work); distinguishes on interplanetary-RTT-aware authority over a hash-chained substrate |

The novelty position is the **system combination of (a)–(e) in §3** operating in the **interplanetary propagation context**. Individual elements have prior art; the combination has not been previously claimed in the inventors' knowledge.

---

## 8. § 101 abstract-idea risk and mitigation

Patent claims framed as "method of doing compliance" face Alice/Mayo § 101 rejection. Mitigation strategies for the practitioner's consideration:

1. **Tie claims to concrete technological improvements:** atomicity locks, dedicated priority lanes, fail-safe state transitions, specific cryptographic substrate (Ed25519 constant-time).
2. **Tie claims to physical substrate:** hardware secure elements (referenced in PHRONESIS HF-8), compute nodes on celestial-body surfaces, specific bundle protocol structures.
3. **Frame as solution to specific technical problem:** the propagation-window failure mode in interplanetary authority systems is a specific technical problem not addressed by prior art — exactly the framing that survives Alice (cf. *Enfish v. Microsoft*, *Berkheimer v. HP*).
4. **Include concrete reductions to practice in dependent claims:** specific intervals (24h / 7d), specific Ed25519 implementations, specific bundle protocol versions.

---

## 9. Filing strategy notes

| Item | Recommendation |
|---|---|
| **Filing class** | U.S. Provisional Patent Application under 35 U.S.C. § 111(b). Establishes priority date for 12 months. |
| **Foreign filing horizon** | Paris Convention 12-month window from PPA filing date for foreign-filing rights. |
| **Sequence with arXiv preprint** | **File PPA BEFORE arXiv preprint** to preserve foreign-filing rights. arXiv publication is a public disclosure that starts the foreign-filing clock. |
| **Coordination with cross-modal Civium PPA** | The cross-modal Civium provisional patent strategy (per project memory) covers procurement AI + physical AI + clinical AI as unified system-and-method. This Caduceus PPA can be filed (i) as standalone, (ii) as continuation-in-part of the cross-modal Civium PPA, or (iii) as a separate embodiment within an expanded cross-modal Civium PPA. Practitioner judgment required. Recommendation: standalone is cleanest for priority establishment of the interplanetary-specific novelty. |
| **Inventor declaration** | Khaalis is sole inventor of record per the discovery context. If any co-inventors emerge (formal-methods collaborator, mission-assurance consultant), declarations must be updated. |
| **Disclosure obligation** | Inventors have an ongoing duty of candor to the USPTO. Any prior art discovered after filing must be disclosed in an Information Disclosure Statement (IDS). |

---

## 10. Open items for practitioner engagement

1. **Formal prior-art search** — particularly on Kerberos / BAN / X.509 prior art combined with delay-tolerant networking literature.
2. **Claim language refinement** — current claims are first-principles drafts; practitioner refinement required for technical precision and § 112 (b) definiteness compliance.
3. **§ 101 risk review** — practitioner judgment on whether current framing survives Alice analysis.
4. **Foreign-filing strategy** — which jurisdictions (USPTO, EPO, JPO, others) and which sequence.
5. **Coordination with cross-modal Civium PPA filing strategy** — standalone vs continuation-in-part vs embodiment.
6. **Drawings preparation** — formal drawings per 37 C.F.R. § 1.84.
7. **Inventor identification** — confirm sole-inventor status.
8. **Assignee documentation** — Visionblox LLC assignment paperwork.

---

## 11. Honest scope statement (recap)

- This document is **drafting target material**, not a filing-ready PPA. Claim language requires practitioner review.
- Confidence on novelty: HIGH — the interplanetary-specific propagation-window failure mode and the §3 combination of (a)–(e) are, to the inventors' first-principles knowledge, not addressed in prior art. A formal practitioner search may surface adjacent or anticipating art that requires claim revision.
- Confidence on enablement: MEDIUM-HIGH — § 4 provides substantial technical detail; remaining gaps (BPSec/Aletheia key derivation per CADUCEUS-PROBE-001 P1.4) should be tightened either in this document or in an accompanying defensive arXiv supplement.
- Confidence on § 101 survival: MEDIUM — mitigation strategies in § 8 are credible but require practitioner judgment.
- **Caduceus-bench v1.2 LEDGER #0004 commit is INDEPENDENT of this filing** and should not be blocked by the PPA timing — but the PPA SHOULD be filed before the arXiv defensive preprint to preserve foreign-filing rights. Sequencing matters.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-IP-001 Rev A · PPA drafting target — Attack 10 interplanetary-RTT-aware authority propagation*
*Status: practitioner engagement required before filing*
