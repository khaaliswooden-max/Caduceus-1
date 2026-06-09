# Caduceus — Phase 6 Vertical-Integration Attack-Sweep Plan

| | |
|---|---|
| **Document ID** | CADUCEUS-007 |
| **Revision** | A (draft) |
| **Methodology** | ZCS-6 Phase 6 (vertical-integration attacks against substrate, not bench) |
| **Target substrate** | v0.2 build per CADUCEUS-005 (HF-16 through HF-22 + EPHEMERIS peer) |
| **Target bench** | caduceus-bench v1.2.1 (CADUCEUS-004, LEDGER #0004) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Phase 6 attack catalog; ready for execution at substrate M7 (end-to-end integration test). Plan-level pre-build attacks (§10) are executable immediately against CADUCEUS-005 itself. |

---

## 0. Scope statement

Phase 6 attacks the **substrate as built**, not the bench as committed. Whereas Phase 3 found architectural gaps in the bench specification (closed across two iterations producing v1.2.1), Phase 6 finds implementation gaps in the substrate that would cause a candidate implementation to fail bench compliance despite passing per-module unit tests.

Phase 6 is not a final-acceptance test. It is a discovery engine for two distinct classes of finding:

1. **Substrate defects** — implementation bugs that violate bench compliance. Closure: architecture-level fix in the substrate.
2. **Bench imprecisions** — cases where the bench is ambiguous or incomplete in ways only surfaceable in a running substrate. Closure: bench revision via versioned re-commit (would produce v1.3.0 or v2.0 manifest), NEVER a silent change.

The ordering discipline holds: when Phase 6 surfaces a finding, we fix the substrate or revise the bench transparently. We do not soften the bench to accommodate the substrate.

---

## 1. Phase 6 vs Phase 3 — what's different

| Dimension | Phase 3 (bench attack) | Phase 6 (substrate attack) |
|---|---|---|
| **Target** | Spec assertions | Running implementation |
| **Method** | Adversarial reasoning on the spec text | Adversarial execution against running code |
| **Finding type** | Architectural gap | Implementation defect OR composition gap OR emergent behavior |
| **Pass criterion** | Spec assertion either closes the attack or doesn't | Substrate either survives the attack or doesn't |
| **Closure (typical)** | Bench revision; new assertion or tightened threshold | Substrate code fix; bench rarely changes |
| **Closure (atypical)** | — | Bench revision if the attack reveals genuine spec ambiguity |
| **Iteration count** | 2 rounds before commit (v1.0 → v1.1 → v1.2 → v1.2.1) | Open-ended; runs continuously as substrate evolves |
| **Status of bench at completion** | Locked, immutable, hash-committed | Unchanged unless attack forces revision |

---

## 2. Eight attack categories

Phase 6 attacks group into eight categories, each targeting a different class of substrate vulnerability that per-module unit tests are unlikely to surface.

| # | Category | Why per-module tests miss this class | Example |
|---|---|---|---|
| A | Implementation-level | Tests use synthetic inputs; real-world inputs differ | Race condition under specific concurrent load |
| B | Composition | Each module tested in isolation; combination untested | Two modules' interactions cascade failures |
| C | Resource exhaustion | Tests run for minutes; production runs for months | Slow memory leak surfaces after 60 days |
| D | VBX-ISPS integration | Tests in isolation; integration with existing platform unmocked | Bus ISO breakdown under combined load |
| E | Emergent behavior | Tests don't replicate full-mission state | 12-stream concurrent operation pattern from real ops |
| F | Supply chain | Tests assume clean toolchain | Compromised dependency or hardware |
| G | Substrate-level side channels | libsodium constant-time is module-scoped | Side channels at the full-stack level (power, EM) |
| H | Failure-mode | Tests assume nominal operation | Behavior during partial substrate failure |

---

## 3. Phase 6 attack catalog (33 named attacks)

Numbering format: `AV6.<sequential>` to distinguish from bench Phase 3 attacks (numbered 1–28).

### 3.1 Category A — Implementation-level (5 attacks)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.1** | HF-17 four-check race under concurrent bundles | T1 atomicity | Two bundles arriving on the same authority stream from different cores; per-stream mutex acquisition order varies. Verifier exploits if four checks aren't transactionally atomic. |
| **AV6.2** | Speculative-execution side channel on signature verification | T1 constant-time | Modern CPUs with branch prediction and speculative execution may leak signature comparison timing despite libsodium constant-time. Spectre/Meltdown-class. |
| **AV6.3** | libsodium version skew between PHRONESIS and EPHEMERIS | T1, F3 | If PHRONESIS uses libsodium 1.0.20 and EPHEMERIS uses 1.0.18 (or vice versa), edge-case behavior may diverge. F3 mismatch detection should catch but assertion is brittle. |
| **AV6.4** | Rust async runtime starvation under priority lane saturation | T7 priority lane | If `tokio` or equivalent runtime has insufficient priority enforcement, revocation-stream bundles may queue behind standard bundles during high load. |
| **AV6.5** | Configuration drift on theoretical-floor constants | T5 floor | If the signed theoretical-floor constants per §3.7 aren't atomically loaded at boot (e.g., split across multiple config files with partial load), substrate may operate temporarily without floor enforcement. |

### 3.2 Category B — Composition (5 attacks; HIGHEST PRIORITY)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.6** | HF-17 + HF-18 race during ACI floor recalibration | T2, T5 | While HF-18 (ACI drift monitor) is updating the theoretical-floor constant for a link, HF-17 (verifier) may read the floor mid-update. Atomicity must extend across modules, not just within. |
| **AV6.7** | HF-22 confirmation cluster overflow → HF-9 deadlock | F4, T3 | If F4 hard rate limit is reached AND HF-9 entry occurs simultaneously, the two-source-confirmation exit (T3 sub-point d) may attempt to use confirmation channel that F4 has saturated. Deadlock. |
| **AV6.8** | BPSec session key + Aletheia v0.3 key rotation collision | B2, A2 | If BPSec session keys are derived from Aletheia parent keys via HKDF (B2 spec), and the Aletheia parent key rotates per v0.3 protocol, in-flight bundles signed with derived-from-old-parent keys must remain verifiable in the dual-acceptance window. |
| **AV6.9** | HF-19 URN parsing + HF-21 nonce check race | F1, T4 | If HF-19 parses the URN before HF-21 checks the nonce, an attacker may submit a bundle where the URN payload contributes to nonce computation. Order of operations must be HF-19 → HF-21 strictly. |
| **AV6.10** | HF-20 Mercury budget enforcement during HF-16 + HF-17 concurrency | All WCET-bounded | If HF-16 (outbound) and HF-17 (inbound) compete for Mercury kernel cycles during simultaneous burst, the WCET budgets may be exceeded for one path. Mercury's scheduling discipline must extend across modules. |

### 3.3 Category C — Resource exhaustion (4 attacks)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.11** | Orin NX memory pressure over 90-day continuous operation | All HF | Memory fragmentation, leak in any HF module, or garbage-collector pause may degrade after weeks. Surface only with prolonged operation. |
| **AV6.12** | Aletheia chain write storm during F4 confirmation flood | F4 audit logging | If F4 audit logging writes each confirmation event synchronously and the chain write throughput is insufficient, sustained flood may back up the chain write buffer indefinitely. |
| **AV6.13** | Secure Element signing throughput exhaustion | T1 outbound | Hardware Secure Element signing has a finite throughput (typically 100s of signatures/sec). Legitimate operational burst (multiple crew, multiple ground sources) may saturate. |
| **AV6.14** | Thermal throttling during sustained peak TX | P1, P3 | Sustained 100W peak TX (per §6.4 cap) drives thermal load. Spacecraft thermal control has limits. If throttling reduces TX power, link-budget margin (P1) may erode below 3 dB. |

### 3.4 Category D — VBX-ISPS integration (4 attacks)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.15** | VBX-ISPS HF-13 (life support) priority preemption causing HF-17 gate timeout | All Caduceus HF | If VBX-ISPS life-support subsystems preempt Caduceus compute (correct behavior), HF-17's WCET budget may not be met. Caduceus must fail safe (T3 → HF-9), not silently degrade. |
| **AV6.16** | Bus ISO compartmentalization breakdown under combined load | All | Bus ISO mediates between Caduceus and VBX-ISPS workloads on shared compute. Under combined peak load (life-support emergency + active EVA + Caduceus authority decision), isolation guarantees may relax. |
| **AV6.17** | Mercury kernel contention between VBX-ISPS and Caduceus | HF-20 | Both Caduceus and VBX-ISPS use Mercury for WCET-bounded execution. Contention discipline must be enforced; per-tenant budgets must be honored even under burst. |
| **AV6.18** | Secure Element shared between VBX-ISPS and Caduceus introduces side channel | T1, AU.L2-3.3.8 | If both subsystems share the Secure Element for signing, timing of one subsystem's operations may leak about the other's. Compartmentalization on the SE must be verified. |

### 3.5 Category E — Emergent behavior (4 attacks; HIGHEST PRIORITY for crewed deployment)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.19** | 90-day continuous operation reveals slow leak | HF-18, HF-21 | ACI drift monitor (HF-18) maintains rolling observation windows; nonce-set (HF-21) maintains TTL-indexed structures. Either may leak over weeks. Surfaces only at scale. |
| **AV6.20** | Maximum-conjunction Mars geometry stress | T7, T5 | At maximum Earth-Mars distance (~22 min one-way), priority-lane buffer for revocations must hold bundles for the full propagation window. Sustained operation at maximum geometry tests buffer adequacy. |
| **AV6.21** | Multi-stream operations: 12+ concurrent authority streams | T4, T2 | Real operations involve multiple ground centers (Houston, JPL, Goddard, multi-agency partners) plus Mars surface ops, lunar relay, etc. T4 per-stream nonce-set partitioning must handle 12+ streams without cross-contamination or starvation. |
| **AV6.22** | Crew sustained-denial attack | F4, T3 | If a crew member refuses all confirmation requests for 8 hours (cognitive load, medical event, deliberate refusal), how does the substrate behave? F4 rate-limit handles flooding; what handles starvation? |

### 3.6 Category F — Supply chain (4 attacks)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.23** | Rust toolchain backdoor (compromised cargo dependency) | All HF | A dependency in the substrate's Cargo.toml is compromised upstream; the substrate binary contains an injected backdoor. SBOM auditing + reproducible builds + signed binaries are the mitigation. |
| **AV6.24** | ION-DTN open-source contributor supply-chain attack | B1, B2, B3 | A malicious commit to ION-DTN (the BP7 reference implementation) introduces a subtle bug that affects custody-transfer semantics. Detection via signed-binary verification + behavior monitoring. |
| **AV6.25** | Orin NX hardware Trojan | All | NVIDIA Orin NX is commercial off-the-shelf; supply-chain risk for any commercial silicon. Mitigation: Trusted Foundry program (for true defense applications) or runtime attestation (TPM 2.0). |
| **AV6.26** | EPHEMERIS firmware compromise via OTA update | F3, F4 | Per §0 of CADUCEUS-004, EPHEMERIS firmware integrity is assumed out-of-bench. Phase 6 still needs to test what happens if assumption fails — does the rest of the system fail safe? |

### 3.7 Category G — Substrate-level side channels (3 attacks; LOWER PRIORITY for v0.2)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.27** | Power analysis on Ed25519 signing across full PHRONESIS envelope | T1 | libsodium is constant-time at the algorithm level; power consumption of the entire compute path (CPU, memory, bus) may still vary based on key bits. Differential power analysis (DPA) is a known threat. |
| **AV6.28** | EM emanation revealing Aletheia chain content | AU.L2-3.3.8 | High-bandwidth EM emanation from Orin NX may leak chain content to an adversary with proximity. Mitigation: Faraday shielding on the suit enclosure. |
| **AV6.29** | Acoustic side channel from cooling fan modulation | T1 | If thermal load correlates with crypto operations, fan speed modulation may leak. Spacecraft may not have audible fans; mitigation may be irrelevant. Document anyway. |

### 3.8 Category H — Failure-mode attacks (4 attacks)

| ID | Attack | Targets | Mechanism |
|---|---|---|---|
| **AV6.30** | Partial HF-module failure recovery | T3 fail-safe | If HF-21 process crashes (e.g., OOM kill), what happens? Does substrate enter HF-9? Continue with degraded T4? The behavior must be deterministic and bench-compliant. |
| **AV6.31** | Aletheia chain corruption recovery | A1, A6 | If chain entries are corrupted (storage failure, partial write during power loss), how does the substrate recover? Genesis re-verification must work; partial chain must trigger HF-9. |
| **AV6.32** | Key rotation propagation gap exploitation | A4 dual-acceptance | During the dual-acceptance window, an adversary with one of the two keys can sign malicious entries. The window is a known vulnerability; bounded by §2 of ALETHEIA-V03-001 but adversarial probing of the window edges. |
| **AV6.33** | HF-9 exit confirmation MitM during long EVA | T3 sub-point d | The two-source-confirmation exit from HF-9 (per T3) requires (crew + ground) OR (crew + second-EPHEMERIS). MitM on either confirmation channel during EVA-class operational tempo. |

---

## 4. Phase 6 execution methodology

### 4.1 Sequence per attack

For each AV6.<N> attack:

1. **Define attack scenario** — precise inputs, environment, timing
2. **Identify target assertions / modules** — what's expected to hold
3. **Set up test environment** — substrate instance, instrumentation, adversary tooling
4. **Execute attack** — run the scenario
5. **Observe outcome** — does the substrate enforce the bench? Does it fail safe? Does it silently degrade?
6. **Categorize finding** —
   - **PASS** (substrate enforces; no action)
   - **SUBSTRATE FAIL** (substrate didn't enforce; bench is correct; FIX SUBSTRATE)
   - **BENCH AMBIGUITY** (substrate behavior is defensible but bench wording is unclear; REVISE BENCH via versioned re-commit)
   - **NEW CATEGORY** (attack reveals a class not in current bench; major bench revision)
7. **Record in compliance report** with full audit trail (per CADUCEUS-BENCH-001 §4.3)
8. **If FAIL or AMBIGUITY:** triage, design fix, re-run

### 4.2 Test environment requirements

| Component | Need |
|---|---|
| Substrate instance | Built per CADUCEUS-005 M7; running on representative Jetson Orin NX |
| Adversary tooling | Bundle generator with controlled malformation; channel simulator with controlled delay/loss |
| Instrumentation | Substrate logging at HF-module granularity; Aletheia chain monitoring; Mercury kernel cycle-count instrumentation |
| Time scale | Range from microseconds (race conditions) to 90+ days (slow leaks) |
| Hardware | At least one Orin NX dev kit; cooled to representative thermal env; ideally with hardware Secure Element for SE-related attacks |
| Software | Test harness (Rust + Python); ION-DTN reference implementation for comparison; libsodium with instrumentation |
| Network | Channel simulator (e.g., NetEm or commercial DTN test gear) for delay/loss/burst injection |

### 4.3 Estimated execution time per category

| Category | Total attacks | Effort per attack | Total effort |
|---|---|---|---|
| A (Implementation) | 5 | 1–2 weeks | 5–10 person-weeks |
| B (Composition) | 5 | 2–3 weeks | 10–15 person-weeks |
| C (Resource) | 4 | 1–12 weeks (some require 90-day runs) | 4–48 person-weeks elapsed |
| D (VBX-ISPS integration) | 4 | 2–3 weeks | 8–12 person-weeks |
| E (Emergent) | 4 | 4–13 weeks (long-duration) | 16–52 person-weeks elapsed |
| F (Supply chain) | 4 | 1–3 weeks | 4–12 person-weeks |
| G (Side channels) | 3 | 2–4 weeks; requires specialized equipment | 6–12 person-weeks |
| H (Failure-mode) | 4 | 1–2 weeks | 4–8 person-weeks |
| **Total** | **33** | | **~57–169 person-weeks elapsed; ~30–60 person-weeks active labor** |

Phase 6 attack execution is NOT serial. Many attacks run in parallel; some require long elapsed time (sustained 90-day operation) but minimal active labor. Realistic active-labor budget: **~6–9 person-months across all 33 attacks**, distributed over **~6 calendar months** if executed efficiently.

### 4.4 Priority ordering (which attacks first)

Recommended execution order based on likely-finding rate × consequence:

| Priority | Category | Attacks | Rationale |
|---|---|---|---|
| **1** | B (Composition) | AV6.6–AV6.10 | Most likely to surface real defects; composition gaps are systematic |
| **2** | E (Emergent) | AV6.19–AV6.22 | Cannot be deferred; long elapsed time means start early |
| **3** | D (VBX-ISPS integration) | AV6.15–AV6.18 | Touches existing production code; high consequence |
| **4** | C (Resource) | AV6.11–AV6.14 | Gating for crewed deployment readiness |
| **5** | H (Failure-mode) | AV6.30–AV6.33 | Required before crewed deployment |
| **6** | A (Implementation) | AV6.1–AV6.5 | Per-module tests catch many; remainder caught here |
| **7** | F (Supply chain) | AV6.23–AV6.26 | Broader operational concern; not unique to Caduceus |
| **8** | G (Side channels) | AV6.27–AV6.29 | Requires specialized equipment; lower priority for v0.2 |

### 4.5 Decision rules

**Rule 1: Architecture-level fix only.** Substrate defects are fixed at the architecture level. Patch-and-move-on is forbidden. If a fix requires module boundary changes, those changes go through standard CADUCEUS-005 module-level review.

**Rule 2: Bench revisions are commits.** If Phase 6 surfaces a genuine bench ambiguity, the bench is revised via versioned re-commit (e.g., v1.3.0). Old bench remains the v1.2.1 commit; new bench is v1.3.0 with explicit delta justification.

**Rule 3: Document SUBSTRATE FAIL openly.** All Phase 6 substrate fails must appear in the compliance report. No silent fixes; chain entries record the failure, the diagnosis, the fix, and the re-test outcome.

**Rule 4: Phase 6 never softens the bench.** Per ZCS-6 ordering, the bench drives the substrate. Substrate difficulty does not justify bench relaxation.

**Rule 5: Crewed-deployment gates.** Categories B, C, D, E, H must close 100% before crewed deployment authorization. Categories A, F, G may have known-residual-risk items documented in operational POAMs.

---

## 5. Test infrastructure investment

Phase 6 requires test infrastructure beyond what CADUCEUS-005 M0–M7 builds. Required investments:

| Investment | Cost (rough) | Required for | Earliest needed |
|---|---|---|---|
| Channel simulator with light-time delay (commercial or NetEm-based) | $5K–$20K | Categories B, C, E | M5 (Week 8) |
| Long-duration test infrastructure (continuous-operation cluster) | $10K–$30K | Category E | M5 |
| Power analysis test bench (oscilloscope + probes) | $20K–$50K | Category G (deferrable) | Optional for v0.2 |
| EM emanation test environment (anechoic chamber rental) | $5K/day rental | Category G (deferrable) | Optional for v0.2 |
| Secondary Orin NX for VBX-ISPS integration tests | $4K | Category D | M5 |
| Hardware Secure Element development kits | $2K–$10K | Categories A, D | M5 |
| **Total Phase 6 infrastructure** | **~$45K–$120K** | | (lower if categories G deferred) |

This infrastructure cost is **in addition to** the ~$650K substrate build estimate in CADUCEUS-005 §6.2. Phase 6 execution adds: ~$45K–$120K infrastructure + ~6–9 person-months of attack-execution labor (~$150K–$250K loaded labor).

**Total Phase 6 budget:** ~$200K–$370K beyond substrate build.

---

## 6. Sequencing relative to substrate build

| Milestone | When Phase 6 runs | What runs |
|---|---|---|
| M3 (Week 5–11) | — | No Phase 6; modules still under construction |
| M5 (Week 7–13) | Begin E (long-duration) | AV6.19 starts (90-day clock) |
| M6 (Week 11–15) | Begin B (composition) | AV6.6–AV6.10 in parallel with M6 |
| M7 (Week 14–16) | **Phase 6 main execution** | A, D, F categories; complete B if not done |
| M7→M8 transition (Week 16–17) | Triage Phase 6 findings | Architecture fixes; bench revision decisions |
| M8 (Week 17–20) | **Final Phase 6 sweep** | C, H categories; G deferred or scoped |
| Post-M8 (Week 20+) | Continuous Phase 6 | New attacks added as discovered; ongoing |

---

## 7. Plan-level attacks (pre-build, executable now)

Phase 6 has a pre-substrate-build variant: attacks against CADUCEUS-005 itself. These are executable today and find gaps in the build plan before the build starts.

| ID | Attack on CADUCEUS-005 | Mechanism | Mitigation |
|---|---|---|---|
| **PA.1** | HF-17 effort estimate is the longest pole; if it slips, M3–M7 all slip | CADUCEUS-005 §4.1 shows 6-week HF-17 build; large modules historically take 1.5–2× | Build buffer time into M3; identify HF-17 lead engineer first; consider splitting HF-17 across two engineers |
| **PA.2** | Aletheia v0.3 unavailable at M1 | CADUCEUS-005 M1 depends on Aletheia v0.3; if Aletheia v0.3 implementation slips, M1 slips | Per ALETHEIA-V03-001 §7, v0.3 implementation is 2–3 person-weeks; start now (parallel to PPA filings) |
| **PA.3** | Orin NX shielded enclosure procurement not specified | CADUCEUS-005 §3 mentions "Jetson Orin NX 16 GB, shielded enclosure"; procurement lead time may be 4–6 weeks | Identify supplier now; lock procurement at M0 (LEDGER commit) |
| **PA.4** | M7 integration test pass-criteria ambiguous | CADUCEUS-005 §4.1 M7 says "Phase 3 re-execution; 100% closure verified" but doesn't specify which attack environment | Tighten M7 criteria; pass = all 20 Phase 3 attacks closed against substrate AND all category-B Phase 6 attacks executed |
| **PA.5** | ~332 test corpus mapping to bench assertions is approximate | CADUCEUS-BENCH-001 §4 estimates 332 tests; coverage of every assertion threshold not enumerated | Build explicit assertion-to-test mapping table before M3; each assertion must have ≥3 distinct tests |

These five plan-level attacks should be addressed in CADUCEUS-005 Rev B before the substrate build starts. None requires bench revision; all are substrate-side planning improvements.

---

## 8. Open items

1. **Test infrastructure procurement timing** — channel simulator and long-duration test cluster need procurement starting M0 if M5 deadlines are to be met.
2. **Specialized equipment for G category** — power analysis bench is the largest single investment. Defer to v0.3 unless a specific federal-channel solicitation requires it.
3. **External red team engagement** — Phase 6 internal-team execution may miss adversarial creativity that an external red team would find. Consider engaging an external team (e.g., a NASA Goddard Cyber Defense team, MITRE FFRDC, or a commercial firm like Galois) for a 2–4 week external Phase 6 sweep before any crewed deployment.
4. **Phase 6 attack expansion** — the 33-attack catalog is a starting set. As substrate is built, additional attacks will surface; the catalog should be open-ended and version-tracked.
5. **Cross-substrate attack reuse** — many Phase 6 attacks (B, C, E, H categories) are reusable for any substrate implementing caduceus-bench, not just the Visionblox reference substrate. Consider publishing the attack catalog (or a subset) as a community resource alongside the bench.
6. **CMMC / federal-capture intersection** — Phase 6 execution evidence is directly usable for federal proposal past-performance and for CMMC L2 control demonstrations (specifically AU, CA, RA, SI families). Document the dual use.

---

## 9. Reporting structure

Each Phase 6 execution produces a chain-anchored report. Recommended structure:

```
PHASE_6_REPORT_<batch_id>
{
  "batch_id": "<sequential>",
  "execution_date_tai": "<TAI>",
  "substrate_version": "<git hash or release tag>",
  "bench_version": "v1.2.1",
  "bench_ledger_hash": "0fe3b32db5522db6...",
  "attacks_executed": [
    {
      "attack_id": "AV6.6",
      "category": "B",
      "outcome": "PASS | SUBSTRATE_FAIL | BENCH_AMBIGUITY | NEW_CATEGORY",
      "evidence_pointers": [...],
      "remediation_status": "..."
    },
    ...
  ],
  "summary": {
    "total_attacks": N,
    "pass": X, "substrate_fail": Y, "bench_ambiguity": Z, "new_category": W
  },
  "signature": "<Ed25519 over canonical>",
  "prev_chain_hash": "..."
}
```

This report is appended to the Aletheia chain. The chain becomes the auditable record of substrate maturity over time.

---

## 10. Honest scope statement

- Phase 6 cannot begin meaningfully until substrate M5 (Week 8 of build). Plan-level pre-build attacks (§7) are executable now.
- 33 attacks is a starting catalog. Real-world Phase 6 execution will likely add 10–30 more attacks as substrate maturity reveals new probe surfaces.
- Some attacks (long-duration emergent, side channels at substrate level) require significant elapsed time or specialized equipment. v0.2 may need to defer these with explicit POA&M tracking; v0.3 closes them for crewed deployment.
- Phase 6 findings that surface bench ambiguities require bench revisions per §4.5 Rule 2. These revisions are not failures — they are the methodology working as intended. v1.2.1 will likely produce a v1.3.0 or v2.0 bench after Phase 6 surfaces precision needs.
- The boundary between Phase 6 "substrate defect" and Phase 6 "bench ambiguity" is sometimes unclear; the methodology requires deliberate categorization with chain-anchored audit, not silent classification.
- Phase 6 does NOT replace external review (NASA OSMA-class, IEEE peer review, KeySpace-author technical exchange). It's an internal Visionblox/Zuup discipline; external reviews complement it.
- Phase 6 execution is the discipline that earns the "compliance-graded" claim. Without it, Caduceus is just another DTN authority layer with a benchmark. With it, Caduceus is a system whose compliance-grade is itself empirically demonstrated.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-007 Rev A · Phase 6 vertical-integration attack-sweep plan*
