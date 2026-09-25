# ZKP_VERIFICATION.md

## Zero-knowledge proof scope

A zero-knowledge proof protocol can let a verifier check a stated formal relation while learning no more about a witness than the protocol permits, subject to its construction and cryptographic assumptions. The exact predicate, public inputs, witness relation, soundness notion, and security model must be specified.

Verification establishes only that the proof is accepted for that relation and those inputs under the verifier's implementation and protocol assumptions. It does not establish that:

- public inputs are authentic or accurately describe the world;
- a witness corresponds to a real-world event;
- the predicate captures the intended natural-language claim;
- the source data were honestly collected;
- a claim is empirically true beyond the formal relation.

## Verification record

Record protocol and version, circuit or relation identifier, public inputs, verifier implementation/version, verification result, trusted setup assumptions if any, and source provenance for the inputs. Do not publish a private witness or secret key. Do not label a claim `VERIFIED_TRUE` solely because its proof verifies.

Use `PROOF_ACCEPTED_FOR_STATED_RELATION` or `PROOF_REJECTED`; use `NOT_CHECKED`, `UNSUPPORTED`, or `ERROR` when verification is unavailable or incomplete. Keep protocol agreement and empirical evidence separate.

## Capability boundary

This Markdown file provides no cryptographic implementation, key custody, setup, trusted-input attestation, or proof verifier.