# NSTXL Membership Application Briefing

| | |
|---|---|
| **Document ID** | CADUCEUS-NSTXL-001 |
| **Revision** | A |
| **Purpose** | Operational briefing for Visionblox LLC to join the National Security Technology Accelerator (NSTXL) consortium and access its managed OTA vehicles |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Target action window** | Q3 2026 (after LEDGER #0004 commit + PPA filings; not gated by them) |

---

## 0. Why NSTXL

Per CADUCEUS-CAPTURE-001 §1.2, NSTXL membership is Priority 1 — cheapest entry to the broadest set of OTA vehicles relevant to Caduceus. The capture posture document identified NSTXL as the recommended consortium membership path because it provides access to multiple consortia under a single membership.

NSTXL manages or co-manages:

- **S2MARTS** — Strategic & Spectrum Mission Advanced Resilient Trusted Systems; cybersecurity, EW, microelectronics
- **SOSSEC** — System of Systems Security Consortium; C4ISR, mission-secure networking, cybersecurity
- **Tradewinds** — DoD's commercial-AI marketplace
- **Prototype Project Opportunities** across multiple DoD components

For Visionblox's compliance-graded interplanetary fabric work, S2MARTS and SOSSEC are the primary value. Tradewinds is a low-overhead supplementary channel for AI-component opportunities.

---

## 1. Membership tiers and fees (as of 2026-06-08 research)

NSTXL membership is structured in tiers based on organization size and revenue. Approximate fees from public sources:

| Tier | Indicative fee | Eligibility |
|---|---|---|
| Small Business | $500–$1,500/year | < $40M revenue or per SBA size standard |
| Mid-Tier | $5,000–$15,000/year | $40M–$500M revenue |
| Large Business | $15,000–$50,000/year | > $500M revenue |
| Academic / Non-profit | $0–$500/year | Educational institutions, research centers |

Visionblox qualifies for **Small Business tier**. Anticipated fee: $500–$1,500/year.

**Verification step (Visionblox-action):** confirm current fee structure at nstxl.org/membership before payment. Fee tiers and amounts change annually.

---

## 2. Application process

NSTXL membership is a straightforward application process. Typical flow:

| Step | Action | Estimated time |
|---|---|---|
| 1 | Create NSTXL account at nstxl.org | 15 minutes |
| 2 | Complete organization profile (NAICS codes, capabilities, certifications) | 30–60 minutes |
| 3 | Submit business documentation (W-9, SAM.gov registration confirmation, articles of incorporation) | 15 minutes if documents are ready |
| 4 | Pay membership fee via credit card or wire | 10 minutes |
| 5 | Sign Consortium Member Agreement (CMA) electronically | 15 minutes |
| 6 | NSTXL membership team review | 5–10 business days |
| 7 | Receive membership confirmation; access to consortium portal | — |

**Total active time:** approximately 1–2 hours of Khaalis's time, plus 5–10 days of NSTXL processing.

---

## 3. Pre-application prerequisites

Before submitting the NSTXL application, Visionblox should have these in place:

| Prerequisite | Status (per project context) | Action if missing |
|---|---|---|
| Active SAM.gov registration | Should be in place per existing federal-tracking posture | Register at sam.gov; ~7-day process |
| DUNS number / UEI | Required; SAM.gov assigns UEI | Same as SAM.gov |
| EIN (Employer Identification Number) | Likely in place for Visionblox LLC | IRS form SS-4 if missing |
| W-9 form completed | Standard | Generate from IRS form |
| Articles of incorporation for Visionblox LLC | In place | — |
| NAICS code declarations | Per memory: 541511, 541512, 541519, 541611, 541690, 518210, 541715 | Update profile to include all |
| Capability statement (brief) | Generate from CADUCEUS-CAPTURE-001 §2 | — |

**Verification step (Visionblox-action):** confirm SAM.gov registration is active and not in renewal lapse. SAM.gov registrations expire annually; lapsed registration disqualifies from federal contracting.

---

## 4. NAICS code positioning for NSTXL profile

The NAICS codes from project memory map to Caduceus capabilities as follows. NSTXL's consortium-administrator team uses these codes to route opportunities to relevant members.

| NAICS | Description | Caduceus relevance |
|---|---|---|
| **541511** | Custom Computer Programming Services | Primary code — Caduceus is a software substrate |
| **541512** | Computer Systems Design Services | Secondary — system architecture work |
| **541519** | Other Computer Related Services | Auxiliary |
| **541611** | Administrative Management & General Management Consulting | Civium compliance-grading is consulting-adjacent |
| **541690** | Other Scientific & Technical Consulting Services | Mission-assurance / formal-methods adjacent |
| **518210** | Data Processing, Hosting, & Related Services | If Caduceus is offered as a service rather than only as software |
| **541715** | Research and Development in the Physical, Engineering, and Life Sciences | R&D track for v0.2 substrate build |

**Recommendation:** declare all seven NAICS codes in the NSTXL profile, with 541511 as primary. Broader declaration captures more inbound opportunities.

---

## 5. Capability narrative for NSTXL profile

Suggested capability statement (200–400 words is typical for NSTXL profile):

> Visionblox LLC develops compliance-graded delay-tolerant communications substrates for interplanetary and high-RTT operational environments. Our flagship platform, Caduceus, is a falsifiable benchmark and substrate plan for bundle-fabric authority propagation with cryptographically-anchored provenance (Aletheia ledger), graph-based authority gating (Civium), drift-aware link-integrity monitoring, frame-disambiguated namespace management, and crew-attestation peer architecture. Caduceus operates above standardized DTN substrate (RFC 9171 BP7, RFC 9172 BPSec) and addresses propagation-window failure modes specific to interplanetary deployments — failure modes not addressable by terrestrial PKI mechanisms.
>
> Visionblox's methodology is the ZCS-6 falsification-first ordering discipline: every benchmark is cryptographically committed before any implementation, preventing benchmark-gaming and Goodhart's-Law failure modes. Our adversarial-sweep discipline surfaced twenty named attacks against caduceus-bench v1.2.1 across two iterations; all twenty are closed in the final commit (Aletheia LEDGER #0004).
>
> Adjacent capabilities include AIBOM-compatible compliance posture (OMB M-25-22), CMMC Level 2 self-attestation, formal-methods composition with the Mercury Subleq kernel via Lean 4 + TLA+, and OSCAL-compatible evidence-pack exporters. Past artifacts include the VBX-ISPS interplanetary spacesuit substrate (Python prototype v0.1, 31/31 tests, IEEE paper produced), the Mercury Subleq RTL simulator (SSRN-published), and five live intelligence watchers operating via GitHub Actions for federal procurement opportunity discovery.
>
> Visionblox is a small-business non-traditional defense performer with substantial methodology rigor and public artifact base, well-positioned for OTA prototype-class work in interplanetary communications, compliance-graded edge networking, and mission-secure C4ISR extensions.

---

## 6. Post-membership onboarding

After NSTXL processes the membership:

| Action | Timing |
|---|---|
| Access NSTXL Members Portal | Within 24 hours of confirmation |
| Subscribe to OTA solicitation announcements (SOSSEC, S2MARTS, Tradewinds) | Same day as portal access |
| Attend next NSTXL Members Meeting (typically monthly virtual) | Within first 30 days |
| Submit at least one introductory project proposal (low-effort, $50K–$250K scale) to a small SOSSEC or S2MARTS call to establish track record | Within first 90 days |
| Network with other Members in the Caduceus-adjacent space (sub-teaming candidates per CADUCEUS-CAPTURE-001 §4) | Ongoing |

---

## 7. First-OTA-pursuit recommendations

Once NSTXL membership is active, the highest-leverage initial pursuits:

### 7.1 SOSSEC OTA — mission-secure networking topic

When a SOSSEC OTA call addresses mission-secure networking, secure communications, or zero-trust implementation, Caduceus is directly aligned. Recommended posture:

- Bid as **sole performer** for prototype scope ($250K–$1M)
- Position Caduceus's interplanetary-specific compliance-grading as differentiator versus terrestrial-PKI competitors
- Reference LEDGER #0004 cryptographic commit as a credibility anchor (no other small business will have a cryptographically-committed benchmark)

### 7.2 S2MARTS OTA — cybersecurity / EW topic

S2MARTS focuses on cybersecurity, electronic warfare, and microelectronics. Caduceus fits the cybersecurity dimension if framed as compliance-graded cyber-resilient communications.

- Bid as **sole performer** initially
- Larger awards may require teaming with established S2MARTS members; identify candidates via portal directory

### 7.3 Tradewinds Solutions Marketplace — AI component

Tradewinds is DoD's commercial AI marketplace. Caduceus's Civium graph-authority engine is AI-adjacent; the Aletheia chain is AIBOM-relevant.

- Upload Visionblox profile with focus on AI compliance components
- Respond to relevant marketplace task orders opportunistically (low overhead, no fee)

---

## 8. Risk factors

| Risk | Mitigation |
|---|---|
| **Membership fee for marginal value** | Fee is small ($500–$1,500); ROI threshold is one prototype award |
| **OTA proposals require substantial effort** | Start with small calls ($250K range); build proposal-writing muscle |
| **Past-performance lacuna** | NSTXL membership itself becomes a past-performance anchor; first OTA award (even small) significantly strengthens future bids |
| **CMMC L2 gating for certain calls** | Per CADUCEUS-CMMC-001, complete L2 self-assessment before pursuing CUI-class calls |
| **Time investment for member-meetings and networking** | Budget ~4–8 hours/month for member engagement |

---

## 9. Decision checklist for Khaalis

Before submitting the application:

- [ ] SAM.gov registration confirmed active
- [ ] UEI number on hand
- [ ] W-9 ready
- [ ] $500–$1,500 budget allocated for membership fee
- [ ] 1–2 hours blocked for application completion
- [ ] Capability statement drafted (use §5 above as starting point)
- [ ] NAICS codes confirmed accurate for Visionblox

After membership active:

- [ ] Monthly member meeting attended
- [ ] First OTA pursuit identified within 90 days
- [ ] First proposal submitted within 6 months

---

## 10. Honest scope statement

- This document is operational guidance. The actual application is performed by Khaalis via the NSTXL portal.
- Fee tiers and processes change; verify at nstxl.org before payment.
- NSTXL membership is necessary but not sufficient for OTA wins. Substantial proposal-writing effort is required for each pursuit.
- First-year ROI may be modest as Visionblox builds member-network relationships and refines proposal craft. Multi-year horizon is typical for federal capture build-up.
- NSTXL is one consortium of several; if a specific call appears in a non-NSTXL consortium (e.g., CMG, ATI, NCMS), separate membership in that consortium may be warranted. NSTXL is the recommended primary, not exclusive, membership.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-NSTXL-001 Rev A · NSTXL membership application briefing*
