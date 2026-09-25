# TRANSMISSION_LOG.md

## Evidence-transfer record

This document specifies metadata for transferring a claim or artifact between systems. It does not provide a communication bus, live monitor, distributed trust network, or tamper-proof log.

Record sender and receiver identities as attested by the host, event time and clock source, artifact/claim identifier, content digest when appropriate, source references, format and schema versions, transformations, access controls, receipt status, and verification results. Keep transfer integrity separate from source authenticity and truth.

On failed or partial transfer, record the failure and do not treat the missing item as received. Verify digest, signature, schema, and access permission only when the actual implementation and keys are available. Hashes, signatures, and timestamps have limits tied to key custody, clock trust, storage, and anchoring. Apply privacy and retention policy to payloads and metadata.

No transmission or anomaly-query service is included in this repository.