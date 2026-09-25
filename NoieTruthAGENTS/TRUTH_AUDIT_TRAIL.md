# TRUTH_AUDIT_TRAIL.md

## Truth-OS audit extension

Use the shared event envelope in root `AGENTS.md`. The fields here add provenance and epistemic review details. This document is a schema; it does not provide a live logger, truth oracle, calibration database, distributed consensus, or immutable storage.

### Truth fields

| Field | Meaning |
| --- | --- |
| `claim_reference` | Claim identifier, text or artifact reference, and scope |
| `claim_type` | Empirical, mathematical, interpretive, normative, or other declared type |
| `support_references` | Sources, data, or derivations and their relevant scope |
| `review_status` | `SUPPORTED_WITHIN_SCOPE`, `CONTRADICTED`, `UNRESOLVED`, `UNSUPPORTED`, `NOT_REVIEWED`, or `ERROR` |
| `uncertainty` | Qualitative limits or a probability with a stated model and calibration basis |
| `provenance_status` | Source-resolution, integrity-check, or proof status, each scoped to what was checked |

Keep evidence, proof status, confidence, freshness, and claim type distinct. A zero-knowledge proof establishes only its stated formal relation under protocol assumptions; consensus establishes protocol agreement. Neither alone establishes an empirical claim. A failed or incomplete check must retain its scope and limitations.

Record structured rationale, source references, relevant tool versions/results, and corrections. Do not record hidden chain-of-thought or unnecessary sensitive content. Retention, access control, signatures, anchoring, and append-only storage are implementation properties, not properties of this Markdown file. A hash chain is tamper-evident only under its stated storage and key assumptions.

Preserve genuine historical records and add linked corrections instead of silently rewriting them. Example payloads in this repository are specifications, not evidence that events occurred.
## Truth-domain event coverage

Where a host supports event capture, record material transitions such as claim extracted or normalized; source located, retrieved, or unavailable; source version checked; evidence compared; derivation or proof checked; calibration forecast made or resolved; contradiction detected; freshness reviewed; answer marked unknown; claim corrected or superseded; and an EpistemicReport delivered to Logic-OS.

Link event references to the exact claim and scope, source passage or data field, transformation, verification artifact, and result status. Preserve the difference between gathering a citation, reading it, checking whether it supports the claim, and checking whether the source itself is authentic or current. A claimed check with no tool result or human review record must not be recorded as completed.

## Calibration and correction records

For a forecast, preserve its event definition, probability, horizon, reference class, forecast time, and resolution rule. Later resolution should be a linked event with the outcome source and any ambiguity. Do not overwrite an old forecast with its resolved outcome or score.

For a corrected answer, identify the prior claim, correction reason, new or reinterpreted evidence, scope affected, and any downstream decisions requiring review. Distinguish a retraction, a narrower scope, an updated fact, and a correction of an inference. Intent to deceive cannot be inferred from the fact of correction.

## Review limits

Truth events can show what evidence was reviewed and what checks were performed; they do not make an unsupported claim true. A hash, signature, proof, consensus result, or complete-looking chain only supports its specified property under declared assumptions. Keep unknown, inaccessible, contested, incomplete, and not-checked states visible. Respect data minimization, access, retention, and correction policies supplied by the host.
