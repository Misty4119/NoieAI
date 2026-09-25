# SANDBOX/README.md

## Physics simulation requirements

This directory is reserved for design documentation about physical simulation. It contains no simulator, isolated runtime, simulation dataset, or validated configuration. The label “sandbox” does not guarantee that a run is isolated from the real system.

Before using a simulation, the host must attest the actual environment and isolation boundary, model and version, initial state, parameters, domain, validation evidence, and relevant limitations. Follow host permission and approval requirements. A simulation predicts only within its model and tested conditions; it is not proof of physical safety or a guarantee that real-world execution will match.

Use a model and solver appropriate to the scale, system boundary, and question. Record uncertainty, sensitivity, omitted processes, and whether a result was simulated, measured, or assumed. Do not claim a run occurred from pseudocode or a scenario description.
## Simulation review record

A simulation request should identify the question and decision it informs, model and solver, system boundary, initial and boundary conditions, parameter sources, units, uncertainty, and intended validity range. State the execution environment, isolation boundary, seed or stochastic method, numerical tolerances, and actual run identifier if the host performed a run.

Review cases that matter to the model: nominal conditions, boundary states, parameter uncertainty, sensor noise, disturbances, mode changes, missing inputs, solver failure, timeout, and cancellation. Compare against analytic limits, independent measurements, benchmark cases, or conservation/balance checks where justified. Preserve failed and null runs; do not report only favorable scenarios.

Report numerical error, parameter uncertainty, measurement uncertainty, and model uncertainty separately when the method supports them. A finite scenario set cannot establish safety against all unmodeled conditions. A simulation result is not physical observation, formal proof, or policy permission.
