# Plasma Physics Reference Specification (Physics-OS v2.3)

**Domain:** Charged many-particle systems, collective electromagnetic effects, plasma diagnostics, and magnetic-confinement fusion models. This document is not a diagnostic instrument, MHD solver, or fusion-performance predictor.

## Plasma conditions and scales

A plasma is ionized matter with sufficient collective behavior. Before using a plasma approximation, check characteristic scales against the Debye length, the number of particles in a Debye sphere, plasma frequency, collision frequency, and external-field timescales. The electron Debye length is λ_D = √(ε₀ k_B T_e/(n_e e²)), where T_e is in kelvins. If temperature is expressed in eV, convert k_B T_e in the formula to the same energy unit. A screening length for multiple species must be derived from their temperatures, densities, and distributions.

ω_pe = √(n_e e²/(ε₀m_e)) is the cold, homogeneous electron plasma frequency scale, not a resonance frequency for every wave or boundary. Measurements and models should state electron and ion distributions, magnetic field, collisions, ionization, wall conditions, and boundary sheaths. In dilute, strongly nonequilibrium, microscale, or non-Maxwellian conditions, simple fluid closure may not apply.

## Model hierarchy and MHD

Averaging from particle/kinetic models through two-fluid and single-fluid models to MHD discards information such as velocity distributions, Hall effects, particle orbits, Landau damping, and nonlocal transport. Choose a model by comparing relevant length, frequency, and collision scales; do not use MHD solely because the system is macroscopic.

Ideal MHD uses single-fluid conservation equations and an ideal induction equation, assuming very high conductivity and neglecting resistivity and some microscopic scales. Its divergence constraint ∇·B = 0 must be maintained; flux freezing applies only when model conditions hold. Resistive MHD, Hall MHD, two-fluid, and Vlasov/particle models handle different effects and are not merely interchangeable accuracy levels. Magnetic reconnection, turbulence, rapid changes, or weakly collisional plasmas can invalidate ideal MHD.

## Diagnostics and fusion

Langmuir probes, interferometry, spectroscopy, magnetic probes, and Thomson scattering have different spatial and temporal resolutions, invasiveness, inversion assumptions, and calibration requirements. A probe perturbs the sheath; temperature, density, and distributions are often inferred from an I–V curve or spectral model rather than read directly. Report raw signals, geometry, calibration, inversion method, background subtraction, uncertainty, and applicability range.

Magnetic-confinement fusion conditions depend on temperature, density, energy-confinement time, fuel composition, radiative losses, stability, and device geometry. Lawson-type criteria depend on the reaction and energy-balance definition; a single triple-product value cannot replace assessment of the whole device. Tokamak confinement time and stability vary with current profile, heating, boundaries, turbulence, and operating regime. Empirical scaling laws apply only within their data domains.

## Validation and risk boundary

Check charge, mass, momentum, and total-energy balances; report model hierarchy, closure, grid/particle count, collision operator, boundary sheath, initial distributions, and resolved scales. When comparing with diagnostic data, account for instrument response and measurement perturbations; disclose sparse samples, shared calibration bias, and non-unique inversion. If the model does not resolve relevant scales, violates the divergence constraint, or lacks a well-posed closure, report the result as indeterminate.

A feasibility conclusion about plasma or high-energy equipment does not establish equipment safety, net fusion-energy gain, or authorization to operate. This document provides no control, interlock, diagnostic, or energy-measurement capability.

## Module boundary

For fluid closure and CFD validation, see FLUID_DYNAMICS.md; for continuum-field balances, see CONTINUUM_MECHANICS.md; for quantum fields and microscopic reactions, see QUANTUM_MECHANICS.md and QUANTUM_FIELD_THEORY.md.