# Security policy

## Scope

This security policy covers:

- **Caduceus specifications** (caduceus-bench v1.x and successor documents)
- **Caduceus reference substrate** (when implemented per [docs/CADUCEUS-005.md](docs/CADUCEUS-005.md))
- **Aletheia chain artifacts** (LEDGER entries, manifest hashes, signatures)
- **Cryptographic key custody and rotation procedures** (when implemented per [docs/ALETHEIA-V03-001.md](docs/ALETHEIA-V03-001.md))

This policy does NOT cover the operational deployment of Caduceus by third parties. Third-party deployers are responsible for their own operational security per their organizational requirements.

---

## Reporting a vulnerability

### Do not file public issues for vulnerabilities

If you believe you have discovered a vulnerability in Caduceus, please **do not** open a public GitHub issue. Public disclosure of an unaddressed vulnerability could place users and downstream deployers at risk.

### Disclosure channels

Email **khaalis.wooden@visionblox.com** with subject line beginning `[SECURITY]`.

Where practical, encrypt the contents with the Visionblox security PGP key (fingerprint published at [visionblox.com/security.txt](https://visionblox.com/security.txt) — placeholder until live).

### Information to include

1. **Affected component** — bench assertion ID, substrate module, chain entry, or governance procedure
2. **Vulnerability description** — what the attacker can do
3. **Reproduction steps** — minimal evidence sufficient to confirm
4. **Suggested mitigation** — if you have one (optional)
5. **Your name and affiliation** — for credit, unless you prefer anonymity
6. **Disclosure preference** — coordinated disclosure timeline (default 90 days from acknowledgment)

### Acknowledgment timeline

- Within **3 business days** — acknowledgment of receipt
- Within **15 business days** — initial assessment and severity classification
- Coordinated disclosure window default: **90 days** from acknowledgment

We will keep you informed of progress through the disclosure window.

---

## Severity classification

Caduceus classifies vulnerabilities into four levels:

| Severity | Definition | Example | Response |
|---|---|---|---|
| **Critical** | Active exploitation feasible; impacts authority decisions | A flaw in T1 verification allowing arbitrary command bundles to pass | Immediate patch; emergency LEDGER entry |
| **High** | Theoretical exploitation; impacts compliance posture | A T7 stale-graph default that opens a window under specific RTT geometry | Patch in next bench revision (versioned re-commit); arXiv supplement |
| **Medium** | Defensive degradation; no direct compromise | An F4 root-cause clustering bypass that increases crew confirmation load but does not enable unauthorized actions | Patch in next planned revision; CHANGELOG entry |
| **Low** | Documentation or operational deficiency | Ambiguous wording in a §3.6 default that could lead to differing implementations | Documentation update in next revision |

Critical and High severity findings, once mitigation is published, generate a **bench revision** per [GOVERNANCE.md](GOVERNANCE.md) — a versioned re-commit (e.g., v1.2.1 → v1.3.0) with explicit delta justification.

---

## Cryptographic key handling

### Visionblox release key (Aletheia chain signing)

The Visionblox release private key is the master signing key for LEDGER entries. **It must never appear in plaintext** in any repository, log, build artifact, CI/CD configuration, or AI-assistant conversation.

Operational practice:
- Held in a Hardware Security Module (HSM) or air-gapped signing workstation
- Signing operations require witnessed ceremony per [docs/CADUCEUS-LEDGER-PREP-001.md](docs/CADUCEUS-LEDGER-PREP-001.md) §3
- Rotation governed by [docs/ALETHEIA-V03-001.md](docs/ALETHEIA-V03-001.md) joint-signature protocol
- Emergency compromise-recovery via M-of-N custody-party multi-signature per §3.5 of ALETHEIA-V03-001

### Custody party keys (genesis and emergency rotation)

When the Aletheia v0.3 chain genesis ceremony is executed:
- N party keys are generated and held in independent HSMs or hardware wallets
- M-of-N threshold (recommended M=3 of N=5) required for genesis or emergency rotation
- No single party holds quorum

### BPSec session keys (derived)

Per CADUCEUS-004 B2 assertion:
- Derived from Aletheia parent signing keys via HKDF (RFC 5869)
- Per-session derivation is one-way
- Compromise of a BPSec session key does NOT compromise the Aletheia parent
- No separate BPSec key infrastructure is maintained

### Reporting suspected key compromise

If you suspect any Caduceus-associated key has been compromised:

1. **Do NOT publicly disclose the suspicion** until coordinated.
2. Email khaalis.wooden@visionblox.com with subject `[SECURITY-CRITICAL]`.
3. Visionblox will execute emergency key rotation per ALETHEIA-V03-001 §3.5 if confirmed.
4. The compromised key is permanently marked in the chain.
5. Affected LEDGER entries are reviewed; downstream deployers are notified.

---

## Supply-chain security

### Dependencies

Caduceus reference substrate (when built per CADUCEUS-005) uses these primary open-source dependencies:

| Component | Source | Verification |
|---|---|---|
| ION-DTN | NASA / open source | Signed releases; reproducible builds |
| libsodium | jedisct1 | Signed releases; SBOM included |
| Rust toolchain | rust-lang.org | Verified rustup installer |
| HKDF (via libsodium) | RFC 5869 | Part of libsodium |
| OpenTimestamps | opentimestamps.org | Verified client binaries |

Supply-chain integrity practices:
- All dependencies declared in lockfiles with cryptographic hashes
- SBOM (CycloneDX 1.6) maintained per OMB M-25-22 / AIBOM compliance
- Reproducible builds for all Rust artifacts
- Hardware Secure Element used for sensitive crypto operations

### Phase 6 supply-chain attacks (Category F)

The Phase 6 attack-sweep plan ([docs/CADUCEUS-007.md](docs/CADUCEUS-007.md)) includes four supply-chain-class attacks:

- **AV6.23** — Rust toolchain backdoor / cargo dependency compromise
- **AV6.24** — ION-DTN open-source contributor supply-chain attack
- **AV6.25** — Orin NX hardware Trojan
- **AV6.26** — EPHEMERIS firmware compromise via OTA update

These attacks must be executed against the v0.2 substrate before crewed-deployment authorization.

---

## Disclosure and credit

Researchers who responsibly disclose vulnerabilities will receive:

- Acknowledgment in the relevant CHANGELOG entry (if desired)
- Acknowledgment in the IEEE paper or arXiv preprint that addresses the finding (if applicable)
- Citation in academic / industry publications discussing the finding (per researcher's preference)

For coordinated disclosures of significant Critical or High severity findings:
- Joint publication / arXiv preprint
- CVE assignment (where applicable)
- Cross-citation in the bench revision delta-justification record

---

## Bug bounty

Caduceus does not currently operate a paid bug bounty program. We may establish one once the substrate reaches v0.2 deployment scale.

Bench-level findings (Phase 3 attacks against the specification) are recognized and credited but not monetarily compensated under the current research-stage posture.

---

## Operational security incidents (Visionblox internal)

For Visionblox internal security incidents (not vulnerability reports from external researchers), see internal incident response documentation. The CMMC L2 Incident Response (IR) control family applies per [docs/CADUCEUS-CMMC-001.md](docs/CADUCEUS-CMMC-001.md).

---

## Related documents

- [GOVERNANCE.md](GOVERNANCE.md) — governance model and bench revision procedures
- [docs/CADUCEUS-CMMC-001.md](docs/CADUCEUS-CMMC-001.md) — CMMC L2 self-assessment framework
- [docs/CADUCEUS-LEDGER-PREP-001.md](docs/CADUCEUS-LEDGER-PREP-001.md) — LEDGER signing operation
- [docs/ALETHEIA-V03-001.md](docs/ALETHEIA-V03-001.md) — Cross-key-epoch chain extension
- [docs/CADUCEUS-007.md](docs/CADUCEUS-007.md) — Phase 6 attack catalog including supply-chain class

---

## Policy versioning

This security policy is itself versioned. Material changes require a pull request with rationale; minor clarifications can be merged with maintainer approval.

Current revision: **A** — initial security policy as of 2026-06-09.

---

*Maintainer: A. Khaalis Wooden, Sr., MBA — Visionblox LLC / Zuup Innovation Lab*
*Contact for security matters: khaalis.wooden@visionblox.com (subject: `[SECURITY]`)*
