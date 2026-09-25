# CONFIDENCE_CALIBRATION.md

## Probability calibration protocol v2.3

Calibration applies to forecasts of a defined event over a specified population and horizon. It does not calibrate a language model’s unqualified “confidence,” prove an individual claim, or replace source review.

Before evaluation, define the event, forecast time, eligible cases, information available at that time, probability format, resolution rule, outcome source, sampling process, evaluation period, and scoring rule. Freeze predictions before outcomes are known. Handle unresolved, censored, or revised outcomes explicitly and prevent hindsight leakage.

For binary outcomes y∈{0,1} and forecast p, the Brier score is (p−y)²; logarithmic score evaluates log probability assigned to the realized outcome. Lower average proper score is better on the chosen sample, but comparisons need uncertainty, a baseline, and identical tasks. Reliability diagrams compare forecast bins with observed frequencies; bin size and sample count matter. Expected calibration error is a bin-dependent descriptive statistic, not a proper scoring rule or universal pass threshold.

Report sample size, uncertainty intervals, subgroup results, missingness, dependence between cases, discrimination separately from calibration, and distribution shift. Calibration on one population does not guarantee calibration on another or correctness in a particular case. No forecasting history, evaluation dataset, fitting tool, or monitoring pipeline is included here.
