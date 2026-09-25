# SANDBOX_TESTS.md

## Simulation review scenarios

The scenarios in this file are test-design prompts, not executed tests or evidence of a working sandbox. The repository contains no simulation runtime. Root `AGENTS.md` governs capability reporting and failure handling.

## Scenario template

For each proposed simulation, record:

- the decision question and alternatives;
- the actual sandbox provider and isolation boundary, or `UNAVAILABLE`;
- model, version, input provenance, initial conditions, and assumptions;
- the range of scenarios and important omitted states;
- expected observations and failure criteria selected before execution;
- model validation evidence and known limits;
- actual run identifier, tool output, and independent review, if a run occurred;
- conclusion limited to the modeled conditions and uncertainty;
- required human approval and follow-up.

Do not claim “no risk,” “safe,” or a predicted real-world outcome because a scenario completed. A simulation cannot establish policy permission, prove model completeness, or rule out unmodeled conditions. Formal checking is a separate capability with a separately stated property and proof scope.

## Test status

Use `DESIGN_ONLY`, `NOT_RUN`, `RUN_FAILED`, `RUN_COMPLETED_WITH_LIMITS`, or `REVIEWED_WITHIN_SCOPE`. Never label an illustrative scenario as passed. Report the exact tool and environment before describing results.
## Coverage matrix

Before an actual run, include at least the relevant cases below and predeclare expected failure behavior:

| Case | What it probes |
| --- | --- |
| Nominal model-valid input | Baseline behavior and reproducibility |
| Boundary values | Discontinuities, singularities, zero or maximum inputs, and domain edges |
| Out-of-domain state | Whether the tool reports invalid scope instead of extrapolating silently |
| Missing or stale input | Failure propagation and whether a dependent action is blocked |
| Contradictory assumptions | Detection of incompatible constraints or premise sets |
| Perturbed parameters | Sensitivity and stability of the reported outcome |
| Partial execution or timeout | Difference between incomplete and completed results |
| Cancellation or shutdown | Whether further effects stop and committed state is reported |
| Tool or model failure | Explicit error status rather than a plausible fabricated result |

A useful test suite records inputs, seeds, versions, environment, outputs, expected result, observed result, tolerances, and reviewer. It should separate software correctness from physical model validity and policy acceptability. A simulation that passes its declared suite supports only the tested properties and conditions.
