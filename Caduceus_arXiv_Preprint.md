# arXiv Defensive Preprint — Submission Package

| | |
|---|---|
| **Document ID** | CADUCEUS-ARXIV-001 |
| **Revision** | A |
| **Source paper** | `Caduceus_IEEE_Paper.tex` + `references.bib` (this directory) |
| **Purpose** | Submission metadata and operational instructions for arXiv defensive preprint of the Caduceus IEEE paper |
| **Critical timing constraint** | **DO NOT SUBMIT BEFORE PPA FILINGS COMPLETE.** arXiv publication is a public-disclosure event; pre-PPA disclosure forfeits Paris Convention foreign-filing rights in non-grace-period jurisdictions. |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |

---

## 0. The content is identical

The arXiv preprint and the IEEE paper draft are the same content. The LaTeX source is `Caduceus_IEEE_Paper.tex`; the references file is `references.bib`. arXiv accepts IEEEtran-formatted papers without modification.

This document provides the arXiv-specific submission metadata and the operational sequence. The actual paper content is in the LaTeX file.

---

## 1. arXiv metadata

### 1.1 Title

> Caduceus: A Compliance-Graded Bundle Fabric for Interplanetary Authority Propagation

### 1.2 Authors

> Aldrich K. Wooden, Sr. (Southern New Hampshire University; Visionblox LLC; Zuup Innovation Lab)

### 1.3 Abstract (for arXiv listing — same as paper abstract)

Interplanetary missions face a propagation-window failure mode unique to communication channels where round-trip light-time exceeds the desired authority-decision cutover time. Existing public-key infrastructure assumes sub-second revocation propagation; standardized space-data link security mechanisms use pre-shared symmetric keys without graph-authority semantics. Recent work on PKI for interplanetary networks demonstrates that terrestrial concepts can be adapted but does not address compliance-graded authority decisions with explicit accommodation for propagation windows of minutes to hours.

We present Caduceus, a compliance-graded delay-tolerant bundle fabric for interplanetary authority propagation. Caduceus sits above Bundle Protocol v7 (RFC 9171) and the Bundle Protocol Security extensions (RFC 9172), providing per-bundle Aletheia-class cryptographic provenance, Civium-graph authority gating, drift-aware link-integrity monitoring, frame-disambiguated timestamps, and a crew-attestation peer model. We define caduceus-bench v1.2.1, a falsifiable benchmark of seventeen normative assertions across four layers, cryptographically committed via the ZCS-6 ordering discipline before any substrate implementation. We document a two-round adversarial sweep surfacing twenty attacks against the bench, including two interplanetary-specific failure modes not addressable by terrestrial mechanisms. We outline a substrate prototype plan, a formal-methods composition with the Mercury Subleq kernel, and honest scope statements on what the bench does and does not measure.

### 1.4 arXiv subject classifications

Primary: `cs.NI` (Networking and Internet Architecture)

Secondary:
- `cs.CR` (Cryptography and Security)
- `cs.DC` (Distributed, Parallel, and Cluster Computing)
- `cs.LO` (Logic in Computer Science) — for the formal-methods composition aspect

### 1.5 Comments (optional arXiv field)

> 8 pages, 4 tables, 1 figure. Includes caduceus-bench v1.2.1 specification anchored to Aletheia LEDGER #0004 (SHA-256 0fe3b32db5522db6...). Companion documents include consolidated benchmark (CADUCEUS-BENCH-001), substrate plan (CADUCEUS-005), and formal-methods composition (CADUCEUS-006).

### 1.6 License

Recommended: `arXiv perpetual non-exclusive license` (default). Allows arXiv to display and distribute; Visionblox retains copyright; downstream readers can read but not redistribute commercially without permission.

Alternative: CC BY 4.0 — broader open access; consider only if commercial-licensing posture is no longer a constraint for Visionblox.

**Recommendation:** start with the arXiv non-exclusive license; can be updated later.

---

## 2. Submission package contents

The following files comprise the arXiv submission:

| File | Purpose |
|---|---|
| `Caduceus_IEEE_Paper.tex` | Main paper source (IEEEtran format) |
| `references.bib` | BibTeX bibliography |
| `IEEEtran.cls` | IEEEtran class file (must be in submission directory if not standard) |
| `IEEEtran.bst` | IEEE bibliography style |
| (optional) `caduceus_stack.pdf` | Pre-rendered figure for the Caduceus layered stack (if not generated via TikZ inline) |

arXiv accepts a tar.gz of these files. Submitter uploads, arXiv compiles via TeXLive, returns rendered PDF for review.

---

## 3. Operational submission sequence

**Critical: this sequence must be followed in order. See §4 for rationale.**

| Step | Action | Owner | Status |
|---|---|---|---|
| 1 | Practitioner engagement complete; all three PPAs filed (CADUCEUS-IP-001/002/003) | Khaalis + practitioner | **GATING** |
| 2 | LEDGER #0004 cryptographic commit on CADUCEUS-004 | Khaalis | After Step 1 |
| 3 | Wait 1 business day for ledger anchor confirmation | — | — |
| 4 | Final review of LaTeX paper for any final corrections | Khaalis | After Step 3 |
| 5 | Create arXiv submitter account if not existing | Khaalis | Any time |
| 6 | Build submission tarball: `tar -czf caduceus_arxiv.tar.gz *.tex *.bib *.cls *.bst` | Khaalis | After Step 4 |
| 7 | Submit via arXiv.org submission interface; specify metadata per §1 | Khaalis | After Step 6 |
| 8 | arXiv compilation review (24–48 hours typically) | arXiv moderators | — |
| 9 | Publish | arXiv | — |
| 10 | Cross-link from Visionblox public repo and from LEDGER #0004 ledger entry | Khaalis | After Step 9 |

Total elapsed time from Step 1 to Step 9: approximately 2 weeks (most of which is PPA preparation by the practitioner).

---

## 4. Why the sequencing matters

Public disclosure of an invention starts the foreign-filing clock under the Paris Convention. The United States provides a 12-month grace period under 35 U.S.C. § 102(b)(1) for the inventor's own disclosures; most non-U.S. jurisdictions do not.

Filing a U.S. Provisional Patent Application (PPA) before public disclosure preserves the priority date and the 12-month Paris Convention foreign-filing right. arXiv publication is a "publication" event in patent law; it starts the clock in non-U.S. jurisdictions immediately.

The correct sequence:
1. **PPA filed** → priority date locked; 12-month Paris window starts
2. **arXiv published** → public-disclosure event; 12-month U.S. grace period also starts (in parallel)
3. **Foreign filings or PCT application** → must be filed within 12 months of PPA filing (per Paris Convention) AND within the relevant grace periods of target jurisdictions

If arXiv publication precedes PPA filing, the inventor loses foreign-filing rights in any non-grace-period jurisdiction. The harm is irreversible.

This is the single most common procedural error inventors make. The sequence above prevents it.

---

## 5. Post-publication actions

After arXiv publishes:

1. **Add the arXiv identifier to the Aletheia LEDGER #0004 entry** as a cross-reference (this can be done via a follow-on chain entry).
2. **Notify the practitioner** so the IDS (Information Disclosure Statement) on the PPA can reference the published preprint if needed.
3. **Update Visionblox public-facing materials** (website, LinkedIn, capability statement) with the arXiv link.
4. **Submit to IEEE conference** (target: IEEE Aerospace Conference or IEEE/ACM ICCPS per project memory) using the same LaTeX source; arXiv preprint is not a bar to conference submission for these venues.
5. **Engage with KeySpace authors at Oxford** if appropriate; the Caduceus work is complementary to their PKI work and an academic dialog may be valuable.
6. **Open the GitHub repository** for the proof artifacts (per CADUCEUS-006 §8 publication strategy) once Phase 5 substrate work begins.

---

## 6. arXiv etiquette and best practices

- **Title:** keep as-is — descriptive and category-honest.
- **Abstract:** keep under 1920 characters per arXiv limit (current draft is ~1800).
- **References:** ensure all are in BibTeX; arXiv compilation breaks on malformed entries.
- **Cross-references:** all `\cite{}` keys must match entries in `references.bib`.
- **Figures:** if TikZ figures fail to compile on arXiv, generate PDF locally and include as `\includegraphics`.
- **License:** confirm the chosen license before submission; cannot be downgraded once published.
- **Author affiliations:** match exactly between paper and arXiv metadata.

---

## 7. Honest scope statement

- This document is operational guidance for submission. The actual submission is performed by Khaalis using an arXiv submitter account.
- The LaTeX paper has been drafted to be self-sufficient; minor revisions may be advised by reviewers or the practitioner before submission.
- arXiv is not peer-reviewed; the preprint serves prior-art anchoring and academic visibility. Subsequent IEEE conference / journal submission provides peer review.
- The sequencing constraint in §3 and §4 is not optional. Pre-PPA arXiv publication forfeits foreign-filing rights and cannot be reversed.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-ARXIV-001 Rev A · arXiv defensive preprint submission package*
