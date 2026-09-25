# Quantum Mechanics Reference Specification (Physics-OS v2.3)

**Domain:** Nonrelativistic quantum states, observables, measurement statistics, entanglement, and open systems. Quantum mechanics has strong predictive success within its tested domain; interpretive questions must be separated from formal predictions. This document is not a quantum simulator or measurement system.

## States and evolution

A pure state of a closed system is a normalized vector |ψ⟩ in a complex Hilbert space; global phase is unobservable. A mixed state is represented by a positive semidefinite, trace-one density operator ρ. Observables are represented by self-adjoint operators on appropriate domains. Continuous spectra, unbounded operators, and boundary conditions require careful treatment of operator domains; do not extrapolate finite-matrix intuition without qualification.

A closed system evolves according to iℏ∂ₜ|ψ⟩ = H|ψ⟩. When the Hamiltonian is self-adjoint and the evolution operator is well-defined, evolution is unitary and preserves the norm. Measurement probabilities follow the Born rule; projective measurement is a particular idealization, not a complete description of every real instrument. State time-dependent Hamiltonians, degeneracies, continuous observations, and approximate basis choices explicitly.

## Measurement, mixed states, and uncertainty

A general quantum measurement can use POVM elements Eₘ to express probabilities p(m) = Tr(ρEₘ); a quantum instrument describes the post-measurement conditional state and state update. POVM probabilities alone do not determine the post-measurement state. Decoherence describes the decay of coherence terms in a reduced state after coupling to an environment; by itself, it does not resolve the measurement problem in every interpretation or authorize treating a particular interpretation as an experimentally verified mechanism.

For self-adjoint observables A and B with finite variance, the Robertson relation is ΔAΔB ≥ ½|⟨[A,B]⟩|. Energy–time relations are not a universal operator inequality of the same form as position–momentum uncertainty; specify whether the relation concerns Mandelstam–Tamm evolution time, lifetime–linewidth, or another definition. Any uncertainty claim must state the state, observables, statistics, and operational definition.

## Composite systems and entanglement

A composite system has a tensor-product state space; local density matrices are obtained by partial trace. Entanglement is nonseparability relative to a specified partition; its detection and quantification depend on dimension, pure or mixed states, and the chosen witness. Bell-inequality violations constrain models satisfying specified local hidden-variable assumptions; experimental inference must address assumptions such as setting independence, detector efficiency, and event selection. Entanglement correlations do not allow faster-than-light messaging and do not imply a traversable channel through space.

Concurrence is commonly used for particular classes of two-qubit states; it is not a universal entanglement measure for arbitrary dimensions or multipartite mixed states. Bell statistics or witnesses must use actual measurement settings, sample count, bias controls, and uncertainty intervals.

## Open systems and approximations

If the joint system–environment state supports Markovian dynamics under the chosen approximations, a Lindblad-type generator can describe it. State conditions such as weak coupling, Born–Markov, and rotating-wave approximations individually. Non-Markovian memory effects, initial system–environment correlations, strong coupling, and finite environments can invalidate simplified models. Decoherence times depend on degrees of freedom, coupling, and environment; no single universal exponential formula determines them.

## Numerical and experimental validation

Numerical approximations should report Hilbert-space truncation, basis, timestep, error control, state fidelity or observable convergence, and resource limits. Check density-matrix positivity and trace, closed-system unitarity, and energy or symmetry conditions. Experimental results require calibration, a readout model, background characterization, statistical error, and data-selection procedure. Without a specified state, Hamiltonian, measurement instrument, or numerical method, a result is indeterminate.

Quantum mechanics does not include a complete theory of quantum gravity; for relativistic quantum field theory, see QUANTUM_FIELD_THEORY.md. Keep quantum interpretations, observer ontology, and physical predictions separate; a philosophical position does not replace a testable calculation.