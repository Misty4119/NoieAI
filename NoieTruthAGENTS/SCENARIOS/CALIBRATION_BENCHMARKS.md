# CALIBRATION_BENCHMARKS.md

## Calibration benchmark design

This file specifies how to design a benchmark; it contains no executed results or universal pass thresholds.

Define a forecasting task, target population and horizon, probability format, scoring rule, outcome source, resolution policy, baseline, sample size rationale, and evaluation period before collecting results. Freeze predictions before outcomes. Report proper scores and reliability analysis with uncertainty and subgroup breakdowns; describe discrimination and calibration separately.

Compare systems on the same information, tasks, and resolution rules. Document exclusions, missing outcomes, dependence between cases, and distribution shift. Do not infer individual truth from aggregate calibration, and do not label performance “passing” using a threshold without a use-specific, predeclared rationale.

Status: `DESIGN_ONLY` until an identified implementation runs the protocol and publishes reproducible results. No benchmark runner or dataset is included here.
## Evaluation cases and failure handling

Include routine and edge cases: forecasts near 0 or 1; rare events; small subgroups; repeated or correlated cases; delayed or ambiguous outcome resolution; changed event definitions; missing forecasts; selective abstention; and distribution shift. Freeze the scoring and exclusion rules before evaluation.

Report reliability by probability range with counts and uncertainty, proper scores relative to a declared baseline, resolution coverage, subgroup results where sample size permits, and calibration/discrimination separately. If outcome resolution is disputed, keep it unresolved instead of forcing a label. A high aggregate score must not hide systematic errors on a material subset.

A benchmark run is not a universal certification. State the population and deployment conditions for which the results are informative, what was not measured, and the next review trigger. Status remains DESIGN_ONLY or NOT_RUN until an actual reproducible evaluation exists.
