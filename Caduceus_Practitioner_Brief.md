# Caduceus IP Strategy — Practitioner Engagement Brief

| | |
|---|---|
| **Document ID** | CADUCEUS-PRACTITIONER-001 |
| **Revision** | A |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **Brief for transmission to a registered USPTO patent practitioner.** This document compiles prior-art findings, trademark search results, and operational sequencing for the Caduceus PPA filing campaign. |
| **Recipient (anticipated)** | TBD — registered USPTO patent attorney or agent with computer-systems and/or aerospace patent prosecution experience |
| **Companion documents** | CADUCEUS-IP-001 (Attack 10 PPA target), CADUCEUS-IP-002 (Attack 27 PPA target), CADUCEUS-IP-003 (F4 PPA target), CADUCEUS-004 (caduceus-bench v1.2.1 source disclosure) |

---

## Honest scope statement (read first)

This brief was prepared by Claude (Anthropic), an AI system, on behalf of the inventor. It is **preparation material, not authoritative legal work product**. The prior-art search reflected here was conducted using web search and accessible literature; it does NOT substitute for the practitioner's authoritative search using Lexis, Westlaw, PatBase, Google Patents Advanced, USPTO PAIR, or equivalent professional databases. The trademark search reflected here approximates a TESS query using the modernized USPTO public-facing search; practitioner should re-run on the official `tmsearch.uspto.gov` and Trademark Status & Document Retrieval (TSDR) systems before action.

A material finding (§2.3 below: KeySpace 2024) emerged during preparation that significantly affects the Attack 10 PPA claim scope. **This brief flags it explicitly to ensure the practitioner can address it before filing.**

---

## 1. Executive summary

Three PPA drafting targets are ready for practitioner review:

| Target | Subject | Novelty confidence | § 101 risk |
|---|---|---|---|
| **CADUCEUS-IP-001** | Interplanetary-RTT-aware compliance-graded authority propagation | **MEDIUM** (revised down from HIGH given KeySpace 2024 finding) | MEDIUM |
| **CADUCEUS-IP-002** | URN-scheme-versioned dual-version retention window for interplanetary registry transitions | MEDIUM-HIGH | MEDIUM |
| **CADUCEUS-IP-003** | Root-cause clustering with hard rate limit for crew confirmation in high-RTT mission ops | MEDIUM | MEDIUM-LOW |

Trademark posture for "Caduceus" in IC 9: **defensible with descriptor**. Multiple existing CADUCEUS marks in IC 9 are in medical/healthcare context; space-tech use is distinguishable but warrants a distinctive trademark form such as "Caduceus Bundle Fabric™" or "Caduceus IPN™".

Filing sequence: all three PPAs in coordinated window, **before** any arXiv defensive preprint, to preserve Paris Convention foreign-filing rights.

---

## 2. Prior-art search report

### 2.1 Search methodology and scope limitations

Searches conducted via web search across:
- Google Patents (USPTO + EPO + WIPO patent literature)
- arXiv (academic preprints)
- IEEE Xplore (academic literature)
- Wikipedia, RFC repository (background standards)

**Searches NOT conducted** (practitioner responsibility):
- Authoritative database searches (Lexis, Westlaw, PatBase, Derwent)
- USPTO PAIR file-wrapper analysis on potentially anticipating patents
- Non-English-language patent literature systematic search
- Trade-secret and unpublished-grant-application discovery

### 2.2 Findings — terrestrial-PKI / Kerberos / authority-system prior art

| Reference | Type | Relevance to claims |
|---|---|---|
| **US6216231B1** ("Specifying security protocols and policy constraints in distributed systems") | Patent | Highly relevant to IP-001. Describes "recent-secure channels" with freshness constraints; revocation services with properties of being "definite, available and recent, contained, adjustable and having bounded recovery". Practitioner must distinguish IP-001 from this on the interplanetary-RTT-specific elements (specifically: dedicated revocation-stream priority lane, succession-implies-revocation rule, enumerated stale-graph defaults table, precedence ordering — none of which appear in US6216231). |
| **US20160065565A1** (Kerberos ticket detection) | Patent | Detection / attack-mitigation patent, not authority-propagation. Adjacent but does not anticipate. |
| **US20210105285A1** (Kerberos ticket forgery detection) | Patent | Detection patent for terrestrial Kerberos. Does not anticipate interplanetary scope. |
| **Kerberos protocol** (RFC 4120) | Standard | Foundational time-bound-ticket prior art. Distinguishes on the system-level combination claimed in IP-001; Kerberos has TTL but lacks dedicated revocation stream, stale-graph defaults, succession-implies-revocation, and interplanetary RTT context. |
| **Burrows-Abadi-Needham logic (1989)** | Academic | Theoretical foundation for authority-time-bounding; not a system or method patent. Cite as background. |
| **X.509 CRL (RFC 5280) and OCSP (RFC 6960)** | Standards | Terrestrial PKI revocation. Distinguishes on interplanetary-RTT-aware design throughout. |

### 2.3 MATERIAL FINDING — KeySpace 2024 (Smailes et al., Oxford)

**arXiv:2408.10963** — "KeySpace: Enhancing Public Key Infrastructure for Interplanetary Networks" by Smailes, Köhler, Birnbach, Strohmeier, Martinovic (University of Oxford and armasuisse Science + Technology), November 2024.

**This is highly relevant prior art for CADUCEUS-IP-001.** It directly addresses PKI for interplanetary networks, including:

- "This paper addresses the challenge of implementing PKI in these complex networks, identifying the essential goals and requirements" — explicitly framed for interplanetary PKI.
- "terrestrial PKI techniques can be adapted for use in highly distributed interplanetary networks, achieving efficient low-latency connection establishment and minimizing the impact of attacks through effective revocation mechanisms" — addresses revocation in interplanetary context.
- "We also establish configurations which optimize performance and security, and propose novel variations on these protocols which provide more effective revocations to protect remote segments more quickly" — proposes novel revocation variations including "OCSP Hybrid" and "relay nodes as a firewall."
- "the predictable links and network topology of interplanetary networks enables many terrestrial PKI concepts to be effective" — counter-thesis to the assumption that interplanetary PKI requires fundamentally new mechanisms.

**Implications for IP-001:**

The broad claim of "interplanetary-RTT-aware authority propagation" as a category is **NOT novel** in light of KeySpace 2024. IP-001's novelty must be tightened to focus specifically on elements NOT addressed in KeySpace:

1. **Stale-graph defaults table** mapping enumerated payload classes to conservative behaviors (KeySpace does not specify this).
2. **Succession-implies-revocation default rule** (KeySpace does not specify this; uses standard OCSP revocation primitives).
3. **Precedence ordering** for multi-class ambiguous bundles (KeySpace does not specify this).
4. **Integration with hash-chained Aletheia provenance** for per-decision audit (KeySpace uses standard X.509, not chain-anchored).
5. **Combination with compliance-graded (Civium-class) authority graph** versus pure X.509 PKI hierarchy.

**Practitioner action required:** revise IP-001 claim language to narrow scope to these specific novel elements. The broad system claim 1 as drafted is at significant anticipation risk. Recommend the practitioner read KeySpace in full (arxiv.org/abs/2408.10963) before refining IP-001.

### 2.4 Findings — DTN key management literature

| Reference | Type | Relevance |
|---|---|---|
| **RFC 4838** (DTN Architecture, 2007) | Standard | Foundational. "This document describes an architecture for delay and disruption-tolerant interoperable networking (DTN). The basis for this architecture lies with that of the Interplanetary Internet, which focused primarily on the issue of deep space communication". Pre-dates Caduceus by ~20 years. Not anticipating; provides substrate. |
| **RFC 9171 / 9172 / 9173 / 9174** (BP7 + BPSec, 2022) | Standards | Caduceus uses these as substrate; not anticipating. BPSec is specifically the security extensions to BP7 that Caduceus B2 uses. |
| **Bhutta et al. 2017** ("Public-key Infrastructure Validation and Revocation Mechanism Suitable for Delay/Disruption Tolerant Networks", IET Information Security) | Academic | Directly addresses PKI validation and revocation for DTN. Practitioner must obtain and distinguish IP-001 from this work. |
| **DICTATE (2005)** | Academic | "DICTATE: DIstributed CerTification Authority with probabilisTic frEshness for Ad Hoc Networks". Probabilistic-freshness model adjacent to time-bound authority. |
| **TruSat (2022)** | Academic | Cyber trust in collaborative spacecraft networks. Adjacent to compliance-graded authority. |
| **Hierarchical Identity-Based Cryptography for DTN** | Academic | "A first contribution to developing a practical cryptosystem for DTNs is based on Hierarchical Identity-Based Cryptography (HIBC). The proposed system is used to create secure channels, to provide mutual authentication, and to allow key revocation". Different cryptographic approach but addresses similar revocation problem. |
| **CCSDS SDLS Blue Book** | Standard | Pre-shared symmetric model. Distinguishes on graph-authority versus symmetric. |

### 2.5 Findings — registry / namespace transition prior art (relevant to IP-002)

No directly anticipating prior art identified in the web search. Adjacent art:
- HTTP version negotiation (HTTP/1.x → HTTP/2 → HTTP/3 transitions) — terrestrial, sub-second.
- TLS handshake version negotiation — terrestrial.
- DNS zone transitions with TTL-based propagation — eventual-consistency, not windowed.
- IPv4/IPv6 dual-stack (RFC 4213) — long-term coexistence, not windowed transition.

Practitioner search should specifically probe: (a) any prior art on cryptographically-anchored protocol-version transitions, (b) any prior art on physics-bounded transition window sizing.

### 2.6 Findings — alert clustering / human confirmation prior art (relevant to IP-003)

Substantial prior art on:
- Clinical alarm management (Joint Commission Sentinel Event Alert #50, 2013) — surface-feature clustering common.
- SIEM/SOC tooling (Splunk, Sumo Logic, etc.) — event-signature clustering common.
- Aviation flight-deck alerting (RTCA DO-178C, ARP-4754A) — rate-limited and prioritized.
- ISA-18.2 industrial control room alarm philosophy.

Practitioner search must distinguish IP-003 from these on the combination of: root-cause clustering (not surface), hard-rate-limit with outright bundle rejection (not just alert suppression), authority-graph-refresh-bounded duration, cryptographic anchoring, and interplanetary RTT operational context.

### 2.7 Overall prior-art conclusions

- **IP-001 (Attack 10):** Material prior art at the broad-claim level (KeySpace 2024). Tighten claim scope to specific novel sub-elements identified in §2.3. **Practitioner action: read KeySpace 2024 in full.**
- **IP-002 (Attack 27):** No anticipating prior art identified at the web-search level. Practitioner search to confirm.
- **IP-003 (F4 clustering):** Substantial adjacent art in clinical alarm and SOC tooling. Claim novelty depends on the specific combination operating in interplanetary RTT crewed-mission context. Practitioner search and § 101 review essential.

---

## 3. Trademark search report (TESS-equivalent)

### 3.1 Search scope and limitations

Searches conducted via web search of public-facing trademark databases (Justia, Trademarkia, TrademarkElite, Furm). **Practitioner must re-run on the official USPTO `tmsearch.uspto.gov` system** before action, including:

- Word mark searches with phonetic variants
- Design code searches for serpent-and-staff designs
- ITU (Intent-to-Use) and pending application searches
- State trademark registry searches
- Common-law trademark searches via general web search

### 3.2 Findings — CADUCEUS marks in IC 9 (computer software)

| Mark | Owner | Class | Status | Goods/Services |
|---|---|---|---|---|
| CADUCEUS MMIS | Caduceus Systems, LLC | IC 9 | CANCELLED 2019 (Section 8) | Computer application software for material management, medical materials inventory |
| CADUCEUS SYSTEMS (stylized + design) | Caduceus Systems, LLC | IC 9 | Registered (#4240004) | Computer software for use with emergency medical computer systems |
| CADUCEUS (mobile app) | Caduceus Occupational Medicine, LLC | IC 9 | Applied (88873092) | Downloadable software in the nature of a mobile application for medical services |
| CADUCEUS (design) | (medical research) | IC 9 | Registered (#2507379) | Computer software for research and reference in the field of medical therapeutics |
| CADUCEUS (design with notepad) | unknown | IC 9 | Registered (#4500472), Section 8 & 15 accepted | Computer software for use in electronic storage of medical data |
| CADUCEUS | POSSIBLE TECH | IC 9 | Filed 1988 | Computerized managing system for use in professional offices |

### 3.3 Findings — CADUCEUS marks in other classes

| Mark | Owner | Class | Notes |
|---|---|---|---|
| CADUCEUS | Caduceus Cellars LLC | IC 25 (clothing) — Registered #3474153 / Registered And Renewed | Wine and apparel; irrelevant class for Visionblox space-tech use. |

### 3.4 Analysis

**Common thread:** all CADUCEUS marks in IC 9 are in **medical or healthcare-related software** contexts. This is consistent with the modern association of the caduceus symbol with medicine in the United States, established in the late 19th and early 20th century as a result of well-documented mistakes and misunderstandings of symbology and classical culture.

**Space-tech use is distinguishable** on the goods/services axis even within IC 9. The TTAB likelihood-of-confusion analysis under *In re E.I. du Pont de Nemours & Co.* (CCPA 1973) considers similarity of goods, channels of trade, conditions of purchase, etc. Visionblox's interplanetary-connectivity software targets aerospace/defense channels distinct from medical software channels.

**Recommended mark form:**

- **Strong:** "Caduceus Bundle Fabric™" or "Caduceus IPN™" — descriptive suffixes establish distinctiveness vs. medical CADUCEUS marks.
- **Acceptable:** "Caduceus by Visionblox™" or "Caduceus / VBX™".
- **Weakest, may face opposition:** bare "Caduceus™" in IC 9.

**Practitioner action:** confirm via `tmsearch.uspto.gov` and TSDR; assess opposition risk; recommend final mark form.

### 3.5 Trademark filing strategy

- ITU (Intent-to-Use) application can be filed concurrent with or after PPA filings; trademark and patent timelines are independent.
- Consider filing in IC 9 (software) + IC 42 (software-as-a-service) + IC 38 (telecommunications) given the comm-fabric nature of the product.
- Foreign trademark protection (Madrid Protocol) parallel to Paris Convention patent foreign-filing window.

---

## 4. Practitioner engagement letter

The following is a draft engagement letter for Khaalis's signature, addressing the practitioner directly. **The practitioner is selected by Khaalis from his existing network or via referral; this letter assumes engagement scope but does not constitute engagement.**

---

> **To:** [Practitioner name, registration number, firm]
>
> **From:** A. Khaalis Wooden, Sr., on behalf of Visionblox LLC
>
> **Re:** IP filing campaign for "Caduceus" — Interplanetary Connectivity Substrate
>
> Dear [Practitioner],
>
> I write to request engagement on a coordinated U.S. Provisional Patent Application (PPA) filing campaign for the "Caduceus" interplanetary connectivity substrate developed by Visionblox LLC in collaboration with Zuup Innovation Lab. Three PPA drafting targets are attached:
>
> 1. **CADUCEUS-IP-001** — Interplanetary-RTT-aware compliance-graded authority propagation (Attack 10).
> 2. **CADUCEUS-IP-002** — URN-scheme-versioned dual-version retention window for interplanetary registry transitions (Attack 27).
> 3. **CADUCEUS-IP-003** — Root-cause clustering with hard rate limit for crew confirmation in high-RTT mission ops (F4).
>
> A companion practitioner brief (CADUCEUS-PRACTITIONER-001) compiles preparatory prior-art findings and a trademark search summary. The brief explicitly flags one material prior-art finding (KeySpace 2024, arXiv:2408.10963 by Smailes et al.) that significantly affects IP-001's claim scope and requires practitioner review before filing.
>
> Source disclosure for all three PPAs is contained in the locked benchmark specification CADUCEUS-004 (caduceus-bench v1.2.1).
>
> Requested scope of engagement:
>
> - **Authoritative prior-art search** on all three subjects via Lexis / Westlaw / PatBase or equivalent.
> - **Claim language refinement** — current claims are first-principles drafts; refinement for technical precision and 35 U.S.C. § 112(b) definiteness compliance.
> - **§ 101 risk assessment** and claim-language mitigation against Alice/Mayo abstract-idea rejection.
> - **Filing strategy advisory** — particularly whether IP-003 should be filed standalone or combined with IP-001.
> - **Sequencing advisory** — PPA filing window vs. arXiv defensive preprint timing, for Paris Convention foreign-filing preservation.
> - **Trademark advisory** — final mark form for "Caduceus" in IC 9 / IC 38 / IC 42, given the existing medical-context CADUCEUS marks documented in the practitioner brief.
> - **Drawings preparation** per 37 C.F.R. § 1.84 for any drawings to be filed with the PPAs.
> - **Filing execution** — preparation and submission of all three PPAs to the USPTO.
>
> Inventor of record: A. Khaalis Wooden, Sr. (sole inventor on present knowledge; co-inventor declarations to be updated if collaborators emerge during practitioner review).
>
> Assignee: Visionblox LLC. Assignment paperwork available on practitioner request.
>
> Anticipated filing window: as soon as practitioner refinement and prior-art search permit. The arXiv defensive preprint is held pending PPA filings.
>
> Please provide engagement-letter terms, estimated fees, and timeline upon review of the attached documents.
>
> Sincerely,
>
> Aldrich K. Wooden, Sr., MBA
> Director of Enterprise Capture & Compliance
> Visionblox LLC

---

## 5. Operational sequencing checklist

For Khaalis's reference, the operational sequence to execute:

| Step | Action | Owner | Dependency |
|---|---|---|---|
| 1 | Identify and engage practitioner | Khaalis | — |
| 2 | Transmit this brief + 3 PPA targets + CADUCEUS-004 to practitioner | Khaalis | Step 1 |
| 3 | Practitioner authoritative prior-art search | Practitioner | Step 2 |
| 4 | Practitioner reviews KeySpace 2024 and revises IP-001 claim scope | Practitioner | Step 3 |
| 5 | Practitioner § 101 review across all three PPAs | Practitioner | Step 4 |
| 6 | Practitioner trademark final-form recommendation | Practitioner | Step 4 |
| 7 | Visionblox signs off on practitioner-revised claims | Khaalis | Step 5 |
| 8 | Practitioner files PPAs (USPTO + possibly Paris Convention members concurrently) | Practitioner | Step 7 |
| 9 | LEDGER #0004 commit on CADUCEUS-004 (caduceus-bench v1.2.1) | Khaalis / Zuup | Step 8 (sequencing optional but recommended for cleaner chain anchoring) |
| 10 | Trademark ITU application filed | Practitioner | Step 6, parallel to Steps 7–9 |
| 11 | arXiv defensive preprint published | Khaalis / Zuup | **After Step 8** to preserve foreign-filing rights |
| 12 | Begin Phase 5 substrate build with Mercury kernel formal-methods composition | Khaalis / Zuup | Independent of IP track |

**Critical sequencing constraint:** Step 11 (arXiv preprint) must follow Step 8 (PPA filing) by at least 1 business day to provide clean priority establishment under 35 U.S.C. § 102 grace-period analysis (12-month US grace; some foreign jurisdictions have no grace and require pre-disclosure filing).

---

## 6. Honest disclosure obligations to USPTO

Once any PPA is filed, the inventor and any practitioner of record are subject to the duty of candor under 37 C.F.R. § 1.56. This duty includes disclosing material prior art known to them.

The KeySpace 2024 finding (§2.3 of this brief) is material to IP-001 and must be disclosed via an Information Disclosure Statement (IDS) when the PPA is converted to a non-provisional within the 12-month window. Practitioner will manage this filing.

Visionblox should also disclose to the practitioner:
- All AI-system-assisted preparation (this brief, the PPA targets, the prior CADUCEUS-001 through CADUCEUS-004 specifications) per recent USPTO guidance on AI-assisted inventorship (2024 guidance documents).
- Any inventor contributions beyond Khaalis that should be evaluated for co-inventorship declarations.

---

## 7. Honest scope statement (recap)

- This brief is preparation material prepared by Claude (Anthropic). It does not substitute for the practitioner's authoritative work.
- The KeySpace 2024 finding is material and changes the IP-001 novelty position. Practitioner must address before filing.
- All three PPA targets are drafting targets, not filing-ready papers.
- The trademark recommendation (use a distinctive descriptor with "Caduceus") is advisory; practitioner gives the binding recommendation after authoritative TESS / TSDR search.
- LEDGER #0004 commit on CADUCEUS-004 (caduceus-bench v1.2.1) is independent of the IP track and can proceed on Khaalis's confirmation.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-PRACTITIONER-001 Rev A · Practitioner engagement brief*
*Compiled by Claude (Anthropic) as preparation material; not authoritative legal work product*
