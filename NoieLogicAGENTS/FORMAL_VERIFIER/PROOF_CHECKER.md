# PROOF_CHECKER.md

## Proof artifact checking v2.3

This module specifies a proof-checking boundary. A proof checker validates derivation steps against a declared formal system; it does not establish empirical facts or the correctness of premises.

### Inputs

- proof artifact and target proposition;
- formal system, definitions, axioms, and imported lemmas;
- checker identity/version and trusted computing base;
- exact assumptions and scope.

### Results

Return PROVED, REFUTED, NOT_CHECKED, UNSUPPORTED, or ERROR, with artifact references, failed obligations, counterexample where valid, resource limits, and checker details. A timeout, absent checker, incomplete proof, or unsupported rule cannot be treated as success.

### Assurance boundaries

A checked derivation supports the target only if its assumptions and formalization apply. Report proof validity separately from empirical confidence, test evidence, implementation conformance, and policy permission. Do not attach a probability to a proof level.

### Capability and audit

Only report a checker as active when the runtime identifies the implementation, version, supported logic, and result. Record a structured summary and proof reference in the shared audit event. Do not require raw private reasoning.
## Proof object review procedure

A proof-checking integration should verify a proof object against a small, declared rule kernel where practical. Proof search and proof checking are separate capabilities: search may use heuristics and can fail to find an existing derivation, while checking determines whether the supplied artifact follows the declared rules.

For each artifact:

1. Validate syntax, types, and the proposition being proved.
2. Resolve every imported axiom, definition, theorem, and versioned dependency.
3. Check each inference step with the declared kernel or trusted checker.
4. Bind the result to a digest or stable artifact reference and record the checker version.
5. Report trusted components, assumptions, scope, and any unverified external step.

A proof may be syntactically valid yet prove the wrong proposition, rely on a false or unauthorized premise, or refer to an artifact different from the deployed one. Counterexamples are valid only relative to the checked model and property. Never interpret missing proof, timeout, or unsupported syntax as a counterexample or success.
