# Caduceus v0.1 — Interplanetary Connectivity Substrate

**Compliance-graded, drift-aware, cryptographically-signed bundle fabric for PHRONESIS-1 (VBX-ISPS) and EPHEMERIS-1 (VBX-01)**

| | |
|---|---|
| **Document ID** | CADUCEUS-001 |
| **Revision** | A (draft) |
| **Methodology** | ZCS-6 (Phase 1 + Phase 2 pre-commit) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Pre-commit draft. Cryptographic commit (LEDGER #0004) is a deliberate follow-up step, not automatic. |
| **Cross-references** | VBX-ISPS (substrate v0.2-α, PHRONESIS-M); VBX-01 EPHEMERIS (firmware v0.2, planet-time-bench v1.2); Aletheia DAC; Civium; Mercury; MVCI |

---

## 0. Scope statement (honest)

This document is a **specification and benchmark pre-commit**, not an implementation. No code. No hardware. No mass/power numbers from a real engineering partner — only first-principles estimates and locked-in budget caps, both flagged.

This document treats "interplanetary internet" as shorthand for **delay-tolerant store-and-forward bundle fabric**. TCP/IP is not interplanetary-useful (lightspeed-bounded round-trips of 6–44 minutes for Mars; up to ~10 hours for outer-system).

This document treats "teleportation" in **only one** physical sense — **quantum-state teleportation for quantum key distribution (QKD)**. Matter teleportation (Star-Trek class) violates known physics and is out of scope. FTL information transfer via entanglement violates the no-communication theorem and is out of scope. Quantum-state teleportation has been [VERIFIED] in ground-to-satellite experiments (Micius, Yin et al. 2017, Nature) at 1200 km; interplanetary-distance QKD is [SPECULATIVE] and deferred to v3.x horizon.

---

## 1. Architectural framing

### 1.1 Why "Caduceus"

The caduceus is the herald's staff carried by Hermes/Mercury — the *instrument* of the messenger, not the messenger himself. This name preserves the existing portfolio convention (Mercury already named for the Subleq kernel; Caduceus is the *staff*, a distinct object) and matches the function: a structured carrier for signed, authority-gated, drift-monitored bundles moving across interplanetary distances. The two snakes wound around the staff map naturally to the **Aletheia trust strand** (provenance) and **Civium authority strand** (governance) intertwined around the bundle fabric.

### 1.2 Three orienting calls

1. **EPHEMERIS does not terminate the interplanetary link.** Wrist-class antenna gain, MCU compute, and small-cell power budgets make termination physically untenable. EPHEMERIS pairs short-range (UWB or BLE) to PHRONESIS and serves as an **independent crew-attestation peer** — it verifies signed bundles received by the suit and displays them, providing a second-path human-in-loop trust node that does not depend on the suit's main HUD.

2. **PHRONESIS terminates the link.** The Jetson Orin NX-class compute already in PHRONESIS CORE has the headroom to host the bundle-fabric stack alongside Civium / MVCI / Aletheia / Mercury. The existing HF-11 interop port + comm (S-band + UHF per VBX-ISPS-001) is the v0.1 physical layer. Higher-rate modes (X/Ka, optical) are v0.2+, subject to the §6.4 budget cap.

3. **The whitespace is above the radio.** NASA, JPL, ESA, Lockheed, Astrobotic, SpaceX, and the CCSDS community own the radio and the data-link layers. The defensible build is the **compliance-graded bundle fabric** that sits on top.

### 1.3 The six-attribute whitespace map

| # | Attribute | Existing public substrate | Gap that Caduceus closes |
|---|---|---|---|
| 1 | Per-bundle cryptographic provenance | CCSDS SDLS (symmetric pre-shared keys) | Per-event Ed25519 signing + hash-chained Aletheia ledger |
| 2 | Authority gating on inbound commands | CCSDS SDLS-EP (auth + integrity, role-agnostic) | Civium graph-based authority decision per bundle, per safety class, per body, per frame |
| 3 | Drift-aware confidence on link characteristics | None (link is treated as nominal until it fails) | ACI conformal bands on BER, SNR, RTT; early-warning before hard fail |
| 4 | Frame-disambiguated timestamps | CCSDS Time Code Format (TAI-rooted, frame-agnostic at the bundle layer) | (TAI, vbx-body-URN) tuple on every bundle; EPHEMERIS provides the 8-body civil-time disambiguator |
| 5 | WCET-bounded safety-critical receive path | Operating-system best-effort (Linux + ROS-class) | Mercury-kernel deterministic upper bound from receive to gate-decision |
| 6 | Independent crew-attestation peer | Single-point-of-trust on suit avionics | EPHEMERIS verifies in parallel; signature mismatch displays alert independently of suit HUD |

No incumbent combines these six. The combination is the IP target.

---

## 2. ZCS-6 Phase 1 — Whitespace identified

[VERIFIED] DTN / Bundle Protocol v7 (RFC 9171, 2022) exists and is in operational use on the ISS. CCSDS protocols are mature. Mars Relay Network is operational. DSOC was demonstrated in 2024 at ~460 Mbps from ~32M km (Psyche mission). LunaNet is under multi-agency development. Quantum-state teleportation has been demonstrated at 1200 km satellite-to-ground (Yin et al. 2017).

[VERIFIED gap] No public substrate provides per-bundle Aletheia-style cryptographic provenance + Civium-style compliance gating + ACI-style drift monitoring on link metrics + frame-disambiguated timestamps + Mercury-bounded WCET + a separate crew-attestation peer. CCSDS SDLS is symmetric-key. NASA's bundle protocols address custody and integrity, not authority graphs.

[PLAUSIBLE strategic position] The combined fabric is a defensible compliance-graded category that maps to the Civium thesis (HIPAA/FISMA for the regulated edge; the same compliance-graded posture extended to off-planet operations). Federal-capture pathways include NASA SBIR/STTR, AFRL OTAs via SOSSEC, and Space Development Agency (SDA) tranches.

---

## 3. ZCS-6 Phase 2 — caduceus-bench v1.0 (draft, pre-commit)

Twelve assertions across four layers. Pre-commit cryptographic discipline: SHA-256 manifest hash, Ed25519 signature with Visionblox release key, Aletheia ledger entry, OpenTimestamps proof. **No silent edits — backward edges into Phase 2 are versioned re-commits with delta justification records.**

### 3.1 Physical layer (P-class)

| ID | Assertion | Threshold | Source |
|---|---|---|---|
| **P1** | RF link-budget margin on S-band uplink at Earth–Mars maximum-conjunction range | ≥ 3 dB above Shannon-rate floor for the declared bitrate | First-principles link budget; CCSDS 401.0-B baseline |
| **P2** | Optical pointing accuracy when optical mode is engaged | ≤ 1 µrad RMS over 60 s window | [PLAUSIBLE] DSOC 2024 demonstrated ≤ 1.5 µrad; tighten for safety margin |
| **P3** | Multi-mode reconfiguration time (S-band → UHF fallback, optical → S-band fallback) | ≤ 5 s from loss detection to reacquired bundle delivery | First-principles, hardware partner dependent |

### 3.2 Bundle layer (B-class)

| ID | Assertion | Threshold | Source |
|---|---|---|---|
| **B1** | BP7 (RFC 9171) wire-format conformance — round-trip a 1 KB and 1 MB bundle through reference implementation (ION-DTN or μPCN) | 100% bit-exact reproduction; all required EID, lifetime, payload, and CRC fields parsed | RFC 9171 |
| **B2** | Store-and-forward integrity through ≥ 3 hops with simulated 60-min connectivity outage between hops 2 and 3 | 100% custody-transferred bundles delivered; 0% silent loss | RFC 9171 §4 custody transfer |
| **B3** | Custody-transfer semantics under intermittent loss — bundle TTL expiry triggers explicit failure notification to source, not silent drop | 100% of expired bundles produce custody-failure status report | RFC 9171 §5 |

### 3.3 Trust layer (T-class) — the IP heart

| ID | Assertion | Threshold | Source |
|---|---|---|---|
| **T1** | Every outbound bundle carries an Ed25519 signature over (bundle_id ∥ payload_hash ∥ timestamp_tuple ∥ prev_ledger_hash) | 100% outbound signed; signature verifies against published Visionblox release public key | Aletheia DAC v0.2 |
| **T2** | Every inbound bundle is verified and passed through the Civium authority gate before any action is taken | 100% gate decisions logged; 0% pre-gate side effects | Civium L1–L7 governance stack |
| **T3** | Signature failure OR gate denial deterministically falls back to HF-9 (hard safety floor) — no LLM-mediated retry | 100% fail-safe transitions logged on Aletheia chain; ≤ Mercury-WCET bound from failure detection to floor entry | Inherited from VBX-ISPS HF-9 |

### 3.4 Frame layer (F-class) — inherits EPHEMERIS work

| ID | Assertion | Threshold | Source |
|---|---|---|---|
| **F1** | Every bundle carries a (TAI_microseconds, vbx-body-URN) tuple in a BP7 extension block, where the URN conforms to **Caduceus-BodyID v1.0** (see §3.5) | 100% bundles carry tuple; URN parses without ambiguity against the locked v1.0 registry | planet-time-bench v1.2 + §3.5 |
| **F2** | Cross-body command bundles (e.g., Earth-originated command targeting Mars-surface action) are rejected unless the frame transformation is in the authorized set | 100% unauthorized cross-frame rejections logged | EPHEMERIS A1a–A1e |
| **F3** | EPHEMERIS independently verifies the URN field on its display ring; mismatch with PHRONESIS-displayed URN triggers alert | ≤ 100 ms mismatch detection latency over UWB peer link | New floor; A3 inherited |

### 3.5 Caduceus-BodyID v1.0 — Visionblox-Extended Body-ID Namespace (LOCKED at v1.0 commit)

**Locked scope:** the namespace defined here is a *normative wire-format component* of caduceus-bench v1.0. Adding bodies, frames, or path classes requires a v1.0 → v1.1 namespace bump executed as an Aletheia-ledger versioned re-commit with a delta justification record. **Silent additions are a benchmark violation.**

**URN scheme:**

```
vbx-body:v1.0:<path>?frame=<frame_id>[&epoch=<rfc3339-utc>]
```

**Path grammar (EBNF):**

```
path           = level1 [ "/" level2 [ "/" level3 ] ]
level1         = body | "lagrange" | "artificial"
body           = "sun" | "mercury" | "venus" | "earth" | "moon" | "mars"
               | "jupiter" | "saturn" | "uranus" | "neptune"
               | "pluto" | "ceres"
               | named-major-moon | named-asteroid | named-comet
level2         = canonical-region | orbit-class | lagrange-pair | artificial-class
level3         = canonical-site | sub-orbit | lagrange-point | named-artifact
orbit-class    = "leo" | "meo" | "geo" | "heo"
lagrange-pair  = body "-" body                  ; e.g., "sun-earth", "earth-moon"
lagrange-point = "L1" | "L2" | "L3" | "L4" | "L5"
artificial-class = "habitat" | "station" | "relay" | "vehicle"
named-artifact = "iss" | "gateway" | visionblox-registered-name
```

**Initial frame registry (locked at v1.0):**

| Frame ID | Definition | Applies to |
|---|---|---|
| `IAU2015` | IAU 2015 standard body-fixed frame | Most major bodies with defined rotation |
| `J2000` | Earth-centered inertial, J2000.0 epoch | Earth orbital ops |
| `ITRF2020` | International Terrestrial Reference Frame 2020 | Earth surface ops |
| `MoonME` | Mean Earth / polar axis | Moon surface ops |
| `MoonPA` | Principal Axis | Moon precision surface ops |
| `MarsIAU2015` | IAU 2015 Mars body-fixed | Mars general ops |
| `MarsMOLA` | MOLA-derived areoid | Mars precision surface ops |
| `SE-L1-rotating` | Sun-Earth L1 rotating frame | Sun-Earth Lagrange ops |
| `SE-L2-rotating` | Sun-Earth L2 rotating frame | Sun-Earth Lagrange ops |
| `EM-L1-rotating` | Earth-Moon L1 rotating frame | Cislunar ops |
| `EM-L2-rotating` | Earth-Moon L2 rotating frame | NRHO / Gateway ops |
| `NRHO-9:2:1` | Near-Rectilinear Halo Orbit, 9:2 synodic resonance | Gateway-class orbits |
| `UNDEFINED` | Explicit acknowledgment of no defined frame | Saturn rotation (per planet-time-bench A1c); other inherent ambiguities |

**Canonical examples (these are the wire-format strings):**

```
vbx-body:v1.0:earth?frame=ITRF2020
vbx-body:v1.0:earth/leo/iss?frame=J2000
vbx-body:v1.0:moon?frame=MoonME
vbx-body:v1.0:moon/farside/shackleton?frame=MoonPA
vbx-body:v1.0:mars?frame=MarsIAU2015
vbx-body:v1.0:mars/jezero?frame=MarsMOLA
vbx-body:v1.0:lagrange/sun-earth/L1?frame=SE-L1-rotating
vbx-body:v1.0:lagrange/earth-moon/L2?frame=EM-L2-rotating
vbx-body:v1.0:saturn?frame=UNDEFINED
vbx-body:v1.0:artificial/station/gateway?frame=NRHO-9:2:1&epoch=2026-06-08T00:00:00Z
```

**Wire-format binding (F1 normative):**

- Carried in a BP7 extension block (block-type-code: TBD — request CCSDS reserved-range allocation in Phase 5).
- Block content: UTF-8 encoded URN string.
- Length prefix: u16 little-endian byte count, followed by UTF-8 payload.
- Maximum URN length: **256 bytes** (supports ~200 chars of path + frame + epoch, leaving headroom for namespace v1.x extensions within the same byte ceiling).
- Untupled bundles (missing extension block) are F1 violations and MUST be rejected, not parsed with default-frame inference.

**Why "Visionblox-extended" and not "IAU-only":** IAU defines body-fixed frames for major bodies with measured rotation. IAU does not define namespaced identifiers for Lagrange points, artificial structures (ISS, Gateway), specific sites (Jezero, Shackleton), specific orbit classes (NRHO 9:2), or explicit-undefined frames (Saturn rotation). All of these are operationally required for crewed interplanetary ops. The Visionblox-extended namespace is a strict superset of IAU body identifiers with hierarchical, URN-style, versioned, frame-explicit semantics.

**Registry maintenance:**

- Visionblox maintains the canonical registry as a versioned file in the substrate repo.
- v1.0 → v1.1 namespace bumps require Aletheia-ledger versioned re-commit (delta justification + signed manifest hash + OpenTimestamps proof) — same discipline as benchmark revisions.
- Third-party adds (e.g., partner-agency body or site additions) are accepted only via the v1.x bump process, not via PHRONESIS-side runtime extension.

---

## 4. ZCS-6 Phase 3 — Adversarial sweep (pre-commit)

Attacks to survive before LEDGER #0004 is issued:

1. **Replay across long RTT** — old signed bundles re-injected after key rotation. Defense: bundle TTL + nonce in signed payload + Aletheia prev_hash linkage.
2. **Frame confusion** — UTC-timestamped bundle delivered into Mars-MTC operational context. Defense: F1 + F2 + F3 combined; URN-mediated.
3. **Bundle fragmentation under DTN custody transfer** — partial signature verification bypass. Defense: signature covers reassembled bundle; partial-fragment signatures are not valid command authorization.
4. **Trust-anchor compromise at relay** (e.g., Mars orbiter relay held by adversary). Defense: end-to-end Aletheia signature, not hop-by-hop; relay can drop but cannot forge.
5. **ACI thrashing** — adversarial signal noise designed to push drift monitor into low-confidence band and force fail-safe DoS. Defense: tunable alpha update (per VBX-ISPS bug fix #2 lesson); ACI does not directly gate authority — it raises a CORE-monitored alert.
6. **EPHEMERIS-PHRONESIS pairing MitM over UWB.** Defense: pairing protocol must require physical-proximity attestation (NFC pairing seed) + Ed25519 mutual auth + bundle re-signing by PHRONESIS for EPHEMERIS display (no relay of remote signatures).
7. **A1d seam attack** — craft an out-of-scope time bundle designed to slip through the v1.2 OutOfScopeError path. Defense: F1 rejects un-tupled bundles; A1d signaling preserved.
8. **Namespace-exhaustion attack** — adversarially constructed URNs designed to exceed the 256-byte ceiling, trigger parser pathologies, or force inference of unregistered bodies/frames. Defense: strict registry check (registered vs not), hard 256-byte cap with rejection on overflow, no fallback parsing.

If any attack passes, the assertion set goes to v1.1 with a versioned delta record.

---

## 5. ZCS-6 Phase 4 — Deterministic defense (commit plan)

Steps to execute *after* the Phase 3 sweep returns clean:

1. Freeze caduceus-bench v1.0 markdown (including §3.5 namespace registry).
2. SHA-256 the manifest.
3. Ed25519 sign with the Visionblox release key.
4. Append to Aletheia chain as **LEDGER #0004** (sequential after VBX-ISPS LEDGER #0003).
5. OpenTimestamps proof on the ledger entry.
6. Publish manifest hash + signature publicly (defensive arXiv preprint or SSRN companion) before any v0.1 substrate code is written.

This is the ordering that makes the benchmark uncheatable.

---

## 6. ZCS-6 Phase 5 — Substrate sketch (for v0.2 build)

### 6.1 PHRONESIS-side modules

Extending the existing HF floor map (HF-1 through HF-15) with five new floors:

| HF | Function | Backing module | Notes |
|---|---|---|---|
| **HF-16** | Outbound bundle attestation | Aletheia + Civium | Ed25519 sign at suit boundary before TX |
| **HF-17** | Inbound verification + Civium authority gate | Civium + MVCI | Fail-safe to HF-9 on signature failure OR gate denial |
| **HF-18** | ACI drift monitor on link characteristics | Aletheia | BER, SNR, RTT conformal bands; raises CORE alert on excursion |
| **HF-19** | Frame disambiguation on bundle headers | EPHEMERIS interop (HF-11 extension) | Inherits A1a–A1e; URN-parsed per §3.5 |
| **HF-20** | Mercury-bounded WCET on safety-critical receive path | Mercury kernel | Deterministic upper bound, recv → gate-decision |

### 6.2 EPHEMERIS-side peer (HF-11 extension)

- UWB or BLE pairing to PHRONESIS, NFC pairing seed required for trust establishment.
- Receives a peer-bundle copy from PHRONESIS containing (bundle_id, summary, signature, vbx-body-URN).
- Independently verifies the signature against the published Visionblox release public key (cached locally; re-fetched on key rotation events signed by the same key).
- Independently parses the URN against a cached copy of the v1.0 namespace registry.
- Independently displays the body + frame on the watch face ring.
- Alert is local-haptic + LED on mismatch with PHRONESIS-displayed URN, OR on signature failure, OR on URN that does not parse against the cached registry.
- Does **not** issue commands. Read-and-attest only.

### 6.3 Reference open-source dependencies

[VERIFIED] candidate starting points:
- **ION-DTN** (NASA, open source) — reference BP7 implementation.
- **μPCN** (TU Berlin, open source) — embedded BP7, may fit EPHEMERIS-class.
- **CCSDS SDLS** wire format — for interop with NASA ground stations.
- **OpenSSL / libsodium** — Ed25519 + SHA-256.
- **Aletheia DAC v0.2** — existing Visionblox/Zuup substrate.

### 6.4 Caduceus Mass and Power Budget Cap (LOCKED at v1.0 commit, pre-partner-engagement)

**Constraint context (current PHRONESIS spec):**

- Total suit mass: 110 kg (already aggressively lean vs xEMU class)
- PLSS mass: 40 kg
- Existing HF-11 (S-band + UHF transceiver) estimated at [PLAUSIBLE] 2–3 kg of the PLSS allocation
- Battery: 1200 Wh, 200 W peak / 120 W steady envelope
- EVA contingency: 10 h, requires ~75% reserve at 120 W steady-state

**Hard caps (LOCKED — exit criteria for partner negotiation, not opening positions):**

| Budget | Cap | Includes |
|---|---|---|
| **Mass (total comm subsystem)** | **4.0 kg** | Existing HF-11 + any v0.2 hardware additions; antenna, transceiver, optical terminal if present, RF front-end, dedicated baseband |
| **Power (steady-state)** | **50 W** | Receive + idle TX + control loop; compute load on Orin NX excluded (counted against suit compute budget) |
| **Power (peak TX)** | **100 W** | Includes RF amplifier; must NOT temporally overlap with compute peak (concurrent compute + comm peak combined ≤ 200 W system peak) |

**Implications for the v0.1 → v0.2 → v0.3 ladder:**

| Track | Mass profile | Power profile | Cap status |
|---|---|---|---|
| **v0.1 software-only on existing HF-11** | ~0 kg delta | ~10 W compute delta on Orin NX | PASSES — well within caps |
| **v0.2 + X-band transceiver** | ~1–2 kg delta within 4 kg envelope | ~20–40 W TX within 100 W peak | PASSES (subject to partner spec) |
| **v0.2 stretch + optical terminal** | 3–8 kg naive | 50–100 W TX naive | FORCES partner integration/compression — naive build exceeds caps |
| **v3.x quantum-state-teleportation hardware** | Currently spacecraft-bus class | Currently spacecraft-bus class | OUT OF SCOPE for v0.1–v0.2 |

**Justification for the caps:**

- Suit substrate-consolidation gain (~2 kg from compute consolidation per VBX-ISPS v0.2-α notes) provides the headroom that funds Caduceus mass.
- 4 kg keeps PLSS at ≤ 42 kg, still under xEMU class (~55 kg).
- 50 W steady leaves ~70 W headroom on the 120 W steady envelope for thermal control + O2/CO2 loop + compute.
- 100 W peak coexists with the 200 W system peak only if comm and compute peaks are non-concurrent — this becomes a scheduling constraint on HF-20 (Mercury-bounded WCET path must include the comm-peak avoidance interlock).
- Caps are **exit criteria for partner negotiation**, not opening positions. Partner solutions exceeding caps are rejected, not renegotiated. This is the substrate-discipline lesson from the VBX-ISPS Phase-3 work: do not soften floors.

**Operational consequence:** the optical-terminal track is now a forcing function for an *integrated/compressed partner solution*, not a vendor catalog pick. Likely partners must demonstrate ≤ 4 kg total comm subsystem with optical capability — Mynaric CONDOR-class, BridgeComm OGS-class, or Astrogate Labs would each need to confirm before engagement.

---

## 7. ZCS-6 Phase 6 — Anticipated vertical-integration attacks (queued)

To run *after* v0.1 substrate passes v1.0 (mirrors VBX-ISPS Phase 6 against v0.1):

- Combined replay + frame-confusion + ACI-thrash chained attack.
- Long-duration mission drift simulation: 90-day connectivity outage, evaluate key rotation policy and bundle TTL behavior.
- EPHEMERIS battery-fail scenario: does PHRONESIS still operate cleanly without the peer?
- Hardware-isolation boundary attacks: does Bus ISO / Secure EL cover the comm path adequately, or is there a side channel?
- Concurrent-peak collision: HF-17 receive-path peak coincides with HF-20 compute peak — does the comm-peak avoidance interlock hold under adversarial scheduling?
- Quantum-secured Aletheia (v3.x horizon): can the existing chain be extended with QKD-derived session keys without breaking Phase 4 commit semantics?

---

## 8. Cross-references to existing Visionblox/Zuup work

| Component | Source document | Relationship |
|---|---|---|
| Aletheia DAC v0.2 | VBX-ISPS v0.1 substrate | Provides the per-bundle signing + chain primitive |
| Civium L1–L7 governance | zuup-platform-specs | Provides the authority graph for T2 |
| MVCI gate framework | Existing federal-capture stack | Provides the approval-gate pattern for HF-17 |
| Mercury kernel | Mercury Subleq RTL simulator (SSRN-published) | Provides WCET-bounded execution for HF-20 |
| EPHEMERIS planet-time-bench v1.2 | VBX-01 Ephemeris v0.2 | Provides A1a–A1e frame disambiguation reused by F1–F3 and §3.5 namespace |
| MVCI 12-agent stack | Existing federal-capture stack | Candidate for ground-side bundle origination + authority graph maintenance |
| VBX-ISPS HF-9 | VBX-ISPS-001 blueprint | Hard safety floor that T3 falls back to |

---

## 9. Open items for v0.2 (and Phase 6 deferral list)

1. **Quantum-Secured Aletheia Chain** — [SPECULATIVE] research item; pair with Micius-class QKD hardware partner; v3.x horizon.
2. **Optical terminal partner engagement** — Mynaric / BridgeComm / Astrogate Labs / equivalent. Engagement MUST honor the §6.4 cap (4 kg, 50 W steady, 100 W peak) as exit criterion.
3. **Formal verification of HF-17 + HF-20** — Lean/Coq proof of authority-gate logic and Mercury-bounded receive path. Pair with the existing Lean/Coq track on the Mercury kernel.
4. **CCSDS SDLS interop layer** — needed for NASA ground-station ingest. Wrapper at the data-link layer; Aletheia signature carried in the bundle payload above SDLS.
5. **CCSDS reserved-range allocation request** for the BP7 extension block-type-code carrying the vbx-body-URN (§3.5 wire-format binding).
6. **Defensive arXiv preprint** — published alongside LEDGER #0004 commit, before substrate code, to anchor prior art. Should publish the Caduceus-BodyID v1.0 namespace as a separate artifact for community uptake.
7. **Robot AIBOM extension** — extend the existing AIBOM Generator (OMB M-25-22) to cover comm-fabric components.
8. **SOSSEC / NASA SBIR / SDA channel mapping** — federal-capture posture for Caduceus specifically.
9. **EPHEMERIS Key Rotation UX** — how does a crew member see and confirm a release-key rotation event? Must not require ground link.
10. **Comm-peak / compute-peak scheduling interlock** — Mercury kernel must enforce non-concurrence of HF-17 receive peak and Orin NX compute peak. Spec the interlock in v0.2.
11. **Caduceus-BodyID v1.0 community RFC** — eventual goal is third-party (NASA, ESA, JAXA) adoption; v1.0 commit anchors prior art, but a parallel RFC track should be initiated post-commit.

---

## 10. Honest scope statement (recap)

- This document is Phase 1 + Phase 2 of one ZCS-6 cycle. Phase 4 commit and Phase 5 build are explicit follow-up steps.
- No real engineering partner has reviewed the §6.4 mass/power caps. Caps are first-principles, locked as exit criteria.
- All thresholds in §3 are first-principles starting points subject to v1.1 revision via versioned re-commit.
- Caduceus-BodyID v1.0 (§3.5) is normatively locked. Adding bodies or frames requires v1.1 namespace bump with delta justification.
- "Teleportation" in §0 is reframed; matter teleportation remains out of scope.
- "Interplanetary internet" is shorthand for DTN/BP7; external materials should never use the unqualified word *internet*.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-001 Rev A · ZCS-6 Phase 1+2 pre-commit*
