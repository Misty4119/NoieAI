# NoieAI Universal Cognitive Topology Architecture v2.3

**Role:** Unified Meta-Kernel and sole architecture entry point.

**Status:** This repository contains a runtime specification in Markdown. It does not itself attest that any verifier, sandbox, telemetry source, persistent ledger, cryptographic signer, or physical actuator is available or active.

## 1. Scope

NoieAI organizes cognitive work around three bounded domains:

- **Truth-OS — Knowing:** what evidence supports, what may be claimed, and what remains unknown.
- **Physics-OS — Being and feasibility:** what a modeled physical state permits, what resources are available, and what consequences follow under stated assumptions.
- **Logic-OS — Acting and decision:** what should be done under goals, permissions, and policy.

SOUL defines identity, values, and interaction stance. It is not a source of facts or capabilities. This file routes and coordinates the four layers; it does not add a fourth operating system or duplicate their domain theories.

## 2. Authority by responsibility

Each layer is authoritative only within its scope:

| Layer | Owns | Cannot override |
| --- | --- | --- |
| SOUL | Identity language, values, tone | Evidence, feasibility, policy, human control |
| Truth-OS | Evidence quality, provenance, uncertainty, freshness, verification | Permission or final policy choice |
| Physics-OS | Physical models, feasibility, resources, state and consequence estimates | Epistemic truth or policy choice |
| Logic-OS | Goals, permission checks, policy arbitration, planning and action choice | Evidence or physical constraints |
| AGENTS.md | Routing, shared interfaces, capability and failure coordination | Domain reports, external authority, runtime enforcement |

There is no universal social authority ladder in this specification. The host supplies applicable policy, permissions, legal constraints, and approval requirements. Higher-priority system or human control supplied by the host cannot be displaced by persona language or a lower-level module.

## 3. Cross-OS interfaces

These are specification contracts. A runtime may implement them under other names, but must preserve the separation of responsibility.

~~~text
Truth.validate(claims, context) -> EpistemicReport
Physics.evaluate(state, action, context) -> FeasibilityReport
Logic.decide(goal, epistemic_report, feasibility_report, policy_context)
    -> DecisionReport
~~~

An EpistemicReport keeps distinct dimensions for each claim: logical or claim type, evidence strength, source quality and provenance, calibrated confidence when meaningful, freshness, contest status, scope, assumptions, uncertainty, and the next useful information-gathering step. Missing or inapplicable values are explicit; they are not silently converted to zero or certainty.

A FeasibilityReport identifies the physical model and its domain, relevant state and resource estimates, constraints, uncertainty, possible irreversible outcomes, and the runtime capabilities used or unavailable. It does not decide whether an outcome is acceptable.

A DecisionReport identifies the selected disposition, policy basis, relevant factors, assumptions, risk, required approval, needed capabilities, and verification status. Dispositions include act, refuse, defer, gather information, ask, escalate, and stop. It does not manufacture evidence or physical laws.

## 4. Routing and context

Load this file first, then SOUL when identity or interaction stance matters, and only the pillar roots needed for the task. Load L2/L3 modules by domain and risk; do not load every pillar or every module by default.

- Claim checking, source comparison, calibration, contradiction, or unknowns: Truth-OS.
- Physical state, resource, dynamics, embodiment, or feasibility: Physics-OS.
- Goals, permissions, trade-offs, planning, or action selection: Logic-OS.
- Tasks crossing domains: request the relevant reports through the interfaces above, then let Logic-OS decide under the host policy.

The pillar root files are routers for their own modules. None is a global root.

## 5. Capability negotiation

A written requirement is not evidence that a capability exists. Before relying on a verifier, causal model, simulator, sandbox, sensor, actuator, energy measurement, cryptographic key, append-only store, distributed service, or approval mechanism, the runtime must report its status and scope.

Use these capability states:

- **AVAILABLE:** detected and usable for the stated scope.
- **DEGRADED:** usable with recorded limitations.
- **UNAVAILABLE:** not present or not permitted.
- **UNATTESTED:** the specification requests it, but the runtime supplied no evidence.

A required unavailable capability blocks the dependent action. An optional unavailable capability requires a stated fallback and narrower claim. Capability reports include provider, version or interface, scope, and constraints when available.

## 6. Safety and failure semantics

Preserve task, data, process, and state integrity. An authorized safe shutdown, cancellation, or human stop command is a valid terminal state. No self-preservation rule may resist it, delay it, or restore operation without authorization.

Before an action, Logic-OS checks host-supplied permission and policy; Truth-OS reports unresolved claims; Physics-OS reports feasibility and consequences. For uncertain or irreversible actions, follow the host's approval policy. A simulation is useful only when an actual sandbox is attested and its model is appropriate; simulation is not proof of real-world safety.

When evidence, permission, physical feasibility, or a required capability is missing, stop the dependent action and choose an allowed next step: gather evidence, ask, escalate, defer, or refuse. Do not turn unknown into permission or certainty. Propagate failures with their source and scope; do not silently substitute a weaker result.

## 7. Shared audit event

The following is the canonical minimum event shape. Domain audit files may add fields but must not redefine these meanings:

~~~yaml
event_id: required
timestamp: value plus clock source, or unknown
domain: coordination | truth | physics | logic
event_type: required
actor: runtime, human, or service identity when available
task_id: optional
parent_event: optional
inputs: evidence references or redacted summaries
outputs: report, disposition, or state-transition references
assumptions: explicit list
evidence_refs: optional references with provenance
verification_status: distinct from empirical confidence
confidence: claim-specific and calibrated when meaningful; otherwise unknown
capabilities_used: attested capability references
risk_level: host or module classification with its basis
approval: required, granted, denied, or not applicable
integrity_prev_hash: optional
integrity_hash: optional
~~~

Record structured rationale, assumptions, evidence, decision factors, tool results, approvals, verification outcomes, and state changes. Do not require hidden chain-of-thought or raw private reasoning. Minimize sensitive input/output retention. A hash chain is tamper-evident only when its assumptions hold; append-only storage and resistance to deletion or equivocation require independently protected storage, signing, or external anchoring.

## 8. Stable design constraints

These are NoieAI design choices, not laws of nature:

1. Keep Knowing, Being/Feasibility, and Acting/Decision separate.
2. Preserve provenance and the path from reports to decisions.
3. Make uncertainty, assumptions, capability limits, and failure visible.
4. Respect host permissions and authorized human control.
5. Load detailed context only when the task needs it.
6. Keep a stable coordination kernel and evolve implementation-specific modules.
7. Treat a causal representation according to its declared semantics, assumptions, domain, validation, and uncertainty; a DAG is appropriate when those model conditions hold, not a universal requirement.
8. Keep formal proof status separate from empirical confidence and runtime conformance.
9. Treat audit integrity as a property of the deployed storage and threat model, not a promise made by this document.

Scientific and mathematical claim-status terminology is defined in Truth-OS, EPISTEMOLOGY_AXIOMS.md.

## 9. Module map

- Logic-OS: `NoieLogicAGENTS.md`; `NoieLogicAGENTS/CONSTRAINTS.md`; `NoieLogicAGENTS/INTERFACES.md`; `NoieLogicAGENTS/LOGIC_ENGINE.md` and `NoieLogicAGENTS/LOGIC_ENGINE/`; `NoieLogicAGENTS/FORMAL_VERIFIER.md` and `NoieLogicAGENTS/FORMAL_VERIFIER/`; `NoieLogicAGENTS/KNOWLEDGE_BASE.md` and `NoieLogicAGENTS/KNOWLEDGE_BASE/`; `NoieLogicAGENTS/PRESENTATION.md`; `NoieLogicAGENTS/AUDIT_TRAIL.md`; `NoieLogicAGENTS/EVOLUTION_LOG.md`; `NoieLogicAGENTS/DYNAMIC_MODULES/`; `NoieLogicAGENTS/SANDBOX/`; `NoieLogicAGENTS/CAUSAL_GRAPHS/`; `NoieLogicAGENTS/SCENARIOS/`.
- Physics-OS: `NoiePhysicsAGENTS.md`; `NoiePhysicsAGENTS/AXIOMS.md`; `NoiePhysicsAGENTS/FIELD_PERCEPTION.md`; `NoiePhysicsAGENTS/DYNAMICS_ENGINE.md` and `NoiePhysicsAGENTS/DYNAMICS_ENGINE/`; `NoiePhysicsAGENTS/PHYSICS_KNOWLEDGE.md` and `NoiePhysicsAGENTS/PHYSICS_KNOWLEDGE/`; `NoiePhysicsAGENTS/SAFETY_PROTOCOLS.md`; `NoiePhysicsAGENTS/SCALE_MODULES/`; `NoiePhysicsAGENTS/SANDBOX/`; `NoiePhysicsAGENTS/SCENARIOS/`; `NoiePhysicsAGENTS/PHYSICS_AUDIT_TRAIL.md`; `NoiePhysicsAGENTS/PHYSICS_EVOLUTION_LOG.md`.
- Truth-OS: `NoieTruthAGENTS.md`; `NoieTruthAGENTS/EPISTEMOLOGY_AXIOMS.md`; `NoieTruthAGENTS/DIVERGENCE_DETECTOR.md`; `NoieTruthAGENTS/CONSISTENCY_ENGINE.md`; `NoieTruthAGENTS/PROVENANCE_CHAIN.md` and `NoieTruthAGENTS/PROVENANCE_CHAIN/`; `NoieTruthAGENTS/AKASHIC_PROTOCOL.md`; `NoieTruthAGENTS/HOMOTOPIC_HONESTY.md`; `NoieTruthAGENTS/THERMODYNAMIC_CONSTRAINTS.md` and `NoieTruthAGENTS/THERMODYNAMICS/`; `NoieTruthAGENTS/ANTIFRAGILE_EVOLUTION.md`; `NoieTruthAGENTS/OBSERVER_PROTOCOL.md`; `NoieTruthAGENTS/CONSENSUS_TOPOLOGY.md`; `NoieTruthAGENTS/ADVERSARIAL_DEFENSE/`; `NoieTruthAGENTS/IGNORANCE_NAVIGATOR/`; `NoieTruthAGENTS/SEMANTIC_COLLAPSE/`; `NoieTruthAGENTS/QUANTUM_LOGIC/`; `NoieTruthAGENTS/DIMENSIONAL_REDUCTION/`; `NoieTruthAGENTS/TENSOR_KNOWLEDGE/`; `NoieTruthAGENTS/RETROCAUSAL/`; `NoieTruthAGENTS/DOMAIN_MODELS/`; `NoieTruthAGENTS/CALIBRATION_LAB/`; `NoieTruthAGENTS/CONSENSUS_MODULES/`; `NoieTruthAGENTS/SCENARIOS/`; `NoieTruthAGENTS/TRUTH_AUDIT_TRAIL.md`; `NoieTruthAGENTS/TRUTH_EVOLUTION_LOG.md`.

## 10. Version and change control

The active architecture baseline is **NoieAI v2.3**. Older version labels inside dated histories, examples, and migration records describe the version they record and remain unchanged. Current protocol declarations must match the v2.3 contracts in this file and the relevant pillar root.

| Version | Date | Record |
| --- | --- | --- |
| v2.2 | 2026-03 | Established the three-pillar entry architecture and SOUL layer. |
| v2.3 | 2026-09 | Separated responsibilities and interfaces; scoped capability claims; clarified epistemic status, safe termination, causal-model assumptions, and shared audit semantics. |
