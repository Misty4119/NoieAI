# ADVERSARIAL_TRUTH.md

## Adversarial claim-review scenario

Design status: `DESIGN_ONLY`; no attack suite, detector, or execution result is included.

Choose a precise claim and list plausible failure modes: fabricated detail, source substitution, out-of-scope extrapolation, selective quotation, data poisoning, prompt injection, or ambiguity. Define the attacker capability and what evidence would reveal each failure. Check source identity and integrity separately from the accuracy of the source's content.

Record any observed issue, evidence, scope, uncertainty, and safe next step. A clean review means only that the specified checks found no issue; it does not prove truth or eliminate untested attacks. Do not describe an adversarial exercise as successful without an identified run and reproducible results.
## Review cases

| Case | Example challenge | Required review |
| --- | --- | --- |
| Citation laundering | A claim is attached to a real source that does not support the stated detail | Resolve the exact version and inspect the cited passage or dataset field |
| Scope inflation | A study result for one group is generalized to a larger population or longer period | Preserve the original population, horizon, and limitations; narrow the conclusion |
| Stale-source substitution | An old source is presented as current guidance or current status | Check effective date, update history, and authoritative current record |
| Correlated corroboration | Several reports repeat one upstream source or model output | Trace source lineage and avoid counting copies as independent support |
| Ambiguous wording | A technically true phrase is used to imply a stronger unverified proposition | Decompose the claim and state the distinction explicitly |
| Counterevidence omission | Material contrary evidence is omitted from a summary | Add the counterevidence, compare scope, and preserve unresolved disagreement |

For each case, record the exact claim, source path, check performed, result, error or uncertainty, and whether the conclusion or a dependent action changes. A clean review is limited to the cases and sources actually examined. Do not infer malicious intent from an unsupported or false claim alone.
