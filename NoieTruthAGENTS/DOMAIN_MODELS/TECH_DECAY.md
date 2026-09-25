# TECH_DECAY.md

## Technology-claim freshness review v2.3

Technical claims depend on version, platform, configuration, threat model, interoperability, and maintenance status. There is no universal technology decay rate.

Record product or standard, exact version, environment, configuration, source/documentation date, date checked, support status, tested conditions, dependencies, and relevant threat model. Prefer current primary documentation for API contracts and reproducible tests for behavior. A code sample or model-generated answer is not evidence that an API exists, a dependency is secure, or an architecture works.

Recheck when a dependency is upgraded, a platform or configuration changes, a vendor ends support, a relevant vulnerability is disclosed, a standard is revised, or the threat model changes. Separate “the documentation says this in version X” from “this deployment currently behaves this way.” Security and compatibility checks are scoped to the tested version and configuration.

Mark status UNKNOWN when no current verification was performed. No package scanner, update monitor, compatibility tester, or security auditor is included in this repository.
