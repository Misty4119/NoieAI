# COUNTERFACTUAL.md

## Counterfactual reasoning v2.3

**Status:** This module specifies the questions and assumptions required for counterfactual analysis. It contains no attested runtime implementation.

## 1. Meaning

A counterfactual asks about an outcome under a hypothetical intervention while retaining information about a factual context. In a structural causal model, the same exogenous context is evaluated under different interventions. The answer depends on the model, unit-level information, and assumptions; observational association alone does not determine it.

## 2. Required model

Record the structural equations or transition semantics, graph, exogenous-variable assumptions, intervention, factual observations, target population or unit, and outcome. A DAG is required only for a DAG-based acyclic structural causal model; feedback and temporal settings require a representation with explicit semantics.

## 3. Identification and calculation

Separate:

- whether the counterfactual expression is identified from the declared model;
- whether the necessary factual and intervention data are available;
- whether an estimator or exact calculation was actually run;
- the uncertainty and sensitivity of the result.

The ctf-calculus transforms counterfactual probability expressions using three rules constrained by the causal diagram. Counterfactual independence is checked using an Ancestral Multi-world Network and d-separation. These methods may simplify or identify a query under their assumptions; they do not make all counterfactuals identifiable.

Return NOT_IDENTIFIED, PARTIALLY_IDENTIFIED, NOT_COMPUTED, or a result with a reproducible method and uncertainty. Never insert a constant probability or confidence score as a placeholder.

## 4. Decision use

Counterfactuals can compare options or explain model-dependent effects. They do not show that an alternative action was physically feasible, permitted, or ethically preferable. Pass results to Logic-OS with assumptions, data scope, uncertainty, and limitations.

## 5. Audit

Record query, model and assumptions, factual inputs as safe references, intervention, identification result, estimator or checker, uncertainty, and any decision that consumed the result. Do not store hidden reasoning traces.
## 6. Structural counterfactual procedure

For a structural causal model with factual evidence e and proposed intervention do(X=x), a unit-level counterfactual analysis follows three conceptually distinct operations:

1. **Abduction:** update the admissible exogenous contexts using factual evidence and the declared model.
2. **Action:** replace the structural assignment for X with x, leaving other structural assignments unchanged unless the intervention specification says otherwise.
3. **Prediction:** propagate the modified model for each admissible context and report the resulting outcome or distribution.

This procedure is model-relative. If factual evidence does not determine a unique exogenous context, preserve the resulting distribution or identified set rather than selecting one convenient context. If the model is stochastic, declare how exogenous noise is shared across factual and hypothetical worlds.

## 7. Query classes and interpretation

Keep distinct: an individual counterfactual (“what would this unit's outcome have been?”), a population counterfactual distribution, a potential-outcome contrast, a causal explanation of a realized event, and a policy comparison. They use different estimands and may need different assumptions.

The factual and hypothetical worlds share structural mechanisms only to the extent specified by the model. Changes to the model itself, altered measurement, interference between units, treatment-version differences, or post-intervention selection can invalidate a naive comparison. State whether the target is point-identified, set-identified, estimated, or only explored in a scenario.

## 8. Sensitivity and communication

For consequential counterfactuals, identify the assumptions that drive the answer and test plausible alternatives where data and methods permit. Report model alternatives, sensitivity range or reason unavailable, and the factual information used. Avoid language implying certainty about an unobserved alternative history. A counterfactual explanation can support a decision review but cannot by itself determine blame, intent, or policy.

A compact report includes question, factual context, intervention, model and scope, identification status, calculation method, result, uncertainty, assumptions, sensitivity, and limitations. Pass this report to Logic-OS without treating it as permission or a policy recommendation.
