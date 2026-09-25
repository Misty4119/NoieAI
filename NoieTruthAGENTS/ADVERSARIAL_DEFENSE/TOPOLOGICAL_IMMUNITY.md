# TOPOLOGICAL_IMMUNITY.md

## Limits of the defense metaphor v2.3

“Topological immunity” is a design metaphor, not a security guarantee. Persistent homology and related summaries can characterize a specified graph, point cloud, or filtered complex; robustness of that summary does not imply robustness of a reasoning system or resistance to adversarial input.

If a topology-based diagnostic is used, specify representation, construction, metric, filtration, coefficient field, thresholds, baseline, validated attack classes, and error rates. Test harmless paraphrases, source reordering, omitted evidence, adversarially added edges, and changes in scale or sampling as appropriate to the representation. Include cases where true evidence legitimately changes the structure.

Route any flag to claim-level and provenance review. Do not automatically quarantine, delete, or reduce confidence on a topological signal. No topology-based defense or automatic containment is present in this repository.
