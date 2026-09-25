# ALGORITHMIC_ENTROPY.md

## Description-length and model-complexity heuristic v2.3

This module replaces “algorithmic entropy proof” as a truth or effort guarantee. Description length can compare candidate representations under a declared encoding or model class. It cannot certify that a claim is true.

Kolmogorov complexity is defined relative to a universal description language but is not computable in general. Practical systems therefore use computable proxies, restricted models, compression schemes, or regularized scores. State the chosen proxy and its limits.

For a comparison, record:

- the object or data being encoded;
- code, model class, and assumptions;
- measured or estimated description length;
- comparison baseline and purpose;
- reproducibility and known bias.

Use this only as a hypothesis prior, regularizer, model-selection aid, or understanding proxy. Simpler explanations can be false, and complex explanations can be correct. A hash of an inference trace, operation count, or energy reading is not a proof of effort, provenance, or truth.