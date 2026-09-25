# CONSENSUS_TOPOLOGY.md

## Multi-party agreement and evidence independence v2.3

**Purpose:** Describe how participants reach an agreed protocol outcome and how evidence independence is assessed. Consensus is not truth verification.

## 1. State the protocol model

Any agreement claim identifies:

- participants and how identities are authenticated;
- message and network assumptions, timing, and membership;
- fault model and maximum faults tolerated;
- quorum, validity, safety, and liveness conditions;
- input-validation and external-oracle assumptions;
- timeout, membership-change, and recovery behavior.

Thresholds such as n > 3f apply only to particular consensus models and assumptions. They are not universal conditions for all distributed agreement. In fully asynchronous systems, termination guarantees depend on additional assumptions or trade-offs.

## 2. Result vocabulary

Use CONSENSUS_ACCEPTED, CONSENSUS_REJECTED, CONTESTED, or INDETERMINATE for protocol outcomes. Record the protocol, quorum, participant identities, faults assumed, and validity rule.

Do not use CONSENSUS_VERIFIED to imply that an empirical claim is true. Agreement establishes only the result guaranteed by the protocol and its inputs.

## 3. Evidence independence

Multiple participants provide stronger corroboration only when their evidence paths are sufficiently independent for the claim. Trace shared sources, copied outputs, common tools, and correlated failure modes. A count of model votes or nodes is not a substitute for independent evidence.

## 4. Truth verification

Truth-OS evaluates the claim, source quality, provenance, and uncertainty. It may use agreement as one evidence feature while preserving protocol limits. Logic-OS decides whether the result is sufficient for a policy or action.

## 5. Capability and audit

No distributed service is active by virtue of this specification. A runtime must attest the protocol, identity layer, membership, keys, fault assumptions, and stored result. Record the consensus outcome and supporting evidence using the common event fields; do not claim tamper-proof storage without independent protection.