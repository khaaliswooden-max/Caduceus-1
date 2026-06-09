# Visionblox LLC — CMMC Level 2 Self-Assessment Completion

| | |
|---|---|
| **Document ID** | CADUCEUS-CMMC-001 |
| **Revision** | A (framework — Visionblox completes the actual attestations) |
| **Purpose** | Framework to finalize the CMMC L2 self-assessment that is in motion per project context; gating document for federal channels requiring CUI handling |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Framework — section by section structure; Visionblox completes evidence and attestation per control |

---

## 0. Honest scope statement

Cybersecurity Maturity Model Certification (CMMC) Level 2 corresponds to implementation of the 110 controls defined in NIST SP 800-171 Revision 2, plus organizational maturity practices. This document provides the **structural framework** for completing Visionblox's L2 self-assessment. The actual attestation requires Visionblox to confirm, on each control, that the implementation is in place and produce supporting evidence. Claude cannot attest on Visionblox's behalf.

CMMC L2 self-assessment is sufficient for contracts at certain CUI levels; higher CUI tiers require third-party assessment by a Cybersecurity Maturity Model Certification Third-Party Assessment Organization (C3PAO). The threshold is contract-specific; the DoD Solicitation will specify whether self-assessment or C3PAO assessment is required.

---

## 1. CMMC L2 control family overview

CMMC L2 organizes the 110 NIST SP 800-171 R2 controls into 14 control families. Visionblox's self-assessment must address each family.

| Family | Code | Control count | Caduceus-relevant emphasis |
|---|---|---|---|
| Access Control | AC | 22 | RBAC on PHRONESIS, Civium authority graph |
| Awareness & Training | AT | 3 | Crew + ground operator training |
| Audit & Accountability | AU | 9 | Aletheia chain provenance |
| Configuration Management | CM | 9 | Cryptographically-committed manifests |
| Identification & Authentication | IA | 11 | Ed25519 + Civium graph |
| Incident Response | IR | 3 | HF-9 fail-safe + ground notification |
| Maintenance | MA | 6 | Substrate-side maintenance |
| Media Protection | MP | 9 | Secure Element + Bus ISO |
| Personnel Security | PS | 2 | Cleared-personnel scope |
| Physical Protection | PE | 6 | Suit + ground hardware enclosure |
| Risk Assessment | RA | 3 | ZCS-6 falsification-first |
| Security Assessment | CA | 4 | caduceus-bench v1.2.1 compliance |
| System & Communications Protection | SC | 16 | Caduceus IS this |
| System & Information Integrity | SI | 7 | T1, T4, T6, T7 enforcement |
| **Total** | | **110** | |

---

## 2. Self-assessment structure per control

For each of the 110 controls, Visionblox must produce:

1. **Implementation status:** Implemented / Partially Implemented / Planned / Not Applicable
2. **Implementation description:** how the control is satisfied (1–3 sentences)
3. **Evidence pointer:** file path, document ID, or system reference demonstrating the control
4. **Responsible party:** named individual (or role) responsible for the control
5. **Last verification date:** TAI epoch of most recent verification

Each control's record gets a SHA-256 hash for chain anchoring (per Visionblox's general practice).

---

## 3. Critical-path control families for Caduceus federal capture

### 3.1 System & Communications Protection (SC) — 16 controls

Caduceus IS a system & communications protection capability. Most SC controls are directly satisfied by the Caduceus architecture itself. Sample mapping:

| Control | Caduceus mechanism |
|---|---|
| SC.L2-3.13.1 (Monitor, control, protect communications at boundaries) | HF-16 outbound attestation + HF-17 inbound gate |
| SC.L2-3.13.5 (Subnetwork isolation) | Per-stream nonce-set partitioning (T4) |
| SC.L2-3.13.8 (Implement cryptographic mechanisms during transmission) | T1 Ed25519 + B2 BPSec |
| SC.L2-3.13.11 (FIPS-validated cryptography) | libsodium with FIPS-compliant configuration |
| SC.L2-3.13.16 (Protect confidentiality at rest) | Bus ISO + Secure Element storage |

### 3.2 Audit & Accountability (AU) — 9 controls

The Aletheia hash-chained provenance ledger directly satisfies AU controls:

| Control | Caduceus mechanism |
|---|---|
| AU.L2-3.3.1 (Create and retain system audit logs) | Every bundle event in Aletheia chain |
| AU.L2-3.3.2 (Ensure unique user identification for audit) | Civium graph + per-user Ed25519 keys |
| AU.L2-3.3.4 (Alert if logging process failure) | HF-9 fail-safe triggers on chain write failure |
| AU.L2-3.3.8 (Protect audit information from unauthorized access) | Secure Element-stored Aletheia chain |
| AU.L2-3.3.9 (Limit audit log management to subset of users) | Civium authority gate on audit-write authority |

### 3.3 System & Information Integrity (SI) — 7 controls

Caduceus's T1, T4, T6, T7 directly satisfy several SI controls:

| Control | Caduceus mechanism |
|---|---|
| SI.L2-3.14.1 (Identify, report, correct flaws) | Phase 3 adversarial sweeps; versioned re-commits |
| SI.L2-3.14.2 (Protect against malicious code) | Bundle-level signature + class enumeration (T1+F2) |
| SI.L2-3.14.3 (Monitor security alerts and advisories) | ACI drift monitoring (T5) + degraded-replay alerts (T4) |
| SI.L2-3.14.7 (Identify unauthorized use of organizational systems) | Civium authority graph violations logged to chain |

### 3.4 Configuration Management (CM) — 9 controls

The ZCS-6 cryptographic-commit discipline directly satisfies CM controls:

| Control | Caduceus mechanism |
|---|---|
| CM.L2-3.4.1 (Establish baseline configurations and inventories) | Bench manifests committed to Aletheia chain |
| CM.L2-3.4.2 (Enforce security configuration settings) | §3.7 theoretical-floor constants signed with release key |
| CM.L2-3.4.3 (Track changes to systems) | LEDGER chain provides immutable change history |
| CM.L2-3.4.6 (Employ principle of least functionality) | F2 closed class namespace |
| CM.L2-3.4.9 (Control and monitor user-installed software) | Civium gate on installation authority |

---

## 4. Plan of Action and Milestones (POA&M) structure

For any control marked Partially Implemented or Planned, Visionblox must produce a POA&M entry:

```
POA&M Entry #<N>
  Control ID: <e.g., AC.L2-3.1.5>
  Current status: Partially Implemented / Planned
  Description of gap: <what is missing>
  Compensating control: <interim mitigation if any>
  Remediation plan: <what will be done>
  Resources required: <effort, cost>
  Target completion date: <TAI epoch>
  Responsible party: <name or role>
```

POA&M entries are themselves audit records and should be committed to the Aletheia chain alongside the self-assessment.

---

## 5. Visionblox-specific posture areas (likely Partial / Planned)

Based on the small-business, new-entrant posture documented in CADUCEUS-CAPTURE-001 §6, several controls likely require POA&M entries:

| Likely Partial / Planned | Rationale | Suggested compensating control |
|---|---|---|
| PS.L2-3.9.1 (Screen individuals) | New entrant; formal background-check program may not be fully implemented | Document interim screening via business attestation; complete formal program before any cleared personnel hire |
| PE.L2-3.10.1 (Limit physical access) | Small-team home-office posture | Document physical-protection practices; upgrade to facility-class controls before any Secret-level work |
| MA.L2-3.7.1 (Perform maintenance) | Maintenance program may not be formalized | Establish documented maintenance schedule; commit to chain |
| IR.L2-3.6.3 (Test incident response capability) | Likely informal | Schedule tabletop exercise; document outcomes |
| AT.L2-3.2.1 (Awareness training) | Small team; formal program may not exist | Establish quarterly awareness session; document attendance |

---

## 6. Self-attestation language template

After all 110 controls are addressed in the self-assessment:

```
SELF-ATTESTATION OF CMMC LEVEL 2 IMPLEMENTATION

I, A. Khaalis Wooden, Sr., as the authorized representative of Visionblox LLC,
attest under penalty of perjury that the foregoing CMMC Level 2 self-assessment
accurately represents the implementation status of all 110 controls defined in
NIST SP 800-171 Revision 2 as of <TAI date>. Controls marked Implemented have
corresponding evidence on file at Visionblox. Controls marked Partially
Implemented or Planned have POA&M entries with target remediation dates.
Controls marked Not Applicable have documented justifications.

This self-assessment is current as of the date above and will be re-verified
at least annually, or upon material change to Visionblox's systems or
operations, whichever occurs first.

Signature:        ____________________________
Printed name:     A. Khaalis Wooden, Sr.
Title:            Director of Enterprise Capture & Compliance
Date:             ____________________________
SPRS posting:     ____________________________
```

The self-attestation is filed with the Supplier Performance Risk System (SPRS) and produces a score that DoD solicitations may reference.

---

## 7. Operational steps to finalize

Sequential steps for Visionblox to complete the self-assessment:

1. **Inventory all 110 controls** in a structured spreadsheet or assessment tool.
2. **For each control, populate** the five fields from §2 (status, description, evidence, responsible party, last verification date).
3. **Identify Partial / Planned controls** and create POA&M entries per §4.
4. **Map Caduceus-relevant controls** explicitly to caduceus-bench v1.2.1 assertions per §3 (this strengthens the assessment by tying compliance to a cryptographically-committed reference).
5. **Conduct internal review** before attestation; ideally with an external advisor familiar with NIST SP 800-171.
6. **Sign the self-attestation** per §6.
7. **Post score to SPRS** at sprs.csd.disa.mil.
8. **Commit the assessment artifact** to the Aletheia chain (compute SHA-256, sign, append).

Estimated effort: 40–80 hours of focused work for a single-person small business with existing partial implementation, distributed across one to two weeks.

---

## 8. Post-completion considerations

After self-attestation:

- **Annual re-attestation** is required per CMMC rule.
- **Continuous monitoring** practices should be established for each implemented control.
- **C3PAO assessment** may be required for specific contracts handling certain CUI tiers; self-attestation does not satisfy all contract requirements.
- **CMMC Level 3** is the next tier; requires additional 24 controls plus expert-led assessment. Not gating for most current Visionblox channels but a future consideration for larger contracts.

---

## 9. Honest scope statement

- This document provides the framework; Visionblox completes the per-control evidence and attestation.
- Many controls require documentation that exists elsewhere (HR records, facility lease, software inventories, etc.); the assessment is the consolidation, not the creation, of those records.
- A genuine self-assessment is not paperwork — it is the operational discipline of running the controls. Visionblox should establish the disciplines, not just the documentation.
- The self-attestation is a federal legal instrument; falsification carries serious consequences. Honest acknowledgment of Partial / Planned controls via POA&M is the correct posture, not over-claiming.
- A small business with substantial methodology rigor (ZCS-6, cryptographic commit discipline, Aletheia chain) is well-positioned for L2 self-attestation because most controls have direct technical implementations; the gap is typically administrative (training programs, formal procedures) rather than technical.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-CMMC-001 Rev A · CMMC L2 self-assessment completion framework*
