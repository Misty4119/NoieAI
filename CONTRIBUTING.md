# Contributing to NoieAI

Thank you for helping improve the NoieAI specifications. This repository contains documentation and examples; it does not contain a deployed NoieAI runtime.

## Before opening a change

- Search existing issues and documentation for the topic.
- For a broad architecture change, open an issue first to agree on scope and affected interfaces.
- Keep each change focused and update the relevant pillar root, module map, or dated evolution log when the architecture requires it.
- Use English for maintained source documentation so the canonical repository stays consistent.

## Documentation standards

- Distinguish observed evidence, formal results, models, interpretations, conjectures, design choices, heuristics, and metaphors.
- State a model's domain, assumptions, inputs, boundaries, uncertainty, and failure conditions where they affect the claim.
- Keep Truth-OS evidence status, Physics-OS feasibility, and Logic-OS policy decisions separate.
- Do not imply that a written specification provides an executable verifier, simulator, sensor, persistent ledger, cryptographic signer, or actuator.
- Prefer primary sources for factual claims that need external support. Include enough context for a reader to assess their scope and date.
- Preserve older material as historical when it is retained for traceability; do not present it as the active contract.

## Pull requests

Describe the problem, the resulting documentation behavior, affected modules, and any material limitation. Include source references for new factual claims and summarize the checks you ran. Keep unrelated formatting changes out of the pull request.

Before submitting, check the diff for accidental files, unresolved local links, unmatched Markdown code fences, and whitespace errors. At minimum, run:

```text
git diff --check
```

There is no application build or runtime test suite in this documentation-only repository.

## Issues

Use the issue forms for documentation corrections and architecture proposals. Include the exact file and section, the current statement, the proposed correction, and supporting evidence or rationale. Do not post secrets, personal data, or security-sensitive details in a public issue.
