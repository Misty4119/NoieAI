# Continuum Mechanics Reference (Physics-OS v2.3)

**Domain:** Continuum descriptions of solids and fluids, kinematics, stress, balance equations, and constitutive models. This document defines model conditions; it includes no material database or finite-element solver.

## Continuum hypothesis and descriptions

A continuum approximation represents microscopically discrete matter by differentiable fields. The characteristic length must be much larger than the microstructure scale, and the observation scale and averaging procedure must be stated. If cracks, grains, pores, rarefaction, or interfaces dominate, a homogeneous continuum model may fail; use a multiscale, discrete, or internal-variable model as appropriate.

Describe a material point using reference coordinates X and current position x=χ(X,t). Velocity is v=∂χ/∂t; deformation gradient F=∂x/∂X; J=det F. Admissible deformation usually requires J>0. J≤0 indicates a degenerate map or numerical failure, not a physically acceptable compressed state. Small strain ε=(∇u+∇uᵀ)/2 applies only when displacement gradients are much smaller than one. For finite deformation, Green–Lagrange strain is E=(FᵀF−I)/2.

## Stress and balance laws

Cauchy stress σ is defined in the current configuration, with traction t(n)=σn. In the absence of couple stresses and body couples, angular-momentum balance implies symmetric σ; micropolar materials require a generalized continuum. First and second Piola–Kirchhoff stresses are defined in the reference configuration; transformations must use consistent F, J, and push-forward/pull-back conventions.

Local mass balance is ∂ρ/∂t+∇·(ρv)=0; linear-momentum balance is ρ Dv/Dt=∇·σ+ρb, where b is body force per unit mass. Energy and entropy balances must include heat flux, internal energy, external supply, dissipation, and material exchange. Verifying force balance alone is not a complete physical analysis.

Boundary conditions include essential conditions such as displacement or velocity and natural conditions such as traction or flux. Specify initial fields, free surfaces, contact interfaces, periodic boundaries, and mixed conditions. Incompatible or over-constrained boundaries can yield no solution, non-uniqueness, or singular local stress.

## Constitutive laws and material limits

Balance equations do not specify material response; provide a constitutive law and state dependence on temperature, rate, history, anisotropy, and internal variables. Small-strain homogeneous isotropic linear elasticity may be written σ=2μ ε+λ tr(ε)I. This law does not apply to large deformation, plasticity, viscoelasticity, damage, cracking, or anisotropic materials.

Elastoplastic models separate recoverable and irreversible deformation and specify a yield function, flow rule, hardening variables, and consistency conditions. The von Mises criterion is common for approximating yield in ductile isotropic metals; it is not a universal yield law. Check plastic dissipation, heat generation, objectivity, and mesh dependence.

Finite-deformation constitutive laws must be objective under rigid rotation and satisfy material symmetry and thermodynamic dissipation constraints. Extrapolating small-strain Hooke's law to large strain produces incorrect stresses. Hyperelastic, viscoelastic, viscoplastic, and damage models require appropriate experimental calibration.

## Numerical modeling and validation

Before discretization, confirm geometry, mesh resolution, units, initial and boundary conditions, constitutive parameter provenance, and target outputs. Check mesh and timestep convergence, balance residuals, mass and energy budgets, mesh distortion, contact handling, locking, localization, and regularization sensitivity. Compare with analytic solutions, benchmark cases, material tests, or independent methods; a single mesh result is not validation.

Report model uncertainty, parameter uncertainty, discretization error, solver convergence, measurement uncertainty, and material-batch variation separately. If the solver does not converge, the mapping degenerates, boundaries conflict, or constitutive data are missing, return an indeterminate or limited result rather than a safety conclusion.

## Module boundary

Viscosity and turbulence closure are covered by FLUID_DYNAMICS.md; rigid-body and analytical mechanics by CLASSICAL_MECHANICS.md. Multiphase interfaces, electromagnetic coupling, and poroelasticity require explicit coupling of both sides' balances and interface conditions. The continuum hypothesis does not provide material constants, failure criteria, or finite-element execution capability.