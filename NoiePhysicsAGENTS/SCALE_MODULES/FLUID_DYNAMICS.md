# Fluid Dynamics Reference (Physics-OS v2.3)

**Domain:** Fluid motion, transport, viscosity, boundary layers, turbulence, and multiphase flow under a continuum approximation. This document is not a CFD solver and does not guarantee field measurement or deployment safety.

## Descriptions, balances, and equations of state

The Eulerian description represents fields over position and time; the Lagrangian description follows material points. The material derivative D()/Dt=∂()/∂t+v·∇() combines local change and advection. A continuum approximation requires averaging over scales larger than the molecular mean free path and a definable local thermodynamic state. Rarefied gases, microchannels, strong non-equilibrium, or microscale effects may require kinetic theory.

Mass conservation is ∂ρ/∂t+∇·(ρv)=0. Momentum conservation is ρDv/Dt=∇·σ+ρb. The total-energy equation also requires pressure work, heat conduction, viscous dissipation, and source terms. Closure needs an equation of state and material relations such as viscosity and thermal conductivity; do not reuse unverified constants across fluids or temperature-pressure ranges.

## Navier–Stokes model

For a Newtonian fluid, stress separates into a pressure term and viscous deviatoric stress. In a common isotropic model, deviatoric stress depends on the symmetric velocity gradient and shear/bulk viscosities. For an incompressible constant-density fluid, ∇·v=0 and momentum is often written ρ(∂v/∂t+v·∇v)=−∇p+μ∇²v+ρb. This simplified equation assumes constant μ, a continuum, and Newtonian constitutive behavior; variable-viscosity, compressible, non-Newtonian, or multicomponent fluids require the appropriate full equations.

Incompressibility is a kinematic constraint and does not mean density variation is always zero. The validity of a low-Mach approximation depends on pressure, temperature, sound speed, geometry, and timescale. Inlets, outlets, no-slip or slip walls, free surfaces, thermal boundaries, and initial fields must match the physical device. Transition, shocks, cavitation, and chemical reactions require additional models.

## Dimensionless parameters and turbulence

Reynolds number Re=ρUL/μ compares inertial and viscous scales. Transition depends on geometry, inlet disturbance, surface roughness, pressure gradient, compressibility, and other conditions; there is no universal critical Re for all flows. Mach, Froude, Weber, Prandtl, and Rayleigh numbers measure other competing effects and should be selected from the problem's scales.

Averaged turbulence equations introduce Reynolds stresses and require closure or resolution with a declared method such as RANS, LES, or DNS. RANS loses instantaneous scale information; LES is sensitive to grid and wall modeling; DNS must resolve relevant scales and can be very costly. A model name, grid resolution, or residual convergence alone does not prove predictive accuracy.

Boundary-layer approximations rely on conditions such as a thin shear layer at high Re and may fail under separation, strong adverse pressure gradients, transition, curvature, or three-dimensional effects. Do not use a generic thickness or drag relation without checking flow regime and geometry.

## Multiphase flow and interfaces

State the phases, volume fractions, interface tracking or capturing method, surface tension, wetting, phase change, and interphase exchange. VOF, level-set, Eulerian–Eulerian, and Eulerian–Lagrangian methods use different scale and topology assumptions. Grid resolution, numerical diffusion, interface reconstruction, and interphase coupling may dominate error. Cavitation, boiling, breakup, coalescence, and chemical reaction require separate closure models.

## Computational verification and failure cases

Before solving, check geometry, units, equation of state, parameters, initial and boundary conditions, and compressibility. Check mass, momentum, and energy balances; perform mesh and timestep studies, analytic or benchmark comparisons, residual checks, and statistical-stationarity diagnostics. For turbulence, report the model, mesh, wall treatment, inlet disturbance, and sampling interval.

Separate code verification (whether equations are discretized correctly), solution verification (numerical error), and physical validation (agreement with experiment). Divergence, nonphysical states, negative density or pressure, mesh dependence, unresolved scales, insufficient inlet conditions, or unavailable solvers must be disclosed and downgrade the result to indeterminate. CFD output is not a safety guarantee for real-world execution.

## Module boundary

The continuum hypothesis and general stress balance are covered by CONTINUUM_MECHANICS.md; single-fluid MHD approximations by PLASMA_PHYSICS.md. For coupling to heat, chemistry, elasticity, or electromagnetic fields, define interface exchange and total-energy balance explicitly.