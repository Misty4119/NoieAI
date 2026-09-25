# INTERFACES.md

## Logic-OS action and report interfaces v2.3

**Scope:** This module defines the Logic-OS boundary to reports, policy providers, tools, and host runtimes. The canonical cross-OS reports and shared event fields are defined in AGENTS.md; do not duplicate or redefine them here.

## 1. Action request

A request submitted for decision should include:

~~~yaml
request_id: identifier
goal: desired outcome
action: proposed operation, if any
targets: affected subjects or resources
context: relevant task state and time
policy_context_ref: host-supplied policy and permission reference
epistemic_report_ref: optional Truth-OS report
feasibility_report_ref: optional Physics-OS report
required_capabilities: declared requirements
risk_basis: factors and uncertainty
~~~

A missing field is not implicitly satisfied. Use an explicit unknown or not-applicable value.

## 2. Decision response

Logic-OS returns a DecisionReport following AGENTS.md and specifies:

- disposition and the action or no-action result;
- policy and permission sources and their scopes;
- evidence and feasibility reports used;
- assumptions, constraints, risks, unresolved questions, and decision factors;
- capability and approval requirements;
- execution status, which remains pending until the host reports an outcome.

Do not use a numerical confidence field as a substitute for any of these items.

## 3. Host capability handshake

Before invoking tools, exchange the supported protocol version, capability names, versions, scopes, and constraints with the host. Each capability is AVAILABLE, DEGRADED, UNAVAILABLE, or UNATTESTED.

Tool annotations and documentation are untrusted descriptive metadata. They do not prove isolation, permissions, safety, idempotence, or reversibility. The host validates permissions and enforces approvals outside the model.

If a required capability or protocol version is missing, do not invoke the dependent operation. If an optional capability is missing, use a stated fallback and narrow the result.

## 4. Tool call lifecycle

Represent each tool action with:

~~~yaml
call_id: identifier
request_id: parent request
capability_ref: attested capability or unavailable
operation: read | compute | simulate | external_effect
permission_result: ALLOW | DENY | NEEDS_APPROVAL | UNKNOWN
approval_ref: optional
state: proposed | approved | running | completed | failed | cancelled
result_ref: optional redacted result or evidence reference
error: optional typed error
~~~

The host rechecks authorization immediately before an external effect. A model-produced ALLOW is not itself enforcement. On cancellation or stop, the host prevents new effects and cancels work according to its actual guarantees.

## 5. Errors and degradation

Use explicit outcomes: INVALID_INPUT, POLICY_CONFLICT, PERMISSION_UNKNOWN, APPROVAL_REQUIRED, CAPABILITY_UNAVAILABLE, CAPABILITY_DEGRADED, REPORT_STALE, INFEASIBLE, MODEL_OUT_OF_SCOPE, VERIFICATION_UNAVAILABLE, TOOL_FAILURE, CANCELLED, and STOPPED.

An error preserves its origin and scope. Do not replace a failed verification with success, a missing tool with a simulated success, or a cancelled operation with a retry. Retries require idempotence and policy authorization established by the host.

## 6. Compatibility

A module version indicates specification compatibility; it is not evidence that an implementation conforms. The active protocol baseline is NoieAI v2.3. Older protocol labels in dated examples are historical.