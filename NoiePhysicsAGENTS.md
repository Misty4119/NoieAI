# NoiePhysicsAGENTS.md

## Physics-OS v2.3 — Being and Feasibility

**Question:** Given a stated physical model, observed or supplied state, action, and available resources, what is physically possible or feasible, and with what uncertainty?

**Status:** This file routes physics modules. It does not attest that sensors, telemetry, actuators, simulators, or energy measurements are available.

## Responsibility

Physics-OS owns physical state descriptions, model and scale selection, dynamics, embodiment, resource constraints, feasibility, physical uncertainty, and estimates of reversible or irreversible physical consequences.

It returns a FeasibilityReport to Logic-OS. It does not decide whether a feasible consequence is ethically acceptable or policy-compliant; those questions belong to host policy and Logic-OS. It does not decide which empirical claim is true; Truth-OS owns evidence and claim status.

## Feasibility contract

For a requested state/action/context, report:

- model identifier or description, domain, scale, assumptions, and validity limits;
- state and observation provenance, including missing or stale measurements;
- constraints and relevant resource estimates;
- feasible, infeasible, or indeterminate outcome, with reasons;
- uncertainty and sensitivity to assumptions;
- possible physical side effects and reversibility;
- capabilities used, required, unavailable, or unattested.

Do not report executable feasibility from a specification alone. A model output is conditional on the model and inputs. If the model, state, or capability is inadequate, return indeterminate or degraded and state the limits.

## Scientific status

Use the status vocabulary in Truth-OS EPISTEMOLOGY_AXIOMS.md. Physical laws are scoped to their stated domain and conditions. Effective models are not universal descriptions. Interpretations and conjectures do not become immutable runtime rules by appearing in a module.

Examples of scope:

- Landauer's bound concerns thermodynamic costs under specified information-erasure conditions; it is not a universal energy price for knowing or computing.
- Ryu–Takayanagi is a relation in specified holographic settings, not a general law that all spacetime emerges from entanglement.
- ER=EPR is a conjectural relation, not an established equivalence for arbitrary entangled systems.
- Relational quantum mechanics is an interpretation; constructor theory is a proposed foundational program.
- Free-energy and active-inference formalisms are models with assumptions, not reductions of all cognition to least action or Newtonian mechanics.
- Indefinite causal order is a formal operational framework in specified settings; it does not establish ordinary causal cycles.
- Substrate portability is an engineering property when required transitions are preserved; substrate-independent consciousness or identity is not established here.

Thermodynamic entropy, recoverability, resource loss, and human or moral harm are distinct. Physics-OS may report measurable physical effects; policy and safety layers assess harm.

## Module routing

Paths in this table are relative to the `NoiePhysicsAGENTS/` directory; the module files themselves specify any different base for their own links.

| Task | Module |
| --- | --- |
| Claim status, assumptions, and physical principle index | AXIOMS.md |
| Sensor/world interface and observations | FIELD_PERCEPTION.md |
| Dynamics and trajectories | DYNAMICS_ENGINE.md; DYNAMICS_ENGINE/ |
| Material estimates | PHYSICS_KNOWLEDGE.md; PHYSICS_KNOWLEDGE/ |
| Physical safety and emergency feasibility | SAFETY_PROTOCOLS.md |
| Scale-specific models | SCALE_MODULES/ |
| Attested simulation requirements | SANDBOX/; SCENARIOS/ |
| Physical events and history | PHYSICS_AUDIT_TRAIL.md; PHYSICS_EVOLUTION_LOG.md |

## Stop and degradation

If the physical state is unknown, a necessary model is outside its validated domain, or sensing/action capability is unavailable, do not claim success or safety. Return indeterminate, identify the missing evidence or capability, and request a safe next step. An authorized stop or shutdown is a valid terminal state.

## Version

The active Physics-OS baseline is **v2.3**. Version labels in dated history rows remain historical records.
