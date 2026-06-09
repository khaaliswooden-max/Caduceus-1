# PPA Filing Target — URN-Scheme-Versioned Dual-Version Retention Window for Interplanetary Registry Transitions

| | |
|---|---|
| **Document ID** | CADUCEUS-IP-002 |
| **Revision** | A (skeleton — practitioner-input required before filing) |
| **Filing class** | U.S. Provisional Patent Application (35 U.S.C. § 111(b)) |
| **Author / inventor** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Assignee (anticipated)** | Visionblox LLC |
| **Drafting date (UTC)** | 2026-06-08 |
| **Status** | **PPA SKELETON — not a final filing.** Practitioner refinement required. |
| **Source disclosure** | CADUCEUS-PHASE3-002 §"Attack 27" (discovery); CADUCEUS-003 §3.5 (v1.2 dual-version retention window mechanism); CADUCEUS-004 §3.5 (final commit form, unchanged from v1.2) |

---

## Honest scope statement (read first)

Same disclaimers as CADUCEUS-IP-001 §"Honest scope statement". Practitioner refinement, prior-art search, § 101 risk review, and drawings preparation all required. This document is drafting target material, not filing-ready papers.

---

## 1. Field of the invention

The invention relates to namespace registry transitions in distributed authority systems operating across communication channels whose round-trip latency exceeds the desired transition cutover time by orders of magnitude, particularly in interplanetary deployments where the speed-of-light propagation budget makes atomic registry transitions across all participating nodes physically impossible.

---

## 2. Background

### 2.1 The atomic-transition assumption

Conventional distributed namespace registries (DNS, X.509 CA certificate registries, programming-language standard libraries, etc.) treat version transitions either as instantaneous logical events (atomic commit-time semantics, where all nodes are assumed to see the transition simultaneously) or as gradual rollouts mediated by application-layer compatibility shims (e.g., HTTP/1.0 → HTTP/1.1 → HTTP/2 transitions on terrestrial networks).

These approaches presume that propagation latency between registry maintainer and registry consumers is small relative to the desired transition cutover time. On terrestrial networks, propagation is sub-second; on satellite-relay networks, propagation is single-digit-second; the assumption holds.

In interplanetary deployments, propagation latency is minutes to hours. A registry update committed at Earth ground at TAI epoch T cannot reach Mars surface PHRONESIS-1 before TAI epoch T + (up to 22 minutes one-way). During that window, Earth uses the new registry version while Mars uses the old. Bundles signed and dispatched during the window cross the version boundary asymmetrically.

### 2.2 Prior art on protocol versioning

Protocol versioning during transitions is well-explored in terrestrial computing: HTTP version negotiation, TLS cipher-suite negotiation, IPv4/IPv6 dual-stack operation. None addresses propagation windows of minutes or hours, and none specifies cryptographically-anchored transition windows tied to physics-bounded propagation.

### 2.3 The interplanetary-specific failure mode

A registry transition that proceeds asymmetrically across an interplanetary link creates a window in which:

- Bundles signed at Earth using new-version registry semantics arrive at Mars before the new registry has propagated; receiving node rejects them as malformed.
- Bundles signed at Mars using old-version registry semantics arrive at Earth after the new registry has cutover; receiving node may reject them as deprecated.
- Bundles in flight at transition time have ambiguous interpretation.

Existing protocol-transition approaches do not address this trilemma.

---

## 3. Summary of the invention

The invention provides a system and method for namespace registry transitions across distributed compute nodes with propagation delays that may exceed the desired cutover time. Distinguishing features include, in combination:

**(a)** Registry version carried in the namespace URN scheme itself (e.g., `vbx-body:v1.0:` vs `vbx-body:v1.1:`), permitting per-bundle disambiguation without external metadata.

**(b)** A dual-version retention window during which receivers accept and parse URNs under both the prior and current registry versions, with window duration computed as a function of physical link characteristics: default window = 2 × max-RTT across the slowest participating link, with a configurable minimum floor.

**(c)** A dedicated `registry-version-transition` bundle class on a priority lane, signed by the registry master key, announcing transition start and end TAI epochs explicitly, so that all receivers know the window bounds before encountering ambiguous bundles.

**(d)** Cryptographic anchoring of the registry version itself in a hash-chained provenance ledger (Aletheia-class), such that the registry-version transition is itself an attested event.

**(e)** Strict version-boundary semantics: after the window closes, prior-version URNs are rejected; before the window opens, new-version URNs are rejected. The window is the only acceptance period for cross-version interpretation.

The combination yields a system that maintains namespace consistency across propagation windows whose duration exceeds the round-trip light-time between participating compute nodes — a circumstance not addressed by prior-art protocol-versioning mechanisms.

---

## 4. Detailed description (enablement disclosure)

### 4.1 URN scheme construction

Namespace URNs include an explicit version field in the scheme portion:

```
<namespace>:v<major>.<minor>:<path>?<query>
```

Example:
```
vbx-body:v1.0:earth?frame=ITRF2020
vbx-body:v1.1:earth?frame=ITRF2020
```

The version field is part of the URN's signed-payload (per the originating signature scheme), so post-issuance tampering of version is detectable.

### 4.2 Retention window duration

For an interplanetary deployment with maximum round-trip time RTT_max across the slowest active link, the dual-version retention window duration is set by default as:

```
W = max(2 × RTT_max, W_minimum)
```

where W_minimum is a mission-policy floor (default: 60 minutes). The factor of 2 accommodates: (1) propagation of the transition announcement bundle, (2) propagation of any cross-version bundle dispatched immediately before the transition, with a margin.

### 4.3 Transition bundle structure

The `registry-version-transition` bundle is dispatched at TAI epoch T_announce, signed by the master registry key, and contains:

- prior_version (string)
- new_version (string)
- transition_start_TAI (epoch when window opens)
- transition_end_TAI (epoch when window closes; transition_end_TAI ≥ transition_start_TAI + W)
- new_registry_hash (cryptographic commitment to the new registry content)
- master_key_signature

The bundle travels the priority lane (above standard bundles, comparable to revocation-bundle priority) so that all receivers process it before any subsequent bundles in standard streams.

### 4.4 Receiver behavior across the window

Before transition_start_TAI: only prior_version URNs accepted; new_version URNs rejected as premature.

During [transition_start_TAI, transition_end_TAI]: both prior_version and new_version URNs accepted and parsed against their respective registry copies. The receiver maintains both registries in memory during the window.

After transition_end_TAI: only new_version URNs accepted; prior_version URNs rejected as deprecated. The prior registry can be retained for audit purposes but is not used for live decisions.

### 4.5 Cryptographic anchoring

The transition event is itself recorded as a hash-chained ledger entry in the originating authority's provenance chain (Aletheia-class). The ledger entry contains:

- new_registry_hash (matching the bundle's hash)
- transition window epochs
- master_key_signature
- prev_ledger_hash (chain linkage)

This makes the transition tamper-evident and forensically auditable. A receiver that later disputes the transition can verify against the ledger entry's signature and chain position.

### 4.6 Operating example

At T_0, ground operations decide to bump registry from v1.0 to v1.1 (e.g., to add Saturn surface frame definitions).

At T_0, ground dispatches `registry-version-transition` bundle:
- prior_version = "v1.0"
- new_version = "v1.1"
- transition_start_TAI = T_0 + 60 min (announcement-to-cutover safety margin)
- transition_end_TAI = T_0 + 60 min + W (window duration W = 2 × 44 min = 88 min for Mars at maximum conjunction)

The bundle propagates to Mars over 3–22 minutes (depending on planetary alignment). Mars PHRONESIS-1 receives it before transition_start_TAI (assuming announcement-to-cutover margin > one-way propagation). Mars loads the v1.1 registry, prepares for dual-version operation.

During [T_0 + 60 min, T_0 + 148 min], both ground and Mars accept v1.0 and v1.1 URNs.

After T_0 + 148 min, only v1.1 URNs accepted. The window has fully closed across all participating nodes.

---

## 5. Claims (DRAFT)

### Independent claim 1 (system)

A system for namespace registry transitions across distributed compute nodes connected by communication channels whose round-trip latency exceeds one second, comprising:

a plurality of compute nodes each maintaining at least one copy of a namespace registry, the namespace registry being versioned with an explicit version identifier;

a namespace identifier scheme wherein the version identifier is carried within each identifier's scheme portion, such that per-identifier version disambiguation is achievable without external metadata;

a dedicated transition-announcement bundle class on a priority lane, the transition-announcement bundle signed by a registry master key and comprising a prior-version identifier, a new-version identifier, a transition-start epoch, a transition-end epoch, a cryptographic commitment to the new registry content, and a signature;

a dual-version retention window of duration W = max(α × RTT_max, W_minimum) where RTT_max is the maximum round-trip time across the slowest active link and α is a configuration constant of at least 2, during which receivers accept and parse identifiers under both prior and new registry versions; and

strict version-boundary acceptance semantics wherein, after the transition-end epoch, prior-version identifiers are rejected, and before the transition-start epoch, new-version identifiers are rejected;

wherein the transition event is recorded as a hash-chained provenance ledger entry signed by the same master key.

### Independent claim 2 (method)

A method for namespace registry transitions across distributed compute nodes connected by communication channels whose round-trip latency exceeds one second, comprising:

versioning namespace identifiers with an explicit version identifier carried within each identifier's scheme portion;

dispatching a transition-announcement bundle signed by a registry master key on a priority lane, the bundle comprising a prior-version identifier, a new-version identifier, a transition-start epoch, a transition-end epoch, and a cryptographic commitment to the new registry content;

at receiving nodes, processing the transition-announcement bundle before any standard bundles arriving subsequently;

at receiving nodes, accepting and parsing identifiers under both prior and new registry versions during the dual-version retention window;

at receiving nodes, rejecting prior-version identifiers after the transition-end epoch and rejecting new-version identifiers before the transition-start epoch; and

recording the transition event as a hash-chained provenance ledger entry signed by the same master key.

### Dependent claims (sketches)

**Claim 3.** The system of claim 1 wherein the configuration constant α is 2 and W_minimum is 60 minutes.

**Claim 4.** The system of claim 1 wherein the at least two compute nodes are located on at least two different celestial bodies.

**Claim 5.** The system of claim 1 wherein the namespace identifier scheme is a Uniform Resource Name (URN) scheme.

**Claim 6.** The system of claim 1 wherein the priority lane is implemented as a dedicated bundle stream of a delay-tolerant networking protocol conformant to RFC 9171.

**Claim 7.** The system of claim 1 wherein the hash-chained provenance ledger uses Ed25519 signatures and SHA-256 hashes.

**Claim 8.** The system of claim 1 wherein RTT_max is dynamically computed from observed link telemetry and is itself updated atomically at registry-transition events.

**Claim 9.** The system of claim 1 wherein the registry master key is held in escrow with multi-party custody and is never deployed to any compute node operating in a remote-body location.

**Claim 10.** The system of claim 1 wherein the cryptographic commitment to the new registry content is a SHA-256 hash of the canonical UTF-8 byte sequence of the new registry.

**Claim 11.** The method of claim 2 further comprising, at the receiving node, retaining a copy of the prior registry for audit purposes after the transition-end epoch.

**Claim 12.** The system of claim 1 wherein at least one of the compute nodes is at an Earth ground station and at least one is on a celestial-body surface or in orbit around a celestial body.

---

## 6. Prior-art positioning

Likely prior-art categories:

| Category | Representative prior art | Distinguishing position |
|---|---|---|
| Protocol version negotiation | HTTP version negotiation, TLS handshake | Sub-second propagation context; no cryptographic transition anchoring; no priority-lane announcement |
| DNS zone transitions | DNS NOTIFY / SOA TTL | Eventual-consistency model; no dual-version-window semantics; no cryptographic master-key anchoring of transitions |
| X.509 / CA certificate transitions | RFC 5280 mechanisms | No interplanetary-RTT-aware window sizing; no priority-lane announcement |
| IPv4/IPv6 dual-stack | RFC 4213 and successors | Long-term coexistence model, not windowed transition; no cryptographic anchoring |
| Bundle Protocol versioning | RFC 9171 vs prior RFC 5050 | Single-version-at-a-time deployment in practice; no specified dual-version retention semantics for interplanetary transitions |
| **KeySpace (Smailes et al. 2024, arXiv 2408.10963)** | Interplanetary PKI revocation and OCSP-Hybrid proposal | Addresses revocation and certificate validity in interplanetary PKI; **does NOT address namespace registry version transitions**; this is adjacent prior art but does not anticipate claim 1's combination |

The novelty position is the **system combination of (a)–(e) in §3** operating in the **interplanetary propagation context with explicit physics-bounded window sizing**. KeySpace 2024 is the closest adjacent art and should be cited in the practitioner search.

---

## 7. § 101 risk and mitigation

Patent claims framed as pure namespace management face Alice/Mayo § 101 rejection. Mitigation:

- Tie claims to concrete technological improvements: cryptographic anchoring, priority-lane signaling, physics-bounded window sizing tied to link telemetry.
- Frame as solution to specific technical problem: the cross-version asymmetry failure mode in interplanetary registries.
- Include concrete reductions to practice: specific URN scheme structure, specific α=2 and W_minimum=60min defaults, specific Ed25519 + SHA-256.

---

## 8. Filing strategy notes

Same as CADUCEUS-IP-001 §9. **File CADUCEUS-IP-002 in the same filing window as IP-001 and IP-003** to establish coordinated priority. Sequencing relative to arXiv preprint: PPA first.

---

## 9. Honest scope statement

- Confidence on novelty: MEDIUM-HIGH. The interplanetary-specific window sizing tied to RTT_max and the cryptographic transition anchoring are the most novel elements. The general concept of versioned URN schemes is not novel; the combination with the other elements may be.
- Confidence on enablement: HIGH for the core mechanism; gaps possible in detailed implementation of priority-lane scheduling.
- Confidence on § 101 survival: MEDIUM.
- Practitioner search is the gating step. KeySpace 2024 must be examined specifically; if KeySpace addresses registry transitions (rather than only certificate revocation), the claim scope may need significant tightening.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-IP-002 Rev A · PPA drafting target — Attack 27 namespace-registry-transition*
*Status: practitioner engagement required before filing*
