# FORMAL_VERIFIER.md

## Formal verification contract v2.3

**Purpose:** State and check properties of explicit formal models, specifications, proofs, or implementation artifacts. This module does not validate empirical truth, policy approval, or runtime capability unless a separately attested integration is provided.

## 1. Report fields

~~~yaml
verification_id: identifier
target: specification | model | proof | implementation | runtime
property: precise property being checked
formal_system: logic, type system, or proof calculus
assumptions: explicit premises and environment model
artifact_ref: proof, model, checker output, or source reference
checker: name and version if available
status: PROVED | REFUTED | NOT_CHECKED | UNSUPPORTED | ERROR
scope: exact artifact and conditions covered
limitations: omitted cases and trusted components
~~~

An empirical confidence estimate is a separate claim with its own dataset, calibration, scope, and provenance. Do not derive it from a proof status. A proof for a model does not show that the model matches the world or that deployed code matches the proved artifact.

## 2. Verification procedure

1. Define the property, artifact, logic, assumptions, and boundary.
2. Establish whether the property is decidable and supported by the chosen checker.
3. Check the artifact and preserve the exact checker result and version.
4. Report PROVED only when the proof obligation is discharged for the declared scope.
5. Report REFUTED only when a valid counterexample or contradiction is produced.
6. Otherwise report NOT_CHECKED, UNSUPPORTED, or ERROR with the reason.
7. Keep proof references and structured decision factors in the audit record.

“Not disproved” is not PROVED. A timeout or missing backend is not a pass. Heuristic linting and model checking over a bounded state space must identify that bound.

## 3. Formal assurance dimensions

There is no universal scalar FV level that represents correctness. If a project uses local assurance levels, define them as coverage or proof-work categories only and retain the fields above. Never map them to percentages or empirical confidence.

Distinguish at least:

- proof validity inside a formal system;
- completeness of the stated assumptions and specification;
- checker or trusted-kernel assurance;
- implementation conformance to the checked artifact;
- empirical behavior on a test distribution;
- operational capability of the deployed runtime.

## 4. Limits and failure

Formal proof depends on sound rules, definitions, and trusted components. Undecidable properties, resource limits, incomplete models, unsupported logics, and missing checkers are explicit failure states. A formally verified decision may still rely on false premises, stale evidence, incorrect physical models, or unauthorized policy.

High-impact use requires the host to define the required proof obligation and assurance process. The presence of this Markdown file does not enable a verifier.

## 5. Module map

- FORMAL_VERIFIER/PROOF_CHECKER.md: proof artifact and checker result.
- FORMAL_VERIFIER/CONSISTENCY_ENGINE.md: consistency relative to declared logic and context.
- FORMAL_VERIFIER/CLOSURE_DETECTOR.md: bounded consequence or closure analysis.
- AUDIT_TRAIL.md: shared event fields and Logic-domain extensions.

Version labels in older examples and histories are preserved as records; the active specification baseline is v2.3.
## 6. Verification methods and their scopes

Choose a method according to the property, artifact, and state space:

| Method | Typical target | Required limitation statement |
| --- | --- | --- |
| Proof checking | A finite derivation in a declared calculus | Logic, axioms, imported definitions, checker, and trusted kernel |
| Theorem proving | Search for a derivation of a stated proposition | Search may be incomplete or resource bounded; failure to find a proof is not refutation |
| Model checking | Properties of a finite or bounded transition system | State-space bounds, abstraction, fairness assumptions, and explored states |
| Static analysis | Syntactic or abstract properties of source artifacts | Abstraction and false-positive/false-negative limitations |
| Runtime monitoring | Events in an executing system | Instrumentation coverage, sampling gaps, monitor failures, and deployment identity |

A result from one method must not silently inherit the assurance of another. A bounded model checker can establish a property of the explored model, not all possible deployments. A runtime monitor can report observed events, not prove that unobserved events did not occur.

## 7. Formalization and proof obligations

Translate the requested property into explicit predicates and quantifiers. Resolve types, time horizon, state boundary, preconditions, invariants, and permitted environment behavior before checking. Record assumptions as premises, not as consequences of the proof.

Separate well-formedness, consistency, satisfiability, validity, reachability, refinement, and implementation conformance. A satisfiable specification has at least one model; it may still permit unsafe behaviors. A consistency check does not prove completeness, usefulness, truth of premises, or conformance to deployed code.

Gödel incompleteness results apply to sufficiently expressive formal systems meeting their mathematical conditions. They do not imply that every practical property is undecidable or that formal checking is futile. State the actual logic and decision procedure limits for the property at hand.
