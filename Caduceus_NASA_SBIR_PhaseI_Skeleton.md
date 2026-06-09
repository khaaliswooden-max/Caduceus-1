# NASA SBIR Phase I Proposal Skeleton — SCaN Cognitive Communication

| | |
|---|---|
| **Document ID** | CADUCEUS-SBIR-NASA-001 |
| **Revision** | A (skeleton — full proposal writing required) |
| **Target solicitation** | NASA SBIR FY27 Phase I (anticipated open window: March–May 2027) |
| **Target subtopic** | Space Communications and Navigation (SCaN) — Cognitive Communication |
| **Target award amount** | $150,000 (NASA Phase I cap) |
| **Period of performance** | 6 months |
| **Proposing organization** | Visionblox LLC (small business) |
| **Principal Investigator** | A. Khaalis Wooden, Sr. |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |

---

## 0. Honest scope statement

This document is a **proposal skeleton with key talking points and recommended content per section**. It is not a submission-ready proposal. A full SBIR Phase I proposal requires approximately 80–150 hours of focused writing, including detailed technical narrative, cost narrative, commercialization plan, and supporting documents. This skeleton provides the structure and the Caduceus-specific positioning that the full writing should preserve.

NASA SBIR FY27 solicitation is anticipated to open March–May 2027 per public sources; verify exact dates and topic structure when the FY27 BAA is released. Topic identifiers (e.g., COMNAV.x.S27a) may differ from FY26.

---

## 1. NASA SBIR Phase I structure

NASA SBIR Phase I proposals follow a standardized structure with section page limits totaling 15 pages of technical content. Submission is via the ProSAMS system.

| Section | Pages | Skeleton status |
|---|---|---|
| Form A — Cover Page | 1 | §2 below |
| Form B — Project Summary | 1 | §3 below |
| Form C — Technical Proposal | 15 (hard limit) | §§4–11 below |
| Form D — Cost Volume | — | §12 below |
| Form E — Briefing Chart | 1 | Generate from §3 abstract |
| Form F — NASA Commercialization Plan | 5 | §13 below |
| Form G — Past Performance | — | §14 below |
| Form H — Key Personnel | — | §15 below |

---

## 2. Form A — Cover Page

| Field | Content |
|---|---|
| Proposal title | Caduceus Cognitive Authority Substrate for SCaN Bundle Fabric |
| Solicitation | NASA SBIR FY27 Phase I, BAA 80NSSCxxRxxxx (TBD) |
| Subtopic | SCaN Cognitive Communication |
| Firm name | Visionblox LLC |
| Principal Investigator | A. Khaalis Wooden, Sr., MBA |
| Business officer | A. Khaalis Wooden, Sr. (or designated; small-business posture) |
| Total proposed cost | $145,000 (within $150,000 cap) |
| Period of performance | 6 months |
| Place of performance | Visionblox LLC (Huntsville, AL or remote) |
| Foreign nationals participating | None |
| Inventions / patents | CADUCEUS-IP-001, CADUCEUS-IP-002, CADUCEUS-IP-003 (filed pre-solicitation) |

---

## 3. Form B — Project Summary (1 page; ~250–400 words)

### 3.1 Identification & Significance

> NASA's Space Communications and Navigation (SCaN) Next Generation Architecture requires cognitive networking capabilities that can operate across the long propagation delays inherent to lunar and Mars communications. Existing public-key infrastructure mechanisms developed for terrestrial networks assume sub-second revocation propagation; standardized space-data-link security protocols use pre-shared symmetric keys without graph-authority semantics. Neither addresses compliance-graded authority decisions where round-trip light-time exceeds the desired authority-decision cutover time.
>
> Visionblox LLC proposes Caduceus, a compliance-graded delay-tolerant bundle fabric specifically designed for SCaN-class interplanetary authority propagation. Caduceus sits above standard Bundle Protocol v7 (RFC 9171) and Bundle Protocol Security (RFC 9172) and provides per-bundle cryptographic provenance (Aletheia ledger, hash-chained blockchain-style data anchoring directly aligned with SCaN's stated interest in blockchain-based data processing), graph-based authority gating, drift-aware link-integrity monitoring, and a versioned namespace for spatial-frame disambiguation.

### 3.2 Technical Objectives (Phase I)

> Phase I will: (a) develop a working substrate prototype of the Caduceus authority-graph and revocation-stream layer on the Visionblox MVCI 12-agent open-source stack; (b) demonstrate the substrate against a NASA-relevant link configuration (Earth-Moon UHF or Earth-Mars S-band, per government-furnished configuration); (c) validate compliance against caduceus-bench v1.2.1, a cryptographically-committed falsifiable benchmark (Aletheia LEDGER #0004) of seventeen normative assertions; (d) produce a Phase II proposal documenting the full v0.2 substrate build for crewed-mission-class deployment.

### 3.3 Work Plan

> The work plan executes ten technical tasks over 6 months at 100% Visionblox labor. Deliverables include a working substrate prototype, a benchmark compliance report signed and committed to the Aletheia chain, an updated arXiv defensive preprint, and a Phase II proposal with v0.3 hardware partnership plan.

### 3.4 Commercial Applications

> Beyond NASA SCaN, Caduceus applies to: Space Development Agency Tranche 3 Ground Entry Point software; commercial lunar/cislunar operations (Gateway, lunar surface ops by commercial providers); compliance-graded edge inference platforms in regulated terrestrial verticals (Civium parent platform serves healthcare and federal civilian compliance markets).

---

## 4. Form C, Section 1 — Identification and Significance of the Innovation (~2 pages)

### 4.1 The propagation-window problem

(Per CADUCEUS-PAPER §1 introduction language — adapt for SBIR-reviewer audience emphasizing NASA mission relevance.)

Key points:
- Earth-Moon RTT: 2.5 seconds (one-way 1.25s × 2)
- Earth-Mars RTT: 6–44 minutes
- Earth-deep-space RTT: 8 hours+
- CRL/OCSP / Kerberos: sub-second propagation assumption
- CCSDS SDLS: pre-shared symmetric, no graph authority
- KeySpace 2024 (Smailes et al.): demonstrates terrestrial PKI adaptable but does not address compliance-graded authority decisions

### 4.2 Caduceus differentiation

Distinguish from KeySpace 2024 (closest prior art):
- KeySpace addresses PKI revocation mechanisms in interplanetary networks
- Caduceus addresses compliance-graded authority **decisions** built on top of any PKI substrate, including KeySpace's mechanisms
- Complementary, not competitive

### 4.3 Significance to NASA SCaN

- SCaN Next Generation Architecture explicitly calls for cognitive networks
- Per SCaN Cognitive Communication subtopic: "STMD recently awarded two Early Career Faculty grants to study topics related to Cognitive Communication including distributed network routing and blockchain-based data processing" — direct alignment with Aletheia hash-chained provenance
- Caduceus's Civium graph-authority and Mercury-bounded WCET deliver autonomous decision substrate consistent with SCaN's autonomy goals

---

## 5. Form C, Section 2 — Technical Objectives (~1 page)

Five specific, measurable Phase I objectives:

| # | Objective | Success criterion |
|---|---|---|
| **O1** | Implement HF-17 (inbound verification + Civium authority gate) module in Rust against caduceus-bench v1.2.1 T1, T2, T3, T7 assertions | Unit tests pass on T1–T7; performance within Mercury-WCET bounds on Jetson Orin Nano (lower-cost Phase I hardware) |
| **O2** | Implement HF-19 (URN parsing + Caduceus-BodyID v1.0 namespace) module | Pass-or-fail tests against F1, F2 assertions; namespace-exhaustion attack (Attack 8) closed |
| **O3** | Demonstrate end-to-end bundle flow Earth-Moon-class scenario using simulated link (latency injection) | Round-trip authority decision under 3-second RTT operates within bench compliance |
| **O4** | Conduct adversarial sweep against the Phase I prototype using Attacks 1, 2, 10, 27 (from CADUCEUS-BENCH-001) | All four attacks closed in prototype implementation |
| **O5** | Produce a Phase II proposal demonstrating v0.2 substrate full build feasibility | Phase II proposal submitted within 30 days of Phase I completion |

---

## 6. Form C, Section 3 — Work Plan (~3 pages)

Ten tasks over 26 weeks (6 months).

### 6.1 Task table

| Task | Description | Weeks | Effort (hrs) |
|---|---|---|---|
| T1 | Project setup, ProSAMS administration, kickoff | 1 | 20 |
| T2 | HF-17 module design and implementation | 2–8 | 200 |
| T3 | HF-19 module design and implementation | 6–10 | 120 |
| T4 | Earth-Moon-class link simulation harness | 5–8 | 80 |
| T5 | End-to-end integration test | 9–12 | 80 |
| T6 | Adversarial sweep (Attacks 1, 2, 10, 27) | 11–14 | 80 |
| T7 | Bench compliance report (signed, committed to Aletheia) | 13–16 | 60 |
| T8 | Phase II proposal writing | 17–22 | 120 |
| T9 | Final report; deliverable hand-over to NASA | 23–25 | 60 |
| T10 | Project closeout | 26 | 20 |
| **Total** | | **26 weeks** | **840 hours** |

### 6.2 Resource loading

PI (Khaalis): 70% allocation × 26 weeks × 40 hrs/wk = 728 hours
Mid-level engineer (subcontractor or co-PI): 20% × 26 weeks × 22 hrs/wk = 114 hours
**Total labor**: 842 hours (matches table at ±2 hrs)

---

## 7. Form C, Section 4 — Performance Schedule

Gantt-equivalent timeline:

```
Week:   1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26
T1     [#]
T2        [#######################]
T3                 [###############]
T4              [#####]
T5                       [############]
T6                          [############]
T7                                [############]
T8                                            [#######################]
T9                                                              [######]
T10                                                                            [#]
```

---

## 8. Form C, Section 5 — Related R&D (~1 page)

- Visionblox VBX-ISPS substrate v0.1 (Python prototype, 31/31 tests, IEEE paper produced)
- Mercury Subleq RTL simulator (SSRN-published)
- Aletheia DAC v0.2 (in production at Visionblox)
- Civium compliance-graded edge inference platform (Visionblox)
- caduceus-bench v1.2.1 (cryptographically committed via LEDGER #0004 pre-solicitation)

### 8.1 Prior NASA / federal funding

> Visionblox LLC has not previously received NASA SBIR or other federal R&D awards as of the proposal date. All Caduceus, VBX-ISPS, Mercury, Aletheia, and Civium work has been internally funded by Visionblox. This proposal represents Visionblox's first NASA SBIR engagement.

---

## 9. Form C, Section 6 — Key Personnel (~1 page)

### 9.1 Principal Investigator

> **A. Khaalis Wooden, Sr., MBA**
> Director of Enterprise Capture & Compliance, Visionblox LLC
> MSIT Candidate, Southern New Hampshire University
>
> 25+ years of [project-context-relevant experience]. Sole author of the Caduceus benchmark spec, VBX-ISPS substrate work, and Mercury kernel design. Methodology lead for the ZCS-6 falsification-first ordering discipline. Author of 1+ peer-reviewed publications (SSRN Mercury) and 1+ patent application portfolios (CADUCEUS-IP-001/002/003 in filing).

### 9.2 Subcontractor or Co-PI

> [Identify Phase I-stage subcontractor; small-business proposals typically benefit from a named academic or industry collaborator. Candidates per CADUCEUS-CAPTURE-001 §4: SNHU CS faculty, Galois Inc. formal-methods consulting, or independent radio-engineering consultant for theoretical-floor work.]

---

## 10. Form C, Section 7 — Facilities and Equipment (~0.5 page)

> Visionblox LLC operates from a [home-office or facility-class] location in [Huntsville, AL or remote]. Phase I work does not require classified facility, optical-test range, or radio-frequency test chamber. All compute work is performed on commercial NVIDIA Jetson Orin Nano development kits (lower-cost Phase I hardware; Phase II will migrate to Orin NX production-class). Software development environment: Rust toolchain, Lean 4 for formal-methods exploration, ION-DTN reference implementation, libsodium cryptographic library.
>
> No classified facility clearance is required for Phase I.

---

## 11. Form C, Section 8 — Consultants and Subcontractors (~0.5 page)

> Visionblox anticipates subcontracting approximately 15% of Phase I labor to a specialized formal-methods or radio-engineering consultant. Specific subcontractor identification will be confirmed at award time.

---

## 12. Form D — Cost Volume

NASA Phase I cost narrative requires:

| Cost element | Amount |
|---|---|
| Direct labor (PI + subcontractor) | $80,000 |
| Fringe benefits (assume 25% of labor) | $20,000 |
| Equipment (Jetson Orin Nano dev kits, radio test gear) | $8,000 |
| Travel (1 NASA site visit + 1 conference) | $4,000 |
| Materials and supplies | $2,000 |
| Subcontractor (formal-methods or radio engineering) | $15,000 |
| Other direct costs (cloud compute, software licenses) | $3,000 |
| Indirect costs (assume 10% rate for small business) | $13,000 |
| **Total** | **$145,000** |

Profit/fee: NASA SBIR Phase I allows up to 7% fee on top of direct + indirect; whether to include depends on Visionblox accounting posture. Conservative approach: do not include profit on Phase I; reinvest into Phase II development.

---

## 13. Form F — NASA Commercialization Plan (~5 pages)

### 13.1 Commercial market analysis

Three commercial markets for Caduceus:

| Market | Size estimate | Timing |
|---|---|---|
| NASA SCaN ground-network and Lunar/Mars relay operations | Internal NASA budget; aligned with $X00M SCaN annual | Direct Phase II / III continuation |
| Space Development Agency Tranche 3+ Ground Entry Point software | $0.5–1B addressable across SDA software-stack work | 2027–2030 deployment horizon |
| Commercial lunar operations (Gateway commercial partners, lunar lander operators, future commercial Mars) | Emerging; $100M–$500M addressable | 2028–2032 |
| Regulated terrestrial edge platforms (Civium parent platform) | Healthcare + federal civilian | Current revenue track |

### 13.2 Commercialization strategy

- Phase II (anticipated $850K, 24 months): full v0.2 substrate build to TRL 5
- Phase III: transition via NASA Phase III contract, SBIR data rights retention, follow-on commercial procurement
- Parallel SDA pursuit via Tranche 3 ground-segment opportunities (commercial pathway via prime sub-teaming)
- IP licensing posture: Visionblox retains background IP (CADUCEUS-IP-001/002/003); offers government-purpose rights on foreground; commercial licenses for terrestrial applications

### 13.3 Investor / customer pipeline (if available)

[Visionblox to complete with actual conversations or letters of intent if obtained pre-submission.]

---

## 14. Form G — Past Performance (~1 page)

Honest disclosure:

> Visionblox LLC has no prior NASA SBIR awards as of the proposal date. The firm's relevant past performance consists of internally-funded research and development resulting in: (a) the VBX-ISPS interplanetary spacesuit substrate (Python prototype v0.1, 31/31 tests passing); (b) the Mercury Subleq RTL simulator (SSRN-published); (c) the AIBOM Generator (M-25-22 compliant, in production); (d) the Aletheia DAC v0.2 (production); (e) the Civium compliance-graded edge inference platform (in production); (f) five live intelligence watchers operational on GitHub Actions for federal procurement opportunity discovery; (g) caduceus-bench v1.2.1 (cryptographically committed via LEDGER #0004 pre-solicitation).
>
> The firm's CMMC L2 self-assessment is in completion (per CADUCEUS-CMMC-001). Active SAM.gov registration, UEI assigned. No prior contract performance ratings exist; the firm is a new federal-contracting entrant.

---

## 15. Form H — Key Personnel

[Resume of A. Khaalis Wooden, Sr., per standard SBIR resume format. Include MSIT enrollment at SNHU, MBA credential, project artifact portfolio.]

---

## 16. Required attachments

- Cover letter
- Visionblox LLC articles of incorporation
- W-9
- SAM.gov registration confirmation
- Subcontractor letters of commitment
- Equipment vendor quotes
- Patent application receipts (CADUCEUS-IP-001/002/003)

---

## 17. Submission checklist

- [ ] ProSAMS account active for Visionblox
- [ ] BAA released; subtopic IDs confirmed for FY27
- [ ] All form pages within page limits
- [ ] No foreign nationals participating; if any, additional forms required
- [ ] Cost volume reconciles to $145,000 ≤ $150,000 cap
- [ ] Commercialization plan complete (5 pages)
- [ ] Resumes for all named key personnel
- [ ] Subcontractor commitments documented
- [ ] Multiple-proposal limit observed (≤ 2 proposals per firm per cycle per NASA policy)
- [ ] Submission deadline confirmed and at least 48 hours of buffer before deadline

---

## 18. Honest scope statement

- This document is a skeleton, not a submission-ready proposal. Approximately 80–150 additional hours of focused proposal writing are required.
- Topic structure for FY27 may differ from FY26; verify exact subtopic IDs and page limits when BAA releases.
- Subcontractor identification is open; depending on selection, the technical narrative will adjust to emphasize that capability.
- The $145,000 cost is illustrative; actual cost will depend on Visionblox's then-current labor rates and subcontractor commitments.
- Past performance is the weakest section for a new federal entrant. Mitigation is comprehensive disclosure of internally-funded artifact base with substantive technical detail demonstrating capability.
- Phase I award rates are competitive but not impossible for a focused, well-aligned proposal; NASA typically funds 10–15% of submitted Phase I proposals per cycle.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-SBIR-NASA-001 Rev A · NASA SBIR Phase I proposal skeleton*
