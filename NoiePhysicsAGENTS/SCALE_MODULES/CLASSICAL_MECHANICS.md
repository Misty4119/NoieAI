# Classical Mechanics Reference (Physics-OS v2.3)

**Domain:** Newtonian mechanics, analytical mechanics, rigid-body dynamics, small oscillations, and classical waves. This is a physical reference specification, not a solver or safety controller.

## Applicability and model selection

Classical mechanics is highly predictive for macroscopic, low-speed, non-quantum systems under an appropriate gravitational approximation. Applicability depends jointly on speed relative to light, action relative to ℏ, field strength, material structure, required precision, and boundary conditions; a fixed length in meters cannot determine it. Choose an inertial frame or explicitly include inertial forces in a non-inertial frame. The framework is established within its domain; that does not mean every equation has been verified or solved by this repository.

## Newtonian form and system boundary

For a fixed-mass particle in an inertial frame, ΣF = dp/dt = m a. More general momentum balance must define the control volume and momentum flux; a rocket, jet, or variable-mass system cannot be modeled by applying F = m a to the changing-mass body alone. List the sources, points of application, and time dependence of forces, and check units and system boundaries.

Newton's third law describes paired interaction forces under the assumptions of the chosen model. A particle subsystem with electromagnetic fields may not exhibit instantaneous paired particle forces; total momentum conservation is checked only after including field momentum in the complete closed system. Whether a conserved quantity exists also depends on external forces, boundary fluxes, and symmetries.

The point-mass inverse-square form F = G m₁m₂/r² is a Newtonian approximation. Extended bodies require integration over their mass distributions; use a relativistic model for compact objects, strong fields, or precision requirements outside this approximation. A point-mass singularity as r approaches zero marks model failure; an arbitrary cutoff is not a physical value.

## Lagrangian mechanics and constraints

The action S = ∫L(q,q̇,t)dt is stationary for physical paths under specified endpoint conditions; it is not universally minimized. For a conservative system with L = T−V, the Euler–Lagrange equations are d/dt(∂L/∂q̇ᵢ)−∂L/∂qᵢ=0. Generalized coordinates can absorb geometric constraints, but the coordinate chart, degrees of freedom, and singularities must be stated.

Holonomic constraints f(q,t)=0 can be added using multipliers λ; the standard variational derivation assumes ideal constraint forces do no virtual work. Nonholonomic velocity constraints, friction, dissipation, impacts, and nonconservative generalized forces require separate models; the multiplier method is not universal. Redundant constraints can make multipliers non-unique.

If L is invariant under time translation, the associated energy is conserved; if a coordinate is cyclic, its canonical momentum is conserved. The correspondence between Noether symmetries and conserved quantities depends on the actual action and boundary conditions. External driving or dissipation changes the conservation statement.

## Hamiltonian form and canonical structure

Canonical momentum is pᵢ=∂L/∂q̇ᵢ. A Legendre transform H(q,p,t)=p·q̇−L is available only when regularity conditions, including an invertible velocity Hessian, hold. Hamilton's equations are q̇ᵢ=∂H/∂pᵢ and ṗᵢ=−∂H/∂qᵢ. H equals total energy only for appropriate standard systems and may depend explicitly on time.

Poisson brackets require a consistent coordinate order and sign convention. Canonical transformations preserve the symplectic form. In long integrations, phase-space geometry and energy drift may matter more than single-step local error. Constrained Hamiltonian systems, singular Lagrangians, and gauge freedoms require additional formalisms; do not apply the ordinary invertible transform without qualification.

## Rigid-body rotation

The inertia tensor I is defined from the mass distribution and reference point. Separate center-of-mass translation from rotation about the center of mass. In body coordinates, angular momentum is L=Iω; for I fixed in body coordinates, rotating-frame equations include the ω×(Iω) term. The diagonal principal-moment form applies only in principal-axis coordinates. Keep torque, reference point, inertia updates, attitude parameterization, and coordinate transforms consistent.

Euler angles become singular at particular attitudes. During attitude integration, monitor rotation-matrix orthogonality or quaternion norm. Gyroscopic precession formulas apply only under specified symmetry, spin, and torque approximations, not to every rigid body or force configuration.

## Small oscillations, damping, and waves

Expanding potential energy to second order around a stable equilibrium gives M q̈ + C q̇ + K q = f. Undamped modes come from a generalized eigenvalue problem; decoupling modes requires the relevant properties of the mass and stiffness matrices and boundary conditions. A single undamped oscillator has natural angular frequency ω₀=√(k/m). For viscous damping, ω_d=√(ω₀²−γ²) applies only to an underdamped case with γ<ω₀ and the stated damping convention.

The ideal one-dimensional uniform-string wave equation is u_tt=c²u_xx, with c=√(T/μ), assuming constant tension T and line density μ and small amplitude. Material dispersion, damping, nonlinearity, changing geometry, and boundary reflections require additional models; the plane-wave formula does not solve an arbitrary boundary-value problem.

## Validation and failure diagnosis

Check dimensions, coordinate conventions, initial and boundary conditions, force closure, and approximation conditions. Cross-check analytic limits, energy and momentum balances, coordinate transformations, timestep convergence, and independent reference cases. Long-term drift may come from omitted physics, the numerical method, timestep, event handling, or open-system flux; do not automatically call it a new physical effect or a software defect.

At minimum, distinguish model-domain mismatch, inconsistent inputs or units, incompatible constraints, numerical non-convergence, inapplicable conservation checks, and unavailable solver capability. This document provides no state estimator, integrator, force feedback, or authorization for real-world action.

## Cross-module boundary

Continuum stress, materials, and finite deformation are covered by CONTINUUM_MECHANICS.md; fluids by FLUID_DYNAMICS.md; electromagnetic forces and field momentum by CLASSICAL_ELECTROMAGNETISM.md; high-speed or strong-gravity cases by SPECIAL_RELATIVITY.md and GENERAL_RELATIVITY.md; quantum and thermal-statistical descriptions by QUANTUM_MECHANICS.md and STATISTICAL_MECHANICS.md.