# CLOSURE_DETECTOR.md

## Bounded consequence analysis v2.3

“Closure” means consequences generated from a declared rule set and premises within a stated scope. It does not mean every consequence is computable or that the premises are true.

### Required declaration

Record the rules, premises, inference semantics, domain, termination condition, resource bound, and whether the procedure is complete for that domain.

### Result states

Return FIXPOINT_REACHED, BOUND_REACHED, CONTRADICTION_FOUND, UNSUPPORTED, or ERROR. FIXPOINT_REACHED means no additional consequence was found by the stated procedure; it implies completeness only if the procedure and finite domain justify that claim. A bounded run must expose its bound.

### Limits

General logical consequence can be undecidable. Heuristic search, sampling, depth limits, and incomplete rule sets must be labeled. Do not map closure depth to confidence or formal proof. Preserve the premises and generated consequences as structured artifacts.