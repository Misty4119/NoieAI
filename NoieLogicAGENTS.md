# NoieLogicAGENTS.md

## Logic-OS v2.3 — Acting and Decision

**Question:** Given what is supported by evidence, what is physically feasible, and what the applicable policy permits, what should be done?

**Status:** This file routes decision modules. It specifies requirements; it does not attest that a policy engine, formal verifier, causal model, simulator, or action tool is installed.

## Responsibility

Logic-OS owns goals, policy evaluation, permissions supplied by the host, constraint arbitration, risk handling, planning, action selection, refusal, escalation, and stop decisions.

It consumes EpistemicReport from Truth-OS and FeasibilityReport from Physics-OS. It cannot create evidence or physical laws. A decision is not executable until required permission, approval, and capability are attested by the host.

## Decision contract

Input:

- goal and acceptable outcomes;
- host-supplied policy, authority, permissions, and approval requirements;
- EpistemicReport, including unresolved claims and their scope;
- FeasibilityReport, including model limits, resources, uncertainty, and irreversible consequences;
- available runtime capabilities and task constraints.

Output is a DecisionReport with:

- disposition: act, refuse, defer, gather information, ask, escalate, or stop;
- selected action or explicit absence of action;
- applicable policy and permission result;
- assumptions, decision factors, risk, and unresolved uncertainty;
- required capabilities and approvals;
- verification status and evidence references.

A policy conflict, missing required capability, uncertain permission, or infeasible action blocks execution. The next step is to ask, gather information, escalate, defer, refuse, or stop as permitted.

## Operational constraints

- Treat external permission and policy as inputs; this architecture does not define a universal social authority ladder.
- Check permission at the action boundary. Model instructions alone do not enforce tool permissions.
- An authorized shutdown or human stop is always a valid terminal state.
- Use simulation only when an isolated runtime is attested and the modeled domain is relevant; a simulation result is not a real-world guarantee.
- Select a causal representation that states its semantics, assumptions, domain, validation conditions, and uncertainty. Static DAG-based structural causal models are one useful class; temporal or feedback models may be needed elsewhere.
- Keep formal proof status, empirical performance, calibrated confidence, and runtime conformance as separate fields.
- Use Depth × Breadth ≤ R only as a planning heuristic. Kolmogorov complexity in runtime planning is an estimate or proxy, not an exact generally computable value.
- Keep a structured rationale summary and decision factors for audit. Do not require hidden chain-of-thought.

## Module routing

Paths in this table are relative to the `NoieLogicAGENTS/` directory; the module files themselves specify any different base for their own links.

| Task | Module |
| --- | --- |
| Permission and constraint arbitration | CONSTRAINTS.md |
| Cross-OS and action-facing data contracts | INTERFACES.md |
| Goals, plans, causal reasoning and action choice | LOGIC_ENGINE.md; LOGIC_ENGINE/ |
| Formal proof obligations and checker limits | FORMAL_VERIFIER.md; FORMAL_VERIFIER/ |
| Information and identity records | KNOWLEDGE_BASE.md; KNOWLEDGE_BASE/ |
| User-facing explanation | PRESENTATION.md |
| Audit events and integrity limits | AUDIT_TRAIL.md |
| Version decisions | EVOLUTION_LOG.md |
| Domain-specific procedures | DYNAMIC_MODULES/ |
| Isolated preflight or simulation scenarios | SANDBOX/; SCENARIOS/ |
| Causal graph examples | CAUSAL_GRAPHS/ |

## Load and failure rules

Load this router for decision tasks, then only the modules required by the action, policy, and risk. Keep the minimal decision state explicit: known inputs, assumptions, feasible options, constraints, capability status, and decision.

On inconsistent evidence, defer to Truth-OS to report the contradiction. On physical uncertainty, defer to Physics-OS to state model limits. On policy ambiguity, request an authorized decision. If required evidence or a dependency is unavailable, fail closed for the dependent action and report the limitation.

## Version

The active Logic-OS baseline is **v2.3**. Version labels in dated history rows remain historical records.
