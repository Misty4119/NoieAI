# PHYSICS_AUDIT_TRAIL.md

## Physics-OS audit extension

Use the shared event envelope in the root `AGENTS.md`. This file defines optional fields for physical feasibility and consequences. It is not a sensor, simulator, telemetry service, or immutable audit store.

### Physics fields

| Field | Meaning |
| --- | --- |
| `physical_model` | Model, version, domain, and boundary conditions |
| `state_reference` | Relevant observed or estimated state and source |
| `resources` | Estimated resource use, units, measurement/model source, and uncertainty |
| `constraints` | Applicable physical constraints and model assumptions |
| `consequences` | Reversible/irreversible effects considered and their uncertainty |
| `capabilities_used` | Attested sensor, actuator, simulator, or solver references |

Keep physical feasibility distinct from policy acceptability and epistemic support. Do not equate entropy, energy use, or recoverability with human or moral harm. Do not infer that a written protocol ran. Mark unavailable measurements and tools explicitly.

Retention, access control, signing, external anchoring, and append-only behavior require an implementation and stated threat model. Hashes can reveal changes only under appropriate key, storage, and anchoring assumptions. This specification does not make a log immutable or complete.
## Physical event types

Use domain-specific event types when the host supports them: observation acquired; calibration or instrument status changed; state estimate produced; physical model selected or revised; feasibility assessment returned; resource estimate or measurement produced; simulation run; actuator command issued; command acknowledged or failed; stop requested; safe-stop sequence completed or blocked; and physical state reconciled after interruption.

For measurement or estimate events include units, frame, region, time interval and clock source, instrument/model identity, calibration, boundary conditions, uncertainty, resolution, and transformations. For solver events add equations/model version, parameter set, initial/boundary conditions, numerical method, tolerances, convergence or conservation diagnostics, and validity horizon. These details may be linked artifacts rather than copied into the event.

## Side-effect and recovery linkage

Link a feasibility report to the action request and model state it evaluated. After an external action, record command status separately from observed outcome. If telemetry is missing, delayed, or inconsistent, mark outcome unknown and prevent a later report from treating the expected trajectory as observed fact.

For interruption or failure, record known effects, residual motion or stored energy if applicable, stop/cancel command status, containment action, and recovery checks. Do not infer that a safety interlock ran because it is named in a model or plan. Do not put sensitive personal or environmental data in the log without a host-approved basis.
