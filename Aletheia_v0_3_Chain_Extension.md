# Aletheia v0.3 — Cross-Key-Epoch Chain Extension Specification

| | |
|---|---|
| **Document ID** | ALETHEIA-V03-001 |
| **Revision** | A (draft) |
| **Purpose** | Extend Aletheia DAC chain semantics to support cross-key-epoch forward-verification, a dependency identified by CADUCEUS-005 M1 (open item #12) and required by caduceus-bench v1.2.1 T1 assertion |
| **Predecessor** | Aletheia DAC v0.2 (in production at Visionblox per VBX-ISPS v0.1 substrate) |
| **Methodology** | ZCS-6 (Phase 1 + Phase 2 — first version of aletheia-bench for v0.3) |
| **Author** | A. Khaalis Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University / Visionblox LLC / Zuup Innovation Lab |
| **Date (UTC)** | 2026-06-08 |
| **Status** | Plan-of-work for v0.3 chain extension. Required before caduceus-bench v1.2.1 substrate build can complete M1 milestone. |

---

## 0. Honest scope statement

Aletheia v0.2 is the existing hash-chained provenance ledger used in VBX-ISPS v0.1 substrate and referenced throughout caduceus-bench v1.2.1. This document specifies a v0.3 extension that adds cross-key-epoch forward-verification.

The specification is written in the same ZCS-6 falsification-first discipline as caduceus-bench: extension claims are committed before implementation. The extension introduces a small set of new assertions (six) and is intentionally additive, not a redesign. Aletheia v0.2 entries remain valid in v0.3 chains.

---

## 1. Problem statement

### 1.1 The v0.2 limitation

Aletheia v0.2 maintains a hash chain of signed events. Each event is signed by a release key. The chain assumes a single release-key epoch:

```
Entry N:    { content_N, sig_N = ed25519_sign(key, content_N || hash(Entry N-1)) }
Entry N+1:  { content_{N+1}, sig_{N+1} = ed25519_sign(key, content_{N+1} || hash(Entry N)) }
...
```

Forward-verification works as long as the same key signs every entry. When the release key rotates (key compromise recovery, scheduled rotation, multi-party custody transition), the v0.2 chain has no defined semantics for cross-key entries:

- A new key signing a new entry would still verify against the new key, but the verifier has no proof that the new key is authorized.
- The chain has no record of the rotation event.
- A verifier walking the chain from genesis cannot prove that the chain remained under legitimate custody across the rotation.

### 1.2 The caduceus-bench v1.2.1 requirement

caduceus-bench v1.2.1 T1 specifies: "prev_hash-matches-last-accepted-within-authority-stream." This requires forward-verification across all chain entries the receiver has accepted, regardless of which key signed them. If a key rotation occurred between two received entries, the verifier must be able to verify both.

Without cross-key-epoch support, T1 fails when the release key rotates. This is a real operational concern: long missions (250+ EVAs over ~900 days per VBX-ISPS spec) will encounter key rotations.

### 1.3 The interplanetary RTT complication

Key rotation events propagate at the speed of light, like all other authority-related events. A naive rotation protocol would create a propagation window during which different receivers have different valid keys, similar to the authority-graph staleness problem (Caduceus Attack 10).

The v0.3 extension addresses both: cross-key-epoch verification and RTT-aware rotation propagation.

---

## 2. Specification overview

### 2.1 Six new assertions

| ID | Assertion | Class |
|---|---|---|
| **A1** | Each key-rotation event is itself a chain entry signed by BOTH the outgoing key AND the incoming key (joint signature) | Trust |
| **A2** | Forward-verification across a key-rotation boundary succeeds iff the rotation entry is verifiable under both predecessor and successor keys | Trust |
| **A3** | Key-rotation entries are propagated on a dedicated priority lane (same lane as authority revocations per caduceus-bench T7) | Propagation |
| **A4** | Until a key-rotation entry is verified at a receiver, the receiver accepts entries signed by EITHER the predecessor OR the successor key for a bounded dual-acceptance window | Propagation |
| **A5** | Genesis entries (chain start) are signed by a multi-party custodial signature requiring at least 2-of-N party signatures (rather than a single release key) | Trust |
| **A6** | Backward compatibility: any Aletheia v0.2 chain entry verifies under Aletheia v0.3 verification logic without modification | Compatibility |

### 2.2 New entry types

Three new entry types in v0.3:

| Type | Purpose | Signed by |
|---|---|---|
| **standard-event** | Same as v0.2 entries; any chain content | Current key |
| **key-rotation** | Announces and authorizes key change | Outgoing key AND incoming key (joint) |
| **genesis** | Chain start; sets initial key custody | 2-of-N party multi-signature |

### 2.3 Chain structure

```
Entry 0 (genesis): { content_0, multi_sig_0 = N-of-M signatures from custody parties }
Entry 1: { content_1, sig_1 = ed25519_sign(key_v1, content_1 || hash(Entry 0)) }
Entry 2: { content_2, sig_2 = ed25519_sign(key_v1, content_2 || hash(Entry 1)) }
...
Entry K (key-rotation): { 
    content_K = { "type": "key-rotation", "outgoing": pubkey_v1, "incoming": pubkey_v2, "effective_tai": ... },
    joint_sig_K = (sig_outgoing = ed25519_sign(key_v1, ...), sig_incoming = ed25519_sign(key_v2, ...))
}
Entry K+1: { content_{K+1}, sig_{K+1} = ed25519_sign(key_v2, content_{K+1} || hash(Entry K)) }
...
```

---

## 3. Detailed protocol

### 3.1 Genesis entry construction

The genesis entry is constructed via a custody ceremony involving N parties (recommended N = 5; threshold = 3-of-5 or 2-of-3 depending on Visionblox custody model):

```
genesis_content = {
    "type": "genesis",
    "chain_id": "visionblox-aletheia-v0.3",
    "schema_version": "aletheia-v0.3",
    "initial_release_key": pubkey_v1,
    "custody_parties": [pubkey_party_1, pubkey_party_2, ..., pubkey_party_N],
    "custody_threshold": M,
    "creation_tai": "<TAI epoch>",
    "creation_location": "<Visionblox facility identifier>"
}
genesis_signatures = [
    sig_party_1 = ed25519_sign(privkey_party_1, hash(genesis_content)),
    sig_party_2 = ed25519_sign(privkey_party_2, hash(genesis_content)),
    ...
]
```

At least M of the N custody-party signatures must be present and valid for the genesis to be accepted.

### 3.2 Key-rotation entry construction

When the current release key rotates from v_n to v_{n+1}, the rotation entry is constructed:

```
rotation_content = {
    "type": "key-rotation",
    "outgoing_key": pubkey_v_n,
    "incoming_key": pubkey_v_{n+1},
    "rotation_reason": "<scheduled | compromise-recovery | custody-transition>",
    "effective_tai": "<TAI epoch when v_{n+1} becomes the primary>",
    "dual_acceptance_window_seconds": <e.g., 2 × max-RTT, minimum 3600>,
    "prev_entry_hash": "<hash of prior chain entry>"
}
joint_signature = {
    "outgoing_sig": ed25519_sign(privkey_v_n, canonical(rotation_content)),
    "incoming_sig": ed25519_sign(privkey_v_{n+1}, canonical(rotation_content))
}
```

Both signatures must be present and valid. This proves:
- The outgoing key authorizes its own retirement.
- The incoming key consents to take over.
- No key can be unilaterally introduced or retired.

### 3.3 Verifier protocol

A verifier walking the chain from genesis to current head executes:

```
def verify_chain(entries):
    # Step 1: verify genesis
    assert entries[0]["type"] == "genesis"
    assert count_valid_party_sigs(entries[0]) >= entries[0]["custody_threshold"]
    current_key = entries[0]["initial_release_key"]
    
    # Step 2: walk entries
    for i in range(1, len(entries)):
        entry = entries[i]
        prev_hash = hash(entries[i-1])
        
        if entry["type"] == "standard-event":
            # Verify under current_key
            assert verify_ed25519(current_key, entry["content"], entry["sig"])
            assert entry["content"]["prev_hash"] == prev_hash
        
        elif entry["type"] == "key-rotation":
            # Verify joint signature
            assert entry["content"]["outgoing_key"] == current_key
            assert verify_ed25519(entry["content"]["outgoing_key"], 
                                    entry["content"], 
                                    entry["joint_sig"]["outgoing_sig"])
            assert verify_ed25519(entry["content"]["incoming_key"], 
                                    entry["content"], 
                                    entry["joint_sig"]["incoming_sig"])
            assert entry["content"]["prev_entry_hash"] == prev_hash
            
            # Key rotates
            current_key = entry["content"]["incoming_key"]
        
        else:
            raise InvalidEntryType
    
    return current_key  # the verifier now holds the current legitimate key
```

### 3.4 Dual-acceptance window during rotation propagation

When a key-rotation entry's `effective_tai` is set in the future (allowing propagation across long-RTT links before cutover), receivers operate under both predecessor and successor keys during the window:

```
def is_in_dual_acceptance_window(current_tai, rotation_entry):
    effective = rotation_entry["content"]["effective_tai"]
    window = rotation_entry["content"]["dual_acceptance_window_seconds"]
    return effective <= current_tai <= effective + window

def verify_signature_during_rotation(entry, rotation_entry, current_tai):
    if is_in_dual_acceptance_window(current_tai, rotation_entry):
        # Accept either predecessor or successor signature
        return (verify_ed25519(rotation_entry["outgoing_key"], entry, entry["sig"]) or
                verify_ed25519(rotation_entry["incoming_key"], entry, entry["sig"]))
    elif current_tai > effective + window:
        # Only successor key valid
        return verify_ed25519(rotation_entry["incoming_key"], entry, entry["sig"])
    else:
        # current_tai < effective; only predecessor key valid
        return verify_ed25519(rotation_entry["outgoing_key"], entry, entry["sig"])
```

This mirrors the caduceus-bench v1.2.1 §3.5 dual-version retention window pattern.

### 3.5 Compromise-recovery rotation (out-of-band)

If the outgoing key is compromised and cannot or should not co-sign the rotation, an alternate path is required:

```
emergency_rotation_content = {
    "type": "emergency-key-rotation",
    "compromised_key": pubkey_v_n,  // declared compromised
    "incoming_key": pubkey_v_{n+1},
    "rotation_reason": "compromise-recovery",
    "effective_tai": "<TAI>",
    "prev_entry_hash": "<hash>"
}
emergency_signatures = [
    sig_party_1 = ed25519_sign(privkey_party_1, canonical(emergency_rotation_content)),
    sig_party_2 = ed25519_sign(privkey_party_2, canonical(emergency_rotation_content)),
    ...
]
```

This requires M-of-N custody party signatures (same threshold as genesis), bypassing the outgoing key. Receivers verify M signatures against the custody party set defined in the genesis entry.

After emergency rotation, the chain continues under the new key. The compromised key is permanently marked as such in the chain.

### 3.6 Backward compatibility with v0.2 chains

Aletheia v0.2 entries are valid v0.3 standard-event entries. A v0.3 verifier operating on a v0.2 chain treats all entries as type "standard-event" and verifies them under the implicit single key. If the v0.2 chain is later extended with a v0.3 key-rotation entry, that entry must include a `compatibility_marker` field identifying the v0.2 chain's release key:

```
v03_extension_rotation_content = {
    "type": "key-rotation",
    "outgoing_key": <v0.2 implicit key>,
    "incoming_key": <new v0.3 key>,
    "rotation_reason": "v0.2-to-v0.3-upgrade",
    "effective_tai": "<TAI>",
    "compatibility_marker": "v0.2-chain-id-<value>",
    "prev_entry_hash": "<last v0.2 entry hash>"
}
joint_signature_v02_to_v03 = {
    "outgoing_sig": ed25519_sign(v0.2_key, canonical(content)),
    "incoming_sig": ed25519_sign(v0.3_key, canonical(content))
}
```

---

## 4. Adversarial sweep (pre-commit)

Attacks to survive before Aletheia v0.3 commit:

| # | Attack | Closure mechanism |
|---|---|---|
| AV.1 | Forge a key-rotation entry with one signature only (e.g., insider with access to one key) | A1 joint-signature requirement |
| AV.2 | Rotate to a malicious successor by colluding with custody parties below threshold | A5 multi-party threshold (M ≥ 2 for credible attestation) |
| AV.3 | Replay an old key-rotation entry to revert to a retired key | Chain order constraint + prev_entry_hash linkage |
| AV.4 | Reject legitimate rotation by withholding propagation (DoS) | A3 priority lane; receiver-side staleness alert if chain head has not advanced within mission policy interval |
| AV.5 | Use the dual-acceptance window to inject malicious entries under the old key shortly before cutover | Dual-acceptance window does not retroactively validate; receiver applies window only to entries with TAI within the window |
| AV.6 | Compromise outgoing key during rotation ceremony | Joint-signature ceremony procedure (hardware security module, multi-party witness) — operational, not bench-asserted |
| AV.7 | Emergency rotation when outgoing key is not actually compromised (malicious custody parties take over) | Custody party threshold M must be sufficient that single bad-actor party cannot trigger; chain audit ex post |

All seven attacks closed under the v0.3 specification as drafted.

---

## 5. Implementation notes

### 5.1 Compatibility with existing Aletheia DAC

The v0.3 extension is additive. The existing Aletheia DAC v0.2 code paths remain valid. v0.3 adds:
- New entry-type field handling
- Joint-signature verification logic
- Multi-party threshold signature verification (for genesis and emergency rotation)
- Dual-acceptance window logic
- Compatibility-marker handling for v0.2 → v0.3 upgrade

Estimated implementation effort: 2–3 person-weeks for an experienced Rust developer familiar with the existing Aletheia v0.2 codebase.

### 5.2 Test corpus

Aletheia v0.3 should pass a benchmark suite analogous to caduceus-bench:

| Test class | Test count |
|---|---|
| Genesis entry validation | ~10 (valid + invalid combinations) |
| Standard event verification | ~20 (valid + tampered) |
| Key-rotation entry validation | ~30 (joint sig present/absent, prev_hash matches/breaks, etc.) |
| Dual-acceptance window logic | ~15 (boundary cases) |
| Emergency rotation | ~15 (threshold met/not met, key actually compromised vs not) |
| v0.2 → v0.3 upgrade | ~10 (compatibility marker handling) |
| Adversarial sweep (AV.1–AV.7) | ~21 (3 variants per attack) |
| **Total** | **~121 tests** |

### 5.3 Wire format

Recommend canonical JSON per RFC 8785 (same as Caduceus LEDGER #0004 entry canonicalization). Backward-compatible with v0.2's existing format because v0.3 entries simply add fields; absent fields imply v0.2 single-key semantics.

### 5.4 Hardware security module requirements

For the joint-signature ceremony at key rotation, the outgoing key may be in HSM. The HSM must support:
- Ed25519 signature
- Single-use authorization for the rotation event (HSM operator must explicitly approve)
- Witness signatures (audit log entry, ideally counter-signed by HSM operator's key)

This is operational discipline, not protocol specification.

---

## 6. Aletheia-bench v1.0 (for v0.3 commit)

Following the ZCS-6 discipline, the Aletheia v0.3 specification itself should be cryptographically committed via aletheia-bench v1.0 — a separate manifest from caduceus-bench. Initial scope of aletheia-bench:

- Six assertions (A1–A6 per §2.1 above)
- Seven attack closures (AV.1–AV.7 per §4)
- Test corpus structure (§5.2)
- Implementation API (§5.1)

Commit sequence:
1. Freeze ALETHEIA-V03-001 (this document)
2. SHA-256 hash
3. Ed25519 sign with current Visionblox release key (v1 — the key being extended; this is bootstrapping)
4. Append to existing Aletheia chain as **LEDGER #0005** (sequential after caduceus-bench LEDGER #0004)
5. OpenTimestamps proof
6. Begin v0.3 implementation

---

## 7. Dependency relationship to caduceus-bench v1.2.1

caduceus-bench v1.2.1 substrate build (per CADUCEUS-005 M1) requires Aletheia v0.3 chain semantics to satisfy T1 prev_hash forward-verification across key rotations. The chain extends naturally to support the LEDGER sequence:

- LEDGER #0001–#0003: VBX-ISPS work (v0.2 chain semantics)
- LEDGER #0004: caduceus-bench v1.2.1 commit (still v0.2 chain semantics; single key)
- LEDGER #0005: Aletheia v0.3 spec commit (this document; still v0.2 semantics; bootstraps v0.3)
- LEDGER #0006: first v0.3 key-rotation entry (when needed; transitions chain to v0.3 semantics)

This sequencing means Aletheia v0.3 implementation can proceed in parallel with caduceus-bench v1.2.1 substrate build per CADUCEUS-005 M1, with the first actual key rotation deferrable until needed operationally.

---

## 8. Honest scope statement

- This document specifies the Aletheia v0.3 extension; it does not implement it. Implementation is 2–3 person-weeks of additional Rust work.
- The specification is written against my understanding of Aletheia v0.2 from the project context; actual v0.2 implementation may have nuances that require minor specification adjustments.
- aletheia-bench v1.0 (the benchmark for v0.3) is sketched but not fully written; full bench-style enumeration of all assertions and thresholds is a separate document.
- The v0.3 extension covers the operational case of key rotation in long-duration missions. It does not address quantum-secured signing (post-Ed25519 signature schemes) — that is a v0.4+ horizon.
- Multi-party custody for genesis (A5) assumes Visionblox has a defined custody model with N parties; if not, custody-model development is a prerequisite to v0.3 deployment.
- The dual-acceptance window mechanism (A4) is intentionally analogous to caduceus-bench §3.5 dual-version registry retention. The two patterns share the same RTT-aware design philosophy.

---

*Aldrich K. Wooden, Sr., MBA; MSIT Candidate, Southern New Hampshire University*
*Visionblox LLC · Zuup Innovation Lab*
*2026-06-08 · ALETHEIA-V03-001 Rev A · Aletheia v0.3 cross-key-epoch extension spec*
