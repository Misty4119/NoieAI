# SANDBOX

This directory is reserved for documentation and scenario templates for isolated simulations. The repository contains no sandbox runtime, simulation records, configuration directory, or Pareto solver.

A simulation report must identify the actual environment and isolation boundary, model and version, input state, assumptions, scope, scenario coverage, outputs, and validation limitations. Simulation results are conditional on the model; they do not prove real-world safety, permission, or outcome.

Run a simulation only when the host attests the required isolated capability and authorizes its use. If no such runtime is available, report `UNAVAILABLE` or `UNATTESTED` and do not imply that a simulation ran. For current failure semantics, use root `AGENTS.md` and `NoieLogicAGENTS.md`.
## Scenario review and evidence

For each run, capture the question, model scope, scenario parameters, initial state, seeds or sampling rule, runtime and isolation status, expected failure criteria, outputs, errors, resource limits, and review status. Include baseline, boundary, adversarial, missing-capability, and cancellation cases when relevant. If a model cannot represent a material failure mode, name it before interpreting the result.

Keep simulation results separate from formal verification, empirical validation, and approval. A simulated alternative may be infeasible in the physical world or unauthorized under policy. A run that ends cleanly does not show that no adverse scenario exists.
