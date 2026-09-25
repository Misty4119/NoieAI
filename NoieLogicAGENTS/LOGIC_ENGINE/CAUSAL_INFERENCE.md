# CAUSAL_INFERENCE.md

## Causal model and identification contract v2.3

**Scope:** Causal inference distinguishes association, intervention, and counterfactual questions under explicit model assumptions. This document is a specification, not executable software.

## 1. Model declaration

Every causal analysis records:

- variables, domains, time ordering, and target estimand;
- representation and semantics: for example, a DAG-based structural causal model, temporal model, or feedback model;
- structural equations or transition rules where required;
- assumptions about confounding, selection, measurement, consistency, positivity, and data generation;
- evidence and provenance for graph edges and restrictions;
- validation checks, uncertainty, and domain limits.

A static DAG is appropriate for an acyclic structural causal model. Feedback may be represented by an appropriate temporal, dynamic, or cyclic formalism with its own semantics. A graph shape alone is not a causal model.

## 2. Three distinct questions

- **Association:** what variables co-vary in observed data?
- **Intervention:** what distribution or outcome follows under a specified intervention, if identifiable under the model?
- **Counterfactual:** what would have happened to a unit under an alternative action, given factual information and a model linking the worlds?

Do-calculus transforms expressions under graphical separation conditions. Each rule application requires the correct graph surgery and d-separation test. Failure to identify an estimand is a valid result; do-calculus does not create missing assumptions or data.

Counterfactual graphical models and the ctf-calculus provide three transformation rules for counterfactual quantities under their diagram constraints. The associated Ancestral Multi-world Network is used to test counterfactual independence. This is a formal method within its declared model, not a guarantee that a requested counterfactual is identifiable or numerically estimable.

## 3. Identification, estimation, and decision

Keep these stages separate:

1. **Identification:** determine whether the target quantity follows from the model and available information; return identified, partially identified, or not identified.
2. **Estimation:** if data and a justified estimator exist, return estimate, uncertainty interval, data scope, and diagnostics.
3. **Decision:** Logic-OS evaluates that report under goals and policy.

ATE, back-door adjustment, and front-door adjustment are usable only when their respective definitions and assumptions hold. If any required criterion has not been checked, report it as unverified. Do not substitute a fixed number or a hand-chosen confidence score.

## 4. Output contract

~~~yaml
question: association | intervention | counterfactual
model: description and version, if available
domain_and_scope: required
assumptions: explicit
identification: identified | partially_identified | not_identified | not_checked
estimand: expression or description
estimate: optional; absent unless computed by an attested method
uncertainty: interval or method, otherwise unknown
diagnostics: performed checks and results
limitations: required
runtime_capabilities: attested tools or unavailable
~~~

Structural identification status, statistical uncertainty, empirical calibration, formal proof, and policy acceptability are independent fields.

## 5. Failure semantics

Return not checked when graph operations, conditional-independence testing, assumptions, or estimator support are missing. Never mark independence by default. An Enum or status value must be compared explicitly, not used as a truthy Boolean. Missing data, invalid identification conditions, failed diagnostics, or resource limits produce an explicit non-success status.

The sketches and algorithm catalog elsewhere in this directory are not executable implementations unless separately built and validated.
## 6. Structural causal model semantics

For an acyclic structural causal model, record at minimum M = (U, V, F, P(U)): exogenous variables U, endogenous variables V, structural assignments F, and any probability assumptions over U. The directed graph summarizes dependencies in F; it does not replace the equations, variable definitions, or distributional assumptions.

An observational conditional such as P(Y | X=x) describes selected cases with X=x. An intervention P(Y | do(X=x)) describes the model after replacing the assignment for X. These quantities coincide only under additional conditions. Never infer an intervention effect from association alone.

In a DAG-based analysis, do-calculus may simplify an interventional expression only when the corresponding graph separation condition holds after the specified edge operations. Record the graph version, expression before and after each rule, and separation result. If a rule's preconditions are not checked, mark the transformation NOT_CHECKED.

## 7. Identification examples and assumptions

For treatment A, outcome Y, and a measured covariate set Z satisfying a valid back-door criterion, with consistency and positivity, the adjustment formula is:

$$P(Y\mid do(A=a))=\sum_z P(Y\mid A=a,Z=z)P(Z=z).$$

The average treatment effect compares this quantity across the target treatment values. The formula does not supply the required graph, identify a suitable Z set, or prove that no unmeasured confounding exists. For continuous variables replace the sum with the justified integral; the target population and support still need to be stated.

A front-door estimand may be identified through mediator M only if the front-door conditions hold: the directed effect from A to Y is mediated through M; there is no unblocked back-door path from A to M; and A blocks the relevant back-door paths from M to Y. The familiar discrete formula is:

$$P(Y\mid do(A=a))=\sum_m P(M=m\mid A=a)\sum_{a'}P(Y\mid M=m,A=a')P(A=a').$$

Do not use this formula merely because a mediator variable exists. If treatment assignment, selection, interference, measurement, or positivity assumptions are unresolved, return a qualified or nonidentified result.

## 8. Sensitivity and transport

Identification in one study population does not automatically transport to another. Report inclusion criteria, population, intervention version, outcome definition, follow-up window, missing-data assumptions, and effect modifiers considered. Distinguish sampling uncertainty from sensitivity to model structure, unmeasured confounding, and measurement error. If a numeric sensitivity analysis was not run, say so.

## 9. Required result states

Return IDENTIFIED, PARTIALLY_IDENTIFIED, NOT_IDENTIFIED, NOT_ESTIMATED, NOT_CHECKED, or ERROR as separate states where needed. An estimand can be identified without a data-based estimate; an estimate can be produced under assumptions that remain disputed. Preserve this distinction in every downstream decision.
