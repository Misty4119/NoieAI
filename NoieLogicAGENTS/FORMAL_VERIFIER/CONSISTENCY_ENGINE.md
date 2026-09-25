# CONSISTENCY_ENGINE.md

## Consistency checking v2.3

Consistency is relative to a declared set of statements, context, time, and logic. This module reports whether a supported checker found a contradiction; it does not decide which conflicting source is true.

### Input contract

Provide the statement set, source/provenance references, scope, time, assumptions, and selected logic. State whether statements are classical, temporal, probabilistic, paraconsistent, or another formalization where relevant.

### Result contract

Return CONSISTENT, CONTRADICTION_FOUND, INDETERMINATE, UNSUPPORTED, or ERROR. Include the conflicting statements and their scope, the rule or checker involved, and limitations. A bounded search that finds no contradiction is not a proof of global consistency unless completeness is established for the declared finite domain.

### Handling conflicts

Separate direct logical contradiction from different time points, definitions, scopes, sources, or model assumptions. Preserve contested evidence for Truth-OS. Do not silently discard a claim by assigning an epistemic level or vote count. Logic-OS owns downstream policy choices.

### Capability

A symbolic checker is active only when its implementation, supported logic, and result are attested. A specification or pseudocode does not constitute a running checker.