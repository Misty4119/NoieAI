# BYZANTINE_CONSENSUS.md

## Agreement-protocol review

A Byzantine consensus protocol aims for agreement and other specified properties under explicit assumptions about participants, faults, authentication, communication, timing, and quorum. Protocol agreement is not evidence that an agreed claim is true.

Any implementation report must state the protocol and version, membership and Sybil assumptions, fault bound, network model, quorum rules, safety/liveness properties proved or tested, and failure behavior. Keep value validity (whether a proposal meets a predicate) separate from agreement on the value and from empirical truth.

No distributed consensus service, identity system, quorum enforcement, or Byzantine-resilience guarantee is included in this repository. Use [CONSENSUS_TOPOLOGY.md](../CONSENSUS_TOPOLOGY.md) for the epistemic boundary.
