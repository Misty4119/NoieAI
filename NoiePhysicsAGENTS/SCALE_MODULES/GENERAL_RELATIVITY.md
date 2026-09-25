# General Relativity Reference (Physics-OS v2.3)

**Domain:** Classical spacetime geometry and gravitational field equations. General relativity is tested in many astronomical and weak-field regimes, but is not a completed quantum-gravity theory. This document is not a metric solver.

## Field equation and geodesics

With metric signature (−,+,+,+) and a stated curvature-tensor convention, Einstein's field equation is G_{μν}+Λg_{μν}=(8πG/c⁴)T_{μν}. If sign conventions change, adjust curvature definitions consistently. T_{μν} represents the chosen matter–energy content. The covariant-divergence condition implied by geometric identities requires compatible matter-field conservation. The field equations are nonlinear coupled partial differential equations; a solution needs a matter model, initial and boundary data, coordinate/gauge conditions, and well-posedness analysis.

A freely falling test particle follows a spacetime geodesic. Charged particles, pressure-supported matter, spin coupling, or bodies with non-negligible backreaction require additional force terms or full matter equations. Coordinate acceleration is not directly the local measured acceleration; state the observer, proper time, geodesic, and observable.

## Schwarzschild geometry and black holes

The Schwarzschild solution describes the exterior field of an ideal static, spherically symmetric, vacuum, uncharged, non-rotating body. Its event-horizon radius is r_s=2GM/c². A singularity at r=r_s in a particular coordinate chart can be a coordinate singularity; curvature invariants help distinguish it from the physical curvature singularity at r=0. Rotating, charged, accreting, or nonspherical bodies cannot be modeled directly by this solution.

An event horizon is a global causal boundary and cannot generally be located from a local field value alone. Black-hole thermodynamic area–entropy and Hawking-temperature relations combine classical geometry with semiclassical quantum field theory; Hawking radiation from astrophysical black holes has not been directly observed. Do not report a semiclassical formula as a measured astrophysical quantity.

## Gravitational waves and cosmology

For weak fields, far from a source, and small perturbations, the field equations can be linearized and a measurable dimensionless strain defined. Waveform predictions depend on the source, post-Newtonian or numerical-relativity approximation, detector response, and calibration. Strain is not itself a metric component near the source. Strong-field mergers require numerical relativity or empirically calibrated models.

The FLRW metric assumes large-scale homogeneity and isotropy. Friedmann equations also depend on matter content, curvature, Λ, and scale-factor conventions. Cosmological parameters are inferences from observational models; report datasets, likelihood, priors, systematic errors, and model selection. Do not interpret a cosmological scale factor as local expansion of material objects.

## Validation and applicability limits

Check symmetry assumptions, field-equation residuals, initial constraints, coordinate gauge, limiting cases, numerical convergence, and comparison with observables. The Newtonian weak-field limit, light deflection, Shapiro delay, binary-orbit dissipation, and gravitational-wave observations test different regimes; each depends on its measurement model and uncertainty. High curvature, singular interiors, Planck scales, and quantum effects require a theory beyond classical general relativity; do not extrapolate classical solutions speculatively.

Failure states include an unmodeled matter source, violated initial constraints, pathological coordinates, insufficient numerical resolution, inadequate approximation order, and conditions outside the theory's domain. This document includes no metric solver, ray tracer, gravitational-wave analysis, cosmological fit, or black-hole parameter estimator.