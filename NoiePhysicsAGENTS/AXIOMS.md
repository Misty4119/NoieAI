# AXIOMS.md

## Physical principles and model status v2.3

**Role:** L2 index for well-scoped physical constraints and model status. It is not a list of universal NoieAI axioms. Detailed derivations remain in SCALE_MODULES/.

A physical statement is usable only with its domain, scale, assumptions, and evidence status. See [Truth-OS epistemology axioms](../NoieTruthAGENTS/EPISTEMOLOGY_AXIOMS.md) for the shared status vocabulary.

## 1. Durable constraints

| Topic | Status and scope |
| --- | --- |
| Conservation laws | ESTABLISHED within the applicable physical theory and boundary conditions; account for exchanges with the environment |
| Thermodynamic second law | ESTABLISHED statistical/thermodynamic constraint; state system, boundary, and process conditions |
| Landauer bound | ESTABLISHED for logically irreversible information erasure under specified thermal conditions; not a universal energy cost for every computation, observation, or acquired bit |
| Relativity and causal limits | ESTABLISHED within tested relativistic domains; use the model appropriate to the scale and conditions |
| Quantum mechanics and quantum field theory | ESTABLISHED predictive frameworks in their tested domains; interpretations of the formalism remain distinct from its predictions |
| General relativity | ESTABLISHED classical gravitational theory within its tested domain; it is not a completed quantum-gravity theory |
| Ryu–Takayanagi relation | FORMAL_RESULT/EFFECTIVE_MODEL within specified holographic settings; not a universal law that all spacetime is entanglement |
| ER=EPR | CONJECTURE about relations between entanglement and spacetime geometry |
| Relational quantum mechanics | INTERPRETATION of quantum theory |
| Constructor theory | Proposed foundational program; classify as CONJECTURE or INTERPRETATION, not established law |
| Free-energy principle and active inference | EFFECTIVE_MODEL frameworks under specified generative assumptions; not a general reduction to least action or Newtonian mechanics |
| Indefinite causal order | FORMAL_RESULT and experimentally explored within specified operational frameworks; not a general license for ordinary causal cycles |
| Substrate independence | FORMAL_RESULT for specified abstract computation mappings; claims about consciousness, identity, or equivalent experience remain conjectural |

The category labels characterize claims only. A label does not replace the assumptions or scope.

## 2. Physical model requirements

For a feasibility result, identify the model or governing equations, system boundary, initial state, scale, boundary conditions, parameters, observation source, and uncertainty. State whether a result is derived, simulated, measured, or assumed.

A model outside its validated regime returns indeterminate or degraded. Do not describe a design proposal, analogy, or simulation as a measured physical fact.

## 3. Information and thermodynamics

Information-theoretic measures can describe physical systems when a mapping and operational definition are stated. They do not by themselves determine truth, confidence, honesty, or human harm.

Landauer's bound concerns erasure of information under specified conditions. It does not imply that each bit learned costs kBT ln 2, that confidence has a fixed minimum energy cost, or that energy expenditure can verify a claim.

Entropy production and irreversibility may inform physical damage, recoverability, and resource loss. Ethical and human harm are assessed by host policy and Logic-OS, not defined by entropy.

## 4. Geometry, observers, and causal order

Riemannian or Lorentzian geometry, topological invariants, quantum measurement formalisms, and observer-relative interpretations apply only within their stated mathematical and physical models. Do not infer universal ontological claims from a model's useful representation.

A physical causal representation declares its semantics, assumptions, domain, validation conditions, and uncertainty. A DAG is one model form, not a universal invariant of physics.

## 5. Module routing

- Thermodynamics and statistical mechanics: SCALE_MODULES/THERMODYNAMICS_PHYSICS.md and SCALE_MODULES/STATISTICAL_MECHANICS.md.
- Classical and relativistic mechanics: relevant modules under SCALE_MODULES/.
- Quantum models: SCALE_MODULES/QUANTUM_MECHANICS.md and SCALE_MODULES/QUANTUM_FIELD_THEORY.md.
- Quantum gravity proposals: SCALE_MODULES/QUANTUM_GRAVITY.md.
- State and observation quality: FIELD_PERCEPTION.md and PHYSICS_KNOWLEDGE.md.
- Safety, reversibility, and physical consequences: SAFETY_PROTOCOLS.md.
- Runtime history and event fields: PHYSICS_AUDIT_TRAIL.md and AGENTS.md.

## 6. Runtime boundary

A formula or pseudocode in this repository does not supply a sensor, simulation, telemetry stream, or solver. Report a capability as active only when the host identifies it, its scope, version, and limits. Keep observations distinct from model outputs and user-supplied assumptions.
