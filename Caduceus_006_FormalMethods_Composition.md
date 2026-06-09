# Caduceus × Mercury — Formal-Methods Composition Specification

| | |
|---|---|
| **Document ID** | CADUCEUS-006 |
| **Revision** | A (draft) |
| **Purpose** | Specify how caduceus-bench v1.2.1 formal-methods obligations compose with the existing Mercury kernel Lean 4 / Coq verification track |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Plan-of-work for v0.2 substrate formal-methods track. Parallel to CADUCEUS-005 substrate build. |
| **Predecessors** | Mercury kernel WCET informal proof (5-cycle deterministic) per VBX-ISPS-001 §HF-9; Mercury Lean 4 NEXT work item per VBX-ISPS v0.2-β |

---

## 0. Scope statement

This document specifies the **formal-methods program** for the Caduceus substrate. It does NOT produce the proofs themselves; it specifies what is to be proved, in what proof system, against what predecessor theorems, with what effort budget.

The proofs themselves are the deliverable of the formal-methods track once executed. Estimated total effort: **4–8 person-months** of Lean 4 + TLA+ work, executable in parallel with the Rust substrate build per CADUCEUS-005 M3–M7.

---

## 1. Composition principle

### 1.1 Layered theorem stack

Mercury proves the **deterministic execution substrate**. Caduceus proves **bundle-fabric properties on top of Mercury**. Caduceus theorems import Mercury theorems as lemmas; Caduceus does not re-prove Mercury's WCET bounds or determinism.

```
┌──────────────────────────────────────────────────────────────┐
│ Caduceus theorems (this document)                             │
│  · T1 atomicity            · T3 fail-safe transition          │
│  · T7 succession rule      · F4 rate-limit invariant          │
│  · §3.6 precedence         · T6 ingress-validity order        │
├──────────────────────────────────────────────────────────────┤
│ Mercury theorems (existing VBX-ISPS v0.2-β NEXT work)         │
│  · 5-cycle Subleq deterministic execution                     │
│  · Bounded WCET on instruction sequences                      │
│  · Lloyd-limit-bounded throughput                             │
│  · Memory isolation across compartments                       │
├──────────────────────────────────────────────────────────────┤
│ Lean 4 mathlib base (community)                               │
│  · Naturals, integers, lists, finite sets                     │
│  · State machine framework                                    │
│  · Temporal logic operators                                   │
└──────────────────────────────────────────────────────────────┘
```

### 1.2 What composes vs. what does not

| Property class | Composes via Mercury | Independent (Lean 4 only) | Out of formal scope |
|---|---|---|---|
| WCET bounds | ✓ (Mercury determinism + cycle counts) | — | — |
| Atomicity of operation sequences | ✓ (Mercury's deterministic single-thread semantics + mutex theorems) | — | — |
| Fail-safe state transitions | ✓ (Mercury state machine framework) | — | — |
| Counter / rate-limit invariants | — | ✓ (pure data-structure invariants) | — |
| Relational properties (precedence) | — | ✓ (pure relational) | — |
| Crew cognitive load | — | — | ✗ Empirical, not formalizable |
| Physical link characteristics | — | — | ✗ Physics, not logic |
| Adversarial behavior in general | — | ⚠ Model-checkable via TLA+, not Lean-provable in closed form | (TLA+ track) |

---

## 2. Lean 4 proof obligations (Caduceus side)

### 2.1 T1 atomicity theorem

**Statement (informal):** For any two bundles B1 and B2 arriving concurrently on the same authority stream, the receiver's four-check sequence (signature, lifetime, nonce-not-seen, prev_hash) executes atomically per bundle: either B1 completes all four checks before B2 begins, or B2 completes all four checks before B1 begins. No interleaving of check operations across bundles is possible on the same stream.

**Statement (Lean 4 sketch):**
```lean
theorem t1_atomicity (s : AuthorityStream) (B1 B2 : Bundle) 
    (h1 : B1.stream = s) (h2 : B2.stream = s) :
  (CompletesBeforeStarts s B1 B2) ∨ (CompletesBeforeStarts s B2 B1) := by
  -- Uses Mercury's deterministic single-thread theorem
  -- Uses mutex acquisition theorem for the per-stream lock
  sorry
```

**Mercury dependencies:** `Mercury.SingleThreadDeterminism`, `Mercury.MutexAcquireBounded`.

**Estimated effort:** ~3 person-weeks.

### 2.2 T3 fail-safe transition theorem

**Statement (informal):** For any bundle B that fails signature verification OR fails the Civium gate decision, the receiver transitions deterministically to the HF-9 state. HF-9 is not exitable via any LLM-mediated retry path; exit requires two-source confirmation per the inlined HF-9 behavior in CADUCEUS-004 §3.3 T3.

**Statement (Lean 4 sketch):**
```lean
theorem t3_failsafe_to_hf9 (B : Bundle) (s : ReceiverState) 
    (h : ¬VerifySignature B ∨ ¬CiviumGate B) :
  ∃ s' : ReceiverState, Transition s B s' ∧ s'.mode = HF9 := by
  -- State machine reachability proof
  -- Uses Mercury's bounded-step transition theorem
  sorry

theorem t3_no_llm_retry_from_hf9 (s : ReceiverState) (h : s.mode = HF9) :
  ∀ retry_action : LLMAction, Transition s retry_action s ∨ Transition s retry_action s := by
  -- HF-9 is absorbing under LLM retry; only two-source-confirmation exits
  sorry
```

**Mercury dependencies:** `Mercury.StateMachineReachability`, `Mercury.DeterministicTransition`.

**Estimated effort:** ~4 person-weeks (HF-9 state semantics are nontrivial).

### 2.3 T7 succession-implies-revocation theorem

**Statement (informal):** For any authority A and successor A' issued for the same scope, the receiver locally revokes A upon receipt of A', even if no explicit revocation bundle for A has yet arrived. Formally: a temporal-logic property — eventually-receive-successor implies always-locally-revoke-predecessor.

**Statement (Lean 4 sketch):**
```lean
theorem t7_succession_implies_revocation 
    (A A' : Authority) (h_scope : sameScope A A') (h_diff : A ≠ A') :
  Eventually (LocallyReceives A') → Always (PostReceive → LocallyRevoked A) := by
  -- Temporal logic property
  -- Uses Lean's lean-core or mathlib temporal-logic library if available
  sorry
```

**Mercury dependencies:** `Mercury.MonotonicState` (revocation, once applied, persists).

**Estimated effort:** ~3 person-weeks. Temporal logic requires mathlib extensions; may need a custom temporal-logic library if mathlib doesn't have one.

### 2.4 F4 hard rate-limit invariant

**Statement (informal):** For any crew member C and any time t, the number of distinct confirmation requests presented to C in [t - window, t] is bounded above by K. Bundles whose confirmation would exceed K are rejected outright until next authority-graph refresh.

**Statement (Lean 4 sketch):**
```lean
theorem f4_rate_limit_invariant 
    (C : CrewMember) (t : Time) (K : Nat) (window : Time) :
  count {req : ConfirmationRequest | req.crew = C ∧ req.time ∈ (t - window, t]} ≤ K := by
  -- Pure counter invariant; proof by induction on the counter update logic
  sorry

theorem f4_excess_bundles_rejected 
    (B : Bundle) (C : CrewMember) (h : RequiresConfirmation B C) 
    (h_full : RateLimitFull C) :
  Rejected B := by
  -- Case analysis on rate-limit state
  sorry
```

**Mercury dependencies:** None — pure counter invariant. Independent of Mercury substrate.

**Estimated effort:** ~2 person-weeks. Cleanest theorem in the set.

### 2.5 §3.6 precedence-ordering theorem

**Statement (informal):** When a bundle's payload_class matches multiple rows in the conservative-defaults table, the most-restrictive default applies per the precedence ordering. Formally: a total ordering on the enumerated default behaviors plus a well-founded selection.

**Statement (Lean 4 sketch):**
```lean
theorem section_3_6_most_restrictive_wins 
    (B : Bundle) (matching_rows : List DefaultRow) 
    (h : ∀ r ∈ matching_rows, Matches B r) :
  AppliedDefault B = mostRestrictive matching_rows := by
  -- Pure relational property
  -- Uses well-founded order on DefaultRow
  sorry
```

**Mercury dependencies:** None.

**Estimated effort:** ~1 person-week. Smallest theorem; pure relation.

### 2.6 T6 ingress-validity ordering theorem

**Statement (informal):** For any bundle B arriving at receiver, F1 verification (URN + timestamp envelope) executes before any T-layer verification, B-layer custody decision, or render-layer pass. Out-of-envelope bundles are quarantined without any T/B/render side effect.

**Statement (Lean 4 sketch):**
```lean
theorem t6_ingress_validity_first 
    (B : Bundle) (s : ReceiverState) :
  ∀ effect ∈ {TLayerEffect, BLayerEffect, RenderEffect}, 
    OccursAfter (F1Check B) effect ∨ ¬F1Check B = Reject := by
  -- Pipeline ordering proof
  sorry
```

**Mercury dependencies:** `Mercury.PipelineOrdering`.

**Estimated effort:** ~2 person-weeks.

### 2.7 Combined Lean 4 proof effort

| Theorem | Effort (person-weeks) |
|---|---|
| T1 atomicity | 3 |
| T3 fail-safe | 4 |
| T7 succession | 3 |
| F4 rate-limit invariant | 2 |
| §3.6 precedence | 1 |
| T6 ingress-validity ordering | 2 |
| Lean 4 library plumbing + Mercury composition glue | 3 |
| **Total Lean 4** | **18 person-weeks ≈ 4 person-months** |

---

## 3. TLA+ model-checking obligations

For properties resistant to closed-form Lean proof but tractable for finite-state exploration via TLC model checker.

### 3.1 T4 per-stream quota behavior

**Property:** Under arbitrary stream activity patterns (including adversarial flooding), the nonce-seen-set per-stream quotas hold, cross-stream isolation is preserved, and live-TTL nonces are never evicted.

**Approach:** TLA+ spec models the nonce-set as a function from stream-id to a bounded set; actions add/evict nonces under TTL constraints. Model checker explores all interleavings up to bounded depth (typical: 7–10 nonce operations across 3–4 streams).

**Estimated effort:** ~3 person-weeks.

### 3.2 T5 floor convergence under adversarial calibration

**Property:** Under any sequence of empirical observations (including adversarial clean-channel injection), the effective floor max(empirical, theoretical) never drops below the theoretical bound. The alpha-lower-bound holds.

**Approach:** TLA+ spec models ACI as a function of recent observations; actions are observation events; check that the floor invariant holds across all reachable states.

**Estimated effort:** ~2 person-weeks.

### 3.3 F2 + F4 interaction

**Property:** Under non-cooperative crew behavior (deliberate refusal to confirm, deliberate confirmation of mismatched bundles, deliberate delay), the combined F2 + F4 system maintains the safety invariant (no unauthorized action-inducing bundle ever produces a side-effect) and the liveness invariant (legitimate bundles in matched-frame are processed within bounded time).

**Approach:** TLA+ spec models crew as a non-deterministic actor; safety + liveness verified.

**Estimated effort:** ~3 person-weeks. Largest TLA+ component.

### 3.4 §3.5 dual-version retention window correctness

**Property:** Under unbounded RTT skew between transition-announcer and receivers, the window-based acceptance semantics produce consistent decisions: every bundle is either accepted under exactly one registry version or rejected; no bundle is silently dropped or doubly-accepted.

**Approach:** TLA+ spec models distributed receivers with non-deterministic delivery delays; check the "exactly one or rejected" invariant.

**Estimated effort:** ~2 person-weeks.

### 3.5 Combined TLA+ effort

| Property | Effort (person-weeks) |
|---|---|
| T4 quota behavior | 3 |
| T5 floor convergence | 2 |
| F2 + F4 interaction | 3 |
| §3.5 dual-version window | 2 |
| TLA+ tooling setup, model-checker integration, run-time tuning | 1 |
| **Total TLA+** | **11 person-weeks ≈ 2.5 person-months** |

---

## 4. Properties NOT formalized (out of scope)

The following Caduceus claims are not amenable to formal proof and require empirical validation or are accepted-as-axiom:

| Property | Reason | Validation path |
|---|---|---|
| F4 K=6 calibration against EVA cognitive load | Empirical psychology | Mission simulation studies; see §3.8 of CADUCEUS-004 |
| T5 theoretical-floor derivation from link physics | Physics, not logic | Radio engineering review per §3.7 of CADUCEUS-004 |
| Crew confirmation fatigue behavior | Empirical | Human-factors validation in EVA simulators |
| Adversarial behavior model completeness | Open-ended | Continuous Phase 3 / Phase 6 sweeps; no closed-form proof of "no other attacks exist" |
| Hardware Secure Element tamper resistance | Hardware engineering | Hardware-partner attestation; CC EAL+ certification path |
| BPSec / HKDF key-derivation correctness | Standardized | RFC 5869 + RFC 9172 are themselves the spec; mature implementations exist |

---

## 5. Tooling stack

| Tool | Version | Role |
|---|---|---|
| **Lean 4** | 4.x stable | Theorem prover for §2 obligations |
| **mathlib4** | latest | Mathematical library; provides state machine framework, possibly temporal logic |
| **TLA+ Toolbox** | latest | TLC model checker for §3 obligations |
| **Apalache** | latest | Alternative TLA+ model checker (symbolic; better for unbounded models) |
| **CI integration** | GitHub Actions | Run Lean 4 build + TLA+ model checks on every commit to the spec or substrate |

---

## 6. Composition with Mercury proof track

### 6.1 Mercury's commitments

Per VBX-ISPS v0.2-β NEXT work item, Mercury kernel is to deliver:

- Machine-checked 5-cycle deterministic Subleq execution proof
- Machine-checked WCET upper bounds on instruction sequences of bounded length
- Machine-checked memory-isolation across compartments
- Machine-checked Lloyd-limit-bounded throughput (theoretical, not implementation)

These become Caduceus's imported axioms.

### 6.2 Composition glue (responsibility of Caduceus track)

The Caduceus side must:

- Express Mercury's theorems in a form Caduceus theorems can import
- Define how Caduceus's HF-16 through HF-22 modules sit on Mercury's compartmentalized execution
- Define lock acquisition semantics (for T1 atomicity) in terms of Mercury's deterministic transitions
- Define state machine transitions (for T3 fail-safe) in terms of Mercury's bounded steps

Effort: ~3 person-weeks of glue work (already counted in §2.7 "Lean 4 library plumbing").

### 6.3 If Mercury's proof is informal at v0.2 build time

The substrate build (CADUCEUS-005 M3–M7) does not strictly require Mercury's proof to be machine-checked. If Mercury remains "verified informal" at v0.2 substrate build time, Caduceus can proceed with proofs that import Mercury theorems **as axioms** (not as proved). The full machine-checked composition becomes a v0.3 milestone.

**Decision point:** at M4 (CADUCEUS-005 Week 9–13), assess Mercury's proof status. If machine-checked, proceed with composition; if informal, defer composition to v0.3 with axiomatized Mercury imports for v0.2.

---

## 7. Total formal-methods effort

| Track | Effort (person-months) |
|---|---|
| Lean 4 (§2) | 4 |
| TLA+ (§3) | 2.5 |
| Mercury composition glue (§6.2) | 0.75 |
| **Total** | **7.25 person-months ≈ 6 calendar months at 1.2 FTE** |

Matches the diversified-probe estimate (3–6 person-months Lean + 1–2 person-months TLA+) at the upper bound.

---

## 8. Publication strategy

The proof artifacts have publication value:

1. **Defensive arXiv supplement** alongside the LEDGER #0004 arXiv preprint — mechanical proofs anchor the bench claims.
2. **Lean Together workshop** — annual Lean conference; appropriate venue for the Caduceus × Mercury composition story.
3. **Formal Methods in System Design** (journal) or **Journal of Automated Reasoning** — for the substantial proof work.
4. **IEEE Aerospace Conference** — for the systems context (per VBX-ISPS publication target).
5. **SOSP / OSDI** workshop track — for the substrate-systems story.

Recommended: publish the Lean 4 proof repository as open-source on GitHub (e.g., `Visionblox/caduceus-proofs`), with the proof artifacts cross-referenced in the LEDGER #0004 arXiv preprint and the substrate code.

---

## 9. Open items

1. **Mercury proof readiness gate** — assess at M4 per §6.3.
2. **Temporal-logic library in mathlib4** — verify availability; if absent, implement minimal temporal-logic primitives in the Caduceus proof repo.
3. **Formal-methods researcher engagement** — 0.5 FTE for 6 months; candidates: PhD students at SNHU CS or partner institutions; alternative: contract via Galois Inc. or similar formal-methods firms.
4. **CI/CD pipeline for proof maintenance** — proofs must be re-run on every substrate or bench change; budget for build infrastructure.
5. **External review of proof artifacts** — independent formal-methods reviewer (e.g., at Oxford KeySpace authors' lab, given their adjacent work; or at MIT CSAIL Bristol) before publication.

---

## 10. Honest scope statement

- This document specifies the proof program; it does not execute the proofs.
- 7.25 person-months estimate is for an experienced Lean 4 + TLA+ practitioner; first-time-in-Lean engineers would take 2–3× longer.
- Mercury proof status determines whether v0.2 ships with full machine-checked composition or with axiomatized Mercury imports. Decision deferred to M4.
- Properties in §4 (out-of-scope for formalization) are NOT failures of the program — they are categories of property that formal methods are inappropriate for, and require empirical / engineering review instead.
- The proof artifacts are valuable independent of the substrate build. They are also valuable independent of the patent track — formal proofs cannot anticipate prior art but they can anchor enablement claims in patent applications and reduce § 112 enablement risk.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-006 Rev A · Mercury × Caduceus formal-methods composition spec*
