# OBSERVER_PROTOCOL.md

## L2 — Observation context and feedback

This module records how an observation was produced and whether an action may have changed the observed system. It is a documentation contract, not an observation, quantum-measurement, or causal-analysis runtime.

## Observation record

For a claim based on an observation, record the relevant fields when known:

| Field | Meaning |
| --- | --- |
| `observer` | Person, instrument, service, or declared model that produced the record |
| `target` | Object or system observed, with a stable identifier where available |
| `method` | Measurement or sampling procedure and its version |
| `conditions` | Relevant time, location, calibration, and environmental conditions |
| `intervention` | Whether an action was deliberately taken; distinguish it from passive observation |
| `selection` | Inclusion, missingness, and selection effects that may affect interpretation |
| `limitations` | Resolution, uncertainty, and known failure modes |
| `provenance` | Source records and transformations, as described in `PROVENANCE_CHAIN.md` |

Do not label an observer “independent” merely because it has a different identifier. Independence is a property of the evidence-generation process and requires an explicit assessment of shared sources, instruments, data, and incentives.

## Feedback and interpretation

An agent's action may change a later observation. Record the action, target, time, and plausible pathways when relevant; use a causal model only when its assumptions are stated. Observation alone does not identify intervention effects.

Relational quantum mechanics is one interpretation of quantum theory. Its observer-relative language does not license arbitrary subjectivity or establish that ordinary claims become true relative to an observer. Keep experimentally supported predictions separate from interpretive commitments.

## Confidence and reporting

Observer context is evidence metadata. It does not supply a universal confidence multiplier. Do not automatically multiply confidence because an observer is “independent,” or halve it because observation is “coupled.” Report the source, method, uncertainty, and inference scope instead. Numerical probabilities require an explicit model, reference class, and calibration basis.

## Capability boundary

This repository defines no live sensor, observer detector, calibration process, or feedback controller. A host must attest any such capability before it is reported as available.
## Independence, dependence, and intervention checklist

When multiple observers or channels appear to corroborate a claim, compare how each result was generated:

- shared upstream dataset, reference text, training source, sensor, clock, calibration, or processing software;
- common selection criteria, sampling frame, or missing-data process;
- dependence on an earlier observer's report or a copied summary;
- common incentives or protocol constraints that may affect reporting;
- whether one measurement was an intervention that changed the target before another measurement.

Record known dependence and leave unknown dependence explicit. Independence is not established by organizational separation or different device names. Where evidence dependence cannot be quantified, avoid multiplying likelihood ratios or treating observations as independent replicates.

For observation-action feedback, include chronology and distinguish passive measurement, randomized assignment, deliberate intervention, and uncontrolled action. A before/after difference may reflect time trends, regression to the mean, selection, or concurrent changes; causal interpretation requires a suitable design or model.

## Cross-observer comparison

Two reports can be compared only after aligning the target, observable, units, coordinate frame, time, measurement operator, preprocessing, and uncertainty. A disagreement can arise from different perspectives, instruments, resolutions, or conditioning information. Preserve each report and compare predicted observable consequences under the declared model.

Relational quantum mechanics is limited to its formal quantum setting and remains an interpretation. The metadata contract here is useful for ordinary measurement provenance and does not adopt observer-relative truth as a general epistemology.
