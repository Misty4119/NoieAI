# EIG_CALCULATOR.md

## Expected information gain v2.3

For a hypothesis Θ, candidate observation Y, and declared prior and predictive model, expected information gain can be defined as mutual information:

$$EIG(Y)=I(\Theta;Y)=H(\Theta)-E_Y[H(\Theta\mid Y)].$$

Equivalently, it is the prior expectation of the KL divergence between posterior and prior under possible observations. This quantity is defined only after specifying the hypothesis space, prior, likelihood, observation outcomes, and entropy convention. Approximation error, model misspecification, and sensitivity to the prior must be reported.

EIG measures expected reduction in uncertainty under that model. It does not measure truth, task utility, safety, permission, privacy cost, or value of the observation. A candidate with high EIG may be irrelevant to the decision or too costly or risky. A lower-EIG check may be required by a host policy.

Report the candidate, model, prior, likelihood, expected outcomes, estimator, approximation, sensitivity, resource cost, and limitations. If probabilities or outcomes are unsupported, return NOT_ESTIMABLE rather than filling in defaults. No information-gain estimator or sensor-selection service is implemented here.
