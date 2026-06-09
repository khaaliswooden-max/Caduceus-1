# Caduceus — Federal Capture Posture

| | |
|---|---|
| **Document ID** | CADUCEUS-CAPTURE-001 |
| **Revision** | A (draft) |
| **Purpose** | Federal-capture strategy for Caduceus across NASA SBIR, SOSSEC OTA, SDA, Navy SBIR, and adjacent channels |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Planning document. Channel selection and proposal triggers depend on LEDGER #0004 commit + PPA filings completing per CADUCEUS-PRACTITIONER-001 §5. |

---

## 0. Honest scope statement

This document analyzes federal-capture channels for Caduceus and recommends prioritized paths. It is **planning material**, not a submitted proposal. Federal procurement is dynamic; specific solicitation deadlines, dollar caps, and topic structures change frequently. All channel-specific intel in §3 below was sourced via web search on 2026-06-08 and must be re-verified against the actual solicitation document before action.

Visionblox is a small business with limited past performance in federal contracting. This document acknowledges that posture honestly and recommends channels appropriate to early-stage capture.

---

## 1. Executive summary

### 1.1 Capability differentiators (the elevator pitch)

Caduceus is a **compliance-graded, drift-aware, cryptographically-signed delay-tolerant bundle fabric** for interplanetary connectivity, built to a cryptographically-committed benchmark (caduceus-bench v1.2.1) via the ZCS-6 falsification-first methodology. Five differentiators for federal capture:

1. **Interplanetary-RTT-aware compliance-graded authority** — handles round-trip propagation delays of minutes to hours; competitors' PKI assumes terrestrial sub-second.
2. **Cryptographically-anchored benchmark** — LEDGER #0004 (Aletheia chain) commits the spec before any implementation; benchmark-gaming impossible.
3. **OMB M-25-22 AIBOM-compatible from day one** — Aletheia provenance ledger is the canonical AIBOM substrate.
4. **Open-source friendly substrate** — ION-DTN (NASA), libsodium, RFC 9171 + 9172 BPSec — no proprietary lock-in.
5. **Single-fault-tolerant v0.2 build, dual-fault-tolerance roadmap to v0.3** — crewed-mission-ready path; honest about gates.

### 1.2 Recommended channel priority (next 24 months)

| Priority | Channel | Rationale | Earliest action |
|---|---|---|---|
| **1** | **NSTXL / SOSSEC consortium membership** | Cheapest entry; broadest OTA access; SOSSEC's focus on mission-secure networking and cyber defense aligns directly | $500–$1,500 fee; within 30 days |
| **2** | **NASA SBIR FY27 Phase I — SCaN Cognitive Communication subtopic** | $150K Phase I; direct SCaN topic match; Aletheia chain aligns with subtopic's "blockchain-based data processing" interest. Anticipated solicitation window: March–May 2027 | Position now; submit on solicitation release |
| **3** | **DIU Commercial Solutions Openings (CSO)** — relevant comms / cyber topics | No membership fee; quarterly opportunities; rapid contracting | Monitor monthly; respond opportunistically |
| **4** | **Navy SBIR DON26BZ03-NV062 "Secure Tasking of Commercial Assets"** | $315K; July 22, 2026 deadline; encryption + secure comms + 90% tasking-time reduction | **6 weeks out — proposal must start now if pursued** |
| **5** | **SDA Tranche 3 Ground Entry Point (GEP) — sub-teaming or IP licensing** | Tranche 3 launches FY2029; ground infrastructure RFP imminent; SDA's NEBULA software is direct adjacent territory | Monitor SDA opportunities; pursue via prime teaming |
| **6** | **AFRL OTAFI ($499M, 5-year) via SOSSEC** | C4ISR prototyping; mission-secure networking direct match; available once SOSSEC membership active | After Priority 1 lands |
| **7** | **AFWERX SBIR Open Topic** | Phase I pitch days; fastest first-award path if NASA / Navy slips | Quarterly review |
| **8** | **NASA SBIR Phase II Direct** — if Phase I from Priority 2 lands | $850K+ continuation | After Priority 2 |

### 1.3 Critical sequencing constraint

**No federal channel action should precede LEDGER #0004 commit and PPA filings.** Proposals built on uncommitted specs carry priority-establishment risk; pre-PPA disclosure to government technical evaluators creates a public-disclosure event that triggers the foreign-filing clock.

After LEDGER commit + PPA filings + arXiv preprint (in that sequence per CADUCEUS-PRACTITIONER-001 §5), federal channel action is unblocked.

---

## 2. Capability statement positioning

### 2.1 Caduceus capabilities map to government priorities

| Government priority | Caduceus alignment | Evidence |
|---|---|---|
| **Mission-secure networking** | Bundle-fabric with per-bundle authority gating; T1+T7+§3.6 architecture | caduceus-bench v1.2.1 §§3.3, 3.6 |
| **Zero-trust implementation** | No implicit trust; every bundle signed + gated + auditable | T1, T2, T3 |
| **AI-enabled mission management** | Civium graph-authority decisions; ACI drift monitoring | T2, T5 |
| **OMB M-25-22 AIBOM compliance** | Aletheia hash-chained provenance ledger is the AIBOM substrate | All bench-committed artifacts are LEDGER entries |
| **C4ISR modernization** | Interplanetary extension of compliance-graded C4ISR posture | Whole Caduceus architecture |
| **Optical communications** | v0.3 optical-terminal track with partner integration under §6.4 caps | CADUCEUS-005 §8 |
| **Cognitive / autonomy networks** | Mercury-bounded WCET + Civium gate + ACI drift = compliance-graded autonomy | Whole architecture; see also VBX-ISPS |
| **Lunar / Mars / Gateway operations** | Caduceus-BodyID v1.0 namespace explicitly supports Gateway, Mars, Lagrange, etc. | §3.5 namespace |
| **CMMC L2 readiness** | Self-assessment in motion per project context; substrate posture aligns | Visionblox compliance roadmap |
| **HIPAA / FISMA edge inference** | Civium compliance-graded substrate; same posture extended to space | Existing Civium track |

### 2.2 What Caduceus is NOT (honest positioning)

- Not a hardware product (no radio, no antenna, no satellite)
- Not a replacement for CCSDS or DTN standards (sits on top as compliance layer)
- Not a classified-level offering at v0.2 (UNCLASSIFIED // FOUO scope); v0.3+ classified-extension is roadmap
- Not crewed-deployment-ready at v0.2 (single-fault-tolerant only; DFT is v0.3)
- Not a substitute for KeySpace 2024's interplanetary PKI work — adjacent, complementary, not anticipating

### 2.3 Past performance (honest disclosure)

Visionblox LLC is a new federal-contracting entrant with no prior CPARS history. Honest mitigations:

- Project leadership has demonstrated relevant artifacts: VBX-ISPS Python substrate v0.1 (31/31 tests), Mercury Subleq RTL simulator (SSRN published), ACI drift monitoring, Aletheia DAC, AIBOM Generator (in progress per M-25-22).
- Five intelligence watchers operational on GitHub Actions (live data feeds; demonstrate production-quality automation discipline).
- ZCS-6 methodology documented across multiple manifests with cryptographically-anchored commit history.
- Open-source publications (SSRN Mercury; planned arXiv Caduceus + RCA).

For federal proposal narratives, frame past performance as "**emerging firm with production-quality methodology and substantial public artifact base**" — not as "established federal performer."

---

## 3. Channel-by-channel posture

### 3.1 NSTXL / SOSSEC consortium membership

**Channel summary.** SOSSEC manages OTAs covering "C4ISR information sharing information systems," cybersecurity environments, and adjacent capabilities. SOSSEC focuses on "cyber defense, mission-secure networking, C4ISR modernization, secure communications, cloud-to-edge architectures, zero-trust implementation, AI-enabled mission" — direct Caduceus alignment.

**Membership cost.** NSTXL membership runs $500-$1,500 for access to S2MARTS, SOSSEC, Prototype Project Opportunities across multiple services. Highest membership ROI for the fee.

**Action.** Within 30 days: complete NSTXL or direct-SOSSEC membership application. Access unlocks 8 active SOSSEC OTAs plus other prototype project opportunities.

**Caduceus pitch in SOSSEC context.** "Compliance-graded mission-secure networking with cryptographically-committed benchmark; small business non-traditional performer; open-source substrate; CMMC L2 in motion."

### 3.2 NASA SBIR Phase I — SCaN Cognitive Communication subtopic

**Channel summary.** NASA SCaN solicits "Optical communications, RF communications, reprogrammable communications systems and flight dynamics. Emphasis is placed on size, weight and power improvements". SCaN Cognitive Communication subtopic specifically notes that "STMD recently awarded two Early Career Faculty grants to study topics related to Cognitive Communication including distributed network routing and blockchain-based data processing" — direct alignment with Aletheia hash-chained provenance.

**FY26 status.** NASA SBIR FY26 Phase I deadline was May 21, 2026, under BAA 80NSSC26R0003. **Window closed.** Target FY27 cycle.

**FY27 timing estimate.** "The next open window is expected between April and May 2026" — that source referred to FY26; FY27 is expected to follow the same March–May annual pattern.

**Phase I award terms.** "NASA's Phase I awards cap at $150,000 — lower than most participating agencies. The six-month period of performance is tight, and reviewers expect a clearly scoped feasibility demonstration rather than an ambitious research program."

**Caduceus Phase I scope proposal.** Build the v0.2 substrate's authority-graph + revocation-stream + stale-defaults table prototype in 6 months, demonstrate against a NASA-relevant link configuration (Earth-Moon UHF or Earth-Mars S-band). Single subtopic per "Each proposal must target a single subtopic, and NASA will not move proposals between topics—fit matters".

**Action.** Position now. Register on ProSAMS. Prepare proposal narrative skeleton. Submit on FY27 solicitation release.

### 3.3 NASA SBIR Phase II Direct (conditional)

**Channel summary.** Available to Phase I awardees who complete successful feasibility. Phase II cap is ~$850K, 24-month period of performance.

**Caduceus path.** Conditional on §3.2 Phase I landing. Phase II scope: full v0.2 substrate build per CADUCEUS-005 + initial v0.3 optical-terminal integration.

### 3.4 Navy SBIR DON26BZ03-NV062 — Secure Tasking of Commercial Assets

**Channel summary.** "Develop a secure satellite tasking platform that enables classified communication between the U.S. Navy and commercial satellite providers. Seeking cybersecurity, encryption, and secure communications solutions that support CUI and Secret-level operations while reducing tasking timelines by up to 90%. Funding up to $315,000."

**Deadline.** July 22, 2026 — approximately 6 weeks from this document's date.

**Caduceus fit assessment.** Strong alignment on encryption + secure comms; partial alignment on classified Navy domain (Visionblox does not currently hold facility clearance for Secret-level). Pursuit requires teaming with a cleared partner.

**Decision.** Pursue **only if** a cleared teaming partner is available within 2 weeks. Otherwise, defer; do not stretch on classified topics without proper clearance posture.

### 3.5 SDA Tranche 3 Ground Entry Point (GEP) — specialty IP / sub-teaming

**Channel summary.** SDA released a planning notice for Tranche 3 GEPs; the GEP scope includes "GEP site civils, procurement, installation, integration, test and verification of GEP site communications shelters, antennas and antenna electronics, optical terminals and optical electronics, and GEP site networking equipment and software at multiple, globally dispersed, GEP installations". Networking equipment and software is the Caduceus-relevant scope.

**Tranche 3 context.** SDA awarded $3.5B in Tranche 3 Tracking Layer contracts to Lockheed Martin, Rocket Lab, Northrop Grumman, and L3Harris in December 2025; satellites launch NET FY2029. Tranche 3 is the relevant near-term horizon for new IP integration.

**SDA software stack context.** SDA's network management is called NEBULA = Network Established Beyond the Upper Limits of the Atmosphere. Caduceus is positioned as an authority-gating + provenance layer that sits above or alongside NEBULA, not as a NEBULA replacement.

**Caduceus path.** Sub-teaming or specialty IP licensing through one of the four Tranche 3 primes (Lockheed, Rocket Lab, Northrop, L3Harris). Direct prime competition is not realistic for Visionblox at current size.

**Action.** Monitor SDA opportunities page at sda.mil/opportunities/; introduce Caduceus to Tranche 3 prime business-development teams once arXiv preprint is published.

### 3.6 AFRL OTAFI — Open Technology and Agility for Innovation OTA

**Channel summary.** "This five-year, $499 million Other Transaction Agreement (OTA) aims to facilitate a coordinated prototyping and testing program with the Government to accelerate the development of command, control" systems. Available via SOSSEC consortium membership.

**Caduceus fit.** Compliance-graded networking and C4ISR direct alignment. Useful follow-on once §3.1 SOSSEC membership is active.

**Action.** Defer until §3.1 lands. Then submit a SOSSEC pre-proposal through the OTAFI channel.

### 3.7 DIU Commercial Solutions Openings (CSO)

**Channel summary.** Rapid commercial-acquisition channel; no consortium membership required. "DIU CSO responses — pick one or two CSOs per quarter matching your stack. No fee."

**Caduceus fit.** DIU regularly issues CSOs for space comms, cyber, autonomy. Match-rate depends on quarterly topic mix.

**Action.** Subscribe to DIU CSO mailing list; review monthly; respond opportunistically. Low-overhead supplementary channel.

### 3.8 AFWERX SBIR Open Topic

**Channel summary.** "AFWERX SBIR Open Topic — no membership, Phase I pitch days make it the fastest legitimate first-award target."

**Caduceus fit.** Open-topic accepts broad space comms / cyber innovation. Phase I awards ~$75K, fast.

**Action.** Alternative to §3.2 if NASA cycle slips or if a faster first-revenue event is needed. Monitor AFWERX pitch days quarterly.

---

## 4. Teaming partner candidates

### 4.1 By role

| Role | Candidate categories | Specific names (illustrative; require BD outreach) |
|---|---|---|
| **Prime for SDA Tranche 3** | Tranche 3 prime contractors | Lockheed Martin (Sunnyvale), Rocket Lab USA (Long Beach), Northrop Grumman (Redondo Beach), L3Harris (Fort Wayne) |
| **Cleared sub for Navy classified work** | Cleared small businesses or mid-tiers | Identify via NAICS 541512 / 541519 cleared SBIR awardees; project memory does not name specifics |
| **Comm hardware (optical terminal)** | Optical comm specialists | Mynaric, BridgeComm, Astrogate Labs, CACI |
| **Formal-methods collaboration** | Lean 4 / TLA+ practitioners | Galois Inc.; academic partners at MIT CSAIL, Oxford (subject to KeySpace prior-art considerations), CMU; SNHU CS faculty |
| **Mission-assurance review** | NASA OSMA-class reviewers | NASA OSMA direct engagement; The Aerospace Corporation; MITRE |
| **Academic publication partners** | IEEE Aerospace / SOSP authors | SNHU; established space-cyber labs |

### 4.2 Teaming agreement posture

Visionblox should position as **technology + IP holder** in teaming agreements with primes. Specifically: Caduceus PPAs filed (CADUCEUS-IP-001/002/003) give Visionblox enforceable IP positions that primes can license or sub-contract for. Typical structure: Visionblox holds IP, prime holds prime contract, sub-task with defined deliverables.

This positioning fits Visionblox's small-business posture: do not compete on volume; compete on IP and methodology.

---

## 5. Compliance posture (federal-relevant)

| Standard | Status | Capture relevance |
|---|---|---|
| **OMB M-25-22 (AIBOM)** | AIBOM Generator in progress per project memory | Directly required by federal AI procurement; Caduceus's Aletheia substrate is M-25-22-native |
| **OMB M-26-04** | Tracking | Required for federal AI; integrated with AIBOM scope |
| **CycloneDX 1.6 / SPDX 2.3** | Output formats supported by AIBOM Generator | Required for SBOM/AIBOM consumption |
| **CMMC Level 2** | Self-assessment in motion | Required for DoD contracts handling CUI; gating for Navy SBIR and SDA paths |
| **FedRAMP** | Not yet | Required if Caduceus is offered as SaaS to federal customers; not gating for SBIR / OTA prototype work |
| **FISMA** | Civium track addresses this | Required for federal information system deployment; v0.3 substrate work |
| **HIPAA** | Civium track addresses this | Relevant for NASA crew biomedical applications |
| **NIST 800-53** | Track | Required for federal information system authorization |
| **NIST 800-171** | Required for handling federal CUI | Tracking via CMMC L2 path |
| **ANSI/RIA R15.08-1/2 / ISO 10218 / ISO/TS 15066** | Track | Relevant if Caduceus extends to robotic / autonomous physical actions; not v0.2 scope |

**Gating gap:** CMMC L2 must be completed before any handling of CUI under §3.4 Navy or §3.5 SDA work. The CMMC L2 self-assessment in motion is on critical path for any path beyond §3.1 SOSSEC + §3.2 NASA SBIR.

---

## 6. Pricing posture

### 6.1 Phase I / OTA prototype pricing model

For early-stage federal capture, recommended posture: **cost-plus or fixed-price-prototype** with high transparency. Match SBIR Phase I caps:

| Channel | Cap | Visionblox proposed |
|---|---|---|
| NASA SBIR Phase I | $150K | $145K (within cap; 6-month) |
| Navy SBIR Phase I | $315K (for this specific topic) | $300K (within cap; if pursued) |
| AFWERX SBIR Phase I | ~$75K | $74K (within cap) |
| SOSSEC OTA prototype | Varies (typical $250K–$2M) | Scope-dependent |
| DIU CSO | Varies | Scope-dependent |

### 6.2 Labor rate posture

Visionblox should establish a labor rate schedule reflecting actual loaded costs (not aspirational rates). Anticipated rates:

- Senior engineer (Khaalis class): $250–$300/hr fully loaded
- Mid-level engineer: $150–$200/hr
- Junior engineer: $90–$130/hr
- Subject-matter expert (formal-methods researcher, radio engineer): $200–$275/hr

DCAA-compliant accounting must be established before any cost-type contracts. CMMC L2 has DCAA-adjacent practices.

### 6.3 IP retention posture

For SBIR awards: **Visionblox retains SBIR data rights for 5 years (now expanded to 20 years under recent SBIR reauthorization).** This is a critical posture. Do not negotiate away IP rights without practitioner review.

For OTA prototype awards: IP terms are negotiable per-agreement. Visionblox's posture: **retain background IP (Caduceus PPAs); license foreground IP under government-purpose rights** unless commercial-use terms are favorable.

---

## 7. Risk factors (honest)

| Risk | Severity | Mitigation |
|---|---|---|
| **No federal-contracting past performance** | HIGH | Frame as new-entrant strength; emphasize methodology rigor and artifact base; team with established performers for larger awards |
| **No facility clearance** | MEDIUM-HIGH | Limits §3.4 Navy and parts of §3.5 SDA. Pursue interim DD-254-based partnerships; do not pursue classified work solo |
| **Small team (single principal currently)** | HIGH | Phase I awards are feasible solo; Phase II and OTA awards require staffing plan; teaming agreements bridge this gap |
| **KeySpace 2024 prior-art exposure** | MEDIUM | Tighten IP-001 claims per CADUCEUS-PRACTITIONER-001 §2.3; position Caduceus as compliance-graded complement to KeySpace's PKI work, not competitor |
| **NASA SBIR FY27 timing dependency** | MEDIUM | If NASA cycle slips, pivot to AFWERX Open Topic or DIU CSO |
| **CMMC L2 gating on most non-NASA channels** | MEDIUM-HIGH | CMMC L2 self-assessment must complete before §3.4, §3.5, §3.6 paths open |
| **Optical terminal partner unavailable under §6.4 caps** | MEDIUM | v0.2 substrate ships without optical; optical is a v0.3 stretch with explicit cap as exit criterion |
| **OTA consortium concentration** | LOW | Top OTA vendors are concentrated; SOSSEC is in the top five — concentration is a feature for visibility, not a risk for member access |

---

## 8. Operational sequencing

| Quarter | Channel actions | Dependency |
|---|---|---|
| **Q3 2026 (Jul–Sep)** | Complete LEDGER #0004 commit; file PPAs; publish arXiv preprint | CADUCEUS-PRACTITIONER-001 §5 sequence |
| **Q3 2026** | Complete NSTXL or SOSSEC membership | §3.1 |
| **Q3 2026 (if pursued)** | Navy SBIR DON26BZ03-NV062 — by July 22 | §3.4; requires cleared teaming partner |
| **Q3 2026** | DIU CSO mailing list subscription; AFWERX pitch-day registration | §3.7, §3.8 |
| **Q4 2026** | Complete CMMC L2 self-assessment | Unlocks classified-adjacent paths |
| **Q4 2026 – Q1 2027** | SDA Tranche 3 prime BD outreach | §3.5; following arXiv publication |
| **Q1 2027 – Q2 2027** | Prepare NASA SBIR FY27 Phase I proposal (SCaN subtopic) | §3.2; aligned to anticipated solicitation window |
| **Q2 2027** | Submit NASA SBIR FY27 Phase I | On solicitation release |
| **Q3 2027 – Q1 2028** | SBIR Phase I execution (if awarded) | §3.2 award |
| **Q4 2027** | SOSSEC OTAFI pre-proposal | §3.6; after SOSSEC membership active and a relevant call exists |
| **Q1 2028** | NASA SBIR Phase II Direct application | §3.3; conditional on Phase I success |
| **Q2 2028 – Q4 2028** | Phase II execution; v0.3 substrate build (CADUCEUS-005 timeline) | §3.3 award |

---

## 9. Open items

1. **NSTXL vs. direct SOSSEC membership** — confirm which provides better access for Visionblox's scope; depends on which consortia each manages.
2. **Cleared teaming partner identification** — required before §3.4 Navy or classified §3.5 SDA. Recommended: small-business OSINT against cleared SBIR awardees in NAICS 541512 / 541519.
3. **CMMC L2 self-assessment completion date** — gating for multiple channels; needs explicit project tracking.
4. **NASA SBIR FY27 solicitation date confirmation** — monitor sbir.nasa.gov starting Q4 2026.
5. **arXiv preprint scope** — should arXiv contain only Caduceus-bench v1.2.1 narrative, or also formal-methods sketch? Practitioner advised; coordination with §3.x narrative posture.
6. **DCAA-compliant accounting setup** — required for any cost-type federal contract; consultant engagement.
7. **Past-performance narrative construction** — non-trivial; honest-yet-compelling framing of new-entrant status with strong artifact base.

---

## 10. Honest scope recap

- This document is planning material. Actual proposal submission requires substantial additional work per channel (proposal writing, cost narrative, compliance documentation, etc.).
- Channel intel as of 2026-06-08 web search; re-verify before any action.
- The recommended sequence (§8) is achievable for a small team but requires sustained capture discipline. The biggest risk is over-pursuing channels and under-delivering on any one.
- Caduceus's compliance-graded posture is genuinely differentiated, but federal capture rewards consistency of presentation; the same capability framing should appear across all proposals.
- Visionblox should commit to **one or two primary channels** in any given quarter, not all of them simultaneously. The §1.2 priority ordering reflects this discipline.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-CAPTURE-001 Rev A · Federal capture posture*
