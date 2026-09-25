# SOURCE_REGISTRY.md

## Source reference contract

This file defines metadata for evidence references. It is not a live registry, source-ranking algorithm, identity verifier, or truth score.

A source record should include a stable identifier, source type, creator or issuing body, title, publication/update date, location, retrieval date, version or digest when useful, access status, relevant excerpt or locator, and known limitations. For data, also record collection method, population, units, licensing, and transformations.

Assess relevance and reliability for the specific claim and context; source-type labels do not give universal numeric reliability scores. A signature or digest can support integrity/authorship checks under key and storage assumptions, but does not establish that the source's content is true. A ZK proof is recorded as a formal verification result for its stated relation, not as evidence of real-world truth unless input authenticity is separately established.

If a source cannot be resolved, mark it inaccessible or unverified. Do not fabricate a citation, assume freshness, or assign fixed confidence multipliers. No source registry service is included in this Markdown repository.
## Source review dimensions

Assess a source in relation to a particular claim rather than assigning it a universal rank:

| Dimension | Review question |
| --- | --- |
| Authority | Is this source responsible for the record, measurement, filing, or rule in question? |
| Directness | Does it directly establish the claim or only report another source? |
| Method | Are collection, analysis, and uncertainty described well enough for the claim? |
| Scope | Do population, jurisdiction, definitions, date, and conditions match? |
| Independence | Does it share upstream evidence or processing with other cited sources? |
| Integrity | Is this the same artifact or dataset that was reviewed? |
| Currency | Is it still current for the decision, or explicitly historical? |
| Access | Can the reviewer retrieve it, and are licensing or privacy constraints respected? |

These dimensions can disagree; preserve them separately. A primary source may be authoritative about what an agency issued while still being incomplete evidence for whether a policy worked. An expert secondary source may explain a field well but not replace a source record when an exact current value is required.

## Registration and correction

At ingestion, retain title, issuing body, source type, publication or effective date, retrieval date, stable locator, version or digest where useful, and exact passage or data field used. For datasets add collection population, unit definitions, missingness, transformations, and licensing. If a source is revised, keep the version originally used and link to the newer version with its effect on dependent claims.

A source correction, retraction, access failure, or identity mismatch should create a review event for dependent claims. It does not automatically refute all claims connected to that source; re-evaluate the actual dependency.
