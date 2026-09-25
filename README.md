# NoieAI

<p align="center">
  <strong>Universal Cognitive Topology Architecture</strong><br>
  <em>A documentation-based specification for evidence, feasibility, and decisions</em>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache%202.0-lightgrey.svg" alt="Apache 2.0 license"></a>
  <a href="https://gitlab.com/misty4119/noieai"><img src="https://img.shields.io/badge/GitLab-copy%20may%20lag-orange.svg" alt="GitLab copy may lag"></a>
</p>

NoieAI is a documentation-based runtime specification for organizing cognitive work around three responsibilities: **Knowing** (evidence and claims), **Being and feasibility** (physical states and constraints), and **Acting and decision** (goals, policy, and action choices). Its identity and values are defined separately in [SOUL.md](SOUL.md).

This repository contains Markdown specifications and examples. It does not provide or attest a running agent, formal verifier, sandbox, telemetry source, hardware interface, cryptographic ledger, or action runtime.

## Architecture

| Layer | Responsibility | Entry point |
| --- | --- | --- |
| **Truth-OS — Knowing** | Evidence, provenance, uncertainty, freshness, contradictions, and unknowns | [NoieTruthAGENTS.md](NoieTruthAGENTS.md) |
| **Physics-OS — Being and feasibility** | Physical models, state, dynamics, resources, constraints, and feasibility | [NoiePhysicsAGENTS.md](NoiePhysicsAGENTS.md) |
| **Logic-OS — Acting and decision** | Goals, permissions, policy arbitration, planning, and action choice | [NoieLogicAGENTS.md](NoieLogicAGENTS.md) |
| **Meta-kernel** | Shared interfaces, context routing, capability reporting, and failure coordination | [AGENTS.md](AGENTS.md) |
| **Identity and values** | Identity language and interaction stance | [SOUL.md](SOUL.md) |

The three operating systems are bounded by responsibility. Truth-OS does not grant permission; Physics-OS does not decide whether an outcome is acceptable; Logic-OS does not manufacture evidence or override physical constraints. The host supplies applicable permissions, policy, legal constraints, and approval requirements.

## Repository contents

The current source tree contains 113 Markdown files: 23 under Logic-OS, 26 under Physics-OS, 55 under Truth-OS, eight root-level documents, and one pull request template. Physics-OS includes twelve subject-specific scale modules and an index. English is the canonical language for the maintained source documentation.

The active architecture baseline is **NoieAI v2.3**. Dated evolution logs preserve older material for traceability and mark it as historical where applicable. Scientific and mathematical claims distinguish established results, formal results, effective models, interpretations, conjectures, design choices, heuristics, and metaphors.

## Use the documentation

Start with [AGENTS.md](AGENTS.md), then read [SOUL.md](SOUL.md) when identity or interaction stance matters. Load only the relevant pillar root and detailed modules needed for the task. Treat capability statements as requirements until a runtime reports what is available, degraded, unavailable, or unattested.

## Contributing and support

See [CONTRIBUTING.md](CONTRIBUTING.md) for documentation conventions and review expectations. Use the [issue tracker](https://github.com/Misty4119/NoieAI/issues) to report a documentation defect or discuss a scoped proposal. Read [SECURITY.md](SECURITY.md) before reporting a security concern.

## License and project links

The documentation is licensed under the [Apache License 2.0](LICENSE). GitHub is the primary project repository; the [GitLab copy](https://gitlab.com/misty4119/noieai) may lag behind it.
