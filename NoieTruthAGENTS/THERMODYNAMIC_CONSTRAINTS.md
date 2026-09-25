# THERMODYNAMIC_CONSTRAINTS.md

## Information processing and thermodynamic limits v2.3

**Scope:** Connect a measured physical information-processing process to thermodynamic quantities when the system, memory, environment, and operation are defined. This module does not assign energy levels to truth, confidence, or honesty.

## 1. Landauer bound

Landauer's principle gives a lower bound for the heat dissipated when a logically irreversible operation erases information under specified conditions. For erasing one initially unknown, unbiased bit in a degenerate memory coupled to a bath at temperature T, the familiar bound is kBT ln 2.

The bound is not a minimum energy cost for every computation, observation, acquired bit, or true belief. Initial probabilities, correlations, memory energies, bath conditions, and the full protocol matter. State the operation and assumptions before applying a bound.

## 2. What runtime measurements can show

If a runtime supplies trusted energy and temperature measurements, a system may record them for physical resource analysis. The measurement must identify its boundary, units, calibration, time interval, uncertainty, and source.

Energy use does not show that a claim is true, that an answer required a minimum amount of work, or that a model was honest. Computational effort, elapsed time, token count, and verification status are separate observables.

## 3. Optional computational records

A computation record may contain an operation identifier, tool/version, input and output references, elapsed time, resource measurements, and verification artifact. It is optional and capability-dependent.

Do not hash or retain hidden reasoning traces as a substitute for proof. Do not define a minimum effort threshold for epistemic certainty. A correct result may be produced with little computation; a wrong result may consume substantial energy.

## 4. Failure and status

When physical instrumentation or a trusted measurement path is unavailable, mark energy as unavailable. Do not estimate an exact physical cost from model tokens or a conceptual equation. When the process conditions do not match the stated Landauer case, report the bound as not applicable or provide a justified generalized analysis.

Use [Physics axioms](../NoiePhysicsAGENTS/AXIOMS.md) and [thermodynamics reference](../NoiePhysicsAGENTS/SCALE_MODULES/THERMODYNAMICS_PHYSICS.md) for physical context; use [epistemology axioms](EPISTEMOLOGY_AXIOMS.md) for claim status and calibration.
