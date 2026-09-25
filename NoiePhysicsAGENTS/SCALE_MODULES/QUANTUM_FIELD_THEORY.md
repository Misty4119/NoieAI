# Quantum Field Theory Reference Specification (Physics-OS v2.3)

**Domain:** Relativistic quantum fields, particle interactions, the Standard Model, and effective field theory methods. The Standard Model has been tested extensively but does not include quantum gravity or explain every observation. This document does not calculate amplitudes or query particle data.

## Formalism and applicability

Quantum field theory uses fields and their quantum excitations to describe relativistic many-particle systems. Calculations depend on a specified action, symmetries, field content, gauge fixing, boundary conditions, regularization, and renormalization scheme. Feynman diagrams are a visual bookkeeping device for terms in a perturbative expansion; they do not represent classical trajectories along which fundamental particles travel. Scattering amplitudes and observable cross sections also require external states, phase space, beams, detector, and background models.

The Standard Model gauge group is SU(3)_C×SU(2)_L×U(1)_Y and includes quarks, leptons, gauge fields, and the Higgs field. The electroweak Higgs mechanism provides the gauge-field and fermion mass structure, but not all particle mass originates from the Higgs vacuum expectation value; most nucleon mass is associated with QCD binding energy. The Standard Model does not include an empirically supported dark-matter particle or quantum gravity; neutrino masses require an extension beyond the minimal Standard Model.

## QED, QCD, and perturbation theory

QED describes the coupling between charged fields and the electromagnetic gauge field. Perturbation theory expands in the coupling constant; truncation order, infrared/ultraviolet treatment, renormalization scale, and scheme affect intermediate quantities. Observable predictions should include higher-order corrections and an assessment of theoretical uncertainty.

QCD describes quark–gluon interactions using a non-Abelian SU(3) gauge field. Asymptotic freedom at short distances makes perturbative calculations effective; the low-energy confinement regime is nonperturbative and is often handled with lattice QCD, effective theories, or experimental fits. Do not extrapolate perturbative formulas through the strong-coupling regime or treat quarks/gluons as freely detectable asymptotic states.

## Renormalization and effective field theory

Renormalization maps regularized parameters to a specified scale and scheme. Running couplings change under the renormalization group; this describes parameters across scales and does not mean that experimental observables vary arbitrarily at will. State the scheme and precision for renormalization-group equations, beta functions, and truncation order.

Effective field theories organize low-energy degrees of freedom by symmetry and encode higher-energy physics in controlled coefficients. State the cutoff or applicability scale, expansion parameter, retained order, leading omitted terms, and coefficient sources. Near the effective theory's cutoff, do not claim that predictions remain controlled by the same error estimate.

## Computation, data, and failure states

Check conventions, units, gauge invariance, conservation laws, renormalization-scale variation, and low-energy limits. Record software version, input cards, PDFs/beam functions or lattice settings, statistical errors, systematic errors, theoretical truncation errors, and experimental selection for each calculation. If perturbation theory fails to converge, infrared safety is untreated, nonperturbative input is missing, or the energy is outside the domain, report the result as indeterminate or qualified.

Particle properties and experimental reviews should use a versioned and dated Particle Data Group source or corresponding experimental data; this specification is not a source of current numerical values. For quantum-gravity candidates, see QUANTUM_GRAVITY.md; for the low-energy nonrelativistic limit, see QUANTUM_MECHANICS.md.