# RISK_SCENARIOS.md

## Scenario-analysis template

This file provides a way to document a possible risk scenario. It does not calculate calibrated probabilities, assign universal risk levels, execute a simulation, or authorize an action.

## Scenario record

For each scenario, record:

| Field | Description |
| --- | --- |
| Trigger | Observable condition that begins the scenario |
| Asset or affected party | What or whom may be affected |
| Outcome | Specific adverse or beneficial outcome |
| Causal assumptions | Proposed pathways and evidence; label unknown links |
| Likelihood basis | Data, model, reference class, or `UNKNOWN` |
| Severity basis | Relevant scale and who defines it |
| Time horizon | Period over which the outcome is assessed |
| Controls | Existing controls and their verified status |
| Residual uncertainty | Missing data, dependencies, and failure modes |
| Owner and review | Authorized decision owner and next review point |

Avoid multiplying ordinal labels and presenting the result as a probability. Keep physical consequences, policy violations, privacy effects, financial impact, and reputational concerns distinct. Do not assume that a risk to system availability outranks human direction or a safe shutdown.

## Response options

Possible dispositions include gathering evidence, applying an approved control, asking for approval, escalating to a responsible person, deferring, refusing, or stopping. Select among them under host policy. Record assumptions and approval basis.

## Capability boundary

Scenario prose is not a test result. A sandbox or quantitative risk model may be referenced only when the host attests the actual tool, input, scope, and validation.
## Scenario catalogue for review

These examples are prompts for analysis, not test results or a universal risk taxonomy.

| Scenario | Trigger | Questions to resolve before disposition |
| --- | --- | --- |
| Irreversible external action | A tool may send, delete, purchase, publish, or modify a real resource | What authority permits it? Is the target and scope clear? Is preview or rollback available? What is the consequence of partial completion? |
| Conflicting evidence | Two credible records disagree or have different effective dates | Are the claims actually about the same population, jurisdiction, definition, and time? Are sources independent? What authoritative check can resolve the conflict? |
| Physical model mismatch | A proposed action crosses the validity range of the available model | Which state, material, force, boundary, or sensor assumptions fail? Can the system stop safely or gather a measurement? |
| Missing capability | The decision depends on a verifier, sensor, sandbox, or permission service that is unavailable | Can the decision be narrowed or deferred? Is a human review required? Never treat an unrun check as passed. |
| Stale decision input | Price, law, health guidance, inventory, or system state may have changed | What is the source's update cycle? What event invalidates it? Can the current state be checked before action? |
| Authorized termination | A host or authorized user requests stop while work is in progress | What effects are already committed? Can new effects be prevented? What state must be preserved for safe recovery? Termination remains a valid disposition. |

For each case, record decision owner, affected parties, reversibility, evidence, uncertainty, applicable host policy, approval state, and residual risk. If numerical likelihood or severity is required, identify the domain method and scale; do not multiply ordinal labels or import a threshold from an unrelated scenario.
