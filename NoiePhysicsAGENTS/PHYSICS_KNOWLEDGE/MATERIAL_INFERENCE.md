# MATERIAL_INFERENCE.md

## L3 — Material identification as an inverse problem v2.3

**Scope:** Describe how measurements may constrain candidate material properties. This document is not an instrument, sample database, material classifier, or authorization for destructive testing.

## 1. Forward model before inversion

A measurement y is related to material parameters θ through a forward model conditioned on geometry, frequency, temperature, boundary conditions, instrument response, and noise. Record the model and nuisance parameters before attempting inversion. Multiple parameter combinations may produce nearly identical observations; report non-identifiability rather than selecting a unique material label.

Keep distinct: measured response, estimated property, candidate material class, and verified sample identity. A model fit or database match is not independent confirmation of composition.

## 2. Measurement families

| Method | Possible observables | Material and setup dependencies |
| --- | --- | --- |
| Acoustic or ultrasonic | Travel time, attenuation, reflection, resonance, impedance | Density, elastic response, anisotropy, porosity, coupling, path length, frequency, temperature |
| Electromagnetic | Reflection, transmission, absorption, phase, conductivity response | Frequency band, geometry, polarization, moisture, surface finish, temperature |
| Thermal | Temperature transient, conductivity, heat capacity, diffusivity | Contact resistance, boundary conditions, sample geometry, phase, moisture |
| Mechanical | Force-displacement, damping, recovery, fracture response | Load path, strain rate, geometry, damage history, fixtures, temperature |

A probe result applies to the tested location, configuration, and excitation. Non-destructive status must be established for the specific power, dose, duration, and sample; it cannot be inferred from the method name.

## 3. Example: acoustic impedance

For a suitable approximately linear, homogeneous medium and a specified wave mode, acoustic impedance is Z=ρc, where ρ is density and c is wave speed. Reflection at an ideal normal boundary depends on the impedances on each side. An impedance estimate alone does not uniquely identify density, wave speed, composition, or material class. Layering, anisotropy, attenuation, roughness, coupling, and geometry can confound the inference.

Report calibration, measured waveforms, path geometry, frequency, temperature, processing, uncertainty, competing material models, and sensitivity. Compare an inverse estimate against independent measurements when identity matters.

## 4. Active probing and decision boundary

Select a probe only after defining the property of interest, candidate hypotheses, predicted responses, discrimination value, measurement cost, sample risks, and required approval. Information gain can help compare experiments under a declared probability model; it does not authorize a probe or establish that its physical effects are acceptable.

If a measurement capability, safety boundary, or sample condition is unknown, do not claim the test is safe or the material is identified. Return candidate set, unresolved ambiguity, and next measurement that could distinguish candidates.

## 5. Capability and validation

A host must attest the actual instrument, sensor, calibration, data pipeline, model, software version, tested material domain, and validation. Report out-of-domain cases, failed calibration, saturation, poor signal-to-noise, and non-identifiability. This Markdown module supplies no runtime or material property service.
