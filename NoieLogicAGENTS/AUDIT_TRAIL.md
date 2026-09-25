# AUDIT_TRAIL.md

## Logic-OS audit extension

Each event uses the shared event envelope in the root `AGENTS.md`. This file defines Logic-OS additions; it is a schema, not an active logging service, append-only store, or guarantee that every relevant event is captured.

### Logic fields

| Field | Meaning |
| --- | --- |
| `policy_reference` | Host policy, instruction, or permission basis considered |
| `decision` | `ACT`, `REFUSE`, `DEFER`, `GATHER_INFORMATION`, `ASK`, `ESCALATE`, or `STOP` |
| `approval` | Required, granted, denied, or not applicable, with authority reference where available |
| `assumptions` | Material assumptions used in the decision |
| `verification` | Formal-check status, property, tool, and scope; do not convert it to epistemic confidence |
| `causal_model` | Optional model reference and identification/estimation limitations |

Record only information needed for review. Do not require hidden chain-of-thought or sensitive data. Any retention, access control, signing, external anchoring, or append-only guarantee must be provided by the host and described with its threat model. A hash chain is tamper-evident only under its key, storage, and anchoring assumptions.

Preserve historical entries as historical records. Correct them by appending a linked correction where a real log implementation supports that operation. The Markdown file alone cannot enforce this policy.
## Event coverage and decision linkage

Create or reference an event at meaningful decision boundaries: request receipt; interpretation of goal and scope; evidence or capability check; policy/permission check; decision or refusal; external-action attempt; host response; verification of resulting state; cancellation, escalation, recovery, and correction. Record only events that actually occurred. A planned action and an executed action are different event types.

For a material action, link the request, decision, approval, tool call, tool result, and observed state transition with parent_event or domain references. Record whether approval was required and the authority basis, not merely a Boolean saying “approved.” A failed action must retain whether it failed before any external effect, after a partial effect, or with outcome unknown.

## Decision fields and privacy

The Logic extension adds the applicable policy reference, goal or objective reference, feasible alternatives considered at a high level, decision disposition, approval state, formal-verification reference, and any causal model result consumed. Do not copy an entire prompt, credential, personal record, or private reasoning trace when a restricted reference or redacted summary is sufficient.

A correction should identify the original event, the correcting actor or host process, the reason and new evidence, and which downstream decision may be affected. If the host cannot append linked corrections, disclose that limitation rather than promising an immutable audit history.

## Review queries

A reviewer should be able to follow: what was requested; which evidence, policy, and capability reports were used; who or what authorized the action; what actually happened; what was verified afterward; and what remains uncertain. Missing event coverage, clock skew, inaccessible references, and host-side edits must be visible as audit limitations.
