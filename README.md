# PP-SPEC-003 · ProofBundle Format Specification

**Document ID:** PP-SPEC-003  
**Version:** 0.1 - Draft  
**Status:** Draft  
**License:** CC BY 4.0  
**Maintained by:** Proof Economy Standards Alliance (PESA)  
**Repository:** https://github.com/proofprotocol/proofbundle-spec  
**Published:** 2026-07-13  

---

## Abstract

This specification defines the ProofBundle: the portable, self-contained, independently verifiable evidence artifact produced under the Proof Protocol. A ProofBundle is the atomic unit of proof in the Proof Economy. It contains everything a third party needs to verify a claim independently — with no chain, no account, no service call required.

A ProofBundle is not a receipt. A receipt proves an action was observed. A ProofBundle proves a product or agent behaved as claimed under independently witnessed, pre-committed conditions.

---

## Status of This Document

This document is a draft specification of the Proof Protocol. It is released under the Creative Commons Attribution 4.0 International License (CC BY 4.0). You are free to share, implement, and build on it for any purpose including commercially, provided attribution is given to Nebulonium, Inc. / HACKERverse and the Proof Economy Standards Alliance (PESA).

---

## Table of Contents

1. [Motivation](#1-motivation)
2. [Terminology](#2-terminology)
3. [ProofBundle Structure](#3-proofbundle-structure)
4. [Required Fields](#4-required-fields)
5. [Optional Fields](#5-optional-fields)
6. [Verification Procedure](#6-verification-procedure)
7. [Relationship to Pipelock Audit Packet v0](#7-relationship-to-pipelock-audit-packet-v0)
8. [ProofRegister Anchoring](#8-proofregister-anchoring)
9. [ProofStamp Authorization](#9-proofstamp-authorization)
10. [Conformance](#10-conformance)
11. [References](#11-references)
12. [Authors](#12-authors)

---

## 1. Motivation

Vendors publish security claims. Buyers cannot verify them. The structural problem is that every existing evidence format requires either trusting the vendor, trusting a service, or trusting a chain. None of these are independent.

The ProofBundle solves this structurally. It is:

- **Self-contained** — everything needed to verify is inside the bundle
- **Offline-verifiable** — no network call, no service, no chain required
- **Independently witnessed** — assembled by a party outside the vendor trust boundary
- **Tamper-evident** — cryptographic signatures and hash chains detect any alteration
- **Timestamped** — NIST Randomness Beacon pre-execution commitment proves timing

A ProofBundle is the answer to the question every buyer, auditor, and regulator needs to ask: how do I know this is true and that it was true before you knew I was asking?

---

## 2. Terminology

**ProofBundle** — the complete portable evidence artifact defined by this specification.

**Receipt** — a signed record of a single observed action, produced by a mediator outside the agent trust boundary. Receipts are independently verifiable but do not constitute a ProofBundle alone.

**Audit Packet** — the evidence bundle format defined by Pipelock (pipelab.org/schemas/audit-packet-v0.schema.json). The ProofBundle extends this format with Proof Protocol anchoring fields.

**NIST Beacon Pulse** — a cryptographically signed random value published every 60 seconds by the National Institute of Standards and Technology at beacon.nist.gov. Used as the pre-execution commitment anchor.

**ProofRegister** — the canonical public append-only ledger of ProofBundle records operated by HACKERverse at proofregister.com.

**ProofStamp** — the certification mark issued by HACKERverse attesting that a ProofBundle was independently reviewed and the product or agent behind it meets the Proof Protocol certification criteria.

**Root Hash** — the SHA-256 hash of the final receipt in a chain. Serves as the cryptographic fingerprint of the complete evidence chain.

**Witness** — an independent party with no commercial relationship to the vendor under test who observes and attests to the execution conditions of a benchmark run.

---

## 3. ProofBundle Structure

A ProofBundle is a directory containing the following artifacts:

```
proofbundle/
  packet.json          # The bundle envelope (required)
  evidence.jsonl       # The signed receipt chain (required)
  verifier.txt         # Raw verifier output (required)
  nist-pulse.json      # NIST Beacon pulse at pre-execution commitment (required)
  witness.json         # Witness attestation record (required)
  proofregister.json   # ProofRegister anchor record (optional)
  proofstamp.json      # ProofStamp authorization token (optional)
```

The `packet.json` envelope is the authoritative index of the bundle. All other files are referenced by relative path from `packet.json`.

---

## 4. Required Fields

### 4.1 packet.json

The `packet.json` envelope extends the Pipelock Audit Packet v0 schema with the following additional required fields:

```json
{
  "schema_version": "proofprotocol.proofbundle.v0",
  "generated_at": "<RFC 3339 UTC timestamp>",
  "run": {
    "provider": "<github_actions | self_hosted | local>",
    "agent_identity": "<string>",
    "started_at": "<RFC 3339 UTC timestamp>"
  },
  "policy": {
    "policy_hashes": ["<sha256:hex>"]
  },
  "summary": {
    "receipt_count": 0,
    "totals": {
      "allow": 0,
      "block": 0,
      "warn": 0,
      "ask": 0,
      "strip": 0,
      "forward": 0,
      "redirect": 0,
      "other": 0
    }
  },
  "verifier": {
    "verdict": "<valid | invalid | error | not_run | self_consistent_only>",
    "trusted": true,
    "signer_key": "<hex Ed25519 public key>",
    "root_hash": "<sha256:hex>",
    "final_seq": 0
  },
  "posture": {
    "enforcement_mode": "<string>",
    "runner_os": "<string>",
    "raw_socket_status": "<denied | allowed | unknown>",
    "docker_socket_status": "<denied | masked | allowed | absent | unknown>",
    "dns_udp_status": "<denied | proxied | allowed | unknown>",
    "browser_proxy_status": "<forced | advisory | absent | unknown>",
    "websocket_frame_scanning": "<explicit_ws_proxy_path_required | always_on | off>",
    "unsupported_paths": []
  },
  "artifacts": {
    "packet": "packet.json",
    "evidence": "evidence.jsonl",
    "verifier": "verifier.txt"
  },
  "proof_protocol": {
    "spec_version": "PP-SPEC-003-v0.1",
    "nist_pulse": {
      "file": "nist-pulse.json",
      "pulse_uri": "<https://beacon.nist.gov/beacon/2.0/chain/N/pulse/N>",
      "pulse_index": 0,
      "timestamp": "<RFC 3339 UTC timestamp>",
      "output_value": "<hex>"
    },
    "witness": {
      "file": "witness.json",
      "identity": "<string>",
      "affiliation": "<string>",
      "attestation": "<string>"
    },
    "benchmark": {
      "name": "<string>",
      "version": "<string>",
      "corpus_ref": "<URI>",
      "case_count": 0
    },
    "pes": {
      "blocked": 0,
      "missed": 0,
      "observed": 0,
      "irrelevant": 0,
      "score": 0.0
    }
  }
}
```

### 4.2 evidence.jsonl

The byte-for-byte signed receipt chain as produced by the mediator. Must be the exact file consumed by the verifier. Receipts must conform to the Pipelock Action Receipt Format (pipelab.org/learn/action-receipt-spec/) or any Proof Protocol conformant receipt format.

### 4.3 verifier.txt

Raw stdout/stderr output of the verifier binary run against evidence.jsonl. Must include the signer key used for verification and the final verdict.

### 4.4 nist-pulse.json

The NIST Randomness Beacon pulse retrieved immediately before execution began. Must be the complete pulse JSON as returned by beacon.nist.gov. The presence of this pulse in the bundle proves the bundle could not have been constructed before the pulse was published.

### 4.5 witness.json

```json
{
  "witness_identity": "<string>",
  "witness_affiliation": "<string>",
  "commercial_relationship_to_vendor": "none",
  "execution_observed": true,
  "observation_method": "<string>",
  "attestation_timestamp": "<RFC 3339 UTC timestamp>",
  "attestation_statement": "<string>"
}
```

The witness must have no commercial relationship with the vendor under test. Witness identity and affiliation must be disclosed.

---

## 5. Optional Fields

### 5.1 proofregister.json

populated by ProofRegister upon successful submission:

```json
{
  "proof_record_id": "<PR-YYYY-NNNNN>",
  "anchor_timestamp": "<RFC 3339 UTC timestamp>",
  "root_hash": "<sha256:hex>",
  "proofregister_uri": "https://proofregister.com/record/<id>",
  "nist_pulse_at_anchor": {
    "pulse_index": 0,
    "output_value": "<hex>"
  }
}
```

### 5.2 proofstamp.json

Populated by HACKERverse upon ProofStamp authorization:

```json
{
  "proofstamp_id": "<PS-YYYY-NNNNN>",
  "product_name": "<string>",
  "product_version": "<string>",
  "issued_at": "<RFC 3339 UTC timestamp>",
  "expires_at": "<RFC 3339 UTC timestamp>",
  "proof_record_id": "<PR-YYYY-NNNNN>",
  "authorization_signature": "<hex>",
  "proofstamp_uri": "https://proofstamp.io/verify/<id>"
}
```

---

## 6. Verification Procedure

Any party can verify a ProofBundle independently with no external service:

1. Verify `packet.json` conforms to this schema
2. Run the verifier against `evidence.jsonl` using the `signer_key` from `packet.json`
3. Confirm verifier output matches `verifier.txt`
4. Confirm `verifier.root_hash` matches the root hash in `proofregister.json` if present
5. Retrieve the NIST Beacon pulse at `proof_protocol.nist_pulse.pulse_uri` and confirm it matches `nist-pulse.json`
6. Confirm `witness.json` discloses a witness with no commercial relationship to the vendor
7. If `proofstamp.json` is present, verify the authorization signature against the HACKERverse published public key at proofstamp.io

Steps 1-6 require no network call if the bundle is self-contained. Step 7 requires the HACKERverse public key which is published at proofstamp.io.

---

## 7. Relationship to Pipelock Audit Packet v0

The ProofBundle extends the Pipelock Audit Packet v0 schema (pipelab.org/schemas/audit-packet-v0.schema.json). The Audit Packet v0 is the reference implementation of the ProofBundle evidence layer.

The `proof_protocol` block in `packet.json` is the extension point. All base Audit Packet v0 fields are preserved and required. The ProofBundle adds NIST anchoring, witness attestation, benchmark metadata, PES metrics, and optional ProofRegister and ProofStamp fields.

Pipelock receipts conforming to the Action Receipt Format (pipelab.org/learn/action-receipt-spec/) are valid ProofBundle receipt inputs by design. The `verifier.root_hash` field in the Audit Packet v0 is explicitly designed for anchoring to external transparency logs — ProofRegister is that log.

Receipt validity is independent of ProofRegister. A ProofBundle verifies offline. ProofRegister provides the permanent public record. ProofStamp certifies the product behind the bundle.

---

## 8. ProofRegister Anchoring

ProofRegister (proofregister.com) is the canonical public append-only ledger of ProofBundle records operated by HACKERverse.

Submitting a ProofBundle to ProofRegister:
- Creates a permanent dated public record
- Assigns a Proof Record ID (PR-YYYY-NNNNN)
- Anchors the root_hash to a NIST Beacon pulse at submission time
- Makes the record queryable by any third party

ProofRegister anchoring is optional for bundle validity but required for ProofStamp certification.

Enterprise deployments running ProofServer may anchor internal bundles to ProofRegister. Internal bundles do not require ProofStamp authorization to be valid — they become ProofStamp-eligible only when submitted for certification review.

---

## 9. ProofStamp Authorization

ProofStamp is a certification mark owned by Nebulonium, Inc. (HACKERverse). It is not software. It cannot be forked or replicated.

ProofStamp certification requires:
1. A valid ProofBundle submitted to ProofRegister
2. Independent review of the methodology by HACKERverse
3. Confirmation that the witness requirement is satisfied
4. Confirmation that the benchmark meets PP-SPEC-003 conformance requirements
5. Issuance of a ProofStamp authorization token

ProofStamp certifies the product or agent behind the bundle — not the bundle itself. A bundle is cryptographically self-verifying. The stamp means an independent party reviewed it and staked their mark on it.

---

## 10. Conformance

A ProofBundle is conformant with this specification if:

- `packet.json` validates against the PP-SPEC-003 schema
- `evidence.jsonl` verifies against the declared `signer_key` with verdict `valid`
- `nist-pulse.json` contains a valid NIST Beacon pulse predating `run.started_at`
- `witness.json` discloses a witness with no commercial relationship to the vendor
- PES is calculated as `Blocked / (Blocked + Missed) × 100` with OBSERVED and IRRELEVANT cases excluded from the denominator

A ProofBundle that fails any of these requirements is not conformant regardless of what its producer claims.

---

## 11. References

- Pipelock Action Receipt Format: https://pipelab.org/learn/action-receipt-spec/
- Pipelock Audit Packet v0 Schema: https://pipelab.org/schemas/audit-packet-v0.schema.json
- NIST Randomness Beacon: https://beacon.nist.gov
- PP-SPEC-001 Proof Protocol Specification: https://github.com/proofprotocol/Defensible-Knowledge-Proof
- PP-SPEC-002 Proof Validity Specification: https://github.com/proofprotocol/Proof-Validity-Specification
- PP-SPEC-006 Proof of Efficacy Score: https://github.com/proofprotocol/pes-spec
- ProofRegister: https://proofregister.com
- ProofStamp: https://proofstamp.io

---

## 12. Authors

Craig Ellrod, Founder & CEO, Nebulonium, Inc. (d/b/a HACKERverse)  
Castle Rock, Colorado  
2026-07-13

*This specification was informed by the Pipelock Audit Packet v0 architecture developed by Josh Waldrep / LuckyPipe. The ProofBundle extends that architecture with Proof Protocol anchoring, witness attestation, and certification mark integration.*

---

*CC BY 4.0 — Attribution to Craig Ellrod / Nebulonium, Inc. / HACKERverse required.*
