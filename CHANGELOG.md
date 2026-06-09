# Changelog

All notable changes to Caduceus are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
adapted for a methodology-and-specification repository (versions correspond to
caduceus-bench releases, not software releases).

Cryptographic commit anchors (Aletheia LEDGER entries) are listed for each release.

---

## [Unreleased]

### Planned
- LEDGER #0004 commit ceremony (after PPA filings complete)
- arXiv defensive preprint publication (after LEDGER commit)
- Aletheia v0.3 implementation and aletheia-bench v1.0 commit as LEDGER #0005
- Substrate v0.2 build per CADUCEUS-005 (20 weeks)
- Phase 6 attack sweep execution at substrate M5–M8

---

## [v1.2.1] — 2026-06-08 — caduceus-bench v1.2.1

**Status:** Commit-ready manifest. LEDGER #0004 pending operational sequencing.

**Manifest hash (SHA-256):** `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874`

**Source manifest:** [docs/CADUCEUS-004.md](docs/CADUCEUS-004.md)

### Added
- §3.7 — theoretical-floor derivation procedure (T5 enablement: SNR, BER, RTT first-principles derivation from link physics)
- §3.8 — F4 K calibration guidance (advisory, not normative)
- Inlined HF-9 normative behavior in T3 sub-points (a)–(g), making CADUCEUS-004 a self-contained commit manifest
- B2 BPSec key derivation via HKDF (RFC 5869) from Aletheia parent signing keys, explicitly specified
- F4 audit logging requirement: every confirmation event (grant / deny / timeout) recorded as Aletheia chain entry with full bundle context
- §0 single-fault-tolerant scope statement with three named v0.3 substrate mechanisms required for dual-fault-tolerance

### Changed
- T3 fail-safe behavior — inlined from VBX-ISPS reference, no longer requires external reference
- T5 theoretical-floor — now explicitly derived per §3.7 procedure rather than abstract reference
- B2 — BPSec session key derivation now specified at enablement grade
- F4 — confirmation events now require 100% Aletheia chain audit trail completeness

### Closed open items from v1.2
- #13 — BPSec key-rooting in Aletheia infrastructure (now explicit via HKDF)
- partial closure of #19 — theoretical-floor derivation procedure now specified

### New open items
- #19 — per-link-configuration theoretical-floor constants must be derived and registered before v0.1 substrate operation
- #20 — HF-9 exit two-source confirmation cryptographic protocol detail (v0.2 substrate work)
- #21 — empirical F4 K calibration against EVA cognitive-load data (mission-policy track, not bench track)

---

## [v1.2] — 2026-06-08 — caduceus-bench v1.2

**Source manifest:** CADUCEUS-003

### Added
- 5 new assertions (T4 per-stream quotas with degraded-replay alert; T5 theoretical-floor binding; T6 ingress-validity ordering; T7 succession-implies-revocation; F4 root-cause clustering with hard rate limit)
- §3.5 — Caduceus-BodyID v1.0 URN namespace with dual-version retention window for interplanetary registry transitions
- §3.6 — conservative-defaults table mapping payload classes to behavior under stale authority graphs, with explicit precedence ordering
- T1 — atomicity clause for the four-check verification sequence
- F2 — closed class namespace (unenumerated payload classes rejected)

### Changed
- 7 assertions revised from v1.1 with tightened thresholds and behaviors
- 5 assertions unchanged from v1.1

### Closed Phase 3 attacks
- All 12 attacks from CADUCEUS-PHASE3-001 round 1 (against v1.0)
- All 8 attacks from CADUCEUS-PHASE3-002 round 2 (against v1.1)
- Total: 20 named attacks closed

---

## [v1.1] — 2026-06-08 — caduceus-bench v1.1

**Manifest hash prefix:** `58bf8744...`

**Source manifest:** CADUCEUS-002

### Added
- T4 (initial form), T5, T6, T7, F4 — first introductions of these assertion classes
- Expanded §3.5 namespace specification
- Expanded §0 scope additions

### Changed
- Revisions to T1–T3 and F1–F3 from v1.0 based on Phase 3 round 1 findings

### Closed open items from v1.0
- 6 gaps identified in CADUCEUS-PHASE3-001 sweep

---

## [v1.0] — 2026-06-08 — caduceus-bench v1.0

**Source manifest:** CADUCEUS-001

### Added
- Initial benchmark specification with 12 normative assertions across four layers (P, B, T, F)
- Six-attribute whitespace map (§1)
- Initial bench-vs-substrate ordering discipline
- Initial test-harness sketch

---

## Document additions outside bench-version commits

### [2026-06-08] — Substrate and methodology documents

- **CADUCEUS-005** — Phase 5 substrate planning document (M0–M8, ~20 weeks, ~3.25 FTE, ~$650K)
- **CADUCEUS-006** — Mercury × Caduceus formal-methods composition spec (~7.25 person-months for Lean 4 + TLA+)
- **CADUCEUS-007** — Phase 6 vertical-integration attack-sweep plan (33 attacks across 8 categories)

### [2026-06-08] — Intellectual property documents

- **CADUCEUS-IP-001** — Attack 10 PPA target (interplanetary-RTT-aware authority propagation)
- **CADUCEUS-IP-002** — Attack 27 PPA target (URN-scheme-versioned dual-version retention window)
- **CADUCEUS-IP-003** — F4 PPA target (root-cause clustering + hard rate limit)
- **CADUCEUS-PRACTITIONER-001** — Combined practitioner brief with prior-art search, TESS-equivalent trademark search, engagement letter
- **CADUCEUS-LEDGER-PREP-001** — LEDGER #0004 commit prep with cryptographic artifacts and signing operation sequence

### [2026-06-08] — Federal capture and compliance documents

- **CADUCEUS-CAPTURE-001** — Federal capture posture across 8 channels
- **CADUCEUS-CMMC-001** — CMMC L2 self-assessment completion framework
- **CADUCEUS-NSTXL-001** — NSTXL membership application briefing
- **CADUCEUS-SBIR-NASA-001** — NASA SBIR Phase I proposal skeleton for SCaN Cognitive Communication

### [2026-06-08] — Publication artifacts

- **Caduceus_IEEE_Paper.tex** + **references.bib** — IEEEtran-formatted 8-page conference paper with 30+ citations
- **CADUCEUS-ARXIV-001** — arXiv defensive preprint submission package (same content as IEEE paper)
- **CADUCEUS-BENCH-001** — Consolidated external-facing benchmark document

### [2026-06-08] — Parent-system extensions

- **ALETHEIA-V03-001** — Aletheia v0.3 cross-key-epoch chain extension spec (6 new assertions, ~2–3 person-weeks implementation)

### [2026-06-09] — Architecture artifacts

- **DWG CADUCEUS-001** — Caduceus system architecture blueprint (SVG + PNG)
- **DWG CADUCEUS-002** — Caduceus deployment topology blueprint (SVG + PNG)

### [2026-06-09] — Repository governance

- Initial README, LICENSE (Apache-2.0), CONTRIBUTING, CHANGELOG (this file), ROADMAP, SECURITY, CODE_OF_CONDUCT, CITATION.cff, GOVERNANCE, CLAUDE.md

---

## Predecessor chain anchors

For chain continuity and audit traceability:

- **LEDGER #0001** — VBX-ISPS-BENCH v1.0
- **LEDGER #0002** — VBX-ISPS-BENCH v1.1 (SHA-256 prefix `58bf8744...`)
- **LEDGER #0003** — VBX-ISPS substrate v0.1 (31 artifacts hashed)
- **LEDGER #0004** — caduceus-bench v1.2.1 (PENDING; this release)
- **LEDGER #0005** — (planned) Aletheia v0.3 spec commit

---

## Version conventions

- **v1.x.y** numbering applies to caduceus-bench specifications.
- **v0.x** numbering applies to substrate prototypes (CADUCEUS-005 v0.2 plan, v0.3 dual-fault-tolerance roadmap).
- Major version increment (v2.0) indicates non-backward-compatible bench changes (architectural redesign, not tightening).
- Minor version increment (v1.3) indicates additive changes (new assertions, expanded scope).
- Patch version increment (v1.2.1) indicates enablement / clarity tightenings without behavior changes.

Per ZCS-6 discipline, all bench versions are cryptographically committed via Aletheia LEDGER entries; the chain provides the immutable audit trail.

---

*Maintainer: A. Khaalis Wooden, Sr., MBA — Visionblox LLC / Zuup Innovation Lab*
