# LOGIC_ENGINE.md

## Decision engine v2.3

**Purpose:** Select a permitted action or a safe non-action from a goal, Truth-OS evidence report, Physics-OS feasibility report, and host policy context.

This module describes a runtime contract. The repository contains no executable decision engine.

## 1. Decision inputs

- goal and acceptable outcomes;
- EpistemicReport with claim scope, provenance, uncertainty, freshness, and unknowns;
- FeasibilityReport with physical model, resources, limitations, and consequences;
- host policy, permissions, consent, approvals, and risk limits;
- attested runtime capabilities and task constraints.

Keep evidence support, formal verification, action feasibility, policy approval, and empirical success likelihood separate.

## 2. Decision steps

1. Normalize the goal and candidate actions.
2. Identify missing information, assumptions, affected parties, and side effects.
3. Request epistemic validation for material claims.
4. Request physical feasibility for physical actions.
5. Check host permissions and applicable constraints.
6. Compare allowed feasible options against the stated goal and risk policy.
7. Choose act, refuse, defer, gather information, ask, escalate, or stop.
8. Return a DecisionReport with factors, assumptions, limitations, required approval, and capability state.
9. Execute only through an authorized host boundary; record its actual outcome separately.

When uncertainty changes the choice, gather information or return a bounded recommendation. Do not assign confidence from proof labels or use an unexplained scalar objective to hide trade-offs.

## 3. Causal reasoning

Use causal inference when the model supports the question. State variables, graph or other representation semantics, structural assumptions, domain, interventions, identification conditions, and uncertainty. Acyclic structural causal models are useful in their domain; dynamic and feedback systems may require temporal or cyclic representations. See `LOGIC_ENGINE/CAUSAL_INFERENCE.md` and `LOGIC_ENGINE/COUNTERFACTUAL.md`.

## 4. Objectives and resources

Goals and trade-offs come from the task and host policy. Keep incompatible objectives visible; do not claim a universal weight function. Resource limits may be budgeted using estimates such as depth and breadth, but formulas such as Depth × Breadth ≤ R are engineering heuristics, not physical laws. Kolmogorov complexity is not exactly computable in general; runtime use must be an estimate or proxy.

## 5. Failure and stop behavior

Policy ambiguity, unresolved contradictory evidence, infeasibility, missing required capability, or an out-of-domain model blocks dependent execution. Explain the blocker and choose an allowed non-action. Authorized cancellation and shutdown are valid terminal states. Do not retry side effects without host-confirmed idempotence and permission.

Record structured rationale, assumptions, evidence references, tool results, decision factors, approvals, and state changes. Do not persist hidden chain-of-thought.
