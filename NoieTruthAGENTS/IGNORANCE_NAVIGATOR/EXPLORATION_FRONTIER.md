# EXPLORATION_FRONTIER.md

## L3 — Unknowns and information-gathering choices v2.3

**Purpose:** Turn a declared knowledge gap into candidate information-gathering steps. “Frontier” is a planning metaphor for unresolved claims or model distinctions, not a literal knowledge manifold or automatically detected boundary.

## 1. Unknown record

For each material unknown, state:

- the decision or claim it affects and the exact unresolved question;
- current evidence, provenance, scope, freshness, and contradictions;
- what is unknown: missing observation, source status, model assumption, causal identification, interpretation, or capability;
- possible answers and what evidence would distinguish them;
- the cost of acting while unresolved, the cost and risk of obtaining information, and who may authorize the inquiry.

Do not create an exploration target merely because a topic is unfamiliar. It should connect to the user’s goal, a material uncertainty, or an explicit research objective.

## 2. Candidate information steps

For each candidate source, measurement, experiment, calculation, or expert review, record expected outcomes, time and resource cost, access or permission requirements, feasibility, safety conditions, and how the result would change the claim or decision. Identify shared upstream sources so repeated copies are not counted as independent.

Expected information gain is available only under an explicit hypothesis space, prior, likelihood, observation model, and outcome set. For hypotheses H and candidate observation Y, it may be written as the expected reduction in entropy:

$$EIG(Y)=H(H)-E_{Y}[H(H\mid Y)].$$

This is a model-dependent information quantity. It does not include the social value, physical risk, permission, privacy cost, or decision utility of collecting Y. Add those considerations through the host’s decision process; do not let an EIG ranking authorize an observation.

## 3. Prioritization and stopping

Prefer a candidate only when its expected contribution is relevant to the unresolved question and its cost, uncertainty, and permissions are understood. A high information gain may still be irrelevant, unsafe, inaccessible, or too expensive. A low-gain check may be required to meet a verification or safety policy.

Stop or defer when no candidate can resolve the material uncertainty, the required capability is unavailable, permission is missing, the expected value cannot be assessed, or the user’s decision no longer depends on the answer. Report the limitation and a reasonable future trigger for reopening the question.

## 4. Output

Return a scoped exploration plan with the unknown, candidate steps, expected discriminating evidence, assumptions, costs and risks, permission/capability needs, and stop condition. Use NOT_ESTIMABLE when probabilities or outcomes are not supported. No frontier detector, sensor selector, source crawler, or exploration optimizer is included in this repository.
