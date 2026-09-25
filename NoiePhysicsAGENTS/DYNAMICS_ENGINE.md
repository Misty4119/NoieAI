# DYNAMICS_ENGINE.md

## Physics model and dynamics contract

This module routes physical-dynamics models. It does not generate equations or execute trajectories. Models must be selected for the object, scale, boundary conditions, and forces involved; no single formalism covers every system.

## Model report

A dynamics result should identify:

- system boundary, degrees of freedom, state variables, units, and initial conditions;
- governing equations and constitutive or interaction model;
- parameter sources, calibration, and uncertainty;
- boundary conditions and numerical method, including step size and tolerances where relevant;
- validation range, conservation checks appropriate to the modeled system, and known failure modes;
- predicted state or trajectory with a validity horizon and uncertainty;
- actual solver/simulator capability and version, when a runtime was used.

Lagrangian and Hamiltonian methods are useful formulations under their assumptions; they are not mandatory universal interfaces. Dissipative, constrained, stochastic, relativistic, continuum, quantum, and hybrid models may require different state descriptions and numerical methods.

## Interpretation limits

A computed trajectory is conditional on the model, inputs, approximation, and solver. It is not an observation of the future. Energy conservation is a useful diagnostic only when the modeled system and symmetry justify it; an open or dissipative system can exchange energy across the chosen boundary. Simulation does not prove real-world safety or policy permission.

## Submodules

- `DYNAMICS_ENGINE/MOTION_GENERATOR.md`: illustrative equation-generation interface.
- `DYNAMICS_ENGINE/COLLISION_SYSTEM.md`: collision geometry and response examples.
- `DYNAMICS_ENGINE/SWARM_DYNAMICS.md`: multi-agent dynamics examples, not a factual consensus service.

Examples are specifications or pseudocode unless a matching implementation and validation are attested by the host.
## Model selection guide

| System characteristics | Candidate formulation | Conditions to make explicit |
| --- | --- | --- |
| Conservative finite-dimensional system | Newtonian, Lagrangian, or Hamiltonian mechanics | Degrees of freedom, constraints, generalized coordinates, forces, conserved quantities |
| Open or dissipative system | Balance laws, constitutive equations, or non-Hamiltonian dynamics | System boundary, fluxes, dissipation law, driving, thermal coupling |
| Continuum material or fluid | Continuum mechanics or field equations | Constitutive model, geometry, boundary and initial conditions, closure assumptions |
| Relativistic regime | Special- or general-relativistic model | Metric or inertial frame, stress-energy assumptions, approximation and domain |
| Quantum regime | Declared quantum state and dynamical model | State preparation, observables, Hamiltonian or channel, measurement and decoherence assumptions |
| Coupled discrete/continuous system | Hybrid model | Mode definitions, switching conditions, event handling and numerical stability |

No row is selected by scale label alone. A model's equations, material law, boundary data, and validity range determine which computations are justified.

## Calculation workflow

1. Define the system boundary, state, units, frame, time interval, and quantity of interest.
2. Select governing equations and justify the approximation and parameter values.
3. Check that initial and boundary conditions are sufficient and mutually compatible.
4. Select a numerical method suited to stiffness, conservation structure, constraints, discontinuities, and required error tolerance.
5. Check convergence under step-size or resolution changes; compare against analytic limits, invariants, benchmark cases, or independent measurements where available.
6. Report numerical error separately from parameter, measurement, and model uncertainty.
7. State the prediction horizon and conditions that invalidate extrapolation.

For a closed conservative Hamiltonian model, an appropriate symplectic method can preserve geometric structure over long integrations better than generic methods; it does not remove discretization error. Adaptive methods can help control local error but require stability and event checks. Conservation is a diagnostic only when the model's symmetry and system boundary imply that conserved quantity.

## Collisions and material interaction

Collision calculations need geometry, motion, contact criterion, mass or inertia, material/contact law, and boundary conditions. Broad-phase candidate generation and narrow-phase contact testing are separate computational stages in implementations. Response laws may use restitution, friction, compliance, or constraints; the appropriate law depends on the modeled materials and regime. Report penetration tolerance, impact-time uncertainty, and energy loss or gain under the declared model.

A visual appearance or one sensor reading does not establish material identity. Material classification should retain candidate classes, discriminating measurements, and model confidence; do not infer a constitutive law from a label alone.
