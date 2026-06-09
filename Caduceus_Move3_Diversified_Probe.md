# Caduceus-bench v1.2 — Diversified-Internal Probe (Move 3)

| | |
|---|---|
| **Document ID** | CADUCEUS-PROBE-001 |
| **Revision** | A |
| **Target** | caduceus-bench v1.2 (CADUCEUS-003), LEDGER #0004 candidate |
| **Methodology** | Diversified-internal probe (NOT external red-team review) |
| **Personas** | (1) patent practitioner, (2) formal-methods reviewer, (3) NASA mission-assurance reviewer |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **VERDICT: COMMIT-ELIGIBLE.** Three pre-commit recommendations are advisory, not blocking. Six post-commit and v0.2-substrate items identified. |

---

## Honest scope statement (read first)

This is an **internal probe** conducted by Claude under explicit instruction to adopt three different reviewer framings. It is NOT a substitute for actual external review by a registered patent practitioner, a credentialed formal-methods researcher, or a NASA OSMA-class mission-assurance reviewer. Findings should be treated as **directional**, not authoritative. Engagement of one or more actual external reviewers post-commit is the recommended path to v1.3 hardening.

The probe's value is breadth — surfacing categories of findings that the original Phase 3 sweeps did not target. Confidence on individual findings: **MEDIUM** (lower than the Phase 3 sweep findings, because adversarial-attack reasoning is closer to Claude's native capability than expert-reviewer reasoning).

---

## Persona 1 — Patent practitioner perspective

Framing: registered USPTO practitioner reviewing v1.2 for prior-art exposure, claim scope, § 101 risk, and enablement adequacy.

### Findings

| # | Area | Finding | Severity |
|---|---|---|---|
| **P1.1** | Prior art on time-bound authority (T2) | Kerberos has TTL-bounded tickets (1980s); Burrows-Abadi-Needham logic (1989) formalizes time-bound authority semantics. T2 as written may face anticipation rejection on the time-bound aspect alone. **Survives anticipation only via the interplanetary-RTT-aware revocation propagation combined with T2** — i.e., the system-level combination is the patentable novelty, not T2 in isolation. | **MEDIUM** |
| **P1.2** | § 101 abstract-idea risk | "Method of doing compliance" framing exposed to Alice rejection. Mitigate by claiming concrete technological improvements: atomicity locks (T1), Mercury-bounded WCET (HF-20), Secure Element key custody (HF-8), specific protocol-level bundle structures. Currently the spec mixes the technological improvements with the policy framing; claims should isolate the technological improvements. | **MEDIUM** |
| **P1.3** | Enablement gap — T5 theoretical-floor | Spec says "theoretical-floor is derived from first-principles link characteristics (thermal noise, antenna pattern, propagation distance)." Insufficient for enablement — a competent radio engineer needs the actual derivation procedure. Recommend tightening T5 language pre-commit or adding an appendix. | **LOW** (fixable in <1 hour) |
| **P1.4** | Enablement gap — B2 BPSec/Aletheia key derivation | Spec invokes BPSec "rooted in Aletheia key infrastructure" but does not specify the derivation. This is currently an open item (#13 in CADUCEUS-002 §9). Recommend a stub specification in v1.2 or a defensive arXiv supplement at commit time. | **LOW** |
| **P1.5** | Strongest patent-defensible novelties | (a) Attack 10 — interplanetary-RTT-aware authority propagation (already queued for standalone PPA); (b) Attack 27 — URN-scheme-versioned dual-version retention window for interplanetary registry transitions; (c) Root-cause clustering with hard rate limit (F4) under high-RTT mission ops; (d) Theoretical-floor binding for ACI in adversarial environments (T5). All four are non-obvious *given the interplanetary RTT context*. | **POSITIVE** |
| **P1.6** | Disclosure timing | LEDGER #0004 publication via arXiv defensive preprint is the right anchoring strategy. Recommend: file standalone PPAs for (a), (b), (c) **before** arXiv publication to preserve the option of foreign-filing rights (under the Paris Convention 12-month window, foreign filing is preserved if PPA is filed before public disclosure). | **POSITIVE if sequenced correctly** |
| **P1.7** | Trademark posture | "Caduceus" is reasonably generic and may face TM scrutiny in space-tech category. Verify USPTO TESS for prior registrations in IC 9 / IC 42. Likely needs descriptor: "Caduceus™ Interplanetary Connectivity Substrate" rather than bare "Caduceus." | **LOW (admin)** |

### Practitioner recommendations

- **Pre-commit (recommended):** address P1.3 and P1.4 (small spec tightenings) to strengthen enablement. Run TESS check (P1.7).
- **Pre-commit (optional but high-value):** prior-art search on time-bound authority + revocation propagation in distributed-systems literature (P1.1). 2–4 hours of practitioner time.
- **Post-commit:** file three PPAs in sequence (a, b, c per P1.5) **before** arXiv preprint. Synchronize timing.

---

## Persona 2 — Formal-methods reviewer perspective

Framing: credentialed formal-methods researcher (Lean 4 / Coq / TLA+ background) reviewing v1.2 for mechanical-verification feasibility, decidable fragments, and properties resistant to formalization.

### Findings

| # | Area | Finding | Severity |
|---|---|---|---|
| **P2.1** | Mechanically verifiable in Lean 4 / Coq | T1 atomicity (mutual exclusion theorem); T3 fail-safe transitions (state-machine reachability); T7 succession-implies-revocation (temporal logic property: ◊succession ⇒ □revocation_of_predecessor); F4 hard rate limit (counter invariant); §3.6 precedence ordering (relational property). **Estimated 3–6 person-months of Lean 4 work** for a graduate-student-level proof effort. | **POSITIVE** |
| **P2.2** | Verifiable via TLA+ model checking | T4 per-stream quota behavior under arbitrary stream patterns; T5 floor convergence under adversarial calibration; F2 + F4 interaction under non-cooperative crew; §3.5 dual-version retention window correctness under unbounded RTT skew. **Estimated 1–2 person-months** for TLA+ specification + TLC model-checker runs. | **POSITIVE** |
| **P2.3** | Resistant to formalization | "Crew confirmation fatigue" (F4) is empirical psychology, not logic; "theoretical-floor derivation" (T5) is physics, not formal-methods territory. These need empirical validation studies, not proofs. Spec correctly handles this by not over-claiming proof scope. | **POSITIVE** |
| **P2.4** | Soundness coverage gap — F2 closed namespace | F2 v1.2 closes the class namespace. Soundness requires proving that EVERY unenumerated class is rejected, not just that enumerated classes are accepted. A naive implementation can leak edge cases (e.g., capitalization variants, whitespace-stripped, multi-byte-encoded variants). Recommend formal coverage analysis as part of substrate verification. | **LOW** |
| **P2.5** | Liveness vs safety distinction | All v1.2 assertions are safety properties (something bad never happens). Liveness properties (something good eventually happens) are conspicuously absent. For an interplanetary fabric, the most important liveness property is: "every signed bundle from a non-revoked authority is eventually either accepted or definitively rejected within bounded time." This is NOT in v1.2. Could be added as a v1.3 item without breaking the v1.2 commit. | **MEDIUM** |
| **P2.6** | Composability with Mercury kernel proofs | Existing Mercury kernel Lean 4 work (per VBX-ISPS v0.2-α NEXT work item) provides a verified deterministic execution substrate. Caduceus's WCET-bounded paths (HF-20) can compose cleanly on top. Recommended: align proof obligations between Mercury and Caduceus tracks so they reuse infrastructure. | **POSITIVE — opportunity** |
| **P2.7** | Aletheia chain forward-verification proof | T1 v1.2 requires `prev_hash` chain forward-verification. This is the same property class as Bitcoin / blockchain chain validity proofs. Mature Lean/Coq formalizations exist (e.g., Mizuhito Ogawa et al. work; Cardano's Plutus formalization). Reuse pattern. | **POSITIVE — leverage** |

### Reviewer recommendations

- **Pre-commit (optional):** no blocking findings. v1.2 is formal-methods-ready.
- **Post-commit (v0.2 substrate track):** initiate Lean 4 mechanical proof track on T1+T3+T7+F4 state machines, composed with existing Mercury kernel proofs (P2.6). Target: machine-checkable safety proofs published as defensive arXiv supplement at v0.2 release.
- **Post-commit (v1.3 spec track):** add liveness properties (P2.5). Specifically: bounded-time bundle resolution, bounded-time revocation propagation acknowledgment, bounded-time crew-confirmation outcome.

---

## Persona 3 — NASA mission-assurance reviewer perspective

Framing: experienced NASA OSMA (Office of Safety and Mission Assurance) reviewer with EVA + mission-ops background, reviewing v1.2 for crew-safety implications, redundancy, fail-safe semantics, and mission-procedural completeness.

### Findings

| # | Area | Finding | Severity |
|---|---|---|---|
| **P3.1** | F4 K=6/hour rate-limit calibration | Plausible but unvalidated against EVA cognitive-load data. Apollo and Artemis EVAs run 6–8 hours; K=6/hour gives ~36–48 confirmations/EVA, which may exceed reasonable cognitive load. Recommend empirical calibration against historical EVA workload data + ground simulations before LEDGER commit. Current K=6 is mission-policy-set per spec, but spec offers no calibration guidance. | **MEDIUM** |
| **P3.2** | EPHEMERIS-PHRONESIS NFC pairing under EVA gloves | NFC pairing seed requires ~4cm proximity AND fine motor control. Under pressurized EVA gloves, this is hard. Spec assumes pre-mission pairing only — fine for nominal ops, but field re-pair (e.g., after EPHEMERIS battery swap) is essentially impossible in EVA. Need explicit pre-mission-only-pairing or alternative re-pair procedure. | **MEDIUM** |
| **P3.3** | "Degraded-replay-protection" alert (T4) — crew response procedure | Spec defines the alert but not the crew-action procedure. Mission-assurance demands operational response. Spec should reference (even if not specify) a CrewOpsManual entry that says what the crew does when this alert fires. | **MEDIUM** |
| **P3.4** | Single-point-of-failure on PHRONESIS for command authority | NASA standard for crewed-mission-critical systems is dual-fault-tolerance (DFT). v1.2 has PHRONESIS as the single command-authority receiver per crew member. v0.2 dual-peer attestation (mentioned as stretch) should become baseline for any actual crewed deployment, not stretch. This is a v0.2-substrate item, not a v1.2-benchmark issue, but the spec should note that the benchmark is single-fault-tolerant and that DFT requires v0.2+ substrate work. | **HIGH (for crewed deployment) / LOW (for bench commit)** |
| **P3.5** | Aletheia chain single-point storage | If chain storage (flash on Orin NX) corrupts, entire trust fabric collapses. NASA practice: triple-redundant storage for mission-critical state. Caduceus spec is silent on chain redundancy. v0.2-substrate item but should be noted. | **HIGH (crewed) / LOW (bench)** |
| **P3.6** | HF-9 (hard safety floor) behavior is referenced but not specified | T3 says "deterministically falls back to HF-9" but HF-9's actual behavior is in VBX-ISPS-001 blueprint, not in Caduceus spec. Mission-assurance demands self-contained safety-critical specs. Caduceus should either inline the relevant HF-9 behavior or include normative reference language. | **MEDIUM** |
| **P3.7** | HF-9 reversibility — locked-out scenarios | T3 transitions to HF-9 on signature or gate failure. If a single malformed bundle triggers HF-9 entry, is the suit locked out of comms for the rest of the EVA? Spec is silent. NASA practice: any safety-floor entry must have a defined exit procedure (typically crew-confirmation + ground-confirmation). | **HIGH** |
| **P3.8** | False-negative vs false-positive risk class for EVA-termination commands | For life-safety class commands, BOTH error modes are potentially fatal. Spec defaults to conservative-rejection (fail-safe = reject). Under genuinely degraded link, this could strand crew. Need explicit override procedure with documented risk acceptance (e.g., "after K consecutive rejections of legitimate-appearing emergency-class commands, crew may invoke ManualOverride with two-crew-confirmation and ground-after-the-fact audit"). | **HIGH** |
| **P3.9** | Audit completeness — crew confirmation events not explicitly logged | Aletheia chain logs bundle-level events. F4 v1.2 requires crew confirmations but does not assert that confirmations themselves are chain entries. Mission-assurance demands action-level audit. Spec should explicitly require confirmation events to be Aletheia-logged. | **MEDIUM** |
| **P3.10** | Ground-side authority emergency push procedure | If a crew member is incapacitated mid-EVA and another crew member's authority must be elevated for safety, what is the procedure? Spec has no defined emergency-push pathway for authority updates. T7 specifies revocation; the inverse (elevation) is missing. | **MEDIUM** |
| **P3.11** | Graceful degradation under link loss | T3 binary failsafe is appropriate for crypto / authority failures. For pure link-loss (no bundles arriving at all), graceful degradation should differ — crew can continue EVA with reduced ground-link dependence. Spec doesn't differentiate. | **MEDIUM** |

### Reviewer recommendations

- **Pre-commit (recommended):** address P3.6 (HF-9 inline reference language — fixable in <1 hour), P3.9 (one-sentence assertion that crew confirmations are Aletheia-logged). 
- **Pre-commit (advisory):** acknowledge in §0 scope statement that v1.2 is single-fault-tolerant and that dual-fault-tolerance for crewed deployment requires v0.2+ substrate (P3.4, P3.5).
- **Post-commit / v0.2 substrate:** P3.1 empirical K calibration; P3.2 EVA-glove pairing procedure or pre-mission-only policy; P3.3 / P3.7 / P3.8 / P3.10 / P3.11 CrewOpsManual companion document.
- **v1.3 spec track:** liveness properties for HF-9 exit, emergency authority elevation, graceful link-loss degradation.

---

## Synthesis — three categories of findings

### Category A — Pre-commit recommended (small, fixable)

1. **P1.3** — tighten T5 theoretical-floor language for enablement (~1 hr)
2. **P1.4** — stub specification for BPSec/Aletheia key derivation OR explicit defer-to-arXiv-supplement note (~1 hr)
3. **P3.6** — inline HF-9 reference language (~1 hr)
4. **P3.9** — explicit assertion that crew confirmations are Aletheia-logged (~30 min)
5. **P3.4 / P3.5 acknowledgment** — §0 scope note that bench is single-fault-tolerant; DFT is v0.2+ substrate scope (~30 min)

**Total estimated effort: ~4 hours.** Yields a slightly tighter v1.2 (call it v1.2.1) for LEDGER commit. **Recommended but not blocking.**

### Category B — Pre-commit optional, post-commit recommended

1. **P1.1** — prior-art search on time-bound authority + interplanetary-RTT-aware revocation (Kerberos, Burrows-Abadi-Needham, etc.) — 2–4 hours practitioner time
2. **P1.7** — TESS check on "Caduceus" trademark — 30 minutes
3. **P1.6** — PPA filing sequence before arXiv preprint — operational sequencing
4. **P3.1** — empirical K=6 calibration against EVA workload data — 1–2 weeks

### Category C — Post-commit substrate + v1.3 spec track

1. **P2.1 + P2.6 + P2.7** — Lean 4 mechanical proof track for T1+T3+T7+F4, composed with Mercury kernel — 3–6 person-months
2. **P2.2** — TLA+ model checking for T4+T5+F2+F4 interactions, §3.5 transition — 1–2 person-months
3. **P2.5** — liveness properties for v1.3
4. **P3.4 / P3.5** — dual-peer EPHEMERIS attestation as v0.2 baseline; chain triple-redundant storage
5. **P3.2 / P3.3 / P3.7 / P3.8 / P3.10 / P3.11** — CrewOpsManual companion document
6. **External engagement** — pursue actual NASA OSMA / mission-assurance reviewer post-commit; this internal probe is necessary but not sufficient

---

## Commit-eligibility verdict

**caduceus-bench v1.2 is COMMIT-ELIGIBLE for LEDGER #0004.**

No Category A finding is blocking. They are quality-of-life improvements that, if applied, yield v1.2.1 — a marginally stronger commit artifact. Most cost-effective move: apply all 5 Category A items (~4 hours total) and commit v1.2.1.

Category B and C findings are recommended follow-ons but do not block commit. They define the post-commit work plan: patent filings, formal-methods substrate work, CrewOpsManual, external review engagement.

**Recommended commit sequence:**

1. Apply Category A items → v1.2.1
2. Run quick prior-art check (Category B P1.1) — 2–4 hr practitioner time
3. File standalone PPAs (Attack 10 + Attack 27 + F4 root-cause clustering) per P1.5
4. Commit LEDGER #0004 on v1.2.1
5. Publish arXiv defensive preprint
6. Begin Category C substrate + v1.3 work

---

## Honest scope statement (recap)

- This probe is internal-only with diversified framing. Findings are directional, not authoritative.
- Patent practitioner persona: confidence MEDIUM on prior-art findings, HIGH on procedural recommendations.
- Formal-methods persona: confidence HIGH on decidability claims (matches Claude's general capability), MEDIUM on person-month estimates (calibration uncertain).
- Mission-assurance persona: confidence MEDIUM-HIGH on procedural gaps (well-documented in NASA literature), LOW on numeric estimates (K=6 calibration depends on actual EVA data this probe cannot access).
- All three personas converge on: **v1.2 is committable as-is or with the 4-hour Category A tightening; substantive follow-on work is post-commit and parallel.**

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-PROBE-001 Rev A · Move 3 diversified-internal probe*
*Verdict: COMMIT-ELIGIBLE (v1.2 or v1.2.1)*
