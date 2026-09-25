# BETTI_NUMBER_CHECKER.md

## Homology ranks for a declared representation v2.3

For a finite chain complex over a specified coefficient field, the k-th Betti number is the rank of the k-th homology group. With boundary maps ∂ₖ, βₖ=dim ker(∂ₖ)−rank(im(∂ₖ₊₁)). The value depends on how the underlying space or complex was constructed, its coefficient field, degree, and any filtering or reduction choices.

For persistent homology, state the filtration, metric or threshold parameter, coefficient field, persistence algorithm, and interval-selection rule. Report sensitivity to construction and parameters, numerical or combinatorial limits, and the exact source and target representation being compared.

Equal Betti numbers are necessary but not sufficient for homotopy equivalence. A change can flag a structural difference in the chosen representation; it does not establish semantic loss, a lie, or a false claim. A valid epistemic interpretation requires a justified mapping from claims and relations to the complex and validation against the intended task. No topology library or validated claim-to-complex mapping is included here.
