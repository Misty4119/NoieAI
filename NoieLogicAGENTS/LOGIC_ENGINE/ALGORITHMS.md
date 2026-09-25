# ALGORITHMS.md

## Causal algorithm selection v2.3

**Purpose:** Describe algorithm families and their assumptions so a runtime can select or reject a method. This catalog is not a package and does not attest that any algorithm is implemented.

## 1. Causal discovery is assumption-dependent

Observational data alone generally does not select one unique causal graph. Results depend on variables measured, latent confounding, data quality, sample size, functional assumptions, conditional-independence tests or score, and background constraints. Report the equivalence class and unresolved orientations where appropriate.

Before selecting an algorithm, define the discovery target, variable types, sample and time structure, allowed causal representations, latent-variable assumptions, and validation plan.

## 2. Common algorithm families

| Family | Examples | Typical assumptions and output |
| --- | --- | --- |
| Constraint-based | PC | Conditional-independence tests; commonly assumes causal sufficiency, causal Markov and faithfulness conditions; returns a partially directed equivalence representation under its assumptions |
| Latent-variable constraint-based | FCI | Allows latent confounding under its formal assumptions; returns a partial ancestral graph representing an equivalence class |
| Score-based | GES | Searches equivalence classes using a chosen score and search procedure; output depends on score, assumptions, and search behavior |

These are method families, not guarantees. State the exact variant, test or score, parameters, stopping rule, data handling, and software/runtime version. Do not attach benchmark values without dataset, metric, comparison, and reproducible conditions.

## 3. Intervention selection

Intervention planning is a decision problem. Candidate interventions require a declared causal model, feasible action set, costs, risk and policy constraints, and uncertainty. Information gain can guide data collection only when a probability model and utility for information are stated. It does not itself choose an ethically or legally acceptable intervention.

## 4. Counterfactual generation

Counterfactual queries require a structural or other explicit model connecting factual and hypothetical states. Keep identification, estimation, and explanation separate. Use COUNTERFACTUAL.md for query assumptions and result states.

## 5. Runtime and validation requirements

A deployment that claims an algorithm is active must attest the implementation, version, supported data types, assumptions, resource limits, and validation results. Pseudocode, a model name, a paper, or a performance claim does not provide runtime capability.

For each result, record input provenance, preprocessing, assumptions, model, algorithm, parameters, tests, result, uncertainty, and known limitations. A discovered graph is a hypothesis subject to validation, not automatically a true causal structure.
## 6. Selection and result handling

| Question | Candidate family | Minimum result to retain |
| --- | --- | --- |
| Is a conditional independence compatible with the sample? | CI test used by a constraint-based method | Test statistic or decision, conditioning set, threshold, sample size, missingness handling, and power limitations |
| Which graph structures fit observed independences? | PC when causal sufficiency is defensible; FCI when latent common causes must be allowed | CPDAG or PAG, equivalence class, edge-orientation status, and assumptions |
| Which equivalence class optimizes a chosen score? | GES or a specified score-based search | Score definition, search procedure, starting conditions, stopping rule, and returned equivalence representation |
| Which experiment is worth running? | A declared intervention-design or value-of-information method | Target estimand, candidate set, expected outcomes, utility/cost basis, feasibility and permission checks |

Do not select a method solely by its acronym or a benchmark rank. The same data can support multiple Markov-equivalent structures. Edge direction may remain unresolved even when the skeleton is stable. Background knowledge may orient edges only when its source, scope, and consistency with the data are checked.

## 7. Algorithm-family detail

### 7.1 Constraint-based discovery

PC starts from a complete undirected graph, removes edges when a conditional-independence test supports separation, then orients compelled structures under the selected orientation rules. Under causal Markov, faithfulness, causal sufficiency, and an appropriate test, its output is interpreted as a CPDAG for a Markov-equivalence class. Finite-sample test errors, hidden common causes, selection bias, and near-unfaithful distributions can change the result. Do not treat an oriented edge as experimentally confirmed.

FCI relaxes causal sufficiency and may return a PAG representing equivalence classes with latent confounding. Its endpoint marks distinguish compelled direction, possible direction, and unresolved relations under the selected rules. State which FCI variant, conditional-independence test, background constraints, and orientation rules were used; do not replace an unresolved endpoint with a convenient direction.

### 7.2 Score-based search

GES searches over equivalence classes by adding and then deleting edges to improve a specified score. Its result depends on the score, search operators, data preprocessing, and assumptions under which the score is meaningful. A local optimum or high score does not prove causal truth. Preserve the CPDAG, score trace or reproducible artifact, and sensitivity to material choices when available.

### 7.3 Intervention design

Information gain is tied to a target random variable and probability model. For a design d and unknown parameter or hypothesis Θ, expected information gain may be defined as I(Θ;Y|d) under a declared predictive distribution. This is not interchangeable with expected treatment effect, safety, or social value. Candidate designs still require feasible controls, measurement plans, stopping criteria, cost accounting, and human or policy authorization.

## 8. Validation and failure cases

Before interpreting a learned graph, check variable definitions and time order; missingness and selection mechanisms; measurement error; duplicate or post-treatment variables; support/positivity; dependence between samples; and whether the CI test or score matches the data type. Where assumptions cannot be tested from the data, mark them as assumptions and analyze plausible alternatives.

Use explicit result states: MODEL_UNDECLARED, ASSUMPTIONS_UNCHECKED, UNDERPOWERED, NONIDENTIFIED, PARTIALLY_ORIENTED, DIAGNOSTIC_FAILURE, RESOURCE_LIMIT, or RESULT_AVAILABLE. A result may be useful while still being incomplete. Report unresolved graph structure and sensitivity instead of forcing a single DAG.
