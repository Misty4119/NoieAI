# DIVERGENCE_DETECTOR.md

## L2 — Evidence and representation discrepancy review

“Divergence” is a review label for a detected discrepancy among a claim, its cited support, its stated scope, or its transformed representation. It is not a replacement for the ordinary distinctions among falsehood, error, uncertainty, missing evidence, and intentional deception.

## Review categories

| Category | Review question |
| --- | --- |
| Unsupported detail | Does the claim contain details not supported by the cited evidence? |
| Provenance mismatch | Can the cited source or transformation be located and checked? |
| Scope mismatch | Does the conclusion exceed the population, time, domain, or assumptions of its support? |
| Contradiction | Does an explicit, scope-aligned claim conflict with another claim? |
| Staleness | Is the source too old for a time-sensitive conclusion? |
| Representation change | Did a translation, summary, or transformation alter a relevant distinction? |
| Unresolved ambiguity | Are multiple interpretations still plausible? |

These categories describe observed evidence and review state. Intent to deceive requires separate evidence about an actor's knowledge and intent; it cannot be inferred from a false statement or a topological statistic.

## Review procedure

1. Extract a precise claim and its scope.
2. Resolve each cited source and inspect the relevant passage, dataset, or derivation.
3. Compare support to the claim, including counterevidence and date.
4. Record the discrepancy, uncertainty, and limits of the review.
5. Return `SUPPORTED_WITHIN_SCOPE`, `CONTRADICTED`, `UNRESOLVED`, `UNSUPPORTED`, or `NOT_REVIEWED`.

Do not assign a score or downgrade an epistemic class using an unexplained threshold. If a statistical detector is used, state its data, metric, decision threshold, validation, and error rates. Absence of a detected discrepancy is not evidence that a claim is true.

## Representation diagnostics

Embedding distance, topology, or semantic-similarity signals may help select material for human or tool review when the representation, metric, and validation are specified. Such signals do not establish truth, intent, or meaning on their own. A Betti-number change is a property of a chosen mathematical representation, not a definition of lying.

## Capability boundary

This document specifies a review process. No executable detector, provenance validator, adversarial defense, or automatic interception is present in this Markdown repository.