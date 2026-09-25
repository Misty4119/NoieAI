# PHYSICS_KNOWLEDGE.md

## Physical claim and model records

This module describes records for physical claims, measurements, and model outputs. It is not an autonomous knowledge base, identity model, memory system, or physical-constant service.

## Record contract

Record a claim or model with:

- statement, quantity, units, domain, and scope;
- claim type/status from [Truth-OS epistemology axioms](../NoieTruthAGENTS/EPISTEMOLOGY_AXIOMS.md);
- source references and derivation, including edition/version and date checked;
- model assumptions, boundary conditions, parameter values, and validity range;
- measurement or estimation method and uncertainty where applicable;
- contradictions, unresolved questions, and known failure cases;
- capability/provider reference if a solver, sensor, or database produced the result.

Keep a stated physical constant, a measured value, a derived quantity, an effective model parameter, and a user-supplied assumption distinct. Do not attach a universal EC-L score or let a record's presence imply truth. Values requiring current metrology or standards should be checked against an authoritative current source.

## Update and contradiction handling

Preserve source versions and mark a record superseded when newer evidence or a corrected derivation changes it. Keep competing models and assumptions visible. A contradiction may indicate different definitions, regimes, measurements, or an error; do not label it a new physical effect before investigation.

## Capability boundary

No database, confidence-pruning algorithm, identity estimator, memory, or automatic update pipeline is included in this Markdown repository. Runtime storage and checks depend on the host.
## Claim classes and model record

A physical record should distinguish:

- **Constant or unit definition:** exact by definition within a measurement system, with its adopted convention and version.
- **Measured quantity:** value, uncertainty, method, instrument, calibration, and conditions.
- **Derived result:** inputs, equations, transformations, and propagated uncertainty.
- **Effective model parameter:** fit or approximation valid for a stated regime.
- **Theoretical prediction:** formal model and assumptions, plus empirical tests if any.
- **Interpretive claim:** relationship between formalism and ontology, explicitly marked as interpretation.
- **User or host premise:** a supplied assumption that has not been independently verified.

A record for a law or constitutive relation should include its variables and units, system boundary, dimensional checks, conditions of applicability, limiting cases, known counterexamples or breakdown regimes, and whether the claim is empirical, formal, or model-dependent. Dimensional consistency is necessary for many physical equations but is not sufficient to establish correctness.

## Constant and unit handling

Store the quantity name, symbol, value, unit, uncertainty or exact-definition status, source and edition, date checked, and conversion basis. Convert units explicitly and preserve significant digits supported by source precision. Do not silently substitute a current estimate for a defined constant or an outdated value for a revised measurement.

## Updating records

When an observation or accepted model changes, preserve the earlier record's source and scope, link a correction or supersession, identify affected conclusions, and re-evaluate dependent decisions. A change in model representation does not necessarily mean the physical law changed; record whether the update concerns evidence, parameter estimate, interpretation, or formal definition.
