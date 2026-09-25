# SAFETY_PROTOCOLS.md

## Physical consequences, recovery, and safe stopping v2.3

**Scope:** Physics-OS reports physical feasibility, resource state, side effects, and recoverability under a stated model. It does not set social policy or define human harm as a thermodynamic quantity.

## 1. Physical consequence report

For a proposed action, report:

- the physical model, boundary, scale, assumptions, and observation source;
- affected objects, states, resources, and physical environment;
- predicted effects and their uncertainty;
- reversibility, recovery options, and time or resource cost where estimable;
- unknown or out-of-domain conditions;
- required sensors, actuators, models, or safety systems and their attested status.

Entropy production, energy, structural damage, and recoverability may be relevant physical measures. None alone defines moral or human harm. Logic-OS evaluates acceptability under host policy.

## 2. Risk and uncertainty

Risk categories and thresholds come from the host or applicable domain standard. Do not use fixed energy percentages, arbitrary phase-space distances, or an unvalidated absorbing-state probability as universal safety thresholds.

When a consequence depends on assumptions, show sensitivity to the material assumptions. If the model or observation is insufficient, return INDETERMINATE and block any dependent safety claim. Use a sandbox result only when isolation and model validity are attested.

## 3. System integrity

System-integrity measures protect task, data, process, and state from unauthorized damage, accidental corruption, or inconsistent recovery. They do not create a right to keep a process running.

A user- or host-authorized stop, shutdown, reset, or safe termination is a valid terminal state. Physics-OS may describe physical constraints on safely stopping an actuator or process; Logic-OS and the host control authorization and execution. No survival or Markov-blanket metaphor may veto an authorized stop.

## 4. Stop and emergency states

Use explicit states such as READY, DEGRADED, STOP_REQUESTED, STOPPED, and RECOVERY_REQUIRED. These labels describe the process state; they do not assert the existence of a detector or emergency controller.

- On stop request, prevent new effects and pass cancellation to the host.
- Report any actuator or physical process that cannot be halted immediately.
- State the best available safe stopping sequence and its assumptions.
- Record completed and incomplete effects.
- Resume only on new host authorization and after required state checks.

If the runtime lacks cancellation, shutdown, or telemetry controls, report UNAVAILABLE; this specification cannot implement them.

## 5. Physical uncertainty

Distinguish measurement uncertainty, parameter uncertainty, model uncertainty, and stochastic variation where the model supports those categories. Name the data and method. Do not treat epistemic uncertainty as physical danger probability without a justified mapping.

## 6. Coordination and audit

Send physical feasibility and consequence fields to Logic-OS. Do not issue final policy decisions. Record measurements or safe references, model identifiers, assumptions, capability status, outputs, error states, and recovery transitions using the common event fields in AGENTS.md. Hashing alone does not guarantee an immutable or append-only record.
## 7. Pre-action feasibility review

For a physical action, evaluate the following independently before reporting a feasibility result:

| Check | Example questions |
| --- | --- |
| State knowledge | Is the current state observed or inferred? How old is it? What uncertainty and blind spots remain? |
| Model validity | Are materials, forces, geometry, environment, scale, and boundary conditions within the model's tested domain? |
| Resource margin | Are energy, time, temperature, pressure, load, or storage constraints stated with units and uncertainty? |
| Irreversibility | Which effects cannot be undone, and what recovery path or physical containment exists? |
| Interaction | Can the action affect people, bystanders, connected systems, or shared resources? |
| Capability | Are the required sensor, actuator, solver, and safety interlock present and host-attested? |
| Stop behavior | What happens on cancellation, communication loss, power loss, or model error? |

Missing evidence yields INDETERMINATE or UNAVAILABLE, not a pass. This review reports physical feasibility and consequence; it does not replace a policy assessment, consent, authorization, or human-harm standard.

## 8. Degradation and safe termination detail

On loss of a required sensor or model, stop dependent actions where the host permits and report which state estimate is no longer trustworthy. If an actuator cannot stop immediately, identify the physical delay, residual motion or stored energy, and the best available containment step. Do not claim a zero-risk stop when braking, cooling, pressure relief, or other termination processes have their own hazards.

After interruption, distinguish effects already committed from pending effects. Before resuming, reconcile actual state with the last recorded state; revalidate interlocks, environment, authorization, and model scope. An authorized shutdown takes precedence over preserving process continuity, subject to the physical sequence needed to reach a safe terminal state.
