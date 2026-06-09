# Contributing to Caduceus

Thank you for considering a contribution to Caduceus. This document explains how to contribute effectively.

---

## Code of conduct

By participating in this project, you agree to abide by the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

---

## Three things to understand before contributing

### 1. ZCS-6 ordering discipline is non-negotiable

Caduceus is developed under the [ZCS-6 falsification-first methodology](GOVERNANCE.md#zcs-6-methodology). The discipline is:

- The benchmark is **specified and cryptographically committed before any substrate exists**.
- When attacks succeed, the substrate is fixed at the architecture level — the benchmark is **never silently softened**.
- Bench revisions are versioned re-commits with explicit delta justification and a new Aletheia chain entry.

Contributions that violate this ordering (e.g., proposing to weaken an assertion threshold to accommodate an implementation difficulty) will be redirected to either an architecture-level substrate fix or a properly-versioned bench-revision proposal.

### 2. Epistemic markers are required on every claim

All technical content must carry one of three markers:

- **VERIFIED** — grounded in cited academic research, verified data, or executed test results
- **PLAUSIBLE** — logically sound but requires validation
- **SPECULATIVE** — theoretical, requires empirical testing

These markers appear in the text alongside the claim they qualify. Contributions that mix verified and speculative claims without marking them will be requested for revision.

### 3. Cross-document linkage matters

Caduceus is a methodology repository, not a codebase. Documents form a connected graph: bench specifications cite Phase 3 attack reports, attack reports cite the bench, substrate plans cite both, etc. When you add a new document or modify an existing one, update the cross-references in all linked documents. The [README.md](README.md) repository layout section enumerates the document map.

---

## Types of contributions welcomed

| Type | Examples | Process |
|---|---|---|
| **Documentation improvements** | Typos, clarifications, broken-link fixes | Pull request |
| **Honest scope expansions** | New limitations identified; new POA&M entries | Pull request with rationale |
| **Phase 3 / Phase 6 attack proposals** | New named attacks against bench or substrate | Issue → discussion → PR with attack mechanism, targeted assertions, and closure proposal |
| **Substrate-side findings** | Discovered defects in proposed substrate plan | Issue first; substrate plan revision through formal review |
| **Federal-capture intel** | Solicitations identified; consortium updates | PR to [docs/CADUCEUS-CAPTURE-001.md](docs/CADUCEUS-CAPTURE-001.md) |
| **Formal-methods proofs** | Lean 4 / TLA+ proof artifacts | PR with proof + machine-checked verification log |

---

## What's NOT accepted as a contribution

- **Silent bench softening.** If your contribution lowers an assertion threshold or removes an assertion without versioned re-commit, it will be rejected.
- **Claims without epistemic markers.** Mixed-confidence claims are revised before merge.
- **PPA-disclosure-adjacent material before filings complete.** See [SECURITY.md](SECURITY.md) — pre-PPA public disclosure forfeits foreign-filing rights.
- **Unattributed text copy-paste from external sources.** All quoted external content must be cited per the [CITATION.cff](CITATION.cff) and bibliography conventions.

---

## Pull request process

### Before opening a PR

1. **Search existing issues and PRs** to avoid duplication.
2. **Open an issue first** for non-trivial changes — give maintainers a chance to align on direction.
3. **Run any document validators** (link checkers, markdown linters) if your change touches structural documents.
4. **Update the [CHANGELOG.md](CHANGELOG.md) Unreleased section** with your change.

### PR submission

1. Fork the repository and create a topic branch:
   `git checkout -b your-name/short-description`
2. Make your changes in atomic commits — each commit a logical unit of work.
3. **Sign off** every commit with `git commit -s` (Developer Certificate of Origin; see below).
4. Push your branch and open a PR against `main`.

### PR title format

```
type(scope): brief description

Where type ∈ {docs, spec, attack, substrate, capture, ip, governance}
And scope is the document family or component affected.
```

Examples:
- `docs(README): fix broken link to CADUCEUS-005`
- `attack(phase6): propose AV6.34 — substrate behavior under cosmic-ray-induced memory error`
- `spec(bench): propose v1.3.0 — bench revision to address Attack 27 enablement gap`

### PR review

PRs are reviewed against:

- **Ordering discipline** — does this respect ZCS-6?
- **Epistemic honesty** — are claims appropriately marked?
- **Cross-reference integrity** — are linked documents updated?
- **Scope appropriateness** — does this belong in the Caduceus repo vs. a parent or sibling project?

PRs that change bench-asserted behavior require a versioned re-commit ceremony (see [GOVERNANCE.md](GOVERNANCE.md)).

---

## Developer Certificate of Origin (DCO)

By signing off your commits with `git commit -s`, you certify the following per the [DCO v1.1](https://developercertificate.org):

```
By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I
    have the right to submit it under the open source license
    indicated in the file; or

(b) The contribution is based upon previous work that, to the best
    of my knowledge, is covered under an appropriate open source
    license and I have the right under that license to submit that
    work with modifications, whether created in whole or in part
    by me, under the same open source license (unless I am
    permitted to submit under a different license), as indicated
    in the file; or

(c) The contribution was provided directly to me by some other
    person who certified (a), (b) or (c) and I have not modified
    it.

(d) I understand and agree that this project and the contribution
    are public and that a record of the contribution (including all
    personal information I submit with it, including my sign-off) is
    maintained indefinitely and may be redistributed consistent with
    this project or the open source license(s) involved.
```

A signed-off commit appears as:
```
Signed-off-by: Your Name <your.email@example.com>
```

---

## Style conventions

### Document naming

- Spec documents: `CADUCEUS-NNN.md` (sequential numbering)
- Phase 3 sweep reports: `CADUCEUS-PHASE3-NNN.md`
- Patent application drafts: `CADUCEUS-IP-NNN.md`
- Topical documents: descriptive name (e.g., `CADUCEUS-PRACTITIONER-001.md`, `CADUCEUS-LEDGER-PREP-001.md`)
- Parent-system extensions: `<PARENT>-Vxx-NNN.md` (e.g., `ALETHEIA-V03-001.md`)

### Document headers

Every document starts with a metadata table:

```markdown
| | |
|---|---|
| **Document ID** | CADUCEUS-NNN |
| **Revision** | A |
| **Purpose** | One-sentence description |
| **Author** | A. Khaalis Wooden, Sr., ... |
| **Date (UTC)** | YYYY-MM-DD |
| **Status** | Status statement |
```

### Tone

- Honest, direct, no hedging where claims are well-grounded.
- Acknowledge limitations openly in an "Honest scope statement" section.
- No marketing language. The repository is technical/methodological, not promotional.

### Cross-references

Always link to other documents in the repo using relative paths from the repo root:
- `[CADUCEUS-005](docs/CADUCEUS-005.md)` from the repo root
- `[CADUCEUS-005](CADUCEUS-005.md)` from within `docs/`

---

## Questions

Open a [GitHub Discussion](../../discussions) for general questions, or contact the maintainers directly via the addresses in [README.md](README.md).

For security-sensitive matters, see [SECURITY.md](SECURITY.md) — do not open public issues.

---

*This contributing guide is itself open to contributions. PRs welcome to improve clarity, add examples, or refine process.*
