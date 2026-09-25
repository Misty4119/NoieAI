# ABDUCTIVE_REASONING.md

## Candidate explanation generation

Abduction proposes one or more explanations that could account for observations. A plausible explanation is not thereby identified as the actual cause. This document specifies a reasoning task; it provides no anomaly detector, causal learner, hypothesis database, or autonomous model update.

## Report contract

For each candidate, state:

- the observations and provenance being explained;
- the candidate assumptions and causal mechanism;
- alternative explanations, including measurement error and missing variables;
- evidence that would distinguish candidates;
- scope, uncertainty, and whether probabilities are supported by a model;
- next checks and possible disconfirming evidence.

Rank candidates only under an explicit comparison rule, evidence model, and assumptions. A prior probability must come from a stated reference class or elicitation method. Description length or simplicity may be a heuristic within a specified model class; it is not proof of truth. Do not use severity as a confidence estimate, insert default probabilities, or treat a “best” explanation as established.

## Downstream use

Pass candidate explanations and uncertainty to the relevant Truth-OS and Physics-OS checks. Logic-OS may consider them in a decision only alongside host policy, permission, physical feasibility, and the cost of being wrong. Do not automatically rewrite a knowledge base or execute an intervention from an abductive result.

## Capability boundary

Any executable detector, causal learner, statistical score, or update pipeline must be identified, validated, scoped, and attested by the host. This Markdown document does not implement one.