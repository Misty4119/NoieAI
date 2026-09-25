# COMPUTATIONAL_ENTROPY.md

## Information measures and model complexity v2.3

Shannon entropy measures uncertainty in a specified probability distribution. For discrete X, H(X)=−Σₓ p(x) log p(x), with log base determining bits or nats. Conditional entropy and mutual information require a joint distribution; finite-sample estimates depend on the data and estimator. None is a generic score of reasoning quality or truth.

Kolmogorov complexity K(x) is the length of a shortest description under a universal description language, up to machine-dependent additive constants. Exact K is not computable in general. Compression length, restricted-model code length, or minimum-description-length scores are computable proxies whose results depend on the encoding, model class, and coding assumptions.

A reproducible comparison identifies the object or data, probability model or code, sample and selection process, estimator, baseline, uncertainty, and intended decision use. A shorter description may regularize a model or serve as a prior; simplicity alone does not establish that a claim is true. A path’s entropy, token count, hash, or compression ratio does not prove effort, provenance, independence, or deception.

No algorithmic-complexity or inference-path monitor is included in this repository.
