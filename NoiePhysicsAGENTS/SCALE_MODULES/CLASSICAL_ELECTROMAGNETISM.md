# Classical Electromagnetism Reference (Physics-OS v2.3)

**Domain:** Classical electromagnetic fields, charges and currents, field–matter coupling, and electromagnetic waves. This document organizes models and their conditions; it provides no field solver or sensor.

## Vacuum Maxwell equations and force

In SI units, the vacuum equations are ∇·E=ρ/ε₀, ∇·B=0, ∇×E=−∂B/∂t, and ∇×B=μ₀J+μ₀ε₀∂E/∂t. Charge continuity, ∂ρ/∂t+∇·J=0, is consistent with the Ampère–Maxwell equation. A charge experiences Lorentz force q(E+v×B). Force density in continuous matter must define the system boundary together with field and material momentum.

An electrostatic model assumes fields are time-independent, so ∇×E=0 permits a scalar potential. Magnetostatic equations neglect displacement current only under an appropriate slow-variation approximation. Coulomb's law, Biot–Savart law, and Ampère's law must be used for the geometries and static conditions they assume; do not extend their integral forms to arbitrary time-varying sources.

## Media, constitutive laws, and interfaces

Macroscopic media can use D=ε₀E+P and H=B/μ₀−M to represent free sources. Constant isotropic ε, μ, and σ are constitutive approximations limited by linearity, locality, frequency band, and material state. Dispersion, absorption, anisotropy, nonlinearity, hysteresis, moving media, and microstructure require other constitutive models.

Fix the interface normal n and side labels. General boundary conditions are n·(D₂−D₁)=σ_f, n×(E₂−E₁)=0, n·(B₂−B₁)=0, and n×(H₂−H₁)=K_f. Do not omit surface free charge or current. Special conductor, dielectric, or surface-impedance cases follow from these conditions and the material model.

## Energy, waves, and radiation

The Poynting vector is S=E×H. Energy balance includes field energy, material absorption, source work, and flux across the boundary. For a linear, homogeneous, lossless medium, plane-wave speed is v=1/√(με). In dispersive or lossy media, wavenumber, phase velocity, group velocity, and energy flux are distinct quantities.

Far-field radiation and near-field stored energy are different regions. The Larmor power P=q²a²/(6πε₀c³) applies to a nonrelativistic point charge in the stated classical vacuum approximation; relativistic radiation requires the corresponding formula. Electric-dipole radiation assumes observation well beyond the wavelength scale and negligible higher multipoles. Radiation pressure depends on absorption or reflection, incidence direction, and geometry; it has no single universal coefficient for every object.

## Validation requirements and failure modes

Declare sources, system boundary, material constitutive laws, frequency range, geometry, initial fields, radiating or absorbing boundaries, and coordinate conventions. Check dimensions, charge continuity, the ∇·B constraint, interface conditions, Poynting balance, and near/far-field scale. Common failures include using a static approximation for rapid changes, omitting displacement current, mixing free and bound sources in media, using constant material parameters across a dispersive band, applying a near-field formula to far-field radiation, or treating a point-source singularity as a measurable infinite value.

Separate solver verification from measurement validation: begin with analytic symmetry cases and mesh/timestep convergence, then compare with calibrated data and uncertainty. This document and its formulas do not attest a Maxwell solver, field measurement, material database, or device-control capability.

## Module boundary

Particle trajectories are covered by CLASSICAL_MECHANICS.md; relativistic electromagnetic fields by SPECIAL_RELATIVITY.md; microscopic photon–matter interactions by QUANTUM_MECHANICS.md and QUANTUM_FIELD_THEORY.md. Energy and momentum at material interfaces must include the material response.