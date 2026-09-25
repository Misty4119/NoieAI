# CONSISTENCY_ENGINE.md

## L2 — Consistency checking contract

This document defines checks a host may perform against explicitly represented claims. It does not implement a theorem prover, natural-language contradiction detector, quantum logic engine, or universal consistency guarantee.

## Scope the check

Every check must identify:

- the input claims and their provenance;
- the language, context, time, and domain assumptions;
- the logic and inference rules used;
- whether the result is complete for that input class;
- unsupported syntax, resource limits, and errors.

A contradictory pair is a claim and its negation under the same interpretation, scope, time, and assumptions. Different times, definitions, modalities, or contexts may remove an apparent contradiction; do not silently merge them. An implication conflict requires an actual derivation under the declared rules, not semantic similarity alone.

## Result vocabulary

| Result | Meaning |
| --- | --- |
| `CONSISTENT_WITHIN_SCOPE` | No contradiction was found by the stated, completed check |
| `CONTRADICTION_FOUND` | A contradiction was derived under the stated interpretation |
| `INCOMPLETE` | The check stopped early or covers only part of the input |
| `UNSUPPORTED` | The input uses a construct the checker does not handle |
| `ERROR` | The check could not be completed reliably |

A successful bounded check is not proof of global consistency. Preserve unresolved conflicts and cite the specific claims involved; do not resolve them by voting, confidence averaging, or deleting the less convenient claim.

## Logic choices

Classical, three-valued, modal, probabilistic, and quantum-logical formalisms have different semantics and applicability. Select a logic for a defined problem and state its semantics. A quantum logic model applies to propositions represented in its formal domain; it is not a general substitute for ordinary factual consistency checking.

Probabilistic coherence, logical consistency, formal proof status, and empirical support are distinct properties. No universal evidence-confidence level (`EC-L`) follows from a consistency result. See `EPISTEMOLOGY_AXIOMS.md` for claim-status reporting and `../NoieLogicAGENTS/FORMAL_VERIFIER.md` only when a formal proposition is actually checked.

## Capability boundary

The files in this repository are Markdown specifications and examples. They do not include an executable parser, solver, proof kernel, or guarantee that arbitrary natural-language claims have been checked.
## Contradiction review procedure

1. Normalize the propositions while preserving scope, modality, quantifier, source, and time.
2. Check that both claims use compatible terms, units, populations, and conditions.
3. Identify the selected logic and inference rules; do not blend logics without a translation rule.
4. Search for a direct contradiction or a derivation of incompatible consequences.
5. Record whether the input representation is complete for the check.
6. Return the paired claim references, exact conflicting consequence, derivation or detector artifact, and unresolved assumptions.
7. Keep both records and link the review; correction or supersession requires evidence and an authorized record update.

Common apparent conflicts include temporal change (“open” at one time and “closed” later), different definitions or measurement units, universal versus existential claims, incompatible populations, conditional claims with different premises, and genuine direct negation. Model a temporal change as two scoped claims rather than deleting one.

## Partial and paraconsistent handling

A contradiction in a knowledge base need not imply that every unrelated claim is usable or false. A host may choose a paraconsistent logic or isolate inconsistent contexts to prevent explosion, but must identify the formal semantics and consequence relation. Classical explosion applies only within classical entailment from a contradiction; it is not a description of every reasoning system.

For incomplete information, three-valued logics may represent TRUE, FALSE, and UNKNOWN. UNKNOWN is not a third truth value that means “partly true”; its interpretation depends on the selected logic. If a truth table or solver is used, identify the logic variant and input encoding.

## Circular-support review

A support cycle occurs when claims ultimately rely on one another without an independent premise or observation. Represent derivation edges and inspect strongly connected components; a cycle is a review signal, not automatically proof that its conclusion is false. Record any external grounding, shared source, or independent measurement that breaks the apparent cycle. Do not call repeated text or multiple copies of one source independent evidence.

## Result examples

- CONSISTENT_WITHIN_SCOPE: a declared complete check found no contradiction among the represented claims; it does not show that the claims are true.
- CONTRADICTION_FOUND: the stated logic derives incompatible propositions under shared scope; retain the proof or conflict pair.
- INCOMPLETE: the check omitted claims or stopped at a resource bound.
- UNSUPPORTED: required language, modality, or logic is outside checker capability.
- ERROR: parsing, resolution, or checker integrity failed; do not report a partial pass as success.
