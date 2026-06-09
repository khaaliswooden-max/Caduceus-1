# Caduceus

**A compliance-graded delay-tolerant bundle fabric for interplanetary authority propagation.**

[![Status](https://img.shields.io/badge/status-bench%20v1.2.1%20commit%20pending-blue)](#current-status)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![Methodology](https://img.shields.io/badge/methodology-ZCS--6-cyan)](#methodology)
[![Bench](https://img.shields.io/badge/bench-caduceus--bench%20v1.2.1-cyan)](docs/CADUCEUS-004.md)
[![LEDGER](https://img.shields.io/badge/ledger-%230004%20pending-orange)](docs/CADUCEUS-LEDGER-PREP-001.md)

---

## What is Caduceus

Caduceus is a compliance-graded bundle fabric that sits above the Bundle Protocol v7 (RFC 9171) and Bundle Protocol Security (RFC 9172) standards to provide authority propagation for communication channels where round-trip light-time exceeds the desired authority-decision cutover time — a regime that emerges only in interplanetary deployments.

The name is literal: the two snakes of the herald's staff map to **Aletheia** (the provenance / trust strand, providing per-bundle cryptographically-anchored audit) and **Civium** (the authority / decision strand, providing graph-based gating). Together they wrap a central Bundle Protocol stream that carries crewed-mission command, telemetry, and advisory traffic across distances of minutes to hours of light-time.

**Five things Caduceus does that nothing else does in combination:**

1. **Interplanetary-RTT-aware authority propagation** — handles revocation, succession, and stale-graph behavior when light-time exceeds the desired cutover time
2. **Cryptographically-committed benchmark** (caduceus-bench v1.2.1) — the spec is hash-anchored before any substrate exists, preventing benchmark-gaming
3. **OMB M-25-22 AIBOM-compatible from day one** — Aletheia hash-chained provenance ledger is the AIBOM substrate
4. **Open-source friendly substrate** — ION-DTN, libsodium, RFC 9171+9172, HKDF — no proprietary lock-in
5. **Single-fault-tolerant v0.2 path to dual-fault-tolerance v0.3** — honest about the crewed-deployment readiness gate

---

## Current status

| Component | Status | Reference |
|---|---|---|
| **caduceus-bench v1.2.1 specification** | LOCKED, commit pending | [docs/CADUCEUS-004.md](docs/CADUCEUS-004.md) |
| **Phase 3 adversarial sweeps** | 20 attacks closed (2 rounds) | [docs/CADUCEUS-PHASE3-*.md](docs/) |
| **Aletheia LEDGER #0004 commit** | Artifacts ready, awaiting PPA sequencing | [docs/CADUCEUS-LEDGER-PREP-001.md](docs/CADUCEUS-LEDGER-PREP-001.md) |
| **Provisional Patent Applications** | 3 drafting targets ready for practitioner | [docs/CADUCEUS-IP-*.md](docs/) |
| **IEEE paper draft** | Ready (IEEEtran LaTeX, 30+ refs) | [paper/Caduceus_IEEE_Paper.tex](paper/) |
| **v0.2 substrate prototype** | Plan locked (M0–M8, 20 weeks, ~3.25 FTE) | [docs/CADUCEUS-005.md](docs/CADUCEUS-005.md) |
| **Formal-methods composition** | Lean 4 + TLA+ obligations specified | [docs/CADUCEUS-006.md](docs/CADUCEUS-006.md) |
| **Phase 6 attack sweep** | 33-attack catalog ready for substrate M5+ | [docs/CADUCEUS-007.md](docs/CADUCEUS-007.md) |
| **CMMC L2 self-assessment** | Framework complete, Visionblox attestation in motion | [docs/CADUCEUS-CMMC-001.md](docs/CADUCEUS-CMMC-001.md) |
| **Federal capture posture** | 8 channels prioritized (NSTXL, NASA SBIR, SDA, others) | [docs/CADUCEUS-CAPTURE-001.md](docs/CADUCEUS-CAPTURE-001.md) |
| **Aletheia v0.3 chain extension** | Spec drafted, ~2–3 person-weeks to implement | [docs/ALETHEIA-V03-001.md](docs/ALETHEIA-V03-001.md) |

---

## Architecture (quick view)

See [DWG CADUCEUS-001](blueprints/caduceus_blueprint_001_architecture.png) for the system blueprint and [DWG CADUCEUS-002](blueprints/caduceus_blueprint_002_deployment.png) for the deployment topology.

Seven hardware-function floors extend the existing VBX-ISPS HF-1 through HF-15 substrate:

```
HF-22  Frame-context interlock + root-cause clustering + audit log    (F4)
HF-21  Nonce-seen-set per-stream partition + quota                    (T4)
HF-19  URN parse + Caduceus-BodyID v1.0 + dual-version window      (F1, F2)
HF-17  Inbound 4-check + Civium gate + revocation stream  (T1, T2, T3, T6, T7)
HF-16  Outbound attestation (Ed25519 over canonical tuple)            (T1)
HF-18  ACI drift monitor + theoretical-floor binding                  (T5)
HF-20  Mercury-bounded WCET enforcement (5-cycle deterministic)  (all WCET)
─────────────────────────────────────────────────────────────────────────────
BP7 (RFC 9171) + BPSec (RFC 9172, 9173) + HKDF (RFC 5869)
Civium · MVCI · Aletheia DAC · Mercury · Bus ISO · Secure Element
Jetson Orin NX (shielded, 16 GB, ≤200 W)
```

---

## Methodology

Caduceus is developed under the **ZCS-6 falsification-first ordering discipline**:

| Phase | Activity | Status |
|---|---|---|
| 1 | Whitespace mapping | Complete |
| 2 | Benchmark specification | LOCKED at v1.2.1 |
| 3 | Adversarial sweep against bench | 20 attacks closed across 2 rounds |
| 4 | Cryptographic commit + deterministic defense | Artifacts ready, LEDGER #0004 pending |
| 5 | Substrate prototype | Plan locked (CADUCEUS-005) |
| 6 | Vertical-integration attacks against substrate | Plan locked (CADUCEUS-007) |

**Core discipline:** the benchmark is committed before any substrate exists. When Phase 6 surfaces a substrate defect, the substrate is fixed at the architecture level — the benchmark is never silently softened. Bench revisions are versioned re-commits with explicit delta justification.

See [CLAUDE.md](CLAUDE.md) for AI-assistant working conventions; see [GOVERNANCE.md](GOVERNANCE.md) for the human governance model.

---

## Repository layout

```
caduceus/
├── README.md                   # This file
├── LICENSE                     # Apache-2.0
├── CONTRIBUTING.md             # How to contribute
├── CHANGELOG.md                # Version history
├── ROADMAP.md                  # Forward plan
├── SECURITY.md                 # Vulnerability reporting
├── CODE_OF_CONDUCT.md          # Contributor Covenant 2.1
├── CITATION.cff                # Citation metadata
├── GOVERNANCE.md               # Governance model
├── CLAUDE.md                   # AI-assistant working conventions
│
├── docs/                       # Specifications and plans
│   ├── CADUCEUS-001.md         #   bench v1.0
│   ├── CADUCEUS-002.md         #   bench v1.1
│   ├── CADUCEUS-003.md         #   bench v1.2
│   ├── CADUCEUS-004.md         #   bench v1.2.1 (LEDGER #0004 manifest)
│   ├── CADUCEUS-005.md         #   v0.2 substrate plan (M0–M8)
│   ├── CADUCEUS-006.md         #   Mercury × Caduceus formal-methods spec
│   ├── CADUCEUS-007.md         #   Phase 6 vertical-integration attacks
│   ├── CADUCEUS-PHASE3-*.md    #   Phase 3 adversarial sweeps
│   ├── CADUCEUS-IP-*.md        #   PPA filing targets
│   ├── CADUCEUS-BENCH-001.md   #   Consolidated external-facing bench
│   ├── CADUCEUS-CMMC-001.md    #   CMMC L2 self-assessment framework
│   ├── CADUCEUS-CAPTURE-001.md #   Federal capture posture
│   ├── CADUCEUS-PRACTITIONER-001.md
│   ├── CADUCEUS-LEDGER-PREP-001.md
│   ├── CADUCEUS-ARXIV-001.md
│   ├── CADUCEUS-NSTXL-001.md
│   ├── CADUCEUS-SBIR-NASA-001.md
│   └── ALETHEIA-V03-001.md     #   Aletheia v0.3 chain extension
│
├── paper/                      # IEEE paper draft + arXiv preprint
│   ├── Caduceus_IEEE_Paper.tex
│   └── references.bib
│
└── blueprints/                 # SVG + PNG architecture sketches
    ├── caduceus_blueprint_001_architecture.svg
    ├── caduceus_blueprint_001_architecture.png
    ├── caduceus_blueprint_002_deployment.svg
    └── caduceus_blueprint_002_deployment.png
```

---

## Citation

If you reference Caduceus or caduceus-bench in academic or industry work:

```bibtex
@misc{caduceus2026bench,
  title  = {caduceus-bench v1.2.1: A Compliance-Graded Bundle Fabric
            Benchmark for Interplanetary Authority Propagation},
  author = {Wooden, Aldrich K., Sr.},
  year   = {2026},
  note   = {Aletheia LEDGER \#0004, SHA-256
            0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874},
  howpublished = {Visionblox LLC / Zuup Innovation Lab, CADUCEUS-004}
}
```

See [CITATION.cff](CITATION.cff) for machine-readable metadata.

---

## Federal capture posture (summary)

For details see [CADUCEUS-CAPTURE-001.md](docs/CADUCEUS-CAPTURE-001.md). Eight channels prioritized:

1. NSTXL / SOSSEC consortium membership ($500–$1,500 entry; broad OTA access)
2. NASA SBIR Phase I — SCaN Cognitive Communication ($150K, FY27 cycle)
3. DIU Commercial Solutions Openings (no membership; quarterly opportunities)
4. Navy SBIR DON26BZ03-NV062 — Secure Tasking ($315K; cleared-partner gated)
5. SDA Tranche 3 Ground Entry Point — sub-teaming with primes
6. AFRL OTAFI ($499M 5-year OTA via SOSSEC)
7. AFWERX SBIR Open Topic (alternative fast-path)
8. NASA SBIR Phase II Direct (conditional on Phase I)

---

## Honest scope

- This is a **specification, plan, and methodology repository**, not a substrate codebase. The substrate build is planned in [CADUCEUS-005](docs/CADUCEUS-005.md) but has not been executed.
- caduceus-bench v1.2.1 is the cryptographically-committed source of truth. All other documents support it.
- The repository deliberately uses epistemic markers (**VERIFIED** / **PLAUSIBLE** / **SPECULATIVE**) to distinguish levels of confidence. See [CONTRIBUTING.md](CONTRIBUTING.md).
- The PPA targets in [docs/CADUCEUS-IP-*.md](docs/) are practitioner-input materials, not filing-ready papers. A USPTO-registered patent practitioner is required.
- LEDGER #0004 commit has not yet been executed. The artifacts are ready but operational sequencing (PPA filings first) is in progress.
- KeySpace 2024 (Smailes et al., Oxford + armasuisse, arXiv:2408.10963) is closely-adjacent prior art on interplanetary PKI; Caduceus is positioned as complementary, not anticipating. See [CADUCEUS-PRACTITIONER-001](docs/CADUCEUS-PRACTITIONER-001.md) §2.3.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE).

Copyright © 2026 Visionblox LLC / Zuup Innovation Lab.

---

## Contact

**A. Khaalis Wooden, Sr., MBA** — Director of Enterprise Capture & Compliance
Visionblox LLC / Zuup Innovation Lab

- khaaliswooden@gmail.com
- khaalis.wooden@visionblox.com
- aldrich.wooden@snhu.edu (academic)

For security disclosures, see [SECURITY.md](SECURITY.md) — do not file public issues for vulnerabilities.

---

*Methodology: ZCS-6 (Phase 4 complete). Benchmark: caduceus-bench v1.2.1. Substrate: v0.2 PLAN.*
