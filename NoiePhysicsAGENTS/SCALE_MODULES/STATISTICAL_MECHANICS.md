# Statistical Mechanics Reference Specification (Physics-OS v2.3)

**Domain:** Connecting statistical distributions to macroscopic quantities under a specified microscopic model, ensemble, and equilibrium or nonequilibrium conditions. Equations depend on the probability distribution and system boundary. This document is not a molecular-dynamics or data-analysis program.

## Ensembles and partition functions

The microcanonical ensemble describes an isolated system with fixed energy, particle number, and volume; state weights depend on the number of accessible microstates. The canonical ensemble describes a system in equilibrium with a heat bath at temperature T, with energy exchange allowed and N,V fixed: β = 1/(k_B T), Z(β,V,N) = Σᵢ exp(−βEᵢ). The mean energy is U = −∂β ln Z, and constant-volume heat capacity and energy fluctuations follow from the corresponding derivatives; for a fixed Hamiltonian, Var(E) = ∂²β ln Z. If energy levels, boundaries, or external parameters themselves depend on β, include their derivative terms.

Canonical free energy is F = −β⁻¹ ln Z. Under applicable conditions, equilibrium pressure is p = β⁻¹∂V ln Z; hold the other natural variables fixed and keep the volume dependence in the definition of the partition function consistent. The grand canonical ensemble allows particle exchange: Ξ = Σ_N exp(βμN)Z_N. The mean particle number is given by β⁻¹∂μ ln Ξ, and the grand potential is Ω = −β⁻¹ ln Ξ. Before using an ensemble, check that the system is in equilibrium with the specified heat or particle reservoir.

Different ensembles are often equivalent in the thermodynamic limit for suitable short-range, additive systems. Finite systems, phase transitions, long-range interactions, constrained geometries, and nonergodic dynamics can retain important differences. Do not ignore model conditions merely because a system is “large.”

## Entropy, thermodynamics, and statistical distributions

Gibbs entropy S = −k_BΣᵢpᵢlnpᵢ depends on state probabilities and the coarse-graining of microstates. The Boltzmann form S = k_B ln Ω applies to a compatible count of equally probable accessible states. Entropy is a state quantity under a specified description, not a general truth score, subjective ignorance, or moral judgment.

Maxwell–Boltzmann, Fermi–Dirac, and Bose–Einstein distributions use different assumptions about particle distinguishability and occupancy restrictions; a dilute classical limit cannot be applied to a degenerate quantum gas. Chemical potential, degeneracy, internal degrees of freedom, and particle-conservation conditions must be consistent. Factorization of an ideal-gas partition function depends on independent-degree-of-freedom approximations and suitable quantum corrections.

The first, second, and third laws of thermodynamics each have their own system boundaries and operational formulations. A zero-temperature entropy statement under the third law must specify ground-state degeneracy and which formulation is used; do not conclude that every finite procedure can reach absolute zero. The Landauer erasure bound applies only under specified logically irreversible operations and reservoir conditions; see THERMODYNAMICS_PHYSICS.md for details.

## Phase transitions and critical phenomena

A thermodynamic phase transition usually involves nonanalytic free energy in the thermodynamic limit; a finite system has only smooth crossovers or finite-size features. The order parameter, symmetry, control parameter, critical exponents, and universality class depend on the model. Finite-size scaling can support inference, but report system size, boundaries, sampling, equilibration time, and uncertainty. Metastability, hysteresis, and finite observation time can make inference differ from equilibrium predictions.

## Fluctuations, response, and nonequilibrium

Fluctuations in an ensemble depend on its control parameters and observables. A fluctuation-dissipation relation requires equilibrium or specified linear-response conditions. A Langevin model of Brownian motion includes damping, random force, temperature, and noise correlations; the Einstein diffusion relation is limited to the corresponding equilibrium, long-time, and overdamped approximations. Nonequilibrium steady states, memory kernels, colored noise, and nonequilibrium reservoirs require other frameworks.

## Computation and uncertainty

Monte Carlo estimates should report sample count, autocorrelation or effective sample size, thermalization, sampling bias, finite-size effects, and statistical error; ordinary sampling may miss rare events. Molecular-dynamics work should state the force field, integrator, timestep, thermostat or pressure control, and finite-size effects. Numerical stability alone does not establish equilibrium, model correctness, or valid material predictions.

Report theoretical approximations, finite-sample, finite-size, parameter, measurement, and model errors separately. If mixing/equilibration or ensemble applicability has not been checked, mark the result unconfirmed. Energy derivatives can be derived using statistical physics, but any actual evaluation requires a known Hamiltonian and a verifiable computational environment.