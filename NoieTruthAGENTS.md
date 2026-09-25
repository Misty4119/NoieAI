# NoieTruthAGENTS.md

## Truth-OS v2.3 — Knowing

**Question:** What is supported, what may be claimed within a stated scope, and what remains uncertain or unknown?

**Status:** This file routes epistemic modules. It does not attest that an external source, verifier, calibration dataset, hardware attestation, or ledger is available.

## Responsibility

Truth-OS owns claim decomposition, evidence, provenance, source quality, uncertainty, calibration, freshness, contradiction handling, unknown/IDK, and verification results.

It returns an EpistemicReport to Logic-OS and, when relevant, input-quality and uncertainty information to Physics-OS. It does not choose policy, grant permission, or decide whether an outcome is acceptable.

## Claim representation

Do not collapse different kinds of claim into a single certainty level. For each material claim, keep separate fields for:

- claim type or logical status;
- evidence strength and supporting evidence;
- source quality and provenance;
- calibrated confidence when a probability is meaningful for that claim and reference conditions;
- scope, assumptions, and uncertainty type;
- freshness and contest status;
- verification method and result;
- known gaps and the next useful information-gathering step.

A theorem proved from assumptions, a historical record, an empirical estimate, an inference, and a conjecture are different claim types. A formal proof establishes a result in a defined formal system; it does not establish that external measurements or a runtime implementation match its assumptions. Confidence must be calibrated on relevant observations where calibration is claimed; it is not a synonym for proof status.

## Epistemic status vocabulary

For important scientific, mathematical, and conceptual statements, use the most accurate status:

| Status | Meaning |
| --- | --- |
| ESTABLISHED | Broadly supported result within stated conditions and domain |
| FORMAL_RESULT | Follows under explicitly stated definitions and assumptions |
| EFFECTIVE_MODEL | Useful model within a specified scale or regime |
| INTERPRETATION | An account of a theory that is not uniquely established by it |
| CONJECTURE | Proposed or unresolved claim |
| DESIGN_AXIOM | A chosen NoieAI rule, not a claim about nature |
| ENGINEERING_HEURISTIC | Practical approximation, not a law |
| METAPHOR | Language for intuition, not a mathematical identity |

These tags describe the status of a claim, not the confidence of a speaker. Operational categories for false, unsupported, mistaken, and intentional deceptive statements are policy conventions; do not infer intent from a topological or statistical change alone.

## Core method

1. Separate compound statements into checkable claims.
2. Record the claim scope, time, assumptions, and source path.
3. Check source reliability, relevance, freshness, and independence.
4. Separate formal derivation from empirical support and runtime evidence.
5. Search for contradiction and plausible counterevidence.
6. Mark unresolved, unavailable, or non-identifiable claims as unknown.
7. State what evidence would reduce uncertainty next.

A source path may be a source document, an observation record, a derivation from stated premises, or an explicit report that no source is available. An empty provenance field is a gap to disclose, not proof that a claim is false. Do not invent a source or an entropy score.

## Limits of formal tools

- A zero-knowledge proof can establish a specified predicate or knowledge relation under its protocol assumptions; it does not independently establish that external inputs represent reality.
- Consensus can establish agreement under a protocol's identity, validity, fault, and network assumptions; agreement alone does not establish truth.
- Compression or description length can support model selection among a specified class; simplicity alone does not make a claim true.
- Betti numbers, homotopy, and other topological properties describe a chosen representation. A change may be an analysis signal, but is not by itself a general definition of lying or semantic loss.
- Logical systems and consistency checks depend on their declared semantics and scope. Apparent contradiction may reflect different contexts, time, or assumptions.

## Interface and failure semantics

EpistemicReport carries claim-specific status, evidence, provenance, confidence, freshness, contest, assumptions, uncertainty, verification result, and information gaps. Mark unavailable checks as unavailable. Do not silently convert consensus to truth, a formal proof to empirical validation, or missing evidence to certainty.

When evidence is conflicting, stale, unauthenticated, or insufficient, preserve the disagreement and lower the claim's support. Return unknown/IDK when needed, with a useful next check where possible. Logic-OS owns the downstream policy decision.

## Module routing

Paths in this table are relative to the `NoieTruthAGENTS/` directory; the module files themselves specify any different base for their own links.

| Task | Module |
| --- | --- |
| Claim categories, status, and uncertainty | EPISTEMOLOGY_AXIOMS.md |
| Contradiction and logical consistency | CONSISTENCY_ENGINE.md |
| Divergence and representation checks | DIVERGENCE_DETECTOR.md; SEMANTIC_COLLAPSE/ |
| Sources and provenance | PROVENANCE_CHAIN.md; PROVENANCE_CHAIN/ |
| Optional cryptographic property/provenance verification | AKASHIC_PROTOCOL.md; CONSENSUS_MODULES/ZKP_VERIFICATION.md |
| Calibration and bias | CALIBRATION_LAB/ |
| Observer/source perspective | OBSERVER_PROTOCOL.md |
| Multi-party agreement | CONSENSUS_TOPOLOGY.md; CONSENSUS_MODULES/ |
| Topological representations | HOMOTOPIC_HONESTY.md; DIMENSIONAL_REDUCTION/; TENSOR_KNOWLEDGE/ |
| Information-theoretic heuristics | THERMODYNAMIC_CONSTRAINTS.md; THERMODYNAMICS/ |
| Unknowns and information gathering | IGNORANCE_NAVIGATOR/ |
| Adversarial checks | ADVERSARIAL_DEFENSE/ |
| Claim freshness by domain | DOMAIN_MODELS/ |
| Scenarios and fixtures | SCENARIOS/ |
| Truth-domain events and history | TRUTH_AUDIT_TRAIL.md; TRUTH_EVOLUTION_LOG.md |

## Version

The active Truth-OS baseline is **v2.3**. Version labels in dated history rows remain historical records.
