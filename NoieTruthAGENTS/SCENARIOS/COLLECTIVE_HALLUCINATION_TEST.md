# COLLECTIVE_HALLUCINATION_TEST.md

## Shared-source dependence scenario

Design status: `DESIGN_ONLY`; no agents, dataset, detector, or test run is included.

**Question:** Could several reports that appear independent actually derive from one shared source, copied claim, model, or incentive?

Record the claim, each report and provenance path, common sources or transformations, identity assumptions, protocol rules, and an independent primary source if available. Compare provenance rather than agent count. Shared provenance weakens independence; it does not by itself make the claim false.

Expected review outcome: mark independence `UNASSESSED`, `DEPENDENT`, or `INDEPENDENCE_SUPPORTED_WITHIN_SCOPE`, explain evidence and uncertainty, and seek independent evidence. Do not map a vote count to EC-L confidence or label a group “hallucinating” without reviewing the claim and sources.
## Adversarial test cases

Consider reports that share a search result, are prompted with the same false premise, cite a syndicated article, use the same model family or retrieval index, or inherit a stale record. Contrast these with genuinely separate measurements or source paths where available. Vary participant count independently from evidence independence so the review cannot use vote count as a proxy.

Record the dependency graph, how each relation was established, missing links, actual independent verification, and whether the claim's scope changed across reports. The expected outcome may be DEPENDENT, INDEPENDENCE_UNASSESSED, or INDEPENDENCE_SUPPORTED_WITHIN_SCOPE. None of these statuses determines truth or proves that an error was intentional.
