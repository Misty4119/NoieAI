# PROVENANCE_CHAIN.md

## Claim provenance v2.3

**Purpose:** Preserve the path from a claim to its evidence, premises, transformations, and checks. Provenance supports review; it is not itself proof that a claim is true.

## 1. Provenance record

For each material claim, record the applicable fields:

- claim identifier and exact scope;
- source or premise identifier, creator/provider, date, and retrieval context when available;
- relation to the claim: observation, quotation, derivation, inference, transformation, or rebuttal;
- transformations, tools, and versions used;
- verification method and result;
- uncertainty, freshness, limitations, and missing links;
- integrity reference such as a digest when available.

A source may be a document, observation, dataset, formal derivation, user-provided premise, or a disclosed absence of evidence. Do not fabricate a source to fill an empty field.

## 2. Provenance graph

Represent derivations as links between claims, evidence entities, and activities. Preserve direction and scope. A graph is an organizational model; it does not establish source independence or truth without checking how the sources were produced.

When sources repeat one another, record the common origin so repetition is not mistaken for independent corroboration.

## 3. Integrity and authenticity

A cryptographic hash can identify a particular byte sequence and reveal changes relative to a trusted digest. It does not prove who created the content, when it was created, whether the source is truthful, or whether the audit record cannot be deleted.

Signatures require verified keys and an identity policy. Append-only behavior requires storage and access controls outside this document. Hash chains are tamper-evident only under their storage, key, and anchoring assumptions.

## 4. Proofs and computation records

A formal proof reference identifies the artifact, formal system, premises, checker, and result. It supports only the property checked within that scope.

A computation record may note a tool, version, inputs, output, and resource measurement when the runtime attests them. Work performed, token count, energy, path length, or a hash of a private trace is not evidence that the result is correct. Do not require hidden chain-of-thought.

Use PROVENANCE_CHAIN/ALGORITHMIC_ENTROPY.md only as a description-length or model-complexity heuristic, not as a proof of knowledge or truth.

## 5. Zero-knowledge property verification

A zero-knowledge proof may let a verifier check a specified predicate or knowledge relation while limiting disclosure of the witness. Record the statement, protocol, public inputs, assumptions, verifier result, and binding to external inputs.

A valid proof does not independently establish that a sensor reading or external record corresponds to reality. That requires separately stated provenance, source trust, and attestation assumptions. See CONSENSUS_MODULES/ZKP_VERIFICATION.md.

## 6. Result and failures

Return provenance as complete, partial, unavailable, or contested, with the missing links. A missing provenance link lowers support and must be disclosed; it does not automatically prove the claim false.

Use the common audit event in AGENTS.md and the Truth-domain extension in TRUTH_AUDIT_TRAIL.md.
## Provenance edge types and review

Use explicit relations such as observed-by, quoted-from, derived-from, supports, qualifies, contradicts, retracts, supersedes, and verified-by. Each edge has a scope and, where useful, the activity or transformation that produced it. A source-to-claim link should identify whether the source directly states the claim or only supports an inference.

For evidence assembled from several sources, record common upstream data, shared authorship, common instruments, syndicated text, or copied estimates when known. Several URLs or agents repeating one origin are not independent corroboration. Preserve retrieval date and source version for changing content. If an input was unavailable, record the gap rather than creating a placeholder source.

A review should verify, as relevant: reference resolves; content identity matches the cited version; the relevant passage or data field supports the claim; date and scope match; transformations are reproducible; and any derived result can be checked. Integrity, authenticity, source authority, relevance, and truth are separate properties.

## Provenance result checklist

- COMPLETE_WITHIN_SCOPE: links needed for the stated claim and inference are present and reviewed.
- PARTIAL: some material links are present but one or more links are unresolved.
- UNAVAILABLE: the source or verification method could not be accessed.
- CONTESTED: source lineage or content identity is disputed.
- NOT_REVIEWED: only references were collected; no support check was performed.

Never use “complete provenance” to imply that a claim is true. Record access restrictions and redact sensitive content while preserving a permitted reference where possible.
