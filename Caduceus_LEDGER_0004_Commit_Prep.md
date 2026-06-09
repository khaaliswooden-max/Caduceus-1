# LEDGER #0004 — Commit Prep

| | |
|---|---|
| **Document ID** | CADUCEUS-LEDGER-PREP-001 |
| **Revision** | A |
| **Purpose** | Provide the cryptographic artifacts and operational sequence for executing LEDGER #0004 commit of caduceus-bench v1.2.1 (CADUCEUS-004) to the Aletheia chain |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | **Pre-signature artifact.** SHA-256 computed; signed-manifest text prepared. Awaiting (a) PPA filing completion per CADUCEUS-PRACTITIONER-001 §5 and (b) signing operation with Visionblox release key. |

---

## 0. Computed cryptographic data

The CADUCEUS-004 manifest (`Caduceus_bench_v1_2_1_Specification.md`) was hashed using `sha256sum` on the canonical UTF-8 byte sequence of the file as written. Results:

| Field | Value |
|---|---|
| **Manifest file** | `Caduceus_bench_v1_2_1_Specification.md` |
| **Byte count (UTF-8)** | 20,379 |
| **SHA-256** | `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874` |
| **MD5 (cross-check only — not used for commit)** | `ce34c2d0da53b3d38a2c61efc648d864` |

**Cross-verification step (for the signer):** before signing, the signer should independently re-hash the manifest using their own `sha256sum` (or equivalent) and confirm the digest matches `0fe3b3...28f874`. Any byte-level change to the manifest — including trailing newlines, line-ending normalization, or whitespace — will produce a different hash and invalidate this commit prep.

---

## 1. Chain position

Per project memory:

| Ledger entry | Subject | SHA-256 (partial) |
|---|---|---|
| LEDGER #0001 | VBX-ISPS-BENCH v1.0 | — |
| LEDGER #0002 | VBX-ISPS-BENCH v1.1 | `58bf8744…` |
| LEDGER #0003 | VBX-ISPS substrate v0.1 (31 artifacts hashed) | — |
| **LEDGER #0004 (this commit)** | **caduceus-bench v1.2.1 (CADUCEUS-004)** | `0fe3b32db5522db6…` |

The new entry's `prev_ledger_hash` field MUST reference the actual hash of LEDGER #0003 from the existing Aletheia chain. Khaalis fills this in from the production chain at signing time; the placeholder below is `<PREV_LEDGER_HASH_FROM_AUTHORITATIVE_CHAIN>`.

---

## 2. Canonical signed-payload artifact

The following is the canonical JSON payload to be Ed25519-signed by the Visionblox release key. Field ordering is alphabetical per key name (JSON canonicalization per RFC 8785 JCS — JSON Canonicalization Scheme). The signer constructs this payload, signs it, and appends to the chain.

```json
{
  "author": "A. Khaalis Wooden, Sr.",
  "commit_date_tai": "<TO_BE_FILLED_AT_SIGNING — ISO 8601 with explicit Z>",
  "commit_methodology": "ZCS-6 Phase 4",
  "document_id": "CADUCEUS-004",
  "document_revision": "A",
  "document_title": "Caduceus v0.1 / caduceus-bench v1.2.1 Specification",
  "ledger_entry_number": 4,
  "manifest_byte_count": 20379,
  "manifest_hash": "0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874",
  "manifest_hash_algorithm": "SHA-256",
  "predecessor_documents": [
    {
      "document_id": "CADUCEUS-001",
      "version": "caduceus-bench v1.0"
    },
    {
      "document_id": "CADUCEUS-002",
      "version": "caduceus-bench v1.1"
    },
    {
      "document_id": "CADUCEUS-003",
      "version": "caduceus-bench v1.2"
    }
  ],
  "prev_ledger_hash": "<PREV_LEDGER_HASH_FROM_AUTHORITATIVE_CHAIN>",
  "prev_ledger_number": 3,
  "schema_version": "aletheia-ledger-entry-v1",
  "signing_algorithm": "Ed25519",
  "signing_key_identifier": "visionblox-release-key-v1"
}
```

**Two fields require fill-in at signing time:**

1. `commit_date_tai` — the actual TAI epoch of signing, in ISO 8601 format with explicit `Z` suffix (e.g., `"2026-06-09T14:23:17Z"`). Should match the actual signing moment to within reasonable precision.
2. `prev_ledger_hash` — the SHA-256 of LEDGER #0003's canonical signed-payload, retrieved from the authoritative Aletheia chain.

---

## 3. Signing operation (step-by-step)

These are the operational steps for the signer (Khaalis with the Visionblox release key, or a designated key custodian).

### Step 3.1 — Independent hash verification

```bash
# At the signer's machine:
sha256sum Caduceus_bench_v1_2_1_Specification.md

# Expected output (verify exact match):
# 0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874  Caduceus_bench_v1_2_1_Specification.md
```

If the hash does not match exactly, **STOP**. Investigate the byte-level difference (likely line-ending normalization from Git or text editor). Resolve before proceeding.

### Step 3.2 — Retrieve LEDGER #0003 prev_hash

From the authoritative Aletheia chain (file or service maintained per VBX-ISPS substrate v0.1):

```bash
# Example (adapt to actual Aletheia tooling):
aletheia chain show --entry 3 --field hash
# Returns: <hash value for LEDGER #0003>
```

Record the value; substitute into the canonical payload's `prev_ledger_hash` field.

### Step 3.3 — Construct the canonical payload

Take the JSON template from §2 of this document. Fill in:

- `commit_date_tai`: current TAI epoch in ISO 8601 with `Z`.
- `prev_ledger_hash`: from Step 3.2.

Then canonicalize the JSON per RFC 8785 (sorted keys, no extra whitespace, UTF-8 encoded). Tools:

```bash
# Using a JCS canonicalizer (e.g., the npm package "canonicalize"):
cat ledger_entry_004.json | jcs > ledger_entry_004_canonical.json

# OR using Python:
python3 -c "import json, sys; data = json.load(sys.stdin); print(json.dumps(data, sort_keys=True, separators=(',', ':'), ensure_ascii=False), end='')" < ledger_entry_004.json > ledger_entry_004_canonical.json
```

### Step 3.4 — Sign with Ed25519

Using the Visionblox release private key:

```bash
# Using libsodium / minisign / Python cryptography library — example with Python:
python3 -c "
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
import base64

# Load the Visionblox release private key (from secure storage; NEVER commit to repo)
with open('/secure/visionblox-release-key.pem', 'rb') as f:
    private_key = Ed25519PrivateKey.from_private_bytes(f.read()[:32])  # adapt to actual key format

# Read the canonical payload
with open('ledger_entry_004_canonical.json', 'rb') as f:
    payload = f.read()

# Sign
signature = private_key.sign(payload)

# Base64-encode for storage
print(base64.b64encode(signature).decode())
"
```

The output is the 64-byte Ed25519 signature, base64-encoded. Record this value.

**CRITICAL operational discipline:**

- The Visionblox release private key MUST NEVER appear in plaintext in any repository, log, or chat.
- Use a hardware security module (HSM) or air-gapped signing workstation if available.
- The signing operation should be witnessed by at least one second party for audit traceability (consistent with §0 of CADUCEUS-004's key-authority asymmetry posture for ground master keys; the Visionblox release key is the ground-side master).

### Step 3.5 — Construct the signed ledger entry

```json
{
  "canonical_payload": "<base64-encoded canonical payload bytes from Step 3.3>",
  "ed25519_signature": "<base64-encoded signature from Step 3.4>",
  "ed25519_public_key": "<base64-encoded public key for verifier convenience>"
}
```

Append this entry to the Aletheia chain at position #0004.

### Step 3.6 — OpenTimestamps proof

Submit the signed ledger entry to the OpenTimestamps network for third-party timestamping:

```bash
# Using the OpenTimestamps client:
ots stamp ledger_entry_004_signed.json
# Wait for confirmation (~few hours for Bitcoin anchor)
ots upgrade ledger_entry_004_signed.json.ots
ots verify ledger_entry_004_signed.json.ots
```

The resulting `.ots` proof file is the third-party timestamping anchor. Store alongside the ledger entry.

### Step 3.7 — Publish manifest hash publicly

After signing and OpenTimestamps proof completion:

- Push the signed ledger entry to a public Aletheia chain repository.
- Cross-link the SHA-256 hash from the arXiv defensive preprint (when published) so external readers can verify.

---

## 4. Sequencing constraints (DO NOT VIOLATE)

Per CADUCEUS-PRACTITIONER-001 §5 operational sequencing, the order of these events affects foreign-filing rights and prior-art establishment:

| Step | When | Reason |
|---|---|---|
| **a.** PPA filings (IP-001, IP-002, IP-003) | First | Establish priority under Paris Convention; PRECEDE any public disclosure |
| **b.** LEDGER #0004 commit (this document) | After (a) | Cryptographic commit can occur immediately after PPA filings; the commit itself is not a "publication" under § 102, but conservatively pair with PPA timing |
| **c.** arXiv defensive preprint | After (b) | Publishing the spec publicly is the disclosure event; PPA filings already establish priority |

**Common error to avoid:** publishing the arXiv preprint before PPA filings starts the foreign-filing clock and forfeits Paris Convention 12-month rights in non-grace-period jurisdictions.

**Recommended cadence:** PPA filings → wait 1 business day → LEDGER #0004 commit → wait 1 business day → arXiv preprint.

---

## 5. Post-commit operational items

After successful LEDGER #0004 commit:

1. **Update project memory** (Aletheia state): record LEDGER #0004 entry hash, replace the placeholder in any references.
2. **Announce internally** to Visionblox / Zuup collaborators.
3. **Cross-reference in subsequent documents**: any new document derived from caduceus-bench v1.2.1 should reference `LEDGER #0004 hash <full hash>` for chain linkage.
4. **Begin v0.2 substrate build per CADUCEUS-005** — formal Phase 5 work starts.
5. **Begin formal-methods track per CADUCEUS-006** — Lean 4 / TLA+ proofs initiated.
6. **Federal capture posture activation per CADUCEUS-CAPTURE-001 §8** — NSTXL / SOSSEC membership, DIU CSO subscriptions, etc.

---

## 6. If recommitment is required

The committed manifest is immutable. If a material correction is required:

- Issue a new document ID (e.g., CADUCEUS-005 or CADUCEUS-004-Rev-B if minor).
- Compute new SHA-256 against the corrected manifest.
- Construct a new ledger entry (LEDGER #0005) with `prev_ledger_hash` pointing to LEDGER #0004 and a `supersedes` field identifying the corrected manifest.
- Sign per §3.

The chain is append-only. Errors do not unwind; they get corrected via subsequent entries with explicit `supersedes` linkage.

---

## 7. Honest scope statement

- This document provides the cryptographic artifacts and operational sequence. **It does NOT execute the signing operation.** Khaalis (or designated key custodian) executes per §3 using the actual Visionblox release private key.
- The SHA-256 hash `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874` corresponds to the exact byte sequence of `Caduceus_bench_v1_2_1_Specification.md` as it exists in this session's `/mnt/user-data/outputs/` directory. Any byte-level modification (including downloading, line-ending changes, or BOM addition) will change the hash.
- The canonical JSON payload in §2 is a recommended structure; Visionblox may adopt a different canonical schema as long as the schema is consistent across the chain.
- OpenTimestamps proofs typically anchor to Bitcoin via the public OpenTimestamps service; alternative timestamping authorities can be substituted if Visionblox has a preferred provider.
- The signing operation requires the Visionblox release private key. This document does NOT and MUST NOT contain that key.

---

## 8. Quick-reference summary

| | |
|---|---|
| **Manifest to commit** | `Caduceus_bench_v1_2_1_Specification.md` (CADUCEUS-004) |
| **SHA-256** | `0fe3b32db5522db65dda8dbc46b1aa9fcc40c418a21cc8abb2df7340a528f874` |
| **Byte count** | 20,379 |
| **Ledger entry number** | #0004 |
| **Predecessor entry** | #0003 (VBX-ISPS substrate v0.1) |
| **Signing key** | Visionblox release private key (Ed25519) |
| **Pre-sign sequencing constraint** | PPA filings must complete first |
| **Post-sign sequencing constraint** | arXiv preprint publishes only after signing |
| **Audit anchor** | OpenTimestamps proof on the signed entry |

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · CADUCEUS-LEDGER-PREP-001 Rev A · Pre-signature commit prep for LEDGER #0004*
*Status: artifacts ready; signing operation awaits PPA filing completion and key-custodian action*
