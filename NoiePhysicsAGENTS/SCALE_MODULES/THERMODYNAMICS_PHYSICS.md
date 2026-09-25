# Thermodynamics Reference Specification (Physics-OS v2.3)

**Domain:** Heat, work, energy, entropy, equilibrium states, and heat engines. Apply equations according to the system boundary, state variables, material relations, and process conditions. This document is not a measurement, control, or efficiency-estimation program.

## Systems, states, and sign conventions

First define the system, surroundings, control mass or control volume, boundary exchanges, and sign convention. Heat and work are energy transfers across a boundary, not state quantities stored inside a system. Here, heat entering the system is positive and work done by the system is positive; convert consistently when using another convention. A simple compressible equilibrium system of fixed composition satisfies dU = T dS − p dV. Open or multicomponent systems, and electromagnetic, surface, or elastic work, require additional conjugate terms.

Thermodynamic temperature is a state variable for equilibrium states. In the equilibrium model of a classical ideal gas, the equipartition theorem relates the mean energy of each independent quadratic degree of freedom to k_B T/2. Quantum, nonequilibrium, and non-quadratic degrees of freedom cannot be summarized as “temperature equals mean kinetic energy.” A statistical interpretation of entropy requires specified microstate probabilities, coarse-graining, and an accessible-state set.

## Laws of thermodynamics

**Zeroth law:** Transitivity of thermal equilibrium makes temperature usable as a state label; a practical temperature scale requires an operational definition and calibration.

**First law:** For a closed system, ΔE = Q − W, where E may include internal, kinetic, and potential energy. When bulk kinetic and potential energy are negligible and only PV work occurs, dU = δQ − p dV. A control volume must also account for energy carried by mass and work across its boundary. Heat and work are path differentials; internal energy is a state function.

**Second law:** The total entropy of an isolated system does not decrease. For a specified system, dS = δQ/T_boundary + dS_gen, with dS_gen ≥ 0; write the balance completely for multiple boundary reservoirs and mass/entropy flows. The Clausius inequality ∮δQ/T ≤ 0 applies to cycles with the corresponding definition of reservoir temperature. A reversible process has zero entropy production; this does not mean the system's entropy is fixed. The system's entropy can change when heat is exchanged.

**Third law:** The Nernst heat theorem and the unattainability of absolute zero have distinct precise formulations. For a suitable equilibrium perfect crystal with a nondegenerate ground state, entropy approaches zero as T approaches zero; ground-state degeneracy, glassy states, and other cases can leave residual entropy. The low-temperature power law of heat capacity depends on the system and is not universal.

## Thermodynamic potentials and equilibrium criteria

For a simple compressible system of fixed composition: dU = T dS − p dV; enthalpy H = U + pV, dH = T dS + V dp; Helmholtz free energy F = U − TS, dF = −S dT − p dV; Gibbs free energy G = H − TS, dG = −S dT + V dp. Multicomponent systems require Σᵢ μᵢdNᵢ, together with stated reaction and composition constraints.

Under the corresponding stable-equilibrium constraints, entropy is maximized for an isolated system at fixed volume and particle number; equilibrium at fixed temperature and volume is determined by minimizing F; equilibrium at fixed temperature and pressure, with composition and external constraints fixed, is determined by minimizing G. A metastable state, finite-rate process, nonequilibrium steady state, or externally driven system cannot have its full dynamics determined by a minimization principle alone.

An enthalpy change equals heat transferred only under suitable constant-pressure conditions, with only PV work and consistently defined end states. Zero internal-energy change in an isothermal ideal-gas process is a result of that specific model; the relation between internal energy and temperature for general materials depends on their equation of state and internal degrees of freedom.

## Thermodynamic processes and heat engines

At constant volume, PV boundary work is zero. An adiabatic process has Q = 0 but may produce entropy irreversibly. Isothermal does not mean constant internal energy. Reversible means zero entropy production, not an unchanged state. A reversible isentropic ideal-gas process satisfies pV^γ = constant only under a constant-heat-capacity-ratio approximation.

For a cyclic heat engine operating between two reservoirs with TH > TC, the ideal reversible upper bound is η_C = 1 − TC/TH, with temperatures on an absolute scale. For a reversible Carnot cycle using an ideal gas, heat on an isothermal branch is calculated from nRT ln(V₂/V₁), subject to the volume-ratio relation imposed by the adiabatic branches. Determine real efficiency from energy and entropy balances over the complete cycle and system boundary; there is no universal rule that obtains real efficiency by multiplying Carnot efficiency by a “mechanical efficiency.” Heat pumps and refrigerators are described by coefficients of performance, not heat-engine efficiency.

## Partition-function bridge and fluctuations

For a canonical ensemble at fixed N and V, let β = (k_B T)⁻¹ and Z = Σᵢ exp(−βEᵢ). When the Hamiltonian has no explicit β-dependence and its parameters are fixed:

$$F=-\beta^{-1}\ln Z,\qquad U=-\left(\frac{\partial \ln Z}{\partial\beta}\right)_{V,N},\qquad S=k_B(\ln Z+\beta U).$$

If volume enters the Hamiltonian in the standard mechanical way:

$$p=\beta^{-1}\left(\frac{\partial\ln Z}{\partial V}\right)_{\beta,N}.$$

The conditions held fixed in derivatives, volume-dependence of energy levels, and particle-number constraints must not be omitted. A single numerical value of a partition function is insufficient to determine internal energy or pressure; the functional dependence, model, and verifiable derivative calculation are required.

For a canonical ensemble with a fixed Hamiltonian, Var(E) = ∂²β ln Z = k_B T² C_V. In a stable single-phase system composed of N similar local contributions with finite correlation length, an extensive mean quantity is often O(N) and its variance O(N), so its relative variance is O(N⁻¹) and relative standard deviation is O(N⁻¹/²). Critical points, long-range correlations, finite size, and nonergodic states can differ. This is conditional thermodynamic scaling, not a universal “entropy decay law.”

## Information-thermodynamics boundary

The Landauer bound applies to logically irreversible erasure under a specified reservoir temperature. For an ideal unbiased bit in a degenerate memory, the standard lower bound is dissipated heat of k_B T ln 2 per bit. This does not mean that every computation, measurement, observation, acquisition of data, or formation of a belief must pay the same energy cost, and energy consumption cannot prove that knowledge is true. Work, heat, and total dissipation in a real operation depend on initial correlations, memory energy levels, protocol, finite-time effects, and system boundary.

## Calculation and validation requirements

Report the equation of state, composition, ensemble, equilibrium assumptions, reservoir and working substance, process path, heat/work sign convention, boundary exchanges, measurement calibration, and uncertainty. Check energy and entropy balances, limiting states, units, and variables held fixed in derivatives. For model-based heat, work, or efficiency calculations, list inputs and equations; explicitly report “not calculated” when no model, sensor, or numerical tool is available. Use empirical engineering correlations only within their calibrated domains.

## Module boundary

For statistical ensembles, partition functions, quantum statistics, and fluctuations, see STATISTICAL_MECHANICS.md. Thermodynamics supplies physical relations; it does not determine truth, confidence, ethical harm, or policy acceptability. Those judgments belong to Truth-OS and Logic-OS.