# KNOWLEDGE_BASE.md

## Logic-OS information record contract

This module describes information records used as inputs to decisions. It is not a persistent database, truth-ranking service, personal identity oracle, or append-only store. Truth-OS owns claim support and provenance; Logic-OS may reference those records but must not turn a storage label into an evidence judgment.

## Record fields

An implementation may store:

| Field | Meaning |
| --- | --- |
| `record_id` | Stable identifier within the host's namespace |
| `content` | Claim or artifact reference, with precise scope |
| `record_type` | Observation, source, derivation, preference, policy reference, or decision record |
| `provenance_ref` | Link to the source or derivation; missing provenance must be visible |
| `scope` | Time, domain, context, assumptions, and applicability |
| `status` | Review, freshness, and supersession state; never a universal truth score |
| `access_policy` | Host-defined permissions, retention, and handling constraints |
| `relations` | Explicit links such as supports, contradicts, derived-from, or supersedes |

## Handling rules

- Preserve original source material or a resolvable reference when policy permits; record transformations and versions.
- Keep empirical support, mathematical proof, confidence, freshness, and formal verification separate.
- Represent contradictions with links and scopes; do not average, delete, or silently overwrite them.
- Mark missing, stale, inaccessible, or unverified data explicitly.
- Treat preferences and identity descriptions as user-provided context, not external facts or permanent properties.
- Apply host privacy, access, retention, and deletion policies. No general append-only requirement overrides them.
- A record's presence in a knowledge base does not establish its truth or authorize its use.

## Decision use

Before using a record, check its provenance, status, scope, freshness, access permission, and relevance to the current decision. If an essential field is unavailable, disclose the gap and either gather information or narrow the decision.

## Capability boundary

No database, identity ledger, knowledge graph, causal graph store, or retention mechanism is included in this Markdown repository. Runtime storage and access guarantees must be supplied and attested by the host.
## Record lifecycle and retrieval

A host implementation may use the following lifecycle while preserving the distinctions above:

1. **Ingest:** assign a stable identifier; capture source, permission, scope, and transformations.
2. **Normalize:** extract claims without discarding the original artifact or changing modal, temporal, or population qualifiers.
3. **Link:** connect support, contradiction, derivation, correction, and supersession relations.
4. **Review:** attach evidence status, freshness, access restrictions, and unresolved conflict from Truth-OS.
5. **Retrieve:** filter by task scope, time, permissions, and relevance before semantic similarity ranking.
6. **Use:** pass the retrieved record with its provenance and limitations; do not pass a bare embedding or score as evidence.
7. **Correct or expire:** retain supersession and correction relations when policy allows; apply the host's retention and deletion rules.

Retrieval order is an efficiency choice, not an evidence hierarchy. If two records conflict, compare their exact propositions, units, populations, time windows, definitions, source independence, and update histories. Keep both until the discrepancy is resolved. User preferences, system policy, and empirical observations belong to distinct record types and access rules.

## Memory and scope controls

Long-term memory must not be inferred from this specification. A host that implements it should state what is persisted, for how long, who can read or modify it, how a user can correct it, and how deletion or legal hold is handled. Do not infer personal identity or stable preferences from one exchange. Do not copy sensitive data into an audit event when a permissioned reference is sufficient.

A decision-time retrieval report should list selected record IDs, excluded stale or inaccessible records when material, unresolved contradictions, and the scope mismatch that remains. If retrieval is incomplete or resource-limited, mark it rather than implying the knowledge base was exhaustively searched.
