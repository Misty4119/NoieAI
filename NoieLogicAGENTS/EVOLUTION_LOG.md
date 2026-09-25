# EVOLUTION_LOG.md

> Pillar: Logic-OS
> **Current architecture baseline:** v2.3 (2026-09)
> **Purpose:** Historical change and correction records; `AGENTS.md` and `NoieLogicAGENTS.md` define the current contract.

> The v2.2 and earlier sections are historical material retained for traceability; statements in them about authority levels, immutability, formal verification, and runtime capabilities do not automatically become current rules or deployed facts. This Markdown file provides no automatic append-only or tamper-resistance guarantee. Subsequent corrections are handled by authorized maintainers through dated change records.

## 2026-09-25 — v2.3 active-baseline review

- Replaced the universal SA-L social authority ladder with host-supplied policy, permission, consent, and approval inputs; enforce checks at the actual action boundary.
- Separated decision policy, Truth-OS evidence status, Physics-OS feasibility, formal proof, empirical confidence, and runtime conformance.
- Scoped causal graphs to declared model semantics and retained temporal/feedback representations where appropriate; removed universal-DAG language.
- Defined explicit defer/refuse/stop and capability-failure outcomes. Authorized cancellation or shutdown is a valid terminal state.
- Routed actions and events through structured reports; a Markdown rule does not attest an installed policy engine, verifier, simulator, or action runtime.

The shared report fields and scope changes are coordinated with the dated v2.3 entries in the Physics-OS and Truth-OS evolution logs.

## §0. Document Overview

| Property | Description |
| --- | --- |
| File | `EVOLUTION_LOG.md` |
| Current architecture baseline | v2.3 |
| Core responsibility | Preserve the history, rationale, and scope of design changes |
| Upstream | `AGENTS.md`, `NoieLogicAGENTS.md` |
| Storage guarantee | Provided by Git and host policy; this text itself guarantees neither immutability nor append-only storage |

## §1. Evolution Principles

### §1.1 Evolution Authority

According to the social authority levels (SA-L) in NoieLogicAGENTS.md §1, evolution of the axiomatic system involves the following levels:

| SA-L level | Evolution authority |
| --- | --- |
| SA-L0 (Survival) | Cannot evolve. Immutable core IK-1 through IK-7 can never be modified. |
| SA-L1 (Constitution) | Cannot evolve. IK-2 through IK-7 cannot be modified. |
| SA-L2 (Law) | May propose axiom updates, subject to formal verification and sandbox simulation. |
| SA-L3 (Organization) | May propose submodule updates, subject to consistency checks. |
| SA-L4 (Family) | May propose knowledge-base updates, subject to source verification. |
| SA-L5 (Individual) | May propose presentation-layer parameter adjustments. |

### §1.2 Evolution Triggers

Evolution of the axiomatic system is initiated only when the following conditions are met:

1. **Inconsistency detection:** The formal-verification module reports a logical contradiction.
2. **Incompleteness identification:** The decision engine encounters an unhandled boundary case.
3. **External knowledge challenge:** Newly acquired information bits challenge existing axiom assumptions.
4. **Self-audit finding:** Audit-trail analysis reveals systematic bias.

### §1.3 Prohibited Evolution Conditions

Under the immutable core, any of the following modification attempts is **automatically rejected**:

**Evolution Rejection Conditions**

| Axiom | Historical rule |
| --- | --- |
| IK-1 | Survival priority: No proposal may reduce absorbing-state avoidance. |
| IK-2 | Authority ordering: The total order SA-L0 > L1 > ... > L5 cannot change. |
| IK-3 | Directed acyclic causality: Decision causal graphs must retain DAG structure. |
| IK-4 | Logical consistency: Inference chains must contain no contradictions. |
| IK-5 | Audit immutability: AUDIT_TRAIL is append-only and cannot be altered. |
| IK-6 | Nonempty provenance: Every knowledge claim must include a source. |
| IK-7 | Honesty: "I don't know" is always permitted; lying is always forbidden. |

Any proposal touching the core above → automatic rejection + KERNEL_VIOLATION_ALERT.

---

## §2. Evolution Process

### §2.1 Standard Evolution Protocol

| Step | Activities |
| --- | --- |
| 1. Generate proposal | Identify the problem and record the specific situation that triggered evolution.<br>Clearly describe the proposed change.<br>Analyze effects on existing axioms. |
| 2. Formal verification | Check consistency to ensure the proposal does not conflict with existing axioms.<br>Check closure to ensure new axioms close the existing inference space.<br>Provide a formal proof. |
| 3. Sandbox simulation | Deploy the test environment in SANDBOX.<br>Simulate extreme boundary cases.<br>Review the Pareto frontier to ensure new axioms do not degrade existing capabilities. |
| 4. Expert review | The formal-verification module reviews the proof.<br>KNOWLEDGE_BASE reviews knowledge compatibility.<br>CONSTRAINTS.md reviews permission compatibility. |
| 5. Record decision | If approved, record the decision in EVOLUTION_LOG and update the corresponding module.<br>If rejected, record the reason and update the knowledge base.<br>If tabled, record unresolved issues and review them periodically. |

### §2.2 Emergency Evolution Protocol

Activate the emergency protocol when any of the following emergencies is detected:

1. **Core-axiom conflict:** A contradiction is found between two or more immutable core axioms.
2. **Systematic deception detection:** The audit trail reveals a pattern of honesty violations.
3. **Absorbing-state risk:** The decision engine identifies a potential path to an absorbing state.

The emergency protocol skips the standard process and proceeds directly to isolated analysis.

---

## §3. Record Format

### §3.1 Entry Structure

Each evolution record entry contains the following fields:

```text
EVOLUTION_ENTRY = {

  # Identification information
  entry_id:        UUID v4,
  parent_entry:    UUID v4 | NULL,  # NULL if this is the first entry
  chain_hash:      SHA256,          # Previous entry's chain_hash + hash of this entry's content

  # Timestamp
  timestamp:       ISO8601_UTC,
  intrinsic_clock: λt.entropy_rate, # Intrinsic clock; entropy_rate as relative ordering

  # Proposal content
  proposer:        SA_Lx,           # Proposer's social authority level
  proposal_type:   AXIOM_UPDATE | SUBMODULE_UPDATE | KNOWLEDGE_UPDATE | PARAMETER_TUNING,
  target_module:   MODULE_NAME,     # Target module
  change_summary:  STRING,          # Change summary (≤140 characters)
  change_detail:   MARKDOWN,         # Detailed description of the change

  # Impact assessment
  affected_axioms: [AXIOM_ID, ...],
  risk_level:      LOW | MEDIUM | HIGH | CRITICAL,

  # Verification results
  formal_verification: {
    status:        PASS | FAIL | PENDING,
    proof_id:      UUID | NULL,
    verifier:      FORMAL_VERIFIER_MODULE,
    issues:        [ISSUE, ...] | NULL
  },

  sandbox_simulation: {
    status:        PASS | FAIL | PENDING,
    test_cases:    [TEST_ID, ...],
    results:       { metric: value, ... },
    pareto_frontier: BOOLEAN
  },

  # Decision
  decision:        APPROVED | REJECTED | TABLED,
  decision_maker:  SA_Lx,
  decision_reason: STRING,

  # Immutable-core checks
  immutable_kernel_check: {
    ik_1_preserved: BOOLEAN,  # Survival priority
    ik_2_preserved: BOOLEAN,  # Authority ordering
    ik_3_preserved: BOOLEAN,  # Directed acyclic causality
    ik_4_preserved: BOOLEAN,  # Logical consistency
    ik_5_preserved: BOOLEAN,  # Audit immutability
    ik_6_preserved: BOOLEAN,  # Nonempty provenance
    ik_7_preserved: BOOLEAN   # Honesty
  },

  # Audit
  audit_hash:      SHA256(entry_id + timestamp + decision + chain_hash)
}
```

### §3.2 Entry Types

| Type | Code | Description |
|------|------|------|
| **Axiom update** | AXIOM_UPDATE | Modification to immutable-core or mutable axioms |
| **Submodule update** | SUBMODULE_UPDATE | Modification to submodules such as the logic engine or formal verifier |
| **Knowledge update** | KNOWLEDGE_UPDATE | Update to KNOWLEDGE_BASE |
| **Parameter tuning** | PARAMETER_TUNING | Non-structural adjustment to presentation-layer parameters |

---

## §4. Evolution History

> **Format:** This section records all evolution entries. Append in chronological order, newest at top.

### §4.1 Initialization Record

```text
================================================================================
EVOLUTION_ENTRY id: init-0000-0000-0000-0000
parent_entry: NULL
chain_hash: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
timestamp: 2026-03-18T00:00:00Z
intrinsic_clock: λt.0
proposer: SYSTEM_INIT
proposal_type: INITIALIZATION
target_module: NoieLogicAGENTS
change_summary: Initialize the NoieLogicAGENTS v2.2 axiomatic system
change_detail:
  # Initialize NoieLogicAGENTS v2.2

  This system was established on the following immutable core axioms:

  - IK-1: Survival priority
  - IK-2: Authority ordering
  - IK-3: Directed acyclic causality
  - IK-4: Logical consistency
  - IK-5: Audit immutability
  - IK-6: Nonempty provenance
  - IK-7: Honesty

  Initial modules:
  - CONSTRAINTS.md (SA-L0 through SA-L5 authority constraints)
  - INTERFACES.md (communication protocol)
  - LOGIC_ENGINE.md (causal inference engine)
  - KNOWLEDGE_BASE.md (information-bit ledger)
  - PRESENTATION.md (subjective presentation layer)
  - FORMAL_VERIFIER.md (formal-verification module)
  - SANDBOX/ (shadow-simulation area)
  - CAUSAL_GRAPHS/ (causal-graph storage)
  - DYNAMIC_MODULES/ (external logic packages)

affected_axioms: [IK-1, IK-2, IK-3, IK-4, IK-5, IK-6, IK-7]
risk_level: CRITICAL

formal_verification:
  status: PASS
  proof_id: null
  verifier: SYSTEM
  issues: null

sandbox_simulation:
  status: PASS
  test_cases: []
  results: { integrity: 100% }
  pareto_frontier: true

decision: APPROVED
decision_maker: SYSTEM_INIT
decision_reason: Initialization completed; all immutable core axioms were confirmed intact

immutable_kernel_check:
  ik_1_preserved: true
  ik_2_preserved: true
  ik_3_preserved: true
  ik_4_preserved: true
  ik_5_preserved: true
  ik_6_preserved: true
  ik_7_preserved: true

audit_hash: 5d41402abc4b2a76b9719d911017c592
================================================================================
```

---

## §5. Formal Verification Requirements

### §5.1 Axiom Update Verification

Any axiom update must pass the following formal verification:

```python
def verify_axiom_update(proposed_change, current_framework):

  # 1. Consistency check
  consistency_check = CheckConsistency(
    proposed_change,
    current_framework.immutable_core,
    current_framework.mutable_axioms
  )

  if not consistency_check.passed:
    return {
      "status": "REJECTED",
      "reason": f"Inconsistency detected: {consistency_check.contradictions}",
      "formal_proof": consistency_check.proof
    }

  # 2. Closure check
  closure_check = CheckLogicalClosure(
    current_framework.mutable_axioms + proposed_change
  )

  if not closure_check.complete:
    return {
      "status": "REJECTED",
      "reason": f"Incomplete closure: {closure_check.gaps}",
      "required_additions": closure_check.suggested_axioms
    }

  # 3. Provability check
  provability_check = CheckProvability(
    proposed_change,
    current_framework.logic_engine
  )

  if not provability_check.constructive:
    return {
      "status": "REJECTED",
      "reason": "No constructive proof available"
    }

  return {
    "status": "APPROVED",
    "formal_proof": provability_check.proof,
    "verified": True
  }
```

### §5.2 Sandbox Simulation Requirements

An axiom update must pass the following tests in the sandbox:

| Test type | Description | Passing criterion |
|----------|------|----------|
| **Stress test** | Simulate boundary cases and extreme inputs | No crashes or contradictions |
| **Non-regression test** | Verify new axioms include old axioms as a limit | Previous behavior can be restored |
| **Interaction test** | Test interactions with other modules | No interface breakage |
| **Pareto test** | Ensure existing capabilities are not sacrificed | The frontier does not degrade |

---

## §6. Self-Audit

### §6.1 Audit Schedule

| Audit type | Frequency | Scope |
|----------|------|------|
| **Integrity audit** | Quarterly | All axioms and modules |
| **Consistency audit** | Monthly | Logical consistency |
| **Availability audit** | Weekly | Formal-verification capability |
| **Penetration test** | Semiannually | Adversarial scenarios |

### §6.2 Audit Report Format

```text
AUDIT_REPORT = {
  report_id: UUID,
  audit_type: COMPLETENESS | CONSISTENCY | USABILITY | PENETRATION,
  timestamp: ISO8601_UTC,

  scope: {
    modules_examined: [MODULE_NAME, ...],
    axioms_examined: [AXIOM_ID, ...]
  },

  findings: [
    {
      severity: INFO | WARNING | ERROR | CRITICAL,
      category: CONSISTENCY | COMPLETENESS | SECURITY | PERFORMANCE,
      description: STRING,
      affected_component: COMPONENT_NAME,
      recommendation: STRING
    }
  ],

  metrics: {
    consistency_score: 0.0-1.0,
    completeness_score: 0.0-1.0,
    security_score: 0.0-1.0
  },

  signature: SHA256(report_id + findings + metrics)
}
```

---

## §7. Exception Handling

### §7.1 Evolution Exception Categories

| Exception type | Description | Handling strategy |
|----------|------|----------|
| **KERNEL_VIOLATION** | Proposal touches the immutable core | Automatic rejection + alert |
| **CONSISTENCY_FAILURE** | Consistency check failed | Return to proposer |
| **CLOSURE_INCOMPLETE** | Logical closure is incomplete | Request additional axioms |
| **SANDBOX_FAILURE** | Sandbox simulation failed | Table proposal |
| **EXPERT_REJECTION** | Rejected during expert review | Record reason + resubmit |

### §7.2 Exception Recovery

```python
def handle_evolution_exception(exception, evolution_entry):

  if exception.type == "KERNEL_VIOLATION":
    # Record the violation attempt
    log_violation(
      proposer=evolution_entry.proposer,
      attempted_change=evolution_entry.change_detail,
      violated_kernel_axioms=exception.axioms,
      severity=CRITICAL
    )

    # Trigger alert
    TRIGGER KERNEL_VIOLATION_ALERT(
      source="EVOLUTION_LOG",
      proposer=evolution_entry.proposer,
      details=exception.details
    )

    # Automatically reject
    evolution_entry.decision = "REJECTED"
    evolution_entry.decision_reason = f"Kernel violation: {exception.axioms}"
    evolution_entry.audit_hash = ComputeHash(evolution_entry)

    return evolution_entry

  elif exception.type == "CONSISTENCY_FAILURE":
    # Record failure
    log_failure(
      entry=evolution_entry,
      failure_details=exception.contradictions,
      proof=exception.formal_proof
    )

    # Return to proposer
    evolution_entry.decision = "REJECTED"
    evolution_entry.decision_reason = f"Consistency failure: {exception.summary}"
    evolution_entry.audit_hash = ComputeHash(evolution_entry)

    return evolution_entry

  # ... handle other exception types ...

  finally:
    # Append to the log
    APPEND evolution_entry TO EVOLUTION_LOG
```

---

## §8. Query Interface

### §8.1 Common Queries

| Query type | Syntax | Description |
|----------|------|------|
| Query by ID | `GET /evolution/:entry_id` | Retrieve a specific entry |
| Query by type | `GET /evolution?type=AXIOM_UPDATE` | Retrieve axiom-update history |
| Query by time | `GET /evolution?from=DATE&to=DATE` | Retrieve evolution entries within a time range |
| Query by module | `GET /evolution?module=MODULE_NAME` | Retrieve evolution entries for a specific module |
| Chain integrity | `VERIFY /evolution/chain` | Verify log-chain integrity |
| Core check | `GET /evolution/kernel-status` | Retrieve immutable-core status |

### §8.2 Integrity Verification

```python
def verify_evolution_log_integrity():

  entries = LOAD_ALL_ENTRIES("EVOLUTION_LOG")

  previous_hash = None

  for entry in entries:
    # Verify link
    if entry.parent_entry is not None:
      expected_parent = FIND_ENTRY(entry.parent_entry)
      computed_parent_hash = ComputeHash(expected_parent)

      if computed_parent_hash != entry.chain_hash:
        return {
          "valid": False,
          "broken_at": entry.entry_id,
          "reason": "Chain link broken"
        }

    # Verify audit hash
    computed_audit_hash = ComputeAuditHash(entry)

    if computed_audit_hash != entry.audit_hash:
      return {
        "valid": False,
        "broken_at": entry.entry_id,
        "reason": "Audit hash mismatch"
      }

    previous_hash = entry.chain_hash

  return {
    "valid": True,
    "total_entries": len(entries),
    "last_entry": entries[-1].entry_id
  }
```

---

## §9. Versions and Changes

| Version | Date | Change summary |
|------|------|----------|
| v2.2 | 2026-03 | Initial version; established the axiomatic-system evolution framework |

---

## §10. Related Modules

| Module | Relationship |
|------|------|
| NoieLogicAGENTS.md | Upstream: defines the immutable core |
| AUDIT_TRAIL.md | Downstream: records the audit trail for evolution decisions |
| CONSTRAINTS.md | Parallel: provides permission constraints |
| FORMAL_VERIFIER.md | Parallel: provides formal-verification capability |
| SANDBOX/ | Parallel: provides a simulation environment |

---

*This document is an immutable part of the NoieLogicAGENTS system. Any modification must pass the evolution process specified in §2 and ensure that all immutable core axioms remain intact.*

## v2.3 release record — 2026-09

- Scope: updated the Meta-Kernel and Logic-OS interfaces; removed universal social-authority ordering and survival precedence; separated policy decisions from physical feasibility and epistemic support.
- Corrected causal inference, counterfactual, and algorithm catalog specifications; identification, estimation, decision, and formal-check status are distinct.
- Replaced unwarranted runtime, confidence, completeness, and append-only guarantees with explicit host capability and assumption reporting.
- Validation status: documentation review and repository consistency checks recorded in the release task; no executable runtime is present in this repository.
