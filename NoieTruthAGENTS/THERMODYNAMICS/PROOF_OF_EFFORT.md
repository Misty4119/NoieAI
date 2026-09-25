# PROOF_OF_EFFORT.md

## Computation records and evidential limits v2.3

A computation record can support reproducibility by preserving what operation ran and which artifact it produced. It is not a proof of correctness, independence, honesty, or sufficient effort. Resource expenditure and epistemic support are different observables.

## Reproducibility record

When useful, permitted, and supported by the host, record:

- task or operation identifier, start/end time and clock source;
- tool, software, model, version, and configuration;
- input and output artifact references, including transformations and redactions;
- parameters, random seed when applicable, dependencies, and environment;
- resource measurements with unit, meter, boundary, calibration, and uncertainty;
- checks run, exact results, failure status, and limits;
- artifact digest or signature with the key and storage assumptions.

Do not preserve a private chain-of-thought, hidden reasoning trace, or secret input as a substitute for a proof artifact. A digest identifies bytes only relative to a trusted comparison; it does not reveal whether the operation was useful or correct.

## What establishes a result

A mathematical claim needs a valid derivation in a declared formal system. A program result needs a reproduced run and suitable tests or validation. An empirical claim needs evidence appropriate to its population and method. A work log can link these artifacts but does not replace them.

Elapsed time, token count, number of tool calls, energy use, path length, or compressed trace length must not promote a claim to a higher epistemic status. No proof-of-effort verifier or resource meter is included in this repository.
