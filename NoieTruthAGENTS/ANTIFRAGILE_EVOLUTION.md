# ANTIFRAGILE_EVOLUTION.md

## L2 — Controlled revision process

“Antifragile” is an aspiration for learning from review, not a guarantee that changes improve a system. Revision means a human or authorized maintainer edits a specification or implementation; this repository has no self-modifying runtime.

## Change record

For a material change, record its date, affected files, motivation, evidence or rationale, compatibility impact, review status, and unresolved questions in the relevant evolution log. Preserve historical entries as historical records. Correct errors by adding a dated correction with a link to the affected entry; do not imply a hash chain or append-only mechanism exists unless the storage implementation enforces and attests it.

## Review gates

1. Identify the exact claim, rule, interface, or behavior proposed to change.
2. Check consistency with the Meta-Kernel and the other pillar interfaces.
3. State tests, evidence, assumptions, and known gaps.
4. Have an authorized maintainer review changes that affect policy, interfaces, or safety behavior.
5. Record the accepted version and the files it governs.

Do not claim geometry, topology, or a numerical phase-transition threshold proves a change is safe or beneficial. Formal verification applies only to a formalized property under its stated model; empirical evaluation applies only to tested conditions.

## Non-self-modification

No automated or agent process receives authority to modify policy, access controls, or immutable infrastructure through this document. A host must enforce authorization and preserve an auditable change history. System shutdown or refusal remains a valid outcome when a requested change cannot be safely assessed.

## Version history

- v2.2 — 2026-03: initial design text.
- v2.3 — 2026-09: clarified that evolution is maintainer-controlled, scoped verification claims, and separated revision history from runtime guarantees.