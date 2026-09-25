# FIELD_PERCEPTION.md

## Observation and sensor interface

This module specifies how a host may describe physical observations. It does not provide sensors, field access, an embodied agent, an unknown-physics detector, or a perception model. A label such as “field” does not establish that a measurable quantity or instrument exists.

## Observation report

For each observation, record when available:

| Field | Meaning |
| --- | --- |
| `quantity` | Defined observable and units |
| `target` | System or region observed |
| `source` | Instrument or user/source reference, model, and version |
| `time` | Measurement time and clock basis, or unknown |
| `conditions` | Calibration, geometry, environment, and acquisition settings |
| `value` | Measured value or artifact reference; distinguish from inferred state |
| `uncertainty` | Measurement uncertainty and stated method |
| `limits` | Resolution, range, saturation, missingness, and known failure modes |
| `transforms` | Filtering, fusion, inference, and model steps applied |
| `capability_status` | Host-attested sensor/runtime status and scope |

Do not conflate sensor output, a model's inferred state, and a prediction. Sensor fusion requires compatible units, timing, calibration, and dependence assumptions. Independent evidence cannot be claimed from different sensor names alone.

## Unknown or anomalous observations

An unexplained residual may reflect noise, calibration error, missing variables, model mismatch, data corruption, or a new effect. Preserve the raw/reference data when permitted, document the comparison model, and seek independent measurement or review. Do not call it a newly discovered physical field without reproducible evidence and expert review.

Measurement disturbance and thermodynamic measurement costs are model-dependent. Do not apply Landauer's erasure bound as a universal cost of observation. See `AXIOMS.md` for scoped principle status.

## Capability boundary

Pseudocode in historical sections is illustrative and may not be executable. No sensor registry, world model, fusion system, user model, or free-energy controller is included in this repository. The host must attest any observation or inference capability used.
## Observation workflow

Treat the observation pipeline as a sequence of distinct artifacts:

1. **Acquisition:** preserve instrument output or a stable source reference, device identity where authorized, timestamp basis, configuration, and calibration state.
2. **Preprocessing:** document unit conversion, filtering, denoising, resampling, coordinate transforms, and discarded or imputed values.
3. **Estimation:** identify the model that maps measurements to a latent state; include parameter source, priors if used, residuals, uncertainty, and validity range.
4. **Fusion:** align clocks and frames, assess correlated errors and common data sources, and state the fusion rule.
5. **Interpretation:** compare the estimate with a declared physical model. Keep this conclusion separate from raw observation and model prediction.
6. **Review:** retain anomalies and competing explanations until resolved; state what independent observation or calibration would discriminate them.

A sensor may be biased, saturated, delayed, occluded, misconfigured, or correlated with another channel. A confidence score from a perception model is not automatically a measurement uncertainty. Use units and coordinate conventions consistently, and disclose when timestamp or calibration metadata is missing.

## Field and observable selection

Before using the word field, identify the observable, domain, units, spatial and temporal resolution, and measurement procedure. Examples include an electromagnetic field estimated from specified probe measurements, a gravitational field represented by a model over a region, or a temperature field reconstructed from sensors. These observables have different instruments and assumptions; a generic field interface does not make them interchangeable.

Quantum-state tomography, thermal sensing, acoustic measurements, and classical field measurements each have domain-specific measurement theory. Record a quantum state estimate as a model-dependent reconstruction from outcomes, not as direct access to an observer-independent hidden state. State preparation, measurement operators, sample count, and reconstruction assumptions matter.

## Anomaly triage

When a residual or unexpected signal appears, check in order: unit and coordinate errors; timestamps and synchronization; sensor health and calibration; preprocessing and missingness; model regime and boundary conditions; shared causes or confounding; replication with a different method; and only then whether a new model is warranted. Preserve negative results and failed replications when permitted. Discovery claims require a reproducible protocol and independent scrutiny; a detector label alone is not a discovery.
