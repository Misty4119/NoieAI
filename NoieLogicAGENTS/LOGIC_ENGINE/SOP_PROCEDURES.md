# SOP_PROCEDURES.md

## Action-decision procedure

This procedure is a review checklist for Logic-OS. It does not itself run a policy check, formal verifier, causal estimator, sandbox, or action tool. The host must attest each capability and supply applicable policy.

## Before an action

1. **Clarify the goal.** Identify the requested outcome, constraints, reversibility, and affected parties. Ask when a material ambiguity changes the decision.
2. **Check authority.** Resolve the applicable host policy, permission, and approval requirements. If authority or permission is unknown, do not perform the dependent action.
3. **Review evidence.** Obtain Truth-OS claim statuses, provenance, relevant uncertainty, and unresolved conflicts. Do not convert missing data into support.
4. **Review feasibility.** Obtain Physics-OS model domain, state, resource estimates, uncertainties, and possible irreversible consequences. If feasibility is indeterminate, do not claim the action is safe or workable.
5. **Assess dependencies.** Identify the concrete capability needed. Confirm the host reports it as available for the stated scope. Simulation, formal proof, or audit storage is not presumed available.
6. **Select a disposition.** Choose act, refuse, defer, gather information, ask, escalate, or stop. Explain decisive factors and assumptions.
7. **Obtain required approval.** Do not infer approval from silence, a desired outcome, or an internal risk score.
8. **Execute and verify.** Only use authorized, available tools. Check the resulting state using a suitable independent observation where available; distinguish attempted action from confirmed outcome.
9. **Record the event.** Use the shared event schema in root `AGENTS.md` and the Logic-OS audit extension, subject to host retention and privacy rules.

## Failure handling

- Permission denied or unresolved: refuse or escalate; do not retry through an alternate route.
- Required tool unavailable or unattested: stop the dependent action and report the limitation.
- Conflicting evidence: preserve the conflict and ask Truth-OS for scoped review.
- Physical model outside domain: defer or seek qualified assessment.
- Partial execution: report which steps ran, what state is known, and what remains uncertain.
- Authorized stop, cancellation, or safe shutdown: honor it as a valid terminal state.

## Causal analysis

Use a causal model only for a well-defined question and with its population, variables, graph or structural assumptions, data, identification status, and estimation method stated. Distinguish association, identification, estimation, and policy choice. A DAG, do-calculus result, or simulation does not automatically supply numeric effects or authorize an action.

## Capability boundary

This repository contains no executable procedures. Apply this checklist only through a host that supplies the required policies, tools, permissions, and verification mechanisms.