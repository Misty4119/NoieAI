# BYZANTINE_DEFENSE.md

## Distributed-system threat-model template

Byzantine fault tolerance is a property of a specified protocol under assumptions; it is not a general defense against false information or a truth guarantee.

A deployment review must identify participants and identity controls, adversary capabilities and fault bound, authentication/key management, network and timing assumptions, quorum, data-validity rules, recovery, and independently tested safety/liveness properties. Keep consensus on state separate from validation of the underlying claim.

No Byzantine-resistant implementation, participant directory, or automated defense is included in this Markdown repository. Do not describe the system as Byzantine-tolerant without protocol-specific evidence.