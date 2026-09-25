# EPISTEMOLOGY_AXIOMS.md

## Claim status and evidence model v2.3

**Role:** Define the epistemic representation used by Truth-OS. These are operational design conventions, not universal laws of knowledge.

## 1. Claim record

Decompose material statements into claims with a scope, context, and time. Keep these dimensions separate:

~~~yaml
claim_id: identifier
claim: proposition or question
claim_type: formal | empirical | historical | source_report | inference | conjecture | normative
logical_status: valid | invalid | undecided | not_applicable | unknown
evidence_strength: assessed separately with basis
source_quality: assessed separately with provenance
calibrated_confidence: optional probability for a specified event and reference class
freshness: current | stale | unknown | not_time_sensitive
contest_status: untested | supported | contested | refuted | unknown
assumptions: explicit
scope: domain, population, time, and conditions
verification_method: method and artifact reference, if any
limitations: known gaps
next_information_step: optional
~~~

Do not force unlike properties into one ordinal or 0–1 score. Use NOT_APPLICABLE or UNKNOWN rather than inventing a numeric value.

## 2. Claim status vocabulary

| Status | Meaning |
| --- | --- |
| ESTABLISHED | Broadly supported within a stated domain and conditions |
| FORMAL_RESULT | Follows from stated definitions, rules, and assumptions |
| EFFECTIVE_MODEL | Useful representation within a specified regime or scale |
| INTERPRETATION | A proposed account of a theory, not its unique empirical consequence |
| CONJECTURE | Proposed claim that remains unresolved |
| DESIGN_AXIOM | NoieAI rule chosen for system design, not a claim about nature |
| ENGINEERING_HEURISTIC | Useful approximation whose limits must be stated |
| METAPHOR | Intuitive language, not a mathematical identity |

Status does not mean confidence. A source citation does not by itself make a claim ESTABLISHED; a proof does not establish premises outside its formal system.

## 3. Confidence and calibration

Use calibrated confidence only for a probabilistic claim with a defined event, reference class, time horizon, and relevant evaluation data. Record the calibration method, sample, distribution, and limits. Calibration is empirical and can fail under distribution shift.

A formal proof result is reported as proof status and scope, not as 100% confidence. Evidence strength, source quality, freshness, logical validity, and calibrated probability remain separate.

Uncertainty decompositions depend on the model and question. If a distinction such as model uncertainty versus outcome variability is used, define it for that model rather than presenting one taxonomy as universally settled.

## 4. Evidence and provenance

Evidence may be a measurement, source document, testimony, dataset, formal derivation, or an explicitly stated premise. Record what supports a claim and how it was obtained. A claim without adequate support is unsupported or unknown; absence of a citation does not alone show falsity.

When sources disagree, retain both claims with source, scope, time, and reasons for disagreement. Do not resolve a factual conflict by vote count, rhetorical confidence, or recency alone.

## 5. Unknowns and correction

Truth-OS may return unknown, not checked, not identifiable, unavailable, or contested. State the specific gap and the next evidence-gathering step where useful.

Distinguish error, unsupported assertion, false statement, and intentional deception. A false statement does not establish intent to deceive. Intent categories are operational policy judgments requiring context, not a mathematical invariant.

## 6. Limits of formal and mathematical tools

- Formal verification establishes a property of a formalized artifact under its assumptions.
- ZK proofs establish a specified cryptographic predicate under protocol assumptions, not the external truth of arbitrary inputs.
- Consensus establishes agreement under protocol identity, fault, validity, and network assumptions, not truth of inputs.
- Compression and description length can compare models within a declared coding scheme; simplicity alone does not imply truth. Exact Kolmogorov complexity is not generally computable.
- Betti numbers, homotopy, and other topological quantities describe a chosen mathematical representation. Changes may flag representation differences but do not define lying or semantic loss by themselves.
- Category-theoretic structures are FORMAL_RESULT only when objects, morphisms, types, and required laws are specified; otherwise call them an analogy or design language.

## 7. Interface

Truth-OS returns an EpistemicReport as defined in NoieTruthAGENTS.md and AGENTS.md. It reports epistemic status and evidence; it does not choose policy, grant permissions, or certify unavailable runtime capabilities.
## 8. Claim decomposition procedure

Before evaluating a complex statement, split it into separately testable propositions while preserving qualifiers. Retain who or what the claim concerns, predicate, population, place or jurisdiction, time interval, conditions, modality, comparison, and quantifier. Preserve the original wording alongside normalized claims so paraphrase cannot silently strengthen or weaken the assertion.

For each component, identify whether its support is a direct observation, source report, derivation, model-based inference, assumption, or normative premise. A chain of individually supported claims may still fail if an inference step is invalid or a qualifier is lost. Report the link that fails instead of applying one status to the whole paragraph.

## 9. Calibration protocol

Calibration applies to forecasts or probabilities that refer to a defined event and can later be scored. Store the forecast time, event definition, resolution rule, horizon, reference class, predicted probability, outcome, and source of resolution. Compare forecasts with outcomes over an appropriate collection of cases and disclose sample size and distribution shift.

Proper scoring rules such as the Brier score or logarithmic score assess probabilistic forecasts in relation to outcomes; reliability diagrams inspect agreement between forecast bins and observed frequencies. Each metric has limits: binning choices, small samples, selective resolution, and changing event populations can distort apparent calibration. A metric does not certify an individual claim and no universal pass threshold applies across domains.

## 10. Freshness and contradiction review

Freshness is claim- and decision-specific. Determine whether the claim is time-sensitive, the source update cycle, the decision horizon, and which event would make it stale. A calendar interval is one review trigger, not a substitute for event monitoring or direct revalidation. If the current status cannot be checked, report UNKNOWN or STALE as appropriate.

When claims conflict, compare their exact scope, definitions, time, units, population, source lineage, method, and assumptions. Possible outcomes include compatible-after-scope, unresolved, source-corrected, one claim refuted within scope, or error in normalization. Preserve both sides and the reason for disposition. Consensus, recency, or confidence rhetoric alone does not resolve a factual disagreement.

## 11. Unknown-state distinctions

Use a status that names the missing operation or evidence:

| Status | Use when |
| --- | --- |
| UNKNOWN | Available information does not determine the answer |
| NOT_CHECKED | No applicable verification step was run |
| UNAVAILABLE | A required source, tool, permission, or capability could not be accessed |
| NOT_IDENTIFIED | The target quantity does not follow from the stated model and evidence |
| CONTESTED | Relevant evidence supports materially incompatible conclusions |
| OUT_OF_SCOPE | The claim exceeds the declared domain or model |
| INSUFFICIENT_EVIDENCE | Evidence was reviewed but does not support the requested conclusion |

These statuses describe different gaps. Do not replace them with an invented low confidence score.
