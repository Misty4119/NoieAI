# TOPOLOGICAL_TRAP_SCANNER.md

## Representation-anomaly triage v2.3

A graph or topological feature can be used as an optional signal to select a representation for review. It cannot identify deception, malicious intent, truth, or source authenticity by itself.

For a proposed scanner, record the input graph or complex, construction and filtration, invariant or feature, baseline, parameter sensitivity, validation set, false-positive and false-negative behavior, and comparison against simple non-topological checks. State whether the feature is computed on claims, sources, citations, transformations, or another object; do not conflate those representations.

A flag should lead to inspection of the underlying claim, source path, scope changes, and transformations. Preserve both flagged and unflagged examples. Do not block or downgrade a claim solely because a topological feature changed. No scanner or security control is implemented in this repository.
