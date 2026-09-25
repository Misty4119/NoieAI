# AKASHIC_PROTOCOL.md

## L2 — Provenance ledger design note

“Akashic record” is a project metaphor for a provenance ledger. This Markdown specification does not provide a distributed service, immutable storage, trusted timestamps, consensus, or a truth oracle. Any implementation must state its storage, identity, key-management, replication, and threat-model assumptions.

## Record contract

A record may contain:

| Field | Meaning |
| --- | --- |
| `record_id` | Identifier assigned by the implementation |
| `claim` | Precisely scoped proposition or artifact reference |
| `claim_type` | Empirical, mathematical, interpretive, normative, or other declared kind |
| `support` | Evidence references or a reproducible derivation, with scope and limitations |
| `observer_context` | Relevant time, conditions, measurement method, and version |
| `review_status` | `SUPPORTED_WITHIN_SCOPE`, `CONTRADICTED`, `UNRESOLVED`, or `NOT_REVIEWED` |
| `integrity_metadata` | Optional digest, signature, or inclusion proof and its verification assumptions |

Corrections should link to the superseded record and explain the change. A ledger's retention and edit policy is an implementation choice; this document cannot enforce append-only behavior.

## Proof and agreement limits

A zero-knowledge proof can establish that a prover knows a witness satisfying a stated formal relation, subject to the protocol and cryptographic assumptions. It does not independently establish that an empirical input is authentic, that a premise describes the world, or that a claim is true. See `CONSENSUS_MODULES/ZKP_VERIFICATION.md`.

Consensus means that participants following a specified protocol reached an agreed state under that protocol's assumptions. It is not a truth predicate. Record participant independence, quorum, adversary, and input assumptions where relevant.

Hashes and signatures can support integrity or authorship checks only under stated key, storage, and anchoring assumptions. A hash chain alone cannot prevent an operator from rewriting an unanchored history.

## Capability boundary

All schemas and procedures here are design guidance. No running ledger, cryptographic verifier, independent reviewer network, or adversarial defense is included in this Markdown repository. Mark such a capability `UNATTESTED` until an implementation and its checks are identified.
