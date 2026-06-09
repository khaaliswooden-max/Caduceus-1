# Caduceus Roadmap

This document describes the forward plan for Caduceus across three time horizons.
Items are organized by dependency rather than calendar date, since federal-capture
timing and PPA filing sequences gate several downstream milestones.

For historical work completed, see [CHANGELOG.md](CHANGELOG.md).

---

## Near horizon (next 6 months)

### Sequencing-critical operational items

These items are gated by external action and must complete in order:

| # | Action | Owner | Gates |
|---|---|---|---|
| 1 | Engage USPTO-registered patent practitioner | Visionblox | — |
| 2 | File PPAs (CADUCEUS-IP-001 / -002 / -003) | Practitioner | After #1 |
| 3 | Execute LEDGER #0004 commit ceremony on CADUCEUS-004 | Visionblox (key custodian) | After #2; wait 1 business day for ledger anchor |
| 4 | OpenTimestamps proof on signed ledger entry | Visionblox | After #3 |
| 5 | Publish arXiv defensive preprint | Visionblox | After #4 |
| 6 | Submit IEEE conference paper (target: IEEE Aerospace or IEEE/ACM ICCPS) | Visionblox | Parallel to #5 |

**Critical:** Step 5 (arXiv) must NOT precede Step 2 (PPA filings). Pre-PPA disclosure forfeits Paris Convention foreign-filing rights. See [docs/CADUCEUS-PRACTITIONER-001.md](docs/CADUCEUS-PRACTITIONER-001.md) §5.

### Federal capture (parallel track, no gating to commit)

| # | Action | Owner | Reference |
|---|---|---|---|
| 7 | Apply for NSTXL/SOSSEC consortium membership ($500–$1,500) | Visionblox | [docs/CADUCEUS-NSTXL-001.md](docs/CADUCEUS-NSTXL-001.md) |
| 8 | Complete CMMC L2 self-assessment; post to SPRS | Visionblox | [docs/CADUCEUS-CMMC-001.md](docs/CADUCEUS-CMMC-001.md) |
| 9 | Subscribe to DIU CSO mailing list; monitor quarterly opportunities | Visionblox | [docs/CADUCEUS-CAPTURE-001.md](docs/CADUCEUS-CAPTURE-001.md) §3.7 |
| 10 | Assess Navy SBIR DON26BZ03-NV062 pursuit (deadline July 22, 2026) | Visionblox | §3.4; requires cleared teaming partner |

### Parent-system extensions (no gating to commit)

| # | Action | Owner | Reference |
|---|---|---|---|
| 11 | Implement Aletheia v0.3 chain extension (cross-key-epoch verification) | Visionblox / Zuup | [docs/ALETHEIA-V03-001.md](docs/ALETHEIA-V03-001.md) — 2–3 person-weeks |
| 12 | Commit Aletheia v0.3 spec as LEDGER #0005 (after #11 implementation) | Visionblox | After #11 |

---

## Medium horizon (6 to 18 months)

### Substrate v0.2 build per CADUCEUS-005

Approximate 20-week build, ~3.25 FTE, ~$650K. Begins after operational sequencing items 1–6 above complete.

| Milestone | Weeks | Deliverables |
|---|---|---|
| **M0** | 0 | LEDGER commit + PPAs filed + arXiv published |
| **M1** | 1–3 | Aletheia v0.3 chain extensions active; Civium revocation-stream extension; theoretical-floor constants per §3.7 for at least one link configuration |
| **M2** | 3–6 | HF-16, HF-19, HF-21 modules implemented and unit-tested |
| **M3** | 5–11 | HF-17, HF-18 modules implemented; Phase 3 attack-suite executed against modules with 100% closure |
| **M4** | 9–13 | HF-20 Mercury kernel integration; comm-peak/compute-peak interlock |
| **M5** | 7–13 | EPHEMERIS peer firmware; UWB/BLE pairing |
| **M6** | 11–15 | HF-22 implementation; 90-day adversarial flood simulation passes |
| **M7** | 14–16 | Full-stack integration test; **Phase 6 main execution begins** |
| **M8** | 17–20 | Phase 6 final sweep; substrate v0.2 → v0.3 iteration if Phase 6 findings warrant |

### Formal-methods composition per CADUCEUS-006

Parallel to substrate build, ~7.25 person-months total over ~6 calendar months at 1.2 FTE.

| Track | Effort | Deliverable |
|---|---|---|
| Lean 4 proofs | 4 person-months | T1 atomicity, T3 fail-safe, T7 succession, F4 rate-limit, §3.6 precedence, T6 ingress-validity ordering — machine-checked |
| TLA+ model checks | 2.5 person-months | T4 quotas, T5 floor convergence, F2+F4 interaction, §3.5 dual-version window — model-checked |
| Composition glue with Mercury kernel | 0.75 person-months | Mercury theorems imported as lemmas; Caduceus theorems compose |

### Phase 6 attack sweep per CADUCEUS-007

33 named attacks across 8 categories. Runs during substrate M5–M8 with continuous execution thereafter.

| Category | Priority | When |
|---|---|---|
| B (Composition) | 1 (highest) | Substrate M6–M7 |
| E (Emergent — long-duration) | 2 | Substrate M5 onwards (start early for 90-day cycles) |
| D (VBX-ISPS integration) | 3 | Substrate M6–M7 |
| C (Resource exhaustion) | 4 | Substrate M7–M8 |
| H (Failure-mode) | 5 | Substrate M7–M8 |
| A (Implementation) | 6 | Substrate M7 |
| F (Supply chain) | 7 | Substrate M8+ |
| G (Side channels) | 8 | Deferrable to v0.3 |

### Federal capture milestones

| Action | Window |
|---|---|
| NASA SBIR FY27 Phase I — SCaN Cognitive Communication subtopic submission | Q2 2027 (anticipated solicitation March–May 2027) |
| SDA Tranche 3 GEP — prime BD outreach for sub-teaming | Q4 2026 – Q1 2027 |
| AFRL OTAFI — pre-proposal via SOSSEC | After NSTXL membership active |
| AFWERX SBIR Open Topic | Quarterly review |

---

## Long horizon (18+ months)

### Substrate v0.3 — dual-fault-tolerance for crewed deployment

Per CADUCEUS-004 §0, v0.3 requires three substrate-side DFT mechanisms not in v0.2:

1. Dual-peer EPHEMERIS attestation (two crew EPHEMERIS units agree before high-stakes ops)
2. Triple-redundant Aletheia chain storage (Orin NX flash + Secure Element + EPHEMERIS-side cache, quorum-recovery)
3. Authority-graph redundant storage (triple-redundant with quorum-verification)

Estimated effort: ~30 weeks build, ~4–5 FTE. Funding likely via NASA SBIR Phase II Direct ($850K+) following successful Phase I.

### Optical-terminal integration (v0.3+ stretch)

The §6.4 mass and power cap (4.0 kg / 50 W / 100 W peak TX) is a hard exit criterion for any optical-terminal partner integration. Candidate partners per [docs/CADUCEUS-005.md](docs/CADUCEUS-005.md) §8:

- Mynaric
- BridgeComm
- Astrogate Labs
- CACI

### Quantum-state QKD (v3.x research horizon, SPECULATIVE)

Quantum-state teleportation for QKD over interplanetary distances remains [SPECULATIVE] per CADUCEUS-004 §0. Reference: Yin et al. 2017 (Micius, satellite-Earth at 1200 km). Interplanetary extension is a v3.x+ research horizon, not a current development item.

### Cross-agency namespace federation

Caduceus-BodyID v1.0 (§3.5 of CADUCEUS-004) is Visionblox-extended. Broader adoption would require:

- IETF or CCSDS RFC submission for vbx-body URN scheme
- Federation discussion with NASA SCaN, ESA, JAXA, CSA, other space agencies
- Operational test deployment alongside CCSDS protocols

### Compliance-graded variants for adjacent domains

The Caduceus methodology (compliance-graded bundle fabric over BP7+BPSec) generalizes to other delay-tolerant high-stakes domains:

- **Subsea / underwater autonomous systems** (DARPA Manta Ray-class)
- **Polar / remote terrestrial deployments** with intermittent connectivity
- **Military tactical edge** (deny / disconnected / intermittent / low-bandwidth)

These are R&D directions, not committed roadmap items.

---

## What is NOT on the roadmap (by design)

- **Caduceus as a commercial SaaS offering.** Caduceus is a substrate / specification, not a service. Visionblox commercial offerings build on top.
- **Caduceus replacing CCSDS or DTN standards.** Caduceus sits on top of and complements RFC 9171, RFC 9172, and CCSDS Bundle Protocol; it does not replace them.
- **Caduceus competing with KeySpace 2024.** KeySpace addresses PKI revocation in interplanetary networks. Caduceus addresses compliance-graded authority decisions on top of any PKI substrate, including KeySpace's mechanisms. The two are complementary.
- **Caduceus as a classified-level offering at v0.2.** Visionblox does not currently hold facility clearance. Classified extensions are v0.3+ pending appropriate clearance posture.
- **Caduceus claiming bench compliance without third-party verification.** External-party verification of compliance claims is mandatory per [docs/CADUCEUS-BENCH-001.md](docs/CADUCEUS-BENCH-001.md) §5.

---

## Reviewing this roadmap

This roadmap is reviewed quarterly. Material changes require:

1. A pull request modifying this file with rationale in the PR description
2. Cross-reference updates in [CHANGELOG.md](CHANGELOG.md)
3. If the change affects bench-asserted behavior, a versioned bench re-commit per [GOVERNANCE.md](GOVERNANCE.md)

Current revision: **A** — initial roadmap as of 2026-06-09.

---

*Maintainer: A. Khaalis Wooden, Sr., MBA — Visionblox LLC / Zuup Innovation Lab*
